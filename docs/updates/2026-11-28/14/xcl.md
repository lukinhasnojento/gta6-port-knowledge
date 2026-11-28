# ivall/xcl — atualização de 2026-11-28 14:00

## Resumo

A fila do `xcl` continua sendo a maior do acervo: 232 commits entre o `commit_cursor` `e5266254` e o
head `b937fbab`, mais 7 PRs mesclados na janela (1 novo) e 2 PRs abertos acima de #3155. Os 40
documentados aqui são os 40 primeiros **não documentados** na ordem em que a página do `compare`
devolve, todos de 2026-11-26 08:25–09:25Z: é a série de identidade de cache de pipeline do `D3D12.X`
(samplers estáticos, `NodeMask`, dispositivo, raiz de assinatura, `HEAP_FLAT`), a série de
*lock/ownership* do estado esparso (criar/liberar/unmap fora do lock global, com geração exata de
fence) e três consertos de infraestrutura de áudio/loader. O único merge novo, #3157 (`22247810`),
torna uniforme a ramificação de *rendezvous* do `DispatchX` e fecha o que o #3150 começou.

## Commits novos (40 de 211 não documentados)

Datas de autor de 2026-11-26 08:25:22Z a 09:25:24Z; 40 de 40 têm autoria de `bmjask`.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `ba44b52c` | bmjask | D3D12.X: key compute driver blobs by static samplers | Acrescenta o conjunto ordenado de `StaticSampler` à identidade do blob de driver de compute. `dispatch_chain.rs` passa `samplers` ao `PipelineBuilder`; `real.rs` troca `driver_cache::name/path_for` por `name_for_root`/`path_for_root`; `shaders.rs` usa a mesma função. Raízes sem sampler mantêm o nome legado, então o cache antigo continua válido. |
| `a97dbe9b` | bmjask | D3D12.X: create sparse placed buffers outside global state | `CreateReservedResource` deixa de rodar sob o `STATE` esparso: um `Mutex<()>` `PLACEMENT` serializa colocação e troca de estado, a criação do recurso acontece fora, e a publicação revalida a geração exata de fence e a janela antes de aceitar. `Release` diferido é drenado antes de o endereço COM reutilizado ser registrado, e o abort libera o dono COM fora do estado. |
| `f05846e7` | bmjask | D3D12.X: cache invariant slots in hot hooks | Cacheia o primeiro slot de cada família de interface (`ID3D12GraphicsCommandList`, `ID3D12Resource`, …) em vez de varrer nomes de método a cada chamada quente. A tabela é preenchida no primeiro uso e um teste percorre todas as versões da família exigindo um único slot não nulo. |
| `96278db7` | bmjask | D3D12.X: shard pending pipeline lookups by stand-in | Troca o `PENDING: Mutex<Option<HashMap>>` por `PendingRegistry` com 64 shards indexados pelo endereço stand-in. Insert, lookup e release viram operações de um shard; a contabilidade de bytes enfileirados deixa de precisar de snapshot global. Testes cobrem vidas paralelas, substituição de endereço bruto e independência dos locks. |
| `711ff3ef` | bmjask | D3D12.X: stop stalled sparse ring reclaim cleanly | `wait_for` do ring de upload passa a devolver `bool` e `wait_for_ms` aceita timeout; quando a espera de 5 s falha o ring não tenta de novo para sempre. `retry_sent_range` marca só o sufixo de 64 bytes não submetido, sem invalidar chunks já enviados da mesma página, e uma leitura síncrona aborta se o upload pré-requisito não completou. |
| `95f12a04` | bmjask | D3D12.X: process 2D texture tiling in contiguous block runs | O detiler/tiler 2D passa a calcular, por layout imutável GFX10, a maior sequência horizontal contígua (`run_elems`) e copia blocos inteiros em vez de refazer a colocação por elemento; mantém um bloco de 4 KiB/64 KiB quente e as checagens de limite para origem truncada. Medição no corpo: 1920x1080 R8 mode-5 detile de 8,8–10,3 ms para 0,76–0,87 ms. |
| `4755740b` | bmjask | D3D12.X: complete failed pipeline worker jobs | `run_job` embrulha o build em `catch_unwind` e converte panic em resultado falho, com `panic_message`; peças paralelas também capturam panic e restauram estado local antes de publicar; zero workers ou fila recusada falham o job exato em vez de deixar o aceito sem executor. Startup de preload especulativo que falha fica dormente. |
| `a0780942` | bmjask | Memory: avoid VirtualQuery walks in sparse windows | Introduz cache afirmativo de proteção por tile de 64 KiB no estado esparso (`not_rw`), consultado por `known_writable` em `dstorage.rs` e `eh.rs`; `VirtualAlloc`/proteção publicam `committed_with_protection`/`protection_changed`, commits publicam proteção atomicamente, e casos incertos continuam usando `VirtualQuery`. Handles `*Ex` do processo atual são rastreados por PID. |
| `79cf5d9b` | bmjask | D3D12.X: abort sparse allocator reuse after wait failure | `begin_slot` devolve `Option<usize>` e `begin_slot_with_timeout` falha fechado quando o fence não chega; `Reset`/`Close` passam a propagar erro com `?` em vez de `let _ =`. O readback aborta antes de reutilizar o allocator ou publicar staging cuja cópia não completou. |
| `839b6ea0` | bmjask | D3D12.X: enqueue texture page waits outside global state | O `Wait` nativo da fila do título deixa de ser chamado sob o registro global `PAGES`: cada fila ganha um gate de preparação de vida curta (`wait_gates()`), o valor de fence monotônico é reservado sob `PAGES` e o wait é enfileirado depois; o gate impede submissões da mesma fila de ultrapassar a espera reservada. |
| `fbcd70e7` | bmjask | D3D12.X: separate compute blobs by node mask | `compute_blob_key_for` inclui `NodeMask` na chave em memória; `compute_blob_key` vira o caso `node_mask = 0`. Dois testes cobrem igualdade para 0 e diferença entre 0 e 2. |
| `5cd0c4ae` | bmjask | D3D12.X: isolate ABI roots by device | A raiz de assinatura ABI deixa de ser um único estático do processo: `ABI_RS_DEV` guarda o dispositivo primário e `ABI_RS_BY_DEV` mapeia os demais; as raízes com samplers passam a ser chaveadas por `(device, sampler set)`. Um teste cobre layouts idênticos em dispositivos distintos. |
| `7049cc3e` | bmjask | D3D12.X: share split compute root signatures | Cacheia a raiz de dispatch-chain por `(dispositivo, layout exato de samplers)` num `LazyLock<Mutex<HashMap<...>>>`, para que programas com peça comum compartilhem o blob compilado. A criação continua reportando erro de tradução quando falha. |
| `2f523330` | bmjask | Recompiler: flatten direct-DXIL compute heap reads | Novo `HEAP_FLAT = 0x200` na chave de janelas de memória: o tradutor DXIL direto lê as janelas de heap de estágios de compute sem branch por acesso (`reads_heap_flat` aceita só `Kernel`/`Compute`), enquanto o caminho HLSL remove o flag e fica byte-idêntico (teste compara `hlsl`). `xcl shaders --precompile` e runtime passam a derivar a chave por `windows_for`/`windows_for_stage`. |
| `c22c0ac2` | bmjask | D3D12.X: defer superseded texture unmaps | Mantém `HeldUnmap` para texturas reservadas substituídas: recurso, tiles usados e marcas `(fence, valor)` de cada fila relevante; o unmap NULL só é liberado quando `all_marks_complete` prova que todo trabalho anterior terminou. Se não for possível marcar com confiança, o mapeamento fica vivo (fail-closed). |
| `efdebc82` | bmjask | D3D12.X: separate persisted compute blobs by node | `name_for_root_node`/`path_for_root_node` acrescentam `-n<mask>` ao nome do arquivo quando `NodeMask != 0`; node 0 mantém o nome legado. Teste fixa que o nome legado é igual ao de node zero e diferente para outro node. |
| `60467292` | bmjask | XAudio2: bulk-copy decoded PCM output | Troca o laço sample a sample do decodificador Opus por `append_pcm16_le` (cópia em bloco de i16 little-endian, ignorando byte ímpar final) e `append_f32_le` (conversão escalar preservada). `const _: () = assert!(cfg!(target_endian = "little"))` documenta a premissa. |
| `f043b319` | bmjask | D3D12.X: signal deferred unmaps outside page state | Separa reserva e sinal: `mark_unmap` devolve `(marks, signals)` e os `native_queue_signal` são chamados depois de soltar `PAGES`. Enquanto o sinal não é emitido o valor não está completo, então o polling mantém o mapeamento; falha de `Signal` só loga e mantém o mapeamento. |
| `e8f7164b` | bmjask | Recompiler: keep graphics cache identity under heap-flat | `windows_for_stage` remove `HEAP_FLAT` de vértice, pixel, mesh e geometria, mantendo-o só em compute, para que ligar o caminho rápido de compute não invalide entradas gráficas byte-idênticas. Teste cobre compute e os outros estágios. |
| `9f6d8ebc` | bmjask | Shaders: claim driver-cache builds across processes | Cria `Claim` sobre `<blob>.claim` com handle exclusivo *delete-on-close*: `ClaimAcquire` distingue `Acquired`/`HeldElsewhere`/`Unavailable`, de modo que dois processos de preparação não compilem o mesmo blob; falha de filesystem continua fail-open. Release do claim é imediata também em término forçado do processo (semântica do kernel). |
| `a2d66d6a` | bmjask | D3D12.X: keep two-pass compute on the pipeline node | `stream_node_mask` lê o subobjeto `NODE_MASK` do stream; `finish_two_pass` recebe o mask e o aplica ao `ComputeDesc.NodeMask`, à chave em memória (`compute_blob_key_for`) e ao nome persistido (`name_for_root_node`). Antes o passe escondido era sempre node 0. |
| `733130a0` | bmjask | Shaders: release pipeline permits during unwind | Substitui os decrementos manuais do contador de execução por `RunningPermit` com `Drop` (lock + `saturating_sub` + `notify_one`), de modo que panic na criação de pipeline não vaze slot nem deixe threads presas no teto de concorrência. Teste de unwind verifica contador em zero e aquisição por thread em espera. |
| `d634138c` | bmjask | PE loader: rewrite console wait instructions after first fault | Depois da primeira exceção de `monitorx`/`mwaitx`, o handler substitui o site alinhado por `nop`s/`pause`+`nop`s: `CODE_REWRITE` serializa mudanças de proteção, a substituição de palavra é repetida se houver interferência, bytes adjacentes são preservados, a proteção RX é restaurada e o cache de instrução é *flushed*. `REWRITTEN_WAITS` reconhece exceção já em voo num site recém-reescrito. |
| `78060a89` | bmjask | XAudio2: preallocate decoded Opus output | Calcula a contagem exata de amostras a partir das durações dos pacotes quando o decodificador está em fronteira de stream (`timestamp_100ns() == 0` em todos os streams) e reserva o `Vec` com esse tamanho; especula só nessa fronteira e limita a reserva a `MAX_PREALLOC_SAMPLES` (16 Mi). |
| `8e7b566a` | bmjask | D3D12.X: share idle command allocator spares | Um allocator sem spare ocioso pode tomar um spare do mesmo tipo de outro namespace (`shared_aside_probes`), com snapshot de *backing* e token de reset do doador; as sondas de fence rodam fora do lock do pool e o candidato só é transferido se a geração exata ainda valer. Doador com reset em curso é pulado. |
| `474ca3b5` | bmjask | Shaders: serialize identical in-process pipeline builds | `PipelineCacheIdentity` (len, dois hashes do DXIL, flags, node mask, chained, palavras de sampler, caminho de cache) indexa builds em voo; workers locais com a mesma identidade esperam o build ativo em vez de entrar juntos no driver. O slot é liberado e os waiters notificados em todo retorno e panic. |
| `8626d5ba` | bmjask | D3D12.X: separate indirect dispatch-chain root ABI | Introduz `ChainRoot::{DirectConstants, IndirectCbv}` e o namespace `indirect-chain.pso`, com `ChainSlabBinding` e `dispatch_chain_indirect_root_signature` (raiz CBV) separada da raiz de constantes. É só a fiação: o PSO indireto ainda precisa ser construído à parte porque pipeline state está preso à root signature. |
| `65704ff1` | bmjask | D3D12.X: complete graphics driver cache identity | A chave de blob gráfico passa de `(hash, bytes)` para `(hash, bytes, endereço da root signature efetiva)`; o hash inclui o GS final (inclusive GS nativo), não só o retângulo gerado pelo XCL. Pipelines com stream-output — estado com arrays por ponteiro não representado na chave — deixam de ser cacheados em vez de aliasar layouts. |
| `f2cb30f8` | bmjask | Recompiler: charge split pieces to the shader cache | A admissão no LRU de shaders traduzidos passa a somar `Vec::capacity` das peças de dispatch-chain, a mesma conta que a evicção já fazia. Teste pega o caso em que o shader inteiro só "cabe" se as peças forem ignoradas. |
| `78a05030` | bmjask | D3D12.X: gate medium AMD compute driver compiles | Cria a faixa AMD de compute frio de 128–256 KiB (`AMD_MEDIUM_COMPUTE`) com slots cujo padrão depende da RAM física (`medium_pso_defaults`), configurável por `XCL_MEDIUM_PSO_CONCURRENCY`/`XCL_MEDIUM_PSO_WANTED_CONCURRENCY`; o vendor é registrado em `ADAPTER_VENDOR` e o pipeline de duas passagens toma um único turno externo para as duas chamadas. |
| `1071d4ab` | bmjask | D3D12.X: retain accepted disk pipeline blobs | Um *hit* de disco bem-sucedido passa a admitir o mesmo `Arc` no cache RAM sob o teto de bytes (`DriverBlobs::admit`), com reserva de build e admissão sob a mesma ordem de locks; a rejeição remove só a geração exata usada pelo driver (`forget_*_if`). |
| `66f0ea54` | bmjask | D3D12.X: cache positive mirror range lookups | A tabela de espelhos de vértice/índice é *append-only*: cada thread mantém um snapshot ordenado e a geração (`MIRROR_GENERATION`) responde consultas positivas sem o mutex global; miss continua indo à tabela autoritativa, preservando a primeira faixa que contém o endereço. |
| `ed98461e` | bmjask | D3D12.X: run ready split pipelines through ExecuteIndirect | Para o formato medido de um `Dispatch` por comando, gera comandos de peça na GPU quando a forma *whole* já existe: `note_dispatch_only_signature` marca a assinatura de stride válido, `execute_indirect` fatia a cadeia e mantém os comandos vivos até o reset do allocator. O primeiro uso e outros formatos continuam no fallback síncrono. |
| `057f463a` | bmjask | D3D12.X: retain UMA until its native allocator resets | `native_backing` mapeia o allocator lógico do título para o *backing* nativo que possui o recording, e os blocos UMA pendentes passam a ser liberados só quando a geração daquele backing é provada ociosa e resetada/aposentada; `Reset`, polling de fence, liberação de UMA e `Release` COM ficam fora do estado do pool. |
| `db157fe5` | bmjask | D3D12.X: measure sampled texture residency on activity runs | Só com `XCL_ACTIVITY` ligado, conta texturas amostradas, com SRV, texels residentes/endereçáveis/virtuais em `TexelStats` (atômicos) e emite uma linha `texture residency:` no relatório; sem a variável não há contabilidade nem lock de `PAGES` na criação de SRV. |
| `afa68354` | bmjask | D3D12.X: attribute submitting-thread recording costs | Novo patamar `XCL_STALL_TRACE_DETAIL`: a primeira submissão do título identifica a thread submissora (`note_submitter`) e só nela mede draw, dispatch, bind, barrier, copy e ciclo de vida de lista (`DetailDraw`…`DetailListLifecycle`). O trace grosso continua ativável sozinho e não liga esses pontos quentes. |
| `7f920d4b` | bmjask | README: document activity texture residency totals | Só documenta a linha de residência de texturas do `XCL_ACTIVITY` (residente / endereçável por SRV / virtual), ajustando o parágrafo que descrevia o relatório de memória. |
| `a05fbc49` | bmjask | D3D12.X: build indirect split pipelines on the CBV ABI | Constrói, numa lane de fundo, o conjunto indireto (peças + whole) sobre a raiz CBV: comando indireto passa de 44 para 256 bytes (oito palavras de constantes + `Dispatch` compartilhado pelas peças da rodada), `ComputeBuild::{DirectChain, IndirectChain}` separam o namespace de disco e `IndirectCharge` contabiliza a fonte retida (retenção e admissão limitadas a 64 MiB cada). |
| `d36718ca` | bmjask | Shaders: report driver-cache write failures | Conta tentativas de persistir blob que falharam (`write_failures`), avisa uma vez com o caminho afetado e soma o total ao resumo de preparação, sem mudar a semântica de conclusão. Teste cobre contagem de sucesso/falha. |
| `e9389076` | bmjask | Shaders: prewarm observed indirect split pipelines | Grava em `used-indirect-split.txt` o hash canônico do programa de compute quando uma cadeia split chega a `ExecuteIndirect`; um `--pipelines --used-only` posterior constrói as peças diretas e, para esses programas, também as peças e o whole sob a raiz indireta-CBV. Programas split não observados ficam como estavam. |

