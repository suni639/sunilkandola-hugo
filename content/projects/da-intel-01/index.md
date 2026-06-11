+++
title = "Automated Digital Asset Intelligence"
date = 2026-05-27
draft = false
description = "A scheduled Python pipeline that runs a collaborative multi-agent AI system (Research Scout, Macro Strategist, and Chief Editor) to curate, enrich, filter, and compile weekly digital asset briefings."
summary = "An automated multi-agent AI system that curates, enriches, and filters institutional digital asset news, compiling and publishing a weekly briefing to email and this website."

tags = ["Python", "AI", "Automation", "Hugo", "Digital Assets", "Gemini", "Agentic AI", "Multi-Agent Systems", "Google GenAI"]
categories = ["Projects"]

showAuthor = true
showTaxonomies = true
showReadingTime = true
+++

{{< lead >}}
An automated Python pipeline that scrapes institutional sources, filters out retail cryptocurrency speculation, and publishes a weekly digital asset briefing every Monday.
{{< /lead >}}

## Problem

Tracking institutional digital assets—wholesale banking, CBDCs, tokenised assets, and regulatory frameworks—requires monitoring multiple sources: the BIS, Ledger Insights, Finextra, Blockworks, The Block, and the Atlantic Council. 

However, these feeds are heavily cluttered with retail cryptocurrency speculation, price fluctuations, exchange listings, and memecoin updates. Manually sorting through these feeds to isolate structural updates is time-consuming, and standard news sources rarely format these updates with an institutional plumbing focus.

---

## Solution

The system (`DA-INTEL-01`) operates as an automated, multi-agent AI pipeline (`pipeline/run_pipeline.py`) triggered by GitHub Actions every Monday at 07:00 UTC.

### System Architecture & Agent Graph

{{< mermaid >}}
graph TD
    A[Ingestion Layer] -->|raw_enriched_feed.json| B[Research Scout]
    B -->|Enriched data via web search| C[Macro Strategist]
    C -->|Filtered strategic_analysis.json| D[Chief Editor & QC]
    D -->|Compiled output_briefing.md| E[Publisher Layer]
    E -->|SMTP Newsletter| F((Gmail Inbox))
    E -->|Git Auto-Publish| G((Hugo Website))
{{< /mermaid >}}

The pipeline operates across five stages:

### 1. Ingestion
The script ingests data from six primary sources:
- **Ledger Insights**: Scrapes HTML article nodes.
- **Finextra, Blockworks, The Block**: Parses RSS feeds.
- **BIS Research**: Parses research paper and speech RSS feeds.
- **Atlantic Council**: Scrapes the CBDC tracker page.

To prevent duplicate runs, processed articles are tracked in `system_state.json`. If a URL has already been processed, it is skipped. When full-page scrapes are blocked by Cloudflare (such as on The Block), the script falls back to using the RSS feed's summary description. These 403 blocks are handled silently as warnings in a local `error_log.txt` instead of throwing fatal errors, keeping the GitHub runner console clean.

### 2. Filtering (The Noise Gate)
Before summarising, the content is filtered using Gemini to separate signal from noise:
- **Keep**: Tokenised deposits, wholesale settlement (DvP), RWA tokenisation, wholesale CBDCs, regulatory updates (MiCA, SAB 121), and post-trade utilities (DTCC, Swift trials).
- **Discard**: Price movements, retail exchange listings, consumer wallets, NFTs, memecoins, and prediction markets.

### 3. Synthesis
Filtered articles are synthesised using Gemini 2.5 Flash. The prompt enforces a structured markdown layout:
- **Macro View**: 5-7 bullet points with a bold summary followed by 1-2 supporting sentences.
- **Core Pillars**: Categorised updates (Banking rails, RWAs, CBDCs, Regulation).
- **Pain Points**: Infrastructure, liquidity, or settlement friction.
- **Targets**: New projects or regulations to track.

The briefing is written in British English with inline source links.

### 4. Publishing
The generated brief is distributed through two channels:
- **Hugo Website**: Saved to `content/intel/` as a markdown file with YAML front matter. The runner automatically commits and pushes this to the Hugo site repository, which triggers a deploy to Cloudflare Pages.
- **Email Newsletter**: Formatted into HTML using a styled template and sent via Gmail SMTP directly to my personal inbox. This serves a dual purpose: it acts as a passive health-check notification confirming the weekly automation pipeline ran successfully, and it delivers a readable summary to consume digital asset news on the go. If email delivery fails, it retries three times (10-second intervals in GitHub Actions, 15 minutes locally) but does not block the website build.

### 5. State Persistence
Upon successful execution, the runner updates `system_state.json` and commits/pushes it back to the main repository. This ensures state is preserved across ephemeral virtual machines so that subsequent runs have the updated list of processed URLs.

---

## Key Learnings

* **Structured Output Prompts**: Providing a markdown schema with clear rules and examples for the model prevents verbose or irregular output formats.
* **Timezone Handling**: Naive local timestamps cause Hugo to hide posts scheduled in the future (such as during BST/UTC+1 offsets). Using explicit UTC strings (`datetime.datetime.now(datetime.timezone.utc)`) ensures articles publish immediately.
* **Deprecation Warning Cleanup**: Replacing deprecated standard functions like `datetime.utcnow()` with modern, timezone-aware `datetime.datetime.now(datetime.timezone.utc)` prevents runtime warning logs from cluttering execution metrics.
* **Process Decoupling**: Separating email delivery from git publishing prevents transient SMTP errors from blocking the website deployment.
* **Submodule Checkouts**: Auto-build engines like Cloudflare Pages can fail to check out Hugo theme submodules. Configuring GitHub Actions to clone with `submodules: recursive` guarantees the theme builds correctly.

### Upgrading to Agentic AI

To learn more about why and how this project was migrated from a single monolithic script to a collaborative multi-agent system, check out the detailed insight post: [Ditching the Dumb Robot: Why I Rebuilt My Digital Asset Intel Tracker with Agentic AI](/insights/agentic_ai_intel/).

---

## Technical Stack

| Component | Technology |
|---|---|
| Ingestion & pipeline | Python 3 (`requests`, `BeautifulSoup4`) |
| AI synthesis | Google Gemini 2.5 Flash (`google-generativeai`) |
| Email delivery | Gmail SMTP via `smtplib` |
| Website | Hugo + Blowfish theme |
| Deployment & CI/CD | GitHub Actions & Cloudflare Pages |
| State Tracking | GitHub Repository Secrets & `system_state.json` |
