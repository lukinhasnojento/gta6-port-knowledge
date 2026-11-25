# Divergencia entre os forks do VIctory (comparacao por conteudo)

Base: Marvisak/VIctory @ 70564ae08c82d43df5fad97b08d34628b07b1d9d (241 commits, raiz f4504f6413ce4509604dad66cdd689000a0204dc).

**Achado previo — CORRIGIDO em 2026-11-25 apos verificacao commit a commit:**

| Fork | HEAD do Marvisak e ancestral? | Commits a frente | merge-base |
|---|---|---|---|
| DexNom | **SIM** | **306** | `70564ae` |
| Belobug-AMD | NAO | n/a | nenhum |
| VItoriaBr | NAO | n/a | nenhum |
| AvatarManz | NAO | n/a | nenhum |
| victory-fork | NAO | n/a | nenhum |

Leitura correta:
- **DexNom** descende linearmente do VIctory: pode ser comparado por commit e aceita cherry-pick conceitual.
- Nos outros quatro nao existe merge-base com o HEAD do Marvisak — ou seja, o unico ancestral comum
  e a raiz f4504f6413ce4509604dad66cdd689000a0204dc. Para esses, comparacao por hash de commit e
  inutil; a unica comparacao confiavel e por conteudo (blob por arquivo), que e o que segue.

Correcao de registro: uma versao anterior deste arquivo afirmava que o HEAD do Marvisak nao era
ancestral de NENHUM dos cinco. Isso estava errado - era verdade para quatro, nao para o DexNom.

| Fork | Commits | Arquivos | So no fork | So na base | Blobs diferentes |
|---|---|---|---|---|---|
| Belobug-AMD | 751 | 186 | 30 | 18 | 78 |
| DexNom | 547 | 195 | 37 | 16 | 101 |
| VItoriaBr | 373 | 197 | 40 | 17 | 102 |
| AvatarManz | 621 | 246 | 89 | 17 | 117 |
| victory-fork | 253 | 187 | 30 | 17 | 91 |

## Belobug-AMD

### Arquivos que so existem neste fork
```
apps/victory/build.rs
crates/victory-gpu/src/ambient.rs
crates/victory-gpu/src/card.rs
crates/victory-gpu/src/compose.rs
crates/victory-gpu/src/memstat.rs
crates/victory-gpu/src/memwait.rs
crates/victory-gpu/src/pipeline/depth_bounds_test.rs
crates/victory-gpu/src/profile.rs
crates/victory-gpu/src/readout.rs
crates/victory-gpu/src/rects.rs
crates/victory-gpu/src/texpool.rs
crates/victory-gpu/src/timeline.rs
crates/victory-hle/src/modules/dualsense.rs
crates/victory-loader/tests/crash_log.rs
crates/victory-shader/src/emit/bitcode/opt.rs
crates/victory-shader/src/emit/image_tests.rs
crates/victory-shader/src/emit/lane_read_loop_tests.rs
crates/victory-shader/src/emit/lds_tests.rs
crates/victory-shader/src/emit/opcode_audit.rs
crates/victory-shader/src/emit/opcode_refusals.txt
crates/victory-shader/src/emit/opcode_tests.rs
crates/victory-shader/src/emit/rectangle.rs
crates/victory-shader/src/emit/scratch_tests.rs
crates/victory-shader/src/emit/split_tests.rs
crates/victory-shader/src/emit/threading.rs
crates/victory-shader/src/sgpr_spill.rs
crates/victory-shader/src/unspill.rs
crates/victory-shader/tests/resource_tracking.rs
docs/view-heap-limit.md
docs/wave64-on-wave32.md
```

### Arquivos da base ausentes neste fork
```
apps/victory/src/prebuild.rs
crates/victory-game/src/cityhash.rs
crates/victory-game/src/pipelines.rs
crates/victory-game/src/shaderpack.rs
crates/victory-gpu/src/bindless.rs
crates/victory-gpu/src/external.rs
crates/victory-hle/src/modules/fidelityfx.rs
crates/victory-hle/tests/fidelityfx.rs
crates/victory-shader/build.rs
crates/victory-shader/src/emit/bvh_tests.rs
crates/victory-shader/src/emit/flow_tests.rs
crates/victory-shader/src/emit/lower/bvh.rs
crates/victory-shader/src/emit/rect.rs
crates/victory-shader/src/emit/semantics_tests.rs
crates/victory-shader/src/emit/stride_tests.rs
crates/victory-shader/src/emit/synthetic_tests.rs
crates/victory-shader/src/emit/waterfall_tests.rs
crates/victory-shader/src/range.rs
```

