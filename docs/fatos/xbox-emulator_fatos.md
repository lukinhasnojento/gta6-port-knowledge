# FATOS VERIFICADOS - xbox-emulator

Gerado por git puro, sem LLM.

## A. Identificacao
- Nome: xbox-emulator
- URL: https://github.com/omonkeyz/xbox-emulator
- Branch analisada: main
- Raiz (commit 0): 2b4bba2ca379068c992038a800d4ae37fc7f38c6
- Primeiro commit: 2b4bba2ca379068c992038a800d4ae37fc7f38c6 | 2026-11-21 02:08:33 -0600 | omonkey | init: progress tracker for TomBox + rage9
- Ultimo commit: a618736df799d21d9a565d08893355fc3c96fdf5 | 2026-11-25 07:10:51 -0600 | omonkey | Merge #347: tombox-xshader: size structured buffer elements from the FSSM stride (stacked on #289)
- Periodo: 2026-11-21 a 2026-11-25 (5 dias com commit)
- Commits em HEAD: 595
- Commits em todas as refs: 595
- Commits so em outras branches: 0
- Arquivos versionados: 1652
- Merges: 104

## Autores
```
   338  omonkey <246595960+omonkeyz@users.noreply.github.com>
    46  GDAgent Team <gdagent@protonmail.com>
    43  TQuentinD <tobestyledintro@gmail.com>
    35  azerioxfr <130263862+azerioxfr@users.noreply.github.com>
    20  alpha <anotheraxiomlegalteam@gmail.com>
    19  MeNoSkills <359902512+MeNoSkills@users.noreply.github.com>
    16  Isaac Wood <isaacwood100197@gmail.com>
    11  Michael Kelly <thatirishguy0@gmail.com>
     8  BED <73585863+yeetbot90@users.noreply.github.com>
     7  Lex_021 <pikadasrt@gmail.com>
     6  enderrr9 <178652333+enderrr9@users.noreply.github.com>
     5  Dea7hsmiles <188018312+dea7hsmiless@users.noreply.github.com>
     5  adriaespii <162989435+adriaespii@users.noreply.github.com>
     4  Lex_021 <lex021@users.noreply.github.com>
     3  nickfmbg <nikolozizarkua@gmail.com>
```

## Commits por dia
```
2026-11-21    128
2026-11-22    175
2026-11-23    212
2026-11-24     20
2026-11-25     60
```

## 30 arquivos mais alterados
```
  113  docs/BLOCKERS.md
  108  crates/tombox-xshader/src/hlsl.rs
   77  tools/progress/progress.json
   71  docs/img/progress.svg
   70  docs/LIBRARY_MAP.md
   69  tools/progress/summary.json
   49  docs/MILESTONES.md
   49  crates/tombox-xshader/tests/exec.rs
   44  docs/img/badge-console.svg
   32  crates/compat/d3d12_xs/src/device_wrap.rs
   31  crates/tombox-xshader/src/rdna2/mnem.rs
   30  tools/progress/data/emulator_exports.json
   29  crates/tombox-xshader/tests/synthetic.rs
   27  THIRD_PARTY/CREDITS
   26  crates/compat/d3d12_xs/src/lib.rs
   24  docs/img/badge-d3d12x.svg
   21  crates/tombox-xshader/tests/exec_mem/mod.rs
   21  crates/compat/d3d12_xs/src/shader_xlat.rs
   21  crates/tombox-xshader/src/lib.rs
   21  tools/progress/gen.py
   19  crates/tombox-xshader/tests/exec_mem/mimg.rs
   19  crates/compat/d3d12_xs/src/present.rs
   18  docs/img/badge-instructions.svg
   17  crates/compat/d3d12_xs/src/upload.rs
   17  Cargo.lock
   17  crates/tombox-xshader/tests/mesh.rs
   16  crates/compat/xg_xs/src/dcc.rs
   16  crates/compat/compat-common/src/gdk/rt.rs
   15  crates/tombox-xshader/tests/opcodes.rs
   15  crates/tombox-xshader/src/gcn.rs
```

