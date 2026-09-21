# Stage 2 and Stage 3: the model and its structure

About 20 weeks. December to May. Start when the Stage 1 milestones pass.

Stage 2 points your instrument at a system whose answer is known. Stage 2 has a
right answer, and you will either get it or not. Stage 3 asks a question that
has no published answer for this system. The two stages share one document,
because the boundary between them is not sharp. If Stage 2 runs long, take the
smaller version of Stage 3 described in section 9.

This document assumes the conventions of `01-tooling.md`, and adds the few that
the interaction and the observables need. It adds no others.

---

# Part A. Stage 2: a model with known answers

## 1. The Hamiltonian

Build the pairing-plus-quadrupole Hamiltonian in the 24-slot space of
`01-tooling.md` section 4.

$$
H(G,\chi) = \sum_p \varepsilon_p\, n_p
 - G\, P^\dagger P
 - \frac{\chi}{2} \sum_{\mu=-2}^{2} (-1)^\mu\, Q_{2\mu} Q_{2,-\mu}
$$

Three terms, three physical ideas.

- $\sum_p \varepsilon_p n_p$ gives each slot its own energy. It is the cost of
  putting a particle in that slot. Here $n_p = a^\dagger_p a_p$ counts the
  particles in slot $p$.
- $-G P^\dagger P$ is **pairing**. It rewards particles that sit in
  time-reversed pairs. You already have $P^\dagger$ from Stage 1 milestone M5.
- The last term is the **quadrupole** interaction. It rewards particles that
  arrange themselves into a deformed, cigar-like shape.

Two numbers control the competition, $G$ and $\chi$. Pairing wins at large $G$
and gives a spherical nucleus with a gap. The quadrupole wins at large $\chi$
and gives a rotating, deformed nucleus. Real nuclei live in between. That is the
reason to use this model and not a random one: it has a knob that moves between
two physically distinct regimes, and both limits are understood.

## 2. The quadrupole operator, and its conventions

The quadrupole operator is a one-body operator built from a table of numbers:

$$
Q_{2\mu} = \sum_{pq} \langle p \rvert r^2 Y_{2\mu} \lvert q \rangle\,
           a^\dagger_p a_q .
$$

Those numbers are nuclear-structure input. Section 3 says where to get them. You
do not derive them.

Three conventions come with it.

### The scalar product

$Q_{2\mu}$ is a rank-2 spherical tensor, which is a set of five components
labeled $\mu = -2 \ldots 2$ that transform into each other under rotation. The
factor $(-1)^\mu$ in the Hamiltonian makes the sum $\sum_\mu (-1)^\mu Q_{2\mu}
Q_{2,-\mu}$ the standard scalar product of that tensor with itself. A scalar
product is rotationally invariant, which is what makes the Hamiltonian's
energies independent of orientation.

### Isoscalar coupling

Build $Q$ from the proton and the neutron slots with the same $\chi$, unless
you decide otherwise and say so. This is a model choice, not a convention, and
it is a reasonable thing to vary later. Record which choice you used, because
it changes the physics.

### Phases

The matrix elements $\langle p \rvert r^2 Y_{2\mu} \lvert q \rangle$ that you
read from a table assume Condon-Shortley phases. Milestone M4 of Stage 1
locked exactly those phases. So the table and your basis agree, as long as you
did not change the ladder-relation convention.

## 3. The numbers you are given

Write each of these into your code with its source next to it. Do not copy a
number out of someone's code without the source.

