# GTA6-Asset-Info - analise de DeepSeek V4.1 Flash

## A. Identificacao

| Item | Valor | Evidencia |
|---|---|---|
| Nome | `GTA6-Asset-Info` | `git remote -v` |
| URL | https://github.com/walkthestars/GTA6-Asset-Info | `git remote -v`; HTML publico, 2026-11-25 |
| Fork ou original | **Original** (`"isFork":false,"isMirror":false`) | HTML publico do repositorio, 2026-11-25 |
| Licenca | GPL-3.0 (`LICENSE`, 675 linhas; `"spdxId":"GPL-3.0"`) | HTML publico; `git show --numstat dd592ba` |
| Linguagem | nenhuma — **34 arquivos rastreados, zero codigo** (`33 .md` + `LICENSE`) | `git ls-files` |
| Branch analisada | `main` (unica; `origin/HEAD -> origin/main`) | `git branch -a -v` |
| Primeiro commit ("commit 0") | `dd592ba` — 2026-11-20 14:47:50 -0500 | `git log --reverse --format="%H\|%ad\|%an\|%s" --date=iso` |
| Ultimo commit | `bad5bc2` — 2026-11-20 15:51:51 -0500 | `git log -1` |
| Periodo coberto | 14:47:50 -> 15:51:51 (-05:00) = **64 min 01 s** | diferenca entre os dois carimbos |
| Total de commits | **5** | `git rev-list --count HEAD` |
| Commits examinados em profundidade | **5 de 5 (100 %)** — `git show` integral de cada um, mais leitura completa dos 34 arquivos | `git show --numstat`; leitura dos 34 arquivos |
| Merges / tags / branches de trabalho | 0 / 0 / 0 | `git log --merges` vazio; `git tag` vazio; `git for-each-ref` so `refs/heads/main` + `origin/main` |
| Modelo responsavel | DeepSeek V4.1 Flash (opencode-go) | — |
| Data da analise | 2026-11-25 | — |

**Autoria e cadeia.** Quatro commits de `Brandon <foiler25@gmail.com>` e um de `walkthestars <64241+walkthestars@users.noreply.github.com>`. Autor e committer identicos em todos; nenhum rebase. Pais (`git rev-list --parents HEAD`): `dd592ba -> 0a52a67 -> 4b3beb1 -> 14b4bff -> bad5bc2`, linear. `.git/logs/HEAD` tem uma unica entrada, `clone: from https://github.com/walkthestars/GTA6-Asset-Info.git` por `lucaslamdev`; `HEAD` = `origin/main` = `bad5bc2`, arvore limpa (`git status --short` vazio). O objeto analisado e o clone, nao um checkout modificado.

**Metadados do GitHub (verificados hoje, 2026-11-25, via fetch do HTML publico).** `createdAt: 2026-11-20T19:53:24Z`; `stargazerCount: 10`, `forksCount: 3`, `watcherCount: 0`; licenca GPL-3.0; nao e fork. A **descricao** do repositorio e literalmente:

> Format notes and tool guide for SixNG, an asset extractor for GTA VI (Xbox Series): RPF9, textures, BCPack, models, map data

A aba **Issues nao aparece** no HTML publico (indicio forte de issues desabilitadas; a causa exata — `has_issues:false` — nao pode ser relida nesta sessao porque `api.github.com` devolveu HTTP 403 por rate limit). Contagem de issues e de PRs: **NAO VERIFICADA** aqui.

**Natureza do objeto.** Nao ha codigo, build, teste, CI, CHANGELOG, `CONTRIBUTING.md`, `.github/`, `TODO` ou `ROADMAP`. O que existe e um corpus de engenharia reversa escrita — a especificacao em Markdown dos formatos de asset de GTA VI (build Xbox Series) e o guia de uma ferramenta extratora (`SixNG`) que **nao esta no repositorio**. Por isso todo o catalogo D e de defeitos **documentais**: contradicao, aritmetica que nao fecha, hash que nao reproduz, estado obsoleto, link/instrucao impossivel. Nenhum item e bug de execucao.

---

## B. Resumo tecnico

**Proposito declarado.** `README.md:1-11`: "Technical notes on the asset formats of GTA VI (Xbox Series build)... This repository holds documentation only: no game files, no extracted assets, no keys." Cobre RPF9, paginas de recurso RAGE 9, dicionarios e stores de textura, BCPack, drawables/fragments/veiculos, meta resources `0x7a`, placements, interiores (MLO), colisao, navmesh, luzes, road graph, audio AWC, PSO e shader cache.

**Arquitetura documental — tres arvores, dois vocabularios de confianca, tres datas de ultimo toque.**

| Arvore | Arquivos | Linhas | Identidade | Vocabulario | Ultimo toque |
|---|---|---|---|---|---|
| `formats/` | 18 (indice + 17 paginas) | 1.265 | `# SixNG format specification` (`formats/README.md:1`) | `confirmed / inferred / unknown` | `dd592ba` (14:47) |
| `tool/` | 6 | 568 | `# SixNG tool guide` (`tool/README.md:1`) | — | `dd592ba` (14:47) |
| `structure/` | 7 | 461 | `# Structure notes — index` | `OBSERVED / INFERRED / UNKNOWN` + `DISPUTED` | `bad5bc2` (15:51) |
| raiz | `README.md` 75 + `LEGAL.md` 53 | 128 | "format notes", sem marca | — | `4b3beb1` / `14b4bff` (14:53) |

Total: **2.422 linhas de Markdown** + 675 de licenca. O commit 0 sozinho e `27 files changed, 2.658 insertions(+)` — o corpus nasce pronto, sem estado incremental. `git log -1 --format=%h -- <arquivo>` mostra que **nenhum arquivo de `formats/` ou `tool/` foi tocado depois de `dd592ba`** (tabela completa na secao K). O ultimo toque por arquivo e a ferramenta mais reveladora deste historico.

**Verificacoes independentes executadas nesta analise.** Tudo o que segue foi recalculado contra o texto: joaat Jenkins one-at-a-time implementado localmente (validado em `hauler -> 0x5A82F9AE`), somas e percentuais refeitos, 154 links Markdown relativos resolvidos com verificacao de ancora no estilo GitHub, e auditoria de vocabulario/marca por `git grep`.

| Alegacao no corpus | Teste | Resultado |
|---|---|---|
| `joaat("hauler") = 0x5A82F9AE` (`structure/05:50-51`) | reimplementacao | **exato** |
| `Position = 0c928991`, `Flags = 4b621302` (`formats/08:31-33`) | joaat case-sensitive | **exatos** (minusculas: `01166164` / `67d26872`, logo "case-sensitive" e verdadeiro) |
| `intensity`, `falloff`, `falloffExponent`, `coneInnerAngle`, `coneOuterAngle` (`formats/15:23-26`) | joaat | **5 de 5 exatos** |
| `lodLevel`, `parentIndex`, `numChildren`, `lodDist`, `childLodDist` (`formats/09:29-33`) | joaat | **5 de 5 exatos** |
| `priorityLevel = 84799847` (`formats/09:34`) | joaat + 2 grafias alternativas | **nao reproduz** (D-20) |
| `lightType = 1cbbbe30`, `timeFlags = 8609d52c` (`formats/15:21-22`, hashes sem nome na pagina) | joaat de nomes de membro de `CLightAttr` | **exatos** — a regra da pagina se confirma em hashes que ela nao nomeia |
| 10 hashes de struct do corpus (`ce501483`, `82d6fc83`, `d3593fa6`, `27922c43`, `f54b227b`, `09cc28be`, `10506455`, `0b1811f1`, `995072ca`, `d6a799f9`) | joaat do nome de classe GTA V citado pela propria pagina | **10 de 10 exatos** (`CEntityDef`, `CBaseArchetypeDef`, `CMapData`, `CExtensionDefLightEffect`, `CLightAttrDef`, …) — refuta `formats/08:34` (D-24) |
| `774,847` luzes = `447,756 + 326,183 + 907` (`formats/15:38-39`) | soma | **774.846** — falta 1 (D-17) |
| "34 of 51 named" node flags (`formats/16:25`) | contagem da tabela `16:27-44` | **32 nomes** em 16 linhas x 2 colunas (D-16) |
| "344 shaders" (`formats/12:8`) | soma da coluna `Shaders` `12:20-27` | **389** (193+44+59+90+2+1) (D-18) |
| `352,461` texturas = 298.332+6.966+14.441+29.310+3.412 (`formats/05:88-96`) | soma | **352.461 exato**; 84,64 / 1,98 / 4,10 / 8,32 / 0,97 % batem |
| `92,925` archetipos, `10.982` links, `25.676` ambiguos (`formats/09:72-77`) | soma | 10.982+25.676 = 36.658; **56.267 sem categoria** (D-19) |
| `305 + 899 + 463 = 1.667`; `1.667 / 1.704 = 97,8 %` (`formats/10:19-24`) | soma e divisao | **exatos** (97,83 %) |
| `2.089` com parent hash, `1.304` resolvem, "785" em aberto (`formats/09:41-42,87`) | subtracao | **785 exato** |
| `round16(10 x 2.078) + 4 = 20.784` (`structure/01:32-33`) | aritmetica | **exato** (20.780 -> 20.784) |
| `0x28 + 30.683 x 8 = 0x3BF00` (`structure/04:45`) | aritmetica | **exato** (245.504) |
| `147.252.026.112 bytes = 137,1 GiB` (`structure/05:56`) | /1024^3 | **137,14 GiB** |
| `163.840 bytes = 40.960 samples x 4` (`structure/03:72-73`) | divisao | **exato** |
| `4,47 / 13,4 / 17,9 / 22,35 m/s = 10 / 30 / 40 / 50 mph` (`formats/16:17-18`) | conversao | **10,0 / 30,0 / 40,0 / 50,0** |
| "289 slot values resolve: 277 + 41 + 10 + 5" (`structure/04:59-61`) | soma | **333**, nao 289 (D-23) |
| 154 links relativos em 33 `.md` | resolucao de caminho + slug de heading | **0 problemas** (ancora de BCPack inclusive) |
| `22,7 + 68,1 + 7,4 = 98,3 %` de area de colisao (`formats/13:68-69`) | soma | **98,2** — compativel com arredondamento, no limite (D-26) |
| Instancias de shader `1,09 M + 32 K + 596 K + 87 K + 14,6 K` vs "1.83 M" (`formats/12:8,22-27`) | soma | **1,8196 M** (D-18) |

**Onde esta a solidez.** Onde o corpus mediu, mediu bem: 15 dos 16 hashes nomeados reproduzem joaat exatamente (e dois hashes nao nomeados tambem), todos os percentuais de cobertura de textura fecham, o grafo de objetos PSO fecha byte a byte (`0x3BF00`), o volume dos stores fecha em 137,1 GiB, as conversoes de unidade fecham, e a aritmetica de interiores fecha exatamente.

**Onde esta a erosao.** Tres causas independentes e cumulativas: (a) o rebrand de `0a52a67`/`14b4bff` parou nas duas paginas de entrada — `formats/` e `tool/` nunca foram tocados e continuam dizendo "SixNG" em 81 ocorrencias/24 arquivos; (b) `structure/` entrou 58 minutos depois como bloco fechado, sem link de entrada e sem reconciliacao, criando contradicoes diretas em campos identicos; (c) a reescrita do README apagou as unicas metricas de cobertura de arquivo inteiro que existiam. O resultado e um repositorio em que a informacao mais confiavel e a menos encontrivel.

---

## C. Linha do tempo

### Fase 0 — pre-historia (inobservavel)

