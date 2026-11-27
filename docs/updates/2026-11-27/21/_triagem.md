# Triagem de 2026-11-27 21:00

Coleta entre 21:05 e 21:20 UTC real — ~18:05-18:20 no relógio do GitHub, que segue cerca de 3 h
atrás e é o relógio em que `last_run` é gravado. Cursor de commits de cada repositório = `last_commit`
do `state/watch.json` lido no início (`runs: 18`). 14 entradas em `repos` no início e no fim; nenhuma
removida, nenhuma acrescentada.

**Sanidade:** `default_branch` reconferido nos 12 repositórios alcançáveis por
`gh api repos/<o>/<r> --jq .default_branch`: idêntico ao gravado em todos, inclusive `master`
(`holdmysocks/SixNG`). Dois repositórios **não respondem mais**: `Belobug/VIctory-Belobug-AMD` e
`avatarmanz/VIctory-7900XTX` devolvem HTTP 404 tanto em `repos/<o>/<r>` quanto em `users/<o>` — a conta
inteira desapareceu da API (detalhe e consequência em `_falhas.md`). Nenhum `last_commit` nulo: não
houve baseline nesta execução. Nenhum `compare` devolveu `diverged`.

| repo | commits atrás | PRs abertos | PRs mesclados | sinal | prioridade |
|---|---|---|---|---|---|
| `R0Y00/Xbox2Windows` | 5 | 3 (113 no total) | 5 | SIM — quatro lowerings novos no caminho direto de compute (`v_dot8_*`, `v_sat_pk_u8_i16`, `v_frexp_*` f32 e f16), cada um com controle de compilação e teste de hardware; critério 2 | 2 — **deep dive** |
| `brcly/TuxBox` | 4 | 8 (11 no total) | 1 | SIM — #312 mesclado mede root SRV de compute depois das tabelas, e `89fb2517` troca o caminho de compilação de shader por um helper que pula `correlated-propagation`; critério 2 | 2 — **deep dive** |
| `holdmysocks/SixNG` | 3 | 2 (2 no total) | 3 | SIM — `fix(metal4)` de empréstimo dos bytes de staging, ferramenta de bake de cache de modelo e chaves de mandíbula no viewer; critério 2 | 2 — **deep dive** |
| `fxceless/victory-fork` | 1 | 3 (947 no total) | 0 | SIM — um único commit direto na `main` com 19 arquivos: contador de threads por grupo (buracos na água), gate de slots dinâmicos ("no dropped draws") e `VICTORY_WATER_LIGHT=1` ligado por padrão; critério 2 | 2 — **deep dive** |
| `Marvisak/VIctory` | 0 | 38 (≥, 1.675 no total) | 0 | SIM — 38 PRs acima de #3521, lote de `Shaders:`/`GPU:` com correção declarada (raw view de 16 bytes, RGB9E5 placed views, `v_bfm_b32` em um `Bfi`) | 3 — zero commit |
| `ivall/xcl` | 0 | 9 | 0 | SIM — #3052 janela de memória de mesh em GPU VA 0, #3047 programa de compute dividido, #3049 contagem de heap | 3 — zero commit |
| `DexNom/VIctory` | 0 | 2 (122 no total) | 0 | SIM fraco — #136 diagnóstico de copy-back, #135 slices de scratch em buffer→textura | 3 — zero commit |
| `uiliannn/VItoriaBr` | 0 | 1 (75 no total) | 0 | SIM fraco — #132 validador DXIL em builds VALU estritos, só teste | 3 — zero commit |
| `omonkeyz/xbox-emulator` | 0 | 1 (25 no total) | 0 | SIM — #395 "Fix the PE relocation block validation" | 3 — zero commit |
| `californiaAlmonds/OpenScarlett` | 0 | 0 (1 no total) | 0 | NAO | — |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 (0) | 0 | NAO | — |
| `stellarr-studio/kelvaroq` | 0 | 0 (0) | 0 | NAO | — |
| `Belobug/VIctory-Belobug-AMD` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO — conta 404 | **bloqueado** |
| `avatarmanz/VIctory-7900XTX` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO — conta 404 | **bloqueado** |

"PRs abertos" = PRs com número acima de `last_pr_number` na janela dos 100 mais recentes por data de
criação (é um piso quando o repositório tem mais de 100 abertos). Totais desta coleta: `VIctory` 1.675,
`victory-fork` 947, `Xbox2Windows` 113, `DexNom` 122, `VItoriaBr` 75, `xbox-emulator` 25, `TuxBox` 11,
`SixNG` 2, `OpenScarlett` 1, `GTA6-Asset-Info` 0, `kelvaroq` 0. `ivall/xcl` continua sem total: a busca
`search/issues` devolve HTTP 422 para o repositório transferido (terceira hora seguida).

"PRs mesclados" = `merged_at` maior que `last_run`, colhidos por
`repos/<o>/<r>/pulls?state=closed&sort=updated&direction=desc` paginando até o `updated_at` mais antigo
cair abaixo do `last_run`, e reconferidos ao fim da coleta (contagens idênticas: 5, 1, 3, 0).

**Contagem de commits x de merges:** nesta hora os dois cursores batem nos quatro documentados — as
5 entradas do `Xbox2Windows` e as 3 do `SixNG` são squash (`parents` = 1, `committer` = GitHub) e o
`merge_commit_sha` de cada PR é o próprio commit; o `TuxBox` tem 1 merge de 2 pais (`8c082e65`) e as
duas entradas de branch (`f5233880`, `49fad5f7`) que ele traz. `victory-fork` tem 1 commit direto e
0 merges.

## Desempate das 4 vagas

O critério 1 (commit ou PR que casa com defeito do catálogo) **não** selecionou ninguém: nenhum dos 13
commits da janela corrige uma entrada `D-xx` do próprio repositório. O critério 2 (commit com sinal de
correção/regressão/reversão) selecionou os quatro repositórios que tiveram commit — `Xbox2Windows`,
`TuxBox`, `SixNG` e `victory-fork` — e eles ocupam as quatro vagas. É o mesmo desempate da hora
anterior, com a diferença de que `xcl` não teve commit nesta janela e `victory-fork` entrou no lugar
dele.

Custo registrado: `Marvisak` (38 PRs novos acima do cursor, 1.675 abertos), `Belobug` (374 na última
leitura, hoje inacessível), `avatarmanz` (240 na última leitura, hoje inacessível) e `xcl` (9) seguem
sem documentação porque o cursor de commits deles não anda. `victory-fork` saiu do backlog.

## Observação sobre a "tabela de pistas"

O passo 2 do procedimento manda usar "arquivo das áreas da tabela de pistas", mas essa tabela não
existe em `docs/automacao/ANALISE_HORARIA.md` (pendência registrada desde as 16:00). As áreas usadas
para o sinal de PR aberto vieram das seções `D` de `catalogo/defeitos.md`, e o sinal dessas linhas é
por título e corpo de PR — não por patch. O deep dive foi decidido pelo diff.

## Falhas

Nenhuma chamada devolveu 403. `gh api rate_limit` marcava 4.783 restantes de 5.000 no início da coleta;
ao fim (incluindo os diffs dos 13 commits e a reconferência de heads e merges), a janela já tinha
reiniciado — 4.890 restantes, 110 usados —, então o gasto total da execução não é legível pelo
contador. Nenhuma chamada foi repetida em laço. Dois repositórios falharam por 404 (conta removida) e
estão em `_falhas.md`; os outros 12 foram triados por inteiro.
