# brcly/TuxBox — atualização de 2026-11-27 17:00

## Resumo

Quatorze commits novos desde `76b7bafe`, nenhum deles no documento das 16:00. Doze chegaram agora pelo merge `497c561a` (#272), da branch `work/mesh-validation-20261126` — são commits datados de 26/11 20:53 a 23:40 que só entraram na `main` neste ciclo, mais um `d3d12` de espera de fila que entrou pelo merge `6ca0d4ac` (#277). Na ponta, dois commits diretos: `18736af5` (13:55:49), que faz o tradutor manter a wave junta em leitura de lane dentro de laço divergente, e `76d1abe0` (14:07:01), que apaga o stand-in de mesh de desktop. A verificação de mesh passa a exigir recibo do worker para os binários realmente carregados e proíbe tradução sem root e stand-in.

## Commits novos (14)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `86cd032043` | Valdi320 | verify: render translated NGG shared-count mesh shaders | Acrescenta um fixture de mesh NGG sintético (`tools/reference/mesh/translated_shared_count.s`, 73 linhas: `v_mbcnt_lo/hi`, contagem de saída compartilhada, só a lane 0 emite triângulo vivo) e um readback de GPU (`tools/check_translated_mesh_gpu.py`, 134 linhas, com o estágio de pixel escrito à parte) que testa a saída do tradutor, não o ABI de lista de comandos. |
| `f07e19500f` | Valdi320 | verify: isolate the deliberate compute stand-in policy | Em `project_tuxbox/verify.py`, o gate de stand-in de compute passa a salvar e restaurar também `TUXBOX_NULL_PIPELINES`: o teste de um programa recusado deixa de herdar a política do jogo real ou de outro gate. Só isso — 4 linhas de código de produção, o resto em teste. |
| `7fac1f9627` | Valdi320 | Merge remote-tracking branch 'origin/main' into work/mesh-validation-20261126 | Merge que traz a `main` para dentro da branch de validação: 59 arquivos, +7.771/−380. Entram os controles de f64 (`check_f64_modes_gpu.py` 408 linhas, `float64.rs` +262), a tabela de mesh do `selftest` (`slot_dispatch_mesh = 129`), heap backing (`heap/backing.rs` +156/−29), `check_image_cmpswap_gpu.py`, e o gate `lanes-bindless-*`. |
| `ee9b6a6552` | Valdi320 | verify: exercise guarded mesh half-register position exports | O fixture de mesh ganha os casos `half-3`, `half-67` e `half-128`: posição exportada como meia palavra com guarda (`v_cmpx_gt_u32 s20, v1`, `v_cvt_f16_f32` e volta a `v_cvt_f32_f16`) antes do export. |
| `4dfd008938` | Valdi320 | verify: require root-aware translated mesh shaders in Lanes | `raster_fixture.py` troca o `MESH_CODE` de uma instrução `s_endpgm` (o programa de desktop substituía) por um NGG sintético real de 67 vértices com export de posição em meia palavra; `verify.py` ganha `missing_mesh_worker_receipt`, que exige que a SHA-256 reportada como "prepared <stage> shader loaded" seja resultado de worker para mesh e pixel. |
| `0ca25e6521` | Valdi320 | verify: isolate raster gate null policy and document translated mesh fixture | O gate de raster passa a fixar `TUXBOX_NULL_PIPELINES=1` no ambiente do teste, e o README de `examples/lanes` é reescrito: os dois estágios do cenário `mesh-samplers` são traduzidos pelo worker contra o root refletido, sem shader de mesh preparado antes do launch. |
| `6ea8f39ac2` | Valdi320 | d3d12: coalesce redundant placed texture queue waits | Novo `runtime/d3d12/src/device/placement/waits.rs`: `include()` junta esperas da mesma timeline privada guardando o maior tick e mantendo a ordem da primeira aparição; `placement.rs` chama isso no `submit`. O teste fixa `[(1,4),(2,8),(1,2),(3,5),(1,7),(2,9)] → [(1,7),(2,9),(3,5)]`. |
| `d171e755c4` | Valdi320 | Merge remote-tracking branch 'origin/main' into work/mesh-validation-20261126 | Segundo merge da `main` na branch, com conflito em `examples/lanes/selftest.cpp` (+170/−8). Traz o cenário `lanes-bindless-compute` (`BINDLESS_INDEX = 80`, `BINDLESS_EXPECTED = 0x7AB1E002`) e o store de UAV cru indexado fora do limite paginado. |
| `3296860409` | Valdi320 | Merge commit 'f07e195' into work/mesh-validation-20261126 | Reaplica na branch o isolamento de `TUXBOX_NULL_PIPELINES` de `f07e1950` depois do conflito do merge anterior. Mesmas duas alterações de `f07e1950`. |
| `bd7fd5a5df` | Valdi320 | Merge current main while retaining lane-spill and translated mesh checks | Terceiro merge da `main`, 27 arquivos, +765/−78. Entram o teste de atributo de pixel em metade alta que não deve ler o VGPR de spill (`hlsl.rs`), o fixture de spill (`examples/lanes/spill_fixture.py`), `runtime_shaders.py` +38/−12 e `check_wave_guard`. |
| `18736af5d8` | brcly | xshader: keep the wave together for lane reads in divergent loops | Um `v_readlane_b32` dentro de laço que as lanes deixam em momentos diferentes era recusado (traduzido por lane, a leitura perderia a lane que saiu). O programa de compute de 32 lanes passa a ser traduzido uma segunda vez com a wave junta (`lockstep.rs` +72, `retry`), e esse modo passa a contar lanes de máscara com `s_bcnt1_i32_b32` sobre a wave, aceitar `s_andn2_wrexec_b32`, baixar `DPP8` e o `DPP16` de espelho de linha usados por esses programas e ler o destino escalar do `v_readlane_b32`. Dois programas novos no check de lockstep. |
| `6ca0d4aca2` | brcly | Merge pull request #277 from Valdi320/placed-wait-coalescing | Merge de #277: traz `6ea8f39a` para a `main` (o mesmo `waits.rs`, +63/−8 em 4 arquivos). |
| `497c561aca` | brcly | Merge pull request #272 from Valdi320/mesh-verification | Merge de #272, 13 arquivos, +525/−51, com conflitos em `examples/lanes/README.md`, `selftest.cpp`, `project_tuxbox/verify.py` e `tests/test_lanes_raster.py`. Traz a frente de verificação de mesh: fixture NGG real, readback de GPU, gate de recibo de worker e a proibição de tradução sem root/stand-in. |
| `76d1abe0da` | brcly | lanes: remove the unused desktop mesh stand-in | Apaga `examples/lanes/mesh_samplers.hlsl` (17 linhas): desde #272 os cenários de sampler de mesh traduzem o programa NGG do console, então o triângulo de tela cheia escrito à mão não é mais compilado nem usado. |

## Pull requests abertos (1)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #309 | shaders: pin the DXC option lookup in the -O0 fallback test | Fixa a opção procurada no teste de fallback `-O0`, para o teste não depender de como o DXC lista opções: +3/−1 em `tests/test_runtime_shader_links.py`, sem tocar código de produção. Aberto em 13:51:50 por `tzbkk` (o corpo do PR não foi lido; a proposta vem do título e do arquivo). | Área de D-03 (`Conversao DXIL->SPIR-V leva minutos em cs_6_6 compilado em -O0`) e D-02 (otimizador do DXC estourando o tempo). Não altera o caminho de compilação, só o teste que o exercita. |

## Pull requests mesclados (2)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #272 | verify: validate translated mesh shaders and explicit fallback policy | `497c561a` | A frente `mesh-validation-20261126` inteira: `translated_shared_count.s` e `translated_guest.cpp` como fixture NGG, `check_translated_mesh_gpu.py` para readback, `missing_mesh_worker_receipt` em `verify.py` exigindo que as SHAs de mesh e pixel carregadas sejam recibo de worker, `MESH_CODE` real no `raster_fixture`, e o README dos cenários. |
| #277 | d3d12: coalesce redundant placed texture queue waits | `6ca0d4ac` | Só `6ea8f39a`: junção de esperas da mesma timeline no `submit` de textura posicionada, com teste que prova o maior tick por fence e a ordem da primeira aparição. |

Os dois merges têm `merged_at` 14:07:09Z e base `main`; `18736af5` (13:55:49) e `76d1abe0` (14:07:01) são commits diretos, não PR.

## Cruzamento com o catálogo

- **D-21** (`Assinatura mesh/pixel incompativel derruba o draw`): #272 leva a exigência de tradução com root para mesh e pixel e o recibo de worker dos binários efetivamente carregados. É o mesmo par mesh/pixel de D-21, mas o diff não toca o caminho de assinatura — é verificação.
- **D-22** (`Failed to remap (-3) em roots raster com varias tabelas de descritor`): `0ca25e65` e `4dfd0089` mudam o cenário `mesh-samplers`, que é justamente uma pipeline de mesh com root próprio; o gate passa a proibir cair na tradução sem root. Área, não correção do remap.
- **D-16** (`Deadlock entre filas por espera de placement atras de um label`): `6ea8f39a` (#277) mexe na lista de esperas de placement — junta esperas redundantes da mesma timeline. É otimização sobre o caminho que D-16 corrigiu (`faf8b19`), não uma reabertura.
- **D-24** (`Resolucao de merge apagou lowering e quebrou o verificador`): os três merges da `main` na branch de trabalho (`7fac1f96`, `d171e755`, `bd7fd5a5`) são exatamente o tipo de resolução que gerou D-24. Nenhum dos três apaga lowering: `bd7fd5a5` tira 7 linhas de `hlsl/mesh.rs` e reescreve `wave_sum.rs` (+11/−16), e as duas mudanças entram junto com testes novos (`tests/hlsl/mesh.rs` +43, `tests/hlsl/wave_sum.rs` +24).
- **Defeito novo**: nenhum identificado por diff. Nenhum hash do catálogo casa com as colunas `introduzido`/`corrigido`.

## Confiança

Verificado: os 14 diffs foram lidos pela API (`commits/<sha>` com patch), e nenhum dos 14 hashes aparece no documento de `docs/updates/2026-11-27/16/TuxBox.md` — a afirmação "nada disso estava documentado" vem dessa comparação, não do assunto.

Não verificado: (i) nenhum `cargo test`, `python -m unittest` ou execução de GPU foi rodada; os números de linhas e as asserções citadas vêm do patch; (ii) `18736af5` tem corpo de commit longo e foi resumido — quatro capacidades adicionais do modo lockstep e dois programas de teste, sem ler cada teste; (iii) a descrição de #309 vem do título e dos arquivos, porque o corpo do PR não foi lido; (iv) o `compare` a partir de `76b7bafe` devolve commits com data de 26/11 porque eles entraram na `main` agora pelo merge `497c561a`, não porque o cursor tenha voltado no tempo.
