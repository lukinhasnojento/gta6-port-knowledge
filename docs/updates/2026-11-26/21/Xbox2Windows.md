# R0Y00/Xbox2Windows — atualização de 2026-11-26 21:00

## Resumo

**40 de 72 commits** na faixa `da2a8f6f..6ee878d8` (teto de 40; `commit_cursor` vai a `1983228b`, 32
ficam para o próximo ciclo). A hora é quase toda do tradutor `gfx2hlsl` fechando formas que o validador
DXIL recusava: `vcc_hi` como escalar em wave32, `BUF_FMT_2_10_10_10_UNORM`, `image_store` em array 2D,
`v_interp_p1ll/p2_f16` com laço, spill de descritor em lane de wave64, SCC de `s_add/s_sub/s_min/s_max` e
o sinal do zero em produto binary16. O cache de shader foi de `x2w-startup-shader-19` a `-27` dentro da
hora — cada correção invalida o anterior. No runtime, o *retire class* passa a contar slots de UAV/RTV/DSV
(`33004fa1`), o heap APU de áudio commita página sob demanda (`24be1786`) e o `xcrdapi` (DirectStorage)
deixa de vazar objetos COM (`f9f4976b`). 57 PRs mesclados na janela, todos *squash* e todos dentro da
faixa de commits.

