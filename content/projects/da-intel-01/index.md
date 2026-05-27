+++
title = "Automated Digital Asset Intelligence"
date = 2026-05-27
draft = false
description = "Pipeline that scrapes, filters, AI-synthesises, and publishes a weekly digital asset intelligence briefing."
summary = "An automated pipeline that ingests news from six institutional sources, applies an AI noise gate to strip retail speculation, synthesises the signal into a structured briefing with Gemini, and publishes it to this site and by email — every Friday, with no manual steps."

tags = ["Python", "AI", "Automation", "Hugo", "Digital Assets", "Gemini"]
categories = ["Projects"]

showAuthor = true
showTaxonomies = true
showReadingTime = true
+++

{{< lead >}}
The digital assets space generates enormous volume. Most of it is noise. This project builds a pipeline that automatically finds the signal, interprets it, and publishes a structured briefing — weekly, without human intervention.
{{< /lead >}}

## Problem

The institutional digital asset landscape — CBDCs, RWA tokenisation, wholesale settlement infrastructure, regulatory frameworks — moves fast. Tracking it properly means reading across a dozen sources daily: BIS publications, Ledger Insights, Finextra, Blockworks, The Block, Atlantic Council CBDC tracker updates. The problem isn't access to information. It's the ratio of signal to noise.

The same feeds that carry a substantive ECB paper on Project Pontes also carry token price pumps, retail exchange listings, and speculative commentary. Reading everything to find the relevant 15% is a poor use of time. And the institutional framing — systemic impact, infrastructure alignment, regulatory motivation — rarely comes pre-applied.

The gap I wanted to close: **a briefing that reads like it was written by a sharp analyst who only cares about institutional plumbing, delivered automatically every Friday morning.**

---

## Solution

The system — DA-INTEL-01 — is a single Python pipeline (`weekly_run.py`) that runs on a Windows Task Scheduler job every Friday at 09:00. It covers five distinct stages:

### 1. Ingestion

The pipeline scrapes six sources in parallel:

- **Ledger Insights** (HTML scrape of the news archive — articles are extracted by parsing `<article>` tags and pulling the full body via a secondary request)
- **Finextra, Blockworks, The Block** (RSS/XML feed parsing via BeautifulSoup with lenient `html.parser` to handle malformed XML gracefully)
- **BIS Research RSS** (structured XML of working papers and speeches)
- **Atlantic Council CBDC Tracker** (HTML scrape of the live tracker page, augmented with hardcoded summary data for the CBDC country count and project status)

Each article is deduplicated against a persisted JSON state file (`system_state.json`) that tracks previously processed URLs. Articles already seen are skipped entirely — so the briefing only ever contains what's new this week.

One operational note: The Block's detail pages return 403 Forbidden errors. The fallback is to use the RSS `<description>` content, which is typically a useful 200-300 word summary. Good enough.

### 2. The Noise Gate

Before any AI is involved, a filtering prompt is passed to the model with a strict binary classification:

**Keep** — tokenised commercial bank liabilities, DvP and atomic settlement, RWA tokenisation (bonds, funds, private credit), wholesale CBDC infrastructure, MiCA/SAB 121/UK Digital Assets Bill updates, post-trade utilities (DTCC, Euroclear, Swift trials).

**Discard** — spot price movements, retail exchange listings, consumer wallet integrations, DeFi yield commentary, NFTs, memecoins, influencer sentiment, prediction markets.

The model doesn't summarise everything. It filters first, then writes only about what passed.

### 3. AI Synthesis (Gemini 2.5 Flash)

The filtered articles are passed to Gemini 2.5 Flash with a structured prompt that enforces a specific output schema:

```
## 1. MACRO VIEW
[5-7 punchy bullets — bold declarative headline, 1-2 sentences of supporting fact]

## 2. CORE PILLAR DEVELOPMENTS
[Banking rails / RWAs / Sovereign CBDCs / Regulatory frameworks]

## 3. STRUCTURAL & OPERATIONAL PAIN POINTS
[Interoperability / Liquidity friction / Post-trade constraints]

## 4. NEW HIGH-SIGNAL TARGETS FOR TRACKING
[3-5 new projects or legislation worth adding to keyword filters]
```

