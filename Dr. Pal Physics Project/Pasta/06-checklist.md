# One page

Print this. Everything here is explained in the other documents. This page is
the index, not the instructions.

## Week one, before anything else

- [ ] **Make the repository public.** Not later. `00-start-here.md` §6.
- [ ] One command that runs all tests. It will be empty. That is fine.
- [ ] Accounts and a compiling CUDA hello-world on every machine you will use.
- [ ] A file called `predictions.md`. Date every entry.
- [ ] A file called `survey.md`. First literature survey goes in it now.

## The spine

Finish these and the project is a success. Everything else is stretch.

| | Item | Done when |
|---|---|---|
| M1 | Box, minimum image, checkpoint | No separation is more than $L\sqrt3/2$. The checkpoint round-trips bit-identically |
| M2 | Potential and force | Analytic force matches central difference to $10^{-8}$, all three pair types |
| M3 | Velocity Verlet | **Energy error vs $\Delta t$ has slope $2.00 \pm 0.05$** over three decades |
| G1 | Naive CUDA kernel | FP64 forces match CPU to $10^{-12}$ from the same checkpoint |
| G2 | Shared-memory tiling | Matches G1. Tile-size scan. One profile you can explain |
| G5 | One pasta phase | $N \ge 16000$, annealed, stable $S_p(q)$ peak over two seeds |
| | Survey | Redone at the start of each stage, written down, dated |
| | One Stage 3 result | The precision ladder at one state point is the default. Pick your own if you find something better |
| | Release | Tagged, documented, reproducible from a stored input and a seed |

## Stretch

G4 cell lists. Alpha-shape topology. G2.5 symmetric tiling. The ladder at more
state points. The phase diagram recomputed with alpha shapes. Screening at
large $N$. Melting. The coda.

Not a queue. Pick from it, and add to it when your survey turns something up.

## Signals of an open question

`04-pasta.md` §5 has the worked examples.

- Two papers disagree and nobody settled it
- A new method is demonstrated but not applied to the old data
- A paper's last paragraph names unfinished work
- A standard technique from a neighboring field is unused here
- A number everyone cites traces to one paper nobody reproduced

## Tests that must always pass

1. Minimum image: nothing is more than $L\sqrt3/2$. Face, edge and corner hand-checked
2. Force is the gradient: finite difference, per pair type, random separations
3. Newton's third law: $\sum_i \mathbf F_i = 0$ to rounding
4. Cutoff continuity: $V$ and $F$ both zero at $r_c$ after shifting
5. Momentum conserved to $10^{-14}$ in NVE
6. Energy scaling: slope 2
7. Reversibility: forward, flip velocities, back, within $10^{-9}$ fm
8. Equipartition: $2\,\mathrm{KE}/(3N-3)$ equals the set temperature
9. Checkpoint round-trip: energy bit-identical
10. CPU-GPU agreement: forces, one step, $10^{-12}$
11. Kernel variants agree: G1 = G2 = G4
12. Determinism: same binary, input, seed, device → bit-identical
13. Resume equivalence: 2000 steps = 1000 + resume + 1000, bit-identical

Tests 1 to 9 are CPU. Tests 10 to 13 are GPU. Split fast from slow, and run
the fast suite on every commit.

## Numbers you will keep needing

| | |
|---|---|
| Nucleon mass | 939 MeV |
| $a$, $b$, $c$, $\Lambda$ | 110, $-26$, $+24$ MeV, 1.25 fm² |
| $b + c\tau_i\tau_j$ | $-2$ MeV like pairs, $-50$ MeV np |
| $e^2$ | 1.44 MeV·fm |
| $\lambda$ | 10 fm by convention. 10 to 27 fm by Thomas-Fermi |
| Saturation density | 0.16 fm⁻³ |
| Pasta range | $n$ 0.01–0.10 fm⁻³, $Y_p$ 0.1–0.4, $T$ 0.5–2 MeV |
| Working point | $n = 0.05$, $Y_p = 0.3$, $T = 1$ MeV |
| $\Delta t$ | 0.5–2 fm/$c$ |
| Box | $L = (N/n)^{1/3}$, and $r_c \le L/2$ always |

## The five traps that cost weeks

1. Minimum image applied in two places, slightly differently.
2. Shifted potential instead of shifted force. This kills the slope-2 test.
3. The $\lambda$ formula transcribed without the $\hbar c$. `02-forces.md` §3.
4. A GPU and CPU *trajectory* compared. You cannot. `03-cuda.md` §4.6.
5. A phase reported from a box too small to hold the Coulomb range.

## Before you claim a result

- Cooling rate plateau shown
- Two seeds agree within error bars
- Error bars from independent samples, not frames
- $\alpha$ scan for $\chi$ shows a plateau
- Converged in $N$
- Prediction recorded before the measurement