### Arquivos com conteudo diferente
```
apps/victory/Cargo.toml
apps/victory/src/main.rs
apps/victory/src/run.rs
apps/victory/src/shader.rs
apps/victory/src/translate.rs
Cargo.lock
Cargo.toml
crates/victory-core/src/config.rs
crates/victory-formats/src/texture.rs
crates/victory-game/src/lib.rs
crates/victory-game/src/sgd5.rs
crates/victory-gpu/Cargo.toml
crates/victory-gpu/src/capture.rs
crates/victory-gpu/src/console.rs
crates/victory-gpu/src/copystat.rs
crates/victory-gpu/src/descriptor.rs
crates/victory-gpu/src/descriptor/host.rs
crates/victory-gpu/src/device.rs
crates/victory-gpu/src/draw.rs
crates/victory-gpu/src/fence.rs
crates/victory-gpu/src/frame.rs
crates/victory-gpu/src/host.rs
crates/victory-gpu/src/indirect.rs
crates/victory-gpu/src/lib.rs
crates/victory-gpu/src/list.rs
crates/victory-gpu/src/memory.rs
crates/victory-gpu/src/object.rs
crates/victory-gpu/src/pipeline.rs
crates/victory-gpu/src/pipeline/host.rs
crates/victory-gpu/src/pipeline/library.rs
crates/victory-gpu/src/pipeline/make.rs
crates/victory-gpu/src/primitive.rs
crates/victory-gpu/src/query.rs
crates/victory-gpu/src/queue.rs
crates/victory-gpu/src/resource.rs
crates/victory-gpu/src/shader_cache.rs
crates/victory-gpu/src/shadow.rs
crates/victory-gpu/src/texture.rs
crates/victory-gpu/src/twin.rs
crates/victory-gpu/src/watch.rs
crates/victory-hle/Cargo.toml
crates/victory-hle/src/modules/audio.rs
crates/victory-hle/src/modules/dstorage.rs
crates/victory-hle/src/modules/files.rs
crates/victory-hle/src/modules/gameinput.rs
crates/victory-hle/src/modules/kernel.rs
crates/victory-hle/src/modules/mod.rs
crates/victory-hle/src/modules/xasync.rs
crates/victory-hle/src/modules/xgamesave.rs
crates/victory-hle/src/modules/xmem.rs
crates/victory-hle/src/modules/xuser.rs
crates/victory-loader/Cargo.toml
crates/victory-loader/src/crash.rs
crates/victory-loader/src/thunks.rs
crates/victory-loader/src/tls.rs
crates/victory-shader/src/dxbc.rs
crates/victory-shader/src/emit/bitcode.rs
crates/victory-shader/src/emit/container.rs
crates/victory-shader/src/emit/diff_tests.rs
crates/victory-shader/src/emit/host.rs
crates/victory-shader/src/emit/lower.rs
crates/victory-shader/src/emit/memory_tests.rs
crates/victory-shader/src/emit/mod.rs
crates/victory-shader/src/emit/shader.rs
crates/victory-shader/src/emit/valu_tests.rs
crates/victory-shader/src/emit/wave.rs
crates/victory-shader/src/gfx10/fields.rs
crates/victory-shader/src/gfx10/mod.rs
crates/victory-shader/src/gfx10/opcodes.rs
crates/victory-shader/src/io.rs
crates/victory-shader/src/ir.rs
crates/victory-shader/src/lib.rs
crates/victory-shader/src/native.rs
crates/victory-shader/src/resource.rs
crates/victory-shader/src/structure.rs
docs/architecture.md
README.md
victory.example.toml
```

## DexNom

### Arquivos que so existem neste fork
```
.github/workflows/screen.yml
apps/victory/src/bin/victory-play.rs
apps/victory/src/bin/victory-warm.rs
apps/victory/src/play_launcher.rs
BENCHMARK.md
crates/victory-game/examples/shaderpack_states.rs
crates/victory-game/src/sgd5/database.rs
crates/victory-gpu/src/debug_hud.rs
crates/victory-gpu/src/inplace.rs
crates/victory-gpu/src/material_demand.rs
crates/victory-gpu/src/memory/producer_tests.rs
crates/victory-gpu/src/memory/ray_probe.rs
crates/victory-gpu/src/texback.rs
crates/victory-gpu/src/writeback.rs
crates/victory-hle/src/modules/gameinput/dualsense.rs
crates/victory-hle/src/modules/gameinput/movement.rs
crates/victory-loader/src/console_insn.rs
crates/victory-loader/src/console_insn_decode.rs
crates/victory-shader/src/bvh.rs
crates/victory-shader/src/emit/bitcode/census.rs
crates/victory-shader/src/emit/bitcode/opt.rs
crates/victory-shader/src/emit/cube_lod_tests.rs
crates/victory-shader/src/emit/guest_address.rs
crates/victory-shader/src/emit/helper_lane_tests.rs
crates/victory-shader/src/emit/intersect.rs
crates/victory-shader/src/emit/intersect_tests.rs
crates/victory-shader/src/emit/own_bits.rs
crates/victory-shader/src/emit/packed_mask.rs
crates/victory-shader/src/emit/threading.rs
crates/victory-shader/src/emit/typed_views.rs
crates/victory-shader/src/emit/wave64_tests.rs
crates/victory-shader/src/spill.rs
docs/fork-ray-tracing.md
Initialize-FormatTables.ps1
README.upstream.md
Setup.cmd
Setup-DexNomRuntime.ps1
```

