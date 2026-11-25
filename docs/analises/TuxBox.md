# TuxBox - analise de DeepSeek V4.1 Flash

## A. Identificacao

| Campo | Valor | Evidencia |
| --- | --- | --- |
| Nome | TuxBox | `README.md:1` |
| URL | https://github.com/brcly/TuxBox | `git remote -v` (origin) |
| Fork ou original | **Original** (`"fork": false` na API), criado `2026-11-21T12:07:28Z` | `GET /repos/brcly/TuxBox` |
| Licenca | Apache-2.0 (`LICENSE`, commit `1b6fbb1`) | API `license.spdx_id` |
| Popularidade | 372 estrelas, 40 forks, 21 issues abertas (leitura 2026-11-25) | API; o prompt citou 371 - numero ja mudou |
| Branch analisada | `main`, HEAD `bd847b5d7c8aa5f3a61fef6015074dad884215dc` | `git rev-parse --abbrev-ref HEAD` |
| Outras refs | `origin/ci/build-guests`, 1 commit nao mergeado: `db5d899` ("ci: build every provider, guest and Lanes") | `git branch -r --no-merged HEAD` |
| Commit 0 | `6d15471b0ebac7e89f4ac3959667a4b6faa7fa60` - 2026-11-21 14:23:06 +0000 - "Initial commit" - 290 arquivos, 28.334 linhas | `git show --stat 6d15471` |
| Ultimo commit | `bd847b5d7c8aa5f3a61fef6015074dad884215dc` - 2026-11-25 22:41:32 +0000 - "launcher: pin GE-Proton11-14 as the default Proton" | `git log -1` |
| Periodo coberto | 2026-11-21 a 2026-11-25 (4 dias 8 h; commits em 5 dias UTC) | `git log --format=%ad` |
| Total de commits | 591 em `main` (592 em todas as refs) | `git rev-list --count HEAD` |
| Merges | 190 (163 de PR + 27 merges locais) | `git log --merges` |
| Arquivos versionados | 1.652 | `git ls-tree -r --name-only HEAD` |
| Tamanho de codigo | 457 arquivos `.rs` (~100.341 linhas), 285 `.py` (~37.982 linhas), 473 JSONs de evidencia, 141 `tools/check_*.py` | contagem direta |
| Autores | 23 nomes distintos; 330 commits `brcly`, 83 `Valdi320`, 50 `Manuel Smith Lazarte Alcocer`, 46 `NathansDomainSolutions`, 10 `Pavel Kirilin`, 9 `Vladimir`, 9 `JohnVonWells`, 8 `Jerkin42`, 7 `Lucas Boyles`... | `git log --format=%an \| uniq -c` |
| Autoria assistida por IA | 116 commits com `Co-Authored-By: Claude`, 11 com "AI-assisted", 10 citando "codex" | `git log --format=%B` |
| Commits examinados em profundidade | 45 (pacote de evidencias) + ~40 abertos individualmente com `git show` (correcoes, reversoes de merge, mudancas de tabela de slots) | ver K |

**Linhagem (CONFIRMADA por documentacao do proprio repo).** TuxBox nao e fork do VIctory. `docs/REFERENCES.md` registra tres antecessores:

1. **drycheetah84/TomBox** - projeto original, "publishes no licence".
2. **brcly/TomBox** `ca6cf6a` - fork usado nas comparacoes anteriores com Frosty; e a origem do emissor HLSL legado que ainda hoje e o caminho padrao (`docs/REFERENCES.md:7`, `docs/PROGRESS.md:36`).
3. **VIctory** (`Marvisak/VIctory` `70564ae`, GPL-3.0-or-later) - usado **apenas como referencia de comparacao**, nao como base de codigo (`docs/REFERENCES.md:12`). E por isso que TuxBox pode ser Apache-2.0.

O repositorio carrega 30 patches `tools/reference/tombox-*.patch` que documentam, diff a diff, o que foi alterado sobre o fork (`docs/REFERENCES.md:41`).

**Modelo responsavel:** DeepSeek V4.1 Flash. **Data da analise:** 2026-11-25.

## B. Resumo tecnico

### Proposito

Camada de compatibilidade de **alto nivel** para jogos Xbox Series X|S desempacotados e ja decifrados, em x86-64 Linux (primeiro alvo) e Windows. A tese arquitetural esta em `docs/ARCHITECTURE.md:3`: o codigo x86-64 do convidado roda nativamente na CPU hospedeira; a intervencao acontece **somente** na fronteira da API de console - chamadas de biblioteca, submissao de comandos de GPU e comportamento do SO de console.

### Camadas

| Camada | Implementacao | Volume |
| --- | --- | --- |
| Graficos | `runtime/d3d12` (provider Rust que exporta `d3d12_xs.dll`, tabelas de slots de metodo do comando/dispositivo/fila Xbox, traducao para D3D12 de desktop) | 187 arquivos, 42.100 linhas |
| Containers de shader | `runtime/dxbc` (leitura de DXBC/XDAT/FSSM/PSV0, `compute_table`, `wave_lane_range`) | 32 arquivos, 6.239 linhas |
| Tradutor de shader | `tools/shader-compiler` (`xshader`, RDNA2 -> HLSL; hlsl.rs + driver/rewrite/emitter/text apos o refactor de 25/11) | 225 arquivos, 41.174 linhas |
| Outras bibliotecas | `runtime/xmem`, `runtime/xg`, `runtime/xg-layout`, `runtime/xgameruntime`, `runtime/xcrdapi`, `runtime/gameinput`, `runtime/acphal`, `runtime/xapu`, `runtime/bvh`, `runtime/dxc` | - |
| Lancador | `project_tuxbox/*.py` + `tuxbox.py` (Python: selecao de Proton, prefixo, drive G:, cache NVIDIA, worker de shaders, `check`/`verify`) | 29 arquivos, 5.743 linhas |
| Jogo de teste | `examples/lanes` (Lanes: replay deterministico, self-test headless, cache de shader, benchmark) | 18 arquivos, 2.702 linhas |
| Evidencia | `evidence/*.json` (473) + `docs/RESULTS.md` + `docs/progress.svg` gerado | - |

### Caminho de shader no Linux

`shader RDNA2 do console` -> `xshader` (HLSL) -> `DXC` (Microsoft, binario Linux ou `dxcompiler.dll` do Windows) -> `dxv` (validacao) -> cache por digest -> `vkd3d-proton` (DXIL -> SPIR-V) -> driver RADV/NVIDIA (`docs/SHADERS.md:3`).

Esse pipeline tem **tres pontos de falha especificos de Linux** que nao existem numa camada Windows-only e que dominam metade do catalogo de defeitos: o compilador da Microsoft como dependencia externa (distro quebrada, timeouts, flags de otimizacao), a conversao DXIL->SPIR-V dentro do vkd3d-proton (tempo e recursao de pilha) e a camada Vulkan/Proton (extensoes, strides de descritor, apresentacao).

### Estado atual (declarado pelo autor)

