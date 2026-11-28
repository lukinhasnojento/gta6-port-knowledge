# ivall/xcl — atualização de 2026-11-28 16:00

## Resumo

Hora de andaime: a maior parte dos 40 commits documentados é da série de `bmjask` que modela evicção
e restauração de pipelines e o caminho de upload esparso — boa parte dela `#[allow(dead_code)]` ou
`#[cfg(test)]`, sem chamador de produção. O que tem consequência hoje é o merge `#3100`
(`bb9a6b4b`), que passa a importar PSOs exatos da biblioteca de pipeline anterior em vez de
recompilá-los quando a chave semântica muda, e dois scripts de medição (`XCL_GPU_TIMING`, gate de
upload esparso). Entre os 9 PRs abertos, `#3167` e `#3169` descrevem falhas com sintoma: listas
enviadas vazias e samplers estáticos perdidos em blobs de outro processo.

## Commits novos (40 de 128 sem documento)

Janela: `compare(97583afd...bb9a6b4b)` = 154 commits. Documentados os 40 primeiros sem documento na
ordem do `compare`; 114 ficam para os próximos ciclos.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `18d38a67` | bmjask | Scripts: compare sparse upload gate timing | Adiciona `Read-SparseUploadGate` ao `perf-compare.ps1`: lê do `xcl.log` a linha `upload gate submit wait/hold …` e soma submit/readback wait/hold; log ausente ou contador ausente viram `missing_log`/`missing_metrics`, não zero. Dois testes PowerShell novos. |
| `b4625a4d` | bmjask | D3D12.X: journal bounded pipeline eviction candidates | Cria `CandidateQueue<T, 1024>`, anel MPMC sem lock com `UnsafeCell<MaybeUninit<Weak<T>>>` e sequência por slot; cada `Build` ganha `candidate_queued: AtomicBool`. Só registra candidatos: nada drena a fila. |
| `a3fdbde4` | bmjask | D3D12.X: model immutable sparse upload plans | Define `UploadPageObservation` (página, pool, buraco, hashes `sent`/`source`) e `PlannedUploadPage`; `#[allow(dead_code)]`, nenhum chamador. |
| `48286fdd` | bmjask | D3D12.X: rank bounded pipeline eviction candidates | Acrescenta `clear_queued_on_drop` para que um candidato devolvido à fila não deixe o `Build` marcado para sempre como enfileirado; `requeue_on` usa o anel de `b4625a4d`. Ainda sem consumidor. |
| `bed14445` | bmjask | D3D12.X: model sparse upload ring reservations | `UploadReservationToken`, `UploadRingRange`, `UploadSubmissionReceipt` e o ledger de ring; tudo `#[allow(dead_code)]`. |
| `3e2a48ec` | bmjask | Tests: compare flat and guarded heap-window reads | Teste de GPU que traduz um kernel e compara leitura de heap plana (DXIL direto) com a guardada em quatro janelas e no maior heap, incluindo lanes inativas, endereço nulo, fora da UMA e carga `dwordx2` no limite; pula sem adaptador. |
| `8828fec8` | bmjask | D3D12.X: model pipeline pressure admission | `PressureAdmissionConfig` com `enabled: false` e limiares 7500/9000 bp; admissão pura sobre uma amostra de memória passada pelo chamador. O próprio comentário diz que nada constrói esse estado. |
| `48ab3bc6` | bmjask | D3D12.X: own sparse upload staging bytes | `OwnedUploadStaging { range, bytes: Box<[u8]> }` — cópia própria para que o callback de record/execute não retenha uma view da memória do título. Sem uso. |
| `5674db9d` | bmjask | D3D12.X: model sparse claim receipt frontier | `ClaimFrontierToken { state_generation, epoch, lease, owner }` e estados Pending/Accepted/Receipted/Obsolete; o `lease` muda no takeover para um dono antigo não publicar pelo novo. Sem uso. |
| `fdd9df7b` | bmjask | D3D12.X: extract exact synchronous dirty upload batches | Extrai `PreparedDirtyBatch { slot, ring_start, updates }` do caminho de upload sujo já existente, com `new`/`reopen`; é refatoração de código vivo, não andaime. |
| `4b0c6ad7` | bmjask | D3D12.X: couple sparse batch ownership | `SparseBatchLedger` põe `WorkClaimLedger` e `ClaimFrontier` sob um mutex e define `SparseBatchError`; comentário declara o ledger inerte. |
| `379aa29d` | bmjask | D3D12.X: track bounded evicted pipeline identities | `LedgerNativeGeneration` e `EvictedBuildState` (Reserved/Evicted/RehydrateQueued/Restored/Released/Poisoned); só tipos. |
| `f663775c` | bmjask | D3D12.X: observe bounded pipeline pressure candidates | `try_requeue_on`, `into_held` e `HeldEvictionCandidate`; liga a observação em `host.rs`. Devolve `Result` em vez de `bool` para o chamador distinguir fila cheia. |
| `c5d854d7` | bmjask | D3D12.X: publish accepted sparse upload reservations | `publish_after_accept(fence)`: fence 0 envenena o registro e devolve erro; sucesso grava o recibo. Falha de publicação é fail-closed, não cancelamento. Código vivo. |
| `e3fb670c` | bmjask | D3D12.X: compose sparse batch submission receipts | `accept` passa a devolver o `WorkClaimToken`; novo `poison_exact` confere geração do token e do trabalho antes de envenenar, para não derrubar um lote de outro dono. Inclui testes. |
| `cfc11ab6` | bmjask | D3D12.X: time native title queue submissions | `XCL_GPU_TIMING=1` mede cada `ExecuteCommandLists` nativo no CPU, grava `submissions`/`slowest_submission_ms`/listas em `report.json` e loga blocos ≥ 50 ms (`XCL_GPU_TIMING_MS`); o comentário diz explicitamente que não é tempo de GPU. |
| `9a077c0f` | bmjask | D3D12.X: model exact pipeline rehydration receipts | `RehydratePriority`, `RehydrateRequestReceipt` e `RehydrateCompletion` com `terminal()`; tipos apenas. |
| `1d942ea8` | bmjask | Tests: add a dependency-free benchmark harness | `benches/support/mod.rs` com `Criterion`/`BenchmarkGroup` próprios sobre `Instant`, mediana/média/mínimo e filtro posicional; sem `criterion` como dependência. |
| `c35d24a8` | bmjask | Tests: add synthetic benchmark foundation | Declara `[[bench]] core` e `formats` com `harness = false` no `Cargo.toml` e documenta `cargo bench -p xcl`; entradas sintéticas, sem arquivos do jogo. |
| `1120fdc9` | bmjask | Tests: benchmark runtime and shader hot paths | Dois alvos novos, `gpu_cpu` e `runtime`; codifica instruções RDNA2 sintéticas para medir tradução e caminhos quentes. |
| `0bcb6444` | bmjask | D3D12.X: coordinate exact pipeline restoration state | Acrescenta `state`, `rehydrate_retry` e `reserve_unique` ao coordenador de restauração, com token exato por slot; a reserva recusa entrada duplicada. |
| `8f3998f1` | bmjask | D3D12.X: model exact foreground sparse waits | `SparseFenceIdentity` e `replace_generation(..., fence_allocation)`: só aceita fence cuja alocação é maior que a atual; geração ou alocação zero não produzem identidade. |
| `cfb7374c` | bmjask | D3D12.X: consolidate coordinated sparse upload foundations | Move `OwnedUploadStaging` para o módulo consolidado e adiciona o teste `late_claim_completion_cannot_change_a_newer_owner`. |
| `1a3bd1d3` | bmjask | D3D12.X: retain exact foreground restoration attempts | Separa `Authority::{Current, Published}` e passa `JobStatus` do `build_queue`; `published()` valida receita e geração residente antes de devolver o `Build`. |
| `3172fd8f` | bmjask | Tests: cover exact pipeline restoration attempts | Ganchos `before_rebuild` sob `cfg(test)` e contador thread-local de reentrada para provar que a restauração não reentra no coordenador. |
| `c3e75fb4` | bmjask | D3D12.X: install exact eviction tokens before detach | Guarda `eviction: Mutex<Option<EvictedBuildToken>>` no `Build`, com `belongs_to`, `eviction_token` e limpeza; o token fica visível antes de os donos nativos serem destacados. |
| `30511e1f` | bmjask | Tests: cover exact eviction token installation | Teste que arma a evicção, confere o token visível antes do commit, limpa e aposenta. |
| `1566d744` | bmjask | Scripts: compare native submission timing | `Read-SubmissionTiming` no `perf-compare.ps1`, com validação de número não negativo e `missing_report`; o comparador passa a ler `report.json`. |
| `f19a07de` | bmjask | D3D12.X: classify exact pipeline restoration without waiting | `StructuredResolve` (Ready/Initial) privado e não ligado, mais o gancho de teste `before_classified_request`; a ideia é classificar sem esperar e só esperar na emissão. |
| `5f04285f` | bmjask | Tests: cover structured pipeline restoration classification | Teste que retém o `Pending` original através da reutilização do endereço do título. |
| `3cf40d1c` | bmjask | D3D12.X: aggregate native submission timing | `report.json` ganha total, média, p95/p99 aproximados e histograma em buckets log2 de ns; o texto do campo deixa claro que é duração de CPU dentro da chamada nativa. |
| `86daee84` | bmjask | Scripts: compare submission timing distributions | O comparador aceita os campos estendidos **ou** os legados, recusa conjunto incompleto e compara histogramas; `histogram_log2_ns` ausente é `null`, não zero. |
| `dd208b9e` | bmjask | D3D12.X: retain pipeline decisions in bind state | O estado de bind passa a guardar `Option<PipelineResolution>`; `clear_state`, `reset` e a cópia entre listas fazem `take()` e soltam a decisão **fora** do lock de `bind::STATES`. |
| `8790512e` | bmjask | Tests: cover retained pipeline decisions in bind state | Teste que afirma `STATES.try_lock().is_ok()` no `Drop` da resolução, provando que ela não cai sob o lock. |
| `38452a17` | bmjask | D3D12.X: own one dormant restoration coordinator | `OnceLock` com um coordenador de 1024 evicções e `begin_pipeline_resolution_on`; obter a identidade não inicia worker nem liga evicção. |
| `885e825c` | bmjask | Tests: cover the dormant restoration coordinator | Prova identidade estável do `OnceLock`, que a classificação dormente não roda driver no chamador e que o `Pending` registrado é o mesmo objeto. |
| `833d2676` | bmjask | Scripts: record Cargo feature performance variants | O manifesto de `perf-compare` passa a validar `build.cargo_features`: sem vazio, sem duplicata, ordem ordinal canônica; manifesto malformado falha a comparação. |
| `622c5e61` | bmjask | Tests: benchmark allocator contention across workers | `benches/runtime.rs` mede 128 jobs de alocação com `Barrier` e `Arc<[u8; 64]>` compartilhado entre workers para medir contenção do alocador. |
| `5c50099c` | bmjask | Tests: seed resubmit lifetime identity | Três linhas em `alloc_pool.rs`: antes do submit sintético, chama `begin_submit(&[list])` e solta o guarda, para a submissão real capturar o endereço exato da lista antes de publicar `LastSubmit`. |
| `3cde0cc3` | bmjask | Recompiler: use larger compact tables on every GPU | Remove o parâmetro `tables`/`TABLE_WORDS_WAVE32` do tradutor compacto e usa `MAX_TABLE_WORDS = 12288` em todo adaptador (antes 8192 em GPU de wave64, 12288 só em wave32). Toca `dxil/compact.rs`, `kernel.rs` (−18/+10) e `recomp/pso.rs`; teste novo cobre tabela inline acima do teto antigo. |

