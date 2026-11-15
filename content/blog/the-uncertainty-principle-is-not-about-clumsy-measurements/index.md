+++
title = "The Uncertainty Principle Is Not About Clumsy Measurements"
date = 2026-03-07
description = "Heisenberg's uncertainty principle explained properly: why it comes from waves, not measuring tools, what Δx·Δp ≥ ħ/2 really says, and how it stops atoms from collapsing."
draft = false

[extra]
seo_title = "Heisenberg's Uncertainty Principle, Explained Properly"
toc = true
+++

## Introduction

Ask most people what Heisenberg's uncertainty principle says and you'll hear something like: "you
can't measure a particle without disturbing it." To see an electron you have to bounce light off it,
the light kicks it, and so you lose track of where it was going.

That explanation is intuitive, and Heisenberg himself used a version of it in 1927. It's also
misleading. The uncertainty principle isn't a limit on our instruments. It's a statement about what
a particle **is**. Even with perfect, gentle, impossible measuring tools, a particle simply does not
have a precise position and a precise momentum at the same time.

In this post I want to make that idea feel obvious rather than mysterious. The trick is to think
about waves. Then I'll use the principle to do something I find delightful: work out the size of a
hydrogen atom in about ten lines of Python.

## TL;DR

- Quantum particles are described by waves, and **momentum is the wavelength** (`p = h/λ`).
- A wave with a precise wavelength has to be spread out. A wave squeezed into a small region has to
  mix many wavelengths. That's not physics, it's a property of all waves.
- So **Δx · Δp ≥ ħ/2**: the spread in position times the spread in momentum can't go below a fixed
  limit. No measurement required.
- The same idea explains why electrons don't spiral into the nucleus, why there's **zero-point
  energy**, and why atoms are the size they are.

## Start with waves, not particles

Forget quantum mechanics for a moment and think about sound.

A pure musical note, a single perfect frequency, is a sine wave that goes on and on. To be *exactly*
one frequency, it has to last forever. If you cut it short, say a quick tap on a drum, the sound is
localised in time, but it no longer has one pitch. A short click contains a whole spread of
frequencies.

This is a mathematical fact about waves, usually expressed through the **Fourier transform**: any
pulse can be built by adding together sine waves of different frequencies, and the shorter the
pulse, the wider the range of frequencies you need. Engineers know this as the **time–bandwidth
product**. It's why a radio signal that carries fast-changing data needs a wide frequency band, and
it's something I first met in signals and systems classes, long before I connected it to quantum
physics.

The same is true in space instead of time. A wave with a single, precise **wavelength** has to
stretch across all of space. A wave packet confined to a small region has to be built from many
different wavelengths.

## Now add quantum mechanics

Here's the quantum ingredient, due to Louis de Broglie: a particle is described by a wave, and its
**momentum is tied to its wavelength**:

```
p = h / λ
```

Put the two ideas together:

- A particle with a **precise momentum** has a precise wavelength, so its wave is spread across all of
  space. Its position is completely undefined.
- A particle with a **precise position** has a wave squeezed into a tiny region, which requires many
  wavelengths mixed together. Its momentum is completely undefined.
- Anything in between is a trade-off.

That's the uncertainty principle. It's not that the particle has a definite position and momentum
that we can't quite pin down. It's that a wave **can't have** both a sharp location and a sharp
wavelength, and a quantum particle is that wave.

## The formula

Made precise, the relationship is:

```
Δx · Δp ≥ ħ / 2
```

- `Δx` is the **spread** (standard deviation) of position.
- `Δp` is the spread of momentum.
- `ħ` ("h-bar") is the reduced Planck constant, `h / 2π ≈ 1.055 × 10⁻³⁴ J·s`.

The exact form with `ħ/2` was proved in 1927 by Earle Kennard, shortly after Heisenberg's paper. The
equality is reached only by one shape of wave packet: a **Gaussian**, the bell curve. Every other
shape is more spread out.

Two notes worth making.

**The spreads are about many repeated experiments.** Prepare a million electrons in the same state.
Measure position on half of them, momentum on the other half. The spreads in those two sets of
results always obey the inequality. Each individual measurement can be as precise as your equipment
allows. What you can't do is prepare a state where both sets of results come out sharp.

**ħ is tiny.** For a 1 gram marble whose position is known to within a micrometre, the minimum
uncertainty in its velocity is about `5 × 10⁻²⁶` m/s. Your marble would take longer than the age of
the universe to drift a millimetre. That's why you never notice this in everyday life. For an
electron, with a mass about 10²⁷ times smaller, the effect is everything.

## Why "measurement disturbs it" is the wrong picture

Heisenberg's original argument was the **gamma-ray microscope**: to locate an electron precisely,
you need short-wavelength light, which carries a big momentum kick, which disturbs the electron's
momentum.

