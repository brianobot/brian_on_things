+++
title = "The Double-Slit Experiment: How One Particle Interferes With Itself"
date = 2026-01-10
description = "The double-slit experiment explained: Young's light waves, electrons that interfere one at a time, what happens when you watch which slit they go through, and what it says about reality."
draft = false

[extra]
seo_title = "The Double-Slit Experiment Explained, One Particle at a Time"
toc = true
+++

## Introduction

Richard Feynman once said the double-slit experiment contains "the only mystery" of quantum
mechanics. Everything else that's weird about the quantum world, he argued, is some version of what
happens here.

2025 was the International Year of Quantum Science and Technology, marking a hundred years since
Heisenberg's 1925 paper that founded quantum mechanics. I spent part of it going back to the basics
I had skimmed through in university physics, and the double-slit experiment was the one that kept
pulling me back. It's simple enough to describe in a paragraph, and nobody has fully made peace with
it in a century.

This post walks through it in three acts: with light, with particles, and with a detector watching.
No maths beyond a couple of ratios, plus a short Python simulation you can run yourself.

## TL;DR

- Light through two narrow slits makes a pattern of bright and dark **stripes** (fringes). That's
  **interference**, the signature of a wave.
- Send **electrons** through, **one at a time**, and each lands as a single dot. But the dots
  gradually build up the **same striped pattern**. Each electron seems to interfere with itself.
- Put a detector at the slits to find out which slit each electron used, and the stripes
  **vanish**. You get two plain bands, as if they were bullets.
- Quantum mechanics describes this with a **wavefunction** that passes through both slits, and the
  **Born rule**: the wave tells you only the **probability** of where a particle will be found.

## Act one: light is a wave

In the early 1800s, Thomas Young shone light through two narrow, closely spaced slits onto a screen.
If light were a stream of tiny particles, as Newton had argued, you'd expect two bright bands, one
behind each slit. Instead Young saw a row of bright and dark stripes.

The explanation is **interference**, the same thing you see when two ripples cross on a pond. Light
spreads out from each slit as a wave. At some points on the screen, the waves from the two slits
arrive in step, crest on crest, and add up to a bright fringe. At others they arrive exactly out of
step, crest on trough, and cancel to darkness.

Whether they arrive in step depends on the difference in distance from each slit. That gives a neat
formula for the spacing between bright fringes:

```
fringe spacing = wavelength × screen distance / slit separation
```

Green light (500 nm) through slits 50 µm apart onto a screen 1 metre away gives fringes 1 cm apart.
That's how 19th-century physicists measured the wavelength of light with nothing more than a ruler.

The crucial point: **darkness appears in places that would be lit if only one slit were open.** Open
a second slit, and some spots on the screen get *less* light. Particles can't do that. Waves can.

So by about 1850 the question was settled: light is a wave. Then Einstein unsettled it in 1905 by
showing light also arrives in discrete lumps of energy, **photons**. And in 1924, Louis de Broglie
proposed the reverse: if waves can act like particles, maybe particles act like waves, with a
wavelength set by their momentum:

```
λ = h / p
```

where `h` is Planck's constant. For anything you can see, that wavelength is absurdly small. For an
electron, it's comparable to the spacing between atoms, which is why electron microscopes can see
things light microscopes can't.

## Act two: one electron at a time

Now the strange part. Replace the light with a beam of electrons, and turn the beam down so low that
only **one electron** is in the apparatus at any moment. The screen is a detector that records
exactly where each electron lands.

The famous version of this was done by Akira Tonomura's team at Hitachi in 1989, using an electron
biprism (a fine charged wire that does the same job as two slits). Here's what they saw:

1. Each electron arrives as a **single dot**, at a single point. Not smeared out. Not half here, half
   there. Particle-like.
2. The positions look **random**. You can't predict where the next one will land.
3. As thousands of dots accumulate, **stripes appear**. The same interference pattern as with light.

You can watch this happen with a short simulation. Each particle lands randomly, but the chance of
landing at each point follows the brightness of the two-slit pattern:

```python
import math
import random

WAVELENGTH = 500e-9   # metres (green light)
SLIT_GAP = 50e-6      # distance between the slits, metres
SCREEN = 1.0          # distance from slits to screen, metres
WIDTH = 0.03          # half-width of the screen we look at, metres

def intensity(y: float) -> float:
    """Relative brightness of the two-slit pattern at height y on the screen."""
    phase = math.pi * SLIT_GAP * y / (WAVELENGTH * SCREEN)
    return math.cos(phase) ** 2

def detect_one_particle() -> float:
    """Pick where a single particle lands, with probability set by intensity."""
    while True:
        y = random.uniform(-WIDTH, WIDTH)
        if random.random() < intensity(y):
            return y

def show(hits: int, bins: int = 60) -> None:
    counts = [0] * bins
    for _ in range(hits):
        y = detect_one_particle()
        counts[min(int((y + WIDTH) / (2 * WIDTH) * bins), bins - 1)] += 1
    peak = max(counts)
    shades = " .:-=+*#%@"
    row = "".join(shades[round(c / peak * (len(shades) - 1))] for c in counts)
    print(f"{hits:>6} particles |{row}|")

random.seed(1)
for n in (10, 100, 1_000, 10_000, 100_000):
    show(n)
```

