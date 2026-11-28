# ivall/xcl — atualização de 2026-11-28 17:00

## Resumo

Hora de infraestrutura de pipeline e de recompilador. Quarenta commits, quase todos de `bmjask`, em
duas frentes: (a) o tradutor direto passa a **recusar** os casos que não consegue ordenar — `s_barrier`
em grupo de várias ondas de hardware, leitura de lane superior em wave64 gráfico dividido, acesso real
a LDS no split wave64 — deixando-os para o caminho HLSL, e reconstrói reduções de onda inteira sobre as
lanes lançadas; (b) o D3D12.X troca o "endereço do título" por identidade exata de pipeline (pin de
geração) em bind, draw e dispatch, com aposentadoria por recibo de submissão. Três merges na janela
(`#3175`, `#3181`, `#3182`) trazem o manifesto ordenado de primeiro uso de PSO e um passo offline que
pré-compila o prefixo gravado até um present.

## Commits novos (40 de 118 sem documento)

Janela: `compare(3cde0cc3...10d27297)` = 120 commits, `status: ahead`, `behind_by: 0`. Dois hashes já
documentados foram pulados por hash (`2dad32e6`, `abf7e177`, ciclo de 12:00). Documentados os 40
primeiros não documentados na ordem do `compare`: `38c81423` → `4701a874`. Restam **78**.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `38c81423` | bmjask | Tests: bound concurrent direct-DXIL translations | `robustness_tests.rs`: troca 4 threads com pilha externa de 64 MiB por 2 threads comuns; comentário registra que cada tradução já roda em worker próprio de 256 MiB. +4/−1. |
| `5e343d1a` | bmjask | Scripts: show Windows test failures before abort | `.github/workflows/ci.yml`: o passo de build/test no Windows passa `--nocapture` ao `test.ps1`, para o fast-fail nativo não esconder asserção Rust anterior. Uma linha. |
| `2ca91276` | bmjask | Recompiler: leave unsafe group barriers to HLSL | `Lw` ganha `barrier_waves` (threads do grupo ÷ lanes de hardware, só compute/mesh); `s_barrier` com `barrier_waves > 1` devolve erro quando `safe_barrier_wanted()`. `emit.rs` extrai `safe_barrier_wanted()` (env `XCL_SAFE_BARRIER` ≠ `0`); `direct_or` e os testes passam a aceitar o fallback HLSL. +71/−4, 5 arquivos. |
| `5a984e02` | bmjask | Recompiler: keep backedge values behind loop phis | `module/opt.rs`: monta `inst_at` (posição serializada de cada instrução) e só elimina phi trivial quando o substituto já está definido **antes** dele; nova cobertura do laço self/backedge depois da simplificação de CFG. +75/−3. |
| `f9f1762f` | bmjask | Recompiler: isolate safe barrier cache policy | `build.rs`: `TRANSLATION_GLUE_REVISION` `pso-glue-v3` → `v4`. `pso.rs::cache_allowed` inclui `XCL_SAFE_BARRIER` entre os overrides que desligam o cache de shader (o override muda admissão e barreiras geradas). Testes de fronteira de onda física, mesh wave32 nativo e override inseguro. |
| `22026ba5` | bmjask | Recompiler: order split wave64 LDS instructions | `Em` ganha `LdsGap` (Open/Read/Met) e `block_starts: BTreeSet<u64>`; `Em::new` recebe os líderes de bloco e o `translate_inner` passa `leaders.clone()`. Elide encontro redundante entre as metades e **não** elide através de início de bloco. +268/−3, 4 arquivos, 189 linhas de teste. |
| `dbe6b30b` | bmjask | Recompiler: route split wave64 LDS through HLSL | `real_lds_access()` reconhece `ENC_DS` fora de GDS; `refuse_split_real_lds` devolve "split wave64 LDS access needs the HLSL path" no per-lane/direto, preservando ondas nativas, geometria e DS cross-lane. +84/−2, 4 arquivos. |
| `e909e148` | bmjask | Recompiler: reuse reduction meetings for LDS order | Marca `self.x64_met()` depois do `xcl_x_meet()` das caudas de redução fundidas, para o próximo acesso a LDS não pagar encontro redundante; teste `split_wave64_reduction_exchange_orders_the_next_lds_access`. +37. |
| `fb0931da` | bmjask | Recompiler: rebuild partial-wave graphics reductions | Reconhece o padrão exato min/max/and/or de onda inteira só com proveniência da máscara salva e fluxo provados (`ReductionScope::{Half32, Whole64}`); reconstrói sobre as lanes lançadas, conserta o acumulador e manda estágio gráfico reconhecido para o HLSL. +263/−69, 6 arquivos (README e `docs/wave64-on-wave32.md` incluídos). |
| `50e85c5b` | bmjask | Recompiler: reject unsafe direct wave64 graphics reads | Propaga o modo de onda ao DXIL direto: wave64 VS/PS com leitura de lane superior em modo 32 lanes falha fechado; `coverage.rs` deixa de forçar `split: false`; `pso.rs` passa `split_wave64` a `direct_graphics`. +61/−5, 4 arquivos. |
| `4d647491` | bmjask | Recompiler: preserve graphics spill slots in partial waves | `Em::upper_presence()` e sombras escalares de slots `V_WRITELANE` cujo lane alvo não foi lançado; usa o valor nativo quando a lane física existe e inclui helper lanes de pixel pelo caminho de quad. +101/−60, 6 arquivos. |
| `8cb7b0a0` | bmjask | D3D12.X: retain exact pipeline decisions at bind | `bind::State` tira snapshot do pipeline registrado **antes** de tomar o lock e guarda a identidade `Pending` exata ou uma geração pronta fixada; gerações despejadas ficam diferidas (esta etapa não inicia restauração). +165/−24. |
| `a4dbe5c5` | bmjask | Tests: smoke and time wave64 graphics translation | `scripts/test.ps1`: `cargo bench -p xcl --bench '*' -- --test` compila e roda cada closure uma vez; `benches/gpu_cpu.rs` ganha `graphics_program(instructions, spills)`. +72/−1, 3 arquivos. |
| `a6130374` | bmjask | D3D12.X: own exact pipeline choices for emission | Consumidor de identidade retida: expõe ponteiro bruto **só** junto do pin de geração imutável; one-shot legado vira disposição transitória explícita, com teste de reuso de endereço do título. +136. |
| `ef9ec838` | bmjask | D3D12.X: consume exact recipe pipelines for draws | Draw deixa de reler o endereço do título: uma leitura da decisão retida fora do lock, registro da geração imutável disponível antes do uso e o pipeline selecionado atravessa `draw_rects`/`bind_abi` até a emissão. +209/−30. |
| `ed5e9e47` | bmjask | D3D12.X: pin exact one-shot pipelines for draws | Remove o caminho `PipelineChoice::Legacy` do resolvedor; `Build` ganha `one_shot_generation: Mutex<Option<Arc<Generation<...>>>>` cuja criação/remoção é serializada por `done`; falha fechada quando não há geração possuída. +239/−63. |
| `077a0afa` | bmjask | D3D12.X: retire exact pipeline generations safely | Gerações gravadas por draws sobrevivem a execução de bundle, submissão e aposentadoria de GPU; `PreparedSplit::initial_pipeline` só empresta geração já possuída (senão começa sem PSO e liga na emissão) e os donos são soltos fora do lock da lista. +317/−62, 5 arquivos. |
| `056fc43b` | bmjask | D3D12.X: consume exact pipeline generations for dispatches | `Dispatch` direto carrega a identidade retida pelo demand diferido, espera de build, pin de geração, ABI e `SetPipelineState` nativo; novo `PipelineResolution::initial_build_pending()`. +274/−29, 4 arquivos. |
| `4a7f1175` | bmjask | D3D12.X: probe generation retirement fences outside locks | `SubmissionReceipt.fence` vira `marker: Arc<QueueFence>`; guarda um dono de marcador por fence, lê progresso fora do mutex e aposenta todos os lotes abaixo do corte monotônico capturado (antes era varredura quadrática com chamada de driver sob lock global). +255/−50. |
| `6300d7e2` | bmjask | Tests: benchmark retained pipeline resolution paths | Feature `pipeline-resolution-bench` no `Cargo.toml`; fixture `PipelineResolutionBench` com owners COM falsos pré-aquecidos, sem workers, DXC ou driver; compara caminho `Initial` repetido, `Ready` retido e classificação fresca. +170/−1. |
| `519731e9` | bmjask | D3D12.X: promote completed retained pipeline choices | Publica a primeira geração exata resolvida num cell compartilhado por todos os clones da decisão retida (um bind que nasceu `Initial` não repete locks de completion/geração/candidatos em cada emission); só escolhas exatas disponíveis entram no cache. +191/−34. |
| `e7567b1b` | bmjask | D3D12.X: index exact generation pins per recording | Troca a varredura linear dos donos de geração por índice de identidade de alocação (`HashSet`), mantém o vetor ordenado para a vida da submissão e reconstrói o índice em bundle e restauração de `Close` falho. +103/−73. |
| `61b4ad70` | bmjask | Tests: benchmark exact generation recording contention | Fixture `EmissionBookkeepingBench` nas profundidades 1/16/64/256; compara lotes iguais em listas distintas, serial e paralelo, com owners COM falsos e sem device. +212, 3 arquivos. |
| `35df1ae6` | bmjask | Tests: benchmark retained bind-state contention | `BindStateBench` e `BindWorkers`: cada worker tem carreador de resolução próprio para isolar o mutex global do bind (e não uma linha de cache de contagem de referências). +148, 3 arquivos. |
| `6ae88983` | bmjask | D3D12.X: preserve bind state across list address reuse | Remove o segundo `bind::forget(pc)` executado depois de `Release` retornar: o primário adotado já esquece o estado antes do `Release` nativo final, e um novo list no mesmo endereço podia ser apagado. +33/−1. |
| `56d19278` | bmjask | D3D12.X: isolate bind state per command list | `STATES: Mutex<Option<HashMap<usize, State>>>` vira registro de 64 shards com `BindEntry` e lock por lista; guardas do registro saem antes do lock da lista; `forget` extrai o estado para soltar transferência UMA e destruição de pipeline fora das duas classes de lock. +189/−36. |
| `a3ae4644` | bmjask | D3D12.X: cache bind entries per recording thread | Quatro referências fracas por thread para pular os shards; invalidação de mão única e checagem `Acquire` pós-lock rejeitam entrada obsoleta após reuso de endereço, sem estender estado nem tempo de vida COM. +170/−13. |
| `b1e44d89` | bmjask | D3D12.X: prepare translated ABI state under one lock | Captura raiz, três reservas de arena e alocador numa única seção crítica; limites de dispatch passam explicitamente (campo `limits` sai do `State`), para falha não vazar limite para o dispatch seguinte. +122/−51. |
| `6bdd99ef` | bmjask | D3D12.X: measure pipeline queue residence | `StallKind::{PipelineQueueSpeculative, PipelineQueueDemand}`; residência gravada por prioridade, com promoção dividida e timings abandonados em trabalho cancelado, agregados fora do lock da fila. +182/−7, 3 arquivos. |
| `84f7dbf6` | bmjask | D3D12.X: shard in-flight compute build coalescing | Chaves de compute em voo passam a 64 shards (`pending_shard_index`), limpeza só no contato exato da chave e remoção terminal por identidade fraca; aposentadoria de contador serializada antes da remoção. +168/−26. |
| `8ad1245f` | bmjask | D3D12.X: make repeated submit demand lock free | `Build.by_submit: AtomicBool` publica a demanda de submit já concluída (promoção/escala feitas), então dispatch diferido repetido usa um load `Acquire` em vez de retomar locks de fila e demanda. +86/−11. |
| `2dbf6e82` | bmjask | D3D12.X: measure write-watch range costs | Mantém a política de 8 MiB e acrescenta cobertura exata opcional e tempo por chamada de `GetWriteWatch`, com planner determinístico e benchmark manual de limiar para ativação medida depois. +243/−19. |
| `9b5b5bf1` | bmjask | D3D12.X: observe initial build completion lock free | `Build.completed: AtomicBool` publica estado terminal de build inicial; esperas exatas, posse de geração, release e evicção continuam serializadas por `done`. +209/−11. |
| `e4c3df14` | bmjask | D3D12.X: measure texture selection contention | `LOOK_FIELDS` 19→25: espera de lock, tempo de posse, trabalho do predicado, matches e uploads retidos na amostra opcional `XCL_ACTIVITY`, sem mudar seleção nem semântica de retenção. +101/−27. |
| `a3b36430` | bmjask | D3D12.X: measure pipeline queue admission | `StallKind::PipelineQueueAdmission`; admissão e promoção contabilizadas separadamente da residência, sem sobreposição, e o worker aceito é acordado antes de publicar o tempo (relatar não pode atrasar quem pode pegar o job). +54/−10, 3 arquivos. |
| `f194ea7d` | bmjask | D3D12.X: measure submission residency phases | `LOOK_FIELDS` 25→35 e `BeforeExecuteTiming` thread-local: gate de execução, lookup de device, polling de fence, estado de página, espera nativa e aposentadoria deixam de ser dobrados em "tempo ativo". +213/−7. |
| `130c0c45` | bmjask | Tests: benchmark texture selection scaling | Feature `texture-selection-bench`; fixture `TextureSelectionBench` com cenários vazio, pendente, GPU-owned, página suja, alias e contenção de lock compartilhado, sobre o predicado de produção. +286/−13. |
| `cbf758ce` | bmjask | D3D12.X: cache queue device ownership | `BeforeExecuteQueue<D>` guarda o lookup bem-sucedido do device no gate de execução da fila (reuso de endereço seguro); falha ou nulo refaz a consulta, e a aposentadoria solta o COM depois de remover a entrada do registro. +168/−17. |
| `c51cfd39` | bmjask | D3D12.X: restore retained pipelines on submit | Identidade exata do ledger capturada no bind, uma requisição de restauração em primeiro plano instalada no submit e espera do ticket na emissão; restauração obrigatória ignora a espera limitada de build inicial e retry residente de outro bind supera demanda falha antiga. +274/−37. |
| `4701a874` | bmjask | Recompiler: use barriers for one-wave split wave64 groups | `x64_spin_for`: grupo de hardware com uma única onda de console usa barreira de grupo (`GroupMemoryBarrierWithGroupSync`) no lugar dos contadores atômicos de chegada; grupos multi-wave mantêm o protocolo limitado e `XCL_WAVE64_SPIN=1` ainda força contadores (diagnóstico). +118/−15. |