The prompt explicitly bans hedge language (*"appears to", "it is evident that", "underscored by"*) and requires British English throughout. Inline markdown links to source URLs are embedded at point of mention rather than relegated to footnotes.

### 4. Dual Publishing

The output is published in two forms simultaneously:

**Website post**: The briefing is written to `content/intel/` in the Hugo site repository as a markdown file with YAML front matter. The date is stored as explicit UTC (`2026-05-27T09:00:00+00:00`) to prevent Hugo's `buildFuture = false` config from silently excluding it. A `git add / commit / push` sequence runs automatically — triggering a GitHub Actions workflow that builds the Hugo site with the Blowfish theme (including submodule initialisation) and deploys to Cloudflare Pages.

**Email newsletter**: The same markdown is converted to styled HTML and sent to my inbox using Python's `markdown` library and a handwritten CSS template. It's delivered via Gmail SMTP with 3 retry attempts spaced 15 minutes apart. The email and site publish are independent — if SMTP fails, the site still gets the briefing.

### 5. State Management

After a successful run, processed article URLs are appended to `system_state.json`. This prevents duplicate coverage across weeks. Errors are written to `error_log.txt` with timestamps, so failures are diagnosable without needing to watch the terminal.

---

## Outcome

The pipeline runs end-to-end without human intervention. A Friday morning produces:

- A new post at [sunilkandola.com/intel/](/intel/) within ~3 minutes of the scheduled run completing (see tab on the top navigation menu)
- An HTML newsletter in the inbox, structured identically to the site post
- A state file updated with the week's processed URLs

The briefing format has been deliberately engineered for scanning. The **Macro View** is a set of bold-led bullets: the headline tells you what happened, the supporting sentence tells you why it matters. You can read the section in 60 seconds and know whether the rest is worth your time.

The system currently tracks approximately 150–200 articles per week across the six sources and typically passes 20–35% through the noise gate for synthesis.

---

## Learnings

**Prompt schema discipline matters more than prompt length.** Early versions of the Gemini prompt described the desired output in prose. The model produced prose back — long, hedged, and structured however it felt appropriate. Switching to an explicit markdown schema with labelled sections and example bullets eliminated most of the variance. If you want consistent output, show the model exactly what consistent looks like.

**Naive timestamps are a silent failure mode in static site generators.** Hugo excludes future-dated content by default and does it without errors or warnings — the file simply doesn't appear. Python's `datetime.now()` produces a naive local timestamp that Hugo interprets as UTC. Running at 09:00 BST (UTC+1) means the timestamp reads 09:00 UTC — one hour in the future. Fixed by using `datetime.utcnow().strftime('%Y-%m-%dT%H:%M:%S+00:00')` to produce an explicit UTC timestamp.

**Decouple publish from delivery.** The original design gated the Git push on successful email delivery. If SMTP failed (transient network, app password expiry), the site also didn't update. These are independent concerns and should be independent code paths.

**Invalid icon names in Hugo themes fail silently at the CDN.** The Blowfish theme uses Font Awesome icon names in menu config. An unsupported name (`brain`) doesn't throw a build error — it just renders nothing and, in this case, appeared to cause Cloudflare Pages to serve a stale cached build. The fix was trivial once diagnosed; finding it required checking the live HTML against the deployed config.

**Git submodule initialisation is a Cloudflare Pages footgun.** Cloudflare Pages' auto-build doesn't reliably initialise git submodules (required here for the Blowfish theme). The solution was a GitHub Actions workflow using `actions/checkout@v4` with `submodules: recursive`, which guarantees correct submodule state on every build regardless of platform behaviour.

---

## Stack

| Component | Technology |
|---|---|
| Ingestion & pipeline | Python 3 (`requests`, `BeautifulSoup4`) |
| AI synthesis | Google Gemini 2.5 Flash (`google-generativeai`) |
| Email delivery | Gmail SMTP via `smtplib` |
| Site | Hugo + Blowfish theme |
| CI/CD | GitHub Actions → Cloudflare Pages |
| Scheduling | Windows Task Scheduler |
| Secrets management | Windows Registry (`HKCU:\Environment`) |
