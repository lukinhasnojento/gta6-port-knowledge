# Triagem de 2026-11-28 12:00

Coleta de 2026-11-28T12:13–12:25Z (UTC real). `last_run` de cada repositório é o cursor de PRs
mesclados gravado pelo ciclo anterior, no relógio do GitHub — que nesta coleta está **alinhado ao UTC
real** (o commit `2b2277a5` do `SixNG`, de 12:16:23Z, apareceu durante a coleta às 12:19Z), ao
contrário das horas anteriores, em que os timestamps vinham cerca de 3 h atrás.

| repo | commits atras | PRs abertos | PRs mesclados | sinal | prioridade |
|---|---|---|---|---|---|
| `ivall/xcl` | 376 | 36 acima de #3075 (100 lidos, máx. #3125) | 48 (29 em `nvidia-dev`) | SIM (critério 2 e 4: 376 > 200, série de pipeline library) | **1 — deep dive** |
| `brcly/TuxBox` | 169 | 11 acima de #339 (13 abertos, máx. #394) | 50 | SIM (critério 2: `fix/`, aceite de recusa, ballot no lugar de `WaveActiveMin`) | **2 — deep dive** |
| `R0Y00/Xbox2Windows` | 121 | 39 acima de #3215 (100 lidos, máx. #3352) | 106 (2 em `codex/native-shader-modules`) | SIM (critério 2: leva de lowerings e correções de runtime) | **3 — deep dive** |
| `holdmysocks/SixNG` | 33 (36 na lista de commits) | 5 acima de #510 (7 abertos, máx. #541) | 34 | SIM (critério 2: região do detalhe de pele, mãos do cabelo, `BAKE_RULE`) | **4 — deep dive** |
| `fxceless/victory-fork` | 8 | 8 acima de #1496 (100 lidos, máx. #1513) | 2 | SIM (critério 2: correção de clippy e dos códigos escalares no unspill) | fica para 13:00 |
| `Marvisak/VIctory` | 0 | 181 acima de #3521 (200 lidos, máx. #3728) | 0 | SIM fraco (critério 3: shaders, compute e GPU por título) | fica para 13:00 |
| `DexNom/VIctory` | 0 | 4 acima de #148 (máx. #152) | 0 | SIM fraco (critério 3: `#149` corrige atomic sub em group memory) | fica para 13:00 |
| `uiliannn/VItoriaBr` | 0 | 3 acima de #126 (máx. #134) | 0 | SIM fraco (critério 3: `#133` é o crash do compilador AMD já catalogado em `XCL D-02`) | fica para 13:00 |
| `californiaAlmonds/OpenScarlett` | 0 | 1 acima de #60 (#66) | 0 | NAO (teste de HDR do shim, sem área de defeito) | — |
| `omonkeyz/xbox-emulator` | 0 | 1 (#395) | 0 | SIM fraco (critério 3, PE loader) | fica para 13:00 |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 | 0 | NAO | — |
| `stellarr-studio/kelvaroq` | 0 | 0 | 0 | NAO | — |
| `Belobug/VIctory-Belobug-AMD` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (HTTP 404, sexta hora) | bloqueado |
| `avatarmanz/VIctory-7900XTX` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (HTTP 404, sexta hora) | bloqueado |

## Como o sinal foi decidido

**Critério 1 (defeito do catálogo):** nenhum acionado. Os 710 hashes na janela dos cinco repositórios
com commit (`xcl` 376, `TuxBox` 169, `Xbox2Windows` 121, `SixNG` 36, `victory-fork` 8) foram
conferidos contra as 306 linhas de `catalogo/defeitos.csv`, colunas `introduzido` e `corrigido`
(262 hashes distintos): zero interseção por prefixo. Os títulos dos commits também não citam
`D-xx`. O cruzamento por **área** está em cada documento, marcado como hipótese.

**Critério 2 (correção/regressão/reversão):** acionado em quatro dos cinco repositórios com commit.
`TuxBox` e `SixNG` têm correções declaradas no assunto e no diff; `Xbox2Windows` tem uma leva de
lowerings que fecham recusas e três consertos de runtime; `xcl` tem a série de *pipeline library* e a
correção do `ballot_px_quad` sem laço para o DXC; `victory-fork` tem dois consertos (clippy e códigos
escalares no unspill) em 8 commits.

**Critério 3 (PR aberto em área de pista):** acionado nos repositórios sem commit. A "tabela de
pistas" continua não existindo em `docs/automacao/ANALISE_HORARIA.md` (desvio já registrado no
`INDEX.md` desde 2026-11-27 17:00); as áreas vieram das seções D do `catalogo/defeitos.md` e do
corpo de cada PR. Em `DexNom` o `#149` é a mesma classe de `XCL D-24`/`D-20` (ordenação de leitura de
argumentos/atomic em NVIDIA); em `VItoriaBr` o `#133` é literalmente a classe de `XCL D-02` (o
otimizador do bitcode desduzindo consultas de dimensão derruba o `amdxc64`).

**Critério 4 (atraso > 200):** acionado só em `xcl` (376). `TuxBox` (169), `Xbox2Windows` (121) e
`SixNG` (33) entraram pelo critério 2.

## Desempate e limite

Cinco repositórios tinham sinal por correção. O limite é 4 deep dives por execução. Ficou fora o de
menor janela, `fxceless/victory-fork` (8 commits, 2 merges, 8 PRs abertos) — o cursor dele **não
avança** e os 8 commits seguem na fila de 13:00, junto com `Marvisak` (181 abertos), `DexNom` (4),
`VItoriaBr` (3), `omonkeyz` (1) e os dois bloqueados.

## Observação de coleta

O `compare` do `SixNG` devolveu `ahead_by=33` às 12:13Z mas a lista paginada trouxe 36 commits: o
head avançou durante a coleta (`700627d9` às 12:14Z, `08137817` às 12:15Z, `2b2277a5` às 12:16Z). Os
36 foram documentados, e o cursor foi para o head medido no fim. Detalhe em `SixNG.md`, seção
Confiança.
