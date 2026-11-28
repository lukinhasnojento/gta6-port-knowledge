# victory-fork - analise de DeepSeek V4.1 Flash

## A. Identificacao

| Campo | Valor |
|---|---|
| Nome | `victory-fork` |
| URL | https://github.com/fxceless/victory-fork |
| Natureza | Fork de `Marvisak/VIctory` (GPL-3.0-or-later), com a historia upstream preservada **abaixo** de um unico commit de squash |
| Branch analisada | `main` (unica branch local e remota; `git branch -a` devolve so `main` e `origin/main`) |
| Remoto | `origin https://github.com/fxceless/victory-fork.git` |
| Commit 0 (raiz) | `f4504f6413ce4509604dad66cdd689000a0204dc` - 2026-11-23 01:15:26 +0100 - Marvis - "Initial commit" |
| Base upstream declarada | `ffe310e` (2026-11-24 10:53) - "Shaders: the console's waterfall loop over a first-lane table offset", tip do `main` do VIctory no momento do fork |
| Primeiro commit proprio | `578d459c344aa44753e9a61ea6ba10a8c68f6ea6` - 2026-11-24 13:02 - fxceless - "victory-gta6: this fork's changes on top of VIctory main (ffe310e)" |
| Ultimo commit (HEAD) | `6d20eed6ae8afe37c52f1911b46537b0ea8bcd0d` - 2026-11-25 18:02:01 -0500 - fxceless |
| Periodo coberto | 2026-11-23 a 2026-11-25 (3 dias com commit: 48 / 182 / 23) |
| Commits em HEAD | 253 (`git rev-list --count HEAD`), 160 nao-merge, 93 merge |
| Commits examinados em profundidade | 40 (posicoes 214-253, ou seja **todo** o historico proprio do fork) + 12 commits upstream relevantes ao delta (`35b054e`, `3827767`, `8c26061`, `8e79c80`, `ffe310e`, entre outros) |
| Arquivos versionados | 187; 127.534 linhas em `*.rs` |
| Modelo responsavel | DeepSeek V4.1 Flash (opencode-go), via harness OpenCode |
| Data da analise | 2026-11-25 |

### A.1 Nota critica sobre a historia

O HEAD do Marvisak **nao e ancestral** deste fork. `git merge-base` com qualquer ref do repositorio base falha ("Not a valid commit name"), conforme registrado em `analise/_fatos/FORK_DIVERGENCIA.md`. Os cinco forks analisados compartilham a mesma raiz `f4504f6`, mas divergem imediatamente depois dela: houve reescrita de historia. Consequencia pratica, respeitada em todo este documento: **nenhuma comparacao e feita por hash entre repositorios**, apenas por conteudo de arquivo (blob por arquivo), e todo hash citado pertence a este repositorio.

Consequencia secundaria, visivel no proprio grafo: 213 dos 253 commits sao o historico do VIctory (de Marvis e ~40 contribuidores externos) rehospedado; o trabalho do fork comeca no commit 214 (`578d459`), que carrega sozinho **61 arquivos, +32.334 / -1.349 linhas** - ou seja, o delta inteiro do fork foi comprimido em um commit e a partir dele evoluiu em 39 commits.

### A.2 Identidade do autor

O prompt descreve o autor como "hyphennn". O autor dos commits do fork e da conta GitHub `fxceless` (`fxceless <36782933+fxceless@users.noreply.github.com>`), e o README declara que "This fork's changes were developed with Claude Code". A ligacao `hyphennn` <-> `fxceless` <-> build privado lider do PortWatch (59,4%) **nao pode ser verificada** com o material disponivel: **NAO VERIFICADA**. O mesmo vale para "ZETTROX e Uilian partem deste fork": nao ha evidencia dentro deste repositorio - **NAO VERIFICADA**.

---

## B. Resumo tecnico

### B.1 Proposito

Camada de compatibilidade que executa o build Xbox Series de um titulo AAA (o README atual diz apenas "the title"; ate `4e17478` dizia o nome) nativamente no Windows, a partir de arquivos fornecidos pelo usuario. Nao contem arquivos de jogo. O objetivo declarado do fork e unico e explicito: "getting the game's world to render correctly and playable on one particular PC" - Windows 11, i7-14700KF, RTX 4070 12 GB, 32 GB.

O fork nao e um fork cosmetico. Contra a base `ffe310e`, o delta e de **117 arquivos, +83.615 / -1.778 linhas** (`git diff --shortstat ffe310e HEAD`) - cerca de dois tercos das 127.534 linhas Rust da arvore. Em volume, este fork e efetivamente um `main` paralelo.

### B.2 Arquitetura (crates)

| Crate | Papel |
|---|---|
| `victory-loader` | Mapeia e executa o PE do titulo, resolve imports, emula instrucoes AMD-only em Intel, unwinding alinhado, relatorios de crash |
| `victory-hle` | Servicos de plataforma (Xbox): `xmem` (memoria fisica, alias, commits), `dstorage`, `xgamesave`, `xuser`, `audio` (XMA/Opus), `gameinput`, `lifecycle`, `d3d12x` (device D3D12.X) |
| `victory-gpu` | Back end D3D12: listas, descritores, sombras/copias de memoria do titulo, texturas, indirect, pipeline library, filas |
| `victory-shader` | Recompilador GFX10 -> DXIL: decode, IR, estrutura, lowering, formas wave64, passes de otimizacao, testes VALU em GPU |
| `victory-formats` | Formatos do console: texturas tileadas, BCPack, XMA |
| `victory-game` | Identidades de build, patches e hooks por build do titulo |
| `victory-core` / `victory-pe` / `victory-pipeline` | Config/policy, parsing PE, cache content-addressed de pipelines e shaders |
| `apps/victory` | CLI: `run`, `translate`, `--warm`, `--pipelines`, diagnostico |

### B.3 O que o fork acrescenta (modulos proprios)

30 arquivos existem **somente** neste fork (lista de `_fatos/FORK_DIVERGENCIA.md`). Os principais, todos em `crates/victory-gpu/src/`:

- **Instrumentacao**: `framelog.rs` (cada frame e sua composicao), `cardmeter.rs` (tempo da placa por tipo de lista), `profile.rs` + `profile/tests.rs` (profiler amostral interno, 2.961 linhas), `shot.rs` (captura de frame dentro da camada, 1.016 linhas), `fpsbox.rs` (fps desenhado no back buffer), `drvwatch.rs` (watch do event log do driver), `backreads.rs` (quem toca o que o copy-back gravou, 694 linhas).
- **Caminhos de copia**: `inplace.rs` (memoria do titulo como heap grafico), `plaincopy.rs` (copia in-place com `rep movsb` + handler de excecao), `queue2.rs` (copy queues dedicadas), `spread.rs` (espalhar threads do titulo por mais CPUs), `texback.rs` (texturas escritas por shader copiadas de volta para a memoria do titulo), `tiles.rs` (levels on demand, 1.419 linhas na introducao).
- **Shader**: `emit/own_bits.rs` (914 linhas), `emit/wave64_tests.rs`, `emit/bitcode/opt.rs` (passes de limpeza, 2.136 linhas na introducao), `emit/bitcode/census.rs`, `emit/intersect.rs` + testes (ray/box e ray/triangle em DXIL), `bvh.rs`, `spill.rs`, `emit/threading.rs`.
- **Loader**: `console_insn.rs` / `console_insn_decode.rs`.
- **Docs**: `docs/fork-ray-tracing.md`, `diagnostics/README.md` + `diagnostics/timing-run-2026-11-25.txt` (log higienizado de 60 s).

### B.4 Mecanismo central herdado (e estendido)

Toda a corretude depende de um unico contrato, descrito em `crates/victory-gpu/src/watch.rs`: o titulo tem memoria que a CPU e a GPU compartilham no console; no PC a camada mantem **copias host** e precisa saber o que mudou de cada lado.

1. **Titulo -> camada**: `xmem` reserva a memoria de GPU com write-watch (o kernel mantem bit sujo por pagina de 4 KiB); a camada pergunta que paginas foram escritas (a pergunta tambem limpa o bit) e mantem um journal por pagina, porque copias host se sobrepoem.
2. **Memoria de pagina fisica** (secoes do `xmem`) nao pode ser write-watched: a copia deixa cada bloco de 64 KiB `read-only` e um handler vetorizado toma a primeira escrita, marca o bloco e devolve o acesso.
3. **Camada -> titulo**: o que os shaders escreveram volta por `copy_back`, por peca de 64 KiB, e o fork adiciona o caminho "na placa": copia em memoria da GPU, comparacao por hash na GPU e fetch so das pecas diferentes (`shadow.rs`).
4. **Toda view de uma pagina fisica e protegida** (nao so o endereco que a copia tomou) - correcao de `8dd30ab`, codigo de myokex/victory-amd adaptado de PR #115/#1122.

### B.5 Estado atual medido (README de HEAD)

- Primeiras salas: **8 a 10 fps**, 97-104 ms por frame a ~3.850 draws, 122 ms numa view mais pesada (6.774 draws).
- Cutscene de abertura: 5 a 8,5 fps por trecho (runs 271-273).
- Menu/inicializacao: 30 fps (como upstream).
- Sombra: **preto** - o bounce light e as sombras por ray tracing do titulo ainda nao rodam.
- Memoria: ~24 GB no processo, 16 GB deles em memoria; placa de 12 GB quase cheia (9,2 de 10,3 GB de orcamento), 2.073 passes sem textura por falta de espaco, 44.613 texturas liberadas e refeitas em 13 minutos.
- Todos os numeros de fps foram medidos **com um conjunto de switches que nao e o default** (o conjunto de `diagnostics/README.md`), e o proprio README avisa: "The plain defaults were last measured at 420 to 450 ms a frame and not since."

---

## C. Linha do tempo

### Fase 0 - Base upstream rehospedada (commits 1-213, 2026-11-23 a 2026-11-24)

Historico do VIctory: `f4504f6` (raiz), `12caefe` (campos GFX10/SP3), `754619f` ("Run Minecraft" - prova de conceito HLE), `d032b8a`/`4b25269`/`bc30709` (DLLs tardias, avisos de thread), `2849477` ("Run Grand Theft Auto VI from first boot into the world"), `015d45b` (README). Depois o ciclo de PRs externos: `c21208c`/`e8f3b1c` (reloc de TLS), `934e67f` (instrucoes AMD em Intel), `005ad49` (XMA com frame de tamanho zero), `8c26061` ("the world stays up; indirect execution behind a switch"), `35b054e` ("pipelines off the render thread and cached; indirect execution on"), `3827767` ("make tables past a capped host view heap afresh"). Fecha com `8e79c80` ("CI: clippy and tests on pull requests and main", `.github/workflows/ci.yml`) e `ffe310e`.

### Fase 1 - O squash que cria o fork (24/11, 13:02)

`578d459` carrega todo o delta: renderizacao de mundo, iluminacao, frame rate e memoria. O README desse commit lista o que consertou: meshlet culling lendo o tipo errado de view; tabelas de descritor por draw esgotadas ("the card hung"); luz empacotada por compute lida de copias vazias (ceu, probes, nevoa); programas que nao traduziam (fog, shadow-mask, hair, skinning, fogo); gathers em mip nomeado lendo nivel 0; samples implicit-level fora de pixel shader gerando DXIL invalido em 730 programas compute. Estado: **~3 fps e tudo em sombra preto**.

### Fase 2 - Corretude de shader e de imagem (24/11, 13:48-22:19)

