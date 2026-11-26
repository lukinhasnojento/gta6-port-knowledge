# FATOS VERIFICADOS - OpenScarlett

Gerado por git puro, sem LLM.

## A. Identificacao
- Nome: OpenScarlett
- URL: https://github.com/californiaAlmonds/OpenScarlett
- Branch analisada: main
- Raiz (commit 0): ec4d6d389a1105e9cd7b6afab56ec1c48bb2d8dc
- Primeiro commit: ec4d6d389a1105e9cd7b6afab56ec1c48bb2d8dc | 2026-11-21 20:05:34 +0530 | californiaAlmonds | chore: initialize OpenScarlett MIT C++20 workspace and engineering guide
- Ultimo commit: 7220ce949800a959535086ce175b1e13b738aa16 | 2026-11-26 05:42:08 +0530 | californiaAlmonds | feat(gpu,d3d12_xs,loader): add v_cmp_class_f32/f16, Slot129 DispatchMesh, and watchdog/chunk guards
- Periodo: 2026-11-21 a 2026-11-26 (6 dias com commit)
- Commits em HEAD: 351
- Commits em todas as refs: 355
- Commits so em outras branches: 4
- Arquivos versionados: 545
- Merges: 101
- Dias com commit: 

## Autores
```
   341  californiaAlmonds <deeja.132024@gmail.com>
     2  marquezini <felipemarquesini10@gmail.com>
     2  californiaAlmonds <160736368+californiaAlmonds@users.noreply.github.com>
     2  Felipe Marquezini <felipemarquesini10@gmail.com>
     1  rpatty <108372609+rpatty-dev@users.noreply.github.com>
     1  aidenhrld <aidenharold@icloud.com>
     1  DeviousCardi <115358213+DeviousCardi@users.noreply.github.com>
     1  Ahmet Dedeler <105894172+Ahmet-Dedeler@users.noreply.github.com>
```

## Commits por dia
```
2026-11-21     61
2026-11-22    164
2026-11-23     96
2026-11-24     25
2026-11-25      1
2026-11-26      4
```

## 40 arquivos mais alterados
```
   56  README.md
   44  src/gpu/d3d12_xs/device_x.cpp
   44  CMakeLists.txt
   42  docs/status.svg
   36  src/loader/hook_dll.cpp
   35  src/gpu/d3d12_xs/command_queue_x.cpp
   31  src/shims/xgameruntime_shim.cpp
   30  src/gpu/d3d12_xs/device_x.h
   29  docs/index.html
   29  tools/generate_status.py
   28  src/gpu/d3d12_xs/command_list_x.cpp
   27  src/shims/d3d12_xs_shim.cpp
   27  src/gpu/rdna2_lifter.cpp
   22  src/gpu/rdna2_ir.cpp
   21  tools/run_hardware_test.ps1
   21  src/shims/dstorage_shim.cpp
   20  .github/workflows/ci.yml
   20  src/gpu/d3d12_xs/command_list_x.h
   18  src/gpu/rdna2_disasm.cpp
   17  src/gpu/d3d12_xs/command_queue_x.h
   16  docs/matrix.svg
   16  src/shims/kernelx_shim.cpp
   16  include/openscarlett/gpu/rdna2_ir.h
   16  src/gpu/d3d12_xs/proxies_x.h
   16  src/shims/gameinput_shim.cpp
   16  src/gpu/d3d12_xs/proxies_x.cpp
   15  src/main.cpp
   15  include/openscarlett/gpu/d3d12_xs_abi.h
   15  include/openscarlett/gpu/rdna2_disasm.h
   15  src/shims/xmem_shim.cpp
   14  src/shims/xaudio2_shim.cpp
   14  src/shims/acphal_shim.cpp
   13  include/openscarlett/gdk/xgameruntime_abi.h
   12  src/gpu/vulkan_backend.cpp
   12  src/shims/xapu_shim.cpp
   12  src/gpu/rdna2_interpreter.cpp
   12  src/loader/vfs_mounts.cpp
   11  src/gpu/rdna2_coverage_tool.cpp
   11  src/loader/apc_injector.cpp
   10  include/openscarlett/io/dstorage_abi.h
```

