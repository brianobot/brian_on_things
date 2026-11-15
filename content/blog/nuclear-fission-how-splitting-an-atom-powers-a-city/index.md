+++
title = "Nuclear Fission: How Splitting an Atom Powers a City"
date = 2025-12-20
description = "How nuclear fission works: why uranium-235 splits, where the 200 MeV comes from, chain reactions and critical mass, and how delayed neutrons make reactors controllable."
draft = false

[extra]
seo_title = "How Nuclear Fission Works: From Uranium to Power Plants"
toc = true
+++

## Introduction

In my [previous post on binding energy](@/blog/binding-energy-why-iron-is-the-end-of-the-line/index.md)
I described nuclear energy as climbing a hill: nuclei release energy whenever they move towards
iron, the most tightly bound nucleus there is. Light nuclei climb by fusing. Heavy nuclei climb by
splitting.

This post is about the second route, **fission**, and it's the one humanity actually has working
at scale. Roughly a tenth of the world's electricity comes from it today. As someone who studied
electrical engineering, I spent years treating power stations as boxes that turn "heat" into
electricity, without asking much about where the heat came from. In a nuclear plant, it comes from
splitting atoms, and the physics of how that's done safely is more interesting than I expected.

You don't need to have read the previous post, but if terms like "binding energy per nucleon"
are new to you, it's a quick detour.

## TL;DR

- A slow neutron hitting **uranium-235** makes it wobble apart into two medium-sized nuclei, a few
  free neutrons, and about **200 MeV** of energy.
- Those freed neutrons can split more uranium, giving a **chain reaction**. The key number is the
  **multiplication factor `k`**: on average, how many new fissions each fission causes.
- Bombs need `k` well above 1 and fast. Reactors need `k` held at **exactly 1**.
- That's only possible because a small fraction of neutrons arrive **seconds** late. These
  **delayed neutrons** slow the reactor's response enough for control rods to keep up.

## A short history

Fission was discovered almost by accident. In December 1938, Otto Hahn and Fritz Strassmann in
Berlin were bombarding uranium with neutrons, expecting to make slightly heavier elements. Instead,
their careful chemistry kept turning up **barium**, an element barely half uranium's mass.

Hahn wrote to his long-time collaborator Lise Meitner, who had fled Nazi Germany to Sweden. Over
the Christmas holiday, Meitner and her nephew Otto Frisch worked out what was happening: the
uranium nucleus wasn't being chipped, it was **splitting in two**. They estimated the energy
released at around 200 MeV per split, borrowed the word "fission" from biology, and published in
early 1939.

Within four years, on 2 December 1942, Enrico Fermi's team in Chicago ran the first self-sustaining
chain reaction in a pile of graphite and uranium under a stadium. Nature had beaten them to it,
though: around two billion years ago, uranium deposits at **Oklo** in Gabon ran as natural fission
reactors, on and off, for hundreds of thousands of years. Groundwater acted as the moderator.

## How a nucleus splits

### The liquid drop picture

A heavy nucleus like uranium is held together by the short-range strong force and pushed apart by
the repulsion between its 92 protons. It's like a drop of liquid with surface tension barely
holding against a charge trying to blow it apart.

When uranium-235 absorbs a neutron, it becomes **uranium-236** in an excited state, carrying an
extra ~6.5 MeV of energy. That energy sets the drop **oscillating**. It stretches into a peanut
shape, and once the neck between the two lobes gets thin enough, the electric repulsion between
the lobes wins over the strong force across the neck. The drop snaps.

The energy needed to reach that snapping point is the **fission barrier**, about 6 MeV for
uranium-236. The neutron's arrival delivers more than that, so fission happens readily.

### Why U-235 and not U-238

Natural uranium is 99.3% uranium-238 and only 0.7% uranium-235. Why does everyone care about the
rare one?

It comes down to **pairing**. Nucleons like to be in pairs. U-235 has an **odd** number of
neutrons (143), so when it captures one, the new neutron completes a pair and the nucleus gets a
bonus chunk of binding energy, enough to clear the fission barrier even if the incoming neutron
was moving slowly.

U-238 already has an even number of neutrons (146). Adding one more leaves an unpaired neutron, so
the energy bonus is smaller and falls short of the barrier. U-238 only fissions if the neutron
brings in more than about 1 MeV of kinetic energy of its own.

Nuclei that fission with slow neutrons are called **fissile**: U-235, U-233 and plutonium-239 are
the important ones. This is why reactor fuel is **enriched**: commercial reactors typically use
uranium that's 3–5% U-235.

### A typical split

There's no single way for U-236 to break. It splits **asymmetrically**, usually into one fragment
around mass 95 and another around mass 140. One common example:

