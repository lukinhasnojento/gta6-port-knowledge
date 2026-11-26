# brcly/TuxBox — atualização de 2026-11-26 20:00

## Resumo

**40 de 84 commits** na faixa `ac4f0993..b3184d35` (teto de 40; `commit_cursor` em `be9055c9`, 44 ficam
para o próximo ciclo). A faixa tem duas metades visíveis: o tradutor `xshader` recebendo correções de
fidelidade de wave (DPP16 row-scan, `ds_swizzle_b32` em `paired_wave64`, SCC em máscaras de root, SGPRs de
tabela vindos dos registros `FSSM`, limites de LDS e de `tb_swizzle`, recusa por orçamento de memória do
DXC), e o runtime `d3d12` atacando importação de páginas XMem, slot 62 com textura arredondada a tile,
substituição da rotina `clzero` do título por stores AVX e pausa do watchdog enquanto um thread do título
espera. Entram também dois ajustes de cachê de shader do launcher que revisitam o D-04 do catálogo (teto de
32 GiB → 4 GiB) e um limite fixo de 90 s no compilador `-O0` que recusava os kernels de ray tracing em todo
boot (`7e0ad65c`). 24 PRs mesclados na janela, todos com merge commit dentro dela.

## Commits novos (40)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| e4fdfe4c | Valdi320 | shader: support portable wave exchanges in multidimensional groups | 12 arquivos, +11 723/−22: os caminhos portáveis de redução e de troca DPP8 passam a aceitar wave64 lógica em qualquer forma de workgroup com 64 invocações (`8x8x1`, `4x4x4`); endereço de troca em group-shared passa a usar o ID local achatado x-fast em vez de só x. Entra `evidence/scarlett-wave-multidimensional-control.json` com 594 casos e 228 096 palavras conferidas em RTX 4080 (driver 615.78.08, GE-Proton11-12). O grosso das linhas é fixture de teste. |
| bd722d00 | Valdi320 | shader: support exact straight DPP8 across multiple waves | 9 arquivos, +20 904/−24: DPP8 reto com fontes só dentro do octeto passa a valer para múltiplas waves em blocos completos de 32 threads até 1 024 invocações, com o array compartilhado dimensionado para o workgroup inteiro; sem fluxo de controle nem alteração de EXEC, e leitura de lane constante continua restrita a um grupo de 32. Evidência: 768 casos, 614 400 palavras. |
| fceed400 | Nick Stallman | host: also hide VK_EXT_descriptor_heap on GPUs that need storage CBVs | `console_cbv.py`, `tests/test_host.py`, `tools/reference/cbv-layer/layer.c` (+16/−8): `VKD3D_DISABLE_EXTENSIONS` passa a listar `VK_EXT_descriptor_buffer,VK_EXT_descriptor_heap`; a camada `layer.c` filtra as duas extensões e define o nome de `VK_EXT_descriptor_heap` para headers antigos. O filtro só age nas GPUs que precisam de CBV de armazenamento. |
| 915eee7b | spook40 | shader-compiler: support ds_swizzle_b32 in paired_wave64 | `paired_wave64.rs` + `check_paired_wave64.py` + teste (+70): `ds_swizzle_b32` entra no modelo de wave emparelhada, com a semântica de `offset` reproduzida no verificador — bit 15 escolhe modo quad (`partner = (l & ~3) \| ((off >> 2*(l&3)) & 3)`) e o outro modo aplica `and`/`or`/`xor` dentro de 32 lanes. Teste com dois `ds_swizzle_b32` sob `s_and_saveexec_b64`. |
| 85bcf0ec | brainbudt | fix(tools): rank pipeline failures by the recorded refusal reason | `tools/rank_pipeline_failures.py` + teste (+32/−3): a classificação passa a preferir o motivo de recusa **gravado** (`CS.refused=…`) ao invés de adivinhar por nome de estágio; testes cobrem `driver compile failed: 0x80004005` e `driver compile deadline`, e garantem que o motivo gravado precede o palpite de lookup. |
| 724cd950 | Nick Stallman | dxbc, xshader: read four-word typed UAVs in a mesh pipeline's pixel layout | `runtime/dxbc/src/mesh_table.rs` + teste + `emitter/image.rs` (+92): a tabela de layout ganha `(0x303, 4, 4)` para quando o programa pixel lê UAV tipado de quatro palavras (`named(6)==1 && named(8)==0`); teste monta um programa `FSSM` com t91 no slot 2 e u13 de quatro palavras no slot 3, root declarando u7 no slot 2. |
| 31dd5a3f | brcly | Merge pull request #211 from spook40/fix/paired-wave64-ds-swizzle | Merge do PR #211; 3 arquivos, +70 — conteúdo idêntico ao de `915eee7b`. |
| bb01537b | brcly | Merge pull request #214 from NickStallman/fix/xshader-unresolved-image-lz | Merge do PR #214; 4 arquivos, +92 — conteúdo idêntico ao de `724cd950`. |
| c71d4d7a | brcly | Merge pull request #213 from brainbudt/fix/rank-recorded-refusal-reason | Merge do PR #213; 2 arquivos, +32/−3 — conteúdo idêntico ao de `85bcf0ec`. |
| 773278ad | brcly | Merge pull request #212 from NickStallman/fix/cbv-layer-descriptor-heap | Merge do PR #212; 3 arquivos, +16/−8 — conteúdo idêntico ao de `fceed400`. |
| a9eaa2ba | Bas Bijsterbosch | xshader: lower long jumps and waterfall lane reads | 5 arquivos, +316/−6: entra `GROUP_GATHER` — cada lane publica seu valor em `tb_swizzle`, barreira de grupo, e lê a lane que o índice nomeia (`tb_swizzle[(tb_lane & ~63u) \| (idx & 63u)]`); `lower_waterfall_gathers` troca N leituras por lane por uma coleta por wave em compute. O ajuste de salto longo deixa de reescrever `insns[i+1]`/`insns[i+2]` por offset e passa a usar a lista de entradas. |
| fc453bd2 | Bas Bijsterbosch | xshader: lower DPP16 row-shift prefix sums | 4 arquivos, +216/−1: `tb_row_scan_add32` é emitido como `WavePrefixSum(v) + v` com `WaveReadLaneAt(t_scan, 15u)` para lanes ≥ 16 — o *scan* inclusivo por linha que os shaders de compactação do ProjectScarlett usam; recusa explicitamente quando o alvo é `pixel_wave` ou `portable_wave64` ("row scan needs a native wave32"). `wave_sum::lower_row_scan` casa o padrão de três `v_add_nc_u32` com `row_shr:1/2/4`. |
| 7102de40 | Bas Bijsterbosch | xshader: follow SCC through packed-root pointer masks | `packed_root.rs` + testes (+64/−6): `State` cresce de 106 para 107 palavras com `SCC = 106`; `writes_scc(ins)` decide por codificação (`Enc::Sopc` sempre; `Sop2` exceto `s_cselect_`, `s_bfm_`, `s_pack_`, `s_mul_*`; `Sop1` só `*saveexec`/`*wrexec`/lista explícita). Antes qualquer `s_cselect_b32`/`s_addc_u32`/`s_subb_u32` fazia a análise desistir com "SCC may depend on a root address". |
| 08d7a86e | Bas Bijsterbosch | xshader: take root table SGPRs from FSSM user-data records | `emitter/bindings.rs` (+13): os registros de user-data do `FSSM` (`compute_table::user_data_records`) passam a decidir qual SGPR carrega o ponteiro de tabela (`location = parâmetro << 8 \| SGPR`); antes o código deduzia pelo `sgpr` do programa, o que errava em programas como `s_swappc_b64 s4, s4` que recebem tabelas em s8/s9. |
| d154f07b | Bas Bijsterbosch | xshader: keep unproved uniform loops in the full-mask mode | 4 arquivos, +31/−12: `wave_loops::bounded_latches` vira opcional — `Some(latches)` usa as travas, `None if uniform_dispatch(&em)` mantém o modo de máscara cheia, `None` sem prova continua recusando; o watchdog estruturado fica com o emissor e todo lane o conta igual. |
| 4c844d12 | Bas Bijsterbosch | xshader: or jump flags together in compute programs | `control_flow.rs`, `scalar.rs`, testes (+41/−13): o guard passa a combinar flags com `\|` em vez de `\|\|` quando o programa é compute (`unoptimised or under a DXC budget`) — o comentário mede "lost a third of its blocks and DXC finished optimising in 45 s" com `\|\|`. Também troca a varredura `target.iter().any(...)` por um `HashMap` de primeira ramificação por label. |
| c4f682b4 | Bas Bijsterbosch | xshader: refuse translations beyond DXC's memory budget | `main.rs` (+12): `MAX_HLSL_BYTES = 8 << 20` e recusa com `Error::Unsupported("translated HLSL of N MiB exceeds the 8 MiB compiler budget")`. O comentário registra ~1,1 GB de DXC por MB de HLSL em `-O0` e dois programas de 10 MB consumindo 11 GB cada. |
| ea15a2d3 | Bas Bijsterbosch | xshader: read heap buffer strides from the runtime's table | 5 arquivos, +41/−3: entra `heap_strides`/`heap_strides_used` — o root carrega a tabela de stride por slot (SRV t1, space 32766) e o emissor de buffer lê a palavra 1 do V# quando o descritor veio de heap, em vez de depender só de `sibling_stride`. |
| 736a58a2 | Bas Bijsterbosch | d3d12, dxbc: bind compute programs to the root the title pairs them with | 21 arquivos, +530/−54: `Pipeline::Regular` ganha um sexto campo (`Option<ID3D12RootSignature>`), e o dispatch compute usa `sampler_root()`/`root()` desse par em vez do estado global; tabelas de stride de heap são publicadas (`SetComputeRootShaderResourceView`) só quando o programa declara `heap_strides`. Antes `Regular` tinha 5 campos e o root vinha de outro caminho. |
| 12255e4e | Bas Bijsterbosch | d3d12: replace ProjectScarlett's hot clzero routine with AVX stores | `clzero.rs` + `survey.rs` + teste (+214/−3): com o RVA `0x4E8AA10` e o fast path de `0x60` bytes, um hook reescreve a rotina de limpeza do título para stores AVX sobre `[align_down(rcx+0x40, 64), +0x7400)` e salta para a cauda em `+0xEC` (que zera `xmm0` e faz `vzeroupper`). Motivação medida: cinco laços `clzero` por chamada, algumas centenas de chamadas por segundo, ~4 milhões de exceções em meia hora. |
| 6f28b2db | Bas Bijsterbosch | d3d12: pause ProjectScarlett's hang watchdog while a title thread waits | 5 arquivos, +162: `title_watchdog.rs` novo com `pause()` guard, usado nos três pontos em que um thread do título espera (compilação do driver, espera até deadline, tradução e pedidos à frente do mesmo digest); teste `recognizes_only_the_exact_watchdog` compara bytes para não pausar em imagem parecida. |
| 7e0ad65c | Bas Bijsterbosch | launcher: scale the -O0 cap with program size, log vkd3d-shader errors | `backend_log.py`, `runtime_shaders.py`, testes (+27/−5): o teto fixo `O0_TIMEOUT` vira `o0_timeout(hlsl)` com `90 s/MB`, limitado a 600 s, e `VKD3D_SHADER_DEBUG=err` entra no ambiente. O comentário registra o motivo: os kernels de ray tracing geram ~2,5 MB de HLSL, ~77 s cada em `-O0`, até quatro em paralelo — o teto de 90 s os recusava em **todo** boot. |
| 12ddd28b | ReverendOzmund | d3d12: accept slot 62 copies of textures padded to a whole tile | `copy_memory.rs` + teste (+28/−3): além de `guest_bytes - span <= span`, aceita `tile_padded(...)` — guest range em linhas inteiras do pitch, arredondado até o próximo múltiplo de `TILE_ROWS = 64`; acima disso segue recusando. |
| 4461c8fb | Exorion | d3d12: import XMem physical pages through their backing | 6 arquivos, +473/−28: `heap/backing.rs` novo — a importação de página física usa a memória privada por trás (o `amdgpu` só importa memória anônima privada), e o importador **segura o backing** até o heap nativo ser liberado (o RADV passa todo buffer vivo em cada submissão; um import com páginas host sumidas faz o kernel recusar com `EFAULT` e perde o device). Saem `direct`/`shadowed` e a validação por `contains_mapped_pages`. |
| a8f2b9c1 | Nick Stallman | verify: present through X images when the adapter is not Xwayland's GPU | `verify.py`, `frame_present.cpp`, testes, `check_lanes_input.py` (+130/−33): entram `_drm_pci_id(node)` e `_adapter_driver(adapter)` (lê `/sys/class/drm/renderD*/device/vendor|device|driver`) e o servidor X privado rootful sai; o present passa a usar X images quando o adaptador não é o do Xwayland. |
| 70d71fbb | Jeffrey C | d3d12 checks: free the memory under an imported buffer | `buffer_import_guest.cpp` + testes (+112/−1): o binário de verificação ganha `unmap_mode` com dois modos — `released` (destrói o buffer antes de liberar as páginas; toda cópia posterior deve funcionar) e `alive` (libera as páginas com o buffer vivo; no `amdgpu` todas as submissões seguintes são recusadas e o device é perdido). O recibo diz qual aconteceu. É o teste do mecanismo de `4461c8fb`. |
| 84b9d9cc | Jeffrey C | tools: filter the translator checks' adapter by its name without the driver tag | `check_translator_gpu.py` + teste (+21/−2): `filter_name` remove o sufixo do driver (`(RADV NAVI22)`, `(DG2)`) antes de aplicar `VKD3D_FILTER_DEVICE_NAME`/`DXVK_FILTER_DEVICE_NAME`, porque o guest do Proton pode ver o nome sem a tag. |
| c3c95b2b | ReverendOzmund | d3d12: say why a pipeline has no paged descriptor layout | `compute_bindings.rs`, `recording.rs`, `compute_scene.rs` + teste (+55/−10): `supports_paged_draw` vira `paged_layout_gap` com três motivos distintos (`no root signature was recorded`, `indexes the resource heap directly`, `root signature could not be parsed`), e o log de skip nomeia qual foi. |
| ed742e8c | brcly | Merge pull request #216 from bbijsterbosch/projectscarlett/start-menu | Merge do PR #216; 57 arquivos, +1 667/−103 (compute roots, compilações mais rápidas, pausa do watchdog). Conteúdo é o conjunto `736a58a2`, `12255e4e`, `6f28b2db`, `7e0ad65c` e vizinhos. |
| 771a4e5b | brcly | Merge pull request #217 from ReverendOzmund/d3d12-slot62-tile-padded | Merge do PR #217; 2 arquivos, +28/−3 — conteúdo idêntico ao de `12ddd28b`. |
| 1ef77321 | brcly | Merge pull request #220 from JeffreyBytes/test/buffer-unmap-control | Merge do PR #220; 3 arquivos, +112/−1 — conteúdo idêntico ao de `70d71fbb`. |
| d1112f92 | brcly | Merge pull request #221 from NickStallman/fix/guest-present-local | Merge do PR #221; 4 arquivos, +130/−33 — conteúdo idêntico ao de `a8f2b9c1`. |
| 0072fe18 | ReverendOzmund | launcher: carry the shader cache over to a new Proton's prefix | `host.py` + teste (+110/−1): `SHADER_CACHE = SOURCE_CACHE.parent` e `seed_shader_cache(prefix, report)` copiam o cache do prefixo mais recente para o prefixo novo (cada entrada é chaveada pelo conteúdo que a produziu), para não retraduzir tudo a cada release do Proton. |
| 00ba85ff | brcly | Merge pull request #223 from ReverendOzmund/d3d12-paged-layout-reason | Merge do PR #223; 4 arquivos, +55/−10 — conteúdo idêntico ao de `c3c95b2b`. |
| 9b31fd8c | brcly | Merge pull request #224 from ReverendOzmund/launcher-carry-shader-cache-pr | Merge do PR #224; 2 arquivos, +110/−1 — conteúdo idêntico ao de `0072fe18`. |
| 6f510a52 | ReverendOzmund | xshader: check group shared accesses against the guest's LDS allocation | 4 arquivos, +51/−23: todo acesso a LDS passa por `tb_lds_load(at)`/`tb_lds_store(at, v)` com limite `TB_LDS_WORDS` (refletido do programa, `784` em micro-poly vote, padrão 8192) em vez do `& 8191u` que dava volta no índice; escritas fora da alocação são descartadas, leituras devolvem 0. Atômicos ganham o mesmo teste de faixa antes do `InterlockedOr`. |
| 65f46ac5 | Valdi320 | Merge upstream main and preserve both wave-reduction regression suites | Merge de `main` upstream com resolução de conflito em 6 arquivos (+6 725/−382): traz os fixtures de row-scan de `0c40c71f` (`v_cndmask_b32` + três `v_add_nc_u32 row_shr` + add entre linhas) e mantém as duas suítes de regressão de redução (a do upstream e a do fork). Não é merge vazio: o diff contra o primeiro pai contém resolução própria. |
| 25e0610a | ReverendOzmund | tools: GPU check that LDS accesses past the allocation read 0 and drop writes | `tools/check_lds_bounds_gpu.py` novo + `check_translator_gpu.py` + exemplo (+139): verificação na placa de que leitura além da alocação devolve 0 e escrita é descartada; o teste escreve 32 KiB adiante e confere que a palavra em faixa mantém o primeiro valor (o `& 8191u` antigo sobrescrevia e lia de volta). |
| c6ce497b | brcly | launcher: limit the NVIDIA shader cache to 4 GiB | `host.py` + testes + dois docs (+9/−7): `NVIDIA_CACHE_BYTES` cai de `32 << 30` para `4 << 30`, com o motivo no código: o cache do ProjectScarlett chegou a 13,7 GB e o driver parou de atualizar o índice com ~5 GB, então nada escrito depois era encontrado. |
| be9055c9 | ReverendOzmund | xshader: check lane reads through the exchange array against its size | 4 arquivos, +14/−11: leituras por `tb_swizzle` passam por `tb_swizzle_read(i)` que devolve 0 fora de `TB_SWIZZLE_WORDS`; cobre o caso de grupo de 32 ou 96 lanes, que termina em meia wave64 e podia nomear lane da metade ausente. |