## Pull requests abertos (2 acima de #3155)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #3158 | Shaders: retain pipeline-library file buffers without copying | Lê a biblioteca de pipeline nativa como `Arc<Vec<u8>>` em vez de copiar para `Arc<[u8]>`, compartilhando o buffer original entre `pipeline_library` e `warm_library`; acrescenta teste de identidade de alocação. Base `main`, 3 arquivos, +38/−13. O corpo declara 2.132 testes passando e 9 falhando na árvore do PR, os mesmos 9 da `main` `e13d6250`. | Não. É economia de memória/cópia na preparação, não correção de defeito catalogado. |
| #3156 | D3D12.X: XCL's render-target arenas and window pools get a residency priority | Com `XCL_RESIDENCY_PRIORITY=1`, dá `HIGH` aos heaps de render-target/depth e aos pools de janela e `NORMAL` aos pools de página de textura via `SetResidencyPriority`, uma vez na criação de cada heap; `sparse.rs` aplica `HIGH` a pool de janela novo. Base `nvidia-dev`, rascunho, e o próprio corpo diz que os testes serão acrescentados antes de sair do rascunho. | Não diretamente. É a área de residência/evicção (#2688, #3098), não um defeito do catálogo. |

## Pull requests mesclados (7 na janela, 3 novos)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #3157 | Recompiler: keep DispatchX rendezvous branch wave-uniform | `22247810` (2026-11-28T14:06:59Z, base `main`) | `emit.rs`: a ramificação do limite parcial de `DispatchX` passa a ser explicitamente uniforme com `WaveActiveAllTrue`, para que as duas metades de uma wave32 em NVIDIA não percam o *rendezvous* limitado e marquem o grupo como morto; `tests.rs` exige a forma uniforme no fixture de HLSL. Empilhado no #3150; o corpo diz que só `ad19a1c1` pertence a este PR. |
| #3150 | Tests: three split-wave64 / direct-DXIL test expectations that fail on main (#3093) | `7ea525a6` (2026-11-28T14:06:40Z, base `main`) | Conserta a checagem de peça split para procurar `ResBind { i32 0, i32 2, i32 2, i8 1 }` em vez da forma antiga, e ajusta `heap_window_tests.rs` e `dxil/module/split.rs` (3 arquivos, +41/−15). É o teste que o #3157 empilha. |
| #3149 | D3D12.X: a list's own GPU writes land in a sparse chunk the CPU changed | `b937fbab` (2026-11-28T14:07:15Z, base `main`) | `sparse.rs` (+40/−11): o copy-back depois de `ExecuteCommandLists` passa a ler também os intervalos que os comandos da própria lista escreveram (`sparse_writes`, registrados por `note_write`), e um chunk alterado que recebeu escrita de GPU mantém o hash antigo — a próxima passagem de write-watch reenvia o chunk inteiro, com os bytes da GPU e da CPU. `readback_now` continua preservando os bytes da CPU. Novo teste `gpu_written_bytes_land_in_a_chunk_the_cpu_changed`. O corpo declara 5/5 testes e2e e 3/3 de `land_page`; a suíte completa e o smoke **não** foram rodados. |
| #1908 | (já documentado às 13:00) | — | `fix/msaa-sample-clamp-main`. |
| #1873 | (já documentado às 13:00) | — | `fix/geometry-setpc-boundary`. |
| #3102 | (já documentado às 13:00) | — | benchmark config. |
| #1809 | (já documentado às 13:00) | — | huge shader compile. |

