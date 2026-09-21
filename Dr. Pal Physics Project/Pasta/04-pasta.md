# Stage 3: the measurement

About 10 weeks. You need Stage 2 finished and G5 passing.

Sections 1 to 4 are technique: how to cool a system, and how to put a number
on a shape. Sections 5 onward are the research question, and they are a menu.

## 1. Annealing

You cannot start cold. A cold random configuration freezes into the mess it
started in.

1. **Melt.** Equilibrate at 4 MeV or above until the system has forgotten its
   initial configuration. Check with $g(r)$ from two seeds.
2. **Cool** slowly toward 1 MeV.
3. **Equilibrate** at the final temperature. Let the defects anneal out.
4. **Measure** over the last part of the hold only.

### Cooling rate is a result, not a setting

Pasta is an ordered phase forming from a fluid, so it has the problem every
crystallization has: cool too fast and you trap defects. A fast-cooled
configuration can have the right local structure and the wrong global
topology, which means the right physics and the wrong Euler characteristic.

There is no correct rate. There is only a rate slow enough that the answer
stops changing. **So measure it.** Run the same state point at three rates
differing by factors of four and plot your order parameter against rate. Quote
the plateau. If there is no plateau you have not cooled slowly enough, and
that is the finding.

This is the most common way a pasta simulation gets a wrong answer that looks
fine.

### Are you equilibrated

Plot potential energy against time: flat, not drifting, over the measurement
window. Same for the order parameter. Two seeds agree within error bars.

Be honest about the error bar. Successive frames are correlated, so the number
of independent samples is the run length over the correlation time, not the
number of frames you saved. Estimate it from the energy autocorrelation, or
use block averaging. Frenkel and Smit chapter 4.

Optional, at two or three state points: compare cooling into a state point
against heating into it. Disagreement means a first-order transition or a
metastable trap, and either way you must say so.

## 2. The structure factor

Your primary order parameter, and what published papers plot.

$$
S(q) = \frac{1}{N}\left| \sum_{j} e^{i\mathbf q \cdot \mathbf r_j} \right|^2
$$

Compute it for **protons only**, normalizing by $N_p$. Protons mark the
clusters and protons are what neutrinos scatter from. A large $S_p$ at low $q$
means coherent scattering, so a large neutrino opacity, which changes how a
neutron star cools. That is the sentence that justifies this to an astronomer.

A pasta phase peaks at finite $q$. A uniform fluid does not.

### The wavevectors are quantized, and this hurts

In a periodic box only $\mathbf q = (2\pi/L)(n_x,n_y,n_z)$ is allowed, so your
resolution is $\Delta q = 2\pi/L$. That is a hard limit.

| $N$ at $n=0.05$ | $L$ | $\Delta q$ | Shells below $q=0.5$ fm$^{-1}$ |
|---:|---:|---:|---:|
| 4000 | 43.1 fm | 0.146 fm$^{-1}$ | 3 |
| 16000 | 68.4 fm | 0.092 fm$^{-1}$ | 5 |
| 100000 | 126.0 fm | 0.050 fm$^{-1}$ | 10 |

At $N=4000$ you locate a peak with three points. You can quote its position
but not its width. **This is the strongest argument for large $N$**, and it is
quantitative.

Direct summation is $O(N N_q)$, the same shape as your force kernel. It maps
onto the GPU with no new ideas and takes a fraction of a second. Average over
many configurations, not one.

### Do not spherically average first

Look at the unaveraged three-dimensional $S(\mathbf q)$ before you average it.
It names the phase almost by itself.

| Structure | Periodic in | $S(\mathbf q)$ intensity |
|---|---|---|
| Gnocchi | 3 directions | discrete spots, like a Bragg pattern |
| Spaghetti | 2 | a plane perpendicular to the rods |
| Lasagna | 1 | one axis, a pair of spots |
| Uniform fluid | nothing | a featureless sphere |

The dimensionality of the bright region is three minus the dimensionality of
the structure. Free information that spherical averaging throws away.

## 3. Topology

$S(q)$ tells you there is a length scale. It does not distinguish rods from
sheets, or spaghetti from its inverse. For that you need topology.

