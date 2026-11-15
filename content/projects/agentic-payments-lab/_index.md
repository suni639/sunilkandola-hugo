+++
title = "Agentic Payments Lab"
draft = false
layout = "series"
featured = true
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
This is a personal learning project. Everything in it is fictional: no real banks, money, data or people. The views are my own.
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

## The six parts

| Part | Question it asks | What gets built | Status |
| --- | --- | --- | --- |
| [1](/projects/agentic-payments-lab/part-1/) | Can an AI pay your bills safely? | An AI that suggests invoice payments, with fixed checks and a human approver | Complete |
| [2](/projects/agentic-payments-lab/part-2/) | How do you give an AI permission to spend, within limits? | Digital "mandates" and spending consents with caps, expiry and cancellation | Complete |
| 3 | How does an AI shop with a card? | A pretend online checkout where an AI pays with a restricted card token, plus disputes | Next |
| 4 | How does a bank spot a misbehaving AI? | A monitoring system that scores AI-initiated payments for fraud | Planned |
| 5 | How do machines pay machines? | An AI paying per use with digital dollars on a test blockchain | Planned |
| 6 | Can one set of controls cover every way of paying? | An AI treasury choosing between bank transfer, card and digital money, under one rulebook | Planned |

Each finished part has a main article, which explains the ideas and the results, and a set of build notes with
the session-by-session detail.

## What the first two parts found

- **What an AI can't do matters more than what it's told.** In Part 1 the AI had no "pay" button, so even a fully
  fooled AI couldn't send money to a fraudster's account.
- **Checks in code beat the AI's judgement.** The AI spotted most tricks, but not every time, and gave slightly
  different answers on different runs.
- **Removing the person removes checks nobody wrote down.** In Part 2 the AI paid regular bills on its own, within
  a signed permission. The permission held, but nobody was left asking "is this bill real?".

## Ground rules

- **Everything is fictional or on test networks.** No real money, banks, companies or personal data.
- **Built in the open, with AI.** The code is written with an AI coding assistant, with a human reviewing every change.
- **Honest results.** Each part reports which safeguards worked and which didn't.

The code for every part is on [GitHub](https://github.com/suni639/agentic-payments-lab).
