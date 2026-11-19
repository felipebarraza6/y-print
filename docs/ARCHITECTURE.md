# Arquitectura

## Piezas

```
┌─────────────────────────────────────────────────────────┐
│  Producto web (Frig u otro)                             │
│  Navegador. Sesión Yggdra propia.                       │
│  Al imprimir: POST http://127.0.0.1:<port>/v1/print     │
└───────────────────────────┬─────────────────────────────┘
                            │ localhost + origin emparejado
                            ▼
┌─────────────────────────────────────────────────────────┐
│  Y Print (Windows, bandeja)                             │
│  • Login Yggdra (Token en Credential Manager)           │
│  • Sucursal activa (X-Branch-ID)                        │
│  • Capacidades = plan × rol × módulos                   │
│  • Descubrimiento de impresoras                         │
│  • Motor: PDF → papel a mm exactos, margen 0 extra      │
└──────────────┬──────────────────────────┬───────────────┘
               │ HTTPS                    │ USB / TCP 9100 / cola Windows
               ▼                          ▼
     Yggdra API (yggdra_arq)        Impresora térmica
     PDFs, login, frontend-config
```

Yggdra **sigue** generando el PDF. Y Print no clona layouts de boleta ni el timbre SII. Rasteriza o spawnea el PDF al ancho físico del rollo.

## Por qué PDF y no ESC/POS en v1

Las boletas chilenas de Frig llevan logo de sucursal y, si hay DTE, timbre gráfico (PDF417 / XML de `TaxDocument`). Reescribir eso en ESC/POS duplica Yggdra y se rompe en cuanto cambie el generador.

Yggdra ya tiene:

- página de **80 mm** (`thermal`) con margen interno 5 mm
- página de **58 mm** (`thermal_58`) con margen interno 3 mm (~52 mm útiles)

Fuente: `api/apps/shared/pdf/thermal_format.py` en `yggdra_arq` (`PLAN_58MM.md`).

El margen que “siempre sale mal” en Frig es el **segundo** margen: Chrome / diálogo de Windows. El motor de Y Print imprime a escala 100 %, papel = ancho del PDF, sin encabezados.

## Stack v1 (cerrado)

Tauri **es** Rust + UI web. No se elige uno u otro.

| Capa | Elección | Por qué |
|------|----------|---------|
| Shell | Tauri 2, Windows | Instalador chico, bandeja, WebView2 ya está en Win10/11 |
| UI | React | Login, sucursal, pairing, tema de familia (mismo lenguaje que Frig) |
| Motor | Rust | pdfium raster 203 DPI al mm exacto → bytes RAW (`WritePrinter` o TCP 9100) |
| Auth persistida | Windows Credential Manager | El Token de DRF no va a un `.env` en disco |
| HTTP local | Axum/hyper en el proceso Tauri, `127.0.0.1:17380` | Vive con la bandeja, ventana cerrada |

El CPU de una boleta no importa. Lo que importa es que el driver **no** “ajuste a la página”. Raster → RAW evita el segundo margen (Chrome / spooler / `scale: fit`).

**No usar** como motor:

- `tauri-plugin-printer` (Sumatra + fit = el mismo margen)
- `tauri-plugin-thermal-printer` (ESC/POS texto: el timbre SII no cabe)

Esos plugins pueden listar impresoras; el job de boleta lo escribe este repo.

Rust puro (egui) se descarta: el motor saldría igual, la UI de producto no. Electron se descarta por peso de agente de bandeja.

Linux / macOS quedan para cuando haya un local que los pida. El contrato localhost no cambia.

## Confianza entre SPA y Y Print

Cualquier pestaña en ese PC podría pegarle a localhost. Por eso:

1. Y Print escucha solo en `127.0.0.1`.
2. La SPA se **empareja** una vez (código de 6 dígitos o confirmación en la bandeja).
3. Cada `POST /v1/print` lleva el origin permitido (`https://app…` / `http://localhost:3000`) y un token de pairing de estación.
4. Sin pairing, Y Print acepta solo su propia UI.

Detalle: [LOCAL-API.md](LOCAL-API.md).

## Configuración: qué vive dónde

| Dato | Dónde |
|------|--------|
| Ancho de rollo 58 / 80 mm | Y Print, por PC (el hardware está aquí) |
| Impresora por defecto de este PC | Y Print |
| “Imprimir al cobrar” sí/no | Producto (en Frig: ajustes de estación `PosConfig`) |
| Layout, logo, timbre, montos | Yggdra PDF |
| Quién puede ver boletas / cocina / finanzas | Yggdra plan × rol |

Frig ya guarda el último `thermal` / `thermal_58` en `localStorage` (`src/lib/pdf-format.ts`). En v1 Y Print es la fuente de verdad del ancho de **ese PC**; Frig lee el ancho al imprimir (`GET /v1/status`) y pide el PDF con ese `pdf_format`.

## Fallos

| Caso | Comportamiento |
|------|----------------|
| Y Print no está instalado / apagado | La SPA muestra “Instalá Y Print” y deja descargar PDF (camino actual) |
| Impresora apagada | Job falla con error claro; no se traga el silencio |
| Token vencido | Y Print pide login otra vez; los jobs esperan o fallan 401 |
| PDF más ancho que el rollo elegido | Rechazo: no se escala “para que quepa” (eso es el margen feo) |