### Arquivos da base ausentes neste fork
```
apps/victory/src/prebuild.rs
crates/victory-formats/src/bcpack-tables.bin
crates/victory-gpu/src/bindless.rs
crates/victory-gpu/src/external.rs
crates/victory-gpu/src/twin.rs
crates/victory-hle/src/modules/fidelityfx.rs
crates/victory-hle/tests/fidelityfx.rs
crates/victory-shader/build.rs
crates/victory-shader/src/emit/bvh_tests.rs
crates/victory-shader/src/emit/flow_tests.rs
crates/victory-shader/src/emit/lower/bvh.rs
crates/victory-shader/src/emit/memory_tests.rs
crates/victory-shader/src/emit/semantics_tests.rs
crates/victory-shader/src/emit/stride_tests.rs
crates/victory-shader/src/emit/synthetic_tests.rs
crates/victory-shader/src/range.rs
```

### Arquivos com conteudo diferente
```
.github/workflows/ci.yml
.gitignore
apps/victory/Cargo.toml
apps/victory/src/main.rs
apps/victory/src/run.rs
apps/victory/src/shader.rs
apps/victory/src/translate.rs
Cargo.lock
Cargo.toml
crates/victory-core/src/config.rs
crates/victory-formats/src/bcpack.rs
crates/victory-formats/src/texture.rs
crates/victory-game/Cargo.toml
crates/victory-game/src/lib.rs
crates/victory-game/src/pipelines.rs
crates/victory-game/src/sgd5.rs
crates/victory-game/src/shaderpack.rs
crates/victory-gpu/Cargo.toml
crates/victory-gpu/src/capture.rs
crates/victory-gpu/src/console.rs
crates/victory-gpu/src/copystat.rs
crates/victory-gpu/src/descriptor.rs
crates/victory-gpu/src/descriptor/host.rs
crates/victory-gpu/src/device.rs
crates/victory-gpu/src/draw.rs
crates/victory-gpu/src/dxgi.rs
crates/victory-gpu/src/fence.rs
crates/victory-gpu/src/frame.rs
crates/victory-gpu/src/host.rs
crates/victory-gpu/src/indirect.rs
crates/victory-gpu/src/lib.rs
crates/victory-gpu/src/list.rs
crates/victory-gpu/src/memory.rs
crates/victory-gpu/src/object.rs
crates/victory-gpu/src/pipeline.rs
crates/victory-gpu/src/pipeline/host.rs
crates/victory-gpu/src/pipeline/library.rs
crates/victory-gpu/src/pipeline/make.rs
crates/victory-gpu/src/primitive.rs
crates/victory-gpu/src/queue.rs
crates/victory-gpu/src/resource.rs
crates/victory-gpu/src/root.rs
crates/victory-gpu/src/shader_cache.rs
crates/victory-gpu/src/shadow.rs
crates/victory-gpu/src/texture.rs
crates/victory-gpu/src/watch.rs
crates/victory-hle/Cargo.toml
crates/victory-hle/src/lib.rs
crates/victory-hle/src/modules/audio.rs
crates/victory-hle/src/modules/d3d12x.rs
crates/victory-hle/src/modules/display.rs
crates/victory-hle/src/modules/dstorage.rs
crates/victory-hle/src/modules/files.rs
crates/victory-hle/src/modules/gameinput.rs
crates/victory-hle/src/modules/kernel.rs
crates/victory-hle/src/modules/lifecycle.rs
crates/victory-hle/src/modules/mod.rs
crates/victory-hle/src/modules/storage.rs
crates/victory-hle/src/modules/xg.rs
crates/victory-hle/src/modules/xgame.rs
crates/victory-hle/src/modules/xgameruntime.rs
crates/victory-hle/src/modules/xgamesave.rs
crates/victory-hle/src/modules/xgameui.rs
crates/victory-hle/src/modules/xmem.rs
crates/victory-hle/src/modules/xnetworking.rs
crates/victory-hle/src/modules/xpackage.rs
crates/victory-hle/src/modules/xstore.rs
crates/victory-hle/src/modules/xsystem.rs
crates/victory-hle/src/modules/xuser.rs
crates/victory-hle/src/runtime.rs
crates/victory-loader/Cargo.toml
crates/victory-loader/src/crash.rs
crates/victory-loader/src/lib.rs
crates/victory-loader/src/stacks.rs
crates/victory-loader/src/threads.rs
crates/victory-loader/src/tls.rs
crates/victory-shader/Cargo.toml
crates/victory-shader/src/dxbc.rs
crates/victory-shader/src/emit/bitcode.rs
crates/victory-shader/src/emit/container.rs
crates/victory-shader/src/emit/diff_tests.rs
crates/victory-shader/src/emit/host.rs
crates/victory-shader/src/emit/lower.rs
crates/victory-shader/src/emit/mod.rs
crates/victory-shader/src/emit/rect.rs
crates/victory-shader/src/emit/shader.rs
crates/victory-shader/src/emit/valu_tests.rs
crates/victory-shader/src/emit/waterfall_tests.rs
crates/victory-shader/src/emit/wave.rs
crates/victory-shader/src/gfx10/fields.rs
crates/victory-shader/src/io.rs
crates/victory-shader/src/ir.rs
crates/victory-shader/src/lib.rs
crates/victory-shader/src/native.rs
crates/victory-shader/src/resource.rs
crates/victory-shader/src/structure.rs
docs/architecture.md
docs/asset-policy.md
README.md
victory.example.toml
xtask/src/policy.rs
```