A contagem de merges (7) não bate com a de commits documentados nesta página (40): os 40 vêm da
página do `compare`, que não é ordenada por merge, e nenhum deles é um commit de merge. A divergência
está registrada na seção Confiança.

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum. Os hashes dos 40 commits e dos 7 merges foram conferidos por prefixo
  contra as colunas `introduzido` e `corrigido` de `catalogo/defeitos.csv` (305 linhas): zero
  interseção. Nenhum assunto cita `D-xx`.
- **Abre defeito novo?** Sem evidência. O que há é **área compartilhada**, e vai marcado como
  hipótese, não como fato:
  - `ba44b52c`, `fbcd70e7`, `efdebc82`, `5cd0c4ae`, `65704ff1`, `2f523330` e `e8f7164b` são
    literalmente a área de **D-11 do XCL** (a chave do cache de artefato de driver não incluía as
    fontes do lowering). Aqui a chave ganha samplers, node mask, dispositivo, root signature e o
    flag de heap; não há prova no diff de que a entrada de D-11 (fontes do lowering) esteja coberta
    — é o mesmo tipo de identidade, não o mesmo campo.
  - `78a05030` mexe no agendamento de compilações AMD (`amdxc64`), que é o palco de **D-02 do XCL**
    (codegen derrubava o `amdxc64`). O diff limita concorrência e memória de compilação; não toca no
    codegen que causava o crash.
  - `ed98461e`/`a05fbc49` usam `ExecuteIndirect`, área de **D-20 do XCL** (argumentos indiretos lidos
    antes da barreira). O diff constrói comando indireto próprio e não altera a ordem barreira/leitura
    do caminho do título.
