+++
title = "When the Wind Stops: Stress-Testing the UK's Clean Power 2030 Plan"
date = 2026-02-12
lastmod = 2026-11-22
draft = false
description = "A Python model that replays the worst week of 2025 weather on the grid the UK plans to have by 2030, and measures what wind, solar, nuclear and batteries can't cover."
summary = "I replayed a cold, still week of January 2025 on the 2030 grid. Even with every wind and solar target met, a gap of up to 51 GW opened up, and the batteries were empty before the week began."

featureImage = "feature.png"
featured = false

tags = ["Python", "Energy", "Clean Power 2030", "Streamlit", "Data Analysis", "Risk"]
categories = ["Projects"]

showAuthor = true
showTaxonomies = true
showReadingTime = true
showTableOfContents = true
showDateUpdated = true
+++

{{< lead >}}
The UK plans to run its electricity system almost entirely on clean power by 2030, mostly wind and solar. I wanted to know what happens on the worst week of the year, when it's cold, dark and still. So I built a model that replays real 2025 weather on the grid planned for 2030, and measures the gap.
{{< /lead >}}

You can [try the dashboard](https://neso-flexibility-gap-ngtgqkkhevm2uu4sa4ew2k.streamlit.app/) in your browser, and the code is [on GitHub](https://github.com/suni639/NESO-flexibility-gap).

## The problem

**Clean Power 2030** is the government's plan for Great Britain's electricity. The target is for clean sources to produce at least 95% of generation in a typical weather year. To get there, the [Clean Power 2030 Action Plan](https://www.gov.uk/government/publications/clean-power-2030-action-plan/clean-power-2030-action-plan-a-new-era-of-clean-electricity-main-report) aims for 43 to 50 GW of offshore wind, 27 to 29 GW of onshore wind, 45 to 47 GW of solar and 23 to 27 GW of batteries.

Two units come up throughout this piece:

- **GW (gigawatts)** measures power: how much electricity is needed at one moment. National demand peaked at about 46 GW in 2025, and NESO expects around 62 GW by 2030 as heating and transport switch to electricity.
- **GWh and TWh (gigawatt-hours, terawatt-hours)** measure energy: how much is used over time, or how much a battery holds. A terawatt-hour is a thousand gigawatt-hours.

The phrase "typical weather year" matters, because the risk sits in the weeks that aren't typical. Germans have a word for them, **Dunkelflaute**, the "dark doldrums": days on end of little wind and little sun, usually in winter, when cold weather pushes demand up. A big high-pressure system can sit over the whole North Sea, so our neighbours often have the same problem at the same time.

So this project asks one question: **if every wind and solar target is met, how much power is missing in the worst week, and can batteries cover it?**

---

## How it works

{{< mermaid >}}
graph TD
    A[2025 weather<br>half-hourly] --> B[Scale to 2030<br>demand and capacity]
    B --> C[Find the<br>worst 7 days]
    C --> D[Run batteries<br>half-hour by half-hour]
    D --> E((What's<br>left over))
{{< /mermaid >}}

1. **Take a real year of weather.** NESO, the National Energy System Operator, publishes demand and wind and solar output for every half-hour. I used all of 2025, about 17,500 half-hours, and turned the wind and solar figures into **load factors**: how hard the turbines and panels were working compared with their maximum.
2. **Scale it to 2030.** Demand is scaled up to NESO's projected 2030 peak of 62 GW, from its Future Energy Scenarios (the Holistic Transition pathway). Wind and solar are set to the top of the Clean Power 2030 ranges: 50 GW offshore, 29 GW onshore, 47 GW solar. Nuclear is a flat 4 GW.
3. **Find the worst week.** The model looks for the seven days with the biggest total shortfall between demand and clean generation.
4. **Run the batteries.** Every half-hour, spare power charges the batteries and shortfalls drain them, with realistic limits on how fast they charge and how much they hold. The default fleet is 30 GW that can run for four hours, slightly above the plan's range.
5. **Measure what's left.** Whatever the batteries can't cover is the gap. In real life, gas plants, imports or people using less electricity would have to fill it.

I built it in Python with an interactive Streamlit dashboard, so anyone can change the battery size or wind target and watch the gap move.

---

## The worst week: 16 to 23 January 2025

The model picked out the week from Thursday 16 January, and it gets worse as it goes on. Wind began fading on the Friday night. By the following Wednesday the whole of Britain's wind fleet, scaled to 2030 size, was producing only a few per cent of its maximum for hours at a time.

{{< figure src="feature.png" alt="Stacked area chart of the week of 16 to 23 January 2025 on the 2030 grid. Nuclear is a thin green band at 4 GW, wind and solar shrink from about 35 GW to under 5 GW, batteries appear briefly on the Friday evening, and a red unmet gap fills most of the space up to a demand line that peaks near 59 GW each evening" caption="The 2030 grid in the worst week of 2025 weather. Red is the demand that wind, solar, nuclear and batteries can't meet." >}}

These were the main results for that week.

| Measure | Result |
|---|---|
| Biggest gap | **51 GW**, at 17:00 on Wednesday 22 January |
| Demand at that moment | 58 GW |
| Average gap across the week | 29 GW |
| Energy missing across the week | **4.8 TWh** |
| Battery contribution | A few hours on the Friday evening, then nothing |

The 51 GW figure is not a forecast of blackouts. The model deliberately leaves out gas, imports, hydro and biomass, because the point is to measure how much of the job they would have to do. What it means in practice is that on that Wednesday evening, almost all of Britain's electricity would have to come from somewhere other than wind, solar and nuclear.

---

## Why batteries don't fix it

I'd assumed the batteries would shrink the gap a lot, but they barely touch it.

The batteries were already empty when the week began. They last held any charge on the Tuesday, two days earlier, and had spent it on the evening peaks. On the Friday afternoon some spare wind topped them up, they spent it on the Friday evening peak, and for the remaining five and a half days there was never any spare power to recharge them.

So I tried a more generous test, with the fleet starting the week completely full.

{{< figure src="battery-endurance.png" alt="Bar chart of how many hours a fully charged battery fleet lasts in the event: 6 hours for 30 GW with 4-hour batteries, 29 hours for 30 GW with 12-hour batteries, 31 hours for 100 GW with 4-hour batteries, against a 168-hour event" caption="Even starting full, the best case covers about a day of a seven-day event." >}}

A full 30 GW, four-hour fleet holds 120 GWh. Against a week that is 4.8 TWh short, that's about 2.5% of what's needed, and it's gone in six hours. Tripling the fleet to 100 GW, or making every battery last 12 hours, buys about a day.

That was the main finding for me. Batteries are very good at moving power from a windy afternoon to the evening peak, but they can't carry the country through a week without wind, because there's nothing to charge them from.

Building more wind doesn't solve it either. Raising offshore wind from 50 to 60 GW cut the peak gap by less than half a gigawatt, because extra turbines produce very little in a calm.

---

## What could fill the gap

Over the whole year, the same 2030 grid produced about 55 TWh more clean power than it needed, mostly on windy days, and much of it with nowhere to go. That's more than ten times the energy missing in the worst week. The shortage comes down to timing, since the spare power arrives weeks or months before the still week and there's no way yet to hold on to it until then.

That points to a few options, each with a catch.

| Option | Rough size | The catch |
|---|---|---|
| Imports through interconnectors | 10 to 15 GW | Neighbours may be in the same calm, cold weather |
| Demand response | 5 to 10 GW | Paying industry and households to use less; untested at this scale |
| Long-duration storage | Weeks of supply | Hydrogen in salt caverns is the leading idea; not yet built at scale |
| Backup power stations | 30 GW or more | Gas, ideally with carbon capture or switched to hydrogen later |

*These sizes are my rough working assumptions, not official figures.*

The government's own plan reaches a similar answer. It expects to keep around 35 GW of unabated gas power stations in reserve for security of supply. And the Royal Society's [large-scale electricity storage study](https://royalsociety.org/news-resources/projects/low-carbon-energy-programme/large-scale-electricity-storage/executive-summary/), which modelled 37 years of real weather, concluded that a 2050 Britain running mainly on wind and solar would need around 60 to 100 TWh of hydrogen storage. Both are planning for weeks like the one in the chart above.

---

## What the model doesn't do

This is a simple model, so it's worth being clear about where it's rough.

- **The wind pattern is a stand-in.** NESO's half-hourly data gives load factors for smaller wind farms connected to local networks. I used that pattern for the whole 2030 fleet, including offshore, which tends to be windier and steadier. The real gap could be somewhat smaller.
- **One year of weather.** 2025 had a severe week, but it may not be the worst on record, which is why the Royal Society used 37 years of data.
- **Britain as an island.** Imports are left out on purpose, as a stress test. In practice they would help, unless the neighbours are short too.
- **A "copper plate" grid.** The model assumes power flows freely from Scotland to London. In reality the cables between them fill up, which would make things worse. That bottleneck is the subject of my [Storm Jocelyn project](/projects/storm-jocelyn-wind-constraints/).

---

## Key takeaways

1. **Batteries cover hours, and a still winter week needs more.** Getting through it takes storage that lasts for weeks, or power stations that can run whatever the weather.
2. **There's enough clean power over a year.** The 2030 grid makes more than it needs across twelve months, but too little of it arrives in the week that matters.
3. **A typical-year target can hide a bad week.** The 95% goal can be met while the system is still exposed to unusual weather.
4. **A simple model was enough to see the problem.** A few hundred lines of Python and public data showed why the official plan keeps 35 GW of gas in reserve.

The obvious next step would be to rerun the model with more years of weather and with imports included, to see how much of the gap remains in a more realistic setup.
