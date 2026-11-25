# xbox-emulator - analise de DeepSeek V4.1 Flash

## A. Identificacao

| Campo | Valor |
|---|---|
| Nome | xbox-emulator (README: "Xbox Emulator") |
| URL (remote `origin`) | `https://github.com/omonkeyz/xbox-emulator.git` |
| Fork de | **Nao.** Raiz `2b4bba2` e um commit independente com 3 arquivos (`README.md`, `STATUS.md`, `.gitignore`). Nenhum dos clones locais de VIctory (`VIctory-7900XTX`, `VItoriaBr`, `victory-fork`, `DexNom-VIctory`, `VIctory-Belobug-AMD`) nem do TuxBox contem o objeto `2b4bba2` (`git cat-file -e 2b4bba2...` falha em todos) |
| Branch analisada | `main`; tambem existem `origin/contrib/batch1`, `origin/contrib/batch2`, `origin/main`. 0 commits exclusivos de outras branches |
| Commit 0 | `2b4bba2` — 2026-11-21 02:08:33 -0600 — omonkey — "init: progress tracker for TomBox + rage9" |
| Ultimo commit | `a618736` — 2026-11-25 07:10:51 -0600 — omonkey — "Merge #347: tombox-xshader: size structured buffer elements from the FSSM stride (stacked on #289)" |
| Periodo coberto | 2026-11-21 a 2026-11-25 (5 dias com commit: 128 / 175 / 212 / 20 / 60) |
| Total de commits | 595 em `HEAD`, 595 em todas as refs, 104 merges |
| Arquivos versionados | 1652 (1329 em `crates/compat`, 135 em `crates/tombox-xshader`, 93 em `tools`, 14 em `docs`) |
| Autores | 32 identidades; `omonkey` 338 commits, `GDAgent Team` 46, `TQuentinD` 43, `azerioxfr` 35, `alpha` 20, `MeNoSkills` 19, `Isaac Wood` 16, `Michael Kelly` 11, `Lex_021` 15 (11+4 em duas identidades), `BED` 8, `enderrr9` 6, `nickfmbg` 3, `Bexx1107` 3, `NemesisDawn` 2, `ChAlanA` 1, `gringoestrangeiro` 1, `brcly` 1, `Louisreed` 1, entre outros |
| Commits examinados em profundidade | 45 no pacote de evidencias + ~35 adicionais abertos com `git show` / `git log -S` / `git log --follow` (total ~80, incluindo todos os merges de correcao citados) |
| Modelo / data | DeepSeek V4.1 Flash — 2026-11-25 |

### A.1 Linhagem: o que o repositorio diz, e o que o git mostra

- O README declara: *"A continuation of [drycheetah84/TomBox](https://github.com/drycheetah84/TomBox)"*, com base declarada `` `7e8064f` `` (README.md, linha 3 e secao "Credit", linha 67). **CONFIRMADA** como *afirmacao do projeto*; a ancestralidade em si e **NAO VERIFICADA** (o repositorio de TomBox nao esta disponivel localmente).
- `THIRD_PARTY/CREDITS` separa duas coisas diferentes: (a) *TomBox* de drycheetah84 — "publishes no licence", usado para decodificacao (`ds_write_b16_d16_hi`, `buffer_store_format_xy/xyz`); (b) *TomBox fork by brcly* (`brcly/TomBox @6b49c14`, "no longer reachable") — patches portados em #36; (c) *brcly/TuxBox* — explicitamente **"tools/reference/tombox-\*.patch only ... the rest of that repository is not used"**, Apache-2.0, com `THIRD_PARTY/LICENSE-TuxBox-APACHE-2.0.txt`.
- Commit real de importacao de TuxBox: `5a25b00` ("d3d12_xs: answer the Xbox Series Device8 IID from the device wrapper"), merge `baadb80` ("Merge brcly/TuxBox port: Xbox Series Device8 IID in the device wrapper (Apache-2.0)") e credito em `05f7422`. Sao **3 commits**, nao uma arvore.
- **Correcao da premissa do PortWatch:** o TuxBox clonado em `repos/portwatch/TuxBox` tem 591 commits e **1652 arquivos — exatamente a mesma contagem** do xbox-emulator, mas o conjunto de caminhos e praticamente disjunto: `Compare-Object` sobre `git ls-files` de ambos da **1645 caminhos so no xbox-emulator e 1645 so no TuxBox** (7 nomes coincidentes, do tipo `README.md`, `.gitignore`). O TuxBox e um projeto Python (`tools/` 629, `evidence/` 520, `runtime/` 340); o xbox-emulator e um workspace Rust (`crates/`). A igualdade de 1652 e **coincidencia numerica**, nao evidencia de ancestralidade (**CONFIRMADA** para a disjuncao de arquivos; **PROVAVEL** para "contagem igual = acaso").
- Portanto: xbox-emulator e uma **continuacao nominal/ideologica** do TomBox (marca `tombox-*` em todos os crates, metricas e estilo herdados de `tools/progress/gen.py`), com **reuso pontual de patches Apache-2.0** do fork brcly/TomBox e do brcly/TuxBox. Nao e fork de nenhum dos dois. **CONFIRMADA**.
- "1.849 estrelas do TuxBox", "build privado do mesmo autor liderou o ranking geral em 76%": **NAO VERIFICADO** — sem acesso a rede nesta sessao, e nenhum arquivo do repositorio registra ranking externo.

## B. Resumo tecnico

**Proposito.** Executar builds de Xbox Series X|S **ja descriptografados** como processos Win32 comuns, reimplementando a camada de plataforma do console. O README e explicito: *"The runtime reimplements the console's platform layer (Xbox-flavoured Direct3D 12, precompiled RDNA2 shaders, unified-memory textures, system libraries) so dumped, already-decrypted Xbox Series X|S builds run as normal Windows processes. It is not tied to any single title."* (README.md:5) e *"The runtime does no key extraction or package decryption."* (README.md:7).

**Arquitetura em quatro camadas** (deduzida dos caminhos e dos comentarios de modulo):

1. **Containers e carga** — `crates/tombox-pe` (parser PE sem panic), `tombox-loader`, `tombox-archive` (ZIP com `validate()` anti-zip-bomb/traversal), `tombox-rpf`, `tombox-disasm`, `tombox-cli`, `crates/tombox-xshader/src/uearchive.rs` (pak UE).
2. **Pipeline de shaders** — `crates/tombox-xshader` (429 KB so em `src/hlsl.rs`): extracao do container Xbox (`DXBC -> chunk DXIL -> XDAT -> zlib -> FSSM -> ISA RDNA2`, documentado em `src/lib.rs:1-16`), decodificador RDNA2 (`src/rdna2.rs`, `src/rdna2/mnem.rs`, tabelas `trans32`/`trans16`), inferencia de bindings (`fssm_bind.rs`, `hlsl/user_data.rs`, `hlsl/rootmem.rs`, `smem_bounds.rs`), emissao de HLSL (`hlsl.rs` + 20 submodulos: `mesh.rs`, `amp.rs`, `tess.rs`, `mimg_d16.rs`, `lds64.rs`, `mode.rs`, ...), compilacao por DXC (`dxc.rs`, SM6/DXIL; FXC SM5 como fallback) e cache por hash (`shader_cache.rs`, `pack_key.rs`).
3. **Shim grafico** — `crates/compat/d3d12_xs` (160 arquivos): reimplementa o `d3d12_xs.dll` do console **sobre o D3D12 do PC**, com remapeamento de slots de vtable (`console_slots*.rs`), traducao de PSO a partir dos *pipeline streams* do console (`mesh_pso.rs`, `pso_async.rs`), alocacao/alias de heaps e recursos colocados (`alias.rs`, `placed_offset.rs`, `msaa_place.rs`), fences/present (`present.rs`, `present_wait.rs`) e tradução de shader no bind (`shader_xlat.rs`).
4. **Bibliotecas de sistema** — ~100 crates em `crates/compat/`: `xmem` (memoria virtual do console), `xg_xs`/`xg_x`, `xcrdapi` (DirectStorage/XVD), `acphal`, `xapu`, `chad`, `xaudio2_9`, `mfplat`, `xbtp`, `xtf*`, `console-sandbox` (namespace de arquivos/registry), `kernel32legacy`/`advapi32legacy`, `gdk/*` dentro de `compat-common` (XAsync, XTaskQueue, XSystem, XUser, gamepad, saves).

**Metricas auto-declaradas** (`docs/MILESTONES.md`, geradas por `tools/progress/gen.py`): 13/18 milestones; imports de sistema 671/671; metodos D3D12.x 200/200; instrucoes RDNA2 810/820; estagios de shader 0/8 (aberto); varredura de shaders 105506/105506; bibliotecas de console 2147/2155. Sao **metricas de cobertura**, nao prova de execucao correta — o proprio `docs/BLOCKERS.md` mantem 117 linhas `- OPEN` e 5 `- PARTIAL` contra 62 `- DONE`.

**Estado de execucao** (README:47-56, com data e maquina): Fortnite (2.0.1461.7919) chega ao lobby BR, 2D correto, 3D com material errado, sem audio; Trackmania **joga** (Training - 01 a ~60 fps, campanha a 8-17 fps, texturas streamed pretas); Terraria chega ao mundo a 60 fps; Minecraft Beta vai ao menu e crasha; XDefiant <1 fps e crasha; GTA VI (1.0.2.1) **nao roda** (dump sem executavel), mas 26.503 shaders foram examinados.

**Escala de engenharia:** 595 commits em 5 dias, 1652 arquivos, 104 merges, 32 autores, ~100 crates. O ritmo e ~119 commits/dia, com pico de 212 em 23/11.

## C. Linha do tempo

**Fase 0 — nascimento como *tracker* de progresso (21/11, 02:08-~03:00).**
`2b4bba2` cria o repo com 3 arquivos e um titulo revelador: *"init: progress tracker for TomBox + rage9"* (`rage9` = nome interno da engine do GTA VI no vocabulario do projeto; ver `README` sobre GTA VI). `3f1762f` restringe o escopo ("scope to Xbox PC port only"), `dd4da0d` copia o estilo de treemap/badges do TomBox, e `7479464` **redefine o projeto como continuacao**: "reframe as a TomBox continuation covering the Xbox Series library". `2e631b5` registra os primeiros marcos de execucao (Fortnite boota, DirectStorage + Media Foundation encenados) e `0e90b75` renomeia para "Xbox Emulator". Nada de codigo ainda: o repositorio nasce como **diario de bordo publico**.

**Fase 1 — descoberta de titulos e automacao de metricas (21/11).**
`cc23333` (dump de XDefiant), `2fafb83` (Terraria, IL2CPP/GameCore), `368e0ad` (Trackmania, console OS 10.0.22621.1852) alimentam o mapa de bibliotecas. `76be322`, `b14ee89`, `fa75ffc` refinam a definicao de "PC twin"/"hooked twin". `4291a50` cria o `.gitignore` que mantem dumps fora do repo — politica que sobrevive ate o fim (`pr_gate.py` bloqueia binarios e dados de jogo no CI, `.github/workflows/pr-gate.yml`).

**Fase 2 — importacao do workspace (21/11).**
`ff39277` "Add emulator source (code only; no game data, binaries or notes)" traz a arvore Rust inteira sob `source/`; `410e01f` move para a raiz. **Todo o historico anterior a este ponto nao existe em git** — e o fato mais importante para a arqueologia deste repositorio (ver L). `9bfb4b9` remove scripts de execucao e o `STATUS.md` (que continha, entre outras, a analise do bug do WinHTTP). `57481b4` abre o projeto a contribuicoes (CONTRIBUTING, templates, licenca MIT/Apache).

**Fase 3 — rodadas de verificacao de bibliotecas de console (21-22/11).**
`7b2f86d` ("Console libs: 37 verifier defects fixed ... workspace builds again") e o primeiro grande lote de correcao. Seguem `afffefd` ("second verifier pass"), `f691d4a` ("round-2 verifier fixes"), `4ad72a4` ("round-2 fixes verified ... XTaskQueueTerminate still held"), `ba24a0b` ("all 37 verifier defects now verified fixed (XTaskQueueTerminate round 3)"). O padrao — *achar defeitos com um verificador, corrigir, re-verificar, e so entao marcar DONE* — e a assinatura metodologica do projeto. `1fc8683` ("Source sync: ISA sweep harness + 13 translator fixes") e `26ced2a` ("ISA sweep verified (169 instructions backed by exec tests)") instalam o segundo pilar: **teste de execucao real de shader**.

