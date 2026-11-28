# trustno1-coder/VIctory-NVIDIA — atualização de 2026-11-28 19:00

## Resumo

Primeira hora documentada deste repositório no acervo, e a maior da rotina até agora: **70 commits** na
janela `93008750...32ecacfa` — 40 commits de conteúdo e 30 merges de PR, todos os merges entre 18:12Z e
18:22Z. O repositório tinha **30 PRs parados** e os mesclou em dois lotes, junto com dois `Merge main`
que trazem a `main` para dentro do ramo. O que o diff mostra de substantivo: o tradutor ganha passes
novos em `bitcode/opt.rs` (+4.665 linhas) e a análise de LDS estática (+3.482 em `emit/lds.rs`); o host
para de re-salvar a biblioteca de pipeline inteira em rajada (`a162ae7f`), o teto de VRAM dos alvos de
render passa a ser medido pelo pico (`05a269d8`) e o candidato a DLSS da Series S deixa de pegar FSR 4
(`ba360f5f`, 28,5 → 30,9 fps medidos pelo autor). Nenhum hash de catálogo na janela.

## Commits novos (70)

O `compare/93008750...32ecacfa` devolve 70 commits: 40 de conteúdo e 30 merges do tipo
`Merge pull request #N`. Os 30 merges estão na tabela de PRs mesclados; a tabela abaixo cobre os 40
commits de conteúdo, que é o teto de 40 commits documentados por repositório. A ordem é a do `compare`.

