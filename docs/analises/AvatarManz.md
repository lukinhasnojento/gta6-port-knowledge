# AvatarManz - analise de DeepSeek V4.1 Flash

## A. Identificacao

| Campo | Valor |
|---|---|
| Nome | AvatarManz |
| URL | https://github.com/avatarmanz/VIctory-7900XTX |
| Natureza | Fork do VIctory (camada de compatibilidade Xbox Series -> Windows), especializado em AMD RDNA 3 / Radeon RX 7900 XTX |
| Origem declarada | `README.md`: "A fork of [VIctory](https://github.com/Marvisak/VIctory)"; creditos citam `fxceless/victory-fork` como a arvore de partida ("fxceless: the base") |
| Ranking | 5o lugar no PortWatch, 362 estrelas, verificado pelo proprio PortWatch (dado fornecido no enunciado; NAO VERIFICADO no repositorio) |
| Branch analisada | `main` (unica branch local; `origin/main` aponta para o mesmo commit) |
| Raiz (commit 0) | `f4504f6413ce4509604dad66cdd689000a0204dc` | 2026-11-23 01:15:26 +0100 | Marvis | "Initial commit" |
| Primeiro commit do fork | `f4438b3` | 2026-11-23 | local | "Shaders: reads of uninitialized SGPRs read as zero, as VGPRs do" (82 commits entre a raiz e este) |
| Ultimo commit | `0a43566532c57b34a5397d6fdc91e419de32e8e9` | 2026-11-25 16:14:34 -0600 | local | "README: where it is cloned from" |
| Periodo coberto | 2026-11-23 a 2026-11-25 (3 dias com commit) |
| Total de commits | 621 em `HEAD`, 621 em todas as refs (0 commits so em outras branches) |
| Merges | 191 |
| Commits nao-merge | 430 |
| Commits substantivos do fork (`local@localhost`, sem merges) | 176 |
| Arquivos versionados | 246 (212 `.rs`, ~175.623 linhas de Rust) |
| Modelo responsavel | DeepSeek V4.1 Flash |
| Data da analise | 2026-11-25 |

**Autoria (topo da distribuicao).** `local <local@localhost>` 276; `Marvis <60314886+Marvisak@...>` 87; `Marvis <marvis@noreply.invalid>` 17; `fxceless` 16; `Duff` 16; `Vyacheslav Venglovsky` 8; `Maraco91` 8; `Manuel Perez` 7; `TG0000` 7; `marquezini` 7; `geiyiliyabumo` 7; `r4.` 6; `quaternion` 6; `9hvn9yt2vk-netizen` 5; `fuzeofficial` 5. O autor `local` e o agente (todos os commits trazem `Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>` ou `Claude Fable 5.2`).

**Relação com a base (por conteudo, nunca por hash).** `FORK_DIVERGENCIA.md` registra o achado decisivo: o HEAD do Marvisak **nao e ancestral** deste fork (`git merge-base` falha), todos compartilham apenas a raiz `f4504f6`, houve reescrita de historico. Comparacao confiavel = blob por arquivo: **89 arquivos so existem aqui, 17 arquivos da base estao ausentes, 117 blobs diferem** — o fork mais divergente em conteudo dos cinco. Consequencia pratica: nao existe cherry-pick direto entre repositorios; toda integracao e por porta manual de conteudo.

**Estado do repo.** `git show-ref` -> 3 refs (`refs/heads/main`, `refs/remotes/origin/HEAD`, `refs/remotes/origin/main`); nao e shallow; nao ha `.mailmap`. CI unico em `.github/workflows/ci.yml`: `cargo clippy --workspace --all-targets --locked -- -D warnings` (build normal e `--all-features`) e `cargo test --workspace --locked` em `windows-latest`, timeout 60 min.

---

## B. Resumo tecnico

**Proposito.** Rodar um titulo Xbox Series (o dump do usuario) nativamente no Windows a partir dos proprios arquivos do jogo, com **foco declarado em velocidade em AMD RDNA 3**, sem sacrificar a imagem. `README.md` e explicito: "the world at 30 frames a second on one PC, without giving up the picture. It is not there yet."

**Arquitetura.** Workspace Rust (edition 2024, MSVC) em 9 crates:

| Crate | Papel |
|---|---|
| `victory-pe`, `victory-loader` | Mapeamento do PE do titulo, imports/thunks, TLS, excecoes, emulacao de instrucoes so-de-console, crash log |
| `victory-hle` | Runtime do console reimplementado (kernel, xmem, dstorage, audio/XMA, xuser, xgamesave, gameinput, display, lifecycle...) |
| `victory-gpu` | Back end D3D12: host/submissao, descritores, sombras de memoria, recursos, texturas, indirect, replay, escrita do titulo (guards), pipeline |
| `victory-shader` | Tradutor GFX10 (console) -> DXIL: IR, `gfx10` (opcodes/fields/SP3), `emit/lower.rs`, `emit/bitcode` (otimizador) |
| `victory-formats` | Layouts de textura do console (swizzles `SW_*` tipo addrlib), BCn/bcpack, XMA |
| `victory-core`, `victory-game`, `victory-pipeline`, `apps/victory` | Config/politica, SGDP, cache de pipeline, CLI (`run`, `shaders`, `warm`, `replay`) |

**A tese tecnica do repositorio** (`docs/findings.md`, secao "The one idea behind most of it"): no console CPU e GPU compartilham memoria; no PC nao. Tudo o que um escreve e o outro le precisa ser (1) **notado**, (2) **movido**, (3) **ordenado**. Os ganhos vieram de tornar cada um mais estreito (dwords, nao paginas), mais largo (threads, AVX2) ou mais tardio (na GPU, atras do frame). Tabela do proprio autor:

| O que | Frame antes | depois |
|---|---|---|
| Registros de descritor deixados para os draws | 280 ms | ~124 |
| Passes de hash sobre copias na placa, 64 threads cada | placa 79 ms | 66 |
| Preparacao em thread propria | 103 | 75 |
| Chamadas de lista traduzidas por 8 threads (`VICTORY_THIN`) | 75 | 68 |
| Submissoes unidas, passos na placa, texturas adiantadas | 68 | ~63 |
| Bytes do titulo sob escritas de shader comparados com AVX2 | 14 ms | 4-7 |
| Stores em memoria guardada olhados antes | 69 ms de thread | ~6 |

**Onde o frame vai hoje** (55 ms na agua): thread que prepara submissoes 48 ms; placa ~41 ms ocupada; 36.000 chamadas de lista por frame (230 ms de trabalho em 8 threads); 400.000 slots de descritor olhados, 40.000 espelhados; 16 MB/frame de "landing".

**Estado atual medido** (`README.md`, uma unica maquina: i9-12900K, 64 GB, RX 7900 XTX 24 GB, Windows 11, 1440p, perfil Series X): menus/videos 30 fps; prologo indoor 13-20 fps; primeira missao de barco 19-22 fps; perseguicao de rua 6-10 fps; a pe/camp 11-16 fps; voando 17-24 fps; sessao mais longa 41 min sem crash; **40-46 GB de memoria privada** (recomenda 64 GB de RAM). Comecou em 280 ms/frame nos mesmos cenarios dois dias antes.

**Escala de configuracao.** 294 variaveis `VICTORY_*` lidas por `std::env::var`/`var_os` (358 nomes distintos citados no codigo), 46 testes `#[ignore]`. A configuracao real de execucao **nao** e o padrao do binario: e `scripts/play.ps1`, que liga ~20 switches (ver D-19).

---

## C. Linha do tempo

### Fase 0 - 2026-11-23: transplante do historico upstream (61 commits no dia)

A raiz `f4504f6` (Marvis, "Initial commit") e imediatamente seguida de historia reescrita do VIctory: `12caefe` "GFX10 operand fields and SP3 text; FSSM wave size", `754619f` "Run Minecraft: runtime HLE, D3D12.X device on the host GPU, shader IR", `2849477` "Run Grand Theft Auto VI from first boot into the world", `015d45b` "README: status, subsystem table, how to run, debugging switches". Nada aqui e trabalho do fork; e a base que sera medida.

### Fase 1 - 2026-11-24 (manha): importacao do fork fxceless (o "commit 0" de conteudo)

- `578d459` (fxceless) "victory-gta6: this fork's changes on top of VIctory main (ffe310e)" — o grosso das diferencas de conteudo nasce aqui.
- `4e17478` (fxceless) "The game is no longer named in the tree".
- `6a2d9f9` (fxceless) "Shadow and trace programs translate, lint clean, ray-tracing write-up" — traz `console_insn_decode.rs` (decodificador de instrucoes so-de-console).
- `8c26061` (Marvis, autor reescrito) "GPU: the world stays up; indirect execution behind a switch" — **introduz o clamp de argumentos indiretos com orcamento por lista** (`static const uint BUDGET = 1u << 27`), a peca que mais tarde causa D-01/D-02.
- `8dd30ab` (fxceless) "A write through one view of a page is seen at its other views; page writes counted; levels on demand (off); warm-up; card memory numbers".
- `35b054e` "GPU: pipelines off the render thread and cached; indirect execution on".

### Fase 2 - 2026-11-24 (tarde/noite, 360 commits): a maquinaria propria nasce

Ordem aproximada, por topologia (datas de autor sao enganosas apos rebase):

1. **Escrita do titulo e guards.** `74045e4` "Holes of view memory known by their bits; landed blocks taken; askings of views left out"; `0d5e4c8` "What shaders wrote is landed on a few threads, beside the guards (`VICTORY_LAND`)"; `2aa8134` "Lighter refreshes of host copies; landing past the guards; the heap pass spread".
2. **Texto indireto.** `84eae35` "Indirect dispatches of more than 65,535 groups run whole (`VICTORY_INDIRECT_WIDE`)" (correcao de D-02).
3. **Chamadas de lista finas.** `3a99ee6` "List calls written down and translated by threads of ours (`VICTORY_THIN=1`, off)" — 45.000-60.000 chamadas/frame, 3,5 us cada, 160-190 ms de CPU.
4. **Submissao assincrona.** `b8b30d1` "Submissions are prepared and sent by a thread of ours (`VICTORY_ASYNC_SUBMIT=1`, off)".
5. **Diagnostico.** `8a8ad35` "Crash tracing that can stay on: `VICTORY_D3D12_DRED=light` e `=faults`"; `336f613` (cherry-pick de `008de5f9`) "A timeline of a few frames, com a cadeia de cada present (`VICTORY_TIMELINE`)".
6. **Replay.** Serie `replay/record.rs` (2.604 linhas), `replay/play.rs` (3.464), `merge/diverge/pixel/upcopies` — o instrumento de medida que sustenta todas as decisoes seguintes.

### Fase 3 - 2026-11-25 (200 commits): traducao especifica para RDNA 3

- `9fea743` "Streaming, copy queue, texture pools and a shader cache that survives rebuilds" — introduz `isHelperLane` no lowering (D-11) e o cache de shader com impressao digital.
- `20b9834` "Max, min, legacy products, medians and fractions as the RDNA card's own (`VICTORY_RDNA_FLOAT`, on)" — **o maior ganho isolado de tradutor documentado**: 18,41 -> 17,58 ms.
- `30f46c9` "Structure builder: fences around its buffer atomics (`VICTORY_BVH_FENCES=0`: none)".
- `2c20d7a` "Structure builder: submissions that ask for structures' sizes are waited for" (corrige D-05).
- `810efe5` "Structure builder and rays: dispatches counted in threads..." (corrige D-12, vindo de DexNom `bc3cd4b0`, `5f4c3df4`).
- `1f58d91` "Shaders read the buffers they pick through raw twins of the heap's views" (corrige D-13, dois device losses).
- `689df31` e `2be0515` (duplicata) "Views mapped beside guarded ones" (corrige D-06).
- `a30829c` "Pipelines known to take long are made beside the others"; `a989668` "Passes over the screen have pipeline workers and a wait of their own"; `b7ab9c4` "The title's pipelines written down... (`victory warm`)".
- `82ca134` "A stand-in for the title's traced indirect light (`VICTORY_AMBIENT`, off)".
- `449830f` "The frame line says how many indirect records ran empty for want of their list's budget..." (instrumenta D-01).
- `95037ba` Revert de `a50c7fd` (replay/cross-adapter, D-10); `d1c8acd` Revert de `9820bdc` (janela stand-in HLE).
- `3b30d6f` "README, findings and a start script of this fork" — nasce `docs/findings.md` e `scripts/play.ps1`.
- `0a43566` (HEAD) "README: where it is cloned from" — apenas documentacao.

### Padrao de trabalho observavel

191 merges em 621 commits (31 %). Diarias de duas familias — `day/shaders`, `day/faults`, `day/light`, `day/textures`, `day/submit`, `night/crash`, `night/cpu`, `night/card`, `night/stream`, `night/textures` — com merges cruzados (`Merge branch 'night/crash' into local/next`, `Merge local/next into night/cpu`) criando o mesmo patch duas vezes (ver D-15). Nenhuma branch sobreviveu: o resultado e uma unica `main` linearizada por merges.

---

## D. Catalogo de defeitos

### D-01 - Orcamento indireto esgotado por um registro lixo derruba todos os draws seguintes da lista

- **Descricao/sintomas.** Arvores, plantas e predios distantes **piscam** ao ar livre, em alguns lugares. Um draw da lista desaparece em um frame e volta no outro, sem padrao fixo.
- **Causa raiz.** O clamp de argumentos indiretos cobra de um **orcamento por lista** (`BUDGET = 1u << 27` vertex-instances/threads; grupo de compute = 64 threads, grupo de mesh = 128). Um registro nao preenchido pelo titulo (slot com lixo) que custa muito e aceito, esgota o orcamento, e **todos os registros seguintes daquela lista rodam vazios** — e qual registro e o culpado pode mudar de frame para frame (ordem vem de dados que shaders escrevem com atomics).
- **Commit que introduziu.** `8c26061` (Marvis, autor reescrito, 2026-11-24) — herdado pela base, nao escrito por este fork.
- **Tentativas de correcao.** `449830f` (2026-11-25) instrumenta: conta por frame quantos registros rodaram vazios e o maior custo visto (palavras 3 e 4 do buffer de orcamento), e adiciona `VICTORY_INDIRECT_REST=0`, que faz um registro que nao cabe **nao consumir** o espaco dos seguintes (`next = (BUDGET - next < c) ? (REST ? BUDGET : next) : (next + c)`).
- **Correcao definitiva.** **SEM CORRECAO.** O padrao continua `REST=1` (comportamento que propaga o esgotamento) e o interruptor e experimental.
- **Arquivos.** `crates/victory-gpu/src/indirect.rs` (HLSL embutido, `SOURCE`), `crates/victory-gpu/src/indirect/batch.rs`, `crates/victory-gpu/src/host.rs` (`BUDGET_NOTES`, linha de frame do `Meter`).
- **Evidencia.** `indirect.rs:64` `static const uint BUDGET = 1u << 27;`; `indirect.rs:156-193` a cobranca por grupo (`for (uint j = 0; j < n; j++) ... next = (BUDGET - next < c) ? (REST ? BUDGET : next)`); mensagem de `449830f`: "Belobug (76387c2d) found draws flickering because the records past a list's budget were not the same ones from frame to frame. Whether ours are refused is not known: the count says it"; `docs/findings.md` "What is next" item 4: "Outdoor flicker: suspected to be one bad indirect record that uses up its list's budget".
- **Estado atual.** Aberto e **explicitamente nao confirmado** pelo autor ("whether ours are refused is not known"). A instrumentacao existe para decidir.
- **Confianca.** Sintoma CONFIRMADA (README lista o flicker); mecanismo como causa PROVAVEL (o proprio autor nao confirma).

### D-02 - Corte em 65.535 grupos perdia a ultima faixa de tiles da imagem

- **Descricao/sintomas.** Faixas/patchs blocky na imagem; no caso medido, os 1.905 tiles mais baixos da imagem nao rodavam.
- **Causa raiz.** D3D12 nomeia 65.535 como maximo de grupos por dispatch; o clamp cortava todo dispatch indireto nesse valor. As listas de tiles do titulo sao **um grupo por tile**: 67.440 e 47.760 grupos (115.200 tiles de 2560x1440) nos programas `2dc364df5d3278b8` e `940c2635dc1312f4`.
- **Commit que introduziu.** O clamp da base (`8c26061`) e o limite anterior a ele.
- **Tentativas de correcao.** `84eae35` (cherry-pick de `b856c46d`): `widest()` devolve `1 << 20` em placa AMD (o console conta 32 bits e o hardware AMD aceita os grupos um a um), 65.535 nas outras; teste de placa `card_runs_wide_indirect_dispatches` confirma "every group once, under its number" na RX 7900 XTX.
- **Correcao definitiva.** `84eae35` para **compute**. Para **mesh dispatch** o corte permanece e virou regra mais estrita: `ef556e4` corta mesh direto em 65.535 ("more is undefined on the host; said once") e `a9c2e99` faz registro mesh indireto acima de 65.535 rodar **vazio** ("all ones" = slot nao preenchido).
- **Arquivos.** `crates/victory-gpu/src/indirect.rs` (`widest`, `fn amd`, HLSL linha 121 e 135-139).
- **Evidencia.** `indirect.rs:279-282` `fn widest(amd: bool) -> u32 { ... if amd && on { 1 << 20 } else { 65_535 } }`; `indirect.rs:121` `g = min(g, uint3(g.x <= WIDEST ? WIDEST : 65535u, ...))`; mensagem de `84eae35` com os dois hashes de programa e a contagem de tiles.
- **Estado atual.** Compute: corrigido e testado na placa. Mesh: limite assumido como intencional.
- **Confianca.** CONFIRMADA.

### D-03 - `VICTORY_SEND_CHANGED` ligado por padrao fez a sala perder luz indireta e explodir a geometria

- **Descricao/sintomas.** Na run `tile1-0`: a sala perde a luz indireta e a geometria "explode". A run `bis1-0`, identica exceto por esse switch, estava correta.
- **Causa raiz.** O refresh de uma copia na placa passou a enviar **apenas as dwords que a memoria do titulo mudou** (`changed_runs`), em vez das paginas de 64 KiB escritas. Uma pagina contada como escrita e "em boa parte nao obra do titulo, ou nao toda ela": uma palavra escrita pelo titulo vem cercada do que shaders escreveram e do que a placa escreveu por cima. Ao mandar so o diff, o que a placa havia escrito na mesma pagina deixa de ser reenviado e a copia fica incoerente.
- **Commit que introduziu.** `c651346` "A refresh of a copy on the card sends the dwords title memory changed, and no others (`VICTORY_SEND_CHANGED`, on)".
- **Tentativas de correcao.** Nenhuma correcao do mecanismo. `6ae0696` apenas inverte o padrao: `send_changed()` passa de `is_none_or(|v| v != "0")` para `is_some_and(|v| v != "0")`, e ajusta o helper de teste `refresh_tests` para o novo padrao.
- **Correcao definitiva.** **SEM CORRECAO** (feature desligada por padrao). Bisseccao feita, causa raiz nao identificada.
- **Arquivos.** `crates/victory-gpu/src/shadow.rs` (`send_changed`, `changed_runs`), `crates/victory-gpu/src/host.rs` (`refresh_tests`).
- **Evidencia.** Comentario corrente em `shadow.rs:1680-1683`: "Off since run tile1-0: the room lost its indirect light and geometry exploded; run bis1-0, the same but for this, was right."
- **Estado atual.** Desligado. O caminho rapido que motivou a feature (enviar dwords em vez de paginas) **nao esta em uso**.
- **Confianca.** CONFIRMADA (regressao documentada e bissectada pelo autor); causa raiz interna NAO VERIFICADA.

