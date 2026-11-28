# R0Y00/Xbox2Windows — atualização de 2026-11-28 13:00

## Resumo

Quarenta commits novos a partir do primeiro não documentado (`2d6dad19`, índice 5 da página): os
cinco primeiros já estavam no documento das 12:00. A hora é a maior leva de **lowerings do DXIL
direto** até agora — 26 dos 40 commits fecham recusas do compute/pixel direto (dot products, `f16`
empacotado, salu de 64 bits com EXEC/VCC, atomics de imagem 2D, cache flags `GLC/DLC/SLC`,
derivadas implícitas no compute, round de `f16`, `div_fixup`, `mad mk/madak`) —, mais cinco consertos
de runtime de verdade: `4d7265c0` (erros de MSVC), `a0b45d43` (caminhos resolvidos no staging),
`c827d8ba` (primeiro draw gráfico nativo dentro do processo), `24eb18b9` (stderr do debugger
preservado no PowerShell 5.1), `20ef1f72` (BC4_UNORM devolvia o formato typeless 79). O repositório
está parado desde 05:26Z: o head é o mesmo `052abe6f` do `head_at_run`.

## Commits novos (40)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `2d6dad19` | legacy-code-archaeologist | shader: lower v_dot2_f32_f16 in direct Compute and Pixel DXIL | 478 linhas: `v_dot2_f32_f16` no `DirectCompute::float_alu` com o contrato `MODE` completo (`((mode_bits ^ 0xf0) & 0x8000ff)` e modo conhecido), `op_sel`/`op_sel_hi`, `neg_lo`/`neg_hi` e `clamp`; decodifica `src0`/`src1` por constante inline de f16 ou `operand32`, alarga com `legacyF16ToF32` e faz `((a0*b0)+(a1*b1))+c`. Preserva acumulador `c` subnormal de f32 quando a soma dos produtos é zero, porque o DXIL de desktop descarta denormal. Refusa `op_sel[2]`, DPP, máscaras reservadas e modo dinâmico. |
| `f9286e7d` | Leyukaka | compute: lower s_or_saveexec_b32 in the direct wave32 path | `s_or_saveexec_b32` em wave32: `D = EXEC antigo`, `EXEC = EXEC | S0`, `SCC = (EXEC != 0)`. Como `s_mov_b32 exec_lo`, o EXEC guarda os bits além do grupo e uma lane só roda se o próprio bit estiver ligado. Destino EXEC é recusado. |
| `24eb18b9` | Louis | startup: preserve debugger stderr and exit status | `tools/startup/run.ps1`: o PowerShell 5.1 embrulha stderr nativo redirecionado como *error record* e abortava o launcher antes de reportar o código. Agora o bloco roda com `$ErrorActionPreference = "Continue"` e guarda `$script:debuggerExit`; o teste ganha `emit_stderr`, `missing_python` e `real_log_error`. |
| `fdb6faaa` | Leyukaka | compute: store prefix-mask images at their exact width in direct compute | `image_store` não-D16 com `dmask` 0x1/0x3/0x7 em 2D, 2D array e 3D vira store tipado de `float`/`float2`/`float3` com todos os componentes escritos; o manifesto v2 carrega `channels 1|2|3` e `tex3d`, e o runtime exige que view e recurso tenham essa contagem. Teste de readback com EXEC `0x55` em mip 32×8. |
| `4d7265c0` | KelboLive | fix: MSVC compilation errors in startup tests | Conserto de compilação: `const void* const* vtbl` → `void* const*` em cinco arquivos de GameInput (13 ocorrências), `static_cast<UINT>` em `sizeof` no rectlist e no indirect draw indexed, e temporários de `D3D12_TEXTURE_COPY_LOCATION` em variáveis nomeadas antes de `CopyTextureRegion` (o MSVC recusa `&temporário`). |
| `c827d8ba` | Royoo | runtime: compile a supported graphics pair on first draw | `#ifdef X2W_NATIVE_SHADERS`: o stand-in lê a requisição gráfica publicada, liga VS/PS pelo caminho comum, usa os caches persistentes do compilador e consome o manifesto suportado; um par já publicado em disco tem precedência e uma recusa de manifesto pode se recuperar depois. Fixture de triângulo com 32 pixels verificados em dois processos. |
| `a0b45d43` | Louis | startup: compare resolved staging paths | `tools/startup/stage_package.ps1` compara **caminhos resolvidos** além da grafia: junction que leva para dentro do pacote é recusada, junction que sai do pacote também, junction interna é aceita, e caminho por GUID de volume funciona. Teste cobre os quatro casos. |
| `0637fced` | Leyukaka | compute: accept EXEC as a 64-bit scalar operand in direct compute | EXEC (126) passa a ser aceito como par de origem de `s_bcnt0/s_bcnt1`, `s_ff0/s_ff1` e `s_flbit_*_b64`, e como destino de `s_bfm_b64`. No modelo wave64 as duas metades do host carregam as duas palavras de EXEC. O controle de fonte inválida passa a usar `exec_hi` ímpar. |
| `9872074a` | legacy-code-archaeologist | shader: accept GLC and DLC cache flags on direct read-only image ops | `image_io` deixa de recusar `GLC` (bit 13) e `DLC` (bit 7) em `image_load`, `_mip`, `sample*`, `gather4*`, `getlod` e `getresinfo`; `LWE`, `TFE`, `R128` e bits reservados seguem recusados, e `GLC`/`DLC` em **store** também. 48 módulos comparados contra a linha de base sem flags. |
| `0fe4cebc` | Leyukaka | compute: allow implicit-derivative image sampling in direct compute | `image_sample*`, `gather4` e `get_lod` deixam de recusar em compute quando o grupo tem múltiplo de quatro threads: o DXIL passa a `numthreads(X*Y*Z,1,1)` e validador 1.8, com o índice linear do guest reconstruído (`wave rank * 32 + lane`). Grupo de 6 threads (quad incompleto) e de 36 (wave parcial) continuam recusados. Medido em RTX 5080 para 4/12/32/96 threads e grupos multidimensionais. |
| `bf1bc769` | saii | compute: read a resource-heap descriptor at an added byte offset | Um waterfall sobre slot de heap escolhido por lane forma o endereço do descritor como `s_add_u32 lo, <heap>, <offset>` + `s_add_u32 hi, 0, src_scc` (ou `s_addc_u32 hi, 0, 0`) + `s_load_dwordx4`; o compilador passa a rastrear SCC como o carry desse add e ler o par como o mesmo descritor indexado. Offset constante, `s_mov` no word alto, `s_cmp` entre os adds e offset imediato no load seguem recusados. |
| `a44e1f95` | Leyukaka | compute: lower s_andn2_wrexec_b64 for wave64 direct compute | `s_andn2_wrexec_b64` em wave64: `EXEC = S0 & ~EXEC` em 64 bits, máscara nova vai para EXEC **e** para o par de destino, `SCC = (EXEC != 0)`. A ordem de operandos e o destino seguem o `op_s_andn2_wrexec_b64` do `gfx2hlsl`. Teste com EXEC de quatro palavras pares. |
| `dbaa010b` | Leyukaka | compute: accept VCC and EXEC in direct 64-bit s_not and s_cselect | `s_not_b64` e `s_cselect_b64` aceitam VCC e EXEC como fontes de 64 bits (lendo `VCC_HI`/`EXEC_HI`), e `s_not_b64` pode escrever EXEC pelo mesmo par de escritas das outras máscaras. `s_cselect_b64` ainda recusa destino EXEC. |
| `5033d2d3` | tedin7 | startup: read translated compute inputs that live only in title memory from a snapshot | `X2W_TRANSLATED_TITLE_INPUT_SNAPSHOT=1` (opt-in): entrada de compute num buffer colocado que não faz alias com a memória do título (logo, buffer comprometido que só tem zeros) é lida de um snapshot da memória do título, em pool próprio de 128 MiB com blocos de 16 MiB reciclados. Entrada com as quatro palavras de descritor zeradas é buffer não ligado e lê 16 bytes de zero; faixa vazia com qualquer palavra não-zero segue recusada. |
| `a166a753` | saii | compute: name why a compiled program gets no runtime manifest | Cada recusa de `format_compute_runtime_manifest` devolve um motivo fixo (dimensão zero, root constants acima do parâmetro 15, mais de 8 num parâmetro, mais de 32 palavras de user data, view de heap, registro não contíguo, tipo sem linha de manifesto, mais de 32 SRVs/8 UAVs, sampler de comparação, stride de view que não é raw vector). `--requests` imprime o motivo no relatório e **não** escreve `.fail` (que é marcador de retry dos dois watchers); `--watch` escreve no `.fail`. |
| `fdba6e15` | saii | shader: set the 64-UAV-slot DXIL flag when a module declares more than 8 UAVs | `DxilModule::finish_compute` soma os tamanhos das faixas de UAV declaradas (limitados a 9 cada) e liga a flag `0x8000` quando passa de 8, porque o validador recusa módulo cujas flags não casam com o uso. Teste: 8 UAVs sem flag, 9 com. |
| `e8dd89ed` | Leyukaka | compute: lower binary16 VOP1 rcp, sqrt, exp and rsq | `v_rcp_f16`, `v_sqrt_f16`, `v_rsq_f16` e `v_exp_f16` (VOP1 e VOP3, com SDWA) alargam a metade selecionada e aplicam a operação em f32; `exp` usa uma tabela de 70 pontos não-empate gerada com 90 dígitos e resolve o empate exato em −25. `rsq` corrige o sinal de −0 (o `Rsqrt` do D3D não garante −infinito). O high half é preservado. |
| `152e93b4` | legacy-code-archaeologist | startup: lower cube image sampling in gfx2hlsl | `image_sample`, `_lz`, `_l` e `_b` com `dim:SQ_RSRC_IMG_CUBE` viram `TextureCube<float4>` e manifesto `texcube`; o helper `x2w_cube_dir(s,t,face)` reconstrói a direção a partir de `s,t ∈ [1,2]` e do id de face. Refusa `image_load` em cube, `sample_d`, `get_lod`, `a16`/`d16`, cube em compute e sampler constante com `DISABLE_CUBE_WRAP`. 22 draws de referência e 19 controles/mutantes em Quadro M1000M e WARP. |
| `6c5f4ed0` | triggered02 | startup: map a console package drive letter to a real folder | `-MapDrive "LETTER=FOLDER"`: o debugger reescreve aberturas numa letra mapeada para a pasta real, dentro do hook de arquivo existente; o `UNICODE_STRING` do chamador é restaurado ao voltar (comprimento, ponteiro e conteúdo) e nomes que não cabem usam buffer emprestado de um pool. `startup_map_drive` testa letra não mapeada, reuso de um `UNICODE_STRING` em 32 aberturas e duas threads com 32 aberturas cada. |
| `f41806a9` | Leyukaka | compute: lower v_dot4c_i32_i8 in direct compute | `v_dot4c_i32_i8` (VOP2 0x0d): produto escalar de bytes com sinal somado ao destino acumulador, módulo 2^32, via `Dot4AddI8Packed`. Aceita VGPR, SGPR, constante inline e literal em `S0`; VOP3, abs/neg/clamp, DPP16 e SDWA recusados. |
| `ff524dff` | Leyukaka | compute: accept GLC, DLC and SLC on raw buffer loads and stores in direct compute | `GLC` (bit 14), `DLC` (bit 15) e `SLC` (bit 22) deixam de recusar `buffer_load`/`buffer_store` crus em compute; não mudam o valor carregado. As máscaras de recusa passam a ser por estágio (`compute ? 0x32000 : 0x3e000`). |
| `dbb1c9aa` | John Connor | runtime: count XMA context and XApu command activity in the audio stand-ins | `X2W_AUDIO_ACTIVITY=1` com `X2W_AUDIO_PERMISSIVE=1`: `AcpHal` compara o array de contextos XMA com uma cópia (máx. uma vez a cada 10 ms) e conta preenchidos/mudados/limpos e submits HAL por tipo; `xapu` conta `XApuEnqueueCommand` por classe. Nenhuma DLL cria thread; exports experimentais devolvem os totais. O fixture prova zero-escrita, contador desligado e descarregamento limpo. |
| `fbef6381` | legacy-code-archaeologist | shader: lower v_madmk_f32 and v_madak_f32 in direct Compute and Pixel DXIL | Estende `unfused_mad`: `v_madmk_f32` (`S0 * K + S1`) e `v_madak_f32` (`S0 * S1 + K`) usam a sequência não fundida com FTZ nas entradas, produto e soma separados e sem contração FMA. VOP3/wide/subword/DPP recusados. 14 casos de IR e 4 recusas. |
| `8a8317e0` | Leyukaka | compute: let direct wave64 scalar loads use VCC as scalar data | No compute direto wave64, `smem_sgpr_limit` deixa de ser 106 e passa a 108: destinos podem terminar em `s107` (`VCC_HI`) e `VCC_LO`/`VCC_HI` podem ser o offset SGPR, como no wave32. Recusa nova e específica quando o lower simbólico de tabela/heap escreveria por cima do VCC do wave64. |
| `c2d327a4` | legacy-code-archaeologist | shader: lower v_rcp_iflag_f32 in direct Compute and Pixel DXIL | `v_rcp_iflag_f32` entra na tabela de ALU (`1.0 / {0}`), no mapa de opcodes (`0x2b`), no conjunto de floats do SDWA e no lowering direto, idêntico a `v_rcp_f32`. 16 casos de compute e 4 de pixel compilam igual. |
| `dbb933d1` | legacy-code-archaeologist | shader: lower v_dot2c_f32_f16 in direct Compute and Pixel DXIL | `v_dot2c_f32_f16` (VOP2 0x02) usa o lowering compartilhado de dot2 com `op_sel=0`, `op_sel_hi=3` e acumulador no próprio `vdst`, preservando subnormal de f32 e o rastreio de modo `0x8000ff`. VOP3 e modificadores recusados. |
| `32221f1a` | yjs | shader: carry pre-branch EXEC mask subset facts into compute execz blocks | `subset_after` rastreia fatos de subconjunto de máscara (`sM <= EXEC`) entre instruções e junções; salvar `s22 <= EXEC` e depois `s_mov_b32 exec_lo, s22` antes de `s_cbranch_execz` prova o re-remask dentro do bloco pulado. O caso `O-loop-subset` passa a produzir `[9,1,2,9,9,9,9,9]`. Estreitamento pré-branch, sobrescrita de máscara e junções não provadas seguem recusados. |
| `da968a50` | rdx | shader: lower v_floor_f16, v_ceil_f16, v_trunc_f16 and v_rndne_f16 in the direct compute path | Os quatro rounds de f16 em VOP1/VOP3A com aritmética inteira de 32 bits exata: `-0.0h`, `±min_sub`, `0.5h`, `1.5h`, `2.5h`, `-1.5h` e `1023.5h` (carry de mantissa para expoente), preservando `D[31:16]`. `clamp` satura para `[+0,1]` com NaN → +0. |
| `240e0fb9` | legacy-code-archaeologist | shader: lower v_pk_fmac_f16 in direct Compute and Pixel DXIL | `v_pk_fmac_f16` (VOP2 0x3c) entra no caminho empacotado com `fused_half_rne`, acumulador lido de `vdst`, contrato de modo `0x8000cc`; o controle negativo antigo que esperava a recusa é removido do `native_f16_alu_check.py`. |
| `b4c8ee5b` | Leyukaka | compute: lower more typed 2D image atomics in direct compute | O caminho de atomics tipados 2D de `#3162` (cmpswap, add, umax) ganha `swap`, `umin`, `and`, `or` e `xor`, mapeados para `AtomicBinOp` 8, 6, 1, 2 e 3. `smin`/`smax`, `sub`, `inc`, `dec` e atomics de float seguem recusados. |
| `fd7ce57a` | pigalot | tooling: add --assert-sites to the startup debugger | `--assert-sites N` (`-AssertSites`) passa por cima de **todo** hit em até N sítios `int3` distintos da imagem principal, então um assert que repete não consome orçamento; o primeiro hit de um sítio a mais para a run. `--continue-asserts N` segue contando hits, e cada orçamento conta só os hits que cobre. |
| `f9a591e3` | Roland Edi | startup: confirm command list slots from the 261000 table | A tabela `ID3D12GraphicsCommandList8` do GXDK 261000 confirma `BeginQuery`/`EndQuery`/`ResolveQueryData` em 28–30, `SetPredication` em 31, marcadores em 56–58, `ExecuteIndirect` em 61 e o reset do console em 139. O header passa a nomear os métodos confirmados e `Reset` vira `Reset (console)`; os slots 59–60 e 148 seguem sem nome. |
| `20ef1f72` | wish131400 | textures: return the typed DXGI value for BC4 UNORM | `tools/texture_blocks.py` devolvia 79 (`BC4_TYPELESS`) para `BC4_UNORM`; agora devolve **80**, o `DXGI_FORMAT_BC4_UNORM` público. Os dois testes de tabela passam a esperar 80 independentemente da tabela do módulo (antes ambos falhavam com `79 != 80`). |
| `aeef65b2` | wish131400 | tooling: reject TTPF manifests that replace requested DDS files | O exportador em lote recusa um caminho de manifesto que nomeia um DDS pedido pelo mesmo lote, ou uma entrada de histórico/calibração. Antes, `--indices 0 --manifest out/tex_00000.dds -o out` dizia sucesso e substituía o DDS pelo JSON; uma exportação falha podia perder o DDS anterior. A checagem roda antes de criar o diretório de saída. |
| `6d9b196d` | Roland Edi | runtime: compare the GDK runtime with the shim | `tools/gdk_runtime_oracle.py` compila o mesmo fixture contra a import library do GRDK e contra o shim, roda os dois, mascara handles/ids/temp e escreve diff normalizado. Medição de 28/11: `XGameRuntimeInitialize` não é exportada no shim, `XGameGetXboxTitleId` devolve `S_OK 0x58325731` no shim e `0x80070490` no GRDK, `XPackageMount` de identificador ausente devolve `S_OK` no shim e `E_NOTIMPL` no GRDK. |
| `11548af6` | Roland Edi | shader: add a GDK shader corpus harness | `tools/gdk_shader_corpus.py` compila seis HLSL originais com o `dxc` do GXDK Scarlett, retargeta o container com `dxil_retarget.py` e grava SHA-256 de fonte e container. Os seis casos compilam e verificam; o retarget remove a metadata `xbox.compiler.options` em todos. |
| `0f43029b` | Roland Edi | runtime: decode the 261000 xmem attribute layout | `era_heap.cpp/h` passa a decodificar o `XALLOC_ATTRIBUTES` do GDK 261000: `pageSize` 2 é **2 MiB** (era 4 MiB), bit 33 é "endereço abaixo de 4 GiB" e há 30 bits reservados acima. O bit 33 passa a ser recusado explicitamente (o alocador não fornece esse endereço) e os 30 reservados são validados. 10/10 cenários `era_heap`. |
| `635eedae` | wish131400 | ci: avoid the Windows NUL device in audio timeline tests | Renomeia o arquivo de teste `nul.csv` para `nul_byte.csv`, porque `nul` no Windows é device e o teste de erro de entrada caía em outro caminho. |
| `e28ac4ab` | legacy-code-archaeologist | shader: support neg_lo, neg_hi, and clamp on v_pk_min/max_f16 | `v_pk_min_f16`/`v_pk_max_f16` aceitam `neg_lo[1:0]`, `neg_hi[1:0]` e `clamp` (satura cada metade para `[+0,1]`, NaN → +0); `src2`, `op_sel[2]` e modificadores de terceiro operando seguem recusados, e nas variantes inteiras `neg`/`clamp` continuam recusados. 9 casos de modificador. |
| `74867491` | Roland Edi | tooling: report GDK knowledge coverage | `tools/gdk_knowledge_report.py` (1.676 linhas) funde a base JSON sob `D:\gdk-oracle\knowledge` num JSON de cobertura determinístico e num resumo Markdown, com conflitos estáveis por tamanho, ordinal, enum e valor; não lê header, library nem binário do GDK. |

