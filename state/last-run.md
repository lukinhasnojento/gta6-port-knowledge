# Execução de 2026-11-27 15:00

## Data e hora

Coleta em 2026-11-27T12:34–12:57 (UTC−3), isto é 15:34–15:57Z. Hora local do operador: 12:34–12:57
(E. South America Standard Time). Diretório de documentos: `docs/updates/2026-11-27/15/` (UTC).
Estado lido: `baselined: true`, `runs: 12` — a etapa de baseline não se aplicou.

## Repositórios com delta

Delta em 11 dos 13. Sem delta: `californiaAlmonds/OpenScarlett` (o único merge, #50, tem `mergedAt` igual ao
cursor `2026-11-26T07:52:24Z` e já foi documentado às 08:00) e `walkthestars/GTA6-Asset-Info` (branch parada
em `bad5bc21`, 0 PR).

Processados, pelo critério do `backlog_note` anterior (commit ou merge na janela primeiro, depois `last_run`
mais antigo):

| Repositório | Commits na faixa | Mesclados na janela | Abertos acima do cursor | Documentado |
|---|---|---|---|---|
| `uiliannn/VItoriaBr` | 1 | 0 | 0 (cursor 126) | 1 / 0 / 0 |
| `fxceless/victory-fork` | 1 | 0 | 35 (1428–1468) | 1 / 0 / 35 |
| `brcly/TuxBox` | 47 (`compare`) | 9 | 0 (cursor 284) | 40 / 8 / 0 |
| `holdmysocks/SixNG` | 24 | 24 | 1 (#472) | 24 / 24 / 1 |

Em `backlog` (8): `Marvisak/VIctory`, `ivall/xcl`, `R0Y00/Xbox2Windows` (46 commits e 47 merges, o maior
delta pendente), `Belobug/VIctory-Belobug-AMD`, `DexNom/VIctory`, `avatarmanz/VIctory-7900XTX`,
`omonkeyz/xbox-emulator` e `brcly/TuxBox` (os 7 commits acima de `90df685a` e o merge #306). Contagens e
prioridade registradas em `state/watch.json`.

## Documentos escritos

4 documentos em `docs/updates/2026-11-27/15/`: `TuxBox.md`, `SixNG.md`, `victory-fork.md`, `VItoriaBr.md`.
Total: **66 commits** (40 + 24 + 1 + 1), **32 PRs mesclados** (8 + 24) e **36 PRs abertos** (35 + 1).

`brcly/TuxBox` é o único com corte: a faixa `9168b2fa..07db52f84f` tem 47 commits e o teto é 40. Os 7 mais
novos (`edf9e0518c`, `6ed257873d`, `c7c84de79e`, `5995ed341c`, `3894902a29`, `cf47e6874c`, `07db52f84f`)
ficam com `commit_cursor` em `90df685ad5` e voltam na próxima hora, junto com o PR **#306**; `last_run` do
repositório para em `2026-11-27T09:37:36Z` (merge do #305).

## Falhas e desvios

- **Execução duplicada da mesma hora.** O diretório começou com `state/last-run.md` e os quatro documentos
  já escritos por uma execução anterior de 15:00 que não chegou a commitar (o `runs` do estado já estava em
  13 e a árvore tinha `state/last-run.md` modificado). Em vez de sobrescrever sem olhar, as contagens foram
  reverificadas de forma independente (`compare`, filtro estrito de mesclados, casamento dos `mergeCommit`)
  e os documentos foram adotados onde batiam; `last-run.md` e `INDEX.md` foram reescritos para ficar
  coerentes. `runs` ficou em 13 (uma execução), não 14.
- **Contagem de commits por listagem estava errada.** `gh api commits?sha=<branch>&per_page=100` ordena por
  data de commit; commits de pré-merge com data anterior ao cursor caem depois dele na lista. No `TuxBox`
  isso deu 38 em vez de 47. A fonte correta é `compare/<last_commit>...<head>` (`ahead_by`). O método errado
  está registrado no campo `note` do estado.
- **Filtro de mesclados estrito.** A instrução desta hora pedia filtrar por `numero > last_pr_number`; o
  procedimento da seção "Por que PR mesclado não usa número" proíbe isso. Usei
  `merged:>last_run` (estrito). Com `>=`, 46 PRs já documentados voltariam como delta: 25 do
  `VIctory-7900XTX`, 17 do `VIctory-Belobug-AMD`, 1 do `OpenScarlett`, 1 do `victory-fork`, 1 do `TuxBox` e
  1 do `xcl`.
- **`search/issues` devolve 422 em repositório transferido.** `ivall/xcl` redireciona para `xcl-dev/xcl`; a
  API de conteúdo aceita o nome antigo e a de busca não. A coleta de mesclados passou a usar
  `gh pr list --state merged --search`, que aceita o nome antigo.
- **Sem subagentes.** A ferramenta de subagente não existe neste ambiente; os quatro repositórios foram
  processados em sequência. O limite de 4 por execução foi respeitado.
- **Rate limit**: `gh api rate_limit` marcava 4.996 de 5.000 no início; nenhuma chamada devolveu 403.
- **Aberto acima do cursor**: `victory-fork` teve os 35 catalogados um a um (#1428 a #1468). A coluna
  "Proposta" desses 35 vem do título e do nome da branch, que neste repositório são descritivos ao nível de
  changelog; só o #1468 teve o patch lido.
- Nenhum `_falhas.md`: não houve erro de coleta ou de escrita em repositório.

## Push

- Commit na `public`: `741eab0` — `docs(updates): 2026-11-27 15:00` (7 arquivos, 593 inserções, 80
  remoções: os 4 documentos, `docs/updates/INDEX.md`, `state/watch.json` e `state/last-run.md`).
- `git push origin public:main`: **sucesso** — `6a99faa..741eab0 public -> main`.
- `git switch main && git merge --no-ff public -m "merge updates 2026-11-27 15:00"`: **sucesso**, merge
  `c9cc65d`; `git push private main`: **sucesso** — `52ceb22..c9cc65d main -> main`.
- Fechamento (registro em `state/last-run.md`): commit `81d7c25` na `public`, `git push origin public:main`
  **sucesso** (`741eab0..81d7c25`); merge `a8f4acc` na `main`, `git push private main` **sucesso**
  (`c9cc65d..a8f4acc`).
- Ajuste posterior, depois que a segunda execução desta hora reescreveu `state/watch.json` e
  `state/last-run.md` na árvore: commit na `public` com os cursores corrigidos (`runs` 13, `TuxBox` em
  `90df685ad5` com `commit_cursor`/`head_at_run` e `last_run` 09:37:36Z, `SixNG` com `last_run` 07:02:25Z),
  push `origin public:main`, merge `--no-ff` na `main` e push `private main`; os hashes estão no commit de
  fechamento seguinte.
- Nenhum merge de `main` na `public` em nenhum momento.