## Pull requests abertos (7)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #3188 | D3D12.X: disable lazy level pages when a sampled texture aliases a live GPU page | Porta de `amd-dev` `dece6dd9` para `nvidia-dev`: sob `XCL_TEXTURE_LEVEL_PAGES=1`, textura amostrada colocada no mesmo VA 64 KiB de uma página viva GPU-owned (RTV, depth, UAV) nunca mapeia tiles e o shader lê zero. Diff: remove o `lazy` quando `Pages::pages` já tem página viva. | Área de página/alias; não casa hash do catálogo. |
| #3187 | Tests: the mixed-window remake tests run in a child process | Os dois testes de remake de janela derrubam o binário de teste no NVIDIA (0x7C10003); passam a rodar em processo filho, com `MIXED_REMAKE` mutex estático. | Não (infra de teste). |
| #3185 | D3D12.X: sweep completed PSO generation pins when nothing new is pinned | `retire_native_generations` só varria lotes concluídos quando outro lote pendente chegava; aposentar recibo já completo retornava cedo. Introduzido em `11613fb6` (#2959). | Sinal de regressão recente na área de geração exata; não é D-xx. |
| #3184 | D3D12.X: preserve bit patterns for non-finite channels in integer RTV clears | `ClearRenderTargetView` em alvo inteiro com canal não finito: D3D12 converte NaN em 0 e ±inf no limite do formato; o título (`ea3b5e4d`) espera os bits crus. +150 linhas em `real.rs` (`integer_clear_bits`). | Não está no catálogo; candidato a linha nova. |
| #3183 | Tests: pipeline library tests no longer share a build with two shader tests | Desde #3100 quatro testes falham juntos na CI (contagem 3 onde se espera 2, depois `PoisonError`); dois testes de `shaders.rs` passam a tomar `BUILD_TEST` e os locks passam a `unwrap_or_else(into_inner)`. | Não (infra de teste); explica falha intermitente pós-#3100. |
| #3179 | D3D12.X: remap superseded targets bound through descriptor tables | Alvo superado por colocação sobreposta (`5d00b390`, `XCL_PLACE_RETIRE=1`) tinha tiles desmapeados e só era remapeado em `ResourceBarrier`/`Copy*`; UAV vivo bindado por tabela perdia escritas (leitura volta 0). | Área de tile/alias. |
| #3178 | D3D12.X: the frame trace counts memory copies of views that went stale | Só mede: censo de cópias obsoletas vistas nas tabelas de descritor em draws, sob `XCL_TRACE_FRAME=1`; pede evidência de execução real. | Não; instrumentação para decidir conserto de T#. |

