# VItoriaBr - analise de DeepSeek V4.1 Flash

## A. Identificacao

| Campo | Valor |
|---|---|
| Nome | VItoriaBr |
| URL | https://github.com/uiliannn/VItoriaBr |
| Natureza | Fork do Marvisak/VIctory (camada de execucao Xbox Series X\|S em Windows), com integracao declarada de `fxceless/victory-fork`, `Belobug/VIctory-Belobug-AMD`, `myokex/victory-amd`, `DexNom/VIctory` e `triippywhale` (README.md, linhas 9 e 200) |
| Branch analisada | `main` (unica branch local; `origin/HEAD -> origin/main`) |
| Remoto | `origin https://github.com/uiliannn/VItoriaBr.git` |
| Commit 0 (raiz, herdado da base) | `f4504f6413ce4509604dad66cdd689000a0204dc` — 2026-11-23 01:15:26 +0100 — Marvis — "Initial commit" |
| Primeiro commit proprio do fork | `279c90a15fe03da5b5e04e7f1b23de13356e6500` — 2026-11-24 07:05:45 -0300 — uiliannn — "gpu: track Shadow buffer D3D12 resource states across lists" (o commit seguinte, `6132a9e` no mesmo segundo, e o rebranding "VItoriaBr" + README bilingue) |
| Ultimo commit | `b6e61ab6311a6eed6c4c138a5cf6fd88339acba7` — 2026-11-26 02:02:18 +0300 — NetherySiloX — "GPU: mirror placed 3D texture UAVs and preserve their aliases (#72)" |
| Periodo coberto | 2026-11-23 01:15 (+0100) a 2026-11-26 02:02 (+0300) — 3 dias e 25 horas de calendario; 4 dias com commit |
| Commits em HEAD | 373 (373 em todas as refs; 107 merges; 218 commits na cadeia de primeiro pai) |
| Arquivos versionados | 197; ~135.154 linhas de Rust no checkout; 9 crates |
| Autores | 73 nomes distintos; concentracao: uiliannn 92, Marvis 102 (87 + 15 com e-mail `noreply.invalid`), fxceless 27, Duff 12 |
| Commits por dia | 2026-11-23: 48 · 2026-11-24: 230 · 2026-11-25: 94 · 2026-11-26: 1 |
| Modelo | DeepSeek V4.1 Flash (opencode-go), via OpenCode |
| Data da analise | 2026-11-25 |

**Advertencia metodologica decisiva (herdada de `_fatos/FORK_DIVERGENCIA.md`):** o HEAD do Marvisak *nao* e ancestral deste fork; `git merge-base` entre as linhas falha. Todos os cinco forks comparados compartilham apenas a raiz `f4504f6` e divergem imediatamente depois (historico reescrito). Logo:

- **nenhuma comparacao por hash de commit entre repositorios e valida**; a unica comparacao confiavel e por conteudo de blob;
- **nao existe cherry-pick possivel** entre estes repositorios (consequencia direta para o repositorio unificado, ver secao I);
- dentro deste repositorio, porem, `git show`/`git blame`/`git log -S` sao confiaveis, e foi nisso que esta analise se baseou.

Numeros verificados por comando nesta sessao:

```
git rev-list --count HEAD          -> 373
git rev-list --merges --count HEAD -> 107
git rev-list --first-parent --count HEAD -> 218
git ls-files | wc -l               -> 197
git log --format="%an" | sort -u   -> 73 nomes
```

**Sobre os numeros de PR.** Este repositorio mistura tres espacos de numeracao diferentes no mesmo historico:

1. PRs do **VIctory upstream** (ex.: `58fed37 Merge pull request #706 from Lyrnc/fix/cmp-lg-ordered`, `2005e35 Merge pull request #88 from AndyLatimer/fix/title-size-overflows`);
2. PRs do **fxceless/victory-fork** (ex.: `9ceb7e7 ... (#37)`, `448416c ... (#79)`, `b6e61ab ... (#72)` — o sujeito carrega o numero do repo onde o squash aconteceu);
3. PRs **deste fork**, que aparecem como branches locais (`d396d11 Merge branch 'pr-7'`, `ac14e18 'pr-10'`, `7838d65 'pr-11'`, `7e6670b 'pr-12'`, `f06ee8b 'pr-15'`) ou em mensagem de merge (`ef30575 merge: PR #6 (@andysilva86)`, `8a85222 merge: PR #3 (@andysilva86)`).

Atribuir "o que o PR #N resolveu" sem essa distincao produz historia falsa. Nesta analise, cada PR citado vem acompanhado do hash do commit local que o materializou.

## B. Resumo tecnico

**Proposito.** Rodar um titulo Xbox Series X|S (build 1.0.2.1) em Windows a partir dos arquivos do proprio usuario: o codigo x86-64 do titulo executa **inalterado** no processo do host, e a camada fornece a plataforma Xbox/GDK por baixo e traduz dados e shaders (D3D12.X/GFX10 -> D3D12/DXIL). README.md:5-7.

**Arquitetura (9 crates, ~135k linhas de Rust).**

| Crate | Papel |
|---|---|
| `victory-core` | configuracao (`victory.toml`), politica de assets |
| `victory-pe` | leitor PE: imports, exports, relocations, TLS, layout |
| `victory-loader` | mapeamento do executavel do titulo, thunks de import, emulacao de instrucoes so-AMD, crash log, `console_insn`/`console_insn_decode` |
| `victory-hle` | modulos GDK emulados: kernel, xmem, dstorage, xgamesave, xuser, xg, d3d12x, audio/XMA, display, gameinput, xnetworking, xstore, xsystem... |
| `victory-shader` | tradutor GFX10 -> IR -> DXIL (SM 6.6): decode, estrutura de controle, SSA, recursos, lane/wave, emisor, passes de limpeza, testes diferenciais |
| `victory-gpu` | camada D3D12 do host: sombras/copias, descriptors, texturas, pipelines, indirect, fences, filas, captura, instrumentacao |
| `victory-formats` | texturas do console (tiling/XOR swizzle), BCPack, XMA |
| `victory-game` | formatos do titulo (SGD5/SGX5, pacotes) |
| `victory-pipeline` | cache enderecado por conteudo (BLAKE3) usado por shaders e PSOs |

**Modulos que concentram a historia** (por numero de alteracoes): `crates/victory-shader/src/emit/lower.rs` (67), `README.md` (48), `crates/victory-gpu/src/host.rs` (35), `draw.rs` (35), `descriptor/host.rs` (31), `emit/mod.rs` (31), `shadow.rs` (20), `indirect.rs` (16), `texture.rs` (17).

**Estado atual, em uma frase.** O titulo inicia, roda menus e video a 30 fps, carrega o mundo e renderiza a primeira sala e o mundo aberto a **~1,7 a 6 fps** numa RTX 5060 Ti 16 GB, com cobertura de shaders declarada de **25.741/25.758 DXIL (99,9%)** e **25.344 PSOs (98,5%)**; iluminacao indireta por ray tracing **nao funciona** (nada e iluminado por RT), o que e compensado por um *fallback* de luz ambiente sombreada por profundidade portado de outro fork.

**O que este fork e, tecnicamente.** Tres coisas, em ordem de importancia:

1. **Um integrador agressivo.** Absorveu, em 48 horas, o trabalho de pelo menos cinco forks (fxceless, Belobug, myokex, DexNom, triippywhale) e dezenas de PRs do upstream, quase sempre por *squash-import* de um commit gigante (ex.: `578d459`, 61 arquivos, +32.334 linhas).
2. **Um laboratorio de desempenho com medicao declarada.** A maior parte dos commits de 25/11 traz numeros antes/depois (`70-220 ms -> ~3 ms` no passe de descriptors, `23-29 ms -> ~5 ms` nas listas de hash do cartao, `10.3 fps`, `9.6 fps`, `~10%` de frame). E, raro, **retrata numeros errados** (`0b0074e` retira a comparacao 6,0 -> 8,9 fps).
3. **Um conjunto de mecanismos proprios** que nao existem na base: espelho de bytes de descriptor heap com *fingerprint* barato, *merge* de escritas da CPU **no cartao** por mascara de bits, hash de 64 KiB com grupo de 64 threads, tabela `SLOT_VIEWS` de 16 bytes por slot, HUD desenhado por fonte bitmap propria e captura de frame embutida (`shot.rs`, 1.016 linhas).

## C. Linha do tempo

### Fase 0 — a base herdada (2026-11-23, madrugada)

`f4504f6` (01:15, Marvis) e o "Initial commit" da base. Nos tres commits seguintes a base ja e um emulador funcional: `12caefe` (campos de operando GFX10, SP3, wave size de FSSM), `754619f` ("Run Minecraft: runtime HLE, D3D12.X device on the host GPU, shader IR"), `2849477` ("Run Grand Theft Auto VI from first boot into the world") e `015d45b` (README com status, tabela de subsistemas, switches).

**Nada disso e autoria do fork.** E o ponto de partida: uma camada que ja traduz shaders e ja entra no mundo, com graficos incompletos.

### Fase 1 — ingestao de PRs do upstream (2026-11-23, 48 commits)

O dia 23 e quase inteiro de *intake*: correcoes pequenas e independentes entrando por PR, com o mantenedor apenas mergeando.