## Pull requests abertos (9)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #3177 | XGameRuntime: export the XG library's older desc computers and helpers | Exporta os 23 entry points de `xg_xs.dll`; hoje o XCL exporta 2, e o stub do loader devolve 0 deixando structs de layout sem init. Empilhado em #3147, só o topo `f4a941b5` conta. | Não por hash; área XG/XGameRuntime, sem linha no catálogo. |
| #3176 | D3D12.X: a title buffer may run on into the next sparse window | Reserva que atravessa a janela esparsa (`0x546000` em `0x5fffd0000`) passa a poder cruzar para a seguinte em vez de ficar presa numa janela. | Hipótese: mesma família de D-01 (janela UMA), sem hash. |
| #3175 | D3D12.X: record ordered pipeline first use | Diagnóstico opt-in de primeiro uso de pipeline, trazido de #2096 para a `main` pós-#3100. | Não. |
| #3173 | kernelx: XMemVirtualAlloc reads its 4th argument as a flags word | O 4º argumento é flags, não `XALLOC_ATTRIBUTES`; o XCL lia tipo de memória do campo de atributos e `2` em page-size somava `0x8000_0000` ao tipo de alocação. | Área de alocação; sem linha. |
| #3172 | D3D12.X: end the title past the NtTerminateProcess hook | Draft: run encerrado por `end_title` (`XCL_RUN_SECONDS`) pode ficar vivo; visto 1 em 40 smokes. | Não. |
| #3171 | XAudio2: SHAPE contexts have six kinds with their own entry sizes | `AcpHalAllocateShapeContexts` recebe 0x60 bytes com seis contagens e seis ponteiros; o XCL lia cinco contagens e uma reserva. | Não. |
| #3170 | D3D12.X: log render-target and depth handles no device call wrote | Diagnóstico: mostra se o título copia descritores RT/depth ele mesmo, nos tamanhos do console, para os heaps do PC. Sem mudança de comportamento. | Não. |
| #3169 | D3D12.X: no disk driver blob for a graphics pipeline with static samplers on NVIDIA | Blob `CachedPSO` feito por **outro processo** perde o filtro linear dos samplers estáticos na NVIDIA; a correção deixa de usar blob de disco nesse caso. | Área de samplers estáticos; sem hash. |
| #3167 | D3D12.X: a closed standby is recorded empty before it can be submitted | `fail_closed` de um split manda um standby vindo direto de `CreateCommandList1` — fechado, nunca gravado — e submeter isso remove o device. Corrige #3166. | Área de submissão/comando; sem linha. |

