# Job types

Un job type es el único gancho para enriquecer Y Print sin tocar el motor.

## v1

| job_type | Documento Yggdra | `pdf_format` | Módulo |
|----------|------------------|--------------|--------|
| `thermal.boleta` | `GET /sales/orders/{id}/generate-boleta-pdf/` o boleta de revenue | `thermal` \| `thermal_58` | `sales` / `finance` |
| `thermal.ticket` | `GET /sales/orders/{id}/generate-ticket-pdf/` | `thermal` \| `thermal_58` | `sales` |
| `thermal.voucher` | `GET /finance/payments/{id}/download-voucher/` o `generate-pdf/`; revenues `download-voucher` | `thermal` \| `thermal_58` | `finance` |

El motor trata los tres igual: PDF de 58 u 80 mm → impresora. El tipo sirve para permisos, logs y la UI (“Reimprimir última boleta”).

## Mapa resource → URL (modo resource)

| resource.app | resource.name | resource.action | URL |
|--------------|---------------|-----------------|-----|
| sales | order | boleta | `/sales/orders/{id}/generate-boleta-pdf/?pdf_format=` |
| sales | order | ticket | `/sales/orders/{id}/generate-ticket-pdf/?pdf_format=` |
| finance | payment | voucher | `/finance/payments/{id}/download-voucher/?pdf_format=` |
| finance | payment | pdf | `/finance/payments/{id}/generate-pdf/?pdf_format=` |
| finance | revenue | boleta | `/finance/revenues/{id}/generate-boleta-pdf/?pdf_format=` |
| finance | revenue | voucher | `/finance/revenues/{id}/download-voucher/?pdf_format=` |

Agregar una fila aquí es el trabajo de “otro producto necesita imprimir X”, siempre que Yggdra ya tenga el PDF.

## Extensión (no v1, el contrato ya los reserva)

| job_type | Cuándo | Nota |
|----------|--------|------|
| `thermal.test` | Instalación | Página de calibración 58/80, sin API |
| `a4.document` | Reportes, boleta domiciliaria | Otro papel; otro driver |
| `label.nutrition` | Frig nutrición | Hoy es `window.print()` en el SPA |
| `label.product` | Inventario | Si Yggdra genera el PDF |
| `kitchen.route` | Varias impresoras | Mismo `thermal.ticket` + `printer_id` |

No se implementan hasta que un producto de la familia los pida con un PDF (o un renderer) real.

## Regla de oro

Y Print no decide el contenido. Si el PDF está mal (aire, fuente, timbre), el arreglo es **yggdra_arq**. Si el papel tiene un marco blanco extra, el arreglo es **Y Print** (escala / márgenes del spooler). Esa es la prueba del spike (issue de laboratorio).
