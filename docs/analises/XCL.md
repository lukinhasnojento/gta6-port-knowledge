# XCL - analise de DeepSeek V4.1 Flash

## A. Identificacao

| Campo | Valor |
|---|---|
| Nome | XCL (camada de compatibilidade Xbox Series X\|S -> Windows, sem emulacao de CPU) |
| Repositorio analisado | `repos\XCL` (clone local, somente leitura) |
| Remote | `origin https://github.com/ivall/xcl.git` (o GitHub redireciona para `xcl-dev/xcl`; CONFIRMADO ao abrir `https://github.com/ivall/xcl/pull/123`, que serve `/xcl-dev/xcl/pull/123`) |
| Original / fork | Repositorio original do projeto (nao e fork de terceiros) |
| Branch analisada | `main` (unica branch local e remota; `origin/HEAD -> origin/main`) |
| Commit 0 ("raiz") | `8fa7fa0` - 2026-11-20 - ivall - "Initial XCL source and TTPF viewer" |
| Ultimo commit | `8e87fc7` - 2026-11-25 - Ray - "Merge pull request #1898 from emirduran707/dxil-round-denorm-inf-nan" |
| Periodo coberto | 2026-11-20 a 2026-11-25 (6 dias distintos com commit) |
| Total de commits | 853 alcancaveis de HEAD (853 de todas as refs; 0 so em outras branches) |
| Merges | 274 de 853 (32%) - fluxo de trabalho em lotes de PR |
| Arquivos versionados | 311 |
| Commits examinados em profundidade | 43 blocos do pacote de evidencias + ~25 abertos diretamente com `git show`/`git log -S` (68 no total, ~8% da historia, mas concentrados em 100% dos reverts e nas cadeias de correcao) |
| Modelo responsavel | DeepSeek V4.1 Flash |
| Data da analise | 2026-11-25 |

Contagem por dia (`_fatos/XCL_fatos.md`): 2026-11-20 = 3, 11-21 = 119, 11-22 = 85, 11-23 = 345, 11-24 = 288, 11-25 = 13.

Autores: 343 commits de `ivall` (40%), depois `bmjask` (80), `theaustinayers` (53 em duas identidades), `gefydev` (59 em duas identidades), `Kalmarv` (28), `raylitebull` (25) e uma cauda longa de ~85 contribuidores de um ou dois commits. Ha 6 commits de `Claude <noreply@anthropic.com>` e varios trailers `Co-Authored-By`/`Assisted-by` (Claude Opus 5.5, Antigravity/Gemini, Codex/GPT), o que explica o volume e o estilo dos commits.

Churn: `crates/xcl/src/hle/d3d12x/real.rs` (108 toques), `HANDOFF.md` (94), `crates/xcl/src/gpu/recomp/tests.rs` (80), `crates/xcl/src/gpu/recomp/emit.rs` (64), `README.md` (54), `hle/d3d12x/list_x.rs` (45), `gpu/recomp/mod.rs` (43), `hle/kernelx.rs` (41).

## B. Resumo tecnico

XCL roda jogos Xbox Series X|S em Windows x64 **sem emular a CPU**: o codigo do jogo executa nativamente e o XCL substitui as bibliotecas do console. Tres pilares:

