# Stage 2: the GPU

About 12 weeks. You need Stage 1 finished with its tests green. You do not
need prior CUDA.

## 1. Purpose

Move the force calculation to a GPU, make it fast, and be able to say how fast
it can be.

The last part separates this from a tutorial. Anyone can beat a CPU loop. The
skill worth having is different. Work out what the hardware can do, measure
what you got, then explain the gap.

## 2. Where you will run

| Environment | For |
|---|---|
| Interactive notebook or debug queue | Learning CUDA. First kernels. Minutes-long runs. |
| Illinois Campus Cluster | Medium production. Parameter sweeps. |
| NCSA Delta | Large production. Final runs. Real FP64. |

Those are the Illinois options. Any machine with an NVIDIA GPU and a scheduler
works. Set all of them up in the first two weeks. Getting CUDA to build
through a module system takes an afternoon the first time and is unrewarding
to do under deadline.

The FP64 rate is not a constant. Data-center cards such as the A100 run double
precision at about half the single-precision rate. Workstation and consumer
cards run it at a thirty-second or sixty-fourth. The same code can be twenty
times slower on one node type than another for reasons unrelated to your code.
So always record which GPU a timing came from. One binary run on two card
types gives you the FP64 penalty on each, for the cost of a second job script.

Checkpointing is not optional. Production runs exceed walltime limits. Write a
checkpoint every few thousand steps, make the job script resume automatically,
and test the resume path by killing a job. Do this in week 1, not after the
first job dies at hour 23.

## 3. Enough of a mental model

Four facts. Use them to predict, before you run anything, whether a change
will help.

1. **Threads run in warps of 32, in lockstep.** Divergent branches execute both
   sides. This is why `if (proton)` is not free.
2. **Global memory wants contiguous reads.** Thread $k$ reading `x[k]` is one
   transaction per warp. Reading `particle[k].x` is 32. This is why Stage 1
   insisted on separate arrays.
3. **Shared memory is fast and small**, 48 to 164 KB per block. It is a
   programmer-managed cache, and tiling is about using it.
4. **Transcendentals use a separate, narrower pipe**, roughly a quarter the
   rate of the fused multiply-add units, and worse on older cards.

## 4. Milestones

### G1. The naive port

One thread per particle, looping over all $N$, reading positions from global
memory. No tiling. Get it right.

*Convincing:* from the same Stage 1 checkpoint, in FP64, GPU forces match CPU
forces to relative error below $10^{-12}$, component by component. A recorded
timing and a speedup over your best CPU version.

*The problem here:* the force accumulation is a reduction. Writing the
reaction force to particle $j$ needs atomics, which are slow and
non-deterministic. **Do not use Newton's third law here.** Do the full
$N\times N$ loop. Arithmetic is what a GPU has in surplus.

IUMD does the same, and says so: they eschew "even the use of Newton's Third
Law, as it entails branching that would slow the GPU." That reasoning covers
the naive kernel only. See G2.5.

### G2. Shared-memory tiling

Each block of $B$ threads loads a tile of positions into shared memory, syncs,
has every thread loop over the tile, syncs, loads the next. Each global
position is read once per block instead of once per thread.

*Convincing:* identical to G1 within FP64 rounding. A measured speedup with an
explanation. A tile-size scan over $B \in \{32,64,128,256,512\}$ with a plot
and a statement of what limited each. Occupancy and achieved bandwidth from
the profiler.

Predict before you measure. Section 5 argues this kernel is compute bound,
which means tiling helps less than the tutorials imply. Write the prediction
down. Either outcome is interesting. Only one is interesting if you did not.

### G2.5. Newton's third law, revisited (optional, one week)

G1's reasoning does not automatically apply to a tiled kernel. Give one block
a *pair* of tiles. It can then accumulate both the direct and the reaction
force in shared memory, and write each out once, with no global atomics.
Process only tile pairs $(I,J)$ with $I \le J$ and you halve the pair count.

In a compute-bound kernel that is a larger lever than either transcendental
trick in section 5. It is not free: extra shared memory, a second reduction,
worse load balance on the diagonal tiles, a harder correctness argument.

*Convincing:* forces identical to G2. A measured speedup, or a measured
slowdown with a profiler explanation.

Timebox this to one week. "We tried symmetric tiling and it lost because of X"
is a legitimate result.

### G3. Precision

Add a compile-time switch over the precision ladder in `04-pasta.md` §6, then
repeat M3 on the GPU for each rung.

