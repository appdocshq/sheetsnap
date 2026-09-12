# `site/` — the public pages, ready to publish

Four files. Copy all four to GitHub Pages and you are done.

```
index.html    a real landing page, so the other two are not orphans
privacy.html  REQUIRED — App Store Connect will not accept a submission without it
support.html  REQUIRED — the Support URL field
style.css     the app's own palette
```

Static, self-contained, no build step. **No JavaScript, no external fonts, no
analytics, no cookies** — which is also why neither page needs a cookie banner
or a cookies section: there is nothing to disclose.

**Everything is filled in.** Written by Asieuzzaman Wasir, © 2026, dated
12 September 2026, support at `appsupport.get@gmail.com`. There are no
placeholders left:

```bash
grep -c 'todo' site/*.html      # 0, 0, 0
```

---

## The support address

`appsupport.get@gmail.com`, written into `privacy.html` §9 and `support.html`,
and set in `Config/Secrets.xcconfig` so the app's **Contact support** row in
Settings uses the same one. Three places, one address.

**Make sure it is actually monitored** — forwarding it to whatever inbox you
read is enough. Apple checks that the Support URL resolves and a reviewer may
write to it, and a bouncing support address is how you get one-star reviews you
never see the cause of.

### Why not your App Store Connect account email?

You asked; the answer is **don't**, for two reasons.

1. **It is your developer account login.** Publishing it on a page every user
   and every scraper can read hands out half of the credentials to the account
   that owns your app. Apple ID phishing aimed at developers is a real and
   targeted thing, and the address is the target.
2. **It is where Apple sends account-critical mail** — expiring certificates,
   agreement changes, review outcomes. Mixing that with "my export won't sum"
   means the important one gets buried.

A free dedicated mailbox costs nothing and solves both. Create it, **set it to
forward to whatever inbox you actually read**, and you get the separation
without checking a second inbox.

`wasir@cefalo.com` is the other thing to avoid here: it is a company domain, so
a personal app's support contact would read as your employer's, and it stops
working the day you leave.

### To change it later

One command, from the repo root — it catches the pages and the docs that quote
them:

```bash
/usr/bin/sed -i '' 's|appsupport\.get@gmail\.com|YOUR@ADDRESS.COM|g' \
  site/*.html site/README.md BLOCKERS.md docs/SUBMISSION-STEPS.md \
  docs/terms-eula-draft.html
```

Then the same value in `Config/Secrets.xcconfig`, which is what reveals the
**Contact support** row in the app's Settings with the version already in the
subject line. Without it the row is hidden rather than opening a dead link.

---

## Publishing it

### ⚠️ Do not enable GitHub Pages on this repo's `docs/` folder

GitHub Pages can only serve a branch **root** or a folder called **`docs/`**.
This repo's `docs/` holds the engineering specs, `BLOCKERS.md`, the submission
steps and the unpublished EULA draft. Serving it would put all of that on the
public internet.

### Option A — a separate repository (simplest)

1. Create a new **public** repo, e.g. `sheetsnap-site`.
2. Copy `index.html`, `privacy.html`, `support.html` and `style.css` into its
   root. Nothing else.
3. Settings → Pages → Source: *Deploy from a branch* → `main` → `/ (root)`.

URLs: `https://<you>.github.io/sheetsnap-site/privacy.html`

### Option B — a `gh-pages` branch in this repo

```bash
git subtree push --prefix site origin gh-pages
```

Settings → Pages → Source: *Deploy from a branch* → `gh-pages` → `/ (root)`.
Re-run that one command whenever you change a page.

URLs: `https://<you>.github.io/sheetsnap/privacy.html`

⚠️ After the first deploy, **check that
`https://<you>.github.io/sheetsnap/docs/BLOCKERS.md` returns a 404.** If it does
not, Pages is serving the wrong branch and your specs are public.

### Either way

