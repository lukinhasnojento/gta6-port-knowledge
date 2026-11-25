# FATOS VERIFICADOS - DexNom

Gerado por git puro, sem LLM.

## A. Identificacao
- Nome: DexNom
- URL: https://github.com/DexNom/VIctory
- Branch analisada: main
- Raiz (commit 0): f4504f6413ce4509604dad66cdd689000a0204dc
- Primeiro commit: f4504f6413ce4509604dad66cdd689000a0204dc | 2026-11-23 01:15:26 +0100 | Marvis | Initial commit
- Ultimo commit: 2e01ab4760db91820ef5f9a9f85761f83cd6b5d0 | 2026-11-25 11:46:07 -0600 | Codex | Keep cached test textures within their reserved guest memory
- Periodo: 2026-11-23 a 2026-11-25 (3 dias com commit)
- Commits em HEAD: 547
- Commits em todas as refs: 547
- Commits so em outras branches: 0
- Arquivos versionados: 195
- Merges: 228

## Autores
```
   221  Marvis <60314886+Marvisak@users.noreply.github.com>
    46  Marvis <marvis@noreply.invalid>
    20  Duff <19514584+Dufftwt@users.noreply.github.com>
    10  DexNom <29156494+DexNom@users.noreply.github.com>
     9  Manuel P├®rez <manuelperezardanuy@gmail.com>
     9  TG0000 <122870448+TG0000@users.noreply.github.com>
     9  Maraco91 <43787609+Maraco91@users.noreply.github.com>
     9  þ╗Öõ╝èÞÄëÚøàÞíÑÚ¡ö <98532177+geiyiliyabumo@users.noreply.github.com>
     8  Luiz Neves <luizgustavobenicio@gmail.com>
     7  marquezini <felipemarquesini10@gmail.com>
     7  quaternion <quaternion@lumina.software>
     6  r4. <192970269+r4tedx@users.noreply.github.com>
     5  Codex <codex@localhost>
     5  AnoHunted <360473103+AnoHunted@users.noreply.github.com>
     5  tedin7 <tommasoderosa94@gmail.com>
```

## Commits por dia
```
2026-11-23     64
2026-11-24    322
2026-11-25    161
```

## 30 arquivos mais alterados
```
   60  crates/victory-shader/src/emit/lower.rs
   33  crates/victory-gpu/src/host.rs
   32  README.md
   30  crates/victory-shader/src/emit/mod.rs
   29  crates/victory-shader/src/emit/valu_tests.rs
   28  crates/victory-gpu/src/descriptor/host.rs
   27  crates/victory-gpu/src/draw.rs
   18  crates/victory-gpu/src/list.rs
   18  crates/victory-loader/src/crash.rs
   16  crates/victory-shader/src/emit/host.rs
   16  crates/victory-gpu/src/shadow.rs
   15  crates/victory-gpu/src/lib.rs
   14  crates/victory-shader/src/emit/shader.rs
   14  crates/victory-gpu/src/pipeline/host.rs
   14  crates/victory-gpu/src/capture.rs
   14  apps/victory/src/translate.rs
   14  crates/victory-shader/src/structure.rs
   13  crates/victory-gpu/src/descriptor.rs
   13  crates/victory-gpu/src/pipeline.rs
   13  crates/victory-gpu/src/resource.rs
   12  crates/victory-shader/src/resource.rs
   12  crates/victory-shader/src/emit/diff_tests.rs
   11  apps/victory/src/run.rs
   11  crates/victory-hle/src/modules/xmem.rs
   11  crates/victory-gpu/src/watch.rs
   11  crates/victory-formats/src/texture.rs
   11  crates/victory-shader/src/emit/waterfall_tests.rs
   11  crates/victory-gpu/src/texture.rs
   11  apps/victory/src/main.rs
   11  docs/architecture.md
```