### D-04 - 16 threads de mapeamento por padrao pioraram o dormitorio de 125 para 260 ms

- **Descricao/sintomas.** No dormitorio, cada `XMemMapPhysicalPages` custa 0,3 ms de um recurso que so um por vez obtem; os 16 threads **nao mapeiam mais paginas** e seguram o lock de mapeamento, dobrando o frame.
- **Causa raiz.** `side_by_side()` passou a devolver `WIDE = 16` automaticamente quando o mapeamento consome mais de um quarto da janela (`Busy`), sem medicao em cena real. O diagnostico que motivou (thread de streaming gastando 400-750 ms/s em chamadas de mapeamento na primeira minuto do mundo) valia para o carregamento, nao para o dormitorio.
- **Commit que introduziu.** `ac11bbb` "Pages are mapped by sixteen threads while the title streams as fast as they are mapped, by the caller alone otherwise (`VICTORY_MAP_THREADS=auto`, which it is unless set)".
- **Tentativas de correcao.** `4e99544` "`VICTORY_MAP_THREADS=auto` is asked for, not the default": `auto` deixa de ser padrao; sem variavel volta a 8 threads (o valor original), `auto` fica opt-in.
- **Correcao definitiva.** `4e99544` (reversao do padrao, com numeros: 260 ms contra 125 ms, runs `textures-d4` e `-d2`). `scripts/play.ps1` fixa `VICTORY_MAP_THREADS='1'`.
- **Arquivos.** `crates/victory-hle/src/modules/xmem.rs` (`side_by_side`, `asked_threads`, `WIDE`, `GROUP = 64`, `Busy`).
- **Evidencia.** `git show 4e99544`: "+/// shortcut: `auto` is not the default. ... 16 threads map no more pages than one, and the groups hold the mapping lock: 260 ms a frame against 125 (textures-d4, -d2)."
- **Estado atual.** Corrigido; `auto` permanece disponivel e nao e usado.
- **Confianca.** CONFIRMADA.

### D-05 - Tamanho de estrutura BVH lido antes de a submissao pousar: copias sobrescreviam vizinhos

- **Descricao/sintomas.** Corrupcao de estruturas de ray tracing; copias de compactacao escrevendo sobre estruturas vizinhas (run `light-11`).
- **Causa raiz.** O titulo pede o tamanho compactado de uma estrutura nova (programa de 1 thread grava num buffer pequeno), **le o valor um ou dois frames depois sem verificar se chegou**, aloca daquele tamanho e copia a estrutura. Como as submissoes podem pousar ate dois frames tarde, ele lia o que o slot continha antes — o tamanho de **outra** estrutura — e a copia transbordava.
- **Commit que introduziu.** O modelo de submissao tardia do fork (fase 2); nao ha um commit unico que introduza o bug, ele e consequencia de `VICTORY_ASYNC_SUBMIT`/landing tardio.
- **Tentativas de correcao.** `2c20d7a` "Structure builder: submissions that ask for structures' sizes are waited for (`VICTORY_BVH_SIZES_WAIT=0`: not); a packed structure is one node longer; `VICTORY_BVH_TRACE`".
- **Correcao definitiva.** `2c20d7a`: `asks_size()` reconhece o programa (1 thread, constantes `[_, hi, 1..=3, ..]`, buffer pequeno) e `Host::submit` chama `settle(&mut submitted, WHY_READ)` antes de seguir; mais um no extra na estrutura compactada.
- **Arquivos.** `crates/victory-gpu/src/memory.rs` (`SIZES`, `sizes_wait`, `asks_size`, `asked_sizes`), `crates/victory-gpu/src/host.rs`, `crates/victory-gpu/src/draw.rs`.
- **Evidencia.** Comentario de `sizes_wait()`: "of 230 such copies of run light-11, 99 were asked for two frames earlier and landed in that frame, none of those with the right size."
- **Estado atual.** Corrigido e ligado por padrao.
- **Confianca.** CONFIRMADA (medicao no proprio comentario).

### D-06 - Corrida entre mapear uma view e abrir os guards das outras views da mesma pagina

- **Descricao/sintomas.** Escrita rara perdida: um teste de stress falhava em **6 de 200** execucoes. Uma escrita do titulo por uma view nao era vista por outra view da mesma pagina ate o proximo "told write" do bloco.
- **Causa raiz.** Duas falhas somadas: (a) as views a abrir eram procuradas **antes** de tomar o lock, entao uma view tornada limpa nesse intervalo continuava limpa com outra view gravavel ao lado; (b) as views eram abertas **uma faixa depois da outra**, entao por microssegundos uma era gravavel enquanto outras ainda eram conhecidas pelos bits (quem pergunta por elas sem lock lia errado).
- **Commit que introduziu.** "Both are older than the faults' work"; `VICTORY_OPEN_ONE_VIEW` (de `7bc7c07`) tornou a primeira comum.
- **Tentativas de correcao.** Duas etapas medidas: "failed in 6 of 200 runs, then 4 of 380 with the first mended, 0 of 603 (four tests each) with both."
- **Correcao definitiva.** `689df31` (e a duplicata `2be0515`): todas as views da pagina sao abertas juntas sob o lock (`flat_map` sobre as faixas, `unguard` de uma vez), e `title_memory_splitting` limpa `clean`/`PUBLISHED` de todas as outras views antes de tornar a primeira gravavel; `published()` passa a exigir `!BESIDE.any(..)`.
- **Arquivos.** `crates/victory-gpu/src/watch.rs` (`views_mapped`, `published`, `block_published`, `title_memory_splitting`).
- **Evidencia.** `git show 689df31 -- crates/victory-gpu/src/watch.rs` (hunks em `published`, `views_mapped`, `title_memory_splitting`); mensagem com as tres contagens de falha.
- **Estado atual.** Corrigido.
- **Confianca.** CONFIRMADA.

### D-07 - Passes sobre a tela perdiam pipelines e a cena ficava sem luz

- **Descricao/sintomas.** Na primeira run apos nova traducao (`textures-d2`): sem lampadas e sem cor na sombra ate o fim da sessao; a segunda run (`textures-d3`) tinha as duas coisas.
- **Causa raiz.** O titulo preenche tabelas de iluminacao com **passes sobre a tela** (sem vertex input, sem depth/stencil, com target, topologia triangulo) uma unica vez, no primeiro frame do mundo. Esses pipelines ficavam atras de uma fila de um mundo inteiro de pipelines e eram **descartados** pelo orcamento de espera de 0,5 s por frame; um pass que roda uma vez e cujo pipeline demora mais que o orcamento e perdido para sempre.
- **Commit que introduziu.** O orcamento de espera de pipeline da base (`pipeline/make.rs`, "The default budget is half a second per title frame").
- **Tentativas de correcao.** `a989668` "Passes over the screen have pipeline workers and a wait of their own (`VICTORY_SCREEN_WAIT_MS=0`: not); after Belobug 18e57db8".
- **Correcao definitiva.** `a989668`: `GraphicsState::screen_pass()` classifica o pipeline; `Pending.screen` e marcado; `screen_budget()` (10 s por padrao, abaixo do watchdog de 30 s do titulo) e workers proprios, com os passes de tela **na frente** dos demais (`screens_first()`).
- **Arquivos.** `crates/victory-gpu/src/pipeline/host.rs`, `crates/victory-gpu/src/pipeline/make.rs`.
- **Evidencia.** `make.rs:14-24` documenta o caso e a run; `git show a989668`.
- **Estado atual.** Corrigido (o autor registra que Belobug achou o mesmo em `18e57db8`).
- **Confianca.** CONFIRMADA.

### D-08 - Compilacao de pipelines travava o titulo por dezenas de segundos

- **Descricao/sintomas.** Primeira run com tradutor novo (caches vazios): 3.020 pipelines iniciados na criacao, todos os 16 workers em programas de segundos a dezenas de segundos (97 feitos em 20 s); depois os rapidos a 100-190/s, "too late": frames de 0,5-1,6 s e ate 2.900 draws por frame cairam de 60 para 150 s.
- **Causa raiz.** Fila unica de criacao: pipelines caros (compilador do driver) monopolizam os workers e bloqueiam os baratos que o frame atual precisa; o orcamento de espera por frame estoura e os draws sao perdidos.
- **Tentativas de correcao.** `b7ab9c4` (`victory warm`: pipelines do titulo escritos em `known-pipelines.bin`, feitos antes em processos idle de 200, para o cache do driver); `a30829c` (pipelines sabidos lentos vao para uma fila propria, atendida por **um quarto** dos workers; `slow-pipelines.txt`); `a989668` (D-07, terceira fila).
- **Correcao definitiva.** Parcial. As tres medidas atacam sintomas distintos da mesma fila; nao ha escalonador com prioridade por custo real medido.
- **Arquivos.** `apps/victory/src/warm.rs`, `crates/victory-gpu/src/pipeline/known.rs` (480+ linhas, so deste fork), `crates/victory-gpu/src/pipeline/library.rs`, `crates/victory-gpu/src/pipeline/make.rs`.
- **Evidencia.** Mensagem de `a30829c` com os numeros acima; `make.rs` `slow_limit()`, `Queue::slow`.
- **Estado atual.** Mitigado; README ainda lista "The first run with a new driver or a new build stutters while the driver makes pipelines" como problema conhecido.
- **Confianca.** CONFIRMADA (medicoes do autor); eficacia final NAO VERIFICADA (sem medicao de "depois" no repositorio).

### D-09 - Device loss a cada ~16 min: textura mostrada antes de o upload rodar

- **Descricao/sintomas.** Perda de dispositivo aproximadamente a cada 16 minutos; page fault dentro de uma textura **criada no mesmo decimo de segundo**, com um slot mostrando-a antes do upload.
- **Causa raiz.** O slot de descritor passa a nomear a textura nova imediatamente; a submissao que copia os texels ainda nao rodou. Um draw que le o slot antes disso le memoria nao inicializada/realocada.
- **Commit que introduziu.** O modelo de upload em fila de copia propria (fase 2), que e o que tornou o intervalo "criada -> mostrada" observavel.
- **Tentativas de correcao.** `3bab23c` "Title slots show a new texture after its upload has run (`VICTORY_SHOW_AFTER=1`; off), and the book says more of one" — coloca uma textura neutra 1x1 (ou a view antiga) no slot ate a submissao com o upload terminar.
- **Correcao definitiva.** Existe, mas **desligada por padrao** (`is_some_and(|v| v != "0")`, ou seja, so liga com a variavel setada). `scripts/play.ps1` a liga (`VICTORY_SHOW_AFTER='1'`).
- **Arquivos.** `crates/victory-gpu/src/host.rs` (`show_after`, `texture_slot_show_after`, contadores), `crates/victory-gpu/src/draw.rs`, `crates/victory-gpu/src/book.rs`.
- **Evidencia.** `host.rs:283` "`VICTORY_SHOW_AFTER=1`; off"; `host.rs:290` `is_some_and(|v| v != "0")`; `docs/findings.md`: "A device loss about every 16 minutes... `VICTORY_SHOW_AFTER=1`; none in the sessions since (likely fixed, not proven)."
- **Estado atual.** Sem perda nas sessoes desde entao, **nao provado**; custo visivel: "Textures of what just came into view appear late while the title streams" (README, problemas conhecidos).
- **Confianca.** Causa PROVAVEL; correcao eficaz NÃO VERIFICADA (o proprio autor diz "likely fixed, not proven").

### D-10 - Correcao do replay feita e revertida sem explicacao (buffers em heaps abertos da memoria)

- **Descricao/sintomas.** O replay nao conseguia recriar buffers que vivem em heaps abertos a partir da memoria do titulo (`OpenExistingHeapFromAddress`/`FromFileMapping`), porque tais buffers exigem `ALLOW_CROSS_ADAPTER` e o replay os recriava sem a flag.
- **Causa raiz.** `a50c7fd` "Replay: buffers in heaps opened from memory are made again (without the cross-adapter flag)" tentou recriar o buffer com `desc.Flags &= !D3D12_RESOURCE_FLAG_ALLOW_CROSS_ADAPTER`.
- **Commit que introduziu.** O caminho de replay (`replay/play.rs`), que reconstroi recursos por descricao.
- **Tentativas de correcao.** `a50c7fd` (unica tentativa).
- **Correcao definitiva.** **SEM CORRECAO.** `95037ba` reverte `a50c7fd` integralmente: "This reverts commit a50c7fdeb55eca4a27c70151b4154665c32e22e9." Sem justificativa no corpo. `git log --format="%h %s" -S"ALLOW_CROSS_ADAPTER"` mostra apenas esses dois commits e `578d459`.
- **Arquivos.** `crates/victory-gpu/src/replay/play.rs` (linhas ~789), `crates/victory-gpu/src/inplace.rs` (a regra vigente).
- **Evidencia.** `git show 95037ba`; `inplace.rs:29-32`: "A buffer in it must have `ALLOW_CROSS_ADAPTER`... A texture must be row-major with one level. Tile mappings into such a heap are refused and REMOVE THE DEVICE: never call them."
- **Estado atual.** Revertido; a limitacao volta a existir. `docs/replay.md` "Limits" lista o que nao e gravado, e nao menciona este caso explicitamente — a lacuna fica sem documentacao.
- **Confianca.** Reversao CONFIRMADA; motivo NÃO VERIFICADO (nao ha mensagem, issue ou comentario).

### D-11 - Helper lanes nao participam das operacoes de wave na AMD: buracos quadrados e bordas tracejadas nas sombras

- **Descricao/sintomas.** "Square holes and dashed edges in shadows"; um shader de pixel que le a posicao dos vizinhos do seu quad via wave reads perde a amostra nas bordas de todo triangulo.
- **Causa raiz.** No console (modo whole-quad) as helper lanes estao em EXEC e participam; nas operacoes de wave do host (AMD incluida) elas sao **excluidas**. Num laco em waterfall, quando so sobraram helper lanes nenhuma casa com o teste; elas rodam ate o limite de iteracoes e saem **sem amostra** para as derivadas do quad.
- **Commit que introduziu.** O lowering de `first_lane`/`readlane` da base, que assume que a wave do host ve as mesmas lanes que a do console.
- **Tentativas de correcao.** `9fea743` introduz `helper_lane()` (`isHelperLane.i1`, op 221) e faz `first_lane()` devolver o **proprio valor da lane** quando ela e helper.
- **Correcao definitiva.** `9fea743` (forma local), documentada como correcao de `#908`/`#1733` no comentario.
- **Arquivos.** `crates/victory-shader/src/emit/lower.rs` (`first_lane`, `helper_lane`, `first_live_lane`), `emit/mod.rs` (flag de helper).
- **Evidencia.** `lower.rs:4789-4793`: "A helper lane of a pixel shader takes its own value: the console's whole-quad mode has helper lanes in EXEC... Once only helper lanes are left in such a loop none of them ever matched: they ran to the iteration limit, and left with no sample for their quad's derivatives, along every triangle's edges"; `docs/findings.md` item correspondente.
- **Estado atual.** Corrigido; as helper lanes dependem de `isHelperLane`, que exige shader model 6.6 no host.
- **Confianca.** CONFIRMADA (comentario + findings).

### D-12 - Contagem de dispatch de ray tracing tomada como grupos em vez de threads: toda ray errava

- **Descricao/sintomas.** "Every ray missed": nenhum raio acerta nada; um estagio de reidratacao rodava ate 64x os threads pedidos e **gravava por cima das estruturas seguintes**.
- **Causa raiz.** O flag `0x20` de `DispatchX` diz que as contagens sao **threads**, nao grupos; o tradutor tratava como grupos.
- **Commit que introduziu.** Herdado da base (traducao de dispatch de compute do console).
- **Tentativas de correcao.** `810efe5`: nas lanes acima da contagem o programa sai de imediato, em todo programa de builder de uma dimensao sem barreira de grupo; `VICTORY_THREAD_DIMENSIONS=0` volta ao antigo.
- **Correcao definitiva.** `810efe5` (de DexNom `bc3cd4b0` e `5f4c3df4`), junto com: filho ausente com bounds infinitos invertidos nao e mais visitado; pagina sem copia le zeros (sem hit); assinaturas de vertice na ordem do DXC.
- **Arquivos.** `crates/victory-shader/src/emit/*`, `crates/victory-gpu/src/pipeline*`, `crates/victory-shader/src/emit/intersect.rs`.
- **Evidencia.** `git show -s 810efe5` (corpo completo citado acima).
- **Estado atual.** Corrigido.
- **Confianca.** CONFIRMADA (mensagem detalha o mecanismo e o efeito medido).

### D-13 - Leitura raw atraves de view tipada nao tem fim na AMD: dois device losses no meshlet culling

- **Descricao/sintomas.** Dois device losses em 2026-11-25, ambos parando no culling de meshlets (`0452fc2d2e2fc4c3`).
- **Causa raiz.** Um shader que **escolhe** um buffer do heap por um indice calculado le bytes crus, qualquer que seja a view que o titulo tem ali — e o indice e o offset vem de dados. Na AMD, um load raw atraves de view tipada/estruturada/constant buffer **nao tem fim** (le ate 4 GiB adiante); o descritor de uma textura lido como buffer e um endereco selvagem; e um indice alem dos slots do titulo (limitados a 2^21) alcanca os slots dinamicos de outras listas.
- **Commit que introduziu.** O lowering de "picked buffers" da base.
- **Tentativas de correcao.** `1f58d91` "Shaders read the buffers they pick through raw twins of the heap's views": heaps passam a 2^22 views; acima de 2^21 existe uma **view raw dos mesmos bytes** para cada view de buffer de slot do titulo (view de constant buffer para constant buffer; view raw nula para todo o resto e todo slot alem dos do titulo); picks raw/constant sao lidos ali. `VICTORY_RAW_VIEWS=0` volta atras.
- **Correcao definitiva.** `1f58d91`, com o custo declarado: "Every shader is translated again (8 min)".
- **Arquivos.** `crates/victory-shader/src/emit/shader.rs` (`set_raw_views`, `raw_views`, clamps de indice), `crates/victory-gpu/src/descriptor*`.
- **Evidencia.** `git show -s 1f58d91`; `shader.rs:161-169`: "On AMD a raw load through a view that is not raw (typed, structured, a constant buffer's) has no end: it reads whatever lies there, up to 4 GiB on, and loses the device where nothing does".
- **Estado atual.** Corrigido e ligado por padrao.
- **Confianca.** CONFIRMADA.

### D-14 - Indice de view alem do heap: a RDNA 3 remove o dispositivo