## VItoriaBr

### Arquivos que so existem neste fork
```
apps/victory/src/warm.rs
crates/victory-formats/src/texture/generate.rs
crates/victory-formats/src/texture/patterns.rs
crates/victory-gpu/src/ambient.rs
crates/victory-gpu/src/cardmeter.rs
crates/victory-gpu/src/drvwatch.rs
crates/victory-gpu/src/fpsbox.rs
crates/victory-gpu/src/framelog.rs
crates/victory-gpu/src/indirect/bench.rs
crates/victory-gpu/src/inplace.rs
crates/victory-gpu/src/plaincopy.rs
crates/victory-gpu/src/profile.rs
crates/victory-gpu/src/profile/tests.rs
crates/victory-gpu/src/queue2.rs
crates/victory-gpu/src/ranges.rs
crates/victory-gpu/src/renderdoc.rs
crates/victory-gpu/src/shot.rs
crates/victory-gpu/src/spread.rs
crates/victory-gpu/src/texback.rs
crates/victory-gpu/src/tiles.rs
crates/victory-loader/src/console_insn.rs
crates/victory-loader/src/console_insn_decode.rs
crates/victory-shader/src/bvh.rs
crates/victory-shader/src/emit/bitcode/census.rs
crates/victory-shader/src/emit/bitcode/opt.rs
crates/victory-shader/src/emit/dot.rs
crates/victory-shader/src/emit/image.rs
crates/victory-shader/src/emit/intersect.rs
crates/victory-shader/src/emit/intersect_tests.rs
crates/victory-shader/src/emit/own_bits.rs
crates/victory-shader/src/emit/test_skip.rs
crates/victory-shader/src/emit/threading.rs
crates/victory-shader/src/emit/wave64_tests.rs
crates/victory-shader/src/spill.rs
docs/comparative-performance-hypotheses.md
docs/fork-ray-tracing.md
docs/performance-architecture.md
docs/precompilacao-shaders.md
docs/visao-geral.md
xtask/src/ci.rs
```

### Arquivos da base ausentes neste fork
```
apps/victory/src/prebuild.rs
crates/victory-game/src/cityhash.rs
crates/victory-game/src/pipelines.rs
crates/victory-game/src/shaderpack.rs
crates/victory-gpu/src/bindless.rs
crates/victory-gpu/src/external.rs
crates/victory-gpu/src/twin.rs
crates/victory-hle/src/modules/fidelityfx.rs
crates/victory-hle/tests/fidelityfx.rs
crates/victory-shader/src/emit/bvh_tests.rs
crates/victory-shader/src/emit/flow_tests.rs
crates/victory-shader/src/emit/lower/bvh.rs
crates/victory-shader/src/emit/memory_tests.rs
crates/victory-shader/src/emit/semantics_tests.rs
crates/victory-shader/src/emit/stride_tests.rs
crates/victory-shader/src/emit/synthetic_tests.rs
crates/victory-shader/src/range.rs
```

