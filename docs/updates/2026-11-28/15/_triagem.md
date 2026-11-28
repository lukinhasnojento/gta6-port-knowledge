# Triagem de 2026-11-28 15:00

Coleta de 2026-11-28T15:07–15:2xZ (UTC real; 12:07 local, UTC−3). `last_run` de cada repositório é o
cursor de PRs mesclados no relógio do GitHub. O relógio do GitHub está alinhado ao UTC real nesta
coleta: o merge `#3159` do `xcl` (`14:48:56Z`), o `#3365` do `Xbox2Windows` (`14:15:27Z`) e a leva de
15:06–15:07Z do `TuxBox` caíram dentro da janela.

| repo | commits atras | PRs abertos | PRs mesclados | sinal | prioridade |
|---|---|---|---|---|---|
| `ivall/xcl` | 193 (a partir do `commit_cursor be9d4057`; 194 a partir do `last_commit e9389076`) | 4 acima de #3158 (#3160, #3161, #3164, #3165; 400 lidos, máx. #3165) | 1 na janela (#3159) | SIM (critério 2: `#3159` restaura a fronteira de bloco do `Texture2DMS` — regressão de **D-24**; critério 4 não aciona: 193 < 200) | **1 — deep dive** |
| `brcly/TuxBox` | 92 (a partir de `1f44aec1`; 93 a partir de `last_commit 94cabfa8`) | 0 acima de #398 (8 abertos, máx. #393) | 12 na janela (11 novos: #320, #375, #376, #394, #398, #395, #397, #399, #400, #396, #371; #392 já documentado) | SIM (critério 2: #399 corrige `0x8007000e` em heap de 2.097.152 descritores, #400 o `/W4 /WX` do MSVC, #398 a liberação de label em 5 s, #396 a aresta de volta do laço, #395 a evicção quadrática) | **2 — deep dive** |
| `R0Y00/Xbox2Windows` | 72 (a partir de `cdce92da`; 73 a partir de `last_commit 1f07b2c1`) | 4 acima de #3364 (#3366, #3367, #3369, #3371) | 1 na janela (#3365) | SIM (critério 2: `#3365` normaliza a realocação na chave do perfil Mesh57; critério 3: os 4 abertos são lowering de `v_cvt_f64` no compute direto e log do caminho de present) | **3 — deep dive** |
| `fxceless/victory-fork` | 0 a partir do cursor; **1 commit preso no próprio cursor** (`368967f8`, chegou 13:25Z e nunca foi documentado) | 0 acima de #1513 (máx. #1509) | 0 | SIM (critério 2, por reversão: `368967f8` apaga `pipeline/sampler_order.rs` e o guia de ray tracing documentados às 13:00) | **4 — deep dive (desbloqueia o cursor)** |
| `Marvisak/VIctory` | 0 | 203 acima de #3521 (400 lidos, máx. #3753) | 0 | SIM fraco (critério 3: shaders, GPU e áudio na fila) | fica (cursor parado) |
| `DexNom/VIctory` | 0 | 7 acima de #148 (máx. #155) | 0 | SIM fraco (critério 3: #152 tira a cópia de primeiro uso de dentro do lock de heap, #153 grava os stores de `InitHeaders` a partir de um registro e mede 0,5→2,2 fps numa RX 7900 XTX) | fica |
| `uiliannn/VItoriaBr` | 0 | 3 acima de #126 (máx. #134) | 0 | SIM fraco (critério 3; #133 "do not reuse texture size query (fixes AMD compiler crash)", área de `XCL D-02`) | fica |
| `omonkeyz/xbox-emulator` | 0 | 1 (#395) | 0 | SIM fraco (critério 3: "Fix the PE relocation block validation") | fica |
| `californiaAlmonds/OpenScarlett` | 0 | 1 acima de #60 (#66) | 0 | NAO (ajuste de teste de HDR, sem commit novo) | — |
| `holdmysocks/SixNG` | 0 | 0 | 0 | NAO | — |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 | 0 | NAO | — |
| `stellarr-studio/kelvaroq` | 0 | 0 | 0 | NAO | — |
| `Belobug/VIctory-Belobug-AMD` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (HTTP 404, nona hora) | bloqueado |
| `avatarmanz/VIctory-7900XTX` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (HTTP 404, nona hora) | bloqueado |

## Como o sinal foi decidido

**Sanidade do estado (passo 1).** `gh api repos/<o>/<r> --jq .default_branch` foi pedido nos 14. Nos 12
acessíveis, `default_branch` é idêntico ao gravado, inclusive `master` (`holdmysocks/SixNG`);
`Belobug/VIctory-Belobug-AMD` e `avatarmanz/VIctory-7900XTX` devolvem HTTP 404 em `repos/` pela nona
hora, cursores parados, triagem NAO VERIFICADO. Nenhum `last_commit` nulo — não houve baseline. Nenhum
`compare` devolveu `diverged` (4 `ahead`, 8 `identical`). Contagem de `repos`: 14 no início e no fim,
nenhuma entrada removida ou acrescentada.

**Critério 1 (defeito do catálogo por hash):** não acionou. Os hashes de todos os commits e merges das
quatro janelas (193 + 92 + 72 + 1 = 358 hashes de commit, mais 14 hashes de merge) foram conferidos por
prefixo de 12 contra as colunas `introduzido`/`corrigido` de `catalogo/defeitos.csv` (305 linhas, 262
hashes distintos): zero interseção.

**Critério 2 (correção/regressão/reversão):** acionou nos quatro escolhidos. `xcl`: `#3159` é o segundo
retorno do padrão de **D-24** (a simplificação de CFG re-fundiu o bloco que separava `GetDimensions` de
`TextureLoad`). `TuxBox`: cinco PRs com sintoma declarado. `Xbox2Windows`: `#3365` normaliza a chave do
perfil. `victory-fork`: `368967f8` é uma **reversão** de árvore que apaga dois consertos documentados.

**Critério 3 (PR aberto em área de pista):** a "tabela de pistas" continua não existindo em
`docs/automacao/ANALISE_HORARIA.md` (desvio registrado desde 2026-11-27 17:00); as áreas vieram das
seções D do catálogo e do corpo de cada PR. Acionou em `Marvisak` (203 abertos), `DexNom` (#152/#153),
`VItoriaBr` (#133) e `omonkeyz` (#395).

**Critério 4 (atraso > 200):** não acionou em ninguém. `xcl` caiu de 232 para 193 e é o maior atraso.

## Desempate e limite

Três repositórios tinham sinal por correção (critério 2) e o limite é 4. O quarto slot foi usado em
`victory-fork` por um motivo de estado, não de mérito: o ciclo de 13:00 gravou `commit_cursor =
368967f8`, que é o **próprio** commit pendente, e `compare(368967f8...368967f8)` devolve zero — o
commit ficaria invisível para sempre. Documentá-lo (1 commit) e avançar o cursor é a única saída que não
perde o delta; o desvio está registrado em `_falhas.md`. `Marvisak` (203 abertos), `DexNom`,
`VItoriaBr` e `omonkeyz` mantêm o cursor parado por decisão.

## Observações de coleta

- **`xcl`:** head `309c3ec5` às 14:48Z, depois do head do ciclo anterior (`b937fbab`). 168 dos 193
  commits do `compare` não têm documento anterior; os 40 desta hora são os 40 primeiros dessa ordem.
  O merge `#3159` está além do cursor e foi documentado pela data.
- **`TuxBox`:** 59 dos 92 commits sem documento; 40 documentados. Onze merges novos, cinco deles com
  `merged_at` exatamente `15:07:14Z` (fila de merge) e cinco em `14:41:16Z` (a pilha de ray tracing).
  O documento de 14:00 registrou o merge do `#377` com o hash `3eb76bd5`, que **não existe**; o real é
  `3eb76bd9` (registrado em `TuxBox.md` → Confiança).
- **`Xbox2Windows`:** 13 dos 72 commits sem documento — a fila deste repositório praticamente zerou.
  O head não andou desde `f5a6f6ba` (14:15Z).
- **Merges x commits:** a contagem de merges não bate com a de commits por construção — a página do
  `compare` não ordena por merge, e o `merge_commit_sha` de PR mesclado por rebase aponta para o tip da
  branch (caso de `#320` em `TuxBox`).
- **Rate limit:** nenhum 403 nesta coleta.
