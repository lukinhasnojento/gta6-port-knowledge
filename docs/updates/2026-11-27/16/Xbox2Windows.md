# R0Y00/Xbox2Windows — atualização de 2026-11-27 16:00

## Resumo

A `main` andou 80 commits desde `e62e3a5e`, com 50 PRs mesclados na janela (44 na `main`, 6 em
`cpp-migration`). Este documento cobre os **40 commits mais antigos** (teto da execução, até
`f52046900a`); os 40 restantes ficam para a próxima hora com `commit_cursor` gravado. O volume maior é o
frontend DXIL nativo em C++ que entrou inteiro em `4036577020` (15.618 linhas, 69 arquivos) e a série de
lowerings que veio depois dele; fora isso, a hora trouxe dois consertos de infraestrutura de medição
(`7beb331d`, `e97c5c7c`) e uma reversão deliberada dos consertos de XMem em `cpp-migration` (`7f951fd3`).

## Commits novos (40 de 80)

Faixa `e62e3a5e0005a695616c3a84a89b4022d0f28d7e...801f4c3bea655baab487bd9298a66ba10e114430` (`ahead_by`
80). Documentados os 40 mais antigos; os 40 seguintes têm hash em `state/watch.json` via `commit_cursor`
e voltam na próxima execução.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `4036577020` | Royoo | Add native gfx1013 DXIL compiler, replay CLI and source-built fixtures | +15.618/−0 em 69 arquivos. Cria o frontend nativo: `src/gfx1013_compute_native.cpp` (1.821 linhas), `src/dxil_module.h` (280), `src/gfx1013_isa_tables.inl` (748, geradas de `aco_opcodes.py` do Mesa 22.3), `src/gfx1013_isa.cpp`, `src/gfx1013_direct_compute.inl` (1.779), `src/gfx1013_direct_graphics.inl` (2.041), `src/gfx1013_graphics_wqm.inl`, `src/gfx1013_pixel.cpp`, `src/gfx1013_vertex_id.cpp`, `src/gfx1013_sampled_pixel.cpp`, `src/guest_shader_container.cpp`, cache (`src/direct_compute_cache.cpp`), compilador DXIL (`src/shader_dxil_compiler.cpp`) e o CLI `tools/startup/compile_dxil.cpp`. Traz ~20 fixtures novos (`tests/native_*`), inclusive `native_wave_request_check.py`, `native_lds_control_flow_check.py`, `native_pixel_ir_dump.cpp`. Não altera nenhum caminho existente: só adiciona, mais 124 linhas em `CMakeLists.txt` e 47 em `docs/experiments.md` |
| `cd752a8c9e` | saii | shader: draw a per-vertex VCC across a vertex-stage join (Refs #2906) (#2914) | +185 em 3 arquivos. Só teste e doc: `tests/native_graphics_vertex_vcc_join_check.py` desenha dois estágios de vértice gfx1013 construídos na fonte, cada braço do branch escreve VCC e o `CNDMASK` consome a máscara unida; compara readback com modelo de CPU em WARP e hardware |
| `ee76dfa81a` | sydney-0o | runtime: bring XMem page-run fixes into cpp-migration (#2918) | +2.544/−94 em 19 arquivos. `src/xmem_mapped_pages.cpp` +504/−34 (quebra de run por `MappingThread`, merge de views entre chamadas), `src/xmem_shim.cpp` ganha `x2w_xmem_stable_call`, fallback de split de placeholder quando o SO não reporta o bit `0x100` (Wine), cinco fixtures novos (`xmem_run_break_readers_check.cpp` +803, `xmem_cross_call_merge_check.cpp` +464) |
| `16099d7cfb` | Yann Bucaille | shader: lower v_pack_b32_f16 in the direct C++ DXIL path (#2917) | +314. Lowering de VOP3 `0x311` só com operações inteiras: `D[15:0]=S0.f16`, `D[31:16]=S1.f16`, seleção de metade por `op_sel`, `abs`/`neg` por bit de sinal. Recusa clamp, omod e terceira fonte. Fixture `native_pack_b32_f16_check.py` (+210) |
| `7f951fd3de` | TheVoid | Revert "runtime: bring XMem page-run fixes into cpp-migration (#2918)" (#2931) | −2.544/+94: desfaz `ee76dfa8` inteiro, incluindo os cinco fixtures e as 118 linhas de `CMakeLists.txt`. Motivo no diff: os 17 commits de XMem são de #2911, ainda em revisão contra a `main`; mantê-los fora deixa `cpp-migration` restrita ao compilador nativo |
| `e4523d974b` | JB2WIN | tests: expect the decoder's refusal for the invalid word in native_wave_request (#2928) | +3/−1 em `tests/native_wave_request_check.py`: a palavra `0xffffffff` passou a ser recusada pelo decodificador antes do lowering, então a asserção muda de `"unsupported instruction"` para `"unknown gfx1013 instruction format at byte"`, mantendo a ausência do `.cso` |
| `e97c5c7ce9` | Matheus Sergio | shader: emit !llvm.ident in direct DXIL modules for AMD PSO creation (#2932) | +370/−17. `src/dxil_module.h` acrescenta o nó `!llvm.ident = !{!"x2w"}` em `finish_compute` (cs/vs/gs/ps) e `src/gfx1013_pixel.cpp` renumera os metadados. O diff documenta a causa: `amdxc64.dll` 8.18.10.0514 (driver 32.0.32029.2010, RX 9070 XT) faz `getNamedMetadata("llvm.ident")` e desreferencia o retorno sem checar nulo em `CreateComputePipelineState`/`CreateGraphicsPipelineState` |
| `7beb331d04` | Mithrandir | shader: read requests at the runtime's revision 28 in compile_dxil (#2939) | +4/−4. `src/shader_compute_request.cpp` troca `"x2w-startup-shader-14"`/`-14cs` por `-28`/`-28cs`; o runtime (`d3d12_draw.inl`) e `gfx2hlsl.py` já publicavam na revisão 28, então todo pedido era recusado como key mismatch. Dois fixtures passam a publicar com o mesmo prefixo |
| `1cd48bd061` | Leyukaka | shader: lower DPP16 quad_perm in the direct C++ compute path (#2936) (#2937) | +210/−9. `src/gfx1013_compute_native.cpp` aceita `pattern > 255` no compute (antes só gráficos); `mirror()` vira `lower_dpp()` e lê a lane `(lane & ~3) \| ((pattern >> 2*(lane&3)) & 3)` via `waveReadLaneAt`. Fixture `native_dpp_quad_check.py` cobre aritmética, seleção de lane e fonte desabilitada |
| `a990f323c9` | sydney-0o | shader: restore compilation of LDS replay requests (#2941) | +82/−15. `tools/startup/compile_lds_request.py` deixa de tratar só `ds_read*` como tendo resultado e passa a olhar `_rtn_` (atômicos que devolvem); `gfx2hlsl.py` separa `record_v_write` do registro do último VGPR. Fixture `native_lds_request_check.py` |
| `c2f6d3de49` | sydney-0o | ci: exercise native DXIL on pull requests into cpp-migration (#2942) | +11/−1. `.github/workflows/windows.yml` passa a disparar em `[main, cpp-migration]` e roda `native_dxil_llvm_ident_controls`; `CONTRIBUTING.md` documenta a regra |
| `ea07313b0e` | Roan System | shader: vote a quad-complete EXEC for WQM EXECZ branches in native pixel draws (#2946) | +487/−8. Novo `enum class MaskSource { survivors, whole_quad, narrowed }` em `src/gfx1013_graphics_wqm.inl` e `raster_vote` em `gfx1013_direct_graphics.inl`: `s_cbranch_execz` depois de `s_wqm` vira `WaveActiveAnyTrue` da EXEC modelada OR-ada no quad, mas só para a forma limitada (escritas escalares e salto para o restore de cobertura). Fixture com 206 linhas novas |
| `641add964b` | Leyukaka | shader: lower v_readlane_b32 and v_writelane_b32 in the direct C++ compute path (#2940) (#2950) | +275/−23. `v_readlane_b32` lê a lane escolhida com `waveReadLaneAt` (metade tirada do índice para wave64 dividida); `v_writelane_b32` escreve só na lane alvo e ignora a EXEC do guest; lane constante ou SGPR mascarada à largura da wave |
| `49e0e36d11` | Leyukaka | shader: accept inline, literal and SGPR-pair values in direct C++ 64-bit shifts (#2951) (#2952) | +163/−3. `v_lshrrev_b64`/`v_lshlrev_b64` aceitam inteiro inline assinado estendido a 64 bits, literal de 32 bits zero-estendido e par de SGPR par (`src <= 104 && (src & 1) == 0`); inline float continua recusado |
| `e3670100c8` | Matheus Sergio | shader: consume startup requests and publish runtime cache entries in compile_dxil (#2955) | +977/−101 em 15 arquivos. Lê `req_<key>.psin` (`psin1 <key> ena= addr=`), valida as máscaras `SPI_PS_INPUT_ENA`/`ADDR`, dobra `b'psin1' + pack('<II', ena, addr)` na chave FNV-1a; publica `<key>.cs.cso`/`.vs.cso`/`.ps.cso`, retira manifestos velhos e grava o `<key>.txt` por último; abre entrada de cache com `FILE_SHARE_DELETE` para `--jobs` concorrente |
| `f934df5217` | Leyukaka | shader: lower s_bfm_b64, s_cselect_b64 and s_not_b64 in the direct C++ compute path (#2957) (#2959) | +222/−3. As três viram duas palavras escalares: `s_bfm_b64` com campos de 6 bits, `s_cselect_b64` seleciona as duas palavras por SCC, `s_not_b64` inverte e põe SCC do resultado de 64 bits |
| `b260984f8a` | saii | runtime: translate slot 67 compute requests in the stand-in (#2958) | +377/−17. Com `X2W_NATIVE_SHADERS=<dir de dxcompiler.dll>` o primeiro `Dispatch` de um pipeline slot 67 lê o pedido publicado, roda o frontend em C++ e o compilador DXIL no processo (`src/startup_stubs/d3d12_compute_dispatch.inl`), em vez de exigir cache pré-compilado |
| `8946fb16e7` | SimSalabimse | shader: lower binary16 add, sub, subrev and mul in direct compute (#2964) | +815. Os quatro VOP2/VOP3 alargam a metade baixa, operam em f32 e arredondam com a conversão inteira de `v_cvt_f16_f32`; metade alta do resultado zerada. Recusa por nome SDWA, DPP, clamp, omod, `op_sel` de metade alta e modo de arredondamento/denormal não padrão |
| `0b1456ce1e` | legacy-code-archaeologist | shader: lower root-pointer global_load_dword in the direct C++ compute path (#2966) | +264. Primeira instrução FLAT aceita: `global_load_dword` na forma `saddr` (segmento 2), quando SADDR é o endereço raiz não avançado de um root SRV; offset `zext(VADDR)+sext(OFFSET)` limitado a `[0, 2^32)` |
| `78a071caab` | Leyukaka | shader: accept byte-window buffer descriptors for stores in the direct C++ compute path (#2965) (#2967) | +169/−2. `buffer_store_raw` aceita o descritor reescrito como janela de bytes: `word1 & 0xc000ffff`, `word2 *= stride`, `word3 \| 0x30000000` passa a ser reconhecido também quando a fonte do `s_or_b32` é `word3` (antes só `clear_oob`) |
| `2b678fd372` | legacy-code-archaeologist | shader: lower image_gather4_c_lz in the direct C++ image path (#2971) | +169/−11. MIMG opcode 79 em 2D e 2D-array com `dmask 0x1` vira `textureGatherCmp`: canal 0, referência na primeira dword de endereço, sampler de comparação constante e o mesmo ajuste de nível-base de `image_gather4_lz` |
| `7bcff286f9` | Leyukaka | shader: lower the SDWA forms of binary16 add, sub, subrev and mul in direct compute (#2963) (#2973) | +242/−17. As formas SDWA leem WORD_0/WORD_1 de cada VGPR (DWORD = metade baixa), aplicam abs/neg, calculam em f32 e arredondam uma vez com `packed_half_rne`; `dst_sel` WORD_0/WORD_1 decide onde a metade cai |
| `05438d652c` | JB2WIN | native: capture and replay view types for GPU-selected global SRV slots (#2972) | +1.503/−35 em 17 arquivos. `dx.op.createHandleFromHeap` (opcode 218) em `dxil_module.h`, `HeapViewContract`/`heap_buffer_srv` no frontend, flag `heap_indexed` na raiz, e os metadados `req_<key>.views` capturados no pedido. Só a classe `raw_buffer` é traduzida; as outras são recusadas por nome |
| `4ff5e36d12` | yjs | tooling: write .fail diagnostics and add read-only --refusals summary to compile_dxil (#2984) | +692/−19. `compile_dxil` grava `<key>.fail` com o motivo da recusa e ganha `--refusals`, que só lê o diretório (não escreve cache); `triage_run.py` reconhece as famílias de mensagem `unsupported instruction`/`unsupported opcode` |
| `b69c111f69` | legacy-code-archaeologist | shader: lower the SAD operations in the direct C++ compute path (#2990) | +216. `v_sad_u8`, `v_sad_hi_u8`, `v_sad_u16`, `v_sad_u32` e `v_msad_u8`: soma de diferenças absolutas de quatro bytes, dois halfwords ou uma dword, mais S2, módulo 2^32; `v_msad_u8` ignora byte cujo par em S1 é zero; `v_sad_hi_u8` desloca a soma 16 bits |
| `c6908ccec2` | legacy-code-archaeologist | shader: lower v_ashrrev_i64 and v_mad_i32_i24 in the direct C++ compute path (#2994) | +228/−3. `v_ashrrev_i64` entra no mesmo caminho dos shifts lógicos com `ashr i64` e literal de 32 bits assinado; `v_mad_i32_i24` completa o par do `v_mad_u32_u24` |
| `79a8db6ca7` | yjs | tooling: add --watch mode to compile_dxil for live startup shader requests (#2992) | +562. `--watch` (contínuo e `--once`) observa os pedidos de startup e recompila o que muda, reaproveitando o ciclo de `<key>.fail` |
| `e0d837594b` | saii | shader: resolve and cache RTV variant requests in compile_dxil --requests (#2989) | +635/−22 em 13 arquivos. Lê `req_<key>.rtv` (`rtv1 <base key> <formato> ...`, até 256 bytes, 1 a 8 formatos), valida a chave do pedido base, tipa o alvo pelo formato em `target_components` e publica variante e manifesto |
| `24d92609d9` | legacy-code-archaeologist | shader: lower v_alignbit_b32, v_alignbyte_b32 and v_bfm_b32 in the direct C++ compute path (#2986) | +166/−1. `v_alignbit_b32` devolve bits `[k+31:k]` de `{S0,S1}` com `k=S2[4:0]`; `v_alignbyte_b32` o mesmo com `k=8*S2[1:0]`; ambos devolvem S1 para k=0; `v_bfm_b32` é `((1<<S0[4:0])-1)<<S1[4:0]` |
| `4701650dbf` | legacy-code-archaeologist | shader: lower v_cvt_pk_u16_u32 and v_cvt_pk_i16_i32 in the direct C++ compute path (#2996) | +175. Empacotamento saturante: `min(S0,0xffff)`/`min(S1,0xffff)` num caso, clamp assinado a `[-32768,32767]` no outro, S0 na metade baixa; recusa SRC2 não zero e `op_sel` |
| `af6d255fdc` | legacy-code-archaeologist | shader: lower v_cvt_pk_u8_f32 in the direct C++ compute path (#2998) | +197. `(S2 & ~(0xff << 8*S1[1:0])) \| (flt32_to_uint8(S0) << 8*S1[1:0])`, com teste inteiro `(bits-1) < 0x7f800000`, `FMin` com 255, `Round_ne` e `fptoui`: NaN, zeros e negativos dão 0, +inf dá 255 |
| `999e17cde9` | yjs | tooling: support .rtv variant requests in compile_dxil --watch and --refusals | +156/−24. `--watch` e `--refusals` passam a tratar `req_<key>.rtv`; `dedupe_graphics_markers` documenta e assume a pré-ordenação em que o `.a` de um stem precede seu `.rtv` |
| `bb347ad769` | rdx | startup: lower packed byte interpolation in gfx2hlsl (#2873) | +216/−1. `v_lerp_u8` em `tools/startup/gfx2hlsl.py`: média por byte com o bit de arredondamento, recusando modificadores codificados não suportados |
| `215934bdf3` | Matheus Sergio | startup: bind root SRV buffers at compute Dispatch (#2953) | +605/−9. `d3d12_compute_dispatch.inl` passa a ligar root SRVs rastreados e limitados registrados pelo slot 43 (`srv <i> buf root <param>`), e restringe a rota indireta a buffers crus de tabela fixa; `list_upload_tail` novo em `d3d12_xs_device.cpp` |
| `cbae12fd79` | Roland Edi | docs: plan EXEC removal and structurization (#2923) | +223 em 1 arquivo (`docs/experiments/exe-structurization.md`). Só plano: prova de fluxo de controle em etapas, com fixtures construídos na fonte e controles negativos. Nenhum código |
| `17f27f85e3` | Roland Edi | docs: plan slot 57 mesh DXIL translation (#2924) | +167 em 1 arquivo (`docs/experiments/mesh57-dxil-plan.md`). Só plano: marcos separáveis de tradutor, criação de pipeline e dispatch de mesh. Nenhum código |
| `2944b0502b` | Roland Edi | tests: model the console BVH node set (#2921) | +1.258/−2. `tests/gfx1030_bvh_box/bvh_model.py` (235 linhas) e +917 em `check.py`: modelo de CPU dos formatos de nó de BVH do console, com testemunhas só de modelo (sem GPU). Nenhum caminho de produto muda |
| `6aa594408c` | JB2WIN | startup: back off missing shader manifest retries (#2925) | +140/−12. Novo `src/startup_stubs/manifest_retry.h`: ausência de `<key>.txt` é reconsultada 1 s depois e o intervalo dobra até 32 s; achar o manifesto reinicia o ciclo. Substitui os dois contadores `retry_manifest_at` soltos em draw e compute |
| `98e4a2eea9` | lukaslw-lab | startup: opt-in release of upload ring chunks at a fenced submit (#2949) | +320/−8. Com `X2W_D3D12_RING_RELEASE_AT_SUBMIT=1`, uma lista fechada devolve seus pedaços do anel quando o submit é cercado, e o anel cresce em blocos de 16 MiB até 512 MiB; um submit posterior da mesma lista antes do `Reset` recusa a lista se algum pedaço foi reusado (`upload_resubmits_refused`). Fecha a issue #2948 |
| `f52046900a` | Leyukaka | startup: lower compute image_store to 1D and 3D texture UAVs (#2902) | +275/−46 em 14 arquivos. `image_store` de compute com `dmask:0xf` em imagem 1D/3D vira store em `RWTexture1D<float4>`/`RWTexture3D<float4>`, e o `Dispatch` liga a view do título depois de validar mip e, no 3D, a faixa de W; o registro de UAV recusada deixa de usar `uav_denied` booleano |

## Pull requests abertos (14)

Janela de `gh pr list --state open --limit 50` (50 PRs, números 2836–3060); acima do cursor 3011 há 14.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 3014 | compile_dxil: add --rekey mode matching shader_cache_rekey.py | modo de reescrita de chave no CLI, espelhando o script do runtime (branch `cpp-migration/compile-dxil-rekey`, base `cpp-migration`) | Não consta no catálogo |
| 3022 | shader: lower v_permlane16_b32 and v_permlanex16_b32 in direct compute | dois lowerings de permutação de lane no frontend nativo (base `cpp-migration`) | Não consta (família de lane access já coberta por `641add96`) |
| 3028 | startup: opt-in skip of the graphics root SRV copy that no draw reads | pula por opção a cópia de root SRV de gráficos que nenhum draw lê | Não consta |
| 3030 | startup: lower s_bitreplicate_b64_b32 in gfx2hlsl | lowering escalar de replicação de bits | Não consta |
| 3031 | startup: lower v_dot4_i32_i8 and v_dot4_u32_u8 in gfx2hlsl | produtos de ponto de 4 bytes no tradutor Python | Não consta |
| 3032 | startup: lower the binary16 BVH box node | caixa de nó BVH em f16 no `image_bvh_intersect_ray` | Relaciona-se a `2944b050` (modelo de nó BVH) e a `0d228580` (#2922, caixa FP32), não é o mesmo trabalho |
| 3033 | startup: lower v_sat_pk_u8_i16 in gfx2hlsl | empacotamento saturado u8/i16 | Não consta |
| 3036 | startup: lower v_dot2_i32_i16 and v_dot2_u32_u16 in gfx2hlsl | produtos de ponto de 2 halfwords | Não consta |
| 3049 | runtime: bind native compute table buffers and texture views | ligação de buffers de tabela e views de textura no compute nativo | Não consta |
| 3056 | tooling: add a hash-gated patch manifest loader | carregador de manifesto de patch com trava por hash | Não consta |
| 3057 | shader: lower v_ldexp_f32 in the direct compute path | `v_ldexp_f32` no frontend nativo (base `cpp-migration`) | Não consta |
| 3058 | shader: round packed U8 conversions with integer arithmetic | arredondamento de conversões U8 empacotadas sem float | Relaciona-se a `af6d255f` (`v_cvt_pk_u8_f32`) e a D-28 (regra de arredondamento UNORM/SNORM do D3D), mas trata de outro caminho |
| 3059 | docs: record DXC without GVN on WARP and its pass shares | nota de medição do DXC | Não consta |
| 3060 | shader: lower packed byte dot products in direct compute | produtos de ponto de bytes empacotados no frontend nativo | Não consta |

## Pull requests mesclados (50)

Cursor anterior: `2026-11-27T05:02:42Z`. Todos os 50 têm `mergedAt` maior que o cursor (coleta REST
`pulls?state=closed&sort=updated`, não `gh pr list --state merged --search`, que nesta hora devolveu 49 e
perdeu o #2882). Base `main` em 44, `cpp-migration` em 6.

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| 2873 | startup: lower v_lerp_u8 in gfx2hlsl | `bb347ad769` (main, 05:26:30Z) | `tools/startup/gfx2hlsl.py` +17/−1 com `v_lerp_u8`; fixture `startup_gfx2hlsl_lerp_u8_check.py` novo. É o commit `bb347ad769` da tabela acima |
| 2953 | startup: bind compute root SRV buffers at Dispatch | `215934bdf3` (main, 05:26:45Z) | commit `215934bdf3`: ligação de root SRV no Dispatch e `list_upload_tail` |
| 2923 | docs: plan EXEC removal and structurization | `cbae12fd79` (main, 05:40:47Z) | commit `cbae12fd79`, só `docs/experiments/exe-structurization.md` (+223) |
| 2924 | docs: plan slot 57 mesh DXIL translation | `17f27f85e3` (main, 05:41:01Z) | commit `17f27f85e3`, só `docs/experiments/mesh57-dxil-plan.md` (+167) |
| 2921 | tests: model the console BVH node set | `2944b0502b` (main, 05:41:11Z) | commit `2944b0502b`: `bvh_model.py` e +917 em `check.py`, nenhum caminho de produto |
| 2925 | startup: back off the retry for an absent shader cache manifest | `6aa594408c` (main, 05:44:12Z) | commit `6aa594408c`: `manifest_retry.h`, 1 s dobrando até 32 s |
| 2949 | startup: opt-in release of upload ring chunks at a fenced submit | `98e4a2eea9` (main, 06:05:48Z) | commit `98e4a2eea9`: `X2W_D3D12_RING_RELEASE_AT_SUBMIT`, fecha #2948 |
| 2902 | startup: lower compute image_store to 1D and 3D texture UAVs (Part of #2022) | `f52046900a` (main, 06:29:31Z) | commit `f52046900a`: store em `RWTexture3D`/`RWTexture1D` com validação de mip e W |
| 2892 | tests: classify mad() fusion from the thread-dependent mac_f32 words (#2883) | `2917babc58` (main, 06:47:26Z) | fora dos 40 commits documentados; conteúdo não lido nesta hora (só título e sha de merge) |
| 2897 | startup: lower s_bitset0_b64 and s_bitset1_b64 in gfx2hlsl | `c09145a7dd` (main, 06:50:12Z) | fora dos 40; não lido |
| 3024 | startup: retain present signature copies on device removal | `f17a08fd2d` (main, 07:22:11Z) | fora dos 40; não lido. D-01 é sobre a superfície apresentada liberada durante a cópia da assinatura (corrigido em `2a898640`); este PR mexe em retenção de cópias na remoção de dispositivo, mas **NÃO VERIFICADO** se reabre ou amplia D-01 |
| 2865 | docs: record the RX 9070 XT run of the four MinLod bridge entries | `221079324b` (main, 07:40:13Z) | fora dos 40; não lido |
| 2870 | startup: read the literal data directory manifest | `30272027f3` (main, 08:02:00Z) | fora dos 40; não lido |
| 2874 | test: cover slot 35 table selection and invalidation at Dispatch | `3b6c38a83b` (main, 08:02:37Z) | fora dos 40; não lido |
| 2884 | textures: harden bool/integer and byte-type bounds in texture_blocks with standalone unit suite | `0e7044687d` (main, 08:34:50Z) | fora dos 40; não lido. Título diz validação de entradas em `texture_blocks` |
| 2887 | cpu: lower MOVNTPS, MOVNTPD, MOVNTDQ and MOVNTI in the shared lowering | `3e9542a65c` (main, 08:48:41Z) | fora dos 40; não lido |
| 2922 | startup: lower the FP32 box case of image_bvh_intersect_ray | `0d2285804f` (main, 09:02:01Z) | fora dos 40; não lido |
| 2875 | startup: lower v_perm_b32 in gfx2hlsl | `e446b07268` (main, 09:04:05Z) | fora dos 40; não lido |
| 3025 | native: read the byte-address rewrite of a GPU-selected heap descriptor | `1507f0e290` (cpp-migration, 09:29:20Z) | fora da faixa da `main`; o merge não entra no cursor de commits, que segue a default |
| 3020 | shader: keep D[31:16] in the direct path's binary16 add, sub, subrev and mul | `f6cdee1068` (cpp-migration, 09:36:38Z) | fora da faixa da `main`. Título indica correção da metade alta que `8946fb16` zera por contrato — **divergência a conferir quando cpp-migration for mesclada; NÃO VERIFICADO** |
| 3035 | startup: lower v_dot8_i32_i4 and v_dot8_u32_u4 in gfx2hlsl | `8a901eb2ad` (main, 09:39:35Z) | fora dos 40; não lido |
| 2893 | cpu: harden bounds and alignment checks in cpu_ranges with standalone PE32+ unit suite | `805d7df719` (main, 09:55:57Z) | fora dos 40; não lido |
| 2888 | ffx: harden bounds and stage validation in ffx_scratch_reference with standalone unit suite | `8bcbc3c86a` (main, 09:57:12Z) | fora dos 40; não lido |
| 2886 | shader: harden atomic output restoration and symlink guards in dxc_cache with CTest unit suite | `41aaf197d1` (main, 09:57:46Z) | fora dos 40; não lido |
| 2878 | ffx: harden atomic output writes, exit propagation, and path guards in ffx_create_context with CLI suite | `a385a82229` (main, 09:59:18Z) | fora dos 40; não lido |
| 2895 | gpu: harden atomic manifest writes and path validation in record_gpu_cases with mocked CLI suite | `15cacec213` (main, 10:00:56Z) | fora dos 40; não lido |
| 2866 | shader: extend uniform compute image_load proof to image_load_mip | `4361a1911b` (main, 10:04:53Z) | fora dos 40; não lido |
| 2910 | startup: refuse compute buffer loads through a modified descriptor (#2901) | `5b70e0015e` (main, 10:08:59Z) | fora dos 40; não lido. Título indica endurecimento (recusa), não correção de defeito catalogado |
| 3039 | Integrate the native C++ shader translator and resource binding updates | `91cbedd69a` (main, 10:13:33Z) | merge de `cpp-migration` na `main`; o diff é o próprio frontend nativo que `4036577020` introduziu, agora na default |
| 3034 | native: read formatted loads through a byte-address rewritten descriptor | `b405cf719d` (cpp-migration, 10:16:00Z) | fora da faixa da `main` |
| 2899 | startup: accelerate gfx2hlsl cache scans and support skipping earlier failures | `98007de5de` (main, 10:17:16Z) | fora dos 40; não lido |
| 2920 | startup: gate the BVH builder compute stages all or none | `e04217f4f2` (main, 10:59:34Z) | fora dos 40; não lido |
| 3042 | native: read one-channel typed loads through a GPU-selected heap descriptor | `a0f6773135` (cpp-migration, 11:01:23Z) | fora da faixa da `main` |
| 3046 | startup: lower v_pk_add/sub_i16 and v_pk_add/sub_u16 in gfx2hlsl | `dd46a90c42` (main, 11:26:39Z) | fora dos 40; não lido |
| 3045 | startup: lower v_pk_lshlrev_b16, v_pk_lshrrev_b16 and v_pk_ashrrev_i16 in gfx2hlsl | `3b4c4f00fc` (main, 11:27:17Z) | fora dos 40; não lido |
| 3047 | startup: lower v_pk_mul_lo_u16, v_pk_mad_u16 and v_pk_mad_i16 in gfx2hlsl | `67e60e8fa6` (main, 11:27:53Z) | fora dos 40; não lido |
| 3040 | shader: correct FP32 BVH descriptor byte units and child results | `1c879506bc` (main, 11:29:45Z) | fora dos 40; não lido. Título indica correção de unidades e de resultado de filho no BVH FP32 |
| 3041 | startup: lower v_mad_i32_i16 in gfx2hlsl | `703e66d800` (main, 11:47:21Z) | fora dos 40; não lido |
| 3043 | startup: lower v_add_nc_i32 and v_sub_nc_i32 in gfx2hlsl | `b42bc32a3d` (main, 11:48:47Z) | fora dos 40; não lido |
| 3044 | startup: lower v_pk_min/max_i16 and v_pk_min/max_u16 in gfx2hlsl | `04b916e7ef` (main, 11:48:59Z) | fora dos 40; não lido |
| 2900 | fix(shader): prove scalar runs before compute EXEC restores | `298a203426` (main, 11:49:13Z) | fora dos 40; não lido |
| 3051 | shader: lower 32-bit bit counts and scans in the direct compute path | `eff2dd7a06` (main, 11:53:21Z) | fora dos 40; não lido |
| 3048 | shader: lower three-source 16-bit integer VOP3 ops in the direct C++ compute path | `77e7a9779c` (main, 11:54:21Z) | fora dos 40; não lido |
| 3018 | shader: lower the non-packed 16-bit integer VOP3 ops in the direct C++ compute path | `35f7197e9e` (main, 11:55:09Z) | fora dos 40; não lido |
| 3053 | tests: compare SDWA binary16 halves without the kept high half | `d645767903` (main, 12:06:43Z) | fora dos 40; não lido. Título sugere que o teste de `7bcff286` foi ajustado à manutenção da metade alta por `f6cdee10` (cpp-migration) — **NÃO VERIFICADO** |
| 3010 | shader: lower DPP16 row_shl, row_shr and row_ror in direct compute | `1ec2c938b9` (main, 12:11:45Z) | fora dos 40; não lido |
| 3029 | startup: lower s_brev_b64 in gfx2hlsl | `232ae71e8d` (main, 12:27:21Z) | fora dos 40; não lido |
| 3016 | shader: lower v_perm_b32 and v_lerp_u8 in the direct C++ compute path | `fca7c35bcf` (main, 12:48:44Z) | fora dos 40; era o head da `main` na coleta de commits |
| 3055 | shader: lower s_absdiff_i32 in the direct compute path | `801f4c3bea` (main, 13:06:51Z) | fora dos 40; merge pai do head `801f4c3b` usado como `head_at_run` |
| 2882 | gpu: verify translated compute writes to two mips of one texture | `db28a79627` (main, 13:14:52Z) | título e branch (`test/translated-compute-two-mips`) lidos na API; o merge entrou na coleta REST mas **não** no compare de commits, que fechou em `801f4c3b`. Diff **NÃO VERIFICADO** nesta hora |

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum dos 40 commits documentados fecha um defeito catalogado de `Xbox2Windows`
  (D-01 a D-29). O mais próximo é `e97c5c7ce9` (#2932), que contorna o crash do `amdxc64` ao criar
  pipeline por falta de `!llvm.ident` — não há entrada no CSV para isso, e o defeito é do driver AMD, não
  do port.
- **Abre defeito novo?** Dois candidatos, ambos sem verificação em GPU:
  - `7beb331d04`: até esse commit, todo pedido publicado na revisão 28 do runtime era recusado por
    `compile_dxil` como key mismatch (`"x2w-startup-shader-14"` contra `-28`). É defeito introduzido e
    corrigido na mesma faixa; nenhum título real foi medido depois do conserto. **NAO VERIFICADO.**
  - `7f951fd3de`: a reversão tira de `cpp-migration` os 17 commits de XMem de #2911 (2.544 linhas,
    `src/xmem_mapped_pages.cpp` e cinco fixtures). Se `cpp-migration` for mesclada antes de #2911 entrar na
    `main`, a branch entrega o compilador nativo sem os consertos de run de página. Não é defeito, é
    divergência de branch com risco de regressão no merge.
- `4036577020` adiciona 15.618 linhas de frontend em um único commit, sem tocar código existente, o que
  dificulta atribuir regressão futura a um ponto; o `#3039` (`91cbedd69a`) já o traz para a `main` como
  merge.

## Confiança

- **Verificado pelo diff**: os 40 commits da tabela, um a um, pelo endpoint `commits/<sha>` (patches,
  estatísticas e nomes de arquivo). As afirmações de linha citam arquivo e mecanismo vistos no patch.
- **Verificado por metadado**: os 50 merges (número, `mergedAt`, `base.ref`, `merge_commit_sha`) pela API
  REST de pulls fechados, ordenada por `updated` descendente e paginada até passar do cursor. A listagem
  `gh pr list --state merged --search "merged:>..."` devolveu 49 e não viu o #2882 — divergência
  registrada, e por isso a coleta de merges desta hora não usa essa via.
- **Não verificado**: o conteúdo dos 40 commits acima de `f52046900a` (teto de 40), o diff dos 42 merges
  que caíram fora dessa faixa de commits, e o #2882 (merge posterior ao compare). Para eles só existem
  número, título e sha de merge. `NÃO VERIFICADO` aparece nas linhas correspondentes.
- **Contagem de commits**: `ahead_by` do endpoint `compare` (`e62e3a5e...801f4c3b`), não a listagem
  `commits?sha=`, que ordena por data de commit e esconde commits de pré-merge com data anterior ao
  cursor.
- **Limite de 40**: o teto desta execução foi atingido. `commit_cursor` fica em `f52046900a` e
  `head_at_run` em `801f4c3b`; a próxima hora continua de `f52046900a`, inclusive o #3055 (`801f4c3b`) e o
  #2882 (`db28a796`), que não tiveram diff lido aqui.