- `6a2d9f9`: 59 de 64 programas de trace traduzem; 51 saem das formas wave64 aproximadas; lint limpo; nasce `own_bits.rs`; **e a forma "wide" wave64 passa a ser o default** (`is_none_or(|v| v != "0")`).
- `d279e69` (ZazaJr24, PR #1): esperas de pipeline configuraveis e prewarm.
- `004bf69`: aviso de estabilidade no README - "the newest commits lose the GPU device shortly after start on the test machine (4 of 4 runs)", diagnostico de stencil.
- `e9765a7`: **correcao** - a forma wide poe barreiras de grupo dentro de branches/loops e derrubou o device em `ptfx_CS_lightAtlas_Sprite`/`_Trail`; passa a opt-in (`VICTORY_WAVE_WIDE=1`) e a chave do cache de shader passa a incluir os switches de wave.
- `7b0f736`, `1684989` (passes de limpeza no DXIL: -43% instrucoes host, ~20 ms/frame; `VICTORY_SHADER_OPT`), `3930989` (teste de mesh pipeline em GPU sem suporte), `90228a7` (PR #11 de cxccxc: pares de lane-mask "parked" restaurados com look-ahead; leituras VGPR relativas nao desligam mais o unspill; traduzem `CS_IrradianceCache_Shade`/`ShadeSecondary`).
- `4e17478`: o nome do jogo sai da arvore ("the title"). Sem mudanca de codigo.
- `9e49b8a` -> `2119b0e`: a mudanca #11 e tirada ("lighting gain could not be shown, ~40 ms de GPU") e **devolvida no minuto seguinte** ("that was not wanted. The tree is again as it was before that commit").
- `3bff182` e `31714de`: branches de forks alheios entram como um commit cada. `31714de` traz o clamp de view heap (ver D-08).
- `a803c87`: texturas escolhidas por indice sao reexaminadas depois do upload (rosto/cabelo do personagem sem cor em 3 de 4 lancamentos).
- `c96b4f2`: depth bounds test funciona (#82); novos defaults (relay em thread propria, wide copy-back, copias na placa de 1 MiB a 1.024 MB, 256 listas em voo): ~230 ms/frame onde era ~400 ms.
- `310c6eb`: texturas empacotadas ficam na fila de upload; regra de staleness por sobreposicao total e substituida por "comeca dentro dos primeiros 4 KiB" (D-05).

### Fase 3 - "A primeira sala renderiza certo" (25/11, 00:44-01:46)

`8dd30ab`: guarda sobre **todas** as views de uma pagina fisica (0,8-1,0 milhao de escritas por outra view em 5 min); leituras do kernel ao lado da copia; levels on demand (off); warm-up `translate --warm`; numeros de memoria da placa. `a26859f`: registros de descritor copiados como bytes sao espelhados (~40.000 por frame), views fora dos elementos da copia ganham bytes proprios, espaco reservado na placa para o primeiro frame do mundo, levels on demand roda na placa pela primeira vez. O README anuncia: "the first room now renders right on the test machine (walls, textures, people, lamps), at 190 to 450 ms a frame".

### Fase 4 - Campanha de frame rate (25/11, 02:30-07:43)

`9376fd6`: fingerprint barato no descriptor sync (70-220 ms -> ~3 ms), slots de tabela deixados para seus draws, write-watch sem chamadas falhas; fetch na placa dentro dos proprios flights (PR #648, off). `f85101b`: thread de submit propria (`VICTORY_ASYNC_SUBMIT`), copy queue (`VICTORY_QUEUE2`), janelas de indirect draw mais seguras (PR #220 com o guard de 2^23 vertices), profiler amostral, pool de listas (7866 insercoes). `a253706`: frame log, card meter, driver watch, remocao da auto-copia proibida, e a descoberta de que os "dips" regulares eram o harness de captura (235 ms por frame salvo). `0b0074e`: copia in-place consertada (D-09), threads da camada nos cores rapidos, e **retratacao** da comparacao de draw windows (D-14).

### Fase 5 - Memoria, captura e o custo real do copy-back (25/11, 07:05-14:33)

`895c0b8` (cherry-pick de Geckomacabre, autor-date 24/11 23:58) e sua continuacao `04df435` (Codex): texturas escritas por shader que a CPU do titulo le sao copiadas de volta - e o que faz o barco flutuar. `3973179`, `5557224` (PR #1014: hash de peca por grupo de 64 threads), `465b94f` (indirect records agrupados), `ca8829f` (fps no canto, refresh por bytes mudados), `d510ee4` (duas tentativas de page mapping **revertidas**), `dd3a14b`, `79cfdd9` (frame shot, window upload por resizable BAR, RenderDoc que nao sobreviveu), `085ee7c`.

### Fase 6 - Fechamento (25/11, 16:06-18:02)

`f5c03f3` mergeia o PR #999 (`VICTORY_TEXTURE_BACK=1`, off). `c36971b`: copias de buffers pequenos na placa (`VICTORY_CARD_BUFFERS_MIN_KB=64`), `changed_runs` por mascara de bytes, loads escalares de 4 dwords por chamada (bit 64 do `VICTORY_SHADER_OPT`), `VICTORY_BACK_READS` para medir o copy-back. `9c184b3`, `44bff61` (log de diagnostico higienizado, `VICTORY_ABSENT_ONCE`), `d827419` (run 279, o melhor conjunto ate aqui), `1391409` (merge do PR #1014) e `6d20eed`, que **torna o hash de 64 threads opt-in**, adiciona a lista de copias sujas e as notas de blocos nao mapeados.

### C.1 Distribuicao

| Dia | Commits | Marco |
|---|---|---|
| 2026-11-23 | 48 | base upstream + primeiros PRs |
| 2026-11-24 | 182 | world render, shaders, texturas, primeiro squash do fork |
| 2026-11-25 | 23 | frame rate, memoria, instrumentacao, merges |

Dos 40 commits do fork: 24 em 24/11 e 16 em 25/11. A janela real de desenvolvimento e de **~29 horas** entre `578d459` e `6d20eed`.

---

## D. Catalogo de defeitos

> Regra aplicada: mensagem de commit nao e prova. Onde so existe a mensagem, a confianca e rebaixada e a evidencia e citada como "assunto/mensagem", nao como resultado de teste.

### D-01 - Forma wave64 "wide" derruba o device no inicio da execucao

- **Descricao / sintomas**: logo apos a atualizacao de wave64, 4 de 4 execucoes perdem o device pouco depois do inicio. README de `004bf69`: "**the newest commits lose the GPU device shortly after start on the test machine (4 of 4 runs), cause not yet found; the last tree that ran clean is the first commit of this fork, `578d459`**".
- **Causa raiz**: a forma "wave-wide" (`widen`, `structure_wide`) poe **barreiras de grupo dentro de branches e loops** para fazer os dois half-waves de um programa wave64 trocarem dados. A submissao de `ptfx_CS_lightAtlas_Sprite` e `_Trail` nessa forma trava a placa ate o reset.
- **Commit que introduziu**: `6a2d9f9` - unica linha que decide o default: `+ let wide_on = std::env::var_os("VICTORY_WAVE_WIDE").is_none_or(|v| v != "0");` (`crates/victory-shader/src/emit/mod.rs`).
- **Tentativas de correcao**: `004bf69` apenas documenta e adiciona diagnostico (plano de stencil, dump). Nao ha tentativa intermediaria de correcao logica.
- **Correcao definitiva**: `e9765a7` - `let wide_on = std::env::var_os("VICTORY_WAVE_WIDE").is_some_and(|v| v == "1");`, com comentario que nomeia os dois programas. A forma de 32 lanes, que nao tem barreira, permanece ligada (ver nota em D-02).
- **Arquivos**: `crates/victory-shader/src/emit/mod.rs`, `crates/victory-gpu/src/shader_cache.rs`, `README.md`.
- **Evidencias**: `git show 6a2d9f9 -- crates/victory-shader/src/emit/mod.rs`; `git show e9765a7 -- crates/victory-shader/src/emit/mod.rs`; `git show 004bf69 -- README.md`.
- **Estado atual**: corrigido e estavel nos runs seguintes ("no driver hang or device loss in normal runs", README de HEAD). A forma wide continua existindo, opt-in, para quem quiser reproduzir.
- **Confianca**: **CONFIRMADA** (introducao e inversao do default sao diffs; a relacao com os 4/4 device losses e afirmada pelo autor, nao medida por mim).

### D-02 - Chave do cache de shader nao incluia os switches de wave (traducoes envenenadas)

- **Descricao / sintomas**: uma execucao com `VICTORY_WAVE_WIDE=0` recebia as traducoes feitas por uma execucao anterior **com o switch ausente** (isto e, com a forma wide). O sintoma observavel e o pior possivel em arqueologia de performance: o experimento "desligado" roda o codigo "ligado" e o resultado e atribuido ao switch errado.
- **Causa raiz**: na epoca (`git show 6a2d9f9:crates/victory-gpu/src/shader_cache.rs`) a chave era `BLAKE3(bytes do proprio executavel || loop_limit || group_sync)`. Como o hash e do **binario**, duas execucoes do MESMO binario com switches diferentes compartilham a chave: o switch mudava o codigo gerado mas nao a identidade do artefato.
- **Commit que introduziu**: `6a2d9f9`, que introduz os switches `VICTORY_WAVE_WIDE`/`_DEEP_SKIPS`/`_OWN_BITS`/`VICTORY_F16_NEAREST` como entradas que mudam a emissao sem acrescenta-las a chave. O esquema de chave por executavel e herdado do fork base/upstream, nao e um erro de `6a2d9f9` isoladamente - o erro e a **combinacao**.
- **Tentativas de correcao**: nenhuma anterior; a correcao veio junto com D-01.
- **Correcao definitiva**: `e9765a7` - append de nome, valor (ou vazio) e NUL para os quatro switches. Comentario: "without them here a run with `VICTORY_WAVE_WIDE=0` was served the translations an earlier run of the same executable had cached with the switch unset". Depois, `8dd30ab` substitui o esquema inteiro: a chave passa a ser `translation_fingerprint()` (fonte do tradutor + fatos do host + switches) e o `build.rs` do crate de shader **descobre sozinho** todo literal `"VICTORY_...` em `crates/victory-shader/src/**/*.rs` e injeta nome e valor na chave.
- **Arquivos**: `crates/victory-gpu/src/shader_cache.rs`, `crates/victory-shader/src/emit/mod.rs`, `crates/victory-shader/build.rs`.
- **Evidencias**: `git show e9765a7 -- crates/victory-gpu/src/shader_cache.rs`; `git show 6a2d9f9:crates/victory-gpu/src/shader_cache.rs` (a chave por executavel); `git show HEAD:crates/victory-shader/src/emit/mod.rs` (funcao `translation_fingerprint`, que inclui `env!("VICTORY_SHADER_SWITCHES")`, `host_max_wave`, `view_heap_limit`, `loop_limit`, `group_sync`, `shader_opt`); `git show HEAD:crates/victory-shader/build.rs`.
- **Estado atual**: corrigido, e a forma atual e **auto-mantida** por construcao. **Risco residual estreito**: um switch lido **fora** de `crates/victory-shader/src` que ainda altere a emissao nao entra na lista automatica - o caso conhecido (limite de view heap, lido em `victory-gpu`) e coberto explicitamente por `view_heap_limit()` no fingerprint. Nao auditei os 219 switches um a um para procurar outro caso - **NAO VERIFICADO**.
- **Confianca**: **CONFIRMADA** para o caso dos quatro switches de wave e para a forma atual da chave.

### D-03 - Janelas de indirect draw serviam registros de lixo (2^24 vertices) e derrubavam a placa

- **Descricao / sintomas**: com `VICTORY_INDIRECT_WINDOW=1` a placa e perdida 2 a 3 vezes; uma vez com page fault na GPU apos 160 s. `9376fd6`: "pull request #220 ... lost the device twice here and is not in"; `f85101b`: "with windows on it lost the device three times here".
- **Causa raiz**: a janela copia para um scratch um **conjunto** de registros que o titulo desenha em sequencia; o checker do proprio PR (`VICTORY_INDIRECT_WINDOW_CHECK=1`) mostrou "one list drawing 90 records of 2^24 vertices from a window, where the per-draw path lets eight through". O caminho por draw aplicava clamp por registro e o orcamento por lista; a janela cobrava o orcamento na hora do clamp, entao registros da cauda nao desenhados (lixo, slots de `0xFFFFFFFF`) passavam.
- **Commit que introduziu**: o codigo do PR #220 entrou em `f85101b` (o PR nunca esteve na arvore antes: `9376fd6` registra "Tried and NOT in this tree"). O clamp por registro e a janela vem de `35b054e` (upstream) - a **regressao** e a combinacao janela + orcamento.
- **Tentativas de correcao**: (a) `VICTORY_INDIRECT_WINDOW_CHECK=1` como checker de diagnostico; (b) regra "um registro de 2^23 vertices por instancia ou mais roda vazio e nao cobra orcamento" (`crate::indirect::bound_ahead`), em `f85101b`; (c) `465b94f` agrupa registros indirect espalhados num clamp por lista (`VICTORY_INDIRECT_GATHER=1`).
- **Correcao definitiva**: `f85101b` (regra de 2^23 + checker), com resultado declarado "five runs with windows on, no device loss, the card's time per frame roughly halved". Em HEAD a funcao `windows()` tambem consulta `crate::drvwatch::tripped()` **fora** do `OnceLock`, entao o primeiro erro de driver desliga as janelas de fato (verificado em `crates/victory-gpu/src/draw.rs:188-191`).
- **Arquivos**: `crates/victory-gpu/src/indirect.rs`, `crates/victory-gpu/src/draw.rs`.
- **Evidencias**: `git show f85101b -- README.md`; `git show HEAD:crates/victory-gpu/src/draw.rs` (comentario "in a window a record that asks for more than can run is left empty and charges nothing").
- **Estado atual**: janelas seguem **off** por default (`VICTORY_INDIRECT_WINDOW`), usadas no conjunto de melhor desempenho. Ainda aparecem perdas de device sem causa conhecida (D-16).
- **Confianca**: **PROVAVEL** (o mecanismo e o fix estao no diff; a contagem de perdas e relato do autor).

### D-04 - Reversao e reaplicacao da mudanca de lane-mask (#11) em um minuto

- **Descricao / sintomas**: o commit `9e49b8a` remove a mudanca #11 ("Its programs translate and run, but on the test machine the lighting gain could not be shown, it costs about 40 ms of GPU time per frame, and the picture is being re-checked one change at a time from a known-good tree"). O commit imediatamente seguinte, `2119b0e`, a devolve: "The previous commit took it out; that was not wanted. The tree is again as it was before that commit."
- **Causa raiz**: disciplina de integracao - uma mudanca de contribuidor externo (`90228a7`, PR #11) foi mergeada, questionada por custo/beneficio nao comprovado e revertida por reflexo; a decisao de manter veio depois. Nao e um bug de codigo, e um **defeito de processo** com custo real: a arvore "known-good" citada no plano de verificacao de imagem muda de definicao duas vezes em 60 segundos.
- **Commit que introduziu**: `9e49b8a` (remocao); `2119b0e` (restauracao).
- **Tentativas de correcao**: `2119b0e` restaura o estado; `90228a7` (autor-date 25/11 04:35, commit-date 24/11 15:35 - cherry-pick) e a versao "definitiva" da mudanca, com look-ahead na restauracao do par e regra para `v_movrels_b32`.
- **Correcao definitiva**: `2119b0e` + `90228a7`.
- **Arquivos**: `crates/victory-shader/src/ir.rs`, `apps/victory/src/translate.rs`.
- **Evidencias**: `git log --format='%h|a:%ad|c:%cd|%s'` mostra a ordem e as datas divergentes; `git show 9e49b8a --stat` e `git show 2119b0e --stat` sao espelhos (40 linhas em `ir.rs`, 19 em `translate.rs`).
- **Estado atual**: mudanca presente; ganho de iluminacao continua **nao comprovado**, custo de ~40 ms/frame registrado na mensagem.
- **Confianca**: **CONFIRMADA** (fato historico); o custo de 40 ms e **NAO VERIFICADO** por mim.

### D-05 - Regra de staleness por sobreposicao total anulava slots de texturas vizinhas (paredes ausentes)

- **Descricao / sintomas**: a primeira sala aparece sem paredes em parte dos lancamentos, e slots de textura ficam nulos as dezenas de milhares por execucao. `310c6eb`: "each view written took the textures beside it out of the upload queue and made their host slots null (tens of thousands of slots a run)".
- **Causa raiz**: o titulo coloca **apenas os niveis pequenos** de uma textura na memoria e empacota a textura seguinte onde estariam os niveis grandes. Logo, os spans se **sobrepoem**; a regra antiga (`Rule::Overlap`) considerava que uma view nova tornava obsoleta qualquer view cujo span, em tamanho cheio, se sobrepusesse - inclusive vizinhas legitimas de um pacote (run 188: tres texturas 512x256 de `0x2c000` bytes, inicios a `0x2000` e `0x4000`, todas lidas no mesmo frame).
- **Commit que introduziu**: a estrutura `Spans` com spans "que nunca se sobrepoem" vem do squash `578d459` (`crates/victory-gpu/src/descriptor/host.rs`, comentario removido em `310c6eb`: "Spans never overlap: ...").
- **Tentativas de correcao**: nenhuma antes de `310c6eb`; o autor usou listagens de slot ("both proven with a slot listing before they were made") e um trace de paredes (`VICTORY_WALLS_TRACE`).
- **Correcao definitiva**: `310c6eb` - enum `Rule { Start, Overlap, Never }`, default `Start`: uma view nova so invalida a antiga quando comeca no mesmo endereco ou **dentro dos primeiros 4 KiB** (`const HEAD: u64 = 4096`), justificado por medicao ("Of 129 pairs of overlapping neighbours in the listings of runs 188 and 189, none starts closer than 0x2000"). Os modos antigos ficam como `VICTORY_STALE_TEXTURES=overlap` e `=0` para A/B.
- **Arquivos**: `crates/victory-gpu/src/descriptor/host.rs`, `crates/victory-gpu/src/draw.rs`, `crates/victory-gpu/src/resource.rs`, `crates/victory-gpu/src/host.rs`.
- **Evidencias**: `git show 310c6eb -- crates/victory-gpu/src/descriptor/host.rs` (diff completo do enum e do `HEAD`); em HEAD `Spans` tem `rule: Rule` e o campo `by_start` documenta "Spans may overlap".
- **Estado atual**: corrigido; `VICTORY_STALE_TEXTURES` default = `Start`. As paredes aparecem na primeira sala, mas "The room's very first shot still comes up without walls in some runs" (`8dd30ab`) - residuo aberto, ver G.
- **Confianca**: **CONFIRMADA** (a mudanca de regra e o racional estao no diff; a causalidade "paredes ausentes -> regra" e relato do autor com contagem de runs).

### D-06 - Textura escolhida por indice era enviada antes de o titulo preencher a memoria (rosto preto, cabelo sem cor)

- **Descricao / sintomas**: rosto do personagem principal sai preto e o cabelo palido em ~3 de 4 lancamentos; quais texturas perdem a corrida muda a cada lancamento.
- **Causa raiz**: uma textura escolhida por indice era enviada **uma vez**, quando o titulo escrevia a view; o titulo costuma preencher a memoria **depois** (as leituras de streaming chegam tarde), entao a copia host guardava zeros pelo resto da execucao. Nao havia segunda olhada.
- **Commit que introduziu**: comportamento herdado (fila de texturas do squash `578d459`); sintoma documentado em `a803c87`.
- **Tentativas de correcao**: (a) `310c6eb` conserta a invalidez entre vizinhas, que tambem retirava texturas da fila; (b) `a803c87` adiciona reexame.
- **Correcao definitiva**: `a803c87` - depois de enviar inteira, a textura e olhada de novo nos frames **4, 12, 40, 120 e 400** (fingerprint de 48 KiB da memoria); se mudou, reenvia e reinicia a serie. `VICTORY_HEAP_RECHECK=0` desliga. Um lancamento reenviou mais de 6.000 texturas assim.
- **Arquivos**: `crates/victory-gpu/src/descriptor/host.rs` (campo `checked: (u64,u64,u64,u8,u64)` em `Waiting`, `TextureQueue::recheck`, `HostHeap::texture_slot_recheck`), `crates/victory-gpu/src/draw.rs`, `crates/victory-gpu/src/texture.rs`.
- **Evidencias**: `git show a803c87 -- crates/victory-gpu/src/descriptor/host.rs`; mensagem: "Known gap: the fingerprint samples the start, middle and end of the memory, so a texture whose other parts arrive later is still missed."
- **Estado atual**: melhora grande, **correcao parcial**. O gap e declarado pelo autor e nao foi fechado: uma textura cujas partes chegam fora das tres amostras continua errada. `HEAP_RECHECK` permanece ligado por default (nao ha `=0` no conjunto de runs).
- **Confianca**: **CONFIRMADA** para o mecanismo; **PROVAVEL** para a taxa "3 de 4 lancamentos".

### D-07 - Registros de descritor copiados como bytes nao eram espelhados (40.000 por frame com slot nulo/velho)

- **Descricao / sintomas**: draws usam slots que nunca foram criados para eles; objetos somem ou desenham com descritor errado. `a26859f`: "about 40,000 records a frame that no call had announced; their host slots were empty or stale", contra "a few hundred views made by calls".
- **Causa raiz**: o titulo copia registros de descritor para o heap shader-visible **como bytes crus**, sem chamada de API. A camada so espelhava `Create*View`; copia crua nao gera evento. Sem write-watch no heap colocado, nada revela quais slots mudaram.
- **Commit que introduziu**: ausencia herdada (nao ha commit que a introduza; e uma lacuna do modelo de espelhamento do squash `578d459`).
- **Tentativas de correcao**: `a26859f` adiciona `sync_heaps()` (write-watch no heap, leitura da pagina pelo kernel, fingerprint por slot); **isto cria uma regressao de desempenho** de 70-220 ms/frame (ver H-05).
- **Correcao definitiva**: `9376fd6` - fingerprint multiplica-e-dobra no lugar de BLAKE3 (35-42 ms/frame em 250.000-300.000 registros) e slots lidos por tabela sao deixados para seus draws (`VICTORY_HEAP_SYNC_TABLES=1` espelha tambem): o passe cai de 70-220 ms para ~3 ms.
- **Arquivos**: `crates/victory-gpu/src/descriptor.rs`, `crates/victory-gpu/src/descriptor/host.rs`, `crates/victory-gpu/src/watch.rs`, `crates/victory-gpu/src/draw.rs`.
- **Evidencias**: `git show a26859f -- crates/victory-gpu/src/descriptor.rs` (doc de `sync_heaps` com a comparacao com upstream `86a5ed6`); `git show 9376fd6 -- crates/victory-gpu/src/descriptor/host.rs`.
- **Estado atual**: corrigido e ligado por default (`VICTORY_HEAP_SYNC=0` desliga). Resta a pergunta de corretude que o proprio codigo levanta: slots "deixados para os draws" so sao seguros se o draw le a tabela no momento certo - o que o autor afirma ter medido (run 200 e a sessao seguinte), mas nao ha teste no repositorio que cubra o caso.
- **Confianca**: **CONFIRMADA** (codigo e numeros no diff); a corretude da dispensa por tabela e **PROVAVEL**.

### D-08 - Indice de view alem do fim do heap: hang na NVIDIA, remocao do device na AMD

- **Descricao / sintomas**: na NVIDIA, um indice de view lido alem do fim do heap **travava** a GPU; na AMD RDNA3 (RX 7900 XT) o mesmo tipo de leitura **removeu o device** ao entrar no mundo.
- **Causa raiz**: o titulo le slots de view que nao existem (lixo de descritor). Sem clamp, o indice e indefinido em qualquer host. O caso AMD e concreto: "a compute program read view slot 16,777,088 (0xffff80) of a heap of 1,507,328, past its end".
- **Commit que introduziu**: o clamp existente antes (`PORTABLE_VIEWS`, aplicado so quando `CreateDescriptorHeap` falhava - caminho NVIDIA) vem do upstream `3827767` e do squash; o furo e o host que **nao** limita heaps.
- **Tentativas de correcao**: (a) clamp condicional antigo, com `set_view_heap_limit(PORTABLE_VIEWS - 1)` chamado antes de traduzir, para 25 pipelines feitos antes do heap do titulo; (b) `31714de` generaliza.
- **Correcao definitiva**: `31714de` - todo heap de views termina em um **slot nulo**: se o host aceita `UNCAPPED_VIEWS = 1 << 21`, o heap tem esse tamanho e o limite e `views - 1`; senao `PORTABLE_VIEWS`; o limite entra na chave do cache de shader (um shader mantem o limite com que foi traduzido) e `v_permlane16/x16` sao lowered.
- **Arquivos**: `crates/victory-gpu/src/descriptor/host.rs`, `crates/victory-gpu/src/shader_cache.rs`, `crates/victory-shader/src/emit/lower.rs`, `crates/victory-shader/src/emit/mod.rs`, `valu_tests.rs`.
- **Evidencias**: `git show 31714de -- crates/victory-gpu/src/descriptor/host.rs` (diff de `probe_view_limit` e `UNCAPPED_VIEWS`).
- **Estado atual**: corrigido; `probe_view_limit()` loga o tamanho escolhido e clampa "from the first shader on".
- **Confianca**: **CONFIRMADA** (mecanismo e numeros no diff e nos comentarios).

### D-09 - Copia in-place tomava as sondagens de 1 byte da camada (1.000-1.300 faults por frame)

- **Descricao / sintomas**: a primeira forma de `VICTORY_COPY_PLAIN` "won nothing": 1.000 a 1.300 faults por frame, quase todos de leituras de 1 byte que a propria camada faz para perguntar "esta pagina esta ai?". Um fault com handler custa dezenas de microssegundos; a leitura pelo kernel que falha custa ~1 us.
- **Causa raiz**: o handler tratava **toda** leitura como candidata a copia in-place, inclusive as sondagens deliberadamente destinadas a falhar (primeira pagina de textura, leitura de registro). O ganho da copia era consumido pelo custo dos faults.
- **Commit que introduziu**: `a253706` (cria `plaincopy.rs`).
- **Tentativas de correcao**: nenhuma intermediaria; `0b0074e` corrige e mede: tempo de kernel da thread de submit cai de 44 para 20 ms/frame, imagem igual, 4-9 faults/frame.
- **Correcao definitiva**: `0b0074e` - duas regras em `plaincopy.rs`: `const LEAST: usize = 256` (copias menores ficam com o kernel) e `RECENT[1024]` com validade de **2 segundos** por bloco de 64 KiB que falhou (`faulted_lately`), voltando direto ao kernel enquanto a nota viver.
- **Arquivos**: `crates/victory-gpu/src/plaincopy.rs`, `crates/victory-gpu/src/host.rs`, `crates/victory-gpu/src/framelog.rs`.
- **Evidencias**: `git show 0b0074e -- crates/victory-gpu/src/plaincopy.rs`; `git show HEAD:crates/victory-gpu/src/plaincopy.rs` linhas 80-110 (comentarios de `LEAST` e `RECENT`).
- **Estado atual**: corrigido, mas **off por default** e usado no conjunto de melhor desempenho. O `RECENT` e um bloom filter de 1024 posicoes indexado pelos bits baixos do bloco: colisoes produzem "kernel mesmo quando nao precisa", nunca incorrecao - decisao consciente, barata e correta.
- **Confianca**: **CONFIRMADA**.

### D-10 - Copia de textura sobre ela mesma (auto-copia proibida pelo host)

- **Descricao / sintomas**: 402 copias de uma textura do titulo para outra que compartilham o **mesmo** recurso host em uma execucao de cinco minutos. O host nao permite copiar uma textura sobre ela mesma; a operacao e invalida (comportamento indefinido/erro de debug layer).
- **Causa raiz**: duas texturas distintas do titulo podem ser alias do mesmo recurso host (views da mesma memoria). A camada registrava a copia por identidade das texturas do titulo, nao do recurso hospedeiro.
- **Commit que introduziu**: herdado (caminho de `CopyTextureRegion` do squash).
- **Tentativas de correcao**: `a253706`: "A copy between two title textures that share one host texture is left out".
- **Correcao definitiva**: `a253706`.
- **Arquivos**: `crates/victory-gpu/src/host.rs`, `crates/victory-gpu/src/list.rs`.
- **Evidencias**: `git show a253706 -- README.md` ("a forbidden self-copy removed"); `git log -S` sobre o texto nao localiza teste dedicado - **NAO VERIFICADO** se ha teste de regressao.
- **Estado atual**: corrigido. A ausencia de teste de regressao e um risco: o caminho de copia foi reescrito varias vezes depois (`queue2`, `window upload`).
- **Confianca**: **PROVAVEL** (o conserto e afirmado e a classe do problema e coerente; nao ha diff de teste).

### D-11 - Texturas escritas por compute nao chegavam a CPU do titulo (o barco afundava)

- **Descricao / sintomas**: as passes de oceano escrevem um height map 256x256 `R32_FLOAT` e um displacement map `R32G32_FLOAT`; o codigo CPU do titulo **amostra** esses dois (cerca de 30 enderecos de codigo, achados com `VICTORY_READ_WATCH=1`) - e o que faz o barco flutuar. A CPU lia memoria que ninguem escreveu, e o barco afundava: a primeira missao falhava de imediato (run 275).
- **Causa raiz**: no console CPU e GPU compartilham memoria; no PC o que o compute escreve fica **na copia host** da textura e nunca volta para a memoria do titulo. O fluxo de copy-back existente tratava buffers e read-backs de render target, nao texturas escritas por shader e lidas pela CPU.
- **Commit que introduziu**: lacuna estrutural herdada; sintoma materializado quando o jogo chegou ao oceano.
- **Tentativas de correcao**: `895c0b8` (cherry-pick de Geckomacabre, portando patch de terceiro) cria `texback.rs`: lista que escreve textura de um tipo listado copia para read-back buffer ao fechar, e apos rodar a lista os texels sao **tileados** na memoria do titulo no layout do console (`victory_formats::Layout::tile`). `04df435` (Codex) rastreia alocacao e fence por readback de agua.
- **Correcao definitiva**: `f5c03f3` (merge do PR #999) com `VICTORY_TEXTURE_BACK=1`, **off por default**. Run 276, com caches de shader limpos: 22.900 read-backs em seis minutos, o testador passou do ponto do barco - "the largest step of the day".
- **Arquivos**: `crates/victory-gpu/src/texback.rs` (150 linhas), `crates/victory-gpu/src/resource.rs`, `crates/victory-gpu/src/host.rs`, `crates/victory-gpu/src/draw.rs`, `crates/victory-gpu/src/lib.rs`.
- **Evidencias**: `git show 895c0b8`; `git show 04df435 --stat`; `git show f5c03f3 --stat`; README de HEAD ("Without it a boat sank here and its mission failed at once (run 275)").
- **Estado atual**: funcional, off por default. **O mesmo run 276 terminou com a placa perdida** (dois erros de driver e "device removed", sem nada da camada rodando na placa naquele momento) - causa desconhecida, D-16.
- **Confianca**: **CONFIRMADA** para o mecanismo e para o resultado funcional (relato do autor com numero de read-backs); a ligacao com a perda de device e **NAO VERIFICADA** (o proprio autor nao a estabelece).

### D-12 - Loads escalares agrupados de 4 dwords leem alem do fim da view (agua preta)

- **Descricao / sintomas**: com `VICTORY_SHADER_OPT=127` (bit 64 ligado), a agua sai **preta** no run 277. Com o bit fora do default (63), 25.635 de 25.635 shaders sao byte-identicos ao build anterior.
- **Causa raiz (hipotese do autor, nao provada na placa)**: uma chamada para cada quatro dwords, num byte 328 de uma view de 340 bytes, **pede um dword depois do fim**. A forma default nunca le esse dword porque a chamada e codigo morto; a forma agrupada le. Cerca de 90 programas de iluminacao tem esse load. O adaptador de software le os tres dentro e devolve 0 para o de fora; a placa nao foi testada.
- **Commit que introduziu**: `c36971b` - `raw_dwords()` em `lower.rs` faz o load com `mask = (1u8 << n) - 1` mas o `rawBufferLoad` acessa o bloco alinhado de 4 dwords. O proprio comentario admite o limite: "Loads of different console instructions are never joined".
- **Tentativas de correcao**: nenhuma de codigo; `44bff61`/`d827419` isolam a causa por experimento (run 279 limpa o `VICTORY_CARD_HASH_LEN` como suspeito e deixa os loads agrupados como causa).
- **Correcao definitiva**: **SEM CORRECAO**. O bit 64 continua fora do default 63.
- **Arquivos**: `crates/victory-shader/src/emit/lower.rs` (`wide_loads`, `raw_dwords`), `crates/victory-shader/src/emit/bitcode/opt.rs`.
- **Evidencias**: `git show c36971b -- crates/victory-shader/src/emit/lower.rs`; `git show HEAD:crates/victory-shader/src/emit/lower.rs` linhas 5457-5503; README de HEAD ("The grouped scalar loads blackened the water in run 277 ... and stay off").
- **Estado atual**: desligado; a otimizacao existe e esta testada no adaptador de software, com um risco de memoria nao resolvido. E o unico caso do repositorio em que uma otimizacao **funcional** e barrada por um defeito de corretude nao compreendido.
- **Confianca**: **HIPOTESE** (a causa e explicitamente "the lead, not proven on the card"); o sintoma e **CONFIRMADO**.

### D-13 - O "dip" periodico de frame era o proprio harness de captura (235 ms por frame salvo)

- **Descricao / sintomas**: leituras de 5 s mostravam frames de 300-500 ms entre 150-215 ms, sem causa no log; em algumas delas uploads de textura 3-5x o normal, em outras nada. O autor chegou a suspeitar de compilacao de shader (descartado: "4-9 first uses and no waiting").
- **Causa raiz**: `VICTORY_CAPTURE_WORLD` salvava cada 8o frame do mundo: leitura de volta e escrita de PNG **dentro do present**, 235 ms cada.
- **Commit que introduziu**: o harness de captura veio no squash `578d459` (usado para verificar imagem); o sintoma e o custo nao foram instrumentados ate `a253706`.
- **Tentativas de correcao**: `a253706` cria `framelog.rs` (todo frame e sua composicao, sem reset de contadores) e `cardmeter.rs` (tempo da placa por tipo de lista); e com esses instrumentos que o custo aparece.
- **Correcao definitiva**: `a253706` - `VICTORY_CAPTURE_WORLD_EVERY` (64 para runs de velocidade) e a nota de honestidade no README. Nao ha mudanca no caminho de captura: o defeito era de **metodologia de medicao**, nao de codigo.
- **Arquivos**: `crates/victory-gpu/src/framelog.rs` (novo, 584 linhas), `crates/victory-gpu/src/cardmeter.rs` (novo, 242), `crates/victory-gpu/src/capture.rs`, `README.md`.
- **Evidencias**: `git show a253706 -- README.md` ("The regular dips of earlier test runs were the test harness...").
- **Estado atual**: resolvido como instrumentacao. **Licao**: durante semanas de trabalho o "problema de frame rate" mais visivel era auto-infligido pelo instrumento - e o mesmo padrao reaparece em D-14.
- **Confianca**: **CONFIRMADA** para o custo de 235 ms e a opcao `EVERY`; a atribuicao de todos os dips a captura e **PROVAVEL** (o autor diz "the regular dips").

### D-14 - Alegacao de ganho de 6,0 -> 8,9 fps comparava cenas diferentes

- **Descricao / sintomas**: `a253706` publica "draw windows measured at 6.0 -> 8.9 frames a second"; `0b0074e` retrata: as duas execucoes estavam em cenas diferentes (cerca de 4.300 contra 1.550 draws por frame) e os numeros sao **withdrawn**.
- **Causa raiz**: comparacao entre execucoes sem controlar a carga (draws por frame), num titulo cujo tempo de frame e **linear em draws** (~32 us por draw all-in, medido em `a253706`).
- **Commit que introduziu**: `a253706` (a publicacao do numero).
- **Tentativas de correcao**: `0b0074e` substitui a comparacao por uma **reta ajustada** de tempo de frame contra draws por frame: windows off 145 ms a 4.500 draws e 271 ms a 10.000; windows on com `COPY_PLAIN=1` 124 ms e 211 ms.
- **Correcao definitiva**: `0b0074e` (retratacao + metodo).
- **Arquivos**: `README.md`, `crates/victory-gpu/src/framelog.rs`.
- **Evidencias**: `git show 0b0074e -- README.md` ("A CORRECTION: the first figures given here ... compared two runs that stood in different scenes ... and are withdrawn").
- **Estado atual**: corrigido no README; o metodo (reta por draws) e reutilizavel.
- **Confianca**: **CONFIRMADA**.

### D-15 - Page mapping do titulo: duas tentativas que quebraram o streaming de textura e foram revertidas

- **Descricao / sintomas**: no streaming, metade do tempo vai para frames de ~490 ms com a placa **ociosa** por ~330 ms. Dentro deles, as copias de page mapping do titulo (`CopyPageMappingsBatchX` -> `xmem::map_alias`) custam **32 ms de cada frame de cutscene** e 246 ms de um frame de streaming: ~850 remaps por frame, cada um uma view removida e mapeada no kernel.
- **Causa raiz**: cada remap e uma chamada de kernel sobre o espaco de enderecos; o titulo mapeia uma pagina por vez para o heap tileado.
- **Commit que introduziu**: comportamento do titulo, nao do fork; o custo nunca foi medido antes de `d510ee4`.
- **Tentativas de correcao**: (a) "deixar em paz paginas que ja mostram o que se pede" - contagem mostrou que **nao existem** tais paginas ("every remap is a real change"); (b) mapear paginas consecutivas do pool como **uma** view - 1,8 a 4,2 paginas por view, tempo de remap 140 -> 79 ms/frame, **mas** faces saem pretas ou vermelhas e uma execucao desenhou lixo em **3 de 3** lancamentos, contra nenhum em 6 sem a mudanca.
- **Correcao definitiva**: **SEM CORRECAO**. `d510ee4` e README-only ("no code change in this commit"): ambas as tentativas foram tiradas.
- **Arquivos**: `crates/victory-hle/src/modules/xmem.rs`, `README.md`.
- **Evidencias**: `git show d510ee4 -- README.md`; `git show d510ee4 --stat` confirma "README.md | 18 ++++++++++++++++++" (somente README).
- **Estado atual**: aberto; "why a longer view breaks texture streaming is not known yet".
- **Confianca**: **CONFIRMADA** para as medicoes e a reversao; a causa do preto/vermelho e **NAO VERIFICADA**.

### D-16 - Perdas de device sem causa identificada (runs 204-279 e mundo aberto)

- **Descricao / sintomas**: sequencia de perdas ao longo do dia 25/11: run 201, 204, 208, 210, 224 (com entradas `nvlddmkm` 153 de 2,7 a 6,3 s antes), run 263 (RenderDoc), run 276 (6 min, dois eventos de driver e hang), run 277 (49 min, "device removed" 8 ms depois de um sweep liberar 150 copias de textura), e uma primeira sessao no mundo aberto (9,2 de 10,3 GB de orcamento, 2.073 passes sem textura, 44.613 texturas refeitas).
- **Causa raiz**: **desconhecida**. Ha tres hipoteses registradas pelo autor e nenhuma fechada: (a) caminhos sob suspeita das janelas de indirect (mitigado por D-03 + driver watch); (b) esgotamento de memoria de textura da placa (9,2/10,3 GB) - "cause of that removal not known either"; (c) o copy-back de textura do PR #999 (run 276 perdeu a placa **com** o switch; run 275 sem ele jogou 20 minutos sem erro - n=1 de cada lado).
- **Commit que introduziu**: nao atribuivel. Cada perda tem um conjunto de switches diferente.
- **Tentativas de correcao**: `a253706` (driver watch: assina o event log, loga, conta e **desliga as janelas** no primeiro erro - a funcao `windows()` consulta `drvwatch::tripped()` a cada chamada); `8dd30ab`/`a26859f` (levels on demand, espaco reservado, refresh quando a placa esta cheia); `79cfdd9` (biblioteca de pipeline recusada guardada sob outro nome, nunca sobrescrita); `44bff61` (DRED + `VICTORY_GPU_TIMING` para quem quiser numeros reais).
- **Correcao definitiva**: **SEM CORRECAO**.
- **Arquivos**: `crates/victory-gpu/src/drvwatch.rs`, `crates/victory-gpu/src/tiles.rs`, `crates/victory-gpu/src/pipeline/library.rs`, `crates/victory-gpu/src/texture.rs`.
- **Evidencias**: `git show HEAD:crates/victory-gpu/src/drvwatch.rs` (doc com os runs 201/204/208/224/210); README de HEAD, secao "The card was lost twice"; `git show 0b0074e -- README.md`.
- **Estado atual**: aberto; e o maior risco de estabilidade do fork. O driver watch reduz o custo de uma perda (desliga o suspeito) mas nao previne a primeira.
- **Confianca**: **CONFIRMADA** para a existencia e os numeros; **NAO VERIFICADA** para qualquer causa.

### D-17 - Divida de verificacao: "this merged tree was built; its test suites were not run"

- **Descricao / sintomas**: um padrao textual recorrente nos commits de integracao do fork: a arvore mergeada e compilada, mas as suites nao rodam nela; os testes citados passaram na arvore de desenvolvimento de onde as mudancas vieram.
- **Causa raiz**: fluxo de trabalho de uma pessoa com um PC e um GPU, integrando branches de terceiros por cherry-pick/squash em vez de merge com CI obrigatorio. O CI existe desde `8e79c80` (`.github/workflows/ci.yml`: clippy `-D warnings` nos dois feature sets + `cargo test --workspace`), mas os commits do fork **nao passam por PR** - sao pushes diretos em `main`.
- **Commit que introduziu**: nao e um commit; e o metodo. Exemplos literais: `8dd30ab` ("its test suites were run on the tree the changes come from (gpu 129, hle 97)"), `a26859f` ("This merged tree was built; its test suites were not run"), `9376fd6` ("its test suites were not run"), `f85101b` ("its whole test suite was not run"), `a253706` ("the whole suite was not"), `0b0074e` ("the unit tests of the changed parts were run on the development tree"), `c36971b` ("its unit tests were not run after the merge").
- **Tentativas de correcao**: nenhuma estrutural; ha correcoes pontuais de teste (`004bf69`, `7b0f736`, `3930989`) e a propria CI upstream.
- **Correcao definitiva**: **SEM CORRECAO**.
- **Arquivos**: `.github/workflows/ci.yml`, todos os crates.
- **Evidencias**: `git show HEAD:.github/workflows/ci.yml`; as mensagens acima (`git log --format='%B'`).
- **Estado atual**: aberto. Consequencia pratica para quem forkal: os numeros de desempenho sao confiaveis (medidos em jogo), os de **corretude** sao menos - varias mudancas nunca rodaram a suite completa na arvore em que estao.
- **Confianca**: **CONFIRMADA** (as frases estao nos commits).

---

## E. Relacao de causa e efeito

**E-01 - Atualizacao de shader derruba a placa e contamina os experimentos seguintes.**
`6a2d9f9` liga a forma wave64 wide por default e melhora a traducao -> 4/4 execucoes perdem o device (`004bf69`) -> `e9765a7` inverte o default e poe os switches de wave na chave do cache -> estabilidade volta. Efeito colateral descoberto **no mesmo commit**: execucoes anteriores com `VICTORY_WAVE_WIDE=0` usavam traducoes cacheadas da forma wide, o que significa que parte da comparacao "wide vs. thread-a-piece" feita antes de `e9765a7` pode ter comparado wide com wide.

**E-02 - Uma regra de invalidez de textura apaga as vizinhas.**
`578d459` assume "spans never overlap" -> o titulo empacota texturas em spans sobrepostos -> `310c6eb` mostra 129 pares de vizinhas sobrepostas em runs 188/189, dezenas de milhares de slots nulos por run, paredes ausentes -> regra nova (`Start` + 4 KiB) -> paredes aparecem em 4/4 runs nas frames seguintes, mas o **primeiro** shot da sala ainda falha as vezes -> residuo em `8dd30ab` e em G.

**E-03 - Corretude de descritor custa desempenho, e o desempenho exige outra representacao.**
`a26859f` espelha registros copiados como bytes (corretude: 40.000 slots/frame vazios/velhos) -> o passe custa 70-220 ms/frame (regressao H-05) -> `9376fd6` troca BLAKE3 por fingerprint multiplica-e-dobra e deixa slots de tabela para seus draws -> ~3 ms -> `6d20eed` adiciona a lista de copias sujas (`VICTORY_CARD_CHANGED_LIST`) porque o mesmo problema reaparece no caminho de copia na placa com ~3.900 copias e 10-18 submissoes por frame.

**E-04 - Otimizacao de copia em disco/CPU e consumida pelas sondagens.**
`a253706` cria copia in-place -> 1.000-1.300 faults/frame, quase todos de leituras de 1 byte feitas para **perguntar** se a pagina existe -> nenhum ganho liquido -> `0b0074e` separa as duas coisas (`LEAST=256`, quarentena de 2 s) -> kernel da thread de submit 44 -> 20 ms/frame, imagem igual, 4-9 faults/frame.

**E-05 - Janela de indirect + orcamento por lista = lixo executavel.**
`35b054e` (upstream) introduz clamp por registro e orcamento por lista -> PR #220 adiciona janelas que copiam um conjunto de registros e cobram orcamento no clamp -> registros da cauda (nao desenhados, lixo `0xFFFFFFFF`) passam -> 90 registros de 2^24 vertices -> perda de device -> `f85101b` faz registro de 2^23+ rodar vazio e nao cobrar orcamento, e `drvwatch` desliga janelas no primeiro erro de driver -> 5 runs sem perda.

**E-06 - O copia-e-volta que ninguem le custa 497 GB.**
O modelo "tudo que o shader escreveu volta para a memoria do titulo" (`shadow.rs`) funciona, mas e caro: `VICTORY_BACK_READS=alt` mede, em 49 minutos de jogo, **497 GB** copiados de volta; de 438.000 stores amostrados, 60% nao foram tocados por ninguem antes do store seguinte, 36% so pelas atualizacoes da propria camada, 3,8% escritos pelo titulo, **0,19%** tocados por qualquer outra coisa (o maximo que o codigo do titulo pode ter lido). Consequencia: `79cfdd9` e `c36971b` atacam o lado "nao copiar de volta o que ninguem le" (`backreads.rs` como passo de medicao; `VICTORY_TEXTURE_BACK` limitado a quatro formatos de agua).

**E-07 - Medir primeiro muda a conclusao duas vezes.**
`a253706` publica 6,0 -> 8,9 fps -> `0b0074e` descobre que as cenas eram diferentes (4.300 vs 1.550 draws) e retrata -> o metodo passa a ser reta de tempo de frame contra draws (145 ms a 4.500; 271 ms a 10.000) -> `c36971b` mede em "like draws" (114,9 -> 109,7 ms) e separa tempo de **placa** de tempo de **frame**. O padrao "ganho de placa nao e ganho de frame" aparece tres vezes (PR #1014: 35 -> 10 ms de placa mas frame 197 -> 163/183 ms; `CARD_BUFFERS_MIN_KB`: -18 ms de placa, -5 ms de frame; `WINDOW_UPLOAD`: -1,9 GB de memoria, 0 de fps).

**E-08 - A thread que prepara submissao vira o limite.**
`f85101b` tira o submit da thread de render (`ASYNC_SUBMIT`) e poe fetch em copy queue (`QUEUE2`) -> o gargalo migra para a thread que prepara submissao -> `0b0074e` poe essa thread nos cores rapidos (`VICTORY_FAST_CORES`, de 6% para 0 de 21.059 observacoes em core lento) -> `c36971b` mede a placa **ociosa 12-40 ms por frame**: "the thread that prepares submissions is the limit".

---

## F. Correcoes importantes

### F-01 - Switch de ambiente como unidade de experimento, com a chave do cache derivada dele

- **O que estava errado**: um interruptor que muda o codigo gerado nao entrava na identidade do artefato cacheado; o experimento "desligado" reutilizava o artefato "ligado". Isso e um erro de **corretude experimental**, nao de desempenho. Na forma antiga a chave era o BLAKE3 do **proprio executavel**: dois runs do mesmo binario com switches diferentes tinham a mesma chave.
- **Como foi corrigido**: `e9765a7` acrescenta manualmente os quatro switches de wave; `8dd30ab` troca o esquema por `translation_fingerprint()`, e `crates/victory-shader/build.rs` **descobre os switches lendo o proprio codigo**: varre `crates/victory-shader/src/**/*.rs`, acha todo literal que comece com `"VICTORY_`, e emite `cargo:rustc-env=VICTORY_SHADER_SWITCHES=<lista>`; em tempo de execucao os valores entram na chave. O mesmo fingerprint inclui fonte do tradutor (FNV-1a de todos os `.rs`), feature `diagnostics`, `debug_assertions`, `host_max_wave`, `view_heap_limit`, `loop_limit`, `group_sync` e `shader_opt`.
- **Por que funciona**: a chave e funcao pura de (fonte do tradutor, configuracao efetiva, fatos do host) e a lista de configuracao e **derivada do codigo**, nao mantida a mao - um switch novo dentro do crate entra na chave sem ninguem lembrar. Um switch que muda a emissao sem literal no crate (ou fora dele) e o unico furo restante.
- **Dependencias**: `victory-pipeline::Cache` (escrita atomica), `include_bytes!("shader_cache.rs")` (invalida quando o proprio arquivo de cache muda), `VICTORY_SHADER_CACHE_KEY=exe` como volta ao esquema antigo.
- **Reutilizavel?** Sim, e o padrao mais valioso do repositorio para qualquer sistema com cache + flags: **toda flag que altera a saida entra na chave, e a lista de flags e gerada a partir do codigo**. Custo: um `blake3` por lookup. Risco: confiar na descoberta textual (um switch lido por nome montado em tempo de execucao escapa).

### F-02 - Guarda sobre todas as views de uma pagina fisica

- **O que estava errado**: o titulo mostra a mesma pagina fisica em varios enderecos; a copia host era protegida em **um** endereco so, entao uma escrita por outra view nao gerava fault e a copia ficava velha - silenciosamente. Medido: 0,8-1,0 **milhao** de escritas por outra view em cinco minutos.
- **Como foi corrigido**: `8dd30ab` (codigo de myokex/victory-amd `c6dc4d8`, adaptando PR #115 de Spexivus, correcao de joroyt, parte de VIctory #1122 de tightfall): ao proteger um bloco, **todas** as views mapeadas daquela pagina ficam read-only; o handler devolve acesso a todas. `VICTORY_VIEW_GUARD=0` desliga.
- **Por que funciona**: a protecao e uma propriedade da **pagina fisica**, nao do endereco virtual; aplicar em todas as views transforma qualquer escrita em fault, de qualquer alias.
- **Dependencias**: o runtime precisa informar map/unmap (`title_memory_remapped`); ha limite `MANY_VIEWS` (paginas com views demais nao sao guardadas); `VICTORY_PAGE_WRITES` conta/abre os blocos sob escrita do storage.
- **Reutilizavel?** Sim, para qualquer emulacao de memoria compartilhada com alias (guest/host). Risco: custo de manter a lista de views e de reaplicar protecao; e um caminho quente, com 3 a 6 views por frame tipicamente.

### F-03 - Copia in-place sem pilha com handler que so aceita leitura

- **O que estava errado**: `ReadProcessMemory` no proprio processo custa o que custa uma syscall que trava e sonda cada pagina: 35 ms/frame para ~100 MB (3 GB/s) onde o `memcpy` faz o triplo.
- **Como foi corrigido**: `0b0074e`/`a253706` - `plaincopy.rs`: funcao em assembly (`victory_copy_guarded`, `rep movsb`, sem uso de pilha, preservando `rsi`/`rdi` em `r9`/`r10`), com dois pontos de retorno rotulados; um handler **primeiro da cadeia** olha cada access violation: se for **leitura** e o `RIP` for exatamente `victory_copy_guarded_at`, retoma em `victory_copy_guarded_fault` (retorna 0) e o chamador cai no kernel; escrita que falha **nao** e tomada.
- **Por que funciona**: o caso comum (pagina presente) e um `rep movsb` puro; o caso raro (pagina ausente) e uma excecao tratada no proprio handler, sem syscall. A restricao a leitura impede que o handler engula corrupcao de destino.
- **Dependencias**: x86-64; `LEAST=256` e `RECENT` (2 s) para nao capturar as sondagens; precisa de um caminho de fallback correto no kernel (`copy_mapped`), que tambem preenche zeros.
- **Reutilizavel?** Sim, com cuidado: `global_asm!` + `unsafe extern "C"` + handler vetorizado e especifico de plataforma. O par (limiar de tamanho, quarentena por bloco) e a parte transferivel e mais subestimada.

### F-04 - Substituir hash criptografico por fingerprint barato no caminho quente

- **O que estava errado**: o sync de descritores calculava BLAKE3 de cada registro para saber se mudou: 35-42 ms/frame em 250.000-300.000 registros, e o passe inteiro custava 70-220 ms/frame.
- **Como foi corrigido**: `9376fd6` - fingerprint de multiplicar-e-dobrar sobre o registro, no lugar; e, mais importante, **nao olhar** os slots que os draws leem por tabela (`VICTORY_HEAP_SYNC_TABLES=1` espelha tambem). Resultado: ~3 ms/frame.
- **Por que funciona**: o objetivo nao e resistencia a colisao criptografica, e detectar "mudou ou nao" em um registrador de descritor de 32-64 bytes; um fold multiplicativo separa bem entradas reais. A segunda metade (dispensar slots) elimina trabalho em vez de acelera-lo.
- **Dependencias**: o draw precisa reconstruir o descritor da tabela no momento do uso; `COPIED_TABLED` conta os dispensados.
- **Reutilizavel?** Sim, com a ressalva: colisao aqui significa **deixar de espelhar** um descritor mudado - um erro silencioso de imagem. O autor mediu em jogo, mas nao ha teste de colisao adversarial.

### F-05 - Hash de peca por grupo de 64 threads com dobra em ordem de lane

- **O que estava errado**: `CARD_COMPARE` dava **uma thread por peca de 64 KiB**, que percorria 4.096 loads de 16 bytes em serie. O tempo de uma pass era o de **uma** peca, independentemente de quantas pecas houvesse, e a placa nao rodava mais nada: "the card waits on one load at a time and runs next to nothing else".
- **Como foi corrigido**: `5557224` (PR #1014 de martindevbr) - `CARD_COMPARE_WIDE`: thread `t` do grupo 64 hasheia os passos `t, t+64, t+128...` (threads vizinhas leem bytes vizinhos, coalescido); a thread 0 dobra os 64 hashes de lane **em ordem de lane** e grava/compara. Uma peca por grupo => no maximo 16.385 grupos por dispatch (copias < 1 GiB).
- **Por que funciona**: transforma latencia serial em paralelismo massivo e mantem o hash **deterministico e igual entre store e compare**, porque a dobra e em ordem fixa.
- **Dependencias**: e **outro hash** - todo o processo tem de usar uma forma so (`card_hash_wide()` escolhido uma vez por `OnceLock`); `card_compare_groups` muda de `pieces.div_ceil(64)` para `pieces`; `VICTORY_CARD_HASH_LEN` muda o comprimento hasheado nas duas formas (`6d20eed`).
- **Reutilizavel?** Sim: e o padrao canonico de "hash paralelo com reducao ordenada" para comparar buffers grandes na GPU. **Risco medido e assumido**: `6d20eed` torna a forma **opt-in** porque "it has not run in a game on this tree yet, and it is another hash with another rare pattern of changes it cannot see".

### F-06 - Driver watch: transformar um sinal externo em fail-safe

- **O que estava errado**: a camada descobria a perda do device quando o device ja tinha ido. Havia, porem, um aviso previo **fora do processo**: entradas do driver de video no event log do Windows (`nvlddmkm`, evento 153), de 2,7 a 6,3 s antes, em todos os runs que perderam a placa; runs sem a entrada nao perderam nada.
- **Como foi corrigido**: `a253706` - `drvwatch.rs` assina o log (`EvtSubscribe` para eventos futuros), loga com relogio proprio, conta, e no primeiro evento marca `TRIPPED`; `draw::windows()` consulta `tripped()` **a cada chamada** (fora do `OnceLock`), entao as janelas caem para o caminho seguro pelo resto do run.
- **Por que funciona**: troca "perder a placa" por "andar mais devagar"; usa um canal de diagnostico do SO em vez de inferencia.
- **Dependencias**: provider do driver (NVIDIA observado); `VICTORY_DRIVER_WATCH=0` desliga; custo de abrir a assinatura no start.
- **Reutilizavel?** Muito. E um padrao de **canario externo** para qualquer camada que possa derrubar um device: assinar o log do driver e degradar preventivamente e mais barato que diagnosticar depois. Limitacao: e especifico de Windows/driver e nao cobre perda sem evento (run 277 teve "device removed" sem evento).

### F-07 - Levels on demand

- **O que estava errado**: a copia host de uma textura era uma textura **committed**: todos os niveis tomavam memoria da placa desde o primeiro upload. O titulo coloca a cadeia inteira no espaco de enderecos mas segura so alguns niveis (os menores, no inicio). Medido no run 189: 2,2-3,0 GB de 4,3-5,5 GB de copias host estavam em niveis cuja memoria do titulo **nao estava mapeada**, em ~1.600 copias; com 2,2-3,1 GB de render targets enchendo a placa, texturas que precisavam de copia nova esperavam por espaco (inclusive o rosto do personagem).
- **Como foi corrigido**: `8dd30ab`/`a26859f` com as correcoes do PR #747 de AscilCH (`tiles.rs`, 1.419 linhas): memoria da placa por nivel, niveis segurados recebem memoria so enquanto ha espaco e sao pedidos de novo (`VICTORY_LEVELS_ON_DEMAND=1`, **off**).
- **Por que funciona**: alinha a alocacao ao que o titulo realmente mantem, e nao ao tamanho declarado da cadeia.
- **Dependencias**: tabela de spans/`Levels min KB`; interage com `Rule::Start` (D-05), porque as duas coisas dependem do mesmo modelo de "o que esta mapeado".
- **Reutilizavel?** Sim, para qualquer titulo que use streaming por nivel (comum em consoles). **Estado**: rodou na placa pela primeira vez em `a26859f` (RTX 4070: 3,1 GB nao alocados, sem perda), mas continua off por default.

### F-08 - Warm-up de shaders com chave estavel por fonte do tradutor

- **O que estava errado**: cada build novo re-traduzia milhares de shaders do titulo, o que mantinha os primeiros minutos de todo run sem pipelines.
- **Como foi corrigido**: `8dd30ab` - `victory translate --warm <arquivo de hashes> --jobs N` traduz para o cache, no processador, tudo que o run vai precisar (4,5-6,5 min); a chave do cache passa a ser a fonte do tradutor, nao o executavel inteiro. O log de um run nomeia shaders por 16 digitos hex, que alimentam o arquivo de warm.
- **Por que funciona**: separa "o que ja foi traduzido" de "qual binario esta rodando"; um build que nao toca o tradutor comeca quente.
- **Dependencias**: `VICTORY_HOST_WAVE` e `VICTORY_VIEW_LIMIT` precisam ser iguais aos do run; `translation_fingerprint` precisa incluir tudo que muda a emissao (D-02).
- **Reutilizavel?** Sim, e um dos padroes mais transferiveis: **warm-up offline + chave por fonte** e uma pratica que qualquer camada de recompilacao deveria ter desde o inicio.

---

## G. Problemas ainda existentes

**G-01 - Sombra preta (ray tracing nao ilumina nada).** O README de HEAD mantem: "Light in shadow | missing: the game's ray-traced bounce light and ray-traced lamp and sun shadows do not run yet". `docs/fork-ray-tracing.md` descreve o estado: 59 de 64 programas de trace traduzem, o teste ray/box e ray/triangle e emulado em DXIL e conferido contra referencia na GPU, mas "Nothing is lit by it yet". **CONFIRMADA**.

**G-02 - O primeiro shot da primeira sala as vezes vem sem paredes.** Depois de `8dd30ab`: paredes nas frames seguintes em 4/4 runs (2/5 antes), mas "The room's very first shot still comes up without walls in some runs". Ha `VICTORY_WALLS_TRACE=1|2` para investigar. **CONFIRMADA** (relato do autor), sem causa.

**G-03 - Desempenho default nao medido.** O README admite: "The plain defaults were last measured at 420 to 450 ms a frame and not since." Ou seja: os 8-10 fps divulgados sao do **conjunto de switches**, nao do que o usuario recebe ao compilar. **CONFIRMADA** (declaracao no README).

**G-04 - Memoria e placa cheias.** ~24 GB de processo (16 GB em memoria), 1-5 GB livres em 32 GB, placa de 12 GB com 9,2 de 10,3 GB de orcamento, 2.073 passes sem textura por falta de espaco, 44.613 texturas liberadas e refeitas em 13 minutos. `VICTORY_LEVELS_ON_DEMAND` e a mitigacao, mas segue **off**. **CONFIRMADA**.

**G-05 - Perdas de device sem causa** (D-16). Duas no run 276 e 277, uma no mundo aberto, uma com `TEXTURE_BACK`. **CONFIRMADA** a ocorrencia, **NAO VERIFICADA** a causa.

**G-06 - Loads agrupados leem alem da view** (D-12). Desligados, causa nao provada, nenhum teste na placa. **HIPOTESE**.

**G-07 - Page mapping do titulo custa 32 ms de cada frame de cutscene e 246 ms de um frame de streaming**, e as duas tentativas de otimizar quebraram o streaming (D-15). "Why a longer view breaks texture streaming is not known yet." **CONFIRMADA**.

**G-08 - 497 GB de copy-back por 49 minutos, 60% sem leitor.** Medido com `VICTORY_BACK_READS=alt`, mas a solucao ("nao copiar de volta o que ninguem le") esta apenas **desenhada**: `backreads.rs` e um passo de medicao, nao de otimizacao. **CONFIRMADA** a medicao.

**G-09 - O gargalo migrou para a thread de preparo de submissao** ("the card now idles 12 to 40 ms a frame: the thread that prepares submissions is the limit") e a solucao nao esta escrita. **CONFIRMADA**.

**G-10 - Traducao de shaders incompleta.** O README upstream (mantido abaixo da linha) declara: mesh 8.191 de 8.506, vertex/pixel 10.617 de 12.346, compute 1.643 de 2.539 - numeros de 23/11, nao atualizados pelo fork; `a26859f` diz "14 more programs translate", sem novo total. **NAO VERIFICADO** o total atual.

**G-11 - Divida de testes** (D-17) e **lints afrouxados**: `6a2d9f9` habilita 8 lints de estilo (`type_complexity`, `needless_range_loop`, `chunks_exact_to_as_chunks`, `sliced_string_as_bytes`, `collapsible_match`, `manual_clamp`, `implicit_saturating_sub`, `min_max`) e `undocumented_unsafe_blocks = "allow"` "until the newer code is tidied". **CONFIRMADA** no `Cargo.toml`.

**G-12 - 164 marcadores `shortcut:`** no codigo Rust (contagem minha, `Select-String -CaseSensitive 'shortcut:'`), isto e, 164 pontos em que o codigo declara aproximar o console em vez de reproduzi-lo. Nao e defeito por si; e o mapa do que **nao** e fiel. **CONFIRMADA** a contagem.

**G-13 - Referencia a documento inexistente.** `crates/victory-hle/src/modules/display.rs:61` remete a "the presentation items in `TODO.md`", e `TODO.md` nao existe na arvore (`git ls-files`). Deriva documental herdada; sem impacto funcional. **CONFIRMADA**.

**G-14 - Switches nunca rodados em jogo.** `6d20eed` declara, para os tres mecanismos que introduz: "All off by default; none has run in a game yet." O mesmo vale para `VICTORY_CARD_HASH_WIDE` (feito opt-in exatamente por isso). **CONFIRMADA**.

---

## H. Regressoes

**H-01 - A atualizacao de wave64 derruba o device (24/11).** `6a2d9f9` melhora a traducao e, junto, liga a forma wide por default. Resultado: 4/4 runs com device perdido (`004bf69`), corrigido em `e9765a7` (D-01). Classica regressao "o ganho veio com um custo escondido".

**H-02 - A chave do cache esconde a regressao anterior.** `e9765a7` revela que rodar com `VICTORY_WAVE_WIDE=0` podia servir traducoes cacheadas da forma wide. Entre `6a2d9f9` e `e9765a7` o desligamento do switch **nao** desligava o codigo; qualquer conclusao tirada nessa janela sobre a forma wide e suspeita. **CONFIRMADA** pelo texto do commit e pelo diff do `shader_cache.rs`.

**H-03 - Reversao e reaplicacao de #11 em 60 segundos.** `9e49b8a` remove, `2119b0e` devolve ("that was not wanted"), `90228a7` traz a versao com look-ahead. A arvore muda de estado tres vezes na mesma sessao; qualquer build entre `9e49b8a` e `2119b0e` tem a mudanca ausente. **CONFIRMADA**.

**H-04 - O PR #220 reabre a instabilidade de device.** Entre `9376fd6` (que registra "not in this tree") e `f85101b` (que o mergeia), as janelas voltam e derrubam a placa 3 vezes antes de o guard de 2^23 entrar no **mesmo** commit. **CONFIRMADA** a sequencia; **PROVAVEL** que o guard fecha o caso (5 runs sem perda e relato do autor).

**H-05 - Correcao de corretude de descritor vira regressao de desempenho.** `a26859f` introduz `sync_heaps()` e o passe custa 70-220 ms/frame (BLAKE3 + espelhar tudo). Corrigido em `9376fd6` (~3 ms). **CONFIRMADA** pelos numeros nos dois commits.

**H-06 - Otimizacao de load escala pinta a agua de preto.** `c36971b` (bit 64) e desligado no default justamente porque o run 277 mostrou agua preta; `44bff61` e `d827419` fecham o diagnostico (hash length limpo, loads agrupados como causa). Nao afeta o default. **CONFIRMADA** (sintoma), causa **HIPOTESE**.

**H-07 - Possivel regressao de estabilidade apos o copy-back de textura.** Run 275 (sem `TEXTURE_BACK`) jogou 20 minutos sem erro; run 276 (com) perdeu a placa em 6 minutos; run 277 perdeu em 49 minutos. n=1 de cada lado e o proprio autor nao conclui. **HIPOTESE**.

**H-08 - Regressao de medicao: 6,0 -> 8,9 fps.** Publicada em `a253706`, retratada em `0b0074e` (D-14). Nao e regressao de codigo, e a regressao mais custosa em credibilidade do repositorio: o numero circulou por um dia e foi usado para justificar as janelas. **CONFIRMADA**.

**H-09 - Previsao nao realizada (nao e regressao, mas foi tratada como tal).** `79cfdd9` esperava que o upload direto por resizable BAR reduzisse as listas de envio e hash; mediu 12,3 contra 13,6 ms/frame "at like draws" e registrou "A memory gain; no speed gain shown". O README do mesmo commit diz "the code of this entry is in the tree" - antes dizia "the code is not in this tree yet". **CONFIRMADA**.

---

## I. Conhecimento reutilizavel

### I.1 O mecanismo que permite varios builds privados divergirem do mesmo codigo

O achado estrutural deste fork nao e um algoritmo: e a **disciplina de switches de ambiente** que transforma um binario em uma familia de builds. Sao **219 nomes distintos** lidos por `env::var`/`env::var_os` (`Select-String` case-sensitive sobre `*.rs`), em 246 pontos de leitura. Juntos, eles cobrem: correcao, desempenho, memoria, diagnostico, validacao e selecao de host.

As regras que o codigo segue, e que explicam por que varios builds privados podem divergir sem forks de codigo:

1. **Default = o que ja rodou em jogo.** Um mecanismo novo nasce desligado; so vira default depois de um run real. Frases literais: "Off by default until a game run has carried it" (`shadow.rs`, `changed_list`), "All off by default; none has run in a game yet" (`6d20eed`), "off by default: it has not run in a game on this tree yet" (`card_hash_wide`).
2. **O switch diz que esta ligado, no log.** `card_hash_wide` loga ao ligar; `changed_list` loga a cada alternancia; `spread` loga os processadores extras; `probe_view_limit` loga o tamanho de heap escolhido. Isso e o que permite reconstruir a configuracao de um run a partir do log - e o que o `diagnostics/timing-run-2026-11-25.txt` faz.
3. **`=alt`: A/B dentro de um unico run.** Vinte segundos com, vinte sem, em turnos, cada troca logada. Resolve o maior problema de medicao do repositorio (dois runs nunca estao na mesma cena - ver D-14). Presente em `VICTORY_CHANGED_RUNS=alt`, `VICTORY_CARD_CHANGED_LIST=alt`, `VICTORY_ABSENT_ONCE=alt`, `VICTORY_SPREAD=alt`, `VICTORY_WIDE_REFRESH=alt`, `VICTORY_STAGING_POOL=alt`, `VICTORY_BACK_READS=alt`, `VICTORY_WINDOW_JUMP=alt`, `VICTORY_CARD_REHASH_RUNS=alt`, `VICTORY_WAVE_LOCAL`.
4. **O switch nao pode mentir sobre o artefato.** Toda flag que muda a emissao entra na chave do cache de shader (D-02, F-01) e o limite de heap do host tambem (`31714de`).
5. **Fallback dinamico, nao apenas estatico.** `drvwatch::tripped()` e `FILL_WAIT`'s `WAIT` mudam de valor **durante** o run: o primeiro erro de driver desliga janelas; o primeiro mismatch religa a espera. O switch e o default; o estado de execucao pode sobrepor.
6. **Pares que precisam andar juntos.** `changed_kept()` responde "manter a lista?" e `changed_list()` "usar a lista?"; `card_hash_wide()` escolhe a forma e `card_compare_groups()` muda o dispatch; `wide_on` e `deep_on` compartilham o gate. Sao os pontos onde um build privado quebra se ligar so metade.
7. **Switch morto fica, desligado, documentado.** O README tem uma secao fixa "Measured and found to give nothing (switches left in, off): `VICTORY_REFRESH_ONCE`, `VICTORY_ASYNC_PRESENT`, `VICTORY_VIEW_KNOWN`, ...". E um registro negativo - tao importante quanto o positivo, porque impede que o proximo build privado repita o experimento.

### I.2 Catalogo minimo de switches (por valor estrategico)

| Switch | Default | O que faz | Por que importa |
|---|---|---|---|
| `VICTORY_ASYNC_SUBMIT` | off | thread propria prepara e envia submissoes | tirou 257 de 292 ms da thread de render |
| `VICTORY_QUEUE2` | off | copy queue para fetch/up | espera do copy-back 125 -> 7 ms |
| `VICTORY_INDIRECT_WINDOW` | off | janela de registros indirect + clamp em lote | maior ganho de placa; exige o guard 2^23 |
| `VICTORY_INDIRECT_GATHER` | off | um clamp por lista para registros espalhados | 10,3 fps nas primeiras salas |
| `VICTORY_COPY_PLAIN` | off | copia in-place com handler | kernel do submit 44 -> 20 ms |
| `VICTORY_CARD_BUFFERS` / `_MIN_MB` / `_MB` / `_MIN_KB` | on / 16 / 512 / (64 off) | copias na memoria da placa | -18 ms de placa, -5 ms de frame com `_MIN_KB=64` |
| `VICTORY_CARD_HASH_LEN` | 0 | hash so dos bytes uteis do buffer | corrige agua preta; ~1/4 mais barato por lista |
| `VICTORY_CARD_HASH_WIDE` | off (opt-in) | hash por grupo de 64 threads | 35 -> 10 ms de placa; outro hash |
| `VICTORY_CARD_CHANGED_LIST` | off | lista de copias sujas em vez de varredura | ~3.900 copias, 10-18 submissoes/frame |
| `VICTORY_CHANGED_RUNS` | mascara | runs de bytes mudados por mascara de 64 bytes | -3,8 ms/frame na thread de submit |
| `VICTORY_WAVE_WIDE` | **off** | forma wave64 com barreiras de grupo | hang de device; so com `=1` |
| `VICTORY_SHADER_OPT` | 63 | passes de limpeza no DXIL (bit 64: loads agrupados) | -43% instrucoes, ~20 ms de GPU |
| `VICTORY_HEAP_SYNC` / `_TABLES` | on / off | espelhar registros copiados como bytes | 40.000 slots/frame |
| `VICTORY_STALE_TEXTURES` | `Start` | regra de invalidez de views | corrige texturas vizinhas |
| `VICTORY_HEAP_RECHECK` | on | reexame de textura enviada inteira | corrige rosto/cabelo |
| `VICTORY_LEVELS_ON_DEMAND` | off | memoria de placa por nivel mantido | -3,1 GB |
| `VICTORY_TEXTURE_BACK` | off | textura escrita por shader volta para a CPU | o barco flutua |
| `VICTORY_VIEW_GUARD` | on | guarda em todas as views da pagina | 0,8-1M escritas invisiveis |
| `VICTORY_PAGE_WRITES` | contado | abre guardas sob escrita do storage | 10k-50k blocos |
| `VICTORY_ABSENT_ONCE` / `ABSENT_MS` | off / 100 | nao reperguntar bloco nao mapeado | 690.000 perguntas/20 s |
| `VICTORY_DRIVER_WATCH` | on | assina event log do driver | canario de perda de device |
| `VICTORY_FAST_CORES` | on | threads da camada nos cores rapidos | 6% -> 0 em core lento |
| `VICTORY_SPREAD` | off | espalha threads presas do titulo | 11 threads em 2,5 cores |
| `VICTORY_FRAME_LOG` / `_CARD_METER` | diag | composicao de cada frame / tempo por lista | o instrumento que achou D-13 |
| `VICTORY_PROFILE` | off (diag) | profiler amostral interno | achou o submit path |
| `VICTORY_SHOT` | off | captura de frame dentro da camada | substitui RenderDoc, que quebrou |
| `VICTORY_GPU_TIMING` / `D3D12_DRED` | off | timing por lista / breadcrumbs | log higienizado publicado |

### I.3 Padroes reutilizaveis (com dependencias e riscos)

**I.3.1 - Lista de objetos sujos ativada por transicao vazio -> nao-vazio.** Problema: varrer N objetos a cada submissao para achar poucos mudados (3.900 copias x 10-18 submissoes/frame). Implementacao (`6d20eed`, `shadow.rs`): cada copia guarda `Weak<Shadow>` em sua propria estrutura; quando `dirty` passa de vazio para nao-vazio, empurra o weak para `CHANGED`; o consumidor faz `mem::take` da lista e `upgrade()` de cada weak. Ordem dos objetos pode diferir; os ranges sao identicos. Riscos: (a) a lista precisa ser **drenada** mesmo no modo varredura, senao cresce sem limite (o codigo faz isso explicitamente); (b) `Weak` exige que o objeto se registre uma vez (`me: OnceLock<Weak<Shadow>>`); (c) transicao perdida = objeto sujo nunca enviado - e por isso que o modo `alt` existe para comparar com a varredura.

**I.3.2 - Duas granularidades para a mesma pergunta.** `changed_runs` tem tres implementacoes no mesmo arquivo: byte a byte (`changed_runs_bytes`), mascara de 64 bytes (`changed_runs_wide`) e alternancia; e dois testes de equivalencia (`changed_runs_are_the_ones_a_walk_byte_by_byte_finds`, `changed_runs_are_exactly_the_bytes_that_differ`). E a forma mais barata de otimizar um diff sem perder a referencia: **manter a versao ingenua como oraculo testado**.

**I.3.3 - Comparar na GPU, buscar so o que difere.** `shadow.rs`: copia "before" na placa, passe de comparacao por peca de 64 KiB, lista de pecas diferentes, fetch so delas. Dependencia: assumir que shaders escrevem apenas em ranges declarados como escritos (o codigo declara essa premissa e o que acontece quando ela falha). Reutilizavel para qualquer readback caro.

**I.3.4 - Amostrar o que a CPU realmente leu, sem tocar em protecao.** `backreads.rs` (694 linhas): depois do store, as paginas de 4 KiB do trecho sao tiradas do working set com `VirtualUnlock` (Documentado: paginas nao locked), mantendo os bytes em memoria fisica porque a view propria da secao segura as mesmas paginas; o proximo acesso por endereco do titulo e um soft fault que o Windows resolve sozinho e que recoloca a pagina no working set, visivel por `QueryWorkingSetEx`. Medido: 5,4 us para trimar 64 KiB com seis threads rodando, 2,8 us para perguntar das 16 paginas, 0,89 us por pagina lida. **Nenhum byte muda, nenhuma protecao muda.** E a tecnica mais transferivel de todo o repositorio para responder "quem le esta memoria?" sem instrumentar o leitor.

**I.3.5 - Fingerprint no lugar de hash, com contagem de dispensados.** `COPIED_TABLED` conta quantos slots foram deixados para os draws; o log imprime por frame. Padrao: toda otimizacao "nao fazer" deve reportar **quanto** deixou de fazer, senao a proxima pessoa nao sabe se esta funcionando.

**I.3.6 - Sanitizar o que se publica.** `diagnostics/README.md` descreve o scrubbing: nomes de usuario/maquina, pastas, nomes de arquivo/produto/pack do titulo, title id, nomes de rede, nomes de shader e programa do titulo sao substituidos por placeholders; hashes de shader e pipeline ficam. Um log real, anonimizado, e mais util que uma tabela de numeros - e o fork publicou um.

### I.3.7 - Onde NAO seguir este repositorio

- **Nao copie a explosao de switches.** 219 nomes, 246 leituras, triplicacao de parsers (`match env::var(...) { Ok("1") => 1, Ok("alt") => 2, _ => 0 }` repetido dezenas de vezes) e pares que precisam andar juntos. Funciona para um operador unico; em equipe, vira um sistema de configuracao sem schema. O que vale copiar e a **regra** (default off, loga, alt, entra na chave), nao a forma.
- **Nao copie o padrao de merge sem teste na arvore mergeada** (D-17).
- **Nao confie em numero de fps de commit message.** Tres dos maiores ganhos anunciados eram de placa, nao de frame (E-07); um foi retratado (D-14).
- **Cuidado com `shortcut:`** (164 ocorrencias): sao aproximacoes conscientes do console, e varias sao especificas deste titulo e desta GPU.

---

## J. Conclusao tecnica

**O que este repositorio e.** Um fork de pesquisa de altissima densidade: 40 commits proprios em ~29 horas, +83.615 linhas contra a base upstream, com infraestrutura de medicao (framelog, card meter, profiler amostral, captura de frame, driver watch, back-read accounting) que a maioria dos projetos comerciais nao tem. O autor nao so otimiza: **constroi o instrumento antes de otimizar** - e quando o instrumento contradiz o numero publicado, retrata (`0b0074e`) e quando o instrumento e a causa do problema, documenta (`a253706`, D-13).

**Onde ele e forte.** (1) Corretude de **imagem** em um alvo dificil: a primeira sala sai com paredes, texturas, pessoas e luz de lampada, o que exigiu consertar staleness de textura (D-05), corrida de upload (D-06), registros de descritor copiados como bytes (D-07), view heap (D-08) e o retorno de texturas compute-escritas para a CPU (D-11). (2) **Desempenho de placa**: PR #1014 (35 -> 10 ms), PR #220 com guard (placa ~metade), `CARD_BUFFERS_MIN_KB` (-18 ms de placa). (3) **Disciplina experimental**: 219 switches com default off, `=alt`, log ao ligar, cache key derivada - este e o mecanismo que permite que varios builds privados divirjam do mesmo codigo sem forks, e e a contribuicao mais reutilizavel do repositorio.

**Onde ele e fraco.** (1) **Estabilidade**: perdas de device sem causa em pelo menos quatro configuracoes (D-16), incluindo uma no mundo aberto por esgotamento de memoria de textura. (2) **Divida de verificacao** (D-17): merges que nunca rodaram a suite na arvore mergeada, 219 switches sem schema, lints afrouxados, 164 `shortcut:`. (3) **Ganho de frame abaixo do ganho de placa**: o gargalo migrou para a thread de preparo (placa ociosa 12-40 ms/frame) e para o page mapping do titulo (32 ms por frame de cutscene); nenhuma das duas solucoes esta escrita. (4) **Default nao e o numero divulgado**: os 8-10 fps sao do conjunto de switches; o default foi medido em 420-450 ms e nao foi remedido.

**Posicao no ecossistema.** Pelo conteudo, este e o fork com o conjunto mais completo de **instrumentacao + switches** entre os cinco comparados; nao e o mais avancado em shaders (AvatarManz tem mais arquivos e um replay de frame; DexNom tem `bvh.rs` e `texback.rs` proprios), mas e o que documenta o metodo com mais honestidade - inclusive os fracassos. Os 59,4% de lideranca do build privado no PortWatch **nao sao verificaveis** a partir deste repositorio (nenhum placar, nenhum log de benchmark externo, nenhum identificador de build); o que e verificavel e que o README declara 8-10 fps nas primeiras salas com o conjunto de switches, o que e consistente com um build de topo entre os forks, e insuficiente para confirmar a colocacao.

**Para quem forka daqui.** O valor nao esta nos numeros: esta nos instrumentos (`framelog`, `cardmeter`, `drvwatch`, `backreads`, `shot`, `profile`), nas tres correcoes de causa raiz que se pagam em qualquer camada (view guard em todas as views, copia in-place com limiar, hash paralelo com reducao ordenada) e na regra de ouro do cache. O risco esta em herdar 219 switches e 164 aproximacoes sem o contexto de qual deles foi validado em jogo - contexto que so o README, os comentarios e este documento carregam.

---

## K. Indice de commits

253 commits, em ordem cronologica (raiz primeiro). Hash curto, data de autoria, autor e assunto. Assuntos acima de 120 caracteres foram truncados com `...`; os assuntos integrais estao em `analise/indices/victory-fork_log_full.txt` e em `git log`. Commits 1-213 sao o historico upstream rehospedado; 214-253 sao o fork (`578d459` e o squash inicial).

| hash | data | autor | assunto |
|---|---|---|---|
| `f4504f6` | 2026-11-23 | Marvis | Initial commit |
| `12caefe` | 2026-11-23 | Marvis | GFX10 operand fields and SP3 text; FSSM wave size |
| `754619f` | 2026-11-23 | Marvis | Run Minecraft: runtime HLE, D3D12.X device on the host GPU, shader IR |
| `d032b8a` | 2026-11-23 | Marvis | Title DLLs at run time: find by name, map on first LoadLibrary |
| `4b25269` | 2026-11-23 | Marvis | Thread notices for title images |
| `bc30709` | 2026-11-23 | Marvis | Thread notices: leave out threads Rust's std starts |
| `2849477` | 2026-11-23 | Marvis | Run Grand Theft Auto VI from first boot into the world |
| `015d45b` | 2026-11-23 | Marvis | README: status, subsystem table, how to run, debugging switches |
| `c21208c` | 2026-11-23 | Exorion | Loader: keep the cleared TLS callback pointer out of base relocations |
| `e8f3b1c` | 2026-11-23 | Marvis | Merge pull request #4 from Exorion1er/fix/tls-callbacks-reloc |
| `934e67f` | 2026-11-23 | ismailchaka1 | loader: emulate AMD-only instructions on Intel CPUs |
| `005ad49` | 2026-11-24 | 9hvn9yt2vk-netizen | xma: stop at a zero-length frame instead of looping forever |
| `d4ed2de` | 2026-11-23 | raffclar | Process exit: start the watchdog before other threads end |
| `d348b9a` | 2026-11-24 | Marvis | Merge pull request #10 from ismailchaka1/emulate-amd-only-instructions |
| `e6a2c54` | 2026-11-24 | 9hvn9yt2vk-netizen | texture: refuse extents past 16384 texels instead of overflowing |
| `36eb0b4` | 2026-11-23 | Marvis | Shaders: med3 with NaN, fract below 1.0, signed bit fields past bit 31 |
| `0b499d7` | 2026-11-24 | Marvis | Shaders: sine and cosine in turns, exact zeros at half and quarter turns |
| `8c26061` | 2026-11-24 | Marvis | GPU: the world stays up; indirect execution behind a switch |
| `3827767` | 2026-11-24 | Nostyxx | GPU: make tables past a capped host view heap afresh |
| `1dd7afd` | 2026-11-24 | Marvis | Merge pull request #14 from 9hvn9yt2vk-netizen/fix-xma-zero-length |
| `e98d3ca` | 2026-11-23 | Manuel Perez | PE: checked file offset in rva_to_file_offset |
| `d85097a` | 2026-11-23 | ifBars | Keep an empty import descriptor for the Windows 10 loader |
| `29b558c` | 2026-11-23 | Jeremy Goldberg | test(gpu): use the SDK root parameter in the serializer fixture |
| `9e04419` | 2026-11-24 | Marvis | Merge pull request #15 from 9hvn9yt2vk-netizen/fix-texture-extents |
| `0c91c03` | 2026-11-23 | raffclar | Loader: test the AMD-only instruction emulation on any CPU |
| `d222b87` | 2026-11-24 | Marvis | Merge pull request #19 from raffclar/exit-watchdog |
| `97c78d1` | 2026-11-24 | Marvis | Merge pull request #21 from Nostyxx/fix/nvidia-view-heap-cap |
| `f00caec` | 2026-11-23 | raffclar | Loader: test 64-bit extrq at index 0, not in AMD's undefined range |
| `0b607b1` | 2026-11-23 | LaunchDoesntCode | GPU: verify WARP adapter is software before using it |
| `ea24ea1` | 2026-11-24 | 9hvn9yt2vk-netizen | shader: a lane mask that needs its own value fails instead of overflowing the stack |
| `71cfe84` | 2026-11-23 | Henrique Bueno | loader: emulate clwb on Intel CPUs before Ice Lake |
| `15b0091` | 2026-11-24 | 9hvn9yt2vk-netizen | shader: VICTORY_SHADER_LOOP_LIMIT to find what makes world frames slow |
| `e158d95` | 2026-11-24 | Vanthys | Guard texture and XMA offsets against overflow |
| `4cca767` | 2026-11-24 | Vanthys | Reject empty XMA test fixture directories |
| `7e42fef` | 2026-11-23 | BaggyALPHA | Report Windows commit-limit failures in crashes.log |
| `ef22723` | 2026-11-24 | Manuel Perez | PE: remove leftover dbg_tls test and check layout offsets |
| `2e5001c` | 2026-11-24 | alessiodeia | shader: keep fract(NaN) as NaN on NVIDIA and add synthetic GPU VALU tests |
| `4da3d63` | 2026-11-23 | ifBars | Clear the IAT directory alongside the import directory |
| `5fbea47` | 2026-11-24 | 9hvn9yt2vk-netizen | shader: look up an input channel without underflowing |
| `071487f` | 2026-11-24 | Yaekai | GPU: emit null buffer SRV/UAV when descriptor names a buffer with null resource |
| `0c56091` | 2026-11-24 | Yaekai | Shaders: lower v_xnor_b32 and v_swap_b32 |
| `4e94c2b` | 2026-11-24 | TG0000 | xmem: write DirectStorage page destinations one view per run |
| `fc807b6` | 2026-11-23 | ifBars | Reuse texture row spans when converting Xbox pixel layouts |
| `35b054e` | 2026-11-24 | Marvis | GPU: pipelines off the render thread and cached; indirect execution on |
| `f3c7f0f` | 2026-11-24 | Manuel Perez | GPU: leave rectangle vertex components past the stride alone |
| `25a3e95` | 2026-11-23 | ifBars | Keep an empty import descriptor so Windows assigns static TLS |
| `78b8404` | 2026-11-23 | meower | xmem: commit physical-page sections whole under Wine |
| `0eca412` | 2026-11-24 | FIocker | GPU diagnostics: skip pipelines by shader hash |
| `c4e915f` | 2026-11-23 | Henrique Bueno | loader: a clwb write-back loop runs in one exception |
| `9a5cbbe` | 2026-11-24 | tedin7 | fix: skip unused export address slots instead of reporting them at RVA 0 |
| `4c1766c` | 2026-11-24 | Exorion | GPU: keep a guarded block's write fault from being declined in a race |
| `73d776b` | 2026-11-24 | TG0000 | GPU: check a sampled texture once per submission, not once per list |
| `c963b71` | 2026-11-24 | tedin7 | fix: read a far jump's offset as signed so backward far jumps decode |
| `a2f5f43` | 2026-11-24 | tedin7 | Shaders: set SCC from the carry of s_addc_u32 and s_subb_u32 |
| `b304b7d` | 2026-11-23 | Jeremy Goldberg | fix(gpu): average copy statistics over the full reporting interval |
| `257d8bb` | 2026-11-24 | Manuel Perez | Audio: reduce the title's XMA read advance before adding it |
| `ec0d155` | 2026-11-24 | Manuel Perez | Watch: saturate the end of announced writes and remaps |
| `38ae55d` | 2026-11-24 | FIocker | Keep the diagnostic skip outside indirect argument validation |
| `0e5953d` | 2026-11-23 | ifBars | Merge branch 'main' into fix/windows10-import-directory |
| `3e982f7` | 2026-11-24 | TG0000 | BCPack: find a slot's bucket from a window table |
| `7d33c46` | 2026-11-24 | Marvis | Merge pull request #45 from HenriqueSBueno/clwb |
| `27b530e` | 2026-11-24 | Duff | Files: send rewritten narrow paths to the host, not to our wide wrappers |
| `27e6442` | 2026-11-24 | Marvis | Merge pull request #28 from ifBars/fix/windows10-import-directory |
| `24480a7` | 2026-11-24 | TG0000 | Shaders: find the written instructions with a worklist, not repeated passes |
| `cbd71a9` | 2026-11-24 | Marvis | Merge pull request #38 from raffclar/emulation-tests |
| `cc51927` | 2026-11-24 | Marvis | Merge pull request #30 from AndyLatimer/fix/pe-rva-offset-overflow |
| `bf6c557` | 2026-11-24 | Marvis | Merge pull request #75 from tedin7/vic-pe-export-holes |
| `062ad14` | 2026-11-24 | Marvis | Merge pull request #50 from Vanthys/fix/format-offset-overflow |
| `a6888a6` | 2026-11-24 | Marvis | Merge pull request #43 from 9hvn9yt2vk-netizen/shader-mask-cycle |
| `8fe0b74` | 2026-11-24 | Marvis | Merge pull request #56 from 9hvn9yt2vk-netizen/find-input-underflow |
| `4a8a058` | 2026-11-24 | Marvis | Merge pull request #58 from Yaekai/feat/shader-xnor-swap |
| `46610eb` | 2026-11-24 | Marvis | Merge pull request #81 from tedin7/vic-scalar-carry |
| `4926f72` | 2026-11-24 | Marvis | Merge pull request #80 from tedin7/vic-far-jump-backward |
| `237f2e8` | 2026-11-24 | Marvis | Merge pull request #37 from alessiodeia/fix/valu-med3-fract-trig |
| `3c86053` | 2026-11-24 | tedin7 | Shaders: lower s_bitcmp0/1_b32 and s_bitcmp0/1_b64 |
| `eda38aa` | 2026-11-24 | Marvis | Merge pull request #57 from Yaekai/fix/null-buffer-views |
| `683278b` | 2026-11-24 | Marvis | Merge pull request #36 from jeremygold02/codex/root-parameter-fixture |
| `fc0c4dd` | 2026-11-24 | Marvis | Merge pull request #77 from Exorion1er/fix/guard-fault-race |
| `0f35dfc` | 2026-11-24 | Marvis | Merge pull request #66 from AndyLatimer/fix/gpu-rect-element-past-stride |
| `fa5ef73` | 2026-11-24 | Marvis | Merge pull request #79 from TG0000/perf/texture-check-once-per-submission |
| `93bc978` | 2026-11-24 | Marvis | Merge pull request #60 from ifBars/perf/reuse-texture-row-spans |
| `97464ba` | 2026-11-24 | Marvis | Merge pull request #61 from TG0000/hle/physical-page-runs |
| `6168fc6` | 2026-11-24 | Marvis | Merge pull request #70 from FIocker/gpu/diagnostic-pipeline-filter |
| `91e9bc9` | 2026-11-24 | Marvis | Merge pull request #46 from 9hvn9yt2vk-netizen/shader-loop-limit |
| `4c0bb89` | 2026-11-24 | Marvis | Merge pull request #41 from LaunchDoesntCode/codex/proton-warp-safety |
| `8a8a38d` | 2026-11-24 | Marvis | Merge pull request #53 from a55uka/fix/wine-physical-pages |
| `9aef3d9` | 2026-11-24 | Marvis | Merge pull request #51 from BaggyALPHA/fix/windows-commit-limit-diagnostic |
| `7102a49` | 2026-11-23 | LUIS LARIOS | GPU: rename the second test module in host.rs so the tests compile |
| `1823079` | 2026-11-24 | lordon1a | Shaders: lower v_cmp_class_f16 and v_cmpx_class_f16 |
| `d2a4bae` | 2026-11-24 | Marvis | Merge pull request #129 from Luislariosss/fix/duplicate-test-module |
| `7d73e2b` | 2026-11-24 | Codex | pipeline: remove staging files when cache publication fails |
| `02fcf72` | 2026-11-24 | Maraco91 | xmem: mapping or unmapping no pages changes nothing |
| `babe2db` | 2026-11-24 | Duff | Crash reports: show the bytes of an instruction the CPU refused |
| `3390e47` | 2026-11-24 | Duff | Crash reports: say what an access violation's address holds |
| `cbb50b8` | 2026-11-23 | julio | GPU: give null texture UAVs the title's view dimension |
| `c88b7ac` | 2026-11-24 | Maraco91 | Game saves: an empty configuration id is the default, not every configuration's folder |
| `afc4641` | 2026-11-24 | Andrea Casale | net: skip newer-Windows-only exports in only_loopback_is_reachable |
| `18b2716` | 2026-11-24 | Andrea Casale | net: rustfmt the presence assertion |
| `d46b078` | 2026-11-24 | Duff | XUser: a gamertag without a suffix has an empty modern suffix |
| `5f670bb` | 2026-11-24 | Duff | Loader: a trapped unimplemented import goes into the crash log |
| `36d6a79` | 2026-11-24 | Maraco91 | XTaskQueue: terminating a queue ends a dispatch waiting on it |
| `79fb722` | 2026-11-24 | Duff | Shaders: s_ttracedata_imm is SOPP opcode 0x28, not 0x26 |
| `63df363` | 2026-11-24 | Duff | GPU: a half-float rounding carry raises the exponent, not halves the value |
| `717f322` | 2026-11-24 | Maraco91 | GPU: GetDevice and the DXGI device hand out the device at every level |
| `5f29f3a` | 2026-11-24 | Duff | Loader: a title DLL that fails to prepare is named in the error |
| `9fe73ba` | 2026-11-23 | artor410 | shader: decode the packed typed-buffer formats |
| `a3d3bc9` | 2026-11-24 | Claude | Shaders: trace descriptors chosen by s_cselect and s_cmov |
| `ffa6635` | 2026-11-24 | Claude | Shaders: lower waterfall loops over per-lane descriptors in one pass |
| `b5c6678` | 2026-11-24 | Abdellah K. | loader: unwind a 16-byte aligned copy of the context |
| `3c81420` | 2026-11-24 | hophaver | GPU: drop draws whose fresh tables cannot be made; 2^18 dynamic slots |
| `cc63aa2` | 2026-11-23 | Enzo Souza Duarte | Config: ignore an empty VICTORY_TEST_TITLES |
| `40c127c` | 2026-11-23 | Enzo Souza Duarte | Chore: clear two workspace Clippy errors |
| `efb3e3d` | 2026-11-24 | FIocker | GPU diagnostics: order tied drop counts consistently |
| `376b3cf` | 2026-11-23 | marquezini | Storage: a path buffer too small fails with E_FAIL, as on Xbox |
| `a4e7f1d` | 2026-11-24 | FIocker | GPU: replay pending descriptors before publishing the host heap |
| `0f369d6` | 2026-11-23 | marquezini | Capture: R16G16B16A16_UNORM targets are integers, format 89 is 10:10:10:2 |
| `ded334b` | 2026-11-23 | Luiz Neves | Shaders: lower v_cvt_norm_i16_f16 and v_cvt_norm_u16_f16 |
| `404d7fd` | 2026-11-24 | Duff | Shaders: PSV0 says a pixel shader runs per sample when an input does |
| `c2f60d6` | 2026-11-24 | Duff | Shaders: a pixel shader writing SV_StencilRef declares StencilRef |
| `2af66ff` | 2026-11-23 | Luiz Neves | Shaders: v_cvt_u16_f16 and v_cvt_i16_f16 saturate at 16 bits |
| `ff8694a` | 2026-11-23 | itsRevela | Window: keep Alt and F10 from freezing the title in menu mode |
| `eb30a0d` | 2026-11-24 | Enzo Souza Duarte | GPU: write half subnormals from f32 rectangle outputs |
| `28c1b07` | 2026-11-24 | marquezini | XGameSave: a read result into a small buffer is E_GS_PROVIDED_BUFFER_TOO_SMALL |
| `d5704aa` | 2026-11-24 | Pedro Fortunato | fix(shader): lower dynamic v_perm_b32 without INT64 |
| `f0ee35e` | 2026-11-24 | marquezini | XGameSave: a full host disk is E_GS_OUT_OF_LOCAL_STORAGE, not the quota |
| `f4438b3` | 2026-11-23 | local | Shaders: reads of uninitialized SGPRs read as zero, as VGPRs do |
| `1106f43` | 2026-11-24 | r4. | Shader tests: run the VALU GPU tests one at a time |
| `077ec8b` | 2026-11-24 | Claude | translate: count per-lane fetched descriptors as loaded |
| `bf84134` | 2026-11-24 | Duff | Shaders: maximums and minimums order -0 below +0, as RDNA2 does |
| `e8d3b64` | 2026-11-24 | hophaver | GPU: the larger dynamic range only where the heap still fits |
| `6285e19` | 2026-11-24 | Claude | Shaders: take the first lane's descriptor as each lane's own only in a waterfall loop |
| `6638098` | 2026-11-24 | r4. | Shaders: per-lane descriptors chosen by control flow |
| `c249f02` | 2026-11-23 | Saitken91 | Shaders: centroid position with conservative depth |
| `9a8908c` | 2026-11-24 | Advait Palve | core: show data_dir in config and check its asset policy |
| `25b0f88` | 2026-11-24 | Dathinel | Test probe: compare direct and joined reads of a shader timestamp |
| `3dd467c` | 2026-11-23 | KoLolX3 | Shaders: a phi that only another read phi consumes is live too |
| `260611a` | 2026-11-23 | Saitken91 | Shaders: image_sample_o with constant offsets |
| `a0efd2d` | 2026-11-24 | AlanQiang0418 | Shaders: decode GFX10 packed inline constants by operand type |
| `b1c8306` | 2026-11-24 | fuzeofficial | Shaders: prove EXEC selection before collapsing a waterfall |
| `6c63dea` | 2026-11-24 | r4. | Shaders: name why a loop is no waterfall, and test the shapes found in review |
| `d74558c` | 2026-11-24 | Carlos Sant'Anna | GPU diagnostics: describe a buffer UAV by its range, not as uav None |
| `764fa19` | 2026-11-23 | lameloball2813-ui | Loader tests: build the probe fixture under CARGO_TARGET_TMPDIR |
| `0064c08` | 2026-11-24 | fuzeofficial | GPU: upload CPU-written textures before CopyResource |
| `af22e5d` | 2026-11-23 | EatCodeSleep2 | GPU: use the host upload path for texture staging copies |
| `5864aab` | 2026-11-23 | lameloball2813-ui | Docs: rustdoc builds the loader, GPU and shader crates without warnings |
| `31ce30f` | 2026-11-24 | fuzeofficial | GPU: unbind vertex buffer slots for null view arrays |
| `ab3eb92` | 2026-11-23 | Louis Faust | Shaders: gate the constant_color test helper to Windows |
| `4f53d6f` | 2026-11-24 | fuzeofficial | GPU: upload CPU-written sources before texture region copies |
| `0f0c872` | 2026-11-23 | Louis Faust | Docs: drop the private intra-doc link to MICRO in texture.rs |
| `51f1627` | 2026-11-24 | Lyrnc | Shaders: v_cmp_lg is ordered on AMD hosts too (olt / ogt, not fcmp one) |
| `3e39eac` | 2026-11-23 | quaternion | fix(hle): return XAsync payloads for nonnegative results |
| `1f858b0` | 2026-11-24 | Marvis | Merge pull request #25 from artor410/packed-buffer-formats |
| `c2e7254` | 2026-11-24 | Marvis | Merge pull request #87 from jeremygold02/codex/copy-stat-frame-window |
| `0e990e4` | 2026-11-24 | Marvis | Merge pull request #104 from TG0000/perf/bitcode-worklist-liveness |
| `b0328ba` | 2026-11-24 | Marvis | Merge pull request #111 from Dufftwt/fix/narrow-paths-translated-twice |
| `3d77d5e` | 2026-11-24 | Marvis | Merge pull request #117 from tedin7/vic-s-bitcmp-2 |
| `80c0d12` | 2026-11-24 | Marvis | Merge pull request #140 from lordon1a/feat/cmp-class-f16 |
| `ba7d873` | 2026-11-24 | Marvis | Merge pull request #152 from Makto94/fix/net-test-win10 |
| `787f665` | 2026-11-24 | Marvis | Merge pull request #161 from lordon1a/fix/pipeline-staging-cleanup |
| `13ac0e6` | 2026-11-24 | Marvis | Merge pull request #167 from Maraco91/xmem/zero-length-unmap |
| `3dd37e6` | 2026-11-24 | Marvis | Merge pull request #198 from Dufftwt/fix/crash-report-instruction-bytes |
| `27c960b` | 2026-11-24 | Marvis | Merge pull request #204 from Dufftwt/fix/crash-report-fault-memory |
| `6af0fae` | 2026-11-24 | Marvis | Merge pull request #205 from asdaasdsaddf/fix/null-uav-dimension |
| `27f9b76` | 2026-11-24 | Marvis | Merge pull request #209 from Maraco91/xgamesave/empty-configuration-id |
| `7c8cecb` | 2026-11-24 | Marvis | Merge pull request #229 from Dufftwt/fix/gamertag-modern-suffix-empty |
| `9e44f38` | 2026-11-24 | Marvis | Merge pull request #239 from Maraco91/xtaskqueue/terminate-ends-dispatch |
| `56d5032` | 2026-11-24 | Marvis | Merge pull request #245 from Dufftwt/fix/import-trap-in-crash-log |
| `88b6658` | 2026-11-24 | Marvis | Merge pull request #249 from Maraco91/gpu/get-device-newer-levels |
| `89e1031` | 2026-11-24 | Marvis | Merge pull request #260 from Dufftwt/fix/f32-to-half-rounding-carry |
| `db89843` | 2026-11-24 | Marvis | Merge pull request #276 from Dufftwt/fix/s-ttracedata-imm-opcode |
| `93c0852` | 2026-11-24 | Marvis | Merge pull request #285 from Dufftwt/fix/name-title-dll-that-fails-to-prepare |
| `5a16e66` | 2026-11-24 | Marvis | Merge pull request #313 from Abdellah-Khanani/loader-aligned-unwind |
| `edf64e2` | 2026-11-24 | Marvis | Merge pull request #336 from hophaver/fix/dynamic-descriptor-exhaustion |
| `533b7e2` | 2026-11-24 | Marvis | Merge pull request #341 from enzoduarte1/config/empty-test-titles-env |
| `2745589` | 2026-11-24 | Marvis | Merge pull request #355 from enzoduarte1/clippy/clean-workspace-lints |
| `42361b0` | 2026-11-24 | Marvis | Merge pull request #356 from FIocker/gpu/stable-drop-rank |
| `7374bfb` | 2026-11-24 | Marvis | Merge pull request #365 from FIocker/gpu/publish-initialized-descriptor-heap |
| `a7e60cf` | 2026-11-24 | r4. | Merge upstream main (42361b0) into shader/per-lane-phi |
| `40b7ed1` | 2026-11-24 | Marvis | Merge pull request #371 from marquezinii/fix/pls-path-small-buffer |
| `f67c50c` | 2026-11-24 | Marvis | Merge pull request #372 from marquezinii/fix/capture-unorm16-formats |
| `ebf3615` | 2026-11-24 | Marvis | Merge pull request #376 from LouisRiverstone/shader-cvt-norm-f16 |
| `c30fc99` | 2026-11-24 | Marvis | Merge pull request #381 from Dufftwt/fix/psv-sample-frequency |
| `faba64c` | 2026-11-24 | Marvis | Merge pull request #382 from Dufftwt/fix/stencil-ref-shader-flag |
| `2e4e280` | 2026-11-24 | Marvis | Merge pull request #384 from LouisRiverstone/shader-cvt16-saturate |
| `3c86238` | 2026-11-24 | Marvis | Merge pull request #418 from enzoduarte1/gpu/half-subnormal-output |
| `dd273df` | 2026-11-24 | Marvis | Merge pull request #419 from itsRevela/fix/swallow-keyboard-menu |
| `9f18eb4` | 2026-11-24 | Marvis | Merge pull request #423 from pedrofort/shader-perm-no-int64 |
| `f0894c2` | 2026-11-24 | Marvis | Merge pull request #429 from marquezinii/fix/xgamesave-documented-errors |
| `ce42e77` | 2026-11-24 | Marvis | Merge pull request #432 from Proudnoob4/fix/uninitialized-sgprs |
| `0ddbce8` | 2026-11-24 | Marvis | Merge pull request #433 from r4tedx/tests/serialise-valu-gpu-tests |
| `dd54de2` | 2026-11-24 | Marvis | Merge pull request #448 from Dufftwt/fix/min-max-signed-zeros |
| `fd92558` | 2026-11-24 | Marvis | Merge pull request #520 from Quat3rnion/victory-xasync-payload |
| `a4d7b16` | 2026-11-24 | Marvis | Merge pull request #565 from SAitken91/shader/conservative-depth-position |
| `204bdcc` | 2026-11-24 | Marvis | Merge pull request #611 from AdvaitPalve11/fix/config-show-data-dir |
| `9a92ccf` | 2026-11-24 | Marvis | Merge pull request #612 from KoLolX3/fix/phi-liveness-transitive |
| `6f0a10a` | 2026-11-24 | Marvis | Merge pull request #634 from SAitken91/shader/image-sample-o |
| `cf0bb43` | 2026-11-24 | Marvis | Merge pull request #637 from AlanQiang0418/codex/gfx10-packed-inline-constants |
| `b4b4110` | 2026-11-24 | Marvis | Merge pull request #654 from SantannaCarlos/fix/uav-describe |
| `16bd577` | 2026-11-24 | Marvis | Merge pull request #666 from lameloball2813-ui/fix/load-probe-target-dir |
| `6db02ea` | 2026-11-24 | Marvis | Merge pull request #671 from fuzeofficlal/fix/copyresource-cpu-texture-source |
| `22700b9` | 2026-11-24 | Marvis | Merge pull request #675 from lameloball2813-ui/docs/rustdoc-private-links |
| `8d23f07` | 2026-11-24 | Marvis | Merge pull request #680 from EatCodeSleep2/fix/wine-staging-upload-128 |
| `5cd7421` | 2026-11-24 | Marvis | Merge pull request #681 from fuzeofficlal/fix/vertex-buffer-null-unbind |
| `02f404e` | 2026-11-24 | Marvis | Merge pull request #685 from catminou4/devin/constant-color-cfg-windows |
| `a51bdfa` | 2026-11-24 | Marvis | Merge pull request #696 from catminou4/devin/formats-doc-micro-link |
| `58fed37` | 2026-11-24 | Marvis | Merge pull request #706 from Lyrnc/fix/cmp-lg-ordered |
| `2005e35` | 2026-11-24 | Marvis | Merge pull request #88 from AndyLatimer/fix/title-size-overflows |
| `c7a5f49` | 2026-11-24 | Marvis | Merge pull request #93 from TG0000/perf/bcpack-bucket-lookup |
| `8e79c80` | 2026-11-24 | Marvis | CI: clippy and tests on pull requests and main |
| `e09dbb6` | 2026-11-24 | r4. | Merge upstream main (8e79c80) into shader/per-lane-phi |
| `74a9d33` | 2026-11-24 | r4. | Shader tests: the waterfall tests take the GPU one at a time, with the VALU tests |
| `ca48fbe` | 2026-11-24 | Marvis | Merge pull request #566 from r4tedx/shader/per-lane-phi |
| `ffe310e` | 2026-11-24 | Marvis | Shaders: the console's waterfall loop over a first-lane table offset |
| `578d459` | 2026-11-24 | fxceless | victory-gta6: this fork's changes on top of VIctory main (ffe310e) |
| `6a2d9f9` | 2026-11-24 | fxceless | Shadow and trace programs translate, lint clean, ray-tracing write-up |
| `d279e69` | 2026-11-24 | ZazaJr24 | Make host pipeline waits configurable and prewarm pending requests (#1) |
| `004bf69` | 2026-11-24 | fxceless | Test fix for hosted runners, stencil diagnostics, and a stability warning |
| `e9765a7` | 2026-11-24 | fxceless | Fix the device loss introduced by the wave64 update |
| `7b0f736` | 2026-11-24 | fxceless | Shader tests: recognise the software adapter from the device's own adapter |
| `1684989` | 2026-11-24 | fxceless | Shader clean-up passes; a test that needs a card skips without one |
| `3930989` | 2026-11-24 | oddiums | Fix mesh pipeline test on unsupported GPUs (#14) |
| `90228a7` | 2026-11-25 | cxccxc | Shaders: restore parked lane-mask pairs a few instructions apart; relative VGPR reads keep parked lanes (#11) |
| `4e17478` | 2026-11-24 | fxceless | The game is no longer named in the tree |
| `9e49b8a` | 2026-11-24 | fxceless | Take the parked lane-mask change (#11) back out for now |
| `2119b0e` | 2026-11-24 | fxceless | Put the parked lane-mask change (#11) back |
| `3bff182` | 2026-11-24 | fxceless | Take the test/fx-prs branch of asdaasdsaddf's fork (as one commit) |
| `31714de` | 2026-11-24 | fxceless | Take the fix/amd-view-heap-clamp branch of triippywhale's fork (as one commit) |
| `a803c87` | 2026-11-24 | fxceless | Textures picked by index are looked at again after their upload (characters' skin and hair without colour in most lau... |
| `c96b4f2` | 2026-11-24 | fxceless | The depth bounds test works (#82); faster defaults; textures are on the card before the character bake; one host copy... |
| `310c6eb` | 2026-11-24 | fxceless | Packed textures stay in the upload queue; a texture on the card is refreshed when the card is full |
| `8dd30ab` | 2026-11-25 | fxceless | A write through one view of a page is seen at its other views; page writes counted; levels on demand (off); warm-up; ... |
| `a26859f` | 2026-11-25 | fxceless | The first room renders right: byte-copied descriptor records are mirrored, views off their copy's elements get their ... |
| `9376fd6` | 2026-11-25 | fxceless | Frame rate, processor side: a cheap fingerprint in the descriptor sync, draw-table slots left to their draws, no fail... |
| `f85101b` | 2026-11-25 | fxceless | Frame rate, second night: a submit thread of the layer's own, a copy queue for fetches, indirect draw windows made sa... |
| `a253706` | 2026-11-25 | fxceless | Frame rate, third session: a frame log and a card meter, the driver watch, a forbidden self-copy removed; draw window... |
| `0b0074e` | 2026-11-25 | fxceless | Copies in place measured and fixed, our threads on the fast cores, and a corrected comparison of draw windows |
| `895c0b8` | 2026-11-24 | Geckomacabre | textures shaders write that the title reads on the CPU are copied back into title memory |
| `04df435` | 2026-11-25 | Codex | Track the allocation and fence for each water texture readback |
| `3973179` | 2026-11-25 | fxceless | What a frame is made of, measured; parts joined and copy-back through our own view (9.6 fps in the first rooms) |
| `5557224` | 2026-11-25 | Martin Hartfelder | The card compare hashes a piece with a group of 64 threads: our passes on the card from about 35 to about 10 ms a frame |
| `465b94f` | 2026-11-25 | fxceless | Scattered indirect records clamped once per list (VICTORY_INDIRECT_GATHER=1): 10.3 fps in the first rooms |
| `ca8829f` | 2026-11-25 | fxceless | The frame rate in the corner of the picture; changed-bytes refresh of card copies (off) |
| `d510ee4` | 2026-11-25 | fxceless | README: the dips while moving, measured; two attempts on the title's page mapping, both taken back out |
| `dd3a14b` | 2026-11-25 | fxceless | README: a frame capture built into the layer, copies of title memory in the card's own memory (fewer long frames in t... |
| `79cfdd9` | 2026-11-25 | fxceless | A frame capture built into the layer (VICTORY_SHOT), windows outside buffers in the card's memory as one buffer in a ... |
| `085ee7c` | 2026-11-25 | fxceless | README: the frame rate in the status header brought up to the measured runs |
| `f5c03f3` | 2026-11-25 | fxceless | Merge pull request #999 (cxy0072019-sketch): textures shaders write that the title reads on the CPU are copied back i... |
| `c36971b` | 2026-11-25 | fxceless | Small buffers' copies in the card's memory (VICTORY_CARD_BUFFERS_MIN_KB), changed_runs by masks, scalar buffer loads ... |
| `9c184b3` | 2026-11-25 | fxceless | README: with the water read-back on (pull request #999) the tester got past the sinking boat; the run later lost the ... |
| `44bff61` | 2026-11-25 | fxceless | A scrubbed diagnostic log (timing run with breadcrumbs), VICTORY_ABSENT_ONCE, and the README for runs 276 to 278 |
| `d827419` | 2026-11-25 | fxceless | README: run 279, the set that plays best so far (hash length on, the water right); what the unmapped-block notes answ... |
| `1391409` | 2026-11-25 | fxceless | Merge pull request #1014 (martindevbr): the card compare hashes a piece with a group of 64 threads |
| `6d20eed` | 2026-11-25 | fxceless | The 64-thread hash of pull request #1014 made opt-in (VICTORY_CARD_HASH_WIDE=1); changed card copies from a list (VIC... |

---

## L. Limitacoes desta analise

1. **Sem rede.** Nao consultei o GitHub: corpos de PR/issue, revisoes e comentarios nao foram lidos. Numeros de PR (#999, #1014, #220, #115, #1122, #747, #648, #670, #82, #11, #14, #1) aparecem apenas como sao citados nos assuntos/mensagens deste repositorio. O conteudo dos PRs originais e **NAO VERIFICADO**.
2. **Sem execucao.** Nao compilei nem rodei nada: e um alvo Windows/x86-64 com GPU e um titulo comercial ausente. Todos os numeros de fps, ms, MB e GB sao **relatos do autor** (README e mensagens), nao medicoes minhas. Onde os numeros vem de diff ou de constante no codigo, isso esta indicado.
3. **Sem arquivos de jogo.** Nada foi validado contra o comportamento real do titulo. "Barco afunda", "rosto preto", "paredes ausentes" sao sintomas relatados.
4. **Identidade do autor.** A ligacao `hyphennn` (prompt) <-> `fxceless` (GitHub) <-> build privado lider do PortWatch <-> ZETTROX/Uilian **nao foi verificada**. O unico dado verificavel e o autor dos commits e do README.
5. **Placar PortWatch (59,4%).** Nao ha, neste repositorio, qualquer artefato de benchmark, submissao ou identificador que ligue o codigo aquele numero. **NAO VERIFICADO.**
6. **Commits upstream nao auditados.** Examinei em profundidade ~12 dos 213 commits herdados, escolhidos por relevancia ao delta do fork. Defeitos e correcoes do upstream fora dessa amostra nao estao catalogados - eles pertencem a analise do repositorio base.
7. **Diffs truncados.** O pacote de evidencias (`_evidencias/victory-fork_evid.md`) traz ate 80 linhas de diff por commit; para os casos em que a conclusao dependia de codigo alem desse limite, fui ao repositorio (`git show HEAD:<arquivo>`), mas nao li integralmente os arquivos maiores: `crates/victory-shader/src/emit/lower.rs` (11.697 linhas), `crates/victory-gpu/src/host.rs` (8.998), `crates/victory-gpu/src/shadow.rs` (4.547) e `crates/victory-gpu/src/draw.rs` (3.648).
8. **Auditoria dos 219 switches incompleta.** Verifiquei defaults e semantica de ~30 switches, escolhidos por risco ou por citacao no README. Nao ha garantia de que nenhum outro switch tenha documentacao divergente do codigo, nem de que todos os que alteram a emissao estejam na chave do cache (D-02, risco residual declarado).
9. **Causas nao confirmadas.** As perdas de device (D-16), a agua preta (D-12) e a quebra de streaming por view longa (D-15) permanecem sem causa estabelecida - nem pelo autor, nem por mim. Nao apresentei nenhuma delas como resolvida.
10. **Data da analise.** 2026-11-25. O repositorio tem 29 horas de vida no momento da leitura e commits a cada poucas horas; qualquer conclusao sobre "estado atual" envelhece rapido. HEAD analisado: `6d20eed`.

## M. Atualizacao de teste posterior (2026-11-27)

Um teste local posterior do pré-aquecimento de pipelines proposto no PR #1226
avançou até o prólogo sem reproduzir o watchdog durante a janela observada. A
execução ainda teve desempenho muito baixo e draws descartados, portanto não
confirma uma correção geral. Os detalhes sanitizados, método e limitações estão
em [`docs/relatorios/prebuild-pipelines-pr1226-2026-11-27.md`](../relatorios/prebuild-pipelines-pr1226-2026-11-27.md).
