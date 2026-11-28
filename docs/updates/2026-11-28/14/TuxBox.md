# brcly/TuxBox — atualização de 2026-11-28 14:00

## Resumo

99 commits entre o `commit_cursor` `02fceb26` e o head `6f89916a`, dos quais 40 são documentados aqui
(os 40 primeiros não documentados na ordem do `compare`), e 30 merges na janela, 24 ainda não
documentados. A faixa é a continuação da frente *two-lane* (derivadas em quádruplas duas a duas,
prefix scan de 64 lanes, `one-of-N` buffer) e o começo da entrega do decodificador BCPack no
`xcrdapi`; no mesmo intervalo entram consertos de runtime com sintoma declarado: `#362` (clear de UAV
com handle de outra arena derrubava o jogo no prólogo), `#367` (`SEC_RESERVE` custava `VirtualQuery`
no wineserver e o título rodava a 0,5 fps), `#386` (carga raw fora de EXEC lia `+0x7ffff000` e
faltava a GPU), `#365`/`#377` (cancelamento e páginas preenchidas depois do enqueue) e o `#398`
aberto, que segura a espera de label quando a escrita ainda vai ser submetida.

## Commits novos (40 de 85 não documentados)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `d4f87228` | brcly | Merge pull request #352 from NickStallman/xshader/two-lane-one-of-n-buffer | Merge de duas linhas de teste (`raw_width.rs` +23) e 5 linhas em `paired_wave64.rs`: se a instrução tem `bsel` (descritor `one-of-N`), o emissor devolve o corpo sem expandir e cada cópia guardada é expandida para as suas lanes. |
| `1e5b389c` | brcly | Merge pull request #353 from NickStallman/xshader/two-lane-derivative-quads | `paired_wave64.rs` passa a aceitar derivadas implícitas quando o grupo é uma linha (`y == 1`) ou tem largura/altura pares, numerando o bloco `4k..4k+3` "dois a dois" (`v0__/v1__` calculados de `flat/4`); novo `tools/measure_compute_quads.py` mede a posição real das threads na wave, e o exemplo aceita grupo `XxY`. |
| `70f5b5ec` | brcly | Merge pull request #354 from NickStallman/xshader/prefix-scan-64-lane | `wave_alloc.rs` refina a análise de vida: acesso DS de 32 bits lê um registrador (não quatro), `MUBUF` lê o índice/offset pela largura de `mask & 3`, e um `v_` sem `64`/sem `cndmask`/sem `_ci_` lê o terceiro operando como um escalar. Com isso o prefix scan de 64 lanes (temporário cross-row e total da metade baixa) deixa de parecer vivo e vira `WavePrefixSum`; `lockstep.rs` exige máscara em EXEC para o `PREFIX`. |
| `4d40ba42` | brcly | Merge main into fix/d3d12-guids-ordinal-imports | Merge de 98 arquivos (+10.308/−474) da `main` para a branch de guids. Traz, entre outros, `first_active_lane()` no lugar de `WaveActiveMin((exec_lo & 1) ? lane : 0xffffffff)`, a lista `ALWAYS_DECLARED` (exec/vcc/scc/m0/nul) para não redeclarar especiais, a numeração "dois a dois" para grupos com linhas em `output.rs`, o carregamento D16 em `buffer.rs`, e a recusa de derivadas em grupo com mais de 16 threads que atravessa waves. |
| `b7f694a7` | brcly | Merge pull request #329 from KainanP/fix/d3d12-guids-ordinal-imports | `verify.py`: `guid_receipt(imports, provider)` resolve imports por ordinal (`#23`, `#54`) contra a tabela de exports do provedor; imports por nome continuam contando como antes. Testes novos em `test_verify.py` cobrem as duas formas. |
| `d5d2dc4e` | brcly | tools: read the two-lane registers as pairs in the float compare check | O check de comparações float passa a procurar `v7__` (o par indexado pela lane) em vez de `v7__0`/`v7__1`, que a forma two-lane deixou de emitir; sem isso todo programa de duas lanes lia zeros. |
| `3b5d5506` | Nick Stallman | xshader: add a split dispatch's group base to the group ID | Cria `tuxbox_dxbc::group_base` (registrador 0, espaço 32766, 3 palavras) e `xshader::group_base`: quando a raiz termina com esse CBV, a tradução injeta `cbuffer TBGroupBase` e reescreve `void main(uint3 gid : SV_GroupID, ...)` para `uint3 gid = tb_launch_gid + tb_group_base`, mantendo o corpo. Testes fixam o cabeçalho e o uso único de `SV_GroupID`. |
| `020f5b32` | Nick Stallman | d3d12: split compute dispatches over 65,535 groups in a dimension | `compute_split.rs` (novo) divide um dispatch com mais de 65.535 grupos por dimensão em vários dispatches, cada um com o `GroupBase` do console; até 1.024 partes, recusa acima. `recording.rs` usa `dispatch_with_root` quando há `split_root`; `compute.rs` cria o `GroupBase` a partir da raiz refletida/do sampler/do título e o desliga quando há `scene_root` (ray query). |
| `a9eb7d2e` | brcly | Merge pull request #345 from NickStallman/xcrdapi/bcpack | Merge do decodificador BCPack: 39 arquivos, +4.776/−18, com `codec/src/{rans.rs, record.rs, lanes.rs, bc7.rs, tables/bc1..bc7.rs}` e testes. As tabelas levam o cabeçalho do `bcpack-rs` (`2d6333b0`, MIT/Apache-2.0) e o SHA-256 do JSON de origem. |
| `014ecd02` | Pavel Kirilin | d3d12: retry root-less compute IDXEN with the heap stride table | Em `compute.rs`, se `compute_with_root` falha e `refused_for_heap_strides` é verdadeiro, reflete a tabela com strides (`reflected_with_strides`), traduz de novo contra ela, loga uma vez "translated a compute shader with the heap stride table" e marca a tradução anterior como superseded. |
| `a8614d4f` | Pavel Kirilin | Created lanes example | Acrescenta o cenário `lanes-compute-heap-stride` ao `verify.py` e ao `selftest.cpp`: programa compute sem raiz embutida, raiz 1.1 sem parâmetros, heap paginado (limite 64), IDXEN com stride vindo da palavra 1 do descritor; cada registro precisa começar com o marcador esperado e manter o "decoy". |
| `01332cfe` | KainanP | d3d12: refuse mesh and ray signatures the host cannot run | O perfil de host passa a registrar `mesh=`/`rays=` (só é possível **tirar** capacidade: `mesh=yes` é recusado), consultado por `host_feature` em `D3D12_OPTIONS5/7`; `features.rs` deriva o tier de mesh do perfil e `indirect.rs` devolve `E_NOTIMPL` para `DISPATCH_MESH`/`DISPATCH_RAYS` quando o host não roda, em vez de deixar o driver da Intel UHD falhar na criação da assinatura. |
| `dfbf8ecc` | brcly | xshader: refuse an operand select a 16-bit VOP3 instruction doesn't read | `int16::handles`/`operation` isolam as formas que leem `OP_SEL`; `vector.rs` recusa "unsupported 16-bit operand select" quando o word é VOP3 (`>> 26 == 0x35`), o mnemônico é de 16 bits e a forma não lê `OP_SEL` (SDWA rebaixado continua com as próprias palavras). Teste cobre `v_add_f16`, `v_cvt_f32_f16` e `v_cmp_eq_f16`. |
| `bb2f5fac` | brcly | xshader: clamp and fuse the mixed-precision multiply-add | `vector_half.rs` emite `mad(a, b, c)` e aplica `saturate` quando `CLAMP` (bit 15 do dword 0); uma constante float inline lida como fonte de 16 bits vira o half do valor (antes pegava a metade baixa do encoding, e 1.0 virava 0). O check de ALU ganha linhas de mix com clamp e com a constante. |
| `4c5f4bb3` | brcly | xshader: refuse SDWA fields a form doesn't apply | Duas recusas novas: `vector.rs` recusa qualquer SDWA nas formas traduzidas antes da seleção (`v_add_co_ci_u32`, `v_sub_co_ci_u32`, `v_subrev_co_ci_u32`, `v_fmac_f16`, `v_fmamk_f16`, `v_fmaak_f16`, `v_movrel*`); `vector_wide.rs` recusa clamp/OMOD SDWA quando a forma não os aplica (exceto `clamped_product24`). Teste com `v_add_co_ci_u32`, `v_rcp_f32` e `v_and_b32`. |
| `c199f307` | brcly | xshader: refuse every DS access to the global data share | `lds.rs` recusa `GDS` (bit 17) para **toda** instrução DS, não só as de 64 bits de compute; antes 25 instruções de 32 bits/byte/halfword liam group shared em vez do GDS. O teste de swizzle passa a esperar "GDS is unsupported" quando o bit está posto. |
| `29a79825` | brcly | xshader: refuse a flat load into group shared memory | `scalar_memory.rs` recusa `FLAT` com bit 13 (LDS) — a carga ia para group shared e o emissor escrevia os registradores; cargas de buffer já recusavam. Teste em `buffer_loads.rs`. |
| `a411a18f` | Nick Stallman | tools: check the operand forms of float multiply-adds, fractions and byte conversions | Novo `check_float_forms_gpu.py` (649 linhas) com formas de `v_mul_legacy_f32`, `v_fmac_f32`, `v_fma_f32`, `v_fma_mix_f32`, `v_fract_f32` e `v_cvt_f32_ubyte0..3`: fontes escalar/literal, abs/neg/CLAMP/OMOD, SDWA, op_sel/op_sel_hi, denormais; modelos seguem `MODE[7:0] = 0xc0` e registram alternativas onde o manual é omisso. Inclui o defeito conhecido do OMOD em produto no NVIDIA. |
| `284e08f8` | Nick Stallman | xshader: apply OMOD as an exponent adjustment | Troca `(e) * 0.5/2/4` por `tb_omod(e, k)`, que soma `k` aos bits de expoente do resultado já arredondado (flush de denormal de saída, overflow vira infinito, NaN/Inf passam). O helper só entra na tradução que o usa; o defeito conhecido do check é removido. |
| `a0c15d4b` | brcly | status: keep game names off the progress map | Remove o painel de milestones do título e reescreve o mapa e os tooltips para não nomear jogos ("Used in recorded shaders", listas de bibliotecas sem cenas); `progress_map.py` perde `FROSTY_GATES` e a leitura de `matrix-awakens-launch.json`; `docs/progress.svg` encolhe de 1.031 para 880 de altura. |
| `99449edc` | brcly | evidence: record instruction use and GPU checks from the harness run at a0c15d4 | Atualiza `evidence/shader-instructions/*.json`: árvore do tradutor `33c51742`, 44.938 traduções (eram 44.799), 59 checks (eram 57) e as instruções novas cobertas; `docs/INSTRUCTIONS.md` e o SVG são regenerados (690/1.085, 63,6%). |
| `0162c62f` | brcly | docs: record translator progress in the roadmap | Só `docs/ROADMAP.md`: T2 a três instruções usadas sem check, T3 com auditoria de modificadores concluída, T8 com branches de EXEC/VCC resolvidos por wave; figuras de tradução atualizadas (38.781 de 42.859 em 64 lanes). |
| `ccf17df0` | brcly | Merge pull request #357 from NickStallman/d3d12/split-large-dispatches | Merge do split de dispatch em 18 arquivos (+900/−18): `group_base.rs` nas duas crates, `compute_split.rs`, `commands/compute.rs`, `tests/test_large_dispatch.py` e o cenário nos exemplos. |
| `70b825a7` | brcly | Merge pull request #355 from s3rius/runtime/rootless-compute-stride-table-retry | Merge do retry por tabela de strides (7 arquivos): `compute.rs`, `compute_bindings.rs` com o log único de vínculo (`paged`/`identity`), `test_lanes_compute_heap_stride.py` e a fixture. O corpo diz que manteve os dois cenários de Lanes quando colidiu com o #357. |
| `bcef0f2d` | brcly | tools: run the last three instructions recorded shaders use in GPU checks | `s_swappc_b64` ganha um programa de vértice que começa com o prólogo de fetch (`s_swappc_b64 s[4:5], s[4:5]` + `s_inst_prefetch 0x3`); `s_setpc_b64` ganha um salto longo com `s_getpc_b64`/`s_add_u32`/`s_addc_u32`; `flat_load_ushort` lê as duas metades de uma palavra da tabela do próprio programa. |
| `df4f27c3` | brcly | evidence: every instruction recorded shaders use is run by a GPU check | `gpu-checks.json` passa a 372/372 instruções usadas e 693/1.085 no total; `docs/INSTRUCTIONS.md`, `ROADMAP.md` (T2 "Done: 372 of 372") e o SVG (76,4%) refletem a corrida do harness em `5aa6c39a`. |
| `aec7338e` | Jerkin42 | d3d12: map a UAV clear's CPU handle from whichever arena holds it | `descriptor_paging::host_handle` mapeia um handle de CPU pelo `copy_source` do registro (acha a arena onde ele estiver e segue registros inline); `depth::clear_uav` usa essa função em vez de mapear só quando o handle está na mesma arena do handle de GPU. Antes o handle convidado chegava cru ao driver e estourava em `d3d12core.dll` (primeiro clear do FSR4). |
| `88e60681` | Jerkin42 | d3d12: publish a descriptor write only to the tables that can hold it | `Arena::publish` guarda `longest` (maior range já inserido) e varre só `ranges.range((first - longest, 0)..(end, 0))` em vez de todas as tabelas residentes; como `longest` só cresce, continua sendo limite válido após evicção. O corpo mede 35% do CPU do jogo nos threads de submissão. |
| `e5c17a40` | brcly | d3d12: replace a compute stand-in when its compile finishes | `late_pipeline.rs` (novo) guarda o slot do stand-in e as bindings com que o pipeline real foi criado (incluindo `image_writes` e `group_base`); quando o compile termina, a próxima bind usa o pipeline real. `TUXBOX_PIPELINE_DEADLINE` aceita segundos fracionários e o cenário `late-pipeline` força um compile grande a estourar 10 ms. |
| `21f8525c` | ale | tools: hash files without hashlib.file_digest so the checks run on Python 3.10 | `check_d3d12_regressions.py` usa `project_tuxbox.inspect.digest`; `check_frame_profile.py` faz o streaming em `hashlib.sha256`; a CI passa a matriz 3.10/3.12 e o `CONTRIBUTING` diz 3.10 e 3.12. |
| `2f2acc87` | Jerkin42 | xmem: create physical-page sections committed | `shared_pages::Section` cria a seção já comprometida (`PAGE_READWRITE`), e só cai para `SEC_RESERVE` (comprometida de uma vez por uma view) ou para commit por view se a criação falhar. Motivo no corpo: `VirtualQuery` em view de `SEC_RESERVE` vira request ao wineserver mesmo com tudo comprometido, e o título pergunta ~3.700 vezes por segundo. |
| `86c93952` | Nick Stallman | xcrdapi: log why a request fails, and don't fail a read cancelled while it runs | `in_flight` guarda `(tag, AtomicBool)` das leituras tiradas da fila, então `CancelRequestsWithTag` alcança leitura em execução; leitura cancelada pula a escrita e completa sem erro. `diag::failure` registra estágio, HRESULT, motivo do decodificador, tamanhos e tokens das primeiras páginas (64 primeiras, depois cada 1.024ª). `decode.rs` passa a devolver `String` com o motivo (status do zlib, modo BCPack, OOM, erro do record). |
| `7bafe6b6` | brcly | xshader: let mixed lane storage programs use group memory, DPP and long jumps | No `mixed_lane.rs`, group memory endereçada por lane e DPP passam (o modo `resolve_wave32` fixa a wave de 32 lanes nas lanes do convidado); trocas por group memory, GDS e barreiras continuam recusadas. Só `v_readlane_b32` de um registrador que `v_writelane_b32` preencheu é reescrito; `wave_loops` aceita LDS que não seja `ds_swizzle/bpermute/permute` nem GDS; salto longo já resolvido vira branch. |
| `44ee2537` | Nick Stallman | xshader: run loops that meet the group at barriers as a group | `group_loops.rs` (novo): um laço cujo corpo tem *rendezvous* de grupo (voto `tb_page_*` ou leitura de lane) itera até nenhuma thread do grupo querer outra iteração, votado por palavra em group shared; a thread que saiu guarda o alvo de salto e pula o corpo inteiro (sem mudar estado) mas encontra todas as barreiras, recuperando o alvo depois. Votos de `wave_cull` que ficavam dentro da guarda do branch passam a ser tomados antes, e thread pulada vota falso; laço de grupo conta até 65.536. |
| `e81864f4` | Nick Stallman | tools: check a wave-voted mesh loop that runs a different count per guest wave | O check de mesh traduzido ganha `loop-3`, `loop-67` e `loop-128`: um laço sobre lanes restantes cujo teste vota na wave do convidado roda 1 vez na primeira wave de 64 e 9 na segunda; contagem errada move o triângulo dois texels para a direita. |
| `90897b2b` | Nick Stallman | xshader: a 32-bit scalar result writes one register in the waterfall proof | `descriptor_broadcast` deixa de tratar toda escrita que não é load como par de 64 bits: SOP1/SOP2/SOPK sem `64` e `v_readfirstlane_b32`/`v_readlane_b32` escrevem **um** registrador, então `s_addc_u32 s55` não apaga mais a máscara guardada em `s[56:57]` nem `v_readfirstlane_b32 s7` a de `s[8:9]`. Testes cobrem 32 e 64 bits. |
| `e4163178` | Nick Stallman | xshader: carry a table descriptor through a proven waterfall copy | `broadcast_word` aceita também `Sym::Desc` (descritor carregado de tabela em offset conhecido), não só origem de heap; `descriptor_size` trata a cópia provada como `s_mov_b32` e a leitura de volta como tão certa quanto a palavra copiada, exceto no teste do próprio laço (`v_cmp_eq_u*`). |
| `9de4d26b` | Nick Stallman | xshader: prove a waterfall copy made in both arms of an if/else | `restored` aceita `v: Option<u8>` e nova `narrows_exec` reconhece `s_and_saveexec_b*`, `v_cmpx` e `s_and_b64`/`s_andn2_b64` com EXEC; as cópias do descritor podem estar nos dois braços do if/else (o `s_mov_b64 exec, saved` do fim alarga a máscara de novo) desde que cada lane tenha passado por uma cópia. Teste monta save/else/join. |
| `cd90049e` | Nick Stallman | lanes: draw with a vertex program's heap sampler read through a waterfall | Cenário `raster-sampler-waterfall`: programa de vértice que carrega sampler do heap em índice constante, copia nos dois braços de um if/else e lê de volta num waterfall; o valor amostrado é o `w` da posição, então sampler errado encolhe o triângulo. O `selftest` passa a vincular também o sampler heap. |
| `94cabfa8` | brcly | launcher: start a new NVIDIA cache file before the index fills | `rotate_nvidia_shader_cache`: o índice do driver endereça o arquivo com offsets de 32 bits e para de achar o que passa de 4 GiB, então cada launch escreve num arquivo numerado (`tuxbox<N>`) e começa o próximo ao chegar a 1 GiB, lendo os outros como *read-only*; nada é movido ou apagado, e definir as variáveis à mão desliga a rotação. |

