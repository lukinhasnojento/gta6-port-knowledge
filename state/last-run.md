# Execução de 2026-11-27 05:00

## Data e hora

Coleta iniciada em 2026-11-27T05:03:17Z (UTC), concluída em 2026-11-27T05:15:04Z. Hora local do
operador: 02:03–02:15 (E. South America Standard Time, UTC-3). Janela documentada: de
`updated` anterior (2026-11-27T04:18:02Z) até o cursor de cada repositório.

## Repositórios com delta

Detectados com delta em 10 dos 13: `Marvisak/VIctory`, `ivall/xcl`, `R0Y00/Xbox2Windows`,
`Belobug/VIctory-Belobug-AMD`, `DexNom/VIctory`, `avatarmanz/VIctory-7900XTX`, `fxceless/victory-fork`,
`omonkeyz/xbox-emulator`, `brcly/TuxBox` e `holdmysocks/SixNG`. Sem delta: `californiaAlmonds/OpenScarlett`,
`uiliannn/VItoriaBr` e `walkthestars/GTA6-Asset-Info`.

Processados (limite de 4 por execução — os quatro com commit ou merge na janela, mais o PR-aberto de
`last_run` mais antigo):

| Repositório | Commits | Mesclados | Abertos documentados | Abertos detectados acima do cursor |
|---|---|---|---|---|
| `R0Y00/Xbox2Windows` | 12 | 16 (12 na `main`, 4 em `cpp-migration`) | 5 | 5 |
| `ivall/xcl` | 6 | 3 (#2924 e #2844 na `main`, #2927 em `nvidia-dev`) | 20 | 180 |
| `holdmysocks/SixNG` | 2 | 2 | 10 | 10 |
| `Marvisak/VIctory` | 0 | 0 | 20 | 380 |

Os outros seis ficaram em `backlog` no estado, com a contagem de cada um.

## Documentos escritos

4 documentos, em `docs/updates/2026-11-27/05/`: `Xbox2Windows.md`, `xcl.md`, `SixNG.md` e `VIctory.md`.
Nenhum doc acima do teto de 40 commits por repositório (o maior foi 12). `_falhas.md` não foi escrito
porque não houve falha de coleta ou de escrita em nenhum repositório.

## Falhas e desvios

- **Perda de merge na listagem padrão**: `gh pr list --state merged --limit 50` devolveu 15 dos 17 PRs
  mesclados do `Xbox2Windows` na janela e perdeu o **#2661** (criado há dias, mesclado às 05:02:20Z), porque
  a listagem corta por data de criação. A coleta passou a usar
  `search/issues?q=repo:<owner>/<repo>+type:pr+is:merged+merged:>=<last_run>`, que devolveu 17 (incluindo
  o #2947, exatamente no cursor, já documentado às 03:00). Fica registrado em `state/watch.json`.
- **Filtro de mesclados**: a instrução desta hora pedia filtrar por `numero > last_pr_number`; o
  procedimento (`docs/automacao/ANALISE_HORARIA.md`, seção "Por que PR mesclado não usa número") proíbe
  isso. Segui o procedimento: `mergedAt > last_run`.
- **Sem subagentes**: a ferramenta de subagente não existe neste ambiente. Os quatro repositórios foram
  processados em sequência pelo próprio executor; o limite de 4 repositórios por execução foi respeitado.
- **Rate limit**: `gh api rate_limit` marcava 4.702 de 5.000 restantes no início; nenhuma chamada devolveu
  403.
- **Janelas de 50**: `Belobug/VIctory-Belobug-AMD`, `DexNom/VIctory`, `avatarmanz/VIctory-7900XTX`,
  `fxceless/victory-fork`, `omonkeyz/xbox-emulator` e `brcly/TuxBox` foram medidos com `--limit 50`, então
  as contagens no backlog são piso. `VIctory` e `xcl` foram medidos com `--limit 500`.

## Push

- Commit na `public`: `2125552` — `docs(updates): 2026-11-27 05:00` (7 arquivos, 419 inserções, 88
  remoções: os 4 documentos novos, `docs/updates/INDEX.md`, `state/watch.json`, `state/last-run.md`).
- `git push origin public:main`: **sucesso** — `7b619b8..2125552 public -> main`.
- `git switch main && git merge --no-ff public -m "merge updates 2026-11-27 05:00"` e
  `git push private main`: **sucesso** — merge `4b86d08`, `e020317..4b86d08 main -> main`. Nenhum merge
  de `main` na `public` foi feito.
- Estado: o commit de documentação é `2125552` em `origin/main` (branch `public`); o merge correspondente
  é `4b86d08` em `private/main`; `e020317..4b86d08 main -> main`. Os commits de fechamento desta hora, que
  só registram estes hashes em `state/last-run.md`, vêm depois dos dois. Árvore limpa, sem merge de `main`
  na `public`.
