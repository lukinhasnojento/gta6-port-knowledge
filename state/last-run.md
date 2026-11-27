# Execução de 2026-11-27 16:00

## Data e hora

Coleta de 2026-11-27T13:00–13:25 no relógio dos dados do GitHub (16:00–16:25 UTC real; 10:00–10:25 no
horário local do operador, UTC−3). Diretório de documentos: `docs/updates/2026-11-27/16/` (UTC).
Estado lido: `baselined: true`, `runs: 13` — a etapa de baseline não se aplicou.

## Repositórios com delta

Delta em 4 dos 13, já filtrado pelo critério de prioridade do `backlog_note` anterior (repositório com
commit ou merge na janela primeiro, depois só-PR-aberto pelo `last_run` mais antigo):

| Repositório | Commits na faixa | Mesclados na janela | Abertos acima do cursor | Documentado |
|---|---|---|---|---|
| `R0Y00/Xbox2Windows` | 80 (`compare`) | 50 | 14 | 40 / 50 / 14 |
| `brcly/TuxBox` | 19 (`compare`) | 2 | 0 | 19 / 2 / 0 |
| `ivall/xcl` | 5 (`compare`) | 3 | 50 (piso) | 5 / 3 / 50 |
| `Marvisak/VIctory` | 0 | 0 | 50 (piso) | 0 / 0 / 50 |

Sem delta, reconferido: `californiaAlmonds/OpenScarlett`, `uiliannn/VItoriaBr`,
`walkthestars/GTA6-Asset-Info` (branch parada, nenhum merge na janela),
`Belobug/VIctory-Belobug-AMD`, `DexNom/VIctory`, `avatarmanz/VIctory-7900XTX`,
`omonkeyz/xbox-emulator`, `fxceless/victory-fork` (só PRs abertos, sem commit nem merge; consumiram a vez).

Em `backlog` (7): `R0Y00/Xbox2Windows` (os 40 commits acima de `f52046900a` e os diffs do `#3055` e do
`#2882`), `DexNom/VIctory`, `omonkeyz/xbox-emulator`, `avatarmanz/VIctory-7900XTX`,
`Belobug/VIctory-Belobug-AMD`, `holdmysocks/SixNG` e `fxceless/victory-fork`. Contagens medidas nesta hora
e a prioridade da próxima estão em `state/watch.json`.

## Documentos escritos

4 documentos em `docs/updates/2026-11-27/16/`: `Xbox2Windows.md`, `xcl.md`, `TuxBox.md`, `VIctory.md`.
Total: **64 commits** (40 + 19 + 5 + 0), **55 PRs mesclados** (50 + 2 + 3) e **64 PRs abertos**
(50 + 50 + 14 + 0, nos quatro onde há delta).

`R0Y00/Xbox2Windows` é o único com corte: a faixa `e62e3a5e..801f4c3b` tem 80 commits e o teto é 40. Os 40
mais antigos foram documentados (até `f52046900a`), e os 40 seguintes ficam com `commit_cursor` em
`f52046900a` e `head_at_run` em `801f4c3b`, voltando na próxima hora. Os 50 merges da janela entraram todos
na tabela do documento, mas 42 deles caíram fora da faixa de commits documentada e estão marcados como
`NÃO VERIFICADO` (só número, título e sha de merge).

## Falhas e desvios

- **`gh pr list --state merged --search "merged:>..."` não serve.** Devolveu lista vazia no `ivall/xcl`
  (repositório transferido de `xcl-dev/xcl`) e 49 dos 50 merges do `Xbox2Windows`, perdendo o `#2882`
  (`db28a796`, `mergedAt 2026-11-27T13:14:52Z`). A coleta passou a usar a API REST
  `pulls?state=closed&sort=updated&direction=desc`, paginada de 100 em 100 até o `updated_at` mais antigo
  cair abaixo do `last_run`. Registrado no campo `note` do estado.
- **Dois relógios.** Os timestamps do GitHub ficaram 3 h atrás do UTC real nesta coleta: o `#3055` do
  `Xbox2Windows` traz `mergedAt 2026-11-27T13:06:51Z` e foi visto às 16:06Z. A execução de 15:00 gravou
  `last_run = 2026-11-27T16:05:00Z` em quatro repositórios **sem merge** (`OpenScarlett`, `VItoriaBr`,
  `victory-fork`, `GTA6-Asset-Info`) — três horas no futuro do relógio dos dados, o que esconderia
  qualquer merge até o relógio dos dados alcançar aquele instante. Os quatro foram reconferidos contra
  `2026-11-26T00:00Z` (zero merges novos: `OpenScarlett` só o #50 de `2026-11-26T07:52:24Z`, já
  documentado; `victory-fork` 14 merges, todos até `2026-11-26T23:06:47Z`, já documentados) e
  normalizados para `2026-11-27T13:05:00Z`. `VIctory`, que não teve merge, foi avançado para o instante da
  consulta no relógio dos dados (`13:20:45Z`), não para o horário da execução.
- **Repositórios se movem durante a execução.** No `TuxBox` o primeiro `compare` devolveu 16 commits e a
  reconferência 19 — `eb2a5000` (data de commit anterior) foi inserido no meio da lista e o `#308` chegou
  com dois commits. No `Xbox2Windows` o `#3055` (`13:06:51Z`) e o `#2882` (`13:14:52Z`) entraram depois do
  `compare` de commits. As contagens publicadas são as da reconferência, e o que chegou depois fica para a
  próxima hora pelo `commit_cursor`.
- **Sem subagentes.** A ferramenta de subagente não existe neste ambiente; os quatro repositórios foram
  processados em sequência. O limite de 4 por execução foi respeitado.
- **Rate limit**: `gh api rate_limit` marcava 4.621 de 5.000 no início (379 usados, sobra de execuções
  anteriores) e 30 de 30 na busca; nenhuma chamada devolveu 403.
- **Abertos acima do cursor**: `xcl` e `VIctory` estão no piso da janela de 50, então o total real acima do
  cursor é ≥ 50 em ambos; os 50 de cada foram catalogados um a um, mas nenhum patch foi lido — a coluna
  "Proposta" vem do título e do nome da branch, como o documento diz.
- Nenhum `_falhas.md`: não houve erro de coleta ou de escrita em repositório.

## Push

- Commit na `public`: `b6d0be4` — `docs(updates): 2026-11-27 16:00` (7 arquivos, 575 inserções, 92
  remoções: os 4 documentos, `docs/updates/INDEX.md`, `state/watch.json` e `state/last-run.md`).
- `git push origin public:main`: **sucesso** — `6820f51..b6d0be4  public -> main`.
- `git switch main && git merge --no-ff public -m "merge updates 2026-11-27 16:00"`: **sucesso**, merge
  `5b4e29e`; `git push private main`: **sucesso** — `d753ac5..5b4e29e  main -> main`.
- Conferido depois: `git rev-parse origin/main` = `b6d0be4fc9c78342b794814b4838541675fdf260`;
  `git rev-parse private/main` = `5b4e29e2014c2fc05857bfdc33fa812356baed34`.
- Fechamento (este registro): commit na `public` e merge `--no-ff` na `main`, com o resultado no fim deste
  arquivo. Nenhum merge de `main` na `public` em nenhum momento: a `public` só recebeu commits próprios e a
  `main` recebeu a `public`.

