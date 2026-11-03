# Getting the money — the honest options

You asked: does Stripe work in the UAE, and can it pay into e& money?

**Short answer: yes, Stripe UAE works — but it will not pay into your e& money
wallet, and getting a Stripe UAE account costs real money every year before you
earn a rupee.**

Verified against Stripe's own help pages on 30 October 2026.

---

## 1. What Stripe UAE actually requires

Stripe supports the UAE. The supported business types are companies, LLCs,
single-member LLCs, sole establishments, free zone establishments, branches of
foreign companies, and partnerships.

Then this line, which decides everything:

> "You also must have a trade license or freelancer permit issued in the UAE to
> be able to use Stripe. We don't currently support individuals without a trade
> license/permit."

And the documents you upload:

- Trade licence **or freelancer permit**, issued in the UAE
- Memorandum of association, showing who owns what
- **Proof of an active bank account — a bank statement dated within the last six
  months**

So the wall isn't "Stripe doesn't support UAE". It's **"show us a UAE licence and
a UAE bank statement."** No licence, no account. There is no individual route.

## 2. Why e& money won't hold the money

e& money is licensed by the UAE Central Bank as a **stored value facility** — a
prepaid wallet with an IBAN attached. That is not the same thing as a bank
account, and the difference is exactly what Stripe's rules test:

- Stripe requires **a bank statement** proving an active account. A wallet gives
  you no bank statement.
- Stripe requires the payout account to **match the licensed business name**. A
  personal wallet is in your own name, not the company's.
- Stripe's UAE onboarding checks that bank account details **match your trade
  licence**.

Could a UAE wallet IBAN occasionally pass an IBAN-shape check? Possibly. But the
statement requirement is not a formatting check — it is a document upload. This
is the one part of your plan I would not build on.

**Practical version: you open a business account at a UAE bank (Wio, Mashreq
NeoBiz, or similar) as part of company formation. Stripe pays there. That account
comes with a card, so you move money to e& money yourself in seconds.**

That's your chain working — Stripe → UAE bank → e& money. Just with one extra
link, and that link is not optional for anyone.

## 3. What it costs

The cheapest legitimate permit is **SHAMS (Sharjah Media City) at AED 5,750 a
year**, licence only. Realistic first-year numbers:

| Item | AED |
|---|---|
| SHAMS freelance permit (basic, no visa) | 5,750 |
| Trade name reservation | 620 |
| Establishment card (usually needed) | 2,000 |
| Bank account setup | 0 |
| **Year one, roughly** | **8,370 – 12,000** |
| Every year after | 5,750 – 8,000 |

That's about **$2,300 – $3,300 in year one**, recurring.

Two catches on SHAMS: its activity list is skewed to creative and media work, and
its basic permit assumes you already hold UAE residency by some other route. If
you don't, add a visa (AED 4,500 – 6,500) and medical/Emirates ID (AED 1,200),
which puts year one nearer **AED 17,000 – 19,000**.

And you will need to be physically in the UAE to open the bank account. That is
the part no agent or workaround removes.

**Note the same licence unlocks every UAE gateway** — Telr, Ziina, Tap, Network
International all require it too. So this cost is the price of *any* UAE card
processing, not Stripe's price specifically.

## 4. The arithmetic nobody shows you

| | Paddle | Stripe UAE |
|---|---|---|
| Upfront | $0 | ~$2,300–3,300 year one |
| Every year | $0 | ~$1,500–2,200 |
| Per sale | 5% + $0.50 | ~2.9% + AED 1, plus 1% on cross-border cards |
| Needs a company | No | Yes |
| Needs to be in the country | No | Yes, to open the bank account |
| Handles sales tax for you | Yes, worldwide | No |
| Payout reaches | Wire or Payoneer to Pakistan | A UAE bank account |

Stripe is roughly **2 points cheaper per sale**. To recover ~$1,565 of fixed cost
at 2 points, you need about **$78,000 of sales a year** before the UAE licence
has paid for itself — before counting flights, renewals and your time.

Below that volume, Paddle is cheaper *and* automatic *and* needs no company.

Above it, the UAE route wins on margin and gives you a real Stripe account you
can also use for other products.

**So: if MedPath is a side project, Paddle. If you are building a business and
you are going to be in the UAE anyway, the licence is worth it.**

## 5. What the site does now

Bank transfer is live and working. Card is written and waiting behind one line in
`config.js`:

```js
gateway: {
  checkoutUrl: "https://your-checkout-url",
  label: "Card",
  verifyUrl: "https://your-site/api/verify",   // optional but recommended
},
```

`checkoutUrl` works with **Stripe Payment Links** directly — set the link's
after-payment redirect to:

```
https://your-site.vercel.app/checkout/mdcat?ref={CHECKOUT_SESSION_ID}
```

Stripe substitutes the session ID automatically. Or use `client_reference_id` if
you prefer; the site reads both.

**What I verified end to end this session** (with a stand-in provider):

- Buyer leaves for the payment page, and **gets nothing if they abandon it** ✓
- Returning with a genuine session unlocks the exam, sealed, correct term ✓
- **Returning with a hand-typed or faked session grants nothing** ✓ and shows
  "we could not confirm that payment" plus the order reference to quote
- The date-and-time warning pops up on a confirmed card payment too ✓
- Repeat returns don't extend the licence twice ✓

That last security point is why `verifyUrl` matters. Without it, the site trusts
the return redirect. With it, the site asks your server "did this session really
get paid?" — and a faked URL cannot mint a licence. A Vercel serverless function
that calls Stripe's API is free and about fifteen lines.

## 6. If you want to move now

1. **Apply to Paddle this week.** Free, no company, no licence, fully automatic.
   Describe the product as "digital educational software" — never "coaching".
2. **In parallel**, if you can get to the UAE, price a SHAMS permit and ask a
   UAE bank about a freelancer business account. If the numbers in section 4 look
   right for your volume, do it.
3. Whichever lands first, paste the URL into `config.js`. Nothing else changes.

There is no configuration where money moves automatically and no institution
ever verifies who you are. Those two things are in tension everywhere in the
world. What you can remove is **your** involvement in each buyer's purchase — and
both routes above do that completely.
