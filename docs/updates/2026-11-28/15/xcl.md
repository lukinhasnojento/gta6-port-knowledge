# ivall/xcl — atualização de 2026-11-28 15:00

## Resumo

Quarenta commits, todos de `bmjask`, todos em `crates/xcl/src/hle/d3d12x/` mais `shaders.rs`,
`stall.rs` e `README.md`: é a base de três frentes que ainda não estão ligadas a política nenhuma —
classes de alocação no `TilePool` (`vram.rs`), gerações nativas de PSO com receita de rebuild
(`pso_async.rs`) e épocas de descriptor heap compartilhado (`vram.rs`/`console_heap.rs`/`list_x.rs`) —,
mais a frente que já tem consequência visível: o estado esparso fora do lock, que na falha de upload
**derruba o título em vez de perder a submissão** (`cfeb2817`, `86bad822`, `f5ce20f3`, `8fd21391`).
Cinco commits são diagnóstico de `Wait64` para o `XCL_THREAD_DUMP_AT`, escritos a partir de um travamento
real sem time-out (`rec 0x619, sub 0x0@qNone`, citado no corpo de `bff80f9b`). O merge da hora, `#3159`
(`309c3ec5`), restaura a fronteira de bloco que o otimizador de CFG re-fundiu e devolve a ordem
`getDimensions` → `textureLoad` em `Texture2DMS` — a regressão de **D-24**.
Quatro PRs abertos, todos em rascunho, em `device_x.rs`/`bind.rs`/`sparse.rs`.

## Commits novos (40)

