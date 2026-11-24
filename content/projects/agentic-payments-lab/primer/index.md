+++
title = "Agentic Payments Lab: A Ten-Minute Payments Primer"
date = 2026-11-15
draft = false
weight = 1
aliases = ["/projects/agentic-payments-lab-primer/"]
description = "How money actually moves between bank accounts, by card and on blockchains, explained from scratch for readers of the Agentic Payments Lab series."
summary = "The payments basics behind the series: who is involved in a payment, how a bank transfer moves, push and pull payments, why payments are hard to reverse, and where cards and digital money fit."

tags = ["Payments", "Operational Risk", "Open Banking", "Cards", "Stablecoins"]
categories = ["Projects"]

showAuthor = true
showTaxonomies = true
showReadingTime = true
showTableOfContents = true
+++

{{< alert "circle-info" >}}
This is a personal learning project. Everything in it is fictional: no real banks, money, data or people. Nothing here is legal advice. The views are my own.
{{< /alert >}}

**In one sentence:** this page explains how payments work, from scratch, so the rest of the
[Agentic Payments Lab](/projects/agentic-payments-lab/) series makes sense without a banking background.

It covers only what the series needs. Each idea links to the part where it matters.

---

## Who is involved in a payment

Even a simple bank transfer has at least five parties:

| Party | Who they are | Example from the series |
| --- | --- | --- |
| **Payer** | Whoever the money comes from | Lantern Works Ltd, the fictional company |
| **Payee** | Whoever the money goes to | A supplier, such as Brightwater Office Supplies |
| **Payer's bank** | Holds the payer's account and sends the money | The pretend bank in Parts 1 and 2 |
| **Payee's bank** | Holds the payee's account and receives the money | Simplified away in this series |
| **Payment scheme** | The shared rules and network the banks use to move money between them | In the UK, Faster Payments for most transfers |

When an AI (artificial intelligence) agent joins in, it adds a sixth party: someone acting *for* the payer who isn't the payer. Most of
this series is about what that changes.

---

## How a bank transfer moves

