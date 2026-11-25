# FATOS VERIFICADOS - victory-fork

Gerado por git puro, sem LLM.

## A. Identificacao
- Nome: victory-fork
- URL: https://github.com/fxceless/victory-fork
- Branch analisada: main
- Raiz (commit 0): f4504f6413ce4509604dad66cdd689000a0204dc
- Primeiro commit: f4504f6413ce4509604dad66cdd689000a0204dc | 2026-11-23 01:15:26 +0100 | Marvis | Initial commit
- Ultimo commit: 6d20eed6ae8afe37c52f1911b46537b0ea8bcd0d | 2026-11-25 18:02:01 -0500 | fxceless | The 64-thread hash of pull request #1014 made opt-in (VICTORY_CARD_HASH_WIDE=1); changed card copies from a list (VICTORY_CARD_CHANGED_LIST); notes about unmapped blocks kept until told (VICTORY_ABSENT_MS=told)
- Periodo: 2026-11-23 a 2026-11-25 (3 dias com commit)
- Commits em HEAD: 253
- Commits em todas as refs: 253
- Commits so em outras branches: 0
- Arquivos versionados: 187
- Merges: 93

## Autores
```
    87  Marvis <60314886+Marvisak@users.noreply.github.com>
    34  fxceless <36782933+fxceless@users.noreply.github.com>
    15  Marvis <marvis@noreply.invalid>
    11  Duff <19514584+Dufftwt@users.noreply.github.com>
     6  r4. <192970269+r4tedx@users.noreply.github.com>
     5  9hvn9yt2vk-netizen <360448330+9hvn9yt2vk-netizen@users.noreply.github.com>
     5  Manuel P├®rez <manuelperezardanuy@gmail.com>
     4  FIocker <4461558+FIocker@users.noreply.github.com>
     4  fuzeofficial <bochunran@gmail.com>
     4  marquezini <felipemarquesini10@gmail.com>
     4  Claude <noreply@anthropic.com>
     4  tedin7 <tommasoderosa94@gmail.com>
     4  TG0000 <122870448+TG0000@users.noreply.github.com>
     4  ifBars <tristens5465@gmail.com>
     4  Maraco91 <43787609+Maraco91@users.noreply.github.com>
```

## Commits por dia
```
2026-11-23     48
2026-11-24    182
2026-11-25     23
```

## 30 arquivos mais alterados
```
   35  crates/victory-shader/src/emit/lower.rs
   32  README.md
   29  crates/victory-gpu/src/host.rs
   23  crates/victory-gpu/src/draw.rs
   19  crates/victory-shader/src/emit/mod.rs
   19  crates/victory-gpu/src/descriptor/host.rs
   19  crates/victory-gpu/src/shadow.rs
   16  crates/victory-gpu/src/list.rs
   14  crates/victory-shader/src/emit/valu_tests.rs
   14  crates/victory-gpu/src/lib.rs
   13  crates/victory-gpu/src/watch.rs
   13  crates/victory-gpu/src/texture.rs
   13  crates/victory-hle/src/modules/xmem.rs
   12  crates/victory-gpu/src/resource.rs
   11  crates/victory-loader/src/crash.rs
   11  crates/victory-shader/src/resource.rs
   10  apps/victory/src/translate.rs
   10  crates/victory-hle/src/modules/dstorage.rs
   10  crates/victory-gpu/src/indirect.rs
   10  crates/victory-gpu/src/capture.rs
    9  crates/victory-gpu/src/descriptor.rs
    9  crates/victory-gpu/src/pipeline.rs
    9  crates/victory-gpu/src/queue.rs
    8  crates/victory-gpu/src/copystat.rs
    8  crates/victory-shader/src/ir.rs
    8  crates/victory-gpu/src/shader_cache.rs
    7  docs/architecture.md
    7  crates/victory-shader/src/emit/shader.rs
    7  crates/victory-loader/src/lib.rs
    7  crates/victory-formats/src/texture.rs
```

