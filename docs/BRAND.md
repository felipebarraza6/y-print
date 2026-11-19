# Marca — mudo, skin de la app del usuario

Y Print **no** es una marca que se vea en caja. Es un servicio. La cara que ve el local es la de **la app que ya usa** (Frig, u otra de la familia) y el tema de su sucursal/organización.

Misma regla que Frig autenticado: no estampar el nombre de plataforma en cada texto. Fuente: `frontend-config.organizationTheme` (`app_name`, `logo`, `primary_color`, `favicon`).

## Antes de login (instalación, primer arranque)

Mínimo absoluto:

- Ventana: título `Impresión` (una palabra).
- Icono de bandeja y taskbar: glifo de impresora, un color, sin wordmark.
- Copy: “Entrar con tu cuenta”, no “Bienvenido a Y Print”.
- Sin logo Yggdra, sin cobre Frig, sin slogan.

El binario en Windows (Add/Remove Programs, exe) puede llamarse **Y Print** para la familia técnica. Eso no se replica en la UI de operación.

## Después de login

El chrome toma el tema de la sucursal activa:

| Superficie | Qué muestra |
|------------|-------------|
| Título de ventana | `{app_name}` o `{branch.business_name}` · Impresión |
| Logo | `organizationTheme.logo` (media URL Yggdra). Si no hay: iniciales del `app_name`. |
| Color de acento | `organizationTheme.primary_color` |
| Favicon / bandeja | `favicon` o `logo` del tema |
| Capacidades | Solo módulos de esa sucursal |
| Tooltip bandeja | `Impresión · {app_name}` |

Orden de nombre (igual espíritu que Frig `productDisplayName`):

1. `organizationTheme.app_name`
2. `branch.business_name` / `fantasy_name`
3. `branch.branch_name`
4. `Impresión` (nunca un fallback ruidoso “Y PRINT”)

Si el `app_name` contiene “frig” y el local opera Frig, se puede mostrar **FRIG** en chrome: es la app configurada, no la marca del agente.

## Qué no hacer

- Splash “Y PRINT” a pantalla completa.
- Wordmark Yggdra en el login de caja.
- Copiar el landing de Frig (plexus, cobre, GitHub) dentro del agente.
- Un skin distinto por `client: frig` hardcodeado: el tema sale de **Yggdra**, igual que el SPA. Si mañana el mismo usuario entra a otra sucursal, el agente cambia de cara.

## Instalador (corte 6)

| Dónde | Nombre |
|-------|--------|
| MSI / exe / Releases | `Y Print` (un binario para toda la familia) |
| Carpeta Start Menu | `Y Print` |
| UI una vez abierto | reglas de arriba |

About (escondido, tres clics): `Y Print {version}` — para soporte, no para el cajero.

## Status localhost

`GET /v1/status` puede incluir `app_name` y `primary_color` ya resueltos, para que Frig muestre “conectado a Impresión · {app_name}” y no “conectado a Y Print”.