```
    10 particles | @   =           =           =         =     =    @   =     |
   100 particles | **   ---%* : :  :**@--:    :-@- :    =-%=:    =-*--=-  :-**|
  1000 particles |#==.  .-%#*=-:  --*##==.  .:=*%*+.  .++*%+=.   -+#@+=:.. -#%|
 10000 particles |%*=:  :=#@%#=:  :=##%*+:  :=*@%#=:  :=*@%#=:  :=*@%*=:  :=#@|
100000 particles |@#=:  :=#@@#+:  :=#@@#+:  :=#@@#+:  :=#@@#=:  :=#@@#+:  :+#@|
```

At ten particles it's noise. By ten thousand, the fringes are unmistakable. (To be clear, this
script *assumes* the interference pattern; it shows what the build-up looks like, not why it
happens. The real experiment is the evidence.)

Now sit with what this means. There is only one electron in the apparatus at a time. There's no
other electron for it to interfere with. Yet the dark fringes are places where electrons almost never
land, places they *would* land if only one slit were open. Somehow, each individual electron is
affected by **both** slits.

This isn't special to electrons. It's been done with neutrons, atoms, and in 1999 with buckyballs,
football-shaped molecules of 60 carbon atoms. Anton Zeilinger's and Markus Arndt's groups in Vienna
have since pushed it to molecules made of around 2,000 atoms. Everything interferes, if you can keep
it isolated enough.

## Act three: what if we look?

The natural instinct is to catch the electron in the act. Put a detector at the slits that registers
which slit each electron passes through, and then see where it lands.

When you do this, the detector always reports **one slit or the other**, never both. And the
interference pattern on the screen **disappears**. You get two overlapping bands, exactly what
you'd expect from particles that each went through one slit.

Remove the detector, and the stripes come back.

It's tempting to say "the electron knows it's being watched", but that framing is misleading. There's
nothing mystical about a conscious observer here. Any physical interaction that leaves a record of
which slit was used, even a single stray photon bouncing off the electron that nobody ever looks
at, destroys the interference. What matters is whether the **information exists** in the
universe, not whether a person reads it.

Physicists call this **complementarity** (Niels Bohr's term): the setup can reveal wave-like
behaviour (interference) or particle-like behaviour (which path), but not both fully at once. There's
even a smooth trade-off. A weak detector that only *partly* tells you the path leaves you with
faded fringes, and the better the path information, the fainter the stripes.

## How quantum mechanics describes it

Quantum mechanics doesn't say the electron is "really" a wave or "really" a particle. It says the
electron is described by a **wavefunction**, usually written `ψ` (psi). The wavefunction behaves
exactly like a wave: it spreads out, goes through both slits, and interferes with itself.

But you never see the wavefunction directly. When the electron hits the screen, it's detected at a
single point. Where it appears is random, and the **Born rule**, proposed by Max Born in 1926, gives
the odds:

```
probability of finding the particle at a point ∝ |ψ|²
```

The wave decides the **probabilities**; each detection is a single **event** drawn from them. That's
precisely what the simulation above does.

The interference lives in the wavefunction. With both slits open, `ψ` is the sum of a piece from
each slit, `ψ₁ + ψ₂`, and squaring a sum gives cross terms that can be positive or negative. That's
where the bright and dark fringes come from. When a detector records which slit was used, the two
pieces become tied to different detector states, they can no longer interfere, and you're left with
`|ψ₁|² + |ψ₂|²`: two plain bands.

## So what is actually going on?

Here's where honest physicists stop agreeing. The maths is not in dispute; it predicts every result
perfectly. What's disputed is what the maths **means**. A few of the main camps:

- **Copenhagen**: the wavefunction is a tool for predicting measurement results. Asking which slit
  the electron "really" went through, when nothing recorded it, is not a meaningful question.
- **Pilot wave (de Broglie–Bohm)**: there really is a particle, always at a definite position, and it
  goes through one slit. But it's guided by a real wave that goes through both, and the wave steers
  it into the fringe pattern.
- **Many worlds**: the wavefunction is all there is, and it never collapses. When a detector
  records the path, the universe branches into one where it went left and one where it went right.

All three predict the same experimental results, which is why the debate has lasted a century. I
don't have a favourite yet. What I've come to appreciate is that each one makes you give up some
piece of everyday intuition. You don't get to keep all of it.

## Conclusion

The double-slit experiment is short to describe and impossible to un-learn:

- **Waves interfere**, and dark fringes appear where a second open slit removes light.
- **Single particles build up interference patterns**, one random dot at a time, as if each passes
  through both slits.
- **Recording the path destroys the pattern**, whether or not anyone looks at the record.
- Quantum mechanics captures it all with a **wavefunction** that interferes and the **Born rule**
  that turns it into probabilities. What that wavefunction *is* remains an open question.

If the double-slit experiment shows that quantum objects don't have a definite path, the next
natural question is whether they have definite properties at all. That's where Heisenberg's
[uncertainty principle](@/blog/the-uncertainty-principle-is-not-about-clumsy-measurements/index.md) comes in.
