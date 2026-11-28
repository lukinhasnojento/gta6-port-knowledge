# Falhas e desvios — 2026-11-28 19:00

## Repositórios inacessíveis (404), décima terceira hora

| Repositório | O que falhou | Efeito |
|---|---|---|
| `Belobug/VIctory-Belobug-AMD` | `gh api repos/Belobug/VIctory-Belobug-AMD` → HTTP 404 `Not Found`; a branch gravada é `a-publier` e não foi possível conferir | Cursor parado em `4eb6cfdf` / #456 / `2026-11-27T16:52:39Z`. Triagem NAO VERIFICADO. A entrada continua em `repos` |
| `avatarmanz/VIctory-7900XTX` | `gh api repos/avatarmanz/VIctory-7900XTX` → HTTP 404 `Not Found` | Cursor parado em `014d52de` / #419 / `2026-11-26T08:17:50Z`. Triagem NAO VERIFICADO. A entrada continua em `repos` |

Não é rate limit: um 404 de `Not Found` na API REST para o repositório, e não 403. As duas entradas
seguem no estado, como manda a regra de nunca reduzir `repos`.

## Desvio de cursor no `ivall/xcl`

O `last_commit` gravado às 18:00 (`f2c0d36ff01a7b5c7d2a58e682249a5e8bfc4647`) **não é ancestral** de
todo o histórico já documentado: `compare/86a90eebad29...f2c0d36f` devolve `diverged` (5 à frente, 3
atrás), e `86a90eeb` foi documentado às 18:00. Consequência medida nesta hora: o `compare` a partir do
cursor devolve **45** commits, dos quais **35 já estão em `docs/updates/*/xcl.md`**. O número "38
restantes" registrado às 18:00 (calculado como 78 − 40) não se reproduz.

Tratamento: os 35 foram pulados **por hash**, os 10 sem documento foram documentados, e o cursor avança
para o head `10d272970f25b02b72ebe3c883d02d74d4d4763b`, que zera a fila de commits. Nenhum documento
carrega hash inventado e nenhum commit foi documentado duas vezes.

## Outros desvios

- **`#3199` do `xcl` fechado durante a coleta.** Estava aberto às 19:07Z (lido por `pulls/3199`) e não
  apareceu na janela de 50 abertos às 19:2xZ; conferido: `state=closed`, `merged=false`. Registrado como
  fechado sem merge, e não como mesclado.
- **PR #3390 e #3392 do `Xbox2Windows` não aparecem** na janela de 50 abertos, embora #3389 e #3391
  apareçam. Não foram inventados nem presumidos; a triagem registra só os que a API devolveu.
- **`#3202` do `xcl` e dez PRs do `NVIDIA` chegaram depois do primeiro snapshot.** A lista de abertos foi
  lida duas vezes; a tabela do documento usa o segundo snapshot e a triagem registra a diferença.
- **Ferramenta de coleta.** `gh api --jq` com aspas duplas embutidas na expressão (por exemplo
  `split("\n")`) é desmontado pelo PowerShell 5.1 e volta como expressão inválida ou com argumentos
  separados. Nesta execução isso aconteceu quatro vezes, sem dano: a coleta foi refeita com `.sha`,
  `.commit.author.date` e `.commit.message` separados e o tratamento de texto no próprio PowerShell.
- **Nenhum 403 de rate limit** nesta coleta (4.833 restantes no início).
- **Escrita:** somente em `G:\gta6-port-knowledge`. Os 15 monitorados foram só leitura de API.
