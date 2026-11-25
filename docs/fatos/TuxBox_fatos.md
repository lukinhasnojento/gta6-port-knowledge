# FATOS VERIFICADOS - TuxBox

Gerado por git puro, sem LLM.

## A. Identificacao
- Nome: TuxBox
- URL: https://github.com/brcly/TuxBox
- Branch analisada: main
- Raiz (commit 0): 6d15471b0ebac7e89f4ac3959667a4b6faa7fa60
- Primeiro commit: 6d15471b0ebac7e89f4ac3959667a4b6faa7fa60 | 2026-11-21 14:23:06 +0000 | brcly | Initial commit
- Ultimo commit: bd847b5d7c8aa5f3a61fef6015074dad884215dc | 2026-11-25 22:41:32 +0000 | brcly | launcher: pin GE-Proton11-14 as the default Proton
- Periodo: 2026-11-21 a 2026-11-25 (5 dias com commit)
- Commits em HEAD: 591
- Commits em todas as refs: 592
- Commits so em outras branches: 1
- Arquivos versionados: 1652
- Merges: 190

## Autores
```
   327  brcly <barclay@tuta.io>
    83  Valdi320 <61891540+Valdi320@users.noreply.github.com>
    47  Manuel Smith Lazarte Alcocer <smith.lazarte.3@gmail.com>
    46  NathansDomainSolutions <184714143+NathansDomainSolutions@users.noreply.github.com>
    10  Pavel Kirilin <s3riussan@gmail.com>
     9  Vladimir <39053957+Vladimir-Human@users.noreply.github.com>
     9  JohnVonWells <3764276+JohnVonWells@users.noreply.github.com>
     8  Jerkin42 <66524771+Jerkin42@users.noreply.github.com>
     7  Lucas Boyles <lucas@postblink.wtf>
     6  user <user@localhost>
     5  ReverendOzmund <catmilker@proton.me>
     5  Thierry PROST <308755+3kynox@users.noreply.github.com>
     5  Jeffrey C <990135+JeffreyBytes@users.noreply.github.com>
     3  brcly <1431358+brcly@users.noreply.github.com>
     3  Andrey <lopsin232@gmail.com>
```

## Commits por dia
```
2026-11-21      6
2026-11-22     74
2026-11-23     48
2026-11-24    271
2026-11-25    192
```

## 30 arquivos mais alterados
```
  105  tools/shader-compiler/src/hlsl.rs
   58  docs/progress.svg
   50  docs/SHADERS.md
   36  project_tuxbox/verify.py
   33  docs/PROJECTSCARLETT.md
   29  tools/shader-compiler/src/hlsl/wave_sum.rs
   27  evidence/library-status.json
   26  README.md
   25  runtime/d3d12/src/commands.rs
   25  docs/PROGRESS.md
   25  build.py
   25  runtime/d3d12/src/device/stream.rs
   24  project_tuxbox/runtime_shaders.py
   24  runtime/d3d12/src/device/heap.rs
   23  runtime/d3d12/src/device.rs
   20  docs/MATRIX_AWAKENS.md
   19  runtime/d3d12/src/device/compute.rs
   19  runtime/xg/src/lib.rs
   18  runtime/d3d12/src/device/views.rs
   17  tuxbox.py
   17  runtime/xmem/guest.c
   17  tools/reference/bvh-node/native/README.md
   16  tools/reference/bvh-node/native/probe.c
   16  project_tuxbox/progress_map.py
   16  project_tuxbox/host.py
   16  examples/lanes/README.md
   16  tests/test_runtime_shader_links.py
   15  docs/LIBRARIES.md
   15  tools/reference/bvh-node/native/probe.cl
   15  tests/test_host.py
```

