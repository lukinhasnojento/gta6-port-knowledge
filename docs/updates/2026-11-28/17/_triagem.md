# Triagem de 2026-11-28 17:00

Coleta de 2026-11-28T17:07Z a 17:38Z (UTC real; 14:07–14:38 local, UTC−3). `last_run` de cada
repositório é o cursor de PRs mesclados no relógio do GitHub. A janela de commits é `commit_cursor`
(ou `last_commit`, quando não há cursor pendente) até o head da branch default lido nesta hora.

| repo | commits atras | PRs abertos | PRs mesclados | sinal | prioridade |
|---|---|---|---|---|---|
| `ivall/xcl` | 120 no `compare` a partir de `3cde0cc3`, **118 sem documento** (2 já documentados: `2dad32e6`, `abf7e177`) | 7 acima de #3177 (#3178–#3188; 200 lidos, máx. #3188) | 3 na janela (#3175 `db3921dc`, #3181 `59e4be57`, #3182 `10d27297`) | SIM (critério 2: as três merges são `fix/`/`perf/` do tradutor — `#3157` `DispatchX` wave-uniforme, `#3159` fronteira do bloco MSAA, `#3085` unidades de MiB; e 6 commits de recompilador com "keep/defer/clamp") | **1 — deep dive** |
| `brcly/TuxBox` | 10 no `compare` a partir de `2af42c94` (18 sem documento na triagem anterior, fila zerada), **+1** de merge que chegou depois do snapshot | 2 acima de #398 (#404, #405; 5 lidos) | 5 na janela (#373 `4cb3347d`, #385 `19dc351b`, #391 `72edad75`, #393 `b90cc37e`, #404 `b8355fa2`) | SIM (critério 2: `#373` mantém heap de render enquanto o range convidado está commitado — sintoma de device loss em 15–18 min; `#391`/`#393` seguem registros de descritor copiados pelo título; `5de0e62b` recicla buffers de mip alterado) | **2 — deep dive** |
| `R0Y00/Xbox2Windows` | 2 (`539c3ad5` + merge `aec2e084`) | 3 acima de #3375 (#3376–#3378; 115 lidos) | 1 na janela (#3332 `aec2e084`) | SIM (critério 2: `#3332` conserta `#3331` — os launchers não conseguiam desligar a retenção de threads do hook de startup; e critério 3: `#3376` recompõe planos apresentados, mesma área da tela preta de `#3373`) | **3 — deep dive** |
| `uiliannn/VItoriaBr` | 0 (`identical`) | 3 acima de #126 (#132–#134; 77 lidos) | 0 | SIM (critério 3: `#133` "do not reuse texture size query (fixes AMD compiler crash)" é a área de `XCL D-02`/`VItoriaBr D-20` — regressão de codegen que derrubou o `amdxc64`) | **4 — deep dive** |
| `Marvisak/VIctory` | 0 | ≥100 acima de #3521 (200 lidos, máx. #3764; cursor é piso) | 0 | SIM (critério 2/3: `#3754` "revives #3035" é regressão declarada; `#3759` "halves (on #3683)"; área de descritor/view heap em `D-01`–`D-03`) | fica (cursor parado) |
| `DexNom/VIctory` | 0 | 1 acima de #200 (#201 restaura `45ed8e3` + `8d67fa5`) | 0 | SIM fraco (critério 2: restauração de commits) | fica |
| `omonkeyz/xbox-emulator` | 0 | 1 acima de #394 (#395 "Fix the PE relocation block validation") | 0 | SIM fraco (critério 3: validação de relocação PE) | fica |
| `trustno1-coder/VIctory-NVIDIA` | 0 | 200 lidos acima de cursor nulo (máx. #1299; `#1290` "Fix main's CI") | 0 | SIM fraco (critério 3; cursor de abertos ainda nulo) | fica (baseline de PRs) |
| `californiaAlmonds/OpenScarlett` | 0 | 1 acima de #60 (#66, testes de HDR) | 0 | NAO (só teste de contrato; sem defeito do catálogo no título/diff) | — |
| `fxceless/victory-fork` | 0 | 0 acima de #1513 (máx. #1509) | 0 | NAO | — |
| `holdmysocks/SixNG` | 0 (`identical`) | 0 | 0 | NAO | — |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 | 0 | NAO | — |
| `stellarr-studio/kelvaroq` | 0 | 0 | 0 | NAO | — |
| `Belobug/VIctory-Belobug-AMD` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (HTTP 404, décima primeira hora) | bloqueado |
| `avatarmanz/VIctory-7900XTX` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (HTTP 404, décima primeira hora) | bloqueado |

## Como o sinal foi decidido

**Sanidade do estado (passo 1).** `gh api repos/<o>/<r> --jq .default_branch` foi pedido nos 15. Treze
responderam com a branch já gravada, inclusive `master` (`holdmysocks/SixNG`) e `a-publier`
(`Belobug`, não lido nesta hora; o estado já aponta `a-publier`). `Belobug/VIctory-Belobug-AMD` e
`avatarmanz/VIctory-7900XTX` devolvem HTTP 404 em `repos/` pela décima primeira hora. Nenhum
`last_commit` nulo — não houve baseline. Nenhum `compare` devolveu `diverged` (3 `ahead`, 10
`identical`). O estado tem **15** entradas em `repos` no início e no fim; nenhuma foi removida, nada
foi acrescentado à mão.

**Critério 1 (defeito do catálogo por hash): não acionou.** Os 270 tokens hexadecimais de
`catalogo/defeitos.csv` foram conferidos contra os 132 hashes de commit da janela
(`xcl` 120, `TuxBox` 10, `Xbox2Windows` 2) e contra os 9 hashes de merge (`db3921dc`, `59e4be57`,
`10d27297`, `4cb3347d`, `19dc351b`, `72edad75`, `b90cc37e`, `b8355fa2`, `aec2e084`): **zero
interseções**, inclusive por prefixo de 7 caracteres. O `#201` do `DexNom` cita `45ed8e3` + `8d67fa5`
(restauração), e nenhum dos dois está no catálogo.

**Critério 2 (correção/regressão/reversão): acionou em `xcl`, `TuxBox` e `Xbox2Windows`.** `xcl` traz
três merges de ramos `fix/` (`#3157`, `#3159`, `#3085`) mais seis commits do recompilador cujo diff
rejeita ou adia um caminho (`2ca91276` recusa `s_barrier` multi-wave no DXIL direto, `50e85c5b`
recusa leitura de lane superior em wave64 gráfico dividido, `4d647491` preserva spill slots em onda
parcial). `TuxBox` mescla cinco PRs de correção com sintoma declarado (`#373` cita a issue #323:
leitura de heap liberado 11 minutos depois, device loss em 15–18 min). `Xbox2Windows` mescla `#3332`
consertando `#3331`.

**Critério 3 (PR aberto em área de pista):** a "tabela de pistas" continua ausente de
`docs/automacao/ANALISE_HORARIA.md`; as áreas vieram das seções D do catálogo e do corpo dos PRs.
Acionou em `Xbox2Windows` (`#3376` planos apresentados), `VItoriaBr` (`#133`/`#134`, compilador AMD),
`Marvisak` (≥100), `DexNom` (`#201`), `omonkeyz` (`#395`), `NVIDIA` (200).

**Critério 4 (atraso > 200): não acionou.** O maior atraso é `xcl`, com 120.

## Desempate

Três repositórios acionaram o critério 2 e o limite é 4. Entraram `xcl` (118 commits sem documento e
três merges com correção), `TuxBox` (11 commits e cinco merges, um deles com device loss documentado),
`Xbox2Windows` (dois commits, um merge e um PR aberto que continua a correção de tela preta) e
`VItoriaBr` (critério 3, com `#133` consertando um crash de compilador AMD — diff de 29 linhas,
verificável). `Marvisak`, `DexNom`, `omonkeyz` e `NVIDIA` mantêm o cursor parado por decisão; ficam
registrados com o que foi lido.

## Observações de coleta

- **`xcl`:** head `10d27297` às 16:59:48Z. O `compare` a partir de `3cde0cc3` devolve 120 commits;
  dois (`2dad32e6`, `abf7e177`, já documentados às 12:00) foram pulados por hash. Documentaram-se os
  40 primeiros não documentados na ordem do `compare` (`38c81423` → `4701a874`). O novo
  `commit_cursor` é `4701a874`; sobram **78** commits sem documento, e o próximo `compare` a partir
  do novo cursor re-inclui os dois hashes já vistos — a regra de pular hash documentado continua
  valendo.
- **`xcl`, ordem instável:** o merge `#3100` (`bb9a6b4b`, documentado às 16:00) traz o commit de
  conteúdo `2de72223`, que nesta lista aparece na posição 35 e **não** está entre os 40
  documentados — entra pelo documento do PR do ciclo anterior, não pela tabela de commits.
- **`TuxBox`:** head lido no `compare` `5de0e62b`; o PR `#404` foi mesclado às 17:12:17Z, **depois**
  do snapshot, e o head agora é `b8355fa2`. O conteúdo do `#404` foi lido no diff do PR (1 arquivo,
  +14/−16) e o merge consta da tabela de mesclados; nenhum commit da faixa ficou sem documento.
- **`Xbox2Windows`:** head `aec2e084`; os 2 commits da janela são o conteúdo (`539c3ad5`) e o merge
  (`aec2e084`). A hora é sobretudo de PRs abertos (3).
- **`VItoriaBr`:** head `15779f81`, parado desde 27/11; não houve commit nem merge na janela. O deep
  dive é só de PRs abertos.
- **Merges x commits:** em `xcl` a contagem bate traseiramente (3 merges, 3 commits de conteúdo
  `59cd71e6`, `61fdf424`, `971f857b`, todos dentro dos 120); em `TuxBox` os 5 merges correspondem a
  4 commits de conteúdo (`ab0751c8`, `a40e5a8c`/`661ef9cd`, `99d8569d`, `0ae06f52`) mais o `#404`,
  cujo único arquivo é de teste. Em `Xbox2Windows`, 1 merge = 1 commit de conteúdo.
- **Rate limit:** nenhum 403 nesta coleta (4.846 restantes no início, 4.777 no meio).
