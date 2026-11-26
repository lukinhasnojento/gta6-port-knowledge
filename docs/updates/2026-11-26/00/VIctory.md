# Marvisak/VIctory — atualização de 2026-11-26 00:00

## Resumo

Nenhum commit novo na `main`: o topo continua em `63bcca43c6a00a758510398fa43096ce7e4f7df0`, o mesmo
cursor gravado pela execução anterior. O que mudou foi a fila de revisão — 24 pull requests abertos com
número acima de 2978, quase todos correções de robustez e de fidelidade de tradução shader, e nenhum PR
mesclado na janela.

## Commits novos (0)

Nada a documentar: `gh api repos/Marvisak/VIctory/commits?sha=main&per_page=100` devolve `63bcca43` como
primeiro elemento, que é exatamente o `last_commit` do estado.

## Pull requests abertos (24)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 2979 | GPU: validate command list Close and Reset state | Rastreia o estado de gravação da command list do convidado em `crates/victory-gpu/src/list.rs` (+56/-7) e devolve `E_FAIL` para `Close` repetido e para `Reset` durante a gravação, antes de tocar o estado do host ou descartar uploads. | Não. O catálogo de VIctory não tem item de ciclo de vida de command list. |
| 2980 | GPU: a panic while making a host pipeline refuses it instead of ending the worker | Envolve a criação de pipeline do host em `pipeline/make.rs` (+32/-1): captura o panic, registra shaders e mensagem, marca o pipeline como recusado e acorda os waiters, para o worker não morrer com a contagem de fila inflada. | Parcialmente: é o segundo item de `Belobug/VIctory-Belobug-AMD#217`, não um D-xx do catálogo. `Belobug-AMD D-06` (pipelines que nunca terminam ou crasham no compilador AMD) descreve sintoma vizinho, causa diferente. |
| 2981 | HLE/DirectStorage: reuse per-worker DecompressorOxide and scratch buffers | `WorkerScratch` por thread em `dstorage::work()`: `DecompressorOxide` reinitado por stream e buffers `source`/`intermediate` reaproveitados, com corte em 8 MiB (`SCRATCH_RETAIN_CAP`) para o RSS ocioso não crescer. | Não. Ganho de alocação, não corretude. |
| 2982 | Loader/Prepare: zero-alloc block-filtered DIR64 relocation clearing for TLS callbacks | Troca `dir64_relocations()` (decodifica a tabela `.reloc` inteira em `Vec`) por `clear_dir64_relocations(bytes, info, target_rva)`, que filtra por bloco de página e compara `(DIR64 << 12) | off` em `prepare.rs` (+231/-25). Bench no PR: 600,24 ms → 0,60 ms. | Não. Opera no mesmo caminho de `D-04` (limpeza do ponteiro de callbacks TLS), mas `D-04` já está corrigido em `25a3e95`; aqui é desempenho. |
| 2983 | Shaders/Emit: clear D3D12 filter reduction bits (0x180) before setting comparison bit (0x80) | `emit::emit` faz `s.filter = (s.filter & !0x180) | 0x80` em `emit/mod.rs` (+106/-1), em vez de só ligar `0x80`. Sem isso, sampler imediato com redução Minimum virava `0x180` (Maximum) em vez de `0x080` (Comparison). | Não. Defeito novo, encontrado no próprio PR (com controle negativo que falhava com 384 em vez de 128). |
| 2984 | GPU: implement ClearState on command lists | Registra `clear_state` no slot 11 (`0x58`) de `METHODS` e implementa `State::clear_state`, que antes caía no stub genérico e deixava `State.pipeline`, root args, heaps, alvos e `Fixed` de replay obsoletos (`draw.rs` +246, `list.rs` +44). | Não. Item novo do catálogo de ciclo de vida de lista. |
| 2985 | GPU: a tracked texture write detiles and copies only the rows it touches (#1135) | `Layout::touched_rows`/`detile_rows` em `victory-formats/src/texture.rs` (+217) e uso em `victory-gpu/src/texture.rs`: uma página de 64 KiB escrita no mip 0 de uma BC7 4K deixa de custar os 16 MiB do mip inteiro. | Não. Desempenho; retoma o patch #1135 de @talakasse. |
| 2986 | GPU: use one warm-up key for reordered graphics shader stages | Normaliza a ordem dos dois estágios gráficos ao ler o stream (`pipeline.rs` +50), para o pré-build e a criação pelo convidado nomearem a mesma pipeline quando o stream traz pixel antes de vertex/mesh. | Não. Item novo (chave de warm-up divergente). |
| 2987 | HLE: retain DirectStorage file references across queued reads | `QueuedRequest` não-`Copy` com `Option<NonNull<Instance<File>>>`: `add_ref_instance` no enqueue e `release` no `Drop`, fechando o use-after-free em que o título liberava a última referência do arquivo com a leitura ainda na fila. | Não diretamente. É um use-after-free, mas em HLE de arquivo, não o `D-07` (copy-back sobrescrevendo escrita da CPU). |
| 2988 | Shaders: lower v_trig_preop_f64 argument reduction | `lower/trig_preop.rs` (+230): janela de 53 bits de 2/π da tabela de 1184 bits da RDNA2, escala de entrada grande e empacotamento dos dois words binary64. Seis formas sintéticas que hoje são recusadas passam a traduzir. | Não. Cobertura nova de instrução. |
| 2989 | Loader: verify OpenThread owner PID before reading thread stack | `thread_in_process()` via `GetProcessIdOfThread` logo após `OpenThread` em `stacks.rs` (+38), para um TID reciclado não fazer o dump suspender thread de outro processo e desreferenciar TEB alheio. | Não. |
| 2990 | HLE: skip junctions in the persistent-storage quota count | `used_bytes` passa a andar com pilha explícita e `file_type()`, pulando links: junction para o próprio pai não estoura mais a pilha e bytes fora da pasta do título não entram na cota (`storage.rs` +53). | Não é D-xx; cita `VIC-BT-040` de `#1249`. |
| 2991 | Loader: clear XMM0 in ReturnZero thunk stubs for floating-point imports | Trampolim `global_asm!` com shadow space, alinhamento de 16 e `.seh_*` que faz `xorps xmm0, xmm0` antes de retornar, para import ausente com retorno `f32`/`f64` devolver `0.0` e não lixo de XMM0 (`thunks.rs` +97). | Não. |
| 2992 | Build and test the portable crates on non-Windows hosts | Adiciona `cfg(windows)` que faltavam em `victory-shader`, `victory-gpu`, `victory-hle`, `victory-loader` e `victory-pe` (10 arquivos), para o workspace portátil compilar em macOS/Linux. | Não. `D-17`, `D-18` e `D-19` foram quebras de build, já corrigidas; este é portabilidade. |
| 2993 | GPU: host copies of title buffers in video memory, under a DXGI budget (opt-in) | `VICTORY_VRAM_COPIES=1` (ou `2`) mantém a cópia do host também em upload heap mapeável pela BAR quando `GPUUploadHeapSupported`, com o lado da CPU sempre em memória de sistema (`vram.rs` +577, `shadow.rs` +670). Desligado por padrão. | Não. Desempenho de memória. |
| 2994 | GPU: keep v0 as the leading vertex in CPU-expanded rectangles | Os dois triângulos da expansão de retângulo em CPU começam em v0, mantêm o winding e compartilham a diagonal por v0 (`primitive.rs` +43), para um flat não mudar no meio do retângulo. | Não. |
| 2995 | HLE: return Winsock sockaddr errors from getnameinfo on invalid input | `name_info_addr` devolve `WSAEFAULT` para `sockaddr` nulo ou curto e `WSAEAFNOSUPPORT` para família não suportada, como o `ws2_32.dll` nativo, em vez de `WSAHOST_NOT_FOUND` (`net.rs` +154). | Não. |
| 2996 | GPU: keep the group grid in bounded indirect mesh dispatches | O clamp de argumentos indiretos preserva os três contadores de grupo quando a grade cabe no teto de 65.535 por registro, em vez de achatar Y e Z para 1 (`indirect.rs` +52). | Não no catálogo de VIctory. `AvatarManz D-01` trata de orçamento indireto esgotado por registro lixo — sintoma diferente. |
| 2997 | Shaders: fix compute lane permutations on WARP | Para o WARP 1.0.22, `ds_bpermute_b32` é lido em grupos de quatro lanes e o resultado pedido é selecionado; software e hardware ganham chaves de cache distintas (`lower.rs` +48, `shader_cache.rs` +35). | Não. Correção específica de adaptador de software. |
| 2998 | HLE: deliver initial XNetworking registration callbacks and track unregister tokens | Entrega os callbacks iniciais de porta UDP (`0x30`, 3074) e de dica de conectividade (`0x80`, OFFLINE) pela `XTaskQueue` do chamador, com tabela de tokens para `0x38`/`0x88` e `E_INVALIDARG` em ponteiro nulo (`xnetworking.rs` +315). | Não. |
| 2999 | Shaders: implicit-level samples in compute follow the console's lane quads | Alinha o quad implícito do compute ao arranjo x-major do console (`lower.rs` +112), para o nível implícito de `CS_ApplyCombinedDeferred` e do blur de sombra de densidade não divergir do console em grupos 2D. | Não. Cobertura nova; parte do conjunto `#2027`. |
| 3000 | emit host: an empty root-constant list is not passed to the runtime | Pula `SetComputeRoot32BitConstants` quando `user_data_dwords() == 0` em `emit/host.rs` (+6), evitando `count = 0` com ponteiro para vetor vazio, que derrubava o processo (0xc0000005) em placa NVIDIA. | Não. Defeito novo do host de teste. |
| 3001 | App: release LoaderLock on unwind via Drop guard | Move a liberação do `LoaderLock` para um `Guard` com `Drop` em `apps/victory/src/run.rs` (+38), para um panic não deixar o lock preso e travar as outras threads em `self.free.wait`. | Não é `D-19` (deadlock de loader lock no cache de módulos HLE, em outro fork). |
| 3002 | GPU: an early list reset leaves a held host list it cannot replace | Em `list::restart`, quando `CreateCommandAllocator`/`CreateCommandList` falham com `E_OUTOFMEMORY`, a lista do host que a fila ainda usa deixa de ser mantida no lugar; o PR descreve o `Close` falhando em 0x80004005 e o risco de rodar a mesma lista duas vezes (+181/-16). | Não. |

## Pull requests mesclados (0)

Nenhum PR com `mergedAt` posterior a 2026-11-26T02:11:35Z. O maior número mesclado devolvido pela API é
2493 — os merges recentes deste repositório usam números antigos, o que confirma que o filtro por
`last_pr_number` sozinho não serve para detectar merge aqui (ver Confiança).

## Cruzamento com o catálogo

- Corrige D-xx? Nenhum. Todos os 27 itens de `catalogo/defeitos.csv` deste repositório já têm hash de
  correção; nenhum PR aberto reivindica um deles.
- Abre defeito novo? Não verificável sem merge. Três PRs declaram defeito descoberto por eles mesmos:
  `#2983` (bits de redução do sampler, com controle negativo 384 vs 128) e `#3000`
  (0xc0000005 em `emit::memory_tests::heap_direct_texture_reads_mark_their_heap_index` na NVIDIA),
  `#2979`/`#2984` (lacunas de `Close`/`Reset`/`ClearState`). Nenhum tem hash em `main` — só viram
  defeito catalogável quando mesclarem.

## Confiança

Verificado: topo da `main` e ausência de commit novo (API de commits, primeiro elemento = `63bcca43`),
números, títulos, corpos, arquivos e contagens de linha dos 24 PRs (`gh pr view --json`), e que nenhum
PR foi mesclado na janela (`gh pr list --state merged` filtrado por `mergedAt`).

Não verificado: comportamento em jogo de qualquer PR (todos são propostas, nenhum foi executado aqui).
O diff de cada PR foi lido pelos corpos e pela lista de arquivos, não patch a patch — as descrições de
"proposta" acima são as dos autores, checadas contra o conjunto de arquivos tocados.
