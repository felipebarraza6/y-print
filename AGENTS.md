# Agentes — Y Print

Leé `README.md` y `docs/` antes de tocar código. El producto es un **agente de impresión de la familia Yggdra**, no un módulo de Frig.

## Repos

- API: `https://github.com/felipebarraza6/yggdra_arq`
- POS (primer cliente): `https://github.com/felipebarraza6/frig`
- Este repo: contrato + app nativa

## Reglas

1. Yggdra genera el PDF. Y Print imprime. No clones layouts de boleta.
2. Auth = `login_complete` + `X-Branch-ID` + `frontend-config`. No inventes roles.
3. Capacidades = plan × rol × módulos. Job types en `docs/PRINT-JOBS.md`.
4. El contrato localhost (`docs/LOCAL-API.md`) es la interfaz de **todas** las apps de la familia. Breaking change = `/v2`.
5. v1 es Windows, una impresora por PC, térmico 58/80 mm.
6. No integres Frig hasta el corte 4 del roadmap.
7. No subas tokens ni PDFs de clientes.

## Issues

Trabajá contra las issues de este repo. Una issue, un corte. El orden está en `docs/ROADMAP.md`.
