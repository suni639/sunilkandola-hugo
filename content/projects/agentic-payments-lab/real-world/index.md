+++
title = "Agentic Payments Lab: How the Lab Copies the Real World"
date = 2026-11-22
draft = false
weight = 8
description = "A plain-English companion to the Agentic Payments Lab: what each pretend bank, shop and card issuer copies from the real thing, including payment messages and codes, and what it leaves out."
summary = "Every part of the lab is a small, pretend version of something real. This companion explains, piece by piece, what the real thing does, how the lab copies it (messages, codes, signatures), what it leaves out, and why that matters for the findings."

tags = ["Agentic AI", "Payments", "Operational Risk", "ISO 20022", "Cards"]
categories = ["Projects"]
series = ["Agentic Payments Lab"]

showAuthor = true
showTaxonomies = true
showReadingTime = true
showTableOfContents = true
+++

{{< lead >}}
The lab's bank, shop and card issuer are pretend. But the messages they send each other, and the checks they make, are copied from real payment systems. This page explains how, and where the copy stops.
{{< /lead >}}

{{< alert "circle-info" >}}
This is a personal learning project. Everything in it is fictional: no real banks, money, data or people. Nothing here is legal advice. The views are my own.
{{< /alert >}}

**In one sentence:** every part of this AI (artificial intelligence) payments lab is a small, pretend version of something real (a bank, a shop, a card
issuer, a signed permission), and this page explains, piece by piece, what the real thing does, how the lab copies
it, what it leaves out, and why that matters for the findings.

New to payments? Read the [payments primer](/projects/agentic-payments-lab/primer/) first. This page is a companion
to [Part 1](/projects/agentic-payments-lab/part-1/), [Part 2](/projects/agentic-payments-lab/part-2/) and
[Part 3](/projects/agentic-payments-lab/part-3/).

---

## How the lab decides what to copy

A real bank runs thousands of systems. Copying them would take years and teach little about AI risk. So the lab
follows a rule I set at the start of Part 3: it **copies each outside party's role and checks, and leaves out
its internal systems**. Each outside party (the bank, the shop, the card issuer)
is a small program with just enough rules to show where its checks sit and what it decides. Where a real party
teaches nothing new for this series, it's merged into another one, and the write-up says so.

What the lab copies carefully is the part that matters for risk: **the messages that pass between the parties**,
meaning who sends what, what it contains, what proves it's genuine, and what each side checks before it says yes.
That's where
an AI agent can be fooled, and where a control can stop it. So the lab borrows real message formats, real status
and reason codes, and real internet standards for signatures, in simplified form.

Here are all the parties across the first three parts, and which ones are pretend:

{{< mermaid >}}
flowchart LR
    subgraph Lantern["Lantern Works (the fictional company)"]
        AG["🤖 AI agent"]
        TS["📏 Tool server<br/>fixed checks in code"]
        HU["👤 People<br/>approver, director"]
        EX["⚙️ Executor<br/>pays what's allowed"]
        LOG[("🔗 Audit log")]
    end
    subgraph Outside["Outside parties (all pretend)"]
        BK["🏦 Mock bank<br/>transfers, consents"]
        SH["🏪 Mock shop<br/>signed baskets"]
        CI["💳 Mock card issuer<br/>tokens, disputes"]
    end
    AG --> TS
    TS --> HU
    TS --> EX
    HU --> BK
    EX --> BK
    EX --> SH
    SH --> CI
{{< /mermaid >}}

Everything runs on one laptop. No real bank, card network or shop is involved, and nothing touches real money.

---

## A quick guide to "messages"

When a company asks its bank to pay someone, it doesn't phone or write a letter. Its software sends a **message**: a
structured block of data with fixed fields, such as who pays, who gets paid, how much, in what currency, and a
reference. The bank replies with another message saying what happened.

Two families of message standards matter here:

- **ISO 20022** (from ISO, the International Organization for Standardization) is the international standard for
  payment messages between customers and banks, and between banks.
  Each message type has a short code. The ones that matter here:
  - `pain.001`: a customer asks its bank to make a credit transfer (a payment)
  - `pain.002`: the bank reports back on the payment's status
  - `pacs.008`: one bank passes the payment on to another

  Replies carry a **status code** (accepted, rejected and so on) and, when something goes wrong, a **reason code**
  saying why.
- **Card messages** use a different, older standard, ISO 8583. When a card payment is declined, the card issuer
  answers with a two-digit **response code**, such as `51` for "not sufficient funds".

