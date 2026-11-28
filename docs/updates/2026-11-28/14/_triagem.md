# Triagem de 2026-11-28 14:00

Coleta de 2026-11-28T14:07–14:5xZ (UTC real). O relógio do GitHub está alinhado ao UTC real nesta
coleta: o merge `#3157` do `xcl` (`14:06:59Z`) e o `#3094` do `Xbox2Windows` (`14:07:49Z`) entraram na
janela. `last_run` de cada repositório é o cursor de PRs mesclados gravado pelo ciclo anterior, no
relógio do GitHub.

| repo | commits atras | PRs abertos | PRs mesclados | sinal | prioridade |
|---|---|---|---|---|---|
| `ivall/xcl` | 232 (a partir do `commit_cursor e5266254`; 233 a partir de `last_commit`) | 2 acima de #3155 (#3156, #3158; 300 lidos, máx. #3158) | 7 na janela (3 novos: #3149, #3150, #3157) | SIM (critério 4: 232 > 200; critério 2: série de identidade de cache e de lock/ownership esparso) | **1 — deep dive** |
| `brcly/TuxBox` | 99 (a partir de `02fceb26`; 101 a partir de `last_commit`) | 1 acima de #397 (#398) | 30 na janela (24 novos) | SIM (critério 2: #362, #367, #377, #386 com sintoma declarado; critério 3: #398 é ordenação de espera/submissão) | **2 — deep dive** |
| `R0Y00/Xbox2Windows` | 84 (a partir de `d649e470`; 90 a partir de `last_commit`) | 6 acima de #3354 (#3355–#3357, #3360, #3362, #3364) | 38 na janela (33 novos) | SIM (critério 2: #3285, #3307, #3334, #3292/#3295 e a leva de fechamento de recusas; critério 3: #3364 é commit de slot novo no XMem) | **3 — deep dive** |
| `holdmysocks/SixNG` | 8 (7 não documentados) | 0 | 10 na janela (7 novos) | SIM (critério 2: #523, #541, #496, #552 corrigem dados e exportação; #538 já documentado) | **4 — deep dive** |
| `Marvisak/VIctory` | 0 | 195 acima de #3521 (300 lidos, máx. #3743) | 0 | SIM fraco (critério 3: shaders, GPU e áudio na fila) | fica (cursor parado) |
| `DexNom/VIctory` | 0 | 5 acima de #148 (máx. #153) | 0 | SIM fraco (critério 3: #152/#153 mexem em lock de heap e memória do título) | fica |
| `uiliannn/VItoriaBr` | 0 | 3 acima de #126 (máx. #134) | 0 | SIM fraco (critério 3; #133 é "do not reuse texture size query (fixes AMD compiler crash)", área de `XCL D-02`) | fica |
| `omonkeyz/xbox-emulator` | 0 | 1 (#395) | 0 | SIM fraco (critério 3: "Fix the PE relocation block validation") | fica |
| `californiaAlmonds/OpenScarlett` | 0 | 1 acima de #60 (#66) | 0 | NAO (ajuste de teste de HDR) | — |
| `fxceless/victory-fork` | 1 (`368967f8`, ainda não documentado) | 0 acima de #1513 | 0 | NAO (o único commit é a troca de árvore "run-403" com a água corrigida; sem sinal de correção catalogada) | fica |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 | 0 | NAO | — |
| `stellarr-studio/kelvaroq` | 0 | 0 | 0 | NAO | — |
| `Belobug/VIctory-Belobug-AMD` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (HTTP 404, oitava hora) | bloqueado |
| `avatarmanz/VIctory-7900XTX` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (HTTP 404, oitava hora) | bloqueado |

## Como o sinal foi decidido

**Critério 1 (defeito do catálogo):** nenhum acionado. Os hashes dos commits e merges das quatro
janelas (232 + 99 + 84 + 8 = 423 no total, 40 documentados por repositório) foram conferidos por
prefixo contra as colunas `introduzido` e `corrigido` de `catalogo/defeitos.csv` (305 linhas, 262
hashes distintos): zero interseção. Nenhum assunto de commit cita `D-xx`.

**Critério 2 (correção/regressão/reversão):** acionado nos quatro escolhidos. `xcl`: a série de
identidade de cache de pipeline (samplers, `NodeMask`, dispositivo, root signature, `HEAP_FLAT`) e a
de estado esparso (criar/liberar/unmap fora do lock, geração exata de fence), mais os merges `#3149`
(copy-back apagava escrita da GPU em chunk alterado pela CPU) e `#3157` (ramo de `rendezvous`
divergente). `TuxBox`: quatro consertos com sintoma no diff (`#362` clear de UAV com handle de outra
arena, `#367` `VirtualQuery` no wineserver, `#377` páginas preenchidas depois do enqueue, `#386`
carga raw fora de EXEC lendo `+0x7ffff000`) e o `#380` (VCC inventado em `v_cmpx` no GFX10).
`Xbox2Windows`: `#3285` (teto do `fract`), `#3307` (UB em `unpin_slot`), `#3334` (`vcc_hi` como
máscara), `#3292`/`#3295` (conversões norm) e o `#3293` (BC2/BC6H em branco). `SixNG`: `#523`/`#541`
(nomeação e fusão de registros de veículo), `#496` (zona do adesivo no vidro) e `#552` (exportação
do personagem).

**Critério 3 (PR aberto em área de pista):** a "tabela de pistas" continua não existindo em
`docs/automacao/ANALISE_HORARIA.md` (desvio registrado desde 2026-11-27 17:00). As áreas vieram das
seções D do `catalogo/defeitos.md` e do corpo de cada PR. Acionou em `TuxBox` #398 (espera de label
vs. submissão), `Xbox2Windows` #3364/#3355 (slot novo do XMem), #3356 (ordem de LDS na wave32) e
`VItoriaBr` #133 (crash do compilador AMD).

**Critério 4 (atraso > 200):** acionado só em `xcl` (232). `TuxBox` (99), `Xbox2Windows` (84) e
`SixNG` (8) entraram pelo critério 2.

## Desempate e limite

Quatro repositórios tinham sinal por correção e o limite é 4 — coube exatamente na ordem acima.
`Marvisak` (195 abertos), `DexNom`, `VItoriaBr` e `omonkeyz` têm sinal fraco e o cursor fica parado
por decisão, não por falta de tempo. `victory-fork` tem 1 commit pendente (`368967f8`, o build
"run-403" com a água corrigida que chegou às 13:25Z); não foi escolhido e o cursor dele continua em
`368967f8`, então o commit será visto de novo no próximo ciclo.

## Observações de coleta

- **`xcl`:** o `compare` a partir de `last_commit` devolve 233 commits (211 sem documento anterior);
  os 40 desta hora são os 40 primeiros não documentados da página. O `head` andou durante a coleta
  (`b937fbab` às 14:07Z, com os merges #3149/#3150/#3157), e esses três merges entraram no documento.
- **`TuxBox`:** 101 commits na página, 85 sem documento anterior; 40 documentados. Dos 30 merges,
  24 não estavam em documento nenhum; 6 já estavam (#392, #356, #351 e os merges de commit #357,
  #345, #329).
- **`Xbox2Windows`:** 90 commits na página, 71 sem documento anterior; 40 documentados. Dos 38
  merges, 33 eram novos; 5 já estavam (#2890, #3335, #3314, #3308, #3269). O `head` `06cfb07f` é o
  merge do #3094, que entrou às 14:07:49Z, dentro da coleta.
- **`SixNG`:** 8 commits desde `2b2277a5`, 7 sem documento; todos os 7 entraram. 10 merges na
  janela, 7 novos.
- **Merges x commits:** em `xcl` a contagem de merges (7) não bate com a de commits (40) por
  construção — a página do `compare` não ordena por merge. O mesmo vale para os outros três.