O commit 0 ja nasce maduro: 27 arquivos, 2.658 linhas, numeros medidos "as of 2026-11-20", coluna de confianca por campo e `Open questions` por pagina. Nao existe no git nenhum estado anterior, nenhuma semente incremental, nenhum "initial dump" separado. `git log --follow` em qualquer arquivo de `formats/` devolve apenas `dd592ba`: o trabalho real (extracao, medicao, decodificacao) ocorreu **fora** do repositorio. **NAO VERIFICADO:** a existencia, o historico e o codigo do projeto `SixNG` citado em `tool/`; nenhum dos 33 `.md` contem uma URL (`git grep -i "http" -- "*.md"` -> exit 1).

### Fase 1 — semeadura (`dd592ba`, 14:47:50)

`27 files changed, 2.658 insertions(+)`: `LEGAL.md` (54), `LICENSE` (675), `README.md` (96), `formats/` (18 arquivos, 1.265 linhas), `tool/` (6 arquivos, 568 linhas). O README desta fase chama o repositorio de **"# SixNG"**: descreve a ferramenta ("asset extractor for GTA VI on Xbox Series", Rust, CLI + GUI), declara o objetivo (mods para BeamNG.drive, Assetto Corsa com CSP, e um jogo proprio em Rust/Bevy) e traz a **unica tabela de cobertura numerica do projeto**: `levels.rpf` com 523.552 entradas / 98,3 % dos bytes / 227.766 nomes recuperados; 2.249.490 entidades em 30.785 map files (95 % resolvidas); 906.922 batches (24,9 M prop + 219,5 M grass); 27.644 tiles de navmesh; 774.847 luzes; 63.808 nos / 145.518 links / 1.546 juncoes.

Ja neste commit existe uma tensao interna que sobrevive ate o HEAD: o README afirma "SixNG is an asset extractor… It is written in Rust" e a licenca "covers SixNG's code and these docs", enquanto `tool/01-install.md:9` (arquivo identico desde entao) diz "**This repository holds the documentation only.** Build from the SixNG source:" seguido de `git clone <SixNG source repository url>`. O leitor do commit 0 recebe, ao mesmo tempo, a ferramenta como sujeito e a instrucao de busca-la em outro lugar.

### Fase 2 — rebrand em tres tempos (`0a52a67`, `4b3beb1`, `14b4bff`; 14:51:47 -> 14:53:24, 97 s)

- `0a52a67` (`README.md` 72+/93−, reescrita, nao patch): o repositorio deixa de ser "a ferramenta SixNG" e passa a ser "notas de formato". A tabela "What it extracts" vira "State of knowledge" (colunas Understood / Not yet); a tabela de numeros de cobertura desaparece; `glb(x, y, z) = rage(x, z, −y)` vira `gltf(x, y, z) = ...`; a frase "34 of 51 node flags" e copiada para dentro do novo README (`README.md:44`); e o link de BCPack nasce apontando para `#how-to-resume`, ancora que nao existe.
- `4b3beb1` (16 s depois, 1+/1−): corrige exatamente esse link, `#how-to-resume` -> `#64-how-to-resume`. E a unica correcao "de bug" do historico.
- `14b4bff` (`LEGAL.md` 30+/31−): reescreve o sujeito juridico de "SixNG" para "these notes", preservando as quatro proibicoes (arquivos de jogo, assets extraidos, chaves, material proprietario/NDA) e a nao-afiliacao.

**Estado intermediario que ninguem reconciliou:** ao fim da fase 2, README e LEGAL dizem "notas, nao ferramenta", mas `formats/README.md:1` ainda e `# SixNG format specification`, as 17 paginas de formato ainda se declaram na linha 3 `*Part of the [SixNG format specification](README.md)...*`, `tool/README.md:1` ainda e `# SixNG tool guide` e `tool/01-install.md:3` repete. Nenhum desses 24 arquivos foi tocado depois de `dd592ba`, e a descricao do GitHub — que so vive fora do git — continua dizendo "tool guide for SixNG" (verificado hoje).

### Fase 3 — camada `structure/` (`bad5bc2`, 15:51:51, 58 min 27 s depois)

`7 files changed, 461 insertions(+)`, zero delecoes, zero modificacoes (`git show --numstat bad5bc2`: 28/0, 82/0, 61/0, 79/0, 70/0, 63/0, 78/0). Autor: `walkthestars`, o dono do remoto, **nao** o autor dos quatro commits anteriores. Os sete arquivos entram de uma vez, com vocabulario proprio (`OBSERVED/INFERRED/UNKNOWN/DISPUTED`), metodologia mais dura ("disagreements are recorded rather than resolved by preference"; "A claim that does not close arithmetically is a hypothesis, and any conclusion drawn from it is void until it does", `structure/00-INDEX.md:26-28`) e conteudo **mais avancado** que o de `formats/`: audio ADAT/Opus mapeado ate o framing, cache de shader RDNA2 identificado por marcadores de instrucao, grafo de objetos PSO contado e fechado, lei do trailer do RPF9, variantes de header por `0x2C`, lei de tamanho de descritor de textura.

Nada em `README.md`, `formats/` ou `tool/` foi ajustado para absorver ou linkar essa camada: `git grep -n "structure/" -- "*.md"` -> exit 1, vazio. A arvore entra **orfa na navegacao** e sem data (nenhum arquivo de `structure/` contem "2026-11-20": `git grep -c "2026-11-20" -- "structure/*.md"` -> vazio). Isto e, a camada mais nova e a unica sem carimbo de data, enquanto a mais antiga se declara datada 33 vezes em 25 arquivos.

### Fase 4 — silencio (15:51:51 -> 2026-11-25)

Nenhum commit, nenhuma tag, nenhuma branch. Cinco commits em 64 minutos e nada depois — 5 dias de congelamento. O corpus nao teve revisao externa registrada dentro do git; qualquer discussao teria de ocorrer fora dele.

---

## D. Catalogo de defeitos

> Todos os itens abaixo sao **documentais**. "Introduzido em" refere-se ao commit que trouxe o texto ao repositorio. Onde nao ha correcao, escrevo **SEM CORRECAO**. Onde a causa nao pode ser decidida pelo texto, digo explicitamente. Nenhum defeito aqui foi "inventado por grep": cada um foi confirmado por leitura do trecho, recalculo ou reimplementacao.

- **D-01 — O rebrand parou nas paginas de entrada: o repositorio tem duas identidades**
  - Descricao/sintomas: os assuntos de `0a52a67` e `14b4bff` dizem que o repositorio deixa de se apresentar como a ferramenta SixNG, mas o HEAD tem **81 ocorrencias de "SixNG" em 24 arquivos**. Parte e uso legitimo (`sixng` como nome de comando: 38 ocorrencias em `tool/02-cli.md`). O defeito e a **identidade**: `formats/README.md:1` = `# SixNG format specification`; as 17 paginas de formato carregam na linha 3 `*Part of the [SixNG format specification](README.md)...*`; `tool/README.md:1` = `# SixNG tool guide` e `tool/01-install.md:3` repete.
  - Causa raiz: rebrand feito arquivo por arquivo a partir da entrada, sem inventario previo (`git grep -i sixng`) e sem tocar `formats/` nem `tool/`. A prova e o ultimo toque: 24 arquivos com `dd592ba`, nenhum com `0a52a67`.
  - Introduzido em: `dd592ba` (identidade original) + `0a52a67`/`14b4bff` (rebrand parcial).
  - Corrigido em: **SEM CORRECAO**.
  - Arquivos: `formats/README.md`, `formats/01-rpf9.md`…`17-other.md`, `tool/README.md`, `tool/01-install.md`.
  - Evidencia: `git grep -i -c "sixng" -- "*.md"` -> 24 arquivos; `git grep -i -o "sixng" -- "*.md" | Measure-Object -Line` -> 81; `git log -1 --format=%h -- formats/README.md` -> `dd592ba`.
  - Estado atual: aberto. O leitor recebe "documentation only, no tool" na porta da frente e "SixNG format specification / SixNG tool guide" em todos os 24 arquivos internos.
  - Confianca: **CONFIRMADA**.

- **D-02 — `structure/` e uma arvore orfa: zero links de entrada**
  - Descricao/sintomas: `bad5bc2` adiciona 7 paginas (461 linhas) que nenhuma outra pagina referencia. `README.md` lista apenas `formats/` e `tool/` (`README.md:13-23`, `67-70`); `formats/README.md` idem. O diretorio mais rigoroso do repositorio e invisivel para quem entra pelo caminho oficial.
  - Causa raiz: commit puramente aditivo (7 arquivos, 461 insercoes, 0 modificacoes). Os indices existentes nao foram editados; nao houve revisao cruzada entre autores.
  - Introduzido em: `bad5bc2`.
  - Corrigido em: **SEM CORRECAO**.
  - Arquivos: `structure/00-INDEX.md`…`06-models-shaders.md`, `README.md`, `formats/README.md`.
  - Evidencia: `git show --numstat bad5bc2` (so adicoes); `git grep -n "structure/" -- "*.md"` -> exit 1; `git grep -n "00-INDEX" -- "*.md"` -> exit 1.
  - Estado atual: aberto, e e a causa de metade dos defeitos abaixo: audio decodificado, shader cache identificado e a lei do trailer continuam sem porta de entrada.
  - Confianca: **CONFIRMADA**.

- **D-03 — O rebrand nunca chegou aos metadados do repositorio**
  - Descricao/sintomas: `README.md` diz "documentation only"; a descricao publica do GitHub diz "**Format notes and tool guide for SixNG, an asset extractor for GTA VI (Xbox Series)**". O primeiro paragrafo que qualquer visitante le anuncia a ferramenta que o README nega ser o sujeito do repositorio.
  - Causa raiz: o rebrand foi tratado como edicao de arquivos do git; o campo `description` vive fora do repositorio e nao tem como ser atualizado por commit, e nao havia checklist pos-rebrand.
  - Introduzido em: `dd592ba` (descricao criada junto com o repo, `createdAt 2026-11-20T19:53:24Z`) + `0a52a67` (divergencia criada).
  - Corrigido em: **SEM CORRECAO** (estado verificado hoje).
  - Arquivos: nenhum do repositorio — metadado do GitHub.
  - Evidencia: fetch de `https://github.com/walkthestars/GTA6-Asset-Info` em 2026-11-25, `<meta name="description">` e `og:description` com o texto acima; `README.md:1-11` com o texto contrario.
  - Estado atual: aberto. Nao e corrigivel por commit; so por edicao de configuracao.
  - Confianca: **CONFIRMADA**.

- **D-04 — `0a52a67` apagou as unicas metricas de cobertura de arquivo inteiro**
  - Descricao/sintomas: `dd592ba:README.md` publicava `levels.rpf`: 523.552 entradas, 98,3 % dos bytes, **227.766** com nomes recuperados; **2.249.490** entidades em **30.785** map files (95 % resolvidas); **27.644** tiles de navmesh. `0a52a67` substituiu a tabela "What it extracts" por "State of knowledge" e **esses quatro numeros desapareceram do repositorio**. Nenhuma pagina de `formats/` os recebeu.
  - Causa raiz: reescrita tratada como mudanca de enquadramento (de "o que a ferramenta extrai" para "o que se entende"), mas o unico bloco com dados primarios de cobertura foi descartado junto com o enquadramento antigo.
  - Introduzido em: `0a52a67`.
  - Corrigido em: **SEM CORRECAO**.
  - Arquivos: `README.md`; ausencia em `formats/09-placements.md` (que descreve 30.785 map files sem dizer que sao 30.785), `formats/14-navmesh.md` (tiles de 128 m sem contagem), `formats/01-rpf9.md` (523.552 substituido por "about 1.1 M streams").
  - Evidencia: `git show dd592ba:README.md | Select-String "227,766|2,249,490|30,785|27,644"` -> linhas 31, 36, 40; `git grep -n --fixed-strings "227,766" -- "*.md"` -> 0 (idem `2,249,490`, `30,785`, `27,644`); o total 523.552 sobrevive **uma vez**, como pergunta aberta em `structure/01:81-82`.
  - Estado atual: aberto; perda permanente dentro do historico (recuperavel so por `git show dd592ba:README.md`). E a regressao mais custosa do repositorio.
  - Confianca: **CONFIRMADA** (ausencia). Se a remocao foi deliberada: **NAO VERIFICADA**.

