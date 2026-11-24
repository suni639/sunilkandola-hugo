+++
title = "Paid to Stop: What Storm Jocelyn Showed About Britain's Grid"
date = 2026-02-11
lastmod = 2026-11-22
draft = false
description = "Using Elexon's balancing data to measure how much wind power Britain paid to switch off during Storm Jocelyn on 24 January 2024, and what replacing it cost."
summary = "On a stormy day in January 2024, Britain paid Scottish wind farms about £1.9 million to switch off, then paid gas power stations to make up the difference. I used the grid's own records to measure it, and got it wrong the first time."

featureImage = "feature.png"
featured = false

tags = ["Python", "Energy", "Wind", "Elexon", "Streamlit", "Data Analysis"]
categories = ["Projects"]

showAuthor = true
showTaxonomies = true
showReadingTime = true
showTableOfContents = true
showDateUpdated = true
+++

{{< lead >}}
On 24 January 2024, Storm Jocelyn brought gale-force winds across the north of the UK. Scotland's wind farms could have produced far more power than the grid could carry south, so Britain paid them to switch off. I used the grid's own records to measure how much, and what it cost.
{{< /lead >}}

This is the companion to [When the Wind Stops](/projects/cp30-stress-test/), which looked at the opposite problem: a week with almost no wind. The code is [on GitHub](https://github.com/suni639/Wind-Constraint-Tracker).

## The problem

Britain's best wind is in Scotland, and much of its demand is in England, especially the south. The power has to travel between them on a limited number of high-voltage lines. Engineers call the main pinch point on the Scotland–England border the **B6 boundary**.

When it's very windy, Scottish wind farms can generate more than those lines can carry. The grid can't let the cables overload, so the system operator, NESO, does two things:

- **Pays wind farms to turn down.** This is called **curtailment**. The wind farm loses the income it would have earned, so it's compensated.
- **Pays other power stations to turn up**, usually gas plants closer to where the demand is, to replace the power that couldn't get through.

Both payments are called **constraint costs**, and they end up on everyone's energy bills. Across January 2021 to April 2023, [Carbon Tracker estimated](https://carbontracker.org/britain-wastes-enough-wind-generation-to-power-1-million-homes/) that Britain spent £1.5 billion curtailing wind.

So I picked one stormy day and asked: **how much wind was switched off, and what did the grid pay for it?**

---

## How it works

Every instruction NESO gives a power station to change its output goes through the **Balancing Mechanism**, and Elexon publishes the results through a free public API.

{{< mermaid >}}
graph TD
    A[Elexon API<br>one day of data] --> B[Label each unit<br>by fuel type]
    B --> C{Wind farm<br>turned down?}
    C -->|Yes| D[Add up volume<br>and payments]
    C -->|No| X[Set aside]
    D --> E((Dashboard<br>and charts))
{{< /mermaid >}}

1. **Download the day.** The pipeline pulls every accepted instruction for 24 January 2024, half-hour by half-hour, with the volume each unit was asked to change and the money that changed hands.
2. **Work out who's who.** The data identifies power stations by codes like `T_SGRWO-6`. A register of these units maps each code to its fuel type, so wind farms can be separated from gas, hydro and the rest.
3. **Count the turn-downs.** In the Balancing Mechanism, an instruction to generate less is a **bid**. Adding up the accepted bids from wind farms gives the energy that was switched off.
4. **Count the cost.** Elexon also publishes what each accepted instruction was worth, which gives the payments to wind farms and to the gas plants that were turned up.

---

## What happened on 24 January

Storm Jocelyn arrived on 23 January, two days after Storm Isha, and was at its worst overnight into the 24th. Over that day, NESO held back wind almost around the clock.

{{< figure src="feature.png" alt="Bar chart of wind power held back on 24 January 2024 by half-hour, around 2 GW overnight, dipping to under 1 GW in the early morning, back to about 2 GW through the middle of the day, then tailing off to almost nothing by 23:00" caption="Wind farms paid to turn down on 24 January 2024, from Elexon's settlement data." >}}

| Measure | Result |
|---|---|
| Wind energy switched off | **32,600 MWh** |
| Equivalent to | The yearly electricity use of about 12,000 typical homes |
| Wind farms involved | 30 |
| Where | At least nine-tenths from Scottish wind farms, led by Seagreen and Moray East |
| Paid to wind farms to turn down | **About £1.9 million** |
| Paid to gas power stations to turn up | About £2.7 million |

*Homes use Ofgem's typical figure of 2,700 kWh of electricity a year. Payments are Elexon's indicative figures.*

At its peak, more than 2 GW of wind was being held back at once.

{{< figure src="paying-twice.png" alt="Bar chart comparing £1.9 million paid to wind farms to turn down with £2.7 million paid to gas power stations to turn up on 24 January 2024" caption="Two sides of the same day. Not all of the gas payments were caused by the storm's constraints, so treat the gas figure as an upper bound." >}}

The chart shows the odd position the grid was in. On a day of gale-force winds, it paid to switch off clean power in the north and paid again to burn gas in the south. Jocelyn wasn't an unusual case either, since Storm Isha two days earlier led to even more wind being switched off on 22 January: about 42,400 MWh.

---

## What I got wrong first time

The first version of this project reported about 170,000 MWh switched off and an estimated cost of £12 million, which was wrong by roughly five times.

The mistake was in step 1, where I'd downloaded the wrong dataset. Instead of the instructions NESO actually *accepted*, I'd pulled the **bid-offer pairs**, which are the prices and volumes every power station *offers* the grid before anything is decided. My filter for "turn-downs" found nothing in them, so the code fell back to counting every wind farm's offers and added them all up. Some of the biggest "turn-downs" in that version came from Hornsea One, an offshore wind farm off Yorkshire that sits south of the bottleneck, and that should have made me look harder.

I only spotted it in November 2026, when I re-ran the analysis for this write-up and checked it against Elexon's settled volumes. The fix was to use the accepted volumes and the actual payments, which is what the figures above come from.

Some of the original work did hold up. Thousands of megawatt-hours of wind really were switched off in a day, mostly in Scotland, and my assumption of about £70 for each megawatt-hour wasn't far out, as the real payments to wind farms worked out at about £60.

---

## What would fix it

There are three broad answers, and the government has made a choice between them.

| Option | What it means | Where it stands |
|---|---|---|
| Build more lines | New links between Scotland and England, including subsea cables down the east coast | Under way, but large projects take years |
| Use the power where it's made | Put demand, like data centres, electrolysers for hydrogen or storage, near the wind | Happening slowly |
| Change the price signal | **Zonal pricing**: cheaper power in Scotland on windy days, so demand moves there | Ruled out by the government in July 2025 |

I originally built this project as evidence for zonal pricing. In its [July 2025 decision](https://www.gov.uk/government/publications/review-of-electricity-market-arrangements-rema-summer-update-2025/review-of-electricity-market-arrangements-rema-summer-update-2025-accessible-webpage) on electricity market reform, the government kept a single national price. Instead it chose "reformed national pricing", including a national plan for where generation and demand should be built. Constraint costs still have to be dealt with under that choice, so they'll be managed through planning and grid building rather than through prices.

---

## What the analysis doesn't do

- **One day.** It looks at 24 January only. A full winter would show the scale much better.
- **Not every payment is a constraint.** The Balancing Mechanism is also used to match supply and demand nationally. The wind turn-downs on a day like this are almost all constraints, but some of the gas payments won't be.
- **The fuel register is incomplete.** Some newer wind farms, such as Moray East, weren't labelled as wind in the register I used, so I matched them by name. A few small units may still be missed.
- **Indicative payments.** The money figures are Elexon's early settlement figures, which can be revised.

---

## Key takeaways

1. **The limit on 24 January was the grid.** About 32,600 MWh of Scottish wind had no way to reach the people who needed it.
2. **Consumers end up paying for both sides.** The bill covers switching the wind off and switching gas on somewhere else.
3. **Check what your data actually is.** My first answer was five times too big because I counted offers rather than instructions, and the numbers looked plausible enough that I didn't question them.
4. **With zonal pricing ruled out, planning carries more weight.** Building the lines and putting new demand in the right places will have to do the work that price signals might have done.

A full winter of data, rather than one day, would be the natural next step for this analysis.