{{< mermaid >}}
graph LR
    P[Payer] -->|1. instruction| PB[Payer's bank]
    PB -->|2. checks| PB
    PB -->|3. clearing message| S[Payment scheme]
    S -->|3. clearing message| RB[Payee's bank]
    RB -->|4. credit| R[Payee]
    PB -.->|5. settlement:<br/>banks square up| RB
{{< /mermaid >}}

1. **Instruction.** The payer tells their bank: pay this amount, to this sort code and account number, with this
   reference.
2. **Checks.** The payer's bank checks the payer is who they say they are, there's enough money, the payment is
   within limits, and it doesn't look like fraud. Many UK banks also run **Confirmation of Payee**: does the name
   the payer typed match the name on the receiving account?
3. **Clearing.** The banks exchange messages saying what is owed to whom.
4. **Credit.** The payee's bank puts the money in the payee's account.
5. **Settlement.** The banks themselves square up, so the money has really moved between them rather than only
   being promised.

In the UK, a Faster Payment usually reaches the payee within seconds, at any time of day. Some take longer, for
example when a bank holds a payment to check it for fraud. Settlement comes later: the banks square up in batches,
three times each working day, through their accounts at the Bank of England. Other systems are slower: Bacs,
which carries Direct Debits, takes about three working days.

**Why the steps matter for this series.** Each step is a place where a control can sit, and each is a place where
something can go wrong. Part 1's pretend bank shows a payment moving through *accepted → settling → settled*,
which mirrors these stages.

---

## Push and pull payments

There are two basic ways money can move:

- **Push.** The payer sends the money. A bank transfer is a push payment. The payer's bank acts only on the payer's
  instruction.
- **Pull.** The payee takes the money, with the payer's earlier permission. Direct Debits and card payments are pull
  payments.

This difference matters a lot for AI agents. In [Part 1](/projects/agentic-payments-lab/part-1/), the AI
*suggests* push payments and a person sends them. In [Part 2](/projects/agentic-payments-lab/part-2/), the
company gives permission in advance, closer to a pull, and the AI pays within it. The newer UK scheme Part 2
borrows from, **variable recurring payments (VRP)**, sits in between. The payer sets up a permission at their bank,
with limits: a maximum per payment, a maximum per month, and start and end dates. Payments are then made within
those limits without asking each time.

---

## Authentication and authorisation

These two words sound alike but mean different things:

- **Authentication** proves *who you are*, for example with a password, a fingerprint or a code sent to your phone.
  UK and EU (European Union) rules
  require **strong customer authentication** for most payments: two separate kinds of proof, such as something you
  know and something you have.
- **Authorisation** is *permission for this payment*. "Yes, I want to pay £482.40 to Brightwater."

AI agents put pressure on both ideas. An agent isn't the account holder, so it can't authenticate as one. And when it pays without
asking, the account holder didn't authorise *this* payment, only payments *like* it. Part 2's signed mandates are one
answer: they turn "I said the AI could" into evidence a computer can check.

---

## Why payments are hard to reverse

Once a bank transfer has settled, it usually can't be pulled back. The payee's bank won't take money out of its
customer's account just because the payer changed their mind. This is called **finality**, and it is why every
safeguard in this series sits *before* the money moves.

The rules on who bears a loss depend on what went wrong:

- **Unauthorised payments.** If money leaves your account without your permission, your bank usually has to refund
  you.
- **Authorised payments to a fraudster.** If you were tricked into sending money yourself, for example by a fake
  "our bank details have changed" letter, the payment counts as authorised. That is called **authorised push
  payment (APP) fraud**. Since October 2024, UK rules require payment firms to reimburse most APP fraud victims who
  paid by Faster Payments or CHAPS (the UK's system for high-value payments), up to £85,000 a claim. That protection covers individuals, micro-businesses and
  charities. A larger business isn't covered, so if it pays a fraudster's account, it usually bears the loss.
- **Card payments** work differently. A cardholder can ask their card issuer to reverse a payment, for example for goods
  that never arrived. This is called a **chargeback**, and it is a big part of Part 3.

An AI agent paying for you raises an awkward question: was a payment it made, within limits you set but not one
you expected, authorised or not? HM Treasury, the UK government's finance ministry, asked exactly that in a July 2026 consultation.
The series leaves the legal question to lawyers and builds the evidence any answer would depend on.

---

## What can go wrong: an operational risk view

**Operational risk** is the risk of loss from failed processes, people, systems or outside events. In payments it
mostly looks like this:

| What goes wrong | Example | Typical controls |
| --- | --- | --- |
| Wrong payee | Paying a fraudster's account after a fake "bank details changed" letter | Approved supplier list; Confirmation of Payee; call back on a known number |
| Wrong amount | A misread invoice, or an extra zero | Amount checked against the invoice; limits |
| Paying twice | The same invoice sent twice, or a "pay" button pressed twice | Duplicate checks; unique references (*idempotency*) |
| Unauthorised payment | Someone pays without the right permission | Maker-checker (one person starts, another approves); separation of duties |
| Systems fail | A bank's service is down mid-payment | Fail safe: if a check can't run, don't pay |
| No evidence | A disputed payment that nobody can explain | An audit trail recording who did what, when and why |

A **control** is anything that stops a risk happening (*preventive*) or spots it when it does (*detective*). Every
part of the series builds controls like these, attacks them, and rates how well they held.

---

## Payment messages and codes

Banks talk to each other in standard messages. The international standard is **ISO 20022**, from ISO, the International Organization for Standardization. It defines what a
payment message contains, and short codes for what happened to it. For example:

| Code | Meaning |
| --- | --- |
| ACCP | Accepted, checks passed |
| ACSC | Settled: the money has moved |
| AM04 | Refused: not enough money in the account |
| AM05 | Refused: duplicate payment |
| AC01 | Refused: the account number is wrong |

The pretend bank in this series borrows these codes, so its answers look like a real bank's. You don't need to
remember them: the articles always say what a code means. I couldn't check `ACSC` against an official page,
because the ISO 20022 site blocks automated reading and the official guides I could read don't use it.

---

## Open banking

**Open banking** lets a company you choose, other than your bank, start payments from your bank account, with your
permission. The company is called a **payment initiation service provider**. Your bank still authenticates you and
applies its own checks.

This is how an AI agent could pay from a bank account in the real world: through a regulated company acting on your
behalf, using permissions you set up at your bank. Part 2's "consent at the bank" is modelled on this.

---

## Cards, in brief

Card payments (the subject of Part 3) involve different parties:

{{< mermaid >}}
graph LR
    C[Cardholder] -->|pays| M[Merchant]
    M --> A[Merchant's bank<br/>the acquirer]
    A --> N[Card network]
    N --> I[Cardholder's bank<br/>the issuer]
    I -.->|approves or declines<br/>in seconds| M
{{< /mermaid >}}

- **Authorisation happens in seconds.** The issuer approves or declines while you wait at the till.
- **Settlement happens later,** usually within a day or two.
- **Tokens replace card numbers.** Your phone or an online shop often stores a *token*, a stand-in for your real card
  number that only works in one place. An AI agent can be given a token with its own limits.
- **Chargebacks** let a cardholder dispute a payment afterwards, so cards give the payer more protection than bank
  transfers do.

---

## Digital money, in brief

Parts 5 and 6 use money that lives on a **blockchain**: a shared record of who owns what, kept by many computers at
once, so that no single party can quietly change it.

- A **stablecoin** is a digital token designed to keep a steady value, usually one US dollar, backed by reserves
  held by the company that issues it.
- A **tokenised deposit** is an ordinary bank deposit, recorded as a token on a blockchain by the bank that holds it.
- **Programmable money** means payments can follow rules written in code: "pay only when the goods arrive", or "pay
  a fraction of a penny per use".
- **Keys are everything.** Whoever holds the secret key to a blockchain account controls its money. Payments are
  usually final within minutes, with no bank to ask for a reversal.

That makes digital money attractive for machines paying machines, and makes controls even more important, because
there is no one to call afterwards.

---

## Where each idea appears in the series

| Part | Payment ideas it relies on |
| --- | --- |
| [Part 1](/projects/agentic-payments-lab/part-1/): an AI suggests bank transfers | Push payments, Confirmation of Payee, duplicates and idempotency, maker-checker, audit trail |
| [Part 2](/projects/agentic-payments-lab/part-2/): an AI pays within a signed permission | Authorisation, variable recurring payments, open banking consents, evidence for disputes |
| Part 3: an AI shops with a card | Card parties, tokens, chargebacks |
| Part 4: a bank watches for a misbehaving AI | Fraud monitoring, operational risk measures |
| Part 5: machines pay machines | Stablecoins, keys, finality |
| Part 6: one rulebook for every way of paying | All of the above |

---

## Sources

Official sources only. The Faster Payments and Bank of England pages were checked on 15 November 2026; the rest on
22 November 2026.

**Fraud refunds (authorised push payment fraud)**
- [Payment Systems Regulator: APP fraud reimbursement protections](https://www.psr.org.uk/information-for-consumers/app-fraud-reimbursement-protections/)
- [Payment Systems Regulator: APP scams reimbursement requirement, consolidated policy statement PS25/5](https://www.psr.org.uk/media/rhelv4op/ps25-5-app-scams-reimbursement-consolidated-policy-statement-may-2025.pdf)

**Faster Payments: speed and settlement**
- [Pay.UK: how Faster Payments work](https://www.wearepay.uk/what-we-do/payment-systems/faster-payment-system/how-faster-payments-work/)
- [Bank of England: working paper on RTGS and instant payment schemes (2026)](https://www.bankofengland.co.uk/-/media/boe/files/working-paper/2026/substitution-between-rtgs-and-instant-payment-schemes.pdf)
- [Bank of England: RTGS and CHAPS service description](https://www.bankofengland.co.uk/-/media/boe/files/payments/rtgs-and-chaps-service-description)

**Payment message codes (ISO 20022)**
- [European Payments Council: SEPA Credit Transfer customer-to-PSP implementation guidelines, 2025 (PDF)](https://www.europeanpaymentscouncil.eu/sites/default/files/kb/file/2024-11/EPC132-08%20SCT%20C2PSP%20IG%202025%20V1.0.pdf)
- [European Payments Council: SEPA Instant Credit Transfer inter-PSP implementation guidelines, 2023 (PDF)](https://www.europeanpaymentscouncil.eu/sites/default/files/kb/file/2023-11/EPC122-16%20SCT%20Inst%20Inter-PSP%20IG%202023%20V1.2_0.pdf)

**Confirmation of Payee**
- [Pay.UK: Confirmation of Payee](https://www.wearepay.uk/what-we-do/overlay-services/confirmation-of-payee/)

**Variable recurring payments**
- [FCA: launch of the UK Payments Initiative scheme (2 June 2026)](https://www.fca.org.uk/news/statements/open-banking-launch-uk-payments-initiative-scheme)
- [Open Banking: variable recurring payments](https://www.openbanking.org.uk/variable-recurring-payments-vrps-cvrps/)

**AI payments and liability**
- [HM Treasury: Modernising payment services regulation, consultation (14 July 2026)](https://www.gov.uk/government/consultations/modernising-payment-services-regulation/modernising-payment-services-regulation-consultation)

---

## Glossary

| Term | Meaning |
| --- | --- |
| Authentication | Proving who you are, for example with a password and a code sent to your phone |
| Authorisation | Permission for a particular payment |
| Authorised push payment (APP) fraud | Tricking someone into sending money to a fraudster themselves |
| Blockchain | A shared record of who owns what, kept by many computers so no one party can quietly change it |
| Chargeback | A card payment reversed after a cardholder disputes it |
| Clearing | Banks exchanging messages about what is owed to whom |
| Confirmation of Payee | A UK check that the name you typed matches the name on the account |
| Control (preventive / detective) | Something that stops a risk (preventive) or spots it (detective) |
| Finality | The point after which a payment can't be reversed |
| ISO 20022 | The international standard for payment messages and their codes |
| Maker-checker | One person starts a payment and a different person approves it |
| Open banking | Letting a company you choose start payments from your bank account, with your permission |
| Operational risk | Risk of loss from failed processes, people, systems or outside events |
| Payee / payer | Who receives the money / who sends it |
| Payment scheme | The shared rules and network banks use to move money between them |
| Push / pull payment | The payer sends the money (push) or the payee takes it with permission (pull) |
| Settlement | Banks squaring up so the money has really moved between them |
| Stablecoin | A digital token designed to keep a steady value, usually one US dollar |
| Strong customer authentication | Proving who you are with two separate kinds of proof |
| Token (card) | A stand-in for a real card number that only works in one place |
| Tokenised deposit | An ordinary bank deposit recorded as a token on a blockchain by the bank that holds it |
| Variable recurring payments (VRP) | A UK scheme letting a payer allow repeated payments within limits set at their bank |