- **D-05 — Instrucao de build aponta para um placeholder de URL que nunca existiu**
  - Descricao/sintomas: `tool/01-install.md:9-15`: "This repository holds the documentation only. Build from the SixNG source:" seguido de `git clone <SixNG source repository url>` / `cd <checkout>` / `cargo build --release`. O placeholder nunca e preenchido e **nenhum dos 33 `.md` contem uma URL** (`git grep -n -i "http" -- "*.md"` -> exit 1). Nao ha `Cargo.toml`, `src/`, CI ou release.
  - Causa raiz: o repositorio nasceu como "docs + guia da ferramenta" sem publicar nem linkar o codigo. O proprio commit 0 ja e incoerente: o README se chama `# SixNG` e diz que a licenca "covers SixNG's code and these docs", enquanto o guia de instalacao diz "documentation only" e manda buscar o fonte em outro lugar.
  - Introduzido em: `dd592ba`; agravado em `0a52a67` (o README passa a afirmar explicitamente "documentation only" sem preencher o placeholder).
  - Corrigido em: **SEM CORRECAO**.
  - Arquivos: `tool/01-install.md`, `README.md`, `LICENSE` (o texto do GPL nao identifica o programa).
  - Evidencia: `git ls-files` -> `LICENSE` + 33 `.md`; `git grep -i http` vazio; `git show --numstat dd592ba -- tool/01-install.md` -> 41/0 (arquivo nunca editado).
  - Estado atual: aberto. Todo o `tool/` (568 linhas de comandos, flags, schemas, tempos) descreve um binario que o leitor nao tem como obter. O efeito e que metade do repositorio e inexecutavel por construcao.
  - Confianca: **CONFIRMADA**.

- **D-06 — Header RPF9: `0x24`/`0x28` com significados incompativeis, e tres contagens para o mesmo `levels.rpf`**
  - Descricao/sintomas: `formats/01-rpf9.md:18-19` diz `0x24` = **entry count** (*inferred*) e `0x28` = `key_id`, "one of 4 values across the install" (*inferred*). `structure/01-rpf9-container.md:18-19` marca os mesmos dois campos como **DISPUTED**: `0x24` e "a count; equals file_count in some readings but not the carved entry count" e `0x28` "varies per archive". Em `0x2C` nao ha conflito de valor — `formats/01:20` grafa `01`/`02 0f 09 00` e `structure/01:20` escreve `0x00090F01`/`0x00090F02` —, mas `formats/` o declara "unknown flags" enquanto `structure/01:25-33` o divide em dois grupos, com uma lei (`round16(10 x file_count)`) que vale **so** para o grupo `0x90F02`.
  - Agravante quantitativo: o mesmo arquivo tem **tres grandezas** publicadas no HEAD — 2.078 (header, `structure/01:33,81`), 523.552 "carved entries" (`structure/01:81`) e "about 1.1 M streams" (`formats/01:53`). Nenhuma pagina declara a relacao entre elas; 523.552/2.078 ≈ 252, um numero que so faz sentido se "entry" significar fragmento de carving, nao arquivo.
  - Causa raiz: `structure/` mediu 22 arquivos e foi commitada como arvore paralela; ninguem voltou a `formats/01`. `git log -S "key_id"` -> apenas `dd592ba`.
  - Introduzido em: `dd592ba` (versao `formats`) + `bad5bc2` (versao `structure`).
  - Corrigido em: **SEM CORRECAO**.
  - Arquivos: `formats/01-rpf9.md`, `structure/01-rpf9-container.md`.
  - Evidencia: linhas citadas; `structure/01:81-82` admite a divergencia sem resolve-la.
  - Estado atual: aberto. Qual lado erra em `0x24`/`0x28` nao e decidivel pelo texto: nenhuma das paginas publica a distribuicao de `0x28` nem reconcilia as tres contagens.
  - Confianca: **CONFIRMADA** (contradicao e as tres contagens); atribuicao de culpa a `formats/01`: **PROVAVEL**.

- **D-07 — `formats/17` declara "planned / not decoded" o que `structure/03` decodificou ate o framing**
  - Descricao/sintomas: a tabela de estado (`formats/17-other.md:5-13`) diz: PSO "exported raw"; shader cache "not decoded"; audio (AWC/ADAT) "**not decoded yet (planned)**". No HEAD, `structure/03-audio-awc.md` documenta header `ADAT` com flags, tabela de chunks por bit, o tag `type:8 | size:28 | offset:28`, o enquadramento de pacotes Opus em blocos de 2.048 B, o chunk de formato com layout de 20/24/32 bytes e as excecoes PCM float32. Contradicao direta: o formato esta documentado ate o byte.
  - Causa raiz: pagina "catch-all" de estado nunca atualizada no commit que trouxe o conhecimento novo; `git log -1 --format=%h -- formats/17-other.md` -> `dd592ba`.
  - Introduzido em: `bad5bc2` (o conhecimento novo entra sem reconciliar a pagina de estado).
  - Corrigido em: **SEM CORRECAO**.
  - Arquivos: `formats/17-other.md`, `structure/03-audio-awc.md`, `tool/03-gui.md` ("Planned: … and an audio browser", `:84-85`).
  - Evidencia: `formats/17:12` vs `structure/03:25-49`; `README.md:23` encaminha o leitor para "Other | PSO, RBF, BVH4, **audio**, animation".
  - Estado atual: aberto e visivel na navegacao principal — o leitor que segue o link oficial le "planned" e nao descobre o mapa que existe.
  - Confianca: **CONFIRMADA**.

- **D-08 — Shader cache: "one compiled cache, no names; not decoded" vs layout por regiao e ISA identificada**
  - Descricao/sintomas: `formats/17:11` diz que o cache e "one compiled cache, no names; **not decoded**". `structure/06-models-shaders.md:42-71` descreve o cache `PS\x00\x09` com regioes e tamanhos (header `0x80`, PSOs graficos `count x 0x70`, compute `count x 0x18`, meta index, shader index, meta blob, raw code), contagens de exemplo (25.471 / 2.693 / 54.576 / 21.522), marcadores de instrucao (`s_endpgm` `0xBF810000`, `s_code_end`, `exp prim`, `image_bvh64_intersect_ray`) e os chunks de binding `0x41`/`0x42`/`0x43`.
  - Nuance que impede chamar de erro factual puro: as duas afirmacoes sao compativeis quanto a "nao ha nomes" (`formats/12:8`, `structure/06:75`); sao incompativeis quanto ao **estado** ("not decoded" e falso para o layout e para a identificacao de ISA). E `structure/06:70-71` ainda registra que duas contagens de estagio divergem — o que a pagina de estado nao menciona.
  - Causa raiz: ver D-02; a pagina de estado e a de detalhe evoluem em arvores separadas.
  - Introduzido em: `bad5bc2`.
  - Corrigido em: **SEM CORRECAO**.
  - Arquivos: `formats/17-other.md`, `structure/06-models-shaders.md`.
  - Evidencia: as duas linhas no HEAD.
  - Estado atual: aberto; atrito de enquadramento, nao de bytes.
  - Confianca: **CONFIRMADA** (contradicao de estado); a leitura de que o cache esta "parcialmente decodificado" e a correta: **PROVAVEL**.

- **D-09 — Familia de tiling de 4 KiB: `SW_4KB_S` ou `SW_4KB_D`?**
  - Descricao/sintomas: `formats/05-ttpf.md:69-72` mapeia modo 0 -> `SW_64KB_D` e modos 1-3 -> `SW_4KB_S`, com a ordem de bits por bloco. `structure/02-resources-textures.md:52` diz "Two tile families are observed: 4 KiB (`SW_4KB_D`) and 64 KiB". Um dos rotulos esta errado.
  - Causa raiz: provavel troca de sufixo ao reescrever o nome do modo AMD; as duas paginas nasceram em commits diferentes, sem revisao cruzada.
  - Introduzido em: `bad5bc2`.
  - Corrigido em: **SEM CORRECAO**.
  - Arquivos: `structure/02-resources-textures.md`, `formats/05-ttpf.md`, `formats/04-xbox-tiling.md`.
  - Evidencia: `formats/04:11-12` publica, para dicionarios e derivado independentemente por simulated annealing, os padroes 8 B `x0 y0 y1 x1 x2 y2 x3 y3` e 16 B `y0 y1 x0 x1 y2 x2 y3 x3` — exatamente os que `formats/05:72` atribui a `S`. Duas paginas concordam na ordem de bits e no rotulo `S`; uma discorda no rotulo.
  - Estado atual: aberto; correcao de uma palavra, mas com efeito real em quem escolhe a familia de deswizzle por nome.
  - Confianca: contradicao **CONFIRMADA**; culpa de `structure/02` **PROVAVEL** (casamento bit a bit com `formats/04`).

- **D-10 — Pergunta aberta ja respondida na arvore canonica (tile mode x header)**
  - Descricao/sintomas: `structure/02:60` lista como aberto "Which tile family applies to which descriptor (not yet correlated with a header bit)". `formats/05:59` registra `u32 d1 bits 24-27 | tile mode | confirmed`. A pergunta foi respondida na arvore que `structure/` nao le.
  - Causa raiz: pergunta herdada de rascunho anterior, commitada sem confrontar `formats/05`.
  - Introduzido em: `bad5bc2`.
  - Corrigido em: **SEM CORRECAO**.
  - Arquivos: `structure/02-resources-textures.md`, `formats/05-ttpf.md`.
  - Evidencia: as duas linhas no HEAD.
  - Estado atual: aberto; efeito pratico e reabrir investigacao ja fechada para quem entra por `structure/`.
  - Confianca: **CONFIRMADA**.

- **D-11 — Largura do campo DXGI: `bits 3–10` contra mascara de 7 bits**
  - Descricao/sintomas: `formats/05:57` declara `u32 d1 bits 3–10 | DXGI format | confirmed` — 8 bits. `structure/02:45-46` e `structure/05:47-48` leem o mesmo campo como `((u1 >> 3) & 0x7F)` — 7 bits (3–9) — e ambos afirmam validacao em **357.115** descritores sem valores desconhecidos.
  - Causa raiz: um dos lados arredondou o intervalo; como todos os enums DXGI usuais cabem em 7 bits, o bit 10 nunca foi exercitado e o teste nao distingue as duas leituras.
  - Introduzido em: `dd592ba` (`formats/05`) + `bad5bc2` (`structure/`).
  - Corrigido em: **SEM CORRECAO**.
  - Arquivos: `formats/05-ttpf.md`, `structure/02-resources-textures.md`, `structure/05-ttpf-stores.md`.
  - Evidencia: as tres linhas no HEAD; `0x7F` so pode representar bits 3–9. Contraprova interna: `formats/05:114` registra "**Format 157** (one vehicle texture slot) is unknown" — 157 > 127, logo, se esse valor veio do mesmo descritor, a leitura de 7 bits o truncaria para 29.
  - Estado atual: aberto; o impacto pratico depende de o valor 157 vir ou nao do mesmo campo. Para os enums mapeados em `formats/03:25` o maior valor e BC7 = 99, que cabe em 7 bits — e por isso o teste de 357.115 descritores nao separa as duas leituras. Uma leitura futura com o bit 10 ligado seria truncada em silencio pela mascara.
  - Confianca: contradicao **CONFIRMADA**; leitura de 8 bits (`formats/05`) **PROVAVEL** pelo argumento do valor 157; origem declarada do 157: **NAO VERIFICADA**.

