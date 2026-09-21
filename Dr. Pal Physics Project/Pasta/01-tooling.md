# Stage 1: build the instrument

About 8 weeks. You need: calculus, Newton's laws, C or C++. You do not need a
GPU.

## 1. Purpose

Build a correct, slow, single-threaded molecular dynamics code, and prove it
is correct. Do not touch CUDA. Do not optimize.

On the CPU at $N=200$, with exact tests and a matrix you can print, a force
sign error takes an hour to find.

On the GPU it can take a month, because **nothing crashes**. A wrong force
still produces a simulation that runs, conserves something, and self-assembles
into some shape. In Stage 3 the right answer is not known in advance. That is
what makes it research, and it is also why a wrong result and an interesting
one look alike.

So the failure mode is not a crash. It is a plausible wrong answer that
invites a physical explanation, and the explanation can be convincing while
being a description of the bug. That is what makes these errors expensive to
find late and cheap to find here.

This code is not throwaway. It is the oracle for every GPU result you produce
for the rest of the year.

## 2. Units

Natural nuclear units throughout. Never convert to SI inside the code.

| Quantity | Unit |
|---|---|
| Length | fm |
| Energy | MeV |
| Time | fm/$c$ ($\approx 3.34\times10^{-24}$ s) |
| Velocity | $c$ |
| Mass | MeV/$c^2$. Nucleon $m = 939$ |
| Temperature | MeV, with $k_B = 1$ |
| Density | fm$^{-3}$. Saturation $n_0 = 0.16$ |

With these, $\ddot{\mathbf r} = \mathbf F/m$ comes out in $c^2/$fm, a velocity
update is in $c$, and a position update is in fm. No conversion factors appear
anywhere.

Take proton and neutron mass as equal. The 1.3 MeV difference does nothing
classically and costs you a branch.

Kinetic energy is $\frac12 m v^2$ with $v$ in units of $c$. Non-relativistic
is correct here: at $T=1$ MeV the typical speed is $0.06c$.

Other constants are in `02-forces.md`.

## 3. The box

Cubic, side $L$, periodic in all directions. **Fix the density and derive the
box:** $L = (N/n)^{1/3}$. $N$ and $n$ are inputs. $L$ never is. An interface
that accepts both $L$ and $n$ admits inconsistent input, and the inconsistency
is silent.

### Minimum image

A pair separation is not $\mathbf r_i - \mathbf r_j$. It is that, shifted into
$[-L/2, L/2)$ per component:

```
dx = xi - xj;
dx -= L * nearbyint(dx / L);
```

Use `nearbyint` or `rint`. A hand-rolled `if (dx > L/2) dx -= L` chain is
wrong once a particle drifts more than one box length. That happens when the
integrator goes unstable, so the two faults appear together and the wrong one
looks like the cause.

This is only valid when $r_c \le L/2$. Past that a particle sees two images of
the same neighbor. Assert it at startup.

This function must be the only place in the program where $L$ appears in a
subtraction.

### Wrapping

Store wrapped positions in $[0,L)$, and keep a separate unwrapped copy if you
ever need diffusion. Wrapped positions keep coordinate magnitudes small, which
matters for floating-point accuracy in Stage 2. The cost is that no
displacement can be computed without the minimum image, anywhere.

### How small can the box be

Coulomb wants $r_c \approx 2\lambda$, and minimum image caps $r_c$ at $L/2$.

| $N$ | $L$ at $n=0.05$ | $L/2$ | Verdict |
|---:|---:|---:|---|
| 200 | 15.9 fm | 7.9 fm | Tests only |
| 1000 | 27.1 fm | 13.6 fm | Still too small for physics |
| 4000 | 43.1 fm | 21.5 fm | Marginal. Fails at the Thomas-Fermi $\lambda$ |
| 16000 | 68.4 fm | 34.2 fm | The working size |
| 100000 | 126.0 fm | 63.0 fm | Morphology unambiguous |

Alcain et al. separately found $A \gtrsim 2000$ is needed to prevent spurious
shell effects. Published production runs reach 51,200 and 409,600.

Stage 1 lives in the top two rows, and that is fine. Correctness tests do not
need many particles. Do not report a phase from $N=1000$ because it rendered
nicely.

## 4. State and layout

Positions, velocities, a species label per particle, plus $L$, time and step
count.

**Store coordinates as three separate arrays** `x[]`, `y[]`, `z[]`, not an
array of structs. On the CPU this makes no difference. On the GPU it is the
difference between a coalesced load and three wasted cache lines per thread.
Changing it later means touching every line of physics code.

Store species as a small integer. In Stage 3 you will index a $2\times2$
parameter table with it.

Keep species out of the force function. It takes a separation and a parameter
struct. A lookup table chooses the struct. If your force function contains the
word "proton", the boundary is wrong.

## 5. The integrator

Velocity Verlet. Nothing else.

