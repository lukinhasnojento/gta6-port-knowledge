# R0Y00/Xbox2Windows — atualização de 2026-11-25 23:00

## Resumo

Quatorze commits novos na `main`, incluindo dois merges (`#2662`, `#2663`). O grosso é a tradução
`gfx2hlsl` ganhando formas novas (`s_flbit_i32_b32`, carry de `s_add_u32` adjacente, `s_buffer_load`
dentro de bloco EXEC-zero do slot 57, pixel stage depth-only) e a camada `d3d12_xs` deixando de
derrubar o device em descrições de view inválidas e em falha de alocação de wrapper. Dois commits
grandes mudam o apresentador (planos múltiplos, óptico) e o perfil de console (leitura de
`ConsoleMode` do registro).

## Commits novos (14)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `572723f9` | SimSalabimse | shader: lower s_flbit_i32_b32 in gfx2hlsl (Part of #1723) (#2641) | `tools/startup/gfx2hlsl.py`: novo `op_s_flbit_i32_b32` que emite `S0 == 0 ? 0xffffffff : 31 - firstbithigh(S0)` e não toca SCC; recusa `mods`, destino fora de `s…s105`, lane masks e SGPR de descritor. `tests/startup_gfx2hlsl_s_flbit_check.py` (229 linhas) avalia o texto traduzido na CPU e, com `dispatch`, compila e compara UAV em WARP/hardware; CMake registra 3 testes |
| `d26cbc54` | NiiRoZz | startup: translate slot 57 mesh requests with a depth-only pixel stage (#2606) | `translate_mesh57` passa a coletar `targets` de todos os `exp`; antes `null_only` levantava `Unsupported('…depth-only…is not covered')`, agora é aceito e só a mistura `null` + outro export é recusada. Novo `tests/startup_mesh57_depth_only_check.py` (78 linhas) exige pixel sem `SV_Target`, o `discard` preservado e `targets` vazio no manifesto; `startup_mesh57_skeleton_check.py` muda a expectativa de 'unsupported' para 'ok' |
| `b2564017` | pigalot | runtime: give unknown-dimension views a default view and skip mismatched ones (#2649) | Novo `src/startup_stubs/view_dimension.h` (105 linhas, puro): `needed(slot, view_dimension)` mapeia slot 18/19/20/21 para dimensão de recurso. Em `d3d12_xs_device.cpp`, SRV (slot 18) com `ViewDimension` 0 recebe a view default do recurso (descrição nula) e view de dimensão incompatível é recusada com log em vez de remover o device. Testes `startup_view_dimension_check.cpp` (122) e `startup_view_dimension_device.cpp` (425) com controle `X2W_D3D12_VIEW_DEFAULTS=off` |
| `63ba73a9` | thepatrickworld | startup: give the placed page aliases and the draw dumps the teardown boundary (#2631) | Quatro `static` locais de função que possuem objetos D3D12 do desktop são destruídos no teardown da CRT, depois de o driver de vídeo ser desanexado (`#2590` corrigiu dois). `d3d12_teardown.h` (+60/-3) e `draw_dump_tracking.h` (+62) ganham fronteira explícita; `tests/startup_teardown_unload.cpp` cresce 432 linhas |
| `c99b7b89` | pigalot | runtime: composite the presented planes in the output window (#2647) | `present_planes.h`: registro de plano medido em 0x48 bytes (`+0x00` token de frame, `+0x08` contagem de recursos, `+0x10` ponteiro do array); `plane_compositor.h` desenha de baixo para cima com `SRC_ALPHA/INV_SRC_ALPHA`. Padrão continua mostrando só o plano 0; `X2W_PRESENT_COMPOSITE=1` compõe todos. Medição registrada: na Series S, plano 0 1920x1080 preto e plano 1 3840x2160 com a imagem; por que o plano 0 fica preto é declarado desconhecido |
| `82c70807` | pigalot | runtime: add an opt-in console profile for the OEM ConsoleMode read (#2650) | Novo `console_mode.h`/`registry_console_mode.inl`: com `X2W_RUNTIME_PROXY` e `X2W_DEVICE_TYPE`, intercepta `RegOpenKeyExW`/`RegQueryValueExW` de `HKLM\Software\Microsoft\Windows NT\CurrentVersion\OEM\ConsoleMode` e responde 6→`0x60`, 7→`0x64`, 1→valor ausente; sem a variável, tudo vai ao registro real. `runtime_gpa_patch.h` ganha `Kind::registry`; o teste com DLL-isca exige que export de mesmo nome não seja hookado |
| `72879797` | triggered02 | startup: refuse modifiers on modifier-free handlers (#1778) | 12 linhas em `gfx2hlsl.py` recusam modificador em handler que não o aceita, em vez de ignorá-lo; `tests/startup_gfx2hlsl_modifier_free_check.py` (179 linhas) cobre cada recusa. Corpo diz **Fixes #1771** |
| `f06646ba` | SimSalabimse | startup: accept entry-prefix DPP broadcasts when a later loop exists (#2657) | O broadcast `quad_perm:[k,k,k,k]` deixa de ser recusado só porque existe loop uniforme posterior: a prova (prefixo reto com `s_wqm` e EXEC intacto) sobrevive a loop/branch à frente, e continua recusada depois de alvo de branch, de mudança de EXEC ou com EXEC divergente. Teste ganha fixture `loop` com duas voltas |
| `31bcd9ad` | RavenCurseBurg | tests: make explicit gradients select the mip above a MinLod floor (#2663) | `tests/spirv_window/minlod.py` e `minlod.spvasm`: novo caso `grad-above-clamp` com gradientes de dois texels numa textura 8x8, que seleciona mip 1 com piso 0. +4/-0, só teste |
| `edbc8b5e` | SimSalabimse | compute: accept gfx1013 image_atomic_umin on R32_UINT (#2662) | `compute_store_aot.py`: a lista de operações que o gfx1013 herda do lowering gfx1030 passa de `add, umax, cmpswap` para `add, umin, umax, cmpswap`; o fixture do gfx1013 usa operando `0x80000011` para separar `umin` de `smin` em vários texels |
| `f4c6998a` | Leyukaka | startup: lower one slot 57 scalar load in a pre-exchange execz block (#2660) | `mesh_skeleton` ganha o parâmetro `wave_loads`: um único `s_buffer_load_dword` dentro de bloco `s_cbranch_execz` para a frente, com resultado vivo no join, é aceito quando EXEC vem de `s_mov_b64 exec, -1` + `v_cmpx_gt_u32_e32` de limite SGPR uniforme; o tradutor emite `(tid & ~63u) < bound`. Recusa outro escalar no bloco, carga mais larga, endereço não uniforme e valor anterior indefinido. +82/-5 em `gfx2hlsl.py`, 125 linhas de teste |
| `341215a9` | saii | startup: return E_OUTOFMEMORY when a D3D12 wrapper cannot be allocated (#2656) | `wrap()` usava o retorno de `_aligned_malloc` sem checagem — falha de alocação virava escrita por ponteiro nulo. Agora devolve `nullptr` e o slot libera o objeto real e retorna `E_OUTOFMEMORY` com saída nula; export só-de-fixture `x2w_d3d12_fail_wraps` arma a falha. Teste cobre criação de device e slots 8, 9, 14, 16, 55, 73, 127, e `GetDevice` de fila viva depois de um create falho. Issue referida: **#2653** |
| `7ff6f3b9` | yjs | shader: lower experimental gfx1030 2D sample clamps (#1638) | `tools/pixel_aot.py` +56/-8 implementam os clamps de amostragem 2D do gfx1030; `tests/pixel_sample_clamp_check.py` novo com 275 linhas; `docs/experiments.md` +80 registra o que foi medido. Marcado como experimental no próprio assunto |
| `d76a954c` | RavenCurseBurg | gfx2hlsl: lower the carry of adjacent compute address adds (#2658) | Novo `adjacent_address_carries()` (36 linhas) reconhece `s_add_u32 sN, a, b` seguido de `s_add_u32 sN+1, 0, src_scc` no mesmo bloco básico — o segundo guarda o carry como word alta e limpa SCC. Branch direto para o add alto, addend de lane-mask ou aritmética simbólica de descritor continuam recusados. Fixture `address-carry` soma IDs de grupo 0..3 a `0xfffffffe` e lê as duas words e o SCC final |

Ordem cronológica acima (mais antigo primeiro), como devolvida pela API em ordem inversa.

## Pull requests abertos (24)

Propostas conforme o corpo do PR; nenhum diff foi lido.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 2661 | startup: pixel image_store sparse DMASK via read-modify-write | Store com DMASK parcial no pixel stage via RMW, com init e readback por canal (item I de #2618) | Não consta no catálogo |
| 2664 | shader: harden spirv_address_window CLI guards and add CLI suite | Validar argumentos/caminhos antes de abrir arquivo, com suíte de CLI mockada | Não |
| 2665 | textures: validate archive inputs and lookup keys upfront in texture_descriptors CLI | Validar caminhos e limites de `--lookup-key` antes de abrir; suíte RDS5/ETPF sintética | Não |
| 2666 | pe: harden call_evidence CLI input/output guards and atomic writes | `_write_atomic` e validação de caminhos em `tools/call_evidence.py` | Não; mesma classe de D-08 (CLI que ignora `--output`/sai 0 em alvo ausente), arquivo diferente |
| 2667 | cpu: normalize Windows compiler basename in EXE CLI reports | `PureWindowsPath` para o basename do compilador, refs #2605 | Não |
| 2668 | cpu: lift BMI1 ANDN, BEXTR, and TZCNT leaf functions to C++ | Lifter AOT `tools/cpu_bmi1_logic_aot.py` + suíte MASM/CTest diferencial | Não |
| 2669 | startup: copy descriptor words into vcc as the upper half of s[104:107] | Item L de #2023: 41 das 43 ocorrências de "mask from descriptor data" copiam para vcc | Não consta; é continuação do `s_load` de descritor em `s[104:107]` de #2057 |
| 2670 | tooling: harden bink_descriptors_aot CLI path guards and atomic writes | Validação de argumentos e escrita atômica + suíte PE32+ sintética | Não |
| 2671 | cpu: add AVX VEX FP compare AOT lifter and test suite | Lifter das 4 famílias de compare FP AVX VEX nos 32 predicados (imm8 & 0x1F) | Não |
| 2672 | runtime: harden startup triage_run CLI guards and add CLI suite | Validação em `_validate_args` e suíte de log/cache sintética | Vizinho de D-13 (endpoint de triagem contando step-over como parada) |
| 2673 | tooling: harden fuzz_cpu CLI path/count guards and atomic writes | Guards em `emit`/`translate`/`check` e escrita atômica | Não |
| 2674 | shader: lower s_ff0_i32_b32 in gfx2hlsl (#1723) | `D = firstbitlow(~S0)`, `0xffffffff` quando todos os bits estão postos; SCC inalterado | Não; mesma issue #1723 dos commits de hoje |
| 2676 | textures: harden paged_resource validation and add unit suite | Rejeitar `bool`/não-`int` e demais entradas inválidas em `tools/paged_resource.py` | Não |
| 2677 | cpu: lift legacy SSE and VEX.128 AES-NI and PCLMULQDQ leaf functions to C++ | Lifter AOT + suíte diferencial para `AESENC`/`AESDEC`/`PCLMULQDQ` | Não |
| 2678 | shader: lower s_quadmask_b64 in gfx2hlsl (#1723) | `D.lo` = OR de cada grupo de 4 bits do fonte de 64 bits; `D.hi` = 0; SCC = `(D.lo | D.hi) != 0` | Não |
| 2679 | d3d12: keep live SRVs intact during resource retirement | Corrige a reescrita opt-in de descritores de #2579 e preserva o rebind de #2468 | Área de D-15/D-17 (heap e liberação de descritor); não é o mesmo defeito |
| 2680 | tests: draw a slot 57 depth-only pair into D32 and compare depth with the CPU | Acrescenta só o desenho que `#2606` deixa de fora; não muda o tradutor | Não |
| 2681 | cpu: lift x87 environment and state save/restore leaf functions to C++ | Lifter dos 7 mnemônicos de ambiente x87 não cobertos | Não |
| 2682 | tooling: harden hardware_reports CLI guards and add CLI suite | Validação de caminho/flag e de comment-ID | Não |
| 2683 | startup: opt-in split of a batched indirect Dispatch (Refs #2626) | Contorno opt-in no slot 61 para o `ExecuteIndirect` de Dispatch da RDNA 1, que roda só o primeiro comando | Próximo de D-15 (assinatura de comando); defeito diferente, ligado à issue #2626 |
| 2685 | startup: probe DirectStorage destinations instead of walking them with VirtualQuery | Trocar `VirtualQuery` por sonda de um byte por página de 4 KiB com OR atômico sob SEH; ~10 ms de kernel por request | **Corrige #2684**; o catálogo não tem linha para isso (VItoriaBr D-01 é a mesma classe de custo de `VirtualQuery`, outro repo) |
| 2686 | pe: validate intake_exe CLI build, candidate, metadata, and report paths upfront | Validação antes de `collect()` + suíte CLI | Não |
| 2687 | docs: create the reusable Vulkan host output directory | Criar `out/vk` antes de compilar no guia não-Windows | Não |
| 2688 | runtime: harden bink_frame_aot CLI with atomic writes and add CLI suite | Validação de caminho e escrita atômica | Não |

## Pull requests mesclados (2)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| 2662 | compute: accept gfx1013 image_atomic_umin on R32_UINT | `edbc8b5e` | Squash de um commit: `umin` entra na lista de atômicos de imagem que o gfx1013 herda do gfx1030 (4 arquivos, +46/-30). O PR prometia exatamente isso; o diff confirma, incluindo o operando `0x80000011` no fixture |
| 2663 | tests: make explicit gradients select the mip above a MinLod floor | `31bcd9ad` | Squash de +4/-0: um caso novo em `minlod.py` e dois `OpConstantComposite` em `minlod.spvasm`. O PR prometia um caso de gradiente acima do piso de MinLod; o diff é só isso |

## Cruzamento com o catálogo

- **Corrige algum D-xx?** Nenhum commit desta faixa casa com hash do catálogo `Xbox2Windows`
  (D-01 `2a898640` … D-29 `22989d61`). O que os commits corrigem são issues próprias: `#2653`
  (`341215a9`), `#1771` (`72879797`), `#2649`/`#2647`/`#2650`/`#2631`/`#2658`/`#2660`/`#2662`/`#2663`/
  `#2641`/`#2606`/`#1778`/`#2657`/`#1638` referenciadas nos assuntos.
- **Abre defeito novo?** Dois candidatos, ambos declarados pelo próprio autor:
  - `b2564017`: antes, uma view com `ViewDimension` 0 ou dimensão incompatível **removia o device**;
    a correção só cobre o caso SRV medido e deixa as outras views "skipped" com log. Se algum título
    depender de RTV/UAV/depth com dimensão 0, a view some em silêncio — comportamento novo, medido em
    um único caso.
  - `63ba73a9`: a correção depende de ordem de destruição de `static` locais de função. O próprio
    commit diz que quatro desses existem; nenhum teste prova que não sobrou um quinto.
- **Fica em aberto no catálogo:** D-05, D-19, D-24, D-25 e o sintoma de D-27 continuam marcados como
  PROVÁVEL no CSV; nada desta faixa os toca.

## Confiança

- **Verificado:** os 14 commits e seus diffs (patch da API por commit, com lista de arquivos,
  adições e remoções); os dois merges por `mergedAt` + `mergeCommit.oid`.
- **Verificado:** os 24 PRs abertos, números, títulos, autores.
- **NAO VERIFICADO:** o conteúdo dos 24 diffs de PR; as colunas "Proposta" e "Cobre defeito
  conhecido?" são declaração do autor.
- **NAO VERIFICADO:** se os testes citados passam — nenhum `ctest` foi executado, e o repositório é
  somente leitura para esta rotina.
