# Start here

An undergraduate research project on the structure of nuclear Hamiltonians.

Level: physics undergraduate
Length: one academic year, part time
You need: linear algebra, and some programming in any language
You do not need: quantum mechanics, nuclear physics, or quantum computing

## 1. The project in plain words

A nucleus is a bag of protons and neutrons. Physicists describe it with a
**Hamiltonian**. A Hamiltonian is a matrix. Its eigenvalues are the energies
that the nucleus can have. Its eigenvectors tell you how the particles arrange
themselves.

That matrix is not a random matrix. It is built from a few simple physical
rules, so it has structure. Some of that structure is well known. Some of it is
not measured at all, because nobody needed the number before.

Structure matters for a practical reason. Every method that makes a large
calculation affordable works by exploiting structure. If you replace the
Hamiltonian with a simpler object that keeps the structure, you get almost the
same physics for much less work. If you throw away the wrong part, you get a
number that looks fine and is wrong.

This project builds a small nuclear Hamiltonian from scratch, measures its
structure, and then asks which parts of it you are allowed to throw away.

## 2. The question

> A nuclear Hamiltonian is a large matrix built from a few simple rules.
> What structure does that matrix have, and which parts of it can you discard
> without changing the physics you care about?

The question splits into two halves that you can answer in order.

1. **Measure the structure.** How much of the Hamiltonian is redundant? You can
   put numbers on this. The rank of its two-body part, the spread of its matrix
   elements, and its channel content are all measurable.
2. **Test what survives.** Discard part of it. Then compare the energies, the
   level spacings, and the transition strengths against the exact answer that
   you computed first.

The second half has no known answer in advance. That is the research.

There is a specific reason to expect something interesting. In quantum
chemistry, the same matrix compresses very well, and a whole family of methods
depends on that fact. Recent work on nuclear interactions reports the opposite:
nuclear Hamiltonians keep many singular values, and some are close to full rank.
If that is true for a small controlled nuclear model as well, it is worth
saying so with data. If it is not true, that is worth saying too.

## 3. Why the project fits an undergraduate

Three reasons.

### The tools are elementary

A many-particle state is a pattern of occupied slots. You store it as an
integer, and you use bit operations on it. Building the Hamiltonian is
bookkeeping plus a sign rule. You need linear algebra. You do not need a
quantum mechanics course. Every formula that comes from quantum mechanics is
written out for you in these documents, with its source.

### The answers are checkable

The first physics target has an exact algebraic formula. Your code either
reproduces it to ten decimal places, or it does not. You never have to wonder
whether your result is right.

### The interesting part is early

You can measure the structure of the matrix in Stage 2, about halfway through.
You do not need a year of background first. The first half of the year gives
you the instrument, and the second half gives you the question.

## 4. The shape of the year

| Stage | What you do | About |
|---|---|---|
| 1 | Build the tooling. Reproduce an exact formula with it. | 10 weeks |
| 2 | Build the model Hamiltonian. Anchor it against known results. | 8 weeks |
| 3 | Measure the structure. Test what survives compression. | 12 weeks |
| Coda | Optional. Convert the Hamiltonian to qubits and count the cost. | if time remains |

Stage 1 has a fixed target, so you always know whether you are on track.
Stage 3 has no fixed target. By then you will have the instrument and the
judgement to pick your own direction inside it.

The coda exists because this project sits next to a separate research effort on
quantum simulation of nuclei. The coda is a bridge to that work. It is not
part of your project, and nothing in your project depends on it.

## 5. The plan is a map, not a checklist

These documents give you a starting direction, the formulas that are otherwise
hard to find, and the traps that cost other people weeks. They do not give you
a task list to work through.

Three rules follow from that.

- **Stage 3 is a menu, not a queue.** It lists more directions than you have
  time for. Pick the ones that look interesting after you see Stage 2 data.
- **A surprise beats the plan.** If you measure something odd, chase it. Tell
  your advisor what you found and what you want to drop in exchange.
- **Write your guess down first.** Before each measurement, record what you
  expect. Then a result that contradicts you is a finding and not a bug hunt.

Two things are not negotiable, because later work depends on them: the sign and
labeling conventions, and a documented file format for your results. Stage 1
covers both.

## 6. Where a paper can come from

You have two plausible papers. You do not have to choose now. The data tells
you which one you have, some time in Stage 3.

### A benchmark note

You measured the structure of a controlled nuclear Hamiltonian, and you showed
which observables survive which approximations. The result that carries such a
note is a mismatch. The approximation that keeps the ground-state energy to a
few keV wrecks the transition strength. The reverse is just as interesting.
This is a short paper. A brief report or an arXiv preprint with a workshop
poster is a realistic target.

### A software paper

You built an open, tested, documented testbed for small shell-model problems,
with a frozen exchange format that other people can use. The physics results
are the demonstration, and the tool is the contribution. This is a realistic
target if the code turns out to be cleaner and more reusable than the physics
is surprising.

Both need the same three things, so work towards them from week one.

- Results you can reproduce from a stored input file.
- An honest error bar or tolerance on every number.
- A recorded expectation for each measurement, including the ones you got wrong.

## 7. What you will be able to do afterwards

- Represent a many-particle system and build operators for it from scratch.
- Test numerical code against cases you can compute by hand.
- Design a data format that another person can use without reading your code.
- Measure the cost of a computation, not only its result.
- Study an approximation, and report its error honestly.

These skills transfer. They are the same skills that condensed-matter, quantum
chemistry, and quantum computing groups need.

## 8. How to start this week

Do not read everything first. Do this instead.

1. Write a program that lists every way to put 2 particles into 6 slots. Print
   the 15 patterns as bit strings. Make sure that the count is 15. 2. Read
   sections 2 to 5 of `01-tooling.md`. Stop there. That document holds every
   convention and formula the first stage needs, and it tells you when you
   need each one. 3. Start on milestone M1.

Step 1 takes an hour. It is also the real first step of the project.

## 9. Two honest notes

This is a benchmark study, not a new method. You are measuring how known
techniques behave on a nuclear Hamiltonian that nobody measured them on. That
is a modest and publishable kind of result. It is not a breakthrough, and the
plan does not pretend otherwise.

The year is tight. If Stage 2 runs long, finish Stage 3 on a smaller model
rather than rushing Stage 2. A clean result on a 6-slot model beats a doubtful
result on a 24-slot model.

## Documents

| File | What it is |
|---|---|
| `01-tooling.md` | Stage 1. Build and test the instrument. |
| `02-structure.md` | Stage 2 and Stage 3. The model, and the research question. |
| `03-quantum-coda.md` | Optional. The bridge to the quantum-simulation project. |

The three documents are self-contained. Every convention, formula, and number
you need is in them, at the point where you need it.
