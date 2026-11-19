# Auth y permisos

Y Print es un cliente más de Yggdra. Misma identidad que Frig.

## Login

```
POST /api/accounts/users/login_complete/
Content-Type: application/json

{ "email": "...", "password": "..." }
```

Respuesta (Frig `LoginCompleteResponse`):

- `token` — DRF Token. Header `Authorization: Token <token>`.
- `user` — incluye `type_user`, asignaciones.
- `branches` — sucursales a las que puede entrar.
- `owned_organizations`
- `permissions.enabled_apps` | `read_only_apps` | `disabled_apps`
- `permissions.user_role`

Base URL configurable (prod `https://api.yggdra.cl/api`, local `http://localhost:8000/api`). Igual que `NEXT_PUBLIC_YGGDRA_API_BASE` en Frig.

El token se guarda en **Windows Credential Manager**, no en un archivo de texto. Logout: `POST /accounts/users/logout/` y borrar credencial.

## Sucursal

Toda llamada autenticada lleva:

```
X-Branch-ID: <id entero o UUID según el tenant>
```

El middleware de Yggdra resuelve `request.branch`. Sin acceso → 403. Y Print no permite “inyectar” otra sucursal: el selector solo lista `login_complete.branches`.

Al cambiar de sucursal se vuelve a pedir `frontend-config`.

## Bootstrap de capacidades

```
GET /api/shared/frontend-config/?branch_id=<id>
```

De ahí sale lo que la UI de Y Print enciende:

| Campo | Uso en Y Print |
|-------|----------------|
| `modules[name].is_enabled` | El job type de ese módulo existe o no |
| `permissions` | Read vs write (reimprimir vs solo ver estado) |
| `role` / `user_role` | Copy (cajero vs owner) |
| `organizationTheme` | Skin mudo: `app_name`, logo, `primary_color`, favicon. Ver [BRAND.md](BRAND.md). |

`GET /api/shared/module-permissions/module-catalog/` es opcional (metadatos). No es el runtime.

## Mapa módulo Yggdra → capacidad Y Print (v1)

El producto es **plan × rol**. Y Print no hardcodea “si es Frig”. Hardcodea job types ligados a módulos que la API ya usa.

| Capacidad Y Print | Módulo(s) Yggdra | Job types |
|-------------------|------------------|-----------|
| Boleta de orden | `sales` | `thermal.boleta` |
| Comanda cocina | `sales` (y POS si el plan lo tiene) | `thermal.ticket` |
| Voucher de pago / ingreso | `finance` | `thermal.voucher` |
| Configurar impresora de este PC | cualquiera autenticado en la sucursal | (local, no API) |

Si `sales` está en `disabled_apps` o `modules.sales.is_enabled === false`, no se muestra boleta/comanda. Si el usuario es VIEWER con read-only en finance, puede reimprimir voucher solo si el ViewSet de Yggdra lo permite (GET). Y Print no relaja el 403.

Roles típicos Frig (operativos en el SPA): cajero, mesero, cocinero, owner/admin. Y Print no replica `isNavHrefAllowed`. Pregunta a `frontend-config` y a los GET de PDF.

## Apps de la familia

Cualquier frontend que ya haga `login_complete` puede:

1. Emparejar origin con Y Print, **o**
2. Pedirle al usuario que inicie sesión en Y Print (el agente baja el PDF con su propio token).

Dos modos de job (ver LOCAL-API):

- **blob**: la SPA ya tiene el PDF (su sesión). Y Print no llama a Yggdra. Sirve aunque el token del agente esté frío, siempre que haya pairing.
- **resource**: la SPA manda `{ resource: "sales.order", id, action: "boleta", pdf_format }`. Y Print baja el PDF con **su** token + `X-Branch-ID`. Hace falta sesión viva en el agente.

v1 implementa **blob** (más simple, Frig ya descarga el PDF). **resource** es el que vuelve a Y Print autónomo (reimprimir desde la bandeja sin Frig abierto). Ambos caben en el mismo `POST /v1/print`.

## Superadmin / multi-org

`login_complete.owned_organizations` + lista de branches. Owner multi-sucursal elige branch igual que en Frig. Y Print no opera “sin sucursal”: los PDF son de un branch.

## Qué no se autentica contra Yggdra

Descubrimiento de impresoras, test de rollo, pairing localhost. Eso es del PC. Sin login, Y Print puede mostrar “conectá tu cuenta Yggdra” y un test de impresora **en blanco** (útil para instalar el driver antes de tener red). Los jobs con datos de negocio requieren sesión o blob emparejado.