Wait for the first build to finish, then **open both pages in a private window**
before you paste the URLs anywhere. GitHub Pages takes a minute or two and fails
silently if it fails at all.

---

## What goes where in App Store Connect

| Field | Value |
|---|---|
| Privacy Policy URL (App Information) | `…/privacy.html` |
| Support URL (the version page) | `…/support.html` |
| Marketing URL | optional — `…/index.html` if you want one |
| License Agreement | **leave it on Apple's standard EULA** |

And in `Config/Secrets.xcconfig`, ⚠️ **without a scheme and with no `//`
anywhere on the line** — in an xcconfig file `//` starts a comment mid-line, so
`PRIVACY_POLICY_HOST = https://example.com/privacy.html` silently becomes
`https:`. This has cost this project four phases once already (BLOCKERS.md B3).
`AppConfig` re-attaches `https://` for you.

```
PRIVACY_POLICY_HOST =yourname.github.io/sheetsnap-site/privacy.html
SUPPORT_EMAIL =appsupport.get@gmail.com
```

Leave `TERMS_HOST` empty — see below. Then run `./scripts/build.sh`, which exits
3 if a value you set here does not reach the build.

---

## About the Terms of Use

**Sheetsnap ships under Apple's standard EULA, and that is a decision rather
than a gap.** App Review accepts it, `AppConfig.termsURL` falls back to it
whenever `TERMS_HOST` is unset, and all three pages here link to that same
document. One EULA, in one place, that cannot go stale.

A custom one is drafted at **`docs/terms-eula-draft.html`**. It is in `docs/`
precisely so it cannot be published by accident: it still contains
`[YOUR ADDRESS]` and `[YOUR COUNTRY / STATE]`, and a live page showing those is
worse than no page. Its own header comment lists the five steps to adopt it.

If you ever do: **its section 10 is the part not to edit away.** Those are the
minimum terms Apple requires a custom EULA to contain (Schedule 1 of the
Developer Program Licence Agreement) — the acknowledgement that the agreement is
not with Apple, scope of licence, maintenance and support, warranty, product
claims, intellectual property, legal compliance, your name and address,
third-party terms, and Apple as a third-party beneficiary. Deleting any of them
is grounds for rejection.

---

## ⚠️ These are drafts, and they are not legal advice

Every factual claim was written against what this codebase actually does, and
each is checkable against the source — the privacy policy's "what we keep" table
is a row-by-row transcription of the real `extraction_logs` columns, and
"no name, no email" is `AuthStore.configure` setting `requestedScopes = []`.

That makes them **accurate**. It does not make them **sufficient in your
jurisdiction**. Have somebody qualified read the privacy policy before you
publish it, particularly if you sell in the UK or the EU.

---

## Keeping them true

These pages make specific promises, and specific promises go stale.

- **The FAQ in `support.html` is the same ten answers as the app's own
  Settings › FAQ** (`sheetsnap/Features/Settings/FAQView.swift`). Change one,
  change both. The in-app one is read first, so fresh here and stale there is
  the wrong way round.
- **The data inventory in `privacy.html` §2 is a table of real database
  columns.** Add a column to `extraction_logs` and it belongs in that table —
  and if the column could hold receipt content, the column is what is wrong.
- **Adding an SDK changes three things on the same day**: this page,
  `sheetsnap/Resources/PrivacyInfo.xcprivacy`, and the App Store privacy labels.
  A manifest that disagrees with the labels is a rejection. PostHog and Sentry
  are the live example — both named in the stack, neither linked, and
  BLOCKERS.md B19 is the decision.
- **The prices** in `index.html` are the ones in `Config/Sheetsnap.storekit` and
  RevenueCat. Change a price in one place, change it in all three.
- **The claim that Anthropic does not train on API inputs** (privacy §4) is true
  of its commercial API terms as written. It is somebody else's document — read
  it yourself before you publish rather than taking this file's word for it.
