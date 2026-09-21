# Stage 1: build the instrument

About 10 weeks. September to November. You need: linear algebra, and a
programming language of your choice. You do not need: quantum mechanics.

This document is self-contained. Every convention, formula, and number that
Stage 1 needs is written down here.

## 1. Purpose

Build the code that turns a physical rule into a matrix, and make sure that it
is correct.

At the end of this stage you can build a small many-particle Hamiltonian and
find its eigenvalues, and you can prove that your signs and labels are right.
You do not touch angular-momentum recoupling, radial integrals, or any
interaction file format. Those come later, and most of them you will be given.

Everything after this depends on this stage. A sign error that you leave here
costs a week to find in Stage 3. The same error costs an hour to find now,
because the answers here are exact.

## 2. The five facts you need

This is the whole physics background for Stage 1. Everything else in this
document is linear algebra, bit operations, and tests.

### Fact 1. A state is a pattern of occupied slots

The system has a fixed list of single-particle **slots**. A slot is one place
that one particle can sit. A many-particle state says which slots are full.
Store it as an integer, one bit per slot. Bit $i$ set means slot $i$ is full.

### Fact 2. Particles of this kind are fermions

Two identical fermions cannot share a slot, so each bit is 0 or 1. Swapping
two particles changes the sign of the state. That sign is the only hard part
of the bookkeeping, and section 5 gives the rule.

### Fact 3. The Hamiltonian is a Hermitian matrix

Index its rows and columns by the patterns from Fact 1. Its eigenvalues are
the energies of the system. Its lowest eigenvalue is the ground-state energy.

### Fact 4. Operators move particles between slots

Write $a^\dagger_i$ for the operator that fills slot $i$, and $a_i$ for the
operator that empties it. Each acts on one pattern and gives back one pattern
and a sign, or gives zero. From these two you can build every operator in the
project.

### Fact 5. Every Hamiltonian has the same shape

$$
H = \sum_{pq} t_{pq}\, a^\dagger_p a_q
  + \frac{1}{4} \sum_{pqrs} V_{pqrs}\, a^\dagger_p a^\dagger_q a_s a_r
$$

The numbers $t_{pq}$ and $V_{pqrs}$ are the input. The first sum is the
**one-body** part: it moves one particle. The second sum is the **two-body**
part: it moves two, and it is where the interaction between particles lives.
The factor $1/4$ goes with a particular meaning for $V$, and section 6 gives it.

That is all. If a formula later in this project needs more quantum mechanics
than this, the plan gives you the formula and its source.

## 3. Start small, then generalize

Do not start with the full 24-slot space. Start with **one orbit of 6 slots**,
which is the $0d_{5/2}$ orbit for one species of particle. With 2 particles it
has 15 states. You can print the whole matrix and read it.

Work in this order.

1. 6 slots, 2 particles. Print matrices. Build the same matrix by hand and
   compare the two. 2. 6 slots, any number of particles. Make sure that the
   dimensions agree with the binomial coefficients. 3. Several orbits, one
   species, 12 slots. 4. Both species, 24 slots. This is the full space of
   section 4.

Each step reuses the code from the step before. If a step needs you to rewrite
the layer below it, the boundary between your layers is in the wrong place.

## 4. The slots

The model space is the `sd` shell: 24 slots, 12 for protons and 12 for
neutrons. Each slot carries five labels, $(n, l, 2j, 2m, 2t_z)$.

The names come from nuclear physics, and you can treat them as labels. $n$ and
$l$ say which orbit the slot belongs to. $j$ is the angular momentum of that
orbit, and $m$ is its projection, which runs in integer steps from $-j$ to $+j$.
$t_z$ says whether the slot holds a proton or a neutron.

### 4.1 Store doubled integers

$j$, $m$, and $t_z$ are half-integers. **Store $2j$, $2m$, and $2t_z$ as
integers.** Never store them as floats.

| Quantity | Proton | Neutron |
|---|---|---|
| $2t_z$ | $-1$ | $+1$ |

Within one orbit $2j$ and $2m$ always have the same parity, so $(2j-2m)/2$ is
exact integer arithmetic and $j-m$ is always a whole number. Phase factors like
$(-1)^{j-m}$ are then an integer parity, and never a floating-point power:

```
sign = +1 if ((2j - 2m) // 2) % 2 == 0 else -1
```

#### The sign of $t_z$

Here the *neutron* has $t_z = +1/2$. This is the nuclear-structure convention,
and it is the one used by the papers and tables you will read. It gives

