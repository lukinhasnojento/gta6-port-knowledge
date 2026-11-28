# Falhas de 2026-11-28 01:00

## `Belobug/VIctory-Belobug-AMD` — HTTP 404 (quinta hora)

- `gh api repos/Belobug/VIctory-Belobug-AMD --jq .default_branch` → `gh: Not Found (HTTP 404)`.
- `default_branch` gravado no estado é `a-publier`; não pôde ser reconferido.
- Cursor de commits (`4eb6cfcf`), de PRs abertos (456) e de merges (`last_run` 2026-11-27T16:52:39Z)
  ficaram parados. Nada foi documentado; a triagem desta hora é **NAO VERIFICADO**.
- O repositório continua no estado; não foi removido.

## `avatarmanz/VIctory-7900XTX` — HTTP 404 (quinta hora)

- `gh api repos/avatarmanz/VIctory-7900XTX --jq .default_branch` → `gh: Not Found (HTTP 404)`.
- Cursor de commits (`014d52de`), de PRs abertos (419) e de merges (`last_run` 2026-11-26T08:17:50Z)
  ficaram parados. Nada foi documentado; a triagem desta hora é **NAO VERIFICADO**.
- O repositório continua no estado; não foi removido.

## Sem outras falhas

Os outros 12 repositórios responderam a todas as consultas (branch default, `compare`, PRs abertos,
PRs mesclados por API REST). Nenhum 403 de rate limit nesta hora (4.833 chamadas restantes no início).