### Arquivos com conteudo diferente
```
.github/workflows/ci.yml
.victory-root
apps/victory/Cargo.toml
apps/victory/src/main.rs
apps/victory/src/run.rs
apps/victory/src/shader.rs
apps/victory/src/translate.rs
Cargo.lock
Cargo.toml
crates/victory-core/src/config.rs
crates/victory-core/src/policy.rs
crates/victory-formats/src/bcpack.rs
crates/victory-formats/src/texture.rs
crates/victory-formats/src/xma.rs
crates/victory-game/Cargo.toml
crates/victory-game/src/lib.rs
crates/victory-game/src/sgd5.rs
crates/victory-gpu/Cargo.toml
crates/victory-gpu/src/capture.rs
crates/victory-gpu/src/console.rs
crates/victory-gpu/src/copystat.rs
crates/victory-gpu/src/descriptor.rs
crates/victory-gpu/src/descriptor/host.rs
crates/victory-gpu/src/device.rs
crates/victory-gpu/src/draw.rs
crates/victory-gpu/src/dxgi.rs
crates/victory-gpu/src/fence.rs
crates/victory-gpu/src/frame.rs
crates/victory-gpu/src/host.rs
crates/victory-gpu/src/indirect.rs
crates/victory-gpu/src/lib.rs
crates/victory-gpu/src/list.rs
crates/victory-gpu/src/memory.rs
crates/victory-gpu/src/pipeline.rs
crates/victory-gpu/src/pipeline/host.rs
crates/victory-gpu/src/pipeline/library.rs
crates/victory-gpu/src/pipeline/make.rs
crates/victory-gpu/src/primitive.rs
crates/victory-gpu/src/queue.rs
crates/victory-gpu/src/resource.rs
crates/victory-gpu/src/root.rs
crates/victory-gpu/src/shader_cache.rs
crates/victory-gpu/src/shadow.rs
crates/victory-gpu/src/texture.rs
crates/victory-gpu/src/watch.rs
crates/victory-hle/Cargo.toml
crates/victory-hle/src/host.rs
crates/victory-hle/src/modules/audio.rs
crates/victory-hle/src/modules/d3d12x.rs
crates/victory-hle/src/modules/display.rs
crates/victory-hle/src/modules/dstorage.rs
crates/victory-hle/src/modules/files.rs
crates/victory-hle/src/modules/gameinput.rs
crates/victory-hle/src/modules/lifecycle.rs
crates/victory-hle/src/modules/mod.rs
crates/victory-hle/src/modules/storage.rs
crates/victory-hle/src/modules/xg.rs
crates/victory-hle/src/modules/xgame.rs
crates/victory-hle/src/modules/xgameruntime.rs
crates/victory-hle/src/modules/xgamesave.rs
crates/victory-hle/src/modules/xgameui.rs
crates/victory-hle/src/modules/xmem.rs
crates/victory-hle/src/modules/xnetworking.rs
crates/victory-hle/src/modules/xpackage.rs
crates/victory-hle/src/modules/xstore.rs
crates/victory-hle/src/modules/xsystem.rs
crates/victory-hle/src/modules/xuser.rs
crates/victory-hle/src/runtime.rs
crates/victory-loader/Cargo.toml
crates/victory-loader/src/crash.rs
crates/victory-loader/src/lib.rs
crates/victory-loader/src/stacks.rs
crates/victory-loader/src/thunks.rs
crates/victory-loader/src/tls.rs
crates/victory-pe/src/image.rs
crates/victory-shader/build.rs
crates/victory-shader/Cargo.toml
crates/victory-shader/src/dxbc.rs
crates/victory-shader/src/emit/bitcode.rs
crates/victory-shader/src/emit/container.rs
crates/victory-shader/src/emit/diff_tests.rs
crates/victory-shader/src/emit/host.rs
crates/victory-shader/src/emit/lower.rs
crates/victory-shader/src/emit/mod.rs
crates/victory-shader/src/emit/rect.rs
crates/victory-shader/src/emit/shader.rs
crates/victory-shader/src/emit/valu_tests.rs
crates/victory-shader/src/emit/waterfall_tests.rs
crates/victory-shader/src/emit/wave.rs
crates/victory-shader/src/gfx10/fields.rs
crates/victory-shader/src/gfx10/opcodes.rs
crates/victory-shader/src/io.rs
crates/victory-shader/src/ir.rs
crates/victory-shader/src/lib.rs
crates/victory-shader/src/native.rs
crates/victory-shader/src/resource.rs
crates/victory-shader/src/structure.rs
docs/architecture.md
README.md
victory.example.toml
xtask/Cargo.toml
xtask/src/main.rs
```

## AvatarManz

### Arquivos que so existem neste fork
```
apps/victory/src/replay.rs
apps/victory/src/warm.rs
apps/victory/tests/census.rs
crates/victory-formats/examples/texture_layout_bench.rs
crates/victory-gpu/build.rs
crates/victory-gpu/src/ab.rs
crates/victory-gpu/src/ambient.rs
crates/victory-gpu/src/book.rs
crates/victory-gpu/src/cores.rs
crates/victory-gpu/src/faultbook.rs
crates/victory-gpu/src/host/aside.rs
crates/victory-gpu/src/host/awake.rs
crates/victory-gpu/src/host/card_steps.rs
crates/victory-gpu/src/host/indirects.rs
crates/victory-gpu/src/host/panel.rs
crates/victory-gpu/src/host/place.rs
crates/victory-gpu/src/host/send.rs
crates/victory-gpu/src/host/spans.rs
crates/victory-gpu/src/host/stamps.rs
crates/victory-gpu/src/host/torn.rs
crates/victory-gpu/src/host/traffic.rs
crates/victory-gpu/src/host/upcheck.rs
crates/victory-gpu/src/indirect/batch.rs
crates/victory-gpu/src/indirect/bench.rs
crates/victory-gpu/src/inplace.rs
crates/victory-gpu/src/land.rs
crates/victory-gpu/src/land/tests.rs
crates/victory-gpu/src/list/thin.rs
crates/victory-gpu/src/picked.rs
crates/victory-gpu/src/pipeline/known.rs
crates/victory-gpu/src/pool.rs
crates/victory-gpu/src/popwatch.rs
crates/victory-gpu/src/queue/behind.rs
crates/victory-gpu/src/replay/edit.rs
crates/victory-gpu/src/replay/gpu.rs
crates/victory-gpu/src/replay/hook.rs
crates/victory-gpu/src/replay/mod.rs
crates/victory-gpu/src/replay/play.rs
crates/victory-gpu/src/replay/play/diverge.rs
crates/victory-gpu/src/replay/play/look.rs
crates/victory-gpu/src/replay/play/merge.rs
crates/victory-gpu/src/replay/play/pixel.rs
crates/victory-gpu/src/replay/play/upcopies.rs
crates/victory-gpu/src/replay/pso.rs
crates/victory-gpu/src/replay/record.rs
crates/victory-gpu/src/replay/snap.rs
crates/victory-gpu/src/replay/states.rs
crates/victory-gpu/src/replay/wire.rs
crates/victory-gpu/src/safecopy.rs
crates/victory-gpu/src/switch.rs
```

