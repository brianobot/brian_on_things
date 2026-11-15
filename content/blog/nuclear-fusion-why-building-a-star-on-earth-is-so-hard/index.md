+++
title = "Nuclear Fusion: Why Building a Star on Earth Is So Hard"
date = 2026-02-14
description = "How nuclear fusion works and why it is so hard: the Coulomb barrier, quantum tunnelling, the Lawson criterion, tokamaks versus lasers, and where fusion energy stands in 2026."
draft = false

[extra]
seo_title = "Nuclear Fusion Explained: Why a Star on Earth Is So Hard"
toc = true
+++

## Introduction

There is an old joke that fusion power is thirty years away, and always will be. It's been told for
so long that it's become part of the subject. And yet the physics is not in doubt: every second,
the Sun fuses around 600 million tonnes of hydrogen, and has been doing it for 4.6 billion years.
Fusion obviously works. The question is why it's so devilishly hard to do it here.

This is the third post in a small series. In the first, I explained the
[binding energy curve](@/blog/binding-energy-why-iron-is-the-end-of-the-line/index.md), and in the
second, how [fission](@/blog/nuclear-fission-how-splitting-an-atom-powers-a-city/index.md) climbs it
from the heavy side. Fusion climbs it from the light side, and it's the bigger prize. You don't
need to have read either post to follow this one.

## TL;DR

- Fusing light nuclei, like hydrogen isotopes into helium, releases energy because helium is far
  more tightly bound. Per kilogram, it beats fission by about **four times**.
- The obstacle is the **Coulomb barrier**: nuclei are positively charged and repel each other, so
  you need them extremely hot (~150 million °C) to get close enough.
- Even then, they only fuse thanks to **quantum tunnelling**.
- A power plant needs hot enough plasma, dense enough, held for long enough: the **Lawson
  criterion**. Magnetic confinement (tokamaks, stellarators) and inertial confinement (lasers) are
  two different bets on how to get there.
- In late 2022, the US National Ignition Facility got more fusion energy out of a target than the
  lasers put in, a first, and it has since improved that. Nobody has yet produced net
  **electricity**.

## The reaction everyone is chasing

The Sun fuses ordinary hydrogen, but slowly (more on that below). Every serious fusion power design
on Earth uses a faster reaction between two heavier hydrogen isotopes, **deuterium** (one proton,
one neutron) and **tritium** (one proton, two neutrons):

```
²H + ³H  →  ⁴He (3.5 MeV) + n (14.1 MeV)
```

Total energy: **17.6 MeV** per reaction. The helium nucleus is the star of the binding energy curve,
unusually tightly bound for its size, and dropping into that state releases a lot of energy.

Per reaction, 17.6 MeV is less than the ~200 MeV of a uranium fission. But only five nucleons are
involved instead of 236. Per kilogram of fuel, D-T fusion releases about **340 terajoules**, roughly
four times more than fission and around **ten million times** more than burning coal.

Notice that four-fifths of the energy leaves with the **neutron**. Neutrons have no charge, so no
magnetic field can hold them. They fly straight out of the plasma and into the walls. That fact
quietly shapes almost every fusion engineering problem, and I'll come back to it.

## The real enemy: the Coulomb barrier

Here is the core difficulty. The strong nuclear force that makes fusion possible only reaches about
a femtometre or two. To feel it, two nuclei have to almost touch. But both nuclei are positively
charged, and the electric repulsion between them grows stronger the closer they get.

For deuterium and tritium, the repulsion peaks at around **0.4 MeV** just before the strong force
takes over. That's the **Coulomb barrier**: a hill of electric potential energy the nuclei have to
climb.

The obvious way to give particles energy is to heat them. In a gas, temperature is a measure of
the particles' average kinetic energy. So how hot would deuterium and tritium need to be to have an
average energy of 0.4 MeV? About **4.6 billion kelvin**. The core of the Sun is about 15 million
kelvin, three hundred times cooler. By classical physics alone, the Sun shouldn't shine.

## Quantum tunnelling to the rescue

Classical physics says a particle without enough energy to get over a hill simply can't. Quantum
mechanics disagrees. A particle is described by a wave, and that wave doesn't stop dead at a
barrier. It decays inside it, and if the barrier is thin enough, some of the wave leaks out the
other side. The particle has a small but real probability of appearing beyond a barrier it couldn't
classically climb. This is **quantum tunnelling**.

The tunnelling probability rises steeply with energy: faster nuclei face a thinner, lower part of
the barrier. Meanwhile, in a hot gas, very few particles are fast. Their energies follow a
Maxwell–Boltzmann distribution, which drops off exponentially at high energy.

Multiply these two together, "plenty of particles but bad tunnelling" at low energy and "great
tunnelling but almost no particles" at high energy, and you get a narrow window where most fusion
actually happens. This is the **Gamow peak**, named after George Gamow who first worked out
tunnelling for nuclear decay in 1928.

Here's a small script that computes where that peak sits:

```python
import math

ALPHA = 1 / 137.036          # fine-structure constant
U_TO_KEV = 931_494.0         # atomic mass unit in keV/c^2
K_B = 8.617e-8               # Boltzmann constant in keV per kelvin

def gamow_peak(Z1, Z2, m1, m2, temperature_k):
    """Return (Gamow energy, thermal energy kT, most likely fusion energy), all in keV."""
    reduced_mass = m1 * m2 / (m1 + m2) * U_TO_KEV
    e_gamow = 2 * reduced_mass * (math.pi * ALPHA * Z1 * Z2) ** 2
    kT = K_B * temperature_k
    peak = (e_gamow * kT ** 2 / 4) ** (1 / 3)
    return e_gamow, kT, peak

cases = {
    "p + p in the Sun's core (15.7 million K)": (1, 1, 1.00728, 1.00728, 15.7e6),
    "D + T in a reactor (150 million K)": (1, 1, 2.01355, 3.01550, 150e6),
}

for label, (Z1, Z2, m1, m2, T) in cases.items():
    e_g, kT, peak = gamow_peak(Z1, Z2, m1, m2, T)
    print(f"{label}\n  kT = {kT:.2f} keV, Gamow peak = {peak:.1f} keV ({peak / kT:.1f} x kT)")
```

```
p + p in the Sun's core (15.7 million K)
  kT = 1.35 keV, Gamow peak = 6.1 keV (4.5 x kT)
D + T in a reactor (150 million K)
  kT = 12.93 keV, Gamow peak = 36.7 keV (2.8 x kT)
```

Even in a fusion reactor, the nuclei that actually fuse carry around 37 keV: a tenth of the height
of the barrier. Every single fusion reaction, in the Sun and in our machines, is a tunnelling
event.

## Why the Sun gets away with it, and we can't

The Sun has two huge advantages. It has **gravity** to hold its fuel together for free, and it has
**time**. Its core is so vast and dense that even a pitifully slow reaction rate adds up to an
enormous output.

And it really is slow. The first step of the Sun's proton–proton chain needs one proton to turn into
a neutron mid-collision, via the weak nuclear force, which almost never happens. A given proton in
the Sun's core waits, on average, billions of years before it fuses. The power density in the
solar core is only around **275 watts per cubic metre**, comparable to a compost heap. The Sun is
bright because it is big, not because it is intense.

On Earth we have neither the gravity nor the size. So we compensate in two ways:

1. **Use a much easier reaction.** D-T needs no weak-force miracle. Its cross-section is around
   10²⁴ times larger than proton–proton fusion at similar energies.
2. **Run much hotter than the Sun.** Around **150 million kelvin**, ten times the temperature of the
   solar core, which is where D-T's reaction rate is near its best per unit of pressure.

At those temperatures, electrons are stripped from their nuclei and the fuel becomes a **plasma**, a
soup of charged particles. No material can touch it. Hold it in a container and the plasma cools
instantly while the container vaporises.

## The Lawson criterion

In 1955, the engineer John Lawson asked the question in its most practical form: when does a fusion
plasma produce more energy than it loses? The modern version for D-T comes down to the **triple
product**:

```
n × T × τ_E  ≳  3 × 10²¹  keV·s/m³
```

- `n`: the **density** of fuel ions. More particles, more collisions.
- `T`: the **temperature**. Hot enough to sit near D-T's sweet spot.
- `τ_E`: the **energy confinement time**. How long the plasma holds its heat before losing it.

If the triple product clears that bar, the energy from the helium nuclei (which, being charged,
stay in the plasma) is enough to keep the plasma hot without any outside heating. That's
**ignition**: a self-sustaining burn, like a fire that no longer needs a match.

Every fusion approach is a strategy for maximising that product, and they fall into two camps.

## Two ways to bottle a star

### Magnetic confinement: low density, long time

Charged particles in a magnetic field spiral along the field lines. Bend the field lines into a
closed loop, and in principle the particles go round forever without touching a wall.

A simple ring of field doesn't quite work. The field is stronger on the inside of the ring than the
outside, and particles drift out. The fix is to **twist** the field lines as they go round, so each
particle samples both sides and the drifts cancel. There are two ways to do the twisting:

- The **tokamak**, a Soviet design from the 1950s, drives a large electric current through the
  plasma itself, and the current's own magnetic field supplies the twist. It's the most advanced
  design, but that current is a source of instabilities. The worst are **disruptions**, where the
  plasma loses confinement in milliseconds and dumps its energy into the walls.
- The **stellarator** builds the twist into the external magnets, which are wound into bizarre
  three-dimensional shapes. No plasma current is needed, so it's naturally steady and calmer, but
  the magnets are much harder to design and build. Germany's Wendelstein 7-X is the flagship.

Magnetic confinement works with thin plasma, around a millionth of the density of air, held for
seconds or longer. The European JET tokamak in the UK set the record for total fusion energy in a
single pulse, 69 megajoules over about five seconds, in its final campaign in 2023 before it shut
down. **ITER**, the enormous international tokamak under construction in southern France, is meant
to produce ten times more fusion power than the heating it receives. Its schedule has slipped
repeatedly; under its 2024 baseline, deuterium–tritium operation isn't due until the late 2030s.