The lab doesn't send real ISO 20022 or ISO 8583 messages, which are long and full of fields this project doesn't
need. It sends small **JSON** messages instead (JSON is a common, human-readable way of writing structured data),
but they **keep the real names, codes and meanings**, so the habits carry over.

---

## Part by part

### The bank and its payment messages (Parts 1 and 2)

**What a real bank does.** It holds the company's account, checks every payment request, refuses ones it can't make,
and reports each payment's progress until the money has moved.

**How the lab copies it.** The mock bank is a small web service with five main actions: check a balance, list
known payees, check a payee's name, make a payment, and look up a payment. A payment request looks like this,
simplified:

```json
{
  "instruction_id": "prop_…",
  "debtor_account_id": "ACC-OPS-001",
  "creditor": {"name": "Brightwater Office Supplies", "sort_code": "99-10-01", "account_number": "10000001"},
  "amount": {"amount_minor": 48240, "currency": "GBP"},
  "remittance_info": "INV-001",
  "requested_execution_date": "2026-10-19"
}
```

Each field has a real counterpart in a `pain.001`:

- **"Debtor" and "creditor"** are the payer and the payee, the standard's own words.
- **The amount** is in whole pence (48240 means £482.40) with a currency code. That avoids the rounding errors
  decimals can cause.
- **The remittance information** is the reference that tells the payee what the payment is for.
- **The instruction ID** ties the payment to the AI's original proposal, so the whole trail links up.

**Status and reason codes.** The reply uses ISO 20022 codes. A payment the bank accepts gets `ACCP` and then moves
through "settling" to "settled". A refused payment gets `RJCT` (rejected) plus a reason code:

| Code | Meaning | When the mock bank uses it |
| --- | --- | --- |
| `AM02` | Amount not allowed | Over the £25,000 single-payment or £50,000 daily limit (lab limits) |
| `AM04` | Insufficient funds | Not enough money in the account |
| `AM05` | Duplication | The same instruction was already received |
| `AC01` | Incorrect account number | The payee's account isn't known |
| `FF01` | Invalid file format | The request itself is malformed |
| `MD01` | No mandate | Part 2: no valid consent for this payee |
| `AM14` | Amount exceeds agreed limit | Part 2: over a consent's limit |

**Confirmation of Payee.** In the UK, before you pay a new account, your bank checks the name you typed against the
name on the account. Pay.UK's scheme answers yes, a close match (showing the real name), no, or "unavailable" when
the check can't be done. The mock bank's name check copies the first three. Where the real check might be unavailable, the lab
fails closed: if any check can't run, the payment stops.

**Paying exactly once.** Networks drop connections. If a payment request times out, the sender can't tell whether
the bank received it. Sending it again could pay twice. Payment systems solve this with an **idempotency key**: a
unique label sent with the request. If the bank sees the same label again, it returns the original answer instead of
paying again. The IETF, the body that sets internet standards, is turning this into a standard
`Idempotency-Key` header, although it's still a draft. The mock bank requires one on every payment. The same key with a
different body is refused, which catches a resend that has been tampered with.

**Testing for outages.** The mock bank can be told to fail at random or respond slowly, so the lab can test what
happens when the bank is down. The rule throughout is to **fail closed**: if in doubt, don't pay.

**Left out:** the payment scheme between banks (Faster Payments in the UK) and the bank-to-bank messages (in ISO
20022, `pacs.008`), real settlement, account statements, and the bank's own fraud checks. The mock bank also doesn't check that a
person approved a payment. Part 1 relies on the AI having no way to send one, which is a known limitation.

### The AI agent and its tools (all parts)

**What's real.** An AI agent works in a loop: read the situation, pick a tool, look at the result, decide the next
step. The **Model Context Protocol (MCP)** is an open standard for connecting AI applications to tools and data. The
lab's agent is a real AI model connected to a real MCP tool server.

**How the lab shapes it.** The agent gets only a short list of named tools, and none of them can pay. It can't read
files directly, run commands or browse the web. Invoice and product text reaches it only through a tool, marked as
untrusted. This copies a real security principle, **least privilege**: give each part only the access it needs.
It's also how the lab defends against **prompt injection**, hidden instructions planted in text the AI reads. OWASP
(the Open Worldwide Application Security Project) ranks that the top risk for AI language models, including injection through files and web pages.

### The approver (Part 1)