- **Regressão?** Nenhuma detectável no diff. Dois pontos que merecem acompanhamento e **não foram
  testados aqui**: `a05fbc49` reduz a retenção de fonte indireta a 64 MiB (duas cotas), e
  `9f6d8ebc` deixa falha de filesystem fail-open — nos dois casos o próprio corpo declara o limite.

## Confiança

**Verificado no diff (API de commits, arquivo por arquivo):** os 40 diffs acima; a autoria e a data
de cada um; o merge `22247810` e o corpo do #3157; o estado de #3158 e #3156 (base, arquivos, corpo).

**Não verificado:** execução de qualquer teste (nenhum foi rodado nesta coleta; os corpos citam
`cargo check` para o alvo Windows GNU e ausência de execução nativa em quase todos); o comportamento
em GPU real; a equivalência entre o que o corpo do PR promete e o que o merge entrega além do que o
diff mostra.

**Lacunas de cobertura, explícitas:**

- **Ordem do `compare` não é estável.** O `commit_cursor` gravado (`e5266254`) aparecia no índice 2
  da página atual, não no 40. Por isso a janela foi escolhida por hash — os 40 primeiros commits da
  página cujo SHA não aparece em nenhum documento de `docs/updates/` — e não por índice. O próximo
  cursor é `be9d40579896933e8acccb6ef2d73e02bf8c4bc9`, o 41º não documentado da página.
- **211 commits não documentados.** A página do `compare` devolve 233 commits (de `ahead_by` 233 a
  partir de `last_commit`; 232 a partir de `commit_cursor`); 22 já estavam em documento anterior e
  40 entraram aqui. Restam 171.
- **Merges x commits.** 7 merges na janela contra 40 commits documentados; nenhum dos 40 é merge, e a
  soma não fecha por construção porque a página do `compare` não ordena por merge.
- **`last_run` antigo.** O cursor de mesclados gravado era `2026-11-28T04:12:54Z`, mais antigo que
  merges já documentados às 13:00 (#1809/#1873/#3102/#1908). Eles foram reconhecidos por número nos
  documentos anteriores e não foram reescritos; o novo `last_run` avança até o maior `merged_at`
  efetivamente coberto (`#3149`, 14:07:15Z, documentado acima e verificado no diff).
- **Nenhum subagente disponível** neste ambiente; os quatro repositórios desta execução foram
  processados em sequência. Somente leitura na API do `xcl`.
