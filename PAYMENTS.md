# How selling works on MedPath

One payment rail: **bank transfer to your e& money account**. No card details are
ever collected, nothing is charged automatically, and no third party holds the
money. What arrives in your account is yours.

---

## The flow the buyer sees

1. **Picks an exam** and presses *Continue to payment*.
2. **Gets your account details** and a reference like `MP-MDCAT-4K7Q`. The page
   asks them to put that reference in the transfer note.
3. **Sends the money**, then presses *I have sent PKR 3,500 — confirm my payment*.
4. **Lands on a waiting screen.** It says, in plain words: *"Waiting for your
   payment to be processed"* — their transfer is being checked against the e&
   money account, and the exam unlocks automatically once it is received.
5. **The screen checks every four seconds by itself.** They are told they can
   leave it open and that no refresh is needed. There is also a *Check again*
   button for impatience.
6. **You confirm the transfer. Their screen changes on its own** to *Payment
   confirmed — MDCAT is open*, and the course is unlocked for the full term.

Nothing is open before you confirm. Someone who taps the button and never sends
the money sits on the waiting screen and gets nothing.

## What you do, each time

1. Open the e& money app. Look for a transfer with that reference in the note.
   **Check the amount matches** the exam's price.
2. Run:

   ```
   node tools/reply.mjs MP-MDCAT-4K7Q mdcat https://your-site.vercel.app
   ```

   It checks the reference belongs to that exam, prints a ready-to-paste reply
   containing a signed link, and self-checks the link before you send it.
3. Paste the reply into the email thread.

That's it — two minutes, and only when money actually arrives. The buyer's open
waiting page usually flips to *Payment confirmed* before they finish reading your
email.

If they have closed the tab, the link opens it for them. If their device is
different from the one they ordered on, the link still works — it is signed for
that exam and that order specifically.

## Why the link is safe

- It is signed with a key that never leaves your machine.
- It is tied to **one exam** and **one order**. Editing the exam in the URL makes
  it invalid.
- It expires after **72 hours** (see `ACTIVATION_LINK_HOURS` in `lib/store.js`).
- Using it twice is harmless: the second use just shows the same success screen
  and does not extend anything.

## Things that protect you

- **No licence without your confirmation.** A buyer cannot unlock anything by
  pressing a button, guessing a URL, or editing their device clock — the licence
  is sealed and signed at the moment it is issued.
- **References are unique per order**, so two people cannot share one.
- **The exam is fixed at purchase.** A confirmed MDCAT payment cannot open USMLE.
- **Unconfirmed orders simply lapse.** Nothing needs cancelling.

## If something goes wrong

| Situation | What to do |
|---|---|
| Money arrived, no email from the buyer | You have no reference, so you cannot match it. Ask them for the reference shown on their waiting screen. |
| Buyer paid the wrong amount | Confirm anyway, or ask for the difference — your call. Nothing in the system forces either. |
| Buyer says they paid, but nothing is in the account | Do not send a link. Their waiting screen stays locked, which is correct. |
| Buyer lost their waiting screen | Send the link — it opens the exam regardless of which page they are on. |
| You sent a link to the wrong person | They can only open the exam named in the link. Nothing else is exposed. |

## The transfer details — copied from the app, not guessed

Take these from the e& money app's own IBAN screen and never from memory:

| Field | Value |
|---|---|
| Account holder's name | `MUHAMMAD TALHA ABDUL REHMAN` |
| IBAN | `AE36 0355 6400 1222 8959 306` |
| SWIFT / BIC | `NBADAEAA` |
| Bank name | First Abu Dhabi Bank (FAB) |

**The SWIFT is `NBADAEAA`, which is FAB's code.** It is *not* `EBILAEAD` —
that is e& money's own corporate code and it was on the site for a while. The
IBAN is a FAB account (bank code `035` is FAB), so an international transfer
routed with `EBILAEAD` is pointed at the wrong institution. The IBAN itself
passes the standard mod-97 checksum, and the test suite now verifies both.

If e& money ever changes the IBAN shown in the app, change the IBAN **and** the
SWIFT together, and re-run the suite — a checksum failure will stop the build.

## The one thing that stops payments: currency

**Your e& money account is a UAE dirham wallet. It only accepts AED.**

It is not a credit card machine and it cannot take a card payment. A buyer cannot
type their card number anywhere on MedPath, because there is nowhere to type it.
Money reaches you as a **bank transfer into the IBAN** — which is a different
thing from paying by card:

- **From a UAE bank app:** add the IBAN as a beneficiary and send the dirham
  amount. Arrives in seconds. Free.
- **From abroad:** an international transfer to that IBAN with SWIFT
  `NBADAEAA`, sent **in AED**. Takes one to two working days and their bank may
  add a fee.
