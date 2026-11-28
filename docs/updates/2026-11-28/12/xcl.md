# ivall/xcl — atualização de 2026-11-28 12:00

## Resumo

376 commits desde `2c717a04` (maior atraso da rotina até agora) e 48 merges na janela, 29 deles em
`nvidia-dev`. Os 40 documentados são uma série de *pipeline library* nativa: receita de PSO gravada em
`used-graphics-pipelines.bin`, corpus curado em `.xcl/pso-corpus`, biblioteca de pipeline do D3D12
serializada e carregada no arranque, tudo com preparação em processos trabalhadores. Junto vêm o
núcleo de atribuição de *stall* (`stall.rs`, `build_queue.rs`, intervalos exatos de escrita) e dois
consertos pequenos e reais: o `ballot_px_quad` do prelúdio de pixel sem laço para o DXC desenrolar e o
teto do *coherent window* em RAM/8 abaixo de 24 GB. Nada foi executado.

## Commits novos (40)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `2dad32e6` | Royalflamejlh | Recompiler: the pixel prelude's quad ballot has no loop for DXC to unroll | `PRELUDE_PIXEL` deixa de ter `[unroll] for (p = 0..3)` em `ballot_px_quad` e passa a quatro `WaveActiveBallot` explícitos (`w0..w3`), combinados por `(w & 0x11111111) << p`. Teste novo garante que não sobra `for`/`while`/`unroll` no trecho. |
| `88bc67da` | theaustinayers | xcl run: a pack's compute pipelines are prepared by worker processes | `present/preparation.rs` roda `prepare_for_launch` num `thread::scope` e imprime uma linha a cada 5 s; `shaders.rs` ganha `prepare_workers()` e o laço de preparação passa a distribuir o trabalho. |
| `5584a825` | theaustinayers | xcl run: the preparation screen counts each pass by itself, and every record | `LaunchPreparation` ganha `workers: AtomicUsize`; quando há trabalhadores, `done`/`total` contam os registros que eles passaram, e a linha de progresso mostra os passes por conta própria. |
| `ad1ff9a6` | theaustinayers | xcl run: the preparation takes the shaders earlier runs used, or none | A preparação parte de `earlier_used_shaders()` (o conjunto que execuções anteriores usaram) e só prepara tudo com `XCL_PREPARE_ALL=1`; `pso_async::shaders_of(key, shaders)` registra os shaders de cada conjunto quando a chave é montada. |
| `f985c1cf` | theaustinayers | xcl run: one preparation worker builds a pipeline, the others leave it | `Pipelines` ganha `elsewhere`; um trabalhador que encontra a pipeline já reivindicada por outro processo do mesmo preparo a deixa para ele e conta em vez de compilar de novo. |
| `53c1d281` | theaustinayers | D3D12.X: record the graphics pipelines a run uses with their state | Arquivo novo `hle/d3d12x/gfx_records.rs` (674 linhas): grava em `used-graphics-pipelines.bin` cada pipeline gráfica usada com a descrição de estado que o título deu (frames com comprimento e *check* de fnv64), para o preparo do próximo arranque reconstruí-la. |
| `192374ea` | theaustinayers | xcl run: the memory line and a device removal give the system's commit | `log_device_removed` passa a imprimir `host::system_commit()` (compromisso e limite do sistema, RAM + page file); `host.rs` ganha `system_commit()`. Motivo declarado no comentário: uma remoção sem falha da GPU tinha 30 GB de compromisso livre. |
| `8f906486` | theaustinayers | xcl run: build the graphics pipelines earlier runs used before the title | `pso::mem_windows()` passa a devolver as janelas do título (`TITLE_WINDOWS`, com `HEAP_WINDOWS` quando aplicável) e `uma::Arena` ganha um modo "processo sem memória do título" que não reserva nem compromete a janela coerente — para o trabalhador de preparo. |
| `0a79e6cc` | CryingLaughing | Shaders: replay compatible two-pass mesh pipelines at startup | O formato de `gfx_records` ganha o tipo de entrada 3 (`key + shaders[2] + layout + state`, stream de mesh de dois passes); `shaders_key` passa a delegar para `shader_set_key`, que aceita a lista de *blobs* em ordem mesh-PS. |
| `4d78a9ba` | bmjask | Shaders: discover and stage pipeline recipes from offline archives | Quatro scripts novos em `scripts/pso_corpus/`: `pso_archive.py` (181 linhas) expande tabelas de seletor de um arquivo em journal `XCLPJ001`; `pso_directory.py` lê dicionários de shader/estado de passo fixo; `stage.py` (267 linhas) monta `.xcl/pso-corpus` a partir de journals; e 282 linhas de teste com fixtures sintéticas. `archive-profile.md` documenta o perfil. |
| `7b392149` | bmjask | Shaders: prepare known native pipelines before releasing the game window | Entrada tipo 4 em `gfx_records` (pipeline com *depth bounds*); a tela de preparação só libera a janela quando o corpus está completo (`check_corpus_complete()`), em vez de só quando não há cancelamento. |
| `0600e862` | bmjask | Shaders: stream staged graphics jobs and preserve mesh cache identities | `used-graphics-pipelines.bin` passa da cache de DXIL compartilhada para a cache do título; os jobs gráficos *staged* passam a ser consumidos em fluxo e as identidades de cache de mesh são preservadas. |
| `f3830798` | bmjask | Shaders: prepare exact pack and history overlaps once at startup | `read_complete` vira `visit_payloads`; `--prepare-corpus` deixa de exigir `--windows` (a configuração sai do registro) e o preparo resolve as sobreposições de pacote e de histórico uma vez só. |
| `02c66cad` | bmjask | Shaders: keep the launch owner responsible for the whole input | `precompile_slice(startup_owner, value)`: o processo que é dono do arranque **não** aceita a fatia `i/n` do input, que fica só para os trabalhadores; sem isso o dono compilava uma fatia e o resto ficava órfão. |
| `7659b88d` | bmjask | Shaders: recover usable graphics history after an interrupted append | `read_history_payloads` novo: o arquivo de histórico pode estar aberto para *append*, então a leitura aceita a entrada final parcial e devolve o que é inteiro. Antes um append interrompido zerava o histórico. |
| `2d473dcd` | bmjask | Shaders: retain verified native preparations across rebuilds | `build.rs` ganha uma constante de revisão do construtor nativo (o mapa de receita para descritores/blobs), separada das revisões de tradutor, driver, adaptador e configuração; `dxc::cache_key` fica `pub(crate)` para ser usado na chave. Preparos verificados sobrevivem a reconstruções. |
| `0d977360` | bmjask | Shaders: share verification between launch and offline workers | `warm_settings_key` é extraído e usado pelos dois lados (arranque e trabalho offline); o corpus passa a imprimir `Reused N verified native graphics recipes`. |
| `b64e4862` | bmjask | Shaders: preserve inactive-stage cache identities on main | `two_pass_stream_key` deixa de colapsar todo payload zerado em `h.u64(0)` para os tipos 1..6/24/25 e passa a distinguir o tipo do estágio inativo — antes dois streams com estágios inativos diferentes geravam a mesma chave. |
| `62c6bda8` | bmjask | Shaders: prove native pipeline library round trips | Módulo novo `shaders/pipeline_library.rs` (286 linhas): dono e serialização da biblioteca de pipeline do D3D12. O ponto que o comentário destaca: `CreatePipelineLibrary` **retém o ponteiro**, então a biblioteca precisa possuir os bytes serializados enquanto o objeto COM viver. |
| `abf7e177` | RodrigoTorreao | UMA: cap the coherent window at RAM/8 below 24 GB of RAM | `gpu/uma.rs`: `AUTO_CAP_BELOW_RAM = 24 GiB`; abaixo disso a janela coerente é limitada a RAM/8. Antes a janela podia ser maior que a memória que o host conseguia comprometer. |
| `b83c3f0d` | bmjask | Shaders: pass null for empty pipeline libraries | `NativeLibrary::open` chama `CreatePipelineLibrary` com ponteiro nulo quando os bytes serializados estão vazios, em vez de passar um buffer de comprimento zero. |
| `0d84fb5e` | bmjask | Shaders: publish semantic corpus identities | `preparation_source_id` vira `preparation_layout_id` (identidade do layout, não do código) e `Corpus` ganha `identity: Option<[u8; 32]>`, publicada junto do corpus. |
| `5ba1c9bd` | bmjask | Shaders: collect prepared PSOs into native libraries | `real.rs` passa a entregar ao coletor cada `ID3D12PipelineState` criado com sucesso, para ser serializado na biblioteca nativa; `pipeline_library` fica `pub(crate)`. |
| `e2d5cd87` | bmjask | Shaders: trust complete native pipeline libraries | Módulo `warm_library` novo e `write_atomic` exposto: uma biblioteca marcada completa é aceita como está, sem reexecutar a verificação receita por receita. |
| `99948a6a` | bmjask | Shaders: load prepared pipelines at runtime | `init_with_remap` chama `pipeline_library::activate(&device)` depois de abrir a UMA; `Active` guarda as bibliotecas de gráficos e compute e as consulta antes de criar PSO. |
| `a19c48b6` | bmjask | Tests: measure distinct pipeline library scaling | Teste `distinct_compute_pipelines_scale_and_remain_individually_loadable`: cria N pipelines de compute distintos, serializa, reabre e confere que cada um continua carregável. Só teste. |
| `e475c76f` | bmjask | Shaders: skip recipe walks for warm libraries | `prepare_for_launch` consulta a biblioteca quente antes de percorrer receitas; `Corpus` guarda `directory` e `expected: [usize; 2]` para decidir sem abrir o diretório outra vez. |
| `ad05bf97` | bmjask | D3D12.X: add an opt-in stall attribution core | `mod stall` novo e `XCL_STALL_TRACE` liga o gravador no início de `host_main`; o módulo é declarado em `lib.rs`. Sem efeito quando a variável não existe. |
| `ce0eb60b` | bmjask | D3D12.X: track exact command-list write intervals | `list_x.rs` passa de `HashMap` para `BTreeMap` e guarda o intervalo de escrita exato de cada trecho de lista de comando, para atribuir *stall* sem varrer tudo. |
| `470e9ca8` | bmjask | D3D12.X: add a priority build queue | `hle/d3d12x/build_queue.rs` (379 linhas): fila de prioridade com identidade estável de job, promoção monotônica e conclusão que sobrevive a panic ou desligamento. O cabeçalho diz explicitamente que **não** está ligada ao runtime. |
| `1dd68d36` | bmjask | Recompiler: attribute translation and DXC stalls | `dxc::Job` ganha `queue_stall: Option<Span>` medindo da submissão até o trabalhador admitir o job sob o limite de memória; `recomp::translate` fica `#[inline]` para a atribuição não inventar um quadro. |
| `3f2c4ee9` | bmjask | D3D12.X: snapshot command-list effects before submit | `list_x.rs` tira um instantâneo dos efeitos da lista antes de submeter (`clear_waits` deixa de ser o único lugar que mexe em `pending_waits`), e `real.rs` executa a partir do instantâneo em vez do estado vivo. |
| `3904c8d4` | bmjask | D3D12.X: separate pipeline builds from stand-ins | `pso_async::Pending` deixa de guardar `Mutex<Option<Job>>`: o job e o *stand-in* passam a ser estruturas distintas, para um bind que não pode esperar a fila não consumir o build. |
| `3dbac13a` | bmjask | D3D12.X: serialize stall attribution in reports | `report.rs` ganha `STALL_SUMMARY_VERSION = 1` e a serialização do resumo; `stall::snapshot()` muda de assinatura para acompanhar. |
| `ae5ae5a8` | Prime1997 | Merge pull request #3045 from bmjask/shaders/pipeline-library-probe | Merge de dois pais de `#3045`. O conteúdo é o de `2d473dcd` (revisão do construtor + `cache_key` exposto). |
| `96247f07` | Prime1997 | Merge pull request #3046 from bmjask/shaders/pipeline-library-identities | Merge de `#3046`; conteúdo de `0d84fb5e`. |
| `c62393c9` | Prime1997 | Merge pull request #3053 from bmjask/shaders/pipeline-library-receipt | Merge de `#3053`; conteúdo de `5ba1c9bd`. |
| `78421697` | Prime1997 | Merge pull request #3054 from bmjask/shaders/pipeline-library-runtime | Merge de `#3054`; conteúdo de `99948a6a`. |
| `2049651b` | Prime1997 | Merge pull request #3055 from bmjask/shaders/pipeline-library-scale-tests | Merge de `#3055`; conteúdo de `a19c48b6`. |
| `753e8b1a` | Prime1997 | Merge pull request #3057 from bmjask/shaders/pipeline-library-fast-discovery | Merge de `#3057`; conteúdo de `e475c76f`. |