**Fase 4 — correcoes dirigidas por execucao (22/11).**
`46d304d` ("Trackmania placed-view device removal fixed"), `c95a909`/`7ef29e7` (registro de PSO com nomes semanticos, sem `CachedPSO`, retendo a root signature, mais `loop_guard` configuravel por `TOMBOX_XS_LOOP_GUARD`), `595eb3a` -> `5d5bc33` -> merge `fe391fe` (leitura de memoria de textura colocada por *snapshot* recuperavel de falha, "root fix for #110"), `a541bbb` (pin de modulos durante varreduras de import, corrigindo AV em `xmem+0x265e01` por corrida de unload). `ceb2b87` relicencia tudo para **GPL-3.0-or-later**. Comecam as contribuicoes externas (PRs #15, #36, #37...).

**Fase 5 — lotes "rNN" do mantenedor (22-23/11).**
Commits-mega com mensagem em prosa: `r24` `2be7211`, `r27` `d759b31` ("translator batch ... command-list PC-slot fixes (#175 + 37 more sites via cmdlist_wrap::pc_method) ... d3d12x 200/200"), `r28` `12ba607` ("16 more Xbox-vtable mis-slot fixes in injected compute paths"), `r29` `30a1b53`, `r31` `a1ff5be` (Trackmania renderiza por padrao), `r31.1` `3de1b1f` (**hotfix**: o prewarm de game-scan esgotava a memoria de titulo do Trackmania), `r32c`..`r32l` (`a7a98e0`, `e67e3d4`, `de1e1da`, `f89e410`). Nesta fase a instrumentacao (gpuprof, passcap, DRED, frame CSV) cresce tanto quanto o emulador.

**Fase 6 — a onda de PRs com conformance (23/11, 212 commits).**
O dia mais produtivo. Dois eixos:
- *Conformance contra o runtime real*: `acfede3` (17 cenarios de XSystem/XGame rodando **tambem** contra o `System32\xgameruntime.dll` 10.0.26100.9656 carregado direto com `InitializeApiImpl(0x4a61,0x2712)`), `8dce1e7`, `9b350ad`, `257bd02`, `5fdd4c9`, `883d1c8`, `e7377c9`, `c3292cc`, `f50222c`.
- *Cobertura de ISA e de container*: `#289` (tabela de bindings FSSM, merge `b17ecef`), `#257`, `#183` (samplers estaticos literais), `#240` (`ds_read_b96/b128`), `#245` (`ds_wrap_rtn_b32`), `#253` (`v_interp` VOP3), `#258` (`s_buffer_store_dword`), `#263` (`buffer_*_d16`), `#264` (DS RMW), `#282` (`image_sample_d_g16`), `#284` (scratch/private memory), `#286` (sampling bit-exato).
- *Infra de prova*: `8c481f4` (GPU-based validation opt-in `TOMBOX_D3D12_GBV=1`, que produz o oraculo que gera D-01/D-02), `#276` (testes que so falham em release), `68f2cf2` (SIB fora da janela de 8 bytes).

**Fase 7 — silicio como oraculo (24/11, 20 commits).**
`f1e8d7c` (`tools/rdna2-hwprobe/cb_probe.*`: sonda Vulkan de export/blend; conclui que RDNA2 **preserva** Inf/NaN em alvos fp16/R11G11B10, ao contrario do clamp do emulador), `bf0f50c` (sonda de selecao de mip/resinfo), `65893a4`/`3f1bc0f` (dobramento do compilador AMD em `fsub`/`f16`/`f64`), `7364bb4` (`sdst` de `v_mad_i64_i32`), `ed3cd79`, `d4c2cc5`, `0256576`, `d550bc0`, `ab6d485`. Nesta fase a fonte de verdade deixa de ser "o que o WARP faz" e passa a ser **"o que um gfx1036/RX 9060 XT faz"**.

**Fase 8 — integracao e fechamento (25/11, 60 commits).**
Sequencia de merges `#300`..`#377`: `#300` (17 cenarios XSystem), `#307` (XTaskQueue, 6 cenarios), `#309` (MODE inicial de PGM_RSRC1), `#310` (gate `console-oracle`), `#320` (blendclamp fp32), `#321` (XAsyncCancel), `#322`/`#323` (testes), `#324` (compute no rect-list), `#325` (placeholders xmem), `#326` (stand-in GameInput), `#327` (flaky alias), `#328` (M0 vs NULL), `#329` (samplers literais via FSSM), `#330` (base de user-data via FSSM), `#335` (slots 127/133), `#337` (novo IID de `IDStorageQueueX`), `#338` (flags de fila), `#346` (ByteAddressBuffer), `#347` (stride por FSSM — **ultimo commit**), `#348` (GBV), `#351` (device removal), `#360`-`#377` (sondas de hardware e fixes de ISA). O projeto termina a janela analisada no pico de maturidade de processo: PR por assunto, teste bit-exato, mutacao que derruba o teste, e merge com squashed authorship.

## D. Catalogo de defeitos

> Convencao de "commit que introduziu": quando o codigo veio no import (`ff39277` / `410e01f`), o historico anterior nao esta no git e escrevo **"import (pre-historico)"**. Nao invento autoria pre-import.

### D-01 — Elemento de StructuredBuffer dimensionado por heuristica do ISA
- **Descricao / sintomas.** O elemento de um `StructuredBuffer`/`RWStructuredBuffer` traduzido era deduzido da extensao em bytes que a ISA toca. Como o D3D12 tira o *stride* do tipo do elemento em HLSL, um chute errado desloca **todos** os elementos apos o primeiro. A GPU-based validation reporta `STRUCTURED_BUFFER_STRIDE_MISMATCH` ("desc: 44, ByteStride for struct used in shader: 48" e "desc: 32 ... 16"). No jogo: leituras deslocadas em buffers de culling/indiretos (o caso citado e o The Matrix Awakens, com 26 erros de GBV).
- **Causa raiz.** O emissor nao tinha acesso ao *stride* original do descritor; `Emitter::sbuf_words()` retornava um palpite com comentario explicito `/// A heuristic (TECHNICAL_DEBT)`.
- **Commit que introduziu.** import (pre-historico); a heuristica sobrevive ate `410e01f`.
- **Tentativas de correcao.** (a) `3e9676d` (#346, merge `36d9c51`) resolveu o subcaso dos *raw buffers* (ver D-02), mas manteve o chute para os estruturados. (b) `b03b39b` (#347, merge `a618736`, ultimo commit do repo) passou a ler o *stride* da **tabela de bindings FSSM**, quando ela existe e nomeia um unico stride multiplo de 4 para o registrador.
- **Correcao definitiva.** Parcial (`b03b39b`). Registers com strides conflitantes, strides 0/1 e shaders sem tabela legivel **mantem a heuristica**, por decisao explicita do commit.
- **Arquivos.** `crates/tombox-xshader/src/hlsl.rs` (`fssm_buffer_strides`, `Emitter::sbuf_words`), `crates/tombox-xshader/tests/fssm_bind.rs`.
- **Evidencia.** Diff de `b03b39b`: `+ if let Some(stride) = self.fssm_strides.get(&key) { return stride / 4; }` acima de `let raw = self.sbuf.get(&key)...`; testes `buffer_elements_take_the_fssm_stride` (stride 44 -> `uint w[11]`) e `conflicting_fssm_strides_keep_the_extent_heuristic`.
- **Estado atual.** Heuristica ativa como fallback. **Confianca: CONFIRMADA.**

### D-02 — Raw buffer declarado como `StructuredBuffer` com elemento de 16 bytes
- **Descricao / sintomas.** Buffers que o PSV0 marca como *raw* (`SRVRaw 4`, `UAVRaw 7`) eram emitidos como `StructuredBuffer<TbElem>` com stride adivinhado de 16 bytes, enquanto o titulo cria views raw (`StructureByteStride 0`). Todo indice era escalado pelo stride errado; a GBV acusa `desc: 0, ByteStride for struct used in shader: 16`. Caso concreto: as filas persistentes de culling Nanite no The Matrix Awakens.
- **Causa raiz.** O tipo HLSL era escolhido pela forma do acesso na ISA, ignorando o tipo de view declarado no PSV0/descritor.
- **Commit que introduziu.** import (pre-historico).
- **Tentativas de correcao.** Nenhuma anterior; `3e9676d` e a primeira.
- **Correcao definitiva.** `3e9676d` (#346, merge `36d9c51`): raw buffers passam a ser `(RW)ByteAddressBuffer`, com `Load/Store/Interlocked*` em endereco de byte, em MUBUF e MTBUF, incluindo atomics e compare-exchange. Raw com `IDXEN` passa a ser **recusado** (o stride de um descritor raw com indice nao e conhecido) em vez de adivinhado.
- **Arquivos.** `hlsl.rs`, `hlsl/atomic.rs`, `hlsl/bufref.rs` (novo `BufRef`), `hlsl/mtbuf.rs`, `tests/exec.rs`, `tests/synthetic.rs`.
- **Evidencia.** Diff de `3e9676d` (`+mod bufref;`, `BufRef::Raw(nm, byte)`), mensagem com a citacao literal do erro de GBV, e `tests/synthetic.rs` recusando raw+IDXEN.
- **Estado atual.** Corrigido. **Confianca: CONFIRMADA.**

### D-03 — Base de user-data deduzida do estagio da API (VS s8), errada em merged ES
- **Descricao / sintomas.** O caminho de root signature estatica chutava onde comecam os user data (VS s8, GS s22, MS s16, CS s4, alvo de `s_swappc` + 7). Um vertex shader *ES mergeado sem `s_swappc`* recebe user data em `s8 + slot`, nao em `s8`; o resultado e recusa ("buffer in table of kind ...") ou traducao contra o parametro errado.
- **Causa raiz.** Suposicao de layout fixo por estagio, sem consultar o cabecalho do container.
- **Commit que introduziu.** import (pre-historico).
- **Correcao definitiva.** `4fcfe13` (#330, merge `b96523a`): `fssm_user_base()` usa o slot de estagio de hardware do FSSM **somente** quando a tabela parseia e concorda com a root signature (`b.disagreements(rs).is_empty()`); caso contrario mantem o chute antigo e o retry por `s_swappc`.
- **Arquivos.** `hlsl.rs`, `tests/fssm_bind.rs` (constantes `MERGED_ES_SLOT=0x28`, `HW_VS_SLOT=0x38`).
- **Evidencia.** Diff de `4fcfe13`; testes `merged_es_vertex_shader_reads_user_data_where_the_stage_slot_puts_it` e `without_an_agreeing_table_the_static_guess_stays` (que tambem corrompe a tabela com `0xDEADBEEF...` para provar o fallback).
- **Estado atual.** Corrigido com fallback conservador. **Confianca: CONFIRMADA.**

### D-04 — `v_mad_i64_i32` escrevia o flag de overflow em `sdst`, nao o bit de sinal
- **Descricao / sintomas.** O lowering escrevia em `sdst` o flag de overflow do somatorio de 64 bits. Em RDNA2 real (gfx1036; 4096 casos aleatorios + 10 fixados) `sdst` e **o bit 64 do resultado exato de 65 bits**, isto e, o sinal: resultado negativo sem overflow seta, overflow para valor negativo de 64 bits limpa. E `bit alto enrolado XOR flag de overflow`.
- **Causa raiz.** Interpretacao errada da semantica de `sdst` no VOP3B de 64 bits.
- **Commit que introduziu.** import (pre-historico). O teste antigo **calculava o flag e o descartava** (`let ovf = ...; let _ = ovf;`).
- **Correcao definitiva.** `7364bb4` (#364, merge `cdc541d`): `((((t_h ^ ((t_ph ^ t_h) & (ch ^ t_h))) >> 31) != 0u) ? 1u : 0u)`; o teste `vector_f64` passa a conferir `sdst` com 10 vetores medidos em gfx1036.
- **Arquivos.** `crates/tombox-xshader/src/hlsl.rs`, `tests/exec.rs`, `tests/isa_sweep/ops_v.rs`.
- **Evidencia.** Diff de `7364bb4` (uma linha de HLSL trocada) + tabela `gfx1036_sign_out` com 10 tuplas `(a, b, cc, neg)`.
- **Estado atual.** Corrigido e coberto por sweep. **Confianca: CONFIRMADA.**

### D-05 — M0 (124) e SGPR_NULL (125) trocados no decode escalar
- **Descricao / sintomas.** Em GFX10.3/RDNA2 o codigo escalar **124 e M0** e **125 e NULL** (NULL so muda para 124 em GFX11+). Quatro lugares de `hlsl.rs` trocavam ou omitiam os dois: `v_readlane_b32` rejeitava 125/107/127; `lower_lane_spills` embrulhava codigos crus em `Opnd::Sgpr(d)` (gerando `s124`/`s125` inexistentes); o decoder VOP3B de `emit_f64` mapeava 124 para `Null` e rejeitava M0; e o carry-out inteiro de `v_add_co_u32`/`v_sub_co_u32`/`*_ci` nao aceitava 125.
- **Causa raiz.** Uma unica tabela de codigos escalares usada para tres contextos diferentes (dst de readlane, sdst de VOP3B, carry-out), com a numeracao de GFX11 assumida no lugar da de GFX10.3.
- **Commit que introduziu.** import (pre-historico). Dois testes existentes (`v_div_scale_f64` com `sdst=124`) **codificavam o bug**.
- **Correcao definitiva.** `9b7c93a` (ChAlanA; merge `894e339`, #328): `readlane_dst()` cobre 0..=105, VCC_LO/HI, M0 (124), NULL (125), EXEC_LO/HI; spill que mira NULL vira `v_nop`; VOP3B corrigido para 125; carry-out aceita 125 e **nao escreve** em Null. Os dois testes foram corrigidos para 125.
- **Arquivos.** `hlsl.rs`, `tests/exec.rs`, `tests/exec_vop/lanes.rs`, `tests/exec_vop/mod.rs`.
- **Evidencia.** Diff de `9b7c93a`: `-124 => Ok(Opnd::Null),` `+125 => Ok(Opnd::Null),` e `- Opnd::Vgpr(124) | Opnd::Null => return Ok(()),` substituido por `if sd == Opnd::Null { ... }`. O merge `894e339` e **1 linha deletada**, porque o resto entrou por cherry-pick.
- **Estado atual.** Corrigido, com testes de recusa e de execucao em WARP. **Confianca: CONFIRMADA.**

### D-06 — Operacoes SOP1 com resultado de 32 bits de fonte de 64 bits escreviam dois SGPRs
- **Descricao / sintomas.** `s_ff0_i32_b64`, `s_ff1_i32_b64`, `s_bcnt0_i32_b64`, `s_bcnt1_i32_b64`, `s_flbit_i32_b64`, `s_flbit_i32_i64` **leem** um par de SGPRs mas **escrevem um unico SGPR**. Quatro helpers de largura tratavam o sufixo `_b64` como destino de 64 bits e consideravam `dst+1` escrito. Consequencias: (a) a passada de simbolos apagava o que havia em `dst+1` — um ponteiro de root table copiado ali se perdia e o `s_load` seguinte era recusado ("base sN is not a root table pointer"); (b) `live_in` marcava `dst+1` como escrito, entao um SGPR de user-data lido depois **nao era semeado e lia 0**; (c) `sgpr_const_in`, `sgpr_dead_after` e o lattice de bounds de SMEM repetiam o erro.
- **Causa raiz.** Regra de largura derivada do sufixo do mnemonic em vez da semantica do opcode.
- **Commit que introduziu.** import (pre-historico).
- **Correcao definitiva.** `8f71091`: `i32_result_of_64(m)` reconhece `s_*_i32_*_b64/_i64` e os quatro sitios pulam `dst+1`; `s_bcnt1_i32_b64` passa a declarar a metade alta da fonte.
- **Arquivos.** `hlsl.rs`, `hlsl/rootmem.rs`, `hlsl/smem_bounds.rs`, `tests/exec_mem/sop1_dst.rs` (novo), `tests/exec_mem/user_data.rs`.
- **Evidencia.** Mensagem com a medicao de regressao: *"Of the 379 shaders in that title that use one of these ops, the other 359 that already translated produce byte-identical HLSL, and none fail that did not fail before."* — e o diff de `hlsl.rs` (`+self.touch(&hi(&s[0]));`).
- **Estado atual.** Corrigido; dois shaders do The Matrix Awakens (`CopyStatsCS`, `MainCS`) passaram a traduzir e compilar. **Confianca: CONFIRMADA.**

### D-07 — `DispatchMesh` lia Z do dword do ponteiro de payload: cada grupo de malha lancado duas vezes
- **Descricao / sintomas.** A entrada de args do AS no console e `{X, 0, payload VA >> 4 high, payload VA >> 4 low}`. `df91928` passou a ler Y e Z dos dwords 1 e 2. No espaco de enderecos do anel do tradutor o VA do payload e `0x20_xxxx_xxxx`, logo o dword 2 vale **2** — e todo grupo de amplificacao lancava seus grupos de malha **duas vezes**. Os pixels nao mudavam (duplicatas desenham os mesmos primitivos), entao o defeito era invisivel para os testes existentes; o custo era 2x de trabalho de GPU.
- **Causa raiz.** Generalizacao de um encoding de 1 eixo (`as3`) para 3 eixos sem fixture.
- **Commit que introduziu.** `df91928` (BED, 2026-11-22) — *"Pass the amplification shader's Y and Z group counts to DispatchMesh"*.
- **Tentativas de correcao.** `53bb998` (enderrr9, 2026-11-23) fixa Z em 1 e le Y de `tb_args[1]`. A verificacao exigiu **contar amostras**, nao comparar pixels: `render_pair()` ganhou uma query de oclusao e o teste sintetico `..._launches_each_mesh_group_once` passou a exigir 3 amostras (antes: 6).
- **Correcao definitiva.** `53bb998` (merge via branch de enderrr9; `BLOCKERS.md` item #115 marcado CHECKED).
- **Arquivos.** `crates/tombox-xshader/src/hlsl/amp.rs`, `tests/amp.rs`, `tests/synthetic.rs`, `docs/BLOCKERS.md`.
- **Evidencia.** Diff: `- DispatchMesh(tb_args[0], max(tb_args[1], 1u), max(tb_args[2], 1u), tb_pl);` `+ DispatchMesh(tb_args[0], max(tb_args[1], 1u), 1u, tb_pl);`.
- **Estado atual.** Corrigido; encoding de Y/Z > 1 continua **nao verificado** (nenhum AS de console nos titulos usa DispatchMesh 2D/3D). **Confianca: CONFIRMADA** (defeito e correcao); encoding geral **NAO VERIFICADO**.

### D-08 — Escritas byte/short no payload de amplificacao sumiam em hardware NVIDIA
- **Descricao / sintomas.** Uma escrita de byte ou short no anel de payload do AS era fundida com `InterlockedAnd`/`InterlockedOr` sobre a variavel `groupshared` do payload. Em uma RTX 4070 Ti esses atomics **nunca chegavam ao estagio de malha**: o payload mantinha o dword antigo. Os mesmos atomics nos dwords de *args* funcionavam.
- **Causa raiz.** Uso de atomics em `groupshared` para implementar um *read-modify-write* parcial, quando todas as threads do grupo executam as mesmas escritas (logo o RMW nao precisa ser atomico).
- **Commit que introduziu.** import (pre-historico); `tb_ring_put` com `InterlockedAnd/Or` ja estava em `410e01f`.
- **Tentativas de correcao.** `d2cc33e` (MeNoSkills) troca o caminho do payload por RMW simples (`tb_pl.w[k] = (tb_pl.w[k] & ~m) | (v & m);`), mantendo atomics so em `tb_args`. Depois `9101363` ("Prototype staging atomic amplification payload writes", 2026-11-24) mexe de novo no caminho — **o efeito dessa segunda mudanca nao foi verificado nesta analise**.
- **Correcao definitiva.** `d2cc33e` para o sintoma relatado. O teste `global_store_byte/short/..._d16_hi into the payload` falha sem a correcao.
- **Arquivos.** `hlsl/amp.rs`, `tests/amp.rs`.
- **Evidencia.** Diff de uma linha em `tb_ring_put`; mensagem com o adaptador e o sintoma. Suite: `tests/amp.rs` cresce com grupo 0, 4 grupos por AS, payload de 40 bytes com stride de 48.
- **Estado atual.** Corrigido para o caso medido; **dependente de driver** (o mesmo padrao passa em WARP). **Confianca: CONFIRMADA** (medicao em hardware); abrangencia em outros drivers **NAO VERIFICADA**.

### D-09 — AS escolhia o ABI de root por *memory* ou *register* conforme a posicao do primeiro `s_load`
- **Descricao / sintomas.** `rootmem.rs` so escolhia o caminho de root memory quando um `s_load` por `s[0:1]` aparecia **antes do primeiro branch**. Um AS cujo primeiro load de root vinha depois de um branch tomava o caminho de registrador, errado em tres pontos: `static_sgpr_init` punha a constante de root 0 em `s4`; `finish_amp` sobrescrevia `s4` com o group id; e o `s_load` de `s[0:1]` era descartado (SGPRs ficavam 0). Resultado: **o AS nao lancava nenhum grupo de malha e nada era desenhado, sem nenhuma recusa**.
- **Causa raiz.** Deteccao heuristica de ABI por ordem textual de instrucoes, quando o console **exige** `FORCE_MEMORY_BASED_ABI` para todo PSO com estagio de amplificacao (`mesh_pso::as_root_rule`).
- **Commit que introduziu.** import (pre-historico).
- **Correcao definitiva.** `3d90f6f` (enderrr9): `detect_rootmem` retorna `true` para `Stage::Amplification` sempre que a root signature tem parametros.
- **Arquivos.** `hlsl/rootmem.rs`, `tests/amp.rs`, `docs/BLOCKERS.md`.
- **Evidencia.** Diff de 4 linhas (`+ if self.ex.stage == Stage::Amplification { return true; }`); teste `gpu_synthetic_amplification_root_load_after_a_branch_still_uses_s0_s1` que, antes, desenhava 0 amostras.
- **Estado atual.** Corrigido. **Confianca: CONFIRMADA.**

### D-10 — O compilador AMD dobra expressoes que a ISA nao dobra (sinal de zero, Inf-Inf, trunc f64)
- **Descricao / sintomas.** Em host AMD (RX 9060 XT, `TOMBOX_EXEC_HW=1`) tres testes de execucao de `#333` falhavam, passando em WARP e NVIDIA:
  1. `v_sub_f32`/`v_subrev_f32` com `inf - inf` retornavam **0** (o DXIL e `fsub float %x, %x`, sem fast-math, e o driver ainda dobra para 0);
  2. `v_add_f16(-0, -0)` retornava **+0** (DXC baixa `x + x` como `fmul fast x, 2.0` e o AMD perde o sinal; `-Gis` nao ajuda);
  3. `v_trunc/floor/ceil/fract_f64` com modificador `neg` retornavam sinal trocado — `trunc(-1.0) = +1.0`, `floor(-1.0) = 0` — porque os helpers truncavam via `(double)(int64_t)x` e o driver descarta um `fneg` dobrado nessa conversao.
  Segunda leva: `v_add_f32` com +0 inline/literal, `v_fma/v_mad_f32` com addend +0 e `v_fmaak/v_madak_f32`/`v_fmaak_f16` com K=0 tambem perdiam o sinal de `-0`; e `v_ldexp_f16(-0, e)` retornava +0.
- **Causa raiz.** O tradutor emitia a expressao HLSL "natural" e confiava no compilador do host para preservar IEEE. O compilador **nao e obrigado** a isso quando pode provar equivalencia algebrica que ignora o sinal do zero, e o ISA do console **exige** o bit exato.
- **Commit que introduziu.** import (pre-historico) para o padrao geral; a exposicao veio com o harness de hardware (`#333`).
- **Tentativas de correcao.** `65893a4` (Isaac Wood): `tb_fsub` (NaN/inf por teste inteiro), `tb_hz`/`tb_hadd`/`tb_hsub`/`tb_hmul`. `3f1bc0f`: `tb_trunc64` por mascara de bits (sem conversao para int64), `tb_pz`/`tb_pzf`, `tb_hz` **setando** o sinal em vez de OR-ar, `v_ldexp_f16` preservando o sinal do zero. Merge `d0fabcb` (#375).
- **Correcao definitiva.** `65893a4` + `3f1bc0f` (merge `d0fabcb`). Falhas do sweep de hardware caem de 11 mnemonicos para 3.
- **Arquivos.** `crates/tombox-xshader/src/hlsl.rs` (helpers `tb_*` e emissores VOP1/VOP3/VOP3P), `tests/exec_vop/amd_fold.rs` (novo), `tests/isa_sweep.rs`.
- **Evidencia.** Diff de `3f1bc0f`: `float tb_trunc64(double x) { uint l, h; asuint(x, l, h); int e = (int)((h >> 20) & 0x7ffu) - 1023; ...}` — substitui a conversao `(double)(int64_t)x`. Teste `amd_host_zero_sign_and_f64_neg` (48 casos); na RX 9060 XT 23 casos falham sem a mudanca.
- **Estado atual.** Corrigido, com **residuo declarado**: `v_add/sub/mul_f16` com SDWA byte-select + `abs` em src0 continua devolvendo 0 no AMD em compute shader (e passa em pixel shader) — **SEM CORRECAO** (`3f1bc0f`, secao "Still failing on AMD"). **Confianca: CONFIRMADA.**

### D-11 — `cl_edition`: leitura de um byte alem da janela de 8 bytes no scan de `call` com SIB
- **Descricao / sintomas.** `after_call(code: &[u8; 8])` lia `code[c - k + 2]` para um candidato de dois bytes cujo ModRM precisa de SIB (`FF 14` no fim da janela) — **um byte alem dos oito** que recebeu — e entrava em panic ("index out of bounds: the len is 8 but the index is 8"). O panic ocorre no caminho que identifica a **edicao GXDK do modulo chamador** para rotear os slots 131/132 de command list, ou seja, em pleno despacho de comandos do titulo.
- **Causa raiz.** Calculo de comprimento de instrucao que nao valida se os bytes existem antes de indexar, num buffer de tamanho fixo.
- **Commit que introduziu.** `6d19943` (GDAgent Team, 2026-11-23), que criou `cl_edition.rs`.
- **Correcao definitiva.** `68f2cf2` (GDAgent Team, 2026-11-24; merge `d68b34a`, #334): `if sib == 1 && k < 3 { continue; }`. "Reported by RomainROCH in review."
- **Arquivos.** `crates/compat/d3d12_xs/src/cl_edition.rs`.
- **Evidencia.** Diff de 5 linhas + dois asserts novos: `assert!(!t(&[0xff, 0x14]));` e `assert!(t(&[0xff, 0x90, 0, 0, 0xff, 0x14]));` (o segundo aceita `call [rax+disp32]`).
- **Estado atual.** Corrigido. **Confianca: CONFIRMADA.** (Nota: o panico so ocorre quando o candidato aparece nos ultimos bytes da janela, por isso sobreviveu a revisao inicial.)

### D-12 — Build limpo quebrado: `mod oracle` declarado sem arquivo; CI cego ao problema
- **Descricao / sintomas.** `crates/compat/xg_xs/src/lib.rs` declara `#[cfg(test)] mod oracle;`, mas `src/oracle.rs` **nunca existiu no repositorio**. Em checkout limpo, `cargo check -p xg_xs --all-targets` falha com `E0583` (arquivo nao encontrado), e como `oracle` define `Obj`, `field`, `first_diff`, `vt`, `Create`, `Oracle` e `SERIAL`, os sete modulos de teste que importam dele tambem falham: **15 erros** e **19 testes unitarios inexequiveis**, inclusive os que nao precisam de dados de console.
- **Causa raiz.** Fixture de console referenciada por caminho que nao foi versionado, mais um gate de CI que nao cobre o alvo: `rust.yml` roda `cargo check --workspace` **sem** `--all-targets` para os crates de compat, e o unico job que usa `--all-targets` nomeia cinco crates e **nao inclui `xg_xs`**.
- **Commit que introduziu.** import (pre-historico).
- **Tentativas de correcao.** `f10db75` corrige a **mesma classe** em outro crate ("gcxaudioext: publish acp_state module (fixes clean-clone build E0583)").
- **Correcao definitiva.** `706a2b7` (Bexx1107; merge `2813b2a`, #310): feature `console-oracle` **off-by-default**, exigida em conjunto com `test` para os oito modulos oracle e para `dcc::watch`; `dcc.rs` ganha `#![cfg_attr(not(feature = "console-oracle"), allow(dead_code))]`.
- **Arquivos.** `crates/compat/xg_xs/Cargo.toml`, `src/lib.rs`, `src/dcc.rs`.
- **Evidencia.** Diff: `-#[cfg(test)]` `+#[cfg(all(test, feature = "console-oracle"))]` em oito sitios; mensagem com "15 errors total, and all 19 unit tests ... unrunnable" e a descricao exata do furo de CI.
- **Estado atual.** **Parcialmente corrigido.** O sintoma sumiu, mas a causa sistemica **permanece**: em `main` (`a618736`) o job `rust-workspace-check` de `.github/workflows/rust.yml` continua com `continue-on-error: true` e roda `cargo check --locked --workspace --exclude ...` sem `--all-targets`. Ver G-09. **Confianca: CONFIRMADA.**

### D-13 — Flags de fila so-do-console repassadas ao D3D12 do PC: `E_INVALIDARG`
- **Descricao / sintomas.** `CreateCommandQueueX` com flags 2 e 3 (filas de compute) era encaminhada com as flags do console intactas; o runtime do PC recusava com `E_INVALIDARG`. Medido em RDNA2/Windows 11 nativo (nao sob Proton).
- **Causa raiz.** Traducao de estrutura copiava tipo/prioridade/no mas **nao** mascarava `Flags`; a unica flag valida no PC e `DISABLE_GPU_TIMEOUT` (bit 0).
- **Commit que introduziu.** import (pre-historico).
- **Correcao definitiva.** `979615b` (GDAgent Team; merge `9a1e9ba`, #338): `const PC_QUEUE_FLAGS: u32 = 0x1;` e `put(d, 8, pc_flags)`.
- **Arquivos.** `crates/compat/d3d12_xs/src/xone_queue.rs`.
- **Evidencia.** Diff: `+let pc_flags = flags & PC_QUEUE_FLAGS;` `+put(d, 8, pc_flags);` e o log passa a imprimir `flags={flags:#x} ... flags={pc_flags:#x}`; teste cobre 0, 1, 2, 3 e um bit alto.
- **Estado atual.** Corrigido. **Confianca: CONFIRMADA.**

### D-14 — Blendclamp "eager" prendia alvos `R32G32B32A32_FLOAT` a ±65504
- **Descricao / sintomas.** `small_float()` listava o formato DXGI **2** (`R32G32B32A32_FLOAT`) junto com os fp16, entao o passe de clamp aplicava ±65504 — faixa de fp16, nao de fp32 — a **todo alvo fp32 com blend**. Medido: tres draws aditivos de 40000 + 40000 + 1 em alvo RGBA32F davam `65504.0` (0x477fe000) no modo eager, contra `80001.0` correto.
- **Causa raiz.** Constante de faixa escolhida por "e um formato de float", sem distinguir 16 de 32 bits. O modo `ps` (`shader_xlat::export_kind`) ja excluia o formato 2 — a inconsistencia estava registrada em `BLOCKERS.md` como "looks unintended".
- **Commit que introduziu.** import (pre-historico).
- **Correcao definitiva.** `ced7994` (enderrr9; merge `92c998d`, #320): `small_float` passa a listar 10/34/54 (fp16) e 26 (R11G11B10).
- **Arquivos.** `crates/compat/d3d12_xs/src/blendclamp.rs`, `blendclamp_hw.rs`, `docs/BLOCKERS.md`.
- **Evidencia.** Diff: `- 2 | 10 | 34 | 54 => Some(true),` `+ 10 | 34 | 54 => Some(true),`; teste WARP `warp_blendclamp_leaves_fp32_targets_above_the_fp16_range` com `bpp()` corrigido para 16 no formato 2 e `assert_eq!(passes, 0, ...)`.
- **Estado atual.** Corrigido no modo eager. O modo eager **continua menos exato que o console** para Inf/NaN de shader — ver G-04. **Confianca: CONFIRMADA.**

### D-15 — Remocao de device D3D12 era interpretada como fence concluida
- **Descricao / sintomas.** `wait_fence` tratava `GetCompletedValue() >= value` como sucesso. Em remocao de device o runtime devolve `UINT64_MAX` na fence, satisfazendo **qualquer** comparacao: a espera retornava sucesso e o codigo seguinte usava um device morto. A remocao tambem acorda o evento de espera, entao o caminho por evento tinha o mesmo erro.
- **Causa raiz.** Ausencia do valor sentinela de remocao no predicado de sucesso.
- **Commit que introduziu.** import (pre-historico).
- **Correcao definitiva.** `d550bc0` (Lex_021; merge `03fea26`, #351), com follow-up de estilo `bd39bf8`: todo caminho passa por `current != u64::MAX && current >= value`.
- **Arquivos.** `crates/compat/d3d12_xs/src/present_wait.rs` (novo `mod tests` com vtable sintetica).
- **Evidencia.** Diff: `- if completed(fence) >= value { return true; }` `+ let current = completed(fence); if current >= value { return current != u64::MAX; }`; teste `device_removal_does_not_complete_a_fence` que injeta `u64::MAX` no `SetEventOnCompletion`.
- **Estado atual.** Corrigido; validado **so por teste unitario** (vtable falsa), nao em titulo. **Confianca: CONFIRMADA** (defeito), execucao real **NAO VERIFICADA**.

### D-16 — `GameInputCreate` falhando devolvia ponteiro nulo para um titulo que nao checa HRESULT
- **Descricao / sintomas.** No console `GameInputCreate` **sempre** sucede, e os titulos usam o `IGameInput` devolvido sem conferir o HRESULT. Em host sem servico GameInput (VM limpa, Windows Sandbox, onde `GameInputSvc` para em `START_PENDING`), a chamada real devolve `0xd0000034`; o titulo entao chama um metodo na interface nula e **falha por access violation**.
- **Causa raiz.** Encaminhamento literal ao host de uma API cujo contrato no console e "nunca falha".
- **Commit que introduziu.** import (pre-historico).
- **Correcao definitiva.** `146286e` (GDAgent Team; merge `9af87c0`, #326): em falha, retorna `S_OK` com um objeto **stand-in** estatico de 22 metodos, cada um com comportamento definido (`GetCurrentTimestamp` via QPC em microssegundos; leituras -> `GAMEINPUT_E_READING_NOT_FOUND` com reading nulo; registros -> token unico com `E_INVALIDARG` para token nulo; `StopCallback`/`UnregisterCallback` esquecendo apenas tokens emitidos; `CreateDispatcher`/`CreateAggregateDevice` -> `E_NOTIMPL`; `FindDevice*` acha o pad virtual, resto `E_NOTFOUND`; `QueryInterface` responde a qualquer IID). O objeto passa pelo mesmo `patch_object` do objeto real, entao o pad virtual, o canal de controle e a ponte de pad do host se acoplam igual. Se um objeto real ja foi patchado, o stand-in nao e adotado e o HRESULT original e devolvido.
- **Arquivos.** `crates/compat/compat-common/src/gamepad.rs` (+246 linhas).
- **Evidencia.** Diff mostra `sd_qi`, `sd_timestamp`, `sd_reading4`; `now_us` foi dividido para `qpc_us` "so the stand-in timestamp does not recurse".
- **Estado atual.** Corrigido; 22 testes de gamepad passando (declarado). **Confianca: CONFIRMADA.**

### D-17 — `XGameGetXboxTitleId` com codigo e ordem de checagem diferentes do runtime do PC
- **Descricao / sintomas.** Sem `TitleId` (processo sem identidade de pacote), o runtime real devolve `E_NOT_FOUND` (0x80070490) e **nao toca** no ponteiro de saida; devolve `E_NOT_FOUND` tambem com ponteiro nulo. O emulador devolvia `E_FAIL`, e `E_INVALIDARG` para saida nula.
- **Causa raiz.** Ordem de validacao invertida (checava o ponteiro antes do titulo) e codigo de erro generico.
- **Commit que introduziu.** import (pre-historico).
- **Correcao definitiva.** `c3292cc` (TQuentinD; merge `fb95d74`, #300): `let Some(id) = title().title_id else { return E_NOT_FOUND };` **antes** da checagem de `out.is_null()`.
- **Arquivos.** `crates/compat/compat-common/src/gdk/system.rs`, `tests/gdk_xsystem_conformance.rs` (lista `DIFFERS` esvaziada).
- **Evidencia.** Diff mostra a inversao exata e `const DIFFERS: &[(&str, &str)] = &[];`. Medicao declarada em `xgameruntime.dll 10.0.26100.9656` com os dois IIDs (`50849859`, `2549f142`).
- **Estado atual.** Corrigido; a tabela emulada nao tem diferenca restante. **Confianca: CONFIRMADA.**

### D-18 — `XAsyncCancel`/`XAsyncGetResult` divergindo do `xgameruntime.dll`
- **Descricao / sintomas.** Quatro divergencias, cada uma com cenario de conformidade rodando nos dois runtimes:
  1. `XAsyncCancel` **so** chama a operacao `Cancel` do provider; a chamada continua pendente ate o provider completa-la, e o `DoWork` agendado ainda roda. O emulador completava com `E_ABORT` e pulava o proximo `DoWork`. E `Cleanup` nao roda dentro da operacao `Cancel`: roda quando `XAsyncCancel` retorna (sem callback) ou depois do callback de conclusao.
  2. Trabalho cancelado por fila em terminacao chama a operacao `Cancel` primeiro; o callback cancelado chama `XAsyncCancel` e so completa com `E_ABORT` se ainda pendente. O emulador completava sem chamar `Cancel`.
  3. Bloco **nunca iniciado**: um bloco zerado le `S_OK` de `XAsyncGetStatus` (com ou sem wait), `XAsyncGetResultSize` = 0 e `XAsyncGetResult` usado = 0; um bloco cujo campo de status le `E_PENDING` devolve `E_PENDING` sem wait e `E_INVALIDARG` com wait. O emulador devolvia `E_INVALIDARG` para todos.
  4. `XAsyncGetResult` exige a **identidade exata** dada a `XAsyncBegin` (nulo incluido); identidade errada, buffer nulo (`E_INVALIDARG`) e buffer curto (`E_NOT_SUFFICIENT_BUFFER`) **encerram a chamada** (Cleanup roda e uma retentativa correta da `E_ILLEGAL_METHOD_CALL`). O emulador mantinha a chamada viva e deixava a retentativa funcionar.
- **Causa raiz.** Modelo de ciclo de vida implementado "por intuicao" em vez de medido; o proprio commit registra que a fonte foi `xgameruntime.dll 10.0.26100.9656` e `microsoft/libHttpClient @c5007fd9` (`Source/Task/AsyncLib.cpp`), sem codigo copiado.
- **Commit que introduziu.** import (pre-historico), com tentativas anteriores em `883d1c8` (#266, "XAsync return codes match the reference") e `8dce1e7`/`9b350ad`/`257bd02`/`5fdd4c9`.
- **Correcao definitiva.** `e7377c9` (TQuentinD; merge `8050509`, #321) para os quatro itens. **Nao corrigido no mesmo commit:** `verify_terminate_waits_for_async_in_flight` "passes on the PC runtime and still fails here; the fix belongs in queue termination".
- **Arquivos.** `crates/compat/compat-common/src/gdk/rt.rs`, `tests/support/xtq.rs`, `tests/gdk_minecraft_slots.rs`, `tests/gdk_xtaskqueue_conformance.rs`.
- **Evidencia.** Diff mostra `canceled: AtomicBool` -> `in_cancel` + `cleanup_after_cancel`, `raw_status(block)` lendo `internal[8..12]`, e `E_PENDING if wait => E_INVALIDARG`. Cada fix tem uma **mutacao** que faz o cenario falhar.
- **Estado atual.** 4 itens corrigidos; 1 cenario ainda falha (G-06). **Confianca: CONFIRMADA.**

### D-19 — `XMemVirtualAlloc` descartava `MEM_RESERVE_PLACEHOLDER` / `MEM_REPLACE_PLACEHOLDER`
- **Descricao / sintomas.** O tipo de alocacao era mascarado para as flags classicas de `VirtualAlloc`, perdendo `MEM_RESERVE_PLACEHOLDER` (0x40000) e `MEM_REPLACE_PLACEHOLDER` (0x4000). Um titulo que gerencia os proprios placeholders (por exemplo para memoria de GPU) reserva um range, divide com `VirtualFree(MEM_RELEASE | MEM_PRESERVE_PLACEHOLDER)` e substitui cada pedaco com `XMemVirtualAlloc(MEM_RESERVE | MEM_REPLACE_PLACEHOLDER | 64K pages)`. Sem as flags, a primeira chamada virava reserva comum, **a divisao falhava e a inicializacao do chamador abortava**.
- **Causa raiz.** Traducao de `AllocType` por whitelist de flags, sem `VirtualAlloc2` nem conhecimento de placeholders.
- **Commit que introduziu.** import (pre-historico).
- **Correcao definitiva.** `2e8e2b8` (GDAgent Team; merge `785a6df`, #325): `placeholder_alloc_type()`, `placeholder_alloc()` via `VirtualAlloc2` no processo corrente, `PAGE_NOACCESS` obrigatorio para reserva de placeholder, contabilidade de commit preservada.
- **Arquivos.** `crates/compat/xmem/src/virt.rs`, `src/phys.rs`.
- **Evidencia.** Diff de 65 linhas; teste `title_placeholder_reserve_split_replace_and_commit` faz reserva/dividir/substituir/commitar e **escreve** nas paginas commitadas.
- **Estado atual.** Suite xmem 89/89 (2 ignorados) em Windows 11 nativo. O proprio commit avisa: *"Not yet verified past this point in a title."* **Confianca: CONFIRMADA** (defeito e teste), uso em titulo **NAO VERIFICADO**.

### D-20 — `console-sandbox::fs::resolve` panicava em caminho com caractere multi-byte
- **Descricao / sintomas.** Em `try_resolve`, o teste de prefixo UNC era `r[..4].eq_ignore_ascii_case("UNC\\")` sobre `&str`. Se o caminho apos `\\?\` ou `\??\` tiver um caractere multi-byte cruzando o byte 4 (por exemplo `\\?\aé€x`), o *slice* cai fora de fronteira de caractere e o processo **entra em panic**. O caminho e controlado pelo titulo (namespace de arquivos do sandbox).
- **Causa raiz.** Indexacao por byte em `&str` sem validar fronteira.
- **Commit que introduziu.** import (pre-historico).
- **Correcao definitiva.** `0284e83` (omonkey, 2026-11-21): comparacao por bytes — `r.as_bytes().get(..4).is_some_and(|p| p.eq_ignore_ascii_case(b"UNC\\"))`.
- **Arquivos.** `crates/compat/console-sandbox/src/fs.rs`, `src/tests.rs`.
- **Evidencia.** Diff de uma linha + quatro asserts novos, entre eles `fs::try_resolve("\\\\?\\aé€x") == Err(STATUS_OBJECT_PATH_NOT_FOUND)` e `"\\??\\é"`; o alfabeto do fuzz (`fuzzed_console_paths_never_leave_their_drive`) ganhou `é`, `€`, `\u{FF1A}`, `\u{2215}`, `\u{FF3C}`.
- **Estado atual.** Corrigido. **Confianca: CONFIRMADA** para o panic. **Ressalva:** a mensagem do commit fala em *"mid-path drive-letter escape"*, mas o diff **so** altera o teste de prefixo — a parte "escape" **nao** aparece neste commit nem em outro commit de `fs.rs` entre `410e01f` e `HEAD` (`git log -- crates/compat/console-sandbox/src/fs.rs` lista apenas `0284e83` e os anteriores). A ligacao entre a mensagem e o codigo e **PROVAVEL**, nao confirmada.

### D-21 — `tombox-archive::validate` aceitava entradas com nomes colidentes
- **Descricao / sintomas.** `validate()` conferia cada nome do diretorio central isoladamente, entao um arquivo podia carregar duas entradas que mapeiam para **o mesmo caminho em filesystem case-insensitive** (por exemplo `Game/Content/a.pak` e `game/content/A.PAK`) e ainda reportar "seguro para extrair". A extracao mantem a ultima, que **nao e o inventario que o chamador auditou**.
- **Causa raiz.** Validacao sem estado entre entradas; normalizacao (separador, barra final, caixa) ausente.
- **Commit que introduziu.** import (pre-historico).
- **Correcao definitiva.** `cce78b6` (gringoestrangeiro; merge `bb15e8e`, #244; `cherry picked from 7516c2d2...`): `BTreeMap<String, (String, bool)>` de nomes normalizados; colisao vira achado **bloqueante**; diretorios repetidos continuam permitidos (nao sobrescrevem dados).
- **Arquivos.** `crates/tombox-archive/src/lib.rs`, `src/tests.rs`.
- **Evidencia.** Diff: `let key = norm.trim_end_matches('/').to_lowercase();` + `block(&mut r, name, &format!("entry name collides with '{previous}' ..."))`.
- **Estado atual.** Corrigido. **Confianca: CONFIRMADA.**

### D-22 — Teste "flaky" era sintoma de comportamento assincrono real do Windows (e o problema de runtime ficou aberto)
- **Descricao / sintomas.** `alias::tests::title_heap_does_not_block_unmap_or_free` falhava ~1 em 4 execucoes, no `VirtualFree` imediatamente apos `flush()`. Diagnostico: o flush **termina**, mas o Windows libera o pin sob um heap `OpenExistingHeapFromAddress` **depois** do ultimo `Release` do heap. Em D3D12 puro, sem codigo do emulador, isso acontece em todos os quatro adapters testados (Radeon iGPU, RTX 3050, Basic Render, WARP): 0-1% de falhas em device ocioso, **24-55%** com um loop de copia de 64 MB, sempre resolvendo em ate 50 ms. Os testes de lib paralelos compartilham o device singleton — dai o "1 em 4".
- **Causa raiz.** Premissa falsa de que `VirtualFree` reflete sincronamente o fim do pin; teste sem referencia propria ao heap.
- **Correcao definitiva.** `ab6d485` (enderrr9; merge `c9deb12`, #327): o teste segura a propria referencia do heap, checa exatamente o lado do alias (`spans_starting_at(p) == 0`, `Release` do heap == 0) e **depois** espera ate 10 s pelo unpin, com polling. Resultado medido: teste antigo 42/200 falhas, novo 0/200, sob a mesma carga; um mutante que nunca libera heaps ociosos falha 10/10.
- **Arquivos.** `crates/compat/d3d12_xs/src/alias.rs`, `docs/BLOCKERS.md`.
- **Evidencia.** Diff do teste + a entrada BLOCKERS reescrita de `OPEN, FLAKY` para `DONE, FLAKY`, com uma **nova** linha `OPEN, not seen in a title run`: o mesmo atraso de unpin se aplica em runtime com `TOMBOX_ALIAS_TITLE=1`, e um `VirtualFree` do titulo emitido em ~50 ms apos o release do ultimo buffer aliasado **pode falhar**, porque o wrapper do xmem passa direto.
- **Estado atual.** Teste corrigido; **consequencia de runtime aberta** (G-07). **Confianca: CONFIRMADA.**

## E. Relacao de causa e efeito

1. **`ff39277`/`410e01f` (import do workspace) -> todo o codigo nasce sem historia -> 19 dos 22 defeitos acima ficam com "introduzido no import"** -> a arqueologia so consegue datar o *surgimento*, nao a *autoria*, e varios defeitos aparecem "corrigidos" em commits cuja mensagem descreve outro assunto (ver E-7).
2. **`8c481f4` (GBV opt-in) -> a GPU-based validation passa a acusar `STRUCTURED_BUFFER_STRIDE_MISMATCH` -> `3e9676d` (#346) corrige o subcaso raw e `b03b39b` (#347) le o stride do FSSM -> D-01/D-02.** Cadeia exemplar: a ferramenta de diagnostico e o que revela o defeito; sem ela, os dois viveram desde o import sem sintoma visivel em WARP.
3. **`b17ecef` (#289, tabela de bindings FSSM) -> habilita `4fcfe13` (#330, base de user-data), `65e8691`/`c59666e` (#329, samplers literais) e `b03b39b` (#347, stride)** -> tres defeitos de inferencia resolvidos pela mesma fonte de verdade. O FSSM e o "metadado que faltava" e a razao pela qual #289 foi pre-requisito de tres correcoes.
4. **`df91928` -> Z=2 no `DispatchMesh` -> duplicacao invisivel de grupos de malha -> `53bb998`** -> a correcao so foi possivel porque `render_pair()` passou a medir **amostras de oclusao**; a comparacao de pixels era cega ao defeito. Consequencia metodologica: testes de estagio grafico precisam de contadores, nao so de imagem.
5. **`6d19943` (novo `cl_edition`) -> `after_call` indexa 1 byte alem do buffer -> `68f2cf2` corrige com `if sib == 1 && k < 3 { continue; }`** -> e o padrao "codigo novo de parsing binario nasce com leitura fora de limite, revisao humana pega (RomainROCH)". O mesmo padrao aparece em `0b4583d` ("bound RVA arithmetic in directory parsers to uphold the no-panic invariant") em `tombox-pe`.
6. **`#333` (harness de execucao em hardware AMD) -> expoe o dobramento do compilador -> `65893a4` -> `3f1bc0f` -> merge `d0fabcb`** -> a segunda rodada e necessaria porque a primeira so cobriu `fsub` e `f16`; o sweep de hardware cai de 11 para 3 mnemonicos. **Cadeia que nao fecha:** o residuo SDWA/abs em `f16` fica declarado como falha conhecida.
7. **`c95a909` e `7ef29e7` carregam a mesma descricao de trabalho** ("Exec mask ... Fortnite derived PSO use-after-free") **mas o diff de `c95a909` e 100% `xtfeventserver`** (COM, ETW, dispatcher) e o de `7ef29e7` e `loop_guard` + PSO. Consequencia: **a mudanca de exec mask nao e datavel por mensagem**; `git log -S"wqm"` em `hlsl.rs` so encontra o import. Classificacao: **CONFIRMADA** a inconsistencia mensagem/diff; a mudanca em si **NAO VERIFICADA** quanto a autoria.
8. **`7b2f86d` -> `afffefd` -> `f691d4a` -> `4ad72a4` -> `ba24a0b`** (quatro rodadas de verificacao de bibliotecas de console) -> o padrao "corrigir, re-verificar, so entao marcar DONE" que sustenta `BLOCKERS.md` como registro confiavel (62 DONE contra 117 OPEN).
9. **`cce78b6` (#244) e `0b4583d`** -> endurecimento de parsers de entrada nao confiavel (ZIP, PE) -> o projeto trata dado de dump como hostil, o que e coerente com `pr_gate.py` recusando binarios no repo.
10. **`ceb2b87` (relicenca GPL-3.0) -> `THIRD_PARTY/CREDITS` passa a ser o unico registro de proveniencia** -> qualquer codigo de origem sem licenca clara (TomBox) vira risco juridico registrado, nao tecnico (G-10).

## F. Correcoes importantes

**F-01 — Ler o *stride* do elemento na tabela de bindings do container (FSSM).** *Problema:* o tipo HLSL do buffer era adivinhado pela extensao que a ISA toca, e o D3D12 deriva o stride do tipo, deslocando todos os elementos. *Como foi corrigido:* `fssm_buffer_strides()` parseia a tabela, exige `stride >= 4`, `stride % 4 == 0`, `register <= 0xffff`, e **descarta** registradores com strides conflitantes. *Por que funciona:* o stride do descritor original e exatamente o que o D3D12 vai reconstruir a partir do tipo HLSL; casar os dois elimina o mismatch na origem. *Dependencias:* `fssm_bind::parse`, `#289`. *Reutilizavel:* sim — a regra "derivar layout de buffer do metadado do container, nunca do padrao de acesso" vale para qualquer tradutor de ISA para API tipada. *Risco:* quando a tabela falta ou conflita, a heuristica volta e o erro volta silenciosamente; o ideal e **recusar** em vez de adivinhar (o projeto faz isso para raw+IDXEN, mas nao aqui). `b03b39b`, `hlsl.rs`.

**F-02 — `ByteAddressBuffer` para buffers raw, com recusa de `IDXEN`.** *Problema:* view raw (stride 0) lida como estruturada de 16 bytes. *Implementacao:* `BufRef::{Elem,Raw}` (`hlsl/bufref.rs`), endereco de byte em MUBUF/MTBUF, `Interlocked*` no endereco, `Load/Store` para sub-dword com mascara. *Por que funciona:* um buffer raw nao tem elemento; a unica unidade correta e o byte. *Reutilizavel:* sim, integralmente — e o mesmo raciocinio que qualquer camada de traducao D3D11/D3D12 precisa. *Risco:* `IDXEN` em raw passa a ser recusado, o que troca corrupcao silenciosa por falha alta; e o trade-off certo. `3e9676d`.

**F-03 — Guardas de IEEE contra o compilador do host (`tb_hz`, `tb_pz`, `tb_pzf`, `tb_fsub`, `tb_trunc64`).** *Problema:* o compilador do host dobra `x - x`, `x + 0.0` e `-0`, e descarta `fneg` dobrado em conversao `double(int64)`. *Implementacao:* helpers que reconstroem o resultado por operacoes inteiras (`asuint`, mascara de fracao, teste de classe NaN/inf) e forcam `+0` onde a ISA exige. `tb_trunc64` trunca por mascara de bits do expoente, sem passar por `int64`. *Por que funciona:* o resultado deixa de depender de provas algebricas que ignoram o bit de sinal do zero; o teste passa a ser bit-exato em WARP, NVIDIA e AMD. *Reutilizavel:* **sim, e um dos achados mais valiosos do repositorio** — qualquer tradutor ISA->HLSL/DXIL que emita aritmetica "natural" herda esse bug. *Risco:* custo de instrucoes; o projeto aplica so onde o caso e alcancavel (`pz` condicionado a operando zero). `65893a4`, `3f1bc0f`, `d0fabcb`.

**F-04 — `readlane_dst()`: uma funcao por contexto de codigo escalar.** *Problema:* o campo de destino de 8 bits tem significado diferente em `v_readlane_b32` (SGPR/M0/NULL/EXEC), em `sdst` de VOP3B e no carry-out inteiro. *Implementacao:* um decodificador unico, com a numeracao de GFX10.3, e recusa alta para o que nao esta mapeado. *Por que funciona:* elimina as tres tabelas divergentes. *Reutilizavel:* sim — o padrao "um decodificador por classe de campo, nao por instrucao" e o que impede a reincidencia. `9b7c93a`.

**F-05 — Regra de largura pelo opcode, nao pelo sufixo.** *Problema:* sufixo `_b64` implicava destino de 64 bits. *Implementacao:* `i32_result_of_64()` centraliza a excecao e os quatro sitios a consultam. *Reutilizavel:* sim; e um caso concreto de "semantica do opcode vence convencao de nome". `8f71091`.

**F-06 — Leitura de memoria de recurso colocado com recuperacao de falha (SEH), em vez de copia.** *Problema:* ler memoria de textura colocada que pode ser *decommitada* por outra thread durante a leitura (corrida com o titulo) causava crash; a primeira solucao (`595eb3a`) **copiava** com *snapshot*, cara e ainda sujeita a janela. *Implementacao:* `guard.rs` (242 linhas) expõe uma chamada que recupera a falha de acesso e devolve "nao disponivel"; `upload.rs` deixa de copiar. *Por que funciona:* a leitura acontece uma vez, no lugar, e uma falha vira **skip do upload** em vez de terminacao do processo. *Reutilizavel:* sim para qualquer camada que leia memoria de GPU compartilhada com o convidado. *Risco:* depende de SEH (`__try`/`__except` no Windows); nao e portavel. `5d5bc33` (+ `fe391fe`).

**F-07 — Stand-in de objeto COM para API que "nunca falha" no console.** *Problema:* `GameInputCreate` pode falhar no host, mas o titulo nao checa. *Implementacao:* objeto estatico com vtable de 22 metodos, tokens de registro unicos, `E_NOTIMPL`/`E_NOTFOUND` reais, e adocao condicionada a nao haver objeto real ja patchado. *Por que funciona:* preserva o contrato do console (nunca nulo) sem mentir sobre capacidades (cada metodo devolve um codigo real). *Reutilizavel:* **sim, e um padrao geral** para qualquer API de plataforma cujo contrato seja "sempre sucede". `146286e`.

**F-08 — Conformance contra o binario real, como harness.** *Problema:* "o que o runtime do console faz" era conjectura. *Implementacao:* o mesmo conjunto de cenarios roda contra o emulador e contra o `System32\xgameruntime.dll` carregado por `LoadLibraryExW` + `InitializeApiImpl(0x4a61, 0x2712)` + `QueryApiImpl`, **sem os hooks do emulador**; um `const DIFFERS` lista os cenarios divergentes e os **pula** com motivo (ou roda com `TOMBOX_XSYS_ALL=1`). *Por que funciona:* transforma comportamento de API em teste executavel e faz a divergencia ser um dado, nao uma opiniao. *Reutilizavel:* sim, com a ressalva de que depende de o runtime real existir na maquina (senao o teste faz `SKIP`). `acfede3`, `c3292cc`, `e7377c9`, `883d1c8`, `8dce1e7`.

**F-09 — GBV como oraculo de traducao.** *Problema:* erros de layout de descritor sao invisiveis em WARP e em comparacao de imagem. *Implementacao:* `TOMBOX_D3D12_GBV=1` habilita `ID3D12Debug1::SetEnableGPUBasedValidation`, opt-in, com o IID conferido por teste contra `d3d12sdklayers.h`. *Por que funciona:* a validacao de GPU compara **views do titulo** com **tipos do shader traduzido**, exatamente a fronteira onde a traducao pode errar. *Reutilizavel:* sim. `8c481f4`.

**F-10 — Prelaunch/`pack` de shaders com chave de modo.** `pack_key::Mode` + `shader_cache` + `xshader_prelaunch` permitem pre-traduzir e servir do pacote, com o custo de a traducao "ao vivo" (e seus avisos, como `gather_lod_blind`) nao acontecer para shaders do pacote. *Reutilizavel:* sim como estrategia de latencia; *risco:* perde telemetria (o proprio `d2dad0c` registra isso). `shader_xlat.rs`, `pack_key.rs`.

## G. Problemas ainda existentes

- **G-01 — Heuristica de elemento de buffer ainda e o padrao.** Sem tabela FSSM legivel ou com strides conflitantes, `sbuf_words()` volta ao chute marcado `TECHNICAL_DEBT` (`b03b39b`). Nenhum teste cobre o caso "tabela ausente em titulo real".
- **G-02 — Localizacao de `PGM_RSRC1` no container e desconhecida; MODE inicial fica em 0xf0.** `bd185fa` conectou MODE a `PGM_RSRC1`, mas a **tabela de localizacao esta vazia** por falta de fixtures: *"where PGM_RSRC1 sits inside the FSSM container is not proven ... so no container yields a value and MODE 0xf0 stays the default"*. O commit tambem declara que **so** o omod de f16 e reportado em `Translated.unknowns`; o tratamento de denormal de f32 depende do mesmo valor e **nao** e reportado. `8e7233d`/`753e851` adicionam o *scanner* que um contribuidor com fixtures pode rodar — nao rodado aqui ("not proven: 0 FSSM fixtures").
- **G-03 — `gather4_l` ignora bias/clamp quando o S# nao e visivel.** `d2dad0c` resolve o caso em que os dwords 1-2 do sampler sao constantes no mesmo bloco basico; quando o S# vem de tabela ou e calculado, o comportamento antigo permanece e so um aviso e emitido (`GATHER4_L: hash=...`). Shaders servidos do pacote de prelaunch **nao sao traduzidos ao vivo e nao sao registrados**.
- **G-04 — O clamp de blend e menos exato que o console para Inf/NaN.** A medicao em RDNA2 (gfx1036, `cb_probe`, `f1e8d7c`) mostra que o console **preserva** Inf e NaN em alvos fp16/R11G11B10 (inclusive por blend aditivo) e satura apenas overflow finito; o modo eager e o `TOMBOX_BLEND_CLAMP=ps` mapeiam Inf->max e NaN->+max. Conclusao registrada no proprio BLOCKERS: *"are less exact than no clamp for Inf/NaN"*. Tambem registra que o comportamento de overflow finito e **dependente de vendor** (NVK/NVIDIA arredonda e estoura para Inf).
- **G-05 — Recusas de ISA que continuam recusas.** `image_atomic_fcmpswap/fmin/fmax` (sem equivalente em DXIL e sem vetores gravados em RDNA2), `R128`/`TFE`/`LWE` em MIMG, `MUBUF TFE`, raw com `IDXEN`, `ds_swizzle` em certos estagios, GS emit/cut em streams 1-3, `s_sendmsghalt`, `ds_ordered_count`. Sao **decisoes** (falhar alto), nao bugs — mas delimitam o que ainda nao roda.
- **G-06 — `verify_terminate_waits_for_async_in_flight` falha no emulador e passa no runtime real.** Declarado em `e7377c9`: *"the fix belongs in queue termination, which open XTaskQueue work rewrites"*. Continua aberto no fim da janela.
- **G-07 — `VirtualFree` de titulo pode falhar ate ~50 ms apos o release do ultimo buffer aliasado.** `ab6d485` deixou a linha `OPEN, not seen in a title run` em `BLOCKERS.md`: com `TOMBOX_ALIAS_TITLE=1`, o wrapper de `VirtualFree` em `xmem` passa direto e nao espera o unpin assincrono do Windows. Observado em **D3D12 puro** (24-55% com carga de copia), ou seja, e comportamento do host.
- **G-08 — Estagios de shader incompletos.** `MILESTONES.md`: "All shader stages translated 0/8 open". Tessellation (hull/domain) tem apenas o **parser** de PSV0 (`10834855`, `parse_psv0_tess`) e a estrutura `TessInfo`; `hlsl/tess.rs` existe, mas o plano (#294) esta na etapa 1. Amplificação e malha existem mas: wrap do anel do console nao modelado (um entry por grupo), encoding 2D/3D de `DispatchMesh` nao verificado, registrador de payload do mesh shader nao estabelecido, e tabelas de descritor em root memory sao recusadas (`3d90f6f`, BLOCKERS).
- **G-09 — O check de workspace do CI nao bloqueia e nao cobre `--all-targets`.** Em `a618736`: job `rust-workspace-check` com `continue-on-error: true` e `cargo check --locked --workspace --exclude connectedstorageservice_proxystub --exclude gamecorecomproxy --exclude wldp`. Foi exatamente esse furo que deixou o `E0583` de `xg_xs` passar (D-12); **o furo estrutural nao foi fechado**, so o sintoma.
- **G-10 — Proveniencia/licenca.** O projeto relicenciou para **GPL-3.0-or-later** (`ceb2b87`) e depende de codigo cujo licenciamento e incerto: `THIRD_PARTY/CREDITS` registra TomBox de drycheetah84 como *"publishes no licence"* e o fork brcly/TomBox como *"no longer reachable"*, com a ressalva de que "o commit portado declarava MIT OR Apache-2.0". Isso e um risco registrado, nao resolvido.
- **G-11 — Metricas de cobertura nao sao prova de execucao.** `MILESTONES.md` marca "All Direct3D 12.x methods covered 200/200" e "Full title shader sweep translated 105506/105506" como *reached*, enquanto o mesmo repositorio registra Fortnite com material errado, Trackmania com texturas pretas e passes de iluminacao escuros, Minecraft crashando e XDefiant a <1 fps. "Traduzido" aqui significa "produziu HLSL que o DXC aceitou", nao "renderiza correto".
- **G-12 — 117 itens `OPEN` e 5 `PARTIAL` em `docs/BLOCKERS.md`** contra 62 `DONE` (contagem por `^- OPEN`/`^- PARTIAL`/`^- DONE` no arquivo em `a618736`). Nao os reproduzo todos; os de maior impacto estao em G-01..G-08.

## H. Regressoes

- **H-01 — `df91928` reintroduziu trabalho duplicado em todo AS.** Ao "corrigir" o caso de Y/Z zero, passou a ler Z do dword do ponteiro de payload: **cada grupo de malha lancado duas vezes**, invisivel em pixels (D-07). Corrigido em `53bb998` ~1 dia depois. E a regressao mais clara do periodo: uma correcao plausivel sem fixture que a validasse, detectada apenas quando o teste passou a contar amostras.
- **H-02 — Prewarm de game-scan esgotou a memoria de titulo do Trackmania.** `3de1b1f` e auto-descrito como *"r31.1 hotfix: game-scan runtime prewarm opt-in (TOMBOX_GAME_SCAN_PREWARM=1); **it exhausted Trackmania's title memory and aborted map load**"*. O prewarm entrou durante a fase `r31` (`a1ff5be` e arredores) e foi convertido em opt-in no dia seguinte; a busca `git log -S"GAME_SCAN_PREWARM"` so retorna `3de1b1f`, entao o commit **introdutor** do prewarm sem gate nao e identificavel por essa via (**PROVAVEL**: `a1ff5be`).
- **H-03 — Teste de conformidade foi afrouxado para acomodar o comportamento novo.** Em `ba8e0e5` (#307), `gdk_process_queue.rs` troca `assert_eq!(cur, mine)` por `assert_ne!(cur, 0); assert_ne!(cur, mine)` e `assert_eq!(cur, def)` por `assert_ne!(cur, 0)`. Num projeto cujo metodo e "igualar o runtime do PC medido", trocar igualdade por desigualdade **remove a garantia** que o teste existia para dar. Pode ser justificado (identidade de handle), mas o commit nao declara medicao do runtime real para esse ponto — diferente do padrao dos commits de `TQquentinD`, que sempre citam `xgameruntime.dll 10.0.26100.9656`. Classificacao: **CONFIRMADA** a mudanca; justificativa **NAO VERIFICADA**.
- **H-04 — BOM UTF-8 inserido em arquivo de teste.** `ba8e0e5` deixa `crates/compat/compat-common/tests/gdk_minecraft_slots.rs` comecando com `EF BB BF` (verificado por leitura dos bytes no HEAD). `rustc` tolera BOM, entao e cosmetico; mas e um artefato de rebase/cherry-pick que indica ausencia de normalizacao de encoding no fluxo de merge.
- **H-05 — `595eb3a` foi uma correcao que precisou de correcao.** A primeira abordagem ao bug #110 (corrida com decommit de textura colocada) lia a memoria por **copia/snapshot**; `5d5bc33` a substitui por uma chamada com recuperacao de falha, e o commit seguinte se descreve como *"root fix for #110"* — ou seja, a primera foi remendo. Nao houve remocao do codigo antigo num commit separado; a substituicao aconteceu no mesmo dia.
- **H-06 — Mensagens de commit que descrevem trabalho ausente.** `c95a909` ("Exec mask ... Fortnite derived PSO use-after-free fix") tem diff integralmente em `xtfeventserver` (COM/ETW); `d585b94` ("Fix #109: recover poisoned shader_xlat mutexes") nao toca `shader_xlat.rs` (o arquivo tem 27 ocorrencias de `into_inner`/`poison` no HEAD, sem commit correspondente localizavel). Isso nao quebra o produto, mas **quebra a auditabilidade** — e e a razao pela qual varios itens deste documento ficam como NAO VERIFICADO.

## I. Conhecimento reutilizavel

### I.1 Arquitetura do tradutor de shaders (o nucleo do projeto)

O que torna este repositorio diferente de uma camada de compatibilidade comum e que **o programa de GPU do convidado e codigo nativo da maquina do console**, e nao um API call. A cadeia, documentada em `crates/tombox-xshader/src/lib.rs:1-16` e em `d3d12_xs/src/shader_xlat.rs:1-6`:

```
DXBC  ->  chunk DXIL (com o bitcode removido, marcado NOBC)
      ->  chunk DXIL substituido por payload XDAT (nativo do console)
      ->  zlib inflate
      ->  container FSSM  (cabecalho + tabela de bindings + ISA)
      ->  microcodigo RDNA2 (gfx10xx)
      ->  decodificador ISA  ->  IR de instrucoes
      ->  emissor HLSL (SM6, as_6_5/ms_6_5/ps_6_x...)
      ->  DXC (dxcompiler.dll)  ->  DXIL
      ->  cache por hash FNV-1a (+ pack de prelaunch)
```

Componentes reaproveitaveis, cada um com um problema bem delimitado:

1. **`extract()` + `fssm_bind::parse()`.** O container do console carrega o que o driver original sabia: o *estagio de hardware*, o *slot* de cada estagio, a **tabela de bindings com registro/classe/stride**, e um byte de contagem de user SGPR (0xd6). Isso resolve tres classes inteiras de bug de traducao (D-01, D-03, e os samplers literais de #329) sem heuristica. **Licao transferivel:** quando o binario do convidado vem de um driver real, o *metadado do driver* e a fonte de verdade do layout; o padrao de acesso na ISA e apenas um palpite.
2. **`meta.rs` / `dxil_meta.rs` (PSV0).** Le nome/tipo/registro/mask de assinaturas de entrada/saida, primitiva de entrada e topologia de saida de GS, `numthreads` de CS, e as uniões de **mesh** (`parse_psv0_mesh`) e **tess** (`parse_psv0_tess`, `10834855`). O PSV0 e o equivalente ao "reflection" do DXIL e e o que permite reconstruir a interface do estagio na HLSL sem root signature.
3. **Inferencia de bindings em camadas, com fallback explicito.** Ordem observada: root signature estatica -> tabela FSSM -> user data por chute de estagio -> retry por `s_swappc`. Cada camada **recusa com nome** quando nao fecha (`"buffer in table of kind None"`, `"base sN is not a root table pointer"`, `"not a root pointer"`). O padrao "camadas com recusa nomeada" e o que permite rodar 105.506 shaders e saber *por que* cada um falhou.
4. **`rootmem.rs` (ABI de root memory).** Implementa o `FORCE_MEMORY_BASED_ABI`: root constants e descritores de CBV/SRV/UAV lidos por `s_load` em `s[0:1]` com offset constante, mapeados para a root signature. Regra de ouro descoberta por medicao: **todo PSO com estagio de amplificacao usa esse ABI**, sem deteccao heuristica (`3d90f6f`).
5. **Modelo de aneis para mesh/amplification (`amp.rs`, `mesh.rs`).** O AS do console nao "desenha": escreve em dois aneis em memoria (`args`, `payload`) e o driver emite `DISPATCH_DIRECT` + `DispatchMesh`. O tradutor fixa `s2/s3` como bases de anel, mapeia stores `flat/global` do grupo para `groupshared` (`tb_args`/`tb_pl`), conta qualquer outro endereco como **falha** (e nao como escrita silenciosa), e emite `DispatchMesh(tb_args[0], max(tb_args[1],1), 1, tb_pl)`. ABI do MS: `v5 = workgroup vertex id`, `s3[27:24] = wave index`, `s_sendmsg 9` (`GS_ALLOC_REQ`) com `m0 = prims << 12 | verts`, `exp pos 12` / `clip-cull 13-15`, `exp prim 20`.
6. **Modelo de modo numerico (`hlsl/mode.rs`).** `MODE` (FP_ROUND, FP_DENORM, DX10_CLAMP, IEEE) e derivado de `PGM_RSRC1` e propagado por `s_setreg`; `reachable`/`analyze` comecam dele e `tb_hw1` e inicializado com ele. **Licao:** o comportamento de denormal/arredondamento **nao e global**, e estado por onda — um tradutor que fixa `0xf0` acerta por acidente em muitos shaders e erra em todos os que dependem de modo.
7. **Refusals como contrato.** O emissor prefere falhar a emitir errado: `pck` so em imagem usada tambem com atomics 32/64 bits (porque loads tipados sempre convertem pelo formato da view), `D16` sem forma pck, atomicos float, `R128/TFE/LWE`, raw+IDXEN. **Licao:** num tradutor de ISA, "nao suportado" com motivo nomeado vale mais que "aproximado".
8. **Prova bit-exata.** `tests/exec*.rs` compilam o HLSL com DXC e **executam** em WARP (`TOMBOX_EXEC_HW=1` para GPU real), comparando bits; `tests/isa_sweep` varre mnemonicos; `src/trans32/amd.rs` embute modelos de `v_log/exp/sin/cos` vindos do rocjitsu (MIT). Este e o ativo mais reutilizavel do repositorio: **um harness que transforma "o shader traduzido esta correto" em teste executavel**.

### I.2 Shims de sistema: onde o projeto se separa de uma camada estilo Wine/XCL

Uma camada estilo Wine (e, pelo que se pode inferir do nome, o XCL do mesmo ecossistema) resolve o problema **na fronteira de API/syscall do sistema operacional**: intercepta chamadas Win32/WinRT, mapeia para o SO hospedeiro, reimplementa objetos COM e entrega o trabalho grafico a API do host por traducao de *chamadas* (D3D->Vulkan, etc.). O programa do convidado e tratado como codigo x86 que so precisa de um ambiente.

O xbox-emulator precisa de tudo isso **e mais tres coisas que uma camada Wine nao tem**:

1. **O programa de GPU do convidado nao e uma chamada de API, e binario de hardware.** Nao ha "D3D11 -> Vulkan": ha ISA RDNA2 -> HLSL -> DXIL. Isso obriga a reconstruir **layout de recurso, ABI de estagio, modo numerico e semantica de cada opcode** — e a manter exatidao bit a bit onde o hardware do console difere do PC (denormals, NaN, `-0`, arredondamento de blend, `MIMG pck`, `SDWA`, GDS, atomics de DS). Uma camada Wine pode dizer "o comportamento e o do driver"; aqui o comportamento e o **do console**, e o driver do host e um adversario.
2. **A ABI de objeto do console nao e a do PC, e muda por versao de SDK.** `d3d12_xs` remapeia *slots de vtable*: 127 (`CreateCommittedResource3`), 133 (`CreatePlacedResourceX` com desc 1) para chamadores GXDK 2504+ (`945731f`), 131/132 que mudam de sentido conforme a edicao (`6d19943`), e uma tabela de shim que cresceu de 128 para 256 entradas para que os slots existam. Pior: o **significado** de um slot depende da edicao do SDK com que o modulo chamador foi compilado, lida do campo `_xbld_edition_sdktype=GXDK,<edition>` na secao `.xbld` do PE do chamador, com o chamador identificado pelo primeiro endereco de retorno fora do emulador (`cl_edition.rs`, 430 linhas). Nao existe equivalente disso em Wine.
3. **O modelo de memoria e de recursos e do console.** Heaps colocados, aliasing de VA, `MSAA` com layout proprio, *tile patterns* (tiling Scarlett consultavel por `GetResourceTiling`, `62c525d`), spans de recurso que precisam ser rastreados e liberados quando o recurso morre (`fa9f1ca`). O PC D3D12 nao expoe essas garantias, entao o shim mantem **bookkeeping proprio** (`alias.rs`, `placed_offset.rs`, `gpu_va.rs`) e as vezes precisa ler memoria que o titulo pode estar desmapeando (F-06).

Em contrapartida, o projeto tambem faz o que uma camada Wine faz, e faz com o mesmo metodo: `compat-common` reimplementa XAsync/XTaskQueue/XSystem/XUser/XGameSave/gamepad/fs/registry/redes; `xg_xs` reimplementa o runtime grafico de baixo nivel; `xcrdapi` reimplementa DirectStorage (incluindo o decodificador **BCPack** com rANS, `crates/compat/xcrdapi/src/bcpack`, portado de `bcpack-rs` e `silicon-rans`). A diferenca de postura e a **fonte de verdade**: aqui ela e medida contra o binario real (`InitializeApiImpl(0x4a61,0x2712)` + `QueryApiImpl` do `xgameruntime.dll`) e contra silicio real (`tools/rdna2-hwprobe`), com a divergencia registrada em `const DIFFERS` em vez de tolerada.

### I.3 Padroes de engenharia reaproveitaveis (independentes de Xbox)

| Padrao | Problema que resolve | Onde | Como adaptar |
|---|---|---|---|
| Falha alta com motivo nomeado | traducao que "funciona" e corrompe | `rectlist::refuse`, refusals de MIMG/raw | trocar todo `unwrap_or(default)` de parsing por erro com contexto |
| Metadado do produtor como fonte de verdade | inferencia de layout por padrao de acesso | `fssm_bind`, PSV0 | se o binario foi gerado por um driver, leia o que o driver sabia |
| Conformance contra o binario de referencia | semantica de API conjecturada | `gdk_*_conformance(_real)` | carregue o DLL real, rode os mesmos cenarios, liste `DIFFERS` |
| Validacao de GPU como oraculo de traducao | erro de descritor invisivel em WARP | `TOMBOX_D3D12_GBV` | habilite GBV/`VK_LAYER_KHRONOS_validation` e trate o relatorio como teste |
| Guarda de IEEE contra o compilador do host | dobramento algebrico muda bits | `tb_pz`, `tb_hz`, `tb_trunc64` | nunca emita `a - b`/`a + 0` crus quando o bit de zero importa |
| Teste por contador, nao por imagem | duplicacao de trabalho invisivel | query de oclusao em `tests/amp.rs` | meça invocacoes/amostras, nao so pixels |
| Fuzz de caminho com alfabeto Unicode | panic por fronteira de caractere | `fuzzed_console_paths_never_leave_their_drive` | qualquer parser de caminho de entrada hostil |
| Validacao com estado entre entradas | colisao em filesystem case-insensitive | `tombox-archive::validate` | normalize antes de comparar; trate colisao como bloqueio |
| Teste que espera assincronia do SO | flaky "de timing" que e comportamento real | `ab6d485` | meça o efeito observavel, depois espere com limite |
| Mutacao que derruba o teste | teste que passa por acidente | varios PRs (`d2dad0c`, `bd185fa`, `753e851`) | para cada fix, escreva a mutacao que o quebra |

## J. Conclusao tecnica

**O que este repositorio e.** Um emulador de camada de plataforma para Xbox Series X|S, escrito em Rust, que ataca o problema no nivel mais dificil: **microcodigo RDNA2 pre-compilado**. Nao ha fonte do shader; o tradutor reconstroi a intencao do programa a partir da ISA e do metadado do container do console, emite HLSL e recompila. Em 5 dias e 595 commits, o projeto saiu de um `README` de 16 linhas para 1652 arquivos, ~100 crates e seis titulos em estados diferentes de execucao (Fortnite, Trackmania, Terraria, Minecraft Beta, XDefiant, GTA VI) — com Trackmania de fato jogavel.

**Qualidade de engenharia: alta, e de um tipo raro.** Tres praticas se destacam e sustentam o resto:
1. **Fonte de verdade medida.** Conformance contra `xgameruntime.dll` real, sondas de silicio (`rdna2-hwprobe` em gfx1036/RX 9060 XT), GBV como oraculo de descritor, e testes de execucao bit-exatos em WARP + GPU. Onde nao foi medido, o texto diz "not proven" — e isso e verificavel no proprio codigo.
2. **Recusa nomeada em vez de aproximacao.** O tradutor prefere nao traduzir a traduzir errado, e a lista de recusas e documentada por opcode.
3. **Rastreabilidade de prova por commit.** Os commits dos contribuidores (TQuentinD, GDAgent Team, Isaac Wood, enderrr9, MeNoSkills) trazem medicao, resultado antes/depois, e a **mutacao** que prova o teste. E o melhor nivel de disciplina de commit que se ve num repositorio deste tamanho.

**Fragilidades reais, em ordem de importancia.**
1. **Historia truncada no import.** Todo o codigo pre-existe a `ff39277`; 19 dos 22 defeitos catalogados so podem ser datados como "import". Isso e uma limitacao de arqueologia, mas tambem um sinal: o projeto publicou o codigo sem o historico que explica as decisoes.
2. **Commits-mega do mantenedor.** Os lotes `rNN` misturam assuntos e, em pelo menos dois casos (`c95a909`, `d585b94`), a mensagem descreve trabalho que **nao esta no diff**. Para um projeto cuja maior virtude e a auditabilidade, isso e uma regressao de processo.
3. **Inferencia ainda heuristica nos pontos de maior impacto.** Stride de buffer (D-01), base de user data (D-03) e localizacao de `PGM_RSRC1` (G-02) dependem de metadado que **nao esta provado** em todos os containers. O padrao de correcao ja existe (#289/#330/#347); falta fixture.
4. **CI nao bloqueia o check de workspace.** O furo que deixou `xg_xs` quebrado em checkout limpo continua aberto (`continue-on-error: true`, sem `--all-targets`). E o tipo de divida que so aparece quando alguem clona o repositorio.
5. **Metricas verdes, produto parcialmente vermelho.** 200/200 metodos D3D12 e 105506/105506 shaders convivem com material errado, texturas pretas e crashes. As metricas medem cobertura de traducao, nao correcao — e o README e honesto sobre isso, mas o `MILESTONES` nao e.

**Veredito.** Como artefato de engenharia, e um dos projetos mais bem instrumentados que este tipo de analise encontra: cada correcao vem com medicao, teste e mutacao, e o registro de bloqueios e um documento vivo com 117 itens abertos declarados. Como **emulador**, esta em estagio inicial de fidelidade: dois titulos rodam, nenhum esta correto em 3D, e o caminho para "3D correto" passa por problemas que o proprio repositorio ja identificou e nao fechou (denormals por modo, Inf/NaN no blend, texturas streamed pretas por escrita de CPU em memoria de GPU). A linhagem com o TomBox e **nominal e metodologica**, com reuso de patches Apache-2.0 — nao e um fork, e a coincidencia de 1652 arquivos com o TuxBox clonado e apenas isso, uma coincidencia.

## K. Indice de commits

Todos os 595 commits de `HEAD` (`main`). Formato: `hash curto | data | autor | assunto`. 104 deles sao merges.

```
2b4bba2 2026-11-21 omonkey init: progress tracker for TomBox + rage9
3f1762f 2026-11-21 omonkey scope to Xbox PC port only, add progress map + log + blockers
dd4da0d 2026-11-21 omonkey progress treemap + badges in the upstream TomBox style
7479464 2026-11-21 omonkey reframe as a TomBox continuation covering the Xbox Series library
2e631b5 2026-11-21 omonkey status: first Fortnite boots, DirectStorage + Media Foundation staged
0e90b75 2026-11-21 omonkey status: first frames on this machine; rename to Xbox Emulator
ad294d6 2026-11-21 omonkey status: local backend, waiting room passed, stuck at checking for updates
f4db225 2026-11-21 omonkey status: login completes, hotfix succeeds, game-mode select reached
0a967fd 2026-11-21 omonkey status: Battle Royale lobby reached, stable 5+ minutes
cc23333 2026-11-21 omonkey libraries: XDefiant dump added, new title modules and newer console OS exports
2b52553 2026-11-21 omonkey progress: own progress map, badges and whole-console library map (Fortnite + XDefiant)
2b11a65 2026-11-21 omonkey status: online services now emulated in-process; lobby reached without a separate server
4291a50 2026-11-21 omonkey gitignore: keep game dumps (games/) and emulator workspace out of the public repo
2fafb83 2026-11-21 omonkey libraries: Terraria dump added (Unity IL2CPP, Xbox One GameCore), third title in the progress map
368e0ad 2026-11-21 omonkey libraries: Trackmania dump added (Nadeo GBX, Xbox One GameCore), newest console OS 10.0.22621.1852
234c512 2026-11-21 omonkey progress: merge verified library work (xmem, d3d12 VM + device slots, rdna2 decoder fixes)
435cb5a 2026-11-21 omonkey progress: Xbox One GameCore driver variants (d3d12_x, xg_x, dxcompiler_x, XFrontPanelDisplay)
19a9f32 2026-11-21 omonkey docs: log the Xbox One driver variants; soften the D3D12.x vtable claim
fa75ffc 2026-11-21 omonkey progress: count only Xbox console libraries in the map and headline
76be322 2026-11-21 omonkey progress: PC twins for console host DLLs, GameInput #1 done, evidence gates, not-targeted list
8b09cc0 2026-11-21 omonkey progress: emulated Xbox One front panel and ESRAM aliasing credited
b14ee89 2026-11-21 omonkey progress: define pc / hooked twin statuses in the map legend and scope
9049041 2026-11-21 omonkey blockers: current missing pieces for Fortnite, Terraria and Trackmania
d9c6b87 2026-11-21 omonkey progress: D3D12XboxSetProcessDebugFlags credited; MF tracer evidence path; GameChat2 title-DLL probe
b288e31 2026-11-21 omonkey progress: finish past libraries (workflow)
a87d989 2026-11-21 omonkey blockers: Trackmania packs fixed; now blocked on Xbox One shaders and command-list close
c26f4c3 2026-11-21 omonkey BLOCKERS: run 33 backdrop diag, hologram/login, performance
e405faa 2026-11-21 omonkey BLOCKERS: Terraria progress (IDXGIDevice, queues, heaps, real GCN xg_x), Trackmania Durango shader container and Close failure details
ccaa634 2026-11-21 omonkey BLOCKERS: run 34-35 (backdrop drawn, menu freezes, low detail, XAsync login)
f9a68fd 2026-11-21 omonkey BLOCKERS: Trackmania presents frames (barrier fix), GCN decoder status, debug layer limits, Terraria frame events checked
061e906 2026-11-21 omonkey BLOCKERS: Durango program range from header 0x1dc
3f479e9 2026-11-21 omonkey BLOCKERS: agent updates (PlayFab, XPackage/saves, Terraria hang, Trackmania, GameChat2)
3da9424 2026-11-21 omonkey progress: MF evidence from run 35
3cbe510 2026-11-21 omonkey progress: system imports 658/671 (MF evidence, XAudio2 ordinals, SILK fix)
3572598 2026-11-21 omonkey BLOCKERS: offline, private lobbies/games only; private-match gaps
9cf0574 2026-11-21 omonkey BLOCKERS: audio record/replay, decoder perf, Terraria texture-VA hang
6845fd2 2026-11-21 omonkey progress: AcpHal + XAudio2 complete (imports 669/671), D3D12.x console slot tables
7423cfc 2026-11-21 omonkey progress: Xbox One cmdlist slots, committed-resource and texture-VA mapping; BLOCKERS: Terraria texture-VA fixes, VS fetch-shader gap
4447af8 2026-11-21 omonkey progress: RDNA2 execution-verified opcodes, audio/xapu/chad tables, D3D12.x batch 2
b897616 2026-11-21 omonkey progress: refresh map
6dca2a2 2026-11-21 omonkey Progress map: imports 671/671, D3D12.x 165/200, RDNA2 526/549, console 1291/2141
d867570 2026-11-21 omonkey Progress map: D3D12.x 183/200, RDNA2 545/549, console 1861/2141
465e69f 2026-11-21 omonkey Map: D3D12.x 188/200, RDNA2 545/549, console 1922/2141; live milestones 9/18
b5eb816 2026-11-21 omonkey Map update: xtfbaseservices, wdbase, wldp, secur32, xcrd XvdXfer host
92b3cbc 2026-11-21 omonkey Map update: pixrt, xbthunk, mfplat, symsrv, d3d12_x, CL splits
c035c0d 2026-11-21 omonkey Map update: s_trap, permlanes, shader compilers, xcrd driver
6997230 2026-11-21 omonkey Map update: msave64xgs, mesh slots, minuserx, gamehost
b2b6e73 2026-11-21 omonkey Map update: xcrd paths/BAT/info, xtfpix server, console 2074
ff39277 2026-11-21 omonkey Add emulator source (code only; no game data, binaries or notes)
312646f 2026-11-21 omonkey README: link to emulator source
410e01f 2026-11-21 omonkey Move emulator source to repo root
9bfb4b9 2026-11-21 omonkey Remove run scripts and STATUS.md
3ae6e84 2026-11-21 omonkey D3D12 189/200: DrawIndexedX decoded
57481b4 2026-11-21 omonkey Open to contributions: CONTRIBUTING, licence, templates
55d9166 2026-11-21 omonkey D3D12 190/200: GPU hardware configuration
734c469 2026-11-21 omonkey D3D12 192/200: GPU counter begin/end
318a7c7 2026-11-21 omonkey D3D12 193/200: counter sets
96fe927 2026-11-21 omonkey D3D12: counter sampling audit notes
6ecd9c3 2026-11-21 omonkey D3D12: bottom-of-pipe write notes
062ef3e 2026-11-21 omonkey D3D12 183/200: audit downgrades 10 methods to partial
e475125 2026-11-21 omonkey D3D12 184/200: MSAA centroid priority
1032236 2026-11-21 omonkey D3D12 Device 82: Scarlett counter set evidence
a6aa1a9 2026-11-21 omonkey D3D12: CL 7, Device 28/61 done; CL 104 partial on Scarlett
ab7717e 2026-11-21 omonkey RDNA2 676/721: VOPC verified on WARP (from community PR #15, re-run 5/5)
89cb6de 2026-11-21 omonkey D3D12 CL 64: DB count control evidence (still partial)
dc36491 2026-11-21 omonkey D3D12 191/200: Device 25-27, 29 follow the release driver
6c29ddf 2026-11-21 omonkey D3D12 190/200: Device 29 back to partial (PC placed-offset alignment), 60/62 ext layouts decoded
ecd5dce 2026-11-21 omonkey D3D12 Device 55/60: Fortnite depth-stencil extension decoded (still partial)
f573b18 2026-11-21 omonkey D3D12 Device 55/60/62: all PSO extension blocks decoded
09f3c1a 2026-11-21 omonkey D3D12 CL 70: per-family bottom-of-pipe write, GPU-clock SYSTEM_TIMESTAMP (still partial)
8236000 2026-11-21 omonkey D3D12 191/200: Device 62 done; pixel-shader options wired for 55/60
1f14ce4 2026-11-21 omonkey D3D12 198/200: slots that run correctly on PC counted done, console-only effects documented
8c43fba 2026-11-21 omonkey D3D12 190/200: hold 55/60/64/70/86/101/104 at partial until verified
38370aa 2026-11-21 omonkey D3D12 Device 29/44: implementation evidence (pending verification)
100f92a 2026-11-21 omonkey D3D12 CL 74: bundle state carry-over evidence (pending verification)
eba1a5a 2026-11-21 omonkey Map: yellow 'awaiting verification' state for RDNA2 instructions and console library exports
1a8dd50 2026-11-21 omonkey D3D12 192/200: 55/60/104 verified; CL 85 back to partial (predication not tracked)
46d304d 2026-11-21 omonkey D3D12: Trackmania placed-view device removal fixed
f4d3e5f 2026-11-21 omonkey D3D12 194/200: CommandList 64 and 70 verified; RDNA2 pending batch (35); ignore partial downloads
885e759 2026-11-21 omonkey RDNA2 map: 779 instructions after mnem_groups resync; 95 pending verification; GWS/ordered_count blockers
8e10e8d 2026-11-21 omonkey RDNA2 771/779: 95 instructions independently verified
1fc8683 2026-11-21 omonkey Source sync: ISA sweep harness + 13 translator fixes, D3D12 slots 29/44/74/85/86/101 fixes and hardware tests, RDNA2 ds_append/consume, xg DCC decode
26ced2a 2026-11-21 omonkey ISA sweep verified (169 instructions backed by exec tests), SDWA float neg/abs, BLOCKERS notes for D3D12 last slots and held RDNA2 ops
4f48148 2026-11-21 omonkey Shader stages audit: Hull/Domain not implemented, VS/PS/CS/GS held until end-to-end verified; console-lib work in progress
c076007 2026-11-21 omonkey Console libs: KernelTraceControl merge, hix_xs hang dump/debug forwards, xbtp sessions, LSA passwordless, wldp toast; 30 new exports held pending verification
de34ec8 2026-11-21 omonkey Console libs verifier pass: 37 defects found in older rows; 15 title-used rows held pending fixes
ba55a47 2026-11-21 omonkey Console libs: 34 new exports verified against console DLLs; xbtp AddPipeEndpoint and xmem working-set layout held
4c94859 2026-11-21 omonkey Map: Minecraft analysed; console-hosted API sets, ordinal and GetProcAddress usage counted; per-OS kernel32legacy table; honest stub labels
7b2f86d 2026-11-21 omonkey Console libs: 37 verifier defects fixed (task queue terminate, XVersion order, GDK error codes, D3D12 device wrap, front panel, minuserx A/W, xmem layout, memstatus); honest stub labels; XGEncodeDCC, MFGetPlatform, XVMAudioService classes; xbtp parallel tests; workspace builds again
f55ed9b 2026-11-21 omonkey Console libs: XGEncodeDCC verified against console xg_xs (1000-case fuzz, 0 mismatches)
afffefd 2026-11-21 omonkey Console libs: second verifier pass on defect fixes; MFGetPlatform verified; remaining failures held pending
f691d4a 2026-11-21 omonkey Console libs: round-2 verifier fixes (task queue terminate on libHttpClient model, FindChunkAvailability, store license validity, minuserx index checks, d3d12_xs per-OS-build ABI, d3d12_x deserializer flags); gamecoreclient via xvcmgmt peer; labels: pass-throughs = forwarded
4ad72a4 2026-11-21 omonkey Console libs: round-2 fixes verified (package, store, minuserx, d3d12_xs per-OS ABI, d3d12_x); XTaskQueueTerminate still held
ba24a0b 2026-11-21 omonkey Console libs: all 37 verifier defects now verified fixed (XTaskQueueTerminate round 3); gamecoreclient verified; sandbox registry/handle fixes
2850610 2026-11-21 omonkey Console libs: 34 console-identical fixed-return rows verified (xcrdapi, mflnmhelp, mfhevcdec, advapi32legacy, pixevt, sc_dll, xbsc_xs, xbtp link, d3d12_x/xs DXGIX); scdxil loader-ref fix; xdxr unwind; gamecoreclient host replies
bbdb0ae 2026-11-21 omonkey Console libs: Minecraft GDK slots verified (XGameStreaming, XTaskQueue monitors/process queue, XSystem ids, XPackage locale, XUser guest, XGameUi text entry); gamecoreclient host broker + xvcmgmt system codes; xbtp/xtf clients pending verification
a852522 2026-11-21 omonkey Console libs: xbtp sessions/dispatcher, xtfpix, xtfdebugmonitor, XtfCreateRemoteRunClient verified (cross-tested against console DLLs on loopback); XbtpWrapDispatcher and xtfremoterun #4 held
49011b6 2026-11-21 omonkey Console libs: wldp toast, hix_xs CollectHangSummary, netprofmsvc_c DllMain, xtffileio #4, xtfconsolecontrol #4 verified; kernelbase family xref and passwordless held
9f82c41 2026-11-21 omonkey Console libs: XbtpWrapDispatcher redirect port + SetSessionKey, xtfremoterun security fixes; xg_xs DCC back to stub (console output contains caller stack addresses); switch to bulk game testing
0284e83 2026-11-21 omonkey Sandbox: fix mid-path drive-letter escape in fs::resolve (console NT name rules, 50k fuzz); kernelbase family xref persisted to emulated HKU; passwordless uses caller token; hix_xs honest KMD results
fd02a3b 2026-11-22 omonkey Console libs: xg_x texture layout rewritten from the console model (0 mismatches vs 3 builds); xg_x DCC sequence-exact; compat list + batch harness
b72625c 2026-11-22 omonkey Console libs: MFGetPlatform implemented; XVMAudioService round 5 progress
ceb2b87 2026-11-22 omonkey Relicense to GPL-3.0-or-later
6741468 2026-11-22 omonkey CI gate, Rust and actionlint workflows; contributor reverse sync
0caa263 2026-11-21 alpha Validate mesh group dimensions and initialize all amplification arguments
e1af88f 2026-11-21 Michael Kelly Test versioned root signature round trips through legacy deserialization
e14ed93 2026-11-21 alpha Allocate the WMA Pro scale-factor lookup table on the heap
8325aa6 2026-11-21 Michael Kelly Test scalar byte offsets in MUBUF loads and stores
32cbb09 2026-11-21 Michael Kelly Cover 32-bit buffer loads and stores with scalar offsets
aeb4205 2026-11-21 Dea7hsmiles tombox-rpf: reject truncated RPF8 headers
a0c9fd5 2026-11-22 omonkey Merge contributor PRs #81 #94 #87 #86 #50 (cherry-picked onto rewritten main)
6b3c617 2026-11-22 Gabriel Spoleto Couto tombox-xshader tests: pick a mesh-capable hardware adapter
3526327 2026-11-21 alpha Keep uniform mesh entry allocation counts local to each invocation
3f28679 2026-11-21 alpha Isolate overlapping WARP draws in mesh shader tests
95b8090 2026-11-21 alpha Avoid reducing and hashing HLSL twice on runtime shader lookups
6700bd4 2026-11-21 Michael Kelly Reject malformed xshader command-line options
672c3eb 2026-11-21 alpha Respect subword shader load widths at GPU memory window boundaries
785b5d1 2026-11-21 Michael Kelly Explain missing root bindings for shader buffer loads
4f89f4e 2026-11-21 TQuentinD xshader: Leonida hardware vectors as exec tests; v_pk_lshlrev_b16 / v_pk_lshrrev_b16
93861d6 2026-11-21 TQuentinD tombox-xshader: wave32 lane ops address the guest half of a 64-lane host wave
b445a1f 2026-11-22 TQuentinD xshader tests: image_get_lod tolerance of one LOD step on hardware
6afb54f 2026-11-21 TQuentinD xshader: ISA-exact results on NVIDIA hardware (RTX 5090)
ed82985 2026-11-21 Michael Kelly Exercise allocation and working-set queries through xmem exports
7694db3 2026-11-21 Michael Kelly Inspect RPF8 tables that extend past the read limit
0b4583d 2026-11-21 Louisreed fix(tombox-pe): bound RVA arithmetic in directory parsers to uphold the no-panic invariant
0a6490a 2026-11-21 Dea7hsmiles tombox-archive: reject fake EOCDs in comments and bound ZIP64/CD reads
d6f9952 2026-11-21 Dea7hsmiles tombox-disasm: bounds-check symbol table parsing
f9ebce6 2026-11-21 Dea7hsmiles tombox-disasm tests: cover arbitrary and truncated symbol buffers
0ad667f 2026-11-21 TQuentinD Match interpolation modifiers between a pixel shader and the stage feeding it
14e873c 2026-11-22 omonkey THIRD_PARTY credits for tombox-xshader and d3d12_xs (Leonida vectors from #64, brcly/TomBox port from #36)
81a1252 2026-11-22 omonkey Merge contributor PRs #98 #92 #88 #82 #80 #75 #61 (#62 #64) #93 #85 #69 #44 #46 #36 (cherry-picked onto rewritten main)
79a3b2f 2026-11-22 omonkey Credits: one THIRD_PARTY/CREDITS list at the root, one line per project; per-crate credit files removed
73c5103 2026-11-21 Dea7hsmiles xshader: translate gfx7 MTBUF dfmt/nfmt to RDNA2 unified formats
d693e69 2026-11-22 adriaespii d3d12_xs: stop caching a false Xbox One / texture-VA answer before d3d12_x.dll loads; register Direct queues
54c3da1 2026-11-22 omonkey Credit LLVM's DfmtNfmt2UFmtGFX10 table used by #37
99f48e3 2026-11-22 omonkey Merge contributor PRs #37 and #118 (partial), cherry-picked onto rewritten main
dba8cfa 2026-11-22 ComputerScape04 shader: clean compiler warnings and ignore pdb files
1708508 2026-11-22 omonkey Merge #122 from ComputerScape04: compiler warning cleanups, ignore .pdb files
d59ed23 2026-11-22 omonkey CREDITS: record the #36 port's upstream (brcly/TomBox fork of drycheetah84/TomBox) and its licence status
aa14bfa 2026-11-22 omonkey Console libs: ntdll UMS (3) and wldp watermark/production rows implemented
d585b94 2026-11-22 omonkey Fix #109: recover poisoned shader_xlat mutexes; Fix #114: refuse MIMG R128/TFE/LWE instead of ignoring them; BLOCKERS notes for #104 #115
bd880fc 2026-11-22 omonkey Fast scene path for every title: fast_run scenes (Trackmania, Terraria, Fortnite, Minecraft, XDefiant), test overlays + seeded test save roots, batch_test defaults to it (-FullBoot for the old path), launcher -LogDir/-Drive; BLOCKERS: scene-run blockers
5339418 2026-11-22 adriaespii tombox-xshader: parse 24-byte ISGN/OSGN signatures and synthesize Durango vertex fetch attributes
208d630 2026-11-22 jawed xshader: load the vertex and instance index into v0 and v3 of a vertex shader
853169c 2026-11-22 adriaespii xg_x: lay out stencil-only views 22/47/120 as full depth-stencil planar surfaces
c8a8a8f 2026-11-22 omonkey Merge contributor PRs #120 #125 #124
e5fe043 2026-11-22 omonkey Console libs: ConIoSrv ServerDllInitialization implemented (console DLL on console-sandbox)
a415541 2026-11-21 Isaac Wood MTBUF d16: RDNA2-measured conversions replace the ISA-only lowering
9ab6f59 2026-11-22 omonkey Merge #133 from SchmashinPrunes: MTBUF d16 conversions measured on RDNA2 hardware (resubmit of #41)
6950f9a 2026-11-22 Michael Kelly Test mesh LDS writes across a reconverged barrier
3ed2c06 2026-11-22 Michael Kelly Test reconverged mesh barriers around masked exports
3440221 2026-11-22 Michael Kelly Cover sparse vertex fetch input register mapping
f181815 2026-11-22 Michael Kelly Render sparse vertex attributes through the D3D12 input layout
8771f98 2026-11-22 omonkey Merge #128 and #131 from thatirishguyyy: mesh LDS barrier and sparse vertex fetch tests
9584ffa 2026-11-22 Lex_021 Translate RDNA2 GWS semaphore counter operations
2ab6155 2026-11-22 omonkey Merge #134 from lex021: GWS semaphore counter ops (ds_gws_init / sema_v / sema_br) for straight-line wave32 compute
c95a909 2026-11-22 omonkey Exec mask: keep full exec, reduce only s_wqm writes to exec; Fortnite derived PSO use-after-free fix in register_pso
7ef29e7 2026-11-22 omonkey d3d12_xs: PSO registry owns semantic names, drops CachedPSO, holds root signature (Fortnite derived PSO fix); exec mask fix; r21
df91928 2026-11-22 BED Pass the amplification shader's Y and Z group counts to DispatchMesh.
103016f 2026-11-22 BED Translate geometry-shader line and triangle adjacency inputs.
57b24e7 2026-11-22 BED Read compute NumThreads from PSVRuntimeInfo2 when the ISA preamble is zero.
7e41abe 2026-11-22 omonkey CONTRIBUTING: note the GNU target's fmaf breaks the FMA test references (#126)
9865896 2026-11-22 omonkey Merge #136, #137, #139 from Yeetbot159: DispatchMesh Y/Z group counts, GS adjacency inputs, PSV compute NumThreads fallback; CONTRIBUTING toolchain note (#126)
f10db75 2026-11-22 omonkey gcxaudioext: publish acp_state module (fixes clean-clone build E0583)
8421e17 2026-11-22 Lex_021 Keep the restored ACP source as text
dfcfc1f 2026-11-22 Lex_021 Provide pinned Opus sources for the Windows workspace check
65ca6dd 2026-11-22 omonkey Merge #143, #145 from lex021: keep acp_state.rs as text (NUL escape), pinned Opus sources for the workspace check
b5a755e 2026-11-22 omonkey r23: frame CSV hitch counters + title QPC game clock (frame_ev), pre-translate pack index (xlat_index), graphics bind-wait default (no skipped draws), xmem heapacct off-switch, dev_stub hit counter, TM hang diagnostics (env-gated)
eb18cd3 2026-11-22 Lex_021 Keep SOP2 coverage focused on 32-bit extraction
734cda4 2026-11-22 BED Order consecutive LDS accesses in a one-wave compute group.
860d34b 2026-11-22 BED Lower GFX7 legacy min and max with a compare so a NaN in source 1 stays NaN.
9d211b5 2026-11-22 BED Lower legacy max with v_cmp_ge_f32 and keep the compare in s104:s105.
fcb99ba 2026-11-22 BED Keep the other three bytes of the pkaccum destination.
595eb3a 2026-11-22 MeNoSkills d3d12_xs: read placed-texture memory through fault-safe snapshots so a racing decommit skips the upload instead of crashing
360cd07 2026-11-22 MeNoSkills d3d12_xs: keep direct reads by default, fault-safe snapshots only with TOMBOX_UPLOAD_SAFE_READ=1
1a1e283 2026-11-22 omonkey Merge #135 (lex021), #138, #140, #141 (Yeetbot159), #146 (MeNoSkills): s_bfe32 tests, one-wave LDS write ordering, GFX7 legacy min/max and pkaccum lowering, opt-in fault-safe texture reads
2be7211 2026-11-22 omonkey r24: edited shader variants via pack, prewarm priority-inversion fix, bind-wait default verified (0 skipped draws), cbv_raw CHECK/CLAMP/INHERIT diagnostics (env-gated), BLOCKERS: root CBV inheritance
4160c9a 2026-11-22 MiguelMatMar tombox-xshader: translate the remaining 32-bit image atomics and honour GLC
a689c70 2026-11-21 GDAgent Team compat-common: optional locale= in tombox.cfg sets the user language a title sees
1570be5 2026-11-21 GDAgent Team d3d12_xs: closing the game window ends the title
604b5a1 2026-11-22 omonkey Merge #147 (MiguelMatMar), #151, #154 (GDAgent): remaining 32-bit image atomics with GLC, opt-in locale= override, closing the window ends the title
789e8fb 2026-11-22 omonkey console libs 2143/2155: xtfdiaginfo, xtfeventserver, coniosrv, ntdll UMS, wldp, gamecore V7, audio position fix
afccebe 2026-11-22 Isaac Wood tombox-xshader: bit-exact RDNA2 f16 transcendentals (rcp, rsq, sqrt, log, exp, sin, cos)
d93b78c 2026-11-22 omonkey tombox-xshader: TOMBOX_XS_F16_NATIVE=1 restores the native f16 transcendental approximations for perf A/B; part of the xlat pack key
18fca55 2026-11-22 omonkey compat-common: build online_ready_up_replay only with tombox_backend_data, matching the online modules it uses
10f9b33 2026-11-22 MeNoSkills tombox-xshader tests: skip mem cases whose shader model the host D3D12 runtime does not support
de7e565 2026-11-22 BED Unpack the RDNA2 viewport index from the layer dword.
cd5817b 2026-11-22 omonkey Merge #150 (SchmashinPrunes), #155 (MeNoSkills), #156 (Yeetbot159): bit-exact RDNA2 f16 transcendentals with TOMBOX_XS_F16_NATIVE opt-out, shader-model skip in exec_mem, viewport index from the layer dword; gate online_ready_up_replay on tombox_backend_data
4c78bb8 2026-11-22 omonkey r25: chad FFT/IFFT + buffer bounds, xapu Opus/PCM/resampler, xtfdiaginfo, xtfeventserver, coniosrv, console-sandbox modules, TM hang capture mode (env-gated hang_cap)
0a9ebcd 2026-11-22 omonkey progress: #157 independent RTX 4070 Ti hw verification (hw_ 38/38, hw_db_count 4/4 at 604b5a1) recorded for d3d12x slots 29, 44, 64, 70, 74, 85, 86, 101; Fortnite runtime-log part still open
e4505ea 2026-11-22 AntonioEspana tombox-xshader: opt-in user-data CBV fallback for shaders without RTS0
f9fd22f 2026-11-22 GDAgent Team xshader: add ds_write_b16_d16_hi and buffer_store_format_xy/xyz
e239468 2026-11-22 GDAgent Team xshader: saturate D16 integer image loads and look through half packing in type inference
d7d72fe 2026-11-22 GDAgent Team xshader: honour MIMG UNORM and A16, and refuse MUBUF LDS/TFE
5d01d45 2026-11-22 omonkey docs: BLOCKERS entry for uninitialised RTS0 root-constant user SGPRs
43d17b0 2026-11-22 omonkey Merge contributor PRs #158 #159 #160 #161 and RTS0 SGPR blocker entry
dbe89c8 2026-11-22 omonkey progress: CommandList slots 64 and 70 back to partial to match their notes (Fortnite runtime-log part still open)
5d5bc33 2026-11-22 MeNoSkills d3d12_xs: guard reads of placed-texture memory with a fault-recovering call instead of copying (root fix for #110)
439826e 2026-11-22 MeNoSkills d3d12_xs: narrow the fault guard to the declared title range, log recovered faults, take plain fn jobs, register last
fe391fe 2026-11-22 omonkey Merge #153 (MeNoSkills): range-scoped fault guard for placed-texture reads, root fix for #110
1e037e7 2026-11-22 GDAgent Team d3d12_xs: map Xbox indirect argument 107 (0x6b) to DISPATCH_MESH
a44b9c7 2026-11-22 GDAgent Team d3d12_xs, compat-common: opt-in client_rect=backbuffer window size and mouse scaling
d04b9e0 2026-11-22 GDAgent Team d3d12_xs: emulate console depth-stencil textures that also allow UAV access
322f313 2026-11-22 GDAgent Team d3d12_xs: create a placed texture committed when its PC layout outgrows the title heap
9cd1305 2026-11-22 omonkey Merge GDAgent PRs #162 #163 #165 (default off) #166: DISPATCH_MESH indirect arg, opt-in client_rect, depth+UAV shadow, committed fallback
c697a1e 2026-11-22 ComputerScape04 shader: infer root signature resource bindings for shaders without RTS0
6176044 2026-11-22 ComputerScape04 d3d12_xs: remove unused AtomicUsize import and unnecessary mut and unsafe qualifiers
a11d361 2026-11-22 omonkey d3d12_xs: bypass the xlat pack index while TOMBOX_INFER_ROOT_SIG is set
3b91864 2026-11-22 omonkey tools: publish_source skips .pr/ dirs and .diff/.patch files
444014d 2026-11-22 omonkey Merge #129 (ComputerScape04): opt-in root-signature inference for shaders without RTS0; #149 lint cleanup
d751a89 2026-11-22 GDAgent Team tombox-xshader: route raw UAV buffer atomics to ubuf instead of colliding utb
1e77fcc 2026-11-22 omonkey Merge #169 (GDAgent): raw UAV buffer atomics use ubuf instead of a colliding utb
fa28145 2026-11-22 MeNoSkills d3d12_xs tests: enable experimental shader models before the first real device is created
202c763 2026-11-22 omonkey Merge #171 (MeNoSkills): enable experimental shader models before the first test device
85d4b0b 2026-11-22 omonkey r25: gpu_va keeps overlapping larger ranges (smallest-containing lookup); first-call logs for CL 70/74/85/86/101; cbv_raw TOMBOX_CBV_RAW auto/on/off (alias TOMBOX_CBRAW, #164 by GDAgent Team) + root_from_shaders through patch_blob; shared prelaunch/runtime pack key (KEY_VERSION 2, one-time recompile) + translator tag stale check; frame CSV timed flush; res_pool (TOMBOX_RES_POOL=1, default off); f861 hang capture tooling
98d3e72 2026-11-22 omonkey progress: d3d12x CommandList 64/70/74/85/86/101 and Device 29 done (HW-verified on 3 GPUs, no title call in fast_20261122_120610_fortnite_fn_r25_slots); Device 44 stays partial (two PC-observable placement gaps in BLOCKERS)
0fdc0dc 2026-11-22 omonkey console libs: offline network view (netview), xtfdiaginfo #10 sections, XVMAudioService round 7, pixrt10 slots 7/8
d22f894 2026-11-22 GDAgent Team tombox-xshader: support 256-thread mesh shaders and per-primitive PSG1 outputs
23505f0 2026-11-22 adriaespii tombox-xshader: map root Constants parameter values onto user SGPRs on Scarlett static path
f0213ec 2026-11-22 omonkey Merge #172 (GDAgent) and #170 (adriaespii): 256-thread mesh shaders with PSG1 per-primitive outputs; root Constants user SGPRs on the static Scarlett path
01a7018 2026-11-22 IseaMan docs: close the s_bfe width blocker from the ISA pseudocode and LLVM folding
6ef1fea 2026-11-22 omonkey Merge #178 (iseaprojects): close the s_bfe width blocker with ISA and LLVM evidence
a70dcb0 2026-11-22 omonkey netview: bind/listen/accept/closesocket socket-table hooks, emulator listeners reported
3159ffa 2026-11-22 omonkey netview: process-wide shared WFP store across DLLs
7047013 2026-11-22 GDAgent Team d3d12_xs: drop primitive topology subobject from mesh shader PSO streams
c852615 2026-11-22 AntonioEspana tombox-xshader: address buffer atomics with SOFFSET and IDXEN+OFFEN
190a4b3 2026-11-22 omonkey Merge #176 (GDAgent) and #177 (AntonioEspana): drop PRIMITIVE_TOPOLOGY from mesh PSO streams; buffer atomics with SOFFSET and IDXEN+OFFEN
d759b31 2026-11-22 omonkey r27: translator batch (#170 root-constant user SGPRs, #172 256-thread mesh, #176 mesh topology drop, #177 buffer-atomic SOFFSET); command-list PC-slot fixes (#175 + 37 more sites via cmdlist_wrap::pc_method); driver PSO library (TOMBOX_PSO_LIBRARY=0 off); Device 44 placement fixes (exact view copies, tiled MSAA aliasing); net_guard socket/WFP hooks; d3d12x 200/200
7748afd 2026-11-22 GDAgent Team d3d12_xs: match mesh PSO primitive topology to PSV0 and drop empty legacy stages
0d5f02a 2026-11-22 omonkey Merge PR #176 update (GDAgent Team): mesh PSO topology from PSV0, drop empty legacy stages
12ba607 2026-11-22 omonkey r28: mesh PSO topology from PSV0 + SUB_PRIMITIVE_TOPOLOGY 14 (#176 update); 16 more Xbox-vtable mis-slot fixes in injected compute paths (sys_ts, ts99, cl2_state predication); PSO library totals every 10 s, worker events on the title frame, 250 ms CSV flush
a541bbb 2026-11-22 omonkey compat-common os_host: pin modules during import sweeps (fix xmem+0x265e01 AV on module unload race)
27122f7 2026-11-22 omonkey XVMAudioService round 8: guest notifications, session events, enumerated sessions 0x77/0x78
f8e28c9 2026-11-22 GDAgent Team d3d12_xs: remap Xbox Series PSO stream subobjects when console-numbered
811c96f 2026-11-22 GDAgent Team d3d12_xs: classify PSO streams by record size and map console subobjects 25-30 from the console parse table
1a45c7d 2026-11-22 omonkey d3d12_xs: cite xgs12_10d parser and handler RVAs in console subobject decisions; BLOCKERS for #168 25-30
47ee74d 2026-11-22 omonkey Merge PR #168 (GDAgent Team): classify PSO streams by record size, map console subobjects 25-30
132d8cc 2026-11-22 omonkey netview: WFP object model and formatters, AcceptEx/ConnectEx and SYN_SENT tracking; xbtp console host 169.254.66.1 local
30a1b53 2026-11-22 omonkey r29: removed-device guard, PSO prewarm + library save fixes, gpu_va nested ranges, netview/WFP + socket hooks, xmem sweep race fix, msimg32 test fix, res_pool cfg
233d56f 2026-11-22 Isaac Wood tombox-xshader: bit-exact RDNA2 f32 rcp, rcp_iflag, rsq, sqrt
13ae0c6 2026-11-22 Isaac Wood tombox-xshader: bit-exact RDNA2 f32 log, exp, sin, cos
227e258 2026-11-22 Isaac Wood tombox-xshader: bit-exact RDNA2 f64 rcp, rsq, sqrt
b9055dd 2026-11-22 adriaespii xshader: ignore unmapped position export 13 without SV_ClipDistance/SV_CullDistance
05c1caf 2026-11-22 omonkey THIRD_PARTY: credit AMD rocjitsu for the #180 f32 transcendental models
f850f50 2026-11-22 omonkey Merge #180 (SchmashinPrunes) and the export-13 half of #181 (adriaespii): bit-exact RDNA2 transcendentals, unmapped position export 13
42f449c 2026-11-22 omonkey net_guard: chain ext extension-pointer guard onto shared WSAIoctl hook (11/11); late_load skips host-injected modules
3828799 2026-11-22 MeNoSkills tombox-xshader: lower ds_swizzle_b32 bit and quad modes in compute shaders
f6639fe 2026-11-22 MeNoSkills tombox-xshader: lower ds_swizzle_b32 rotate and FFT modes in compute shaders
cde6015 2026-11-22 MeNoSkills tombox-xshader: translate s_getpc_b64 + s_add_u32 + s_addc_u32 + s_setpc_b64 as a direct branch
42511d7 2026-11-22 omonkey Merge #184, #186 and #185 (MeNoSkills): ds_swizzle_b32 in compute (bit, quad, rotate, FFT); PC-relative far jumps as direct branches
5a25b00 2026-11-22 brcly d3d12_xs: answer the Xbox Series Device8 IID from the device wrapper
05f7422 2026-11-22 omonkey THIRD_PARTY: credit brcly/TuxBox reference patches (Apache-2.0) and add the licence text
baadb80 2026-11-22 omonkey Merge brcly/TuxBox port: Xbox Series Device8 IID in the device wrapper (Apache-2.0)
1f37ca3 2026-11-22 MeNoSkills CI: exclude the three crates that need local data files from the workspace check, and name the missing file in their build scripts
b2a77f3 2026-11-22 MeNoSkills tombox-xshader tests: pin the FLAT/GLOBAL/SCRATCH offset width against the DLC, GLC and SLC bits
59c2475 2026-11-22 Isaac Wood RDNA2 silicon evidence for 16 held-back or unverified instructions
a4cc592 2026-11-22 omonkey Merge #189, #190 (MeNoSkills) and #191 (SchmashinPrunes): workspace check excludes data-file crates; FLAT offset width test; RDNA2 silicon replay tests
c4b7607 2026-11-22 omonkey pixrt10: ETW controller layer for slot 0 (StartTraceW/EnableTraceEx2/rundown/stop, real calls, console failure path); console-sandbox in-process RPC server + COM/WinRT registry; netprofmsvc_c connectivity aggregation (parked)
b17ef2c 2026-11-22 GDAgent Team tombox-xshader, d3d12_xs: support 64-bit SM 6.6 image atomics with OPTIONS9 gate
267d08e 2026-11-23 GDAgent Team d3d12_xs: probe OPTIONS9 only on objects that answer IID_ID3D12Device
c2a3522 2026-11-22 omonkey Merge #173 (GDAgent Team): 64-bit SM 6.6 image atomics behind an OPTIONS9 gate
50a13be 2026-11-23 Isaac Wood tombox-xshader tests: replay RDNA2-recorded image atomic results
c4e72f0 2026-11-22 omonkey Merge #195 (SchmashinPrunes): replay RDNA2-recorded image atomic results
06110e0 2026-11-23 Isaac Wood rdna2_silicon: compare f16 FMA NaN results as a class on hardware GPUs
6e6095f 2026-11-22 omonkey pixrt10: capture session 0x1a94c, XgsKmd10 IOCTL 0x7900c1e0 in xgs_kmd_emu, rows 1/2 proposed unsupported (admin/console driver); XVMAudioService 0x61/0x64-0x66/0x7b/0x2d oversize
9e62058 2026-11-22 omonkey Merge #197: compare f16 FMA NaN results as a class on hardware GPUs (fixes #194)
2e981c5 2026-11-22 GDAgent Team tombox-xshader: translate image_gather4_l with an explicit LOD
9a77dd6 2026-11-22 omonkey BLOCKERS: record that the gather4_l footprint ignores sampler bias and LOD clamp
2eda8b0 2026-11-22 omonkey Merge #196: translate image_gather4_l with an explicit LOD
a7f858d 2026-11-23 TQuentinD xshader: v_med3_f32 with a NaN operand is MIN3 with the NaN ignored
8e4abb5 2026-11-23 TQuentinD xshader: inline float constants in f16 operands
54db2e3 2026-11-22 GDAgent Team tombox-xshader: translate the RDNA2 VOP3P packed 16-bit integer ops
2820575 2026-11-21 TQuentinD xshader: check RDNA2 opcode tables against gfx1013 references; fix f16 FMA names
f04f768 2026-11-22 omonkey leonida tests: drop trailing comments from the new vectors
24e6c10 2026-11-22 omonkey Merge #203, #202, #198, #201: v_med3_f32 NaN, f16 inline constants, VOP3P packed int16 ops, gfx1013/gfx7 opcode table check
28f54b2 2026-11-22 GDAgent Team tombox-xshader: decode and translate the LDS D16 loads and ds_write_b8_d16_hi
3f728a6 2026-11-23 TQuentinD xshader: the SMEM immediate is signed
0f84770 2026-11-21 TQuentinD Host input for the virtual pad: XInput controllers, keyboard mapping, rumble
35c979b 2026-11-23 TQuentinD rdna2-hwprobe: VALU/SALU probe kernels for f16 FMA, s_bfe widths, NaN rules and norm conversions
52b2890 2026-11-22 Isaac Wood rdna2-hwprobe: check the measured NaN rules in valu_probe.py show
17dd550 2026-11-23 TQuentinD rdna2-hwprobe: README section for valu_probe.py and its gfx1036 results
f9c3ec5 2026-11-22 omonkey Drop trailing comments from the new pad_host key table, the SMEM offset decode and valu_probe.py
563322a 2026-11-22 omonkey pad_host: host XInput and keyboard input are opt-in (TOMBOX_PAD_XINPUT / TOMBOX_PAD_KEYS = 1 or auto, or the pad_xinput / pad_keys cfg entries)
ad24144 2026-11-22 omonkey Merge #205, #206, #207, #208: LDS D16 loads, signed SMEM offset, opt-in host pad input, valu_probe.py
a1ff5be 2026-11-22 omonkey r31: Trackmania race renders by default (MSAA dim, Durango VS vertex id, FMASK identity, CBV copy default), mdbg exception cap fix, Terraria table-CBV raw views
9a782d2 2026-11-23 TQuentinD compat-common: XTaskQueue / XAsync conformance scenarios as tests
439a010 2026-11-22 omonkey BLOCKERS: record the XTaskQueue / XAsync conformance differences from #211
a4c2355 2026-11-22 omonkey Merge #211: XTaskQueue / XAsync conformance scenarios as tests
3de1b1f 2026-11-22 omonkey r31.1 hotfix: game-scan runtime prewarm opt-in (TOMBOX_GAME_SCAN_PREWARM=1); it exhausted Trackmania's title memory and aborted map load
d6ae2cb 2026-11-23 TQuentinD compat-common tests: keep TOMBOX_DXC and TOMBOX_DXC_DIR for the nested DLL build
c16f0fa 2026-11-22 omonkey Merge #212: keep TOMBOX_DXC and TOMBOX_DXC_DIR for the nested DLL build in compat-common tests
a246352 2026-11-23 TQuentinD xshader: v_mad_f32 flushes a subnormal result again after the output modifier
2f1a94c 2026-11-22 omonkey Merge #213: v_mad_f32 flushes a subnormal result again after the output modifier
689f690 2026-11-23 TQuentinD xshader: lower ds_mskor_b32 and ds_wrxchg2_rtn_b32 on compute LDS
44597ab 2026-11-22 omonkey Merge #214: lower ds_mskor_b32 and ds_wrxchg2_rtn_b32 on compute LDS
8d6f934 2026-11-22 omonkey xshader: an f16 output modifier acts only with IEEE 0 and f16 denormals flushed
ab3fcc8 2026-11-22 omonkey xshader: image_get_resinfo past the last level returns 0 size, keeps levels
e35aa5a 2026-11-22 omonkey Merge port/leo-f16omod: f16 output modifier follows MODE (Leonida port)
7e85f79 2026-11-22 omonkey Merge port/leo-resinfo: resinfo past the last level (Leonida SHADER-388)
691b84b 2026-11-22 omonkey xshader: packed int16 VOP3P takes float inline constants, mad clamp saturates
a7a98e0 2026-11-22 omonkey r32c: Trackmania campaign scene, PSO root-signature fix, pso_lib final-root key, DXC 35% faster, xmem title ledger, milestones per title
c998b44 2026-11-22 omonkey Merge port/leo-pk16: packed int16 VOP3P float constants and mad clamp (Leonida SHADER-302)
dfed19f 2026-11-22 omonkey README: per-title milestones and tested titles (Trackmania, Terraria, Minecraft, XDefiant)
f6ae886 2026-11-22 omonkey Merge commit 'c998b447438e3b6c9fbad8f5f1d75da575ce752f'
8e9987b 2026-11-23 TQuentinD ci: run the xshader exec tests in four parallel jobs
55011fb 2026-11-23 TQuentinD ci: keep rust-core as the single gate over the test jobs
3eac085 2026-11-22 alpha Match 19041 XMem hysteresis query width at 4 GiB
8e8a97c 2026-11-22 alpha Reset the XMem free hook when clearing the allocator
9475037 2026-11-22 alpha Reject invalid XMem allocation statistic types on build 19041
9e8d6d7 2026-11-23 azerioxfr CONTRIBUTING: list the hooked and unsupported export statuses
1dc9fff 2026-11-22 omonkey Merge #215, #217, #218, #219, #220: CI exec shards with rust-core gate, XMem 19041 fixes, CONTRIBUTING statuses
a615344 2026-11-22 omonkey gdk save: log the first call of each XGameSave slot
417f934 2026-11-22 alpha Port build 19041 XMem partition and in-place heap creation
84987a6 2026-11-22 alpha Query partition page counts in the 19041 XMem statistics layout
5603d8c 2026-11-23 alpha Report the emulated title frame range through both 19041 layout queries
6e9eb2f 2026-11-23 azerioxfr pr_gate: check added lines that start with ++
fe843fa 2026-11-23 azerioxfr xshader: exec test for v_pk_mul_lo_u16, v_pk_ashrrev_i16 and packed int16 min/max
d1fe8b5 2026-11-23 omonkey BLOCKERS: flaky d3d12_xs alias title-heap unmap/free test
b7f137f 2026-11-23 omonkey Merge #224, #226: pr_gate checks ++ lines, packed int16 mul/shift/minmax exec test; BLOCKERS flaky alias test
ee06399 2026-11-23 azerioxfr progress: 36 RDNA2 mnemonics verified by existing exec tests
6d0e24a 2026-11-23 azerioxfr progress: v_pk_mul_lo_u16, v_pk_ashrrev_i16 and packed int16 min/max verified
ac7cd85 2026-11-23 azerioxfr tombox-archive: warn when a large entry claims zero compressed bytes
ed9863a 2026-11-23 azerioxfr BLOCKERS: mark the xdxr ds_swizzle and SOFFSET atomic refusals fixed
bfd70be 2026-11-23 omonkey Merge #228, #230, #231: progress mnemonics, tombox-archive zero-compressed warning, BLOCKERS xdxr refusals fixed
f73fd27 2026-11-23 omonkey BLOCKERS: debug-layer probe result (r32f_9), r32f Trackmania board and CB findings
ad0fbf3 2026-11-23 omonkey Merge commit 'bfd70bedcecef75b0b315cf4a533c7f418ec85b7'
283ac34 2026-11-23 azerioxfr tombox-xshader: decode and translate the MUBUF D16 byte/short loads and d16_hi stores
6a1a4b1 2026-11-23 omonkey r32g: write-watch hash-confirmed uploads, unwatched resources on the event path, submit-time upload hook (stage B), xmem title accounting, opt-in ww_heap, XGameSave slot logging, XMem 19041 partition heaps and stats
4796d81 2026-11-22 alpha Decode stream-zero OSG5 output signatures on both Xbox shader paths
4ebf862 2026-11-23 omonkey Merge #222: decode stream-zero OSG5 output signatures on both Xbox shader paths
9226252 2026-11-23 omonkey BLOCKERS: Fortnite GDK hooks and fortnite_match inert without local offsets; ww_heap 4-slot limit
451fe11 2026-11-23 omonkey Record the r32g batch authorship (#221, #223, #225, XGameSave slot logging)
d66ae9c 2026-11-23 omonkey Merge commit '451fe119fa8b6e921b889cc44e3d77754d27995a'
82e330a 2026-11-23 omonkey pr_gate: reject title RVA-style constants in compat crates
902209a 2026-11-23 omonkey pr_gate: limit the RVA rule to 6-8 significant hex digits
df8e0c0 2026-11-23 omonkey Merge commit '902209ac26cbacae18d4c96da4d8a29d3ffc03bd'
4e8fbbf 2026-11-23 azerioxfr BLOCKERS: the run-35 system-function gap list is closed (671/671)
5544d41 2026-11-23 azerioxfr BLOCKERS: tests/amp.rs is ignored without the console fixtures, the PSV0 v1 gap stays open
6e81ffc 2026-11-23 omonkey Merge #259 and #260: BLOCKERS updates for the run-35 system-function list and tests/amp.rs
e70ae11 2026-11-23 omonkey BLOCKERS: replace title RVAs and offsets with symbolic references (exact values kept in local notes)
1172d5d 2026-11-23 omonkey Merge commit '6e81ffc4a05788a84c29e589370020573d645862'
e67e3d4 2026-11-23 omonkey r32h: residency cache, passcap ps_ms trigger, xmem fixes, title offsets moved to local JSON, MUBUF d16 (#235)
d8d37e4 2026-11-23 omonkey Record #235 authorship (MUBUF D16 byte/short loads and d16_hi stores)
bcecd4c 2026-11-23 omonkey BLOCKERS: Fortnite GDK hooks resolve from Symbols.bin (r32i)
f10c620 2026-11-23 omonkey BLOCKERS: board second-resource entry in symbolic form
377a22a 2026-11-23 omonkey r32i: GPU queue/wait profiling, VRAM and upload logging, VA alias census, Symbols.bin resolution for GDK hooks, XAsync return codes (#266)
f552e9c 2026-11-23 omonkey BLOCKERS: board second-resource theory ruled out; mip lead
883d1c8 2026-11-23 TQuentinD gdk rt: XAsync return codes match the reference (6 conformance scenarios)
e3c62b7 2026-11-23 omonkey Merge #266: XAsync return codes match the reference
19806e8 2026-11-23 omonkey Merge commit 'e3c62b71bb8229be43029daca1569316cacb2220'
de1e1da 2026-11-23 omonkey r32j: per-kind GPU timing of injected and title non-draw commands, first-use log, per-mip pass capture, exact-start alias census, XAsync pending-Begin fix, full-boot harness offline handling
148cf0f 2026-11-23 omonkey r32k: calibrated CPU/GPU timeline profiler, save long-name hashing, 44 redirected file APIs, v_fract special values
c47dfc2 2026-11-23 azerioxfr BLOCKERS: the GWS counter ops are lowered, only the wait ops are refused
d3a5928 2026-11-23 azerioxfr BLOCKERS: the write-watch submit hook and the xmem reservations have landed
2d4cbd5 2026-11-23 omonkey Merge #274 and #275: BLOCKERS updates for the GWS counter ops and the write-watch submit hook
fdd426f 2026-11-23 omonkey docs: BLOCKERS gap for rectlist expansion on stream-built PSOs
4eec5e8 2026-11-23 omonkey BLOCKERS: XUser pad association and token requests (r32m)
20f9340 2026-11-23 omonkey Merge remote-tracking branch 'origin/main'
f89e410 2026-11-23 omonkey r32l: gpuprof 2 s interval cap + longest events + b014 draw args, passcap 3D/cb dumps, XUser change/device/audio registries with real unregister-wait, gdk_multi_dll test staging fix
3bd526e 2026-11-23 TQuentinD d3d12_xs: keep the input layout alive in the pso_xrec skip test
619627b 2026-11-23 TQuentinD d3d12_xs: check the wrapped heap-properties slot by behaviour, not address
9f49e26 2026-11-23 TQuentinD d3d12_xs tests: the kept-source test ignores an entry left at a reused address
c40dd17 2026-11-23 omonkey Merge #276: d3d12_xs tests: three --lib tests that fail or crash on a release build (dangling temporary, function address, stale map entry)
67a4f3a 2026-11-23 alpha Use the configured console build for XMem allocation pools and statistics
61ffabe 2026-11-23 omonkey Merge #227: Respect the console build override for XMem pools and allocation statistics
ed803ac 2026-11-23 alpha Support the 19041 xvcmgmt four-counter SFR query
32098a1 2026-11-23 omonkey Merge #229: Implement the build 19041 xvcmgmt four-counter query
cce78b6 2026-11-23 gringoestrangeiro tombox-archive: block colliding entry names
bb15e8e 2026-11-23 omonkey Merge #244: tombox-archive: block colliding entry names
cf79741 2026-11-23 alpha Query partition and page attributes in build 19041 XMemVirtualQuery
dfaf4d5 2026-11-23 omonkey Merge #236: Port build 19041 XMemVirtualQuery partition and page classification
d4fac1d 2026-11-23 alpha Deliver in-process XBTP overlapped I/O to Windows completion ports
13542f0 2026-11-23 omonkey Merge #238: Support Windows I/O completion ports on in-process XBTP links
0118742 2026-11-23 alpha pixrt10: initialize the build 19041 title runtime and serve XBTP queries
8b16440 2026-11-23 omonkey Merge #250: pixrt10: initialize the build 19041 title runtime and serve XBTP queries
257bd02 2026-11-23 TQuentinD gdk rt: implement XTaskQueueRegisterWaiter and XTaskQueueUnregisterWaiter
5fdd4c9 2026-11-23 TQuentinD gdk rt: queue termination ends the work port, then the completion port
d39a963 2026-11-23 omonkey Merge #270: gdk rt: XTaskQueueRegisterWaiter and UnregisterWaiter, two-phase queue termination (5 conformance scenarios)
8dce1e7 2026-11-23 TQuentinD gdk rt: Cleanup after the completion callback for a call completed in Begin; XAsyncSchedule on a completed call
9b350ad 2026-11-23 TQuentinD gdk rt: XAsyncGetStatus(wait) waits for the completion signal, not for the status
132b5a7 2026-11-23 TQuentinD gdk rt tests: XAsyncGetStatus(wait) with a manual completion queue
139f94c 2026-11-23 omonkey Merge #210: gdk rt: XAsyncGetStatus(wait) waits for the completion callback, as the PC runtime does
d57140b 2026-11-23 TQuentinD gdk tests: interface ids the PC runtime serves and the emulator refuses (12 ids, 9 providers)
ab25c62 2026-11-23 omonkey Merge #279: gdk tests: 12 interface ids the PC runtime serves and the emulator refuses, one method table per family on the PC
1fac0aa 2026-11-23 GDAgent Team d3d12_xs: name the group counts of each Dispatch in the TOMBOX_DRED=2 report
89d07c1 2026-11-23 omonkey Merge #273: d3d12_xs: name the group counts of each Dispatch in the TOMBOX_DRED=2 report
fa9f1ca 2026-11-23 GDAgent Team d3d12_xs: drop gpu_va ranges when their resource is destroyed
0cc4e3b 2026-11-23 omonkey Merge #283: d3d12_xs: drop gpu_va ranges when their resource is destroyed
995ea5a 2026-11-23 GDAgent Team d3d12_xs: map the Xbox rect-list primitive topology type 0x13 to TRIANGLE in non-mesh PSO streams
c606691 2026-11-23 GDAgent Team d3d12_xs: expand rect-list draws on pipelines built from a pipeline stream
2014b35 2026-11-23 omonkey Merge #280: d3d12_xs: rect-list pipelines from PSO streams: map console topology type 0x13 and expand the draws to full rects
202b1d8 2026-11-23 TQuentinD Mesh: fill merged_wave_info TGsize and per-wave counts, ISA GS_ALLOC_REQ widths
9d565d6 2026-11-23 omonkey Merge #200: xshader mesh: full merged_wave_info in s3, GS_ALLOC_REQ field widths per the ISA (was #58)
a91690e 2026-11-23 TQuentinD xshader: v_mad_u64_u32 and v_mad_i64_i32 write 0 to sdst for a lane off in EXEC
dd1ca11 2026-11-23 omonkey Merge #237: xshader: v_mad_u64_u32 and v_mad_i64_i32 write 0 to sdst for a lane off in EXEC
5eb870f 2026-11-23 azerioxfr tombox-xshader: decode and translate v_qsad_pk_u16_u8, v_mqsad_pk_u16_u8 and v_mqsad_u32_u8
74ea579 2026-11-23 omonkey Merge #247: tombox-xshader: decode and translate v_qsad_pk_u16_u8, v_mqsad_pk_u16_u8 and v_mqsad_u32_u8
0e92311 2026-11-23 azerioxfr tombox-xshader: decode and translate v_movrelsd_2_b32 and v_swaprel_b32
380f755 2026-11-23 omonkey Merge #252: tombox-xshader: decode and translate v_movrelsd_2_b32 and v_swaprel_b32
b318331 2026-11-23 azerioxfr tombox-xshader: decode the remaining SOP1 saveexec ops and translate s_andn1/s_andn2_wrexec
0447a97 2026-11-23 omonkey Merge #242: tombox-xshader: decode the remaining SOP1 saveexec ops and translate s_andn1/s_andn2_wrexec
543583e 2026-11-23 azerioxfr tombox-xshader: decode and translate the FLAT/GLOBAL D16 sub-dword loads
3b3028b 2026-11-23 omonkey Merge #249: tombox-xshader: decode and translate the FLAT/GLOBAL D16 sub-dword loads
77ca205 2026-11-23 TQuentinD xshader: v_pk_add/sub/mad_i16 with CLAMP use sat_int, as the other 16-bit clamps do
e4c1b20 2026-11-23 omonkey Merge #267: xshader: v_pk_add/sub/mad_i16 with CLAMP use sat_int (wrong high lane on an NVIDIA RTX 5090)
ee977a5 2026-11-23 marquezini tombox-xshader: saturate the 24-bit multiplies on VOP3 clamp
2e7f5c3 2026-11-23 omonkey Merge #285: tombox-xshader: saturate the 24-bit multiplies on VOP3 clamp
3826705 2026-11-23 azerioxfr tombox-xshader: decode and translate the f32 IEEE division helpers
ebc3148 2026-11-23 omonkey Merge #272: tombox-xshader: decode and translate the f32 IEEE division helpers
4169a9f 2026-11-23 azerioxfr tombox-xshader: lower the GCN f32 clamp transcendental VOP1 ops
b24ac90 2026-11-23 omonkey Merge #281: tombox-xshader: lower the GCN f32 clamp transcendental VOP1 ops
e0169ae 2026-11-23 azerioxfr tombox-xshader: decode and translate buffer_load/store_format_d16_x and _d16_hi_x
8b15fe5 2026-11-23 omonkey Merge #263: tombox-xshader: decode and translate buffer_load/store_format_d16_x and _d16_hi_x
986c4fe 2026-11-23 azerioxfr tombox-xshader: decode and lower the DS src2 read-modify-write ops
6361ba0 2026-11-23 omonkey Merge #264: tombox-xshader: decode and lower the DS src2 read-modify-write ops
476dca4 2026-11-23 TQuentinD xshader: DXIL container digest in Rust, checked against DXC and the D3D12 runtime
0164cc3 2026-11-23 omonkey Merge #265: xshader: DXIL container digest in Rust, checked against DXC and the D3D12 runtime
193d416 2026-11-23 azerioxfr tombox-xshader: decode and translate image_sample_d_g16 and image_sample_d_cl_g16
e65f9ce 2026-11-23 omonkey Merge #282: tombox-xshader: decode and translate image_sample_d_g16 and image_samÔÇª
db38a61 2026-11-23 azerioxfr tombox-xshader tests: pin the gather4 bias and dmask refusals
de4317e 2026-11-23 omonkey Merge #248: tombox-xshader tests: pin the gather4 bias and dmask refusals
42074e0 2026-11-23 azerioxfr tombox-xshader tests: pin the image-dimension refusals
1a4745a 2026-11-23 omonkey Merge #255: tombox-xshader tests: pin the image-dimension refusals
bfaf07c 2026-11-23 azerioxfr tombox-xshader tests: pin the multisampled UAV load refusal
b946ac8 2026-11-23 omonkey Merge #256: tombox-xshader tests: pin the multisampled UAV load refusal
910bcb9 2026-11-23 MeNoSkills tombox-xshader tests: bit-exact pixel sampling cases for image_sample, _b, _d, _c_lz, _lz_o, gather4_lz and image_load_mip
be98784 2026-11-23 omonkey Merge #286: tombox-xshader tests: bit-exact pixel sampling cases for image_sample, _b, _d, _c_lz, _lz_o, gather4_lz and image_load_mip
d9fe75b 2026-11-22 MeNoSkills tombox-xshader: lower scratch (private memory) loads and stores to a per-thread array
4f1cd12 2026-11-23 omonkey Merge #284: tombox-xshader: lower scratch (private memory) loads and stores to a per-thread array
857e37e 2026-11-23 MeNoSkills tombox-xshader tests: bit-exact v_interp_p1_f32, v_interp_p2_f32 and v_interp_mov_f32 through a real vertex shader
9f353b9 2026-11-23 omonkey Merge #287: tombox-xshader tests: bit-exact v_interp_p1_f32, v_interp_p2_f32 and v_interp_mov_f32 through a real vertex shader
62c2d37 2026-11-23 azerioxfr tombox-xshader tests: pin the refusal of GS emit and cut on streams 1-3
6d86929 2026-11-23 omonkey Merge #232: tombox-xshader tests: pin the refusal of GS emit and cut on streams 1-3
388a7d6 2026-11-23 azerioxfr tombox-xshader tests: pin the s_sendmsghalt and ds_ordered_count refusals
eb557e5 2026-11-23 omonkey Merge #233: tombox-xshader tests: pin the s_sendmsghalt and ds_ordered_count refusals
16c9ddf 2026-11-23 azerioxfr tombox-xshader tests: pin the ds_swizzle stage and QDMode refusals
c06a3f7 2026-11-23 omonkey Merge #243: tombox-xshader tests: pin the ds_swizzle stage and QDMode refusals
0dcfef1 2026-11-23 azerioxfr tombox-xshader tests: pin the export-target refusals
0644a7a 2026-11-23 omonkey Merge #261: tombox-xshader tests: pin the export-target refusals
8650276 2026-11-23 azerioxfr tombox-xshader tests: pin the mesh export, entry-count and LDS refusals
9e731d3 2026-11-23 omonkey Merge #269: tombox-xshader tests: pin the mesh export, entry-count and LDS refusals
1e0fc23 2026-11-23 azerioxfr tombox-xshader: decode and translate the VOP3-encoded v_interp f32 forms
19d0be4 2026-11-23 omonkey Merge #253: tombox-xshader: decode and translate the VOP3-encoded v_interp f32 forms
16c6adc 2026-11-22 shdann xshader: resolve RDNA2 literal static samplers
4f61662 2026-11-23 omonkey Merge #183: xshader: resolve RDNA2 literal static samplers
19d9a76 2026-11-23 azerioxfr tombox-xshader: name and lower s_dcache_wb as a no-op
df310a4 2026-11-23 omonkey Merge #254: tombox-xshader: name and lower s_dcache_wb as a no-op
95905b3 2026-11-23 azerioxfr tombox-xshader: translate s_buffer_store_dword* into UAV writes
ecc06dd 2026-11-23 omonkey Merge #258: tombox-xshader: translate s_buffer_store_dword* into UAV writes
5982fb8 2026-11-23 azerioxfr tombox-xshader: decode and translate ds_read_b96, ds_read_b128, ds_write_b96 and ds_write_b128
00052ef 2026-11-23 omonkey Merge #240: tombox-xshader: decode and translate ds_read_b96, ds_read_b128, ds_wrÔÇª
a02a9c4 2026-11-23 azerioxfr tombox-xshader: decode and translate ds_wrap_rtn_b32 on compute LDS
a1d04e8 2026-11-23 omonkey Merge #245: tombox-xshader: decode and translate ds_wrap_rtn_b32 on compute LDS
4d69631 2026-11-22 GDAgent Team tombox-xshader: declare UAVs accessed with GLC or DLC globallycoherent
e543de8 2026-11-23 omonkey Merge #199: tombox-xshader: declare UAVs accessed with GLC or DLC globallycoherent
7f61ae8 2026-11-23 GDAgent Team tombox-xshader: seed merged_wave_info, vertex and instance index for merged vertex shaders
5c7d61c 2026-11-23 GDAgent Team tombox-xshader: seed merged wave info by program wave size
343c15a 2026-11-23 omonkey Merge #278: tombox-xshader: seed merged_wave_info, vertex and instance index for merged vertex shaders
a25e7d9 2026-11-23 GDAgent Team tombox-xshader: parse the FSSM root-binding table
0481c6b 2026-11-23 GDAgent Team tombox-xshader: opt-in FSSM-table fallback for shaders without a root signature
b17ecef 2026-11-23 omonkey Merge #289: tombox-xshader: read the FSSM root-binding table (opt-in fallback for shaders without RTS0)
79c5478 2026-11-23 azerioxfr tombox-xshader: decode and translate the MUBUF 64-bit integer buffer atomics
e086d23 2026-11-25 omonkey Merge #251: tombox-xshader: decode and translate the MUBUF 64-bit integer buffer ÔÇª
2bb0192 2026-11-23 TQuentinD xshader: 64-bit LDS atomics under a group lock, LDS sized from COMPUTE_PGM_RSRC2
547e1f3 2026-11-23 TQuentinD xshader lds64: COMPUTE_PGM_RSRC2 location keyed on the FSSM sub-version, LDS index wrap chosen at emission
b5cd984 2026-11-25 omonkey Merge #257: xshader: 64-bit LDS atomics under a group lock, LDS sized from COMPUTE_PGM_RSRC2
ea24558 2026-11-23 NemesisDawn Fix integer vertex selection in v_interp_mov_f32
d38321c 2026-11-25 omonkey Merge #291: Fix integer vertex selection in v_interp_mov_f32
1083485 2026-11-23 nickfmbg tombox-xshader: parse PSV0 hull and domain runtime info
b5c3e14 2026-11-25 omonkey Merge #295: tombox-xshader: parse PSV0 hull and domain runtime info
e4b43fb 2026-11-23 MeNoSkills tombox-xshader tests: bit-exact buffer_load/store_dwordx2/3/4, buffer_load/store_format_*, buffer_atomic_swap/umin and v_mbcnt_lo/hi
a0feb8f 2026-11-25 omonkey Merge #297: tombox-xshader tests: bit-exact buffer_load/store_dwordx2/3/4, buffer_*_format_*, buffer_atomic_swap/umin and v_mbcnt_lo/hi
2c4be57 2026-11-23 MeNoSkills tombox-xshader tests: s_swappc_b64 vertex fetch with two link registers and edge values, ds_append/ds_consume in the verified list, LDS-mode refusal pinned
5ec2fde 2026-11-25 omonkey Merge #298: tombox-xshader tests: s_swappc_b64 vertex fetch with two link registers, ds_append/ds_consume verified, LDS-mode refusal pinned
580ada0 2026-11-23 nickfmbg tombox-xshader: translate domain shaders (hardware VS stage)
3e70f78 2026-11-25 omonkey Merge #301: tombox-xshader: translate domain shaders (hardware VS stage)
5a37765 2026-11-23 TQuentinD tombox-xshader: decode the MIMG pck and float-atomic opcodes, translate pck on atomic images
06e683f 2026-11-25 omonkey Merge #302: tombox-xshader: decode MIMG pck and float-atomic opcodes, translate pck on atomic images
9e049f5 2026-11-23 enderrr9 tombox-xshader: decode the VOP3 v_interp f16 forms and refuse them with attr/chan/half
accb803 2026-11-25 omonkey Merge #304: tombox-xshader: decode the VOP3 v_interp f16 forms and refuse them with attr/chan/half
b57a976 2026-11-23 MeNoSkills tombox-xshader tests: bit-exact v_cmp_class_f32 / v_cmpx_class_f32 over every class, and buffer_gl0_inv / buffer_gl1_inv
36079a1 2026-11-25 omonkey Merge #305: tombox-xshader tests: bit-exact v_cmp_class_f32 / v_cmpx_class_f32 and buffer_gl0_inv / buffer_gl1_inv
1a19304 2026-11-23 BaggyALPHA tombox-xshader: read scalar NULL operands as zero
73db902 2026-11-23 BaggyALPHA tombox-xshader: preserve NULL semantics in SOPK source reads
15d9930 2026-11-25 omonkey Merge #308: tombox-xshader: read scalar NULL operands as zero
388b1d1 2026-11-23 nickfmbg tombox-xshader: generate the pass-through hull shader for tessellation
aac1f6e 2026-11-25 omonkey Merge #312: tombox-xshader: generate the pass-through hull shader for tessellation
b8432ab 2026-11-23 enderrr9 tombox-xshader tests: synthetic amplification + mesh pair rendered on WARP
d510bf1 2026-11-25 omonkey Merge #313: tombox-xshader tests: synthetic amplification + mesh pair rendered on WARP
53bb998 2026-11-23 enderrr9 tombox-xshader: keep amplification DispatchMesh Z at 1, dword 2 is the payload pointer
c1ee4bf 2026-11-25 omonkey Merge #315: tombox-xshader: keep amplification DispatchMesh Z at 1 (args dword 2 is the payload pointer)
3d90f6f 2026-11-23 enderrr9 tombox-xshader: amplification shaders always use the memory-based root ABI
d19af94 2026-11-25 omonkey Merge #316: tombox-xshader: amplification shaders always use the memory-based root ABI
81758f6 2026-11-23 alpha Translate RDNA2 scalar bit replication
1f01e1a 2026-11-25 omonkey Merge #318: Translate RDNA2 S_BITREPLICATE_B64_B32
9b7c93a 2026-11-24 ChAlanA tombox-xshader: fix RDNA2 M0 (124) vs NULL (125) scalar destinations
894e339 2026-11-25 omonkey Merge #328: tombox-xshader: fix RDNA2 M0 (124) vs NULL (125) scalar destinations
8f71091 2026-11-23 GDAgent Team tombox-xshader: SOP1 ops with a 32-bit result of a 64-bit source write one SGPR
2f32df0 2026-11-25 omonkey Merge #331: tombox-xshader: SOP1 ops with a 32-bit result of a 64-bit source write one SGPR
17087d4 2026-11-23 timmmyyyyyyy tombox-xshader: add v_bfe_u32 to the RDNA2 isa sweep
c9730be 2026-11-25 omonkey Merge #339: tombox-xshader: add v_bfe_u32 to the RDNA2 isa sweep
fb19085 2026-11-23 timmmyyyyyyy tombox-xshader: add v_bfe_i32 to the RDNA2 isa sweep
be634cf 2026-11-25 omonkey Merge #341: V bfe i32 sweep
d5cc080 2026-11-23 timmmyyyyyyy tombox-xshader: add s_bfm_b32 to the RDNA2 isa sweep
3cf92b5 2026-11-25 omonkey Merge #342: tombox-xshader: add s_bfm_b32 to the RDNA2 isa sweep
18c5d62 2026-11-24 BuHHuTlyx Translate RDNA2 scalar quad masks
4d1664a 2026-11-25 omonkey Merge #344: Translate RDNA2 scalar quad masks
d2cc33e 2026-11-23 MeNoSkills tombox-xshader: amplification payload byte/short stores were lost on hardware; edge-case AS tests
0af05c5 2026-11-23 MeNoSkills tombox-xshader tests: mesh shader sub-dword payload loads (ubyte/sbyte/ushort/sshort) extend as the ISA says
9101363 2026-11-24 BuHHuTlyx Prototype staging atomic amplification payload writes
2a4de1c 2026-11-25 omonkey Merge #345: Amp staged payload experiment
3e9676d 2026-11-23 GDAgent Team tombox-xshader: declare raw buffers as ByteAddressBuffer and address them by byte
36d9c51 2026-11-25 omonkey Merge #346: tombox-xshader: declare raw buffers as ByteAddressBuffer and address them by byte
b5dfa13 2026-11-24 Lex_021 Validate RTS0 range tables against their serialized extent
fbf6c52 2026-11-25 omonkey Merge #353: Validate RTS0 descriptor table ranges against the serialized data
f1e8d7c 2026-11-24 Isaac Wood rdna2-hwprobe: colour export and blend probe; RDNA2 keeps shader Inf/NaN in fp16 and R11G11B10 targets
dfd120b 2026-11-25 omonkey Merge #360: rdna2-hwprobe: colour export and blend probe; RDNA2 keeps shader Inf/NaN in fp16 and R11G11B10 targets
bf0f50c 2026-11-24 Isaac Wood rdna2-hwprobe: mip selection and resinfo probe; RDNA2 rules for gather4_l bias/clamp and 2D-array resinfo
b931581 2026-11-25 omonkey Merge #361: rdna2-hwprobe: mip selection and resinfo probe; RDNA2 rules for gather4_l bias/clamp and 2D-array resinfo
d4c2cc5 2026-11-24 Isaac Wood tombox-xshader: translate depth-compare samples with LOD clamp, bias and gradients
a5e467b 2026-11-25 omonkey Merge #362: tombox-xshader: translate depth-compare samples with LOD clamp, bias and gradients
ed3cd79 2026-11-24 Isaac Wood tombox-xshader: f16 omod in denorm mode 1 and underflow flush after the scale, from gfx1036 measurements
5ad807c 2026-11-25 omonkey Merge #363: tombox-xshader: f16 omod in denorm mode 1 and underflow flush after the scale, from gfx1036 measurements
7364bb4 2026-11-24 Isaac Wood tombox-xshader: v_mad_i64_i32 sdst is the sign of the exact result, as on gfx1036
cdc541d 2026-11-25 omonkey Merge #364: tombox-xshader: v_mad_i64_i32 sdst is the sign of the exact result, as on gfx1036
65893a4 2026-11-24 Isaac Wood xshader: guard f32 sub and f16 add/sub/mul against AMD host folding
3f1bc0f 2026-11-24 Isaac Wood xshader: bit-mask f64 trunc and zero-sign guards for AMD host folding
d0fabcb 2026-11-25 omonkey Merge #375: xshader: guard f32 sub and f16 add/sub/mul against AMD host folding (fixes three #333 failures)
0256576 2026-11-24 Ctrl-Alt-Haze tombox-xshader: exact v_cvt_rpi_i32_f32 floor and sign-saturated NaN for rpi/flr
474314f 2026-11-25 omonkey Merge #377: tombox-xshader: exact v_cvt_rpi_i32_f32 floor and sign-saturated NaN for rpi/flr
706a2b7 2026-11-23 Bexx1107 xg_xs: gate the console-oracle test modules behind an off-by-default feature
2813b2a 2026-11-25 omonkey Merge #310: xg_xs: gate the console-oracle test modules behind an off-by-default feature
62c525d 2026-11-24 Lex_021 Report Scarlett 2D resource tiling through XG slot 10
9b4d12d 2026-11-24 Lex_021 Follow upstream source style
2970171 2026-11-25 omonkey Merge #352: Implement GetResourceTiling for Scarlett 2D textures
ced7994 2026-11-23 enderrr9 d3d12_xs: eager blendclamp leaves R32G32B32A32_FLOAT targets alone
92c998d 2026-11-25 omonkey Merge #320: d3d12_xs: eager blendclamp leaves R32G32B32A32_FLOAT targets alone
1112c4a 2026-11-23 GDAgent Team d3d12_xs tests: size the non-oversize test textures from GetResourceAllocationInfo
a1d913d 2026-11-25 omonkey Merge #322: d3d12_xs tests: size the non-oversize test textures from GetResourceAllocationInfo
397866d 2026-11-23 GDAgent Team d3d12_xs tests: check every detiled element and time the 1 MB detile only in optimized builds
282d358 2026-11-25 omonkey Merge #323: d3d12_xs tests: check every detiled element and time the 1 MB detile only in optimized builds
ab6d485 2026-11-23 enderrr9 d3d12_xs tests: wait for the OS to unpin a destroyed title heap instead of assuming it is synchronous
c9deb12 2026-11-25 omonkey Merge #327: d3d12_xs tests: wait for the OS to unpin a destroyed title heap (flaky alias test)
da08a7e 2026-11-23 GDAgent Team d3d12_xs: skip compute pipelines quietly in the rect-list path
56944cf 2026-11-25 omonkey Merge #324: d3d12_xs: skip compute pipelines quietly in the rect-list path
6d19943 2026-11-23 GDAgent Team d3d12_xs: route command-list slots 131/132 by the caller's GXDK edition
68f2cf2 2026-11-24 GDAgent Team d3d12_xs: skip SIB call candidates that do not fit in the return-address window
d68b34a 2026-11-25 omonkey Merge #334: d3d12_xs: route command-list slots 131/132 by the caller's GXDK edition
945731f 2026-11-23 GDAgent Team d3d12_xs: device slots 127 (CreateCommittedResource3) and 133 (CreatePlacedResourceX with desc 1)
495cadf 2026-11-25 omonkey Merge #335: d3d12_xs: device slots 127 (CreateCommittedResource3) and 133 (CreatePlacedResourceX with a resource desc 1) for GXDK 2504+ callers
cf90d9e 2026-11-23 GDAgent Team xcrdapi: accept the newer IDStorageQueueX IID and CreateQueue at factory slot 10
5129361 2026-11-25 omonkey Merge #337: xcrdapi: accept the newer IDStorageQueueX IID and CreateQueue at factory slot 10
979615b 2026-11-23 GDAgent Team d3d12_xs: drop console-only command queue flags, keep DISABLE_GPU_TIMEOUT
9a1e9ba 2026-11-25 omonkey Merge #338: d3d12_xs: drop console-only command queue flags, keep DISABLE_GPU_TIMEOUT
8c481f4 2026-11-23 GDAgent Team d3d12_xs: opt-in GPU-based validation with TOMBOX_D3D12_GBV=1
0e6bde2 2026-11-25 omonkey Merge #348: d3d12_xs: opt-in GPU-based validation with TOMBOX_D3D12_GBV=1
d550bc0 2026-11-24 Lex_021 Treat D3D12 device removal as a failed fence wait
bd39bf8 2026-11-24 Lex_021 Follow upstream source style
03fea26 2026-11-25 omonkey Merge #351: Treat D3D12 device removal as a failed fence wait
f737cc1 2026-11-24 Lex_021 Preserve Scarlett rectangle coverage for each pivot vertex
75514dd 2026-11-25 omonkey Merge #372: Preserve RECTLIST coverage for all three corner positions
8ab4426 2026-11-24 Lex_021 Give native SRV and UAV creation the actual resource extent
f650947 2026-11-25 omonkey Merge #373: Expose the backing resource extent to native SRV and UAV creation
d2dad0c 2026-11-23 TQuentinD tombox-xshader: apply the S# LOD bias and clamp to the gather4_l footprint mip
6f2b507 2026-11-25 omonkey Merge #306: tombox-xshader: apply the S# LOD bias and clamp to the gather4_l footprint mip (follow-up of #196)
bd185fa 2026-11-23 TQuentinD xshader: initial MODE from PGM_RSRC1, reported as unknown when not located
8e7233d 2026-11-23 TQuentinD xshader: fixture scan for the PGM_RSRC1 location in FSSM containers
753e851 2026-11-23 TQuentinD xshader: RSRC2, descriptor and stage-slot probes in the PGM_RSRC1 fixture scan
4c7c544 2026-11-25 omonkey Merge #309: xshader: initial MODE from PGM_RSRC1, fixture scan for its FSSM location
974495f 2026-11-23 alpha Resolve the PIX driver marker using the Windows file code page
8a0a9b6 2026-11-25 omonkey Merge #319: Resolve PIX driver markers with the Windows file code page
acfede3 2026-11-23 TQuentinD gdk tests: XSystem and XGame title id conformance against the PC runtime (17 scenarios)
f50222c 2026-11-23 TQuentinD gdk system: serve XSystemGetAppSpecificDeviceId at slot 5 of XSystem e349bd1a
c3292cc 2026-11-23 TQuentinD gdk system: XGameGetXboxTitleId without a TitleId returns E_NOT_FOUND, checked before the output pointer
fb95d74 2026-11-25 omonkey Merge #300: gdk: XSystem and XGame title id conformance against the PC runtime (17 scenarios, 3 fixes)
adef185 2026-11-23 Bexx1107 gdk rt: fix XTaskQueue handle duplication, pool dispatch, monitors, close and manual terminate wakeup
becdf35 2026-11-23 Bexx1107 gdk rt: exit Dispatch(INFINITE) per-queue only after port termination marker runs
ba8e0e5 2026-11-25 omonkey Merge #307: gdk rt: fix XTaskQueue handle duplication, pool dispatch, monitors, close, and manual terminate wakeup (6 conformance scenarios)
6e44eae 2026-11-23 BaggyALPHA xbtp: honor the session QueryInterface output contract
74afeb3 2026-11-25 omonkey Merge #311: xbtp: honor the session QueryInterface output contract
e7377c9 2026-11-23 TQuentinD gdk rt: XAsyncCancel leaves completion to the provider; GetResult errors end the call; never-started blocks
8050509 2026-11-25 omonkey Merge #321: gdk rt: XAsyncCancel leaves completion to the provider, GetResult errors end the call, never-started blocks (4 conformance scenarios)
2e8e2b8 2026-11-23 GDAgent Team xmem: honour placeholder reservations in XMemVirtualAlloc
785a6df 2026-11-25 omonkey Merge #325: xmem: honour MEM_RESERVE_PLACEHOLDER and MEM_REPLACE_PLACEHOLDER in XMemVirtualAlloc
146286e 2026-11-23 GDAgent Team compat-common: hand out a stand-in IGameInput when the host has no GameInput
9af87c0 2026-11-25 omonkey Merge #326: compat-common: hand out a stand-in IGameInput when the host has no GameInput
0667441 2026-11-24 NemesisDawn tombox-archive: accept ZIP64 archives with long comments
daf7521 2026-11-25 omonkey Merge #340: Accept ZIP64 archives with long comments in zip-inspect
65e8691 2026-11-23 GDAgent Team tombox-xshader: bind literal static samplers through the FSSM table
c59666e 2026-11-23 GDAgent Team tombox-xshader: literal-sampler dataflow keys scalar control ops by opcode
2b3a132 2026-11-25 omonkey Merge #329: tombox-xshader: bind literal static samplers through the FSSM table
4fcfe13 2026-11-23 GDAgent Team tombox-xshader: take the user-data base from the FSSM stage slot
b96523a 2026-11-25 omonkey Merge #330: tombox-xshader: take the user-data base from the FSSM stage slot
b03b39b 2026-11-23 GDAgent Team tombox-xshader: size structured buffer elements from the FSSM stride
a618736 2026-11-25 omonkey Merge #347: tombox-xshader: size structured buffer elements from the FSSM stride (stacked on #289)
```



## L. Limitacoes desta analise

**L-01 — O historico usado nao e o historico do projeto.** O commit `ff39277` ("Add emulator source (code only; no game data, binaries or notes)") importa a arvore Rust inteira, e `410e01f` a move para a raiz. Nada do que foi escrito antes de 2026-11-21 esta em git. Consequencia direta: **19 dos 22 defeitos de secao D so podem ser datados como "import"**, nao como autoria; e varios trechos que parecem "corrigidos" foram na verdade **ja corrigidos antes** de entrarem no repositorio. Nao inventei autoria pre-import em nenhum ponto.

**L-02 — Nada foi executado.** Nao compilei o workspace, nao rodei `cargo test`, nao executei nenhum titulo. Todas as afirmacoes sobre "passa", "89/89", "22 testes de gamepad", "0/200 falhas" sao **repeticoes de texto de commit/mensagem/documento**, nao verificacoes proprias. Onde o commit cita medicao em hardware (RTX 4070 Ti, RX 9060 XT, gfx1036, RTX 5090, RDNA2), eu nao tinha essa maquina.

**L-03 — Sem rede.** Nao consultei GitHub: issues e PRs (`#15`, `#36`, `#104`-`#115`, `#169`-`#377`, `#235`, `#289`, `#333`, `#346`, `#347`) sao citados **apenas** como aparecem em mensagens de commit e em `docs/BLOCKERS.md`. Numero de estrelas, ranking de 76%, discussao de review e autoria de PR fora do git: **NAO VERIFICADO**. Em particular, a atribuicao de "RomainROCH in review" (D-11) e a de `Co-Authored-By: Claude Opus 5.5` sao transcritas da mensagem, nao confirmadas.

**L-04 — A comparacao de linhagem e estrutural, nao genealogica.** Comparei `git ls-files` dos dois clones locais e testei a existencia do objeto raiz em cada clone. Nao tentei `git log --all --graph` cruzado entre repositorios nem procuirei *patch-id* entre commits de TuxBox e xbox-emulator, o que seria a prova definitiva de reuso. O que afirmo e o que o dado sustenta: caminhos disjuntos, contagem igual, e tres commits de importacao de patches creditados. A afirmacao do README sobre `drycheetah84/TomBox @7e8064f` **nao** foi verificavel offline.

**L-05 — Cobertura de leitura.** Li integralmente `README.md`, `docs/MILESTONES.md`, as primeiras 60 linhas de `docs/DEVELOPMENT_LOG.md`, `THIRD_PARTY/CREDITS`, os dois workflows de CI relevantes e ~80 commits. **Nao** li `docs/BLOCKERS.md` inteiro (1165 linhas) — usei contagem por prefixo (`117 OPEN`, `5 PARTIAL`, `62 DONE`) e leitura dirigida das secoes ligadas aos commits que abri. **Nao** li `docs/LIBRARY_MAP.md` (gerado, ~2.1 MB de tabela por funcao) nem `docs/LIBRARY_DISCOVERY.md`. Nao abri os ~100 crates de `crates/compat` um a um; a descricao de secao B vem da estrutura de diretorios, dos nomes e dos comentarios de modulo de `hlsl.rs`, `shader_xlat.rs`, `lib.rs` e `fssm_bind.rs`.

**L-06 — Assimetria de profundidade.** Examinei com diff real os commits de correcao, merge, teste e mudanca arquitetural. Nao examinei diff a diff os ~200 commits de documentacao/`progress.json`/badges SVG (que sozinhos respondem por 113 mudancas de `docs/BLOCKERS.md`, 77 de `tools/progress/progress.json`, 71 de `docs/img/progress.svg`) nem os lotes `rNN` inteiros — nesses casos leio o assunto e, quando possivel, o arquivo especifico citado.

**L-07 — Afirmacoes que deixo explicitamente em aberto.**
- D-20: a mensagem de `0284e83` fala em "drive-letter escape"; o diff **so** mostra a correcao de fronteira de caractere. Nao consegui localizar o commit do "escape" (pode nunca ter existido como tal). **PROVAVEL** que a mensagem esteja imprecisa.
- H-02: o commit que introduziu o prewarm sem gate nao e identificavel por `git log -S"GAME_SCAN_PREWARM"` (so `3de1b1f`). **PROVAVEL** que seja `a1ff5be` (r31), pelo contexto.
- H-03: o afrouxamento de asserts em `gdk_process_queue.rs` pode ser correto (identidade de handle), mas o commit nao cita medicao no runtime real. **NAO VERIFICADO**.
- G-02: nao ha fixtures de shader nesta maquina; o proprio teste do repositorio falha com `not proven: 0 FSSM fixtures`. Toda a discussao sobre localizacao de `PGM_RSRC1`/`RSRC2` e sobre a tabela de estagio e **documental**, nao reproduzida.
- D-08: a segunda mudanca no caminho (`9101363`, "Prototype staging atomic amplification payload writes", 2026-11-24) **nao foi analisada** — nao sei se ela substitui, complementa ou reintroduz o problema que `d2cc33e` corrigiu. Marcado como pendente.

**L-08 — Viés de fonte.** Boa parte do que sei sobre causas vem de mensagens de commit escritas pelo proprio autor da correcao, com incentivo a apresentar o diagnostico como definitivo. Onde havia diff, conferi o diff (foi assim que achei as inconsistencias de H-06 e D-20). Onde nao havia (medicao em hardware, comportamento do console), registrei como declaracao, nao como fato. A secao E marca cada cadeia com o grau de confianca correspondente.

**L-09 — O que uma segunda passada deveria fazer.** (a) `git patch-id`/`git log --cherry-pick` cruzado com `repos/portwatch/TuxBox` e com os clones VIctory para medir reuso real; (b) leitura integral de `docs/BLOCKERS.md` mapeando cada linha `OPEN` para um commit ou para "sem correcao"; (c) rodar `cargo check --workspace --all-targets` e a suite de `tombox-xshader` para substituir as declaracoes de CI por medicao propria; (d) abrir `9101363` e o restante do historico de `hlsl/amp.rs`; (e) se houver rede, ler os PRs `#289`, `#333`, `#346`, `#347` e o review citado em D-11.