$$
T_z = \sum_{\text{filled}} t_z = \frac{N - Z}{2},
$$

which is the standard nuclear definition, and which is positive for a nucleus
with more neutrons than protons.

Particle physics uses the opposite sign, with $t_z = +1/2$ for the proton. So do
not copy a $t_z$-dependent formula out of a paper without checking which
convention that paper uses.

In this model the sign is not load-bearing: the interaction treats both species
alike and there is no Coulomb term, so $t_z$ only says which of two otherwise
identical blocks a slot lives in. It is fixed now because it stops being free
the moment an isospin-dependent interaction or a Coulomb term appears.

### 4.2 The orbits

Three orbits, ordered by decreasing $j$.

| Orbit index | Label | $n$ | $l$ | $2j$ | Number of $m$ states | Offset in the block |
|---:|---|---:|---:|---:|---:|---:|
| 0 | $0d_{5/2}$ | 0 | 2 | 5 | 6 | 0 |
| 1 | $0d_{3/2}$ | 0 | 2 | 3 | 4 | 6 |
| 2 | $1s_{1/2}$ | 1 | 0 | 1 | 2 | 10 |

So the orbit offsets are $(0, 6, 10)$ and each species block holds 12 slots.

Within an orbit, $m$ ascends, from $-j$ to $+j$. One consequence is used
later: the states with $m > 0$ are the **tail slice** of that orbit's block.
The pairing operator of section 8 then needs a slice, and not a search.

### 4.3 The full table

This is the proton block. The neutron block is identical, with $2t_z = +1$ and
every index shifted by 12. Protons keep the low bits, so the sector label $(Z,
N, 2M)$ of section 5.3 reads in the same order as the bits.

| Index | Orbit | $n$ | $l$ | $2j$ | $2m$ | $2t_z$ |
|---:|---|---:|---:|---:|---:|---:|
| 0 | $0d_{5/2}$ | 0 | 2 | 5 | $-5$ | $-1$ |
| 1 | $0d_{5/2}$ | 0 | 2 | 5 | $-3$ | $-1$ |
| 2 | $0d_{5/2}$ | 0 | 2 | 5 | $-1$ | $-1$ |
| 3 | $0d_{5/2}$ | 0 | 2 | 5 | $+1$ | $-1$ |
| 4 | $0d_{5/2}$ | 0 | 2 | 5 | $+3$ | $-1$ |
| 5 | $0d_{5/2}$ | 0 | 2 | 5 | $+5$ | $-1$ |
| 6 | $0d_{3/2}$ | 0 | 2 | 3 | $-3$ | $-1$ |
| 7 | $0d_{3/2}$ | 0 | 2 | 3 | $-1$ | $-1$ |
| 8 | $0d_{3/2}$ | 0 | 2 | 3 | $+1$ | $-1$ |
| 9 | $0d_{3/2}$ | 0 | 2 | 3 | $+3$ | $-1$ |
| 10 | $1s_{1/2}$ | 1 | 0 | 1 | $-1$ | $-1$ |
| 11 | $1s_{1/2}$ | 1 | 0 | 1 | $+1$ | $-1$ |
| 12 to 23 | the neutron block | | | | | $+1$ |

The index of a slot is

$$
\mathrm{idx} = 12\,\tau + \mathrm{offset}(\text{orbit}) + (m + j),
$$

where $\tau$ is the species index,

$$
\tau = t_z + \tfrac{1}{2} = \frac{2t_z + 1}{2},
$$

so $\tau = 0$ for a proton and $\tau = 1$ for a neutron. In doubled units,
$m + j = (2m + 2j)/2$.

**Indices are labels. Never derive them from the single-particle energies.**
Some tests need all the energies equal, and Stage 2 varies them. If your index
order depends on the energies, the basis relabels itself in the middle of a
study and every matrix you stored becomes meaningless. Keep the energies in a
separate array with its own index.

## 5. Many-particle states

A many-particle state is a 24-bit integer, the **mask**. Bit $i$ set means
slot $i$ is full. Bits 0 to 11 are protons, and bits 12 to 23 are neutrons.

### 5.1 The sign rule

Define the reference pattern by filling slots in **ascending index order**:

$$
\lvert n_0\, n_1 \ldots n_{23} \rangle
= (a^\dagger_0)^{n_0} (a^\dagger_1)^{n_1} \cdots (a^\dagger_{23})^{n_{23}}
  \lvert 0 \rangle .
$$

That definition forces the sign rule. It is not a free choice:

$$
a^\dagger_i \lvert n \rangle
= (-1)^{\sum_{k<i} n_k}\, (1 - n_i)\, \lvert \ldots n_i{+}1 \ldots \rangle ,
\qquad
a_i \lvert n \rangle
= (-1)^{\sum_{k<i} n_k}\, n_i\, \lvert \ldots n_i{-}1 \ldots \rangle .
$$

The exponent counts the filled slots **strictly below** $i$. In bit terms:

```
parity = popcount(mask & ((1 << i) - 1))
```

### 5.2 Derived quantities

From a mask you read off, with `popcount` the count of set bits:

```
Z   = popcount(mask & 0x0FFF)            # protons
N   = popcount((mask >> 12) & 0x0FFF)    # neutrons
A   = Z + N
2M  = sum of 2m over the filled slots    # an integer, equal to 2*J_z
2Tz = sum of 2tz over the filled slots   # an integer, equal to N - Z
```

$A$ counts the particles in these 24 slots only. In nuclear language these
slots sit outside a filled `16O` core, and $A$ does not include the 16 core
particles.

All energies are measured **relative to that core**. The core energy is zero,
so the one-body term is just $\sum_p \varepsilon_p n_p$ over the 24 slots. There
is no core-energy constant anywhere in this project.

### 5.3 Sectors

The Hamiltonian does not mix states with different $Z$, different $N$, or
different $2M$. So the matrix is block diagonal, and each block is a
**sector** labeled $(Z, N, 2M)$. Build and diagonalize one sector at a time.
This is the cheapest speed-up in the project, and you get it for free from the
enumeration.

## 6. The two-body convention

This is the one place where a silent factor of 2 or 4 can hide for months.
Settle it in Stage 1 and write down what you settled.

The Hamiltonian of Fact 5 is

$$
H = \sum_{pq} t_{pq}\, a^\dagger_p a_q
  + \frac{1}{4} \sum_{pqrs} V_{pqrs}\, a^\dagger_p a^\dagger_q a_s a_r .
$$

Note the operator order: $s$ comes before $r$.

Here $V_{pqrs}$ is the **antisymmetrized** matrix element,

$$
V_{pqrs} = \langle pq \rvert V \lvert rs \rangle
         - \langle pq \rvert V \lvert sr \rangle ,
$$

and all four indices run over all 24 slots, with no restriction. The $1/4$ is
tied to that choice. The two common forms are

| Meaning of $V_{pqrs}$ | Prefactor | Operator order |
|---|---:|---|
| antisymmetrized, $\langle pq \rvert V \lvert rs \rangle - \langle pq \rvert V \lvert sr \rangle$ | $1/4$ | $a^\dagger_p a^\dagger_q a_s a_r$ |
| plain, $\langle pq \rvert V \lvert rs \rangle$ | $1/2$ | $a^\dagger_p a^\dagger_q a_s a_r$ |

The two agree. Relabel $r \leftrightarrow s$ in the second term and use
$a_r a_s = -a_s a_r$:

$$
\frac{1}{2} \sum_{pqrs}
 \bigl[ \langle pq \rvert V \lvert rs \rangle
      - \langle pq \rvert V \lvert sr \rangle \bigr]
 a^\dagger_p a^\dagger_q a_s a_r
= \sum_{pqrs} \langle pq \rvert V \lvert rs \rangle\,
  a^\dagger_p a^\dagger_q a_s a_r .
$$

Do not take that on trust. Milestone M3 asks you to make sure that it holds
numerically.

If you ever restrict a sum, for example to $p < q$ and $r < s$, the
prefactor changes. Re-derive it for that code path and record it. Do not carry
$1/4$ across.

## 7. Keep the physics out of the machinery

Your code has three layers. Only the third contains any physics.

| Layer | What is in it | Physics |
|---|---|---|
| Slots and patterns | The slot table, the index formula, the enumeration of a sector, the bit operations. | none |
| Algebra | $a^\dagger$ and $a$ on a pattern, the sign rule, a list of terms turned into a sparse matrix, and a slow dense version to compare it against. | none |
| Physics input | The numbers $t_{pq}$ and $V_{pqrs}$ for a particular interaction. | one page, given |

One design rule makes this work. **The algebra layer takes numbers, not
physics.** It must accept any $t$ and any $V$ and build the correct matrix. The
interactions are then data that you hand to it.

You can test that boundary. Feed the assembler random $t$ and random $V$. If the
matrix is Hermitian, has the right sector structure, and matches a slow dense
reference, your machinery is correct, and no physics was involved in proving it.