Janela: os 40 primeiros commits **sem documento anterior** na ordem do `compare`
`be9d4057...309c3ec5` (193 commits à frente, 168 sem documento). Ordem do `compare` não é estável
(ver Confiança), então a janela foi escolhida por hash.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `c580dd31` | bmjask | D3D12.X: separate tile pool allocation classes | `vram.rs`: `TilePool` ganha `heap_evictable: Vec<bool>` e `add_heap_at(slot, pool_pages, evictable)` passa a receber a classe; +54/−20. Nada é evictado ainda — o corpo diz "behavior-neutral". |
| `e009ba2c` | bmjask | D3D12.X: retain tile ownership by mip group | `vram.rs`: `Target`/`Levels` ganham `group_tiles: Vec<(Vec<usize>, Vec<(usize, usize)>)>` inicializado por grupo (`vec![(Vec::new(), Vec::new()); groups.len()]`); as chaves de página de cada grupo passam a sobreviver a release, unmap adiado e cancelamento. |
| `4cbefd18` | bmjask | Shaders: prewarm the first indirect split dispatch | `dispatch_chain.rs`: `EARLIER_INDIRECT_USED` (`LazyLock<HashSet<u64>>`, vazio em teste) e o registro do root direto já usado; `shaders.rs`: `split_whole_roots(prewarm_indirect)` devolve `[DirectChain, IndirectChain]`. O `compare` também muda o `README.md`. |
| `c5fb9fe5` | bmjask | Shaders: report durable pipeline preparation outcomes | `shaders.rs`: `AtomicBool` novo ao lado dos contadores; `write_failures: AtomicUsize` mantido e cada criação fresca recebe **um** desfecho terminal (saved / unavailable / disabled / write failed). |
| `b1c3e98d` | bmjask | D3D12.X: attribute CPU-write looks only during activity runs | `sparse.rs`: os `static WATCH: [AtomicU64; 2]` e `PHASES: [AtomicU64; 4]` viram contadores de uma linha de "looks"; `title_textures.rs::refresh` recebe `measure`; `host.rs` loga o delta. Com `XCL_ACTIVITY` desligado não há leitura de clock. |
| `680ea612` | bmjask | D3D12.X: create lists on the active allocator backing | `device_x.rs`: `active_list_allocator_with(alloc, current)` resolve a backing ativa; `list_x.rs::on_create_with_backing` separa allocator lógico do título e backing nativo usado no `CreateCommandList`. |
| `3168aea1` | bmjask | D3D12.X: attribute submitting-thread descriptor costs | `stall.rs`: `StallKind::DetailViewDescriptor` e `ALL` de 20 → 21; `report.rs` ganha o nome `detail_view_descriptor`; `real.rs` abre `detail_span` nos dois `dev_create_*_x`. |
| `75485a22` | bmjask | D3D12.X: make activity look intervals coherent | `sparse.rs`: os 10 contadores de `LOOK` são reorganizados (61 → 87 linhas) para publicar cada look como um registro; `refresh` separa `wait_t0` de `active_t0` e o `README.md` documenta a linha nova. |
| `3b5bc7b8` | bmjask | D3D12.X: preserve tile allocation class invariants | `vram.rs`: o doc de `heap_evictable` deixa de afirmar classe de evicção e passa a "fresh-allocation packing class", com o comentário de que página compartilhada pode ter dono das duas classes; `keep_spare` mantém um pool vazio por classe. |
| `1dccf831` | bmjask | D3D12.X: report retained pipeline age | `pso_async.rs`: `last_use_frame` e `completed_frame` (`AtomicU64`) por `Build`; `host.rs` loga `pipeline retention: N completed background PSOs behind stand-ins; M unused for 60+ frames…`. |
| `04de3234` | bmjask | D3D12.X: preserve CPU writes during sparse copy-back | `sparse.rs`: `land_page()` novo — o copy-back da GPU aterrissa a página **chunk de 64 bytes por chunk**, e um chunk cujo hash não casa mais com o último enviado mantém os bytes da CPU e o hash antigo para o próximo look reenviar. |
| `60a90b12` | bmjask | xcl run: `XCL_THREAD_DUMP_AT` also logs the fences and queue waits behind | `list_x.rs::log_sync_state(at)` roda numa thread própria (`xcl-sync-state`) para não ficar presa no lock do stall; `host.rs` chama no mesmo laço do dump de threads. |
| `3caff9ef` | bmjask | xcl run: `XCL_THREAD_DUMP_AT` also logs the `Wait64` cells behind | `list_x.rs`: dump das células `Wait64BitValueX` com `rec` acima de `sub` e sem espera armada, com `try_lock` (o dump pode chegar com o grafo na mão do submissor); `dstorage.rs::read_own` passa a `pub(crate)`. |
| `bff80f9b` | bmjask | xcl run: the sync-state dump names the list behind a `Wait64` cell | `list_x.rs`: `SyncCell`/estado da lista ganham `recorded_by: Option<(usize, Instant)>`; `on_release` e `on_reset` passam a emitir `ListEvent::Released`/`Reset` no traço. |
| `976faf72` | bmjask | xcl run: a Reset or Release ends the recording the `Write64` trace follows | `list_x.rs`: o traço deixa de ser por endereço de lista e passa a ser por **gravação** (`Arc`, importado de `std::sync`), então `Reset`/`Release`/reuso de endereço não mostram a execução antiga na célula nova. |
| `9d8af31c` | bmjask | xcl run: trace immutable submission generations | `list_x.rs`: `write64_trace: Option<Arc<Mutex<ListTrace>>>` e `traces: Vec<Option<...>>` capturados junto de `effects`; `host.rs` chama `enable_sync_state_tracing()` quando `XCL_THREAD_DUMP_AT` está setado. |
| `1b937474` | bmjask | D3D12.X: retain opt-in pipeline rebuild recipes | `pso_async.rs`: `recipe: Option<Recipe>` em `Pending` (com `#[allow(dead_code)]`); `real.rs::OwnedDesc` deixa de guardar a root signature emprestada do título e documenta que todo ponteiro aponta para backing imutável. Ligado por `XCL_PSO_REBUILD_RECIPES=1`. |
| `608cf741` | bmjask | D3D12.X: count sampled array slices in residency metrics | `vram.rs`: `mip_mask()` é removida e o cálculo de cobertura SRV passa a guardar intervalos de fatia disjuntos por mip, somando só texels novos; continua restrito a `XCL_ACTIVITY`. |
| `776fb303` | bmjask | D3D12.X: expose exact submission retirement receipts | `alloc_pool.rs`: recibo novo com o dono do fence mantido (progresso exato da fila depois do `ExecuteCommandLists`), +186/−9; `#[allow(dead_code)]` até o estágio de pin. |
| `d87f1a05` | bmjask | D3D12.X: track live texture SRV metadata | `vram.rs` +307: `XCL_TEXTURE_BUDGET_EVICT=1` liga o registro de identidades SRV vivas (heap, slot, geração); `console_heap.rs` passa a tirar o heap do mapa antes de liberar; `real.rs` toma `lock_tracked_srv_write` e limpa o slot. |
| `9d08eceb` | bmjask | D3D12.X: publish sparse upload hashes after queue receipt | `sparse.rs`: `SubmitResult { Submitted, NotSubmitted, Unordered }` e `uploads_ordered: bool` — depois de um `ExecuteCommandLists` cujo `Signal` falhou, nenhum allocator nem byte de staging é reutilizado; hash só é publicado após o recibo. |
| `dcfbb49c` | bmjask | D3D12.X: own immutable native pipeline generations | `pso_async.rs`: geração nativa monotônica com um dono forte antes de o resultado cru ficar visível; `complete()` passa a devolver `(orphan, replaced_generation)`. |
| `cfeb2817` | bmjask | D3D12.X: fail title submits closed on sparse upload loss | `real.rs`: `prepare_title_submission(queue)` substitui `title_textures::refresh` nas **duas** entradas (`q_execute_command_lists` e `kickoff_x`) e encerra o título quando `PreparationOutcome != Ready`, porque a API de submissão é `void`; `sparse.rs` ganha `PreparationOutcome { Ready, RetryRequired, ExecutedUntracked }` e `retry_pages: Vec<usize>`. |
| `9d7bea0f` | bmjask | D3D12.X: capture shared descriptor heap use | `list_x.rs`: `shared_view_heap: bool` e `uses_shared_view_heap: bool` por segmento físico; `bind.rs` marca quando `ensure_view_heap` devolveu heap compartilhado e o tracking está ligado; `real.rs` acrescenta o slot 27 (`execute_bundle`). |
| `c524972c` | bmjask | D3D12.X: retire shared descriptor heap epochs | `vram.rs`: `SharedViewEpoch(u64)` e `EpochRetirement` (+201); `list_x.rs::submit_title_chunk` só pede recibo quando a gravação usa o heap compartilhado; `real.rs` troca `gate.submit(...)` por essa chamada. |
| `665ce029` | bmjask | D3D12.X: capture native PSO generations in recordings | `list_x.rs`: `native_generations: Vec<NativeGeneration>` + `pin_native_generation` com `push_unique` por identidade; `bind.rs`/`device_x.rs`/`real.rs` trocam `resolve_wait` por `resolve_and_pin_if_enabled`; `console_list.rs` deixa de limpar a captura no wrapper externo. |
| `86bad822` | bmjask | D3D12.X: release sparse state during upload waits | `sparse.rs` +288: `UploadStep`, `BeginSlot`, `WaitKind { Slot, Ring }` e `WaitTicket { state_generation, mapping_epoch, dev, fence, target, kind }` — a espera acontece com `STATE` liberado, e a revalidação é por geração. |
| `9da465f1` | bmjask | D3D12.X: serialize native PSO generation rebuilds | `pso_async.rs`: `RebuildResult` compartilhado entre pedidos sobrepostos; `recipe.publish(real).1` devolve a geração substituída; falha mantém a geração corrente. |
| `11613fb6` | bmjask | D3D12.X: retire native PSO generation pins | `pso_async.rs`: `GenerationRetirements { pending: Vec<(R, Vec<G>)>, poisoned: Vec<G> }`; `list_x.rs` expõe `native_generations` no `KickoffEffects`; `real.rs` só pede recibo quando `rebuild_recipes_enabled()` e o chunk tem gerações. |
| `d4084ba0` | bmjask | D3D12.X: pin tracked SRV resources across release | `vram.rs`: `TRACKED_SRV_LIFETIMES: Mutex<()>` com `lock_tracked_srv_lifetime()`; `real.rs::res_release` toma o gate **antes** de o objeto nativo poder morrer e só depois solta os donos COM. |
| `c1be67ec` | bmjask | D3D12.X: queue native pipeline rebuild requests | `pso_async.rs`: `RebuildRequest { Queued, Coalesced }`; a tentativa é reservada antes de ir para a banda de workers especulativos e recheca a posse do título antes do driver e da publicação. |
| `19680364` | bmjask | D3D12.X: choose a ready sparse upload slot | `sparse.rs`: `begin_slot` varre o anel pequeno de slots **antes** de esperar; se todos estão ocupados, espera o menor valor do fence compartilhado sem avançar a posse do slot. |
| `7dd1390d` | bmjask | D3D12.X: deduplicate native generation pins linearly | `pso_async.rs`: `identity()` usa `Arc::as_ptr(&self.0)` em vez do ponteiro cru do pipeline (que o driver reusa), e a deduplicação passa a `HashSet` linear. |
| `a3b43590` | bmjask | D3D12.X: lease exact descriptor heap slots | `console_heap.rs`: `HEAP_LIFETIMES: Mutex<()>`/`HeapLifetimeGuard`; `real.rs::release_taking_first` toma o gate antes de liberar o heap; `vram.rs::forget_srv_heap_after_release` remove candidatos sem tomar o gate de escrita. |
| `00dfb305` | bmjask | D3D12.X: revalidate tracked SRVs before mutation | `vram.rs` +181: transação de mutação **não chamada** que compõe quiescência de fila, lease exato de recurso e heap, exclusão de escrita e revalidação das oito palavras; `console_heap.rs::exact_view_slot_words(geração, k)`. |
| `f5ce20f3` | bmjask | D3D12.X: release sparse upload gate during fence waits | `sparse.rs`: o doc de lock order deixa de ser `WAIT_GATES → UPLOAD_GATE → STATE` e passa a um gate que serializa só slots/cursors/journal, com a backing nativa retida por geração durante a espera. |
| `c1bdf10e` | bmjask | D3D12.X: detach evicted native pipeline generations | `pso_async.rs` +262: `EvictResult` e a transição de evicção (não chamada) que desanexa dono cru e dono de geração atomicamente, excluindo publicação de rebuild. |
| `8fd21391` | bmjask | D3D12.X: split sparse upload gate timing | `sparse.rs`: `with_upload_gate_released` é substituída por medição separada de aquisição e posse do gate entre preparação de submissão e readback síncrono; a linha do `README.md` passa a mostrar `upload gate submit wait/hold`. |
| `85c26b1e` | bmjask | D3D12.X: batch exact descriptor mutation leases | `vram.rs` +270: `ExactSrvMutation { … }` (com `#[allow(dead_code)]`) amortiza quiescência e exclusão de escrita num conjunto determinístico de gerações SRV, soltando todo dono fora dos locks de tracker e escrita. |
| `8eb35ad7` | bmjask | D3D12.X: queue evicted pipeline rehydration | `pso_async.rs` +291: `RehydrateRequest { Queued, Coalesced }`; recusa, cancelamento e unwind restauram estado retentável, e release durante o trabalho do driver não ressuscita a geração. |