The measures are the **Minkowski functionals**: volume $V$, surface area $A$,
mean breadth $B$, and Euler characteristic $\chi$. The question is how to get
a surface from a list of positions.

### Use alpha shapes, not voxels

The older route smears positions onto a grid, thresholds the grid, and runs
marching cubes. Do not use it. López, Chávez and Morozov (2025) showed that
cubic voxelization gives "inaccurate estimations of geometric properties", and
replaced it with alpha shapes.

An alpha shape builds the solid directly from the point cloud, through the
Delaunay triangulation, with no grid and no smearing. The 2025 authors compute
theirs with the Python library `Diode`.

It is more accurate and it is less code. Delaunay triangulations and alpha
complexes are also real computational geometry, which is closer to what you
said you wanted than a marching-cubes table.

**The parameter trap moves. It does not disappear.** An alpha shape has a
parameter $\alpha$. It sets the scale at which the point cloud counts as
solid. Too small and the structure shatters. Too large and it fuses. So scan
$\alpha$ and plot $\chi$ against it. A real phase shows a plateau. Report the
plateau and its range. A single $\chi$ at a single $\alpha$ on a single frame
is not a measurement.

### Reading the functionals

Classify on the signs of $B$ and $\chi$ together. $\chi$ counts topology. $B$
carries the sign of mean curvature, which says whether you are looking at
matter or its inverse.

| $B$ | $\chi$ | Structure |
|---|---|---|
| $>0$ | $>0$ | gnocchi |
| $>0$ | $\approx 0$ | spaghetti |
| $\approx 0$ | $\approx 0$ | lasagna |
| $>0$ | $<0$ | waffle or jungle gym |
| $<0$ | $<0$ | anti-jungle gym |
| $<0$ | $\approx 0$ | anti-spaghetti |
| $<0$ | $>0$ | anti-gnocchi, swiss cheese |

That table is a guide to the scheme, not a rule. Derive your own boundaries,
and test them on configurations you can see.

### Your validation target is itself under revision

Section 6 suggests checking against the public Muñoz-López classifiers. Do,
but know what you are comparing to. Those models were trained on voxelized
functionals, the method the 2025 paper calls inaccurate. The two papers also
disagree about whether structure depends on temperature.

So treat them as a reference point, not as ground truth. If your alpha shapes
disagree, that is not automatically your bug.

## 4. Visualization

Export extended XYZ and open it in OVITO. Color by species. Render the protons
only at first, because the neutron gas fills the box and hides everything.

Visualization is for finding gross errors and for communicating a result. It
is not for deciding what phase you have. Human eyes find structure in noise
even when there is none, which is why section 3 exists.

Make a movie of the anneal. It takes an afternoon, and it is the most
effective thing you can put in a talk.

## 5. Survey the field, on a schedule

This is the most important section in the document, and it is why section 6 is
short.

At the start of each stage, and once in the middle of Stage 3, spend half a
day on this. Write the result in a dated `survey.md`:

1. Search for work published since your last survey.
2. **Make sure that anything relevant exists before you believe it.** Find the
   paper, read the abstract, and look at the venue and the year. Citations get
   garbled, including in this document.
3. One line each: what it does, whether it changes what you are doing.
4. If it changes your plan, say so in writing, with the date.

Twenty minutes of searching and two hours of reading, five times a year.

### How to recognize an opening

- **Two papers disagree and nobody settled it.** The 2024 and 2025 papers in
  section 3 disagree about temperature dependence. That question is open.
- **A new method is demonstrated but not applied.** Apply it to an old dataset.
  That is a real and finishable contribution.
- **A paper's last paragraph names unfinished work.** Read every last
  paragraph.
- **A standard technique from a neighboring field is unused here.**
- **A number everyone cites traces to one old paper.** Check whether anyone
  reproduced it.

When you find something, take it. Write down what you found, what you want to
do, and what you are dropping in exchange. Tell your advisor. Update
`survey.md`. Then go.

## 6. The menu

This is a menu, not a queue. It names one default and does not rank the rest.
Pick after you have Stage 2 data and after your own survey.