## Merges (primeiros 80)
```
b17ecef|2026-11-23|Merge #289: tombox-xshader: read the FSSM root-binding table (opt-in fallback for shaders without RTS0)
343c15a|2026-11-23|Merge #278: tombox-xshader: seed merged_wave_info, vertex and instance index for merged vertex shaders
e543de8|2026-11-23|Merge #199: tombox-xshader: declare UAVs accessed with GLC or DLC globallycoherent
a1d04e8|2026-11-23|Merge #245: tombox-xshader: decode and translate ds_wrap_rtn_b32 on compute LDS
00052ef|2026-11-23|Merge #240: tombox-xshader: decode and translate ds_read_b96, ds_read_b128, ds_wrÔÇª
ecc06dd|2026-11-23|Merge #258: tombox-xshader: translate s_buffer_store_dword* into UAV writes
df310a4|2026-11-23|Merge #254: tombox-xshader: name and lower s_dcache_wb as a no-op
4f61662|2026-11-23|Merge #183: xshader: resolve RDNA2 literal static samplers
19d0be4|2026-11-23|Merge #253: tombox-xshader: decode and translate the VOP3-encoded v_interp f32 forms
9e731d3|2026-11-23|Merge #269: tombox-xshader tests: pin the mesh export, entry-count and LDS refusals
0644a7a|2026-11-23|Merge #261: tombox-xshader tests: pin the export-target refusals
c06a3f7|2026-11-23|Merge #243: tombox-xshader tests: pin the ds_swizzle stage and QDMode refusals
eb557e5|2026-11-23|Merge #233: tombox-xshader tests: pin the s_sendmsghalt and ds_ordered_count refusals
6d86929|2026-11-23|Merge #232: tombox-xshader tests: pin the refusal of GS emit and cut on streams 1-3
9f353b9|2026-11-23|Merge #287: tombox-xshader tests: bit-exact v_interp_p1_f32, v_interp_p2_f32 and v_interp_mov_f32 through a real vertex shader
4f1cd12|2026-11-23|Merge #284: tombox-xshader: lower scratch (private memory) loads and stores to a per-thread array
be98784|2026-11-23|Merge #286: tombox-xshader tests: bit-exact pixel sampling cases for image_sample, _b, _d, _c_lz, _lz_o, gather4_lz and image_load_mip
b946ac8|2026-11-23|Merge #256: tombox-xshader tests: pin the multisampled UAV load refusal
1a4745a|2026-11-23|Merge #255: tombox-xshader tests: pin the image-dimension refusals
de4317e|2026-11-23|Merge #248: tombox-xshader tests: pin the gather4 bias and dmask refusals
e65f9ce|2026-11-23|Merge #282: tombox-xshader: decode and translate image_sample_d_g16 and image_samÔÇª
0164cc3|2026-11-23|Merge #265: xshader: DXIL container digest in Rust, checked against DXC and the D3D12 runtime
6361ba0|2026-11-23|Merge #264: tombox-xshader: decode and lower the DS src2 read-modify-write ops
8b15fe5|2026-11-23|Merge #263: tombox-xshader: decode and translate buffer_load/store_format_d16_x and _d16_hi_x
b24ac90|2026-11-23|Merge #281: tombox-xshader: lower the GCN f32 clamp transcendental VOP1 ops
ebc3148|2026-11-23|Merge #272: tombox-xshader: decode and translate the f32 IEEE division helpers
2e7f5c3|2026-11-23|Merge #285: tombox-xshader: saturate the 24-bit multiplies on VOP3 clamp
e4c1b20|2026-11-23|Merge #267: xshader: v_pk_add/sub/mad_i16 with CLAMP use sat_int (wrong high lane on an NVIDIA RTX 5090)
3b3028b|2026-11-23|Merge #249: tombox-xshader: decode and translate the FLAT/GLOBAL D16 sub-dword loads
0447a97|2026-11-23|Merge #242: tombox-xshader: decode the remaining SOP1 saveexec ops and translate s_andn1/s_andn2_wrexec
380f755|2026-11-23|Merge #252: tombox-xshader: decode and translate v_movrelsd_2_b32 and v_swaprel_b32
74ea579|2026-11-23|Merge #247: tombox-xshader: decode and translate v_qsad_pk_u16_u8, v_mqsad_pk_u16_u8 and v_mqsad_u32_u8
dd1ca11|2026-11-23|Merge #237: xshader: v_mad_u64_u32 and v_mad_i64_i32 write 0 to sdst for a lane off in EXEC
9d565d6|2026-11-23|Merge #200: xshader mesh: full merged_wave_info in s3, GS_ALLOC_REQ field widths per the ISA (was #58)
2014b35|2026-11-23|Merge #280: d3d12_xs: rect-list pipelines from PSO streams: map console topology type 0x13 and expand the draws to full rects
0cc4e3b|2026-11-23|Merge #283: d3d12_xs: drop gpu_va ranges when their resource is destroyed
89d07c1|2026-11-23|Merge #273: d3d12_xs: name the group counts of each Dispatch in the TOMBOX_DRED=2 report
ab25c62|2026-11-23|Merge #279: gdk tests: 12 interface ids the PC runtime serves and the emulator refuses, one method table per family on the PC
139f94c|2026-11-23|Merge #210: gdk rt: XAsyncGetStatus(wait) waits for the completion callback, as the PC runtime does
d39a963|2026-11-23|Merge #270: gdk rt: XTaskQueueRegisterWaiter and UnregisterWaiter, two-phase queue termination (5 conformance scenarios)
8b16440|2026-11-23|Merge #250: pixrt10: initialize the build 19041 title runtime and serve XBTP queries
13542f0|2026-11-23|Merge #238: Support Windows I/O completion ports on in-process XBTP links
dfaf4d5|2026-11-23|Merge #236: Port build 19041 XMemVirtualQuery partition and page classification
bb15e8e|2026-11-23|Merge #244: tombox-archive: block colliding entry names
32098a1|2026-11-23|Merge #229: Implement the build 19041 xvcmgmt four-counter query
61ffabe|2026-11-23|Merge #227: Respect the console build override for XMem pools and allocation statistics
c40dd17|2026-11-23|Merge #276: d3d12_xs tests: three --lib tests that fail or crash on a release build (dangling temporary, function address, stale map entry)
20f9340|2026-11-23|Merge remote-tracking branch 'origin/main'
2d4cbd5|2026-11-23|Merge #274 and #275: BLOCKERS updates for the GWS counter ops and the write-watch submit hook
19806e8|2026-11-23|Merge commit 'e3c62b71bb8229be43029daca1569316cacb2220'
e3c62b7|2026-11-23|Merge #266: XAsync return codes match the reference
d8d37e4|2026-11-23|Record #235 authorship (MUBUF D16 byte/short loads and d16_hi stores)
1172d5d|2026-11-23|Merge commit '6e81ffc4a05788a84c29e589370020573d645862'
6e81ffc|2026-11-23|Merge #259 and #260: BLOCKERS updates for the run-35 system-function list and tests/amp.rs
df8e0c0|2026-11-23|Merge commit '902209ac26cbacae18d4c96da4d8a29d3ffc03bd'
d66ae9c|2026-11-23|Merge commit '451fe119fa8b6e921b889cc44e3d77754d27995a'
451fe11|2026-11-23|Record the r32g batch authorship (#221, #223, #225, XGameSave slot logging)
4ebf862|2026-11-23|Merge #222: decode stream-zero OSG5 output signatures on both Xbox shader paths
ad0fbf3|2026-11-23|Merge commit 'bfd70bedcecef75b0b315cf4a533c7f418ec85b7'
bfd70be|2026-11-23|Merge #228, #230, #231: progress mnemonics, tombox-archive zero-compressed warning, BLOCKERS xdxr refusals fixed
b7f137f|2026-11-23|Merge #224, #226: pr_gate checks ++ lines, packed int16 mul/shift/minmax exec test; BLOCKERS flaky alias test
1dc9fff|2026-11-22|Merge #215, #217, #218, #219, #220: CI exec shards with rust-core gate, XMem 19041 fixes, CONTRIBUTING statuses
f6ae886|2026-11-22|Merge commit 'c998b447438e3b6c9fbad8f5f1d75da575ce752f'
c998b44|2026-11-22|Merge port/leo-pk16: packed int16 VOP3P float constants and mad clamp (Leonida SHADER-302)
7e85f79|2026-11-22|Merge port/leo-resinfo: resinfo past the last level (Leonida SHADER-388)
e35aa5a|2026-11-22|Merge port/leo-f16omod: f16 output modifier follows MODE (Leonida port)
44597ab|2026-11-22|Merge #214: lower ds_mskor_b32 and ds_wrxchg2_rtn_b32 on compute LDS
2f1a94c|2026-11-22|Merge #213: v_mad_f32 flushes a subnormal result again after the output modifier
c16f0fa|2026-11-22|Merge #212: keep TOMBOX_DXC and TOMBOX_DXC_DIR for the nested DLL build in compat-common tests
a4c2355|2026-11-22|Merge #211: XTaskQueue / XAsync conformance scenarios as tests
ad24144|2026-11-22|Merge #205, #206, #207, #208: LDS D16 loads, signed SMEM offset, opt-in host pad input, valu_probe.py
24e6c10|2026-11-22|Merge #203, #202, #198, #201: v_med3_f32 NaN, f16 inline constants, VOP3P packed int16 ops, gfx1013/gfx7 opcode table check
2eda8b0|2026-11-22|Merge #196: translate image_gather4_l with an explicit LOD
9e62058|2026-11-22|Merge #197: compare f16 FMA NaN results as a class on hardware GPUs (fixes #194)
c4e72f0|2026-11-22|Merge #195 (SchmashinPrunes): replay RDNA2-recorded image atomic results
c2a3522|2026-11-22|Merge #173 (GDAgent Team): 64-bit SM 6.6 image atomics behind an OPTIONS9 gate
a4cc592|2026-11-22|Merge #189, #190 (MeNoSkills) and #191 (SchmashinPrunes): workspace check excludes data-file crates; FLAT offset width test; RDNA2 silicon replay tests
baadb80|2026-11-22|Merge brcly/TuxBox port: Xbox Series Device8 IID in the device wrapper (Apache-2.0)
42511d7|2026-11-22|Merge #184, #186 and #185 (MeNoSkills): ds_swizzle_b32 in compute (bit, quad, rotate, FFT); PC-relative far jumps as direct branches
f850f50|2026-11-22|Merge #180 (SchmashinPrunes) and the export-13 half of #181 (adriaespii): bit-exact RDNA2 transcendentals, unmapped position export 13
```