## Pull requests mesclados (3)

Contagem confere com os commits: os três merges têm um commit de conteúdo cada, todos dentro dos 120
da janela.

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #3175 | D3D12.X: record ordered pipeline first use | `db3921dc` (16:12:27Z) | `pso_manifest.rs` novo (201 linhas): sob `XCL_PSO_MANIFEST=1` grava `use`/`build`/`present` em ordem, com chave do shader-set, compute/gráfico, se já estava construído e `driver_ms`/`waited_ms`; pontos em `bind`/`pso_async`/`real.rs`/`present.rs`. Só registro: nada é construído, reordenado ou pulado. Conteúdo: `59cd71e6`. |
| #3181 | D3D12.X: record first use of synchronous compute PSOs | `59e4be57` (16:59:41Z) | PSO de compute criado sincronamente não tinha entrada `Pending` e sumia do manifesto: registro `note_ready_compute` no `SetPipelineState` quando não há resolução, `track_used` no `Pending`, e geração segura no `pso_release` porque o driver pode reusar o endereço depois do `Release`. Conteúdo: `61fdf424`. |
| #3182 | Shaders: precompile a bounded recorded startup prefix | `10d27297` (16:59:49Z) | `shaders/milestone.rs` (166 linhas): com manifesto v1 gravado, `--through-present N` resolve **antes** do trabalho de GPU o prefixo de compute usado até aquele present, em ordem de primeiro uso; recusa prefixo acima de 500 pipelines e para se o corte ou a chave não casarem. Conteúdo: `971f857b`. |

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum por hash: os 120 hashes da janela e os 3 de merge (`db3921dc`, `59e4be57`,
  `10d27297`) não têm interseção com os 270 tokens hexadecimais de `catalogo/defeitos.csv`.