If you ever find angular-momentum logic inside the matrix assembler, the
boundary moved. Put it back.

## 8. Milestones

Five milestones. Each one names what convinces a skeptic that it works.

### M1. Slots and states

You can build the slot table of section 4, enumerate every state in a sector
$(Z, N, 2M)$, and map a mask to an index and back.

*Convincing:* your table matches section 4.3 at the block boundaries, that is at
indices 0, 5, 11, 12, 17, and 23. Inside each orbit $2m$ increases by 2 per
step. The $m > 0$ states are exactly the tail slice. Your state counts match
the binomial coefficients. The $Z$ and $N$ you read from `popcount` agree with
counting the filled slots by hand.

### M2. The operators $a^\dagger$ and $a$

You can apply them to a pattern and get the right sign and the right new
pattern.

*Convincing:* $a^\dagger_p a^\dagger_q = -a^\dagger_q a^\dagger_p$ and
$a^\dagger_p a^\dagger_p = 0$, as operators on the whole space and not only on
one state. A two-particle case reproduces a matrix that you built by hand.

### M3. The assembler

You can turn any $t$ and $V$ into a sparse matrix.

*Convincing:* the matrix is Hermitian. Sparse and dense agree. Random input
works. The two forms in the table of section 6 give the same matrix on random
input, which settles the $1/4$. With an interaction that treats both species
alike, the neutron-neutron block equals the proton-proton block, that is
$H_{nn}[i,j] = H_{pp}[i-12, j-12]$.

### M4. Angular momentum

You can build $J_z$, $J_+$, $J_-$, and $J^2$ as one-body operators.

The one-body matrix elements are fixed by the standard ladder relation

$$
J_+ \lvert j, m \rangle = \sqrt{(j-m)(j+m+1)}\ \lvert j, m+1 \rangle ,
$$

with $J_-$ its conjugate and $J_z \lvert j,m\rangle = m \lvert j,m \rangle$.
This relation also pins the relative phase of the $m$ states inside an orbit.
That phase convention is called Condon-Shortley, and every table of matrix
elements you use in Stage 2 assumes it. So M4 locks the phases as well as
testing the algebra.

*Convincing:* $[J_+, J_-] = 2 J_z$ and $[J^2, J_z] = 0$ to machine precision.
Small sectors show the multiplet structure you expect from the dimensions.

### M5. The pairing capstone

The **pairing** operator creates a particle in slot $(j,m)$ and one in slot
$(j,-m)$ at the same time:

$$
P^\dagger = \sum_j \sum_{m>0} (-1)^{j-m}\, a^\dagger_{jm} a^\dagger_{j,-m} ,
\qquad
H_{\mathrm{pair}} = -G\, P^\dagger P .
$$

The sum over $m > 0$ is the tail slice of section 4.2, and the phase uses the
integer form of section 4.1. $P$ is the conjugate of $P^\dagger$, because
$(a^\dagger_{jm} a^\dagger_{j,-m})^\dagger = a_{j,-m} a_{jm}$.

Build this in a single orbit and reproduce the known ground-state formula

$$
E = -\frac{G}{4}\,(N - v)\,(2\Omega - N - v + 2),
\qquad \Omega = j + \tfrac{1}{2},
$$

with $v = 0$ for even particle number $N$ and $v = 1$ for odd $N$.

*Convincing:* agreement to $10^{-10}$ for at least three values of $j$. Make
sure also that the minimum sits at half filling, and not at the full orbit.
That surprises people, and it is correct.

M5 is the real test of Stage 1. It exercises the whole chain at once: states,
signs, assembler, and eigenvalue solver. It needs no recoupling and no tables.
If M5 passes, your instrument works.

### A warning about M5

The $G/4$ in the formula belongs to the $P^\dagger$ written above, with no
$1/2$ and no $1/\sqrt{2}$ in front. If you take $P^\dagger$ from a textbook
with a different normalization, the coefficient changes. Do not mix the two.
Whichever you choose, write it down.

## 9. Tests that lock the conventions

Write these before the physics code. They are cheap, and they catch the errors
that are expensive to find later. All of them are covered by the milestones
above. This is the checklist.

1. **Table fingerprint.** Assert $(n, l, 2j, 2m, 2t_z)$ at indices 0, 5, 11, 12,
   17, 23 against section 4.3. Catches an off-by-one in the offsets.
2. **Ascending $m$.** Inside each orbit, $2m$ increases by 2 per step, and the
   $m > 0$ states are exactly the tail slice.
