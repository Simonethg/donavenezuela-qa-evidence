# QA Run — donavenezuela.com — 2026-06-28 — 🐞 FAIL

**Target / build:** https://www.donavenezuela.com (production)
**Scope:** Full regression — donation flow, i18n (es/en/pt/ja), /gracias, responsive, OpenGraph/SEO, console health
**QA:** Simoneth · **Viewports:** Desktop 1440×900, Mobile 393×852 (emulated)
**Figma:** N/A · **Method:** Playwright single-pass capture (`donavenezuela-qa` skill), code review of `app/`
**Evidence folder:** [`REG-2026-06-28/`](https://github.com/Simonethg/donavenezuela-qa-evidence/tree/main/REG-2026-06-28)

> Note: this report lives in GitHub while the Notion "QA" (ibis) page is being connected. It will be mirrored there as a QA Run entry.

## Verdict

🐞 **FAIL** — 3 Minor defects (passThreshold = `minor`). The core donation path (Binance Pay) and the 4-language i18n function correctly. No Critical/Major defects.

## Test cases

| ID | Title | Severity | Verdict |
|----|-------|----------|---------|
| TC-2026-06-28-01 | Donation form renders (amounts $10/$25/$50/$100/Otro, method, aid type, destination, Donar) | — | ✅ |
| TC-2026-06-28-02 | Binance Pay (default, active) → live USDT checkout reachable | — | ✅ |
| TC-2026-06-28-03 | Method dropdown lists the offered methods | — | ✅ |
| TC-2026-06-28-04 | Inactive methods (Tarjeta/PayPal/Transferencia) route correctly | Minor | 🐞 |
| TC-2026-06-28-05 | i18n — es/en/pt/ja render, no raw keys, persist in localStorage | — | ✅ |
| TC-2026-06-28-06 | /gracias renders + share action present | — | ✅ |
| TC-2026-06-28-07 | /gracias with return params shows donation summary (amount + institution) | — | ✅ |
| TC-2026-06-28-08 | OpenGraph/SEO meta + image on canonical host | Minor | 🐞 |
| TC-2026-06-28-09 | No console errors / failed requests on critical path (clean load) | — | ✅ |
| TC-2026-06-28-10 | Spanish copy localization quality (diacritics) | Minor | 🐞 |

## Bugs

### Bug #1 — OpenGraph metadata uses non-canonical host
- **Severity:** Minor (SEO)
- **Environment:** Chromium, Desktop + Mobile, https://www.donavenezuela.com
- **Steps:** 1. Load homepage. 2. Inspect `<meta property="og:image">` / `metadataBase`.
- **Expected:** OG URLs use the canonical host `https://www.donavenezuela.com`.
- **Actual:** `og:image = https://donavenezuela.com/opengraph-image?...` and `metadataBase = donavenezuela.com` (`app/layout.tsx:7`). The bare host 308-redirects to `www`; scrapers that don't follow the redirect may fail to render the share image.
- **Related test case:** TC-2026-06-28-08

### Bug #2 — Spanish UI copy missing diacritical accents (pervasive)
- **Severity:** Minor (localization / copy quality)
- **Environment:** Chromium, Desktop + Mobile, language = Español (default)
- **Steps:** 1. Load homepage in Spanish. 2. Read hero + complete a sample /gracias.
- **Expected:** Correct Spanish orthography.
- **Actual:** Hand-written UI copy is missing accents throughout — home hero ("…en todo el **pais**. …el **envio** de ayuda… quienes **mas** la necesitan"), /gracias ("Tu ayuda ya **esta** en camino", "Cada **donacion**", "**quedo** registrado", "la **atencion**", "**estan** atendiendo"), and the share button ("Compartir mi **donacion**"). Data-sourced strings (e.g. "Cáritas Venezuela") are correct — the issue is the copy strings.
- **Evidence:** ![](https://raw.githubusercontent.com/Simonethg/donavenezuela-qa-evidence/main/REG-2026-06-28/desktop/05b-gracias-params.png)
- **Related test case:** TC-2026-06-28-10

### Bug #3 — Inactive payment methods redirect off-site with no notice
- **Severity:** Minor (UX) — confirm intended with product
- **Environment:** Chromium, Desktop + Mobile, donation form
- **Steps:** 1. Open **Método**, pick **Tarjeta** (or PayPal / Transferencia). 2. Press **Donar**.
- **Expected:** An in-context flow, or a clear notice the donor is being sent to a partner site.
- **Actual:** Navigates off-site to `https://trazandofuturos.org/donavenezuela/` (`app/page.tsx:1090`) with no warning. The only `active` native method is Binance Pay.
- **Evidence:** ![](https://raw.githubusercontent.com/Simonethg/donavenezuela-qa-evidence/main/REG-2026-06-28/desktop/02-donation-form.png)
- **Related test case:** TC-2026-06-28-04

## Retracted during this run (adversarial re-test)

- **404 + 400 on load (previously suspected):** RETRACTED. On a clean homepage load with no payment action, console errors = 0 and failed responses = 0 on both breakpoints. The 404/400 and the GA `ga-audiences` CSP block seen earlier were triggered by the **Binance Pay redirect / Google Analytics**, not by donavenezuela.com. Not a site defect.

## Observations (not defects)

- **Default & only active native method is Binance Pay (crypto/USDT).** "Donar" with defaults sends a general donor to a live Binance Pay USDT QR checkout (merchant "Lcc Opentech, C.A"). Flag for product/CRO — confirm this is the intended primary path vs. a card-first flow. ![](https://raw.githubusercontent.com/Simonethg/donavenezuela-qa-evidence/main/REG-2026-06-28/desktop/03-donorbox-dialog.png)
- **`crypto-exchanges` method** is defined in code (`app/page.tsx:507`) but is **not rendered** in the production method dropdown (only Tarjeta / PayPal / Transferencia / Binance Pay appear).
- **`app/donorbox-dialog.tsx` is dead code** — not imported anywhere in the current build. Wire it in or remove.
- **i18n verified across es/en/pt/ja**: headings translate (Ayuda a Venezuela / Help Venezuela / Ajude a Venezuela / ベネズエラ支援), `<html lang>` updates, selection persists in `localStorage["donavenezuela-language"]`.

## Evidence index

| File | Shows |
|------|-------|
| `desktop/01-hero.png`, `mobile/01-hero.png` | Landing / hero |
| `desktop/02-donation-form.png` | Donation form (amounts, method, aid, destination) |
| `desktop/02b-methods.png` | Method dropdown open |
| `desktop/03-donorbox-dialog.png` | Binance Pay live checkout (reached via default method) |
| `desktop/04-i18n-{es,en,pt,ja}.png` | Each language rendered |
| `desktop/05-gracias.png` | /gracias (bare) |
| `desktop/05b-gracias-params.png` | /gracias with amount + institution summary |
| `desktop/desktop-walkthrough.mp4`, `mobile/mobile-walkthrough.mp4` | Full scroll-through |

## Forwardable summary

```
🐞 donavenezuela.com — Full regression — www.donavenezuela.com (prod) — FAIL
• OG metadata on non-canonical host (Minor, SEO)
• Spanish UI copy missing accents — home + /gracias + share (Minor, L10n)
• Inactive methods redirect off-site to trazandofuturos.org with no notice (Minor, UX)
Observations: default method = Binance Pay (crypto) — confirm intended · crypto-exchanges not rendered in prod · donorbox-dialog.tsx is dead code · i18n es/en/pt/ja OK
Retracted: earlier 404/400 were from the Binance/GA redirect, not the site.
```