- **Descricao/sintomas.** Leitura de descritor fora do heap: "GPU-based validation: descriptor heap index out of bounds" na NVIDIA (trava a GPU) e **page fault com remocao do dispositivo** na AMD RDNA3.
- **Causa raiz.** O console nao limita o heap; o host limita (NVIDIA 1.000.000 views) e o indice vem de dados do titulo. Um heap comercial e maior que o do host.
- **Commit que introduziu.** Herdado (`31714de`, fxceless, "Take the fix/amd-view-heap-clamp branch of triippywhale's fork").
- **Tentativas de correcao.** `31714de`: todo heap termina em um **slot nulo** e indices a partir dele sao clampados (heaps grandes ganham 2^21 views; quem limita mantem o heap limitado); o limite entra na chave do cache de shader; `v_permlane16/x16` passam a ser baixados. `1f58d91` leva a 2^22.
- **Correcao definitiva.** `31714de` + `1f58d91`.
- **Arquivos.** `crates/victory-shader/src/emit/shader.rs` (`VIEW_LIMIT`, `set_view_heap_limit`), `crates/victory-gpu/src/descriptor/host.rs`.
- **Evidencia.** `shader.rs:144-149` (texto citado); `emit/mod.rs:298` (limite na impressao digital do cache).
- **Estado atual.** Corrigido.
- **Confianca.** CONFIRMADA.

### D-15 - O mesmo patch commitado duas vezes, com bases diferentes (duplicatas de cherry-pick)

- **Descricao/sintomas.** Dois pares de commits com assunto identico e conteudo equivalente, ambos **ancestrais de `HEAD`**: `689df31` / `2be0515` ("Views mapped beside guarded ones...") e `9c2168a` / `160d222` ("Views guarded while they are mapped stay writable; tests take turns on WARP", ambos filhos de `c897fa2`).
- **Causa raiz.** Trabalho feito em `night/*` e `day/*` e integrado por merges cruzados; o mesmo patch foi cherry-picked nos dois lados e depois os dois lados foram mergeados.
- **Commit que introduziu.** O par; nao ha commit "errado" isolado.
- **Tentativas de correcao.** Nenhuma (nada foi deduplicado).
- **Correcao definitiva.** **SEM CORRECAO.** A arvore final esta limpa (`watch.rs` tem exatamente uma definicao de `views_mapped`, `published`, `block_published` — verificado por grep), mas o historico carrega as duas versoes com contextos de linha diferentes.
- **Arquivos.** `crates/victory-gpu/src/watch.rs` (principal), `crates/victory-gpu/src/frame.rs`.
- **Evidencia.** `git log --no-merges --format="%s" | Group-Object | Where Count -gt 1` -> exatamente esses dois assuntos; `git merge-base --is-ancestor 689df31 HEAD` e o mesmo para `2be0515`, ambos verdadeiros; `git diff --stat 2be0515 689df31` mostra 22 arquivos e 1.575 linhas de diferenca entre as duas versoes por causa das bases distintas.
- **Estado atual.** Sem efeito funcional hoje; risco de merge futuro (um rebase/merge que reaplique a versao antiga de `watch.rs` reintroduz a corrida de D-06).
- **Confianca.** CONFIRMADA (duplicatas); impacto atual HIPOTESE.

### D-16 - Duas convencoes opostas para o mesmo tipo de switch, em 294 variaveis

- **Descricao/sintomas.** Comportamento imprevisivel de `VICTORY_*` "booleanos": `VICTORY_AMBIENT` e um numero (0 = off), `VICTORY_SHOW_AFTER` so liga com a variavel presente, `VICTORY_SEND_CHANGED` era o oposto. `VICTORY_AB=abc` desliga; `VICTORY_THIN=outra-coisa` desliga; mas para 98 switches qualquer valor diferente de `"0"` **liga** (inclusive `"abc"`, `"false"`, `" "`).
- **Causa raiz.** Cada funcao le a variavel inline, sem registro central nem parser comum. Nao ha `Config` unico para switches de diagnostico.
- **Commit que introduziu.** Crescimento incremental ao longo dos 3 dias; `77582c3` (upstream, "GPU: VICTORY_* switches set to 0 are off") tentou fixar uma convencao que este fork so segue em parte.
- **Tentativas de correcao.** Nenhuma estrutural.
- **Correcao definitiva.** **SEM CORRECAO.**
- **Arquivos.** Todo `crates/*/src`, `apps/victory/src`.
- **Evidencia.** Contagem por grep: `is_none_or(|v| v != "0")` ocorre **98** vezes; `is_some_and(|v| v != "0")` **7** vezes; 294 nomes distintos lidos por `env::var`/`var_os`; 358 nomes `VICTORY_*` citados. Contraste direto: `emit::builder_fences()` (`is_none_or`, liga por padrao) contra `host::show_after()` (`is_some_and`, so liga com a variavel setada); e ainda uma terceira forma, `map_or(true, ..)` em `indirect::rest()`.
- **Estado atual.** Aberto; e risco direto de consolidacao (mesmo nome, semantica diferente entre forks).
- **Confianca.** CONFIRMADA (contagem reprodutivel); o impacto em uso e PROVAVEL.

### D-17 - README nega codigo de XCL; dois arquivos declaram port de XCL (GPL-2.0-only) num repo GPL-3.0-or-later

- **Descricao/sintomas.** Inconsistencia factual com consequencia de licenciamento.
- **Causa raiz.** O codigo entrou pela base (`578d459`, `6a2d9f9`, de fxceless) e o README deste fork foi escrito depois (`3b30d6f`) afirmando o contrario.
- **Commit que introduziu.** Codigo: `578d459` (`console_insn_decode.rs`) e `6a2d9f9`; afirmacao do README: `3b30d6f`.
- **Tentativas de correcao.** Nenhuma.
- **Correcao definitiva.** **SEM CORRECAO.**
- **Arquivos.** `crates/victory-loader/src/console_insn.rs` (linhas 1-6), `crates/victory-loader/src/console_insn_decode.rs` (linhas 1-5), `README.md` (linhas 108-109), `LICENSE`.
- **Evidencia.** `console_insn.rs:5-6`: "Ported from XCL (github.com/ivall/xcl, GPL-2.0-only): `insn.rs` (decoder, as `console_insn_decode.rs`, unchanged) and `eh.rs` `emulate_console_only`." Contra `README.md`: "**[XCL](https://github.com/ivall/xcl)** (GPL-2.0-only) was read for how the same title behaves under another layer; no code of it is here." `Cargo.toml`/`LICENSE`: `GPL-3.0-or-later`.
- **Estado atual.** Contradicao aberta. **Nao e parecer juridico**: GPL-2.0-only e GPL-3.0-or-later sao, em regra, incompatriveis para combinacao direta; a obrigacao concreta NAO VERIFICADA.
- **Confianca.** Contradicao textual CONFIRMADA; consequencia juridica NÃO VERIFICADA.

### D-18 - Constantes de layout fixas por titulo (slots de descritor e formatos de textura)

- **Descricao/sintomas.** Duas features dependem de numeros magicos descobertos por trace de **um** titulo.
- **Causa raiz.** Sem descoberta automatica de layouts; o autor fixou o que mediu.
- **Commit que introduziu.** `82ca134` (ambiente) e `23925a4` (read-back de textura, de fxceless `f5c03f3b`).
- **Tentativas de correcao.** Nenhuma.
- **Correcao definitiva.** **SEM CORRECAO** (por design, com switches de escape).
- **Arquivos.** `crates/victory-gpu/src/ambient.rs`, `crates/victory-gpu/src/texback.rs`.
- **Evidencia.** `ambient.rs`: `pub(crate) const LIGHT: (u8, u32) = (1, 176); pub(crate) const DEPTH: (u8, u32) = (0, 464); pub(crate) const RADIANCE: (u8, u32) = (1, 16);` — "root parameter e posicao na tabela, em dwords", amarrados aos nomes `CS_ScreenProbes_TemporalResolve` e `CS_CacheScreenSpaceRadiance`. `texback.rs`: `const DEFAULT: &str = "256x256f41,256x256f16,64x64f16,128x128f36";` — "the four water formats found by the upstream trace".
- **Estado atual.** Funciona para este titulo; quebra silenciosamente em outro build/idioma/patch do jogo (o read-back simplesmente nao casa nenhum formato, e o barco afunda de novo).
- **Confianca.** CONFIRMADA.

### D-19 - A configuracao documentada nao e a configuracao medida: `play.ps1` liga ~20 switches

- **Descricao/sintomas.** `README.md` apresenta como "What is different here" features que **nao estao ligadas por padrao** no binario (`VICTORY_THIN` exige `=1`; `VICTORY_SHOW_AFTER`, `VICTORY_AMBIENT`, `VICTORY_CARD_STEPS`, `VICTORY_INDIRECT_BATCH`, `VICTORY_RELAY`...). Os numeros de fps do README so sao reproduziveis via `scripts/play.ps1`.
- **Causa raiz.** Nao ha arquivo de configuracao versionado com os defaults do fork (o `victory.example.toml` nao cobre switches); a "configuracao oficial" e um script PowerShell com um hashtable.
- **Commit que introduziu.** `3b30d6f` (README + script no mesmo commit).
- **Tentativas de correcao.** Nenhuma.
- **Correcao definitiva.** **SEM CORRECAO.**
- **Arquivos.** `scripts/play.ps1`, `README.md`, `crates/victory-gpu/src/list/thin.rs` (`wanted()` exige literalmente `"1"`).
- **Evidencia.** `play.ps1:15-26`: hashtable com **16 chaves fixas** (`VICTORY_TEXTURE_NO_WAIT`, `VICTORY_RELAY`, `VICTORY_WIDE_BACK`, `VICTORY_CARD_REHASH_RUNS`, `VICTORY_JOIN_COPIES`, `VICTORY_INDIRECT_WINDOW`, `VICTORY_RELAY_COPY`, `VICTORY_CARD_STEPS`, `VICTORY_INDIRECT_BATCH`, `VICTORY_MAP_THREADS='1'`, `VICTORY_TITLE_PRIORITIES`, `VICTORY_CARD_AWAKE`, `VICTORY_SHOW_AFTER='1'`, `VICTORY_AMBIENT='0.8'`, `VICTORY_D3D12_DRED`, `VICTORY_SKIP_PIPELINES`) + 3 condicionais (`VICTORY_UNDER_WIDE`/`VICTORY_LAND_ECHO` com `-Safe`; `VICTORY_THIN='1'`/`VICTORY_JOIN_PARTS='8'`/`VICTORY_UPLOAD_AHEAD` sem ele) + `VICTORY_BVH_BUILDER` com `-RayTracing`; `thin.rs:56-59` `is_some_and(|v| v == "1")`.
- **Estado atual.** Aberto; qualquer medicao feita com `victory run` puro mede **outro** programa.
- **Confianca.** CONFIRMADA.

---

## E. Relacao de causa e efeito

1. **`8c26061` (clamp indireto com orcamento por lista) -> um registro lixo esgota o orcamento -> draws posteriores rodam vazios -> flicker de arvores/predios.** `449830f` instrumenta (conta registros vazios por frame) e oferece `VICTORY_INDIRECT_REST=0`; nada resolve. Cadeia hoje termina em **observacao**, nao em correcao.

2. **`8c26061` (corte em 65.535 grupos) -> 1.905 tiles nao rodam -> faixas blocky.** `84eae35` mede na placa que a AMD executa todos os grupos e eleva o limite a 2^20 -> compute corrigido. O mesmo raciocinio **nao** foi aplicado a mesh, e `ef556e4`/`a9c2e99` endurecem o corte (mesh > 65.535 = vazio/cortado).

3. **Submissao tardia (preparacao assincrona) -> o titulo le um tamanho de BVH que ainda nao pousou -> aloca do tamanho errado -> copia transborda e sobrescreve estruturas vizinhas.** `2c20d7a` reconhece o programa de "pedido de tamanho" e **forca pouso** antes de continuar. Causa tratada na origem (ordem), nao no sintoma.

4. **`ac11bbb` (16 threads de mapeamento por padrao) -> contençao no lock de mapeamento no dormitorio -> 260 ms contra 125 ms.** `4e99544` reverte o padrao para 8 e deixa `auto` como opt-in. Regressao de desempenho, nao de correcao.

5. **Wave reads do host excluem helper lanes -> laco em waterfall so com helpers nao casa -> sem amostra para derivadas -> buracos quadrados e bordas tracejadas nas sombras.** `9fea743` faz a helper lane devolver o proprio valor (`isHelperLane.i1`).

6. **Indice de view de dados -> fora do heap -> page fault e remocao do dispositivo na RDNA3.** `31714de` (slot nulo + clamp 2^21) e `1f58d91` (raw twins, 2^22) fecham duas portas distintas do mesmo problema. `1f58d91` tambem explica os dois device losses de 2026-11-25 parados em `0452fc2d2e2fc4c3` (meshlet culling).

7. **`c651346` (enviar so dwords alteradas) -> pagina parcialmente reenviada -> estado da placa perdido -> sala sem luz indireta, geometria explodindo (`tile1-0`).** `6ae0696` desliga por padrao. Cadeia termina em **feature morta**: o ganho esperado nao foi realizado e a causa nao foi encontrada.

8. **Fila unica de pipelines + orcamento de 0,5 s por frame -> passes de tela (que rodam uma vez) perdem seus pipelines -> cena sem lampadas nem cor na sombra.** `a989668` cria classificacao `screen_pass`, orcamento de 10 s e workers proprios. Em paralelo, `a30829c` separa pipelines sabidamente lentos e `b7ab9c4` os pre-aquece em disco: tres remendos na **mesma** fila, nenhum escalonador por custo medido.

9. **Textura mostrada antes do upload -> leitura de texels nao escritos -> page fault -> device loss a cada ~16 min.** `3bab23c` poe textura neutra ate o upload pousar (opt-in). Efeito colateral aceito: texturas que acabaram de entrar em cena aparecem tarde.

10. **`VICTORY_THIN` (chamadas de lista so anotadas) + `VICTORY_ASYNC_SUBMIT` -> o titulo passa a nao executar nada na hora -> toda leitura do titulo sobre o resultado de GPU precisa de ordem explicita -> `WHY_READ`/`settle` e o caso BVH (D-05).** A arquitetura de desempenho **cria** a classe de bugs de ordem que depois consome commits de correcao.

11. **`a50c7fd` (replay recria buffer sem `ALLOW_CROSS_ADAPTER`) -> `95037ba` reverte -> limitacao do replay volta sem documentacao.** Cadeia termina em **reversao muda**.

---

## F. Correcoes importantes

### F-1. Slot nulo no fim do heap de views (D-14)

