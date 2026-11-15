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

1. **Collect.** Every two hours on weekdays, the scanner reads regulator feeds. In the UK that means the Bank of England, PRA, FCA and HM Treasury, including the FCA's consultation papers and policy statements, where many rules are first set out. Abroad it covers ESMA and the EBA in the EU; the OCC, Federal Reserve, SEC and FinCEN in the US; FINMA in Switzerland; and the global standard setters: the BIS, the Financial Stability Board and IOSCO.
2. **Filter.** A keyword screen keeps items on stablecoins, tokenisation, custody, settlement and operational resilience. Most publications stop here.
3. **Rate.** Each remaining item gets an impact score. Anything scoring 60 or more is also checked against a library of UK, EU, US and Singapore rules for divergence.
4. **Deliver.** Red alerts go out straight away. Every item scoring 60 or more, alerts included, appears in the Monday digest. Publications on the same story are grouped. When the SEC proposed new crypto custody rules, two commissioners published statements the same day, and all three became one entry with the statements linked underneath.

---

## How the scoring works

Each item can receive two ratings.

**Impact score (0 to 100).** How directly, and how soon, the publication would force a bank to change how it moves, holds or settles money.

| Score | Typical publication | Result |
|---|---|---|
| 85+ | Final rule or supervisory decision affecting settlement, reserves, custody or capital | Red alert the same day, then the Monday digest |
| 60–84 | Consultation, guidance or speech signalling a change | Divergence check, then the Monday digest |
| Under 60 | Research, background or loosely related news | Logged, no email |

**Divergence (High, Medium, Low or Aligned).** How far the same requirement differs across the UK, EU, US and Singapore. Only items scoring 60 or more are checked, in three steps.

1. **Find the comparable rules.** The scanner searches a library of about 30 rule summaries, each written from and checked against the regulator's own text, and takes the closest for each country. It also draws on recent UK publications it has already scored, so the UK position stays current. Rules on a different subject are set aside.
2. **Label each country.** Each country is judged on its own against the new publication and given one label:

| Label | Meaning |
|---|---|
| Conflict | One arrangement cannot comply with both, for example incompatible mandatory reserve rules |
| Adjustment | Both can be met, but only by changing processes, structures or thresholds |
| Minor | Differences in detail, timing or reporting only |
| Same rules | Materially the same requirement |
| No evidence | Nothing relevant on file for that country |

3. **Apply fixed rules.** Any Conflict makes divergence High, as long as the new publication is final. A proposal, speech or statement cannot create a binding conflict yet, so it tops out at Medium. Otherwise any Adjustment gives Medium, only Minor differences give Low, and Same rules everywhere gives Aligned.

The labels appear in each email's comparison table, so the reason for a rating is visible at a glance. Because the rating follows from the labels rather than an overall judgement, the same evidence gives the same answer: in testing, three repeat runs agreed on all ten items, against six out of ten before.

**When an alert fires.** An impact score of 85 or more, or High divergence: a conflict between a final rule and the rules elsewhere.

The impact score is still a judgement and can move by a point or two between runs, so the bands are deliberately wide and every email links to the original publication.

---

## Case study: USDC becomes a significant e-money token

On 14 October the European Banking Authority classified USDC as the first *significant* e-money token in the EU. This is how the scanner handled it.

**Collected.** The announcement appeared in the EBA's news feed.

**Filtered.** "E-money token" is on the keyword list, so the item went through.

**Impact score: 87.** A formal supervisory decision that changes how an issuer must hold its reserves sits in the top band. The scanner tagged it under reserves, custody, settlement and operational resilience.

**Divergence: Medium.** Significant status under MiCA requires at least 60% of reserves to be held as bank deposits. The UK's final stablecoin rules ask for at least 5% in on-demand deposits, and US and Singapore rules lean on short-dated government debt. The UK, US and Singapore were each labelled Adjustment: a single reserve could meet the EU rule and theirs, but only with changes. With no outright conflict, divergence is Medium, and the alert came from the impact score.

The OCC's final rule implementing the GENIUS Act shows the contrast. It was rated High, because the UK and EU columns are both Conflict: a reserve held mainly in US Treasuries cannot meet the Bank of England's requirement for at least 30% in central bank deposits, or MiCA's bank-deposit minimum.

**Delivered.** A red alert that opens with the two ratings and the areas affected, then sets out the impact, compares the rules across the four jurisdictions with each column's label, lists three checks for a second-line risk team and ends with a three-sentence summary ready for a risk committee paper.

{{< figure src="feature.png" alt="Red alert email for the EBA's USDC decision showing a score of 87 out of 100, medium divergence, and a comparison table whose columns are labelled Adjustment for the UK, US and Singapore and This publication for the EU" caption="The top of the red alert for the USDC decision, with each column's label." >}}

The same run looked at 291 feed entries, and 12 passed the filter. Two qualified as red alerts: this one and the OCC's final rule implementing the GENIUS Act. Three more scored between 60 and 84, including the Bank of England's approach to tokenised collateral in central clearing. The other seven were logged without an email.

### The Monday digest

By the following Monday, with the SEC, FinCEN, FSB and IOSCO feeds added, the digest held ten publications. It opens with one line on the week, *"8 signals this week (10 publications): 2 high priority and 6 to watch. 3 are UK developments."* The SEC's crypto custody proposal and two commissioners' statements on it count as one signal. The highest-scoring item follows in full, in the same format as an alert. The rest sit in two short tables, UK developments first and then international, each showing score, divergence and publication date. A UK consultation stays visible even in a week when a foreign rule scores higher, and the whole digest takes under a minute to read.

{{< figure src="digest.png" alt="Weekly digest email: a one-line summary of eight signals, the OCC GENIUS Act rule as the top signal with high divergence, its comparison table labelled Conflict for the UK and EU, This publication for the US and Adjustment for Singapore, and the start of the UK developments table" caption="An excerpt from the Monday digest. Dashed lines mark where longer sections are cut." >}}

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