## Pull requests abertos (36)

Dos 36 acima do cursor #3075, a maioria (26) aponta para `nvidia-dev` ou `amd-dev`, não para a default.
Lidos por título e corpo; **nenhum patch de PR aberto foi lido**.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #3125 | D3D12.X: retain resources when retirement fences fail | Não libera nem reusa recurso quando a conclusão da fila não pode ser estabelecida: o D3D12 relata remoção como `UINT64_MAX`, e tratá-la como fence concluída liberava recurso cedo. | Área de **D-20** (barreira/ordem), não o defeito. |
| #3124 | D3D12.X: fail lazy texture placement when mappings are unfinished | Segue `#3116`: recusa colocação preguiçosa se os mapeamentos não terminaram. | Não. |
| #3123 | Memory: defer private physical-home GPU tiles on NVIDIA | Adaptação NVIDIA de `#1916`. | Não. |
| #3122 | D3D12.X: retire idle NVIDIA command pools between presents | Devolve listas/realocadores ociosos quando o título para de chamar `PresentX`. | Não. |
| #3120 | Shaders: a Series S cache starts from the manifests both profiles shared | Segue `#2860`: cache `lockhart` parte dos manifestos que os dois perfis compartilhavam. | Não. |
| #3119 | D3D12.X: a removal with no fault recorded is not logged as a fault at VA 0 | Só registro: não imprime DRED de VA 0 quando o DRED não gravou falha (estado 0 = remoção sem trabalho da GPU). | Não. |
| #3118 | README: document XCL_NETWORK=0 | Só README. | Não. |
| #3116 | D3D12.X: a new texture with lazy levels has its tiles mapped by a call | Com `XCL_TEXTURE_LEVEL_PAGES=1` a entrada no mundo perde o dispositivo (`0x887A0006`); mapeia as tiles por chamada. | **Candidato a defeito novo** (device hung). |
| #3113 | README: document XCL_PREPARE_WORKERS and XCL_PIPELINE_THREADS | Só README. | Não. |
| #3112 | D3D12.X: add opt-in XCL_SPARSE_VRAM_CAP_MB | Teto de VRAM para pools esparsos, para GPUs de 8 GB em `amd-dev`; desligado por padrão. | Não. |
| #3111 | D3D12.X: drop CopyScratch resources outside COPY_SCRATCH lock | Fecha `#3069`: `Release` dentro do mutex global `COPY_SCRATCH` (reentrância). | Não. |
| #3110 | D3D12.X: trim idle title_list spares and put-aside command allocators on PresentX | Em `amd-dev` (RX 6650 XT, 8 GB) os spares cresciam até 1.815–4.096 listas. | Não. |
| #3107 | D3D12.X: recycle driver timing slots after thread exit | 64 slots por processo faziam threads posteriores perderem a medição. | Não. |
| #3106 | README: document XCL_UMA_WINDOWS and XCL_HEAP_WINDOWS | Só README. | Não. |
| #3105 | D3D12.X: stage upload arguments for indirect mesh draws | Argumentos/contagens em heap UPLOAD para dois passes indiretos de mesh em `nvidia-dev`. | Não. |
| #3104 | D3D12.X: publish CPU writes only after GPU completion | Remove a queda de 10 s que publicava *readback* antes da cópia rodar. | Não. |
| #3103 | Recompiler: count split shader pieces in the NVIDIA cache budget | Porte do conserto de admissão de cache: inserção omitia peças do dispatch-chain e a evicção as subtraía. | Não. |
| #3102 | Tests: construct benchmark config without private fields | Só compilação de teste (`E0451`). | Não. |
| #3101 | Shaders: verify native two-pass recipe receipts | Segue `#2829`: o recibo exigia nomes `tp-ms-`/`tp-cs-` que o cache nativo não escreve. | Não. |
| #3100 | Shaders: import exact PSOs from prior libraries | Impede que uma mudança de recibo semântico descarte PSOs exatos ainda compatíveis. | Não. |
| #3099 | Wine: raw internal fences + native-vtable guards + XG CopyFromSubresource 7-arg ABI | Compatibilidade com vkd3d-proton que identifica objetos por tabela de vtable. | Não. |
| #3098 | D3D12.X: evict top texture mips under local VRAM budget | `XCL_TEXTURE_BUDGET_EVICT=1`, opt-in. | Não. |
| #3097 | D3D12.X: XCL_VIEW_MEM=1 maps the title's GPU-view memory into heap windows 4..15 | Empilhado em `#3096`/`#1055`. | Não. |
| #3096 | Recompiler: heap-window shaders reach all 16 window slots of the heap | O heap reserva 16 slots e os dois tradutores só usavam parte. | Não. |
| #3095 | README: document the automatic coherent-window cap | Documenta `#3068`, que é o `abf7e177` desta janela. | Não. |
| #3094 | D3D12.X: a sampled texture in the memory of a written one of another format is copied from it (XCL_ALIAS_COPY=1) | Céu, *light probes* e *bounce* leem memória de textura escrita em outro formato. | Não. |
| #3092 | Memory: use a 2 MiB grain for allocator page size two | Corrige arredondamento de 4 MiB para 2 MiB em `XMemAllocDefault` PageSize=2. | Não. |
| #3091 | Shaders: share owned bytecode across derived and pending pipelines | Cópias imutáveis compartilhadas dos bytes de shader. | Não. |
| #3089 | D3D12.X: the depth plane of an R24G8 buffer has a twin | `XCL_DEPTH_UAV_TWIN=1`: gêmeo UAV do plano de profundidade para o lighting. | Não. |
| #3088 | README: document xcl shaders --prepare-corpus | Só README. | Não. |
| #3087 | ran cargo fix --lib -p xcl | `fetch_update` → `try_update` em `hle/xgameruntime/mod.rs`. | Não. |
| #3083 | Launcher: resolve relative XCL_DATA_ROOT before starting the host | Caminho relativo gravava `boot.json` no diretório errado. | Não. |
| #3082 | PE loader: a title's CLZERO loops run as native code on a CPU without CLZERO | Porte do `#2892` para `main`: laço `clzero; add rax,0x40; cmp; jb` em CPU Intel. | Não. |
| #3081 | DirectStorage: gate BC7 rotation by mode | Os bits 3–4 do BC7 só são rotação nos modos 4 e 5; o porte BCPack lia em todo modo. | Não. |
| #3079 | D3D12.X: copy out-of-window two-pass indirect arguments into the window | Argumentos de `DispatchMesh` fora das janelas eram descartados. | Não. |
| #3076 | D3D12.X: command list #142 applies the quad sample mask | `SetQuadSampleMaskX(u64)` é *stub* na `main`: todo pixel é desenhado. | Não. |

