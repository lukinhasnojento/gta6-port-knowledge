# SixNG - analise de DeepSeek V4.1 Flash

## A. Identificacao

| Campo | Valor |
|---|---|
| Nome | SixNG |
| URL | `https://github.com/holdmysocks/SixNG.git` (origin; clone local em `repos/SixNG`) |
| Fork ou original | Original. Nao e fork de projeto anterior; e um projeto novo escrito para GTA VI (Xbox Series). |
| Autor principal | Brandon Villar / Brandon `<foiler25@gmail.com>` — 280 de 303 commits (92,4 %) |
| Contribuidores externos (via PR) | Ahmet Dedeler (5), Vlad Bondarenko (5), maple-quartz18 (5), Tarek (3), ifBars (3), Snorps37 (1), labj1987 (1) = 23 commits (7,6 %) |
| Branch analisada | `master` (unico branch local) |
| Commit 0 | `6b68f75adad905b4d5fc8de384572a04314c47a3` — 2026-11-20 — "Initial commit: SixNG, a GTA VI (Xbox Series, RPF9) asset extractor" |
| Ultimo commit | `0b14859a9943183c147cd9f06cbf244f9543f961` — 2026-11-24 — "Vehicles: number plates follow their switches (#223)" |
| Periodo | 2026-11-20 a 2026-11-24 (5 dias corridos com commit) |
| Total de commits em HEAD | 303 |
| Commits em todas as refs | 635 (332 apenas em branches) |
| Branches remotas | 191; 189 nao contidas em HEAD |
| Merges reais em HEAD | 2 (`af96038` sync de 11-21; `06c5cf6` pull request #10) |
| Versoes publicadas | 0.2.0 (`cfb2f57`), 0.2.1 (`ffc8255`), 0.2.2 (`c3c39c9`), 0.3.0 (`507b59b`), 0.3.1 (`f9224b1`), 0.4.0 (`a669c73`), 0.5.0 (`606faf7`), 0.6.0 (`24c863b`), 0.6.1 (`702435c`), 0.7.0 (`f8791cc`), 0.8.0 (`08df174`/`ab122bc`) |
| Tamanho de HEAD | 193 arquivos versionados; 166 arquivos `.rs`; ~81.000 linhas Rust; 258 `#[test]` |
| Commits examinados em profundidade | 45 (pacote de evidencias) + ~60 adicionais por `git show`/`git log -S` para cadeias de causa |
| Modelo / data | DeepSeek V4.1 Flash (opencode-go) — 2026-11-25 |

Metodo: leitura do diff de cada commit candidato (nunca da mensagem como prova), busca de reincidencia com `git log <hash>..HEAD -S<trecho>`, comparacao de blobs entre commits de branch e o commit `(#N)` correspondente em master, e verificacao de presenca em HEAD por `git show HEAD:<arquivo>`. Ver secao L para o que nao foi verificado.

## B. Resumo tecnico

**Proposito.** SixNG extrai assets do GTA VI (build Xbox Series, arquivos RPF9): modelos (drawables e fragments), texturas (dicionarios `.#td` e stores `ttpf`), mapa (placements, colisao, sector grids, heightmap), audio, texto, metadados (PSO/meta `0x7a`), animacao, roupas/peds, veiculos (cores, rodas, luzes) e escrita em GLB/PNG/DDS/JSON.

**Arquitetura.** Tres crates:

- `sixng-core` — 75 modulos na raiz de `src/`; o nucleo faz tudo: `scan.rs` (indice por carving de streams zlib, porque o TOC do RPF9 nao e lido — ver D-02), `rpf9.rs`, `resource.rs`, `meta7a.rs`/`pso.rs` (meta auto-descritivo), `drawable.rs`/`bounds.rs` (geometria, skin, LOD), `texture.rs`/`ttpf.rs`/`gfx10.rs` (tiling AMD Gfx10 + formatos), os decoders BCPack (`bc1pack.rs`, `bc3pack.rs`, `bc45pack.rs`, `bc6pack.rs`, `bc6h.rs`, `bc7pack.rs` + tabelas), `material.rs`/`matlib.rs` (shaders e bibliotecas de material em camadas), `export.rs`/`gltf.rs`/`glb` (GLB, materiais), `veh*` (veiculos), `ped*`/`wardrobe.rs`/`outfit.rs` (peds e roupas), `groom.rs`/`hair*`/`furlink.rs` (cabelo e pelo), `world.rs`/`sectorgrid.rs`/`heightmap.rs`/`navmesh.rs`, `audio.rs`/`audiometa.rs`, `text.rs`/`rbf.rs`/`scriptdoc.rs`/`timecycle.rs`/`ptfx.rs`.
- `sixng-cli` — frente de linha de comando (`sixng <subcomando>`), 128 alteracoes no historico; e o contrato de saida (JSON/JSONL/GLB) do projeto.
- `sixng-gui` — visualizador egui/wgpu (`model_view.rs` com 211 alteracoes, o arquivo mais mexido depois do README) mais um renderer Metal 4 opcional para macOS.

**Estado atual (HEAD `0b14859`).** Projeto maduro para um desenvolvimento de 5 dias, com 0.8.0 e 258 testes. Cobertura funcional declarada no README (que funciona como especificacao viva, com datas e numeros medidos): todos os seis formatos BC com chunks BCPack decodificando; 357 K texturas dos stores; 2,25 M entidades de mapa; veiculos com esqueleto, cores por peca, rodas, luzes, direcao e liveries; personagens com outfit rigado e animacoes. Limitacoes estruturais remanescentes: TOC do RPF9 nao lido (D-02), Shader/parametros nomeados parcialmente, script code cifrado, e um conjunto de regras de jogo inferidas por medicao em vez de extraidas (documentadas como tal no README).

**Padrao de engenharia dominante.** Quase toda correcao vem acompanhada de uma metrica antes/depois (dB contra o mip vizinho, contagem de violacoes de regra, contagem de registros ruins, contagem de texturas afetadas) e de uma frase explicita sobre o que *nao* foi verificado. Esse e o traco mais reutilizavel do repositorio.

## C. Linha do tempo

**F0 — 2026-11-20, genese (12 commits, `6b68f75`..`c3c39c9`).** O commit inicial ja entrega o esqueleto completo: scanner de streams, RPF9, meta `0x7a`, drawables, export GLB, GUI e CLI. No mesmo dia: Library/GUI (`668d8c1`), animais e personagens montados por partes (`244936b`), normais reconstruidas quando o stream nao as traz (`b0eaf94`), audio (`cb9833d`), posicoes 24-bit e padding de chunk `ttpf` (`3afa6d1`), arrays e cube maps (`4ac636e`). Versoes 0.2.0-0.2.2. O indice nasce com `INDEX_VERSION = "4"` — ou seja, o cache de indice ja e versionado no dia zero.

**F1 — 2026-11-21, texturas e dados de jogo (32 commits, `9b4f281`..`3af3a69`).** Fase de "fazer os dados aparecerem": nomes de shader/parametro/campo (`9b4f281`), LOD links (`952e65e`), Map 3D com orcamento de RAM/VRAM (`42603f2`, `3af3a69`), PSO e tabelas de veiculo (`dc64093`), contagem real de mips (`8fa1281`), formatos de 1/2/4/8 bytes em todos os tile modes (`a07ac25`, `37f52ae`), paletas de tint embutidas e `COLOR_0` (`8e8cb9b`, `1dfe787`, `37f52ae`), stores de content-pack (`5a3e35b`), fallbacks de textura ausente (`6282863`). Entram os primeiros PRs externos: CI nos tres SOs (`e077498`), app universal macOS (`10fb323`), suporte macOS e orcamento de memoria (`e4d1380`), `f32` literais (`2ea5b0c`). Versoes 0.3.0, 0.3.1, 0.4.0.

**F2 — 2026-11-22, a campanha BCPack e a largura de escopo (78 commits, `08e7457`..`28c0129`).** O dia mais decisivo tecnicamente. Comeca a decodificacao dos chunks BCPack (o codec de hardware do Xbox Series): BC1 (`08e7457`, PR #6), BC3 (`c2150e4`), BC7 (`543ce5e`), BC4/BC5/BC6H (`d7430ce`). Como o BCPack codifica por cadeia de predicao a partir de 256 "preset slots" que nao existem no arquivo, cada formato exige *ajustar* tabelas contra o mip vizinho — e isso vira uma serie de refits que atravessa os tres dias seguintes. No mesmo dia: meta/audio/PTFX/texto/script-docs, animais e fur (`f6eb4dd`), grooms (`488e7b8`), Map 3D com streaming por celulas (`f555c9f`, `94f699e`), o bug de "modelo antigo sob nome novo" (`3d510aa`, issue #9), o decoder BC6H proprio substituindo `texture2ddecoder` (`2979c57`, PR #23), o estado one-region→two-region do BC6H (`3df6347`, PR #15), meshlets com contagem u32 e pintura de carroceria restrita (`a52f939`, PR #29), eixos de posicao pelo header (`e7aae6b`, PR #32). Versoes 0.5.0, 0.6.0, 0.6.1, 0.7.0.

**F3 — 2026-11-23, veiculos, pessoas e convergencia das tabelas (90 commits, `d2dbb28`..`9ce31d3`).** Explosao de dominio: esqueletos completos, rodas duplas/laterais, roof down (`cae7bd0`), todos os clips e discos de freio (`57533e0`), pecas de mod (`e9c6391`), armas montadas por partes (`cdedcea`), luzes e padroes de sirene (`9a23bb8`, `1206155`), direcao (`8e2a5aa`), steering lock do handling (`db16349`), cores de peca (quatro abordagens sucessivas: `3b50a33` → `ed9ecf4` → `4c2fe2a` → `1f4318f`), rig e poses de personagem (`d84b18f`), outfit com clips (`d207a54`), trem de pouso retratil e seus consertos (`c1bec41` → `95029ce` → `20219d8`), correcoes do "whole-fleet check" (`cd044f5`, PR #77), `extract --force` (`6e23570`, PR #79), scroll e foco da lista de modelos (`f74276f`, PR #106), 3 regras exatas de preset BCPack (`8eff158`, PR #125), 9 slots BC1 (`67c84ab`, PR #126). Versao 0.8.0.

**F4 — 2026-11-24, correcao de dados lidos errado, index v6/v7 e o que ficou fora (91 commits, `daf3867`..`0b14859`).** O dia das correcoes de maior alcance: parametros de material lidos no offset errado desde sempre (`2cf005b`, PR #163 — "valores errados em exports anteriores a 2026-11-24"), tipos de constante (`9588bbd`, PR #183), UV de 4 camadas por constante e nao por suposicao (`938c75f`, PR #171), index v6 para o agrupamento de streams (`b7fbe3a`, PR #156) e v7 para chunks armados sem compressao (`d7e9bdc`, PR #221), `--float` sem clamp (`e23a836`, PR #211), decode BC em bandas paralelas (`9d08383`, PR #206), sector grids (`98b28c4`, PR #210), cutscenes (`bc1e967`, PR #201), cabelo gerado como o programa do jogo (`d3aeb9a`, PR #212), placas de veiculo (`0b14859`, PR #223). **No mesmo dia, fora de master:** a leitura do TOC do RPF9 (`5433b45`, 1.294 linhas, branch `claude/rpf9-toc-decrypt`) — a maior peca de codigo do repositorio que nunca foi mergeada (D-02).

**F5 — pos-HEAD (nao em master).** `origin/claude/rpf9-toc` (`gamecipher.rs` + `rpf9toc.rs`) e `origin/claude/people-hair-lighting` (`hairlight.rs`, serie "WIP hair lighting", 18 commits). Sao as duas unicas branches cujo conteudo inclui arquivos que HEAD nao tem (verificado por comparacao de `git ls-tree` de todas as 191 refs remotas).

## D. Catalogo de defeitos

> Convencao: "introduzido" e o commit mais antigo em que o defeito esta presente de forma verificavel. Como `6b68f75` e uma importacao em bloco de ~todo o nucleo, varios defeitos nascem ali por construcao; nesses casos isso e dito explicitamente.

### D-01 — Recurso RAGE dividido em duas entradas de indice (modelos vazios)
- **Sintomas:** 22 recursos saiam partidos em dois: o "cabeca" virava entrada sem tipo e o resto um drawable sem modelos, ou um dicionario de textura com os dados fora de faixa (11 avisos "texture data out of range" num export completo). Modelos de mapa exportavam vazios.
- **Causa raiz:** `group_streams` decidia continuacao por `last_u == CHUNK || !starts_new_file(kind) || misaligned_obj`. Um stream contiguo de RAGE-object que comeca exatamente em 4 KiB + 0x10 (o alinhamento normal de um recurso) era tratado como *novo* recurso, mesmo quando o page-map pointer do recurso anterior ainda apontava para bytes alem do system page ja acumulado.
- **Introduzido:** `6b68f75` (logica de agrupamento presente desde o commit inicial).
- **Tentativas:** `015d17a` (branch) = `b7fbe3a` (master, PR #156) — blob de `scan.rs` identico (`aa8bc39`).
- **Correcao definitiva:** `b7fbe3a` (com `593c106`, que sobe `INDEX_VERSION` 5 → 6 para forcar reconstrucao; `levels.rpf` passa de ~3 min para ~6 min por causa do scan cacheado).
- **Arquivos:** `crates/sixng-core/src/scan.rs`, `crates/sixng-core/src/lib.rs`, `crates/sixng-core/examples/regroup.rs` (novo), `README.md`.
- **Evidencia:** `git show b7fbe3a` — `fn sys_need_of` (`pm >> 28 == 5` → `(pm & 0x0FFF_FFFF) + 8`), `sys_short = out.last().is_some_and(|e| e.usize < sys_need)` somado a condicao `cont`, e dois testes `group_tests` que fixam o caso (um recurso que continua na fronteira, e um recurso novo depois de pagina completa). README: entradas em `levels.rpf` 523.552 → 523.532; nomeados 227.766 → 236.604.
- **Estado atual:** corrigido e coberto por teste; `sys_need_of` presente em HEAD.
- **Confianca:** CONFIRMADA.

### D-02 — TOC do RPF9 nunca lido: nomes e offsets vem de carving (limitacao estrutural)
- **Sintomas:** o indice e reconstruido varrendo streams zlib (1,12 M streams, 98,3 % de cobertura de bytes, ~3-6 min por arquivo); nomes recuperados "de dentro dos recursos"; D-01 e possivel apenas porque o TOC nao e lido.
- **Causa raiz:** o TOC do RPF9 e cifrado; o projeto decidiu nao decifrar (README em HEAD: "the TOC is encrypted and SixNG does not decrypt it").
- **Introduzido:** decisao de arquitetura em `6b68f75`.
- **Tentativas:** `5433b45` ("Read the RPF9 table of contents (toc, toc-extract)", 1.294 linhas: `gamecipher.rs` 408, `rpf9toc.rs` 755, CLI `toc`/`toc-extract`, mais `e4b377c` mergeando master nessa branch). **Nao esta em master** — `git ls-tree HEAD` nao tem `gamecipher.rs` nem `rpf9toc.rs`, e o README de HEAD ainda traz a linha "not read".
- **Correcao definitiva:** **SEM CORRECAO no master.** A correcao existe e e a maior peca nao mergeada do repositorio.
- **Arquivos:** `crates/sixng-core/src/gamecipher.rs`, `crates/sixng-core/src/rpf9toc.rs`, `crates/sixng-cli/src/main.rs` (branch `origin/claude/rpf9-toc`).
- **Evidencia:** `git show 5433b45 --stat`; `git show origin/claude/rpf9-toc:crates/sixng-core/src/gamecipher.rs` — mapeia o executavel do jogo como imagem (sem executa-lo) e chama as rotinas de cifra do proprio jogo: `BLOCK_SETKEY 0x55d_c660`, `STREAM_SETKEY 0x55d_c890`, `KEY_LIST 0x66e_5900`, 24 chaves, com `BUILD_STAMP 0x6af7_83d4` fixando uma build; `SUPPORTED = cfg!(windows x86_64)`. Outra ferramenta (`toc-extract`) escreve os arquivos que o indice de carving nao consegue ler, validando cada um contra os checksums zlib e o tamanho da tabela.
- **Estado atual:** aberto em master; resolvido (ao menos para uma build do jogo) na branch.
- **Confianca:** CONFIRMADA (ausencia em master e conteudo da branch verificados por arvore).

### D-03 — Normal map iluminado em frame tangente errado e flip duplo de verde
- **Sintomas:** normal maps iluminados "de lados arbitrarios" em modelos cujo UV0 e UV1 divergem; o flip de verde no shader estava certo em alguns modelos e errado em outros.
- **Causa raiz:** duas somadas. (a) O frame tangente armazenado nos modelos segue o **segundo** conjunto de UV (medido: `rhino_hi` concorda 100 % com UV1 e 49 % com UV0; jeans de ped 99,9 % / 93,5 %), mas o normal map e amostrado em UV0. (b) O shader aplicava `tn.y = -tn.y` ("DirectX green") sobre esse frame, o que dobra a inversao quando o bitangente ja apontava ao contrario.
- **Introduzido:** `6b68f75` (o flip `tn.y` e o uso do frame armazenado existem desde o commit inicial; `git log -S"tn.y = -tn.y"` aponta so para `6b68f75` e para o conserto).
- **Tentativas/correcao definitiva:** `839e852` (branch) = `3bfa416` (master, PR #120). Conteudo identico nos arquivos de codigo.
- **Arquivos:** `crates/sixng-gui/src/glb.rs` (`fn uv0_tangents`, 46 linhas: acumula dP/du e dP/dv por triangulo em UV0, ortogonaliza contra a normal, deriva `w` de `dot(cross(n,t), b)`), `crates/sixng-gui/src/shader.wgsl`, `crates/sixng-gui/src/render3d_metal4.rs`, `crates/sixng-core/src/drawable.rs` (comentario documentando que o frame armazenado e UV1).
- **Evidencia:** `git show 3bfa416` remove `tn.y = -tn.y` e adiciona `if has_tangents && uv_sets >= 1 { uv0_tangents(&mut v, &indices); }`.
- **Estado atual:** corrigido no visualizador; `uv0_tangents` em HEAD, aplicado em 3 pontos (malha, e dois caminhos de export/cache). **O exportador GLB continua escrevendo `TANGENT` a partir do frame armazenado (UV1)** — `crates/sixng-core/src/gltf.rs` em HEAD escreve `attrs["TANGENT"] = json!(b.vec4(&t))` sem reconstruir por UV0. Ver G-02.
- **Confianca:** CONFIRMADA (visualizador); PROVAVEL (inconsistencia remanescente no GLB exportado).

### D-04 — Modelo antigo exibido sob o nome novo (corrida de carregamento assincrono)
- **Sintomas:** ao trocar de modelo rapidamente, um carregamento lento anterior terminava depois e substituia o modelo recem-escolhido; o viewport mostrava um modelo antigo com o nome novo. Escolhas feitas durante um carregamento eram ignoradas; um carregamento que falhava deixava o modelo anterior na tela.
- **Causa raiz:** `Pending = Arc<Mutex<Option<Result<Loaded, String>>>>` sem identidade de requisicao: o ultimo a escrever vencia, nao o ultimo a ser pedido.
- **Introduzido:** `6b68f75` (o tipo `Pending` nasce no commit inicial).
- **Correcao:** `3d510aa` (issue #9, 2026-11-22): `Pending` passa a `Arc<Mutex<Option<(u64, Result<Loaded,String>)>>>`, `load_gen: AtomicU64`, `finish_load(&slot, &latest, gen, r)` descarta resultado cujo `gen` nao e o mais novo; falha limpa o viewport; `--then` para testar a troca.
- **Arquivos:** `crates/sixng-gui/src/model_view.rs`, `crates/sixng-gui/src/main.rs`.
- **Evidencia:** `git show 3d510aa` — `type Pending` muda de `Option<Result<Loaded,String>>` para `Option<(u64, Result<Loaded,String>)>`, com `load_gen: Arc<AtomicU64>` e `next_load()`. Reincidencia em caminho irmao: `cd044f5` (PR #77) mostra que o **carregador de rodas** sofria do mesmo mal ("the viewer loads default wheels first; when those finished loading after the vehicle's own wheels had been requested, they were shown") e o conserta com chave `rim|tyre` por carga — um dia depois da correcao de D-04.
- **Estado atual:** corrigido nos dois caminhos (geracao de carga e chave de roda). O proprio commit `cd044f5` declara: "the same stale-load race may exist for brake models" — nao verificado depois.
- **Confianca:** CONFIRMADA.

### D-05 — Instancias de prop-batch decodificadas como lixo (posicoes 1e10-1e35, escalas inf)
- **Sintomas:** 130.066 de 24,9 M instancias em `levels` (e 5.728 de 57.545 no minigame pack) com posicao 0 ou 1e10-1e35 e escala 0, 1e3-1e18 ou `inf`. Tambem ~950 arrays de entidade e ~4.500 outros arrays de extensao.
- **Causa raiz:** duas. (a) `Meta::ptr` tentava a leitura de ponteiro real **antes** da referencia de pool; uma referencia de pool com offset de byte 0x500-0x5FF ou 0x600-0x6FF e byte baixo zero (`0x50000003` = bloco 3, offset 0x500) tambem le como ponteiro sys/gfx alguns bytes dentro da pagina. (b) Quando o ultimo stream de um arquivo de mapa preenche chunks inteiros de 64 KiB (105 arquivos em `levels`), sua graphics page nao fica logo apos: e comprimida como stream proprio alguns KiB adiante e indexada como entrada `bin` separada.
- **Introduzido:** `6b68f75` (leitor de meta e `load_meta`).
- **Correcao:** `38de2e5` (branch) = `d986342` (master, PR #48).
- **Arquivos:** `crates/sixng-core/src/meta7a.rs`, `crates/sixng-core/src/world.rs`/`resource.rs` (carregamento de meta), `README.md`.
- **Evidencia:** mensagem de `38de2e5` com o racional e a validacao ("all 14.471 ambiguous values point into a block of the expected struct"; "0 bad records in either archive by the same checks"; luzes 775.132 → 778.985 posicionadas). README de HEAD: entidades 1,47 M → 2,25 M apos o conserto de split de recurso.
- **Estado atual:** corrigido. O salto de 1,47 M para 2,25 M entidades no README e atribuido ao conserto de *split de recurso* (D-01), nao a este commit: este elimina os registros com lixo dentro das instancias que ja existiam.
- **Confianca:** CONFIRMADA.

### D-06 — BC6H decodificado errado por tres bugs da dependencia `texture2ddecoder`
- **Sintomas:** BC6H_UF16/SF16 saiam com cores erradas; 234 de 401 texturas UF16 e 17 de 25 SF16 mudaram de imagem apos o conserto.
- **Causa raiz:** `texture2ddecoder 0.1.2`: (a) `unquantize` sem sinal mapeia `1 << (bits-1)` para 0xFFFF onde a especificacao manda `(1 << bits) - 1`; (b) o modo one-region de 16 bits monta a mascara de endpoint em `u16`, estourando para 0 em build release — o segundo endpoint virava 0; (c) blocos com sinal eram tratados como sign-magnitude.
- **Introduzido:** `d7430ce` (primeira decodificacao BC4/BC5/BC6H BCPack, 2026-11-22) ao adotar a crate.
- **Correcao:** `2979c57` (branch) = `4b40433` (master, PR #23) — `bc6h.rs` passa a decodificar por especificacao (UF16/SF16) sobre `bc6pack::unpack`; `texture.rs` usa o novo decoder para `Fmt::Bc6u`/`Bc6s`; convencao de saida 8-bit preservada.
- **Refinamento posterior:** `742079c` = `e0ebb4b` (PR #24, ifBars) — selecao do sinal do zero com sinal **depois** do escalonamento final.
- **Arquivos:** `crates/sixng-core/src/bc6h.rs` (novo), `crates/sixng-core/src/texture.rs`.
- **Evidencia:** mensagem de `2979c57` lista os tres desvios e a medicao por arquivo (`data/ttdb/lighting/lighting.ttpf`: 234/401 e 17/25 PNGs de mip 0 mudam; demais formatos byte-identicos).
- **Estado atual:** corrigido; `bc6h.rs` presente em HEAD.
- **Confianca:** CONFIRMADA.

### D-07 — BC6H: estado herdado de bloco one-region para two-region produz speckle
- **Sintomas:** blocos depois de uma mudanca de modo one-region → two-region saiam salpicados.
- **Causa raiz:** um bloco one-region transformado deixa `w` (escrito na largura de delta do campo y, deslocado por metade da faixa) como estado y da regiao 1; a regra antiga copiava `w` cru, que so vale para modos nao-transformados.
- **Introduzido:** `d7430ce` (implementacao do decoder BC6H BCPack).
- **Correcao:** `3df6347` (branch) = `a4063d3` (master, PR #15); tabelas de preset reajustadas sob a nova regra.
- **Arquivos:** `crates/sixng-core/src/bc6pack.rs`, `crates/sixng-core/src/bc6pack_tables.rs`.
- **Evidencia:** mensagem de `3df6347`: blocos dentro de 0,15 stops 81 % → 86 %; tonemapped 23,4 → 27,2 dB; 23 de 36 texturas melhores, nenhuma pior (props ate +26 dB).
- **Estado atual:** corrigido.
- **Confianca:** CONFIRMADA.

### D-08 — Tabelas de preset BCPack (BC6H) com deslocamento constante: speckle em areas planas
- **Sintomas:** speckle em areas planas de texturas BC6H-only; violacoes da regra "no-wrap" (um endpoint em modo transformado e `w + delta` e nunca sai de `[0, 2^bits)`) em 23.162 de 30,2 M endpoints dependentes de tabela.
- **Causa raiz:** as 256 posicoes de preset nao existem no arquivo (o BCPack so guarda o indice); cada slot foi obtido por ajuste. Slots com erro sistematico de sinal constante deslocam a cadeia inteira de blocos que comeca neles.
- **Introduzido:** `d7430ce` (tabelas inicialmente ajustadas sem as regras).
- **Serie de tentativas (todas em `bc6pack_tables.rs`):** `455cf5d` (PR #20, score index-aware), `9329a2b` (PR #21, ultimos slots), `b57cdb0` (PR #58, refit conjunto + presets de indice ancorados), `480d32d` (PR #103, no-wrap), `2a70bca` (11-23, penalidade de wrap: 23.162 → 599; medias 22,0 → 24,7 dB), `23ff140` (PR #134: chunks de lighting pre-filtrado deixam de fornecer alvo de ajuste, mas continuam valendo para a regra; 599 → 221), `18bc31c` (PR #170, continuidade de borda: 46 slots com passo log2 > 1,0; down-mip 25,1 → 32,7 dB), `eb4373c` (PR #205, +17 slots).
- **Correcao definitiva:** **parcial por natureza.** Nao existe valor "correto" a extrair; o estado e o melhor ajuste medido. HEAD: violacoes 85 de 30,2 M; mip check das 89 texturas BC6H-only: media 30,49 dB, 4,5 % abaixo de 20 dB.
- **Arquivos:** `crates/sixng-core/src/bc6pack_tables.rs` (22 alteracoes — o arquivo mais retocado do nucleo).
- **Evidencia:** mensagens medindo cada passo; README de HEAD admite "blocks whose reference chain starts in a mis-fitted slot (near the start of a chunk) can come out speckled".
- **Estado atual:** funcional, com speckle residual admitido perto do inicio de chunk.
- **Confianca:** CONFIRMADA (serie e metricas); o residuo e CONFIRMADO como admitido, nao como quantificado por esta analise.

### D-09 — BC1: seis dos nove slots de janela substituidos eram piores que o default
- **Sintomas:** buracos pretos em texturas BC1 (blocos em modo de tres cores usando indice 3, o transparente) e, depois, texturas ate 13 dB piores em alguns casos.
- **Causa raiz:** a regra usada para substituir slots ("um bloco em modo three-colour nao usa indice 3") e *quase* exata — texturas com alpha recortado a quebram — e a selecao foi feita sobre poucas texturas de referencia, dando peso desproporcional a cada uma.
- **Introduzido:** `08e7457` (PR #6: BC1 passa a decodificar com a janela default; a regra three-colour ja e violada em 0,018 % dos blocos nas cadeias que partem da predicao default e ate 24 % nas que partem de certos slots — os "buracos pretos").
- **Tentativa (parcialmente errada):** `67c84ab`/`9ce31d3` (PR #126, primeira substituicao de 9 slots).
- **Correcao:** `063d35e` = `4406256` (PR #154): cada textura conta uma vez; seis dos nove slots (48, 73, 120, 159, 194, 208) voltam ao default porque "are no better than the default under that measure" (o 194 piorava texturas em ate 13 dB); 17 slots sao adicionados. Resultado: 1.834 texturas melhores / 81 piores; abaixo de 20 dB 0,79 % → 0,40 %; blocos three-colour usando indice 3 (amostra de 5 %) 121.918 → 50.334.
- **Arquivos:** `crates/sixng-core/src/bc1pack.rs`, `crates/sixng-core/src/bc1pack_tables.rs`.
- **Estado atual:** corrigido e medido; `OVERRIDES` em HEAD.
- **Confianca:** CONFIRMADA (inclusive a regressao parcial, ver H-02).

### D-10 — Ordem de eixos da posicao escolhida pelos normais em vez do header
- **Sintomas:** 114 malhas (em 51 modelos) decodificadas em eixo errado — uma corrente de 52.586 triangulos em pe e ortogonal aos LODs inferiores; a unica de 55 cabecas amostradas em outro eixo; pecas de rotor de helicoptero desalinhadas.
- **Causa raiz:** o header do blob de malha em +0x6C diz a ordem usada pelo quantizador (2 = identidade, 1 e 0 = rotacionado), mas o decoder tambem pontuava as seis ordens contra as normais armazenadas e trocava quando outra pontuava 0,05 acima. As trocas vinham de malhas com poucas normais utilizaveis (37 de 114 com menos de 20 triangulos) ou normais que nao servem a ordem nenhuma.
- **Introduzido:** `6b68f75`.
- **Correcao:** `e7aae6b` (branch) = `a7c3a1a` (master, PR #32): o header e aceito para 0, 1 e 2; o score so decide em outros valores.
- **Arquivos:** `crates/sixng-core/src/drawable.rs`.
- **Evidencia:** mensagem de `e7aae6b` (7 casos conferidos por outro caminho, todos errados antes; ordem do header pontua ≥ 0,95 em 90 % das malhas). Limitacao declarada: 107 malhas trocadas nao conferidas uma a uma.
- **Estado atual:** corrigido; exportacoes antigas afetadas precisam ser refeitas.
- **Confianca:** CONFIRMADA.

### D-11 — Valores de parametro de material lidos no offset errado (quase todos errados)
- **Sintomas:** "nearly every value came from the wrong place"; um material de ground dava 28 valores (errados) em vez de 86; o segundo buffer de constantes das shaders de 4 camadas nao era lido.
- **Causa raiz:** o descritor de parametro guarda numero de componentes (1-4 floats), **offset em bytes** e qual buffer de constantes; o leitor antigo tratava o offset como indice de float dentro do proprio bloco.
- **Introduzido:** `6b68f75`.
- **Correcao:** `2cf005b` (PR #163) para os valores; `9588bbd` (PR #183) para o tipo (`float`..`float4`, `int`/`int3` escritos como inteiros e nao como floats minusculos), com a tabela de tipos vinda de `shaders.rpf` (`<work>/index/shader_params.json`).
- **Arquivos:** `crates/sixng-core/src/drawable.rs`, `crates/sixng-core/src/material.rs`, `crates/sixng-core/src/export.rs`, `README.md`.
- **Evidencia:** README de HEAD: "**Parameter values were wrong in exports made before 2026-11-24 and should be re-exported**"; validacao em ~13.000 materiais de tres arquivos (todo parametro cai dentro do seu buffer; cabeca 38 de 38).
- **Estado atual:** corrigido, com invalidacao explicita de cache documentada.
- **Confianca:** CONFIRMADA.

### D-12 — Carroceria do veiculo aplicada a interior, carcasas de lampada e pecas mecanicas
- **Sintomas:** interior, lanternas e pecas mecanicas recebiam a cor da pintura, porque `standard_vehicle_2lyr` cobre o carro inteiro.
- **Causa raiz:** classificacao "body paint" por familia de shader, sem olhar os detalhes de material nem os ossos em que as malhas penduram.
- **Introduzido:** `6b68f75` (`fn is_paint` por familia desde o inicio).
- **Correcao:** `a52f939` (branch) = `e6ac2c6` (master, PR #29): um material so e pintura quando nenhuma das duas camadas liga um detail texture (`lyr0diffusetex`, `lyr1diffusetex`) **e** as malhas penduram em ossos de carroceria; caso contrario recebe stand-in neutro (`stand_in: "trim"`). Regra escrita em `classification.paint_rule`.
- **Arquivos:** `crates/sixng-core/src/material.rs`, `crates/sixng-core/src/export.rs`.
- **Estado atual:** corrigido por inferencia declarada (a paleta por peca ainda nao era decodificada na epoca; ver D-13/D-14).
- **Confianca:** CONFIRMADA.

### D-13 — Tint de prop aplicado a shaders de veiculo: cacamba em faixas de arco-iris
- **Sintomas:** pecas de veiculo sem part set (cacambas compartilhadas) exportadas com faixas de arco-iris.
- **Causa raiz:** a paleta de tint do modelo (embutida, indexada pelo canal azul da cor de vertice) foi aplicada como `COLOR_0` para **todo** modelo. Em veiculos, a cor de vertice nao e indice de paleta: e id de peca.
- **Introduzido:** `8e8cb9b` ("GLB: apply the model's tint palette as COLOR_0", 2026-11-21) — uma correcao valida para props de mapa que virou regressao em veiculos.
- **Correcao:** `fe08116` (branch) = `be16132` (master, PR #97): `.filter(|st| st.vehicle_palette.is_none())` antes de aplicar o tint; `EXPORT_VERSION` 7 → 8.
- **Arquivos:** `crates/sixng-core/src/export.rs`, `crates/sixng-core/src/vehcolour.rs`.
- **Evidencia:** `git show fe08116`.
- **Estado atual:** corrigido; filtro presente em HEAD. Ver H-01.
- **Confianca:** CONFIRMADA.

### D-14 — Cores de peca do veiculo tiradas da primeira configuracao
- **Sintomas:** `comet7` com o deck traseiro preto: a primeira configuracao deixa o deck no registro "black", enquanto 15 das outras 16 ligam boot e carroceria a pintura.
- **Causa raiz:** as cores de peca de um modelo eram tomadas da **primeira** configuracao encontrada; configuracoes divergem entre si.
- **Introduzido:** `3b50a33`/`f1f0ec6` (PR #59, primeira extracao de cores por part set).
- **Correcao:** `ed9ecf4` = `032dacf` (PR #66): o part set do modelo passa a ser o resultado que **a maioria** das configuracoes da (o mais antigo em caso de empate); 314 de 319 modelos ligados ganham set proprio; `vehicle_parts.json` formato 5, exports versao 4.
- **Refinamentos seguintes:** `50d41ea`/`1f4318f` (PR #137, cor por pixel a partir da paleta de material), `d9acd24`/`4c2fe2a` (PR #136, pecas de biblioteca herdam a cor do veiculo em que estao), `be45ad2`/`0c8c3f0` (PR #124, pecas de mod), `27c045c`/`1051ea5` (PR #184, pecas montadas alteram cores — body kit assume a pintura; `PART_REF 0x51aa_b7c1` tipo 5).
- **Arquivos:** `crates/sixng-core/src/vehcolour.rs` (35 alteracoes), `vehexport.rs`, `export.rs`, `crates/sixng-gui/src/model_view.rs`.
- **Estado atual:** a abordagem "resultado da maioria" foi superada pela paleta por pixel; o defeito especifico (primeira configuracao) esta corrigido.
- **Confianca:** CONFIRMADA.

### D-15 — Slot de cor do pneu aplicado: pneu inteiro vermelho ou cinza claro
- **Sintomas:** cinco veiculos (boxville, burrito, journey, rentalbus, stanier) tinham o pneu pintado (vermelho ou cinza claro) em algumas configuracoes.
- **Causa raiz:** uma regra decodificada corretamente (canal `f3bd038d`, dez pecas de biblioteca ligando o grupo do pneu — id 35, o unico id de todo modelo `tyre_*` — a esse canal) foi aplicada como se fosse cor de peca; o que o jogo faz com ela nao esta decodificado.
- **Introduzido:** `4c2fe2a`/`d9acd24` (PR #136, pecas de biblioteca passam a receber cores).
- **Correcao:** `8aebab3` = `5f7a635` (PR #198): `if ch == TYRE_COLOUR_CHANNEL { continue; }` — o slot fica **nao aplicado** e o pneu mantem a borracha do part set; `EXPORT_VERSION` 23 → 24; `fn is_tyre_shader` em `material.rs`; follow-up `a42230e` inclui o shader `_lod` (`b994d670`), que era o unico material de `tyre_truck_001`.
- **Arquivos:** `crates/sixng-core/src/vehcolour.rs`, `material.rs`, `vehexport.rs`, `export.rs`, `model_view.rs`.
- **Estado atual:** corrigido como "nao aplicar" — decisao consciente de nao adivinhar.
- **Confianca:** CONFIRMADA.

### D-16 — Strings de text tables lidas sempre na primeira pagina do blob
- **Sintomas:** cerca de um terco das strings das tabelas grandes saia como a cauda de outra string.
- **Causa raiz:** o `u64` de referencia em +8 e `(offset << 20) | numero_do_bloco` (bloco contado de 1 na tabela de blocos); so o offset era lido, sempre dentro da primeira pagina.
- **Introduzido:** `6b68f75`.
- **Correcao:** `5492154`: le todas as paginas de blob; 14.482 tabelas de `data.rpf` decodificam com 0 erros e toda referencia de uma tabela amostrada cai no inicio de uma string.
- **Arquivos:** `crates/sixng-core/src/text.rs`.
- **Estado atual:** corrigido.
- **Confianca:** CONFIRMADA.

### D-17 — Meshlets com contagem de 32 bits lida como u16: moto desenhada em espinhos
- **Sintomas:** `alvinov1_hi` (70.057 triangulos, 69.901 entradas de remap) cortada para 4.521 triangulos e desenhada como espinhos.
- **Causa raiz:** as contagens de triangulo e remap no header sao `u32`, nao `u16` seguido de zero; o inicio de remap de 16 bits nos descritores estoura nessa malha e precisa ser desenrolado na ordem dos meshlets.
- **Introduzido:** `6b68f75`.
- **Correcao:** `a52f939` = `e6ac2c6` (PR #29).
- **Arquivos:** `crates/sixng-core/src/drawable.rs`.
- **Estado atual:** corrigido.
- **Confianca:** CONFIRMADA.

### D-18 — Lista de modelos pulando e roubando o foco do teclado
- **Sintomas:** cada Up/Down rolava a lista para deixar a linha a 120 px do topo; a lista pulava porque egui move o foco com as setas e rola ate o widget focado, enquanto as setas tambem moviam o cursor.
- **Causa raiz:** (a) scroll sempre por offset fixo em vez de "o minimo para caber"; (b) a linha clicada retinha o foco de teclado; (c) a lista nao tinha id fixo.
- **Introduzido:** `6b68f75`.
- **Correcao:** `f74276f` (branch) = `91f866a` (master, PR #106), mergeado junto com a ordenacao por nome (`a65aaf6`, que resolve o conflito de campos do picker).
- **Arquivos:** `crates/sixng-gui/src/model_picker.rs`.
- **Estado atual:** corrigido (`surrender_focus()`, `id_salt("picker_list")`, `list_view` guardando offset e altura).
- **Confianca:** CONFIRMADA.

### D-19 — `extract --force` nao reescrevia texturas compartilhadas
- **Sintomas:** PNGs de `ttpf`/texturas compartilhadas e texturas embutidas em modelo eram pulados quando o arquivo ja existia, entao **nenhuma correcao de decoder chegava a uma pasta de saida existente**.
- **Causa raiz:** cache por existencia de arquivo sem invalidacao por versao/forca.
- **Introduzido:** `6b68f75` (comportamento de skip); o impacto cresce com os consertos de decoder de F2/F3.
- **Correcao:** `6e23570` (branch) = `74bd58b` (master, PR #79): com `--force` cada textura e decodificada de novo, uma vez por execucao.
- **Arquivos:** `crates/sixng-cli/src/main.rs`, `crates/sixng-core/src/export.rs`, `crates/sixng-core/src/ttpf.rs`.
- **Estado atual:** corrigido. Observacao: e o mesmo problema de classe que D-11, resolvido ali por bump de versao de export e aqui por flag — nao ha politica unica de invalidacao (ver I-03).
- **Confianca:** CONFIRMADA.

### D-20 — Expressao da lista de cores do sticker (defeito de branch, nunca chegou ao master)
- **Sintomas:** nenhum observavel no master.
- **Causa raiz:** `1af1b8b` escreveu `"colours": ["colorRed","colorGreen","colorBlue"].iter().map(|n| hex(u(named(s, n)))).collect::<Vec<_>>()` — com `n: &&str` passado a `fn named<'v>(v: &'v Value, name: &str)`. A coercao de deref `&&str → &str` e valida, portanto compilava e produzia o mesmo resultado.
- **Correcao:** `ab56ca6` ("Stickers: fix the colour list expression") troca por tres chamadas explicitas.
- **Arquivos:** `crates/sixng-core/src/vehstickers.rs`.
- **Evidencia:** `git log HEAD -S'["colorRed"'` nao retorna nada: a forma antiga **nunca existiu no historico de master**; o commit que entrou foi `6a7e082` (PR #180) com a forma ja corrigida.
- **Estado atual:** irrelevante — sem efeito funcional e sem alcance em master. Classificado como **ruido** no que diz respeito a comportamento.
- **Confianca:** CONFIRMADA (ausencia em master); PROVAVEL (equivalencia semantica — nao foi executado `cargo check` sobre o commit antigo).

### D-21 — Array R16F lido no dobro do tamanho (cada 1 KiB alternado sem uso)
- **Sintomas:** volume/array de textura R16_FLOAT lido com o dobro do tamanho real, com metade dos KiB "vazios".
- **Causa raiz:** tamanho do elemento/estrato assumido errado para R16F em array (cada 1 KiB alternado ficava sem uso).
- **Introduzido:** `6b68f75` (leitor de store do commit inicial).
- **Correcao:** `98693e0` (2026-11-21).
- **Arquivos:** `crates/sixng-core/src/ttpf.rs`/`gfx10.rs`.
- **Estado atual:** corrigido.
- **Confianca:** CONFIRMADA (mensagem + diff de uma linha de raciocinio; sem medicao detalhada no corpo).

### D-22 — Contagem de mips subcontada pelo descritor (~44,9 mil texturas embaralhadas)
- **Sintomas:** cerca de 44,9 mil texturas (a maioria normais de terreno 1024² em tile modes 1 e 2, ~2,7 mil sprites de UI) decodificadas com uma faixa embaralhada dos mips menores no topo, ou como ruido em sprites pequenos.
- **Causa raiz:** o campo de mips do descritor subconta; o mip 0 fica no fim, entao a leitura errada desloca a cadeia.
- **Introduzido:** `6b68f75` (leitura do descritor pelo campo declarado).
- **Correcao:** `8fa1281` ("recover the real mip count from the store layout", 2026-11-21).
- **Arquivos:** `crates/sixng-core/src/ttpf.rs`.
- **Estado atual:** corrigido; PNGs em cache se redecodificam sozinhos (README).
- **Confianca:** CONFIRMADA.

### D-23 — UV das camadas de ground em 4 camadas: 64 por unidade em vez de 128
- **Sintomas:** escala de detalhe errada nas camadas do shader de ground de 4 camadas (suposicao medida por ajuste de plano).
- **Causa raiz:** a escala era inferida do ajuste de UV por modelo ("64 vezes por unidade de TEXCOORD_0"), e nao lida das constantes do material e do codigo do shader.
- **Introduzido:** `2ef8b8c` ("Measure multilayer UV mappings per material", 2026-11-22), que fixou a suposicao de 64 repeticoes por unidade de TEXCOORD_0; confirmado em HEAD.
- **Correcao:** `938c75f` (PR #171) = `6c2d97d` (branch): `four_layer_uv()` le `724bed95` (seletor de UV por camada), `4344d2cc` (escala/offset do primeiro set) e `9ecaaa1e` (do segundo) e substitui os valores supostos; confianca passa a "high (material constants and shader code)".
- **Arquivos:** `crates/sixng-core/src/material.rs`.
- **Estado atual:** corrigido; `four_layer_uv` em HEAD.
- **Confianca:** CONFIRMADA.

### D-24 — Cor da configuracao tratada como linear quando e valor de display (pintura escura)
- **Sintomas:** no visualizador, a pintura escolhida pela configuracao saia escura/dessaturada; no bake do `sixng vehicle` (branch), a cor era convertida de sRGB para linear duas vezes, "which made the baked paint too dark".
- **Causa raiz:** a cor da configuracao e das paletas e valor de **display** (sRGB); o renderer e o GLB esperam linear. O visualizador usava o valor direto, e o bake usava `material::parse_paint`, que ja aplica `lin()` (conversao sRGB→linear).
- **Introduzido:** `2f67255` ("Show decoded vehicle data in the model viewer (spawn colours, rims, tyres, palettes, wheel size)", 2026-11-21) para o visualizador; `609e916` (branch `claude/vehicle-export`, 2026-11-24) para o bake.
- **Correcao:** `2985152` (branch) = `e56a7bd` (master, PR #113, "a configuration's body colour is a display value, not linear") adiciona `srgb_to_linear()` e o aplica nos dois pontos que usam cor de configuracao; `23336c7` (branch) troca `parse_paint` pela leitura direta dos bytes (`v >> 16 & 255` etc., sem `lin()`).
- **Arquivos:** `crates/sixng-gui/src/model_view.rs`, `crates/sixng-core/src/vehexport.rs`.
- **Evidencia:** `git show 2985152` (`self.look.paint = *c` → `srgb_to_linear(*c)`; `self.look.paint = col` → `srgb_to_linear(col)`); `git log HEAD -S"self.look.paint = col"` → `2f67255` e `2985152`; `git log HEAD -S"crate::material::parse_paint" -- crates/sixng-core/src/vehexport.rs` → **vazio**, ou seja a dupla conversao do bake nunca existiu no historico de master (mesmo padrao de D-20).
- **Estado atual:** corrigido nos dois caminhos; nao ha commit posterior mexendo nesses pontos.
- **Confianca:** CONFIRMADA (mecanismo e hashes); o defeito de bake e classificado como defeito de branch, sem alcance em master.

### Triagem dos 45 commits do pacote de evidencias

Veredito dado pelo diff, nao pela mensagem. `CORRIGE` = remove um comportamento errado verificavel; `AJUSTA` = recalibra tabela/numero por medicao (nem sempre melhora tudo); `FUNCIONALIDADE` = adiciona capacidade, nao conserta; `DUPLICATA` = mesmo conteudo de outro commit (rebase/squash); `SYNC` = merge de master no branch; `RUÍDO` = sem efeito funcional.

| commit | veredito | nota |
|---|---|---|
| `5433b45` | FUNCIONALIDADE | TOC do RPF9; nao esta em master (D-02) |
| `5ae2d5c` | AJUSTA | desempata slots 5/85 do BC6H (D-08) |
| `783eaa4` | AJUSTA (corrige) | devolve o invariante no-wrap 288 → 85 (H-03) |
| `27c045c` | FUNCIONALIDADE | `part_patches`: peca montada muda cor (D-14) |
| `ab56ca6` | RUÍDO | expressao de tipo; nunca chegou a master (D-20) |
| `f44f70d` | AJUSTA (introduz regressao) | melhora imagem, piora no-wrap 221 → 288 (H-03) |
| `593c106` | CORRIGE | sobe `INDEX_VERSION` 5 → 6 (D-01) |
| `015d17a` | CORRIGE | `sys_need_of` (D-01); par de `b7fbe3a` |
| `023216e` | AJUSTA | no-wrap 599 → 221 (D-08) |
| `67c84ab` | AJUSTA (introduz regressao) | 9 slots BC1, 6 ruins (D-09, H-02) |
| `8eff158` | AJUSTA | BC3/BC4/BC5/BC7 sob 3 regras exatas |
| `839e852` | CORRIGE | frame tangente por UV0 (D-03); par de `3bfa416` |
| `ef4e085` | FUNCIONALIDADE | `asset` de 64 bits nos elementos de peca |
| `e63034d` | SYNC | merge de master em `pr110-fix` |
| `9aec774` | RUÍDO | README; conserta uma linha quebrada |
| `01815ed` | SYNC | merge em `picker-sort-fix` |
| `a65aaf6` | SYNC | merge de #106 no picker (conflito de campos) |
| `f74276f` | CORRIGE | scroll/foco da lista (D-18) |
| `dd23f11` | CORRIGE | "Used by" para pele de ped (issue #96) |
| `2a70bca` | CORRIGE | no-wrap 23.162 → 599 (D-08) |
| `d10d53c` | DUPLICATA | blob de `bc6pack_tables.rs` igual a `2a70bca` |
| `20219d8` | CORRIGE | tirantes do trem de pouso; par de `95029ce` |
| `95029ce` | DUPLICATA | lado branch de `20219d8` |
| `c1bec41` | FUNCIONALIDADE | trem de pouso retratil |
| `fc6f3e0` | CORRIGE | looks de pecas proprias, faces, camadas sob jaqueta |
| `97d5064` | SYNC | merge em `fix/force-textures` |
| `6e23570` | CORRIGE | `--force` reescreve texturas (D-19) |
| `4d055d8` | CORRIGE | whole-fleet: rodas, aro, barcos, magenta (D-04) |
| `cd044f5` | CORRIGE | lado branch de `4d055d8` |
| `c32565e` | FUNCIONALIDADE | vidros laterais descem |
| `c3de3a6` | CORRIGE | nomes de tag/body class no wardrobe (#70) |
| `db16349` | FUNCIONALIDADE | steering lock do handling |
| `ed9ecf4` | CORRIGE | cor de peca pela maioria das configuracoes (D-14) |
| `8e4fd84` | FUNCIONALIDADE | nomes legiveis no picker |
| `8e2a5aa` | FUNCIONALIDADE | direcao; par de `a0740c4` |
| `9a23bb8` | FUNCIONALIDADE | lampadas; par de `e6e9095` |
| `a0740c4` | DUPLICATA | lado branch de `8e2a5aa` |
| `e6e9095` | DUPLICATA | lado branch de `9a23bb8` |
| `25b0aea` | FUNCIONALIDADE | grupos de pintura, razao de aro |
| `3b50a33` | FUNCIONALIDADE (introduz defeito) | cores por part set; origem de D-14 e D-15 |
| `bca2da3` | AJUSTA | refit conjunto do BC6H + presets ancorados |
| `57533e0` | FUNCIONALIDADE | clips, capo/porta-malas, discos, rodas por pneu |
| `cae7bd0` | FUNCIONALIDADE | esqueletos completos, rodas duplas/laterais |
| `cdedcea` | FUNCIONALIDADE | armas montadas por partes |
| `247c422` | FUNCIONALIDADE | tamanho/posicao de roda pelos discos de colisao |

Nenhum dos 45 commits **introduziu** defeito sem que o proprio diff mostrasse a compensacao: as duas regressoes reais do pacote (`f44f70d` no invariante no-wrap, `67c84ab` em 6 slots BC1) sao de **calibracao**, nao de logica — e ambas foram corrigidas por commits posteriores do mesmo autor.

## E. Relacao de causa e efeito

Cadeias explicitas, com hashes. Setas entre commits indicam ordem cronologica verificada por `git log --date=format` (nao por data de autoria).

**E-01 — Carving sem TOC → split de recurso → index v6.**
`6b68f75` (TOC nao decifrado; indice por streams) → agrupamento por regra heuristica → `b7fbe3a`/`015d17a` (page-map pointer define o fim do system page) → 22 recursos deixam de se partir, 227.766 → 236.604 entradas nomeadas → `593c106` sobe `INDEX_VERSION` 5 → 6 porque o formato do indice muda → `d7e9bdc` sobe 6 → 7 quando o scanner passa a juntar chunks de 64 KiB armados sem compressao (efeito colateral: indices antigos precisam ser refeitos; ~6 min por arquivo com scan cacheado).

**E-02 — Decoder de terceiro com bug → decoder proprio → regra de estado do modo.** 
`d7430ce` (adoção de `texture2ddecoder` para BC6H) → tres desvios da spec → `2979c57` substitui por `bc6h.rs` → `3df6347` corrige o estado `w` na transicao one-region → two-region → `742079c` corrige o sinal do zero apos o escalonamento → `455cf5d` troca o score de ajuste, que era "leniente na segunda regiao" (README: blocos batendo com o vizinho decodificado 58 % → 62 %).

**E-03 — Tabelas ajustadas por metrica errada: um conserto que piora a regra e depois se recupera.** 
`2a70bca` (11-23, penalidade de wrap: 599 violacoes restantes) → `23ff140` (PR #134, 05:23 de 11-24: 599 → 221, descartando chunks de lighting como alvo) → `18bc31c` (PR #170, 10:18: continuidade de borda melhora a imagem, **mas** as violacoes voltam de 221 para 288) → `783eaa4` (288 → 85, mais 16 slots) → `5ae2d5c` (slot 5 reajustado com o 85 mantido, porque os dois trocavam uma constante entre varreduras; uma textura 19,7 → 28,6 dB) → `eb4373c` (PR #205, +17 slots, 85 mantidas). Resultado: a regra volta a ser quase satisfeita sem perder qualidade — mas so porque a regra foi medida de novo depois de otimizar a imagem.

**E-04 — Fix de slots BC1 medido de forma enviesada → meia reversao.** 
`08e7457` (BC1 com janela default) → `67c84ab`/`9ce31d3` (9 slots substituidos) → `063d35e`/`4406256` (cada textura conta uma vez: 6 dos 9 slots piores que o default, um deles -13 dB; 17 slots novos) → estado atual.

**E-05 — Corrida de carregamento: conserto local, reincidencia em caminho irmao.** 
`6b68f75` (`Pending` sem identidade) → `3d510aa` (geracao de carga no modelo) → `cd044f5` (mesmo defeito no carregador de rodas: "the viewer loads default wheels first... they were shown and the load counted as done"; conserto por chave `rim|tyre`) → `cd044f5` tambem declara que a corrida "may exist for brake models" (aberto).

**E-06 — Paleta de tint: correcao valida para props, regressao em veiculos.** 
`8e8cb9b` (tint como `COLOR_0`, 11-21) → veiculos sem part set com faixas arco-iris → `fe08116`/`be16132` (PR #97, filtro `vehicle_palette.is_none()`) → `1f4318f`/`50d41ea` (PR #137, cores de peca por pixel, que e o caminho correto para veiculos).

**E-07 — Cores de peca: quatro abordagens em dois dias.** 
`3b50a33` (pintura + part set) → `ed9ecf4` (resultado que a maioria das configuracoes da) → `4c2fe2a` (pecas de biblioteca herdam cor do veiculo) → `1f4318f` (cor por pixel via paleta de material, que resolve o que o vertex colour quer dizer: id de peca) → `27c045c` (peca montada altera cores: body kit assume a pintura). Cada passo corrige uma pergunta que o anterior respondia errado, e cada um sobe `EXPORT_VERSION`/`FORMAT` — o que mostra que o autor tratou o cache de export como contrato versionado.

**E-08 — Erro de leitura de dados, propagado por tres dias de features.** 
`6b68f75` (offset de constante lido como indice de float) → todos os `materials.json` ate 11-24 com valores errados → features construidas sobre esses valores (four-layer, tint, wear) → `2cf005b` (PR #163, offset em bytes + segundo buffer) → `938c75f` (PR #171, UV por constante em vez de suposicao) → `9588bbd` (PR #183, tipos) → README passa a exigir reexportacao.

**E-09 — Eixo de posicao: heuristica vence o header.** 
`6b68f75` (score contra normais) → 114 malhas em eixo errado → `e7aae6b`/`a7c3a1a` (header aceito para 0/1/2) → exportacoes antigas invalidadas.

**E-10 — TOC do RPF9: a correcao que existe e nao foi mergeada.** 
`6b68f75` (carving) → `5433b45` na branch `claude/rpf9-toc` (mapeia o executavel do jogo e chama as rotinas de cifra, 1.294 linhas) → `e4b377c` (merge de master nessa branch) → **nada em master**. Consequencia: `sys_need_of` (D-01), a recuperacao de nomes por conteudo e todo o custo de indexacao existem para compensar uma funcionalidade que ja esta escrita e nao entrou.

## F. Correcoes importantes

**F-01 — `sys_need_of`: agrupar streams pelo page-map pointer (`b7fbe3a`).**
O que estava errado: a decisao de "novo recurso" ignorava quanto system page o recurso anterior ainda precisava. Por que a correcao funciona: o ponteiro de page-map (+0x08, `0x5xxxxxxx`) e a unica informacao no proprio fluxo que diz o tamanho do system page; `(pm & 0x0FFF_FFFF) + 8` da o total necessario, e a comparacao `e.usize < sys_need` transforma "parece um recurso novo" em "ainda e a continuacao". Por que e reutilizavel: o mesmo principio (usar o ponteiro interno em vez de heuristica de alinhamento) vale para qualquer formato RAGE com page map. Dependencias: nenhuma alem do proprio `scan.rs`. Risco ao adaptar: assume `pm >> 28 == 5` (ponteiro de sistema); se a build usar outro espaco de endereco, o teste `group_tests` existente protege o comportamento antigo mas nao o novo formato.

**F-02 — `uv0_tangents`: construir o frame do UV com que o mapa e amostrado (`3bfa416`).**
O que estava errado: usar o frame armazenado (UV1) para amostrar um mapa em UV0, com um flip fixo de verde por cima.
Por que funciona: o frame e reconstruido do proprio conjunto de UV usado na amostragem (`dP/du`, `dP/dv` acumulados por triangulo, ortogonalizados contra a normal, `w` derivado do sinal de `cross(n,t)·b`); e o shader deixa de aplicar o flip porque a convencao DirectX agora esta embutida no `w`.
Reutilizavel: sim, para qualquer pipeline que receba tangentes de um DCC/engine com convencao de UV diferente da do consumidor.
Risco: o custo e O(triangulos) na carga e o frame e descartado para vertices sem area de UV (que mantem o frame armazenado — um caso misto).

**F-03 — Ordem de leitura de ponteiro: pool antes de ponteiro real (`38de2e5`).**
O que estava errado: `Meta::ptr` preferia a leitura de ponteiro real; valores como `0x50000003` sao ambiguos por construcao.
Por que funciona: no espaco de valores observado, a interpretacao de pool e a correta para todos os 14.471 valores ambiguos (todos caem num bloco do struct esperado); a ordem inverte o custo do erro. Reutilizavel: e um caso de "desambiguacao por ordem, validada por censo" — o padrao (medir todos os casos ambiguos, nao uma amostra) e o que da confianca.
Risco: se um arquivo futuro usar ponteiro real com offset baixo, ele sera lido como pool; o censo cobre os arquivos atuais.

**F-04 — Invalidacao de cache por versao (`INDEX_VERSION`, `EXPORT_VERSION`, `FORMAT`).**
O projeto tem tres numeros de versao independentes e os sobe sempre que o conteudo/estrutura muda: `INDEX_VERSION` 4 → 5 → 6 → 7 (7 em `d7e9bdc`); `vehcolour::EXPORT_VERSION` 7 → 8 (D-13), e depois um numero por mudanca de `model.json`, com a legenda no proprio codigo (21 = `_COLOR1` no GLB, 22 = `part_patches`, 23 = `sliding_doors`, 24 = slot de cor do pneu nao aplicado, D-15); `vehcolour::FORMAT` 18 → 19 (`27c045c`) → 20 (D-15); `vehicles::FORMAT` 11 → 12 (`ef4e085`). E a defesa sistematica contra o defeito de D-19 (cache velho escondendo conserto de decoder). Reutilizavel: sim — mas o repositorio mostra o limite: `--force` foi necessario **porque** nem todo cache esta coberto por versao (texturas compartilhadas), ver I-03.

**F-05 — Decoder por especificacao em vez de dependencia (`2979c57`).**
Quando a crate de terceiro tem tres desvios da especificacao, o conserto e escrever o decoder. Reutilizavel como decisao: o `bc6h.rs` e pequeno, testavel contra a spec e remove um risco de dependencia. Custou: uma tarde e a obrigacao de reprocessar 234+17 texturas.

**F-06 — Medir a regra depois de otimizar o objetivo (`783eaa4`).**
A licao operacional mais nitida do repositorio: `18bc31c` melhorou a imagem e **piorou** um invariante (no-wrap 221 → 288). O conserto seguinte mediu o invariante de novo e o trouxe a 85. Isso so aconteceu porque cada commit reporta as duas metricas (qualidade e regra) lado a lado. Reutilizavel: sempre reportar a metrica de correcao junto da metrica de qualidade, no mesmo commit.

## G. Problemas ainda existentes

**G-01 — TOC do RPF9 nao lido em master (D-02).** Impacto: custo de indexacao (3-6 min por arquivo), nomes recuperados por conteudo, e a classe inteira de defeitos de agrupamento (D-01, `d7e9bdc`) existe apenas por causa disso. A correcao esta escrita na branch `claude/rpf9-toc` e nao foi mergeada. Confianca: CONFIRMADA.

**G-02 — `TANGENT` exportado segue UV1 enquanto o normal map usa TEXCOORD_0 (D-03).** `crates/sixng-core/src/gltf.rs` em HEAD escreve `attrs["TANGENT"]` a partir de `p.tangents` (frame armazenado), sem reconstrucao por UV0 como a que o visualizador ganhou em `3bfa416`. Consequencia provavel: GLBs de modelos com UV0 != UV1 iluminam normal maps errado em qualquer visualizador externo (glTF 2.0 nao permite escolher o conjunto de UV do normal map). Nao ha commit posterior tratando disso. Confianca: PROVAVEL (o dado que sustenta e o proprio comentario do commit: frame armazenado = UV1 em 100 %, UV0 em 49 %).

**G-03 — Speckle residual em BC7/BC6H.** README de HEAD: "blocks whose reference chain starts in a mis-fitted slot (near the start of a chunk) can come out speckled". Nao quantificado por esta analise. Confianca: CONFIRMADA (declarado).

**G-04 — Corrida de carregamento possivelmente ainda em modelos de freio.** `cd044f5` corrigiu rodas por chave e declarou: "the same stale-load race may exist for brake models". Nao ha commit posterior que trate freios explicitamente. Confianca: NAO VERIFICADA (declarada pelo autor; nao inspecionado o caminho de freio em HEAD).

**G-05 — Regras de jogo inferidas, nao extraidas.** Varias decisoes do projeto sao inferencia com margem declarada: escala de roda `1.625 × wheelScale` ("fitted by eye on phoenix", "not the game's verified rule"); razao de aro em duas rodas (`wheelScale` = raio do aro, heuristica 0.55-0.95); magenta = "sem cor" (inferencia); regra de cabelo sob chapeu (`002d90f`, e `f184275` a documenta como stand-in); "cada peca conta uma vez" no BC1; tinta de ped (61-68 % de concordancia com as tags de cor). Sao limitacoes assumidas, nao defeitos ocultos — mas delimitam o que o export significa. Confianca: CONFIRMADA (documentadas).

**G-06 — Script code e alguns metadados nao decodificados.** README: o bloco de 51,7 MB de scripts compilados tem entropia 7,85-8,0 e nenhum header de compressao; "looks encrypted, and SixNG does not decrypt". Consequencia: 36 % dos tags de `script-docs` e 7 % dos valores de string resolvem. Confianca: CONFIRMADA.

**G-07 — `bvh4` sem geometria.** 1.237 estruturas de aceleracao; 90,7 % dos bytes sao zero; o arquivo "holds nothing the model does not". Nao e defeito, e limite do dado. Confianca: CONFIRMADA.

**G-08 — Trabalho de iluminacao de cabelo/pelo nao mergeado.** `hairlight.rs` so existe em `origin/claude/people-hair-lighting` (18 commits com prefixo "WIP hair lighting"). Master tem `hairgen.rs`/`hairlink.rs`/`hairtone.rs` (geometria e cor), nao o modelo de iluminacao de fibra. Confianca: CONFIRMADA (arquivo ausente em HEAD).

**G-09 — 107 malhas trocadas de eixo nao reconferidas (D-10).** O proprio commit declara a lacuna. Confianca: CONFIRMADA (declarada).

**G-10 — Sem politica unica de invalidacao de cache.** Tres mecanismos coexistem (versao de indice, versao de export/formato, `--force`), e D-19 nasceu exatamente do buraco entre eles. Confianca: CONFIRMADA (por inspecao dos tres).

## H. Regressoes

**H-01 — Tint de prop aplicado a veiculos (faixas arco-iris).** Correcao valida (`8e8cb9b`) que quebrou outro dominio; consertada por filtro em `fe08116`/`be16132` (PR #97), um dia e meio depois. E o caso mais claro de "a correcao de hoje e a regressao de amanha" no repositorio, porque a paleta de tint e um dado de prop e a cor de vertice de veiculo tem outro significado. Confianca: CONFIRMADA.

**H-02 — 6 dos 9 slots BC1 substituidos eram piores que o default.** `9ce31d3` (PR #126) piorou texturas (uma delas em 13 dB) antes de `4406256` (PR #154) reverter seis slots. A regressao foi causada pela **metrica** (poucas texturas de referencia, cada uma pesando muito), nao pela regra. Confianca: CONFIRMADA (a mensagem de `063d35e` nomeia os seis slots).

**H-03 — Refit BC6H que violou a regra no-wrap (221 → 288).** `18bc31c` (PR #170) melhorou down-mip de 25,1 para 32,7 dB e simultaneamente elevou violacoes de 221 para 288 — regressao explicita em um invariante que ja havia sido corrigido (`23ff140`). Recuperada em `783eaa4` (85) e mantida em `eb4373c`. Confianca: CONFIRMADA (numeros nas mensagens em sequencia).

**H-04 — Flip de verde que compensava um frame errado.** O shader aplicava `tn.y = -tn.y` com o comentario "// DirectX green" desde o commit inicial, sobre um frame que vinha de UV1 enquanto o mapa era amostrado em UV0. O flip estava certo quando o bitangente derivado apontava ao contrario e errado nos demais casos; quando o frame passou a ser reconstruido por UV0, o flip teve de sair (`3bfa416`). Nao e a regressao de um commit unico, mas duas compensacoes parciais cujo produto se anulava em parte dos modelos — o que explica por que o defeito sobreviveu a tres dias de trabalho visual. Confianca: PROVAVEL.

**H-05 — `extract --force` como sintoma de cache sem versao.** Nao e uma regressao de codigo, mas de processo: cada conserto de decoder (BC6H `2979c57`, BC1 `08e7457`, mips `8fa1281`) ficava invisivel para quem ja tinha pasta de saida. Confianca: CONFIRMADA.

**H-06 — Rebase duplicando commits no historico visivel.** 332 commits so existem em branches; em pares verificados o codigo e byte-identico ao commit `(#N)` de master (`2a70bca`/`d10d53c`, `a0740c4`/`8e2a5aa`, `4d055d8`/`cd044f5`, `2979c57`/`4b40433`, `015d17a`/`b7fbe3a`: blob do arquivo principal igual). Nao e regressao funcional, mas **e** uma regressao de rastreabilidade: `git log --all` mostra 635 commits para um trabalho de 303, e um leitor que procure "o que mudou" encontra a mesma mudanca duas vezes com hashes diferentes. Confianca: CONFIRMADA.

## I. Conhecimento reutilizavel

**I-01 — Indice de formato binario por carving, com versao e com correcao dirigida por ponteiro interno.**
Problema resolvido: extrair de um container cujo diretorio e cifrado. Implementacao: varrer streams zlib, classificar por cabecalho, agrupar continuacoes por tres sinais (chunk de 64 KiB exato, tipo que nao inicia arquivo, desalinhamento), e — o ponto nao obvio — **estender o grupo enquanto o ponteiro interno de page map do proprio recurso nao estiver coberto** (`sys_need_of`). Repo: SixNG. Commits: `6b68f75`, `b7fbe3a`, `d7e9bdc`. Arquivos: `crates/sixng-core/src/scan.rs`. Dependencias: nenhuma. Riscos ao adaptar: o sinal `pm >> 28 == 5` e especifico do espaco de endereco do console; a heuristica precisa de teste unitario com streams sinteticos (o repositorio tem: `group_tests`). Recomendacao: copiar o padrao (ponteiro interno como fonte de verdade + versao de cache), nao os numeros.

**I-02 — Reconstruir frame tangente do UV de amostragem.**
Problema: tangentes armazenadas por um engine com convencao de UV diferente da do consumidor. Implementacao: acumular `dP/du`, `dP/dv` por triangulo (com sinal do determinante), ortogonalizar contra a normal, derivar `w` do sinal de `cross(n,t)·b`; nao aplicar flip de verde. Repo: SixNG, `3bfa416`, `crates/sixng-gui/src/glb.rs::uv0_tangents`. Riscos: perde o frame para vertices sem area de UV; custo linear em triangulos na carga. Recomendacao: aplicar tambem no **exportador** (e o que falta em G-02).

**I-03 — Invalidacao de cache: versionar o artefato, nao o codigo.**
Problema: conserto de decoder invisivel para quem tem saida antiga. Implementacao observada: `INDEX_VERSION` para o indice, `EXPORT_VERSION`/`FORMAT` para os JSON de export, e a mensagem ao usuario no README ("should be re-exported"). Falha observada: texturas compartilhadas ficaram fora e precisaram de `--force` (`6e23570`). Recomendacao: um unico `CACHE_VERSION` que cubra **todos** os artefatos derivados, com o `--force` como escape e nao como politica.

**I-04 — Ajustar tabela ausente por medicao contra o mip vizinho, com a regra como restricao.**
Problema: BCPack nao guarda as 256 posicoes de preset. Implementacao: ajustar os slots contra o mip seguinte decodificado ("plain reference"), incluindo a asercao de que o resultado e *plausivel* (no-wrap, three-colour sem indice 3, p-bit igual entre endpoints de um subset, `e0 >= e1` em BC4/BC5) como penalidade, e rejeitar slots cuja contribuicao nao se sustenta em held-out. Repo: SixNG, `bc6pack_tables.rs`/`bc7pack_tables.rs`/`bc1pack_tables.rs`/`bc45pack_tables.rs`/`bc3pack_tables.rs`; commits `8eff158`, `063d35e`, `783eaa4`. Riscos: **cada metrica nova pode reverter slots antigos** (H-02) — o conjunto de validacao tem de ser fixo e por textura, nao por par. Recomendacao: guardar o conjunto de validacao e o script de ajuste junto das tabelas (no repositorio os ajustes nao estao versionados — ver L).

**I-05 — Codificar geometria/estado em "desambiguacao por ordem, validada por censo".**
`Meta::ptr` prefere pool a ponteiro real; a escolha e justificada por um censo de **todos** os 14.471 valores ambiguos, nao por amostra (`38de2e5`). Padrao reutilizavel: quando um campo tem duas leituras possiveis, medir todos os casos ambiguos e escolher a ordem que zera os absurdos (posicoes 1e10, escalas inf).

**I-06 — README como especificacao viva com datas e numeros.**
240 alteracoes no README (mais que qualquer arquivo de codigo). Cada linha de estado carrega a data da mudanca e a medicao ("since 2026-11-21", "81 % → 86 %", "before, 22 resources were split"). Reutilizavel como pratica: para projetos de engenharia reversa, o README e o unico lugar onde "o que se sabe" e "quao certo se sabe" cabem juntos; changelog separado perderia a segunda metade.

**I-07 — Chamar as rotinas de cifra do proprio executavel do jogo (branch `claude/rpf9-toc`).**
Problema: TOC cifrado, chaves derivadas em runtime. Implementacao: mapear o PE como imagem (sem executa-lo), prover apenas as rotinas de memoria C que as rotinas usadas importam, **trap em todo outro import**, e chamar `setkey`/`decrypt` de bloco e de stream nos enderecos fixos de uma build identificada por `BUILD_STAMP` + `IMAGE_SIZE`; lista de 24 chaves em `KEY_LIST 0x66e_5900`, com `key_flag` 2. Repo: SixNG, `5433b45`, `crates/sixng-core/src/gamecipher.rs`. Dependencias: Windows x64 + AVX2; uma unica build do jogo. Riscos: offsets fixos quebram em qualquer atualizacao do jogo; executar codigo de terceiro no processo (ainda que sem `CreateProcess`) e uma superficie de ataque; o codigo so existe fora de master (G-01). Recomendacao: adaptar com deteccao de build por assinatura (nao so por stamp) e fallback para carving.

## J. Conclusao tecnica

SixNG e um projeto de 5 dias e 303 commits que faz o que a maioria dos projetos de extracao nao faz: **mede a propria duvida**. As correcoes mais importantes nao sao de sintaxe nem de logica de negocio — sao de *leitura de dados*: um offset interpretado como indice (D-11), um sinal de ponteiro antes do outro (D-05), um eixo escolhido por heuristica em vez de pelo header (D-10), um frame tangente do UV errado (D-03), um TOC que nunca foi lido (D-02). O padrao se repete: o codigo estava correto em relacao a uma suposicao errada sobre o formato, e o conserto consistiu em substituir a suposicao pela fonte de verdade (header, ponteiro interno, constante do material, especificacao).

O segundo traco e a **disciplina de medicao**: quase todo conserto vem com antes/depois em unidade verificavel (dB contra o mip vizinho, contagem de violacoes de uma regra, numero de registros ruins, numero de texturas afetadas). Isso permitiu ao autor detectar as proprias regressoes — H-02 e H-03 so sao visiveis porque a metrica antiga foi remedida.

O terceiro traco e o custo desse metodo: **`bc6pack_tables.rs` foi alterado 22 vezes** porque nao existe valor correto a extrair; o estado e um ajuste. E a versao de 0.8.0 e boa (media 30,49 dB nas texturas BC6H-only, 4,5 % abaixo de 20 dB, contra 30,27 dB e 5,6 % — e 22,0 dB / 36 % no inicio da serie de refits em 11-23), mas nao e "decodificacao exata" e o README e honesto sobre isso.

O achado negativo mais relevante nao e um defeito: e o **desperdicio de 1.294 linhas prontas**. A leitura do TOC do RPF9 existe, funciona por chamada as rotinas do proprio jogo e resolveria de raiz a limitacao que gerou D-01, D-02 e todo o custo de indexacao — e esta parada em `origin/claude/rpf9-toc` porque o ultimo merge de master e anterior a ela. Junto com ela, 18 commits de iluminacao de cabelo (`hairlight.rs`) tambem ficaram fora. Ou seja: o estado de master nao e o estado do trabalho.

Para um projeto dessa idade, o passivo real e menor do que o volume sugere: dos defeitos catalogados aqui, os de impacto funcional estao corrigidos; o que resta sao limitacoes declaradas (G-03, G-05..G-07, G-09), uma inconsistencia de export provavel (G-02), uma corrida admitida e nao tratada (G-04), trabalho pronto que ficou fora de master (G-01, G-08) e a ausencia de uma politica unica de invalidacao de cache (G-10).

## K. Indice de commits

303 commits de `master`, em ordem cronologica (`git log --reverse`), formato `hash data autor assunto`. Autor abreviado: B = Brandon, BV = Brandon Villar (mesma pessoa, duas identidades de commit), AD = Ahmet Dedeler, VB = Vlad Bondarenko, MQ = maple-quartz18; os demais contribuidores (Tarek, ifBars, Snorps37, labj1987) aparecem com o nome por extenso. Os 332 commits que existem apenas em branches nao entram aqui; ver D, E e L para o que eles contem (dois arquivos nunca mergeados).

```
6b68f75 2026-11-20 B Initial commit: SixNG, a GTA VI (Xbox Series, RPF9) asset extractor
668d8c1 2026-11-20 B GUI: Library home with thumbnails, item pages, Jobs, Status, Settings, setup wizard and Ctrl+K search
244936b 2026-11-20 B GUI: whole animals and characters from their parts, People and Animals categories, readable names
b0eaf94 2026-11-20 B Rebuild normals that the normal stream does not hold
963a969 2026-11-20 B Viewer: Parts panel to toggle animal and character parts
cb9833d 2026-11-20 B Audio: find sound banks in audio.rpf, play and export them
e87c48a 2026-11-20 B Ped clothing drawn opaque; libopus licence notice
cfb2f57 2026-11-20 B Version 0.2.0
3afa6d1 2026-11-20 B Decode 24-bit x/y positions, ttpf chunk padding and the 7-bit format field
ffc8255 2026-11-20 B Version 0.2.1
4ac636e 2026-11-20 B Decode ttpf texture arrays and cube maps
c3c39c9 2026-11-20 B Version 0.2.2
9b4f281 2026-11-21 B Resolve shader, parameter and meta field names
952e65e 2026-11-21 B Resolve archetype LOD distances and LOD parents (lod-links)
42603f2 2026-11-21 B Stream the whole map in Map 3D within RAM and VRAM caps
507b59b 2026-11-21 B Version 0.3.0
9381c68 2026-11-21 B Mark interior-local and group-local placements (interior-entities)
dc64093 2026-11-21 B Decode PSO files, vehicle handling and vehicle tables (paint, wheels)
f9224b1 2026-11-21 B Version 0.3.1
2f67255 2026-11-21 B Show decoded vehicle data in the model viewer (spawn colours, rims, tyres, palettes, wheel size)
3399080 2026-11-21 B Carve Bink videos, export UI texture stores, decode small mip-tail textures
2ea5b0c 2026-11-21 VB Spell out f32 literals the compiler now warns about (#3)
e4d1380 2026-11-21 VB macOS support: file-manager buttons, settings folder, memory budget (#2)
08e7457 2026-11-22 AD Decode BC1 BCPack texture chunks (#6)
a669c73 2026-11-21 B Version 0.4.0
25f2603 2026-11-21 VB Textures tab: browse with the mouse wheel and arrow keys (#7)
10fb323 2026-11-21 VB macos-app.sh --universal: one SixNG.app for Apple Silicon and Intel (#5)
e077498 2026-11-21 VB CI: build and test on Windows, macOS and Linux (#4)
4a7beb1 2026-11-21 B Ped clothing skinning: 8 influences and the shared ped skeleton
af96038 2026-11-21 B Merge remote-tracking branch 'origin/master'
8fa1281 2026-11-21 B Texture stores: recover the real mip count from the store layout
a07ac25 2026-11-21 B Texture stores: decode 1-byte formats (A8, R8) in all tile modes
c39f615 2026-11-21 B Ped clothing: body shape option (game body-shape rig on the shared skeleton)
9b445a4 2026-11-21 B needles example: raw u32 and u64 needles (0x + 8 or 16 hex digits)
c874c53 2026-11-21 B Ped heads: face textures (head skins)
37f52ae 2026-11-21 B Textures: 4-, 2- and 8-byte formats in all tile modes; embedded tint palettes
8e8cb9b 2026-11-21 B GLB: apply the model's tint palette as COLOR_0
e996f50 2026-11-21 B GUI: show the tint palette in the model viewer, with a toggle
5a3e35b 2026-11-21 B Textures: use the content-pack stores (pack/*/data/ttdb)
4306725 2026-11-21 B README: order of the texture store notes
1dfe787 2026-11-21 B Per-instance tint palette rows: tint_row in placements, row layout in model.json
98693e0 2026-11-21 B Decode the R16F array stored at twice its size (every other KiB unused)
6282863 2026-11-21 B Fallback values for texture references not in the install; hide decals without their colour texture
614b45a 2026-11-21 B Document what vertex colour red and green show (still unverified)
3af3a69 2026-11-21 B Map 3D: smoother streaming, GPU memory freed, BC textures
747ada2 2026-11-22 B Vertex colour red: record negative world-level occlusion test
5a5a976 2026-11-22 B Model list: keep generic clothing pieces under 'hide LOD copies' (issue #9)
c2150e4 2026-11-22 B Decode BC3 BCPack texture chunks
f16ad70 2026-11-22 B Map 3D: nearest models first, faster first loads, uploads off the frame
543ce5e 2026-11-22 B Decode BC7 BCPack texture chunks
06c5cf6 2026-11-22 BV Merge pull request #10 from holdmysocks/claude/bcpack-bc7-e505bb
f555c9f 2026-11-22 B Map 3D: map cells, far placements merged per area
80ac189 2026-11-22 B Map 3D: coverage fallbacks for near-camera gaps, flat ground near LODs, dark building glass
94f699e 2026-11-22 B Map 3D: near-camera ground fills only uncovered pixels; full-size SLOD atlases near the camera
4c496f2 2026-11-22 labj1987 Drawable dictionaries: one model, name, link and export per member (#8)
9e66cd0 2026-11-22 B Jobs tab: load the command list in the background
fbb783b 2026-11-22 B Vertex colour red/green: record negative surface, tree, grass-AO and road tests
4914dba 2026-11-22 B Audio: decode multichannel music streams, one Opus or WAV file per sub-stream
606faf7 2026-11-22 B Version 0.5.0
908a301 2026-11-22 AD Audio: read the codec id at 0x1C; export PCM as WAV and XMA2 as RIFF XMA2 (#11)
488e7b8 2026-11-22 B Grooms: export strand hair and fur as GLB line strips
74d8c41 2026-11-22 B Hair picker, cloth and head identity deltas
d7430ce 2026-11-22 BV Decode BC4, BC5 and BC6H BCPack texture chunks
45bb6c2 2026-11-22 B ttpf: BC6H decodes too
d207a54 2026-11-22 AD sixng outfit: a character as one rigged GLB with an animation clip; ped skin fixes
0619601 2026-11-22 B sixng text: decode the localised text tables
a4063d3 2026-11-22 BV BC6H BCPack: fix the state a one-region block hands to a two-region block (#15)
d2dbb28 2026-11-23 MQ Opt-in Metal 4 renderer for macOS; wgpu 30 and eframe 0.36
de366fb 2026-11-22 B Outfit: animals on their root skeleton, rigid one-bone parts, ad-hoc characters, several clips, clip-only files; sixng characters
530b5af 2026-11-22 B sixng ptfx: list particle effect dictionaries, rule names and keyframe curves
159dca9 2026-11-22 B sixng fonts: UI font libraries (DefineFont3) as TrueType and three.js typeface files
4a1bd3d 2026-11-22 B Export: emissive materials light up in glTF (emissive shaders by name and family); --paint sets the car-paint colour
65e230a 2026-11-22 B README: characters, animals and clip-only files, ptfx, fonts, emissive and --paint
6046d19 2026-11-22 B Four-layer ground shaders: multilayer block (layers, lookup weights, macro maps) in model.json and the GLB extras
59ebf24 2026-11-22 Tarek Name the map-format structs and nine more fields
1456a9b 2026-11-22 B Shader constants from the parameter table; four-layer ground materials export them (per-layer vec4s split by layer)
3d510aa 2026-11-22 B Model viewer: never show an older model under a newer name (issue #9)
da11771 2026-11-22 B Map prop_2lyr_batch and the uv3 four-layer variant
2ef8b8c 2026-11-22 B Measure multilayer UV mappings per material
24c863b 2026-11-22 B Version 0.6.0
eeb3076 2026-11-22 B prop_2lyr_batch: overlay weight is 1 - COLOR0.a
e63f21d 2026-11-22 B prop_2lyr notes: tint goes on the base layer, dirt UV unresolved
702e6e0 2026-11-22 B ptfx: decode effect events, keyframe properties and keyframe-texture links
bb94402 2026-11-22 B ptfx: name keyframe properties and behaviours
899b3c1 2026-11-22 B ptfx note: scalar units are unverified
ae93328 2026-11-22 B ptfx: mark the colour-over-life properties; keyframe textures are not life curves
22f1962 2026-11-22 B prop_2lyr: dirt overlay on TEXCOORD_2 (seam test)
455cf5d 2026-11-22 BV BC6H BCPack: refit the preset tables with an index-aware score (#20)
b412ee7 2026-11-22 B Two-layer walls: dirt overlay on TEXCOORD_2 (low confidence)
a225c66 2026-11-22 B Four-layer: record the c35e27af tiling tests (still undecided)
6100458 2026-11-22 B ptfx: resolve particle sprites to the vfx texture store (--sprites writes PNGs)
5492154 2026-11-22 B Text tables: read strings from every blob page
189f3c6 2026-11-22 B sixng annotate: name collision surface ids and road-node streets
0fc9321 2026-11-22 B ptfx: flipbook frame counts and a grid guess for animated sprites
702435c 2026-11-22 B Version 0.6.1
9329a2b 2026-11-22 BV BC5 and BC6H BCPack: fit the last preset slots (#21)
2ffe3c0 2026-11-22 B Audio decoding is an optional feature: build without CMake
67f4740 2026-11-22 BV Vehicles: tyres, lamp lenses, reflectors and lamps get their own classes (#22)
3293a94 2026-11-22 B Minigame pack: R16 textures, meta roots, tail-stream collision, map clip dictionaries, box volumes, audio metadata
4b40433 2026-11-22 BV BC6H: spec-correct decoder in place of texture2ddecoder's (#23)
557bf56 2026-11-22 B Textures: "Used by" panel and sixng texture-users index
ee8481c 2026-11-22 B Model viewer: show only the side-panel controls that apply to the model
b1398e5 2026-11-22 B Release workflow: Linux and macOS archives with checksums in the notes
bc30a1c 2026-11-22 BV Animals: colour textures from material presets; variants without parts (#25)
e0ebb4b 2026-11-22 ifBars BC6H: select the signed zero sign after final scaling (#24)
4d89229 2026-11-22 B Audio metadata: sound graph (waves to bank streams, children, reachable streams); script data documents to XML
c825387 2026-11-22 B Audio metadata: decode the sound header (sub-masks, fixed-size fields, nested masks) and keep each class body
7881042 2026-11-22 B Names: seven texture parameters of the water and interior shaders
f6eb4dd 2026-11-22 BV Animals: fur from the strand grooms, linked by archive order (#26)
a5c1441 2026-11-22 B Audio: child sounds of class 15 and nine more sound classes
520f08b 2026-11-22 B README: bvh4 files hold no geometry of their own
e6ac2c6 2026-11-22 BV Meshlets with 32-bit counts; vehicle interiors off the body paint; fur toggle (#29)
264e630 2026-11-22 B meta: --instances writes every struct instance
c69b22f 2026-11-22 B README: why the last 12 shader parameters stay unnamed
a327383 2026-11-22 BV Fur: grow the full coat from the stored per-triangle strand counts (#30)
a7c3a1a 2026-11-22 BV Meshes: take the axis order from the header, not from the normals (#32)
3890537 2026-11-22 B ped-variations: wardrobe variations to part lists
d724355 2026-11-22 BV BC7 BCPack: fit the last 34 endpoint preset slots (#31)
3c0b691 2026-11-22 ifBars Preserve A8 mask alpha in texture previews (#28)
f8791cc 2026-11-22 B Version 0.7.0
5475976 2026-11-22 BV Hair on people: find the grooms that lie on a head or character (#33)
d246790 2026-11-22 ifBars BC1/2/3 previews: follow the specification's colour modes (#35)
28c0129 2026-11-22 BV ped-variations: tags, dirt presets and texture remaps; outfit --tex-remap (#34)
e6ab25f 2026-11-23 BV ped-variations: dirt and blood preset definitions from common.rpf (#36)
8b5dc72 2026-11-23 BV ttpf: single mips without a tail, cube mip counts, 3D volumes (#37)
ad94efc 2026-11-23 BV Vehicles: leave out runtime screen overlays, wiper masks and water stencils (#38)
a48a681 2026-11-23 BV audio-meta: join split sound files, fitted header field sizes, child refs for 20 sound classes (#39)
71bcd6e 2026-11-23 BV BC7 BCPack: refine the preset tables against plain reference mips (#40)
32cf940 2026-11-23 BV BC6H BCPack: refit the preset slots that only BCPack-only textures use (#41)
d328a4b 2026-11-23 MQ Add MetalFX temporal upscaling to the Metal 4 renderer (#42)
cb38fb9 2026-11-23 BV Vehicles: read the 4 x u16 joint stream; rim stand-in; spin only the wheels (#44)
2e60c27 2026-11-23 BV Add timecycle/fog export (sixng timecycle) (#45)
0426996 2026-11-23 BV Ped clothing colours from the wardrobe tint palettes (#46)
5396fce 2026-11-23 BV Vehicles: wheel size from the rim radius; model info joined by handlingId (#47)
4bec3df 2026-11-23 MQ Package macOS app with Game Mode Info.plist (#43)
d986342 2026-11-23 BV Fix garbage prop-batch instances (pool refs first, separate graphics page stream) (#48)
41da1f9 2026-11-23 BV Vehicles: wheel size, width and position from the fragment's collision discs (#49)
5663c39 2026-11-23 Tarek Name valueHash, contentType, childIndex, States and the distance_map shader (#50)
d43d4c3 2026-11-23 BV Names: five more field names (colours, colour, bonerotation, BankName, pedtype) (#51)
5f74e1c 2026-11-23 BV Weapons: list data.rpf in the picker and assemble weapons from their parts (#53)
3e01b0f 2026-11-23 MQ Add Metal I/O texture cache to the Metal 4 renderer (#52)
fbaf9ee 2026-11-23 BV Vehicles: whole skeletons, dual wheels, rail wheels, scooter wheels, roof down (#54)
76179ed 2026-11-23 BV Convertibles: the roof animation in the export and the viewer (#55)
7670dc5 2026-11-23 BV Vehicles: all clips, bonnet and boot, brake discs, wheels by tyre size (#56)
eb55594 2026-11-23 BV Vehicles: mod parts from the vehicle data, with a Parts panel in the viewer (#57)
f1f0ec6 2026-11-23 BV Vehicles: part colours from the part sets and the material palette (#59)
dbde704 2026-11-23 BV Vehicles: body-paint groups from the configuration, rim size from the wheel fitment (#60)
0a17ad3 2026-11-23 BV Viewer: vehicle lamps (head, tail, brake, reversing, indicators) (#61)
483e075 2026-11-23 BV Vehicles: the game's display names, friendly names in the model picker (#62)
b57cdb0 2026-11-23 BV BCPack: BC7 secondary-index reset rule, BC6H joint refit (#58)
1a51bcf 2026-11-23 BV Viewer: steering (front wheels, fitted wheels and brakes, steering wheel) (#63)
b6ffda5 2026-11-23 BV Picker: readable names for vehicle parts, weapons and wheels (#64)
81a060d 2026-11-23 BV Viewer: --gpu <name> picks the graphics adapter (#65)
032dacf 2026-11-23 BV Vehicles: part colours from the result most configurations give (#66)
1c71efa 2026-11-23 BV Vehicles: the steering lock from the handling records (#67)
d7abb57 2026-11-23 BV Viewer: brake glow (#68)
8c9de05 2026-11-23 BV Vehicles: part colours per configuration, option lists followed (#71)
b669e20 2026-11-23 AD Textures: reconstruct tangent-space Z on decoded BC5 normal maps (#69)
c3de3a6 2026-11-23 AD Wardrobe: prefer tag names and ped body class / head prefix in ped-variations (#70)
f0f43c8 2026-11-23 BV Viewer: rim and tyre colours from the vehicle's part colours (#72)
18be470 2026-11-23 BV Viewer: side windows roll down (#74)
c4cfa60 2026-11-23 BV Viewer: a screenshot waits for the vehicle's wheels and data (#75)
08f1c2e 2026-11-23 BV Vehicles: metalness and palette position from the paint shader's reading (#76)
4d055d8 2026-11-23 BV Viewer: fixes from the whole-fleet check (#77)
65e1138 2026-11-23 BV Metal renderer: the body paint swap on vehicle parts (#73)
4f21726 2026-11-23 BV People: named characters wear their outfits, with hair, a body slider and heeled feet (#78)
a1ab8ca 2026-11-23 BV People: hair grown full, checked against the head, stored hair colour (#80)
74bd58b 2026-11-23 BV extract --force rewrites shared textures (#79)
e9f07ec 2026-11-23 BV Names: 231 vehicle material library field names (#82)
ca22d2a 2026-11-23 BV Vehicles: light patterns (sirens, strobes), siren lights and siren sound in the viewer (#81)
4b123c4 2026-11-23 BV CI: run once per PR push (#83)
fcb6c24 2026-11-23 BV Vehicles: lamp colours from the light archetypes (#84)
d4ecbb6 2026-11-23 BV Vehicles: palette rows from their own slice, surface blend from the material map (#85)
d8c0f19 2026-11-23 BV People: Lucia and Jason, own-parts characters, faces and layers (#86)
4abd67f 2026-11-23 BV Viewer: aircraft rotors, propellers and control surfaces (#87)
5e1398e 2026-11-23 BV People: a rig for characters, with pose controls and presets (#88)
0353566 2026-11-23 BV Viewer: retractable landing gear (#89)
3fbfbd3 2026-11-23 BV People: play the game's animation clips on characters (#90)
72b2cd4 2026-11-23 BV Names: ped wardrobe field names and property tag keys (#91)
067f6dd 2026-11-23 BV Viewer: landing gear doors close level, stray wheels follow their leg (#92)
20219d8 2026-11-23 BV Viewer: landing gear braces go up with their leg, door rods are put away (#93)
0bb3cd3 2026-11-23 BV Viewer: landing gear doors close over the bay (#94)
622f9e5 2026-11-23 BV Vehicles: truck bodies and beds from the configuration's parts (#95)
be16132 2026-11-23 BV Vehicles: no prop tint on vehicle shaders (#97)
4a8da27 2026-11-23 BV Names: vehicle surface reference fields (#98)
d27f2d8 2026-11-23 BV People: clean fabric shading, hair colour swatches, other hair styles (#100)
116d628 2026-11-23 Snorps37 Names: natural-ground surface names recovered by hash (#99)
afc30aa 2026-11-23 BV Viewer: occlusion from the vehicle material map (#101)
5ed86e5 2026-11-23 BV Viewer: fit a part's models only on mounting bones the vehicle has (#102)
480d32d 2026-11-23 BV BCPack: BC7 presets with alpha, BC6H refit with the no-wrap rule (#103)
d397de6 2026-11-23 BV Viewer: parts on a bed's bones mount on the bed (#104)
91f866a 2026-11-23 BV Viewer: the model list keeps the selected row on screen without jumping (#106)
631d1e5 2026-11-23 BV Materials: decode the layered-material libraries (clothing and weapon colours) (#107)
087f7dd 2026-11-23 BV Textures: Used by lists heads and characters for ped skin textures (#105)
9278ff7 2026-11-23 BV Viewer: the model list is in the order of the names shown (#108)
54e26da 2026-11-23 BV People: clothing colours from the garment presets (#109)
f1e976d 2026-11-23 BV Vehicles: part colours for vehicles in shared tables, no gradient map as base colour (#111)
0187d22 2026-11-23 BV Materials: weapon models, tiles, parameter names (#110)
2985152 2026-11-23 BV Viewer: a configuration's body colour is a display value, not linear (#113)
f84dfee 2026-11-23 BV Viewer: weapon parts in the colours of their material presets (#114)
e3871a5 2026-11-23 BV Materials: tint-mask layers of a preset's sub-materials (#115)
0b44e75 2026-11-23 BV People: per-part clothing colours and preset colours in outfit exports (#116)
8d2183b 2026-11-23 BV Vehicles: a part element names its model by asset id; pickup beds from their part (#118)
eadab44 2026-11-23 BV Export: weapon parts in their preset colours (#119)
3bfa416 2026-11-23 BV Viewer: normal maps in a tangent frame built from the UV set they are sampled with (#120)
8315a34 2026-11-23 BV People: fabric baked into garment textures (weave, colour gradient, tint-mask layers) (#122)
9e672a3 2026-11-23 BV Viewer: weapon finishes per part type (#121)
ab122bc 2026-11-23 BV Version 0.8.0 (#123)
0c8c3f0 2026-11-23 BV Vehicles: mod part models take their vehicle's part colours (#124)
9fa7c0a 2026-11-23 BV Vehicles: the body paint's finish from the data (#127)
7338d0e 2026-11-23 BV BCPack: BC3, BC4, BC5 and BC7 presets made consistent with three more exact rules (#125)
9ce31d3 2026-11-23 BV BC1 BCPack: replace nine preset window slots (#126)
daf3867 2026-11-24 BV Viewer: fabric weave per pixel for a garment's main sub-material (#128)
24124ff 2026-11-24 BV Weapons: metalness and roughness per sub-material (#129)
1d046ec 2026-11-24 BV Weapons: part type names, finishes for magazines and sights, colour labels (#130)
9b38e3c 2026-11-24 BV Vehicles: stock alternatives of the chassis and other parts (switches) (#131)
23ff140 2026-11-24 BV BC6H BCPack: preset refit with lighting chunks kept for the rules only (#134)
560901b 2026-11-24 MQ Viewer: add MetalFX frame generation (#132)
75ee571 2026-11-24 BV People: more characters load whole and dressed (#135)
4c2fe2a 2026-11-24 BV Vehicles: shared truck beds and other library parts take the colours of the vehicle they are on (#136)
1f4318f 2026-11-24 BV Vehicles: part colours per pixel from the material palette (#137)
68d2267 2026-11-24 BV Vehicles: the second layer of part surfaces (wear), with a wear control in the viewer (#138)
da2223b 2026-11-24 BV Weapons: finishes from the weapon, part set and mod records; default finish (#139)
c44cc61 2026-11-24 BV Export: readable materials.json beside each model (#140)
2d87cee 2026-11-24 BV Heads: export the wrinkle region masks and the face region mask (#141)
50b2436 2026-11-24 BV Export: materials.json keeps every texture when a slot key repeats (#143)
002d90f 2026-11-24 BV People: hair stays under a hat (#144)
eac3929 2026-11-24 BV CLI: sixng vehicle, a whole vehicle with the parts of a configuration as one GLB (#142)
d15dbef 2026-11-24 BV Decode the whole-world height map (sixng heightmap) (#145)
03c814c 2026-11-24 BV Shaders: declared resources and constants of every program and effect (shader-bindings) (#147)
00ad6ec 2026-11-24 BV Viewer: Randomize button for a vehicle's parts (#148)
67a8ca4 2026-11-24 BV Textures: decode R32_FLOAT, R16G16_FLOAT and R10G10B10A2; store chunk table past 4 GiB (#149)
fb0786b 2026-11-24 BV People: clips move over the ground (#150)
d2bd2c1 2026-11-24 BV Export: materials.json lists a texture's slices (#151)
6969454 2026-11-24 BV RBF binary XML to XML: sixng rbf command and viewer preview (#152)
4406256 2026-11-24 BV BC1 BCPack: preset window checked per texture, 20 slots replaced (#154)
88bb24c 2026-11-24 BV Shaders: pipeline cache joined to the programs (shader-bindings --cache) (#155)
8c9171e 2026-11-24 BV Viewer: bonnet and boot struts follow the panel (#153)
14d4279 2026-11-24 BV Vehicles: a widebody kit moves the wheels out (#157)
b7fbe3a 2026-11-24 BV Index: keep a resource whose system page continues on a resource boundary in one entry (index version 6) (#156)
c8037b3 2026-11-24 BV Shaders: techniques and passes of every effect (shader-bindings) (#158)
2a7e7fa 2026-11-24 BV Map 3D: fill far-field terrain without collision from the height map (#159)
44da5e6 2026-11-24 BV People: 25 more characters open with shoes and clothes (#160)
e45299b 2026-11-24 BV Four-layer ground: layer order, weights and data maps confirmed from the shader code (#161)
2cf005b 2026-11-24 BV Materials: read constant values from their buffer at their byte offset (#163)
15f4532 2026-11-24 BV People: swimwear counts as clothes; missing garments are listed (#165)
b2066ad 2026-11-24 BV Library: recents and favourites show the item's name; card text stays inside the card (#167)
85c7dfa 2026-11-24 BV Export: the second vertex colour stream, which holds a vehicle's sticker zones (#162)
0f6984c 2026-11-24 BV Vehicles: wheel fitment records of parts without a model, and in sixng vehicle (#169)
d0913eb 2026-11-24 BV Vehicles: liveries as data (brands, stickers, textures, layouts) (#168)
938c75f 2026-11-24 BV Four-layer ground: layer UVs from the material's constants (128 per unit, not 64) (#171)
c510697 2026-11-24 BV Outfit export: clips carry their movement over the ground (#166)
6c6c7fe 2026-11-24 BV Grooms: export the full strand set, and document how it is grown (#173)
18bc31c 2026-11-24 BV BC6H BCPack: endpoint presets refitted on edge continuity (#170)
eff6945 2026-11-24 Tarek Names: import a name dictionary as a last-resort source (#175)
83c2a60 2026-11-24 BV Stickers: texture export by id (sixng stickers) and the sticker zone in the viewer's vertices (#172)
a2b308c 2026-11-24 BV sixng vehicle: rim size, brakes, and picking single parts (#174)
58b355a 2026-11-24 BV Vehicles: boats never get wheels (#177)
972bf89 2026-11-24 BV Viewer: draw stickers placed on a vehicle (custom liveries) (#176)
ca6818a 2026-11-24 BV Viewer: eyes follow the head when it is posed (#181)
e0003b6 2026-11-24 BV CI: skip macOS on merges to master (#182)
929b656 2026-11-24 BV Notices: thank the gta6-namehash contributors (#179)
65a9724 2026-11-24 BV Materials: each constant's type from the shader tables; integers written as integers (#183)
6a7e082 2026-11-24 BV Liveries: a sticker's colours are colorRed, colorGreen, colorBlue (#180)
1051ea5 2026-11-24 BV Vehicles: a body kit fitted by hand takes the paint (#184)
af75bda 2026-11-24 BV Viewer: export button for hair and fur, with full strands (#185)
49eca9f 2026-11-24 BV Four-layer ground: where each material's layer weights come from (vertex colours or lookup texture) (#187)
7efd16a 2026-11-24 BV Viewer: pose controls for shoulders, twists, wrists, ankles and eyes (#188)
d2ff12a 2026-11-24 BV Four-layer ground: height-based blend, detail array and data map channels from the shader code (#189)
13de593 2026-11-24 BV Viewer: Liveries panel to place stickers on a vehicle by hand (#186)
f0c2bb6 2026-11-24 BV Viewer: sliding doors slide (#190)
73eb023 2026-11-24 BV Clothing materials: describe the stitch layer (#193)
f040646 2026-11-24 BV Viewer: animation clips on animals, with the fur following (#192)
6ec24cf 2026-11-24 BV World sector grids in levels.rpf: decoder and sector-grid command (#194)
e34f65d 2026-11-24 BV Four-layer ground: tint formula per material (#195)
2f02f1b 2026-11-24 BV Viewer: tick boxes to move doors, windows, bonnet and boot one by one (#191)
9ea455f 2026-11-24 BV Names: mount paths, audio.rpf names and audio bank names (#196)
8f3e0a7 2026-11-24 BV Four-layer ground: describe all 18 variants (#197)
5f7a635 2026-11-24 BV Tyres stay black: leave the tyre colour slot unapplied (#198)
6572044 2026-11-24 BV Characters: list every ped model of the outfit list, named where a name is recovered (#199)
f73d2f7 2026-11-24 BV CLI: text --find searches every text string (#200)
af7b5e9 2026-11-24 BV Clothing stitches: name the real stitch atlas and correct the description (#202)
088a245 2026-11-24 BV Viewer: characters' hair takes the colours of its hair preset (#203)
bc1e967 2026-11-24 BV Cutscenes: cast, props, cameras and clips of every story scene (SIX-100) (#201)
0771f56 2026-11-24 BV Ped model names: wider search with a control, uncertain flag (#204)
e90b421 2026-11-24 BV Truck side guards: one length and height, picked by the part's switch (#209)
9d08383 2026-11-24 BV Speed: --bench with checksums, BC decode in bands (SIX-99) (#206)
eb4373c 2026-11-24 BV BC6H BCPack: 17 more endpoint presets refitted on edge continuity (#205)
89357cb 2026-11-24 BV Ped models: the name the game shows, beside the model name (#207)
fcaceef 2026-11-24 BV Names: bank list containers, stream names and more audio bank names (#208)
98b28c4 2026-11-24 BV Sector grids: per-file cell sizes and kinds, map groups switched by world-state rules (#210)
e23a836 2026-11-24 BV ttpf: --float writes float textures' unclamped values (#211)
d3aeb9a 2026-11-24 BV Hair: grow strands the way the game's generation program does (#212)
002aa3e 2026-11-24 BV Cutscenes: join the streamed section dictionaries into scene clips (#213)
00041b6 2026-11-24 BV Audio names: bank-list groups give each bank its pack; looser stream-name rule (#214)
bef6b80 2026-11-24 BV Vehicles: a part model listed twice on one bone is fitted once (#215)
fb3c3fe 2026-11-24 BV Cutscenes: join the last missing section windows (#216)
0049900 2026-11-24 BV Viewer: sector grid toggle in Map 2D and Map 3D (#218)
e46a7f8 2026-11-24 BV Vehicles: brakes per axle; drum brakes come from the brake slots (#220)
2936076 2026-11-24 BV Audio stream names: read the names banks store; only metadata-backed stems from the search (#219)
d7e9bdc 2026-11-24 BV Scanner: join stored (uncompressed) 64 KiB chunks into their chunked entry (#221)
d481bad 2026-11-24 BV Viewer: Scenes tab to browse cutscenes, and Open in scenes for each character (#222)
0b14859 2026-11-24 BV Vehicles: number plates follow their switches (#223)
```

## L. Limitacoes desta analise

**Acesso a rede.** Nao foi consultado o GitHub: as referencias `#N` foram lidas apenas dos assuntos de commit (o repositorio squash-aplica cada PR com `(#N)` no assunto). Descricao de issue, discussao de PR, comentario de revisor, CI verde/vermelho: **NAO VERIFICADO**. Dois merges reais existem em master (`af96038`, `06c5cf6`); os demais `(#N)` sao aplicacoes diretas, nao merge commits — o que significa que a discussao do PR nao chegou ao historico local.

**Execucao.** Nada foi compilado nem executado: sem `cargo build`, `cargo test`, sem a instalacao do jogo. Portanto (a) as afirmacoes sobre a semantica de `&&str` em D-20 sao raciocinio de tipos, nao compilacao; (b) todas as metricas citadas (dB, contagens de violacao, cobertura) sao **citacoes das mensagens de commit e do README**, nao remedicoes; (c) nao foi verificado se HEAD compila. A arvore de trabalho estava limpa (`git status --porcelain` vazio), entao o que foi lido corresponde a `0b14859`.

**Commits examinados.** 45 blocos do pacote de evidencias, dos quais os diffs vinham truncados em 90 linhas; para os defeitos catalogados em D busquei o diff completo com `git show`. Os outros 258 commits de master foram lidos apenas por assunto no indice — nao afirmo ausencia de defeito neles. Os 332 commits de branch foram triados em bloco (comparacao de `git ls-tree` de todas as 191 refs contra HEAD, `git cherry`, e comparacao de blob por commit), nao um a um.

**Duplicacao de commits.** Cinco pares branch/master foram confirmados com blob identico no arquivo principal; **nao** verifiquei os 332 individualmente, entao "todo o resto das branches e equivalente a master" e conclusao por comparacao de arvores (arquivos existentes), nao por comparacao de conteudo de cada hunk. Um hunk de branch que tenha sido descartado na aplicacao do `(#N)` nao seria detectado por esse metodo.

**Cadeias de causa.** Onde a mensagem de commit nao datava a introducao, usei `git log -S<simbolo>` para achar o commit mais antigo que contem o defeito. Isso encontra o commit que **introduziu aquele texto**, nao necessariamente o momento em que o comportamento passou a ser errado: em `6b68f75` (importacao em bloco) essa distincao se perde. As classificacoes "introduzido em `6b68f75`" devem ser lidas nesse sentido.

**Razoes de defeito declaradas pelo autor.** As frases "not decoded", "inference", "not the game's verified rule", "stand-in" foram tratadas como evidencia de limitacao conhecida (secao G) e nao como prova de que o comportamento esta errado. Nao ha como verificar sem o jogo.

**Metodo de squash.** Como cada PR entra como um unico commit `(#N)`, a granularidade de "tentativa de correcao" e a do branch (ex.: `2a70bca`, `18bc31c`), nao a do commit de master — que e o mesmo conteudo com outra arvore. Nas cadeias usei o commit de branch quando ele carrega a mensagem com as medicoes (varios dos `(#N)` correspondentes tem corpo vazio) e o hash de master quando a pergunta era "esta em HEAD".

**Nao verificado especificamente.** (a) A afirmacao do README de que os bones de exportacoes de veiculo anteriores a 2026-11-20 estao errados ("u16 joints for skeletons over 255 bones, fixed 2026-11-20") — nao achei o commit que faz essa mudanca em `gltf.rs`/`drawable.rs` com as buscas que tentei, e o commit inicial e do mesmo dia, o que torna a frase ambigua; fica como **NAO VERIFICADO** e fora do catalogo. (b) `G-04` (corrida em modelos de freio). (c) Se o exportador emite `TEXCOORD_0` para o normal map em todos os casos (G-02 e PROVAVEL, nao CONFIRMADA). (d) Se `sample_mip`/fallbacks (`6282863`) cobrem todos os casos de textura ausente.

**Escopo.** Nao avaliei qualidade de codigo, cobertura de teste real, desempenho, nem seguranca das rotinas do jogo chamadas na branch `rpf9-toc`. A analise e historica e comportamental: o que mudou, por que, e o que a mudanca resolveu.