## Pull requests mesclados (1)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #3100 | Shaders: import exact PSOs from prior libraries | `bb9a6b4b` (2026-11-28T15:37:54Z) | Abre a biblioteca de pipeline anterior como fonte de importação oportunista durante a preparação, carrega só o nome content-addressed atual com o descritor e a root signature atuais e copia PSOs de compute/gráfico/mesh aceitos para a biblioteca nova; rejeição do D3D12, entrada ausente, biblioteca não suportada ou arquivo corrompido caem na compilação normal. Arquivos: `shaders/pipeline_library.rs` (+80/−16), `shaders.rs` (+24/−10), `pso_corpus.rs` (+3/−1), `warm_library.rs` (+2/−1), `README.md` (+3/−1). O commit de conteúdo é `2de72223`. |

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum hash da janela casa com as colunas `introduzido`/`corrigido` (154 commits e
  1 merge conferidos por prefixo de 8 contra 262 hashes distintos). `#3167` fecha o issue #3166, que
  não é linha do catálogo.
- **Abre defeito novo?** Não verificado por falta de execução do título. `#3100` foi **mesclado como
  draft** segundo o próprio corpo do PR ("This remains a draft until the required clean main/head
  smoke blocks are available") e não tem smoke comparison nem build release no commit rebaseado
  (`2de72223`); a suíte completa rodou 1.999 pass / 7 fail, com as 7 falhas declaradas herdadas da
  `main` `e13d6250`, uma delas `direct_msaa_loads_precede_get_dimensions` — a mesma área de D-24.
- **Por área:** `#3169` toca samplers estáticos em blob de disco (família de D-02, sem hash); `#3176`
  toca reserva esparsa que atravessa janela (família de D-01, sem hash); `#3167` toca lista de comando
  submetida sem gravação (sem linha).

## Confiança

- **Verificado por diff:** todos os 40 commits (patch lido via `commits/<sha>`, arquivos e hunks) e o
  merge `#3100` (arquivos e corpo do PR). Autores e datas vêm do `compare` e dos `commits/<sha>`.
- **Verificado por API:** estado dos 9 PRs abertos (título, corpo, `created_at`) e `merged_at` de
  `#3100`.
- **NÃO VERIFICADO:** execução de qualquer teste ou do título; comportamento em AMD/Intel (o PR
  testou só RTX 4090 e 5070); a afirmação de que `#3176`, `#3169` e `#3167` corrigem as classes
  citadas — são leitura de corpo de PR e de diff, não reprodução.
- **Cobertura:** 40 de 128 commits sem documento. O cursor novo (`3cde0cc3`) re-inclui na próxima
  janela `2dad32e6` e `abf7e177`, documentados às 12:00; a ordem do `compare` não é topológica.
- **Contagem de merges x commits:** 1 merge (`#3100`) contra 154 commits na janela — o merge traz um
  único commit de conteúdo e o resto da janela é trabalho direto na `main`.