| Input | Value | Source |
|---|---|---|
| Single-particle energies $\varepsilon_p$ | $d_{5/2} = -3.9257$, $s_{1/2} = -3.2079$, $d_{3/2} = +2.1117$ MeV (USDB) | Brown and Richter, Phys. Rev. C **74**, 034315 (2006) |
| Degenerate limit | all $\varepsilon_p$ equal | the control case, needed for the SU(3) anchor |
| Reduced matrix elements of $r^2 Y_2$ | two independent `sd`-shell integrals, $d$-to-$d$ and $s$-to-$d$ | tabulated in Brussaard and Glaudemans, *Shell-Model Applications in Nuclear Spectroscopy*, or Suhonen, *From Nucleons to Nucleus* |
| Oscillator length $b$ | $\hbar\omega = 45 A^{-1/3} - 25 A^{-2/3}$ MeV, so $b^2 = 41.47/(\hbar\omega)$ fm². At $A = 20$ this gives $b^2 = 3.14$ fm², $b = 1.77$ fm | Blomqvist and Molinari, Nucl. Phys. A **106**, 545 (1968) |
| Effective charges | $e_p = 1.35\,e$, $e_n = 0.35\,e$ | Richter and Brown, Phys. Rev. C **67**, 034317 (2003) |
| `20Ne` level energies | $2^+$ 1633.674(15) keV, $4^+$ 4247.7(11) keV, $6^+$ 8777.6 keV | ENSDF adopted values |
| `20Ne` transition strength | $B(E2; 0^+ \to 2^+) = 340(30)\ e^2\,\mathrm{fm}^4$ | Pritychenko et al., At. Data Nucl. Data Tables **107**, 1 (2016) |
| $G$ and $\chi$ | choose a handful of representative points | see section 8.4 |

Make sure of every citation yourself before it goes into your writeup.

## 4. Observables, and the traps in them

### 4.1 The two quadrupole operators are different objects

The $Q_{2\mu}$ inside the Hamiltonian is part of the interaction. The
quadrupole operator used for transition strengths carries the effective
charges $e_p$ and $e_n$ and is an observable. They are built from the same
matrix elements, and they are not the same operator. Give them different names
in the code and in the report. People lose weeks here.

### 4.2 A transition strength has no meaning without its conventions

Report $b$, the effective charges, and the direction with every $B(E2)$
number. The definition is

$$
B(E2; J_i \to J_f)
 = \frac{\lvert \langle J_f \Vert E2 \Vert J_i \rangle \rvert^2}{2J_i + 1},
\qquad\text{so}\qquad
B(E2; 0^+ \!\to 2^+) = 5\, B(E2; 2^+ \!\to 0^+).
$$

That factor of 5 is the single most common error in this subject.

### 4.3 Compare quantities that do not depend on a phase

Energies, transition strengths, and the absolute value of an overlap are
physical. The components of an eigenvector are not: an eigenvector carries an
arbitrary overall sign or phase, and a degenerate eigenvalue gives you an
arbitrary basis inside its subspace.

So compare energies, $B(E2)$ values, and $\lvert\langle \psi_1 \vert \psi_2
\rangle\rvert$. Never compare eigenvector components one by one, in a test or in
a plot. Tests written that way fail at random and teach you nothing.

### 4.4 Normal ordering: do not drop the induced one-body term

$Q \cdot Q$ is a product of two one-body operators. When you write it in the
standard form of `01-tooling.md` Fact 5, it does **not** become a purely
two-body object. It gives a two-body part, a one-body part that shifts
$\varepsilon_p$, and a constant.

If you extract "the two-body part of $Q \cdot Q$" without care, code will
silently drop the one-body piece. Every energy afterwards is then wrong by a
smooth amount that looks plausible.

Test it. The extracted two-body operator must give zero on the empty state and
on every one-particle state. That test is a few lines, and it makes the mistake
impossible to miss.

## 5. Three anchors

An anchor is a comparison against a result you did not produce. Do them in
this order. They get more expensive and more interesting as you go.

### 5.1 Pairing (free)

Set $\chi = 0$, make the $\varepsilon_p$ degenerate, and use a single orbit.
You must reproduce the seniority formula from Stage 1 M5. This costs you
nothing, because the code already exists. Run it first. If anything breaks
later, run it again.

### 5.2 The SU(3) limit

Set $G = 0$ and make the $\varepsilon_p$ degenerate. Now the quadrupole
interaction alone generates a rotational band: a ladder of levels spaced like
$L(L+1)$.

