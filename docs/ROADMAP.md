# Roadmap

Orden de trabajo. Las issues de este repo siguen estos cortes. No se mezcla Frig hasta el corte 4.

## Corte 0 — Laboratorio (sin producto)

Imprimir un PDF de Yggdra (boleta 80 y 58) desde un lector, escala 100 %, márgenes 0. Anotar si el aire que queda es el margen interno del PDF (5/3 mm) o el spooler.

Si el PDF ya se ve bien: el agente solo tiene que repetir esa impresión.  
Si el PDF se ve aireado: issue en `yggdra_arq` para bajar márgenes, **además** de Y Print.

## Corte 1 — App nativa vacía

Shell Tauri Windows, bandeja, ventana login/sucursal placeholder, health HTTP en `17380`.

## Corte 2 — Identidad Yggdra

`login_complete`, token en Credential Manager, `X-Branch-ID`, `frontend-config`, UI de capacidades según módulos. Base URL configurable.

## Corte 3 — Motor + impresoras

Listar impresoras Windows. Elegir default + 58/80. Imprimir PDF 1:1. Botón “página de prueba”.

## Corte 4 — Contrato localhost

`/v1/status`, `/v1/print` blob, pairing. Documentado en `LOCAL-API.md`. Aquí Frig puede empezar su cliente.

## Corte 5 — Modo resource

Y Print baja el PDF a Yggdra (reimprimir desde la bandeja). Mapa `PRINT-JOBS.md`.

## Corte 6 — Instalador

MSI / NSIS, auto-start, icono de familia. Code signing queda issue aparte (SmartScreen).

## Después (cuando un producto lo pida)

- Segunda impresora (`printer_id` por job, cocina vs caja)
- A4
- Etiquetas
- SDK npm `@yggdra/print-client`
- macOS / Linux

## Qué no se hace en paralelo

No se reescribe ReportLab en este repo.  
No se agrega ESC/POS a Yggdra “por si acaso”.  
No se integra Frig antes de tener `/v1/print` estable.
