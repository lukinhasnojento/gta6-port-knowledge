# Falhas da execução de 2026-11-27 22:00

## Duas contas continuam fora da API (segunda hora)

| Repositório | Sintoma | O que foi tentado | Efeito |
|---|---|---|---|
| `Belobug/VIctory-Belobug-AMD` | HTTP 404 em `repos/Belobug/VIctory-Belobug-AMD` (mensagem `Not Found`) | `gh api repos/<o>/<r> --jq .default_branch`; nada além disso nesta hora | `default_branch` não reconferida, head não lido, commits e PRs não medidos. Cursor parado em `4eb6cfdf` / `last_pr_number` 456 / `last_run` 2026-11-27T16:52:39Z. Nada documentado |
| `avatarmanz/VIctory-7900XTX` | HTTP 404 em `repos/avatarmanz/VIctory-7900XTX` | idem | Cursor parado em `014d52de` / 419 / 2026-11-26T08:17:50Z. Nada documentado |

O diagnóstico da hora anterior continua valendo: a conta inteira responde 404 (não é branch trocada nem
repositório renomeado), e a API não distingue conta suspensa, apagada ou repositório tornado privado. As
duas entradas **permanecem** em `state/watch.json` com os cursores intactos e o campo `unreachable`, e
seguem no `backlog` com prioridade **bloqueado**. Não foram triadas nem documentadas.

## O que não falhou

- Nenhuma chamada devolveu 403 de rate limit. `gh api rate_limit` marcava 4.889 restantes de 5.000 no
  início da coleta; não houve repetição em laço.
- Nenhum `compare` devolveu `diverged`; nenhum `last_commit` era nulo; a contagem de `repos` ficou em 14
  no início e no fim.
- Os outros 12 repositórios foram triados por inteiro; 4 entraram no deep dive e os 39 commits da janela
  foram documentados (nenhum repositório chegou ao teto de 40).
- `ivall/xcl`: a listagem de PRs abertos funcionou (`pulls?state=open`), 20 acima do cursor. O total de
  abertos do repositório não foi medido com `search/issues` nesta hora (a chamada devolve HTTP 422 para o
  repositório transferido desde as 19:00), então a linha da triagem registra "≥100" pelo teto da própria
  listagem, não um total.