- **D-12 — `PSIN` e big-endian numa arvore e little-endian na outra — e nas duas dentro do mesmo arquivo**
  - Descricao/sintomas: `formats/17:7` diz "PSO (`PSIN`; sections LSCH/LENM/LSNS/LGID/LRSC/CHKS) | **little-endian**, hashed field names…". `structure/04:5` diz "**Big-endian.** Sections are walked by a u32 length that includes its own 8-byte header". Pior: o proprio `structure/04` mistura as convencoes — a linha 5 declara big-endian e o enquadramento de `LSNS` (`:26`) escreve `u32 LE 1 | 01 01 00 00 | u32 LE tail_length`.
  - Causa raiz: duas medicoes independentes sem cross-reference; `structure/04` foi escrito depois e nao substituiu o texto antigo, e a convencao global do arquivo nao foi aplicada ao exemplo.
  - Introduzido em: `dd592ba` (`formats/17:7`) + `bad5bc2` (`structure/04:5,26`).
  - Corrigido em: **SEM CORRECAO**.
  - Arquivos: `formats/17-other.md`, `structure/04-pso-objectgraph.md`.
  - Evidencia: citacoes acima. Nota: `structure/04:18` diz que os corpos `LRSC` comecam com `0DRP` "(u32 little-endian `0x50524430`)" — o que e aritmeticamente verdadeiro para os bytes `30 44 52 50` e mostra que o autor alterna as duas leituras no mesmo texto.
  - Estado atual: aberto; um decodificador nao consegue implementar sem arbitrar.
  - Confianca: **CONFIRMADA** (conflito). A resolucao correta exigiria amostras: **NAO VERIFICADA**.

- **D-13 — "Geometria e meshlets, nao arrays de vertice" contra o mesh blob documentado**
  - Descricao/sintomas: `structure/06:5-7` afirma "Drawable geometry is stored as **meshlets, not as raw vertex arrays**. Every raw-coordinate interpretation fails arithmetic or range tests". `formats/07:31` documenta "**Vertex streams** (tightly packed arrays of V elements)" com tabela de slots `k = 0..22` (`formats/07:25`), quantizacao de posicao (`u64` -> `x/y/z` por base e step, `formats/07:48-54`) e `V = (menor offset > 0x80 − 0x80) / 8`. Meshlets aparecem so como **index buffer** (`formats/07:16,75-89`).
  - Causa raiz: provavel colisao de vocabulario/escopo — `structure/06` reage a leitura de triplas `float` cruas (que de fato falha, porque a posicao e `u64` quantizada) e generaliza para "nao existem arrays de vertice".
  - Introduzido em: `bad5bc2`.
  - Corrigido em: **SEM CORRECAO**.
  - Arquivos: `structure/06-models-shaders.md`, `formats/07-models.md`.
  - Evidencia: linhas citadas; ambas concordam no ponto verificavel — os descritores de meshlet comecam em `+0x0C` (`formats/07:83-84`; `structure/06:21-22`).
  - Estado atual: aberto; risco de o leitor descartar `formats/07` (a pagina mais completa do corpus, 188 linhas) por causa da frase absoluta.
  - Confianca: contradicao textual **CONFIRMADA**; a reconciliacao "float triples vs quantiser" e **HIPOTESE**.

- **D-14 — A descoberta de que a chave do `ttpf` e joaat nao foi propagada**
  - Descricao/sintomas: `formats/05:8` fala da "**64-bit texture hash** that drawable shaders reference" e `formats/07:182` lista "Shader, parameter and **texture name hash function**" entre as perguntas abertas. `structure/05:50-52` diz que os hashes aparecem como **joaat** simples em pelo menos um caso (`hauler` -> `0x5A82F9AE`), "so the index is name-keyed rather than path-keyed".
  - Causa raiz: `formats/05`/`07` descrevem o campo como `u64` (verdade) e concluem "funcao desconhecida" sem notar que o valor observado e um joaat de 32 bits promovido; `structure/05` fez a descoberta e nao voltou.
  - Introduzido em: `dd592ba` (afirmacao) + `bad5bc2` (descoberta nao propagada).
  - Corrigido em: **SEM CORRECAO**.
  - Arquivos: `formats/05-ttpf.md`, `formats/07-models.md`, `structure/05-ttpf-stores.md`.
  - Evidencia: joaat one-at-a-time reimplementado localmente -> `hauler` = `0x5A82F9AE`, **exato**; o mesmo metodo reproduz `Position` = `0x0C928991`, `Flags` = `0x4B621302`, `intensity` = `0xEFCD993D`.
  - Estado atual: aberto; alto impacto pratico (permite casar texturas sem decifrar o TOC ou o indice).
  - Confianca: **CONFIRMADA** (valor e contradicao); alcance geral ("todas as chaves sao joaat") e **PROVAVEL** — `structure/05:63` deixa aberta a funcao "for values other than the confirmed joaat case".

- **D-15 — Duas contagens do mesmo universo de texturas, sem denominador declarado: 357.115 x 352.461**
  - Descricao/sintomas: `formats/05:88` titula "Coverage (all **57 stores, 352,461 textures**)"; `structure/05:47-48` valida o campo de formato em "**357,115** of 357,115 descriptors"; `structure/05:56` fecha o mesmo conjunto em 1.026.833 secoes e 137,1 GiB. O volume e o numero de secoes coincidem, logo e o mesmo dado; o delta de **4.654** esta no que cada pagina conta (descritor x textura exportavel).
  - Causa raiz: cada arvore escolheu um denominador e nenhuma declarou o criterio; provavelmente descritores que nao geram textura (formato nao suportado, array/cube, duplicata).
  - Introduzido em: `dd592ba` (`formats/05`) + `bad5bc2` (`structure/05`).
  - Corrigido em: **SEM CORRECAO**.
  - Arquivos: `formats/05-ttpf.md`, `structure/05-ttpf-stores.md`.
  - Evidencia: os tres numeros no HEAD; 357.115 − 352.461 = 4.654 (1,3 %).
  - Estado atual: aberto; nao e necessariamente erro, e um denominador nao declarado — mas quem somar as duas paginas obtem duas populacoes diferentes com o mesmo nome.
  - Confianca: **PROVAVEL** (inconsistencia de denominador; bondade do delta: **HIPOTESE**).

- **D-16 — "34 de 51 node flags", mas a tabela nomeia 32**
  - Descricao/sintomas: `formats/16:25`: "## Node flags (`PNF_<Name>`; **34 of 51** named, confirmed)". A tabela (`:27-44`) tem 16 linhas x 2 colunas = **32 nomes** (bits 0-7, 9, 10, 12-14, 17-19, 21-30, 32, 38, 39, 41, 42, 50). `README.md:44` propaga "34 of 51 node flags".
  - Causa raiz: contagem nao recomputada apos poda da tabela; o numero foi copiado para o README em `0a52a67`, ganhando uma segunda sede.
  - Introduzido em: `dd592ba` (frase em `formats/16`); propagado em `0a52a67` (README).
  - Corrigido em: **SEM CORRECAO**.
  - Arquivos: `formats/16-roads.md`, `README.md`.
  - Evidencia: contagem direta das linhas; o proprio texto diz que o bit 31 e o unico nao nomeado explicitamente (`:46-47`), e 32 + 1 = 33, nao 34.
  - Estado atual: aberto; erro de 6 % num numero usado como credencial de cobertura, em duas paginas.
  - Confianca: **CONFIRMADA**.

- **D-17 — Total de luzes nao fecha: 774.847 declarado, 774.846 somado**
  - Descricao/sintomas: `formats/15-lights.md:38-39`: "**774,847 placed lights**: 447,756 exterior, 326,183 inside interiors, 907 in unplaced prefab groups (group space)". 447.756 + 326.183 + 907 = **774.846**. Falta exatamente 1 luz.
  - Causa raiz: uma das parcelas (ou o total) foi atualizada sem recomputar; provavel categoria residual de 1 item, ou uma contagem que devia ser 908.
  - Introduzido em: `dd592ba`.
  - Corrigido em: **SEM CORRECAO**. A reescrita de `0a52a67` removeu o total do README, mas nao tocou a pagina: hoje o numero errado tem **uma** sede (`formats/15:38`), nao duas.
  - Arquivos: `formats/15-lights.md`.
  - Evidencia: aritmetica sobre quatro valores da mesma frase; `git grep --fixed-strings "774,847"` -> 1 ocorrencia; `git show dd592ba:README.md` -> linha 42 (segunda sede historica).
  - Estado atual: aberto; cosmetico em si, mas e o unico total de luzes do repositorio e mina a confianca nos numeros grandes vizinhos (219.484.128 instancias de grama, `formats/09:55`).
  - Confianca: **CONFIRMADA** (a soma). Qual parcela erra: **NAO VERIFICADA** (o README nao arbitra).

- **D-18 — Coluna "Shaders" dos buckets soma 389; o texto declara 344 (e as instancias somam 1,82 M, nao 1,83 M)**
  - Descricao/sintomas: `formats/12:8-9` declara "**344 shaders** over 254 K drawables (**1.83 M** shader instances)". A tabela (`:20-27`) lista 193 + 44 + 59 + 90 + 2 + 1 = **389** na coluna `Shaders`. As instancias somam 1,09 M + 32 K + 596 K + 87 K + 14,6 K = **1,8196 M**.
  - Causa raiz (coluna Shaders): a coluna conta **pares (shader, bit)** e nao shaders distintos — `+0x3C` e uma mascara de bits (`formats/12:15`), e a propria tabela mostra um shader com dois bits (`0x10, 0x20`). Falta rotulo. Causa raiz (instancias): as parcelas estao arredondadas a 2-3 significativos e o total foi arredondado para cima.
  - Introduzido em: `dd592ba`.
  - Corrigido em: **SEM CORRECAO**.
  - Arquivos: `formats/12-materials.md`.
  - Evidencia: soma das seis celulas; 389 − 344 = 45 membros excedentes, compativel com sobreposicao; o delta de instancias e 10,4 K (0,57 %), acima do maximo que o arredondamento das celulas explica (~6,5 K).
  - Estado atual: aberto. A explicacao benigna para 389 e forte; para 1,83 M e mais fraca. Nao "consertar" 344 -> 389 sem antes rotular a coluna.
  - Confianca: discrepancia aritmetica **CONFIRMADA**; explicacao por multi-bucket **HIPOTESE**; delta de instancias **PROVAVEL** (arredondamento).

- **D-19 — A conta dos archetipos sem nome nao fecha: faltam 56.267**
  - Descricao/sintomas: `formats/09:72-77` declara o universo e dois resultados parciais: "**92,925** archetypes have no named drawable" … "Result: **10,982** links (5,307 by unique bbox, 5,675 by bbox plus locality); **25,676** remain ambiguous." 10.982 + 25.676 = 36.658; sobram **56.267** (60,6 %) sem categoria declarada.
  - Causa raiz: falta um balde explicito ("sem candidato por bbox") ou os 25.676 sao apenas os ambiguos *entre candidatos* e o restante nunca foi contado. O texto nao permite decidir.
  - Introduzido em: `dd592ba`.
  - Corrigido em: **SEM CORRECAO**.
  - Arquivos: `formats/09-placements.md`.
  - Evidencia: as duas somas internas fecham (5.307+5.675 = 10.982; 2.089−1.304 = 785), o que mostra que a aritmetica da pagina e confiavel nos outros pontos — e por isso a lacuna de 56.267 e um dado faltante, nao um erro de soma.
  - Estado atual: aberto; e a maior lacuna aritmetica do corpus e sustenta a unica afirmacao de cobertura de "archetypes ligados".
  - Confianca: **CONFIRMADA** (a soma nao fecha); a causa e **HIPOTESE**.