## Merges (primeiros 80)
```
1391409|2026-11-25|Merge pull request #1014 (martindevbr): the card compare hashes a piece with a group of 64 threads
f5c03f3|2026-11-25|Merge pull request #999 (cxy0072019-sketch): textures shaders write that the title reads on the CPU are copied back into title memory (VICTORY_TEXTURE_BACK=1; off by default)
ca48fbe|2026-11-24|Merge pull request #566 from r4tedx/shader/per-lane-phi
e09dbb6|2026-11-24|Merge upstream main (8e79c80) into shader/per-lane-phi
c7a5f49|2026-11-24|Merge pull request #93 from TG0000/perf/bcpack-bucket-lookup
2005e35|2026-11-24|Merge pull request #88 from AndyLatimer/fix/title-size-overflows
58fed37|2026-11-24|Merge pull request #706 from Lyrnc/fix/cmp-lg-ordered
a51bdfa|2026-11-24|Merge pull request #696 from catminou4/devin/formats-doc-micro-link
02f404e|2026-11-24|Merge pull request #685 from catminou4/devin/constant-color-cfg-windows
5cd7421|2026-11-24|Merge pull request #681 from fuzeofficlal/fix/vertex-buffer-null-unbind
8d23f07|2026-11-24|Merge pull request #680 from EatCodeSleep2/fix/wine-staging-upload-128
22700b9|2026-11-24|Merge pull request #675 from lameloball2813-ui/docs/rustdoc-private-links
6db02ea|2026-11-24|Merge pull request #671 from fuzeofficlal/fix/copyresource-cpu-texture-source
16bd577|2026-11-24|Merge pull request #666 from lameloball2813-ui/fix/load-probe-target-dir
b4b4110|2026-11-24|Merge pull request #654 from SantannaCarlos/fix/uav-describe
cf0bb43|2026-11-24|Merge pull request #637 from AlanQiang0418/codex/gfx10-packed-inline-constants
6f0a10a|2026-11-24|Merge pull request #634 from SAitken91/shader/image-sample-o
9a92ccf|2026-11-24|Merge pull request #612 from KoLolX3/fix/phi-liveness-transitive
204bdcc|2026-11-24|Merge pull request #611 from AdvaitPalve11/fix/config-show-data-dir
a4d7b16|2026-11-24|Merge pull request #565 from SAitken91/shader/conservative-depth-position
fd92558|2026-11-24|Merge pull request #520 from Quat3rnion/victory-xasync-payload
dd54de2|2026-11-24|Merge pull request #448 from Dufftwt/fix/min-max-signed-zeros
0ddbce8|2026-11-24|Merge pull request #433 from r4tedx/tests/serialise-valu-gpu-tests
ce42e77|2026-11-24|Merge pull request #432 from Proudnoob4/fix/uninitialized-sgprs
f0894c2|2026-11-24|Merge pull request #429 from marquezinii/fix/xgamesave-documented-errors
9f18eb4|2026-11-24|Merge pull request #423 from pedrofort/shader-perm-no-int64
dd273df|2026-11-24|Merge pull request #419 from itsRevela/fix/swallow-keyboard-menu
3c86238|2026-11-24|Merge pull request #418 from enzoduarte1/gpu/half-subnormal-output
2e4e280|2026-11-24|Merge pull request #384 from LouisRiverstone/shader-cvt16-saturate
faba64c|2026-11-24|Merge pull request #382 from Dufftwt/fix/stencil-ref-shader-flag
c30fc99|2026-11-24|Merge pull request #381 from Dufftwt/fix/psv-sample-frequency
ebf3615|2026-11-24|Merge pull request #376 from LouisRiverstone/shader-cvt-norm-f16
f67c50c|2026-11-24|Merge pull request #372 from marquezinii/fix/capture-unorm16-formats
40b7ed1|2026-11-24|Merge pull request #371 from marquezinii/fix/pls-path-small-buffer
a7e60cf|2026-11-24|Merge upstream main (42361b0) into shader/per-lane-phi
7374bfb|2026-11-24|Merge pull request #365 from FIocker/gpu/publish-initialized-descriptor-heap
42361b0|2026-11-24|Merge pull request #356 from FIocker/gpu/stable-drop-rank
2745589|2026-11-24|Merge pull request #355 from enzoduarte1/clippy/clean-workspace-lints
533b7e2|2026-11-24|Merge pull request #341 from enzoduarte1/config/empty-test-titles-env
edf64e2|2026-11-24|Merge pull request #336 from hophaver/fix/dynamic-descriptor-exhaustion
5a16e66|2026-11-24|Merge pull request #313 from Abdellah-Khanani/loader-aligned-unwind
93c0852|2026-11-24|Merge pull request #285 from Dufftwt/fix/name-title-dll-that-fails-to-prepare
db89843|2026-11-24|Merge pull request #276 from Dufftwt/fix/s-ttracedata-imm-opcode
89e1031|2026-11-24|Merge pull request #260 from Dufftwt/fix/f32-to-half-rounding-carry
88b6658|2026-11-24|Merge pull request #249 from Maraco91/gpu/get-device-newer-levels
56d5032|2026-11-24|Merge pull request #245 from Dufftwt/fix/import-trap-in-crash-log
9e44f38|2026-11-24|Merge pull request #239 from Maraco91/xtaskqueue/terminate-ends-dispatch
7c8cecb|2026-11-24|Merge pull request #229 from Dufftwt/fix/gamertag-modern-suffix-empty
27f9b76|2026-11-24|Merge pull request #209 from Maraco91/xgamesave/empty-configuration-id
6af0fae|2026-11-24|Merge pull request #205 from asdaasdsaddf/fix/null-uav-dimension
27c960b|2026-11-24|Merge pull request #204 from Dufftwt/fix/crash-report-fault-memory
3dd37e6|2026-11-24|Merge pull request #198 from Dufftwt/fix/crash-report-instruction-bytes
13ac0e6|2026-11-24|Merge pull request #167 from Maraco91/xmem/zero-length-unmap
787f665|2026-11-24|Merge pull request #161 from lordon1a/fix/pipeline-staging-cleanup
ba7d873|2026-11-24|Merge pull request #152 from Makto94/fix/net-test-win10
80c0d12|2026-11-24|Merge pull request #140 from lordon1a/feat/cmp-class-f16
3d77d5e|2026-11-24|Merge pull request #117 from tedin7/vic-s-bitcmp-2
b0328ba|2026-11-24|Merge pull request #111 from Dufftwt/fix/narrow-paths-translated-twice
0e990e4|2026-11-24|Merge pull request #104 from TG0000/perf/bitcode-worklist-liveness
c2e7254|2026-11-24|Merge pull request #87 from jeremygold02/codex/copy-stat-frame-window
1f858b0|2026-11-24|Merge pull request #25 from artor410/packed-buffer-formats
d2a4bae|2026-11-24|Merge pull request #129 from Luislariosss/fix/duplicate-test-module
9aef3d9|2026-11-24|Merge pull request #51 from BaggyALPHA/fix/windows-commit-limit-diagnostic
8a8a38d|2026-11-24|Merge pull request #53 from a55uka/fix/wine-physical-pages
4c0bb89|2026-11-24|Merge pull request #41 from LaunchDoesntCode/codex/proton-warp-safety
91e9bc9|2026-11-24|Merge pull request #46 from 9hvn9yt2vk-netizen/shader-loop-limit
6168fc6|2026-11-24|Merge pull request #70 from FIocker/gpu/diagnostic-pipeline-filter
97464ba|2026-11-24|Merge pull request #61 from TG0000/hle/physical-page-runs
93bc978|2026-11-24|Merge pull request #60 from ifBars/perf/reuse-texture-row-spans
fa5ef73|2026-11-24|Merge pull request #79 from TG0000/perf/texture-check-once-per-submission
0f35dfc|2026-11-24|Merge pull request #66 from AndyLatimer/fix/gpu-rect-element-past-stride
fc0c4dd|2026-11-24|Merge pull request #77 from Exorion1er/fix/guard-fault-race
683278b|2026-11-24|Merge pull request #36 from jeremygold02/codex/root-parameter-fixture
eda38aa|2026-11-24|Merge pull request #57 from Yaekai/fix/null-buffer-views
237f2e8|2026-11-24|Merge pull request #37 from alessiodeia/fix/valu-med3-fract-trig
4926f72|2026-11-24|Merge pull request #80 from tedin7/vic-far-jump-backward
46610eb|2026-11-24|Merge pull request #81 from tedin7/vic-scalar-carry
4a8a058|2026-11-24|Merge pull request #58 from Yaekai/feat/shader-xnor-swap
8fe0b74|2026-11-24|Merge pull request #56 from 9hvn9yt2vk-netizen/find-input-underflow
a6888a6|2026-11-24|Merge pull request #43 from 9hvn9yt2vk-netizen/shader-mask-cycle
```

