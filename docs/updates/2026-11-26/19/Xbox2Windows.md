# R0Y00/Xbox2Windows — atualização de 2026-11-26 19:00

## Resumo

A faixa desde `562f7ddb` tem **112 commits**; os **40 mais antigos** foram documentados (teto de 40 por
repositório), o restante fica no `commit_cursor`. O bloco é quase todo tradutor `gfx2hlsl` e periféricos do
runtime de startup: SAD `u8` alto/mascarado, `s_bfm_b64`, `s_quadmask_b64`, `v_bfm_b32`, `v_mad_*64*`, half
empacotado, SDWA binary16, DPP quad read, escalares `execz`, LDS compartilhado por grupo no `gfx1013`, e um
lote de endurecimento de diagnóstico (CLI do `hardware_reports`, refusals nomeados, censo de slot 70).

## Commits novos (40 documentados de 112)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| 2215c26d | Ammar Abdelfattah | tooling: harden hardware_reports CLI guards and add CLI suite (#2682) | `tools/hardware_reports.py` passa a recusar `--after` negativo, caminho de comentários que é diretório ou não existe, e `--readme` inexistente/diretório; `load_comments` aceita `id` inteiro **não negativo** (rejeita `bool` e `<0`), o que faz `id=0` deixar de ser erro. Novo `tests/hardware_reports_cli_check.py` (288 linhas) exercita Markdown/JSON, `--after 0`, ordenação por id e o caminho de subprocesso com `encoding="utf-8"`. |
| feec219e | Royoo | Bind a proven live index alias for direct draws without sliced streams (#2724) | Em `src/startup_stubs/d3d12_draw.inl`, um draw indexado direto sem SRV de stream de vértice passa a montar `D3D12_INDEX_BUFFER_VIEW` sobre o alias colocado vivo quando início, alinhamento e tamanho cabem em `pass.ib.SizeInBytes`; `require_state(INDEX_BUFFER)` antes do draw. Sem alias, mantém o `frame_copy` do snapshot de gravação. Novo `tests/startup_direct_index_late_write_check.py` compara 32 pixels depois da fence, inclusive reescrita pré-submit e view `R32_UINT` com elemento-isca. |
| 1c3abf59 | Asım Mirza Çiftler | runtime: validate ApuAlloc outputs and synchronize AcpHal heap range queries (#1902) | `ApuCreateHeap` recusa heap cujo tamanho arredondado a 64 KiB excede 4 GiB com `E_INVALIDARG` **antes** de `VirtualAlloc`; `heap_snapshot` lê base/tamanho sob `heap_lock` e `in_apu_range` testa `[0x10000, min(bytes,4 GiB))`. Saída nula ou não gravável vira `E_POINTER` sem alocar; tabela do `AcpHalCreate` passa a função estática única. Teste cobre saídas `PAGE_READONLY`, heap de 4 GiB + 1 byte e `AcpHalCreate` concorrente. |
| 1226f02e | NiiRoZz | startup: accept compute execz live definitions whose reads are proven (#2557) | O passe de escrita de `exec` em `tools/startup/gfx2hlsl.py` passa a fazer varredura forward por caminho: aceita escrita de `exec` cujo resultado deixa o bit da lane comprovadamente limpo em todos os caminhos; definições vivas no join são aceitas quando toda leitura usa `W & exec`, `exec & W`, `~(W & exec)` ou `W \| ~exec`, e quando a máscara só é estreitada por `s_andn2 W, W, X` com `X` provado limpo. Cache de parse por texto derruba três requests grandes de 19,1 s para 8,4 s. Casos NM/O e controles `n-unmasked`/`o-mask-kept`. |
| 343dc04f | archielcqp-a11y | compute: lower group-shared LDS and s_barrier on gfx1013 (#1566) | `compute_store_aot.py` ganha o perfil LDS compartilhado: `--lds-bytes N` vira `groupshared uint lds[N/4]`, com `ds_write/read_b32/_b64/_b128`, `ds_write2/read2(_st64)_b32/_b64`, `ds_add/sub/and/or/xor/min/max_u32` via `Interlocked*` e `s_barrier`. Cada DS é seguido de `GroupMemoryBarrierWithGroupSync()` e o perfil é **uma wave convidada por grupo** (33–64 threads exigem `--wave-size 64`; grupos maiores são recusados). Acesso fora do contrato é limitado por `index < N/4`. Testes em WARP 1.0.22 e RTX 3080. |
| 418f9245 | Chad Hyatt | runtime: guard memory SRV wrapper allocation failures (#2727) | `unplaced_memory_buffer` e `publish_tiled_texture` (`src/startup_stubs/d3d12_memory_srv.inl`) passam a devolver `nullptr` quando `wrap()` falha, em vez de desreferenciar. O teste injeta falha com `x2w_d3d12_fail_wraps(1)` e exige que a publicação falha **não** altere a view de 32 bytes nem o registro de descritor. |
| 434aa90d | RonLogic | shader: name the unsupported implicit gather mip level (#2735) | `op_image_gather4` agora recusa com `image_gather4 requires the guest implicit mip level` em vez de cair no caminho de mip fixo, que leria o primeiro mip visível da view D3D e não o nível que o convidado pediu. Controle do teste fixa a mensagem. |
| a768ebe0 | JB2WIN | startup: lower s_bfm_b64 in gfx2hlsl (#2007) | `s_bfm_b64` (SOP2 opcode 37) passa a ser traduzido como `((1ULL << S0[5:0]) - 1) << S1[5:0]`, escrevendo as duas palavras do par destino pelo helper `shl64` e dobrando constantes. Fontes de máscara são recusadas **antes** de tocar nas palavras/flags; a prova de uniformidade vale para as duas palavras. Teste cobre 64×64 pares larg/offset e readback em WARP/hardware. |
| 091c2a48 | Asım Mirza Çiftler | shader: lower compute VOP1 bfrev and ffbh_i32 (#1591) | `v_bfrev_b32` vira `reversebits(x)` e `v_ffbh_i32` vira `firstbitlow(reversebits(x ^ ashr(x,31)))`, que é `31 - firstbithigh(m)` e dá `0xffffffff` para `0` e `-1`. `_e64`, SDWA/DPP, modificadores, ranges e os vizinhos `v_ffbh_u32`/`v_ffbl_b32` continuam recusados. `tests/compute_alu_vop1_check.py` compara 320 palavras em 70 lanes ativas. |
| a766f287 | triggered02 | shader: lower v_sad_hi_u8 and v_msad_u8 in gfx2hlsl (#2733) | Formas VOP3A `0x15b` e `0x171` entram em `VEC_U` apoiadas no helper sob demanda `sad4_u8(a,b,masked)`: `sad_hi_u8 = (sad4_u8(a,b,false) << 16u) + c`, `msad_u8 = sad4_u8(a,b,true) + c` (lane cujo byte de `src1` é zero fica fora da soma). O texto dos helpers `sad_u8/u16/u32` não muda. 864 triplas por forma, mais linhas fixas de zero e carry 2^32. |
| 6bf9bf1a | tedin7 | tests: replay the compiled gfx2hlsl compute cases in the GPU check package (#2728) | `tools/record_gpu_cases.py` grava os 23 casos e 22 controles de mutação de `startup_gfx2hlsl_compute_check.py` como `.cso`/`.case` mais `manifest.tsv`; `Run-GpuCheck.ps1` despacha cada um em `offscreen_check --compute-store-case` e soma a seção `gfx2hlsl_shaders`. Manifesto ausente ou vazio falha o pacote. |
| 2a9c252f | tedin7 | test: run startup_resource_address_index without MSVC Debug iterator checks (#2633) | Só build: o alvo `startup_resource_address_index_check` recebe `_ITERATOR_DEBUG_LEVEL=0` em MSVC Debug, porque as 180 000 buscas em `std::multimap`/`std::map` levavam minutos com os checks de iterador. O teste continua verificando as respostas de busca. |
| b408241e | RonLogic | test: cover wrapper allocation failures for placed resources (#2740) | `startup_wrap_allocation` cobre também o slot 133 em modo *placed*: buffer com endereço desalinhado e textura (que cria uma textura comprometida no desktop antes de tentar o wrap). O comentário do CMake passa a listar os dois caminhos do slot 133. |
| fd033046 | tedin7 | tests: make startup_run_ps1_check stable under parallel load (#2721) | `unwrap_first_line` junta as linhas em que o PowerShell quebrou um `Write-Warning` no limite da largura do host, **somente** quando a concatenação é exatamente o texto esperado; quatro controles cobrem uma linha, quebra em 80 colunas, texto diferente e aviso ausente. |
| 8ece2649 | SimSalabimse | shader: lower s_quadmask_b64 in gfx2hlsl (#1723) (#2678) | `s_quadmask_b64` colapsa cada nibble da fonte de 64 bits em um bit de `D.lo` (bit *g* = nibble *g* ≠ 0), `D.hi = 0` e `SCC = (D.lo \| D.hi) != 0`; constantes inline são de 64 bits e literais de 32 bits zero-estendem. Máscaras de lane e SGPR de descritor são recusados. |
| 83331487 | tedin7 | feat: accept key-independent body scalars read after a compute waterfall (#2737) | O `waterfall_refusal` ganha o conjunto `keyed`: uma escrita do corpo deixa de depender da chave quando nenhuma leitura do corpo leu valor `keyed` nem fez `buffer_load_dword` desde a última escrita. Isso aceita escalares de corpo lidos depois do laço sem aceitar o que carrega pelo índice da iteração. |
| 7f2ed4ce | Royoo | tests: distinguish two copied raw vertex SRVs in one table (#2743) | Só teste: `startup_address_srv`/`startup_stream_draw`/`startup_stream_setup` distinguem duas cópias de SRV de vértice cru na mesma tabela, para que um bind que troca as duas não passe. |
| caa24ad7 | Leyukaka | startup: copy descriptor words into vcc as the upper half of s[104:107] (#2669) | Carregar descritor em `s106/s107` marca `vcc_lost` e registra `('load_descriptor', valor)` por PC; a leitura de volta por cópia para `vcc` passa a ser aceita no modelo quando os valores são os do descritor. Sem isso, `vcc` lido depois de um load de descritor era recusado sempre. |
| 601eb621 | tedin7 | shader: lower v_bfm_b32 and missing VOP2/VOP3 ops in gfx2hlsl (Part of #1723) (#2087) | `v_bfm_b32` (`(((1u << (s0 & 31)) - 1) << (s1 & 31))`) entra na tabela de declarações; a grafia `_e32` desse opcode passa a ser recusada explicitamente, porque `v_bfm_b32` só existe em VOP3 (`0x363`). |
| 786970ac | tedin7 | startup: record a UAV of a resource without unordered access as not created (#2710) | `descriptor_has_live_view` consulta o slot-sombra e o token materializado antes de registrar UAV; um pedido de UAV sobre recurso sem `ALLOW_UNORDERED_ACCESS` é registrado como **não criado**, em vez de gerar um registro falso de UAV. |
| 651577a0 | tedin7 | feat: keep wave-uniformity through pure integer VALU ops in gfx2hlsl (#2722) | Ops inteiras de 32 bits puramente por lane preservam a prova de uniformidade (fonte iguais ⇒ resultado igual em toda lane, sem carry-out nem lane id). `UNIFORM_EXPR_LIMIT = 4096` limita a expressão que ainda vale recomputar para um `v_readfirstlane`. |
| a5817910 | tedin7 | shader: add a WARP readback for src_scc (#2725) | Novo readback D3D12 para `s_cmp_eq_u32` + `s_mov_b32 sX, src_scc`, comparando com a referência de CPU em WARP/hardware, com controle de lowering mutado (mesmo padrão de `s_ff1`). |
| e6902a11 | tedin7 | startup: lower constant-lane SGPR spills on wave64 lanes and in vertex stages in gfx2hlsl (Part of #1723) (#2712) | O modelo de spill por lane passa a cobrir lanes 0..63 (wave64) e estágios de vértice. Como wave32 usa a lane módulo 32, `k` e `k+32` são o mesmo slot ali e dois slots em wave64; o tradutor, que não conhece o tamanho da wave, recusa VGPR que contenha esse par. |
| 1ec01d70 | tedin7 | startup: lower DPP quad reads in pixel programs with uniform branches and loops (Part of #1723) (#2705) | DPP quad read passa a ser aceito quando o fluxo é provado uniforme: registra branches divergentes, o `s_wqm` que provou EXEC de quad inteiro e recusa o caso em que o offset cai em laço com saída divergente (`divergent_exit_loop`), onde a próxima passada veria quads incompletos. |
| 2462ab37 | tedin7 | shader: lower v_mad_u64_u32 and v_mad_i64_i32 in gfx2hlsl (Part of #1723) (#2047) | `D = S0 * S1 + S2` módulo 2^64 com dois fatores de 32 bits e `S2` de 64. A forma sem sinal escreve também o carry-out numa máscara de lane; a forma com sinal só é aceita com destino nulo descartável, porque o carry-out dela não está estabelecido. |
| fe5cf85d | tedin7 | gfx2hlsl: lower v_pk_mul_f16, v_pk_add_f16 and v_pk_fma_f16 (#2738) | Half empacotado: metade baixa com `op_sel[i]`, alta com `op_sel_hi[i]` (padrão `[0,0,0]`/`[1,1,1]`), ambas escritas sem zerar. `neg_lo`, `neg_hi`, `clamp` e fonte que não é VGPR simples são recusados. Modelo `tests/f16_packed_model.py`. |
| 594bb3ce | NiiRoZz | startup: bind a compute heap record whose index is read from a table buffer (#2580) | O índice de registro do heap pode vir de uma palavra lida de buffer de tabela: `mark_heap_index` separa a marcação de `s_lshl_b32` e o dispatch resolve `(tabela, byte, índice)`, com teste de 94+160 linhas para o caso de palavra e para o registro. |
| 5f27d8cf | tedin7 | shader: lower SDWA binary16 add, sub, subrev and mul in gfx2hlsl (#2739) | `sdwa_half`: `src0_sel`/`src1_sel` escolhem `WORD_0`/`WORD_1` (DWORD lê a metade baixa), `dst_sel` `WORD_0`/`WORD_1` com `UNUSED_PAD` ou `UNUSED_PRESERVE`; `dst_sel DWORD`, seleção de byte, `sext()`, `clamp` e modificadores de saída seguem sem lowering. |
| 6f696d48 | brainbudt | shader: lower two disjoint LDS dword fills before a compute barrier (#2751) | `check_lds_exchange` deixa de exigir **um** preenchimento contíguo: aceita dois preenchimentos de dword disjuntos antes da barreira uniforme, mantendo a exigência de que as leituras só toquem dwords inicializados. |
| 0498f937 | tedin7 | startup: fold s_mov_b32 copies of a constant SGPR so inline samplers translate (Part of #1723) (#2708) | `s_mov_b32` de um SGPR que guarda constante conhecida (não o placeholder marcado) passa a ser dobrado nessa constante — é o caso `s_mov_b32 s12, s15` depois de `s_mov_b32 s15, 0` que o llc emite para palavra de sampler inline. |
| 6fa22819 | tedin7 | shader: support 64-bit source operands in gfx2hlsl (Part of #1723) (#2039) | `i64_words` ganha `signed`/`literal`: literal de 32 bits zero-estende ou sign-estende conforme o helper (`shr64i`/com sinal), e o par low/high de VGPR/SGPR continua sendo a forma normal. Teste `startup_gfx2hlsl_shift64_literal_check.py`. |
| 400c0c76 | Chad Hyatt | tooling: preserve form reference output permissions (#2752) | `tools/form_reference.py` grava por arquivo temporário irmão e copia o modo POSIX do destino antes do `os.replace`, para que a referência não perca permissão de execução. Teste CLI cobre o caso. |
| d0093943 | tedin7 | tests, docs: optional per-file fragments for test registration, checks and experiment notes (#2655) | `tools/run_checks_parallel.py` ganha `--from-dir DIR` e `merge_checks(inline, directory)`, que acrescentam comandos listados em `DIR/*.txt`; `tests/check`/`tests/cmake` ganham fragmentos e um teste que valida os fragmentos. Reduz conflito em `CMakeLists.txt`/workflow. |
| bddd1071 | NiiRoZz | startup: bind LOD-clamped static samplers (smpl) in the translated compute Dispatch (#2585) | O parser do Dispatch de compute aceita a linha `smpl` (sampler estático com clamp de LOD) pelo mesmo decodificador do draw; `smpc` e `smpheap` passam a recusar com motivo nomeado em vez de cair no erro genérico. |
| 59218765 | tedin7 | startup: keep the implicit LOD of a plain image_sample past a uniform join in gfx2hlsl (#2699) | `resolve_plain_samples` preserva o LOD implícito de `image_sample` simples quando o sample fica depois de um join uniforme; se o offset está num laço com branch divergente, a tradução é recusada. Teste `startup_image_sample_implicit_lod_check.py`. |
| 1c061f87 | NiiRoZz | startup: read a uniform compute buffer load past the entry block (#2607) | Carga uniforme de buffer fora do bloco de entrada passa a guardar a palavra em `uniform_temps` (`ul<pc>_<n>`) para que toda lane mantenha o mesmo valor e o `v_readfirstlane` possa recomputá-lo. Teste `readfirstlane_load_recompute`. |
| c458bba9 | yjs | shader: lower remaining gfx1030 finite-normal buffer float atomics (#2756) | `fcmpswap` entra no conjunto; `fmin`/`fmax` viram laço de `InterlockedCompareExchange` sobre a palavra crua, agora com variante de 64 bits (`wide=True`). Perfil MUBUF e de imagem compartilham o mesmo helper. |
| c8b7b973 | NiiRoZz | startup: compute entry values from the words of a bound buffer descriptor (#2596) | Valores de entrada do compute podem vir do descritor do buffer ligado: `stride`, `descword3`, e para saída `uav_descword3`, `uav_stride`, `uav_numrecords`. Draw e Dispatch leem os mesmos campos; teste de 130 linhas. |
| 97265a92 | NiiRoZz | startup: name the condition a compute buffer input failed instead of one refusal (#2624) | `indirect_table_buffer` recebe `why` e devolve o motivo exato (endereço, offset, stride, bytes) em vez de uma recusa única; a mensagem chega ao log do draw e do dispatch. |
| da2a8f6f | Matheus Sergio | startup: lower compute global_load_dword saddr form in gfx2hlsl (#2715) | `global_load_dword` com endereço em SGPR (`saddr`) entra no subconjunto de compute, com guardas de encoding: `n != 8`, bit reservado 25, `nv` (bit 23 de `word1`) e campo de dados não usado são recusados nomeadamente. Teste de 826 linhas. |

## Pull requests abertos (63 novos; 20 documentados)

A lista de abertos do repositório tem **100 PRs** (mínimo #97, máximo #2914); **63** têm número acima do cursor
2725. Foram documentados os 20 de menor número, para que `last_pr_number` avance de forma monotônica; os outros
43 ficam para a próxima execução.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 2749 | shader: run ordinary gfx1030 biased gather probes through Vulkan | Entrada Vulkan source-built para os casos de gather com bias de #1643, com mips RGBA32F 8×8/4×4/2×2 e leitura exata contra referência de CPU em `image_gather4_b`/`_b_o`. | Não. Área de `image_gather4*`, sem ID no catálogo. |
| 2750 | shader: probe native gfx1030 2D gather clamps | Sondas nativas para `image_gather4_cl`, `_b_cl`, `_c_cl`, `_c_b_cl` e as quatro formas com offset assinado, mais um draw de `image_get_lod`. | Não. |
| 2753 | shader: lower experimental gfx1030 2D gradient sample clamps | Estende o experimento opt-in `--sample-clamp-mips` a oito formas de gradiente (`image_sample_d_cl`, `_cd_cl`, `_g16` etc.) traduzidas para `SampleGrad` com LOD mínimo conhecido. | Não. Experimento atrás de flag. |
| 2766 | tests: serve GPU draw cases from one offscreen_check device | Cria o device D3D12 uma vez por processo de teste em vez de uma vez por caso (~0,25 s por launch contra ~0,03 s por caso). | Não; fecha #448 (custo de teste). |
| 2773 | gpu: defer CreateCommittedResource for read-only slot 133 textures | Materialização lazy opt-in (`X2W_D3D12_LAZY_TEXTURES=1`) para texturas slot 133 somente-leitura, adiando `CreateCommittedResource` e o commit do orçamento. | Não; declara corrigir #2764 e referenciar #2379/#1199. |
| 2782 | perf: remember a refused buffer memory SRV for a short, validated time | Memoiza a recusa `buffer_unavailable` do slot 70 por tempo curto validado, para títulos que republicam o mesmo SRV recusado a cada frame (log citado: 654 893 recusas). | Não; parte de #1723. |
| 2794 | test: allow one 8-bit filter weight step in the linear sampler probes | Troca a tolerância fixa de 0,01 por um passo de peso de 8 bits nas sondas de filtro linear, por causa de quantização legal em GTX 1080 Ti. | Não; fecha #2781. |
| 2817 | startup: print raw= and completed= in the DirectStorage status lines | `format_request_counts()` passa a imprimir `raw=/inflated=/failed=/bcpack-blank=/completed=` nas linhas de status, para que pedidos comprimidos não apareçam como `done=0`. | Não; fecha #2816. |
| 2820 | startup: warn when the shader cache and the translator come from different revisions | Compara as três strings de revisão que iniciam as chaves de request e emite um aviso quando cache e tradutor vêm de revisões diferentes, em vez de "nada traduz". | Não; fecha #2815. Parente do D-05 do OpenScarlett (chave de cache sem versão do tradutor), mas em outro repositório. |
| 2822 | startup-tools: save the title's window every N seconds (--capture-every) | `debug_launch.py --capture-every <s>` (e `run.ps1 -CaptureEvery`, `run.sh --capture-every`) salva a janela do título com o tempo, para ler uma execução que para numa tela tardia. | Não; fecha #2819. |
| 2827 | build: startup_runtime target and runtime preset for a title run | Alvo `startup_runtime` e preset `runtime` que constroem só as oito DLLs de stand-in mais o carimbo de revisão, em vez da árvore inteira de testes. | Não; fecha #2826. |
| 2829 | pe: load pe_run_package providers without a critical-error dialog | Põe `SEM_FAILCRITICALERRORS` ao redor de `LoadLibraryExW` dos provedores para que uma imagem inválida (`0xc0000020`) não abra diálogo modal em sessão interativa. | Não. Corrige falha de teste observada em `pe_run_package_cli_check`. |
| 2833 | startup: system commit, job limit and page-pool counters in the budget line | Novo `commit_state.h` com `system_commit()` (`GlobalMemoryStatusEx`), `job_memory()` (`QueryInformationJobObject`) e contadores de page-pool, para atribuir erro 1455/8 no log. | Não; fecha #2832. |
| 2836 | startup: name the cause of a slot 70 texture_unavailable refusal in the memory SRV census | Separa as três causas de `texture_unavailable` do slot 70 e as nomeia no censo, no log por chamada e em `triage_run.py` (`SLOT70_REASONS`). | Não; fecha #2831. |
| 2841 | startup: slot 70 structured buffer views at sub-allocated addresses take an exact-start upload copy | SRV de buffer estruturado em endereço subalocado (não múltiplo do elemento depois da página de 64 KiB) passa a publicar com cópia de upload começando no endereço exato. | Não; fecha #2839. |
| 2846 | startup: refuse unmapped source ranges before a probe or a copy | `mapped_prefix()` consulta `VirtualQuery` com cache por thread e recusa só `MEM_FREE`, evitando uma access violation de primeira chance por leitura de memória não mapeada. | Sim, indiretamente: é o sintoma descrito em **D-13** (triagem conta first-chance step-over como parada de debugger). Muda diagnóstico, não a contagem. |
| 2853 | Rebase & merge: SDWA, class comparison, 64-bit shifts + RTX 4070 Ti SUPER report | Rebase de trabalho de SDWA, comparação de classe e shifts de 64 bits, mais um relatório de hardware RTX 4070 Ti SUPER. Declara substituir #2701. | Não. Mistura código e relatório de hardware num PR. |
| 2857 | pixel_aot: lower 10 binary16 cvt_norm forms across VOP1/VOP3/DPP16/DPP8/SDWA | Dez formas `v_cvt_norm_i16_f16`/`v_cvt_norm_u16_f16` nas cinco codificações em `tools/pixel_aot.py`, atualizando `docs/coverage/*.tsv`. | Não; avança #839. |
| 2859 | startup: X2W_LOW_ADDRESSES keeps the title's own memory below 2^40 | Launcher arma `PROCESS_CREATION_MITIGATION_POLICY_BOTTOM_UP_ASLR_ALWAYS_ON` para manter alocações do título abaixo de 2^40, para o construtor de acceleration structure do pacote. | Não; fecha #2858. |
| 2861 | shader: lower bounded per-lane heap record indices in compute waterfalls (#2644 T3) | Rastreio de limite superior de VGPR por CFG (`v_and_b32`, `v_min_u32`, `v_bfe_u32`, `v_lshrrev_b32`, `v_lshlrev_b32 ...,5`, `v_mov_b32`) para aceitar índice de registro de heap limitado por lane dentro de waterfall de `v_readfirstlane`. | Não; completa T3 de #2644. |

## Pull requests mesclados (38 dentro da faixa documentada)

A busca `merged:>=2026-11-26T04:15:00Z` devolveu **110 merges** no repositório. Desses, **38** têm
`mergeCommit.oid` igual a um dos 40 commits acima e estão listados; os outros pertencem aos 72 commits que ficam
para a próxima execução. Dois commits da faixa (`2215c26d`, `feec219e`) não aparecem na busca de mesclados desde
o cursor — pelo filtro `merged:>=`, isso significa `mergedAt` anterior a `2026-11-26T04:15:00Z` —, embora seu
diff esteja documentado.

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| 1566 | compute: lower group-shared LDS and s_barrier on gfx1013 | 343dc04f | commit 5 da tabela acima (2026-11-26T04:33:02Z) |
| 1591 | shader: lower compute VOP1 bfrev and ffbh_i32 | 091c2a48 | commit 9 da tabela acima (2026-11-26T04:51:39Z) |
| 1902 | runtime: validate ApuAlloc outputs and synchronize AcpHal heap range queries | 1c3abf59 | commit 3 da tabela acima (2026-11-26T04:25:35Z) |
| 2007 | startup: lower s_bfm_b64 in gfx2hlsl | a768ebe0 | commit 8 da tabela acima (2026-11-26T04:48:34Z) |
| 2039 | shader: support 64-bit source operands in gfx2hlsl (Part of #1723) | 6fa22819 | commit 31 da tabela acima (2026-11-26T06:04:40Z) |
| 2047 | shader: lower v_mad_u64_u32 and v_mad_i64_i32 in gfx2hlsl (Part of #1723) | 2462ab37 | commit 25 da tabela acima (2026-11-26T05:47:38Z) |
| 2087 | shader: lower v_bfm_b32 and missing VOP2/VOP3 ops in gfx2hlsl (Part of #1723) | 601eb621 | commit 19 da tabela acima (2026-11-26T05:32:50Z) |
| 2557 | startup: accept compute execz live definitions whose reads are proven | 1226f02e | commit 4 da tabela acima (2026-11-26T04:28:01Z) |
| 2580 | startup: bind a compute heap record whose index is read from a table buffer | 594bb3ce | commit 27 da tabela acima (2026-11-26T05:50:39Z) |
| 2585 | startup: bind LOD-clamped static samplers (smpl) in the translated compute Dispatch | bddd1071 | commit 34 da tabela acima (2026-11-26T06:08:14Z) |
| 2596 | startup: compute entry values from the words of a bound buffer descriptor | c8b7b973 | commit 38 da tabela acima (2026-11-26T06:15:45Z) |
| 2607 | startup: read a uniform compute buffer load past the entry block | 1c061f87 | commit 36 da tabela acima (2026-11-26T06:11:54Z) |
| 2624 | startup: name the condition a compute buffer input failed instead of one refusal | 97265a92 | commit 39 da tabela acima (2026-11-26T06:25:56Z) |
| 2633 | tests: run startup_resource_address_index without MSVC Debug iterator checks | 2a9c252f | commit 12 da tabela acima (2026-11-26T05:02:35Z) |
| 2655 | tests, docs: optional per-file fragments for test registration, checks and experiment notes | d0093943 | commit 33 da tabela acima (2026-11-26T06:06:58Z) |
| 2669 | startup: copy descriptor words into vcc as the upper half of s[104:107] (Part of #2023) | caa24ad7 | commit 18 da tabela acima (2026-11-26T05:14:49Z) |
| 2678 | shader: lower s_quadmask_b64 in gfx2hlsl (#1723) | 8ece2649 | commit 15 da tabela acima (2026-11-26T05:11:43Z) |
| 2699 | startup: keep the implicit LOD of a plain image_sample past a uniform join in gfx2hlsl | 59218765 | commit 35 da tabela acima (2026-11-26T06:08:45Z) |
| 2705 | startup: lower DPP quad reads in pixel programs with uniform branches and loops (Part of #1723) | 1ec01d70 | commit 24 da tabela acima (2026-11-26T05:42:48Z) |
| 2708 | startup: fold s_mov_b32 copies of a constant SGPR so inline samplers translate (Part of #1723) | 0498f937 | commit 30 da tabela acima (2026-11-26T06:04:21Z) |
| 2710 | startup: record a UAV of a resource without unordered access as not created | 786970ac | commit 20 da tabela acima (2026-11-26T05:35:09Z) |
| 2712 | startup: lower constant-lane SGPR spills on wave64 lanes and in vertex stages in gfx2hlsl (Part of #1723) | e6902a11 | commit 23 da tabela acima (2026-11-26T05:40:08Z) |
| 2715 | startup: lower compute global_load_dword saddr form in gfx2hlsl | da2a8f6f | commit 40 da tabela acima (2026-11-26T06:27:26Z) |
| 2721 | tests: make startup_run_ps1_check stable under parallel load | fd033046 | commit 14 da tabela acima (2026-11-26T05:03:02Z) |
| 2722 | feat: keep wave-uniformity through pure integer VALU ops in gfx2hlsl | 651577a0 | commit 21 da tabela acima (2026-11-26T05:35:27Z) |
| 2725 | shader: add a WARP readback for src_scc | a5817910 | commit 22 da tabela acima (2026-11-26T05:39:50Z) |
| 2727 | runtime: guard memory SRV wrapper allocation failures | 418f9245 | commit 6 da tabela acima (2026-11-26T04:36:26Z) |
| 2728 | tests: replay the compiled gfx2hlsl compute cases in the GPU check package | 6bf9bf1a | commit 11 da tabela acima (2026-11-26T04:57:00Z) |
| 2733 | shader: lower v_sad_hi_u8 and v_msad_u8 in gfx2hlsl | a766f287 | commit 10 da tabela acima (2026-11-26T04:52:04Z) |
| 2735 | shader: name the missing implicit mip level for gather4 | 434aa90d | commit 7 da tabela acima (2026-11-26T04:45:03Z) |
| 2737 | feat: accept key-independent body scalars read after a compute waterfall | 83331487 | commit 16 da tabela acima (2026-11-26T05:13:20Z) |
| 2738 | gfx2hlsl: lower v_pk_mul_f16, v_pk_add_f16 and v_pk_fma_f16 | fe5cf85d | commit 26 da tabela acima (2026-11-26T05:47:58Z) |
| 2739 | shader: lower SDWA binary16 add, sub, subrev and mul in gfx2hlsl | 5f27d8cf | commit 28 da tabela acima (2026-11-26T05:57:38Z) |
| 2740 | tests: cover wrapper allocation failures for placed resources | b408241e | commit 13 da tabela acima (2026-11-26T05:02:39Z) |
| 2743 | tests: distinguish two copied raw vertex SRVs in one table | 7f2ed4ce | commit 17 da tabela acima (2026-11-26T05:13:53Z) |
| 2751 | shader: lower two disjoint LDS dword fills before a compute barrier | 6f696d48 | commit 29 da tabela acima (2026-11-26T06:02:04Z) |
| 2752 | tooling: preserve form reference output permissions | 400c0c76 | commit 32 da tabela acima (2026-11-26T06:05:12Z) |
| 2756 | shader: lower remaining gfx1030 finite-normal buffer float atomics | c458bba9 | commit 37 da tabela acima (2026-11-26T06:13:05Z) |

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum dos 40 hashes é o `corrigido` de um defeito do `catalogo/defeitos.csv` para
  `Xbox2Windows` (D-01…D-29 têm hashes dos dossiês). O que existe é proximidade:
  - `2215c26d` endurece a CLI do `hardware_reports`, mesma ferramenta de **D-22** (`pe_inventory` quebrava com
    inventário malformado), mas outro comando.
  - `97265a92` e `bddd1071` trocam recusa genérica por motivo nomeado — o espírito de **D-20**
    (`SHADER_REASONS` nunca exercitada), sem tocar naquela tabela.
  - `59218765` mexe em `image_sample` com LOD implícito, mesma família de **D-27** (`a16` lido como dois
    registradores), mas o conserto de D-27 é `c3c3ab3c` e este commit não altera `a16`.
- **Abre defeito novo?** Três candidatos, **NAO VERIFICADO** (não há caso de título nem log que os confirme):
  1. `343dc04f` — uma barreira `GroupMemoryBarrierWithGroupSync()` após **cada** instrução DS reproduz a ordem
     de uma única wave convidada; o próprio diff documenta que grupos de mais de uma wave são recusados. Se
     algum dia o `--wave-size 64` for aceito com grupo de 32×2, a ordem entre waves passa a ser imposta pelo
     host e o resultado difere do convidado (`tools/compute_store_aot.py`, `shared_lds_step`).
  2. `786970ac` — `descriptor_has_live_view` (`src/startup_stubs/d3d12_capture.inl`) passa a registrar como
     **não criado** o UAV de recurso sem `ALLOW_UNORDERED_ACCESS`. Se algum título escrever por esse UAV usando
     o token materializado, o registro deixa de existir e o rastreio de escrito (write-watch / `note_resource`)
     pode não cobrir a região. Sem teste de título.
  3. `1226f02e` — a prova de `execz` aceita definições vivas no join quando toda leitura usa `W & exec` etc. A
     regra é mais ampla que a anterior (que só aceitava máscara de lane com zero antes do branch); um padrão de
     leitura que o passe classifique como absorvida mas que o hardware trate como dado real passaria a traduzir
     errado em vez de recusar. Nenhum caso de título citado no diff.
- **Mantido por desenho:** `b408241e` documenta no CMake a mudança de #2653 (alocação de wrapper devolve
  `E_OUTOFMEMORY` com saída nula nos slots 8, 9, 14, 16, 55, 73, 127 e 133); `418f9245` cobre dois desses
  caminhos no runtime.

## Confiança

- **Verificado por diff:** os 40 hashes acima foram lidos em `gh api repos/R0Y00/Xbox2Windows/commits/<sha>`
  (arquivos, estatísticas e hunks), não pela mensagem. Hash, autor e assunto vêm do mesmo objeto.
- **Teto aplicado:** 40 de 112 commits. `last_commit`/`commit_cursor` ficam em `da2a8f6f` (40º);
  `4992a46e` é o head no momento da coleta. Os 72 restantes não foram lidos.
- **Merges:** 110 na janela; 38 casados por `mergeCommit.oid` com os commits documentados. A diferença para 110
  corresponde aos commits não documentados. `mergedAt` máximo entre os 38 é `2026-11-26T06:27:26Z`, que é o novo
  cursor de PRs mesclados.
- **PRs abertos:** 63 novos; 20 documentados (título e proposta vêm do corpo do PR, não do diff). O `last_pr_number`
  avança para 2861; os 43 restantes ficam para a próxima execução.
- **NAO VERIFICADO:** os três candidatos a defeito novo; qualquer comportamento em hardware real dos commits
  `343dc04f` (só WARP/RTX 3080 no relatório do autor), `1226f02e` e `1ec01d70` (provas de fluxo, sem caso de
  título); e o efeito de `c458bba9` em GPUs que não sejam gfx1030.