- **D-20 — `priorityLevel = 84799847` nao e o joaat do proprio nome**
  - Descricao/sintomas: `formats/09:29-35` lista hashes de campo do bloco LOD. Seis dos sete reproduzem a convencao declarada ("case-sensitive joaat do nome"); `priorityLevel` nao.
  - Causa raiz: hipotese — valor transcrito de outro campo, ou nome real diferente. Nenhuma das grafias testadas reproduz `0x84799847`.
  - Introduzido em: `dd592ba`.
  - Corrigido em: **SEM CORRECAO**.
  - Arquivos: `formats/09-placements.md`.
  - Evidencia (joaat local, validado em `hauler -> 0x5A82F9AE`): `priorityLevel` -> `0x2691F019`; `prioritylevel` -> `0x14C2D830`; `PriorityLevel` -> `0x093A7EF6`. Contraste: `lodLevel` -> `0x6C8F1715`, `parentIndex` -> `0xD89231BD`, `numChildren` -> `0xA687AC89`, `lodDist` -> `0xBF74DFA7`, `childLodDist` -> `0xCA974BCD`, todos exatos.
  - Estado atual: aberto; o campo e exportado por `tool/05-schemas.md` (`lod.priority`), entao o erro se propaga para quem consome os JSONs.
  - Confianca: nao-reproducao **CONFIRMADA**; causa **HIPOTESE**.

- **D-21 — Type bytes do chunk AWC: a lei vale para 3 de 8 casos diretos**
  - Descricao/sintomas: `structure/03:25` afirma "Type bytes are the low byte of a joaat of the chunk name" e tabela oito tags. Recalculando `joaat(nome) & 0xFF`: `peak` `0x36` OK, `data` `0x55` OK, `markers` `0xBD` OK; `stream_format` `0xF9` != `0x48`, `animation` `0xAF` != `0x5C`, `format` `0xFA` != `0x76`, `seek_table` `0xB8` != `0xA3`, `metadata` `0xCD` != `0xB2`. **Cinco de oito nao fecham.**
  - Achados adicionais: remover o separador resolve dois (`streamformat` -> `0x48`, `seektable` -> `0xA3`), o que a pagina nao diz; e `0x76`, atribuido a `format`, e exatamente `joaat("anim") & 0xFF`, enquanto a propria pagina chama o chunk de formato de "## Format chunk (**0x76 / 0xFA**)" em `:51` — ou seja, `0xFA` (o joaat de `format`) aparece no cabecalho e nao na tabela.
  - Causa raiz: lei generalizada a partir dos 3 casos que fecharam; os outros nomes provavelmente nao sao as strings hasheadas (prefixo de path, caixa, ou nome distinto do exibido).
  - Introduzido em: `bad5bc2`.
  - Corrigido em: **SEM CORRECAO**.
  - Arquivos: `structure/03-audio-awc.md`.
  - Evidencia: tabela `:28-37`; joaat recomputado; `0x5C` nao foi reproduzido por nenhuma das ~110 grafias plausiveis testadas (`animation`, `anim`, `animations`, `track`, `midi`, `pcm`…). Para `0xB2`, apenas `padding` reproduz — 1 acerto em ~110 nomes e indistinguivel de acaso (esperado ~0,4), logo nao sustenta hipotese.
  - Estado atual: aberto; a tabela serve como vocabulario observado, a lei que a justifica nao. Um decodificador que use a tabela literalmente tratara o chunk de animacao como formato.
  - Confianca: as 5 divergencias: **CONFIRMADA**; troca de rotulos entre `animation`/`format`: **PROVAVEL**; grafias de `0x5C`/`0xB2`: **NAO VERIFICADA**.

- **D-22 — `structure/00-INDEX` promete rotulo em toda afirmacao e uma secao de conflitos que nao existem**
  - Descricao/sintomas: `structure/00-INDEX.md:16-22` diz "Every statement is marked" com `OBSERVED / INFERRED / UNKNOWN`; `:24-28` diz "disagreements are recorded rather than resolved by preference — **see the conflicts section of each file**". No HEAD, os rotulos aparecem apenas em 4 dos 7 arquivos (`00`, `01`, `02`, `05`); `03-audio-awc.md`, `04-pso-objectgraph.md` e `06-models-shaders.md` nao usam nenhum deles. E `git grep -i "conflict"` devolve **1 ocorrencia** — a do proprio indice. Nao existe cabecalho `## Conflicts` em arquivo algum.
  - Causa raiz: o indice descreve a intencao editorial do autor, nao o estado dos sete arquivos que ele proprio introduziu no mesmo commit; o mecanismo de arbitragem nunca foi usado.
  - Introduzido em: `bad5bc2`.
  - Corrigido em: **SEM CORRECAO**.
  - Arquivos: `structure/00-INDEX.md`, `structure/03-audio-awc.md`, `structure/04-pso-objectgraph.md`, `structure/06-models-shaders.md`.
  - Evidencia: `git grep -c "OBSERVED" -- "structure/*.md"` -> 4 arquivos; `git grep -n -i "conflict" -- "*.md"` -> 1 linha (`00-INDEX:27`); `structure/03` e `06` nao tem sequer coluna de confianca nas tabelas.
  - Estado atual: aberto. A consequencia e direta: sem `conflicts`, as contradicoes D-06, D-09, D-10, D-11, D-12, D-13 e D-21 nao tem onde ser arbitradas.
  - Confianca: **CONFIRMADA**.

- **D-23 — `structure/04`: "289 slot values resolve", mas as parcelas somam 333**
  - Descricao/sintomas: `structure/04:59-61`: "**289** slot values resolve: 277 of 483 root ids are vehicle model names, 41 of 56 values in type `0xC48AB516` at `+0x10`, 10 keyref keys, 5 of 8 bone slots." 277 + 41 + 10 + 5 = **333**.
  - Causa raiz: provavel mistura de duas unidades — 289 pode contar **valores distintos** e 333 **slots** (44 duplicatas). O texto nao distingue, e num corpus cuja propria regra diz que "a claim that does not close arithmetically is a hypothesis", a frase nao fecha.
  - Introduzido em: `bad5bc2`.
  - Corrigido em: **SEM CORRECAO**.
  - Arquivos: `structure/04-pso-objectgraph.md`.
  - Evidencia: soma das quatro parcelas; 483 − 277 = 206, que fecha com "Unresolved after an expanded 1.3M-token universe: **206 roots**" (`:64`), confirmando que as outras tres parcelas nao estao contidas nos 277.
  - Estado atual: aberto; o resto da secao e aritmeticamente exemplar (30.683 registros, `0x28 + count x 8 = 0x3BF00` exato, 30.200 ponteiros), o que isola a frase como excecao.
  - Confianca: discrepancia **CONFIRMADA**; explicacao por valores distintos **HIPOTESE**.

- **D-24 — `formats/08` afirma que nomes de struct nao seguem joaat; 10 structs do proprio corpus provam o contrario**
  - Descricao/sintomas: `formats/08:31-34` estabelece a regra de que nomes de campo e de enum sao joaat case-sensitive ("`Position` = `0c928991`, `Flags` = `4b621302`") e conclui: "**Struct names do not follow this.**" No HEAD, dez hashes de struct publicados nas paginas 09, 10, 15 e 08 reproduzem joaat exato do nome de classe GTA V que a propria pagina cita entre parenteses.
  - Evidencia (joaat local): `CEntityDef` -> `0xCE501483` (`formats/09:11`, `formats/15:6`), `CBaseArchetypeDef` -> `0x82D6FC83` (`formats/09:60`, `formats/15:5`), `CMapData` -> `0xD3593FA6` (`formats/09:7`), `CExtensionDefLightEffect` -> `0x27922C43` (`formats/15:6`), `CLightAttrDef` -> `0xF54B227B` (`formats/15:8`), `CMloInstanceDef` -> `0x09CC28BE` e `CMloArchetypeDef` -> `0x10506455` (`formats/10:7-8`), `CMloRoomDef` -> `0x0B1811F1`, `CMloPortalDef` -> `0x995072CA`, `CMloEntitySet` -> `0xD6A799F9` (`formats/10:8`). **10 de 10 exatos.**
  - Contraponto honesto: ha structs que **nao** sao joaat dos nomes citados — `ba049770` (`PropInstanceList`), `7c475b5d` (`GrassInstanceList`), `0c69a100` (no de estrada), `0f2da4c2` (link), `eba8aebb` (juncoes) —, o que provavelmente motivou a frase.
  - Causa raiz: generalizacao a partir dos casos em que o nome GTA V nao resolve (structs novos de VI ou com nome de classe desconhecido), escrita como regra universal e nunca testada contra os structs que a propria pagina nomeia.
  - Introduzido em: `dd592ba`.
  - Corrigido em: **SEM CORRECAO**.
  - Arquivos: `formats/08-meta.md`; afetadas `formats/09`, `10`, `15`.
  - Estado atual: aberto. O efeito real e caro: quem implementa a tabela de structs de meta resource (`formats/08:29`, "instance block: u32 struct hash") le que a hash nao e joaat e nao tenta a resolucao que funciona em 10 casos.
  - Confianca: refutacao da frase **CONFIRMADA**; a leitura correta ("struct hash e joaat quando o nome e uma classe conhecida; os 5 structs acima sao novos/nao nomeados") e **PROVAVEL**.

- **D-25 — `README` promete que "toda pagina" termina em open questions e da confianca por campo; `formats/17` nao faz nem um nem outro**
  - Descricao/sintomas: `README.md:25-26`: "Every page gives each field a confidence level and **ends with its open questions**". `formats/17-other.md` (17 linhas) tem tabela `| Format | State |` sem coluna de confianca e **nao tem secao `## Open questions`** (`git grep -n "^## Open questions"` -> ausente em `formats/17`; `formats/06` usa "## 6.3 Open unknowns", equivalente funcional).
  - Causa raiz: promessa editorial escrita para as 17 paginas de `dd592ba`; `formats/17` e a unica que nasceu como tabela de estado e nunca foi promovida.
  - Introduzido em: `dd592ba` (pagina) / `0a52a67` (promessa no README).
  - Corrigido em: **SEM CORRECAO**.
  - Arquivos: `README.md`, `formats/17-other.md`.
  - Evidencia: `git grep -n "^## Open questions" -- "*.md"` -> 21 ocorrencias, nenhuma em `formats/17`.
  - Estado atual: aberto; defeito de baixa gravidade, mas e o mesmo README que encaminha o leitor para essa pagina (`README.md:23`).
  - Confianca: **CONFIRMADA**.

- **D-26 — `formats/13`: as tres faixas de confianca somam 98,2 %, nao 98,3 %**
  - Descricao/sintomas: `formats/13:68-69`: "**98.3 % of collision area is categorised (22.7 % high, 68.1 % medium, 7.4 % low confidence), 243 ids**". 22,7 + 68,1 + 7,4 = **98,2**.
  - Causa raiz: ou uma das quatro celulas foi arredondada para longe, ou 98,3 e a area coberta e as faixas de confianca cobrem so 98,2 (0,1 % de area categorizada sem faixa). O texto nao permite decidir.
  - Introduzido em: `dd592ba`.
  - Corrigido em: **SEM CORRECAO**. `README.md:41` repete "98.3 % of area".
  - Arquivos: `formats/13-collision.md`, `README.md`, `tool/05-schemas.md:105` (`area_covered: 0.983`).
  - Evidencia: soma das tres parcelas; o schema do exportador usa 0,983 como valor de area coberta, o que sugere que 98,3 e o campo correto e as faixas perderam 0,1 % no arredondamento.
  - Estado atual: aberto; diferenca dentro da margem de arredondamento (cada parcela a 1 decimal pode esconder ate +-0,15 no total), por isso **nao** trato como erro de contagem.
  - Confianca: **PROVAVEL** (arredondamento), nao confirmado como erro.