## Pull requests abertos (1 acima de #397)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #398 | d3d12: hold a label wait for a write that is still to run | A espera de 5 s (`STALL` em `memory_wait.rs`) libera uma fila mesmo quando a lista que escreve o label ainda não foi submetida — o título grava as listas de escrita e submete as de leitura antes. O PR mantém a espera enquanto houver lista gravada que escreve aquele label. 11 arquivos, +384/−38, base `main`. O corpo cita o primeiro frame com ray tracing em `a880dfd`, em que quatro filas de compute foram liberadas assim. | Não é defeito do catálogo; é a área de ordenação de submissão/espera. |

## Pull requests mesclados (30 na janela, 24 novos)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #388 | d3d12: name the queue and the error when a batch is rejected | `7b7d71e8` (11:19:37Z) | `submit.rs` +20/−4: a mensagem passa a nomear o tipo de fila e o HRESULT e a dizer que lotes posteriores são descartados. |
| #387 | d3d12: create address UAVs at byte offsets, as address SRVs are | `d68e556b` (11:19:36Z) | `address_uav_guest.cpp` +110: import no alinhamento apertado do host quando nenhuma página de 64 KiB é um elemento, com shader `address_uav.hlsl`; antes o `CreatePlacedRawUnorderedAccessViewX` abortava o título. |
| #386 | xshader: read a raw buffer only in lanes inside EXEC | `f9d3299e` (11:19:37Z) | `buffer.rs` +3/−5: toda carga raw testa o bit EXEC da lane; as lanes fora liam o root SRV (sem limites) em `+0x7ffff000` e faltavam a GPU. |
| #384 | d3d12: gather a batch's placed views once per heap | `8aa4339b` (11:10:24Z) | `placement.rs`/`descriptors.rs`: ranges amostrados e lidos são fundidos antes da varredura, um range de leitura coberto por um amostrado é pulado; mesmo conjunto de texturas/escritas. |
| #366 | tools: hash files without hashlib.file_digest so the checks run on Python 3.10 | `6c3da5c2` (11:10:24Z) | Os dois checks e a CI (matriz 3.10/3.12), como em `21f8525c`. |
| #378 | d3d12: give placed textures memory at first GPU use | `8fce45f0` (11:05:23Z) | `residency.rs` (novo, +180): textura colocada nasce como recurso reservado, sem memória; o primeiro batch/cópia/flush mapeia tiles num heap próprio via `UpdateTileMappings` na fila de upload, antes do upload. |
| #374 | launcher: keep each prefix's Mesa shader cache | `1d862d46` (09:22:22Z) | `host.py` +41: `MESA_SHADER_CACHE_DIR` vai para `shadercache` do prefixo e `MESA_SHADER_CACHE_MAX_SIZE=4G`; antes o cache global de 1 GB era esvaziado por um boot de ~1,2 GB. |
| #362 | d3d12: map a UAV clear's CPU handle from whichever arena holds it | `ac53f29e` (09:22:22Z) | O mapeamento por `copy_source` de `aec7338e`. |
| #383 | tools: execute recorded compute shaders in the shader harness | `b426c1aa` (09:22:22Z) | `shader_harness.py execute --baseline REF --sample N`: traduz a requisição com a baseline e com a árvore, executa as duas e compara a saída; `execute_recorded_shaders.py` e `guest.cpp` novos. |
| #381 | d3d12: read placed textures into a scratch buffer that is not zeroed again | `6c16526f` (09:22:22Z) | `placement.rs`: o scratch só cresce e cada leitura escreve todos os bytes; só mip com digest alterado é copiado para upload. |
| #380 | xshader: keep VCC when a paired-wave64 v_cmpx narrows EXEC | `b981142b` (09:22:22Z) | `paired_wave64.rs` deixa de reconstruir VCC a partir de ballot: no GFX10 `v_cmpx` escreve só EXEC; `host_profiles.txt` é refixado. |
| #377 | xcrdapi: read page-array entries the title fills after enqueueing | `3eb76bd5` (09:22:22Z) | `queue.rs` +111: entradas que estavam zeradas no enqueue são relidas na execução via `ReadProcessMemory`, preservando as não zero copiadas; log das 16 primeiras matrizes preenchidas tarde. |
| #372 | xshader: run loops that meet the group at barriers as a group | `3e1df03e` (09:22:22Z) | `group_loops.rs` e o laço de grupo descritos em `44ee2537`. |
| #370 | xshader: prove a heap descriptor copied in both arms of an if/else for its waterfall read | `163704d9` (09:22:22Z) | `descriptor_broadcast.rs` +174 e o cenário de raster com sampler waterfall. |
| #368 | d3d12: compose a present with several planes | `57347619` (09:22:22Z) | `present/compose.rs` (novo, +258): cada plano no back buffer, o primeiro opaco e os demais alpha-blended, usando source rect/destino dos bytes 24/32 do plano de 72 bytes; até quatro planos; antes `E_NOTIMPL` deixava a janela branca no modo Series S. |
| #367 | xmem: create physical-page sections committed | `f14905bf` (09:22:22Z) | A seção comprometida de `2f2acc87`. |
| #365 | xcrdapi: cancel reads already taken off the queue, and say why a read failed | `b4cd89ff` (09:22:22Z) | O cancelamento em voo e as linhas de motivo de `86c93952`. |
| #364 | xshader: apply OMOD as an exponent adjustment | `b7a35f1c` (09:22:22Z) | `tb_omod` e o check de formas float (`284e08f8`, `a411a18f`). |
| #363 | tools: check the operand forms of float multiply-adds, fractions and byte conversions | `f179c436` (09:22:22Z) | `check_float_forms_gpu.py` +649 e a referência em Python. |
| #361 | d3d12: publish a descriptor write only to the tables that can hold it | `248f76d4` (09:22:22Z) | O `longest` de `88e60681`. |
| #355 | d3d12: retry root-less compute IDXEN with the heap stride table | `70b825a7` (02:44:18Z) | O retry de `014ecd02` e o cenário `compute-heap-stride`. |
| #354 | xshader: lower a 64-lane prefix scan whose scratch only looked alive | `70f5b5ec` (01:17:41Z) | A análise de vida refinada de `70f5b5ec`. |
| #353 | xshader: take derivatives in the two-lane form, two by two in a group with rows | `1e5b389c` (01:17:30Z) | A numeração dois a dois e `measure_compute_quads.py`. |
| #352 | xshader: choose a one-of-N buffer once for both lanes in the two-lane form | `d4f87228` (01:17:19Z) | A escolha fora da expansão (1 programa recuperado no corpus, 0 regressões, segundo o corpo). |
| #392 | (já documentado no ciclo das 12:00) | `6f89916a` | Head atual; o commit é da janela mas o número já está em documento anterior. |
| #386, #356, #357, #345, #329, #351 | — | — | #386/#357/#345/#329 aparecem como commits acima; #356 e #351 já estavam documentados (12:00 e 13:00). |

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum por hash. Os hashes dos 40 commits e dos 30 merges foram conferidos contra
  as colunas `introduzido`/`corrigido` de `catalogo/defeitos.csv` (305 linhas): zero interseção.
