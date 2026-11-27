# Belobug/VIctory-Belobug-AMD — atualização de 2026-11-27 00:00

## Resumo

Nenhum commit novo na `main`, mas vinte PRs mesclados — **todos em `a-publier`**, que é a branch default
real do repositório e não a `main` que `state/watch.json` acompanha. As vinte fusões aconteceram em uma
rajada de dois segundos (`2026-11-26T22:30:22Z` a `22:30:23Z`), então o que a janela registra é um lote,
não vinte integrações espalhadas. O conteúdo é manutenção: robustez de GPU (`#354` partes de sync do port
2, `#351` diagnóstico de heap indexado recusado), CLI e CI (`#326`, `#330`, `#332`, `#340`, `#286`),
diagnóstico (`#316`, `#348`), um contorno para crash do compilador AMD (`#324`) e testes que falhavam em
hardware real (`#288`, `#290`, `#328`). Cinquenta PRs abertos novos na janela da API — de **239 abertos no
total**, o que excede em muito a janela de 50.

## Commits novos (0)

Nenhum. O topo da `main` continua `4f62c2a5` ("The ambient light follows the sky by default", commit de
`2026-11-25T20:56:22Z`), idêntico ao cursor. Ver a seção Confiança para a divergência de branch.

## Pull requests abertos (50)