- **D-27 — Ancora do README apontava para um slug inexistente (unico defeito corrigido do historico)**
  - Descricao/sintomas: `README.md:50` linkava `formats/06-bcpack.md#how-to-resume`, mas o heading real e `## 6.4 How to resume` (`formats/06-bcpack.md:123`). O link abria a pagina no topo em vez de rolar — justamente a secao "how to resume" do maior problema aberto declarado (BCPack). O leitor chegava ao inicio de uma pagina de 149 linhas sem nenhuma indicacao de onde estava o plano de retomada.
  - Causa raiz: slug escrito a mao a partir do texto ("how to resume") em vez de derivado do heading numerado. O slugger do GitHub remove a pontuacao e baixa a caixa: `6.4` -> `64`.
  - Introduzido em: `0a52a67` (a reescrita do README criou o link).
  - Corrigido em: `4b3beb1` (16 s depois).
  - Arquivos: `README.md`, `formats/06-bcpack.md`.
  - Evidencia: `git show 4b3beb1` -> `-[...](#how-to-resume)` / `+[...](#64-how-to-resume)`; `git log --all -S "how-to-resume" --format="%h %s"` -> so `0a52a67` (o commit de correcao mantem a substring, por isso nao aparece); auditoria de 154 links + ancoras no HEAD -> 0 problemas, `#64-how-to-resume` incluso.
  - Estado atual: **resolvido**. `README.md:50` usa `#64-how-to-resume`.
  - Confianca: **CONFIRMADA**.

- **D-28 — `formats/07`: o modo de eixo 0 e um espelhamento, mas o paragrafo seguinte jura que so existem permutacoes ciclicas**
  - Descricao/sintomas: a tabela do campo `0x6C` (`formats/07:27`) define tres modos: 2 = z, slots `(x, y, z)` (identidade); 1 = y, slots `(z, x, y)`; **0 = x, slots `(z, y, x)`, "a swap and not a rotation"**. O paragrafo "Axis rotation (confirmed)", tres linhas abaixo (`:56-61`), afirma que o quantizador "rotates the other two **cyclically**, so the decoded (x, y, z) are model axes (a, b, c) with (a, b, c) = (0,1,2), (2,0,1) or (1,2,0). Pick the order whose face normals best agree". O conjunto `{(0,1,2), (2,0,1), (1,2,0)}` nao contem `(2,1,0)` — o modo 0. E a leitura anterior, `(y, z, x)` = `(1,2,0)`, **esta** no conjunto: a correcao registrada como "2026-11-20" trocou uma permutacao ciclica por um espelhamento, ao mesmo tempo em que o texto declara ciclicas como confirmadas.
  - Terceira tensao no mesmo par: `:27` diz que "SixNG also scores **all six** permutations against the stored normals"; `:59` manda "Pick the order" entre **tres**. As duas frases descrevem o mesmo procedimento com espacos de busca diferentes.
  - Causa raiz: hipotese — a coluna de modo e a coluna de tupla foram atualizadas em momentos diferentes, ou o "fix" foi escrito a partir de um caso em que a pontuacao de normais favoreceu o espelhamento; o paragrafo de confirmacao nao foi reescrito.
  - Introduzido em: `dd592ba` (o texto que carrega as duas afirmacoes); a correcao que gerou a divergencia e anterior ao commit 0 — `git log --all -S "tipped"` devolve so `dd592ba`, logo nao ha diff dela.
  - Corrigido em: **SEM CORRECAO**.
  - Arquivos: `formats/07-models.md` (§7.2).
  - Evidencia: as duas passagens no HEAD; aritmetica de permutacoes (o modo 0 e a transposicao `y`/`z`; os modos 1 e 2 sao as duas rotacoes ciclicas nao triviais).
  - Estado atual: aberto. Impacto pratico e exatamente o sintoma que o texto diz ter corrigido: paineis de parede girados 90 graus. Se a nova leitura estiver certa, a regra "so ciclicas" esta errada; se a regra estiver certa, a "correcao" reintroduziu o erro por outro caminho.
  - Confianca: inconsistencia interna **CONFIRMADA**; qual leitura e correta: **NAO VERIFICADA** (exigiria malha de referencia).

---

## E. Relacao de causa e efeito

**Cadeia 1 — reescrita de enquadramento -> link quebrado + perda de dados.**
`0a52a67` reescreve o README inteiro (72+/93−, nao patch) -> cria uma ancora escrita a mao (`#how-to-resume`) **e** descarta a tabela "What it extracts" -> `4b3beb1`, 16 s depois, corrige so a ancora -> o defeito de navegacao morre (D-27: hoje o link resolve), mas os numeros 227.766 / 2.249.490 / 30.785 / 27.644 sobrevivem perdidos porque ninguem checou "todo numero orfao?" (D-04). Uma reescrita que durou minutos produziu a unica correcao do historico **e** a maior perda de conteudo.

**Cadeia 2 — rebrand por arquivo -> identidade dividida em tres camadas.**
`dd592ba` cria a identidade SixNG em 24 arquivos -> `0a52a67`/`14b4bff` trocam o sujeito so em `README.md` e `LEGAL.md` (D-01) -> nenhum commit posterior toca `formats/` ou `tool/` -> o metadado do GitHub, que nao e arquivo, nunca e tocado em momento algum (D-03). Resultado: o repositorio se apresenta de duas formas conforme a porta de entrada, e de uma terceira no indice do GitHub.

**Cadeia 3 — arvore nova sem reconciliacao -> nove defeitos em um commit.**
`bad5bc2` adiciona `structure/` (461 linhas, zero edicoes) -> (a) nada linka a arvore (D-02); (b) `structure/01` contradiz `formats/01` em `0x24`/`0x28` e agrava com uma terceira contagem (D-06); (c) `structure/03`/`04`/`06` contradizem a pagina de estado `formats/17` (D-07, D-08); (d) `structure/02` erra o rotulo do tiling e reabre questao fechada (D-09, D-10); (e) `structure/02`/`05` reduzem o campo DXGI a 7 bits enquanto `formats/05` publica 8 (D-11); (f) `structure/04` declara big-endian contra `formats/17` e contra o proprio exemplo (D-12); (g) `structure/06` nega os vertex streams de `formats/07` (D-13); (h) `structure/03` generaliza uma lei de hash a partir de 3/8 casos (D-21); (i) o proprio indice da arvore promete rotulos e conflitos que ela nao cumpre (D-22). Um `git diff` do commit parece inofensivo: so adicoes.

**Cadeia 4 — descoberta nao propagada.**
`structure/05` descobre que a chave do `ttpf` e joaat (`hauler`) -> `bad5bc2` nao edita `formats/05`/`07` -> o HEAD mantem "64-bit hash desconhecido" ao lado da resposta (D-14), e o bloqueio do TOC continua sendo tratado como se dependesse de decifrar o indice inteiro. O custo e de uma linha de edicao; o beneficio seria destravar o casamento de texturas.

**Cadeia 5 — numero errado ganha duas sedes.**
`formats/16:25` afirma 34 de 51 enquanto a tabela lista 32 (D-16) -> `0a52a67` copia o numero para `README.md:44` -> corrigir so uma das sedes piora a inconsistencia. Padrao identico ao das luzes (D-17), onde a reescrita **removeu** a segunda sede em vez de corrigir o numero: hoje o erro esta so em `formats/15`.

**Cadeia 6 — pagina de estado como ponto unico de erosao.**
`formats/17` e a unica visao consolidada de "o que falta" -> `bad5bc2` adiciona tres formatos que ela ainda declara "planned/not decoded" (D-07, D-08) -> `README.md:23` encaminha o leitor exatamente para ela -> quem entra pelo caminho oficial recebe o estado de 14:47, nao o de 15:51. Nao ha CI, checklist ou data por afirmacao que force a atualizacao: a unica data do corpus e global ("as of 2026-11-20", 33 ocorrencias em 25 arquivos) e a arvore mais nova nao tem data nenhuma.

**Cadeia 7 — vocabulario de confianca incompativel.**
`formats/README.md:10-15` define `confirmed/inferred/unknown`; `structure/00-INDEX.md:16-22` define `OBSERVED/INFERRED/UNKNOWN` e `structure/01:18-19` usa ainda `DISPUTED` -> nao ha contradicao factual, mas o leitor nao consegue comparar forca de evidencia entre as arvores, e `DISPUTED` nao tem definicao em lugar algum. Introduzido em `bad5bc2`, nunca reconciliado; e a causa formal por tras de D-06.

**Cadeia 8 — uma regra correta foi escrita como absoluta.**
`formats/08:34` generaliza "struct names do not follow this" a partir dos structs que nao resolvem (D-24) -> como a mesma pagina ensina que a hash e o caminho para nomear campos, o leitor descarta a resolucao de struct -> e perde 10 casos que funcionam. O erro nao esta no dado (todos os 10 hashes reproduzem), esta na quantificacao da excecao.

---

## F. Correcoes importantes

**F-1 — Ancora de heading numerado (`4b3beb1`, `README.md:50`).**
Errado: `#how-to-resume` para `## 6.4 How to resume`. Correcao: `#64-how-to-resume`. Funciona porque o slug do GitHub baixa a caixa, remove a pontuacao e transforma espacos em hifens, preservando o prefixo numerico — `6.4` vira `64`. Reutilizavel em qualquer Markdown com headings numerados; e a classe de link que quebra em silencio, porque o renderizador nao avisa. Dependencia: nenhuma. Risco: se o heading for renumerado, a ancora quebra de novo — e um acoplamento manual; heading sem numero + ancora estavel seria mais robusto. Verificacao atual: auditoria dos 154 links relativos do repositorio (caminho + slug) retorna 0 problemas.

**F-2 — Reenquadramento legal sem perda de escopo (`14b4bff`, `LEGAL.md`).**
Errado: o texto juridico descrevia a ferramenta ("SixNG works only on game files from your own copy… It does not download, unlock or decrypt anything"). O repositorio passou a ser so documentacao. Correcao: reescrever o sujeito de "SixNG" para "these notes", preservando as quatro proibicoes (arquivos de jogo, assets extraidos, chaves, material proprietario/NDA) e a nao-afiliacao, e acrescentando uma clausula que faltava: "Short identifiers that appear in them (structure offsets, hash values, flag names, sample asset names) are there to describe the formats, not to reproduce game content" (`LEGAL.md:22-25`). Essa clausula e a correcao mais bem pensada do historico: ela antecipa a objecao de que um doc de RE reproduz conteudo proprietario, e separa identificador de conteudo. Reutilizavel como padrao quando um repo docs-only se separa do codigo. Risco residual: ver H-05.

**F-3 — Lei do header `0x2C` (`structure/01:25-33`).** Errado: `0x2C` = "unknown flags" (`formats/01:20`). Correcao: dois grupos de valores (`0x90F01` para audio/navmeshes/packs/textures/script/restofworld/bvh4; `0x90F02` para levels/data/anim/story000/movies/common/core_bvh4), e no segundo grupo `0x2C = round16(10 x file_count)` exato (2.078 x 10 + 4 = 20.784). Converte um campo "unknown" em predicao testavel, mas so para um dos grupos — o texto e explicito que o primeiro nao segue a lei. Reutilizavel: usar um campo constante por classe como rotulo de variante e mais barato que decodifica-lo.