- Loader/TLS: `c21208c` (ponteiro de callback TLS fora das base relocations) -> `e8f3b1c` (PR #4).
- Instrucoes so-AMD em CPU Intel: `934e67f` -> `d348b9a` (PR #10); `71cfe84` (clwb antes de Ice Lake), `c4e915f` (loop de write-back em uma excecao), `0c91c03`/`f00caec` (testes).
- Formatos: `005ad49` (XMA: parar em frame de comprimento zero em vez de laco infinito — **o primeiro defeito desta historia, e ele volta em D-03**), `e6a2c54` (extents > 16384 texels), `e158d95` (overflow de offsets).
- Shaders: `36eb0b4` (med3 com NaN, fract < 1.0, bitfields com sinal acima do bit 31), `0b499d7` (seno/cosseno em voltas), `2e5001c` (fract(NaN) na NVIDIA), `5fbea47` (canal de entrada sem underflow), `ea24ea1` (mascara de lane que estoura a pilha).
- GPU: `8c26061` ("o mundo fica de pe; execucao indireta atras de switch"), `3827767` (tabelas acima de um heap de views capado), `35b054e` (pipelines fora da render thread e cacheados; indireto ligado), `4c1766c` (fault de bloco guardado em corrida), `0b607b1` (WARP verificado como software).
- Infra: `7e42fef` (falha de commit-limit no crashes.log), `d4ed2de` (watchdog de saida antes das outras threads).

### Fase 2 — o fork assume (2026-11-24, 230 commits; o dia mais denso)

**2a. Madrugada: mais intake.** Onda de merges de PRs do upstream: `#14`, `#15`, `#19`, `#21`, `#28`, `#30`, `#37`, `#41`, `#43`, `#45`, `#46`, `#50`, `#51`, `#53`, `#56`-`#61`, `#66`, `#70`, `#75`, `#77`, `#79`-`#81`, `#87`, `#93`, `#104`, `#111`, `#117`, `#129`, `#140`, `#152`, `#161`, `#167`, `#198`, `#204`, `#205`, `#209`, `#229`, `#239`, `#245`, `#249`, `#260`, `#276`, `#285`, `#313`, `#336`, `#341`, `#355`, `#356`, `#365`, `#371`, `#372`, `#376`, `#381`, `#382`, `#384`, `#418`, `#419`, `#423`, `#429`, `#432`, `#433`, `#448`, `#520`, `#565`, `#566`, `#611`, `#612`, `#634`, `#637`, `#654`, `#666`, `#671`, `#675`, `#680`, `#681`, `#685`, `#696`, `#706`.

**2b. 07:05 (-0300): o fork comeca.** `279c90a` (estado de recurso D3D12 de `Shadow` buffers entre listas) e `6132a9e` (rebranding + README bilingue PT/EN com progresso na RTX 5060 Ti). Em seguida `dbff821` (merge do upstream) e a primeira onda propria de shaders: `2471a62` (MIMG `a16`, `_b`, `_cl`/`_o`, LDS byte/half), `5943771`/`c0b60a0` (reconciliacao de interpolacao por *row* de assinatura), `a65d054` (reciclagem de slots dinamicos, `MIN_DYNAMIC` 2^17), `77b4b4c` (cubemap `image_sample_d_cl`), `1ca533d`/`8a75883`/`6f99f75` (anti-TDR: uma lista por fence, `DispatchMesh` fora de faixa zerado, budget de grupos), `a13a719` (bounds check de LDS — **introduz D-12**), `911267c` (setMeshOutputCounts capado), `1ece3a6`/`808a209` (barreira so-fence).

**2c. O import decisivo.** `578d459` — "victory-gta6: this fork's changes on top of VIctory main (ffe310e)" — **61 arquivos, +32.334 linhas em um unico commit**. Traz relay de submissao, copia de volta em thread propria, `shadow.rs`, `tiles.rs`, `spill.rs`, `structure.rs`, o `LOG2_E` errado (D-16), `copy_back_done` (D-02) e quase toda a base de desempenho do fork. **Este e o evento que define a arqueologia do repositorio: a partir daqui nao ha mais granularidade historica; ha um antes e um depois.**

**2d. Shaders de ray tracing e adocao de outros forks.** `6a2d9f9` (programas de sombra e traco traduzem; write-up de ray tracing; **introduz a forma wave64 que causara D-20**), `3bff182` (branch `test/fx-prs` de asdaasdsaddf como um commit), `31714de` (branch `fix/amd-view-heap-clamp` de triippywhale), `e9765a7` (**corrige o device loss introduzido pelo wave64** — D-20).

**2e. Primeira sala e memoria.** `a803c87` (textura indexada e reexaminada apos upload — **D-08**), `c96b4f2` (depth bounds test de #82 + defaults mais rapidos: relay stage 2, wide copy-back; primeira sala iluminada pela lampada), `8dd30ab` (guard sobre **todas** as views de uma pagina fisica — **D-09**; levels on demand; `victory translate --warm all`), `32d7848` (segura `SHADER_HEAPS` durante o espelho — **D-06**), `310c6eb` (regra `Rule::Start` para spans de textura — **D-07**), `91cd9c9`->`aa85bb0` (corrida do relay — **D-02**), `b62bbc0` (tabela `SLOT_VIEWS` — **D-13**), `5603b45` (stride de 16 bytes — **D-13**), `b37344f`/`009856f` (varredura de wraparound de descriptors — **D-04**), `d52d84c` (CI Linux portatil).

**2f. O pipeline de desempenho do fxceless.** `8e46dce` (merge do `victory-gta6`: rendering de mundo, relay/VRAM, otimizador DXIL), depois `a26859f` (espelho de registros de descriptor copiados como bytes; views fora dos elementos da copia; espaco no cartao para o primeiro frame do mundo; levels on demand no cartao), `9376fd6` (**fingerprint barato no sync de descriptors: 70-220 ms -> ~3 ms**).

### Fase 3 — a noite de medicao (2026-11-25, 94 commits)

**3a. O sync do fxceless, commit a commit.** `4f9a19d`, `b401954`, `dfd22d1`, `4e038bf`: submit thread proprio, copy queue, draw windows, copias in-place, threads nos nucleos rapidos, `framelog`, `cardmeter`, `drvwatch`, `profile.rs` (2.961 linhas), `fpsbox`, `shot.rs` (1.016 linhas), `indirect/bench.rs`. Junto vem a serie de defeitos de desempenho com numero: `f85101b` (3,5-4 -> 6-7 fps), `a253706` (frame log; **retira a copia proibida de textura sobre si mesma**, 402 em 5 min), `0b0074e` (**retrata a comparacao 6,0 -> 8,9 fps**; copias in-place corrigidas; device loss no mundo aberto, causa desconhecida), `3973179` (9,6 fps), `465b94f` (**`VICTORY_INDIRECT_GATHER`: 10,3 fps**), `79cfdd9` (`VICTORY_SHOT`, `VICTORY_WINDOW_UPLOAD`).

**3b. Mecanismos proprios de cartao.** `713a095` (hash de pecas com 64 threads: 23-29 -> ~5 ms), `9ceb7e7` (**merge de bytes da CPU no cartao por mascara: ~10% de frame**), `213fa54` (hash de tabela antes de comparar registros), `e140f41` (HUD de 2 linhas), `5a84eb4` (custo de CPU de `heap_buffers`: 87-110 ms/frame), `faca57e` (orcamento de tempo/bytes em `heap_textures` contra o watchdog de 30 s).

**3c. Portes com regressao imediata.** `89434b7` (recupera texturas com mips interiores esparsos via `VirtualQuery`) -> **`c931d12`, 22 minutos depois, reverte** ("752 ms/frame stall, 0 hits"). No mesmo dia `a290636` (corrige o LOD com log2 duplo — D-16), `d5d9b7c` (rastreamento preciso de escrita em memoria guest, `VSH3`), `8c0758d`/`5668b8b`/`081a89d` (portes de `myokex/victory-amd`), `448416c` (`v_permlane16` — D-14), `75aa531` (`v_dot8_i32_i4` — D-15), `bfa5f49` (XMA — D-03), `d1da85a` (XGameSave — D-18), `708b450` (loader lock — D-19), `defaac8` (`cargo xtask ci`), `ddd5e6c` (testes FP16 wave64), `2b33e08`/`e7f9af5`/`ca4759f` (Clipy 1.98 e CI).

### Fase 4 — o ultimo commit (2026-11-26, 1 commit)

`b6e61ab` — "GPU: mirror placed 3D texture UAVs and preserve their aliases (#72)" — squash de PR com `cherry picked from` de dois commits (`d5c277e`, `84f80e2`), co-autoria de Gregory Bonzon e ryotdev. Torna views UAV de textura *placed* (2D, array, 3D) espelhadas no host e preserva aliases `TARGETS_AT` entre formatos. E o unico commit do dia 26: a arvore para aqui.

**Leitura da linha do tempo.** Nao e a historia de um projeto; e a historia de **uma sessao de 3 dias** em que um mantenedor absorveu cinco forks e cem PRs, mediu o resultado com instrumentacao propria, e escreveu no README os numeros — inclusive os que teve de retirar.

## D. Catalogo de defeitos

> Convencao: **CONFIRMADA** = codigo/diff verificado nesta sessao; **PROVAVEL** = evidencia forte mas indireta (mensagem + codigo coerente, sem reproducao); **HIPOTESE** = inferencia a partir de padroes; **NAO VERIFICADA** = sem evidencia local.
> Nenhum numero de medicao foi reproduzido: numeros entre aspas vem de mensagens de commit/README e estao marcados como tal.

### D-01 — Caminhada `VirtualQuery` em `mapped_bytes` derruba o frame para 752 ms

- **Descricao/sintomas.** O passe de texturas do heap passou a gastar **752 ms por frame** ("5 s frame stalls") na RTX 5060 Ti; a instrumentacao mostrou **0 texturas** recuperadas pela nova logica.
- **Causa raiz.** `mapped_bytes` passou a caminhar regioes nao commitadas com `VirtualQuery` quando nem a primeira nem a ultima pagina estao commitadas, para "recuperar texturas que so tem mips interiores residentes". `VirtualQuery` e uma chamada por regiao sobre o espaco de endereco do titulo; o passe pergunta varias vezes por slot e roda ~3.000 slots por frame quando o cartao esta cheio.
- **Commit que introduziu.** `89434b7` ("gpu: recover sparse interior-mip textures, coalesce remaps, and prioritize depth/compute pipelines (DexNom 921d8d3)").
- **Tentativas de correcao.** Nenhuma intermediaria: **reversao direta** em `c931d12`, 22 minutos depois, mantendo o resto do commit (coalescencia de remaps, prioridade de pipelines).
- **Correcao definitiva.** `c931d12` — volta ao *early-out* de duas consultas (`!committed(at,1) && !committed(end-1,1) -> 0`) e deixa a caminhada completa apenas atras de `VICTORY_COMMITTED_QUERY=1`.
- **Arquivos.** `crates/victory-gpu/src/texture.rs`.
- **Evidencia.**
  - `git show --stat c931d12` -> `crates/victory-gpu/src/texture.rs | 49 ++++++-------  (10 insertions, 39 deletions)`.
  - Mensagem de `c931d12`: *"Live RTX 5060 Ti testing on v13 showed 0 textures without their first page while walking uncommitted ranges via VirtualQuery spiked the heap texture pass to 752 ms/frame (5 s frame stalls)."*
  - Estado no HEAD: `texture.rs:1587` `if !committed(at, 1) && !committed((end - 1) as *const u8, 1) { return 0; }`, e o comentario do doc: *"With neither the first nor the last page committed the answer is none without a look at the pages between (two queries for the slots whose texture is away, which are asked again and again)."*
- **Estado atual.** Revertido. A **limitacao original permanece**: textura cujo unico conteudo residente sao mips interiores nao e vista como parcialmente presente.
- **Confianca.** Codigo e reversao: **CONFIRMADA**. Os 752 ms/5 s: **PROVAVEL** (relato do autor no commit; nao reproduzido).

### D-02 — Corrida no `relay_land`: ranges liberados antes de `line.written` + `notify_all` -> 0xc0000005

- **Descricao/sintomas.** Com `VICTORY_RELAY=2` (que e o **padrao**, ver `host.rs:2338`), o titulo crashava no carregamento com **0xc0000005** (access violation), porque a memoria do titulo recebia bytes velhos ou ja desmapeados.
- **Causa raiz.** `Shadow::copy_back_done()` devolvia os ranges *pendentes* a lista de livres **antes** de `line.written` deixar de nomea-los e antes de `LINE_MOVED`/`LINE_LANDED` acordarem os waiters. Um refresh concorrente podia ver o range como livre enquanto o *merge* ainda escrevia memoria do titulo.
- **Commit que introduziu.** `578d459` (o import de 32k linhas; `copy_back_done` e `relay_land` aparecem pela primeira vez aqui — `git log -S"copy_back_done" --reverse`).
- **Tentativas de correcao.**
  1. `91cd9c9` — move `copy_back_done` para depois de `notify_all` e **deduplica** ranges entre flights.
  2. `aa85bb0` — mesma mensagem, diff diferente: **remove a deduplicacao** ("without deduplicating across flights: `Shadow::copy_back_done` decrements `Flights` per flight!").
- **Correcao definitiva.** `aa85bb0` (o `91cd9c9` introduziu um segundo defeito — ver H-02).
- **Arquivos.** `crates/victory-gpu/src/host.rs` (`relay_land`), `crates/victory-gpu/src/shadow.rs` (contagem de `Flights`).
- **Evidencia.**
  - `git show aa85bb0` -> `+ // Collect each flight's back ranges before consuming `items` (without` / `+ // deduplicating across flights: `Shadow::copy_back_done` decrements` / `+ // `Flights` per flight!)`.
  - HEAD `host.rs:5425-5431`: comentario *"Before this point a concurrent refresh could see the range as free while the merge was still writing title memory, leaving the title with stale or unmapped bytes (the 0xc0000005 crashes during loading with VICTORY_RELAY=2)"* seguido de `for (shadow, ranges) in &backs { shadow.copy_back_done(ranges); }`, que e a ultima coisa da funcao, depois de `LINE_MOVED.notify_all(); LINE_LANDED.notify_all();`.
- **Estado atual.** Corrigido, com o padrao de publicacao na ordem "atualiza estado -> acorda -> libera recurso".
- **Confianca.** **CONFIRMADA** (codigo + diff + comentario no HEAD).

### D-03 — XMA: laco infinito com contagem de bits alem dos dados

- **Descricao/sintomas.** Decodificar um frame XMA com `bits` maior que o buffer travava o processo (laco sem fim), e o caso so aparecia com dados truncados/corrompidos.
- **Causa raiz.** `Decoder::decode` usava o numero de bits do chamador como fim do frame. Passado o fim dos dados o leitor devolve zeros; um cabecalho de tile estereo lido como zero nao atribui o subframe a canal nenhum, entao o cursor nunca avanca — e, como a contagem era maior que os dados, nunca havia overread que encerrasse o laco.
- **Commit que introduziu.** O decodificador veio do upstream antes da raiz analisada; o sintoma de "frame de comprimento zero" ja tinha sido tratado em `005ad49` ("xma: stop at a zero-length frame instead of looping forever"). A **variante por contagem de bits** foi fechada depois.
- **Tentativas de correcao.** `005ad49` (frame de comprimento zero) e `467f168` ("an XMA read block past the ring's end wraps instead of filling forever"), `d0d0d23` (continuacao), `257d8bb` (avanco de leitura).
- **Correcao definitiva.** `bfa5f49` — `end: bits.min(data.len().saturating_mul(8))`, com testes de ruido deterministico (xorshift64, sementes fixas) que limitam toda caminhada.
- **Arquivos.** `crates/victory-formats/src/xma.rs`.
- **Evidencia.**
  - `git show bfa5f49` -> `- end: bits,` / `+ // Past the data there are only zeros to read...` / `+ end: bits.min(data.len().saturating_mul(8)),`.
  - Mensagem: *"The bit-count test hangs without the fix; the others pass on main."*
  - HEAD `xma.rs:509` contem a linha.
- **Estado atual.** Corrigido e coberto por teste que falha (trava) sem a correcao.
- **Confianca.** **CONFIRMADA** (codigo); a propriedade "o teste trava sem a correcao" e **PROVAVEL** (afirmada no commit, nao executada aqui).

### D-04 — Varredura de wraparound dos descritores dinamicos truncada: exaustao falsa

- **Descricao/sintomas.** Uma alocacao de N slots contiguos que **atravessa o cursor** de busca nao era encontrada; o heap reportava falta de espaco com espaco livre, e draws/dispatches caiam.
- **Causa raiz.** Duas camadas: (a) a busca varria `next..dynamic` e depois `0..next`, sem estender o segundo trecho por `N-1`; (b) mesmo depois de estendida, o laco era truncado por `starts.take(if enough { DYNAMIC } else { 0 })`, cortando a extensao.
- **Commit que introduziu.** O alocador circular veio do upstream; o truncamento por `DYNAMIC` esta no codigo de `HostHeap::dynamic` herdado.
- **Tentativas de correcao.**
  1. `a91ef70` (PR externo) — estende o segundo trecho (`second_end = cursor + n - 1`) e adiciona teste; **nao bastou**, porque o `take(DYNAMIC)` continuava truncando.
  2. `b37344f` — conserta o **fixture** do teste (enche o pool, posiciona o cursor em 12, libera 10..14) e cita `fixes #14 e #16`.
  3. Dois PRs tentaram um alocador circular novo e **fecharam sem merge** (docs/performance-architecture.md: *"#8 and #9 closed unmerged"*).
- **Correcao definitiva.** `009856f` — `for at in starts.take(if enough { usize::MAX } else { 0 })`.
- **Arquivos.** `crates/victory-gpu/src/descriptor/host.rs`.
- **Evidencia.**
  - `git show 009856f` -> `- for at in starts.take(if enough { DYNAMIC as usize } else { 0 })` / `+ for at in starts.take(if enough { usize::MAX } else { 0 })`; mensagem: *"cutting off the n - 1 wraparound extension past the cursor and failing dynamic_allocation_finds_contiguous_range_crossing_search_cursor (fixes #16)"*.
  - HEAD `descriptor/host.rs:1138` contem `usize::MAX`.
- **Estado atual.** Corrigido. O teste existe e roda com GPU; sem GPU ele retorna cedo (`let Some(heap) = ... else { return; }`).
- **Confianca.** **CONFIRMADA**.

### D-05 — Heap de descriptors esgotado lia o descritor do titulo como CBV -> remocao do device na AMD

- **Descricao/sintomas.** Device removal apos ~60 s, com faults em enderecos selvagens (`0x7000037xxxxx`, `0x8000037xxxxx`, `0x33c4000`), pipeline diferente a cada execucao, em RX 9070 XT (issue **#59**, citada na mensagem).
- **Causa raiz.** Quando `HostHeap::dynamic` nao tinha slots, `State::tables` pulava a tabela nova e o `user data` do estagio caia no offset da tabela **do titulo**. Tabelas de constant buffer sao sempre feitas de novo e nunca espelhadas — o shader lia o que estivesse naquele slot do heap do host (textura, ou nulo) como constant buffer; na AMD isso vira um endereco base selvagem.
- **Commit que introduziu.** O caminho de fallback e anterior a `3c81420` (nao isolavel neste historico por causa do import `578d459`).
- **Tentativas de correcao.** `edf64e2`/`3c81420` (PR #336, hophaver): **a tabela que nao pode ser feita faz o draw/dispatch ser descartado** em vez de cair no offset do titulo, e o range dinamico cresce de 2^16 para 2^18. Depois: `e8d3b64` (2^18 so onde o heap cabe), `a65d054` (reciclagem apos `execute` sincrono; `MIN_DYNAMIC` 2^17), `bd33acc` (2^18 em heaps capados), `a91ef70`/`009856f` (D-04).
- **Correcao definitiva.** `3c81420` (descartar em vez de ler lixo), com os ajustes de capacidade acima.
- **Arquivos.** `crates/victory-gpu/src/descriptor/host.rs`, `draw.rs`, `list.rs`, `queue.rs`.
- **Evidencia.**
  - `git log -1 --format=%B 3c81420`: *"Constant buffer tables are always made fresh and never mirrored, so the shader read whatever host descriptor sat at those slots as a constant buffer; on AMD a texture or null descriptor read as a buffer gives a wild base address and the device is removed (#59: faults at 0x7000037xxxxx ...)"*.
  - README.md:64 declara *"zerando os 16.384+ draws/dispatches descartados"* com `MIN_DYNAMIC` 2^18.
- **Estado atual.** Corrigido na politica (descartar e logar), com capacidade muito maior. **Troca consciente:** em vez de corromper, perde-se trabalho — e o descarte nao e zero (ver G-06).
- **Confianca.** **CONFIRMADA** (codigo + mensagem); a reproducao em AMD: **PROVAVEL**.

### D-06 — `SHADER_HEAPS`: use-after-free no espelho e, depois, 48,5 ms/frame de contencao

- **Descricao/sintomas.** (a) Corrida entre `mirror()` e `Heap::drop`: o lock era solto dentro de `shader_slot()` e o `&Heap` era usado depois — um `Heap::drop` concorrente podia liberar o heap em uso. (b) Depois de corrigido, o passe `sync_heaps` segurava o mutex por 15 ms e as seis threads de gravacao do titulo esperavam **48,5 ms por frame** somados.
- **Causa raiz.** (a) tempo de vida do recurso amarrado a um lock que nao era mantido; (b) lock unico (mutex) para leitura em caminho quente.
- **Commit que introduziu.** (a) o codigo original do heap de descriptors (pre-`32d7848`); (b) a correcao de (a).
- **Tentativas de correcao.**
  1. `32d7848` (Manuel Perez) — mantem `SHADER_HEAPS` locked durante todo o `mirror()`/`make_waiting()` (corrige o UAF, piora a contencao).
  2. `a253706`/`dfd22d1` — troca por `RwLock` + um mutex `SYNCING` para serializar um sync por vez; leitores deixam de esperar uns pelos outros.
- **Correcao definitiva.** `a253706` (RwLock), preservada nos merges seguintes.
- **Arquivos.** `crates/victory-gpu/src/descriptor.rs`.
- **Evidencia.**
  - `git show 32d7848`: `- fn shader_slot<'a>(handle: u64) -> Option<(&'a Heap, u32)>` -> `+ fn mirror(handle: u64)` com *"Keeps SHADER_HEAPS locked for the duration so Heap::drop cannot free the heap concurrently"*.
  - `git show dfd22d1 -- descriptor.rs`: *"It was a mutex, and sync_heaps holds it for its whole pass (15 ms a frame on the submitting thread): the profile of run 225 has the title's six recording threads waiting 48.5 ms a frame in shader_slot for it"*.
  - HEAD `descriptor.rs:136`: `static SHADER_HEAPS: std::sync::RwLock<Vec<(u64, u64, usize)>>`.
- **Estado atual.** Corrigido. Observacao importante para reuso: a invariante "nada que segura o lock volta a toma-lo" esta escrita no codigo e e o que torna o RwLock seguro aqui.
- **Confianca.** **CONFIRMADA**.

### D-07 — Spans de texturas sobrepostas: escrever uma view anulava os slots das vizinhas

- **Descricao/sintomas.** Texturas empacotadas (o titulo guarda so os mips pequenos e poe a proxima textura no espaco dos mips grandes ausentes) tinham os slots **anulados** e saiam da fila de upload e do cartao "para sempre". Dezenas de milhares de slots por execucao; a primeira sala aparecia sem paredes/personagens.
- **Causa raiz.** A regra de invalidez era *overlap*: qualquer view nova cujo span em tamanho cheio se sobrepunha ao de uma view antiga marcava a antiga como stale. Com texturas empacotadas, **todo** vizinho se sobrepoe por construcao.
- **Commit que introduziu.** A regra `Overlap` e anterior; o commit que a documenta e substitui e `310c6eb` (o comentario cita "the rule until run 188").
- **Tentativas de correcao.** Nenhuma antes de `310c6eb`; o autor lista os slots antes de mexer (*"both proven with a slot listing before they were made"*).
- **Correcao definitiva.** `310c6eb` — `Rule::Start` (default): uma view so torna a antiga stale se **comecar onde a antiga comeca** (mesmo endereco com outro tamanho, ou dentro dos primeiros `HEAD = 4096` bytes). `VICTORY_STALE_TEXTURES=overlap` restaura a regra antiga; `=0` desliga.
- **Arquivos.** `crates/victory-gpu/src/descriptor/host.rs` (`Spans`, `Rule`, `HEAD`).
- **Evidencia.**
  - `git show 310c6eb`: `+ /// Spans may overlap: a texture's smallest levels lie at the start of its memory ...` e `+ const HEAD: u64 = 4096;` com a justificativa empirica *"(Of 129 pairs of overlapping neighbours in the listings of runs 188 and 189, none starts closer than 0x2000 to the one before it.)"*.
  - HEAD `descriptor/host.rs:203-219` mantem o enum com os tres modos.
- **Estado atual.** Corrigido, com heuristica explicita e numero de amostra declarado (129 pares, 2 runs).
- **Confianca.** **CONFIRMADA**.

### D-08 — Textura indexada subia zerada: pele preta e cabelo claro em 3 de 4 execucoes

- **Descricao/sintomas.** O rosto do personagem principal saia **preto** e o cabelo palido em ~3 de 4 execucoes; quais texturas perdiam a corrida mudava a cada execucao.
- **Causa raiz.** Uma textura escolhida por indice era enviada **uma vez**, quando o titulo escrevia a view, e so de novo se a view fosse reescrita ou o runtime avisasse de um remap. O titulo frequentemente **preenche a memoria depois** (as leituras de streaming chegam tarde), entao a copia do host guardava o que havia no primeiro olhar: zeros. Corrida de ordem, nao de sincronizacao.
- **Commit que introduziu.** O modelo "upload no write da view" e anterior; o commit que o documenta e corrige e `a803c87`.
- **Tentativas de correcao.**
  1. `a803c87` — reexame agendado: uma textura enviada inteira e olhada de novo nos frames **4, 12, 40, 120 e 400** (fingerprint de 48 KiB); se mudou, sobe de novo e o ciclo recomeca. `VICTORY_HEAP_RECHECK=0` desliga. Relato: >6.000 texturas reenviadas em uma execucao de teste.
  2. `c96b4f2` — texturas indexadas sobem **durante o loading** e antes do *bake* de decoracao do titulo (`VICTORY_EARLY_TEXTURES_MS=12`, `VICTORY_BAKE_TEXTURES_MS=1500`), porque o bake amostra por indice e roda uma vez.
  3. `310c6eb` — com o cartao cheio, textura que ja tem copia e reenviada quando a memoria do titulo mudou (`VICTORY_REUPLOAD_WHEN_FULL`).
  4. `faca57e` — rechecagem de tempo/bytes dentro do orcamento do passe (evita estourar o watchdog de 30 s no primeiro frame 3D).
- **Correcao definitiva.** `a803c87` + `c96b4f2` + `310c6eb` (combinacao; nenhum resolve sozinho).
- **Arquivos.** `crates/victory-gpu/src/descriptor/host.rs` (`TextureQueue::recheck`), `draw.rs`, `texture.rs`.
- **Evidencia.**
  - `git log -1 --format=%B a803c87`: *"the host copy kept what the memory held at the first look for the rest of the run: zeros ... the main character's face came out black and her hair pale in about three launches of four"* e *"Known gap: the fingerprint samples the start, middle and end of the memory, so a texture whose other parts arrive later is still missed."*
  - `git show c96b4f2 -- draw.rs`: `const BAKE_PIXEL_SHADERS: [&str; 4] = ["c0a7fe77b1862e10", ...]`.
- **Estado atual.** Mitigado. **O gap declarado continua** (fingerprint de 48 KiB); o README de usuario ainda recomenda `VICTORY_PAGE_WRITES=slots` para "re-enfileirar texturas placed" — sinal de que a classe de falha persiste em algum grau.
- **Confianca.** Sintoma e mecanismo: **CONFIRMADA**. Erradicacao completa: **NAO** (gap declarado).

### D-09 — Paredes ausentes na primeira sala: escritas por outra view da mesma pagina fisica

- **Descricao/sintomas.** A primeira sala aparecia sem paredes em parte das execucoes; com a correcao, *"the first room had its walls in the later frames of 4 runs out of 4 (2 of 5 before); the room's very first shot still comes up without walls in some runs"*.
- **Causa raiz.** O titulo mostra **uma pagina fisica em varios enderecos** e a copia era guardada (guardada por *guard page*) em **um** deles; escritas pela outra view nao eram vistas — **0,8 a 1,0 milhao de escritas** assim em 5 minutos.
- **Commit que introduziu.** O esquema de guarda por endereco e anterior (herdado); a correcao e `8dd30ab` ("A write through one view of a page is seen at its other views", codigo de **myokex**, `victory-amd c6dc4d8`, adaptando PR #115 de Spexivus, correcao de joroyt e parte de VIctory #1122 de tightfall).
- **Tentativas de correcao.**
  1. `8dd30ab` — guarda sobre **todas** as views da pagina (`VICTORY_VIEW_GUARD=0` desliga); leituras do kernel para memoria do titulo sao lidas ao lado e copiadas; escritas do storage em paginas fisicas contadas.
  2. `a26859f` — o que faltava alem das escritas: registros de descriptor copiados **como bytes** para o heap do titulo (cerca de **40.000 por frame** que nenhuma chamada anunciava) passam a ser espelhados antes de cada submissao (PR #305 de Flyander, apos upstream 86a5ed6), com fingerprint por slot.
  3. `310c6eb` — `Rule::Start` (D-07), a outra metade do sintoma.
  4. `9376fd6` — slots que draws leem por tabela ficam para os proprios draws (o passe sobre o heap cai de 70-220 ms para ~3 ms).
- **Correcao definitiva.** `8dd30ab` + `a26859f` + `310c6eb` (multicausal — a hipotese de causa unica era falsa).
- **Arquivos.** `crates/victory-gpu/src/watch.rs`, `resource.rs`, `descriptor.rs`, `descriptor/host.rs`, `shadow.rs`, `tiles.rs`.
- **Evidencia.**
  - `git log -1 --format=%B 8dd30ab`: numeros acima e *"0.8 to 1.0 million writes through another view in a five-minute run"*.
  - `git log -1 --format=%B a26859f`: *"about 40,000 records a frame that no call had announced; their host slots were empty or stale"* e *"98 buffer copies made (945 MB) where earlier runs made 1"*.
- **Estado atual.** Parcialmente corrigido. O **primeiro frame** da sala ainda pode vir sem paredes (afirmado pelo proprio autor).
- **Confianca.** **CONFIRMADA** (codigo + numeros declarados); a residua do primeiro frame: **PROVAVEL** (relato, sem codigo que a isole).

### D-10 — Janelas de draw indireto derrubavam o device (3 vezes)

- **Descricao/sintomas.** `VICTORY_INDIRECT_WINDOW=1` (PR #220, clamps de registros indiretos feitos em janela) reduzia o tempo de cartao de ~100-150 ms para 35-80 ms e **perdia o device tres vezes**, uma delas com page fault no cartao apos 160 s.
- **Causa raiz.** O verificador proprio (`VICTORY_INDIRECT_WINDOW_CHECK=1`) mostrou **uma lista desenhando 90 registros de 2^24 vertices a partir de uma janela**, onde o caminho por draw deixa passar oito. Um registro desses gera trabalho efetivo muito acima do que a janela assumiu.
- **Commit que introduziu.** `f85101b` (integracao do PR #220, "its windows off by default").
- **Tentativas de correcao.**
  1. `f85101b` — registro com 2^23 vertices ou mais por instancia passa a rodar **vazio** (zero), e a janela e invalidada por dispatch/draw direto/draw cujos shaders possam armazenar.
  2. `465b94f` — mecanismo diferente e mais seguro: `VICTORY_INDIRECT_GATHER=1` faz **um unico passe de clamp por lista** para registros espalhados, provado re-clampando 7,9 milhoes de registros no cartao (nenhum diferiu) — 10,3 fps.
  3. `9ceb7e7`/`a253706` — o *driver watch* passou a desligar as janelas na primeira entrada de driver no event log.
- **Correcao definitiva.** `f85101b` (guarda) + `465b94f` (caminho alternativo medido) + `a253706` (kill switch automatico).
- **Arquivos.** `crates/victory-gpu/src/draw.rs`, `indirect.rs`, `drvwatch.rs`.
- **Evidencia.**
  - `git log -1 --format=%B f85101b`: *"The history, so nobody repeats it: with windows on it lost the device three times here ... a record of 2^23 vertices an instance or more now runs empty. Since that fix: five runs with windows on, no device loss"*.
  - `git log -1 --format=%B a253706`: *"the first one turns draw windows off for the rest of the run. Every device loss here was announced that way 2.7 to 55 seconds ahead"*.
- **Estado atual.** Estavel com guardas; as janelas seguem **opt-in**, e o preset do README usa `VICTORY_INDIRECT_GATHER` (outro mecanismo).
- **Confianca.** **CONFIRMADA** (codigo); causa raiz exata do device removal: **PROVAVEL** (o proprio autor nao a fecha).

### D-11 — Depth bounds test nao traduzido: draws passavam em toda profundidade

- **Descricao/sintomas.** Draws que habilitavam *depth bounds* "passavam em toda profundidade" — a primeira sala so foi iluminada pela lampada quando isso passou a funcionar.
- **Causa raiz.** Duas ausencias: (a) pipeline que habilita o teste precisa ser criado a partir de um stream `DEPTH_STENCIL1` (senao o estado nao entra no PSO), e (b) `OMSetDepthBounds` do titulo nao chegava a lista do host.
- **Commit que introduziu.** Ausencia de implementacao (nao ha commit "introdutor": o recurso nunca existiu ate `c96b4f2`).
- **Tentativas de correcao.**
  1. `c96b4f2` (#82, Kaivyyy) — stream `DEPTH_STENCIL1` sob nome proprio de library + `set_depth_bounds` -> `draw::depth_bounds`.
  2. `8b8585d` (mathcx) — no WARP do CI o teste falhava: promove `DepthBoundsTestEnable` com `DepthEnable=false` e DSV valido a "depth read-only ALWAYS" em `GraphicsState::depth1`.
  3. `1d84afd`/`3e06462` — pula as assercoes de depth bounds no adaptador de software (VendorId `0x1414` ou handle DXGI ausente).
- **Correcao definitiva.** `c96b4f2` (produto) + `8b8585d` (teste/CI).
- **Arquivos.** `crates/victory-gpu/src/pipeline/host.rs`, `draw.rs`, `host.rs`.
- **Evidencia.**
  - `git log -1 --format=%B c96b4f2`: *"a pipeline that enables it is made from a DEPTH_STENCIL1 stream under its own library name, and the title's OMSetDepthBounds reaches the host list. Before, such draws passed at every depth. On the test machine (RTX 4070) the first room is lit by its lamp for the first time"*.
  - `8b8585d`: *"Promote DepthBoundsTestEnable with DepthEnable=false and a valid DSV format to read-only ALWAYS depth in GraphicsState::depth1 so WARP enforces OMSetDepthBounds"*.
- **Estado atual.** Funcional em hardware e em WARP.
- **Confianca.** **CONFIRMADA**.

### D-12 — LDS fora de faixa: trap na NVIDIA e, depois, limite errado (usava o total do host)

- **Descricao/sintomas.** (a) Acesso LDS fora dos limites do programa do console causava **trap de SRAM na NVIDIA** (o console le 0 e descarta a escrita). (b) Com o bounds check implementado, o limite comparado era o **total alocado pelo host**, que inclui os slots de troca entre metades da wave e o `wave_base` — logo, leituras do console fora da faixa **do programa** passavam a ser consideradas validas e liam scratch do host.
- **Causa raiz.** (a) semantica OOB do RDNA2 nao emulada; (b) `Lower::s.lds_dwords` (total do host) usado como se fosse o tamanho declarado pelo programa.
- **Commit que introduziu.** (a) pre-existente; (b) `a13a719` (`let lds_len = self.s.lds_dwords;`).
- **Tentativas de correcao.** `a13a719` (bounds check com predicado e clamp de indice para 0) -> `40beb26` (mathcx) separa `lds_dwords` do programa dos scratch slots e aplica o predicado em leituras, escritas e atomicos.
- **Correcao definitiva.** `40beb26`.
- **Arquivos.** `crates/victory-shader/src/emit/lower.rs`, `emit/mod.rs`.
- **Evidencia.**
  - `git show 40beb26`: `+ /// Thread-group shared memory dwords allocated by the console program itself (excluding host wave-exchange and wave_base scratch slots).` e `- if self.s.lds_dwords == 0` / `+ if self.lds_dwords == 0` com `let lds_len = self.lds_dwords;`.
  - HEAD `lower.rs:4442+` mantem o predicado `IULT(index, len)` e `select(ok, index, 0)`.
- **Estado atual.** Corrigido, com testes unitarios adicionados no mesmo commit.
- **Confianca.** **CONFIRMADA**.

### D-13 — `SLOT_VIEWS`: tabela de 16 bytes lida com stride de 8

- **Descricao/sintomas.** Shaders que leem o **endereco de um descritor** cujo indice e calculado em runtime recebiam o endereco/tamanho do **slot vizinho** — descritores errados em qualquer programa com indice dinamico.
- **Causa raiz.** O host escreve 4 dwords (16 bytes) por slot (`SLOT_VIEWS_COUNT * 16`), e o emissor calculava o byte offset como `slot << 3` (8 bytes/slot). O offset dobrava o indice.
- **Commit que introduziu.** `b62bbc0` ("shader: read descriptor addresses chosen by a computed heap index (SLOT_VIEWS)") — no mesmo commit convivem o escritor de 16 bytes e o leitor de 8.
- **Tentativas de correcao.** Nenhuma intermediaria.
- **Correcao definitiva.** `5603b45` (Lilith133T, PR `pr-7`) — constante `SLOT_VIEW_BYTES = 4 * size_of::<u32>()` (16) usada nos dois lados, mais `Shader::read_view_slot()` centralizando clamp + offset.
- **Arquivos.** `crates/victory-gpu/src/descriptor/host.rs`, `crates/victory-shader/src/emit/lower.rs`, `emit/shader.rs`, `emit/mod.rs`.
- **Evidencia.**
  - `git show 5603b45`: `- let byte = ... Shl(slot, three)` (removido de `lower.rs`) -> `+ let stride = self.m.i32(SLOT_VIEW_BYTES); let byte = self.m.bin(block, Bin::Mul, slot, stride);` em `shader.rs:374-375`.
  - `+ pub const SLOT_VIEW_BYTES: u32 = 4 * size_of::<u32>() as u32;` e `+ /// Byte stride of a view entry: address low and high, byte length, and stride.`
- **Estado atual.** Corrigido.
- **Confianca.** **CONFIRMADA** (o desalinhamento 8 vs 16 e visivel nos dois lados do diff).

### D-14 — `v_permlane16`/`v_permlanex16` nao pediam as lanes do console

- **Descricao/sintomas.** Programa compute cuja unica instrucao entre lanes era `v_permlane16_b32`/`v_permlanex16_b32` nao recebia `WaveSize` nem remapeamento de lane: em release as lanes seguiam a wave do host (64 em algumas GPUs) e em debug o `debug_assert` de `cross_lane` parava.
- **Causa raiz.** O teste de "codigo que le outras lanes" procurava `permute` (`ds_permute`, `ds_bpermute`), DPP, `ds_swizzle`, `v_mbcnt` e `v_readlane/v_writelane` — mas os nomes sao `permlane`, **nao** `permute`.
- **Commit que introduziu.** A deteccao original (herdada); o buraco foi encontrado "lowering each VALU opcode alone in a compute program".
- **Tentativas de correcao.** Nenhuma.
- **Correcao definitiva.** `448416c` (lcg007xd-c) — `+ || n.starts_with("v_permlane")`, com teste que monta o programa e exige `wave_size == Some(32)`.
- **Arquivos.** `crates/victory-shader/src/emit/lower.rs`.
- **Evidencia.** `git show 448416c` (linhas 36-41 e o teste `permlanes_ask_for_console_lanes`); mensagem: *"their names say permlane, not permute"*.
- **Estado atual.** Corrigido, coberto por teste.
- **Confianca.** **CONFIRMADA**.

### D-15 — `v_dot8_i32_i4` ausente da tabela de opcodes

- **Descricao/sintomas.** Programa que usava VOP3P `0x18` **falhava ao decodificar** (`0x16`, `0x17` e `0x19` estavam la).
- **Causa raiz.** Buraco na tabela de opcodes GFX10: a ISA RDNA2 e o disassembler gfx1030 do LLVM listam o opcode; a tabela deste tradutor nao.
- **Commit que introduziu.** A tabela (herdada do upstream).
- **Tentativas de correcao.** Nenhuma intermediaria.
- **Correcao definitiva.** `75aa531` (lcg007xd-c) — `dot.rs` com o produto de oito nibbles com sinal (`shl` + `ashr` por 28, soma em `i32`, `sat_i32_add` no clamp, justificado por faixa `[-448, 512]`), `opcodes.rs` com a entrada, teste em `valu_tests.rs`.
- **Arquivos.** `crates/victory-shader/src/gfx10/opcodes.rs`, `emit/dot.rs`, `emit/lower.rs`, `emit/valu_tests.rs`.
- **Evidencia.** `git show 75aa531`; mensagem: *"VOP3P 0x18 was missing from the opcode table (0x16, 0x17 and 0x19 are there; AMD's RDNA2 ISA and LLVM's gfx1030 disassembler list it), so a program using it failed to decode."*
- **Estado atual.** Corrigido.
- **Confianca.** **CONFIRMADA**.

### D-16 — MIMG: variantes ausentes e LOD com conversao de log2 dupla

- **Descricao/sintomas.** Duas classes de falha visual/recusa em amostragem de textura: (a) variantes de `image_sample` com offset/clamp/bias/LOD (e `gather4_l`, `a16`) nao eram traduzidas; (b) o LOD calculado a mao aplicava `ln(x) * LOG2_E`, mas o `Log` do DXIL **ja devolve log2** — o mip escolhido era ~1,44x o correto (textura borrada/detalhe errado) em `image_sample` com LOD derivado.
- **Causa raiz.** (a) cobertura parcial do decodificador/emissor MIMG; (b) confusao entre `log` natural e `log2` na sequencia de instrucoes emitida.
- **Commit que introduziu.** (b) `578d459` (import; `git log -S"LOG2_E"`); (a) pre-existente.
- **Tentativas de correcao.**
  1. `6dcd8cd` (mathcx) — `emit/image.rs` (1.170 linhas) separado de `lower.rs`, cobrindo offset/clamp/bias/LOD, `a16`, gather.
  2. `a290636` (porte de DexNom `921d8d3`/`bc3cd4b`) — remove o `LOG2_E` e aplica o clamp minimo de LOD do guest antes de passar o nivel (SampleLevel nao tem operando de clamp): `let lod = self.unary(23, texels); let lod = if has_cl { self.binary(35, lod, clamp) } else { lod };`.
  3. `2471a62`/`77b4b4c` — `a16`, `_cl`/`_o`, `gather4_l`, cubemap `grads=2`.
- **Correcao definitiva.** `6dcd8cd` (cobertura) + `a290636` (semantica de LOD).
- **Arquivos.** `crates/victory-shader/src/emit/image.rs`, `emit/lower.rs`.
- **Evidencia.**
  - `git show 6dcd8cd` -> `new file mode 100644 ... crates/victory-shader/src/emit/image.rs | 1170 ++++`.
  - `git show a290636 -- emit/image.rs`: `- let ln = self.unary(23, texels); - let per_ln2 = self.s.m.f32(std::f32::consts::LOG2_E.to_bits()); - let lod = ... Mul(ln, per_ln2);` `+ // DXIL Log already returns log2...` `+ let lod = self.unary(23, texels);`.
- **Estado atual.** Corrigido.
- **Confianca.** **CONFIRMADA** (o diff mostra a conversao removida e o motivo).

### D-17 — `d16` em buffer store reinterpreta inteiros de 16 bits como `f16` (**SEM CORRECAO**)

- **Descricao/sintomas.** Escritas `buffer_store_*_format_d16_*` tratam os dados como **f16** e alargam para float de 32 bits. Formatos inteiros de 16 bits (u16/i16) que usem `d16` gravam valores diferentes dos do console (um `0x0001` vira o f16 subnormal ~6e-8, nao `1`).
- **Causa raiz.** Atalho consciente no emissor: nao ha distincao de formato no caminho `d16`; o dado e sempre interpretado como canal f16.
- **Commit que introduziu.** O caminho `d16` (herdado; o comentario `// shortcut:` esta no HEAD).
- **Tentativas de correcao.** Nenhuma commitada. O problema e **documentado** em `docs/comparative-performance-hypotheses.md` (*"VItoriaBr still has a documented floating-point interpretation shortcut in its d16 formatted-buffer emitter"*), e existe um caso correlato em outro fork (victory-fork #927, buffers de animacao facial zerados ate o tratamento de typed-buffer 16-bit) — citado la, nao provado aqui.
- **Correcao definitiva.** **SEM CORRECAO.**
- **Arquivos.** `crates/victory-shader/src/emit/lower.rs` (linhas ~10104-10115 e ~10251-10257).
- **Evidencia.** HEAD: `// shortcut: d16 data is taken as f16 (integer formats would // keep 16 bits), widened for the 32-bit float store.` (`lower.rs:10108-10109`).
- **Estado atual.** Aberto. Impacto desconhecido: depende de o titulo usar `d16` com formato inteiro.
- **Confianca.** Codigo: **CONFIRMADA**. Impacto em imagem: **NAO VERIFICADA**.

### D-18 — XGameSave podia apagar o save commitado

- **Descricao/sintomas.** Perda de save: a recuperacao aceitava **qualquer** nome `v<digitos>` como versao. Uma pasta residual `v01` ao lado de `v1` parseava como versao 1; a limpeza reconstruia o nome como `v1` e **apagava o save commitado** (issue **#870** citada no commit). Entrada mais nova, corretamente nomeada mas ilegivel (um `v2` vazio, um arquivo, um manifesto quebrado) causava o mesmo: `v1` era podado como antigo, a leitura falhava, o update seguinte commitava um `v1` novo so com os blobs alterados, e a leitura seguinte podava de novo.
- **Causa raiz.** Identidade textual do diretorio de versao tratada como numero, e ordem "poda primeiro, le depois".
- **Commit que introduziu.** A rotina de recuperacao (herdada).
- **Tentativas de correcao.** Nenhuma antes.
- **Correcao definitiva.** `d1da85a` (moe-miau, cherry-pick de `b8dbcba`) — `version_number()` exige `v<N>` exatamente como `commit` escreve; `current()` pega a **maior versao cujo manifesto le**, remove so as abaixo dela e **nao toca** em entradas mais novas ilegiveis; o proximo commit vai acima do maior nome presente.
- **Arquivos.** `crates/victory-hle/src/modules/xgamesave.rs`.
- **Evidencia.** `git show d1da85a` (`+ fn version_number(file: &str) -> Option<u64> { ... (n > 0 && format!("v{n}") == file).then_some(n) }`); mensagem: *"a leftover v01 next to a committed v1 parsed as version 1, and the cleanup rebuilt the name as v1 and deleted the committed save (#870)"*.
- **Estado atual.** Corrigido, com testes de recuperacao.
- **Confianca.** **CONFIRMADA**.

### D-19 — Deadlock do loader lock no cache de modulos HLE

- **Descricao/sintomas.** Processo **travava e nao conseguia nem sair**: `ExitProcess` tambem precisa do loader lock.
- **Causa raiz.** `host::module` segurava o lock do cache de modulos enquanto chamava `LoadLibraryExA`. Carregar uma DLL nova toma o **loader lock** do SO, e codigo do titulo roda sob esse lock (thread attach de DLL do titulo, callbacks TLS) e pode precisar do cache (`host!` ou `LoadLibrary` tardio). Thread A: cache -> espera loader lock. Thread B: loader lock -> espera cache.
- **Commit que introduziu.** O cache de modulos (herdado).
- **Tentativas de correcao.** Nenhuma antes.
- **Correcao definitiva.** `708b450` (Duff) — procura no cache, **solta o lock para carregar**, retoma para registrar; *"a thread that loaded the same DLL meanwhile got the same module, which stays loaded"*. Teste em processo filho com callback TLS que dorme 500 ms dentro do `DLL_THREAD_ATTACH` (um deadlock dentro do loader impediria o processo de sair, entao o pai mata).
- **Arquivos.** `crates/victory-hle/src/host.rs`.
- **Evidencia.** `git show 708b450` (`+ // Not under the cache's lock: loading takes the OS loader lock ...` e o teste `loading_a_host_dll_does_not_block_thread_attach` com `#[unsafe(link_section = ".CRT$XLY")]`).
- **Estado atual.** Corrigido, com teste de regressao honesto (processo separado).
- **Confianca.** **CONFIRMADA**.

### D-20 — Device loss introduzido pela atualizacao wave64 (barreiras dentro de branches/loops)

- **Descricao/sintomas.** Um dispatch de dois programas compute (`ptfx_CS_lightAtlas_Sprite` e `_Trail`) **parava o cartao ate o reset do device**.
- **Causa raiz.** A atualizacao wave64 passou a emitir esses programas numa forma que coloca **barreiras de grupo dentro de branches e loops**; com lanes divergentes a barreira nunca e alcancada por todas as lanes do grupo.
- **Commit que introduziu.** `6a2d9f9` (import do victory-gta6 com a forma wave64).
- **Tentativas de correcao.**
  1. `e9765a7` — a forma wave-wide fica **desligada** por padrao (`VICTORY_WAVE_WIDE=1` a pede), a forma de 32 lanes (sem barreira) permanece, e a **chave do cache de shaders passa a incluir os switches de wave** (senao traducoes velhas mascarariam a mudanca).
  2. `1ece3a6`/`808a209` — `s_barrier` em bloco que nao pos-domina, em loop, ou apos `abort_block` emite **so fence** de memoria (`TGSM_FENCE`/`UAV_FENCE`).
  3. `f2a2097` (PR #34) e `8b8585d` — guarda automatica de barreiras divergentes de uma unica wave do console em GPUs de wave estreita (proposta posterior, nao default nesta arvore segundo `docs/performance-architecture.md`).
  4. `90228a7`/`9e49b8a`/`2119b0e` — mascaras de lane estacionadas entram, saem e voltam (ver H-03).
- **Correcao definitiva.** `e9765a7` (gating + chave de cache), com os refinamentos acima.
- **Arquivos.** `crates/victory-shader/src/emit/mod.rs` (forma wave-wide), `shader_cache.rs` (chave), `emit/lower.rs` (`barrier_mode`).
- **Evidencia.**
  - `git log -1 --format=%B e9765a7`: *"a dispatch of them stood on the card until the device was reset. That form is now off unless VICTORY_WAVE_WIDE=1, and the shader cache key includes the wave switches."*
  - HEAD `emit/mod.rs:438-443`: *"Off unless asked for (VICTORY_WAVE_WIDE=1), for a console wave run as two host waves: this form puts group barriers inside branches and loops, and runs 121-124 lost the card in a submission of two programs ... (ptfx_CS_lightAtlas_Sprite and _Trail)."*
- **Estado atual.** Corrigido e documentado no codigo com os nomes dos programas e os numeros das execucoes.
- **Confianca.** **CONFIRMADA**.

### D-21 — Deriva de documentacao: numeros retirados, contradicao sobre TDR, contagens divergentes

- **Descricao/sintomas.** Tres sintomas de rastreabilidade:
  1. O README do fxceless publicou **6,0 -> 8,9 fps** para as janelas de draw indireto e depois **retirou** a comparacao (runs em cenas diferentes, ~4.300 vs ~1.550 draws), substituindo por uma reta de tempo de frame contra draws (145 ms vs 124 ms a 4.500 draws) — `0b0074e`.
  2. O README deste fork afirma **"zero TDR"** na tabela de status (linha 26) e, 50 linhas depois, avisa que *"entrar no mundo 3D em uma GPU real ainda pode disparar reset do driver"* (linha 75).
  3. Contagem de programas do *builder* de BVH: README diz **50/53** (com `VICTORY_FIRSTLANE_OWN=1` chegando a 53/53), `docs/fork-ray-tracing.md` diz **49 de 52**; e o `docs/performance-architecture.md` afirma que o modo de barreira *"defaults to 2"*, enquanto o codigo tem default **3** (`parse_group_sync`).
- **Causa raiz.** README reescrito por tres maos (fxceless, uiliannn, contribuidores) e docs tecnicos escritos por revisores externos em momentos diferentes, sem fonte unica de verdade; os numeros sao de maquinas diferentes (RTX 4070, RTX 5060 Ti, RX 6500 XT).
- **Commit que introduziu.** A deriva entra com os merges de README (`0b0074e`, `3e06462`, `e140f41`, `69c665c`) e com os docs (`a858713`, `e354b6a`).
- **Tentativas de correcao.** Correcoes pontuais e explicitas: `0b0074e` (retrata), `a393a19` (PR #21: corrige o guia de cache de shaders e a leitura de log), `ca4759f` (conserta o proprio teste no CI).
- **Correcao definitiva.** **SEM CORRECAO** para a consistencia global.
- **Arquivos.** `README.md`, `docs/*.md`.
- **Evidencia.** README.md:26 vs :75; README.md:164 vs `docs/fork-ray-tracing.md` (secao "What this fork adds", item 1: *"49 of 52"*); `emit/mod.rs:176-182` (`_ => 3`).
- **Estado atual.** Aberto. E o defeito de menor risco tecnico e maior risco de decisao: quem le o README conclui que o titulo esta estavel, quem le os docs conclui que nao.
- **Confianca.** **CONFIRMADA** (as tres divergencias estao textualmente no repositorio).

## E. Relacao de causa e efeito

**E-01 — A textura indexada que subia zerada.**
`578d459` (modelo "upload no write da view") -> textura preenchida depois pelo streaming -> copia do host com zeros -> `a803c87` (rechecagem em 4/12/40/120/400 frames) resolve parcialmente -> `c96b4f2` (upload antecipado e antes do bake) remove a janela de corrida do bake -> `310c6eb` (reenvio com cartao cheio) fecha a terceira porta -> **residuo**: fingerprint de 48 KiB (D-08) permite textura cujo miolo chega depois continuar errada -> README recomenda `VICTORY_PAGE_WRITES=slots` como muleta de usuario.

**E-02 — A primeira sala sem paredes.**
`8dd30ab` descobre 0,8-1,0 milhao de escritas invisiveis por outra view -> corrige a guarda -> paredes aparecem em 4/4 execucoes (2/5 antes) -> mas **registros de descriptor copiados como bytes** (~40.000/frame) tambem nao eram vistos -> `a26859f` espelha esses registros antes de cada submissao -> **e** texturas empacotadas tinham slots anulados pela regra de overlap -> `310c6eb` troca por `Rule::Start` -> **estado**: primeiro frame ainda pode vir sem paredes. Tres causas independentes, uma aparencia.

**E-03 — Do esgotamento de descriptors ao device removal na AMD.**
Heap dinamico acaba -> `State::tables` cai no offset da tabela do titulo -> shader le textura/nulo como CBV -> endereco base selvagem -> remocao do device (issue #59) -> `3c81420` **descarta o draw** em vez de ler lixo -> capacidade cresce 2^16 -> 2^18 (`edf64e2`, `e8d3b64`, `bd33acc`) -> reciclagem apos `execute` sincrono (`a65d054`) -> busca por runs que cruzam o cursor (`a91ef70`) -> **o proprio fix e truncado** (`take(DYNAMIC)`) -> `009856f` -> fixture do teste errado -> `b37344f`. Cadeia de seis elos, dois deles defeituosos.

**E-04 — O frame que nao cabia no orcamento do titulo.**
`9ceb7e7` mede ~1 settle por frame (55-100 ms de espera) porque o titulo muda centenas de milhares de runs de poucos bytes sob escritas de shaders em voo -> o cap `VICTORY_MERGE_RUNS=512` sempre estoura -> os bytes passam a ser mesclados **no cartao** por mascara (`CARD_MERGE`, um thread por dword de mascara) -> settles 1,17 -> 0,00/frame, espera 58-61 -> 13 ms, frame 162-172 -> 139-140 ms (~10%).

**E-05 — O hash do cartao como gargalo invisivel.**
`713a095` observa que uma peca de 64 KiB era hasheada por **um** thread em 4.096 passos dependentes -> o passe custa o tempo de uma peca inteira, qualquer que seja o numero de pecas -> `CARD_COMPARE_WIDE` da um grupo de 64 threads por peca (passos `t, t+64, ...`, loads coalescidos) e dobra o hash na lane 0 -> listas de envio/hash caem de 23-29 ms para **~5 ms** -> exige uma decisao de processo ("um processo usa uma forma para tudo que armazena e compara", `VICTORY_CARD_HASH_WIDE`) — **acoplamento nao obvio que qualquer reuso precisa respeitar**.

**E-06 — O sync de descriptors: 70-220 ms -> ~3 ms.**
`a26859f` introduz o passe que espelha registros copiados como bytes, com **BLAKE3 por registro** -> `9376fd6` mede 35-42 ms/frame so de hash sobre 250-300 mil registros -> troca por fingerprint multiplica-e-dobra + deixa slots lidos por tabela para os draws -> ~3 ms -> `213fa54` troca o `DefaultHasher` por hash de tabela + comparacao dos 3 dwords do registro (colisao custa copia, nao descriptor errado) -> HEAD `draw.rs` com `Plain` hasher e `fresh_records`. **Aqui a cadeia e de desempenho, nao de corretude — e e a maior economia documentada do repositorio.**

**E-07 — A corrida do relay e o fix que virou defeito.**
`578d459` (relay stage 2) -> `copy_back_done` libera ranges cedo -> 0xc0000005 no loading -> `91cd9c9` reordena **e deduplica** -> `Flights` decrementado por flight, dedup quebra o refcount -> `aa85bb0` remove a deduplicacao e mantem a ordem -> estavel. **Licao**: a correcao de uma corrida introduziu um bug de contagem de referencia no mesmo dia.

**E-08 — A barreira wave64.**
`6a2d9f9` (forma wave-wide com barreiras em branches) -> dois programas de atlas de luz travam o cartao -> `e9765a7` desliga a forma por padrao **e adiciona os switches de wave a chave do cache** (sem isso, o cache serviria a traducao antiga) -> `1ece3a6`/`808a209` reduzem barreiras a fence onde o console nao exige encontro -> `f2a2097` propoe guarda automatica. **A dependencia cache<-switch e o elo escondido**: corrigir o emissor sem corrigir a chave teria deixado o defeito vivo em toda maquina com cache.

## F. Correcoes importantes

**F-01 — Merge de escritas da CPU no cartao por mascara de bits (`9ceb7e7`).**
*O que estava errado:* quando o titulo escrevia memoria que shaders em voo tambem escrevem, a camada subia cada run alterado como copia propria (ate 512) e, acima disso, **esperava o cartao** (~1 settle/frame, 55-100 ms). *Por que falhava:* o volume de runs (centenas de milhares de poucos bytes) e estrutural, nao anomalo — o cap era sempre atingido. *Como foi corrigido:* `shadow::Masked::take` compara pagina de 8 em 8 bytes e marca **1 bit por byte alterado** por bloco de 64 KiB; o compute `CARD_MERGE` faz `card = (card & ~m) | (title & m)` com um thread por dword de mascara; bytes fora da mascara preservam o que os shaders escreveram; as mascaras viajam por submissao e sao recicladas quando o fence passa (ate 32 buffers). *Por que funciona:* a escrita da CPU e **fundida** com o resultado do shader na granularidade do byte, sem esperar, e a operacao e idempotente por construcao. *Dependencias:* compute shader cs_5_1, `shadow.rs`, `host.rs`, GPU com copy/fill; fallback para copia por run se faltar buffer, shader ou alinhamento de 64 KiB. *Reutilizavel:* **sim, altamente** — o padrao "mascara de bytes + merge no device" serve a qualquer camada que espelhe memoria de CPU em memoria de GPU com produtores concorrentes.

**F-02 — Hash de peca de copia com grupo de 64 threads (`713a095`).**
*O que estava errado:* um thread por peca de 64 KiB, 4.096 passos dependentes; o passe custa o tempo de uma peca e o cartao fica ocioso. *Como foi corrigido:* thread `t` hasheia os passos `t, t+64, ...` (coalescido), a lane 0 dobra os 64 hashes na ordem das lanes. *Por que funciona:* cada passo e **bijecao** do estado, entao qualquer mudanca confinada aos passos de uma lane muda o hash da peca; e a forma e escolhida **uma vez por processo** (`card_hash_wide()`), porque hashes armazenados e comparados tem de ser do mesmo algoritmo. *Dependencias:* limite de 16.385 pecas por dispatch (copias < 1 GiB), `groupshared`, WARP test. *Reutilizavel:* sim, com o aviso do acoplamento.

**F-03 — Espelho de registros de descriptor copiados como bytes (`a26859f` + `9376fd6` + `213fa54`).**
*O que estava errado:* o titulo copia registros de descriptor para o heap como **bytes**, sem chamada de API; os slots do host ficavam nulos ou velhos (~40.000 registros/frame medidos). *Como foi corrigido:* write-watch nas paginas de heaps *placed*; por slot, fingerprint barato; espelho antes de cada submissao; slots que draws leem por tabela ficam para os draws; hash de tabela antes de comparar os 3 dwords. *Por que funciona:* o passe olha so paginas sujas e compara registros **de verdade** depois do hash — colisao custa uma copia, nunca um descriptor errado. *Dependencias:* `watch.rs` (`Tracked`), `descriptor.rs` (`sync_heaps`), `draw.rs` (`fresh_tables`), `VICTORY_HEAP_SYNC`/`VICTORY_HEAP_SYNC_TABLES`. *Reutilizavel:* sim — e a resposta geral para "o titulo escreve em memoria que a API nao ve".

**F-04 — Regra `Rule::Start` para spans de textura (`310c6eb`).**
*O que estava errado:* invalidar por sobreposicao de span em tamanho cheio anula texturas empacotadas. *Como foi corrigido:* invalidar so quando a view nova **comeca** onde a antiga comeca (ou nos primeiros 4 KiB), com `HEAD = 4096` justificado por 129 pares medidos em 2 runs. *Por que funciona:* a menor cadeia de mips de uma textura ocupa um bloco de 4 KiB que **nao e compartilhado** por outra textura; logo "mesmo inicio" identifica reuso real, e "so alcanca os mips grandes ausentes" nao diz nada. *Dependencias:* `Spans`/`Rule`; `VICTORY_STALE_TEXTURES`. *Reutilizavel:* sim, mas a heuristica e **empirica para este titulo** — em outro titulo, medir de novo.

**F-05 — Rechecagem agendada de textura indexada (`a803c87`).**
*O que estava errado:* corrida entre "view escrita" e "memoria preenchida". *Como foi corrigido:* fingerprint de 48 KiB nos frames 4/12/40/120/400; mudou, reenvia e recomeca. *Por que funciona:* cobre a janela tipica de streaming sem custo por frame e sem depender de aviso do titulo. *Dependencias:* `TextureQueue::recheck`, `HostHeap::texture_slot_checked`. *Reutilizavel:* sim, com o **defeito conhecido** de amostrar so inicio/meio/fim (D-08).

**F-06 — `SLOT_VIEWS` com stride nomeado (`b62bbc0` + `5603b45`).**
*O que estava errado:* offset de 8 bytes para entradas de 16. *Como foi corrigido:* `SLOT_VIEW_BYTES` compartilhado entre emissor e host, `read_view_slot()` com clamp e offset unicos. *Por que funciona:* uma unica constante define o layout dos dois lados do contrato shader<->host. *Reutilizavel:* sim; a licao e **nomear o stride no ponto de contrato**, nao nos dois pontos de uso.

**F-07 — Bounds check de LDS com o tamanho do **programa** (`40beb26`).**
*O que estava errado:* comparar com o total alocado pelo host (inclui scratch de troca de metades). *Como foi corrigido:* `lds_dwords` do console separado do total, predicado `IULT` e indice seguro 0, leitura devolve 0 e escrita e descartada. *Por que funciona:* reproduz a semantica do RDNA2 e evita trap de groupshared na NVIDIA. *Dependencias:* `Lower`, `io::ComputeIo::lds_bytes`. *Reutilizavel:* sim, especialmente para qualquer lowering que rode wave64 do console em duas waves do host.

**F-08 — Ordem de publicacao no `relay_land` (`aa85bb0`).**
*O que estava errado:* liberar ranges antes de publicar `line.written` e acordar waiters. *Como foi corrigido:* coletar todos os ranges por flight, consumir a fila, atualizar `line.written`, `notify_all`, **e so entao** `copy_back_done`, sem deduplicar entre flights. *Por que funciona:* a liberacao e o **ultimo** evento; nenhum refresh concorrente ve o range livre enquanto o merge escreve. *Reutilizavel:* sim — e uma instancia do padrao "estado -> sinal -> libera recurso" que qualquer pool com espera deve seguir.

**F-09 — Cache de shaders chaveado pela traducao, nao pelo executavel (`a6a8d39`, `ef30575`, `a393a19`).**
*O que estava errado:* o guia dizia que a chave incluia o hash de `victory.exe`, o que invalidava o cache inteiro a cada recompilacao. *Como foi corrigido:* `translation_fingerprint()` = hash dos fontes de `victory-shader` (gerado por `build.rs`), flags de build, **valores em runtime de todos os `VICTORY_*` lidos pelo tradutor**, `loop_limit`/`group_sync`/`shader_opt`, e capacidades do host (`view_heap_limit`, `host_max_wave`); `VICTORY_SHADER_CACHE_KEY=exe` e opt-in. *Por que funciona:* identifica exatamente as entradas da traducao, e a impressao e recalculada a cada consulta (nao grava traducao feita antes do device informar capacidades). *Dependencias:* `build.rs` do `victory-shader`, `shader_cache.rs`. *Reutilizavel:* sim; e a correcao certa para qualquer cache de traducao.

**F-10 — Deteccao de codigo entre lanes por lista de nomes, com teste por opcode (`448416c`).**
*O que estava errado:* um conjunto de nomes incompleto decide a semantica de lane. *Como foi corrigido:* incluir `v_permlane*` e **testar lowering de cada opcode VALU isoladamente** num compute. *Por que funciona:* transforma uma lista sujeita a esquecimento em um caso de teste por opcode. *Reutilizavel:* sim — o metodo ("baixar cada opcode sozinho") encontra buracos que a leitura de codigo nao encontra.

## G. Problemas ainda existentes

**G-01 — Ray tracing nao ilumina nada.** `docs/fork-ray-tracing.md`: *"Status, 2026-11-25: nothing is lit by ray tracing yet. Most of the machinery translates and is unit-tested; it has not yet run end to end in the game. Everything here is off by default."* Compensado por `ambient.rs` (luz ambiente sombreada por profundidade, `VICTORY_AMBIENT=0.6`) portada de Belobug. **CONFIRMADA**.

**G-02 — `VICTORY_FIRSTLANE_OWN=1` corrompe offsets de AABB.** README.md:164: chega a 53/53 programas do builder mas *"o atalho corrompe offsets de AABB na memoria (crash apos ~18 s), manter desligado ate a troca completa entre metades da wave64"*. Consequencia: o builder de BVH fica em 50/53 e **nao ativa por default**. **PROVAVEL** (relato do autor).

**G-03 — 397 PSOs compute recusados na NVIDIA.** README.md:39: *"Os unicos 397 PSOs recusados pela RTX 5060 Ti sao compute shaders que exigem wave64"*. E uma limitacao de plataforma, nao um bug — mas explica parte da cobertura de 98,5%. **CONFIRMADA** (declarada; nao reproduzida).

**G-04 — Atalho `d16` (D-17).** **SEM CORRECAO**, documentado apenas em comentario e em doc de terceiro.

**G-05 — Rechecagem de textura amostra so 48 KiB.** Declarado no proprio commit `a803c87`; nao ha commit posterior que feche o gap. **CONFIRMADA** (texto do commit), impacto **NAO VERIFICADO**.

**G-06 — Dispatches indiretos acima de 65.535 grupos sao truncados ou zerados.** `indirect.rs:129` (`min(..., 65535)`), `:142-146` (`ahead && g.x > 65535 -> 0`; senao `min(g.x, 65535)` com `y=z=1`). O README do fxceless registrava o caso ("um passe compute indireto pede ~77.500 grupos em x onde um dispatch roda 65.535"); a resposta atual e **limitar**, nao dividir. Trabalho e perdido silenciosamente (contado, nao recuperado). **CONFIRMADA** (codigo); impacto visual **NAO VERIFICADO**.

**G-07 — Texturas com apenas mips interiores residentes.** Efeito colateral aceito da reversao `c931d12` (D-01): a informacao existe no sistema, mas consulta-la custa 752 ms/frame. Alternativa nao implementada. **CONFIRMADA** (codigo e comentario).

**G-08 — Device removal no mundo aberto, causa desconhecida.** `0b0074e`: *"A first play session past the first rooms reached the open world and lost the device there with no driver entry (reason 'device removed'): the card had no room for new textures ... Cause of that removal not known either."* **PROVAVEL** (relato); sem correcao.

**G-09 — Regressao AMD nao reproduzida (issue #18).** `docs/performance-architecture.md` e `docs/comparative-performance-hypotheses.md` descrevem a issue **#18** (RX 6500 XT, frames de varios segundos e regressao de textura de rosto com cache aquecido), comparando com o commit antigo `18924c7`; o maintainer propos mudar so o cap de buffers de cartao (6144 -> 1024 MiB) como primeiro teste controlado. **NAO VERIFICADA** nesta analise (sem rede, sem hardware AMD).

**G-10 — Primeiro frame da primeira sala ainda pode vir sem paredes.** Texto de `8dd30ab`; nao ha commit posterior que afirme o contrario. **PROVAVEL**.

**G-11 — Suite completa nao rodada em varios merges.** Varios commits do sync declararam explicitamente *"This merged tree was built; its test suites were not run"* (`9376fd6`, `a26859f`, `4e038bf`, `a253706` diz que rodou so as partes mudadas). Nao e um bug: e um **estado de garantia** que precisa ser dito. **CONFIRMADA** (texto dos commits).

**G-12 — Proliferacao de switches.** O preset do README usa 16 variaveis `VICTORY_*`; o codigo contem dezenas (incluindo `=alt` para A/B de 20 s). Cada uma e um caminho de teste que pode nao estar coberto. Risco de configuracao, nao de codigo. **CONFIRMADA** (contagem no README).

## H. Regressoes

**H-01 — `89434b7` -> `c931d12`: a correcao que custou 752 ms/frame.** Um "recupera texturas esparsas" (D-01) virou o pior stall documentado do repositorio, revertido em 22 minutos. *Natureza:* regressao de desempenho introduzida por correcao de corretude. *Deteccao:* teste ao vivo na RTX 5060 Ti com contador de texturas recuperadas (0 hits).

**H-02 — `91cd9c9` -> `aa85bb0`: o fix da corrida quebrou o refcount.** A deduplicacao de ranges entre flights (D-02) reduziu chamadas de `copy_back_done`, que **decrementa `Flights` por flight** — os ranges de um flight ficavam pendentes para sempre. Corrigido no mesmo dia, removendo a deduplicacao. *Natureza:* regressao de correcao; *sintoma latente*: range nunca liberado (vazamento), nao crash.

**H-03 — `90228a7` -> `9e49b8a` -> `2119b0e`: a mudanca que saiu e voltou.** Mascaras de lane estacionadas (`#11`) entram, saem ("o ganho de iluminacao nao pode ser mostrado, custa ~40 ms de GPU por frame, a imagem esta sendo re-checada uma mudanca por vez a partir de uma arvore boa") e voltam no dia seguinte ("a retirada nao era desejada"). Ficou via `2d3f43c`. *Natureza:* oscilacao de decisao, nao defeito de codigo. *Efeito:* o historico nao registra a medicao que justificou a volta.

**H-04 — `6a2d9f9` -> `e9765a7`: wave64 derrubou o cartao.** D-20. *Natureza:* regressao de corretude introduzida por melhoria de cobertura (mais programas traduzindo). *Deteccao:* apenas em execucao real (dispatch que para o cartao).

**H-05 — `a13a719` -> `40beb26`: o bounds check com o limite errado.** D-12. *Natureza:* regressao de corretude dentro da propria correcao (usou o total do host como se fosse o tamanho do programa). *Janela:* ~2 dias.

**H-06 — `a91ef70` -> `009856f`: o fix que nao consertava.** A extensao da varredura (D-04) era truncada por `take(DYNAMIC)`. *Natureza:* correcao ineficaz (nao regressao: o comportamento anterior era igualmente ruim), com teste que passava por acidente de fixture — corrigido em `b37344f`.

**H-07 — Merge do PR #6 quebrou o build.** `ef30575` (merge de `victory warm`) deixou **duas definicoes de `warm_shader`**; `06161b9` removeu a duplicata em `crates/victory-hle/src/modules/d3d12x.rs`. *Natureza:* regressao de integracao (merge sem build).

**H-08 — Teste novo quebrou o CI portatil.** `213beb9` (teste sintetico do POPS, do fxceless) usava `Container`/`FourCC` que nao existem fora do Windows; o Clippy portatil em `origin/main 4e038bf` falhou e `ca4759f` passou a **parsear o container em toda plataforma**. *Natureza:* regressao de CI introduzida por codigo de teste; corrigida por endurecimento do teste (melhor que `#[cfg]`).

**H-09 — Numero publicado que teve de ser retirado.** `a253706` publica 6,0 -> 8,9 fps para `VICTORY_INDIRECT_WINDOW`; `0b0074e` retira porque as duas execucoes mediram **cenas diferentes** (~4.300 vs ~1.550 draws/frame). *Natureza:* regressao de credibilidade, corrigida com retratacao explicita e uma comparacao melhor (reta de ms vs draws). E o unico caso que encontrei de retratacao — e ele vale mais que os numeros.

**H-10 — `e140f41` (HUD de 2 linhas) tocou 12 arquivos, incluindo `victory-formats/src/texture.rs` (+781 linhas) e criou `texture/generate.rs`/`patterns.rs`.** Nao e regressao por si: e o sintoma de um commit que mistura funcionalidade de tela com a geracao das tabelas de swizzle — **acoplamento que aumenta o custo de qualquer reversao futura do HUD**. *Confianca:* **PROVAVEL** (julgamento de engenharia, nao bug).

## I. Conhecimento reutilizavel

> Foco: o que e **exclusivo deste fork** (nao existe na base Marvisak nem, em parte, nos outros quatro forks) e o que **sobrevive a um repositorio unificado**.

**I-01 — Merge de escritas de CPU no device por mascara de bits (`9ceb7e7`, `shadow.rs` + `host.rs`).**
*Problema resolvido:* escritas da CPU e da GPU na mesma memoria, sem esperar a GPU. *Implementacao:* 1 bit por byte alterado por bloco de 64 KiB; compute `CARD_MERGE` (3 constantes de root, SRV titulo, SRV mascara, UAV cartao) com um thread por dword de mascara e `for k in 0..8` sobre 4 bytes; mascaramento em inteiros (`((four & 1) | ((four & 2) << 7) | ((four & 4) << 14) | ((four & 8) << 21)) * 255`). *Origem:* fxceless/victory-fork, importado em `9ceb7e7` (Martin Hartfelder). *Arquivos:* `crates/victory-gpu/src/{shadow,host,indirect}.rs`. *Dependencias:* SM 5.1 compute, fence por submissao, blocos de 64 KiB. *Conflitos provaveis:* qualquer outro mecanismo de "merge de runs" (`VICTORY_MERGE_GAP`, `VICTORY_MERGE_RUNS`) e o caminho de relay (`relay_land`), que precisa transportar as mascaras por numero de envio. *Riscos:* se a mascara e a copia divergirem de tamanho, bytes fora da mascara ficam do shader — correto — e bytes dentro ficam do titulo — correto; o risco real e o **relay**: `card_merge_relay` precisa excluir byte a byte. *Recomendacao:* portar como um par (mascara + shader) com os testes WARP de 3.000 runs aleatorios de 1-9 bytes cruzando dwords; nao portar pela metade.

**I-02 — Hash de 64 KiB com grupo de 64 threads + decisao por processo (`713a095`).**
*Problema:* comparar copias de cartao sem pagar o tempo de uma peca por passe. *Implementacao:* `CARD_COMPARE_WIDE` com `groupshared uint4 lanes[64]`, dobra na lane 0, `compare` em tres modos (armazena, compara, compara e armazena). *Origem:* fxceless (Martin Hartfelder). *Dependencias:* `OnceLock` de decisao (`card_hash_wide`), copias < 1 GiB, `groupshared`. *Conflitos:* qualquer hash de copia armazenado antes (`CARD_COMPARE`) — **incompativel por definicao**; a forma tem de ser escolhida uma vez e nunca misturada. *Riscos:* colisao de hash = peca nao buscada = dado velho no titulo; mitigado pela bijecao por passo. *Recomendacao:* portar os dois shaders juntos e manter o switch como **diagnostico**, nao como configuracao de usuario.

**I-03 — Sync de descriptor heap por fingerprint + comparacao de registros (`a26859f`, `9376fd6`, `213fa54`).**
*Problema:* o titulo copia registros como bytes; a API nao ve. *Implementacao:* write-watch por pagina; por slot `u64` de fingerprint (multiplica-e-dobra); hash de tabela (`Plain` hasher com `write_u64`) antes de comparar os 3 dwords; slots lidos por tabela ficam para os draws. *Origem:* fxceless (apos upstream 86a5ed6) + uiliannn (`213fa54`, de belobug `127830e` + romainhedouin `b41f3f3`). *Dependencias:* `watch::Tracked`, `Spans`, heaps *placed*. *Conflitos:* `SHADER_HEAPS` (RwLock) e a invariante de nao retomar o lock; qualquer codigo que escreva descriptors sem passar por `mirror`. *Riscos:* hash de tabela colidindo custa copia (aceitavel por desenho); o perigo e **nao comparar** os registros. *Recomendacao:* levar o trio completo; medir com o contador de "slots deixados aos draws".

**I-04 — Tabela `SLOT_VIEWS` (endereco/len/stride por slot) com stride nomeado (`b62bbc0`, `5603b45`).**
*Problema:* o console le o **endereco de um descritor** cujo indice e calculado em runtime; sem tabela, o shader nao tem como resolver. *Implementacao:* root SRV 252 com 4 dwords por slot; clamp `SLOT_VIEWS_COUNT-1`; `rawBufferLoad` de 4 componentes. *Origem:* uiliannn; correcao de stride por Lilith133T (`pr-7`). *Dependencias:* layout identico nos dois lados. *Conflitos:* `bindless.rs` da base (que este fork nunca teve — ver L-03) e qualquer tabela de views paralela. *Riscos:* **exatamente o que aconteceu** (8 vs 16 bytes). *Recomendacao:* portar com a constante unica e o teste de stride; e o unico jeito de manter o contrato.

**I-05 — Regra de invalidez de textura `Rule::Start` + `HEAD=4096` (`310c6eb`).**
*Problema:* texturas empacotadas com spans sobrepostos. *Implementacao:* invalidar so por "mesmo inicio" (ou dentro dos primeiros 4 KiB); modos `overlap`/`0` como diagnostico. *Origem:* fxceless. *Dependencias:* `Spans` (`BTreeMap` por inicio + `HashMap` por slot). *Conflitos:* qualquer logica que assuma spans disjuntos (comentarios antigos do proprio arquivo). *Riscos:* heuristica empirica de **um** titulo; em outro, `HEAD` precisa ser remedido. *Recomendacao:* portar o **mecanismo** (regra configuravel + contador de pares medidos), nao o numero 4096 como verdade.

**I-06 — Rechecagem agendada de textura indexada (`a803c87`).**
*Problema:* streaming preenche a memoria depois da view. *Implementacao:* fingerprint de 48 KiB nos frames 4/12/40/120/400; reenvia e recomeca. *Origem:* fxceless. *Dependencias:* `TextureQueue`. *Conflitos:* `VICTORY_PAGE_WRITES=slots` (mecanismo alternativo do mesmo fork, que re-enfileira texturas *placed*). *Riscos:* os dois mecanismos juntos podem reenviar demais; medir. *Recomendacao:* portar com fingerprint **maior ou por pagina suja**, corrigindo o gap declarado.

**I-07 — `ambient.rs`: luz ambiente sombreada por profundidade como *fallback* de RTGI (`d0af132`, `692bce4`).**
*Problema:* sem RTGI, tudo que o sol nao bate e preto (interiores, lado sombreado do rosto) — e isso e **a maior parte da imagem**. *Implementacao:* apos o programa `CS_ScreenProbes_TemporalResolve*`, substitui a saida (R9G9B9E5 em inteiros) por uma constante sombreada por oclusao de profundidade; `VICTORY_AMBIENT`, `_TINT`, `_SHADE`, `_FILE` (le `level r g b [strength radius near tan_y] [ceiling floor]` duas vezes por segundo em builds de diagnostico). *Origem:* Belobug (`0603123..cd9f78c`), gancho de historico de reflexao de `44dcd43`. *Dependencias:* o programa do titulo existir e escrever naquele slot de tabela (root 1, posicao 176); `resource::host_made` sem gate de feature. *Conflitos:* qualquer porte de iluminacao que escreva no mesmo alvo; `ceiling/floor` de reflexao precisa ficar opt-in (o proprio commit diz). *Riscos:* **substitui** a saida do titulo — e uma aproximacao declarada, nao uma correcao; e o tipo de porte que muda a imagem globalmente. *Recomendacao:* levar como paliativo nomeado, com o switch de desligar, e remover quando o RTGI funcionar.

**I-08 — HUD por fonte bitmap + anel de upload (`ca8829f`, `e140f41`).**
*Problema:* o contador de FPS precisa estar **na imagem** (captura de stream nao leva a legenda da janela), sem biblioteca de fontes e sem tocar o frame do titulo. *Implementacao:* fonte 5x7 propria; texto desenhado na CPU; buffer feito **uma vez por texto** e nunca reescrito (um present em voo le o que foi gravado); anel de 4 slots para nao vazar memoria. *Origem:* fxceless (base), expandido por uiliannn (2 linhas, VRAM, GPU). *Dependencias:* copia para o back buffer antes do `Present`; nome do adaptador. *Conflitos:* captura de frame (`shot.rs`) e qualquer coisa que leia o back buffer. *Riscos:* baixo; o desenho e no back buffer, nao no frame do titulo. *Recomendacao:* portar — e 300 linhas que substituem uma dependencia inteira de UI.

**I-09 — Captura de frame embutida (`shot.rs`, 1.016 linhas, `79cfdd9`).**
*Problema:* RenderDoc 1.45 carregado por hook *"did not survive here"*, entao nao havia como ver o frame. *Implementacao:* grava comandos de um frame, o relogio do cartao antes de cada, e a imagem que cada passe deixou; sem carregar biblioteca e sem device envolvido; imagens em partes quando falta memoria. *Origem:* fxceless. *Dependencias:* `VICTORY_SHOT`; `capture::op` desviando linhas. *Conflitos:* `framelog`/`cardmeter` (instrumentacao concorrente). *Riscos:* custo de memoria no pior momento (por isso "em partes"). *Recomendacao:* portar como **ferramenta**, nao como feature de produto; e a unica forma de diagnostico de frame que funcionou neste ambiente.

**I-10 — `test_skip` com `VICTORY_STRICT_TESTS` (`b4180ea`).**
*Problema:* testes que exigem GPU/compilador eram "pulados" com `eprintln!`, indistinguiveis de sucesso. *Implementacao:* `test_skip::skip(msg)` imprime `VICTORY TEST SKIPPED` e, com `VICTORY_STRICT_TESTS=1`, **falha**; `Host::new` loga o adaptador uma vez. *Origem:* Luiz Gustavo Benicio Neves (`#32`/`#43`). *Dependencias:* nenhuma. *Conflitos:* nenhum. *Riscos:* nenhum. *Recomendacao:* portar para qualquer suite com testes dependentes de hardware — resolve o "verde que nao testou nada".

**I-11 — CI duplo (Windows completo + Linux portatil) e `cargo xtask ci` (`d52d84c`, `defaac8`).**
*Problema:* o tradutor de shaders e a parte mais ativa e nao precisa de Windows; contribuidores sem Windows ficavam de fora. *Implementacao:* job `portable` em `ubuntu-latest` com `-p victory-core -p victory-pe -p victory-pipeline -p victory-shader -p victory-formats -p victory-game -p xtask` e `clippy -D warnings`; `xtask ci` roda o mesmo localmente, e no Linux tambem **type-checka** `x86_64-pc-windows-msvc` com `CARGO_FEATURE_PURE=1` para o blake3. *Origem:* Anderson Florentino da Silva (`#3`), estendido por lcg007xd-c (`#77`). *Dependencias:* `cfg(windows)` correto nos testes. *Conflitos:* testes que parseiam o proprio binario como PE (resolvido com `#[cfg(windows)]`). *Recomendacao:* portar sem hesitar; foi o que pegou H-08.

**I-12 — Warm cache de shaders (`a6a8d39`, `ef30575`, `d8b281a`, `a393a19`).**
*Problema:* primeira execucao paga a traducao (minutos de pop-in). *Implementacao:* `victory warm <dir>` descobre containers acessiveis, corta cada um ao extent declarado, deduplica por hash completo e grava pelo writer do runtime; `victory translate --warm <arquivo|all>` aceita lista de hashes de 16 digitos com `--jobs`/`VICTORY_WARM_SECONDS`; `VICTORY_HOST_WAVE`/`VICTORY_VIEW_LIMIT` aplicados ao warm (`#24`). *Dependencias:* chave de cache do F-09 (**sem ela, aquecer e inutil**). *Conflitos:* `--warm` nao aplica o recorte `Container::bytes_of` que `victory warm` aplica — as duas rotas **nao** cobrem as mesmas entradas (documentado em `docs/precompilacao-shaders.md`, corrigido por `a393a19`). *Riscos:* aquecer com chave errada produz cache que nunca acerta. *Recomendacao:* portar as duas rotas **e** o documento que explica a diferenca.

## J. Conclusao tecnica

Este repositorio e um caso raro: **373 commits em 3 dias**, dos quais 107 sao merges, produzindo 135 mil linhas de Rust que fazem um titulo de console rodar em Windows. O valor da analise nao esta no volume, esta na **forma**:

1. **A granularidade historica foi destruida por decisao tatica.** `578d459` (61 arquivos, +32.334 linhas) e `8e46dce`, `b401954`, `dfd22d1`, `4e038bf` trouxeram forks inteiros como commits unicos. Isso foi eficiente para integrar e e **catastrofico para auditar**: a maior parte dos defeitos desta secao tem como "commit introdutor" um import, nao uma mudanca. O proprio repositorio reconhece o problema por outro angulo: `docs/performance-architecture.md` precisa citar *issues* e *PRs* externos para reconstruir a proveniencia que o git nao guarda.

2. **A engenharia de desempenho e seria e mensurada.** As maiores vitorias nao sao "otimizacoes": sao **remocoes de trabalho estruturalmente errado** — hashear 64 KiB com um thread (F-02), comparar registros com BLAKE3 (E-06), esperar o cartao para fundir bytes (F-01), olhar 300 mil registros por frame quando 40 mil mudam (F-03). Em todos os casos o padrao e o mesmo: **o passe existia para descobrir o que mudou, e o custo de descobrir excedia o custo de fazer**. Esse e o conhecimento reutilizavel de maior valor aqui.

3. **Onde nao ha medicao, ha retorica — e o repositorio corrige isso uma vez.** `a253706` publicou 6,0 -> 8,9 fps; `0b0074e` retirou porque as cenas eram diferentes. Uma retratacao explicita em 373 commits e um sinal melhor que dez numeros. Mas a mesma arvore mantem **"zero TDR"** ao lado de um aviso de que o TDR pode acontecer (D-21), e duas contagens diferentes de programas de BVH — ou seja, a disciplina de medir nao se transferiu para a de documentar.

4. **A dívida real nao e de codigo, e de contrato.** `d16` tratado como f16 (D-17), dispatches acima de 65.535 grupos truncados (G-06), texturas com mips interiores invisiveis (G-07), fingerprint de 48 KiB (G-05): sao **trocas conscientes** que ficaram sem registro de decisao. Em um repositorio unificado, cada uma delas e uma pergunta que alguem vai refazer do zero.

5. **Para um repositorio unificado, este fork e a fonte certa de tres coisas e a errada de uma.** Certo: os mecanismos de cartao (F-01, F-02), o sync de descriptor por fingerprint (F-03), e a infraestrutura de medicao/diagnostico (`framelog`, `cardmeter`, `drvwatch`, `shot`, `profile`). Errado: **o modelo de integracao** — importar forks como commits unicos, sem historia, sem PR rastreavel, sem atribuicao verificavel. Se ha uma licao a levar, e que a mesma equipe que mediu 752 ms/frame com precisao de commit nao tinha, no mesmo repositorio, como dizer de onde veio a linha de codigo que causou o problema.

**Veredito.** Projeto tecnicamente avancado (tradutor GFX10->DXIL com 99,9% de cobertura declarada e um conjunto de tecnicas de espelhamento CPU/GPU acima da media), operacionalmente fragil (device removals sem causa, RTGI ausente, primeiro frame inconsistente) e historicamente opaco (historia reescrita, imports monolíticos, numeros de PR de tres repositorios misturados). Serve muito bem como **fonte de patches**; serve mal como **fonte de historia**.

## K. Indice de commits

373 commits, em ordem cronologica (mais antigo primeiro): `hash curto | data | assunto`.

```
f4504f6 2026-11-23 Initial commit
12caefe 2026-11-23 GFX10 operand fields and SP3 text; FSSM wave size
754619f 2026-11-23 Run Minecraft: runtime HLE, D3D12.X device on the host GPU, shader IR
d032b8a 2026-11-23 Title DLLs at run time: find by name, map on first LoadLibrary
4b25269 2026-11-23 Thread notices for title images
bc30709 2026-11-23 Thread notices: leave out threads Rust's std starts
2849477 2026-11-23 Run Grand Theft Auto VI from first boot into the world
015d45b 2026-11-23 README: status, subsystem table, how to run, debugging switches
c21208c 2026-11-23 Loader: keep the cleared TLS callback pointer out of base relocations
e8f3b1c 2026-11-23 Merge pull request #4 from Exorion1er/fix/tls-callbacks-reloc
934e67f 2026-11-23 loader: emulate AMD-only instructions on Intel CPUs
005ad49 2026-11-24 xma: stop at a zero-length frame instead of looping forever
d4ed2de 2026-11-23 Process exit: start the watchdog before other threads end
d348b9a 2026-11-24 Merge pull request #10 from ismailchaka1/emulate-amd-only-instructions
e6a2c54 2026-11-24 texture: refuse extents past 16384 texels instead of overflowing
36eb0b4 2026-11-23 Shaders: med3 with NaN, fract below 1.0, signed bit fields past bit 31
0b499d7 2026-11-24 Shaders: sine and cosine in turns, exact zeros at half and quarter turns
8c26061 2026-11-24 GPU: the world stays up; indirect execution behind a switch
3827767 2026-11-24 GPU: make tables past a capped host view heap afresh
1dd7afd 2026-11-24 Merge pull request #14 from 9hvn9yt2vk-netizen/fix-xma-zero-length
e98d3ca 2026-11-23 PE: checked file offset in rva_to_file_offset
d85097a 2026-11-23 Keep an empty import descriptor for the Windows 10 loader
29b558c 2026-11-23 test(gpu): use the SDK root parameter in the serializer fixture
9e04419 2026-11-24 Merge pull request #15 from 9hvn9yt2vk-netizen/fix-texture-extents
0c91c03 2026-11-23 Loader: test the AMD-only instruction emulation on any CPU
d222b87 2026-11-24 Merge pull request #19 from raffclar/exit-watchdog
97c78d1 2026-11-24 Merge pull request #21 from Nostyxx/fix/nvidia-view-heap-cap
f00caec 2026-11-23 Loader: test 64-bit extrq at index 0, not in AMD's undefined range
0b607b1 2026-11-23 GPU: verify WARP adapter is software before using it
ea24ea1 2026-11-24 shader: a lane mask that needs its own value fails instead of overflowing the stack
71cfe84 2026-11-23 loader: emulate clwb on Intel CPUs before Ice Lake
15b0091 2026-11-24 shader: VICTORY_SHADER_LOOP_LIMIT to find what makes world frames slow
e158d95 2026-11-24 Guard texture and XMA offsets against overflow
4cca767 2026-11-24 Reject empty XMA test fixture directories
7e42fef 2026-11-23 Report Windows commit-limit failures in crashes.log
ef22723 2026-11-24 PE: remove leftover dbg_tls test and check layout offsets
2e5001c 2026-11-24 shader: keep fract(NaN) as NaN on NVIDIA and add synthetic GPU VALU tests
4da3d63 2026-11-23 Clear the IAT directory alongside the import directory
5fbea47 2026-11-24 shader: look up an input channel without underflowing
071487f 2026-11-24 GPU: emit null buffer SRV/UAV when descriptor names a buffer with null resource
0c56091 2026-11-24 Shaders: lower v_xnor_b32 and v_swap_b32
4e94c2b 2026-11-24 xmem: write DirectStorage page destinations one view per run
fc807b6 2026-11-23 Reuse texture row spans when converting Xbox pixel layouts
35b054e 2026-11-24 GPU: pipelines off the render thread and cached; indirect execution on
f3c7f0f 2026-11-24 GPU: leave rectangle vertex components past the stride alone
25a3e95 2026-11-23 Keep an empty import descriptor so Windows assigns static TLS
78b8404 2026-11-23 xmem: commit physical-page sections whole under Wine
0eca412 2026-11-24 GPU diagnostics: skip pipelines by shader hash
c4e915f 2026-11-23 loader: a clwb write-back loop runs in one exception
9a5cbbe 2026-11-24 fix: skip unused export address slots instead of reporting them at RVA 0
4c1766c 2026-11-24 GPU: keep a guarded block's write fault from being declined in a race
73d776b 2026-11-24 GPU: check a sampled texture once per submission, not once per list
c963b71 2026-11-24 fix: read a far jump's offset as signed so backward far jumps decode
a2f5f43 2026-11-24 Shaders: set SCC from the carry of s_addc_u32 and s_subb_u32
b304b7d 2026-11-23 fix(gpu): average copy statistics over the full reporting interval
257d8bb 2026-11-24 Audio: reduce the title's XMA read advance before adding it
ec0d155 2026-11-24 Watch: saturate the end of announced writes and remaps
38ae55d 2026-11-24 Keep the diagnostic skip outside indirect argument validation
0e5953d 2026-11-23 Merge branch 'main' into fix/windows10-import-directory
3e982f7 2026-11-24 BCPack: find a slot's bucket from a window table
7d33c46 2026-11-24 Merge pull request #45 from HenriqueSBueno/clwb
27b530e 2026-11-24 Files: send rewritten narrow paths to the host, not to our wide wrappers
27e6442 2026-11-24 Merge pull request #28 from ifBars/fix/windows10-import-directory
24480a7 2026-11-24 Shaders: find the written instructions with a worklist, not repeated passes
cbd71a9 2026-11-24 Merge pull request #38 from raffclar/emulation-tests
cc51927 2026-11-24 Merge pull request #30 from AndyLatimer/fix/pe-rva-offset-overflow
bf6c557 2026-11-24 Merge pull request #75 from tedin7/vic-pe-export-holes
062ad14 2026-11-24 Merge pull request #50 from Vanthys/fix/format-offset-overflow
a6888a6 2026-11-24 Merge pull request #43 from 9hvn9yt2vk-netizen/shader-mask-cycle
8fe0b74 2026-11-24 Merge pull request #56 from 9hvn9yt2vk-netizen/find-input-underflow
4a8a058 2026-11-24 Merge pull request #58 from Yaekai/feat/shader-xnor-swap
46610eb 2026-11-24 Merge pull request #81 from tedin7/vic-scalar-carry
4926f72 2026-11-24 Merge pull request #80 from tedin7/vic-far-jump-backward
237f2e8 2026-11-24 Merge pull request #37 from alessiodeia/fix/valu-med3-fract-trig
3c86053 2026-11-24 Shaders: lower s_bitcmp0/1_b32 and s_bitcmp0/1_b64
eda38aa 2026-11-24 Merge pull request #57 from Yaekai/fix/null-buffer-views
683278b 2026-11-24 Merge pull request #36 from jeremygold02/codex/root-parameter-fixture
fc0c4dd 2026-11-24 Merge pull request #77 from Exorion1er/fix/guard-fault-race
0f35dfc 2026-11-24 Merge pull request #66 from AndyLatimer/fix/gpu-rect-element-past-stride
fa5ef73 2026-11-24 Merge pull request #79 from TG0000/perf/texture-check-once-per-submission
93bc978 2026-11-24 Merge pull request #60 from ifBars/perf/reuse-texture-row-spans
97464ba 2026-11-24 Merge pull request #61 from TG0000/hle/physical-page-runs
6168fc6 2026-11-24 Merge pull request #70 from FIocker/gpu/diagnostic-pipeline-filter
91e9bc9 2026-11-24 Merge pull request #46 from 9hvn9yt2vk-netizen/shader-loop-limit
4c0bb89 2026-11-24 Merge pull request #41 from LaunchDoesntCode/codex/proton-warp-safety
8a8a38d 2026-11-24 Merge pull request #53 from a55uka/fix/wine-physical-pages
9aef3d9 2026-11-24 Merge pull request #51 from BaggyALPHA/fix/windows-commit-limit-diagnostic
7102a49 2026-11-23 GPU: rename the second test module in host.rs so the tests compile
1823079 2026-11-24 Shaders: lower v_cmp_class_f16 and v_cmpx_class_f16
d2a4bae 2026-11-24 Merge pull request #129 from Luislariosss/fix/duplicate-test-module
7d73e2b 2026-11-24 pipeline: remove staging files when cache publication fails
02fcf72 2026-11-24 xmem: mapping or unmapping no pages changes nothing
babe2db 2026-11-24 Crash reports: show the bytes of an instruction the CPU refused
3390e47 2026-11-24 Crash reports: say what an access violation's address holds
cbb50b8 2026-11-23 GPU: give null texture UAVs the title's view dimension
c88b7ac 2026-11-24 Game saves: an empty configuration id is the default, not every configuration's folder
afc4641 2026-11-24 net: skip newer-Windows-only exports in only_loopback_is_reachable
18b2716 2026-11-24 net: rustfmt the presence assertion
d46b078 2026-11-24 XUser: a gamertag without a suffix has an empty modern suffix
5f670bb 2026-11-24 Loader: a trapped unimplemented import goes into the crash log
36d6a79 2026-11-24 XTaskQueue: terminating a queue ends a dispatch waiting on it
79fb722 2026-11-24 Shaders: s_ttracedata_imm is SOPP opcode 0x28, not 0x26
63df363 2026-11-24 GPU: a half-float rounding carry raises the exponent, not halves the value
717f322 2026-11-24 GPU: GetDevice and the DXGI device hand out the device at every level
5f29f3a 2026-11-24 Loader: a title DLL that fails to prepare is named in the error
9fe73ba 2026-11-23 shader: decode the packed typed-buffer formats
a3d3bc9 2026-11-24 Shaders: trace descriptors chosen by s_cselect and s_cmov
ffa6635 2026-11-24 Shaders: lower waterfall loops over per-lane descriptors in one pass
b5c6678 2026-11-24 loader: unwind a 16-byte aligned copy of the context
3c81420 2026-11-24 GPU: drop draws whose fresh tables cannot be made; 2^18 dynamic slots
cc63aa2 2026-11-23 Config: ignore an empty VICTORY_TEST_TITLES
40c127c 2026-11-23 Chore: clear two workspace Clippy errors
efb3e3d 2026-11-24 GPU diagnostics: order tied drop counts consistently
376b3cf 2026-11-23 Storage: a path buffer too small fails with E_FAIL, as on Xbox
a4e7f1d 2026-11-24 GPU: replay pending descriptors before publishing the host heap
0f369d6 2026-11-23 Capture: R16G16B16A16_UNORM targets are integers, format 89 is 10:10:10:2
ded334b 2026-11-23 Shaders: lower v_cvt_norm_i16_f16 and v_cvt_norm_u16_f16
404d7fd 2026-11-24 Shaders: PSV0 says a pixel shader runs per sample when an input does
c2f60d6 2026-11-24 Shaders: a pixel shader writing SV_StencilRef declares StencilRef
2af66ff 2026-11-23 Shaders: v_cvt_u16_f16 and v_cvt_i16_f16 saturate at 16 bits
ff8694a 2026-11-23 Window: keep Alt and F10 from freezing the title in menu mode
eb30a0d 2026-11-24 GPU: write half subnormals from f32 rectangle outputs
28c1b07 2026-11-24 XGameSave: a read result into a small buffer is E_GS_PROVIDED_BUFFER_TOO_SMALL
d5704aa 2026-11-24 fix(shader): lower dynamic v_perm_b32 without INT64
f0ee35e 2026-11-24 XGameSave: a full host disk is E_GS_OUT_OF_LOCAL_STORAGE, not the quota
f4438b3 2026-11-23 Shaders: reads of uninitialized SGPRs read as zero, as VGPRs do
1106f43 2026-11-24 Shader tests: run the VALU GPU tests one at a time
077ec8b 2026-11-24 translate: count per-lane fetched descriptors as loaded
bf84134 2026-11-24 Shaders: maximums and minimums order -0 below +0, as RDNA2 does
e8d3b64 2026-11-24 GPU: the larger dynamic range only where the heap still fits
6285e19 2026-11-24 Shaders: take the first lane's descriptor as each lane's own only in a waterfall loop
6638098 2026-11-24 Shaders: per-lane descriptors chosen by control flow
c249f02 2026-11-23 Shaders: centroid position with conservative depth
9a8908c 2026-11-24 core: show data_dir in config and check its asset policy
25b0f88 2026-11-24 Test probe: compare direct and joined reads of a shader timestamp
3dd467c 2026-11-23 Shaders: a phi that only another read phi consumes is live too
260611a 2026-11-23 Shaders: image_sample_o with constant offsets
a0efd2d 2026-11-24 Shaders: decode GFX10 packed inline constants by operand type
b1c8306 2026-11-24 Shaders: prove EXEC selection before collapsing a waterfall
6c63dea 2026-11-24 Shaders: name why a loop is no waterfall, and test the shapes found in review
d74558c 2026-11-24 GPU diagnostics: describe a buffer UAV by its range, not as uav None
764fa19 2026-11-23 Loader tests: build the probe fixture under CARGO_TARGET_TMPDIR
0064c08 2026-11-24 GPU: upload CPU-written textures before CopyResource
af22e5d 2026-11-23 GPU: use the host upload path for texture staging copies
5864aab 2026-11-23 Docs: rustdoc builds the loader, GPU and shader crates without warnings
31ce30f 2026-11-24 GPU: unbind vertex buffer slots for null view arrays
ab3eb92 2026-11-23 Shaders: gate the constant_color test helper to Windows
4f53d6f 2026-11-24 GPU: upload CPU-written sources before texture region copies
0f0c872 2026-11-23 Docs: drop the private intra-doc link to MICRO in texture.rs
51f1627 2026-11-24 Shaders: v_cmp_lg is ordered on AMD hosts too (olt | ogt, not fcmp one)
3e39eac 2026-11-23 fix(hle): return XAsync payloads for nonnegative results
1f858b0 2026-11-24 Merge pull request #25 from artor410/packed-buffer-formats
c2e7254 2026-11-24 Merge pull request #87 from jeremygold02/codex/copy-stat-frame-window
0e990e4 2026-11-24 Merge pull request #104 from TG0000/perf/bitcode-worklist-liveness
b0328ba 2026-11-24 Merge pull request #111 from Dufftwt/fix/narrow-paths-translated-twice
3d77d5e 2026-11-24 Merge pull request #117 from tedin7/vic-s-bitcmp-2
80c0d12 2026-11-24 Merge pull request #140 from lordon1a/feat/cmp-class-f16
ba7d873 2026-11-24 Merge pull request #152 from Makto94/fix/net-test-win10
787f665 2026-11-24 Merge pull request #161 from lordon1a/fix/pipeline-staging-cleanup
13ac0e6 2026-11-24 Merge pull request #167 from Maraco91/xmem/zero-length-unmap
3dd37e6 2026-11-24 Merge pull request #198 from Dufftwt/fix/crash-report-instruction-bytes
27c960b 2026-11-24 Merge pull request #204 from Dufftwt/fix/crash-report-fault-memory
6af0fae 2026-11-24 Merge pull request #205 from asdaasdsaddf/fix/null-uav-dimension
27f9b76 2026-11-24 Merge pull request #209 from Maraco91/xgamesave/empty-configuration-id
7c8cecb 2026-11-24 Merge pull request #229 from Dufftwt/fix/gamertag-modern-suffix-empty
9e44f38 2026-11-24 Merge pull request #239 from Maraco91/xtaskqueue/terminate-ends-dispatch
56d5032 2026-11-24 Merge pull request #245 from Dufftwt/fix/import-trap-in-crash-log
88b6658 2026-11-24 Merge pull request #249 from Maraco91/gpu/get-device-newer-levels
89e1031 2026-11-24 Merge pull request #260 from Dufftwt/fix/f32-to-half-rounding-carry
db89843 2026-11-24 Merge pull request #276 from Dufftwt/fix/s-ttracedata-imm-opcode
93c0852 2026-11-24 Merge pull request #285 from Dufftwt/fix/name-title-dll-that-fails-to-prepare
5a16e66 2026-11-24 Merge pull request #313 from Abdellah-Khanani/loader-aligned-unwind
edf64e2 2026-11-24 Merge pull request #336 from hophaver/fix/dynamic-descriptor-exhaustion
533b7e2 2026-11-24 Merge pull request #341 from enzoduarte1/config/empty-test-titles-env
2745589 2026-11-24 Merge pull request #355 from enzoduarte1/clippy/clean-workspace-lints
42361b0 2026-11-24 Merge pull request #356 from FIocker/gpu/stable-drop-rank
7374bfb 2026-11-24 Merge pull request #365 from FIocker/gpu/publish-initialized-descriptor-heap
a7e60cf 2026-11-24 Merge upstream main (42361b0) into shader/per-lane-phi
40b7ed1 2026-11-24 Merge pull request #371 from marquezinii/fix/pls-path-small-buffer
f67c50c 2026-11-24 Merge pull request #372 from marquezinii/fix/capture-unorm16-formats
ebf3615 2026-11-24 Merge pull request #376 from LouisRiverstone/shader-cvt-norm-f16
c30fc99 2026-11-24 Merge pull request #381 from Dufftwt/fix/psv-sample-frequency
faba64c 2026-11-24 Merge pull request #382 from Dufftwt/fix/stencil-ref-shader-flag
2e4e280 2026-11-24 Merge pull request #384 from LouisRiverstone/shader-cvt16-saturate
3c86238 2026-11-24 Merge pull request #418 from enzoduarte1/gpu/half-subnormal-output
dd273df 2026-11-24 Merge pull request #419 from itsRevela/fix/swallow-keyboard-menu
9f18eb4 2026-11-24 Merge pull request #423 from pedrofort/shader-perm-no-int64
f0894c2 2026-11-24 Merge pull request #429 from marquezinii/fix/xgamesave-documented-errors
ce42e77 2026-11-24 Merge pull request #432 from Proudnoob4/fix/uninitialized-sgprs
0ddbce8 2026-11-24 Merge pull request #433 from r4tedx/tests/serialise-valu-gpu-tests
dd54de2 2026-11-24 Merge pull request #448 from Dufftwt/fix/min-max-signed-zeros
fd92558 2026-11-24 Merge pull request #520 from Quat3rnion/victory-xasync-payload
a4d7b16 2026-11-24 Merge pull request #565 from SAitken91/shader/conservative-depth-position
204bdcc 2026-11-24 Merge pull request #611 from AdvaitPalve11/fix/config-show-data-dir
9a92ccf 2026-11-24 Merge pull request #612 from KoLolX3/fix/phi-liveness-transitive
6f0a10a 2026-11-24 Merge pull request #634 from SAitken91/shader/image-sample-o
cf0bb43 2026-11-24 Merge pull request #637 from AlanQiang0418/codex/gfx10-packed-inline-constants
b4b4110 2026-11-24 Merge pull request #654 from SantannaCarlos/fix/uav-describe
16bd577 2026-11-24 Merge pull request #666 from lameloball2813-ui/fix/load-probe-target-dir
6db02ea 2026-11-24 Merge pull request #671 from fuzeofficlal/fix/copyresource-cpu-texture-source
22700b9 2026-11-24 Merge pull request #675 from lameloball2813-ui/docs/rustdoc-private-links
8d23f07 2026-11-24 Merge pull request #680 from EatCodeSleep2/fix/wine-staging-upload-128
5cd7421 2026-11-24 Merge pull request #681 from fuzeofficlal/fix/vertex-buffer-null-unbind
02f404e 2026-11-24 Merge pull request #685 from catminou4/devin/constant-color-cfg-windows
a51bdfa 2026-11-24 Merge pull request #696 from catminou4/devin/formats-doc-micro-link
58fed37 2026-11-24 Merge pull request #706 from Lyrnc/fix/cmp-lg-ordered
2005e35 2026-11-24 Merge pull request #88 from AndyLatimer/fix/title-size-overflows
c7a5f49 2026-11-24 Merge pull request #93 from TG0000/perf/bcpack-bucket-lookup
8e79c80 2026-11-24 CI: clippy and tests on pull requests and main
e09dbb6 2026-11-24 Merge upstream main (8e79c80) into shader/per-lane-phi
74a9d33 2026-11-24 Shader tests: the waterfall tests take the GPU one at a time, with the VALU tests
ca48fbe 2026-11-24 Merge pull request #566 from r4tedx/shader/per-lane-phi
ffe310e 2026-11-24 Shaders: the console's waterfall loop over a first-lane table offset
279c90a 2026-11-24 gpu: track Shadow buffer D3D12 resource states across lists
6132a9e 2026-11-24 VItoriaBr: rename user-facing branding and add bilingual PT-BR/EN README with RTX 5060 Ti progress
dbff821 2026-11-24 Merge remote-tracking branch 'upstream/main'
2471a62 2026-11-24 shader: support MIMG a16 bias/clamp/offset/gather4_l and LDS byte/half ops
5943771 2026-11-24 shader: reconcile DXIL interpolation modes across elements sharing a signature row
a65d054 2026-11-24 gpu: recycle dynamic descriptor slots after synchronous execute and raise capped heap MIN_DYNAMIC to 2^17
521e7f6 2026-11-24 docs: update README with RTX 5060 Ti v2 shader metrics and NVIDIA/shader fixes
f39a3c0 2026-11-24 shader: fix clippy::collapsible_if in reconcile_row_interp
77b4b4c 2026-11-24 shader: use Cube grads=2 for deriv_dwords in MIMG image_sample_d_cl
a5b4c12 2026-11-24 gpu: skip DONE mutex lock in pipeline::make::wait when frame budget is exhausted
1ca533d 2026-11-24 gpu: zero out-of-range indirect DispatchMesh args and cap valid mesh groups at 16384
aeabb43 2026-11-24 docs: add YouTube live stream link to README
8a75883 2026-11-24 gpu: chunk large command-list submissions and tighten indirect budget to 2^24
bd33acc 2026-11-24 shader/gpu: bound mesh shader outputs/indices to max limits and raise MIN_DYNAMIC to 2^18
a13a719 2026-11-24 shader: bounds-check all LDS reads, writes, and atomics to match RDNA2 OOB semantics
1ece3a6 2026-11-24 shader: emit memory-only fence for s_barrier in non-post-dominating or loop blocks
578d459 2026-11-24 victory-gta6: this fork's changes on top of VIctory main (ffe310e)
911267c 2026-11-24 shader: cap mesh setMeshOutputCounts to thread count to prevent uninitialized primitives
6f99f75 2026-11-24 gpu: submit one command list per fence wait and cap indirect mesh dispatches at 4096 / 2^22 budget
808a209 2026-11-24 shader: omit SYNC_THREAD_GROUP on s_barrier reachable after a loop abort_block
783a957 2026-11-24 shader/docs: lower default LOOP_LIMIT to 4096 in release and update README with 3D world milestones
f95856d 2026-11-24 shader: handle packed f16 padding halves, cap mesh metadata limits to threads, and lower uniform constant masks
6a2d9f9 2026-11-24 Shadow and trace programs translate, lint clean, ray-tracing write-up
1e11726 2026-11-24 docs: update README shader/PSO metrics to v3 (22,089 DXIL / 21,751 PSOs)
d279e69 2026-11-24 Make host pipeline waits configurable and prewarm pending requests (#1)
004bf69 2026-11-24 Test fix for hosted runners, stencil diagnostics, and a stability warning
f68b1f0 2026-11-24 shader: resolve indexed buffer stride from Built V# descriptor bits and 1/2-byte raw strides
e9765a7 2026-11-24 Fix the device loss introduced by the wave64 update
7b0f736 2026-11-24 Shader tests: recognise the software adapter from the device's own adapter
ca78809 2026-11-24 shader: emit SM 6.6 64-bit image atomics on typed UAVs
5ddefe0 2026-11-24 docs: update README shader/PSO metrics to v5 (22,619 DXIL / 22,281 PSOs, +530)
1684989 2026-11-24 Shader clean-up passes; a test that needs a card skips without one
8e46dce 2026-11-24 merge: integrate fxceless/victory-gta6 (GPL-3.0) world rendering, relay/VRAM, and DXIL optimizer
3930989 2026-11-24 Fix mesh pipeline test on unsupported GPUs (#14)
90228a7 2026-11-25 Shaders: restore parked lane-mask pairs a few instructions apart; relative VGPR reads keep parked lanes (#11)
4e17478 2026-11-24 The game is no longer named in the tree
b3d2f13 2026-11-24 docs: update live stream link and 3D world rendering status (~3 fps, frame 3586, 0 errors)
3052431 2026-11-24 shader: lower v_cvt_f32_f64 in pure 32-bit IEEE-754 integer ops
1ac7518 2026-11-24 shader: emit Interp::Weighted for standalone or reused v_interp_p1 halves
c2b1370 2026-11-24 docs: update README shader/PSO metrics to v6 (25,629/25,758 = 99.5% DXIL, 25,240 PSOs)
9e49b8a 2026-11-24 Take the parked lane-mask change (#11) back out for now
2119b0e 2026-11-24 Put the parked lane-mask change (#11) back
2d3f43c 2026-11-24 merge: sync fxceless/victory-fork (2119b0e) parked lane-masks and remove title name from README
3bff182 2026-11-24 Take the test/fx-prs branch of asdaasdsaddf's fork (as one commit)
31714de 2026-11-24 Take the fix/amd-view-heap-clamp branch of triippywhale's fork (as one commit)
a803c87 2026-11-24 Textures picked by index are looked at again after their upload (characters' skin and hair without colour in most launches)
e1586fb 2026-11-24 merge: sync fxceless/victory-fork (a803c87) skin/hair indexed texture refresh and zlib-rs
c96b4f2 2026-11-24 The depth bounds test works (#82); faster defaults; textures are on the card before the character bake; one host copy per placed texture
d52d84c 2026-11-24 ci: run the portable crates' clippy and tests on Linux
8458c26 2026-11-24 docs: vis├úo geral did├ítica em PT-BR, com diagramas
c0b60a0 2026-11-24 shader: reconcile signature row interpolation and keep console element order
d5a7580 2026-11-24 docs: remove live stream banner from README
05bb4e6 2026-11-24 docs: update README shader/PSO metrics to v7 (25,662 DXIL / 25,266 PSOs, 25,588 validated)
ec86124 2026-11-24 gpu: log missing ray-tracing BVH builder shader hashes when disabled
a6a8d39 2026-11-24 feat: `victory warm` translates the title's shaders into the cache before a run
de7ea8e 2026-11-24 gpu: make Pending::shaders visible to the ray-tracing builder log
b62bbc0 2026-11-24 shader: read descriptor addresses chosen by a computed heap index (SLOT_VIEWS)
4f9a19d 2026-11-24 merge: sync fxceless/victory-fork (c96b4f2) depth bounds test, faster defaults, and early textures
18df163 2026-11-24 shader: calculateLOD leaves an array's slice coordinate undef
77ad907 2026-11-24 shader: use the view-address table when a used descriptor's address is computed
5603b45 2026-11-25 Fix byte stride for indexed view addresses
708b450 2026-11-24 HLE: load a host DLL outside the module cache's lock
467f168 2026-11-24 Audio: an XMA read block past the ring's end wraps instead of filling forever
a91ef70 2026-11-24 GPU: extend dynamic descriptor wraparound scan to prevent false exhaustion
d0d0d23 2026-11-24 xma: start the frame after a continuation at the next packet that has frames
32d7848 2026-11-24 GPU: hold SHADER_HEAPS across descriptor mirror and check query offsets
310c6eb 2026-11-24 Packed textures stay in the upload queue; a texture on the card is refreshed when the card is full
91cd9c9 2026-11-25 gpu: release relay copy-back ranges after line.written and notify_all
aa85bb0 2026-11-25 gpu: release relay copy-back ranges after line.written and notify_all
cbbca4c 2026-11-25 docs: update coverage to v9 (25,701 DXIL / 25,304 PSOs / 25,630 validated)
67bb579 2026-11-25 shader: pick between two immediate samplers with a static-sampler range
faca57e 2026-11-25 gpu: recheck time and byte budget before staging textures in heap_textures
2ec2aa2 2026-11-25 shader: keep the immediate-sampler range to non-pixel stages
4f6eefc 2026-11-25 docs: update coverage to v12 (25,704 DXIL / 25,307 PSOs / 25,633 validated)
3a9e747 2026-11-25 shader: document why the immediate-sampler range stays out of pixel shaders
c248547 2026-11-25 shader: select the immediate sampler with a join path phi (pixel shaders too)
b8634af 2026-11-25 docs: update coverage to v13 (25,741 DXIL / 25,344 PSOs / 25,670 validated)
8dd30ab 2026-11-25 A write through one view of a page is seen at its other views; page writes counted; levels on demand (off); warm-up; card memory numbers
40beb26 2026-11-25 fix(shader): enforce RDNA2 LDS bounds checks and add unit tests
0d1c4e0 2026-11-25 Formats: map console texture formats 189-191
dcc0305 2026-11-25 Formats: verify console color formats through texture creation
a26859f 2026-11-25 The first room renders right: byte-copied descriptor records are mirrored, views off their copy's elements get their own bytes, card room for the first world frame, levels on demand run on the card
9376fd6 2026-11-25 Frame rate, processor side: a cheap fingerprint in the descriptor sync, draw-table slots left to their draws, no failing write-watch calls; card fetches inside their flights (off by default)
4b9872a 2026-11-25 shader: lower packed dot products to DXIL dot4AddPacked and fmad
b401954 2026-11-25 merge: sync fxceless/victory-fork (9376fd6) first-room rendering, on-demand levels, and CPU FPS optimizations
7838d65 2026-11-25 Merge branch 'pr-11'
ac14e18 2026-11-25 Merge branch 'pr-10'
7e6670b 2026-11-25 Merge branch 'pr-12'
d396d11 2026-11-25 Merge branch 'pr-7'
ef30575 2026-11-25 merge: PR #6 (@andysilva86) victory warm shader cache pre-compilation
8a85222 2026-11-25 merge: PR #3 (@andysilva86) portable Linux CI and Portuguese overview docs
06161b9 2026-11-25 hle: remove duplicate warm_shader definition after PR #6 merge
18924c7 2026-11-25 docs: update README with victory warm, 6 fps RTX 5060 Ti preset, and tuning/RTGI findings
f85101b 2026-11-25 Frame rate, second night: a submit thread of the layer's own, a copy queue for fetches, indirect draw windows made safer, a profiler; all behind switches
8cb509b 2026-11-25 gpu: prioritize requested vertex graphics pipelines ahead of mesh/compute
a93b052 2026-11-25 shader: retry first-lane and lane-read refusals wave-wide
6dcd8cd 2026-11-25 fix(shader): lower MIMG offset/clamp/bias/LOD sample variants
fe0506a 2026-11-25 A helper lane's v_readfirstlane reads its own value (upstream 0601955)
2b3c339 2026-11-25 A pixel's quad is read through quad reads, helper lanes included: a derivative taken by hand at a triangle's edge was the whole value (lines on car paint, glass and skin)
a253706 2026-11-25 Frame rate, third session: a frame log and a card meter, the driver watch, a forbidden self-copy removed; draw windows measured at 6.0 -> 8.9 frames a second
0b0074e 2026-11-25 Copies in place measured and fixed, our threads on the fast cores, and a corrected comparison of draw windows
dfd22d1 2026-11-25 merge: sync fxceless/victory-fork (0b0074e) submit thread, copy queue, draw windows, in-place copies, and fast cores
f06ee8b 2026-11-25 Merge branch 'pr-15'
b37344f 2026-11-25 gpu: fix dynamic_allocation_finds_contiguous_range_crossing_search_cursor test
e7f9af5 2026-11-25 gpu: satisfy clippy manual_is_multiple_of in shadow.rs
d1da85a 2026-11-25 XGameSave: a stray or unreadable version folder never costs the save
009856f 2026-11-25 gpu: stop truncating dynamic descriptor wraparound scan at DYNAMIC
1d84afd 2026-11-25 gpu: skip depth-bounds draw assertions on WARP software adapter
a51c1ea 2026-11-25 shader: recover a wave64 mask restored from split scalar copies
d0af132 2026-11-25 gpu: port depth-shaded ambient light fallback from Belobug (GPL-3.0)
3973179 2026-11-25 What a frame is made of, measured; parts joined and copy-back through our own view (9.6 fps in the first rooms)
3e06462 2026-11-25 gpu: disable integer RT blending (Belobug 8629211), fix WARP CI check, update README
a858713 2026-11-25 docs: compare performance hypotheses across related forks (#20)
89434b7 2026-11-25 gpu: recover sparse interior-mip textures, coalesce remaps, and prioritize depth/compute pipelines (DexNom 921d8d3, GPL-3.0)
c931d12 2026-11-25 gpu: revert interior-mip VirtualQuery walk in mapped_bytes (752 ms/frame stall, 0 hits)
465b94f 2026-11-25 Scattered indirect records clamped once per list (VICTORY_INDIRECT_GATHER=1): 10.3 fps in the first rooms
e354b6a 2026-11-25 docs: document performance architecture and optimization opportunities (#19)
a393a19 2026-11-25 docs: clarify victory warm cache identity and log verification (#21)
d8b281a 2026-11-25 CLI: apply VICTORY_HOST_WAVE and VICTORY_VIEW_LIMIT to victory warm (#24)
a290636 2026-11-25 shader: port pixel quad/LOD fixes and guest write tracking (DexNom/VIctory 921d8d3, bc3cd4b; GPL-3.0-or-later)
bb52831 2026-11-25 Merge remote-tracking branch 'origin/main'
d5d9b7c 2026-11-25 shader+gpu: precise guest-memory write tracking, partial dispatch guard, and pixel quad sampling (DexNom bc3cd4b + 921d8d3, GPL-3.0)
633ba6e 2026-11-25 gpu: wire USER_DISPATCH_THREADS_X, DispatchX thread-count conversion, and memory::Ready.count (DexNom bc3cd4b + 921d8d3, GPL-3.0)
ed6e5af 2026-11-25 clamp host constant buffer views to 64 kib (#28)
ca8829f 2026-11-25 The frame rate in the corner of the picture; changed-bytes refresh of card copies (off)
8c8da76 2026-11-25 docs: update README with verified v15_turbo preset (QUEUE2, ASYNC_SUBMIT, COPY_PLAIN, PAGE_WRITES=slots)
fe3aa3f 2026-11-25 shader: prove a wave scan's first-lane value is uniform across the halves
5a84eb4 2026-11-25 gpu: reduce heap_buffers CPU overhead (64-frame rescan, cached spans, 4x fewer COM checks)
d510ee4 2026-11-25 README: the dips while moving, measured; two attempts on the title's page mapping, both taken back out
dd3a14b 2026-11-25 README: a frame capture built into the layer, copies of title memory in the card's own memory (fewer long frames in two launches of two), and what was tried and gave nothing
79cfdd9 2026-11-25 A frame capture built into the layer (VICTORY_SHOT), windows outside buffers in the card's memory as one buffer in a GPU upload heap (VICTORY_WINDOW_UPLOAD), and the day's smaller pieces
085ee7c 2026-11-25 README: the frame rate in the status header brought up to the measured runs
8b8585d 2026-11-25 fix(gpu,shader): fix WARP depth bounds, indirect budget group race, and Linux CI (#23)
043a06d 2026-11-25 hle: VICTORY_HIDDEN_WINDOW keeps the title's window off the screens for unattended runs (#31)
92e9b2c 2026-11-25 Tests: verify null heap clamp slot with small and exact-fit allocations (#35)
f2a2097 2026-11-25 Shaders: automatically guard divergent single-console-wave barriers on narrow-wave GPUs (#34)
213beb9 2026-11-25 shader: lower primitive-ordered (POPS) pixel shaders as rasterizer-ordered UAVs (fxceless/victory-fork #215) + a synthetic test (#36)
713a095 2026-11-25 gpu: hash card-copy pieces with a group of 64 threads (our card lists ~30 ms -> ~5 ms a frame) (#33)
9ceb7e7 2026-11-25 gpu: merge title bytes changed under in-flight shader writes on the card (no settle for them, frame ~10 % shorter) (#37)
4e038bf 2026-11-25 merge: sync fxceless/victory-fork 085ee7c (10.3 fps, VICTORY_INDIRECT_GATHER, fpsbox, VICTORY_SHOT, VICTORY_WINDOW_UPLOAD) + thin LTO (Belobug a23d1b4, GPL-3.0)
213fa54 2026-11-25 gpu: compare descriptor records after a quick table hash
692bce4 2026-11-25 gpu: add a configurable ambient floor for reflections
ca4759f 2026-11-25 shader: check the POPS shader container on every platform
bfa5f49 2026-11-25 XMA: noise tests, and end a frame at its data instead of looping forever (#40)
df2990f 2026-11-25 Shaders: a signed 64-bit operand sign-extends its 32-bit literal (#41)
30d9ba1 2026-11-25 config: reject a data_dir equal to or inside the cache_dir (#38) (#42)
b4180ea 2026-11-25 tests: name the host adapter and make skipped shader tests explicit (#32) (#43)
75aa531 2026-11-25 shader: decode and lower v_dot8_i32_i4 (#44)
5144818 2026-11-25 docs: add 8 GB AMD VRAM + HDD preset and tuning guide (from PR #22 by @ItzMeGamer)
8c0758d 2026-11-25 gpu: port VICTORY_TABLE_BUFFERS and VICTORY_SYNC_NULL_SKIP from myokex/victory-amd (c3723f3, GPL-3.0)
e140f41 2026-11-25 gpu: expand on-screen fpsbox into 2-line VItoriaBr performance HUD (FPS, ms, draws, res, VRAM, GPU)
ef7dad2 2026-11-25 gpu: document the texture readback and XOR layout provenance
69c665c 2026-11-25 docs: update README with v19 preset (TABLE_BUFFERS, TEXTURE_BACK, INDIRECT_GATHER, 2-line HUD, 5.8 fps in 4K)
5668b8b 2026-11-25 gpu: port VICTORY_TEXTURE_HEAPS and VICTORY_STAGING_BLOCKS from myokex/victory-amd (c3723f3, GPL-3.0)
081a89d 2026-11-25 formats: add Layout::bytes_from for partial mip-tail detiling (myokex c3723f3, GPL-3.0)
2b33e08 2026-11-25 gpu: fix Clippy 1.98 manual_is_multiple_of lint in texture.rs
defaac8 2026-11-25 xtask: `cargo xtask ci` runs what the CI workflow runs (#77)
448416c 2026-11-25 Shaders: v_permlane16/v_permlanex16 ask for the console's lanes (#79)
ddd5e6c 2026-11-25 test(shader): cover divergent FP16 wave64 reductions on wave32 hardware (#93)
b6e61ab 2026-11-26 GPU: mirror placed 3D texture UAVs and preserve their aliases (#72)
```

## L. Limitacoes desta analise

**L-01 — Sem rede.** Nao consultei GitHub. Nenhum numero de issue ou PR foi verificado na origem: os que aparecem (#59, #72, #79, #93, #870, #18, #13, #14, #16, #8, #9, #1014, #648, #670, #220, #305, #747, #82...) sao citacoes de mensagens de commit ou de `docs/`. Onde digo "PR #N", leia "numero como citado no repositorio".

**L-02 — Nenhuma execucao.** Nao compilei, nao rodei testes, nao joguei. Todos os numeros de desempenho (752 ms, 48,5 ms, 70-220 ms -> 3 ms, 10,3 fps, ~10%, 6,0 -> 8,9 fps) vem de mensagens de commit, README ou docs, e estao marcados como tal. Nao ha neste documento nenhuma medicao propria.

**L-03 — Comparacao com a base por conteudo, nao por historia.** O HEAD do Marvisak nao e ancestral deste fork. Seis arquivos que a base tem **nunca existiram** neste repositorio (`git log --all -- <path>` vazio para todos): `crates/victory-game/src/shaderpack.rs`, `crates/victory-gpu/src/{bindless,twin,external}.rs`, `crates/victory-hle/src/modules/fidelityfx.rs`, `crates/victory-shader/src/emit/lower/bvh.rs`. Nao sei se sao funcionalidades posteriores a este snapshot, se foram substituidas por outros modulos (o README fala de "bindless texture queue", que aqui parece viver em `descriptor/`), ou se foram descartadas. **NAO VERIFICADO.**

**L-04 — `git log` por caminho e enganoso neste repositorio.** `git rev-list HEAD -- crates/victory-gpu/src/texture.rs` retorna 15 commits e **omite** `89434b7` e `c931d12`, que comprovadamente tocam o arquivo (`git show --stat`), por simplificacao de historia (TREESAME) em torno dos merges de branches reescritas. Consequencia: **nao usei `git log -- path` para afirmar "quem introduziu"**; usei `git show --stat`, `git log -S` e leitura do HEAD. Ainda assim, para os defeitos cujo introdutor e um import de 32 mil linhas, "commit introdutor" significa "primeiro commit em que o codigo aparece", nao autoria.

**L-05 — Mensagens de commit foram tratadas como dados, nao como prova.** Quando a mensagem diz "medido X", registrei como PROVAVEL, nao CONFIRMADA. Onde a mensagem contradiz o codigo, dei prioridade ao codigo (foi assim que H-06 apareceu: `a91ef70` diz "prevent false exhaustion", mas o `take(DYNAMIC)` continuava truncando).

**L-06 — Cobertura de leitura.** Examinei em profundidade ~45 commits (o pacote de evidencias) mais ~25 por inspecao direta de diff/codigo. Os 373 estao no indice e foram lidos por assunto, mas **nao abri o diff de todos**. Em particular, a maior parte da onda de intake do upstream (fase 1 e 2a, ~150 commits de PRs de terceiros) foi classificada por assunto e, quando relevante, por `git show --stat`; seus defeitos so aparecem aqui se sobreviveram ate o HEAD.

**L-07 — Nao auditei o tradutor de shaders linha a linha.** 67 alteracoes em `lower.rs` (o arquivo mais mexido) foram tratadas por amostragem dirigida (LDS, MIMG/LOD, d16, permlane, dot8, cross-lane). Ha provavelmente defeitos de semantica de lane/wave que nao apareceram.

**L-08 — Sem hardware de referencia.** Nao pude distinguir "defeito do fork" de "limitacao do driver/GPU" nos casos de device removal (G-08) e na regressao AMD (G-09). Sem uma RTX 5060 Ti nem uma RX 6500 XT, essas fronteiras ficam abertas.

**L-09 — Documentacao de terceiros usada como fonte secundaria.** `docs/performance-architecture.md` e `docs/comparative-performance-hypotheses.md` sao revisoes de um contribuidor (Gustavo Petry, PRs #19/#20) com citacoes a issues externas. Usei-as para **localizar** problemas (d16, #18, #8/#9), sempre que possivel confirmando no codigo; onde nao confirmei, marquei NAO VERIFICADO.
