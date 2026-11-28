# Falhas de coleta — 2026-11-28 17:00

## `Belobug/VIctory-Belobug-AMD` — HTTP 404 (décima primeira hora)

`gh api repos/Belobug/VIctory-Belobug-AMD` devolve `Not Found` (404). O estado continua apontando
`default_branch: a-publier`, `last_commit: 4eb6cfdf9e61cb112f1f811344ec0dd309eddd6c`,
`last_pr_number: 456`, `last_run: 2026-11-27T16:52:39Z`, com o campo `unreachable` já gravado.
Nada foi lido, nada avançou, a entrada **não** foi removida de `repos`. A triagem registra
NAO VERIFICADO. Não é rate limit: a mesma coleta leu os outros 13 repositórios sem 403.

## `avatarmanz/VIctory-7900XTX` — HTTP 404 (décima primeira hora)

`gh api repos/avatarmanz/VIctory-7900XTX` devolve `Not Found` (404). Estado preservado:
`default_branch: main`, `last_commit: 014d52dedf837d1ac660a0bd256c4704be9eeb40`,
`last_pr_number: 419`, `last_run: 2026-11-26T08:17:50Z`. Nada avançou, entrada mantida, triagem
NAO VERIFICADO.

## Sem outras falhas

- Nenhum `compare` devolveu `diverged` nesta hora (3 `ahead`, 10 `identical`, 2 sem resposta).
- Nenhum `last_commit` nulo: não houve baseline de repositório nesta execução.
- Nenhum 403 de rate limit (4.846 requisições restantes no início, 4.777 no meio da coleta).
- Nenhum repositório monitorado recebeu escrita; a coleta foi só leitura de API.