- **Lanes**: totalmente jogavel; 66 checagens de convidado passam em RTX 4070 Ti SUPER com Proton de stock (`docs/PROGRESS.md:13`, `evidence/lanes-checks-rtx4070tis.json`), junto com os 17 controles de regressao D3D12 (contagem citada na issue #191, que relata 16 de 17 passando em RADV).
- **The Matrix Awakens (Frosty)**: menu, Wake Up e o primeiro corner da caca; 37.026 de 37.031 entradas nao-ray-tracing traduzem (99,99%) e 43.229 de 43.407 no total dos dois arquivos (99,59%) (`docs/SHADERS.md:9,11`).
- **ProjectScarlett (GTA VI 1.0.2.1)**: `Game Running`, frame 32.534, 18 min 11 s, com buffers de apresentacao **3840x2160**; o prologo nao aparece e a execucao termina em assert do jogo por esgotamento de memoria de descritores (`evidence/projectscarlett-gameplay.json`, `evidence/projectscarlett-startup.json`).
- **Nenhum jogo comercial e jogavel** (`README.md:17`).

**Sobre a alegacao "primeiro frame do GTA VI em 4K numa RTX 4070 Ti SUPER":** o que a evidencia sustenta e **apresentacao em 4K**, nao imagem correta verificada. `evidence/projectscarlett-startup.json` registra `present_dimensions: [3840, 2160]`, `swapchain_sources: 2`, 4.618 presents contados pelo MangoHud e uma captura da tela de selecao de conta com controles visiveis ("Account Picker (Y)", "Screen Narration (Off) (X)", "TuxBox (A)") em 4K - commit `c217aa4`, 2026-11-23 22:25 UTC. No mesmo arquivo, `desktop_image: "black with MangoHud after presentation began"` no primeiro instante, e `not_verified` inclui "image correctness" na execucao de gameplay. Classificacao: **CONFIRMADA** para "apresentou buffers 4K"; **NAO VERIFICADA** para "primeiro do mundo" e para "imagem 4K correta e jogavel". A issue aberta #180 ("anyone have actual visuals?", tela preta para um usuario) mostra que a experiencia nao e universal.

**Sobre o irmao `omonkeyz/xbox-emulator`:** o prompt afirma "exatamente os mesmos 1.652 arquivos". Isso e **falso em conteudo**: ambos tem 1.652 arquivos versionados, mas apenas **7 caminhos coincidem** (`.gitignore`, `LICENSE`, `README.md`, `CONTRIBUTING.md`, `Cargo.toml`, `Cargo.lock`, `.github/pull_request_template.md`) - ou seja, os arquivos obrigatorios de qualquer repo. Ver E-10.

## C. Linha do tempo

Datas em UTC. Contagem por dia: 21/11 = 6, 22/11 = 74, 23/11 = 48, 24/11 = 271, 25/11 = 192.

### Fase 0 - Importacao e mapa de cobertura (2026-11-21, 6 commits)

`6d15471` "Initial commit" traz 290 arquivos e 28.334 linhas de uma vez: o runtime derivado do TomBox, reorganizado em `runtime/`, `tools/` e `project_tuxbox/`. O repositorio no GitHub e criado as 12:07:28Z, antes do commit. `151c088` implementa e verifica seis providers parciais; `feee428` passa a rastrear cobertura de export por biblioteca; `260e104` renderiza o codigo do menu do Frosty e mapeia a cobertura de shader; `d5abfc0` restaura os "code trails" e liga shaders raster sob demanda; `5725d77` traduz Nanite mesh e culling.

### Fase 1 - Ray tracing, XG e XMem (2026-11-22, 74 commits)

Dia de levantamento sistematico, quase todo por `brcly`, com subjects no imperativo de medicao ("Validate...", "Measure...", "Survey..."): BVH serializado, sombras, hair, Lumen, layouts de textura XG (1D/3D/mips/tails), XMem (heaps, scatter pages, write-watch), PIX PDB, XTF. `accd7e1` roda a caca de highway e a cidade do Frosty no Linux; `4f91131` traduz shaders simples de forma independente e roda o Frosty em Proton de stock.

### Fase 2 - ProjectScarlett aparece (2026-11-23, 48 commits)

`759701f` expande D3D12/XMem/XG/DXC e verifica o startup do Frosty. `fe6b0e1` leva o ProjectScarlett de "Game Init" a "Game Running" (substituicao de formatos, heaps enderecados, eventos de frame, pipelines nulos). `c217aa4` - "Now displays GTA VI properly on linux." - e o marco de apresentacao 4K, com a evidencia JSON anexada. Primeiros PRs externos entram: `91f775d` (#2, resolve raiz do pacote no `inspect`) e `57fe562` (#3, unittest no Windows). `84c35b3` mapeia o pacote no drive `G:`; `baa315b` conserta o link (`synchronization.lib`) e rejeita DXC quebrado; `d7773e2` adiciona o retry em `-O0` para o DXC.

### Fase 3 - Explosao de contribuidores (2026-11-24, 271 commits)

O dia mais denso: 271 commits e 117 merges, dos quais 95 sao merges de PR (#4 a #115). Entram em cena `NathansDomainSolutions`, `Valdi320`, `Jerkin42`, `s3rius`/`Pavel Kirilin`, `postblink`/`Lucas Boyles`, `ReverendOzmund`, `3kynox`, `chunfeng11221`, entre outros. Temas: paginacao e aliasing de descritores, placement de textura, tabelas de descritor raster, sampler heap, D16, asserts de tile-heap, cache NVIDIA, XWayland (#34/#35), build Linux com Rust/clang atuais (#4/#15), stack de 64 MiB para pipelines (`c679f13`), e a primeira evidencia de dano de merge (`1a50eb9`, `8bd56af`). `72766e7` restringe o desligamento de `VK_EXT_descriptor_buffer` por GPU.

### Fase 4 - Intro do GTA VI, Lanes como gate e refactor (2026-11-25, 192 commits)

- **Marco**: `a93613a` documenta o ProjectScarlett chegando ao story intro; `a30aa2a` registra o mesmo em RTX 3080 Ti; `2852d6f`/`d83f5b1` atualizam a preparacao de shaders.
- **Sincronizacao**: `84c12e6` (esperas no meio da lista em filas compute), `c87c57d`/`78319b5` (cenarios que reproduzem o bug), `6bae710` (reescrita de 898 linhas em `memory_wait.rs`), `4ba74bc` (regressao de direct-copy encontrada pelos novos cenarios), `e8af9fd` (escritas de fence do titulo), `faf8b19` (deadlock por placement atras de label).
- **Infra de teste**: `4ae23ab` cria o self-test de GPU e o comando unico `check`; `52292e6` adiciona cenarios e benchmark; `940ab23` cronometra frames; `a79aedb` torna a passagem no Lanes obrigatoria para merge; `bd8049f` registra 66 checagens; `db5d899` (branch `ci/build-guests`, **nao mergeada**) constroi tudo no CI.
- **Escala**: `98043df` recupera slots de descritor com lease e fence pinning; `2edd4a9`/`6161046` tratam slot 80 e `WaveSize(N)`; `95b242b` reflete roots raster com varias tabelas.
- **Refactor**: `aa42ae9`, `9e491ff`, `9399e4f`, `c8f4b53`, `6f08e06` quebram arquivos gigantes (hlsl.rs de 10.367 linhas vira 4 pastas) e instituem limite de 1.200 linhas por arquivo Rust.
- **Fechamento**: `e9c8fb6` (`-O1 -disable-lifetime-markers` para `cs_6_6`), `1b0dfaa` (repetir Present nos checks), `bd847b5` (fixar GE-Proton11-14 como padrao).

## D. Catalogo de defeitos

Todos os itens abaixo foram lidos no diff, nao apenas na mensagem. "Estado atual" refere-se a `bd847b5`.

---

### D-01 - Estouro de pilha dentro do vkd3d-proton ao criar pipelines

- **Descricao/sintomas**: no startup do ProjectScarlett, 7-8 threads de worker morrem com `virtual_setup_exception stack overflow` dentro do compilador, e o startup trava antes de carregar as configuracoes.
- **Causa raiz**: o vkd3d-proton compila DXIL->SPIR-V **sincronamente na thread chamadora** dentro de `Create*PipelineState`, e sua analise de fluxo de controle recursa fundo em shaders traduzidos grandes. O provider criava pipelines em threads de worker com 1 MiB de pilha.
- **Introducao**: arquitetura original do provider (threads de 1 MiB herdadas do desenho anterior); nao ha um commit unico de introducao - o defeito so se manifesta quando os shaders ficam grandes o bastante, o que ocorre no fim da fase 2.
- **Tentativas**: nenhuma anterior; a correcao foi direta.
- **Correcao definitiva**: `c679f13` (PR #17, `7666f54`) - cada criacao de pipeline roda em uma thread auxiliar com **64 MiB de pilha reservada**, com join. `d10b369` amplia para `run_logged`, que reporta progresso a cada 5/15 s.
- **Arquivos**: `runtime/d3d12/src/big_stack.rs`, `runtime/d3d12/src/device/{compute,graphics,stream}.rs`.
- **Evidencia**: `git log -1 --format=%B c679f13` - "In every launch on an RTX 4070 with Proton Experimental 11.0-20261112, 7-8 of those threads hit `virtual_setup_exception stack overflow` inside the compiler (...) With the change, the same launches report no stack overflows".
- **Estado atual**: corrigido e coberto por teste (`deep_recursion_fits_on_the_helper_stack`, que empilha ~8 MiB de frames).
- **Confianca**: CONFIRMADA.

### D-02 - Otimizador do DXC estoura o tempo; shaders viram pipeline nulo

- **Descricao/sintomas**: programas compute de skinning/edge-emit do ProjectScarlett levam 94-300+ s no otimizador do DXC, estouram o limite de 50 s e o jogo recebe pipeline nulo (nada e desenhado).
- **Causa raiz**: custo de otimizacao do DXC cresce superlinearmente com o tamanho do HLSL gerado; o tradutor emite programas de milhares de instrucoes para um unico shader.
- **Introducao**: limite fixo de 50 s no lancador (`project_tuxbox/runtime_shaders.py`, estado anterior a `d7773e2`).
- **Tentativas, em ordem**:
  1. `d7773e2` - em `TimeoutExpired`, recompilar com `-O0` e limite de 120 s.
  2. `444368a` - forcar `cs_6_6` direto em `-O0` e **baixar todos os limites para 15 s** (ver H-01: isso cria uma regressao).
  3. `2d887b3` - `O0_TIMEOUT = 90`, porque `CS_Emit_Dynamic_Model_Edge` (7.813 instrucoes, 859 KB de HLSL) precisa de ~18 s em `-O0` e era recusado "em todo boot".
  4. `e9c8fb6` - para `cs_6_6`, usar `-O1 -disable-lifetime-markers` com `COMPUTE_TIMEOUT = 60`, mantendo `-O0` como fallback.
- **Correcao definitiva**: `e9c8fb6` (parcial, ver D-03).
- **Arquivos**: `project_tuxbox/runtime_shaders.py`, `project_tuxbox/dxil_store.py`, `tests/test_runtime_shader_links.py`.
- **Evidencia**: `git show 2d887b3` (comentario `O0_TIMEOUT`); `git show e9c8fb6` - "DXC took 4.8 s instead of 8.3 s, the converter 30 s instead of 364 s, and the block count fell from 34,035 to 8,524".
- **Estado atual**: funciona; a escolha de flags entrou na chave de cache (`VARIANTS`) e no fingerprint do compilador.
- **Confianca**: CONFIRMADA.

### D-03 - Conversao DXIL->SPIR-V leva minutos em `cs_6_6` compilado em `-O0`

- **Descricao/sintomas**: mesmo com DXC rapido, o carregamento trava; um programa levava **364 s** dentro do conversor.
- **Causa raiz**: DXIL sem otimizacao explode em blocos (34.035), e o passe de fluxo de controle do vkd3d-proton e sensivel a isso. Com otimizacao padrao, o DXC gasta o tempo em *Jump Threading*, alimentado por *lifetime markers*.
- **Introducao**: consequencia direta de `444368a` (forcar `-O0` para `cs_6_6`).
- **Tentativas**: nenhuma antes de `e9c8fb6`; a medicao que separa o custo do DXC do custo do conversor aparece pela primeira vez na mensagem de `e9c8fb6`.
- **Correcao definitiva**: `e9c8fb6` - `-O1 -disable-lifetime-markers`; blocos caem de 34.035 para 8.524 e o conversor de 364 s para 30 s.
- **Arquivos**: `project_tuxbox/runtime_shaders.py`, `project_tuxbox/dxil_store.py`.
- **Estado atual**: corrigido; `PROGRESS.md` ainda lista "grandes shaders compute levam ate quase uma hora" como prioridade, o que indica casos residuais.
- **Confianca**: CONFIRMADA.

### D-04 - Cache de pipeline da NVIDIA no lugar errado e com teto de tamanho

- **Descricao/sintomas**: o vkd3d reusa seu SPIR-V, mas o driver recompila os mesmos pipelines a cada boot.
- **Causa raiz**: por padrao a NVIDIA grava em `~/.cache/nvidia/GLCache`, um bucket **compartilhado por todo processo Wine** e limitado a ~128 MB; um unico boot do Scarlett escreve mais que isso e o driver descarta o bucket. Pior: com `__GL_SHADER_DISK_CACHE_PATH` definido, o driver le `<path>/GLCache/<device>/<app>`, e um bucket copiado para `<path>/<device>` **nunca e aberto**.
- **Introducao**: `9b18f7c` (aponta o caminho para o prefixo) - o caminho novo tornou inutil um bucket pre-existente copiado ao lado.
- **Tentativas**: `d10b369` (adota o bucket mal posicionado, movendo para `GLCache/<device>/<app>` e preservando arquivos ja corretos); `d439070` (aplica tamanho e `SKIP_CLEANUP` tambem quando o caminho vem do ambiente).
- **Correcao definitiva**: `d10b369` + `d439070`.
- **Arquivos**: `project_tuxbox/host.py` (`configure_nvidia_shader_cache`, `adopt_nvidia_glcache`), `tests/test_host.py`.
- **Evidencia**: docstring de `d10b369`; `9b18f7c` define `__GL_SHADER_DISK_CACHE_SIZE` em 8 GiB.
- **Estado atual**: corrigido; o codigo trata explicitamente "arquivo ja em GLCache pertence a este caminho".
- **Confianca**: CONFIRMADA para o layout e o teto; PROVAVEL para quem copiou o bucket (a origem da copia mal posicionada nao e identificada no repo).

### D-05 - Varredura O(n) case-insensitive do Wine na cache de shaders

- **Descricao/sintomas**: com 39.463 arquivos na cache do ProjectScarlett e 13.812 copias na sessao, as threads de carregamento passavam ate 80% do tempo ocupadas, tres quartos em kernel, sem leitura de disco util. O trecho "sair da primeira tela ate o streaming" levava 3 min 45 s.
- **Causa raiz**: o Wine casa nomes de arquivo **sem diferenciar maiusculas**. Abrir um nome ausente faz o `readdir` da pasta inteira procurando outra grafia. Cada requisicao de shader pedia varios nomes ausentes (resposta da sessao duas vezes, `.sha256`, `.unsupported` e o destino do rename).
- **Introducao**: desenho original de `device/shader_cache/compile.rs` (anterior a `4b4c08c`).
- **Tentativas**: nenhuma anterior; o defeito foi localizado com `perf` antes de qualquer mudanca.
- **Correcao definitiva**: `4b4c08c` (PR #165, merge `5dd0d84`) - listar cada diretorio **uma vez** em um `HashSet` (`Names`), abrir so nomes conhecidos, e ler direto da cache em vez de copiar para a sessao.
- **Arquivos**: `runtime/d3d12/src/device/shader_cache/compile.rs`.
- **Evidencia**: `git log -1 --format=%B 4b4c08c` - 3 min 45 s -> 1 min 16 s; tempo de kernel de ~360% para ~170% de um core; `perf` mostrando `utf8_mbstowcs`, `hash_short_file_name`, `readdir64`. O proprio autor ressalva que a segunda execucao tinha cache maior e cache de driver, "so not all of the difference need be this change".
- **Estado atual**: corrigido.
- **Confianca**: CONFIRMADA para o mecanismo; PROVAVEL para a magnitude do ganho.

### D-06 - Present unico nao chega a tela no GE-Proton11-14

- **Descricao/sintomas**: nas checagens de apresentacao, a janela fica inalterada pelos 4 s inteiros apos **uma** chamada `PresentX`, embora o frame exista.
- **Causa raiz**: comportamento do backend GE-Proton11-14 - uma imagem apresentada pode so aparecer em um present posterior. Um titulo real apresenta a cada frame, e por isso nunca ve o problema; a checagem apresentava uma vez so.
- **Introducao**: nao e defeito do TuxBox; e incompatibilidade entre a premissa do teste e o backend.
- **Tentativas**: nenhuma no codigo do provider; a unica mudanca foi no proprio teste.
- **Correcao definitiva**: `1b0dfaa` - repetir o mesmo present ate 3 vezes, a cada 250 ms, mantendo a falha quando nenhum frame chega.
- **Arquivos**: `runtime/d3d12/frame_present.cpp`.
- **Evidencia**: `git show 1b0dfaa`; mensagem registra "All five presentation checks passed three times in a row on GE-Proton11-14 and once on Proton Experimental".
- **Estado atual**: corrigido; e a base para a fixacao do GE-Proton11-14 (`bd847b5`).
- **Confianca**: CONFIRMADA.

### D-07 - Amostragem de tela falha em XWayland rootless (`BadMatch`)

- **Descricao/sintomas**: em KDE Plasma/GNOME Wayland, todas as checagens `guest-present*` falham com cores `0/0/0/0/0` mesmo com os frames visiveis. Issue #34 (fechada).
- **Causa raiz**: o Wine implementa `GetDC(nullptr)`/`GetPixel` sobre a janela raiz do X. O XWayland rootless recusa `GetImage` nessa janela com `BadMatch`, logo toda amostra le zero.
- **Introducao**: dependencia do Wine/XWayland, nao de codigo do repo; o teste assumia um X com raiz legivel.
- **Tentativas**: `1de4d36` (detecta e reporta a causa em vez de amostrar preto); `00200ba` (sobe um **Xwayland rootful privado** com `-displayfd`, cookie MIT-MAGIC-COOKIE-1 em arquivo 0600, geometria 2560x1440, e desliga ao sair); `1b9df5d` (corrige **colisao de numero de display** e limpeza de processos - ver H-06).
- **Correcao definitiva**: `00200ba` + `1b9df5d`.
- **Arquivos**: `project_tuxbox/verify.py` (`screen_capture_problem`, `private_x_server`, `_free_display_number`), `project_tuxbox/host.py` (`_descendant_pids`, `Host.run` com `start_new_session` e kill da arvore).
- **Evidencia**: `git show 1de4d36` e `00200ba`; `git show 1b9df5d` (leitura de `/proc/<pid>/stat` para matar descendentes de `umu-run`/`proton`).
- **Estado atual**: corrigido; o relatorio registra `present_display` quando o caminho privado e usado.
- **Confianca**: CONFIRMADA.

### D-08 - Pacote DXC da distribuicao (Arch) aborta em todo shader

- **Descricao/sintomas**: no Arch, `directx-shader-compiler 1.10.2605.37` aborta com assercao `LLVM HungOffUses` (SIGILL) mesmo em um shader de uma linha. O build falha de forma opaca. Issue #4 (fechada).
- **Causa raiz**: build de distro do DXC defeituoso; o projeto so testava o release da Microsoft.
- **Introducao**: dependencia externa de binario empacotado pela distribuicao (nao ha commit que introduza; e o estado inicial do build em Linux).
- **Tentativas**: nenhuma anterior; a correcao e a propria deteccao.
- **Correcao definitiva**: `baa315b` - compilar um shader minimo com o DXC selecionado **antes** de construir e falhar com mensagem que nomeia o release da Microsoft; documentar tambem os headers Vulkan que a camada CBV precisa.
- **Arquivos**: `build.py`, `examples/lanes/README.md`.
- **Evidencia**: `git log -1 --format=%B baa315b` ("Arch's directx-shader-compiler 1.10.2605.37 aborts with an LLVM HungOffUses assertion (SIGILL) on every shader"), `Fixes #4`; issue #4 confirmada como fechada na API.
- **Estado atual**: corrigido; `--windows-dxc` e o caminho recomendado.
- **Confianca**: CONFIRMADA.

### D-09 - Toolchain Linux: `synchronization.lib` ausente e `clang 19` com `/W4 /WX`

- **Descricao/sintomas**: (a) link de `tuxbox_xhttp.dll` falha com simbolos indefinidos; (b) `clang 19` adiciona `-Wcast-function-type-mismatch` a `/W4` e o `/WX` rejeita os casts `GetProcAddress` -> ponteiro tipado; (c) toolchain antigo quebra dentro de `tuxbox-xmem` com E0658 ("unstable library feature").
- **Causa raiz**: o std do Rust 1.100 para `x86_64-pc-windows-msvc` passou a importar `WaitOnAddress`/`WakeByAddress*`, que exigem `synchronization.lib`; e o codigo usa `LazyLock::get` (1.94) e `try_update` (1.95).
- **Introducao**: atualizacao do toolchain do host (Rust e clang), nao do repositorio - o mesmo codigo compilava em versoes anteriores.
- **Tentativas**: `baa315b` (adiciona o lib a `RUST_STD_LIBS`), `ca772f1` (desliga o warning so para `clang-cl`, depois dos flags por arquivo), `04641a6` (`rust-version = "1.95"` no workspace).
- **Correcao definitiva**: os tres, em conjunto.
- **Arquivos**: `build.py`, `Cargo.toml`, `examples/lanes/README.md`.
- **Evidencia**: mensagens de `ca772f1` e `04641a6`; CI fixa `dtolnay/rust-toolchain@1.95.0`.
- **Estado atual**: corrigido; CI tambem documenta que `cargo test --workspace` no Linux nao constroi porque `windows-future` procura `IMarshal` no alvo Linux (`.github/workflows/ci.yml`).
- **Confianca**: CONFIRMADA.

### D-10 - Jogo nao abre arquivos do pacote sem o drive `G:`

- **Descricao/sintomas**: o ProjectScarlett 1.0.2.1 verifica `appdata0.rpf` no System Init; sem `G:` a lista de perfis de hardware fica vazia e o titulo para em `int3` (RVA `0x2237178`) ~3 s depois, seguido de upload de minidump.
- **Causa raiz**: titulos Xbox abrem o pacote por `G:`, como no console; um prefixo Wine nao tem esse drive.
- **Introducao**: ausencia de mapeamento desde o inicio.
- **Tentativas**: `84c35b3` (mapeia via mount manager do Wine ao lado de `T:`); `c561d03` (raiz do pacote vem do diretorio lancado, aceita `microsoft_gameconfig.bin`, e **grava/restaura** os links `g:`/`g::` do prefixo); `22caa19` (restaura tambem quando o setup falha); `23c848b` (passa o diretorio do pacote pelo caminho de launch mergeado).
- **Correcao definitiva**: `c561d03` + `22caa19`.
- **Arquivos**: `project_tuxbox/host.py` (`claim_package_drive`, `restore_package_drive`), `runtime/xgameruntime/src/title_temp.rs`.
- **Evidencia**: `git show 84c35b3` (RVA e sintoma), `git show c561d03` (registro `tuxbox-package-drive.json`), teste `test_linux_package_drive_record_is_removed_when_setup_fails`.
- **Estado atual**: corrigido.
- **Confianca**: CONFIRMADA.

### D-11 - Desligar `VK_EXT_descriptor_buffer` em todas as GPUs corrompe heap no RADV

- **Descricao/sintomas**: no RX 9060 XT (RADV), o ProjectScarlett aborta com corrupcao de heap em **5 de 6** execucoes e loga violacoes de acesso RADV na sexta.
- **Causa raiz**: o contorno de alinhamento de constant buffer (para GPUs cujo `minUniformBufferOffsetAlignment > 16`) desabilitava `VK_EXT_descriptor_buffer` **globalmente**, forcando o caminho de descriptor set em GPUs que nao precisavam dele.
- **Introducao**: contorno de CBV do console (`project_tuxbox/console_cbv.py`, estado anterior a `72766e7`).
- **Tentativas**: `9b18f7c` e o proprio contorno original usavam variavel de ambiente global; a correcao muda a decisao para dentro da camada Vulkan.
- **Correcao definitiva**: `72766e7` (PR #42, merge `5f777d6`) - a camada Vulkan passa a esconder a extensao **apenas** quando `minUniformBufferOffsetAlignment > 16 && minStorageBufferOffsetAlignment <= 16`; o fork de Proton mantem o override por ambiente.
- **Arquivos**: `project_tuxbox/console_cbv.py`, `tools/reference/cbv-layer/layer.c`, `tests/test_host.py`.
- **Evidencia**: `git show 72766e7` - "Six launches with the extension enabled had neither".
- **Estado atual**: corrigido.
- **Confianca**: CONFIRMADA.

### D-12 - Stride de descritor inline incompativel com o RADV (32 bytes)

- **Descricao/sintomas**: pipelines raster falham/leem descritores errados em RADV; o controle `buffer-remap` falha e importacoes de memoria do host sao recusadas (issue #191, aberta).
- **Causa raiz**: o RADV reporta incremento de descritor **maior** que o esperado pelo convidado, e o provider usava esse incremento para calcular offsets de arenas inline do `Device11`.
- **Introducao**: desenho das arenas inline (`device/descriptor_paging`).
- **Tentativas**: nenhuma anterior; a divergencia ficou visivel quando o RADV entrou na matriz de testes (evidencias `cachyos-rx6900xt`, `nixos-rx9060xt`).
- **Correcao definitiva**: `cf427e8` (PR #184, merge `f489d3c`) - `guest_increment_size` devolve 32 bytes (`INLINE_BYTES`) quando o dispositivo e `device11_queue`, o tipo e CBV_SRV_UAV e o incremento nativo e multiplo de 32; todas as buscas/copias passam a usar `arena.guest_stride()`.
- **Arquivos**: `runtime/d3d12/src/device.rs`, `device/descriptor_copy.rs`, `device/descriptor_paging.rs`, `device/descriptor_paging/lookup.rs`, `commands/descriptors.rs`.
- **Evidencia**: diff de `cf427e8`; tests em `runtime/d3d12/src/tests/device/descriptor_paging.rs`.
- **Estado atual**: corrigido para o caso inline. **Aberto**: issue #191 (`buffer-remap` falha em RX 6700 XT / Mesa 26.3.1 RADV, com `vkd3d_import_host_memory: Failed to import host memory, hr 0x8007000e`), reproduzido 131 vezes em 300 s no ProjectScarlett; ha PRs #199/#200 propondo contorno por shadow heaps anonimos.
- **Confianca**: CONFIRMADA para o stride; CONFIRMADA para a issue #191 estar aberta.

### D-13 - Slot 62 interpretado como upload de CPU em vez de `CopyMemoryToMemoryX(dst, src)`

- **Descricao/sintomas**: a leitura de textura do ProjectScarlett sobrescrevia a textura com o staging buffer vazio - a copia ia na direcao errada.
- **Causa raiz**: o slot 62 foi mapeado como "Scarlett video upload" (`linear_upload`), que lia bytes de CPU na gravacao e os copiava no `ExecuteCommandLists`; a assinatura real e `CopyMemoryToMemoryX(Destination, Source, SizeBytes, Flags)`, e o provider tratava o **primeiro** argumento como origem de CPU e escrevia no segundo.
- **Introducao**: `3086e0b` (cria `commands/linear_upload.rs`, 500 linhas, e registra `slots[62] = linear_upload`).
- **Tentativas**: nenhuma intermediaria; a correcao substituiu o modulo.
- **Correcao definitiva**: `97b8ad3` - `mod linear_upload` removido, `commands/copy_memory.rs` no lugar, copia executada **na GPU** entre os dois enderecos, inclusive textura->memoria; tambem instrumenta enderecos de espera, markers e contagem de frames.
- **Arquivos**: `runtime/d3d12/src/commands.rs`, `commands/copy_memory.rs` (antes `linear_upload.rs`), `commands/standard.rs`, `commands/memory_wait.rs`, `device/views.rs`.
- **Evidencia**: `git show 97b8ad3` (remove `uploads`, `snapshot_uploads`, `adopt_bundle`); diff de `3086e0b` (500 linhas de upload de CPU).
- **Estado atual**: corrigido. O cenario Lanes `copy-memory` e `texture-readback` passaram a cobrir o caminho.
- **Confianca**: CONFIRMADA.

### D-14 - Tabela de slots da lista Scarlett deslocada (61/62/131/132/133)

- **Descricao/sintomas**: o titulo chama slot 61 com assinatura de um comando e aborta em slot nao mapeado; `SyncTopOfPipeX`/`SyncBottomOfPipeX` (131/132) e o build agrupado (133) estavam trocados; o builder de BVH chamava **copy** no ponto de entrada de **build**, de modo que nenhuma estrutura era copiada.
- **Causa raiz**: duas geracoes de Xbox (Frosty e Scarlett) usam tabelas de metodo diferentes depois de `EndEvent`; a tabela herdada tinha apenas o layout antigo.
- **Introducao**: `LIST_SLOTS`/`DEVICE11_LIST_SLOTS` originais.
- **Tentativas**: `bedf503` (move `ExecuteIndirect` para 61, adiciona `DriverHintX` em 101); `e8af9fd` (131/132 viram `SyncTopOfPipeX`/`SyncBottomOfPipeX`, build agrupado vai para 133); `047de82` (BVH chama build e copy nos slots corretos).
- **Correcao definitiva**: os tres, em sequencia.
- **Arquivos**: `runtime/d3d12/src/commands.rs`, `commands/build_raytracing.rs`, `runtime/bvh/src/lib.rs`.
- **Evidencia**: mensagens de `bedf503`, `e8af9fd`, `047de82`.
- **Estado atual**: corrigido.
- **Confianca**: CONFIRMADA.

### D-15 - Escritas de fence do titulo perdidas e um comando derrubando a lista inteira

- **Descricao/sintomas**: as filas compute/grafica esperam labels de fence; as escritas se perdiam, as esperas travavam e o watchdog de GPU do titulo derrubava o jogo.
- **Causa raiz**: (a) `Write64BitValueBottomOfPipeX` (slot 72) era tratado como escrita de **timestamp**, nao de valor arbitrario, e rejeitava o argumento de flush; (b) qualquer comando nao traduzivel colocava a lista em estado de erro, descartando **todo** o resto - inclusive as escritas de fence que outras filas esperavam.
- **Introducao**: desenho original de `write_marker`/`Recording.error`.
- **Tentativas**: `e8af9fd` introduz `Recording::skip`, que descarta so o comando e loga; `faf8b19` acrescenta diagnostico de espera travada (lista que escreve o label, primeira operacao nao passada por fila).
- **Correcao definitiva**: `e8af9fd` (+ diagnostico em `faf8b19`).
- **Arquivos**: `runtime/d3d12/src/commands.rs`, `commands/memory_wait.rs`, `commands/build_raytracing.rs`, `commands/queries.rs`.
- **Evidencia**: `git show e8af9fd`; cenario Lanes `skipped-command` (introduzido em `4ae23ab`) falha sem essa correcao, segundo a mensagem de `4ae23ab`.
- **Estado atual**: corrigido, mas os stalls de fence no intro **persistem** (`docs/PROGRESS.md:15,24`) - a liberacao e por timeout de 5 s, nao por correcao.
- **Confianca**: CONFIRMADA para a correcao; CONFIRMADA para a persistencia dos stalls.

### D-16 - Deadlock entre filas por espera de placement atras de um label

- **Descricao/sintomas**: a fila grafica esperava um lote compute que esperava um label que a propria fila grafica escrevia; travamento mutuo.
- **Causa raiz**: o posicionamento de textura fazia uma fila esperar por trabalho que estava **atras** de uma espera de label do titulo.
- **Introducao**: logica de `device/placement.rs` anterior a `faf8b19`.
- **Tentativas**: `faf8b19` (placement nao espera trabalho atras de label + diagnostico de espera travada) e, depois, `84c12e6`/`6bae710` (segmentar a lista em esperas e reproduzir estado).
- **Correcao definitiva**: `faf8b19` - placement nao espera mais trabalho atras de label; `Waits::new` rastreia esperas detidas; BVH passa a registrar cada build que consegue e a pular o que nao expressa.
- **Arquivos**: `runtime/d3d12/src/commands.rs`, `device/placement.rs`, `commands/build_raytracing.rs`, `commands/queue_trace.rs`.
- **Evidencia**: `git show faf8b19` - "ProjectScarlett still stalls at the story intro" (o autor declara que a correcao nao fecha o sintoma).
- **Estado atual**: **parcial**. A cadeia continua em `84c12e6` -> `6bae710` (reescrita de `memory_wait.rs`, 898 linhas) -> regressao `4ba74bc` -> `e4783b4`. Os stalls permanecem.
- **Confianca**: CONFIRMADA.

### D-17 - Liberacao imediata do backing de descritor causa falha de MMU na GPU

- **Descricao/sintomas**: um warp SM dispara falha de MMU (Xid 13, classe compute, mesmo PC em todas as execucoes) quando um descritor e substituido enquanto um dispatch ainda o referencia.
- **Causa raiz**: a escrita de descritor vale imediatamente na CPU, mas listas ja submetidas continuam usando o descritor antigo; o backing antigo (buffer importado ou textura posicionada) era liberado **na hora**, sob o dispatch em execucao.
- **Introducao**: `DescriptorMap::store`/`copy` originais.
- **Tentativas**: `e4783b4` - aposentar valores substituidos em fila e descartar **depois de 3 s** (solucao por tempo, nao por fence).
- **Correcao definitiva**: `98043df` (PR #172, merge `9ca0060`) - recuperacao de slots com *lease* e *fence pinning* (1.526 linhas em `descriptor_paging.rs` e `index_map.rs`).
- **Arquivos**: `runtime/d3d12/src/device/descriptor_paging.rs`, `descriptor_paging/index_map.rs`, `device/placement/descriptors.rs`, `commands/compute_bindings.rs`.
- **Evidencia**: `git show e4783b4` (Xid 13); `git show 98043df --stat`.
- **Estado atual**: **parcial**. `docs/PROGRESS.md:25` mantem "Recuperar slots de descritor visiveis ao shader, que nunca sao liberados e acabam em execucoes longas" como prioridade 2, e o registro de gameplay termina com `"command_list_rejected": "descriptor memory exhausted"`.
- **Confianca**: CONFIRMADA para o defeito e para o lease; CONFIRMADA para o esgotamento remanescente.

### D-18 - Aliasing de textura posicionada derrubando a fila, e o contorno que matou o watchdog

- **Descricao/sintomas**: duas imagens posicionadas cobrindo a mesma memoria do convidado faziam `placement::submit` devolver `E_NOTIMPL`, envenenando a fila direta; `PresentX` devolvia o erro guardado e o titulo entrava em crash deliberado.
- **Causa raiz**: o provider exigia ordenacao que uma unica submissao nao pode garantir entre duas imagens sobre a mesma memoria.
- **Introducao**: `device/placement.rs` anterior a `cb185e0`.
- **Tentativas**: `cb185e0` - degradar em vez de falhar: imagem que sobrepoe escrita anterior espera ate **2 s** na CPU e recarrega; escritor sobreposto no mesmo lote e logado sem garantia.
- **Regressao**: `524cd21` - a espera de 2 s na CPU **nunca** terminava, porque a escrita podia estar atras de trabalho que espera um fence sinalizado so depois do submit; travava a thread de render e o watchdog matava o titulo. Trocado por refresh adiado para um lote posterior, sem espera de CPU.
- **Correcao definitiva**: `524cd21` (com testes em `eac38ec`).
- **Arquivos**: `runtime/d3d12/src/device/placement.rs`, `device/placement/upload.rs`.
- **Evidencia**: mensagens de `cb185e0` e `524cd21`.
- **Estado atual**: corrigido.
- **Confianca**: CONFIRMADA.

### D-19 - Asserts de tile-heap remendados por RVA e bytes

- **Descricao/sintomas**: o titulo dispara asserts internos ao pedir mapeamentos menores que a run de paginas marcada, e um assert de iterador fim na caminhada da tabela de runs.
- **Causa raiz**: o titulo pergunta por intervalos **alem** da sua tabela de runs; o provider aceita o pedido, mas o proprio codigo do jogo considera isso impossivel e aborta.
- **Introducao**: divergencia de contrato entre o provider e o runtime do titulo, presente desde a primeira execucao do ProjectScarlett.
- **Tentativas**: `8017bf5` (`0x4ffd879`); `dd0f8b2` (mais dois RVAs, `0x4ffd859` e `0x51d251e`).
- **Correcao definitiva**: SEM CORRECAO - os commits sao explicitamente paliativos por RVA+bytes.
- **Arquivos**: `runtime/d3d12` (modo survey / patch de RVA).
- **Evidencia**: ambas as mensagens explicitam "Both match on RVA and exact bytes, so another build cannot match. They hide the symptom".
- **Estado atual**: **paliativo assumido**. Nao e correcao de causa; o autor declara que esconde o sintoma.
- **Confianca**: CONFIRMADA (o proprio autor classifica como paliativo).

### D-20 - Instrucoes exclusivas de AMD em CPUs Intel

- **Descricao/sintomas**: o ProjectScarlett 1.0.2.1 usa `clzero`, `monitorx`, `mwaitx` e `insertq` (Zen 2); em Intel cada uma levanta `EXCEPTION_ILLEGAL_INSTRUCTION` e o titulo sai com codigo 29.
- **Causa raiz**: o jogo foi compilado assumindo a CPU do console; o host Intel nao implementa essas instrucoes.
- **Introducao**: codigo do titulo, nao do repositorio; o provider precisou compensar no host.
- **Tentativas**: `e1262ac` (clzero via handler vetorizado, limpando a linha de 64 bytes e, quando o laco e reconhecido, a faixa inteira; `monitorx`/`mwaitx` -> `NOP` reescrevendo o segundo byte do opcode para `1F`); `371c839` (`clwb` -> `clflushopt` em CPUs anteriores a Ice Lake); `7d281f8` (`insertq`).
- **Correcao definitiva**: os tres (PRs #146, #153, #80).
- **Arquivos**: `runtime/d3d12/src/clzero.rs`, `mwaitx.rs`, `clwb.rs`, `survey.rs`, `lib.rs`.
- **Evidencia**: `git show e1262ac` (descricao completa das sequencias e do reaproveitamento do laco).
- **Estado atual**: corrigido para os quatro casos; nao ha garantia de que outros opcodes AMD-only nao aparecam.
- **Confianca**: CONFIRMADA.

### D-21 - Assinatura mesh/pixel incompativel derruba o draw

- **Descricao/sintomas**: drivers de desktop rejeitam pipelines mesh/pixel cujas assinaturas divergem em registrador, mascara ou tipo de componente (`vkd3d: "Input signature element ... not compatible with ..."`); o slot 86 devolve `E_INVALIDARG` e o draw se perde. Mesh e o caminho principal de geometria do titulo (3.240 pedidos contra 2.114 de vertex).
- **Causa raiz**: o programa mesh nao era retraduzido contra o layout de entrada do **consumidor pixel** especifico, como ja se fazia para VS/GS.
- **Introducao**: o caminho de link existia para VS/GS desde a fase 2, mas nao para mesh (que virou caminho principal de geometria no ProjectScarlett).
- **Tentativas**: `6a2ca92` (reconciliar tipos de componente do consumidor pixel) e `9838454` (exercitar o link portavel pelo worker do slot 86) preparam o terreno para `315b287`.
- **Correcao definitiva**: `315b287` (PR #150, merge `53bb4d6`) - em `E_INVALIDARG`, submeter requisicao `.link` com o mesh como produtor; `xshader` ajusta tipos de componente e ordem de registradores; `runtime_shaders.py` passa a aceitar `ms_6_5/ms_6_6` em requisicoes linkadas.
- **Arquivos**: `runtime/d3d12/src/device/stream.rs`, `device/shader_cache.rs`, `tools/shader-compiler/src/hlsl/mesh.rs`, `project_tuxbox/runtime_shaders.py`.
- **Estado atual**: corrigido (com testes `9838454`, `6a2ca92`).
- **Confianca**: CONFIRMADA.

### D-22 - `Failed to remap` (-3) em roots raster com varias tabelas de descritor

- **Descricao/sintomas**: o ProjectScarlett cria a maioria dos pipelines raster com roots do tipo `[table, table, constants]`; o provider aceitava apenas `[table]` e `[table, constants]`, mantinha o root do convidado com traducoes "root-less" e o vkd3d-proton recusava o estagio com `Failed to remap`, resultado -3.
- **Causa raiz**: `reflected()` so reconhecia dois formatos de root raster; os demais caiam no caminho sem reflexao.
- **Introducao**: `reflected()` original, escrito para os roots mais simples vistos no Frosty.
- **Tentativas**: nenhuma anterior; a cadeia `a2e5f35` (restaurar static samplers omitidos por root multi-tabela) e `4b56c70` (parametro de tabela de vertex shader) cobre casos vizinhos depois.
- **Correcao definitiva**: `95b242b` (PR #168, merge `de47600`) - le tabelas iniciais multiplas (registro FSSM nomeia a tabela na metade alta da localizacao, tabela N em `s8 + N`), resolve root SRV/UAV, faz `merge_raster_bindings` e cai no comportamento antigo quando os estagios divergem.
- **Arquivos**: `runtime/dxbc/src/compute_table.rs`, `runtime/d3d12/src/device/compute_table.rs`, `device/stream.rs`.
- **Estado atual**: corrigido, com trace opt-in `TUXBOX_TRACE_RASTER_ROOTS=1`.
- **Confianca**: CONFIRMADA.

### D-23 - `clang 19` vetoriza errado o clear RGB10 e a sonda de present escreve em ponteiro

- **Descricao/sintomas**: (a) toda checagem de present do convidado crashava com `0xC0000005` **depois** do `E_NOTIMPL` esperado; (b) o clear RGB10 chegava como `1/255` e lia `0x00401000` em vez de `0xfffffc00`.
- **Causa raiz**: (a) a sonda de "plane option nao suportada" gravava `1` em `plane.options[0]`, que no plano retail de 72 bytes e o ponteiro do retangulo de origem - o provider desreferenciava; (b) `clang 19.1 -O2` vetorizava o laco byte->float em mascaras 0/1 e perdia a escala 255.
- **Introducao**: (a) codigo de teste desde a fase 1; (b) atualizacao do clang no host (Bazzite/Fedora 41), nao do repositorio.
- **Tentativas**: nenhuma anterior em (b) - o defeito so aparece com clang 19; em (a) o `E_NOTIMPL` era esperado e o crash aparecia depois dele, atrasando o diagnostico.
- **Correcao definitiva**: `40e1565` (PR #18) - sondar o espaco de cor no byte 40; normalizar as cores em tempo de compilacao.
- **Arquivos**: `runtime/d3d12/frame_present.cpp`, `examples/lanes/screen.cpp` (guests de clear).
- **Evidencia**: `git show 40e1565`, testado em Bazzite/Fedora 41 com `clang 19.1.7`.
- **Estado atual**: corrigido; o caso (b) e um exemplo de defeito dependente de versao de compilador do host.
- **Confianca**: CONFIRMADA.

### D-24 - Resolucao de merge apagou lowering e quebrou o verificador

- **Descricao/sintomas**: (a) `s_flbit_i32_b64`, `s_ff0_i32_b32` e `s_ff0_i32_b64` voltaram a ser nao suportados, com helpers "unused"; (b) `tools/check_d3d12_regressions.py` ficou com `IndentationError` e `SyntaxError`, impedindo importar o teste.
- **Causa raiz**: resolucao de conflito manual: em (a) o merge do PR #58 removeu os `match arms` do PR #56 e manteve os helpers; em (b) o merge manteve linhas dos **dois** lados de dois conflitos (perdeu `return` e duplicou a ultima linha do dicionario de ambiente).
- **Introducao**: merges de 2026-11-24 (`1a50eb9` cita o merge do #58; `8bd56af` cita o merge de `main` no branch de regressao).
- **Tentativas**: nenhuma automatica; ambos foram encontrados por inspecao/execucao do teste, nao por CI.
- **Correcao definitiva**: `1a50eb9` (restaura os arms exatamente como #56) e `8bd56af` (restaura o `return` e unifica o dicionario).
- **Arquivos**: `tools/shader-compiler/src/hlsl.rs` (depois dividido em `hlsl/`), `tools/check_d3d12_regressions.py`.
- **Evidencia**: mensagens de `1a50eb9` e `8bd56af`.
- **Estado atual**: corrigido; e a evidencia mais forte de que o fluxo de merges e o ponto fraco do processo (ver H-07).
- **Confianca**: CONFIRMADA.

### D-25 - Profundidade 16 bits com stencil sem plano UAV gravavel

- **Descricao/sintomas**: `D16_UNORM_S8_UINT` e suas views nao tinham plano de profundidade gravavel por UAV; o titulo pedia `R32_FLOAT` para o plano par.
- **Causa raiz**: substituicao de formato para a familia `D24S8` sem ajustar heap/views.
- **Introducao**: `fe6b0e1` (substitui `D16_UNORM_S8_UINT` pela familia `D24S8` e cria imagens de profundidade pedindo UAV como *depth-only*).
- **Tentativas**: `0e00f83` e a propria correcao; a issue vem de o titulo pedir `R32_FLOAT` para o plano par.
- **Correcao definitiva**: `0e00f83` (PR #162, merge `63bec84`).
- **Arquivos**: `runtime/d3d12/src/device/{depth,format,heap,views}.rs`.
- **Evidencia**: diff de `63bec84` (`device/format.rs`, `device/depth.rs`, `device/heap.rs`, `device/views.rs`, 44 insercoes).
- **Estado atual**: corrigido.
- **Confianca**: CONFIRMADA.

### D-26 - `R9G9B9E5_SHAREDEXP` como UAV

- **Descricao/sintomas**: o startup do ProjectScarlett para em `GetResourceAllocationInfo` de 2560x1440, formato 118, com `result_size: UINT64_MAX` (`evidence/projectscarlett-startup.json`, `blocking_call`).
- **Causa raiz**: formato empacotado de console sem caminho de UAV no host.
- **Introducao**: `fe6b0e1` (substituicao de formatos) nao cobria o formato empacotado.
- **Tentativas**: `cda1912` (suporte via formatos *castable* `R32_UINT`); `91a9de2` (correcao de achados de auditoria: interop, gating nativo, buffer UAVs).
- **Correcao definitiva**: `cda1912` + `91a9de2` (PR #183/#188, merges `4cb6a4a`/`b5ed526`).
- **Arquivos**: `runtime/d3d12/src/device/{allocation,heap,reserved,resources,views}.rs`, `commands/barriers.rs`.
- **Evidencia**: mensagens de `cda1912` e `91a9de2`; o registro de startup nomeia o bloqueio.
- **Estado atual**: corrigido.
- **Confianca**: CONFIRMADA para a correcao de R9G9B9E5; **PROVAVEL** para a correspondencia entre o indice de formato 118 do registro e `R9G9B9E5_SHAREDEXP` (nao confirmei a tabela de formatos do ABI Xbox).

### D-27 - Slot 80 trata dimensoes em threads como grupos (issue #164, aberta)

- **Descricao/sintomas**: `dispatch_flagged` (slot 80) com bit `0x20` (`USE_THREAD_DIMENSIONS`) passa **contagem de threads**, nao de grupos; o provider fazia dispatch comum.
- **Causa raiz**: o slot foi registrado lendo os argumentos como grupos e ignorando a palavra de flags.
- **Introducao**: `18e4e08` (PR #156, merge `8ce988d`) - registra slot 80 (`dispatch_flagged`) e slot 148 (list memory) para o `XboxBvhBuilder`, lendo os argumentos como grupos e ignorando a palavra de flags.
- **Tentativas**: `2edd4a9` (PR #170, merge `0eaa8c6`) - le `[numthreads]` do PSO vinculado (PSV0, SHEX, XDAT/FSSM), converte com `div_ceil`, falha fechado sem PSO e loga arredondamento.
- **Correcao definitiva**: `2edd4a9`.
- **Arquivos**: `runtime/dxbc/src/fspx.rs`, `runtime/d3d12/src/commands/compute.rs`, `examples/lanes/selftest.cpp` (cenario `thread-dimensions`).
- **Evidencia**: `git show 2edd4a9` ("Refs #164"); issue #164 confirmada como **aberta** na API.
- **Estado atual**: **parcial** - a issue #164 continua **aberta** em 2026-11-25, embora o commit diga "Refs #164".
- **Confianca**: CONFIRMADA para o commit; PROVAVEL para cobertura completa.

### D-28 - `WaveSize(N)` incompativel recusado so no driver (issue #151, aberta)

- **Descricao/sintomas**: pipelines compute que exigem `[WaveSize(64)]` em GPU so-wave32 eram recusados pelo driver sem razao registrada.
- **Causa raiz**: o tradutor exige wave nativa de 64 lanes quando o caminho portavel de wave unica nao se aplica, mas nada verificava a faixa suportada pelo dispositivo antes de chamar o driver.
- **Introducao**: comportamento do emissor documentado na issue #151 (referencia `tools/shader-compiler/src/hlsl.rs:1583` no relato).
- **Tentativas**: `6161046` (PR #171, merge `3aa9241`) - consulta `D3D12_FEATURE_D3D12_OPTIONS1` uma vez por dispositivo, le a faixa de `PSV0` e recusa **antes** do `CreateComputePipelineState`, registrando `CS.refused=host-lacks-wave<N>`.
- **Correcao definitiva**: `6161046` (recusa honesta); a traducao portavel e o PR #201, nao mergeado.
- **Arquivos**: `runtime/d3d12/src/device/{compute,dxgi,shader_cache,stream}.rs`, `runtime/dxbc/src/fspx.rs`, `runtime/d3d12/src/failures.rs`.
- **Evidencia**: `git show 6161046` (teste `wave_range_compatibility_checks_interval_overlap`); issue #151 **aberta**.
- **Estado atual**: **parcial** - issue #151 **aberta**, com o PR #201 propondo lowering portavel de ballot/vote/exec para Wave64 em GPUs nao-64.
- **Confianca**: CONFIRMADA para a recusa antecipada; CONFIRMADA para a issue #151 estar aberta.

### D-29 - Filas de GPU ainda param em esperas de fence no story intro

- **Descricao/sintomas**: depois de "Game Running", as filas compute e grafica param em esperas de label. O registro da execucao reporta `memory_waits_released_after_5s_at_least: 16` e termina em assert do jogo com `exit_code: 3`.
- **Causa raiz**: nao estabelecida. `docs/PROJECTSCARLETT.md` lista "Fence stalls" e `docs/PROGRESS.md:24` pede que cada causa seja reproduzida antes como cenario Lanes. As hipoteses acumuladas no repo sao: escrita de label perdida (D-15), espera presa atras de placement (D-16), espera no meio da lista em fila compute (`84c12e6`), e dependencia cruzada entre submissoes (`6bae710`).
- **Introducao**: aparece quando o titulo entra no intro (2026-11-25), depois das correcoes de `e8af9fd`/`faf8b19`.
- **Tentativas**: `84c12e6` (esperas no meio da lista em filas compute), `6bae710` (segmentacao e replay de estado, fence persistente, notifier em vez de polling de 1 ms), `4ba74bc`/`78319b5` (cenario `direct-copy-wait` para reproduzir a ordenacao da fila direta). Nenhuma fecha o sintoma.
- **Correcao definitiva**: **SEM CORRECAO**. Mitigacao atual: espera liberada apos 5 s com log das listas que escrevem o label e do ponto onde cada fila parou.
- **Arquivos**: `runtime/d3d12/src/commands/memory_wait.rs`, `commands/recording.rs`, `commands/queue_trace.rs`, `commands.rs`.
- **Evidencia**: `evidence/projectscarlett-gameplay.json` (`memory_waits_released_after_5s_at_least: 16`, `crash_cause_established: false`); `docs/PROGRESS.md:15,24`.
- **Estado atual**: aberto, e a prioridade numero 1 declarada pelo autor.
- **Confianca**: CONFIRMADA (o fenomeno e declarado e medido); a causa raiz e **NAO VERIFICADA**.

## E. Relacao de causa e efeito

**E-01 - Do shader grande ao pipeline nulo (cadeia completa).**
`d7773e2` (retry `-O0`, 120 s) -> programas grandes passam a compilar -> `444368a` forca `-O0` para `cs_6_6` e baixa tudo para 15 s -> **15 s recusa `CS_Emit_Dynamic_Model_Edge` em todo boot** (`2d887b3`) -> sobe para 90 s -> o DXIL `-O0` tem 34.035 blocos -> o conversor do vkd3d-proton leva 364 s -> `e9c8fb6` troca por `-O1 -disable-lifetime-markers` (8.524 blocos, 30 s) -> `bd847b5` fixa GE-Proton11-14 porque seu `dxil-spirv` faz o mesmo em 1,1 s. **Cinco commits, tres produtos diferentes (DXC, vkd3d-proton, GE-Proton) para um unico sintoma "carrega devagar".**

**E-02 - Do present unico ao Proton fixado.**
`1b0dfaa` mostra que o GE-Proton11-14 e o backend onde os cinco checks de apresentacao passam de forma repetivel -> no mesmo dia, `bd847b5` mede que o `dxil-spirv` desse Proton converte um programa grande em 1,1 s contra 36,6 s do Experimental (e < 16 s em tres que o antigo nao terminava em 200 s) -> GE-Proton11-14 vira o Proton padrao, sem fallback silencioso para o Experimental. Um efeito colateral de backend virou decisao de dependencia.

**E-03 - Do descritor substituido ao MMU fault e ao esgotamento.**
`DescriptorMap` libera o backing na escrita -> GPU ainda referencia o descritor antigo -> **Xid 13 / falha de MMU** -> `e4783b4` aposenta por 3 s (tempo fixo) -> `98043df` troca por lease + fence pinning -> `docs/PROGRESS.md:25` ainda pede recuperacao de slots, e o run de gameplay termina com `descriptor memory exhausted`. **Correcao de causa (fence) sem correcao de capacidade (reciclagem).**

**E-04 - Do alias de textura ao watchdog.**
`cb185e0` faz a fila degradar em vez de falhar e espera 2 s na CPU -> a escrita esperada esta atras de um fence sinalizado **depois** do submit -> a espera nunca termina, a thread de render trava, o watchdog do titulo mata o jogo -> `524cd21` remove a espera de CPU e adia o refresh. **A primeira correcao introduziu um defeito pior que o original.**

**E-05 - Do slot 62 errado a leitura de textura corrompida.**
`3086e0b` mapeia slot 62 como upload de CPU (500 linhas em `linear_upload.rs`) -> `CopyMemoryToMemoryX` real tem `(dst, src)` e o provider inverte -> a leitura de textura do Scarlett sobrescreve a textura com staging vazio -> `97b8ad3` apaga o modulo e implementa a copia na GPU. **Um erro de identificacao de slot sobreviveu ~13 h (2026-11-24 13:51 -> 2026-11-25 02:32).**

**E-06 - Da tabela de slots errada ao draw perdido.**
`bedf503` corrige `ExecuteIndirect` (59 -> 61) mas a tabela continua com 131/132 como sync antigo -> `e8af9fd` descobre que 131/132 sao `SyncTop/BottomOfPipeX` e o build agrupado e 133 -> `047de82` descobre que o builder de BVH chamava **copy** no ponto de entrada de **build**, entao nenhuma estrutura era copiada. **Tres commits para a mesma tabela.**

**E-07 - Do XWayland rootless a arvore de processos morta.**
`1de4d36` detecta `BadMatch` -> `00200ba` sobe Xwayland rootful privado -> o numero de display escolhido colide com socket/lock existente e `umu-run`/`proton`/`wineserver` sobrevivem ao timeout -> `1b9df5d` escolhe display livre por `/tmp/.X<n>-lock` e mata a arvore de descendentes via `/proc`. **Uma correcao de teste expos um vazamento de processos que envenenava execucoes seguintes.**

**E-08 - Do cache NVIDIA ao cache inutil.**
`9b18f7c` aponta `__GL_SHADER_DISK_CACHE_PATH` para `<prefix>/shadercache` -> o driver passa a ler `<path>/GLCache/<device>/<app>`; um bucket copiado em `<path>/<device>` nunca e aberto -> `d10b369` adota o bucket; `d439070` percebe que caminho vindo do ambiente ficava sem tamanho e o driver descartava o bucket no meio do boot.

**E-09 - Do merge ao lowering perdido.**
PR #56 adiciona arms `s_flbit_i32_b64`/`s_ff0_*` -> merge do PR #58 resolve conflito removendo os arms e mantendo os helpers -> instrucoes voltam a ser recusadas -> `1a50eb9` restaura. Em paralelo, merge de main em `tools/check_d3d12_regressions.py` deixa `IndentationError`/`SyntaxError` -> `8bd56af` conserta. **Dois defeitos independentes, mesma causa: resolucao manual de conflito sem build.**

**E-10 - Divergencia das linhagens (TuxBox x xbox-emulator).**
Ambos derivam de `drycheetah84/TomBox`. TuxBox: repositorio criado 2026-11-21T12:07Z, primeiro commit 14:23Z ja com 290 arquivos e 28.334 linhas, Apache-2.0, Linux-first, 591 commits, `runtime/*` + `tools/shader-compiler`. `omonkeyz/xbox-emulator`: repositorio criado 2026-11-21T08:08Z como **rastreador de progresso de 3 arquivos**, codigo importado em `410e01f` (16:34Z, 1.161 arquivos), GPL-3.0-or-later, Windows-first ("run as normal Windows processes"), 595 commits, workspace por DLL em `crates/compat/*` + `crates/tombox-xshader`. **Portanto: o repo do irmao e ~4 h mais velho, mas o codigo do TuxBox chegou ~2 h antes.** Nenhum dos dois e fork do outro: 7 caminhos em comum, apenas os obrigatorios; o irmao cita TuxBox somente em `THIRD_PARTY/CREDITS` ("tools/reference/tombox-*.patch only ... the rest of that repository is not used").

**E-11 - O que cada linhagem corrigiu e a outra nao (CONFIRMADA por arquivo).**
TuxBox tem e o irmao nao tem (porque roda em Windows nativo): drive `G:` via mount manager do Wine, cache GLCache por prefixo, Xwayland rootful privado, stack de 64 MiB para recursao do vkd3d, flags `-O1 -disable-lifetime-markers`, esconder `VK_EXT_descriptor_buffer` por GPU, `synchronization.lib`/`rust-version`, pin do GE-Proton. O irmao tem e o TuxBox nao tem: decodificador **BCPack** (11 arquivos em `crates/compat/xcrdapi/src/bcpack`, com `bcpack-rs`/`silicon-rans`), leitura de container **XVD** (`xcrd_file`/`xcrd_xvdfile`), suporte a titulos **Xbox One GameCore** (Terraria, Trackmania) e regra de digest de container DXIL reimplementada (`dxil_hash.rs`). TuxBox lista "BCPack, swizzled or physical-page requests, console-specific storage controls" como **restante** em `evidence/library-status.json`. Nao comparei correcao por correcao (ver L).

## F. Correcoes importantes

### F-01 - Thread auxiliar com 64 MiB para criar pipeline (`c679f13`)

**O que estava errado:** criar pipeline em thread de 1 MiB. **Por que falhava:** o vkd3d-proton compila DXIL->SPIR-V na thread chamadora e sua analise de fluxo recursa fundo; shaders traduzidos grandes estouram a pilha dentro do compilador. **Como foi corrigido:** thread *scoped* com pilha reservada de 64 MiB por criacao, com join; fallback para a thread atual se a criacao falhar. **Por que funciona:** a pilha e reservada, nao comprometida, entao o custo e virtual; o join preserva a semantica sincrona exigida pelo D3D12. **Dependencias:** nenhuma alem de `std::thread::scope` (Rust >= 1.63). **Reutilizavel:** sim, para qualquer camada D3D12->Vulkan que compile shader no `Create*PipelineState`. E o tipo de correcao que o autor de uma camada Windows-only nunca precisa fazer.

### F-02 - `-O1 -disable-lifetime-markers` para compute grande (`e9c8fb6`)

**O que estava errado:** usar `-O0` como unica saida para programas que estouram o otimizador. **Por que falhava:** `-O0` quadruplica os blocos de DXIL e o conversor DXIL->SPIR-V do vkd3d-proton explode (364 s). **Como foi corrigido:** `-O1` com *lifetime markers* desligados, mantendo `-O0` como fallback em falha/timeout de 60 s; as flags entram na **chave de cache** (`VARIANTS`) e no *fingerprint* do compilador. **Por que funciona:** o custo do DXC esta em *Jump Threading* alimentado por *lifetime markers*; desligando-os, `-O1` fica barato e o resultado e otimo o bastante para o driver. **Dependencias:** DXC >= 1.9 (release da Microsoft). **Reutilizavel:** sim - e uma receita para "DXIL que o vkd3d-proton nao digere". **Risco:** a chave de cache precisa incluir as flags, senao DXIL antigo e reaproveitado (o proprio commit trata isso).

### F-03 - Esconder `VK_EXT_descriptor_buffer` por GPU, em camada Vulkan (`72766e7`)

**O que estava errado:** desabilitar a extensao para todas as GPUs. **Por que falhava:** em RADV que nao precisa do caminho de descriptor set, isso forcava um caminho que corrompia heap (5 de 6 execucoes). **Como foi corrigido:** a camada `VK_LAYER_TUXBOX_console_cbv` intercepta `vkEnumerateDeviceExtensionProperties` e esconde a extensao apenas quando `minUniformBufferOffsetAlignment > 16 && minStorageBufferOffsetAlignment <= 16`. **Por que funciona:** o criterio e exatamente a condicao em que o contorno de CBV de 16 bytes e necessario; nas demais GPUs o driver fica no caminho nativo. **Reutilizavel:** sim, como padrao "contorno por camada Vulkan com predicado de dispositivo" em vez de variavel de ambiente global.

### F-04 - Nomes listados uma vez para evitar `readdir` do Wine (`4b4c08c`)

**O que estava errado:** abrir nomes que talvez nao existam no cache. **Por que falhava:** o Wine resolve nomes sem diferenciar maiusculas lendo o diretorio inteiro; com ~40 mil arquivos isso domina o tempo de CPU do processo. **Como foi corrigido:** `Names` guarda um `HashSet<String>` por diretorio (lista uma vez, com `OnceLock<Mutex<...>>`), consulta antes de abrir e acrescenta nomes criados pelo proprio processo; a traducao passa a ser lida da cache em vez de copiada para a sessao. **Por que funciona:** elimina a syscall patologica; um arquivo criado depois da listagem so custa uma requisicao, respondida pela cache do host. **Reutilizavel:** sim, para qualquer codigo que abra muitos arquivos em prefixo Wine/Proton, independente de shader.

### F-05 - Presentation check com present repetido (`1b0dfaa`)

**O que estava errado:** assumir que uma unica `PresentX` produz pixels visiveis. **Por que falhava:** no GE-Proton11-14 a imagem pode so aparecer em um present posterior; o teste lia a janela inalterada por 4 s. **Como foi corrigido:** extrair o `submit` em lambda e reapresentar ate 3 vezes, a cada 250 ms, durante a janela de amostragem. **Por que funciona:** reproduz o que um titulo faz (present por frame) sem afrouxar o criterio (continua falhando se nenhum frame chegar). **Reutilizavel:** sim, para qualquer teste de apresentacao em Proton; e um sintoma que nao existe em camada Windows-only.

### F-06 - Xwayland rootful privado com cookie e display livre (`00200ba` + `1b9df5d`)

**O que estava errado:** amostrar a tela pela janela raiz do X do desktop. **Por que falhava:** XWayland rootless recusa `GetImage` (`BadMatch`). **Como foi corrigido:** subir `Xwayland -displayfd -geometry 2560x1440 -noreset -auth <arquivo 0600>` com um `MIT-MAGIC-COOKIE-1` aleatorio em registro `FamilyWild`, trocar `DISPLAY`/`XAUTHORITY` so durante as checagens, escolher numero de display livre por `/tmp/.X<n>-lock` e matar a arvore de processos no timeout. **Por que funciona:** a raiz do servidor privado e legivel, e o isolamento evita colisao com a sessao do usuario. **Reutilizavel:** sim, para qualquer teste de janela em Linux/Wayland. **Risco:** depende de `WAYLAND_DISPLAY` e do binario `Xwayland`; sem eles falha rapido com a causa.

### F-07 - Escrita de valor de fence em vez de timestamp (`e8af9fd`)

**O que estava errado:** tratar `Write64BitValueBottomOfPipeX` como escrita de timestamp e falhar a lista inteira quando um comando nao era traduzivel. **Por que falhava:** as filas do titulo esperam labels arbitrarios; perder a escrita = travar; derrubar a lista = perder tambem a escrita. **Como foi corrigido:** a escrita aceita valor comum e flush, com ordenacao de marker-out; `Recording::skip` isola o comando problematico. **Por que funciona:** a lista continua valida e as dependencias entre filas permanecem satisfeitas. **Reutilizavel:** sim - e a regra "nunca falhar a lista inteira por um comando" que qualquer camada de traducao de command list deveria seguir.

### F-08 - `recording` como estrutura unica com `Segments` (`6bae710`)

**O que estava errado:** `CommandRecording` misturava a lista nativa e o estado do convidado sob um lock; uma espera de fence compute podia substituir a lista nativa no meio da gravacao. **Como foi corrigido:** `Recording` passa a carregar `active`, `list_type`, `compact`, `segments`, `written_ranges` e snapshots de estado grafico; a lista e dividida em segmentos e reproduzida por tipo de fila (COPY/COMPUTE/DIRECT), com fence persistente por fila e notifier compartilhado em vez de polling de 1 ms. **Por que funciona:** a espera no meio da lista vira uma fronteira explicita de segmento, com replay do estado vinculado (root signature, PSO, RTV/DSV snapshots). **Dependencias:** `queue_trace::note` e `retire_fence`. **Reutilizavel:** conceitualmente sim (o padrao "segmentar a lista em esperas"), mas o codigo e muito especifico do ABI Xbox. **Risco:** foi essa reescrita que gerou a regressao de `direct-copy` (`4ba74bc`).

## G. Problemas ainda existentes

1. **Stalls de fence no intro do ProjectScarlett** - declarado em `docs/PROGRESS.md:15,24`. Mitigacao: espera liberada apos 5 s com log; nao e correcao. CONFIRMADA.
2. **Esgotamento de memoria de descritores** - `"command_list_rejected": "descriptor memory exhausted"` no registro de gameplay; `docs/PROGRESS.md:25`. `98043df` introduziu lease/fence pinning, mas a prioridade continua aberta. CONFIRMADA.
3. **Tempo de compilacao de grandes shaders compute** - `docs/PROGRESS.md:26` ("as vezes a maior parte de uma hora"); `e9c8fb6` melhorou um caso medido. CONFIRMADA.
4. **4.496 de 25.079 requisicoes de shader sem suporte (17,9%)** - `evidence/projectscarlett-shader-preparation.json`. `docs/PROGRESS.md:27` pede suporte as formas restantes. CONFIRMADA.
5. **Prologo/cutscene inicial nunca apareceu** - `"prologue_cutscene_seen": false`; `docs/PROGRESS.md:15`. CONFIRMADA.
6. **Correcao de imagem, entrada, audio e frame times nao verificados** - `not_verified` no registro de gameplay. CONFIRMADA.
7. **Issue #191 aberta** - `buffer-remap` falha em RX 6700 XT / RADV/Mesa 26.3.1: `vkd3d_import_host_memory: Failed to import host memory, hr 0x8007000e`, 131 ocorrencias em 300 s de ProjectScarlett; causa nao estabelecida pelo autor. CONFIRMADA (leitura da issue).
8. **Issue #164 aberta** - slot 80 / `USE_THREAD_DIMENSIONS`; `2edd4a9` cobre o caso, mas a issue segue aberta. PROVAVEL.
9. **Issue #151 aberta** - `WaveSize(64)` em GPU wave32; `6161046` recusa cedo em vez de traduzir; PR #201 propoe lowering portavel. PROVAVEL.
10. **Relatos de tela preta** - issue #180: usuario ouve audio e menus, mas nao ve imagem. Coerente com `desktop_image: "black with MangoHud"` nas primeiras amostras. NAO VERIFICADA como problema geral (um relato).
11. **Assert de tile-heap remendado por RVA** - `8017bf5`/`dd0f8b2`; o autor declara que "hide the symptom". CONFIRMADA.
12. **Matrix Awakens**: 5 entradas pixel sem traducao (hair 2409/2414, cull distance-field 726/727/730) e falha de memoria de GPU da NVIDIA antes do tutorial de tiro (`docs/PROGRESS.md:7,31-32`). CONFIRMADA.
13. **Nenhum jogo comercial jogavel** - `README.md:17`. CONFIRMADA.
14. **Consumo de memoria**: ~15 GB de RAM + swap; `/tmp` em memoria precisa ser limpo antes (`docs/PROJECTSCARLETT.md:271`). CONFIRMADA (declarado).
15. **Dependencia de binario DXC externo**: o padrao agora e `--windows-dxc` (DLLs do Windows sob Proton), o que amarra o projeto a um release especifico da Microsoft. CONFIRMADA.
16. **Branch `ci/build-guests` nao mergeada** - o job que constroi todos os providers e Lanes existe apenas fora de `main` (`db5d899`). CONFIRMADA.

## H. Regressoes

**H-01 - Teto de 15 s no DXC (`444368a` -> `2d887b3`).** `444368a` unificou todos os limites em 15 s. `CS_Emit_Dynamic_Model_Edge` precisa de ~18 s em `-O0`, entao era **recusado em todo boot**: a correcao de latencia criou uma recusa deterministica. Corrigido em `2d887b3` com `O0_TIMEOUT = 90`. CONFIRMADA.

**H-02 - Espera de 2 s na CPU em placement (`cb185e0` -> `524cd21`).** A primeira correcao do alias de textura esperava na CPU por uma escrita que, por construcao, so poderia ser publicada **depois** do submit; a espera sempre estourava, travava a thread de render e o watchdog do titulo matava o jogo - pior que o `E_NOTIMPL` original. Corrigido em `524cd21`. CONFIRMADA.

**H-03 - Caminho de cache NVIDIA tornando bucket existente inutil (`9b18f7c` -> `d10b369`).** Ao mover o cache para o prefixo, o driver passou a ler `GLCache/<device>/<app>` e um bucket copiado em `<path>/<device>` deixou de ser aberto. Corrigido por adocao em `d10b369`. PROVAVEL (quem criou a copia mal posicionada nao e identificado).

**H-04 - Desabilitar `VK_EXT_descriptor_buffer` globalmente (`72766e7`).** O contorno de CBV virou causa de corrupcao de heap em RADV. Nao foi "reintroducao de defeito corrigido", mas um contorno que criou defeito novo em outra GPU. CONFIRMADA.

**H-05 - Lowering apagado por merge (`1a50eb9`).** PR #56 entregue e funcional; o merge do PR #58 removeu os arms. Regressao de funcionalidade ja corrigida, detectada por inspecao, nao por teste. CONFIRMADA.

**H-06 - Colisao de display e processos orfaos (`00200ba` -> `1b9df5d`).** O Xwayland privado escolhia numero de display que podia colidir, e o timeout de `subprocess.run` deixava `umu-run`/`proton`/`wineserver` vivos, envenenando execucoes seguintes. Corrigido em `1b9df5d`. CONFIRMADA.

**H-07 - Verificador de regressao quebrado por merge (`8bd56af`).** O merge de main no branch deixou `IndentationError`/`SyntaxError` em `tools/check_d3d12_regressions.py`, impedindo importar o teste. Corrigido no mesmo commit. CONFIRMADA.

**H-08 - Fixture de f64 sem os campos novos (`bd7108c`).** Merge adicionou campos e nao atualizou a fixture; teste de shader f64 parava de compilar. Corrigido com "give the f64 fixture the new fields". CONFIRMADA.

**H-09 - Fallback de CLI do DXC persistente quase perdido (`e8898ce`).** O merge do PR #134 exigiu um commit dedicado para "preserve validated CLI fallback for persistent DXC" - sinal de que a resolucao de conflito poderia ter removido o caminho de fallback. CONFIRMADA (o commit existe e declara isso); NAO VERIFICADO se sem ele o fallback teria sumido.

**H-10 - Regressao de direct-copy encontrada pelos cenarios novos (`78319b5` -> `4ba74bc`).** A reescrita de esperas (`6bae710`) mudou a ordenacao entre `CopyBufferRegion` e espera de label na fila direta; o cenario `direct-copy-wait` foi escrito para reproduzir e o PR #187 o incorporou ao gate. CONFIRMADA.

**H-11 - Teste de recuperacao de descritor dependente de limite fixo (`0705a9d`).** Depois de `98043df`, o self-test contava em um limite de descritores que mudou; o commit fixa ("pin") o limite. CONFIRMADA - regressao de teste, nao de produto.

## I. Conhecimento reutilizavel

### Especifico de Linux/Proton (nao existe em camada Windows-only)

**I-01 - Pilha de 64 MiB para compilar pipeline no vkd3d-proton.** Problema: `virtual_setup_exception stack overflow` em threads de 1 MiB. Implementacao: thread *scoped* com pilha reservada de 64 MiB por `Create*PipelineState`, join. Origem: TuxBox `c679f13`, `runtime/d3d12/src/big_stack.rs`. Dependencias: Rust >= 1.63 (`std::thread::scope`). Conflitos: qualquer codigo que assuma pilha pequena ou que use o stack para passar estado entre threads. Riscos: 64 MiB reservados por thread concorrente (virtual, mas conta no *overcommit*). Recomendacao: adotar sempre que a camada chamar um compilador recursivo (vkd3d-proton, dxil-spirv) dentro de uma API sincrona.

**I-02 - Flags de otimizacao para DXIL que o vkd3d-proton digere.** `-O1 -disable-lifetime-markers` para `cs_6_6`, com `-O0` como fallback e **timeout separado**; a chave de cache e o *fingerprint* do compilador devem incluir as flags. Origem: `e9c8fb6`. Riscos: esquecer as flags na chave faz o cache devolver DXIL de outra configuracao (o commit trata explicitamente). Recomendacao: reutilizavel em qualquer pipeline HLSL->DXIL->SPIR-V.

**I-03 - Esconder `VK_EXT_descriptor_buffer` por predicado de dispositivo.** Camada Vulkan que intercepta `vkEnumerateDeviceExtensionProperties` e esconde a extensao apenas quando `minUniformBufferOffsetAlignment > 16 && minStorageBufferOffsetAlignment <= 16`. Origem: `72766e7`, `tools/reference/cbv-layer/layer.c`. Conflito: forks de Proton que ja tem override proprio (o codigo mantem os dois caminhos). Risco: predicado errado reintroduz corrupcao de heap em RADV. Recomendacao: nunca desabilitar extensao Vulkan globalmente por variavel de ambiente.

**I-04 - Evitar abertura de nomes ausentes em prefixo Wine.** Listar o diretorio uma vez em `HashSet`, abrir so nomes conhecidos, e aceitar que arquivos criados depois custam uma requisicao. Origem: `4b4c08c`. Ganho medido: 3 min 45 s -> 1 min 16 s em um trecho de carregamento. Recomendacao: qualquer cache de arquivos com dezenas de milhares de entradas sob Wine.

**I-05 - Cache de pipeline NVIDIA por prefixo, no formato certo.** `__GL_SHADER_DISK_CACHE_PATH=<prefix>/shadercache`, `__GL_SHADER_DISK_CACHE_SIZE=8GiB`, `__GL_SHADER_DISK_CACHE_SKIP_CLEANUP=1`, e os arquivos devem estar em `<path>/GLCache/<device32hex>/<app>`. Origem: `9b18f7c`, `d10b369`, `d439070`. Risco: sobrescrever um caminho definido pelo usuario sem preservar o tamanho faz o driver descartar o bucket no meio do boot.

**I-06 - Teste de apresentacao em Wayland.** Subir `Xwayland` rootful privado (`-displayfd`, cookie `MIT-MAGIC-COOKIE-1` em arquivo 0600, `-noreset`), escolher display livre por `/tmp/.X<n>-lock`, trocar `DISPLAY`/`XAUTHORITY` so no bloco, matar a arvore de processos via `/proc`. Origem: `00200ba`, `1b9df5d`, `project_tuxbox/verify.py`. Risco: depender de `WAYLAND_DISPLAY`/`Xwayland`; sem eles, falhar rapido com causa explicita - nunca amostrar preto silenciosamente.

**I-07 - Present repetido como premissa de teste.** Em Proton, uma `PresentX` isolada pode nao chegar a tela; repetir ate 3 vezes a cada 250 ms mantem o teste util sem afrouxar o criterio. Origem: `1b0dfaa`.

**I-08 - Pre-flight do DXC.** Compilar um shader minimo com o `dxc` selecionado antes do build, e falhar nomeando o release da Microsoft quando o binario da distro estiver quebrado. Origem: `baa315b` (Arch, `LLVM HungOffUses`, SIGILL). Reutilizavel: qualquer projeto que dependa de DXC/SPIRV-Cross/DXC-like empacotado por distro.

**I-09 - Link e toolchain Windows a partir do Linux.** `rustc --print native-static-libs` e a fonte de verdade para `synchronization.lib` (Rust 1.100+); `-Wcast-function-type-mismatch` do clang 19 precisa ser desligado **depois** dos flags por arquivo, senao `/W4` reativa; `rust-version` no workspace evita erro E0658 confuso em toolchain antigo. Origem: `baa315b`, `ca772f1`, `04641a6`.

**I-10 - Mapear o pacote em `G:` e restaurar depois.** Registrar `dosdevices/g:` e `g::` antes do launch, gravar `tuxbox-package-drive.json`, restaurar no fim **e** quando o setup falhar, e nao tocar em `G:` que nao seja symlink. Origem: `84c35b3`, `c561d03`, `22caa19`. Riscos: um launch morto deixa o link trocado; a restauracao no proximo launch cobre isso.

**I-11 - Emulacao de instrucoes AMD-only em CPU Intel.** `clzero`: handler vetorizado que limpa a linha de 64 bytes e, quando reconhece o laco, a faixa inteira; `monitorx`/`mwaitx`: reescrever o segundo byte do opcode para `1F` (`NOP` de mesmo tamanho) na primeira falha; `clwb` -> `clflushopt` em pre-Ice Lake. Origem: `e1262ac`, `371c839`, `7d281f8`. Riscos: reescrever codigo do titulo exige checar `CPUID` e que a falha esteja na imagem do titulo; memoria nao gravavel deve continuar falhando.

**I-12 - Fixar o Proton por medicao, nao por padrao.** `bd847b5` escolhe GE-Proton11-14 porque seu `dxil-spirv` faz 1,1 s onde o Experimental levava 36,6 s. Licao: em Linux-first, a versao do runtime Proton e parte da especificacao de desempenho; um fallback silencioso para "Experimental" esconde regressao de minutos.

### Especifico de traducao de command list Xbox (vale tambem em Windows)

**I-13 - Nunca falhar a lista inteira por um comando.** `Recording::skip` descarta o comando, loga com taxa limitada (16 primeiras + potencias de 2) e preserva as escritas de fence das quais outras filas dependem. Origem: `e8af9fd`. Risco: mascarar bugs de traducao - mitigado pelo log com RVA do chamador.

**I-14 - Aposentar descritor por fence, nao por tempo.** `e4783b4` usa 3 s; `98043df` troca por *lease* + *fence pinning*. Licao: qualquer estrutura que a GPU le por handle precisa de vida util ancorada em fence de conclusao, e nao em relogio.

**I-15 - Segmentar a lista em esperas e reproduzir o estado vinculado.** `6bae710`: dividir em segmentos nos pontos de espera, replicar heaps, root signature/argumentos, PSO, estados IA/SO/RS/OM com snapshots de RTV/DSV, predication; reter fallback de barreira apenas para casos nao snapshotaveis. Reutilizavel como desenho, nao como codigo.

## J. Conclusao tecnica

**O que este repositorio e.** Um projeto de 4 dias que produziu ~140 mil linhas, 591 commits, 190 merges e uma camada de compatibilidade funcional o suficiente para: (a) rodar um jogo proprio de teste com 66 checagens em GPU; (b) traduzir 99,59% de 43.407 entradas de shader do Frosty e 82,1% de 25.079 requisicoes gravadas do ProjectScarlett; (c) apresentar buffers 3840x2160 do GTA VI 1.0.2.1 sob Proton de stock e alcancar `Game Running` no frame 32.534. Isso e engenharia de verdade, nao demonstracao.

**A tese Linux-first e honesta.** O projeto nao trata Linux como porta: o lancador e Python/Proton, o build acontece em Linux com `clang-cl`+`cargo-xwin`, o CI constroi tudo em `ubuntu-24.04` e a unica etapa que o CI nao faz e rodar na GPU. Em contrapartida, o preco aparece exatamente onde se espera: 13 dos 29 defeitos catalogados so existem porque o host e Wine/Proton/Vulkan (GLCache, dirent case-insensitive, XWayland, stack do vkd3d, flags do DXC reduzindo o custo do conversor, `VK_EXT_descriptor_buffer`, stride do RADV, `synchronization.lib`, DXC da distro). Uma camada Windows-only nao tem nenhum deles - e e por isso que o irmao `xbox-emulator` chega a jogos jogaveis (Trackmania, Terraria) sem resolver nada dessa lista.

**A qualidade da evidencia e o melhor ativo.** Cada correcao relevante vem com numero medido, versao de Proton, GPU e, frequentemente, um cenario Lanes que falha sem ela (`skipped-command`, `shared-texture`, `direct-copy-wait`, `descriptor-reclaim`). `check` exige GPU e a politica de merge exige a saida do `check` no PR (`a79aedb`). Essa disciplina e o que permite uma auditoria como esta chegar a conclusoes confiaveis.

**Os tres riscos reais.** (1) **Sincronizacao de GPU**: a reescrita de `memory_wait` (898 linhas) gerou regressao imediata e os stalls do intro persistem; a mitigacao atual e timeout de 5 s, nao corretude. (2) **Capacidade de descritores**: o descritor nunca liberado e o que encerra a execucao mais longa; `98043df` ataca a vida util, nao a reciclagem. (3) **Processo de merge**: 190 merges em 4 dias, 27 deles nao-PR, e pelo menos tres defeitos nasceram de resolucao manual de conflito (`1a50eb9`, `8bd56af`, `e8898ce`). A velocidade de 271 commits em um dia e impressionante e simultaneamente o maior passivo do repositorio - e 116 commits declaram coautoria de Claude, o que explica a escala e agrava o risco de conflito mal resolvido.

**Sobre "primeiro frame do GTA VI em 4K".** A parte verificavel e verdadeira e esta documentada com hash de log, GPU, driver e dimensoes: o jogo apresentou buffers 4K em 2026-11-23 numa RTX 4070 Ti SUPER, e a tela de selecao de conta aparece em 4K. O que **nao** esta verificado e que a imagem seja correta (o proprio registro lista "image correctness" como nao verificado) e que isto seja o primeiro do mundo (sem fonte externa). A issue #180 mostra pelo menos um usuario com tela preta - o que e coerente com "apresenta, mas nem sempre mostra o conteudo certo". A formulacao honesta e: **TuxBox levou o GTA VI 1.0.2.1 ao story intro com apresentacao 4K em Linux/Proton; a imagem ainda nao e verificada e nao ha jogo jogavel.**

**Sobre a relacao com o irmao.** A afirmacao do PortWatch de que `xbox-emulator` e "a continuacao deste projeto" e de que os dois tem "exatamente os mesmos 1.652 arquivos" nao se sustenta: os conjuntos de arquivos sao disjuntos (7 caminhos em comum, todos obrigatorios), as licencas divergem (Apache-2.0 x GPL-3.0-or-later), os alvos divergem (Linux/Proton x Windows) e o irmao declara explicitamente que so usa os patches `tools/reference/tombox-*.patch` do TuxBox. Ambos sao **irmandades paralelas do mesmo upstream TomBox**, com ~2 h de diferenca na chegada do codigo e estrategias opostas: TuxBox aposta em rodar sobre Wine/Proton e herda todos os problemas do host; o irmao aposta em Windows nativo e chega a jogos jogaveis sem nenhum deles. Nenhum dos dois "corrigiu o que o outro nao corrigiu" no sentido de conserto de bug - eles **resolvem problemas diferentes**.

## K. Indice de commits

Indice completo, em ordem cronologica (mais antigo primeiro), gerado por `git log --reverse --format="%h %ad %s" --date=short`. Os 591 commits de `main` estao aqui; nenhum foi omitido.

```text
6d15471 2026-11-21 Initial commit
151c088 2026-11-21 Implement and verify six partial Xbox DLL providers
feee428 2026-11-21 Track per-library export coverage and expand shared Xbox APIs
260e104 2026-11-21 Render Frosty's menu code and map Xbox shader coverage
d5abfc0 2026-11-21 Restore Frosty's code trails and link raster shaders on demand
5725d77 2026-11-21 Translate Frosty's Nanite mesh and culling shaders
accd7e1 2026-11-22 Run Frosty's highway chase and city free roam on Linux
4f91131 2026-11-22 Translate simple Xbox shaders independently and run Frosty on stock Proton
77642e1 2026-11-22 Improve Frosty’s menu glyphs and prepare shaders through the chase
521a741 2026-11-22 Reduce branch overhead when translating Xbox pixel shaders
393f485 2026-11-22 Measure Frosty’s shooting tutorial and menu texture detail on stock Proton
346b50e 2026-11-22 Preserve integer page-table reads in Xbox street materials
594c62d 2026-11-22 Expand Xbox ray tracing and shader translation
45f88ce 2026-11-22 Validate serialized BVH traversal and shadow shaders
b61def9 2026-11-22 Validate Xbox BVH nodes and shadow instances
b63805c 2026-11-22 Expand shadow culling and BVH validation
0e69566 2026-11-22 Validate RDNA BVH instructions and shadow culling
912ae06 2026-11-22 Expand shader census and RDNA traversal controls
640a42a 2026-11-22 Extend shadow traversal and shader controls
b43c65a 2026-11-22 Validate ray compaction and BVH ordering
98f6c76 2026-11-22 Validate ray visualization and BVH boundaries
adf70dd 2026-11-22 Characterize BVH traversal and GPU ordering
35d5208 2026-11-22 Validate retained BVH traversal and reflection scans
e76f612 2026-11-22 Expand reflection and BVH reference validation
5e400cc 2026-11-22 Expand reflection and shadow resource checks
85e8e1d 2026-11-22 Validate mesh SDF offsets and GPU shadow sources
4a4770f 2026-11-22 Validate bounded GPU shadow reads and tile shaders
1500678 2026-11-22 Validate hair deformation and parallel BVH checks
a067ba9 2026-11-22 Validate dynamic BVH and guarded GPU traversal
5d61036 2026-11-22 Test provider hair culling and BVH edge arithmetic
7d8fe16 2026-11-22 Validate hair prefix through Xbox provider
5298e51 2026-11-22 Validate hair compaction and GPU address reads
24c49e2 2026-11-22 Reject ambiguous acceleration structure addresses
2935150 2026-11-22 Translate hair cluster metadata culling shader
2018002 2026-11-22 Test exceptional BVH rounding behavior
9f026ef 2026-11-22 Compare direct lighting ray segments on both GPUs
6a19897 2026-11-22 Translate depth-filtered hair visibility compaction
066744d 2026-11-22 Document inline ray distance boundaries
9c52ce4 2026-11-22 Verify native triangle distance boundary
249e78e 2026-11-22 Check native zero-distance triangle candidates
2dd6dbc 2026-11-22 Test widened ray query distance filtering
6bd38de 2026-11-22 Translate indirect hair visibility compaction
ae42eb8 2026-11-22 Test zero-distance procedural ray candidates
310320e 2026-11-22 Translate depth-only subsurface setup
78e8991 2026-11-22 Test procedural candidates with GPU triangle data
c2dfdbf 2026-11-22 Translate classified subsurface setup
099445e 2026-11-22 Test sidecar directory with guarded ray candidates
29a4795 2026-11-22 Validate GPU-generated procedural sidecars
aea0d4f 2026-11-22 Translate Lumen visibility indirect arguments
d1cda6f 2026-11-22 Translate guarded LDS exchange operations
76db0e4 2026-11-22 Translate Lumen update grid tile shader
7cbfeda 2026-11-22 Clarify progress map and project documentation
1aaa1c4 2026-11-22 Validate paired triangle sidecar bounds
4cbab30 2026-11-22 Validate sphere graph sidecar generation
19176db 2026-11-22 Expand graphics and memory export coverage
ac78271 2026-11-22 Measure Xbox DXR compiler factory interface
94f4556 2026-11-22 Translate texture layouts and allocation hooks
a1bfa1a 2026-11-22 Support checked 1D layouts and write-watch queries
d9545fd 2026-11-22 Support write-watch suspension and resume
473b1e7 2026-11-22 Survey global pixel wave elections
c5bcc4f 2026-11-22 Support checked linear 3D texture layouts
d29cde1 2026-11-22 Measure pixel wave behavior on both GPUs
a745ff0 2026-11-22 Expand XG tile shapes and write-watch allocations
ada7a3e 2026-11-22 Validate XG depth swizzle modes
db6d614 2026-11-22 Add PIX status logging and scatter page mapping
af943b8 2026-11-22 Expand verified CPU virtual allocation profiles
062a726 2026-11-22 Implement swizzle selection and PIX PDB lookup
173db5f 2026-11-22 Survey PIX PDB parser behavior
595c9f7 2026-11-22 Expand XMem CPU heap attribute support
971c1a1 2026-11-22 Measure PIX parser ownership and mutation
249b2f4 2026-11-22 Support tiled 3D textures and XTF assertion state
cc23b46 2026-11-22 Add XTF diagnostic severity helpers
9e216b1 2026-11-22 Implement checked XMem title heap operations
d2e5108 2026-11-22 Support unpacked tiled 3D mip chains
09c88fd 2026-11-22 Update library documentation and coverage summary
95bebe3 2026-11-22 Measure XTF assertion handler contracts
c23c83c 2026-11-22 Preserve XMem null heap free behavior
949862a 2026-11-22 Support checked packed 3D mip tails
ad78374 2026-11-22 Measure overlapping XTF assertion dispatch
a0afb42 2026-11-22 Expand XG texture layouts and batch startup tracing
759701f 2026-11-23 Expand D3D12, XMem, XG and DXC providers and verify Frosty startup
6607af2 2026-11-23 Track verified project milestones and focus the library map on Frosty
e82aaf2 2026-11-23 Match the Series custom microcode export through driver dispatch
3a0e412 2026-11-23 Measure Series library and shader coverage and retire unused providers
858d057 2026-11-23 Track verified game behavior and improve Xbox shader and graphics support
f2aebd2 2026-11-23 Translate Frosty hair visibility and expand compute shader coverage
5be71bc 2026-11-23 Complete Frosty non-ray compute translation and expand FidelityFX shader support
f47dfcd 2026-11-23 Run Frosty through Wake Up with packaged Lumen settings
03ca42e 2026-11-23 Advance ProjectScarlett startup and Frosty rendering
115fcce 2026-11-23 Resolve package root in inspect.member() before containment checks
46f1dbf 2026-11-23 Make Python unittest suite pass on Windows
91f775d 2026-11-23 Merge pull request #2 from Batushn/fix/inspect-resolve-root
57fe562 2026-11-23 Merge pull request #3 from Batushn/fix/windows-unittest-compatibility
fe6b0e1 2026-11-23 Bring ProjectScarlett from Game Init to Game Running
33d513e 2026-11-23 Add offline platform services and graphics diagnostics
938c402 2026-11-23 Merge ProjectScarlett runtime and graphics updates
c217aa4 2026-11-23 Now displays GTA VI properly on linux.
52a24b8 2026-11-23 Recognize ProjectScarlett mesh shader exports
89af910 2026-11-23 Prewarm ProjectScarlett shaders and speed up texture writeback
9cbc5b9 2026-11-24 Reconcile mesh and resolve shader bindings
1b6fbb1 2026-11-24 License TuxBox under Apache 2.0
0b80c58 2026-11-24 Expand mesh shader coverage and report Scarlett progress
baa315b 2026-11-23 Link synchronization.lib and reject broken DXC builds on Linux
84c35b3 2026-11-23 Map the package to G: for Wine and Proton launches
9a81229 2026-11-24 Translate SM 6.6 descriptor-heap resources in compute shaders
cf0abde 2026-11-23 Bound, cache, and fall through the DXC probe
c561d03 2026-11-23 Find the package root from the launch and restore G: afterwards
2cc9ff3 2026-11-23 Fix check_frosty_cached_pages imports
a15bf14 2026-11-23 xgameruntime: log served classes, print canonical GUIDs
1c321f0 2026-11-23 Log distinct d3d12 pipeline failures and per-kind counts
dab1a10 2026-11-23 xg-layout: add host unit tests for layout invariants
2bf99e3 2026-11-23 Serve XPackageEnumeratePackages on XPackage slot 39
aaaad1d 2026-11-23 xmem: commit physical-page sections whole instead of per view
ca772f1 2026-11-24 Fix Linux build with current Rust and clang
444066e 2026-11-24 Lower mesh resource-heap loads and whole-wave allocations
cc9d34d 2026-11-24 Add an opt-in Project Scarlett graphics profile
05c7e3e 2026-11-24 Merge pull request #15 from ReverendOzmund/fix-linux-build-current-toolchains
ff7fd07 2026-11-24 Merge pull request #6 from NathansDomainSolutions/fix/package-drive
c7487e4 2026-11-24 Merge pull request #9 from NathansDomainSolutions/fix/cached-pages-imports
0651924 2026-11-24 Merge pull request #14 from NathansDomainSolutions/perf/xmem-commit-section-whole
1cfd0c6 2026-11-24 Merge pull request #5 from NathansDomainSolutions/build/linux-toolchain-checks
faa99ae 2026-11-24 Merge pull request #10 from NathansDomainSolutions/fix/xgameruntime-query-log
1a9929a 2026-11-24 Merge pull request #11 from NathansDomainSolutions/fix/d3d12-failure-diagnostics
56d40c7 2026-11-24 Merge pull request #12 from NathansDomainSolutions/test/xg-layout-invariants
97c1cad 2026-11-24 Merge branch 'translator-bindless-resources'
23c848b 2026-11-24 Pass the package directory through the merged launch path
669451e 2026-11-24 Merge pull request #13 from NathansDomainSolutions/feat/xpackage-slot-39
c679f13 2026-11-23 runtime/d3d12: create host pipelines on a thread with a 64 MiB stack
7666f54 2026-11-24 Merge pull request #17 from NathansDomainSolutions/fix/pipeline-big-stack
40e1565 2026-11-24 Fix frame tests: present probe crash and clang 19 RGB10 clear
9e2fd66 2026-11-24 Merge pull request #18 from ReverendOzmund/fix-frame-present-tests
b0dc9a1 2026-11-24 Bind uncovered compute descriptors that FSSM and PSV agree on
22caa19 2026-11-23 Restore package drive state when launch setup fails
76591e4 2026-11-23 Skip malformed pipeline timing events
e360e3e 2026-11-24 Merge pull request #20 from NathansDomainSolutions/fix/package-drive-launch-cleanup
344a03b 2026-11-23 Accept symlinked Proton launchers with renamed targets
18e81b5 2026-11-24 Merge pull request #21 from NathansDomainSolutions/fix/pipeline-trace-events
f22f388 2026-11-24 Merge pull request #22 from NathansDomainSolutions/fix/proton-runner-symlink
d130bf1 2026-11-23 launcher: add run --offline to block crash uploads on Linux
ca20f93 2026-11-23 docs: add a ProjectScarlett setup guide
2edd22c 2026-11-23 dxbc, d3d12: support s9 raster descriptor tables and track null compute dispatches
d7773e2 2026-11-23 runtime shaders: retry DXC at -O0 when the optimised compile times out
ca29374 2026-11-23 Bound and test the retry after a DXC timeout
78be8ab 2026-11-24 Merge pull request #23 from NathansDomainSolutions/fix/dxc-timeout-fallback
19786f2 2026-11-24 Merge pull request #8 from NathansDomainSolutions/docs/scarlett-setup
cc752ba 2026-11-23 xshader: translate image_gather4_l, resolve root-less descriptors, and handle typed heap stores
c039cb4 2026-11-23 xshader: translate image_gather4_l on 2D images as four texel-centre samples
791d59a 2026-11-24 Merge pull request #24 from NathansDomainSolutions/feat/gather-explicit-lod
2bad2b7 2026-11-24 Merge pull request #25 from NathansDomainSolutions/fix/rage-shader-translation
b8ee94a 2026-11-24 Merge pull request #27 from NathansDomainSolutions/feat/offline-launch
6a57c65 2026-11-23 gameinput: turn the keyboard gamepad on by default, as documented
4359e7f 2026-11-24 Merge pull request #28 from NathansDomainSolutions/feat/input-joystick
dd491c6 2026-11-24 Merge pull request #26 from NathansDomainSolutions/fix/raster-s9-diagnostics
c9eaf42 2026-11-23 tools: rank xshader refusals over ProjectScarlett's recorded requests
118c71d 2026-11-24 Merge pull request #29 from NathansDomainSolutions/tools/scarlett-refusal-triage
1233e21 2026-11-23 xshader: match literal sampler words in any shader with static samplers
243b695 2026-11-23 xshader: FSSM fallback for uncovered raster slots; private root-less samplers
f8abe8f 2026-11-23 xshader: private samplers when the root declares no static samplers
4afc67d 2026-11-23 xshader: MUBUF SOFFSET as an address term; raw SRVs behind root descriptors
290f307 2026-11-24 launcher: skip Scarlett's pre-game shader archive
345b23a 2026-11-24 xshader: reuse one heap resource local per uniform SGPR index
d73502d 2026-11-24 Merge pull request #30 from NathansDomainSolutions/fix/scarlett-samplers-raster
ebd3022 2026-11-23 xshader: fetch compute samplers from the sampler heap at a run-time offset
c90ff8e 2026-11-24 Merge pull request #31 from NathansDomainSolutions/fix/sload-dynamic-soffset
c418316 2026-11-23 xshader, triage: harden the run-time sampler-heap fetch and the triage CLI
21c8d05 2026-11-24 Merge pull request #33 from NathansDomainSolutions/fix/heap-sampler-hardening
9b18f7c 2026-11-24 launcher: keep each prefix's NVIDIA shader cache
444368a 2026-11-24 launcher: compile cs_6_6 at -O0 and cap DXC at 15s
8017bf5 2026-11-24 d3d12: continue past the Scarlett tile-heap assert
d10b369 2026-11-24 Fix linux shader caching issues
1de4d36 2026-11-24 verify: report rootless XWayland instead of sampling a black screen
04641a6 2026-11-24 build: declare the Rust 1.95 floor in the workspace
e1d7f5a 2026-11-24 xshader: pin raw buffer transfers to one dword per Load/Store
855f2da 2026-11-24 Merge pull request #37 from postblink/test/xshader-per-dword-raw-transfers
b87ece1 2026-11-24 Merge pull request #36 from postblink/build/declare-rust-version
dc0b8c1 2026-11-24 Merge pull request #35 from postblink/fix/present-rootless-xwayland
00200ba 2026-11-24 verify: run the present checks in a private rootful Xwayland
11a278a 2026-11-24 Merge pull request #38 from postblink/feat/present-private-xwayland
832d0aa 2026-11-24 runtime_shaders: strip whitespace from profile output before validation
00fa5f9 2026-11-24 test: add Bazzite RTX 4080 Proton evidence
72766e7 2026-11-24 host: hide VK_EXT_descriptor_buffer only on GPUs that need it
8180866 2026-11-24 scarlett: rank shader refusals during preparation
4a518ef 2026-11-24 dxbc: recover literal comparison samplers
0f6d97c 2026-11-24 scarlett: rank refusals directly from runtime logs
2ac1e6d 2026-11-24 xgameruntime: store game saves in containers
d9a5482 2026-11-24 shader: accept proven uniform forward reduction guards
e7e911f 2026-11-24 test: verify guarded wave32 reductions on GPU
48e4431 2026-11-24 shader: emulate verified single-wave64 reductions with group shared exchange
e2860b5 2026-11-24 shader: share literal sampler matching and recover scalar pair copies
3d17a00 2026-11-24 shader: support verified byte-stride indexed raw SRV reads
2a7fd48 2026-11-24 shader: generalize verified non-returning wide image maxima
0a964da 2026-11-24 docs: regenerate progress map with RTX 4080 evidence
5f8cda4 2026-11-24 scarlett: compare validated shader coverage across compiler revisions
d575b34 2026-11-24 xshader: support s_ff1_i32_b64 and 16_16_16_16_UINT typed buffers
dbd7525 2026-11-24 test: enforce expected shader outcomes and reject inconclusive comparisons
be20365 2026-11-24 d3d12: add opt-in allocation size diagnostics
02dcc15 2026-11-24 scarlett: warn about missing Windows DXC dependencies
ff033aa 2026-11-24 shader cache: schedule independent live translations concurrently
285f686 2026-11-24 xshader: accept row_mirror DPP16 v_mov in wave reductions.
4ca3bff 2026-11-24 shader: lower verified ordinary scalar bit operations
e3ed1a0 2026-11-24 scarlett: turn shader comparisons into prioritized replay and regression worklists
4cd8d1c 2026-11-24 Merge pull request #45 from Valdi320/work/scarlett-comparison-samplers
f988b31 2026-11-24 Merge pull request #46 from Valdi320/work/scarlett-uniform-reduction-guards
01928f2 2026-11-24 Merge pull request #43 from Valdi320/work/valdi-rtx4080-bazzite
306344e 2026-11-24 Merge pull request #40 from trigger337/fix/shader-profile-whitespace
5f777d6 2026-11-24 Merge pull request #42 from Jerkin42/fix/cbv-descriptor-buffer-per-gpu
6e48a58 2026-11-24 Merge pull request #49 from Valdi320/work/scarlett-verified-wide-atomic
914d664 2026-11-24 Merge pull request #55 from s3rius/xshader/row-mirror
a2c1e9e 2026-11-24 d3d12: return a null compute pipeline when the driver compile passes its deadline
3086e0b 2026-11-24 d3d12: upload command-list slot 62 from the CPU and mirror a failed CBV import
f44166c 2026-11-24 test: generate native scalar programs and compare with independent CPU oracle
893daa0 2026-11-24 docs: record CPU oracle coverage and detected refusals
1229a76 2026-11-24 test: recheck scalar oracle on updated upstream
fad9835 2026-11-24 tools: diagnose Scarlett launch blockers and compare aligned frames offline
0d08d73 2026-11-24 tools: analyze memory retention and paired performance regressions
11b70eb 2026-11-24 shader: retain proven scalar provenance through dependent compute operations
0ee91ad 2026-11-24 xshader: support s_flbit_i32_b64, s_ff0_i32_b32 and s_ff0_i32_b64
e55d354 2026-11-24 game_log, verify: close descriptor on fdopen error and clean up staging on failure
e6ca8f2 2026-11-24 test: preserve minimized dependent-scalar regression cases
fe4ab60 2026-11-24 test: bounded scalar differential campaigns with minimized reproducers
fae402f 2026-11-24 shader: lower verified ordinary packed 16-bit integer operations
7ca880e 2026-11-24 launcher: reject launcher options placed after the game path
fcbcf5b 2026-11-24 docs: complete the Lanes setup and verify commands
7b7dec7 2026-11-24 Merge pull request #68 from JohnVonWells/docs/fresh-setup-hints
34ba13a 2026-11-24 shader: validate saturating packed integer arithmetic
cc59e5f 2026-11-24 Merge pull request #67 from JohnVonWells/launcher/friendlier-arg-errors
fd3a6f4 2026-11-24 Merge pull request #66 from Valdi320/work/scarlett-packed-integers
1a0b6f3 2026-11-24 shader: validate integer half chains with CPU oracle and add three ordinary forms
02953b0 2026-11-24 ci: run host checks on pull requests and pushes to main
f9e7883 2026-11-24 shader: lower compute-heap 2D image_get_resinfo
00f69b8 2026-11-24 d3d12: record the command-list slots shared by both Xbox generations
3366551 2026-11-24 Merge pull request #47 from Jerkin42/feat/xgamesave-containers
3b11973 2026-11-24 Merge pull request #52 from Valdi320/work/scarlett-dxc-preflight
092e96f 2026-11-24 scarlett: report preparation refusal buckets without sharing shader inputs
8fed08c 2026-11-24 xshader: support QDMode ds_swizzle_b32 in compute shaders
2c65e37 2026-11-24 Merge current upstream main and retain both sets of documentation
8fd6a99 2026-11-24 Merge current upstream main and retain both sets of documentation
67198a2 2026-11-24 Merge current upstream main and retain both sets of documentation
d101b96 2026-11-24 Merge current upstream main and retain both sets of documentation
02359d9 2026-11-24 Merge current upstream main and retain both sets of documentation
0c99119 2026-11-24 Merge current upstream main and retain both sets of documentation
6030b68 2026-11-24 diagnose current Scarlett driver failures and sampled fallback counts
56cbb05 2026-11-24 diagnose native host heap import failures without changing placement
7d8be17 2026-11-24 xshader: v_interp_p1ll_f16 support.
4db1076 2026-11-24 tools: correlate Linux kernel GPU faults with explicit launch windows
1ab02ab 2026-11-24 diagnostics: distinguish sampled compute failures and latest milestone
2dd567d 2026-11-24 shader: prove zero SMEM offsets and literal-uniform wave guards
072e007 2026-11-24 shader: propagate exact scalar offsets and prove uniform scalar chains
1d20aa5 2026-11-24 scarlett: compare aggregate refusal transitions on identical requests
188463f 2026-11-24 shader: extend verified heap offsets, mesh ranges and wave elections
326c500 2026-11-24 scarlett: distinguish twelve additional fixed refusal families
66c57cf 2026-11-24 diagnostics: aggregate pipeline linkage, driver results and allocation sizes
371c839 2026-11-25 d3d12: rewrite unsupported clwb to clflushopt on pre-Ice Lake CPUs
242121b 2026-11-24 shader: lower raw idxen buffer accesses in the HLSL emitter
aeed457 2026-11-24 d3d12, xmem: reserved resources, tile commands and virtual memory updates
5d951b2 2026-11-25 build: fix native Windows MSVC link forwarders and /W4 /WX warnings
c3c4c57 2026-11-24 d3d12: reuse interleaved live page imports with a bounded weak cache
0bbb39d 2026-11-24 d3d12: retain embedded-root compute translation and link portable pixel consumers
dbc8d41 2026-11-24 d3d12: accept height-padded single-subresource linear uploads
e25075c 2026-11-24 Merge pull request #79 from Valdi320/work/scarlett-offset-wave-proofs
f6bdf40 2026-11-24 Merge pull request #70 from JohnVonWells/xshader/compute-quad-swizzle
4ba2386 2026-11-24 Merge pull request #69 from Valdi320/work/scarlett-packed-cpu
7995cef 2026-11-24 Merge pull request #54 from Valdi320/work/scarlett-parallel-shader-requests
a8dfc7c 2026-11-24 Merge branch 'main' into work/scarlett-refusal-ranking
3d9185a 2026-11-24 Merge pull request #82 from Valdi320/work/scarlett-runtime-bottlenecks
a31b92d 2026-11-24 Merge branch 'main' into work/scarlett-prep-reasons
699e50c 2026-11-24 Merge pull request #75 from Valdi320/work/scarlett-prep-reasons
621d6da 2026-11-24 Merge pull request #59 from trigger337/fix/staging-descriptor-cleanup
dbdc3fb 2026-11-24 Merge pull request #78 from Valdi320/work/linux-gpu-journal-correlation
a64cadc 2026-11-24 Merge pull request #80 from chunfeng11221/d3d12/clwb-fallback
c2640de 2026-11-24 Merge pull request #65 from s3rius/xshader/v_interp_p1ll_f16
cf95792 2026-11-24 Merge pull request #56 from trigger337/feat/sop1-bit-instructions
e7b3f2a 2026-11-24 Merge remote-tracking branch 'origin/main' into refresh/pr48-mail-20261124
ff28ede 2026-11-24 Merge remote-tracking branch 'origin/main' into refresh/pr58-mail-20261124
482889e 2026-11-24 Merge remote-tracking branch 'origin/main' into refresh/pr60-mail-20261124
761857b 2026-11-24 Merge remote-tracking branch 'origin/main' into refresh/pr51-mail-20261124
dc1601f 2026-11-24 Merge remote-tracking branch 'origin/main' into refresh/pr44-mail-20261124
b7e935e 2026-11-24 Merge remote-tracking branch 'origin/main' into refresh/pr61-mail-20261124
84fe4a2 2026-11-24 Merge remote-tracking branch 'origin/main' into refresh/pr62-mail-20261124
0ec0b42 2026-11-24 Merge pull request #44 from Valdi320/work/scarlett-refusal-ranking
64f1af7 2026-11-24 Merge pull request #48 from Valdi320/work/scarlett-verified-raw-index
2885072 2026-11-24 Merge pull request #58 from Valdi320/work/scarlett-scalar-bit-operations
af8e581 2026-11-24 Merge branch 'main' into work/scarlett-cpu-oracle
e365d24 2026-11-24 Merge branch 'main' into work/scarlett-observability
356ac1a 2026-11-24 Merge branch 'main' into work/scarlett-resource-analysis
a9b7739 2026-11-24 wave reduction: accept int and floating masked prologues in wave_sum lowering
2ca683f 2026-11-24 d3d12: rank recorded null pipeline usage without native pointer identities
dddbcbf 2026-11-24 shader: prove bounded runtime scalar loads from embedded code tables
dbc5492 2026-11-24 tools: select private replay requests from sampled null pipeline usage
e7cc159 2026-11-24 d3d12: retire GPU imports whose pages a remap replaces
dd0f8b2 2026-11-24 d3d12: recover from two ProjectScarlett tile-heap run-table asserts
bc15cb9 2026-11-24 Merge pull request #84 from Valdi320/work/scarlett-dynamic-offset-bounds
a674a9a 2026-11-24 Merge pull request #83 from Valdi320/work/scarlett-null-usage
8acef7c 2026-11-24 Merge pull request #73 from s3rius/xshader/wave_reduction
0e762f9 2026-11-24 shader: fix packed signed saturation in dependent GPU chains
97642c7 2026-11-24 d3d12: bound saturated weak page cache reclamation work
bedf503 2026-11-24 d3d12: add Scarlett command-list ExecuteIndirect (slot 61) and DriverHintX (slot 101)
95af9d3 2026-11-24 docs: record ProjectScarlett shader preparation at 6,487 of 16,025 requests
2f2f0b3 2026-11-24 test: exercise Scarlett indirect and driver hint command-list slots
dc6c1d6 2026-11-24 test: automate offline GPU controls and verify null compute usage
acbd67e 2026-11-24 shader: prove finite runtime CBV descriptor selection
befbfd7 2026-11-24 Merge remote-tracking branch 'origin/main' into refresh/pr51-twenty-20261124
b9e42ec 2026-11-24 Merge pull request #85 from Valdi320/work/scarlett-autonomous-validation
0008dac 2026-11-24 tools: verify paired shader inputs and export private-safe aggregate outcomes
8fa657f 2026-11-24 Merge pull request #86 from Valdi320/work/scarlett-page-cache-bounded-sweep
44a5b2d 2026-11-24 Merge pull request #87 from Valdi320/work/scarlett-twenty-validation
d58956c 2026-11-24 Merge pull request #89 from Valdi320/work/scarlett-bounded-cbv-descriptors
0eda69f 2026-11-24 xshader: support v_sat_pk_u8_i16
5cee10f 2026-11-24 Merge pull request #88 from Valdi320/work/scarlett-null-usage-fixture
326fd2d 2026-11-24 Add CachyOS + RX 6950 XT verify results
c9f90f2 2026-11-24 Merge pull request #51 from Valdi320/work/scarlett-allocation-trace
7b39959 2026-11-24 Merge pull request #90 from Valdi320/work/scarlett-next20-validation
1a50eb9 2026-11-24 shader: restore s_flbit_i32_b64 and s_ff0_i32 lowering dropped by a merge
f5ab25e 2026-11-24 Merge pull request #63 from JohnVonWells/xshader/support-v-sat-pk-u8-i16
f2184c1 2026-11-24 Merge pull request #64 from JohnVonWells/verify/arch-rdna2-results
3102329 2026-11-24 shader: extend portable wave64 reductions to proven extrema
17a61e7 2026-11-24 evidence: run the synthetic GPU controls on an RX 6950 XT
2ce238c 2026-11-24 test: disable null fallback in positive GPU regression controls
a171f17 2026-11-24 docs: record portable extrema GPU validation
c6136a3 2026-11-24 test: validate saturating pack chains and bounded CBV reflection
db6a4bf 2026-11-24 xshader: lower fourteen ordinary VOP3 integer forms
81b8aec 2026-11-24 xshader, dxbc: link raster stages whose interpolant component types differ
2d887b3 2026-11-24 runtime shaders: give the unoptimised DXC fallback 90 seconds
81d4a92 2026-11-24 Merge pull request #97 from NathansDomainSolutions/fix/dxc-o0-timeout
be864f5 2026-11-24 Merge pull request #95 from JohnVonWells/xshader/ordinary-integer-vop3
2d7f215 2026-11-24 Merge pull request #93 from Valdi320/work/scarlett-wave-extrema-controls
f057e9b 2026-11-24 Merge pull request #96 from NathansDomainSolutions/fix/interpolant-component-types
63fa17c 2026-11-24 Add CachyOS + RX 6900 XT (Proton Experimental) verify results
ff2cdfb 2026-11-24 d3d12: add CreatePlacedRawUnorderedAccessViewX (device slot 71)
b6647ec 2026-11-24 d3d12: map CopyResourceX, DXR and DispatchMesh in the Scarlett command-list table
6171f56 2026-11-24 d3d12: preserve guest buffer identity after late commitment
6842ef0 2026-11-24 shader: add a root signature dump example
63ac019 2026-11-24 Merge pull request #98 from Valdi320/work/scarlett-mailblock-runtime
d402b85 2026-11-24 Merge pull request #94 from Valdi320/work/scarlett-sat-cbv-controls
d94e4e5 2026-11-24 Merge pull request #91 from JohnVonWells/evidence/rx6950xt-gpu-controls
f80dff3 2026-11-24 xshader: lower exact f64 rounding and f64/int32 conversions
d2fa361 2026-11-24 Merge pull request #81 from chunfeng11221/build/native-msvc
919f148 2026-11-24 Merge pull request #62 from Valdi320/work/scarlett-resource-analysis
0dab8e6 2026-11-24 Merge pull request #99 from JohnVonWells/xshader/exact-f64-rounding
cb185e0 2026-11-24 d3d12: degrade placed-texture aliasing instead of failing the queue
45c7395 2026-11-24 Verify queue-ordered XMem remaps against physical GPU readback
63343eb 2026-11-24 xshader: translate most ProjectScarlett programs
3e74127 2026-11-24 Merge main
bd7108c 2026-11-24 xshader: give the f64 fixture the new fields
de5639f 2026-11-24 xshader: lower SAD forms and v_mad_i64_i32
0ad2fa9 2026-11-24 Merge pull request #101 from Jerkin42/feat/xshader-coverage
524cd21 2026-11-24 d3d12: defer placed-texture refresh instead of waiting on the CPU
8ed271c 2026-11-24 Merge pull request #102 from JohnVonWells/xshader/sad-and-mad-i64
7b815f2 2026-11-24 Merge pull request #100 from Valdi320/work/scarlett-remap-validation
1b9df5d 2026-11-24 Fix private Xwayland display collision and guest-process cleanup.
7d7dc72 2026-11-24 shader: preserve proven straight-line DPP8 lane exchanges
b5b25f4 2026-11-24 Merge pull request #104 from Valdi320/work/scarlett-lone-lane-validation
eac38ec 2026-11-24 test: verify nonblocking placed-texture refresh and late publication
854923d 2026-11-24 Merge pull request #105 from s3rius/bugfix/wayland-displays
6aeed55 2026-11-24 Merge pull request #106 from Valdi320/work/scarlett-alias-deferred-validation
5c73bee 2026-11-24 d3d12: import a 64 KiB window behind root CBVs
2dfceb3 2026-11-25 shader: preserve heap offset and straight-line descriptor provenance
5a67cef 2026-11-24 Merge pull request #107 from NathansDomainSolutions/fix/root-cbv-64k-window
9167968 2026-11-24 Merge pull request #108 from Valdi320/work/scarlett-lane-mask-provenance
4eed160 2026-11-25 tests: run the Python suite on Windows
ade292d 2026-11-24 Merge pull request #109 from davutsimsekk/fix/windows-python-tests
c0d5369 2026-11-24 tests: resolve the temporary root in the shader inventory test
d18ca6c 2026-11-24 Added evidence arch-rtx5090.
1181af3 2026-11-25 shader: correlate dynamic heap descriptor origins through direct CFG
3ad360e 2026-11-24 Merge pull request #110 from s3rius/feature/evidence-arch-rtx5090
8661492 2026-11-24 Merge pull request #111 from Valdi320/work/scarlett-heap-cfg-provenance
a25ee08 2026-11-24 Merge branch 'main' into work/scarlett-strict-gpu-regression
7fa2c83 2026-11-24 xgameruntime: serve XSystemGetAppSpecificDeviceId (XSystem slot 5)
285a7ab 2026-11-24 xcrdapi: run the reads between queue barriers in parallel
329ab83 2026-11-24 Merge pull request #112 from NathansDomainSolutions/feat/xsystem-app-device-id
8bd56af 2026-11-24 test: repair the GPU regression checker after the main merge
4db194e 2026-11-25 test: verify full Scarlett root CBV windows across pages
2e96986 2026-11-24 Merge pull request #113 from NathansDomainSolutions/perf/xcrdapi-parallel-reads
53bf140 2026-11-24 Speed up ProjectScarlett shader preparation
4bce37a 2026-11-24 Merge pull request #115 from NathansDomainSolutions/speedup/prep-jobs-and-pack-seed
9139f2f 2026-11-24 Merge pull request #92 from Valdi320/work/scarlett-strict-gpu-regression
6f5af52 2026-11-25 Merge upstream strict regression controls and extend CBV coverage
f111f5c 2026-11-24 xshader: size mesh vertex output to DXC's 32 KiB limit
fa92644 2026-11-25 docs: record five-window strict root CBV validation
8dfe439 2026-11-24 Merge pull request #114 from Valdi320/work/scarlett-root-cbv-window-validation
6a2ca92 2026-11-25 shader: reconcile portable pixel consumer component types
36b6a61 2026-11-24 Add progress overlay to shader translation / compilation stage of games
5213877 2026-11-24 tools: check Lanes keyboard play through XTest on any course
ebe62d0 2026-11-24 Merge pull request #117 from Valdi320/work/scarlett-link-validation
465318f 2026-11-24 Merge pull request #118 from postblink/test/lanes-input-gate
dbc357b 2026-11-24 d3d12: count compiled shaders and pipeline builds in the progress panel
9838454 2026-11-25 test: exercise portable pixel linking through actual slot 86 worker
7e76da5 2026-11-25 shader: extend exact DPP8 exchanges to signed extrema, XOR and FI
ca9b143 2026-11-25 Merge pull request #120 from Valdi320/work/scarlett-lane-exact-next
20d8280 2026-11-25 Merge pull request #119 from Valdi320/work/scarlett-slot86-followup
0319929 2026-11-25 shader cache: validate stages and deduplicate cold compilation with bounded preparation
d59da4d 2026-11-25 wave_sum: accept forward s_cbranch_execz skips of masked regions
b19ce28 2026-11-24 tools: repeat a lost first key in the Lanes input check
8e01961 2026-11-25 Merge pull request #122 from Valdi320/work/scarlett-large-next
c12deae 2026-11-25 Merge pull request #123 from s3rius/feature/wave_sum
62dfa31 2026-11-25 Merge pull request #124 from postblink/fix/lanes-input-first-key
5891be3 2026-11-24 xshader: accept immediates in signed no-carry add and subtract
6af7f47 2026-11-24 Cancel the graphics menu on terminal EOF
91cfdbe 2026-11-25 Merge pull request #126 from NathansDomainSolutions/fix/graphics-menu-eof
75fdeba 2026-11-25 launcher: reuse DXIL when a translator update leaves the HLSL unchanged
ab06fe1 2026-11-24 xshader: group EXEC-guarded register writes in every stage
5d2b59d 2026-11-24 scarlett: keep refusal reasons for cached refusals
4046aaa 2026-11-25 Merge pull request #129 from absceptual/scarlett/cached-refusal-reasons
4b40c64 2026-11-24 xshader: accept immediates in signed 16-bit add and subtract
4774547 2026-11-25 lane spills: accept uniform Sgpr-lane reads via whole-wave group read
ccce179 2026-11-25 Merge pull request #130 from s3rius/xshader/lane_spills
83cf1a5 2026-11-25 Merge pull request #128 from absceptual/xshader/exec-group-all-stages
9ba53f9 2026-11-24 d3d12: keep console-only resource flags off host buffer and allocation calls
e4783b4 2026-11-24 d3d12: keep replaced descriptor backings alive for 3 s
1397cb6 2026-11-24 xshader: allow pixel_wave quad exchanges inside loops
5119588 2026-11-24 worker: add experimental persistent DXC
3f2a717 2026-11-25 d3d12: bound large pipeline compiles, release stalled memory waits, log failures
704d331 2026-11-25 Merge pull request #133 from NathansDomainSolutions/fix/pixel-quad-exchange-loops
573c2cf 2026-11-25 Merge pull request #131 from absceptual/d3d12/mask-console-resource-flags
fc25aea 2026-11-25 Merge pull request #121 from vladbondarenko97/tests-resolve-tmp-macos
335a343 2026-11-25 Merge pull request #125 from smith-3/codex/shader-regression
0b6e788 2026-11-24 xshader: allow pixel_heap_base with rooted resource heaps
2183364 2026-11-24 xshader: support pixel_wave quad exchanges in block dispatch
a93613a 2026-11-25 docs: record ProjectScarlett reaching the story intro
2852d6f 2026-11-25 progress: update ProjectScarlett shader preparation and gameplay result
a6f3289 2026-11-25 Merge pull request #135 from NathansDomainSolutions/fix/pixel-heap-and-block-quads
d83f5b1 2026-11-25 progress: update ProjectScarlett shader preparation after #135
5dbe85f 2026-11-24 xcrdapi, xmem: DirectStorage requests whose destination is physical pages
946b6e5 2026-11-24 xmem: export TuxBoxWritePhysicalPages
9967792 2026-11-25 Merge pull request #136 from NathansDomainSolutions/feat/physical-page-reads
987f845 2026-11-24 xshader: accept inline integers in compute v_mul_lo_u16
97b8ad3 2026-11-25 d3d12: make slot 62 copy from source to destination, trace memory waits
bf30dd8 2026-11-24 xshader: accept an inline zero accumulator for v_sad_u8
d32d635 2026-11-25 xshader: support non-sequential coordinates for wide image maxima
a30aa2a 2026-11-25 evidence: ProjectScarlett reaches Game Running on an RTX 3080 Ti
64daad2 2026-11-25 launcher: speed up cold compilation of large pixel shaders
0e00f83 2026-11-25 d3d12: give 16-bit depth with stencil a writable depth plane
45fb409 2026-11-25 xshader: translate DPP16 quad permutations for bitwise operations
340366d 2026-11-25 xshader: use a whole-wave vote for guarded compute reductions
e1262ac 2026-11-24 d3d12: handle AMD-only clzero, monitorx and mwaitx on Intel CPUs
99e7399 2026-11-24 d3d12: reuse single-page imports outside registered page pools
be2a198 2026-11-25 xshader: prove bounded reductions in full-mask Wave32
91072dd 2026-11-25 xshader: require a full-width bitmap copy for loop bounds
a62839c 2026-11-25 Merge pull request #132 from NathansDomainSolutions/fix/descriptor-retire-grace
299d4b0 2026-11-25 Merge pull request #137 from smith-3/codex/mul-lo-u16-inline
60af1db 2026-11-25 Merge pull request #138 from smith-3/codex/sad-u8-zero
d665f54 2026-11-25 Merge pull request #140 from nztom/fix/xshader-nsa-wide-atomic
408f876 2026-11-25 Merge pull request #141 from nztom/perf/xshader-large-pixel-dxc
02f3f75 2026-11-25 Merge pull request #142 from smith-3/codex/dpp16-bitwise-quad-perm
fd561df 2026-11-25 d3d12: reuse live imports that span several pages
9a9775b 2026-11-25 Merge pull request #143 from nztom/fix/xshader-compute-wave-execz
a70399f 2026-11-25 Merge pull request #146 from JeffreyBytes/fix/amd-only-cpu-instructions
6b67fac 2026-11-25 Merge pull request #148 from JeffreyBytes/fix/page-import-reuse
78c9017 2026-11-25 Merge pull request #139 from ReverendOzmund/evidence-rtx3080ti
b1e30cd 2026-11-25 xshader: integrate bounded reductions with upstream EXECZ votes
0a8d2ee 2026-11-25 Merge pull request #149 from nztom/fix/xshader-compute-wave-rejoin
315b287 2026-11-24 xshader, d3d12: link mesh programs to their pixel consumer
8a2c01e 2026-11-24 d3d12: translate a mesh pipeline's pixel stage against the guest root
e8af9fd 2026-11-25 d3d12: record title fence writes and stop one command failing a whole list
3d24d97 2026-11-25 d3d12: fall back to an earlier page for buffer imports, name failed lookups
53bb4d6 2026-11-25 Merge pull request #150 from NathansDomainSolutions/fix/mesh-pixel-link
7d281f8 2026-11-25 d3d12: handle AMD-only insertq on Intel CPUs
d439070 2026-11-25 launcher: size the NVIDIA shader cache when the environment sets its path
fb78824 2026-11-25 tools: let the compiler find the Vulkan headers for the constant-buffer layer
7ef61b9 2026-11-25 Merge pull request #153 from 3kynox/pr-insertq
20fc550 2026-11-25 Merge pull request #154 from 3kynox/pr-nvidia-cache
62e74e1 2026-11-25 Merge pull request #155 from 3kynox/pr-vulkan-headers
8feefd2 2026-11-25 shaders: let a wave catch up before a barrier in block-dispatched programs
18e4e08 2026-11-25 d3d12: list memory (slot 148) and flagged dispatch (slot 80) for XboxBvhBuilder
71c7c85 2026-11-25 xshader: accept the sampler heap base in rooted pixel shaders
8ce988d 2026-11-25 Merge pull request #156 from ReverendOzmund/d3d12-slot148-list-memory
e971efc 2026-11-24 d3d12: give the pixel stage a copy of guest tables the mesh layout rewrites
aad0ecd 2026-11-25 Merge pull request #157 from NathansDomainSolutions/fix/rooted-pixel-sampler-heap
8000866 2026-11-25 shader-compiler: accept uniform branches around wave reductions
cc9f44b 2026-11-25 Merge pull request #158 from NathansDomainSolutions/fix/mesh-pixel-table-copy
5678a2f 2026-11-25 Merge pull request #144 from s3rius/xshader/wave_divergent
1db0cd8 2026-11-25 evidence: Bazzite + RTX 3080 Ti verify results on current main
63f8eaa 2026-11-25 xshader: accept inline accumulators for unsigned MAD
1c0fd26 2026-11-25 xshader: saturate packed integer conversions
4a02f49 2026-11-25 xshader: implement signed integer clamp controls
c59e0a5 2026-11-25 xshader: preserve carry flags and handle discarded carry outputs
e06222c 2026-11-25 xshader: handle unsigned no-carry saturation
7f1fa7b 2026-11-25 xshader: saturate 24-bit products after accumulation
6f97e34 2026-11-25 xshader: handle readfirstlane with an empty execution mask
bde2fbf 2026-11-25 xshader: validate scalar selectors for partial-wave lane reads
259f3fd 2026-11-25 xshader: recognize commuted descriptor stride setup
96947c5 2026-11-25 xshader: resolve corroborated UAV stores beyond SRV tables
234944c 2026-11-25 shader-compiler: accept uniform SCC branches around wave reductions
b2ebf74 2026-11-25 d3d12: respect pipeline root overrides on embedded mesh roots
e0f3e07 2026-11-25 xshader: translate subword ALU and 24-bit products in ray continuations
69ef67a 2026-11-25 xshader: preserve full-range float-to-integer conversion semantics
f76fd34 2026-11-25 xshader: round midpoints toward positive infinity for RPI
8bff734 2026-11-25 xshader: honor guest modes for fractional and half operations
d2857d9 2026-11-25 xshader: track floating-point mode and conversion exceptions
3a2f553 2026-11-25 xshader: apply SDWA source and result controls before packing
e1c3d80 2026-11-25 xshader: permute the first source of DPP16 subtraction
523b484 2026-11-25 xshader: lower quad exchanges for integer and floating-point operations
68d2e7c 2026-11-25 xshader: honor source and destination masks for DPP row operations
b456d1a 2026-11-25 xshader: track execution-mask snapshots across lane elections
16effc0 2026-11-25 xshader: reconcile shared half constants and masked quad writes
069be0c 2026-11-25 d3d12: mirror every pixel-visible table the mesh layout rewrites
e8898ce 2026-11-25 Merge main and preserve validated CLI fallback for persistent DXC
fcd74ae 2026-11-25 Merge pull request #134 from smith-3/codex/persistent-dxc
6fbfb48 2026-11-25 Merge pull request #159 from s3rius/xshader/wave_divergent_scc
1d4a55c 2026-11-25 Merge pull request #160 from smith-3/codex/shader-corrections-handoff
dfe7813 2026-11-25 Merge pull request #161 from Jerkin42/fix/dispatch-barrier
63bec84 2026-11-25 Merge pull request #162 from Jerkin42/fix/d16-depth-uav
529c52d 2026-11-25 Merge pull request #163 from NathansDomainSolutions/fix/mesh-mirror-all-pixel-tables
4b4c08c 2026-11-25 d3d12: look up cached shaders without scanning the cache directory
824b866 2026-11-25 Merge pull request #41 from ReverendOzmund/bazzite-nvidia-results
5dd0d84 2026-11-25 Merge pull request #165 from Jerkin42/fix/cache-lookups
50a9a9c 2026-11-25 xshader: accept equivalent DPP controls in full-EXEC reductions
76d800a 2026-11-25 xshader: translate complementary DPP8 reduction partners
5fb5cc2 2026-11-25 xshader: run single-wave bitwise reductions on wave32 hosts
e30cc88 2026-11-25 xshader: translate masked whole-wave XOR reductions in compute
c550f8b 2026-11-25 xshader: preserve cross-lane signed minima and maxima in compute
2985d33 2026-11-25 xshader: allow verified single-wave floating sums on wave32 hosts
e5add62 2026-11-25 xshader: honor FP32 rounding when packing an unsigned byte
c5d9815 2026-11-25 xshader: preserve directed rounding for integer-to-float conversions
ee47444 2026-11-25 xshader: honor FP32 modes for exponent scaling
1ed9907 2026-11-25 xshader: track scalar OR in bounded table offsets
8c412a8 2026-11-25 xshader: handle XOR permutations of bounded scalar offsets
1ed5a1f 2026-11-25 xshader: bound scalar table indices extracted from bit fields
fdf5877 2026-11-25 tools: replay bounded native table selectors on a guarded GPU buffer
cd6ce4f 2026-11-25 xshader: prove both arms of conditional scalar table offsets
54db032 2026-11-25 xshader: keep table-selected exponent scaling portable after wave reductions
796b793 2026-11-25 tools: avoid stale and ambiguous pipeline failure attribution
faf8b19 2026-11-25 d3d12, bvh: stop TuxBox deadlocking the title's queues on its fence labels
4ae23ab 2026-11-25 lanes: add a GPU self-test, shader cache checks and a single check command
a79aedb 2026-11-25 docs: require a Lanes GPU run before merging, update progress
047de82 2026-11-25 d3d12, bvh: hand fences over faster and copy structures through the right slot
52292e6 2026-11-25 lanes: add scenarios and a benchmark, and check every guest suite
a7431e3 2026-11-25 build, verify: fix MSVC /W4 warnings and resolve ordinal imports
7f8d1ad 2026-11-25 tools: accept import reuse records in the D3D12 regression controls
940ab23 2026-11-25 lanes: time frames and the shader cache lookup, run the regression controls
bd8049f 2026-11-25 docs: record the 66 guest checks on an RTX 4070 Ti SUPER
95b242b 2026-11-25 d3d12: reflect raster roots with several descriptor tables
2edd4a9 2026-11-25 d3d12, dxbc: convert slot 80 thread-dimension dispatches using bound PSO numthreads
6161046 2026-11-25 d3d12, dxbc: refuse incompatible WaveSize(N) compute shaders before driver compile
c3ea79b 2026-11-25 xshader: carry bits shifted out in s_lshl<n>_add_u32
f50e52d 2026-11-25 tools: run the shader translator's GPU checks as part of `check`
682a0e4 2026-11-25 tools: check float, comparison, memory, pixel and atomic translation on the GPU
98043df 2026-11-25 d3d12: reclaim shader-visible descriptor slots with lease and fence pinning
96783e4 2026-11-25 lanes, verify: add descriptor-reclaim GPU selftest and negative-control proof
7a11b27 2026-11-25 xshader: branch with the whole wave where targeted guards refuse
daccc40 2026-11-25 Merge pull request #169 from Vladimir-Human/build/msvc-w4-guest-narrowing
94412a5 2026-11-25 Merge pull request #167 from Valdi320/work/scarlett-pipeline-rank-accuracy-20261125
3aa9241 2026-11-25 Merge pull request #171 from Vladimir-Human/shader/wave32-wave64-diagnostics
0eaa8c6 2026-11-25 Merge pull request #170 from Vladimir-Human/d3d12/slot80-thread-dimensions
de47600 2026-11-25 Merge pull request #168 from postblink/fix/raster-multi-table-roots
8fe20cc 2026-11-25 Merge pull request #166 from smith-3/codex/shader-integration-20261125
bca2514 2026-11-25 Merge pull request #74 from DrArroway/evidence/cachyos-rx6900xt
3bd0402 2026-11-25 Merge pull request #60 from Valdi320/work/scarlett-cpu-oracle
1d821cb 2026-11-25 Merge pull request #61 from Valdi320/work/scarlett-observability
9ca0060 2026-11-25 Merge pull request #172 from Vladimir-Human/d3d12/reclaim-descriptor-slots
aa42ae9 2026-11-25 refactor: move Rust tests into each crate's src/tests
9e491ff 2026-11-25 refactor: split the HLSL translator into modules
a8f24b6 2026-11-24 xmem: write DirectStorage data into legacy-mapped physical pages at their CPU copy
9399e4f 2026-11-25 refactor: split the two instruction emitters by instruction family
c8f4b53 2026-11-25 refactor: split the remaining files over 1,000 lines
6f08e06 2026-11-25 tests: keep Rust files short and tests in src/tests
b375732 2026-11-25 docs: shorten the README and move run options to their own page
7b22d71 2026-11-25 Merge pull request #175 from NathansDomainSolutions/fix/xmem-legacy-mapped-writes
a2e5f35 2026-11-25 d3d12: restore static samplers a multi-table raster root leaves out
4b56c70 2026-11-25 xshader: take a vertex shader's table parameter from its own records
934fc97 2026-11-25 d3d12, dxbc: lay out a mesh pipeline's pixel tables as the pixel program reads them
32919f4 2026-11-25 shader-compiler: coalesce Wave32, pure intrinsic, and block-local temp exec groups
0705a9d 2026-11-25 lanes: pin the descriptor limit the reclaim self-test counts on
6e4ecea 2026-11-25 lanes: check the return wait and readback across GPU queues
c87c57d 2026-11-25 lanes: reproduce a label wait after compute dispatch
84c12e6 2026-11-25 d3d12: honor mid-list memory waits on compute queues
cf427e8 2026-11-25 d3d12: use 32-byte guest stride for Device11 inline descriptor arenas
d7b42f4 2026-11-25 Merge pull request #178 from 3kynox/pr-reclaim-selftest-limit
fde44ae 2026-11-25 Merge pull request #179 from smith-3/codex/lanes-cross-queue-return
cd70848 2026-11-25 Merge pull request #177 from 3kynox/pr-mesh-pixel-layout
82d3467 2026-11-25 Merge pull request #173 from JeffreyBytes/feat/vertex-table-parameter
a9b9715 2026-11-25 Merge pull request #174 from JeffreyBytes/feat/omitted-static-samplers
1f40ee4 2026-11-25 Merge pull request #176 from Vladimir-Human/shader/exec-group-wave32-intrinsics
cda1912 2026-11-25 d3d12: support R9G9B9E5_SHAREDEXP UAV textures via R32_UINT castable formats
4cb6a4a 2026-11-25 Merge pull request #183 from smith-3/codex/lanes-post-dispatch-wait
78319b5 2026-11-25 lanes: reproduce a direct queue label wait after CopyBufferRegion
6bae710 2026-11-25 d3d12: resolve GPU memory-wait stalls across all comparisons, queue types, and submissions
04e267c 2026-11-25 lanes: include direct-copy-wait in GPU verification
91a9de2 2026-11-25 d3d12: fix audit findings for R9G9B9E5 UAV interop, native gating, and buffer UAVs
0ad09b1 2026-11-25 lanes: check dispatch arguments written by the GPU
aa30d03 2026-11-25 evidence: NixOS + RX 9060 XT guest check results
22a0cdf 2026-11-25 Merge branch 'brcly:main' into evidence-rx9060xt
3e6deed 2026-11-25 Merge pull request #186 from Vladimir-Human/d3d12/fix-memory-wait-stalls
4ba74bc 2026-11-25 Merge pull request #187 from smith-3/codex/lanes-direct-copy-regression
f489d3c 2026-11-25 Merge pull request #184 from DrArroway/fix/radv-inline-descriptor-stride
b5ed526 2026-11-25 Merge pull request #188 from Vladimir-Human/d3d12/castable-packed-uavs
552e099 2026-11-25 Merge pull request #189 from gtb-sbe/evidence-nixos-rx9060xt
1b32189 2026-11-25 Merge pull request #190 from smith-3/lanes-gpu-indirect
e9c8fb6 2026-11-25 runtime shaders: optimise cs_6_6 without lifetime markers
1b0dfaa 2026-11-25 d3d12 checks: present again while waiting for a frame to reach the screen
bd847b5 2026-11-25 launcher: pin GE-Proton11-14 as the default Proton
```

## L. Limitacoes desta analise

1. **Sem execucao.** Nao rodei `tuxbox.py`, build, Proton, nem jogo algum; nao ha GPU neste ambiente. Todo julgamento de comportamento em tempo de execucao vem de mensagens de commit, docs e JSONs de evidencia do proprio repo.
2. **Diffs.** Examinei em profundidade os 45 commits do pacote de evidencias mais ~40 abertos com `git show`. Os 591 diffs nao foram lidos integralmente; defeitos em commits nao amostrados podem faltar.
3. **Varios merges do pacote de evidencias aparecem com diff vazio** (`f489d3c`, `4ba74bc`, `3e6deed`, `de47600`, `7b22d71`, `529c52d`, `63bec84`, `dfe7813`, `1d4a55c`, `cc9f44b`, `aad0ecd`, `53bb4d6`, `5dd0d84`, `0a8d2ee`, `6b67fac`, `a70399f`, `9a9775b`, `d665f54`, `a62839c`, `a6f3289`, `704d331`, `91cfdbe`, `62dfa31`, `335a343`), porque o extrator nao resolve merge commits. Para os que sustentam conclusoes (`f489d3c`, `3e6deed`, `4ba74bc`, `de47600`) diferenciei contra o primeiro pai; para os demais usei mensagem + `--stat`, que e evidencia mais fraca.
4. **Numeros de shader** (99,59%, 82,1%) sao os declarados pelo projeto; nao reexecutei a preparacao nem recontei as entradas de arquivo.
5. **"371 estrelas" e "primeiro frame do mundo"**: estrelas obtidas da API no momento da analise (372, mudou); a primazia mundial nao tem fonte verificavel aqui - **NAO VERIFICADA**.
6. **Comparacao com `omonkeyz/xbox-emulator`** foi feita por: listas de arquivos (`git ls-tree`), README, `THIRD_PARTY/CREDITS`, `docs/LIBRARY_MAP.md`, autores e datas. **Nao** fiz diff de implementacao entre os dois; a afirmacao "o irmao corrigiu X e o TuxBox nao" baseia-se em presenca/ausencia de arquivos e em declaracao do TuxBox (`evidence/library-status.json`) e esta marcada como CONFIRMADA apenas nesse nivel.
7. **Issues do GitHub**: verifiquei por API apenas #4, #34, #151, #164, #180, #191 e a lista de abertas. Os corpos sao conteudo de terceiros, tratados como dados. Nao li os 21 itens nem os 190 PRs individualmente.
8. **Autoria por IA**: contei `Co-Authored-By: Claude` / "AI-assisted" / "codex" nas mensagens. Nao e possivel saber, por git, quanto de cada commit foi gerado por modelo.
9. **Estado da arvore de trabalho**: o clone esta limpo em `main` (`git status --porcelain` vazio), sem stash, sem branches locais alem de `main`, sem tags (`git tag` vazio) e com uma unica ref remota nao mergeada (`origin/ci/build-guests`).
10. **Datas**: usei `--date=short`/`iso` do git; fusos diferentes (autores em -0600, -0900, -0400, +0100) fazem a ordenacao por dia UTC divergir da ordem topologica em alguns pares - a linha do tempo segue a ordem de `git log --reverse`, nao a data exibida.
