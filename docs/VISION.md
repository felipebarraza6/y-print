# Visión — Y Print

Y Print es el **brazo de impresión** de la familia Yggdra. Un solo programa en el PC del local. Todas las apps que ya hablan con la API pueden usarlo.

## Por qué no vive dentro de Frig

Frig es un frontend web (Next.js) que corre en el navegador. El navegador no puede hablar USB ni mandar un PDF al rollo sin márgenes de página.

La API (`yggdra_arq`) es el núcleo de negocio de **todos** los productos: gastronomía (Frig), IoT / Smart Hydro, coaching (Proyecto 90), agenda, agentes, etc. El mismo tenant, el mismo `Token`, el mismo `X-Branch-ID`, el mismo `frontend-config`.

Si el agente fuera “Frig Print”, cada producto nuevo repetiría instalador, pairing y drivers. Si es **Y Print**, Frig es el primer cliente. El siguiente producto solo implementa el contrato localhost.

## Analogía de familia

| Nombre | Rol |
|--------|-----|
| Yggdra (`yggdra_arq`) | API, datos, PDF, permisos, planes |
| Frig | SPA de operación (POS, ventas, caja, cocina) |
| Y Print | Proceso nativo en el PC de la impresora |
| Otros frontends | Mismos headers, mismas sucursales, otros módulos |

El prefijo **Y** marca pertenencia a Yggdra, igual que la API. No es un plugin de un solo POS.

## Qué ve el usuario

1. Se baja e instala Y Print en el PC de caja (o cocina, o oficina).
2. Entra con el mismo usuario Yggdra (email + password → `login_complete`).
3. Elige sucursal (`X-Branch-ID`).
4. La UI muestra **solo** las capacidades que su plan y rol permiten.
5. Empareja ese PC con la app web (Frig u otra) una vez.
6. A partir de ahí, imprimir es nativo.

Un cajero de gelatería ve boleta 58/80, voucher, comanda. Un operador de otro producto, el día que exista un job de etiquetas o A4, verá eso si el módulo está en su plan. El motor de impresión es el mismo.

## Cómo se enriquece

Y Print **no** conoce cada pantalla de Frig. Conoce:

- una sesión Yggdra (token + sucursal + permisos)
- impresoras locales
- un catálogo de **job types** (`thermal.boleta`, `thermal.ticket`, `thermal.voucher`, luego `label.*`, `a4.*`)
- un HTTP local estable (`docs/LOCAL-API.md`)

Cada producto de la familia:

1. Genera o pide el documento a Yggdra (PDF ya existe para lo térmico).
2. Llama `POST /v1/print` en localhost con el job type y el PDF.
3. Respeta el ancho que el usuario configuró en Y Print (58 o 80 mm).

Agregar un uso nuevo es: un job type + permiso de módulo + (si hace falta) un generador PDF en Yggdra. El instalador no cambia.

## Éxito

- El comprobante térmico sale al rollo sin margen extra de Chrome.
- El local solo respeta el mm del papel.
- Frig deja de usar `window.print()` para esos documentos.
- Otra app de la familia puede imprimir el mismo día, sin fork del agente.
