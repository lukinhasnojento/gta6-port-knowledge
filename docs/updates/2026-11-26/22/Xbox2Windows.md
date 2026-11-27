# R0Y00/Xbox2Windows — atualização de 2026-11-26 22:00

## Resumo

Trinta e dois commits fecham a faixa que a execução das 21:00 deixou pendente por teto — nenhum commit
novo caiu em `main` desde então (`6ee878d8` já era o `head_at_run`). O grosso é tradução de shader
(`gfx2hlsl`): uniformidade de `v_readfirstlane_b32`, reduções wave32 em estágio de pixel, primeiro lane
ativo, `v_lerp_u8`/`v_perm_b32`/`bitcmp`/`image_store` 2D-array, mais dois consertos de runtime com
impacto direto (anel de upload reaproveitado só após conclusão da GPU; *tables* de view consistentes
quando a alocação falha). Seis PRs foram mesclados na janela, mas **todos em `cpp-migration`**, não na
branch monitorada.

## Commits novos (32)

Faixa `1983228b..6ee878d8`, 32 commits, nenhum corte: `head_at_run` das 21:00 era exatamente `6ee878d8`.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `295afb7a` | RenzoWit35 | gpu: reject meshlet max_vertices below 3 (#2811) | `build_meshlets` passa a exigir `max_vertices` em [3, 256] e recusa 1–2, porque sempre coloca o primeiro triângulo e um triângulo pode precisar de três slots. Testes cobrem `max_vertices=3`, `max_primitives=1` e triângulos com vértice repetido. `src/meshlet_builder.cpp/.h`, `tests/meshlet_builder_check.cpp`. |
| `174f963d` | RenzoWit35 | tooling: run dxc_cache_check on Windows with a .cmd fake compiler (#2803) | O fake compiler deixava de ser um script com shebang executado direto (WinError 193 no Windows); agora escreve `fakedxc_impl.py` + `fakedxc.cmd` chamado com `sys.executable`. Caminho POSIX inalterado. Só `tests/dxc_cache_check.py` (+11/−3). |
| `d38dc93e` | RenzoWit35 | tooling: skip a UTF-8 byte-order mark in check listings (#2810) | `read_check_files` lê com `utf-8-sig`; um listing salvo com BOM não transforma mais U+FEFF no primeiro path nem trata comentário/linha vazia como arquivo. `tools/run_checks_parallel.py`, `tests/run_checks_fragments_check.py`. |
| `b0f49ce5` | RenzoWit35 | shader: pin v_fma_mix half selection per source position (#2806) | Só teste: verifica cada fonte de `v_fma_mix_f32`/`v_fma_mixlo_f16` separadamente (`op_sel_hi` lê a metade baixa, `op_sel`+`op_sel_hi` a alta, `op_sel` sozinho é recusado), com controle de fontes trocadas. `tests/rdna2_tools_check.py` (+86). |
| `e2271cce` | RenzoWit35 | startup: count acceleration-structure refusals by name in the slot 70 census (#2808) | `as_unaligned`, `as_not_placed` e `as_wrong_state` passam a constar de `memory_srv_reasons` e de `SLOT70_REASONS` do triage, em vez de caírem em "other"; o teste do censo deriva os nomes de `as_srv_refusal_name`. 6 arquivos. |
| `849c448d` | RenzoWit35 | runtime: list R8_TYPELESS, not R16_SINT, as typeless in the default-view check (#2807) | `typeless()` listava o formato 59 (`R16_SINT`) onde o 60 (`R8_TYPELESS`) devia estar: `R8_TYPELESS` ganhava view default e `R16_SINT` era recusado. Corrige o número e adiciona um caso por formato em `startup_view_dimension`. `src/startup_stubs/view_dimension.h` (+3/−1). |
| `e954fc55` | Alexofbulgaria | docs: record Intel Arc A770 hardware coverage (#2813) | Uma linha no README: Arc A770, driver 32.0.101.9175, 32 lanes (16 em run direto), faixa 8–32, 408 queries por run, duas execuções, debug layer indisponível. |
| `92934813` | RenzoWit35 | runtime: refuse page-token writes to pages without a grant (#2809) | `x2w_xmem_write_physical` criava seção de backing para qualquer página; uma escrita em página nunca concedida (ou com grant liberado) deixava uma seção fora de grant e o próximo grant daquele número mapeava os bytes escritos em vez de zeros. Agora checa `era_physical::owns` sob o mesmo mutex e devolve `FALSE`/`ERROR_INVALID_PARAMETER` antes de criar seção. Novo `tests/xmem_physical_write_owner_check.py`. |
| `49f53ae6` | RenzoWit35 | startup: input script rejects a relative time that overflows to infinity (#2812) | `parse_when` validava o número antes de somar o cursor, então `1@+1e308` virava `at = inf` e um wait infinito travava o resto do script em silêncio. Passa a validar o tempo resultante; formas finitas inalteradas. `src/startup_stubs/input_script.h`, teste de sintaxe. |
| `410f6004` | Not-Saii | tests: accept 4 ulps for v_log_f32 in the transcendental fixture (#2814) | O lane runner aceita limite de ulps por operação; `v_log_f32` passa a tolerar 4 ulps (RTX 4090 lê `log2(0.125)` como `c03fffff`, 1 ulp acima de −3,0, via FXC e DXC). O renderer continua comparação exata e a mutação precisa cair dentro do limite da mutação e fora do de referência. 3 arquivos. |
| `02075500` | Not-Saii | startup: lower v_fma_mix* through a precise mad so a zero product keeps its sign (#2818) | Nas RX 5700 XT (#2729) e RX 9070 XT (#2746) o caso −0 voltava `00000000`: operando binary16 ampliado por `f16tof32` alimentando multiplicação/mad não-preciso perdia o sinal. As três formas passam a chamar `mix32(a,b,c)` (mad `precise`); ordem de operandos, seleção de metade e modificadores inalterados; saturate fica fora. Cache revision avança. 13 arquivos (+68/−39). |
| `25eb746e` | thepatrickworld | readme: record the reported lane range for the RX 9070 XT wave64 run (#2821) | README: a coluna de wave64 forçado da RX 9070 XT passa de "Passed, 64 lanes" para a faixa medida 32–64, de build em `49f53ae6`, com link de evidência. |
| `7256cecf` | Chad Hyatt | gpu: preserve live upload reservations (#2823) | Recusa armazenamento de upload esgotado em vez de reutilizar capacidade sem conclusão; confere cursor de reserva e backing retido, com fixture portátil. 9 arquivos (+174/−62), incluindo `upload_reservation.h` e `startup_address_console_check.py`. |
| `d4dfe97b` | Luuk Motz | startup: accept the early exit of a kill in the slot 57 pixel stage (Part of #2413) (#2824) | `kill_exit_block` aceita o bloco `exp null ... done vm` seguido de `s_endpgm` que LLVM SILateBranchLowering/Mesa ACO põem após o fim do programa, desde que todo desvio para ele entre com EXEC zero (`s_cbranch_execz` ou `s_cbranch_scc0` logo após `s_cselect exec, exec, 0`). Demais formas mistas seguem recusadas. Testes com dez quase-acertos e desenho comparado a rasterizador de CPU. |
| `83086b92` | Mithrandir | shader: clear the lane-mask mark on scalar writes of descriptor data in gfx2hlsl (Part of #1723) (#2795) | ALU escalar sobre dados de descritor gravava só um símbolo e pulava `set_s`, deixando marca de lane-mask de uma escrita anterior; o clamp de stride de conversão structured→raw (`s_bfe_u32` + `s_max_u32`) era lido como leitura de máscara. Agora limpa a marca nos SGPRs de destino antes de escrevê-los. `tools/startup/gfx2hlsl.py` (+68/−2). |
| `8c9f168f` | Roland Edi | startup: document the storage path as a drive root (#2825) | Documenta que `X2W_STORAGE_PATH` é a raiz do drive que o título vê, não a pasta de save: o título anexa `x2w-gamesave`, então `S:\` grava em `S:\x2w-gamesave` e `S:\x2w-gamesave` grava em `S:\x2w-gamesave\x2w-gamesave`. `docs/startup-runtime.md`, `tools/startup/run.ps1`. |
| `6cf44bec` | rdx | startup: support pixel-stage 2D-array image_store and UAV binding (#2830) | Estágio de pixel ganha `image_store` em 2D-array e binding de UAV correspondente: `src/startup_stubs/d3d12_draw.inl` (+40/−15), testes `startup_pixel_uav.cpp` (+136) e checagens; `gfx2hlsl.py` muda uma linha. |
| `da782506` | Rachel Amber | gpu: allow live aliased indices with un-sliced placed vertex streams (#2834) | Troca a recusa global de aliasing de índice quando há vertex streams (`srv.source == 1`) por `stream_needs_slicing`, que só desliga o aliasing vivo se algum stream exigir fatiamento; streams placed/alinhados mantêm `live_indices` e as escritas tardias em índice e vértice. Retém os recursos de alias dos vertex buffers placed. `Fixes #2714`. 3 arquivos (+83/−10). |
| `4562c572` | yjs | shader: prove v_log_f32 and v_ceil_f32 results uniform for v_readfirstlane_b32 (#2838) | Prova de uniformidade para `v_log_f32`/`v_ceil_f32` alimentando `v_readfirstlane_b32` (`Refs #2644`): `gfx2hlsl.py` (+3/−2) e três checagens de recomputação/uniformidade. |
| `4992a46e` | pigalot | docs: point the Series S console profile at the plane composite (#2840) | Documenta que um título no perfil Series S pode apresentar em um plano acima do plano 0 preto, deixando a janela preta; liga a seção do perfil a "Presented planes" e nomeia `X2W_PRESENT_COMPOSITE=1` e `X2W_PRESENT_PLANE`, sem afirmar que todo título faz isso. `docs/startup-runtime.md` (+8). |
| `a6064651` | ColorMeBoom | startup: keep the split indirect Dispatch cache past DLL detach (#2734) | Com `X2W_SPLIT_INDIRECT_DISPATCH=1` numa RX 9070 XT o processo saía `0xC0000409` (fail-fast em `amdxc64.dll`) no destrutor do static local `SplitIndirectDispatch` durante o detach da DLL, que liberava o PSO da reescrita ali. `split_indirect_dispatch()` passa a devolver instância de heap nunca destruída; `docs/experiments.md` registra a vida e a falha. 2 arquivos (+34/−3). |
| `22aa7bf9` | Roland Edi | startup: record a bounded per-draw texel history (Part of #1268) (#2828) | Novo `src/startup_stubs/pixel_history.h` (+79) e registro limitado por draw em `d3d12_draw.inl` (+66/−3), com frame e coordenadas do texel por amostra; `startup_pixel_history.cpp` (+284) e registro no CMake. 8 arquivos (+467/−4). |
| `7538470e` | stan495 | shader: prove uniform compute image_load results uniform for v_readfirstlane_b32 (#2842) | Prova de uniformidade para `image_load` de compute alimentando `v_readfirstlane_b32`; `gfx2hlsl.py` (+77/−2) e checagens de recomputação e de uniformidade (inclui correção de offset de rótulo para llvm-mc). |
| `2c56d896` | Roan System | shader: recognise whole wave32 reductions in gfx2hlsl pixel stages (Part of #2125) (#2835) | Reconhece a cadeia completa de redução wave32 em estágio de pixel retilíneo nas três formas dos censos de #2125 (`s_or_saveexec_b32 sS, -1`; preenchimento de identidade por `v_cndmask_b32_e64`; DPP16 `quad_perm`/`row_half_mirror`/`row_mirror`; `v_permlanex16_b32`; combinação; `s_mov_b32 exec_lo`; `v_readlane_b32`) para `v_or_b32`, `v_min_u32` e `v_max_u32`, baixando para `WaveActiveBitOr/Min/Max`. 8 arquivos, +1169 linhas, incluindo 799 de teste e doc de experimento. |
| `3dfec4c1` | jj | tests: keep two hardware negative controls meaningful on RDNA 1 (#2730) (#2844) | `startup_gfx2hlsl_f32_med3`: o helper restaurado vira `precise`, senão a RX 5700 XT dobra o mediano contra limites constantes e desenha `0x3f800000` no lane NaN com FXC e DXC. `compute_store_aot_mac_f32`: o readback é comparado aos modelos fundido e não-fundido (diferem na palavra 0) antes de exigir falha da mutação. 3 arquivos (+53/−7). |
| `1940788d` | brainbudt | fix(shader): select the completed difference for SDWA subtract with borrow (#2850) | A RX 5700 XT devolvia o subtraendo na cadeia SDWA uniforme (#2731): `v_sub_co_ci_u32_sdwa` passa a selecionar `~(sb − sa)` (a diferença completada em aritmética uint) quando a condição vale, em vez de `cp`; `v_add_co_ci_u32_sdwa` inalterado. `tools/pixel_aot.py` (+7/−1) e asserção em `tests/pixel_aot_check.py`. |
| `8d389f5e` | yjs | startup: exercise the pixel-history hook through the translated draw path (#2851) | `Fixes #2843`: exercita o hook de pixel-history no caminho real de draw traduzido e em `queue_present`; novo `tests/startup_pixel_history_draw_check.py` (+110), +110 em `startup_stream_draw.cpp` e CMake registrando backends warp e hardware. |
| `6a094052` | JVProg | runtime: keep the view tables consistent when an allocation fails (#2799) | `note_view` inseria em `view_records`, `view_materialized` e `view_ids` em sequência sem tratar falha de alocação; agora tudo que aloca roda antes, e o registro entra nos três contêineres ou em nenhum, com log do contêiner, descritor destino não rastreado e liberação da classe de retire. Novo teste injeta falha nos três pontos (`startup_view_alloc`, +307). 5 arquivos. |
| `5a41adba` | brainbudt | fix(shader): identify compute loop exits that cross a backward SCC branch (#2852) | Distingue saída de laço contado que cruza um ramo SCC para trás dentro de bloco EXEC-zero (#2645 T2): o lane pode sair enquanto a wave pega o latch ou pegar o latch depois de a wave sair. `gfx2hlsl.py` (+6/−1), doc com fixture gfx1013 de 112 bytes e teste de traço. |
| `52905a32` | RenzoWit35 | startup: lower the first active lane of a pixel stage in gfx2hlsl (Refs #2618) (#2854) | `s_ff1_i32_b32 sD, exec_lo` seguido de `v_readlane_b32 sX, vY, sD` em estágio de pixel retilíneo vira `WaveReadLaneFirst(vY)` sobre os lanes ativos da wave desktop, sob o contrato de empacotamento de #2835; `sD` vira símbolo sem valor de run-time e qualquer outro uso é recusado (ramos, EXEC mudado, `s_wqm`, `v_writelane`, `exec_hi`). `gfx2hlsl.py` (+61), doc e teste de 330 linhas. |
| `b505c32b` | rdx | cpu: lower ENDBR64 and ENDBR32 as no-ops in the shared lowering (#2856) | `ENDBR64`/`ENDBR32` (`F3 0F 1E FA/FB`) passam a ser aceitos como NOP selecionados pelo `Code` exato do iced, sem efeito em registrador, flag ou memória; `RDSSPD`/`RDSSPQ` continuam recusados. `tools/ffx_version_query.py` (+11/−5) e testes `hint_nop_*`. |
| `6ee878d8` | brainbudt | fix(startup): reuse upload ring chunks after GPU completion (#2849) | O anel de upload recicla um chunk quando nenhum recording o retém e toda fila que o usou concluiu; textos que diziam que o storage nunca é resetado são corrigidos e o modo `gate_ring_reuse` é nomeado. `d3d12_xs_device.cpp` (+56/−37), `upload_reservation.h` (+63/−1), `d3d12_draw.inl` (+31/−25), testes. 9 arquivos (+236/−74). |

## Pull requests abertos (20)

Detectados 50 com número acima de 2861 (2867–2930, com lacunas). Documentados os 20 de menor número;
os 30 restantes ficam para a próxima execução com o cursor em `2887`.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 2867 | startup: preserve debugger stderr and exit status | Em PowerShell 5.1, stderr do debugger dispara `NativeCommandError` sob stop-on-error e o launcher sai 1 antes do resumo; escopa `Continue` à invocação e captura o exit code. +47/−9. | Não. Robustez do launcher. |
| 2868 | gpu: explicitly clear the pixel-history fixture target | O clear value otimizado de `CreateCommittedResource` não inicializa conteúdo; limpa explicitamente antes de cada sequência, sem mudar o oráculo de readback. +6/−12. | Não. Teste de #2828. |
| 2869 | gpu: isolate pixel-history fixture from inherited dump diagnostics | Remove `X2W_PIXEL*`/frame/dump herdados do ambiente do caso de dump completo; preserva asserções. +2. | Não. Isolamento de fixture. |
| 2870 | startup: read the literal data directory manifest | `-DataDir` com curingas fazia `run.ps1` ler `MicrosoftGame.config` de diretório irmão (`data [test]` → `data t`); usa caminho literal na checagem e na leitura. +19/−7. | Não. |
| 2871 | startup: read single-mip 0x105 volumes in slot 70 | Aceita `Texture3D` em layout `0x105` com um mip, uma amostra, formato não comprimido de tamanho conhecido e ≤2048 texels por lado, lendo o bloco espesso do addrlib. +659/−36. | Não. Amplia o slot 70; não é defeito do catálogo. |
| 2872 | startup: compare resolved staging paths | `stage_package.ps1` compara paths por grafia; compara caminhos resolvidos (subst, junction) de pacote, executável e ancestral existente mais próximo do destino antes de escrever. +129/−5. | Não. |
| 2873 | startup: lower v_lerp_u8 in gfx2hlsl | `disasm_rdna2.py` já decodifica `v_lerp_u8` (0x14d), mas `gfx2hlsl.py` não baixava: adiciona a `VEC_U` e emite `lerp_u8` com a semântica de média por byte com arredondamento. +216/−1. | Não. Lacuna de tradução. |
| 2874 | test: cover slot 35 table selection and invalidation at Dispatch | Porta de teste do PR NiiRoZz/Xbox2Windows#1: seleciona o par certo no índice 2, preserva tabela ao re-bindar o heap (320 palavras contra referência de CPU) e restaura root computacional. +80/−12. | Não. Só teste (caminho já em `main` por #2532/#2636). |
| 2875 | startup: lower v_perm_b32 in gfx2hlsl | Adiciona `v_perm_b32` (0x344) a `VEC_U`, com `perm_byte`/`perm_b32` emitidos sob demanda. +246/−1. | Não. Lacuna de tradução. |
| 2876 | startup: read 0x105 volume mip chains in slot 70 | Depende de #2871: lê cadeia de mips de `Texture3D` em `0x105`, mesma forma aceita, com mips cabendo na cadeia. +845/−41. | Não. |
| 2877 | shader: carry pre-branch EXEC mask subset facts into compute execz blocks | Implementa #2645 T1: dataflow direto de fatos `sM <= EXEC` por instruções escalares e junções, propagado em máscaras e limpo em escritas que estreitam EXEC. +185/−4. | Não. |
| 2878 | ffx: harden atomic output writes, exit propagation, and path guards in ffx_create_context | Valida paths (loader, `.cpp`, symlink, colisão de arquivo), escrita atômica de saída e propagação de exit; suíte CLI/mutação. +545/−5. | Não. Endurecimento de ferramenta. |
| 2879 | pe: harden module name and row schema guards in import_routing | `classify`/`annotate` validam nome de módulo e esquema da linha com `ValueError` em vez de `AttributeError`/classificação errada; suíte unitária. +266. | Não. |
| 2880 | startup: lower s_bitcmp0_b64 and s_bitcmp1_b64 in gfx2hlsl | Baixa SOPC de comparação de bit 64: `SCC = (S0.u64[S1.u[5:0]] == 0/1)`, com pares SGPR alinhados, `null` e inteiros inline. +441. | Não. Lacuna de tradução. |
| 2881 | startup: read block-compressed 0x105 volumes in slot 70 | Depende de #2876: aceita BC1–BC7 em `Texture3D` `0x105`, um mip ou cadeia, maior mip em blocos 4×4 inteiros. +970/−44. | Não. |
| 2882 | gpu: verify translated compute writes to two mips of one texture | Fixture RGBA16_FLOAT 32×8 + 16×4 mips; um Dispatch slot 67 escreve por dois UAVs slot 19 em entradas separadas; compara 1.280 meias e inverte views para provar o mapeamento. +138/−29. | Não. Teste de #2490. |
| 2884 | textures: harden bool/integer and byte-type bounds in texture_blocks | Recusa `bool`/não-`int` em dimensões e pitch, e nome de formato não-`str`, em `_grid`, `surface_bytes`, `linearize_*` e `tile_*`; suíte de mutação. +317/−13. | Não. |
| 2885 | runtime: harden bool/integer and page bounds in host_commit | Valida `page_size`, `limit_pages`, `used_pages` e orçamento em `snapshot_from_pages`/`require_commit_budget`; suíte com WinAPI mockada. +266/−4. | Não. |
| 2886 | shader: harden atomic output restoration and symlink guards in dxc_cache | `_write_file` recusa symlink/diretório e grava por `mkstemp`+`os.replace`, limpando temporário em qualquer falha. +464/−3. | Não. |
| 2887 | cpu: lower MOVNTPS, MOVNTPD, MOVNTDQ and MOVNTI in the shared lowering | Baixa os quatro stores não-temporais para `_mm_stream_si128/si32/si64`; `MOVNTPS/PD/DQ` exigem 16 bytes alinhados e faultam como `MOVAPS`. +383. | Não. |

## Pull requests mesclados (0 em `main`; 6 em `cpp-migration`)

Os seis merges da janela têm `base.ref = cpp-migration`, branch de trabalho do port em C++; **nenhum
toca `main`**, então não entram como mudança da branch monitorada. Ficam registrados para não se perder
o rastro:

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| 2914 | shader: draw a per-vertex VCC across a vertex-stage join | `cd752a8c` | Merge em `cpp-migration` às 21:18:03Z; não em `main`. |
| 2918 | runtime: bring XMem page-run fixes into cpp-migration | `ee76dfa8` | Merge às 21:23:40Z; revertido 23 minutos depois por #2931. |
| 2917 | shader: lower v_pack_b32_f16 in the direct C++ DXIL path | `16099d7c` | Merge às 21:37:05Z; não em `main`. |
| 2931 | Revert "runtime: bring XMem page-run fixes into cpp-migration (#2918)" | `7f951fd3` | Reverte #2918 às 21:47:04Z — 23 min de vida, mesma classe do **D-29** (merge e revert no mesmo dia). |
| 2928 | tests: expect the decoder's refusal for the invalid word in native_wave_request | `e4523d97` | Merge às 21:55:55Z; não em `main`. |
| 2932 | shader: emit !llvm.ident in direct DXIL modules for AMD PSO creation | `e97c5c7c` | Merge às 21:59:23Z; não em `main`. |

## Cruzamento com o catálogo

- **Corrige defeito conhecido?** Nenhum dos 32 commits casa com entrada do catálogo do `Xbox2Windows`.
  A faixa é quase toda lacuna de tradução (`gfx2hlsl`) e robustez de ferramenta/teste.
- **D-29** (mutação temporária em `main`, revertida 88 s depois): o par #2918 → #2931 repete o padrão
  na branch `cpp-migration` — merge `ee76dfa8` às 21:23:40Z, revert `7f951fd3` às 21:47:04Z. Não é
  `main`, mas o catálogo só descreve o sintoma; vale ampliar a entrada para "qualquer branch".
- **D-28** (conversão UNORM/SNORM do store formatado com `round()`): `6cf44bec` mexe em `image_store`
  de estágio de pixel (2D-array) mas não toca a regra de arredondamento; não corrige.
- **D-27** (`a16` de `image_sample`/`image_sample_lz`): `7538470e` trata `image_load` de compute e
  uniformidade para `v_readfirstlane_b32`; outro caminho, não corrige.
- **D-21** (seleções SDWA `movrel`): `1940788d` é SDWA subtract-with-borrow, defeito distinto; a entrada
  do catálogo não cobre o novo caso.
- **Abre defeito novo?** Três candidatos, todos com hash:
  - `1940788d` — `v_sub_co_ci_u32_sdwa` devolvia o subtraendo na RX 5700 XT (#2731); `tools/pixel_aot.py`.
  - `849c448d` — `typeless()` com o número de formato trocado (59 vs 60); `src/startup_stubs/view_dimension.h:?`.
  - `6ee878d8` / `7256cecf` — anel de upload reutilizava capacidade sem conclusão da GPU em `d3d12_xs_device.cpp` e `upload_reservation.h`.
- **Cross-repo:** `4992a46e` documenta o plano composto de apresentação do Series S (`X2W_PRESENT_COMPOSITE`),
  o mesmo problema que `85a663d3` do `avatarmanz/VIctory-7900XTX` resolve no host (dois planos compostos).

## Confiança

Verificado: os 32 hashes e autores via `gh api .../compare/1983228b...6ee878d8` e `.../commits/<sha>`
(arquivos, adições e remoções lidos em cada um); a lista de PRs abertos (`--state open --limit 50`) e os
corpos dos 20 documentados via `.../pulls/<n>`; o `base.ref` dos seis mesclados, um a um.

Não verificado: se os PRs abertos compilam ou passam nos testes; o conteúdo dos 30 PRs abertos não
documentados; se os commits de `cpp-migration` chegarão a `main` e por qual merge. A contagem de PRs
mesclados (6) não corresponde a nenhum commit da janela de `main` — a divergência é o próprio fato
registrado: os seis visam `cpp-migration`.
