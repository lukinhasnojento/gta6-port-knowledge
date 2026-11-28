# R0Y00/Xbox2Windows — atualização de 2026-11-28 12:00

## Resumo

121 commits e 106 merges desde o cursor `886a5a28`; 40 documentados (teto), 35 deles squash de PR na
`main` e 2 em `codex/native-shader-modules`. A hora é a maior leva de *lowerings* do caminho DXIL
direto até agora — 27 dos 40 commits tocam `src/gfx1013_direct_compute.inl`, `gfx1013_compute_native.cpp`
ou o manifesto de runtime —, com três consertos de runtime de verdade (`4b0350ea` ring de upload,
`fad97f47` leitura guardada, `2e12dd8e` retângulo girado) e o começo de um frontend nativo grande que
entrou como merge de branch (`fd41d5a7`). Nada foi compilado nem executado.

## Commits novos (40)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `ef4ee2e8` | pigalot | startup: read single-mip 0x105 volumes in slot 70 | `gfx10_layout.h`: novo `supports_volume()`, que aceita TEXTURE3D de modo de swizzle 5 (0x105) até 2.048³, `MipLevels == 1`, elemento não comprimido; `Layout` ganha `depth`, `block_depth` e `z_bits` (o deslocamento em bloco contribuído por z). `d3d12_draw.inl` confere `Footprint.Depth == console_layout->layers(i)` e passa o passo de camada ao `detile`; `d3d12_memory_srv.inl` conta **uma** fatia para volume (as footprints já cobrem as camadas) em vez de multiplicar por `DepthOrArraySize`. |
| `eb36ad01` | pigalot | startup: read 0x105 volume mip chains in slot 70 | Tira o `MipLevels == 1` do volume e amplia `address_texture.h` para considerar a profundidade no teste de "maior lado permite N mips". `Layout` perde `blocks_high` e a documentação passa a dizer que `layer_size` cobre a camada de cada mip. |
| `5ac7ed06` | pigalot | startup: read block-compressed 0x105 volumes in slot 70 | `supports_volume` deixa de exigir `element.texels == 1`: elemento comprimido é um bloco 4×4 de uma camada. Comentário e contas de mip passam a arredondar para blocos 4×4 inteiros antes do tamanho. |
| `bd78c395` | Ammmae0x | shader: harden compile_lds_request CLI, path guards, and replay checks with offline unit suite | `tools/startup/compile_lds_request.py` ganha `_validate_regular_file`/`_validate_writable_dir` (recusa diretório, symlink, caminho não regular) e `main()` passa a capturar `OSError/ValueError/Unsupported` com código de saída; 593 linhas de suíte CLI offline e alvo CMake novo. Sem mudança no tradutor. |
| `698a057f` | Leyukaka | compute: accept DPP16 on binary16 VOP2 add, sub, subrev and mul (#3188) | Remove a recusa `"DPP form"` para binary16 VOP2 e passa a aceitar máscara de linha/banco parcial (`dpp_row_mask`, `dpp_bank_mask` lidos de `control >> 28` e `>> 24`); a fonte permutada é mascarada com `and 65535` antes da aritmética de meia precisão, e `alu_rules` deixa de ser exigido para nomes binary16. |
| `e582a9a4` | Ammmae0x | cpu: add dedicated cpu_amd_monitor_rdpru_aot lifter for MONITORX, MWAITX, and RDPRU (#2430) | 809 linhas novas em `tools/cpu_amd_monitor_rdpru_aot.py`: traduz RDPRU (ECX ∈ {0,1}, ambos viram leitura serializada de RDTSC — **não** devolve MPERF/APERF distintos), MONITORX (arma monitor com ECX=EDX=0) e MWAITX (`_mm_pause()` cercado, sem timer mode); CMake gera fixtures original/altered e mutantes. |
| `f962458f` | Leyukaka | shader: record a diagnostic draw while translation is missing (#3218) | `d3d12_draw.inl`: `draw_missing_translation()`, atrás de `X2W_DRAW_MISSING_SHADER=1`, desenha um triângulo de diagnóstico quando não há tradução — só com lista DIRECT, topologia TRIANGLELIST, alvo único e viewport/scissor definidos —, com PSO em cache por chave FNV (device, root, DSV, formatos de alvo). `list_draw` deixa de contar `translated/untranslated` nesse caminho. |
| `22f38526` | RonLogic | shader: lower typed 2D image atomics in direct compute (#3162) | `ComputeViewKind::uint_texture_uav` novo; `is_image_atomic`/`image_atomic` tratam op 16 (compare-exchange), 17 e 23 com máscara 1 ou 3, coordenada em `i.b & 255` e recusa explícita do resto. O manifesto ganha a linha `uav N tex2d uint table P O channels 1`, e o runtime a lê com `atomic_uint_texture` e dimensão TEXTURE2D. |
| `f6206217` | tedin7 | build: startup_runtime target and runtime preset for a title run (#2827) | Só construção: alvo `startup_runtime`, preset novo em `CMakePresets.json`, documentação e um teste Python que confere o alvo. |
| `b66cdce6` | lambirou | tooling: report the startup exit code and last presented frame (#3176) | `tools/startup/exit_report.py` (255 linhas) lê os logs do `debug_launch.py`/`run.ps1` e imprime código de saída, um de seis rótulos de estado final, o último contador de present e a superfície; afirma não copiar caminhos, módulos nem endereços. |
| `06de199e` | lambirou | tooling: add a startup UI regression matrix (#3173) | `tools/startup/ui_regression_matrix.py` (435 linhas) roda fixtures CTest de UI já registrados e escreve matriz Markdown/JSON; a seleção padrão cobre os consertos de sampler/BC7 de `#3153`, os caminhos `MipLevels==0`/`CopyTextureRegion` de `#3141` e o retângulo indexado de `#3142`. Nada é executado por este commit. |
| `790c69eb` | tedin7 | startup: slot 70 structured buffer views at sub-allocated addresses take an exact-start upload copy (#2841) | Novo `unplaced_view_needs_exact_start()`: uma view estruturada/ tipada cujo endereço **não** é múltiplo do elemento a partir da base de página de 64 KiB recebe cópia que começa no endereço; view raw nunca (mantém a base para o teste de alinhamento de 16 B). `WrapperOwn` ganha `unplaced_srv_backing` e `exact_copy`, e `existing_memory_buffer` recusa reutilizar backing sintético. |
| `d6912f58` | legacy-code-archaeologist | shader: lower v_fma_f16, v_fmac_f16, v_fmamk_f16 and v_fmaak_f16 in direct compute (#3187) | `lower` novo para as quatro formas: recusa DPP, SDWA, omod, `op_sel` de metade alta e o terceiro operando modificado de `v_fmac_f16`; emite a soma de três produtos em f16 empacotado. |
| `44c910c8` | lambirou | runtime: add opt-in input trace for the virtual gamepad (#3177) | `gameinput.cpp`: `X2W_INPUT_TRACE=1` emite uma linha `x2w-input-trace` a cada mudança do estado resolvido (depois de teclado, XInput, bloco compartilhado, `X2W_INPUT_PRESS` e script) e no máximo uma a cada 250 ms enquanto o estado não muda. |
| `c1de64a5` | Leyukaka | shader: lower v_permlane16_b32 and v_permlanex16_b32 in direct compute (#3022) | Decodificador recusa S0 literal em VOP3 op 0x377/0x378; `lower_permlane` exige contagem de threads divisível por 16 (ou 32 para `x16`), lê FI/BOUND_CTRL de `op_sel` e trata D e S0 como VGPR, S1/S2 como SGPR/constante. |
| `0b17c524` | xiaoqi72665 | runtime: bind raw pixel buffer UAVs for native graphics draws (#3215) | `native_buffer_store_view.h` passa a exigir `FirstElement % (RAW_UAV_SRV_BYTE_ALIGNMENT/4) == 0`; o manifesto gráfico aceita `uav N raw table P O` quando é **uma** UAV e não há SRV nem binding de vértice; `PixelUav` ganha `raw`. Primeira fatia: só pixel, sem aliases. |
| `e9d1d6a0` | xiaoqi72665 | shader: support defined numeric M0 in direct compute (#3205) | M0 (registrador 124) entra na análise como símbolo numérico separado (`initial[124] = unbound`) e é aceito como operando numérico de ALU — **exceto** em MUBUF/SMEM, onde continua implícito; `s_mov_b32` passa a aceitar destino 124. |
| `14d56502` | tedin7 | startup: warn when the shader cache and the translator come from different revisions (#2820) | Cria `namespace shader_cache` com três revisões literais (`x2w-startup-shader-29`, `-29cs`, `mesh-57-3`), grava `<cache>/revision.txt` uma vez por processo antes do primeiro pedido, e `tools/startup/shader_cache_revision.py` compara o arquivo com os literais do tradutor, avisa e sai 1 em divergência; `-FreshShaderCache` move a cache cheia para `*.old-<data>-<hora>`. |
| `4b0350ea` | tedin7 | startup: count a failed buffer copy as the null cause ring_full and refuse the draw; report upload ring statistics (#2864) | Separa "memória do guest ilegível" de "ring de upload não deu espaço": quando o intervalo é legível e `reserve_upload_ring` devolve nulo, a causa é a nova `NullCause::ring_full` e o draw é **recusado** em vez de ligar zeros. Contadores novos (`upload_ring_reserved_bytes`, `capacity`, `reservations`, `refusals`) entram no rodapé de totais. |
| `2e12dd8e` | lambirou | startup: identify a rotated rectangle corner by its diagonal (#3142) | `gfx1013_rect_geometry.cpp` e `indexed_rect_geometry.h` ganham `corner_off_longest`/`corner_between` estendido: o canto reto é o vértice oposto à **diagonal** (a maior de três arestas) e os lados precisam ser perpendiculares. Sem isso o retângulo girado era descartado inteiro e a região ficava sem pintura (`#3131`). Cantos por eixo mantêm precedência. |
| `9345c21f` | xiaoqi72665 | shader: lower native pixel raw buffer atomic add (#3217) | `buffer_atomic` ganha predicado de pixel opcional e é aceito dentro de região mascarada quando é `buffer_atomic_add` sem GLC e no estágio de pixel (não vértice). |
| `fad97f47` | tedin7 | startup: refuse unmapped source ranges before a probe or a copy (#2846) | `copy_title_bytes` e `guarded_copy` deixam de usar `__try/__except` local e passam a chamar `x2w_local::guarded_memcpy(...)`, que consulta o intervalo antes de copiar; o rodapé de totais ganha `reads refused before a fault`. |
| `05d18732` | xiaoqi72665 | test: trace GPU-written indirect Dispatch to offscreen pixels (#3222) | Suíte nova (fixture `startup_indirect_dispatch_trace`) que liga Dispatch indireto escrito pela GPU a pixels fora da tela. **Só teste.** |
| `58e3183a` | lambirou | runtime: count decoded Opus completions and desktop audio submissions (#3178) | `audio_sink.h/cpp` ganham `accepted` e `not_open`; arquivo novo `audio_telemetry.h` com contadores que saturam em `UINT64_MAX` (e o aviso de que a leitura não é um instantâneo coerente); `xapu.cpp` conta decodificações Opus. |
| `b7280e02` | RonLogic | runtime: size Compute texture views for the desktop SDK (#3223) | `ViewRecord` define `srv_desc_bytes`/`uav_desc_bytes = 40` e o `desc` passa de `unsigned char[40]` para uma união de `D3D12_SHADER_RESOURCE_VIEW_DESC`/`D3D12_UNORDERED_ACCESS_VIEW_DESC`; `view_slot` usa as constantes. Corrige o recorte do descritor quando o SDK de desktop define structs maiores. |
| `ad306425` | pigalot | startup: read linear (0x100) textures in slot 70 (#2926) | `gfx10_layout.h` passa a tratar o modo linear 0x100 (pitch de 256 B do addrlib) e `address_texture.h` deixa de recusá-lo; antes "a extensão dessas linhas não estava estabelecida". |
| `c7215add` | pigalot | startup: read one-dimensional linear (0x100) textures in slot 70 (#2929) | No modo 0x100, TEXTURE1D não-array também é aceita (uma linha por mip, sem formato comprimido); `desktop_address_view_supported` passa a exigir `D3D12_FORMAT_SUPPORT1_TEXTURE1D` para dimensão 1D. |
| `7fcde81f` | pigalot | startup: bind cube views of console-layout textures in slot 70 (#2938) | `kind 5` (cube) no `Translation::Srv`, `address_view_fits` aceita TEXTURECUBE e o suporte de formato passa a exigir `D3D12_FORMAT_SUPPORT1_TEXTURECUBE`. |
| `b1e92256` | pigalot | startup: read 0x105 volume mip chains in slot 70 (#2876) | Reaplica/duplica o conserto de `eb36ad01` na árvore (`address_texture.h` + `gfx10_layout.h`) junto com a restrição 1D não-array; o merge `fd41d5a7` mais tarde reconcilia as duas árvores. |
| `fd41d5a7` | tedin7 | merge: bring main into slot70/console-3d-bc | Merge de dois pais: traz a `main` (com todo o frontend DXIL nativo) para a branch de slot 70. Entram 26 arquivos novos de `src/` (`direct_compute_cache.cpp` 925 linhas, `dxil_module.h`, `gfx1013_isa*`, `gfx1013_pixel*`, `guest_shader_container`, `native_compute_pipeline`, `shader_dxil_compiler`, `native_buffer_store_view.h`), além de reescrever `d3d12_draw.inl` (+563/−113) e `d3d12_compute_dispatch.inl` (+253/−46). Não é squash e não representa trabalho novo da hora: é a integração das duas linhas. |
| `f75ad02c` | R0Y00 | docs: link to experiments and coverage from the README (#818) | Só README: links para experimentos e cobertura. |
| `38129ba3` | lambirou | tests: allow the shader cache revision stamp in two exact-set checks (#3231) | Dois testes de conjunto exato passam a ignorar `revision.txt`, criado por `14d56502`. Só teste. |
| `350785e8` | RonLogic | compute: derive descriptor stride SCC from bound buffer metadata (#3227) | `NativeComputeShader` ganha `stride_srvs` (palavras extras de `b0` lidas de SRVs ligados); `descriptor_strides` indexa por (SRV, palavra) no lowering; a pipeline liga as SRVs de stride como constantes de root e o manifesto/`encode` serializam a lista, com recusa se a SRV declarada não estiver ligada. |
| `86beae95` | legacy-code-archaeologist | shader: accept inline constants in packed binary16 arithmetic and min/max in direct DXIL (#3230) | `packed_f16_inline_constant(240..248)` devolve o valor de meia precisão embutido (bits [15:0], alto zerado) e as operações empacotadas passam a usar a constante em vez de recusar. |
| `7364e6a9` | Leyukaka | shader: read the wave size of slot 67 compute requests from .sa | `ComputeRequest` ganha `wave32`, lido do bit 20 do dword 4 do state block A (`sa[18] & 0x10`); o lowering escolhe wave64 quando `direct_dxil && !request.wave32 && threads > 32`, com a ressalva de que o modelo não escreve a palavra alta zerada; `native_compute_output_revision` novo invalida a saída. |
| `8d9897fe` | Not-Saii | runtime: name each refused condition of a compute output buffer (#3229) | Cada motivo de recusa de output de compute (token de view, tamanho zero, não múltiplo de 4, > 16 MiB, range não colocado) passa a ter nome próprio no relatório, em vez de um único "não colocado". |
| `57e94668` | Leyukaka | compute: lower op_sel on VOP3 v_fma_f16 (#3234) | `op_sel` de VOP3 em `v_fma_f16` deixa de ser recusado (só quando há clamp ou outra instrução); cada fonte com o bit ligado é lida da metade alta, exceto registradores < 256 tratados pela regra normal. |
| `e3633742` | Leyukaka | compute: clamp SDWA widening half conversions (#3203) | Aceita `clamp` em `v_cvt_f32_f16` SDWA (conversão de alargamento) e o decodificador deixa de recusá-lo; `v_cvt_f16_f32` (estreitamento) continua recusando clamp fora de SDWA. |
| `8007a539` | Leyukaka | compute: lower DPP8 sources in direct compute (#3236) | `Inst` ganha `dpp8_selectors`; o grupo DPP passa a ser 8 quando há seletores DPP8, com recusa própria para contagem de threads não divisível por oito e tratamento dos oito seletores de 3 bits. |
| `50e4f763` | Leyukaka | compute: accept SGPR and inline-constant sources in binary16 SDWA (#3240) | Remove a recusa "SDWA binary16 source must be a VGPR" (bit 7), deixa a seleção de sub-palavra operar sobre a palavra inteira do operando e usa `packed_f16_inline_constant` quando a fonte é constante embutida. |

## Pull requests abertos (39)

Todos criados depois das 22:00Z de 27/11 (o primeiro às 23:01Z, o último às 12:12Z de 28/11). Lidos por
título e corpo; **nenhum patch de PR aberto foi lido**.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #3352 | compute: translate divergent multi-wave wave64 groups as lane pairs | Etapa 2 de `#3237`: wave64 do guest como dois wave32 do host trocando palavras de máscara em barreira de grupo. | Não é `D-xx`. Área de D-05 (máscaras por braço). |
| #3351 | runtime: bind cube texture SRVs for published native compute | Segue `#3306`: cube declarado como `TextureCubeArray` (kind 9, camada = face >> 3). | Não. |
| #3350 | runtime: preserve CPU fence signal rewinds (replacement for #3279) | Serializa por fence o `Signal` nativo, a observação de conclusão e a publicação da palavra em cache (SRW lock por fence). | Não. |
| #3349 | shader: replace #3225 image descriptor work | Substitui `#3225`: lowering dos campos de tamanho do descritor de imagem no estágio de pixel, com correções de proveniência e `s_lshl`. | Não. |
| #3348 | [WIP] shader: offline native compute cache follow-up | Rascunho substituto de `#1994`, declarado incompleto. | Não. |
| #3347 | gpu: slot 133 texture replacement for #2773 | Revisão substituta de `#2773`, sincronizada com `24fcd0b5`; o merge expôs divergência só de fixture. | Não. |
| #3346 | bcpack: BCPack refusal inspector review follow-up | Revisão alternativa de `#1940` com as correções de review. | Não. |
| #3345 | tooling: test remaining GDK export regressions | Só teste; segue `#3284` (parar em `Summary` e o resto do parser). | Não. |
| #3344 | shader: formatted buffer byte-window review follow-up | Substituto de `#3232`. | Não. |
| #3343 | d3d12: compute dispatch root-binding review follow-up | Substituto de `#2491`. | Não. |
| #3342 | startup: output-window close review follow-up | Substituto de `#2344`. | Não. |
| #3341 | runtime: validate placed acceleration-structure updates | Nomeia causas de recusa de BLAS colocado e resolve `PERFORM_UPDATE` com estrutura de origem; grava proveniência de `ALLOW_UPDATE`. | Não. |
| #3340 | runtime: harden atomic report writes and output path guards in ui_regression_matrix with CLI suite | Valida `--build-dir`/`--output`/`--json` (sem symlink, sem diretório), escrita atômica do relatório. | Não. |
| #3339 | shader: admit remaining lowered VOP1/VOP2 SDWA profiles | Admite nove perfis SDWA inteiros/bit e três de ponto flutuante já baixados, mantendo a recusa no estágio gráfico; fecha `#3303`. | Não. |
| #3338 | tooling: add bad-pointer fault-equivalence area (fault_reads) to fuzz_cpu | Área determinística `fault_reads` no fuzz de CPU. | Não. |
| #3337 | shader: harden path, span, and collision guards in compile_sampled_pixel_request | Extrai `main(argv)` e trata `OSError/ValueError/Unsupported/CalledProcessError` com suíte CLI. | Não. |
| #3336 | compute: lower 3D and NSA typed image atomics | Parte de `#3224`: atômico tipado 3D e forma NSA na mesma `image_atomic` de `#3162`. | Não. |
| #3333 | compute: lower root-UAV global DWORD stores | Parte de `#3224`: a recusa de store global DWORD com par escalar tinha 9 shaders / 19.528 dispatches no censo do Not-Saii em `8007a539`. | Não. |
| #3332 | startup: let run.ps1 and run.sh turn off the hook thread holding | `-SuspendOthers 0` / `--suspend-others 0`; fecha `#3331`. O padrão continua 1. | Não. |
| #3330 | compute: lower sub-dword LDS stores and loads | Parte de `#3224`. | Não. |
| #3329 | compute: copy descriptor words through VCC as a scalar temporary | Parte de `#3224`: programas que guardam palavras de descritor em VCC e movem de volta. | Não. |
| #3328 | shader: lower v_div_fmas_f32 | Fecha `#3327`; lê o bit de VCC por lane. | Não. |
| #3326 | shader: lower v_div_fixup_f16 | Fecha `#3325`; predicados exatos em inteiro de 32 bits. | Não. |
| #3324 | shader: lower v_div_fixup_f32 | Fecha `#3323`. | Não. |
| #3321 | gpu: trace descriptor history and GPU-written Mesh arguments | Refere `#3310`: histórico de descritor e argumentos de Mesh escritos pela GPU, para o caso de contagem indireta zero. | Não. |
| #3320 | compute: lower ds_bpermute_b32 without an LDS allocation | Parte de `#3224`: usa a barra de LDS sem ler nem escrever LDS, então não exige alocação. | Não. |
| #3319 | compute: lower 2D image_gather4_lz_o | Parte de `#3224`: gather em LOD 0 com deslocamento por lane em 6 bits de x e y. | Não. |
| #3317 | shader: lower v_cvt_pknorm_i16_f16 and v_cvt_pknorm_u16_f16 | Fecha `#3316`; arredondamento inteiro exato. | Não. |
| #3315 | shader: pin the per-lane heap index at its waterfall mark | Fecha `#3304`: o índice SGPR era lido no `s_load` e não na marca do waterfall, entregando o índice trocado quando o SGPR mudava entre os dois. | Não, mas é a classe dos lowerings de índice por lane de `#2861`. |
| #3311 | 'Fix compiler warnings in tests' | Corrige erros de compilação em `startup_gameinput_trace.cpp` e `startup_rect_geometry.cpp`. | Não. |
| #3305 | runtime: publish narrow half-float texture UAV stores | Parte de `#3224`/`#2935`: stores D16 de 1 e 2 canais param em `half_scalar_uav`/`half2_texture_uav`. | Não. |
| #3300 | shader: lower v_sin_f16 and v_cos_f16 | VOP1 0x60/0x61, VOP3 0x1e0/0x1e1 e SDWA, por `trig_turns()`. | Não. |
| #3279 | runtime: preserve CPU fence signal rewinds | Versão anterior do `#3350`: `maximum` atômico publicava sinal mesmo depois de falha do desktop. | Não. |
| #3266 | shader: lower v_min3_f16, v_max3_f16, and v_med3_f16 | Fecha `#3264`. | Não. |
| #3248 | Integration stack: native PRs on current main | Pilha de integração em rascunho, declarada **não** candidata a merge. | Não. |
| #3241 | shader: merge current main into the native shader module stack | PR só de merge para `codex/native-shader-modules`, base `71e23baf`. | Não. |
| #3232 | Carry formatted buffer loads through byte-address windows to main | Leva o `#3034` de `cpp-migration` para `main` (janela de bytes). | Não. |
| #3225 | shader: lower pixel image descriptor size fields read as numbers | Lê largura/altura/profundidade do descritor de 8 dwords no pixel; `#3349` é o substituto. | Não. |
| #3221 | native: bind a single directly indexed heap flag as a descriptor table | Parte de `#870`: root signature com só uma das flags de heap direto deixa de ser recusada. | Não. |

## Pull requests mesclados (35)

Os outros 71 merges da janela foram para branches que não são a default (`codex/native-shader-modules`
2, o resto em `main` mas com o commit correspondente **fora** dos 40 documentados — o teto de commits
cortou a faixa). Esta tabela cobre os merges cujo commit está entre os 40.

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #3188 | compute: accept DPP16 on binary16 VOP2 add, sub, subrev and mul | `698a057f` | Máscara parcial de linha/banco; confere com o título. |
| #2430 | cpu: add dedicated cpu_amd_monitor_rdpru_aot lifter | `e582a9a4` | Lifter + fixtures + mutantes; RDPRU vira RDTSC, declarado no comentário. |
| #3218 | shader: record a diagnostic draw while translation is missing | `f962458f` | Desenho de diagnóstico opt-in, não substituição do caminho normal. |
| #3162 | shader: lower typed 2D image atomics in direct Compute DXIL | `22f38526` | `uint_texture_uav` + linha de manifesto + leitura no runtime. |
| #2827 | build: startup_runtime target and runtime preset | `f6206217` | Só construção. |
| #3176 | tooling: report the startup exit code and last presented frame | `b66cdce6` | Script de relatório; sem mudança de runtime. |
| #3173 | tooling: add a startup UI regression matrix | `06de199e` | Script de matriz; os testes que ele seleciona não são alterados. |
| #2841 | startup: slot 70 structured buffer views at sub-allocated addresses take an exact-start upload copy | `790c69eb` | Cópia com início exato e recusa de reuso de backing sintético. |
| #3187 | shader: lower v_fma_f16, v_fmac_f16, v_fmamk_f16 and v_fmaak_f16 in the direct C++ path | `d6912f58` | As quatro formas, com as recusas declaradas. |
| #3177 | runtime: add opt-in input trace for the virtual gamepad | `44c910c8` | Traço opt-in, 250 ms de intervalo. |
| #3022 | shader: lower v_permlane16_b32 and v_permlanex16_b32 in direct compute | `c1de64a5` | Exige grupo divisível por 16/32; S0 não pode ser literal. |
| #3215 | runtime: bind raw pixel buffer UAVs for native graphics draws | `0b17c524` | Só a primeira fatia: uma UAV raw, sem SRV, no estágio de pixel. |
| #3205 | shader: support defined numeric M0 in direct compute | `e9d1d6a0` | M0 numérico, menos em MUBUF/SMEM. |
| #2820 | startup: warn when the shader cache and the translator come from different revisions | `14d56502` | `revision.txt` + ferramenta de aviso; **não** invalida a cache. |
| #2864 | startup: count a failed buffer copy as ring_full and refuse the draw | `4b0350ea` | Recusa o draw em vez de ligar zeros; contadores do ring. |
| #3142 | startup: identify a rotated rectangle corner by its diagonal | `2e12dd8e` | Canto oposto à diagonal, com perpendicularidade exigida. |
| #3217 | shader: lower native pixel raw buffer atomic add | `9345c21f` | Atômico dentro de região mascarada, só pixel e sem GLC. |
| #2846 | startup: refuse unmapped source ranges before a probe or a copy | `fad97f47` | `guarded_memcpy` antes da cópia; contador de leituras recusadas. |
| #3222 | test: trace GPU-written indirect Dispatch to offscreen pixels | `05d18732` | Só teste. |
| #3178 | runtime: count decoded Opus completions and desktop audio submissions | `58e3183a` | Contadores `accepted`/`not_open` e telemetria de Opus. |
| #3223 | runtime: preserve Compute texture views with Agility SDK headers | `b7280e02` | Uniões de descritor e tamanhos 40 B. |
| #2926 | startup: read linear (0x100) textures in slot 70 | `ad306425` | Modo linear com pitch de 256 B. |
| #2929 | startup: read one-dimensional linear (0x100) textures in slot 70 | `c7215add` | 1D não-array no modo linear. |
| #2938 | startup: bind cube views of console-layout textures in slot 70 | `7fcde81f` | Kind 5 e exigência de suporte a cube. |
| #2876 | startup: read 0x105 volume mip chains in slot 70 | `b1e92256` | Mesmo conserto de `eb36ad01`, aplicado na branch. |
| #818 | docs: clarify README status and contribution links | `f75ad02c` | README. |
| #3231 | tests: ignore the shader cache revision stamp in two exact-set checks | `38129ba3` | Só teste. |
| #3227 | Compute descriptor stride SCC from bound buffer metadata | `350785e8` | `stride_srvs` no manifesto, pipeline e `encode`; recusa se a SRV não estiver ligada. |
| #3230 | shader: accept inline constants in packed binary16 arithmetic and min/max in direct DXIL | `86beae95` | Constantes embutidas 240–248 em meia precisão. |
| #3006 | shader: read the wave size of slot 67 compute requests from .sa | `7364e6a9` | `wave32` do bit 20 de `sa[18]`; revisão de saída nova. |
| #3229 | runtime: name each refused condition of a compute output buffer | `8d9897fe` | Motivos nomeados; `#3224` e `#3229` citados no corpo. |
| #3234 | compute: lower op_sel on VOP3 v_fma_f16 in direct compute | `57e94668` | `op_sel` aceito só em `v_fma_f16` VOP3 sem clamp. |
| #3203 | compute: accept clamp on SDWA v_cvt_f32_f16 | `e3633742` | Clamp aceito só no alargamento. |
| #3236 | compute: lower DPP8 sources in the direct compute path | `8007a539` | Grupo de 8 e oito seletores. |
| #3240 | compute: accept SGPR and inline-constant sources in binary16 SDWA | `50e4f763` | Remove a exigência de VGPR; usa constante embutida. |

## Cruzamento com o catálogo

- **Nenhum dos 40 hashes está no catálogo** (colunas `introduzido`/`corrigido`, 262 hashes distintos).
  Nenhum PR mesclado ou aberto cita `D-xx`.
- **Hipótese, não provada — D-21** ("checagem *device-free* das seleções SDWA `movrel` não detectava
  seleção removida", `bc03f191`/`2e5fdca4`): `50e4f763` (#3240) remove a recusa de fonte não-VGPR em
  SDWA binary16 e `8007a539` (#3236) acrescenta DPP8. É a mesma área; o diff não toca o teste
  citado por D-21 e não há como afirmar extensão do defeito.
- **Hipótese, não provada — D-27** ("coordenadas 2D `a16` lidas como dois registradores binary32",
  corrigido em `c3c3ab3c`): `22f38526` (#3162) interpreta a coordenada de atômico de imagem como
  `i.b & 255` e recusa o resto. Não é a mesma instrução (`image_atomic` vs `image_sample`) e o commit
  não menciona `a16`.
- **Hipótese, não provada — D-05** ("loads pendentes não unidos nas fronteiras dos braços mascarados"):
  `8007a539` estende o que pode estar dentro de uma região mascarada (`buffer_atomic_add`, DPP8
  grupo de 8). O diff não altera a união de loads.
- **Candidato a defeito novo, sem prova em jogo:** `7364e6a9` (#3006) escolhe o modelo wave64 por
  `threads > 32` quando o state block diz wave32 **falso**, e o comentário do próprio commit admite
  que esse modelo "não escreve a palavra alta zerada que um compare ou carry-out wave64 guarda num
  par de SGPR". Um guest wave64 com 32 threads ou menos (o caso que o commit mantém em wave32) e um
  compare de 64 bits que grava par de SGPR ficaria com a metade alta indefinida. Não há caso de
  título no repositório; fica como observação, não como linha do catálogo.
- Nenhum defeito do catálogo é fechado por esta janela.

## Confiança

- **Verificado por API:** 40 commits (sha, data, autor), 35 merges com `merge_commit_sha` conferido
  contra o sha documentado, 39 PRs abertos (título e corpo), e o patch de cada um dos 40 commits —
  `files[]` com nome, status, adições e remoções lidos, e o `patch` lido nos arquivos de código.
- **NÃO VERIFICADO:** nada foi compilado nem executado; nenhum teste do repositório foi rodado. As
  afirmações de "fecha a recusa" descrevem o que o lowering passou a aceitar, não que o shader real
  do título traduza. Os números de impacto citados em corpos de PR (`19.528 dispatches`, "9 shaders")
  são declaração do autor, não medição desta rotina. Os 2 merges em `codex/native-shader-modules`
  (`#3138`, `#3308`) foram contados mas **não** entram no cursor de commits da default.
- **Teto de 40 atingido:** 40 de 121. `last_commit` fica em `886a5a28` (inalterado) e o estado grava
  `commit_cursor = 50e4f763` (o 40º documentado) e `head_at_run = 052abe6f`. `compare` a partir de
  `50e4f763` devolve **86** commits, contra 81 restantes da janela original: a diferença de 5 vem de
  commits que são ancestrais do 40º (o último documentado é um squash cuja linhagem inclui parte dos
  anteriores). O próximo ciclo começa em `commit_cursor`; a diferença está registrada aqui para que
  ela não passe em silêncio.
- **Divergência de contagem commits × merges:** 40 commits = 35 squashes + 1 merge de dois pais
  (`fd41d5a7`) + 4 push direto (`ef4ee2e8`, `eb36ad01`, `5ac7ed06`, `bd78c395`, todos de 26–27/11 e
  anteriores ao cursor em data, mas não em ancestralidade). 35 + 1 + 4 = 40. Bate.
- **`last_run`** gravado em `2026-11-28T01:42:49Z`: é o maior `merged_at` entre os **35 merges
  documentados**, o do `#3240` (`50e4f763`). O maior `merged_at` da janela é 11:47:40Z (`#2890`,
  `052abe6f`), mas esse commit ficou fora do teto de 40, então o cursor de mesclados não o alcança —
  e não deve alcançar, pelo mesmo motivo do cursor de commits.