There is a subtlety here that will cost you a week if nobody tells you. The
physical operator $r^2 Y_2$ does **not** generate the symmetry exactly. The
operator that does is the symmetrized, or algebraic, one:

$$
\mathcal{Q}_{2\mu} = \sum_i
 \bigl( r^2 Y_{2\mu} + b^4 p^2 Y_{2\mu}(l_i) \bigr) / b^2 .
$$

So run two versions.

#### A. The exact case

Use the algebraic operator $\mathcal{Q}$. You get a clean $L(L+1)$ band, the
$(\lambda,\mu) = (8,0)$ band of `20Ne`. You can also make sure that the
algebraic identity holds,

$$
\mathcal{Q}\cdot\mathcal{Q} = 4\,C_2(\lambda,\mu) - 3 L^2,
\qquad
C_2(\lambda,\mu) = \lambda^2 + \mu^2 + \lambda\mu + 3(\lambda + \mu).
$$

This is a sharp pass-or-fail test.

#### B. The broken case

Use the physical operator $r^2 Y_2$. You get a band that is only approximately
rotational. Measure how far it departs. Version B is the more useful result,
and it is the one that connects to real nuclei. The departure is physics, not
a bug.

### 5.3 `20Ne`

One comparison with experiment. Use the USDB energies and both interactions
switched on.

Read this before you compute, or a correct result will look like a failure.
`20Ne` is not a rigid rotor.

| Ratio | Measured | Rigid rotor |
|---|---|---|
| $E(4^+)/E(2^+)$ | 2.60 | 3.33 |
| $E(6^+)/E(2^+)$ | 5.37 | 7.00 |

Expect your numbers near the measured column, not near the rigid-rotor column.

If this goes smoothly and you want a second nucleus, `24Mg` is the natural one.
It is optional. Two anchors plus one experimental comparison is already enough
to trust the model.

## 6. The observable ladder

Stage 3 measures how observables respond to approximations. You need more than
one observable, because different observables probe different parts of the
state. Build them in this order, cheapest first.

| Observable | What it probes | Cost to you |
|---|---|---|
| Ground-state energy $E_0$ | the lowest state only | one eigenvalue |
| Excitation energy $E(2^+)$ | the gap, so two states | two eigenvalues |
| Occupation numbers $\langle n_p \rangle$ | how particles spread over slots | one expectation value |
| Overlap $\lvert\langle \psi_1 \vert \psi_2 \rangle\rvert$ of two ground states | whether the state itself moved | one inner product |
| $B(E2; 0^+ \to 2^+)$ | off-diagonal structure, shape | the transition operator |

The last row is the most informative and the most work, because it needs the
transition operator with effective charges. The first four need only what you
already have.

This matters for planning. If the transition operator becomes a time sink, you
can still do a complete Stage 3 study with the first four rows. Do not let one
observable block the project.

## 7. Stage 2 is done when

- The pairing anchor and the SU(3) anchor both pass.
- The `20Ne` comparison is recorded, with the ratio caution stated.
- The normal-ordering test of section 4.4 passes.
- Every row of the table in section 3 is written down with its source.
- The exact observables, at your chosen $(G, \chi)$ points, are stored in bundle
  files. **This is the reference dataset.** Stage 3 measures every error against
  it. Freeze it.

---

# Part B. Stage 3: the structure question

## 8. What you are actually asking

> How much of this Hamiltonian is redundant, and does the redundant part matter?

The Hamiltonian is built from a few physical rules, so you expect it to be
simpler than a generic matrix of the same size. "Simpler" can mean several
different things, and they do not have to agree. Your job is to measure some of
them and see whether they do.

### 8.1 The menu

These are directions, not a queue. Each one is a few days of work once the
instrument exists. Pick two or three. Pick them after you have seen your Stage
2 data, not now.

#### Rank of the two-body part

Arrange $V_{pqrs}$ as a matrix $W$ with row index $(pq)$ and column index
$(rs)$. Take its eigenvalue decomposition. How many modes carry the weight?
Plot the spectrum of $W$. This is the most direct measure of redundancy.