## Pull requests abertos (4)

Todos em rascunho (`draft: true`), nenhum com commit na janela.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #3160 | D3D12.X: derived pipelines ignore hint patches; derived compute is the base | `apply_patch` passa a ignorar os tipos 4 (opções de pixel shader), 6 (limites gráficos) e 7 (tesselação) como já ignora o 5, com uma linha de log por tipo; `create_derived_compute_pso_x` (device #63), que era stub que retornava `S_OK` sem escrever o out, devolve a pipeline base por `QueryInterface`. Tipos 8+ seguem `E_FAIL`. | Não. O corpo declara que o título de varejo não chama #55 nem #63. |
| #3161 | D3D12.X: unmodelled console resource and counter-set creators fail visibly | Devices #45/#77 (recurso com planos e metadados em endereços separados) e #82 (`counter_set`) passam de stub `S_OK` para `E_NOTIMPL`, com `null` escrito no out onde a posição é conhecida. | Não. |
| #3164 | D3D12.X: hash sparse uploads from a cached copy, not the staging ring | Desde `9d08eceb` (commit desta janela), cada upload sujo lê de volta o staging ring (memória write-combined) para tirar hash. O PR copia uma vez para `upload_scratch` em `State`, tira o hash ali e copia para o ring (+14/−5). Medição declarada em RTX 5070 Ti: 309→1.677 e 315→1.679 frames em 60 s, 12→60 presents por 2 s. | Desempenho, não defeito de catálogo. Se aceito, muda o custo do `9d08eceb` que está nesta janela. |
| #3165 | D3D12.X: a draw's two-pass buffer goes past the headroom cap, not skipped | `bind.rs`: `env_two_pass_cap`, `draw_two_pass_ceiling`, `make_two_pass_past_cap`; o record buffer e o argument scratch passam a poder ir até `min(orçamento local / 4, 2048 MiB)` (1024 MiB sem orçamento) e só recusam acima disso. `XCL_TWO_PASS_MB` continua teto duro. Relato: numa RTX 3060 com 16 heap windows, o primeiro `ExecuteIndirect` de two-pass era pulado e cinco filas ficavam em `Wait64BitValueX >= 0x713`. | Não. Área de espera entre filas, sem hash de catálogo. |

