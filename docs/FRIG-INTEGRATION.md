# Integración Frig (cliente v1)

Este documento es el contrato hacia el SPA. El código de Frig se hace **en el repo frig**, después de que Y Print tenga `GET /v1/status` y `POST /v1/print` funcionando.

## Hoy en Frig (para no perder contexto)

| Superficie | Archivo | Qué hace |
|------------|---------|----------|
| Ancho 58/80 | `src/lib/pdf-format.ts` | `localStorage` por `branchId` |
| Descarga boleta/comanda | `src/lib/api/orders.ts` | `generate-boleta-pdf`, `generate-ticket-pdf` |
| Post-cobro | `src/components/pos/post-sale-modal.tsx` | Descarga PDF + `window.print()` del ticket simple |
| Terminal POS | `src/app/(app)/pos/terminal/page.tsx` | Descarga comanda; `PosConfigModal` sin impresora |
| Config estación | `src/lib/store/pos-config.ts` | Flags de módulos POS en `configuration_data` |
| Ventas | `src/app/(app)/sales/page.tsx` | Flyout PDF 80/58/A4 |
| Pagos | `src/app/(app)/payments/page.tsx` | Voucher thermal / 58 / A4 |

No hay UI de impresoras. Auditorías Frig lo marcan como P1 (`auditoria-go-to-market`, `validacion-e2e-equipos-iot`).

## Flujo objetivo al cobrar

```
Pago OK
  → PostSaleModal
  → GET Y Print /v1/status
      no hay agente → botones de descarga como hoy
      hay agente
        → pdf_format según status.paper_mm
        → downloadOrderThermalPdf (blob)
        → si posConfig.print_on_pay
              POST /v1/print { job_type: thermal.boleta, blob }
           else
              botón Imprimir (Frig) → mismo POST
```

El diálogo de Chrome desaparece para boleta/voucher/comanda. El ticket simple TXT puede quedar como fallback o salir también por Y Print cuando haya blob.

## Cambios de UI en Frig (cuando toque)

1. **Ajustes de estación** (`PosConfigModal`): switch “Imprimir al cobrar”, estado del agente (conectado / emparejar / no instalado) con `app_name` de `/v1/status`, link al instalador. Copy de caja: “agente de impresión”, no “Y Print” en cada pantalla.
2. **PostSaleModal**: botón Imprimir usa Y Print; se oculta o se degrada “Imprimir ticket simple” Chrome.
3. **Ventas / pagos / ingresos**: acción Imprimir al lado de Descargar, mismo `POST /v1/print`.

El ancho 58/80 se elige en Y Print (el PC tiene la impresora). Frig deja de ser la única fuente de `frig.thermalPdfFormat`, o lo sincroniza desde `/v1/status`.

## Pairing

Desde el POS, “Emparejar este navegador”:

1. Y Print muestra código.
2. Frig pide el código.
3. Guarda `station_token` por origin (localStorage o IndexedDB, no es el Token de Yggdra).

Un PC de caja, un Y Print, varios cajeros en el mismo Chrome-perfil pueden compartir el pairing.

## Issues en frig

No se abren hasta el corte “localhost v1 listo” (issue de contrato en este repo). Entonces:

- feat: cliente Y Print (`src/lib/y-print.ts`)
- feat: `print_on_pay` en PosConfig
- feat: post-sale e impresión de ventas/pagos por Y Print
- docs: help “Instalar Y Print”