#### Two notions of rank, which are not the same

This is the single most confusing point in this literature, and clearing it up
is a real contribution.

| Notion | Meaning |
|---|---|
| Eigenvalue rank | the number of modes kept in the eigenvalue decomposition of $W$ |
| Sum-of-squares rank | the number of squared one-body operators needed to write the interaction |

Demonstrate the difference numerically. $P^\dagger P$ has eigenvalue rank 1. The
quadrupole term is a sum of five squares by construction, so its sum-of-squares
rank is at most 5, and yet its $W$ is generically full rank. Two different
numbers, one operator. Report both, and say which one you mean every time.

#### Sparsity and the spread of matrix elements

How many entries of $W$ are non-zero? What does the distribution of their
sizes look like? Does it change between the pairing regime and the quadrupole
regime?

#### Monopole content

The monopole part of an interaction is the angle-averaged piece that mostly
moves the single-particle energies around:

$$
V^{\mathrm{mono}}_{ab}
 = \frac{\sum_J (2J+1)\,\langle ab; J \rvert V \lvert ab; J \rangle}
        {\sum_J (2J+1)} .
$$

How much of the interaction is monopole? Does the monopole part behave
differently under compression from the rest?

#### Growth

How do the rank, the sparsity, and the assembly cost change as the space
grows? You recorded these numbers from Stage 1 week 1. The curve costs you
nothing extra.

### 8.2 Compression, and what survives it

This is the part with no known answer.

1. Keep the top $R$ modes of $W$. Throw the rest away. Rebuild the Hamiltonian.
2. Recompute the observables from the ladder in section 6.
3. Compare against the frozen Stage 2 reference. Plot the error against $R$.
4. Repeat in each regime.

The interesting question is not whether the error goes down as $R$ goes up. It
does. The interesting question is **whether the different observables need the
same $R$**.

There is a reason to expect that they do not. The energy is a variational
quantity, and it is insensitive to small changes in the state. A transition
strength is not. So an approximation can hold the ground-state energy to a few
keV while the transition strength moves by twenty per cent. If you measure that
and quantify it, you have the core of a paper.

A result in which everything converges together is also a result. Report it as
one.

### 8.3 Is this model special?

The model has strong structure by construction. Pairing is rank 1. That raises
an honest objection: maybe your conclusions are properties of this model and
not of nuclear Hamiltonians in general.

The clean way to address it is to interpolate towards a generic interaction:

$$
H(\eta) = (1-\eta)\, H_{\mathrm{model}} + \eta\, H_{\mathrm{ensemble}} ,
$$

where $H_{\mathrm{ensemble}}$ is a random two-body interaction. Then report how
your structure measures move as $\eta$ goes from 0 to 1. A continuous family
tells you much more than two isolated end points, and costs little extra.

One constraint matters, and it is easy to get wrong. Draw the random
interaction so that it keeps the symmetries: random values for the coupled
matrix elements $V^{JT}$, not independent random numbers for every matrix
element in your slot basis. This is also the first place in the project where
the sign of $t_z$ stops being a free label. Make sure of the isospin
convention of any coupled elements that you use, against `01-tooling.md`
section 4.1. Without rotational symmetry, $E(2^+)$ and $B(E2)$ are not even
defined, so there is nothing to compare. This is cheap to get right at the
start and expensive to discover at the end.

"Generic" here means typical under an ensemble. It does not mean realistic. Say
so in the writeup.

### 8.4 Where in the phase diagram

Do everything at a small number of $(G, \chi)$ points, chosen to sit in three
places: pairing-dominated, quadrupole-dominated, and the mixed region between
them. Three or four points is enough. A dense scan of the plane is a lot of
compute and very little extra information.

Pick the points by looking at an observable that separates the regimes, for
example $E(4^+)/E(2^+)$, or the ratio of the pairing and quadrupole expectation
values in the ground state.

## 9. If time gets short

