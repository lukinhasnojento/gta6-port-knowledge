# DexNom/VIctory — atualização de 2026-11-28 16:00

## Resumo

Nenhum commit novo (head `4ad8692`, parado desde anteontem), mas 52 PRs abertos acima do cursor #148,
e 42 deles são `cherry-pick -x` de commits que o publish `570a941` derrubou da `main`. É a primeira
vez que o critério 1 da rotina aciona por hash: o PR `#183` restaura `274f698`, que o catálogo
registra como o conserto de **D-12** (`clwb_loop` com overflow de u64). O mesmo publish é apontado
pelo catálogo como introdutor de D-01, D-04, D-05, D-06, D-08, D-09 e D-14 do repositório. Dois PRs
não são para merge (`#155`, `#173`) e ficam fora da contagem de restauração.

## Commits novos (0)

`compare(4ad8692...4ad8692)` = `identical`, 0 commits. O repositório não recebeu push desde a coleta
anterior.

## Pull requests abertos (52)

Todos acima de #148; 100 lidos (o cursor lê no máximo 100 por página, então a contagem de 52 é o
total desta página, não um piso). O commit-raiz comum é `570a941` ("Publish DexNom Runtime Radeon
sources with upstream credits and build instructions", 2026-11-25T12:18:03Z, 170 arquivos,
+49.715/−36.762), ancestral da `main` atual (`compare(570a941...4ad8692)` = 20 à frente).

### Restaurações (42)