- **Toca área de D-xx?** Sim, sem fechar linha:
  - `fb0931da`, `50e85c5b`, `4d647491` mexem em máscara/lane wave64 e leitura de lane superior — a mesma
    região de **D-16** ("máscara wave64 obsoleta reportada só na metade baixa") e de **D-14**
    (`v_cmpx` float descartando o EXEC anterior). O diff **não** restaura nem altera o comportamento
    descrito em D-16; muda o que o tradutor direto aceita.
  - `f9f1762f` altera a política de cache do shader (`TRANSLATION_GLUE_REVISION` e `cache_allowed`) —
    área de **D-11** ("entradas de cache do per-lane não incluíam as fontes do lowering: DXIL obsoleto
    servido do disco"). Aqui o ajuste é outro: `XCL_SAFE_BARRIER` passa a invalidar o cache.
- **Abre defeito novo?** Não verificado por execução. Dois candidatos a linha nova no catálogo, ambos
  de risco declarado no próprio código: (1) `2ca91276`/`dbe6b30b` recusam programas no DXIL direto e
  os mandam ao HLSL — se o caminho HLSL não cobrir o mesmo conjunto, o sintoma vira falha de tradução
  em vez de corrupção, mas isso **não foi testado** aqui; (2) `c51cfd39` instala restauração
  obrigatória no submit e "ignora a espera limitada de build inicial", o que pode transformar um
  build lento em espera de submissão (`hash`, arquivo `pso_async.rs`, linha não verificada).

## Confiança

**Verificado por diff.** Os 40 commits foram lidos pelo patch de `repos/ivall/xcl/commits/<sha>`
(arquivos, `+/-` e hunks), não pela mensagem. As 3 merges foram lidas pelo `pulls/<n>/files` e pelo
commit de conteúdo; os 7 abertos, pelo corpo e pelos arquivos do PR (#3178–#3188; `#3182`/`#3181`
compartilham arquivos por serem empilhados). Contagem de commits: `compare` devolve `ahead_by: 120`,
`behind_by: 0`; 2 hashes pulados por já estarem documentados; 118 − 40 = **78 pendentes**.

**Não verificado.** Nada foi compilado nem executado: sem `cargo test`, sem GPU, sem rodar o título.
Não li os 78 commits restantes. Não conferi se o caminho HLSL cobre os programas agora recusados pelo
diretor (a afirmação vem do comentário do commit, não de teste). `#3185` cita `11613fb6` (#2959) como
introdutor e `#3183` culpa o estado compartilhado pós-#3100: ambos são afirmações do PR, não
reproduzidas aqui.

**Ordem do `compare`.** Instável, como nas horas anteriores: `2de72223` (conteúdo do `#3100`,
documentado às 16:00) aparece na posição 35 desta lista e não entrou nos 40. O novo `commit_cursor` é
`4701a874`; o próximo ciclo deve pular hash já documentado, senão `2dad32e6`/`abf7e177` voltam.

**Relógio.** O head é `10d27297`, commit de 16:59:48Z no relógio do GitHub; `last_run` gravado em
`2026-11-28T16:59:49Z` (último `merged_at` coberto).