## Pull requests abertos (41)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #3221 | native: bind a single directly indexed heap flag as a descriptor table | Habilita programas de compute nativo cuja root signature liga só uma das duas flags de heap direto. | Não. |
| #3225 | shader: lower pixel image descriptor size fields read as numbers | Estágio de pixel que usa descritor de imagem de 8 dwords passa a extrair largura, altura e níveis. | Não. |
| #3232 | Carry formatted buffer loads through byte-address windows to main | Porta o #3034 do `cpp-migration` para a `main` (buffer formatado por janela de bytes). | Não. |
| #3241 | shader: merge current main into the native shader module stack | Atualiza a branch de #3091 para a `main` `7e5c378d`. | Não. |
| #3248 | Integration stack: native PRs on current main | Draft: empilha os PRs nativos abertos para build conjunto. | Não. |
| #3266 | shader: lower v_min3_f16, v_max3_f16, and v_med3_f16 in direct Compute and Pixel DXIL | Lowering de VOP3 `0x351/0x354/0x357`. | Não. |
| #3279 | runtime: preserve CPU fence signal rewinds | `Signal` de CPU pode rebobinar fence; o stand-in publica todo valor por máximo atômico, inclusive depois de falha. | Não. |
| #3300 | shader: lower v_sin_f16 and v_cos_f16 in direct Compute DXIL | VOP1 `0x60/0x61`, VOP3 e SDWA. | Não. |
| #3305 | runtime: publish narrow half-float texture UAV stores | Stores de imagem D16 de 1 e 2 canais emitem DXIL mas param no runtime. | Área de **D-27**? não — é store, D-27 é leitura de coordenada `a16`. Sem linha. |
| #3311 | 'Fix compiler warnings in tests' | Corrige erros de build em `startup_gameinput_trace.cpp` e `startup_rect_geometry.cpp`. | Não. |
| #3315 | shader: pin the per-lane heap index at its waterfall mark | Closes #3304: o lowering de registro de heap por lane lia o SGPR no `s_load`, não no marca d'água do waterfall. | Área de **#2644 T3** (não é linha do catálogo). |
| #3317 | shader: lower v_cvt_pknorm_i16_f16 and v_cvt_pknorm_u16_f16 | VOP3 `0x312/0x313` no caminho direto. | Não. |
| #3319 | compute: lower 2D image_gather4_lz_o | Gather em LOD 0 com offset por lane nos bits do primeiro VGPR. | Não. |
| #3320 | compute: lower ds_bpermute_b32 without an LDS allocation | `ds_bpermute_b32` usa a crossbar de LDS mas não lê nem escreve LDS; o caminho direto recusava. | Não. |
| #3321 | gpu: trace descriptor history and GPU-written Mesh arguments | Contagem indireta de Mesh zero pode vir de cópia de descritor ausente/velha, SRV no escalar errado ou consumidor diferente. | Não. |
| #3324 | shader: lower v_div_fixup_f32 in the direct compute path | VOP3 `0x15f` com aritmética inteira IEEE-754 exata. | Não. |
| #3326 | shader: lower v_div_fixup_f16 | VOP3 `0x35f`, idem em f16. | Não. |
| #3328 | shader: lower v_div_fmas_f32 | VOP3A `0x16f`. | Não. |
| #3329 | compute: copy descriptor words through VCC as a scalar temporary | Programas que estacionam palavras de descritor em VCC (`s_mov_b64 vcc, s[18:19]`) e as movem de volta. | Não. |
| #3330 | compute: lower sub-dword LDS stores and loads | Stores/loads de LDS menores que dword no caminho direto. | Não. |
| #3332 | startup: let run.ps1 and run.sh turn off the hook thread holding | `-SuspendOthers 0` passa a opção ao `debug_launch.py`; padrão continua 1. Fixes #3331. | Área de **D-14** (call hooks do debugger), que trata de hooks armados por padrão; sem prova. |
| #3333 | compute: lower root-UAV global DWORD stores in the direct path | Fecha a recusa "direct global memory supports only DWORD loads with a scalar address pair" (9 shaders, 19.528 dispatches não traduzidos no censo de Not-Saii). | Não. |
| #3336 | compute: lower 3D and NSA typed image atomics | Atomics tipados uint aceitavam só 2D com endereço contíguo. | Não. |
| #3337 | shader: harden path, span, and collision guards in compile_sampled_pixel_request | Extrai `main(argv)` e trata `OSError`/`ValueError`/`Unsupported`. | Não. |
| #3338 | tooling: add bad-pointer fault-equivalence area to fuzz_cpu | Área `fault_reads` no fuzzer. | Não. |
| #3339 | shader: admit remaining lowered VOP1/VOP2 SDWA profiles | Admite nove perfis SDWA inteiros e três de float já baixados, mantendo recusas de gráficos. | **Área de D-21** (checagem SDWA `movrel`), sem prova. |
| #3340 | runtime: harden atomic report writes and output path guards in ui_regression_matrix | Valida `--build-dir` e caminhos de relatório, rejeita symlink. | Não. |
| #3341 | runtime: validate placed acceleration-structure updates | Nomeia as causas de recusa de builds de estrutura de aceleração colocada e resolve `PERFORM_UPDATE` com estrutura de origem. | Não. |
| #3342 | startup: output-window close review follow-up | Substituição de #2344 com o follow-up de review. | Não. |
| #3343 | d3d12: compute dispatch root-binding review follow-up | Substituição de #2491. | Não. |
| #3344 | shader: formatted buffer byte-window review follow-up | Substituição de #3232. | Não. |
| #3345 | tooling: test remaining GDK export regressions | Só teste: regressões do tooling de exports do GDK (follow-up de #3284). | Não. |
| #3346 | bcpack: BCPack refusal inspector review follow-up | Revisão alternativa de #1940. | Não. |
| #3347 | gpu: slot 133 texture replacement for #2773 | Substituição de #2773 sincronizada com a `main` `24fcd0b5`. | Não. |
| #3348 | [WIP] shader: offline native compute cache follow-up | Draft declaradamente incompleto. | Não. |
| #3349 | shader: replace #3225 image descriptor work | Substituição de #3225 com o feedback de descritor aplicado. | Não. |
| #3350 | runtime: preserve CPU fence signal rewinds (replacement for #3279) | Serializa `Signal` nativo da fence, observação de conclusão e publicação em cache por SRW lock. | Não. |
| #3351 | runtime: bind cube texture SRVs for published native compute | O tradutor declara cube como `TextureCubeArray`; o runtime ainda não liga essa view. | Não. |
| #3352 | compute: translate divergent multi-wave wave64 groups as lane pairs | Etapa 2 de #3237: wave64 do guest roda como duas wave32 que trocam palavras de máscara em barreiras. | Não. |
| #3353 | shader: lower direct MTBUF packed half stores | O caminho direto aceitava store empacotado D16 por MUBUF mas recusava MTBUF. | Não. |
| #3354 | compute: return lane 0 for an empty EXEC in split-halves v_readfirstlane_b32 | Depende de #3352; corrige a eleição de lane quando o EXEC está vazio na forma dividida. | Não. |

