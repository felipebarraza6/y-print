# Y Print

Agente de impresión nativo de la **familia Yggdra**. Se instala en el PC que tiene la impresora, se autentica contra la API (`yggdra_arq`) y muestra opciones según el plan, el rol y los módulos de esa sucursal.

Cualquier producto de la familia (Frig, y los que vengan) habla con Y Print en `localhost`. Y Print habla con Yggdra y con la impresora. El navegador no imprime.

**Repos relacionados**

| Pieza | Repo |
|-------|------|
| API | [`yggdra_arq`](https://github.com/felipebarraza6/yggdra_arq) |
| POS / gastronomía | [`frig`](https://github.com/felipebarraza6/frig) |
| Este agente | [`y-print`](https://github.com/felipebarraza6/y-print) |

## Qué resuelve

Hoy Frig descarga un PDF térmico (58 mm o 80 mm) o llama `window.print()`. Chrome trata el rollo como una hoja y le pone margen. Yggdra ya genera el PDF con ancho correcto (`thermal` / `thermal_58`); el último metro (PC → impresora) no existía.

Y Print imprime ese PDF 1:1, sin diálogo de Chrome. El local solo elige el ancho del rollo.

## Principio

Yggdra genera el documento (logo, montos, timbre SII). Y Print no redibuja boletas. Imprime. Las capacidades que ve el usuario salen de `login_complete` + `frontend-config` (plan × rol × módulos).

## Documentación

| Doc | Contenido |
|-----|-----------|
| [docs/VISION.md](docs/VISION.md) | Producto de familia, no solo Frig |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | Piezas, flujo, stack |
| [docs/REQUIREMENTS.md](docs/REQUIREMENTS.md) | Requisitos extraídos de Frig + Yggdra |
| [docs/AUTH-AND-PERMISSIONS.md](docs/AUTH-AND-PERMISSIONS.md) | Login, sucursal, qué se muestra |
| [docs/LOCAL-API.md](docs/LOCAL-API.md) | Contrato localhost para cualquier app de la familia |
| [docs/PRINT-JOBS.md](docs/PRINT-JOBS.md) | Tipos de trabajo v1 y extensión |
| [docs/FRIG-INTEGRATION.md](docs/FRIG-INTEGRATION.md) | Cómo se enchufa el POS |
| [docs/ROADMAP.md](docs/ROADMAP.md) | Orden de issues |

El desarrollo arranca por las **issues** de este repo. No hay código de aplicación todavía a propósito.

## v1 (alcance)

- Windows
- Una impresora por estación / PC
- Comprobantes térmicos 58 mm y 80 mm que Yggdra ya genera
- Login Yggdra, sucursal, capacidades por permiso
- API local para que Frig (y otras apps) manden a imprimir
- Toggle “imprimir al cobrar” vive en el producto (Frig POS); Y Print ejecuta

## Fuera de v1

Android / iPad, varias impresoras por tipo de documento, ESC/POS texto, etiquetas de nutrición, A4 de reportes, firma de código Windows (SmartScreen) en el primer corte.
