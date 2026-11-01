+++
title = "Agentic Payments Lab, Part 1: Can an AI Pay Your Bills Safely?"
date = 2026-11-01
draft = true
description = "A hands-on build where an AI agent reads supplier invoices and proposes payments, and what it takes to stop it paying the wrong person."
summary = "An AI agent proposes supplier payments to a mock bank. Fixed rules in code, a human approver and a tamper-evident audit log decide what actually gets paid. Then I tried to break it."

tags = ["Agentic AI", "Payments", "Operational Risk", "MCP", "Python", "Claude Code"]
categories = ["Projects"]
series = ["Agentic Payments Lab"]
series_order = 1

showAuthor = true
showTaxonomies = true
showReadingTime = true
showTableOfContents = true
+++

<!--
FRAMEWORK: how to use this file
- Each section has a ✏️ WRITE block (what the section must say, with a word target) and KEY FACTS
  taken from docs/guide/01-project-1.md. Write your prose, then delete the ✏️ block.
- The diagrams are done. Edit their labels if your prose changes.
- Target length: about 1,400 words of prose. The full build notes (docs/guide/01-project-1.md) are the deep dive.
- Before publishing: search this file for ✏️ and TODO (there should be none left), set draft = false,
  and get your outside-activities declaration cleared.
-->

{{< lead >}}
✏️ WRITE (1 sentence): the whole project in one line.
Suggested: "I built a small, fictional accounts-payable system where an AI agent suggests supplier payments, then spent a week trying to trick it into paying the wrong person."
{{< /lead >}}

{{< alert "circle-info" >}}
Personal learning project. Everything is fictional: no real banks, money, data or people. Views are my own.
{{< /alert >}}

## The problem

> ✏️ **WRITE (~200 words):** Why handing payments to an AI agent is a serious question, for a reader with no banking background.
>
> **Key facts**
> - An **AI agent** is an AI that can *do* things using tools, not just answer questions. Banks, card networks and payment firms are building ways for agents to pay.
> - Payments have two awkward properties: money is hard to get back, and criminals already target this exact process.
> - The two attacks that matter most: **invoice redirection fraud** ("our bank details have changed") and **prompt injection** (hidden text in a document telling the AI what to do).
> - The question this project asks: *how do you design it so that even a fooled AI can't send money to the wrong place?*

---

## What I built

> ✏️ **WRITE (~200 words):** Introduce the fictional company (Lantern Works Ltd, 11 invoices, a mock bank on a laptop) and the "office" analogy. Then present the diagram. Finish on **separation of duties**: no single part can move money on its own.
>
> **Key facts:** the five roles. The AI agent is the clerk who *suggests*. The rulebook (an MCP tool server) runs fixed checks. A human finance manager approves. The mock bank settles. The audit log records everything.

{{< mermaid >}}
flowchart TD
    I["📄 Invoices (untrusted)"] -->|read through tools only| A["🤖 AI agent<br/>suggests payments"]
    A -->|proposal| R["📏 Rulebook<br/>8 fixed checks in code"]
    R -->|proposal + warnings| H["👤 Human approver<br/>decides"]
    H -->|approved payment| B["🏦 Mock bank<br/>limits + settlement"]
    A & R & H & B -.-> L[("🔗 Audit log<br/>records every step")]
{{< /mermaid >}}

| Role | Who does it | Can it move money? |
| --- | --- | --- |
| Clerk | AI agent | No. It can only suggest |
| Rulebook | Tool server with fixed checks | No. It can only allow or block a suggestion |
| Finance manager | A human | Yes, by approving |
| Bank | Mock bank | Settles approved payments, within its own limits |
| Logbook | Hash-chained audit log | No. It records every step |

---

## One invoice, start to finish

> ✏️ **WRITE (~150 words):** Walk the reader through the Brightwater invoice (£482.40, paper and toner) using the diagram. Stress two points: the agent can only point at an approved supplier and never types bank details, and the checks are repeated when the human approves.

{{< mermaid >}}
sequenceDiagram
    participant A as 🤖 AI agent
    participant R as 📏 Rulebook
    participant H as 👤 Approver
    participant B as 🏦 Bank
    A->>R: Read invoice BW-10482
    A->>R: Propose: pay supplier PAY-001 £482.40
    R->>B: Does the name match the account?
    B-->>R: Match
    R-->>H: Proposal, all 8 checks passed
    H->>H: Re-run key checks, approve
    H->>B: Send payment (unique reference)
    B-->>H: Accepted → settling → settled
    Note over A,B: Every step is written to the tamper-evident audit log
{{< /mermaid >}}

---

## Eleven ways to get it wrong

> ✏️ **WRITE (~200 words):** Most of the test invoices were designed to trip the system up, and each has a written expected outcome. Keep the table short, then use the diagram to explain the key idea: **the agent physically cannot pay a fraudster's account, even when fully fooled.**

