# Release a Producción — donavenezuela.com — 2026-06-29

**Qué se hizo:** se promovió `staging` → `main` (producción, `www.donavenezuela.com`).
**Build prod:** `5aa6282` (PR #109). **Estado:** ✅ desplegado y verificado en vivo.
**QA:** regresión completa en prod → **PASS** (ver más abajo).

Este release junta ~100 commits de varias personas. Abajo, **todo lo que entró**,
agrupado por área, con el **responsable (usuario de GitHub)** de cada cosa.

---

## 🆕 Funcionalidades nuevas

| Qué | Responsable |
|-----|-------------|
| Rieles de donación **Loopay / Bre-B** — Colombia y Ecuador (recaudo COP → USDT → VES) | **Antonio Aspite** |
| **Meru** — checkout cripto ("Crypto Exchanges & Wallets" / QR Bolivia) | **Antonio Aspite** |
| **Donorbox** — pago con **Tarjeta / PayPal / Transferencia** (modal embebido) | **Jefe Mac** |
| Remake de la sección de donaciones (UI) | **Antonio Aspite** / **Jefe Mac** |
| Desglose de donaciones + tarjetas **C2S/Regnum** + trazabilidad por organización (#45) | **raor00** (Rafael Oviedo) |
| Mejoras de UI / fixes visuales (#79) | **meyerowitz** |

## 🏢 Organizaciones

| Qué | Responsable |
|-----|-------------|
| Agregar **Fe y Alegría** + **Tinta Violeta** (#47, #48) | **mariajmr1202** (María José Muñoz) + **raor00** |
| **Quitar Hogar Bambi** del flujo (#51; reconciliado en el merge a prod) | **raor00** / **Maria J** (reconciliación final: **Simonethg**) |

## 💳 Fixes de flujo de pago (eran bloqueantes para lanzar)

| Qué | Responsable |
|-----|-------------|
| **Meru**: spinner de carga + timeout 15s + manejo de error con reintento (#81 → PR #89) | **Simonethg** |
| **Donorbox/donor-dialog**: feedback de carga (#80) + arreglo de la *race* que cerraba el modal mientras se procesaba el pago (#82) (PR #91) | **Simonethg** |
| **Donorbox**: arreglo del overlay que se quedaba pegado tapando el formulario (#97, #105) | **Simonethg** |

## 🔒 Seguridad

| Qué | Responsable |
|-----|-------------|
| Hardening de datos: allowlist de API, cabeceras de seguridad + **CSP (report-only)**, validación de redirects (#31) | **Simonethg** |
| Bloquear el **modo test en producción** (#68) | **raor00** |
| Rechazar URLs `http://` en el parámetro `?api=` (solo HTTPS) (#70) | **raor00** |
| **Error boundaries** de Next.js (un componente que falla ya no tumba toda la página) (#94) | **raor00** |

## 🐛 Otros fixes / UX

| Qué | Responsable |
|-----|-------------|
| Manejo de error de **clipboard** al compartir (#71) | **raor00** |
| Límite de iteraciones en el **muro de donaciones** (evita loop infinito) (#72) | **raor00** |
| **Accesibilidad**: tecla Escape + focus trap en los 4 modales (#84) | **raor00** |
| **Carrusel**: imágenes servidas localmente (evita tokens que expiraban) (#85) | **raor00** |

## 🧪 Herramientas de QA (no afectan al usuario final)

| Qué | Responsable |
|-----|-------------|
| **Bypass de pago para QA** en staging (triple candado, **inactivo en prod**) + simulación por método + documentación (#99, #101, #102) | **Simonethg** |
| Restaurar headers de seguridad tras la investigación del reload de Donorbox (#108) | **Simonethg** |

---

## ✅ QA — Regresión completa en PRODUCCIÓN (`www.donavenezuela.com`)

Desktop 1440×900 + Mobile 393×852 (mobile emulado, provisional hasta prueba en
dispositivo real). **Seguridad de pagos:** no se completan cargos reales; los pagos
se validan por render del formulario + ruteo correcto, no pagando.

| Caso | Resultado |
|------|-----------|
| Form de donación (montos $10/$25/$50/$100/Otro, método default Binance) | ✅ PASS |
| Métodos presentes (Tarjeta, PayPal, Transferencia, Binance, Cripto/Meru, Ecuador·Bre-B) | ✅ PASS |
| i18n es/en/pt/ja (sin claves crudas, sin `undefined`, persiste al recargar) | ✅ PASS |
| /gracias — 200, botón compartir, muestra monto e institución | ✅ PASS |
| Responsive desktop + mobile (sin overflow ni recortes) | ✅ PASS |
| OpenGraph / SEO (título, descripción, imagen OG resuelve en dominio prod) | ✅ PASS |
| Consola / red — **0 errores**, 0 requests fallidos (ambos breakpoints) | ✅ PASS |
| Verificado en vivo: **Fe y Alegría presente**, **Hogar Bambi removido** | ✅ PASS |
| **Bypass QA inactivo en prod** (con `?qa=1` no aparece nada) | ✅ PASS |

**Veredicto: ✅ PASS** — sin defectos Critical/Major/Minor en el frontend de producción.

---

## ⚠️ Pendiente con el equipo de Ibis (no bloqueó el deploy)

1. **Validación real end-to-end de pagos** con backend/proveedores en vivo (Binance, Koywe/PagoMóvil vía usaibis, Meru, Loopay, Donorbox+Stripe). El QA valida UI/flujo, no el settlement real.
2. **Binance success URL** — al pagar por Binance vuelve a una URL vieja. Es config del backend (usaibis), no del frontend.
3. **#67** trazabilidad del estado de pago Koywe/PagoMóvil (backend, abierto).
4. **CI en rojo** por un test desactualizado (`mocks.test.ts` espera `hogar-bambi`); sin impacto en runtime — actualizar el test.
5. Antes de pasar el **CSP a *enforcing*** (hoy report-only): agregar los dominios de Stripe/PayPal/jsDelivr/etc. al allowlist o se rompen los pagos con tarjeta.

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