## Merges (primeiros 80)
```
4118f33|2026-11-25|Merge pull request #928 from BastienRodz/gpu/device-removed-reason
4436fca|2026-11-25|Merge pull request #1696 from Calvix2k/codex/shader-interp-p2-f16-029
5fa5de3|2026-11-25|Merge pull request #1733 from moe-miau/fix/shader-helper-lane-readfirstlane
fcdefcf|2026-11-25|Merge pull request #1322 from KevinPratap/fix/shadow-busy-title-writes
1bde03c|2026-11-25|Merge pull request #1695 from zhaibingye/fix/waterfall-require-device
3688adc|2026-11-25|Merge pull request #1636 from davutsimsekk/fix/flat-offset-twelve-bits
13d1237|2026-11-25|Merge pull request #1626 from AwsAli05/gpu/throttle-swap-chain-retries
3c47fd4|2026-11-25|Merge pull request #1562 from BastienRodz/gpu/fence1-iid
f73e8e1|2026-11-25|Merge pull request #1476 from Texbirds/perf/dxbc-md5-table
520fefc|2026-11-25|Merge pull request #1471 from Texbirds/fix/bitcode-fold-dead-phi-latch
24effa8|2026-11-25|Merge pull request #1446 from geiyiliyabumo/fix/list-create-initial-pipeline
95f6c6e|2026-11-25|Merge pull request #1443 from Luizbc2/fix/rdna2-quadmask
5e3bc84|2026-11-25|Merge pull request #1391 from AnoHunted/shaders-cvt-off-f32-i4
67bcf47|2026-11-25|Merge pull request #1371 from fuzeofficlal/fix/shader-phi-path-compression-950
21b4175|2026-11-25|Merge pull request #1366 from mgunes1311-lab/shader-native-ms-outputs
170ab23|2026-11-25|Merge pull request #1337 from mgunes1311-lab/shader-aniso-mip-point
87f01ff|2026-11-25|Merge pull request #1290 from sklsp/shader-tests/show-device-error
825c045|2026-11-25|Merge pull request #1282 from EatCodeSleep2/fix/shader-branch-to-fallthrough-1242
6160c9e|2026-11-25|Merge pull request #1221 from bartikele/shader-cndmask-other-masks
82494a9|2026-11-25|Merge pull request #1180 from pes7/perf/instruction-name
e2a80cf|2026-11-25|Merge pull request #1159 from geiyiliyabumo/fix/crash-unwind-test-win10
ef65f88|2026-11-25|Merge pull request #1125 from EatCodeSleep2/fix/display-deferral-test-727
19469bd|2026-11-25|Merge pull request #1052 from LouisRiverstone/shaders/int-min3-max3-med3-test
fdce5f5|2026-11-25|Merge pull request #1050 from LouisRiverstone/shaders/f16-minmax-neg-zero-test
6e6004c|2026-11-25|Merge pull request #1021 from CaravelaLabs/caravela/smem-atc-probe
924457e|2026-11-25|Merge pull request #1016 from heromike405-arch/perf/xma-frame-bytes
c6c244d|2026-11-25|Merge pull request #974 from pushkarreddyy/fix/descriptor-cursor-range
02fcf5b|2026-11-25|Merge pull request #661 from coderanonymus/sp3-text-operands
9574cee|2026-11-25|Merge pull request #629 from AlanQiang0418/codex/reject-linear-msaa-layouts
8bf20ad|2026-11-25|Merge pull request #603 from AwsAli05/formats/detile-copy-runs
40f5553|2026-11-25|Merge pull request #560 from Quat3rnion/victory-image-publication
e56919d|2026-11-25|Merge pull request #553 from broccolize/gpu/clear-dxgi-enumeration-outputs
13de0cc|2026-11-25|Merge pull request #536 from AnoHunted/xma-write-block-wrap
9ce9976|2026-11-25|Merge pull request #368 from Quat3rnion/victory-shader-coverage
d763582|2026-11-25|Merge pull request #363 from enzoduarte1/config/empty-config-env
e8e494c|2026-11-25|Merge pull request #361 from booomxd/input/walk-all-devices
32b9fbf|2026-11-25|Merge pull request #351 from pedrofort/fix-texture-depth-pitch
7448632|2026-11-25|Merge pull request #288 from Yaekai/fix/shader-bdd-memo-cap
38011a0|2026-11-25|Merge pull request #270 from Maraco91/gpu/every-object-answers-iunknown
3dce4ba|2026-11-25|Merge pull request #221 from AndyLatimer/fix/loader-stack-dump-safety
a3169a1|2026-11-25|Merge pull request #197 from Alpaca-exe/fix/victory-keys-with-connected-pad
338808e|2026-11-25|Merge pull request #183 from AnoHunted/xma-noise-tests
8a772ea|2026-11-25|Merge pull request #174 from AndyLatimer/fix/gpu-frame-wait-reserves-tick
7d99f63|2026-11-25|Merge pull request #172 from TG0000/perf/xma-imdct-rotations
83211b9|2026-11-25|Merge pull request #162 from Rareuncommon/shaders/sad
256d7b9|2026-11-25|Merge pull request #156 from Rareuncommon/shaders/mul-hi-24
aa1bf51|2026-11-25|Merge pull request #153 from TG0000/perf/gpu-private-data-release
8479c18|2026-11-25|Merge pull request #1726 from npanu420/dstorage/late-page-lists
94856e2|2026-11-25|Merge pull request #1635 from pes7/perf/indirect-skip-clamp
8fb0b28|2026-11-25|Merge pull request #1549 from venglov/formats/addrlib-reference-layouts
fe88382|2026-11-25|Merge pull request #1510 from rouges78/fix/indirect-budget-in-order
41b11b2|2026-11-25|Merge pull request #1477 from geiyiliyabumo/fix/shader-static-sampler-cmp
7ae5e17|2026-11-25|Merge pull request #1474 from Texbirds/fix/shader-lane-bit-wave-ops
aa83e3f|2026-11-25|Merge pull request #1246 from LaunchDoesntCode/gpu/grow-list-upload-scratch
1c16afd|2026-11-25|Merge pull request #1214 from divyeshganta/codex/hle-apu-unaligned-output-ub
181825d|2026-11-25|Merge pull request #1146 from jamesn-xt/fix/capture-output-directory
0299812|2026-11-25|Merge pull request #990 from LouisRiverstone/shaders/cvt-f16-int-test
0cffc54|2026-11-25|Merge pull request #937 from nino-exe/policy-resolve-dotdot
ea48bd3|2026-11-25|Merge pull request #669 from iurysf/dred-breadcrumb-identity
5c0ebce|2026-11-25|Merge pull request #646 from nathanbzrr/shader/wave-exec-const-and-prefix-validation
1134418|2026-11-25|Merge pull request #447 from Dufftwt/fix/signed-64-bit-literals
a9d6f17|2026-11-25|Merge pull request #322 from jsthope/gpu/title-copies-without-region-queries
15de8f9|2026-11-25|Merge pull request #214 from Makto94/fix/clzero-loops
71ac890|2026-11-25|Merge pull request #213 from Dufftwt/fix/crash-log-names-host
46bde7f|2026-11-25|Merge pull request #169 from Dufftwt/fix/tbuffer-d16-loads-packed
c813777|2026-11-25|Merge pull request #126 from TG0000/perf-pr5
12fa535|2026-11-25|Merge pull request #96 from tedin7/vic-xma-continuation
4d140e2|2026-11-25|Merge pull request #1718 from tjthebigbearcat-hub/fix/third-pr
280e484|2026-11-25|Merge pull request #941 from BastienRodz/loader/clwb-near-page-end
2a45e75|2026-11-25|Merge pull request #194 from Maraco91/audio/xma-loop-start-skip
d6e00a8|2026-11-25|Merge pull request #133 from Dufftwt/fix/bare-exe-name-title-folder
f69e487|2026-11-25|Merge pull request #1667 from imjustprism/fix/save-kept-blob-sync
a9d233f|2026-11-25|Merge pull request #1660 from HailLucifer666/fix/pipeline-library-atomic-write-race
711e930|2026-11-25|Merge pull request #1658 from HailLucifer666/fix/texture-staging-oob-and-committed-guard
cbb3431|2026-11-25|Merge pull request #1594 from AwsAli05/cli/propagate-title-exit
ebcd90f|2026-11-25|Merge pull request #1592 from AwsAli05/cli/list-emitted-shaders
17de445|2026-11-25|Merge pull request #1539 from Texbirds/fix/loader-clwb-loop-overflow
e011e88|2026-11-25|Merge pull request #1515 from Calvix2k/codex/ui-dialog-command-guard-033
a985b9d|2026-11-25|Merge pull request #1455 from fluffyflattershy-afk/feat/window-size-limit
12ee4a1|2026-11-25|Merge pull request #1453 from Luizbc2/fix/rectangle-normalized-vertices
```

