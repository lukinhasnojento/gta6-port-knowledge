# OpenScarlett - analise de DeepSeek V4.1 Flash

## A. Identificacao

- **Nome:** OpenScarlett
- **URL:** https://github.com/californiaAlmonds/OpenScarlett
- **Fork ou original:** original. A raiz e `ec4d6d3` ("chore: initialize OpenScarlett MIT C++20 workspace and engineering guide"), sem ancestral externo. Ha um sinal claro de que o projeto foi **renomeado no meio do caminho**: `a2763f4` (2026-11-21) troca strings de log "ScarlettX" por "OpenScarlett" e o `AGENTS.md` em HEAD ainda aponta toolchain em `E:\ScarlettX-dev\...` e builds em `E:\OpenScarlett\...`.
- **Branch analisada:** `main`, HEAD = `7220ce9`.
- **Commit 0:** `ec4d6d389a1105e9cd7b6afab56ec1c48bb2d8dc`, 2026-11-21 20:05:34 +0530, autor californiaAlmonds, "chore: initialize OpenScarlett MIT C++20 workspace and engineering guide".
- **Ultimo commit:** `7220ce949800a959535086ce175b1e13b738aa16`, 2026-11-26 05:42:08 +0530, "feat(gpu,d3d12_xs,loader): add v_cmp_class_f32/f16, Slot129 DispatchMesh, and watchdog/chunk guards".
- **Periodo coberto:** 2026-11-21 a 2026-11-26 (6 dias distintos com commit).
- **Total de commits:** 351 alcancaveis de HEAD; 355 em todas as refs; 4 existem apenas em `origin/feat/ue5-matrix-awakens-imports`.
- **Commits examinados em profundidade:** os 50 blocos de `analise-evidencias/OpenScarlett_evid.md` (diff real) mais cerca de 30 hashes adicionais abertos com `git show`, `git log -S` e `git log --first-parent` para fechar cadeias de introducao, correcao e regressao. Os 351 estao no indice da secao K.
- **Dias com commit:** 2026-11-21 (61), 2026-11-22 (164), 2026-11-23 (96), 2026-11-24 (25), 2026-11-25 (1), 2026-11-26 (4).
- **Merges:** 101 de 351 (28,8%).
- **Autores:** 8 identidades, com concentracao extrema: californiaAlmonds 343 (341 + 2 com e-mail noreply), Felipe Marquezini/marquezini 4, e uma contribuicao cada de rpatty, aidenhrld, DeviousCardi e Ahmet Dedeler.
- **Arquivos versionados:** 545. Os mais alterados sao `README.md` (56), `src/gpu/d3d12_xs/device_x.cpp` (44), `CMakeLists.txt` (44), `docs/status.svg` (42), `src/loader/hook_dll.cpp` (36), `src/gpu/d3d12_xs/command_queue_x.cpp` (35).
- **Licenca:** MIT.
- **Modelo responsavel / data:** DeepSeek V4.1 Flash, 2026-11-26.
- **Correcao de premissa 1:** o contexto desta tarefa diz que "o README declara 67/67 testes". O README em HEAD declara **`89/89` automated tests** no cabecalho escrito a mao (linha 22) e **`90/90` passing** no bloco auto-gerado (linha 122). O valor real de registros de teste nos arquivos de build e **90** (88 chamadas `openscarlett_add_test(` + 2 `add_test` explicitos). `67/67` e o numero de testes que rodavam por maquina em uma fase anterior (Sprint 13) e aparece hoje como resultado **por PC** nos relatorios de hardware. Ou seja: o README em HEAD esta internamente inconsistente (89 vs 90) e o "67/67" do briefing esta defasado.
- **Correcao de premissa 2:** o pedido situa o projeto como camada de compatibilidade "para Xbox Series". O README e explicito que **titulos comerciais AAA alem do Minecraft Preview nao estao completamente jogaveis**; o que existe sao tres frentes separadas: (a) amostras MIT oficiais da Microsoft rodando, (b) dois jogos abertos (Freedoom/ScarlettArena), (c) bring-up de titulos retail em que o boot avanca, desenhos sao descartados e asserts sao neutralizados (secoes G-01, G-02, G-03). "Bootar" e "renderizar" aqui tem significado mais fraco do que a leitura literal sugere.

## B. Resumo tecnico

**Proposito.** Camada de compatibilidade C++20, licenca MIT, para executar binarios **Xbox Series X|S (Scarlett, GDK/GRDK)** x64 nativamente no Windows 10/11. O produto e um conjunto de DLLs que se colocam no lugar das DLLs do console (`d3d12_xs.dll`, `xgameruntime.dll`, `GameInput.dll`, `dstorage_xs.dll`, `kernelx.dll`, `xapu.dll`, `chad.dll` etc.), um loader que injeta essas DLLs em processo suspenso via APC, um tradutor de shaders RDNA2 (GFX10.3) para HLSL/DXIL/SPIR-V e um launcher nativo Win32 com testador de hardware.

**Arquitetura em seis camadas.**

1. **Loader e VFS.** `src/loader/apc_injector.cpp` (injecao em processo suspenso), `src/loader/hook_dll.cpp` (7.738 linhas: hooks de Win32/NT/CRT, roteamento de DLL por tabela, VFS copy-on-write aninhado sobre `G:`/`D:`/`T:`/`S:`/`P:`, VEH de instrucoes ilegais Zen 2 e de traps `int3`), `src/loader/vfs_mounts.cpp` (mounts, resolucao de work dir, mapeamento host<->virtual), `src/loader/pe_inspector.cpp`.
2. **Shims GDK.** 33 DLLs, 1.372 exports no bloco gerado. Os maiores: `src/shims/xgameruntime_shim.cpp` (12.087 linhas, 292/292 thunks + 24 interfaces COM via `QueryApiImpl`), `src/shims/kernelx_shim.cpp` (VirtualAlloc2/MapViewOfFile3, placeholders, `XMem*`), `src/shims/dstorage_shim.cpp` (IOCP, filas com prioridade, decompressores), `src/shims/gameinput_shim.cpp` (XInput + Raw Input + GIP opcional), `src/shims/xapu_shim.cpp`/`acphal_shim.cpp`/`chad_shim.cpp` (audio), `src/shims/xsapi_shim.cpp`, `xcrdapi`, `winrt_xs`.
3. **Camada grafica D3D12.** `src/gpu/d3d12_xs/device_x.cpp` (**18.879 linhas**) mais `command_queue_x.cpp`, `command_list_x.cpp`, `proxies_x.cpp`. Implementa proxys COM por slot de vtable para a ABI retail Scarlett (`Version >= 0x00020000`), sanitizacao de enums/flags console-only, descritores, heaps, aliasing de VA, residencia, `PresentX` com swapchain Win32 e captura, e o caminho de "auto-lift" que traduz o bytecode do titulo em PSO na primeira vinculacao.
4. **Tradutor RDNA2.** `rdna2_disasm.cpp`, `rdna2_ir.cpp`, `rdna2_lifter.cpp` (8.988 linhas), `rdna2_isa_scalar_vector.cpp`, `rdna2_isa_mem_img.cpp`, `rdna2_interpreter.cpp` (referencia CPU), `rdna2_xbox_container.cpp` (desempacota `DXBC -> DXIL -> XDAT -> XBCH(zlib) -> FSSM v5/v6`), `rdna2_coverage_tool.cpp`, mais `shader_cache.cpp` (cache em disco e compilacao in-process via `dxcompiler.dll`).
5. **Backend alternativo.** `src/gpu/vulkan_backend.cpp` (Vulkan 1.2/1.4 sem SDK, `VK_EXT_mesh_shader`, ray query, swapchain) e roteamento direto `d3d12_xs -> Vulkan`.
6. **Medida e documentacao.** 90 testes CTest, `tools/generate_status.py` (gera `README.md`, `docs/status.svg`, `docs/index.html`, `docs/matrix.svg`), `tools/run_hardware_test.ps1`, `OpenScarlettLauncher.exe` (Win32/Direct2D, testador de 60 s, inspetor de PE, updater WinHTTP + BCrypt), `tools/os_live_profiler.cpp`.

**Estado atual.** Alpha (`v0.6.0-alpha`). O que esta provado e heterogeneo: as amostras MIT oficiais bootam; um conjunto de propriedades de shader e de ABI e verificado em GPU real do autor (GTX 1650 Ti) e em WARP; os numeros de cobertura (779/779 opcodes decodificados, 3.120 shaders no corpus) sao de ferramenta, nao de titulo. O que **nao** esta provado: qualquer titulo comercial completo, e a propria definicao de "bootar" depende de mecanismos que alteram o comportamento do convidado (secao G).