## Pull requests mesclados (37 de 71 na janela)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #3239 | shader: lower v_dot2_f32_f16 | `2d6dad19` | O lowering descrito acima, com o teste de 30 casos e o caso de subnormal de f32. Confere. |
| #3193 | compute: lower s_or_saveexec_b32 | `f9286e7d` | Máscara OR com destino EXEC recusado. Confere. |
| #2867 | startup: preserve debugger stderr and exit status | `24eb18b9` | `$script:debuggerExit` e os três casos novos de teste. Confere. |
| #3243 | compute: store prefix-mask images at their exact width | `fdb6faaa` | Manifesto v2 com `channels` e as recusas de contagem. Confere. |
| #3260 | build: fix MSVC compilation errors in startup tests | `4d7265c0` | As 13 trocas de vtable, os `static_cast<UINT>` e os temporários nomeados. Confere. |
| #2981 | runtime: compile a supported graphics pair on first use in the stand-in | `c827d8ba` | `try_native_graphics` e o fixture de 32 pixels. Confere. |
| #2872 | startup: compare resolved staging paths | `a0b45d43` | Junções resolvidas e o teste de volume GUID. Confere. |
| #3245 | compute: accept EXEC as a 64-bit scalar operand | `0637fced` | `s0 == 126` e `i.dst != 126` liberados nos bitscans. Confere. |
| #3244 | shader: accept GLC and DLC cache flags on direct read-only image ops | `9872074a` | Máscara `store ? 0x3e0c0 : 0x3c040`. Confere. |
| #3255 | compute: allow implicit-derivative image sampling in direct compute | `0fe4cebc` | `validator_minor = 8` e `finish_compute(x*y*z,1,1)`. Confere. |
| #3253 | compute: read a resource-heap descriptor at an added byte offset | `bf1bc769` | `heap_offset_add`, `heap_offset_carry`, `heap_offset_address`. Confere. |
| #3247 | compute: lower s_andn2_wrexec_b64 for wave64 | `a44e1f95` | `updates_exec = save || wrexec` e escrita de EXEC. Confere. |
| #3262 | compute: accept VCC and EXEC in direct 64-bit s_not and s_cselect | `dbaa010b` | `scalar64` para as duas operações. Confere. |
| #2862 | startup: read translated compute inputs … from a snapshot | `5033d2d3` | Snapshot com pool de 128 MiB e o caso longo de 48 frames. Confere. |
| #3268 | compute: name why a compiled program gets no runtime manifest | `a166a753` | `refuse()` com motivo e a separação `--requests`/`--watch` no `.fail`. Confere. |
| #3270 | shader: set the 64-UAV-slot DXIL flag | `fdba6e15` | Soma limitada a 9 por faixa e `0x8000`. Confere. |
| #3220 | compute: lower binary16 VOP1 rcp, sqrt, exp and rsq | `e8dd89ed` | A tabela de 70 pontos e a correção de sinal do `rsq`. Confere. |
| #2960 | startup: lower cube image sampling in gfx2hlsl | `152e93b4` | `x2w_cube_dir` e `texcube`, com as recusas declaradas. Confere. |
| #1251 | startup: map a console package drive letter to a real folder | `6c5f4ed0` | `--map-drive`, restauração do `UNICODE_STRING` e o caso de sobreposição. Confere. |
| #3246 | compute: lower v_dot4c_i32_i8 | `f41806a9` | `Dot4AddI8Packed` com acumulador no destino. Confere. |
| #3251 | compute: accept GLC, DLC and SLC on raw buffer loads and stores | `ff524dff` | Máscaras por estágio `0x32000`/`0xa00000`. Confere. |
| #2107 | runtime: count XMA context and XApu command activity | `dbb1c9aa` | As duas contagens, sem thread e com export. Confere. |
| #3250 | shader: lower v_madmk_f32 and v_madak_f32 | `fbef6381` | `lit_float()` e a escolha do addendo por mnemônico. Confere. |
| #3265 | compute: let direct wave64 scalar loads use VCC as scalar data | `8a8317e0` | `smem_sgpr_limit(direct_compute)` e a recusa simbólica. Confere. |
| #3254 | shader: lower v_rcp_iflag_f32 | `c2d327a4` | Entrada em `alu_rules`, opcode `0x2b`, SDWA e lowering direto. Confere. |
| #3257 | shader: lower v_dot2c_f32_f16 | `dbb933d1` | `dot2c` com `op_sel=0`, `op_sel_hi=3` e acumulador em `i.dst`. Confere. |
| #2877 | shader: carry pre-branch EXEC mask subset facts | `32221f1a` | `subset_after` e o caso `O-loop-subset`. Confere. |
| #3135 | shader: lower v_floor/ceil/trunc/rndne_f16 | `da968a50` | A aritmética inteira e os casos de `-0.0h` e `1023.5h`. Confere. |
| #3259 | shader: lower v_pk_fmac_f16 | `240e0fb9` | `packed_f16_alu(i, permuted)` com `fmac`. Confere. |
| #3271 | compute: lower more typed 2D image atomics | `b4c8ee5b` | `image_atomic_binop` com 8/6/1/2/3 e o deslocamento do teste de `smin/smax`. Confere. |
| #2916 | tooling: add --assert-sites to the startup debugger | `fd7ce57a` | Dois orçamentos independentes e o `STARTUP_DEBUG_PROBE_REVISIT`. Confere. |
| #3272 | startup: confirm command list slots from the 261000 table | `f9a591e3` | Nomes confirmados em 28–31 e 56–58; `Reset (console)` em 139. Confere. |
| #3275 | textures: return the typed DXGI value for BC4 UNORM | `20ef1f72` | 79 → 80 nos três lugares e nos dois testes. Confere. |
| #3277 | tooling: reject TTPF manifest paths that overwrite DDS or profile inputs | `aeef65b2` | A recusa antes de criar o diretório de saída e as seis variantes. Confere. |
| #3273 | runtime: decode the 261000 xmem attribute layout | `0f43029b` | 2 MiB, bit 33 e 30 reservados. Confere. |
| #3278 | ci: avoid the Windows NUL device in audio timeline tests | `635eedae` | `nul.csv` → `nul_byte.csv`. Confere. |
| #3263 | shader: support neg_lo, neg_hi, and clamp on v_pk_min/max_f16 | `e28ac4ab` | `bad_mods` por tipo e os 9 `MOD_CASES`. Confere. |