## Branches remotas nao mergeadas
```

```

## Reverts explicitos
```

```

## Candidatos a correcao
```
ccfb029|2026-11-25|Ivan|Tests: a machine with no GPU has no hardware adapter to check shaders on
5fa5de3|2026-11-25|Marvis|Merge pull request #1733 from moe-miau/fix/shader-helper-lane-readfirstlane
fcdefcf|2026-11-25|Marvis|Merge pull request #1322 from KevinPratap/fix/shadow-busy-title-writes
1bde03c|2026-11-25|Marvis|Merge pull request #1695 from zhaibingye/fix/waterfall-require-device
3688adc|2026-11-25|Marvis|Merge pull request #1636 from davutsimsekk/fix/flat-offset-twelve-bits
520fefc|2026-11-25|Marvis|Merge pull request #1471 from Texbirds/fix/bitcode-fold-dead-phi-latch
24effa8|2026-11-25|Marvis|Merge pull request #1446 from geiyiliyabumo/fix/list-create-initial-pipeline
95f6c6e|2026-11-25|Marvis|Merge pull request #1443 from Luizbc2/fix/rdna2-quadmask
67bcf47|2026-11-25|Marvis|Merge pull request #1371 from fuzeofficlal/fix/shader-phi-path-compression-950
825c045|2026-11-25|Marvis|Merge pull request #1282 from EatCodeSleep2/fix/shader-branch-to-fallthrough-1242
e2a80cf|2026-11-25|Marvis|Merge pull request #1159 from geiyiliyabumo/fix/crash-unwind-test-win10
ef65f88|2026-11-25|Marvis|Merge pull request #1125 from EatCodeSleep2/fix/display-deferral-test-727
c6c244d|2026-11-25|Marvis|Merge pull request #974 from pushkarreddyy/fix/descriptor-cursor-range
02fcf5b|2026-11-25|Marvis|Merge pull request #661 from coderanonymus/sp3-text-operands
32b9fbf|2026-11-25|Marvis|Merge pull request #351 from pedrofort/fix-texture-depth-pitch
7448632|2026-11-25|Marvis|Merge pull request #288 from Yaekai/fix/shader-bdd-memo-cap
3dce4ba|2026-11-25|Marvis|Merge pull request #221 from AndyLatimer/fix/loader-stack-dump-safety
a3169a1|2026-11-25|Marvis|Merge pull request #197 from Alpaca-exe/fix/victory-keys-with-connected-pad
8a772ea|2026-11-25|Marvis|Merge pull request #174 from AndyLatimer/fix/gpu-frame-wait-reserves-tick
fe88382|2026-11-25|Marvis|Merge pull request #1510 from rouges78/fix/indirect-budget-in-order
41b11b2|2026-11-25|Marvis|Merge pull request #1477 from geiyiliyabumo/fix/shader-static-sampler-cmp
7ae5e17|2026-11-25|Marvis|Merge pull request #1474 from Texbirds/fix/shader-lane-bit-wave-ops
181825d|2026-11-25|Marvis|Merge pull request #1146 from jamesn-xt/fix/capture-output-directory
5c0ebce|2026-11-25|Marvis|Merge pull request #646 from nathanbzrr/shader/wave-exec-const-and-prefix-validation
1134418|2026-11-25|Marvis|Merge pull request #447 from Dufftwt/fix/signed-64-bit-literals
15de8f9|2026-11-25|Marvis|Merge pull request #214 from Makto94/fix/clzero-loops
71ac890|2026-11-25|Marvis|Merge pull request #213 from Dufftwt/fix/crash-log-names-host
46bde7f|2026-11-25|Marvis|Merge pull request #169 from Dufftwt/fix/tbuffer-d16-loads-packed
4d140e2|2026-11-25|Marvis|Merge pull request #1718 from tjthebigbearcat-hub/fix/third-pr
d6e00a8|2026-11-25|Marvis|Merge pull request #133 from Dufftwt/fix/bare-exe-name-title-folder
f69e487|2026-11-25|Marvis|Merge pull request #1667 from imjustprism/fix/save-kept-blob-sync
a9d233f|2026-11-25|Marvis|Merge pull request #1660 from HailLucifer666/fix/pipeline-library-atomic-write-race
711e930|2026-11-25|Marvis|Merge pull request #1658 from HailLucifer666/fix/texture-staging-oob-and-committed-guard
17de445|2026-11-25|Marvis|Merge pull request #1539 from Texbirds/fix/loader-clwb-loop-overflow
12ee4a1|2026-11-25|Marvis|Merge pull request #1453 from Luizbc2/fix/rectangle-normalized-vertices
7db1e07|2026-11-25|Marvis|Merge pull request #1411 from BaggyALPHA/fix/clear-blocked-name-result
6355117|2026-11-25|Marvis|Merge pull request #1274 from EatCodeSleep2/fix/xpackage-unregister-progress-1249
0fcbee4|2026-11-25|Marvis|Merge pull request #1177 from DoubleD17/fix/census-deduplicate-inputs
d73d8c9|2026-11-25|Marvis|Merge pull request #1109 from divyeshganta/fix/cache-staging-collisions
9bd146d|2026-11-25|Marvis|Merge pull request #618 from JFIREonFIRE/fix/append-aligned-input-offsets
ce2d91f|2026-11-25|Marvis|Merge pull request #475 from nathanbzrr/fix/xstore-missing-result-slots
43b7c08|2026-11-25|Marvis|Merge pull request #474 from nathanbzrr/fix/gpu-console-sampler-validation
ac2525d|2026-11-25|Marvis|Merge pull request #373 from marquezinii/fix/module-file-name-null-buffer
897efe0|2026-11-25|Marvis|Merge pull request #370 from marquezinii/fix/text-entry-length-limit
8595259|2026-11-25|Marvis|Merge pull request #359 from Dufftwt/fix/manifest-quoted-angle-bracket
c5b83b9|2026-11-25|Marvis|Merge pull request #219 from DiegoFR20/investigation/pe-validation
1117251|2026-11-25|Marvis|Merge pull request #52 from AndyLatimer/fix/gpu-descriptor-mirror-and-query-bounds
26e03c3|2026-11-25|Marvis|Merge pull request #1729 from moe-miau/fix/xgamesave-stray-versions
88de5fc|2026-11-25|Marvis|Merge pull request #1611 from AgneyaK/fix/missing-package-dlls
76bcd2b|2026-11-25|Marvis|Merge pull request #1080 from giusepperainone/fix/constrained-before-suspend
e82a6ad|2026-11-25|Marvis|Merge pull request #615 from KoLolX3/fix/env-switch-zero
2d8cac2|2026-11-25|Marvis|Merge pull request #476 from nathanbzrr/fix/census-dedupe-without-hint
a222fe3|2026-11-25|Marvis|Merge pull request #357 from Dufftwt/fix/host-dll-load-outside-cache-lock
c7f1ae3|2026-11-25|Marvis|Merge pull request #335 from marquezinii/fix/store-package-identifier-error
ed6429e|2026-11-25|Marvis|Merge pull request #132 from Dufftwt/fix/terminate-cancels-delayed-callbacks
f8c7498|2026-11-25|Marvis|Merge pull request #122 from Dufftwt/fix/xasync-ended-call-reused-block
c47b529|2026-11-24|Marvis|Shader warm-up (translate --cache), a cache keyed by the translator, pipeline queue fixes
77053a0|2026-11-24|Marvis|Shaders: accuracy fixes for discards, f16 rounding, denormals, med3 and signed zeros
4fc9d70|2026-11-25|HailLucifer666|GPU: keep Staging::detile inside the staging buffer and harden committed()
be824d9|2026-11-25|HailLucifer666|GPU: serialize pipeline library atomic writes with unique temp files
a14a545|2026-11-24|Maledetto|Merge remote-tracking branch 'origin/main' into fix/clzero-loops-merge
60d2cf9|2026-11-24|BastienRdz|Loader: a clwb in the last bytes of a page is emulated instead of faulting
52f0a09|2026-11-24|nino-exe|policy: resolve a `..` in the not-yet-existing part of a path
eb8067f|2026-11-24|quaternion|fix(hle): keep apostrophes intact when splitting relaunch args
71e8866|2026-11-23|quaternion|fix(hle): honor the requested executable on relaunch
ca362a4|2026-11-23|quaternion|fix(hle): read manifest fields past inline XML comments
1aac20a|2026-11-23|quaternion|fix(loader): stage prepared images in per-writer files
d0dd9fc|2026-11-23|quaternion|fix(pe): keep every name for a shared export ordinal
c1be75f|2026-11-24|pushkarreddyy|Merge pull request #1 from MichaelSmithAI/test/871-cursor-crossing
121412b|2026-11-24|þ╗Öõ╝èÞÄëÚøàÞíÑÚ¡ö|loader(crash): verify unwind against known caller frames when RtlCaptureStackBackTrace returns empty
a32eacd|2026-11-24|DocBlack89|fix(pipeline): retry collisions when creating cache staging files
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
882664e|2026-11-24|Manuel P├®rez|GPU: keep the frame wait's tick for the diagnostics log
52d528b|2026-11-24|OpenBot|GPU: match a breadcrumb node's name followed by the debug layer's note; drop the position fallback
8a358e3|2026-11-24|OpenBot|GPU: find the lists in the DRED breadcrumbs by position when the debug layer hides them
3e39eac|2026-11-23|quaternion|fix(hle): return XAsync payloads for nonnegative results
6526965|2026-11-24|OpenBot|GPU: find a list in the DRED breadcrumbs by its name, not only its pointer
764fa19|2026-11-23|lameloball2813-ui|Loader tests: build the probe fixture under CARGO_TARGET_TMPDIR
12eba49|2026-11-24|nathanbzrr|Shaders: fix whole-wave EXEC inline constant sign-extension and prefix lane validation
6325df7|2026-11-24|LF2000 Builder|Shaders: print only the operands an instruction has in SP3 text
691236d|2026-11-24|JFIREonFIRE|GPU: align appended input-element offsets like D3D12
a4974e4|2026-11-23|KoLolX3|GPU: VICTORY_* switches set to 0 are off
9a8908c|2026-11-24|Advait Palve|core: show data_dir in config and check its asset policy
65062e1|2026-11-23|LF2000 Builder|xmem: report allocations the host refuses
2c22e2b|2026-11-24|nathanbzrr|XStore: wire offline *Result slots and fix OOB XStoreShowProductPageUIResult slot
e6fe8ec|2026-11-24|nathanbzrr|GPU: fix CreateSamplerX mip-none base level, LOD range, bias clamp, and invalid enum values
d5704aa|2026-11-24|Pedro Fortunato|fix(shader): lower dynamic v_perm_b32 without INT64
c000581|2026-11-23|marquezini|Kernel: GetModuleFileName of a title image into a null buffer
93a24b7|2026-11-23|Pedro Fortunato|fix(formats): reject overlapping texture depth slices
b5c6678|2026-11-24|Abdellah K.|loader: unwind a 16-byte aligned copy of the context
5f670bb|2026-11-24|Duff|Loader: a trapped unimplemented import goes into the crash log
acdcc41|2026-11-24|Manuel P├®rez|Loader: suspend a thread before reading its TEB; skip bad dump times
d46b078|2026-11-24|Duff|XUser: a gamertag without a suffix has an empty modern suffix
e57e76a|2026-11-23|Diego|fix(pe): validate section raw data bounds
afc4641|2026-11-24|Andrea Casale|net: skip newer-Windows-only exports in only_loopback_is_reachable
11ca3ce|2026-11-24|Maraco91|Media: a share is refused however it is spelled
7f00a62|2026-11-24|Duff|Crash reports: name the host's CPU and Windows build
3390e47|2026-11-24|Duff|Crash reports: say what an access violation's address holds
d4d1822|2026-11-24|AnoHunted|XMA: noise tests, and end a frame at its data instead of looping forever
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
628b95d|2026-11-24|Manuel P├®rez|GPU: hold SHADER_HEAPS across descriptor mirror and check query offsets
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