## Merges (primeiros 100)
```
bd8568e|2026-11-24|Merge branch 'neon/wt-rdna2-fssm'
cef939b|2026-11-24|Merge branch 'neon/wt-bvh-slots'
9b334cd|2026-11-24|Merge branch 'neon/wt-lockhart-mem'
46d8e2c|2026-11-24|Merge branch 'main' into neon/audio
022c5e1|2026-11-24|merge(audio): sync neon/audio with main (ef763eb) keeping real xapu engine & SRWLOCK
035034e|2026-11-23|merge(s16): integrate W4 OS, VFS, GDK runtime & retire all remaining 6 non-media stub DLLs
0a65ecd|2026-11-23|merge(s16): integrate W3 storage, codecs, audio & video completeness (DirectStorage X2, BC7 98 presets/TTPF, XAudio2/AcpHal/xapu, mfhevcdec/mflnmhelp, Bink2 GPU YUV->RGBA)
211ec9e|2026-11-23|merge(s16): integrate W2 RDNA2 shader compiler completeness (single s_buffer_load bounds check, BVH clamping, last-mile ISA, all 7 stages GPU verified, MT async PSO cache)
934879b|2026-11-23|merge(s16): integrate W1 graphics engine completeness (page heaps, unbounded samplers, Copy queue, ExecuteIndirect, FMASK/DCC/HTILE GPU resolve, Vulkan 1.4, residency)
cc86780|2026-11-23|merge(main): integrate feat/minecraft-preview-perf (Minecraft 60 FPS 3D, UE5 7-DLL surface, os_live_profiler, 85/85 CTest)
cbfd4f1|2026-11-23|Merge branch 'feat/ue5-matrix-awakens-imports' into feat/minecraft-preview-perf
74c3a5c|2026-11-23|Merge branch 'main' into feat/minecraft-preview-perf
ad6643c|2026-11-23|merge(s15): integrate s15-w4-shims (XML entities, VFS/ReplaceFile/ANSI/delay-load, XAsync, GDK, KernelX, ETW)
2733bfb|2026-11-23|merge(s15): integrate s15-w3-graphics (placed-tex promotion, UAV/MSAA/DXGIX, Vk GPU rank, XG 3D, BC6H/TTPF)
3d4a8d7|2026-11-23|merge(s15): integrate s15-w2-shaders (ds_swizzle, PC-rel branch, SMEM/LDS D16, Wave64, Blackwell/NaN, SPIR-V 1.2)
ce6b27d|2026-11-23|merge(s15): integrate s15-w1-rdna2-bench
004fc58|2026-11-23|Merge branch 'main' into feat/ue5-matrix-awakens-imports
37674f0|2026-11-23|Merge remote-tracking branch 'origin/main' into feat/minecraft-preview-bringup
59eb76b|2026-11-23|merge(main): integrate retail Scarlett D3D12/GameInput/AcpHal/xapu/XMem ABI fixes
ac47c20|2026-11-23|merge: integrate feat/minecraft-preview-bringup (09c65d1) into main (75/75 CTest)
4d7cf5e|2026-11-23|merge(ue5): Worker 4 dxcompiler_xs routing, static missing-DLL stub synthesis & Launcher copy UX
1d3014c|2026-11-23|merge(ue5): Worker 3 AcpHal APU heap, XMem allocation hooks & MFPlat System32 forwarding
100e39c|2026-11-23|merge(ue5): Worker 2 d3d12_xs COM GUID data exports & D3D12Xbox helpers
ff6ff88|2026-11-23|merge(ue5): Worker 1 dxcompiler_xs, XCRDAPI DStorageGetFactory & xg_xs XGCreateTextureComputer
fd65845|2026-11-23|Merge branch 'mine-w3-routing-apisets' into feat/minecraft-preview-bringup
6b7b51e|2026-11-23|Merge branch 'mine-w2-d3d12-xmem' into feat/minecraft-preview-bringup
cc6bace|2026-11-23|merge(s14): Worker 5 Issue #29 RDNA4 VOPC hardening, SM 6.6 + all-stage profiles, RayQuery lifter & 779/779 opcodes
aa33c6b|2026-11-23|merge(s14): Worker 4 DirectStorage X2, RFC1950 Zlib, BCPack BC1-BC7, concurrent IOCP (>64GiB) & MSAA tiled layouts
c43d660|2026-11-23|merge(s14): Worker 3 XboxBvhBuilder 12-method C++ ABI, dual-path BVH4 rehydration & LiftDxrStateObjectDesc
67ce440|2026-11-23|merge(s14): Worker 2 D3D12X Agility SDK, unwrapped Enhanced Barriers, live VRAM residency & PSO hardening
163d93b|2026-11-23|merge(s14): Worker 1 P0 shims, XAudio2 ordinals, FFX 2.0 ABI, middleware routing & full MicrosoftGame.Config
40320bf|2026-11-22|merge(s13): Worker 5 6/6 untouched MIT _GAMING_XBOX_SCARLETT samples (+SimpleBezier, +SimpleCompute)
b7f1aaf|2026-11-22|merge(s13): Worker 4 offline-first XSTS, XTaskQueue waiter re-arm, dialog/XError bounds & UMS/wldp shims
a6476a2|2026-11-22|merge(s13): Worker 3 shader CFG/MIMG/V#/LDS safety & 764/779 real GPU-verified opcodes
78ab969|2026-11-22|merge(s13): Worker 2 Vulkan OPTIONS2/7 & custom border color, tiled placed textures & XG stencil/HTILE
963bd26|2026-11-22|merge(s13): Worker 1 live Xbox DXBC/PM4/RDNA2 + RayQuery DeviceX pipeline & unified texture sync
1be8083|2026-11-22|merge(s12): Worker 5 live PeForwarder/LSA/probe, TLS slot 0, NtCreateFile VFS, GameInput & 128-voice XAudio2
2b4e485|2026-11-22|merge(s12): Worker 4 hardened DirectStorage/BCPack/metadata coherency, page aliasing & VRAM residency
6c17dc8|2026-11-22|merge(s12): Worker 3 live RDNA2 RayQuery DXR/SPIR-V bridge, 64-bit multi-window BVH4 & SBT remapping
37e54cf|2026-11-22|merge(s12): Worker 2 Xbox DXBC/PM4/SGPR shader container, CFG hardening & in-proc DXC cache
eadac72|2026-11-22|merge(s12): Worker 1 live D3D12X island wiring, GXDK extensions & PresentX pacing
80081d5|2026-11-22|merge(s12): Worker 6 4 untouched Microsoft Xbox-GDK-Samples (SimpleTriangle, SimpleLighting, SimpleTexture, SimpleInstancing)
902c38e|2026-11-22|merge(s11): Worker 1 615/779 real GPU-verified RDNA2 opcodes, 414 spirv-val, 432 dxc & 4 GFX10.3 HW accuracy rules
95fde10|2026-11-22|merge(s11): Worker 4 12 KiB XVD inspector, mixed-case PE forwarders, Console LSA/PFN policy & 2 extra MIT GDKX samples
b8bbf32|2026-11-22|merge(s11): Worker 3 console Sampler Feedback readback/clear, QueryAccumulatorX, predicated byte copies & GPU-QPC clock calibration
c492ca7|2026-11-22|merge(s11): Worker 2 AS TaskEXT SPIR-V 1.6 (130/130 spirv-val), Wave64 LDS & barycentric CullDistance
e555736|2026-11-22|feat(gdkx): fix real _GAMING_XBOX_SCARLETT d3d12_xs ABI & boot untouched SimpleTriangle + SimpleLighting (53/53 CTest)
df175a2|2026-11-22|merge(s910): Worker 1 d3d12_xs 10 COM proxies, Xbox extensions & multi-queue thread safety
c39a175|2026-11-22|merge(s910): Worker 6 First-Boot Probe VEH crash dumper, frame capture & launcher UI
d3f2277|2026-11-22|merge(s910): Worker 5 MFMediaEngine video decode, 64-voice XAudio2, Bink2 streaming & GameInput
dca4cc7|2026-11-22|merge(s910): Worker 4 7-core affinity remap, WinRT lifecycle & non-blocking offline auth
e3cb233|2026-11-22|merge(s910): Worker 3 GPU compute untiler/retiler, HS/DS/GS, DXR SBT, Wave64-on-32 & async PSO pool
18da085|2026-11-22|merge(s910): Worker 2 64-bit TitleVAMirror & RDNA2 V#/T#/S# bindless SGPR bridge
8633ad7|2026-11-22|Merge pull request #15 from marquezinii/fix/build-cmd-toolchain-discovery
8319812|2026-11-22|Merge pull request #16 from marquezinii/fix/hw-report-repo-path
c518759|2026-11-22|merge(sprint8): Worker 4 launcher icon resource & 3-tier updater zip extraction
554073a|2026-11-22|merge(sprint8): Worker 3 GFX10.3 779-opcode coverage auditor & 3,120-shader corpus bench
05b1df9|2026-11-22|merge(sprint8): Worker 2 GFX10.3 MUBUF/MTBUF/MIMG/DS/FLAT/EXP ISA (449 mnemonics)
3208ca1|2026-11-22|merge(sprint8): Worker 1 GFX10.3 scalar & vector ISA (744 mnemonics, SDWA/DPP16/DPP8)
54c2457|2026-11-22|merge: Wave 2C (Native Win32/Direct2D dark GUI shell & launcher/updater test suite)
686dd75|2026-11-22|merge: Wave 2B (Hardware test runner, GPU telemetry & PE inspector bridge)
fb45982|2026-11-22|merge: Wave 2A (WinHTTP + BCrypt SHA-256 opt-in updater)
360d909|2026-11-22|merge: Wave 1 Worker 4 (3D shadowed skinned BC1/BC3 GDK scene & dual-API test)
f5bfe24|2026-11-22|merge: Wave 1 Worker 3 (PE/loader hardening, XMemVirtualQuery, GDK parity & 6 stubs)
666ffcc|2026-11-22|merge: Wave 1 Worker 2 (RDNA2 lockstep LDS barriers, no-op opcodes & diff ISA)
4447291|2026-11-22|merge: Wave 1 Worker 1 (d3d12_xs->Vulkan routing & cross-vendor edge cases)
9685348|2026-11-22|merge: integrate feat/playable-scarlett-game
ea82eb8|2026-11-22|merge: integrate feat/bcpack-bvh4-codecs
9a3f027|2026-11-22|merge: integrate feat/loader-vfs-child-hook
51f8386|2026-11-22|merge: integrate feat/dstorage-mfplat-audio-shims
5dbc8ff|2026-11-22|merge: integrate feat/xgr-100pct-queryapi
269170a|2026-11-22|merge: integrate feat/gpu-d3d12x-vk-tiling
55c660f|2026-11-22|merge: integrate feat/rdna2-corpus-parity
c05c58e|2026-11-22|feat(wave2): merge wave2/vulkan-mesh-showcase
a7bb35b|2026-11-22|feat(wave2): merge wave2/gdk-hdr-cubemap
9aacf54|2026-11-22|feat(wave2): merge wave2/gdk-simple-lighting
fd31b27|2026-11-22|Merge branch 'feat/loader-vfs-routing'
364c70d|2026-11-22|Merge branch 'feat/xgameruntime-thunks-parity'
96d41b3|2026-11-22|Merge branch 'feat/xcurl-xsapi-parity'
92aa4e4|2026-11-22|feat(wave1): merge wave1/mingw-ci-portability
480945a|2026-11-22|feat(wave1): merge wave1/gdk-display-audio-loader
bab0de4|2026-11-22|feat(wave1): merge wave1/hdr10-d3d12-catches
af78d02|2026-11-22|feat(wave1): merge wave1/rdna2-isa-catches
3316f2b|2026-11-22|feat(wave1): merge wave1/vulkan-full-graphics
69f0d1b|2026-11-22|feat(wave2): merge wave2/external-gdk-bringup
a80932e|2026-11-21|feat(wave2): merge final wave2/gdk-meshlet-cull (Win10 WARP fix + HW/WARP parity)
a6a1a05|2026-11-21|feat(wave2): merge wave2/gdk-meshlet-cull
851a20c|2026-11-21|feat(wave2): merge wave2/gdk-dxr-triangle
d41617c|2026-11-21|feat(wave2): merge wave2/gdk-simple-pbr
8aa18bf|2026-11-21|feat(wave1): merge wave1/presentx-windowing
a212196|2026-11-21|feat(wave1): merge wave1/secondary-shims-queryapi
f3ab0f9|2026-11-21|feat(wave1): merge wave1/loader-vfs-config
1a0e592|2026-11-21|feat(wave1): merge wave1/audio-textures-dstorage
78614ba|2026-11-21|feat(wave1): merge wave1/rdna2-aaa-cache
f69c998|2026-11-21|feat(bringup): merge Sprint 3 real MIT Microsoft Xbox-GDK-Samples live bring-up
71436e8|2026-11-21|feat(shaders): merge AAA RDNA2 lifter with MIMG, DS/LDS, wave64-on-wave32, and SM6.5 Mesh/AS
90dce25|2026-11-21|feat(dxr): merge Hardware DXR Tier 1.1 pipeline with SAH software BVH fallback
e4bc196|2026-11-21|feat(graphics): merge modular d3d12_xs COM proxy, xg_xs swizzle, high-perf adapter selector, and GPU FSR 1.0
2c8c5cd|2026-11-21|feat(gdk): merge clean-room xgameruntime, GameInput, XAudio2/AcpHal, dstorage_xs, and XSAPI shims
efe0954|2026-11-21|feat(kernelx): merge VirtualAlloc2/MapViewOfFile3 memory aliasing, GpuTilePool, and kernelx.dll
```