## Branches remotas nao mergeadas
```

```

## Reverts explicitos
```

```

## Candidatos a correcao
```
e7377c9|2026-11-23|TQuentinD|gdk rt: XAsyncCancel leaves completion to the provider; GetResult errors end the call; never-started blocks
ba8e0e5|2026-11-25|omonkey|Merge #307: gdk rt: fix XTaskQueue handle duplication, pool dispatch, monitors, close, and manual terminate wakeup (6 conformance scenarios)
adef185|2026-11-23|Bexx1107|gdk rt: fix XTaskQueue handle duplication, pool dispatch, monitors, close and manual terminate wakeup
fb95d74|2026-11-25|omonkey|Merge #300: gdk: XSystem and XGame title id conformance against the PC runtime (17 scenarios, 3 fixes)
acfede3|2026-11-23|TQuentinD|gdk tests: XSystem and XGame title id conformance against the PC runtime (17 scenarios)
4c7c544|2026-11-25|omonkey|Merge #309: xshader: initial MODE from PGM_RSRC1, fixture scan for its FSSM location
753e851|2026-11-23|TQuentinD|xshader: RSRC2, descriptor and stage-slot probes in the PGM_RSRC1 fixture scan
8e7233d|2026-11-23|TQuentinD|xshader: fixture scan for the PGM_RSRC1 location in FSSM containers
bd185fa|2026-11-23|TQuentinD|xshader: initial MODE from PGM_RSRC1, reported as unknown when not located
8c481f4|2026-11-23|GDAgent Team|d3d12_xs: opt-in GPU-based validation with TOMBOX_D3D12_GBV=1
68f2cf2|2026-11-24|GDAgent Team|d3d12_xs: skip SIB call candidates that do not fit in the return-address window
397866d|2026-11-23|GDAgent Team|d3d12_xs tests: check every detiled element and time the 1 MB detile only in optimized builds
706a2b7|2026-11-23|Bexx1107|xg_xs: gate the console-oracle test modules behind an off-by-default feature
d0fabcb|2026-11-25|omonkey|Merge #375: xshader: guard f32 sub and f16 add/sub/mul against AMD host folding (fixes three #333 failures)
d2cc33e|2026-11-23|MeNoSkills|tombox-xshader: amplification payload byte/short stores were lost on hardware; edge-case AS tests
8f71091|2026-11-23|GDAgent Team|tombox-xshader: SOP1 ops with a 32-bit result of a 64-bit source write one SGPR
894e339|2026-11-25|omonkey|Merge #328: tombox-xshader: fix RDNA2 M0 (124) vs NULL (125) scalar destinations
9b7c93a|2026-11-24|ChAlanA|tombox-xshader: fix RDNA2 M0 (124) vs NULL (125) scalar destinations
3d90f6f|2026-11-23|enderrr9|tombox-xshader: amplification shaders always use the memory-based root ABI
b8432ab|2026-11-23|enderrr9|tombox-xshader tests: synthetic amplification + mesh pair rendered on WARP
d38321c|2026-11-25|omonkey|Merge #291: Fix integer vertex selection in v_interp_mov_f32
ea24558|2026-11-23|NemesisDawn|Fix integer vertex selection in v_interp_mov_f32
547e1f3|2026-11-23|TQuentinD|xshader lds64: COMPUTE_PGM_RSRC2 location keyed on the FSSM sub-version, LDS index wrap chosen at emission
2bb0192|2026-11-23|TQuentinD|xshader: 64-bit LDS atomics under a group lock, LDS sized from COMPUTE_PGM_RSRC2
0dcfef1|2026-11-23|azerioxfr|tombox-xshader tests: pin the export-target refusals
16c9ddf|2026-11-23|azerioxfr|tombox-xshader tests: pin the ds_swizzle stage and QDMode refusals
476dca4|2026-11-23|TQuentinD|xshader: DXIL container digest in Rust, checked against DXC and the D3D12 runtime
4169a9f|2026-11-23|azerioxfr|tombox-xshader: lower the GCN f32 clamp transcendental VOP1 ops
3826705|2026-11-23|azerioxfr|tombox-xshader: decode and translate the f32 IEEE division helpers
995ea5a|2026-11-23|GDAgent Team|d3d12_xs: map the Xbox rect-list primitive topology type 0x13 to TRIANGLE in non-mesh PSO streams
fa9f1ca|2026-11-23|GDAgent Team|d3d12_xs: drop gpu_va ranges when their resource is destroyed
c40dd17|2026-11-23|omonkey|Merge #276: d3d12_xs tests: three --lib tests that fail or crash on a release build (dangling temporary, function address, stale map entry)
619627b|2026-11-23|TQuentinD|d3d12_xs: check the wrapped heap-properties slot by behaviour, not address
f89e410|2026-11-23|omonkey|r32l: gpuprof 2 s interval cap + longest events + b014 draw args, passcap 3D/cb dumps, XUser change/device/audio registries with real unregister-wait, gdk_multi_dll test staging fix
de1e1da|2026-11-23|omonkey|r32j: per-kind GPU timing of injected and title non-draw commands, first-use log, per-mip pass capture, exact-start alias census, XAsync pending-Begin fix, full-boot harness offline handling
e67e3d4|2026-11-23|omonkey|r32h: residency cache, passcap ps_ms trigger, xmem fixes, title offsets moved to local JSON, MUBUF d16 (#235)
5544d41|2026-11-23|azerioxfr|BLOCKERS: tests/amp.rs is ignored without the console fixtures, the PSV0 v1 gap stays open
4e8fbbf|2026-11-23|azerioxfr|BLOCKERS: the run-35 system-function gap list is closed (671/671)
451fe11|2026-11-23|omonkey|Record the r32g batch authorship (#221, #223, #225, XGameSave slot logging)
f73fd27|2026-11-23|omonkey|BLOCKERS: debug-layer probe result (r32f_9), r32f Trackmania board and CB findings
bfd70be|2026-11-23|omonkey|Merge #228, #230, #231: progress mnemonics, tombox-archive zero-compressed warning, BLOCKERS xdxr refusals fixed
ed9863a|2026-11-23|azerioxfr|BLOCKERS: mark the xdxr ds_swizzle and SOFFSET atomic refusals fixed
1dc9fff|2026-11-22|omonkey|Merge #215, #217, #218, #219, #220: CI exec shards with rust-core gate, XMem 19041 fixes, CONTRIBUTING statuses
a7a98e0|2026-11-22|omonkey|r32c: Trackmania campaign scene, PSO root-signature fix, pso_lib final-root key, DXC 35% faster, xmem title ledger, milestones per title
3de1b1f|2026-11-22|omonkey|r31.1 hotfix: game-scan runtime prewarm opt-in (TOMBOX_GAME_SCAN_PREWARM=1); it exhausted Trackmania's title memory and aborted map load
a1ff5be|2026-11-22|omonkey|r31: Trackmania race renders by default (MSAA dim, Durango VS vertex id, FMASK identity, CBV copy default), mdbg exception cap fix, Terraria table-CBV raw views
2820575|2026-11-21|TQuentinD|xshader: check RDNA2 opcode tables against gfx1013 references; fix f16 FMA names
9e62058|2026-11-22|omonkey|Merge #197: compare f16 FMA NaN results as a class on hardware GPUs (fixes #194)
50a13be|2026-11-23|Isaac Wood|tombox-xshader tests: replay RDNA2-recorded image atomic results
267d08e|2026-11-23|GDAgent Team|d3d12_xs: probe OPTIONS9 only on objects that answer IID_ID3D12Device
59c2475|2026-11-22|Isaac Wood|RDNA2 silicon evidence for 16 held-back or unverified instructions
30a1b53|2026-11-22|omonkey|r29: removed-device guard, PSO prewarm + library save fixes, gpu_va nested ranges, netview/WFP + socket hooks, xmem sweep race fix, msimg32 test fix, res_pool cfg
a541bbb|2026-11-22|omonkey|compat-common os_host: pin modules during import sweeps (fix xmem+0x265e01 AV on module unload race)
12ba607|2026-11-22|omonkey|r28: mesh PSO topology from PSV0 + SUB_PRIMITIVE_TOPOLOGY 14 (#176 update); 16 more Xbox-vtable mis-slot fixes in injected compute paths (sys_ts, ts99, cl2_state predication); PSO library totals every 10 s, worker events on the title frame, 250 ms CSV flush
d759b31|2026-11-22|omonkey|r27: translator batch (#170 root-constant user SGPRs, #172 256-thread mesh, #176 mesh topology drop, #177 buffer-atomic SOFFSET); command-list PC-slot fixes (#175 + 37 more sites via cmdlist_wrap::pc_method); driver PSO library (TOMBOX_PSO_LIBRARY=0 off); Device 44 placement fixes (exact view copies, tiled MSAA aliasing); net_guard socket/WFP hooks; d3d12x 200/200
202c763|2026-11-22|omonkey|Merge #171 (MeNoSkills): enable experimental shader models before the first test device
fe391fe|2026-11-22|omonkey|Merge #153 (MeNoSkills): range-scoped fault guard for placed-texture reads, root fix for #110
5d5bc33|2026-11-22|MeNoSkills|d3d12_xs: guard reads of placed-texture memory with a fault-recovering call instead of copying (root fix for #110)
789e8fb|2026-11-22|omonkey|console libs 2143/2155: xtfdiaginfo, xtfeventserver, coniosrv, ntdll UMS, wldp, gamecore V7, audio position fix
2be7211|2026-11-22|omonkey|r24: edited shader variants via pack, prewarm priority-inversion fix, bind-wait default verified (0 skipped draws), cbv_raw CHECK/CLAMP/INHERIT diagnostics (env-gated), BLOCKERS: root CBV inheritance
595eb3a|2026-11-22|MeNoSkills|d3d12_xs: read placed-texture memory through fault-safe snapshots so a racing decommit skips the upload instead of crashing
f10db75|2026-11-22|omonkey|gcxaudioext: publish acp_state module (fixes clean-clone build E0583)
7ef29e7|2026-11-22|omonkey|d3d12_xs: PSO registry owns semantic names, drops CachedPSO, holds root signature (Fortnite derived PSO fix); exec mask fix; r21
c95a909|2026-11-22|omonkey|Exec mask: keep full exec, reduce only s_wqm writes to exec; Fortnite derived PSO use-after-free fix in register_pso
d585b94|2026-11-22|omonkey|Fix #109: recover poisoned shader_xlat mutexes; Fix #114: refuse MIMG R128/TFE/LWE instead of ignoring them; BLOCKERS notes for #104 #115
99f48e3|2026-11-22|omonkey|Merge contributor PRs #37 and #118 (partial), cherry-picked onto rewritten main
0b4583d|2026-11-21|Louisreed|fix(tombox-pe): bound RVA arithmetic in directory parsers to uphold the no-panic invariant
6afb54f|2026-11-21|TQuentinD|xshader: ISA-exact results on NVIDIA hardware (RTX 5090)
93861d6|2026-11-21|TQuentinD|tombox-xshader: wave32 lane ops address the guest half of a 64-lane host wave
0284e83|2026-11-21|omonkey|Sandbox: fix mid-path drive-letter escape in fs::resolve (console NT name rules, 50k fuzz); kernelbase family xref persisted to emulated HKU; passwordless uses caller token; hix_xs honest KMD results
9f82c41|2026-11-21|omonkey|Console libs: XbtpWrapDispatcher redirect port + SetSessionKey, xtfremoterun security fixes; xg_xs DCC back to stub (console output contains caller stack addresses); switch to bulk game testing
a852522|2026-11-21|omonkey|Console libs: xbtp sessions/dispatcher, xtfpix, xtfdebugmonitor, XtfCreateRemoteRunClient verified (cross-tested against console DLLs on loopback); XbtpWrapDispatcher and xtfremoterun #4 held
2850610|2026-11-21|omonkey|Console libs: 34 console-identical fixed-return rows verified (xcrdapi, mflnmhelp, mfhevcdec, advapi32legacy, pixevt, sc_dll, xbsc_xs, xbtp link, d3d12_x/xs DXGIX); scdxil loader-ref fix; xdxr unwind; gamecoreclient host replies
ba24a0b|2026-11-21|omonkey|Console libs: all 37 verifier defects now verified fixed (XTaskQueueTerminate round 3); gamecoreclient verified; sandbox registry/handle fixes
4ad72a4|2026-11-21|omonkey|Console libs: round-2 fixes verified (package, store, minuserx, d3d12_xs per-OS ABI, d3d12_x); XTaskQueueTerminate still held
f691d4a|2026-11-21|omonkey|Console libs: round-2 verifier fixes (task queue terminate on libHttpClient model, FindChunkAvailability, store license validity, minuserx index checks, d3d12_xs per-OS-build ABI, d3d12_x deserializer flags); gamecoreclient via xvcmgmt peer; labels: pass-throughs = forwarded
afffefd|2026-11-21|omonkey|Console libs: second verifier pass on defect fixes; MFGetPlatform verified; remaining failures held pending
7b2f86d|2026-11-21|omonkey|Console libs: 37 verifier defects fixed (task queue terminate, XVersion order, GDK error codes, D3D12 device wrap, front panel, minuserx A/W, xmem layout, memstatus); honest stub labels; XGEncodeDCC, MFGetPlatform, XVMAudioService classes; xbtp parallel tests; workspace builds again
de34ec8|2026-11-21|omonkey|Console libs verifier pass: 37 defects found in older rows; 15 title-used rows held pending fixes
c076007|2026-11-21|omonkey|Console libs: KernelTraceControl merge, hix_xs hang dump/debug forwards, xbtp sessions, LSA passwordless, wldp toast; 30 new exports held pending verification
1fc8683|2026-11-21|omonkey|Source sync: ISA sweep harness + 13 translator fixes, D3D12 slots 29/44/74/85/86/101 fixes and hardware tests, RDNA2 ds_append/consume, xg DCC decode
46d304d|2026-11-21|omonkey|D3D12: Trackmania placed-view device removal fixed
7423cfc|2026-11-21|omonkey|progress: Xbox One cmdlist slots, committed-resource and texture-VA mapping; BLOCKERS: Terraria texture-VA fixes, VS fetch-shader gap
3cbe510|2026-11-21|omonkey|progress: system imports 658/671 (MF evidence, XAudio2 ordinals, SILK fix)
f9a68fd|2026-11-21|omonkey|BLOCKERS: Trackmania presents frames (barrier fix), GCN decoder status, debug layer limits, Terraria frame events checked
a87d989|2026-11-21|omonkey|blockers: Trackmania packs fixed; now blocked on Xbox One shaders and command-list close
d9c6b87|2026-11-21|omonkey|progress: D3D12XboxSetProcessDebugFlags credited; MF tracer evidence path; GameChat2 title-DLL probe
234c512|2026-11-21|omonkey|progress: merge verified library work (xmem, d3d12 VM + device slots, rdna2 decoder fixes)
f4db225|2026-11-21|omonkey|status: login completes, hotfix succeeds, game-mode select reached
```
