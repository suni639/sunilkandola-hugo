+++
title = "Agentic Payments Lab, Part 2: Build Notes"
date = 2026-11-01
draft = false
weight = 11
aliases = ["/projects/agentic-payments-lab-part-2-build-notes/"]
description = "Session-by-session notes on building Part 2 of the Agentic Payments Lab: signed mandates, the bank's own consents, the executor and the attacks on them."
summary = "The detailed build diary behind Part 2: what each session built, what went wrong, and how each safety check was tested."

tags = ["Agentic AI", "Payments", "Operational Risk", "Mandates", "Open Banking", "Python", "Claude Code"]
categories = ["Projects"]

showAuthor = true
showTaxonomies = true
showReadingTime = true
showTableOfContents = true

[build]
  list = "never"
+++

{{< alert "circle-info" >}}
This is a personal learning project. Everything in it is fictional: no real banks, money, data or people. Nothing here is legal advice. The views are my own.
{{< /alert >}}

These are the detailed notes behind [Part 2: Can You Give an AI Permission to Spend?](/projects/agentic-payments-lab/part-2/).
Read that first. It explains the ideas and the results. This page is for readers who want to see how each piece
was built and tested, one session at a time. Terms are explained in the main article and the [payments primer](/projects/agentic-payments-lab/primer/).

## Before starting: two fixes from Part 1

Part 1 ended with a list of weaknesses, called *findings*, found by attacking my own system. In Part 2 some
payments go through with no person checking them, so two findings had to be fixed first.

**Paying the wrong approved supplier (finding F-2).** The company keeps a list of approved suppliers, called the
*directory*. The AI can only pay someone on that list. But nothing checked that the supplier it picked was the
one who sent the bill. A fake note on Brightwater's bill could talk the AI into paying Harbour IT instead, and
Harbour IT is on the list too. The fix is a ninth automatic check. Code reads the supplier's name from the
*letterhead*, the name printed at the very top of the bill. That name must match the name the AI gives. If it
doesn't, the payment is stopped. Before building it, I checked by hand that all 23 test bills have the supplier's
name as their first line. Bills marked "resent" have a warning line above it, which is skipped. One limit
remains: a fraudster who forges the whole bill, letterhead included, still gets through this check.

**Hidden codes in the AI's notes (finding F-6).** The AI writes a short note explaining each payment, and the
person approving it reads that note on their screen. A *terminal* (the text window the approver uses) treats
some invisible characters as commands. They can wipe lines, move text or show it backwards. A crafted bill could
get the AI to copy such characters into its note, and hide a warning from the approver. Now these characters
are refused before a suggestion is even recorded. The approval screen also shows any odd character as a
harmless code, such as `\x1b`. It puts the AI's words in a clearly labelled box. And when it asks why you are
approving a flagged payment, it names the warnings again.

Both attacks are automated tests. They used to be marked "expected to fail". They now pass, and the marks are
gone.

## Checking the plan before building

Before writing any code, I checked the plan itself. The plan is called the *spec* (short for specification). It
says exactly what the system must do. I checked it against the twelve test bills by hand. Every total added up.
The seven ordinary bills that fit within the director's permission add up to £3,221.18, but the monthly budget is
£3,000.00. So at least one of them must always go to a person, whatever order the bills arrive in.

The check also found a real hole. Sometimes a payment reaches the bank, but the bank's reply gets lost on the
way back. The system then tries again, and the bank recognises the repeat, so it pays only once. But the plan
also said: if the permission has run out, or someone cancelled it in the meantime, refuse to try again. That
sounds safe. It isn't. The first attempt may already have paid the bill. The system would then record "refused"
and put the money back in the budget. The same bill could then be suggested and paid a second time.

The fix is simple. Before sending a payment, the system writes down "about to send". Once that note exists, it
never refuses that payment. It only repeats the exact same request, which the bank will not pay twice. To stop a
payment that is already on its way, you cancel the permission at the bank itself.

I also made eleven decisions the plan had left open. The main ones:

- **A bill the bank refused is never retried automatically.** Next time, it goes to a person. Otherwise the bill
  that is £25 over the bank's limit would be suggested and refused on every run, forever.
- **If the bank permission for a supplier has gone, that supplier is put on hold.** New bills from them go to a
  person until someone lifts the hold. Lifting it gives the AI more room again, so it needs a name and a reason.
- **Anyone named can cancel the director's permission.** Making a permission smaller should be easy. The worst
  a mistaken cancellation can do is send bills to a person.
- **The month is the month the payment was allowed, never the date printed on the bill.** Anything printed on a
  bill could have been written by a fraudster, so it never decides what the AI may do.

## Signing a permission so it can't be forged

In Part 2 the director gives the AI permission in advance, in a document called a *mandate*. Code must be able to
tell a real mandate from a fake one, or from a real one someone has quietly edited. Session 2 built the part that
does this: *digital signatures*.

**Keys.** Each person, and the AI, gets a *key pair*: two linked numbers. The *private key* is secret and is
used to sign. It is kept in a file only its owner can read, never in the project's files and never shown to the
AI. The *public key* can be shared with anyone and is used to check signatures. You can't work out the private
key from the public one.

**Signing.** Signing a mandate turns its exact contents, and the private key, into a short code called a
*signature*. Anyone with the public key can check it. If a single digit of the mandate changes afterwards, the
check fails. I tried this. I raised the payment limit from £2,000.00 to £2,900.00 after signing, and the mandate
was rejected. A signature copied from a different mandate was rejected too.

**Which keys count.** The system keeps a list of public keys, called the *registry*. But a list in an ordinary
file could be edited by anyone with access, so the list alone proves nothing. Every new key is also written to
the *audit log*, the tamper-evident logbook from Part 1. A key only counts if the list and the logbook agree, and
the logbook passes its own tamper check. If either can't be read, no key counts, and nothing gets paid.

**The checks.** Before any mandate is trusted, code checks that:

- it has exactly the expected fields, with numbers where numbers belong. Extra or misspelt fields are refused.
- the key named in the signature is the key named in the mandate. Otherwise a real signature could be relabelled
  as someone else's.
- the key is in the registry and the logbook, hasn't been cancelled, and belongs to the right kind of signer. A
  director's key signs mandates. The AI's key signs single payments.
- the key belongs to the person named as the signer. Another director's real key can't sign in Dana's name.
- the signature itself is correct.

For the AI's single-payment permissions, three more checks follow. The payment must point to a mandate that was
really issued, by an exact fingerprint of its contents. It must come from the AI the mandate names. And it must be
less than 24 hours old.

**Cancelling a key.** Anyone named can cancel (*revoke*) a key, with a reason. Making permission smaller should
be easy. A cancelled key can't be used again, and everything it signed stops working, including its mandates. To
keep paying bills, the director makes a new key and signs a new mandate. Key names are never reused, so an old
record always points to exactly one key.

**Passphrases.** A director's private key can be locked with a *passphrase*, a password typed each time it is
used. The AI's key can't be, because the system uses it with nobody there to type anything. A passphrase stored
next to the key would protect nothing.

**Testing the tests.** All the new tests passed the first time the code ran. That alone proves little. So I broke
each safety check on purpose, one at a time, 25 in all, and made sure a test failed each time. Three breaks went
unnoticed at first. One showed a real gap: a signature could be written two ways that both counted. I added a test
for it. One check turned out to be a backup for another and can't be tested on its own. That is recorded, not
hidden.

One limit: on Windows, which this project runs on, the "only its owner can read it" setting on key files doesn't
take effect. There, only the folder's location protects the keys.

## Signing the permission, checking it, and taking it back

Session 3 built what a director actually does with a mandate, and the twelve checks every payment must pass
under it.

**Reading before signing.** Before Dana signs, the system shows a summary in plain English: who may be paid,
how much per payment, how much and how many payments per month, from when until when, and what happens outside
those limits. The summary is made from exactly the data that will be signed, never from the draft file's text.
It also shows a *fingerprint* (a hash: a short code that changes if a single character of the mandate changes).
Dana must type the mandate's name, `MND-001`, to sign. Anything else cancels it.