## Pull requests mesclados (1)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #3159 | Recompiler: preserve MSAA load block boundary | `309c3ec5` (2026-11-28T14:48:55Z) | 3 arquivos, +34/−1: `gpu/dxil/module.rs` marca alvos de bloco que a simplificação de CFG não pode fundir, `module/opt.rs` mantém alvos marcados fora do merge linear de blocos e `dxil/lower_image.rs` preserva a fronteira antes dos loads diretos de MSAA. O corpo explica a causa: o commit `56237a2d` já tinha posto a consulta de sample-count antes dos loads, mas a simplificação de CFG fundiu o bloco de predecessor único e o DXIL voltou a pôr `GetDimensions` antes de `TextureLoad` no mesmo bloco — o padrão que derruba o `amdxc64`. Teste novo `simplify_cfg_keeps_preserved_block_boundary`. |

`#3157` (`22247810`, `fix/dispatchx-wave-uniform`) foi mesclado às 14:06:59Z e já está documentado em
`docs/updates/2026-11-28/14/xcl.md`; não é recontado.

## Cruzamento com o catálogo

- **Corrige D-xx? Por hash, nenhum.** Os 40 hashes da janela e os 2 hashes de merge (`309c3ec5`,
  `22247810`) foram conferidos por prefixo de 12 contra as colunas `introduzido`/`corrigido` das 305
  linhas de `catalogo/defeitos.csv` (262 hashes distintos): zero interseção.