| # | Título | Proposta (diff) | Cobre defeito conhecido? |
|---|---|---|---|
| #156 | HLE clears the result of blocked address lookups | `cherry-pick -x` de `2b13ae4`; `victory-hle/src/modules/net.rs` +57/−4 | Não por hash |
| #157 | HLE: guard dialog commands before initialization | `cherry-pick -x` de `9ccb83c`; `modules/xgameui.rs` +18/−1 | Não |
| #158 | policy: resolve a `..` in the not-yet-existing part of a path | `cherry-pick -x` de `52f0a09`; `victory-core/src/policy.rs` +13 | Não |
| #159 | fix(hle): honor the requested executable on relaunch | `71e8866`; `modules/xgame.rs` +82/−8 | Não |
| #160 | fix(hle): read manifest fields past inline XML comments | `ca362a4`; `hle/src/runtime.rs` +41/−2 | Não |
| #161 | fix(pe): keep every name for a shared export ordinal | `d0dd9fc`; `victory-pe/src/image.rs` +79/−24 | Não |
| #162 | fix(pe): validate section raw data bounds | `e57e76a`; `victory-pe/src/image.rs` +94/−1 | Não |
| #163 | HLE: a `>` in a quoted manifest attribute does not end its tag | `d549b63`; `hle/src/runtime.rs` +25/−1 | Não |
| #164 | Audio: XMA read block past the ring's end wraps instead of filling forever | `1054fe8`; `modules/audio.rs` +85/−1 | Não |
| #165 | XAsync: a new block where a result was left behind begins a call | `5cff000`; `modules/xasync.rs` +77/−1 | Não |
| #166 | Media: a share is refused however it is spelled | `11ca3ce`; `modules/media.rs` +103/−5 | Não |
| #167 | Audio: an XMA loop's skip is of its first frame, not its last | `985d8d4`; `modules/audio.rs` +72/−2 | Não |
| #168 | Kernel: a title image's file name that fits clears the last error | `3c7d845`; `modules/kernel.rs` +49/−1 | Não |
| #169 | XTaskQueue: terminating a queue cancels its delayed callbacks too | `b9ff219`; `modules/xasync.rs` +54/−9 | Não |
| #170 | XAsync: end the call that finished, not whatever call the block holds now | `d01b170`; `modules/xasync.rs` +155/−14 | Não |
| #171 | HLE: require two hex digits in percent escapes | `75fe6c0`; `modules/media.rs` +12 | Não |
| #172 | XStore: wire offline `*Result` slots and fix OOB `XStoreShowProductPageUIResult` slot | `2c22e2b`; `modules/xstore.rs` +133/−5 | Não |
| #174 | Files: a title run by its file name alone gets its own folder as G: | `01ae55a`; `modules/files.rs` +19/−1 | Não |
| #176 | GPU: guard zero/overflowing stride in `expand_rects` and add `primitive.rs` tests | `e3c9dd0`; `victory-gpu/src/primitive.rs` +87/−2 | Área de **D-10** (`subresource_len`/stride), hash diferente |
| #177 | Formats: reject multisampled linear texture layouts | `350f91c`; `formats/src/texture.rs` +22/−1 | Não |
| #178 | Loader: `clzero` loops in one exception; `rdpru` | `0de9efc`; `loader/src/crash.rs` +277/−3 | Não |
| #179 | Shaders: do not parse VS inputs when reading mesh shader stage I/O | `4e155a3`; `victory-shader/src/native.rs` +27/−15 | Não |
| #180 | fix(pipeline): retry collisions when creating cache staging files | `a32eacd`; `victory-pipeline/src/cache.rs` +87/−10 | Área de **D-11** (`.tmp` de nome fixo), hash diferente (`a9d233f` é o do catálogo) |
| #181 | XStore: a store id without an installed package is an invalid argument | `e3bef08`; `modules/xstore.rs` +15/−3 | Não |
| #183 | Loader: guard `clwb_loop` shortcut against u64 overflow and wraparound | `274f698`; `loader/src/crash.rs` +59/−8 | **Sim: D-12** — reintroduz `clwb_loop_end` com `checked_mul`/`checked_add` e `(end > limit).then_some(end)`, exatamente o helper que o catálogo diz estar ausente |
| #184 | Loader: a `clwb` in the last bytes of a page is emulated instead of faulting | `60d2cf9`; `loader/src/crash.rs` +68/−28 | Não |
| #185 | XGameUi: a text entry's limit is the console's | `936e5e5`; `modules/xgameui.rs` +62/−7 | Não |
| #186 | HLE: complete `XPackage` progress callback unregistration | `4f54c47`; `modules/xpackage.rs` +39/−1 | Não |
| #187 | Shaders: `s_atc_probe`'s immediate is not a register | `3d71ad1`; `gfx10/fields.rs` + `gfx10/sp3.rs` +100/−9 | Não |
| #188 | HLE: load a host DLL outside the module cache's lock | `c6959ac`; `hle/src/host.rs` +96/−4 | Não |
| #189 | xma: start the frame after a continuation at the next packet that has frames | `ba42db6`; `formats/src/xma.rs` +43/−4 | Não |
| #190 | Shaders: fold empty loop latches containing only dead phis | `be3202d`; `emit/bitcode.rs` +40/−1 | Não |
| #191 | Shaders: print only the operands an instruction has in SP3 text | `6325df7`; `gfx10/sp3.rs` +72/−4 | Não |
| #192 | Census: report directory traversal failures | `b1bdd89`; `apps/victory/src/census.rs` +49/−8 | Não |
| #193 | Census: deduplicate PE imports without hint | `a3f89ed`; `apps/victory/src/census.rs` | Não |
| #194 | Shaders: compress replaced phi chains in `remove_trivial_phis` | `5c44562`; `victory-shader/src/ir.rs` +225/−9 | Não |
| #195 | Shaders: hardware test for the integer `v_min3`/`v_max3`/`v_med3` | `8610ffb`; `emit/valu_tests.rs` +45 | Não |
| #196 | Shaders: negative-zero ordering test for the f16 min/max forms | `259ba31`; `emit/valu_tests.rs` +25 | Não |
| #197 | HLE: test that `XGSuggestSwizzleMode` picks linear for a 1D texture | `246ecea`; `modules/xg.rs` | Não |
| #198 | GPU: add synthetic unit tests for `query.rs` and `signature.rs` | `0be58e6`; +289 | Não |
| #199 | PE: add synthetic unit tests for `implib.rs` and `layout.rs` | `131944d`; +729 | Área de **D-15** (testes sintéticos apagados por `570a941`) |
| #200 | Shader tests: require a device readback in every waterfall test | `6493dd9`; `emit/waterfall_tests.rs` +8/−8 | Não |

### Demais PRs (10)

