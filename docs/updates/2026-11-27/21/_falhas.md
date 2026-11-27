# Falhas da execução de 2026-11-27 21:00

## Dois repositórios monitorados deixaram de existir para a API

| Repositório | Sintoma | O que foi tentado | Efeito |
|---|---|---|---|
| `Belobug/VIctory-Belobug-AMD` | HTTP 404 em `repos/Belobug/VIctory-Belobug-AMD` e em `users/Belobug`; `https://github.com/Belobug` responde 404 | `gh api repos/<o>/<r>`, `gh api users/<o>`, requisição HTTP direta à página do usuário, busca `search/repositories?q=VIctory-Belobug-AMD` | Não foi possível ler `default_branch`, head, commits nem PRs. Cursor parado em `4eb6cfdf` / `last_pr_number` 456 / `last_run` 2026-11-27T16:52:39Z. Nada documentado |
| `avatarmanz/VIctory-7900XTX` | HTTP 404 em `repos/avatarmanz/VIctory-7900XTX` e em `users/avatarmanz`; `https://github.com/avatarmanz` responde 404 | idem | Cursor parado em `014d52de` / 419 / 2026-11-26T08:17:50Z. Nada documentado |

A busca por nome mostra dezenas de forks dos mesmos projetos sob outros donos (`JIFO0/VIctory-Belobug-AMD`,
`HrHd/VIctory-7900XTX`, `alvarez190/VIctory-7900XTX` etc.), então o projeto não desapareceu do GitHub; o
que desapareceu foram **essas duas contas**. Não há como distinguir, pela API, conta suspensa de conta
apagada ou de repositório tornado privado — as três devolvem 404 para um cliente sem permissão.

Consequência para a rotina: as duas entradas **permanecem** em `state/watch.json` (a regra proíbe reduzir
`repos`) com os cursores intactos e um campo `unreachable` novo, e continuam no `backlog` com prioridade
**bloqueado**. Elas não podem ser triadas nem documentadas enquanto o 404 persistir.

## O que não falhou

- Nenhuma chamada devolveu 403 de rate limit; nenhum `compare` devolveu `diverged`; nenhum `last_commit`
  era nulo.
- Os outros 12 repositórios foram triados por inteiro.
- `ivall/xcl` continua sem total de PRs abertos: `search/issues` devolve HTTP 422 para o repositório
  transferido (terceira hora seguida). A contagem de PRs acima do cursor (9) veio da listagem de abertos,
  que funciona; o **total** é que não.
