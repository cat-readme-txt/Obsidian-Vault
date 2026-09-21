# Start here

An undergraduate project on simulating nuclear pasta on a GPU.

Level: physics or computer science undergraduate. Length: one academic year,
part time. You need: calculus, Newton's laws, and some C or C++. You do not
need: nuclear physics, or any prior CUDA.

## 1. The project in plain words

Deep in a neutron star crust, matter is squeezed to about a tenth of the
density inside a nucleus. At that density the nucleus stops being a ball.

Two forces disagree. The nuclear force pulls nucleons together over one or two
femtometers. The electric force pushes protons apart over a much longer range.
Neither wins. Matter picks a compromise shape: blobs, then rods, then sheets,
then the same shapes inverted as density climbs. These are the pasta phases.

You can get those shapes from Newton's laws. Put a few thousand particles in a
box, give them a potential with short-range attraction and long-range
repulsion, cool them, and watch them self-assemble. The physics input is one
function of one variable.

The cost is dominated by one operation: for each particle, sum the force from
every other particle. That is the best-understood problem in GPU computing. It
is also one where a naive version and a good version differ by a large factor
that you can measure.

## 2. The question

> You observe a phase at some density, proton fraction and temperature. How
> much does that observation depend on things that are supposed to be
> invisible: the number of particles, the box, the cooling rate, and the number
> of bits in the arithmetic?

The first half is reproduction, with published answers to check against. The
second half is open.

## 3. Why it fits a student who wants to compute

### The physics is one function

Everything the simulation knows about nuclear matter is the potential in
`02-forces.md`.

### The tests are exact

Momentum conserves to rounding error. The integrator is exactly reversible.
The energy error falls as the square of the timestep, with a slope of exactly
2 on a log-log plot. You never have to wonder whether the code is right.

### The performance work is real

A naive kernel, a tiled kernel, and a spatial grid are three genuinely
different programs. You measure each, then explain the gap between what you
got and what the hardware can do.

### The result is a picture

When it works, you can see it. That matters more in February than it does in
September.

## 4. The shape of the year

| Stage | What you do | About |
|---|---|---|
| 1 | Build a correct CPU reference. Prove it is correct. | 8 weeks |
| 2 | Port to CUDA. Tile it. Bin it. Profile it. | 12 weeks |
| 3 | Pick a question and answer it. | 10 weeks |
| Coda | Optional. Where the classical picture breaks. | if time remains |

Stage 1 is slow on purpose. A force error left there does not crash anything.
It produces a plausible wrong answer in Stage 3, where there is no known
result to check it against.

### The spine

Thirty part-time weeks, with no CUDA experience at the start, have no slack.
Stage 2 will probably run long, so decide now what survives if it does.

The spine is:

- A public repository from week one, and a dated survey each stage.
- M1 to M3, which is correctness.
- G1 and G2, which is a working tiled kernel with a profile.
- G5, one annealed pasta phase at $N \ge 16000$.
- One Stage 3 question answered properly.
- A tagged release that runs from a stored input and a seed.

`06-checklist.md` carries the same list with acceptance criteria.

Everything else is stretch: cell lists, the topology pipeline, more state
points, and the coda. That is not a demotion. The spine is a complete piece of
work.

Re-read this in January and be honest about which list you are on.

## 5. Which half of this plan to trust

Stages 1 and 2 are prescriptive on purpose. Units, the minimum image, the
integrator, the test order: these are craft with known right answers, and
getting them wrong costs months. Follow them.

Stage 3 is not prescriptive. Where it reads that way, push back on it.

The plan will go stale. Stage 3 was rewritten three times while this document
was prepared. Each pass found published work that changed what was open, and
one pass reopened a question an earlier pass had closed. That is what an
active field looks like from inside. For that reason `04-pasta.md` §5 makes
the literature survey a recurring milestone.

Three rules follow.

- **Stage 3 is a menu, not a queue.** Adding something not on it is encouraged.
- **A surprise beats the plan.** Chase it. Tell your advisor what you found and
  what you want to drop in exchange.
- **Write your guess down first.** Then a contradiction is a finding, not a bug
  hunt.

Two things are fixed, because later work depends on them: the conventions in
`02-forces.md`, and a documented checkpoint format.

## 6. Where a paper can come from

Much of the physics is a reproduction. Assume a referee has read everything
you have.

But the field is more open than a first look suggests. Between two drafts of
this plan, a 2025 methods paper turned out to reopen a question a 2024 paper
appeared to have closed. There is probably another.

### The realistic target is a methods note

Research Notes of the AAS publishes three-page notes, and this community
publishes in it. Achievable by April.

### The code is worth publishing later

Not this year, and be careful with the claim: LAMMPS is open, GPU-capable, and
has produced published pasta results. "No open code exists" is too strong.

### The one decision you must take in week one

**Make the repository public now, and commit steadily all year.**

JOSS rejects any repository with less than six months of public history, and
it runs automated checks on commit distribution. Seven months of real history
from September clears that. A private year and an April release does not, and
cannot be fixed retroactively. `04-pasta.md` §9 has the detail.

## 7. What you will be able to do afterwards

- Write, debug and profile a CUDA kernel, and explain its performance against
  what the hardware can do.
- Choose a data structure and measure what the choice bought you.
- Test numerical code against cases with exact answers.
- Reason about floating-point error in a long calculation.
- Survey a literature and find your own question in it.
- Report an approximation's error honestly.

## 8. How to start this week

1. Write a program that puts 100 points at random in a cubic box of side 20.
   Print the distance from point 0 to every other point, under the minimum image
   convention. Make sure by hand that no distance is more than
   $\frac{\sqrt3}{2}\times 20 \approx 17.3$.
2. Read `01-tooling.md` sections 1 to 4. Stop there.
3. Start M1.

Step 1 takes an hour. A broken minimum image is silent, and every later result
depends on it.

## 9. Two honest notes

### The long run gets short

A pasta anneal is $10^5$ to $10^6$ steps. With your first GPU kernel that is
overnight, and days at $N=10^5$. With cell lists it is minutes. That gap is
the argument for Stage 2.

### One GPU is enough

You will not need MPI. `03-cuda.md` §6 says why, and the throughput you want
comes from submitting many independent runs.

## Documents

| File | What it is |
|---|---|
| `01-tooling.md` | Stage 1. Units, box, integrator, tests. |
| `02-forces.md` | Reference. The potential and its traps. |
| `03-cuda.md` | Stage 2. The port, tiling, binning, profiling. |
| `04-pasta.md` | Stage 3. Annealing, order parameters, the menu. |
| `05-coda.md` | Optional. The bridge to the quantum project. |
| `06-checklist.md` | One page. Print it. |
