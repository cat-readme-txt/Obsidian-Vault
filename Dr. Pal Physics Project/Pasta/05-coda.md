# Optional coda: where the classical picture stops

Do this only if Stage 3 finished early, or as a two-week diversion if you need
one. Nothing depends on it.

## 1. Why this exists

Your simulation treats nucleons as classical point particles. They are not.
They are fermions and they obey the Pauli principle. At these densities they
are strongly degenerate, so quantum mechanics does a lot of work that your
code knows nothing about.

Everyone uses a classical model anyway. The quantum calculation is out of
reach at $10^4$ particles, and the frustration that makes the shapes is
classical. Both reasons hold. But there is a difference between an
approximation you have justified and one you have inherited.

The gap is also narrowing. Fore, Kim, Hjorth-Jensen and Lovato applied
neural-network quantum states to exactly this density range and report cluster
formation that other quantum methods miss. They do it at a few tens of
nucleons, not a few hundred thousand. The gap is still enormous, but
"impossible" is now the wrong word.

## 2. The droplet, and its missing pressure

You built this in M5. Turn the density right down, cool slowly, and you get a
self-bound drop: a classical caricature of a nucleus. Measure two things.

### Binding energy

Compute $B/A$ at low temperature for $N$ from about 20 to 300 and plot against
$N$. Compare to the semi-empirical mass formula,

$$
\frac{B}{A} = a_V - a_S A^{-1/3} - a_C \frac{Z(Z-1)}{A^{4/3}} - a_A
\frac{(A-2Z)^2}{A^2},
$$

with roughly $a_V = 15.8$, $a_S = 18.3$, $a_C = 0.71$, $a_A = 23.2$ MeV. Real
nuclei give $B/A \approx 8$ MeV.

Yours will not. The interesting part is the size and sign of the discrepancy,
and whether the $A^{-1/3}$ surface term at least has the right shape. Fit the
same functional form and compare the coefficients you extract.

### Central density, the sharper test

Measure the density inside the drop, away from the surface, and compare to
$n_0 = 0.16$ fm$^{-3}$.

Real nuclei sit at $n_0$ and do not compress further. What stops them is
degeneracy pressure: the Pauli principle forbids two identical nucleons from
sharing a state, so squeezing costs kinetic energy. Your simulation has no
Pauli principle. At $T\to0$ the only thing resisting collapse is a soft 110
MeV Gaussian core.

Predict first, then measure. If your drop is denser than $n_0$, the excess is
a direct measurement of the degeneracy pressure your model is missing. That is
a much better statement than "the model is classical."

The standard patch is a Pauli potential: a momentum-dependent repulsion
between identical nucleons, tuned to mimic exclusion. To implement one is well
beyond a coda. To know that it exists is a paragraph you want.

## 3. The cost figure

One plot. Horizontal axis: number of nucleons, 1 to $10^6$, log. Vertical:
computational cost, log. Four curves.

### Classical MD

Your project. Linear in $N$ with cell lists. You have measured the constant.
Mark where you actually ran.

### Exact diagonalization

For $A$ nucleons in $M$ single-particle states the basis is $\binom{M}{A}$.
For the $sd$ shell, $M=24$, and the basis peaks at about 2.7 million around
$A=12$. That is the entire reachable range.

### Neural-network quantum states

Polynomial rather than exponential, with a published reach of a few tens of
nucleons in this density range. Mark the reach with an arrow. Do not try to
model the scaling.

### Quantum simulation

Under Jordan-Wigner one qubit per single-particle state, so qubit count is
linear in $M$. Gate count is not: roughly $M^4$ terms, times whatever the
algorithm needs.

Caption it: *classical molecular dynamics reaches $10^5$ nucleons by dropping
quantum mechanics. Quantum methods now reach a few tens without dropping it.*

Get the diagonalization and qubit numbers from the person on the nuclear
structure project. Do not derive them yourself. The figure is better with two
sets of measured numbers than one measured and two estimated.

## 4. Talk to the other project

There is a companion undergraduate project on the structure of nuclear
Hamiltonians. It looks unrelated. It is closer than it looks.

### Same ingredients

Both Hamiltonians are short-range nuclear attraction plus Coulomb repulsion
between protons. Yours is classical, at a hundredth of saturation density,
with $10^4$ particles. Theirs is quantum, at saturation, with eight.

### Same discipline

Both freeze a file format in the first stage. Both record the cost of every
calculation from week one, and build a slow correct reference before a fast
one. That was not copied between them. It is what these problems demand.

### Same output shape

Both end with "this approximation costs this much accuracy for this much
speed." Yours is about bits and neighbor lists. Theirs is about rank
truncation. Same sentence.

Produce the section 3 figure jointly. Better figure, better talk.

## 5. Out of scope

Implementing a Pauli potential. Quantum or antisymmetrized molecular dynamics.
Running a quantum circuit. Hartree-Fock or density functional treatment of the
crust. Each is a project. This is a coda, and its job is to say honestly what
your model leaves out and what it costs to put back.

## 6. Reading

- Krane, *Introductory Nuclear Physics*, ch. 3, for the mass formula and
  degeneracy pressure.
- Fore, Kim, Hjorth-Jensen and Lovato, "Investigating the crust of neutron stars
  with neural-network quantum states", arXiv:2407.21207. Read the abstract and
  the reach, not the method.
- The companion project's `../structure/00-start-here.md` and
  `../structure/03-quantum-coda.md`.
