# Triagem de 2026-11-27 22:00

Coleta entre 22:05 e 22:35 UTC real — ~19:05–19:35 no relógio do GitHub, que segue cerca de 3 h
atrás e é o relógio em que `last_run` é gravado. Cursor de commits de cada repositório = `last_commit`
do `state/watch.json` lido no início (`runs: 19`). 14 entradas em `repos` no início e no fim; nenhuma
removida, nenhuma acrescentada.

**Sanidade:** `default_branch` reconferido nos 12 repositórios alcançáveis por
`gh api repos/<o>/<r> --jq .default_branch`: idêntico ao gravado em todos, inclusive `master`
(`holdmysocks/SixNG`) e `a-publier` (`Belobug`, inacessível). Dois repositórios continuam **sem
resposta**: `Belobug/VIctory-Belobug-AMD` e `avatarmanz/VIctory-7900XTX` devolvem HTTP 404 em
`repos/<o>/<r>` (segunda hora seguida) — detalhe em `_falhas.md`. Nenhum `last_commit` nulo: não houve
baseline nesta execução. Nenhum `compare` devolveu `diverged`.

| repo | commits atrás | PRs abertos | PRs mesclados | sinal | prioridade |
|---|---|---|---|---|---|
| `brcly/TuxBox` | 25 | 2 (6 no total) | 7 | SIM — 17 commits de ramo em `xshader` (descritores, eleição de lane, half-wave, contagem de chaves) e retenção de cache de shader; critério 2 | 1 — **deep dive** |
| `holdmysocks/SixNG` | 8 | 8 (8 no total) | 8 | SIM — 8 squash de features/correções no viewer e no export (culling por mesh, leitura de livery fora da thread de UI, perfil de subsuperfície); critério 2 fraco | 1 — **deep dive** |
| `R0Y00/Xbox2Windows` | 5 | 10 (≥100 no total) | 5 | SIM — `pe_run_package` carrega provedor inválido sem diálogo, contadores `raw=`/`completed=` no DirectStorage, harness de GPU com um device só; critério 2 | 1 — **deep dive** |
| `fxceless/victory-fork` | 1 | 0 (≥100 no total) | 0 | SIM — commit direto que corrige as listas de tiles da luz da água em host de 32 lanes (blocos de água escura); critério 2 | 1 — **deep dive** |
| `Marvisak/VIctory` | 0 | 55 (≥100 no total) | 0 | SIM — 55 PRs acima de #3.521, lote `Shaders:`/`GPU:` (loads de memória do título via UAV, quad reads, cache key com opções de tradução); critério 3 | 3 — zero commit |
| `ivall/xcl` | 0 | 20 (≥100 no total) | 0 | SIM — bibliotecas nativas de pipeline (10 PRs de `bmjask`), janela de memória em VA 0, readback esparso obsoleto; critério 3 | 3 — zero commit |
| `DexNom/VIctory` | 0 | 2 (≥100 no total) | 0 | SIM fraco — #136 sondas de diagnóstico de copy-back, #135 slices de scratch em buffer→textura | 3 — zero commit |
| `omonkeyz/xbox-emulator` | 0 | 1 (25 no total) | 0 | SIM — #395 "Fix the PE relocation block validation" | 3 — zero commit |
| `uiliannn/VItoriaBr` | 0 | 1 (75 no total) | 0 | SIM fraco — #132 validador DXIL em builds VALU estritos, só teste | 3 — zero commit |
| `californiaAlmonds/OpenScarlett` | 0 | 0 (1 no total) | 0 | NAO | — |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 (0) | 0 | NAO | — |
| `stellarr-studio/kelvaroq` | 0 | 0 (0) | 0 | NAO | — |
| `Belobug/VIctory-Belobug-AMD` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO — conta 404 | **bloqueado** |
| `avatarmanz/VIctory-7900XTX` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO — conta 404 | **bloqueado** |

"PRs abertos" = PRs com número acima de `last_pr_number` na janela dos 100 mais recentes por data de
criação (é um piso quando o repositório tem mais de 100 abertos). Totais exatos desta coleta:
`TuxBox` 6, `SixNG` 8, `VItoriaBr` 75, `xbox-emulator` 25, `OpenScarlett` 1, `GTA6-Asset-Info` 0,
`kelvaroq` 0. Os demais bateram no teto de 100 da listagem: `VIctory`, `xcl`, `Xbox2Windows`,
`victory-fork`, `DexNom`.

"PRs mesclados" = `merged_at` maior que `last_run`, colhidos por
`repos/<o>/<r>/pulls?state=closed&sort=updated&direction=desc` (até cinco páginas) e filtrados por
`merged_at`. Contagens: `TuxBox` 7, `SixNG` 8, `Xbox2Windows` 5, `victory-fork` 0, e 0 nos outros oito
alcançáveis.

**Contagem de commits x de merges:** nesta hora as duas contagens batem nos quatro documentados. No
`SixNG` e no `Xbox2Windows` os 8 e 5 merges são squash (`parents` = 1) e o `merge_commit_sha` de cada PR
é o próprio commit. No `TuxBox`, os 25 commits da janela são 17 commits de ramo + 7 merges de PR (2 pais)
+ 1 merge de `main` no ramo do #319 (`dc92a00e`, 2 pais, 51 arquivos) — este último **traz de volta**
seis commits de 19:00–21:00 já documentados (`f5233880`, `49fad5f7`, `e3cf2096`, `7953f0a5`, `8c082e65`,
`89fb2517`). `victory-fork` tem 1 commit direto e 0 merges.

## Desempate das 4 vagas

O critério 1 (commit ou PR que casa com defeito do catálogo) **não** selecionou ninguém: nenhum dos 39
commits da janela corrige uma entrada `D-xx` do próprio repositório — o mais próximo é o `TuxBox`, cujos
commits de descritor e de eleição ficam na vizinhança de D-22/D-28 sem tocar o defeito (ver o documento
do repositório). O critério 2 (commit com sinal de correção/regressão/reversão) selecionou os quatro
repositórios que tiveram commit — `TuxBox` (25), `SixNG` (8), `Xbox2Windows` (5) e `victory-fork` (1) —
que ocupam as quatro vagas. `SixNG` é o mais fraco dos quatro: cinco dos oito commits são features, e só
#485 (culling) e #488 (leitura fora da thread de UI) têm forma de correção.

Custo registrado: `Marvisak` (55 PRs novos acima do cursor), `xcl` (20), `DexNom` (2), `VItoriaBr` (1) e
`xbox-emulator` (1) seguem sem documentação porque o cursor de commits deles não anda. `Belobug` e
`avatarmanz` seguem bloqueados por 404.

## Observação sobre a "tabela de pistas"

O passo 2 do procedimento manda usar "arquivo das áreas da tabela de pistas", mas essa tabela não existe
em `docs/automacao/ANALISE_HORARIA.md` (pendência registrada desde as 16:00). As áreas usadas para o
sinal de PR aberto vieram das seções `D` de `catalogo/defeitos.md`, e o sinal dessas linhas é por título
e corpo de PR — não por patch. O deep dive foi decidido pelo diff.

## Falhas

Nenhuma chamada devolveu 403. `gh api rate_limit` marcava 4.889 restantes de 5.000 no início; a coleta
usou 39 leituras de commit, 12 listagens de PRs abertos, 12 varreduras de PRs fechados (até cinco páginas
cada), 12 `default_branch`, 12 heads, 12 `compare` e 8 `compare` de merge. Duas contas continuam em 404
(`Belobug`, `avatarmanz`) e estão em `_falhas.md`; os outros 12 foram triados por inteiro.