- **D-24 por área, com evidência no corpo do PR.** `D-24` é "Ordem `dx.op.getDimensions` →
  `dx.op.textureLoad` em Texture2DMS derrubava o compilador da AMD" (introduzido `2dcc926`, corrigido
  `425b869`). O #3159 é a **segunda** ocorrência do mesmo padrão: o corpo diz que `56237a2d` pôs a
  consulta antes e a simplificação de CFG desfez, "restoring the AMD compiler crash pattern guarded by
  the regression test". O conserto de 2026-11-26 não cobria o merge de blocos; este cobre. Não há
  linha nova de catálogo proposta aqui, porque é o mesmo defeito e o catálogo já o tem.
- **Abre defeito novo?** Não comprovado. Três candidatos com evidência no próprio diff, todos
  declarados como preparação e com `#[allow(dead_code)]`: (a) `d87f1a05` registra identidades SRV mas
  o próprio doc diz que não torna substituição de descritor segura para a GPU (área de D-11:
  DXIL/descritor obsoleto servido de cache); (b) `665ce029` captura gerações de PSO em gravações e
  `11613fb6` as aposenta, mas o caminho continua opt-in por `XCL_PSO_REBUILD_RECIPES`; (c) `cfeb2817`
  transforma falha de upload em encerramento do título — é fail-closed deliberado, não perda silenciosa,
  e o corpo diz por quê.

## Confiança

**Verificado:** os 40 commits foram lidos por API (`repos/ivall/xcl/commits/<sha>`) com lista de
arquivos, `+/-` e, para cada um, os cabeçalhos de hunk e as primeiras linhas alteradas de cada arquivo;
oito deles (`c580dd31`, `04de3234`, `cfeb2817`, `9d08eceb`, `86bad822`, `60a90b12`, `9d8af31c`,
`976faf72`) tiveram o patch lido em extensão. O `compare` `be9d4057...309c3ec5` devolveu 193 commits,
168 sem documento anterior (dedupe por prefixo de 8 contra todo `docs/updates/**/*.md`). O merge #3159
veio de `repos/ivall/xcl/pulls/3159` (corpo, 3 arquivos, +34/−1). Os 4 PRs abertos vieram de
`pulls/<n>` com corpo e contagem de arquivos.

**Não verificado:** nenhuma build, teste ou execução; os números de frames do #3164 e de `Wait64`
citados no corpo dos commits são relato dos autores. Não abri o diff dos commits `309c3ec5` e
`22247810` além do que o corpo do PR descreve, nem os diffs dos 4 PRs abertos. Não sei se `#3160` e
`#3161` quebram o teste `#1921` que o próprio corpo menciona.

**Cobertura:** 40 de 168 commits não documentados. `last_commit` avança até `8eb35ad7`, o 40º da
janela; `commit_cursor` = `97583afd` (o 41º sem documento, na ordem atual do `compare`); `head_at_run`
= `309c3ec5`. O merge `#3159` está **além** do cursor (chegou às 14:48Z, depois do head coletado no
ciclo de 14:00) e foi documentado por data; quando a varredura chegar nele, o hash já aparece neste
documento e a deduplicação o reconhece.