```
²³⁵U + n  →  ²³⁶U*  →  ¹⁴¹Ba + ⁹²Kr + 3n
```

Check the bookkeeping: mass numbers `235 + 1 = 141 + 92 + 3 = 236`, and protons `92 = 56 + 36`.
Nothing is created or destroyed, just rearranged.

## Where the 200 MeV goes

Barium and krypton both sit closer to iron on the binding energy curve than uranium does. Each
nucleon gains just under 1 MeV of binding, and with ~236 nucleons that sums to around 200 MeV.

I can estimate it with the semi-empirical mass formula from the previous post, by comparing the
binding energy of the products to the binding energy of the compound nucleus:

```python
released = binding_energy(56, 141) + binding_energy(36, 92) - binding_energy(92, 236)
print(f"U-236 -> Ba-141 + Kr-92 + 3n releases about {released:.0f} MeV")
# U-236 -> Ba-141 + Kr-92 + 3n releases about 155 MeV
```

A rough formula gives 155 MeV for the split itself, a bit below the measured ~170 MeV for this
channel, but the right ballpark. The rest comes later: the fragments are packed with too many
neutrons and **beta-decay** over the following seconds to years, releasing more energy as they go.

Averaged over all fissions, the budget for U-235 looks roughly like this:

| Where the energy goes | MeV |
| --- | --- |
| Kinetic energy of the two fragments | ~169 |
| Kinetic energy of the free neutrons | ~5 |
| Prompt gamma rays | ~7 |
| Beta and gamma decay of fragments | ~13 |
| Antineutrinos (escape, lost) | ~10 |
| **Total** | **~200** |

The bulk of it is the two fragments flying apart, pushed by their mutual electric repulsion. They
slam into surrounding atoms within a few micrometres and their kinetic energy becomes **heat**.
That's the heat that boils water and spins turbines.

Note the fragment decay line. That energy keeps coming even after the chain reaction stops. Right
after shutdown, a reactor still produces about 6–7% of its full power as **decay heat**, falling to
around 1% after a few hours. This is why reactors need cooling long after they're switched off,
and why losing that cooling at Fukushima in 2011 was a disaster even though the reactors had
already shut down.

### Putting it in perspective

200 MeV per atom is about `3.2 × 10⁻¹¹` joules, which sounds tiny. But a kilogram of U-235 has
around `2.6 × 10²⁴` atoms, so fissioning all of it releases about **82 terajoules**. A kilogram of
good coal gives about 25 megajoules. That's a ratio of over **three million to one**.

## The chain reaction

Each fission releases on average about **2.4 neutrons**. If at least one of those goes on to cause
another fission, the process sustains itself. The number that matters is the **multiplication
factor**:

```
k = (fissions in this generation) / (fissions in the previous generation)
```

- `k < 1`, **subcritical**: the reaction dies out.
- `k = 1`, **critical**: the reaction holds steady. This is a reactor running normally.
- `k > 1`, **supercritical**: the reaction grows exponentially.

Note that "critical" is a calm, steady state, not an emergency, despite how films use the word.

Not every neutron causes another fission. Some escape through the surface. Some get absorbed by
U-238, structural materials, or fission products without causing a split. So `k` depends on the
material, its enrichment, its shape, and its size. That last one leads to the idea of **critical
mass**: a lump has to be big enough that neutrons are more likely to hit another nucleus than leak
out. For a bare sphere of pure U-235 that's around 50 kg; for plutonium-239 it's around 10 kg.

## Slowing neutrons down: the moderator

Neutrons are born fast, at around 2 MeV. But U-235 is hundreds of times more likely to absorb a
**slow** (thermal) neutron than a fast one. So most reactors deliberately slow their neutrons
down with a **moderator**.

The trick is to let the neutrons bounce off light nuclei. Think of snooker: a ball hitting another
ball of the same mass can lose all its speed in one collision, but a ball hitting a bowling ball
just bounces off at almost the same speed. Hydrogen, with nearly the same mass as a neutron, is
the best at this. That's why ordinary water is the moderator (and coolant) in most reactors in the
world. Graphite and heavy water are the common alternatives, both absorbing fewer neutrons, which
is why reactors moderated with them can run on unenriched natural uranium.

Water moderation also gives a built-in safety feature: if the water overheats and turns to steam,
it gets less dense, moderates less, and `k` drops. The reactor turns itself down.

## Why reactors can be controlled at all

This is the part that genuinely surprised me when I first worked through the numbers.

In a water-moderated reactor, a neutron generation lasts somewhere around **0.1 milliseconds**:
born, slowed down, absorbed, causing the next fission. That's 10,000 generations per second. If `k`
were even slightly above 1, say 1.001, the power would multiply by `1.001^10000`, about **22,000
times**, in a single second. No mechanical control rod could possibly keep up.

