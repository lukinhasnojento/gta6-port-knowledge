# Triagem de 2026-11-27 17:00

Coleta dos 14 repositórios de `state/watch.json`, sem documentação por diff. Cursor de commits: `compare/<last_commit>...<head>`, campo `ahead_by`. Cursor de mesclados: `last_run` comparado com `merged_at`, lido por `pulls?state=closed&sort=updated` (a busca `--search "merged:>="` continua descartada). Janela de PRs abertos: `pulls?state=open&sort=created&desc`, 50 por página.

| repo | commits atrás | PRs abertos | PRs mesclados | sinal | prioridade |
|---|---|---|---|---|---|
| `Marvisak/VIctory` | 0 | 3 (#3522, #3523, #3525) | 0 | SIM | 3 |
| `ivall/xcl` | 0 | 4 (#3023, #3024, #3025, #3026) | 0 | SIM | 3 |
| `R0Y00/Xbox2Windows` | 79 (47 novos, 32 já no doc das 16:00) | 5 (#3063, #3065, #3067, #3068, #3071) | 6 | SIM | 1 |
| `californiaAlmonds/OpenScarlett` | 0 | 1 (#60, abaixo do cursor) | 0 | NAO | — |
| `Belobug/VIctory-Belobug-AMD` | 0 | 50 na janela (piso #551; cursor 456) | 0 | SIM | 3 |
| `DexNom/VIctory` | 0 | 26 (#107 a #134) | 0 | SIM | 2 |
| `uiliannn/VItoriaBr` | 0 | 0 | 0 | NAO | — |
| `avatarmanz/VIctory-7900XTX` | 0 | 37 (#420 a #468) | 0 | SIM | 3 |
| `fxceless/victory-fork` | 0 | 10 (#1469 a #1480) | 0 | SIM | 3 |
| `omonkeyz/xbox-emulator` | 0 | 1 (#395) | 0 | SIM | 3 |
| `brcly/TuxBox` | 14 | 1 (#309) | 2 (#272, #277) | SIM | 1 |
| `holdmysocks/SixNG` | 1 | 1 (#472, abaixo do cursor) | 1 (#474) | SIM | 2 |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 | 0 | NAO | — |
| `stellarr-studio/kelvaroq` | 0 | 0 | 0 | NAO | — |

## Sinal

- **SIM** — algum commit ou PR da janela casa com defeito do `catalogo/defeitos.csv`, traz termo de correção/regressão/reversão, ou toca área listada na tabela de pistas. A tabela de pistas citada pelo procedimento **não existe** em `docs/automacao/ANALISE_HORARIA.md`; na ausência dela, as áreas foram tomadas das seções D de `catalogo/defeitos.md` (tradutor DXIL/`gfx2hlsl`, bind de descritor e heap, memória e XMem, textura e tiling, present e save). O desvio está registrado em `state/last-run.md`.
- **NAO** — nenhum commit, PR aberto ou PR mesclado na janela.

## Prioridade

1. defeito do catálogo ou correção/reversão medida na janela — entrou no deep dive.
2. correção/merge com diffs substantivos — entrou no deep dive.
3. sinal só por PR aberto em área das pistas — fica na fila, cursor parado.
4. nada acima e atraso acima de 200 commits — não houve caso nesta hora.

Documentados nesta hora (4): `R0Y00/Xbox2Windows`, `brcly/TuxBox`, `holdmysocks/SixNG`, `DexNom/VIctory`.

## Notas por repositório

**`R0Y00/Xbox2Windows`** — 79 por `compare` desde `f5204690`, mas 32 deles (`40365770` a `999e17cd`) são os commits da linha `cpp-migration` já documentados às 16:00: eles não são ancestrais de `f5204690` e reaparecem no `compare` até o cursor passar pelo merge `91cbedd6` (merge de #3039 em `main`, 10:13). Os 47 restantes são novos. Dois deles corrigem defeito introduzido na própria janela: `f6cdee10` (#3020) devolve `D[31:16]` ao binário16 direto, contra o contrato de #2964 documentado às 16:00, e `1c879506` (#3040) conserta a escala do descritor BVH de `0d228580` (#2922). Também tocam catálogo: `f17a08fd` (#3024, área de D-01) e `98007de5` (#2899, área de D-19).

**`brcly/TuxBox`** — 14 por `compare` desde `76b7bafe`, nenhum deles no documento das 16:00. Doze vêm da branch `work/mesh-validation-20261126` (commits datados de 26/11 20:53 a 23:40) e entraram agora pelo merge `497c561a` (#272); `6ea8f39a` entrou pelo merge `6ca0d4ac` (#277). `18736af5` (13:55:49) e `76d1abe0` (14:07:01) são diretos na `main`.

**`holdmysocks/SixNG`** — um commit, o merge `2d9c230d` de #474: `first = me.first()` e `base = 0` no `draw_indexed` da máscara de blur, contra o deslocamento somado duas vezes desde #470.

**`DexNom/VIctory`** — sem commit e sem merge; 26 PRs abertos acima do cursor 106, todos datados de 26/11 20:58 a 27/11 13:25. Entrou no deep dive por sinal de correção em área de memória/descritor/textura (#107, #111, #112, #127, #128, #132, #133), a fila mais antiga ainda não lida entre os repositórios só de PR.

**`Belobug/VIctory-Belobug-AMD`** — a janela de 50 abertos começa em #551, acima do cursor 456, como já registrado na pendência estrutural 2: o cursor não prova cobertura dos PRs antigos. Sem commit e sem merge desde `last_run` (16:52:39).

**`avatarmanz/VIctory-7900XTX` / `fxceless/victory-fork` / `omonkeyz/xbox-emulator` / `ivall/xcl` / `Marvisak/VIctory`** — só PRs abertos; cursores de commit e de merge ficam parados. `xcl` tem `last_run` de 08:50 e nenhum merge novo em 8 h; `VIctory` idem desde 13:20.

**Sem delta** — `californiaAlmonds/OpenScarlett`, `uiliannn/VItoriaBr`, `walkthestars/GTA6-Asset-Info`, `stellarr-studio/kelvaroq`: head igual ao cursor, nenhum PR na janela.

## Sanidade do estado (passo 1)

- 14 entradas em `repos`, igual ao lido no início; nenhuma foi removida.
- `default_branch` reconferido por `gh api repos/<o>/<r> --jq .default_branch` nos 14: todos iguais ao gravado (`Belobug/VIctory-Belobug-AMD` = `a-publier`, `holdmysocks/SixNG` = `master`, os outros 12 = `main`).
- Nenhum `last_commit` nulo; `stellarr-studio/kelvaroq` tem `last_pr_number` nulo e nenhum PR aberto, e o head é o mesmo do cursor.
- Nenhum `compare` devolveu `status="diverged"` nesta coleta.
- Rate limit do GitHub no início: 4.919 de 5.000 restantes.
