+++
title = "Entanglement: The Experiment That Proved Einstein Wrong"
date = 2026-11-08
description = "Quantum entanglement and Bell's theorem explained: the EPR paradox, local hidden variables, the CHSH inequality simulated in Python, the experiments that won the 2022 Nobel Prize, and why it can't send signals faster than light."
draft = false

[extra]
seo_title = "Entanglement: The Experiment That Proved Einstein Wrong"
toc = true
+++

## Introduction

Einstein never accepted quantum mechanics as the final word. He was happy with its predictions; what
he refused was the idea that a particle doesn't have definite properties until it's measured, the idea
at the heart of the [uncertainty principle](@/blog/the-uncertainty-principle-is-not-about-clumsy-measurements/index.md). In 1935,
with Boris Podolsky and Nathan Rosen, he wrote a paper arguing that quantum mechanics must be
**incomplete**: there had to be hidden details it was leaving out.

For about thirty years this looked like a philosophical argument that could never be settled. Then in
1964, John Bell found a way to turn it into an **experiment**. The experiment has since been done,
over and over, with steadily fewer loopholes. Einstein's picture lost. The 2022 Nobel Prize in Physics
went to three of the people who did those experiments.

What I love about Bell's theorem is that the core of it is just counting. You can check it with a
simulation and a bit of arithmetic, which is exactly what I'll do in this post.

## TL;DR

- Two **entangled** particles have perfectly linked measurement results, however far apart they are.
- Einstein argued the results must be **pre-set** at the source, like a pair of gloves split into two
  boxes. That's a **local hidden variable** theory.
- Bell showed that **any** such theory obeys a limit on how strongly the results can be correlated.
  In the CHSH version, a number called `S` can be at most **2**.
- Quantum mechanics predicts `S` up to **2√2 ≈ 2.83**, and experiments agree. So the results can't
  have been pre-set by anything local.
- It still **can't be used to send messages faster than light**.

## Entanglement in one paragraph

Two particles can be prepared in a joint quantum state where neither has a definite value on its own,
but their values are linked. A standard example is a pair of electrons in the **singlet** state: if you
measure the spin of both along the same axis, you always get opposite answers. One up, one down. Which
one is up is completely random, 50/50, but they're always opposite. This holds whether the electrons
are a millimetre apart or on different continents.

## The EPR argument: gloves in boxes

At first glance, perfect correlation isn't spooky at all. Take a pair of gloves, put one in each of two
boxes without looking, and post one to Lagos and one to Tokyo. Open the box in Lagos, find a left glove,
and you instantly know Tokyo has the right one. Nothing travelled between the boxes. The answer was
decided when the gloves were packed.

Einstein, Podolsky and Rosen argued that entangled particles must work like this. If you can predict
the result at Tokyo with certainty, without touching it, then that result must correspond to some
**element of reality** that was there all along. Quantum mechanics doesn't include these pre-set
values in its description, so it must be incomplete.

This view has two ingredients:

1. **Realism**: the measurement results are determined in advance by properties the particles carry.
   Physicists call these **hidden variables**.
2. **Locality**: what happens to one particle can't instantly affect the other. Nothing travels faster
   than light.

Together they make a **local hidden variable theory**. It sounds like common sense. Bell showed it makes
a testable prediction, and that prediction is wrong.

## Where the gloves analogy breaks: choosing the angle

The twist is that spin can be measured along **any direction**, and the measurer chooses.

Measure both electrons along the same axis and the results are always opposite. Measure them along
axes at 90° and the results are completely uncorrelated. In between, quantum mechanics predicts the
correlation follows a smooth cosine:

```
E(a, b) = −cos(a − b)
```

where `E` is the average of the product of the two results (+1 or −1 each), and `a`, `b` are the
two measurement angles. `E = −1` means always opposite, `0` means no relation, `+1` means always the same.

A local hidden variable model can try to reproduce this. Each pair could carry a little instruction
sheet saying what to answer for every possible angle. The question Bell asked is whether *any*
instruction sheet can match that cosine. The answer is no.

## The CHSH inequality

The most practical version of Bell's result came in 1969 from John Clauser, Michael Horne, Abner
Shimony and Richard Holt, and it's known by their initials, **CHSH**.

Alice and Bob each have two measurement settings: Alice chooses `a` or `a′`, Bob chooses `b` or `b′`.
They measure lots of pairs, choosing settings at random, and compute:

```
S = E(a, b) − E(a, b′) + E(a′, b) + E(a′, b′)
```

Here's why local hidden variables can't push `|S|` past 2. Suppose each pair carries pre-set answers
`A, A′` for Alice and `B, B′` for Bob, all ±1. For one pair:

```
A·B − A·B′ + A′·B + A′·B′  =  A·(B − B′) + A′·(B + B′)
```

`B` and `B′` are each ±1, so either they're equal, and `B − B′ = 0` while `B + B′ = ±2`, or they're
opposite, and it's the other way round. Either way, one bracket is 0 and the other is ±2, so the whole
thing is exactly **±2**. Average over any mix of pairs and you can't get beyond 2.

That's the whole proof. No quantum mechanics involved, only the assumption that the answers existed
before the measurement and that Alice's choice can't change Bob's answer.

Quantum mechanics, with angles `a = 0°`, `a′ = 90°`, `b = 45°`, `b′ = 135°`, predicts:

```
|S| = 4 × cos(45°) = 2√2 ≈ 2.83
```

It breaks the limit. And 2√2 is the most any quantum system can reach, a bound found by Boris Tsirelson
in 1980.

## Simulating it in Python

Let's check both claims. The first source is a reasonable local hidden variable model: each pair leaves
the source carrying a hidden angle, and each detector answers based only on its own setting and that
angle. The second samples outcomes from quantum mechanics' predicted probabilities.