1. **Loader / HLE** (`crates/xcl/src/{loader,resolver,hle/*,pe,manifest}.rs`): PE loader proprio, hooks de excecao C++ (`eh.rs` via `RtlPcToFileHeader`/`RtlAddFunctionTable`), resolucao em cascata HLE -> DLL do jogo -> Windows -> stub automatico (`data/stubdb.toml`). Implementa `kernelx` (memoria virtual, threads, timers), `XGameRuntime` (XAsync/XTaskQueue/XUser/XPackage/PLS/game saves), `D3D12.X` (o runtime grafico do console), `DirectStorage`, `XAudio2`+Opus, `XNetworking`, `VFS` (overlay de arquivos) e `GameInput`.
2. **D3D12.X** (`crates/xcl/src/hle/d3d12x/`, ~20 modulos): traduz a API grafica do console para D3D12 do PC. Destaques: listas de comando do console (`list_x.rs`, `console_list.rs`, "record writers"), heaps/tiles esparsos (`sparse.rs`, `vram.rs`), allocator pool (`alloc_pool.rs`), PSO assincrono (`pso_async.rs`), texturas do titulo (`title_textures.rs`), fences (`fence_x.rs`), apresentacao (`present.rs`).
3. **Recompilador de shaders** (`crates/xcl/src/gpu/`): decodificador/assembler RDNA2 (`rdna2.rs`), tradutor para HLSL (`recomp/emit.rs` + `recomp/mod.rs`, com forma estruturada ou "pc blocks"), compilacao com DXC in-process (`recomp/dxc.rs`), cache em disco (`recomp/pso.rs`), e — a partir de 2026-11-24 — um **tradutor direto RDNA2 -> DXIL** (`gpu/dxil/*`, "Direct DXIL", PR #1426) mais um backend **per-lane** (`gpu/recomp/lane_masks.rs`, `recomp/lower.rs`, `recomp/lower_dxil.rs`, PR #1745) que preserva o CFG do console e trata EXEC como bit por lane.

Estado atual (HEAD `8e87fc7`): o caminho **padrao continua sendo HLSL -> DXC**. O tradutor direto (`XCL_DIRECT_DXIL=1`) e o per-lane (`XCL_PER_LANE=1`, exige o direto) sao **opt-in** por variavel de ambiente (`lower_dxil::enabled()`, `pso::Backend::current()`), e ambos caem para o HLSL quando recusam um programa. Consequencia importante para ler este documento: os defeitos do backend per-lane/direto **nao afetam uma execucao padrao**; afetam cobertura e experimentos, e todos os relatos de "cai para o HLSL" nos commits sao sobre isso.

Documentacao no repo: `README.md` (manual), `docs/specs/2026-11-20-xcl-design.md`, `docs/specs/2026-11-24-direct-dxil.md`, `HANDOFF.md` (1854 linhas, diario tecnico), `CONTRIBUTING.md` e `AGENTS.md` (regras de contribuicao e de agentes de IA).

## C. Linha do tempo

**Fase 0 - 2026-11-20 (3 commits): importacao e sondagem.**
`8fa7fa0` "Initial XCL source and TTPF viewer" e depois `a21d3df` "Session 5: console DLLs, tiling, BVH format, compute PSO translation" e `57b529c` "Console root-argument layout, D3D12.X inline command list finding, memory cap for probes". Aqui nascem o layout de root arguments do console, o formato de lista de comando inline e a biblioteca de tiling — e ja aparece `gs_start` (a forma split do geometry), que sete dias depois estara no centro de um defeito (D-19).

**Fase 1 - 2026-11-21 (119 commits): amplitude do HLE.**
Sequencia de "bring-up" por API: `3c546ef` (biblioteca de layout de textura do console, `xg_xs.dll`), `b56a1b2`/`e714e2a` (listas de comando do console; XboxBvhBuilder construindo BVH na GPU do PC), `3453dd9` (buffers do console em memoria unificada), `71d2f82`/`26cf5ec` (correcoes de DXIL validation: `SampleGrad` em cube, depth conservador), `885e6a2`/`576b4db` (XGameRuntime pela tabela de thunks GDK), `2bd8fcc` (Opus real via MFT do Windows), **`f3b15b9` "Title memory above 4 GB: UMA in up to four 4 GiB windows"** — a origem de toda a saga de janelas de memoria (D-01) —, `98d32ec` (traducao do repositorio para ingles) e `bb363d1` (desfaz um `rustfmt` de crate inteiro mantendo as mudancas: o repo nao e rustfmt-clean e nao deve ser).

**Fase 2 - 2026-11-22 (85 commits): titulos reais.**
`507c441` (XDefiant: title screen), `3a8bc96`/`8eb3cf4`/`9e759de` (Fortnite: splash, UI, main menu), `550d979`/`bb58372` (Minecraft: create-world, gamepad), `126268c` (Unicorn Overlord jogavel), `0fb82c6` (raytracing/BVH 64-bit), `3ecf4cf` (wave64 do console em GPUs so-wave32 por duas metades) e a serie de "GPU tests skip" (`ff9430e`, `66e23c5`, `2bc2e37`) — o padrao `SKIPPED <test>: <reason>` que mais tarde sera aplicado aos testes per-lane (D-13).

**Fase 3 - 2026-11-23 (345 commits): lote grande de PRs, e o unico revert.**
Entram #24 a #130 em bloco. No mesmo minuto (15:01) entram **#123** (`778e329`, carry de janela de memoria) e **#124** (`828d9db`, MS/GS com as janelas do titulo); **onze minutos depois** `70f8dc4` reverte #123 porque o compilador de shader da AMD passou a crashar XDefiant (D-01/D-02). Seguem `c4df2fe` (#119, dots 8-bit por `dot4_u8`), `1a51a82` (#127, checagem de CPU no start), `23c53fc` (#129, frame clock), e depois o segundo lote (#244-#762) com `fe7d70b`/`927adcc` (#187, janelas de memoria pelo descriptor heap, sem switch por acesso), `b8b1f49` (estruturacao de saltos para loops externos — que **introduz** D-03), `a99f64c` (sem `VirtualQuery` por instrucao emulada) e `7a033c8` ("Retail game goes past its first menu").

**Fase 4 - 2026-11-24 (288 commits): Direct DXIL e o backend per-lane.**
De manha: `746f7dd`/`a8569bf` trazem o tradutor direto RDNA2->DXIL (#1426); `f9e5ee9` (#792, 00:16) resolve D-01 de vez. Ao meio-dia comeca a serie per-lane de `gefydev`: `1bab757` (analise de lane masks por BDD), `b3c0b37` (lowering kernel -> IR), `f33f640` (compute), `82c875d`/`27751a0`/`3aea7a9` (programas de teste f32/f16/f64/inteiros que **revelam** D-04, D-06, D-07), `ddfbf39` (runtime opt-in `XCL_PER_LANE=1`), `d2cc6ff`/`27d28ce`/`a24899e` (EXEC de lancamento, `s_wqm`, VS/PS), `372be9d` (s_cselect/s_wqm/v_cmpx), `45025eb`/`f90033c` (operacoes cross-lane), `3827fb5` (coalescencia de runs guarded), `66879ef`/`9dcf2f6`/`fe78c81` (tres rodadas de correcao de semantica de mascara), `188ee17` (D-04), `bfb8133` (D-11). Em paralelo, no D3D12.X: `3bc7143` (#1747, freeze de 180 s), `0b64aa0` (#1748, OOM de shaders gigantes), `4e3515e` (#1791, ExecuteIndirect na NVIDIA), `6c89285` (#1195, ExecuteIndirect + compute nao traduzido), `f3242b2` (#1689, geometry split).

**Fase 5 - 2026-11-25 (13 commits): cauda de correcoes.**
`ecb7547` (underflow em `s_setreg`), `50228bb` (#1930, CSE atraves de `dx.op.discard`), `15fff3b` (mascaras inline/NULL no per-lane), `425b869` (#1900, ordem `textureLoad`/`getDimensions` para nao crashar o `amdxc64`), `8e87fc7` (#1898). Ou seja: o ultimo dia e dominado por **correcoes de borda do tradutor direto**, nao por funcionalidade nova.

## D. Catalogo de defeitos

> Convencao: "introduzido" = commit cujo diff criou o comportamento defeituoso; "correcao definitiva" = commit em HEAD que elimina a causa (nao apenas o sintoma). Onde a ordem topologica diverge da data de autor (lotes foram reescritos no merge), vale a topologia, verificada com `git merge-base --is-ancestor`.

### D-01 - Janela de memoria UMA escolhida pelo dword alto da base, sem carry/borrow de 32 bits

- **Descricao / sintomas.** Com mais de uma janela de 4 GiB (`mem_windows > 1`), um acesso de shader cujo **offset** estoura o dword baixo do endereco (`base = ...FFFF_F000` + `0x2000`) ia para a janela da *base*, no endereco baixo enrolado. O mesmo com offset global negativo que pede empréstimo. Efeito: leitura/escrita de memoria errada (dados de outro recurso), silenciosamente. Afetava `s_load_*`, `s_buffer_load_*`, MUBUF load/store/atomic/d16 e FLAT/GLOBAL (normal e atomico).
- **Causa raiz.** `Em::window(hi)` emitia `xcl_w = ({hi}) & 3u` a partir apenas do dword alto da base, **antes** de somar os offsets; o offset era somado em 32 bits no endereco baixo (`uint {t}_a = ... + {t}_off`), perdendo o carry.
- **Introduzido por.** `f3b15b9` (2026-11-21, "Title memory above 4 GB: UMA in up to four 4 GiB windows") — `git log -S"fn window(&mut self, hi: &str)"` retorna `f3b15b9` e `f9e5ee9`.
- **Tentativas de correcao.**
  1. `a787c52` (branch `gefydev/recomp-window-carry`), mergeado em `778e329` (#123) em 2026-11-23 15:01: novo `window_at(hi, terms, signed)` que soma `(uint64_t)termo`s, extrai `>> 32` como carry e faz `xcl_w = (hi + carry) & 3u`; `flat_window`/`buffer_window` reescritos; teste `gpu_memory_window_carry` (55 linhas).
  2. `70f8dc4` (15:12) **reverte** o merge (ver D-02). O defeito volta.
  3. PR #138 (`b7177b1`, "carry in 32 bits") — **nao esta neste clone** (`git cat-file -t b7177b1` => "Not a valid object name"), fechado como superseded.
  4. `fe7d70b` + `927adcc` (#187): janelas pelo **descriptor heap**, sem `switch` por acesso (`Options::heap_windows`), com fallback para a forma switch quando o driver recusa.
- **Correcao definitiva.** `f9e5ee9` (merge `aaa00b2`, #792, 2026-11-24 00:16): endereco efetivo de 64 bits calculado **antes** de escolher a janela (`uint64_t {t}_ea = ...`), `window_read`/`window_write` separados, base da UMA vinda por novo slot de I/O (`IO_UMA_BASE_HI`, slot 112), `default` dos accessors passa a devolver 0 / descartar em vez de apelidar `mem`. `fn window` foi **removida** (`Select-String "self\.window\("` em HEAD: nenhum resultado).
- **Arquivos.** `crates/xcl/src/gpu/recomp/emit.rs`, `recomp/tests.rs`, `recomp/mod.rs`, `recomp/runner.rs`, `hle/d3d12x/bind.rs`, `hle/d3d12x/console_list.rs`.
- **Evidencia.** `git show a787c52`; `git show f9e5ee9 -- crates/xcl/src/gpu/recomp/emit.rs` (o diff mostra `- xcl_w = ({hi}) & 3u` virando `+ xcl_w = select({t}_hi == 0u, 0u, {t}_hi - xcl_uma_base)`); teste em HEAD `crates/xcl/src/gpu/recomp/tests.rs:3326` `memory_windows_reject_out_of_uma_aliasing`, com o caso `// 64-bit carry target: (w0 + 0xFFFF_FFF0) + 0x20 == w1 + 0x10`; a condicao `assert!(base_hi >= 4)` torna o teste sensivel ao `& 3u` antigo.
- **Estado atual.** Corrigido. O teste `gpu_memory_window_carry` **nao voltou** com esse nome; a cobertura equivalente (carry 0->1, borrow abaixo da janela 0, `hi == 0`) vive em `gpu_memory_windows_reject_out_of_uma_aliasing` (113 linhas) e `gpu_memory_windows_above_4_gib`.
- **Confianca.** CONFIRMADA.

### D-02 - Regressao: o codegen de #123 derrubou o compilador de shader da AMD (amdxc64)

- **Descricao / sintomas.** Com `a787c52` aplicado, XDefiant crasha ~10 s depois de entrar, **dentro do compilador de shader da AMD**: `unhandled exception 0xc0000005 in amdxc64.dll+0xe8f875 (write at address 0x2000000c)`, ao criar o pipeline do compute shader `8e4af296b9d7fd9c` (2 552 instrucoes RDNA2, 4 janelas de memoria). Deterministico; reverter **apenas** #123 faz o jogo rodar.
- **Causa raiz.** Nao e erro de HLSL: o HLSL e valido. A *forma* do codigo gerado — aritmetica de 64 bits por acesso antes do `switch` de janela — dispara um bug do compilador da AMD. Diagnostico do mantenedor: "likely the 64-bit address arithmetic before the window switch at every access trips the AMD driver".
- **Introduzido por.** `a787c52` (merge `778e329`).
- **Tentativas de correcao.**
  1. `70f8dc4` (revert mecanico, `-m 1` de `778e329`, referenciando `5231403`): devolve `emit.rs` ao estado anterior (restaura `self.window(&d1)` em 5 pontos e remove `window_at`/`buffer_window`/`flat_window(off)`), apaga `gpu_memory_window_carry`.
  2. #138 (`b7177b1`, ausente deste clone): carry em 32 bits por comparacao (`xcl_ws += t; xcl_wc += (uint)(xcl_ws < t)`), zero `uint64_t` nesse caminho. Nao mergeado na epoca (o mantenedor queria cache quente nos outros merges).
  3. #187 (`fe7d70b`/`927adcc`): ao remover o `switch` por acesso (descriptor heap), muda a forma do codigo o suficiente para o problema de driver deixar de dominar.
- **Correcao definitiva.** `70f8dc4` para o crash; `f9e5ee9` restaura a correcao funcional — e **volta a usar `uint64_t`**, a mesma forma acusada em #123, o que so e defensavel porque (a) o caminho padrao passou a ser o descriptor heap e (b) a substituicao do `& 3u` por um range-check muda o padrao. Ver G-03: nao ha evidencia, neste repositorio, de que o CS `8e4af296b9d7fd9c` do XDefiant foi re-testado depois de #792.
- **Arquivos.** `crates/xcl/src/gpu/recomp/emit.rs`, `crates/xcl/src/gpu/recomp/tests.rs`.
- **Evidencia.** `git show --stat 70f8dc4` (13 insercoes, **93 remocoes**, dos quais 55 linhas de teste); comentario do mantenedor no PR #123: "This one had to come out again ... With it, XDefiant crashes ~10 s in inside AMD's shader compiler ... while creating the pipeline of a compute shader (8e4af296b9d7fd9c, 2 552 RDNA2 instructions, 4 memory windows)"; `HANDOFF.md:957-959` repete o diagnostico ("Held: **#123** ... reverting only #123 runs").
- **Estado atual.** O crash nao ocorre com HEAD (o commit esta revertido e a solucao posterior tem outra forma). O que se perdeu no revert: (i) a correcao funcional do D-01 ficou ausente por ~12 h (de 2026-11-23 15:12 a 2026-11-24 00:16); (ii) o teste `gpu_memory_window_carry` desapareceu e nunca voltou com esse nome; (iii) nada mais — o diff revertido era puramente aditivo a `emit.rs`.
- **Confianca.** CONFIRMADA (causa e efeito documentados pelo autor do revert; o crash em si nao e reproduzivel aqui).

### D-03 - Guardas da forma estruturada crescem com o quadrado dos flags: 148 MB de HLSL e OOM no DXC

- **Descricao / sintomas.** Um programa de compute do pack retail passou a gerar **148 MB de HLSL a partir de 334 KB de codigo** (3 861 flags pendentes, guardas de ate 597 termos, 99 MB so em linhas de guarda), contra 14 MB na forma `pc` blocks. O DXC consumia memoria de commit a ~1 GB/s ate esgotar a maquina.
- **Causa raiz.** A forma estruturada envolve os itens apos um salto para a frente numa guarda que **lista todos os flags de skip pendentes** e a reabre quando o conjunto muda; com muitos flags simultaneos o tamanho cresce quadraticamente. `b8b1f49` (que estruturou saltos para cabecas de loop externas e para dentro de loops) aumentou muito o numero de flags pendentes em um programa.
- **Introduzido por.** `b8b1f49` (2026-11-24, "jumps to outer loop heads and into loops are structured"). Verificado: `git merge-base --is-ancestor b8b1f49 69a892c` => verdadeiro (a data de autor de `b8b1f49` e posterior a de `69a892c`, porque o lote foi reescrito; a topologia e que vale).
- **Tentativas de correcao.** Nenhuma intermediaria: `69a892c` ("oversized structured code falls back to pc blocks (#1307)") e a unica.
- **Correcao definitiva.** `69a892c`: constante `STRUCTURED_BYTES_PER_CODE_BYTE: usize = 128`; apos `emit_structured`, se `code.len() > 128 * program.len()`, re-traduz com `Options { structured: false, ..opt }`. Teste `oversized_structured_code_falls_back_to_pc_blocks` (200 saltos condicionais que cruzam o limite viram `pc` blocks; 50 nao).
- **Arquivos.** `crates/xcl/src/gpu/recomp/mod.rs` (+10), `crates/xcl/src/gpu/recomp/tests.rs` (+32).
- **Evidencia.** `git show 69a892c`; mensagem: "Since b8b1f49 one compute program of the retail pack reaches it: 148 MB of HLSL from 334 KB of code ... against 14 MB as pc blocks"; "Over the whole pack ... only that program changes, to 13.8 MB. The next largest structured program is at 61 bytes per byte".
- **Estado atual.** Corrigido e estavel: `git log -S"STRUCTURED_BYTES_PER_CODE_BYTE"` retorna **apenas** `69a892c` — o limite nunca foi ajustado. O ganho de `b8b1f49` foi preservado (1 670 -> 170 programas como `pc` blocks).
- **Confianca.** CONFIRMADA.

### D-04 - No backend per-lane, carry-ins e `v_cndmask` liam sempre o bit 0 da mascara

- **Descricao / sintomas.** Nos programas de inteiros de teste, `V_ADD_CO_CI_U32`/`V_SUB_CO_CI_U32` liam carry-in **0 em todas as lanes** (v11, v13, v17, v18 saiam "um a menos" nas lanes 1-31). Ate a correcao, esses programas eram **recusados** com "it reads register 21 the per-lane form did not pass" — ou seja, o defeito estava mascarado por uma recusa.
- **Causa raiz.** `DxilIr` (adaptador IR -> DXIL do per-lane) construia o estado com `Lw::new(...)` mas **nao chamava `Lw::setup_lanes()`** para kernel; sem isso `Lw::lane31` ficava a constante 0 e `Lw::lane_of` tomava o bit 0 da mascara para toda lane. Adicionalmente `#1426` le a mascara como **par de SGPRs** em wave32, e o bridge passava so o SGPR baixo.
- **Introduzido por.** `77db49c` ("the DXIL adapter holds #1426's Lw"), que criou `DxilIr::new` sem setup de lanes.
- **Tentativas de correcao.**
  1. `77a5eee` (split wave64): passou a chamar `lw.setup_lanes()` **apenas** quando `lw.split` — corrigiu o caso split, deixou o kernel comum quebrado.
  2. `82c875d` (testes de inteiros) documenta o defeito no proprio commit: "the kernel form never sets up #1426's lanes (`DxilIr::new` with `Entry::Kernel` calls no `Lw::setup_lanes*`), so `Lw::lane31` stays 0".
- **Correcao definitiva.** `188ee17` ("carry-ins and v_cndmask through #1426 read the right lane bit"): `Entry::Kernel => lw.setup_lanes()` incondicional, e o bridge passa o SGPR superior (0) com o bit da lane no inferior. Teste: o programa de carries agora roda por lane e concorda com o caminho HLSL na GPU.
- **Arquivos.** `crates/xcl/src/gpu/recomp/lower_dxil.rs` (linhas 81-94 em HEAD), `crates/xcl/src/gpu/recomp/lower.rs`.
- **Evidencia.** `git show 188ee17` e `Select-String lower_dxil.rs -Pattern "a kernel always"` (comentario em HEAD: "a kernel always: `Lw`'s lane bit, `lane31`, is what #1426's lowering of a carry-in or a `v_cndmask` reads, and stays 0 without it").
- **Estado atual.** Corrigido. Impacto historico limitado: `ddfbf39` (que ligou o per-lane no runtime) e de 15:51 e `188ee17` de 16:21 (datas de autor), e mesmo em HEAD o per-lane exige `XCL_PER_LANE=1`.
- **Confianca.** CONFIRMADA.

### D-05 - Typo de campo (`OPSEL` em vez de `OP_SEL`): o modificador `op_sel` do VOP3 nunca era recusado

- **Descricao / sintomas.** Uma instrucao VOP3 com `op_sel` (que escolhe a metade alta/baixa de um operando) era baixada **como se nao tivesse modificador**: o lowering nao sabia honrar `op_sel` e deveria recusar o programa (fallback para HLSL). Resultado: miscompile silencioso, nao um erro.
- **Causa raiz.** A guarda era `["ABS","NEG","CLAMP","OMOD","OPSEL"].iter().any(|f| fv(f).unwrap_or(0) != 0)`; o campo nao existe (`rdna2::field_value` devolve `None`, `unwrap_or(0)` = 0), portanto **a guarda nunca disparava**.
- **Introduzido por.** `b3c0b37` ("lower kernel programs to DXIL-shaped IR on the lane masks") — `git log -S'"OPSEL"'` retorna `b3c0b37` e `ac34cb6`.
- **Tentativas de correcao.** Nenhuma anterior; achado "de passagem" enquanto se corrigia outra coisa.
- **Correcao definitiva.** `ac34cb6`: `"OPSEL"` -> `"OP_SEL"`, mais o teste `vop3_operand_selects_are_refused` (um `V_ADD_NC_U32` com `OP_SEL 1` e recusado; sem `OP_SEL` passa).
- **Arquivos.** `crates/xcl/src/gpu/recomp/lower.rs`.
- **Evidencia.** `git show ac34cb6 -- crates/xcl/src/gpu/recomp/lower.rs` (a linha `-... "OPSEL" ...` / `+... "OP_SEL" ...`); mensagem do commit: "a VOP3 instruction's op_sel was never refused (the field is `OP_SEL`, not `OPSEL`)".
- **Estado atual.** Corrigido. Janela de exposicao: `b3c0b37` (12:45) a `ac34cb6` (13:07), ~22 min, e o per-lane ainda nao estava ligado no runtime (isso so acontece em `ddfbf39`, 15:51) — logo, sem impacto em usuario.
- **Confianca.** CONFIRMADA.

### D-06 - Produtos f16 do tradutor direto arredondam para zero em vez de para o par mais proximo

- **Descricao / sintomas.** `V_MUL_F16` e `V_PK_MUL_F16` davam `0x7BFF` onde console e caminho HLSL dao `0x7C00` (infinito por overflow), e os empates de denormal arredondavam **para baixo** (`x * 2^-15` nas lanes 3, 7, ..., 31).
- **Causa raiz.** O lowering de `#1426` escreve f16 atraves de `LegacyF32ToF16`, cujo arredondamento e **definido pela implementacao** (a propria documentacao de `gpu/dxil/ops.rs` registra: "the RTX 4090 rounds towards zero (others round to nearest even)"); o console arredonda para o par mais proximo (MODE.FP_ROUND = 0).
- **Introduzido por.** `746f7dd`/`a8569bf` (o tradutor direto, PR #1426) — o comportamento de `LegacyF32ToF16` nao foi escolhido, mas o uso dele nesses dois opcodes e a causa.
- **Tentativas de correcao.** `fe8dc84` ("f16 products stay on the fallback until #1426 rounds them"): adiciona `"V_MUL_F16"` e `"V_PK_MUL_F16"` a `DIFFERS_ON_1426` no bridge do per-lane, fazendo esses programas cairem para o HLSL. E mitigacao (recusa), nao correcao.
- **Correcao definitiva.** **SEM CORRECAO** no periodo analisado.
- **Arquivos.** `crates/xcl/src/gpu/recomp/lower.rs` (`DIFFERS_ON_1426`), `crates/xcl/src/gpu/dxil/ops.rs`, `crates/xcl/src/gpu/dxil/lower*.rs`.
- **Evidencia.** `Select-String lower.rs -Pattern DIFFERS_ON_1426` em HEAD: a lista ainda contem `"V_CVT_F16_F32"`, `"V_MUL_F16"`, `"V_PK_MUL_F16"`, com o comentario "(vladbondarenko97's f16 programs; itsSio is fixing it on `main`)"; `git log -S"LegacyF32ToF16"` retorna apenas `a8569bf` (introducao) e `fe8dc84` (documentacao/mitigacao).
- **Estado atual.** Aberto. Como o per-lane e opt-in e o direto tambem, o efeito e "programas com multiplicacao f16 caem para o HLSL" — custo de cobertura, nao corrupcao visivel ao usuario padrao.
- **Confianca.** CONFIRMADA (o comportamento); a afirmacao de que "itsSio esta corrigindo em `main`" e externa a este clone: NAO VERIFICADA.

### D-07 - `V_CVT_F16_F32` do tradutor direto devolve 0 abaixo do menor denormal f16

- **Descricao / sintomas.** Converter para f16 um valor menor que o menor denormal representavel devolve **0**, onde console e caminho HLSL arredondam para o denormal.
- **Causa raiz.** Mesma familia de D-06: a semantica do lowering de `#1426` para essa conversao difere da do console.
- **Introduzido por.** `746f7dd`/`a8569bf` (tradutor direto).
- **Tentativas de correcao.** `878afda` ("the per-lane form lowers unknown instructions with #1426's semantics") cria a lista `DIFFERS_ON_1426` com `V_CVT_F16_F32` — de novo mitigacao por recusa (fallback HLSL). O proprio commit diz que o problema foi "reported on #1426".
- **Correcao definitiva.** **SEM CORRECAO**.
- **Arquivos.** `crates/xcl/src/gpu/recomp/lower.rs`.
- **Evidencia.** `git show 878afda` (a lista nasce com um item); `Select-String` em HEAD confirma o item.
- **Estado atual.** Aberto (o opcode continua em `DIFFERS_ON_1426`).
- **Confianca.** CONFIRMADA.

### D-08 - O caminho HLSL perde `-0.0` no driver AMD e o runtime compila sem `-Gis`

- **Descricao / sintomas.** No teste de f32 do per-lane, o **caminho HLSL** (referencia) devolveu `+0` onde o console e o DXIL per-lane dao `-0`, em `floor(-(+0)) * 2` (lane 16, v10), num RX 9070 XT. O teste comparava com `==` de float, que nao distingue `-0.0` de `0.0`, e assim escondia a diferenca.
- **Causa raiz.** "DXC's `fast` flags include no-signed-zeros and the AMD driver folds by them". Sem `-Gis`, o compilador pode dobrar `-0` em `+0`.
- **Introduzido por.** NAO VERIFICADO quanto ao commit exato (o defeito e do par DXC+driver, nao de uma linha); o ponto de observacao e `b1bfd1e`.
- **Tentativas de correcao.** `b1bfd1e` ("Tests: the f32 reference keeps signed zeros on every driver"): a **referencia de teste** passa a compilar com `-Gis` e `check_f32_rounding` compara `v10` bit a bit (`r[10] == (2.0 * (-x).floor()).to_bits()`).
- **Correcao definitiva.** **SEM CORRECAO para o caminho de runtime.** O commit e explicito: "the HLSL path itself still gives +0 there on that driver".
- **Arquivos.** `crates/xcl/src/gpu/recomp/dxc.rs` (invocacao do DXC: `-E -T -HV 2021` + `-O0`/`-O1`, sem `-Gis`), `crates/xcl/src/gpu/recomp/lower_dxil.rs` (so os testes usam `-Gis`).
- **Evidencia.** `git show b1bfd1e`; `Select-String -Path crates\xcl\src\gpu\**\*.rs -Pattern '\-Gis'` em HEAD: apenas duas ocorrencias, ambas em `lower_dxil.rs` (testes, linhas 665 e 886) e um comentario em `emit.rs:138`.
- **Estado atual.** Aberto no runtime, fechado no teste (que agora **afirma** o `-0` em vez de esconder). Severidade baixa (perda de sinal de zero), mas o padrao "referencia de teste mais precisa que o produto" merece registro.
- **Confianca.** CONFIRMADA para a discrepancia; PROVAVEL para o efeito em jogos (nenhum caso de jogo e citado).

### D-09 - CSE block-local reaproveitava `dx.op.wave*`/`dx.op.quad*` atraves de `dx.op.discard`

- **Descricao / sintomas.** Duas chamadas identicas a `dx.op.waveActiveBallot` (ou `dx.op.quadReadLaneAt`) em lados opostos de um `dx.op.discard`, **no mesmo bloco basico**, eram fundidas em uma. `dx.op.discard` rebaixa lanes a helper lanes **sem terminar o bloco**, logo o conjunto de lanes ativas muda e o valor pre-discard nao vale depois. Efeito: ballot/quad-read com mascara errada em pixel shaders que descartam.
- **Causa raiz.** `cse_kind()` em `gpu/dxil/module/opt.rs` mantinha um mapa `local` por bloco para chamadas puras de wave/quad e nao o invalidava em `dx.op.discard`.
- **Introduzido por.** `223311f` ("dxil: fix VS/PS block order panics, translation time, output size, validator errors") — `git log -S"fn cse_kind"` retorna apenas esse commit.
- **Tentativas de correcao.** Nenhuma anterior.
- **Correcao definitiva.** `50228bb` (merge `36ac23d`, #1930): ao visitar `dx.op.discard`, `local.clear()`, com teste `block_cse_invalidates_wave_and_quad_calls_across_discard` que verifica 1 chamada sem discard e 2 com discard, para `waveActiveBallot` e `quadReadLaneAt`.
- **Arquivos.** `crates/xcl/src/gpu/dxil/module/opt.rs` (+66).
- **Evidencia.** `git show 50228bb`; `Select-String opt.rs -Pattern "dx.op.discard"` em HEAD (linha 471).
- **Estado atual.** Corrigido. Nota de projeto: a invalidacao e **grosseira** (limpa tudo no bloco), o que e seguro e custa apenas CSE perdido.
- **Confianca.** CONFIRMADA.

### D-10 - Panic de underflow em `s_setreg_b32` de `HW_REG_MODE` bit 3 (tradutor direto)

- **Descricao / sintomas.** `s_setreg_b32` escrevendo **so o bit 3** de `HW_REG_MODE` (`off == 3`, `dp_mask == 2`, ex. `SIMM16 = 0x00C1`) em shader com arredondamento f16 dinamico fazia o tradutor **panicar**: `attempt to subtract with overflow`. Em release, `2 - off` enrolaria para um shift gigante — resultado errado em vez de panic.
- **Causa raiz.** `let sh = 2 - off;` era calculado **antes** do `match dp_mask`, embora so os bracos `dp_mask == 3` e `dp_mask == 1` usem o valor, e neles `off <= 2`.
- **Introduzido por.** `746f7dd` (o port do tradutor direto; `git log -S"fn setreg"` lista `746f7dd` e `93072dc`, sendo `93072dc` o caminho HLSL).
- **Tentativas de correcao.** Nenhuma anterior.
- **Correcao definitiva.** `ecb7547`: `2 - off` movido para dentro dos dois bracos (`self.lshrc(src, 2 - off)`), mais teste de paridade com HLSL na GPU (`s_setreg_b32_mode_bit3_does_not_underflow`, com MODE[3:2] = 0b10 e valores `+1.0+2^-23` / `-1.0-2^-23` esperando `0x3C00` / `0xBC01`).
- **Arquivos.** `crates/xcl/src/gpu/dxil/lower.rs`, `crates/xcl/src/gpu/dxil/core_tests.rs`.
- **Evidencia.** `git show ecb7547`; mensagem: "`Lw::setreg` computed `let sh = 2 - off;` before matching `dp_mask`, panicking with integer underflow".
- **Estado atual.** Corrigido (`Select-String lower.rs -Pattern "2 - off"` mostra as duas ocorrencias dentro dos bracos).
- **Confianca.** CONFIRMADA.

### D-11 - Entradas de cache do per-lane nao incluiam as fontes do lowering: DXIL obsoleto servido do disco

- **Descricao / sintomas.** Um container traduzido pelo per-lane era gravado em disco com a identidade `xcl-per-lane` e **reutilizado depois de uma mudanca no proprio lowering**, servindo codigo gerado por uma versao anterior. Depois de uma correcao de semantica, o "conserto" nao aparecia.
- **Causa raiz.** `identity_for(&id, "xcl-per-lane")` nao derivava das fontes que produzem aquele container (o `XCL_TRANSLATOR_SOURCE_ID` cobria so o caminho HLSL).
- **Introduzido por.** `ddfbf39` ("the per-lane form in the runtime behind XCL_PER_LANE=1") — `git log -S'"xcl-per-lane"'` retorna apenas esse commit.
- **Tentativas de correcao.** Nenhuma anterior.
- **Correcao definitiva.** `bfb8133`: `build.rs` passa a hashear `PER_LANE_SOURCES` (`lane_masks.rs`, `lower.rs`, `lower/wave.rs`, `lower_dxil.rs`, `ssa.rs`) em `XCL_PER_LANE_SOURCE_ID`, e a identidade vira `concat!("xcl-per-lane-", env!("XCL_PER_LANE_SOURCE_ID"))`. As entradas do caminho HLSL mantem a identidade antiga (nao invalidadas).
- **Arquivos.** `crates/xcl/build.rs` (+19), `crates/xcl/src/gpu/recomp/pso.rs` (+1/-1).
- **Evidencia.** `git show bfb8133` (a constante `PER_LANE_SOURCES` e o `concat!`).
- **Estado atual.** Corrigido. Efeito colateral aceito: cada commit que toca essas 5 fontes invalida o cache per-lane (recompilacao), o que e exatamente o comportamento desejado.
- **Confianca.** CONFIRMADA.

### D-12 - A analise per-lane tratava `s_cbranch_vccz` como branch por lane e ignorava escritas de MODE

- **Descricao / sintomas.** Dois erros de semantica em programas que a forma per-lane **aceitava**:
  1. `s_cbranch_vccz`/`vccnz`, `s_cbranch_scc*` apos op de mascara, e leituras escalares do SCC de uma mascara (`s_cselect`, `s_cmov`, `s_addc`, `s_subb`) decidem **pela wave toda** no console; a analise tomava o bit de cada lane. Com VCC = lanes 0-15 e EXEC = 32 lanes, dava 7 onde o console da 0.
  2. `s_setreg` de MODE era **no-op**, entao um modo de arredondamento nao-padrao era descartado silenciosamente, onde o caminho HLSL recusa o programa.
- **Causa raiz.** A analise (`lane_masks.rs`) nasceu com a premissa "branch em mascara = branch por lane" e com `S_SETREG_*` listado entre as instrucoes ignoradas ("only float instructions read it, and the lowering refuses every float instruction so far — revisit when floats come in").
- **Introduzido por.** `1bab757` (analise) e `f5d3e05` (survey de cobertura, que introduziu `sets_mode_only` para aceitar mais programas — `git log -S"sets_mode_only"`).
- **Tentativas de correcao.** `66879ef` e a correcao; `9dcf2f6` (D-13) e `fe78c81` (D-14) tratam casos irmaos descobertos na mesma rodada.
- **Correcao definitiva.** `66879ef` ("the per-lane form refuses whole-wave mask decisions and modes it drops"): novo `uniform_over(reach, exec, cond)` (a condicao e a mesma em toda lane que chega com seu bit de EXEC) e recusa explicita — "a branch on whether any lane of a mask is set, with lanes in EXEC without its bit" e "a scalar read of the SCC of a mask op, with lanes in EXEC without its bit"; `S_SETREG_*`/MODE passa a ir pelo lowering do `#1426`, com `dyn_rm`/`has_f64_round_op` do adaptador. O teste `a_vccz_branch_is_a_lane_branch` e reescrito como `a_vccz_branch_is_a_lane_branch_only_where_vcc_is_the_exec`.
- **Arquivos.** `crates/xcl/src/gpu/recomp/lane_masks.rs` (+46), `lower.rs` (+35/-9), `lower_dxil.rs` (+24).
- **Evidencia.** `git show 66879ef`; a mensagem cita a revisao de Kalmarv em #1745 com programas na GPU contra o caminho HLSL.
- **Estado atual.** Corrigido por recusa: esses programas hoje caem para o HLSL.
- **Confianca.** CONFIRMADA.

### D-13 - SCC de uma operacao de mascara carregado em volta de um loop virava valor uniforme

- **Descricao / sintomas.** `s_and_b32 s0, vcc, exec` dentro do corpo de um loop define SCC; na iteracao seguinte um `s_cselect_b32` o le. A juncao na cabeca do loop contava so as arestas cujas mascaras usa, e **arestas de retorno nao contam ali**, entao o SCC saia uniforme e o `s_cselect` selecionava pelo bit de cada lane: **7** onde o console da **5** nas lanes 16-31.
- **Causa raiz.** `lane = edges.iter().any(|(_, s)| s.scc.is_some_and(|x| x.1))` considerava apenas as arestas usadas pela juncao.
- **Introduzido por.** `1bab757` (a analise per-lane).
- **Tentativas de correcao.** `66879ef` recusou as decisoes whole-wave, mas nao cobria o SCC vindo de aresta de retorno.
- **Correcao definitiva.** `9dcf2f6`: o flag "SCC e de lane" passa a vir de **toda** aresta de entrada, arestas de retorno incluidas, e um SCC desconhecido que e "de lane" em alguma aresta vira atomo (`Atom::Join(b, SCC)`), forcando a recusa pela regra whole-wave. Teste: o programa do thehollowedone e recusado (era aceito antes).
- **Arquivos.** `crates/xcl/src/gpu/recomp/lane_masks.rs` (+11/-1), `lower.rs` (+17).
- **Evidencia.** `git show 9dcf2f6`.
- **Estado atual.** Corrigido por recusa.
- **Confianca.** CONFIRMADA.

### D-14 - `v_cmpx` float via bridge descartava o EXEC anterior

- **Descricao / sintomas.** EXEC = lanes 0-15 (`s_and_saveexec`), depois `v_cmpx_le_f32 0, v1` (verdadeiro em toda lane), depois `v_mov v2, 7`. Console, caminho HLSL e a forma por instrucao escrevem lanes 0-15; a forma per-lane escrevia **as 32 lanes**.
- **Causa raiz.** O bridge roda o lowering do `#1426` com guarda `true` e le de volta o bit da lane — que e **so o resultado da comparacao**. O novo EXEC precisa ser `comparacao AND exec_anterior`.
- **Introduzido por.** `372be9d` ("s_cselect, s_wqm and v_cmpx in the per-lane form"), que habilitou `v_cmpx_*` no bridge.
- **Tentativas de correcao.** Nenhuma anterior.
- **Correcao definitiva.** `fe78c81`: le `old_exec` antes da instrucao quando `cmpx` e faz `bit = And(bit, old_exec)`; teste GPU `a_bridged_v_cmpx_keeps_the_lanes_outside_exec_out` (falha na lane 16 sem o fix).
- **Arquivos.** `crates/xcl/src/gpu/recomp/lower.rs` (+7), `lower_dxil.rs` (+29 de teste).
- **Evidencia.** `git show fe78c81`; `Select-String lower.rs -Pattern "let old_exec"` em HEAD (linha 1670).
- **Estado atual.** Corrigido.
- **Confianca.** CONFIRMADA.

### D-15 - `stale()`: valor copiado num bloco parcial mantinha o bloco de origem

- **Descricao / sintomas.** `stale()` chaveia cada valor escalar pelo bloco que o produziu. Uma escrita num bloco que **algumas lanes pulam** de um valor feito em outro lugar (`s_mov s4, s0` copiando uma mascara antiga, ou uma constante compartilhada pelo builder) mantinha o bloco antigo — logo uma leitura **depois** de as lanes se encontrarem nao era recusada, embora a escrita do console alcance todas as lanes e cada lane aqui guarde a sua.
- **Causa raiz.** `def_block.entry(v).or_insert(b)` registrava o primeiro bloco do **valor**, nao o bloco da **escrita**.
- **Introduzido por.** `b3c0b37` (o lowering kernel -> IR, que criou `stale`/`def_block`).
- **Tentativas de correcao.** Nenhuma anterior; foi "encontrado por MiguelMachado-dev enquanto escrevia a coalescencia de guardas" (`3827fb5`).
- **Correcao definitiva.** `a23b6ff`: nova funcao `own(b, v, bit)` que, quando o valor vem de outro bloco e o bloco atual e `partial`, cria uma copia propria (`or v, 0` / `or v, false`) definida no bloco parcial; aplicada em `write_scc`, `write_num` e `write_bit` (exceto EXEC).
- **Arquivos.** `crates/xcl/src/gpu/recomp/lower.rs` (+27/-5).
- **Evidencia.** `git show a23b6ff`; `Select-String lower.rs -Pattern "fn own\("` em HEAD (linha 370). Teste: um `s_mov s4, s0` no braco pulado e um `v_cndmask_b32` depois passam a ser recusados com "some lanes were elsewhere".
- **Estado atual.** Corrigido.
- **Confianca.** CONFIRMADA.

### D-16 - Mascara wave64 obsoleta reportada so na metade baixa

- **Descricao / sintomas.** Na saida do `lower_kernel`, para uma mascara cujas lanes guardaram bits proprios, so `r` era empurrado para `stale`. Em wave64, `r+1` vem da metade alta do **mesmo** ballot e carrega os mesmos bits obsoletos — nao reportado. Efeito no teste: VCC depois de um loop do qual lanes saem uma a uma dava `[106]` em vez de `[106, 107]`; na GPU (RX 7800 XT, wave64 nativo) o programa "leaving loop" divergia em `s107` (`0xFFFFFFFF` no lowering contra `0x80000000` na GPU, com so a lane 63 ativa no ultimo `v_cmp`).
- **Causa raiz.** `stale.push(r)` sem tratar o par de SGPRs do wave64.
- **Introduzido por.** `b3c0b37` (`git log -S"stale.push(r)"`).
- **Tentativas de correcao.** Nenhuma anterior.
- **Correcao definitiva.** `13ac196`: `if l.wave64 { stale.push(r + 1); }`, mais `a_wave64_mask_left_behind_is_reported_in_both_halves` (CPU) e `the_wave64_lowering_agrees_with_the_hlsl_path_on_the_gpu` (5 programas wave64 com mascaras `_B64` e limiares acima de 32). Ambos falham sem o fix.
- **Arquivos.** `crates/xcl/src/gpu/recomp/lower.rs` (+103, quase todo teste).
- **Evidencia.** `git show 13ac196`; `Select-String lower.rs -Pattern "stale.push\(r \+ 1\)"` em HEAD (linha 1507).
- **Estado atual.** Corrigido. Observacao: nao ha guarda contra `r == 127` (seria `128`); o efeito seria apenas uma entrada extra em `stale`, sem consequencia pratica.
- **Confianca.** CONFIRMADA.

### D-17 - Mascaras inline/NULL recusadas no bridge do per-lane

- **Descricao / sintomas.** `V_CNDMASK_B32`, `V_ADD_CO_CI_U32`, `V_SUB_CO_CI_U32`, `V_SUBREV_CO_CI_U32` eram recusados sempre que a mascara nao fosse um SGPR (`m > 127`: carry-in inline `0` = 128, `-1` = 193, `NULL` = 125), embora `dxil::Lw::read64` ja decodificasse esses operandos. Perda de cobertura: programas legitimos caiam para o HLSL.
- **Causa raiz.** A checagem `if m > 127 { return Err(format!("{name}: mask operand {m}")) }` no bridge; e `src_bit()` aceitava apenas SGPR, 128 e 193 (recusando NULL, inteiros inline `129..=192`, `194..=208` e literais de 32 bits `255`); `src()`/`src64()` nao tratavam `125`.
- **Introduzido por.** `878afda` (o bridge).
- **Tentativas de correcao.** Nenhuma anterior.
- **Correcao definitiva.** `15fff3b` (#1745): `bridge()` nao injeta SGPR quando `m > 127 || m == 125`; `src_bit()` extrai o bit da lane de NULL, de inteiros inline (sign-extended a 64 bits para negativos) e de literais; `src()`/`src64()` tratam 125.
- **Arquivos.** `crates/xcl/src/gpu/recomp/lower.rs`.
- **Evidencia.** `git show 15fff3b`; `git log -S'mask operand {m}'` retorna `878afda` e `15fff3b`.
- **Estado atual.** Corrigido (2026-11-25, o ultimo dia).
- **Confianca.** CONFIRMADA.

### D-18 - `s_wqm` nao era seguido: todo PS mantinha guarda em toda instrucao vetorial

- **Descricao / sintomas.** Quase todo pixel shader comeca com `s_wqm exec, exec`. A analise nao seguia `s_wqm`; como o EXEC de lancamento de um PS passou a ser "desconhecido" (`d2cc6ff`), o EXEC depois do `s_wqm` tambem era uma mascara desconhecida e **toda instrucao vetorial do programa mantinha a guarda** — mais blocos, mais `select`, mais custo.
- **Causa raiz.** `mask_src`/`step` nao tinham regra para `WQM`, e faltava a observacao de que "toda lane que roda um PS ou esta no EXEC de lancamento ou e helper de um quad com lane nele" — logo `s_wqm` do EXEC de lancamento e **toda lane**.
- **Introduzido por.** `d2cc6ff` ("the lane-mask analysis can start from an unknown EXEC"), que criou `LaunchExec::Unknown` para PS sem a regra correspondente.
- **Tentativas de correcao.** `27d28ce` e a correcao.
- **Correcao definitiva.** `27d28ce`: `fn wqm(a)` segue `s_wqm` quando o resultado e conhecido (nenhuma lane, toda lane, ou o proprio EXEC de lancamento via `Atom::Launch`); teste `whole_quad_mode_of_the_launch_exec_is_every_lane`. `a24899e` completa com `Summary::wqm_all` e recusa `s_wqm` de mascaras arbitrarias.
- **Arquivos.** `crates/xcl/src/gpu/recomp/lane_masks.rs` (+32; +12 em `a24899e`).
- **Evidencia.** `git show 27d28ce`; mensagem: "EXEC after it was an unknown mask too and every vector instruction of the program kept its guard".
- **Estado atual.** Corrigido para o caso comum; `s_wqm` de outra mascara continua recusado.
- **Confianca.** CONFIRMADA (o ganho e descrito, nao medido em jogo).

### D-19 - Geometry na forma split: a fase ES caia dentro do codigo do GS e os records saiam vazios

- **Descricao / sintomas.** Na forma split wave64 do mode geometry (ES e GS como duas fases do mesmo corpo, `pc` comecando em 0 para o ES e em `gs_start` para o GS), um trecho estruturado podia atravessar o `s_setpc` do ES (que nao e um `term()`): com **qualquer** branch no programa, a fase ES executava tambem o codigo do GS e a fase GS nao encontrava o `if (pc == gs_start)`, rodando nada — records vazios. Afetava apenas GPUs so-wave32.
- **Causa raiz.** `structured_span` nao tratava o `s_setpc` do ES como fronteira na forma split.
- **Introduzido por.** `a21d3df` (2026-11-20, "Session 5") criou `gs_start`; o defeito se materializa quando a forma estruturada (`8c6f1c3`, 2026-11-21) passou a poder cruzar esse ponto. Atribuicao precisa do instante de introducao: NAO VERIFICADA.
- **Tentativas de correcao.** O defeito foi **escondido** por um teste: `geometry_tests.rs` excluia a comparacao com a forma split do caminho HLSL, com o comentario explicando exatamente o sintoma. Nao havia correcao.
- **Correcao definitiva.** `f3242b2` (#1689, commit `87d421b`): `structured_span` para no `s_setpc` na forma split; a forma nativa wave64 fica intacta. Testes: o teste de tira ES/GS **com branches** na GPU e um teste de forma do HLSL para entrada/saida das fases. `2a2d54b` entao remove a exclusao do teste (`if !split` -> comparacao sempre) e atualiza o comentario: "fixed by #1689".
- **Arquivos.** `crates/xcl/src/gpu/recomp/mod.rs` (+11), `recomp/tests.rs` (+61), `gpu/dxil/geometry_tests.rs`.
- **Evidencia.** `git show 87d421b`; `git show 2a2d54b` (o diff mostra a remocao de "The HLSL path's *split* form is not compared ... empty records").
- **Estado atual.** Corrigido.
- **Confianca.** CONFIRMADA (fix e teste); PROVAVEL quanto ao commit exato de introducao.

### D-20 - ExecuteIndirect lia argumentos indiretos antes da barreira que os escreve (NVIDIA)

- **Descricao / sintomas.** O console ordena um dispatch que **escreve** argumentos indiretos antes do `ExecuteIndirect` que os **le** com uma barreira que chega ao PC como UAV barrier. O driver da NVIDIA (RTX 5090) buscava os argumentos **sem esperar** por ela: o `ExecuteIndirect` rodava a contagem que a memoria tinha antes da lista — inclusive quando o dispatch escrevia pela janela de memoria (como fazem os shaders traduzidos). No teste: 40 de 40 tentativas rodaram a contagem velha.
- **Causa raiz.** Dependencia real de dados expressa apenas por UAV barrier, que o driver nao honra para `ExecuteIndirect`.
- **Introduzido por.** NAO VERIFICADO (a sequencia de listas do console com escrita de argumentos indiretos e comportamento do titulo; nao ha commit unico identificavel).
- **Tentativas de correcao.** Nenhuma anterior.
- **Correcao definitiva.** `4e3515e` (#1791, commit do PR sem corpo): cada `ExecuteIndirect` passa a ser registrado **entre duas transicoes de um buffer proprio da XCL** para `INDIRECT_ARGUMENT` ("A transition to INDIRECT_ARGUMENT makes the driver wait - of any buffer, as measured"). So listas criadas pela XCL (`list_x::on_create`) e nao bundles recebem as transicoes; o tipo vem de `list_x::tracked_list_type`, sem chamar a lista. Teste `execute_indirect_reads_the_arguments_written_before_it`.
- **Arquivos.** `crates/xcl/src/hle/d3d12x/bind.rs` (+191), `list_x.rs` (+8/-3).
- **Evidencia.** `git show --stat 4e3515e`; a mensagem do branch commit (recuperada em `git log c4d37be^1..4e3515e^2 --format=%B`).
- **Estado atual.** Corrigido; nenhum commit posterior toca `bind.rs` depois de `4e3515e` (`git log 4e3515e..HEAD -- .../bind.rs` vazio), sinal de que nao houve regressao conhecida.
- **Confianca.** CONFIRMADA (causa e correcao documentadas com medicao); o commit de introducao: NAO VERIFICADO.

### D-21 - `ExecuteIndirect` de um pipeline de compute nao traduzido rodava com a root signature errada

- **Descricao / sintomas.** `Dispatch` pula um pipeline de compute do console **sem traducao** (build que falhou). `ExecuteIndirect` enviava o mesmo pipeline para a chamada do PC **depois de setar apenas a graphics root signature** — o dispatch entao rodava com a root signature de compute da lista, que podia ainda ser a ABI da XCL de um dispatch traduzido anterior. O debug layer do D3D12 reporta esse par como **erro 953** durante o world load de um titulo retail.
- **Causa raiz.** Falta de rastreio de (a) assinaturas de comando com argumento `Dispatch` e (b) pipelines de compute feitos de shader do console que roda sem traducao.
- **Introduzido por.** NAO VERIFICADO (o caminho de `ExecuteIndirect` com compute existe desde o inicio do D3D12.X).
- **Tentativas de correcao.** `2d9a9fe` ("ExecuteIndirect with a pipeline still being built is skipped") tratou o caso "ainda construindo"; nao este.
- **Correcao definitiva.** `6c89285` (#1195): assinaturas com `Dispatch` sao anotadas quando criadas (e esquecidas quando outra assinatura sem `Dispatch` e criada no mesmo endereco); pipelines de compute feitos de shader que roda sem traducao sao anotados e descartados com o pipeline; `ExecuteIndirect` pula assinatura `Dispatch` **apenas** para esses, deixando passar pipeline nativo de compute do PC.
- **Arquivos.** `hle/d3d12x/bind.rs` (+63), `device_x.rs` (+3), `real.rs` (+30).
- **Evidencia.** `git show 6c89285`; mensagem: "The D3D12 debug layer reports that pair as error 953 during a retail title's world load".
- **Estado atual.** Corrigido.
- **Confianca.** CONFIRMADA.

### D-22 - `xcl shaders`: `--export-hlsl` pulava registros nao-compute em silencio; flags sem valor eram aceitas

- **Descricao / sintomas.** Achados de revisao em `--export-hlsl` / `--measure-pso`: (a) registros nao-compute eram pulados **sem contagem nem aviso** (`continue` antes de `matched = true`); (b) `--export-hlsl`, `--record-hash`, `--measure-pso` sem valor (ou com outro flag no lugar) eram aceitos silenciosamente; (c) `--measure-pso` nao liberava a root signature em todos os caminhos e nao desligava caches do driver; (d) um arquivo unico de shader era resumido e retornava, em vez de ser entrada; (e) `--export-hlsl`/`--measure-pso` nao usavam heap windows (`XCL_HEAP_WINDOWS=1`) como `--precompile`.
- **Causa raiz.** Ferramenta de diagnostico escrita incrementalmente: `value(name)` devolvia `Option<&String>` sem validar o proximo argumento, e o filtro de compute estava no lugar errado.
- **Introduzido por.** NAO VERIFICADO (multiplos commits de `shaders.rs`).
- **Tentativas de correcao.** Nenhuma anterior.
- **Correcao definitiva.** `4143a09` ("fix --export-hlsl / --measure-pso review findings"): `required(name)` que erra se faltar valor (codigo de saida 2); `runtime_windows()` unico; arquivo unico vira `Input::Files`; registro requisitado por hash que falha **para o scan**; nao-compute e pulado em silencio **a menos** que um hash tenha sido pedido.
- **Arquivos.** `crates/xcl/src/shaders.rs` (+125/-64).
- **Evidencia.** `git show 4143a09`.
- **Estado atual.** Corrigido.
- **Confianca.** CONFIRMADA.

### D-23 - Testes GPU falhavam (em vez de pular) em adaptadores sem waves de 32 lanes; e o teste de IR falhava sem `dxil.dll`

- **Descricao / sintomas.** No CI o adaptador e WARP, que nao tem waves de 32 **nem** de 64 lanes: todo teste per-lane falhava em `CreateComputePipelineState` com `0x80070057` em vez de pular. Em paralelo, `ir_text_assembles_signs_and_runs` falhava com "no dxil.dll to sign with" — o `bin\<ver>\x64` de um Windows SDK tem `dxcompiler.dll` **sem** `dxil.dll`.
- **Causa raiz.** As guardas usavam `runner::adapter().is_err()` (ha GPU) em vez de "ha GPU com waves de 32 lanes"; e o caminho de erro de `assemble` nao distinguia "sem assinador" de "IR invalido".
- **Introduzido por.** Serie per-lane (`b3c0b37` e seguintes) e `a8569bf` (assemble).
- **Tentativas de correcao.** `9e27370` e `6bdef80` sao as correcoes; `b1bfd1e`/`9e27370` tambem revelam que o adaptador do CI nunca roda esses testes de verdade.
- **Correcao definitiva.** `9e27370`: `gpu_waves()` = `runner::adapter().is_ok() && runner::wave32_supported() == Ok(true)`, usado por 7 testes; mensagem registra `XCL_ADAPTER=warp`: 876 passed, 0 failed, 22 ignored. `6bdef80`: `Err(e) if e.contains("no dxil.dll")` -> `skip_test("no dxil.dll")`.
- **Arquivos.** `gpu/recomp/lower.rs`, `lower_dxil.rs`, `recomp/dxil/assemble.rs`, `shaders.rs` (tamanhos em KB).
- **Evidencia.** `git show 9e27370`, `git show 6bdef80`; `git log -S"fn gpu_waves"` retorna `9e27370`.
- **Estado atual.** Corrigido. Risco residual: com WARP, os testes per-lane **nunca** rodam no CI — a cobertura GPU real depende de contribuidores (as mensagens citam GTX 1650, RTX 3080 Ti/4090/5090, RX 7800/7900 XT, RX 9060/9070 XT).
- **Confianca.** CONFIRMADA.

### D-24 - Ordem `dx.op.getDimensions` -> `dx.op.textureLoad` em Texture2DMS derrubava o compilador da AMD

- **Descricao / sintomas.** Em texture MSAA indexada dinamicamente por heap (`Texture2DMS`/`Texture2DMSArray`), se `dx.op.getDimensions` precedesse `dx.op.textureLoad` **no mesmo bloco basico**, o `amdxc64.dll` da AMD dereferenciava nulo: `0xc0000005`.
- **Causa raiz.** Bug do compilador da AMD sensivel a ordem das duas chamadas (o valor de `getDimensions` e usado para comparar a amostra com a contagem).
- **Introduzido por.** `2dcc926` ("dxil: Texture2DMS / Texture2DMSArray ops, MSAA image loads in the direct translator").
- **Tentativas de correcao.** Nenhuma anterior.
- **Correcao definitiva.** `425b869` (merge `20cf130`, #1900): todas as chamadas `texture_load_sample` sao emitidas **antes** de `getDimensions` (coletadas num `Vec` e consumidas depois). Teste `direct_msaa_loads_precede_get_dimensions` que **desmonta o DXIL** e verifica a ordem por bloco, exigindo >= 15 blocos com load.
- **Arquivos.** `crates/xcl/src/gpu/dxil/lower_image.rs` (+8/-4), `msaa_tests.rs` (+50).
- **Evidencia.** `git show 425b869`; `Select-String lower_image.rs -Pattern "amdxc64"` em HEAD (linha 857).
- **Estado atual.** Corrigido como workaround documentado. Os loads sao emitidos mesmo quando `sv >= count` (mais trabalho, sem efeito observavel).
- **Confianca.** CONFIRMADA.

### Defeitos examinados e considerados NAO-defeito ou ruido

- `1d027e5` + `e9ac9f0` (workflow `stale.yml`): politica de CI, nao defeito. `e9ac9f0` e merge de `origin/main`, sem conteudo proprio.
- `36ac23d`, `20cf130`, `c4178e8`, `4e3515e`, `6c89285`, `f3242b2` (merges sem texto de diff no pacote): o conteudo foi lido no primeiro pai do branch, nao no merge.
- `27751a0`, `82c875d`, `3aea7a9`, `9e27370`, `6bdef80`, `2a2d54b`, `b1bfd1e`: apenas testes/documentacao — exceto onde revelaram defeito, registrado acima.
- `bc0d77c`, `77db49c`, `ad39e4d`, `f90033c`, `45025eb`, `acdf922`, `f33f640`, `d2cc6ff`, `a24899e`, `14e367e`, `372be9d`, `3827fb5`, `1b...`: funcionalidade nova; os defeitos que carregavam estao catalogados (D-05, D-12, D-13, D-14, D-18, D-17, D-04, D-11).
- `c4d37be` (#1747, freeze de 180 s): correcao de desempenho, sem defeito subsequente. `git log c4d37be..HEAD -- .../vram.rs` e vazio. O teste adicionado mede tempo mas **afirma** sobre contagem de chamadas (`assert!(new_calls < old_calls)`), nao sobre wall-clock — nao e um teste instavel.
- `ecb7547`: defeito real (D-10).

## E. Relacao de causa e efeito

1. **Janelas de memoria (o eixo mais longo do repositorio).**
   `f3b15b9` (UMA em ate 4 janelas de 4 GiB) introduz `window(hi) = hi & 3u` -> `a787c52` corrige o carry somando em 64 bits -> `778e329` (#123) mergeia -> o codegen de 64 bits derruba `amdxc64` em XDefiant (`0xc0000005`, write em `0x2000000c`) -> `70f8dc4` reverte 93 linhas, incluindo o teste `gpu_memory_window_carry` -> **o defeito de D-01 volta e fica ~12 h** -> #138 tenta carry em 32 bits (nao mergeado) -> `fe7d70b`/`927adcc` (#187) removem o `switch` por acesso via descriptor heap -> `f9e5ee9` (#792) calcula 64 bits antes da janela, separa `window_read`/`window_write`, adiciona range-check contra a base da UMA e remove `fn window` -> cobertura volta como `memory_windows_reject_out_of_uma_aliasing`.

2. **Forma estruturada.**
   `8c6f1c3` (controle estruturado) -> `b8b1f49` estrutura saltos para loops externos, criando programas com milhares de flags pendentes -> guardas quadráticas -> 148 MB de HLSL e OOM do DXC -> `69a892c` corta em 128 B por byte de programa e cai para `pc` blocks.

3. **Vertical do backend per-lane (5 defeitos em 6 h).**
   `1bab757` (analise) + `b3c0b37` (lowering) -> premissas erradas sobre `vccz` e `op_sel` (D-05, D-12) -> `82c875d`/`27751a0`/`3aea7a9` (programas de teste) revelam D-04, D-06, D-07 -> `ddfbf39` liga no runtime (15:51) **antes** de `188ee17` corrigir D-04 (16:21) -> review de Kalmarv/gefydev fecha D-12, D-13, D-14 em tres rodadas (`66879ef`, `9dcf2f6`, `fe78c81`).

4. **Reuso do que foi gerado.**
   `ddfbf39` (identidade `xcl-per-lane` sem as fontes) -> `bfb8133` (hash das 5 fontes em `XCL_PER_LANE_SOURCE_ID`). O mesmo padrao ja existia para o caminho HLSL (`XCL_TRANSLATOR_SOURCE_ID`, em `build.rs`), o que mostra que a licao era conhecida e simplesmente nao foi aplicada na primeira versao.

5. **Assincronia de lista de comando.**
   `2d9a9fe` (pipeline ainda em construcao) -> `6c89285` (pipeline sem traducao) -> `4e3515e` (argumentos indiretos lidos antes da barreira). Tres defeitos distintos na mesma funcao (`ExecuteIndirect`), cada um descoberto por um titulo/Driver diferente.

6. **Erros de driver que viram constraint de codegen.**
   `amdxc64`: ordem `getDimensions`/`textureLoad` (`425b869`), aritmetica de 64 bits por acesso (`70f8dc4`), reordenacao de comparações float apos `!(a>=b)` (`emit.rs:136-139`), `no-signed-zeros` (`b1bfd1e`). Nenhum desses e bug da XCL no sentido logico; todos sao restricoes reais do alvo.

## F. Correcoes importantes

**F-1. Endereco efetivo de 64 bits antes da escolha da janela (`f9e5ee9`).**
O que estava errado: a janela era escolhida pelo dword alto da base, e o offset somava em 32 bits. Por que importa: uma janela de 4 GiB e exatamente o tamanho de um espaco de 32 bits, entao **qualquer** offset nao trivial pode cruzar a fronteira. Como foi corrigido: `uint64_t {t}_ea = <endereco completo>`, depois `window_read((uint)(ea >> 32))` ou `window_write(...)`; a base da UMA entra por `IO_UMA_BASE_HI` (io slot 112) e o range-check e `hi - xcl_uma_base` com os accessors usando `default` para "fora da UMA" (leitura 0, escrita/atômico descartado). Por que funciona: elimina a aritmetica de 32 bits no ponto errado e, de bonus, impede que um ponteiro fora da UMA **apelide** uma janela viva. Dependencias: `mem_accessors` precisa de um `default` que nao seja o `mem` da janela 0; `window_read` e `window_write` diferem de proposito (`hi == 0` e tabela de descritores de 32 bits do console: le da janela 0, mas **nao** escreve). Reutilizavel em qualquer camada de traducao de GPU com janelas: regra geral = "nunca derive o seletor de regiao de um pedaco do endereco antes de completar o endereco".

**F-2. Reverter e uma decisao de engenharia, nao um fracasso (`70f8dc4`).**
O que estava errado: um crash deterministico dentro de um componente de terceiro (o compilador da AMD), disparado por uma mudanca correta em nivel de linguagem. Como foi corrigido: revert mecanico (`git revert -m 1`) isolando **um** PR, com o motivo registrado no PR original e em `HANDOFF.md`. Por que funciona: restaura o estado que rodava, sem tocar no que veio junto no lote (o merge de #124, onze minutos depois, fica). Licao reutilizavel: quando dois PRs entram no mesmo minuto e um quebra, o revert por merge (nao por commit) e o unico jeito de separa-los; e `git revert -m 1 <merge>` gera exatamente esse diff (13+/93-).

**F-3. Fallback como instrumento de correcao (`fe8dc84`, `878afda`, `ac34cb6`, `66879ef`).**
Em vez de "consertar" um lowering incompleto, o bridge do per-lane mantem a lista `DIFFERS_ON_1426` e recusa aquelas instrucoes; o programa cai para o caminho HLSL, que e correto. Isso e uma correcao **de corretude** com custo de cobertura, e e o padrao dominante do repositorio (`Translation::complete()` / `unsupported_why`). Reutilizavel: em tradutor com dois backends, uma lista de divergencias conhecidas por opcode, com o motivo e o link do bug, e mais segura do que tentar igualar semantica sob pressao.

**F-4. Recusa por "uniformidade sobre EXEC" (`66879ef`, `uniform_over`).**
O que estava errado: a analise respondia "o bit desta lane?" quando a pergunta do console era "alguma lane da wave?". Como foi corrigido: uma pergunta BDD — a condicao e constante em toda lane que chega com seu bit de EXEC (`implies(reach & exec, c)` ou `implies(reach & exec, !c)`)? Se nao, recusa. Por que funciona: cobre os dois casos legitimos (`s_and_saveexec` + `s_cbranch_execz`, VCC == EXEC) sem enumerar padroes de codigo. Reutilizavel em qualquer lowering que troque semantica wave-uniforme por per-lane.

**F-5. "Todo valor escrito num bloco que algumas lanes pulam precisa de definicao propria" (`a23b6ff`).**
O que estava errado: o mapa `def_block` (usado para detectar leitura de valor obsoleto) registrava o bloco do **valor**, nao o da **escrita**; uma copia (`s_mov s4, s0`) num bloco parcial herdava o bloco antigo e a leitura posterior deixava de ser recusada. Como foi corrigido: `own()` cria `or v, 0` no bloco parcial para materializar a escrita. Por que funciona: torna o valor "deste bloco" sem mudar o resultado numerico. Reutilizavel: e o mesmo problema de "SSA com caminhos divergentes" que aparece em qualquer lowering por lane.

**F-6. Identidade de cache derivada das fontes do produtor (`bfb8133`).**
`build.rs` hasheia `PER_LANE_SOURCES` em `XCL_PER_LANE_SOURCE_ID`; a identidade vira `xcl-per-lane-<hash>`. Separar a identidade por caminho (`TRANSLATOR_SOURCES` vs `PER_LANE_SOURCES`) evita invalidar o cache do HLSL quando so o per-lane muda. Reutilizavel literalmente: `cargo:rerun-if-changed` + hash FNV do conteudo + `concat!` na chave.

**F-7. Isolar o driver: transicao de `INDIRECT_ARGUMENT` como barreira portavel (`4e3515e`).**
Quando uma dependencia de dados so existe via UAV barrier e um driver a ignora, usar uma transicao de estado que o driver **precisa** respeitar, num recurso proprio da camada (nao do titulo), e um padrao robusto e barato (duas transicoes por `ExecuteIndirect`). Reutilizavel em qualquer camada D3D12 que sintetize listas.

**F-8. Testes que afirmam a **forma** do codigo gerado (`425b869`, `0b64aa0`, `2a2d54b`).**
`direct_msaa_loads_precede_get_dimensions` desmonta o DXIL e verifica ordem por bloco; `multi_window_mem_format_and_bvh_validate` valida o DXIL para 1..=4 janelas; `gpu_geometry_direct_records` passou a comparar a forma split do HLSL. Isso protege contra regressoes de **forma** que nenhum teste de resultado capturaria (e que, no caso da AMD, sao a diferenca entre rodar e crashar).

## G. Problemas ainda existentes

- **G-01. f16 no tradutor direto (D-06, D-07).** `DIFFERS_ON_1426` ainda tem `V_CVT_F16_F32`, `V_MUL_F16`, `V_PK_MUL_F16`. Todo programa per-lane com multiplicacao f16 cai para o HLSL. O comentario aponta uma correcao "no `main`" por `itsSio` — fora deste clone: NAO VERIFICADO.
- **G-02. `-0` no caminho HLSL em driver AMD (D-08).** O runtime compila sem `-Gis`; so os testes de referencia usam. Nenhum commit posterior mexe nas flags de `dxc.rs`.
- **G-03. A forma de 64 bits por acesso voltou em `f9e5ee9` e o CS do XDefiant nao foi re-testado.** `f9e5ee9` reintroduz `uint64_t {t}_ea` em `s_buffer_load`, `s_load`, MUBUF (3 pontos) e FLAT (`emit_flat_addr`), exatamente o padrao que o mantenedor culpou pelo crash do `amdxc64` em #123. A justificativa implicita e que o caminho padrao virou descriptor heap (#187) e o `& 3u` virou range-check. Os relatos de teste de #792 sao em **RDNA4** (RX 9060/9070 XT) com smoke do titulo retail; nenhum menciona XDefiant. Risco: um crash de driver que so aparece em RDNA2/RDNA3 com a forma switch. Classificacao: **HIPOTESE**.
- **G-04. Backends novos sao opt-in e incompletos.** `XCL_DIRECT_DXIL`/`XCL_PER_LANE` nao sao o padrao; a cobertura e o limitador (a primeira varredura do per-lane falava de 250 de 2 689 programas de compute; as mensagens de `66879ef`/`c6b...`/`15fff3b` sao todas sobre recusas). Nao ha, neste clone, um numero final de cobertura.
- **G-05. `hi == 0` le da janela 0 mas nao escreve.** Decisao deliberada de `f9e5ee9` ("32-bit console descriptor table pointers"), testada. Se um titulo escrever legitimamente por um ponteiro de 32 bits com dword alto 0, a escrita e **silenciosamente descartada**. Estado: por desenho; risco para titulos nao verificados. **PROVAVEL** como risco, **CONFIRMADA** como comportamento.
- **G-06. Valores escalares obsoletos em bloco parcial continuam recusados.** `ac34cb6`: "Taking the last-leaving lane's value is left for later". Programas com waterfall loop caem para o HLSL.
- **G-07. `s_wqm`/mascaras arbitrarias.** So o `s_wqm` do EXEC de lancamento (e de none/all) e seguido; o resto recusa.
- **G-08. Sem cobertura GPU no CI.** `gpu_waves()` faz os testes per-lane pularem em WARP; a validacao depende de hardware de contribuidores e de `scripts/smoke.ps1` em titulos. Ver D-23.
- **G-09. Teste de regressao perdido no revert.** `gpu_memory_window_carry` (55 linhas) nao voltou; a cobertura equivalente existe sob outro nome, mas nao ha teste que amarre especificamente **todos** os 6 tipos de acesso originais com carry. Classificacao: **PROVAVEL** lacuna de cobertura (o teste atual cobre s_load, buffer load/store, global store/atomic e carry/borrow, nao o `s_buffer_load` com `stride` e o dword16).
- **G-10. `then_some` com aritmetica ainda existe.** `upper_lanes.rs:322` faz `(i + n < 16).then_some(lane + n)` — o argumento e avaliado sempre. Nao ha evidencia de overflow (os operandos vem de campos pequenos), mas e a mesma classe do bug de `dd566c0`. **HIPOTESE** de risco.

## H. Regressoes

- **R-01 (`a787c52` -> `70f8dc4`): correcao que quebra um jogo.** A correcao de D-01 introduziu um crash deterministico do compilador de shader da AMD em XDefiant. E a **unica** regressao que exigiu revert no periodo (`git log --all --grep="Revert"` retorna so `70f8dc4`). Detalhe relevante: o merge de #123 e o de #124 aconteceram no mesmo minuto, e o revert precisou ser por merge para nao levar #124 junto.
- **R-02 (`b8b1f49`): otimizacao que virou OOM.** Estruturar saltos para loops externos reduziu 1 670 -> 170 programas em `pc` blocks (bom) e fez **um** programa gerar 148 MB de HLSL (ruim). Corrigido por limite heuristico (`69a892c`), nao por repensar a forma estruturada.
- **R-03 (`ddfbf39`): ligar no runtime antes da correcao.** O per-lane entrou no runtime as 15:51 e a correcao de D-04 (carry-ins lendo 0) veio as 16:21; entre os dois, `XCL_PER_LANE=1` produzia resultados errados em programas com carry. Janela de ~30 min, opt-in, sem release no meio.
- **R-04 (`d2cc6ff` -> `27d28ce`): generalizacao que degradou o caso comum.** Suportar PS (EXEC de lancamento desconhecido) fez **todo** PS manter guardas em toda instrucao vetorial ate `s_wqm` ser seguido. Nao e corrupcao, e regressao de qualidade de codigo gerado, corrigida no mesmo dia.
- **R-05 (`e9ac9f0`/`1d027e5`): nao e regressao de codigo, mas e um risco operacional.** O workflow `stale.yml` fecha PRs apos 7 dias sem atividade com `operations-per-run: 300` e `sort-by: updated`; com ~1 300 PRs abertos (o proprio commit diz "Nearly 1000 PRs are open"), os primeiros a serem tocados sao os mais antigos. Nao ha evidencia de PR fechado indevidamente neste clone. **NAO VERIFICADO.**

## I. Conhecimento reutilizavel

**I-1. Seletor de regiao a partir de endereco completo (D-01 / F-1).**
Problema: janelas de 4 GiB + offsets de 32 bits. Implementacao: `uint64_t ea = base64 + (uint64_t)off;` e depois `w = ea >> 32` menos a base; leitura e escrita com regras diferentes; `default` que nao apelida. Origem: XCL `f9e5ee9`. Arquivos: `gpu/recomp/emit.rs` (`window_read`, `window_write`, `emit_flat_addr`), `gpu/recomp/runner.rs`, `hle/d3d12x/bind.rs` (slot 112). Dependencias: um slot de I/O para a base da UMA. Conflitos: com a forma switch (cada acesso vira um `switch`), que foi o gatilho do bug do driver. Riscos: a aritmetica de 64 bits por acesso e a forma acusada no crash de #123 — em alvo AMD, preferir descriptor heap (uma comparacao) ou carry em 32 bits. Recomendacao: adotar, com teste de carry **e** de borrow, e um teste que verifique que um `hi` fora da faixa nao apelida janela viva.

**I-2. Guarda por nome de campo: falhe alto (D-05).**
Problema: `["...","OPSEL"]` nunca dispara porque `field_value` devolve `None` e `unwrap_or(0)` engole. Implementacao recomendada: um modo de teste em que `field_value(nome_desconhecido)` e erro, ou uma lista de campos validada contra a tabela de encoding. Origem: XCL `b3c0b37`/`ac34cb6`. Riscos: nenhum. Recomendacao: obrigatorio em decodificadores tabelados.

**I-3. Identidade de cache = hash das fontes do produtor (D-11 / F-6).**
Origem: `crates/xcl/build.rs` (`TRANSLATOR_SOURCES`, `PER_LANE_SOURCES`, FNV-64, `cargo:rerun-if-changed`). Reutilizavel em qualquer pipeline com cache em disco de artefato gerado. Recomendacao: uma lista **por caminho de traducao**, nunca uma global.

**I-4. Lista de divergencias conhecidas entre backends (F-3).**
Origem: `DIFFERS_ON_1426` em `gpu/recomp/lower.rs`. Estrutura: `const [&str]` + comentario com sintoma e autor do relato. Recomendacao: manter a lista **com data e issue**, e um teste que falhe quando o item for corrigido no outro backend (senao ela apodrece).

**I-5. Recusa por uniformidade sobre EXEC (D-12 / F-4).**
Origem: `lane_masks.rs::uniform_over`. Aplicavel a qualquer lowering que troque semantica wave-uniforme por per-lane (ballot, `readlane`, `mbcnt`, decisoes "any lane").

**I-6. Diagnostico de driver: quatro padroes AMD ja mapeados.**
(a) `getDimensions` antes de `textureLoad` em MSAA indexado por heap -> null deref (`425b869`); (b) aritmetica de 64 bits antes de um `switch` de janela -> null deref no compilador (`70f8dc4`); (c) reorganizacao de comparacoes float apos `!(a>=b)` -> resultado unordered errado (`emit.rs:136`); (d) `no-signed-zeros` implicito -> `+0` no lugar de `-0` (`b1bfd1e`). Uso: lista de "coisas que o alvo faz e nao estao na spec", cada uma com um teste que verifica a **forma** do codigo gerado, nao so o resultado.

**I-7. Emular barreira de dados com transicao de estado (`4e3515e` / F-7).**
Origem: `hle/d3d12x/bind.rs` (duas transicoes `INDIRECT_ARGUMENT` num buffer proprio em volta de cada `ExecuteIndirect`). Aplicavel a qualquer backend onde o driver comprovadamente ignora UAV barriers para uma operacao especifica.

**I-8. Testes de GPU que pulam, com razao legivel (D-23).**
Padrao: `fn gpu_waves() -> bool` + `runner::skip_test("no GPU with 32-lane waves")` + contagem no fim de `scripts/test.ps1` + `XCL_REQUIRE_GPU=wave64|mesh` para transformar skip em falha quando se quer cobertura de verdade (`ff9430e`). Recomendacao: nunca `#[ignore]`, sempre skip explicito — e **sempre** um modo que falhe.

**I-9. `then(|| ...)` em vez de `then_some(...)` quando o valor pode panicar (D-16/`dd566c0`).**
`then_some((r - 1, true))` avalia `r - 1` mesmo quando a condicao e falsa (underflow em `s0`). Bug de uma linha, panic em debug, comportamento errado em release.

**I-10. Limite heuristico para codegen patologico (`69a892c`).**
`STRUCTURED_BYTES_PER_CODE_BYTE = 128` com re-traducao em outra forma. Padrao geral: medir o tamanho da saida contra o da entrada e trocar de estrategia; mais robusto do que prever quais programas estouram.

## J. Conclusao tecnica

O XCL e um projeto de 6 dias com 853 commits, 274 merges e 311 arquivos — um ritmo que so se sustenta com revisao automatizada e testes em hardware de terceiros, e o historico mostra exatamente isso: quase todo commit de correcao cita **quem** achou o problema, em **que** GPU, com **que** numero. A densidade de defeitos reais no periodo analisado e alta mas a **qualidade do diagnostico** e incomum: cada correcao nos commits examinados vem com causa raiz, sintoma numerico e um teste que falha sem ela.

Tres conclusoes estruturais:

1. **O eixo "janelas de memoria" e o mais instrutivo do repositorio.** Uma feature correta (`f3b15b9`) carregou um erro de 32 bits por tres dias; a primeira correcao (`a787c52`) estava certa em logica e derrubou um jogo por causa da **forma** do codigo gerado; o revert (`70f8dc4`) devolveu o bug; a solucao final (`f9e5ee9`) veio junto com uma mudanca arquitetural (descriptor heap, #187) que tornou a forma segura. E o unico caso no periodo em que uma correcao precisou ser desfeita, e ele mostra que, num projeto desses, **o compilador do alvo faz parte do sistema**.

2. **A segunda metade do periodo e uma migracao de backend.** O tradutor direto RDNA2->DXIL (`#1426`) e o lowering per-lane (`#1745`) concentram 22 dos 43 commits candidatos e **todos** os defeitos de corretude de semantica de GPU do periodo (D-04 a D-07, D-12 a D-18). O padrao de correcao e consistente: quando a semantica nao e conhecida, **recusar** e cair para o HLSL (`DIFFERS_ON_1426`, `uniform_over`, `stale`), o que mantem o produto correto as custas de cobertura. E, como ambos sao opt-in, nenhum desses defeitos chegou ao usuario padrao.

3. **As correcoes mais duradouras sao as que mudam a representacao, nao as que tratam o sintoma.** `69a892c` (limite + forma alternativa), `188ee17` (setup de lanes incondicional), `a23b6ff` (dono do valor), `bfb8133` (identidade do cache) e `f9e5ee9` (endereco completo + range-check) sao invariantes. As que tratam sintoma (`425b869`, `fe8dc84`, `878afda`, `b1bfd1e`) sao honestas — estao comentadas como workaround — mas continuam abertas.

Risco principal que fica: **G-03**. A forma de 64 bits por acesso que o mantenedor identificou como causa do crash do `amdxc64` esta de volta em `HEAD` (no caminho switch), e o caso concreto que falhou (o CS `8e4af296b9d7fd9c` do XDefiant) nao tem evidencia de re-teste. E o unico ponto em que este historico sugere que a licao pode nao ter sido totalmente absorvida.

## K. Indice de commits

Todos os 853 commits de `main`, em ordem cronologica (`git log --reverse --format="%h %ad %an %s"`), um por linha: `hash data autor assunto`. Os 43 commits do pacote de evidencias foram lidos com diff; os demais constam aqui para completude. Merges aparecem como tais.

```
8fa7fa0 2026-11-20 ivall Initial XCL source and TTPF viewer
a21d3df 2026-11-20 ivall Session 5: console DLLs, tiling, BVH format, compute PSO translation
57b529c 2026-11-20 ivall Console root-argument layout, D3D12.X inline command list finding, memory cap for probes
e714e2a 2026-11-21 ivall D3D12.X compute command list format from XboxBvhBuilder (fake list probe, record writer scan)
b56a1b2 2026-11-21 ivall Console command lists: the console XboxBvhBuilder builds a BVH on the PC GPU
575c768 2026-11-21 ivall Console command lists hooked into creation, Close/Reset and ExecuteCommandLists
cc29f47 2026-11-21 ivall HANDOFF: do not emulate the console FSR4 (fp8 ML model, console-cooked pipelines); HLE it with the exe
8c6f1c3 2026-11-21 ivall Translator: structured control flow, loop-free float atomics, select-based VGPR writes; TLAS probe
700c83e 2026-11-21 ivall Branch-free HLSL: DXC optimises big console shaders again; TLAS on the GPU confirmed
a02eb66 2026-11-21 ivall FSR4 context on XCL's device: console resource creation and descriptor heap ABI
9f217bc 2026-11-21 ivall Console descriptor tables and heaps (learned from the console FSR4 dispatch)
bb363d1 2026-11-21 ivall Undo the crate-wide rustfmt of 9cb9c94, keep its changes
3453dd9 2026-11-21 ivall Console buffers in unified memory; UAV clears and texture copies on console lists
2b7e79b 2026-11-21 ivall Translator survey over the whole retail shader pack
a89f714 2026-11-21 ivall Translator: faster DXC - skip-flag structuring, branch-free helpers, DXC worker pool
b64dbd3 2026-11-21 ivall isa_coverage: translator coverage of every decoder instruction (ignored diagnostic)
74cd9db 2026-11-21 ivall MS (NGG) and GS pipelines from console shaders in PSO creation
c2123ec 2026-11-21 ivall More RDNA2 instructions, complete console descriptor heaps, more structured loops
71d2f82 2026-11-21 ivall Cube-map sample_d: LOD from the face gradients (DXC rejected SampleGrad on TextureCube)
dc902f1 2026-11-21 ivall README: current state, shorter (details in HANDOFF.md)
26cf5ec 2026-11-21 ivall PS with conservative depth: position input noperspective centroid (DXIL validation)
77aefa5 2026-11-21 ivall Settings from the game's MicrosoftGame.Config: PLS quota, SMT, title memory mode
c2a9ce6 2026-11-21 ivall Title sees the PC's processors (no console processor count, no SMT setting)
092a330 2026-11-21 ivall Static samplers: S# built from constants in console code become root signature static samplers
c6996f7 2026-11-21 ivall Console textures from console memory: GFX10 mip chains and upload to PC textures
885e6a2 2026-11-21 ivall XGameRuntime through the GDK thunks: InitializeApiImpl / QueryApiImpl with XCL and PC API objects
98d32ec 2026-11-21 ivall Translate the repository to English: comments, log/error/CLI messages, tests, docs, HANDOFF
0b17b2e 2026-11-21 ivall README with a detailed status table; drop the shader gallery; no personal paths in docs
f3b15b9 2026-11-21 ivall Title memory above 4 GB: UMA in up to four 4 GiB windows
23e241f 2026-11-21 ivall README: highlights and status grouped by area (code, services, graphics, media)
3235ebe 2026-11-21 ivall Non-blocking pipeline creation: a stand-in at once, the real pipeline built in the background
576b4db 2026-11-21 ivall XGameRuntime from the GDK function table: persistent local storage, console device type, more APIs
6e92fa2 2026-11-21 ivall Pipeline release fixes; README: no empty cells under What is missing
61492f6 2026-11-21 ivall 3D (thick and thin) and linear textures
1c8fae9 2026-11-21 ivall Tests: no process-wide switch in a parallel test, registry cleanup and advapi32 routing checked
a6ccf9c 2026-11-21 ivall XGameRuntime: gamer picture, default audio endpoint, text entry through the PC runtime
d575a8c 2026-11-21 ivall HANDOFF: the GDK function count (123 XCL, 39 not implemented)
c3b10f1 2026-11-21 raylitebull XAudio2: real Opus decoder via Windows MSOpusDecoder MFT
2bd8fcc 2026-11-21 raylitebull XAudio2: real Opus decoder via Windows MSOpusDecoder MFT
38b17ba 2026-11-21 ivall Merge PR #3: real Opus decoding through Windows' MSOpusDecoder MFT
3365f6e 2026-11-21 ivall Opus: the AWC block size when the voice format has no nBlockAlign; checked on the game's audio
aa8280f 2026-11-21 ivall Merge PR #3 (rebased head): its content is already on main
06a4a49 2026-11-21 bmjask Olympus isa compatibility
4cf10c7 2026-11-21 bmjask Olympus assets compatibility
c3049b0 2026-11-21 bmjask Olympus media compatibility
3bd4dfe 2026-11-21 ivall Merge PR #5: more RDNA2 instructions
e819d36 2026-11-21 ivall PR #5 follow-up: f64 ceil / sqrt / rsq on the GPU, the swaprel test
9ebb9b1 2026-11-21 ivall Merge PR #8: DXIL cache single flight, RPF scanning (without its XAudio2 part, superseded by PR #3)
b1477f8 2026-11-21 ivall PR #8 follow-up: a cached-only attempt does not wait for a compile in flight
82efa14 2026-11-21 ivall Merge PR #6: texture and resource validation (the linear layout stays addrlib's)
eec9fb4 2026-11-21 ivall PR #6 follow-up: linear textures keep addrlib's layout
59eaf46 2026-11-21 ivall Persistent local storage: used bytes measured, the quota reduced by them
7c98008 2026-11-21 ivall HANDOFF: the merged contributions (PRs #5, #6, #8, part of #7)
5aae781 2026-11-21 bmjask Olympus: validate fresh DXIL cache output
23c7099 2026-11-21 bmjask Olympus: key DXIL cache by toolchain
fb3e8e8 2026-11-21 bmjask Olympus: fingerprint the loaded DXC toolchain
b8f4b0a 2026-11-21 bmjask Olympus: normalize DXIL cache lock paths
c9565af 2026-11-21 bmjask Olympus: use DXC metadata cache identity
27bc5c2 2026-11-21 bmjask Olympus: validate GameSave read name arrays
d0d3062 2026-11-21 bmjask Olympus: preserve GameSave result retries
813172a 2026-11-21 bmjask Olympus: honor disabled process task queues
f410fa1 2026-11-21 ivall 64-bit atomic inc/dec: the first read through the memory window accessors
88b68b4 2026-11-21 bmjask Olympus: probe large RPF9 archives safely
660a62a 2026-11-21 bmjask Olympus: load UTF-16 game configurations
55ebf2b 2026-11-21 bmjask Olympus: validate D3D12 texture dimensions
122bff2 2026-11-21 bmjask Olympus: emulate geometry wave64 on wave32
63c8bd7 2026-11-21 ivall Merge pull request #6 from bmjask/olympus-assets
895abb0 2026-11-21 ivall Merge pull request #9 from bmjask/codex/olympus-dxil-cache
7897f13 2026-11-21 ivall Merge pull request #10 from bmjask/codex/olympus-gamesave-safety
3bb4323 2026-11-21 ivall Merge pull request #11 from bmjask/codex/olympus-process-queue
299cd6f 2026-11-21 ivall Merge pull request #13 from bmjask/codex/olympus-rpf-streaming
28c95b4 2026-11-21 bmjask Olympus: preserve native geometry wave64
b81284c 2026-11-21 ivall Test directory names; HANDOFF: the second round of contributions (#6, #9-#11, #13)
b2cc8ed 2026-11-21 bmjask Olympus: share geometry wave selection
a3ec898 2026-11-21 bmjask Olympus: report supported runtime features
83f476b 2026-11-21 bmjask Olympus: align runtime feature availability
5a5c32b 2026-11-21 ivall PresentX and console frame pacing
861869f 2026-11-21 ivall D3D12.X command lists in the console's vtable layout
e280e6a 2026-11-21 ivall Merge pull request #12 from bmjask/codex/olympus-runtime-features
daac133 2026-11-21 ivall Merge pull request #14 from bmjask/codex/olympus-wave32-geometry
d4dc2f5 2026-11-21 bmjask Olympus: report mesh wave requirements
d5703bf 2026-11-21 ivall D3D12.X: the console device's own methods
50688d5 2026-11-21 ivall D3D12.X: CreateDerivedPipelineStateX
eddf5a7 2026-11-21 ivall D3D12.X: the console command list's own methods
2ff23c3 2026-11-21 ivall D3D12.X: nested occlusion queries
e8106bc 2026-11-21 ivall Merge pull request #15 from bmjask/codex/olympus-mesh-wave-analysis
3f68644 2026-11-21 ivall D3D12.X device as IDXGIDevice; XMemVirtualAlloc
fae7655 2026-11-21 ivall D3D12.X: IDXGIDevice only for console code
ea98a6b 2026-11-21 ivall HANDOFF: IDXGIDevice for console code only
972c589 2026-11-21 ivall Note: the console pipe configuration differs between Series S and X
b8151eb 2026-11-21 bmjask Olympus: reject invalid mesh branch targets
f81441a 2026-11-21 ivall Fortnite's console imports: xmem statistics and physical pages, IID data, DXC
9f62692 2026-11-21 ivall Merge pull request #16 from bmjask/codex/olympus-mesh-invalid-targets
c768791 2026-11-21 bmjask Olympus: add mesh wave dispatcher foundation
70835a2 2026-11-21 ivall Console user32 / legacy DLLs routed; task queue handles checked
c2e1f8d 2026-11-21 ivall Stub database: console DirectStorage and xg_x fail instead of a fake S_OK
68c2c20 2026-11-21 bmjask Olympus: isolate GameSave async validation test
a379e68 2026-11-21 ivall Console DirectStorage (xcrdapi.dll)
3c546ef 2026-11-21 ivall Console texture layout library (xg_xs.dll); the console's pipe configuration
bdae5a4 2026-11-21 ivall D3D12.X: textures in title memory follow it
fd74662 2026-11-21 ivall Physical pages mapped into reserved address space
92beeab 2026-11-21 bbareth RPF9: unify header diagnostics across scan modes
18cb400 2026-11-21 ivall XG metadata planes; the console shader compiler as a recompiler test generator
6da1947 2026-11-21 raylitebull XAudio2: pure-Rust XMA1/XMA2 decoder
e555b87 2026-11-21 ivall XDefiant: the first stops on the way in; more console shader kernels
6abe17d 2026-11-21 raylitebull DirectStorage: submitted cancellation, intermediate_size, and physical-page destinations
6a27beb 2026-11-21 raylitebull HLE: implement mfplat.dll!MFResetDXGIDeviceManagerX with system fallback
f12d724 2026-11-21 ivall Console command lists of the title's own code; compute root arguments; safety nets
97e103d 2026-11-21 ivall Graphics root arguments of console code; title-memory resources; diagnostics
58c1f42 2026-11-21 ivall Probe: D3D12 heaps over title address space (sections in placeholders)
ba422b7 2026-11-21 ivall heap_probe: no needless casts
e89985f 2026-11-21 raylitebull XGameRuntime & advapi32legacy: UE4 startup, login, and hotfix parity
f6ad90c 2026-11-21 ivall Aliased GPU-owned textures in title memory (video-memory arenas)
e7ca2cd 2026-11-22 Antariska test: accept all valid Windows MMDevice endpoint ID formats
e511b94 2026-11-21 ivall Merge pull request #24 from raylitebull/xgameruntime-startup-parity
a2baa21 2026-11-21 ivall Merge pull request #18 from bmjask/codex/olympus-gamesave-test-isolation
872fbe0 2026-11-21 ivall Merge pull request #19 from BBareth/codex/rpf-header-diagnostics
3754f01 2026-11-21 ivall Merge pull request #17 from bmjask/codex/olympus-mesh-dispatch-pr
0f16f59 2026-11-21 ivall Merge pull request #20 from raylitebull/xma2-decoder
13e2af9 2026-11-21 ivall Merge pull request #25 from Lum7nt/fix/audio-endpoint-test-format
72bb284 2026-11-21 ivall Merge pull request #21 from raylitebull/dstorage-enhancements
d82829c 2026-11-21 ivall Merge pull request #22 from raylitebull/mfplat-reset-dxgi
3dfbcfb 2026-11-21 raylitebull d3d12x: implement KickoffX, Wait32/64BitValueX, and RSSetDepthBiasX
660f6dc 2026-11-22 ivall XDefiant: title memory on the GPU (sparse windows), graphics root arguments, vertex buffers
0fb82c6 2026-11-22 raylitebull D3D12.X & GPU: raytracing SRVs, 64-bit multi-window BVH traversal, and title-list BVH builder slots
af6aaa1 2026-11-22 bbareth Manifest: scope the local storage quota to its section
c13c774 2026-11-22 bbareth Disassembler: honor explicit byte ranges
c2e6ce9 2026-11-22 bbareth Shader tools: keep DXC timings out of JSON output
0fac0d3 2026-11-22 bbareth RPF9: reject conflicting probe options
74eaad0 2026-11-22 bbareth Launcher: honor configured executable before auto-detection
507c441 2026-11-22 ivall XDefiant title screen: shared descriptor heap, quad lane reads, audio processor, GameInput
e7e0dd0 2026-11-22 ivall Merge pull request #26 from raylitebull/raytracing-dxr
7e74038 2026-11-22 ivall Merge pull request #23 from raylitebull/list-x-extensions
3a8bc96 2026-11-22 ivall Fortnite reaches its splash screen: TLS slot 0, deferred texture uploads, console heaps and resources, merged ES+GS
6038bb1 2026-11-22 KrigS.exe docs: record focused XNetworking Rust review
6c5af1b 2026-11-22 KrigS.exe Update XNetworking Rust review with additional context
546f056 2026-11-22 KrigS.exe Enhance XNetworking Rust review with detailed guidelines
8eb3cf4 2026-11-22 ivall Fortnite UI and title screen draw: PS attributes and VS params are registers
63a05b0 2026-11-22 ivall Merge pull request #27 from BBareth/codex/manifest-storage-quota
1fb776c 2026-11-22 ivall Merge pull request #28 from BBareth/codex/disasm-byte-range
b9002e0 2026-11-22 ivall Merge pull request #29 from BBareth/codex/shaders-json-compile
9db80ef 2026-11-22 ivall Merge pull request #30 from BBareth/codex/rpf-probe-option-conflicts
54df001 2026-11-22 ivall Merge pull request #32 from Krigsexe/harden/networking-unsafe-config
d272821 2026-11-22 ivall Merge pull request #31 from BBareth/codex/launcher-config-executable
9e759de 2026-11-22 ivall Minecraft reaches its main menu: title window as the console screen, heap buffers in unified memory
1903ff6 2026-11-22 ivall No Xbox Live tokens by default: titles take their offline paths
e80ce8d 2026-11-22 ivall Whole shader pack: effect-qualified records, far jumps, every program translates
550d979 2026-11-22 ivall Minecraft's create-world screen: helper lanes of other quads, unread VS inputs, PLS drive
bb58372 2026-11-22 ivall Minecraft is playable: Ore UI gamepad, no draw skipped, keyboard and mouse as a gamepad
6cee70f 2026-11-22 ivall README: graphics works, what is still missing, console drives and data folders
145bc56 2026-11-22 grooving6 Float atomic min / max in memory: -0.0 operands, f64 forms
126268c 2026-11-22 ivall Unicorn Overlord is playable: the newer console runtime's queue and list layouts
004b3db 2026-11-22 wrxqed recomp: support wave32 split execution for console mesh shaders
cf08a50 2026-11-22 ivall Merge pull request #38 from grooving6/fix/float-atomic-minmax
4a8344c 2026-11-22 raylitebull Route D3D12 buffer heaps, XMem graphics memory, and descriptor heaps to sparse VRAM
2dd04da 2026-11-22 Graham Calihan XCL_ADAPTER: choose the D3D12 adapter, for titles and the GPU tests
2bc2e37 2026-11-22 Graham Calihan GPU tests: skip mesh draws on adapters without mesh shaders
3031945 2026-11-22 Graham Calihan Test title: take the device from d3d12_xs.dll, not the system's d3d12.dll
5c15581 2026-11-22 Graham Calihan README: XCL_ADAPTER for the GPU tests
8ebba61 2026-11-22 Graham Calihan gpu_pso_e2e: skip the mesh-pass parts on adapters without mesh shaders
bb43b5f 2026-11-22 ivall Merge pull request #40 from raylitebull/sparse-vram-pools
acbbc55 2026-11-22 ivall Sparse VRAM: keep window 0's descriptor range out of the title's memory
bb9f0b5 2026-11-22 ivall Committed console textures leave the coherent window room; memory and fps diagnostics
7396683 2026-11-22 ivall Remove tools/ttpf-viewer; README: no performance gap from GPU buffers in system memory
5dd2d56 2026-11-22 ivall The title's window stays hidden whichever way it is made
a312759 2026-11-22 Graham Calihan Split wave32 geometry: refuse prefix counts over a mask
afaf413 2026-11-22 raylitebull Sparse VRAM: async ring-buffered uploads, cached write-watch ranges, and coherent headroom
798ff0d 2026-11-22 Graham Calihan recomp: refuse split wave32 mesh programs with cross-half wave64 semantics
4de9016 2026-11-22 ivall Committed console resources XCL does not place get the PC's description
aac59d6 2026-11-22 ivall A small menu when xcl.exe is opened from Explorer; README trimmed
e2f185b 2026-11-22 ivall Merge pull request #45 from raylitebull/sparse-vram-perf-and-coherent-headroom
912bb5d 2026-11-22 ivall Test title: what the retail games do, as checks anyone can run; readback buffers may use the kept room
28ee5d7 2026-11-22 gefydev XNetworking: read CERT_CHAIN_CONTEXT at its real offsets
5f7ec80 2026-11-22 ivall Merge pull request #44 from grahamster2/fix/wave32-mask-prefix-count
2bfcd7a 2026-11-22 ivall Merge pull request #39 from wrxqed/fix/mesh-wave32
7b42c08 2026-11-22 ivall README: mesh shaders are required; what GPUs with 32-lane waves run (#42)
4e5c45a 2026-11-22 gefydev Config: the Series S title memory for every Series S spelling
5d6829e 2026-11-22 ivall Merge pull request #41 from grahamster2/adapter-select
f5e6b7a 2026-11-22 ivall Merge pull request #47 from gefydev/xnetworking-chain-offsets
055b0ad 2026-11-22 ivall Merge pull request #48 from gefydev/config-series-s-memory
9a15c53 2026-11-22 ivall XCL window: shader compilation in its title; the game's display name
521ec08 2026-11-22 ivall Shader compilation shown for every DXC compile; with little commit left, compiles go one at a time
61c6be8 2026-11-22 gefydev VFS: a console drive mount leaves the PC's own folders on that drive
a1869a5 2026-11-22 ivall Merge pull request #49 from gefydev/vfs-host-drive-paths
ff9430e 2026-11-22 Foniass GPU tests that cannot run on the adapter say so: SKIPPED <test>: <reason> on stderr, the count at the end of scripts/test.ps1, XCL_REQUIRE_GPU=wave64|mesh to fail instead (#42)
a3feb68 2026-11-22 Mavixot Report DXIL cache activity and compilation failures
a081a65 2026-11-22 gefydev MicrosoftGame.config: attribute values with their character references replaced
6f8c4ec 2026-11-22 ivall Merge pull request #50 from Mavixot/pso-cache-diagnostics
79769cb 2026-11-22 ivall Merge pull request #51 from Foniass/t3/make-gpu-test-skips-visible
dd713e3 2026-11-22 ivall Merge pull request #52 from gefydev/xml-attr-entities
48aabc1 2026-11-22 gefydev XAsync: cancellation is the provider's to do
6a270f5 2026-11-22 gefydev Task queues: terminating cancels delayed callbacks before the termination callback
cfed905 2026-11-22 gefydev Game saves: names Windows does not keep as file names stay intact
5c6bbc5 2026-11-22 gefydev Task queues: a terminated queue takes no new work
f2d5b6f 2026-11-22 gefydev VFS: moves through MoveFileWithProgressW are redirected too
2ca2b73 2026-11-22 gefydev XAsync: a failing Begin fails the call, and E_PENDING does not complete
935155c 2026-11-22 gefydev VFS: CopyFileW is redirected too
f6a249f 2026-11-22 Mavixot cache compiled shaders between pso requests
33efe92 2026-11-22 gefydev MicrosoftGame.config: commented-out entries are not the game's
817328d 2026-11-22 gefydev MicrosoftGame.config settings: comments skipped, UTF-16 BE read
80489d0 2026-11-22 gefydev VFS: FindFirstFileExW with FindExSearchLimitToDirectories is listed too
3283d11 2026-11-22 gefydev XUserGetGamertag: components as XUserGamertagComponent documents them
654b87b 2026-11-22 gefydev XSystemGetConsoleId: the console id in its documented format
c8d4a2f 2026-11-23 Foniass v_fract_f32: ±inf gives NaN and a NaN stays NaN, as on RDNA
319bcd6 2026-11-23 Foniass Console ResourceBarrier: UAV barriers for all resources
22ebfae 2026-11-23 Foniass ds_add_rtn test: the old values may wrap around 0xFFFFFFFF
3ecf4cf 2026-11-23 Foniass Wave64 console programs on GPUs with 32-lane waves only: two exchanging wave32 halves
8a52568 2026-11-23 Foniass Recompiler prelude: drop the half-local wave64 form nothing generates any more
ab71385 2026-11-22 gefydev VFS: ReplaceFileW is redirected
21bbb85 2026-11-23 Foniass README / HANDOFF: wave64 programs on 32-lane GPUs
66e23c5 2026-11-23 Foniass GPU tests skip on adapters with neither 32- nor 64-lane waves (WARP)
ba81c3d 2026-11-22 grooving6 Float compares with NaN operands, f64 math past the f32 and int64 ranges
b8fca79 2026-11-23 grooving6 Run the f64 and NaN compare GPU tests as wave32 programs too
21e42cf 2026-11-23 Tems Fix D3D12 frame and texture handling
a8918b7 2026-11-23 unknown GPU tests: skip mesh_dispatch test when no SM 6.6 GPU is available
6ececa1 2026-11-23 unknown gpu: load optional D3D12 Agility SDK at runtime for SM 6.6+ on Windows 10
9a85c6e 2026-11-23 unknown xnetworking: serialize default security info with offset_of! instead of copying struct padding
ed37916 2026-11-23 unknown xnetworking: fail closed on cert verification errors and malformed pin metadata
4ee1b6e 2026-11-23 unknown agility: trim env path, absolutize relative paths, dynamic D3D12GetInterface, and expand tests
3b74cda 2026-11-23 unknown gpu/recomp: implement DS_WRITE_ADDTID_B32 and DS_READ_ADDTID_B32
31a4001 2026-11-22 gefydev Tests: a console compute shader through the D3D12.X PSO hook, dispatched and read back
1ad0d4a 2026-11-23 Foniass Console descriptor heaps: places in the shared heap a page at a time (NVIDIA flicker)
527fe98 2026-11-23 unknown gpu/recomp: exact RNE rounding and NaN handling for V_CVT_(PK)NORM_*_F16
a4b2e92 2026-11-23 unknown hle/dstorage: fix queue error reset, event handle duplication, close/release races, and pointer validation
8138894 2026-11-23 unknown gpu/adapter: fall back to numeric model token when index is out of range
385dc5f 2026-11-23 Foniass Console descriptor heaps: a heap's last page takes only the descriptors it has
98cb80a 2026-11-23 unknown xgameruntime: add XAccessibility closed caption and high contrast fallback
8ad0b44 2026-11-23 Foniass Placed textures larger on the PC than on the console get memory of their own
30d1e67 2026-11-23 unknown hle/kernelx: harden null-arg, VirtualFreeEx, GetThreadName, and overflow paths
15243ad 2026-11-23 unknown vfs: fix TRUNCATE_EXISTING on un-overlaid game files
73fb221 2026-11-23 unknown hle/xaudio2: enforce XAudio2 play/loop region invariants in clamp_regions and silent_buffer
7ac0828 2026-11-23 unknown hle/acphal: fix APU range overflow/overlap, SHAPE rollback, and XApuConnect OOM
93072dc 2026-11-23 unknown recomp: handle s_round_mode/s_setreg* and pin non-fused MAD/MAC
a963568 2026-11-23 ivall Merge pull request #53 from gefydev/xasync-cancel-to-provider
2e000cb 2026-11-23 ivall Merge pull request #54 from gefydev/taskqueue-terminate-delayed
331b734 2026-11-23 ivall Merge pull request #55 from gefydev/gamesave-windows-names
daa27b0 2026-11-23 ivall Merge pull request #56 from gefydev/xasync-begin-complete-contract
eb8ed07 2026-11-23 ivall Merge pull request #57 from gefydev/taskqueue-submit-after-terminate
6ca99f2 2026-11-23 ivall Merge pull request #58 from gefydev/vfs-move-file-with-progress
8603bcd 2026-11-23 ivall Merge pull request #59 from gefydev/vfs-copy-file
533187c 2026-11-23 ivall Merge pull request #60 from gefydev/vfs-replace-file
8064fe3 2026-11-23 ivall VFS: MoveFileA, MoveFileExA and MoveFileWithProgressA are redirected too
e38cb65 2026-11-23 ivall XAsyncSchedule: a terminated queue takes no new work either
ff5c940 2026-11-23 ivall Game saves: a device name followed by spaces before its extension is escaped too
848e740 2026-11-23 ivall HANDOFF: PRs #53-#60 and what was measured on Windows
ef4d7d6 2026-11-23 ivall Merge pull request #84 from Foniass/fix/descriptor-heap-pages
a929884 2026-11-23 ivall Merge pull request #68 from Foniass/nvidia-uav-barriers-fract
55fca5a 2026-11-23 ivall Merge pull request #69 from Tems-py/fix/d3d12-frame-and-texture-handling
9f353e7 2026-11-23 ivall Merge pull request #67 from Foniass/wave64-on-wave32
593a237 2026-11-23 ivall Console lists bind a title's own view heap only when it is shader visible
46a1c3f 2026-11-23 ivall Compute: console thread ids follow the hardware lanes in 2-D / 3-D groups
ba905cf 2026-11-23 ivall XCL_SPLIT_WAVE64=1: the 32-lane GPUs' split wave64 form on an RDNA GPU too
55df81b 2026-11-23 ivall HANDOFF: PRs #84, #68, #67, #69 and the 2-D group lane layout found on AMD
be1994b 2026-11-23 ivall Merge pull request #46 from grooving6/fix/f64-math-nan-compares
6a41c55 2026-11-23 ivall Recompiler tests: #46's wave sizes with #67's gpu_ready
b4edefb 2026-11-23 ivall HANDOFF: #46 merged, #43 held (MSAA against the console's XG), XG versions in the samples
f112834 2026-11-23 ivall Merge pull request #61 from Mavixot/shader-compile-time
5412c24 2026-11-23 ivall Merge pull request #63 from gefydev/manifest-shared-decoder
3fa4203 2026-11-23 ivall Merge pull request #64 from gefydev/vfs-find-limit-to-directories
2c1c9ed 2026-11-23 ivall Merge pull request #65 from gefydev/xuser-gamertag-components
5ed035c 2026-11-23 ivall Merge pull request #66 from gefydev/xsystem-console-id-format
c2887bb 2026-11-22 gefydev recomp: f64 abs / neg modifiers on the sign bit (#72)
71c4b9b 2026-11-23 ivall HANDOFF: PRs #61-#66 and #72
949907b 2026-11-23 ivall Merge pull request #74 from certified-retart/fix/mesh-dispatch-skip-no-gpu
e132ed4 2026-11-23 ivall Merge pull request #76 from certified-retart/fix/xnetworking-security-info-padding
822c9b0 2026-11-23 ivall Merge pull request #77 from certified-retart/fix/xnetworking-cert-verify
9bc3a47 2026-11-23 ivall Merge pull request #78 from gefydev/compute-e2e
c94d6b7 2026-11-23 ivall Merge pull request #79 from certified-retart/feat/recomp-ds-addtid
f7d631e 2026-11-23 ivall Merge pull request #75 from certified-retart/feat/d3d12-agility-sdk
d70f8e9 2026-11-23 ivall HANDOFF: PRs #74-#79
92456af 2026-11-23 ivall Merge pull request #80 from certified-retart/fix/recomp-cvt-norm-f16
f8e7c2b 2026-11-23 ivall Merge pull request #81 from certified-retart/fix/dstorage-queue-bugs
2bba48b 2026-11-23 ivall Merge pull request #82 from certified-retart/feat/adapter-numeric-model-fallback
28132e8 2026-11-23 ivall Merge pull request #85 from certified-retart/feat/xaccessibility-apis
bd174d5 2026-11-23 ivall Merge pull request #86 from Foniass/fix/placed-texture-overlap
079b541 2026-11-23 Ivan John Ozimchuk VFS hooks: the guard keeps the thread's last error
22d21f0 2026-11-23 Ivan John Ozimchuk Loader: under Wine, PEB.ImageBaseAddress keeps the real main image
9284dcd 2026-11-23 ivall HANDOFF: PRs #80-#82, #85, #86
1f82be3 2026-11-23 ivall HANDOFF: PRs #80-#82, #85, #86
25f943e 2026-11-23 ivall Merge pull request #87 from certified-retart/fix/kernelx-edge-cases
83199a9 2026-11-23 ivall Merge pull request #88 from certified-retart/fix/vfs-truncate-existing
ef6b342 2026-11-23 ivall Merge pull request #89 from certified-retart/fix/xaudio2-region-clamping
c5aeee9 2026-11-23 ivall Merge pull request #90 from certified-retart/fix/acphal-overflow-and-oom
088d682 2026-11-23 ivall Merge pull request #91 from certified-retart/fix/recomp-fp-mad-and-round-mode
697dbb1 2026-11-23 ivall Recompiler tests: #91's GPU tests gate on gpu_ready (#67 removed gpu_ready_wave32)
ecbad66 2026-11-23 ivall HANDOFF: PRs #87-#91, issue #73
b4019c4 2026-11-23 hexgeon1 Split wave64: masks read only lane by lane stay in their half
3121db5 2026-11-23 ivall GameInput: the PC's controller – connect / disconnect logged, empty slots polled once a second, the real stick wins
d1e8d9e 2026-11-23 ivall VFS: device interface paths are not the title's files – the PC's controller reaches titles
9d1760f 2026-11-23 ivall Merge pull request #92 from ozimchuk/wine-keep-last-error
01966b6 2026-11-23 ivall Merge pull request #93 from ozimchuk/wine-keep-image-base
f5999e4 2026-11-23 ivall Merge pull request #94 from hexgeon1/wave64-local-masks
60599a9 2026-11-23 ivall HANDOFF: #92-#94, XDefiant in the split wave64 form on AMD
3885b86 2026-11-23 raylitebull d3d12x: fix sparse page holes, tile reclamation, query offsets, coherent buffer sanitization, descriptor copies, and DMA list release
9cd48da 2026-11-23 raylitebull Input: scancode WASD/EQ, focus-loss reset, keyboard_mouse gate, GameInput QI & callback sync
c204343 2026-11-23 raylitebull XGameSave: reject empty container name in XGameSaveDeleteContainer
d903a80 2026-11-23 raylitebull VFS: fix case-only rename data loss, overwrite flags, and failed copy/move whiteouts
d7eae06 2026-11-23 raylitebull VFS: fix whited-out directory listing, type checks, and directory recreation whiteouts
2d94699 2026-11-23 raylitebull XGameRuntime: fix XGameSave atomic/NTFS/null-out, XPackageGetIdentifier, locale fallback, and controller ID
0a3b802 2026-11-23 raylitebull xgameruntime: fix XAsync/XTaskQueue termination deadlocks, cleanup races, port tracking, and XErrorReport lock
eac85f9 2026-11-23 raylitebull Fix ANSI path encoding, HLE alias stubs/handles, ordinal/forwarder resolution, and stub race
c9c8677 2026-11-23 raylitebull d3d12x: fix async compute PSO tracking, initial PSO binding, DrawIndexedX, and KickoffX MRT restore
c85cd6f 2026-11-23 ivall HANDOFF: XDefiant's cold-cache UI checked – not a bug, the text's pipelines compile in a later burst
aaaa7ad 2026-11-23 raylitebull GPU recomp: fix PS interp ints, wave ops EXEC zeroing, ALU edge cases, and BVH det underflow
8f7a96a 2026-11-23 ivall Merge pull request #95 from raylitebull/fix/gamesave-empty-container-delete
6ea7a64 2026-11-23 ivall Merge pull request #96 from raylitebull/fix/vfs-move-copy-and-whiteouts
8247a60 2026-11-23 ivall Merge pull request #97 from raylitebull/fix/xgameruntime-package-and-saves
9c7bc7e 2026-11-23 ivall Merge pull request #98 from raylitebull/fix/xgameruntime-async-and-taskqueue
bbdede7 2026-11-23 ivall Merge pull request #99 from raylitebull/fix/d3d12x-sparse-offsets-and-heaps
62c9664 2026-11-23 ivall Merge pull request #100 from raylitebull/fix/input-scancodes-and-gameinput
fe63082 2026-11-23 ivall Merge pull request #101 from raylitebull/fix/resolver-aliases-and-ansi-paths
b3dd82d 2026-11-23 unknown XGameSave: defer container deletion until work dispatch
5587780 2026-11-23 ivall Merge pull request #102 from raylitebull/fix/d3d12x-async-compute-and-bind
41cc3d5 2026-11-23 unknown D3D12: check device removal before using GPU readbacks
02d1302 2026-11-23 ivall GameInput: refused QueryInterface IIDs are logged
668f550 2026-11-23 ivall HANDOFF: PRs #95-#102
c813f08 2026-11-23 ivall Merge pull request #107 from raylitebull/fix/recomp-alu-interp-and-wave-ops
5ec6570 2026-11-23 ivall Merge pull request #109 from certified-retart/fix/d3d12-fence-removal
221b5fd 2026-11-23 ivall Merge pull request #108 from certified-retart/fix/gamesave-defer-delete
f90e644 2026-11-23 ivall README: a Controls section (gamepad recommended, keyboard and mouse as a gamepad), "What is missing" removed; HANDOFF: PRs #107-#109
ea4bd62 2026-11-23 ivall Fixes from the audits in #103 / #106: PresentX on another queue, frame intervals either side, xcl.toml kept when unreadable
ef4aae1 2026-11-23 unknown hle/opus: honor Ogg Opus EOS granule trimming and pre-skip
58309b9 2026-11-23 ivall XDefiant's UI text back: v_interp of int / uint PS inputs converted again (#107 regression)
07f12e8 2026-11-23 Foniass Translated shaders: one memory-window switch per four dwords, no branches for loads
56fa1dc 2026-11-23 Foniass Shader compiles on every core but one
517d983 2026-11-23 Foniass xcl shaders --precompile: the HLSL the runtime asks for
f311ca5 2026-11-23 ivall Merge pull request #113 from Foniass/t3/faster-shader-compile
8ba374c 2026-11-23 gefydev xcl run: say what the PC is missing before the game starts
f7a9f5c 2026-11-23 gefydev Issue templates: a game report form that asks for report.json and xcl.log
ad48c26 2026-11-23 gefydev Issue templates: the game report form gets the bug label
c806a0b 2026-11-23 bmjask Keep draw and reset state transactional
0056934 2026-11-23 gefydev recomp: cube T#s are cube arrays – the face coordinate's slice is sampled
34efd81 2026-11-23 gefydev recomp: omod on f64 and f32/f16 conversions, integer clamp on the unsigned and packed adds and dots
a01d61c 2026-11-23 gefydev D3D12.X: queue #18 before the layout is known writes 8 bytes
70d7e0a 2026-11-23 gefydev D3D12.X: video-memory arenas never overlap
a787c52 2026-11-23 gefydev recomp: memory window from the full 64-bit address (carry out of the low dword)
3304bb1 2026-11-23 Foniass MS and GS programs translate with the title's memory windows
1a51a82 2026-11-23 Guyliano Buyssens Runtime: CPU check at start, bounded reads in the exception logger, thread dump and probes guarded, PE directory bounds (#106)
23c53fc 2026-11-23 Guyliano Buyssens Frame clock: each frame its own origin, ancillary handles outlive the clock tick (#103)
fdeacf0 2026-11-23 ivall FidelityFX: the console FSR4 upscaler's passes translate and a frame runs on the GPU
7bf9896 2026-11-23 ivall Merge pull request #110 from certified-retart/fix/opus-eos-trim
fe9f2dd 2026-11-23 ivall Merge pull request #115 from gefydev/preflight
44a115d 2026-11-23 ivall Merge pull request #116 from gefydev/issue-templates
7f1e3f4 2026-11-23 ivall Merge pull request #117 from gefydev/recomp-cube-arrays
a39781b 2026-11-23 ivall Merge pull request #118 from bmjask/codex/draw-reset-correctness
73c7bbe 2026-11-23 ivall Merge pull request #119 from gefydev/recomp-clamp-omod
0a82d92 2026-11-23 ivall Merge pull request #121 from gefydev/d3d12x-queue-slot18
5231403 2026-11-23 ivall Merge pull request #122 from gefydev/d3d12x-vram-arenas
778e329 2026-11-23 ivall Merge pull request #123 from gefydev/recomp-window-carry
828d9db 2026-11-23 ivall Merge pull request #124 from Foniass/fix/mesh-gs-windows
ac18a27 2026-11-23 ivall Merge pull request #127 from gbuyssens/pr/eh-pe-cpu-check
275409e 2026-11-23 ivall Merge pull request #129 from gbuyssens/pr/frame-clock
c4df2fe 2026-11-23 ivall Recompiler: #119's clamped 8-bit dots through dot4_u8 / dot4_i8 again
70f8dc4 2026-11-23 ivall Revert "Merge pull request #123 from gefydev/recomp-window-carry"
95bda11 2026-11-23 ivall HANDOFF: PRs #110-#130 (merged, held, closed)
af0311d 2026-11-23 Foniass Wave64 VS / PS in 32-lane waves: the lanes 32..63 they read
fc60bf5 2026-11-23 Foniass Graphics pipelines translate VS / PS for the device's wave size
55156f0 2026-11-23 Foniass upper_lanes: v_swaprel, SDWA v_cmpx and absorbing constants
84414e1 2026-11-23 ivall Merge pull request #133 from Foniass/nvidia/wave64-graphics-lanes
484dd57 2026-11-23 ivall HANDOFF: PR #133 checked on the RX 9070
c8d9549 2026-11-23 Tiago Leal Title textures: uploads start from the state the title's transitions left (#104 item 2)
0511fed 2026-11-23 Nik3310 tests: fail when a configured shader sample cannot be read
1058c0b 2026-11-23 Tiago Leal Compute TG_SIZE: a partial last wave counts (#105 "also noticed")
0b89bdf 2026-11-23 Tiago Leal CI: build and test on a Windows runner (no GPU)
20a35f0 2026-11-23 Tiago Leal CI: the test title's run left out (no Gaming Services, no GPU on the runner); test.ps1 passes its arguments to the tests
b5380ad 2026-11-23 Tiago Leal Loader: a game DLL that failed to load is not left registered (#106)
8e2aec9 2026-11-23 Tiago Leal Tests: the compute PSO-hook end-to-end tests skip on an adapter without 32- or 64-lane waves (WARP)
87fe289 2026-11-23 Tiago Leal D3D12.X: draws with untranslated pipelines get the title's root signature
7495271 2026-11-23 Ivan John Ozimchuk Compute dispatches: a shader takes no more arguments from user SGPRs than it has
1ffa570 2026-11-23 Ivan John Ozimchuk Test without a GPU: no argument in a register a builder shader does not have
d21e372 2026-11-23 mmmharius Report: a crash reported while the report is being written is kept (#106)
6cb71db 2026-11-23 mmmharius Config: title_memory is read in any case (#106)
9add721 2026-11-23 ivall Retail game bring-up: physical pages, 1D/3D layouts, newer device, list and fence methods
b210576 2026-11-23 cortexreaver2099-creator D3D12.X: a title's TEXTURECUBE view is the cube array of its one cube
66e8e97 2026-11-23 ivall Retail game: commit physical page homes once, the DSP, list Reset #139, rect lists
db5dc26 2026-11-23 ivall Shader loading: dumps on request, translations kept on disk, precompile of compute
323767f 2026-11-23 Ethan Barr VFS: report existing game files on CREATE_ALWAYS
c00a305 2026-11-23 Ethan Barr VFS: preserve the destination when an overlay rename fails
956a476 2026-11-23 ivall Retail game reaches 30 fps: frame events sharing a deadline, ExecuteCommandListsX, big pipelines in the background
deccf2b 2026-11-23 Ethan Barr Recompiler: round f16 conversions and products to nearest-even by default
f13bc2e 2026-11-23 ryosen0 DirectStorage: name a refused factory IID in the log (#161)
fc4b2d4 2026-11-23 WarmMilkCodes VS / PS: the 32-lane form on every GPU that can run 32-lane waves (RDNA2)
10b50ea 2026-11-23 mstr-vinegar VFS: preserve destination for locked overlay moves
229708a 2026-11-23 ryosen0 XCL window: window_pos setting – center on the primary monitor or a clamped x,y
a6e52be 2026-11-23 ryosen0 kbm: free the cursor while the window is dragged or resized
5911fb0 2026-11-23 BaggyALPHA pe: reject overflowing directory field RVAs
a3dedf1 2026-11-23 BaggyALPHA user: validate RegisterClassExA descriptor size
f7c7a48 2026-11-23 TheHoracio e2e: host preflight warnings no longer fail the console-title check
d66f4c7 2026-11-23 Thomas Allow redirected Cargo progress in Windows PowerShell 5.1
e80b5b3 2026-11-23 ALi Add a security policy (#177)
782e45b 2026-11-23 ALi README: the Agility SDK folders read as two places, not one twice
c69c4d1 2026-11-23 ALi TLS: an image without an index variable gets its slot without a write through address 0 (#106)
af46b71 2026-11-23 @pauliusdotpro Report sampled frame content from optional frame dumps
62e0e25 2026-11-23 RandomGithub12309 Recompiler: wave32 src_vccz / src_execz and SDWA f16 inline constants
b8a7794 2026-11-23 Guyliano Buyssens Refused interfaces are logged: XGameRuntime objects and DirectStorage
3d99e80 2026-11-23 Guyliano Buyssens XCL_D3D12_TRACE shows the arguments: the registers, two stack arguments, what pointers point at
0b28a39 2026-11-23 Guyliano Buyssens Console CPU instructions a PC CPU lacks are emulated: clwb, clzero, rdpru, monitorx / mwaitx, rdpid
9392eae 2026-11-23 Guyliano Buyssens XCL_TRACE_IMPORTS: every call of an HLE import logged with its arguments and result
4c3c156 2026-11-23 Kalmarv xgameruntime tests: compare slots by the function bound to their name, not by function address
bb66204 2026-11-23 Kalmarv DirectStorage: count BCPack refusals per mode, opt-in solid fill of their destinations
38b05b7 2026-11-23 ivall Retail game shows its first menu: target pages instead of arenas, lazy physical pages, PC view formats
488e5fe 2026-11-23 Kalmarv Frame events: queue an origin before the wait it wakes can take it
8558ff1 2026-11-23 ivall Merge PR #160
e0cc44b 2026-11-23 ivall Merge PR #137
0636e7a 2026-11-23 ivall Merge PR #151
c3746f8 2026-11-23 ivall Merge PR #191
ad71cde 2026-11-23 ivall Merge PR #166
2f6b723 2026-11-23 ivall Merge PR #136
dfc1332 2026-11-23 ivall Merge PR #144
3aa8d4a 2026-11-23 ivall Merge PR #185
7b60990 2026-11-23 ivall Merge PR #180
0fb0fd8 2026-11-23 ivall Merge PR #171
2abc562 2026-11-23 ivall Merge PR #155
e8564f1 2026-11-23 RandomGithub12309 D3D12.X: fix SetPredicationBufferX resource state lifecycle and KickoffX restore
31635bd 2026-11-23 Remington Wilcox D3D12.X: format queries about the console's own formats (116 – 120) ask about their PC formats
68a6640 2026-11-23 Kalmarv Sparse windows: commits before the device are merged page ranges, not a list scanned per page
eaa36b6 2026-11-23 ivall Buffer mirrors: review fixes from PR #195; a test of the PC's depth formats and views
f17a5ee 2026-11-23 theaustinayers Title textures: small render targets the CPU fills are kept in step (glyph caches)
8fde77a 2026-11-23 BlestTerror Tests under Wine: Windows' own answers are checked only on Windows
d0846cf 2026-11-23 ivall Merge PR #135
3d62cb7 2026-11-23 ivall Merge PR #141
2514938 2026-11-23 ivall Merge PR #143
7786340 2026-11-23 ivall Merge PR #146
2aaf76a 2026-11-23 ivall Merge PR #148
61f6a78 2026-11-23 ivall Merge PR #149
e43fdac 2026-11-23 ivall Merge PR #158
9e28431 2026-11-23 ivall Merge PR #167
8dd9020 2026-11-23 ivall Merge PR #176
c4b5674 2026-11-23 ivall Merge PR #178
52c5195 2026-11-23 ivall Merge PR #179
d877df9 2026-11-23 ivall Merge PR #181
2297960 2026-11-23 ivall Merge PR #182
aaddda0 2026-11-23 ivall Merge PR #183
ed4af10 2026-11-23 ivall Merge PR #190
5af7d71 2026-11-23 ivall Merge PR #200
128c414 2026-11-23 ivall Merge PR #201
9e6b63d 2026-11-23 ivall Merge PR #204
4e3741f 2026-11-23 ivall Merge PR #205
1b0c610 2026-11-23 ivall modules: a crash report survives an address below its module's base (PR #147, gefydev)
ddfcffd 2026-11-23 ivall Merge PR #188
6da33fa 2026-11-23 ivall Merge PR #207
5a537b3 2026-11-23 1bugproof Docs: contribution guidelines for people and AI agents
fa2d1d3 2026-11-23 Kalmarv D3D12.X: rect lists drawn whole through a generated geometry shader
4bf5466 2026-11-23 Kalmarv Rect lists: test the full-screen pass order reported downstream
c217b1f 2026-11-23 Kalmarv Rect lists: test that a CPU-expanded rect's second triangle is completed too
938fdd2 2026-11-23 Kalmarv Rect lists: retry a refused GS pipeline without it, map untranslated stream rect lists
82cba12 2026-11-23 Kalmarv Rect lists: real::has_rect_list_gs, the pipelines given the generated GS
7ac4048 2026-11-23 Kalmarv Rect lists: no CPU expansion for a pipeline with the rect list GS
033a6d1 2026-11-23 Kalmarv Recompiler: preserve the FSPX compute user SGPR count
5ceb8fc 2026-11-23 Kalmarv D3D12.X: check at start-up that the GPU sees CPU writes to unified memory
b856022 2026-11-24 AliceWood Spread aligned sparse tile keys across hash buckets
a0dce86 2026-11-23 Bloxive Respect the PLS quota in GetDiskFreeSpaceEx
0cd0a53 2026-11-23 ivall SSE4a on Intel CPUs: EXTRQ and INSERTQ emulated (#216)
247d909 2026-11-23 Kalmarv DXC: log and report the loaded version, note below 1.8, newest DXC among SDKs first
09863ba 2026-11-23 Kalmarv D3D12.X tests: DrawIndexedX test no longer depends on what earlier tests allocated
7b029e2 2026-11-23 yous0serious Thread affinity: console masks with processors the PC lacks are fitted to the process's (#106)
af20b4b 2026-11-23 ivall SSE4a as a Zen 5 runs it: high quadword zeroed, the last of F2 / F3 decides, 0x67 and early REX ignored
5424468 2026-11-24 AliceWood Merge commit '0cd0a53b584a8be0fd779d16e5826d97a0e66344' into perf/sparse-tile-hash-eaa36b6
b25582a 2026-11-24 Jefter perf(insn): CLZERO loop fast-forwarding and CLWB NOP hot-patching
327c24c 2026-11-23 Kalmarv Video-memory arenas: D32S8 and MSAA targets get heaps of their own size, let go when unused
4c1630b 2026-11-23 ivall Merge PR #213
13bb3e1 2026-11-23 ivall Merge PR #219
1200aa4 2026-11-23 ivall Merge PR #220
3242c03 2026-11-23 ivall Merge PR #221
c118c7c 2026-11-23 ivall Merge PR #222
2918c9f 2026-11-23 ivall Merge PR #223
f0a917b 2026-11-23 ivall Merge PR #199
f159441 2026-11-23 ivall Merge PR #211
31def9e 2026-11-23 ivall Merge PR #217
20271d4 2026-11-23 artor410 Recompiler: image_sample_o under an implicit LOD moves each blended mip by its own texels
594f63a 2026-11-23 Kalmarv KernelX: memory tracing keeps the caller's last error
ad289ef 2026-11-23 ivall Merge PR #210
d4f17ca 2026-11-23 ivall D3D12.X: Wait64BitValueX compares the whole value (PR #218, Sr-0w)
bc59624 2026-11-23 Kalmarv Console CPU instructions: CLZERO with an address-size prefix zeroes the line at EAX
05a5fd5 2026-11-23 Bloxive Merge upstream main into the PLS quota branch
8a98da6 2026-11-23 artor410 Recompiler: image_sample_o under an implicit LOD moves each blended mip by its own texels
0c63fd9 2026-11-23 ivall Merge PR #225
8c5d16e 2026-11-23 ivall Merge PR #230
1eee7ad 2026-11-23 ivall Merge PR #231
91844fb 2026-11-23 ivall Merge PR #226
bc9ef13 2026-11-23 ivall Merge PR #227
0f34407 2026-11-23 ivall Merge PR #229
6b9f2ba 2026-11-23 Kalmarv Console CPU instructions: CLZERO with an address-size prefix zeroes the line at EAX
5c2eaf5 2026-11-23 ivall Merge PR #227
a6f5368 2026-11-23 ivall Merge PR #229
60565c4 2026-11-23 ivall Rect lists' and image_sample_o's test runners as PR #229 has them (draw_gs_views)
5ba2b6a 2026-11-23 slopgobblercobbler Runtime: validate Zen 2 ISA features and CPUID at startup in launcher and host
608a79b 2026-11-23 Claude kernelx: memory in use from PagefileUsage where PrivateUsage is 0
c858190 2026-11-23 Claude testgame-mw: no unit-test harness
77905f6 2026-11-23 Logan Lazy physical pages: a thread that loses the race to commit a page retries
7a033c8 2026-11-23 ivall Retail game goes past its first menu: struct-returning device methods tell the return buffer from the device
1775ebb 2026-11-23 Claude d3d12x null device: descriptor heap start handles in rax
8adb9dd 2026-11-23 Claude xgameruntime: say what is missing without System32\xgameruntime.dll
163b55e 2026-11-23 ivall Merge PR #212
63b9930 2026-11-23 ivall Docs: what a PR needs to be merged - tests run, a regression run on a title, checked by PR checks
9ab7d76 2026-11-23 ivall Docs: no PRs written with small or open-weight models; PR checks refuses them in AI use
2f5979e 2026-11-24 ryosen0 this_first: a test of the stale-vtable return buffer and either argument order
2c5fe3f 2026-11-23 Kalmarv Memory: a scatter or map home of n pages costs O(n), not O(n²)
f03fe38 2026-11-23 Kalmarv Rect lists: the GS carries flat outputs, v0 the provoking vertex
d54597a 2026-11-23 Claude e2e: stage the test title under CARGO_TARGET_TMPDIR
ebbf377 2026-11-23 Kalmarv Memory: a freed sparse block is decommitted before its address is reused
130a143 2026-11-24 ryosen0 Merge main into this-first-tests
9e9551b 2026-11-24 Alp Pesen Console CPU instructions: fix ClzeroLoop ModRM check and loop semantics
d0b6079 2026-11-23 theaustinayers DirectStorage: a destination page array entry that is 0 at enqueue is read at the request's turn
9507d05 2026-11-23 AAigars kernelx: the finest system timer for the title (choppy audio, #260)
0e946cf 2026-11-23 ivall PR checks: only the PR's own run must be clean, main's run must be from main; game data refused
e93dc9c 2026-11-24 Jesse kernelx: GPU-visible memory outside the windows is write-watched (menu text)
9bd7635 2026-11-24 Jesse Memory: GPU views of the title's CPU memory read the CPU view (intro video)
2e67909 2026-11-23 Bloxive Recompiler: preserve flat pixel parameter bits (#249)
7bdc97f 2026-11-23 Claude d3d12x null device: heap start test calls #9 / #10 with their real signature
a99f64c 2026-11-23 imnicho Console CPU instructions: no VirtualQuery at every emulated instruction
4017f8f 2026-11-23 Austin Ayers Merge pull request #289 from theaustinayers/dstorage-late-page-array
8f53764 2026-11-24 ivall Merge PR #263
dfc19ed 2026-11-24 ivall Merge PR #290
c6b6aab 2026-11-24 ivall Merge PR #293
ef97948 2026-11-24 ivall Merge PR #254
ad79932 2026-11-23 Gennoun Marso Iliass D3D12.X: back kept-in-step title textures by shared VRAM pages (#248)
5a7346f 2026-11-24 ivall Merge PR #259
ab4324e 2026-11-24 ivall Merge PR #270
276cdd5 2026-11-24 ivall Merge pull request #244 from Cattn/fix-lazy-page-race
ee0adee 2026-11-24 ivall Host: no second CPU check (PR #259 review)
21b8723 2026-11-24 ivall Merge pull request #283 from jessenic/fix/gpu-mirror-view
85bdf21 2026-11-24 ivall Merge main (PR #283) into the PR batch
c8e3ca0 2026-11-23 Logan DirectStorage: EnqueueRequest reads its page array and file vtable without sizing regions
603cdd3 2026-11-23 imnicho VFS: XCL's shader cache on a drive a console mount names stays where it is
d6cbfcd 2026-11-23 Kalmarv D3D12.X: a full write-watch buffer goes on after its last page
01a7446 2026-11-24 Ivan John Ozimchuk D3D12.X: constant buffer views are float4 rows (stride 16), not raw buffers
db1ddb2 2026-11-24 ivall XCL_NETWORK=0 for the whole process: the system's WinHTTP, WinINet and Winsock refuse too (#305)
340189c 2026-11-24 ivall Merge PR #306
aeaeecb 2026-11-24 ivall Merge PR #288
29fb6b7 2026-11-24 ivall Title textures test: the process's video memory is not asserted (other GPU tests allocate at the same time)
4403887 2026-11-24 Xavi Fortes D3D12.X: resolve GPU timestamps targeting memory outside unified memory
2b4bb43 2026-11-24 ivall xapu: decode the title's Opus packets into its output buffer; libopus where Windows has none (PR #294, simnJS)
bb436a0 2026-11-23 Prime1997 Tests: read back mirrored textures across writes and page remapping
3516080 2026-11-24 ivall Title textures test: an array or cube a driver will not make reserved is committed, not a failure
f673b54 2026-11-24 ivall Merge PR #335
1efacca 2026-11-24 ivall Merge PR #322
da28099 2026-11-24 ivall Merge PR #331
0fdefe4 2026-11-24 ivall Merge PR #330
a8072aa 2026-11-24 ivall Merge PR #338
97dc49c 2026-11-23 Graham Calihan Scripts: smoke.ps1 names the adapter XCL ran on, from report.json
fa9ad89 2026-11-23 SandimL Scripts: run smoke.ps1 on Windows PowerShell 5.1
ef5312f 2026-11-24 ivall Memory: readable answers a short range from the working set (PR #332, Vaximous)
a6f5f7a 2026-11-24 ivall Title textures test: skipped on an adapter that makes no reserved cube with mips
a94e151 2026-11-23 Logan Textures: a layout's high-coordinate tables hold one period, not 65536 entries
c55783e 2026-11-24 1984742628 D3D12.X: title texture uploads reuse their queue's buffer and list
7d60a48 2026-11-24 1984742628 Textures: detiling a mip level computes column and row parts once
2cefc88 2026-11-24 1984742628 D3D12.X: the title textures of a big batch are detiled on several threads
2997c9a 2026-11-23 Mavix updated discord link
7695539 2026-11-24 nicho D3D12.X: a placement on title pages sweeps the idle ones, not every page
1f282c7 2026-11-24 emirduran707 Textures: linear levels are detiled a row at a time
27c3571 2026-11-24 Xavi Fortes D3D12.X: tolerate concurrent page decommits in title texture reads
69f0635 2026-11-23 theaustinayers D3D12.X: a title texture whose memory is as at its last upload is not uploaded again
7f94c6f 2026-11-23 Vax Memory: decommitted sparse tiles stay mapped until the next submit
7ebf1b8 2026-11-23 Austin Ayers Merge pull request #486 from theaustinayers/texture-skip-unchanged
b86fba4 2026-11-23 Mavixot D3D12.X: show live FPS in the game window title
9783066 2026-11-24 Frs Merge branch 'ivall:main' into title-textures-safe-read-committed
508a0c7 2026-11-23 Mavix Merge pull request #516 from Mavixot/present-fps-counter
712bbbe 2026-11-23 Dudis Title textures: the write watch scans in a few calls, not thousands
d96812a 2026-11-23 Kalmarv D3D12.X: the sparse state is replaced outside its lock
38e58e1 2026-11-24 frozht Memory: map_pages asks VirtualQuery once per run of fresh pages
2d9a9fe 2026-11-23 theaustinayers D3D12.X: ExecuteIndirect with a pipeline still being built is skipped
86a5d0b 2026-11-24 ryosen0 Tests: ExecuteIndirect's pending-pipeline skip gate
b74867b 2026-11-23 Bloxive CI: allow reviewed test-only diffs without retail smoke (#452)
9ce2070 2026-11-23 Austin Ayers Merge pull request #451 from 1984742628/perf/reuse-upload-memory
3984bf1 2026-11-24 Deividas Nastajus Title textures: detile and hash the title's memory in place, not a copy
b9f3d0f 2026-11-24 ofc0urse D3D12.X: a translated compute program built again gets the driver's blob of its first build
c3d350a 2026-11-24 0x6461726B D3D12.X: promote R9G9B9E5_SHAREDEXP render/UAV targets to R11G11B10_FLOAT
cb868b2 2026-11-23 Austin Ayers Merge pull request #300 from AAigars/kernelx-timer-resolution
3db536e 2026-11-23 Austin Ayers Merge pull request #435 from Cattn/tiling-swizzle-table-size
bdab194 2026-11-23 Mavix Merge pull request #602 from dns54/perf/title-bytes-in-place
cedf8ce 2026-11-23 Austin Ayers Merge pull request #574 from ryosen0/ei-test
372f3ab 2026-11-23 Mavix Merge pull request #302 from ZekkJJ/codex/ps-flat-integer-inputs
18c105a 2026-11-23 Mavix Merge pull request #507 from ZekkJJ/codex/test-only-pr-checks
dd2ae9a 2026-11-23 Mavix Merge pull request #636 from ofc0urse/perf/compute-pso-driver-blob
9182f27 2026-11-23 theaustinayers D3D12.X: the driver's blob of a compute program is kept on disk
63cdc25 2026-11-23 Austin Ayers Merge pull request #634 from theaustinayers/driver-pso-cache
91f9a5e 2026-11-23 Mavix Update README.md
7296a8b 2026-11-23 theaustinayers D3D12.X: pipelines an earlier run used are built ahead of their first use
9683e7a 2026-11-23 grahamster2 Scripts: smoke blocks name the game folder by a hash, not its name
a8db4b0 2026-11-23 Austin Ayers Merge pull request #690 from theaustinayers/pso-preload
82c233d 2026-11-23 Austin Ayers Merge pull request #484 from XaviFortes/title-textures-safe-read-committed
1387b4b 2026-11-23 Austin Ayers Merge pull request #348 from Kalmarv/fix/sparse-free-decommit-first
d852f9a 2026-11-23 Austin Ayers Merge pull request #443 from Kalmarv/fix/sparse-state-replace-outside-lock
72a97d8 2026-11-23 Austin Ayers Merge pull request #490 from imnicho/vram-idle-sweep
f4ce27c 2026-11-23 Austin Ayers Merge pull request #347 from Kalmarv/fix/write-watch-resume
2cc79fb 2026-11-23 Prime1997 D3D12.X: publish driver cache blobs from private staging files
4e0acbd 2026-11-23 bmjask CI: keep full symbols in release builds
64b3f35 2026-11-23 Austin Ayers Merge pull request #713 from bmjask/build/full-release-symbols
07c535d 2026-11-23 CallMeBrado Shaders: keep offline compute pipelines in the runtime driver cache
242a6ea 2026-11-23 bmjask Shaders: preserve translated cache across unrelated rebuilds
d5442d2 2026-11-23 Austin Ayers Merge pull request #762 from bmjask/shaders/semantic-translated-cache
3f54f4d 2026-11-24 gefydev README: point the Discord invite at discord.gg/xcl
6025201 2026-11-24 Austin Ayers Merge pull request #771 from gefydev/readme/discord-invite
d208e14 2026-11-23 Mavix Update README.md
066ae5b 2026-11-24 gsucristo Recompiler: default split-wave64 exchange to bounded spin
f9e5ee9 2026-11-24 gsucristo Recompiler: range-check UMA memory windows and propagate 64-bit carry/borrow
0c02b87 2026-11-24 gsucristo Recompiler: make multi-warp Compute and Mesh s_barrier early-exit-safe
8a5643f 2026-11-24 gsucristo D3D12.X: bridge Wait64BitValueX to GPU fences with exact cycle breaking
285560f 2026-11-24 gsucristo D3D12.X: split title_list command lists around mid-list Wait64 after Write64
e50836b 2026-11-24 ivall Update README.md
aaa00b2 2026-11-24 Austin Ayers Merge pull request #792 from gsucristo/recomp/uma-window-bounds-and-carry
6c98f92 2026-11-24 Austin Ayers Merge pull request #798 from gsucristo/recomp/safe-s-barrier-and-x64-spin
78cc0f6 2026-11-24 Austin Ayers Merge pull request #795 from gsucristo/d3d12x/wait64-gpu-bridge-and-cycles
beaa245 2026-11-24 Austin Ayers Merge pull request #797 from gsucristo/d3d12x/wait64-midlist-split
f1e2113 2026-11-24 Robin Snyders KernelX: a page mapped away moves to its next address directly, not by way of its home
e7da8a1 2026-11-24 theaustinayers D3D12.X: a resource's last Release waits for the queues to pass the work in flight
a1672ef 2026-11-24 Austin Ayers Merge pull request #866 from theaustinayers/late-release
dbe9324 2026-11-24 theaustinayers D3D12.X: name XCL's own tile-mapping lists and say when a wait for a queue gives up
1ec4c98 2026-11-24 Austin Ayers Merge pull request #873 from theaustinayers/queue-wait-diagnostics
03912e8 2026-11-24 Enrique Villa D3D12.X: a buffer with a mirror is bound again without VirtualQuery
5fab112 2026-11-24 Robin Snyders D3D12.X: XCL_PSO_WAIT_MS bounds how long a bind waits for a pipeline being built
11920b5 2026-11-24 theaustinayers D3D12.X: an allocator Reset keeps the recordings the title still submits
1c87b1d 2026-11-24 Austin Ayers Merge pull request #897 from theaustinayers/alloc-swap
5c3ff31 2026-11-24 theaustinayers D3D12.X: a released allocator leaves nothing behind in alloc_pool
09940bb 2026-11-24 Austin Ayers Merge pull request #907 from theaustinayers/alloc-release
5b693e6 2026-11-24 Robin Snyders KernelX: a page whose move cannot be committed stays mapped where it was
8cada26 2026-11-24 Robin Snyders D3D12.X: XCL_PSO_WAIT_MS=0 waits until the pipeline is built, as unset
71bda93 2026-11-24 Maxime Cambou D3D12.X: a sparse pool retired a moment ago is taken back (#913)
077ce40 2026-11-24 ivall Update README.md
9beaa73 2026-11-24 imnicho D3D12.X: the late-release drain gets the sparse queue without the sparse state's lock
38ea451 2026-11-24 artor410 D3D12.X: ask for tiled support before making a GPU-owned texture reserved (#940)
6922eaf 2026-11-24 ivall Merge PR #923
6bac4b4 2026-11-24 ivall Merge PR #849
e1c4020 2026-11-24 unknown Tests: cover xcl shaders across files, folders, and RPF9 packs
236e1a3 2026-11-24 ivall Merge PR #532
3ca8566 2026-11-24 ivall Merge PR #556
9aec53d 2026-11-24 ivall Merge fix: a lazy page's move keeps map_pages' runs (#556 with #849)
6522c24 2026-11-24 aarushkha PR checks: two xcl-smoke blocks in one code fence are read as two
87fae1f 2026-11-24 emirduran707 D3D12.X: GFX10 image format 134 (1_5_5_5) is B5G5R5A1, not 135 (#946)
9e80056 2026-11-24 ivall Merge PR #891
de1809c 2026-11-24 ivall Merge PR #693
afe4226 2026-11-24 ivall Merge PR #551
02bfd1d 2026-11-24 13hourstilliland Memory: link xcl.exe without high-entropy ASLR so title memory stays under 2^40 (PR #631)
ef48a18 2026-11-24 aarushkha PR checks: build inputs outside src need a regression run
6efdabc 2026-11-24 ivall Merge PR #988
c1a9b52 2026-11-24 raylitebull Memory: skip VirtualQuery on committed coherent UMA windows in cpu_writes and kernelx
8cad517 2026-11-23 Brandon Strong DirectStorage: unpack BCPack BC1, BC3, BC4, BC5, BC6H and BC7 streams
8add9a3 2026-11-24 ivall DirectStorage: BCPack provenance, README row, one Request::check again
10915eb 2026-11-24 ivall Tests: BCPack lanes against independent synthetic fixtures, models against the patent
e74dda2 2026-11-24 ivall Merge PR #995
67a3735 2026-11-24 ivall Merge PR #905
f52ec36 2026-11-24 nova-aryan D3D12.X: keep reusable allocators after a failed Reset
b55c478 2026-11-24 nova-aryan D3D12.X: stop released lists holding allocator recordings
01d4c77 2026-11-24 nova-aryan D3D12.X: move allocator recordings only after successful Reset
9a19d7e 2026-11-24 Local Test PR checks: reject untouched PR template placeholders (#965)
384f7f7 2026-11-24 ivall Merge PRs #1009, #1012, #1023 (allocator pool: failed Resets, released lists)
4f4db64 2026-11-24 raylitebull Fast-boot: opt-in Bink intro skip, frame interval override, device type and TTS mute env vars
a610eb5 2026-11-24 ivall Merge PR #990
dc6a283 2026-11-24 ivall Merge PR #1037
6828e07 2026-11-24 ivall Merge PR #951
24d8b92 2026-11-24 ivall D3D12.X: a format without tiled support is not reservable, whatever the MSAA query says
d78597d 2026-11-24 ivall Tests: truncate_existing_validation_errors reads the last error before under_wine()
47efa2e 2026-11-24 Afonso Pereia VFS: reject removal of non-empty virtual directories
21de1a8 2026-11-24 ofc0urse D3D12.X: a title's Map of a resource the PC cannot map does not reach the PC's Map
cf244b0 2026-11-24 ofc0urse D3D12.X: reserved resources are known from their creation, not asked for a heap
8f66b3b 2026-11-24 Afonso Pereia Scripts: print the right register class for root descriptors
3b6e9ea 2026-11-24 Afonso Pereia Scripts: retain every name for a PE export ordinal
7d834d4 2026-11-24 ofc0urse D3D12.X: a reserved resource is known by its address and description
de85f97 2026-11-24 Afonso Pereia Scripts: accept unprefixed command-writer branch addresses
66660bd 2026-11-24 emirduran707 DirectStorage: a refused request that is cancelled is not a failure
f4bbf26 2026-11-24 emirduran707 D3D12.X: CreatePlacedResource converts the clear value of a console format
1d83fd1 2026-11-24 emirduran707 Scripts: pr_check refuses smoke blocks without a title, time or frames
38283a8 2026-11-24 emirduran707 Scripts: pr_check refuses Code Llama, Codestral, QwQ and the like
21bc548 2026-11-24 Zohaib D3D12.X: a copy within one memory window reads through a scratch buffer
392dc0c 2026-11-24 Zohaib D3D12.X: cap and retire same-window copy scratch buffers
2e17d3d 2026-11-24 emirduran707 DirectStorage: the error record keeps the request's offset, size and tag
5bd7cd8 2026-11-24 emirduran707 DirectStorage: a new status array's slots start complete with S_OK
560216b 2026-11-24 Local Test D3D12.X: fall back to an in-memory shader cache session (#1108)
365d7c8 2026-11-24 emirduran707 D3D12.X: a pipeline stream cut inside a subobject payload is not rewritten
615cb20 2026-11-24 emirduran707 D3D12.X: default SRVs of multisampled depth buffers are Texture2DMS views
35665aa 2026-11-24 emirduran707 D3D12.X: a root signature with a huge constant count gets no layout
d6ff44c 2026-11-24 ofc0urse D3D12.X: an ExecuteIndirect dispatch of an untranslated compute pipeline is skipped, as Dispatch is
6cef69f 2026-11-24 ivall Merge PR #744
25c6f7e 2026-11-24 ivall D3D12.X: a big pipeline build the title waits for does not wait for its turn
9a272d4 2026-11-24 ivall Recompiler: a DXC compile the title waits for does not queue for memory
b8b1f49 2026-11-24 ivall Recompiler: jumps to outer loop heads and into loops are structured
010e1ec 2026-11-24 ivall Merge PR #993
930e8b4 2026-11-24 ivall D3D12.X: a new command list takes no copy scratch of an earlier one
5383506 2026-11-24 emirduran707 D3D12.X: depth+UAV buffers on the PC fallback paths start as depth writes
60e7603 2026-11-24 emirduran707 D3D12.X: allocation info of several resources is padded to their alignment
40c81c1 2026-11-24 CallMeBrado D3D12.X: prioritize partially executed lists in DRED reports
a8569bf 2026-11-24 bmjask gpu: direct DXIL emitter (bitcode, container, op library, graphics stages)
cbde110 2026-11-24 bmjask dxil: encode atomic orderings with bitcode codes (seq_cst = 6, not 7)
2815b2a 2026-11-24 bmjask dxil: derive Int64Ops / Doubles / DoubleExtensions from instruction types
9230fbc 2026-11-24 bmjask dxil: randomised differential tests of the emitter
7c672cd 2026-11-24 bmjask dxil: validate.rs FFI hardening
6b1170b 2026-11-24 bmjask dxil: mask float constant bits to the type width
746f7dd 2026-11-24 bmjask gpu/dxil: direct RDNA2 -> DXIL translator and XCL_DIRECT_DXIL routing
065aef3 2026-11-24 bmjask docs: direct DXIL main-based port status
2008399 2026-11-24 Austin Ayers Merge pull request #1137 from emirduran707/pr-check-model-names
797dda1 2026-11-24 Austin Ayers Merge pull request #1133 from emirduran707/pr-check-smoke-fields
769c7c2 2026-11-24 Austin Ayers Merge pull request #1091 from anakatt/scripts/record-writer-branch-addresses
a97e66c 2026-11-24 Austin Ayers Merge pull request #1084 from anakatt/scripts/pe-export-aliases
1d4549e 2026-11-24 Austin Ayers Merge pull request #1079 from anakatt/scripts/root-descriptor-registers
037aecd 2026-11-24 Austin Ayers Merge pull request #959 from certified-retart/tests/shaders-e2e
97d5259 2026-11-24 bmjask dxil: disk-cache direct DXIL translations under their own identity
cbea872 2026-11-24 Austin Ayers Merge pull request #968 from imnicho/late-release-sparse-queue
f09ac4c 2026-11-24 Austin Ayers Merge pull request #1330 from CallMeBrado/pr/dred-active-lists
b45a6c9 2026-11-24 bmjask dxil: test byte-identical translation across runs and threads
43e6ca5 2026-11-24 Austin Ayers Merge pull request #1224 from emirduran707/dstorage-status-slots-start-complete
cc1af70 2026-11-24 Austin Ayers Merge pull request #1223 from emirduran707/dstorage-error-record-fields
17216a1 2026-11-24 Austin Ayers Merge pull request #1127 from emirduran707/dstorage-cancel-refused
6dd40de 2026-11-24 Austin Ayers Merge pull request #1289 from emirduran707/d3d12x-fallback-stream-bounds
f538b86 2026-11-24 Austin Ayers Merge pull request #1290 from emirduran707/d3d12x-root-constants-overflow
53657c6 2026-11-24 Austin Ayers Merge pull request #1354 from emirduran707/d3d12x-allocation-info-alignment
345661b 2026-11-24 Austin Ayers Merge pull request #1353 from emirduran707/d3d12x-depth-uav-initial-state
b9789a1 2026-11-24 Austin Ayers Merge pull request #1150 from emirduran707/placed-clear-value
fc0462b 2026-11-24 Austin Ayers Merge pull request #1265 from emirduran707/d3d12x-default-srv-msaa-depth
c85ce8d 2026-11-24 Austin Ayers Merge pull request #1099 from ofc0urse/fix/map-cpu-invisible
b7aa9f3 2026-11-24 Austin Ayers Merge pull request #1211 from ksanjeev284/pr-1108-shader-cache-session
86ab4cd 2026-11-24 bmjask gpu/dxil: lower an instruction under a caller-chosen guard (Lw::inst_guarded)
942b6c7 2026-11-24 bmjask dxil: builder misuse recorded as an error instead of panicking
02ec584 2026-11-24 bmjask dxil: move DXC reference containers to tests/data/dxil
3c01233 2026-11-24 bmjask dxil tests: skip runner draws where wave64 programs cannot run
55e9c3b 2026-11-24 bmjask dxil fuzz: emit atomicrmw sub as add of the negation
ac9328f 2026-11-24 bmjask dxil: emit !llvm.ident like DXC
b6239db 2026-11-24 bmjask dxil validate: skip the validation step with a validator older than 1.8
185810f 2026-11-24 bmjask dxil: coverage sweep (xcl shaders --direct) and live fallback stats
9cdae10 2026-11-24 imnicho kernelx: Sleep does not count the time a bind waits for a pipeline build
bc93a3a 2026-11-24 bmjask Merge branch 'dxil/cache-errors' into shaders/direct-dxil-main
fe7d70b 2026-11-23 Remington Wilcox Recompiler: memory windows from the descriptor heap, no per-access switch
cad94a2 2026-11-23 Remington Wilcox D3D12.X: SetDescriptorHeaps(0, NULL) clears the list's record of bound heaps
927adcc 2026-11-23 Remington Wilcox D3D12.X: a pipeline the driver refuses with heap windows is built in the switch form
deb4b92 2026-11-24 imnicho Recompiler: a heap window's view ends 32 bytes below 4 GiB
9e3831e 2026-11-24 ivall Merge PR #680
34acfbf 2026-11-24 Kalmarv Recompiler: direct DXIL stages may branch before their exports
f372cfb 2026-11-24 Kalmarv Recompiler: direct DXIL declares waveGetLaneIndex readonly
469d8b4 2026-11-24 Kalmarv Recompiler: direct DXIL translator version 2
7d24e97 2026-11-24 bmjask gpu/dxil: write blocks in reverse post-order; Module::replace_all_uses
dc2d8f1 2026-11-24 bmjask gpu/dxil: work around AMD driver faults; check dx.op attributes against DXC
9f65f95 2026-11-24 bmjask Recompiler: avoid dynamic vector indices in format helpers
724b9e2 2026-11-24 bmjask D3D12.X: limit concurrent huge pipeline compiles
3a06645 2026-11-24 bmjask Shaders: export and measure exact compute pipelines
223311f 2026-11-24 bmjask dxil: fix VS/PS block order panics, translation time, output size, validator errors
76e6c29 2026-11-24 bmjask Merge branch 'shaders/direct-dxil-main' into dxil/sweep-fixes
424d315 2026-11-24 bmjask docs: direct DXIL coverage after the sweep fixes
2851585 2026-11-24 bmjask Merge origin/main into shaders/direct-dxil-main
4e31065 2026-11-24 bmjask D3D12.X: identify driver compiles in the PSO log
ee7f2f3 2026-11-24 bmjask README: describe the numbered driver compile log lines
def2134 2026-11-24 bmjask dxil validate: refuse input shorter than a container header
d24db9f 2026-11-24 bmjask dxil: keep this PR under GitHub's 20 000-line diff limit
6976bc4 2026-11-24 bmjask dxil: split nodes to make irreducible guest CFGs reducible; Countbits.i64 per half
ac4799f 2026-11-24 bmjask dxil: XCL_DXIL_DUMP=<hash> and xcl shaders --direct --hash/--dump
76c527b 2026-11-24 bmjask dxil: guarded regions merge only what they write (SSA region joins)
81d2ad5 2026-11-24 bmjask docs: drop the coverage write-up from this PR (diff size limit)
c160d86 2026-11-24 Ray Merge pull request #1426 from bmjask/shaders/direct-dxil-main
b07fbab 2026-11-24 bmjask dxil: geometry mode (ES+GS) in the direct translator
2dcc926 2026-11-24 bmjask dxil: Texture2DMS / Texture2DMSArray ops, MSAA image loads in the direct translator
65154b3 2026-11-24 bmjask dxil: directed f32 rounding (MODE round toward +inf / -inf / zero)
f458b55 2026-11-24 bmjask dxil: fuzz tests (random programs, direct vs HLSL on the GPU)
210f773 2026-11-24 bmjask dxil fuzz: amd_reduced_seeds (ignored) for seeds 722 / 1599; debug_seed FUZZ_WATCH
ee85bc2 2026-11-24 bmjask dxil: geometry GPU record test (loops, cuts), CfgBlock::pc for split copies, TRANSLATOR_VERSION 5
1495e6d 2026-11-24 Bloxive DirectStorage: decode eligible file requests in parallel
87d421b 2026-11-24 bmjask Recompiler: split-form geometry keeps the ES and GS phases apart with branches
5329d45 2026-11-24 bmjask Merge origin/main (direct DXIL, #1426) into perf/pso-runtime-eval
051d2c8 2026-11-24 bmjask D3D12.X: log the record hash and time only the driver's compile
4143a09 2026-11-24 bmjask Shaders: fix --export-hlsl / --measure-pso review findings
b5d6322 2026-11-24 bmjask Direct DXIL: memory windows through the descriptor heap (XCL_HEAP_WINDOWS)
69a892c 2026-11-24 Thierry Recompiler: oversized structured code falls back to pc blocks (#1307)
9039900 2026-11-24 Ray Merge pull request #1693 from bmjask/dxil/heap-windows
fa78c6f 2026-11-24 bmjask Merge origin/main (heap windows) into the direct DXIL follow-up; TRANSLATOR_VERSION 6
7ccf774 2026-11-24 Ray Merge pull request #1288 from imnicho/kernelx-sleep-hold
6c89285 2026-11-24 Ray Merge pull request #1195 from ofc0urse/fix/execute-indirect-untranslated-compute
85e3826 2026-11-24 Ray Merge pull request #1524 from bmjask/perf/pso-runtime-eval
f3242b2 2026-11-24 Ray Merge pull request #1689 from bmjask/recomp/split-geometry-fix
b9b6ea6 2026-11-24 Ray Merge pull request #819 from ZekkJJ/codex/dstorage-parallel-decompression
4c64cc4 2026-11-24 bmjask Merge remote-tracking branch 'origin/main' into shaders/direct-dxil-followup
0b64aa0 2026-11-24 raylitebull dxil: eliminate giant-shader IR and CFG blowup in FormatLoad and multi-window memory
2a2d54b 2026-11-24 bmjask dxil: geometry record test also compares the HLSL split form (fixed by #1689)
4178e88 2026-11-24 Ray Merge pull request #1748 from raylitebull/fix/direct-dxil-size-oom
7a84e7a 2026-11-24 bmjask Direct DXIL: image_get_lod with a static sampler; slowest programs in the sweep report
758e341 2026-11-24 bmjask Merge origin/main (#1748, #819); TRANSLATOR_VERSION 7
9486494 2026-11-24 wondercrash D3D12.X: an ExecuteIndirect reads the arguments written before it
1bab757 2026-11-24 gefydev Recompiler: per-lane lane-mask analysis for a direct DXIL backend
9b8db83 2026-11-24 Robin Snyders Recompiler: assemble and sign a DXIL module from LLVM IR text
b3c0b37 2026-11-24 gefydev Recompiler: lower kernel programs to DXIL-shaped IR on the lane masks
430f0a3 2026-11-24 gefydev Recompiler: scalar and vector memory in the direct lowering
f33f640 2026-11-24 gefydev Recompiler: compute mode in the direct lowering
511eadd 2026-11-24 gefydev Recompiler: write the direct lowering through gpu::dxil's module builder
ac34cb6 2026-11-24 gefydev Recompiler: refuse scalar values a lane kept from before a write it skipped
13ac196 2026-11-24 Frtuchinno Recompiler: report both halves of a stale wave64 mask in the lowering
f90033c 2026-11-24 Robin Snyders Recompiler: cross-lane operations in the lowering
ac8f8b6 2026-11-24 gefydev Recompiler: lane reads and bit counts through gpu::dxil in the direct lowering
f5d3e05 2026-11-24 gefydev Recompiler: per-lane coverage survey and the first coverage gaps
45025eb 2026-11-24 Robin Snyders Recompiler: half-local wave operations in the lowering's Ir
fbb3a7b 2026-11-24 gefydev Recompiler: name the hardware register of a refused s_setreg in the coverage survey
a5311f7 2026-11-24 Robin Snyders Recompiler: thread ids of any compute group shape in the lowering's Ir
9816a85 2026-11-24 gefydev Recompiler: drop trivial phis through Module::replace_all_uses in the DXIL adapter
6bdef80 2026-11-24 gefydev Tests: the IR assemble test skips without dxil.dll; survey sizes in KB
8593700 2026-11-24 Kalmarv Recompiler: survey every per-lane refusal, and the mesh pass both ways
f9be7cc 2026-11-24 gefydev Recompiler: an unsupported VALU opcode is refused before its sources are read
acdf922 2026-11-24 upipibi Recompiler: whole-wave mask reads need every lane where an Ir meets another wave
9231d48 2026-11-24 Vlad Bondarenko Tests: f32 programs for the per-lane lowering, on the HLSL path and per lane
ea683a0 2026-11-24 Ray Merge pull request #689 from grahamster2/scripts-smoke-title-hash
ad39e4d 2026-11-24 Robin Snyders Direct DXIL: Lw::new and Lw::finish out of the kernel translator
77db49c 2026-11-24 Robin Snyders Recompiler: the DXIL adapter holds #1426's Lw; 2-D / 3-D compute groups
878afda 2026-11-24 gefydev Recompiler: the per-lane form lowers unknown instructions with #1426's semantics
ddfbf39 2026-11-24 gefydev Recompiler: the per-lane form in the runtime behind XCL_PER_LANE=1
5963c26 2026-11-24 gefydev Recompiler: far jumps in the per-lane form
bc0d77c 2026-11-24 gefydev Recompiler: xcl shaders --direct --lanes measures the per-lane form
372be9d 2026-11-24 gefydev Recompiler: s_cselect, s_wqm and v_cmpx in the per-lane form
77a5eee 2026-11-24 upipibi Recompiler: the split wave64 form in the direct lowering, through #1426's Lw
d2cc6ff 2026-11-24 teidenzero Recompiler: the lane-mask analysis can start from an unknown EXEC
27d28ce 2026-11-24 teidenzero Recompiler: lane masks follow s_wqm of the launch EXEC
a24899e 2026-11-24 teidenzero Recompiler: VS / PS stages in the per-lane lowering
5d2d65c 2026-11-24 gefydev Recompiler: s_wqm in a VS / PS keeps the launch-EXEC rule
b1bfd1e 2026-11-24 gefydev Tests: the f32 reference keeps signed zeros on every driver
3aea7a9 2026-11-24 Vlad Bondarenko Tests: f64 programs for the per-lane lowering, on the HLSL path and per lane
82c875d 2026-11-24 Vlad Bondarenko Tests: integer programs for the per-lane lowering, on the HLSL path and per lane
188ee17 2026-11-24 gefydev Recompiler: carry-ins and v_cndmask through #1426 read the right lane bit
fe8dc84 2026-11-24 gefydev Recompiler: f16 products stay on the fallback until #1426 rounds them
27751a0 2026-11-24 Vlad Bondarenko Tests: f16 and packed programs for the per-lane lowering, on the HLSL path and per lane
3827fb5 2026-11-24 Miguel Machado Recompiler: coalesce straight-line guarded runs in the per-lane lowering
e9e94ce 2026-11-24 ivall Merge PR #1321
a23b6ff 2026-11-24 gefydev Recompiler: a value copied where some lanes are elsewhere is that block's write
dd566c0 2026-11-24 gefydev Recompiler: no r - 1 for s0 in the mask-pair check of cross-lane ops
fdb4c10 2026-11-24 gefydev Recompiler: --lanes runs the per-lane form under the sweep's timeout
e40ac38 2026-11-24 gefydev Recompiler: the runtime's split compute programs ask for 32-lane waves
14e367e 2026-11-24 teidenzero Recompiler: a VS starts with every lane in EXEC in the per-lane form
29eccc0 2026-11-24 Itsmetibo Recompiler: replace the per-lane form's trivial phis in one pass
f251762 2026-11-24 gefydev Recompiler: the per-lane form takes heap memory windows
4e057cb 2026-11-24 Miguel Machado Tests: cover split wave64 coalesced runs on the GPU
9e27370 2026-11-24 gefydev Tests: the per-lane GPU tests skip on adapters without 32-lane waves
66879ef 2026-11-24 gefydev Recompiler: the per-lane form refuses whole-wave mask decisions and modes it drops
9dcf2f6 2026-11-24 gefydev Recompiler: a mask op's SCC carried around a loop stays a lane's
bfb8133 2026-11-24 gefydev Recompiler: per-lane cache entries change when the per-lane sources do
7f7e768 2026-11-24 Ray Merge pull request #1745 from gefydev/direct-on-1426
3bc7143 2026-11-24 raylitebull D3D12.X: keep vram::Pages tile mappings contiguous per heap
266b845 2026-11-24 Ray Merge pull request #1806 from bmjask/dxil/coverage-full
177cd40 2026-11-24 Ray Merge pull request #605 from 0x6461726B/d3d12x-r9g9b9e5-target
4e3515e 2026-11-24 Ray Merge pull request #1791 from wondercrash/pr/ei-arg-order
c4d37be 2026-11-24 Ray Merge pull request #1747 from raylitebull/fix/world-freeze-180s
281fc7d 2026-11-24 bmjask Merge origin/main into shaders/direct-dxil-followup
1d027e5 2026-11-24 ivall CI: mark PRs idle for 3 days stale and close them after a week
e9ac9f0 2026-11-24 bmjask Merge origin/main (CI stale workflow)
15fff3b 2026-11-25 emirduran707 Recompiler: allow inline-constant and NULL mask operands in the per-lane form
689f2db 2026-11-24 Ray Merge pull request #1679 from bmjask/shaders/direct-dxil-followup
ecb7547 2026-11-25 emirduran707 Direct DXIL: avoid `2 - off` underflow in `s_setreg_b32` targeting `HW_REG_MODE` bit 3
fe78c81 2026-11-24 gefydev Recompiler: a float v_cmpx through #1426's lowering keeps lanes outside EXEC out
e1217e7 2026-11-25 emirduran707 Direct DXIL: truncate `s` instead of returning `native` on special values in `round_exact`
425b869 2026-11-24 raylitebull DXIL: emit MSAA textureLoad before getDimensions to avoid amdxc64 crash
438d420 2026-11-24 Kalmarv Recompiler: track the SCC of mixed mask and number operations
20cf130 2026-11-25 Ray Merge pull request #1900 from raylitebull/fix/amd-msaa-and-driver-crashes
50228bb 2026-11-25 Alp Pesen Recompiler: invalidate block-local DXIL CSE wave/quad cache across dx.op.discard
a713101 2026-11-25 Ray Merge pull request #1005 from raylitebull/pr/vq-coherent-window
f5136a6 2026-11-25 Ray Merge pull request #1053 from raylitebull/bringup/skip-intro-fast-boot
95ae589 2026-11-25 Ray Merge pull request #1876 from gefydev/per-lane-cmpx-exec
62265d3 2026-11-25 Ray Merge pull request #1866 from Kalmarv/per-lane-mixed-mask
36ac23d 2026-11-25 Ray Merge pull request #1930 from AlpPesen/fix/dxil-cse-invalidate-across-discard
fc14fab 2026-11-25 Ray Merge pull request #1864 from emirduran707/recomp-per-lane-const-mask-operands
332122d 2026-11-25 Ray Merge pull request #1869 from emirduran707/dxil-setreg-mode-bit3-underflow
8e87fc7 2026-11-25 Ray Merge pull request #1898 from emirduran707/dxil-round-denorm-inf-nan
```

## L. Limitacoes desta analise

- **Nada foi compilado nem executado.** O repositorio e Windows-only + GPU SM 6.6 (`AGENTS.md`), e a regra desta analise e somente leitura. Nenhum teste foi rodado; todos os numeros de teste citados vem das **mensagens de commit** ou dos relatos nos PRs, e estao marcados como tal.
- **O pacote de evidencias tem 43 blocos de commit, nao 45** como o cabecalho declara; 8 desses blocos sao merges sem `diff` (o gerador nao seguiu o primeiro pai). Para esses, abri o branch commit correspondente com `git log <merge>^1..<merge>^2 --format=%B` e `git show`. Pode haver commit de branch nao representado.
- **PRs e issues do GitHub foram consultados apenas para tres casos** (#123, #138, #792), que sao os que sustentam o achado central (D-01/D-02). Para todos os outros, "NÃO VERIFICADO" quando a mensagem do commit nao bastava. Nao ha acesso ao estado das issues #105, #107, #1269, #1307 e #1745 citadas nos commits.
- **`b7177b1` (PR #138) nao existe neste clone** (`fatal: Not a valid object name`); tudo que digo sobre ele vem da pagina do PR. VR #174 (linking de accessors de memoria) e listado como "Open" na pagina de #138 e **nao** aparece neste historico — nao sei se foi mergeado depois.
- **Atribuicao de "introduzido por"** usa `git log -S` e `git merge-base --is-ancestor`. Quando a string aparece em varios commits, o mais antigo topologicamente e o escolhido; em quatro casos (D-08, D-19, D-20, D-21) isso nao foi possivel e esta marcado.
- **Datas de autor nao refletem a ordem dentro dos lotes.** Varios commits de 2026-11-24 tem data de commit identica (18:00) por squash/rebase; por isso as afirmacoes de ordem usam topologia, e sinalizo quando a data engana (D-03).
- **Nao avaliei desempenho.** As afirmacoes sobre "mais rapido" (95 s -> 6 s, 148 MB -> 13.8 MB, 40 de 40 tentativas) sao citacoes de terceiros.
- **A tipologia de defeito e minha.** O repositorio nao tem taxonomia; "defeito" aqui inclui de trabalho (falso positivo de teste, OOM de ferramenta) e de produto indiscriminadamente, porque ambos tiveram custo real.
- O `AGENTS.md`/`CONTRIBUTING.md` do projeto nao aceita PRs escritos por modelos open-weight (DeepSeek incluido). Esta analise e um documento de leitura, nao uma contribuicao ao repositorio, e nenhum PR foi aberto.
