# Requisitos — extraídos de Frig y Yggdra

Fecha de lectura: 2026-11-19.  
API: [`yggdra_arq`](https://github.com/felipebarraza6/yggdra_arq) (`PLAN_58MM.md`, `thermal_format.py`).  
Front: [`frig`](https://github.com/felipebarraza6/frig) (`src/lib/pdf-format.ts`, POS, ventas, pagos).

## Problema actual (Frig)

Tres caminos de “imprimir”, ninguno nativo:

1. **Descarga PDF** — `apiFile` a Yggdra, el usuario abre el archivo y usa el diálogo del lector / Chrome.
2. **Ticket simple POS** — `PostSaleModal.handlePrintTicket` abre una ventana y `window.print()` de un `<pre>`. Sin `@page { margin: 0 }`.
3. **Totem / nutrición** — `window.print()` de la página. Nutrición sí pone `@page { margin: 0 }`; el POS no.

Al cobrar, Frig abre `PostSaleModal` (`src/components/pos/post-sale-modal.tsx`):

- Boleta PDF 80 o 58 mm (recuerda el último formato por sucursal en `localStorage`)
- Ticket simple TXT (descarga)
- “Imprimir ticket simple” → Chrome

`PosConfigModal` configura modo de operación (ventas, mesas, delivery…). **No** hay impresora ni “imprimir al cobrar”.

Yggdra **no** tiene endpoint ESC/POS. Documentado en Frig `docs/api-map.md` §11 como trabajo futuro (`services/printer`). Ese trabajo se hace aquí, como producto de familia, no como microservicio en el server.

## Documentos térmicos que ya existen en la API

Query: `pdf_format` (alias legacy `format`). Valores térmicos: `thermal` (80 mm, default), `thermal_58` (58 mm). `a4` existe y **queda fuera de Y Print v1**.

| Método | Ruta | Quién lo usa en Frig |
|--------|------|----------------------|
| GET | `/api/sales/orders/{id}/generate-boleta-pdf/?pdf_format=` | POS post-venta, Ventas |
| GET | `/api/sales/orders/{id}/generate-ticket-pdf/?pdf_format=` | POS comanda, Ventas |
| GET | `/api/sales/orders/{id}/generate-boleta-domiciliaria-pdf/` | A4 domicilio — **fuera de v1** |
| GET | `/api/finance/revenues/{id}/generate-boleta-pdf/?pdf_format=` | Ingresos |
| GET | `/api/finance/revenues/{id}/download-voucher/?pdf_format=` | Ingresos |
| GET | `/api/finance/payments/{id}/download-voucher/?pdf_format=` | Pagos / caja |
| GET | `/api/finance/payments/{id}/generate-pdf/?pdf_format=` | Pagos |

Helper API: `api/apps/shared/pdf/thermal_format.py`

| Formato | Ancho página | Margen interno PDF | Área útil |
|---------|--------------|--------------------|-----------|
| `thermal` | 80 mm | 5 mm | ~70 mm |
| `thermal_58` | 58 mm | 3 mm | ~52 mm |

Ese margen interno es del generador (cabezal térmico). Y Print **no** le suma otro. Si en prueba de laboratorio (lector PDF, 100 %, márgenes 0) el ticket sigue “aireado”, se abre issue en `yggdra_arq` para bajar 5/3 mm. No se “arregla” escalando en el agente.

## Auth y contexto (obligatorio, igual que Frig)

De Frig `docs/api-map.md` y `src/lib/api/auth.ts`:

```
POST /api/accounts/users/login_complete/
Authorization: Token <key>
X-Branch-ID: <id>
GET  /api/shared/frontend-config/?branch_id=
```

`login_complete` devuelve `token`, `user`, `branches`, `owned_organizations`, `permissions.enabled_apps` / `read_only_apps` / `disabled_apps`.

`frontend-config` devuelve menú ya filtrado, `modules` (enabled + submodule_config), `permissions`, tema. Es el mismo bootstrap que Frig.

Y Print usa **exactamente** esos dos llamados. No inventa un rol propio.

Capas de permiso en API (`docs/architecture/PERMISSIONS_LAYERS.md`):

> plan (módulos activos) × rol (`BranchRoleDefinition`) → apps accesibles → `ModuleRolePermission` en el ViewSet.

Si el cajero no tiene `sales` o `finance` en write/read, Y Print no ofrece reimprimir boletas. Si el módulo `pos` está off, no ofrece el job de comanda de estación. El PDF endpoint ya 403; la UI se adelanta con `frontend-config`.

## Requisitos funcionales v1

**RF1.** Instalador Windows. Corre en bandeja. Arranca con el usuario de Windows.

**RF2.** Login email/password contra `login_complete`. Token en Credential Manager. Logout borra token (mismo espíritu que `/accounts/users/logout/`).

**RF3.** Selector de sucursal entre las de `login_complete.branches`. Todas las llamadas a Yggdra llevan `X-Branch-ID`.

**RF4.** Tras login, `GET /shared/frontend-config/?branch_id=`. Las pestañas / acciones de Y Print se filtran con `modules` + `permissions`.

**RF5.** Descubrir impresoras del PC. El usuario elige una por defecto y el ancho 58 o 80 mm. Ese par es de **este PC**.

**RF6.** Motor: recibir un PDF térmico y mandarlo a la impresora a escala 100 %, sin diálogo, sin encabezado, sin “ajustar a la página”.

**RF7.** HTTP loopback `docs/LOCAL-API.md` para que Frig (u otra app) envíe jobs. Pairing por origin.

**RF8.** Job types v1 (ver [PRINT-JOBS.md](PRINT-JOBS.md)): `thermal.boleta`, `thermal.ticket`, `thermal.voucher`. El PDF lo trae el caller o Y Print lo baja a Yggdra con el token de la sesión si el caller manda `{ resource, id, pdf_format }`.

**RF9.** Página de prueba: “imprimir test” (página en blanco con reglas de 58/80 mm) para validar el rollo sin una venta.

**RF10.** Si localhost no responde, Frig conserva descarga PDF. Y Print no es un SPOF silencioso.

## Requisitos no funcionales

**RNF1.** Un job de boleta corta no debe sentirse “lento” en caja (objetivo: < 2 s desde `POST /v1/print` hasta que la impresora tira papel, red local + PDF ya en mano).

**RNF2.** Sin telemetría de contenido de boletas a un cloud de Y Print. El PDF puede ir browser → localhost o Y Print → Yggdra; no a un tercer host.

**RNF3.** CORS / origins: lista de emparejados, no `*`.

**RNF4.** Logs locales rotados: job id, impresora, mm, resultado. Sin cuerpo del PDF.

## Requisitos de Frig (cliente v1)

Viven en el repo Frig; aquí se listan para no olvidar el corte:

| ID | Cambio en Frig |
|----|----------------|
| F1 | Detectar Y Print (`GET /v1/status`). UI “conectado / no instalado”. |
| F2 | Pairing desde POS o ajustes de estación. |
| F3 | Post-cobro: si auto-print, `POST /v1/print` con boleta; si no, botón Imprimir de Frig (no Chrome). |
| F4 | Toggle `print_on_pay` en `PosConfig` (hoy no existe). |
| F5 | Pedir `pdf_format` al status de Y Print, no solo a `localStorage`. |
| F6 | Ventas, pagos, ingresos: mismo `POST /v1/print` para sus PDFs térmicos. |
| F7 | Fallback descarga si Y Print no está. |

Esas issues de Frig se abren **en frig** cuando Y Print tenga el contrato localhost estable. Este repo no implementa el SPA.

## No requisitos (v1)

- Reimplementar generadores ReportLab.
- Endpoint ESC/POS en Yggdra.
- Cola en el server (Y Print es el que ve la impresora).
- Multi-impresora “caja vs cocina” (se anota en roadmap; el contrato admite `printer_id` para no pintarse en una esquina).
- Nutrición / totem `window.print()`.
- A4 domiciliaria.
