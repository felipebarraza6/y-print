# Contrato localhost (familia Yggdra)

Cualquier producto de la familia habla con Y Print aquí. Versión **v1**. Breaking change = bump a `/v2`.

Base: `http://127.0.0.1:17380` (puerto fijo de Y Print; si está ocupado, el siguiente libre se anuncia en un archivo de instancia — ver abajo).

Solo loopback. TLS no en v1 (localhost).

## Descubrimiento del puerto

Default `17380`. Si no responde:

1. Leer `%LOCALAPPDATA%\YPrint\instance.json` → `{ "port": 17380, "pid": … }`.
2. Si no hay archivo ni proceso, la SPA asume “no instalado”.

## Auth de la SPA hacia Y Print

No se reenvía el Token de Yggdra en cada print (el blob ya es el documento). Se usa **pairing**:

1. `GET /v1/status` es público en loopback (sin secretos de cuenta).
2. `POST /v1/pair/start` en Y Print genera un código de 6 dígitos (UI del agente).
3. La SPA manda `POST /v1/pair` `{ origin, code }`.
4. Respuesta: `{ station_token }` (opaco, por origin + PC).
5. Jobs: header `X-YPrint-Station: <station_token>` y `Origin` del SPA.

El usuario confirma en la bandeja la primera vez. Revocar: Y Print → Emparejamientos.

## Endpoints

### `GET /v1/health`

```json
{ "ok": true, "name": "y-print", "version": "0.1.0" }
```

### `GET /v1/status`

```json
{
  "ok": true,
  "authenticated": true,
  "api_base": "https://api.yggdra.cl/api",
  "branch_id": 5,
  "branch_name": "Bizantni Gelato",
  "paper_mm": 80,
  "printer_id": "usb:EPSON-TM-T20",
  "printer_name": "EPSON TM-T20",
  "capabilities": ["thermal.boleta", "thermal.ticket", "thermal.voucher"],
  "paired_origins": ["http://localhost:3000"]
}
```

`paper_mm` es `58` o `80`. La SPA pide el PDF con `pdf_format=thermal_58` o `thermal`.

`capabilities` ya viene filtrado por permisos de la sesión del agente. Si el agente no está logueado, `capabilities` puede ser `[]` y igual se aceptan jobs **blob** de origins emparejados (el documento lo trajo la SPA con *su* permiso).

### `GET /v1/printers`

Requiere estación emparejada o la UI nativa.

```json
{
  "printers": [
    { "id": "usb:EPSON-TM-T20", "name": "EPSON TM-T20", "kind": "windows" }
  ]
}
```

### `POST /v1/print`

Header: `X-YPrint-Station`. Body JSON:

```json
{
  "job_type": "thermal.boleta",
  "paper_mm": 80,
  "copies": 1,
  "printer_id": null,
  "client": "frig",
  "blob_base64": "<pdf bytes>",
  "resource": null
}
```

O modo resource (sesión del agente):

```json
{
  "job_type": "thermal.boleta",
  "paper_mm": 80,
  "copies": 1,
  "client": "frig",
  "resource": {
    "app": "sales",
    "name": "order",
    "id": "uuid-or-int",
    "action": "boleta",
    "pdf_format": "thermal"
  }
}
```

Reglas:

- Exactamente uno de `blob_base64` o `resource`.
- `paper_mm` debe coincidir con el ancho del PDF. Si no, `409` `{ "error": "paper_mismatch", "expected_mm": 80 }`. No se escala.
- `printer_id` null → impresora por defecto de Y Print.
- `job_type` desconocido → `400`.
- `job_type` no en capabilities **y** no hay blob emparejado → `403`.

Respuesta 202:

```json
{ "job_id": "…", "state": "queued" }
```

Errores: `401` sin pairing, `404` impresora, `502` Yggdra al bajar resource, `503` impresora no lista.

### `GET /v1/jobs/{job_id}`

```json
{ "job_id": "…", "state": "printed" | "queued" | "failed", "error": null }
```

v1 puede resolver síncrono y devolver `200` con `state: printed` en el mismo `POST /v1/print` si el spool es inmediato. El `job_id` igual se emite para logs.

### `POST /v1/pair`

```json
{ "origin": "http://localhost:3000", "code": "482193" }
```

→ `{ "station_token": "…" }`

### `POST /v1/pair/start`

Solo desde la UI de Y Print (no la SPA). Crea código con TTL corto (2 min).

## Clientes previstos

El campo `client` es informativo (métricas locales): `frig`, `muninn`, `p90`, `smarthydro`, `unknown`. No cambia el motor.

## CORS

Respuesta a origins emparejados:

```
Access-Control-Allow-Origin: <origin emparejado>
Access-Control-Allow-Headers: Content-Type, X-YPrint-Station
Access-Control-Allow-Methods: GET, POST, OPTIONS
```

Nunca `*`. Preflight OPTIONS sin token.

## SDK

v1 no publica npm. Frig implementa 40 líneas (`fetch` al puerto). Cuando haya segundo cliente, extraer `@yggdra/print-client` en este repo o en un paquete de familia. No antes.
