# QA Run — donavenezuela.com — Full regression (staging → prod readiness)

- **Date:** 2026-06-29
- **Target:** `https://staging.donavenezuela.com` (branch `staging`)
- **Build:** `391bec9` (Merge #108 — security headers restored)
- **QA:** Simonethg
- **Scope:** Full regression for **promotion to production**, with focus on the
  merged blockers (#80 / #81 / #82) and the QA payment bypass prod-safety.
- **Verdict:** ✅ **GO (frontend)** — full frontend regression PASSES. Promotion is
  approved **subject to the pre-prod checklist below** (real end-to-end payment
  validation + 1 backend item are owned by Ibis and cannot be closed by frontend QA).

> Mobile evidence is emulated (`mobile-emulated`) — provisional until a real-device pass.
> Payment SAFETY: QA never completes a real charge; payment paths are verified by form
> render + method wiring + loading/error states, not by paying.

---

## Test case results

| ID | Criterion | Result |
|----|-----------|--------|
| TC-01 | Donation form renders: amounts $10/$25/$50/$100/Otro, Método dropdown, Tipo de ayuda, Destino, Donar | ✅ PASS |
| TC-02 | Default method = Binance Pay; method list complete (Tarjeta, PayPal, Transferencia, Binance, Crypto/Meru, Ecuador·Bre-B) | ✅ PASS |
| TC-03 | Method wiring (code-verified): Binance/Koywe → usaibis; card/paypal/bank-transfer → Donorbox; crypto → Meru; Colombia/Ecuador → donor dialog → Loopay | ✅ PASS |
| TC-04 | **#81** Meru checkout: loading spinner + 15s timeout + error/retry overlay | ✅ PASS (shipped & verified) |
| TC-05 | **#80** Loading feedback: Donorbox form + donor-dialog show real steps; QA sim button at the payment step | ✅ PASS |
| TC-06 | **#82** Donor-dialog backdrop does not close while submitting (race fixed) | ✅ PASS (code + build) |
| TC-07 | i18n — es/en/pt/ja render, no raw keys, no `undefined`, persists across reload | ✅ PASS |
| TC-08 | /gracias — 200, renders, share action present, amount + institution params render | ✅ PASS |
| TC-09 | Responsive desktop 1440×900 + mobile 393×852 — no overflow/clipping | ✅ PASS (mobile-emulated) |
| TC-10 | OpenGraph/SEO — title, description, OG image resolves, html lang | ✅ PASS |
| TC-11 | Console/runtime health — 0 JS errors, 0 failed/critical requests (both breakpoints) | ✅ PASS |
| TC-12 | **QA bypass prod-safety**: dormant without `?qa=1`; triple-locked (env + non-prod host + `?qa=1`); env-off build renders nothing | ✅ PASS |

No Critical/Major/Minor defects on the frontend regression.

---

## Observations (non-blocking for prod)

- **OBS-1 — Donorbox form reload on staging (NOT a prod issue).** The Donorbox form
  reloads in a loop on `staging.donavenezuela.com` (4×) but loads once on
  `www.donavenezuela.com` (1×) — isolated to the **Donorbox campaign domain
  allowlist** (staging domain not allowed), not our code or the security headers
  (both ruled out by test). **Does not affect prod.** Optional: add
  `staging.donavenezuela.com` to the Donorbox campaign's allowed domains for QA.
- **OBS-2 — CSP is Report-Only and misses some 3rd-party domains** (js.stripe.com,
  paypal.com, cdn.jsdelivr.net, rsms.me, vercel.live). Report-only → does **not**
  block today. **Before switching CSP to enforcing**, add these to the allowlist or
  card/PayPal payments will break.
- **OBS-3 — `NEXT_PUBLIC_QA_BYPASS`** must remain **unset in the Production
  environment** (it is Preview-only). The hostname lock blocks `donavenezuela.com`
  regardless, but keep the env off in prod.
- **OBS-4 — CI red:** `lib/donations/mocks.test.ts` expects `hogar-bambi` (now
  `fe-alegria`). Stale test only; no runtime impact. Update expectations.

---

## Pre-prod checklist (owned by Ibis / coordination)

These are **not closeable by frontend QA** and must be confirmed before/at prod:

1. **Real end-to-end payment validation** for each method with live backend/providers
   (Binance, Koywe/PagoMóvil via usaibis, Meru, Loopay/Bre-B, Donorbox + Stripe test
   card). QA validated the UI/wiring/loading states, not real settlement.
2. **Known prod backend bug — Binance success URL:** paying via Binance returns to
   the **old success URL**. Configured in usaibis / Binance merchant (Ibis), not the
   frontend. Fix before relying on Binance in prod.
3. **#67 traceability** (Koywe/PagoMóvil payment status on /gracias) — backend
   dependency, still open.

---

## Go / No-Go

✅ **GO for prod from the frontend side** — full regression passes, blockers
#80/#81/#82 verified, QA bypass is prod-safe, no console errors, i18n/responsive/SEO
clean. Promote `staging` → prod, then run the pre-prod checklist with Ibis (real
payment validation + Binance success URL) before announcing payments as fully live.

---

## Evidence

Desktop & mobile captures (hero, donation form, methods, i18n es/en/pt/ja, /gracias,
scroll-through video) under this run folder.

**Desktop**
![hero](https://raw.githubusercontent.com/Simonethg/donavenezuela-qa-evidence/main/STAGING-FULL-REGRESSION-2026-06-29/desktop/01-hero.png)
![donation form](https://raw.githubusercontent.com/Simonethg/donavenezuela-qa-evidence/main/STAGING-FULL-REGRESSION-2026-06-29/desktop/02-donation-form.png)
![gracias](https://raw.githubusercontent.com/Simonethg/donavenezuela-qa-evidence/main/STAGING-FULL-REGRESSION-2026-06-29/desktop/05-gracias.png)
<video src="https://cdn.jsdelivr.net/gh/Simonethg/donavenezuela-qa-evidence@main/STAGING-FULL-REGRESSION-2026-06-29/desktop/scrollthrough-desktop.mp4" controls></video>

**Mobile** (`mobile-emulated`)
![hero](https://raw.githubusercontent.com/Simonethg/donavenezuela-qa-evidence/main/STAGING-FULL-REGRESSION-2026-06-29/mobile/01-hero.png)
![donation form](https://raw.githubusercontent.com/Simonethg/donavenezuela-qa-evidence/main/STAGING-FULL-REGRESSION-2026-06-29/mobile/02-donation-form.png)
![gracias](https://raw.githubusercontent.com/Simonethg/donavenezuela-qa-evidence/main/STAGING-FULL-REGRESSION-2026-06-29/mobile/05-gracias.png)
<video src="https://cdn.jsdelivr.net/gh/Simonethg/donavenezuela-qa-evidence@main/STAGING-FULL-REGRESSION-2026-06-29/mobile/scrollthrough-mobile.mp4" controls></video>