## Pull requests mesclados (8)

Dos 48 merges da janela, 29 foram para `nvidia-dev` (não entram no cursor de commits da default) e 11
para `main` com o commit além do teto de 40. Documentados:

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #3045 | Shaders: prove native pipeline library round trips | `ae5ae5a8` | Revisão do construtor nativo e chave de cache compartilhada; o *round trip* propriamente dito vem em `62c6bda8`. |
| #3046 | Shaders: publish semantic corpus identities | `96247f07` | `preparation_layout_id` e `Corpus::identity`. |
| #3053 | Shaders: trust complete native pipeline libraries | `c62393c9` | Entrega dos PSOs criados ao coletor. |
| #3054 | Shaders: load prepared pipelines at runtime | `78421697` | `activate(&device)` depois da UMA. |
| #3055 | Tests: measure distinct pipeline library scaling | `2049651b` | Teste de escala e carregamento individual. |
| #3057 | Shaders: skip recipe walks for warm libraries | `753e8b1a` | Atalho para biblioteca quente. |
| #2829 | Shaders: discover offline pipeline recipes and prepare them before launch | `b64e4862` | Entrou **só a preservação de identidades de estágio inativo** (`two_pass_stream_key`); a descoberta offline que o título promete está em `4d78a9ba`, sem merge correspondente nesta janela. |
| #3051 | Shaders: collect prepared PSOs into native libraries | `5ba1c9bd` | Coleta dos PSOs; nada além. |

