+++
title = "FraudBusters: Can Banks Catch Fraud Together Without Sharing Data?"
date = 2024-06-15
lastmod = 2026-11-22
draft = false
description = "A four-phase learning project on credit card fraud detection: a pooled model, federated learning across five simulated banks, model updates logged on a blockchain, and an unfinished attempt at zero-knowledge proofs."
summary = "Banks would catch more fraud if they pooled their data, but they can't share it. I tested federated learning, where each bank trains on its own data and only shares the model, then added a blockchain and tried zero-knowledge proofs. Two of those worked."

featureImage = "feature.png"
featured = false

tags = ["Python", "Machine Learning", "Federated Learning", "Fraud", "Blockchain", "Zero-Knowledge Proofs", "Payments"]
categories = ["Projects"]

showAuthor = true
showTaxonomies = true
showReadingTime = true
showTableOfContents = true
showDateUpdated = true
+++

{{< lead >}}
Fraudsters work across banks, but each bank can only see its own customers. Pooling transaction data would help, but privacy rules and competition make that close to impossible. In 2024 I spent a few weeks testing whether banks could build a fraud model together without ever handing over their data.
{{< /lead >}}

{{< alert "circle-info" >}}
This was a personal learning project using a public, anonymised dataset. No real bank data was involved. The aim was to understand the technologies, not to build the best fraud model.
{{< /alert >}}