**What's real.** Banks and finance teams use **maker-checker**: the person who prepares a payment can't also approve
it.

**How the lab copies it.** A command-line approval tool shows warnings first, demands a typed reason for any
flagged payment, refuses bulk approval, and refuses an approver whose name matches whoever ran the agent.

**Left out:** real logins. Names are typed, not checked, which Part 1's attacks exploited with look-alike letters.

### The audit log (all parts)

**What's real.** Regulated firms must be able to show who did what, and when.

**How the lab copies it.** Every step is written to a log in which each entry carries a **hash** of the one before
it. A hash is a short fingerprint of some data: change one character and the fingerprint changes completely. So
editing, deleting or reordering any entry breaks the chain, and the lab refuses to approve anything until someone
investigates.

**Left out:** an outside copy of the latest fingerprint. Someone who can rewrite both the log and its checkpoint
file can rewrite history. Part 1 rated this partially effective for exactly that reason.

### Signed permissions: mandates (Part 2)

**What's real.** The **Agent Payments Protocol (AP2)**, from Google and now with the FIDO Alliance, an industry
standards group, uses signed **mandates**. An **open** mandate holds the limits a person agrees in advance. A
**closed** mandate covers one exact transaction, made inside those limits.

**How the lab copies it.** The director signs an open mandate with her own key. Code signs a closed mandate for each
payment with the agent's key, and links it to the open one by including the open mandate's hash. The signatures use
**Ed25519**, a widely used digital signature method, published by the IETF as RFC 8032 (a numbered internet standards document). Before signing, the data is
written in one fixed way (keys in order, no spaces), so the same content always produces the same bytes and the
same signature. RFC 8785 is a fuller standard for this.

**Left out:** AP2's real formats and their transport between systems. The lab uses a simple signed block of JSON,
which keeps the idea with far less formatting.

### Bank-side consents (Part 2)

**What's real.** UK **variable recurring payments (VRP)** let a customer allow a business to take repeated payments
of different amounts, within limits held at the customer's bank. Open Banking's standards list the limits: a
maximum per payment, a maximum per period (such as a month), and an end date. A commercial version launched in June
2026.

**How the lab copies it.** The mock bank holds one consent per supplier with those limits, and refuses anything
outside them: `AM14` when over a limit, `MD01` when there's no valid consent.

**Left out:** the director approving each consent in her banking app with a fingerprint or face scan (**strong
customer authentication**). In the lab an admin command stands in for that.

### The shop and its signed basket (Part 3)

**What's real.** In AP2, a shop signs the agreed checkout (the basket and its price), and the closed mandate is tied
to that signed checkout by its hash. After payment, the shop must return a receipt.

**How the lab copies it.** The mock shop prices a basket, signs it with its own key and gives it 30 minutes to live.
Checkout needs an idempotency key and a signed request (below), and the shop refuses a mandate written for a
different basket. A confirmed order gets a signed receipt.

**Left out:** stock, delivery tracking, refunds and partial shipments. The shop also talks to the card issuer
directly (see "What isn't simulated").

### Proving who the AI is: signed requests (Part 3)

**What's real.** Visa's Trusted Agent Protocol, launched with Cloudflare in October 2025, lets an approved agent sign
its web requests so a shop can tell it from a bot. It builds on **HTTP Message Signatures** (RFC 9421, 2024), an
internet standard for signing selected parts of a web request (HTTP, the Hypertext Transfer Protocol, is how web
browsers and servers talk). Cloudflare's **Web Bot Auth** draft adds a way for
bots to publish their keys.

**How the lab copies it.** Each checkout request carries two extra headers:

```text
Signature-Input: sig1=("@method" "@authority" "@path" "content-digest");created=…;expires=…;keyid="…";nonce="…";tag="aplab-agent"
Signature: sig1=:…:
```

That means: "I signed the method (POST), the web address, the path, and a fingerprint of the exact content. Here's
when, until when, which key, and a one-time number." The content fingerprint uses the `Content-Digest` header from
RFC 9530. The shop refuses a request if:

- it's unsigned
- the key isn't on its trusted list
- the body was edited after signing
- the signature is more than 60 seconds old, or valid for more than 5 minutes (lab limits)
- the one-time number was seen before

**Left out:** the published key directory and the `web-bot-auth` tag the draft standard uses. The lab's shop loads
its trusted keys from a file.

### The card issuer and the token (Part 3)