**Everything here was true in September 2026**, and it will not all be true
when you read it. The entry on the phase diagram, for one, was written as
closed and then reopened by a paper from the year before. Treat the table as a
dated snapshot, not a map of what is possible.

| Direction | Status, September 2026 |
|---|---|
| Phase diagram in $(n,Y_p,T)$ | **Reopened.** Published in 2024 using voxelization. The 2025 paper says voxelization is inaccurate, and nobody recomputed the diagram with alpha shapes. |
| Finite-size effects | Done. Alcain et al. (2014), $A\gtrsim2000$. |
| Screening sensitivity | Done once, in 2014, at small $N$. Not revisited with a code that can afford large boxes. |
| Crust rheology, plasticity | Actively being done by the leaders. Do not race it. |
| Shear viscosity, conductivity | Done, 2008 and 2020. |
| Bulk viscosity of pasta | Did not surface in a search. The 2017 review wants it. Check properly. |
| Precision effects on pasta | Not found. |

### Precision: the recommended default

This is the default for three reasons. It is self-contained, it is the most
purely computational item here, and nobody can take it while you work.

The wider MD literature says coordinate representation, not force arithmetic,
is the dominant error channel, because FP32 coordinates quantize space too
coarsely. MINT32 (Lee, *J. Chem. Inf. Model.* **66**, 4645, 2026) responds by
mapping the box to a 32-bit integer grid.

What is specific here: FP32 spacing at $L=126$ fm is about $7.6\times10^{-6}$
fm against 10 fm structures, a ratio of $10^6$. Protein MD in a 1000 Å box
with 1 Å bonds runs at $10^4$. **Nuclear pasta is a hundred times better
placed on the axis that argument is about.** The force sum, meanwhile, runs
over up to 1700 neighbors. There, naive FP32 accumulation gives about
$5\times10^{-6}$ relative error.

So the hypothesis, written down before you run anything:

> For nuclear pasta, unlike biomolecular MD, the limiting FP32 error channel is
> force accumulation, not coordinate representation.

| Rung | Coordinates | Pair arithmetic | Accumulator |
|---|---|---|---|
| 1 | FP64 | FP64 | FP64 |
| 2 | FP32 | FP32 | FP32 |
| 3 | FP32 | FP32 | FP64 |
| 4 | INT32 fixed point | FP32 | FP32 |
| 5 | INT32 fixed point | FP32 | FP64 or Kahan |

A 32-bit grid over 126 fm gives uniform $3\times10^{-8}$ fm resolution, better
than FP64 at the far corner of the box, at FP32 cost. It also makes minimum
image **exact**: with integer positions the wrap is a bitmask, with no
cancellation and no `nearbyint`.

Rungs 2 and 4 differ only in coordinates, so that pair isolates the coordinate
channel. Rungs 4 and 5 differ only in the accumulator, so that pair isolates
accumulation. If the hypothesis holds, 2 and 4 agree and 5 differs from 4.

Run it at three or four state points including one near a phase boundary.
Compare the $S_p(q)$ peak, $\chi$ and $B$ with error bars over seeds, and
report the cost of each rung on two GPU types.

### Recompute the phase diagram with alpha shapes

Everything needed is public, or something you are building anyway. That is
your own configurations from G5, an alpha-shape pipeline from section 3, and
the Dangelo classifiers to compare against. The question is well posed, the
answer is unknown, and it is finishable.

Two cautions. Make sure that the gap is still there before you commit. This is
the direction most likely to be taken by someone else during your year. It is
also more physics analysis than computation.

### Revisit screening at large $N$

$\lambda$ is density-dependent and usually fixed at 10 fm for comparability
(`02-forces.md` §3). Alcain et al. showed the choice matters and can
manufacture an artifact. They showed it at the box sizes available in 2014.
Your code, if G4 lands, can afford boxes where that artifact cannot close.
This carries higher risk than the precision ladder, and more physics.

### Melting

Fix $n$ and $Y_p$ and heat the system. Locate where the order parameter
collapses, and compare across phases. This is cheap and self-contained, and it
is a good fallback.

## 7. Write your guesses down first

Before each measurement, record what you expect, in a dated file. It takes
five minutes and it changes how you read your own results.