### Arquivos da base ausentes neste fork
```
apps/victory/src/prebuild.rs
crates/victory-game/src/cityhash.rs
crates/victory-game/src/pipelines.rs
crates/victory-game/src/shaderpack.rs
crates/victory-gpu/src/bindless.rs
crates/victory-gpu/src/external.rs
crates/victory-gpu/src/twin.rs
crates/victory-hle/src/modules/fidelityfx.rs
crates/victory-hle/tests/fidelityfx.rs
crates/victory-shader/src/emit/bvh_tests.rs
crates/victory-shader/src/emit/flow_tests.rs
crates/victory-shader/src/emit/lower/bvh.rs
crates/victory-shader/src/emit/memory_tests.rs
crates/victory-shader/src/emit/semantics_tests.rs
crates/victory-shader/src/emit/stride_tests.rs
crates/victory-shader/src/emit/synthetic_tests.rs
crates/victory-shader/src/range.rs
```

### Arquivos com conteudo diferente
```
apps/victory/Cargo.toml
apps/victory/src/census.rs
apps/victory/src/inspect.rs
apps/victory/src/main.rs
apps/victory/src/run.rs
apps/victory/src/shader.rs
apps/victory/src/translate.rs
Cargo.lock
Cargo.toml
crates/victory-core/src/config.rs
crates/victory-core/src/policy.rs
crates/victory-formats/src/bcpack.rs
crates/victory-formats/src/texture.rs
crates/victory-formats/src/xma.rs
crates/victory-game/Cargo.toml
crates/victory-game/src/lib.rs
crates/victory-game/src/sgd5.rs
crates/victory-gpu/Cargo.toml
crates/victory-gpu/src/capture.rs
crates/victory-gpu/src/console.rs
crates/victory-gpu/src/copystat.rs
crates/victory-gpu/src/descriptor.rs
crates/victory-gpu/src/descriptor/host.rs
crates/victory-gpu/src/device.rs
crates/victory-gpu/src/draw.rs
crates/victory-gpu/src/dxgi.rs
crates/victory-gpu/src/fence.rs
crates/victory-gpu/src/frame.rs
crates/victory-gpu/src/host.rs
crates/victory-gpu/src/indirect.rs
crates/victory-gpu/src/lib.rs
crates/victory-gpu/src/list.rs
crates/victory-gpu/src/memory.rs
crates/victory-gpu/src/object.rs
crates/victory-gpu/src/pipeline.rs
crates/victory-gpu/src/pipeline/host.rs
crates/victory-gpu/src/pipeline/library.rs
crates/victory-gpu/src/pipeline/make.rs
crates/victory-gpu/src/primitive.rs
crates/victory-gpu/src/query.rs
crates/victory-gpu/src/queue.rs
crates/victory-gpu/src/resource.rs
crates/victory-gpu/src/root.rs
crates/victory-gpu/src/sampler.rs
crates/victory-gpu/src/shader_cache.rs
crates/victory-gpu/src/shadow.rs
crates/victory-gpu/src/signature.rs
crates/victory-gpu/src/texture.rs
crates/victory-gpu/src/watch.rs
crates/victory-hle/Cargo.toml
crates/victory-hle/src/host.rs
crates/victory-hle/src/lib.rs
crates/victory-hle/src/modules/apu.rs
crates/victory-hle/src/modules/audio.rs
crates/victory-hle/src/modules/d3d12x.rs
crates/victory-hle/src/modules/display.rs
crates/victory-hle/src/modules/dstorage.rs
crates/victory-hle/src/modules/files.rs
crates/victory-hle/src/modules/gameinput.rs
crates/victory-hle/src/modules/kernel.rs
crates/victory-hle/src/modules/lifecycle.rs
crates/victory-hle/src/modules/media.rs
crates/victory-hle/src/modules/mod.rs
crates/victory-hle/src/modules/net.rs
crates/victory-hle/src/modules/storage.rs
crates/victory-hle/src/modules/xasync.rs
crates/victory-hle/src/modules/xg.rs
crates/victory-hle/src/modules/xgame.rs
crates/victory-hle/src/modules/xgameruntime.rs
crates/victory-hle/src/modules/xgamesave.rs
crates/victory-hle/src/modules/xgameui.rs
crates/victory-hle/src/modules/xmem.rs
crates/victory-hle/src/modules/xnetworking.rs
crates/victory-hle/src/modules/xpackage.rs
crates/victory-hle/src/modules/xstore.rs
crates/victory-hle/src/modules/xsystem.rs
crates/victory-hle/src/modules/xuser.rs
crates/victory-hle/src/runtime.rs
crates/victory-loader/Cargo.toml
crates/victory-loader/src/crash.rs
crates/victory-loader/src/lib.rs
crates/victory-loader/src/stacks.rs
crates/victory-loader/src/tls.rs
crates/victory-loader/src/windows.rs
crates/victory-pe/src/image.rs
crates/victory-pe/src/implib.rs
crates/victory-pe/src/layout.rs
crates/victory-pipeline/src/cache.rs
crates/victory-shader/build.rs
crates/victory-shader/Cargo.toml
crates/victory-shader/src/dxbc.rs
crates/victory-shader/src/emit/bitcode.rs
crates/victory-shader/src/emit/container.rs
crates/victory-shader/src/emit/diff_tests.rs
crates/victory-shader/src/emit/host.rs
crates/victory-shader/src/emit/lower.rs
crates/victory-shader/src/emit/mod.rs
crates/victory-shader/src/emit/rect.rs
crates/victory-shader/src/emit/sampler.rs
crates/victory-shader/src/emit/shader.rs
crates/victory-shader/src/emit/valu_tests.rs
crates/victory-shader/src/emit/waterfall_tests.rs
crates/victory-shader/src/emit/wave.rs
crates/victory-shader/src/gfx10/fields.rs
crates/victory-shader/src/gfx10/mod.rs
crates/victory-shader/src/gfx10/sp3.rs
crates/victory-shader/src/io.rs
crates/victory-shader/src/ir.rs
crates/victory-shader/src/lib.rs
crates/victory-shader/src/native.rs
crates/victory-shader/src/resource.rs
crates/victory-shader/src/structure.rs
docs/architecture.md
docs/asset-policy.md
README.md
victory.example.toml
xtask/src/policy.rs
```