The rescue is that not all neutrons arrive instantly. About **0.65%** of the neutrons from U-235
fission are **delayed**: they're emitted by fission fragments that decay a few seconds to a minute
later. The average delay is around 13 seconds.

0.65% sounds negligible, but it stretches the **average** time between generations dramatically.
Here's the arithmetic:

```python
import math

PROMPT_LIFETIME = 1e-4    # seconds between neutron generations, prompt only
BETA = 0.0065             # fraction of U-235 fission neutrons that are delayed
DELAY = 12.7              # mean delay of those neutrons, in seconds

def growth_after(seconds: float, k: float, lifetime: float) -> float:
    """Factor the neutron population (and power) grows by after `seconds`."""
    period = lifetime / (k - 1)
    return math.exp(seconds / period)

effective_lifetime = PROMPT_LIFETIME + BETA * DELAY

for k in (1.0001, 1.001):
    prompt = growth_after(1.0, k, PROMPT_LIFETIME)
    delayed = growth_after(1.0, k, effective_lifetime)
    print(f"k = {k}: prompt only x{prompt:,.1f} per second, with delayed neutrons x{delayed:.3f} per second")
```

```
k = 1.0001: prompt only x2.7 per second, with delayed neutrons x1.001 per second
k = 1.001: prompt only x22,026.5 per second, with delayed neutrons x1.012 per second
```

With delayed neutrons included, the effective generation time is closer to **0.08 seconds**, and
the same slight excess in `k` grows the power by about 1% per second instead of 22,000-fold. That's
slow enough for control rods, made of neutron-hungry materials like boron or cadmium, to be moved in
and out to keep `k` at exactly 1.

The golden rule of reactor design follows from this: the reactor must **never be critical on prompt
neutrons alone**. As long as it needs the delayed neutrons to stay critical, it changes slowly
enough to control. Chernobyl in 1986 is the tragic counterexample: a combination of design flaws
and operating decisions pushed reactor 4 past this line, and its power spiked by orders of
magnitude in seconds.

A bomb is the opposite. It has no moderator, no delayed neutrons to wait for, and it's assembled to
be strongly supercritical on fast neutrons. The whole reaction is over in about a microsecond.

## Inside a power reactor

Putting the pieces together, a typical **pressurised water reactor** looks like this:

1. **Fuel**: pellets of uranium dioxide enriched to 3–5% U-235, stacked in long metal rods, bundled
   into assemblies.
2. **Moderator and coolant**: water flows between the rods, slowing neutrons and carrying heat away.
   It's held at about 155 bar so it stays liquid at around 300 °C.
3. **Control rods**: slide between fuel rods to absorb neutrons and adjust `k`. Dissolved boron in
   the water provides fine adjustment.
4. **Steam generator**: the hot, pressurised water heats a separate water loop, which boils.
5. **Turbine and generator**: the steam spins a turbine connected to a generator. From here on it's
   the same as a coal or gas plant.

That last step is the humbling part for an electrical engineer: all that nuclear physics, and at
the end of the day it's an extremely sophisticated kettle.

## The catch: waste

The fission fragments that made the energy are themselves the problem. They're neutron-rich and
intensely radioactive. Some, like caesium-137 and strontium-90, have half-lives around 30 years.
On top of that, U-238 in the fuel absorbs neutrons and turns into plutonium and other heavy
elements, some of which stay radioactive for tens of thousands of years.

The volume is small. All the spent fuel a large reactor produces in a year would fit in a modest
room. But it has to be cooled for years, then isolated for millennia. Finland's Onkalo repository,
carved into bedrock hundreds of metres down, is the first deep geological disposal site for spent
fuel. For most countries, though, the long-term plan is still a plan.

## Conclusion

Fission is the side of the binding energy hill we have actually conquered. The core ideas:

- **U-235 splits easily** because absorbing one neutron completes a pair and pushes it over the
  fission barrier.
- **~200 MeV per fission**, mostly the kinetic energy of two fragments repelling each other, which
  becomes heat.
- **Chain reactions** are governed by `k`, and reactors live at `k = 1`.
- **Delayed neutrons** are the quiet hero: a 0.65% sliver of late neutrons is the difference between
  a power station and an uncontrollable spike.

Fission's drawbacks, long-lived waste, the risk of losing cooling, and its link to weapons, are
exactly what make the other side of the hill so tempting. In the [next post](@/blog/nuclear-fusion-why-building-a-star-on-earth-is-so-hard/index.md) I'll look at fusion:
what it would take to build a star on Earth, and why it's taken so long.