3. **Antisymmetry.** $a^\dagger_p a^\dagger_q = -a^\dagger_q a^\dagger_p$ and
   $a^\dagger_p a^\dagger_p = 0$, as operators.
4. **Species symmetry.** The neutron-neutron block equals the proton-proton
   block, for an interaction that treats the species alike.
5. **Hermiticity.** $H = H^{\mathsf{T}}$ for a real Hamiltonian.
6. **Particle number.** The bit-counted $Z$ and $N$ agree with explicit counting.

One command must run all of them.

## 10. Three things you decide, and must record

These are open. Decide each one, pin it with a test, and write the decision in
your notes. Later work reads your notes, not your mind.

1. **The $1/4$.** Make sure that the identity of section 6 holds numerically,
   and state which form your stored $V$ uses. This is the item most likely to
   give a silent factor of 2 or 4. 2. **Restricted sums.** Decide whether any
   code path restricts $p < q$ or $r < s$. If one does, re-derive its
   prefactor and record it next to the code. 3. **The phase of $P^\dagger$.**
   Make sure on a single-orbit toy that the $(-1)^{j-m}$ pairs $(m, -m)$ as
   written above, and that the sign is not absorbed somewhere else. M5 will
   tell you: a wrong phase does not reproduce the formula.

## 11. The file format

At the end of Stage 1, write down a format for storing one Hamiltonian. Call
it a **bundle**. Give it a version number.

A bundle holds the sector it belongs to, the slot ordering or a hash of it, the
arrays $t$ and $V$, the units, and the parameters that produced it.

This is worth an afternoon for three reasons. Stage 3 compares many Hamiltonians
against each other, and it can only do that if they are stored the same way. The
optional coda reads your files without reading your code. And a paper needs
results that someone else can reproduce from a stored input.

Freeze the slot ordering when you freeze the format. Some quantities that you
measure later depend on the ordering, so a change invalidates earlier numbers.

## 12. Measure cost from week one

Each time you build a matrix, record the sector, the size of the space, the
number of non-zero entries, the time to assemble, and the time to diagonalize.
Append a row to a file. It costs three lines of code.

By Stage 3 you will have a growth curve that you did not have to go back and
measure. Cost is one of the axes the project is about.

## 13. What you choose, and what is fixed

You choose the language, the code layout, the storage library, and the test
framework. The repository is set up for Python with `uv`. That is a default,
not a requirement. Reorganize it if you prefer something else.

Three things are fixed, because Stage 3 and the coda depend on them.

1. You implement and test the conventions in this document.
2. The bundle format is documented, versioned, and readable without your code.
3. The setup is reproducible. Pin the dependencies. Record the test command.

## 14. Not in this stage

The quadrupole operator and radial integrals. The Wigner-Eckart theorem and
reduced matrix elements. SU(3). Transition strengths. Rank truncation. Random
interaction ensembles. Anything with qubits.

All of these come later, and Stage 2 hands you the numbers for most of them.

## 15. A rhythm, not a schedule

| Weeks | Roughly |
|---|---|
| 1 to 2 | Read sections 2 to 5. Choose your stack. M1. Get one test command working. |
| 3 to 4 | M2, on the 6-slot space. The hand-built comparison. |
| 5 to 7 | M3. Sparse against dense. Settle the $1/4$. |
| 8 | M4. |
| 9 | M5. The capstone. |
| 10 | The bundle format. A short writeup: the conventions, the test list, one growth curve. |

If you are two weeks behind at week 6, drop the 24-slot space from this stage
and do M5 in a single orbit. That is where M5 lives anyway.

## 16. Ways this goes wrong

| Trap | What to do |
|---|---|
| You build a framework instead of a calculation. | Do not add an abstraction until you need it twice. A list of (coefficient, creations, annihilations) terms and a map from mask to index are enough. |
| You test only the cases you thought of. | Use random $t$ and random $V$ as the oracle. Run them often. They catch what you did not think of. |
| You build dense matrices that do not fit. | Stay in small spaces here. Learn the sparse path now, while the dense answer is still available to compare against. |
| Physics leaks into the machinery. | If the assembler knows what angular momentum is, the boundary is wrong. |
| You change a convention to make a test pass. | Find out why the test fails first. These conventions are consistent with each other, and Stage 2 and the coda assume them. |

## 17. Reading

You need one reference in this stage, and only its first few pages.

- P. Van Isacker, *Seniority in quantum many-body systems*, arXiv:1010.2415.
  Background for the formula in M5.

Make sure of any citation before you use it in writing. Reference lists,
including machine-generated ones, contain wrong identifiers often enough to
matter.