Before starting Stage 3, commit to four falsifiable predictions for whichever
direction you picked. For the precision ladder they are:

1. Which rungs reproduce the FP64 phase assignment.
2. Whether rungs 2 and 4 agree.
3. Where the cell-list crossover in $N$ is.
4. What cooling rate is slow enough.

Some of them will be wrong, and that is the point. A recorded prediction that
fails is a result. An unrecorded one leaves nothing to be surprised by.

## 8. What a skeptical reader will ask

Have a figure for each.

| Question | The figure |
|---|---|
| Is the cutoff converged? | Energy per particle against $r_c$, with a plateau. |
| Is the timestep converged? | The M3 slope-2 plot, plus an observable against $\Delta t$. |
| Is it equilibrated? | Observable against time, two seeds, two starting configurations. |
| Did you cool slowly enough? | Observable against cooling rate, plateau. |
| Converged in $N$? | Observable against $N$. |
| Are the error bars real? | Correlation time or block averaging, stated. |
| How did you assign the phase? | The $\alpha$ scan for $\chi$, and $S(\mathbf q)$. |
| Does GPU agree with CPU? | The three-tier table, `03-cuda.md` §4.6. |
| Why not LAMMPS? | Section 9. Answer it unprompted. |
| Is a classical model appropriate? | `02-forces.md` §7, stated plainly. |

The last one is not a defect to hide. Every paper in this area uses a
classical model and says so.

## 9. Which paper you have

Aim for a methods note in Research Notes of the AAS. It publishes three-page
notes, this community uses it, and a short note on your result is achievable
by April. Talk to your advisor about the code as a separate paper later. A
repository that is public from week one is what keeps that option open.

Three things to get right in any writeup.

Answer "why not LAMMPS" before anyone asks. LAMMPS is open, GPU-capable, and
has produced published pasta results, so "no open code exists" is too strong.
The defensible claim is narrower: there is no purpose-built, tested
implementation of this potential, with a pinned convention set and a measured
precision ladder.

Do not write up rheology, crust breaking or plasticity. Caplan and Bransgrove
named the convergence problem in 2026, and the same group published the
follow-up one month later. To take it on is to race them with their own code.
What you can do instead is make it cheaper: if single precision is adequate,
runs they call "computationally expensive" get two to thirty times cheaper.

The phase diagram is not on that list. Section 6 says why.

If none of it reaches a paper this year, that is not a failure. A tested GPU
code with a public history, a performance analysis and a reproduced phase is a
strong undergraduate project. It is a good basis for a talk, a thesis and a
graduate application.

## 10. Freeze this for the writeup

Code at a tagged commit. Input files and seeds for every figure. Checkpoints
for production runs. Analysis scripts, including the plotting ones. Machine,
GPU model, compiler and flags for every timing. Your predictions file and your
`survey.md`.

The last two are optional for the paper and mandatory for your own honesty.

## 11. Reading

Verified by search in September 2026 from publisher and arXiv pages. Check
before citing.

### Order parameters

- López, Chávez and Morozov, "Characterizing nuclear pasta with alpha shapes",
  *Nucl. Phys. A* **1064**, 123225 (2025). Read before section 3.
- Caplan and Horowitz, *Rev. Mod. Phys.* **89**, 041002 (2017).
  arXiv:1606.03646. The $(B,\chi)$ classification and the state of the field.
- Muñoz and López, *Dynamics* **4**(1), 157–169 (2024). The published phase
  diagram. Code and models public as **Dangelo**. Configurations on request.

### Computation

- Dorso et al., *Nucl. Phys. A* **1002**, 122004 (2020). arXiv:2005.09142.
  Pasta in LAMMPS.
- Lee, "MINT32", *J. Chem. Inf. Model.* **66**(8), 4645 (2026). The fixed-point
  rung.
- Frenkel and Smit, *Understanding Molecular Simulation*, 2nd ed., ch. 4 on
  error estimation and correlation times.

### Know what you are not doing

- Caplan and Bransgrove, *RNAAS* **10**, 107 (2026). arXiv:2605.02101.
- "Plasticity of Neutron Star Crusts", arXiv:2606.06706 (2026). The follow-up,
  one month later.