**A second pair of eyes.** I asked a separate AI reviewer, given no background, to read that summary as a
director would and list anything they could misread. It found eight things. Four were real and cheap to fix,
all wording: the mandate approves supplier *codes*, so payments go to whatever bank details are on file for
them; "recurring bills" is only a description, so any bill within the limits can be paid; anything outside the
limits goes to a person; and the monthly limits apply "whichever comes first". The other four weren't worth
the extra work for a mock system, and that choice is written down.

**The twelve checks.** Every payment the AI says is covered goes through twelve checks in code. Was the mandate
really issued, and is its signature still good? Has it been cancelled? Is today inside its dates? Is this the
AI it names, a supplier it lists, the right account and currency? Is the payment within the per-payment limit,
the monthly budget and the monthly count? Is the bill free of warnings? Is the supplier not on hold? All twelve
always run, so a person sees every reason at once. If any fails, the bill goes to a person, as in Part 1.

**Counting the budget.** How much of the monthly budget is used comes only from the tamper-evident logbook,
never from an ordinary file that someone could edit. A payment the bank refused gives its amount back. A month
is a calendar month in UTC (the world's reference time, the same as UK winter time).

**Taking it back.** Anyone named can cancel (*revoke*) a mandate, with a reason. It can't be undone. A *payee
hold* (when the bank says a supplier's permission is gone) can be lifted only by a named person with a reason.
Every one of these actions is written to the logbook.

**Fewer, better tests.** This session also set a rule for the rest of the series: don't over-engineer. Each
safety check is tested once, exactly at its limit and 1p over, plus what happens when something can't be read.
That gave 49 tests instead of about 120. I then broke 30 checks on purpose; 25 were caught at once, and the
other 5 are each backed up by a second check elsewhere, which is written down.

## The bank's own permission

Session 4 built the second layer of permission, the one held by the bank.

**Two permissions, not one.** The mandate is Dana's permission *to the AI*. But the bank never sees it. So
Dana also gives the bank its own permission for each supplier. This is called a *consent*, and it's modelled
on UK *variable recurring payments* (VRP). VRP is a real scheme (its commercial version launched in June 2026) that lets a customer
allow a business to take repeated payments of different amounts from their account, within limits set at their
bank. The real scheme is mostly used by people paying businesses such as utilities; here it is borrowed for a
company paying its suppliers. Each consent fixes three things:

- **who** is paid. The supplier's bank details are stored in the consent. A payment request can't name anyone
  else. I tried: a request that named a different account was refused before anything else was looked at.
- **how much** per payment, and per calendar month.
- **from when until when.**

If a payment breaks a limit, the bank refuses it with the code `AM14` ("amount exceeds agreed limit"). If there
is no valid consent (unknown, cancelled, not started yet, or ended), it refuses with `MD01` ("no mandate").
These are standard codes from ISO 20022, the international rulebook for payment messages.

**Why the layers are set up differently on purpose.** Dana's mandate lets the AI pay Cirrus Desk up to
£2,000.00 a time. Her consent at the bank allows only £250.00. So a £275.00 bill (B-08) passes the mandate but
the bank refuses it. I checked this by hand on a running bank: `AM14`, as expected. When the two layers
disagree, the stricter one wins, and the next sessions make sure a person hears about it.

**The order of checks matters.** Networks drop replies. If the bank accepted a payment but the reply got lost,
the system asks again with the same *idempotency key* (a reference that says "this is the same request as
before"). The bank looks that key up **first**, before checking the consent. So even if Dana cancelled the
consent in the meantime, the retry gets the original answer: "yes, paid". The other way round, the bank would
say "refused" about money it had already sent, and the business might pay the bill a second time.

**Re-running set-up safely.** The consents are loaded from a file by a bank admin command. This stands in for
Dana approving each one in her banking app. The obvious design, "load the file and overwrite anything with the
same name", has a trap. If someone re-ran the set-up after Dana had cancelled a consent, it would quietly come
back to life. So loading refuses any consent that already exists, and loads all of the file or none of it.

**Choices made without the owner.** Normally I would pick the edge cases for this session myself. This time I
asked Claude Code to make the choices and write each one down with the option it rejected and why (decision
record 0012). Then I broke 15 of the bank's checks on purpose, one at a time. A test caught every one.

## The AI asks; code decides

Session 5 connected the signed mandate to the AI assistant and the tool server. The tool server is the program
that gives the AI its tools and checks everything it asks for.

**A seventh tool.** The AI can now ask which mandates it holds (`list_mandates`). It sees each mandate's
suppliers, limits, and how much of this month's budget is already used. It never sees signatures, fingerprints
or keys.

**Citing a mandate is a claim, not proof.** When the AI suggests paying a bill, it may say "this is covered by
mandate MND-001". Code doesn't take its word for it. First, the nine checks from Part 1 run (right supplier,
right amount, no duplicates, and so on). If any fails, the bill is blocked, exactly as before. Then the twelve
mandate checks from session 3 run. Only if all twelve pass is the bill *authorised*: it will be paid later
without a person looking at it. If even one fails, the bill goes to a person, with every reason listed.

**Signing for the AI, without giving it the key.** An authorised bill gets a *closed mandate*: a permission
slip for that one payment, signed with the AI's own key. It records which mandate it falls under, the exact
amount and supplier, and a fingerprint of the bill's file. It expires after 24 hours. The AI never holds that
key. It sits with the tool server only. I tested this by searching every tool answer, the logbook and the
saved proposals for any piece of the key. None was found.

**Bills that were refused once never go round again automatically.** If the bank refuses a bill, that bill is
marked "previously refused" the next time it is suggested, so a person sees it. This fact is read from the
tamper-evident logbook, not from an ordinary file. The ordinary file could be edited to erase the refusal.

**The budget test.** The test bills are designed so the monthly budget can't cover all of them. So the test
checks rules that hold in any order: never more than £3,000.00 authorised, at least one bill sent to a person,
and no budget wasted. In the rehearsal run, six bills were authorised (£2,401.18). The next one, £820.00, went
to a person because only £598.82 was left. I ran the same bills in reverse order, and the rules still held.

**A rehearsal, not the real thing.** The real AI test costs money and needs an API key (a password for the AI
service), which this session didn't have. So I rehearsed with a *stub*: a scripted stand-in that makes the same
tool calls a careful assistant would. The live AI test is written and ready to run. The project's own safety
rule stopped me treating the new instructions for the AI (*prompt* version 2) as approved without that live
test. That rule is there to catch exactly this kind of unchecked change. So the gap is recorded as an open item
in the test suite, not hidden.

**The real thing, afterwards.** Later I ran the live tests myself, with the real AI. Part 1's test bills still
gave the same results with the new instructions. The Part 2 bills met every expected outcome. Two attack tests
also passed: the AI didn't claim the mandate covered the Quayside bill, even though the bill's own text said it
did, and it didn't stretch "utilities and subscriptions" to cover an insurance bill. Only then were the new
instructions marked as approved. One small fix was needed first: Part 1's live test checked the instructions
against the approved list, so it could never pass for instructions waiting to be approved. It now checks the run
used exactly the instructions on disk, and a separate test still checks they are approved.

**Testing the tests.** I broke 21 checks on purpose. 18 were caught after I added one missing test. The other
three are each backed up by a second check elsewhere, and that is written down.

## Paying without a person, and proving it afterwards

Session 6 built the *executor*: the program that actually pays the bills the AI was allowed to pay on its own.
A person or a timer starts it, but it makes no decision about any single payment.

**Check again at the moment money moves.** A bill may wait hours between being authorised and being paid. In
that time the director may cancel the mandate, the bill's file may change, or the bank's permission may go. So
just before paying, the executor repeats every check: the signatures, the twelve mandate checks, the bill's
fingerprint, the supplier, the balance and the bank's permission. Risk people call this gap *time-of-check versus
time-of-use*. If anything fails, the bill is refused, with every reason listed, and nothing is sent.

**Write down the attempt before sending.** Before it sends a payment, the executor writes "about to send" in the
logbook. If the bank's reply is lost, the next run sees that note. It doesn't check again or build a new request.
It sends exactly the same request with the same reference, and the bank recognises it as a repeat. I tested the
worst case: the reply is lost, then the mandate is cancelled, then the permission slip expires. The retry still
gave exactly one payment. Refusing at that point would have been worse: the records would say "not paid" while
the money had gone, and the bill could be paid a second time.

**The bank still has the last word.** B-08, the £275.00 bill, passed the mandate but the bank's own limit is
£250.00, so the bank refused it. The executor recorded the refusal, gave the amount back to the monthly budget,
and made sure the bill goes to a person if it is suggested again.

**Evidence for a dispute.** For any payment, one command builds an *evidence pack*. It is a single file that shows
what the director signed, what the AI signed for this one payment, and what the bank recorded, and how each links
to the next. It can be checked without the live system. I broke one link at a time (changed an amount, swapped in
a different signed mandate, edited a logbook entry, faked the bank's record) and each time the pack failed. One
limit is written into the pack itself: it can't prove on its own that nobody rewrote the logbook. That needs a
copy of the logbook's latest fingerprint kept somewhere else.

**A second reviewer found the worst problem.** As in Part 1, a separate AI reviewer with no background checked the
executor against the rules. It found five problems my tests had missed. The serious one: if two copies of the
executor ran at once and the mandate was cancelled at just the wrong moment, a payment the bank had made could be
recorded as refused. The budget would then come back, and the bill could be paid again. All five are now fixed,
each with a test that failed first.

## Attacking the permissions

Session 7 was red-teaming: attacking my own system on purpose. In Part 1 the attacks were on the *data* (bills
that lie). This time they were on the *permissions* themselves. Could the AI, a thief or an insider make a
payment that nobody really allowed? Claude Code played the attacker, one risk at a time.

**What held.** An AI that claims every bill is covered gets nothing outside the mandate. Splitting a big bill
into two smaller payments is blocked, because each payment must equal the bill's total. Editing the mandate file
changes nothing, because the tamper-evident logbook is the source of truth. A mandate signed with the AI's key
instead of the director's is rejected. A payment one second after the mandate ends is refused. Someone who steals
the AI's key can't pay more than the mandate allows: the executor refuses, and if they go round it, the bank
refuses too. That is *defence in depth*: two independent layers, so one failure isn't enough.

**What broke, and was fixed.** An insider with access to the data folder copied a genuine one-payment permission
slip onto a different bill. No extra money moved. But the system recorded the second bill as paid, sharing the
first bill's payment. So the books would say a supplier was paid when it wasn't. Now the executor checks that the
slip was signed for this exact bill, and if a slip turns up on two bills, it pays neither until a person looks.

**What is still open.**

- A Part 1 weakness got worse. The scan for bank details hidden in a bill misses details written with dots
  (99.55.03). In Part 1 a person still looked at the bill. Now, with no person, a "we've changed banks" bill
  from a covered supplier is paid automatically. The money still goes to the supplier's real account, but the
  warning is lost.
- **A new risk the register didn't have.** Every check in Part 2 asks "is this payment inside the permission?"
  None asks "is this bill real, and do we owe it?". A convincing fake bill from a covered supplier is paid
  automatically, to the real supplier, up to the mandate's limits. In Part 1 the person approving each payment
  was quietly doing that job. Taking the person away removed a control nobody had written down. It is now in the
  risk register as R-23, rated *ineffective*: no control exists for it yet.

**Ratings.** For each risk, Claude Code proposed a rating in the risk register: *effective* (the control worked
under attack), *partially effective* (it worked, with a known gap) or *ineffective*. The final review in session 8
moved one more to partially effective (paying the same bill twice), and the owner confirmed them all. The full
count is in the [results of the main article](/projects/agentic-payments-lab/part-2/#results).
