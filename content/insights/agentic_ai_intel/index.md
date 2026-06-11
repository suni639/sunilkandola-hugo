---
title: "Ditching the Dumb Robot: Why I Rebuilt My Digital Asset Intel Tracker with Agentic AI"
date: 2026-06-11
description: "An insight into upgrading my weekly digital asset tracker from a monolithic automation script to a collaborative multi-agent AI system."
series: ["Building the Hub"]
series_order: 2
tags: ["Hugo", "Engineering", "AI", "Agentic"]
---

Tracking wholesale digital asset infrastructure is mostly an exercise in filtering. The genuine structural developments—like central bank digital currencies and tokenised bank deposits—are constantly drowned out by speculative retail noise. I initially built a standard Python script to automate my weekly tracking, which worked fine, but relied on a single monolithic Gemini API call to do all the heavy lifting. To get higher quality results, I wanted something smarter than just throwing raw data at a single prompt. Here is how I rebuilt the pipeline into a team of collaborative AI agents.

---

### The Monolithic Setup: Single-Prompt Limits

My original script was reliable, but structured linearly. Every Monday, it scraped RSS feeds, bundled the raw text together, and fed it to a large language model in one giant prompt to produce the briefing. 

While the script ran successfully, relying on a single API call introduced clear limits:
* **Prompt Overload**: Asking one prompt to parse, filter out retail hype, translate terms, and write a structured briefing meant the model had to multitask too much, occasionally watering down the analysis.
* **No Room for Research**: If a harvested news snippet was too dry, the system had no way to pause, search for more context, and enrich the details.
* **Rigid Filtering**: Without a multi-stage reasoning process, separating structural banking plumbing from retail crypto noise (like spot price changes and memecoin gossip) required highly complex prompting that was difficult to tune.

I wanted a system that behaved less like a text parser and more like a modular research team.

---

### The Upgrade: Specialised AI Agents

The new setup splits the workload into a collaborative team of agents, each handling a distinct step in the reasoning chain:

{{< mermaid >}}
graph TD
    A[Research Scout] -->|raw_enriched_feed.json| B[Macro Strategist]
    B -->|strategic_analysis.json| C[Chief Editor & QC]
    C -->|output_briefing.md| D[Publisher]
    D -->|Gmail & Hugo| E((External Delivery))
{{< /mermaid >}}

* **The Research Scout (`scout`)**: Scrapes CoinDesk, The Block, the Bank for International Settlements (BIS), Ledger Insights, the European Central Bank (ECB), and the Bank of England (BoE). If it spots a terse summary of a major project (like Project Agorá or mBridge), it autonomously queries search engines to find the underlying technical specifications and enriches the data.
* **The Macro Strategist (`strategist`)**: Acts as the Noise Gate. It evaluates every article against strict wholesale market filters, immediately discarding spot price changes, memecoin news, or retail wallet updates, and categorising institutional developments.
* **The Chief Editor & QC (`editor`)**: Compiles the briefing. It runs a self-correction loop to eliminate generic AI phrases (e.g. *appears to*, *it is evident that*) and enforces strict British English spelling (*tokenised*, *utilising*, *programmes*).
* **The Publisher (`publisher`)**: Converts the markdown into a styled HTML newsletter, emails it to my inbox, prepends website metadata, and commits/pushes the post to my Hugo-based website repository.

---

### Why Agentic is Better

Standard automation follows a single linear path; agentic systems follow objectives. By separating the workflow into distinct cognitive stages, each step is optimised. The Strategist's focus is purely on filtering, the Editor's focus is on phrasing and QC, and the Scout handles research. The final briefing is significantly sharper, completely free from price speculation, and reads like a professional analyst note.