```python
import math
import random

def local_pair(a: float, b: float) -> tuple[int, int]:
    """Local hidden variables: each pair leaves the source carrying a hidden angle."""
    hidden = random.uniform(0, 2 * math.pi)
    alice = 1 if math.cos(a - hidden) >= 0 else -1
    bob = -1 if math.cos(b - hidden) >= 0 else 1
    return alice, bob

def quantum_pair(a: float, b: float) -> tuple[int, int]:
    """Spin singlet: outcomes sampled from quantum mechanics' predicted probabilities."""
    alice = random.choice((1, -1))
    opposite = random.random() < math.cos((a - b) / 2) ** 2
    bob = -alice if opposite else alice
    return alice, bob

def correlation(source, a: float, b: float, trials: int = 200_000) -> float:
    total = 0
    for _ in range(trials):
        alice, bob = source(a, b)
        total += alice * bob
    return total / trials

def chsh(source) -> float:
    a, a2 = 0.0, math.radians(90)
    b, b2 = math.radians(45), math.radians(135)
    return (correlation(source, a, b) - correlation(source, a, b2)
            + correlation(source, a2, b) + correlation(source, a2, b2))

random.seed(7)
print(f"Local hidden variables: |S| = {abs(chsh(local_pair)):.3f}")
print(f"Quantum mechanics:      |S| = {abs(chsh(quantum_pair)):.3f}")
print(f"Bell's limit: 2, quantum maximum: 2√2 = {2 * math.sqrt(2):.3f}")
```

```
Local hidden variables: |S| = 1.997
Quantum mechanics:      |S| = 2.826
Bell's limit: 2, quantum maximum: 2√2 = 2.828
```

The hidden-angle model gets perfect anti-correlation at equal angles, just like the real thing. It
even sits right *at* the limit of 2. But no matter how you tweak it, you can't get it past 2. You're
welcome to try: change `local_pair` however you like, as long as Alice's answer only uses `a` and
`hidden`, and Bob's only uses `b` and `hidden`.

Notice the cheat in `quantum_pair`: Bob's result is computed using **both** angles, `a − b`. That's
the only way a classical program can reproduce quantum statistics. To fake entanglement classically,
one side has to know the other side's setting.

## The experiments

The prediction is clear. Testing it took decades of increasingly clever experiments, usually with
pairs of polarisation-entangled photons rather than electrons:

- **1972**: John Clauser and Stuart Freedman at Berkeley ran the first test and found a violation of
  Bell's inequality, in agreement with quantum mechanics.
- **1982**: Alain Aspect's team in Paris switched the measurement settings **while the photons were in
  flight**, so no signal at light speed could tell one detector what the other had chosen.
- **1998 onwards**: Anton Zeilinger's group in Innsbruck and Vienna used random number generators and
  longer distances to close more gaps, and later used entanglement for quantum teleportation.

Sceptics pointed to **loopholes**. If detectors miss many photons, maybe the ones they catch are a
biased sample (the detection loophole). If the detectors are close enough, maybe a signal could pass
between them (the locality loophole). In **2015**, three groups, at Delft, Vienna and NIST in
Colorado, closed both loopholes in the same experiments. The Delft team used electrons in diamonds
1.3 km apart. The violation held.

In 2022, Aspect, Clauser and Zeilinger shared the Nobel Prize in Physics "for experiments with
entangled photons, establishing the violation of Bell inequalities and pioneering quantum information
science."

## What it does and doesn't mean

Bell's theorem rules out **local** hidden variables. At least one of Einstein's two ingredients has to
go:

- Give up **realism**: the results really don't exist until measured. This is the mainstream reading.
- Give up **locality**: there are hidden variables, but they're nonlocal, as in the pilot-wave
  interpretation, where the guiding wave connects distant particles instantly.

What it **doesn't** allow is faster-than-light communication. Look at Alice's results on their own:
they're a random 50/50 sequence no matter what Bob does. The correlation only appears when Alice and
Bob bring their records together and compare, which needs an ordinary, light-speed-limited channel.
This is called the **no-signalling** theorem. Entanglement is a correlation, not a telephone.

## Why it matters now

What started as an argument about the nature of reality has become engineering:

- **Quantum key distribution**: Artur Ekert's 1991 protocol uses Bell tests to check that nobody has
  intercepted a key. If an eavesdropper interfered, the correlation would drop below the quantum value.
  China's Micius satellite has distributed entangled photons between ground stations over 1,000 km
  apart.
- **Quantum computing**: entanglement between qubits is the resource that lets quantum algorithms
  outperform classical ones on some problems. The 2025 Nobel Prize went to John Clarke, Michel Devoret
  and John Martinis for the 1980s experiments showing quantum behaviour in superconducting circuits, the
  same kind of circuits many of today's quantum computers are built from.
- **Device-independent security**: because a Bell violation can't be faked by local means, you can
  certify randomness or security without trusting the devices you bought.

## Conclusion

Bell's theorem is one of the rare cases where a philosophical debate got settled by experiment:

- Entangled particles show **perfect correlations**, which on their own could be explained by pre-set
  answers, like gloves in boxes.
- But with **changeable measurement angles**, any local pre-set-answer theory obeys `|S| ≤ 2`, and the
  proof is a few lines of algebra.
- Quantum mechanics predicts and **experiments confirm** `|S| ≈ 2√2`, with every major loophole closed
  since 2015.
- Nature is not locally realistic, and yet still can't be used to **signal faster than light**.

Einstein was right that something about quantum mechanics is strange. He was wrong about which part.
The strangeness isn't a gap in the theory; it's the way the world actually works.