Os seis primeiros são *merges* de dois pais cujo conteúdo duplica os commits de branch
(`2d473dcd`, `0d84fb5e`, `5ba1c9bd`, `99948a6a`, `a19c48b6`, `e475c76f`). As 40 entradas da tabela de
commits correspondem a 34 mudanças distintas.

## Cruzamento com o catálogo

- **Nenhum dos 376 hashes da janela está no catálogo.**
- **Área de D-02** ("regressão: o codegen derrubou o compilador da shader da AMD", `a787c52`/`70f8dc4`):
  `2dad32e6` tira o laço do `ballot_px_quad` exatamente porque o DXC o desenrolava e o compilador da AMD
  quebrava (`#134`). O comentário do arquivo cita o caso; o defeito do catálogo é anterior e já
  corrigido por outro caminho. **Relação de classe, não de hash.**
- **Área de D-11** ("entradas de cache do per-lane não incluíam as fontes do lowering: DXIL obsoleto
  servido do disco"): a série de *pipeline library* é a mesma família — chave de receita, revisão do
  construtor (`2d473dcd`), identidade do corpus (`0d84fb5e`) e "confiar em biblioteca completa"
  (`e2d5cd87`). `e2d5cd87` afrouxa a verificação exatamente no ponto que D-11 descreve; **não há
  evidência de regressão no diff** (a biblioteca só é aceita com o marcador de completa e a identidade
  publicada), mas é a área e merece a próxima coleta.
- **Candidato a defeito novo, com evidência no corpo do PR:** `#3116` — com `XCL_TEXTURE_LEVEL_PAGES=1`
  a entrada no mundo perde o dispositivo (`0x887A0006`, device hung) em boa parte dos arranques. `#3124`
  é a alternativa ao conserto. Não está provado aqui: o patch do PR não foi lido.
- `abf7e177` (teto de RAM/8 na janela coerente) é a resposta a um limite de memória, não a um defeito do
  catálogo.
- Nenhum defeito do catálogo é fechado por esta janela.

## Confiança

- **Verificado por API:** 40 commits (sha, data, autor), patches dos arquivos de código lidos,
  48 merges da janela por `pulls?state=closed` filtrado em `merged_at > 2026-11-27T21:27:54Z`, 36 PRs
  abertos (título e corpo). O `merge_commit_sha` dos 8 merges documentados casa com o sha da tabela.
- **NÃO VERIFICADO:** nada foi compilado nem executado; nenhum teste do repositório foi rodado. As
  afirmações de que a biblioteca quente "é aceita sem verificação" e de que a preparação em
  trabalhadores "conta cada passe" vêm do diff e dos comentários, não de execução. O *device hung* do
  `#3116` é declaração do autor.
- **Teto de 40 atingido:** 40 de 376 — este repositório entra em violação estrutural do teto: são
  **336 commits** restantes. `last_commit` fica em `2c717a04`, `commit_cursor = 3dbac13a` (40º
  documentado) e `head_at_run = e429936c`. `compare` a partir de `3dbac13a` devolve **338** commits,
  contra 336 restantes: 38 dos 40 documentados não são ancestrais do 40º (que não é merge, mas o
  histórico tem merges de dois pais na faixa). Com 376 commits de atraso e teto de 40 por hora, este
  repositório levaria ~9 horas de deep dive exclusivo para zerar o atraso. Fica registrado como
  problema estrutural, não como falha desta execução.
- **`last_run`** gravado em `2026-11-28T01:14:26Z` (o `merged_at` de `#3057`, o mais recente entre os
  8 merges documentados). Os 40 merges restantes, até 04:12:54Z, ficam para o próximo ciclo.
