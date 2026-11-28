# brcly/TuxBox — atualização de 2026-11-28 15:00

## Resumo

Quarenta commits (12 merge commits) e **onze PRs mesclados novos**, quase todos na mesma frente: ray
tracing de título que constrói em memória própria. A pilha `#320 → #375 → #376 → #394` entrou junta às
14:41Z e cobre o que o `vkd3d-proton` descartava — builds de BLAS em endereço cru do título, índices em
formato 53, posições empacotadas `R32G32_FLOAT` e as estruturas "desidratadas" que o
`XboxBvhBuilder::RehydrateRaytracingAccelerationStructureBatched2X` recebe. Às 15:06–15:07Z entrou a
leva de runtime (`#395`/`#398`/`#399`/`#400`) e a de ferramentas (`#397`, `#371`). Fora do ray tracing,
o achado com sintoma é `a5891254`: no caminho *paired-wave64*, `v_cmpx` reconstruía VCC e apagava os
bits das lanes que a comparação deixou de fora (GFX10 escreve só `EXEC`) — o mesmo padrão de `D-14` do
catálogo do XCL, em outro tradutor.

## Commits novos (40)

Janela: os 40 primeiros sem documento anterior na ordem do `compare` `1f44aec1...ff2c9f41` (92
commits à frente, 59 sem documento).

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `1eeb42ef` | gtb-sbe | d3d12: build acceleration structures that titles place in their own memory | Arquivos novos `commands/build_raytracing/guest_addresses.rs` (122) e `device/blas/guest_memory.rs` (187). `Buffer.native` vira `Option<ID3D12Resource>` (pool sem storage nativo, cada estrutura em storage device-local publicada no endereço do convidado); `views/addressed.rs` ganha `command_buffer_address_flags`; `scan_view` usa `structure_pool`. `examples/lanes/selftest.cpp` +117 com o cenário `ray-tracing-guest-memory`. |
| `2e6d675f` | gtb-sbe | tools: expect refused AS builds to be skipped in the BLAS control | `tools/reference/blas/guest.cpp` +8/−6: desde `e8af9fd` (o conserto de **TuxBox D-15**) um build que o provider não consegue gravar é omitido e o resto da lista fica, então `Close` tem sucesso; o controle ainda esperava erro e parava antes de qualquer ray query. |
| `7284c791` | gtb-sbe | d3d12: publish guest-memory AS storage when the build is recorded | `device/blas/guest_memory.rs` +9: o pool escolhe o storage na gravação e o publica, porque uma cena pode ser submetida antes da lista que constrói uma de suas estruturas (quando essa lista espera um fence do título). |
| `561b8059` | gtb-sbe | bvh: log refused rehydrations | `runtime/bvh/src/lib.rs` +46: rehidratação devolve `E_NOTIMPL` e nada é construído nos destinos; agora um log limitado nomeia os primeiros destinos. `Cargo.toml` sobe a versão do `windows`. |
| `5ff179c5` | brcly | d3d12: paint the package splash into the title's window | `present/splash.rs` −308/+148, `frame_present.cpp` +16/−47, `frame_present.h` −2, `present.rs` −7: a splash deixa de ser uma segunda janela transparente sobre a do título (sob Hyprland a janela do título ficava branca e o compositor recortava a splash) e passa a ser pintada numa thread na própria janela; `present.rs` deixa de adiar o swap chain enquanto `created`. |
| `c50867c7` | brcly | launcher: launch with the umu id 'tuxbox' so game windows are named as games | `project_tuxbox/host.py` +2/−2 e `backend_log.py` +7/−2: id `default` → `tuxbox`, para as regras de janela de jogo do desktop casarem. |
| `a81bdd6e` | Jerkin42 | d3d12: compose a present with several planes | Arquivo novo `present/compose.rs` (258). `PresentX` com mais de um plano devolvia `E_NOTIMPL`; no modo Series S o ProjectScarlett apresenta a imagem 3D escalada e a interface em tamanho cheio como dois planos e a janela ficava em branco. O compositor desenha as camadas no back buffer. |
| `dd147122` | brcly | launcher: rotate the NVIDIA cache on whichever of its files is full | `host.py` +11/−6, `tests/test_host.py`: o driver anexa um dígito ao nome do cache e mantém vários arquivos; a rotação olhava o que termina em 0 enquanto uma sessão escrevia 3 GB no que termina em 2. |
| `acaacf3c` | brcly | launcher: limit glibc arenas for Proton and Wine launches | `host.py` +18: `MALLOC_ARENA_MAX`/`MALLOC_MMAP_THRESHOLD_`/`MALLOC_TRIM_THRESHOLD_` no lançamento; `docs/ENVIRONMENT.md` e teste com `MallocLimits`. |
| `ce43fdb7` | Jerkin42 | launcher: keep each prefix's Mesa shader cache | `host.py` +41: `MESA_CACHE_SIZE = "4G"` e cache por prefixo, porque um boot do ProjectScarlett em RADV escreve ~1,2 GB contra o teto padrão de 1 GB compartilhado com o desktop. |
| `60710667` | brcly | d3d12: remove the splash code the repaint left unused | `present/splash.rs` −27 (`UNLAYER_WAIT`) e `present/window.rs` −6 (`work_area`) — limpeza do que o repaint de `5ff179c5` deixou. |
| `cba382c4` | brcly | d3d12: compare placed textures by digest instead of keeping a copy | Arquivo novo `device/placement/digest.rs` (40): cada textura posicionada guardava uma cópia exata dos bytes do convidado para detectar mudança; agora um digest de 256 bits por mip, com o custo de memória de uma textura a menos por textura. |
| `7886a3ce` | Jerkin42 | d3d12: decode meshlet-coded indices in title-memory BLAS builds | Arquivo novo `build_raytracing/meshlet_indices.rs` (94), `FORMAT = 53`. O `geometry()` dimensionava qualquer formato que não fosse `R16_UINT`/`R32_UINT` como 0 byte, então o endereço do título nunca era resolvido e chegava ao build do host como índice; agora o stream é decodificado e o range dimensionado pelo maior índice. |
| `c800b907` | Jerkin42 | d3d12: decode meshlet-coded indices only in title memory | `guest_addresses.rs` +2: o stream é lido na CPU no endereço; se o endereço já é conhecido pelo host, é um endereço de GPU — recusa com `E_NOTIMPL` em vez de ler a CPU no mesmo número. |
| `02f9e102` | Jerkin42 | d3d12: decode packed positions in title-memory BLAS builds | Arquivo novo `build_raytracing/packed_positions.rs` (83). Para `R32G32_FLOAT` o kernel do console lê registros de 8 bytes com três coordenadas quantizadas, escaladas de uma origem num cabeçalho de 32 bytes antes do buffer de vértices; os records são decodificados para `R32G32B32_FLOAT`. |
| `6e994ab9` | Nick Stallman | xcrdapi: read page-array entries the title fills after enqueueing | `runtime/xcrdapi/src/queue.rs` +47/−5, `runtime/xcrdapi/guest.cpp` +43: a matriz de páginas era copiada em `EnqueueRequest`; o ProjectScarlett preenche algumas depois e antes do `Submit`, e esses requests escreviam na página 0 e falhavam com `ERROR_INVALID_ADDRESS`. Agora a leitura é no run, com `ReadProcessMemory`; `diag.rs` ganha `tokens()` e o check de `verify.py` passa a esperar `pages=2`. |
| `e1f407b9` | NathansDomainSolutions | d3d12: give placed textures memory at first GPU use | Arquivo novo `device/placement/residency.rs` (180): `Residency::Committed` × imagem reservada; `Texture` ganha `residency` e o `committed(...)` de criação sai do caminho. `TUXBOX_LAZY_TEXTURES` (padrão ligado). O corpo mede: numa placa de 12 GB, 99,6% da memória retida por cópias nativas na tela de carregamento vinha de views sobre texturas que o título talvez nunca desenhe. |
| `1eb8ac9a` | Jerkin42 | d3d12: read placed textures into a scratch buffer that is not zeroed again | `placement.rs` +21: `snapshot_mip` lia para um scratch que era limpo e preenchido com zero até o tamanho do mip antes de cada leitura, e um mip alterado tomava o buffer; agora o buffer só cresce e os bytes são sobrescritos pela leitura. |
| `a5891254` | NathansDomainSolutions | xshader: keep VCC when a paired-wave64 v_cmpx narrows EXEC | `hlsl/paired_wave64.rs`: `written` passa a ser `Vec::new()` quando a instrução é `v_cmpx_*` — o decoder ainda nomeia VCC como destino da comparação, e reconstruí-lo apagava os bits das lanes que a comparação deixou de fora (RDNA2 escreve só `EXEC`). `tools/check_paired_wave64.py` ganha o caso `cmpx_vcc`; `host_profiles.txt` tem 19 linhas refixadas. |
| `d828a00d` | MathiewMay | xshader: read a raw buffer only in lanes inside EXEC | `hlsl/emitter/buffer.rs`: o caso `mask & 2` (load com guarda de `exec_lo & 1u`) deixa de ser exceção e o load passa a ser guardado sempre; uma lane fora de EXEC não faz acesso no console, e num root SRV não há limite para conter o endereço. Teste novo em `raw_width.rs` que conta loads fora de EXEC. |
| `61d316aa` | brcly | d3d12: upload changed placed textures in batches | `placement/upload.rs` +21/−4: o flush lia todos os mips alterados antes de montar qualquer staging e enviava tudo num buffer; o ProjectScarlett altera mais de 26.000 mips (1,5 GB) num flush. Agora envia a cada 128 MB lidos. |
| `6c1617c7` | MathiewMay | d3d12: name the queue and the error when a batch is rejected | `commands/submit.rs`: `rejection_text` nomeia o tipo de fila e o `HRESULT` (antes qualquer fila dizia "direct queue rejected…") e avisa que a fila descarta os lotes seguintes; teste novo `tests/commands/submit.rs`. |
| `5c83265a` | Nick Stallman | d3d12: gather a batch's placed views once per heap | `placement/descriptors.rs` +66/−15 e `placement.rs`: `DescriptorMap<PlacedView>` com `coalesce` por heap; antes cada lista de um lote de 25 listas sobre um heap de 33.000 texturas nomeava o heap inteiro e `Submit` ordenava os duplicados fora. |
| `a93688b9` | NathansDomainSolutions | tools: execute recorded compute shaders in the shader harness | Arquivos novos `tools/execute_recorded_shaders.py` (219), `project_tuxbox/shader_execute.py` (337), `tools/reference/execute-shaders/guest.cpp` (374). O harness traduzia e compilava mas nunca executava, então uma tradução que ainda compila podia computar outra coisa. Fecha o item T4 do roadmap (#379) com amostra comparada entre builds e placas. |
| `4e8b3e13` | Nick Stallman | d3d12: reuse readback buffers and skip publications that change nothing | Arquivos novos `placement/spares.rs` (92) e `placement/readback_stats.rs` (211); `writeback.rs` +165/−45. Cada lote que escreve texturas posicionadas criava um readback buffer novo com allocator e lista: numa RTX 5090 um de 925 MiB custava 52 ms no thread submissor. Os `Parts` ficam para a próxima publicação da fila; `TUXBOX_LOG` informa quem foi lido de volta e se a publicação mudou bytes. |
| `3eb76bd9` | brcly | Merge pull request #377 from NickStallman/xcrdapi/page-array-at-run | Merge de dois pais; os 4 arquivos são exatamente o `6e994ab9` (`queue.rs`, `guest.cpp`, `diag.rs`, `verify.py`). Nada além disso. |
| `bed62ec7` | MathiewMay | d3d12: create address UAVs at byte offsets, as address SRVs are | Arquivos novos `runtime/d3d12/address_uav.hlsl` (13) e `address_uav_guest.cpp` (110). `CreatePlacedRawUnorderedAccessViewX` (slot 71) exigia import começando em elemento inteiro; o caso do ProjectScarlett é um UAV de dois registros de 20 bytes 432 bytes dentro de uma alocação, que não cai em página de 64 KiB nem em fronteira de 128 bytes. `views/addressed.rs` passa a aceitar o start em byte. |
| `c9fd064d` | Nick Stallman | d3d12: evict descriptor tables in one pass and keep submission off the table lock | `descriptor_paging.rs`: `TableEntry.pins` vira `Arc<AtomicU32>` compartilhado com `Arena::pins`, `lookup.rs` usa `tables.touch(key, tick)` em vez de `get_mut`, e entra `time::{Duration, Instant}`. A evicção era quadrática (varria todas as tabelas residentes por tabela que não cabia) e rodava sob o lock `tables`, que `ExecuteCommandLists` também toma para pinar. |
| `a4747456` | Jerkin42 | d3d12: read a packed-position stride through its low 14 bits | `guest_addresses.rs` usa `packed_positions::stride(StrideInBytes)`; o builder do console lê o stride por um campo de 14 bits e o ProjectScarlett passa `0x8008` para registros de 8 bytes. Com o stride cru, um import de 180 vértices cobria 5,8 MB. Teste `the_builder_reads_only_the_low_14_bits_of_the_stride`. |
| `470b749a` | Nick Stallman | Merge upstream/main into bvh/rehydrate-as-build | Merge de 14 arquivos contra `cf0a0824`, incluindo `docs/INSTRUCTIONS.md` novo (1.525 linhas), `Cargo.toml`/`Cargo.lock` com `runtime/xcrdapi/codec`, CI com os testes do codec e o banner do README. É integração, não trabalho novo. |
| `928abcab` | Nick Stallman | d3d12: build rehydrated console structures as host builds | Arquivo novo `commands/build_raytracing/rehydrate.rs` (332); `runtime/bvh/src/lib.rs` registra o entry `Rehydrate` e `d3d12_xs.def` exporta `TuxBoxRehydrateAccelerationStructures`. O `XboxBvhBuilder` respondia `E_NOTIMPL` às estruturas que o título transmite "desidratadas" (pelo menos 256 por cena, 16 por chamada); cada entrada vira um build de host da geometria que a chamada nomeia, depois do decode de índices 53 e posições empacotadas. |
| `bf704d59` | Nick Stallman | tools: give HLSL size and translation time per host in the harness report | `shader_harness.py` +13 e `shader_survey.py` +12: por build e host, o tamanho do HLSL escrito e o tempo total/mais lento das traduções que passaram (tempo indicativo: a varredura roda em paralelo). |
| `d1e4aed5` | Nick Stallman | tools: measure each stage from a recorded shader to a ready pipeline | Arquivos novos `tools/stage_baseline.py` (366), `project_tuxbox/stage_baseline.py` (510), `tools/reference/stage-baseline/{guest.cpp (246), layer.c (295), layer.json, probe.py (86), sample.json (885)}`. Mede tradutor, DXC, DXV, vkd3d-proton e driver para uma amostra fixa de requests, com a camada Vulkan separando o tempo do driver. |
| `a880dfd0` | brcly | d3d12: keep a reserved image's heap while the transfer queue uses it | `placement/residency.rs`: `Held { _image: ID3D12Resource, _heap: Option<ID3D12Heap> }` e `Texture::hold()`; `upload.rs` guarda `Vec<Held>` em vez de `Vec<ID3D12Resource>`. Um tile mapping não referencia o heap e só a textura o possuía — um título que liberava a textura posicionada com upload ou mapping ainda na fila liberava a memória sob ele. |
| `7ebe5561` | brcly | progress: count shader programs instead of compile requests | `runtime/d3d12/src/progress.rs` +58/−37 e `project_tuxbox/shader_pack.py`: o ProjectScarlett pede um programa por root signature e por variante narrow, 46.226 requests para 19.017 programas; o painel contava requests. `TUXBOX_SHADER_PROGRAMS` fixa o total. |
| `8a5fb665` | Pavel Kirilin | tools: rank actionable refusals by pipeline impact | Arquivo novo `tools/refusal_impact.py` (143) e `tests/test_refusal_impact.py` (84): separa linhas que não são recusa do tradutor (deadline/falha de driver, stand-ins, não listados) e ordena os buckets acionáveis por pipelines bloqueados, não por contagem de requests. |
| `5425e373` | brcly | xshader: translate the signed group-memory minimum and maximum | `hlsl/emitter/lds.rs` aceita `ds_min_i32`/`ds_max_i32` (e as formas `_rtn_`) no mesmo caminho que `ds_min_u32`/`ds_max_u32`, comparando e trocando até nenhuma lane intervir (LDS é declarado sem sinal); `lds_order.rs` inclui os dois em `exchange_waits`; `docs/INSTRUCTIONS.md` passa o grupo DS de 36 para 40 traduzidas. |
| `10f54d84` | brcly | xshader: let a mesh program add to a number it multiplied into VCC | `hlsl/mesh.rs`: `validate_lane_ops` deixa de tratar toda leitura numérica de VCC como leitura de máscara quando `scalar_in_vcc(body, index)` prova que a mesma sequência reta deixou ali o resultado de um `s_mul_i32`; seis programas de mesh do ProjectScarlett usam `vcc_lo` como escalar de reserva. |
| `24b0dcbf` | brcly | xshader: choose pixel inputs by what the program reads when declarations fail | `hlsl/params.rs` +100: `declared_inputs` monta o layout a partir do que o programa lê (`BTreeMap`), usado quando a assinatura não bate com a contagem de hardware (o compilador do console tinha descartado parâmetro que a assinatura ainda diz ler). |
| `b9972eba` | Pavel Kirilin | Improved ranking according to PR comments. | `tools/triage_scarlett_shaders.py` +61: agrupa requests cujo *sibling* difere só na root signature e exclui a recusa quando o programa traduziu por outro root (o provider tenta vários roots para um pipeline, então uma recusa é muitas vezes tentativa descartada). `docs/SHADERS.md` registra o efeito: 4.383 traduzidos de 9.102 requests seguem, mas a contagem de recusas deixa de somar tentativas. |

## Pull requests abertos (0 acima do cursor)

Os 8 abertos do repositório têm número `≤ 393`, abaixo do cursor `last_pr_number = 398`; nenhum PR novo
foi aberto nesta hora. Os 8 não foram relidos nesta execução.

## Pull requests mesclados (12 na janela, 11 novos)

Todos com `merged_at` na janela `11:19:37Z → 15:07:14Z`. `#392` já estava documentado.

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #320 | d3d12: build acceleration structures that titles place in their own memory | `561b8059` (14:41:16Z) | 17 arquivos, +710/−66. O que entrou é o commit `1eeb42ef` (`guest_addresses.rs`, `blas/guest_memory.rs`, `Buffer.native: Option<_>`, `command_buffer_address_flags`) mais `2e6d675f`/`7284c791`. O `merge_commit_sha` do PR aponta para o **tip** da branch (merge por rebase/queue), cuja mensagem é "bvh: log refused rehydrations" — não é um commit de merge de dois pais. |
| #375 | d3d12: decode meshlet-coded indices in title-memory BLAS builds | `c800b907` (14:41:16Z) | 19 arquivos, +896/−66; a parte nova é `7886a3ce` (formato 53) e `c800b907` (só em memória do título). O resto é a pilha de #320. Confere com o corpo ("#320's head plus two commits"). |
| #376 | d3d12: decode packed positions in title-memory BLAS builds | `470b749a` (14:41:16Z) | 21 arquivos, +1.070/−66; a parte nova é `02f9e102` (records de 8 bytes) e `a4747456` (stride pelos 14 bits), mais o merge de `main` (`470b749a`). Confere com o corpo. |
| #394 | bvh, d3d12: rehydrate console structures as host builds | `dd432e6f` (14:41:16Z) | 26 arquivos, +1.923/−70; a parte nova é `928abcab` (`rehydrate.rs`, export `TuxBoxRehydrateAccelerationStructures`, `SceneCommand::Rehydrate`) e `bf704d59`/`d1e4aed5` de ferramentas. |
| #398 | d3d12: hold a label wait for a write that is still to run | `05c691da` (14:41:16Z) | 11 arquivos, +384/−38: a liberação de 5 s de uma fila presa num label (`STALL` em `commands/memory_wait.rs`) deixa de ser incondicional e passa a olhar o que o título está fazendo, porque o título pode submeter a lista que espera o label bem antes da que o escreve. |
| #395 | d3d12: evict descriptor tables in one pass and keep submission off the table lock | `ff2c9f41` (15:07:14Z) | 3 arquivos, +286/−97 = `c9fd064d` (pins atômicos, `touch`, evicção em um passe). |
| #397 | tools: per-stage baseline of translation and pipeline compile cost (T14) | `b1357dd8` (15:07:14Z) | 12 arquivos, +2.754/−3 = `d1e4aed5` (T14). Confere com o corpo: "só mede; nada muda como um shader é compilado". |
| #399 | d3d12: keep a large guest view heap in smaller CPU heaps | `ba02a79c` (15:06:22Z) | 8 arquivos, +250/−46. Corrige o gate `lanes-selftest-descriptor-reclaim` que falha numa Intel UHD com `create oversized view heap failed: 0x8007000e`: um heap de 2.097.152 descritores era feito num único heap CPU-only do tamanho do heap do convidado. |
| #400 | xcrdapi: build the read test again under MSVC /W4 /WX | `0f845250` (15:06:13Z) | 1 arquivo, +13/−12. `python build.py all` parava em `runtime/xcrdapi/guest.cpp(429): warning C4456: declaration of 'count' hides previous local declaration` (e em 459, idem); o teste volta a compilar sob `/W4 /WX`. |
| #396 | xshader: decide a loop's back edge for the whole wave too | `ddbbc1c8` (15:07:14Z) | 4 arquivos, +52/−3: `8724e8f` tinha transformado branches para frente em `WaveActiveAnyTrue` onde os ballots de máscara se aplicam, mas a **aresta de volta** de um laço estruturado continuava com o bit de cada lane (`hlsl/emitter/control_flow.rs`); agora a decisão é da wave. |
| #371 | tools: rank actionable refusals by pipeline impact | `d9ed737d` (15:07:14Z) | 7 arquivos, +431/−10 = `8a5fb665` (ranking por pipelines) e os 8 buckets fixos de recusas nomeadas que caíam em `other_refusal`. |
| #392 | d3d12: reuse readback buffers and skip publications that change nothing | `6f89916a` (12:07:42Z) | **Já documentado** em `docs/updates/2026-11-28/12/TuxBox.md`; não recontado. |

## Cruzamento com o catálogo

- **Corrige D-xx? Por hash, nenhum.** Os 40 hashes da janela e os 11 hashes de merge novos foram
  conferidos por prefixo de 12 contra as colunas `introduzido`/`corrigido` das 305 linhas do catálogo:
  zero interseção.
- **D-15, citado dentro de um commit.** `2e6d675f` é consequência do conserto de **TuxBox D-15**
  ("Escritas de fence do título perdidas e um comando derrubando a lista inteira", corrigido em
  `e8af9fd`): o controle `tools/reference/blas/guest.cpp` esperava erro no build recusado e passou a
  esperar que ele seja omitido com `Close` bem-sucedido. O catálogo não é alterado — é o mesmo defeito,
  e o commit é a adaptação do teste.
- **D-16 por área, sem hash.** `#398` (`05c691da`) mexe na espera de label entre filas, que é a área de
  **TuxBox D-16** ("Deadlock entre filas por espera de placement atrás de um label", corrigido em
  `faf8b19`). O defeito aqui é o oposto: a fila era **liberada cedo demais** depois de 5 s, não travada.
  Não afirmo que reabra D-16.
- **Abre defeito novo?** Não comprovado, dois candidatos com evidência no diff: (a) `e1f407b9` passa
  texturas posicionadas a imagem reservada com tiles mapeados na primeira submissão — o diff garante
  que toda utilização passa por `submit` ou flush de escrita da CPU, mas não há teste de uso após
  liberação; é a janela de **D-17** (liberação de backing) e **D-18** (aliasing de textura posicionada);
  (b) `a81bdd6e` compõe planos no back buffer e o corpo não mede o custo por frame.

## Confiança

**Verificado:** os 40 commits foram lidos por API com lista de arquivos, `+/-`, cabeçalhos de hunk e as
primeiras linhas alteradas; nove tiveram patch lido em extensão (`1eeb42ef`, `6e994ab9`, `e1f407b9`,
`a5891254`, `d828a00d`, `c9fd064d`, `a4747456`, `a880dfd0`, `5ff179c5`). Os 12 merges vieram de
`pulls?state=closed` filtrando `merged_at > last_run`, e cada `merge_commit_sha` foi conferido contra a
lista de commits da janela. Os corpos de #320/#375/#376/#394/#395/#396/#397/#398/#399/#400/#371 foram
lidos.

**Divergência registrada:** o documento de 2026-11-28 14:00 cita `3eb76bd5` como o merge do `#377`.
Esse SHA **não existe** no repositório (`commits/3eb76bd5` → 404); o merge real é `3eb76bd9`
(`merged_at` 2026-11-28T09:22:22Z). O hash documentado precisa de correção — não foi corrigido nesta
execução para não reescrever documento publicado.

**Não verificado:** nenhuma build, teste ou execução; as medições citadas (925 MiB / 52 ms, 99,6% da
memória, 46.226 requests para 19.017 programas, 0x8007000e no UHD) são relato do corpo dos PRs e
commits. Não abri o diff de `05c691da` (#398) nem `dd432e6f` (#394) além de `928abcab`; o que afirmo
sobre eles vem do corpo do PR e dos commits da pilha. Não sei se a pilha `#320→#394` funciona ponta a
ponta: o próprio #394 diz "it has not been shown to make the title's ray tracing work".

**Cobertura:** 40 de 59 commits não documentados. `last_commit` avança até `b9972eba`, o 40º da janela;
`commit_cursor` = `99eef7ab`; `head_at_run` = `ff2c9f41`. Os cinco merges de 15:06–15:07Z entraram
depois do início da coleta e estão no documento pela data; os commits que chegarem depois de
`ff2c9f41` ficam para o próximo ciclo.