Os 50 mais recentes devolvidos por `gh pr list --state open --limit 50` (números `387`–`456`). O
repositório tem **239 PRs abertos**, de `#3` a `#456`: os 189 restantes não aparecem nesta janela porque a
listagem é ordenada do mais novo para o mais antigo.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| `456` | warm: create the pipelines folder on a fresh cache | `victory warm` falha numa cache que o título nunca escreveu; cria a pasta `pipelines`. +37/-2 | Não |
| `454` | fix(gpu): close upload-ahead races and defer new pool flips | Reserva a entrada em voo antes do staging, não marca textura como enviada quando a cópia falha e passa `pool::begun_before` aos flips para respeitar `VICTORY_SHOW_NEW_LATER`. +187/-47 | Não |
| `453` | Improve Radeon rendering and support writable placed textures | SRV e UAV de placed texture compartilham a textura de base; teste de GPU escreve pelo UAV e confere 4.096 pixels. Integra correções de warm-up e conversão de textura de `avatarmanz/VIctory-7900XTX` #333/#335/#351. +995/-189 | Não |
| `451` | GPU: pool and recycle shadow::exact() buffers with GPU quarantine (#346) | `shadow::exact()` segurava um `Mutex` global, varria o mapa com `retain` a cada erro acima de 256 entradas e criava `CommittedResource` sob o lock; com ~716 erros por frame as threads de lista serializavam. +642/-50 | Não (D-05 é exaustão de slots de descritor; este é turnover de ring buffer) |
| `450` | Tests: an address freed during a change of mapping is not given to another test | Estabiliza os testes de `watch` em paralelo: endereço liberado durante troca de mapeamento não vai para outro teste. +23/-4 | Não |
| `449` | Fix black world rendering by legalizing host barrier states | Sanitiza os estados antes e depois das barreiras de textura com `legal_state` e pula transições que viram no-op. +7/-2 | Não |
| `448` | Simple build batch script - make-vi.bat | Script de build `make-vi.bat [DEBUG/RELEASE/TESTS/DEF]`. +89/-0 | Não |
| `446` | Tests: the watch tests' random failures came from a view another test freed mid-change | Causa dos testes instáveis: `many_askers_tests` mapeia e desmapeia uma view de uma página 300 vezes e libera o endereço durante a troca. +39/-3 | Não |
| `444` | 12 GB cards keep title buffers of 1 MiB and more on the card (1024 MiB) | Separa o tier: orçamento acima de 9 GiB e até 12 GiB recebe 1 MiB / 1024 MiB em `VICTORY_CARD_BUFFERS_MIN_MB`/`_MB`; 8 GB segue 16/512. +20/-7 | **D-13** (consumo de memória) — alivia pressão de VRAM, não corrige o defeito |
| `443` | List threads no longer queue behind exact and loose copies being made; same page mappings are not redone | Duas correções de lock nas threads de lista e um par de chamadas de mapeamento a menos na thread principal. +119/-25 | Não |
| `442` | Shaders: console threads of 2-D and 3-D groups come from the wave counter (#216, a-publier) | Na `a-publier`, programa de compute com grupo 2-D/3-D maior que uma onda do host ainda tira a thread do primeiro id plano da onda; porta o #236 da `main`. +111/-11 | Não |
| `441` | Shaders: precompile answers for its own shader, not by the shared miss count (#291) | O braço `None` de `precompile` não é alcançável na `a-publier`; o PR corrige a atribuição de resposta ao shader certo. +70/-5 | Não |
| `440` | Memory Acceleration: Eliminate Quadratic GPU Snapshot Processing | Troca subtração repetida de intervalos por varredura ordenada na preparação de faixas de snapshot. +99/-18 | Não |
| `437` | FPS Boost: Accelerate GPU Scheduling and Eliminate Quadratic Dependency Scans | Indexa níveis de dependência sobrepostos por range maximum com compressão de coordenadas. +139/-2 | Não |
| `436` | Report per-shader failure reasons during translate --warm | O erro de escrita da cache some da saída do warm-up; passa a reportar a razão por shader. +81/-17 | Não |
| `434` | GPU uploads: plan long runs and scatter chunks in one scan | Reduz travessia de faixas no lado da CPU na preparação de upload. +108/-23 | Não |
| `433` | Shaders & GPU: support HwStage::Gs geometry shader translation and pipelines | Tradução e PSO para `HwStage::Gs` (ES-GS mesclado), cobrindo os 195 `GS_VoxelizeGeneric_*` restantes — 25.889 de 25.889 shaders do arquivo do título. O próprio PR diz que o caminho de render atual não despacha esses shaders. +1835/-125 | Não |
| `432` | Add opt-in frame-rate samples for benchmarks | `VICTORY_LOG_FPS=1` emite amostra `frames`/`elapsed_s`/`fps` cerca de uma vez por segundo. +115/-21 | Não |
| `431` | Report unsupported pixel-shader barycentrics on pipeline refusal | Reporta a capacidade de barycentrics do dispositivo e, em `E_INVALIDARG` de pipeline mesh/vertex, acrescenta a razão quando o DXIL do pixel exige barycentrics. +80/-5 | Não |
| `430` | Distinguish pipeline-library lookups from store failures | Separa erro de lookup de erro de store, grava o HRESULT real do store e mede criação de pipeline separada do lookup. +101/-15 | Não |
| `428` | Relay: the copy-queue fetch never runs beside writes to the copies it reads (RDNA 2 resets, #376) | Com `VICTORY_RELAY=2` e `VICTORY_RELAY_COPY=1`, a busca na fila de cópia não era ordenada contra escritas posteriores nas mesmas cópias; `host/fetchguard.rs` passa a ordenar. +305/-8 | Não (D-07 é uso-após-liberar; aqui é corrida de fila de cópia) |
| `427` | Texture store: serialize resident cache admission and byte accounting | Serializa admissão na cache residente e contagem de bytes, que podia passar de 4 GiB ou cobrar o mesmo blob duas vezes. +76/-14 | Não |
| `426` | Audio: raise XMA decoder thread priority to prevent CPU-load audio crackle | `THREAD_PRIORITY_HIGHEST` na thread "victory XMA decoder", que acorda a cada 2 ms e era faminta por threads de descompressão acima do normal. +11/-0 | Não |
| `424` | GPU: track direct buffer states in HostList and restore COPY_DEST to COMMON in place_copy (#350) | Rastreia estado dos buffers diretos por lista e restaura `COPY_DEST` para `COMMON` no `place_copy`. +837/-27 | Não |
| `423` | Shaders: reject DXBC part offsets into container metadata | Rejeita contêiner DXBC cujo offset de parte aponta para o cabeçalho ou a tabela de offsets. +38/-0 | Não |
| `422` | Shaders: a buffer's size query has a handle of its own (AMD's compiler crashes on RDNA4) | Na RX 9060 XT (amdxc64 32.0.32029.2010), `the_record_count_of_a_structured_buffer_read_by_index` morre com `STATUS_ACCESS_VIOLATION` em `amdxc64.dll` offset `0x861b9f`, na criação do PSO; WARP passa. +36/-4 | **D-06** (compilador AMD que crasha) — mesma família do `#324`, ainda aberta |
| `421` | Linux/Proton: pipeline library, worker stacks, watchdog, Wine HLSL fixes (black screens, dark interiors, compare pass) | Quatro commits para Proton: biblioteca de pipeline nula, pilhas de 64 MiB para workers, watchdog opcional e correções de HLSL do Wine. +620/-55 | Não |
| `420` | Input: read PS5 DualSense pads directly over HID | Lê DualSense/Edge por HID, sem Steam Input ou DS4Windows. +643/-14 | Não |
| `419` | Handle Windows close requests through title suspension | `WM_CLOSE` direto na janela principal passa pelo caminho de suspensão do título em vez de destruir a janela com o processo vivo. +92/-4 | Não |
| `418` | Tests: identify native compute PSO crashes before HRESULT handling | Diagnóstico só de teste em volta da criação de PSO de compute nativo; reproduzido no PR #360. +21/-1 | Não |
| `415` | Lists: the per-draw maps keyed by address use AddressHash, not SipHash | Cinco mapas consultados a cada draw (estados de textura, gêmeos de profundidade, first-use, slots de root SRV/UAV) passam a usar `AddressHash`. +5/-8 | Não |
| `414` | Draws look at the early texture pass's frame before swapping it | `early_textures` fazia `FRAME.swap(frame)` a cada draw em cada thread; passa a ler antes e só trocar quando o frame difere. +4/-1 | Não |
| `413` | Title memory reads zero only the bytes no run covers, not the whole buffer first | `read_there` zerava o buffer inteiro antes de ler as faixas; passa a zerar só antes, entre e depois delas. +30/-1 | Não |
| `412` | Device-removed report: "nothing of ours was running" only when every submission had finished | O relatório dizia que nada rodava na GPU quando a thread não estava esperando, mesmo com listas `late` em voo (`VICTORY_RELAY=2` é padrão). +96/-8 | Não |
| `410` | Pipelines: one still being made waits less after its first frame | `VICTORY_PIPELINE_WAIT_AGAIN_MS=<n>` limita a espera por pipeline a partir do segundo frame; um compile de 3,4 s custa ~7 frames de até 500 ms. +55/-1 | Não |
| `409` | GPU: placed texture sizes leave out the filler words; the twin count is diagnostics-only and capped | Três lugares ainda usavam as 14 palavras cruas da descrição; `PLACED_RAW` crescia sem poda e só alimentava uma linha de log de diagnóstico. +97/-37 | Não |
| `407` | XMem: say how much of the commit limit the physical page pool needs, and how much it has committed | O pool de ~6,1 GiB é `SEC_RESERVE` comprometido página a página e cobrado do commit do sistema, não dos bytes privados; passa a avisar antes e reportar o comprometido. +82/-2 | **D-13** (consumo de memória) — instrumenta, não corrige |
| `406` | Diagnostics: count texture staging buffers detiled while an upload from them is in the air | Conta detiles em staging buffer com cópia ainda não executada; o defeito se auto-cura (`dirty`) e não derruba o dispositivo, então o PR só mede. +52/-3 | Não |
| `404` | Texture store: switches for the most its queue and memory hold | `VICTORY_TEXTURE_STORE_QUEUE_MB` e teto de memória para `KEPT`/fila sem limite do texture store. +75/-8 | Não |
| `403` | play.ps1: -Record, -Warm and -Shaders, so warm has pipelines and the run's switches | `play.ps1` nunca setava `VICTORY_PIPELINE_BOOK` e removia todo `VICTORY_*` herdado; `warm` então dizia "0 pipelines known". +49/-4 | Não |
| `402` | DirectStorage: worker count and priority switches | `VICTORY_DSTORAGE_WORKERS` (1..64) e `VICTORY_DSTORAGE_PRIORITY`; o scheduler abria uma thread por processador lógico. +34/-3 | Não |
| `401` | GPU: a grown up-queue buffer goes through the bury lag instead of being dropped at once | Buffer crescido do up-queue era largado na hora, único caminho que pulava `retire_resource`; as perdas de dispositivo de 2026-11-25 foram faltas de página em memória que nada mais nomeava. +31/-3 | **D-07** (uso-após-liberar na GPU) — fecha o último caminho que pulava o adiamento, não o defeito catalogado |
| `398` | Run: a folder given as the executable is said so; README clones this repository (#395) | Mensagem clara quando o caminho é pasta; README passa a clonar o repositório. +9/-2 | Não |
| `397` | RT: add opt-in AMD RDNA 2+ hardware BVH4 ray-node intersection (VICTORY_HW_BVH=1) | Traversal de BVH por intrínseco `IAmdExtD3DShaderIntrinsics::IntersectBvhNode` (opcode 0x22), adaptado de `Marvisak/VIctory#3294`. Desligado por padrão. +1121/-17 | Não |
| `396` | GPU: keep rechecking heap textures known to change in place | Textura de heap que já mudou sem notificação de descritor mantém o intervalo final de 400 frames; as sem mudança observada saem após a quinta checagem. +112/-48 | Não |
| `394` | Memory switches for 32 GB PCs (all off by default) | Cinco chaves para PCs de 32 GB cujo commit acaba antes da RAM (#333, #297, #298): espera só de texturas, linha de cache de textura, staging em BAR, pressão só física, linha BAR do Windows. +165/-18 | **D-13** (consumo de memória) — contorna, não corrige |
| `393` | Formats: reuse row runs when copying tiled texture mips | Padrão de swizzle imutável guarda offsets de corridas de coluna; ambos os sentidos tratam corrida final parcial e padding de linha. +206/-13 | Não |
| `389` | Add a QHD launcher for the GeForce RTX 4060 Laptop | Preset que seleciona a 4060 Laptop por nome pedindo 2560×1440 com perfil Series S e `-Safe`, vindo do fork `VIctory-RTX4060-Laptop-BR`. +46/-6 | Não |
| `388` | Replay: --retranslate translates for the host it is told (#302) | `replay --retranslate` fixava host RDNA de 64 lanes ignorando a run; passa a ler o host pedido. +139/-7 | Não |
| `387` | pipeline: keep stale warm-bad.txt entries in runtime until warm-done.txt confirms them | Seguimento do `#294`: entradas de `warm-bad.txt` deixavam de valer assim que o fingerprint mudava (atualização de driver, bump de `TRANSLATOR_VERSION`); passam a valer até `warm-done.txt` confirmar. +172/-47 | Não |

## Pull requests mesclados (20)

Todos com `baseRefName = a-publier`. Nenhum merge commit aparece na `main`; ver Confiança.

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| `354` | victory-gpu: keep port 2 sync parts disjoint and fix fallback counters | `d3cf70dc` | +228/-15 em 2 arquivos. Corta ou descarta corridas de marca do port 2 que sobrepõem páginas de checagem de fallback, mantendo as partes de `sync_heaps` disjuntas como `host::Copied` exige; rastreia slots tomados cobertos por páginas de fallback (`was_taken_in_check`) para o fallback não reportar marca já tomada como perdida. Fecha #338 |
| `351` | GPU: say when the D3D12 runtime refuses directly indexed heaps, and log the title's refused root signatures (#168, port of #317) | `044eda92` | +187/-17 em 3 arquivos. Na criação do dispositivo, pergunta ao serializador do runtime se ele aceita `CBV_SRV_UAV_HEAP_DIRECTLY_INDEXED \| SAMPLER_HEAP_DIRECTLY_INDEXED`; se recusar, emite **um** erro explicando que todo pipeline do host será recusado (runime in-box do Windows 10) e que só uma versão mais nova (Agility SDK) aceita. Passa também a logar as root signatures recusadas do título |
| `348` | Distinguish local GPU usage, OS budget and dedicated capacity in the HUD | `7d5c7a8c` | +123/-88 em 2 arquivos. A linha de VRAM mostrava `0.0/6.7 GB` misturando uso local com orçamento do processo, e uma consulta falha zerava os três valores. Passa a duas linhas: uso local + orçamento do SO, e capacidade dedicada + uso não-local; leitura ausente vira `N/A`, zero medido continua `0.0`, rótulos em GiB |
| `347` | Document community GPU results alongside the 7900 XTX benchmark | `4deb42b8` | +15/-2 em `README`. Rotula máquina, cena e RT dos números existentes, identifica os 10,5 GB como uso observado e liga os relatos da comunidade (#313 e outros) |
| `340` | victory config: a policy violation fails the command (#301) | `77bb0519` | +31/-7. `victory config` imprimia `POLICY VIOLATION` e saía com 0; a checagem passa por `check_policy`, escreve cada violação em stderr e devolve erro, então o comando sai 1. Valores resolvidos continuam em stdout |
| `336` | FidelityFX: -Fsr4K takes FSR 3.1 where the SDK offers no 4.1.2 (#323) | `4dc700c9` | +80/-18 em 5 arquivos. `select_version` exigia provider `4.1.2` e falhava; com isso `-Fsr4K` não iniciava em placa sem FSR 4 no SDK da AMD. Passa a cair para FSR 3.1 |
| `332` | Translate: --warm uses every logical core by default, as --jobs says | `038c507c` | +20/-5. `translate --warm` usava 4 threads sem `--jobs`, contra a ajuda do flag; os três caminhos passam a tirar a contagem de `worker_threads`, com mínimo 1. `victory warm` (dois terços dos núcleos) não muda |
| `330` | ci: make Clippy pass on a-publier (default and --all-features) (#315) | `392766f8` | +3/-8 em 4 arquivos. Anota os dois `std::mem::transmute` de `descriptor_sites.rs` e demais avisos, com Rust 1.98.1 |
| `328` | Make the depth-bounds fixture work on WARP | `8b75af9f` | +16/-9. Habilita comparação de profundidade always-pass com escrita desligada para o depth bounds decidir visibilidade; o WARP reportava suporte e ignorava os limites. Só teste |
| `327` | Default to light DRED only for diagnostic launches | `52326642` | +8/-1. `VICTORY_D3D12_DRED=light` como padrão em `-Build diag`; `release` fica sem DRED. README explica o requisito da feature `diagnostics` |
| `326` | Reject zero worker counts in the CLI | `61b11340` | +53/-3. `--jobs 0` vira erro de uso no parse, cobrindo `translate`, `translate --warm`, `shaders` e `warm` |
| `325` | Panel: a core's CPU counters that step back read as idle, not a panic | `b7f6420a` | +32/-3. `min > max` em `panel.rs:409` derrubava a thread `victory-submit` quando um contador de CPU andava para trás; passa a ler como ocioso |
| `324` | Shaders: a size query is not reused (AMD's compiler crashes on 29 compute pipelines on RDNA4) | `4eb6cfdf` | +30/-1 em 2 arquivos. Os 29 erros "the driver's compiler crashed making a pipeline" do #298 na RX 9070 são todos `ped_*:CS_RaytracingResolve` (13 variantes de material), todos no mesmo ponto de `amdxc64.dll` 32.0.32029.2010. O conserto é **não reusar** a consulta de tamanho; reproduzível com `VICTORY_TRANSLATE_ONLY=<hash> victory translate --pipelines` |
| `316` | profile: report late and expired profiling windows accurately | `751618c7` | +191/-2. Janela que começa atrasada passa a dizer que cobre só o resto; janela expirada deixa de ser silenciosa. O agendamento em si não muda |
| `303` | Report malformed host-wave overrides during translation | `26d4073b` | +96/-21. `VICTORY_HOST_WAVE`/`VICTORY_HOST_MAX_WAVE` resolvidos antes de despachar os dois modos de `translate`, com trim e relato de cada valor não parseável (inclusive não-UTF-8); `0` explícito continua significando largura desconhecida |
| `295` | Input: add opt-in GTA V PC-style keyboard and mouse mapping (VICTORY_KBM_GTA) | `d18b9aae` | +575/-13. Esquema de teclado e mouse no estilo GTA V PC para o gamepad do stand-in de GameInput, atrás de `VICTORY_KBM_GTA=1`, desligado por padrão |
| `294` | warm: scope warm-bad.txt by fingerprint, honor VICTORY_WARM_BAD=0 and --again (#280) | `70a2d5d8` | +353/-28 em 5 arquivos. `warm-bad.txt` passa a ter entradas escopadas por fingerprint, compatível com binários antigos e novos, em `victory warm` e na construção de pipeline do runtime |
| `290` | Shader tests: skip WaveSize(32) pipelines on 64-lane-only hosts | `479bb254` | +38/-5. Quatro testes de hardware criavam pipeline `WaveSize(32)` e falhavam com `E_INVALIDARG` em GCN de 64 lanes (Polaris); passam a ser pulados |
| `288` | Test: views_are_made_beside_the_heap_and_copied_in fails on AMD (RDNA 2) | `b2e0d2af` | +26/-3. Porte do `avatarmanz/VIctory-7900XTX#298`: o teste comparava os 32 bytes do slot com os que `staged()` fez ao lado, e os bytes de descritor são do driver — passa na CI porque a CI roda em WARP |
| `286` | ci: run checks on pushes to the default branch | `2d63b090` | +1/-1. `a-publier` entra no filtro de push ao lado de `main`; o histórico mostrava zero execuções de push em `a-publier` e sete em `main` |

## Cruzamento com o catálogo

- **Corrige D-xx?** Um contato real, nenhuma correção fechada.
  - **D-06** ("Compilador AMD: pipelines que nunca terminam ou crasham, 16 GB em dois minutos", sem hash
    de correção): `#324` (`4eb6cfdf`) elimina **uma instância concreta** — 29 pipelines de compute,
    todas `ped_*:CS_RaytracingResolve`, que crashavam `amdxc64.dll` 32.0.32029.2010 no mesmo ponto —
    deixando de reusar a consulta de tamanho. O `#422`, ainda aberto, mostra a mesma classe de crash em
    outro caminho (consulta de tamanho de buffer, `STATUS_ACCESS_VIOLATION` em `0x861b9f`). É **redução de
    escopo do D-06**, não fechamento: o D-06 continua aberto.
  - **D-13** (consumo de memória, 22 GB de RSS): `#444`, `#394` e `#407`, todos abertos, atacam o mesmo
    sintoma por VRAM, por switches de 32 GB e por instrumentação do commit do pool de páginas. Nenhum
    está mesclado. **Não corrige.**
  - **D-07** (uso-após-liberar na GPU, corrigido em `a873b24`): `#401` (aberto) fecha o último caminho que
    pulava o adiamento de `retire_resource`, e o corpo liga explicitamente as perdas de 2026-11-25 a faltas
    de página em memória não nomeada. É endurecimento, não reabertura. **Não corrige.**
  - **D-05** (exaustão de slots dinâmicos de descritor): `#351` (`044eda92`) só **diagnostica** a recusa de
    heaps indexados direto pelo runtime; `#451` (aberto) mexe em buffers de `shadow::exact`, não em slots
    de descritor. **Não corrige.**
- **Abre defeito novo?** Nenhum commit da janela introduz regressão demonstrável por diff. Três
  observações ficam registradas:
  - `#324` documenta que o compilador AMD crasha em **29 pipelines** que o runtime simplesmente deixa de
    fora; o efeito visível (shaders de ray tracing resolvido ausentes) não está no catálogo.
  - O lote de 20 merges em dois segundos indica integração em massa sem execução de CI por PR; o `#330` e
    o `#286` desta mesma janela são justamente consertos de CI na `a-publier`, o que sugere que a branch
    rodava menos verificação que a `main`.
  - `#288` e `#290` mostram testes que **passavam na CI e falhavam em hardware real** (WARP versus RDNA 2 e
    GCN 64 lanes). Não é defeito do produto, mas é a mesma classe de ponto cego de CI.

## Confiança

Verificado: `repos/Belobug/VIctory-Belobug-AMD/commits?sha=main&per_page=100` devolve `4f62c2a5` como
topo, igual ao cursor, de `2026-11-25T20:56:22Z` — nenhum commit novo na `main`. Os 20 PRs mesclados em
`pulls?state=merged&search=merged:>=2026-11-26T04:15:00Z`, cada um aberto em `prs view` com
`baseRefName`, `mergeCommit`, arquivos, contagens e corpo; **os 20 têm `baseRefName = a-publier`**. Os 50
PRs abertos mais recentes, todos acima do cursor `223`. `repos/<repo>` confirma `default_branch =
a-publier` e `pushed_at = 2026-11-26T22:30:21Z`; `compare/main...a-publier` devolve `diverged`,
`ahead_by = 638`, `behind_by = 550`, base de merge `8e79c80e`.

**Divergências registradas (não resolvidas aqui):**

1. **A branch default do repositório é `a-publier`, não `main`.** A tabela do procedimento e
   `state/watch.json` dizem `main`. A `main` parou em `4f62c2a5` (2026-11-25 20:56) e as 20 fusões desta
   hora foram todas para `a-publier`. Enquanto o estado acompanhar `main`, **todo o trabalho que entra em
   `a-publier` fica invisível para o cursor de commits** — inclusive 638 commits desde a base de merge
   `8e79c80e`. Não troquei o cursor de branch por conta própria: isso mudaria a semântica de
   `last_commit` e precisa de decisão explícita. Registro aqui e em `state/last-run.md`.
2. **A janela de PR aberto não cobre o repositório.** São 239 PRs abertos, de `#3` a `#456`; a listagem
   de 50 traz só os mais novos. Os 189 mais antigos nunca foram documentados e o cursor por número
   (`last_pr_number`) não consegue expressá-los: qualquer valor acima de 456 os marca como vistos sem
   tê-los visto. Não inventei linha para eles.

Não verificado: a contagem de PRs mesclados (20) contra commits na janela — não há commits na `main`,
e os merges estão em `a-publier`, cuja lista de commits não foi percorrida; os `merge_commit_sha` dos 20
(`d3cf70dc`, `044eda92`, `7d5c7a8c`, `4deb42b8`, `77bb0519`, `4dc700c9`, `038c507c`, `392766f8`,
`8b75af9f`, `52326642`, `61b11340`, `b7f6420a`, `4eb6cfdf`, `751618c7`, `26d4073b`, `d18b9aae`, `70a2d5d8`,
`479bb254`, `b2e0d2af`, `2d63b090`) foram lidos do PR, não conferidos na história de `a-publier`. O
mecanismo de cada merge vem do corpo do PR e da lista de arquivos; não abri `commits/<sha>` de nenhum
deles. Não executei build nem teste. Nenhum teste foi executado aqui; os 13 repositórios monitorados são
somente leitura.
