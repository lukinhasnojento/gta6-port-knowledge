# Triagem de 2026-11-28 16:00

Coleta de 2026-11-28T16:07–16:23Z (UTC). `last_run` de cada repositório é o cursor de PRs mesclados no
relógio do GitHub. A janela de commits é `commit_cursor` (ou `last_commit`, quando não há cursor
pendente) até o head da branch default lido nesta hora.

| repo | commits atras | PRs abertos | PRs mesclados | sinal | prioridade |
|---|---|---|---|---|---|
| `ivall/xcl` | 154 no `compare` a partir de `97583afd`, **128 sem documento** | 9 acima de #3165 (#3167–#3177; 100 lidos, máx. #3177) | 1 na janela (#3100, `bb9a6b4b`) | SIM (critério 2: `#3100` importa PSOs exatos da biblioteca anterior; os 9 abertos incluem `#3167` "closed standby recorded empty" e `#3169` "no disk driver blob ... static samplers on NVIDIA") | **1 — deep dive** |
| `brcly/TuxBox` | 41 no `compare` a partir de `99eef7ab`, **18 sem documento** | 0 acima de #398 (7 abertos, máx. #393) | 2 na janela (#401 `e3f3e153`, #402 `2af42c94`) | SIM (critério 2: `#401` corrige o `Map` com range de 32 bytes num buffer de 16; os commits incluem `055adb92` label esperando escrita gravada, `2ea48307` pipelines liberados até o primeiro bind, `ce8a7c86`/`b295769f` orçamento de VRAM e mips parciais) | **2 — deep dive** |
| `DexNom/VIctory` | 0 | 52 acima de #148 (100 lidos, máx. #200) | 0 | SIM (**critério 1**: `#183` restaura `274f698`, que é o conserto de **D-12** na linha de prosa do catálogo; e critério 2: 42 PRs são `restore` de commits que `570a941` derrubou) | **3 — deep dive** |
| `R0Y00/Xbox2Windows` | 0 | 4 acima de #3371 (#3372–#3375) | 0 | SIM (critério 3: `#3373` compõe todos os planos quando há mais de um — tela preta; `#3375` mantém o índice de mesh em 9 bits; `#3372` a16 com dados de 32 bits) | **4 — deep dive** |
| `Marvisak/VIctory` | 0 | ≥100 acima de #3521 (100 lidos, máx. #3759; piso) | 0 | SIM fraco (critério 2/3: `#3754` "revives #3035" é regressão declarada; `#3759` meia wave antes de barreira) | fica (cursor parado) |
| `uiliannn/VItoriaBr` | 0 | 3 acima de #126 (#132, #133, #134) | 0 | SIM fraco (critério 3: `#133` "do not reuse texture size query (fixes AMD compiler crash)", área de `XCL D-02`) | fica |
| `omonkeyz/xbox-emulator` | 0 | 1 (#395 "Fix the PE relocation block validation") | 0 | SIM fraco (critério 3: área de PE/relocação no catálogo do repositório) | fica |
| `trustno1-coder/VIctory-NVIDIA` | 0 | 100 lidos, máx. #1226 (cursor de abertos nulo desde a inclusão às 15:50Z) | 0 | SIM fraco (critério 3; primeira triagem do repositório, sem leitura por diff) | fica (baseline de PRs) |
| `californiaAlmonds/OpenScarlett` | 0 | 1 acima de #60 (#66, teste de HDR) | 0 | NAO | — |
| `fxceless/victory-fork` | 0 | 0 acima de #1513 (máx. #1509) | 0 | NAO (cursor zerado às 15:00) | — |
| `holdmysocks/SixNG` | 0 (`identical`) | 0 | 0 | NAO | — |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 | 0 | NAO | — |
| `stellarr-studio/kelvaroq` | 0 | 0 | 0 | NAO | — |
| `Belobug/VIctory-Belobug-AMD` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (HTTP 404, décima hora) | bloqueado |
| `avatarmanz/VIctory-7900XTX` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (HTTP 404, décima hora) | bloqueado |

## Como o sinal foi decidido

**Sanidade do estado (passo 1).** `gh api repos/<o>/<r> --jq .default_branch` foi pedido nos 15. Treze
responderam com a branch já gravada, inclusive `master` (`holdmysocks/SixNG`) e `a-publier`
(`Belobug`, não lido nesta hora). `Belobug/VIctory-Belobug-AMD` e `avatarmanz/VIctory-7900XTX`
devolvem HTTP 404 em `repos/` pela décima hora. Nenhum `last_commit` nulo — não houve baseline.
Nenhum `compare` devolveu `diverged` (2 `ahead`, 11 `identical`). O estado tem **15** entradas em
`repos` (a de `trustno1-coder/VIctory-NVIDIA` foi acrescentada às 15:50Z, depois da execução das
15:00); nenhuma foi removida.

**Critério 1 (defeito do catálogo por hash): acionou em `DexNom/VIctory`.** O PR aberto `#183`
restaura `274f698`, e o catálogo (`catalogo/defeitos.csv`, linha de prosa de D-12) registra
`274f698` / `17de445` como o par "corrigido / reintroduzido" do overflow de u64 no `clwb_loop`. É a
primeira interseção de hash com o catálogo desde que a rotina passou a conferir (as horas anteriores
registraram zero em 358, 631 e 710 hashes). Nos outros repositórios, os 154 + 41 hashes de commit da
janela e os 3 hashes de merge (`bb9a6b4b`, `e3f3e153`, `2af42c94`) não têm interseção: zero.

**Critério 2 (correção/regressão/reversão): acionou em `xcl`, `TuxBox` e `DexNom`.** `DexNom` é o caso
mais forte em volume: 42 dos 52 PRs abertos são `cherry-pick -x` de commits que o publish `570a941`
(+49.715/−36.762, 170 arquivos, 2026-11-25T12:18:03Z) derrubou da `main`.

**Critério 3 (PR aberto em área de pista):** a "tabela de pistas" continua ausente de
`docs/automacao/ANALISE_HORARIA.md`; as áreas vieram das seções D do catálogo e do corpo dos PRs.
Acionou em `Xbox2Windows` (4), `Marvisak` (≥100), `VItoriaBr` (3), `omonkeyz` (1) e `NVIDIA` (piso de
100).

**Critério 4 (atraso > 200):** não acionou. O maior atraso é `xcl`, com 154.

## Desempate

Três repositórios tinham sinal por correção e um por defeito de catálogo, e o limite é 4. Entraram
`xcl` (154 commits, o maior atraso e o merge do dia), `TuxBox` (41 commits, dois merges com sintoma),
`DexNom` (critério 1 por hash, 52 PRs) e `Xbox2Windows` (correção de tela preta com diff de 3 linhas,
verificável). `Marvisak`, `VItoriaBr`, `omonkeyz` e `NVIDIA` mantêm o cursor parado por decisão; os
quatro estão registrados com o que foi lido.

## Observações de coleta

- **`xcl`:** head `bb9a6b4b` às 15:37Z. O `compare` a partir de `97583afd` devolve 154 commits, dos
  quais 128 sem documento; documentaram-se os 40 primeiros **não documentados** na ordem do `compare`
  (`18d38a67` … `3cde0cc3`). O novo `commit_cursor` é o 40º, `3cde0cc3`; `compare` a partir dele
  devolve 114 e re-inclui `2dad32e6` e `abf7e177`, já documentados às 12:00 — o próximo ciclo deve
  pular hash já documentado.
- **`TuxBox`:** head `2af42c94`. Os 18 commits sem documento foram todos documentados, incluindo os
  merges `#401` e `#402`; a fila de commits do repositório zera e o `commit_cursor` sai do estado.
- **`DexNom`:** head `4ad8692` não andou em 24 h. O PR `#155` é "Findings (not for merge)" e o `#173`
  é "Vanviro 16gb" — os dois ficam fora da contagem de restaurações.
- **`Xbox2Windows`:** head `f5a6f6ba`, parado desde 14:15Z; a hora é só de PRs abertos.
- **Merges x commits:** em `xcl` o merge `#3100` (`bb9a6b4b`) traz o commit de conteúdo `2de72223`,
  que aparece no `compare` na posição 147 e **não** está entre os 40 documentados; ele entra pelo
  documento do PR, não pela tabela de commits.
- **Rate limit:** nenhum 403 nesta coleta (4.9 mil restantes no início).
