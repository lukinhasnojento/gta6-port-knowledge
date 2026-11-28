# ivall/xcl — atualização de 2026-11-28 13:00

## Resumo

Quarenta commits novos a partir do `commit_cursor 3dbac13a`, todos de 2026-11-26 entre 06:54Z e
08:24Z — a série D3D12.X de **lock/ownership**: cada commit tira uma chamada de driver (fence
`GetCompletedValue`, `CopyDescriptorsSimple`, `Signal`, `Reset`, `ExecuteCommandLists`) de dentro de um
lock global e a substitui por "fotografa donos fortes, consulta fora do lock, revalida a geração
exata antes de publicar". O padrão se repete em 20 dos 40 commits, com testes de corrida em quase
todos. Junto vêm quatro consertos independentes: `5ee15471` (objetos submetidos são retidos quando a
espera de fallback falha), `15aa7f36` (dobra de `select` guardado no DXIL), `fb5e7d15` (deleção de
blob de driver em volume sem `FileDispositionInfoEx`) e `8e3755a1` (journal de pipelines usados
sobrevive à morte do worker). O repositório **não avançou durante a coleta**: o head medido foi o
mesmo `e429936c`, então estes 40 são backlog, não trabalho novo. Ele voltou a andar **depois** da
janela: às 13:23–13:24Z entraram 13 commits (quatro merges: #1809, #3102, #1873, #1908), que não estão
neste documento — ver Confiança.

## Commits novos (40)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `2d8c09d9` | bmjask | D3D12.X: match texture downloads to writer queues | `title_textures.rs`: o plano de download passa a carregar `queue_type`; só `DIRECT` ou `COMPUTE` são aceitos (`COPY` recusado por `download_queue_type_supported`), e allocator/lista são criados com esse tipo em vez de sempre `DIRECT`. É preparação não ativada de download ordenado antes do `Signal`. |
| `a0762934` | bmjask | D3D12.X: preserve extended rasterizer stream state | `real.rs` (+280): quando o stream gráfico usa `D3D12_RASTERIZER_DESC1`/`2` com `DepthBias` que não é inteiro exato, ou modo de linha diferente de `ALIASED`, o pipeline passa por `GraphicsStream` (tradução nativa adaptada do PR #2621) em vez do descritor legado; streams ilegíveis falham fechado e o fallback de lista de retângulos fica restrito a `E_INVALIDARG`. |
| `d3e60ecb` | bmjask | D3D12.X: report memory by tracked owner | Arquivo novo `memory_report.rs` (197 linhas): linha `memory by owner (MB)` separando commit, VRAM local e não-local por dono XCL, com `other` calculado por subtração — negativo expõe amostra sobreposta ou obsoleta. Memória esparsa passa a contar páginas de 4 KiB comprometidas; totais saturam. |
| `d9403543` | bmjask | D3D12.X: launch DispatchX thread dimensions safely | `DispatchX` com `USE_THREAD_DIMENSIONS` chega ao replay diferido: contagens de thread viram grupos inteiros, os últimos grupos parciais são mascarados por ballot nos dois emissores, e dimensões D3D grandes ou forma de grupo desconhecida são puladas antes de ligar/gravar. `TRANSLATION_GLUE_REVISION` sobe para `pso-glue-v3`. |
| `7a37be8f` | bmjask | D3D12.X: serialize each queue submission turn | Arquivo novo `queue_submit.rs` (175 linhas): cada fila ganha um *lease* reentrante por geração (dono = `ThreadId`, profundidade), porque o XCL emite trabalho de dentro de uma chamada da fila do título. `ExecuteCommandLists` nativo, allocator e publicação de estado/escrita de textura passam a ser um turno; sinal e espera usam o mesmo primitivo. |
| `5fb57918` | bmjask | D3D12.X: cache native extended graphics streams | Streams gráficos traduzidos ganham identidade canônica sem ponteiro e cache de blob de driver limitado (RAM ou disco); a recusa de um blob em cache é retentada antes do fallback de GS de lista de retângulos. A invalidação passa a ser por geração exata (`Weak` no mapa persistido), para que uma rejeição atrasada não expulse um blob mais novo. |
| `fa87c6c9` | bmjask | D3D12.X: measure contended queue submission turns | Três linhas: só registra span de `LockContention` quando outra thread é dona do mesmo lease; operação não contendida não abre span. |
| `c3d701a6` | bmjask | D3D12.X: call native queue slots for internal work | `Execute`, `Signal` e `Wait` do próprio XCL passam a chamar os slots originais da fila do PC (`native_queue_execute/signal/wait`) em vez de poderem cair em wrapper deslocado de layout adotado ou em stub que devolve sucesso. Onze arquivos trocam `queue.Signal(...)` por esses helpers; um teste sintético cobre o caso da vtable deslocada. |
| `47b8e3fd` | bmjask | D3D12.X: capture texture transitions in submission plans | Transições de textura rastreadas saem de notas por endereço de lista e passam a efeitos do segmento físico, com geração: `list_x` guarda `texture_transitions`, limpa no reset, e publica estado + dono de escrita GPU atomicamente só depois de o gate nativo aceitar cada pedaço. Remove `title_textures::list_reset` do ciclo de criação/liberação. |
| `652e9cc1` | bmjask | XAudio2: remove locks and copies from source submission | `xaudio2.rs` (+214): tabela atômica fixa de 32 originais de vtable patcheada, voz com PCM16 próprio entregue sem segunda cópia (`prepare_decoded_pcm`), mapas de voz vazios ignorados, e silêncio de fallback limitado a 64 KiB (`OWNED_SILENCE_BYTES`) quando o mapeamento grande não pôde ser alocado. |
| `ec5c3e21` | bmjask | DirectStorage: precompute BCPack bit packing | `bcpack/victory.rs`: `SPREAD4`/`SPREAD8` como tabelas `const` e máscaras por campo substituem os laços por bit, residual e âncora; `Widths` passa a guardar máscaras e comprimentos ativos em vez de larguras. O corpo afirma saída idêntica, com hashes de linha de base dos fixtures e de todos os modos BC6H/BC7. |
| `01fad57f` | bmjask | XAudio2: reuse validated Opus multistream scratch | `opus.rs` (+293): `split_self_delimited_into` reaproveita o `Vec` de saída; `split_multistream_into` valida **todos** os subpacotes (e a duração de 48 kHz de cada um) antes de devolver qualquer coisa, para o decodificador nunca consumir pacote parcialmente válido; mapa de canais é pré-calculado com `255`/fora de faixa → silêncio. |
| `c18d8122` | bmjask | D3D12.X: detach heap state before final Release | `console_heap` separa `take()` de `free()`: o estado endereçado por objeto é removido **antes** do último `Release` nativo (o endereço pode ser reutilizado), e a memória só é liberada depois de o driver chegar a zero. `release_taking_first` sonda `AddRef == 2` e cai no `take()` pós-free quando a liberação simultânea faz a sonda errar. |
| `8de71805` | bmjask | D3D12.X: count driver pipeline builds in the window title | O título da janela contava só DXC; agora `compiling_text` mostra `DXC N running, M done | driver N building (W waiting for a slot), M done`. `DriverBuilds` conta `Create*PipelineState` sem blob (`BLOB_SUPPLIED` marca a thread quando o driver recebe blob e não conta como build). |
| `c3da8f3d` | bmjask | D3D12.X: poll allocator fences outside global state | `GetCompletedValue` pode entrar no driver e bloquear ou reentrar; `note_resubmits` passa a fotografar referências fortes de fence e geração de lista sob o lock, consultar fora, e revalidar `same_submit` (fila, fence, valor, recording) antes de contar ou avisar. |
| `5ee15471` | bmjask | D3D12.X: retain submitted texture staging after wait failure | `drop_after_wait`: se o `Signal` da fence de aposentadoria falha e a espera de fallback **também** falha, os objetos de comando e o staging são deliberadamente vazados (`mem::forget`) em vez de liberados, porque o trabalho submetido ainda pode referenciá-los. Teste com contador de `Drop`. |
| `3b86ad5c` | bmjask | D3D12.X: bypass title hooks for internal queue execution | Submissões internas do XCL (uploads de sparse, readbacks, uploads de textura, zeragem de tile) passam direto pelo slot nativo quando estão em `own_work()`, pulando `present::note_execute` → `late_release::drain_known` → `vram::recycle_ready`; sem isso, um `ExecuteCommandLists` interno chamado com locks de sparse/VRAM presos reentra neles e trava. |
| `15aa7f36` | bmjask | Recompiler: fold unobserved EXEC selects in direct DXIL | `module/opt.rs` ganha `fold_guarded_selects`: uma escrita de VGPR preserva lanes inativas como `select(exec, novo, antigo)`, mas se o valor só é observado sob a mesma condição, a perna preservada pode ser lida e o DCE remove os selects mortos. `lane_local` exclui divisão inteira indefinida, memória e operações que observam outras lanes. `XCL_DXIL_NO_OPT` aceita `select`. |
| `d59c05d5` | bmjask | D3D12.X: signal allocator retirement fences outside global state | A fence por fila vira `Arc<QueueFence>` com `next_value` atômico monotônico; criação e `Signal` acontecem no turno da fila, com o lock de estado solto, e uma geração substituída mantém o trabalho preso em vez de liberá-lo. |
| `30766d09` | bmjask | D3D12.X: publish GPU texture writes at title fence completion | Antes de o título sinalizar a fila que escreveu subrecursos suportados, planos de readback imutáveis são construídos, validados e reservados no turno daquela fila, e os readbacks entram **antes** do sinal; o espelho da fence só publica o valor depois de `complete_fence_downloads`. `GetCompletedValue` passa a ser hookado. |
| `1229ce48` | bmjask | D3D12.X: publish single descriptors outside shared heap locks | `publish()` não chama mais `CopyDescriptorsSimple` segurando os registries: resolve cada página e aluga cada faixa de destino sob `HEAPS` (geração exata + dono forte da origem) e emite as cópias fora dos locks. `PublishLeases` adia liberações que se sobrepõem a escritas ativas. |
| `8bf0d6fa` | bmjask | D3D12.X: include texture refresh in submit stall timing | Uma linha: o span `SubmitPreparation` começa **antes** do refresh de texturas, para que varredura de escrita, detiling e upload apareçam no breakdown grosso de stall. |
| `fb5e7d15` | bmjask | D3D12.X: fall back when extended cache deletion is unavailable | Em filesystem que não implementa `FileDispositionInfoEx` com semântica POSIX, a deleção cai para `FileDispositionInfo` no **mesmo handle** aberto com DELETE, e confere `DeletePending`; assim a rejeição invalida a geração exata do blob carregado e não um caminho substituído. |
| `fab5da17` | bmjask | D3D12.X: publish descriptor ranges outside heap locks | Publicação de faixas: resolve e aluga todas as corridas de destino sob a ordem de lock existente, retém a geração do heap de origem e emite `CopyDescriptorsSimple` em lote sem lock de allocator. |
| `a019f55f` | bmjask | Tests: model allocator fence references | Só teste: o fixture de lista liberada ganha objeto COM com contagem de referência real (`refs`, `final_releases` atômicos) e vtable em `Box::leak`, porque as sondas de allocator sem lock passaram a clonar a fence. |
| `95348a34` | bmjask | D3D12.X: poll allocator backing fences outside global state | `BackingProbe` fotografa geração, usuários, marcas (fence+valor), pins e dono de liberação; reset/aposentadoria/liberação consultam fora do lock e revalidam namespace e todos os campos antes de reusar. `namespace_probes`/`namespace_matches` substituem o teste `Backing::free` que consultava fence sob lock. |
| `7d9f21f8` | bmjask | D3D12.X: promote deferred compute before list preparation | `defer()` publica a demanda (`need_by_submit(pso)`) **antes** de dividir e preparar a lista, para que a promoção do worker e a admissão do compilador ocorram em paralelo; o teste verifica a ordem (`demanded` já é 1 dentro do `split`). |
| `5e525c99` | bmjask | D3D12.X: serialize title texture uploads per queue | O cache de upload de textura do título deixa de usar um mutex global: `ON_QUEUE` guarda `Arc<Mutex<Option<QueueUploads>>>` por fila, e os `InFlight` aposentados são largados depois de o lock da fila sair. |
| `4b01fde4` | bmjask | D3D12.X: poll VRAM retirement fences outside registry locks | `RetireQueue::completion_fences()` devolve donos fortes de toda fence que pode mudar uma decisão de varredura; `retire_heaps_and_sweep` consulta fora do registry e varre os lotes contra essas observações — marca nova ou concorrente continua aposentada. |
| `c4f866a1` | bmjask | D3D12.X: publish texture downloads outside the fence registry | `FenceDownloadState` com `processing` e `tying`: um lote de downloads é destacado sob o mutex e publicado fora dele (poll, mapeamento, destiling reverso, escrita na memória do título); trabalho não publicado é restaurado, e `SAFE_FENCE_VALUES` só cresce (`note_safe_value` com `max`). |
| `1c24a0d5` | bmjask | D3D12.X: isolate VRAM retirement work per device | `RETIRE` passa a mapear dispositivo → `Arc<Mutex<RetireQueue>>`; o registry só é segurado para alugar o estado, e criação de fence, `Signal`, poll e destruição de COM ficam fora. Geração removida continua viva enquanto operações ativas a seguram. |
| `34273586` | bmjask | D3D12.X: reuse bounded workers for texture detiling | Pool de detiling de vida longa (metade dos núcleos, 1..8 threads) em vez de criar e juntar até oito threads por lote. Se um worker falhar no meio, `requeue_failed_preparation` devolve à fila só a geração exata de textura CPU e limpa o fingerprint de conteúdo prematuro. |
| `8303b3a1` | bmjask | D3D12.X: reset allocator backings outside global state | `Reset`, `GetDevice` e `CreateCommandAllocator` saem do lock global: uma operação de reset é reservada por token (`resetting`), com donos fortes dos candidatos; se namespace ou qualquer campo do backing mudou, a operação é cancelada e o rollback é feito fora do lock. |
| `20748ea6` | bmjask | D3D12.X: poll sparse retirement fences outside global state | `poll_unique` deduplica fences e consulta fora de `STATE`; `drain_quarantined_with`/`retire_free_pools_with` recebem as observações e só drenam o prefixo FIFO satisfeito por elas — fence nova ou marca maior continua em quarentena. |
| `15083245` | bmjask | D3D12.X: enqueue sparse title waits outside global state | A espera nativa da fila do título deixa de ser chamada com o estado esparso preso: a reserva monotônica é feita sob o estado, o estado é solto, e a espera é enfileirada depois, serializada por um gate por fila (`WAIT_GATES`) para que uma submissão concorrente não ultrapasse quem reservou. |
| `2b395256` | bmjask | Recompiler: prune unused split-piece resource bindings | Peças de shader dividido herdavam a metadata de recursos do shader inteiro; `referenced_bindings()` lê os `dx.op.createHandleFromBinding` alcançáveis e `prune_unused_bindings` mantém só a metadata casada (descritor dinâmico mantém a tabela completa). O fixture de paridade confere que peças só-scratch omitem o binding de memória do título. |
| `31930d85` | bmjask | D3D12.X: retire spare recordings and reclaim idle allocators | `retire_recording` desanexa a lista do backing sem tocar em lifetime de submissão; allocators postos de lado são resetados fora do estado, com marca `already_reset` e token de namespace para que um Reset só aconteça uma vez. |
| `8e3755a1` | bmjask | D3D12.X: preserve used pipelines when the writer stops | `used_journal.rs`: cada escrita do arquivo é seguida de `flush`; se o worker não sobe ou morre, as chaves ordenadas ficam em memória e são anexadas num ponto de flush, com o marcador `!` invalidando um fragmento hexadecimal interrompido para que ele não vire rank de preload falso. |
| `b11f4df1` | bmjask | D3D12.X: count queued pipeline builds without a frame scan | Cada `Build` visível ao título carrega `tracked` e `queue_state` (`IDLE`/`ACTIVE`/`DONE`); `note_queued` publica o contador antes de `ACTIVE` e `finish_queued` desconta uma vez só, acabando com a varredura O(N) de stand-ins adormecidos a cada present. |
| `4e59a814` | bmjask | D3D12.X: reuse pending pipeline entries on bind | Um bind procurava o mesmo stand-in duas vezes no mapa global e pipelines de compute já concluídos entravam no caminho de espera; agora resolve pela primeira entrada `Pending` retida e devolve o pipeline real depois de um `acquire` load, mantendo os caminhos de orçamento/`spared`/falha. |

## Pull requests abertos (62)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #3076 | D3D12.X: command list #142 applies the quad sample mask | `SetQuadSampleMaskX(u64)` é stub na `main` e todo pixel é desenhado; o PR aplica a máscara. | Não. |
| #3079 | D3D12.X: copy out-of-window two-pass indirect arguments into the window | ExecuteIndirect de duas passadas (DispatchMesh) com argumentos/contagem num buffer de PC fora das janelas era pulado; o PR copia para dentro. | Área de **D-20** (argumentos lidos antes da barreira), não a mesma falha. |
| #3081 | DirectStorage: gate BC7 rotation by mode | Os bits 3–4 do BC7 só codificam rotação nos modos 4 e 5; o port atual lê como rotação em todo modo. | Não. |
| #3082 | PE loader: a title's CLZERO loops run as native code on a CPU without CLZERO | Port de #2892: em CPU sem `CLZERO` (todo Intel) cada corrida de 12 bytes do título vira código nativo. | Não. |
| #3083 | Launcher: resolve relative XCL_DATA_ROOT before starting the host | `XCL_DATA_ROOT` relativo fazia o launcher escrever `boot.json` no diretório errado. | Não. |
| #3087 | ran cargo fix --lib -p xcl (fix deprecated methods) | `fetch_update` → `try_update` em `hle/xgameruntime`; menos ruído de compilação. | Não. |
| #3088 | README: document xcl shaders --prepare-corpus | Só documentação de um comando que o HELP já lista. | Não. |
| #3089 | D3D12.X: the depth plane of an R24G8 buffer has a twin | `XCL_DEPTH_UAV_TWIN=1`: o plano de profundidade de um R24G8 ganha twin desenhado antes de o DSV ser ligado (#970). | Não. |
| #3091 | Shaders: share owned bytecode across derived and pending pipelines | Pipelines derivados e receitas pendentes retêm cópias imutáveis compartilhadas dos bytes de shader. | Não. |
| #3092 | Memory: use a 2 MiB grain for allocator page size two | Corrige o arredondamento de `PageSize=2` de 4 MiB para 2 MiB. | **Sim, por área:** é a mesma correção que o `Xbox2Windows` fez em `0f43029b` (layout 261000), mas em `XCL`. Sem linha no catálogo do XCL. |
| #3094 | D3D12.X: a sampled texture in the memory of a written one is copied from it | `XCL_ALIAS_COPY=1`: textura amostrada colocada na memória de uma escrita de outro formato é copiada na GPU. | Não. |
| #3095 | README: document the automatic coherent-window cap | Documenta `XCL_COHERENT_AUTO_CAP` (commit `abf7e177`, já documentado). | Não. |
| #3096 | Recompiler: heap-window shaders reach all 16 window slots of the heap | Com `XCL_HEAP_WINDOWS=1`, a janela k vem do descritor `MEM_HEAP_BASE + k`; o heap reserva 16 slots e o tradutor usava menos. | Não. |
| #3097 | D3D12.X: XCL_VIEW_MEM=1 maps the title's GPU-view memory into heap windows 4..15 | Empilhado em #3096: mapeia a memória de views do título nas janelas 4..15. | Não. |
| #3098 | D3D12.X: evict top texture mips under local VRAM budget | `XCL_TEXTURE_BUDGET_EVICT=1`: evicção automática dos mips de topo sob orçamento de VRAM local. | Não. |
| #3099 | Wine: raw internal fences + native-vtable guards + XG CopyFromSubresource 7-arg ABI | Uso interno de D3D12.X correto contra vkd3d-proton que identifica objetos por igualdade de vtable. | Não. |
| #3100 | Shaders: import exact PSOs from prior libraries | Uma mudança de recibo semântico descartava a biblioteca nativa anterior mesmo com PSOs exatos compatíveis; o PR importa os exatos. | Área de **D-11** (entradas de cache sem as fontes do lowering), sem prova. |
| #3101 | Shaders: verify native two-pass recipe receipts | O cache nativo grava arquivos `gfx-` de mesh e compute com hash; o índice de recibos exigia nomes que ninguém escreve. | Não. |
| #3102 | Tests: construct benchmark config without private fields | A CI do Windows não compila o benchmark porque `Config.input_profile` é privado. | Não (dívida de teste). |
| #3103 | Recompiler: count split shader pieces in the NVIDIA cache budget | Porte do conserto de admissão: inserção omitia capacidade de peças do dispatch-chain enquanto a evicção subtraía. | Não. |
| #3104 | D3D12.X: publish CPU writes only after GPU completion | `nvidia-dev`: escritas de CPU no fim do pipe esperam conclusão confirmada em vez do fallback de 10 s. | Não. |
| #3105 | D3D12.X: stage upload arguments for indirect mesh draws | Habilita argumentos/contagem em heap UPLOAD para mesh indireto de duas passadas em `nvidia-dev`. | Não. |
| #3106 | README: document XCL_UMA_WINDOWS and XCL_HEAP_WINDOWS | Documentação dos switches de janela de memória. | Não. |
| #3107 | D3D12.X: recycle driver timing slots after thread exit | O ramo NVIDIA dava 64 slots para a vida do processo; o PR libera o slot quando a thread sai. | Não. |
| #3110 | D3D12.X: trim idle title_list spares and put-aside command allocators on PresentX | Em `amd-dev` com RX 6650 XT, `SegmentBook::spares` cresceu a 1.815–4.096 spares num carregamento longo. | Não. |
| #3111 | D3D12.X: drop CopyScratch resources outside COPY_SCRATCH lock | Fixes #3069: `CopyScratchCache` largava wrappers COM dentro do lock global e reentrava. | Não. |
| #3112 | D3D12.X: add opt-in XCL_SPARSE_VRAM_CAP_MB | Faz pools esparsos transbordarem para memória de sistema; desligado por padrão. | Não. |
| #3113 | README: document XCL_PREPARE_WORKERS and XCL_PIPELINE_THREADS | Documentação de concorrência da preparação. | Não. |
| #3116 | D3D12.X: a new texture with lazy levels has its tiles mapped by a call | Com `XCL_TEXTURE_LEVEL_PAGES=1` o NVIDIA perde o device (`0x887a0006`) ao entrar no mundo; o PR mapeia os tiles por chamada. | Área de **D-15** (perdas de device/streaming), hipótese. |
| #3118 | README: document XCL_NETWORK=0 | Documentação do switch offline. | Não. |
| #3119 | D3D12.X: a removal with no fault recorded is not logged as a fault at VA 0 | `log_device_removed` registrava a page fault do DRED incondicionalmente; agora só quando há fault. | Não. |
| #3120 | Shaders: a Series S cache starts from the manifests both profiles shared | O `lockhart` mantém manifests próprios e o PR semeia a partir do que ambos os perfis compartilham. | Não. |
| #3122 | D3D12.X: retire idle NVIDIA command pools between presents | Listas divididas ociosas e allocators alternativos ficavam presos depois da gravação, inclusive com o mundo parado. | Não. |
| #3123 | Memory: defer private physical-home GPU tiles on NVIDIA | Adaptação de #1916: tiles físicos privados postos em memória do host, diferidos. | Não. |
| #3124 | D3D12.X: fail lazy texture placement when mappings are unfinished | Alternativa a #3116: falha explícita quando o mapeamento de tiles não terminou, preservando o contorno de mapeamento NULL. | Área de **D-15**. |
| #3125 | D3D12.X: retain resources when retirement fences fail | Mantém recursos e tiles esparsos presos quando a conclusão da fila não pode ser estabelecida — a remoção de device é `UINT64_MAX`, que era tratada como conclusão. | **Sim, por área:** é a mesma classe de `5ee15471` desta janela. |
| #3128 | D3D12.X: ExecuteIndirect checks of a run of draws share one pass | Cada ExecuteIndirect checado gravava barreira UAV total + `Dispatch(1,1,1)` de contagem + ...; o PR compartilha uma passada. | Área de **D-20/D-21**. |
| #3130 | D3D12.X: preserve GPU wait watchdog wakeups on nvidia-dev | Backport de #2499 (Tiago Leal). | Não. |
| #3131 | Shaders: prepare whole split fallbacks on nvidia-dev | Adaptação de #2551 (René). | Não. |
| #3132 | D3D12.X: release replaced two-pass entries outside the lock on nvidia-dev | Backport de #2882. | Não. |
| #3133 | Shaders: prepare only previously used compute pipelines at startup | `XCL_PREPARE_ALL=1` desliga; prepara no arranque só o que o histórico diz ter sido usado (parte de #2088). | Não. |
| #3134 | D3D12.X: the newer queue layout's Signal2X and WaitX signal and wait | Slots #34 e #37 do layout novo de fila passam a sinalizar/esperar de fato. | Não. |
| #3135 | D3D12.X: console device slots #83-#126 run the PC's newer device methods | A vtable do device do console continua com os métodos novos do PC em ordem fixa. | Não. |
| #3136 | D3D12.X: console device slots for the *1 / *2 resource methods use the console's descriptions | Os métodos `*1`/`*2` de recurso usam as descrições do console. | Não. |
| #3137 | D3D12.X: console pipeline-state flags reach the PC as PC flags | O enum de flags de PSO do console não é o do PC (`TOOL_DEBUG` 0x8000_0000 etc.); traduz. | Não. |
| #3138 | D3D12.X: log pipeline-state streams that use console subobject types | Loga streams que usam numeração de subobjeto do console acima do tipo 22. | Não. |
| #3139 | D3D12.X: log the first title call of each command-list slot the newer layout moved | O layout novo insere dois marcadores em #59/#60 e desloca o resto; loga a primeira chamada de cada slot movido. | Não. |
| #3140 | D3D12.X: console Resource and Heap methods past the PC's run their console meaning | Depois de #14 a vtable de Resource do console difere (getter de descrição em #15, etc.). | Não. |
| #3141 | D3D12.X: the console's acceleration-structure resource flag becomes UAV | A flag 0x80 do console é buffer de estrutura de aceleração; no PC é referência de vídeo. | Não. |
| #3142 | D3D12.X: console query-heap types become the PC's | Enums de query heap diferentes: 7/11 do console são timestamp de copy engine. | Não. |
| #3143 | D3D12.X: console formats 187-191 become the PC formats of the same size | Formatos no topo da faixa DXGI diferem entre console e PC. | Não. |
| #3144 | D3D12.X: console device #124 CreateCommandQueue1 creates a console queue | #124 (PC #75) é `CreateCommandQueue1`. | Não. |
| #3145 | D3D12.X: a console DispatchX or padding that ends a command signature reaches the PC | `create_command_signature_x` mapeava só o tipo 105; `DispatchX` é 104. | Área de **D-15**? não — é assinatura de comando; **D-21** é execute indireto de compute não traduzido. Hipótese fraca. |
| #3146 | Frame clock: XCL_FRAME_PERIOD=1 holds an origin while the period's frames are unflipped | `SetFrameIntervalX` com periodicidade 1–4 é quantos intervalos um frame pode levar até o flip. | Não. |
| #3147 | XGameRuntime: the XG texture address computer has all 14 methods | A tabela tinha 12 métodos; chamadas de #12/#13 liam além do fim. | Não. |
| #3148 | kernelx: XMemGetWorkingSetStatistics fills the fast-memory fields | Só `+0` e `+8` eram escritos; engines que dimensionam pools pelo par de memória rápida liam lixo. | Não. |
| #3149 | D3D12.X: a list's own GPU writes land in a sparse chunk the CPU changed | Desde #2959 `land_page` preserva bytes de CPU de todo chunk de 64 bytes mudado; o PR faz a escrita da própria lista pousar. | Não. |
| #3150 | Tests: three split-wave64 / direct-DXIL test expectations that fail on main | Conserta três expectativas de teste que falham na `main` (inclui o check de `2b395256`). | Não (dívida de teste). |
| #3151 | D3D12.X: console pipeline-state streams are parsed with the console's subobject numbers | Acima do tipo 22, os subobjetos do console têm outra numeração. | Não. |
| #3152 | D3D12.X: console DMA-list methods past CopyTiles run their PC equivalents | Os métodos da lista de DMA a partir de #15 eram stubs que devolviam `S_OK`: cópias e fills de fila de cópia nunca aconteciam. | Não. |
| #3154 | Recompiler: preserve compute prologue dominance in direct DXIL | O prólogo de compute pode ramificar antes do primeiro sampler estático; em lowering compacto a máscara inicial de EXEC também. | Não. |
| #3155 | README: document XCL_ASYNC_PSO | Documentação de um switch que já existe em `pso_async.rs`. | Não. |

## Pull requests mesclados (19)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #3085 | xcl run: preparation workers count the commit left in MiB, not bytes | `4577f09c` | `shaders.rs` +18/−1: a linha de commit restante passa a ser em MiB. Confere. |
| #2959 | Tests: compare benchmark regression thresholds | `b0b041fd` | 85 arquivos, +48.456/−5.103: infraestrutura de benchmark (benches, CI, README) e comparação de limiares. Muito maior que o título sugere. |
| #2960 | D3D12.X: read wait cells without faulting | `d85f2c6c` | `list_x.rs` +79/−4: leitura de células de espera sem provocar fault. Confere. |
| #2961 | D3D12.X: own sparse planning observations | `f2c0d36f` | `sparse.rs` +77/−3: o planejamento esparso passa a possuir as observações que usa. Confere. |
| #2962 | D3D12.X: keep unlocked sparse planning inert | `b801633e` | `sparse.rs` +190/−3: planejamento feito fora do lock fica inerte até ser revalidado. Confere. |
| #2963 | Tests: identify benchmark result environments | `f1171e4e` | `benches/support` +123/−2 e `build.rs` +13: o resultado de benchmark passa a identificar o ambiente. Confere. |
| #2964 | Shaders: fail closed on wave64 rendezvous timeout | `86e83e76` | `dxil/lanes.rs` +60/−18 e `recomp/emit.rs` +45/−8: reunião de wave64 que estoura passa a falhar fechado. Confere. |
| #2965 | D3D12.X: correlate submissions with GPU completion | `336fddda` | `real.rs` +215/−2, `alloc_pool.rs` e `list_x.rs`: correlação de submissão com conclusão de GPU. Confere. |
| #2872 | Recompiler: the pixel prelude's quad ballot has no loop for DXC to unroll | `5d2ea8f8` | `recomp/emit.rs` +20/−7: é o conserto `2dad32e6` já documentado às 12:00 (o laço `[unroll]` do `ballot_px_quad` que quebrava o `amdxc64`). O merge fecha o PR. |
| #2782 | D3D12.X: fence-quarantine released shared descriptor ranges | `14292a3f` | `console_heap.rs` +520/−34: faixas compartilhadas liberadas ficam em quarentena até a fence. É a base do `1229ce48` desta janela. |
| #2898 | D3D12.X: pack sparse pool tile reuse into the fullest heap | `814298b1` | `sparse.rs` +190/−29: reuso de tile empacotado no heap mais cheio. Confere. |
| #3068 | UMA: cap the coherent window at RAM/8 below 24 GB of RAM | `392db2cf` | `gpu/uma.rs` +40/−1: é o `abf7e177` já documentado (teto RAM/8 abaixo de 24 GB). |
| #3005 | D3D12.X: XCL_UMA_DRIVER=mixed keeps part of the coherent window in VRAM | `c536d81e` | `uma.rs`, `real.rs`, `texture_upload.rs`, `title_textures.rs` +593/−22: mantém parte da janela coerente em VRAM. Confere. |
| #3086 | Shaders: share verified compute blobs across preparation worker counts | `e429936c` | `shaders/warm_blobs.rs` +125/−29: blobs verificados passam a ser compartilhados entre contagens de worker. Este é o **head** do repositório. |
| #2771 | Tests: some_ranges in texture_upload wraps its generator instead of overflowing | `e48e74ca` | `texture_upload.rs` +1/−1: só o gerador do teste. Confere. |
| #3006 | D3D12.X: XCL_UMA_DRIVER=mixed falls back without GPU_UPLOAD heaps | `3ec566f5` | `real.rs` +47 e `uma_driver.rs` +108: fallback sem heaps `GPU_UPLOAD`, buffers de console num segmento. Confere. |
| #3007 | D3D12.X: XCL_UMA_UPLOAD_PRIORITY=1 keeps a mixed window's GPU_UPLOAD segment resident | `d526cbff` | `uma_driver.rs` +20: mantém o segmento residente. Confere. |
| #3008 | D3D12.X: XCL_SMALL_TEXTURES=1 puts small sampled textures on 4 KiB placements | `01b94f5f` | 254 linhas em `device_x.rs`, `late_release.rs`, `real.rs`: placements de 4 KiB. Confere. |
| #3090 | D3D12.X: keep barriers carrying console hint flags | `54225f58` | `list_x.rs` +177/−1: barreiras que carregam flags de dica do console são preservadas. Confere. |

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum hash do catálogo na janela (250 commits da página + 19 merges conferidos
  por prefixo contra os 262 hashes das 305 linhas).
- **Por área, com prova no diff:**
  - **D-11** (entradas de cache do per-lane não incluíam as fontes do lowering, DXIL obsoleto
    servido do disco): `5fb57918` e `fb5e7d15` mexem exatamente na **identidade e invalidação** do
    cache de blob de driver — `5fb57918` dá identidade canônica sem ponteiro e invalida por geração
    exata; `fb5e7d15` garante que a rejeição apague o handle certo em filesystem sem POSIX. Não são a
    linha de D-11 (que é o cache do per-lane no DXIL), mas é a mesma classe de falha e é o achado
    mais próximo do catálogo nesta janela.
  - **D-20/D-21** (`ExecuteIndirect` lendo argumentos antes da barreira / rodando com pipeline não
    traduzido): `47b8e3fd` publica transições e donos de escrita GPU **depois** de o gate nativo
    aceitar o pedaço, o que é a mecânica de ordenação que D-20 descreve. Não há prova de que fecha
    D-20.
  - **D-02** (codegen derrubou o compilador AMD): não tocado nesta janela.
- **Abre defeito novo?** Um candidato com evidência no diff e um na descrição:
  - `15aa7f36` faz o otimizador **ler a perna preservada** de `select(exec, novo, antigo)` quando o
    valor só é observado sob a mesma condição. A correção depende de `lane_local` excluir tudo que
    observa outras lanes, memória ou divisão indefinida; se a lista estiver incompleta, o fold remove
    uma preservação que importava. O diff lista as exclusões, mas não há prova de completude — é
    candidato a linha de catálogo, não um defeito confirmado.
  - `d9403543` muda `DispatchX` para mascarar grupos parciais por ballot e sobe a revisão da glue
    (`pso-glue-v3`), o que invalida traduções antigas; o corpo diz que "kernels multidimensionais
    usam HLSL para ordenação de wave linear" — não verifiquei os dois emissores além dos trechos.

## Confiança

**Verificado:** os 40 commits foram lidos pelo diff (13:20–13:50Z), com arquivos, contagens e trechos
de patch; um `fetch` do commit `4b01fde4` falhou na primeira passada e foi refeito com sucesso (o
documento descreve o diff). Os 19 merges foram conferidos um a um pelo commit de merge (pais, stats,
arquivos). Os 62 PRs abertos acima de #3075 e os 19 mesclados desde `2026-11-28T01:14:26Z` vieram da
API REST.

**Não verificado:** nenhuma build, nenhum teste e nenhuma execução; os números de linhas, ULP,
pipelines e cobertura são os que os commits declaram. O conteúdo de #1504/#1513 citado no
`victory-fork` não tem relação com este repositório. O corpo de #2959 (benchmark) tem 48 mil linhas
de diff e só a estrutura foi lida.

**Corte e cursor:** a página do `compare` a partir de `3dbac13a` devolve **250 commits de 338**
(`ahead_by=338`, `commits.length=250`); os 40 documentados são os 40 primeiros dessa página e a
interseção com o documento das 12:00 é **zero**. O próximo cursor é `e5266254` (índice 40). Isso
significa que a API não entrega, nesta chamada, 88 dos 338 commits: eles não foram documentados nem
silenciosamente descartados — ficam registrados aqui como lacuna de cobertura do `compare`, e o
`ahead_by` continua a ser a medida de atraso.

**Head moveu durante a execução.** A coleta fechou com `e429936c`; às 13:23–13:24Z o head passou a
`e13d6250`, com 13 commits novos acima dele: `3c586fce` (#1809, "huge compute programs through the
direct DXIL translator by default", trazendo `afec444f`, `56237a2d`, `632c699f`, `098caf55`),
`e582b323` (#3102, "construct benchmark config without private fields", que estava aberto na coleta),
`773046d1` (#1873, "geometry ES's `s_setpc` and `gs_start` bound structured stretches") e `e13d6250`
(#1908, "load an in-range MSAA sample before zeroing past-the-count ones"). **Não** foram
documentados: chegaram depois da janela e o teto de 40 commits já estava consumido pelo backlog. O
`commit_cursor` gravado (`e5266254`) continua sendo o próximo do recorte de 40, e esses 13 serão
vistos pelo `compare` do próximo ciclo.

**Divergência de contagem:** 19 PRs mesclados na janela de merges; nenhum deles tem
`merge_commit_sha` dentro dos 40 commits documentados (são merges de 03:18–04:12Z, posteriores ao
recorte). `last_run` avança para `2026-11-28T04:12:54Z` (#3090), o maior `merged_at` documentado.
