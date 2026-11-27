# Execução de 2026-11-27 15:00

Horário: 2026-11-27 15:00 UTC (12:00 BRT). Executor: rotina horária do acervo `gta6-port-knowledge`.
Estado lido de `state/watch.json` com `baselined: true`, `runs: 12`. Primeira etapa (baseline) não se
aplicou.

## Repositórios com delta (11 de 13)

| Repositório | Commits na janela | PRs mesclados | PRs abertos acima do cursor | Documentado? |
|---|---|---|---|---|
| `brcly/TuxBox` | 47 (compare) | 9 | 0 (cursor 284) | sim — 40 commits, 8 merges |
| `holdmysocks/SixNG` | 24 | 24 | 1 (#472) | sim — 24, 24, 1 |
| `fxceless/victory-fork` | 1 | 0 | 35 (1428-1468) | sim — 1, 0, 35 |
| `uiliannn/VItoriaBr` | 1 | 0 | 0 (cursor 126) | sim — 1, 0, 0 |
| `R0Y00/Xbox2Windows` | 46 | 47 | 12 | não — backlog |
| `ivall/xcl` | 5 | 4 | 50 (piso) | não — backlog |
| `Marvisak/VIctory` | 0 | 0 | 50 (piso) | não — backlog |
| `Belobug/VIctory-Belobug-AMD` | 0 | 0 | 50 (piso) | não — backlog |
| `DexNom/VIctory` | 0 | 0 | 25 | não — backlog |
| `avatarmanz/VIctory-7900XTX` | 0 | 0 | 33 | não — backlog |
| `omonkeyz/xbox-emulator` | 0 | 0 | 1 (#395) | não — backlog |

Sem delta: `californiaAlmonds/OpenScarlett` (1 PR aberto, nenhum acima do cursor 60; 1 merge na busca com
`mergedAt` anterior ao cursor) e `walkthestars/GTA6-Asset-Info` (branch parada em `bad5bc21`).

Critério aplicado, o mesmo registrado no `backlog_note` da execução anterior: repositório com commit ou
merge na janela antes de repositório só com PR aberto; dentro de cada grupo, o `last_run` mais antigo.
Isso deu `VItoriaBr` (26/11 04:15Z), `victory-fork` (26/11 23:06Z), `TuxBox` (27/11 02:54Z) e `SixNG`
(27/11 04:37Z). `xcl` (05:02:20Z) e `Xbox2Windows` (05:02:42Z) ficaram de fora pelo limite de 4 e vão para
o backlog.

## Documentos escritos

4 documentos em `docs/updates/2026-11-27/15/`:

- `TuxBox.md` — 40 commits (de 47), 8 merges (de 9), 0 PRs abertos novos.
- `SixNG.md` — 24 commits, 24 merges, 1 PR aberto.
- `victory-fork.md` — 1 commit, 0 merges, 35 PRs abertos.
- `VItoriaBr.md` — 1 commit, 0 merges, 0 PRs abertos.

Total: 66 commits e 32 merges documentados, mais 36 PRs abertos. Nenhum repositório passou do teto de 40
commits além do `TuxBox`, que foi cortado em 40 exatamente.

## Falhas e desvios

- **`search/issues` devolve 422 para repositório transferido.** `ivall/xcl` redireciona para
  `xcl-dev/xcl`; a API de conteúdo aceita o nome antigo, a busca não
  (`The listed users and repositories cannot be searched`). A contagem de mesclados do `xcl` (4) foi
  medida com `repo:xcl-dev/xcl`. Registrado no campo `note` do estado. Não afetou os quatro documentados.
- **Faixa do `TuxBox` cresceu entre coleta e documentação.** A coleta às 12:36 viu 44 commits novos; o
  `compare` às 12:40 viu 47 (`ahead_by=47`). O teto de 40 foi aplicado sobre o `compare`, e os 7 commits
  restantes ficam com `commit_cursor` em `90df685ad5`. O merge #306 (`07db52f8`, 12:36:16Z) caiu fora do
  teto, e por isso o `last_run` do `TuxBox` avançou só até o `mergedAt` do #305 (09:37:36Z) — não até o
  horário da execução.
- **Instrução de filtro de merge contrariada pelo procedimento.** O pedido desta hora diz "PRs mesclados:
  filtre por numero > last_pr_number"; o procedimento (`docs/automacao/ANALISE_HORARIA.md`, seção "Por
  que PR mesclado não usa número") proíbe isso e manda usar `mergedAt` contra `last_run`. Segui o
  procedimento. Nesta hora os dois critérios dariam resultados diferentes: no `SixNG` o cursor de abertos
  era 460 e os merges foram #447-#470, todos **abaixo** dele.
- **Sem subagentes.** Não há ferramenta de subagente neste ambiente; os quatro repositórios foram
  processados em sequência pelo próprio executor, respeitando o limite de 4.
- **Janelas de 50.** As contagens de PRs abertos de `ivall/xcl`, `Marvisak/VIctory`,
  `Belobug/VIctory-Belobug-AMD`, `DexNom/VIctory` e `avatarmanz/VIctory-7900XTX` vêm de `--limit 50`:
  são piso, não total.
- **Rate limit.** `gh api rate_limit` marcava 4.996 de 5.000 no início e 4.636 restantes no fim da
  documentação. Nenhuma chamada devolveu 403.
- Nenhum `_falhas.md` foi escrito porque nenhum dos quatro repositórios documentados falhou na coleta ou
  na escrita.

## Estado

`updated` = `2026-11-27T15:56:00Z`, `runs` = 13. Cursores avançados: `VItoriaBr` (`15779f81`),
`victory-fork` (`9a9091d1`, aberto até 1468), `TuxBox` (`90df685a` com `commit_cursor` e `head_at_run`
`07db52f8`, `last_run` 09:37:36Z), `SixNG` (`790a3c67`, aberto até 472, `last_run` 07:02:25Z).
`fxceless/victory-fork` e `uiliannn/VItoriaBr` saíram/ficaram fora do backlog; `ivall/xcl`,
`R0Y00/Xbox2Windows`, `Marvisak/VIctory`, `Belobug/VIctory-Belobug-AMD`, `DexNom/VIctory`,
`avatarmanz/VIctory-7900XTX`, `omonkeyz/xbox-emulator` e `brcly/TuxBox` (resto da faixa) ficaram.

## Push

(Pendente de preenchimento depois da publicação.)
