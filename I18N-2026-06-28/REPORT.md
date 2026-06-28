# QA — donavenezuela.com — i18n audit + consolidated triage — 2026-06-28

**Target / build:** https://www.donavenezuela.com (production — same build as REG-2026-06-28; the latest pushed changes are NOT live yet, `x-vercel-cache: HIT`, og hash unchanged)
**Scope:** Full bilingual revision (es/en, plus pt/ja). Goal: find sections where the selected language is not respected (untranslated text leaks). The selected language must be honored **everywhere, including error messages.**
**QA:** Simoneth · **Method:** Playwright per-language render + cross-language text diff (`scripts/i18n_audit.mjs`) + source review of `lib/i18n.ts` and `app/deferred-sections.tsx`
**Evidence:** [`I18N-2026-06-28/`](https://github.com/Simonethg/donavenezuela-qa-evidence/tree/main/I18N-2026-06-28) — `es/en/pt/ja-fullpage.png`, `leaks.json`

---

## How to read the leak counts

The raw audit flagged 73 (en) / 90 (pt) / 74 (ja) identical blocks across languages. **Most are false positives** that should NOT be translated: donor names (Claudia, Kelvin, Luis…), countries + flags (México 🇲🇽, Ecuador 🇪🇨, Puerto Rico 🇵🇷), partner/brand names (Wallib, Muney, Meru, Ibis, Pardux, Coco Mercado, Vank, Trazando Futuros), institution names (Hogar Bambi, Cáritas Venezuela), and app/keyword link titles (venezuela reporta, tilin app, centinela).

After removing proper nouns, the **real leaks** are below.

## Sections where the selected language is NOT respected (real leaks)

| # | Section | Leaked content (shown in Spanish regardless of language) | Languages affected | Source |
|---|---------|----------------------------------------------------------|--------------------|--------|
| L1 | **Iniciativas / Initiatives** (cards) | Card descriptions & titles: "Plataforma para centralizar información verificada…", "Reporte de personas desaparecidas", "Reporte de daños estructurales", "Apoyo presencial y rescate" | en, pt, ja | **Hardcoded** in `app/deferred-sections.tsx:114-196` (bypasses the dictionary) |
| L2 | **Donation form labels** | "Metodo", "Destino", "Idioma", "Filtros" still in Spanish | **pt** (es/en OK) | Dictionary `pt` block missing/equal to es |
| L3 | **Donation feed purpose** | "para Apoyo para cualquier necesidad (Rescate, Medicina…)", "para Compañeros de C2S en Venezuela" | en, pt, ja | **Backend data** (user-entered) — not a frontend i18n bug; see note |

> **Error messages:** the validation strings (`donorDialog.errors.*`, `interestDialog.*`) DO exist per-language in `lib/i18n.ts` (es 276-325, en 627-676). They could not be triggered on prod without initiating a real payment (the only active method is Binance Pay), so **error-message language was verified in source, not live** — recommend a dev/preview pass that forces the donor/interest dialog to confirm each error renders in the selected language at runtime.

L3 is **not a code bug**: those strings are donor-entered purposes coming from the live donations API; user-generated content can't be auto-translated. Noted for awareness only.

---

## Consolidated error totals (this run + carried over from REG-2026-06-28)

All 3 defects from the previous run were re-verified on prod and **still reproduce** (prod build unchanged).

| ID | Defect | Severity | Status | Carried from |
|----|--------|----------|--------|--------------|
| B1 | OpenGraph metadata uses non-canonical host (`donavenezuela.com` vs `www`) | Minor (SEO) | 🔴 Open | REG-2026-06-28 |
| B2 | Spanish UI copy missing diacritical accents (home + /gracias + share) | Minor (L10n) | 🔴 Open | REG-2026-06-28 |
| B3 | Inactive payment methods redirect off-site to `trazandofuturos.org` with no notice | Minor (UX) | 🔴 Open | REG-2026-06-28 |
| L1 | Initiatives section hardcoded in Spanish → leaks in en/pt/ja | Major (i18n) | 🔴 New | this run |
| L2 | Portuguese form labels (Metodo/Destino/Idioma/Filtros) untranslated | Minor (i18n) | 🔴 New | this run |

**Total open: 5 defects** (1 Major, 4 Minor) + 1 awareness note (L3) + 1 unverified-at-runtime area (error messages).
**Retracted (REG-2026-06-28):** the 404/400 console errors — confirmed they came from the Binance/GA redirect, not the site.

---

## Fix triage — safe to fix now vs. needs an experienced dev

### ✅ Safe to fix now (text-only, no logic change — low risk)

- **B2 — Missing accents.** Edit string literals in `lib/i18n.ts` (e.g. `informacion`→`información`, `Busqueda`→`Búsqueda`, `logistica`→`logística`, `esta`→`está`, `donacion`→`donación`, `atencion`→`atención`, `anos`→`años`) and the matching hardcoded copy. Pure text, no logic. **Caveat:** do NOT add accents to URLs, search-keyword link titles, or `href`s.
- **L2 — Portuguese labels.** Fill the missing/Spanish-equal values in the `pt` dictionary block (`Metodo`→`Método`, etc.). Text-only dictionary edit.

### ⚠️ Safe but verify (config — do carefully)

- **B1 — OG canonical host.** One-line: set `metadataBase` to `https://www.donavenezuela.com` (`app/layout.tsx:7`) or set `NEXT_PUBLIC_RELIEF_BASE_URL`. Low risk, but it changes canonical/OG URLs globally — confirm `www` is the intended canonical (it is: bare host 308→www) and re-check OG after deploy.

### 🔴 Needs a more experienced dev (logic / architecture / product)

- **L1 — Initiatives i18n.** The initiatives data is a **hardcoded Spanish array** in `app/deferred-sections.tsx:114-196`. Fixing it means refactoring that data to be language-aware (move the card titles/descriptions into the per-language dictionary and reference `t.…`, across es/en/pt/ja). Structural change touching a data model + 4 locales → experienced dev.
- **B3 — Inactive-method redirect.** UX + logic + product call (what should Tarjeta/PayPal/Transferencia do, and should Binance Pay be the default for a general-public portal). Needs dev + product alignment.
- **Error-message runtime verification.** Forcing the donor/interest dialogs to confirm errors render in the selected language requires a dev/preview environment (can't be done on prod without a real payment) → dev.

---

## Forwardable summary

```
🐞 donavenezuela.com — i18n audit + carry-over — www (prod, unchanged build)
5 open defects (1 Major, 4 Minor):
• L1 Initiatives section hardcoded Spanish → leaks in EN/PT/JA (Major) — needs dev
• B1 OG metadata on non-canonical host (Minor) — safe-with-care
• B2 Spanish copy missing accents (Minor) — SAFE to fix (text only)
• B3 Inactive methods redirect off-site w/o notice (Minor) — needs dev+product
• L2 Portuguese form labels untranslated (Minor) — SAFE to fix (dictionary)
Notes: donation-feed purposes are backend/user data (not a bug); error-message
language verified in source only — needs a preview pass to confirm at runtime.
```