**Concentracao de risco.** Duas classes dominam os defeitos encontrados. A primeira e **ABI declarada a partir do cabecalho de desktop em vez do cabecalho Xbox** — apareceu de forma independente em DirectStorage, GameInput, `XMemGetWorkingSetStatistics` e no duplo de teste de XAudio2; tres delas so foram descobertas por titulos reais ou por contribuidor externo, nunca pela suite. A segunda e **estado do autor vazando para o produto**: caminho `E:\OpenScarlett`, `C:\` proibido, benchmark de wall-clock, `%TEMP%` de runner, driver NVIDIA especifico. A terceira, menor em numero mas maior em consequencia, e **o pipeline que publica os numeros**: ele le o ultimo log de CTest da maquina e imprime literais fixos no template, o que ja fez o README anunciar "6/6 testes".

## C. Linha do tempo

**Fase 0 - bootstrap (2026-11-21, 61 commits).** Em uma tarde o repositorio sai do zero para loader + VFS + 19 DLLs + proxy D3D12 + lifter RDNA2 + DXR + amostras GDK. Ordem: `ec4d6d3` (workspace e guia de engenharia), `775af44` (inspetor PE32+, telemetria de chamadas), `d971f0d` (GDeflate/BCPack/XMA2/Bink2), `2fa61ca` (Vulkan 1.2/1.4 + decoder RDNA2 + IR SSA), `9e710fd` (WinRT/HSTRING), `c6857ea`+`40a2ddf` (loader APC + VFS `G:`/`T:`/`S:`), `8eddb1e`/`efe0954` (kernelx, VirtualAlloc2, GpuTilePool), `a429535`/`2c8c5cd` (xgameruntime, GameInput, XAudio2/AcpHal, **dstorage_xs**, XSAPI), `d58f0b6`/`e4bc196` (proxies `d3d12_xs`, `xg_xs`, FSR 1.0), `dd2efb5` (DXR Tier 1.1 + BVH SAH), `538e904` (opcodes AAA, parser DXBC, interpretador wave32/64, **cache de shader em disco**, auto-lift), `dacad04`/`f69c998` (amostras MIT rodando). As primeiras correcoes ja sao sintomaticas: `a2763f4` (sobrou "ScarlettX" nos logs — projeto renomeado), `3e9b42a` (deduplicar `FrameBufferRgba8`, sanear flags D3D12 Xbox), `5e76043` (copiar `VulkanDeviceInfo` antes do `Shutdown`), `d8518c2`/`529e0a6`/`7d97bb9` (tres tentativas de caminho temporario portatil e deteccao de WARP no CI), `cb2e7c7` (crash de JIT DXIL do MeshletCull no WARP do Win10).

**Fase 1 - amplitude e portabilidade (2026-11-22, 164 commits).** Sprints 5 a 13. Chegam Vulkan 1.4 completo, HDR10, launcher Win32/D2D, GFX10.3 com 744 + 449 mnemonicos, auditor de 779 opcodes, corpus de 3.120 shaders, `TitleVAMirror`, `DescriptorSgprBridge`, First-Boot Probe com VEH, afinidade de 7 nucleos, RayQuery, 128 vozes de XAudio2 e as 6 amostras `_GAMING_XBOX_SCARLETT` intocadas. E o dia em que o projeto descobre, na pratica, que o Windows do CI nao e o Windows do autor: `39c8520` adiciona legs Clang-CL e MinGW, `841d872` torna o Clang-CL **nao bloqueante**, `0eb97f9`/`4d23c6b`/`c5d4a7a`/`7b90c9c`/`d8a5a27`/`4a7e79b`/`a23044c` sao sete correcoes de toolchain no mesmo dia. Tambem e o dia em que a ABI e corrigida por quem tem o cabecalho certo: `76adeff` (#25, contribuidor externo) descobre que `IDStorageQueueX::EnqueueRequest` lia o struct **de desktop** em vez do struct Xbox de 0x48 bytes; `5d3809b` (#26) contorna um bug do driver NVIDIA no `ds_sub` em LDS. E o dia do pior commit da historia do repositorio em termos de publicacao: `4af2702` faz o README anunciar **"6/6" testes automatizados**.

**Fase 2 - retail e inflacao de status (2026-11-23, 96 commits).** Sprints 14, 15 e 16 mais o bring-up "GRDK 2504/2510 Stops 1-17" e "Stops 18-34". Entram `XboxBvhBuilder`, DirectStorage X2, Enhanced Barriers, Agility SDK, `chad.dll`, os motores de audio reais, Minecraft Preview em 60 FPS e a superficie de imports do UE5. E o dia com mais correcoes de qualidade real e mais regressao de processo ao mesmo tempo: `3112404` conserta o bitmask de GameInput, `675d9f4` descobre a assinatura de 2 argumentos de `XMemGetWorkingSetStatistics`, `e7c5f85` conserta o passthrough de `D:\` que faltava em `GetFileAttributes`/`CreateDirectory`, `d71ed46` para de evictar heaps vivos, `f7566aa` passa a alinhar footprints de `CopyTextureRegion`, `2b42f72` solta o mutex da telemetria antes de chamar a hook DLL, `a208562` e `16c816d` fecham corridas de `condition_variable` no DirectStorage, `05a24bc` conserta o overflow do QPC. No mesmo dia: `cbfd4f1` (merge do ramo UE5) **derruba** o wrap do `ScarlettRetailDeviceProxy`, `f2c0610` o restaura; `7e2dd9b` institucionaliza `E:\` e `ctest --parallel 4` no `AGENTS.md` e nos scripts, `1ffd939` precisa marcar o benchmark como `RUN_SERIAL` por causa disso; e `d25a849`/`438fdb0` consertam testes que exigiam "fora de `C:`" quando o proprio build estava em `C:`.

**Fase 3 - o titulo retail (2026-11-24, 25 commits, o dia mais denso por commit).** E aqui que o alvo real aparece. `82bd3fa`/`c82ca43` limpam os "Stops 1-34"; `0cbc742` descobre que topologias console-only (`17/18/19/20`) passadas ao host **travam o pushbuffer da NVIDIA** com `0x887A0006` em vez de retornar erro; `4808ad8` cria o heap de aliasing de 256 MiB com `VirtualAddress & (pool-1)`; `ef763eb` fecha "249/249 init tasks" desbloqueando um `XApuGetResultCount` que devolvia a contagem em `EAX`; `f8e24d4`/`bda9242`/`c382f68`/`86d18ac` fazem o Scaleform renderizar 4K; `b616d32` corrige `VOP2 cndmask` e alcancabilidade de `s_endpgm`; o audio ganha motores reais (`b08fa23` xapu Opus, `3e9162c` chad FFT, `defb915` AcpHal XMA2, `1c56104` rollback de OOM/SHAPE) e `3cc1984` revela que o **cabecalho de teste** de XAudio2 tinha 8 slots onde a vtable real tem 19. No fim do dia, tres merges `neon/*` fecham FSSM v5/v6, slots de BVH e memoria Lockhart.

**Fase 4 - desempenho e cauda longa (2026-11-25 e 2026-11-26, 5 commits).** `022a8b2` remove as esperas sincronas de fence do caminho de submissao (5,0 FPS -> 161-250 FPS a 720p, numeros do autor), adiciona composicao de dois planos no `PresentX` e escala de VRAM para 4 GB. Em 26/11, quatro commits fecham `v_cmpx`, Bink2, untiling, `Slot129 DispatchMesh`, split de dispatch acima de 65535 e "watchdog/chunk guards" no loader (o VEH passa a reconhecer mais padroes de byte do titulo).

## D. Catalogo de defeitos

Escopo e metodo: cada um dos 50 blocos do pacote de evidencias foi julgado **pelo diff**, nao pela mensagem. Cada cadeia foi fechada com `git show` do commit introdutor, `git log <hash>..HEAD -S"<trecho>"` para verificar se o problema voltou, `git log --first-parent -- <arquivo>` para separar o que entrou por merge e leitura do arquivo em HEAD. Nenhum hash, issue ou resultado de teste abaixo foi inferido. Classes de confianca conforme o SPEC.

- **D-01** DirectStorage le o pedido do titulo no layout de **desktop**, nao no layout Xbox
  - Sintomas: qualquer titulo compilado contra `dstorage_xs.h` enfileirava um struct de 0x48 bytes (opcoes em bitfield de 64 bits, `Destination` @0x08, `DestinationSize` @0x18, `File/Source` @0x20, `FileOffset` @0x28, `SourceSize` @0x30, `IntermediateSize` @0x34, `CancellationTag` @0x38, `Name` @0x40). O shim lia com a forma de `dstorage.h` de PC (`Options` de 1 byte, unions Source/Destination, `UncompressedSize`): **todo campo era lido no offset errado**. I/O de titulo real lia lixo — destino, tamanho e offset arbitrarios.
  - Causa raiz: `include/openscarlett/io/dstorage_abi.h` foi escrito a partir do cabecalho de desktop, nao da ABI de console; `EnqueueRequest` nao tinha nenhum `static_assert` de tamanho/offset que denunciasse a diferenca.
  - Introduzido em: `a429535` (2026-11-21, criacao do shim; o arquivo nasce nesse commit com `--diff-filter=A`).
  - Tentativas de correcao: nenhuma intermediaria; o fix e o proprio `76adeff`.
  - Correcao definitiva: `76adeff` (2026-11-22, PR #25 de Ahmet Dedeler). Reescreve `dstorage_abi.h` com `DSTORAGE_REQUEST`, `DSTORAGE_REQUEST_OPTIONS`, `DSTORAGE_QUEUE_DESC` (`UINT64 SourceType`, sem `Device`), `DSTORAGE_ERROR_RECORD` (0x290), `DSTORAGE_MAX_QUEUE_CAPACITY` 0x2000 e **static_asserts de tamanho e offset**; o struct antigo e rebaixado a extensao de host (`OpenScarlettDStorageRequestDesc`) com export proprio; `EnqueueRequest` passa a **rejeitar** zlib/BCPack com `E_DSTORAGE_DECOMPRESSION_ERROR` e swizzle/physical-page com `E_INVALIDARG` em vez de adivinhar.
  - Arquivos: `include/openscarlett/io/dstorage_abi.h`, `src/shims/dstorage_shim.cpp`, `tests/test_dstorage_xbox_request_layout.cpp`, `cmake/gdk_targets.cmake`, `cmake/wave2_external_gdk.cmake`.
  - Evidencia: diff de `76adeff` (o bloco de offsets na descricao do commit e o `static_assert` no header); teste novo que escreve cada campo no offset documentado de um buffer de 0x48 bytes com padding envenenado. `git log -S'UncompressedSize' -- include/openscarlett/io/dstorage_abi.h` mostra que a forma desktop sai em `76adeff`.
  - Estado atual: corrigido. Em HEAD o teste `test_dstorage_xbox_request_layout` esta registrado no CTest.
  - Confianca: CONFIRMADA.

- **D-02** `GameInputGamepadButtons` deslocado em dois bits
  - Sintomas: o shim declarava `Menu=0x4`, `View=0x8`, `A=0x10`, `B=0x20`, `X=0x40`, `Y=0x80`, `DPadUp=0x100`... A ABI oficial do GDK e `Menu=0x1`, `View=0x2`, `A=0x4`, `B=0x8`, `X=0x10`, `Y=0x20`, `DPadUp=0x40`, ..., `RightThumbstick=0x2000`. O efeito e um deslocamento de dois bits: o A fisico era reportado no bit 0x10, que na ABI real e X, e assim por diante — nenhum botao casava com o esperado pelo titulo.
  - Causa raiz: os valores foram digitados de memoria na criacao do shim (`0x0001..0x2000` virou `0x0004..0x8000`), sem copiar o cabecalho oficial nem fixar um teste contra ele.
  - Introduzido em: `a429535` (2026-11-21; `-S'GameInputGamepadMenu = 0x00000004u'` aponta so para ele).
  - Tentativas de correcao: nenhuma intermediaria.
  - Correcao definitiva: `3112404` (2026-11-23). Corrige o enum no header de produto **e** no cabecalho de compatibilidade usado pelos testes (`tests/external_gdk/gdk_compat_headers/GameInput.h`), que tinha o mesmo erro copiado.
  - Arquivos: `include/openscarlett/input/gameinput_abi.h`, `tests/external_gdk/gdk_compat_headers/GameInput.h`.
  - Evidencia: os dois diffs identicos de 14 linhas cada; em HEAD os dois arquivos trazem `GameInputGamepadMenu = 0x00000001u`.
  - Estado atual: corrigido; nao voltou (o valor 0x4 nao existe mais em nenhum arquivo sob `include/` ou `tests/`).
  - Confianca: CONFIRMADA.

- **D-03** `XMemGetWorkingSetStatistics` com assinatura de um argumento e deteccao por `VirtualQuery`
  - Sintomas: titulos retail chamam `BOOL XMemGetWorkingSetStatistics(DWORD flags, XMEM_WORKING_SET_STATISTICS* outStats)`; o shim declarava `int32_t XMemGetWorkingSetStatistics(void* lpBuffer)` e fazia `memcpy` de um `uint64_t` do **primeiro** argumento, que na chamada real e uma palavra de flags (ex.: `0x20000001`). Resultado: leitura de memoria em endereco baixo e escrita de 32 bytes em lugar nenhum.
  - Causa raiz: ABI inventada na implementacao inicial do shim de kernelx, sem cabecalho de referencia.
  - Introduzido em: `669f0b9` (2026-11-23, "XMem stats" dentro do Sprint 14 P0/P1). O arquivo `src/shims/kernelx_shim.cpp` nasce em `8eddb1e`, mas a funcao entra em `669f0b9` (`git log -S'XMemGetWorkingSetStatistics'`).
  - Tentativas de correcao: nenhuma; o "fix" e uma heuristica.
  - Correcao definitiva: **parcial** em `675d9f4` (2026-11-23), refinada depois. Em vez de corrigir a assinatura, a funcao passa a aceitar dois `uintptr_t` e **adivinhar** qual ABI o chamador usa: se `arg0` nao aponta para regiao `MEM_COMMIT` gravavel e `arg1` aponta, assume 2 argumentos; se `arg1` e nulo e `arg0` tem o bit `0x20000000`, tambem. Em HEAD (`kernelx_shim.cpp:3533-3571`) a heuristica continua, com o comentario "Preserve backwards compatibility with the 1-arg (void* lpBuffer) test ABI by probing".
  - Arquivos: `src/shims/kernelx_shim.cpp`, `src/shims/xmem_shim.cpp`, `tests/test_mine_d3d12_xmem.cpp`.
  - Evidencia: diff de `675d9f4` (o bloco `is_two_arg_xbox_abi`); leitura do arquivo em HEAD confirmando que a heuristica sobreviveu a todos os refactors posteriores.
  - Estado atual: **funciona por heuristica, nao por contrato**. Se o buffer de saida cair em pagina somente-leitura ou o valor de `arg0` por coincidencia apontar para memoria gravavel, a funcao escolhe a ABI errada e faz `memcpy` de/para o lugar errado.
  - Confianca: CONFIRMADA para a assinatura errada e para a heuristica; HIPOTESE para o caminho de falha (nao ha caso medido).

- **D-04** O duplo de teste de XAudio2 declarava uma vtable de 8 slots onde a real tem 19
  - Sintomas: `tests/external_gdk/gdk_compat_headers/xaudio2_xs.h` define `IXAudio2Voice` com `GetVoiceDetails`, `SetOutputVoices`, `SetVolume`, `GetVolume`, `SetChannelVolumes`, `GetChannelVolumes`, `DestroyVoice` — 7 metodos, `Start`/`Stop` de `IXAudio2SourceVoice` caindo no slot 8. A vtable real do XAudio2.9 tem **19 slots** (efeitos, filtros, matriz de saida). O teste `test_external_gdk_bringup` falhava na voz PCM16 e a causa foi atribuida ao teste, nao ao produto.
  - Causa raiz: o cabecalho de compatibilidade dos testes foi escrito a mao, com uma versao reduzida da interface, e **nao foi sincronizado** com `include/openscarlett/audio/xaudio2_abi.h`, que tinha os 19 slots desde o inicio. Enquanto o duplo de teste tivesse 8 slots, nenhum teste podia detectar erro de slot na vtable do shim — o oraculo estava errado junto com o objeto medido.
  - Introduzido em: `59bd45a` (2026-11-22, "GDK-header external binaries and 19-DLL live telemetry bring-up"; `--diff-filter=A` no arquivo).
  - Tentativas de correcao: nenhuma antes do fix.
  - Correcao definitiva: `3cc1984` (2026-11-24, 1 arquivo). Sincroniza o header de teste com os 19 slots e move o `#pragma pack(pop)` para depois de `XAUDIO2_DEBUG_CONFIGURATION`, de modo que `XAUDIO2_BUFFER`/`VOICE_STATE` fiquem em `pack(1)` sem empacotar os structs seguintes.
  - Arquivos: `tests/external_gdk/gdk_compat_headers/xaudio2_xs.h`.
  - Evidencia: diff de `3cc1984` (12 metodos virtuais inseridos na posicao correta) contra `include/openscarlett/audio/xaudio2_abi.h` em HEAD, que ja tinha a sequencia completa; `git grep -c 'XA2_COMPAT_CALL'` no header de teste em HEAD = 53 ocorrencias, consistente com a vtable cheia.
  - Estado atual: corrigido.
  - Confianca: CONFIRMADA.

- **D-05** A chave do cache de shader em disco nao contem a versao do tradutor
  - Sintomas: `ComputeShaderCacheKey` e um hash de `(blob do shader, perfil alvo, flags de shader model, kShaderCacheFormatVersion)`. A saida do tradutor (HLSL/DXIL) **nao** entra na chave. Toda vez que a semantica de lowering muda — e ela mudou dezenas de vezes entre 22 e 26/11 — o mesmo blob de entrada continua produzindo a mesma chave, e um `shader_cache/` sobrevivente de uma execucao anterior serve o bytecode antigo. O cache e ligado por padrao (`%TEMP%\openscarlett_shader_cache`).
  - Causa raiz: o cache foi desenhado como cache de *entrada* (conteudo do shader) e nao como cache de *traducao*; nao existe constante de revisao do lifter na chave. `kShaderCacheFormatVersion` continua `1u` desde a criacao e so foi tocado por um commit (`git log -S` retorna apenas `538e904`), apesar de correcoes que mudam o codigo gerado (`b616d32` VOP2 cndmask/`s_endpgm`, `f8e24d4` MUBUF, `6fb3502` `v_cmpx`, `7220ce9` `v_cmp_class`).
  - Introduzido em: `538e904` (2026-11-21).
  - Tentativas de correcao: `d7f1b7d` diz "harden shader cache SM key" na mensagem, mas o diff **nao toca** `shader_cache.cpp`; muda apenas o teste para usar o shader model do **device efetivo** em vez do probe de hardware (que era a causa real da flakiness no CI), mais a degradacao headless do `PresentX` (D-09).
  - Correcao definitiva: **SEM CORRECAO**. Em HEAD `include/openscarlett/gpu/shader_cache.h:21` continua `constexpr uint32_t kShaderCacheFormatVersion = 1u;`.
  - Arquivos: `include/openscarlett/gpu/shader_cache.h`, `src/gpu/shader_cache.cpp` (`ComputeShaderCacheKey`, `VerifyHeader`), `src/gpu/d3d12_xs/device_x.cpp` (cache in-process por `stage_blob_hash ^ (sm<<33) ^ modo`).
  - Evidencia: `git grep -n 'kShaderCacheFormatVersion'` retorna definicao + 5 usos, todos com o mesmo valor; `git log -S'kShaderCacheFormatVersion = '` retorna so `538e904`; a lista de commits que mudam a emissao de HLSL mas nao o header de cache e longa (todos os `feat(gpu)` de 24 a 26/11).
  - Estado atual: **aberto**. Qualquer usuario que rode duas versoes do OpenScarlett sobre o mesmo `%TEMP%` pode receber DXIL pre-fix sem nenhum aviso. Nao existe teste que force o bump.
  - Confianca: CONFIRMADA para o mecanismo (chave e ausencia de bump); PROVAVEL para o sintoma (nao ha relato de renderizacao corrompida por cache no repositorio).

- **D-06** O gerador de status le o ultimo `LastTest.log` e publica "6/6 testes"
  - Sintomas: `4af2702` ("avoid AMD RDNA2 v_mul_lo_u16 mask elision") altera o README de **`66/66` para `6/6`** e o KPI de `docs/index.html` de `66/66` para `6/6`. O numero publicado de testes automatizados caiu por um fator de 11 porque a ultima execucao local de CTest foi um subconjunto (`build.cmd --test <regex>`, pratica recomendada pelo proprio `AGENTS.md` §3).
  - Causa raiz: `tools/generate_status.py:47` (`read_ctest`) abre `build/Testing/Temporary/LastTest.log` e conta as linhas `Test Passed/Failed` **daquela execucao**. Nao le a lista de testes registrados no CMake, nao valida contra o total esperado e nao distingue execucao completa de filtrada.
  - Introduzido em: o gerador existe desde `54e4fe1`; o defeito se materializou em `4af2702` (2026-11-23).
  - Tentativas de correcao: `f350622` (2026-11-23) republica `67/67`, mas apenas rodando a suite inteira antes de gerar — o mecanismo permanece.
  - Correcao definitiva: **SEM CORRECAO**. O gerador em HEAD e o mesmo `read_ctest` sobre `LastTest.log`.
  - Arquivos: `tools/generate_status.py`, `README.md`, `docs/index.html`, `docs/status.svg`.
  - Evidencia: diff de `4af2702` (as duas linhas `66/66` -> `6/6`); diff de `f350622` (`6/6` -> `67/67`); leitura de `read_ctest` em HEAD.
  - Estado atual: **aberto**. Os `89/89` e `90/90` de hoje tem a mesma fragilidade: sao o resultado da ultima execucao local, nao uma propriedade do codigo.
  - Confianca: CONFIRMADA.

- **D-07** As familias do dashboard somam 1.211 "feitos" contra 779 "totais", e o numero de opcodes verificados e um literal no template
  - Sintomas: em `docs/index.html` (gerado), `FAMILIES` traz `VOP 476/330`, `SOP 206/130`, `MIMG 99/85`, `FLAT 138/70`, `DS 125/70`, `MUBUF 95/65`, `SMEM 71/28`, `EXP 1/1`. Soma dos `done` = **1.211**; soma dos `total` = **779**. Sete das oito barras passam de 100% e o grafico so nao mostra isso porque o percentual e `min(100, ...)`.
  - Causa raiz: `tools/generate_status.py:114-122` conta `done` com um predicado de **mnemonicos** (`m.startswith("v_")`, etc.) e compara com um `total` de **opcodes** fixo por familia. As duas unidades nunca foram iguais: um opcode pode ter varios mnemonicos com sufixo (`_dpp`, `_sdwa`, `_e64`), e a classificacao por prefixo conta cada um.
  - Introduzido em: `7ba25f7` (2026-11-22, o commit que introduz `measure_rdna2_families` e os "honest progress bars"); a divergencia cresce a cada expansao de ISA.
  - Tentativas de correcao: nenhuma.
  - Correcao definitiva: **SEM CORRECAO**.
  - Arquivos: `tools/generate_status.py`, `docs/index.html`, `docs/matrix.svg`.
  - Evidencia: `git show HEAD:docs/index.html` (a linha `const FAMILIES = [...]`) e a leitura do gerador. O mesmo arquivo contem, no template, o literal `775/779 real-GPU verified` e `4/4 CPU privileged trap/halt harness` — ou seja, **o numero de opcodes verificados em hardware nao e medido por nada**, e digitado no gerador (`generate_status.py:408`).
  - Estado atual: **aberto**; o dashboard e a fonte declarada dos "honest progress" do README.
  - Confianca: CONFIRMADA.

- **D-08** O cabecalho escrito a mao do README diverge do bloco auto-gerado
  - Sintomas: a linha 22 do README diz `89/89` testes e `1,360` exports em 33 DLLs; a linha 122, dentro do bloco `<!-- OPENSCARLETT_STATUS_START -->`, diz `90/90` e `1372` exports. O numero real de registros de teste e 90.
  - Causa raiz: o bloco gerado e reescrito por `generate_status.py`, mas o paragrafo de abertura e manual e nao tem nenhuma verificacao; `AGENTS.md` §4 manda "regenerate status badges after every milestone", o que atualiza o bloco e deixa o texto acima para tras.
  - Introduzido em: incremental, a cada milestone; visivel em HEAD.
  - Tentativas de correcao: `b9d988c`, `87cdba6`, `c7a7e46`, `bd6e9d8` atualizam o texto, cada um deixando um numero novo defasado.
  - Correcao definitiva: **SEM CORRECAO**.
  - Arquivos: `README.md` linhas 22 e 122.
  - Evidencia: `git grep -n '89/89'` e `git grep -n '90/90'` no README; contagem independente de registros (`git grep -c 'openscarlett_add_test('` = 88, mais 2 `add_test` explicitos em `CMakeLists.txt`).
  - Estado atual: **aberto** (inconsistencia de 1 teste e 12 exports entre dois paragrafos do mesmo arquivo).
  - Confianca: CONFIRMADA.

- **D-09** Teste que nao pode rodar reporta `Passed`, e nao existe `SKIP_RETURN_CODE` em lugar nenhum
  - Sintomas: em runner sem adaptador D3D12, testes imprimem `SKIP: No D3D12 adapter available` e fazem `return;` da funcao de teste; `main` termina com 0 e o CTest registra **Passed**. Nao ha uma unica propriedade `SKIP_RETURN_CODE` nos arquivos de build (`git grep` em todos os `*.cmake` e `CMakeLists.txt`: 0 ocorrencias). Logo "90/90 passing" nao distingue "passou" de "nao rodou".
  - Causa raiz: convencao de skip implementada dentro de cada teste (early return) em vez de um codigo de retorno dedicado + propriedade CTest; a suite foi crescendo com testes de hardware e nenhum deles ganhou o marcador.
  - Introduzido em: distribuido, desde os primeiros testes de GPU (`d8518c2`, `529e0a6`).
  - Tentativas de correcao: nenhuma estrutural. `f99d57e` (2026-11-23) remove um `OS_REQUIRE(hw_mesh)` que falhava em Pascal, mas mantem o padrao de "degrada e conta como passe".
  - Correcao definitiva: **SEM CORRECAO**.
  - Arquivos: `tests/test_minecraft_perf_hardening.cpp:390-394` e outros pontos com `[INFO] ... skipping` (`test_d3d12_xs_live.cpp`, `test_s13_vk_xg_tiled.cpp`, `test_unified_mem_bindless.cpp`); `cmake/build_speedups.cmake`; `CMakeLists.txt`.
  - Evidencia: `git grep -c SKIP_RETURN_CODE -- '*.cmake' 'CMakeLists.txt'` = 0; leitura do `return;` em `test_minecraft_perf_hardening.cpp`.
  - Estado atual: **aberto**. Os relatorios "67/67 PASS" por PC e os "90/90" do CI tem o mesmo vicio.
  - Confianca: CONFIRMADA para o mecanismo; o efeito exato no numero publicado depende de quantos testes degradam naquela maquina (NAO VERIFICADO).

- **D-10** `PresentX` degrada para headless mascarando falha de criacao de swapchain (e a mensagem fala de outra coisa)
  - Sintomas: quando `CreateSwapChainForHwnd` falha com `DXGI_ERROR_UNSUPPORTED`, `DXGI_ERROR_NOT_CURRENTLY_AVAILABLE` ou `E_ACCESSDENIED`, `PresentX` passa a **fingir sucesso**: `do_swapchain_present = false; hr = S_OK;` e o frame segue sem apresentar. Um titulo real numa maquina com esse erro passa a "rodar" sem janela e sem erro, e o unico sinal fica num log de fila. Ao mesmo tempo, `should_pace` foi trocado de `do_swapchain_present` para `(use_swapchain && !window_minimized)`, entao o frame continua sendo **paced** sem present.
  - Causa raiz: correcao de CI (Session-0 sem DWM) escrita dentro do caminho de produto, sem flag de ambiente e sem telemetria de degradacao; o assunto do commit fala de "shader cache SM key", que e o que o diff faz **no teste**, nao no produto.
  - Introduzido em: `d7f1b7d` (2026-11-22).
  - Tentativas de correcao: nenhuma. Em HEAD o comportamento persiste (`git grep 'DXGI_ERROR_NOT_CURRENTLY_AVAILABLE'` em `command_queue_x.cpp`).
  - Correcao definitiva: **SEM CORRECAO**.
  - Arquivos: `src/gpu/d3d12_xs/command_queue_x.cpp` (`PresentX`).
  - Evidencia: diff de `d7f1b7d` (o bloco `degrade cleanly to headless present instead of failing the frame` e a troca de `should_pace`); `git show d7f1b7d --stat` mostrando que o commit toca `command_queue_x.cpp` + 3 testes e **nenhum** arquivo de cache de shader.
  - Estado atual: **aberto**, com risco de mascarar regressao de apresentacao em hardware real.
  - Confianca: CONFIRMADA.

- **D-11** Um merge derrubou o wrap do `ScarlettRetailDeviceProxy` e ele teve de ser restaurado
  - Sintomas: em `cbfd4f1` ("Merge branch 'feat/ue5-matrix-awakens-imports' into feat/minecraft-preview-perf") o parametro `bool retail_scarlett_vtable` **desaparece** de `CreateWrappedDeviceInternal` e o wrap deixa de ser aplicado, embora a classe `ScarlettRetailDeviceProxy` continue no arquivo. Consequencia: `D3D12XboxCreateDevice` com `Version >= 0x20000` devolvia o `DeviceX` cru, com vtable de desktop, para um titulo que espera a vtable retail — todos os slots a partir de 43 apontariam para funcoes erradas.
  - Causa raiz: resolucao de conflito de merge escolhendo o lado do ramo UE5, que tinha reescrito a mesma funcao (`acc9bc8` "export D3D12 COM IID GUID data symbols e D3D12Xbox process/device helpers") sem conhecer a mudanca de `09c65d1`.
  - Introduzido em: `cbfd4f1` (2026-11-23). O primeiro pai (`74c3a5c`) tem o parametro; o resultado nao tem.
  - Tentativas de correcao: `f2c0610` (mesmo dia), com a mensagem "restore ScarlettRetailDeviceProxy wrapping in D3D12XboxCreateDevice **after UE5 merge**" — o autor sabia da origem.
  - Correcao definitiva: `f2c0610` (2026-11-23): reintroduz o parametro, o `SetRetailScarlettAbi(true)`, o `new ScarlettRetailDeviceProxy(dev_x)` para `ID3D12Device`/`IUnknown` e ainda devolve `EnvEnablesD3D12Debug()` ao calculo de `debug_layer` (que tambem tinha sumido).
  - Arquivos: `src/shims/d3d12_xs_shim.cpp`.
  - Evidencia: comparacao textual do arquivo em `74c3a5c` (tem `bool retail_scarlett_vtable`), `004fc58` (nao tem nem o parametro nem a classe) e `cbfd4f1` (tem a classe, nao tem o uso); `git log -S'retail_scarlett_vtable'` mostra `09c65d1` (adiciona) e `f2c0610` (readiciona) — a remocao so aparece em merge, que `-S` nao lista sem `-m`.
  - Estado atual: corrigido; parametro presente em HEAD.
  - Confianca: CONFIRMADA.

- **D-12** Espera sincrona de fence no caminho de submissao (quatro paradas de 50 ms por frame)
  - Sintomas: `CommandQueueX::Signal`/`ExecuteCommandLists` faziam `WaitForInternalFenceLocked()` e, no caminho de diagnostico, um `Signal` + `SetEventOnCompletion` + `WaitForSingleObject(5000)` por submissao. Medido pelo autor: **5,0 FPS** no titulo retail; apos a correcao, 161-250 FPS a 720p e 57-104 FPS a 4K.
  - Causa raiz: o sincronismo era usado como rede de seguranca para (a) garantir que a GPU terminou antes de escrever fences/valores na CPU e (b) permitir que `GetDeviceRemovedReason` apontasse a lista culpada. As duas coisas foram implementadas como espera bloqueante no caminho quente.
  - Introduzido em: o padrao aparece desde `538e904` e foi endurecido em `ee10610` e `5ce3bc5`.
  - Tentativas de correcao: `5ce3bc5` (ring assincrono de `PresentX`, "disable retail per-frame readback") aliviou parte; `afcd167` mexeu em pacing.
  - Correcao definitiva: `022a8b2` (2026-11-25): `FenceWaitDispatcher` atualiza a palavra de fence na CPU de forma assincrona; as escritas diferidas vao para um fence proprio (`deferred_write_fence_`); o drain sincrono de diagnostico passa a exigir `OPENSCARLETT_SYNC_DIAG_DRAIN` e o caminho sincrono de writes, `OPENSCARLETT_SYNC_DEFERRED_WRITES`.
  - Arquivos: `src/gpu/d3d12_xs/command_queue_x.cpp`, `command_queue_x.h`.
  - Evidencia: diff de `022a8b2` (o `ExecProfGuard`, o `commit_writes` assincrono, os dois `EnvFlagTrueA`); a mensagem declara os FPS, que sao **numero do autor, nao reproduzido aqui**.
  - Estado atual: corrigido no caminho padrao; os caminhos sincronos continuam existindo atras de flag (bom: preservam a capacidade de diagnostico).
  - Confianca: CONFIRMADA para o mecanismo e para a remocao do stall; PROVAVEL para os numeros de FPS.

- **D-13** Footprint nao alinhado do `CopyTextureRegion` do Xbox era passado direto ao D3D12 de desktop
  - Sintomas: o Xbox Series permite `PlacedFootprint.Offset` alinhado a 64/256 bytes e `RowPitch` menor que 256; o desktop exige 512 (`D3D12_TEXTURE_DATA_PLACEMENT_ALIGNMENT`) e 256 de pitch. O shim repassava o descritor como veio: `CopyTextureRegion` falhava (ou pior, copiava com pitch errado) em qualquer upload de textura cujo footprint nao fosse multiplo de 512.
  - Causa raiz: nenhuma camada de traducao entre as regras de alinhamento das duas plataformas; o descritor era considerado "o mesmo struct".
  - Introduzido em: `d58f0b6` (2026-11-21, primeiro `command_list_x.cpp`).
  - Tentativas de correcao: nenhuma antes do fix.
  - Correcao definitiva: `f7566aa` (2026-11-23): detecta `bad_offset`/`bad_pitch`, calcula `num_rows` com altura de bloco para BC, alinha o pitch para 256, dimensiona um staging de 64 KiB e faz o upload via `CopyBufferRegion` (que aceita offset de 1 byte) antes do `CopyTextureRegion` alinhado.
  - Arquivos: `src/gpu/d3d12_xs/command_list_x.cpp`, `command_list_x.h`.
  - Evidencia: diff de `f7566aa` (o bloco `bad_offset || bad_pitch` e o staging).
  - Estado atual: corrigido.
  - Confianca: CONFIRMADA.

- **D-14** Listas com `Close()` falho eram descartadas sem honrar as escritas de CPU diferidas
  - Sintomas: `ExecuteCommandLists` passou a pular listas invalidas (`if (clx->LastCloseFailed()) continue;`) — correto para nao entregar lixo ao host, mas o `continue` tambem abandonava as escritas de fence/marcador que o **CPU** esperava. Um `Wait` de CPU sobre um valor que so seria escrito por aquela lista ficava preso para sempre.
  - Causa raiz: o fix de robustez (D-13, mesmo commit) tratou a lista invalida como "nada a fazer", sem separar o que e GPU (descartavel) do que e CPU (obrigatorio).
  - Introduzido em: `f7566aa` (2026-11-23).
  - Tentativas de correcao: nenhuma intermediaria.
  - Correcao definitiva: `022a8b2` (2026-11-25): antes do `continue`, chama `clx->ExtractDeferredCpuOps(deferred_writes, deferred_waits)` e `clx->ExtractTransientResources(...)`, com o comentario "Still honor CPU fence/marker writes & retirements so CPU wait loops never stall".
  - Arquivos: `src/gpu/d3d12_xs/command_queue_x.cpp`, `command_list_x.cpp`.
  - Evidencia: o `continue` de `f7566aa` e a insercao de `ExtractDeferredCpuOps` em `022a8b2` (trecho visivel no diff deste ultimo).
  - Estado atual: corrigido.
  - Confianca: CONFIRMADA.

- **D-15** O orcamento de residencia era cumprido evictando heaps vivos
  - Sintomas: `PrepareResidencyBudgetForAllocation` chamava `EvictLruVictimsLocked` sempre que o total rastreado passava do budget. `ID3D12Device::Evict` em recurso que o titulo ainda usa nao e um aviso: o WDDM pagina de volta no meio do comando e o resultado e `DXGI_ERROR_INVALID_CALL` / `DXGI_ERROR_DEVICE_REMOVED`.
  - Causa raiz: tratar um budget **sintetico** (numero calculado pelo shim) como se fosse um limite imposto pelo driver, e "resolver" o excesso despejando recursos em uso em vez de recusar a alocacao ou deixar o WDDM paginar.
  - Introduzido em: `a92359c` (2026-11-23, "Sprint 14 ... live VRAM residency").
  - Tentativas de correcao: nenhuma antes do fix.
  - Correcao definitiva: `d71ed46` (2026-11-23): eviccao automatica **so** quando ha override explicito (`residency_budget_override_bytes_ > 0`), que e o caso dos testes; em gameplay o WDDM pagina sozinho e o spill L0 cobre o resto. O commit aproveita para instrumentar `DEVICE REMOVED right after ExecuteCommandLists` (opt-in).
  - Arquivos: `src/gpu/d3d12_xs/device_x.cpp`, `command_queue_x.cpp`, `src/shims/gameinput_shim.cpp` (mudanca nao relacionada, ver H-05).
  - Evidencia: diff de `d71ed46` (a condicao `residency_budget_override_bytes_ > 0ull` e o comentario sobre WDDM); `git log -S'EvictLruVictimsLocked'` mostra `a92359c` como introducao.
  - Estado atual: corrigido.
  - Confianca: CONFIRMADA.

- **D-16** A regra de passthrough de caminho de host foi aplicada API a API, nao como regra
  - Sintomas: cadeia de tres dias. `990c3ff` (22/11) fez `CreateFileW` passar adiante caminhos que nao sao virtuais (`VfsTranslateStatus::NotVirtual`), mas `GetFileAttributesW` e `CreateDirectoryW` continuaram traduzindo tudo para dentro do overlay. Resultado: o titulo cria o mundo LevelDB, `CreateDirectory` "da certo" no VFS e `GetFileAttributes` no caminho real responde "nao existe" — a criacao de mundo do Minecraft falhava. Depois disso, `be0fda1` (23/11) precisou hookar as APIs de CRT (`fopen`, `_wfopen`) para o mesmo motivo, e `4bc2db4` (23/11) precisou de um guard em `GetFinalPathNameByHandleW`.
  - Causa raiz: a decisao "este caminho e virtual?" mora em `ShouldPassThroughHostDDriveW`/`TranslateForAccessW`, mas cada API hookada reimplementa o fluxo; a regra existe uma vez e o **uso** da regra existe em N lugares. Cada API nova ou esquecida vira um bug de comportamento.
  - Introduzido em: `990c3ff` (2026-11-22) — o fix que acertou `CreateFile` e deixou as outras APIs para tras.
  - Tentativas de correcao: `e7c5f85` (23/11, passthrough em `GetFileAttributes`/`CreateDirectory` + `TraceVfsFailureW`), `be0fda1` (23/11, CRT), `4bc2db4` (23/11, `GetFinalPathNameByHandleW`), `d25a849` (23/11, caminho 8.3 — ver D-17).
  - Correcao definitiva: **parcial e incremental**; em HEAD a regra continua espalhada por `hook_dll.cpp` (7.738 linhas) em pontos distintos por API.
  - Arquivos: `src/loader/hook_dll.cpp`, `src/loader/vfs_mounts.cpp`.
  - Evidencia: os quatro diffs; o comentario de `e7c5f85` nomeando o sintoma ("unblocks Minecraft LevelDB world creation"); `git log -S'NotVirtual' -- src/loader/hook_dll.cpp`.
  - Estado atual: funcional para as APIs cobertas; estruturalmente aberto para qualquer API nova.
  - Confianca: CONFIRMADA.

- **D-17** `E:\OpenScarlett` compilado no produto e testes que exigem "fora de `C:`"
  - Sintomas: `ResolveDefaultWorkDirW` recusa qualquer caminho no drive do sistema e, quando nao acha alternativa gravavel, retorna a constante `kDefaultWorkDirFallbackW = L"E:\\OpenScarlett\\.tmp\\openscarlett_run"` — o caminho absoluto da maquina do autor, dentro do binario. Em paralelo, os testes passaram a **afirmar** a politica: `test_s16_os_vfs_gdk_shims` exigia `!StartsWithDriveCW(work_dir)` e `test_loader_vfs_config` esperava o caminho derivado do anchor. Numa maquina cujo build esta em `C:` (o runner do GitHub, ou qualquer PC comum), os dois testes falham por construcao.
  - Causa raiz: uma preferencia de higiene de disco da maquina de desenvolvimento (`AGENTS.md` §2: "ALL builds, caches & temp files strictly on `E:\` — NEVER write ... to `C:\`") foi implementada como **regra de produto** e depois **codificada em assercao de teste**, sem separar "onde eu quero escrever" de "o que e correto".
  - Introduzido em: politica em `7e2dd9b` (2026-11-23, `AGENTS.md` + scripts); assercoes nos testes junto com os testes.
  - Tentativas de correcao: `91364e4` (22/11, guards de bundle `C:\`), `438fdb0` (23/11, PR #40, "let the work-dir checks pass when the build lives on C:"), `4e350ba` (x64 prompt), `d25a849` (8.3).
  - Correcao definitiva: **parcial**. `438fdb0` faz os testes aceitarem o fallback `%TEMP%` quando o sandbox esta em `C:` e so exigirem "sem `C:`" quando os binarios estao fora. A constante `E:\...` continua em `src/loader/vfs_mounts.cpp:15` e continua sendo o retorno final quando `TEMP` esta vazio.
  - Arquivos: `src/loader/vfs_mounts.cpp`, `tests/test_loader_vfs_config.cpp`, `tests/test_s16_os_vfs_gdk_shims.cpp`, `AGENTS.md`, `build.cmd`.
  - Evidencia: `git grep -n 'kDefaultWorkDirFallbackW'` (definicao + 3 usos em HEAD); diff de `438fdb0` (o `c_temp_fallback` e o `sandbox_off_c`); diff de `7e2dd9b` no `AGENTS.md`.
  - Estado atual: **aberto**. Num host sem `E:` e sem `TEMP`/`TMP`, a funcao devolve um caminho em um drive inexistente.
  - Confianca: CONFIRMADA.

- **D-18** Habilitacao de `ctest --parallel 4` quebrou o benchmark de wall-clock
  - Sintomas: depois que `build.cmd` passou a rodar `ctest --parallel 4` com quatro jobs disputando CPU/GPU, `test_s15_rdna2_bench_trap_harness` (que impoe limite de tempo) passou a falhar de forma intermitente por contencao, nao por regressao.
  - Causa raiz: um teste cujo oraculo e **tempo de parede** foi deixado no pool paralelo. A suite cresceu para 90 testes disputando 8 threads e uma GPU.
  - Introduzido em: `7e2dd9b` (2026-11-23).
  - Tentativas de correcao: `1ffd939` (2026-11-23) marca o teste `RUN_SERIAL TRUE`.
  - Correcao definitiva: `1ffd939`; o padrao foi generalizado em `cmake/build_speedups.cmake` (`RESOURCE_LOCK window`, `audio`, `vfs_saves`; `RUN_SERIAL` para benchmarks), que e uma solucao razoavel.
  - Arquivos: `cmake/build_speedups.cmake`, `build.cmd`, `build_wt.cmd`.
  - Evidencia: diff de `7e2dd9b` (o `--parallel 4` e o arquivo novo `build_speedups.cmake`) e de `1ffd939` (uma linha).
  - Estado atual: corrigido.
  - Confianca: CONFIRMADA.

- **D-19** Overflow de microssegundos do QPC congela timestamps depois de ~10,67 dias de uptime
  - Sintomas: `(now.QuadPart * 1000000LL) / s_freq.QuadPart` em `LONGLONG` (com sinal) estoura quando `ticks > 2^63/10^6`. Com QPC de 10 MHz isso acontece em ~10,67 dias de uptime do host. A partir dai os timestamps do GameInput e os temporizadores de reprobe de slot ficam negativos/presos: controle "desconecta" e nao volta.
  - Causa raiz: conversao ingenua de ticks para microssegundos com multiplicacao antes da divisao, em tipo com sinal, num driver de dispositivo que roda por dias.
  - Introduzido em: `a429535` (2026-11-21, `gameinput_shim.cpp`).
  - Tentativas de correcao: `afcd167` mexeu em polling e recuperacao de Alt-Tab, sem tocar a conversao.
  - Correcao definitiva: `05a24bc` (2026-11-23): `QpcTicksToMicros(ticks, freq)` decomposto (segundos + resto) sem overflow; o teste `test_s12_os_loader_gameinput` ganha caso de uptime alto.
  - Arquivos: `src/shims/gameinput_shim.cpp`, `tests/test_s12_os_loader_gameinput.cpp`.
  - Evidencia: `git grep -n 'QpcTicksToMicros'` em HEAD (definicao + chamada) e o diff de `05a24bc`.
  - Estado atual: corrigido.
  - Confianca: CONFIRMADA para o mecanismo; o instante exato depende do `QueryPerformanceFrequency` do host.

- **D-20** Topologia primitiva console-only entregue ao runtime de desktop trava o pushbuffer
  - Sintomas: `IASetPrimitiveTopology(17/18/19/20)` (LINELOOP/RECTLIST/etc. do console) chegava ao `ID3D12GraphicsCommandList` do host. Em vez de `E_INVALIDARG`, a NVIDIA devolvia `DXGI_ERROR_DEVICE_HUNG` (0x887A0006) e o dispositivo ficava removido. Sem debug layer, nada indicava a causa.
  - Causa raiz: o shim repassava enums de topologia sem validar contra a faixa legal do D3D12 de desktop; a premissa era que "topologia e topologia".
  - Introduzido em: `d58f0b6` (2026-11-21, primeiro `command_list_x.cpp`), exposto quando um titulo retail passou a desenhar com RECTLIST.
  - Tentativas de correcao: `82bd3fa`/`c82ca43` (23/11) tentaram limpar o caminho sem atacar a origem; `8d285d9` e `2241564` mexeram em PSO/Vulkan do mesmo sintoma em outras GPUs.
  - Correcao definitiva: `0cbc742` (2026-11-24): em `IASetPrimitiveTopology`, quando `IsRetailScarlettAbi()`, valida a faixa (`1..5`, `10..13`, `33..64`) e remapeia o resto para `TRIANGLELIST`, com log limitado a 5 ocorrencias.
  - Arquivos: `src/gpu/d3d12_xs/command_list_x.cpp`, `device_x.cpp` (mesmo remapeamento no parse do stream de PSO).
  - Evidencia: diff de `0cbc742`; em HEAD ha dois pontos de remapeamento (`command_list_x.cpp:2537` e `device_x.cpp:10617`).
  - Estado atual: corrigido para topologia. Nota: o comentario do codigo diz `19=RECTLIST, 18=QUADLIST, 17=LINELOOP`, enquanto o README diz `17=RECTLIST->TRIANGLELIST` (ver H-07).
  - Confianca: CONFIRMADA para o remapeamento; o diagnostico "trava o pushbuffer NVIDIA" e do autor (nao reproduzido aqui), mas e coerente com o `0x887A0006` citado.

- **D-21** `ds_sub` em LDS emitido como `InterlockedAdd` de valor negado — driver NVIDIA recusa o pipeline
  - Sintomas: em RTX 5070 (driver 621.24), `CreateComputePipelineState` retornava `E_OUTOFMEMORY` para **qualquer** compute shader com `InterlockedAdd` em groupshared cujo operando e negacao de runtime. DXC dobra toda grafia de `-x` (`0u - x`, `~x + 1u`, `x * 0xFFFFFFFF`, `asuint(-asint(x))`) em `sub 0, x`. O mesmo padrao em UAV era aceito. O kernel `mem_ds_atomics` falhava em GPU real e derrubava `test_rdna2_corpus_bench` e `test_rdna2_gpu_accuracy`.
  - Causa raiz: lowering escolheu a forma mais curta (`InterlockedAdd(addr, 0u - v)`) sem que nenhum teste rodasse em driver AMD/NVIDIA especifico; o bug e do par DXC+driver, mas o shim e quem decide a forma emitida.
  - Introduzido em: o lowering de `ds_sub` no lifter (arquivo `rdna2_isa_mem_img.cpp`).
  - Tentativas de correcao: nenhuma antes do PR.
  - Correcao definitiva: `5d3809b` (2026-11-22, PR #26 de Felipe Marquezini): `ds_sub` em LDS passa pelo mesmo laco `compare-exchange` que `ds_inc`/`ds_dec`/min/max assinados ja usavam; caminho de buffer/UAV tipado fica inalterado.
  - Arquivos: `src/gpu/rdna2_isa_mem_img.cpp`.
  - Evidencia: diff de 4 linhas + o comentario que registra driver e versao; `git grep -n 'emit_cas_loop'` no arquivo.
  - Estado atual: corrigido **so para LDS**. O caminho UAV continua emitindo `InterlockedAdd(..., (0u - v))` (`else` no mesmo `case 0x33`), por decisao declarada de que UAV nao e afetado.
  - Confianca: CONFIRMADA para o diff e o sintoma relatado; o comportamento do driver em outra versao NAO VERIFICADO.

- **D-22** `notify_one`/`notify_all` fora do lock no DirectStorage — use-after-free
  - Sintomas: `IocpReadOp` e alocado na pilha do chamador; o worker marcava `completed = true` sob `op->mutex`, **saia do lock** e so entao chamava `op->cv.notify_one()`. O leitor podia observar `completed` e destruir o objeto antes do `notify_one` retornar → escrita em `condition_variable` liberada. Mesmo padrao em `CloseInternal` com `close_cv_` e o proprio `DStorageQueueCore`.
  - Causa raiz: padrao "sinalize fora do lock" (que e a recomendacao geral de performance) aplicado a um objeto cuja vida termina no despertar do waiter.
  - Introduzido em: `a429535` (2026-11-21) e endurecido nos commits de fila (`a79fd8d`, `ed0ad71`).
  - Tentativas de correcao: `16c816d` serializou o dispatch de barreiras contra barreiras em voo (problema vizinho, nao este).
  - Correcao definitiva: `a208562` (2026-11-23): `notify_one`/`notify_all` movidos para **dentro** do lock (`op->mutex`, `mutex_`) com comentarios explicando a janela; `CloseInternal` ganha `weak_from_this().lock()` como keepalive.
  - Arquivos: `src/shims/dstorage_shim.cpp`.
  - Evidencia: diff de `a208562` (11 insercoes, 4 remocoes, tres `notify` movidos e um `self_keepalive`).
  - Estado atual: corrigido nesses tres pontos. Nao ha varredura equivalente nos outros shims (G-06).
  - Confianca: CONFIRMADA.

- **D-23** Mutex da telemetria mantido durante a chamada para a hook DLL
  - Sintomas: `CallTelemetry::Record` pegava `mutex_`, atualizava a entrada e chamava `ForwardToHookDllIfPresent` **com o lock ainda tomado**. A hook DLL, ao processar o evento, pode chamar de volta uma funcao instrumentada (ou logar), reentrando em `Record` com `std::mutex` nao recursivo → deadlock do processo; no melhor caso, contencao global em toda chamada de API console.
  - Causa raiz: o encaminhamento cross-DLL foi adicionado ao mesmo bloco critico da contabilidade, sem separar "atualizar estado" de "notificar terceiro".
  - Introduzido em: `acbdb7c` (2026-11-21, "cross-DLL telemetry forwarding").
  - Tentativas de correcao: nenhuma antes do fix.
  - Correcao definitiva: `2b42f72` (2026-11-23): bloco de escopo para o `lock_guard`; o `ForwardToHookDllIfPresent` passa a rodar fora do lock.
  - Arquivos: `src/core/telemetry.cpp`.
  - Evidencia: diff de `2b42f72` (o `{ ... }` em volta do lock) e `git log -S'ForwardToHookDllIfPresent'` (introducao em `acbdb7c`).
  - Estado atual: corrigido.
  - Confianca: CONFIRMADA para o mecanismo; o deadlock observado em campo NAO VERIFICADO.

- **D-24** `XApuGetResultCount` devolvia a contagem em `EAX` e travava a inicializacao
  - Sintomas: o titulo chamava `XApuGetResultCount(handle, 0)` e esperava a contagem no registrador de retorno; o shim respondia por outro caminho, a contagem ficava "pendente" e o loop do titulo girava **2,14 bilhoes** de vezes segurando o mutex global de audio, parando a inicializacao.
  - Causa raiz: convencao de retorno da ABI de console (`EAX`) implementada como se fosse retorno por ponteiro/`HRESULT`, num shim sem cabecalho de referencia.
  - Introduzido em: o shim `xapu` original (`a429535`/`ed0ad71`) e as versoes reais em `b08fa23`.
  - Tentativas de correcao: `1c56104` (rollback de OOM/SHAPE do AcpHal) tratou o subsistema vizinho.
  - Correcao definitiva: `ef763eb` (2026-11-24): o contador passa a ser devolvido no registrador de retorno quando o segundo argumento e 0.
  - Arquivos: `src/shims/xapu_shim.cpp`, `src/gpu/d3d12_xs/proxies_x.cpp`, `src/shims/d3d12_xs_shim.cpp`, `src/gpu/d3d12_xs/device_x.cpp`.
  - Evidencia: mensagem e diff de `ef763eb` (32 linhas em `xapu_shim.cpp` com o comentario do deadlock de 2,14B iteracoes e o endereco do mutex).
  - Estado atual: corrigido.
  - Confianca: CONFIRMADA para o diff; a contagem de iteracoes e do autor (NAO VERIFICADA independentemente).

## E. Relacao de causa e efeito

1. `a429535` escreve a ABI do DirectStorage a partir do cabecalho de desktop -> todo campo de `DSTORAGE_REQUEST` e lido no offset errado para um titulo de console -> `76adeff` reescreve o struct e adiciona `static_assert` de tamanho/offset, que e o que impede a repeticao.
2. `a429535` digita `GameInputGamepadMenu = 0x4` em vez de `0x1` -> todo botao e interpretado dois bits acima e o teste de bring-up do GDK passa porque o **cabecalho de teste tem o mesmo erro** -> `3112404` corrige os dois arquivos.
3. `59bd45a` cria um `IXAudio2Voice` de 8 slots no cabecalho de compatibilidade -> o duplo de teste valida o shim contra a vtable errada e nao pode detectar erro de slot -> `3cc1984` sincroniza com os 19 slots reais.
4. `669f0b9` declara `XMemGetWorkingSetStatistics` com um argumento -> titulos chamam com dois e o shim le a palavra de flags como ponteiro -> `675d9f4` troca a assinatura por dois `uintptr_t` e **adivinha** a ABI com `VirtualQuery`, em vez de fixar contrato.
5. `538e904` faz a chave do cache de shader depender so da entrada -> qualquer correcao no lowering (`b616d32`, `f8e24d4`, `6fb3502`, `7220ce9`) deixa bytecode pre-fix valido no disco -> nenhuma correcao; `kShaderCacheFormatVersion` continua `1u`.
6. `7e2dd9b` habilita `ctest --parallel 4` com benchmark de wall-clock no pool -> `test_s15_rdna2_bench_trap_harness` falha por contencao -> `1ffd939` marca `RUN_SERIAL`, e `build_speedups.cmake` generaliza `RESOURCE_LOCK`.
7. `990c3ff` aplica passthrough de caminho de host so em `CreateFileW` -> `CreateDirectory`/`GetFileAttributes` continuam traduzindo e o titulo nao enxerga o diretorio que acabou de criar -> `e7c5f85` estende a regra e adiciona `TraceVfsFailureW`; `be0fda1` e `4bc2db4` estendem de novo, para CRT e `GetFinalPathNameByHandleW`.
8. `a92359c` resolve excesso de budget de VRAM com `Evict` em heaps vivos -> WDDM pagina de volta no meio do comando -> `DEVICE_REMOVED` -> `d71ed46` exige override explicito para evictar.
9. `f7566aa` descarta listas com `Close()` falho com `continue` -> as escritas de fence que o CPU espera nunca acontecem -> `022a8b2` extrai `DeferredCpuOps` antes do `continue`.
10. `d58f0b6` repassa topologia console-only ao host -> NVIDIA trava o pushbuffer (`0x887A0006`) em vez de retornar erro -> `0cbc742` valida a faixa e remapeia para `TRIANGLELIST`; a mesma regra precisa ser replicada no parse do stream de PSO (`device_x.cpp:10617`).
11. `09c65d1` adiciona o wrap do `ScarlettRetailDeviceProxy` -> `004fc58`/`cbfd4f1` (merges do ramo UE5) trazem uma versao da funcao que nao conhece o parametro e o wrap some -> `f2c0610` restaura, inclusive o `EnvEnablesD3D12Debug()` que tambem tinha sumido.
12. `7e2dd9b` transforma "nunca escrever em `C:`" em regra de produto (`ResolveDefaultWorkDirW`) e em assercao de teste -> qualquer host com build em `C:` falha testes corretos -> `438fdb0` aceita o fallback `%TEMP%`, mas a constante `E:\OpenScarlett\...` fica no binario.
13. `a429535` converte QPC com `(now.QuadPart * 1000000LL) / freq` em `LONGLONG` -> overflow em ~10,67 dias de uptime -> `05a24bc` decompoe a conversao.
14. `4af2702` roda a suite filtrada e regenera o status -> `read_ctest` le `LastTest.log` daquela execucao -> README anuncia `6/6` -> `f350622` republica `67/67`, mas o mecanismo continua.
15. `d7f1b7d` liga a degradacao de `PresentX` para headless (D-10) no mesmo commit que conserta a chave de shader no teste -> o produto passa a esconder falha de swapchain e a mensagem nao descreve o diff.

## F. Correcoes importantes

- **F-01 -- `76adeff` (ABI do DirectStorage com `static_assert`).** O valor nao esta em acertar os offsets, e sim em **tornar o erro impossivel de compilar**: `sizeof`/`offsetof` verificados no build. Todo shim que declara struct de console deveria ter esse par de assercoes; sem elas, o desvio so aparece quando um titulo real enfileira I/O. Reutilizavel em qualquer camada que traduza ABI (`dstorage`, `xaudio2`, `gameinput`, `xmem`).
- **F-02 -- `675d9f4` (deteccao de ABI em vez de recusa).** Discutivel como engenharia (ver G-05), mas didatico: mostra que **a alternativa a uma assinatura correta e uma heuristica que pode errar**, e que a decisao precisa ser explicita no codigo. O comentario descreve o chamador real (flags `0x20000001`, struct de 32 bytes nao inicializado) e e a melhor documentacao de ABI do repositorio.
- **F-03 -- `022a8b2` (assincronia de fence com caminho sincrono preservado).** O acerto e nao remover a espera: ela vira `OPENSCARLETT_SYNC_DIAG_DRAIN`/`OPENSCARLETT_SYNC_DEFERRED_WRITES`. Diagnostico de `DEVICE_REMOVED` continua possivel, e o caminho quente deixa de pagar 200 ms por frame. Reutilizavel em qualquer shim que precise de "modo confiavel" e "modo rapido" no mesmo binario.
- **F-04 -- `d71ed46` (nao evictar o que o titulo usa).** Separa budget **sintetico de teste** de budget **real de driver**. A regra "so evictar com override" e a formulacao correta: um limite que voce inventou nao autoriza destruir estado de terceiro.
- **F-05 -- `f7566aa` (traduzir regra de alinhamento entre plataformas).** Xbox 64/256 vs desktop 512/256. O padrao -- detectar o caso ilegal, alinhar em staging e usar a API que aceita granularidade fina (`CopyBufferRegion`) -- vale para qualquer par console/desktop com regras de pitch diferentes.
- **F-06 -- `5d3809b` (contorno de bug de compilador no lowering).** Quando o compilador do host dobra uma expressao para uma forma que o driver recusa, a saida e escolher uma forma que ele nao dobre (laco CAS), nao tentar outra grafia da mesma algebra. O comentario registra driver e versao, o que torna o contorno auditavel.
- **F-07 -- `438fdb0` (teste que aceita duas topologias de ambiente).** Em vez de proibir `C:`, o teste passa a exigir "sem `C:`" **somente quando os binarios estao fora de `C:`** e a comparar com o fallback esperado. E a forma correta de testar uma politica dependente de ambiente sem transformar o ambiente em pre-condicao.
- **F-08 -- `0cbc742` (validar enum antes de entregar ao runtime).** Faixa legal explicita (`1..5`, `10..13`, `33..64`) e remapeamento com log limitado. Um valor invalido entregue a um driver nao produz erro: produz hang, e hang nao tem stack.

## G. Problemas ainda existentes

- **G-01 -- O VEH neutraliza `int3` do executavel principal por padrao.** `src/loader/hook_dll.cpp:6025-6031` define `continue_int3_mode = -1` ("Auto: continue single-byte inline `__debugbreak()` inside guest title EXE") quando `OPENSCARLETT_CONTINUE_INT3` nao esta setado. Em `:6192`, qualquer `0xCC` no modulo principal **nao seguido de outro `0xCC`** recebe `Rip += 1` e a execucao continua. Ou seja: todo assert do titulo vira no-op por default. Isso e provavelmente necessario para um build retail, mas significa que "bootou" nao distingue "o assert nao disparou" de "o assert disparou e foi ignorado". Nao ha contador publicado de quantos `int3` foram absorvidos por execucao.
- **G-02 -- Asserts especificos sao convertidos em resultados plausiveis por padrao de byte.** `hook_dll.cpp` reconhece sequencias exatas (`0xCC 0xEB 0xC5` com `C3` em `p-16` e `Rdx == R9`; `0xCC 0xE9 0xA5 0xFD 0xFF 0xFF` precedido de `31 C0 E9` e `Rdi == Rdx`) e reescreve `Rip` para o epilogo, fazendo a funcao "retornar uma contagem valida de chunks". O mecanismo e habilidoso e e o que permite o boot avancar; tambem e **especifico de uma build** — qualquer recompilacao do titulo muda os bytes e o tratamento deixa de casar, sem aviso.
- **G-03 -- Desenhos com PSO stand-in ou ausente sao silenciosamente descartados.** `command_list_x.cpp` registra `DrawInstanced SKIPPED #n` / `DrawIndexedInstanced SKIPPED #n` a cada 500 ocorrencias e segue. Um frame "renderizado a 30 FPS" pode conter apenas os desenhos cujo shader foi levantado; nao ha contador agregado no relatorio de status que diga qual fracao dos desenhos foi descartada. Combinado com G-01/G-02, e a razao pela qual "roda" e "renderiza" precisam ser lidos com cuidado.
- **G-04 -- O heap de aliasing de 256 MiB mapeia VAs distintas nos mesmos bytes.** `device_x.cpp:13108` calcula `heap_off = (VirtualAddress & (pool_bytes - 1)) & ~(req_align - 1)`. Dois recursos cujas VAs do Xbox difiram por um multiplo exato do pool caem no mesmo offset do host, sem deteccao de colisao e sem `MakeResident`/`Evict` por comando. A premissa ("GTA VI aliasa centenas de texturas transitorias dentro de ~256 MiB", comentario em `:13060`) e do titulo e nao e verificada pelo shim. Se dois recursos vivos colidirem, o resultado e corrupcao visual silenciosa, nao erro.
- **G-05 -- A heuristica de ABI do `XMemGetWorkingSetStatistics` continua.** Em HEAD ha dois caminhos de adivinhacao (`arg1` valido, ou `arg1` nulo com bit `0x20000000` em `arg0`). Um buffer de saida em pagina somente-leitura, ou uma flag que por acaso aponte para memoria gravavel, escolhe a ABI errada e faz `memcpy` de/para o lugar errado. Nao ha teste que exercite explicitamente as duas ABI no mesmo binario.
- **G-06 -- A classe de corrida do DirectStorage nao foi varrida nos outros shims.** `a208562` corrigiu tres `notify` no `dstorage_shim.cpp`. `xgameruntime_shim.cpp` (12.087 linhas), `xapu_shim.cpp`, `acphal_shim.cpp` e `xaudio2_shim.cpp` usam `condition_variable` e nao passaram por auditoria equivalente neste historico.
- **G-07 -- Documentacao e codigo discordam sobre a numeracao das topologias.** O README diz "console-only topology sanitization (17=RECTLIST->TRIANGLELIST)"; o comentario de `command_list_x.cpp` diz "19=RECTLIST, 18=QUADLIST, 17=LINELOOP" e `device_x.cpp:10617` diz "RECTLIST (0x13 = 19) / QUADLIST (0x11 = 17)". Ambos remapeiam o intervalo ilegal, entao nao ha defeito funcional; ha tres descricoes diferentes do mesmo fato, e a do README esta errada.
- **G-08 -- `tools/generate_status.py` publica literais.** O template carrega `775/779 real-GPU verified`, `4/4 CPU privileged trap/halt harness`, `3,120/3,120`, `15 PCs` e frases como "3/3 live retail Frame-1 draws executing `S_OK`" (`generate_status.py:408`) sem nenhuma leitura de artefato que os produza. Os unicos numeros realmente medidos sao `opcodes_decoded`/`opcodes_total` (de `docs/rdna2_coverage.json`) e o resultado do ultimo CTest.
- **G-09 -- O monolito `device_x.cpp` (18.879 linhas) concentra o risco.** Ele recebeu 44 commits, incluindo um diff de 3.981 linhas (`6fb3502`) e outro de 1.970 (`022a8b2`). `hook_dll.cpp` tem 7.738 linhas e recebeu um commit de +1.678 (`c922caa`). Nessa escala, revisao por diff deixa de existir na pratica, e e exatamente onde os defeitos deste catalogo se acumulam (D-10, D-12, D-15, D-20, G-04).
- **G-10 -- O fallback de diretorio de trabalho ainda aponta para o drive do autor.** `kDefaultWorkDirFallbackW = L"E:\\OpenScarlett\\.tmp\\openscarlett_run"` e retornado quando nao ha `TEMP`/`TMP` (`vfs_mounts.cpp:1130`) e em plataformas nao-Windows (`:1133`). Num host sem `E:` o caminho nao existe.
- **G-11 -- Nada aqui prova execucao de titulo comercial.** O README e honesto ("Commercial AAA titles beyond Minecraft are **not fully playable yet**"), mas os numeros de destaque (Stops 1-36, 360+ frames 4K, 3/3 draws) sao relatos do autor, sem artefato reproduzivel no repositorio. Os 50 blocos do pacote de evidencias nao contem nenhum log de execucao de titulo.
- **G-12 -- O cache de shader em disco sobrevive a atualizacao do binario.** Ver D-05. Nao ha `--clear-cache`, versao na chave, nem aviso; `%TEMP%\openscarlett_shader_cache` e o default (`shader_cache.h:129`).
- **G-13 -- O README declara "4/4 CI toolchains green" com dois legs nao bloqueantes.** `.github/workflows/ci.yml` em HEAD tem tres jobs: MSVC (`build-and-test`), `clang-cl-build-and-test` e `mingw64-build`. Os dois ultimos carregam `continue-on-error: true`, entao podem falhar sem reprovar o workflow. A frase do Sprint 15 ("`79/79` CTest, 4/4 CI toolchains green") nao pode ser sustentada por esse pipeline: verde em Clang-CL e MinGW nao e verificado, apenas registrado.

## H. Regressoes

- **H-01 -- `cbfd4f1` (merge) removeu o wrap do `ScarlettRetailDeviceProxy`.** O primeiro pai `74c3a5c` tem `bool retail_scarlett_vtable`; o resultado nao. `f2c0610` restaura no mesmo dia com a expressao "after UE5 merge" na mensagem. E a regressao mais cara do repositorio em potencial: sem o wrap, todo slot de vtable acima dos de desktop aponta para a funcao errada num titulo retail.
- **H-02 -- `f7566aa` corrigiu o descarte de listas invalidas e criou um travamento de CPU.** O `continue` sem `ExtractDeferredCpuOps` (D-14) fez esperas de CPU nunca satisfeitas. Corrigido em `022a8b2`, cerca de 39 h depois; e o caso mais claro de "fix que introduz defeito no proprio caminho que conserta".
- **H-03 -- `d7f1b7d` removeu o caminho de erro do `PresentX`.** Antes, falha de `CreateSwapChainForHwnd` retornava `hr`; depois, tres codigos passam a degradar para headless com `S_OK` (D-10). Nao ha teste que exija o erro de volta.
- **H-04 -- `7e2dd9b` ligou CTest paralelo e derrubou um teste de tempo.** Corrigido em `1ffd939` (D-18). A regressao e de infraestrutura, nao de produto, mas aparece como falha de teste e consome tempo de diagnostico.
- **H-05 -- `d71ed46` embutiu uma mudanca de GameInput nao relacionada.** O commit que para de evictar heaps tambem altera a varredura de slots do XInput (`preferred_slot` + fallback 0..3 + trace opt-in). Duas mudancas de risco independente num commit de 3 arquivos — o oposto de "small, atomic commits" que o proprio `AGENTS.md` §4 exige.
- **H-06 -- `4af2702` publicou `6/6` testes.** Regressao de documentacao com numero errado por ~4 horas, entre `4af2702` e `f350622` (D-06). O mecanismo que a produziu continua ativo.
- **H-07 -- `0cbc742` remapeia topologia e deixa tres descricoes conflitantes.** README (`17=RECTLIST`), comentario de `command_list_x.cpp` (`19=RECTLIST`) e `device_x.cpp` (`0x13 = 19`). Nao ha regressao de comportamento, mas qualquer leitor que confie no README procura o numero errado.
- **H-08 -- `05a24bc` empacotou cinco correcoes nao relacionadas.** QPC, polling XInput, foco/raw mouse, validacao de nome de container XGameSave e clamp de MSAA no mesmo commit (7 arquivos). Cada uma e correta; juntas, tornam bisseccao impossivel e escondem qual mudanca quebrou o que. Mesmo padrao em `6a6243e` (VFS + audio + dstorage + d3d12_xs), `f7566aa` (alinhamento + trace + skip de lista) e `d71ed46`.
- **H-09 -- `c57f29b` commitou um snapshot "wip" sem corpo.** 30+ arquivos, incluindo a remocao de seis `INTEGRATION_w*.txt`, com a mensagem apenas "snapshot uncommitted GDKX ABI fixes". Nove minutos depois `e555736` faz o merge e a mensagem real ("fix real `_GAMING_XBOX_SCARLETT` d3d12_xs ABI & boot untouched SimpleTriangle + SimpleLighting") descreve o que o commit anterior fez. Quem le o historico linear ve um commit sem descricao carregando a mudanca.

## I. Conhecimento reutilizavel

- **I-01 -- Struct de ABI de console: declare e prove com `static_assert`.** `sizeof` e `offsetof` de cada campo, no build, no header do shim. Sem isso, a divergencia so aparece quando um titulo real enfileira I/O, e o sintoma e memoria arbitraria. Arquivos: `include/openscarlett/io/dstorage_abi.h`; commit `76adeff`.
- **I-02 -- Cabecalho de compatibilidade de teste e parte do oraculo.** Se o duplo de teste declara uma vtable/struct menor que a real, a suite valida o shim contra a coisa errada e nao pode falhar. Sincronize o header de teste com o de produto e prefira gerar um do outro. Arquivos: `tests/external_gdk/gdk_compat_headers/xaudio2_xs.h`; commits `59bd45a`, `3cc1984`.
- **I-03 -- Enums de console nao sao enums de desktop.** Valide a faixa antes de entregar ao runtime e remapeie com log limitado; um valor ilegal nao retorna erro, trava o pushbuffer. Arquivos: `command_list_x.cpp`, `device_x.cpp`; commit `0cbc742`.
- **I-04 -- Alinhamento de footprint difere entre console e desktop.** Xbox aceita offset de 64/256 e pitch < 256; desktop exige 512/256. Planeje um caminho de staging por `CopyBufferRegion`, que aceita granularidade de 1 byte. Arquivo: `command_list_x.cpp`; commit `f7566aa`.
- **I-05 -- `condition_variable` de objeto cuja vida termina no despertar: notifique dentro do lock.** "Notificar fora do lock" e otimizacao para objetos que sobrevivem ao waiter; aqui e use-after-free. Arquivo: `src/shims/dstorage_shim.cpp`; commit `a208562`.
- **I-06 -- Nunca chame codigo de terceiro com o proprio mutex tomado.** Atualize o estado, solte, notifique. Arquivo: `src/core/telemetry.cpp`; commits `acbdb7c`, `2b42f72`.
- **I-07 -- Budget sintetico nao autoriza evictar estado de terceiro.** Se o limite foi inventado pelo shim, a resposta a estourar e recusar alocacao ou paginar, nunca `Evict` em recurso vivo. Arquivo: `device_x.cpp`; commit `d71ed46`.
- **I-08 -- Cache de traducao precisa da versao do tradutor na chave.** Um cache de entrada serve bytecode de outra versao do lowering depois de qualquer correcao; a versao precisa ser bumpada no mesmo commit que muda a semantica, e um teste deve forcar a igualdade das duas constantes. Arquivos: `shader_cache.h`, `shader_cache.cpp`; commit `538e904`. Estado: aberto (D-05).
- **I-09 -- Regra de traducao de caminho deve ser uma funcao, nao N blocos.** "Este caminho e virtual?" foi aplicada em `CreateFile`, depois em `GetFileAttributes`/`CreateDirectory`, depois em CRT, depois em `GetFinalPathNameByHandleW`. Cada API nova e um bug. Arquivo: `src/loader/hook_dll.cpp`; commits `990c3ff`, `e7c5f85`, `be0fda1`, `4bc2db4`.
- **I-10 -- Politica de ambiente nao entra no produto nem na assercao.** "Nunca escrever em `C:`" e higiene de uma maquina; virou `ResolveDefaultWorkDirW` e assercao de teste, e quebrou em qualquer host com build em `C:`. Separe "onde eu prefiro escrever" de "o que e correto". Arquivos: `vfs_mounts.cpp`, `test_loader_vfs_config.cpp`; commits `7e2dd9b`, `438fdb0`.
- **I-11 -- Metrica publicada precisa ser medida no mesmo ato em que e publicada.** Ler `LastTest.log` publica o resultado da ultima execucao, nao o da suite; rodar um subconjunto imprime `6/6` no README. Leia a lista de testes registrados no build (ou `ctest -N`) e valide contra o esperado. Arquivo: `tools/generate_status.py`; commit `4af2702`.
- **I-12 -- Skip precisa de codigo dedicado.** Early `return` num teste de hardware conta como `Passed`; sem `SKIP_RETURN_CODE` (ou equivalente), "90/90" nao significa "90 testes exercitaram o produto". Arquivos: `tests/*`, `cmake/build_speedups.cmake`; estado: aberto (D-09).
- **I-13 -- Tempo de parede e oraculo toxico em suite paralela.** Qualquer teste com limite de tempo precisa de `RUN_SERIAL` e isolamento de GPU. Arquivo: `cmake/build_speedups.cmake`; commits `7e2dd9b`, `1ffd939`.
- **I-14 -- Contorno de bug de compilador: mude a forma, nao a grafia.** DXC dobra toda algebra equivalente de `-x`; o caminho e um laco CAS, documentado com driver e versao. Arquivo: `rdna2_isa_mem_img.cpp`; commit `5d3809b`.
- **I-15 -- Injetar `int3` continuado por padrao e uma decisao de produto, nao um detalhe.** Se asserts do convidado sao absorvidos, registre a contagem por execucao e exponha no relatorio; sem isso, "bootou" perde significado diagnostico. Arquivo: `src/loader/hook_dll.cpp`; estado: aberto (G-01).

## J. Conclusao tecnica

O OpenScarlett e um projeto de engenharia de sistemas serio disfarcado de projeto caotico. Em seis dias e 351 commits ele construiu, do zero, um loader com injecao APC, um VFS copy-on-write aninhado, 33 DLLs de shim com 1.372 exports, uma camada D3D12 completa com proxys de vtable retail, um backend Vulkan 1.4 independente de SDK, um tradutor RDNA2 com 779 opcodes e um corpus de 3.120 shaders, um launcher nativo e uma suite de 90 testes. Nada disso e comum, e boa parte das correcoes deste catalogo mostra alguem que entende o que esta fazendo: o laco CAS para `ds_sub`, o staging de footprint, o `notify` dentro do lock, o `static_assert` de ABI.

O que separa o projeto de um port confiavel nao e capacidade tecnica, sao tres habitos.

**Primeiro: a ABI e reconstruida de memoria, e o teste herda o erro.** Quatro defeitos independentes tem a mesma forma — DirectStorage, GameInput, `XMemGetWorkingSetStatistics` e a vtable de XAudio2. Em tres deles o valor foi digitado a partir do que se lembrava do cabecalho de console ou do de desktop, e em dois o **cabecalho de teste foi escrito com o mesmo erro**, de modo que a suite passou a certificar o defeito. O que consertou de fato foi o `static_assert` (D-01) e a sincronizacao do duplo de teste (D-04); o que ficou pela metade foi a heuristica de `VirtualQuery` (D-03/G-05).

**Segundo: o ambiente do autor vaza para o artefato.** `E:\OpenScarlett` esta compilado no produto; "nunca `C:`" esta no resolvedor de diretorio e nas assercoes; o gerador de status le o log da ultima execucao local; o CI tem dois legs `continue-on-error` enquanto o README fala em "4/4 CI toolchains green"; e a correcao do `PresentX` que existe para o Session-0 do runner remove o caminho de erro do produto. Esse vazamento e o que produz a classe mais corrosiva do repositorio — nao um bug, mas um **numero publicado que nao corresponde ao codigo** (D-06, D-07, D-08, D-09).

**Terceiro: a compatibilidade e obtida alterando o convidado, e o custo disso nao aparece nos numeros.** O VEH continua `int3` de um byte por padrao, converte cold asserts especificos em "contagem valida de chunks" por casamento de bytes, desenhos sem PSO levantado sao descartados em silencio, e texturas transitorias de VAs distintas sao sobrepostas num heap unico por aritmetica de modulo. Cada uma dessas decisoes e defensavel isoladamente — e provavelmente a unica forma de fazer um titulo retail avancar num prazo de dias — mas juntas elas mudam o significado de "boota": significa que o fluxo chegou ao proximo estagio, nao que os estagios anteriores funcionaram. O README e honesto sobre o resultado ("not fully playable"), e menos honesto sobre o metodo: em nenhum lugar ele diz que asserts sao neutralizados e desenhos sao pulados.

Para o objetivo deste acervo — saber o que este repositorio ensina — a resposta e: ele ensina, com muita competencia, **como shimar uma ABI de console** (proxys por slot, sanitizacao de enums, traducao de alinhamento, cache de PSO) e ensina, por contraexemplo, **como nao medir o proprio progresso**. A parte reaproveitavel esta nas secoes F e I; a parte a nao copiar esta em D-06 a D-09 e em G-01 a G-04.

## K. Indice de commits

Todos os 351 commits alcancaveis de `HEAD`, em ordem cronologica (commit 0 primeiro), no formato `hash|data|assunto`. Autoria nao repetida aqui (ver secao A); a atribuicao de cada commit aparece em `git log`. Os 4 commits que existem apenas em `origin/feat/ue5-matrix-awakens-imports` nao entram neste bloco - ver L.

```text
ec4d6d3|2026-11-21|chore: initialize OpenScarlett MIT C++20 workspace and engineering guide
775af44|2026-11-21|feat(core): add PE32+ import/delay-load inspector, live call telemetry, and CLI GPU probe
d971f0d|2026-11-21|feat(codecs): add clean-room GDeflate v1, BCPack, XMA2 IMDCT, and Bink2 codecs
2fa61ca|2026-11-21|feat(gpu): add dynamic Vulkan 1.2/1.4 compute backend and RDNA2/GFX10.3 decoder + SSA IR
9e710fd|2026-11-21|feat(winrt): add clean-room WinRT activation and HSTRING runtime (winrt_xs.dll)
54e4fe1|2026-11-21|docs(status): add human-readable README and auto-generated live status dashboard
a2763f4|2026-11-21|fix(gpu): replace leftover ScarlettX log strings with OpenScarlett and ignore .worktrees/.tools
acbdb7c|2026-11-21|chore(build): add modular CMake target hooks and cross-DLL telemetry forwarding
7b4aeaa|2026-11-21|ci(windows): add GitHub Actions MSVC/Ninja CI and portable win64 test bundle
5e76043|2026-11-21|fix(test): copy VulkanDeviceInfo before Shutdown so GPU name logs accurately
c6857ea|2026-11-21|feat(loader): add suspended-process APC hook loader, G:/T:/S: VFS, and openscarlett run
40a2ddf|2026-11-21|feat(loader): merge suspended-process APC hook loader, VFS mounts, and openscarlett run
8eddb1e|2026-11-21|feat(kernelx): add VirtualAlloc2/MapViewOfFile3 memory aliasing, GpuTilePool, and kernelx.dll
922ef27|2026-11-21|docs(status): update dashboard after APC hook loader & VFS merge
efe0954|2026-11-21|feat(kernelx): merge VirtualAlloc2/MapViewOfFile3 memory aliasing, GpuTilePool, and kernelx.dll
1317b55|2026-11-21|docs(status): update dashboard after kernelx & GpuTilePool merge
7cb9125|2026-11-21|chore(build): add optional dxr_targets.cmake hook for Sprint 2 worktrees
a429535|2026-11-21|feat(gdk): add clean-room xgameruntime, GameInput, XAudio2/AcpHal, dstorage_xs, and XSAPI shims
2c8c5cd|2026-11-21|feat(gdk): merge clean-room xgameruntime, GameInput, XAudio2/AcpHal, dstorage_xs, and XSAPI shims
b68fdd9|2026-11-21|docs(status): update dashboard after core GDK shims merge
d58f0b6|2026-11-21|feat(graphics): add modular d3d12_xs COM proxy, xg_xs swizzle, high-perf adapter selector, and GPU FSR 1.0
e4bc196|2026-11-21|feat(graphics): merge modular d3d12_xs COM proxy, xg_xs swizzle, high-perf adapter selector, and GPU FSR 1.0
7487b05|2026-11-21|docs(status): update dashboard after Sprint 1 completion (d3d12_xs, xg_xs, FSR 1.0)
dd2efb5|2026-11-21|feat(dxr): add Hardware DXR Tier 1.1 pipeline with SAH software BVH fallback
90dce25|2026-11-21|feat(dxr): merge Hardware DXR Tier 1.1 pipeline with SAH software BVH fallback
8ca13f6|2026-11-21|docs(status): update dashboard after DXR Tier 1.1 merge
3e9b42a|2026-11-21|fix(graphics): deduplicate FrameBufferRgba8 and sanitize Xbox D3D12 flags + 0-plane PresentX
fbf9ae5|2026-11-21|chore(build): add optional gdk_live_targets.cmake hook for Sprint 3 real-binary bring-up
9ecc1d2|2026-11-21|feat(shaders): add AAA RDNA2 lifter with MIMG, DS/LDS, wave64-on-wave32, and SM6.5 Mesh/AS shaders
71436e8|2026-11-21|feat(shaders): merge AAA RDNA2 lifter with MIMG, DS/LDS, wave64-on-wave32, and SM6.5 Mesh/AS
7b3c455|2026-11-21|docs(status): update dashboard after Sprint 2 AAA RDNA2 shader lifter merge
dacad04|2026-11-21|feat(bringup): run real MIT Microsoft Xbox-GDK-Samples live via openscarlett run
f69c998|2026-11-21|feat(bringup): merge Sprint 3 real MIT Microsoft Xbox-GDK-Samples live bring-up
d8518c2|2026-11-21|fix(ci): ensure portable temp/dxc paths and WARP-only CI fallback while preserving strict HW checks
cde5b2a|2026-11-21|docs(status): publish live Sprint 1-3 hardware compatibility matrix and 13/13 status dashboard
529e0a6|2026-11-21|fix(ci): detect Hyper-V Basic Render Driver (0x1414/0x008C) as WARP and fix test_apc_loader temp path
a08e3f7|2026-11-21|feat(testing): add 1-click hardware test runner, bundled dxc lookup, GitHub Issue submission, and v* release automation
7e1897a|2026-11-21|chore(build): add modular Wave 1/2 cmake hooks and DeviceX frame-interval getters
ed3f818|2026-11-21|docs(readme): feature RTX 2060 SUPER / Win11 DXR Tier 12 community report, contributor thanks, and hero tester callout
7d97bb9|2026-11-21|fix(tests): resolve runtime exe dir & TEMP scratch portably (fixes #1) and hide build/ in CI smoke test
26a9949|2026-11-21|feat(presentx,input): add interactive Win32 PresentX swapchain windowing, pacing, and GameInput frame loop
c421483|2026-11-21|feat(loader): add MicrosoftGame.Config parser, CoW overlay VFS, PE forwarder/TLS/.pdata hardening, and data-driven DLL routing
a4f24d1|2026-11-21|feat(shims): implement xgameruntime QueryApiImpl dispatch and 9 secondary GDK/console shim DLLs
80bb4f2|2026-11-21|feat(audio,xg,dstorage): add Opus MFT + IMA-ADPCM + live WASAPI, full GFX10 2D/3D/mip-tail xg_xs, and dstorage queue hardening
538e904|2026-11-21|feat(shaders): add AAA RDNA2 opcodes, Xbox DXBC parser, CPU wave32/64 interpreter, disk shader cache, and d3d12_xs auto-lifting
78614ba|2026-11-21|feat(wave1): merge wave1/rdna2-aaa-cache
1a0e592|2026-11-21|feat(wave1): merge wave1/audio-textures-dstorage
f3ab0f9|2026-11-21|feat(wave1): merge wave1/loader-vfs-config
a212196|2026-11-21|feat(wave1): merge wave1/secondary-shims-queryapi
8aa18bf|2026-11-21|feat(wave1): merge wave1/presentx-windowing
67cf01b|2026-11-21|chore(wave1): fix C4324 warning, portable test_loader_vfs_config paths, and update status dashboard (19/19 CTest)
326a3d7|2026-11-21|chore(build): add modular per-agent cmake hooks for Wave 2 GDK bring-up
55db73e|2026-11-21|feat(bringup): add SimplePBR multi-frame 3D MSAA/depth GDK live sample
f55763e|2026-11-21|feat(bringup): add DXRTriangle hardware-DXR/SAH-BVH GDK live sample
d41617c|2026-11-21|feat(wave2): merge wave2/gdk-simple-pbr
851a20c|2026-11-21|feat(wave2): merge wave2/gdk-dxr-triangle
1292331|2026-11-21|feat(bringup): add MeshletCull AS+MS+PS GPU culling GDK live sample
a6a1a05|2026-11-21|feat(wave2): merge wave2/gdk-meshlet-cull
cb2e7c7|2026-11-21|fix(bringup): fix MeshletCull Win10 WARP DXIL JIT crash and tighten verification
a80932e|2026-11-21|feat(wave2): merge final wave2/gdk-meshlet-cull (Win10 WARP fix + HW/WARP parity)
59bd45a|2026-11-22|feat(bringup): add GDK-header external binaries and 19-DLL live telemetry bring-up
69f0d1b|2026-11-22|feat(wave2): merge wave2/external-gdk-bringup
df979fe|2026-11-22|fix(d3d12_xs): add MinGW-w64 COM aggregate-return and UUID compatibility (#6)
9aa3955|2026-11-22|fix(tests,docs): polish HW report UX/logs and update RTX 2060 SUPER 13/13 pass (#8)
d7f1b7d|2026-11-22|fix(ci): harden shader cache SM key, Opus MFT probe, and headless PresentX for Windows Server CI
9ad1971|2026-11-21|tests: add NPOT mip-chain and 32x32x32 3D LUT vectors to test_xg_dstorage (#11)
2c0c7d1|2026-11-22|docs(readme): feature AMD RX 7900 XTX & RX 9070 XT 13/13 passes and PR #11 (#9, #10)
bcc2895|2026-11-22|docs(readme): add AMD RX 7600 & NVIDIA RTX 3080 Ti 13/13 passes (#12, #13)
d522293|2026-11-22|chore(cmake): add modular optional includes for v0.3.0-alpha parallel waves
39c8520|2026-11-22|ci(portability): add Clang-CL & MinGW-w64 CI legs and compiler portability profile (#6)
9fd9098|2026-11-22|feat(gdk,audio,loader): add XDisplay HDR, extended GDK UI/Invite/Speech, X3DAudio/XAudio2FX, and topological DLL init ordering
ee10610|2026-11-22|feat(d3d12_xs): HDR10/scRGB PresentX swapchains, default array SRVs, unified-memory tracking & fail-closed stubs
e8915fa|2026-11-22|feat(vulkan): add zero-SDK Vulkan 1.2/1.4 rasterization, VK_EXT_mesh_shader, 3D textures & Win32 swapchain
ff98830|2026-11-22|feat(loader): eliminate C:\ default VFS/telemetry leak, add versioned GDK DLL routing aliases, and add Store-bootstrap diagnostics
7e413ef|2026-11-22|feat(rdna2): implement 5 GFX10.3 ISA, LDS zero-init, quad deriv, wave64 mesh, & WARP catches
3316f2b|2026-11-22|feat(wave1): merge wave1/vulkan-full-graphics
af78d02|2026-11-22|feat(wave1): merge wave1/rdna2-isa-catches
bab0de4|2026-11-22|feat(wave1): merge wave1/hdr10-d3d12-catches
480945a|2026-11-22|feat(wave1): merge wave1/gdk-display-audio-loader
92aa4e4|2026-11-22|feat(wave1): merge wave1/mingw-ci-portability
72582ff|2026-11-22|docs(status): refresh live status dashboard after Wave 1 (26/26 CTest passing)
7cc28a7|2026-11-22|fix(build,ci): prevent MSVC 17.10+ std::mutex crash on older msvcp140.dll
9fa5019|2026-11-22|feat(shims): achieve XCurl 16/16 real-import parity and expand XSAPI to 178 exports
0b3d66b|2026-11-22|feat(gdk): expand xgameruntime.dll thunks parity and 12-GUID QueryApiImpl dispatch
96d41b3|2026-11-22|Merge branch 'feat/xcurl-xsapi-parity'
364c70d|2026-11-22|Merge branch 'feat/xgameruntime-thunks-parity'
fd31b27|2026-11-22|Merge branch 'feat/loader-vfs-routing'
b5930b0|2026-11-22|feat(gdk): add HDRCubeMap real-binary GDK workload (FP16 cubemap, default Texture2DArray SRV, HDR10 ST.2084)
d814027|2026-11-22|feat(showcase): add VulkanMeshShowcase RDNA2 XS2 AS+MS+PS + 3D LUT cross-API parity suite
9028d7b|2026-11-22|feat(gdk): add SimpleLighting 3D Blinn-Phong workload with XDisplay, X3DAudio/FX & Vulkan 1.4 parity
9aacf54|2026-11-22|feat(wave2): merge wave2/gdk-simple-lighting
a7bb35b|2026-11-22|feat(wave2): merge wave2/gdk-hdr-cubemap
c05c58e|2026-11-22|feat(wave2): merge wave2/vulkan-mesh-showcase
9595eea|2026-11-22|docs,release: upgrade portable tester, status dashboard & README for v0.3.0-alpha (closes #3, #5, #6)
55b9473|2026-11-22|fix(gdk,tests): handle 3-arg XGamingRuntimeThunks QueryApiImpl(clsid, iid, ppv) safely and clean CI VFS overlay
aa3a175|2026-11-22|fix(readme,status): stop re.sub from injecting NUL byte for XS2\\0 in README.md and update GDK shim metrics
841d872|2026-11-22|fix(ci,tests): make test_gdk_extended_loader portable for hidden-build smoke test and make Clang-CL leg non-blocking
0eb97f9|2026-11-22|fix(ci,docs): fix Clang-CL STL check & MinGW d3d12_xs export macro, switch hardware test refs to .ps1
4d23c6b|2026-11-22|fix(ci): preserve /EHsc in Clang-CL CMAKE_CXX_FLAGS so C++ exceptions stay enabled
e264e94|2026-11-22|ci(release): make publish-release idempotent with clobber/edit for tag republishes
84718cd|2026-11-22|feat(playable-game): add Scarlett full-stack Freedoom/Arena host & real GDK thunks verification
0a61312|2026-11-22|feat(codecs,gpu): harden BCPack BC1-BC7 entropy layer and add RDNA2 BVH4 decoder
ed0ad71|2026-11-22|feat(shims): add DirectStorage cancel/bounds/phys-page, mfplat/diag DLLs, XMem/XThread & WASAPI parity
095964e|2026-11-22|feat(loader): add unified DLL routing updates, hardened CoW VFS, and CreateProcess child hook propagation
19e7889|2026-11-22|feat(xgr): 100% XGamingRuntimeThunks parity (292/292) & 3-arg QueryApiImpl COM ABI
1efa12b|2026-11-22|feat(gpu): add D3D12X extensions, Vulkan sparse/mip parity, and xg_xs metadata layouts
fc2699d|2026-11-22|feat(gpu): add RDNA2/GFX10.3 corpus parity, PSV0 wave bounds, mesh contracts, and DXIL re-signing
55c660f|2026-11-22|merge: integrate feat/rdna2-corpus-parity
269170a|2026-11-22|merge: integrate feat/gpu-d3d12x-vk-tiling
5dbc8ff|2026-11-22|merge: integrate feat/xgr-100pct-queryapi
51f8386|2026-11-22|merge: integrate feat/dstorage-mfplat-audio-shims
9a3f027|2026-11-22|merge: integrate feat/loader-vfs-child-hook
ea82eb8|2026-11-22|merge: integrate feat/bcpack-bvh4-codecs
9685348|2026-11-22|merge: integrate feat/playable-scarlett-game
d373def|2026-11-22|feat(sprint6): integrate 7-worker parity wave, 292/292 GDK thunks, child-hook propagation & playable Scarlett game (37/37 CTest)
6a15cbf|2026-11-22|fix(ci,vfs,gpu): honor explicit VFS env overrides unless forbid_sys_writes set, fix WARP GetResourceTiling mip split & v0.3.1-alpha release notes
afc6e8e|2026-11-22|feat(gdk-3d-scene): add 3D shadowed skinned BC1/BC3 GDK title and dual-API test
2abe1ec|2026-11-22|feat(gpu): direct d3d12_xs -> Vulkan 1.4 routing, cross-vendor Vulkan edge cases & separated PresentX timings
de60489|2026-11-22|feat(gpu): single-wave lockstep LDS barriers, explicit no-ops, and differential ISA verification
567ef39|2026-11-22|feat(s7): PE/loader hardening, XMemVirtualQuery, Minecraft-GDK surface, 6 stub DLLs, raw mouse & XGEncodeDCC
4447291|2026-11-22|merge: Wave 1 Worker 1 (d3d12_xs->Vulkan routing & cross-vendor edge cases)
666ffcc|2026-11-22|merge: Wave 1 Worker 2 (RDNA2 lockstep LDS barriers, no-op opcodes & diff ISA)
f5bfe24|2026-11-22|merge: Wave 1 Worker 3 (PE/loader hardening, XMemVirtualQuery, GDK parity & 6 stubs)
360d909|2026-11-22|merge: Wave 1 Worker 4 (3D shadowed skinned BC1/BC3 GDK scene & dual-API test)
d073446|2026-11-22|feat(sprint7): integrate Wave 1 (d3d12_xs->Vulkan routing, RDNA2 lockstep LDS & diff ISA, PE/GDK runtime & 3D GDK scene; 41/41 CTest)
6aeeb86|2026-11-22|feat(launcher): define Wave 2 contract headers for updater and hardware-test runner bridge
afd0d3e|2026-11-22|docs(readme,branding): human-readable README, subtle Series X|S banner & icon, clean status.svg and interactive dashboard
90dd58d|2026-11-22|feat(launcher): implement clean-room WinHTTP/BCrypt updater core
0881167|2026-11-22|docs: fix hardware test references from .bat to run_hardware_test.ps1
27b5097|2026-11-22|feat(launcher): implement hardware test runner, telemetry bridge, and PE export inspector
915e602|2026-11-22|feat(launcher): add native Win32/Direct2D dark GUI shell and updater test suite
fb45982|2026-11-22|merge: Wave 2A (WinHTTP + BCrypt SHA-256 opt-in updater)
686dd75|2026-11-22|merge: Wave 2B (Hardware test runner, GPU telemetry & PE inspector bridge)
54c2457|2026-11-22|merge: Wave 2C (Native Win32/Direct2D dark GUI shell & launcher/updater test suite)
58f3fd8|2026-11-22|feat(sprint7): integrate Wave 2 native Win32/D2D OpenScarlettLauncher GUI, 60s hardware tester & BCrypt SHA-256 updater (42/42 CTest)
ba1c58f|2026-11-22|fix(ci,tests): skip HW-only Vulkan SPIR-V execution when no ICD is present and honor OPENSCARLETT_VK_FORCE_SOFTWARE
84d3689|2026-11-22|docs(status): publish verified codebase metrics (30 DLLs / 926 exports, 224/779 RDNA2 opcodes, 42/42 CTest) across README, SVG & dashboard
c9172fc|2026-11-22|fix(ci,tests): guard remaining HW Vulkan SPIR-V paths in test_rdna2_lockstep_isa when no ICD is present
0e851da|2026-11-22|fix(updater): use built-in tar.exe with .NET ZipFile fallback to avoid pwsh7 PSModulePath Expand-Archive failure on CI
34b8196|2026-11-22|fix(launcher,updater): harden zip extraction for CI/spaces and embed docs/icon.ico
a8b4193|2026-11-22|feat(gpu): add GFX10.3 779-opcode coverage auditor and 3,120-shader corpus benchmark
91364e4|2026-11-22|fix(loader,tests): fix C:\ portable bundle test guards and skip APC-injecting dxc.exe (#14)
43e8c32|2026-11-22|fix(launcher): embed failing test diagnostics in pre-filled GitHub hardware report URLs (#14)
7ba25f7|2026-11-22|docs(readme,status): add Discord invite, real-GDK tester callout, honest progress bars and 30-DLL / RDNA2 / D3D12-Vulkan coverage visualizer
0392f81|2026-11-22|feat(gpu): complete GFX10.3 MUBUF/MTBUF/MIMG/DS/FLAT/EXP ISA (449 mnemonics, CPU ref, SPIR-V 1.3 & SM 6.5 live GPU parity)
1d46efa|2026-11-22|feat(rdna2): expand GFX10.3 scalar/vector ISA to 744 mnemonics with SDWA/DPP16/DPP8, CPU ref, and SPIR-V/DXIL live GPU parity
db26565|2026-11-22|fix(build): find MSVC, CMake, Ninja and dxc instead of fixed paths
3208ca1|2026-11-22|merge(sprint8): Worker 1 GFX10.3 scalar & vector ISA (744 mnemonics, SDWA/DPP16/DPP8)
05b1df9|2026-11-22|merge(sprint8): Worker 2 GFX10.3 MUBUF/MTBUF/MIMG/DS/FLAT/EXP ISA (449 mnemonics)
554073a|2026-11-22|merge(sprint8): Worker 3 GFX10.3 779-opcode coverage auditor & 3,120-shader corpus bench
c518759|2026-11-22|merge(sprint8): Worker 4 launcher icon resource & 3-tier updater zip extraction
c2c93aa|2026-11-22|fix(tools): mask the repo folder in hardware reports from source builds
78543c9|2026-11-22|feat(sprint8): wire GFX10.3 ISA modules, rdna2-bench CLI & CTest targets (45/45 passing, 0 warnings)
efc35f5|2026-11-22|docs(sprint8): publish verified GFX10.3 coverage report (774/779 opcodes, 3,120/3,120 shaders, 45/45 CTest)
bee85df|2026-11-22|fix(ci,tests): use portable temp_directory_path in test_rdna2_corpus_bench and skip WARP in D3D12 HW init
8319812|2026-11-22|Merge pull request #16 from marquezinii/fix/hw-report-repo-path
8633ad7|2026-11-22|Merge pull request #15 from marquezinii/fix/build-cmd-toolchain-discovery
4e350ba|2026-11-22|fix(build,launcher): harden x64 prompt check in build.cmd and mirror <REPO_DIR> sanitizer
6d38371|2026-11-22|docs(readme,status): add RTX 5070 Blackwell report (#17) by @marquezinii (7 PCs verified)
e01f21b|2026-11-22|feat(gpu,kernel): add TitleVAMirror unified memory mirror and RDNA2 DescriptorSgprBridge
b9c26e8|2026-11-22|feat(probe,capture,launcher): add First-Boot Probe VEH dumper, per-draw PNG frame capture & Launcher controls
126a234|2026-11-22|feat(os-winrt-auth): Xbox 14-thread affinity remapping, WinRT lifecycle & offline XSTS/Store/XSAPI/XCurl boot
634c5a1|2026-11-22|feat(gpu): add GPU compute GFX10 untiler, HS/DS/GS lifting, DXR SBT mapper, Wave64-on-Wave32 & async PSO pool
dd71dd3|2026-11-22|feat(media,audio,input): implement MFMediaEngine video decode, 64-voice XAudio2 effects, Bink2 streaming & GameInput hot-plug
9a964ff|2026-11-22|feat(d3d12_xs): complete 10 COM object proxies, Xbox extensions, and multi-queue thread safety
18da085|2026-11-22|merge(s910): Worker 2 64-bit TitleVAMirror & RDNA2 V#/T#/S# bindless SGPR bridge
e3cb233|2026-11-22|merge(s910): Worker 3 GPU compute untiler/retiler, HS/DS/GS, DXR SBT, Wave64-on-32 & async PSO pool
dca4cc7|2026-11-22|merge(s910): Worker 4 7-core affinity remap, WinRT lifecycle & non-blocking offline auth
d3f2277|2026-11-22|merge(s910): Worker 5 MFMediaEngine video decode, 64-voice XAudio2, Bink2 streaming & GameInput
c39a175|2026-11-22|merge(s910): Worker 6 First-Boot Probe VEH crash dumper, frame capture & launcher UI
df175a2|2026-11-22|merge(s910): Worker 1 d3d12_xs 10 COM proxies, Xbox extensions & multi-queue thread safety
04a8eba|2026-11-22|feat(s910): wire Sprint 9+10 compatibility engine targets, hooks, VEH probe & LDS/dxc fixes (51/51 CTest)
e4463d0|2026-11-22|docs(s910): update status badges, dashboard, CI release notes & README for v0.5.0-alpha (51/51 CTest)
7b90c9c|2026-11-22|fix(ci): resolve Clang-CL narrowing, MinGW __mingw_uuidof IIDs & host-DXR GetShaderStackSize on CI WARP
a23044c|2026-11-22|fix(rdna2): keep PROC_THREAD_ATTRIBUTE_HANDLE_LIST array alive across CreateProcessA (Clang-CL err=87)
0c61420|2026-11-22|feat(s11): add 12 KiB XVD geometry inspector, mixed-case PE forwarder resolver, Console LSA/PFN policy & 2 untouched MIT GDKX samples
c57f29b|2026-11-22|wip(gdkx): snapshot uncommitted GDKX ABI fixes and untouched SimpleTriangle/SimpleLighting bring-up
e555736|2026-11-22|feat(gdkx): fix real _GAMING_XBOX_SCARLETT d3d12_xs ABI & boot untouched SimpleTriangle + SimpleLighting (53/53 CTest)
974836d|2026-11-22|feat(d3d12_xs): add console Sampler Feedback readback/clear, QueryAccumulatorX, predicated byte copies & GPU-QPC clock calibration
d3371a5|2026-11-22|feat(gpu): emit SPIR-V 1.6 TaskEXT for Amplification shaders, Wave64 LDS helpers & VS/GS/MS CullDistance
d8a5a27|2026-11-22|fix(ci): resolve Clang-CL constexpr enum range and MinGW-w64 ID3D12Device10 pCastableFormats constness
c492ca7|2026-11-22|merge(s11): Worker 2 AS TaskEXT SPIR-V 1.6 (130/130 spirv-val), Wave64 LDS & barycentric CullDistance
b8bbf32|2026-11-22|merge(s11): Worker 3 console Sampler Feedback readback/clear, QueryAccumulatorX, predicated byte copies & GPU-QPC clock calibration
95fde10|2026-11-22|merge(s11): Worker 4 12 KiB XVD inspector, mixed-case PE forwarders, Console LSA/PFN policy & 2 extra MIT GDKX samples
d5b1f50|2026-11-22|feat(rdna2): expand real-GPU verification to 615/779 opcodes and fix 4 GFX10.3 HW accuracy rules
902c38e|2026-11-22|merge(s11): Worker 1 615/779 real GPU-verified RDNA2 opcodes, 414 spirv-val, 432 dxc & 4 GFX10.3 HW accuracy rules
44b1aa8|2026-11-22|feat(sprint11): enable AS TaskEXT SPIR-V in corpus (3120/3120), publish 615/779 GPU-verified RDNA2 coverage & 57/57 CTest
4a7e79b|2026-11-22|fix(ci): resolve Clang-CL constexpr enum range in gdkx_headers/d3d12_xs.h and portable temp path in test_xvd_loader_lsa
f599765|2026-11-22|feat(gdkx): bring up 4 untouched Microsoft Xbox-GDK-Samples (add SimpleTexture + SimpleInstancing)
d348487|2026-11-22|feat(gpu,dxr): Sprint 12 RDNA2 RayQuery bridge, 64-bit multi-window BVH4, XboxBvhBuilder conversion & SBT remapping
dac08f1|2026-11-22|feat(s12): harden DirectStorage/BCPack/metadata coherency and add VRAM residency + page aliasing
75d23ee|2026-11-22|feat(s12-os-loader): wire PeForwarder/XvdInspector/ConsoleLsaPolicy live, TLS slot 0, NT VFS, GameInput, xgameruntime & lock-free 128-voice XAudio2
c5d4a7a|2026-11-22|fix(ci): pass /arch:AVX2 under Clang-CL for _GAMING_XBOX_SCARLETT DirectXMath intrinsics
6989d70|2026-11-22|feat(d3d12_xs): wire Sprint 12 AAA GPU islands, GXDK extensions & PresentX pacing
d3866a2|2026-11-22|feat(gpu): add Xbox dxc_xs shader container parser, CFG hardening, and in-process DXC shader cache
80081d5|2026-11-22|merge(s12): Worker 6 4 untouched Microsoft Xbox-GDK-Samples (SimpleTriangle, SimpleLighting, SimpleTexture, SimpleInstancing)
eadac72|2026-11-22|merge(s12): Worker 1 live D3D12X island wiring, GXDK extensions & PresentX pacing
37e54cf|2026-11-22|merge(s12): Worker 2 Xbox DXBC/PM4/SGPR shader container, CFG hardening & in-proc DXC cache
6c17dc8|2026-11-22|merge(s12): Worker 3 live RDNA2 RayQuery DXR/SPIR-V bridge, 64-bit multi-window BVH4 & SBT remapping
2b4e485|2026-11-22|merge(s12): Worker 4 hardened DirectStorage/BCPack/metadata coherency, page aliasing & VRAM residency
1be8083|2026-11-22|merge(s12): Worker 5 live PeForwarder/LSA/probe, TLS slot 0, NtCreateFile VFS, GameInput & 128-voice XAudio2
664af08|2026-11-22|feat(sprint12): finalize AAA wall integration, portable test paths, hardware runner & 62/62 status docs
debcef5|2026-11-22|docs(wiki): add comprehensive 15-page OpenScarlett wiki and GitHub Wiki sync script
b015934|2026-11-22|docs(wiki): link README badge to live GitHub Wiki and rewrite .md links on publish
990c3ff|2026-11-22|fix(vfs): preserve real host D:\ paths across VFS/Win32/NT hooks on CI runners & host drives
fc77262|2026-11-22|feat(loader): add opt-in OPENSCARLETT_TRACE_ODS OutputDebugStringA hook for guest diagnostics
cd7f163|2026-11-22|chore(ci): trace GDKX startup checkpoints under OPENSCARLETT_TRACE_ODS for Clang-CL diagnosis
5e72a2c|2026-11-22|chore(ci): trace USER32 RegisterClassExA/CreateWindowExA under OPENSCARLETT_TRACE_ODS
fe86185|2026-11-22|fix(gdkx): remove UNICODE macro on ANSI GDKX samples so DefWindowProcA matches RegisterClassExA under Clang-CL
85e2d3d|2026-11-22|fix(dstorage): eliminate lost-wakeup race in DStoragePriorityScheduler::NotifyWorkAvailable
c108476|2026-11-22|feat(gdkx): add 2 more untouched MIT _GAMING_XBOX_SCARLETT samples (6/6 live 4K verified)
380db11|2026-11-22|feat(gdk): offline-first XSTS default, XTaskQueue waiter re-arm, dialog/XError validation, and UMS/wldp shims
6591162|2026-11-22|feat(gpu): add Vulkan OPTIONS2/7 & custom border color, tiled placed textures, and XG stencil/HTILE parity
dcfe215|2026-11-22|feat(gpu): wire live Xbox DXBC/RDNA2 + RayQuery pipeline into DeviceX and unified texture sync
8bb8429|2026-11-22|feat(shader): harden RDNA2 CFG/MIMG/V#/LDS safety and expand real-GPU verification to 700+ opcodes
963bd26|2026-11-22|merge(s13): Worker 1 live Xbox DXBC/PM4/RDNA2 + RayQuery DeviceX pipeline & unified texture sync
78ab969|2026-11-22|merge(s13): Worker 2 Vulkan OPTIONS2/7 & custom border color, tiled placed textures & XG stencil/HTILE
a6476a2|2026-11-22|merge(s13): Worker 3 shader CFG/MIMG/V#/LDS safety & 764/779 real GPU-verified opcodes
b7f1aaf|2026-11-22|merge(s13): Worker 4 offline-first XSTS, XTaskQueue waiter re-arm, dialog/XError bounds & UMS/wldp shims
40320bf|2026-11-22|merge(s13): Worker 5 6/6 untouched MIT _GAMING_XBOX_SCARLETT samples (+SimpleBezier, +SimpleCompute)
a7bf894|2026-11-22|feat(sprint13): finalize live Xbox shader E2E, VK/XG parity, 764/779 GPU opcodes, 6/6 GDKX & 66/66 status docs
762771c|2026-11-22|fix(test): guard hardware-Vulkan checks in test_s13_shader_cfg_safety for headless CI runners
0f01d67|2026-11-22|chore(docs): streamline repository wording across docs, comments, and wiki
5d3809b|2026-11-22|fix(rdna2): lift LDS ds_sub as a CAS loop so NVIDIA can build the pipeline (#26)
76adeff|2026-11-22|fix(dstorage): read titles' requests with the Xbox dstorage_xs.h layout (#25)
ec89742|2026-11-23|feat(xgameruntime): XSystemHandleTrack / XSystemIsHandleValid handle registry (#24)
4af2702|2026-11-23|fix(rdna2): avoid AMD RDNA2 v_mul_lo_u16 mask elision, harden S12 audio stress & update HW reports
f350622|2026-11-23|chore(docs): refresh 67/67 CTest status badges
34bfbfc|2026-11-23|chore(release): prepare v0.6.0-alpha (Sprints 11-13, 6/6 untouched GDKX samples, 764/779 real-GPU RDNA2, 67/67 CTest)
db2b053|2026-11-23|feat(bvh): export XboxBvhBuilder C++ ABI, dual-path BVH4 rehydration, and DXR StateObject lifter
669f0b9|2026-11-23|feat(shims,loader): Sprint 14 P0/P1 XAudio2 ordinals, FFX 2.0 ABI, XMem stats, XCurl/HC offline exports, PE hardening & GameConfig PLS
d467436|2026-11-23|fix(rdna2): use atomic-free per-lane ballot for sv64 VOPC (#29) & log HW reports (#30, #31)
a79fd8d|2026-11-23|feat(dstorage,xg): DirectStorage X2, IOCP >64GiB I/O, Xbox Zlib/BCPack/Swizzle & MSAA layout
a92359c|2026-11-23|feat(d3d12_xs): Sprint 14 Agility SDK, unwrapped Enhanced Barriers, live VRAM residency & PSO hardening
8d285d9|2026-11-23|feat(gpu): fix Issue #29 RDNA4 VOPC, SM 6.6 + off-C cache, RayQuery wiring, radar rules, 779/779 + live MS/AS
163d93b|2026-11-23|merge(s14): Worker 1 P0 shims, XAudio2 ordinals, FFX 2.0 ABI, middleware routing & full MicrosoftGame.Config
67ce440|2026-11-23|merge(s14): Worker 2 D3D12X Agility SDK, unwrapped Enhanced Barriers, live VRAM residency & PSO hardening
c43d660|2026-11-23|merge(s14): Worker 3 XboxBvhBuilder 12-method C++ ABI, dual-path BVH4 rehydration & LiftDxrStateObjectDesc
aa33c6b|2026-11-23|merge(s14): Worker 4 DirectStorage X2, RFC1950 Zlib, BCPack BC1-BC7, concurrent IOCP (>64GiB) & MSAA tiled layouts
cc6bace|2026-11-23|merge(s14): Worker 5 Issue #29 RDNA4 VOPC hardening, SM 6.6 + all-stage profiles, RayQuery lifter & 779/779 opcodes
59f8094|2026-11-23|chore(cmake): register modular UE5 Matrix Awakens import bringup cmake files
a83fc6c|2026-11-23|feat(shims): add dxcompiler_xs, XCRDAPI DStorageGetFactory & xg_xs XGCreateTextureComputer
5327625|2026-11-23|feat(loader): register xapu.dll shim routing and ext-ms-win-rtcore-ntuser API-sets
002289a|2026-11-23|feat(loader,launcher): route dxcompiler_xs, synthesize static missing-DLL stubs & add full inspector copy
66c8219|2026-11-23|feat(shims): export d3d12_xs IID_ID3D12* GUIDs and xmem XMemTransferMemory
3c972bf|2026-11-23|feat(audio): add xapu.dll shim and AcpHal Apu* heap exports for Minecraft/UE5
6b7b51e|2026-11-23|Merge branch 'mine-w2-d3d12-xmem' into feat/minecraft-preview-bringup
fd65845|2026-11-23|Merge branch 'mine-w3-routing-apisets' into feat/minecraft-preview-bringup
b5787d2|2026-11-23|feat(shims): implement AcpHal heap, XMem allocation hooks & MFPlat System32 forwarding
acc9bc8|2026-11-23|feat(d3d12_xs): export D3D12 COM IID GUID data symbols and D3D12Xbox process/device helpers
ff6ff88|2026-11-23|merge(ue5): Worker 1 dxcompiler_xs, XCRDAPI DStorageGetFactory & xg_xs XGCreateTextureComputer
100e39c|2026-11-23|merge(ue5): Worker 2 d3d12_xs COM GUID data exports & D3D12Xbox helpers
1d3014c|2026-11-23|merge(ue5): Worker 3 AcpHal APU heap, XMem allocation hooks & MFPlat System32 forwarding
4d7cf5e|2026-11-23|merge(ue5): Worker 4 dxcompiler_xs routing, static missing-DLL stub synthesis & Launcher copy UX
cd71bc6|2026-11-23|feat(sprint14): wire LiftDxrStateObjectDesc into DeviceX, 779/779 GFX10.3 opcodes, 72/72 CTest & status docs
6c40c28|2026-11-23|fix(d3d12_xs): guard D3D12_DEPTH_STENCIL_DESC2 for MinGW-w64 d3d12.h compatibility
675d9f4|2026-11-23|fix(xmem): support real 2-arg Xbox XMemGetWorkingSetStatistics(flags, outStats) ABI
2241564|2026-11-23|fix(gpu): Intel Arc B390 mesh PSO/Vulkan hardening & log RTX 3060 HW report
2b42f72|2026-11-23|fix(core): release CallTelemetry mutex before forwarding to hook DLL
0019eff|2026-11-23|feat(gdk): align GameInput COM ABI, legacy XGameRuntime IIDs, XGameStreaming slots, and P:\ PLS VFS
cb4e44f|2026-11-23|feat(audio): support retail Xbox AcpHal.dll and xapu.dll ABI for FMOD
09c65d1|2026-11-23|feat(d3d12_xs): add retail Scarlett ID3D12Device/CommandQueue/CommandList ABI proxies and desktop alignment
ac47c20|2026-11-23|merge: integrate feat/minecraft-preview-bringup (09c65d1) into main (75/75 CTest)
bc7ae08|2026-11-23|fix(tests): use temp_directory_path in test_mine_routing_apisets for portable bundle smoke test
3cfbced|2026-11-23|test(ue5): add 7-DLL Matrix Awakens E2E import bringup test & CI dxcompiler_xs packaging
7c1a928|2026-11-23|docs(status): refresh badges & README for 31 DLLs / 1213 exports and 77/77 CTest
59eb76b|2026-11-23|merge(main): integrate retail Scarlett D3D12/GameInput/AcpHal/xapu/XMem ABI fixes
28ac576|2026-11-23|chore(docs): log NVIDIA RTX 4070 Laptop GPU (Ada Lovelace SM 6.7, 67/67 PASS) HW report and refresh 75/75 status
51e6a02|2026-11-23|docs(readme): sync Hardware Tier & Community Hardware Reports tables (RTX 4070 Laptop, RTX 3060, Arc B390, 75/75 CTest)
8e23124|2026-11-23|Sprint 15 W1: 20x rdna2_bench speedup (11.9s) & honest 775/779 GPU + 4/4 trap harness
e3c8f2c|2026-11-23|Sprint 15 W4: Loader/VFS hardening, XAsync callback wait, GDK ABI & ETW controller
3112404|2026-11-23|fix(gameinput): match official Microsoft GDK GameInputGamepadButtons bitmask (0x0001..0x2000)
7e2dd9b|2026-11-23|perf(build,test): add shared sccache, parallel CTest locks, rdna2_lifter /Og fix & agent speed rules
5f51f35|2026-11-23|feat(gpu,xg,bcpack): Sprint 15 D3D12X, Vulkan, XG 3D & BC6H/BCPack parity
be0fda1|2026-11-23|fix(vfs,gameinput): hook CRT file APIs for LevelDB/P: saves and fix focus gating & gamepad capability separation
37674f0|2026-11-23|Merge remote-tracking branch 'origin/main' into feat/minecraft-preview-bringup
004fc58|2026-11-23|Merge branch 'main' into feat/ue5-matrix-awakens-imports
e0a9505|2026-11-23|Sprint 15 W2: RDNA2 shader & ISA peer parity across disasm/IR/CPU/HLSL/SPIR-V
d71ed46|2026-11-23|fix(d3d12_xs): do not auto-evict live GPU heaps without explicit budget override
ce6b27d|2026-11-23|merge(s15): integrate s15-w1-rdna2-bench
3d4a8d7|2026-11-23|merge(s15): integrate s15-w2-shaders (ds_swizzle, PC-rel branch, SMEM/LDS D16, Wave64, Blackwell/NaN, SPIR-V 1.2)
2733bfb|2026-11-23|merge(s15): integrate s15-w3-graphics (placed-tex promotion, UAV/MSAA/DXGIX, Vk GPU rank, XG 3D, BC6H/TTPF)
ad6643c|2026-11-23|merge(s15): integrate s15-w4-shims (XML entities, VFS/ReplaceFile/ANSI/delay-load, XAsync, GDK, KernelX, ETW)
2761df0|2026-11-23|feat(sprint15): Zero-Gaps Sprint ÔÇö 28x faster rdna2_bench (8.68s), honest 775/779 GPU + 4/4 trap harness, full peer parity (79/79 CTest)
f7566aa|2026-11-23|fix(d3d12_xs): stage unaligned Xbox CopyTextureRegion footprints to 512B/256B desktop alignment and skip failed command lists
c7a7e46|2026-11-23|docs(readme,status): highlight Sprint 14-15 next-gen AAA gap closure, active Minecraft/UE5 bring-up & updated compatibility row
e7c5f85|2026-11-23|fix(vfs): pass through host D: paths in GetFileAttributes/CreateDirectory hooks (unblocks Minecraft LevelDB world creation)
74c3a5c|2026-11-23|Merge branch 'main' into feat/minecraft-preview-perf
cbfd4f1|2026-11-23|Merge branch 'feat/ue5-matrix-awakens-imports' into feat/minecraft-preview-perf
5ce3bc5|2026-11-23|perf(d3d12_xs,telemetry): async PresentX ring, disable retail per-frame readback, fullscreen stretch fix, staging ring & sharded telemetry
f2c0610|2026-11-23|fix(d3d12_xs): restore ScarlettRetailDeviceProxy wrapping in D3D12XboxCreateDevice after UE5 merge
b01ed32|2026-11-23|chore(sprint16): split mflnmhelp/mfhevcdec shims and register s16 modular cmake hooks
6a6243e|2026-11-23|fix(vfs,audio,dstorage,d3d12_xs): clean-room hardening & cached barrier unwrap
e48fede|2026-11-23|test(ue5): update DllRoutingTable rule counts for xapu.dll merge (20/25/31)
5788857|2026-11-23|feat(tools): add os_live_profiler real-time resource & x64 stack-sampling profiler
c922caa|2026-11-23|feat(s16-w4): complete VFS CoW/NT hooks, GDK runtime/UI parity, and retire all 6 non-media stub DLLs
8e82301|2026-11-23|feat(media,storage,audio): complete Sprint 16 W3 100% hardening & GPU video
969f183|2026-11-23|perf(d3d12_xs,xgr): lock-free 64MiB CBV align ring, L0 WRITE_BACK retail upload buffers & zero-VirtualQuery QueryApiImpl
bd6e9d8|2026-11-23|docs(readme,status): document Minecraft Preview 60 FPS 3D gameplay, perf wins & os_live_profiler
cc86780|2026-11-23|merge(main): integrate feat/minecraft-preview-perf (Minecraft 60 FPS 3D, UE5 7-DLL surface, os_live_profiler, 85/85 CTest)
e44397b|2026-11-23|feat(gpu): Sprint 16 Workstream 1 ÔÇö Graphics Engine 100% completeness
62ee124|2026-11-23|feat(gpu): Sprint 16 RDNA2/GFX10.3 shader compiler 100% completeness, all-7-stage GPU bench & async PSO cache
934879b|2026-11-23|merge(s16): integrate W1 graphics engine completeness (page heaps, unbounded samplers, Copy queue, ExecuteIndirect, FMASK/DCC/HTILE GPU resolve, Vulkan 1.4, residency)
211ec9e|2026-11-23|merge(s16): integrate W2 RDNA2 shader compiler completeness (single s_buffer_load bounds check, BVH clamping, last-mile ISA, all 7 stages GPU verified, MT async PSO cache)
0a65ecd|2026-11-23|merge(s16): integrate W3 storage, codecs, audio & video completeness (DirectStorage X2, BC7 98 presets/TTPF, XAudio2/AcpHal/xapu, mfhevcdec/mflnmhelp, Bink2 GPU YUV->RGBA)
035034e|2026-11-23|merge(s16): integrate W4 OS, VFS, GDK runtime & retire all remaining 6 non-media stub DLLs
afcd167|2026-11-23|fix(input,gpu,perf): instant XInput/Alt-Tab recovery, OreUI gamepad connect, live Raw Mouse & anti-microstutter PresentX/telemetry
74e6c68|2026-11-23|chore(s16): finalize Sprint 16 portability, CI headless/MinGW/portable-smoke fixes & status (89/89 CTest, 0 stub DLLs)
1ffd939|2026-11-23|fix(ci): mark test_s15_rdna2_bench_trap_harness RUN_SERIAL TRUE to prevent parallel CTest wall-clock contention
16c816d|2026-11-23|fix(dstorage): serialize barrier dispatch against in-flight barriers & eliminate priority test wakeup jitter
a208562|2026-11-23|fix(dstorage): eliminate IocpReadOp & CloseInternal condition_variable use-after-free races
4bc2db4|2026-11-23|fix(input,vfs,window): native System32 GameInput GIP backend, target HWND/focus handoff & VFS GetFinalPathNameByHandleW guard
d25a849|2026-11-23|fix(vfs,docs): expand 8.3 short paths in TryMapHostPathToVirtualW (fixes CI) & update README for Sprint 16 (89/89 CTest, 0 stub DLLs)
438fdb0|2026-11-23|fix(tests): let the work-dir checks pass when the build lives on C: (#40)
0a13a0c|2026-11-23|docs(status): log RTX 5090 (#36) and Ryzen Z1 Extreme / Radeon 780M (#38) HW reports (12 PCs verified)
f99d57e|2026-11-23|fix(tests): gate DispatchMesh live test on MeshShaderTier >= 1 (fixes #44)
cee8977|2026-11-23|fix(gameinput): add opt-in F8 cursor-lock auto-recenter and WM_SETCURSOR hiding (#41)
05a24bc|2026-11-23|fix(input,gdk,gpu): fix QPC microsecond overflow, XInput-first polling, Base64 XGameSave names & MSAA sample-count clamping
82bd3fa|2026-11-23|feat(scarlett,d3d12_xs,xmem): clear retail GRDK 2504/2510 boot Stops 1-17 (chad.dll, __XMemAllocationHooks__, Zen 2 CLWB VEH, 4K RT + BVH init)
b9d988c|2026-11-23|docs(readme,status): highlight GRDK 2504/2510 Stops 1-17 (chad.dll, 33 DLLs / 1,360 exports), input/save/MSAA fixes, UE5 4K PresentX & PRs #40/#41/#44
c82ca43|2026-11-23|feat(d3d12_xs,dstorage,xmem,audio): retail GRDK 2504/2510 live 4K PresentX bring-up (Stops 18-34)
0cbc742|2026-11-24|feat(d3d12_xs,rdna2): fix GTX 1650 Ti 0x887A0006 hang & unpack retail XDAT/XBCH/FSSM shaders
e5db0c6|2026-11-24|feat(gpu,xgameruntime): live FSSM v5/v6 RDNA2 PSO lifting on GTX 1650 Ti & GRDK 2504/2510 COM IIDs
4808ad8|2026-11-24|perf(gpu,kernel): 256MB render-graph VA aliasing heap & 64KB dirty-hash mirroring (Stops 35-36)
87cdba6|2026-11-24|docs(readme,status): document GRDK 2504/2510 Stops 1-36, live FSSM v5/v6 RDNA2 PSO lifting, 256MB VA aliasing heap (360+ 4K frames) & HW reports #45-#47 (15 PCs)
1c56104|2026-11-24|fix(audio): restore AcpHal OOM/SHAPE rollback, SRWLOCK sync, and XApu shm test parity
b08fa23|2026-11-24|feat(audio): implement real xapu.dll Opus/PCM decode & SRC engine with SRWLOCK
3e9162c|2026-11-24|feat(audio): implement real chad.dll FFT/IFFT/Convolution DSP engine with SRWLOCK
defb915|2026-11-24|feat(audio): upgrade AcpHal SHAPE/IAcpHal XMA2 decode & XAudio2 SRWLOCK/console ABI
cb55d0b|2026-11-24|test(audio): add focused unit tests for retail xapu, chad, AcpHal & XAudio2 engines
3cc1984|2026-11-24|fix(audio): align external_gdk xaudio2_xs.h with 19-slot IXAudio2Voice vtable and pack(1) ABI
ef763eb|2026-11-24|feat(boot,d3d12_xs,xapu): clear Stops 37-41 to complete 249/249 GTA VI init tasks at 30 FPS
022c5e1|2026-11-24|merge(audio): sync neon/audio with main (ef763eb) keeping real xapu engine & SRWLOCK
ef4448a|2026-11-24|fix(audio): harden Opus MFT System32 loading & retail XApu raw-packet/pitch/channel fidelity
f8e24d4|2026-11-24|feat(gpu): live 4K retail Scaleform rendering via guest descriptor materializer, RECTLIST expansion & RDNA2 MUBUF fixes
bda9242|2026-11-24|fix(gpu,rdna2,input): harden descriptor materializer/barriers, add 6 VOP1 math ops & post-landing shader support
c382f68|2026-11-24|feat(gpu,xmem,loader): space2 multi-texture RDNA2 lifting, UMA shadow sync, paired alias VA & SSE4A
46d8e2c|2026-11-24|Merge branch 'main' into neon/audio
86d18ac|2026-11-24|feat(gpu,input,loader): Scaleform space1 RS ranges, whole-slice upload & live GameInput steering
f538782|2026-11-24|feat(kernel): Lockhart 8 GiB budget, MEM_WRITE_WATCH guest allocs & persistent TitleVAMirror staging
acd0cdc|2026-11-24|feat(d3d12_xs,gdk): 8-slot async upload ring, isolated RT/DS aliasing, lazy CS XDAT lift & GRDK 2504 COM IIDs
9b334cd|2026-11-24|Merge branch 'neon/wt-lockhart-mem'
44b3696|2026-11-24|feat(streaming,loader,bvh): zero-alloc DirectStorage X2 scratch, Zen2 VEH + OEM ConsoleMode hooks, and DXR=0 BVH4 safety
cef939b|2026-11-24|Merge branch 'neon/wt-bvh-slots'
b616d32|2026-11-24|feat(gpu): unlock XDAT FSSM v5/v6 CS/MS/VS/PS lifting and fix VOP2 cndmask & s_endpgm reachability
bd8568e|2026-11-24|Merge branch 'neon/wt-rdna2-fssm'
022a8b2|2026-11-25|feat(d3d12_xs,gpu,loader): unlock RAGE9 720p/4K rendering, async fence pacing & 4GB VRAM scaling
6fb3502|2026-11-26|feat(gpu,d3d12_xs): fix Bink2 video, GFX10 v_cmpx ramps, untiling, and Scaleform FTUE UI
4cfcc43|2026-11-26|feat(gpu,d3d12_xs): lift retail XDAT/Slot57 mesh PSOs, Param0..31/MRT0..7, and prune >32KiB MS outputs
4ea8411|2026-11-26|feat(d3d12_xs): add async UMA readback ring, >65535 dispatch split, and 16-bit RT stand-ins
7220ce9|2026-11-26|feat(gpu,d3d12_xs,loader): add v_cmp_class_f32/f16, Slot129 DispatchMesh, and watchdog/chunk guards
```

## L. Limitacoes desta analise

- **Nenhum teste foi executado.** Nao rodei `ctest`, nao compilei o projeto e nao abri nenhum binario. Tudo o que esta escrito sobre comportamento vem de leitura de codigo, de diff e de mensagem de commit — inclusive os numeros de FPS, contagem de iteracoes, frames e "Stops", que sao **relatos do autor** e estao marcados como tal.
- **Sem GPU e sem Windows de teste.** As afirmacoes sobre `0x887A0006`, `E_OUTOFMEMORY` de driver, comportamento de `Evict` e limites de VRAM sao coerentes com o codigo e com os diffs, mas nao foram reproduzidas.
- **A amostra e de 50 commits.** O pacote de evidencias cobre 50 dos 351; outros ~30 foram abertos por necessidade de fechar cadeias (`git show`, `git log -S`, `git log --first-parent`). Ha commits de 2026-11-22 (o dia com 164 commits) que nao foram lidos individualmente; um defeito introduzido e corrigido dentro do mesmo dia pode nao ter sido visto.
- **Diffs de merge nao aparecem em `git log -S`.** Commits de merge que alteram conteudo so sao visiveis com `-m` ou comparando arvores. Usei comparacao textual de arquivo por revisao para o caso `cbfd4f1` (D-11); outros merges com o mesmo padrao podem ter passado.
- **O repositorio nao contem o titulo.** Nao ha binario, log de execucao ou captura no repositorio. Toda a avaliacao de "boot" e "render" depende do que os commits relatam.
- **`docs/rdna2_coverage.json` nao foi auditado campo a campo.** Os 779/779 e 3.120/3.120 vem desse arquivo, que e entrada do gerador de status; nao verifiquei se cada opcode listado tem teste correspondente.
- **Sem acesso a issues/PRs.** As referencias a #24, #25, #26, #29, #40, #41, #44 aparecem nas mensagens de commit; nao consultei o GitHub, entao o conteudo das discussoes e o estado dos PRs sao NAO VERIFICADOS.
- **A contagem de testes (90) e estatica.** Contei registros nos arquivos de build; nao sei quantos efetivamente compilam e rodam em cada configuracao (Release/Debug, MSVC/Clang-CL/MinGW), nem quantos degradam para WARP ou fazem skip na pratica.
- **Atribuicao de autoria.** 343 dos 351 commits sao de californiaAlmonds; varios commits citam `Co-authored-by: Copilot` e um cita `Claude Opus 5.5`. Nao tentei separar autoria humana de assistida; a analise julga o diff, nao o autor.
- **Numeros do gerador de status.** `89/89`, `90/90`, `1.360`, `1372`, `775/779`, `15 PCs` sao reproduzidos aqui como **o que o repositorio publica**, nao como fato verificado. A unica coisa que verifiquei independentemente foi a contagem de registros de teste (90) e a divergencia interna do README.
