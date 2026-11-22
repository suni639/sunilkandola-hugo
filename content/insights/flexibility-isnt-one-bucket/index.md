---
title: "Flexibility Isn't One Bucket"
date: 2026-11-22
draft: false
description: "Why counting gigawatts of batteries says little about whether a clean grid can cope, and what my two energy projects showed about the different kinds of flexibility Britain needs."
summary: "Energy plans talk about 'flexibility' as if it were one number. After two projects on Britain's grid, I think it's at least three separate jobs, and we're building for the shortest of them much faster than the longest."
featureImage: "feature.png"
tags: ["Energy", "Clean Power 2030", "Storage", "Grid", "Opinion"]
showTableOfContents: true
---

{{< lead >}}
Energy plans tend to talk about "flexibility" as a single quantity, usually counted in gigawatts of batteries. After two projects on Britain's grid this year, I think it's at least three separate jobs that happen on different timescales and in different places, and we're building for the shortest of them much faster than the longest.
{{< /lead >}}

## Three different jobs

Flexibility means being able to match supply and demand when the wind and sun don't line up with when people need power. That covers a few quite different situations:

| The job | Timescale | What does it |
|---|---|---|
| Smoothing the evening peak | Minutes to hours | Batteries, and shifting when cars charge and homes heat |
| Getting through a still, cold week | Days to weeks | Long-duration storage, backup power stations, imports |
| Using power where it's generated | Whenever it's windy | Grid lines, or putting demand near the wind |

Each one needs different technology, and adding more of one does very little for the others. When a plan or a press release quotes gigawatts, it's mostly describing the first job.

---

## What my model showed about batteries

In [When the Wind Stops](/projects/cp30-stress-test/) I replayed the worst week of 2025 weather on the grid Britain plans to have by 2030. Wind and solar output dropped away for several days, and at the worst point demand was 51 GW higher than clean supply.

I had expected the batteries to make a real dent in that, and they barely registered. I gave the model a 30 GW fleet with four hours of storage, which is slightly more than the government plans. Even if it started the week completely full, it ran dry after about six hours, having covered around 2.5% of the energy that was missing. Tripling its size only got it to about a day.

Batteries are very good at what they're designed for, which is moving power from a windy afternoon into the evening. A week without wind is a different scale of problem, and a tank that holds a few hours can't stretch to it.

---

## Storm Jocelyn and the location problem

[Paid to Stop](/projects/storm-jocelyn-wind-constraints/) looked at the opposite kind of day. During Storm Jocelyn in January 2024, Scottish wind farms could generate far more than the lines to England could carry. The grid paid them about £1.9 million to switch off roughly 32,600 MWh, and then paid gas plants further south to replace it.

Extra batteries in the south would have done nothing about that. Storage in Scotland helps only if it can send the power south later, once the lines have room. This is the third job on the list, and it depends on building transmission lines or on putting new demand, such as data centres or hydrogen production, close to the wind farms.

---

## Plenty of clean power, at the wrong times

One number from the 2030 model changed how I think about all of this. Over the whole year, the grid produced about 55 TWh more clean electricity than it needed, mostly on windy days. The worst week was 4.8 TWh short, which is less than a tenth of that surplus.

{{< figure src="feature.png" alt="Bar chart on a log scale comparing a fully charged 2030-sized battery fleet at 0.12 TWh, the energy missing in the worst week at 4.8 TWh, the year's spare clean power at 55 TWh, and the Royal Society's estimate of 60 to 100 TWh of hydrogen storage needed by 2050" caption="Each gridline is ten times the last. A full battery fleet holds about a fortieth of what one bad week needs." >}}

So the problem in 2030 is unlikely to be the amount of clean electricity across a year. What's missing is a way to hold some of it from the windy weeks until the still ones. The Royal Society reached a similar conclusion in its [storage study](https://royalsociety.org/news-resources/projects/low-carbon-energy-programme/large-scale-electricity-storage/executive-summary/), which used 37 years of weather data and estimated that a 2050 Britain running mainly on wind and solar would need around 60 to 100 TWh of hydrogen storage. That's several hundred times what a 2030-sized battery fleet holds.

---

## Why the system copes anyway

The obvious reply is that there are already answers to the long gap. The government's Clean Power 2030 plan keeps around 35 GW of gas power stations in reserve, interconnectors bring in power from Europe, and industry and households can be paid to use less. My models are also simple on purpose, with one year of weather, no imports and a rough stand-in for how offshore wind behaves.

I agree with all of that, and keeping gas on standby is a sensible answer for 2030. It also shows where things stand. The short job is being handed to new, clean technology, while the long job falls back on gas because nothing clean is ready to take it on at that scale. Imports will help on some days, though the same calm, cold weather often sits over neighbouring countries at the same time.

There is some movement. In April 2025 Ofgem opened a [cap and floor scheme](https://ofgem.gov.uk/decision/long-duration-electricity-storage-cap-and-floor-application-window-1) for long-duration storage, which guarantees projects a minimum income in return for limiting how much they can earn. By September, 77 projects had passed its eligibility checks. It's an attempt to pay for the long job on its own terms, rather than hoping the market for short-term services will cover it.

---

## Reading the headlines

When I see an announcement about gigawatts of new batteries now, I ask two things. The first is how long it lasts, because gigawatts measure power at a moment, and getting through a still week takes storage measured in terawatt-hours. The second is where it sits, because power that can't reach the people using it doesn't help, however much of it there is.

The growth in batteries is good news and they're solving a real problem. The harder problem, keeping the lights on through a long calm spell, is still mostly in the hands of gas, and I'd like to see it get as much attention as the short one.