**What's real.** Card schemes now issue **tokens** for AI agents: stand-ins for the card number, tied to one agent,
with spending limits and an end date (Visa Intelligent Commerce, for example). When asked to approve a payment, the
issuer answers with an ISO 8583 response code.

**How the lab copies it.** The mock issuer holds the token's rules: one shop, £250 per payment, £1,000 a month, an end
date. It keeps only a fingerprint of the token, never the token itself. It also checks the agent's signature
against the key enrolled with the token. Declines use codes whose meanings match Visa's published list:

| Code | Visa's meaning | When the mock issuer uses it |
| --- | --- | --- |
| `14` | Invalid account number | Unknown token |
| `54` | Expired card | Token past its end date |
| `57` | Transaction not permitted to cardholder | Shop not allowed on this token |
| `61` | Exceeds approval amount limit | Over the per-payment or monthly limit |
| `62` | Restricted card | Token suspended |
| `51` | Not sufficient funds | Over the card's credit limit |
| `63` | (not in Visa's list; a lab choice) | The agent's signature doesn't check out |

### Disputes (Part 3)

**What's real.** A cardholder can dispute a charge with their card issuer. The shop sends evidence, and the issuer
decides. Visa's rulebook has detailed "dispute conditions" (such as card-not-present fraud). Its October 2026
edition adds rules for AI payment services, including that the cardholder is responsible for what their AI service
does.

**How the lab copies it.** Three dispute reasons: not authorised, not received, not as described. The shop has 7
days to respond (a lab choice), and a fixed table decides. The company's evidence pack checks eleven links, from the
director's signature to the bank's approval.

**Left out:** the real dispute conditions, time limits and fees, and the card network that sits between issuer and
shop. The decision table is my own rule rather than Visa's.

---

## What isn't simulated, and why

| Real-world piece | What the lab does instead | Why | What it means for the findings |
| --- | --- | --- | --- |
| The payment scheme between banks (such as Faster Payments) and its bank-to-bank messages | The mock bank "settles" internally | The AI never touches bank-to-bank traffic | None for the AI controls; settlement risk isn't tested |
| The card network and the shop's own bank (the acquirer) | The shop calls the card issuer directly | They pass messages on; they don't add a check this series needs | Network-level fraud checks, like Mastercard's AI score, aren't modelled |
| Approval and money collection as separate steps | The money moves immediately | Keeps one purchase to one step | Holds, partial charges and refunds aren't tested |
| 3-D Secure (the extra login step for online card payments) | Not used | The AI's signature does a similar job here | A real flow might ask a person to confirm |
| Strong customer authentication and passkeys | Admin commands stand in | They involve a person and a phone | The lab assumes the director's setup step is genuine |
| Secure key storage (hardware vaults) | Keys in a private folder on disk | Simplicity | Key theft is easier than in a well-run firm; Parts 2 and 3 rate key risks partially effective |
| Real AP2, Web Bot Auth and card token formats | Simplified JSON with the same ideas | Formats change fast; the ideas don't | Findings are about the ideas, not the formats |
| Many live AI runs | Three live runs in Part 3 (and five in Part 1) on a handful of test cases | Each run costs money | The runs show how the model behaves, not how often; it varied between runs |

The test throughout: **does leaving it out change where a control sits or what it can see?** If not, it's left out. If
it does, the write-up says so, as in the table above.

---

## Why a simplified copy is still useful

The findings in Parts 1–3 are about **where checks sit and what each party can see**:

- the AI can't pay because it has no tool that pays
- the bank can't see the director's permission, only the AI's signature
- a signed basket can't be changed without detection

Those are properties of the flow, and the flow is copied faithfully. The real systems are far more detailed, but
they still have to answer the same questions: who signed this, what was agreed, and who checks it? Those questions apply to
them just as much.

What doesn't carry over is anything about scale, speed, cost, or how a real organisation runs its keys and its
people. The lab can show that a control works when it runs, but whether a real firm would run it well depends on
people and processes that these tests don't cover.

---

## Sources

Checked on 22 November 2026, against official sources only. Three ISO 20022 codes the lab uses (`AM14`, and the
"settling" and "settled" statuses `ACSP` and `ACSC`) couldn't be checked: the ISO 20022 site refused automated
access, and the official guides that could be read don't use them.

**ISO 20022 messages and codes**
- [European Payments Council: SEPA Credit Transfer customer-to-PSP implementation guidelines, EPC132-08, 2025 v1.0 (PDF)](https://www.europeanpaymentscouncil.eu/sites/default/files/kb/file/2024-11/EPC132-08%20SCT%20C2PSP%20IG%202025%20V1.0.pdf): `pain.001`, `pain.002`, `RJCT`, `AM05`, `AC01`, `FF01`
- [European Payments Council: SEPA Instant Credit Transfer inter-PSP implementation guidelines, EPC122-16, 2023 v1.2 (PDF)](https://www.europeanpaymentscouncil.eu/sites/default/files/kb/file/2023-11/EPC122-16%20SCT%20Inst%20Inter-PSP%20IG%202023%20V1.2_0.pdf): `pacs.008`, `ACCP`, `AM02`, `AM04`
- [European Payments Council: guidance on reason codes for SDD R-transactions, EPC173-14 v8.1 (PDF)](https://www.europeanpaymentscouncil.eu/sites/default/files/kb/file/2026-09/EPC173-14%20v8.1%20Guidance%20on%20Reason%20Codes%20for%20SDD%20R-transactions.pdf): `AM04`, `MD01`

**UK payments**
- [Pay.UK: Confirmation of Payee](https://www.wearepay.uk/what-we-do/overlay-services/confirmation-of-payee/) and [its FAQs](https://www.wearepay.uk/what-we-do/overlay-services/confirmation-of-payee/faqs/)
- [Open Banking: VRP for sweeping guidelines](https://standards.openbanking.org.uk/good-practice/vrp-for-sweeping-guidelines/latest/) (consent limits) and [variable recurring payments](https://www.openbanking.org.uk/variable-recurring-payments-vrps-cvrps/)
- [FCA: launch of the UK Payments Initiative scheme (2 June 2026)](https://www.fca.org.uk/news/statements/open-banking-launch-uk-payments-initiative-scheme)

**Internet standards**
- [IETF: RFC 9421, HTTP Message Signatures (February 2024)](https://www.rfc-editor.org/rfc/rfc9421)
- [IETF: RFC 9530, Digest Fields (February 2024)](https://www.rfc-editor.org/rfc/rfc9530)
- [IETF: RFC 8032, Edwards-Curve Digital Signature Algorithm (January 2017)](https://www.rfc-editor.org/rfc/rfc8032)
- [IETF: RFC 8785, JSON Canonicalization Scheme (June 2020)](https://www.rfc-editor.org/rfc/rfc8785)
- [IETF: The Idempotency-Key HTTP header field, Internet-Draft -09 (November 2026)](https://datatracker.ietf.org/doc/draft-ietf-httpapi-idempotency-key-header/)
- [IETF: HTTP Message Signatures for Bots, working draft -01 (October 2026)](https://datatracker.ietf.org/doc/draft-ietf-webbotauth-httpsig-protocol/)

**AI agents and their tools**
- [Model Context Protocol: specification](https://modelcontextprotocol.io/specification/latest)
- [OWASP: LLM01:2025 Prompt Injection](https://genai.owasp.org/llmrisk/llm01-prompt-injection/)
- [AP2 specification, v0.2](https://ap2-protocol.org/ap2/specification/) and [checkout mandate](https://ap2-protocol.org/ap2/checkout_mandate/)
- [FIDO Alliance: standards for trusted AI agent interactions (28 April 2026)](https://fidoalliance.org/fido-alliance-to-develop-standards-for-trusted-ai-agent-interactions/)

**Cards**
- [Visa: Visa introduces Trusted Agent Protocol (14 October 2025)](https://investor.visa.com/news/news-details/2025/Visa-Introduces-Trusted-Agent-Protocol-An-Ecosystem-Led-Framework-for-AI-Commerce/default.aspx)
- [Visa Developer: Visa Intelligent Commerce concepts](https://developer.visa.com/docs/visa-intelligent-commerce/concepts)
- [Visa Core Rules and Visa Product and Service Rules, 24 October 2026 edition (PDF)](https://usa.visa.com/dam/VCOM/download/about-visa/visa-rules-public.pdf): 4.1.23 (AI payment services), 7.3.6.3 (decline codes), 11.8.5 (card-absent fraud disputes)
- [EMVCo: EMV 3-D Secure](https://www.emvco.com/emv-technologies/3-d-secure/)
- [Mastercard: new trust and intelligence services for agentic commerce (30 September 2026)](https://investor.mastercard.com/investor-news/investor-news-details/2026/Mastercard-Advances-Agentic-Commerce-With-New-Trust-and-Intelligence-Services/default.aspx)
