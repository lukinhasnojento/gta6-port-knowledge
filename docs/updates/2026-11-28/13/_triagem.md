# Triagem de 2026-11-28 13:00

Coleta de 2026-11-28T13:07–13:52Z (UTC real). O relógio do GitHub está alinhado ao UTC real nesta
coleta: o merge `#541` do `SixNG` (13:05:41Z) e o commit `3b6201a4` (13:03:30Z) caíram dentro da
janela. `last_run` de cada repositório é o cursor de PRs mesclados gravado pelo ciclo anterior, no
relógio do GitHub.

| repo | commits atras | PRs abertos | PRs mesclados | sinal | prioridade |
|---|---|---|---|---|---|
| `ivall/xcl` | 338 (a partir do `commit_cursor 3dbac13a`; 376 a partir de `last_commit`) | 62 acima de #3075 (400 lidos em 4 páginas, máx. #3155) | 19 | SIM (critério 4: 338 > 200; critério 2: série D3D12.X de lock/ownership e a correção do logaritmo vizinho) | **1 — deep dive** |
| `brcly/TuxBox` | 148 (a partir do `commit_cursor 8420ba03`) | 14 acima de #339 (máx. #397) | 47 | SIM (critério 2: aceite de recusa, ballot no lugar de `WaveActiveMin`, cache da NVIDIA; critério 3: #344 é a área de D-22) | **2 — deep dive** |
| `R0Y00/Xbox2Windows` | 86 (a partir do `commit_cursor 50e4f763`) | 41 acima de #3215 (máx. #3354) | 71 | SIM (critério 2: leva de lowerings e consertos de runtime; critério 3: D-18/D-19 em `a166a753`, D-27 em `152e93b4`) | **3 — deep dive** |
| `fxceless/victory-fork` | 8 | 8 acima de #1496 (máx. #1513) | 2 | SIM (critério 2: clippy e códigos escalares; critério 3: #1513 carrega a chave de cache com todos os switches, área de D-02) | **4 — deep dive** |
| `holdmysocks/SixNG` | 7 | 1 acima de #541 (#554) | 9 | SIM (critério 2: vidro de veículo mantém a zona do adesivo em `_COLOR0`, registro sem modelo nomeado pelo chassi) | fica para 14:00 (limite) |
| `Marvisak/VIctory` | 0 | 187 acima de #3521 (300 lidos, máx. #3734) | 0 | SIM fraco (critério 3: shaders, compute e GPU por título) | fica |
| `DexNom/VIctory` | 0 | 4 acima de #148 (máx. #152) | 0 | SIM fraco (critério 3) | fica |
| `uiliannn/VItoriaBr` | 0 | 3 acima de #126 (máx. #134) | 0 | SIM fraco (critério 3; #133 é a classe de `XCL D-02`) | fica |
| `omonkeyz/xbox-emulator` | 0 | 1 (#395) | 0 | SIM fraco (critério 3, PE loader) | fica |
| `californiaAlmonds/OpenScarlett` | 0 | 1 acima de #60 (#66) | 0 | NAO (teste de HDR do shim, sem área de defeito) | — |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 | 0 | NAO | — |
| `stellarr-studio/kelvaroq` | 0 | 0 | 0 | NAO | — |
| `Belobug/VIctory-Belobug-AMD` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (HTTP 404, sétima hora) | bloqueado |
| `avatarmanz/VIctory-7900XTX` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (HTTP 404, sétima hora) | bloqueado |

## Como o sinal foi decidido

**Critério 1 (defeito do catálogo):** nenhum acionado. Os hashes de todos os commits e merges da
janela dos quatro repositórios com commit — 269 no `xcl` (250 na página do `compare` + 19 merges),
195 no `TuxBox` (148 + 47), 157 no `Xbox2Windows` (86 + 71) e 10 no `victory-fork` (8 + 2) — foram
conferidos contra as 305 linhas de `catalogo/defeitos.csv`, colunas `introduzido` e `corrigido` (262
hashes distintos): zero interseção por prefixo. Nenhum
título de commit cita `D-xx`. O cruzamento por **área** está em cada documento, marcado como hipótese
onde não há prova no diff.

**Critério 2 (correção/regressão/reversão):** acionado nos quatro escolhidos. `xcl` tem a série
D3D12.X de lock/ownership (o commit `5ee15471` retém objetos submetidos quando a espera falha, e
`07db0df9` da hora anterior é vizinho) e a série de `compare` que devolve `diverged` só se o
histórico for reescrito — não ocorreu. `TuxBox` tem quatro aceites de recusa que são consertos
declarados (`94f3803b` barreira `ACCESS_COMMON`, `24ebba7e` ballot no lugar de `WaveActiveMin`,
`5f926353` handle nulo para heap invisível, `903567aa` barreira legada) e o conserto do cache da
NVIDIA (`0e2e6ef3`). `Xbox2Windows` tem `4d7265c0` (erros de MSVC), `a0b45d43` (caminhos resolvidos),
`4b0350ea`-classe ausente nesta janela, e `20ef1f72` (BC4_UNORM devolvia o formato typeless).
`victory-fork` tem `38f404ac` (clippy) e `8ddcf358` (tabela de códigos escalares 128 → 256).

**Critério 3 (PR aberto em área de pista):** a "tabela de pistas" continua não existindo em
`docs/automacao/ANALISE_HORARIA.md` (desvio já registrado no `INDEX.md` desde 2026-11-27 17:00). As
áreas vieram das seções D do `catalogo/defeitos.md` e do corpo de cada PR. Acionou em:
`TuxBox` #344 (slots de descritor que um root raster deixa de fora — área de D-22) e #373 (tempo de
vida do render heap — área de D-17); `victory-fork` #1513, que carrega a chave do cache de shader com
todos os switches de wave — é literalmente a classe de `victory-fork D-02`; `Xbox2Windows` #3315
(índice de heap por lane preso ao marca d'água do waterfall — área de #2644 T3) e #3352/#3354
(wave64 em pares de lanes e `v_readfirstlane` com EXEC vazio).

**Critério 4 (atraso > 200):** acionado só em `xcl` (338). `TuxBox` (148), `Xbox2Windows` (86) e
`victory-fork` (8) entraram pelo critério 2.

## Desempate e limite

Cinco repositórios tinham sinal por correção e o limite é 4. Ficou fora `holdmysocks/SixNG`, que é o
de menor janela com trabalho novo de verdade (7 commits, 9 merges, todos entre 12:15Z e 13:05Z) e
portanto não perde nada por esperar uma hora: o cursor dele está no head `2b2277a5`, e os 7 commits
seguem inteiros para 14:00. Os três bloqueados por 404 e os quatro com sinal fraco continuam com o
cursor parado.

## Observações de coleta

- **`xcl`:** `compare` a partir de `3dbac13a` devolve `ahead_by=338` mas apenas 250 commits na
  página; os 40 documentados são os 40 primeiros dessa página e a interseção com o documento das
  12:00 é **zero**. Detalhe na seção Confiança de `xcl.md`.
- **`TuxBox` e `Xbox2Windows`:** o `commit_cursor` gravado apontava para o 40º commit do ciclo
  anterior, mas a ordem do `compare` não é a ordem em que o ciclo anterior documentou. Medi a
  sobreposição antes de escrever: em `TuxBox` os 19 primeiros da página já estavam no documento das
  12:00, e em `Xbox2Windows` os 5 primeiros. As janelas desta hora começam no primeiro commit não
  documentado (índices 19 e 5), e por isso os dois documentos têm 40 commits **novos**, sem repetir
  linha.
- **`SixNG`:** `compare` a partir de `2b2277a5` devolve 7 commits e a lista de mesclados devolve 9,
  todos dentro desta hora.