## Merges (primeiros 80)
```
1b32189|2026-11-25|Merge pull request #190 from smith-3/lanes-gpu-indirect
552e099|2026-11-25|Merge pull request #189 from gtb-sbe/evidence-nixos-rx9060xt
b5ed526|2026-11-25|Merge pull request #188 from Vladimir-Human/d3d12/castable-packed-uavs
f489d3c|2026-11-25|Merge pull request #184 from DrArroway/fix/radv-inline-descriptor-stride
4ba74bc|2026-11-25|Merge pull request #187 from smith-3/codex/lanes-direct-copy-regression
3e6deed|2026-11-25|Merge pull request #186 from Vladimir-Human/d3d12/fix-memory-wait-stalls
22a0cdf|2026-11-25|Merge branch 'brcly:main' into evidence-rx9060xt
4cb6a4a|2026-11-25|Merge pull request #183 from smith-3/codex/lanes-post-dispatch-wait
1f40ee4|2026-11-25|Merge pull request #176 from Vladimir-Human/shader/exec-group-wave32-intrinsics
a9b9715|2026-11-25|Merge pull request #174 from JeffreyBytes/feat/omitted-static-samplers
82d3467|2026-11-25|Merge pull request #173 from JeffreyBytes/feat/vertex-table-parameter
cd70848|2026-11-25|Merge pull request #177 from 3kynox/pr-mesh-pixel-layout
fde44ae|2026-11-25|Merge pull request #179 from smith-3/codex/lanes-cross-queue-return
d7b42f4|2026-11-25|Merge pull request #178 from 3kynox/pr-reclaim-selftest-limit
7b22d71|2026-11-25|Merge pull request #175 from NathansDomainSolutions/fix/xmem-legacy-mapped-writes
9ca0060|2026-11-25|Merge pull request #172 from Vladimir-Human/d3d12/reclaim-descriptor-slots
1d821cb|2026-11-25|Merge pull request #61 from Valdi320/work/scarlett-observability
3bd0402|2026-11-25|Merge pull request #60 from Valdi320/work/scarlett-cpu-oracle
bca2514|2026-11-25|Merge pull request #74 from DrArroway/evidence/cachyos-rx6900xt
8fe20cc|2026-11-25|Merge pull request #166 from smith-3/codex/shader-integration-20261125
de47600|2026-11-25|Merge pull request #168 from postblink/fix/raster-multi-table-roots
0eaa8c6|2026-11-25|Merge pull request #170 from Vladimir-Human/d3d12/slot80-thread-dimensions
3aa9241|2026-11-25|Merge pull request #171 from Vladimir-Human/shader/wave32-wave64-diagnostics
94412a5|2026-11-25|Merge pull request #167 from Valdi320/work/scarlett-pipeline-rank-accuracy-20261125
daccc40|2026-11-25|Merge pull request #169 from Vladimir-Human/build/msvc-w4-guest-narrowing
5dd0d84|2026-11-25|Merge pull request #165 from Jerkin42/fix/cache-lookups
824b866|2026-11-25|Merge pull request #41 from ReverendOzmund/bazzite-nvidia-results
529c52d|2026-11-25|Merge pull request #163 from NathansDomainSolutions/fix/mesh-mirror-all-pixel-tables
63bec84|2026-11-25|Merge pull request #162 from Jerkin42/fix/d16-depth-uav
dfe7813|2026-11-25|Merge pull request #161 from Jerkin42/fix/dispatch-barrier
1d4a55c|2026-11-25|Merge pull request #160 from smith-3/codex/shader-corrections-handoff
6fbfb48|2026-11-25|Merge pull request #159 from s3rius/xshader/wave_divergent_scc
fcd74ae|2026-11-25|Merge pull request #134 from smith-3/codex/persistent-dxc
e8898ce|2026-11-25|Merge main and preserve validated CLI fallback for persistent DXC
5678a2f|2026-11-25|Merge pull request #144 from s3rius/xshader/wave_divergent
cc9f44b|2026-11-25|Merge pull request #158 from NathansDomainSolutions/fix/mesh-pixel-table-copy
aad0ecd|2026-11-25|Merge pull request #157 from NathansDomainSolutions/fix/rooted-pixel-sampler-heap
8ce988d|2026-11-25|Merge pull request #156 from ReverendOzmund/d3d12-slot148-list-memory
62e74e1|2026-11-25|Merge pull request #155 from 3kynox/pr-vulkan-headers
20fc550|2026-11-25|Merge pull request #154 from 3kynox/pr-nvidia-cache
7ef61b9|2026-11-25|Merge pull request #153 from 3kynox/pr-insertq
53bb4d6|2026-11-25|Merge pull request #150 from NathansDomainSolutions/fix/mesh-pixel-link
0a8d2ee|2026-11-25|Merge pull request #149 from nztom/fix/xshader-compute-wave-rejoin
b1e30cd|2026-11-25|xshader: integrate bounded reductions with upstream EXECZ votes
78c9017|2026-11-25|Merge pull request #139 from ReverendOzmund/evidence-rtx3080ti
6b67fac|2026-11-25|Merge pull request #148 from JeffreyBytes/fix/page-import-reuse
a70399f|2026-11-25|Merge pull request #146 from JeffreyBytes/fix/amd-only-cpu-instructions
9a9775b|2026-11-25|Merge pull request #143 from nztom/fix/xshader-compute-wave-execz
02f3f75|2026-11-25|Merge pull request #142 from smith-3/codex/dpp16-bitwise-quad-perm
408f876|2026-11-25|Merge pull request #141 from nztom/perf/xshader-large-pixel-dxc
d665f54|2026-11-25|Merge pull request #140 from nztom/fix/xshader-nsa-wide-atomic
60af1db|2026-11-25|Merge pull request #138 from smith-3/codex/sad-u8-zero
299d4b0|2026-11-25|Merge pull request #137 from smith-3/codex/mul-lo-u16-inline
a62839c|2026-11-25|Merge pull request #132 from NathansDomainSolutions/fix/descriptor-retire-grace
9967792|2026-11-25|Merge pull request #136 from NathansDomainSolutions/feat/physical-page-reads
a6f3289|2026-11-25|Merge pull request #135 from NathansDomainSolutions/fix/pixel-heap-and-block-quads
335a343|2026-11-25|Merge pull request #125 from smith-3/codex/shader-regression
fc25aea|2026-11-25|Merge pull request #121 from vladbondarenko97/tests-resolve-tmp-macos
573c2cf|2026-11-25|Merge pull request #131 from absceptual/d3d12/mask-console-resource-flags
704d331|2026-11-25|Merge pull request #133 from NathansDomainSolutions/fix/pixel-quad-exchange-loops
83cf1a5|2026-11-25|Merge pull request #128 from absceptual/xshader/exec-group-all-stages
ccce179|2026-11-25|Merge pull request #130 from s3rius/xshader/lane_spills
4046aaa|2026-11-25|Merge pull request #129 from absceptual/scarlett/cached-refusal-reasons
91cfdbe|2026-11-25|Merge pull request #126 from NathansDomainSolutions/fix/graphics-menu-eof
62dfa31|2026-11-25|Merge pull request #124 from postblink/fix/lanes-input-first-key
c12deae|2026-11-25|Merge pull request #123 from s3rius/feature/wave_sum
8e01961|2026-11-25|Merge pull request #122 from Valdi320/work/scarlett-large-next
20d8280|2026-11-25|Merge pull request #119 from Valdi320/work/scarlett-slot86-followup
ca9b143|2026-11-25|Merge pull request #120 from Valdi320/work/scarlett-lane-exact-next
465318f|2026-11-24|Merge pull request #118 from postblink/test/lanes-input-gate
ebe62d0|2026-11-24|Merge pull request #117 from Valdi320/work/scarlett-link-validation
8dfe439|2026-11-24|Merge pull request #114 from Valdi320/work/scarlett-root-cbv-window-validation
6f5af52|2026-11-25|Merge upstream strict regression controls and extend CBV coverage
9139f2f|2026-11-24|Merge pull request #92 from Valdi320/work/scarlett-strict-gpu-regression
4bce37a|2026-11-24|Merge pull request #115 from NathansDomainSolutions/speedup/prep-jobs-and-pack-seed
2e96986|2026-11-24|Merge pull request #113 from NathansDomainSolutions/perf/xcrdapi-parallel-reads
329ab83|2026-11-24|Merge pull request #112 from NathansDomainSolutions/feat/xsystem-app-device-id
a25ee08|2026-11-24|Merge branch 'main' into work/scarlett-strict-gpu-regression
8661492|2026-11-24|Merge pull request #111 from Valdi320/work/scarlett-heap-cfg-provenance
3ad360e|2026-11-24|Merge pull request #110 from s3rius/feature/evidence-arch-rtx5090
```