*Convincing:* the M3 plot with several lines. FP64 has slope 2 all the way
down. Reduced precision has slope 2 at large $\Delta t$, then flattens onto a
noise floor. Locate that floor. State the $\Delta t$ at which a smaller step
starts to make the answer worse.

Then the physics half: run the same anneal at each rung from the same
checkpoint and compare an observable. Does the phase change?

*Warning:* runs at different precision diverge exponentially. You compare
statistics, not trajectories. See section 4.6.

### G4. Cell lists

Replace the $O(N^2)$ loop with a spatial grid: bin particles into cells of
side $r_c$, examine only the 27 neighboring cells. Build **two** structures
per `02-forces.md` §5.

Binning is the part that takes the time. Compute a cell index per particle,
sort by it with CUB or Thrust, then find each cell's start and end offset. Do
not write your own radix sort unless you want to, in which case say so in
advance.

*Convincing:* identical energies to G2 at three densities. Rebinning frequency
justified. Timing against $N$ for G2 and G4 on one log-log plot.

The stake, for a $10^6$-step anneal:

| $N$ | Tiled $O(N^2)$ | Cell lists |
|---:|---:|---:|
| 4,000 | ~9 min | ~45 s |
| 16,000 | ~2.4 h | ~3 min |
| 100,000 | ~4 days | ~20 min |

G4 turns a queue-busting job into a coffee break. Check these estimates
against your own timings.

And find the crossover. Your tiled kernel can beat the cell list below some
$N$, because it is regular and the cell list is not. IUMD saw a version of
this across architectures, with their GPU all-pairs sum beating a CPU neighbor
list at 51,200 nucleons. Tiled against cell list on one device is the
measurement nobody published.

### G5. The capstone

Reproduce a pasta phase. $N \ge 16000$, $n = 0.05$ fm$^{-3}$, $Y_p = 0.3$,
annealed from 4 MeV to 1 MeV.

Why 16000 and not 4000. The table in `01-tooling.md` §3 marks 4000 as
marginal. Its box cannot hold the Coulomb range at the Thomas-Fermi $\lambda$,
and that is the size where Alcain et al. found a one-pasta-per-cell artifact.
From the table above, 16000 costs three minutes instead of forty-five seconds.

*Convincing:* $S_p(q)$ has a clear finite-$q$ peak, stable over the last half
of the run and across two seeds. The phase you name from the order parameters
is the phase you see in the picture. The run resumed from a checkpoint at
least once, and you can regenerate it from a stored input and a seed.

### 4.6 Why your GPU trajectory will not match your CPU

It will not, it cannot, and it is not a bug. Address this before it costs you
a week.

Floating-point addition is not associative, so the GPU sums forces in a
different order and differs in the last bits. Molecular dynamics is chaotic: a
$10^{-15}$ difference becomes order 1 after about fifty doublings, which is a
short run.

| What you are checking | How |
|---|---|
| The kernel is correct | Forces from one step, same checkpoint, FP64, to $10^{-12}$ |
| The integrator is correct | $10^2$ to $10^3$ steps, energies to $10^{-10}$ |
| The long run is correct | Statistics only: mean energy, $g(r)$, $S(q)$, phase, with error bars over seeds |

Never test a long run by comparing positions. This table is the answer when
someone asks whether your GPU code gives "the same answer."

It is also why the precision question is real rather than obvious. Reduced
precision does not give a wrong trajectory, because there is no right
trajectory after a few thousand steps. Whether it gives wrong *statistics* has
to be measured.

## 5. What is actually the bottleneck

Work this out on paper, then check.

Per pair the nuclear force needs about 30 to 40 ordinary operations plus two
transcendentals. In a tiled kernel with tile size $B$, arithmetic intensity is
roughly $3B$ operations per byte. At $B=128$ that is several hundred, and
every GPU of the last decade has a roofline ridge point well under 100. **This
kernel is compute bound by a wide margin.**

So tiling helps less than the tutorials suggest, because you were never
bandwidth limited. Optimize the arithmetic instead.

The first thing to try is one exponential in place of two. Since
$e^{-r^2/\Lambda} = (e^{-r^2/2\Lambda})^2$, compute $u = \exp(-r^2/2\Lambda)$
once and square it. That is exact and it is free.

How much it buys depends on which exponential you had. With the accurate
`expf()` the saving is large. With `__expf()`, roughly one hardware
instruction, it is a few percent. A rough count on an A100 also puts this
kernel on the FMA limit rather than the transcendental limit. Working out
which limit you are against, and being wrong, is the exercise.

