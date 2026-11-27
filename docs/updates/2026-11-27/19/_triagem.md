# Triagem de 2026-11-27 19:00

Coleta entre 19:07 e 19:40 UTC real (16:07–16:40 no relógio do GitHub, que segue 3 h atrás; é esse
relógio que grava `last_run`). Cursor de commits de cada repositório = `last_commit` do
`state/watch.json` lido no início (`runs: 16`). 14 entradas em `repos` no início e no fim; nenhuma
removida. `default_branch` reconferido nos 14 por `gh api repos/<o>/<r> --jq .default_branch`:
idêntico ao gravado em todos, inclusive `a-publier` (Belobug) e `master` (SixNG). Nenhum
`last_commit` nulo: não houve baseline nesta execução. Nenhum `compare` devolveu `diverged`.

"PRs abertos" = PRs com número acima de `last_pr_number` na janela dos 100 mais recentes por data de
criação. Onde o repositório tem mais de 100 abertos, o número é um piso (marcado ≥). "PRs mesclados" =
`merged_at` maior que `last_run`, colhidos por `pulls?state=closed&sort=updated` paginando até o
`updated_at` mais antigo cair abaixo de `last_run`.

| repo | commits atrás | PRs abertos | PRs mesclados | sinal | prioridade |
|---|---|---|---|---|---|
| `R0Y00/Xbox2Windows` | 16 | 10 | 16 | SIM — 15 commits fecham recusas do lowering direto ("Fixes #30xx"), #2919 persiste receitas de PSO | 2 — **deep dive** |
| `brcly/TuxBox` | 12 | 0 | 3 | SIM — #310 retenta recusa de stride IDXEN, #239 deduplica compilação DXIL, #311 splash | 2 — **deep dive** |
| `fxceless/victory-fork` | 6 | 3 | 0 | SIM — correção de stall de lock, orçamento de páginas reciclado, ray tracing ligado por padrão | 2 — **deep dive** |
| `ivall/xcl` | 2 | 6 | 1 | SIM — #3033–#3035 em `list_x`/`cpu_writes`: ordem escrita→espera na lista (área de D-20/D-21) | 3 — **deep dive** |
| `holdmysocks/SixNG` | 1 | 1 | 1 | SIM fraco — `bab14f08` agrupa batches de célula (render), sem termo de correção; #476 `fix(metal4)` | 3 — delta 1 |
| `Belobug/VIctory-Belobug-AMD` | 0 | ≥100 (357 abertos no total) | 0 | SIM — lote de ports e `fix(...)` em GPU/shader/HLE | 3 — zero commit |
| `avatarmanz/VIctory-7900XTX` | 0 | 39 (235 no total) | 0 | SIM — "14 bug fixes", "five XGameSave fixes", portas de GPU | 3 — zero commit |
| `Marvisak/VIctory` | 0 | 19 (1.656 no total) | 0 | SIM — GPU/shader: quadmask, frexp, RGB9E5, BVH skinning | 3 — zero commit |
| `omonkeyz/xbox-emulator` | 0 | 1 (#395) | 0 | SIM — "Fix the PE relocation block validation" | 3 — zero commit, 1 PR |
| `DexNom/VIctory` | 0 | 1 (#135) | 0 | SIM fraco — área GPU (cópias buffer→textura), sem termo de correção | 3 — zero commit |
| `uiliannn/VItoriaBr` | 0 | 1 (#132) | 0 | SIM fraco — testes de carregamento do validador DXIL em builds VALU estritos | 3 — zero commit |
| `californiaAlmonds/OpenScarlett` | 0 | 0 | 0 | NAO | — |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 | 0 | NAO | — |
| `stellarr-studio/kelvaroq` | 0 | 0 | 0 | NAO | — |

## Desempate da 4ª vaga

`Belobug`, `avatarmanz`, `Marvisak` e `xcl` empatam no critério 3 (PR aberto em área de defeito do
catálogo), e `Belobug` tem o maior passivo (≥100 PRs acima do cursor). O critério 2 do procedimento
fala em **commit** com sinal de correção/regressão/reversão; os três primeiros têm zero commit na
janela. `ivall/xcl` tem 2 commits, 1 merge e 6 PRs abertos que atacam a ordem escrita→espera da
lista — o mesmo eixo de XCL D-20/D-21. Desempate por delta real, como na execução das 18:00.
`Belobug` e `avatarmanz` seguem no backlog, com o passivo medido.

## Observação sobre a "tabela de pistas"

O passo 2 do procedimento manda usar "arquivo das áreas da tabela de pistas", mas essa tabela não
existe em `docs/automacao/ANALISE_HORARIA.md` (pendência registrada desde as 16:00). As áreas foram
tomadas das seções D de `catalogo/defeitos.md`, e o sinal dessas linhas é por título/corpo de PR —
não por patch. O deep dive, esse sim, foi decidido pelo diff.

## Falhas

Nenhuma chamada devolveu 403. `gh api rate_limit` marcava 4.985 de 5.000 no início. A busca
`search/issues` recusou `ivall/xcl` com HTTP 422 (repositório transferido, mesmo motivo que já
obriga a API REST em vez de `gh pr list --search`); a contagem total desse repositório ficou como
"≥100 na janela", sem número global.
