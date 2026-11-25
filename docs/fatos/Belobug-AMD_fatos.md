# FATOS VERIFICADOS - Belobug-AMD

Gerado por git puro, sem LLM.

## A. Identificacao
- Nome: Belobug-AMD
- URL: https://github.com/Belobug/VIctory-Belobug-AMD
- Branch analisada: main
- Raiz (commit 0): f4504f6413ce4509604dad66cdd689000a0204dc
- Primeiro commit: f4504f6413ce4509604dad66cdd689000a0204dc | 2026-11-23 01:15:26 +0100 | Marvis | Initial commit
- Ultimo commit: 4f62c2a5e27e7beebf3e8d68f77898e43dc7b77c | 2026-11-25 15:56:22 -0500 | beloooo | The ambient light follows the sky by default
- Periodo: 2026-11-23 a 2026-11-25 (3 dias com commit)
- Commits em HEAD: 751
- Commits em todas as refs: 751
- Commits so em outras branches: 0
- Arquivos versionados: 186
- Merges: 287

## Autores
```
   189  Claude <noreply@anthropic.com>
   148  beloooo <belobug69@gmail.com>
    87  Marvis <60314886+Marvisak@users.noreply.github.com>
    43  mich0206 <antonin.michel@etudiant.univ-reims.fr>
    20  Duff <19514584+Dufftwt@users.noreply.github.com>
    19  Manuel P├®rez <manuelperezardanuy@gmail.com>
    19  r4. <192970269+r4tedx@users.noreply.github.com>
    14  TG0000 <122870448+TG0000@users.noreply.github.com>
    13  Marvis <marvis@noreply.invalid>
    12  tedin7 <tommasoderosa94@gmail.com>
    11  wrothgaril <211408787+wrothgaril@users.noreply.github.com>
     9  Maraco91 <43787609+Maraco91@users.noreply.github.com>
     8  romainhedouin <romain.hedouin@mantra.ms>
     7  Nikita Romanovich <romanovicnikita1@gmail.com>
     7  Yaekai <21324379+Yaekai@users.noreply.github.com>
```

## Commits por dia
```
2026-11-23    107
2026-11-24    574
2026-11-25     70
```

## 30 arquivos mais alterados
```
  129  crates/victory-shader/src/emit/lower.rs
   76  crates/victory-gpu/src/host.rs
   60  crates/victory-gpu/src/draw.rs
   54  crates/victory-shader/src/emit/mod.rs
   54  crates/victory-gpu/src/descriptor/host.rs
   45  crates/victory-gpu/src/shadow.rs
   32  crates/victory-gpu/src/resource.rs
   31  crates/victory-gpu/src/list.rs
   28  README.md
   24  crates/victory-gpu/src/watch.rs
   23  crates/victory-shader/src/emit/valu_tests.rs
   21  crates/victory-gpu/src/lib.rs
   20  crates/victory-gpu/src/pipeline.rs
   20  crates/victory-gpu/src/queue.rs
   20  crates/victory-gpu/src/texture.rs
   19  crates/victory-shader/src/emit/shader.rs
   18  crates/victory-loader/src/crash.rs
   18  crates/victory-hle/src/modules/xmem.rs
   17  crates/victory-shader/src/resource.rs
   15  crates/victory-gpu/src/capture.rs
   14  crates/victory-shader/src/native.rs
   13  crates/victory-gpu/src/descriptor.rs
   12  crates/victory-hle/src/modules/xasync.rs
   12  crates/victory-gpu/src/pipeline/host.rs
   11  crates/victory-shader/src/emit/bitcode.rs
   11  apps/victory/src/translate.rs
   10  crates/victory-hle/src/modules/mod.rs
   10  crates/victory-hle/src/modules/kernel.rs
   10  crates/victory-gpu/src/shader_cache.rs
   10  crates/victory-hle/src/modules/gameinput.rs
```

