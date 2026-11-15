+++
title = "A Regulatory Early-Warning System for Digital Assets and Payments"
date = 2026-11-15
draft = false
description = "A scheduled Python pipeline that watches regulator feeds in the UK, EU, US and beyond, scores each publication for execution risk with Gemini, compares it against rules in other jurisdictions, and emails red alerts and a Monday digest."
summary = "Regulators publish something relevant to stablecoins, tokenisation or payments almost every week. I built a scanner that reads the feeds every two hours, scores what matters, compares it with the rules elsewhere, and only emails me when it's worth it."

featureImage = "feature.png"

tags = ["Python", "AI", "Automation", "Gemini", "RAG", "Regulation", "Digital Assets", "Stablecoins", "Payments", "Supabase"]
categories = ["Projects"]

showAuthor = true
showTaxonomies = true
showReadingTime = true
showTableOfContents = true
+++

{{< lead >}}
Regulators publish something that matters for stablecoins, tokenisation or payments almost every week, and the important items are buried among press releases about consumer contracts and scam warnings. I built a scanner that reads the feeds for me, scores each item, and checks how it compares with the rules in other countries.
{{< /lead >}}

## The problem

If you work anywhere near digital assets or payments, the rulebook is moving in several places at once. The Bank of England and FCA are building a UK stablecoin regime. The EU's MiCA is live and its supervisors are now using it. The US passed the GENIUS Act and its regulators are writing the detail. Global bodies like the BIS set standards that tend to reach UK and EU law a year or two later.

Keeping up means checking a dozen regulator websites, most of which publish far more noise than signal. And reading a new rule is only half the job. The question that matters is usually: *how does this compare with everyone else?* A stablecoin reserve set-up that satisfies one regulator can fail another. Those gaps are where cross-border payments and custody arrangements break.

My earlier [Digital Asset Intelligence](/projects/da-intel-01/) project produces a weekly news briefing. This one is narrower and stricter: primary regulator publications only, scored for risk, with a comparison across jurisdictions.

---

## How it works

Every two hours on weekdays, a GitHub Actions job runs the scanner. Each publication passes through four steps, and most fall away early.

{{< mermaid >}}
graph TD
    A[Regulator RSS feeds<br>BoE, PRA, FCA, ESMA, EBA,<br>OCC, Fed, BIS, FINMA] --> B{Keyword filter}
    B -->|Not about digital<br>assets or payments| X[Ignored]
    B --> C[Gemini scores<br>execution risk 0-100]
    C -->|Below 60| D[(Stored)]
    C -->|60 or above| E[Compare against UK, EU,<br>US and Singapore rules]
    E --> D
    D -->|85+ or high divergence| F((Instant red alert))
    D -->|Mondays| G((Weekly digest))
{{< /mermaid >}}

**1. A cheap filter first.** A plain keyword check throws out anything that isn't about stablecoins, tokenisation, custody, settlement, operational resilience and the like. It costs nothing, and it means the AI only sees items that might matter.

**2. A risk score.** Gemini reads each remaining item as a payments and digital asset risk analyst would, and gives it an *execution risk* score from 0 to 100. In plain terms, that's how directly, and how soon, the publication forces a bank to change how it moves, holds or settles money. Gemini also suggests two or three checks a second-line risk team should run, and writes a three-sentence summary in the style of a risk-committee paper.

**3. A comparison across jurisdictions.** For anything scoring 60 or more, the scanner searches a small library of rule summaries covering the UK, EU, US and Singapore. It does this by meaning rather than keywords, using embeddings stored in a Postgres database. Gemini then writes a four-column table showing where a workflow that passes in one place would fail in another. It also gives a second rating, *divergence*, from High through Medium and Low to Aligned. That measures how far apart the rules are, i.e. whether the same arrangement would pass in one country and fail in another.

**4. Email only when it's worth it.** The score decides what happens next:

| Score | Typically | What happens |
|---|---|---|
| 85–100 | Final rules affecting settlement, reserves, custody or capital | Instant red-alert email, and in the Monday digest |
| 60–84 | Consultations and speeches signalling a change | Cross-country comparison, then the Monday digest |
| Below 60 | Background or only loosely related | Stored, not emailed |

High divergence also triggers a red alert, even below 85: a gap between countries can matter more than the rule itself.

---

## What it found on day one

The first real run picked up 12 items from the last two months. Scores ranged from 25 to 88, and the order was about right. (The two red alerts were held back on this first run, for reasons explained below.)

| Score | Publication | Outcome |
|---|---|---|
| 88 | OCC final rule implementing the GENIUS Act | Red alert (high divergence) |
| 87 | EBA classifies USDC as the EU's first *significant* e-money token | Red alert (high divergence) |
| 72 | Bank of England approach to tokenised collateral in central clearing | Monday digest |
| 68 | FCA on preparing for quantum computing risk | Monday digest |
| 25 | FCA on fair consumer contracts | Stored only |

The score is a judgement, not a measurement. Running the same items twice moved some scores by a few points; USDC came out at 88, then 87. That's why the bands are wide, and why every email links to the source.

The USDC item shows what the comparison step adds. Being classed as significant under MiCA means the issuer must hold at least 60% of its reserves as deposits at commercial banks. The GENIUS Act lets US issuers back tokens mainly with short-dated Treasury bills and repo. Singapore and the UK proposals allow different mixes again. So one stablecoin can be compliant in the US and fall short in the EU, and the scanner put that in its headline.

---

## Lessons along the way

**Review the AI's first plan.** I started from a design another AI drafted. The idea was sound, but the code wouldn't have worked as written. Two of its ten feed addresses were dead. The comparison step searched a rules database that nothing ever filled, so Gemini would have quietly invented the other jurisdictions' positions. It also used an embedding model that had since been retired. None of this would have crashed loudly. It would just have produced confident nonsense.

**Ground the comparison, and show your working.** The prompt tells Gemini to use only the retrieved rule summaries, and to write "No baseline on file" rather than fill a gap from memory. Each email lists which rules it compared against. The comparison is only as good as that library, so the summaries carry a "checked on" date that I fill in as I verify each one against the source.

**Don't flood the inbox on day one.** The first run treats everything already in the feeds as new. So when the database is empty, the scanner stores and scores everything without sending a single alert.

**The free tier wasn't enough.** Gemini's free tier allowed 20 requests a day, and the first backfill used that up halfway through. The scanner now stops cleanly when the quota runs out and picks up the rest next time. Moving to a small prepaid credit removed the limit, and the cost at this volume is tiny.

**Check your environment variables.** After I moved to the paid tier, the free-tier errors carried on. The cause was an old API key from another project, saved in my Windows settings, which was quietly taking priority over this project's settings. Ten minutes of confusion, one line of code to fix.

---

## Technical stack

| Component | Technology |
|---|---|
| Pipeline | Python (`httpx`, `feedparser`, `pydantic`) |
| Scoring and comparison | Google Gemini, with schema-checked JSON output |
| Rule search | Gemini embeddings + pgvector on Supabase Postgres |
| Email | Gmail SMTP |
| Scheduling | GitHub Actions (every 2 hours on weekdays, plus a Monday digest) |

*The scores and comparisons are generated automatically and are a prompt to read the source, not a substitute for it.*