## Branches remotas nao mergeadas
```
origin/feat/ue5-matrix-awakens-imports
```

## Reverts explicitos
```

```

## Candidatos a correcao
```
6fb3502|2026-11-26|californiaAlmonds|feat(gpu,d3d12_xs): fix Bink2 video, GFX10 v_cmpx ramps, untiling, and Scaleform FTUE UI
022a8b2|2026-11-25|californiaAlmonds|feat(d3d12_xs,gpu,loader): unlock RAGE9 720p/4K rendering, async fence pacing & 4GB VRAM scaling
b616d32|2026-11-24|californiaAlmonds|feat(gpu): unlock XDAT FSSM v5/v6 CS/MS/VS/PS lifting and fix VOP2 cndmask & s_endpgm reachability
bda9242|2026-11-24|californiaAlmonds|fix(gpu,rdna2,input): harden descriptor materializer/barriers, add 6 VOP1 math ops & post-landing shader support
f8e24d4|2026-11-24|californiaAlmonds|feat(gpu): live 4K retail Scaleform rendering via guest descriptor materializer, RECTLIST expansion & RDNA2 MUBUF fixes
ef4448a|2026-11-24|californiaAlmonds|fix(audio): harden Opus MFT System32 loading & retail XApu raw-packet/pitch/channel fidelity
ef763eb|2026-11-24|californiaAlmonds|feat(boot,d3d12_xs,xapu): clear Stops 37-41 to complete 249/249 GTA VI init tasks at 30 FPS
3cc1984|2026-11-24|californiaAlmonds|fix(audio): align external_gdk xaudio2_xs.h with 19-slot IXAudio2Voice vtable and pack(1) ABI
3e9162c|2026-11-24|californiaAlmonds|feat(audio): implement real chad.dll FFT/IFFT/Convolution DSP engine with SRWLOCK
1c56104|2026-11-24|californiaAlmonds|fix(audio): restore AcpHal OOM/SHAPE rollback, SRWLOCK sync, and XApu shm test parity
4808ad8|2026-11-24|californiaAlmonds|perf(gpu,kernel): 256MB render-graph VA aliasing heap & 64KB dirty-hash mirroring (Stops 35-36)
0cbc742|2026-11-24|californiaAlmonds|feat(d3d12_xs,rdna2): fix GTX 1650 Ti 0x887A0006 hang & unpack retail XDAT/XBCH/FSSM shaders
c82ca43|2026-11-23|californiaAlmonds|feat(d3d12_xs,dstorage,xmem,audio): retail GRDK 2504/2510 live 4K PresentX bring-up (Stops 18-34)
b9d988c|2026-11-23|californiaAlmonds|docs(readme,status): highlight GRDK 2504/2510 Stops 1-17 (chad.dll, 33 DLLs / 1,360 exports), input/save/MSAA fixes, UE5 4K PresentX & PRs #40/#41/#44
82bd3fa|2026-11-23|californiaAlmonds|feat(scarlett,d3d12_xs,xmem): clear retail GRDK 2504/2510 boot Stops 1-17 (chad.dll, __XMemAllocationHooks__, Zen 2 CLWB VEH, 4K RT + BVH init)
05a24bc|2026-11-23|californiaAlmonds|fix(input,gdk,gpu): fix QPC microsecond overflow, XInput-first polling, Base64 XGameSave names & MSAA sample-count clamping
cee8977|2026-11-23|aidenhrld|fix(gameinput): add opt-in F8 cursor-lock auto-recenter and WM_SETCURSOR hiding (#41)
f99d57e|2026-11-23|californiaAlmonds|fix(tests): gate DispatchMesh live test on MeshShaderTier >= 1 (fixes #44)
438fdb0|2026-11-23|Felipe Marquezini|fix(tests): let the work-dir checks pass when the build lives on C: (#40)
d25a849|2026-11-23|californiaAlmonds|fix(vfs,docs): expand 8.3 short paths in TryMapHostPathToVirtualW (fixes CI) & update README for Sprint 16 (89/89 CTest, 0 stub DLLs)
4bc2db4|2026-11-23|californiaAlmonds|fix(input,vfs,window): native System32 GameInput GIP backend, target HWND/focus handoff & VFS GetFinalPathNameByHandleW guard
a208562|2026-11-23|californiaAlmonds|fix(dstorage): eliminate IocpReadOp & CloseInternal condition_variable use-after-free races
16c816d|2026-11-23|californiaAlmonds|fix(dstorage): serialize barrier dispatch against in-flight barriers & eliminate priority test wakeup jitter
1ffd939|2026-11-23|californiaAlmonds|fix(ci): mark test_s15_rdna2_bench_trap_harness RUN_SERIAL TRUE to prevent parallel CTest wall-clock contention
74e6c68|2026-11-23|californiaAlmonds|chore(s16): finalize Sprint 16 portability, CI headless/MinGW/portable-smoke fixes & status (89/89 CTest, 0 stub DLLs)
afcd167|2026-11-23|californiaAlmonds|fix(input,gpu,perf): instant XInput/Alt-Tab recovery, OreUI gamepad connect, live Raw Mouse & anti-microstutter PresentX/telemetry
6a6243e|2026-11-23|californiaAlmonds|fix(vfs,audio,dstorage,d3d12_xs): clean-room hardening & cached barrier unwrap
a241ef4|2026-11-23|californiaAlmonds|merge(main,xbox): integrate Sprint 15 Zero-Gaps + Xbox D3D12/VFS/GameInput fixes
f2c0610|2026-11-23|californiaAlmonds|fix(d3d12_xs): restore ScarlettRetailDeviceProxy wrapping in D3D12XboxCreateDevice after UE5 merge
5ce3bc5|2026-11-23|californiaAlmonds|perf(d3d12_xs,telemetry): async PresentX ring, disable retail per-frame readback, fullscreen stretch fix, staging ring & sharded telemetry
67ce767|2026-11-23|californiaAlmonds|wip(ue5): checkpoint Matrix Awakens live bring-up ABI fixes
e7c5f85|2026-11-23|californiaAlmonds|fix(vfs): pass through host D: paths in GetFileAttributes/CreateDirectory hooks (unblocks Minecraft LevelDB world creation)
f7566aa|2026-11-23|californiaAlmonds|fix(d3d12_xs): stage unaligned Xbox CopyTextureRegion footprints to 512B/256B desktop alignment and skip failed command lists
2761df0|2026-11-23|californiaAlmonds|feat(sprint15): Zero-Gaps Sprint ÔÇö 28x faster rdna2_bench (8.68s), honest 775/779 GPU + 4/4 trap harness, full peer parity (79/79 CTest)
d71ed46|2026-11-23|californiaAlmonds|fix(d3d12_xs): do not auto-evict live GPU heaps without explicit budget override
be0fda1|2026-11-23|californiaAlmonds|fix(vfs,gameinput): hook CRT file APIs for LevelDB/P: saves and fix focus gating & gamepad capability separation
7e2dd9b|2026-11-23|californiaAlmonds|perf(build,test): add shared sccache, parallel CTest locks, rdna2_lifter /Og fix & agent speed rules
3112404|2026-11-23|californiaAlmonds|fix(gameinput): match official Microsoft GDK GameInputGamepadButtons bitmask (0x0001..0x2000)
59eb76b|2026-11-23|californiaAlmonds|merge(main): integrate retail Scarlett D3D12/GameInput/AcpHal/xapu/XMem ABI fixes
bc7ae08|2026-11-23|californiaAlmonds|fix(tests): use temp_directory_path in test_mine_routing_apisets for portable bundle smoke test
ac47c20|2026-11-23|californiaAlmonds|merge: integrate feat/minecraft-preview-bringup (09c65d1) into main (75/75 CTest)
2b42f72|2026-11-23|californiaAlmonds|fix(core): release CallTelemetry mutex before forwarding to hook DLL
2241564|2026-11-23|californiaAlmonds|fix(gpu): Intel Arc B390 mesh PSO/Vulkan hardening & log RTX 3060 HW report
675d9f4|2026-11-23|californiaAlmonds|fix(xmem): support real 2-arg Xbox XMemGetWorkingSetStatistics(flags, outStats) ABI
6c40c28|2026-11-23|californiaAlmonds|fix(d3d12_xs): guard D3D12_DEPTH_STENCIL_DESC2 for MinGW-w64 d3d12.h compatibility
8d285d9|2026-11-23|californiaAlmonds|feat(gpu): fix Issue #29 RDNA4 VOPC, SM 6.6 + off-C cache, RayQuery wiring, radar rules, 779/779 + live MS/AS
d467436|2026-11-23|californiaAlmonds|fix(rdna2): use atomic-free per-lane ballot for sv64 VOPC (#29) & log HW reports (#30, #31)
4af2702|2026-11-23|californiaAlmonds|fix(rdna2): avoid AMD RDNA2 v_mul_lo_u16 mask elision, harden S12 audio stress & update HW reports
ec89742|2026-11-23|DeviousCardi|feat(xgameruntime): XSystemHandleTrack / XSystemIsHandleValid handle registry (#24)
76adeff|2026-11-22|Ahmet Dedeler|fix(dstorage): read titles' requests with the Xbox dstorage_xs.h layout (#25)
5d3809b|2026-11-22|Felipe Marquezini|fix(rdna2): lift LDS ds_sub as a CAS loop so NVIDIA can build the pipeline (#26)
762771c|2026-11-22|californiaAlmonds|fix(test): guard hardware-Vulkan checks in test_s13_shader_cfg_safety for headless CI runners
85e2d3d|2026-11-22|californiaAlmonds|fix(dstorage): eliminate lost-wakeup race in DStoragePriorityScheduler::NotifyWorkAvailable
fe86185|2026-11-22|californiaAlmonds|fix(gdkx): remove UNICODE macro on ANSI GDKX samples so DefWindowProcA matches RegisterClassExA under Clang-CL
fc77262|2026-11-22|californiaAlmonds|feat(loader): add opt-in OPENSCARLETT_TRACE_ODS OutputDebugStringA hook for guest diagnostics
990c3ff|2026-11-22|californiaAlmonds|fix(vfs): preserve real host D:\ paths across VFS/Win32/NT hooks on CI runners & host drives
c5d4a7a|2026-11-22|californiaAlmonds|fix(ci): pass /arch:AVX2 under Clang-CL for _GAMING_XBOX_SCARLETT DirectXMath intrinsics
4a7e79b|2026-11-22|californiaAlmonds|fix(ci): resolve Clang-CL constexpr enum range in gdkx_headers/d3d12_xs.h and portable temp path in test_xvd_loader_lsa
d5b1f50|2026-11-22|californiaAlmonds|feat(rdna2): expand real-GPU verification to 615/779 opcodes and fix 4 GFX10.3 HW accuracy rules
d8a5a27|2026-11-22|californiaAlmonds|fix(ci): resolve Clang-CL constexpr enum range and MinGW-w64 ID3D12Device10 pCastableFormats constness
e555736|2026-11-22|californiaAlmonds|feat(gdkx): fix real _GAMING_XBOX_SCARLETT d3d12_xs ABI & boot untouched SimpleTriangle + SimpleLighting (53/53 CTest)
c57f29b|2026-11-22|californiaAlmonds|wip(gdkx): snapshot uncommitted GDKX ABI fixes and untouched SimpleTriangle/SimpleLighting bring-up
a23044c|2026-11-22|californiaAlmonds|fix(rdna2): keep PROC_THREAD_ATTRIBUTE_HANDLE_LIST array alive across CreateProcessA (Clang-CL err=87)
7b90c9c|2026-11-22|californiaAlmonds|fix(ci): resolve Clang-CL narrowing, MinGW __mingw_uuidof IIDs & host-DXR GetShaderStackSize on CI WARP
04a8eba|2026-11-22|californiaAlmonds|feat(s910): wire Sprint 9+10 compatibility engine targets, hooks, VEH probe & LDS/dxc fixes (51/51 CTest)
c39a175|2026-11-22|californiaAlmonds|merge(s910): Worker 6 First-Boot Probe VEH crash dumper, frame capture & launcher UI
4e350ba|2026-11-22|californiaAlmonds|fix(build,launcher): harden x64 prompt check in build.cmd and mirror <REPO_DIR> sanitizer
8633ad7|2026-11-22|californiaAlmonds|Merge pull request #15 from marquezinii/fix/build-cmd-toolchain-discovery
8319812|2026-11-22|californiaAlmonds|Merge pull request #16 from marquezinii/fix/hw-report-repo-path
bee85df|2026-11-22|californiaAlmonds|fix(ci,tests): use portable temp_directory_path in test_rdna2_corpus_bench and skip WARP in D3D12 HW init
c2c93aa|2026-11-22|marquezini|fix(tools): mask the repo folder in hardware reports from source builds
db26565|2026-11-22|marquezini|fix(build): find MSVC, CMake, Ninja and dxc instead of fixed paths
43e8c32|2026-11-22|californiaAlmonds|fix(launcher): embed failing test diagnostics in pre-filled GitHub hardware report URLs (#14)
91364e4|2026-11-22|californiaAlmonds|fix(loader,tests): fix C:\ portable bundle test guards and skip APC-injecting dxc.exe (#14)
34b8196|2026-11-22|californiaAlmonds|fix(launcher,updater): harden zip extraction for CI/spaces and embed docs/icon.ico
0e851da|2026-11-22|californiaAlmonds|fix(updater): use built-in tar.exe with .NET ZipFile fallback to avoid pwsh7 PSModulePath Expand-Archive failure on CI
c9172fc|2026-11-22|californiaAlmonds|fix(ci,tests): guard remaining HW Vulkan SPIR-V paths in test_rdna2_lockstep_isa when no ICD is present
ba1c58f|2026-11-22|californiaAlmonds|fix(ci,tests): skip HW-only Vulkan SPIR-V execution when no ICD is present and honor OPENSCARLETT_VK_FORCE_SOFTWARE
0881167|2026-11-22|californiaAlmonds|docs: fix hardware test references from .bat to run_hardware_test.ps1
6a15cbf|2026-11-22|californiaAlmonds|fix(ci,vfs,gpu): honor explicit VFS env overrides unless forbid_sys_writes set, fix WARP GetResourceTiling mip split & v0.3.1-alpha release notes
4d23c6b|2026-11-22|californiaAlmonds|fix(ci): preserve /EHsc in Clang-CL CMAKE_CXX_FLAGS so C++ exceptions stay enabled
0eb97f9|2026-11-22|californiaAlmonds|fix(ci,docs): fix Clang-CL STL check & MinGW d3d12_xs export macro, switch hardware test refs to .ps1
841d872|2026-11-22|californiaAlmonds|fix(ci,tests): make test_gdk_extended_loader portable for hidden-build smoke test and make Clang-CL leg non-blocking
aa3a175|2026-11-22|californiaAlmonds|fix(readme,status): stop re.sub from injecting NUL byte for XS2\\0 in README.md and update GDK shim metrics
55b9473|2026-11-22|californiaAlmonds|fix(gdk,tests): handle 3-arg XGamingRuntimeThunks QueryApiImpl(clsid, iid, ppv) safely and clean CI VFS overlay
7cc28a7|2026-11-22|californiaAlmonds|fix(build,ci): prevent MSVC 17.10+ std::mutex crash on older msvcp140.dll
d7f1b7d|2026-11-22|californiaAlmonds|fix(ci): harden shader cache SM key, Opus MFT probe, and headless PresentX for Windows Server CI
9aa3955|2026-11-22|californiaAlmonds|fix(tests,docs): polish HW report UX/logs and update RTX 2060 SUPER 13/13 pass (#8)
df979fe|2026-11-22|californiaAlmonds|fix(d3d12_xs): add MinGW-w64 COM aggregate-return and UUID compatibility (#6)
a80932e|2026-11-21|californiaAlmonds|feat(wave2): merge final wave2/gdk-meshlet-cull (Win10 WARP fix + HW/WARP parity)
cb2e7c7|2026-11-21|californiaAlmonds|fix(bringup): fix MeshletCull Win10 WARP DXIL JIT crash and tighten verification
67cf01b|2026-11-21|californiaAlmonds|chore(wave1): fix C4324 warning, portable test_loader_vfs_config paths, and update status dashboard (19/19 CTest)
7d97bb9|2026-11-21|californiaAlmonds|fix(tests): resolve runtime exe dir & TEMP scratch portably (fixes #1) and hide build/ in CI smoke test
529e0a6|2026-11-21|californiaAlmonds|fix(ci): detect Hyper-V Basic Render Driver (0x1414/0x008C) as WARP and fix test_apc_loader temp path
d8518c2|2026-11-21|californiaAlmonds|fix(ci): ensure portable temp/dxc paths and WARP-only CI fallback while preserving strict HW checks
3e9b42a|2026-11-21|californiaAlmonds|fix(graphics): deduplicate FrameBufferRgba8 and sanitize Xbox D3D12 flags + 0-plane PresentX
5e76043|2026-11-21|californiaAlmonds|fix(test): copy VulkanDeviceInfo before Shutdown so GPU name logs accurately
a2763f4|2026-11-21|californiaAlmonds|fix(gpu): replace leftover ScarlettX log strings with OpenScarlett and ignore .worktrees/.tools
```