Os outros 34 merges desde `2026-11-28T01:42:49Z` (`#3269` a `#3299` e a série de 06:xx) estão à
frente do 40º commit documentado e ficam para o próximo ciclo.

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum hash do catálogo na janela (86 commits + 71 merges conferidos por prefixo
  contra os 262 hashes das 305 linhas).
- **Por área, com prova no diff:**
  - **D-11** (testes de hardware sem `SKIP_RETURN_CODE 77`): os testes novos desta hora todos declaram
    `SKIP_RETURN_CODE 77` (`native_dot2c`, `native_pk_fmac_f16`, `native_image_atomic_binops`,
    `native_dot4c`, `native_wrexec64`, `native_smem_vcc_wave64`, `native_mad32_k`). É a convenção que
    D-11 corrigiu; nenhum caso novo viola.
  - **D-18/D-19** (compilação de shader não retomada depois de rodada manifest-only; documentação do
    ciclo `.fail` obsoleta): `a166a753` é exatamente essa área — diz **por que** um programa compilado
    não recebe manifesto e reorganiza quem escreve `.fail` (`--watch`) e quem não escreve
    (`--requests`, marcador de retry). Não há prova de que fecha D-18; a documentação do `.fail` é
    atualizada no diff (`docs/experiments/native-compile-dxil-refusals.md`).
  - **D-27** (`a16` em coordenadas 2D de `image_sample`/`image_sample_lz` lidas como …): `152e93b4`
    mexe em `image_sample` de cubo e recusa `a16` explicitamente. Não é a linha de D-27 (que é
    leitura de coordenada `a16`), é a mesma família de modificador MIMG.
  - **D-17** (caso de heap em placed-buffer assumia prefixo reservado): `5033d2d3` trata justamente
    entrada de compute em buffer colocado que não faz alias com a memória do título, com opt-in e
    recusa explícita quando não há snapshot. Área de D-17, sem prova de correção da linha.
  - **D-05** (loads pendentes não unidos nas fronteiras dos braços mascarados): `32221f1a` carrega
    fatos de subconjunto de EXEC para blocos `execz` — mesma família de máscara/ramo, em `gfx2hlsl`
    e não no lowering direto.