## Branches remotas nao mergeadas
```

```

## Reverts explicitos
```
d510ee4|2026-11-25|README: the dips while moving, measured; two attempts on the title's page mapping, both taken back out
```

## Candidatos a correcao
```
d510ee4|2026-11-25|fxceless|README: the dips while moving, measured; two attempts on the title's page mapping, both taken back out
0b0074e|2026-11-25|fxceless|Copies in place measured and fixed, our threads on the fast cores, and a corrected comparison of draw windows
f85101b|2026-11-25|fxceless|Frame rate, second night: a submit thread of the layer's own, a copy queue for fetches, indirect draw windows made safer, a profiler; all behind switches
a26859f|2026-11-25|fxceless|The first room renders right: byte-copied descriptor records are mirrored, views off their copy's elements get their own bytes, card room for the first world frame, levels on demand run on the card
310c6eb|2026-11-24|fxceless|Packed textures stay in the upload queue; a texture on the card is refreshed when the card is full
31714de|2026-11-24|fxceless|Take the fix/amd-view-heap-clamp branch of triippywhale's fork (as one commit)
90228a7|2026-11-25|cxccxc|Shaders: restore parked lane-mask pairs a few instructions apart; relative VGPR reads keep parked lanes (#11)
3930989|2026-11-24|oddiums|Fix mesh pipeline test on unsupported GPUs (#14)
e9765a7|2026-11-24|fxceless|Fix the device loss introduced by the wave64 update
004bf69|2026-11-24|fxceless|Test fix for hosted runners, stencil diagnostics, and a stability warning
2005e35|2026-11-24|Marvis|Merge pull request #88 from AndyLatimer/fix/title-size-overflows
58fed37|2026-11-24|Marvis|Merge pull request #706 from Lyrnc/fix/cmp-lg-ordered
5cd7421|2026-11-24|Marvis|Merge pull request #681 from fuzeofficlal/fix/vertex-buffer-null-unbind
8d23f07|2026-11-24|Marvis|Merge pull request #680 from EatCodeSleep2/fix/wine-staging-upload-128
6db02ea|2026-11-24|Marvis|Merge pull request #671 from fuzeofficlal/fix/copyresource-cpu-texture-source
16bd577|2026-11-24|Marvis|Merge pull request #666 from lameloball2813-ui/fix/load-probe-target-dir
b4b4110|2026-11-24|Marvis|Merge pull request #654 from SantannaCarlos/fix/uav-describe
9a92ccf|2026-11-24|Marvis|Merge pull request #612 from KoLolX3/fix/phi-liveness-transitive
204bdcc|2026-11-24|Marvis|Merge pull request #611 from AdvaitPalve11/fix/config-show-data-dir
dd54de2|2026-11-24|Marvis|Merge pull request #448 from Dufftwt/fix/min-max-signed-zeros
ce42e77|2026-11-24|Marvis|Merge pull request #432 from Proudnoob4/fix/uninitialized-sgprs
f0894c2|2026-11-24|Marvis|Merge pull request #429 from marquezinii/fix/xgamesave-documented-errors
dd273df|2026-11-24|Marvis|Merge pull request #419 from itsRevela/fix/swallow-keyboard-menu
faba64c|2026-11-24|Marvis|Merge pull request #382 from Dufftwt/fix/stencil-ref-shader-flag
c30fc99|2026-11-24|Marvis|Merge pull request #381 from Dufftwt/fix/psv-sample-frequency
f67c50c|2026-11-24|Marvis|Merge pull request #372 from marquezinii/fix/capture-unorm16-formats
40b7ed1|2026-11-24|Marvis|Merge pull request #371 from marquezinii/fix/pls-path-small-buffer
edf64e2|2026-11-24|Marvis|Merge pull request #336 from hophaver/fix/dynamic-descriptor-exhaustion
93c0852|2026-11-24|Marvis|Merge pull request #285 from Dufftwt/fix/name-title-dll-that-fails-to-prepare
db89843|2026-11-24|Marvis|Merge pull request #276 from Dufftwt/fix/s-ttracedata-imm-opcode
89e1031|2026-11-24|Marvis|Merge pull request #260 from Dufftwt/fix/f32-to-half-rounding-carry
56d5032|2026-11-24|Marvis|Merge pull request #245 from Dufftwt/fix/import-trap-in-crash-log
7c8cecb|2026-11-24|Marvis|Merge pull request #229 from Dufftwt/fix/gamertag-modern-suffix-empty
6af0fae|2026-11-24|Marvis|Merge pull request #205 from asdaasdsaddf/fix/null-uav-dimension
27c960b|2026-11-24|Marvis|Merge pull request #204 from Dufftwt/fix/crash-report-fault-memory
3dd37e6|2026-11-24|Marvis|Merge pull request #198 from Dufftwt/fix/crash-report-instruction-bytes
787f665|2026-11-24|Marvis|Merge pull request #161 from lordon1a/fix/pipeline-staging-cleanup
ba7d873|2026-11-24|Marvis|Merge pull request #152 from Makto94/fix/net-test-win10
b0328ba|2026-11-24|Marvis|Merge pull request #111 from Dufftwt/fix/narrow-paths-translated-twice
c2e7254|2026-11-24|Marvis|Merge pull request #87 from jeremygold02/codex/copy-stat-frame-window
3e39eac|2026-11-23|quaternion|fix(hle): return XAsync payloads for nonnegative results
764fa19|2026-11-23|lameloball2813-ui|Loader tests: build the probe fixture under CARGO_TARGET_TMPDIR
9a8908c|2026-11-24|Advait Palve|core: show data_dir in config and check its asset policy
d5704aa|2026-11-24|Pedro Fortunato|fix(shader): lower dynamic v_perm_b32 without INT64
b5c6678|2026-11-24|Abdellah K.|loader: unwind a 16-byte aligned copy of the context
5f670bb|2026-11-24|Duff|Loader: a trapped unimplemented import goes into the crash log
d46b078|2026-11-24|Duff|XUser: a gamertag without a suffix has an empty modern suffix
afc4641|2026-11-24|Andrea Casale|net: skip newer-Windows-only exports in only_loopback_is_reachable
3390e47|2026-11-24|Duff|Crash reports: say what an access violation's address holds
babe2db|2026-11-24|Duff|Crash reports: show the bytes of an instruction the CPU refused
d2a4bae|2026-11-24|Marvis|Merge pull request #129 from Luislariosss/fix/duplicate-test-module
9aef3d9|2026-11-24|Marvis|Merge pull request #51 from BaggyALPHA/fix/windows-commit-limit-diagnostic
8a8a38d|2026-11-24|Marvis|Merge pull request #53 from a55uka/fix/wine-physical-pages
0f35dfc|2026-11-24|Marvis|Merge pull request #66 from AndyLatimer/fix/gpu-rect-element-past-stride
fc0c4dd|2026-11-24|Marvis|Merge pull request #77 from Exorion1er/fix/guard-fault-race
683278b|2026-11-24|Marvis|Merge pull request #36 from jeremygold02/codex/root-parameter-fixture
eda38aa|2026-11-24|Marvis|Merge pull request #57 from Yaekai/fix/null-buffer-views
237f2e8|2026-11-24|Marvis|Merge pull request #37 from alessiodeia/fix/valu-med3-fract-trig
062ad14|2026-11-24|Marvis|Merge pull request #50 from Vanthys/fix/format-offset-overflow
cc51927|2026-11-24|Marvis|Merge pull request #30 from AndyLatimer/fix/pe-rva-offset-overflow
27e6442|2026-11-24|Marvis|Merge pull request #28 from ifBars/fix/windows10-import-directory
0e5953d|2026-11-23|ifBars|Merge branch 'main' into fix/windows10-import-directory
ec0d155|2026-11-24|Manuel P├®rez|Watch: saturate the end of announced writes and remaps
257d8bb|2026-11-24|Manuel P├®rez|Audio: reduce the title's XMA read advance before adding it
b304b7d|2026-11-23|Jeremy Goldberg|fix(gpu): average copy statistics over the full reporting interval
c963b71|2026-11-24|tedin7|fix: read a far jump's offset as signed so backward far jumps decode
4c1766c|2026-11-24|Exorion|GPU: keep a guarded block's write fault from being declined in a race
9a5cbbe|2026-11-24|tedin7|fix: skip unused export address slots instead of reporting them at RVA 0
35b054e|2026-11-24|Marvis|GPU: pipelines off the render thread and cached; indirect execution on
5fbea47|2026-11-24|9hvn9yt2vk-netizen|shader: look up an input channel without underflowing
2e5001c|2026-11-24|alessiodeia|shader: keep fract(NaN) as NaN on NVIDIA and add synthetic GPU VALU tests
7e42fef|2026-11-23|BaggyALPHA|Report Windows commit-limit failures in crashes.log
4cca767|2026-11-24|Vanthys|Reject empty XMA test fixture directories
ea24ea1|2026-11-24|9hvn9yt2vk-netizen|shader: a lane mask that needs its own value fails instead of overflowing the stack
97c78d1|2026-11-24|Marvis|Merge pull request #21 from Nostyxx/fix/nvidia-view-heap-cap
9e04419|2026-11-24|Marvis|Merge pull request #15 from 9hvn9yt2vk-netizen/fix-texture-extents
29b558c|2026-11-23|Jeremy Goldberg|test(gpu): use the SDK root parameter in the serializer fixture
e98d3ca|2026-11-23|Manuel P├®rez|PE: checked file offset in rva_to_file_offset
1dd7afd|2026-11-24|Marvis|Merge pull request #14 from 9hvn9yt2vk-netizen/fix-xma-zero-length
e6a2c54|2026-11-24|9hvn9yt2vk-netizen|texture: refuse extents past 16384 texels instead of overflowing
005ad49|2026-11-24|9hvn9yt2vk-netizen|xma: stop at a zero-length frame instead of looping forever
934e67f|2026-11-23|ismailchaka1|loader: emulate AMD-only instructions on Intel CPUs
e8f3b1c|2026-11-23|Marvis|Merge pull request #4 from Exorion1er/fix/tls-callbacks-reloc
015d45b|2026-11-23|Marvis|README: status, subsystem table, how to run, debugging switches
2849477|2026-11-23|Marvis|Run Grand Theft Auto VI from first boot into the world
754619f|2026-11-23|Marvis|Run Minecraft: runtime HLE, D3D12.X device on the host GPU, shader IR
```
