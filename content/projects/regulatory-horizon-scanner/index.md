+++
title = "A Regulatory Early-Warning System for Digital Assets and Payments"
date = 2026-11-15
draft = false
description = "A scanner that reads regulator feeds across the UK, EU, US and beyond, rates each publication for impact and cross-border divergence, and emails red alerts and a Monday digest."
summary = "Regulators publish something that matters for stablecoins, tokenisation or payments most weeks. This scanner reads the feeds every two hours, rates each item, and emails me only the ones that need attention."

featureImage = "feature.png"
featured = true

tags = ["Python", "Automation", "Regulation", "Digital Assets", "Stablecoins", "Payments", "RAG", "Supabase"]
categories = ["Projects"]

showAuthor = true
showTaxonomies = true
showReadingTime = true
showTableOfContents = true
+++

{{< lead >}}
Regulators publish something that matters for stablecoins, tokenisation or payments most weeks, usually among consumer notices and newsletters. This scanner reads the feeds every two hours, rates each item, and emails me only the ones that need attention.
{{< /lead >}}

## Why I built it

Digital asset rules are being written in several places at once. The UK is consulting on stablecoins, the EU is applying MiCA, and the US is implementing the GENIUS Act. Global bodies such as the BIS set standards that tend to reach national rulebooks a year or two later.

Reading each publication is only half the work. The more useful question is how a new rule compares with everyone else's, because an arrangement that is compliant in one country can fall short in the next. Those gaps are where cross-border payment and custody arrangements come under pressure.

---

## How it works

{{< mermaid >}}
graph TD
    A[Collect<br>regulator feeds] --> B{Filter<br>on topic?}
    B -->|No| X[Dropped]
    B -->|Yes| C[Rate<br>impact score]
    C -->|60+| D[Compare<br>UK, EU, US, SG]
    C -->|Below 60| L[(Logged)]
    D --> E((Red alert<br>or Monday digest))
{{< /mermaid >}}

1. **Collect.** Every two hours on weekdays, the scanner reads the feeds of the Bank of England, PRA, FCA, ESMA, EBA, OCC, Federal Reserve, BIS and FINMA.
2. **Filter.** A keyword screen keeps items on stablecoins, tokenisation, custody, settlement and operational resilience. Most publications stop here.
3. **Rate.** Each remaining item gets an impact score. Anything scoring 60 or more is also checked against a library of UK, EU, US and Singapore rules for divergence.
4. **Deliver.** Red alerts go out straight away. Other items scoring 60 or more wait for the Monday digest.

---

## How the scoring works

Each item can receive two ratings.

**Impact score (0 to 100).** How directly, and how soon, the publication would force a bank to change how it moves, holds or settles money.

| Score | Typical publication | Result |
|---|---|---|
| 85+ | Final rule or supervisory decision affecting settlement, reserves, custody or capital | Red alert the same day |
| 60–84 | Consultation, guidance or speech signalling a change | Divergence check, then the Monday digest |
| Under 60 | Research, background or loosely related news | Logged, no email |

**Divergence (High, Medium, Low or Aligned).** How far the same requirement differs across the UK, EU, US and Singapore. High means an arrangement that passes in one country would fail in another. Only items scoring 60 or more are checked.

**When an alert fires.** An impact score of 85 or more, or High divergence at any score from 60 up. A large gap between countries can matter as much as the rule itself.

The ratings are judgements. Rerunning the same item can move its score by a point or two, so the bands are deliberately wide and every email links to the original publication.

---

## Case study: USDC becomes a significant e-money token

On 14 October the European Banking Authority classified USDC as the first *significant* e-money token in the EU. This is how the scanner handled it.

**Collected.** The announcement appeared in the EBA's news feed.

**Filtered.** "E-money token" is on the keyword list, so the item went through.

**Impact score: 88.** A formal supervisory decision that changes how an issuer must hold its reserves sits in the top band. The scanner tagged it under reserves, custody, settlement and operational resilience.

**Divergence: High.** Significant status under MiCA requires at least 60% of reserves to be held as deposits at commercial banks. The US GENIUS Act allows a stablecoin to be backed mainly by short-dated Treasury bills and repo. Singapore and the UK proposals each use a different mix again. A reserve structure built for the US would not meet the EU requirement.

**Delivered.** A red alert with the score, a four-country comparison table, three checks for a second-line risk team and a three-sentence summary written for a risk committee paper.

{{< figure src="feature.png" alt="Red alert email showing a score of 88 out of 100, high divergence, and the start of a UK, EU, US and Singapore comparison table for the EBA's USDC decision" caption="The top of the red alert for the USDC decision." >}}

The same run looked at 291 feed entries, and 12 passed the filter. Two qualified as red alerts: this one and the OCC's final rule implementing the GENIUS Act. Three went to the Monday digest, including the Bank of England's approach to tokenised collateral in central clearing. The other seven were logged without an email.

---

## Under the hood

| Component | Technology |
|---|---|
| Pipeline | Python |
| Ratings and comparison | Google Gemini, with a structured output format |
| Rule library and search | Postgres with pgvector, hosted on Supabase |
| Email | Gmail |
| Scheduling | GitHub Actions: every two hours on weekdays, digest on Mondays at 07:30 UTC |

*Ratings are generated automatically. Each email links to the source publication, which remains the authority.*