- **Abre defeito novo?** Dois candidatos, ambos com evidência declarada no próprio diff:
  - `8a8317e0` removeu as recusas `wave64-smem-offset-106/107` e `wave64-smem-vcc-dst` dos testes e as
    trocou por uma recusa simbólica mais estreita. Se o modelo de VCC do wave64 estiver errado, o
    caminho deixou de falhar fechado — o diff não prova que todos os casos antigos continuam corretos.
  - `0f43029b` muda a semântica de `PageSize=2` de 4 MiB para 2 MiB. O próprio texto admite que as
    duas palavras gravadas do título deixam o bit 33 e os 30 reservados em zero e portanto **não
    distinguem** o layout 261000 do layout antigo do SlimEra. A mudança está baseada no header, não em
    medição do título.

## Confiança

**Verificado:** os 40 commits foram lidos pelo diff (13:05–13:45Z), com arquivos, contagens e trechos
de patch; os 37 merges casaram por `merge_commit_sha` com a janela; os 41 PRs abertos acima de #3215 e
os 71 mesclados desde `2026-11-28T01:42:49Z` vieram da API REST.

**Não verificado:** nenhuma build e nenhum teste. Os resultados de GPU citados (Quadro M1000M, RTX
5080, RTX 4070, HD 530 com skip 77) são os que os commits relatam; não os reproduzi. O diff completo
de `#3239` (478 linhas) foi lido em trechos, não linha a linha.

**Corte e cursor:** o `commit_cursor` gravado (`50e4f763`) era o 40º commit do ciclo anterior, mas a
ordem do `compare` difere: os 5 primeiros da página já estavam documentados às 12:00. A janela desta
hora começa no índice 5 (`2d6dad19`) e vai até o índice 44 (`74867491`), sem repetição. Próximo
cursor: `d649e470` (índice 45); restam 46 dos 86. O repositório está parado desde 05:26Z.

**Divergência de contagem:** 71 PRs mesclados desde `last_run`, 37 com `merge_commit_sha` na janela.
`last_run` avança para `2026-11-28T05:24:33Z` (#3263), o maior `merged_at` documentado.
