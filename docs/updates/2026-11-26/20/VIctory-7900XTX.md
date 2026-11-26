# avatarmanz/VIctory-7900XTX — atualização de 2026-11-26 20:00

## Resumo

**40 de 147 commits** na faixa `b0cfe2ba..2de99801` (teto de 40 aplicado; `commit_cursor` gravado em
`306722bc`, 107 commits ficam para o próximo ciclo). A faixa é dominada por três frentes que se cruzam:
uniformidade de wave no tradutor (`lane_uniform`, provas de LDS-flag, BVH), o caminho de memória/cópias
(`safecopy` com guard pages, `watch` com ranges pedidos adiantados por busca binária, pool de texturas em
múltiplos de 64 KiB) e o diagnóstico de quadro (medidor em thread própria, mapa de memória a cada 5 min,
categorias de memória). Entram também dois consertos de robustez do livro de pipelines (`1438f09d`,
`33439c8d`) e o `VICTORY_SHOW_NEW_LATER` ligado por padrão (`4cd2d133`, `91018a5d`, `cf9ba084`), que ataca
exatamente o mecanismo do defeito D-09 do catálogo. 32 PRs mesclados, todos com merge commit dentro da
faixa — a maioria além do teto, portanto listada por conjunto de arquivos, sem leitura de hunk.