## Branches remotas nao mergeadas
```
origin/ci/build-guests
```

## Reverts explicitos
```

```

## Candidatos a correcao
```
f489d3c|2026-11-25|brcly|Merge pull request #184 from DrArroway/fix/radv-inline-descriptor-stride
4ba74bc|2026-11-25|brcly|Merge pull request #187 from smith-3/codex/lanes-direct-copy-regression
3e6deed|2026-11-25|brcly|Merge pull request #186 from Vladimir-Human/d3d12/fix-memory-wait-stalls
91a9de2|2026-11-25|Vladimir|d3d12: fix audit findings for R9G9B9E5 UAV interop, native gating, and buffer UAVs
6bae710|2026-11-25|Vladimir|d3d12: resolve GPU memory-wait stalls across all comparisons, queue types, and submissions
4b56c70|2026-11-25|Jeffrey C|xshader: take a vertex shader's table parameter from its own records
7b22d71|2026-11-25|brcly|Merge pull request #175 from NathansDomainSolutions/fix/xmem-legacy-mapped-writes
de47600|2026-11-25|brcly|Merge pull request #168 from postblink/fix/raster-multi-table-roots
940ab23|2026-11-25|brcly|lanes: time frames and the shader cache lookup, run the regression controls
7f8d1ad|2026-11-25|brcly|tools: accept import reuse records in the D3D12 regression controls
a7431e3|2026-11-25|Vladimir|build, verify: fix MSVC /W4 warnings and resolve ordinal imports
52292e6|2026-11-25|brcly|lanes: add scenarios and a benchmark, and check every guest suite
4ae23ab|2026-11-25|brcly|lanes: add a GPU self-test, shader cache checks and a single check command
5dd0d84|2026-11-25|brcly|Merge pull request #165 from Jerkin42/fix/cache-lookups
529c52d|2026-11-25|brcly|Merge pull request #163 from NathansDomainSolutions/fix/mesh-mirror-all-pixel-tables
63bec84|2026-11-25|brcly|Merge pull request #162 from Jerkin42/fix/d16-depth-uav
dfe7813|2026-11-25|brcly|Merge pull request #161 from Jerkin42/fix/dispatch-barrier
1d4a55c|2026-11-25|brcly|Merge pull request #160 from smith-3/codex/shader-corrections-handoff
cc9f44b|2026-11-25|brcly|Merge pull request #158 from NathansDomainSolutions/fix/mesh-pixel-table-copy
aad0ecd|2026-11-25|brcly|Merge pull request #157 from NathansDomainSolutions/fix/rooted-pixel-sampler-heap
71c7c85|2026-11-25|NathansDomainSolutions|xshader: accept the sampler heap base in rooted pixel shaders
53bb4d6|2026-11-25|brcly|Merge pull request #150 from NathansDomainSolutions/fix/mesh-pixel-link
e8af9fd|2026-11-25|brcly|d3d12: record title fence writes and stop one command failing a whole list
0a8d2ee|2026-11-25|brcly|Merge pull request #149 from nztom/fix/xshader-compute-wave-rejoin
6b67fac|2026-11-25|brcly|Merge pull request #148 from JeffreyBytes/fix/page-import-reuse
a70399f|2026-11-25|brcly|Merge pull request #146 from JeffreyBytes/fix/amd-only-cpu-instructions
9a9775b|2026-11-25|brcly|Merge pull request #143 from nztom/fix/xshader-compute-wave-execz
fd561df|2026-11-25|Jeffrey C|d3d12: reuse live imports that span several pages
d665f54|2026-11-25|brcly|Merge pull request #140 from nztom/fix/xshader-nsa-wide-atomic
a62839c|2026-11-25|brcly|Merge pull request #132 from NathansDomainSolutions/fix/descriptor-retire-grace
a30aa2a|2026-11-25|ReverendOzmund|evidence: ProjectScarlett reaches Game Running on an RTX 3080 Ti
a6f3289|2026-11-25|brcly|Merge pull request #135 from NathansDomainSolutions/fix/pixel-heap-and-block-quads
335a343|2026-11-25|brcly|Merge pull request #125 from smith-3/codex/shader-regression
704d331|2026-11-25|brcly|Merge pull request #133 from NathansDomainSolutions/fix/pixel-quad-exchange-loops
91cfdbe|2026-11-25|brcly|Merge pull request #126 from NathansDomainSolutions/fix/graphics-menu-eof
62dfa31|2026-11-25|brcly|Merge pull request #124 from postblink/fix/lanes-input-first-key
5213877|2026-11-24|Lucas Boyles|tools: check Lanes keyboard play through XTest on any course
6f5af52|2026-11-25|Valdi320|Merge upstream strict regression controls and extend CBV coverage
9139f2f|2026-11-24|brcly|Merge pull request #92 from Valdi320/work/scarlett-strict-gpu-regression
53bf140|2026-11-24|Claude|Speed up ProjectScarlett shader preparation
8bd56af|2026-11-24|brcly|test: repair the GPU regression checker after the main merge
7fa2c83|2026-11-24|NathansDomainSolutions|xgameruntime: serve XSystemGetAppSpecificDeviceId (XSystem slot 5)
a25ee08|2026-11-24|brcly|Merge branch 'main' into work/scarlett-strict-gpu-regression
ade292d|2026-11-24|brcly|Merge pull request #109 from davutsimsekk/fix/windows-python-tests
5a67cef|2026-11-24|brcly|Merge pull request #107 from NathansDomainSolutions/fix/root-cbv-64k-window
854923d|2026-11-24|brcly|Merge pull request #105 from s3rius/bugfix/wayland-displays
1b9df5d|2026-11-24|Pavel Kirilin|Fix private Xwayland display collision and guest-process cleanup.
bd7108c|2026-11-24|Jerkin42|xshader: give the f64 fixture the new fields
cb185e0|2026-11-24|brcly|d3d12: degrade placed-texture aliasing instead of failing the queue
919f148|2026-11-24|brcly|Merge pull request #62 from Valdi320/work/scarlett-resource-analysis
d2fa361|2026-11-24|brcly|Merge pull request #81 from chunfeng11221/build/native-msvc
f057e9b|2026-11-24|brcly|Merge pull request #96 from NathansDomainSolutions/fix/interpolant-component-types
81d4a92|2026-11-24|brcly|Merge pull request #97 from NathansDomainSolutions/fix/dxc-o0-timeout
2ce238c|2026-11-24|Valdi320|test: disable null fallback in positive GPU regression controls
5cee10f|2026-11-24|brcly|Merge pull request #88 from Valdi320/work/scarlett-null-usage-fixture
b9e42ec|2026-11-24|brcly|Merge pull request #85 from Valdi320/work/scarlett-autonomous-validation
0e762f9|2026-11-24|Valdi320|shader: fix packed signed saturation in dependent GPU chains
621d6da|2026-11-24|brcly|Merge pull request #59 from trigger337/fix/staging-descriptor-cleanup
3d9185a|2026-11-24|brcly|Merge pull request #82 from Valdi320/work/scarlett-runtime-bottlenecks
5d951b2|2026-11-25|chunfeng11221|build: fix native Windows MSVC link forwarders and /W4 /WX warnings
326c500|2026-11-24|Valdi320|scarlett: distinguish twelve additional fixed refusal families
00f69b8|2026-11-24|brcly|d3d12: record the command-list slots shared by both Xbox generations
e6ca8f2|2026-11-24|Valdi320|test: preserve minimized dependent-scalar regression cases
0d08d73|2026-11-24|Valdi320|tools: analyze memory retention and paired performance regressions
5f777d6|2026-11-24|brcly|Merge pull request #42 from Jerkin42/fix/cbv-descriptor-buffer-per-gpu
306344e|2026-11-24|brcly|Merge pull request #40 from trigger337/fix/shader-profile-whitespace
e3ed1a0|2026-11-24|Valdi320|scarlett: turn shader comparisons into prioritized replay and regression worklists
72766e7|2026-11-24|Jerkin42|host: hide VK_EXT_descriptor_buffer only on GPUs that need it
832d0aa|2026-11-24|Andrey|runtime_shaders: strip whitespace from profile output before validation
dc0b8c1|2026-11-24|brcly|Merge pull request #35 from postblink/fix/present-rootless-xwayland
04641a6|2026-11-24|Lucas Boyles|build: declare the Rust 1.95 floor in the workspace
d10b369|2026-11-24|brcly|Fix linux shader caching issues
9b18f7c|2026-11-24|brcly|launcher: keep each prefix's NVIDIA shader cache
21c8d05|2026-11-24|brcly|Merge pull request #33 from NathansDomainSolutions/fix/heap-sampler-hardening
c418316|2026-11-23|NathansDomainSolutions|xshader, triage: harden the run-time sampler-heap fetch and the triage CLI
c90ff8e|2026-11-24|brcly|Merge pull request #31 from NathansDomainSolutions/fix/sload-dynamic-soffset
ebd3022|2026-11-23|NathansDomainSolutions|xshader: fetch compute samplers from the sampler heap at a run-time offset
d73502d|2026-11-24|brcly|Merge pull request #30 from NathansDomainSolutions/fix/scarlett-samplers-raster
4afc67d|2026-11-23|NathansDomainSolutions|xshader: MUBUF SOFFSET as an address term; raw SRVs behind root descriptors
f8abe8f|2026-11-23|NathansDomainSolutions|xshader: private samplers when the root declares no static samplers
243b695|2026-11-23|NathansDomainSolutions|xshader: FSSM fallback for uncovered raster slots; private root-less samplers
1233e21|2026-11-23|NathansDomainSolutions|xshader: match literal sampler words in any shader with static samplers
dd491c6|2026-11-24|brcly|Merge pull request #26 from NathansDomainSolutions/fix/raster-s9-diagnostics
b8ee94a|2026-11-24|brcly|Merge pull request #27 from NathansDomainSolutions/feat/offline-launch
2bad2b7|2026-11-24|brcly|Merge pull request #25 from NathansDomainSolutions/fix/rage-shader-translation
cc752ba|2026-11-23|NathansDomainSolutions|xshader: translate image_gather4_l, resolve root-less descriptors, and handle typed heap stores
78be8ab|2026-11-24|brcly|Merge pull request #23 from NathansDomainSolutions/fix/dxc-timeout-fallback
ca20f93|2026-11-23|NathansDomainSolutions|docs: add a ProjectScarlett setup guide
d130bf1|2026-11-23|NathansDomainSolutions|launcher: add run --offline to block crash uploads on Linux
f22f388|2026-11-24|brcly|Merge pull request #22 from NathansDomainSolutions/fix/proton-runner-symlink
18e81b5|2026-11-24|brcly|Merge pull request #21 from NathansDomainSolutions/fix/pipeline-trace-events
e360e3e|2026-11-24|brcly|Merge pull request #20 from NathansDomainSolutions/fix/package-drive-launch-cleanup
9e2fd66|2026-11-24|brcly|Merge pull request #18 from ReverendOzmund/fix-frame-present-tests
40e1565|2026-11-24|ReverendOzmund|Fix frame tests: present probe crash and clang 19 RGB10 clear
7666f54|2026-11-24|brcly|Merge pull request #17 from NathansDomainSolutions/fix/pipeline-big-stack
1a9929a|2026-11-24|brcly|Merge pull request #11 from NathansDomainSolutions/fix/d3d12-failure-diagnostics
faa99ae|2026-11-24|brcly|Merge pull request #10 from NathansDomainSolutions/fix/xgameruntime-query-log
c7487e4|2026-11-24|brcly|Merge pull request #9 from NathansDomainSolutions/fix/cached-pages-imports
ff7fd07|2026-11-24|brcly|Merge pull request #6 from NathansDomainSolutions/fix/package-drive
05c7e3e|2026-11-24|brcly|Merge pull request #15 from ReverendOzmund/fix-linux-build-current-toolchains
cc9d34d|2026-11-24|brcly|Add an opt-in Project Scarlett graphics profile
444066e|2026-11-24|brcly|Lower mesh resource-heap loads and whole-wave allocations
ca772f1|2026-11-24|ReverendOzmund|Fix Linux build with current Rust and clang
2cc9ff3|2026-11-23|NathansDomainSolutions|Fix check_frosty_cached_pages imports
c561d03|2026-11-23|NathansDomainSolutions|Find the package root from the launch and restore G: afterwards
9a81229|2026-11-24|brcly|Translate SM 6.6 descriptor-heap resources in compute shaders
84c35b3|2026-11-23|NathansDomainSolutions|Map the package to G: for Wine and Proton launches
baa315b|2026-11-23|NathansDomainSolutions|Link synchronization.lib and reject broken DXC builds on Linux
fe6b0e1|2026-11-23|brcly|Bring ProjectScarlett from Game Init to Game Running
57fe562|2026-11-23|brcly|Merge pull request #3 from Batushn/fix/windows-unittest-compatibility
91f775d|2026-11-23|brcly|Merge pull request #2 from Batushn/fix/inspect-resolve-root
7d8fe16|2026-11-22|brcly|Validate hair prefix through Xbox provider
```