## victory-fork

### Arquivos que so existem neste fork
```
crates/victory-gpu/src/backreads.rs
crates/victory-gpu/src/cardmeter.rs
crates/victory-gpu/src/drvwatch.rs
crates/victory-gpu/src/fpsbox.rs
crates/victory-gpu/src/framelog.rs
crates/victory-gpu/src/indirect/bench.rs
crates/victory-gpu/src/inplace.rs
crates/victory-gpu/src/plaincopy.rs
crates/victory-gpu/src/profile.rs
crates/victory-gpu/src/profile/tests.rs
crates/victory-gpu/src/queue2.rs
crates/victory-gpu/src/renderdoc.rs
crates/victory-gpu/src/shot.rs
crates/victory-gpu/src/spread.rs
crates/victory-gpu/src/texback.rs
crates/victory-gpu/src/tiles.rs
crates/victory-loader/src/console_insn.rs
crates/victory-loader/src/console_insn_decode.rs
crates/victory-shader/src/bvh.rs
crates/victory-shader/src/emit/bitcode/census.rs
crates/victory-shader/src/emit/bitcode/opt.rs
crates/victory-shader/src/emit/intersect.rs
crates/victory-shader/src/emit/intersect_tests.rs
crates/victory-shader/src/emit/own_bits.rs
crates/victory-shader/src/emit/threading.rs
crates/victory-shader/src/emit/wave64_tests.rs
crates/victory-shader/src/spill.rs
diagnostics/README.md
diagnostics/timing-run-2026-11-25.txt
docs/fork-ray-tracing.md
```

### Arquivos da base ausentes neste fork
```
apps/victory/src/prebuild.rs
crates/victory-game/src/cityhash.rs
crates/victory-game/src/pipelines.rs
crates/victory-game/src/shaderpack.rs
crates/victory-gpu/src/bindless.rs
crates/victory-gpu/src/external.rs
crates/victory-gpu/src/twin.rs
crates/victory-hle/src/modules/fidelityfx.rs
crates/victory-hle/tests/fidelityfx.rs
crates/victory-shader/src/emit/bvh_tests.rs
crates/victory-shader/src/emit/flow_tests.rs
crates/victory-shader/src/emit/lower/bvh.rs
crates/victory-shader/src/emit/memory_tests.rs
crates/victory-shader/src/emit/semantics_tests.rs
crates/victory-shader/src/emit/stride_tests.rs
crates/victory-shader/src/emit/synthetic_tests.rs
crates/victory-shader/src/range.rs
```