## Commits novos (40)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| ad0f9dda | FrancescoGelosoEterea | Shaders: a lane-branch region that only raises an LDS flag runs as two host waves | Em `victory-shader/src/emit/lower.rs` (+339/−2) entra `lds_flag_region`: uma região de branch por lane que lê e escreve LDS passa a poder correr como duas host waves quando toda escrita grava a mesma constante em endereço constante (`ds_read_b32`/`ds_write_b32`) e o que sobra são flags. Antes, `if (0..n).any(\|r\| reads[r] && writes[r]) { return None }` recusava a divisão. Interruptor `VICTORY_LDS_FLAGS` (`0` desliga). |
| 25025d2f | gomeaw | Check the final LDS flag from every lane on a 32-lane adapter | Só `emit/wave64_tests.rs` (+64/−6): novo programa nativo de 64 threads (duas host waves) que zera a flag, levanta dentro do branch e relê; todo thread grava a flag final em offset próprio de UAV, e o teste compara o resultado nas duas host waves. Também corrige o programa de fixture anterior (constantes de codificação e `buffer_store_dword` com `offen`). |
| b295a683 | gomeaw | Use legal D3D12 states and bindings in the replay fixture | `tests/replay.rs` (+19/−2): `DescriptorTable(SRV(...))` recebe `flags=DATA_VOLATILE`, e entra uma `ResourceBarrier` que devolve `self.shared.0` a `NON_PIXEL_SHADER_RESOURCE` antes do uso seguinte; o relatório do debug layer passa a ignorar só a nota `errors 527` do experimento de submissão única (`one_submit`), em vez de aceitar qualquer erro. |
| 631aadc9 | gomeaw | Declare the clamp argument inputs volatile before they are rewritten | `tests/replay.rs` (+33/−34): o root signature do fixture de clamp passa a declarar `SRV(t0, flags=DATA_VOLATILE)` e `SRV(t1, flags=DATA_VOLATILE)`; a verificação de erros vira uma leitura real de `ID3D12InfoQueue` (percorre `GetNumStoredMessages`, aloca `D3D12_MESSAGE` e filtra severidade ≤ WARNING) em vez do contador anterior. |
| 0a4253f7 | gomeaw | Share hardware adapter selection with offline pipeline checks | Seis arquivos, +175/−33: a escolha de adaptador sai de `apps/victory/src/translate.rs` para `victory-gpu/src/host.rs` e `victory-shader/src/emit/host.rs`; `Host::new` loga nome, vendor/device PCI e VRAM; `VICTORY_D3D12_ADAPTER` aceita índice DXGI ou trecho do nome; o caminho `--pipelines` usa o mesmo adaptador e imprime `pipeline adapter: <nome>`. README documenta a variável. |
| 3bb5980a | Lux-Designs | replay: defer clamp scratch copy until split transition completes | `replay/edit.rs` e `replay/play.rs` (+98/−29): `keep_clamped` passa a devolver `Result<Vec<Cmd>>` e só copia o scratch do clamp quando a barreira `Transition { after: INDIRECT_ARGUMENT }` correspondente aparece com `flags & 1 == 0` (fim de `BEGIN_ONLY`/`END_ONLY`); a versão anterior considerava a barreira pronta por qualquer transição do mesmo recurso, inclusive a de começo. |
| fa116500 | zocram4cc | copies: a guard page fails the copy and stays a guard page | `safecopy.rs` e `watch.rs` (+58/−4): entra `guarded(at)` via `VirtualQuery` (`Protect & PAGE_GUARD`), usado para reconhecer a página de guarda na origem; teste novo confirma que a cópia falha e que a página continua de guarda. O rearme anterior (`rearm_guard` sobre `STATUS_GUARD_PAGE_VIOLATION`) sai do handler. |
| 8699d96a | UwUCammie | GPU: count a frame after the swap chain accepts the present | `host.rs` (+41/−3): `record_present(hwnd, status)` só chama `self.frame(hwnd)` quando `Present` devolve `S_OK`; status diferentes do anterior geram `tracing::warn!`. Antes o `let _ = Present(...)` descartava o HRESULT e o medidor contava quadro recusado. |
| b68e65e0 | gomeaw | Keep structured buffer slots raw on hosts without descriptor twins | `descriptor/host.rs` (+40/−16): `off_elements` ganha `has_raw_twins` (`victory_shader::emit::raw_views() != 0`); sem gêmeo raw, uma view estruturada sem tipo também exige cópia própria alinhada a 16 bytes, e a mesma cópia é escolhida na criação e no refresh (`probe_view_limit()` chamado quando o heap do título ainda não existe). |
| ce42cc74 | gomeaw | Anchor the structured view regression to a registered buffer | `descriptor/host.rs` (+16/−7): o teste de regressão deixa de usar `start + 456` como âncora e registra o intervalo (`crate::shadow::register(key, start, 4 * 456)`); passa a afirmar que descritor e refresh compartilham a mesma cópia e que `(address, bytes) == (start + 456, 3 * 456)`. Só teste. |
| 888bf1e2 | UwUCammie | Mask out other parts before scanning buffer descriptor slots | `descriptor/host.rs` e `draw.rs` (+40/−16): a varredura round-robin de slots passa de um predicado `which(slot)` para máscaras de 64 bits — `all(mask)` e `part::<PARTS>(part)` constroem a máscara com `step_by` e usam `bits & mask` antes de `trailing_zeros`; evita percorrer slots que aquela parte não escaneia. |
| e29bfe27 | gomeaw | Include the LDS flag switch in shader and pipeline cache keys | `shader_cache.rs` (+1): adiciona `"VICTORY_LDS_FLAGS"` à lista de variáveis que entram na chave de cache. Sem isso, alternar a flag reutilizava traduções feitas sob o outro valor — a mesma classe do defeito D-02 do `victory-fork`. |
| 787c7b43 | FictionHub | GPU: state the DRED stop command as the first uncompleted op | `host.rs` (+66/−4): `stopped_command(history, done, count)` devolve `history[done]` como a primeira operação **não** concluída (a documentação conta `pLastBreadcrumbValue` como número de operações completas) e a linha passa a dizer os dois índices em base zero; antes imprimia `stopped at command {done}` e era lido como a última concluída. |
| c2775339 | zocram4cc | copies: count a guard page that cannot be re-armed; destination guard tests | `safecopy.rs` e `watch.rs` (+122/−15): o rearme de guarda vira caminho de teste com `guard_second_page()` (`VirtualProtect` de `PAGE_READWRITE \| PAGE_GUARD` na segunda página); entram testes de cópia **para** página de guarda e um contador de guarda que não pôde ser rearmada. |
| 431cab5d | justrealyme | Shaders: first-lane reads of values alike in a console wave, under conditions alike in it | `emit/lower.rs` (+270/−43): `lane_uniform` ganha parametrização por `all_lane_blocks` e passam a contar como uniformes valores escritos sob condição uniforme (sobre valor já uniforme), carregados de LDS em endereço uniforme e o `thread id x` deslocado além de 6 bits; phi só conta com todos os predecessores alcançados juntos, e loop-carried roda a ponto fixo (valor que depende de si mesmo nunca é uniforme). |
| abbbd6b0 | ryotdev | Shader: mask the shift count when proving wave-uniform thread IDs | `emit/lower.rs` (+21/−3): a prova de `tid >> k` passa a mascarar `k & 31` (a instrução usa só os cinco bits baixos) e a exigir `((k - 128) & 31) >= 6` para o codificado inline; teste novo varre `shift` de 0 a 64 com `v_lshrrev_b32`. Antes a faixa `134..=192` era tratada como constante crua. |
| 7bf91de4 | gomeaw | Keep the BVH uniformity proof within explicit operands | `emit/lower.rs` (+72/−35): em `wave_all`, operandos do tipo `Operand::Vector { dwords }` são lidos dword a dword só na parte explícita (não no destino preservado de `reads`, que antes contava como entrada), e `_d16`/`gds` saem do conjunto considerado; `lane_regions` passa a ser computado uma vez e reusado por bloco. |
| 06fe22a7 | Mumogg | Shaders: count threads of wave64 builder programs on 32-lane hosts | `emit/lower.rs` (+188/−16): entra `partial_active` para grupo final parcial (`USER_DISPATCH_THREADS_X`): quando uma wave de console é representada por duas host waves com troca por LDS, os threads além da contagem **não saem** na entrada (as barreiras de grupo precisam deles) e ficam fora de memória, ballots de EXEC e reduções via `launched()`. |
| 1438f09d | Yoshi-233 | Retry incomplete pipeline book appends and flush on orderly exit | Cinco arquivos, +107/−8: `pipeline/known.rs` ganha `flush()` público com `WRITER: Mutex<()>` serializando timer e shutdown durante I/O; `run.rs` registra `set_before_thread_shutdown(victory_hle::known_pipelines::flush)` e chama `flush()` antes de encerrar. Corrige acréscimo perdido quando o processo termina de forma ordenada. |
| 7348b80a | gomeaw | Account for implicit lane inputs in the wave uniformity proof | `emit/lower.rs` (+34/−3): a prova passa a considerar entradas implícitas de VGPR (`reads` do tipo `Reg::V`) e o operando de máscara de `v_cndmask_b32` (SCC/sgpr com máscara conhecida); `v_movrel*`, `v_swaprel*` e `v_div_fmas` ficam de fora. Teste para o caso de máscara vinda de sgpr. |
| 70cc7d2e | Adika713 | GPU: add optional memory category log | `copystat.rs` (+71/−3): `category_text(fields)` formata contagens de bytes já coletadas; `VICTORY_MEMORY_CATEGORIES=1` emite uma linha periódica. O próprio comentário registra que as categorias se sobrepõem e não devem ser somadas. |
| 4fd34fdd | sawetco | Shaders: translate tiled cascade shadows exactly on wave32 hosts | `emit/wave64_tests.rs` e `ir.rs` (+332/−30): entra o gerador `far_early_out` (wave64 de 128 threads com `s_cbranch_execnz` sobre salto longo até o fim, como o compilador do console emite) e `whole_uav` para ler N dwords de UAV; o tradutor deixa de tratar todo bloco só de `Sopp` no fim como destino de saída e passa a reescrever o salto. |
| 5cfe32df | feanorsun | Drop the invalid play.ps1 skip default and fail on missing executables | `scripts/play.ps1` (+3/−3): sai `VICTORY_SKIP_PIPELINES = 'none'` do bloco de padrões e os dois `Test-Path` passam de `return` para `exit 1`. O valor `none` não é lido por nenhum switch do código — é a classe do D-19 (configuração documentada ≠ configuração lida). |
| ffc2cf6f | local | GPU: cap idle windows under system-RAM pressure | `host.rs` e `shadow.rs` (+238/−6): entra `RamBand { Calm, Watch, High }` com histerese (`ram_band(previous, …)` exige marca d'água mais alta para entrar que para sair) e o teto das janelas ociosas passa a consultar pressão de RAM do sistema; o caminho antigo (`ram_idle_ms(available_physical_ram())`) é removido. |
| fb28d49e | local | The memory map walked every 60 lines (VICTORY_MEMORY_MAP=<n>, was 6), the meter's kind of memory asked once per address; what the asking finds in ranges asked alone | `copystat.rs` e `watch.rs` (+86/−17): `memory_map()` passa de booleano a intervalo (`VICTORY_MEMORY_MAP=<n>`, padrão 60 linhas ≈ 5 min; `0` desliga). O comentário mede o custo: 250 000 regiões, 0,9 s, segurando o mapa de memória do processo — as *askings* do thread de submissão e os faults do título esperam (duas de 11 e 13 ms em `lane2-0`). |
| d792ebd4 | sharan104 | The switches the documents name are the ones the code reads, with their defaults; xtask checks every VICTORY_* name in the Markdown is read | Cinco arquivos, +137/−17: `xtask/src/switches.rs` novo varre o Markdown e compara com os `VICTORY_*` que o código lê; `docs/findings.md` e `README-fxceless.md` são corrigidos (por exemplo `VICTORY_PAGE_WRITES=guards` passa a "ligado por padrão" e `VICTORY_MERGE_RACE` sai, com a nota de que o contador não achou nada em 138 000 copy-backs). |
| effa1dc8 | sharan104 | VICTORY_WRITE_SITES is not written inside a source tree | `titlewrite.rs` +39/−4 e `Cargo.toml`/`Cargo.lock`: o arquivo nomeado por `VICTORY_WRITE_SITES` passa por `outside_repo()` e fica em cache (`SITES_FILE`), com `Option::is_none` significando "recusado" — depois da recusa o caminho não tenta mais abrir. Nova dependência `victory-core`. |
| c278f0d5 | local | Pooled textures at 64 KiB multiples unless VICTORY_POOL_SMALL=1; diagnostics: a pooled texture whose first level the card reads past its piece is said loudly; test: a placed texture holds its heap | `book.rs` e `pool.rs` (+147/−2): `book::address(texture)` lê `GetDesc` e devolve o endereço do primeiro nível; o alinhamento de colocação deixa de ser `D3D12_SMALL_RESOURCE_PLACEMENT_ALIGNMENT` para texturas ≤ 64 KiB e passa a múltiplos de 64 KiB, salvo `VICTORY_POOL_SMALL=1`; a checagem "nível além do pedaço" fica em nível de erro. |
| 122d3546 | local | Frame meter's lines of every second on its thread too; askings without the kernel's reset (VICTORY_ASK_NORESET) and on several threads (VICTORY_ASK_SPREAD), both off and not yet measured | `host.rs` e `watch.rs` (+192/−106): as linhas de 1 s e de 5 s passam a ser montadas por `meter_lines(...)` numa thread própria (medido: ~3 ms/s e 30 ms/5 s no thread de present); as contagens são tomadas lá. `VICTORY_ASK_NORESET` e `VICTORY_ASK_SPREAD` ficam **desligadas e não medidas**. |
| c3d5cd80 | feanorsun | Report file scan outcomes in shader-info and shaders | Seis arquivos, +233/−26: `apps/victory/src/shader/scan.rs` novo (`FileScan` com `files(root)` e `Display`) substitui o `WalkDir` inline; `--verbose` imprime uma linha por container e os caminhos pulados; `tempfile` e `tracing` entram no crate. |
| 33439c8d | Yoshi-233 | Give the pipeline book writer a bounded flush after a fatal GPU report | `pipeline/known.rs` + `known/writer.rs` (novo) + `d3d12x.rs` (+93/−14): o livro passa a `Recorder { book: Mutex<Book>, writer: writer::Writer }`, com writer próprio que dá flush limitado depois de um relatório fatal de GPU; o laço antigo (`spawn` + `sleep(2s)` + `flush`) sai. |
| 4cd2d133 | local | Textures made while lists sent before them run are shown in their slots only once those ran (VICTORY_SHOW_NEW_LATER, on; 0: at once); the heap test says what it finds | `descriptor/host.rs`, `host.rs`, `pool.rs` (+128/−8): `show_new_later()` (padrão ligado, A/B em build de diagnóstico) segura o slot que mostraria textura criada enquanto listas já enviadas ainda podem rodar; o comentário liga os device losses de `steps-ab2`, `start-3` e `play-1125-2107` a page faults no primeiro byte de textura recém-colocada com slot trocado "de imediato". Sai o `RECENT` de 1 s e o assert de referência do heap vira checagem de contenção. |
| 91018a5d | local | VICTORY_SHOW_NEW_LATER: a slot held back is held once, not again at each draw that reads it | `descriptor/host.rs` (+20): `NEW_PENDING: Mutex<HashSet<(heap, slot, chave)>>` deduplica o pedido — cada draw que lê o slot devolvido voltava a registrar a mesma espera; as entradas são removidas quando o slot fica pronto. |
| 96a7bd7b | Yoshi-233 | Decode packed 16-bit BGR diagnostic targets | `capture.rs` (+50/−4): o dump RGB passa a converter `B5G6R5` (85), `B5G5R5A1` (86) e `B4G4R4A4` (115) pelos bits reais de cada canal (`low 5 bits = azul`, `unorm8` com arredondamento), em vez de escalar o maior valor finito. |
| 01400899 | Diogo Sousa | Timing runs: the card's time by kind of list | `copystat.rs` e `host.rs` (+122/−14): entra `enum Timed { Prelude, Title(usize), Steps, Compare, Other }` e a submissão agrupa as listas por tipo quando `timing()`; antes só existia o total por submissão. |
| cf9ba084 | local | Merge fps/pool: a slot is not switched at once to a pooled texture made while lists sent before it may read it (VICTORY_SHOW_NEW_LATER), pooled textures at 64 KiB | Merge de ramo de trabalho; `--stat` do próprio merge dá 4 arquivos, +291/−6. Cobre `show_new_later` e o alinhamento de pool em 64 KiB, o mesmo conteúdo de `4cd2d133` + `c278f0d5`. |
| 85d56416 | local | The pool's check of a texture's first level past its piece only at debug level: false alarms on this driver | `pool.rs` (+4/−1): a checagem "primeiro nível além do pedaço" desce de `tracing::error!` para `tracing::debug!`; o comentário registra 56 alarmes falsos em `pool-1` porque o driver não mantém endereço = início do bloco + offset do pedaço. |
| c83e54c8 | local | Ranges asked ahead found by halving, not one by one | `watch.rs` (+76/−12): `struct Ahead { ranges, most }` guarda os ranges ordenados por início e o maior fim acumulado, e `holds(first, last)` decide por busca binária; antes cada refresh varria a lista inteira (10 000 refreshes por frame, algumas centenas de ranges). |
| 71537d00 | local | Ranges asked ahead: no is_empty of their own (only a test asked it; a warning) | `watch.rs` (+1/−5): remove `Ahead::is_empty` e o teste passa a usar `ahead.ranges.is_empty()`; nenhum código de produção chamava o método. |
| 306722bc | local | Merge fps/ask: a submission's asked ranges looked up by binary search; the memory-map walk every 5 min | Merge de ramo; `--stat` do merge dá 3 arquivos, +337/−122. Reúne `c83e54c8`, `71537d00` e `fb28d49e` — busca binária dos ranges pedidos adiantados e caminhada do mapa de memória a cada 60 linhas. |

## Pull requests abertos (20)

Sessenta PRs abertos no momento da coleta (`newest` = #374). Documentados os 20 mais recentes, todos com
número acima do cursor 109; os outros 40 ficam para o próximo ciclo. "Proposta" lida no título e no conjunto
de arquivos do PR (GraphQL `files(first:5)`), sem leitura de patch.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 374 | Replay: heaps and resources made resident; what the breadcrumbs say when the device is lost | `replay/play.rs` +115/−3: torna heaps e recursos residentes no replay e imprime o breadcrumb do DRED. | Não mapeado no catálogo. |
| 372 | GPU: the skipped shaders are looked up in a set, not a list | `pipeline.rs` +6/−2: troca a lista de shaders pulados por conjunto (o autor mede 3,9 → 2,9 µs por chamada com lista longa). | Não. |
| 369 | Up-copy check: what is known of copies that are gone is let go | `host/upcheck.rs` +35/−1: libera o registro de cópias já ausentes no teste de up-copy. | Não. |
| 368 | victory warm: a number the pipelines' file does not hold is told as not made | `pipeline/known.rs` +36/−2: trata como "não feito" o número ausente do arquivo de pipelines no `warm`. | Vizinho de D-08 (travas de compilação), sem relação direta. |
| 364 | Memory: track private BVH arena writes | `memory.rs`, `shadow.rs`, `watch.rs` +128/−20: rastreia escritas na arena privada de BVH. | Vizinho de D-05 (tamanho de BVH lido antes da submissão); mecanismo diferente. |
| 363 | Renderer port 5: table constant buffers in host copies read through their raw twin (VICTORY_PORT_CBV_COPIES=1) | `port_cbv.rs` +138/−19, `draw.rs` +23/−5: CBVs de tabela em cópias do host lidos pelo gêmeo raw, atrás de switch. | Vizinho de D-13 (leitura raw por view tipada); não é o mesmo caminho. |
| 362 | Fix the three clippy warnings that fail the diagnostics build | `watch.rs`, `descriptor_sites.rs` +2/−2. | Não. |
| 361 | Upcheck tests: copies on the card on every card (VICTORY_CARD_BUFFERS_MIN_MB=1), and the on-the-card ones say when they are not | `host/upcheck.rs` +30/−14: força cópias na placa em todos os adaptadores e reporta quando não são. | Não. |
| 360 | Diagnostics: an indirect draw's "prepare failed" names the exit that dropped it | `draw.rs` +28/−11: nomeia o `exit` que descartou o indirect draw. | Vizinho de D-01 (orçamento indireto esgotado); só diagnóstico. |
| 359 | Renderer port 2: tables the title fills whole mark every slot they overlap | `descriptor_sites.rs` +338/−15, `port_descriptors.rs` +102/−13: tabelas preenchidas inteiras marcam todos os slots sobrepostos. | Vizinho de D-01/D-13 pela via de slots; não verificado. |
| 357 | RDNA 4: a raw view's size asked through a handle of its own (amdxc64.dll+0x861b9f) | `emit/lower.rs` +70/−3, `bitcode.rs` +14: pergunta o tamanho de raw view por handle próprio na RDNA 4. | Vizinho de D-14 (índice de view além do heap). |
| 356 | xmem: opt-in lazy/file-backed physical pages for commit-limit pressure | `xmem.rs` +267/−32 (draft): páginas físicas lazy/em arquivo sob `VICTORY_XMEM_LAZY`. | Não. |
| 355 | Replay: --pixel reads a depth texture's texels from the copy of its depth plane in that plane's format | `replay/play/pixel.rs` +14/−2. | Não. |
| 354 | Shaders: a pixel program's killed lanes go on with their live quad (s_wqm), as on the console | `structure.rs` +83/−3, `lower.rs`, `emit/mod.rs`: `s_wqm` reconstrói o quad vivo. | Vizinho de D-11 (helper lanes fora das operações de wave); mecanismo distinto. |
| 353 | Shaders: a pixel program the console tests depth before (DB_SHADER_CONTROL.DEPTH_BEFORE_SHADER) is declared [earlydepthstencil] | `lower.rs` +18, `shader.rs` +3: emite `[earlydepthstencil]`. | Não mapeado. |
| 351 | Card steps: leave out copies outside their buffers instead of recording them (GPU page fault and whole-GPU reset on RADV) | `host/card_steps.rs` +24: descarta cópia fora do buffer antes de gravar. | Vizinho de D-09 (page fault por textura recém-colocada); caminho de `card_steps`, não de slot. |
| 350 | Improve D3D12 geometry, streaming and graphics controls for RX 6600 XT | 23 arquivos, +1902/−158: ajustes de geometria/streaming e controles gráficos por placa. | Não; mudança ampla, sem verificação. |
| 349 | Lower vram usage | 28 arquivos, +6854/−884: reduz consumo de VRAM. | Vizinho de D-01/D-17 (orçamentos), não verificado. |
| 348 | PE: reject unterminated and oversized TLS callback arrays | `victory-pe/src/image.rs` +96/−2: recusa arrays de callback TLS sem terminador ou grandes demais. | Não. |
| 347 | watch: clear BESIDE/opened on remap/free and fix parallel test flakes (#321) | `watch.rs` +245/−13: limpa estado `BESIDE`/`opened` em remap/free. | Vizinho de D-06 (corrida entre mapear view e abrir guards); correção de estado, não a corrida citada. |
| 344 | hle: speak the title's lines with the host's voices | `hle/modules/speech.rs` +762/−71: fala as linhas do título com vozes do host. | Não. |
| 341 | frame_rate: report the frames the title drops before presenting | `loader/src/frame_rate.rs` +219/−27: mede quadros perdidos antes do present. | Vizinho de `8699d96a` (contar quadro só depois do present); complementar. |

## Pull requests mesclados (32)

Todos com `mergeCommit.oid` **dentro** da faixa (`b0cfe2ba..2de99801`, confirmado por `merge-base
--is-ancestor`), mas a maioria além do teto de 40 commits — nesses casos a linha traz o conjunto de
arquivos do merge contra o primeiro pai, sem leitura de hunk. `mergedAt` de todos: 2026-11-26T08:17:49–50Z
(a leva inteira foi mesclada no mesmo segundo pelo bot `Merge PR #N:`).

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| 147 | Our threads prefer the big-cache cores on dual-CCD X3D Ryzens (VICTORY_CACHE_CORES) | 4c3f311a | 3 arquivos, +202/−22: `README.md`, `crates/victory-gpu/src/cores.rs`, `crates/victory-gpu/src/queue.rs` — seleção de núcleos por cache. |
| 144 | Trace shader and driver work on pipeline workers | 022db78e | 4 arquivos, +23/−3, em `pipeline/`: tracing nos workers de pipeline. |
| 140 | Fix workspace Clippy -D warnings failures across CI targets | 8ec1189d | 28 arquivos, +53/−46: quase tudo ajuste para `-D warnings`, espalhado por `victory-formats`, `victory-gpu` e `apps/victory`. |
| 139 | Honor both host wave override names in translate and translate --warm | 4dde9d57 | 6 arquivos, +134/−24; novos `apps/victory/src/translate/host_wave.rs` e teste de integração. |
| 137 | Report diagnostic settings discarded by play.ps1 | f31e8fa1 | Só `scripts/play.ps1`, +8/−1. |
| 132 | Distinguish named and ordinal imports in census missing reports | 2016f8de | Só `apps/victory/src/census.rs`, +61/−14. |
| 130 | Correct the required switches for three opt-in upcheck tests | 153cf84b | Só `host/upcheck.rs`, +4/−4. |
| 128 | Report warm worker startup failures without marking pipelines as bad | 930a35cf | Só `apps/victory/src/warm.rs`, +196/−57. |
| 127 | Run default and diagnostics tests independently of Clippy | c02db953 | Só `.github/workflows/ci.yml`, +36/−3. |
| 123 | Report storage quota errors when save usage cannot be read | 450b6124 | Só `hle/modules/storage.rs`, +104/−27 — mesmo assunto do commit `8211226`, que está fora do teto e não foi lido em hunk. |
| 121 | GPU: back host sampler heaps with 2,048 slots and reserve slot 2,047 as null sentinel | 93292908 | Só `descriptor/host.rs`, +30/−2 — mesmo assunto de `dd67ae7` (fora do teto). |
| 120 | GPU tests: fall back to 1,000,000 view descriptors in replay_big on capped adapters (#116) | d38720a2 | Só `tests/replay_big.rs`, +10/−5 — mesmo assunto de `d0aaed0f` (41º commit, fora do teto). |
| 117 | Report file scan outcomes in shader-info and shaders | bd687863 | 6 arquivos, +233/−25, `shader/scan.rs` incluído — praticamente o mesmo diff de `c3d5cd80` (+233/−26). |
| 106 | Timing runs: the card's time by kind of list | 1778ac5f | 2 arquivos, +122/−14 — mesmos números de `01400899`. |
| 104 | Decode packed 16-bit BGR targets in diagnostic captures | dfeabb14 | Só `capture.rs`, +50/−4 — idêntico a `96a7bd7b`. |
| 90 | Docs: switches as the code reads them, and an xtask check for dead names (#49) | 1eed7793 | 5 arquivos, +137/−17, `xtask/src/switches.rs` incluído — idêntico a `d792ebd4`. |
| 89 | VICTORY_WRITE_SITES: refuse a file inside the source tree (#33) | e102fca1 | 3 arquivos, +39/−4 — idêntico a `effa1dc8`. |
| 82 | Translate tiled cascade shadows exactly on wave32 GPUs | 0cb13721 | 2 arquivos, +332/−30 — idêntico a `4fd34fdd`. |
| 78 | Drop the invalid play.ps1 skip default and fail on missing executables | 4651f89c | Só `play.ps1`, +3/−3 — idêntico a `5cfe32df`. |
| 77 | GPU: opt-in system-RAM budget for idle windows | 4c04dda3 | 2 arquivos, +238/−6 — idêntico a `ffc2cf6f`. |
| 69 | GPU: add optional memory category log | 1a7d64a2 | Só `copystat.rs`, +71/−3 — idêntico a `70cc7d2e`. |
| 66 | Shaders: count threads of wave64 builder programs on 32-lane hosts (NVIDIA) | 29c64295 | 2 arquivos, +188/−16 — idêntico a `06fe22a7`. |
| 63 | Retry incomplete pipeline book writes and flush on orderly exit | 15f1a48e | 7 arquivos, +199/−21: junta `1438f09d` (+107/−8, 5 arquivos) e `33439c8d` (+93/−14, 3 arquivos), com um arquivo em comum. |
| 39 | GPU: state the DRED stop command as the first uncompleted op (#22) | eb797463 | Só `host.rs`, +66/−4 — idêntico a `787c7b43`. |
| 24 | Skip unrelated buffer slots in each spread heap scan | ab9f1b72 | 2 arquivos, +40/−16 — idêntico a `888bf1e2`. |
| 23 | Keep structured buffer reads raw on heaps without descriptor twins | 86bb00ec | Só `descriptor/host.rs`, +49/−16. Não é idêntico a `b68e65e0` (+40/−16): o merge tem 9 linhas somadas a mais. |
| 17 | Count a frame only after DXGI accepts the present | 64d2b76c | Só `host.rs`, +41/−3 — idêntico a `8699d96a`. |
| 14 | copies of title memory: a guard page fails the copy and stays a guard page | 150f287e | 2 arquivos, +165/−4, `safecopy.rs` + `watch.rs` — mesmo assunto de `fa116500` e `c2775339` (o merge soma os dois, −4 contra −4/−15). |
| 9 | replay: defer clamp scratch copy until split transition completes | 0dc8c655 | 2 arquivos, +98/−29 — idêntico a `3bb5980a`. |
| 7 | Share GPU selection between runtime and shader validation | f3f6bb62 | 5 arquivos, +177/−32 — praticamente `0a4253f7` (+175/−33). |
| 5 | Fix resource-state and binding errors in the replay fixture | 2ba2a664 | Só `tests/replay.rs`, +43/−27 — soma o efeito de `b295a683` e `631aadc9` no mesmo arquivo; o diff é diferente dos dois isolados. |
| 4 | Shaders: run LDS flag regions on 32-lane GPUs | a5e42338 | 3 arquivos, +398/−2 (`shader_cache.rs`, `lower.rs`, `wave64_tests.rs`) — cobre `ad0f9dda`, `25025d2f` e `e29bfe27`. |

## Cruzamento com o catálogo

- **Corrige D-09?** (`Device loss a cada ~16 min: textura mostrada antes de o upload rodar`, `corrigido`
  vazio no catálogo). `4cd2d133` + `91018a5d` + `cf9ba084` implementam `VICTORY_SHOW_NEW_LATER`, **ligado por
  padrão**, que segura o slot até as listas enviadas antes terminarem. O comentário do commit descreve o
  mesmo mecanismo do defeito — page fault no primeiro byte de textura colocada naquele milissegundo, com o
  slot trocado "de imediato". Há também `c278f0d5`, que passa a colocar texturas ≤ 64 KiB em múltiplos de
  64 KiB para evitar o nível superior além do pedaço. **NAO VERIFICADO** em hardware: nenhum log de run com
  a mudança foi anexado à faixa, e o próprio `85d56416` registra 56 alarmes falsos do verificador no driver
  desta máquina. Não gravei hash em `corrigido` — o catálogo só deve receber hash depois de uma execução sem
  device loss.
- **Corrige D-19?** (`A configuração documentada não é a configuração medida: play.ps1 liga ~20 switches`,
  introduzido em `bddfb4a5`). Parcial, e por dois lados: `5cfe32df` remove `VICTORY_SKIP_PIPELINES='none'`
  do `play.ps1` (valor que nenhum switch lê) e `d792ebd4` cria `xtask/src/switches.rs`, que compara todo
  `VICTORY_*` citado no Markdown com os que o código lê. Isso fecha a classe "nome documentado que não é
  lido", não a classe "conjunto de switches ligados por padrão difere do medido". `corrigido` vazio
  permanece correto.
- **Toca D-16?** (`Duas convenções opostas para o mesmo tipo de switch, em 294 variáveis`). `d792ebd4` só
  verifica existência de nome, não convenção de valor; `122d3546` acrescenta dois switches (`VICTORY_ASK_NORESET`,
  `VICTORY_ASK_SPREAD`) seguindo a convenção `1`/ausente. Não corrige.
- **Abre defeito novo?** Dois candidatos, ambos **NAO VERIFICADO**:
  1. `e29bfe27` prova que a chave de cache **não** incluía `VICTORY_LDS_FLAGS` até este commit
     (`crates/victory-gpu/src/shader_cache.rs`). Todo binário anterior que alternasse a flag reutilizava
     tradução feita sob o outro valor — a mesma classe do D-02 do `victory-fork`, com outro nome de switch.
     O commit corrige, mas o período em que o cache esteve envenenado não é delimitado por teste.
  2. `fb28d49e` documenta que a caminhada do mapa de memória pergunta ao Windows por ~250 000 regiões
     (0,9 s) segurando o mapa do processo, e que as *askings* de write-watch e os faults do título esperam
     por ela (11 e 13 ms em `lane2-0`). O commit reduz a frequência (6 → 60 linhas) mas não elimina o
     bloqueio; em build de diagnóstico a pausa de 0,9 s continua existindo a cada 5 min.
- **Fecha lacuna de catálogo, sem hash:** `70cc7d2e` registra no próprio código que as categorias de memória
  se sobrepõem e não podem ser somadas — um leitor que somasse a linha `VICTORY_MEMORY_CATEGORIES=1` teria
  número inflado. Não é defeito de comportamento; é armadilha de leitura da saída.

## Confiança

- **Verificado por diff:** os 40 hashes foram lidos em clone local `--filter=blob:none` de
  `avatarmanz/VIctory-7900XTX` (commit `2de99801`), com patch `-U0` por commit, lista de arquivos e
  `--shortstat`; os dois merges de ramo (`cf9ba084`, `306722bc`) foram conferidos por arquivos e estatística.
- **Cobertura:** 40 de 147 commits da faixa. `last_commit` avança para `306722bc` (último documentado);
  `commit_cursor` = `306722bc`, `head_at_run` = `2de99801`. Os 107 restantes (a partir de `d0aaed0f`, que
  inclui a leva de merges do bot às 08:17Z) ficam para o próximo ciclo.
- **PRs:** 20 de 60 abertos documentados (`last_pr_number` 109 → 374, o maior documentado); 32 mesclados com
  `mergeCommit` confirmado como ancestral de `HEAD` e não ancestral do cursor. `last_run` avança para
  `2026-11-26T08:17:50Z` (último `mergedAt` documentado), **não** para o horário da execução — assim os
  merges posteriores a esse instante não são pulados.
- **Ressalva nas linhas de PR mesclado:** a coluna "o que entrou de fato" sai do subject do merge commit, do
  conjunto de arquivos (`git diff --name-status <merge>^1 <merge>`) e do `--shortstat` de cada merge, com
  leitura de hunk apenas quando o commit de conteúdo correspondente caiu dentro do teto de 40. Onde escrevo
  "idêntico", os arquivos e as estatísticas batem exatamente com o commit de conteúdo citado; onde escrevo
  "mesmo assunto" ou cito números diferentes, o merge **não** foi provado idêntico — não usei `git patch-id`.
- **Divergência registrada:** 32 PRs mesclados contra 147 commits na janela. A explicação visível é que cada
  PR deixou **dois** commits no histórico — o `Merge PR #N:` do bot e o commit de conteúdo com o mesmo
  assunto e `(#N)` no fim (por exemplo `d38720a2` e `d0aaed0f`). Isso é o mesmo padrão do defeito D-15 do
  catálogo (patch duplicado com bases diferentes); não contei os pares como commits independentes na
  descrição, mas eles existem na faixa e entraram na contagem de 147.
- **NAO VERIFICADO:** a eficácia de `VICTORY_SHOW_NEW_LATER` contra D-09 em hardware; os dois candidatos a
  defeito novo; o comportamento das provas de uniformidade (`431cab5d`, `7348b80a`, `7bf91de4`, `06fe22a7`)
  em títulos além do caso de teste; os 60 PRs abertos não documentados e os 107 commits restantes.