## Merges (primeiros 80)
```
14545fa|2026-11-25|Merge pull request #96 from Key-rs: thin LTO and one codegen unit for the release profile
127830e|2026-11-25|Merge pull request #34 from romainhedouin: A table made afresh is looked up by a quick hash and its record words are compared, instead of a SipHash over them (a collision now costs a copy, not wrong descriptors)
a1ef2dd|2026-11-25|Merge pull request #33 from romainhedouin: A draw keeps its root constants on the stack, its fresh-table slots in a bit set and the tables it made in a buffer the state reuses, so a draw no longer allocates for them (up to 2 + 1 + 1 allocations per draw)
0145ca7|2026-11-25|Merge pull request #24 from romainhedouin: A root constant buffer not 256-byte aligned is copied into list scratch once for the draws of a list that bind it, until a title copy or run intervenes (up to 64 KiB less copied per repeat)
58655f9|2026-11-25|Merge pull request #29 from romainhedouin: A range shaders write is declared once until the title next writes memory, and the submission finds the unique ranges with a set, so a list of thousands of draws no longer scans and re-snapshots the same ranges per step
f53ae3d|2026-11-25|Merge pull request #30 from romainhedouin: Buffer-to-texture copies take their staging bytes from the list's pooled scratch chunks instead of creating and mapping a committed upload buffer each, which is much cheaper
684a9b5|2026-11-25|Merge pull request #28 from romainhedouin: Clamps hoisted into one list bind their root signature, pipeline and budget once, not before every clamp (3 fewer D3D12 calls per indirect draw)
ed273ef|2026-11-25|Merge pull request #25 from romainhedouin: A pipeline a list binds again is not named, queried and kept again (one lookup by address instead of two QueryInterface calls, three locks and a name search per switch)
13d6b73|2026-11-25|Merge pull request #27 from romainhedouin: The host heap behind a descriptor heap is returned without taking the pending lock once it is made, since nothing is queued after that
64170dd|2026-11-25|Merge pull request #18 from PikPakPik/perf-copies-paralleles: sparse copies snapshot and copy back their tiles on several threads
d6070f1|2026-11-25|Merge pull request #45 from Arvi23/pr/integer-target-blend: no blending on integer render targets in host pipelines
04b1a1d|2026-11-24|Merge official main up to 8e79c80 (CI), before the per-lane phi and waterfall changes
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
e136a3f|2026-11-23|Merge PR 413 (bindless heap materialization) into the local branch
da7a7f8|2026-11-23|Merge PR 414 (community integration) into the local branch
d3917df|2026-11-24|Merge PR #342 (Shaders: lower partial-channel typed buffer stores) into the integration
3287834|2026-11-24|Merge PR #333 (Shaders: native blobs must place root parameters, vertex inputs and thread ids in real registers) into the integration
9a7c323|2026-11-24|Merge PR #332 (Shaders: lower partial-channel image_store) into the integration
d9e7d31|2026-11-24|Merge PR #329 (Query record counts for structured buffers read only through their descriptor) into the integration
e0a596a|2026-11-24|Merge PR #328 (Shaders: lower MSAA image_load and image_get_resinfo) into the integration
4a8d5ec|2026-11-24|Merge PR #320 (Shaders: translate for the host's wave sizes; console flow for lane reads where lanes branch differently) into the integration
efd7063|2026-11-24|Merge PR #317 (Shaders: support cube and MSAA dimensions in descriptor_data) into the integration
eb962e1|2026-11-24|Merge PR #306 (Shader: fix biased 16-bit image samples and DXIL validator failures) into the integration
e186586|2026-11-24|Merge PR #305 (Shaders: lower 64-bit image atomics) into the integration
60303c3|2026-11-24|Merge PR #288 (Shaders: cap BDD memo table and bail out early on overflow) into the integration
```

## Branches remotas nao mergeadas
```

```