**F-4 — Largura de indice de osso por tamanho de secao (`formats/07:140-146`).** Errado: assumir u8 porque o format code e `0x00`. Correcao: largura = `(offset da proxima secao − offset desta) / V`; slot 8 tem **2 bytes por vertice** em esqueletos com >255 ossos (corpo de carro high-LOD tem 295); exportar `JOINTS_0` como u16 quando o indice passa de 255; reexportar GLBs antigos. Principio geral: quando codec-code e layout divergem, o layout vence.

**F-5 — Regra dos system streams extras (`formats/02:14-22`).** Errado: parar nos streams do page-map; continuacoes de sistema caiam no lado grafico e deslocavam todos os offsets ("the model has no geometry"). Correcao: anexar o proximo stream enquanto houver >= `max(2, size / 64 KiB)` valores `0x5…` **alinhados a 4** apontando para dentro dele. O texto documenta as duas tentativas: exigir alinhamento 8 e estrito demais (meta resources alcancam a continuacao por ponteiros 4-alinhados), e valores tipo hash puxam streams graficos se o alinhamento nao for filtrado. Heuristica com modo de falha bidirecional explicitado — o melhor exemplo de metodo do corpus.

**F-6 — Mip tail e ordem invertida de mips (`formats/05:74-86`).** Errado: sem regra de tail, os mips pequenos sao impossiveis (compartilham um bloco e nao seguem o passo dos grandes). Correcao: mips do menor para o maior; `max_mips_in_tail` = 8 (blocos de 4 KiB) / 12 (64 KiB); `m = max_mips_in_tail − 1 − k`; `off = m > 6 ? 16 << m : m << 8`; origem por bits pares/impares de `off`. Validado por downsample (erro 0-4 niveis) em BC1/3/4/5/7 em todos os modos.

**F-7 — Recuperacao de mip inferior para BCPack (`formats/05:98-107`).** Errado: textura com mip 0 em BCPack descartada como irrecuperavel. Correcao: chunks sao faixas de bytes, nao texturas, e os mips vao do menor para o maior, entao os mips pequenos frequentemente caem em chunks zlib; exportar o maior mip inteiro fora dos chunks BCPack. Impacto declarado: 14.441 texturas (4,1 %) exportadas em mip inferior; 29.310 (8,3 %) sem nenhum mip legivel. Padrao reutilizavel para qualquer codec opaco armazenado em blocos independentes.

**F-8 — Walk de `PSIN` e causa unica das falhas (`structure/04:29-36`).** Errado: tratar falhas de secao como variantes de formato. Correcao: `tail_length = size − 12` (o tamanho inclui o header de 8 bytes) e a causa real das falhas de walk e o corte de pagina de 64 KiB do container, nao o formato. Reutilizavel: erro recorrente em offsets altos ⇒ suspeitar do container antes do formato.

---

## G. Problemas ainda existentes

- **G-01 — Desconhecimento real, declarado (escopo, nao defeito).** BCPack: "0 % of BCPack data is decoded as of 2026-11-20" (`formats/06:8-9`), 29.310 texturas (8,3 %) sem mip legivel; funcao de hash de nomes de shader/parametro/superficie de colisao desconhecida — ~1.500 nomes candidatos testados contra joaat (duas caixas), FNV-1/1a, CRC-32, Murmur3, djb2, sdbm, Adler e formas byte-swapped, sem acerto (`formats/12:5-7`); cifra/chave do TOC do RPF9 (`formats/01:25-28`); dados de veiculo (pintura, rodas, handling) em PSO com hashes de 64 bits (`formats/11:17-25`).
- **G-02 — Contradicoes abertas entre documentos:** D-06, D-07, D-08, D-09, D-10, D-11, D-12, D-13, D-14, D-15, D-21, D-24, D-28. Todas do mesmo tipo: `structure/` mediu melhor, `formats/` nao foi atualizado. D-10, D-13, D-24 e D-28 sao correcoes de uma frase; as demais exigem decisao editorial, nao dados novos.
- **G-03 — Contagens erradas verificaveis sem o jogo:** D-16 (34 vs 32), D-17 (off-by-one), D-18 (389 vs 344), D-19 (56.267 sem balde), D-23 (289 vs 333), D-26 (98,2 vs 98,3). Todas corrigiveis por quem so tem o texto.
- **G-04 — Ambiguidade de `0x28` no header RPF9.** `formats/01:19` diz "one of 4 values across the install"; `structure/01:19` diz "varies per archive" (DISPUTED). Nenhuma das duas publica a contagem de valores distintos nem a distribuicao por arquivo. Falta dado, nao argumento. **NAO VERIFICADA.**
- **G-05 — Contabilidade dos metodos de link de interior.** `formats/10:19-22` mistura contagens fechadas (`exact 305`, `ordered 899`) com cotas compartilhadas ("floor fit — part of 463", "bbox — rest"). A aritmetica fecha (305 + 899 + 463 = 1.667), mas a coluna "Share" muda de unidade no meio da tabela: duas linhas sao contagens e duas sao fracoes da mesma cota. O mesmo vale para "79,3 % at high or medium confidence" (`:24-25`), que nao e derivavel das linhas (1.204/1.667 = 72,2 %), porque parte da cota de 463 tambem e high/medium. **PROVAVEL** (redacao, nao contagem).
- **G-06 — `tool/` descreve binario inalcancavel.** D-05: placeholder de URL, zero URLs em 33 `.md`, sem `Cargo.toml`, sem release, sem tag.
- **G-07 — Numeros de desempenho inauditaveis.** `tool/` traz medicoes de uma maquina so ("levels.rpf takes about 3–6 minutes", "~180 GB with GLBs", "one vehicle is 248 parts and about 100 MB"), com a unica qualificacao "one desktop with the game files on an NVMe drive". Nao sao falsos; sao inverificaveis, e nao ha data por comando.
- **G-08 — `structure/06` admite uma divergencia que nao quantifica.** "Two independent counts of the same cache disagree on stage totals, so stage classification depends on the classifier used" (`:70-71`). E exatamente o tipo de afirmacao que a regra do proprio indice (`00-INDEX:27-28`) manda tratar como hipotese — e ela nao vem com os dois numeros. **NAO VERIFICADA.**
- **G-09 — Erosao de estado sem mecanismo de sincronizacao.** O corpus se declara datado ("as of 2026-11-20", 33 ocorrencias em 25 arquivos), mas `structure/` nao tem data nenhuma, nao ha CI, nao ha checklist e nao ha campo de data por pagina de estado. E o que produziu D-07, D-08 e D-04.
- **G-10 — `formats/12` usa "1.83 M" e a soma das celulas da 1.82 M** (D-18). Diferenca pequena, mas e o unico total de instancias de shader do repositorio.

---

## H. Regressoes

- **H-01 — `0a52a67`: perda de dados primarios de cobertura (D-04).** Um commit que "so" mudava o enquadramento apagou 227.766 nomes recuperados, 2.249.490 entidades em 30.785 map files, 27.644 tiles de navmesh e a taxa de 95 % de resolucao, sem repo-los em nenhuma pagina de `formats/`. Nao e reintroducao de defeito: e perda liquida de conteudo. Gravidade alta, porque e irreversivel dentro do historico.
- **H-02 — `0a52a67`: link quebrado criado e corrigido em 16 s (D-27).** Regressao de navegacao nascida na propria reescrita; `4b3beb1` a elimina. Exemplo limpo de "reescrita total cria defeito que edicao incremental nao criaria" — e de como o autor verificava links a mao.
- **H-03 — `0a52a67`: divergencia de terminologia de coordenadas.** `dd592ba:README.md` dizia `glb(x, y, z) = rage(x, z, −y)`; a reescrita passou a `gltf(...)` (`README.md:63`), enquanto `tool/04-output.md:53`, nao tocado, continua `glb(x, y, z) = rage(x, z, −y)`. A transformacao e a mesma, o nome do formato nao: o README deixou de falar a lingua do guia de saida que ele proprio linka. **CONFIRMADA** (texto); impacto baixo (cosmetico), risco real de o leitor tratar `glb` e `gltf` como saidas distintas.
- **H-04 — `bad5bc2`: regressao de confianca sem editar nada.** O commit so adiciona arquivos, mas degrada onze afirmacoes preexistentes ao contradize-las (D-06 a D-15, D-21) e deixa a arvore nova orfa na navegacao (D-02). O `git diff` do commit e 100 % aditivo; a coerencia do corpus caiu. Caso mais instrutivo do repositorio: num corpus documental, "so adicionar" pode ser regressao.
- **H-05 — `14b4bff`: `LEGAL.md` deixou de descrever o comportamento da ferramenta.** O texto antigo afirmava "It does not download, unlock or decrypt anything. It does not bypass copy protection, and it cannot read the encrypted parts of the archives." O novo afirma "Nothing here explains how to obtain game files, decrypt protected data or bypass copy protection" — uma promessa sobre o **texto**, nao sobre o **programa**. Como `tool/01-install.md:9-24` continua instruindo a obter o programa e apontar `--game` para a instalacao, o repositorio perdeu a unica pagina que declarava o comportamento do binario. Classificacao: **PROVAVEL** regressao de escopo — pode ser intencional (o repo nao responde mais pela ferramenta), mas nao foi compensada em lugar nenhum.
- **H-06 — Nao ha reversoes.** `git log --all -i --grep="revert"` vazio; `git log --all --merges` vazio. A unica correcao dentro do historico e `4b3beb1`. Nenhum outro defeito documental foi corrigido por commit — dos 28 itens de D, 27 seguem abertos no HEAD e apenas D-27 esta resolvido.

---

## I. Conhecimento reutilizavel

