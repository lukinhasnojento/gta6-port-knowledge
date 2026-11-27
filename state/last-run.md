# Execução de 2026-11-26 23:00 (-03:00)

- **Início da coleta:** 2026-11-27T02:03Z (23:03 local). **Gravação do estado:** 2026-11-27T02:14Z.
- **Estado lido:** `baselined=true`, `runs=8`, `updated=2026-11-27T01:11:27Z`. Execução de delta, não de
  baseline.
- **Janela:** os cursores de `state/watch.json`, cada um no seu horário (de `2026-11-26T04:15:00Z` a
  `2026-11-27T01:11:27Z`), até a coleta. Não é janela de uma hora: os repositórios em `backlog` tinham
  cursor de várias horas atrás.

## Repositórios com delta (10)

| Repositório | Commits na faixa | PRs abertos novos | PRs mesclados na janela | Situação |
|---|---|---|---|---|
| avatarmanz/VIctory-7900XTX | 25 | 18 | 0 | documentado: faixa e PRs inteiros; `commit_cursor`/`head_at_run` removidos |
| fxceless/victory-fork | 20 | 28 | 7 | documentado: 20 commits, 20 PRs abertos, 7 merges; 8 PRs abertos no backlog |
| brcly/TuxBox | 35 | 2 | 12 | documentado: faixa e PRs inteiros |
| DexNom/VIctory | 1 | 50 | 0 | documentado: 1 commit, 20 PRs abertos; 30 no backlog |
| holdmysocks/SixNG | 12 | 14 | 12 | backlog (cursor de commits `e0fd6ffa`; 12 commits, 14 abertos, 12 merges) |
| R0Y00/Xbox2Windows | 0 | 43 | 16 | backlog (16 merges para `main`; 43 abertos) |
| ivall/xcl | 0 | 50 | 6 | backlog |
| Marvisak/VIctory | 0 | 50 | 0 | backlog |
| Belobug/VIctory-Belobug-AMD | 0 | 50 | 20 | backlog |
| omonkeyz/xbox-emulator | 0 | 1 (#395) | 0 | backlog |

**Sem delta (3):** `californiaAlmonds/OpenScarlett` (head `48a98fcd` = cursor; o único aberto é o #60, já
coberto por `last_pr_number`, e nenhum merge depois de `07:52:24Z`), `uiliannn/VItoriaBr` (head
`1361e190` = cursor; maior aberto 126 = `last_pr_number`) e `walkthestars/GTA6-Asset-Info` (head
`bad5bc21` = cursor; sem PRs).

## Documentos escritos

Quatro, em `docs/updates/2026-11-26/23/`: `VIctory-DexNom.md`, `VIctory-7900XTX.md`, `victory-fork.md`,
`TuxBox.md`. `docs/updates/INDEX.md` ganhou uma linha e um parágrafo.

Tetos aplicados:
- **4 repositórios por execução.** Não há ferramenta de subagente neste ambiente; processados em
  sequência. Critério: os quatro com commit pendente, do cursor mais antigo para o mais novo — `DexNom`
  (`last_run` 04:15), `VIctory-7900XTX` (08:17), `victory-fork` (18:05) e `TuxBox` (21:27). `SixNG`
  (22:01) é o quinto com commit e ficou para a próxima hora.
- **40 commits por repositório.** Nenhum corte: 25, 20, 35 e 1.
- **PRs abertos.** Documentados 40 de 98 detectados: 18 de 18 em `VIctory-7900XTX`, 2 de 2 em `TuxBox`,
  20 de 28 em `victory-fork` e 20 de 50 em `DexNom`. `last_pr_number` avançou só até o maior documentado
  (1411 e 106); o resto volta a ser detectado.

## Divergências registradas

- **PR mesclado sem commit de merge (victory-fork).** O corpo de `b1d777a8` afirma que os PRs #1397,
  #1399 e #1400 foram "merged by hand". A API não os devolveu como mesclados (nem entre os 100 fechados
  mais recentes por `updated`). Não há hash de merge a citar; o conteúdo deles está descrito dentro do
  próprio `b1d777a8`, não como tabela de PR mesclado.
- **PR #235 do TuxBox.** O `merge_commit_sha` que a API devolve (`d26d8978`) é um merge interno da branch
  do PR ("Merge commit 'd49808d6' into work/review-bindless-235-20261126"), não um `Merge pull request
  #235`. Os 12 PRs mesclados batem com 12 commits de merge na faixa, mas esse hash não é o que o padrão
  do repositório produz nos outros 11.
- **`last_run` de PRs mesclados** avançou até o `mergedAt` do último merge documentado, não até o horário
  da execução: `victory-fork` → `2026-11-26T23:06:47Z` (#1312) e `TuxBox` → `2026-11-27T01:54:42Z`
  (#291).

## Falhas

- **Nenhuma falha de coleta.** As 13 consultas de head, as 13 listas de PRs abertos, as 13 de fechados
  (`pulls?state=closed&sort=updated&per_page=100`), 5 listas de commits (`per_page=100`) e 81 leituras de
  `commits/<sha>` responderam. Nenhum 403.
- **Nenhum `_falhas.md`.** Todo repositório com delta não documentado entrou em `backlog`.
- **Nenhum hash, PR ou número inventado.** Todo fato das tabelas de commits sai do `patch` da API; o que
  não deu para verificar está marcado como não verificado em cada documento.

## Push

Pendente no momento da gravação deste arquivo; o resultado e os hashes entram no commit de estado
seguinte, sobre a `public`, e são mesclados na `main` pelo mesmo caminho.
