# VIctory - analise de DeepSeek V4.1 Flash

## A. Identificacao

| Campo | Valor |
|---|---|
| Nome | VIctory |
| Origem | `https://github.com/Marvisak/VIctory.git` (remote `origin`; clone local analisado em `repos\VIctory`) |
| Fork / original | Repositorio original do autor principal (`Marvis`, 130 dos 241 commits). Nao e fork declarado. |
| Branch analisada | `main` (unica branch local; `origin/HEAD -> origin/main`; 0 commits fora de HEAD) |
| Commit 0 | `f4504f6` - 2026-11-23 01:15 (+0100) - "Initial commit" (69 arquivos, 7.952 insercoes) |
| Ultimo commit | `70564ae` - 2026-11-24 22:36 (+0100) - "README: status of 2026-11-24" |
| Periodo coberto | 2026-11-23 01:15 a 2026-11-24 22:36 (1 dia e 21 horas) |
| Commits | 241 alcancaveis de HEAD, 91 merges, 88 numeros de PR distintos (#4 a #706) |
| Arquivos versionados | 174 |
| Autores | 49 nomes distintos (54 identidades nome+email) |
| Commits examinados em profundidade | 45 blocos do pacote de evidencias + ~45 commits adicionais abertos com `git show`/`git log -S` (todos os 91 merges foram mapeados para o commit de origem) |
| Modelo responsavel | DeepSeek V4.1 Flash (opencode-go) |
| Data da analise | 2026-11-25 |

Contexto do repositorio: e um projeto de engenharia de reimplementacao de plataforma, nao um aplicativo. Roda o executavel x86-64 de Grand Theft Auto VI (build Xbox Series X|S) inalterado dentro do proprio processo, substituindo a camada de plataforma (GDK/Xbox) por HLE, recompilando os shaders GFX10 (RDNA2) para DXIL e traduzindo a API D3D12 do console para D3D12 de PC. O historico tem dois dias e densidade de um projeto de meses: 241 commits, 42.518 linhas no commit 7, 91 merges (88 PRs externos) em ~20 horas.

## B. Resumo tecnico

**Proposito.** Executar o titulo (GTA VI, Series X|S) em Windows a partir de arquivos fornecidos pelo usuario, com o codigo do titulo rodando sem modificacao. Tudo abaixo dele e nosso: loader PE, runtime da plataforma, formatos, GPU, shaders.

**Workspace (crates).**

- `victory-core` - configuracao (`victory.toml`, variaveis `VICTORY_*`), diretorios de dados/cache.
- `victory-loader` - mapeia o executavel e as DLLs do titulo, resolve imports por thunks (Trap/Zero), crash handler, threads, TLS, emulacao de instrucoes AMD-only.
- `victory-pe` - parser PE, layout, realocacoes, exports/imports.
- `victory-hle` - a plataforma: `xmem` (memoria fisica/mapeamentos), `d3d12x`, `xgamesave`, `xuser`, `xasync`/`XTaskQueue`, `files` (drives virtuais), `audio`/XMA, `storage`, `net`, `lifecycle`, `fidelityfx`.
- `victory-shader` - decodificador GFX10, IR, estrutura de controle, raising de recursos, lowering para DXIL, cache.
- `victory-gpu` - D3D12 de PC: heaps de descritores bindless, views, sombras (copias host) da memoria do titulo, page table, indirect, pipelines, captura.
- `victory-formats` - texturas tiled (AddrLib-like), XMA, BCPack.
- `victory-pipeline` - biblioteca de pipelines em disco.
- `apps/victory` - CLI: `inspect`, `census`, `lib-exports`, `shader-info`, `translate [--cache --pipelines]`, `run`, `config`.
- `xtask` - politica de assets, hooks de commit.

**Mecanismos centrais** (de `docs/architecture.md` e dos comentarios de modulo):

1. **Shaders AOT.** `translate --cache` traduz todos os shaders que o titulo pode pedir para um cache (~1.3 GB); `--pipelines` pre-compila ~27.000 pipelines numa biblioteca de 4 MB. Sem isso, traducao no primeiro uso e draws esperam.
2. **Memoria do titulo como copia host.** Buffers e texturas tem copia host (`shadow`, `resource`), com diario de mudancas por pagina de 64 KiB (`watch`) alimentado por guards de escrita, escritas do kernel e mapeamentos aliasados.
3. **Views dirigidas por heap.** O titulo e quase todo bindless: shaders leem descritores em indices que vem de root constants ou de memoria. Cada slot escrito ganha uma view host; listas que indexam o heap tem os ranges atualizados na submissao (`bindless`).
4. **Page table de dois niveis** para shaders que calculam enderecos (BVH, tracers): ate 512 regioes de 256 MiB por tabela (`memory.rs:50`).
5. **Pipelines fora da render thread**, com biblioteca em disco e "warm start" dos pipelines desenhados na execucao anterior.
6. **Fidelidade de erro.** Modulos HLE retornam os HRESULT documentados do Xbox (nao os do PC), porque o titulo decide com base neles.

**Estado atual** (README do HEAD, `70564ae`): **nao jogavel**. Boota, roda menus a 30 fps, carrega o mundo (~50 s) e desenha a primeira missao. Frames de 1-2 s, superficies sem iluminacao, faixa de geometria distante ausente, personagens quebrados em alguns planos. Cobertura de traducao: mesh 8.765/8.765, vertex/pixel 12.550/12.592, compute 2.579/2.637. Testado em uma maquina (Windows 11, RX 6700 XT).

## C. Linha do tempo

**Fase 1 - Esqueleto (1-6, 2026-11-23 01:15-15:29).** `f4504f6` cria o workspace (69 arquivos: PE, HLE, shader decode). `12caefe` completa campos de operando GFX10 e texto SP3. `754619f` ("Run Minecraft") e o primeiro salto: HLE em runtime, device D3D12.X no host, IR de shader, 61 arquivos / 12.772 linhas. `d032b8a` resolve DLLs do titulo por nome no primeiro `LoadLibrary`; `4b25269`/`bc30709` instrumentam threads do titulo.

**Fase 2 - Bring-up do GTA VI (7-18, 23:10-00:12).** `2849477` ("Run Grand Theft Auto VI from first boot into the world") e o commit decisivo: 107 arquivos, 42.518 insercoes - heaps de descritores com slots dinamicos (`DYNAMIC = 2^16`), sombras de memoria do titulo, texturas, XMA, XGameSave, views por heap. No mesmo periodo entram os primeiros PRs externos (`e8f3b1c` #4, `d348b9a` #10, `1dd7afd` #14, `9e04419` #15, `d222b87` #19, `97c78d1` #21) e os primeiros lotes de fidelidade de shader (`36eb0b4`, `0b499d7`). `8c26061` mantem o mundo de pe por 240 s ao corrigir descritores nulos do tipo errado, e poe execucao indireta atras de switch.

**Fase 3 - Onda de PRs externos (19-91, 00:12-01:57).** 70 commits em menos de duas horas, quase todos correcoes: overflows de formato/XMA/textura, semantica de instrucoes (bitcmp, cmp_class, cvt_norm, cvt16 saturate), erros de HLE (XGameSave, storage, xmem), diagnosticos de crash, testes. E aqui que aparecem os defeitos de "arvore que nao compila" (modulo de teste duplicado, `lowers()` com aridade antiga). `35b054e` liga execucao indireta por padrao e move a criacao de pipelines para threads.

**Fase 4 - Correcao em massa por PR (92-207, 01:57-09:47).** Segunda onda, ainda maior: `#152` a `#706`. Shader: `v_cmp_lg`, `-0/+0` em min/max, carry de meio-float, `s_addc/s_subb`, far jump com sinal, `s_ttracedata_imm`, `v_perm_b32` sem INT64, formatos packed de buffer tipado. HLE: XTaskQueue, XAsync `S_FALSE`, gamertag, XGameSave (data loss por config id vazio), storage. GPU: upload de texturas escritas pela CPU antes de copias, unbind de vertex buffer nulo, staging sob Wine. Diagnostico: crash reports com bytes da instrucao e conteudo do endereco.

**Fase 5 - CI e o ramo per-lane (208-213, 09:47-10:53).** `8e79c80` cria CI (clippy `-D warnings` + `cargo test --workspace`) e no mesmo commit conserta o build de testes do `main`. O PR #566 (`ca48fbe`, ramo `shader/per-lane-phi`) traz a saga de descritores por lane: `ffa6635` colapsa loops waterfall, `6285e19` restringe o tracado a loops provados, `6638098` trata phis por lane, `b1c8306` prova a selecao de EXEC antes de colapsar, `6c63dea` melhora os diagnosticos. `ffe310e` estende o mesmo mecanismo ao waterfall do compilador do console (offset de tabela).

**Fase 6 - Empurrao final de GPU (214-241, 22:36 do dia 24).** Um lote importado de uma vez: FidelityFX no host, atomics de imagem 64-bit, interseccao de BVH, lane masks, strides bindless (`0b35b7f`); page table para memoria em endereco cru (`27c3dc9`); views que os shaders escolhem do heap com um diario unico (`c75fdba`); guarda contra descritor de textura lido como buffer (`d71c388`); uma textura host por memoria de textura e "twins" graváveis para depth (`13bbd6f`); custo de frame do mundo (`6d021fe`); warm-up de shaders e cache por tradutor (`c47b529`); pre-build de pipelines (`9c58a1c`). O README final (`70564ae`) declara o estado: o mundo e desenhado e mostrado, lento e sem iluminacao.

## D. Catalogo de defeitos

Cada item abaixo foi decidido **pelo diff**, nao pela mensagem. Quando a mensagem do commit e a unica fonte de um fato de campo (ex.: "3 de 3 execucoes falharam"), isso esta marcado. Nos itens sem a linha "Tentativas de correcao", o commit de correcao citado e a **primeira** tentativa: nao houve correcao anterior no historico.

---

### D-01 · Esgotamento de slots dinamicos de descritor fazia o draw ler o descriptor do titulo · remocao de device (#59)

- **Descricao / sintomas.** Remocao de device (DXGI_ERROR_DEVICE_REMOVED) cerca de 60 s depois de entrar no mundo, com falhas em `0x7000037xxxxx`, `0x8000037xxxxx` e `0x33c4000`, em um pipeline diferente a cada execucao. Apos a correcao, o sintoma equivalente passou a ser "draws faltando" em vez de falha de GPU.
- **Causa raiz.** `HostHeap::dynamic(n)` devolvia `None` quando o range de slots dinamicos (2^16) acabava. Em `State::tables`, o `continue` pulava a tabela nova e o user data do estagio continuava valendo o offset da tabela **do titulo**. Tabelas de constant buffers sao sempre feitas de novo e nunca espelhadas, entao o shader lia como constant buffer o que estivesse naquele slot host (descritor de textura ou nulo); na AMD, um descritor assim lido como buffer da um endereco base absurdo e o driver remove o device.
- **Introduzido por.** `2849477` (2026-11-23), que cria os slots dinamicos e o caminho de fallback silencioso.
- **Tentativas de correcao.** `3c81420` (2026-11-24, PR #336) faz `bind` devolver `false` e derrubar draw/dispatch/mesh/indirect sem tabela, e sobe `DYNAMIC` para 2^18. `e8d3b64` (mesmo PR) corrige um efeito colateral do anterior (ver H-01).
- **Correcao definitiva.** `edf64e2` (merge do PR #336, 2026-11-24) contendo `3c81420` + `e8d3b64`. O codigo atual mantem `table_failed` e o retorno booleano de `bind` (`crates/victory-gpu/src/draw.rs:509,578,1127`).
- **Arquivos.** `crates/victory-gpu/src/descriptor/host.rs`, `crates/victory-gpu/src/draw.rs`.
- **Evidencia.** `git show 3c81420` mostra `- self.bind(list, p, false);` -> `+ if !self.bind(list, p, false) { return false; }` e, em `tables`, `- continue` -> `+ self.table_failed = true;`. `git log -S"const DYNAMIC" -- crates/victory-gpu/src/descriptor/host.rs` devolve apenas `2849477`. `git show 3c81420^:crates/victory-gpu/src/descriptor/host.rs` contem `ONCE.call_once(|| tracing::warn!(n, "no free dynamic descriptor slots: a table is not made"))` seguido de `return None`.
- **Estado atual.** Corrigido. O custo e explicito: um draw sem tabela e descartado (geometria ausente) e o descarte e logado em potencias de dois. E uma das origens dos "draws dropped" que o README ainda reporta.
- **Confianca.** CONFIRMADA.

---

### D-02 · Heap de views limitado pelo host (teto de 1.000.000 da NVIDIA) · slot de tabela do titulo aliasava slot dinamico ou passava do fim · device hung

- **Descricao / sintomas.** Em NVIDIA, o titulo travava a GPU (`DXGI_ERROR_DEVICE_HUNG`) nos primeiros screens, depois de quadros pretos e com glitches, tipicamente 45-60 s apos o inicio (3 de 3 execucoes, segundo a mensagem). Em AMD nada acontecia.
- **Causa raiz.** O heap de views do titulo tem 1.245.184 slots; o host cai para `PORTABLE_VIEWS = 1_000_000`, cujo trecho do titulo termina em 934.464 e cujos slots dinamicos vem depois. O titulo usa um anel de tabelas por frame nos slots altos (934.464 a 1.245.184, andando 4.096 slots por frame), entao esses offsets ou apontavam para slots dinamicos (conteudo alheio) ou passavam do fim do heap.
- **Introduzido por.** `2849477`, no fallback de heap limitado (`(make(PORTABLE_VIEWS), PORTABLE_VIEWS - extra)`).
- **Tentativas de correcao.** Nenhuma anterior; o PR #21 e a primeira.
- **Correcao definitiva.** `3827767` (2026-11-24), merge `97c78d1` (PR #21). Toda tabela que termina depois dos slots de titulo do host e refeita em slots dinamicos, com o novo offset entregue ao shader; `HostHeap::count()` expoe o limite; `mirror()` loga o slot mais alto visto alem do heap.
- **Arquivos.** `crates/victory-gpu/src/descriptor/host.rs`, `crates/victory-gpu/src/draw.rs`.
- **Evidencia.** `git show 3827767` contem `+ if index >= host.count() { fresh.push(u.slot); }` em `draw.rs` e o novo `pub(crate) fn count()`. O merge `97c78d1` e "Merge pull request #21 from Nostyxx/fix/nvidia-view-heap-cap".
- **Estado atual.** Corrigido para o caso "tabela que passa do heap". **Residual documentado:** em `crates/victory-gpu/src/descriptor/host.rs:137` ainda esta escrito `// shortcut: a host that caps its heaps below \`HEAP_VIEWS\` (NVIDIA) faults on an index past them, as before the clamp.` Ou seja, um heap host menor que `HEAP_VIEWS = 2^21` pode faultar em indice alto vindo de dados. Ver G-01.
- **Confianca.** CONFIRMADA.

---

### D-03 · Descritores nulos do tipo/dimensao errados · remocao de device segundos depois de entrar no mundo

- **Descricao / sintomas.** Device removido alguns segundos depois do mundo comecar; um shader lia um descritor de tipo diferente do esperado e gerava indice de heap absurdo, lendo muito alem do heap de views. Depois das correcoes, o mundo passou a rodar ate o limite de 240 s sem remocao (fato de campo da mensagem de `8c26061`).
- **Causa raiz.** Slots nunca escritos (ou escritos com recurso nulo) recebiam uma view nula 2D fixa, inclusive onde o shader esperava buffer ou constant buffer. Um descritor de textura/nulo lido como buffer nao e inofensivo no host: gera endereco base invalido.
- **Introduzido por.** `2849477`.
- **Tentativas / correcoes (cadeia, todas por diff).**
  1. `8c26061` (2026-11-24) - descritores nulos passam a ter o tipo que o shader espera (raw buffer, constant buffer, ou textura da dimensao do titulo), alem de manter vivos texturas/pipelines/root signatures/heaps nomeados pela lista.
  2. `071487f` (PR #57, merge `eda38aa`) - `null_for()` passa a checar `desc[1] == 1` (dimensao BUFFER) tambem quando o recurso e 0, emitindo SRV/UAV de buffer nulo em vez de view 2D.
  3. `6af0fae` (PR #205, commit `cbb50b8`) - UAV de textura nula recebe a dimensao de view do titulo (1D/array/3D/cube), nao sempre 2D.
  4. `4051149` (2026-11-24) - view de buffer vazia ou que passa da copia host vira view nula; o console aceita, o host remove o device.
  5. `d71c388` (2026-11-24) - guarda contra descritor de **textura** lido como buffer em indice calculado: consulta a tabela de strides e troca por `NULL_BUFFER_VIEW`.
- **Arquivos.** `crates/victory-gpu/src/descriptor/host.rs`, `draw.rs`, `crates/victory-shader/src/emit/lower.rs`, `emit/shader.rs`.
- **Correcao definitiva.** A classe foi fechada em cinco passos (nao ha um commit unico): `8c26061` + `071487f` + `6af0fae` + `4051149` + `d71c388`, todos no HEAD.
- **Evidencia.** `git show 8c26061` (mensagem + `descriptor/host.rs`), `git show 071487f`, `git show 4051149` (`+ fn in_buffer(...)`, `count > 0 && (first + u64::count) * element <= width`), `git show d71c388` (`+ fn buffer_guard(...)` e `NULL_BUFFER_VIEW = HEAP_VIEWS - 2` em `emit/shader.rs`).
- **Estado atual.** Corrigido em camadas; cada camada com teste. `host.rs` mantem dois "shortcuts" conhecidos (views de textura colocadas pelo titulo sao nulas; slots sem nenhuma view recebem null 2D) - ver G-02.
- **Confianca.** CONFIRMADA.

---

### D-04 · Imagem preparada da DLL: ponteiro de callbacks TLS zerado mas realocacao mantida; ida e volta no diretorio de import

- **Descricao / sintomas.** Sob Wine, `LoadLibraryExW` faultava em `ntdll` antes de qualquer codigo do titulo rodar. No Windows nada acontecia porque a copia cai em `0x140000000` e o delta de rebase e 0. Depois, no Windows 10, o loader do OS podia ler o diretorio de import mesmo com a IAT anunciada, e o TLS estatico deixou de ser atribuido quando o diretorio de import foi zerado por inteiro.
- **Causa raiz.** `prepare()` zera `TLS.AddressOfCallBacks` mas deixa a entrada `IMAGE_REL_BASED_DIR64` que aponta para esse campo; ao mapear fora da base preferida, a realocacao transforma o 0 no delta de rebase e o loader le callbacks TLS daquele endereco. Em paralelo, a politica para `IMPORT`/`IAT`/`BOUND_IMPORT` estava errada nas duas direcoes.
- **Introduzido por.** `2849477` (prepare com limpeza de diretorios e TLS); a parte de realocacao existe desde o mesmo commit.
- **Tentativas de correcao (cronologia real, por pai):**
  1. `c21208c` (PR #4, merge `e8f3b1c`, 2026-11-23) - reescreve a entrada DIR64 do campo zerado como `IMAGE_REL_BASED_ABSOLUTE` e adiciona teste que verifica que nenhuma realocacao mira o campo.
  2. `d85097a` (2026-11-23) - aponta o diretorio de import para o descritor vazio final, "porque o Windows 10 le o diretorio quando uma IAT ainda esta anunciada".
  3. `4da3d63` (2026-11-23) - **desfaz** a abordagem: zera `IMPORT`, `BOUND_IMPORT` e `IAT` juntos.
  4. `25a3e95` (2026-11-23, merge `0e5953d` -> `27e6442`, PR #28) - **volta** ao descritor vazio, agora com outra justificativa (o Windows atribui TLS estatico a DLL preparada quando a tabela de import vazia esta presente) e com a contagem corrigida (`info.imports.len()`, nao `Size`), alem de um teste que mapeia uma DLL de verdade e verifica o indice de TLS atribuido.
- **Correcao definitiva.** `25a3e95` para import/TLS; `c21208c` para a realocacao. Ambos presentes no codigo atual (`prepare.rs:69-80`, `prepare.rs:104`).
- **Arquivos.** `crates/victory-loader/src/prepare.rs`, `crates/victory-pe/src/layout.rs` (`dir::IAT`).
- **Evidencia.** `git show c21208c` (`write_u16(&mut bytes, entry, 0)` com comentario `IMAGE_REL_BASED_ABSOLUTE`), `git show d85097a`, `git show 4da3d63` (o diff remove `import_terminator`), `git show 25a3e95` (reintroduz `IMPORT_DESCRIPTOR_SIZE` e o teste `windows_assigns_static_tls_to_a_prepared_dll`).
- **Estado atual.** Estavel; nenhum commit posterior toca `prepare.rs` (`git log -- crates/victory-loader/src/prepare.rs` lista 25a3e95, 4da3d63, d85097a, c21208c, f4504f6).
- **Confianca.** CONFIRMADA (a ida e volta e os diffs). A razao *final* (TLS estatico exige tabela de import presente) e PROVAVEL: vem da mensagem, sustentada por um teste que mapeia a DLL, mas nao foi reproduzida aqui.

---

### D-05 · Caminho narrow reescrito duas vezes · toda abertura narrow de arquivo do titulo falhava

- **Descricao / sintomas.** Com o jogo em um drive host cujo letter tambem e montado como drive virtual (`G:` para `GTA6`), `x64\a.rpf` virava `GTA6\x64\a.rpf` e depois `GTA6\GTA6\x64\a.rpf`. Toda abertura narrow de arquivo do titulo falhava. Mesmo efeito com `T:` quando a pasta de dados esta em um `T:` host.
- **Causa raiz.** `CreateFileA`, `GetFileAttributesA`, `fopen`, `fopen_s` e `std::_Fiopen` narrow reescreviam o caminho virtual para o caminho host e passavam o resultado para os **nossos** wrappers wide, que reescreviam de novo ao ver um letter montado.
- **Introduzido por.** `754619f` (2026-11-23), que cria `narrow()` e os wrappers.
- **Tentativas de correcao.** Nenhuma anterior; `27b530e` e a primeira.
- **Correcao definitiva.** `27b530e` (2026-11-24), merge `b0328ba` (PR #111). As cinco funcoes passam a chamar a funcao wide do **host** diretamente (`host!("kernel32.dll", "CreateFileW"...)` etc.), como `DeleteFileA`, `_open` e `_stat64i32` ja faziam. Teste `narrow_paths_are_rewritten_once` monta um drive dentro de outro e prova que nada e criado "atras" de um caminho reescrito duas vezes.
- **Arquivos.** `crates/victory-hle/src/modules/files.rs`.
- **Evidencia.** `git show 27b530e` (cinco sitios trocados de chamada interna para `host!(...)`); `git log --format="%h %s" -- crates/victory-hle/src/modules/files.rs` lista `754619f`, `2849477`, `27b530e`, `c75fdba` (este ultimo mexe em 4 linhas, nao no caminho narrow).
- **Estado atual.** Corrigido; `CreateFileA` atual chama `host!("kernel32.dll","CreateFileW"...)` (`files.rs:288`). A causa de fundo (montar um letter que e drive real, `mount_at`, `files.rs:69-83`) continua **por desenho**: o titulo exige `G:` e `T:`.
- **Confianca.** CONFIRMADA.

---

### D-06 · Descritor lido de `v_readfirstlane_b32` tratado como "o de cada lane" em qualquer lugar · amostragem errada

- **Descricao / sintomas.** Um shader que le o descritor da primeira lane fora de um loop waterfall, ou sem o `v_cmp_eq` que da a vez a cada lane, amostrava por lanes que o programa pulou. Reportado no PR com a back edge do teste substituida por `s_nop`.
- **Causa raiz.** `ffa6635` fez o tracer seguir `v_readfirstlane_b32` de dwords de descritor ate a fonte por lane (`Source::Fetched`/`Source::Lanes`) e baixou os `v_readfirstlane` como "o dword da lane" e os compares como verdadeiros, **sempre**. Isso so e valido dentro de um waterfall provado. Pior: o colapso do loop nao exigia prova de que os compares realmente estreitam EXEC.
- **Introduzido por.** `ffa6635` (2026-11-24, ramo do PR #566).
- **Tentativas de correcao (na ordem do ramo):**
  1. `077ec8b` - conserta o build: `translate.rs` fazia `match` manual em `Source` e os novos `Select`/`Fetched`/`Lanes` o deixaram nao exaustivo (ver D-19).
  2. `6285e19` - um compare so conta se ele e o `v_readfirstlane` que ele le estao no mesmo loop (cada bloco alcanca o outro); todo descritor lido da primeira lane precisa de todos os dwords comparados assim, senao a traducao para com "a descriptor read from the first lane outside a waterfall loop". Dois testes cobrem a back edge e o compare ausentes.
  3. `b1c8306` - prova de satisfabilidade da selecao de EXEC antes de colapsar: o acesso deve servir exatamente as lanes que casam, e a back edge deve manter exatamente as outras. Quatro formas invalidas ganham teste (compare que nao mascara o acesso, loop que descarta lanes que nao casam, descritor usado depois de EXEC restaurado, wave64 com `exec_hi` sobrescrito).
  4. `6c63dea` - cada falha da prova passa a ser reportada com um motivo distinto; testes com as formas achadas em review.
- **Correcao definitiva.** `b1c8306` + `6c63dea` (dentro do PR #566, merge `ca48fbe`).
- **Arquivos.** `crates/victory-shader/src/resource.rs`, `emit/lower.rs`, `emit/waterfall_tests.rs`.
- **Evidencia.** `git show b1c8306` (`+ fn waterfall(&self, b: usize, accesses: &[usize], reads: &[Value]) -> Option<Waterfall>`, testes `a_compare_that_does_not_mask_the_access_is_no_waterfall`, `a_loop_that_drops_unmatched_lanes_is_no_waterfall`); `git show 6285e19`; ordem por pais: `a3d3bc9 -> ffa6635 -> 077ec8b -> 6285e19 -> 6638098 -> b1c8306 -> 6c63dea -> 74a9d33`.
- **Estado atual.** Corrigido e generalizado: `ffe310e` (depois do merge) usa a **mesma** prova para o waterfall do compilador do console (offset de tabela lido da primeira lane), e nao reintroduz o caso amplo - o gate continua sendo "provado ou nao colapsa". Restricao remanescente: ver G-03.
- **Confianca.** CONFIRMADA.

---

### D-07 · Copy-back sobrescrevia o que a CPU do titulo tinha escrito durante a submissao · crash do titulo

- **Descricao / sintomas.** O titulo crashava quando escrevia, pela CPU, em memoria que shaders tambem escrevem: o retorno pos-submissao devolvia o range inteiro declarado, nao so o que a GPU mudou.
- **Causa raiz.** `shadow` devolvia "o range inteiro depois de toda submissao que o declara" (o proprio codigo dizia `shortcut:`), sem distinguir o que a GPU escreveu.
- **Introduzido por.** `2849477` (mecanismo de shadow e de escrita pela GPU).
- **Tentativas de correcao.** Nenhuma anterior.
- **Correcao definitiva.** `35b054e` (2026-11-24): instantaneo dos bytes do range antes da submissao, devolucao apenas dos bytes que diferem, um instantaneo por copia e submissao, crescido para cobrir todos os ranges declarados, em memoria reutilizada (pool; instantaneos novos custavam ~2,3 milhoes de page faults por frame do mundo).
- **Arquivos.** `crates/victory-gpu/src/shadow.rs`.
- **Evidencia.** `git show 35b054e -- crates/victory-gpu/src/shadow.rs`: `+ before: Mutex<Option<(usize, Vec<u8>)>>`, `+ fn pooled(n: usize)`, e a troca do comentario `shortcut: the whole range goes back after every submission` por `shortcut: what shaders wrote is found by keeping the range's bytes before the submission`.
- **Estado atual.** Corrigido; o comentario atual (`shadow.rs:224`) ainda admite que uma mascara de dirty no lado da GPU seria melhor se os ranges crescerem.
- **Confianca.** CONFIRMADA.

---

### D-08 · Copia de textura escrita pela CPU nao era enviada ao host antes de `CopyResource` / `CopyTextureRegion`

- **Descricao / sintomas.** Uma copia cuja origem era uma textura que o titulo escreveu pela CPU (staging do titulo) podia levar conteudo velho ou vazio para a GPU.
- **Causa raiz.** `copy_resource` e `copy_texture_region` chamavam `ensure_state` e emitiam a copia direto na textura host, sem passar pelo caminho de upload que `texture::upload` faz para texturas marcadas como escritas pelo titulo.
- **Introduzido por.** `2849477` (quando texturas do titulo passaram a ter copia host e a flag `title_written`).
- **Tentativas de correcao.** Dois commits, de autores diferentes, no **mesmo** PR #671: `0064c08` (CopyResource) e `4f53d6f` (CopyTextureRegion).
- **Correcao definitiva.** `6db02ea` (merge do PR #671, 2026-11-24).
- **Arquivos.** `crates/victory-gpu/src/list.rs`.
- **Evidencia.** `git show 0064c08` (`+ if crate::texture::title_written(from) { crate::texture::upload(list, src, from); }`), `git show 4f53d6f` (mesmo padrao em `copy_texture_region`), e `git show 35b054e:crates/victory-gpu/src/list.rs` mostrando que `copy_resource` nao tinha o upload antes.
- **Estado atual.** Corrigido; os dois sitios estao no HEAD (`list.rs:449`, `list.rs:543`).
- **Confianca.** CONFIRMADA (codigo). O sintoma visivel exato e PROVAVEL: nao ha registro de captura antes/depois.

---

### D-09 · Pacote XMA danificado com frame de comprimento 0 · laco infinito (`next_frame`)

- **Descricao / sintomas.** `next_frame` anda os frames anteriores a `start` para saber se o frame em `start` e o ultimo do pacote. Um frame com comprimento 0 nunca avancava `at`; em debug, `number` estourava depois de ~4 bilhoes de voltas e entrava em panic. Encontrado por harness de entrada aleatoria.
- **Causa raiz.** Faltava em `next_frame` a regra que `read_frame` ja aplicava: comprimento que nao cobre o proprio campo de 15 bits nao e frame.
- **Introduzido por.** `2849477` (criacao de `crates/victory-formats/src/xma.rs`).
- **Correcao definitiva.** `005ad49` (PR #14, merge `1dd7afd`): `if bits <= LENGTH_BITS as usize { return None; }`, com teste que trava sem a correcao.
- **Arquivos.** `crates/victory-formats/src/xma.rs`.
- **Evidencia.** `git show 005ad49` (o hunk e exatamente esse) e `git log --format="%h %s" -- crates/victory-formats/src/xma.rs` (primeiro commit `2849477`).
- **Estado atual.** Corrigido. Um segundo overflow do mesmo arquivo foi fechado em `e158d95` (`packet()` com `checked_mul`).
- **Confianca.** CONFIRMADA.

---

### D-10 · `ADVANCE_XMA_READ` somava antes de reduzir · overflow de u32 (panic em debug) e bloco errado

- **Descricao / sintomas.** `read = (campo + blocks) % size` com `blocks` perto de `u32::MAX`: overflow em build com checagem; em release, resultado errado do bloco de leitura do ring.
- **Causa raiz.** Soma em `u32` antes do modulo, com o numero de blocos vindo do titulo (nao confiavel).
- **Introduzido por.** `2849477`.
- **Correcao definitiva.** `257d8bb` (PR #88, merge `2005e35`): `(campo + blocks % size) % size`, com teste `submit(ADVANCE_XMA_READ, &[1, u32::MAX])`.
- **Arquivos.** `crates/victory-hle/src/modules/audio.rs`.
- **Evidencia.** `git show 257d8bb` (hunk de uma linha + teste).
- **Estado atual.** Corrigido.
- **Confianca.** CONFIRMADA.

---

### D-11 · Aritmetica de layout de textura/XMA estourava com valores do titulo (4 sitios + offsets)

- **Descricao / sintomas.** `Layout::new` so checava "diferente de zero" para width/height/depth; valores ate `u32::MAX` estouravam `offset * depth`, `slice_size * slices`, os tamanhos por mip, e um `next_multiple_of` em u32. Debug entrava em panic; release voltava tamanho errado silenciosamente. Depois, mesmo com extent limitado, `detile`/`tile`/copias de volume estouravam com pitch grande.
- **Causa raiz.** Ausencia de limite superior e de aritmetica checada em caminhos que recebem descritores do titulo.
- **Introduzido por.** `2849477`.
- **Tentativas / correcoes.**
  1. `e6a2c54` (PR #15, merge `9e04419`) - `MAX_EXTENT = 1 << 14` (16384 texels por lado, o maximo que um descritor de imagem GFX10 carrega) para width/height/depth.
  2. `e158d95` (PR #50, merge `062ad14`) - `checked_mul`/`checked_add` no calculo de `need` em copies, e `packet()` do XMA com `checked_mul` para o indice do pacote.
- **Correcao definitiva.** `e158d95` (fecha os sitios restantes; a mensagem de `e6a2c54` fala em "quatro sitios de overflow, nenhum restante", e `e158d95` cobre os offsets).
- **Arquivos.** `crates/victory-formats/src/texture.rs`, `crates/victory-formats/src/xma.rs`.
- **Evidencia.** `git show e6a2c54` (`+ const MAX_EXTENT: u32 = 1 << 14;`, `(1..=MAX_EXTENT).contains(&d.width)`), `git show e158d95` (`checked_mul(depth_pitch).zip(...checked_mul(row_pitch))...`).
- **Estado atual.** Corrigido e testado (`extents_past_the_hardware_limit_are_invalid`, `texture_copies_reject_overflowing_pitches`, `volume_copy_rejects_overflowing_depth_pitch`).
- **Confianca.** CONFIRMADA.

---

### D-12 · `XMemMapPhysicalPages` com contagem 0 · panic de `BTreeMap` dentro de `extern "system"` · abort do processo

- **Descricao / sintomas.** Mapear/desmapear zero paginas entrava em `unmap` com range vazio; a busca por views que comecam dentro pedia ao mapa um range invertido, no qual `BTreeMap` entra em panic; em funcao `extern "system"` isso aborta o processo. Sem o panic, a view que contem o endereco seria desmapeada e remapeada como duas.
- **Causa raiz.** Falta de caso base `len == 0`.
- **Introduzido por.** `2849477` (views por endereco em `xmem`).
- **Correcao definitiva.** `02fcf72` (PR #167, merge `13ac0e6`): `if len == 0 { return; }`, com teste que cobre contagem 0 no inicio, no meio e depois de uma view, com e sem paginas, e pela copia do mapeamento de paginas da GPU.
- **Arquivos.** `crates/victory-hle/src/modules/xmem.rs`.
- **Evidencia.** `git show 02fcf72` (o early return + `mapping_no_pages_changes_nothing`).
- **Estado atual.** Corrigido.
- **Confianca.** CONFIRMADA.

---

### D-13 · `XGameSave` com configuration id vazio · conversao de containers apagava os saves de todas as configuracoes (perda de dados)

- **Descricao / sintomas.** Um id vazio dava a **pasta do usuario** (onde ficam as pastas das outras configuracoes). Ao abrir o provider, containers do layout antigo (pastas sem `~` ou ponto no nome) sao convertidos: cada pasta de configuracao foi tomada por um container, um container vazio era gravado no lugar e a pasta era removida **com todos os saves dentro**.
- **Causa raiz.** `config.unwrap_or("default")` nao tratava string vazia; `file_name("")` devolve o proprio diretorio pai.
- **Introduzido por.** `2849477`.
- **Correcao definitiva.** `c88b7ac` (PR #209, merge `27f9b76`): `config.filter(|c| !c.is_empty()).unwrap_or("default")`, com teste que abre tres configuracoes e prova que os saves da primeira sobrevivem.
- **Arquivos.** `crates/victory-hle/src/modules/xgamesave.rs`.
- **Evidencia.** `git show c88b7ac` (hunk de `Provider::open` + teste `an_empty_configuration_id_is_the_default_one`).
- **Estado atual.** Corrigido. E o unico defeito do historico com potencial de **destruir dados do usuario**.
- **Confianca.** CONFIRMADA.

---

### D-14 · Codigos de erro do HLE divergentes do Xbox (`XGameSave`, `storage`)

- **Descricao / sintomas.** (a) Uma atualizacao que cabia na cota de 256 MB mas nao no disco host falhava com `E_GS_QUOTA_EXCEEDED`, mandando o jogador apagar saves; (b) leitura em buffer pequeno devolvia `E_NOT_SUFFICIENT_BUFFER` em vez de `E_GS_PROVIDED_BUFFER_TOO_SMALL`; (c) `XPersistentLocalStorageGetPath` devolvia `ERROR_INSUFFICIENT_BUFFER` em vez do `E_FAIL` documentado.
- **Causa raiz.** O HLE reutilizava o erro do PC/do proprio runtime em vez do erro documentado do contrato do console; a checagem de espaco tambem creditava os blobs substituidos, embora a versao nova seja escrita antes de a antiga sair.
- **Introduzido por.** `2849477` (xgamesave, storage).
- **Correcoes.** `f0ee35e` (`no_room()` separa cota de disco, `E_GS_OUT_OF_LOCAL_STORAGE`), `28c1b07` (`BUFFER_TOO_SMALL` em `read_result`), `376b3cf` (`E_FAIL` em `path()`); merges `f0894c2` (#429) e `40b7ed1` (#371).
- **Arquivos.** `crates/victory-hle/src/modules/xgamesave.rs`, `modules/storage.rs`.
- **Correcao definitiva.** `f0894c2` (merge do PR #429, com `f0ee35e` e `28c1b07`) e `40b7ed1` (merge do PR #371, com `376b3cf`).
- **Evidencia.** `git show f0ee35e` (`+ fn no_room(written, replaced, used, free) -> Option<HRESULT>` com cota antes de disco), `git show 28c1b07`, `git show 376b3cf` (`Some(path) if size <= path.len() => E_FAIL`).
- **Estado atual.** Corrigido nos tres sitios, com testes de tabela.
- **Confianca.** CONFIRMADA.

---

### D-15 · `XTaskQueueDispatch(queue, WORK, INFINITE)` nao terminava quando a fila era terminada · thread e join travados

- **Descricao / sintomas.** O laco classico de dispatcher (`while (XTaskQueueDispatch(q, WORK, INFINITE));`, que termina ao terminar a fila) ficava esperando para sempre; quem dava join na thread tambem. Um dispatch iniciado apos o terminate idem.
- **Causa raiz.** `XTaskQueueTerminate` recusava novos callbacks e drenava os enfileirados, mas nao acordava quem esperava no `Condvar` do port.
- **Introduzido por.** `754619f` (XTaskQueue).
- **Correcao definitiva.** `36d6a79` (PR #239, merge `9e44f38`): `take()` recebe um predicado `ended`; `XTaskQueueTerminate` faz `notify_all` **depois** de pegar o lock da fila (para ninguem perder flag e wake), e a flag e da fila - dispatch de outra fila no mesmo port continua esperando.
- **Arquivos.** `crates/victory-hle/src/modules/xasync.rs`.
- **Evidencia.** `git show 36d6a79` (`wait_while(tasks, waiting)` com `waiting = |t| t.is_empty() && !ended()`, `port.ready.notify_all()`, teste `terminate_ends_a_waiting_dispatch` com dois testes de "outra fila nao termina esta").
- **Estado atual.** Corrigido. A ordem lock-depois-notify esta documentada no proprio codigo.
- **Confianca.** CONFIRMADA.

---

### D-16 · `XAsync` tratava so `S_OK` como sucesso · payload de `S_FALSE` nunca voltava e a chamada ficava registrada para sempre

- **Descricao / sintomas.** `XAsyncGetResultSize` e `XAsyncGetResult` so aceitavam `S_OK`; um provider que completa com `S_FALSE` e um payload deixava os buffers do chamador intactos e a chamada registrada indefinidamente.
- **Causa raiz.** Comparacao `hr == S_OK` em vez de "HRESULT nao negativo".
- **Introduzido por.** `754619f`.
- **Correcao definitiva.** `3e39eac` (PR #520, merge `fd92558`): `hr >= 0` nos dois sitios, `used` reportado, chamada liberada; teste de regressao que completa com `S_FALSE` e 8 bytes.
- **Arquivos.** `crates/victory-hle/src/modules/xasync.rs`.
- **Evidencia.** `git show 3e39eac` (o diff inteiro, com `fn a_nonnegative_completion_still_returns_its_payload`).
- **Estado atual.** Corrigido.
- **Confianca.** CONFIRMADA.

---

### D-17 · `#[cfg(test)] mod tests` duplicado em `victory-gpu/src/host.rs` · `cargo test --workspace` nao compila (E0428)

- **Descricao / sintomas.** `error[E0428]: the name 'tests' is defined multiple times`, em `host.rs:1974`. Nenhum teste do workspace rodava.
- **Causa raiz.** Dois commits diferentes adicionaram um `mod tests` no mesmo arquivo: `0b607b1` (perto do topo, teste do adaptador WARP) e `73d776b` (no fim, testes de `Batch::texture_due`).
- **Introduzido por.** `73d776b` (2026-11-24) - e o segundo, que fecha a duplicata.
- **Correcao definitiva.** `7102a49` (PR #129, merge `d2a4bae`): o bloco do fim vira `mod batch` (uma linha).
- **Arquivos.** `crates/victory-gpu/src/host.rs`.
- **Evidencia.** `git show 73d776b -- crates/victory-gpu/src/host.rs` termina com `+#[cfg(test)] +mod tests { use super::Batch;`; `git show 0b607b1` insere `mod tests { fn warp_adapter_accepts_only_software`. HEAD: `host.rs:282` (`mod tests`) e `host.rs:2479` (`mod batch`, com o comentario explicando).
- **Estado atual.** Corrigido.
- **Confianca.** CONFIRMADA.

---

### D-18 · `lowers()` mudou de aridade em um PR e o chamador antigo ficou · build de testes do `main` quebrado

- **Descricao / sintomas.** `scalar_bit_tests_lower` chamava `lowers(&words)` com a assinatura nova `lowers(words, rsrc2)`; `cargo test` nao compilava no `main`.
- **Causa raiz.** Dois PRs em paralelo: `a2f5f43` criou o helper `fn lowers(words: &[u32])`; `25b0f88` mudou para `fn lowers(words: &[u32], rsrc2: u32)` sem atualizar o chamador.
- **Introduzido por.** `25b0f88` (2026-11-24, cherry-pick, entrou pelo merge `9a92ccf`).
- **Correcao definitiva.** `8e79c80` (2026-11-24) - `lowers(&words, 0)`, junto com a criacao do CI. A mensagem do commit e explicita: "main did not build its tests".
- **Arquivos.** `crates/victory-shader/src/emit/lower.rs`, `.github/workflows/ci.yml`.
- **Evidencia.** `git show 25b0f88 -- crates/victory-shader/src/emit/lower.rs` mostra `- fn lowers(words: &[u32])` -> `+ fn lowers(words: &[u32], rsrc2: u32)`; `git show 8e79c80` mostra `- lowers(&words).unwrap()` -> `+ lowers(&words, 0).unwrap()`.
- **Estado atual.** Corrigido; o CI adicionado no mesmo commit impede a recorrencia.
- **Confianca.** CONFIRMADA.

---

### D-19 · `match` nao exaustivo em `apps/victory/src/translate.rs` apos novos `Source` · `cargo check -p victory` falha

- **Descricao / sintomas.** O tally de recursos casava cada variante de `Source` a mao; `Select`, `Fetched` e `Lanes` deixaram o `match` nao exaustivo.
- **Causa raiz.** Variantes novas em `resource::Source` sem atualizar o consumidor no app.
- **Introduzido por.** `ffa6635`/`6638098` (ramo do PR #566).
- **Correcao definitiva.** `077ec8b` (dentro do PR #566): descritor buscado conta como carregado; selecoes e escolhas por lane nunca sao folhas.
- **Arquivos.** `apps/victory/src/translate.rs`.
- **Evidencia.** `git show 077ec8b` (6 insercoes, 2 remocoes) e a propria mensagem descrevendo o erro de compilacao.
- **Estado atual.** Corrigido.
- **Confianca.** CONFIRMADA.

---

### D-20 · `v_cmp_lg` baixado como `fcmp one` · verdadeiro para NaN no driver AMD

- **Descricao / sintomas.** `v_cmp_lg_{f32,f16}` ("less or greater") e ordenado e diferente: falso se qualquer operando e NaN, na RDNA2 e na ISA. `fcmp one` diz o mesmo, mas o driver D3D12 da AMD o executa como unordered (verdadeiro com NaN); NVIDIA e WARP acertavam.
- **Causa raiz.** Dependencia de semantica de predicado que o driver alvo nao honra.
- **Introduzido por.** `2849477` (tabela de predicados do lowering).
- **Correcao definitiva.** `51f1627` (PR #706, merge `58fed37`): `olt | ogt`; `pred::ONE` fica marcado como nao emitido.
- **Arquivos.** `crates/victory-shader/src/emit/lower.rs`, `emit/bitcode.rs`, `emit/valu_tests.rs`.
- **Evidencia.** `git show 51f1627` (o `if op == "lg"` com `OLT`/`OGT` e o teste `v_cmp_lg_is_false_for_nan` com 10 casos f32 e 10 f16, incluindo sNaN e `+0` vs `-0`).
- **Estado atual.** Corrigido, com teste que roda em hardware e WARP.
- **Confianca.** CONFIRMADA.

---

### D-21 · `min`/`max` nao ordenavam `-0` abaixo de `+0` · sinal errado propagado (ex.: `rsqrt`)

- **Descricao / sintomas.** `v_max_f32`, `v_min_f32`, formas de tres operandos e f16, e `v_pk_max/min_f16` iam para `FMax`/`FMin` do DXIL, que sao "compare e selecione": com `-0` e `+0`, qualquer um dos zeros pode sair, dependendo de ordem e host. Um `-0` que devia ser `+0` transforma, por exemplo, o `+inf` de um `rsqrt` seguinte em `-inf`.
- **Causa raiz.** Semantica do DXIL mais fraca que a da RDNA2 (que ordena `-0 < +0`).
- **Introduzido por.** `2849477`.
- **Correcao definitiva.** `bf84134` (PR #448, merge `dd54de2`): helper `max_min` que mantem `FMax`/`FMin` para tudo, menos quando os dois operandos sao zeros - aí resolve pelos bits de sinal.
- **Arquivos.** `crates/victory-shader/src/emit/lower.rs`, `emit/valu_tests.rs`.
- **Evidencia.** `git show bf84134` (`+ fn max_min(&mut self, max: bool, a: V, b: V) -> V` com `zeros = cmp(EQ, either & 0x7FFF_FFFF, 0)`; teste `max_and_min_order_negative_zero_below_positive_zero`).
- **Estado atual.** Corrigido.
- **Confianca.** CONFIRMADA.

---

### D-22 · `f32_to_half` somava o carry de arredondamento no expoente com `or` · valor caia para a potencia anterior

- **Descricao / sintomas.** `2 - 2^-12` virava `1.0` em vez de `2.0` quando o expoente era impar (o carry do arredondamento era OR-ado e se perdia). Usado no quarto vertice de listas de retangulo com elementos half-float.
- **Causa raiz.** `sign | (exp << 10) | ((man + 0x1000) >> 13)` - carry do mantissa deve ser **somado** ao campo de expoente.
- **Introduzido por.** `2849477`.
- **Correcao definitiva.** `63df363` (PR #260, merge `89e1031`): `sign | (half + ((man + 0x1000) >> 13)) as u16`, com teste `halves_round_up_into_the_next_power_of_two` e um caso end-to-end em `expand_rects`.
- **Arquivos.** `crates/victory-gpu/src/primitive.rs`.
- **Evidencia.** `git show 63df363` (o hunk e o teste).
- **Estado atual.** Corrigido. O mesmo arquivo recebeu `eb30a0d` (subnormais half na saida de retangulos), sinal de que a familia toda era fragil.
- **Confianca.** CONFIRMADA.

---

### D-23 · `fract()` corrigido em `36eb0b4` quebrou em NVIDIA (dobrado para `minNum`, engolia NaN) · regressao consertada em `2e5001c`

- **Descricao / sintomas.** `v_fract_f32(NaN)` passava a devolver `0x3f7fffff` e `v_fract_f16(NaN)` `0x3bff` em hardware NVIDIA. Em WARP/AMD, correto.
- **Causa raiz.** A correcao usava `select(OGT(r, k), k, r)`, que o compilador DXIL da NVIDIA dobra para `FMNMX` (minNum), que descarta NaN.
- **Introduzido por.** `36eb0b4` (2026-11-23).
- **Correcao definitiva.** `2e5001c` (PR #37, merge `237f2e8`): comparar com `pred::OGE` contra um limiar distinto (1.0 para f32, `1 - 2^-12` para f16) impede a dobra e preserva NaN e `+-inf`, mais testes sinteticos de GPU que rodam em hardware e WARP.
- **Arquivos.** `crates/victory-shader/src/emit/lower.rs`, `emit/valu_tests.rs`.
- **Evidencia.** `git show 2e5001c` (mensagem: "Fixes a real-hardware NVIDIA bug in fract() from 36eb0b4"; diff troca `OGT` por `OGE`).
- **Estado atual.** Corrigido.
- **Confianca.** CONFIRMADA (regressao introduzida e reparada no historico, ambas com diff).

---

### D-24 · `s_addc_u32`/`s_subb_u32` nao devolviam carry · SCC ausente · cadeia de 64 bits nao traduzia

- **Descricao / sintomas.** Um `s_cselect`/`s_cbranch` que dependia do carry de uma soma/subtracao escalar de 64 bits falhava a traducao; o programa inteiro era descartado.
- **Causa raiz.** Os dois opcodes usavam `(r, None)` - sem saida de carry para SCC - mientras `v_add_co_ci_u32`/`v_sub_co_ci_u32` ja tinham a logica certa.
- **Introduzido por.** `2849477`.
- **Correcao definitiva.** `a2f5f43` (PR #81, merge `46610eb`): extrai `add_sub_carry` e o usa nos dois lados (vetorial e escalar), com teste `a_scalar_carry_chain_reaches_the_select` em add e sub.
- **Arquivos.** `crates/victory-shader/src/emit/lower.rs`.
- **Evidencia.** `git show a2f5f43` (o `(r, None)` removido e o helper `fn add_sub_carry`).
- **Estado atual.** Corrigido. Nota: foi esse commit que introduziu o helper `lowers()` que depois quebrou o build (D-18).
- **Confianca.** CONFIRMADA.

---

### D-25 · Testes VALU de GPU travavam `cargo test` (dispatches nunca completavam)

- **Descricao / sintomas.** `cargo test` as vezes nunca terminava: sete testes de `emit::valu_tests` que rodam em hardware e em WARP comecavam juntos, nenhum dispatch completava e o binario ficava com um core ocupado ate ser morto; qualquer outro teste de GPU em execucao travava junto. Medicao de campo (mensagem): 3 de 30 execucoes travavam em `main` `d2a4bae`; com lock de processo, 0 de 60.
- **Causa raiz.** Uso concorrente do mesmo device/fila pelos testes, com espera girando no fence.
- **Introduzido por.** Os testes de GPU sinteticos entraram com `2e5001c`/`2849477` e cresceram nos lotes seguintes.
- **Correcao definitiva.** `1106f43` (PR #433, merge `0ddbce8`): lock de processo tolerante a poisoning em `run_cases`; `74a9d33` estende o mesmo lock aos testes de waterfall.
- **Arquivos.** `crates/victory-shader/src/emit/valu_tests.rs`, `emit/waterfall_tests.rs`.
- **Evidencia.** `git show 1106f43` (5 linhas: `ONE_AT_A_TIME`) e `git show 74a9d33`.
- **Estado atual.** Corrigido (serializacao). Nao ha correcao de causa raiz no device.
- **Confianca.** CONFIRMADA (o diff e a medicao de campo; a causa raiz exata dentro do driver nao foi investigada no historico) - causa raiz PROVAVEL.

---

### D-26 · Offset de `s_branch`/far jump lido como unsigned · saltos para tras nao decodificavam

- **Descricao / sintomas.** Um far jump para tras era decodificado com offset positivo (gigante), gerando destino invalido e falha de traducao.
- **Causa raiz.** Campo de 32 bits lido como `u32` em vez de `i32`.
- **Introduzido por.** `12caefe`/`2849477` (decodificador GFX10).
- **Correcao definitiva.** `c963b71` (PR #80, merge `4926f72`): leitura com sinal, com teste de decodificacao.
- **Arquivos.** `crates/victory-shader/src/gfx10/mod.rs`.
- **Evidencia.** `git show c963b71` (30 insercoes em `gfx10/mod.rs`, teste de ida e volta).
- **Estado atual.** Corrigido.
- **Confianca.** CONFIRMADA.

---

### D-27 · `s_ttracedata_imm` com opcode SOPP errado (0x26 em vez de 0x28) · shader com marcador de trace nao traduzia

- **Descricao / sintomas.** A tabela SOPP vinha da lista da Mesa, que traz `s_ttracedata_imm` em 0x26; a RDNA2 codifica em 0x28 (ISA da AMD, tabelas GFX10 do LLVM). Um shader com o marcador era decodificado como opcode desconhecido e falhava a traducao, embora a IR descarte o marcador.
- **Causa raiz.** Fonte de opcode errada (Mesa vs ISA).
- **Introduzido por.** `12caefe`.
- **Correcao definitiva.** `79fb722` (PR #276, merge `db89843`): opcode corrigido e comentario citando as tres fontes.
- **Arquivos.** `crates/victory-shader/src/gfx10/opcodes.rs`.
- **Evidencia.** `git show 79fb722` (19 insercoes em `opcodes.rs`).
- **Estado atual.** Corrigido. E o exemplo mais claro do padrao "tabela de terceiro nao e especificacao" neste repositorio (ver I-06).
- **Confianca.** CONFIRMADA.

## E. Relacao de causa e efeito

Cadeias explicitas, todas verificadas por diff e por ancestralidade (`git log --merges --ancestry-path`).

**E-01 - Slots dinamicos (D-01) e o tripe do heap limitado (D-02, H-01).**
`2849477` cria slots dinamicos com 2^16 e, quando acabam, deixa o draw usar a tabela do titulo -> `3c81420` (PR #336) derruba o draw e sobe para 2^18 -> num host limitado, 2^18 encurta os slots de titulo para 737.856 (contra 934.464) e o titulo indexa ate 917.140 -> `e8d3b64` (mesmo PR) volta a 2^16 onde o host nao aceita o heap grande. Em paralelo, `3827767` (PR #21) resolve o sintoma anterior do mesmo heap limitado: tabela que passa dos slots de titulo passa a ser refeita em slots dinamicos.

**E-02 - Descritores nulos (D-03) em quatro iteracoes.**
`2849477` (tipo unico de nulo) -> `8c26061` (tipo por uso: buffer/raw/CBV/textura da dimensao do titulo) -> `071487f` (dimensao BUFFER quando `desc[1] == 1`) -> `6af0fae` (dimensao 1D/array/3D/cube para UAV de textura nula) -> `4051149` (view vazia/passando da copia vira nula) -> `d71c388` (descritor de textura lido como buffer em indice calculado cai em `NULL_BUFFER_VIEW`). Cada passo fecha um caminho pelo qual um descritor do tipo errado chegava ao shader.

**E-03 - `prepare()`: uma correcao revela a proxima (D-04).**
`c21208c` conserta a realocacao do ponteiro TLS zerado (Wine passa a carregar DLLs) -> `d85097a` tenta consertar o loader do Windows 10 apontando o import para o descritor vazio -> `4da3d63` reverte para "zera IMPORT+IAT" -> `25a3e95` restaura o descritor vazio porque, sem tabela de import presente, o Windows nao atribui TLS estatico a DLL preparada, e corrige a contagem do terminador. Tres politicas diferentes para o mesmo campo em quatro commits.

**E-04 - Uma feature nova, dois consumidores quebrados (D-19, D-18).**
`ffa6635`/`6638098` adicionam `Source::Select`/`Fetched`/`Lanes` -> `077ec8b` conserta o `match` manual em `translate.rs`. `a2f5f43` cria `lowers(words)` -> `25b0f88` muda para `lowers(words, rsrc2)` -> `8e79c80` conserta o chamador e, no mesmo commit, adiciona CI para que o proximo caso seja pego na hora.

**E-05 - Duplicata de modulo de teste (D-17).**
`0b607b1` acrescenta `mod tests` no topo de `host.rs` -> `73d776b` acrescenta outro `mod tests` no fim (testes de `Batch`) -> E0428, o workspace inteiro deixa de compilar testes -> `7102a49` renomeia o segundo para `mod batch`. Tempo entre introducao e correcao: cerca de 59 commits.

**E-06 - Waterfall por lane: do "quase certo" ao provado (D-06, H-04).**
`ffa6635` colapsa todo `v_readfirstlane` de descritor em "o de cada lane" -> `6285e19` exige que o compare esteja no mesmo loop -> `b1c8306` exige prova de que os compares estreitam EXEC e que a retry mantem exatamente as outras lanes -> `6c63dea` da nome a cada falha -> `ffe310e` (depois do merge) usa a mesma prova para o formato do compilador do console. A cada passo, o conjunto de programas aceitos encolhe e a garantia cresce.

**E-07 - Preco do fix de `fract` (D-23).**
`36eb0b4` corrige `fract` para nunca devolver 1.0 -> em NVIDIA o padrao `select(OGT...)` e dobrado para `minNum` e engole NaN -> `2e5001c` troca por `OGE` contra limiar distinto. Uma correcao de fidelidade introduziu um bug dependente de compilador; o teste sintetico em hardware/WARP que veio junto e o que o pegou.

**E-08 - Copia de textura escrita pela CPU (D-08).**
`2849477` da copia host e a flag `title_written` as texturas do titulo, mas so o caminho de amostragem passa por `texture::upload` -> `35b054e` (copy-back por diff) melhora o retorno, nao a ida -> `0064c08` e `4f53d6f` (dois autores, mesmo PR #671) adicionam o upload antes de `CopyResource` e `CopyTextureRegion`.

**E-09 - Falha de GPU por descritor, tres geracoes.**
Gen 1: nulo do tipo errado (`8c26061`). Gen 2: descritor de textura lido como buffer em indice vindo de dados (`d71c388`, com a tabela de strides). Gen 3: view de buffer vazia ou passando da copia, que o console aceita e o host remove o device (`4051149`). Todas as tres produzem o mesmo sintoma observavel (device removido, endereco baixo absurdo) e so o `VICTORY_D3D12_GBV`/DRED distingue.

**E-10 - Custo tratado como defeito de primeira classe.**
`73d776b` (textura verificada uma vez por submissao em vez de uma vez por lista: 576 -> 48 hashes, ~11x), `4e94c2b` (um view por run de paginas consecutivas: 3.5 -> 7.8 GiB/s com 8 threads), `24480a7` (worklist em vez de passes repetidos sobre todas as instrucoes) e `6d021fe` (custo do frame do mundo) nao mudam comportamento observavel: removem trabalho repetido. Em dois deles a causa raiz e a mesma familia dos defeitos de heap - trabalho por objeto (hash por textura por lista, map/unmap por pagina) onde cabia trabalho por submissao. Com o mundo a 1-2 s por frame (README), esse eixo ainda domina a experiencia.

## F. Correcoes importantes

**F-01 - Derrubar o draw em vez de degradar em silencio (D-01, `3c81420`).**
O que estava errado: um recurso esgotado (slots dinamicos) era tratado como "pode continuar com o que o titulo deixou ali". Por que funcionava mal: a tabela do titulo nao tem constant buffer views nos slots que o host usa para tabelas frescas; ler um descritor de textura como buffer gera endereco base invalido e o driver remove o device. Como foi corrigido: `bind()` devolve `bool`; `tables()` marca `table_failed`; o chamador descarta draw/dispatch/mesh/indirect. Por que funciona: a unidade de trabalho (o draw) e descartavel, o device nao e. Dependencias: precisa de contadores de descarte (`DROPPED`, log em potencia de dois). Reutilizavel: sim - o padrao "recurso escasso -> falha local, nunca fallback silencioso em estrutura binaria" e geral.

**F-02 - Heap dimensionado pela capacidade real do host (D-02, `3827767` + `e8d3b64`).**
O que estava errado: assumir que o heap do titulo cabe no host. Correcao: (a) toda tabela que passa dos slots de titulo e refeita em slots dinamicos, com o offset novo entregue ao shader; (b) o range dinamico e escolhido conforme o host aceite `count + DYNAMIC`, com `MIN_DYNAMIC` como piso. Por que funciona: o shader nunca indexa um slot que o host nao tem - ou o indice foi traduzido para um slot novo, ou o draw cai. Dependencias: `HostHeap::count()`, `mirror()` com aviso. Reutilizavel: sim, para qualquer emulacao de recurso com limite do host menor que o do alvo.

**F-03 - Prova antes de colapsar um loop (D-06, `b1c8306`).**
O que estava errado: reconhecer a **forma** de um waterfall (readfirstlane + compare + retry) e assumir a **semantica** (cada lane le o seu). Correcao: prova de satisfabilidade sobre condicoes de lane - o acesso deve servir exatamente as lanes que casam, a back edge deve manter exatamente as que faltam - com `meet` de condicoes; sem prova, nada e emitido e a razao e reportada. Por que funciona: transforma uma otimizacao em teorema verificavel sobre o CFG, com contraexemplos virando testes. Dependencias: `structure` com condicoes por lane (`Structured::reach`), IR com leitura de EXEC por instrucao. Reutilizavel: altamente - e o modelo para qualquer lowering que dependa de "todas as lanes deste acesso concordam".

**F-04 - Politica de imagem PE preparada (D-04, `c21208c` + `25a3e95`).**
O que estava errado: (a) zerar um campo que ainda tem realocacao; (b) escolher entre "zerar diretorios" e "apontar para o terminador vazio" sem entender o efeito no loader do OS. Correcao: reescrever a entrada DIR64 do campo zerado como ABSOLUTE; manter o diretorio de import apontando para o descritor nulo (com RVA calculado por `imports.len()`, nao pelo `Size` declarado) para o Windows atribuir TLS estatico. Por que funciona: o loader do OS ve uma tabela de import valida e vazia, e nenhuma realocacao ressuscita o ponteiro zerado. Dependencias: parser PE com contagem validada de descritores. Reutilizavel: sim, para qualquer loader que mapeie uma imagem "preparada" com imports proprios.

**F-05 - Copy-back por diff (D-07, `35b054e`).**
O que estava errado: devolver o range inteiro declarado pela GPU sobrescrevia escritas concorrentes da CPU do titulo (crash real). Correcao: instantaneo previo do range, comparacao byte a byte apos a submissao, devolucao so do que difere; buffer de instantaneo reutilizado em pool (sem isso, ~2,3 M page faults por frame). Por que funciona: separa "o que a GPU escreveu" de "o que a CPU escreveu depois", sem mascara de dirty no lado da GPU. Reutilizavel: sim; e o padrao classico para memoria compartilhada CPU/GPU com escrita dos dois lados.

**F-06 - Reescrita de caminho uma vez so (D-05, `27b530e`).**
O que estava errado: chamar o proprio wrapper wide com um caminho que ja era host, reentrando na traducao de drive virtual. Correcao: os wrappers narrow chamam a funcao wide do **host**. Por que funciona: ha exatamente um ponto de traducao por chamada. Dependencias: `host!` (resolucao tardia de export do sistema). Reutilizavel: sim - regra geral: nunca passar um caminho ja traduzido por um tradutor.

**F-07 - Limitacao de entrada antes da aritmetica (D-09, D-11).**
O que estava errado: aritmetica de layout e de pacotes confiando em valores do titulo. Correcao: (a) validar contra o maximo que o hardware pode enderecar (`16384` texels por lado), nao contra um numero inventado; (b) `checked_*` nos caminhos de copia; (c) regra de auto-consistencia no parser de frame (`bits > LENGTH_BITS`). Por que funciona: o limite e derivado do descritor de hardware, entao nenhum dado valido e recusado. Reutilizavel: sim; o harness de entrada aleatoria que achou a maioria destes (`005ad49`, `e6a2c54`, `5fbea47`, `ea24ea1`) e a tecnica reaproveitavel, e `e158d95` mostra a continuacao necessaria: fechar os sitios vizinhos que o primeiro limite nao cobria.

**F-08 - Fidelidade de erro como requisito funcional (D-14, `f0ee35e`, `28c1b07`, `376b3cf`).**
O que estava errado: devolver o erro do PC onde o contrato do console especifica outro. Correcao: separar cota de disco (`E_GS_OUT_OF_LOCAL_STORAGE` vs `E_GS_QUOTA_EXCEEDED`), `E_GS_PROVIDED_BUFFER_TOO_SMALL` para buffer pequeno, `E_FAIL` para path curto. Por que funciona: o titulo ramifica no HRESULT; a mensagem ao jogador e o comportamento seguem. Reutilizavel: sim - em HLE, o codigo de erro e parte da API.

## G. Problemas ainda existentes

**G-01 - Heap host menor que `HEAP_VIEWS` ainda pode faultar (NVIDIA).**
`crates/victory-gpu/src/descriptor/host.rs:137`: `// shortcut: a host that caps its heaps below \`HEAP_VIEWS\` (NVIDIA) faults on an index past them, as before the clamp.` O clamp de indices (`lower.rs:4835`, `HEAP_VIEWS - 1`) protege o caso geral, mas o proprio codigo declara que um heap limitado pode faultar. Confianca: CONFIRMADA (o comentario e a politica atual); impacto medido NAO VERIFICADO.

**G-02 - Views sem descricao e texturas colocadas pelo titulo viram nulas.**
`descriptor/host.rs:998` (`default_view`) e `:619`: uma view sem descricao de textura cujo host tem outro formato e nula; views de texturas que o titulo colocou por conta propria nao sao feitas ("shortcut"). Efeito: conteudo ausente, nao crash. CONFIRMADA como limitacao declarada.

**G-03 - Waterfall por lane: duas restricoes abertas.**
`resource.rs:1820`: "only the accesses are proven. Any other per-lane work in a waterfall loop is taken to run once for each lane" - trabalho por lane que nao e o acesso e executado uma vez, nao por lane. `6c63dea` tambem registra que um loop por operando (sampler dentro do loop do recurso) e recusado por nome. `ffe310e` cita 249 programas que ainda precisam de consultas de tamanho por lane em loops sem acesso. CONFIRMADA.

**G-04 - Nada e liberado no lado host: texturas, copias de janela e views antigas.**
`resource.rs:1588` ("never released (one host texture per distinct address and format)"), `resource.rs:1606` ("the old one is kept"), `shadow.rs:547` ("copies of windows are never freed"). Crescimento monotono de memoria host por execucao longa. CONFIRMADA como shortcut documentado.

**G-05 - Page table limitada a 512 regioes de 256 MiB por tabela.**
`memory.rs:50` `const CAPACITY: usize = 512;` e o comentario "a table names copies in at most CAPACITY 256 MiB regions". O mundo usa "algumas dezenas" (comentario), entao ha folga, mas o limite e rigido e enderecos fora dele leem o que a pagina tiver. CONFIRMADA.

**G-06 - Estado funcional do produto.**
README do HEAD: nao jogavel; 1-2 s por frame no mundo; superficies sem iluminacao; faixa de geometria distante ausente; personagens quebrados em alguns planos; 195 programas com estagios de geometria fundidos + ~100 outros sem traducao; render-target metadata planes (HTILE/CMASK/FMASK/DCC) ausentes (`texture.rs:758`); HDR, depth bounds, queries reais ausentes. CONFIRMADA (documentada pelo proprio projeto no commit `70564ae`).

**G-07 - FMA segue o host.**
`77053a0` sonda o host e avisa quando ele nao funde; `lower.rs:5453` (`shortcut: DXIL has no fused f32 multiply-add; FMad lets the driver fuse`). Resultado de shader que depende de fusao pode diferir entre GPUs. CONFIRMADA como diferenca conhecida, nao como defeito.

**G-08 - Testes de GPU serializados por lock, nao consertados.**
`1106f43` evita a concorrencia que travava `cargo test`; a interacao com o driver nao foi explicada. Se o lock for removido, o hang pode voltar. PROVAVEL.

**G-09 - 124 marcas `shortcut:` no codigo.**
`Select-String "shortcut:" crates/*/src` conta 124 ocorrencias no HEAD. Nao sao bugs por si: sao um passivo declarado (ver I-01). Alguns tem impacto visual direto (DCC/HTILE ausentes), outros sao limites de formato (`bcpack.rs:403`, `texture.rs:514`).

**G-10 - Duas politicas de import/IAT conviveram; a decisao nao esta registrada.**
`prepare.rs` implementa a politica final, mas nao ha ADR/decision file explicando por que zerar IMPORT e errado e apontar para o terminador e certo (a explicacao esta em mensagens de commit e no teste). Risco de reverter por engano. PROVAVEL.

## H. Regressoes

**H-01 - `3c81420` encolheu os slots de titulo em heap limitado (corrigida por `e8d3b64`).**
Subir `DYNAMIC` de 2^16 para 2^18 num host que so aceita 1.000.000 views deixou 737.856 slots de titulo (contra 934.464), e o titulo indexa ate 917.140 - leitura fora do heap. Correcao no mesmo PR: usar o range grande apenas quando o host aceita `count + DYNAMIC`; senao `MIN_DYNAMIC = 2^16`. Evidencia: `git show e8d3b64` (o diff introduz `dynamic: u32` e `MIN_DYNAMIC`). CONFIRMADA.

**H-02 - `36eb0b4` quebrou `fract` em NVIDIA (corrigida por `2e5001c`).**
Ver D-23. O `select(OGT(...))` foi dobrado para `minNum` pelo compilador DXIL da NVIDIA e passou a engolir NaN. Correcao por `OGE` contra limiar distinto, com testes em hardware e WARP. CONFIRMADA.

**H-03 - `d85097a` foi revertida por `4da3d63` e a ideia voltou em `25a3e95`.**
Tres politicas para o mesmo campo em quatro commits (ver D-04/E-03). A reversao intermediaria nao era "erro": era outra hipotese sobre o que o loader do Windows 10 exigia. Estado final: `25a3e95`. CONFIRMADA (churn).

**H-04 - `ffa6635` generalizou demais o tracado por lane (corrigida por `6285e19`, endurecida por `b1c8306`).**
Ver D-06/E-06. A primeira correcao (`6285e19`) exigia o compare no mesmo loop, mas nao provava que ele seleciona; `b1c8306` provou. CONFIRMADA.

**H-05 - `73d776b` quebrou o build de testes (corrigida por `7102a49`).**
O commit que trouxe o teste de `Batch::texture_due` (e uma melhoria de performance de 11x) duplicou `mod tests`. CONFIRMADA.

**H-06 - `25b0f88` quebrou o build de testes do `main` (corrigida por `8e79c80`).**
Mudanca de aridade de helper de teste sem atualizar o chamador de outro PR. CONFIRMADA.

**H-07 - `2849477` e a origem da maior parte dos defeitos de GPU.**
Nao e regressao (foi o commit que criou o suporte), mas e o ponto de introducao de D-01, D-02, D-03, D-08, D-09, D-10, D-11, D-12, D-13, D-14, D-20, D-21, D-22, D-24. Concentrar 42.518 linhas em um commit e o que explica a densidade de defeitos encontrados nos dois dias seguintes. CONFIRMADA por `git log -- <arquivo>` em cada caso.

## I. Conhecimento reutilizavel

**I-01 - Registro de `shortcut:` como passivo explicito.**
- Problema resolvido: separar "nao implementado de proposito" de "bug", num codebase que precisa rodar um titulo real em dias.
- Implementacao: comentario iniciando por `shortcut:` no ponto exato, dizendo o que falta e qual e o efeito; 124 ocorrencias no HEAD.
- Origem: VIctory, pratica presente desde `2849477` (`crates/victory-gpu/src/memory.rs:30`, `shadow.rs:153`, `texture.rs:13`).
- Conflitos/riscos: virou um passivo grande e sem dono; alguns shortcuts tem efeito visual (DCC/HTILE), outros sao limites de formato. Recomendacao: manter, mas gerar um relatorio (`xtask`) que liste por arquivo e ligue a um teste que falha quando o comportamento mudar.

**I-02 - Falha local em vez de fallback silencioso (D-01, F-01).**
- Problema: recurso escasso com fallback "usa o que esta la" produz corrupcao de estado de GPU, nao erro.
- Implementacao: `bind() -> bool`, `table_failed`, descarte da unidade de trabalho + contador.
- Origem: `crates/victory-gpu/src/draw.rs` (`3c81420`).
- Riscos: descarte silencioso vira "geometria faltando" e pode esconder bug; por isso o log em potencia de dois e o contador por frame.
- Recomendacao: adaptar para qualquer emulador de recurso com limite (descritores, handles, mappings).

**I-03 - Prova de equivalencia antes de otimizar lowering (D-06, F-03).**
- Problema: reconhecer a forma de um padrao de compilador e assumir a semantica.
- Implementacao: satisfabilidade sobre condicoes por lane (`meet`/`reach`), testes com as formas invalidas.
- Origem: `crates/victory-shader/src/resource.rs` (`b1c8306`).
- Riscos: custo de compilacao e falsos negativos (programa valido recusado); mitigado por mensagens de recusa especificas (`6c63dea`).
- Recomendacao: alto valor para tradutores de ISA e para qualquer lowering que dependa de convergencia de lanes/warp.

**I-04 - Instantaneo + diff para memoria compartilhada CPU/GPU (D-07, F-05).**
- Problema: devolver a GPU o range inteiro sobrescreve escritas da CPU.
- Implementacao: copia previa do range (pool), comparacao, devolucao so do que difere.
- Origem: `crates/victory-gpu/src/shadow.rs` (`35b054e`).
- Riscos: custo de copia (resolvido por pool; sem ele, 2,3 M page faults/frame) e granularidade (ranges declarados).
- Recomendacao: usar quando nao houver mascara de dirty do lado da GPU; e o caminho mais simples e correto.

**I-05 - Limite de entrada derivado do hardware (D-11, F-07).**
- Problema: valores do titulo em aritmetica de layout.
- Implementacao: `MAX_EXTENT = 1 << 14` (o maximo que um descritor GFX10 endereca) + `checked_*` nos caminhos de copia.
- Origem: `crates/victory-formats/src/texture.rs` (`e6a2c54`, `e158d95`).
- Riscos: se o alvo mudar (GFX11+), o limite muda; documentar a origem do numero.
- Recomendacao: sempre derivar o limite do formato/descritor, nunca de um "numero redondo".

**I-06 - Tabela de terceiro nao e especificacao (D-27, D-20).**
- Problema: opcode e semantica de predicado vieram de fontes secundarias (Mesa) ou de uma leitura "equivalente" (`fcmp one`).
- Implementacao: corrigir pela ISA/LLVM/medicao em hardware, com teste que falha no comportamento errado.
- Origem: `79fb722` (opcode), `51f1627` (predicado), `36eb0b4`/`0b499d7` (medicoes de hardware de outro projeto, issue #2).
- Riscos: divergencia por driver (o caso AMD de `fcmp one`) so aparece em hardware especifico.
- Recomendacao: para cada instrucao critica, um teste que roda em hardware **e** em WARP.

**I-07 - Mapear imagem PE "preparada" com imports proprios (D-04, F-04).**
- Problema: mapear a imagem com imports zerados quebra o loader do OS (TLS estatico, leitura de diretorio) e a realocacao ressuscita campo zerado.
- Implementacao: import aponta para o descritor nulo (RVA por contagem de descritores validada), DIR64 do campo zerado vira ABSOLUTE.
- Origem: `crates/victory-loader/src/prepare.rs` (`c21208c`, `25a3e95`).
- Riscos: depende do comportamento do loader do Windows 10/11 e do Wine; precisa de teste que mapeie uma DLL de verdade (existe: `windows_assigns_static_tls_to_a_prepared_dll`).
- Recomendacao: reutilizavel para qualquer emulador que injete imports.

**I-08 - Harness de entrada aleatoria sobre parsers e sobre o tradutor inteiro.**
- Problema: parsers de formato e lowering recebem dados hostis por definicao.
- Implementacao: gerar entradas aleatorias decodificaveis; rodar `ir -> resource -> io -> structure -> emit`. Achados: XMA zero-length, 4 sitios de overflow de textura, underflow de canal de input, stack overflow de lane mask, e "um milhao de programas sem panic/hang/overflow" como criterio de saida.
- Origem: `005ad49`, `e6a2c54`, `e158d95`, `5fbea47`, `ea24ea1`, `4cca767`.
- Riscos: cobertura depende do gerador; nao substitui teste semantico (D-20/D-21 vieram de medicao, nao de fuzz).
- Recomendacao: adotar como gate de CI noturno, com seed fixa.

**I-09 - Descritor como dado: guarda por tabela de strides (D-03, `d71c388`).**
- Problema: shader pega indice de descritor da memoria; o slot pode conter textura onde se espera buffer, e textura lida como buffer e endereco base absurdo (shift de 8 bits).
- Implementacao: tabela de stride por slot (root SRV `VIEW_STRIDES`); stride 0 -> `NULL_BUFFER_VIEW`; clamp de indice em `HEAP_VIEWS - 1`.
- Origem: `crates/victory-shader/src/emit/lower.rs` (`fn buffer_guard`), `emit/shader.rs` (`NULL_BUFFER_VIEW`).
- Riscos: um load extra por acesso (custo) e a tabela precisa cobrir slots dinamicos.
- Recomendacao: essencial para qualquer runtime bindless sobre D3D12; e a diferenca entre "leitura errada" e "device removido".

## J. Conclusao tecnica

O que este repositorio mostra, em 241 commits e menos de dois dias, e um metodo: reproduzir o sintoma, achar o mecanismo exato no diff, corrigir com o menor numero de linhas que fecha a classe de falha, e deixar um teste que falharia antes. As melhores correcoes do historico (`3c81420`, `b1c8306`, `25a3e95`, `35b054e`, `d71c388`) tem todas a mesma forma: **transformam uma suposicao implicita em invariante verificada**. Onde a suposicao ficou implicita, ela voltou como defeito - `3c81420` supos que todo host aceita "titulo + 2^18" e `e8d3b64` teve de reparar; `36eb0b4` supos que `select(OGT(...))` sobrevive ao compilador da NVIDIA e `2e5001c` teve de reparar; `73d776b` e `25b0f88` supoem que ninguem mais esta mexendo no mesmo arquivo e quebraram o build de testes.

O que mais chama atencao tecnicamente:

1. **A camada de GPU e onde o projeto vive ou morre.** Dos 27 defeitos catalogados, 7 sao de GPU (D-01, D-02, D-03, D-07, D-08, D-22, D-25) e os de maior impacto (remocao de device) sao todos da mesma familia: um descritor ou slot de heap e lido com a semantica errada. A resposta do projeto foi construir defesas em profundidade (clamp de indice, tabela de strides, nulos do tipo certo, descarte de draw) - e o preco e que agora existem varias camadas de "silenciosamente nao desenha".
2. **O tradutor de shaders e o ativo mais bem cuidado.** Cada correcao de fidelidade (D-20, D-21, D-23, D-24, cvt saturate, med3/NaN, sin/cos em voltas, bit fields assinados, formatos packed) vem com teste sintetico que roda em hardware **e** em WARP. A regressao D-23 e a prova de que esse investimento se paga: foi pega por teste, nao por relato.
3. **A disciplina de erro HLE e rara.** D-13/D-14 mostram um projeto que trata HRESULT como contrato (inclusive o que o Xbox documenta e o PC nao faz) e que aceitou reescrever a politica de import/TLS quatro vezes ate o loader do OS se comportar.
4. **O passivo e explicito.** 124 `shortcut:` documentados. Isso e honesto e util, mas tambem e o mapa do que falta: DCC/HTILE, liberacao de recursos host, waterfall por lane fora do acesso, heap limitado em NVIDIA, e a performance do mundo (1-2 s por frame).

Veredito: como arqueologia, o historico e quase ideal - cada defeito relevante tem causa, tentativa e correcao identificaveis por diff, e as poucas regressoes foram pegas e reparadas dentro da mesma janela. Como produto, o estado e o que o README declara: o mundo aparece, lento e incompleto. O que separa os dois e exatamente o que as secoes G e H listam.

## K. Indice de commits

Ordem cronologica (mais antigo primeiro), gerado de `git log --reverse --format="%h %ad %an %s" --date=short`. 241 linhas.

```
f4504f6 2026-11-23 Marvis Initial commit
12caefe 2026-11-23 Marvis GFX10 operand fields and SP3 text; FSSM wave size
754619f 2026-11-23 Marvis Run Minecraft: runtime HLE, D3D12.X device on the host GPU, shader IR
d032b8a 2026-11-23 Marvis Title DLLs at run time: find by name, map on first LoadLibrary
4b25269 2026-11-23 Marvis Thread notices for title images
bc30709 2026-11-23 Marvis Thread notices: leave out threads Rust's std starts
2849477 2026-11-23 Marvis Run Grand Theft Auto VI from first boot into the world
015d45b 2026-11-23 Marvis README: status, subsystem table, how to run, debugging switches
c21208c 2026-11-23 Exorion Loader: keep the cleared TLS callback pointer out of base relocations
e8f3b1c 2026-11-23 Marvis Merge pull request #4 from Exorion1er/fix/tls-callbacks-reloc
934e67f 2026-11-23 ismailchaka1 loader: emulate AMD-only instructions on Intel CPUs
005ad49 2026-11-24 9hvn9yt2vk-netizen xma: stop at a zero-length frame instead of looping forever
d4ed2de 2026-11-23 raffclar Process exit: start the watchdog before other threads end
d348b9a 2026-11-24 Marvis Merge pull request #10 from ismailchaka1/emulate-amd-only-instructions
e6a2c54 2026-11-24 9hvn9yt2vk-netizen texture: refuse extents past 16384 texels instead of overflowing
36eb0b4 2026-11-23 Marvis Shaders: med3 with NaN, fract below 1.0, signed bit fields past bit 31
0b499d7 2026-11-24 Marvis Shaders: sine and cosine in turns, exact zeros at half and quarter turns
8c26061 2026-11-24 Marvis GPU: the world stays up; indirect execution behind a switch
3827767 2026-11-24 Nostyxx GPU: make tables past a capped host view heap afresh
1dd7afd 2026-11-24 Marvis Merge pull request #14 from 9hvn9yt2vk-netizen/fix-xma-zero-length
e98d3ca 2026-11-23 Manuel Pérez PE: checked file offset in rva_to_file_offset
d85097a 2026-11-23 ifBars Keep an empty import descriptor for the Windows 10 loader
29b558c 2026-11-23 Jeremy Goldberg test(gpu): use the SDK root parameter in the serializer fixture
9e04419 2026-11-24 Marvis Merge pull request #15 from 9hvn9yt2vk-netizen/fix-texture-extents
0c91c03 2026-11-23 raffclar Loader: test the AMD-only instruction emulation on any CPU
d222b87 2026-11-24 Marvis Merge pull request #19 from raffclar/exit-watchdog
97c78d1 2026-11-24 Marvis Merge pull request #21 from Nostyxx/fix/nvidia-view-heap-cap
f00caec 2026-11-23 raffclar Loader: test 64-bit extrq at index 0, not in AMD's undefined range
0b607b1 2026-11-23 LaunchDoesntCode GPU: verify WARP adapter is software before using it
ea24ea1 2026-11-24 9hvn9yt2vk-netizen shader: a lane mask that needs its own value fails instead of overflowing the stack
71cfe84 2026-11-23 Henrique Bueno loader: emulate clwb on Intel CPUs before Ice Lake
15b0091 2026-11-24 9hvn9yt2vk-netizen shader: VICTORY_SHADER_LOOP_LIMIT to find what makes world frames slow
e158d95 2026-11-24 Vanthys Guard texture and XMA offsets against overflow
4cca767 2026-11-24 Vanthys Reject empty XMA test fixture directories
7e42fef 2026-11-23 BaggyALPHA Report Windows commit-limit failures in crashes.log
ef22723 2026-11-24 Manuel Pérez PE: remove leftover dbg_tls test and check layout offsets
2e5001c 2026-11-24 alessiodeia shader: keep fract(NaN) as NaN on NVIDIA and add synthetic GPU VALU tests
4da3d63 2026-11-23 ifBars Clear the IAT directory alongside the import directory
5fbea47 2026-11-24 9hvn9yt2vk-netizen shader: look up an input channel without underflowing
071487f 2026-11-24 Yaekai GPU: emit null buffer SRV/UAV when descriptor names a buffer with null resource
0c56091 2026-11-24 Yaekai Shaders: lower v_xnor_b32 and v_swap_b32
4e94c2b 2026-11-24 TG0000 xmem: write DirectStorage page destinations one view per run
fc807b6 2026-11-23 ifBars Reuse texture row spans when converting Xbox pixel layouts
35b054e 2026-11-24 Marvis GPU: pipelines off the render thread and cached; indirect execution on
f3c7f0f 2026-11-24 Manuel Pérez GPU: leave rectangle vertex components past the stride alone
25a3e95 2026-11-23 ifBars Keep an empty import descriptor so Windows assigns static TLS
78b8404 2026-11-23 meower xmem: commit physical-page sections whole under Wine
0eca412 2026-11-24 FIocker GPU diagnostics: skip pipelines by shader hash
c4e915f 2026-11-23 Henrique Bueno loader: a clwb write-back loop runs in one exception
9a5cbbe 2026-11-24 tedin7 fix: skip unused export address slots instead of reporting them at RVA 0
4c1766c 2026-11-24 Exorion GPU: keep a guarded block's write fault from being declined in a race
73d776b 2026-11-24 TG0000 GPU: check a sampled texture once per submission, not once per list
c963b71 2026-11-24 tedin7 fix: read a far jump's offset as signed so backward far jumps decode
a2f5f43 2026-11-24 tedin7 Shaders: set SCC from the carry of s_addc_u32 and s_subb_u32
b304b7d 2026-11-23 Jeremy Goldberg fix(gpu): average copy statistics over the full reporting interval
257d8bb 2026-11-24 Manuel Pérez Audio: reduce the title's XMA read advance before adding it
ec0d155 2026-11-24 Manuel Pérez Watch: saturate the end of announced writes and remaps
38ae55d 2026-11-24 FIocker Keep the diagnostic skip outside indirect argument validation
0e5953d 2026-11-23 ifBars Merge branch 'main' into fix/windows10-import-directory
3e982f7 2026-11-24 TG0000 BCPack: find a slot's bucket from a window table
7d33c46 2026-11-24 Marvis Merge pull request #45 from HenriqueSBueno/clwb
27b530e 2026-11-24 Duff Files: send rewritten narrow paths to the host, not to our wide wrappers
27e6442 2026-11-24 Marvis Merge pull request #28 from ifBars/fix/windows10-import-directory
24480a7 2026-11-24 TG0000 Shaders: find the written instructions with a worklist, not repeated passes
cbd71a9 2026-11-24 Marvis Merge pull request #38 from raffclar/emulation-tests
cc51927 2026-11-24 Marvis Merge pull request #30 from AndyLatimer/fix/pe-rva-offset-overflow
bf6c557 2026-11-24 Marvis Merge pull request #75 from tedin7/vic-pe-export-holes
062ad14 2026-11-24 Marvis Merge pull request #50 from Vanthys/fix/format-offset-overflow
a6888a6 2026-11-24 Marvis Merge pull request #43 from 9hvn9yt2vk-netizen/shader-mask-cycle
8fe0b74 2026-11-24 Marvis Merge pull request #56 from 9hvn9yt2vk-netizen/find-input-underflow
4a8a058 2026-11-24 Marvis Merge pull request #58 from Yaekai/feat/shader-xnor-swap
46610eb 2026-11-24 Marvis Merge pull request #81 from tedin7/vic-scalar-carry
4926f72 2026-11-24 Marvis Merge pull request #80 from tedin7/vic-far-jump-backward
237f2e8 2026-11-24 Marvis Merge pull request #37 from alessiodeia/fix/valu-med3-fract-trig
3c86053 2026-11-24 tedin7 Shaders: lower s_bitcmp0/1_b32 and s_bitcmp0/1_b64
eda38aa 2026-11-24 Marvis Merge pull request #57 from Yaekai/fix/null-buffer-views
683278b 2026-11-24 Marvis Merge pull request #36 from jeremygold02/codex/root-parameter-fixture
fc0c4dd 2026-11-24 Marvis Merge pull request #77 from Exorion1er/fix/guard-fault-race
0f35dfc 2026-11-24 Marvis Merge pull request #66 from AndyLatimer/fix/gpu-rect-element-past-stride
fa5ef73 2026-11-24 Marvis Merge pull request #79 from TG0000/perf/texture-check-once-per-submission
93bc978 2026-11-24 Marvis Merge pull request #60 from ifBars/perf/reuse-texture-row-spans
97464ba 2026-11-24 Marvis Merge pull request #61 from TG0000/hle/physical-page-runs
6168fc6 2026-11-24 Marvis Merge pull request #70 from FIocker/gpu/diagnostic-pipeline-filter
91e9bc9 2026-11-24 Marvis Merge pull request #46 from 9hvn9yt2vk-netizen/shader-loop-limit
4c0bb89 2026-11-24 Marvis Merge pull request #41 from LaunchDoesntCode/codex/proton-warp-safety
8a8a38d 2026-11-24 Marvis Merge pull request #53 from a55uka/fix/wine-physical-pages
9aef3d9 2026-11-24 Marvis Merge pull request #51 from BaggyALPHA/fix/windows-commit-limit-diagnostic
7102a49 2026-11-23 LUIS LARIOS GPU: rename the second test module in host.rs so the tests compile
1823079 2026-11-24 lordon1a Shaders: lower v_cmp_class_f16 and v_cmpx_class_f16
d2a4bae 2026-11-24 Marvis Merge pull request #129 from Luislariosss/fix/duplicate-test-module
7d73e2b 2026-11-24 Codex pipeline: remove staging files when cache publication fails
02fcf72 2026-11-24 Maraco91 xmem: mapping or unmapping no pages changes nothing
babe2db 2026-11-24 Duff Crash reports: show the bytes of an instruction the CPU refused
3390e47 2026-11-24 Duff Crash reports: say what an access violation's address holds
cbb50b8 2026-11-23 julio GPU: give null texture UAVs the title's view dimension
c88b7ac 2026-11-24 Maraco91 Game saves: an empty configuration id is the default, not every configuration's folder
afc4641 2026-11-24 Andrea Casale net: skip newer-Windows-only exports in only_loopback_is_reachable
18b2716 2026-11-24 Andrea Casale net: rustfmt the presence assertion
d46b078 2026-11-24 Duff XUser: a gamertag without a suffix has an empty modern suffix
5f670bb 2026-11-24 Duff Loader: a trapped unimplemented import goes into the crash log
36d6a79 2026-11-24 Maraco91 XTaskQueue: terminating a queue ends a dispatch waiting on it
79fb722 2026-11-24 Duff Shaders: s_ttracedata_imm is SOPP opcode 0x28, not 0x26
63df363 2026-11-24 Duff GPU: a half-float rounding carry raises the exponent, not halves the value
717f322 2026-11-24 Maraco91 GPU: GetDevice and the DXGI device hand out the device at every level
5f29f3a 2026-11-24 Duff Loader: a title DLL that fails to prepare is named in the error
9fe73ba 2026-11-23 artor410 shader: decode the packed typed-buffer formats
a3d3bc9 2026-11-24 Claude Shaders: trace descriptors chosen by s_cselect and s_cmov
ffa6635 2026-11-24 Claude Shaders: lower waterfall loops over per-lane descriptors in one pass
b5c6678 2026-11-24 Abdellah K. loader: unwind a 16-byte aligned copy of the context
3c81420 2026-11-24 hophaver GPU: drop draws whose fresh tables cannot be made; 2^18 dynamic slots
cc63aa2 2026-11-23 Enzo Souza Duarte Config: ignore an empty VICTORY_TEST_TITLES
40c127c 2026-11-23 Enzo Souza Duarte Chore: clear two workspace Clippy errors
efb3e3d 2026-11-24 FIocker GPU diagnostics: order tied drop counts consistently
376b3cf 2026-11-23 marquezini Storage: a path buffer too small fails with E_FAIL, as on Xbox
a4e7f1d 2026-11-24 FIocker GPU: replay pending descriptors before publishing the host heap
0f369d6 2026-11-23 marquezini Capture: R16G16B16A16_UNORM targets are integers, format 89 is 10:10:10:2
ded334b 2026-11-23 Luiz Neves Shaders: lower v_cvt_norm_i16_f16 and v_cvt_norm_u16_f16
404d7fd 2026-11-24 Duff Shaders: PSV0 says a pixel shader runs per sample when an input does
c2f60d6 2026-11-24 Duff Shaders: a pixel shader writing SV_StencilRef declares StencilRef
2af66ff 2026-11-23 Luiz Neves Shaders: v_cvt_u16_f16 and v_cvt_i16_f16 saturate at 16 bits
ff8694a 2026-11-23 itsRevela Window: keep Alt and F10 from freezing the title in menu mode
eb30a0d 2026-11-24 Enzo Souza Duarte GPU: write half subnormals from f32 rectangle outputs
28c1b07 2026-11-24 marquezini XGameSave: a read result into a small buffer is E_GS_PROVIDED_BUFFER_TOO_SMALL
d5704aa 2026-11-24 Pedro Fortunato fix(shader): lower dynamic v_perm_b32 without INT64
f0ee35e 2026-11-24 marquezini XGameSave: a full host disk is E_GS_OUT_OF_LOCAL_STORAGE, not the quota
f4438b3 2026-11-23 local Shaders: reads of uninitialized SGPRs read as zero, as VGPRs do
1106f43 2026-11-24 r4. Shader tests: run the VALU GPU tests one at a time
077ec8b 2026-11-24 Claude translate: count per-lane fetched descriptors as loaded
bf84134 2026-11-24 Duff Shaders: maximums and minimums order -0 below +0, as RDNA2 does
e8d3b64 2026-11-24 hophaver GPU: the larger dynamic range only where the heap still fits
6285e19 2026-11-24 Claude Shaders: take the first lane's descriptor as each lane's own only in a waterfall loop
6638098 2026-11-24 r4. Shaders: per-lane descriptors chosen by control flow
c249f02 2026-11-23 Saitken91 Shaders: centroid position with conservative depth
9a8908c 2026-11-24 Advait Palve core: show data_dir in config and check its asset policy
25b0f88 2026-11-24 Dathinel Test probe: compare direct and joined reads of a shader timestamp
3dd467c 2026-11-23 KoLolX3 Shaders: a phi that only another read phi consumes is live too
260611a 2026-11-23 Saitken91 Shaders: image_sample_o with constant offsets
a0efd2d 2026-11-24 AlanQiang0418 Shaders: decode GFX10 packed inline constants by operand type
b1c8306 2026-11-24 fuzeofficial Shaders: prove EXEC selection before collapsing a waterfall
6c63dea 2026-11-24 r4. Shaders: name why a loop is no waterfall, and test the shapes found in review
d74558c 2026-11-24 Carlos Sant'Anna GPU diagnostics: describe a buffer UAV by its range, not as uav None
764fa19 2026-11-23 lameloball2813-ui Loader tests: build the probe fixture under CARGO_TARGET_TMPDIR
0064c08 2026-11-24 fuzeofficial GPU: upload CPU-written textures before CopyResource
af22e5d 2026-11-23 EatCodeSleep2 GPU: use the host upload path for texture staging copies
5864aab 2026-11-23 lameloball2813-ui Docs: rustdoc builds the loader, GPU and shader crates without warnings
31ce30f 2026-11-24 fuzeofficial GPU: unbind vertex buffer slots for null view arrays
ab3eb92 2026-11-23 Louis Faust Shaders: gate the constant_color test helper to Windows
4f53d6f 2026-11-24 fuzeofficial GPU: upload CPU-written sources before texture region copies
0f0c872 2026-11-23 Louis Faust Docs: drop the private intra-doc link to MICRO in texture.rs
51f1627 2026-11-24 Lyrnc Shaders: v_cmp_lg is ordered on AMD hosts too (olt | ogt, not fcmp one)
3e39eac 2026-11-23 quaternion fix(hle): return XAsync payloads for nonnegative results
1f858b0 2026-11-24 Marvis Merge pull request #25 from artor410/packed-buffer-formats
c2e7254 2026-11-24 Marvis Merge pull request #87 from jeremygold02/codex/copy-stat-frame-window
0e990e4 2026-11-24 Marvis Merge pull request #104 from TG0000/perf/bitcode-worklist-liveness
b0328ba 2026-11-24 Marvis Merge pull request #111 from Dufftwt/fix/narrow-paths-translated-twice
3d77d5e 2026-11-24 Marvis Merge pull request #117 from tedin7/vic-s-bitcmp-2
80c0d12 2026-11-24 Marvis Merge pull request #140 from lordon1a/feat/cmp-class-f16
ba7d873 2026-11-24 Marvis Merge pull request #152 from Makto94/fix/net-test-win10
787f665 2026-11-24 Marvis Merge pull request #161 from lordon1a/fix/pipeline-staging-cleanup
13ac0e6 2026-11-24 Marvis Merge pull request #167 from Maraco91/xmem/zero-length-unmap
3dd37e6 2026-11-24 Marvis Merge pull request #198 from Dufftwt/fix/crash-report-instruction-bytes
27c960b 2026-11-24 Marvis Merge pull request #204 from Dufftwt/fix/crash-report-fault-memory
6af0fae 2026-11-24 Marvis Merge pull request #205 from asdaasdsaddf/fix/null-uav-dimension
27f9b76 2026-11-24 Marvis Merge pull request #209 from Maraco91/xgamesave/empty-configuration-id
7c8cecb 2026-11-24 Marvis Merge pull request #229 from Dufftwt/fix/gamertag-modern-suffix-empty
9e44f38 2026-11-24 Marvis Merge pull request #239 from Maraco91/xtaskqueue/terminate-ends-dispatch
56d5032 2026-11-24 Marvis Merge pull request #245 from Dufftwt/fix/import-trap-in-crash-log
88b6658 2026-11-24 Marvis Merge pull request #249 from Maraco91/gpu/get-device-newer-levels
89e1031 2026-11-24 Marvis Merge pull request #260 from Dufftwt/fix/f32-to-half-rounding-carry
db89843 2026-11-24 Marvis Merge pull request #276 from Dufftwt/fix/s-ttracedata-imm-opcode
93c0852 2026-11-24 Marvis Merge pull request #285 from Dufftwt/fix/name-title-dll-that-fails-to-prepare
5a16e66 2026-11-24 Marvis Merge pull request #313 from Abdellah-Khanani/loader-aligned-unwind
edf64e2 2026-11-24 Marvis Merge pull request #336 from hophaver/fix/dynamic-descriptor-exhaustion
533b7e2 2026-11-24 Marvis Merge pull request #341 from enzoduarte1/config/empty-test-titles-env
2745589 2026-11-24 Marvis Merge pull request #355 from enzoduarte1/clippy/clean-workspace-lints
42361b0 2026-11-24 Marvis Merge pull request #356 from FIocker/gpu/stable-drop-rank
7374bfb 2026-11-24 Marvis Merge pull request #365 from FIocker/gpu/publish-initialized-descriptor-heap
a7e60cf 2026-11-24 r4. Merge upstream main (42361b0) into shader/per-lane-phi
40b7ed1 2026-11-24 Marvis Merge pull request #371 from marquezinii/fix/pls-path-small-buffer
f67c50c 2026-11-24 Marvis Merge pull request #372 from marquezinii/fix/capture-unorm16-formats
ebf3615 2026-11-24 Marvis Merge pull request #376 from LouisRiverstone/shader-cvt-norm-f16
c30fc99 2026-11-24 Marvis Merge pull request #381 from Dufftwt/fix/psv-sample-frequency
faba64c 2026-11-24 Marvis Merge pull request #382 from Dufftwt/fix/stencil-ref-shader-flag
2e4e280 2026-11-24 Marvis Merge pull request #384 from LouisRiverstone/shader-cvt16-saturate
3c86238 2026-11-24 Marvis Merge pull request #418 from enzoduarte1/gpu/half-subnormal-output
dd273df 2026-11-24 Marvis Merge pull request #419 from itsRevela/fix/swallow-keyboard-menu
9f18eb4 2026-11-24 Marvis Merge pull request #423 from pedrofort/shader-perm-no-int64
f0894c2 2026-11-24 Marvis Merge pull request #429 from marquezinii/fix/xgamesave-documented-errors
ce42e77 2026-11-24 Marvis Merge pull request #432 from Proudnoob4/fix/uninitialized-sgprs
0ddbce8 2026-11-24 Marvis Merge pull request #433 from r4tedx/tests/serialise-valu-gpu-tests
dd54de2 2026-11-24 Marvis Merge pull request #448 from Dufftwt/fix/min-max-signed-zeros
fd92558 2026-11-24 Marvis Merge pull request #520 from Quat3rnion/victory-xasync-payload
a4d7b16 2026-11-24 Marvis Merge pull request #565 from SAitken91/shader/conservative-depth-position
204bdcc 2026-11-24 Marvis Merge pull request #611 from AdvaitPalve11/fix/config-show-data-dir
9a92ccf 2026-11-24 Marvis Merge pull request #612 from KoLolX3/fix/phi-liveness-transitive
6f0a10a 2026-11-24 Marvis Merge pull request #634 from SAitken91/shader/image-sample-o
cf0bb43 2026-11-24 Marvis Merge pull request #637 from AlanQiang0418/codex/gfx10-packed-inline-constants
b4b4110 2026-11-24 Marvis Merge pull request #654 from SantannaCarlos/fix/uav-describe
16bd577 2026-11-24 Marvis Merge pull request #666 from lameloball2813-ui/fix/load-probe-target-dir
6db02ea 2026-11-24 Marvis Merge pull request #671 from fuzeofficlal/fix/copyresource-cpu-texture-source
22700b9 2026-11-24 Marvis Merge pull request #675 from lameloball2813-ui/docs/rustdoc-private-links
8d23f07 2026-11-24 Marvis Merge pull request #680 from EatCodeSleep2/fix/wine-staging-upload-128
5cd7421 2026-11-24 Marvis Merge pull request #681 from fuzeofficlal/fix/vertex-buffer-null-unbind
02f404e 2026-11-24 Marvis Merge pull request #685 from catminou4/devin/constant-color-cfg-windows
a51bdfa 2026-11-24 Marvis Merge pull request #696 from catminou4/devin/formats-doc-micro-link
58fed37 2026-11-24 Marvis Merge pull request #706 from Lyrnc/fix/cmp-lg-ordered
2005e35 2026-11-24 Marvis Merge pull request #88 from AndyLatimer/fix/title-size-overflows
c7a5f49 2026-11-24 Marvis Merge pull request #93 from TG0000/perf/bcpack-bucket-lookup
8e79c80 2026-11-24 Marvis CI: clippy and tests on pull requests and main
e09dbb6 2026-11-24 r4. Merge upstream main (8e79c80) into shader/per-lane-phi
74a9d33 2026-11-24 r4. Shader tests: the waterfall tests take the GPU one at a time, with the VALU tests
ca48fbe 2026-11-24 Marvis Merge pull request #566 from r4tedx/shader/per-lane-phi
ffe310e 2026-11-24 Marvis Shaders: the console's waterfall loop over a first-lane table offset
0b35b7f 2026-11-24 Marvis Shaders and GPU: FidelityFX on host and console paths, 64-bit image atomics, the BVH intersection, lane masks, bindless strides
a7fc47a 2026-11-24 Marvis Shaders: a phi edge is live only if a lane can take it to a read
13091f6 2026-11-24 Marvis Shaders: pixel and mesh I/O with reused registers, 16-bit packing and signature rows
27c3dc9 2026-11-24 Marvis Shaders and GPU: title memory at raw addresses through a page table
8af0472 2026-11-24 Marvis Shaders: lane masks spilled and combined in compute programs
4051149 2026-11-24 Marvis GPU: null views for title buffer views that are empty or run past their copy
328eec1 2026-11-24 Marvis Shader tests: instruction semantics and control flow against the console's float mode
75677cb 2026-11-24 Marvis Shaders: buffer sizes read from descriptors, and size-only waterfall loops
0f63757 2026-11-24 Marvis GPU: root views of programs that use title memory go through the page table
77053a0 2026-11-24 Marvis Shaders: accuracy fixes for discards, f16 rounding, denormals, med3 and signed zeros
4f57f46 2026-11-24 Marvis Shaders: cube samples with gradients take their level from the gradients
c75fdba 2026-11-24 Marvis GPU: views the shaders pick from the heap, and one change journal for title memory
d71c388 2026-11-24 Marvis GPU and shaders: bounded root views, and a guard against texture descriptors read as buffers
683ad7a 2026-11-24 Marvis GPU and shaders: rectangle lists through a geometry shader, table constants read in place
7389303 2026-11-24 Marvis Game formats: effects, techniques and passes of the shader database; shader-info --effects
286e33f 2026-11-24 Marvis Game formats: the shader pack; shader-info lists packs
532cb7c 2026-11-24 Marvis Diagnostics: VICTORY_POKE writes title bytes twice a second
c47b529 2026-11-24 Marvis Shader warm-up (translate --cache), a cache keyed by the translator, pipeline queue fixes
819f622 2026-11-24 Marvis translate: progress counts finished shaders, not ones handed out
e0034c2 2026-11-24 Marvis Shaders: register-relative moves reach only what their index can
13bbd6f 2026-11-24 Marvis GPU: one host texture per texture memory, and writable twins for depth textures
224e786 2026-11-24 Marvis GPU tests: the frame pacing test tolerates a busy host
86a5ed6 2026-11-24 Marvis GPU: views off their buffer's elements get their own copy; heap records copied as bytes are mirrored
6d021fe 2026-11-24 Marvis GPU: world frame cost
bacb06a 2026-11-24 Marvis GPU: a heap slot holds its view's own copy
9c58a1c 2026-11-24 Marvis Pipeline pre-build from the shader pack (translate --cache --pipelines); mesh pipelines load from the library
7000573 2026-11-24 Marvis Diagnostics: title memory peeks, slot states, small buffers and 1D/3D read-back in listed frames
70564ae 2026-11-24 Marvis README: status of 2026-11-24
```

## L. Limitacoes desta analise

1. **Sem execucao.** Nada foi compilado nem executado. Nao ha jogo, GPU compativel nem tempo para `cargo test --workspace` neste ambiente. Todo "estado atual" e o codigo do HEAD (`70564ae`), nao comportamento observado.
2. **Sem acesso a rede.** Nao consultei issues ou PRs no GitHub. As referencias `#59`, `#2`, `#16`, `#11`, `#307`, `#604`, `#736`, `#566` sao apenas o que aparece em mensagens de commit e comentarios. Os PRs #4 a #706 foram identificados pelos merges locais (`git log --merges`), sem ler discussao.
3. **Mensagens como fonte de fatos de campo.** Alguns dados so existem na mensagem do commit: "3 de 3 execucoes", "0 de 60", "0.15 s por frame", "11x", "3.5 -> 7.8 GiB/s", "1.3 GB", "8.765/8.765". Foi marcado quando usei isso. Nao ha artefato de medicao no repositorio (benchmarks sao `#[ignore]`).
4. **Ordem do historico.** As datas de autor sao anteriores aos merges (PRs abertos horas antes). Usei ordem topologica (`git rev-list --reverse`) para a linha do tempo e verifiquei ancestralidade (`git merge-base --is-ancestor`) nos casos em que a ordem importava (ramo `shader/per-lane-phi`, heap limitado, prepare/TLS).
5. **Blocos sem diff no pacote de evidencias.** 33 dos 45 blocos eram merges e vieram sem diff. Nesses casos abri o commit de origem do PR (`^2`/`--ancestry-path`) e li o diff real; quando o commit de origem tambem nao estava claro, o achado nao foi usado.
6. **Profundidade desigual.** Os commits de GPU e de shader foram lidos linha a linha; os de HLE menos triviais tambem; os ~50 PRs de uma a tres linhas (docs, clippy, cfg, testes) foram classificados como ruido ou correcao trivial apenas por `--stat` e mensagem. Nao ha catalogo exaustivo de todos os 241 commits.
7. **Nao verificado.** (a) se a regressao D-06 ainda existe em algum caminho nao coberto pela prova; (b) se o `shortcut` do heap NVIDIA (`host.rs:137`) causa fault na pratica; (c) se a politica final de `prepare.rs` e correta em Windows 11 sem o teste que so roda em Windows; (d) qualquer afirmacao sobre o titulo real (nomes de shaders, contagens) que venha so das mensagens.
8. **Ferramenta.** Git somente leitura; nenhum comando de escrita no repositorio foi executado. A unica escrita foi este arquivo.
9. **Modelo.** DeepSeek V4.1 Flash. O documento nao passou por revisao humana; conclusoes marcadas PROVAVEL/HIPOTESE devem ser tratadas como tal.