$$
\begin{aligned} \mathbf v(t + \tfrac{\Delta t}{2}) &= \mathbf v(t) +
\tfrac{\Delta t}{2}\,\mathbf F(t)/m \\ \mathbf r(t + \Delta t) &= \mathbf r(t)
+ \Delta t\, \mathbf v(t + \tfrac{\Delta t}{2}) \\ \mathbf v(t + \Delta t) &=
\mathbf v(t + \tfrac{\Delta t}{2}) + \tfrac{\Delta t}{2}\,\mathbf F(t+\Delta
t)/m \end{aligned}
$$

One force evaluation per step. Store the force array between steps.

Three properties make this right, and all three are testable. It is
**symplectic**, so energy does not drift secularly. It oscillates in an
envelope whose size scales as $\Delta t^2$. It is **exactly time-reversible**.
It **conserves momentum** to rounding error, given antisymmetric forces.

Do not improve it. Do not add a higher-order term. Do not adapt the timestep.
Each of those breaks one of the three properties, and those properties are
your test suite.

Start at $\Delta t = 0.5$ fm/$c$. Published work uses up to 2 fm/$c$, where
the energy envelope is sixteen times wider. Record $\Delta t$ in every output
file.

## 6. Initial conditions

### Positions

Uniformly random. No lattice, because it is low-entropy and biases you toward
the cubic order you are trying to measure. No overlap rejection, because the
potential is finite at $r=0$.

Assign the first $\lfloor Y_p N \rfloor$ particles as protons, then shuffle.
Record the integer proton count. $Y_p N$ is rarely an integer.

### Velocities

Each Cartesian component from a Gaussian with zero mean and variance $T/m$.
Then, in order: subtract the mean velocity so net momentum is exactly zero,
then rescale to the target kinetic energy.

After subtracting the mean there are $3N-3$ degrees of freedom, so

$$
\langle \mathrm{KE} \rangle = \tfrac{3N-3}{2}\,T .
$$

Using $3N$ makes the thermometer wrong by $N/(N-1)$. At $N=200$ that is half a
percent: large enough to matter, small enough to look like noise.

Seed the generator and record the seed.

## 7. The thermostat

Newtonian dynamics conserves energy. Pasta forms by cooling. So you need to
remove energy, and it is not optional.

### Velocity rescaling

Every $k$ steps multiply all velocities by
$\sqrt{T_{\text{target}}/T_{\text{measured}}}$, with $T_{\text{measured}} =
2\,\mathrm{KE}/(3N-3)$. Crude, does not sample the canonical ensemble
correctly, five lines of code, and adequate for annealing. Use $k$ between 10
and 100. IUMD production runs use 100.

### Langevin

Add friction and noise: $\mathbf F_i \to \mathbf F_i - \gamma m \mathbf v_i +
\sqrt{2\gamma m T}\,\boldsymbol\eta_i$, with $\boldsymbol\eta$ discretized as
a unit Gaussian over $\sqrt{\Delta t}$. Samples the canonical ensemble
correctly and parallelizes trivially. Breaks momentum conservation and
reversibility, so you must be able to turn it off for the tests. Use $\gamma
\sim 10^{-3}$ to $10^{-2}$ $c/$fm.

Make the thermostat a flag with three modes: `nve`, `rescale`, `langevin`. The
tests need `nve`.

## 8. Milestones

### M1. Box, minimum image, checkpoint

*Convincing:* a position wrapped and unwrapped returns bit-identically. No
separation exceeds $L\sqrt3/2$. For a pair straddling a face, an edge and a
corner, the separation matches what you worked out on paper. A checkpoint
written and re-read reproduces the total energy to the last bit. The $r_c \le
L/2$ assertion fires when it must.

### M2. Potential and force

*Convincing:* take twenty random separations in $(0.1, r_c)$, for each of nn,
np and pp. Your analytic force matches a central difference of your own $V$,

$$
F(r) \approx -\frac{V(r+h) - V(r-h)}{2h}, \qquad h = 10^{-5}\ \text{fm},
$$

to relative error below $10^{-8}$. Check against your own $V$, not the formula
on paper. That is what catches a parameter typed wrong in one of the two
functions, which is the common failure.

Also: $V$ and $F$ continuous at $r_c$ after shifting, $\mathbf F_{ij} =
-\mathbf F_{ji}$, force along the line joining the pair.

### M3. The integrator, and the capstone

This is the test that either passes to many digits or does not.

In `nve` mode at $N=200$, from the same initial condition, for a fixed
physical duration, at $\Delta t \in \{2, 1, 0.5, 0.25, 0.125, 0.0625\}$
fm/$c$, record

$$
\varepsilon = \frac{\mathrm{rms}_t\left[E(t) - \langle E
\rangle\right]}{|\langle E\rangle|} .
$$

Plot $\log \varepsilon$ against $\log \Delta t$.

*Convincing:* a straight line of slope $2.00 \pm 0.05$ over at least three
decades. Plus momentum constant to $10^{-14}$ relative. Plus a forward run of
$10^4$ steps, velocities negated, $10^4$ more, returning every position to
within $10^{-9}$ fm.

This plot is the most valuable thing Stage 1 produces. The reduced-precision
versions go on the same axes in Stage 2.

