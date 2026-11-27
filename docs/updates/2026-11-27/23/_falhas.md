# Falhas da execução de 2026-11-27 23:00

## Dois repositórios inacessíveis (HTTP 404, terceira hora seguida)

| Repositório | Primeiro 404 registrado | O que foi tentado nesta hora | Resultado |
|---|---|---|---|
| `Belobug/VIctory-Belobug-AMD` | 2026-11-27T21:05Z | `gh api repos/Belobug/VIctory-Belobug-AMD --jq .default_branch`; `gh api users/Belobug` | `HTTP 404 Not Found` nas duas. Sem redirect |
| `avatarmanz/VIctory-7900XTX` | 2026-11-27T21:05Z | `gh api repos/avatarmanz/VIctory-7900XTX --jq .default_branch`; `gh api users/avatarmanz` | `HTTP 404 Not Found` nas duas. Sem redirect |

Consequências, todas aplicadas:

- `default_branch` **não** pôde ser reconferida — a regra 1 do procedimento não foi executada para os
  dois. Os valores gravados (`a-publier` e `main`) ficam como estão, marcados inverificados.
- Cursor de commits e de PRs **parado**; nada foi documentado deles.
- Triagem dos dois registrada como `NAO VERIFICADO`, prioridade `bloqueado`, em `_triagem.md`.
- `unreachable` mantido em `state/watch.json`; `last_error` atualizado com a terceira hora.
- As duas entradas **não** saíram de `repos` (a contagem ficou em 14).

Não é falha de coleta nossa: `users/<login>` também devolve 404, o que indica conta renomeada ou removida
sem redirect. Ação sugerida para o operador: confirmar o nome novo e atualizar a chave em
`state/watch.json` à mão — a rotina não adivinha nomes (regra 2: nunca reconstruir `repos`).

## Nenhuma outra falha

- 12 repositórios responderam a `repos/<o>/<r>`, `commits/<branch>` e `compare`.
- Nenhum `403` de rate limit: 4.838 restantes no início, 4.921 depois do reset da janela, 0 durante a
  execução.
- Nenhum `compare` com status `diverged`.
- Nenhum `last_commit` nulo — não houve baseline nesta execução.