| # | Título | Proposta (diff) | Cobre defeito conhecido? |
|---|---|---|---|
| #149 | Shaders: an atomic subtraction on group memory is a compare-exchange loop on NVIDIA | +239 em `gpu/src/host.rs`, `shader_cache.rs` e `emit/` | Não |
| #150 | Series S menus, watchdog on slow runs, and Steam Deck notes | +753/−22 em `apps/victory/src/run.rs`, `gpu/queue.rs`, `loader/watchdog.rs`, `docs/steam-deck.md` | Não |
| #151 | Add 720p scene render options | +100/−29 em `apps/victory/src/{fsr_preset,native_1440p,play_launcher,run}.rs` | Não |
| #152 | Make first-use host copies outside the heap lock; skip failing write-watch asks on view memory | +93/−15 em `gpu/descriptor/host.rs`, `gpu/watch.rs`, `hle/modules/gameinput.rs` | Já citado na triagem de 15:00; sem hash |
| #153 | Write `InitHeaders`' stores into title memory from a record of them (0,5 → 2,2 fps em RX 7900 XTX) | +313/−11 em `gpu/{capture,draw,memory,shader_cache}.rs` e `emit/` | Não |
| #154 | Keep the early pipeline list under Proton; document Linux and Proton | +129/−2 em `gpu/pipeline/library.rs`, `docs/linux-proton.md` | Não |
| #155 | Findings (not for merge): wave64 on 32-lane hosts | +322 em `docs/findings-wave64-halves-2633.md` | Não é para merge |
| #173 | Vanviro 16gb | +1.016.414/−5.149: vendoriza `cold-project-vi/` inteiro (crates, workflows, cmds) | Não é para merge; 1 M de linhas, fora do escopo da rotina |
| #175 | GPU: speed up hot PSO prebuild by hashing small descriptions in one frame | +47/−27 em `gpu/pipeline/{host,library}.rs` | Não |
| #182 | XAsync: publish the final status before removing a completed call | +99/−14 em `hle/modules/xasync.rs` | Não |

## Pull requests mesclados (0)

Nenhum `merged_at` acima de `2026-11-26T04:15:00Z` no repositório.

## Cruzamento com o catálogo

- **Corrige D-xx?** **D-12** por hash: `#183` restaura `274f698`, que o `catalogo/defeitos.csv` (linha
  de prosa de D-12) registra como conserto do overflow de u64 no `clwb_loop` e do `clwb_loop_end`
  ausente. O diff lido confirma: `clwb_loop_end` volta com `checked_mul(64)` e `checked_add(step)` e
  devolve `None` quando o `add base, 0x40` estouraria. O catálogo diz ainda que D-10, D-11 e D-12
  estão **abertos por reversão**; `#183` fecha D-12 **se for mesclado** — hoje está aberto.
- **Área, sem hash:** `#180` (colisão de `.tmp` no cache de pipeline) é a família de D-11; `#176`
  (stride em `expand_rects`) é a família de D-10; `#199` (testes sintéticos de PE) é a família de
  D-15, cuja linha diz que `570a941` apagou `synthetic_tests.rs`. Nenhum desses três traz o hash do
  conserto registrado no catálogo.
- **Abre defeito novo?** Não verificado. `#173` adiciona 1 M de linhas de um vendor inteiro
  (`cold-project-vi/`) e declara "Vanviro 16gb" — se mesclado, muda o conjunto de fontes compiladas e
  pode reintroduzir código divergente; está fora do escopo da rotina e não foi lido.

## Confiança

- **Verificado por diff:** `#183` (diff completo do helper e do teste), `#194` (diff de `ir.rs` e
  cherry-pick `5c44562`), `#156` (`net.rs`, cherry-pick `2b13ae4`), `#200` (8 substituições em
  `waterfall_tests.rs`, cherry-pick `6493dd9`). Para os outros 48, leram-se título, corpo, arquivos e
  estatísticas pela API — **não** o diff linha a linha.
- **Verificado por API:** `570a941` existe, é ancestral da `main` (20 à frente), tem 170 arquivos e
  +49.715/−36.762; os 52 PRs abertos e seus arquivos.
- **NÃO VERIFICADO:** que cada `cherry-pick` aplica **exatamente** o commit citado — a checagem foi o
  corpo do PR (`(cherry picked from commit …)`) e a mensagem do commit de topo, não uma comparação de
  patch contra o original; exceção: os quatro acima, em que o diff foi lido. A contagem de 52 é o que
  a página de 100 retorna acima de #148; não há garantia de que não exista PR mais antigo ainda aberto
  abaixo do cursor.
- **Cobertura:** 0 commits, 52 PRs lidos (4 por diff), 0 merges.
