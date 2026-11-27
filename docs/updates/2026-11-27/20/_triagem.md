# Triagem de 2026-11-27 20:00

Coleta entre 20:07 e 20:21 UTC real (17:07-17:21 no relógio do GitHub, que segue 3 h atrás; é esse
relógio que grava `last_run`). Cursor de commits de cada repositório = `last_commit` do
`state/watch.json` lido no início (`runs: 17`). 14 entradas em `repos` no início e no fim; nenhuma
removida. `default_branch` reconferido nos 14 por `gh api repos/<o>/<r> --jq .default_branch`: idêntico
ao gravado em todos, inclusive `a-publier` (Belobug) e `master` (SixNG). Nenhum `last_commit` nulo: não
houve baseline nesta execução. Nenhum `compare` devolveu `diverged`.

"PRs abertos" = PRs com número acima de `last_pr_number` na janela dos 100 mais recentes por data de
criação. Onde o repositório tem mais de 100 abertos, o número é um piso (marcado ≥). Totais de abertos
nesta coleta: `VIctory` 1.663, `victory-fork` 946, `VIctory-Belobug-AMD` 374, `VIctory-7900XTX` 240,
`Xbox2Windows` 115. "PRs mesclados" = `merged_at` maior que `last_run`, colhidos por
`pulls?state=closed&sort=updated&direction=desc` paginando até o `updated_at` mais antigo cair abaixo do
`last_run`, e reconferidos ao fim da coleta.

| repo | commits atrás | PRs abertos | PRs mesclados | sinal | prioridade |
|---|---|---|---|---|---|
| `R0Y00/Xbox2Windows` | 37 | 10 | 3 | SIM — série #2911 de page runs do XMem (retry de fault por stamp, join com teto de 64 páginas, split antes do map), #3114 `s_quadmask` no lowering direto; critério 2 e possível variante de D-02 | 2 — **deep dive** |
| `brcly/TuxBox` | 6 | 1 | 1 | SIM — `tb_float_relation` e `tb_f16_cvt` reescritos sem branch, com as 48 linhas de `host_profiles.txt` refixadas; critério 2 | 2 — **deep dive** |
| `holdmysocks/SixNG` | 4 | 1 (#476) | 4 | SIM — picking de livery contra a malha posada e junção de draws por material; critério 2 | 2 — **deep dive** |
| `ivall/xcl` | 2 | 3 | 1 | SIM — #880 corrige o tamanho de wave usado pela estatística e pelo dump de `xcl shaders`; critério 2 | 2 — **deep dive** |
| `Marvisak/VIctory` | 0 | 26 (≥, 1.663 no total) | 0 | SIM — lote de `fix(...)` em GPU/shader: raw view alinhada que removia o device, RGB9E5 placed views, CBV tipado, `v_bfm_b32` em um `Bfi` | 3 — zero commit |
| `Belobug/VIctory-Belobug-AMD` | 0 | ≥100 (374 no total) | 0 | SIM — 100 abertos na janela, quase todos `fix(...)` em GPU shader e HLE | 3 — zero commit, maior passivo |
| `avatarmanz/VIctory-7900XTX` | 0 | 43 (240 no total) | 0 | SIM — "14 bug fixes" e "five XGameSave fixes" reempacotados, trims de pressão de memória, PAGEMAP_SCAN sob Wine | 3 — zero commit |
| `fxceless/victory-fork` | 0 | 2 (946 no total) | 0 | SIM fraco — #1489 restaura shaders indiretos e de cópia de card sob Proton, #1491 convolução de áudio | 3 — zero commit |
| `omonkeyz/xbox-emulator` | 0 | 1 (#395) | 0 | SIM — "Fix the PE relocation block validation" | 3 — zero commit |
| `DexNom/VIctory` | 0 | 1 (#135) | 0 | SIM fraco — pool de upload para buffer→textura, sem termo de correção | 3 — zero commit |
| `uiliannn/VItoriaBr` | 0 | 1 (#132) | 0 | SIM fraco — validador DXIL em builds VALU estritos, só teste | 3 — zero commit |
| `californiaAlmonds/OpenScarlett` | 0 | 0 | 0 | NAO | — |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 | 0 | NAO | — |
| `stellarr-studio/kelvaroq` | 0 | 0 | 0 | NAO | — |

## Desempate das 4 vagas

O procedimento ordena os critérios e o critério 2 (commit com sinal de correção/regressão/reversão)
vem antes do 3 (PR aberto em área de defeito). Nesta hora há quatro repositórios com commit na janela e
sinal de correção — `Xbox2Windows`, `TuxBox`, `xcl` e `SixNG` —, e eles ocupam as quatro vagas.
`Marvisak`, `Belobug` e `avatarmanz` seguem com sinal apenas de PR aberto, sem commit desde o cursor.

Esse desempate tem custo e ele fica registrado: `Belobug` acumula 374 PRs abertos, `Marvisak` 1.663 e
`avatarmanz` 240, e nenhum deles foi documentado nas últimas quatro execuções. Como o cursor deles não
anda, o passivo não se perde, mas também não diminui. A próxima hora sem commit sinalizado deve pegar
`Belobug` primeiro, pelo tamanho do passivo, depois `avatarmanz`, depois `Marvisak`.

## Observação sobre a "tabela de pistas"

O passo 2 do procedimento manda usar "arquivo das áreas da tabela de pistas", mas essa tabela não existe
em `docs/automacao/ANALISE_HORARIA.md` (pendência registrada desde as 16:00). As áreas foram tomadas das
seções D de `catalogo/defeitos.md`, e o sinal dessas linhas sem commit é por título e corpo de PR — não
por patch. O deep dive, esse sim, foi decidido pelo diff.

## Observação sobre PR mesclado fora da branch default

`R0Y00/Xbox2Windows` #3130 foi mesclado às 16:22:59 com base `codex/native-graphics-integration`. O
`merge_commit_sha` `b80513fd` não está entre os 37 commits da faixa do `compare` e não é ancestral do
head `63a30dc1`. Ou seja: o cursor de mesclados (por `mergedAt`) enxerga merges que não chegam à
`main`, e o cursor de commits não. O mesmo padrão já vale para `ivall/xcl` (`nvidia-dev`) e aparece em
três dos dez PRs abertos do `Xbox2Windows` (`codex/native-shader-modules`, `codex/native-graphics-integration`).
Contagens de merge e de commit não vão bater enquanto isso existir; a divergência está registrada em
cada documento.

## Falhas

Nenhuma chamada devolveu 403. `gh api rate_limit` marcava 4.999 de 5.000 no início e 4.794 depois da
coleta de commits e PRs dos quatro documentados. A busca `search/issues` funcionou nos cinco repositórios
onde foi usada para contar abertos; a contagem total de `xcl` não foi pedida (repositório transferido,
HTTP 422 na execução das 19:00), então esse repositório aparece sem total. Nenhum repositório falhou.

**Coleta que mudou durante a execução:** `holdmysocks/SixNG` recebeu dois merges entre a primeira
coleta e a reconferência (#477 às 17:10:02, #479 às 17:14:27 no relógio do GitHub). O `compare` foi
refeito de `2d9c230d` até o head novo `67bcc0ec` e os dois commits entraram na documentação; a
reconferência de mesclados também foi refeita, e fechou em 4. `Xbox2Windows`, `TuxBox` e `xcl` tiveram
head e lista de merges estáveis entre a coleta e a reconferência.