## Pull requests abertos (20)

Vinte e nove PRs abertos na coleta (`newest` = #266). Documentados os 20 mais recentes, todos acima do
cursor 210; "Proposta" sai do título e do conjunto de arquivos (GraphQL), sem leitura de patch.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 266 | d3d12: give a mesh pipeline's pixel stage the literal samplers its root leaves out | draft; `examples/lanes/*` e `project_tuxbox/verify.py`: amostradores literais para o estágio pixel de mesh. | Vizinho de D-21 (assinatura mesh/pixel incompatível); mesmo assunto, correção diferente. |
| 265 | xshader: translate 32-bit image_atomic_cmpswap on typed UAVs | `hlsl.rs` +3, `image_cmpswap.rs` +33, `check_image_cmpswap_gpu.py` +183 e shader de referência: `atomic_cmpswap` de 32 bits em UAV tipado. | Não mapeado no catálogo. |
| 264 | verify: run Wave64 mask ballot controls on Wave32-only hosts | draft; roda controles de ballot de máscara wave64 em host só wave32. | Vizinho de D-28 (`WaveSize(N)` incompatível recusado só no driver). |
| 262 | launcher: discover Proton in Flatpak and secondary Steam libraries | draft; `host.py` +6/−4 e teste: acha Proton em Flatpak e bibliotecas Steam secundárias. | Não. |
| 261 | tools: replay Wave32 variants exactly and audit compute loop guards | draft; `audit_shader_guards.py` +124, doc: auditoria de guardas de laço em compute. | Não. |
| 260 | d3d12: translate root-less compute programs against a root reflection cannot describe | `runtime/d3d12/src/device/compute.rs` +19, `examples/lanes/selftest.cpp` +112: programa compute sem root. | Vizinho de D-22 (`Failed to remap` em roots raster); caminho compute. |
| 259 | d3d12: check device removal during prefix fence waits | draft; `memory_wait.rs` +30/−17, `fence_poll.rs` +26 e teste: detecta remoção do device durante espera de fence. | Vizinho de D-29 (filas param em esperas de fence); só diagnóstico. |
| 258 | xmem: back physical pages with private memory and export backing holds | draft; `physical_pages/backing.rs` +519, `private_views.rs` +337: backing privado e holds exportados. | Continuação de `4461c8fb`; não é defeito do catálogo. |
| 257 | xshader: admit proven pixel spill lifetimes and partial attribute layouts | draft; `rewrite/lanes.rs` +49, `params.rs` +44: aceita spill comprovado e layout parcial de atributos. | Não. |
| 256 | xshader: permit proven bounded pixel code-table offsets | draft; `emitter/buffer.rs` +6, `check_pixel_code_offset.py` +87. | Não. |
| 255 | launcher: identify staged provider before comparing revisions | draft; `launch_provenance.py` +39 e teste +82. | Não. |
| 254 | dxbc: recover corroborated descriptors in later compute tables | draft; `compute_table.rs` +14 e teste +50: recupera descritores corroborados em tabelas compute posteriores. | Vizinho de D-14 (tabela de slots deslocada); não é a mesma tabela. |
| 253 | d3d12: import non-contiguous XMem backings through a reserved buffer | `device/allocation.rs` +88, `heap/backing.rs` +156, `heap/import.rs` +132: backing não contíguo via buffer reservado. | Continuação de `4461c8fb`. |
| 252 | xshader: resolve inline compute root SRVs by the actual root parameter | draft; `emitter/resources.rs` +38, `check_raw_index.py` +28. | Vizinho de D-12 (stride inline incompatível); caminho diferente. |
| 250 | xshader: lower cube-image samples with a LOD clamp | `check_translator_gpu.py`; amostragem de cubemap com clamp de LOD. | Vizinho de D-16 (MIMG/LOD) do catálogo do `VItoriaBr`, não deste. |
| 249 | diagnostics: identify compute compile requests and sample memory pressure | draft; `compute.rs` +39, `shader_cache.rs` +28, `big_stack.rs` +12. | Não. |
| 248 | xg: coalesce contiguous elements in tiled color texture transfers | draft; `element.rs` +28, `copies.rs` +8. | Não. |
| 247 | launcher: reuse one bounded verified DXIL snapshot | draft; `dxil_store.py` +18, `shader_blob.py` +39. | Vizinho de D-27 dos caches; não. |
| 246 | d3d12: restore missing literal samplers in mesh pipelines | `mesh_table.rs` +59, `inline_sampler.rs` +88 e teste +161. | Vizinho de D-21. |
| 245 | xshader: trim paired Wave64 arithmetic register and mask shadows | draft; `paired_wave64.rs` +44, `check_paired_wave64.py` +31. | Não. |

## Pull requests mesclados (24)

Todos com `mergedAt > 2026-11-26T04:15:00Z` (cursor de mesclados) e merge commit dentro de
`ac4f0993..b3184d35`. A coluna de conteúdo usa o conjunto de arquivos do merge contra o primeiro pai; hunk
lido só quando o commit de conteúdo caiu dentro dos 40 documentados.

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| 263 | Fixed copying per test case because of mid-test timeouts. | cd366ab5 | Só `tools/check_d3d12_regressions.py`: cópia por caso de teste em vez de uma só, por causa de timeouts no meio do teste. |
| 244 | nix: add vulkan-tools to the dev shell | c828fa50 | Só `flake.nix`. |
| 241 | d3d12, dxbc: give literal sampler states the root lacks private registers above its own | e328023a | 9 arquivos: `compute_table.rs`, `inline_sampler.rs`, `mesh_table.rs`, `stream.rs` e fixtures — registradores privados para estados de sampler literal. |
| 237 | docs: update ProjectScarlett status for descriptor reclaim and NVIDIA… | cfc56eb8 | Só `docs/PROGRESS.md` e `docs/PROJECTSCARLETT.md`. |
| 230 | devcontainer: add a Dockerfile-based development container | f3149ab6 | 7 arquivos novos em `.devcontainer/` (Dockerfile, post-create, start-display). |
| 226 | d3d12 tests: make mid-list split close regression work under vkd3d-proton | e1451068 | Só `tests/commands/memory_wait_regressions.rs`. |
| 225 | xshader: check group shared accesses against the guest's LDS allocation | 3b8a220f | 9 arquivos: `emitter/exchange.rs`, `emitter/lds.rs`, `emitter/output.rs`, `check_lds_bounds_gpu.py` novo — inclui o conteúdo de `6f510a52` e a ferramenta de `25e0610a`. |
| 224 | launcher: carry the shader cache over to a new Proton's prefix | 9b31fd8c | `project_tuxbox/host.py` + teste — conteúdo de `0072fe18`. |
| 223 | d3d12: say why a pipeline has no paged descriptor layout | 00ba85ff | `compute_bindings.rs` + `recording.rs` — conteúdo de `c3c95b2b`. |
| 222 | tools: select the translator checks' adapter by device UUID | 4d3dd267 | `check_translator_gpu.py` + teste: filtro por UUID do device. |
| 221 | verify: present through X images when --adapter is not Xwayland's GPU | d1112f92 | 4 arquivos — conteúdo de `a8f2b9c1`. |
| 220 | d3d12 checks: free the memory under an imported buffer | 1ef77321 | 3 arquivos — conteúdo de `70d71fbb`. |
| 218 | d3d12: import XMem physical pages through their backing | ba193dd4 | 6 arquivos em `device/heap/` — conteúdo de `4461c8fb`. |
| 217 | d3d12: accept slot 62 copies of textures padded to a whole tile | 771a4e5b | `copy_memory.rs` + teste — conteúdo de `12ddd28b`. |
| 216 | ProjectScarlett start menu: compute roots, faster compiles, watchdog pause | ed742e8c | 57 arquivos, +1 667/−103: `big_stack.rs`, `clzero.rs`, `compute_bindings.rs`, `descriptors.rs`, `runtime_shaders.py` e vizinhos — o conjunto `736a58a2`/`12255e4e`/`6f28b2db`/`7e0ad65c`. |
| 214 | dxbc, xshader: read four-word typed UAVs in a mesh pipeline's pixel layout | bb01537b | 4 arquivos — conteúdo de `724cd950`. |
| 213 | fix(tools): rank pipeline failures by the recorded refusal reason | c71d4d7a | 2 arquivos — conteúdo de `85bcf0ec`. |
| 212 | host: also hide VK_EXT_descriptor_heap on GPUs that need storage CBVs | 773278ad | 3 arquivos — conteúdo de `fceed400`. |
| 211 | shader-compiler: support ds_swizzle_b32 in paired_wave64 | 31dd5a3f | 3 arquivos — conteúdo de `915eee7b`. |
| 210 | shader-compiler ignore v_nop in IDXEN stride | 70cca743 | `emitter/buffer.rs` + teste: `v_nop` ignorado na inferência de stride do IDXEN. |
| 204 | shader: make exact wave reductions and DPP8 exchanges portable across workgroup shapes | 872c1989 | 14 arquivos: `check_dpp8_exchange.py`, `check_wave_guard.py` e evidência JSON — conteúdo de `e4fdfe4c`/`bd722d00`. |
| 197 | xshader: read whole lane masks where a program counts one | 1c9a1bdd | 17 arquivos: `check_mask_ballots_gpu.py` novo, `examples/mask_ballots.rs`, `docs/SHADERS.md`. |
| 185 | xshader: lower constant-lane v_readlane_b32 for Wave32 and portable Wave64 | f11b269d | 16 arquivos: `check_readlane_gpu.py` novo, `readlane_probe.rs`, `hlsl/driver/pipeline.rs`. |
| 181 | shader-compiler: Lower wave32 masked DPP prefixes and totals in wave allocation | 0eb91a8b | 8 arquivos: `wave_alloc.rs`, `wave_alloc_fixture.py`, `examples/lanes/selftest.cpp`. |

## Cruzamento com o catálogo

- **Reabre/refina D-04** (`Cache de pipeline da NVIDIA no lugar errado e com teto de tamanho`, introduzido
  em `9b18f7c`, corrigido em `d10b369`). `c6ce497b` derruba o teto de `32 GiB` para `4 GiB` e registra no
  código a medição: o cache do ProjectScarlett chegou a **13,7 GB** e o driver parou de atualizar o índice
  com ~5 GB, então todo pipeline compilado depois era recompilado a cada boot. O teto de 32 GiB era a
  correção anterior e estava acima do ponto de quebra do driver. O `corrigido` de D-04 (`d10b369`) continua
  sendo o conserto do lugar/teto; falta a este acervo decidir se D-04 recebe também `c6ce497b` ou se o
  ajuste de valor vira uma linha nova. Não gravei nada no catálogo.
- **Estende D-13** (`Slot 62 interpretado como upload de CPU`, introduzido em `3086e0b`, corrigido em
  `97b8ad3`). `12ddd28b` acrescenta a aceitação de buffer convidado com cauda de até um tile de 64 linhas
  (`tile_padded`), mantendo o limite de "uma pegada" para o resto. É ampliação do mesmo caminho de cópia,
  não regressão.
- **Estende D-11** (`Desligar VK_EXT_descriptor_buffer em todas as GPUs corrompe heap no RADV`, corrigido
  em `72766e7`). `fceed400` mantém a regra "só nas GPUs que precisam" e acrescenta `VK_EXT_descriptor_heap`
  à lista escondida — a correção de escopo de D-11 é preservada; nenhum caminho novo desliga as extensões
  globalmente.
- **Classe de D-02/D-03 reaparece no launcher** (`Otimizador do DXC estoura o tempo` / `Conversao DXIL leva
  minutos em cs_6_6 -O0`, ambos corrigidos em `e9c8fb6`). `7e0ad65c` registra que um teto **fixo** de 90 s
  recusava os kernels de ray tracing (~2,5 MB de HLSL, ~77 s cada, até quatro em paralelo) em todo boot, e
  troca o teto por `90 s/MB` até 600 s. É a mesma falha de classe (orçamento de compilação curto demais),
  em outro lugar; o conserto entra no mesmo commit. Candidato a linha nova no catálogo, **NAO VERIFICADO**
  em hardware.
- **Abre defeito novo?** Dois candidatos, **NAO VERIFICADO**:
  1. `12255e4e` substitui a rotina de limpeza do título em RVA fixo (`0x4E8AA10`, fast path `0x60`) e
     assume que as cinco voltas `clzero` cobrem exatamente `[align_down(rcx+0x40, 64), +0x7400)`. O
     comentário argumenta que a cauda em `+0xEC` zera `xmm0`/`vzeroupper` e que os registradores voláteis
     (`rdx`, `r8`–`r10`) ficam com valores diferentes, mas sem consequência; nada disso é verificado por
     teste contra o binário retail além da comparação de bytes do `survey`.
  2. `65f46ac5` é um merge de `main` upstream com conflito resolvido em 6 arquivos (+6 725/−382). Merge
     com resolução manual é exatamente o padrão do defeito D-24 (`Resolucao de merge apagou lowering e
     quebrou o verificador`); aqui não há teste na própria faixa que confirme que as duas suítes de
     redução ficaram realmente verdes.
- **Fecha lacuna de leitura, sem hash:** `c4f682b4` limita o HLSL gerado a 8 MiB por causa do consumo de
  memória do DXC (~1,1 GB por MB em `-O0`). Shaders que passem disso passam a ser **recusados** em vez de
  traduzidos — o efeito visível é pipeline ausente, e o aviso é `Error::Unsupported`, não um fallback.

## Confiança

- **Verificado por diff:** os 40 hashes foram lidos em clone local `--filter=blob:none` de `brcly/TuxBox`
  (commit `b3184d35`), com patch `-U0`. Quatro merges de PR (`31dd5a3f`, `bb01537b`, `c71d4d7a`,
  `773278ad`) foram comparados com os commits que agregam: mesmos arquivos, mesmas estatísticas.
- **Cobertura:** 40 de 84 commits da faixa. `last_commit` avança para `be9055c9`; `commit_cursor` =
  `be9055c9`, `head_at_run` = `b3184d35`. Os 44 restantes (a partir de `1d01c329`) ficam para o próximo
  ciclo, incluindo os merges #210/#204/#197/#185/#181.
- **PRs:** 20 de 29 abertos documentados (`last_pr_number` 210 → 266); 24 mesclados na janela. `last_run`
  avança para `2026-11-26T19:34:00Z` (mergedAt de #263), não para o horário da execução.
- **Divergência registrada:** 84 commits para 24 PRs mesclados. Parte da diferença é merge commit + commit
  de conteúdo para o mesmo PR (pares como `31dd5a3f`/`915eee7b`), parte são commits diretos de
  `Bas Bijsterbosch` que não passaram por PR. Não atribuí cada commit direto a um PR.
- **NAO VERIFICADO:** os dois candidatos a defeito novo; o comportamento real do hook de `clzero` em
  ProjectScarlett 1.0.2.1; a fidelidade dos novos caminhos de LDS e de `tb_swizzle` em títulos além dos
  fixtures; os 20 PRs abertos não documentados e os 44 commits restantes.
