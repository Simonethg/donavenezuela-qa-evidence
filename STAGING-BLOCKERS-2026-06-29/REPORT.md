# QA Run — donavenezuela.com (staging)

- **Date:** 2026-06-29
- **Scope:** Blocker PRs merged to `staging` — #89 (Meru checkout loading/timeout/error, issue #81) and #91 (Donorbox/donor-dialog loading feedback #80 + backdrop close race #82).
- **Target (build):** `staging` @ `37d9b80` → preview `https://donavenezuela-bqxi2c4yj-ibis-1221cf3c.vercel.app`
- **Breakpoints:** desktop 1440×900, mobile 393×852 (emulated — `mobile-emulated`, provisional until a real-device pass)
- **QA:** Simonethg
- **Verdict:** 🐞 **FAIL** — one Major defect found (live on staging at time of QA), **fixed in PR #97** and re-verified on a local production build. PR #97 has since been merged to `staging` (`7132765`).

---

## Test cases

| ID | Criterion | Result |
|----|-----------|--------|
| TC-01 | #80 — Donorbox loading spinner appears immediately on open (Tarjeta/PayPal/Transferencia) | ✅ PASS (desktop + mobile) |
| TC-02 | #80 — Loading overlay **dismisses when the Donorbox form mounts** | 🐞 **FAIL** (overlay stuck; see BUG-01) → ✅ after #97 |
| TC-03 | #81 — Meru checkout loading/timeout/error overlay shipped & wired | ✅ PASS (code shipped to bundle; overlay driven by SDK `onReady`, not the #80 pattern). Not exercised live to avoid creating a real payment session. |
| TC-04 | #82 — Backdrop does not close the donor dialog while submitting | ✅ PASS (code + build; sub-100ms race not manually observable) |
| TC-05 | Donation form: Binance Pay default, amounts $10/$25/$50/$100, method list correct | ✅ PASS |
| TC-06 | i18n — es/en/pt/ja render, no raw keys, no `undefined`, persists across reload | ✅ PASS |
| TC-07 | /gracias — reachable (200), share action present, amount/institution params render | ✅ PASS |
| TC-08 | Responsive layout desktop + mobile, no horizontal overflow | ✅ PASS |
| TC-09 | OpenGraph/SEO — title, description, OG image resolves, html lang | ✅ PASS |
| TC-10 | Console/runtime health — no uncaught errors on the home path | ✅ PASS (0 console errors; see OBS-01 for a report-only CSP notice on the card flow) |

---

## BUG-01 — Donorbox loading overlay never dismisses (stuck over the loaded form)

- **Severity:** Major
- **Environment:** desktop 1440×900 & mobile 393×852, staging preview (`37d9b80`)
- **Preconditions:** Donation form open, method = Tarjeta / PayPal / Transferencia bancaria.
- **Steps:**
  1. Open the donation form, select "Tarjeta".
  2. Click "Donar" → the Donorbox dialog opens with a loading spinner.
  3. Wait.
- **Expected:** The spinner dismisses once the Donorbox form mounts, revealing the donation form.
- **Actual:** The white loading overlay stayed up permanently, visually covering the fully loaded form. Iframe mounted at **~1968ms**; overlay still visible at **16s** (`overlayDismissedAtMs: null`).
- **Root cause:** The `MutationObserver` effect in `app/donorbox-dialog.tsx` depended on `[open, …]` and ran once while `hasOpened` was still `false`, so the container ref was `null` and the observer/timeout were never attached; the effect never re-ran.
- **Fix:** PR #97 — gate the effect on `open && hasOpened` and add `hasOpened` to the deps. Re-verified on a local production build: overlay dismisses at **~5125ms** via the observer (`dismissedViaTimeout: false`); form renders.
- **Evidence:** `donorbox-observe/desktop/80-donorbox-loaded.png` (bug — stuck spinner), `donorbox-fix-verify/spinner-on-open.png` + `donorbox-fix-verify/form-loaded-overlay-gone.png` (after fix).
- **Failing test case:** TC-02.

---

## OBS-01 — CSP report-only: js.stripe.com not in `frame-src` (non-blocking)

- **Type:** Observation (report-only; no functional impact today). Pre-existing, not from the blocker PRs.
- **Detail:** Opening the Donorbox card form logs: `Framing 'https://js.stripe.com/' violates the following report-only Content Security Policy directive: "frame-src 'self' https://checkout.meru.com https://donorbox.org"`. Because it is **report-only**, Stripe still frames and card entry works.
- **Risk:** If the CSP is ever switched to enforcing, card payments via Donorbox/Stripe would break. Add `https://js.stripe.com` (and any Stripe frame origins Donorbox uses) to `frame-src` before enforcing.

---

## Evidence index

- `desktop/` & `mobile/` — regression captures: hero, donation form, methods, i18n (es/en/pt/ja), /gracias, scroll-through mp4.
- `donorbox-observe/{desktop,mobile}/` — Donorbox dialog open: spinner-on-open (PASS) and loaded state (showed BUG-01 before the fix).
- `donorbox-fix-verify/` — after #97: spinner on open, then form rendered with the overlay gone.

> Mobile media is emulated (`mobile-emulated`) — provisional until a real-device pass.
