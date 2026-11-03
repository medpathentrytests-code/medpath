# Making MedPath yours — publish, domain, email, payments

The honest, ordered checklist. Read step 3 before you take a single payment.

> **First, a myth to clear up:** you do not "publish on Chrome". Chrome, Safari and Edge are just
> programs that *read* websites. Publishing means putting your files on a computer that is switched on
> 24/7 (a **host**) and pointing a **domain** at it. Chrome is what your students use to open the result.

**Fastest path to a live link (about 3 minutes, no account, no technical steps):**

1. Download **`medpath-site.zip`** from this workspace.
2. Go to **https://app.netlify.com/drop** in Chrome.
3. Drag the zip onto the page (do not unzip it first — Netlify accepts the zip).
4. It uploads, builds nothing, and gives you a URL like `https://calm-medpath-1a2b3c.netlify.app`.
5. Open that link on your phone. That is your website, live, on the internet.

Then claim it with a free account so it does not expire, rename it, and add a domain (steps 1–2 below).

**Want to see it before publishing?** Double-click **`index.html`** inside the folder — the whole site
runs offline in Chrome from your own disk, no server needed.

---

## 1. Publish it (10 minutes, free)

The deployable package is at **`/home/user/publish/`** — 12 files, 7.7 MB, verified behind SPA rewrites.
No build tooling, no unlocked review copy.

### Netlify Drop — fastest, no account needed to start

1. Open **https://app.netlify.com/drop**
2. Drag the whole `publish` folder — or just `medpath-site.zip` — onto the page.
3. You get a live URL immediately, e.g. `https://medpath-xyz123.netlify.app`.
4. Claim the site (free account), then **Site settings → Change site name** → `https://medpath.netlify.app`.

`_redirects` is already inside, so `/exams`, `/exam/mdcat`, `/checkout/imat` and `/app` all work.

### Vercel

**https://vercel.com/new** → deploy → upload the folder. `vercel.json` is included.

### GitHub Pages — free forever, uses your own repo

```bash
cd /home/user/publish
git init && git add . && git commit -m "MedPath"
git branch -M main
git remote add origin https://github.com/<your-username>/medpath.git
git push -u origin main
# then: repo → Settings → Pages → Source: main / root
```

`.nojekyll` is included. GitHub Pages serves a real 404 for deep links, but the `404.html` in the folder
*is* the site, so the router still takes over. Netlify/Vercel are cleaner.

### Your own hosting (Hostinger, cPanel, any VPS)

Upload the contents of `publish/` into `public_html/`. Nothing else is required — no Node, no database.

> **Never upload `medpath2/dist/notes-review.html`.** That build unlocks every chapter for proof-reading.
> `tools/pack.mjs` excludes it and then asserts it cannot be reached.

---

## 2. Put your name on it (a custom domain)

Buy a domain — `.com` is roughly $10–12/year:

| Registrar | Why | Notes |
|---|---|---|
| **Cloudflare Registrar** | cheapest, no markup, free DNS | domain must be added to Cloudflare first |
| **Namecheap** | simple, good support | frequent first-year discounts |
| **GoDaddy / Hostinger** | familiar, pushy upsells | decline everything except the domain |

Then in Netlify: **Domain management → Add a domain → `medpath.com`**, and follow the nameserver/CNAME
instructions. HTTPS is automatic and free; "Force HTTPS" is one toggle.

Names worth checking: `medpath.com` · `medpath.app` · `getmedpath.com` · `medpath.co`.

**Once the domain resolves**, add the absolute URL to the social tags in `medpath2/shell.html`:

```html
<meta property="og:url" content="https://medpath.com/" />
<meta property="og:image" content="https://medpath.com/og.png" />
```

(the relative `./og.png` works on most platforms; the absolute URL is what Instagram, WhatsApp and
LinkedIn reliably fetch). Then `node medpath2/build.mjs && node medpath2/tools/pack.mjs`.

