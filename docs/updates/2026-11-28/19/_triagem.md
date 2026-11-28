# Triagem de 2026-11-28 19:00

Coleta de 2026-11-28T19:07Z a 19:3xZ (UTC real; 16:07–16:3x local, UTC−3). `last_run` de cada
repositório é o cursor de PRs mesclados no relógio do GitHub. A janela de commits é `commit_cursor`
(ou `last_commit`, quando não há cursor pendente) até o head da branch default lido nesta hora.

O estado tem **15 entradas** em `repos` (o procedimento lista 15; a mensagem da rotina fala em 14).
Nenhuma foi removida e nada foi acrescentado à mão.

| repo | commits atras | PRs abertos | PRs mesclados | sinal | prioridade |
|---|---|---|---|---|---|
| `ivall/xcl` | 45 no `compare` a partir de `f2c0d36f`, dos quais **35 já estão em `docs/updates/*/xcl.md`** e **10 sem documento** | 2 acima de #3198 (#3199, fechado sem merge durante a coleta; #3200; 50 lidos, máx. #3200 — #3202 chegou às 19:08Z) | 0 na janela | SIM (critério 2: `bba710e2` preserva a fronteira de bloco `getDimensions`/`TextureLoad` que a fusão de CFG desfazia — **segunda ocorrência da classe de D-24**; `2de1544d` estende a fronteira `s_setpc`/`gs_start` ao wave64 nativo — classe de D-19; `5f284cc2` faz as escritas da própria lista caírem num chunk esparso que a CPU alterou) | **1 - deep dive** |
| `R0Y00/Xbox2Windows` | 3 (`2e460b26`, `f81feb16`, merge `eeb52277`) | 6 acima de #3387 (#3388, #3389, #3391, #3393, #3394, #3395; 50 lidos; #3390/#3392 não estão na janela de abertos) | 1 na janela (#3357 `eeb52277`, 19:07:04Z) | SIM (critério 2: `s_bfm_b64` recusava **VCC** como destino — a máscara de destinos válidos aceita 106 e continua recusando o ímpar 107; o controle negativo do teste mudou de VCC para M0) | **2 - deep dive** |
| `trustno1-coder/VIctory-NVIDIA` | 70 (`93008750` → `32ecacfa`): 40 commits de conteúdo + 30 merges | 50 lidos acima de cursor nulo (#1265–#1409 no snapshot; #1409 criado 19:2xZ) | **30** na janela (#33–#132, todos mesclados entre 18:12Z e 18:22Z) | SIM (critério 2: os 30 PRs são consertos e desempenho com medição — `#81` biblioteca de pipeline re-salva em rajada, `#82` raios acertando estruturas não construídas, `#68` FSR 4 no lugar do upscaler próprio; e 4 `fix(...)` de script/CLI) | **3 - deep dive** |
| `Marvisak/VIctory` | 0 (`identical`) | 7 acima de #3775 (#3776, #3778–#3783; 50 lidos, máx. #3783 às 19:01Z) | 0 | SIM (critério 3: `#3780` corrige `IA_INDEX_BUFFER` anunciado para **todo** formato no `FORMAT_SUPPORT` — mesma máscara de `#3771`, documentado às 18:00; `#3776` dobra constante half conhecida; `#3783` aplica escala XMA no lugar) | **4 - deep dive** |
| `brcly/TuxBox` | 0 (`identical`) | 1 acima de #407 (#408, criado 18:49Z) | 0 | SIM fraco (critério 2/3: `#407` SCC de `s_andn2_b64` em `wave_cull`, `#408` lê recursos estruturados/tipados/imagem só em lanes dentro de EXEC; ambos abertos, sem commit) | fica (fila #408) |
| `fxceless/victory-fork` | 0 (`identical`) | 0 acima de #1519 (50 lidos, máx. #1519) | 0 | SIM fraco (critério 2: `#1506`/`#1507` compares de 64 bits e carry de `v_mad_u64_u32`; `#1518` barras de 24 px no tile-sort da NVIDIA 595.x) — todos criados **antes** do cursor #1519, já triados às 18:00 | fica (sem fila) |
| `DexNom/VIctory` | 0 (`identical`) | 0 acima de #201 (50 lidos, máx. #201) | 0 | SIM fraco (critério 2: `#199`–`#201` restauram `131944d`, `6493dd9`, `45ed8e3`+`8d67fa5`; documentados às 16:00) | fica (sem fila) |
| `uiliannn/VItoriaBr` | 0 (`identical`) | 0 acima de #134 (50 lidos, máx. #134) | 0 | SIM fraco (critério 3: `#133` "do not reuse texture size query (fixes AMD compiler crash)" e `#134` handle próprio para consulta de tamanho de buffer) — criados antes do cursor #134 | fica (sem fila) |
| `omonkeyz/xbox-emulator` | 0 (`identical`) | 0 acima de #395 (25 abertos, máx. #395) | 0 | SIM fraco (critério 3: `#395` validação do bloco de relocação PE) — anterior ao cursor | fica (sem fila) |
| `californiaAlmonds/OpenScarlett` | 0 (`identical`) | 1 acima de #60 (#66, já registrado) | 0 | NAO (teste de contrato de HDR) | - |
| `holdmysocks/SixNG` | 0 (`identical`) | 0 | 0 | NAO | - |
| `walkthestars/GTA6-Asset-Info` | 0 (`identical`) | 0 | 0 | NAO | - |
| `stellarr-studio/kelvaroq` | 0 (`identical`) | 0 | 0 | NAO | - |
| `Belobug/VIctory-Belobug-AMD` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (HTTP 404 em `repos/`, décima terceira hora) | bloqueado |
| `avatarmanz/VIctory-7900XTX` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (HTTP 404 em `repos/`, décima terceira hora) | bloqueado |

## Como o sinal foi decidido

**Sanidade do estado (passo 1).** `gh api repos/<o>/<r> --jq .default_branch` foi pedido nos 15.
Treze responderam com a branch já gravada, inclusive `master` (`holdmysocks/SixNG`) e `a-publier` não
pôde ser conferido porque o repositório inteiro devolve 404. Nenhuma branch divergiu do estado, então
nada foi corrigido antes do delta. Nenhum `last_commit` nulo — não houve baseline nesta execução.
Nenhum `compare` devolveu `diverged` entre cursor e head.

**Critério 1 (defeito do catálogo por hash): não acionou.** Os 262 tokens hexadecimais de
`catalogo/defeitos.csv` foram conferidos contra os 118 hashes das três janelas com commit
(`xcl` 45, `Xbox2Windows` 3, `NVIDIA` 70): **zero interseções**, inclusive por prefixo de 7 caracteres.

**Critério 2 (correção/regressão/reversão): acionou em `xcl`, `Xbox2Windows` e `NVIDIA`.** Em `xcl`
`bba710e2` é guarda de regressão da classe de D-24 e `2de1544d` estende o limite de `s_setpc`/`gs_start`
ao wave64 nativo (classe de D-19); em `Xbox2Windows` o merge `#3357` fecha uma recusa de destino no
tradutor direto de compute; em `NVIDIA` os 30 merges são, na maioria, consertos com número medido no
corpo do commit.

**Critério 3 (PR aberto em área de pista):** a "tabela de pistas" continua ausente de
`docs/automacao/ANALISE_HORARIA.md` (a única menção é a linha 233 do próprio procedimento, que a
referencia). As áreas vieram das seções D do catálogo e do corpo dos PRs. Acionou em `Marvisak`
(`#3780` na máscara de `FORMAT_SUPPORT`, `#3776`/`#3781`/`#3782` no tradutor), `TuxBox`
(`#407`/`#408`), `victory-fork`, `omonkeyz` e `VItoriaBr`.

**Critério 4 (atraso > 200): não acionou.** O maior atraso é `NVIDIA` com 70 commits, seguido de
`xcl` com 45 (10 sem documento).

## Desempate

Três repositórios acionaram o critério 2 com substância — `NVIDIA` (70 commits, 30 merges, 30 PRs de
conserto), `xcl` (10 commits, três deles conserto de fronteira) e `Xbox2Windows` (1 merge e 2 commits,
diff pequeno e inteiramente verificável) — e ocuparam as três primeiras vagas. A quarta foi para
`Marvisak`, que tem a maior leva de PRs novos da hora (7) em áreas do tradutor e da consulta de
capacidades. `TuxBox`, `victory-fork`, `DexNom`, `VItoriaBr` e `omonkeyz` mantêm o cursor parado por
decisão; ficam registrados com o que foi lido.

## Observações de coleta

- **`xcl`: o cursor gravado não é ancestral de tudo o que foi documentado.** `compare/f2c0d36f...head`
  devolve 45 commits, mas `compare/86a90eeb...f2c0d36f` devolve `diverged` (5 à frente, 3 atrás), então
  `86a90eeb` — documentado às 18:00 — **não** é ancestral do cursor. Efeito: o `compare` a partir do
  cursor volta a listar 35 commits já documentados. O número "38 restantes" gravado às 18:00 não se
  reproduz; a medição desta hora é 45 no `compare` e **10 sem documento**. Os 35 foram pulados **por
  hash** contra `docs/updates/*/xcl.md`.
- **`xcl`, três commits são o mesmo patch rebasado.** `93ea614b`, `f52da002` e `dc684f21` têm a mesma
  mensagem e o mesmo `benches/runtime.rs` +2/-4 de `15b1845a`; os quatro são o mesmo conserto de
  `E0451`. Contam como um achado, não quatro.
- **`Xbox2Windows`: merge durante a coleta.** `#3357` fechou às 19:07:04Z, depois do primeiro snapshot;
  a faixa foi refeita com `compare/f2972840...eeb52277` (3 commits) e o conteúdo lido no diff do PR.
- **`NVIDIA`: os 30 merges chegaram em dois lotes.** 24 entre 18:12:15Z e 18:12:20Z e 6 entre 18:20:42Z
  e 18:22:18Z. A ordem de merge não acompanha o número do PR: `#124`/`#126`/`#127`/`#132` entraram antes
  de `#35`/`#52`, e `#68` (o maior número do lote) entrou no primeiro grupo. Confirma a regra de não
  usar `last_pr_number` para mesclados.
- **`NVIDIA`: dois commits da janela são "Merge main: ..."**, isto é, a `main` trazida para dentro do
  ramo (`0eb34b84`, `76e3c697`). O diff deles contra o primeiro pai é o que a `main` já tinha (FSR 3,
  `save_share`, DIAG do Agility SDK) e **não** é trabalho novo do ramo.
- **`Marvisak`: o teto de PRs abertos segue no piso.** 50 lidos, menor #3729 (12:22Z) ainda acima do
  cursor anterior #3521; o total real é ≥50 e o cursor não prova cobertura dos antigos. Nesta hora só
  os 8 acima de #3775 foram lidos.
- **`NVIDIA`: lado de PRs abertos sem piso.** O cursor era nulo; foram lidos 50 (#1265–#1409). Abaixo de
  #1265 não há leitura nesta execução e isso vai para o `backlog`.
- **Merges x commits:** `xcl` 0 merges e 10 commits de conteúdo; `Xbox2Windows` 1 merge para 3 commits
  (2 de conteúdo + o merge); `NVIDIA` 30 merges para 40 commits de conteúdo — a diferença são os
  commits que entraram por `Merge main` e os que o mantenedor comitou direto na `main`.
- **Rate limit:** nenhum 403 (4.833 restantes no início, 4.6xx no fim).