### Arquivos com conteudo diferente
```
apps/victory/Cargo.toml
apps/victory/src/main.rs
apps/victory/src/run.rs
apps/victory/src/shader.rs
apps/victory/src/translate.rs
Cargo.lock
Cargo.toml
crates/victory-core/src/config.rs
crates/victory-formats/src/bcpack.rs
crates/victory-formats/src/texture.rs
crates/victory-game/Cargo.toml
crates/victory-game/src/lib.rs
crates/victory-game/src/sgd5.rs
crates/victory-gpu/Cargo.toml
crates/victory-gpu/src/capture.rs
crates/victory-gpu/src/console.rs
crates/victory-gpu/src/copystat.rs
crates/victory-gpu/src/descriptor.rs
crates/victory-gpu/src/descriptor/host.rs
crates/victory-gpu/src/device.rs
crates/victory-gpu/src/draw.rs
crates/victory-gpu/src/dxgi.rs
crates/victory-gpu/src/fence.rs
crates/victory-gpu/src/frame.rs
crates/victory-gpu/src/host.rs
crates/victory-gpu/src/indirect.rs
crates/victory-gpu/src/lib.rs
crates/victory-gpu/src/list.rs
crates/victory-gpu/src/memory.rs
crates/victory-gpu/src/pipeline.rs
crates/victory-gpu/src/pipeline/host.rs
crates/victory-gpu/src/pipeline/library.rs
crates/victory-gpu/src/pipeline/make.rs
crates/victory-gpu/src/primitive.rs
crates/victory-gpu/src/queue.rs
crates/victory-gpu/src/resource.rs
crates/victory-gpu/src/root.rs
crates/victory-gpu/src/shader_cache.rs
crates/victory-gpu/src/shadow.rs
crates/victory-gpu/src/texture.rs
crates/victory-gpu/src/watch.rs
crates/victory-hle/Cargo.toml
crates/victory-hle/src/modules/audio.rs
crates/victory-hle/src/modules/d3d12x.rs
crates/victory-hle/src/modules/display.rs
crates/victory-hle/src/modules/dstorage.rs
crates/victory-hle/src/modules/files.rs
crates/victory-hle/src/modules/gameinput.rs
crates/victory-hle/src/modules/lifecycle.rs
crates/victory-hle/src/modules/mod.rs
crates/victory-hle/src/modules/storage.rs
crates/victory-hle/src/modules/xg.rs
crates/victory-hle/src/modules/xgame.rs
crates/victory-hle/src/modules/xgameruntime.rs
crates/victory-hle/src/modules/xgamesave.rs
crates/victory-hle/src/modules/xgameui.rs
crates/victory-hle/src/modules/xmem.rs
crates/victory-hle/src/modules/xnetworking.rs
crates/victory-hle/src/modules/xpackage.rs
crates/victory-hle/src/modules/xstore.rs
crates/victory-hle/src/modules/xsystem.rs
crates/victory-hle/src/modules/xuser.rs
crates/victory-hle/src/runtime.rs
crates/victory-loader/Cargo.toml
crates/victory-loader/src/crash.rs
crates/victory-loader/src/lib.rs
crates/victory-loader/src/tls.rs
crates/victory-shader/build.rs
crates/victory-shader/Cargo.toml
crates/victory-shader/src/dxbc.rs
crates/victory-shader/src/emit/bitcode.rs
crates/victory-shader/src/emit/container.rs
crates/victory-shader/src/emit/diff_tests.rs
crates/victory-shader/src/emit/host.rs
crates/victory-shader/src/emit/lower.rs
crates/victory-shader/src/emit/mod.rs
crates/victory-shader/src/emit/rect.rs
crates/victory-shader/src/emit/shader.rs
crates/victory-shader/src/emit/valu_tests.rs
crates/victory-shader/src/emit/waterfall_tests.rs
crates/victory-shader/src/emit/wave.rs
crates/victory-shader/src/gfx10/fields.rs
crates/victory-shader/src/io.rs
crates/victory-shader/src/ir.rs
crates/victory-shader/src/lib.rs
crates/victory-shader/src/native.rs
crates/victory-shader/src/resource.rs
crates/victory-shader/src/structure.rs
docs/architecture.md
README.md
victory.example.toml
```

## Arquivos que mais divergem entre os forks

Contagem = em quantos dos 5 forks o blob difere da base.
```
 5/5  crates/victory-shader/src/emit/wave.rs
 5/5  crates/victory-shader/src/emit/mod.rs
 5/5  crates/victory-gpu/src/fence.rs
 5/5  crates/victory-gpu/src/queue.rs
 5/5  crates/victory-loader/Cargo.toml
 5/5  README.md
 5/5  crates/victory-hle/src/modules/xgamesave.rs
 5/5  crates/victory-gpu/src/device.rs
 5/5  crates/victory-loader/src/crash.rs
 5/5  crates/victory-shader/src/dxbc.rs
 5/5  crates/victory-hle/src/modules/xuser.rs
 5/5  crates/victory-loader/src/tls.rs
 5/5  crates/victory-shader/src/emit/shader.rs
 5/5  apps/victory/src/run.rs
 5/5  victory.example.toml
 5/5  Cargo.lock
 5/5  crates/victory-gpu/src/indirect.rs
 5/5  apps/victory/src/main.rs
 5/5  crates/victory-hle/src/modules/dstorage.rs
 5/5  crates/victory-shader/src/lib.rs
 5/5  crates/victory-shader/src/emit/host.rs
 5/5  crates/victory-gpu/src/descriptor/host.rs
 5/5  crates/victory-gpu/src/shadow.rs
 5/5  crates/victory-gpu/src/pipeline.rs
 5/5  crates/victory-gpu/src/primitive.rs
 5/5  crates/victory-gpu/src/list.rs
 5/5  crates/victory-shader/src/ir.rs
 5/5  crates/victory-gpu/src/host.rs
 5/5  crates/victory-game/src/sgd5.rs
 5/5  crates/victory-core/src/config.rs
 5/5  crates/victory-gpu/src/copystat.rs
 5/5  crates/victory-gpu/src/texture.rs
 5/5  crates/victory-shader/src/emit/lower.rs
 5/5  crates/victory-shader/src/emit/bitcode.rs
 5/5  apps/victory/Cargo.toml
 5/5  crates/victory-hle/src/modules/gameinput.rs
 5/5  crates/victory-gpu/src/pipeline/library.rs
 5/5  crates/victory-gpu/src/watch.rs
 5/5  crates/victory-hle/src/modules/mod.rs
 5/5  crates/victory-formats/src/texture.rs
```