---

## 3. ⚠️ Payments — read this before you charge anyone

**Right now the checkout is a simulation.** The buy button runs a 1.1-second timer and then writes the
licence into the browser's storage (`setTimeout` → `store.unlock()` in `site/src/pages.jsx`). No gateway
is contacted, **no money moves, and nothing arrives in your e& account.** It was built that way because
real card processing needs your own merchant account and secrets, which cannot live in static HTML —
anyone could read them.

So:

### Option A — sell today with manual verification (recommended first)

1. Buyers pay your IBAN `AE36 0355 6400 1222 8959 306` (SWIFT `EBILAEAD`), JazzCash or Easypaisa.
2. They submit the order form.
3. **You** check the statement, then issue the licence.

For that to be honest the site needs one change: the receipt must say "**Pending verification — send your
transfer, then quote this order code**" instead of "Payment received", and unlock must happen only when
you mark it paid. Roughly half a day of work, no third-party approval needed.

### Option B — a real gateway (what "flawless" requires)

You need a **merchant account**:

| Gateway | Cards | JazzCash | PayPal | What you need |
|---|---|---|---|---|
| **Safepay** | ✅ | ✅ | ✗ | business registration, PK bank account |
| **PayFast.pk** | ✅ | ✅ | ✗ | business registration |
| **Stripe** | ✅ | ✗ | ✗ | a company in a supported country (UAE works), not Pakistan personally |
| **PayPal** | ✅ | ✗ | ✅ | PayPal Business; payouts to PK are restricted |
| **2Checkout / Paddle** | ✅ | ✗ | ✗ | they act as merchant of record — easiest for a sole trader |

**PayPal cannot be the only rail** — PayPal does not let Pakistan-registered businesses receive payments
normally. A UAE-based account fits your e& setup; otherwise use a merchant of record like Paddle.

Integration shape once you have keys:

```
POST /create-order   → gateway order id + amount
buyer pays on the gateway's page
POST /webhook        → verify signature → write licence to a real database
/verify-order        → the site asks "is this order paid?" and unlocks
```

That needs a server and a database: a licence in `localStorage` lasts as long as the buyer feels like
keeping it. Clearing browser data loses it, and a determined user can set it by hand.

**Order of operations:** launch with Option A (manual, honest, zero gateway fees), get your first paying
students, then move to Option B when volume justifies the paperwork. Say the word and I'll build Option A
this week.

---

## 4. Email and Instagram

- **Email:** `medpath.enterytests@gmail.com` is already on the site, receipts and terms. Gmail is fine to
  start; later move to `support@medpath.com` (Google Workspace, ~$6/month).
- **Instagram:** `@medpath_enterytests` is already in the footer and contact block.
- **Profile picture:** `medpath2/art/medpath-logo.png` (1024×1024), or the transparent
  `medpath2/art/medpath-shield-512.png` if you want just the shield.

---

## 5. Before you announce it

- [ ] Domain resolves and HTTPS is on.
- [ ] Share the link in WhatsApp to yourself — the preview card shows the shield and the slogan.
- [ ] Read `/terms` on the live site: AI disclosure and "no refunds" are there.
- [ ] Decide the payments route. **Do not take real money while the receipt can still say "Payment
      received" without you being paid.**
- [ ] Send yourself one real test purchase and confirm the expiry date.
- [ ] Back up `publish/` and `medpath2/` to Google Drive + GitHub.

---

## What "owning" it means

| You own | Where it is |
|---|---|
| The domain | your registrar — renew yearly; this is the one thing that can be lost |
| The code and content | `/home/user/medpath2/` — 273 chapters, 12,285 questions, all editable |
| The brand | shield logo, name "MedPath", slogan |
| The hosting account | Netlify/Vercel/GitHub — free tier is enough |
| Student data | **nowhere yet** — licences live in the buyer's browser; that changes with Option B |