- **Defeito novo?** Um caso com sintoma e causa no diff, sem entrada no catálogo hoje:
  - `#380`/`b981142b`: no GFX10 `v_cmpx` escreve **só** EXEC, mas o decodificador dá a toda VOPC um
    destino implícito `VccLo`; a forma two-lane reconstruía VCC por ballot e EXEC. O diff corrige; o
    defeito que existia antes (VCC inventado) não está catalogado.
  - `#386`/`f9d3299e`: carga raw sem índice era emitida para toda lane, e lane fora de EXEC lia o root
    SRV sem limites. É a mesma família do `XCL D-17`/máscaras, mas em outro repositório e sem entrada
    própria — candidato a entrada nova.
  - `#398` (aberto) descreve o caso oposto ao de `#365`: espera expirada libera trabalho cuja escrita
    ainda não foi submetida. Se for confirmado no merge, é candidato a defeito novo.
- **Área compartilhada (hipótese):** `#367`/`2f2acc87` e `#377`/`86c93952` mexem em paginação de
  memória e I/O assíncrono, áreas cobertas por `TuxBox D-17`? Não: D-17 do TuxBox é o `#[cfg(test)]
  mod tests` duplicado no `victory-gpu`. Não há interseção de arquivo com nenhum defeito catalogado.

## Confiança

**Verificado no diff:** os 40 commits e os 24 merges novos, com hash, autor, data, arquivos e linhas;
o corpo de cada PR citado; o estado do #398 (base, arquivos, corpo). Os consertos com sintoma descrito
(#362, #367, #377, #386) foram lidos no diff, não apenas no corpo.

**Não verificado:** nenhum teste foi executado nesta coleta; as medições citadas (0,5 fps, 1,2 GB de
cache Mesa, 35% do CPU, 524 de 650 stand-ins, 4.776 linhas de BCPack) são declarações dos corpos ou de
mensagens de commit. A alegação de `#372`/`44ee2537` de que o laço de grupo explica o *hang* da GPU é
marcada pelo próprio autor como inferência.

**Cobertura:** 85 commits não documentados na faixa; 40 entraram. 6 dos 30 merges já estavam em
documento anterior (#392, #356, #351 e os que aparecem como commit: #357, #345, #329 — o #357 está
listado como commit desta página e também como merge novo, contagem única em `state`). `last_run`
avança até `#388` (11:19:37Z), o maior `merged_at` coberto. Nenhum subagente; leitura apenas na API.
