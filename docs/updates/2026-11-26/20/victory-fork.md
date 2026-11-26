# fxceless/victory-fork — atualização de 2026-11-26 20:00

## Resumo

**22 commits** na faixa `988aa416..52ac4ec0`, todos documentados (abaixo do teto de 40). Duas partes bem
distintas: oito commits de diagnóstico (corner box com carga da máquina, medidor do cartão por quadro, seis
runs “scrubbed” com 197 599 e 82 174 linhas de log, split do tempo de remapeamento, instrumentos de
DirectStorage e de pipelines negados) e o miolo de desempenho — a passagem de texturas em thread própria
(`VICTORY_HEAP_WORKER`, PR #1007), checagens em paralelo, despacho compute acima de 65 535 grupos espalhado
em z, mais room para *flights* antes do settle, e cinco correções de descritor/slot vindas de PRs (#1320,
#1321, #1324, #1336, #1353). Um detalhe que vale registrar: `52ac4ec0`, o HEAD, é o squash do PR #1324 e
**não altera nenhum arquivo** contra o pai — o conteúdo do #1324 já está em `e4245f3c`.

## Commits novos (22)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| 619b2dc5 | victory-local | A constant buffer view of no bytes stays one (2ba6e1f made it 256 bytes of the host copy; a floor flickered) | `descriptor/host.rs` (+7/−1): `constant_view_size(0)` volta a ser `0` — a expressão passa de `size.max(1).next_multiple_of(256)` para `size.next_multiple_of(256)`, com asserções para 0 e 1. Antes, uma view de zero bytes arredondava para 256 bytes da cópia do host, lendo o anel de constantes por draw: o piso da primeira sala tremulava. |
| 5d659e0c | fxceless | Two switches, both off by default: a draw's stages share the tables they make afresh (VICTORY_TABLE_STAGES), and the host heap's two busiest locks are spared (VICTORY_HEAP_LOCKS) | `descriptor/host.rs` + `draw.rs` (+226/−10): `VICTORY_TABLE_STAGES=1` faz os estágios de um draw compartilharem as tabelas recém-feitas; `VICTORY_HEAP_LOCKS=1` espalha o `kept` em `KEPT_PIECES = 64` locks por `i % 64` e permite achar em `made_now` sem o lock dos slots. O comentário mede a linha de base: 49,6 ms/frame esperando lock no run 298. Ambos **desligados por padrão**. |
| 2095f154 | fxceless | The corner box shows the machine's load beside the frame rate: processors, the busiest thread, the card, its memory, the PC's memory | `fpsbox.rs` + `stats.rs` + `lib.rs` (+617/−53): a caixa passa de uma linha (FPS/ms) para seis linhas atualizadas a cada segundo — CPU%, núcleos usados, thread mais ocupada, GPU por engine, VRAM usada/orçamento, RAM commitada e RAM livre do PC. O cabeçalho documenta como ler: thread a ~100 % com cartão ocioso = quadro preso na thread. |
| 0672327d | fxceless | Diagnostics: a line of the card's direct queue per frame: a list of ours running, and the gaps between lists by their length | `cardmeter.rs` (+82): `spans: Vec<(u64,u64)>` guarda início/fim de cada lista no relógio da fila e `gaps_line` publica, por quadro, quanto tempo uma lista nossa rodou (união dos stamps) e quanto tempo nenhuma rodou, agrupado por tamanho do buraco. Antes a linha “card: idle” só dizia o tempo sem nada enfileirado. |
| 8ef60971 | fxceless | diagnostics: runs 301 to 306 (…), scrubbed | Só `diagnostics/`: README + seis logs, **197 599 linhas**. Números registrados: `VICTORY_HEAP_LOCKS=alt` leva o lock dos slots de 79 142 para 10 240 tomadas/frame e a espera de 53,3 para 15,1 ms, sem ganho de quadro (+0,7 ± 5,2 ms); tabelas compartilhadas levam a taxa de acerto de 39,0 para 45,9 % com **+7,4 ms** de quadro; `VICTORY_DISPLAY_MODE=1920x1080` é reportado e **ignorado** pelo título, que continua apresentando 3840x2160; `--console series-s` apresenta 1920x1080 com imagem preta. |
| be721d5a | fxceless | Merge pull request #1260 from LeonardoS11: a constant buffer view of no bytes stays one | Merge do PR #1260; 1 arquivo, +7/−1 — conteúdo idêntico ao de `619b2dc5` (o mesmo `constant_view_size`). |
| d4a528ba | fxceless | Textures picked by index can be uploaded by a thread of their own (VICTORY_HEAP_WORKER=1, off by default; fork pull request #1007, its worker commit 233a65f4) | Dez arquivos, +542/−16: `heap_worker.rs` novo e `View::view_and_key(handle)` (lê `[tag, i, generation]`, valida `VIEW_TAG`) para o worker publicar/consumir views; `heap_textures` sai do caminho síncrono do draw e passa a um thread próprio atrás de `VICTORY_HEAP_WORKER=1` (**desligado**). |
| d8986cd7 | fxceless | The frame log splits the remapping time: the system's unmapping, its mapping, committing, our own notes; views cut and the bytes mapped again | `framelog.rs` + `hle/modules/xmem.rs` (+40/−1): o tempo de remapeamento ganha seis contadores (`RemapUnmapUs`, `RemapMapUs`, `RemapCommitUs`, `RemapNotesUs`, `RemapCuts`, `RemapCutBytes`) e a linha passa a imprimir a divisão e quantas views foram cortadas e remapeadas. |
| 46bf5351 | fxceless | diagnostics: runs 307 to 309 (…), scrubbed | Só `diagnostics/`: README + três logs, 82 174 linhas. Run 307 mede 238 chamadas e 45 ms por quadro usual (unmap 25,1 / map 17,7 / notas 1,5), 979 chamadas e 197 ms em quadro de streaming; run 308 registra **nenhuma** cópia de page-mapping repetida em 103 024 páginas; run 309 mede `VICTORY_HEAP_WORKER=1` levando a passagem de 96–116 ms para 33 ms, sem mudar quadros apresentados (9,3/s). |
| f6a09d3b | fxceless | The texture pass on its own thread measured (+40 % world frames at the boat); legacy multiply-add as RDNA 2 runs it; … | Nove arquivos, +1 439/−58: README de status reescrito (8,2–8,6 fps no barco com o worker, 6,2–6,7 sem), `emit/lower.rs` faz o multiply-add legado rodar fundido com fontes descarregadas como na RDNA 2, views tipadas de inteiro de 32 bits passam a ser feitas como float da mesma largura para mover os bits intactos, e entram instrumentos de remoção de device (16 s antes) e de quadro de mundo. |
| 06702fe8 | fxceless | More room for flights before they are settled (…); compute dispatches past 65,535 groups spread across z; blank texture slots asked again; BC7 rotation fix; … | 19 arquivos, +1 456/−64: `VICTORY_SETTLE_ROOM=1` (medido −11,5 ms do trabalho da thread de preparo), despacho compute acima de 65 535 grupos dividido em z (o título pede até 115 200 grupos e perdia 43 %), `VICTORY_REASK_BLANK=1` (slots que liam zeros caem de ~385 para 0 no barco), correção de rotação BC7 só nos modos 4 e 5 (`victory-formats/src/bcpack.rs`), instrumentos de espera do DirectStorage e de pedidos falhos, e caminho para negar build de pipeline. |
| ceb9e1f7 | fxceless | Texture checks side by side on the helper threads (…); the slowest DirectStorage requests; … VICTORY_DISPATCH_SPREAD | Seis arquivos, +270/−6: `VICTORY_PARALLEL_REFRESH=1` roda as checagens de textura de uma submissão em paralelo (−8,7 e −9,4 ms, duas medições), o switch do espalhamento é renomeado de `VICTORY_SPREAD` para `VICTORY_DISPATCH_SPREAD` (valor `alt`), e o README registra a descoberta de que com ambos ligados o gargalo passa a ser o **cartão** (84–128 ms/quadro). |
| a20809de | fxceless | Meters for the card and the page mappings, and a dump of the console's native pipelines (all off by default; the build of run 334) | Seis arquivos, +566/−28: `cardmeter.rs` ganha amostragem de clamp (`VICTORY_CARD_METER_CLAMPS=<n>`, 1–4096, com dois timestamps e um resolve por clamp medido), `device.rs`/`queue.rs` medem mapeamento de páginas, e o dump de pipelines nativos do console é escrito por switch. Tudo desligado por padrão. |
| 58de9bf4 | fxceless | Mouths move: 16-bit values moved through a table's typed views as they are (pull request #927), SDWA sign extension of 16-bit results (upstream VIctory #3124); … | 23 arquivos, +1 884/−62: `VICTORY_NATIVE_HEADS=<file>` grava os primeiros bytes de cada programa nativo com a linha do container (para casar os métodos 0x1c8/0x218), `VICTORY_HEADS_ONLY=1` lista sem traduzir; correções de view tipada de 16 bits e de extensão de sinal SDWA; `clamp windows` e `occlusion queries` entram como switches desligados. |
| 9613092b | fxceless | Translation diagnostics for the two refused 64-lane water programs (5dc3b25986622949, 6f225b641c5b0708) | `emit/lower.rs`, `emit/mod.rs`, `diagnostics/` (+208/−10): `VICTORY_WAVE_WHY=1` lista, por região que lê e escreve LDS, as instruções `Fields::Ds` (exceto `ds_swizzle`/`permute`) e o registrador de endereço que as alimenta, para explicar por que dois programas de água de 64 lanes continuam recusados. |
| 93db653a | fxceless | Fix the build: the second deep-skips form names itself for VICTORY_WAVE_WHY too | `emit/mod.rs` (+1/−1): `form` passa a receber a descrição (`"without skips, scalars followed"`) junto com `ir::without_skips(p, true)`; antes a chamada tinha aridade errada e não compilava. |
| e4245f3c | fxceless | Card hashing a group of threads a piece (measured: card -10 ms a frame), 4 KiB dirty tracking, gather segments, the black-face texture fixes, shadow and translator fixes from pull requests | 21 arquivos, +2 320/−185: agrupamento de threads no hash do cartão (−10 ms/quadro medidos), rastreio de sujo em 4 KiB, segmentos de gather, “texturas que o cartão não tinha espaço” passam a ser pedidas de novo (#3198 upstream), correções de textura de rosto preto, e — importante para o #1324 — `release_dynamic(heap, id, first, n)` passa a exigir a identidade do heap (`HostHeap::id`) antes de devolver slots dinâmicos, porque o título pode liberar o heap e outro nascer no mesmo endereço. |
| af23c652 | cccw | GPU: the lists a split submission runs at once give their descriptor tables' slots back (#1320) | `queue.rs` (+6): chama `crate::list::tables_done(&lists[..cut])` depois que o grupo de uma submissão dividida rodou — sem isso, os slots dinâmicos das tabelas frescas não voltavam antes de o título resetar as listas. |
| 2440c7bb | cccw | Shader cache: a staging file a killed process left behind no longer loses the translation (#1321) | `victory-pipeline/src/cache.rs` (+58/−7): se `create_new` do arquivo de staging falhar com `AlreadyExists` (PID reciclado pelo Windows), o arquivo é aberto com `truncate(true)` e reescrito; antes a conversão já feita era perdida e aquele shader nunca traduzia naquela execução. |
| 46da5eec | Filip Michalski | GPU: finish a pipeline queue entry when its build panics (#1353) | `pipeline/make.rs` (+33/−1): o build do pipeline roda dentro de `catch_unwind(AssertUnwindSafe(...))` e, no `Err`, loga `host pipeline build panicked` e devolve `None` — a entrada da fila é completada e os waiters acordam. Antes o panic deixava a entrada pendente. |
| a463dcb1 | cccw | GPU: pin the texture a placed view aliases while the copy uses it (#1336) | `draw.rs` + `resource.rs` (+63/−5): `Held`/`alias_of` passam a segurar a textura do título com `object::add_ref_alive` **sob o lock** e a devolver um handle que a mantém viva durante a cópia; antes devolvia o ponteiro nu, e o título podia soltar a última referência no meio da cópia. |
| 52ac4ec0 | cccw | GPU: dynamic descriptor slots go back only to the heap that took them (#1324) | **Diff vazio** contra o pai (`git diff 52ac4ec^ 52ac4ec` = 0 arquivos). O conteúdo do #1324 — `release_dynamic` com identidade de heap — já entrou em `e4245f3c`; este commit é o registro do merge, sem alteração própria. Registrado por ser o HEAD da faixa. |

## Pull requests abertos (20)

Sessenta PRs abertos na coleta (`newest` = #1380). Documentados os 20 mais recentes, todos acima do cursor
1258. "Proposta" lida no título e no conjunto de arquivos (GraphQL), sem leitura de patch.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 1380 | clzero on a guarded line faults into the guard instead of ending the run | `loader/src/console_insn.rs` +151/−13: trata `clzero` em linha de guarda. | Não mapeado no catálogo. |
| 1378 | run: VICTORY_CONSOLE picks the console model from the environment | `apps/victory/src/main.rs` +8/−1. | Não. |
| 1377 | GPU: retain depth-plane SRV descriptors for command-list lifetime | `host.rs` +56/−45, `resource.rs` +16: mantém descritores SRV do plano de profundidade vivos enquanto a lista existe. | Vizinho de D-10 (cópia de textura sobre si mesma) só pelo tipo de objeto. |
| 1376 | GPU: re-register invalidated texture readback tickets | `list.rs` +63, `texback.rs` +16: re-registra tickets de readback invalidados. | Não. |
| 1374 | GPU: include compute PSO creation details in failure logs | `pipeline/host.rs` +11/−2. | Não. |
| 1373 | GPU: reuse dirty descriptor set allocations between heap passes | `descriptor/host.rs` +25: reaproveita alocações sujas entre passagens de heap. | Vizinho de D-07 (registros de descritor não espelhados); mecanismo distinto. |
| 1372 | GPU: forget descriptor range caches when their heap is destroyed | `descriptor.rs` +1, `draw.rs` +41/−4: descarta cache de faixa quando o heap morre. | Vizinho de `e4245f3c`/`52ac4ec0` (identidade de heap); complementar. |
| 1371 | GPU: preserve RTV and DSV descriptor values across command-list cuts | `descriptor.rs` +56, `draw.rs` +30/−23: preserva RTV/DSV em corte de lista. | Não. |
| 1369 | Restore shader lowering modes after a worker panic | `emit/lower.rs` +56/−12: restaura modo de lowering depois de panic de worker. | Complementa `46da5eec` (#1353) por outro lado; não é defeito do catálogo. |
| 1367 | GPU: a list's small first scratch chunk may be a kept 4 MiB one; 512 MiB of spare chunks (follow-up to #1349) | `host.rs` +52/−15: chunk pequeno inicial reaproveitado; 512 MiB de chunks de reserva. | Não. |
| 1366 | GPU: validate host fence completion before background publication | `host.rs` +35/−11: valida fence do host antes de publicar em background. | Vizinho de D-10/D-13 (publicação); não verificado. |
| 1365 | GPU/watch: batch GetWriteWatch queries by memory reservation | `watch/reservation.rs` +453: agrupa `GetWriteWatch` por reserva de memória. | Vizinho de D-09 (sondagens de 1 byte); mesma família de custo de watch. |
| 1364 | Diagnostics: record exact CPU present-call intervals | `framelog.rs` +7, `Measure-FrameIntervals.ps1` +54: intervalos exatos do present na CPU. | Não. |
| 1362 | GPU: opt-in loose-window reuse via key aliasing and superseded reclamation (VICTORY_WINDOW_REUSE=1) | `shadow.rs` +294/−43: reuso de janela ociosa por alias de chave, atrás de switch. | Não. |
| 1361 | Policy: allow up to 2 MiB for Rust source files under asset policy | `xtask/src/policy.rs` +34, `docs/asset-policy.md`: teto de 2 MiB para fonte Rust. | Não. |
| 1360 | GPU: opt-in hot-block pinning in watch::guard (VICTORY_WATCH_PIN, addresses #1238) | `watch.rs` +225/−8: fixa blocos quentes no `watch::guard`. | Vizinho de D-09 (custo de guarda); endereça #1238, fora do catálogo. |
| 1359 | texback: reuse read-back buffers instead of making one per copy-back | `texback.rs` +111/−6: reaproveita buffers de readback. | Não. |
| 1358 | GPU: VICTORY_COLD_SKIP=1: refreshes pass over 2 MiB chunks with no write since (off by default) | `cold_skip.rs` +390: pula chunks sem escrita desde o último refresh. | Vizinho de D-09/D-13; altera o que é copiado, não verificado. |
| 1357 | Move host.rs's and lower.rs's test modules to their own files, under the asset policy's 512 KiB limit | 11 arquivos, +1407/−1415: só movimentação de testes. | Não. |
| 1354 | Tests: cover tiled mip claims without a device | draft; `tiles.rs` +132: testes de claim de mip em tile sem device. | Não. |

## Pull requests mesclados (6)

Todos são squash merges na linha principal: o `mergeCommit.oid` **é** o commit da faixa (um pai só), então
"o que entrou de fato" é o próprio diff já descrito acima.

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| 1353 | GPU: finish a pipeline queue entry when its build panics | 46da5eec | `pipeline/make.rs`: `catch_unwind` em volta do build, `None` + warning no panic. `mergedAt` 2026-11-26T18:05:03Z. |
| 1336 | GPU: pin the texture a placed view aliases while the copy uses it | a463dcb1 | `draw.rs`/`resource.rs`: `Held` com `add_ref_alive` sob o lock. `mergedAt` 18:05:10Z. |
| 1324 | GPU: dynamic descriptor slots go back only to the heap that took them | 52ac4ec0 | **Nada** neste commit: diff vazio, um pai só. O conteúdo (`release_dynamic` com `HostHeap::id`) está em `e4245f3c`. `mergedAt` 18:05:17Z. |
| 1321 | Shader cache: a staging file a killed process left behind no longer loses the translation | 2440c7bb | `victory-pipeline/src/cache.rs`: staging existente é truncado e reescrito. `mergedAt` 18:05:00Z. |
| 1320 | GPU: the lists a split submission runs at once give their descriptor tables' slots back | af23c652 | `queue.rs`: `tables_done(&lists[..cut])`. `mergedAt` 18:04:52Z. |
| 1260 | GPU: a constant buffer view of no bytes stays one | be721d5a | Merge de 2 pais; `descriptor/host.rs` +7/−1 — o `constant_view_size` de `619b2dc5`. `mergedAt` 2026-11-26T05:16:35Z. |

## Cruzamento com o catálogo

- **Introduz defeito que o catálogo não tem, e o corrige na mesma faixa:** `619b2dc5` fecha uma regressão de
  `2ba6e1f` (CBV de zero bytes virou 256 bytes da cópia do host; o piso da primeira sala tremulava). Não há
  linha correspondente em `victory-fork`. Candidato a linha nova com `introduzido=2ba6e1f`,
  `corrigido=619b2dc5` — **NAO VERIFICADO** fora do relato do commit (não há log de run anexado à faixa
  mostrando o piso tremulando antes e estável depois).
- **Continua D-06** (`Textura escolhida por índice era enviada antes de o título preencher a memória`,
  corrigido em `a803c87`). `d4a528ba` move a passagem de texturas por índice para um thread próprio
  (`VICTORY_HEAP_WORKER=1`, **off**), o que só é seguro porque a correção de ordem de `a803c87` já existe; o
  run 309 mede 96–116 ms → 33 ms sem erro e **sem** mudança de quadros apresentados.
- **Continua D-16** (`Perdas de device sem causa identificada`). O README reescrito em `f6a09d3b` registra:
  “the graphics card is still lost now and then 30 to 50 minutes into a session, before the title's next
  save point; cause not known yet”. D-16 permanece aberto; esta faixa acrescenta instrumentos (16 s de log
  antes da remoção, pipelines desenhados recentemente) mas nenhuma correção.
- **Toca D-14 sem resolvê-lo** (`Alegação de ganho de 6,0 → 8,9 fps comparava cenas diferentes`). As
  medições desta faixa repetem “at equal draws” e “frames counted only where every frame is a world frame”
  (`f6a09d3b`, `ceb9e1f7`), e o run 306 (`8ef60971`) descarta uma alternância porque “the scene changes by
  itself there”. É disciplina de medição aplicada, não uma linha de catálogo.
- **Abre defeito novo?** Três candidatos, **NAO VERIFICADO**:
  1. `e4245f3c` pede de novo as texturas que o cartão não tinha espaço (#3198) — o comentário conta as
     ocorrências e quantas foram feitas depois (`NOT_MADE_FOR_NOW`, `MADE_AFTER_NO_ROOM`), mas não há teto:
     uma cena com pressão permanente de VRAM pode repetir o pedido a cada uso.
  2. `06702fe8` espalha despachos acima de 65 535 grupos em z. O comentário diz “No visible change yet (the
     pass most likely gathers ray-traced light, which does not run)”: o caminho novo não foi exercitado de
     forma observável.
  3. `52ac4ec0` é um commit **vazio** cuja mensagem descreve uma mudança que já estava em `e4245f3c`. Quem
     procurar o #1324 por `git log -S` acha o commit errado — e o histórico passa a ter um commit que afirma
     ter feito algo que não fez. Não é defeito de runtime; é defeito de rastreabilidade.
- **Fecha lacuna de leitura, sem hash:** `8ef60971` (run 303) registra que `VICTORY_DISPLAY_MODE=1920x1080`
  é **reportado e ignorado** pelo título, que continua apresentando 3840x2160 e renderizando 2560x1440. Quem
  ligar o switch esperando mudança de resolução não terá nenhuma.

## Confiança

- **Verificado por diff:** os 22 hashes foram lidos em clone local `--filter=blob:none` de
  `fxceless/victory-fork` (commit `52ac4ec0`), com patch `-U0`. O merge `be721d5a` foi comparado com
  `619b2dc5` (mesmo arquivo, mesmas linhas). A vacuidade de `52ac4ec0` foi medida (`git diff --name-status`
  e `--stat` contra o primeiro pai: zero arquivos).
- **Cobertura:** 22 de 22 commits. `last_commit` avança para `52ac4ec0` (= head). Nenhum backlog de commits
  neste repositório.
- **PRs:** 20 de 60 abertos documentados (`last_pr_number` 1258 → 1380); 6 mesclados. `last_run` avança para
  `2026-11-26T18:05:17Z` (último `mergedAt` documentado).
- **Divergência registrada:** os commits de 2026-11-27 (`af23c652`, `2440c7bb`, `a463dcb1`, `52ac4ec0`) têm
  data de autor no dia seguinte ao da execução (UTC−3 em relação aos `mergedAt` de 2026-11-26T18:05Z). Não é
  erro de coleta: `git log --date=short` mostra a data do autor no fuso do autor.
- **NAO VERIFICADO:** os três candidatos a defeito; a estabilidade de `VICTORY_HEAP_WORKER=1` em sessões
  longas (o único número na faixa é uma execução de 180 s, run 309); o efeito do espalhamento em z em passes
  que realmente rodam; os 40 PRs abertos não documentados.