- **I-1 — joaat como oraculo de consistencia documental.** Problema resolvido: distinguir "campo desconhecido" de "hash transcrito errado". Implementacao: reimplementar o Jenkins one-at-a-time e recalcular **todo** hash que o documento nomeia; validar o oraculo antes num valor conhecido (`hauler -> 0x5A82F9AE`). Resultado neste corpus: 15 de 16 exatos, 1 falha (D-20), mais 2 hashes nao nomeados reproduzidos (`lightType`, `timeFlags`). Origem: `formats/08:31-33`, `formats/09:29-35`, `formats/15`, `structure/05:50-51`. Dependencias: nenhuma; ~15 linhas. Riscos: falso negativo se o documento usar hash de 64 bits ou outra funcao; colisao de 32 bits e irrelevante para nomes curtos distintos. Adaptacao: qualquer corpus de RE que publique hashes de nomes pode ser auditado assim, e o mesmo teste descobre nomes que o documento nao revelou.
- **I-2 — Hash de struct = joaat do nome da classe, quando a classe e conhecida.** Descoberta desta analise (D-24): 10 de 10 structs nomeadas pelo corpus reproduzem joaat do nome GTA V (`CEntityDef`, `CBaseArchetypeDef`, `CMapData`, `CExtensionDefLightEffect`, `CLightAttrDef`, `CMloInstanceDef`, `CMloArchetypeDef`, `CMloRoomDef`, `CMloPortalDef`, `CMloEntitySet`), enquanto 5 structs novos/nao nomeados nao reproduzem nada. Utilidade: resolve a tabela de structs de meta resource e a tabela de tipos de PSO por dicionario de nomes de classe. Conflito: a regra escrita pelo corpus diz o contrario (`formats/08:34`) — nao seguir a regra escrita, testar.
- **I-3 — Fechamento aritmetico como filtro de leitura.** Problema: separar erro real de impressao. Implementacao: para cada paragrafo com total + parcelas, somar; para cada percentual, dividir; para cada "N de M", subtrair; para cada "N named" com tabela, contar linhas. Neste corpus, das 23 checagens da tabela da secao B, 16 fecham e 7 nao (D-16 a D-20, D-23, D-26). Origem: a regra e do proprio corpus (`structure/00-INDEX.md:27-28`). Risco: somas que legitimamente excedem o total (mascaras de bits, D-18) e cotas compartilhadas (G-05). Recomendacao: rodar antes de usar qualquer numero como especificacao, e nunca "consertar" o total antes de descobrir qual parcela e a unidade errada.
- **I-4 — Cadeia de 64 KiB do RPF9 como causa unica de falhas de parse.** Implementacao (`structure/01:52-58`): um bloco continua no proximo stream **se e somente se** decodificou exatamente 65.536 bytes e o proximo comeca no slot 16-alinhado seguinte; nao ha flag. Aplicacao: walk de `PSIN`, paginas de recurso, streams extras de sistema. Riscos: regioes vazias sao ausentes em vez de padding (`structure/01:58`); archives aninhados exigem varredura de magic a cada 4 KiB (`structure/01:60-65`). Recomendacao: implementar a remontagem antes de qualquer parser de estrutura.
- **I-5 — Largura implicita em streams compactos.** Implementacao: `largura = (offset da proxima secao − offset desta) / V`; nunca confiar so no format code (`formats/07:41,140-146`). Risco: falha silenciosa se `V` estiver errado por 2x. Invariante de checagem: soma das larguras x V = tamanho da secao.
- **I-6 — Projecao por alinhamento + densidade.** Implementacao: F-5. Risco: limiar `max(2, size/64 KiB)` e empirico. Recomendacao: documentar tambem as tentativas rejeitadas (alinhamento 8), como o repositorio fez — e o que torna a regra auditavel.
- **I-7 — Ordem invertida de mips + mip tail (RDNA2 tiled).** Implementacao: F-6/F-7. Dependencia: familia de tiling por descriptor (bits 24-27, `formats/05:59`). Reutilizavel em qualquer extracao de console com blocos de 4/64 KiB; a receita "exportar o maior mip fora do codec opaco" e independente do formato.
- **I-8 — Teste do bit de fronteira para campos mascarados.** D-11 mostra que validar um campo so contra os valores observados nao prova a largura: 357.115 descritores "sem valores desconhecidos" nao distinguem `&0x7F` de `&0xFF` porque o bit 10 nunca aparece. Regra: para cada campo mascarado, verificar se o **bit mais alto declarado** ja foi visto ligado; se nao, a largura e inferencia, nao observacao. Corolario util: um valor unico acima do limite (aqui, formato 157) derruba a mascara estreita sozinho.
- **I-9 — Metodo de auditoria do corpus (o que deu retorno).** Em ordem de retorno por minuto gasto: (1) `git log -1 --format=%h -- <arquivo>` por arquivo, para provar quais arvores pararam de ser tocadas; (2) auditoria mecanica de links + ancoras (slug do GitHub); (3) reimplementacao de hashes e somas contra o texto; (4) busca de **vocabulario divergente** (`confirmed/inferred/unknown` x `OBSERVED/INFERRED/UNKNOWN/DISPUTED`) como detector de camadas escritas por autores diferentes; (5) `grep` de "not decoded" contra `grep` de "decoded" para achar estados obsoletos; (6) verificacao de que o numero prometido no titulo de uma secao aparece na tabela abaixo dele; (7) grep dos numeros que sumiram entre duas versoes do README (`git show <commit>:README.md` x HEAD). Custo total: minutos. Retorno: 28 defeitos, 26 confirmados, 1 ja corrigido.

---

## J. Conclusao tecnica

`GTA6-Asset-Info` e um corpus de engenharia reversa de alta densidade preso a uma historia git de 64 minutos e cinco commits. Os quatro primeiros, de um autor, entregam uma especificacao ja madura de `formats/` + `tool/` (1.833 linhas) e em seguida tentam reposiciona-la de "documentacao da ferramenta" para "notas de formato" - rebrand que atinge apenas `README.md` e `LEGAL.md`, deixa 81 ocorrencias da marca antiga em 24 arquivos e nunca chega ao metadado do GitHub. O quinto, de outro autor, 58 minutos depois, injeta uma camada de medicao mais rigorosa (`structure/`, 461 linhas) que **contradiz** a primeira em dez pontos materiais, **nao e linkada em lugar nenhum** e **nao tem data**.

O resultado e um repositorio em que a informacao mais confiavel e a menos encontrivel, e a mais encontrivel esta parcialmente obsoleta. Onde o corpus mediu, mediu bem: 15 de 16 hashes nomeados reproduzem joaat case-sensitive (e dois hashes nao nomeados tambem), todos os percentuais de cobertura de textura fecham sobre 352.461, o grafo de objetos PSO fecha byte a byte, o volume dos stores fecha em 137,1 GiB, as conversoes de unidade fecham, os 154 links resolvem. Onde o corpus **emendou** texto de fontes diferentes, errou: `PSIN` com duas ordens de bytes (e uma terceira dentro do mesmo arquivo), audio "nao decodificado" ao lado de um mapa completo de Opus, `priorityLevel` com hash que nao e o do proprio nome, tags AWC com 5 de 8 bytes inexplicados e um deles igual ao joaat de outro nome, um modo de eixo que contradiz a propria regra de permutacoes da pagina, luzes somando um a menos, archetipos somando 56 mil a menos, 34 flags onde existem 32, 344 shaders onde a coluna soma 389.

A assimetria tem uma causa unica e identificavel: **o corpus cresceu por adicao, nunca por reconciliacao**. Nenhum dos dois autores voltou a uma pagina existente para contradize-la, exceto na correcao de 16 segundos que so mudou uma ancora. Em 64 minutos foram criados tres corpos documentais com tres vocabularios, duas identidades e uma data compartilhada que nao cobre o corpo mais novo — e nenhum commit de sincronizacao.

Para quem for usar isto como especificacao, a ordem correta de leitura e a inversa da navegacao: comecar por `structure/` (mais novo, mais duro, com aritmetica verificavel e sem marca), usar `formats/` como catalogo de campos e de numeros de escala, e tratar qualquer afirmacao de estado ("not decoded", "planned", "confirmed") como datada de 14:47 de 2026-11-20. Quatro leituras de uma linha resolvem a maior parte do atrito: aceitar que a chave do `ttpf` e joaat, que o tile mode esta nos bits 24-27, que audio esta mapeado, e que structs nomeados sao joaat. O maior ganho disponivel nao e decodificar BCPack — e reconciliar 461 linhas com 1.833, o que custa uma tarde e nao exige o jogo.

---

## K. Indice de commits

| # | Hash | Data (-05:00) | Autor | Assunto | Delta |
|---|---|---|---|---|---|
| 1 | `dd592ba` | 2026-11-20 14:47:50 | Brandon `<foiler25@gmail.com>` | GTA VI asset format docs and SixNG tool guide (2026-11-20) | 27 arquivos, +2.658 (LICENSE 675; formats 1.265; tool 568; README 96; LEGAL 54) |
| 2 | `0a52a67` | 2026-11-20 14:51:47 | Brandon `<foiler25@gmail.com>` | README: describe the repo as GTA VI format info, not the SixNG tool | `README.md` 72+/93− |
| 3 | `4b3beb1` | 2026-11-20 14:52:03 | Brandon `<foiler25@gmail.com>` | README: fix BCPack anchor | `README.md` 1+/1− |
| 4 | `14b4bff` | 2026-11-20 14:53:24 | Brandon `<foiler25@gmail.com>` | LEGAL: describe the repo as format notes, not the SixNG tool | `LEGAL.md` 30+/31− |
| 5 | `bad5bc2` | 2026-11-20 15:51:51 | walkthestars `<64241+walkthestars@users.noreply.github.com>` | structure: byte-level layout notes for RPF9, resources/textures, AWC audio, PSO and object graph, TTPF stores, models and shaders | 7 arquivos, +461, 0− |

Cadeia de pais: `dd592ba -> 0a52a67 -> 4b3beb1 -> 14b4bff -> bad5bc2` (linear; sem merges, tags, branches de trabalho ou PRs).

Commits por arquivo (ultimo toque, `git log -1 --format=%h -- <arquivo>`): `README.md` `4b3beb1`; `LEGAL.md` `14b4bff`; `LICENSE` `dd592ba`; **todos** os 18 de `formats/` e os 6 de `tool/` `dd592ba`; **todos** os 7 de `structure/` `bad5bc2`.

---

## L. Limitacoes desta analise

1. **Sem os dados do jogo.** Nao ha arquivos do jogo neste ambiente. Nenhuma afirmacao de formato pode ser testada contra bytes; todas as confirmacoes desta analise sao sobre **o texto** (aritmetica, hashes, links, coerencia interna, coerencia entre paginas), nunca sobre o formato em si. Isso vale inclusive para as classificacoes "CONFIRMADA" dos defeitos: confirmam a inconsistencia, nao qual lado esta certo.
2. **Sem o codigo.** O `SixNG` citado em `tool/` nao esta no repositorio e sua URL e um placeholder. Comandos, tamanhos de saida, tempos e categorias de `tool/02-cli.md` e `tool/05-schemas.md` nao foram executados nem verificados; G-07 fica inverificavel por construcao.
3. **API do GitHub bloqueada.** `api.github.com` devolveu HTTP 403 (rate limit) nesta sessao. Descricao, `isFork`, licenca, `createdAt`, estrelas e forks foram lidos do HTML publico em 2026-11-25; contagem de issues e de PRs **NAO VERIFICADA**. A ausencia da aba Issues e um indicio, nao uma prova de `has_issues:false`.
4. **Historico raso por construcao.** Cinco commits em 64 minutos, sem merges, branches, PRs ou tags. Nao ha "arqueologia de evolucao" a fazer: a evolucao real ocorreu fora do git. A secao C registra publicacao, nao desenvolvimento. As correcoes anteriores ao commit 0 (eixo do modo 0, indice de osso u16, system streams) existem so como relato no texto, sem diff — a de eixo, inclusive, chega ao HEAD carregando a contradicao de D-28.
5. **Semantica nao decidivel pelo texto.** Endianness do `PSIN`; semantica de `0x24`/`0x28` do RPF9; qual familia de tiling e `S` ou `D`; grafias de `0x5C`/`0xB2` do AWC; se o campo DXGI tem 7 ou 8 bits; se `289` em `structure/04` conta valores ou slots; se a perda de cobertura em `0a52a67` foi deliberada. Todas **NAO VERIFICADAS** — exigiriam amostras do jogo ou o autor.
6. **`walkthestars` != `Brandon`.** Nao foi possivel verificar se sao a mesma pessoa, se houve transferencia de repositorio ou se o commit 5 e um patch de terceiro aplicado pelo dono. A autoria do commit e o que o git registra; a relacao entre as duas identidades e **NAO VERIFICADA**.
7. **Datas assumidas.** Os commits sao de 2026-11-20 e o HTML confirma `createdAt 2026-11-20T19:53:24Z`; a analise assume os carimbos como verdadeiros (nao ha como auditar relogio de autor).
8. **Cobertura de leitura.** 100 % dos 34 arquivos rastreados foram lidos integralmente (2.422 linhas), mas a verificacao aritmetica foi exaustiva apenas onde havia totais explicitos. Tabelas de offset por campo (centenas de linhas em `formats/05`, `06`, `07` e `structure/01`-`06`) foram lidas e checadas quanto a coerencia interna, nao re-derivadas — re-deriva-las exigiria os binarios. Nao houve tentativa de reproduzir o coder rANS do BCPack nem o cipher do TOC.
9. **Um verificador, uma implementacao.** joaat foi implementado uma vez e validado em tres valores conhecidos (`hauler`, `Position`, `Flags`); a mesma implementacao foi usada em todos os testes de hash. Um erro sistematico na implementacao seria detectado por esses tres valores, mas nao por um quarto independente.
