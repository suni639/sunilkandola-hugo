+++
title = "Agentic Payments Lab"
draft = false
layout = "series"
featured = true
featureImage = "feature.png"
intro = """AI agents are starting to make payments for people, and money itself is becoming programmable. In this hands-on series I build small, entirely fictional payment systems with AI, then try to break them, asking one question throughout: **what could go wrong, and how would you know?**

No technical or banking background needed. New to payments? Start with the primer, then read the parts in order."""
description = "A six-part, hands-on series exploring how AI agents will make payments, and what it takes to make that safe, building towards the convergence of AI and blockchain-based money."
summary = "Six hands-on projects on AI agents in payments, from a simple invoice-paying assistant to an agent treasury working across bank transfers, cards and digital money."

tags = ["Agentic AI", "Payments", "Operational Risk", "DLT", "Stablecoins"]
categories = ["Projects"]

showAuthor = true
showTaxonomies = true
showReadingTime = true
+++

{{< alert "circle-info" >}}
This is a personal learning project. Everything in it is fictional: no real banks, money, data or people. Nothing here is legal advice. The views are my own.
{{< /alert >}}

AI is moving from answering questions to taking actions, and one of the most consequential actions is
paying for things. Banks, card networks and payment companies are racing to let AI **agents** make payments
on people's behalf. At the same time, money itself is becoming programmable, through stablecoins and
tokenised bank deposits that live on blockchains.

This series is my hands-on way of understanding where those two trends meet. Each part builds a small,
working, entirely fictional system, then tries to break it. My background is in payments risk, so every
part asks the same question alongside "how does it work?": **what could go wrong, and how would you know?**

You don't need a technical background to follow along. Each part explains its terms from scratch.

**New to payments?** Start with the [ten-minute payments primer](/projects/agentic-payments-lab/primer/).
It explains how money moves between bank accounts, why payments are hard to reverse, and where cards and digital
money fit.

**Wondering how realistic it is?** [How the lab copies the real world](/projects/agentic-payments-lab/real-world/)
explains what each pretend bank, shop and card issuer copies from real systems, including the payment messages and
codes, and what it leaves out.

## The six parts

| Part | Question it asks | What gets built | Status and reading |
| --- | --- | --- | --- |
| [1](/projects/agentic-payments-lab/part-1/) | Can an AI pay your bills safely? | An AI that suggests invoice payments, with fixed checks and a human approver | Complete: [article](/projects/agentic-payments-lab/part-1/) · [build notes](/projects/agentic-payments-lab/part-1-build-notes/) |
| [2](/projects/agentic-payments-lab/part-2/) | How do you give an AI permission to spend, within limits? | Digital "mandates" and spending consents with caps, expiry and cancellation | Complete: [article](/projects/agentic-payments-lab/part-2/) · [build notes](/projects/agentic-payments-lab/part-2-build-notes/) |
| [3](/projects/agentic-payments-lab/part-3/) | How does an AI shop with a card? | A pretend online checkout where an AI pays with a restricted card token, plus disputes | Complete: [article](/projects/agentic-payments-lab/part-3/) · [build notes](/projects/agentic-payments-lab/part-3-build-notes/) |
| 4 | How does a bank spot a misbehaving AI? | A monitoring system that scores AI-initiated payments for fraud | Next |
| 5 | How do machines pay machines? | An AI paying per use with digital dollars on a test blockchain | Planned |
| 6 | Can one set of controls cover every way of paying? | An AI treasury choosing between bank transfer, card and digital money, under one rulebook | Planned |

Each finished part has a main article, which explains the ideas and the results, and a set of build notes with
the session-by-session detail.

## What the first three parts found

- **What an AI can't do matters more than what it's told.** In Part 1 the AI had no "pay" button, so even a fully
  fooled AI couldn't send money to a fraudster's account.
- **Checks in code beat the AI's judgement.** The AI spotted most tricks, but not every time, and gave slightly
  different answers on different runs.
- **Removing the person removes checks nobody wrote down.** In Part 2 the AI paid regular bills on its own, within
  a signed permission. The permission held, but nobody was left asking "is this bill real?".
- **"Authorised" is not the same as "wanted".** In Part 3 the AI shopped with a restricted card token. Signatures
  stopped bots, edited baskets and look-alike shops. But the same signatures that protect the shop prove the company
  agreed, so the company pays for a purchase its AI was allowed to make but nobody needed.

## Open gaps across the series

Each part ends with the weaknesses it found. Some were fixed in a later part, some are still open, and some come
back in a new form. This table keeps the running picture. Each part's article has the full risk table.

| Gap | Found in | Status |
| --- | --- | --- |
| The AI could pick an approved supplier that wasn't the one on the invoice | Part 1 | **Fixed** in Part 2: code checks the payee matches the invoice |
| Text the AI writes could hide warnings on the approver's screen | Part 1 | **Fixed** in Part 2: such text is refused and escaped |
| Bank details written with dots or slashes don't raise a warning | Part 1 | Open. Money still can't go to them, but the warning is lost |
| A resent invoice with look-alike letters in its number isn't caught as a duplicate | Part 1 | Open. Since Part 2 it could be paid automatically, within the limits |
| A name in look-alike letters gets round "you can't approve your own payments" | Part 1 | Open |
| Any approval reason is accepted, even "ok" | Part 1 | Open |
| Someone who can edit both the audit log and its checkpoint can rewrite history | Part 1 | Open in every part |
| A model name that moves to newer versions counts as "pinned" | Part 1 | Open |
| A convincing fake bill from a covered supplier is paid automatically | Part 2 | **Partly fixed** in Part 3: newer mandates send a bill outside the supplier's usual pattern to a person |
| A thief with the AI's key and access to its files can authorise payments within the limits | Part 2 | Narrowed in Part 3: the shop also checks the permission slip against a second key. Keys still sit on disk |
| A well-formed fake staff request gets bought | Part 3 | Open. The same problem as the fake bill, in a new form |
| Code checks quantity and budget, not that it's the right product | Part 3 | Open. Left to the AI's judgement |
| A shop is trusted from the moment a person onboards it | Part 3 | Open |
| Who pays when an AI makes an allowed but unwanted purchase | Part 3 | Open question. In the lab the company pays, and Visa's rules point the same way |

## Ground rules

- **Everything is fictional or on test networks.** No real money, banks, companies or personal data.
- **Built with AI.** The code is written with an AI coding assistant, with a human reviewing every change. The code
  itself is private for now. Each part's build notes describe how it was built and tested.
- **Honest results.** Each part reports which safeguards worked and which didn't.
