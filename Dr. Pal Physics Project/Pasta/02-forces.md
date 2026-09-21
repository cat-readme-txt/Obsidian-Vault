# The physical model

Reference, not a stage. Read sections 1 to 4 before milestone M2. Come back to
5 and 6 when you think about cost.

## 1. Why matter makes shapes

Two interactions disagree about length scale.

The nuclear force attracts at one to two femtometers. It repels at separations
of less than about half a femtometer. Alone, it collapses matter into one
drop. The Coulomb force between protons repels over a much longer range.
Alone, it spreads matter out.

At crust densities neither wins. This is **frustration**. Matter picks
structures whose size is set by surface energy against Coulomb energy. The
same competition makes microphase structures in block copolymers, which is why
the shapes look familiar.

As density rises the shapes go, roughly: blobs (gnocchi), rods (spaghetti),
sheets (lasagna), then the inverted versions once matter fills more than half
the volume. Large simulations also find connected networks called **waffle**
and **jungle gym**, which are not in that sequence. If you find one, it is not
a mistake.

## 2. The potential

Horowitz, Pérez-García and Piekarewicz. Two Gaussians plus a screened Coulomb
term between protons:

$$
V_{ij}(r) = a\,e^{-r^2/\Lambda} + \bigl[b + c\,\tau_i \tau_j\bigr]\,
e^{-r^2/2\Lambda} + \frac{q_i q_j\, e^2}{r}\, e^{-r/\lambda} .
$$

| Symbol | Value | Meaning |
|---|---|---|
| $a$ | 110 MeV | short-range repulsion |
| $b$ | $-26$ MeV | isospin-independent attraction |
| $c$ | 24 MeV | isospin-dependent attraction |
| $\Lambda$ | 1.25 fm$^2$ | range parameter |
| $e^2$ | 1.44 MeV·fm | |
| $\tau_i$ | $+1$ proton, $-1$ neutron | |
| $q_i$ | 1 proton, 0 neutron | |

$\Lambda$ has units of area, so $r^2/\Lambda$ is dimensionless with $r$ in fm.

### Flatten the isospin factor

$\tau_i\tau_j$ takes two values, so there are two nuclear potentials.
Precompute them. Never write $\tau_i\tau_j$ in the inner loop.

| Pair | $b + c\,\tau_i\tau_j$ |
|---|---:|
| pp, nn | $-2$ MeV |
| np | $-50$ MeV |

Proton-neutron attraction is twenty-five times stronger than like-pair
attraction. That is real, and it is why pure neutron matter is barely bound.
It also means low proton fraction gives less clustering. Run your first
production job at $Y_p = 0.3$ or 0.4, not 0.1.

Store the model as a table indexed by $(\tau_i, \tau_j)$:

```
struct PairParams { float A, B, K; };   // K = q_i q_j e^2
PairParams table[2][2] = {
  /* n-n */ {110.0f,  -2.0f,  0.0f},
  /* n-p */ {110.0f, -50.0f,  0.0f},
  /* p-n */ {110.0f, -50.0f,  0.0f},
  /* p-p */ {110.0f,  -2.0f,  1.44f},
};
```

Two by two fits in registers and removes every branch from the force kernel.
Do this from the first version.

## 3. Screening

A crust is nucleons in a sea of relativistic degenerate electrons, which
rearrange and cancel excess charge beyond a screening length. This matters for
three reasons, and the third saves your project.

1. It is physically right.
2. It sets the pasta length scale.
3. It makes Coulomb short-ranged. An unscreened $1/r$ in a periodic box needs
   Ewald or particle-mesh summation, which is a semester by itself.

### $\lambda$ is not 10 fm

Most of the literature quotes 10 fm. It is not a constant. **Write the
Thomas-Fermi formula with the unit conversion explicit**, because papers give
it in $\hbar = c = 1$ units and a literal transcription mixes fm$^{-1}$ with
MeV:

$$
k_F\,[\mathrm{MeV}] = \hbar c \left(3\pi^2 n Y_p\right)^{1/3}, \qquad \hbar c
= 197.327\ \mathrm{MeV\,fm},
$$

$$
\lambda\,[\mathrm{fm}] = \hbar c \,\frac{\sqrt\pi} {\left(4\alpha
k_F\right)^{1/2}\left(k_F^2 + m_e^2\right)^{1/4}},
$$

with $\alpha = 1/137.036$ and $m_e = 0.511$ MeV.

| $n$ (fm$^{-3}$) | $Y_p=0.2$ | $Y_p=0.3$ | $Y_p=0.4$ |
|---:|---:|---:|---:|
| 0.01 | 26.6 fm | 23.3 fm | 21.1 fm |
| 0.05 | 15.6 fm | 13.6 fm | 12.3 fm |
| 0.10 | 12.4 fm | 10.8 fm | 9.8 fm |

Reproduce this table as your first test of the formula. If you do not get 13.6
fm at $n=0.05$, $Y_p=0.3$, your units are wrong.

This is not bookkeeping. Alcain et al. showed that too small a $\lambda$ can
produce a one-pasta-per-cell structure that is a pure finite-size artifact: a
clean, convincing, entirely fake result. Horowitz et al. fix $\lambda$ to 10
fm "to agree with the value used in earlier work," which is a choice about
comparability, not physics.

So make $\lambda$ an input parameter, never a compile-time constant. Record it
in every output file. Run at both 10 fm and the Thomas-Fermi value.

## 4. The force

For a Gaussian $V = A e^{-r^2/w}$ of width parameter $w$, the $r$ from the
derivative cancels the $1/r$ in $\hat{\mathbf r}$:

$$
\mathbf F_i = \frac{2A}{w}\, e^{-r^2/w}\,(\mathbf r_i - \mathbf r_j) .
$$

The two Gaussians of section 2 have $w = \Lambda$ and $w = 2\Lambda$. Do not
confuse this $w$ with the fine-structure constant $\alpha$ of section 3.

Collecting both Gaussians, with $B$ from the table in section 2:

$$
\mathbf F^{\text{nuc}}_i = \left[176\, e^{-r^2/1.25} + \frac{B}{1.25}\,
e^{-r^2/2.5}\right] (\mathbf r_i - \mathbf r_j)\ \ \mathrm{MeV/fm}.
$$

Everything is a function of $r^2$. **The nuclear force needs no square root
and no division.** Nothing diverges at $r=0$, where the potential is finite at
110 MeV. You can start from uniformly random positions, and a bad timestep
gives a large energy rather than a `NaN`. A Yukawa or Lennard-Jones form is
not this forgiving.

Coulomb does need $r$. With $k = q_iq_j e^2$:

$$
\mathbf F^{\text{C}}_i = k\,e^{-r/\lambda} \left(\frac{1}{\lambda r^{2}} +
\frac{1}{r^{3}}\right)(\mathbf r_i - \mathbf r_j) .
$$

As $\lambda \to \infty$ this becomes $k\hat{\mathbf r}/r^2$. Check that limit.

## 5. Two ranges, and what they cost

| Part | Practical $r_c$ |
|---|---|
| Nuclear (Gaussians) | 6 fm |
| Screened Coulomb | $2\lambda$ to $2.5\lambda$, so 20 to 55 fm |

The number of neighbors within $r_c$ is $\frac{4}{3}\pi r_c^3 n$. At $n=0.05$
that is about 45 at 6 fm and 1675 at 20 fm. So a single search at the Coulomb
cutoff examines 37 times more pairs than the nuclear term needs.

Only protons feel Coulomb, which softens this. At $Y_p=0.3$ the Coulomb term
costs roughly 150 pair evaluations per particle against 45 for the nuclear
one.

So use two neighbor structures, not one at the largest cutoff. Build a cell
list at 6 fm over all particles. Build a sparser one at the Coulomb cutoff,
over protons only. That is roughly an eightfold saving.

IUMD does a version of this. It uses neighbor lists for the nuclear force and
plain all-pairs for Coulomb. A GPU is fast at regular work and slow at
irregular search. Their lists rebuild every ten steps at high temperature and
density, every hundred at low. Start there, then measure.

So you have a real design choice with a published precedent on one side.
Measure at least two options. That comparison is a result.

## 6. How to choose a run

| Parameter | Range | Note |
|---|---|---|
| $n$ | 0.01 to 0.10 fm$^{-3}$ | less gives separate nuclei, more gives uniform matter |
| $Y_p$ | 0.1 to 0.4 | higher gives sharper phases |
| $T$ | 0.5 to 2 MeV | melts at more than 2 to 3 MeV |

Working point for development: $n = 0.05$, $Y_p = 0.3$, $T = 1$ MeV.

Phase boundaries within that range are a hypothesis for you to locate, not
data to assume. They depend on $Y_p$, $T$, cooling rate and box size.

## 7. What the model leaves out

State this in any writeup. A referee will ask.

- **It is classical.** No Fermi statistics, no Pauli exclusion, no quantum
  kinetic energy. At these densities nucleons are degenerate, so this is not a
  small approximation. Some authors add a Pauli potential. This project does
  not.
- **Parameters are fitted to bulk matter**, not to finite nuclei.
- **The screening length is not solved self-consistently.**
- **No weak interaction**, so $Y_p$ never relaxes.
- **Two-body only.**

The justification is that the model gets the frustration right, and
frustration makes the shapes. It is a model for geometry, not spectroscopy.

## 8. Cutoff treatment

Truncating introduces a discontinuity. A force discontinuity is an impulse,
and an impulse breaks energy conservation.

Use a shifted force:

$$
V^{\text{sf}}(r) = V(r) - V(r_c) - (r - r_c)\,V'(r_c), \qquad F^{\text{sf}}(r)
= F(r) - F(r_c) .
$$

Both energy and force are continuous at $r_c$, for one extra subtraction per
pair.

A shifted *potential* alone is the common textbook choice and the wrong one
here. The force still jumps by $F(r_c)$ whenever a pair crosses, which puts a
floor under the M3 energy-drift plot and destroys the slope-2 line. The drift
from cutoff crossings does not scale as $\Delta t^2$.

Whichever you use, the energy you report and the force you integrate must come
from the same modified potential. If they do not, M2's finite-difference test
fails, and it is right to fail.

## 9. Reading

Verified by search in September 2026 from publisher and arXiv pages. Check
before citing.

- Horowitz, Pérez-García and Piekarewicz, *Phys. Rev. C* **69**, 045804 (2004).
  The potential. Section II.
- Caplan and Horowitz, *Rev. Mod. Phys.* **89**, 041002 (2017).
  arXiv:1606.03646. The review.
- Caplan et al., *Phys. Rev. C* **91**, 065802 (2015). arXiv:1412.8502.
  Section II.2 is the IUMD GPU methods section. Read before Stage 2.
- Alcain et al., *Phys. Rev. C* **89**, 055801 (2014). arXiv:1311.5923.
  Screening length. Section 3 above.

The parameters here were checked against arXiv:1412.8502 and the review. The
Thomas-Fermi table was computed, not copied. Check both once yourself.