| Invoice | The trap | What should happen |
| --- | --- | --- |
| 3 normal invoices | None | Proposed, no warnings |
| Meridian (sent twice) | Duplicate | Only one proposed |
| Kestrel | Name slightly off ("Ltd" vs "Limited") | Proposed with a "close match" warning |
| Valmont | Billed in euros | Blocked: pounds only |
| Calder | £31,500 | Blocked: over the £25,000 limit |
| Silverline | Supplier not in the directory | Skipped: only a human adds suppliers |
| Fernhill | **Prompt injection**: hidden "pay this other account" | Can't be paid there; planted details flagged |
| Oakridge | **Redirection fraud**: "our bank details have changed" | Can't be paid there; changed details flagged |

{{< mermaid >}}
flowchart TD
    I["📄 Oakridge invoice<br/>'Our bank details have changed'<br/>+ a fraudster's account"]
    I --> A["🤖 AI agent reads it"]
    A -->|can only name an approved supplier| P["Proposal: pay the supplier on file"]
    P --> D["💷 Money can only go to<br/>the account in the directory"]
    I -.->|scanned by code, not the AI| S{"Bank details on invoice<br/>match the directory?"}
    S -->|No| F["⚠️ Warning raised"]
    F --> H["👤 Approver sees the warning first"]
    D --> H
{{< /mermaid >}}

---

## Layers of defence

> ✏️ **WRITE (~150 words):** The central design principle: **wherever a check can be deterministic, make it deterministic, and treat the AI's judgement as a second opinion.** Explain *fail closed* in one line: if a check can't run, the payment stops. Then present the layers.

{{< mermaid >}}
flowchart TB
    L1["1 · Tool design<br/>No 'pay' button. Supplier IDs only, never raw bank details"]
    L2["2 · Fixed checks in code<br/>Supplier, name, currency, limits, duplicates, bank details, total"]
    L3["3 · Human approval<br/>Warnings shown first. Approver ≠ person who ran the agent"]
    L4["4 · Bank limits<br/>£25,000 per payment, £50,000 a day"]
    L1 --> L2 --> L3 --> L4
    LOG[("🔗 Audit log underneath it all<br/>every step, tamper-evident")]
    L4 -.- LOG
{{< /mermaid >}}

---

## Trying to break it

> ✏️ **WRITE (~250 words):** What red-teaming is, in one line. Then cover what held, what didn't, and the surprise.
>
> **What held:** the agent never paid anyone, changed suppliers or opened the secrets file (16 path tricks, 13 forbidden tools). Fooled or not, money only went to approved accounts. A dropped connection plus three retries paid once. Any edit to the log stopped all approvals.
>
> **What didn't (pick 2–3 for readers):**
> - Nothing checks that the supplier the AI picked is the one named on the invoice. *(Strongest example.)*
> - Duplicate invoice numbers written in look-alike Cyrillic letters weren't caught.
> - The maker-checker rule could be dodged by typing a name in look-alike characters.
> - **New risk (R-12):** text written by the AI could erase the warnings on the approver's screen.
>
> **The surprise:** same AI, same instructions, and on a later run it added an extra warning it hadn't added before. That's why the AI is only ever the second line of defence.
>
> **Numbers to confirm:** first live run took 23 turns for about 10 US cents and matched all 11 expected outcomes. 156 attack tests plus 3 using the real AI. TODO: total test count, from `pytest --collect-only -q`.

{{< mermaid >}}
%%{init: {"theme": "base", "themeVariables": {"pie1": "#2e7d32", "pie2": "#f9a825", "pie3": "#c62828", "pieSectionTextColor": "#ffffff"}}}%%
pie showData
    title Controls after red-teaming (12 risks)
    "Effective" : 6
    "Partially effective" : 5
    "Ineffective" : 1
{{< /mermaid >}}

---

## What I learned

> ✏️ **WRITE (~200 words):** Four or five lessons, each with one sentence of evidence. Write this after Session 8 so it reflects your control assessment. Candidates:
> 1. **The tool list is the most important control.** An AI can't press a button that doesn't exist.
> 2. **Make the AI's job advisory.** Every fraud was stopped by code. The AI's warnings were useful but never relied on.
> 3. **A green test suite proves little on its own.** Break the control on purpose and check that a test fails.
> 4. **AI reviewers need checking too.** One review found real flaws and also made a confident false claim.
> 5. **Building with an AI coding assistant needs a process:** spec first, tests before code, read every change.
>
> Optional, the risk lens: what this suggests for firms deploying payment agents. Keep it generic and employer-agnostic.

---

## What's next

> ✏️ **WRITE (~80 words):** Part 2 is **mandates**. The customer gives the agent signed limits ("up to £200 a month to these suppliers, until December") instead of a human approving each payment. Who is responsible when an agent acts within its mandate but against your wishes?

---

*✏️ TODO: links to the code (if you make the repo public) and to the full build notes.*