- **O que estava errado.** Indice de view vindo de dados do titulo podia passar do fim do heap; na NVIDIA isso trava a GPU, na RDNA3 remove o dispositivo.
- **Como foi corrigido.** `set_view_heap_limit(null_slot)` grava o limite na traducao; toda leitura de view indexada e clampada para esse slot, que e mantido nulo, e o limite entra na chave do cache de shader (`translation_fingerprint`).
- **Por que funciona.** Transforma comportamento indefinido (leitura fora do heap) em leitura de descritor nulo definida, sem custo de branch por acesso (um `UMin` — o comentario registra que a forma `compare+select` gerava bitcode invalido quando emitida em `dx.op.binary`).
- **Dependencias.** Requer `shader model 6.6` + heap de 2^22; requer que a traducao conheca o limite no momento de emitir (por isso ele entra na impressao digital).
- **Reutilizavel.** Sim, e **obrigatorio** em qualquer camada que rode shaders de console sobre D3D12 com heap limitado. E o mesmo padrao que outros dois forks resolveram de forma independente (`fix/amd-view-heap-clamp` de triippywhale, `fix/nvidia-view-heap-cap` do PR #21 do Marvisak).

### F-2. Raw twins: uma view raw dos mesmos bytes para cada view de buffer do heap (D-13)

- **O que estava errado.** "Picked buffer" (buffer escolhido por indice calculado) lido como bytes crus: na AMD, load raw por view tipada **nao tem fim**; descritor de textura lido como buffer = endereco selvagem; indice alem dos slots do titulo alcanca slots dinamicos de outras listas.
- **Como foi corrigido.** Heaps de 2^22 views; acima de 2^21, para cada view de buffer de slot do titulo existe uma view raw dos mesmos bytes (view de constant buffer para constant buffer; view raw **nula** para todo o resto e para todo slot alem dos do titulo). Picks raw e constant passam a ler ali. Uma view que nao comeca em 16 bytes nao pode ser raw: seu twin e a propria view.
- **Por que funciona.** O acesso passa a ter um limite real imposto pelo hardware, e o tipo do descritor deixa de depender do que o titulo pos no slot.
- **Custo.** "Every shader is translated again (8 min)"; heaps 2x maiores.
- **Conflitos possiveis.** Colide com qualquer outro esquema de "slot reservado" no fim do heap (F-1 usa o ultimo slot; os twins ocupam a faixa 2^21..2^22). Ao consolidar, os dois esquemas precisam ser desenhados **juntos**.
- **Riscos.** `VICTORY_RAW_VIEWS` e parte da traducao; um cache de shader compartilhado entre forks com valores diferentes corrompe silenciosamente.

### F-3. Regras de float da RDNA escritas como a instrucao unica do driver (D-11 / secao I)

- **O que estava errado.** `v_max_f32`/`v_min_f32` ordenam -0 abaixo de +0; `DXIL FMax/FMin` devolvem qualquer zero. `v_mul_legacy_f32` faz `0 * x = 0` (assinatura do zero pode ser a outra). `v_med3_f32` com NaN devolve o **minimo** dos tres. `v_fract_f32` nunca chega a 1.0. A traducao "correta em qualquer host" gastava 7 operacoes por max/min, 5 por produto legacy, etc.
- **Como foi corrigido.** `set_host_rdna(amd && WaveLaneCountMin==32 && WaveLaneCountMax==64)`; com `VICTORY_RDNA_FLOAT` (bits `MAX_MIN|LEGACY|MED3|FRACT`, default todos), cada instrucao vira a operacao unica que o driver compila na instrucao nativa; a analise `bounded()` prova por 12 niveis que um fator nao pode ser infinito/NaN para poder usar produto simples.
- **Por que funciona.** O hardware do host **e** o hardware do console (RDNA2 no console, RDNA3 no host): as regras de NaN/zero sao as mesmas.
- **Ganho medido.** 18,41 -> 17,58 ms no frame do quintal (12 loops), 0,4 ms so dos produtos legacy; imagem igual (79 dB; dois replays do mesmo conjunto diferem por 68).
- **Reutilizavel.** Sim, **e o item de maior valor para uma consolidacao**: e o unico ganho de tradutor medido e reprodutivel em RDNA3.

### F-4. Clamp indireto em compute shader com cobranca por grupo (base, mantido)

- **O que estava errado.** Argumentos indiretos escritos por passes do titulo; um pass dropado ou mal traduzido deixa lixo e "um draw de quatro bilhoes de vertices roda ate o host resetar o driver".
- **Como foi corrigido.** Um compute de 64 threads copia e limita os registros ativos para um scratch device-local, sem tocar nos buffers do titulo; cada grupo cobra o custo dos seus registros com **uma** operacao atomica (soma saturada em ordem de lane), e cada thread le se o seu registro coube.
- **Por que funciona.** O custo e cobrado saturando sem wrap de 32 bits, entao milhares de registros lixo nao rodam por segundos; e o scratch fica legivel como argumentos indiretos depois do clamp, sem barreira por execucao (os draws que leem registros clampados juntos seguem sem nada entre eles).
- **Reutilizavel.** Sim — mas ver D-01: o **rateio** do orcamento e a parte que ainda machuca.

### F-5. Texturas que shaders escrevem e a CPU le voltam para a memoria do titulo (barcos flutuam)

- **O que estava errado.** Mapas de altura/deslocamento do oceano sao escritos por compute e **amostrados pela fisica na CPU**; nunca voltavam para a memoria do titulo. "Boats sank, the sea plane went under."
- **Como foi corrigido.** `23925a4` (de fxeceless `f5c03f3b` / PR 999, Geckomacabre `895c0b8f`, `04df4350`): quando uma lista fecha com escrita de shader numa textura de certos tipos, copia para um read-back buffer e, quando a lista roda, **retila** os texels na memoria do titulo no layout do console.
- **Por que funciona.** A fisica le exatamente os bytes que a GPU teria deixado.
- **Dependencia.** Lista de 4 formatos fixos (D-18) — o ponto fragil.

### F-6. Emulacao de instrucoes AMD que o Intel nao tem (monitorx/mwaitx/clzero/extrq/insertq)

- **O que estava errado.** O console e Zen 2; o titulo usa `monitorx`, `mwaitx`, `clzero`, `rdpru`, SSE4a `extrq/insertq`, `clwb`. Num i9-12900K cada uma levanta `STATUS_ILLEGAL_INSTRUCTION` — **25.000 excecoes por segundo** nos lacos `clzero` de uma funcao.
- **Como foi corrigido.** Handler vetorado decodifica e emula; `2132366` **substitui no lugar** no primeiro fault (`monitorx` -> nop de 3 bytes, `mwaitx` -> `pause` + nop); `13ffc28` cobre waits que passam do fim de 8 bytes e transforma lacos `clzero` em stores simples. Um cache de "regiao lembrada" evita perguntar ao Windows a cada instrucao (a query custava 2-9 ms em heaps de 0,6-2,2 GB e 89 ms no pool de 6,2 GB do titulo).
- **Por que funciona.** O laco do titulo volta a olhar o valor logo depois do wait; `pause`+nop preserva a semantica observavel; o store em vez de `clzero` e identico em memoria coerente.
- **Custo.** Auto-modificacao de codigo do titulo (precisa da pagina gravavel).

### F-7. Detiling de cadeia parcial de mips (`bytes_from`)

- **O que estava errado.** Uma textura streamada em que so os mips menores estao presentes tem outra textura logo atras na memoria; exigir `Layout::size` bytes para detilar falhava com `TooSmall` (e o titulo nao tinha o buffer inteiro).
- **Como foi corrigido.** `Layout::bytes_from(first)` calcula o prefixo necessario: o maior fim de mip a partir de `first` (alinhamento quando o mip tem `tail`); vale o layout inteiro se houver mais de uma slice, se for `thick`, 3D, multisample ou `first >= mips`.
- **Por que funciona.** Os mips menores ficam **no inicio** do bloco tiled; teste `smaller_mips_need_only_the_start_of_the_texture` verifica byte a byte que detilar o prefixo da o mesmo resultado que detilar o todo, e que um byte a menos da `TooSmall`.
- **Origem.** victory-amd `bddfb4a5` (myokex).

---

## G. Problemas ainda existentes

| # | Problema | Evidencia | Estado |
|---|---|---|---|
| G-01 | Flicker de arvores/plantas/predios ao ar livre; suspeita de orcamento indireto (D-01) | README "What is known to be wrong"; findings item 4 | Aberto, causa nao confirmada |
| G-02 | Salas com pouca luz direta escuras sem `-RayTracing`; mitigado por um **estimador** de luz indireta com offsets fixos (D-18) | README; `ambient.rs` | Mitigado por heuristica |
| G-03 | Texturas de objetos que acabaram de entrar em cena aparecem tarde durante o streaming | README; custo de `SHOW_AFTER` | Aberto |
| G-04 | Primeira run com driver/build novo engasga enquanto o driver compila pipelines | README; D-08 | Mitigado |
| G-05 | Gravar um frame no mundo faz o watchdog do titulo encerra-lo | README | Aberto |
| G-06 | Device loss por textura mostrada antes do upload | D-09 | "likely fixed, not proven"; correcao off por padrao |
| G-07 | 40-46 GB de memoria privada (listas 9 GB, janelas 7, uploads 4) | findings "Memory"; README | Aberto |
| G-08 | `replay` nao representa recursos tiled, ray tracing, bundles, barriers enhanced, predication, stream output, ClearUnorderedAccessView, shading rate, `CreateCommittedResource1+` | `docs/replay.md` "Limits" | Limite declarado |
| G-09 | Revert de `a50c7fd` sem explicacao: buffers em heaps abertos da memoria nao voltam ao replay | D-10 | Aberto e nao documentado |
| G-10 | `VICTORY_SEND_CHANGED` morto: envio por dword nao pode ser reativado com seguranca | D-03 | Aberto (causa nao achada) |
| G-11 | 294 switches sem registro, duas convencoes de "booleano" (D-16) | contagem por grep | Aberto |
| G-12 | README nega codigo de XCL; dois arquivos declaram port (D-17) | D-17 | Aberto |
| G-13 | Historico com patches duplicados e 191 merges em 3 dias (D-15) | `Group-Object` | Aberto |
| G-14 | `VICTORY_WAVE_WIDE` (forma wave-wide) **perdeu a placa** nas runs 121-124 | `emit/mod.rs:509-514` | Desligado por padrao |
| G-15 | Nenhum teste de integracao com o titulo; CI roda clippy + testes de unidade sem GPU | `.github/workflows/ci.yml` | Estrutural |
| G-16 | 46 testes `#[ignore]` (inclui o de replay grande e o de `v_cvt_f32_f64` flush-to-zero) | contagem por grep | Aberto |

---

## H. Regressoes

1. **`c651346` -> `6ae0696` (D-03).** Feature de desempenho ligada por padrao que **quebrou a cena** (luz indireta perdida, geometria explodindo). Revertida por padrao, nao por correcao. Regressao classica de "otimizacao que assume coerencia de pagina".
2. **`ac11bbb` -> `4e99544` (D-04).** Otimizacao de carregamento (16 threads de mapeamento) aplicada como padrao global piorou cena que nao estava carregando (dormitorio: 125 -> 260 ms). Mesmo padrao: medicao em um cenario promovida a default universal.
3. **`a50c7fd` -> `95037ba` (D-10).** Correcao de replay revertida integralmente, sem substituta.
4. **`9820bdc` -> `d1c8acd`.** Cherry-pick upstream ("HLE: show title windows through a stand-in that always reads its messages", 322 linhas em `lifecycle.rs`) revertido sem justificativa. Efeito: o cursor de "ocupado" e o fantasma "Not responding" voltam em frames lentos. `git log -S"stand-in that always reads"` mostra apenas origem e reversao — **nenhuma reintroducao**.
5. **`a9c2e99` endurece o clamp de mesh:** depois de `84eae35` liberar dispatches indiretos largos em AMD, `a9c2e99` faz registros mesh > 65.535 rodarem **vazios** e `ef556e4` corta mesh direto. Nao e regressao de bug corrigido, mas e **regressao de capacidade** introduzida para conter um risco de crash — vale saber ao consolidar: o limite de mesh nao acompanhou o de compute.
6. **`3bab23c` (opt-in) como "correcao" de device loss (D-09):** enquanto o switch esta desligado, o device loss continua possivel; enquanto ligado, texturas aparecem tarde. Um trade-off virou default **apenas em `play.ps1`**.
7. **Duplicatas (D-15):** o par `689df31`/`2be0515` significa que o mesmo conserto de corrida esta em duas versoes no historico. Nao ha regressao funcional hoje (arvore limpa), mas qualquer rebase que reaplique a versao antiga **reintroduz** a corrida de D-06.
8. **`VICTORY_MAP_THREADS` default mudou duas vezes** (8 -> auto/16 -> 8, com `auto` opt-in). Quem le apenas `4e99544` pode concluir que "auto" e o padrao; nao e.

---

## I. Conhecimento reutilizavel

Priorizo o que e **exclusivo deste fork** (89 arquivos so dele) e o que e **codigo especifico de GPU**, como pedido.

### I-1. Tradutor: regras de float da RDNA (maximo valor de consolidacao)

- **Problema resolvido.** Instrucoes VALU do console (`v_max_f32`, `v_min_f32`, `v_mul_legacy_f32`, `v_med3_f32`, `v_fract_f32`) traduzidas por formas portaveis custam de 5 a 7 operacoes; nas formas nativas custam 1.
- **Implementacao.** `crates/victory-shader/src/emit/lower.rs`: `max_min()` (linha ~3710), `med3()` (~3847), `fract()` (~3887), `legacy_mul()` (~3940); dispatch em `set_host_rdna` (`emit/lower.rs:1120-1141`); bits em `emit/mod.rs:249-263`; deteccao em `host.rs:922-927` (`WaveLaneCountMin == 32 && WaveLaneCountMax == 64` e `VendorId == 0x1002`).
- **Detalhe que importa.** `max_min` sem o bit RDNA emite a sequencia que preserva a ordem de zeros: `or` dos bits, mascara `0x7FFF_FFFF`, comparacao com zero, escolha do zero com sinal (`and` para max, `or` para min). `legacy_mul` usa `Module::bounded(v, 12)` — analise de **magnitude** que devolve `ceil(log2(|k|))` para constantes e propaga por conversoes (`UiToFp`/`SiToFp` -> 32), com corte em `b < 126` para nao aceitar potencia que estoura — para provar que nenhum fator pode ser infinito/NaN e entao usar produto simples.
- **Ganho.** 18,41 -> 17,58 ms/frame no frame do quintal, 0,4 ms so dos produtos legacy; mesma imagem (79 dB).
- **Dependencias.** Precisa de `WaveLaneCountMin/Max` do host **e** identificacao do vendor; precisa entrar na impressao digital do cache (`translation_fingerprint` ja inclui `host_rdna`).
- **Riscos na consolidacao.** O bit `LEGACY` **muda a assinatura do zero** ("a zero may have the other sign"): qualquer lugar que compare `mul_legacy` por bits (hashes de frame, replay `--compare`, testes de diff) pode divergir. Medido e aceito pelo autor.

### I-2. Tradutor: wave64 do console em wave32 do host, com escada de retry

- **Problema.** Programas de compute wave64 do console rodando num host de waves de 32; barreiras de grupo dentro de branches/loops sao indefinidas quando nem toda thread do grupo as alcanca.
- **Implementacao.** `emit/mod.rs:440-620`: escada ordenada — (1) forma idiom (`lower::wave_local`, exata), (2) remocao de skips que o codigo seguinte nao ve (`ir::without_skips`), (3) `widen()` (branches por wave inteira, **2 barreiras de grupo por branch**), (4) "local waves" aproximadas por host wave, (5) "any mask". Diagnostics por `VICTORY_WAVE_WHY`, `VICTORY_WAVE_LOCAL`, `VICTORY_WAVE_DEEP_SKIPS`, `VICTORY_WAVE_WIDE`, `VICTORY_GROUP_SYNC` (0/1/2) e a **regra de ouro**: com `GROUP_SYNC=0` programas wave64 em duas waves calculam errado (documentado no proprio codigo).
- **Helper lanes.** `first_lane()` (`lower.rs:4785-4803`) e `helper_lane()` (`isHelperLane.i1`, op 221).
- **Risco.** A forma (3) "lost the card in a submission of two programs that had just moved to it (`ptfx_CS_lightAtlas_Sprite` e `_Trail`, runs 121-124)": e opt-in, nao padrao.

### I-3. Descritores do host com o layout da RDNA 2

- **Fato.** Views de 32 bytes e samplers de 16 — **o tamanho dos descritores de imagem e sampler da RDNA 2** — para que um titulo que **copia descritores ele mesmo** mova descritores inteiros e corretos. Views de render target e depth sao so do host; 128 bytes deixam folga para qualquer stride que o titulo suponha (`descriptor.rs:1-7`).
- **Consequencia pratica.** Qualquer tabela de descriptor mirroring, comparacao de descritores do titulo ou decode de bits (I-5) depende desse layout. Ao consolidar com forks que usam 64/32 bytes ou stride diferente, **todo** o espelhamento de descritores quebra.
- **Contrapartida de VRAM.** 2^22 views x 32 B por heap (F-2) e um custo fixo que precisa entrar no orcamento de I-6.

### I-4. `book.rs`: decode dos bits de descritor AMD para diagnostico de device loss

- **Problema.** Um device loss nomeia um endereco que faltou e, de uma lista curta, o que foi liberado ali; uma textura "sumida ha minutos" cai em "nada nomeado"; qual descritor ainda a nomeia nao e dito.
- **Implementacao.** `crates/victory-gpu/src/book.rs` (so deste fork). Mantem "livro" de todo buffer, textura e bloco de textura com o **endereco desde que foi criado** e a hora em que saiu; no loss, quem tem/teve o endereco (mais novo primeiro) e **cada slot do heap cuja descricao alcanca o endereco**. `VICTORY_SLOT_CHECK=1` varre o heap com uma thread enquanto nada esta perdido. Decodifica o descritor de 32 bytes como o hardware AMD tem desde RDNA 2 (`decode`), com teste de bytes reais lidos na RX 7900 XTX (textura 64x64 e 2048x2048, buffer raw).
- **Aplicabilidade.** Ferramenta de campo para qualquer camada D3D12 sobre RDNA 2/3. **Nao** funciona em outra placa por design ("on another card nothing is decoded as an address we know, and nothing is said").
- **Custo.** `GONE = 400_000` entradas; `LONGEST = 2 GiB`; entradas acima de `LONG = 64 MiB` sao procuradas uma a uma.

### I-5. VRAM: o "budget" da RX 7900 XTX e metade do cartaz

- **Fato medido.** `host.rs:9051-9088`: `IDXGIAdapter3::QueryVideoMemoryInfo` com `DXGI_MEMORY_SEGMENT_GROUP_LOCAL`/`NON_LOCAL`. O autor registra que o budget local **caiu de 11.242 MB para ~10.300 MB** no primeiro frame do mundo. Ou seja: numa placa de 24 GB, o orcamento que o Windows reporta e ~11 GiB, nao 24 — e o `victory` nao reserva nada.
- **Bug que isso causou.** Sem reserva, a fila de texturas enchia a placa ate 99-100,5 % e **as copias de buffer eram recusadas** (runs 193/197/198: 652 MB antes do mundo, 2.255-2.352 MB logo depois do primeiro frame, 2.800-3.100 MB dois minutos depois, "none of them idle").
- **Correcao reutilizavel.** `VICTORY_CARD_ROOM_FOR_TARGETS_MB` (padrao **3.072 MB**, teto 8.192, 0 = nenhum) — a fila de texturas deixa essa folga para render targets e copias que ainda nao existem (`resource.rs:1707-1737`, de fxceless `a26859f`).
- **Limiares de "copia de buffer vai para a placa" escalonados por VRAM** (`shadow.rs:180-248`): budget <= 12 GiB -> 16 MiB / 512 MiB; < 20 GiB -> 4 MiB / 2.048 MiB; >= 20 GiB -> 1 MiB / `min(budget/4, 8 GiB)` (= **6.144 MiB** numa de 24 GB). Overrides: `VICTORY_CARD_BUFFERS_MIN_MB`, `VICTORY_CARD_BUFFERS_MB`, e `VICTORY_CARD_BUFFERS_MIN_KB` (desce abaixo de 1 MiB; 64 KiB = todo buffer do titulo, "in the PC's memory every load, store and atomic a shader does on them crosses the bus"). `VICTORY_CARD_ALL=1` põe tudo na placa.
- **Recomendacao de adaptacao.** Nao copiar os numeros; copiar a **forma**: orcar por `QueryVideoMemoryInfo`, reservar folga para targets, escalonar o limiar de copia por VRAM, e logar budget/uso quando o budget varia mais de 256 MB.

### I-6. Alocacao de textura: pool de blocos, nao alocacao por textura

- **Fato.** Na RX 7900 XTX **uma alocacao custa ~310 microssegundos independentemente do tamanho**, numa thread (`texture::allocation_costs`, citado em `pool.rs:1-20`). O pass que cria as texturas que shaders escolhem por indice fazia 350-500 das 24.000 enfileiradas a cada turno de 250 ms — "some forty seconds of a world without its textures".
- **Solucao.** `pool.rs` (so deste fork): texturas e staging cortados de blocos feitos uma vez; peca devolvida so e reutilizada **2 segundos** depois (nada que a placa recebeu antes disso ainda esta rodando — "Windows resets a card that takes that long over one submission") e so quando nenhuma lista a nomeia. `VICTORY_TEXTURE_POOL=0` volta ao antigo.

### I-7. Formato de bloco e tiling: swizzles `SW_*` + `bytes_from`

- **Swizzles do console** (`victory-formats/src/texture.rs:43-55`): `SW_4KB_S/D`, `SW_64KB_S/D`, `SW_64KB_S_T/D_T`, `SW_4KB_S_X/D_X/R_X`, `SW_64KB_Z_X/S_X/D_X/R_X` — a familia addrlib. Tabelas imutáveis compartilhadas (`8961e15`, TG0000) e **teste contra os proprios tamanhos e enderecos da addrlib** (`8f08131`, Vyacheslav Venglovsky) — este teste e o ativo mais valioso para confiar no detiling.
- **Benchmark de layout** (`crates/victory-formats/examples/texture_layout_bench.rs`, so deste fork): BC7 `64KB_D`, BC1 `64KB_S`, RGBA8 `4KB_D`, R8 `64KB_S`, volume RGBA8, linear — 50.000 chamadas por caso, com `black_box`.
- **Cadeia parcial de mips** (`bytes_from`, F-7) — o caso real de streaming de textura.
- **Armadilha que o fork documenta** (`inplace.rs:29-32`): uma textura em heap aberto da memoria do titulo precisa ser **row-major com um nivel**; **tile mappings nesse heap sao recusados e REMOVEM O DISPOSITIVO**. Ou seja: nao se pode combinar "heap aberto do titulo" com "textura tiled do console" — a detiling tem de acontecer na copia.

### I-8. Memoria do titulo: guards por bits, "beside", e o que o Windows recusa

- **Fatos operacionais raros e uteis** (`inplace.rs`): `OpenExistingHeapFromFileMapping` aceita secao `SEC_COMMIT` e **recusa** `SEC_RESERVE` (como o `xmem` faz o pool do titulo); abrir um heap de 256 MiB custa 0,2 ms e nao traz working set; enquanto o heap aberto existe, `VirtualFree`/`VirtualProtect` das paginas dele **falham com erro 87** e nao se pode desmapear uma view; `MEM_WRITE_WATCH = 0x20_0000` nao esta no modulo de memoria do `windows-sys`.
- **Escrita "beside"** (`f19e55b`, `83a1bcf`): em vez de mudar protecao, a escrita do titulo vai por **nossa propria view** da pagina, e o bloco e marcado como escrito por um bit que a proxima pergunta consome — "no protection changes, for the write or for the next taking". `memcpy`/`memmove`/`memset` interceptados (imports nossos) e stores que faultam decodificados no handler; "too many to a block, or code that fills blocks: opened as before". Os dois ficam **desligados** por padrao (`VICTORY_COPY_BESIDE=1`, `VICTORY_STORE_BESIDE=1`) e `scripts/play.ps1` **nao** os liga — so `VICTORY_RELAY`/`VICTORY_RELAY_COPY` (o caminho relacionado, de relay) estao no script.
- **Escritas que nao mudam nada nao sao feitas** (`83a1bcf`): store que faulta ou parte de copia cujo byte ja esta la e omitida; o arquivo `VICTORY_WRITE_SITES=<file>` guarda **offsets** (nao codigo) dos `or byte, const` do titulo que seguem essa regra, aplicados no load.
- **Bloco que acabou de ser escrito "descansa"** antes de ser guardado de novo (`VICTORY_GUARD_REST`, `7bc7c07`), e um fault abre **so a view por onde veio** (`VICTORY_OPEN_ONE_VIEW`).
- **Reuso.** Todo esse conjunto e a resposta ao problema central de camadas de console: notar escrita de CPU sem `VirtualProtect` por pagina. E **ortogonal** ao tradutor e ao backend, entao e o candidato mais direto a entrar num repositorio unificado.

### I-9. Indireto: clamp, orcamento, e o que a placa aguenta

- **Numeros que valem como ponto de partida:** `BUDGET = 2^27` (vertex-instances ou threads por lista; grupo de compute conta 64 threads, mesh 128); `MOST_DRAW = 2^23` (vertices/instancias de um registro de draw); `GROUPS = 2^20` (grupos por dispatch); mesh maximo 65.535; `WIDEST = 2^20` em AMD para compute indireto (testado na 7900 XTX: "every group once, under its number").
- **Heuristicas de "slot nao preenchido"** (todas conservadoras, todas capazes de cortar trabalho legitimo): draw com `want.x > 2^24` ou `want.y > 2^24` quando ha janela `ahead` -> vazio; `want.x >= 2^23` -> vazio; mesh `g.x > 65535` -> vazio. O autor documenta o achado que motivou ("90 of 2^24 vertices in one list", de victory-fork `f85101b6`).
- **Diagnostico no proprio buffer:** palavras 4/8 = maior contagem de mesh e total pedidos; 12/16 = registros recusados por orcamento e maior custo visto (>= 2^22); consumidas pela linha de frame do `Meter`. **Este e o instrumento que falta em outros forks para provar D-01.**

### I-10. Replay de frame (o maior ativo de engenharia exclusivo)

- **O que e.** `crates/victory-gpu/src/replay/*` (15 arquivos, ~13.800 linhas; mais `apps/victory/src/replay.rs` na CLI): grava um frame do mundo (tudo entre dois presents), roda de novo sem o titulo, mede por comando, compara variantes de shader turno a turno, e descobre **qual comando mudou um pixel**.
- **Numeros.** Captura de um frame de tamanho de titulo: +3-10 ms por frame com o gravador ligado; a captura para 5,4 s e ocupa 9 GB (8 GB de memoria da placa a 2,3 GB/s + 1 GB de CPU) em `docs/replay.md`.
- **Capacidades que sustentam todas as conclusoes deste fork:** `--compare`, `--retranslate`, `--shaders-b`, `--pixel`, `--diverge`/`--diverge-picture`, `--upcopies`, `--flow`, `--images`, `--watch`, `--slots`, `--skip-*`, `--buffers vram|sysmem`.
- **Limitacao conceitual documentada:** shaders que distribuem lugares com atomics dao resultados diferentes a cada run; o replay **funde** as copias da CPU com o que a placa escreveu nesta run (`merge.rs`), e `--no-merge` desliga isso. Sem esse cuidado, "lists e o que aponta para elas sao de duas runs".
- **Reuso.** E o instrumento que permite medir GPU sem o jogo. Para consolidar, e mais valioso que qualquer otimizacao individual.

### I-11. Preparacao assincrona: thin lists, submit thread, landing, card steps

- `VICTORY_THIN` (`list/thin.rs`): chamadas de lista so **anotam** argumentos em `CHUNK = 1024` palavras; 8 threads traduzem ao lado. Motivo medido: 45.000-60.000 chamadas/frame, 3,5 us cada, 160-190 ms de CPU.
- `VICTORY_ASYNC_SUBMIT` (`b8b30d1`): uma thread prepara e envia; o titulo nao executa nada na hora — **esta e a origem de toda a classe de bugs de ordem** (D-05).
- `VICTORY_LAND` (`0d5e4c8`, `2aa8134`): o que shaders escreveram volta para a memoria do titulo **ao lado dos guards**, por dwords alteradas, em thread propria; 16 MB/frame.
- `VICTORY_CARD_STEPS` (`=2` no script): fills de buffer que precisam de bytes que ainda estao na placa sao feitos **na placa** (`host/card_steps.rs`) em vez de esperar.
- **Reuso.** Conjunto coerente e medido; mas cada peca depende de contadores proprios e de um modelo de ordem global. Ao consolidar, **nao misturar** com outro modelo de submissao: o par thin+async e indivisivel na pratica (`thin` exige `queue::line_on()`).

### I-12. Impressao digital do tradutor (cache de shader que sobrevive a rebuild)

- **Problema.** 2.500 pipelines levam minutos; um rebuild que nao mudou o tradutor nao deveria invalidar nada.
- **Implementacao.** `victory-gpu/build.rs` (so deste fork): BLAKE3 sobre os **fontes** de `victory-shader` + `victory-shader/Cargo.toml` + `shader_cache.rs` + `Cargo.lock` + `rust-toolchain.toml`, com `\r` removido (line endings iguais em todo checkout) -> `VICTORY_TRANSLATOR` (env em tempo de build). Em runtime, `translation_fingerprint()` (`emit/mod.rs:283-304`) concatena: hash do fonte, `cfg!(feature="diagnostics")`, `cfg!(debug_assertions)`, **valor de cada `VICTORY_*` que o tradutor le** (lista gerada em build), `host_max_wave`, `host_rdna`, `view_heap_limit`, `raw_views`, `loop_limit`, `group_sync`, `shader_opt`.
- **Reuso.** Modelo para qualquer camada que traduza shader: a chave do cache deve ser (fonte do tradutor, switches lidos, fatos do dispositivo), nao o executavel.
- **Risco.** O desenho e correto, mas **quebrou uma vez na pratica**: `VICTORY_BVH_FENCES` teve de ser adicionado explicitamente (`30f46c9` adiciona o nome na lista de `shader_cache.rs:74-77`), o que mostra que a lista de switches e manual e pode esquecer um.

### I-13. CPU do console: instrucoes AMD ausentes no Intel (ver D/F-6)

- Reutilizavel em qualquer camada que rode binarios de console em CPU Intel; a origem e XCL (ver D-17 — **nao copiar sem resolver a licenca**).

### I-14. Pipeline: pre-aquecimento e particao por custo

- `apps/victory/src/warm.rs` + `pipeline/known.rs`: todo pipeline que o titulo cria e escrito em `known-pipelines.bin` (containers em `known-shaders.bin`, no cache — "they are the title's code"); `victory warm` os fabrica em processos idle de 200 (agora 40), grava `warm-done.txt`/`warm-bad.txt`; `VICTORY_EARLY_ALL` faz isso dentro do titulo. Chave por **versao do driver** (`host.driver()` = `IDXGIAdapter::CheckInterfaceSupport(&IDXGIDevice::IID)`) e tradutor.
- `make.rs`: tres filas (tela, lenta, normal), `slow-pipelines.txt` alimentado por medicao real (>= 1 s) em `warm.rs` e em `library.rs` — **duas constantes `SLOW_MS` separadas** (`apps/victory/src/warm.rs:22` `f64 = 1000.0`; `pipeline/library.rs:705` `u64 = 1000`), que precisam concordar e nao sao compartilhadas (a de `library` e `pub(crate)`). Ponto de atencao na consolidacao.

### I-15. O que **nao** vale a pena repetir (medido e descartado)

Registrado pelo autor em `docs/findings.md` e nos comentarios — economiza dias de quem consolidar:
pipeline early unordered para tudo (o compilador da AMD crashou); perfil Series S, texturas menores, segunda fila de desenho, `ASYNC_SUBMIT` off; mapear paginas em 16 threads (D-04); 4 dwords por load (`VICTORY_SHADER_OPT=127`: 18,47 -> 18,45 ms, "the driver joins the loads"); tamanhos de wave por programa de compute (+0,1 a +0,26 ms); `v_mad` como FMA; reciprocos rapidos; buffers "picked" lidos sem perguntar o tipo (0,12 ms); **remover todas as barreiras (0)**; rodar o codigo do console nativamente na placa (o driver do Windows nao aceita codigo cru; "what is left to gain from shader code was guessed at under 1 ms").

---

## J. Conclusao tecnica

**O que este repositorio e.** Um laboratorio de desempenho de tres dias, feito por um agente, com uma disciplina experimental incomum: 176 commits substantivos, dos quais **30 citam explicitamente outro fork/commit como origem**, **15 citam uma run nomeada** e **24 citam medicao em ms**. As decisoes nao sao opinadas — sao bissetadas (`tile1-0` vs `bis1-0`), contadas (`6 de 200`, `0 de 603`, `99 de 230`), replayadas turno a turno (18,47 -> 18,45 ms) e comparadas por imagem (79 dB). O instrumento que torna isso possivel — `crates/victory-gpu/src/replay/*`, ~9.000 linhas — e provavelmente a contribuicao mais duradoura do fork, mais que qualquer otimizacao isolada.

**Qualidade da engenharia.** Alta no codigo, com comentarios que registram **o experimento, o numero e o risco** no lugar onde o leitor vai precisar ("On an RX 7900 XTX it is worth nothing: 18.47 -> 18.45 ms... the driver joins the loads"). O nivel de detalhe GPU-e-specifico e raro: tamanho de descritor da RDNA 2, `isHelperLane` para helper lanes, raw twins para o load sem fim da AMD, slot nulo para o page fault da RDNA3, `monitorx/mwaitx/clzero` no lugar, orcamento de VRAM com o budget real de ~11 GiB numa placa de 24 GB. Baixa na **higiene de configuracao e documentacao**: 294 switches, duas convencoes opostas de "booleano", a configuracao oficial num script PowerShell, o README descrevendo como padrao features que so o script liga, e um README que nega codigo de XCL que dois arquivos declaram ter.

**Maturidade.** O produto **nao** cumpre a meta do proprio README (30 fps): entrega 6-10 fps em perseguicao de rua e 13-20 fps no prologo, com 40-46 GB de RAM privada e uma lista de seis defeitos visuais/crash conhecidos. Mas a trajetoria e real e medida: 280 -> 46-65 ms/frame em dois dias, com 41 minutos sem crash como melhor sessao.

**O que sobrevive a uma consolidacao.** Nesta ordem: (1) o **tradutor RDNA** (float nativo, helper lanes, raw twins, slot nulo, fences de BVH) — e o unico conjunto de ganhos medidos e especificos de hardware; (2) o **subsistema de replay**, que e ferramenta, nao feature; (3) a **maquinaria de escrita do titulo** (`inplace`, `beside`, guards por bit, `write_sites`) e os fatos de Windows/D3D12 que ela documenta; (4) o **gerenciamento de VRAM**; (5) **detiling de mip parcial** e o teste contra addrlib; (6) os **"nao vale a pena"**, que economizam trabalho alheio.

**O que nao sobrevive sem cuidados.** O modelo `THIN`+`ASYNC_SUBMIT`+landing e um sistema fechado (nao misturar com outro modelo de ordem); o orcamento indireto tem um defeito aberto (D-01) e heurísticas que cortam trabalho legitimo; `ambient.rs` e `texback.rs` sao amarrados a um titulo por constantes magicas; e o codigo de emulacao de CPU tem **conflito de licenca declarado** (D-17) que precisa ser resolvido antes de qualquer incorporacao a um repositorio unificado.

**Veredito.** Fork tecnicamente serio, experimentalmente honesto, documentalmente descuidado e juridicamente ambiguo. Para consolidacao, e a melhor fonte de conhecimento **especifico de AMD/RDNA 3** entre os cinco forks — e a pior fonte de configuracao padrao reproduzivel.

---

## K. Indice de commits

621 commits, `git log --format="%h %ad %s" --date=short` (mais novo primeiro). Nenhum omitido.

```text
0a43566 2026-11-25 README: where it is cloned from
3b30d6f 2026-11-25 README, findings and a start script of this fork; fxceless' README kept in docs; the translator's file may be 640 KiB
57e55d0 2026-11-25 Merge commit '449830f6' into local/next
5bf4906 2026-11-25 Merge branch 'day/shaders' into local/next
44629de 2026-11-25 Heaps' records copied as bytes: pages read, compared and sorted out on several threads, empty slots that show nothing left alone; pages of places read by steps on the card asked about once
200a9cc 2026-11-25 Scalar buffer loads four dwords a call (VICTORY_SHADER_OPT=127; off): fxceless c36971b2's shader part
449830f 2026-11-25 The frame line says how many indirect records ran empty for want of their list's budget, and the largest cost; VICTORY_INDIRECT_REST=0: a record that does not fit takes nothing from the ones after it
ca1d519 2026-11-25 The same render targets are not set again (after ZJOU acf478b8, author local); VICTORY_BIND_CACHE=0 sets them every time
c96525d 2026-11-25 A root constant buffer not 256-byte aligned is copied into scratch memory once for the draws of a list that bind it (from Belobug 84ea9c12, romainhedouin)
ab86ef7 2026-11-25 A pipeline a list binds again is not kept and named again (from Belobug 92d353f8, romainhedouin)
b32fb99 2026-11-25 A range shaders write is declared once until the list next writes title memory (from Belobug dd799aad, romainhedouin); VICTORY_DECLARE_ONCE=0 for the old way
d906830 2026-11-25 Merge branch 'day/faults' into local/next
17c4717 2026-11-25 Title buffers' copies down to any size in the card's memory (VICTORY_CARD_BUFFERS_MIN_KB), the card's hashes over the buffer's own bytes (both after fxceless c36971b2)
393e74f 2026-11-25 Merge local/next into day/shaders
53abe45 2026-11-25 wip wide
35bd011 2026-11-25 Merge local/next into day/shaders; pipeline workers are all woken
2be0515 2026-11-25 Views mapped beside guarded ones: all views of the page are opened together, under the lock
23925a4 2026-11-25 Textures shaders write that the title reads on the CPU are copied back into title memory (from fxceless f5c03f3b, pull request 999: Geckomacabre 895c0b8f, 04df4350); on by default
5491ac7 2026-11-25 The pages shaders' writes were put into are noted without a lock
b46de36 2026-11-25 Merge branch 'local/next' into day/faults
30f7859 2026-11-25 Copies in the card's memory: the changed runs sorted once and stably, piece runs joined while built; steps on the card: slots made on several threads
689df31 2026-11-25 Views mapped beside guarded ones: all views of the page are opened together, under the lock
a30829c 2026-11-25 Pipelines known to take long are made beside the others, by a quarter of the workers (VICTORY_PIPELINE_SLOW=0: in the order created)
b620284 2026-11-25 Title bytes under flights' writes compared with AVX2 on threads of their own, the pairs sent up made on them; helpers that work ahead are not kept on the fast cores
95037ba 2026-11-25 Revert "Replay: buffers in heaps opened from memory are made again (without the cross-adapter flag)"
091151c 2026-11-25 Merge day/light
c3e643e 2026-11-25 Sets of heap slots are bits, the heaps' records have threads of their own, what is mirrored of them is counted
f4cc8d8 2026-11-25 Structure builder: the wait for sizes is off (VICTORY_BVH_SIZES_WAIT=1 turns it on): it mended nothing and costs some 20 ms a frame (light-15)
9ab7302 2026-11-25 Test of parts taken: other tests' notes at the address
455f4d6 2026-11-25 Merge branch 'local/next' into day/faults
e8d3d91 2026-11-25 Look-first stubs for moves and vector stores (VICTORY_WRITE_SITES); descriptors copied onto themselves are not written (VICTORY_SAME_RECORDS=0: they are); of tracked pages the parts written are taken (VICTORY_TAKE_PARTS=0: pages); waits for the trackers' locks and the heap sync's parts counted
79615b7 2026-11-25 Merge day/textures
c9b3f45 2026-11-25 The panel is drawn by the diagnostics build only
b7ab9c4 2026-11-25 The title's pipelines written down (VICTORY_PIPELINE_BOOK=1, off) and made ahead of a run, each batch in a process of its own (victory warm)
35155be 2026-11-25 Frame replay: --shaders takes passes of ours too, by their own hash
9b90ad1 2026-11-25 The compare of card copies loads eight places at once (the same hashes)
4e99544 2026-11-25 VICTORY_MAP_THREADS=auto is asked for, not the default: 16 threads map no more pages than one in the dorm, and a frame takes longer
01e1f52 2026-11-25 Merge day/shaders
61a9b88 2026-11-25 Merge day/light
0da08df 2026-11-25 Merge day/faults
158b9aa 2026-11-25 The image's name is kept for VICTORY_WRITE_SITES (the program did not build)
0a9dcea 2026-11-25 Merge day/submit
780f9cc 2026-11-25 VICTORY_PLACED_ORDER is off unless asked for: the case did not come up in three runs
83a1bcf 2026-11-25 Writes of the title's that change nothing are not made: faulting stores and copies beside the guards, and flags it sets that are set (VICTORY_WRITE_SITES=<file>: places changed at load to look first; off without it)
e162982 2026-11-25 What a landing put into title memory is not sent up again (VICTORY_LAND_ECHO=0: sent)
e4d0837 2026-11-25 A flipped slot's quick record is forgotten
adf5706 2026-11-25 Structure builder: accesses stay near the root views, a size that cannot be one counts as none (VICTORY_BVH_BOUNDS=0: neither)
d078074 2026-11-25 Merge day/faults
7eb1850 2026-11-25 Merge day/light
9e73141 2026-11-25 Merge day/textures
7c607d6 2026-11-25 Merge day/submit: the panel, parts of submissions prepared as one (VICTORY_JOIN_PARTS, off), fills cut free of copies
6912fb1 2026-11-25 Frame replay: shaders of another translation (--retranslate, --shaders, --shaders-b turn by turn), times per pass as medians
20b9834 2026-11-25 Max, min, legacy products, medians and fractions as the RDNA card's own (VICTORY_RDNA_FLOAT, on; 0: spelled out as before)
f19e55b 2026-11-25 The title's writes into guarded blocks go in beside the guard: its memcpy, memmove and memset (VICTORY_COPY_BESIDE=1) and single stores that fault (VICTORY_STORE_BESIDE=1); both off
d79ec0e 2026-11-25 Two measurements for placing memory: bytes over the bus by host copy, and what the title asks its memory to be
073d2d5 2026-11-25 The panel shows what helps, and what it costs; nothing but a key asked for while it is off
6c17ce9 2026-11-25 Host heaps' slots are made under locks of their own; the heaps' records and the buffer views picked by index are made on several threads (VICTORY_VIEW_THREADS)
72f7f13 2026-11-25 Ray tracing document: the stand-in, the sizes, what is known
a989668 2026-11-25 Passes over the screen have pipeline workers and a wait of their own (VICTORY_SCREEN_WAIT_MS=0: not); after Belobug 18e57db8
7da523a 2026-11-25 The stand-in for the indirect light can bound what the air scatters (VICTORY_AMBIENT_AIR, off)
ac11bbb 2026-11-25 Pages are mapped by sixteen threads while the title streams as fast as they are mapped, by the caller alone otherwise (VICTORY_MAP_THREADS=auto, which it is unless set)
7bc7c07 2026-11-25 Write faults: a block just written rests before it is guarded again (VICTORY_GUARD_REST=0: not), a fault opens the view it came through alone (VICTORY_OPEN_ONE_VIEW=0: every view), guarded blocks kept as bits
37332f4 2026-11-25 Parts of submissions prepared as one (VICTORY_JOIN_PARTS=8; off), fills cut free of copies at their place (VICTORY_PLACED_ORDER)
82ca134 2026-11-25 A stand-in for the title's traced indirect light (VICTORY_AMBIENT, off): estimated from the light visible surfaces send out
2c20d7a 2026-11-25 Structure builder: submissions that ask for structures' sizes are waited for (VICTORY_BVH_SIZES_WAIT=0: not); a packed structure is one node longer; VICTORY_BVH_TRACE
a50c7fd 2026-11-25 Replay: buffers in heaps opened from memory are made again (without the cross-adapter flag)
30f46c9 2026-11-25 Structure builder: fences around its buffer atomics (VICTORY_BVH_FENCES=0: none)
3a99ee6 2026-11-25 List calls written down and translated by threads of ours (VICTORY_THIN=1, off)
e77a7d9 2026-11-25 Ranges of memory that changed under texture views are joined before they are looked up (DexNom/VIctory 921d8d3b)
3315f29 2026-11-25 The run's numbers on the picture (VICTORY_PANEL=1; off; F3 shows and hides it)
9a6c829 2026-11-25 Shaders: normalized buffer codes give the nearest float, cube gradients take DXIL's base-2 logarithm, d16 data of integer views keeps its bits, unordered views of vertex programs are declared
350c686 2026-11-25 First uploads of textures picked by index run on a copy queue of their own, their slots show them when they have (VICTORY_UPLOAD_AHEAD=1; off); blips found while the camera moves
d1c412c 2026-11-25 A slot written with another texture goes on showing one that can stand for it (VICTORY_KEEP_SHOWN=1; 0: black, then grey); textures that pop are counted (VICTORY_POP_WATCH)
810efe5 2026-11-25 Structure builder and rays: dispatches counted in threads, empty slots never entered, pages that are not there never hit; vertex signatures as DXC orders them
972bb4b 2026-11-25 Merge branch 'night/crash' into local/next
3bab23c 2026-11-25 Title slots show a new texture after its upload has run (VICTORY_SHOW_AFTER=1; off), and the book says more of one
8b01744 2026-11-25 Merge branch 'night/cpu' into local/next
6acfbb8 2026-11-25 A copy's buffer is asked for its bytes when they are first needed (copies of tests have none)
6e5afc7 2026-11-25 Merge branch 'local/next' into night/cpu
4e77a87 2026-11-25 A list makes a view its tables hold once and copies its descriptor (VICTORY_LIST_VIEWS=0: for every table); a draw's tables are gone through without memory of the heap
005c663 2026-11-25 Merge branch 'night/crash' into local/next
a74de31 2026-11-25 Merge branch 'local/next' into night/crash
3b73cb0 2026-11-25 Textures are cut from blocks older than every list on the card (VICTORY_RIPE_BLOCKS=0: from new ones too)
f1d3eb2 2026-11-25 New card memory is waited for until a queue of its own has run after it (VICTORY_WAIT_RESIDENT=0: not)
a1bdba4 2026-11-25 Recording: a lock a descriptor slot, a thread's own answers about buffers, tables left as they are when nothing changed
fe59a13 2026-11-25 Merge branch 'local/next' into night/crash
a7e8c35 2026-11-25 A measurement: textures of new blocks read by lists sent before; the book knows a block again whose pointer something else had
336f613 2026-11-24 A timeline of a few frames, with the chain each present came out of (VICTORY_TIMELINE)
0322b39 2026-11-25 New memory is resident before anything names it (VICTORY_WAIT_RESIDENT=0: as before)
ba837ee 2026-11-25 Merge branch 'night/cpu' into local/next
a0b4909 2026-11-25 Objects let go and lists that ran are kept about two frames longer (VICTORY_BURY_LAG=0: not)
9e30a46 2026-11-25 Scripted keys can be held from one time to another
b77262b 2026-11-25 Merge branch 'night/crash' into local/next
ef556e4 2026-11-25 The crash report names direct mesh dispatches; buffers and groups stay inside what there is
96f2f32 2026-11-25 The threads a frame waits for are kept on the fast cores (VICTORY_FAST_CORES=0: not; 1: without the helpers)
793d8b0 2026-11-25 Merge branch 'night/card' into local/next
c0104bc 2026-11-25 Merge branch 'night/crash' into local/next
6747198 2026-11-25 A book of who has and had the card's memory; a check of every view slot
13ffc28 2026-11-25 Wait instructions over the end of eight bytes are replaced too; clzero loops become plain stores (VICTORY_PATCH_CLZERO=0: emulated)
040aa27 2026-11-25 VICTORY_CARD_AWAKE: menus keep the card awake too (comment)
55babd1 2026-11-25 Merge night/cpu: processor work, wait instructions patched, submit thread (off)
35d4815 2026-11-25 Merge branch 'night/light' into local/next
d2a8f03 2026-11-25 Title memory by address: what streaming brings is taken when the list is submitted, not when it is recorded
51d3f4d 2026-11-25 A texture is not copied onto itself; the process opts out of the efficiency mode (victory-fork a2537066)
28fbbfd 2026-11-25 Title memory by address: every list brings up every copy, whoever made it
5de948c 2026-11-25 Merge branch 'night/textures' into local/next
a0ec112 2026-11-25 Merge branch 'night/crash' into local/next
be085e5 2026-11-25 Merge commit 'f61ecf3b' into night/light
3c07210 2026-11-25 docs: ray tracing, the state after the first runs
c51ca57 2026-11-25 Title memory by address: the dump names what was built and unpacked; a small buffer's results once a list
19be44d 2026-11-25 Merge local/next (f61ecf3b) into night/cpu
2eb7d0c 2026-11-25 Raw twins of views off dwords are moved there too, not null
4dd5710 2026-11-25 Merge local/next into night/textures
59bccac 2026-11-25 The first copy into a texture the title fills goes over zeros
b8b30d1 2026-11-25 Submissions are prepared and sent by a thread of ours (VICTORY_ASYNC_SUBMIT=1, off)
2132366 2026-11-25 AMD's wait instructions are replaced in place at their first fault (VICTORY_PATCH_WAITS=0: emulated each time)
f61ecf3 2026-11-25 Merge branch 'night/card' into local/next
8d6b1a6 2026-11-25 Batched clamp: an unfilled mesh record is all zeros, as the clamp of one leaves it
2c982ee 2026-11-25 Landing goes beside the guards while mappings change; rectangles read without that lock
e156128 2026-11-25 Title memory by address: packed structures that are gone give their pages back; ranges only for views written
2dfba4d 2026-11-25 Texture uploads wait for no mapping, ask no guards and hash cheaply; the sweep looks at a part; bakes pass once for what is new
c95cf7e 2026-11-25 Merge local/next into night/cpu
917e1a7 2026-11-25 Merge branch 'night/crash' into local/next
a9c2e99 2026-11-25 Draw records of 2^23 vertices or more and unfilled mesh records run empty
1420b07 2026-11-25 Every buffer read as raw bytes through the heap is read at its raw twin
96bfd0e 2026-11-25 Stores by address are bounded: title buffers take the builder's only where it was given them to write
73fcfa8 2026-11-25 Merge branch 'local/next' into night/card
2ded49e 2026-11-25 Card memory the CPU maps: scratch switch gone, what the others measured; a test
2c79c5b 2026-11-25 Merge local/next (b69eab84) into night/cpu
4180818 2026-11-25 The title's page mappings are mapped in runs, and not again where they are (VICTORY_MAP_RUNS=0: page by page)
f6db479 2026-11-25 The heaps' records are also looked at between submissions (VICTORY_HEAP_SYNC_EARLY=0: not)
dc2a3ad 2026-11-25 Copies that shaders write are brought up to date together ahead of a submission's steps (VICTORY_REFRESH_AHEAD=0: one by one)
3ead56e 2026-11-25 Refreshes copy and send the 4 KiB parts of a page that were written (VICTORY_PAGE_PARTS=0: the 64 KiB page); the early asking is off
b69eab8 2026-11-25 Merge branch 'night/stream' into local/next
ea375ca 2026-11-25 Merge branch 'local/next' into night/light
565f4a5 2026-11-25 Pages are mapped side by side: eight threads stand in the host's queue instead of one
0d065d3 2026-11-25 The title's idle and lowest threads run below normal: loading no longer waits for idle processors
78389b5 2026-11-25 Merge branch 'night/crash' into local/next
02f36e4 2026-11-25 Views are made beside the heaps the card sees and copied in whole
0c32e34 2026-11-25 Merge branch 'night/textures' into local/next
773b403 2026-11-25 Title memory by address: packed structures go up whole when the builder unpacks them; small buffers' pages stay title memory
f3ba713 2026-11-25 The first copy into a texture the title fills goes over what title memory holds
bde604e 2026-11-25 Title memory by address: pages in arenas, structures' pages made ready ahead
5f86fa3 2026-11-25 VICTORY_CARD_AWAKE: the card's clocks kept up between submissions
59c1f2f 2026-11-25 Merge branch 'night/crash' into local/next
f672130 2026-11-25 The pass over the heap's buffer views is made beside the recording (VICTORY_HEAP_PASS_ASIDE=0: in it)
aff4def 2026-11-25 A submission's fills of upload memory are done together on several threads (VICTORY_WIDE_FILLS=0: one by one)
1a411ae 2026-11-25 A thread asks the write watch ahead of the submissions (VICTORY_ASK_EARLY); the time ahead of the preparing is split
53c5256 2026-11-25 The heaps' records are brought up to date beside the preparing (VICTORY_HEAP_SYNC_ASIDE=0); a made slot is told without the lock (VICTORY_QUICK_MIRROR=0)
52e0c03 2026-11-25 Snapshot memory, host lists and send memory kept for reuse (VICTORY_SNAPSHOT_POOL=8: as before)
40580d7 2026-11-25 The title's bytes under flights' writes go up by whole words (VICTORY_UNDER_WORDS=0: by bytes)
3e64a64 2026-11-25 Card steps: the plan joins its ranges once a round, each place is looked up once
7130d52 2026-11-25 Bakes take what is due now first; textures that change unnoticed are looked at each frame; placed textures over unlike targets said
bd68f5e 2026-11-25 Card memory the CPU maps: windows, sends and scratch (VICTORY_WINDOWS_BAR_MB, VICTORY_SEND_BAR, VICTORY_SCRATCH_BAR); the card's own stamps (VICTORY_CARD_SPANS)
2ba5225 2026-11-25 Merge night/textures: first level viewed, equal uploads left out, exact bakes
f76060a 2026-11-25 Merge night/cpu: a sampling profiler
9136131 2026-11-25 Merge night/stream
d77556c 2026-11-25 Fewer empty staging blocks kept; the raw twins made are counted
b558934 2026-11-25 Structure builder: root views as memory by address, windows of its own on the card, upload memory below 2^37
c278c67 2026-11-25 The process runs above normal priority, pipeline workers below the title's loading
49d7126 2026-11-25 Streaming: mapping calls ask only for what they need; the thread that loads runs ahead; a streaming log
6ff1e14 2026-11-25 Diagnostics: a sampling profiler (VICTORY_PROFILE=<from>,<to>,<ms>,<file>[;...])
8d79927 2026-11-25 Textures from the first level viewed; equal uploads left out; bakes take textures as they are; kept by the card's real room
9526cb2 2026-11-25 Formats: a texture of which only the smaller mips are there detiles from its start
1f58d91 2026-11-25 Shaders read the buffers they pick through raw twins of the heap's views
7d3e179 2026-11-25 Memory the card was given stays with it while there is room
ac34b4d 2026-11-25 Crash report names the indirect executions a list stopped at; constant buffer slots hold their scratch memory
ad30f2c 2026-11-25 Shared locks for buffer and window lookups; early pipeline list without a small cap; crash report names what a list binds
1f1a433 2026-11-25 Speed and room: draws' own records left to the draws in the heap sync, 2^20 table slots, textures kept while the card has room
1c8dd31 2026-11-25 Pixel programs read their quad's neighbours with quad operations: helper lanes take no part in wave reads
8a8ad35 2026-11-25 Crash tracing that can stay on: VICTORY_D3D12_DRED=light and =faults; lost texture writes named
d1c8acd 2026-11-25 Revert "HLE: show title windows through a stand-in that always reads its messages"
ff1bd66 2026-11-25 Merge local/upcopies
751bc32 2026-11-25 Merge local/lighting
b9c18b0 2026-11-25 VIctory e29391f, 13185f5 and 4fc9d70, by hand
64c3178 2026-11-24 Audio: an XMA write block past the ring's end wraps before the first write
913d4f0 2026-11-24 fix(hle): keep apostrophes intact when splitting relaunch args
daeeb28 2026-11-25 VIctory's 88 commits fitted in; clzero loops end as the console's do
70868f8 2026-11-25 XGameSave: a stray or unreadable version folder never costs the save
3729ba0 2026-11-24 gfx10: resolve the opcode name once, at decode time
137bdb5 2026-11-24 README: status of 2026-11-24
3f0055e 2026-11-24 Diagnostics: VICTORY_POKE writes title bytes twice a second
f4d1863 2026-11-25 gpu(list): log initial pipeline bound on reset/reset_x for parity with create_list
9820bdc 2026-11-24 HLE: show title windows through a stand-in that always reads its messages
5f1a45b 2026-11-24 flush a kept save blob that had to be copied
a5c98fe 2026-11-25 GPU: serialize pipeline library atomic writes with unique temp files
9c63a43 2026-11-24 run: stop with a clear message when the title's own DLLs are missing
903b568 2026-11-24 GPU: fences answer for ID3D12Fence1 as documented
8f08131 2026-11-24 Formats: test texture layouts against addrlib's own sizes and addresses
32676e3 2026-11-24 Loader: guard clwb_loop shortcut against u64 overflow and wraparound
241a709 2026-11-24 HLE: guard dialog commands before initialization
200bde4 2026-11-24 Shaders: precompute MD5 round constants and avoid per-block Vec in md5_block
dc155d2 2026-11-24 Shaders: fold empty loop latches containing only dead phis
482ee84 2026-11-24 HLE: allow an explicit maximum title window size
eb84fc3 2026-11-24 Test scalar quadmask lowering on D3D12
2d34e46 2026-11-24 HLE clears the result of blocked address lookups
b6ca119 2026-11-24 Shaders: do not parse VS inputs when reading mesh shader stage I/O
c339306 2026-11-25 Shaders: compress replaced phi chains in remove_trivial_phis
1b744f8 2026-11-24 docs: explain Windows shader-test validator prerequisite
635360d 2026-11-24 Shaders: fold conditional branches to their fallthrough
6bd18af 2026-11-24 HLE: complete XPackage progress callback unregistration
4ae3490 2026-11-24 XNetworking: wire UDP port change callbacks and UTF-16 security queries
20e295f 2026-11-24 HLE: avoid misaligned f32 output slice in XApu
6d24aaa 2026-11-24 Deduplicate image paths in victory census
0d2ac11 2026-11-24 XAsync: canceling a call that already completed leaves it alone, as documented
54d9c1b 2026-11-24 README: how to install Rust on Windows (#1087)
1c0015f 2026-11-24 HLE: constrain the title before suspending it
3e3c491 2026-11-24 GPU: log the TDR settings, and warn when a GPU hang would freeze the PC
74c9545 2026-11-24 GPU: validate null args in create_deserializer to prevent d3d12 AV and add root.rs tests
c0cb573 2026-11-24 GPU: guard zero/overflowing stride in expand_rects and add primitive.rs unit tests
d8b7d66 2026-11-24 Shaders: hardware test for the integer v_min3/v_max3/v_med3
ae0952d 2026-11-24 XMA: gather frame payloads one byte at a time
467bb36 2026-11-24 CLI: add AVX2 pre-flight CPU check before executing titles
b2e23f8 2026-11-24 HLE: test that XGSuggestSwizzleMode picks linear for a 1D texture
ea78bb9 2026-11-24 Loader: a clwb in the last bytes of a page is emulated instead of faulting
41dc5da 2026-11-23 Config: ignore an empty VICTORY_CONFIG
bb0a242 2026-11-24 policy: resolve a `..` in the not-yet-existing part of a path
6ee01b6 2026-11-23 fix(hle): honor the requested executable on relaunch
d23f0b8 2026-11-23 fix(hle): read manifest fields past inline XML comments
7a4f001 2026-11-23 fix(loader): stage prepared images in per-writer files
98b36e1 2026-11-23 fix(pe): keep every name for a shared export ordinal
3af6d8f 2026-11-24 GPU: add synthetic unit tests for query.rs and signature.rs
ed07828 2026-11-24 PE: add synthetic unit tests for implib.rs and layout.rs
c754111 2026-11-24 List successfully emitted shaders in offline translation scans
62d1b81 2026-11-24 GPU: seek directly to private data on final release
6aa6fc1 2026-11-24 loader(crash): verify unwind against known caller frames when RtlCaptureStackBackTrace returns empty
8961e15 2026-11-24 Formats: share immutable texture swizzle tables
eabb826 2026-11-24 docs(architecture): list BCPack as implemented in victory-formats
0aa61ef 2026-11-24 fix(pipeline): retry collisions when creating cache staging files
efbf107 2026-11-24 HLE: implement the RPC runtime's GUID calls
3f497da 2026-11-24 Handle unsupported display deferrals in the HLE test
e8eb93a 2026-11-24 Show on-disk file size in victory inspect
379f4e7 2026-11-24 Formats: reject multisampled linear texture layouts
886afce 2026-11-24 translate: refuse a VICTORY_EMIT_DUMP directory inside the repo
939ba82 2026-11-24 Shaders: print only the operands an instruction has in SP3 text
77582c3 2026-11-23 GPU: VICTORY_* switches set to 0 are off
20d9d1e 2026-11-24 xtask: count \x escapes and 0x hex arrays as encoded runs in the policy check
ea357d4 2026-11-24 Shaders: reuse lane guards for unchanged EXEC within a block
bda8238 2026-11-24 GPU: clear DXGI outputs on out-of-range enumeration
ee054d4 2026-11-23 xmem: report allocations the host refuses
45f9f49 2026-11-24 HLE: require two hex digits in percent escapes
67d1d02 2026-11-24 Census: deduplicate PE imports without hint
5dcfa24 2026-11-24 XStore: wire offline *Result slots and fix OOB XStoreShowProductPageUIResult slot
2e11203 2026-11-23 Kernel: GetModuleFileName of a title image into a null buffer
2ea5f0f 2026-11-23 XGameUi: a text entry's limit is the console's
d02932f 2026-11-24 Shader info: show stage I/O even when instruction decoding fails
803ada0 2026-11-24 HLE: a '>' in a quoted manifest attribute does not end its tag
f4a7ae2 2026-11-23 fix(formats): reject overlapping texture depth slices
c719d0c 2026-11-24 HLE: load a host DLL outside the module cache's lock
4100012 2026-11-23 XStore: a store id without an installed package is an invalid argument
89fb38b 2026-11-24 GPU: every object answers for IUnknown
7e4bb28 2026-11-24 Audio: an XMA read block past the ring's end wraps instead of filling forever
3d567d4 2026-11-24 XAsync: a new block where a result was left behind begins a call
91d41b2 2026-11-24 Loader: suspend a thread before reading its TEB; skip bad dump times
8b4f50f 2026-11-23 fix(pe): validate section raw data bounds
1466eb6 2026-11-24 Loader: clzero loops in one exception; rdpru
fc0d5e7 2026-11-24 Media: a share is refused however it is spelled
8b5f54b 2026-11-24 Audio: an XMA loop's skip is of its first frame, not of its last
717b3e8 2026-11-24 XMA: noise tests, and end a frame at its data instead of looping forever
ade1da1 2026-11-24 XMA: build the inverse transform's rotations once per subframe size
9a34298 2026-11-24 Files: a title run by its file name alone gets its own folder as G:
37e0760 2026-11-24 XTaskQueue: terminating a queue cancels its delayed callbacks too
cd6c15f 2026-11-23 Census: report directory traversal failures
6b9035c 2026-11-24 XAsync: end the call that finished, not whatever call the block holds now
32f970c 2026-11-24 xma: start the frame after a continuation at the next packet that has frames
9e1014d 2026-11-24 GPU: hold SHADER_HEAPS across descriptor mirror and check query offsets
925736d 2026-11-25 Merge fork main a26859f (the first room renders right)
955c8b4 2026-11-25 A texture filled while the runtime remaps takes the pages that are there
a26859f 2026-11-25 The first room renders right: byte-copied descriptor records are mirrored, views off their copy's elements get their own bytes, card room for the first world frame, levels on demand run on the card
ab71750 2026-11-25 Merge local/world: fork main 8dd30ab (every view of a page is guarded)
49c44f6 2026-11-25 Merge fork main 8dd30ab: writes through one view of a page are seen at its other views
6ae0696 2026-11-25 VICTORY_SEND_CHANGED off by default: run tile1-0 lost light and geometry with it
350f7f4 2026-11-25 Title buffer copies stand where the list has them (VICTORY_COPIES_IN_PLACE)
7232f45 2026-11-25 Stores by raw address declare what their dispatch names, not every copy reached so far
240ea7a 2026-11-24 Buffer views that begin between elements read their own bytes
31e2f75 2026-11-25 Merge Each submission's shaders read the bytes it was prepared with (VICTORY
4a8244c 2026-11-25 Merge Merge local/next
47ae173 2026-11-25 Merge commit '572bd4c' into local/upcopies
bf9ebeb 2026-11-25 Merge local/next
ba509c5 2026-11-25 Addresses of ray-tracing structures: views of a structure have no buffer
0b5f191 2026-11-25 A pipeline sync waits for shader writes on the host
0fdcaa3 2026-11-25 victory replay --flow, --images, --until; shader-info names the hashes
18c0129 2026-11-24 Each submission's shaders read the bytes it was prepared with (VICTORY_SEND_OWN)
572bd4c 2026-11-24 Merge local/world: VICTORY_IMAGE_WATCH
84eae35 2026-11-24 Indirect dispatches of more than 65,535 groups run whole (VICTORY_INDIRECT_WIDE)
8ece2ff 2026-11-24 Merge local/shaders2: the dynamic descriptor search
1a93f0f 2026-11-24 Dynamic descriptor slots: the search goes past the cursor as far as it should
4f6bd62 2026-11-24 Merge commit 'c651346' into local/upcopies
5b422ba 2026-11-24 wip: submissions send their own bytes
af2baf5 2026-11-24 Merge victory replay --pixel: which commands change a pixel, in every target
958c34b 2026-11-24 Merge How much of a texture is mapped is read from the runtime's own record
8ad8002 2026-11-24 Merge A refresh of a copy on the card sends the dwords title memory changed,
9713df6 2026-11-24 Merge Merge local/next (e6f492d) into local/tests
c05f949 2026-11-24 victory replay --pixel: which commands change a pixel, in every target
c651346 2026-11-24 A refresh of a copy on the card sends the dwords title memory changed, and no others (VICTORY_SEND_CHANGED, on)
bd80bbe 2026-11-24 Words of the title's image logged each second (VICTORY_IMAGE_WATCH; off)
cd37859 2026-11-24 Merge local/next (e6f492d) into local/tests
c27a8ec 2026-11-24 How much of a texture is mapped is read from the runtime's own record of its views
3bc3455 2026-11-24 Merge local/shaders2: upstream translator and state fixes
8dd30ab 2026-11-25 A write through one view of a page is seen at its other views; page writes counted; levels on demand (off); warm-up; card memory numbers
fbc1830 2026-11-24 DirectStorage reads files on a handle per worker; its page lookups do not wait for the title's mapping calls
e6f492d 2026-11-24 Merge local/copies: holes known by their bits, landed blocks taken
4e3e065 2026-11-24 Typed buffer loads of 16-bit channels pack two per register; skipped devices say why (upstream c430b1d, 100de0e)
75cfb23 2026-11-24 Upstream graphics state fixes: rectangle attributes, sampler bounds, A8 copies, depth footprints
011735f 2026-11-24 Anisotropic samplers with point mips keep them where the card has them (upstream e45bacd, 23fc8a5, 7dfa86f)
c5a2f8d 2026-11-24 Upstream instruction fixes: 64-bit literals, exact normalized ends, flat offsets, rare forms
50983d8 2026-11-24 Merge branch 'local/next' into local/copies
77b82c9 2026-11-24 Mesh primitive exports: corner indices are nine bits, the edge flags dropped (upstream c0228d9)
26a801e 2026-11-24 Immediate sampler slots are kept apart by comparison mode (upstream 2286e03)
cb72fc0 2026-11-24 Merge commit '74045e4' into local/upcopies
74045e4 2026-11-24 Holes of view memory known by their bits; landed blocks taken; askings of views left out
795d55e 2026-11-24 Merge local/upcopies: checks of what goes up to card copies, picked writes (both off)
26d818c 2026-11-24 Kernel reads, remapped views, freed memory and lost writes of title memory
b1da728 2026-11-24 Writes through views picked from the heap come back (VICTORY_PICKED_WRITES)
fecaa14 2026-11-24 Merge local/streaming: requests start at the title's submit
68e10e5 2026-11-24 A DirectStorage queue starts its requests at the title's submit, not at half its capacity
827ec2d 2026-11-24 Merge local/copies (wide textures, joined writes)
535a383 2026-11-24 Merge local/next (a79e248) into local/copies; a test of the bits against remapping
338f424 2026-11-24 Merge local/next (a79e248) into local/tests
151816c 2026-11-24 Title bytes under overlapping shader writes once; textures on several threads; cheaper walks of a submission's steps
a79e248 2026-11-24 Merge local/shaders
2000401 2026-11-24 Merge local/copies
d4254dc 2026-11-24 Merge local/next d4772b3
875a63a 2026-11-24 Buffers loaded past the cache are coherent; record counts of buffers picked from the heap
9c2168a 2026-11-24 Views guarded while they are mapped stay writable; tests take turns on WARP
ee188c1 2026-11-24 Merge e1702e2 (landing on threads, card steps 2) into local/copies
d4772b3 2026-11-24 Merge local/gpucost
002d7fa 2026-11-24 Merge local/shaders
23b0197 2026-11-24 Merge local/tests
017fb0b 2026-11-24 What goes up to card copies: a check, and fetches caught in between put right
e5841a0 2026-11-24 Shader forms: lives of a spill lane, one lane of what a whole-wave section leaves
c802da4 2026-11-24 Merge local/next into local/copies
e1702e2 2026-11-24 Merge local/waits (second version)
007fe55 2026-11-24 Steps done on the card take along what depends on them (VICTORY_CARD_STEPS)
0d5e4c8 2026-11-24 What shaders wrote is landed on a few threads, beside the guards (VICTORY_LAND)
0375962 2026-11-24 Argument clamps in batches (VICTORY_INDIRECT_BATCH)
160d222 2026-11-24 Views guarded while they are mapped stay writable; tests take turns on WARP
2aa8134 2026-11-24 Lighter refreshes of host copies; landing past the guards; the heap pass spread
389db9e 2026-11-24 Shader forms: sizes of views nothing reads, ordered sections, wave sums off the beaten path, counters
65b4c43 2026-11-24 Merge local/waits
d626ecc 2026-11-24 Merge local/replay
e5f735d 2026-11-24 Merge fork main 310c6eb
cc03ecb 2026-11-24 Reads of title memory under flights done on the card (VICTORY_CARD_STEPS)
f669746 2026-11-24 The hash passes over copies in the card's memory walk a piece with 64 threads
4754092 2026-11-24 Frame replay: the CPU's copies leave what the card wrote; buffers in the card's memory fit
bdae3b3 2026-11-24 Frame replay: where identical runs diverge; resources dying as a capture starts
310c6eb 2026-11-24 Packed textures stay in the upload queue; a texture on the card is refreshed when the card is full
c897fa2 2026-11-24 Merge local/replay: frame recorder and replayer
66afb2b 2026-11-24 Only changed dwords go back into title memory; cheap safe copies; title's own arguments; more shader forms
af48187 2026-11-24 Frame replay: usage doc, mesh draws and a title's size in its tests
d7ecfd2 2026-11-24 victory replay, larger tests of it
1deaf0e 2026-11-24 Record and replay a frame's D3D12 work (diagnostics builds)
7913a98 2026-11-24 Merge fork main c96b4f2
c96b4f2 2026-11-24 The depth bounds test works (#82); faster defaults; textures are on the card before the character bake; one host copy per placed texture
1ae8f26 2026-11-24 Merge PR 220
fa967c0 2026-11-24 Merge fork main a803c87
9fea743 2026-11-24 Streaming, copy queue, texture pools and a shader cache that survives rebuilds
a803c87 2026-11-24 Textures picked by index are looked at again after their upload (characters' skin and hair without colour in most launches)
31714de 2026-11-24 Take the fix/amd-view-heap-clamp branch of triippywhale's fork (as one commit)
3bff182 2026-11-24 Take the test/fx-prs branch of asdaasdsaddf's fork (as one commit)
7e239bf 2026-11-25 GPU: the indirect clamp's groups past a count touch the budget not at all
d5c15cb 2026-11-25 GPU: the indirect budget lives in the card's memory
32c6229 2026-11-25 GPU tests: the benchmark also runs with the budget in the card's memory
23daba3 2026-11-25 GPU: the clamp walks only its group's records; benchmark against main's clamp
3db5a0e 2026-11-24 GPU: the indirect clamp charges the budget once per thread group
a4a745e 2026-11-24 GPU tests: a benchmark of the indirect scratch buffer's barriers
cc11ccc 2026-11-24 GPU: the indirect scratch buffer stays readable between executions
2119b0e 2026-11-24 Put the parked lane-mask change (#11) back
9e49b8a 2026-11-24 Take the parked lane-mask change (#11) back out for now
7f166af 2026-11-24 Merge PR 45
1cbc278 2026-11-24 Merge PR 42
675275a 2026-11-24 Merge PR 38
40a2f6a 2026-11-24 Merge PR 23
ffdde86 2026-11-24 Pipeline workers follow the CPU; one pad that never disconnects
2d12b19 2026-11-24 Merge PR 52
cc8d3f7 2026-11-24 Merge PR 40
0e1e518 2026-11-24 Merge PR 25
75f1741 2026-11-24 Merge PR 9
fa06da4 2026-11-24 Merge PR 58
3489be5 2026-11-24 Merge PR 51
751c212 2026-11-24 Merge PR 24
010b90a 2026-11-24 gpu: share video_memory query in heap_textures and lower 16 GB cutoff to 12 GiB
de4ae61 2026-11-24 gpu: lower 16 GB VRAM budget cutoff to 12 GiB and saturate env override shifts
c5b12f1 2026-11-24 Textures: guarded textures are hashed at every check while they may change behind the guard
81a8e2e 2026-11-24 Skip first-half use tracking when no interpolation is present
15cc76c 2026-11-24 Shaders: retain live interpolation first halves
eb5061c 2026-11-24 GPU: no blending on integer render targets in host pipelines
9beff3c 2026-11-24 gpu: scale texture VRAM floor and per-frame staging budget from VRAM
fffafee 2026-11-24 gpu: auto-scale card buffer limits from local VRAM budget
61e5f79 2026-11-24 Shaders: use makeDouble.f64 overload and DXIL ShaderFlags bit 2 for DOUBLES
4d3aaab 2026-11-24 Shaders: lower row permutations on wave32 hosts
05ee438 2026-11-24 GPU: parallelize copy refreshes on multicore hosts
5a32c08 2026-11-24 GPU: document adaptive copy refresh
4e17478 2026-11-24 The game is no longer named in the tree
90228a7 2026-11-25 Shaders: restore parked lane-mask pairs a few instructions apart; relative VGPR reads keep parked lanes (#11)
3930989 2026-11-24 Fix mesh pipeline test on unsupported GPUs (#14)
cb80bf0 2026-11-24 Shaders: lower immediate samplers chosen by uniform control flow
1d2f476 2026-11-24 GPU: constant buffer views of at most 64 KiB
1684989 2026-11-24 Shader clean-up passes; a test that needs a card skips without one
04512df 2026-11-24 hle/gameinput: increase reading history from 32 to 256 for low-FPS frames
b17b69f 2026-11-24 Shaders: model v_add_f64 whole-wave sums and s_mov_b32 copies
7b0f736 2026-11-24 Shader tests: recognise the software adapter from the device's own adapter
e9765a7 2026-11-24 Fix the device loss introduced by the wave64 update
342245e 2026-11-24 Shaders: lower v_cvt_f32_f64 in 32-bit IEEE-754 integer arithmetic
004bf69 2026-11-24 Test fix for hosted runners, stencil diagnostics, and a stability warning
d279e69 2026-11-24 Make host pipeline waits configurable and prewarm pending requests (#1)
6a2d9f9 2026-11-24 Shadow and trace programs translate, lint clean, ray-tracing write-up
578d459 2026-11-24 victory-gta6: this fork's changes on top of VIctory main (ffe310e)
ffe310e 2026-11-24 Shaders: the console's waterfall loop over a first-lane table offset
ca48fbe 2026-11-24 Merge pull request #566 from r4tedx/shader/per-lane-phi
74a9d33 2026-11-24 Shader tests: the waterfall tests take the GPU one at a time, with the VALU tests
e09dbb6 2026-11-24 Merge upstream main (8e79c80) into shader/per-lane-phi
8e79c80 2026-11-24 CI: clippy and tests on pull requests and main
c7a5f49 2026-11-24 Merge pull request #93 from TG0000/perf/bcpack-bucket-lookup
2005e35 2026-11-24 Merge pull request #88 from AndyLatimer/fix/title-size-overflows
58fed37 2026-11-24 Merge pull request #706 from Lyrnc/fix/cmp-lg-ordered
a51bdfa 2026-11-24 Merge pull request #696 from catminou4/devin/formats-doc-micro-link
02f404e 2026-11-24 Merge pull request #685 from catminou4/devin/constant-color-cfg-windows
5cd7421 2026-11-24 Merge pull request #681 from fuzeofficlal/fix/vertex-buffer-null-unbind
8d23f07 2026-11-24 Merge pull request #680 from EatCodeSleep2/fix/wine-staging-upload-128
22700b9 2026-11-24 Merge pull request #675 from lameloball2813-ui/docs/rustdoc-private-links
6db02ea 2026-11-24 Merge pull request #671 from fuzeofficlal/fix/copyresource-cpu-texture-source
16bd577 2026-11-24 Merge pull request #666 from lameloball2813-ui/fix/load-probe-target-dir
b4b4110 2026-11-24 Merge pull request #654 from SantannaCarlos/fix/uav-describe
cf0bb43 2026-11-24 Merge pull request #637 from AlanQiang0418/codex/gfx10-packed-inline-constants
6f0a10a 2026-11-24 Merge pull request #634 from SAitken91/shader/image-sample-o
9a92ccf 2026-11-24 Merge pull request #612 from KoLolX3/fix/phi-liveness-transitive
204bdcc 2026-11-24 Merge pull request #611 from AdvaitPalve11/fix/config-show-data-dir
a4d7b16 2026-11-24 Merge pull request #565 from SAitken91/shader/conservative-depth-position
fd92558 2026-11-24 Merge pull request #520 from Quat3rnion/victory-xasync-payload
dd54de2 2026-11-24 Merge pull request #448 from Dufftwt/fix/min-max-signed-zeros
0ddbce8 2026-11-24 Merge pull request #433 from r4tedx/tests/serialise-valu-gpu-tests
ce42e77 2026-11-24 Merge pull request #432 from Proudnoob4/fix/uninitialized-sgprs
f0894c2 2026-11-24 Merge pull request #429 from marquezinii/fix/xgamesave-documented-errors
9f18eb4 2026-11-24 Merge pull request #423 from pedrofort/shader-perm-no-int64
dd273df 2026-11-24 Merge pull request #419 from itsRevela/fix/swallow-keyboard-menu
3c86238 2026-11-24 Merge pull request #418 from enzoduarte1/gpu/half-subnormal-output
2e4e280 2026-11-24 Merge pull request #384 from LouisRiverstone/shader-cvt16-saturate
faba64c 2026-11-24 Merge pull request #382 from Dufftwt/fix/stencil-ref-shader-flag
c30fc99 2026-11-24 Merge pull request #381 from Dufftwt/fix/psv-sample-frequency
ebf3615 2026-11-24 Merge pull request #376 from LouisRiverstone/shader-cvt-norm-f16
f67c50c 2026-11-24 Merge pull request #372 from marquezinii/fix/capture-unorm16-formats
40b7ed1 2026-11-24 Merge pull request #371 from marquezinii/fix/pls-path-small-buffer
a7e60cf 2026-11-24 Merge upstream main (42361b0) into shader/per-lane-phi
7374bfb 2026-11-24 Merge pull request #365 from FIocker/gpu/publish-initialized-descriptor-heap
42361b0 2026-11-24 Merge pull request #356 from FIocker/gpu/stable-drop-rank
2745589 2026-11-24 Merge pull request #355 from enzoduarte1/clippy/clean-workspace-lints
533b7e2 2026-11-24 Merge pull request #341 from enzoduarte1/config/empty-test-titles-env
edf64e2 2026-11-24 Merge pull request #336 from hophaver/fix/dynamic-descriptor-exhaustion
5a16e66 2026-11-24 Merge pull request #313 from Abdellah-Khanani/loader-aligned-unwind
93c0852 2026-11-24 Merge pull request #285 from Dufftwt/fix/name-title-dll-that-fails-to-prepare
db89843 2026-11-24 Merge pull request #276 from Dufftwt/fix/s-ttracedata-imm-opcode
89e1031 2026-11-24 Merge pull request #260 from Dufftwt/fix/f32-to-half-rounding-carry
88b6658 2026-11-24 Merge pull request #249 from Maraco91/gpu/get-device-newer-levels
56d5032 2026-11-24 Merge pull request #245 from Dufftwt/fix/import-trap-in-crash-log
9e44f38 2026-11-24 Merge pull request #239 from Maraco91/xtaskqueue/terminate-ends-dispatch
7c8cecb 2026-11-24 Merge pull request #229 from Dufftwt/fix/gamertag-modern-suffix-empty
27f9b76 2026-11-24 Merge pull request #209 from Maraco91/xgamesave/empty-configuration-id
6af0fae 2026-11-24 Merge pull request #205 from asdaasdsaddf/fix/null-uav-dimension
27c960b 2026-11-24 Merge pull request #204 from Dufftwt/fix/crash-report-fault-memory
3dd37e6 2026-11-24 Merge pull request #198 from Dufftwt/fix/crash-report-instruction-bytes
13ac0e6 2026-11-24 Merge pull request #167 from Maraco91/xmem/zero-length-unmap
787f665 2026-11-24 Merge pull request #161 from lordon1a/fix/pipeline-staging-cleanup
ba7d873 2026-11-24 Merge pull request #152 from Makto94/fix/net-test-win10
80c0d12 2026-11-24 Merge pull request #140 from lordon1a/feat/cmp-class-f16
3d77d5e 2026-11-24 Merge pull request #117 from tedin7/vic-s-bitcmp-2
b0328ba 2026-11-24 Merge pull request #111 from Dufftwt/fix/narrow-paths-translated-twice
0e990e4 2026-11-24 Merge pull request #104 from TG0000/perf/bitcode-worklist-liveness
c2e7254 2026-11-24 Merge pull request #87 from jeremygold02/codex/copy-stat-frame-window
1f858b0 2026-11-24 Merge pull request #25 from artor410/packed-buffer-formats
3e39eac 2026-11-23 fix(hle): return XAsync payloads for nonnegative results
51f1627 2026-11-24 Shaders: v_cmp_lg is ordered on AMD hosts too (olt | ogt, not fcmp one)
0f0c872 2026-11-23 Docs: drop the private intra-doc link to MICRO in texture.rs
4f53d6f 2026-11-24 GPU: upload CPU-written sources before texture region copies
ab3eb92 2026-11-23 Shaders: gate the constant_color test helper to Windows
31ce30f 2026-11-24 GPU: unbind vertex buffer slots for null view arrays
5864aab 2026-11-23 Docs: rustdoc builds the loader, GPU and shader crates without warnings
af22e5d 2026-11-23 GPU: use the host upload path for texture staging copies
0064c08 2026-11-24 GPU: upload CPU-written textures before CopyResource
764fa19 2026-11-23 Loader tests: build the probe fixture under CARGO_TARGET_TMPDIR
d74558c 2026-11-24 GPU diagnostics: describe a buffer UAV by its range, not as uav None
6c63dea 2026-11-24 Shaders: name why a loop is no waterfall, and test the shapes found in review
b1c8306 2026-11-24 Shaders: prove EXEC selection before collapsing a waterfall
a0efd2d 2026-11-24 Shaders: decode GFX10 packed inline constants by operand type
260611a 2026-11-23 Shaders: image_sample_o with constant offsets
3dd467c 2026-11-23 Shaders: a phi that only another read phi consumes is live too
25b0f88 2026-11-24 Test probe: compare direct and joined reads of a shader timestamp
9a8908c 2026-11-24 core: show data_dir in config and check its asset policy
c249f02 2026-11-23 Shaders: centroid position with conservative depth
6638098 2026-11-24 Shaders: per-lane descriptors chosen by control flow
6285e19 2026-11-24 Shaders: take the first lane's descriptor as each lane's own only in a waterfall loop
e8d3b64 2026-11-24 GPU: the larger dynamic range only where the heap still fits
bf84134 2026-11-24 Shaders: maximums and minimums order -0 below +0, as RDNA2 does
077ec8b 2026-11-24 translate: count per-lane fetched descriptors as loaded
1106f43 2026-11-24 Shader tests: run the VALU GPU tests one at a time
f4438b3 2026-11-23 Shaders: reads of uninitialized SGPRs read as zero, as VGPRs do
f0ee35e 2026-11-24 XGameSave: a full host disk is E_GS_OUT_OF_LOCAL_STORAGE, not the quota
d5704aa 2026-11-24 fix(shader): lower dynamic v_perm_b32 without INT64
28c1b07 2026-11-24 XGameSave: a read result into a small buffer is E_GS_PROVIDED_BUFFER_TOO_SMALL
eb30a0d 2026-11-24 GPU: write half subnormals from f32 rectangle outputs
ff8694a 2026-11-23 Window: keep Alt and F10 from freezing the title in menu mode
2af66ff 2026-11-23 Shaders: v_cvt_u16_f16 and v_cvt_i16_f16 saturate at 16 bits
c2f60d6 2026-11-24 Shaders: a pixel shader writing SV_StencilRef declares StencilRef
404d7fd 2026-11-24 Shaders: PSV0 says a pixel shader runs per sample when an input does
ded334b 2026-11-23 Shaders: lower v_cvt_norm_i16_f16 and v_cvt_norm_u16_f16
0f369d6 2026-11-23 Capture: R16G16B16A16_UNORM targets are integers, format 89 is 10:10:10:2
a4e7f1d 2026-11-24 GPU: replay pending descriptors before publishing the host heap
376b3cf 2026-11-23 Storage: a path buffer too small fails with E_FAIL, as on Xbox
efb3e3d 2026-11-24 GPU diagnostics: order tied drop counts consistently
40c127c 2026-11-23 Chore: clear two workspace Clippy errors
cc63aa2 2026-11-23 Config: ignore an empty VICTORY_TEST_TITLES
3c81420 2026-11-24 GPU: drop draws whose fresh tables cannot be made; 2^18 dynamic slots
b5c6678 2026-11-24 loader: unwind a 16-byte aligned copy of the context
ffa6635 2026-11-24 Shaders: lower waterfall loops over per-lane descriptors in one pass
a3d3bc9 2026-11-24 Shaders: trace descriptors chosen by s_cselect and s_cmov
9fe73ba 2026-11-23 shader: decode the packed typed-buffer formats
5f29f3a 2026-11-24 Loader: a title DLL that fails to prepare is named in the error
717f322 2026-11-24 GPU: GetDevice and the DXGI device hand out the device at every level
63df363 2026-11-24 GPU: a half-float rounding carry raises the exponent, not halves the value
79fb722 2026-11-24 Shaders: s_ttracedata_imm is SOPP opcode 0x28, not 0x26
36d6a79 2026-11-24 XTaskQueue: terminating a queue ends a dispatch waiting on it
5f670bb 2026-11-24 Loader: a trapped unimplemented import goes into the crash log
d46b078 2026-11-24 XUser: a gamertag without a suffix has an empty modern suffix
18b2716 2026-11-24 net: rustfmt the presence assertion
afc4641 2026-11-24 net: skip newer-Windows-only exports in only_loopback_is_reachable
c88b7ac 2026-11-24 Game saves: an empty configuration id is the default, not every configuration's folder
cbb50b8 2026-11-23 GPU: give null texture UAVs the title's view dimension
3390e47 2026-11-24 Crash reports: say what an access violation's address holds
babe2db 2026-11-24 Crash reports: show the bytes of an instruction the CPU refused
02fcf72 2026-11-24 xmem: mapping or unmapping no pages changes nothing
7d73e2b 2026-11-24 pipeline: remove staging files when cache publication fails
d2a4bae 2026-11-24 Merge pull request #129 from Luislariosss/fix/duplicate-test-module
1823079 2026-11-24 Shaders: lower v_cmp_class_f16 and v_cmpx_class_f16
7102a49 2026-11-23 GPU: rename the second test module in host.rs so the tests compile
9aef3d9 2026-11-24 Merge pull request #51 from BaggyALPHA/fix/windows-commit-limit-diagnostic
8a8a38d 2026-11-24 Merge pull request #53 from a55uka/fix/wine-physical-pages
4c0bb89 2026-11-24 Merge pull request #41 from LaunchDoesntCode/codex/proton-warp-safety
91e9bc9 2026-11-24 Merge pull request #46 from 9hvn9yt2vk-netizen/shader-loop-limit
6168fc6 2026-11-24 Merge pull request #70 from FIocker/gpu/diagnostic-pipeline-filter
97464ba 2026-11-24 Merge pull request #61 from TG0000/hle/physical-page-runs
93bc978 2026-11-24 Merge pull request #60 from ifBars/perf/reuse-texture-row-spans
fa5ef73 2026-11-24 Merge pull request #79 from TG0000/perf/texture-check-once-per-submission
0f35dfc 2026-11-24 Merge pull request #66 from AndyLatimer/fix/gpu-rect-element-past-stride
fc0c4dd 2026-11-24 Merge pull request #77 from Exorion1er/fix/guard-fault-race
683278b 2026-11-24 Merge pull request #36 from jeremygold02/codex/root-parameter-fixture
eda38aa 2026-11-24 Merge pull request #57 from Yaekai/fix/null-buffer-views
3c86053 2026-11-24 Shaders: lower s_bitcmp0/1_b32 and s_bitcmp0/1_b64
237f2e8 2026-11-24 Merge pull request #37 from alessiodeia/fix/valu-med3-fract-trig
4926f72 2026-11-24 Merge pull request #80 from tedin7/vic-far-jump-backward
46610eb 2026-11-24 Merge pull request #81 from tedin7/vic-scalar-carry
4a8a058 2026-11-24 Merge pull request #58 from Yaekai/feat/shader-xnor-swap
8fe0b74 2026-11-24 Merge pull request #56 from 9hvn9yt2vk-netizen/find-input-underflow
a6888a6 2026-11-24 Merge pull request #43 from 9hvn9yt2vk-netizen/shader-mask-cycle
062ad14 2026-11-24 Merge pull request #50 from Vanthys/fix/format-offset-overflow
bf6c557 2026-11-24 Merge pull request #75 from tedin7/vic-pe-export-holes
cc51927 2026-11-24 Merge pull request #30 from AndyLatimer/fix/pe-rva-offset-overflow
cbd71a9 2026-11-24 Merge pull request #38 from raffclar/emulation-tests
24480a7 2026-11-24 Shaders: find the written instructions with a worklist, not repeated passes
27e6442 2026-11-24 Merge pull request #28 from ifBars/fix/windows10-import-directory
27b530e 2026-11-24 Files: send rewritten narrow paths to the host, not to our wide wrappers
7d33c46 2026-11-24 Merge pull request #45 from HenriqueSBueno/clwb
3e982f7 2026-11-24 BCPack: find a slot's bucket from a window table
0e5953d 2026-11-23 Merge branch 'main' into fix/windows10-import-directory
38ae55d 2026-11-24 Keep the diagnostic skip outside indirect argument validation
ec0d155 2026-11-24 Watch: saturate the end of announced writes and remaps
257d8bb 2026-11-24 Audio: reduce the title's XMA read advance before adding it
b304b7d 2026-11-23 fix(gpu): average copy statistics over the full reporting interval
a2f5f43 2026-11-24 Shaders: set SCC from the carry of s_addc_u32 and s_subb_u32
c963b71 2026-11-24 fix: read a far jump's offset as signed so backward far jumps decode
73d776b 2026-11-24 GPU: check a sampled texture once per submission, not once per list
4c1766c 2026-11-24 GPU: keep a guarded block's write fault from being declined in a race
9a5cbbe 2026-11-24 fix: skip unused export address slots instead of reporting them at RVA 0
c4e915f 2026-11-23 loader: a clwb write-back loop runs in one exception
0eca412 2026-11-24 GPU diagnostics: skip pipelines by shader hash
78b8404 2026-11-23 xmem: commit physical-page sections whole under Wine
25a3e95 2026-11-23 Keep an empty import descriptor so Windows assigns static TLS
f3c7f0f 2026-11-24 GPU: leave rectangle vertex components past the stride alone
35b054e 2026-11-24 GPU: pipelines off the render thread and cached; indirect execution on
fc807b6 2026-11-23 Reuse texture row spans when converting Xbox pixel layouts
4e94c2b 2026-11-24 xmem: write DirectStorage page destinations one view per run
0c56091 2026-11-24 Shaders: lower v_xnor_b32 and v_swap_b32
071487f 2026-11-24 GPU: emit null buffer SRV/UAV when descriptor names a buffer with null resource
5fbea47 2026-11-24 shader: look up an input channel without underflowing
4da3d63 2026-11-23 Clear the IAT directory alongside the import directory
2e5001c 2026-11-24 shader: keep fract(NaN) as NaN on NVIDIA and add synthetic GPU VALU tests
ef22723 2026-11-24 PE: remove leftover dbg_tls test and check layout offsets
7e42fef 2026-11-23 Report Windows commit-limit failures in crashes.log
4cca767 2026-11-24 Reject empty XMA test fixture directories
e158d95 2026-11-24 Guard texture and XMA offsets against overflow
15b0091 2026-11-24 shader: VICTORY_SHADER_LOOP_LIMIT to find what makes world frames slow
71cfe84 2026-11-23 loader: emulate clwb on Intel CPUs before Ice Lake
ea24ea1 2026-11-24 shader: a lane mask that needs its own value fails instead of overflowing the stack
0b607b1 2026-11-23 GPU: verify WARP adapter is software before using it
f00caec 2026-11-23 Loader: test 64-bit extrq at index 0, not in AMD's undefined range
97c78d1 2026-11-24 Merge pull request #21 from Nostyxx/fix/nvidia-view-heap-cap
d222b87 2026-11-24 Merge pull request #19 from raffclar/exit-watchdog
0c91c03 2026-11-23 Loader: test the AMD-only instruction emulation on any CPU
9e04419 2026-11-24 Merge pull request #15 from 9hvn9yt2vk-netizen/fix-texture-extents
29b558c 2026-11-23 test(gpu): use the SDK root parameter in the serializer fixture
d85097a 2026-11-23 Keep an empty import descriptor for the Windows 10 loader
e98d3ca 2026-11-23 PE: checked file offset in rva_to_file_offset
1dd7afd 2026-11-24 Merge pull request #14 from 9hvn9yt2vk-netizen/fix-xma-zero-length
3827767 2026-11-24 GPU: make tables past a capped host view heap afresh
8c26061 2026-11-24 GPU: the world stays up; indirect execution behind a switch
0b499d7 2026-11-24 Shaders: sine and cosine in turns, exact zeros at half and quarter turns
36eb0b4 2026-11-23 Shaders: med3 with NaN, fract below 1.0, signed bit fields past bit 31
e6a2c54 2026-11-24 texture: refuse extents past 16384 texels instead of overflowing
d348b9a 2026-11-24 Merge pull request #10 from ismailchaka1/emulate-amd-only-instructions
d4ed2de 2026-11-23 Process exit: start the watchdog before other threads end
005ad49 2026-11-24 xma: stop at a zero-length frame instead of looping forever
934e67f 2026-11-23 loader: emulate AMD-only instructions on Intel CPUs
e8f3b1c 2026-11-23 Merge pull request #4 from Exorion1er/fix/tls-callbacks-reloc
c21208c 2026-11-23 Loader: keep the cleared TLS callback pointer out of base relocations
015d45b 2026-11-23 README: status, subsystem table, how to run, debugging switches
2849477 2026-11-23 Run Grand Theft Auto VI from first boot into the world
bc30709 2026-11-23 Thread notices: leave out threads Rust's std starts
4b25269 2026-11-23 Thread notices for title images
d032b8a 2026-11-23 Title DLLs at run time: find by name, map on first LoadLibrary
754619f 2026-11-23 Run Minecraft: runtime HLE, D3D12.X device on the host GPU, shader IR
12caefe 2026-11-23 GFX10 operand fields and SP3 text; FSSM wave size
f4504f6 2026-11-23 Initial commit
```

---

## L. Limitacoes desta analise

1. **Sem acesso a rede.** Nao consultei GitHub, issues, PRs, releases nem o ranking do PortWatch. As referencias a PRs (`#908`, `#1733`, `#946`) aparecem apenas como texto em mensagens de commit do proprio repositorio; nao verifiquei seu conteudo. O dado "5o lugar, 362 estrelas, verificado pelo PortWatch" vem do enunciado, **NAO VERIFICADO** aqui.
2. **Comparacao entre repositorios por conteudo, nao por hash.** Confirmado que o HEAD do Marvisak nao e ancestral deste fork. Nao executei a comparacao blob-a-blob eu mesmo: usei `_fatos/FORK_DIVERGENCIA.md` como fonte para "89 so no fork / 117 blobs diferentes / 17 ausentes". Nao auditei essa contagem arquivo por arquivo.
3. **`git` somente leitura, sem build e sem execucao.** Nao compilei (`cargo`), nao rodei testes, nao rodei o titulo, nao medi FPS. Todo numero citado e do autor (mensagem de commit, comentario de codigo ou `docs/`). Nenhum foi reproduzido.
4. **Replay e instrumentos de medida nao exercitados.** Nao validei que `victory replay` funciona; descrevi o que `docs/replay.md` e o codigo dizem.
5. **Pacote de evidencias com um hash corrompido.** `_evidencias/AvatarManz_evid.md` registra `COMMIT 2be051cbec59334798d16318112eaf725dc1510`; o hash real no repositorio e **`2be0515cbec59334798d16318112eaf725dc1510`** (falta o `5` apos `2be051`). Usei o hash do git. Nao sei se ha outros hashes truncados no pacote; os demais que usei foram verificados com `git cat-file`/`rev-parse`.
6. **Armadilha de autoria que corrigi no meio do trabalho.** `git log --author="local"` casa tambem `pedrofortunato@MacBook-Pro-de-Pedro.local` (o `.local` do hostname). As contagens de "commits do fork" neste documento usam `--author="local@localhost"` (176 sem merges, 276 com). A contagem 182/276 obtida com o filtro ingenuo esta errada e nao foi usada.
7. **Datas de autor nao sao confiaveis.** Ha commits com data de autor 2026-11-23 que aparecem **depois** de commits de 2026-11-24 na ordem topologica (rebase preserva data de autor, muda a de commit). A linha do tempo da secao C usa topologia + datas de forma qualitativa; nao reconstrui a ordem exata de cada um dos 176 commits substantivos.
8. **Profundidade.** Examinei em detalhe ~70 commits (todos os de correcao/reversao/arquitetura citados nas secoes D-H, mais os do pacote de evidencias). Os outros ~550 estao apenas no indice, sem leitura de diff. Os 191 merges foram tratados como transporte, nao como conteudo.
9. **Nao verifiquei** se o CI esta verde, se ha releases binarias, se o repositorio tem tags, nem o historico remoto real (o remoto local e `origin` para `avatarmanz/VIctory-7900XTX`, mas nao ha `git fetch` no ambiente).
10. **D-01 permanece sem causa confirmada** — e o autor diz o mesmo. Nao afirmei o contrario em nenhum ponto.
11. **D-17 nao e parecer juridico.** A contradicao textual e CONFIRMADA; a obrigacao legal decorrente de GPL-2.0-only em obra GPL-3.0-or-later **NAO VERIFICADA** e fora do escopo desta analise de codigo.
12. **Contagens por grep** (294 switches, 98 vs 7 convencoes, 46 `#[ignore]`) dependem do padrao usado e podem subestimar: nao cobrem switches lidos por indirecao (`env::var(name)` com nome de variavel), nem `Cargo.toml` feature flags.