- **By credit card:** not a normal payment. Banks treat a card-to-account
  transfer as a cash advance, with interest and a fee. Buyers should not do it.

This used to be a real problem: nine of the eleven exams are priced in rupees,
euros, dollars, pounds or rupees, and the old checkout asked buyers to send
"$39.99" to a dirham wallet that would reject it. **That is now fixed** — the
checkout converts every price into the dirham figure and asks for that instead.
See `aedLabel()` in `lib/fx.js`.

| Exam | Priced at | Buyer is asked to send |
|---|---|---|
| MDCAT | PKR 3,500 | AED 46.41 |
| IMAT | €34.99 | AED 143.70 |
| FCPS 1 | PKR 7,500 | AED 99.44 |
| FCPS 2 | PKR 10,000 | AED 132.59 |
| USMLE Step 1 | $39.99 | AED 146.86 |
| USMLE Step 2 CK | $69.99 | AED 257.03 |
| PLAB 1 | £34.99 | AED 172.03 |
| PLAB 2 | £44.99 | AED 221.19 |
| EMREE | AED 99.99 | AED 99.99 |
| JEE | ₹1,500 | AED 57.89 |
| NEET | ₹1,500 | AED 57.89 |

**Keep the rates current.** They live in `config.js` under
`BRAND.payment.fx.perAed`, with an `asOf` date. A rate that is months out of date
means buyers send slightly the wrong figure. Refresh it every few months — the
test suite fails if any rate looks inverted or absurd.

### What you have to watch

Because the prices are quoted in five currencies and paid in one, **the amount
that lands will not always be round**, and it will drift as rates move. When you
confirm a transfer, check the figure is roughly right rather than exact — the
checkout's tolerance is 3%.

## What the site claims about e& money — and why it is worded so carefully

The checkout carries a short "Why this account is safe" panel. Its three lines
are deliberately precise, because getting this wrong on a payment page is a
legal risk, not a marketing opportunity:

> - e& money is licensed and regulated by the Central Bank of the UAE.
> - Your virtual IBAN is issued by First Abu Dhabi Bank PJSC (FAB).
> - Your transfer goes to a real, named UAE account — never to a card form on this site.

**Both are true, per e& money's own terms.** Their operator, Digital Financial
Services LLC (registration 815764), is licensed by the UAE Central Bank as a
stored value facility, and their terms state: *"we may provide you with a virtual
IBAN issued by First Abu Dhabi Bank PJSC ("FAB")"*.

**What is NOT true, and must never go on the site:**

- ❌ "e& money is a bank" — it is a wallet under a stored-value licence, not a bank.
- ❌ "e& money is owned by FAB" — it is owned by e& (formerly Etisalat). FAB issues the IBAN.
- ❌ "e& money is a subsidiary of First Abu Dhabi Bank" — same error.

If a buyer asks, the honest sentence is: *"e& money is a UAE wallet regulated by
the Central Bank; the IBAN on the account is issued by First Abu Dhabi Bank, one
of the largest banks in the UAE."* That is both accurate and reassuring, which is
why the panel says exactly that and nothing more. The test suite fails if the
copy ever drifts into an overclaim.

## The account name matters as much as the IBAN

The e& money account is held personally, in the name **MUHAMMAD TALHA ABDUL REHMAN** — not
"MedPath". That is fine, and the site now says so plainly, but get these right:

- **The name on the site must be exactly the name on the wallet**, spelling
  included. UAE banks match beneficiary name against IBAN, and a transfer whose
  name does not match can be **held or returned**. `BRAND.payment.holder` in
  `config.js` is the single place this lives.
- **Buyers may hesitate** when they see a person's name instead of a business.
  The checkout now explains it in one line — "MUHAMMAD TALHA ABDUL REHMAN is the
  owner of MedPath, this is his personal e& money account, and it is where every
  payment goes" — and tells them to use that name, not "MedPath", when adding the
  beneficiary. There is a Copy button beside it so nobody mistypes it.
- **If you ever open a business account in the company's name**, change
  `BRAND.payment.holder` and `iban` together. Never change one without the other.

If in doubt, open the e& money app and copy the account name and IBAN straight
from the account details screen rather than from memory.

## Where the money lands

e& money account, **MUHAMMAD TALHA ABDUL REHMAN**, IBAN `AE36 0355 6400 1222 8959 306`,
SWIFT `NBADAEAA`.

Buyers outside the UAE will usually be quoted international transfer fees by
their own bank. That is between them and their bank; MedPath charges one price.

## Tuning

- `TRIAL_DAYS` and the trial helpers still exist in `lib/store.js` for devices
  that bought under the older three-day-pass flow. New purchases do not use them.
- The confirmation link's lifetime is `ACTIVATION_LINK_HOURS` in `lib/store.js`.
- Prices and terms live in `config.js` under `PRODUCTS`.
