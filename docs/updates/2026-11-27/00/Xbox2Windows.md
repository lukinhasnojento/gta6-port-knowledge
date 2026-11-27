# R0Y00/Xbox2Windows — atualização de 2026-11-27 00:00

## Resumo

Dois commits novos na `main` (`6ee878d8..20a497c2`), e dezoito PRs mesclados na janela — mas **dezesseis
deles foram mesclados em `cpp-migration`, não na `main`**. A `main` recebeu só a linha de cobertura de
hardware da RX 7900 XT (`eff2e27a`) e o backport do clear explícito do alvo de pixel-history
(`20a497c2`). Todo o resto — dezoito commits de tradução de shader no frontend C++ nativo, mais a
habilitação de CI nessa branch (`#2942`) — entrou na `cpp-migration`, que não é a branch default e não é
rastreada por `state/watch.json`. Quarenta e cinco PRs abertos novos, dos quais 41 miram `main` e 4 miram
`cpp-migration`.

## Commits novos (2)

Faixa `6ee878d8..20a497c2`. A `main` avançou exatamente dois commits, e são exatamente os dois PRs da
janela cujo `baseRefName` é `main`.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `eff2e27a` | xiaoqi72665 | docs: add RX 7900 XT hardware coverage (#2982) | 1 arquivo, +1/-0 (`README.md`). Uma linha nova na tabela de cobertura de hardware: AMD Radeon RX 7900 XT, driver 32.0.32029.2010, 32 lanes, `bvh_waves` passa, `bvh_wave64` **pulado** porque o Shader Model está limitado a 6.5 no Windows 10 build 19045, `bvh_differential` passa com 408 comparações, camada de debug do D3D12 indisponível. Só documentação, nenhum código. |
| `20a497c2` | chadhyatt | gpu: explicitly clear the pixel-history fixture target (#2868) | 1 arquivo, +6/-12 (`tests/startup_pixel_history.cpp`). `renew_texture()` criava um `CreateCommittedResource` novo e assumia que o alvo nascia preto; o valor de clear otimizado passado nessa chamada **não inicializa** o conteúdo (contrato da API D3D12), então o fixture dependia de o driver entregar zeros. Troca por `clear_target()`, que dá `Reset` na lista, chama `ClearRenderTargetView(target.rtv, black, …)` e submete com espera; é chamado uma vez antes da sequência principal e de novo antes do controle negativo. O oráculo de readback/histórico fica intacto. Backport de `x2w-microlift`; verificado só como cross-compile de sintaxe em macOS ARM64, sem execução em MSVC/GPU. |

## Pull requests abertos (45)

Todos com número acima do cursor `2887`. Quarenta e um miram `main`; `#2984`, `#2981`, `#2975` e `#2912`
miran `cpp-migration`. Quatro estão em rascunho (`#2960`, `#2938`, `#2930`, `#2912`).

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| `2984` | tooling: write .fail diagnostics and add read-only --refusals summary to compile_dxil | Ciclo de vida por pedido do `<stem>.fail` em `compile_dxil --requests` e resumo `--refusals` só de leitura; fecha #2983. +692/-19 em 8 arquivos | Área do **D-19** (documentação do ciclo de `.fail`), que já foi corrigido em `5b29fe0f`; este PR reescreve o ciclo, não reabre o defeito |
| `2981` | runtime: compile a supported graphics pair on first use in the stand-in | Com `X2W_NATIVE_SHADERS`, compila VS/PS em processo no primeiro uso e cria o pipeline pelo manifesto nativo. +122/-5, 5 arquivos. Testado em WARP e RTX 4060 Laptop | Não |
| `2979` | shader: track both VCC halves for unsigned MAD64 carry | Inclui `v_mad_u64_u32` na classificação de "pode escrever a metade alta" para o carry não sobreviver a bloco pulado; gira as chaves de cache de gráficos e computação. +44/-19, 12 arquivos | Não |
| `2977` | startup: opt-in alias binding for table buffer views over aliased placed buffers | `X2W_D3D12_TABLE_ALIAS=1` liga a view à faixa de placed buffer que ela cobre em vez de copiar 16 MiB por draw; fecha #2976. +205/-0, 7 arquivos | Não |
| `2975` | runtime: bind 3D and 1D texture UAVs and write them in the C++ compute manifest | Metade de runtime do #2902 para UAVs `tex3d`/`tex1d`, tirada sem alteração do PR aberto na `main`. +271/-38, 7 arquivos | Não |
| `2970` | shader: lower two immutable samplers at uniform explicit-LOD joins | Admite um único diamante SCC provado que escreve as quatro palavras de dois samplers imutáveis e escolhe entre `SampleLevel` completos. +368/-9, 10 arquivos | Não |
| `2968` | runtime: retain presented readback ownership until fence completion | Diagnóstico da superfície apresentada passa a checar `Close`/`Signal`/fence/`Map` e a reter superfície e fila até a cópia terminar. +344/-16, 8 arquivos | Área do **D-01** (superfície apresentada liberada durante a cópia da assinatura, corrigido em `2a898640`), mas o objeto é o diagnóstico de readback, não a cópia da assinatura — não corrige |
| `2961` | startup: bind compute outputs named by a buffer UAV view token | Liga saídas de Dispatch nomeadas por token de view de buffer (slot 19), hoje recusadas com "an output is not an aliased placed buffer range with unordered access". +68/-7 | Não |
| `2960` | startup: lower cube image sampling in gfx2hlsl | `image_sample[_lz/_l/_b]` com `dim:SQ_RSRC_IMG_CUBE` viram `TextureCube` e o manifesto ganha o kind `texcube`; fecha #2943. +589/-29, 15 arquivos | Não (a forma `a16` 2D do **D-27** é outra) |
| `2956` | shader: admit proven calculated far early-outs | Extende o perfil de dois exits do pixel gfx1013 com um salto calculado provado para continuação terminante. +664/-13 | Não |
| `2954` | shader: lower group-shared ds_sub_u32 as two additions | `ds_sub_u32` deixa de sair como `InterlockedAdd(lds[i], 0u - v)`, que faz o `CreateComputePipelineState` devolver `E_OUTOFMEMORY` numa RTX 5080; passa a duas adições. +47/-8 | **Abre defeito novo** (não catalogado): compilador NVIDIA recusa o PSO com `InterlockedAdd` sobre `groupshared` com operando negado; arquivo `tools/compute_store_aot.py` |
| `2953` | startup: bind compute root SRV buffers (srv <i> buf root <param>) at Dispatch | Consome `srv <i> buf root <param>` no manifesto e liga o buffer de root SRV gravado no `Dispatch`; rastreia endereço por parâmetro de root em slot 43. +605/-9 | Não |
| `2949` | startup: opt-in release of upload ring chunks at a fenced submit | `X2W_D3D12_RING_RELEASE_AT_SUBMIT=1` solta os chunks do anel de upload quando o submit é cercado por fence em vez de esperar `Reset`; fecha #2948. +296/-8 | Não |
| `2947` | shader: lower startup 2D-array image LOD queries | Consultas de LOD em SRV 2D-array consomem o endereço de slice mas consultam só com coordenadas espaciais. +175/-48 | Não |
| `2938` | startup: bind cube views of console-layout textures in slot 70 | Metade de runtime para views cúbicas no slot 70; rascunho empilhado em #2929/#2926, sem tradutor que leia as views ainda. +699/-65 | Não |
| `2930` | startup: accept SGPR, hex and inline 0 sources in packed f16 add, mul and fma | `v_pk_add_f16`/`v_pk_mul_f16`/`v_pk_fma_f16` aceitam fonte única em VGPR, SGPR numérico, literal hex e `0` inline, como min/max do #2927. +355/-14 | Não |
| `2929` | startup: read one-dimensional linear (0x100) textures in slot 70 | Descrição `0x100` 1D no slot 70 deixa de ser recusada; empilhado em #2926. +478/-54 | Não |
| `2927` | startup: lower v_pk_max_f16 and v_pk_min_f16 in gfx2hlsl | Lowering por lane com `packed_half`, com `op_sel`/`op_sel_hi` por metade. +254/-8 | Não |
| `2926` | startup: read linear (0x100) textures in slot 70 | Extensão de superfície linear vinda do `Gfx10Lib::HwlComputeSurfaceInfoLinear` do addrlib; antes só um texel era aceito. +315/-31 | Não |
| `2925` | startup: back off the retry for an absent shader cache manifest | Reconsulta de `<key>.txt` ausente passa de 1 s fixo para 1→32 s com reset no acerto, dentro de `x2w_startup::ManifestRetry`. +140/-12 | Área do **D-18** (compilação não retomada depois de rodada manifest-only, `b87a99a3`); aqui é cadência de retry, não retomada — não corrige |
| `2924` | docs: plan slot 57 mesh DXIL translation | Plano em `docs/experiments/mesh57-dxil-plan.md` para a primeira tradução completa de mesh fixo do slot 57. +167/-0 | Não |
| `2923` | docs: plan EXEC removal and structurization | Plano para trocar o tratamento de EXEC padrão a padrão por um passe de estruturação, com as famílias medidas de `.fail` (3.007 registros). +223/-0 | Não |
| `2922` | startup: lower the FP32 box case of image_bvh_intersect_ray | Fatia estreita: só forma de 32 bits e nós de caixa. +665/-4 | Não |
| `2921` | tests: model the console BVH node set | Modelo puro em Python do teste de nó BVH do console, sobre o fixture gfx1030. +1258/-2 | Não |
| `2920` | startup: gate the BVH builder compute stages all or none | `X2W_COMPUTE_BUILDER_GATE=1` impede rodar estágios do builder que dependem de dados de estágio ausente; o caso observado rodou 14 de 35. +519/-3 | Não |
| `2919` | startup: persist and reuse translated PSO recipes | `X2W_PSO_RECIPE_DIR=<dir>` guarda o estado efetivo de blend/depth/raster/formato que a cache de shader não guarda. +1050/-39 | Não |
| `2916` | tooling: add --assert-sites to the startup debugger | `--assert-sites N` conta sites distintos de `int3` em vez de reusar o orçamento de `--continue-asserts`. +56/-12 | Não |
| `2913` | startup: lower v_ldexp_f32 to ldexp32 in gfx2hlsl | `ldexp32` bit a bit no preâmbulo comum, preservando zero com sinal, subnormal, infinito e NaN. +397/-6 | Não |
| `2912` | test: native DXIL read of a GPU-selected descriptor heap slot (probe) | Sonda: leitura de `ResourceDescriptorHeap[index]` num módulo DXIL nativo; não captura nem reencena tipos de view ainda. +559/-4 | Não |
| `2911` | xmem: make page runs cheaper under Wine | Reduz o custo de pedidos de página sob Wine. +3211/-164, 23 arquivos | Não |
| `2910` | startup: refuse compute buffer loads through a modified descriptor | `Stage.buffer_address()` passa a recusar quando palavras 1–3 do descritor foram editadas sem transformação provada; fecha #2901. +139/-9 | Não |
| `2904` | test: add startup_compute_dispatch C++ test | Teste C++ autônomo de Dispatch com slots 33/25 numa lista de comando do console. +366/-61 | Não |
| `2903` | startup: keep s_cselect_b32 on descriptor data symbolic in compute | `s_cselect_b32` entre `descnum` e zero deixa de emitir HLSL e passa a carregar símbolo; leitura numérica continua recusando. +535/-0 | Não |
| `2902` | startup: lower compute image_store to 1D and 3D texture UAVs | `image_store` com `dmask:0xf` vira `RWTexture3D`/`RWTexture1D`; metade de runtime no #2902/#2975. +274/-46 | Não |
| `2900` | fix(shader): prove scalar runs before compute EXEC restores | Prova que lanes que saem não observam trabalho do caminho que fica antes de restaurar EXEC salvo. +222/-12 | Não |
| `2899` | startup: accelerate gfx2hlsl cache scans and support skipping earlier failures | Troca `os.listdir`/`exists`/`getmtime` por `os.scandir` num passo só em `pending_keys`/`request_keys`. +259/-60 | Não |
| `2897` | startup: lower s_bitset0_b64 and s_bitset1_b64 in gfx2hlsl | Limpa/liga o bit `S0[5:0]` no par de SGPR sem tocar SCC, com fold e cuidado de alias da metade baixa. +469/-0 | Não |
| `2895` | gpu: harden atomic manifest writes and path validation in record_gpu_cases | Valida `dxc` e diretório de saída, grava `manifest.tsv` atomicamente e adiciona suíte de CLI. +368/-9 | Não |
| `2894` | shader: harden input validation and bridge conflict guards in spirv_sampler_views | Valida tipos, `OpTypePointer`/`OpTypeArray` ausentes e literais ASCII de decoração. +390/-6 | Não |
| `2893` | cpu: harden bounds and alignment checks in cpu_ranges | Rejeita `bool`/negativo/desalinhado e valida `.pdata`, preservando a cadeia de unwind. +385/-1 | Não |
| `2892` | tests: classify mad() fusion from the thread-dependent mac_f32 words | Separa modelo fundido e não fundido nas palavras dependentes de thread; a palavra 0 é dobrada pelo DXC. +43/-7 | Não |
| `2891` | startup: lower s_cmp_eq_u64 and s_cmp_lg_u64 in gfx2hlsl | Compara as duas metades e escreve SCC via `Stage.set_scc`. +403/-0 | Não |
| `2890` | cpu: add dedicated cpu_adx_aot lifter for 32-bit and 64-bit ADCX and ADOX | Lifter AOT com isolamento exato de CF/OF e recusa estática de flags indefinidas. +1825/-0 | Não |
| `2889` | shader: harden directory, symlink, and output path guards in probe_root_devices | Guardas de diretório/symlink/saída e suíte de CLI. +556/-3 | Não |
| `2888` | ffx: harden bounds and stage validation in ffx_scratch_reference | Valida `simm16`, `stage` em (24, 26) e offsets de `s_buffer_load_dwordx16`. +310/-1 | Não |

## Pull requests mesclados (18)

Dezesseis com `baseRefName = cpp-migration`; dois (`#2982`, `#2868`) com base `main`, e são exatamente os
dois commits que a `main` ganhou.

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| `2982` | docs: add RX 7900 XT hardware coverage | `eff2e27a` (main) | Uma linha em `README.md`; igual ao commit |
| `2973` | shader: lower the SDWA forms of binary16 add, sub, subrev and mul in direct compute | `7bcff286` | +242/-17 em 8 arquivos. As formas SDWA de `v_add_f16`/`v_sub_f16`/`v_subrev_f16`/`v_mul_f16`, que o #2964 recusava, passam a baixar no compute direto seguindo `sdwa_half()` do `gfx2hlsl.py`: `src*_sel` escolhe a metade, cada metade vai pelo caminho binary32 e é arredondada uma vez para binary16 por `packed_half_rne`. Entra teste `native_sdwa_f16_check.py` (+152) |
| `2972` | native: capture and replay view types for GPU-selected global SRV slots | `05438d65` | +1503/-35 em 17 arquivos. `x2w::HeapViewContract` (`none`/`raw_buffer`/`other`) autoriza a tradução da carga de descritor da heap em SMEM; sem contrato continua a recusa. Sonda `native_heap_handle_probe.cpp` (+292) e teste de replay (+451). Continua o #2912 e fecha #2908 |
| `2971` | shader: lower image_gather4_c_lz in the direct C++ image path | `2b678fd3` | +169/-11 em 4 arquivos. `image_gather4_c_lz` (MIMG 79) vira `textureGatherCmp` (DXIL 74) para 2D e 2D-array com `dmask 0x1`, canal 0, sampler de comparação constante; corrige de passagem o LOD estático do `image_gather4_lz` para mip zero |
| `2967` | shader: accept byte-window buffer descriptors for stores in the direct C++ compute path | `78a071ca` | +169/-2. `binding()` aceita, para `buffer_store_raw`, o mesmo padrão de janela de bytes que já valia para cargas (stride zerado em `& 0xc000ffff`, `s_bfe`/`s_max` sobre o stride, `OOB_SELECT` 3); liga o descritor não modificado, como o `gfx2hlsl.py` |
| `2966` | shader: lower root-pointer global_load_dword in the direct C++ compute path | `0b1456ce` | +264/-0. Forma `saddr`: ADDR = SADDR + zext(VADDR) + sext(OFFSET), com SADDR no endereço raiz de um root SRV; guarda de wrap em [0, 2^32) e alinhamento de DWORD, iguais ao `op_global_load_dword` |
| `2964` | shader: lower binary16 add, sub, subrev and mul in direct compute | `8946fb16` | +815/-0. Formas VOP2 e `_e64` de `v_add_f16`/`v_sub_f16`/`v_subrev_f16`/`v_mul_f16`: abs/neg por bit de sinal, operação em f32 e arredondamento inteiro nearest-even; metade alta zerada. SDWA/DPP, clamp/omod e `op_sel` alto continuam recusados. Teste de 586 linhas |
| `2959` | shader: lower s_bfm_b64, s_cselect_b64 and s_not_b64 in the direct C++ compute path | `f934df52` | +222/-3. `s_bfm_b64` vira `((1 << (S0 & 63)) - 1) << (S1 & 63)` com as duas palavras escritas e SCC intacto; `s_cselect_b64` seleciona as duas palavras por SCC; liveness passa a nomear os `s_cselect_b32/_b64` como leitores de SCC |
| `2958` | runtime: translate slot 67 compute requests in the stand-in | `b260984f` | +377/-17 em 10 arquivos. Com `X2W_NATIVE_SHADERS` apontando para `dxcompiler.dll`/`dxil.dll`, o stand-in traduz um pedido de compute do slot 67 em processo no primeiro `Dispatch`, sem cache de shader e sem vigia Python. O corpo diz explicitamente que **não** usa `NativeRecording`/`NativeDispatchStates` |
| `2955` | shader: consume startup requests and publish runtime cache entries in compile_dxil | `e3670100` | +977/-101 em 15 arquivos. `compile_dxil --requests` consome pedidos de revisão 28, incluindo `req_<key>.psin` ASCII (com recusa de linha malformada e de `ena & ~addr != 0`), e publica binários por estágio e manifestos `<key>.txt` que os carregadores de runtime aceitam; teste de 419 linhas |
| `2952` | shader: accept inline, literal and SGPR-pair values in direct C++ 64-bit shifts | `49e0e36d` | +163/-3. `v_lshrrev_b64`/`v_lshlrev_b64` aceitam inteiro inline (-16..64, estendido com sinal), literal de 32 bits (zero-estendido) e par de SGPR par (`k <= 52`), como `i64_words`; `v_ashrrev_i64` continua recusado |
| `2950` | shader: lower v_readlane_b32 and v_writelane_b32 in the direct C++ compute path | `641add96` | +275/-23. `v_readlane` usa `waveReadLaneAt` e troca metades na onda 64 dividida; `v_writelane` grava só na lane escolhida, sem EXEC. Lane constante ou SGPR, mascarada à largura da onda |
| `2946` | shader: vote a quad-complete EXEC for WQM EXECZ branches in native pixel draws | `ea07313b` | +487/-8. `PixelWqm::raster_vote` reduz `s_cbranch_execz` depois de `s_wqm exec` a `WaveActiveAnyTrue` do EXEC modelado, depois de OR com `QuadReadAcrossX/Y/Diagonal`; só a forma limitada é aceita. Fecha #2907 |
| `2942` | ci: check native DXIL generation on cpp-migration pull requests | `c2f6d3de` | +11/-1. O filtro de PR do workflow do Windows não incluía `cpp-migration`, então o frontend nativo podia entrar nessa branch **sem rodar o workflow**; habilita os jobs existentes para essa base e acrescenta um passo que compila `compile_dxil` e roda os controles de container DXIL com `--no-tests=error`. O evento hospedado não foi exercitado |
| `2941` | shader: restore compilation of LDS replay requests | `a990f323` | +82/-15. `compile_lds_request.py` quebrava com `TypeError: load_compute_request() got an unexpected keyword argument 'pipeline_type'`; o loader passa a construir o pipeline pedido **depois** de validar a entrada. Atômicos de LDS validam operandos antes de gravar, inclusive com endereço e destino iguais |
| `2939` | compile_dxil: read shader requests at revision 28 | `7beb331d` | +4/-4 em 3 arquivos. `read_graphics_request`/`read_compute_request` hasheavam com `-14`/`-14cs` enquanto runtime e `gfx2hlsl.py` usam `-28`/`-28cs`, então `compile_dxil` recusava **todo** pedido que o runtime atual escrevia ("request key mismatch or missing root"). Duas linhas de código e duas de teste |
| `2937` | shader: lower DPP16 quad_perm in the direct C++ compute path | `1cd48bd0` | +210/-9. `quad_perm` (padrões 0–255) deixa de ser exclusivo de gráficos no decodificador DPP16; `mirror()` vira `lower_dpp()` e lê a lane `(lane & ~3) \| ((pattern >> 2*(lane & 3)) & 3)` com `waveReadLaneAt`. Padrões > 255 sem espelhamento de linha continuam recusados |
| `2868` | gpu: explicitly clear the pixel-history fixture target | `20a497c2` (main) | +6/-12, só teste; igual ao commit |

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum defeito do catálogo é fechado por esta janela.
  - **D-01** (superfície apresentada liberada durante a cópia da assinatura, `2a898640`): o `#2968`
    aberto trabalha no **diagnóstico** de readback da superfície apresentada, com retenção até a fence;
    não é a cópia da assinatura. **Não reabre nem corrige.**
  - **D-18** (compilação não retomada depois de rodada manifest-only, `b87a99a3`): os merges `2939`,
    `2941` e `2955` tratam de **chave de pedido** e de publicação de cache, não da retomada manifest-only.
    **Não corrige.**
  - **D-19** (documentação do ciclo de `.fail` obsoleta, introduzido `b87a99a3`, corrigido `5b29fe0f`):
    o `#2984` aberto reescreve o ciclo de vida do `.fail` ao acrescentar as reservas de `<stem>.fail` no
    modo lote. É documentação nova sobre o mesmo mecanismo. **Não corrige nem contradiz.**
  - **D-27** (coordenadas `a16` 2D de `image_sample` lidas como dois binary32, `c3c3ab3c`): `#2960`
    aberto trata da dimensão **cúbica**. **Não corrige.**
- **Abre defeito novo?**
  - `c2f6d3de` (#2942) é a evidência mais forte, e é de processo: o filtro de PR do workflow não incluía
    `cpp-migration`, e 16 dos 18 PRs desta janela entraram nessa branch. Não verifiquei se algum deles
    rodou CI por outro gatilho; o que o commit afirma é que "updates to the native frontend could merge
    into that branch without running the workflow".
  - `7beb331d` (#2939) e `a990f323` (#2941) consertam regressões que já estavam na `cpp-migration`:
    chave de pedido em revisão 14 contra runtime em 28, e um `TypeError` que impedia compilar qualquer
    pedido de replay de LDS. Nenhuma das duas está no catálogo — são defeitos introduzidos **dentro da
    branch**, e o efeito é que a `cpp-migration` esteve quebrada para `compile_dxil` até `7beb331d`.
  - `20a497c2` registra um defeito de teste que existia desde a criação do fixture: o valor de clear
    otimizado de `CreateCommittedResource` não inicializa o conteúdo, e o fixture assumia preto.
  - `#2954` (aberto) documenta defeito não catalogado: o compilador da NVIDIA devolve `E_OUTOFMEMORY`
    ao criar o PSO do programa `shared_atomics` quando a negação chega como `InterlockedAdd` sobre
    `groupshared` — arquivo `tools/compute_store_aot.py`, emissão de `ds_sub_u32`.

## Confiança

Verificado: os 2 SHAs e metadados em `repos/R0Y00/Xbox2Windows/commits?sha=main&per_page=100`, com o
cursor `6ee878d8` na posição 3; `eff2e27a` e `20a497c2` lidos em `commits/<sha>` com patch completo
(são pequenos); os 18 PRs mesclados em `pulls?state=merged&search=merged:>=2026-11-26T21:59:23Z`, cada um
aberto em `prs view` com `baseRefName`, `mergeCommit`, arquivos e corpo; os 45 PRs abertos, todos com
número acima de 2887.

**Divergência registrada (obrigatória pelo procedimento):** a contagem de PRs mesclados (18) **não** bate
com a de commits novos na `main` (2). A causa está verificada, não inferida: 16 dos 18 PRs têm
`baseRefName = cpp-migration`; só `#2982` e `#2868` têm `baseRefName = main`, e são exatamente os dois
commits da faixa `6ee878d8..20a497c2`. Os `merge_commit_sha` dos 16 (`7bcff286`, `05438d65`, `2b678fd3`,
`78a071ca`, `0b1456ce`, `8946fb16`, `f934df52`, `b260984f`, `e3670100`, `49e0e36d`, `641add96`,
`ea07313b`, `c2f6d3de`, `a990f323`, `7beb331d`, `1cd48bd0`) não são alcançáveis a partir da `main` —
`compare/7bcff286...main` devolve `diverged`, 2 à frente e 22 atrás.

**Consequência para o estado:** `last_commit` avança para `20a497c2` (topo da `main`), mas os 18 commits
da `cpp-migration` não são rastreados por `state/watch.json`, porque o repositório só tem `main` como
`default_branch`. Se a `cpp-migration` for mesclada na `main`, eles chegarão de uma vez e serão
documentados então — não ficam perdidos, mas também não são vistos hora a hora.

Não verificado: o conteúdo dos 16 commits da `cpp-migration` além do que os corpos dos PRs e as listas de
arquivos descrevem — não abri `commits/<sha>` de cada merge porque o diff é o do PR, já coberto por
`files` e `additions/deletions`; não executei nenhum teste nem build; não verifiquei se os 16 PRs rodaram
CI antes do `#2942`; não verifiquei as execuções em hardware citadas nos corpos (RX 7900 XT, RTX 5080,
RTX 4060 Ti). Nenhum teste foi executado aqui; os 13 repositórios monitorados são somente leitura.