That's a real effect, but it suggests the electron *had* a precise position and momentum before you
interfered with it. Experiments say otherwise. The famous tests of
[Bell's theorem](@/blog/entanglement-and-bells-theorem-the-experiment-that-proved-einstein-wrong/index.md) rule out any theory in which particles carry definite, pre-existing values for
all their properties, if those values only depend on local influences. The uncertainty is in the
particle, not in our clumsiness.

There's also an elegant way to see it from the
[double-slit experiment](@/blog/the-double-slit-experiment-how-one-particle-interferes-with-itself/index.md). Send a particle through a
single narrow slit. Narrowing the slit pins down its sideways position (small `Δx`), and the
particle's landing spot on the screen spreads out wider (large `Δp`). That's **diffraction**. No one
is measuring anything at the slit. It's just a wave going through a gap.

## Why atoms don't collapse

This is the part that made the uncertainty principle click for me.

Classical physics has an embarrassing problem with atoms. An electron orbiting a proton is an
accelerating charge, and accelerating charges radiate. It should lose energy and spiral into the
nucleus in about a hundred-billionth of a second. Every atom in the universe should have collapsed.

The uncertainty principle stops it. If the electron gets squeezed into a smaller region of size `r`,
its momentum spread grows to roughly `ħ / r`, which means more kinetic energy, roughly `ħ²/(2mr²)`.
Meanwhile its electric potential energy falls as `-ke²/r`. The kinetic term grows faster as `r`
shrinks, so there's a size where the total energy is lowest. That's the atom.

Let's find it:

```python
HBAR = 1.054_571_8e-34     # reduced Planck constant, J·s
M_E = 9.109_383_7e-31      # electron mass, kg
KE2 = 2.307_077_6e-28      # e² / (4πε₀), J·m
EV = 1.602_176_6e-19       # joules per electron-volt

def energy(r: float) -> float:
    """Total energy of an electron squeezed into a region of size r."""
    kinetic = HBAR ** 2 / (2 * M_E * r ** 2)   # from Δp ≈ ħ / r
    potential = -KE2 / r                       # attraction to the proton
    return kinetic + potential

# Scan sizes from 0.01 to 5 ångström and keep the one with the lowest energy
sizes = [i * 1e-13 for i in range(100, 50_000)]
best = min(sizes, key=energy)

print(f"Most stable size: {best * 1e10:.3f} Å")
print(f"Energy there:     {energy(best) / EV:.2f} eV")
for r in (0.1e-10, 0.25e-10, best, 2e-10):
    print(f"  r = {r * 1e10:5.2f} Å -> E = {energy(r) / EV:8.2f} eV")
```

```
Most stable size: 0.529 Å
Energy there:     -13.61 eV
  r =  0.10 Å -> E =   237.00 eV
  r =  0.25 Å -> E =     3.36 eV
  r =  0.53 Å -> E =   -13.61 eV
  r =  2.00 Å -> E =    -6.25 eV
```

0.529 ångström is the **Bohr radius**, the textbook size of a hydrogen atom. And −13.6 eV is
**exactly** the measured ground-state energy of hydrogen: the energy it takes to ionise it.

I'll confess that landing on both numbers exactly is partly luck: `Δp ≈ ħ/r` is a rough estimate,
and a different choice of constant would shift the answer by a factor of order one. But the point
stands. Squeeze the electron and its kinetic energy blows up (look at the 237 eV at 0.1 Å). The
uncertainty principle is literally what gives matter its size. It's the reason you don't fall through
the floor.

## Zero-point energy

The same argument says a quantum system can never be perfectly still. A particle sitting motionless
at the bottom of a valley would have precise position *and* zero momentum, which is forbidden. So
even at absolute zero, there's some unavoidable jiggling, called **zero-point energy**.

It has real consequences. It's why **helium never freezes** at normal pressure, even at absolute
zero: helium atoms are so light and so weakly attracted to each other that their zero-point motion
is enough to stop them locking into a crystal. You have to squeeze it to about 25 atmospheres to
make solid helium.

## Other pairs

Position and momentum aren't the only pair with this trade-off. In general, any two quantities whose
quantum operators don't commute have one. The best known others:

- **Spin** along different axes. An electron can't have a definite spin both up–down and
  left–right. Measure one and the other becomes completely random.
- **Energy and time**, in a slightly different sense: a state that only lasts a short time has a
  spread of energies. That's why short-lived excited states emit light with a broader range of
  colours, and why unstable particles have a "width" in their mass.

## Conclusion

The uncertainty principle is more natural, and more fundamental, than its popular description
suggests:

- It comes from **waves**: a sharp position and a sharp wavelength can't coexist, and in quantum
  mechanics wavelength *is* momentum.
- `Δx · Δp ≥ ħ/2` is about the **state of the particle**, not the quality of the measuring tools.
- It's not a nuisance at the edge of physics. It sets the **size of atoms**, gives every system a
  **zero-point energy**, and keeps helium liquid at absolute zero.

The next question is the one that troubled Einstein most: if a particle doesn't have definite
properties until they're measured, what happens when two particles share a state and are measured
far apart? That's
[entanglement](@/blog/entanglement-and-bells-theorem-the-experiment-that-proved-einstein-wrong/index.md), and it gets its own post.
