# Triagem de 2026-11-28 18:00

Coleta de 2026-11-28T18:07Z a 18:52Z (UTC real; 15:07–15:52 local, UTC−3). `last_run` de cada
repositório é o cursor de PRs mesclados no relógio do GitHub. A janela de commits é `commit_cursor`
(ou `last_commit`, quando não há cursor pendente) até o head da branch default lido nesta hora.

| repo | commits atras | PRs abertos | PRs mesclados | sinal | prioridade |
|---|---|---|---|---|---|
| `ivall/xcl` | 80 no `compare` a partir de `4701a874`, **78 sem documento** (2 já documentados: `2dad32e6`, `abf7e177`) | 10 acima de #3188 (#3189–#3198; 50 lidos, máx. #3198) | 0 na janela (o `compare` ainda devolve os merges de 17:00, já documentados) | SIM (critério 2: três commits do recompilador que **recusam** caminho não ordenado — `86a90eeb` rendezvous wave64, `dd84e0ad`/`c7752525` espera sem prova — mais `40d3a312` que troca a sonda `VirtualQuery` por `ReadProcessMemory`, e os PRs abertos `#3192`/`#3194` que são conserto) | **1 — deep dive** |
| `R0Y00/Xbox2Windows` | 11 (`9` no snapshot do `compare` + `24b895d7` e `f2972840` que chegaram depois) | 7 acima de #3378 (#3379–#3387; 50 lidos) | 4 na janela (#3031 `d418f70e`, #3125 `6685de7c`, #3139 `7291aef2`, #3375 `f2972840`) | SIM (critério 2: `#3375` corrige a máscara de índice de mesh de 1023 para 511 — com o flag de aresta no bit 9 a primitiva era culled; `#3139` recupera o par de disco publicado quando o PSO nativo falha) | **2 — deep dive** |
| `holdmysocks/SixNG` | 1 (`cff3f31b`) | 0 | 1 na janela (#555 `cff3f31b`) | SIM (critério 2: nome de decoração que é **conjunto** (struct `0x91501997`) não acha registro e é descartado — tatuagem de tronco some; cache de catálogo vai a 6) | **3 — deep dive** |
| `Marvisak/VIctory` | 0 (`identical`) | ≥50 acima de #3521 (#3722–#3775; 200 lidos, máx. #3775; 12 criados desde 16:55Z) | 0 | SIM (critério 3: `#3774` recusa descritor uniforme carregado que escapa do laço — área de `VIctory D-01`/`D-06`; `#3771` `DXGI_FORMAT_UNKNOWN` anunciava sampling e RTV; `#3767` limita a varredura de nomes SGD5) | **4 — deep dive** |
| `californiaAlmonds/OpenScarlett` | 0 (`identical`) | 1 acima de #60 (#66, testes de HDR — já registrado) | 0 | NAO (só teste de contrato) | — |
| `DexNom/VIctory` | 0 (`identical`) | 1 acima de #200 (#201 restaura `45ed8e3` + `8d67fa5`, já registrado) | 0 | SIM fraco (critério 2: restauração de commits, nenhum no catálogo) | fica |
| `uiliannn/VItoriaBr` | 0 (`identical`) | 0 acima de #134 (52 abertos, máx. #134) | 0 | NAO (sem delta; os três PRs de compilador AMD já foram documentados às 17:00) | — |
| `fxceless/victory-fork` | 0 (`identical`) | 3 acima de #1513 (#1517, #1518, #1519) | 0 | SIM fraco (critério 3: `#1518` "fixes NVIDIA 595.x 24px tile-sort bars" é área de `victory-fork D-06`/`VItoriaBr D-14`) | fica |
| `omonkeyz/xbox-emulator` | 0 (`identical`) | 1 acima de #394 (#395, já registrado) | 0 | SIM fraco (critério 3: validação de relocação PE) | fica |
| `brcly/TuxBox` | 0 (`identical`) | 2 acima de #405 (#406, #407) | 0 | SIM fraco (critério 2/3: `#407` corrige SCC de `s_andn2_b64` em modo wave_cull; `#406` mantém cena fora de estruturas que o host não construiu) | fica |
| `trustno1-coder/VIctory-NVIDIA` | 1 (`93008750`) | 50 lidos acima de cursor nulo (máx. #1363; `#1290` "Fix main's CI") | 0 | NAO (o commit é workflow de issues: fecha e tranca pedidos do jogo; não é código do emulador) | fica (baseline de PRs) |
| `walkthestars/GTA6-Asset-Info` | 0 (`identical`) | 0 | 0 | NAO | — |
| `stellarr-studio/kelvaroq` | 0 (`identical`) | 0 | 0 | NAO | — |
| `Belobug/VIctory-Belobug-AMD` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (HTTP 404, décima segunda hora) | bloqueado |
| `avatarmanz/VIctory-7900XTX` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (HTTP 404, décima segunda hora) | bloqueado |

## Como o sinal foi decidido

**Sanidade do estado (passo 1).** `gh api repos/<o>/<r> --jq .default_branch` foi pedido nos 15.
Treze responderam com a branch já gravada, inclusive `master` (`holdmysocks/SixNG`); `Belobug`
(`a-publier` no estado) e `avatarmanz` devolvem HTTP 404 em `repos/` pela décima segunda hora e não
puderam ser conferidos. Nenhuma branch divergiu do estado, então nada foi corrigido no passo 1. Nenhum
`last_commit` nulo — não houve baseline. Nenhum `compare` devolveu `diverged` (4 `ahead`: `xcl` 80,
`Xbox2Windows` 9, `SixNG` 1, `NVIDIA` 1; 9 `identical`). O estado tem **15** entradas em `repos` no
início e no fim; nenhuma removida, nada acrescentado à mão.

**Critério 1 (defeito do catálogo por hash): não acionou.** Os 270 tokens hexadecimais de
`catalogo/defeitos.csv` foram conferidos contra os 93 hashes de commit da janela (`xcl` 80,
`Xbox2Windows` 11, `SixNG` 1, `NVIDIA` 1) e contra os 5 hashes de merge (`d418f70e`, `6685de7c`,
`7291aef2`, `f2972840`, `cff3f31b`): **zero interseções**, inclusive por prefixo de 7 caracteres.

**Critério 2 (correção/regressão/reversão): acionou em `xcl`, `Xbox2Windows` e `SixNG`.** `xcl` traz a
série de "falhar fechado" do runtime D3D12.X (`dd84e0ad` recusa continuar quando o tempo decorrido não
prova que o produtor terminou, `c7752525` deixa de tratar célula de espera ilegível como satisfeita,
`40d3a312` lê com `ReadProcessMemory` para o Windows conter o fault) e do recompilador (`86a90eeb`
deixa de substituir por zero um rendezvous wave64 que não chegou). `Xbox2Windows` mescla `#3375`, que
corrige uma máscara de 10 bits aplicada a um campo de 9 bits, e `#3139`, que recupera o par de disco
quando o PSO nativo falha. `SixNG` mescla `#555`, que conserta nomes de decoração descartados.

**Critério 3 (PR aberto em área de pista):** a "tabela de pistas" continua ausente de
`docs/automacao/ANALISE_HORARIA.md`; as áreas vieram das seções D do catálogo e do corpo dos PRs.
Acionou em `Marvisak` (`#3774` descritor carregado, `#3771` `FORMAT_UNKNOWN`), `victory-fork`
(`#1518`/`#1519`), `omonkeyz` (`#395`) e `TuxBox` (`#406`/`#407`).

**Critério 4 (atraso > 200): não acionou.** O maior atraso é `xcl`, com 80 no `compare` (78 sem
documento).

## Desempate

Três repositórios acionaram o critério 2 e um o critério 3 com substância. Entraram `xcl` (78 commits
sem documento e a série de recusa explícita), `Xbox2Windows` (11 commits, 4 merges, um deles fora da
`main`, e 7 PRs abertos), `SixNG` (1 commit e 1 merge, diff pequeno e verificável) e `Marvisak`
(12 PRs novos na hora, um deles recusando descritor que escapa do laço). `TuxBox`, `victory-fork`,
`omonkeyz` e `DexNom` mantêm o cursor parado por decisão; ficam registrados com o que foi lido.

## Observações de coleta

- **`xcl`:** head `10d27297` às 18:07Z. O `compare` a partir de `4701a874` devolve 80 commits; dois
  (`2dad32e6`, `abf7e177`, já documentados às 12:00) continuam aparecendo na faixa e foram pulados
  **por hash** — a regra de pular hash documentado segue necessária. Documentaram-se os 40 primeiros
  não documentados na ordem do `compare` (`529fb19c` → `f2c0d36f`). O novo `commit_cursor` é
  `f2c0d36f`; sobram **38** commits sem documento.
- **`xcl`, merge x conteúdo:** `4577f09c` (#3085), `b0b041fd` (#2959) e `d85f2c6c`/`f2c0d36f`
  (#2960/#2961) são merges cujo diff contra o primeiro pai repete o do commit de conteúdo
  (`651867a4`, `23c06c9f`, `40d3a312`, `51cda2f4`). Ambos entram na tabela, como nas horas anteriores,
  e a contagem de commits únicos da faixa é 76, não 80.
- **`Xbox2Windows`:** o snapshot do `compare` pegou `aec2e084...6685de7c` (9 commits); `24b895d7` e
  `f2972840` chegaram às 18:08Z. O head final é `f2972840`. **`#3139` foi mesclado em
  `codex/native-graphics-integration`, não na `main`:** o merge `7291aef2` não está no histórico da
  default (o `compare` `7291aef2...main` devolve `diverged`, 16 à frente e 1 atrás), então 1 dos 4
  merges da janela não entrou na faixa de commits.
- **`SixNG`:** o commit `cff3f31b` é o squash do `#555` ("…(#555)"), não um merge de dois pais; 1
  merge = 1 commit de conteúdo.
- **`Marvisak`:** 50 PRs abertos lidos; o menor número da janela (#3722, criado 11:34Z) ainda é maior
  que o cursor #3521, então o total real é ≥ 50 e o cursor não prova cobertura dos antigos. 12 foram
  criados depois de 16:55Z e são os documentados.
- **Merges x commits:** `xcl` 0 merges e 0 commits de conteúdo novos (`compare` até `10d27297`, que é
  merge de 17:00); `Xbox2Windows` 4 merges contra 11 commits (3 dos merges são de ramo interno e 1 é
  fora da `main`); `SixNG` 1 = 1.
- **Rate limit:** nenhum 403 nesta coleta (4.984 restantes no início, 4.812 no fim).
