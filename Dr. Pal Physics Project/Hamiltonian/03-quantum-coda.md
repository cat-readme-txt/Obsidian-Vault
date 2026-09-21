# Optional coda: the same Hamiltonian on a quantum computer

Read this only if Stage 3 is finished and time remains.

Nothing in Stage 1, Stage 2, or Stage 3 depends on this document. Your project
is complete without it, and so is your paper.

## 1. Why this exists

This project sits next to a separate research effort. That effort asks what it
costs to simulate a nucleus on a future quantum computer. The cost depends on
the *form* in which you write the Hamiltonian, and different forms of the same
physics cost very different amounts.

Your Stage 3 work measured the physics side of that trade: which parts of the
Hamiltonian you can discard and still get the right answer. This coda measures
the other side: what discarding them saves.

You are not expected to finish this. Even reaching section 3 is a useful
result, and it makes a good final section in a report.

## 2. What you need to know first

One mapping and one number.

### The mapping

A quantum computer has qubits, not particles. The Jordan-Wigner transformation
gives you one qubit per slot, in the slot order you fixed in Stage 1. Qubit
$i$ is 1 when slot $i$ is full, and

$$
a^\dagger_p \;\longrightarrow\;
 \frac{X_p - i\,Y_p}{2} \prod_{k<p} Z_k ,
$$

where $X$, $Y$, and $Z$ are the Pauli matrices. The string $\prod_{k<p} Z_k$
counts the parity of the filled slots below $p$. That is exactly the fermion
sign you implemented in `01-tooling.md` section 5.1, because both come from
the same ascending-order definition of a state. As a result, the two
conventions agree by construction. Do not re-derive one from the other, and do
not "fix" a sign here to make a test pass.

### The number

Write the Hamiltonian as a sum of products of Pauli operators, $H =
\sum_\alpha c_\alpha P_\alpha$. The **1-norm** is the sum of the sizes of
those coefficients, leaving out the identity term:

$$
\lambda = \sum_{\alpha \ne I} \lvert c_\alpha \rvert .
$$

Many quantum-simulation algorithms have a cost that grows with $\lambda$, so
$\lambda$ is a proxy for cost. The number of Pauli terms is a second, different
proxy. Report the two separately. They are not interchangeable.

## 3. Steps

### C1. Map the Hamiltonian

Convert one of your Stage 2 bundles to Pauli form. Diagonalize the qubit
Hamiltonian in the right sector and compare the spectrum against your
fermionic result. They must agree. Do not go further until they do.

### C2. Baseline

Record the qubit count, the number of Pauli terms, the number of non-identity
terms, the largest coefficient, and $\lambda$. Do this for the full
Hamiltonian at each $(G, \chi)$ point. These numbers are the baseline that
every later comparison uses.

### C3. Cost against compression

For each truncated Hamiltonian from Stage 3, compute $\lambda$. Plot $\lambda$
against the rank $R$.

Do not assume this curve goes down. Truncation changes every coefficient, so
$\lambda$ can move either way. Plot it and look.

### C4. The joint plot

Put your Stage 3 accuracy on one axis and $\lambda$ on the other. Label each
point with its rank $R$. That single figure is the point of both projects at
once: how much accuracy a given cost buys.

### C5. If you still have time

Constants are free. Replacing the occupation $n_p$ by $(1 - Z_p)/2$ removes a
constant and can cut the one-body part of $\lambda$ substantially, with no
change to any physics. More generally, you can add to the Hamiltonian any
operator that is zero in the sector you care about. Choose that operator to
make $\lambda$ smaller. That is a small optimization problem. It is a good
stopping point, and it is where the neighbouring research project starts.

## 4. Two things that will bite you

### $\lambda$ is not a property of the physics

It depends on the slot ordering, on the choice of mapping, and on the
single-particle basis. Relabel the qubits and $\lambda$ changes, even though
the operator is the same up to a unitary transformation. So every $\lambda$
you report must state the ordering and the mapping used. This is why Stage 1
asked you to freeze the ordering, and why the slot ordering belongs in the
bundle file.

### A trick that is exact in one sector is wrong in another

Any operator you add in C5 is zero only in the target sector. Make sure of
every such step with a fresh eigenvalue calculation in that sector. If that
calculation disagrees, the fault is in the ordering, the sector definition, or
a sign. It is not in the physics.

## 5. Out of scope

Quantum hardware. Variational algorithms. Error correction. Fault-tolerant
resource counts. Realistic interaction files and three-body forces. These
belong to the neighbouring project, and some of them are research problems in
themselves.

## 6. Reading

- A. Perez-Obiol, A. M. Romero, J. Menendez, A. Rios, A. Garcia-Saenz,
  B. Julia-Diaz, *Nuclear shell-model simulation in digital quantum computers*,
  Sci. Rep. **13** (2023), arXiv:2302.03641. The shell model mapped to qubits,
  with the `sd` shell as one of its examples.
- V. von Burg et al., Phys. Rev. Research **3**, 033055 (2021),
  arXiv:2007.14460. Where the particle-number shift of C5 comes from.

Ask your advisor before going further than this. The literature past this point
assumes a lot of background that the rest of this project deliberately avoids.