Drop the 24-slot space before you drop the science. A structure study in a
12-slot space, done properly with anchors and error bars, is a real result.
The same study in a 24-slot space, rushed and unanchored, is not.

Drop $B(E2)$ before you drop having several observables. The occupation numbers
and the ground-state overlap already show whether the state itself moved.

## 10. Write your guesses down first

Before you run each measurement, record what you expect. Date it. Then a
contradiction is a finding and not a panic.

Three guesses worth recording at the start of Stage 3.

- **H1.** $B(E2)$ needs a higher rank than $E_0$ does.
- **H2.** The rank you need depends on the regime.
- **H3.** The structured model is not typical of the random ensemble.

Report the outcome of each one, including the ones you got wrong. A recorded
wrong guess is one of the more convincing things a report can contain.

## 11. What a skeptical reader will ask

Keep this list next to you while you work. If your notes answer all six, you
have a paper.

1. How do I know your code is right? (the anchors in section 5) 2. Compared to
   what exact answer? (the frozen reference dataset) 3. How big is the error,
   and how big is your numerical noise? 4. Does the result depend on the
   model, or does any nuclear Hamiltonian do it? (section 8.3) 5. Does it
   depend on a convention you chose? (which rank, the $B(E2)$ direction, the
   $P^\dagger$ normalization, the slot ordering) 6. Can I reproduce it?
   (bundle files plus one command)

## 12. Deciding which paper you have

Some time in Stage 3, around March, look at your data and choose.

### A benchmark note

You have a benchmark note if the observables separate. You found a compression
that keeps one observable and breaks another, and you can show the crossover
and say where it comes from. Write the physics paper. The comparison between
the two notions of rank belongs in it, because it is a point the literature
keeps blurring.

### A software paper

You have a software paper if the physics result turns out to be "everything
converges together", but your testbed is clean, tested, documented, and
reusable. Then the contribution is the instrument, and the structure study is
the worked example that demonstrates it. The null physics result still belongs
in the paper, stated plainly.

### Neither, this year

You have a poster and a good year if neither is ready. That is a normal
outcome for a first project, and the tooling still stands. Say what you
measured, and say what is still open.

Talk to your advisor at this decision point. Do not decide it alone.

## 13. Freeze this for the writeup

- The bundle files for the full Hamiltonian at every $(G, \chi)$ point.
- The bundle files for every truncated Hamiltonian.
- The exact reference observables.
- The error tables behind every plot.
- The conventions: which rank, the $B(E2)$ direction, the $P^\dagger$
  normalization, the slot ordering, and the units.

The optional coda in `03-quantum-coda.md` reads exactly these files.

## 14. Reading

Read these when you reach the section that needs them. Not before.

For the model and the SU(3) anchor:

- J. P. Elliott, Proc. R. Soc. London A **245**, 128 and 562 (1958).
- J. Escher, C. Bahri, D. Troltenier, J. P. Draayer, Nucl. Phys. A **633**, 662
  (1998). Gives the algebraic quadrupole operator used in section 5.2.
- A. Bohr, B. R. Mottelson, D. Pines, Phys. Rev. **110**, 936 (1958), for where
  pairing-plus-quadrupole comes from.

For the structure question:

- A. Tichai, P. Arthuis, K. Hebeler, M. Heinz, J. Hoppe, A. Schwenk,
  Phys. Lett. B **821**, 136623 (2021). Low-rank decompositions of nuclear
  interactions. This is the closest published work to your question.
- M. Motta et al., npj Quantum Inf. **7**, 83 (2021). The sum-of-squares notion
  of rank, in its original chemistry setting.
- E. Caurier, G. Martinez-Pinedo, F. Nowacki, A. Poves, A. P. Zuker,
  Rev. Mod. Phys. **77**, 427 (2005), for the monopole interaction.
- J. B. French, S. S. M. Wong, Phys. Lett. B **33**, 449 (1970), for random
  two-body ensembles.

Make sure of every identifier before you cite it.