The most interesting recent shift is in **magnets**. A tokamak's performance rises steeply with
magnetic field strength. New **high-temperature superconducting** tape (REBCO) can produce much
stronger fields than the superconductors ITER uses, which means a reactor could get the same
performance in a far smaller machine. Commonwealth Fusion Systems demonstrated a 20-tesla magnet
built this way in 2021 and is building a compact tokamak called **SPARC** around it, aiming to show
net energy gain in the plasma later this decade.

### Inertial confinement: high density, tiny time

The other camp turns the problem inside out. Instead of holding a thin plasma for a long time, crush
a speck of fuel to enormous density and let it burn in the instant before it flies apart. The fuel's
own inertia is the "container".

The **National Ignition Facility (NIF)** in California does this with 192 lasers. They fire into a
small gold cylinder called a **hohlraum**, about the size of a pencil eraser, turning the laser light
into X-rays. Those X-rays blast the surface off a peppercorn-sized capsule of frozen D-T, and the
recoil compresses the fuel inward like a rocket in reverse. For a few billionths of a second the
centre reaches over 100 million kelvin and densities around a hundred times that of lead.

On 5 December 2022, NIF delivered 2.05 MJ of laser energy to the target and got **3.15 MJ** of
fusion energy back. For the first time, a controlled fusion experiment produced more energy than
was put into the fuel. NIF has repeated and beaten that since, reaching yields of around 8.6 MJ in
2025.

## Gain: the fine print

Headlines about "net energy" need careful reading, and this is where my engineering side wants to
be pedantic. There are several different ratios people call **Q**:

| Measure | What it compares | Status |
| --- | --- | --- |
| Scientific gain | Fusion energy out ÷ energy delivered to the fuel | Above 1 at NIF since 2022 |
| Facility gain | Fusion energy out ÷ energy drawn from the grid to run the machine | Far below 1 |
| Engineering gain | Electricity sold ÷ electricity used | Not yet attempted |

NIF's lasers draw around 300 MJ from the grid to deliver those 2 MJ of laser light, because the
lasers are about 1% efficient (they were built in the 1990s for weapons research, not efficiency).
And a power plant would then need to convert fusion heat to electricity at maybe 40%, and fire
several shots **per second**, not one a day. The 2022 result was a genuine scientific milestone.
It's not yet a power plant, nor close to one.

## The problems after ignition

Getting the plasma to burn is the famous problem. Turning that into a power station brings a second
list, and these are mostly materials and engineering problems, not physics:

- **The neutrons.** Those 14.1 MeV neutrons carry 80% of the energy and hammer the walls, knocking
  atoms out of place and making the structure brittle and mildly radioactive over time. We still
  need materials that can survive years of it. A dedicated test facility (IFMIF-DONES in Spain) is
  being built specifically for this.
- **Tritium.** Tritium is radioactive with a half-life of 12.3 years, so there's almost none in
  nature, and the world's civilian stockpile is only a few tens of kilograms. A reactor would have
  to **breed** its own by surrounding the plasma with a lithium **blanket**, where neutrons do
  `⁶Li + n → ³H + ⁴He`. Breeding more tritium than you burn, reliably, has never been demonstrated.
- **The exhaust.** Heat leaving the plasma is funnelled onto a component called the **divertor**,
  which faces heat fluxes comparable to a spacecraft re-entering the atmosphere, continuously.
- **Staying on.** A power plant needs to run steadily for months. Tokamaks have to manage
  instabilities, and laser plants would need to make, place and hit hundreds of thousands of
  precision targets every day, cheaply.

## Why bother?

Because the payoff is extraordinary, and the downsides compared with fission are small:

- **Fuel.** Deuterium can be extracted from seawater, about one in every 6,400 hydrogen atoms.
  Lithium for breeding tritium is plentiful. There is fuel for millions of years.
- **No runaway.** There's no chain reaction. A fusion plasma is so hard to sustain that any fault
  makes it go out. The total fuel in the chamber at any moment is around a gram.
- **No long-lived waste.** The helium product is harmless. The reactor structure becomes
  radioactive from the neutrons, but with the right materials that decays over decades to a century,
  not millennia.
- **No carbon**, and a power station that runs regardless of weather, unlike wind and solar.

## Conclusion

Fusion is hard for one reason that sits on top of everything else: positively charged nuclei don't
want to get close to each other, and the only way past that is to make them so hot no material
can hold them. The rest of the story follows:

- **D-T fusion** releases 17.6 MeV per reaction and about four times more energy per kilogram than
  fission.
- Nuclei only fuse by **quantum tunnelling** through the Coulomb barrier, even at 150 million kelvin.
- **Lawson's triple product**, density × temperature × confinement time, is the scorecard every
  machine is judged by.
- **Scientific breakeven has happened**, at NIF in 2022, but engineering breakeven, with real
  electricity, hasn't.

I've come away from writing this series more optimistic than I expected. The 30-year joke used to be
funny because nobody had crossed a single real milestone. That's no longer true. There's still a
long engineering road between "it works" and "it's on the grid", but for the first time, it's an
engineering road.