The code is on GitHub: [Phases 1 and 2](https://github.com/suni639/fraudbusters), [Phase 3A](https://github.com/suni639/fraudsters_phase3a) and [Phase 3B](https://github.com/suni639/fraudsters_phase3b).

## The problem

Fraud models get better the more examples of fraud they see, but fraud is rare and each bank only sees its own slice of it. A scam that hits five banks looks like five small, unrelated incidents.

The obvious fix, putting everyone's data in one place, runs into data protection law, customer confidentiality and the fact that banks compete with each other.

**Federated learning** is one way around this. Instead of sending data to the model, you send the model to the data:

1. Each bank trains a model on its own transactions, behind its own walls.
2. Each bank shares only the result: the model's settings, a list of numbers.
3. A coordinator averages those settings into one shared model and sends it back.

No customer transaction ever leaves the bank that holds it. Google researchers introduced the approach in 2016 and went on to use it for phone keyboard predictions, without collecting what people typed.

So this project asks: **does a federated model catch fraud as well as one trained on pooled data, and can newer technologies make the process more trustworthy?**

---

## How it works

I used a public [Kaggle dataset](https://www.kaggle.com/mlg-ulb/creditcardfraud) of 284,807 card transactions by European cardholders over two days in September 2013. Only 492 are fraud, 0.17%. The details are anonymised: apart from the time and amount, each transaction is described by 28 scrambled numbers.

The project ran in four phases, each building on the last.

{{< mermaid >}}
graph LR
    A[Phase 1<br>One pooled model] --> B[Phase 2<br>Federated across<br>5 banks]
    B --> C[Phase 3A<br>Updates logged<br>on a blockchain]
    B --> D[Phase 3B<br>Zero-knowledge<br>proofs]
{{< /mermaid >}}

1. **One pooled model.** The baseline: all the training data in one place, as if banks could share freely.
2. **Federated learning.** The training data split evenly across five simulated banks, each training its own model. The five were then averaged into one, a method called **federated averaging**.
3. **Phase 3A: a blockchain.** The same five banks, but each one's model was recorded on a blockchain before averaging, so every contribution was logged and couldn't be changed later.
4. **Phase 3B: zero-knowledge proofs.** An attempt to let each bank prove its model was trained properly without revealing anything about its data.

Every phase used **logistic regression**, one of the simplest machine learning models. It weighs each feature of a transaction and turns the total into a probability of fraud. I chose it because it runs on a laptop and its "settings" are just 31 numbers, which makes them easy to average and store.

That choice came from an earlier attempt. My first federated project used a neural network on a larger dataset of 6 million online payments, split across 20 clients. It ran out of memory before finishing. Switching to logistic regression on a smaller dataset kept the focus on how federated learning works, rather than on fighting my laptop.

**Dealing with rarity.** With fraud at 0.17%, a model that says "never fraud" is right 99.8% of the time and useless. Each bank therefore used **SMOTE**, a technique that creates synthetic fraud examples during training so the model learns what fraud looks like. All three models were tested on the same 56,962 transactions that none of them had seen, including 98 frauds.

---

## Results

| | Pooled model | Federated | Federated with blockchain |
|---|---|---|---|
| Frauds caught (of 98) | 90 | 89 | 86 |
| Genuine transactions wrongly flagged | 1,461 | 643 | 460 |
| False alarms per fraud caught | 16 | 7 | 5 |
| Overall ranking quality (AUPRC) | 0.73 | 0.72 | 0.71 |

{{< figure src="feature.png" alt="Bar chart of false alarms per fraud caught at a 50% cut-off: 16 for the pooled model (90 of 98 frauds caught), 7 for the federated model (89 caught) and 5 for the federated model with blockchain logging (86 caught)" caption="False alarms for every fraud caught, worked out from each model's recorded test results." >}}

The main result is that **the federated models caught almost as much fraud as the pooled one**: 89 of the 98 frauds against the pooled model's 90.

The false alarms look like a big win for federated learning, but that needs care. **AUPRC** measures how well a model ranks transactions from most to least suspicious, across every possible cut-off. All three score almost the same on it, which means the federated models are no better at telling fraud from genuine spending. They happen to land at a stricter point when the default "50% likely to be fraud" cut-off is applied, and tuning the pooled model's cut-off could probably give a similar trade-off.

Even at five false alarms per fraud, a real bank would be blocking or checking a lot of genuine payments, which is one reason real fraud systems use far richer data and models than this.

---

## Phase 3A: what the blockchain added

The weak point in federated learning is trust. How do you know a bank sent the model it actually trained? How does anyone check the coordinator averaged honestly?

I wrote a small **smart contract**, a program that runs on a blockchain, on a private test network on my laptop. Each bank's model settings were submitted to it as a transaction, and the coordinator read them back from the chain to average them. Once recorded, nobody could quietly change a contribution, and anyone with access could re-run the average and check the result.

{{< figure src="blockchain-updates.png" alt="Screenshot of the Ganache test blockchain showing five contract-call transactions, each from the same sender address to the FederatedLearning contract" caption="The five model updates recorded on the test blockchain. All five came from the same address, which is one of the gaps listed below." >}}

It worked, and the federated model performed much the same with the blockchain in the loop, although my version only covered part of the trust problem:

- **No access control.** Anyone could submit an update to the contract. A real network would need to limit it to member banks.
- **One sender.** All five updates came from the same test account, so the record couldn't show which bank sent which model.
- **Plain text.** The model settings were stored openly. Every participant could read every other bank's model.

Each of these has a known fix in permissioned networks, a signing key for each bank and encryption. It's also fair to ask whether a blockchain is needed at all, since a shared, append-only log signed by each bank would give much of the same audit trail. The main thing a blockchain adds is that no single party controls the record.

---

## Phase 3B: where it got stuck

Even without raw data, model settings can leak information. Research has shown that, in some cases, details of the training data can be reconstructed from model updates. **Zero-knowledge proofs** offer a way round this: mathematical proofs that let you show something is true without revealing why. In principle, a bank could prove "I trained this model correctly on real data" without exposing the model or the data.

The project stopped at this phase. I tried PySnark, a Python library for building these proofs. Turning even one training step of a model into a proof means rewriting the maths as a circuit of whole-number operations, and logistic regression uses decimals and exponentials throughout. After several attempts the code still didn't produce a working proof, even for toy numbers, let alone the real models. I logged the work as unresolved and moved on.

Zero-knowledge machine learning was a young field in 2024, and proving that a model was trained correctly is still much harder than proving a single prediction.

---

## What the project doesn't do

- **Identical banks.** The five simulated banks had evenly split, similar data. Real banks differ a lot in their customers, and federated learning is known to struggle more when they do.
- **One round.** Each bank trained once and the models were averaged once. Real federated systems repeat this many times.
- **Old, anonymised data.** Two days of 2013 card data, with the meaningful details scrambled. Good for learning, not for judging real-world performance.
- **One simple model.** Logistic regression keeps things understandable, but real fraud models are far more complex.

---

## Key takeaways

1. **Sharing models instead of data works.** On this dataset, five banks that never shared a transaction built a model that caught almost as much fraud as one trained on everything.
2. **Accuracy was the wrong measure here.** Every model scored 97 to 99%, which said almost nothing; frauds caught, false alarms and ranking quality were far more useful.
3. **The blockchain gave an audit trail, and the rest still had to be designed.** Access control, signatures and privacy weren't provided automatically.
4. **Some of the technology wasn't ready for me yet.** Zero-knowledge proofs for training were beyond what I could build in 2024, and finding that out was useful in itself.

The obvious next steps would be to try Phase 3B again with current zero-knowledge tools, and to run the federated setup with banks whose data looks less alike.