## Commits novos (40)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| e3cf6a91 | tedin7 | startup: treat wave32 vcc_hi as an ordinary scalar register in gfx2hlsl (Part of #1723) (#2703) | `tools/startup/gfx2hlsl.py` +76, teste novo (+237), `CMakeLists.txt` +10 (+332/−1): em wave32 só `vcc_lo` é máscara e o compilador aloca `vcc_hi` como escalar comum; o tradutor passa a renomear `vcc_hi` para um SGPR que o programa não nomeia quando isso é exato nos dois tamanhos de wave — nenhuma instrução nomeia o par `vcc`, e ou um compare/carry-out escreve `vcc_hi` (ou SGPR ímpar), ou uma máscara VOP3 lê um, ou nenhum VALU e nenhum branch `vccz`/`vccnz` usa `vcc_lo` como máscara. Escrita escalar em `vcc_hi`/`exec_hi` vira recusa nomeada em vez de `TypeError`. Desenhos 16×16 em WARP e hardware contra referência de CPU. |
| f0101e79 | Royoo | tests: follow the shared word3 decoder in draw preflight (#2762) | `tests/startup_vertex_fetch_formats_check.py` (+11/−2): o controle passa a procurar `formatted_buffer_word3(` em vez de `formatted_buffer_supported(` no decodificador compartilhado, e a mutação que remove a checagem de FORMAT tem de reprovar o controle. Só teste. |
| 41cd090b | tedin7 | shader: support BUF_FMT_2_10_10_10_UNORM fetch format in gfx2hlsl (Part of #1723) (#2006) | 19 arquivos (+180/−24): `BUF_FMT_2_10_10_10_UNORM` (formato gfx10 50, DXGI `R10G10B10A2_UNORM` 24) decodifica um dword com x, y, z em 9:0/19:10/29:20 (÷1023) e w em 31:30 (÷3), o layout de 10 bits no x que o Mesa usa para R10G10B10A2; `BUF_FMT_10_10_10_2_UNORM` (44) fica de fora. Revisão de cache `x2w-startup-shader-20`. Teste Python + tabela DXGI + leituras em `startup_vertex_fetch_formats_{warp,hardware}` com stride 6/offset 2. |
| 33004fa1 | triggered02 | startup: track UAV/RTV/DSV descriptor slots in the retire class (slice 3a) (#2719) | `d3d12_capture.inl` +56, `d3d12_xs_device.cpp` +38, teste +214, docs (+373/−54): com `X2W_D3D12_RELEASE_RETIRED=1`, os slots 19 (UAV), 20 (RTV) e 21 (DSV) passam a contar referência como o slot 18 já fazia — um slot guarda uma referência e um UAV 19 guarda outra para o recurso de contador. `note_view` adiciona na escrita e libera a anterior; `note_copy` move; o `Release` do heap libera o que sobrou. Slot não sombreado mantém o `retire_leave_class`. Antes um slot 19–21 tirava o recurso da classe incondicionalmente e uma textura nunca era liberada. Casos 8 e 9 em WARP e GTX 1660 SUPER, com quatro mutantes que reprovam o fixture. |
| 6a6eecfe | tedin7 | startup: keep a descriptor loaded in an execz arm for reads before the exec restore in gfx2hlsl (#2713) | `gfx2hlsl.py` +63, teste +77: num join para o qual só arestas `s_cbranch_execz` pulam, a palavra de descritor escrita pelo braço de fall-through é mantida até `exec` ser reescrito, um branch/join ser alcançado ou uma instrução escalar lê-la (as lanes que pularam têm `exec == 0` ali). Leitura depois do restore continua recusada; um reload entre o join e o restore é definição nova. Três fixtures wave64 (mascarado, leitura escalar no join, reload). |
| 24be1786 | triggered02 | acphal: reserve APU heap and commit/decommit pages on demand (#2767) | `acphal.cpp` +34, teste +176, docs (+225/−26): com `X2W_AUDIO_PERMISSIVE=1`, `ApuCreateHeap` só reserva (`MEM_RESERVE`, 0 commitado) e `ApuAlloc` commita a faixa de 4 KiB que cobre o bloco, desfazendo a mudança na free list e devolvendo `E_OUTOFMEMORY` se o commit falhar; `ApuFree` devolve, funde vizinhos e decommita só páginas 4 KiB **interiores** (`[ceil_page(begin), floor_page(end))`), preservando as parciais compartilhadas. Antes um título que aloca/libera em fluxo esgotava o commit do heap. Teste com 4 threads × 1 000 rodadas e `VirtualQuery`/`PagefileUsage`. |
| cd1ff54d | NiiRoZz | startup: bind a compute input behind a memory view token in its own resource (#2625) | `d3d12_draw.inl` +12, testes (+40/−3): quando o buffer está num heap custom fora do índice de placement (o caminho de memory view coloca no heap do próprio título), `own_memory` é verdadeiro e o offset da view é vinculado **no recurso desktop do próprio token** em vez de `base + offset`. Novo caso `dispatch-input-memory-view`: input atrás de token de buffer view (slot 70), 16 bytes adentro, 320 palavras conferidas. |
| 9e5efae4 | rdx | gpu: add D3D12 meshlet amplification and mesh shader pipeline with vertex fallback (#2462) | 10 arquivos novos (+1 602): `mesh_shader_pipeline.{h,cpp}` (385 linhas), `meshlet_builder.{h,cpp}` (309), `shaders/meshlet_pipeline.hlsl` (154) e testes. Pipeline com estágios `as_6_5`/`ms_6_5`/`vs_6_0`/`ps_6_0` compilados pelo DXC no build, com fallback de vértice quando o adaptador não tem mesh shader; o builder valida tamanho de meshlet contra o limite de saída do shader e recusa `max_vertices < 3`. Testes `mesh_shader_pipeline_{hardware,warp}` e `..._fallback_...`. |
| 756e4bb4 | RavenCurseBurg | Prove uniform shifts of masked and extracted compute thread IDs (#2769) | `gfx2hlsl.py` +34, testes (+104/−13): no bloco de entrada de compute, o tradutor limita os bits do `tid.x` intocado pelo tamanho do grupo local; para `((tid.x & mask) \| scalar) >> n` o resultado é `scalar >> n` quando todo bit selecionado é deslocado fora, e para `v_bfe_u32` com offset/largura literais a extração é aplicada antes ao conjunto de bits possível. Fixtures `masked-tid` e `extracted-tid` leem o resultado após um branch. |
| b457f7ff | yjs | shader: lower gfx1030 comparison sample gradients (#2748) | `tools/pixel_aot.py` +47, teste novo (+256), docs +45 (+394/−47): `image_sample_c_d`, `_c_d_o`, `_c_d_g16` e `_c_d_o_g16` viram `SampleCmpGrad` para view 2D `R32_FLOAT` com dois mips visíveis e `dmask:0x1`; endereço contíguo = offset XY assinado opcional, referência de comparação, gradientes em `(du/dx, dv/dx, du/dy, dv/dy)` e UV de largura cheia, com G16 empacotando dois binary16 (U no low, V no high). Teste de 8×8 e 16×16 contra referência de CPU; encosta em mip 1 com inclinação −1/4 em V e +1/4 em U. |
| 55cbbc43 | rdx | shader: lower mesh param0..param12 exports and gfx2hlsl pixel linkage (#2427) | `tools/mesh_param_link.py` novo (+142), teste novo (+444), wiring em `mesh_entry_aot.py`, `mesh_dynamic_aot.py` e `gfx2hlsl.py` (+718/−32): liga exportações `param0..param12` (EXP alvo 32+N, limite de 32 vetores do D3D12) ao estágio pixel, com `PARAM0..12` densos, cópias flat `FPARAM*` (`nointerpolation`), metades binary16 `HPARAML*/HPARAMH*`, pareamento de zero parâmetro só-profundidade e TEXCOORD0 legado para quem só usa `param0`; recusa `param13+`, export incompleto, assinatura de variante divergente e >32 vetores. Desenhos `ms_6_5` + `ps_6_0`. |
| f1a8db63 | tedin7 | gfx2hlsl: accept mask-preserving lane-mask writes in an execz block (#2723) | `gfx2hlsl.py` +119, testes +136, docs (+285/−48): `phi_mask_ok` passa a aceitar três escritas de máscara que preservam bits de lane: `s_andn2_b32 M, M, exec_lo` e `s_or_b32 M, M, T` (T escrita no bloco por `v_cmp` ou `s_and` de exec), um `v_cmp` em M quando `s_mov_b32 M, 0` é a última escrita antes do branch, e `s_orn2_b32 M, X, exec_lo` quando `M` era todo -1. Zero e -1 não se misturam; outras formas mantêm a recusa. Revisão `x2w-startup-shader-21` recusa uma forma que a 20 traduzia (`s_and_b32 M, X, exec_lo` com X dado). Casos P/Q com dispatch de wave parcial (3, 6 e todas as lanes). |
| 6fb06d95 | rdx | shader: lower s_bitcmp0/1_b32 and b64 in bounded mesh lowerers (#2444) | `mesh_entry_aot.py` +30, `mesh_dynamic_aot.py` +8, teste novo (+357), docs (+414/−10): implementa `s_bitcmp0_b32/bitcmp1_b32` (`SCC = ((S0 >> (S1 & 31)) & 1) == 0/1`) e as formas `_b64` (`S1 & 63`), com teste de bit de palavra de constant buffer seguido de branch SCC que forma join de descritor verificado. Descritor, operando não escrito e predicado dinâmico de 64 bits continuam recusados. |
| f30a9e3a | triggered02 | xmem: lazily reserve below-40 startup blocks and reclaim empty slabs (#2770) | `xmem_mapped_pages.cpp` +51, `xmem_slab_slots.h` +56, testes (+255/−63): blocos ≥ 32 MiB colocados abaixo de 2^40 por `X2W_XMEM_STARTUP_BELOW_40` também passam a ser reservados de forma preguiçosa com `X2W_XMEM_LAZY_STARTUP=1`; quando todos os slots de um slab são liberados e despinados, `reclaim_empty_slabs` fecha o slab e devolve o commit ao SO. `unpin_slot` coleta os slabs fechados; slot 133 commita o bloco inteiro antes de colocar recurso de GPU. |
| 6427564b | RavenCurseBurg | startup: refuse uniform load proofs across EXEC-changing loops (#2768) | `gfx2hlsl.py` +22, teste +26, docs (+87/−37): a prova de load uniforme de compute passa a seguir as arestas de fluxo a partir de escritores de EXEC, inclusive desvios fora da faixa do laço: um laço que reencontra o load depois de mudar EXEC invalida a prova (a máscara estreitada pode ficar vazia, e a RDNA 2 lê o VGPR anterior da lane 0, o que um reload sem guarda não reproduz). Revisões `x2w-startup-shader-22`/`-22cs`, avançadas juntas. |
| cd5fb38b | RavenCurseBurg | startup: run a GPU-written indirect Dispatch on a translated slot 67 pipeline (#2771) | 10 arquivos (+396/−42): para pipeline slot 67 traduzido com buffers crus de tabela fixa, o slot 61 aceita um registro de `Dispatch` sem root em argument buffer colocado — vincula o mesmo pipeline e descritores do dispatch direto, transiciona o argument buffer e grava `ExecuteIndirect` desktop; a GPU lê as contagens de grupo. Fixture com produtor compute, cópia do argumento e consumidor indireto na mesma lista; contagens 2 e 5 escrevem exatamente N palavras e batem com o dispatch direto. |
| 475bb730 | TheVoid | startup: keep the object receiver for 8-byte struct returns when rdx holds a wrapper (#2774) | `d3d12_xs_device.cpp` (+9/−8), teste +43: `receiver(a, b, …)` decide qual argumento é o objeto — um wrapper no **primeiro** registro é o objeto mesmo quando o chamador deixou outro wrapper vivo no segundo. Antes, uma chamada de slot 9 (`GetCPUDescriptorHandleForHeapStart`) que passa só `this` em rcx e lê rax caía na forma (buffer de retorno, objeto) e escrevia o handle por cima da vtable do heap. Caso 10 do fixture com um segundo heap vivo. |
| 624f5927 | triggered02 | shader: lower image_store on SQ_RSRC_IMG_2D_ARRAY in gfx2hlsl (#2744) | `gfx2hlsl.py` +21, teste +134, docs (+166/−41): `op_image_store` deixa de aceitar só `dim:SQ_RSRC_IMG_2D`; a forma 2D-array toma x, y e a camada em três registradores de endereço, vincula `RWTexture2DArray<float4>` (manifesto `tex2da`) e grava por `uint3(x, y, layer)`, com `dmask:0xf`, quatro VGPR de dado (dois no d16), guarda EXEC e caminhos de tabela fixa/pixel inalterados. 1D, 3D e cube mantêm a recusa. Fixture grava nas camadas 1 e 3 de quatro e compara as 5 376 palavras. |
| 83dcc922 | tedin7 | startup: refuse a lane mask read as numeric data by the remaining ALU forms in gfx2hlsl (Part of #1723) (#2702) | `gfx2hlsl.py` +131, teste +199, docs (+396/−28): recusa a forma VALU que lê máscara de lane como dado numérico (`v_cvt_f32_u32 v5, vcc_lo`, `v_add_nc_u32 v5, vcc_lo, v5`) e o movimento da metade alta de uma máscara wave64. Revisão `x2w-startup-shader-23`; o teste fixa que a recusa de máscara-como-número vem antes da cadeia. |
| 8d9f5201 | Chad Hyatt | shader: refuse whole-word lane reads as wave32 vcc_hi masks (#2776) | `gfx2hlsl.py` (+15/−8), teste +53, docs (+86/−23): `vcc_hi_scalar_mask_read` passa a tratar um escritor de palavra inteira por lane (`v_readfirstlane`/`v_readlane`) de `vcc_hi` — em ordem de branch também — como motivo para recusar a leitura VOP3 posterior de máscara em `vcc_hi`, em vez de reduzi-la ao bit 0. Revisão 24 invalida traduções que a 23 aceitou por engano. Casos `READFIRSTLANE_MASK` e `READFIRSTLANE_BRANCH`. |
| 96a7b4fb | Buhe | runtime: bind startup FP32 compute array UAV views (#2777) | `d3d12_compute_dispatch.inl` +20, teste novo (+105), doc novo (+33) (+198/−13): o consumidor do manifesto aceita `uav N tex2da float table P O channels 4`, exige UAV 2D-array explicitamente descrito com mip e faixa de slice válidos e de plano único, preserva `FirstArraySlice`/`ArraySize` do chamador e recusa UAV 2D antes do Dispatch. Fixture com recurso 32×8 RGBA32_FLOAT de seis slices e sentinelas distintas: camadas relativas 1 e 3 gravam slices físicas 2 e 4; compara 6 144 palavras. |
| 4bbe52ee | RavenCurseBurg | shader: preserve the zero sign of widened binary16 products (#2779) | `gfx2hlsl.py` (+6/−4), testes (+41/−20), docs +9: o produto binary16 passa por um produto binary32 `precise` antes do arredondamento para binary16, preservando o sinal do zero nas formas plain, packed e SDWA. Fixture lê `e000abcd` (e `6000abcd` se o sinal se perder) no caso SDWA de high-word preserve. Revisão `x2w-startup-shader-25`. |
| d1b2be6a | JB2WIN | startup: lower slot 57 pre-exchange execz blocks with several scalar loads (Part of #2413) (#2783) | `gfx2hlsl.py` +68, teste +111, doc novo (+97) (+276/−34): o bloco pré-exchange do mesh slot 57 aceita qualquer número de `s_buffer_load_dword*`/`s_load_dword*` e ALU escalar em linha reta dentro do `s_cbranch_execz`, não só um load como em #2660; o reset de EXEC pode estar mais acima desde que nada entre ele e o compare nomeie EXEC/VCC e nenhum branch de fora caia na faixa. Recusas específicas para reset ausente, segundo load vivo, load SMEM comum/mais largo. |
| d527e36c | JB2WIN | startup: accept dead scalar ALU in any slot 57 EXEC-zero block (Part of #2413) (#2785) | `gfx2hlsl.py` (+11/−1), teste +21, doc novo (+39) (+71/−10): a regra de #2643 (ALU escalar sem leitor depois da troca de contagem) passa a valer para **todo** bloco `s_cbranch_execz`, não só o que pula para a troca de contagem. A instrução precisa ser escalar aceita por `mesh_wave_block_scalar` (SGPR/SCC/VCC, sem EXEC/M0/`src_*`/MODE/memória/branch/mensagem) e nenhum registrador escrito — inclusive SCC — pode estar vivo no join. |
| 63451c5e | tedin7 | startup: keep sample_b and get_lod past a divergent EXEC with a per-quad vote (#2736) | `gfx2hlsl.py` +182, testes novos (+315/+502), docs (+1 072/−32): `image_sample_b` e `image_get_lod` tomam derivadas implícitas dos quatro lanes do quad; um programa pixel sem branch para trás passa a decidir `s_cbranch_execz`/`s_cbranch_execnz` por quad (`x2w_quad_any(exec)`: algum lane do quad 2×2 tem seu bit, por derivadas finas do bit 0/1), como o guest, que só pula o bloco quando nenhum lane está ativo. Um branch votado não parte quad. Recusa `sample_b` com bias por lane depois de EXEC divergente. |
| 169c4ee5 | triggered02 | gpu: cap slot 133 existing heaps at 256 MiB and grow upload ring on demand (#2775) | `d3d12_xs_device.cpp` +108, `d3d12_draw.inl` +35, `linear_buffer_backing.h` +20, teste novo (+818) (+1 046/−69): `X2W_D3D12_CAP_EXISTING_HEAP=1` limita `OpenExistingHeapFromAddress` a 256 MiB; sem a chave, alocações privadas maiores mantêm o caminho antigo, porque um placement copiado e limitado é recusado por `CopyBufferRegion` como `title_backed_source` e perde escrita tardia do guest. `reserve_upload_ring` cresce em pedaços de 16 MiB até o teto de 256 MiB em vez de commitar 256 MiB de uma vez; `commit_lazy_block` só roda para buffer elegível (MEM_PRIVATE, offset alinhado a 64 KiB, dentro da alocação). |
| 38628ca1 | RavenCurseBurg | tests: reuse and overlap binary16 draw shader compiles (#2790) | `tests/startup_gfx2hlsl_f16_alu_draw_check.py` (+33/−17): `draw_shaders` agenda com `ThreadPoolExecutor(max_workers=3)` e chave `(profile, source)`, então um pixel shader por grupo e até dois vertex shaders são compilados em paralelo e reusados; resultados consumidos na ordem de desenho para uma falha tardia não esconder uma anterior. |
| 564f9bad | Buhe | shader: support packed D16 compute array image stores (#2787) | `gfx2hlsl.py` (+1/−3), teste +46, doc +19 (+136/−51): aceita `image_store d16` em 2D e 2D-array com `dmask:0xf`, X/Y (e camada) de largura cheia e oito palavras de descritor intactas; R/G ocupam low/high do primeiro registrador e o manifesto registra `uav N tex2da float table P O channels 4`. Reusa o binding de array de #2777 e o alargamento D16 2D. Fixture compara 6 144 halfwords com cinco controles negativos (camada errada 128, EXEC perdido 32, metades trocadas 32, VGPRs trocados 64, store omitido 64). |
| 02e0a78d | tedin7 | startup: allow a backward branch around the v_interp_p1ll_f16 / v_interp_p2_f16 pair (#2754) | `gfx2hlsl.py` (+4/−2), teste +70: o par de interpolação binary16 passa a aceitar um laço de contagem constante em volta dele e das conversões de largura cheia; o scan não guarda estado através do branch, então a aresta de volta não precisa de recusa própria. Continuam recusados o laço que carrega um resultado de `p1ll` pela aresta de volta e o que carrega `p2` sem converter. Dois casos negativos no teste. |
| c5641b50 | triggered02 | docs: record design B retention pressure and the 16 GiB commit reduction (#2780) | `docs/experiments.md` (+91): registra a decisão de manter o design B (#2679) — descritor materializado e cópias seguram o recurso até o slot ser sobrescrito, copiado por cima ou o heap destruído — e o par medido em RX 9070 XT 16 GB com `X2W_D3D12_RELEASE_RETIRED` desligado/ligado: 8× `0x8007000e` e access violation em 490,7 s contra nenhuma falha em 600 s; textura retida no pico 10,67 GB/4 874 contra 9,41 GB/4 364; 53 000 referências desktop liberadas pelo drain. |
| 79410a80 | tedin7 | startup: model the SCC of s_add, s_sub, s_min and s_max in gfx2hlsl (#2778) | `gfx2hlsl.py` +38, teste +127, docs (+230/−51): o SCC de `s_add/s_sub/s_min/s_max` deixa de ficar como estava e passa a ser emitido — carry em `s_add_u32`, borrow em `s_sub_u32`, overflow assinado em `_i32`, `D == S0` em min/max; uma escrita de SCC ainda não modelada recusa os leitores por nome. Fixture em grade 4×4 com bordas `0`, `-1`, `INT_MAX`, `INT_MIN`, 16 bits de SCC e hash dos 16 resultados por grupo; controles `s-min-strict` e `s-borrow-flipped`. Revisão 27. |
| b2cf7dc5 | sydney-0o | docs: record the GTX 1080 Ti hardware CTest run (#2781) | `docs/experiments/hardware-gtx1080ti.md` novo (+50): revisão `96a7b4fb`, Windows 10 22H2 build 19045.6456, driver 32.0.15.8278 (582.78), Debug MSVC 19.31.31105.0, Python 3.12.14, DXC 1.8.2502.11; comando `ctest -R '_hardware'` serial e a nota de que fixture sem capacidade suportada conta como skip. Só registo. |
| 700522ef | Royoo | gpu: bind large backed raw views in translated indirect draws (#2477) | `d3d12_draw.inl` +6, teste +41, docs +7 (+55/−16): views de buffer de tabela vinculam a faixa completa de bytes a partir da alocação de upload da gravação, até o limite de raw view do D3D12 (2^32 elementos), com `limit = 4 << D3D12_REQ_BUFFER_RESOURCE_TEXEL_COUNT_2_TO_EXP`; acima de 16 MiB a mensagem de recusa muda e o alias de endereço só é tentado para ≤ 16 MiB. Teste lê o último dword de uma raw view de GPU que passa 16 bytes de 16 MiB. |
| 9b45cbb4 | RenzoWit35 | shader: intersect definitions at joins in the input mask scan (#2798) | `ps_input_scan.py` +67, teste +57, docs +4 (+128/−24): a regra de primeira leitura passa a intersectar, em cada join, os registradores escritos em todas as arestas de entrada — uma escrita que um branch condicional para frente pode pular não esconde mais uma leitura live-in depois do join. Casos `diamond_skip` (v2 vivo no join) e `diamond_both` (v2 escrito nas duas arestas); a máscara-isca sem v2 reprova o contrato por cobertura C1. |
| 4989a6df | Yann Bucaille | startup: spill descriptors, pointers and lane masks to VGPR lanes in compute (#2759) | `gfx2hlsl.py` +136, testes novos (+455/+522), docs (+1 158/−26): o idioma de spill `v_writelane_b32 vN, sM, k` / `v_readlane_b32 sD, vN, k` deixa de ser recusado para `k ≥ 32` e em estágio de vértice; passa a valer em compute para descritor, ponteiro e máscara de lane, em wave64 e através de branches. Entram `startup_gfx2hlsl_compute_spill` e `..._compute_mask_spill` com controles `s-arm-swapped`, `s-store-dropped`, `p-store-dropped`. |
| 10d45a9c | Roland Edi | startup: correct the title-memory leak comment on the wrap() failure path (#2801) | `d3d12_xs_device.cpp` (+5/−1): o comentário que dizia que o alocador do título não tem hook de liberação é corrigido — o mesmo arquivo documenta `__XMemAllocationHooks__` como `{alloc, free, …}` e usa o alloc em todo bloco de memória do título. **O bloco continua vazando** nesse caminho: a assinatura e os atributos da entrada `free` não foram verificados contra o título. |
| f9f4976b | Roland Edi | startup: free xcrdapi COM objects at final Release and join its queue worker (#2800) | `xcrdapi.cpp` +207, teste novo (+281), `CMakeLists.txt` +13 (+503/−10): o stand-in de DirectStorage passa a liberar de fato `Object`/`FileObject`/`QueueObject` no `Release` final (antes só decrementava), o worker da fila recebe sinal de saída e é aguardado antes de a fila ser liberada, e uma entrada pendente segura o array de status e o arquivo de origem que usa. `x2w_xcrdapi_live_objects`/`_live_workers` expõem as contagens ao fixture; mutante `naive` (deletar sem parar o worker) tem de reprovar. |
| 2e3070c6 | RenzoWit35 | tooling: refuse non-finite --timeout in dxc_compile_time (#2802) | `tools/dxc_compile_time.py` (+3/−2), teste +4: `--timeout` passa por `math.isfinite(args.timeout) and > 0` e `parser.error("--timeout must be a finite positive number")` antes de qualquer trabalho; `inf`, `-inf`, `nan` e `1e999` reprovam em vez de estourar `OverflowError`/`ValueError` no `communicate`. |
| c02650f0 | RenzoWit35 | tooling: reject malformed debug_launch.py option values before launching (#2804) | `tools/startup/debug_launch.py` (+12/−3), teste novo (+70), CMake +7: `--timeout`, `--memory-gib` e `--continue-asserts` são validados **antes** da conversão, com mensagem "expected …" e saída 2 antes de criar job object ou processo; `--memory-gib` finito, positivo e dentro de `size_t`. O teste usa executável inexistente de propósito. |
| 1983228b | RenzoWit35 | tooling: reject a script given twice on the run_checks_parallel command line (#2805) | `tools/run_checks_parallel.py` (+3), teste +10: `main` passa a recusar um script repetido na linha de comando (`duplicate command-line script: <path>`, status 1, nada roda), como `read_check_files` já fazia para os arquivos de `tests/checks/*.txt` e o README prometia. |

## Pull requests abertos (20 de 50)

Cinquenta PRs abertos acima do cursor 2 861 (lista limitada a 50). Documentados os 20 de número mais alto.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 2925 | startup: back off the retry for an absent shader cache manifest | `manifest_retry.h` novo, `d3d12_*`, teste: recuo na retentativa de manifesto de cache ausente. | Não. |
| 2924 | docs: plan slot 57 mesh DXIL translation | `docs/experiments/mesh57-dxil-plan.md`: plano. | Não. |
| 2923 | docs: plan EXEC removal and structurization | `docs/experiments/exe-structurization.md`: plano. | Não. |
| 2922 | startup: lower the FP32 box case of image_bvh_intersect_ray | `gfx2hlsl.py` + teste: caixa FP32 de interseção de BVH. | Vizinho de D-13 (leitura raw por view tipada no meshlet culling); caminho diferente. |
| 2921 | tests: model the console BVH node set | `bvh_node_model.txt`, `bvh_model.py`: modelo do conjunto de nós. | Não. |
| 2920 | startup: gate the BVH builder compute stages all or none | `compute_builder_gate.h` novo, `d3d12_xs_device.cpp`: gate tudo-ou-nada dos estágios de build de BVH. | Não. |
| 2919 | startup: persist and reuse translated PSO recipes | `pso_recipe.h` novo, `d3d12_draw.inl`: persiste receita de PSO traduzido. | Não. |
| 2918 | runtime: bring XMem page-run fixes into cpp-migration | 19 arquivos: porta as correções de page-run de XMem. | Continua `f30a9e3a`/`169c4ee5`. |
| 2917 | shader: lower v_pack_b32_f16 in the direct C++ DXIL path | draft; `gfx1013_direct_compute.inl`, `native_pack_b32_f16_check.py`. | Não. |
| 2916 | tooling: add --assert-sites to the startup debugger | `debug_launch.py`, `run.ps1`, `run.sh`, docs: lista sítios de assert. | Não. |
| 2914 | shader: draw a per-vertex VCC across a vertex-stage join | `native_graphics_vertex_vcc_join_check.py`, doc: VCC por vértice através de join. | Não. |
| 2913 | startup: lower v_ldexp_f32 to ldexp32 in gfx2hlsl | `gfx2hlsl.py` + teste. | Não. |
| 2912 | test: native DXIL read of a GPU-selected descriptor heap slot (probe) | draft; `native_heap_handle_probe.cpp`, `native_compute_pipeline.cpp`. | Não. |
| 2911 | xmem: make page runs cheaper under Wine | 18 arquivos, `xmem_shim.cpp`, `xmem_page_runs_check.py`: page runs mais baratos sob Wine. | Vizinho de D-17 (liberação imediata do backing de descritor). |
| 2910 | startup: refuse compute buffer loads through a modified descriptor (#2901) | `gfx2hlsl.py` + teste: recusa load por descritor modificado. | Não. |
| 2904 | test: add startup_compute_dispatch C++ test | `upload_reservation.h` novo, `startup_stream_draw.cpp`: teste do dispatch. | Não. |
| 2903 | startup: keep s_cselect_b32 on descriptor data symbolic in compute (Refs #2618) | `gfx2hlsl.py` + doc: mantém `s_cselect` simbólico sobre descritor. | Não. |
| 2902 | startup: lower compute image_store to 1D and 3D texture UAVs (Part of #2022) | 14 arquivos: store em UAV 1D/3D. | Continua `624f5927`/`564f9bad`. |
| 2900 | fix(shader): prove scalar runs before compute EXEC restores | `gfx2hlsl.py` + teste: prova de execução escalar antes de restore de EXEC. | Não. |
| 2899 | startup: accelerate gfx2hlsl cache scans and support skipping earlier failures | `gfx2hlsl.py`, teste do watch: cache scans mais rápidos. | Não. |

## Pull requests mesclados (57)

Todos *squash*: `mergeCommit.oid` é o SHA da tabela de commits. "= linha N" aponta a linha da tabela acima;
"além do teto" marca PR cujo commit caiu nos 32 que ficam para a próxima execução — o merge é desta hora,
a leitura do diff não.

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| 2762 | tests: follow the shared word3 decoder in draw preflight | f0101e79 | = linha 2 |
| 2767 | audio: commit the stand-in APU heap on demand | 24be1786 | = linha 6 |
| 2769 | startup: prove uniform shifts of masked and extracted thread IDs | 756e4bb4 | = linha 9 |
| 2770 | xmem: lazily reserve below-40 startup blocks and reclaim empty slabs | f30a9e3a | = linha 14 |
| 2768 | startup: refuse uniform load proofs across EXEC-changing loops | 6427564b | = linha 15 |
| 2771 | startup: run a GPU-written indirect Dispatch on a translated slot 67 pipeline | cd5fb38b | = linha 16 |
| 2774 | startup: keep the object receiver for 8-byte struct returns when rdx holds a wrapper | 475bb730 | = linha 17 |
| 2776 | shader: refuse whole-word lane reads as wave32 vcc_hi masks | 8d9f5201 | = linha 20 |
| 2777 | runtime: bind startup FP32 compute array UAV views | 96a7b4fb | = linha 21 |
| 2779 | shader: preserve signed zero in plain, packed and SDWA binary16 multiplication | 4bbe52ee | = linha 22 |
| 2783 | startup: lower slot 57 pre-exchange execz blocks with several scalar loads (Part of #2413) | d1b2be6a | = linha 23 |
| 2785 | startup: accept dead scalar ALU in any slot 57 EXEC-zero block (Part of #2413) | d527e36c | = linha 24 |
| 2775 | gpu: cap slot 133 existing heaps at 256 MiB and grow upload ring on demand | 169c4ee5 | = linha 26 |
| 2790 | tests: reuse and overlap binary16 draw shader compiles | 38628ca1 | = linha 27 |
| 2787 | shader: support packed D16 compute array image stores | 564f9bad | = linha 28 |
| 2754 | startup: lower a loop around the v_interp_p1ll_f16 / v_interp_p2_f16 pair in the pixel stage | 02e0a78d | = linha 29 |
| 2780 | docs: record design B retention pressure and the 16 GiB commit reduction | c5641b50 | = linha 30 |
| 2778 | startup: model the SCC of s_add, s_sub, s_min and s_max in gfx2hlsl | 79410a80 | = linha 31 |
| 2781 | docs: record the GTX 1080 Ti hardware CTest run | b2cf7dc5 | = linha 32 |
| 2798 | shader: intersect definitions at joins in the pixel input mask scan | 9b45cbb4 | = linha 34 |
| 2759 | startup: spill descriptors, pointers and lane masks to VGPR lanes in compute | 4989a6df | = linha 35 |
| 2801 | startup: correct the title-memory leak note on the wrap() failure path | 10d45a9c | = linha 36 |
| 2800 | startup: free xcrdapi COM objects at final Release and join its queue worker | f9f4976b | = linha 37 |
| 2802 | tooling: refuse non-finite --timeout in dxc_compile_time | 2e3070c6 | = linha 38 |
| 2804 | tooling: reject malformed debug_launch.py option values before launching | c02650f0 | = linha 39 |
| 2805 | tooling: reject a script given twice on the run_checks_parallel command line | 1983228b | = linha 40 |
| 2811 | gpu: reject meshlet max_vertices below 3 | 295afb7a | além do teto (linha 41) |
| 2803 | tooling: run dxc_cache_check on Windows with a .cmd fake compiler | 174f963d | além do teto (linha 42) |
| 2810 | tooling: skip a UTF-8 byte-order mark in check listings | d38dc93e | além do teto (linha 43) |
| 2806 | shader: pin v_fma_mix half selection per source position | b0f49ce5 | além do teto (linha 44) |
| 2808 | startup: count acceleration-structure refusals by name in the slot 70 census | e2271cce | além do teto (linha 45) |
| 2807 | runtime: list R8_TYPELESS, not R16_SINT, as typeless in the default-view check | 849c448d | além do teto (linha 46) |
| 2813 | docs: record Intel Arc A770 hardware coverage | e954fc55 | além do teto (linha 47) |
| 2809 | runtime: refuse page-token writes to pages without a grant | 92934813 | além do teto (linha 48) |
| 2812 | startup: reject a relative input-script time that overflows to infinity | 49f53ae6 | além do teto (linha 49) |
| 2814 | tests: accept 4 ulps for v_log_f32 in the transcendental fixture | 410f6004 | além do teto (linha 50) |
| 2818 | startup: lower v_fma_mix* through a precise mad so a zero product keeps its sign | 02075500 | além do teto (linha 51) |
| 2821 | readme: record the reported lane range for the RX 9070 XT wave64 run | 25eb746e | além do teto (linha 52) |
| 2823 | gpu: preserve live upload reservations | 7256cecf | além do teto (linha 53) |
| 2824 | startup: accept the early exit of a kill in the slot 57 pixel stage (Part of #2413) | d4dfe97b | além do teto (linha 54) |
| 2795 | shader: clear the lane-mask mark on scalar writes of descriptor data in gfx2hlsl (Part of #1723) | 83086b92 | além do teto (linha 55) |
| 2825 | startup: document the storage path as a drive root | 8c9f168f | além do teto (linha 56) |
| 2830 | startup: support pixel-stage 2D-array image_store and UAV binding | 6cf44bec | além do teto (linha 57) |
| 2834 | gpu: allow live aliased indices with un-sliced placed vertex streams | da782506 | além do teto (linha 58) |
| 2838 | shader: prove v_log_f32 and v_ceil_f32 results uniform for v_readfirstlane_b32 | 4562c572 | além do teto (linha 59) |
| 2840 | docs: point the Series S console profile at the plane composite | 4992a46e | além do teto (linha 60) |
| 2828 | startup: record a bounded per-draw texel history (Part of #1268) | 22aa7bf9 | além do teto (linha 62) |
| 2842 | shader: prove uniform compute image_load results uniform for v_readfirstlane_b32 | 7538470e | além do teto (linha 63) |
| 2835 | shader: recognise whole wave32 reductions in gfx2hlsl pixel stages (Part of #2125) | 2c56d896 | além do teto (linha 64) |
| 2844 | tests: keep two hardware negative controls meaningful on RDNA 1 (#2730) | 3dfec4c1 | além do teto (linha 65) |
| 2850 | fix(shader): select the completed difference for SDWA subtract with borrow | 1940788d | além do teto (linha 66) |
| 2851 | startup: exercise the pixel-history hook through the translated draw path | 8d389f5e | além do teto (linha 67) |
| 2799 | runtime: keep the view tables consistent when an allocation fails | 6a094052 | além do teto (linha 68) |
| 2852 | fix(shader): identify compute loop exits that cross a backward SCC branch | 5a41adba | além do teto (linha 69) |
| 2854 | startup: lower the first active lane of a pixel stage in gfx2hlsl (Refs #2618) | 52905a32 | além do teto (linha 70) |
| 2856 | cpu: lower ENDBR64 and ENDBR32 as no-ops in the shared lowering | b505c32b | além do teto (linha 71) |
| 2849 | fix(startup): reuse upload ring chunks after GPU completion | 6ee878d8 | além do teto (linha 72) |

## Cruzamento com o catálogo

- **Não corrige nenhum D-xx diretamente.** `6a6eecfe` mexe no load de descritor dentro de braço
  mascarado, o mesmo subsistema do D-05 ("loads pendentes não unidos nas fronteiras dos braços
  mascarados"), mas o sintoma aqui é recusa, não load errado.
- **Vizinho de D-03** (rollback do commit de slab deixa slab aberto): `f30a9e3a` mexe no ciclo de vida do
  slab (fecha slab vazio e devolve commit), mas não no rollback descrito no catálogo.
- **Vizinho de D-17** (liberação imediata do backing de descritor): `169c4ee5` limita heaps existentes a
  256 MiB e cresce o anel de upload sob demanda; o backing de descritor não é tocado.
- **Corrige, sim, um defeito que não está no catálogo e é declarado como tal:** `33004fa1` fecha a
  contagem de referência de slots 19–21 que impedia uma textura de classe de ser liberada; `24be1786`
  fecha o esgotamento de commit do heap APU; `f9f4976b` fecha o vazamento de objetos COM do `xcrdapi`.
- **Abre candidato novo (vazamento declarado):** `10d45a9c` — o bloco de memória do título vaza no
  caminho de falha de `wrap()`, e o próprio comentário diz que a entrada `free` de
  `__XMemAllocationHooks__` não foi verificada contra o título.
- **Abre candidato novo (ABI de retorno de struct):** `475bb730` — `small_result` escolhia a forma
  (buffer, objeto) quando o segundo registrador tinha um wrapper vivo, escrevendo o handle de 8 bytes
  por cima da vtable do heap; release builds crashavam no `Release` seguinte.
- **Abre candidato novo (self-catch de revisão):** `8d9f5201` — a revisão 24 invalida traduções que a 23
  aceitou por engano (leitura de palavra inteira em `vcc_hi`). O commit declara o erro, o que é sinal de
  que a validação por controle negativo ainda não pegava esse caso.
- **Abre candidato novo (limite de raw view):** `700522ef` — views acima de 16 MiB e o limite de
  elementos do D3D12; antes o limite era fixo em 16 MiB e a recusa não distinguia o motivo.

## Confiança

- **Verificado por diff:** os 40 commits tiveram o *patch* lido pela API. Revisões de cache
  (`x2w-startup-shader-19` a `-27`), constantes (`0x3b80_8081` não se aplica aqui; `2^40`, `256 MiB`,
  `16 MiB`, `0xdeadbeef`, `4096` extents) e contagens (320, 5 376, 6 144, 53 000, 10,67/9,41 GB) saem do
  diff ou do texto que o commit adiciona.
- **Corte de 40 aplicado:** a faixa tem 72 commits; 32 ficam depois de `1983228b` (inclui `295afb7a`,
  `d38dc93e`, `6ee878d8` e os PRs marcados "além do teto"). `head_at_run` = `6ee878d8`.
- **Contagem de merges fecha com a de commits da janela:** 57 PRs com `mergedAt` posterior a
  `2026-11-26T06:27:26Z` (comparação de string UTC, não de `datetime` local) e 72 commits na faixa; cada
  `mergeCommit.oid` dos 57 está na faixa. Não há merge órfão.
- **Divergência de leitura registrada:** a primeira contagem de PRs mesclados usou `[datetime]::Parse`
  sobre o timestamp UTC e o PowerShell o interpretou como hora local (−03:00), o que trouxe três PRs de
  antes do cursor (#2751, #2752, #2756). Refiz com comparação de string ISO e o número caiu de 60 para
  57. A tabela acima é a de 57.
- **Não verificado:** corpos dos PRs abertos (proposta tirada do título e dos caminhos de arquivo); os
  resultados de hardware citados (WARP, GTX 1660 SUPER, RX 9070 XT 16 GB, GTX 1080 Ti, RX 7900 XT) não
  foram reproduzidos; as revisões de cache posteriores a 27 não existem nesta faixa.
