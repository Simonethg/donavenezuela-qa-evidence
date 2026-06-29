# Release a Producción — donavenezuela.com — 2026-06-29

**Qué se hizo:** se promovió `staging` → `main` (producción, `www.donavenezuela.com`).
**Build prod:** `5aa6282` (PR #109). **Estado:** ✅ desplegado y verificado en vivo.
**QA:** regresión completa en prod → **PASS** (ver más abajo).

Este release junta ~100 commits de varias personas. Abajo, **todo lo que entró**,
agrupado por área, con **quién hizo el trabajo** y **quién hizo el merge** de cada ticket.

> Handles: **raor00** = Rafael Oviedo (RO) · **mariajmr1202** = María José Muñoz ·
> **antoninu** = Antonio Aspite · **hercdevelopment-ve** = cuenta de CI (Hercules) ·
> **Simonethg** = Simoneth · **meyerowitz**, **Jefe Mac**, **Amilcar Erazo**.

---

## 🆕 Funcionalidades nuevas

| Ticket / Qué | Trabajo (autor) | Merge por |
|--------------|-----------------|-----------|
| Rieles **Loopay / Bre-B** — Colombia y Ecuador (recaudo COP→USDT→VES) | **Antonio Aspite** | vía `develop` (Antonio Aspite) |
| **Meru** — checkout cripto (Crypto/QR Bolivia) | **Antonio Aspite** | vía `develop` (Antonio Aspite) |
| **Donorbox** — Tarjeta / PayPal / Transferencia (modal embebido) | **Jefe Mac** | vía `develop` (Antonio Aspite) |
| Remake de la sección de donaciones (UI) | **Antonio Aspite** / **Jefe Mac** | vía `develop` (Antonio Aspite) |
| #45 — Desglose + tarjetas C2S/Regnum + trazabilidad por organización | **raor00** | **raor00** |
| #79 — Mejoras de UI / fixes visuales | **meyerowitz** | hercdevelopment-ve (CI) |

## 🏢 Organizaciones

| Ticket / Qué | Trabajo (autor) | Merge por |
|--------------|-----------------|-----------|
| #47 — Agregar **Fe y Alegría** | **mariajmr1202** | **mariajmr1202** |
| #48 — **Fe y Alegría** + **Tinta Violeta** | **raor00** | **antoninu** (Antonio Aspite) |
| Quitar **Hogar Bambi** del flujo (#51; reconciliado al promover) | **raor00** / **Maria J** | reconciliación: **Simonethg** |

## 💳 Fixes de flujo de pago (eran bloqueantes)

| Ticket / Qué | Trabajo (autor) | Merge por |
|--------------|-----------------|-----------|
| #81 (PR #89) — **Meru**: spinner + timeout 15s + error/reintento | **Simonethg** | **Simonethg** |
| #80 + #82 (PR #91) — feedback de carga Donorbox/donor-dialog + race del backdrop | **Simonethg** | **Simonethg** |
| #97, #105 — **Donorbox**: arreglo del overlay que tapaba el formulario | **Simonethg** | **Simonethg** |

## 🔒 Seguridad

| Ticket / Qué | Trabajo (autor) | Merge por |
|--------------|-----------------|-----------|
| #31 — Hardening: allowlist API, cabeceras + **CSP (report-only)**, validación de redirects | **Simonethg** | **antoninu** (Antonio Aspite) |
| #68 — Bloquear **modo test en producción** | **raor00** | hercdevelopment-ve (CI) |
| #70 — Rechazar `http://` en `?api=` (solo HTTPS) | **raor00** | hercdevelopment-ve (CI) |
| #94 — **Error boundaries** de Next.js | **raor00** | hercdevelopment-ve (CI) |

## 🐛 Otros fixes / UX

| Ticket / Qué | Trabajo (autor) | Merge por |
|--------------|-----------------|-----------|
| #71 — Manejo de error de **clipboard** al compartir | **raor00** | hercdevelopment-ve (CI) |
| #72 — Límite de iteraciones en el **muro de donaciones** (evita loop) | **raor00** | hercdevelopment-ve (CI) |
| #84 — **Accesibilidad**: Escape + focus trap en los 4 modales | **raor00** | hercdevelopment-ve (CI) |
| #85 — **Carrusel**: imágenes locales (evita tokens que expiraban) | **raor00** | hercdevelopment-ve (CI) |

## 🧪 Herramientas de QA (no afectan al usuario final)

| Ticket / Qué | Trabajo (autor) | Merge por |
|--------------|-----------------|-----------|
| #99, #101, #102 — **Bypass de pago para QA** (triple candado, inactivo en prod) + simulación por método + docs | **Simonethg** | **Simonethg** |
| #108 — Restaurar headers de seguridad tras la investigación del reload | **Simonethg** | **Simonethg** |
| #109 — **Release: promover staging → prod** | **Simonethg** | **Simonethg** |

---

## ✅ QA — Regresión completa en PRODUCCIÓN (`www.donavenezuela.com`)

Desktop 1440×900 + Mobile 393×852 (mobile emulado, provisional hasta prueba en
dispositivo real). **Seguridad de pagos:** no se completan cargos reales.

| Caso | Resultado |
|------|-----------|
| Form de donación (montos, método default Binance) | ✅ PASS |
| Métodos presentes (Tarjeta, PayPal, Transferencia, Binance, Cripto/Meru, Ecuador·Bre-B) | ✅ PASS |
| i18n es/en/pt/ja (sin claves crudas, sin `undefined`, persiste) | ✅ PASS |
| /gracias — 200, compartir, muestra monto e institución | ✅ PASS |
| Responsive desktop + mobile | ✅ PASS |
| OpenGraph / SEO (imagen OG resuelve en dominio prod) | ✅ PASS |
| Consola / red — **0 errores**, 0 requests fallidos | ✅ PASS |
| En vivo: **Fe y Alegría presente**, **Hogar Bambi removido** | ✅ PASS |
| **Bypass QA inactivo en prod** (con `?qa=1` no aparece nada) | ✅ PASS |

**Veredicto: ✅ PASS** — sin defectos Critical/Major/Minor en prod (frontend).

---

## ⚠️ Pendiente con el equipo de Ibis (no bloqueó el deploy)

1. **Validación real end-to-end de pagos** con backend/proveedores en vivo. — *Ibis*
2. **Binance success URL** vuelve a una URL vieja (config del backend usaibis). — *Ibis*
3. **#67** trazabilidad del estado de pago Koywe/PagoMóvil (backend, abierto). — *Ibis*
4. **CI en rojo** por test desactualizado (`mocks.test.ts` espera `hogar-bambi`); sin impacto en runtime — actualizar el test.
5. Antes de pasar el **CSP a *enforcing*** (hoy report-only): agregar dominios de Stripe/PayPal/jsDelivr/etc. al allowlist.

---

## Evidencia (prod)

**Desktop**
![hero](https://raw.githubusercontent.com/Simonethg/donavenezuela-qa-evidence/main/PROD-REGRESSION-2026-06-29/desktop/01-hero.png)
![form](https://raw.githubusercontent.com/Simonethg/donavenezuela-qa-evidence/main/PROD-REGRESSION-2026-06-29/desktop/02-donation-form.png)
![gracias](https://raw.githubusercontent.com/Simonethg/donavenezuela-qa-evidence/main/PROD-REGRESSION-2026-06-29/desktop/05-gracias.png)
<video src="https://cdn.jsdelivr.net/gh/Simonethg/donavenezuela-qa-evidence@main/PROD-REGRESSION-2026-06-29/desktop/scrollthrough-desktop.mp4" controls></video>

**Mobile** (`mobile-emulated`)
![form](https://raw.githubusercontent.com/Simonethg/donavenezuela-qa-evidence/main/PROD-REGRESSION-2026-06-29/mobile/02-donation-form.png)
<video src="https://cdn.jsdelivr.net/gh/Simonethg/donavenezuela-qa-evidence@main/PROD-REGRESSION-2026-06-29/mobile/scrollthrough-mobile.mp4" controls></video>