*Slope 1:* a force that is not the gradient of your potential, a discontinuity
at the cutoff, or updates in the wrong order. *Slope 2 but offset high:* fine.
Only the slope is predicted. *Reversibility fails but energy is fine:* you are
wrapping positions inside the integrator and losing information, or the
thermostat is not really off.

### M4. Thermostat and equilibration

*Convincing:* measured temperature settles to target and stays. Two seeds give
the same mean potential energy within error bars. $g(r)$ from the two runs
lies on top of itself. A lattice start and a random start converge to the same
$g(r)$.

Write $g(r)$ now. It is twenty lines and the cheapest diagnostic in the
project. A broken minimum image shows up in it at once, as a sharp feature at
$L/2$.

### M5. The droplet

At $n \le 0.001$ fm$^{-3}$, $N=100$, $Y_p=0.4$, cool slowly from 5 MeV to 0.1
MeV. The particles collect into one drop in vacuum: a classical caricature of
a nucleus.

*Convincing:* binding energy per particle lands in the range of a few MeV, not
0.1 or 100. The drop is roughly spherical. Its radius scales about as
$N^{1/3}$ at $N=50$ and $N=200$.

Do not expect 8 MeV. This model has no quantum kinetic energy in it. The gap
is the subject of `05-coda.md`.

### The test list

Milestones M1 to M5 say what convinces a skeptic. `06-checklist.md` has the
same thing as a flat list of thirteen assertions, nine of which belong to this
stage, with the fast and slow split. Write them before the physics code, and
make one command run all of them.

## 9. Three decisions to record

Decide, pin with a test, write down.

1. **Cutoff treatment.** `02-forces.md` §8 recommends shifted force. M3 fails
   for anything else.
2. **Wrapped or unwrapped storage.** Assert the invariant somewhere.
3. **Where the thermostat acts.** Before the first half-kick, after the second,
   or split. Pick one and never move it silently.

## 10. The checkpoint format

Freeze it at the end of Stage 1 and give it a version number. It holds the
state and the parameters that produced it. The state is positions, velocities,
species, $L$, $N$, $n$, $Y_p$, time, step and $\Delta t$. It also holds the
full potential parameter set including $\lambda$, the thermostat setting, the
RNG seed and state, and the code version.

**`.xyz` is not a checkpoint.** No velocities, no box, no metadata. It is a
visualization export and you need that too, as extended XYZ, which OVITO
reads. Different thing.

A binary array file plus a sidecar JSON manifest is enough. Do not invent a
clever self-describing format. The value is in the manifest.

This is worth an afternoon for four reasons. Stage 2 compares GPU against CPU
from a shared state. Stage 3 compares many runs. Cluster jobs get killed and
must resume. A paper needs a stored input that someone else can run.

## 11. Measure cost from week one

Append a row per run: $N$, $n$, $\Delta t$, steps, wall time, time per step,
particle-steps per second. Three lines of code.

By Stage 2 you have a CPU baseline without going back to measure it. Every
speedup you quote then has a defensible denominator. Quote against your best
CPU code, not your first. A 500x speedup over a Python loop is not a result.

## 12. Not in this stage

CUDA. Neighbor structures. Cutoffs chosen for speed. $N$ of more than about
1000. Structure factors or anything that names a pasta phase. Optimization of
      any kind beyond the layout in section 4.

Stage 1 is allowed to be slow. It is not allowed to be wrong.

## 13. A rhythm, not a schedule

| Weeks | Roughly |
|---|---|
| 1 | Minimum image on paper and in code. Public repo. One test command. |
| 2 | M1. |
| 3 | Read `02-forces.md`. M2, with the finite-difference check. |
| 4–5 | M3. The slope-2 plot and reversibility. |
| 6 | M4. Thermostats, $g(r)$. |
| 7 | M5. |
| 8 | Freeze the checkpoint format. Build and test on the cluster. Write up. |

Two weeks behind at week 5? Drop M5 and Langevin. Keep M3.

## 14. Ways this goes wrong

| Trap | What to do |
|---|---|
| You build a framework instead of a simulation. | Arrays, a force function, a step function, a main loop. That is the whole year. |
| You optimize the CPU code. | Stage 2 replaces the hot loop. Make it readable instead. |
| Minimum image applied in two places, slightly differently. | One function. Grep for `L` and check every hit. |
| You test only with the parameters you will use. | Random separations, $N$, $L$, species mixtures. |
| You skip the finite-difference force check because the algebra was easy. | It is always easy and often wrong. Ten minutes. |
| You change a tolerance to make a test pass. | Find out why it failed. |
| You cannot reproduce last week's run. | Seed, code version, full parameters in every output file. |

## 15. Reading

One chapter of each, not cover to cover.

- Frenkel and Smit, *Understanding Molecular Simulation*, 2nd ed. Chapter 4 for
  the integrator, chapter 6 for thermostats.
- Allen and Tildesley, *Computer Simulation of Liquids*, 2nd ed. Chapter 1 and
  appendix B for minimum image, cutoffs and $g(r)$.