Dois deles (`0eb34b84`, `76e3c697`) são `Merge main: ...` — a `main` trazida para dentro do ramo. O
**diff deles contra o primeiro pai é o que a `main` já tinha**, não trabalho novo do ramo, e está marcado
como tal na tabela.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `00d5623c` | Vladimir-Human | perf(shader): add DXIL folds, pure LICM, local/shared reuse, and barrier dedup | `emit/bitcode/opt.rs` +4.665/-87: passes novos sobre o bitcode — dobras algébricas/bitwise/compare/select, LICM de código puro (bit 128), rastreio de carga/armazenamento por raiz de ponteiro (forwarding store→load em espaço 0 thread-private, reuso load→load em espaço 3 não aliasado, eliminação de store morto sobrescrito) e dedup de barreira adjacente (bit 256). O forwarding store→load em `groupshared` foi **deixado de fora de propósito** (store constante invalida o slot; store dinâmico limpa as cargas em cache sem inserir) para não quebrar o padrão sem barreira `lds[b] = lane; if (lds[b] == lane)`. `DEFAULT` vira 447, `ALL` 511 |
| `3f6ba011` | Ahyildiz | Textures: a prelude upload checks again at submission that the host has not written the texture (confetti faces) | `gpu/texture.rs` +40/-1: `PreludeUpload` passa a carregar o `resource`; `finish_refresh` consulta `host_written` **de novo** na submissão e deixa o upload de fora quando o host escreveu, em vez de copiar por cima a memória do título (que ele nunca preencheu — quem copia é a GPU do console). Antes, lista gravada em outra thread podia submeter depois da lista que escreve a textura e o rosto pintado ficava com manchas de cor pela cena inteira. Aviso nomeando tamanho e formato (8 primeiros, depois potências de 2) e `VICTORY_PRELUDE_RECHECK=0` restaurando o antigo |
| `af242dab` | k0ma2k15 | perf(gpu): snapshot placed-copy constants on GPU and enable `VICTORY_PLACED_ORDER` by default | `gpu/host/place.rs` +152: constantes lidas na placa direto da cópia colocada, com contador `CONSTANTS`; `draw.rs` +7 devolve a constante da GPU em vez de esperar; `host.rs` +16 guarda `has_placed` e os ranges escritos na GPU por `Step::Placed`; `card_steps.rs` +3/-5 muda o comentário de "não apareceu uma vez" para o caminho agora ligado por padrão |
| `759f05c1` | wike1701 | Shaders: hoist constant LDS accumulator initialisations behind an entry barrier | `emit/lower.rs` +151: `lds_init_hoist_on` (desligável com `VICTORY_LDS_INIT_HOIST=0`) sobe os stores constantes de acumulador em LDS para a entrada do programa, atrás de barreira de grupo, quando **todo** store naquele dword escreve a mesma constante em endereço constante e todo atômico sobre ele trata a constante como identidade. Sem isso, thread 0 retorna cedo e os `ds_min_u32`/`ds_max_u32` do tile combinam com LDS não inicializado (manchas escuras piscando). `emit/wave64_tests.rs` +77 com o padrão do tile |
| `e1dc7703` | wike1701 | feat(gpu): port opt-in GPU compute texture detiling (`VICTORY_GPU_DETILE`) | `formats/texture.rs` +164 (`Swizzle::name`, `DetileParams::address`, `Layout::detile_params`/`pattern_tables`), `gpu/gpudetile.rs` +714 (detiler em compute HLSL→DXIL, root signature, PSO, planejador de jobs, escritores de layout/máscara/tabela e teste WARP byte a byte), `gpu/texture.rs` +174 (staging `gpu`), `gpu/host.rs` +48 (lista `detiles` no prelude antes das cópias), `lib.rs` +2. Restrito a textura inteira começando no mip 0 com buffer dedicado em offset 0; comportamento antigo quando o switch está fora |
| `c0ab82aa` | Fleurrenza | DirectStorage: workers below normal priority | `hle/modules/dstorage.rs` +25/-3: workers que desempacotam o stream passam a rodar abaixo do normal (`VICTORY_DSTORAGE_PRIORITY=0` restaura; `VICTORY_DSTORAGE_WORKERS=n` fixa a contagem) |
| `ac24e23b` | Vladimir-Human | perf(rt): add opt-in `VICTORY_BVH_NO_HIT` and dead-RT compute pass skipping | `gpu/pipeline/make.rs` +321: classes de compute que só alimentam ou percorrem estruturas de ray tracing; `shader_cache.rs` +56 chaveia os switches **só quando ativos**; `shader/emit/intersect.rs` +41 faz o teste de nó responder miss sem ler o nó; `intersect_tests.rs` +120. Três switches independentes, todos fora por padrão, e `VICTORY_BVH_BUILDER=1` força os três a desligar |
| `69e7e009` | xss3009 | Fix transmission tile allocation on wave32 hosts | `emit/water_tile_append.rs` +63 (novo): baixa a alocação wave-wide das duas listas de tile de transmissão para host de 32 lanes, em vez de recusar o programa; `emit/mod.rs` +6 registra o módulo |
| `a6f6279b` | Vladimir-Human | fix(descriptor,ambient): honor `VICTORY_DYNAMIC_SLOTS` on `PORTABLE_VIEWS` hosts and add opt-in chroma scrub | `gpu/descriptor/host.rs` +592/-19: `dynamic_slots()` comparava `null < PORTABLE_VIEWS` com o slot nulo em 999.999 contra 1.000.000, então em NVIDIA o `VICTORY_DYNAMIC_SLOTS` era **ignorado em silêncio** e as slots dinâmicas ficavam no teto de 262.144; agora honra o explícito até `PORTABLE_DYNAMIC_CAP` (524.288) sem encolher o `count` quando não definido. `ambient.rs` +439 acrescenta `VICTORY_AMBIENT_CHROMA_SCRUB=1` (fora por padrão) para picos verde-ácido/ciano-elétrico em folhagem fina, intactos os caminhos magenta/violeta. `indirect.rs` +6 expõe `WarpHarness` para os testes |
| `075785eb` | Vladimir-Human | shader: static Wave32 LDS alias and lane-private hazard analysis | `emit/lds.rs` +3.482: substitui a checagem grosseira de leitura+escrita em `lds_meetings` por análise estática de alias e de perigo lane-private — refino A (ranges de byte constantes disjuntos sobre base uniforme do grupo, sem RAW/WAR/WAW, acíclico) e refino B (slots lane-private nas dimensões ativas do grupo, footprints meio-abertos para grupos parciais 33..63, com oráculo exaustivo de 48.212 casos e zero falsos aceites). Região só-escrita `!all` passa a exigir prova positiva; o atalho antigo fica atrás de `VICTORY_LDS_WRITE_ONLY_OK=1`. `lower.rs` +79/-108 passa a chamar `lds::analyze`; `wave64_tests.rs` +447 |
| `e399f154` | compaccount1-coder | Shaders: a select on a lane mask held as a number keeps the console's lanes | `emit/lower.rs` +14: `selects_on_mask_data` passa a contar para `console_lanes`. `v_cndmask_b32` sobre máscara guardada como **número** (padrão de bits, valor lido da memória) baixa para o bit da lane do host; sem a marca, um programa cuja única instrução cross-lane é esse select era baixado sem tamanho de onda e sem relabelling — em release cada lane pegava o bit da lane do host em que caiu, em debug o `debug_assert` de `cross_lane` disparava. `wave64_tests.rs` +22 com o caso mínimo (select lane-par em programa wave32 de 64 threads) |
| `ba360f5f` | KyleS98 | Series S without DLSS: the title keeps its own upscaler, not FSR 4 (28,5 → 30,9 fps) | `apps/victory/src/run.rs` +71/-1 e `crates/victory-game/src/lib.rs` +69/-2: duas funções do build conhecido passam a devolver 6 (o valor da Series X), então o perfil Series S renderiza com o upscaler do título; entra o conceito de `CodePatch` por build. `victory-loader/prepare.rs` +81: o BLAKE3 dos patches faz parte do nome da cópia preparada, para imagem remendada e imagem pura nunca se substituírem. Não se aplica quando o caminho FidelityFX do título é pedido (DLSS, geração de quadros, `VICTORY_FSR_PATH`) |
| `ed4cb994` | VI Personal | Launcher: `-Fps` dá ao título a própria taxa de quadros; sem teto o título recebe 59 | `scripts/play-foxhound.ps1` +12/-2: o launcher passa a definir `VICTORY_FRAME_INTERVAL_MS` (que também remenda a taxa do próprio título); `-FixedStep` mantém só o relógio do emulador |
| `066f779c` | VI Personal | Launcher: 12 pipeline makers at normal priority, no slow limit | `scripts/play-foxhound.ps1` +4: `VICTORY_PIPELINE_WORKERS=12`, `VICTORY_PIPELINE_LOWEST=0`, `VICTORY_PIPELINE_SLOW=0`; a espera por pipeline continua em 500 ms |
| `05a269d8` | victory-4060 | gpu: size the render targets' room by the peak they have held, not a flat 3 GiB | `gpu/resource.rs` +49: `PEAK` atômico por execução; o espaço reservado para os alvos vira 1,5× o pico já ocupado (mínimo 256 MiB, máximo o valor do switch) em vez dos 3.072 MiB fixos, que só eram devolvidos quando os alvos ocupavam metade. Efeito descrito: numa 4060 Laptop de 8 GB, 2.675 MB ficavam presos e a fila de texturas recusava texturas novas (rostos pretos, tela do celular vazia) |
| `1124487c` | mattiolocoding | fix(gpu): validate typed buffer formats against host support | `gpu/descriptor/host.rs` +121/-8: `buffer_view_ok` ganha o parâmetro `uav` e passa a validar o formato tipado contra o suporte do host antes de criar o descritor, em vez de aceitar qualquer formato e deixar o driver falhar |
| `a162ae7f` | harrisotiropoulos-jpg | Pipelines: the whole library is not saved again mid-burst or under memory pressure (stutters on a 32 GB PC) | `pipeline/library.rs` +42: novo `save_while_busy()`; `make.rs` +1/-1 troca `save(false)` por ele. Salvar serializa a biblioteca inteira (mais de 1 GB quente) e segura todo store; em rajada o intervalo mínimo passa a 180 s (`VICTORY_LIBRARY_SAVE_BUSY_S`) e sob pressão de memória o salvamento periódico espera 600 s. Os salvamentos de fila vazia e de saída não mudam |
| `0721b910` | vvzxcvv | Rays miss every node while the structure builder is off (purple plants on NVIDIA) | `shader/emit/intersect.rs` +27: com `VICTORY_BVH_MISS=1` todo teste de nó vira miss sem ler o nó (caixa devolve `bvh::NONE`, triângulo `bvh::MISS`), então a caminhada termina no primeiro nó; `apps/victory/src/play.rs` +3 e `shader_cache.rs` +1 ligam o switch no perfil de jogo. Fica inativo com `VICTORY_BVH_BUILDER=1`, quando as estruturas são reais |
| `dbe0895a` | vvzxcvv | NaN scrub: NaN and infinity in written colours become 0 | `shader/emit/lower.rs` +41: `Lower::scrub_nan` (com `VICTORY_NAN_SCRUB=1`) zera componente de cor exportada e de store de imagem float onde `|x| >= inf`, por comparação não ordenada (pega NaN também). Entra na chave do cache de shader |
| `e29d2c03` | Caasares | fix(scripts): resolve `-Exe` and DLSS folders before the launcher changes folder | `scripts/play-foxhound.ps1` +6: `Resolve-Path` em `-Exe`, `VICTORY_STREAMLINE_DIR` e `VICTORY_FIDELITYFX_DIR` **antes** do `cd` para a pasta do repositório; antes o título era procurado no lugar errado |
| `0bf686cb` | Caasares | fix(cli): keep network-share paths when `victory play` starts a title | `apps/victory/src/play.rs` +33/-3: `fs::canonicalize` devolve `\\?\UNC\server\share\...` e o código removia só `\\?\`, deixando `UNC\server\...` (que não é caminho e ainda ia para o restart do menu); agora `\\?\UNC\` vira `\\` e `\\?\` continua sendo removido |
| `4b8219d9` | Caasares | fix(scripts): launchers exit with the title's failure, not 0 | `play-foxhound.ps1` +5/-3 e `play.ps1` +3/-2: os `return` de falha (sem `cargo`, sem build, sem executável) passam a `exit 1` e o código de saída de `victory run` é repassado |
| `c291db80` | wike1701 | fix(gpu): skip copy-queue `texture::ahead` for GPU-detiled stagings | `gpu/texture/ahead.rs` +1/-1 e `texture.rs` +5: `claim()` filtra `!s.is_gpu()`. Com `VICTORY_GPU_DETILE=1` e `VICTORY_UPLOAD_AHEAD=1` a cópia ia para a fila de cópia, que não roda o passe de detile e lê `staging.buffer` com footprint linear — dava `E_INVALIDARG` (0x80070057) |
| `2b028ead` | ballsimulation | Shaders: four more 64-thread LDS programs taken wave-wide on 32-lane hosts | `emit/mod.rs` +13: quatro programas de compute de 1.0.2.1 (digests `5e3dd99e…`, `ae0deeaf…`, `a10179ab…`, `57f1b81d…`) entram na lista de `scene_kernel` que roda como onda de 64 num host de 32; `VICTORY_WAVE_WHY` passa a imprimir o digest do programa recusado. O de 1024 threads com LDS acima de 32 KiB continua recusado |
| `88e34a4c` | 1mosce | Reflections stand-in: the light around where the miss program has no depth of the picture (`VICTORY_AMBIENT_SPECULAR_BOUND=1`) | `gpu/ambient.rs` +149: passe adicional `#ifdef BOUND` que, quando o programa de miss não tem profundidade da imagem, preenche cada texel com a luz dos pontos ao redor da superfície sem usar profundidade; `draw.rs` +24 agenda a imagem dos raios sozinha |
| `a115e71c` | 1mosce | Helpers' last work is waited for awake (`VICTORY_WAIT_SPIN=<µs>`) | `gpu/shadow.rs` +38 (`spin_for`) e `land.rs` +9: a thread que distribui trabalho espera acordada, por tempo limitado, o último lote, antes de dormir. O que se espera não muda |
| `7ab985d0` | 1mosce | A submission's refreshes go to the kept threads several at a time (`VICTORY_WIDE_CHUNK=<n>`) | `gpu/shadow.rs` +110 (`wide_each_from`, `wide_chunk`) e `host.rs` +3/-2: texturas continuam uma por thread, cópias vão em blocos de `n` (1 por padrão, como antes) e nenhuma thread é acordada além das fatias |
| `edbf86e8` | 1mosce | A submission's one asking begun beside its first steps (`VICTORY_ASK_FIRST=1`) | `gpu/watch.rs` +142 e `host.rs` +46: a "asking" do write-watch começa numa thread própria assim que a submissão pega o lock, ao lado do planejamento e da olhada em voos; se voos foram esperados no meio, refaz inteira |
| `e080431a` | dcastell1 | feat(hle): per-monitor v2 DPI awareness and aspect-preserving ultrawide fullscreen | `hle/modules/lifecycle.rs` +199/-12: janela de fundo preta própria atrás de uma janela fullscreen ajustada por aspecto (`victory.backdrop`) e DPI v2 por monitor |
| `0eb34b84` | VI Personal | Merge main: `-Preload` e o bloco de geração de quadros FSR 3 ao lado de `-FixedStep` e do intervalo de quadro | **Merge da `main` no ramo; diff contra o primeiro pai = o que a `main` já tinha.** Arquivos que aparecem: `apps/victory/src/play.rs` +505 (FSR 3, Agility SDK, preload), `gpu/streamline/fsr.rs` +950 (FSR 3 do FidelityFX), `scripts/get-amd-fsr.ps1` +57, `scripts/get-d3d12.ps1` +60, `apps/victory/build.rs` +10 (exports `D3D12SDKVersion`/`D3D12SDKPath`), `main.rs` +92. Não é trabalho novo desta janela |
| `014b9335` | Codex | hle: opt in to GameInput XInput slots with per-slot system fallback | `hle/modules/gameinput.rs` +44: slots XInput via GameInput com fallback por slot para o XInput do sistema; desligado preserva o XInput do sistema e a política HID anterior |
| `e179a701` | Codex | gpu: opt in to epoch-validated texture reads | `gpu/texture.rs` +24: `VICTORY_REFRESH_LOCKLESS` liga a leitura de textura validada por época; desligado, caminho e bytes originais. É a metade "sem lock" do que o `#59` anuncia — a outra metade ("compartilhar uma leitura entre oito") não aparece como commit próprio nesta faixa |
| `8ea59cd6` | superboynick | hle: say, once per slot, whether a pad answered it | `hle/modules/gameinput.rs` +30/-1: com `VICTORY_GAMEPAD_LOG=1` cada slot declara **uma vez** por qual caminho respondeu e o que o XInput do sistema disse quando não respondeu |
| `6da6be57` | Codex | gpu: opt in to fenced copy-queue uploads of submission snapshots | `gpu/host/upq.rs` +204 (novo): buffer privado na fila de cópia, promoção de COMMON e decadência depois de `ExecuteCommandLists`, com o slot segurado até a conclusão dos comandos DIRECT que consomem a cópia; `host.rs` +20, `host/send.rs` +11/-3 |
| `ef87380a` | superboynick | gpu: gate the upload copy queue by size | `gpu/host/upq.rs` +58/-7 e `host.rs` +5/-3: `VICTORY_UPLOAD_QUEUE` deixa de ser booleano e vira portão em MiB — ausente/0 mantém o caminho original, 1 manda todo upload não vazio pela fila de cópia, N acima de 1 só os de pelo menos N MiB |
| `cf9a417b` | superboynick | gpu: a MiB count too large to hold is not a threshold | `gpu/host/upq.rs` +20/-1: o limiar passa de `mebibytes << 20` para `checked_mul`; contagem grande demais deixa de dar a volta e virar portão pequeno ou zero, e cai no mesmo caminho de configuração ilegível |
| `76e3c697` | VI Personal | Merge main: a pilha de memória e `-FrameGenReady` ao lado dos pipeline makers e de `-FixedStep` | **Merge da `main` no ramo; diff contra o primeiro pai = o que a `main` já tinha.** Arquivos: `gpu/host.rs` +709/-80, `gpu/descriptor.rs` +437/-7 (views sem dono), `gpu/save_share.rs` +672 (exportar/importar save pelo menu F1), `gpu/ahead.rs` +165, `gpu/pool.rs` +186, `gpu/pipeline/samplers.rs` +287 (samplers estáticos em ordem estável), `hle/modules/dsp/convolve.rs` +232 (convolução de DSP na CPU), `hle/modules/gameinput/playstation.rs` +1.197 substituindo `dualsense.rs` (-670), `hle/modules/xmem.rs` +80. Não é trabalho novo desta janela |
| `924911fb` | Yorivel | Host lists: copy sources come from a pool; under pressure with memory to spare, keep 128 MiB of spares | `gpu/host.rs` +110/-9: buffers de origem de cópia saem de um pool por classe de tamanho (potência de 2, mínimo 64 KiB) e voltam no reset, com teto `VICTORY_UPLOAD_POOL_MB` (1.024 por tipo); `trim_spare_uploads` mantém `SPARE_MOST` (128 MiB) sob pressão quando há 8 GiB ou mais livres; `residency.rs` +5 expõe `physical_free` |
| `f6782755` | trustno1-coder | Launcher skips the D3D12 check under Wine; KEY BINDINGS under MOUSE AND KEYBOARD | `apps/victory/src/main.rs` +24 detecta Wine por `wine_get_version` no ntdll e pula a checagem/baixar do Agility SDK (sob Wine o D3D12 é vkd3d-proton e nunca carrega o `D3D12Core.dll` da Microsoft); `play.rs` +8; `menu.rs` +12/-11 move a linha `KEY BINDINGS` para baixo de `MOUSE AND KEYBOARD` |
| `32ecacfa` | trustno1-coder | Tests: the wave64 tests of `#35` and `#52` kept whole beside `#43`'s | `shader/emit/wave64_tests.rs` +70/-61: o merge de união intercalou os testes dos três PRs e deixou uma função sem fechar; o arquivo volta a ser a versão do `#43` com os testes de `#35` e `#52` inteiros depois dela |