## Reverts explicitos
```
e0c582e|2026-11-23|Revert the integration's old PR 185 (copies gigabytes per frame in the world)
a62995f|2026-11-24|Take out upstream PRs that do not serve the 3D picture or stability
85f90f1|2026-11-24|Revert PR #234 (xmem section views, overlapped DirectStorage) from the integration
d48de53|2026-11-24|Reapply "Merge remote-tracking branch 'pr/12' into integrate/all"
50a3f1e|2026-11-24|Reapply "Merge remote-tracking branch 'pr/150' into integrate/all"
8302f6a|2026-11-24|Reapply "Merge remote-tracking branch 'pr/213' into integrate/all"
e1dd56b|2026-11-24|Revert "Merge remote-tracking branch 'pr/12' into integrate/all"
d0f97ef|2026-11-24|Revert "Merge remote-tracking branch 'pr/245' into integrate/all"
5da8aaf|2026-11-24|Revert "Merge remote-tracking branch 'pr/213' into integrate/all"
4270e8f|2026-11-24|Revert "Merge remote-tracking branch 'pr/204' into integrate/all"
13d385d|2026-11-24|Revert "Merge remote-tracking branch 'pr/198' into integrate/all"
fe72473|2026-11-24|Revert "Merge remote-tracking branch 'pr/207' into integrate/all"
011e7de|2026-11-24|Revert "Merge remote-tracking branch 'pr/133' into integrate/all"
9b32471|2026-11-24|Revert "Merge remote-tracking branch 'pr/26' into integrate/all"
4eb9643|2026-11-24|Revert "Merge remote-tracking branch 'pr/87' into integrate/all"
7dfc31b|2026-11-24|Revert "Merge remote-tracking branch 'pr/115' into integrate/all"
b79192f|2026-11-24|Revert "Merge remote-tracking branch 'pr/244' into integrate/all"
2645fcd|2026-11-24|Revert "Merge remote-tracking branch 'pr/150' into integrate/all"
```