Measure with Nsight Compute on a single kernel launch: achieved occupancy, SM
against memory throughput, warp stall reasons, branch efficiency, registers
per thread. Use Nsight Systems for the whole application. It shows the
host-to-device copies and the allocations inside the loop that are invisible
in the source. A first GPU port often spends most of its time outside the
kernel.

Do not report a speedup without a profile. "Three times faster" invites "out
of how much available?", and the roofline answers it.

## 6. Why this stays single-GPU

You will not need MPI, and you will not need a second card.

Memory never forces it. Nine numbers per particle is 144 MB at $N=10^6$,
against 16 to 80 GB on the card.

Nor can you decompose the box. Domain decomposition needs each subdomain
thicker than twice the cutoff, and the Coulomb cutoff is about 25 fm:

| $N$ | $L$ | Max useful GPUs, 1D slabs |
|---:|---:|---:|
| $10^4$ | 58 fm | 1.2 |
| $10^5$ | 126 fm | 2.5 |
| $10^6$ | 271 fm | 5.4 |

At $N=10^5$, already larger than most published runs, you get two GPUs, and
about 80% of each slab is halo. That is the physics, not a limit of your code.

One GPU is also enough. IUMD used 128 nodes for 409,600 nucleons largely
because they compute Coulomb by all-pairs: $1.7\times10^{11}$ pair evaluations
per step, against $1.4\times10^{8}$ for a cell list. That factor of 1200 is
larger than the factor of about 800 between 128 Kepler cards and one A100. So
one modern GPU with cell lists is in the same league as their whole campaign.
Measure that and put it in your writeup.

The throughput you want is free anyway. Stage 3 is a sweep of independent
runs, so submit a Slurm array with one GPU per task.

If you still want to try it, talk to your advisor first. The short answer is
that a two-GPU slab decomposition is worth doing as a measurement, not as a
feature.

## 7. Tests that must keep passing

Every Stage 1 test still applies. In addition, see `06-checklist.md` items 10
to 13. The one people skip is resume equivalence, and it is the one that ruins
a production campaign.

Keep G1 in the repository forever as the reference kernel. It is slow and
correct, and it reduces a tiling bug from a fortnight of guessing to a
five-minute diff.

## 8. Ways this goes wrong

| Trap | What to do |
|---|---|
| You optimize before it is correct. | G1 first, and keep it. |
| You use atomics for Newton's third law in G1. | Do the full loop. |
| You allocate or copy inside the timestep loop. | Allocate once. Look with Nsight Systems. It is invisible in the source. |
| You forget `cudaDeviceSynchronize()` before a timer. | Launches are asynchronous, so the timer measures the launch, not the kernel. A thousandfold speedup is the symptom. Check the error code too. |
| You debug on the cluster. | Reproduce at $N=200$ interactively. `compute-sanitizer` finds races in seconds. |
| You compare GPU and CPU trajectories. | Section 4.6. |
| You report a speedup against your slowest CPU code. | Against your best. State CPU, compiler and flags. |
| You blame the GPU for a physics bug. | Run the same input through the Stage 1 code. That is what it is for. |

## 9. A rhythm, not a schedule

| Weeks | Roughly |
|---|---|
| 1 | Accounts, modules, hello-world kernel everywhere. Checkpoint-resume in the job script. |
| 2–3 | CUDA basics on toy problems: vector add, a reduction, then N-body with Lennard-Jones. Not your own physics. |
| 4–5 | G1 and the CPU-GPU force test. |
| 6–7 | G2, the tile scan, the first profile. |
| 8 | G3. |
| 9–10 | G4. |
| 11 | G5. |
| 12 | Write up: performance table, roofline, scaling plot, precision plot. |

Weeks 2–3 are not wasted. Learning CUDA on a problem whose answer you know is
much faster than on one you do not.

Three weeks behind at week 8? Drop G4 and do G5 with the tiled kernel at
$N=16000$. A finished G5 beats an unfinished cell list.

## 10. Reading

- Nyland, Harris and Prins, "Fast N-Body Simulation with CUDA", *GPU Gems 3*
  ch. 31. Free online. The tiling pattern, by the people who wrote it. Read
  before G2.
- NVIDIA *CUDA C++ Programming Guide*, memory hierarchy and performance
  chapters. Reference.
- NVIDIA *CUDA C++ Best Practices Guide*. Read the coalescing section properly.
- Nsight Compute docs on speed-of-light and roofline. Short, and it is what
  makes the profiler output mean something.