## Pull requests abertos (50)

Cursor anterior nulo, então os 50 lidos são a janela. Snapshot às 19:2xZ: #1274 a #1409. Uma leitura
anterior, dez minutos antes, tinha #1265 a #1400 — **10 PRs novos em dez minutos**. Nenhum corpo foi
lido: a coluna "Proposta" vem do título. Não há catálogo para este repositório no acervo (não tem
análise ainda), então a última coluna responde só por área.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| `#1409` | Queue and device tables: SuspendX, ResumeX and WaitX; ID3D12Device9 and 10 | Completa as tabelas de fila e de dispositivo do HLE | Sem catálogo |
| `#1408` | BCPack: BC7 dictionary indices read as Microsoft's decoder reads them | Corrige a leitura dos índices de dicionário BC7 do BCPack | Sem catálogo |
| `#1407` | Sample mask: masked draws discard the pixels `OMSetSampleMaskX` leaves out | Aplica máscara de amostra a draws mascarados | Sem catálogo |
| `#1403` | perf(host): overlap send upload copies with sync/check D3D12 recording in `card_before` | Sobreõe cópias de upload com a gravação de sync/check | Sem catálogo |
| `#1400` | perf(shadow): batch `card.dirty` in `merge_under_writes` and binary-search skip in `land_card` | Lote de `card.dirty` e salto por busca binária | Sem catálogo |
| `#1399` | GPU: experimental graphics pipeline variant reconstruction (0x1B8) | Reconstrução de variante de pipeline gráfico | Sem catálogo |
| `#1397` | perf(host): parallelize HostList prelude recording and skip empty copying map | Paraleliza a gravação do prelude | Sem catálogo |
| `#1394` | fix(gpu): a copy the title records holds back thin lists' uploads | Cópia gravada pelo título segura uploads de listas thin | Sem catálogo |
| `#1393` | Script launcher: mouse and keyboard work from it too (#1369) | Mouse e teclado pelo launcher de script | Sem catálogo |
| `#1392` | gpu: write-back L0 HostList scratch buffers, zero-COM `Chunk.gpu`, lock-free `Card::before`, unified parallel `send::take` | Quatro mudanças de desempenho no host | Sem catálogo |
| `#1391` | GPU: reuse retired staging buffers for oversized uploads | Reusa staging aposentado em uploads grandes | Sem catálogo |
| `#1390` | GPU: reuse retired upload buffers for oversized submissions | Idem para submissões grandes | Sem catálogo |
| `#1382` | Add native Graphics settings alongside F1 | Menu de gráficos nativo ao lado do F1 | Sem catálogo |
| `#1375` | perf(gpu): overlap card-before sends and batch sync command list | Sobreõe envios e loteia lista de sync | Sem catálogo |
| `#1373` | perf(gpu): batch `nothing_new` refresh checks under one `JournalRead` lock and single-lock `watch::writes` | Loteia checagens de refresh | Sem catálogo |
| `#1363` | The inplace gate enforces the module's own rule about late wait | Portão inplace passa a impor a regra de espera tardia | Sem catálogo |
| `#1360` | The WARP tests cannot skip in silence any more | Teste WARP que pula sem dizer deixa de ser aceito | Sem catálogo |
| `#1359` | Guardian | Guardião (escopo não legível pelo título) | NAO VERIFICADO |
| `#1358` | perf(gpu): offload `bury()` COM drops off relay thread and batch dynamic slot releases | Tira drops COM da thread de relay | Sem catálogo |
| `#1357` | Frame generation asks the base-frame pace guard before it runs | Geração de quadros consulta o guarda de ritmo antes | Sem catálogo |
| `#1356` | Native 1:1 mouse aiming, vehicle controls, pre-launch DLSS warm-up, and portable launcher | Quatro frentes de entrada e launcher | Sem catálogo |
| `#1355` | Tests: validate copied D3D12 descriptors through GPU-visible semantics | Valida descritores copiados pela semântica visível à GPU | Sem catálogo |
| `#1351` | Bundle submission, streaming, texture and shader improvements | Pacote de melhorias | Sem catálogo |
| `#1348` | The sweep lets go placed host copies whose title memory is unmapped | Varredura solta cópias colocadas cuja memória do título foi desmapeada | Sem catálogo |
| `#1345` | perf(gpu): dedup `GpuWrite`/declared in `Batch`, reuse `front_spans`, zero-copy `Arc` `card_dirty`, 1-pass `send::take` | Quatro mudanças de desempenho | Sem catálogo |
| `#1343` | GPU: honor the staging spare limit when returned ranges become reusable | Honra o limite de spares quando ranges voltam | Sem catálogo |
| `#1339` | Scripts: screenshots and the game's memory paired with the log (`watch-screen.ps1`, `shots_vs_log.py`) | Diagnóstico: screenshot e memória pareados com o log | Sem catálogo |
| `#1337` | Texture pool: fullest block and best range first, eviction from the emptiest blocks (8 GB card stays under budget) | Política de pool de textura para caber em 8 GB | Sem catálogo |
| `#1335` | streamline: give the cleared frame generation tags the back buffer ex… | Não legível pelo título (truncado no GitHub) | NAO VERIFICADO |
| `#1331` | Add standalone Windows hardware diagnostics for QA | Diagnóstico de hardware para QA | Sem catálogo |
| `#1328` | Launcher: the F1 menu's frame-generation multiplier wins over the auto-4x (#966) | Multiplicador do menu vence o automático | Sem catálogo |
| `#1326` | FSR 3 frame generation: the DLAA mode still reached the `+0x2219650` break | Corrige o break do patch de FSR 3 em DLAA | Sem catálogo |
| `#1322` | perf(gpu): split `CardShared` out of `CARD_WORK` lock and avoid full-plan flags scan in relay gather | Divide lock e evita varredura de flags | Sem catálogo |
| `#1321` | GPU: validate indirect argument and count ranges against their resources | Valida ranges de argumento/contagem indiretos | Sem catálogo |
| `#1319` | Add game test log report for VIctory-NVIDIA | Relatório de log de teste do jogo | Sem catálogo |
| `#1316` | A changed executable is told what starts without the 1440p patch (#1300) | Avisa o que arranca sem o patch de 1440p quando o executável mudou | Sem catálogo |
| `#1313` | GPU/shader perf, DLSS FG 2x fallback, water volumetrics Wave32 fix & faster preload | Pacote: desempenho, fallback 2x, correção de volumetria de água | Sem catálogo |
| `#1299` | Series S without DLSS: the title's own TAA in place of its FSR4 network (`VICTORY_RENDERER_TEMPORAL=1`, +16% numa 3060) | Troca a rede FSR4 pelo TAA do título na Series S | Sem catálogo |
| `#1298` | perf(gpu): cut submitting-thread CPU overhead in refreshes, `card_steps`, `safecopy`, and list recycling | Reduz CPU da thread de submissão | Sem catálogo |
| `#1296` | Wave64 folded translator, upload guard and placed-view twins: hair, beards and trails right on NVIDIA | Tradutor wave64 dobrado e guarda de upload | Sem catálogo |
| `#1295` | Sampler switches: `VICTORY_ANISO` (forced anisotropy) and `VICTORY_MIP_BIAS`, off by default | Dois switches de sampler, fora por padrão | Sem catálogo |
| `#1294` | The test-gate and the workflow shape checks: a green run must have run something | Portão de teste: verde tem de ter rodado algo | Sem catálogo |
| `#1290` | Fix main's CI: four clippy lints and the policy allowlist that failed the last four pushes | Conserta a CI da `main` | Sem catálogo |
| `#1286` | port: default constants port off on Ampere adapters (#1223) | Desliga o port de constantes por padrão em Ampere | Sem catálogo |
| `#1285` | Shaders: honor byte width and signedness on CBV buffer loads | Honra largura de byte e sinal em cargas de CBV | Sem catálogo |
| `#1284` | Decode signed 16-bit normal target captures as signed channels | Decodifica captura de normal 16 bits como canal com sinal | Sem catálogo |
| `#1282` | GPU: extrapolate packed RGB10A2 UNORM rectangle attributes | Extrapola atributos RGB10A2 UNORM empacotados | Sem catálogo |
| `#1279` | perf(gpu): optimize `card_steps` planner, batched fill-to lookup, sorted writer search | Desempenho do planejador e buscas | Sem catálogo |
| `#1278` | perf(gpu): single-pass `ask_once`, binary-searched ahead ranges, `Batch` add caches | Desempenho do write-watch e do batch | Sem catálogo |
| `#1274` | Series X: room for a full-size 3840x2160 scene (DLAA, `VICTORY_NATIVE_4K`) | Espaço para cena 4K nativa | Sem catálogo |

## Pull requests mesclados (30)

Todos com `merged_at` entre 18:12:15Z e 18:22:18Z. Os 30 merges e os 30 títulos de PR aparecem na
**mesma ordem** dentro do `compare` e na resposta de `pulls`, então a correspondência merge → commit de
conteúdo é a da ordem, e o commit citado é o que foi lido no diff.

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| `#33` | Textures: a prelude upload checks again at submission that the host has not written the texture | `84651a25` | `3f6ba011` — `host_written` reconferido no `finish_refresh`; upload de prelude deixa de sobrescrever textura escrita pelo host (manchas de cor no rosto) |
| `#34` | perf(gpu): snapshot placed-copy constants on GPU and enable `VICTORY_PLACED_ORDER` | `ef2cab5e` | `af242dab` — constantes de cópia colocada fotografadas na GPU; `VICTORY_PLACED_ORDER` ligado por padrão |
| `#36` | feat(gpu): port opt-in GPU compute texture detiling | `19dadf59` | `e1dc7703` — detile em compute na placa, opt-in, com teste WARP byte a byte |
| `#39` | DirectStorage: workers below normal priority | `72a1149e` | `c0ab82aa` — prioridade dos workers de desempacotamento abaixo do normal |
| `#42` | perf(shader): add DXIL folds, pure LICM, local/shared reuse, and barrier dedup | `c4e4fcfe` | `00d5623c` — o maior diff da janela (+4.665 em `bitcode/opt.rs`); `DEFAULT` 447 |
| `#43` | shader: static Wave32 LDS alias and lane-private hazard analysis | `1789b869` | `075785eb` — análise estática de LDS (+3.482) substituindo a checagem grosseira |
| `#45` | Fix transmission tile allocation on wave32 hosts | `d583714d` | `69e7e009` — alocação wave-wide de tile de transmissão baixada para 32 lanes |
| `#47` | perf(rt): add opt-in `VICTORY_BVH_NO_HIT` and dead-RT compute pass skipping | `6a407628` | `ac24e23b` — três switches de ray tracing independentes, fora por padrão |
| `#48` | fix(descriptor,ambient): honor `VICTORY_DYNAMIC_SLOTS` and add opt-in chroma scrub | `768160e6` | `a6f6279b` — o off-by-one `null < PORTABLE_VIEWS` que ignorava o switch em NVIDIA |
| `#57` | Host lists: pool the copy sources; trim spares to 128 MiB instead of nothing | `2ecb3d95` | `924911fb` — pool por classe de tamanho e `SPARE_MOST` sob pressão |
| `#59` | gpu: two opt-in refresh changes — read without the mapping lock, share one read across eight | `b76a9ca3` | `e179a701` — a metade "sem lock" (validada por época); a metade "compartilhar uma leitura entre oito" não aparece como commit próprio na faixa |
| `#60` | gpu: opt in to fenced copy-queue uploads of submission snapshots, gated by size | `564b98ca` | `6da6be57` — `upq.rs` novo, cópia na fila de cópia com slot segurado até a conclusão |
| `#61` | hle: opt in to GameInput XInput slots with a per-slot system fallback | `39c4b995` | `014b9335` — slots XInput via GameInput com fallback por slot |
| `#65` | Launcher: real game speed above 30 FPS, 12 pipeline makers at normal priority | `69f05d6c` | `ed4cb994` + `066f779c` — `VICTORY_FRAME_INTERVAL_MS` e os 12 makers |
| `#68` | Series S without DLSS: the title keeps its own upscaler, not FSR 4 | `5583a872` | `ba360f5f` — duas funções do build devolvem 6 e a Series S usa o upscaler do título; patches entram no nome da imagem preparada |
| `#74` | GPU: check typed buffer format support before creating descriptors | `2ce7a3c1` | `1124487c` — `buffer_view_ok` com `uav` e validação de formato tipado |
| `#76` | gpu: size the render targets' room by the peak they have held, not a flat 3 GiB | `14e5deda` | `05a269d8` — pico medido, 1,5× com piso de 256 MiB |
| `#81` | Pipelines: the library is not saved again mid-burst or under memory pressure | `60a6ca85` | `a162ae7f` — `save_while_busy`, 180 s em rajada e 600 s sob pressão |
| `#82` | NVIDIA: rays miss unbuilt structures (purple plants) + NaN scrub | `e73ae09d` | `0721b910` + `dbe0895a` — `VICTORY_BVH_MISS` e `VICTORY_NAN_SCRUB`, ambos ligados no perfil de jogo |
| `#92` | fix(scripts): resolve `-Exe` and DLSS folders before the launcher changes folder | `2414bfad` | `e29d2c03` — `Resolve-Path` antes do `cd` |
| `#101` | Shaders: four more 64-thread LDS programs taken wave-wide on 32-lane hosts | `991887d1` | `2b028ead` — quatro digests entram em `scene_kernel` |
| `#124` | Reflections stand-in: the light around where the miss program has no depth | `42b24366` | `88e34a4c` — passe `BOUND` sem profundidade |
| `#126` | A submission's refreshes go to the kept threads several at a time | `0aed7ece` | `7ab985d0` — `wide_each_from` e `VICTORY_WIDE_CHUNK` |
| `#127` | A submission's one asking begun beside its first steps | `17eaae2d` | `edbf86e8` — `VICTORY_ASK_FIRST` no `watch.rs` |
| `#132` | feat(hle): per-monitor v2 DPI awareness and aspect-preserving ultrawide fullscreen | `74c18249` | `e080431a` — janela de fundo e DPI v2 |
| `#35` | Shaders: hoist constant LDS accumulator initialisations behind an entry barrier | `1aeb43e7` | `759f05c1` — stores constantes de acumulador sobem para a entrada, atrás de barreira |
| `#52` | Shaders: a select on a lane mask held as a number keeps the console's lanes | `ed998ea2` | `e399f154` — `selects_on_mask_data` conta para `console_lanes` |
| `#93` | fix(cli): keep network-share paths when `victory play` starts a title | `f3a1bf5b` | `0bf686cb` — `\\?\UNC\` vira `\\` |
| `#94` | fix(scripts): launchers exit with the title's failure, not 0 | `0f9dfc5b` | `4b8219d9` — `exit 1` e repasse do código de `victory run` |
| `#125` | Helpers' last work is waited for awake | `6c8f918c` | `a115e71c` — `VICTORY_WAIT_SPIN` em `shadow::spin_for` |

## Cruzamento com o catálogo

- **Hash de catálogo na janela: nenhum.** Os 262 tokens de `catalogo/defeitos.csv` foram conferidos
  contra os 70 hashes da faixa: zero interseções, inclusive por prefixo de 7 caracteres. Este
  repositório **não tem catálogo próprio** no acervo (não há análise dele ainda), então o critério 1
  não podia acionar de outra forma.
- **Classe do VIctory D-06** ("descritor lido de `v_readfirstlane_b32` tratado como 'o de cada lane'").
  `e399f154` é a mesma classe em outra instrução: select sobre máscara guardada como **número** era
  baixado sem `console_lanes`, e cada lane do host pegava o bit da máscara errado.
- **Classe do VIctory D-08** ("cópia de textura escrita pela CPU não era enviada ao host antes de
  `CopyResource`"). `3f6ba011` é o caso espelhado: a textura escrita pelo **host** era sobrescrita pelo
  prelude na submissão. Mesma família — ordem entre escrita da CPU/host e upload de prelude.
- **Classe do VIctory D-02** (heap de views limitado pelo host). `a6f6279b` corrige o mesmo tipo de teto:
  `VICTORY_DYNAMIC_SLOTS` era ignorado em NVIDIA por um off-by-one contra `PORTABLE_VIEWS`, e as slots
  ficavam presas em 262.144.
- **Abre defeito novo?** Dois pontos de atenção, nenhum provado: (a) `00d5623c` muda a **chave do cache
  de shader** por meio de `VICTORY_TRANSLATOR`/`VICTORY_SHADER_SOURCE_HASH` — o corpo diz que a
  invalidação é automática, e o diff de `shader_opt()` não foi lido linha a linha; (b) `#59` anuncia duas
  mudanças e só a metade sem lock aparece como commit próprio, então a outra metade não foi localizada
  por hash nesta faixa. Candidatos a linha nova no catálogo, para o próximo ciclo: `#1408` (índices de
  dicionário BC7 lidos como o decodificador da Microsoft lê) e `#1407` (máscara de amostra em draw
  mascarado).

## Confiança

- **Verificado:** os 70 commits por `repos/trustno1-coder/VIctory-NVIDIA/compare/93008750...32ecacfa`
  (`ahead` 70, 0 atrás) e, para os 40 de conteúdo, `commits/<sha>` com arquivos e patch (até 3 linhas de
  hunk por arquivo); os 30 merges com hash e ordem; o `merged_at` de cada um dos 30 PRs pela API REST de
  pulls fechados; a lista de 50 abertos em dois snapshots (1265–1400 e 1274–1409).
- **NAO VERIFICADO:** os corpos dos 50 PRs abertos (só título; `#1359` e `#1335` não são legíveis pelo
  título); se algum dos 30 merges foi testado em GPU antes do merge (as mensagens citam medições, o diff
  não prova execução); a outra metade do `#59`; se `#1409`/`#1408` foram criados já depois do snapshot
  final.
- **Limite de documentação:** 40 commits descritos individualmente (teto) e os 30 merges descritos pela
  tabela de PRs — 70 no total na janela, nenhum commit de conteúdo ficou fora.
- **Relógio:** `merged_at` 19:01:28Z do último lote e `pushed_at` 19:01:26Z do repositório batem com o
  UTC real desta coleta (19:0xZ). Não há desvio a corrigir.
- **Cursor:** `last_commit` avança para `32ecacfa` e o lado de PRs abertos sai de `null` para #1409; o
  piso abaixo de #1265 não foi lido e vai para o `backlog`.