## Candidatos a correcao
```
ae75a66|2026-11-25|beloooo|Clean-up passes over translated shaders, from the fork at ba8697aa
1503444|2026-11-24|beloooo|Buffer element counts by slot from the host's table, not a size query (AMD compiler crash)
9956958|2026-11-24|beloooo|Tiles of copies no longer given back (crash 120 frames into the world); view indices kept inside the heap
a873b24|2026-11-24|beloooo|victory-gpu: fix GPU use-after-free in texpool/Staging/HostHeap and coalesce VirtualProtect
2b6adbb|2026-11-24|beloooo|victory-gpu: cache unmapped holes in watch::guard (fix 2.5GB/frame fault storm) and set 240 FPS default cap
d3a5f2d|2026-11-24|beloooo|Shaders: fix f16 inline constants/EXEC init/wave restore; add f64 ops, f16/f64 wave sums, and mesh MaskData::Host
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
0d2f6a6|2026-11-24|beloooo|Survive driver compiler crashes; diagnostics for target dumps on demand
e0c582e|2026-11-23|beloooo|Revert the integration's old PR 185 (copies gigabytes per frame in the world)
161dc7f|2026-11-23|beloooo|Diagnostics: keys and captures on demand; test fixes after the merge
d5704aa|2026-11-24|Pedro Fortunato|fix(shader): lower dynamic v_perm_b32 without INT64
eb962e1|2026-11-24|Claude|Merge PR #306 (Shader: fix biased 16-bit image samples and DXIL validator failures) into the integration
302285d|2026-11-24|Claude|Merge PR #284 (Shaders: RDNA2's legacy multiply-adds are the fused ones) into the integration
a62995f|2026-11-24|Claude|Take out upstream PRs that do not serve the 3D picture or stability
536aa29|2026-11-23|SkippySteve|Shaders: a native vertex input must load into the VGPRs
85f90f1|2026-11-24|Claude|Revert PR #234 (xmem section views, overlapped DirectStorage) from the integration
6cdedbe|2026-11-24|Claude|Fix the three Windows tests that failed on integrate/all
808e921|2026-11-23|SkippySteve|Shaders: a native program's root parameters must sit in its SGPRs
ee1f683|2026-11-24|Claude|Merge fix/packed-half: keep #176's half_inputs and add its end-to-end test
0758d0a|2026-11-24|Claude|Merge fix/validate (DXIL validator fixes) into the integration
2d43710|2026-11-24|Claude|Merge fix/firstlane (per-lane descriptors through phis) into the integration
b5c6678|2026-11-24|Abdellah K.|loader: unwind a 16-byte aligned copy of the context
6b0f69d|2026-11-24|wrothgaril|Shader: fix biased 16-bit image samples and DXIL validator failures
e4c7c8a|2026-11-24|Claude|Merge PR #200 (Shaders: lower v_div_scale_f32, v_div_fmas_f32 and v_div_fixup_f32) into the integration
d48de53|2026-11-24|Claude|Reapply "Merge remote-tracking branch 'pr/12' into integrate/all"
50a3f1e|2026-11-24|Claude|Reapply "Merge remote-tracking branch 'pr/150' into integrate/all"
8302f6a|2026-11-24|Claude|Reapply "Merge remote-tracking branch 'pr/213' into integrate/all"
5cd3c93|2026-11-24|Claude|Merge PR #234 (HLE: overlapped DirectStorage I/O and persistent xmem section views) into the integration
f653c1c|2026-11-24|Claude|Merge PR #216 (Loader & HLE: harden VEH crash handler, PE prep/exports/TLS, and allow oleaut32) into the integration
d18cd26|2026-11-24|Claude|Shaders: write signature parts as the validator rebuilds them
e1dd56b|2026-11-24|Claude|Revert "Merge remote-tracking branch 'pr/12' into integrate/all"
d0f97ef|2026-11-24|Claude|Revert "Merge remote-tracking branch 'pr/245' into integrate/all"
5da8aaf|2026-11-24|Claude|Revert "Merge remote-tracking branch 'pr/213' into integrate/all"
4270e8f|2026-11-24|Claude|Revert "Merge remote-tracking branch 'pr/204' into integrate/all"
13d385d|2026-11-24|Claude|Revert "Merge remote-tracking branch 'pr/198' into integrate/all"
fe72473|2026-11-24|Claude|Revert "Merge remote-tracking branch 'pr/207' into integrate/all"
011e7de|2026-11-24|Claude|Revert "Merge remote-tracking branch 'pr/133' into integrate/all"
9b32471|2026-11-24|Claude|Revert "Merge remote-tracking branch 'pr/26' into integrate/all"
4eb9643|2026-11-24|Claude|Revert "Merge remote-tracking branch 'pr/87' into integrate/all"
7dfc31b|2026-11-24|Claude|Revert "Merge remote-tracking branch 'pr/115' into integrate/all"
b79192f|2026-11-24|Claude|Revert "Merge remote-tracking branch 'pr/244' into integrate/all"
2645fcd|2026-11-24|Claude|Revert "Merge remote-tracking branch 'pr/150' into integrate/all"
e232d3d|2026-11-23|desp|Shaders: query a descriptor's level count only when word 3 is read
3f8a1de|2026-11-24|Claude|Merge PR #136 (Shader: support s_buffer descriptor/structured loads, dead descriptor phis, PSV0/ISG1 fixes, calculateLOD and image_gather4_l) into the integration
01c3a5a|2026-11-24|Claude|Merge branch 'fix/spill' into test/nuit
db20637|2026-11-24|Nikita Romanovich|Shaders: fix v_div_scale_f32 src1/src2 order (den, num) and zero->NaN
7f95b87|2026-11-24|Manuel P├®rez|Shaders: share and serialize WARP host operations across threads
1d5e1bf|2026-11-24|Claude|Merge the night's shader fixes into the test stack
f435ff5|2026-11-24|Claude|Merge branch 'fix/waterfall' into stack/shader-fixes-on-d2a4bae
34e0a70|2026-11-24|Claude|Merge branch 'fix/interp' into stack/shader-fixes-on-d2a4bae
203d9f3|2026-11-24|wrothgaril|HLE: offload async saves, reuse audio buffers, pin APU heap, and decouple input rumble/dialogs
d105484|2026-11-24|mich0206|Merge fork branch fix/gpu-hang-texture-lifetime (7dddec2)
5f670bb|2026-11-24|Duff|Loader: a trapped unimplemented import goes into the crash log
a5af45f|2026-11-24|Nikita Romanovich|Shaders: fix v_mad_i64_i32 65-bit carry, refuse clamp, add -3*5+0 test
b87a968|2026-11-23|shadowhacker|HLE: report the host's commit headroom before the title starts
acdcc41|2026-11-24|Manuel P├®rez|Loader: suspend a thread before reading its TEB; skip bad dump times
d46b078|2026-11-24|Duff|XUser: a gamertag without a suffix has an empty modern suffix
e57e76a|2026-11-23|Diego|fix(pe): validate section raw data bounds
d38b73f|2026-11-24|mich0206|Merge fork branch stack/shader-fixes-on-d2a4bae (2a10e49)
afc4641|2026-11-24|Andrea Casale|net: skip newer-Windows-only exports in only_loopback_is_reachable
2a10e49|2026-11-24|Claude|Merge branch 'fix/wholewave' into stack/shader-fixes-on-d2a4bae
c263e28|2026-11-24|Claude|Merge fix/gather onto the shader stack
a8bfe78|2026-11-24|Claude|Shaders: follow lane writes and scalar steps in whole-wave sections
6269b57|2026-11-24|wrothgaril|Loader & HLE: harden VEH crash handler, PE prep/exports/TLS, and allow oleaut32
56d3a5d|2026-11-24|r4.|GPU: occlusion, statistics and timestamp queries run on the host
728a8e3|2026-11-24|r4.|GPU: the depth bounds test reaches the host
11ca3ce|2026-11-24|Maraco91|Media: a share is refused however it is spelled
7f00a62|2026-11-24|Duff|Crash reports: name the host's CPU and Windows build
3e8f2d3|2026-11-24|Nikita Romanovich|Shaders: lower v_div_scale_f32, v_div_fmas_f32 and v_div_fixup_f32
3b039fd|2026-11-24|mich0206|translate: a selected descriptor source is no leaf either (stack fix for the shader-fixes branch)
bccb4d9|2026-11-24|mich0206|Merge fork branch fix/gpu-hang-split-submissions (082e9a2)
5df733f|2026-11-24|mich0206|Merge fork branch stack/shader-fixes-on-d2a4bae (9707daf)
3390e47|2026-11-24|Duff|Crash reports: say what an access violation's address holds
ce9c884|2026-11-24|Claude|Merge remote-tracking branch 'fork/fix/sgpr-address-not-root' into stack/approx
e1d9f29|2026-11-24|Claude|Merge remote-tracking branch 'fork/fix/shader-gaps-combined' into stack/approx
d4d1822|2026-11-24|AnoHunted|XMA: noise tests, and end a frame at its data instead of looping forever
babe2db|2026-11-24|Duff|Crash reports: show the bytes of an instruction the CPU refused
0434f3d|2026-11-24|mich0206|Merge fix/sgpr-address-not-root
e55eb69|2026-11-24|Ortomanic|Shader: key heap handles by element type too
be7369a|2026-11-24|r4.|Shaders: texel offsets, comparison and gather forms of image samples
f96b197|2026-11-24|r4.|Shaders: an audit of which opcodes lower
d2a4bae|2026-11-24|Marvis|Merge pull request #129 from Luislariosss/fix/duplicate-test-module
4c5545a|2026-11-24|mich0206|Merge fork branch fix/gpu-hang-with-pr32 (new tip)
34a0823|2026-11-24|Kam471|Input: read DualSense pads over HID when XInput has none
4f49a25|2026-11-24|mich0206|list: pass the host list by reference to texture upload in readbacks (stack fix: #67 on top of the PR32 branch)
8fb5435|2026-11-24|mich0206|Merge branch 'fix/shader-gaps-combined' of https://github.com/AntoninMichelq/VIctory into victory-stack
78b3463|2026-11-24|mich0206|Merge fork branch fix/gpu-hang-with-pr32
a6731dd|2026-11-24|Claude|Merge branches 'fix/address-from-root-param' and 'fix/scalar-read-as-data' into combo-test
8ae232f|2026-11-24|Dom|Shader: support s_buffer descriptor/structured loads, dead descriptor phis, PSV0/ISG1 fixes, calculateLOD and image_gather4_l
6b867e6|2026-11-24|AntoninMichelq|GPU: keep the indirect budget from wrapping; drop dispatches past the group limit
219e3ce|2026-11-23|Jeremy Goldberg|fix(gpu): preserve pixels outside texture readback regions
9aef3d9|2026-11-24|Marvis|Merge pull request #51 from BaggyALPHA/fix/windows-commit-limit-diagnostic
8a8a38d|2026-11-24|Marvis|Merge pull request #53 from a55uka/fix/wine-physical-pages
0322732|2026-11-24|lordon1a|Merge remote-tracking branch 'origin/main' into fix/med3-dxil-isnan
0f35dfc|2026-11-24|Marvis|Merge pull request #66 from AndyLatimer/fix/gpu-rect-element-past-stride
fc0c4dd|2026-11-24|Marvis|Merge pull request #77 from Exorion1er/fix/guard-fault-race
683278b|2026-11-24|Marvis|Merge pull request #36 from jeremygold02/codex/root-parameter-fixture
eda38aa|2026-11-24|Marvis|Merge pull request #57 from Yaekai/fix/null-buffer-views
237f2e8|2026-11-24|Marvis|Merge pull request #37 from alessiodeia/fix/valu-med3-fract-trig
062ad14|2026-11-24|Marvis|Merge pull request #50 from Vanthys/fix/format-offset-overflow
cc51927|2026-11-24|Marvis|Merge pull request #30 from AndyLatimer/fix/pe-rva-offset-overflow
27e6442|2026-11-24|Marvis|Merge pull request #28 from ifBars/fix/windows10-import-directory
f468457|2026-11-24|tedin7|fix: model v_permlane16_b32 in whole-wave sections
6e6d3fc|2026-11-24|tedin7|fix: accept shader loops that branch to the first instruction
0e5953d|2026-11-23|ifBars|Merge branch 'main' into fix/windows10-import-directory
ec0d155|2026-11-24|Manuel P├®rez|Watch: saturate the end of announced writes and remaps
257d8bb|2026-11-24|Manuel P├®rez|Audio: reduce the title's XMA read advance before adding it
b304b7d|2026-11-23|Jeremy Goldberg|fix(gpu): average copy statistics over the full reporting interval
a80eb17|2026-11-24|Faultz|GPU: cap a sampler heap past the host's limit instead of losing it
c963b71|2026-11-24|tedin7|fix: read a far jump's offset as signed so backward far jumps decode
4c1766c|2026-11-24|Exorion|GPU: keep a guarded block's write fault from being declined in a race
9a5cbbe|2026-11-24|tedin7|fix: skip unused export address slots instead of reporting them at RVA 0
6c8af09|2026-11-24|Yaekai|Shaders: support image_sample_b with a16 and _cl LOD-clamp variants
35b054e|2026-11-24|Marvis|GPU: pipelines off the render thread and cached; indirect execution on
5fbea47|2026-11-24|9hvn9yt2vk-netizen|shader: look up an input channel without underflowing
2e5001c|2026-11-24|alessiodeia|shader: keep fract(NaN) as NaN on NVIDIA and add synthetic GPU VALU tests
628b95d|2026-11-24|Manuel P├®rez|GPU: hold SHADER_HEAPS across descriptor mirror and check query offsets
7e42fef|2026-11-23|BaggyALPHA|Report Windows commit-limit failures in crashes.log
4cca767|2026-11-24|Vanthys|Reject empty XMA test fixture directories
ea24ea1|2026-11-24|9hvn9yt2vk-netizen|shader: a lane mask that needs its own value fails instead of overflowing the stack
97c78d1|2026-11-24|Marvis|Merge pull request #21 from Nostyxx/fix/nvidia-view-heap-cap
9e04419|2026-11-24|Marvis|Merge pull request #15 from 9hvn9yt2vk-netizen/fix-texture-extents
29b558c|2026-11-23|Jeremy Goldberg|test(gpu): use the SDK root parameter in the serializer fixture
b3ecede|2026-11-24|Manuel P├®rez|HLE: leave a released-out COM count at zero
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
