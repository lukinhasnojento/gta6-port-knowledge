# Falhas da execução de 2026-11-28 00:00

## Dois repositórios inacessíveis (HTTP 404, quarta hora seguida)

| Repositório | Primeiro 404 registrado | O que foi tentado nesta hora | Resultado |
|---|---|---|---|
| `Belobug/VIctory-Belobug-AMD` | 2026-11-27T21:05Z | `gh api repos/Belobug/VIctory-Belobug-AMD`; `gh api users/Belobug` | `HTTP 404 Not Found` nas duas. Sem redirect |
| `avatarmanz/VIctory-7900XTX` | 2026-11-27T21:05Z | `gh api repos/avatarmanz/VIctory-7900XTX`; `gh api users/avatarmanz` | `HTTP 404 Not Found` nas duas. Sem redirect |

Consequências, todas aplicadas:

- `default_branch` **não** pôde ser reconferida — a regra 1 do procedimento não foi executada para os
  dois. Os valores gravados (`a-publier` e `main`) ficam como estão, marcados inverificáveis.
- Cursor de commits e de PRs **parado**; nada foi documentado deles.
- Triagem dos dois registrada como `NAO VERIFICADO`, prioridade `bloqueado`, em `_triagem.md`.
- `unreachable` mantido em `state/watch.json`; `last_error` atualizado com a quarta hora.
- As duas entradas **não** saíram de `repos` (a contagem ficou em 14).

Não é falha de coleta nossa: `users/<login>` também devolve 404, o que indica conta renomeada ou removida
sem redirect. Ação sugerida para o operador: confirmar o nome novo e atualizar a chave em
`state/watch.json` à mão — a rotina não adivinha nomes (regra 2: nunca reconstruir `repos`).

## Falha de coleta corrigida na hora (não é falha de repositório)

- A primeira chamada de `compare` nos 12 repositórios respondidos montou a URL com o SHA devolvido por
  `gh api ... --jq .sha`, que vem com `\n` final: `net/url: invalid control character in URL` em todos os
  12. O SHA foi aparado e a medição refeita. Nenhum número desta execução vem da tentativa que falhou;
  nenhum cursor foi gravado a partir dela.

## Nenhuma outra falha

- 12 repositórios responderam a `repos/<o>/<r>`, `commits/<branch>`, `compare` e `pulls`.
- Nenhum `403` de rate limit: 4.999 restantes no início da coleta e nenhum 403 durante a execução.
- Nenhum `compare` com status `diverged` (12 medições: 4 `ahead`, 8 `identical`).
- Nenhum `last_commit` nulo — não houve baseline nesta execução.
- Nenhum dos 4 deep dives chegou ao teto de 40 commits (o maior foi 12).
