# DexNom - analise de DeepSeek V4.1 Flash

## A. Identificacao

- **Nome / URL / natureza:** `DexNom/VIctory` - https://github.com/DexNom/VIctory. Fork de `Marvisak/VIctory`.
  Raiz comum: `f4504f6413ce4509604dad66cdd689000a0204dc` ("Initial commit", Marvis, 2026-11-23 01:15:26 +0100).
  O fork se apresenta, no proprio README, como **"DexNom Runtime is a community fork of Marvisak's VIctory"**, republicado a
  partir de um renderizador Radeon privado e creditando `ZSlayerHQ/victory-world-runtime` como baseline de renderizacao
  (`README.md` @ HEAD, secao *Credits and licensing*).
- **Branch analisada:** `main` (`HEAD` = `origin/main` = `2e01ab4`). Outras refs presentes:
  `origin/backup/main-before-intro-2026-11-25` -> `921d8d3` e `origin/snapshot/intro-tested` -> `b8ff22e`
  (`git for-each-ref refs/remotes`). As duas refs marcam, nominalmente, o estado *antes* e *depois* do trabalho de
  "intro" (cutscene de abertura) - sao checkpoints humanos, nao branches de desenvolvimento.
- **Primeiro commit ("commit 0"):** `f4504f6` | 2026-11-23 01:15:26 +0100 | Marvis | *Initial commit*.
- **Ultimo commit:** `2e01ab4760db91820ef5f9a9f85761f83cd6b5d0` | 2026-11-25 11:46:07 -0600 | Codex |
  *Keep cached test textures within their reserved guest memory*.
- **Periodo coberto:** 2026-11-23 01:15 (+0100) a 2026-11-25 11:46 (-0600) - cerca de 58 horas de calendario, 3 dias com commit
  (64 / 322 / 161 commits por dia).
- **Volume:** 547 commits em `HEAD` (547 em todas as refs, 0 commits so em outras branches), **228 merges**, 319 commits nao-merge,
  283 commits na primeira linha de ancestrais. **195 arquivos** versionados, dos quais **160 `.rs`** somando **120.287 linhas**
  de Rust, com **666 funcoes `#[test]`** (contagem de `^\s*#\[test\]` em `HEAD`).
- **Workspace:** 10 crates (`victory-core`, `-formats`, `-game`, `-gpu`, `-hle`, `-loader`, `-pe`, `-pipeline`, `-shader`) +
  `apps/victory` + `xtask`.
- **Autores (topo):** Marvis 267 (221 + 46), Duff 20, DexNom 10, Manuel Perez 9, TG0000 9, Maraco91 9, geiyiliyabumo 9,
  Luiz Neves 8, marquezini 7, quaternion 7, r4. 6, Codex 5, AnoHunted 5, tedin7 5. Ao todo **107 autores distintos**, dos quais
  **87 com 4 commits ou menos** - ou seja, o repositorio e a soma de uma multidão rasa sobre um nucleo de um.
- **Profundidade:** indice completo dos 547 commits na secao K; ~45 commits examinados com `git show` integral (todos os do
  overlay DexNom/Codex, todos os que tocam os defeitos catalogados, e os merges cujo diff realmente existe).

### Correcao de um fato da base de analise

`_fatos/FORK_DIVERGENCIA.md` afirma, como "achado previo e decisivo", que *"o HEAD do Marvisak NAO e ancestral de nenhum dos
cinco forks (`git merge-base` falha)"*. **Isso nao se reproduz neste repositorio.**

```
$ git merge-base --is-ancestor 70564ae08c82d43df5fad97b08d34628b07b1d9d HEAD ; echo $?
0
$ git rev-list --count 70564ae..HEAD
306
$ git rev-list --count 70564ae
241
```

`70564ae` ("README: status of 2026-11-24", Marvis) e ancestral de `HEAD`, e a base tem exatamente 241 commits como o documento
descreve. O fork **acrescenta 306 commits** sobre essa base. Classificacao: **CONFIRMADA** (verificacao direta). A conclusao
pratica muda muito: nao houve reescrita de historico entre DexNom e a base; houve **acrescimo e depois sobrescrita de conteudo**
pela importacao da linha privada (secao H). A afirmacao do documento pode valer para os outros forks, mas nao para este; nao
consegui reproduzir a falha de `merge-base` em nenhuma combinacao local.

## B. Resumo tecnico

**Proposito.** Rodar um executavel Xbox Series (o jogo do repositorio e *Grand Theft Auto VI*, Series X 1.0.2.1) em Windows x64:
carregar o PE do titulo, emular as APIs Xbox por HLE, traduzir shaders GFX10 (RDNA2) para DXIL e executar o trabalho de GPU sobre
D3D12. O alvo declarado nao e emulacao ciclo-a-ciclo, e **compatibilidade + traducao**: o programa do console e traduzido uma vez
e cacheado (`emit`, `shader_cache`, `pipeline/library`), e o estado do console e reinterpretado em objetos D3D12.

**Modulos e responsabilidades**

| Crate | Papel |
|---|---|
| `victory-loader` | Carregador PE (imports, TLS, relocacoes), threads do titulo, emulacao de instrucoes ausentes no host (`crash.rs`: `clwb`, `clzero`, `rdpru`, SSE4a), captura de stacks, log de crash |
| `victory-pe` | Parser PE: secoes, export directory, import lib, layout |
| `victory-hle` | Modulos de API do console: `xmem`, `xg`, `xuser`, `xgamesave`, `xstore`, `xpackage`, `xasync`, `xtaskqueue`, `display`, `gameinput`, `dstorage`, `audio`/XMA, `net`, `files`, `kernel`, `lifecycle` |
| `victory-shader` | Decodificador GFX10 (`gfx10/`), IR e passes (`ir.rs`, `structure.rs`), emissor DXIL (`emit/`), referencia Rust do teste de raio (`bvh.rs`), `dxbc.rs` |
| `victory-gpu` | Implementacao D3D12 da API de GPU do console: descritores/heaps, listas, draw, texturas e staging, memoria do titulo (page table), fences, filas, sombras (shadow copies), readback, DRED, `pipeline/`, e o lado host do ray tracing |
| `victory-formats` | Texturas (layouts, tiling, BCPack), XMA (audio) |
| `victory-game` | Formatos do jogo: `sgd5` (banco de shaders), `pipelines`, `shaderpack` |
| `victory-pipeline` / `pipeline/library` | Cache em disco de pipelines (D3D12 pipeline library serializada) |
| `apps/victory` | CLI: `run`, `translate` (traducao offline + cache), `census`, `inspect`, `shader`, `play_launcher` |
| `xtask` | Politica de repositorio (`policy.rs`: varredura de assets/binarios) |

**Arquitetura de execucao (o que e especifico deste fork).** O console tem uma unica memoria; o PC nao. O fork resolve isso com
tres mecanismos que atravessam shader e host:

1. **Page table de memoria do titulo.** Um diretorio, uma entrada por 64 KiB de espaco de endereco do titulo, leva o shader a
   copia host daquela pagina; loads/stores/atomicos passam por ele (`docs/fork-ray-tracing.md`, item 2). O host mantem no maximo
   **1.023 slots de copia** (limite citado em `BENCHMARK.md`, secao *Known gaps*).
2. **Escrita de convidado rastreada por pagina.** `writeback.rs` (dono dos bytes enquanto escritas de GPU estao em voo),
   `inplace.rs` (heap D3D12 aberto sobre memoria do proprio titulo), `shadow.rs` (copias de ranges declarados como escritos).
3. **Ray tracing do proprio titulo, emulado.** O jogo nao usa API de RT: ele traz BVHs pre-construidas no formato do console e
   usa duas instrucoes de hardware (`image_bvh64_intersect_ray`, `image_bvh_intersect_ray`); o fork emula essas instrucoes em
   DXIL (`emit/intersect.rs`, referencia em `bvh.rs`) e traduz os 52 programas construtores de estrutura (`docs/fork-ray-tracing.md`).

**Estado atual (HEAD).** O runtime inicia o jogo, chega a Chapter 1 City, e a traducao cobre **25.742 de 25.758 shaders** do
arquivo do jogo (15 recusas restantes, `BENCHMARK.md`). O caminho de ray tracing funciona com o construtor dinamico **desligado
por padrao** (`VICTORY_BVH_BUILDER=1` liga); com ele ligado, City roda a **~1 FPS** na RX 7900 XTX (969 ms de frame medio,
medido). Iluminacao interna, cobertura de shaders do jogo inteiro e o resultado visual do RT por passe continuam abertos
(`BENCHMARK.md`, `docs/fork-ray-tracing.md`).

## C. Linha do tempo

### Fase 0 - bootstrap (2026-11-23, 01:15-~03:00, 8 commits)

| Commit | Assunto | Papel |
|---|---|---|
| `f4504f6` | Initial commit | raiz |
| `12caefe` | GFX10 operand fields and SP3 text; FSSM wave size | primeiro decodificador GFX10 |
| `754619f` | Run Minecraft: runtime HLE, D3D12.X device on the host GPU, shader IR | primeira prova de conceito (outro titulo) |
| `d032b8a` | Title DLLs at run time: find by name, map on first LoadLibrary | carregamento tardio de DLLs do titulo |
| `4b25269` / `bc30709` | Thread notices for title images / leave out Rust's std threads | instrumentacao de threads |
| `2849477` | **Run Grand Theft Auto VI from first boot into the world** | marco: o jogo roda |
| `015d45b` | README: status, subsystem table, how to run, debugging switches | documentacao minima |

`2849477` e o commit mais importante do repositorio: e o estado que, mais tarde, sera usado como **baseline publico** de tudo
que o fork republica (`Initialize-FormatTables.ps1` baixa as tabelas BCPack exatamente desse hash).

### Fase 1 - construcao do runtime, base Marvisak (2026-11-23 -> 2026-11-24, ate `70564ae`, 241 commits)

Commits arquiteturais grandes, um por subsistema, na ordem em que aparecem:

- `27c3dc9` *Shaders and GPU: title memory at raw addresses through a page table* - nasce a page table (o mecanismo central).
- `0b35b7f` *FidelityFX on host and console paths, 64-bit image atomics, the BVH intersection, lane masks, bindless strides* -
  uma unica commit com cinco frentes; traz a interseccao de BVH para o tradutor.
- `35b054e` *GPU: pipelines off the render thread and cached; indirect execution on* - execucao indireta e cache de pipelines
  (`pipeline/library`), incluindo `write_atomically` (semente de D-11).
- `286e33f` *Game formats: the shader pack* / `7389303` *effects, techniques and passes* - leitura do banco de shaders do jogo.
- `c47b529` *Shader warm-up (translate --cache), a cache keyed by the translator, pipeline queue fixes* - pre-aquecimento e chave
  de cache por fonte do tradutor.
- `6d021fe` *GPU: world frame cost* / `7000573` *Diagnostics: title memory peeks, slot states...* - instrumentacao de custo.
- `77053a0` *Shaders: accuracy fixes for discards, f16 rounding, denormals, med3 and signed zeros* - rodada de correcao numerica.
- `70564ae` *README: status of 2026-11-24* - **HEAD da base**.

### Fase 2 - a leva de merges (2026-11-25, 01:07-01:24, 287 commits, 137 merges, 98 arquivos)

Entre `70564ae` e `4118f33` o repositorio recebe, em ~17 minutos de relogio, **137 `Merge pull request #N`**, com numeros de PR
de **#12 a #1733** - uma varredura de integracao de contribuicoes acumuladas. Exemplos representativos (todos com diff real ou
mensagem substancial):

`ed0dbfd` (*GPU: copy title writes under ranges a submission's shaders write*), `0601955` (*a helper lane's v_readfirstlane reads
its own value*), `4fc9d70` (*keep `Staging::detile` inside the staging buffer*), `be824d9` (*serialize pipeline library atomic
writes*), `274f698` (*guard clwb_loop against u64 overflow*), `4ab20fc` (*indirect budget charged in record order*), `c1c3281`
(*descriptor wraparound scan*), `121412b` (*verify unwind when RtlCaptureStackBackTrace returns empty*), `d4d1822` (*XMA: end a
frame at its data instead of looping forever*), `60d2cf9` (*clwb in the last bytes of a page*), `b8dbcba`, `52f0a09`, `eb8067f`,
`71e8866`, `0de9efc`, `0b607b1`.

Esta e a fase que explica a diferenca de tamanho entre este fork (547) e a base (241): **os 306 commits a mais sao
majoritariamente estes merges**. 98 arquivos foram tocados.

### Fase 3 - trabalho de Marvis sobre readback (2026-11-25, 01:24-02:22)

| Commit | Assunto |
|---|---|
| `42363ab` | Tests: hardware-only shader checks skip on a machine without a GPU |
| `ccfb029` | Tests: a machine with no GPU has no hardware adapter to check shaders on |
| `cd94d35` | **GPU: shaders mark the pages they write through views; copy-backs compare only those** (2,8-4,5 GB/frame -> somente paginas marcadas) |
| `940d32f` | GPU: a marked page's copy-back does not save the page for its own copy |

`cd94d35` e a maior correcao de desempenho do historico compartilhado: substitui o snapshot integral dos ranges "declarados como
escritos" por uma **bitmap por pagina de 64 KiB marcada pelo proprio shader traduzido** (`emit::Options::view_marks`).

### Fase 4 - a publicacao DexNom (2026-11-25, 06:18-07:43)

| Commit | Assunto | Efeito |
|---|---|---|
| `570a941` | **Publish DexNom Runtime Radeon sources with upstream credits and build instructions** | 170 arquivos, **+49.715 / -36.762**, 26 adicionados, 24 removidos, 120 modificados |
| `42b6565` | Enforce the source-only policy for every path on the public branch | `docs/asset-policy.md`, `xtask/src/policy.rs` |
| `ef8f4f1` | Build and install DexNom Runtime from the source ZIP with Windows Setup | `Setup.cmd`, `Setup-DexNomRuntime.ps1` |
| `8c7937a` | Verify setup dependencies with the built-in .NET SHA-256 implementation | troca `Get-FileHash` por `SHA256::Create()` |
| `ddb7332` | **Restore 40 shader variants and repair integer-target blending** | 40 shaders recuperados; bypass de blending em alvo inteiro |
| `fa007d1` | Correct Windows setup and attribution link separators | README |
| `76749ce` | **Recover voxel visibility shader when a wave64 mask is restored from scalar copies** | +1 shader |
| `5f4c3df` | **Improve City texture streaming and recover Radeon shadow pipelines** | 27 arquivos, +2.611/-349: `guest_address.rs`, `packed_mask.rs`, `USER_MEMORY_BASE`, streaming |

`570a941` nao e um commit de feature: e a **substituicao do conteudo da arvore publica pelo conteudo de uma linha privada**
(o proprio `b8ff22e`, depois, diz "sync ... onto public source baseline `921d8d3`"). Ele apaga 24 arquivos, entre eles o
`crates/victory-shader/build.rs` e 10 artefatos de teste/benchmark. Ver secao H.

### Fase 5 - preview de RT e reparos (2026-11-25, 08:51-11:46)

| Commit | Assunto | Papel |
|---|---|---|
| `bc3cd4b` | **Restore ray structure building and track guest shader writes precisely** | `writeback.rs` (321 linhas), `inplace.rs`, BVH: 2a metade do no, slots vazios, pagina ausente |
| `d71f197` | Add an opt-in Windows shortcut for experimental ray tracing | atalho de Start menu |
| `8cca4d2` | Document experimental RT launch and measured City frame times | `BENCHMARK.md`, `docs/fork-ray-tracing.md` |
| `921d8d3` | **Repair pixel quad sampling, grow ray buffer views, and recover sparse textures** | LOD, quad ops, `USER_MEMORY_BASE`, `mapped_bytes` |
| `b8ff22e` | **Export the tested Radeon renderer for the optional RT preview** | 15 arquivos, +1.899/-305; `material_demand.rs`, `typed_views.rs`; marcado por `snapshot/intro-tested` |
| `4775e8a` | Isolate the material readback test from concurrent frame advancement | `TEST_FRAME_LOCK` |
| `2e01ab4` | Keep cached test textures within their reserved guest memory | fecha o OOB do teste de `b8ff22e` |

A existencia de `backup/main-before-intro-2026-11-25` (-> `921d8d3`) e `snapshot/intro-tested` (-> `b8ff22e`) mostra que o time
tratou o "intro" como operacao de risco: checkpoint antes, snapshot depois.

### Fase 6 - congelamento

`HEAD` (`2e01ab4`) e **test-only**. Depois de `b8ff22e` o repositorio nao recebe mais mudanca de runtime: os dois ultimos commits
consertam o proprio teste que `b8ff22e` introduziu.

## D. Catalogo de defeitos

> Convencao: "introduzido por" e sempre um commit **deste** repositorio. Quando o codigo veio da arvore privada
> (`570a941`/`5f4c3df`/`b8ff22e`), o defeito pode ser mais antigo que a importacao; nesses casos digo isso explicitamente e
> rebaixo a confianca da atribuicao (nao do defeito).

---

### D-01 - Alvo de renderizacao inteiro com blending ligado derruba a criacao do pipeline

- **Descricao / sintomas.** O pipeline do "windshield wiper" (limpador de para-brisa) da City **falhava na criacao** na RX 7900 XTX.
  Nao havia crash: o D3D12 recusa criar o PSO, e o desenho nunca acontece.
- **Causa raiz.** O estado de blend do convidado era encaminhado ao host **sem inspecionar o formato dos alvos**. D3D12 valida
  `BlendEnable` contra *todos* os formatos ativos na criacao do pipeline; com um alvo inteiro (`R8_UINT`, `R32_UINT`, ...) na
  lista, a criacao falha. Alem disso, com `IndependentBlendEnable = false` o D3D12 le o slot 0 para todos os alvos, entao
  desligar o blend de um alvo exigia primeiro **expandir** o estado compartilhado (inclusive o write mask).
- **Introduzido por.** `570a941` - `git log -S"BlendEnable" -- crates/victory-gpu/src/pipeline/host.rs` lista **apenas** `570a941`
  antes de `ddb7332`: o encaminhamento do estado de blend para o PSO nasce com a importacao, sem filtro por formato de alvo.
- **Tentativas de correcao.** Nenhuma antes de `ddb7332`.
- **Correcao definitiva.** `ddb7332` - `GraphicsState::apply_integer_blend_bypass()` (`crates/victory-gpu/src/pipeline/host.rs`),
  chamada em `create_stream` (`pipeline.rs`). Expande `RenderTarget[..n]` quando o blend e compartilhado, liga
  `IndependentBlendEnable` se houver mais de um alvo, e zera `BlendEnable` somente nos alvos inteiros.
- **Arquivos.** `crates/victory-gpu/src/pipeline/host.rs`, `crates/victory-gpu/src/pipeline.rs`.
- **Evidencia.**
  - `git show ddb7332 -- crates/victory-gpu/src/pipeline/host.rs`: `graphics.apply_integer_blend_bypass();` e a lista de formatos
    inteiros.
  - `BENCHMARK.md` @ HEAD: *"On an RX 7900 XTX the original windshield-wiper pipeline failed creation with blending enabled on an
    integer target. The corrected pipeline created successfully with the same shaders and framebuffer formats. The other active
    targets retained their blending state and all three write masks were preserved."*
  - Teste `integer_target_bypasses_blending_and_preserves_mixed_target_masks` no mesmo commit (alvos `R32_FLOAT`, `B4G4R4A4_UNORM`,
    `R8_UINT`; verifica que os dois primeiros ficam intocados e o terceiro perde o blend).
- **Estado atual.** Corrigido e presente em `HEAD`.
- **Confianca.** **CONFIRMADA** (o relato de falha e do proprio repositorio; o teste fixa o comportamento).

---

### D-02 - Sampler estatico escolhido por desvio uniforme: "an untracked sampler"

- **Descricao / sintomas.** Shaders que escolhem entre **dois samplers imediatos** num desvio uniforme eram **recusados** na
  traducao. No arquivo do jogo: **56 de 25.758 shaders recusados**; os testes offline dessas 56 recusas recuperaram
  **37 pixel shaders + 3 mesh shaders = 40 variantes**.
- **Causa raiz.** `Lower::image` buscava o sampler com `self.descriptors.get(&(site, true))`, que so conhece **um** descritor por
  sitio. Quando o sampler vinha de um `Source::Phi` de duas constantes, a busca falhava com `Err("an untracked sampler")` e o
  shader inteiro era rejeitado - sem tentar as duas alternativas.
- **Introduzido por.** `2849477` - *este defeito vem da base, nao do fork*: `git log -S"an untracked sampler"` devolve **apenas**
  `2849477` ("Run Grand Theft Auto VI from first boot into the world"), e o lookup unico por sitio nunca mudou. O numero de recusas
  no arquivo do jogo (56) e medido no fork; na base nao ha medicao equivalente (**NAO VERIFICADO**).
- **Tentativas de correcao.** Nenhuma no historico antes de `ddb7332`.
- **Correcao definitiva.** `ddb7332` - `image()` reconhece o phi de duas constantes, **amostra duas vezes** com as derivadas
  originais (via novo `image_with_sampler`), e seleciona o texel no mesmo *edge* que forneceu o sampler do console
  (`set_phi` booleano escolhendo entre os dois resultados). Casos degenerados (um dos predecessores sem bloco) caem para o
  sampler daquele lado.
- **Arquivos.** `crates/victory-shader/src/emit/lower.rs` (funcoes `image` / `image_with_sampler`).
- **Evidencia.**
  - `git show ddb7332 -- crates/victory-shader/src/emit/lower.rs`: `if let Some(Source::Phi { block, args }) = sampler ...`
    seguido de dois `image_with_sampler` e `select`.
  - Teste `a_uniform_branch_can_choose_between_immediate_samplers` (mesmo commit) afirma `shader.samplers.len() == 2` e a
    presenca de `[0,0,0,0]` (wrap) e `[0x92,0,0,0]` (clamp).
  - `BENCHMARK.md`: *"Before the static sampler fix, the archive scan translated 25,702 shaders and refused 56."*
- **Estado atual.** Corrigido; a varredura completa passou a traduzir **25.742 de 25.758**; das 56 recusas originais, **41
  validaram** e **15 continuam recusadas** (ver G-02).
- **Confianca.** **CONFIRMADA**.

---

### D-03 - Mascara wave64 restaurada por dois `s_mov_b32` nao era reconhecida

- **Descricao / sintomas.** O shader de *voxel visibility* (visibilidade de voxels) era recusado. A mascara de lane de 64 bits
  chegava em dois registradores escalares, e o tradutor nao a associava de volta ao `v_cmp` que a produziu.
- **Causa raiz.** `Lanes::mask` (`crates/victory-shader/src/structure.rs`) procurava a mascara por identidade do par de valores.
  O passe de spill transforma o par em **dois `s_mov_b32`** (um por metade), entao o par reconstituido nao era igual a chave
  original e a mascara era derivada de novo - o que, em wave64 sobre wave32, da uma mascara errada.
- **Introduzido por.** `2849477` - *tambem vem da base*: `git show 2849477:crates/victory-shader/src/structure.rs` ja contem
  `fn mask(&mut self, k: Key)` com a busca por identidade em `self.masks` / `self.other` e sem qualquer tratamento de `s_mov_b32`
  de 64 bits para wave64.
- **Tentativas de correcao.** Nenhuma no historico antes de `76749ce`.
- **Correcao definitiva.** `76749ce` - para `wave64`, segue a cadeia de `s_mov_b32` (`move_source`) de cada metade e, **somente se
  as duas palavras vierem da mesma mascara conhecida e na ordem original**, registra a equivalencia (`self.masks.insert(k, m)`).
- **Arquivos.** `crates/victory-shader/src/structure.rs`.
- **Evidencia.**
  - `git show 76749ce -- crates/victory-shader/src/structure.rs`: bloco `if self.p.wave64 { let source = (self.move_source(k.0),
    self.move_source(k.1)); ... }`.
  - Teste `split_scalar_copies_across_a_branch_preserve_both_words_of_a_wave64_mask`, que **parametriza o caso negativo**
    (`(0xBE83_036A, false)`): se a metade alta for copiada de `vcc_lo`, a equivalencia NAO deve ser aceita.
  - `BENCHMARK.md`: *"A follow-up test recovered one more pixel shader used for voxel visibility. The corrected translator
    recognized both halves of a restored wave64 lane mask."*
- **Estado atual.** Corrigido e presente.
- **Confianca.** **CONFIRMADA** (o teste negativo e a prova mais forte: o commit nao apenas "aceita mais", ele restringe).

---

### D-04 - LOD calculado com logaritmo natural vezes log2(e), e clamp descartado

- **Descricao / sintomas.** Nivel de mip errado em amostragem com `image_sample*` que calcula LOD a partir de footprint. O efeito
  visivel e textura borrada ou serrilhada - nao ha crash, por isso passou.
- **Causa raiz.** O codigo fazia `lod = ln(texels) * LOG2_E`. A funcao DXIL usada (`unary(23, ...)`, `Log`) **ja retorna log2**;
  multiplicar por `log2(e) = 1,4427` superestima o LOD em 44,27% (`log2(x)*log2(e)` em vez de `log2(x)`). No mesmo trecho, o
  operando **clamp** da instrucao do console era simplesmente ignorado (`SampleLevel` nao tem clamp: o clamp tem de ser aplicado
  antes, como `max(lod, clamp)`).
- **Introduzido por.** `570a941` (`git log -S"LOG2_E"` lista `570a941` e `921d8d3` apenas).
- **Tentativas de correcao.** Nenhuma antes de `921d8d3`.
- **Correcao definitiva.** `921d8d3` - remove a multiplicacao (`let lod = self.unary(23, texels);`) e aplica
  `if has_cl { self.binary(35, lod, clamp) } else { lod }`.
- **Arquivos.** `crates/victory-shader/src/emit/lower.rs`.
- **Evidencia.** `git show 921d8d3 -- crates/victory-shader/src/emit/lower.rs`:
  `- // log2 = natural logarithm / ln 2 ...` / `+ // DXIL Log already returns log2 ... SampleLevel has no clamp operand; apply
  the guest instruction's minimum LOD before passing the level.`
- **Estado atual.** Corrigido.
- **Confianca.** **CONFIRMADA** para o erro de matematica e para a ausencia do clamp (o diff e inequivoco); **PROVAVEL** para o
  impacto visual concreto (nao ha medicao de imagem no repositorio).

---

### D-05 - `USER_MEMORY_BASE` respondia a base do buffer para as duas dwords; shader lia descritores fora do bind

- **Descricao / sintomas.** Leitura de descritores de pagina pela shader podia passar do que foi preparado para aquele bind: uma
  lista de comandos **posterior** publica mais paginas, e um shader que consultasse um indice acima do que o bind atual cobria
  lia lixo (ou uma entrada de outro recurso).
- **Causa raiz.** Duas falhas somadas.
  (a) No host, `user_memory_base` (dword 1) retornava `m.base` para **qualquer** dword; o dword 1 deveria ser a **contagem**
  (`m.count`). O shader portanto nao tinha como comparar o indice com o tamanho.
  (b) No tradutor, nenhuma comparacao de faixa era emitida: o indice ia direto para a leitura.
- **Introduzido por.** `570a941` - em `draw.rs`, `if kind == USER_MEMORY_BASE { return memory.map_or(0, |m| m.base); }` para
  qualquer dword. `5f4c3df` ampliou o uso da constante no tradutor sem corrigir o host.
- **Tentativas de correcao.** Nenhuma antes de `921d8d3`.
- **Correcao definitiva.** `921d8d3`, nos dois lados:
  `draw.rs`: `return memory.map_or(0, |m| if dword == 0 { m.base } else { m.count });`
  `lower.rs`: `let count = self.host_word(RootType::Other(USER_MEMORY_BASE), 0, 1); let in_range = self.s.m.cmp(c, pred::IULE,
  copy, count); let copy = self.s.m.select(c, in_range, copy, zero);`
- **Arquivos.** `crates/victory-gpu/src/draw.rs`, `crates/victory-shader/src/emit/lower.rs`.
- **Evidencia.** Diff de `921d8d3` nos dois arquivos, com o comentario *"A newer command list can publish more pages after this one
  has recorded. Only reference descriptors prepared for this particular bind."*
- **Estado atual.** Corrigido.
- **Confianca.** **CONFIRMADA** (o par host/tradutor so faz sentido junto: antes o `count` era inalcancavel pelo shader).

---

### D-06 - Textura com apenas mips intermediarios residentes era considerada ausente

- **Descricao / sintomas.** Texturas em streaming "esparso" (paginas comprometidas no meio, pontas nao) **nunca eram
  recuperadas**: o runtime concluia "nao ha nada aqui" e nao fazia staging. Efeito: material que nunca aparece, sem erro.
- **Causa raiz.** `mapped_bytes` fazia um curto-circuito: se a **primeira** e a **ultima** pagina nao estivessem comprometidas,
  retornava 0 sem olhar o meio. O comentario original ate justificava ("two queries for the slots whose texture is away"), mas a
  premissa "os mips menores ficam no inicio" nao vale para todos os layouts de streaming. Alem disso, a varredura por pagina
  contava `MEM_COMMIT` sem rejeitar `PAGE_NOACCESS` / `PAGE_GUARD`.
- **Introduzido por.** `570a941` (`git log -S"fn mapped_bytes"` -> apenas `570a941`). A versao anterior a importacao era outra
  (`4fc9d70` a endureceu; ver D-10 - o mesmo trecho tem historia propria em H-01).
- **Tentativas de correcao.** Nenhuma antes de `921d8d3`.
- **Correcao definitiva.** `921d8d3` - separa `mapped_bytes` (soma real) de `any_mapped` (booleano para a varredura), e extrai
  `mapped_regions(start, end, first_only)`, que **rejeita `PAGE_NOACCESS`/`PAGE_GUARD`** e para na primeira regiao quando o
  chamador so quer saber "existe algo". O curto-circuito sobrevive apenas para o caso degenerado de uma unica pagina.
- **Arquivos.** `crates/victory-gpu/src/texture.rs`.
- **Evidencia.**
  - Diff de `921d8d3`: `- if !committed(at, 1) && !committed((end - 1) as *const u8, 1) { return 0; }` e
    `+ if info.State == MEM_COMMIT && info.Protect & (PAGE_NOACCESS | PAGE_GUARD) == 0`.
  - Teste `a_texture_with_only_interior_pages_counts_its_resident_bytes`: reserva 5 blocos de 64 KiB, compromete **so o do meio**,
    e afirma `mapped_bytes(base, 5*BLOCK) == BLOCK` e `any_mapped(...) == true`.
- **Estado atual.** Corrigido.
- **Confianca.** **CONFIRMADA** (o teste e o caso exato).

---

### D-07 - Operacoes de quad em pixel shader rebaixadas por leitura de lane da wave inteira

- **Descricao / sintomas.** Amostragem/derivadas em pixel shader com operacoes de quad (`v_quad_read`, DPP com padrao de quad)
  produziam valores errados em pixels de borda. O commit chama isso de *"Repair pixel quad sampling"*.
- **Causa raiz.** O emissor so tratava operacoes de quad como nativeis para **mesh shader** (`let mesh_quad = self.s.mesh.is_some()
  && quad;`); em pixel shader caia em `cross_lane(quad)` + `read_lane`, isto e, em leitura de lane da wave inteira por indice -
  que nao expressa "o quad do pixel" (nem o comportamento das *helper lanes* nas bordas do triangulo). A deteccao de DPP8 tambem
  era grosseira: qualquer `Dpp::Row` com `ctrl <= 0xFF` era tratada como quad-local, sem verificar se cada destino realmente
  escolhe fonte no proprio quad.
- **Introduzido por.** `2849477` (`git log -S"mesh_quad"` -> `2849477` e `921d8d3`), ou seja, **anterior a importacao**.
- **Tentativas de correcao.** Nenhuma antes de `921d8d3`.
- **Correcao definitiva.** `921d8d3`:
  - `let use_quad = quad && (self.s.pixel || self.s.mesh.is_some());`
  - `quad_pick(x, src, pattern)` ganha atalhos por padrao: `0xe4` -> identidade; `0x00|0x55|0xaa|0xff` -> **um** `quadReadLaneAt`;
    `0xb1|0x4e|0x1b` -> `quadOp` com direcao XOR; demais casos mantem os quatro `quadReadLaneAt` + selects.
  - DPP8 passa a ser quad-local so quando `(0..8).all(|i| ((sel >> (3*i)) ^ i) & 4 == 0)`.
- **Arquivos.** `crates/victory-shader/src/emit/lower.rs`.
- **Evidencia.** Diff de `921d8d3` (duas ocorrencias de `mesh_quad` substituidas) e o comentario *"Read this quad, including pixel
  helpers across a triangle edge."*
- **Estado atual.** Corrigido.
- **Confianca.** **CONFIRMADA** para a mudanca de mecanismo e para o defeito de deteccao de DPP8 (a condicao nova e estritamente
  mais restritiva); **PROVAVEL** para a descricao do sintoma visual (o repositorio nao registra comparacao de imagem).

---

### D-08 - BVH: slots vazios entrando na ordenacao, grupo de 4 dwords cruzando a metade do no, e pagina ausente lida como geometria

- **Descricao / sintomas.** O construtor/traversal de raio produzia resultados inconsistentes: nos vazios do construtor
  apareciam como candidatos, nos podiam ser lidos do buffer errado e uma pagina de convidado sem copia era indistinguivel de
  geometria real na origem.
- **Causa raiz.** Tres defeitos no mesmo caminho:
  1. `bvh::boxes` aceitava um filho **vazio** (`children[n] == NONE`) cujos limites invertidos/infinitos do slot nao preenchido
     passavam no teste de slab, contaminando `distances` (a "miss distance" da ordenacao).
  2. `bvh_node_words` lia o no em grupos de 4 dwords decidindo o handle por `node_byte < 64`; com `offset = 16` e 24 dwords, um
     grupo podia **comecar em 60 e atravessar 64** - lendo 3 dwords do buffer da primeira metade (outra pagina de convidado, outro
     buffer host).
  3. Uma leitura de pagina sem copia devolve **zero**; zero e um ponteiro de filho e uma coordenada validos, entao o teste de slab
     sozinho nao distingue "ausente" de "geometria na origem".
- **Introduzido por.** `570a941` - `crates/victory-shader/src/bvh.rs` e `emit/intersect.rs` **nascem** nesse commit
  (`git log --diff-filter=A` lista apenas `570a941` para os dois). O defeito pode ser anterior na arvore privada, mas nao ha
  registro dele neste historico.
- **Tentativas de correcao.** Nenhuma antes de `bc3cd4b`.
- **Correcao definitiva.** `bc3cd4b`:
  - `bvh.rs`: `if children[n] != NONE && !b[0].is_nan() && ...`.
  - `intersect.rs`: novo `bvh_second_half(at)` calcula o `NodeAt` da segunda metade **de uma vez** e `bvh_node_words` passa a ler
    **12 dwords a partir de 16** e depois **12 dwords a partir de 0** da segunda metade - nunca um grupo que cruze a fronteira.
  - `NodeAt::Memory` ganha `mapped: V`; `residency` combina as duas metades (`[first, first, both, both]`) e o hit exige
    `child_present && pages[n]`.
- **Arquivos.** `crates/victory-shader/src/bvh.rs`, `crates/victory-shader/src/emit/intersect.rs`, `intersect_tests.rs`.
- **Evidencia.** `git show bc3cd4b -- crates/victory-shader/src/emit/intersect.rs`: `let mut words = self.bvh_node_words(at, None,
  16, 12); let second = self.bvh_second_half(at); words.extend(self.bvh_node_words(second, None, 0, 12));` e o comentario *"A zero
  returned by a missing page is a valid child pointer or coordinate, so the slab test alone cannot distinguish it from real
  geometry."*
- **Estado atual.** Corrigido. O teste de GPU correspondente (2, 3 e 3 unidades para dois triangulos espelhados, depois miss,
  e uma pagina deliberadamente nao mapeada gerando falta) esta descrito em `BENCHMARK.md`.
- **Confianca.** **CONFIRMADA**.

---

### D-09 - Toda janela de memoria mapeada era declarada "escrita": 12,8 GB comparados por frame

- **Descricao / sintomas.** Com o RT ligado, City levava **3.469 ms por frame** (0,29 FPS). Deste total, **2.036 ms/frame** eram
  espera por leituras de memoria do convidado, e **12.793 MB por frame** de memoria fora dos buffers registrados eram comparados
  (snapshot antes / comparacao depois de cada submissao).
- **Causa raiz.** O host tratava **toda** janela de memoria mapeada como escrita pela GPU, mesmo quando o shader de traversal
  apenas **lia** atraves do diretorio de paginas. O resultado era readback + comparacao de memoria que ninguem escreveu, com
  sincronizacao CPU/GPU por frame.
- **Introduzido por.** `570a941`. Em `crates/victory-gpu/src/memory.rs` a propria documentacao do modulo descreve o desenho e,
  sem saber, o defeito: *"Every dispatch of such a shader brings all numbered copies up to date and, if it can store, declares all
  of them written (the bytes are compared on the CPU afterwards, as for any unordered access view): **a dirty-page mask written by
  the shader would bound both**."* O codigo e `if writes && (first_write || n >= numbered) { shadow.gpu_writes(list, start, len); }`
  - `writes` e uma propriedade da **lista** ("pode armazenar"), nao da pagina.
- **Tentativas de correcao.** `940d32f` (pagina marcada nao salva a pagina para a propria copia) resolve um caso de borda do
  mesmo mecanismo, mas nao a causa.
- **Correcao definitiva.** `bc3cd4b` (*"track guest shader writes precisely"*): `writeback.rs` passa a ser o **dono dos bytes do
  convidado enquanto escritas de GPU estao em voo** (intervalos disjuntos + numero de submissao), e o tradutor distingue escrita em
  memoria de convidado de escrita em UAV separado - exatamente a "dirty-page mask" que o comentario de `memory.rs` pedia.
- **Arquivos.** `crates/victory-gpu/src/writeback.rs` (novo, 321 linhas), `inplace.rs`, `memory.rs`, `draw.rs`, `watch.rs`,
  `shadow.rs`.
- **Evidencia.** `BENCHMARK.md` @ HEAD, tabela *City measurement*:

  | Metrica | Antes | Depois |
  |---|---:|---:|
  | Frame medio | 3.469 ms | 969 ms |
  | FPS medio | 0,29 | 1,03 |
  | Mediana | 3.355 ms | 936 ms |
  | P95 | 4.457 ms | 1.214 ms |
  | Memoria de convidado comparada/frame | 12.793 MB | 2.655 MB |
  | Espera/frame por leituras | 2.036 ms | 95 ms |
  | Draws/frame | 12.583 | 12.520 |

  O numero de draws praticamente nao muda (12.583 -> 12.520): o ganho e **sincronizacao**, nao menos trabalho de render.
- **Estado atual.** Corrigido; City com RT segue em ~1 FPS (ver G-01).
- **Confianca.** **CONFIRMADA** (medicao do proprio repositorio, com antes/depois na mesma placa e mesmo save).

---

### D-10 - `Staging::detile` escreve alem do buffer mapeado (UB) - **corrigido e depois reintroduzido**

- **Descricao / sintomas.** Potencial corrupcao/UB silenciosa no upload de textura: `std::slice::from_raw_parts_mut` recebia um
  comprimento **maior** que o buffer de staging mapeado. Nao ha crash garantido - e exatamente o tipo de defeito que so aparece
  como "textura estranha" em producao.
- **Causa raiz.** `D3D12 GetCopyableFootprints` **nao preenche** o padding de row pitch na ultima linha do ultimo subrecurso.
  Logo `pitch * rows * depth` pode exceder `total - footprint.Offset`. O codigo usava essa multiplicacao direta como tamanho do
  slice.
- **Introduzido por.** `2849477` (`git log -S"let len = pitch * rows as usize * f.Depth as usize"` -> `2849477`, `4fc9d70`,
  `570a941`).
- **Tentativas de correcao.** `4fc9d70` (merge `711e930`, PR #1658): clampa com `subresource_len(offset, pitch, rows, depth, total)`,
  mantem `Staging.size` fora do `#[cfg(feature = "diagnostics")]`, endurece `committed()` (`checked_add`, rejeicao de
  `PAGE_NOACCESS`/`PAGE_GUARD`, guarda contra `next <= p`) e adiciona testes de regressao para os dois.
- **Correcao definitiva.** **NENHUMA no estado atual.** `570a941` restaurou a linha original: `git show HEAD:crates/victory-gpu/src/
  texture.rs` contem `let len = pitch * rows as usize * f.Depth as usize;` e **nao** contem `fn subresource_len`.
- **Arquivos.** `crates/victory-gpu/src/texture.rs`.
- **Evidencia.**
  ```
  $ git log -S"let len = pitch * rows as usize * f.Depth as usize" --format="%h|%ad|%s" --date=short
  570a941|2026-11-25|Publish DexNom Runtime Radeon sources with upstream credits and build instructions
  4fc9d70|2026-11-25|GPU: keep Staging::detile inside the staging buffer and harden committed()
  2849477|2026-11-23|Run Grand Theft Auto VI from first boot into the world
  ```
  A presenca de `570a941` no topo da lista de `-S` (mesma string adicionada e removida) e a prova de que a linha **voltou**.
- **Estado atual.** **ABERTO** (reintroduzido). Ver H-01.
- **Confianca.** **CONFIRMADA**.

---

### D-11 - Corrida na escrita atomica da pipeline library (`.tmp` de nome fixo)

- **Descricao / sintomas.** Saves concorrentes da pipeline library (salvamento periodico do worker + save de shutdown) podiam
  escrever no **mesmo** arquivo temporario e renomear duas vezes; resultados possiveis: save perdido, arquivo truncado, ou rename
  falhando.
- **Causa raiz.** `write_atomically` usava `path.with_extension("tmp")` - nome **fixo** - e nenhuma serializacao:
  ```rust
  fn write_atomically(path, bytes) { let tmp = path.with_extension("tmp"); std::fs::write(&tmp, bytes)?; std::fs::rename(&tmp, path) }
  ```
  Dois escritores: um trunca o `.tmp` enquanto o outro renomeia.
- **Introduzido por.** `35b054e` *"GPU: pipelines off the render thread and cached"* (`git log -S"write_atomically"` -> `35b054e`,
  `9c58a1c`, `be824d9`, `570a941`). Antes de `35b054e` nao havia salvamento fora da thread de render, portanto nao havia corrida.
- **Tentativas de correcao.** `be824d9` (merge `a9d233f`, PR #1660): mutex `WRITING`, contador `NEXT`, nome
  `host-pipelines.bin.tmp-<pid>-<n>`, remocao do temporario em caso de erro, e teste com 4 threads x 30 escritas.
- **Correcao definitiva.** **NENHUMA no estado atual.** `git show HEAD:crates/victory-gpu/src/pipeline/library.rs` mostra
  `let tmp = path.with_extension("tmp"); std::fs::write(&tmp, bytes)?; std::fs::rename(&tmp, path)` - **byte a byte** o codigo
  pre-`be824d9`.
- **Arquivos.** `crates/victory-gpu/src/pipeline/library.rs`.
- **Evidencia.** Diff de `be824d9` (as tres linhas removidas) contra `HEAD` (as mesmas tres linhas presentes). `git grep "tmp-"` em
  `HEAD` -> ausente.
- **Estado atual.** **ABERTO** (reintroduzido). Ver H-02.
- **Confianca.** **CONFIRMADA**.

---

### D-12 - `clwb_loop`: overflow de u64 no calculo do `base` final

- **Descricao / sintomas.** Emulacao de laco de `clwb` em uma unica excecao. Se `limit - base` estiver proximo de `u64::MAX`, o
  passo calculado transborda e o `base` resultante fica errado; o laco do titulo continua a partir de um endereco arbitrario.
- **Causa raiz.**
  ```rust
  *base = if *base <= limit { base.wrapping_add(((limit - *base) / 64 + 1) * 64) } else { base.wrapping_add(64) };
  ```
  `(limit - *base) / 64 + 1) * 64` e aritmetica `u64` sem `checked_*`: em release, `wrapping`. `wrapping_add` no `base` apenas
  esconde o estrago.
- **Introduzido por.** `c4e915f` *"loader: a clwb write-back loop runs in one exception"* (2026-11-23) - `git log -S'((limit - *base)
  / 64 + 1) * 64'`.
- **Tentativas de correcao.** `274f698` (merge `17de445`, PR #1539): extrai `clwb_loop_end(base, limit) -> Option<u64>`, que
  devolve `None` no overflow/wraparound e cai para a emulacao instrucao-a-instrucao; mais testes de borda.
- **Correcao definitiva.** **NENHUMA no estado atual.** `git show HEAD:crates/victory-loader/src/crash.rs` contem exatamente a
  expressao original e **nao** contem `clwb_loop_end`.
- **Arquivos.** `crates/victory-loader/src/crash.rs`.
- **Evidencia.** Bloco `if let Some(pass) = clwb_loop(&bytes[..n])` em `HEAD` com o `if *base <= limit` original; ausencia de
  `clwb_loop_end` (`git show HEAD:...crash.rs | Select-String clwb_loop_end` -> vazio).
- **Estado atual.** **ABERTO** (reintroduzido). Ver H-03.
- **Confianca.** **CONFIRMADA** (mesma expressao, ausencia da funcao de guarda).

---

### D-13 - Testes novos de material demand: recurso posicionado fora da reserva e corrida com o relogio global de frames

- **Descricao / sintomas.** Dois testes introduzidos no ultimo export **falhavam de forma intermitente** e, no caso do OOB,
  tocavam memoria fora da alocacao reservada (o "teste" era o bug).
- **Causa raiz.** Duas causas independentes:
  1. O teste reservava `2 << 20` bytes em `base`, mas posicionava o "writer" em `addr - small` (antes do inicio da alocacao) e
     nao garantia que o segundo recurso coubesse em `[base, base + writer_bytes)`.
  2. O teste lia o **frame global** (`crate::frame::current()`) e o usava como carimbo de idade; outras threads/ testes avancam o
     frame, entao a idade calculada mudava durante o teste.
- **Introduzido por.** `b8ff22e` (o teste e novo; `git log -S"material_demand_revives"` -> somente `b8ff22e`).
- **Tentativas de correcao.** `4775e8a` cria `frame::TEST_FRAME_LOCK` e o toma nos testes que dependem do frame (parcial: so o
  teste de readback de material). `2e01ab4` completa: reserva o footprint **inteiro** do writer, posiciona cada view dentro dele,
  afirma `base <= at && at + bytes <= base + writer_bytes`, e toma o mesmo lock.
- **Correcao definitiva.** `2e01ab4` ("Keep cached test textures within their reserved guest memory", ultimo commit).
- **Arquivos.** `crates/victory-gpu/src/descriptor.rs`, `crates/victory-gpu/src/frame.rs`.
- **Evidencia.**
  - `git show 2e01ab4`: `- let _writer = crate::resource::texture_at(addr - small, &writer).unwrap();` ->
    `+ let _writer = crate::resource::texture_at(base, &writer).unwrap();` e `assert!(base <= at && at + bytes <= base + writer_bytes);`
  - `git show 4775e8a`: `pub(crate) static TEST_FRAME_LOCK: Mutex<()> = Mutex::new(());`
- **Estado atual.** Corrigido em duas etapas; `HEAD` e test-only por causa disso.
- **Confianca.** **CONFIRMADA**.

---

### D-14 - Tabelas BCPack fora do repositorio: `build.rs` removido e verificacao de hash dependente do host

- **Descricao / sintomas.** O build limpo falha (ou usa tabela errada) se as tabelas BCPack nao existirem; a verificacao de
  integridade do script de setup dependia de um cmdlet que nem toda instalacao tem.
- **Causa raiz.** Politica "source-only": `570a941` **remove** `crates/victory-formats/src/bcpack-tables.bin` (77.982 bytes) e
  `crates/victory-shader/build.rs`, e passa a baixar/gerar as tabelas em `Initialize-FormatTables.ps1` (SHA-256 esperado
  `1F1D4760...C36`, fonte `raw.githubusercontent.com/Marvisak/VIctory/2849477.../`). A verificacao usava
  `(Get-FileHash -LiteralPath $Path -Algorithm SHA256).Hash`.
- **Introduzido por.** `570a941` (remocao) e `ef8f4f1` (script de setup).
- **Tentativas de correcao.** `8c7937a` troca `Get-FileHash` por `[Security.Cryptography.SHA256]::Create()` +
  `[BitConverter]::ToString(...).Replace('-','')`, com `Dispose` em `finally`.
- **Correcao definitiva.** `8c7937a` para a verificacao. A ausencia do binario e **decisao de projeto**, nao bug - mas e a causa
  de uma dependencia externa de rede no primeiro build (ver G-06).
- **Arquivos.** `Initialize-FormatTables.ps1`, `Setup.cmd`, `Setup-DexNomRuntime.ps1`, `crates/victory-formats/src/bcpack.rs`.
- **Evidencia.** Diff de `8c7937a`; `git show 570a941 --name-status` com `D crates/victory-formats/src/bcpack-tables.bin` e
  `D crates/victory-shader/build.rs`; `ci.yml` @ HEAD chama `./Initialize-FormatTables.ps1` como primeiro passo.
- **Estado atual.** Resolvido como projetado; build limpo exige rede uma vez.
- **Confianca.** **CONFIRMADA** para os fatos; **PROVAVEL** para a motivacao exata da troca de `Get-FileHash` (nao ha mensagem
  explicando; o commit e de uma linha de assunto).

---

### D-15 - Deteccao de "hardware" nos testes de shader: WARP passava por GPU

- **Descricao / sintomas.** CI vermelho de forma persistente em dois testes hardware-only:
  `gathers_at_a_level_match_the_hardware` (comparava arredondamento do hardware contra WARP) e
  `permutes_read_the_lane_named_and_zero_from_lanes_left_out` (precisa de wave de 32, que WARP recusa - rodava em device nenhum).
- **Causa raiz.** `Host::new(false)` usava `D3D12CreateDevice(None, ...)`, isto e, o **adaptador padrao**, que no runner e WARP. O
  teste chamava isso de "hardware".
- **Introduzido por.** O proprio desenho dos testes `synthetic_tests` (presentes ate `940d32f`).
- **Tentativas de correcao.**
  1. `42363ab`: recusa adaptador padrao sinalizado `DXGI_ADAPTER_FLAG_SOFTWARE`, e `permutes...` passa a exigir `assert!(warp, ...)`
     quando nao ha pipeline.
  2. **Insuficiente** - `ccfb029` (Ivan) documenta o porque: *"42363ab refuses a default adapter flagged DXGI_ADAPTER_FLAG_SOFTWARE,
     but on the runner the default adapter is not flagged software and its device still refuses a wave of 32, so `Host::new(false)`
     succeeded and the 'hardware' runs were not hardware."*
- **Correcao definitiva.** `ccfb029`: `hardware_adapter(factory)` percorre os adaptadores e aceita o primeiro que seja GPU por
  **dois sinais** - nao sinalizado software **e** com `DedicatedVideoMemory > 0` (WARP e o basic render driver reportam zero);
  sem nenhum, `Err("no hardware adapter")`, que os testes ja sabem tratar como skip.
- **Arquivos.** `crates/victory-shader/src/emit/host.rs`, `crates/victory-shader/src/emit/synthetic_tests.rs`.
- **Evidencia.** Mensagem e diff de `ccfb029` (a citacao acima e do proprio commit); `42363ab` mostra a tentativa anterior.
- **Estado atual.** **O arquivo alvo foi apagado 3 commits depois**: `570a941` remove `crates/victory-shader/src/emit/synthetic_tests.rs`
  (`git cat-file -e 42363ab:...synthetic_tests.rs` -> existe; `git show 570a941 --name-status` -> `D`). Ou seja: a correcao
  `ccfb029` foi **tornada obsoleta por remocao**, nao por substituicao equivalente.
- **Confianca.** **CONFIRMADA** (inclusive a obsolescencia). O comportamento equivalente de skip hoje esta em `emit/mod.rs` e nos
  `*_tests.rs` sobreviventes, controlado por `VICTORY_TESTS_ALLOW_SOFTWARE_SKIP=1` no `ci.yml`.

---

### D-16 - Falsa exaustao de descritores dinamicos quando o run livre cruza o cursor

- **Descricao / sintomas.** Alocacao de N descritores dinamicos falhava mesmo havendo N contiguos livres, quando o run livre
  atravessava o cursor de busca. Sintoma de superficie: draw descartado / material sem recurso.
- **Causa raiz.** A busca era `(*next..dynamic).chain(0..*next)`: ela comeca em `next` e depois **recomeca em 0**, entao um run
  contiguo que **cruza** `next` (ex.: 10..14 com cursor 12) nunca era encontrado - os candidatos 10 e 11 ficavam na parte "0..12",
  mas a varredura ja tinha sido reiniciada e o encadeamento exigia contiguidade a partir do indice visitado.
- **Introduzido por.** O alocador dinamico do host heap (anterior a esta arvore; visivel em `crates/victory-gpu/src/descriptor/host.rs`
  antes de `c1c3281`).
- **Tentativas de correcao.** `c1c3281` (*"extend dynamic descriptor wraparound scan to prevent false exhaustion"*): segunda volta
  limitada a `cursor + n - 1` (`second_end`), com teste que reproduz exatamente o caso (cursor 12, run livre 10..14).
- **Correcao definitiva.** `c1c3281`, integrada por `c6c244d` (*"Merge pull request #974 ... (fixes #871)"*) e exercitada no host real
  por `072fde0` (*"Exercise a free run straddling the cursor on the actual host heap"*).
- **Arquivos.** `crates/victory-gpu/src/descriptor/host.rs`.
- **Evidencia.** Diff de `c1c3281` com o comentario *"extending past cursor by n - 1 so contiguous runs crossing the cursor are not
  missed"*; `HEAD` ainda contem `second_end` (verificado por `git show HEAD:...descriptor/host.rs | Select-String second_end`).
- **Estado atual.** **Corrigido e sobrevivente** (um dos poucos fixes da leva de merges que a importacao nao desfez).
- **Confianca.** **CONFIRMADA**.

## E. Relacao de causa e efeito

### E-1 - A cadeia central do fork: importacao -> regressao -> reparo incremental

```
2849477 (o jogo roda; baseline publico)
   -> 35b054e (pipelines fora da thread de render + cache em disco)   -> introduz D-11
   -> 4fc9d70 (clampa Staging::detile)                                  -> corrige D-10
   -> 274f698 (guarda o overflow do clwb_loop)                          -> corrige D-12
   -> e6fe8ec (normaliza LOD do sampler sem mip)                         -> corrige parte de G-3
...
570a941 (importa a linha privada Radeon sobre a arvore publica)
   -> sobrescreve os 98 arquivos que a leva de merges havia tocado
   -> REVERTE 4fc9d70 (D-10), be824d9 (D-11), 274f698 (D-12)
   -> APAGA synthetic_tests.rs (obsolescencia de ccfb029, D-15)
   -> introduz D-01, D-04, D-06, D-08, D-09; D-02, D-03 e D-07 vem da base e ficam sem correcao ate ddb7332/76749ce/921d8d3
   -> 5f4c3df amplia a page table e o streaming, estendendo D-05
ddb7332 -> corrige D-01, D-02
76749ce -> corrige D-03
bc3cd4b -> corrige D-08, D-09
921d8d3 -> corrige D-04, D-05, D-06, D-07
b8ff22e -> introduz D-13
4775e8a + 2e01ab4 -> corrigem D-13
```

Consequencia: **o fork e mais novo que a base em funcionalidade e mais velho que a base em correcoes**. Nao ha, em nenhum ponto,
um merge que reaplique os fixes da leva; os reparos posteriores sao reescritas proprias, e tres fixes conhecidos nao voltaram.

### E-2 - Cadeia do RT: da falha catastrofica a ~1 FPS

```
0b35b7f (interseccao de BVH no tradutor da base)
   -> 570a941 (BVH do fork: bvh.rs + emit/intersect.rs; page table com USER_MEMORY_BASE)
        -> memory.rs declara TODAS as copias numeradas como escritas a cada dispatch que PODE armazenar
           (o proprio comentario pede "a dirty-page mask written by the shader")   <- D-09
   -> 5f4c3df (page table ampliada, streaming, shadow pipelines)
        -> builder parcial escreve limites invalidos na memoria do titulo -> titulo crasha
        -> por isso nasce a regra de seguranca: builder_runs() exige host pipeline para TODOS
           os pipelines do builder, senao nenhum roda (+ VICTORY_BVH_QUARANTINE)
   -> D-09: toda janela mapeada declarada escrita -> 2.036 ms/frame de espera
   -> bc3cd4b: writeback.rs (dono dos bytes) + distincao escrita-de-convidado x UAV + BVH (D-08)
        -> 3.469 ms -> 969 ms
   -> 921d8d3: count de pagina (D-05) + mip esparso (D-06) + quad (D-07) + LOD (D-04)
   -> BENCHMARK: 6.074.512 visitas de no, 848 triangulos validos ate o frame 1.204 (um shader)
```

A mensagem de `pipeline.rs` (`builder_off`) e a doc (`docs/fork-ray-tracing.md`, item 5) registram a causa do crash - isso e
**documentacao de causa raiz no proprio codigo**, raro e valioso.

### E-3 - Cadeia de traducao de shader: cada "recusa" e um tipo de shader morto

```
25.758 shaders no arquivo
   -> 56 recusas (D-02: sampler por phi)                     -> ddb7332 recupera 40 variantes
   -> +1 recusa (D-03: wave64 restaurado por s_mov_b32)      -> 76749ce recupera 1
   -> 25.742 traduzidos; 15 recusas permanecem              -> G-02
```

O mesmo padrao se repete em escala menor: `13091f6` (2.191 vertex shaders, 39 falhando no validador DXIL -> 2.191 passando) e
"5 shadow compute shaders" (BENCHMARK), sempre com a mesma forma: **o tradutor aceita a forma que ja viu e recusa a forma nova**.

### E-4 - Cadeia de readback (compartilhada com a base)

```
3a(?): snapshot de todos os ranges declarados escritos
   -> cd94d35: shaders marcam paginas de 64 KiB que escrevem por view; copy-back compara so essas
        (2,8-4,5 GB/frame, 98% inalterado, 0,45-0,7 s de um frame de 1,5-1,7 s)
   -> 940d32f: a pagina marcada nao salva a pagina para a propria copia
   -> fcdefcf / ed0dbfd: copia de escrita do titulo sob ranges que os shaders da submissao escrevem
```

Esta cadeia e o melhor exemplo de otimizacao baseada em medicao no historico: o numero (98% inalterado) e a justificativa do
desenho (bitmap por pagina marcada pelo tradutor).

## F. Correcoes importantes

### F-1 - Bypass de blending em alvo inteiro (`ddb7332`)

**O que estava errado.** Estado de blend do console repassado ao D3D12 sem considerar formatos de alvo inteiros; o PSO era
recusado na criacao. **Por que a correcao funciona.** O console, para superficie inteira, ignora o blender; o D3D12 **valida** a
combinacao na criacao. Desligar `BlendEnable` apenas nos alvos inteiros reproduz o console e satisfaz o validador. O detalhe que
faz a diferenca e a **expansao do estado compartilhado** antes de sobrescrever um alvo: sem ela, `IndependentBlendEnable = false`
faria o D3D12 ler o slot 0 para todos, e o write mask dos outros alvos seria perdido. **Dependencias.** Nenhuma externa; so o
estado de pipeline. **Reutilizavel?** Sim, integralmente, em qualquer camada que traduza estado de blend de console para D3D12.
**Risco de adaptacao.** A lista de formatos inteiros precisa ser completa para o seu conjunto de formatos (aqui: 20 formatos
`UINT`/`SINT`, sem os `_UNORM`/`_FLOAT`).

### F-2 - Sampler por phi com duas amostragens e selecao no edge (`ddb7332`)

**O que estava errado.** Um sampler vindo de um phi de duas constantes era "nao rastreado" e o shader era recusado. **Por que a
correcao funciona.** Amostrar duas vezes com as **mesmas derivadas** e escolher o resultado resolve o caso porque a escolha do
sampler e uniforme (mesmo edge para toda a wave); o custo e uma amostra extra, nao uma reestruturacao de CFG. O caso de um
predecessor nao finalizado e tratado separadamente (usa o sampler daquele lado), o que evita um phi com aresta indefinida.
**Dependencias.** Exige que `Val::I32` de todas as saidas seja reconstruivel (`i32_of`), o que limita a forma das instrucoes
tratadas. **Reutilizavel?** Sim: o padrao "duplicar a operacao e selecionar no join" serve para qualquer operando uniforme vindo
de phi. **Risco.** Dobra o custo da amostragem nesses shaders; para shaders de particula/UI pode ser irrelevante, para G-buffer
nao.

### F-3 - Escrita de convidado rastreada (`bc3cd4b`, `writeback.rs`)

**O que estava errado.** Declarar toda janela mapeada como escrita forcava snapshot + comparacao de 12,8 GB por frame e 2 s de
espera. **Por que a correcao funciona.** Duas coisas separadas:
(1) *Quem escreveu* - o tradutor sabe, por construcao, se um store vai para memoria de convidado ou para UAV do host; a
informacao e propagada para o host.
(2) *Quem e o dono dos bytes* - `writeback.rs` mantem intervalos disjuntos com o numero da submissao mais recente ja iniciada,
porque **uma fence pode completar depois de o titulo desmapear e reusar o endereco**. Sem isso, o copy-back escreveria em memoria
que ja pertence a outro objeto. **Dependencias.** `STARTED`/`PUBLISHED`/`LAST_CHANGE` com `Release`/`Acquire`; a publicacao tem de
terminar antes da libertacao do intervalo. **Reutilizavel?** Sim, e um pattern de **writeback com posse temporal** aplicavel a
qualquer camada de memoria unificada emulada sobre memoria discreta. **Risco.** Se a ordenacao fence -> merge mudar, a garantia
some silenciosamente; o modulo tem 321 linhas e merece teste de concorrencia proprio (nao ha teste de corrida nele).

### F-4 - Bitmap de paginas marcadas pelo tradutor (`cd94d35`)

**O que estava errado.** Comparar 2,8-4,5 GB por frame para descobrir que 98% nao mudou. **Por que a correcao funciona.** O
tradutor emite, para cada store/atomico **atraves de view de buffer nao ordenada**, um bit por pagina de 64 KiB numa bitmap; o
host compara so as paginas marcadas. Uma tabela de marcas por slot vive no host view heap (`marks`, dois dwords por slot,
`0xFFFF...` = sem marca). A pagina que o titulo escreve e que os shaders tambem podem escrever guarda os bytes de antes (salvos
pelo guard na primeira escrita), preservando **os dois escritores**. **Dependencias.** Requer `emit::Options::view_marks` e que o
cache de shader seja invalidado quando a opcao muda (o cache e chaveado pelo tradutor, `c47b529`). **Reutilizavel?** Sim - e a
ideia mais transferivel do repositorio: *nao compare o que ninguem escreveu; faca o tradutor dizer o que escreveu*.

### F-5 - `OpenExistingHeapFromAddress` para memoria do titulo (`inplace.rs`)

**O que estava errado.** Toda leitura/escrita de memoria do titulo passava por uma copia host, com sincronizacao. **Por que a
correcao funciona (quando funciona).** `inplace.rs` documenta, com medicao, as regras reais do Direct3D neste PC: o
`OpenExistingHeapFromAddress` aceita **so a base de uma alocacao `VirtualAlloc` inteira**, com todas as paginas comprometidas e
uma unica protecao; abrir trava todas as paginas na memoria do PC (256 MiB nunca tocados = 70 ms e 256 MB de working set). A
variante 1 (`ID3D12Device13`) aceita qualquer parte comprometida, inclusive views sobrepostas; e
`OpenExistingHeapFromFileMapping` aceita `SEC_COMMIT` e recusa `SEC_RESERVE` (que e como `xmem` faz o pool do titulo hoje).
**Reutilizavel?** Sim - este arquivo e um **estudo de caso documentado** de zero-copy de memoria de convidado em D3D12.
**Risco.** A elegibilidade depende do driver e do arranjo de `xmem`; o proprio modulo traz um probe
(`VICTORY_INPLACE_PROBE`) e um censo - ou seja, o autor nao presumiu, mediu.

### F-6 - Regra de seguranca do builder de BVH (`5f4c3df`, `pipeline.rs`)

**O que estava errado.** Um builder parcialmente funcional escrevia limites invalidos na memoria do titulo e **crashava o jogo**.
**Por que a correcao funciona.** `builder_runs()` so devolve `true` se **todo** pipeline do builder que o titulo criou tem
pipeline de host; qualquer queda (`builder_off`) desliga **todos** os estagios de uma vez. Assim o conjunto nunca executa meio
certo. `VICTORY_BVH_QUARANTINE=1` mantem os stores do builder fora da memoria do titulo para um primeiro teste.
**Reutilizavel?** Sim, como padrao: **atomicidade de conjunto** para subsistemas cujo funcionamento parcial e pior que a ausencia.

### F-7 - `subresource_len` para staging (D-10) - a correcao que nao sobreviveu

Vale registrar por si: a correcao estava certa (clampar o slice ao `total - offset`), tinha teste (`subresource_len(0,256,4,1,784)
== 784`), e mesmo assim **nao esta no HEAD**. E o exemplo mais claro de por que "existe um fix no historico" nao equivale a
"existe um fix no produto".

## G. Problemas ainda existentes

### G-1 - Custo de frame do RT: ~1 FPS com o builder ligado
`BENCHMARK.md`: 969 ms de frame medio, **2.655 MB/frame** ainda comparados e **95 ms/frame** de espera. A propria doc aponta
*"Buffer snapshot, comparison and CPU/GPU synchronization account for a substantial part of the measured frame time."*
Classificacao: **CONFIRMADA** (medicao no repositorio). Nao ha, no historico, commit posterior a `bc3cd4b` que ataque esse custo
(o repositorio congela em `2e01ab4`, test-only).

### G-2 - 15 shaders do arquivo continuam recusados
`BENCHMARK.md`: *"Retesting all 56 original refusals produced 41 validated shaders and 15 remaining refusals."* Nao ha lista das
15 no repositorio, nem commit posterior. Classificacao: **CONFIRMADA** a existencia; a identidade das 15 e **NAO VERIFICADA**.

### G-3 - Sampler sem mip filter: LOD derivado de `min_lod`, sem normalizacao de `NaN`/ordem
`HEAD` (`crates/victory-gpu/src/sampler.rs`) usa `MinLOD: self.min_lod` e `MaxLOD: if self.mip == 0 { self.min_lod } else {
self.max_lod }`, **sem** o `finite_lod`/ordenacao que `e6fe8ec` (11-24) introduziu (`git log -S"finite_lod"` lista `e6fe8ec` e
`570a941`; `finite_lod` **nao** existe em `HEAD`). Com `min_lod != 0` o sampler de "sem filtro de mip" **nao** amostra o mip 0,
que e o comportamento documentado no proprio comentario do codigo. Classificacao: **PROVAVEL** (a ausencia da funcao e
CONFIRMADA; o efeito exige um titulo que passe `min_lod` diferente de zero sem mip filter).

### G-4 - Diretorio de paginas do convidado satura em 1.023 slots
`BENCHMARK.md`: *"City can still exhaust the page directory's 1,023 backing-copy slots. A missing mapping can return zero or lose
a write."* `docs/fork-ray-tracing.md`: *"An access to a title page that has no host copy is lost (a read gives zero, a store is
dropped)."* Isso e uma **perda de dados silenciosa** por desenho, sob pressao. Classificacao: **CONFIRMADA**.

### G-5 - Iluminacao interna da City e dois objetos escuros sem causa
`BENCHMARK.md`: *"The City interior can render with an overexposed doorway and a silhouetted character"*; *"the tabletop item and
upright weight plate were already dark in both native TAA outputs ... the cause of these two objects' color is unresolved"*.
Classificacao: **CONFIRMADA** como limitacao conhecida; causa **NAO VERIFICADA**.

### G-6 - Build limpo depende de rede e de fonte externa
`Initialize-FormatTables.ps1` baixa as tabelas BCPack de `raw.githubusercontent.com/Marvisak/VIctory/2849477.../` e verifica um
SHA-256 fixo. Sem rede, o primeiro build nao completa (o `ci.yml` chama esse passo antes de instalar o toolchain). Classificacao:
**CONFIRMADA** (o script e o CI provam a dependencia).

### G-7 - Importacao reverteu fixes: tres abertos, e um varrimento nao feito
D-10, D-11, D-12 estao **abertos** por reversao. Nao encontrei nenhum commit posterior que os reaplique, e nao ha, no repositorio,
registro de uma auditoria "o que da leva de merges sobreviveu a importacao". O padrao medido (0 de 98 arquivos identicos) sugere
que os outros 95 arquivos tambem podem ter perdido correcoes - **nao verifiquei um a um** (ver L-3).

### G-8 - Cobertura de teste de shader reduzida por remocao
`570a941` remove `emit/bvh_tests.rs`, `flow_tests.rs`, `memory_tests.rs`, `semantics_tests.rs`, `stride_tests.rs`,
`synthetic_tests.rs`, `benches/guard_runs.rs`, `tests/census.rs`, `hle/tests/fidelityfx.rs`, `loader/tests/crash_log.rs`. O
equivalente existe em parte (`valu_tests.rs`, `diff_tests.rs`, `waterfall_tests.rs`, `intersect_tests.rs`, `cube_lod_tests.rs`,
`helper_lane_tests.rs`, `wave64_tests.rs`), mas nao ha mapeamento 1:1. Classificacao: **CONFIRMADA** a remocao; a perda efetiva de
cobertura e **NAO VERIFICADA** (nao rodei a suite).

### G-9 - `committed()` mudou de mecanismo e manteve uma soma nao verificada
`HEAD` resolve `committed` com `K32QueryWorkingSetEx` + `read_into` por pagina (`fn readable`), em vez do `VirtualQuery` de
`4fc9d70`. O mecanismo novo **testa legibilidade de verdade**, o que cobre `PAGE_NOACCESS`/`PAGE_GUARD`. Duas fragilidades restam,
e ambas estavam no escopo declarado de `4fc9d70`: (a) `fn readable` abre com `at.saturating_add(len - 1)` - no caminho padrao
`committed` retorna `true` antes, para `len == 0`, mas a garantia mora no chamador, nao na funcao; (b) no caminho
`VICTORY_COMMITTED_QUERY=1`, `committed_by_query` faz `let end = p + len;` **sem `checked_add`** - exatamente o *"unchecked p + len
(wrapping to end < p in release)"* que `4fc9d70` corrigiu. Classificacao: **CONFIRMADA** para o codigo; **HIPOTESE** para
explorabilidade (exige `len` proximo de `usize::MAX`, que nenhum chamador atual produz).

### G-10 - Nenhum TODO no codigo, 44 "shortcut:" no lugar
`git grep -E "TODO|FIXME|HACK|XXX"` em `HEAD` devolve **uma** ocorrencia (uma referencia ao `TODO.md` em
`hle/src/modules/display.rs`). As limitacoes estao marcadas com `shortcut:` em **44 arquivos** e em prosa nos docs. E uma escolha
disciplinada, mas significa que **levantar o estado real exige ler comentarios**, nao procurar marcadores.

## H. Regressoes

### H-01 - Regressao mestre: `570a941` sobrescreve a arvore publica com a linha privada

**Fato medido.**

```
$ git diff --name-only 70564ae 4118f33 | wc -l        # arquivos tocados pela leva de 287 commits
98
$ git show 570a941 --name-only --format="" | wc -l    # arquivos tocados pela importacao
170
$ (interseccao)                                        # dos 98, quantos a importacao tocou
98
$ (dos 98, quantos estao byte-identicos em HEAD)       # rev-parse 4118f33:path vs HEAD:path
0
$ (dos 84 nao tocados pela leva, quantos estao identicos em HEAD)
37 identicos / 47 diferentes
```

Ou seja: **todos os 98 arquivos que a leva de merges havia modificado foram reescritos pela importacao, e nenhum deles ficou no
estado pos-merge.** Em contraste, 37 arquivos fora da leva permaneceram intactos. Isso e consistente com a mensagem posterior de
`b8ff22e` (*"sync ... onto public source baseline `921d8d3`"*) e com o `Initialize-FormatTables.ps1`, que aponta para `2849477`:
a linha privada e descendente de um estado **muito anterior** a leva de merges.

**Efeito.** Nao e "perda de commits" (o historico preserva tudo); e **perda de efeito**: o codigo que valia em `4118f33` deixou de
valer em `570a941`.

**Instancias verificadas de perda** (nao inferidas por ausencia de nome - conferi o codigo):

| Fix | Commit | Verificacao em `HEAD` |
|---|---|---|
| D-10 `subresource_len` | `4fc9d70` / `711e930` | a linha `let len = pitch * rows as usize * f.Depth as usize;` esta de volta; `fn subresource_len` ausente |
| D-11 escrita atomica | `be824d9` / `a9d233f` | `write_atomically` tem `path.with_extension("tmp")` e nenhum mutex/contador |
| D-12 overflow do `clwb_loop` | `274f698` / `17de445` | expressao original presente; `clwb_loop_end` ausente |
| D-15 skip de hardware | `ccfb029` | `synthetic_tests.rs` **apagado** por `570a941` |
| Sampler sem mip (G-3) | `e6fe8ec` | `finite_lod` ausente; `MaxLOD` derivado de `min_lod` |

**Nao sao regressoes** (verifiquei o mecanismo, nao o nome): `4ab20fc` (orcamento indireto) - `HEAD` tem um laco CAS que satura em
`BUDGET` e nao envolve; `8a772ea`/`810f442` (tick do frame wait) - `HEAD` tem versao **mais completa** (logica de "late waits");
`60d2cf9` (`clwb` no fim da pagina) e `c1c3281` (cursor de descritores) - presentes.

**Classificacao.** **CONFIRMADA**.

### H-02 - `b8ff22e` reintroduz a dependencia de frame global nos testes

`4775e8a` criou `TEST_FRAME_LOCK` **e** `b8ff22e` (commit imediatamente anterior, do mesmo autor Codex, no mesmo arquivo) trouxe
um teste que nao o tomava. Nao e regressao de produto, e regressao de teste: o ultimo commit do repositorio existe porque o
penultimo exportou um teste que nao podia passar de forma deterministica. **CONFIRMADA**.

### H-03 - `570a941` apaga o `build.rs` e o binario de tabelas, mudando o contrato de build

Antes: `cargo build` bastava (build.rs gerava as tabelas). Depois: e obrigatorio rodar um script PowerShell que baixa um arquivo
de um hash especifico do upstream. Isso e uma **regressao de ergonomia** deliberada (politica source-only) com custo: build
offline quebra, e o script vira superficie de ataque da cadeia de suprimentos (mitigada pelo SHA-256 fixo + `[BitConverter]`,
`8c7937a`). **CONFIRMADA**.

### H-04 - Reducao de cobertura por remocao de testes

Ver G-8. **CONFIRMADA** a remocao; impacto nao medido.

## I. Conhecimento reutilizavel

> Foco: o que e **exclusivo deste fork** (ausente na base e nos outros forks segundo `_fatos/FORK_DIVERGENCIA.md`) e vale levar
> para um repositorio unificado. Para cada item: problema, implementacao, arquivos, dependencias, conflitos, risco, recomendacao.

### I-1 - Marca de escrita emitida pelo tradutor (`view_marks`) + bitmap por pagina no host
- **Problema resolvido.** Copy-back/sincronizacao de GPU baseada em "ranges declarados como escritos" custa GB/frame. Aqui:
  2,8-4,5 GB/frame, 98% inalterado, 0,45-0,7 s de um frame de 1,5-1,7 s (`cd94d35`).
- **Implementacao.** `emit::Options { view_marks: true }` faz o emissor marcar, por store/atomico **atraves de view de buffer nao
  ordenada**, um bit por pagina de 64 KiB numa bitmap; o host mantem uma **tabela de marcas por slot** no view heap (dois dwords
  por slot; `VIEW_MARK_NONE` = sem marca, inicializado `0xFF`); cada copia host rastreada e dona de um run de bits; apos a
  submissao, so as paginas marcadas sao comparadas e escritas de volta. Pagina escrita pelos dois lados guarda os bytes de antes.
- **Arquivos.** `crates/victory-shader/src/emit/{mod.rs,lower.rs,shader.rs}`, `crates/victory-gpu/src/{shadow.rs,watch.rs,draw.rs,
  descriptor/host.rs,copystat.rs,shader_cache.rs}`.
- **Dependencias.** `emit::Options` tem de entrar na chave do cache de shader (senao um shader traduzido sem marcas e reusado);
  contadores de diagnostico (`C::MarkedPages`, `MarkedBytes`, `MarkedUs`, `MarkedKept`, `SavedPages`).
- **Conflitos possiveis.** Com `bindless`/`external` da base (que aqui nao existem) a enumeracao de views muda; com qualquer
  caminho que escreva em buffer **sem** view (descritor direto) a marca nao sai - o codigo cai no snapshot, o que e seguro mas
  silencioso.
- **Risco.** Um `store` que o tradutor nao reconheca como escrita gera **perda de bytes** (nao corrupcao), porque a pagina nao e
  comparada. Mitigacao correta: manter o caminho de snapshot para copias nao rastreadas (o codigo faz isso).
- **Recomendacao.** **Adotar**, e o item de maior valor da lista. Comecar pelos contadores (`MarkedKept`/`SavedPages`) para
  detectar perda silenciosa.

### I-2 - Posse temporal dos bytes do convidado durante escritas em voo (`writeback.rs`)
- **Problema resolvido.** Uma fence pode completar **depois** de o titulo desmapear e reusar o endereco; o copy-back escreveria
  em memoria de outro objeto.
- **Implementacao.** `BTreeMap` de intervalos disjuntos com o numero da submissao mais recente ja iniciada; `STARTED` (`Release`),
  `PUBLISHED` (`fetch_max`), `LAST_CHANGE`; o intervalo so e libertado depois de a publicacao terminar.
- **Arquivos.** `crates/victory-gpu/src/writeback.rs` (321 linhas, novo em `bc3cd4b`).
- **Dependencias.** Ordem fence -> merge -> publicacao na thread de submissao (o comentario de cabecalho fixa essa premissa).
- **Risco.** Nao ha teste de concorrencia no modulo; a garantia e um argumento de ordenacao. Se outro fork paralelizar o merge,
  quebra.
- **Recomendacao.** **Adotar** o conceito (ownership por intervalo + numero de submissao) mesmo que a implementacao seja
  reescrita; escrever um teste com fence atrasada e reuso de endereco.

### I-3 - Memoria do titulo como memoria da placa (`inplace.rs`)
- **Problema resolvido.** Copia host no caminho de toda leitura/escrita de shader.
- **Implementacao.** Tres aberturas D3D12 documentadas com medicao: `OpenExistingHeapFromAddress` (so base de alocacao inteira,
  todas as paginas comprometidas, uma protecao; 256 MiB = 70 ms + 256 MB de working set), `OpenExistingHeapFromAddress1`
  (`ID3D12Device13`, qualquer parte comprometida, views sobrepostas), `OpenExistingHeapFromFileMapping` (aceita `SEC_COMMIT`,
  recusa `SEC_RESERVE`). Inclui probe (`VICTORY_INPLACE_PROBE`) e censo do que seria elegivel.
- **Arquivos.** `crates/victory-gpu/src/inplace.rs`, `memory.rs`.
- **Conflitos.** O pool do titulo e feito por `xmem` com `SEC_RESERVE` - justamente o que
  `OpenExistingHeapFromFileMapping` recusa. Unificar exige mudar `xmem` ou usar a via de endereco.
- **Recomendacao.** **Adotar como referencia medida**; o valor imediato e o texto (as regras do runtime/D3D12), que evita semanas
  de tentativa e erro. Validar no driver alvo, nao no WARP.

### I-4 - `texback`: textura que o shader escreve e a CPU do titulo le (`texback.rs`)
- **Problema resolvido.** No console CPU e GPU compartilham memoria: um passe de compute escreve um height map e o codigo de CPU
  do titulo o amostra. No PC, a textura fica na copia host e a CPU le memoria que ninguem escreveu. Sintoma observado: **os barcos
  afundavam** (`run rw1`, `VICTORY_READ_WATCH=1`: ~30 leituras do titulo a esses dois tipos).
- **Implementacao.** Para os tipos listados em `VICTORY_TEXTURE_BACK` (`<w>x<h>f<format>`), uma lista cujos shaders escrevem a
  textura copia para um read-back buffer ao fechar e, terminada a lista, **retila** os texels no layout do console dentro da
  memoria do titulo.
- **Arquivos.** `crates/victory-gpu/src/texback.rs` (novo neste fork).
- **Limitacao declarada.** E por **tipo**, nao por textura: texturas do tipo que o titulo nunca le tambem sao copiadas (~1 MB por
  lista). E "a copia mostra o que a ultima escrita da lista deixou": uma leitura do titulo no meio ve o frame anterior.
- **Recomendacao.** **Adotar** - e um caso de "shared memory semantics" que quase todo emulador de console esquece; o padrao
  (detectar por read-watch, corrigir por tipo, documentar o atraso) e reutilizavel.

### I-5 - `material_demand`: leitura antecipada de descritores guiada pelo proprio material
- **Problema resolvido.** Shader nativo indexa textura por indice vindo direto do registro de material; o heap pass sobe as
  texturas depois, e o objeto aparece escuro por varios frames (a "waterfront house" levou 18 s para ganhar detalhe,
  `BENCHMARK.md`).
- **Implementacao.** Le **o mesmo buffer de GPU que o shader** le, em ordem de comando, e agenda as texturas indicadas; o heap pass
  normal continua dono de validacao, upload e retry. `LIMIT = 8`, `MAX_AGE = 4`, `TABLE_DWORDS = 688`, `SLOT_BYTES = 32`; opt-in
  por `VICTORY_MATERIAL_TEXTURE_DEMAND=1`.
- **Arquivos.** `crates/victory-gpu/src/material_demand.rs` (novo em `b8ff22e`), `descriptor/host.rs`, `draw.rs`, `shadow.rs`.
- **Risco.** Exige conhecer o layout exato do registro de material (688 dwords, slots de 32 bytes) - e **especifico do titulo**.
  Se o layout mudar, agenda lixo.
- **Recomendacao.** **Adotar com isolamento**: manter atras de switch e por titulo, como esta. O conceito (prefetch de recurso
  dirigido pelo dado que o shader vai ler) e generico.

### I-6 - Emulacao das instrucoes de BVH do console em DXIL + referencia Rust bit-a-bit
- **Problema resolvido.** O jogo nao usa API de RT; traz BVH no formato do console e usa `image_bvh64_intersect_ray` /
  `image_bvh_intersect_ray`. Mapear para DXR e "nao pratico" (a camada teria de conhecer geometria que o titulo nunca descreve) -
  a justificativa esta escrita em `docs/fork-ray-tracing.md`.
- **Implementacao.** `bvh.rs` e a **referencia em Rust** (slab test, rede de ordenacao, teste agua-estanque de raio/triangulo);
  `emit/intersect.rs` emite DXIL equivalente; **311 casos de raio/no** rodam na GPU e sao comparados bit a bit com a referencia.
  `packed_mask.rs`/`own_bits.rs`/`wave64_tests.rs` tratam mascaras de lane como numeros e wave64 sobre wave32.
- **Arquivos.** `crates/victory-shader/src/bvh.rs`, `emit/intersect.rs`, `emit/intersect_tests.rs`, `emit/typed_views.rs`,
  `emit/guest_address.rs`.
- **Conflitos.** A base tem `emit/lower/bvh.rs` e `emit/bvh_tests.rs` com outra estrutura; unificar exige escolher um dos dois
  (o deste fork tem a distincao de residencia de pagina, essencial com page table).
- **Recomendacao.** **Adotar `bvh.rs` como referencia de teste**, independentemente da implementacao escolhida. O padrao
  "referencia Rust + comparacao bit-a-bit na GPU" e o que torna essa emulacao auditavel.

### I-7 - Regra de atomicidade de conjunto para subsistema que "funciona pela metade"
- **Problema resolvido.** Builder parcial escrevia limites invalidos e crashava o titulo.
- **Implementacao.** `builder_runs()` exige host pipeline para **todos** os pipelines do builder; `builder_off(why)` desliga tudo;
  `VICTORY_BVH_BUILDER=1` liga, `VICTORY_BVH_QUARANTINE=1` mantem stores fora da memoria do titulo.
- **Arquivos.** `crates/victory-gpu/src/pipeline.rs`.
- **Recomendacao.** **Adotar.** E a defesa certa para qualquer tradutor cujo parcial seja pior que o ausente.

### I-8 - `WAVE_OPS`/quad nativo para pixel e mesh, com atalho por padrao
- **Problema resolvido.** Operacoes de quad em pixel shader precisam ser quad-nativeis (incluindo *helper lanes* de borda).
- **Implementacao.** `use_quad = quad && (pixel || mesh)`; `quad_pick(x, src, pattern)` com identidade para `0xe4`, um
  `quadReadLaneAt` para `0x00/0x55/0xaa/0xff`, `quadOp` para `0xb1/0x4e/0x1b`; DPP8 so quad-local se cada destino fica no proprio
  quad.
- **Arquivos.** `crates/victory-shader/src/emit/lower.rs`.
- **Recomendacao.** **Adotar**; o atalho por padrao e o que viabiliza desempenho em shaders que usam DPP em todo lugar.

### I-9 - Deteccao de adaptador de hardware por dois sinais
- **Problema resolvido.** CI sem GPU rodava testes "hardware-only" no WARP.
- **Implementacao.** `hardware_adapter()`: percorre adaptadores e aceita o primeiro **nao** `DXGI_ADAPTER_FLAG_SOFTWARE` **e** com
  `DedicatedVideoMemory > 0`; sem nenhum, `Err("no hardware adapter")` -> skip conhecido.
- **Arquivos.** hoje so `crates/victory-shader/src/emit/host.rs` (o arquivo de teste que a usava foi apagado).
- **Recomendacao.** **Adotar em qualquer CI de GPU.** O sinal "flag software" sozinho nao basta - isso esta provado no CI deste
  repositorio (`ccfb029`).

### I-10 - Convencao de documentar compromissos no codigo com `shortcut:`
- **O que e.** 44 arquivos marcam a decisao consciente com `// shortcut: ...`, incluindo o custo ("BC1 ~30-60 MB/s"), o limite
  ("descriptors past the host's limit are left out") e o que fazer ("add them when a title uses"). Sem `TODO`/`FIXME`.
- **Por que vale.** Num codebase de emulacao, "nao implementado" e "implementado com atalho" sao estados diferentes e ambos
  precisam ser auditaveis. `git grep shortcut:` da um mapa de risco em um comando.
- **Recomendacao.** **Adotar como convencao** e indexar num documento unificado (o fork nao tem indice; so o grep).

### I-11 - `snapshot/` e `backup/` como refs de checkpoint nomeadas
`origin/backup/main-before-intro-2026-11-25` e `origin/snapshot/intro-tested` documentam, no proprio nome, o antes/depois de uma
operacao de risco. Custo zero, valor alto em arqueologia: permitiu reconstruir que `b8ff22e` foi o "intro testado" e `921d8d3` o
"antes". **Recomendacao:** adotar.

### I-12 - Correcoes de tradutor que valem para a base (nao sao especificas do fork)
Tres dos consertos catalogados atacam codigo **compartilhado com a base**, e nao o overlay privado:

| Correcao | Defeito | Por que serve na base |
|---|---|---|
| `ddb7332` (sampler por phi) | D-02, presente desde `2849477` | `Lower::image` e o mesmo; qualquer titulo com sampler escolhido por desvio uniforme perde os mesmos shaders |
| `76749ce` (mascara wave64 via `s_mov_b32`) | D-03, presente desde `2849477` | `structure.rs` e o mesmo; e o caso de wave64 sobre wave32, que a base tambem enfrenta |
| `921d8d3` (quad/Dpp8) | D-07, presente desde `2849477` | `mesh_quad` e literalmente a mesma linha na base |

- **Conflitos.** `ddb7332` depende de `image_with_sampler` (refatoracao local) e `76749ce` de `move_source`; ambos sao aditivos.
  `921d8d3` mexe em `quad_pick`, cuja assinatura muda (`pattern`), e em `use_quad` - conflito textual garantido com qualquer fork
  que tenha mexido no emissor.
- **Recomendacao.** **Portar estes tres primeiro**, antes de qualquer coisa da lista: sao os itens com melhor relacao
  valor/risco, porque corrigem codigo que a base tambem tem e vem com teste (inclusive um teste **negativo** em D-03).

## J. Conclusao tecnica

O DexNom/VIctory e um caso raro e instrutivo: um fork que **nao** e um fork incremental. Ele e, ao mesmo tempo,

1. **um integrador** - nos 17 minutos entre `01:07` e `01:24` de 2026-11-25 absorveu 137 PRs (287 commits, 98 arquivos), o que
   explica 306 dos seus 547 commits e a maior parte da sua vantagem de funcionalidade sobre a base;
2. **uma republicacao de uma linha privada** - `570a941` substituiu 170 arquivos (+49.715 / -36.762) por um estado descendente de
   `2849477`, e com isso **reescreveu todos os 98 arquivos** que a leva de merges havia tocado, sem deixar nenhum no estado
   pos-merge;
3. **um repositorio de reparo** - os 14 commits seguintes a `570a941` sao, quase todos, consertos do que a importacao quebrou:
   40 variantes de shader (`ddb7332`), 1 shader de voxel (`76749ce`), BVH e rastreio de escrita (`bc3cd4b`), LOD/quad/paginas/
   mip esparso (`921d8d3`), e dois testes (`4775e8a`, `2e01ab4`).

Tecnicamente, o que o fork entrega de novo e substancial e bem medido: ray tracing do **proprio titulo** emulado em DXIL com
referencia bit-a-bit (`bvh.rs` + 311 casos), memoria do titulo por endereco bruto atraves de um diretorio de paginas, escrita de
convidado rastreada com posse temporal (`writeback.rs`), retilagem de textura escrita por shader e lida pela CPU (`texback.rs`),
e uma reducao de frame time de City com RT de **3.469 ms para 969 ms** na mesma placa e mesmo save. A cobertura de traducao subiu
de 25.702 para **25.742 de 25.758** shaders, com **41 das 56** recusas originais recuperadas e validadas no DXIL da Microsoft.

O que ele entrega de pior e estrutural, nao pontual: a importacao **desfez correcoes que ja estavam no historico**, e tres delas
continuam desfeitas em `HEAD` - o clamp do staging (`D-10`, UB em `from_raw_parts_mut`), a escrita atomica da pipeline library
(`D-11`, corrida com `.tmp` de nome fixo) e a guarda de overflow do `clwb_loop` (`D-12`). Nenhuma delas e exotica: sao fixes
pequenos, com teste, integrados horas antes. O padrao 0-de-98 mede o tamanho do risco residual nos outros 95 arquivos, que nao
auditei.

O estado final e, portanto, **melhor em capacidade e pior em higiene** do que a base no mesmo ponto: um runtime que faz coisas
que a base nao faz, carregando regressoes que a base ja havia corrigido, com documentacao de causa raiz excelente (o
`docs/fork-ray-tracing.md` e o `BENCHMARK.md` sao modelos) e sem nenhum processo que detecte a divergencia - nao ha CI que
compare com a base, nao ha auditoria de importacao, e o ultimo commit do repositorio e o conserto de um teste que o penultimo
introduziu.

Para um repositorio unificado, a recomendacao e assimetrica e clara: **trazer os mecanismos** (I-1, I-2, I-4, I-6, I-7, I-9),
**portar os tres consertos de tradutor que atacam codigo compartilhado** (I-12: `ddb7332`, `76749ce`, `921d8d3`), **nao trazer a
arvore**; e, antes de qualquer coisa, reaplicar `4fc9d70`, `be824d9` e `274f698`, que sao a divida tecnica concreta deste fork.

## K. Indice de commits

Todos os 547 commits de `HEAD`, em ordem cronologica (mais antigo primeiro). Formato: `hash data autor assunto`.

```text
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
f5cb5ff 2026-11-23 Peter Bonanni Capture startup breakpoint diagnostics
a3acc87 2026-11-23 Peter Bonanni Check the required startup archive before executing Series 1.0.2.1
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
628b95d 2026-11-24 Manuel Pérez GPU: hold SHADER_HEAPS across descriptor mirror and check query offsets
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
ba42db6 2026-11-24 tedin7 xma: start the frame after a continuation at the next packet that has frames
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
d01b170 2026-11-24 Duff XAsync: end the call that finished, not whatever call the block holds now
46610eb 2026-11-24 Marvis Merge pull request #81 from tedin7/vic-scalar-carry
4926f72 2026-11-24 Marvis Merge pull request #80 from tedin7/vic-far-jump-backward
b1bdd89 2026-11-23 Bloxive Census: report directory traversal failures
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
e5b7159 2026-11-24 TG0000 Shaders: walk a lane-mask diagram once in vars and given
b9ff219 2026-11-24 Duff XTaskQueue: terminating a queue cancels its delayed callbacks too
91e9bc9 2026-11-24 Marvis Merge pull request #46 from 9hvn9yt2vk-netizen/shader-loop-limit
4c0bb89 2026-11-24 Marvis Merge pull request #41 from LaunchDoesntCode/codex/proton-warp-safety
8a8a38d 2026-11-24 Marvis Merge pull request #53 from a55uka/fix/wine-physical-pages
9aef3d9 2026-11-24 Marvis Merge pull request #51 from BaggyALPHA/fix/windows-commit-limit-diagnostic
01ae55a 2026-11-24 Duff Files: a title run by its file name alone gets its own folder as G:
7102a49 2026-11-23 LUIS LARIOS GPU: rename the second test module in host.rs so the tests compile
1823079 2026-11-24 lordon1a Shaders: lower v_cmp_class_f16 and v_cmpx_class_f16
d2a4bae 2026-11-24 Marvis Merge pull request #129 from Luislariosss/fix/duplicate-test-module
7d73e2b 2026-11-24 Codex pipeline: remove staging files when cache publication fails
02fcf72 2026-11-24 Maraco91 xmem: mapping or unmapping no pages changes nothing
4eb0898 2026-11-24 TG0000 XMA: build the inverse transform's rotations once per subframe size
810f442 2026-11-24 Manuel Pérez GPU: a frame wait takes its tick before it sleeps
babe2db 2026-11-24 Duff Crash reports: show the bytes of an instruction the CPU refused
d4d1822 2026-11-24 AnoHunted XMA: noise tests, and end a frame at its data instead of looping forever
3c7d845 2026-11-24 Maraco91 Kernel: a title image's file name that fits clears the last error
985d8d4 2026-11-24 Maraco91 Audio: an XMA loop's skip is of its first frame, not of its last
3390e47 2026-11-24 Duff Crash reports: say what an access violation's address holds
64484b4 2026-11-24 Alpaca-exe Input: apply VICTORY_KEYS to a connected host pad too
c63262f 2026-11-24 Peter Bonanni Merge upstream main into startup breakpoint diagnostics
7f00a62 2026-11-24 Duff Crash reports: name the host's CPU and Windows build
cbb50b8 2026-11-23 julio GPU: give null texture UAVs the title's view dimension
11ca3ce 2026-11-24 Maraco91 Media: a share is refused however it is spelled
c88b7ac 2026-11-24 Maraco91 Game saves: an empty configuration id is the default, not every configuration's folder
0de9efc 2026-11-24 Maledetto Loader: clzero loops in one exception; rdpru
afc4641 2026-11-24 Andrea Casale net: skip newer-Windows-only exports in only_loopback_is_reachable
18b2716 2026-11-24 Andrea Casale net: rustfmt the presence assertion
e57e76a 2026-11-23 Diego fix(pe): validate section raw data bounds
d46b078 2026-11-24 Duff XUser: a gamertag without a suffix has an empty modern suffix
acdcc41 2026-11-24 Manuel Pérez Loader: suspend a thread before reading its TEB; skip bad dump times
5f670bb 2026-11-24 Duff Loader: a trapped unimplemented import goes into the crash log
36d6a79 2026-11-24 Maraco91 XTaskQueue: terminating a queue ends a dispatch waiting on it
79fb722 2026-11-24 Duff Shaders: s_ttracedata_imm is SOPP opcode 0x28, not 0x26
5cff000 2026-11-24 Maraco91 XAsync: a new block where a result was left behind begins a call
63df363 2026-11-24 Duff GPU: a half-float rounding carry raises the exponent, not halves the value
1054fe8 2026-11-24 AnoHunted Audio: an XMA read block past the ring's end wraps instead of filling forever
717f322 2026-11-24 Maraco91 GPU: GetDevice and the DXGI device hand out the device at every level
5f29f3a 2026-11-24 Duff Loader: a title DLL that fails to prepare is named in the error
92f2c60 2026-11-24 Maraco91 GPU: every object answers for IUnknown
9fe73ba 2026-11-23 artor410 shader: decode the packed typed-buffer formats
a3d3bc9 2026-11-24 Claude Shaders: trace descriptors chosen by s_cselect and s_cmov
ffa6635 2026-11-24 Claude Shaders: lower waterfall loops over per-lane descriptors in one pass
b5c6678 2026-11-24 Abdellah K. loader: unwind a 16-byte aligned copy of the context
b58d198 2026-11-24 jsthope GPU: copy into title memory without asking for its region first
e3bef08 2026-11-23 marquezini XStore: a store id without an installed package is an invalid argument
3c81420 2026-11-24 hophaver GPU: drop draws whose fresh tables cannot be made; 2^18 dynamic slots
cc63aa2 2026-11-23 Enzo Souza Duarte Config: ignore an empty VICTORY_TEST_TITLES
c6959ac 2026-11-24 Duff HLE: load a host DLL outside the module cache's lock
93a24b7 2026-11-23 Pedro Fortunato fix(formats): reject overlapping texture depth slices
d549b63 2026-11-24 Duff HLE: a '>' in a quoted manifest attribute does not end its tag
e6ebec9 2026-11-24 FIocker Shader info: show stage I/O even when instruction decoding fails
40c127c 2026-11-23 Enzo Souza Duarte Chore: clear two workspace Clippy errors
efb3e3d 2026-11-24 FIocker GPU diagnostics: order tied drop counts consistently
3f5eff5 2026-11-24 b000m input: with no device, previous and next readings walk every device in time order
936e5e5 2026-11-23 marquezini XGameUi: a text entry's limit is the console's
376b3cf 2026-11-23 marquezini Storage: a path buffer too small fails with E_FAIL, as on Xbox
a4e7f1d 2026-11-24 FIocker GPU: replay pending descriptors before publishing the host heap
0f369d6 2026-11-23 marquezini Capture: R16G16B16A16_UNORM targets are integers, format 89 is 10:10:10:2
c000581 2026-11-23 marquezini Kernel: GetModuleFileName of a title image into a null buffer
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
7d04943 2026-11-24 Duff Shaders: a signed 64-bit operand sign-extends its 32-bit literal
077ec8b 2026-11-24 Claude translate: count per-lane fetched descriptors as loaded
bf84134 2026-11-24 Duff Shaders: maximums and minimums order -0 below +0, as RDNA2 does
e6fe8ec 2026-11-24 nathanbzrr GPU: fix CreateSamplerX mip-none base level, LOD range, bias clamp, and invalid enum values
2c22e2b 2026-11-24 nathanbzrr XStore: wire offline *Result slots and fix OOB XStoreShowProductPageUIResult slot
a3f89ed 2026-11-24 nathanbzrr Census: deduplicate PE imports without hint
75fe6c0 2026-11-24 Enzo Souza Duarte HLE: require two hex digits in percent escapes
b956320 2026-11-24 broccolize GPU: handle depth/stencil copy footprints
e8d3b64 2026-11-24 hophaver GPU: the larger dynamic range only where the heap still fits
6285e19 2026-11-24 Claude Shaders: take the first lane's descriptor as each lane's own only in a waterfall loop
65062e1 2026-11-23 LF2000 Builder xmem: report allocations the host refuses
7a5d85e 2026-11-24 AnoHunted Audio: an XMA write block past the ring's end wraps before the first write
b796c8b 2026-11-24 broccolize GPU: clear DXGI outputs on out-of-range enumeration
6638098 2026-11-24 r4. Shaders: per-lane descriptors chosen by control flow
c249f02 2026-11-23 Saitken91 Shaders: centroid position with conservative depth
705a2f6 2026-11-24 Aws Inline uniform micro-tile runs in texture copies
9a8908c 2026-11-24 Advait Palve core: show data_dir in config and check its asset policy
25b0f88 2026-11-24 Dathinel Test probe: compare direct and joined reads of a shader timestamp
3dd467c 2026-11-23 KoLolX3 Shaders: a phi that only another read phi consumes is live too
81b81df 2026-11-24 Devin Shaders: reuse lane guards for unchanged EXEC within a block
e950e81 2026-11-24 LF2000 Builder xtask: count \x escapes and 0x hex arrays as encoded runs in the policy check
a4974e4 2026-11-23 KoLolX3 GPU: VICTORY_* switches set to 0 are off
691236d 2026-11-24 JFIREonFIRE GPU: align appended input-element offsets like D3D12
6325df7 2026-11-24 LF2000 Builder Shaders: print only the operands an instruction has in SP3 text
988fd9a 2026-11-24 LF2000 Builder translate: refuse a VICTORY_EMIT_DUMP directory inside the repo
350f91c 2026-11-24 AlanQiang0418 Formats: reject multisampled linear texture layouts
260611a 2026-11-23 Saitken91 Shaders: image_sample_o with constant offsets
a0efd2d 2026-11-24 AlanQiang0418 Shaders: decode GFX10 packed inline constants by operand type
12eba49 2026-11-24 nathanbzrr Shaders: fix whole-wave EXEC inline constant sign-extension and prefix lane validation
b1c8306 2026-11-24 fuzeofficial Shaders: prove EXEC selection before collapsing a waterfall
43e2ca9 2026-11-24 Thomas Askharoun Show on-disk file size in victory inspect
6c63dea 2026-11-24 r4. Shaders: name why a loop is no waterfall, and test the shapes found in review
d74558c 2026-11-24 Carlos Sant'Anna GPU diagnostics: describe a buffer UAV by its range, not as uav None
764fa19 2026-11-23 lameloball2813-ui Loader tests: build the probe fixture under CARGO_TARGET_TMPDIR
0064c08 2026-11-24 fuzeofficial GPU: upload CPU-written textures before CopyResource
af22e5d 2026-11-23 EatCodeSleep2 GPU: use the host upload path for texture staging copies
5864aab 2026-11-23 lameloball2813-ui Docs: rustdoc builds the loader, GPU and shader crates without warnings
6526965 2026-11-24 OpenBot GPU: find a list in the DRED breadcrumbs by its name, not only its pointer
31ce30f 2026-11-24 fuzeofficial GPU: unbind vertex buffer slots for null view arrays
e34cb53 2026-11-24 nathanbzrr GPU: keep finite (min_lod, min_lod) in CreateSamplerX when mip filter is none
ab3eb92 2026-11-23 Louis Faust Shaders: gate the constant_color test helper to Windows
4f53d6f 2026-11-24 fuzeofficial GPU: upload CPU-written sources before texture region copies
0f0c872 2026-11-23 Louis Faust Docs: drop the private intra-doc link to MICRO in texture.rs
51f1627 2026-11-24 Lyrnc Shaders: v_cmp_lg is ordered on AMD hosts too (olt | ogt, not fcmp one)
3e39eac 2026-11-23 quaternion fix(hle): return XAsync payloads for nonnegative results
8a358e3 2026-11-24 OpenBot GPU: find the lists in the DRED breadcrumbs by position when the debug layer hides them
52d528b 2026-11-24 OpenBot GPU: match a breadcrumb node's name followed by the debug layer's note; drop the position fallback
882664e 2026-11-24 Manuel Pérez GPU: keep the frame wait's tick for the diagnostics log
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
efe102e 2026-11-24 EatCodeSleep2 Handle unsupported display deferrals in the HLE test
7216ab7 2026-11-24 Nixitov HLE: implement the RPC runtime's GUID calls
2005e35 2026-11-24 Marvis Merge pull request #88 from AndyLatimer/fix/title-size-overflows
c7a5f49 2026-11-24 Marvis Merge pull request #93 from TG0000/perf/bcpack-bucket-lookup
8e79c80 2026-11-24 Marvis CI: clippy and tests on pull requests and main
e09dbb6 2026-11-24 r4. Merge upstream main (8e79c80) into shader/per-lane-phi
74a9d33 2026-11-24 r4. Shader tests: the waterfall tests take the GPU one at a time, with the VALU tests
ca48fbe 2026-11-24 Marvis Merge pull request #566 from r4tedx/shader/per-lane-phi
ffe310e 2026-11-24 Marvis Shaders: the console's waterfall loop over a first-lane table offset
a32eacd 2026-11-24 DocBlack89 fix(pipeline): retry collisions when creating cache staging files
aaacc2d 2026-11-24 给伊莉雅补魔 docs(architecture): list BCPack as implemented in victory-formats
64b80c3 2026-11-24 TG0000 Formats: share immutable texture swizzle tables
121412b 2026-11-24 给伊莉雅补魔 loader(crash): verify unwind against known caller frames when RtlCaptureStackBackTrace returns empty
3e98bd7 2026-11-24 TG0000 GPU: seek directly to private data on final release
6d174ae 2026-11-24 Aws List successfully emitted shaders in offline translation scans
c430b1d 2026-11-24 Duff Shaders: tbuffer_load_format_d16 loads pack two 16-bit channels per VGPR
c1c3281 2026-11-24 pushkarreddyy GPU: extend dynamic descriptor wraparound scan to prevent false exhaustion
3d71ad1 2026-11-24 CaravelaLabs Shaders: s_atc_probe's immediate is not a register
072fde0 2026-11-24 swarm Exercise a free run straddling the cursor on the actual host heap
c1be75f 2026-11-24 pushkarreddyy Merge pull request #1 from MichaelSmithAI/test/871-cursor-crossing
131944d 2026-11-24 给伊莉雅补魔 PE: add synthetic unit tests for implib.rs and layout.rs
f821cb9 2026-11-24 TG0000 GPU: pipelines of different names load from the library at the same time
0be58e6 2026-11-24 给伊莉雅补魔 GPU: add synthetic unit tests for query.rs and signature.rs
d0dd9fc 2026-11-23 quaternion fix(pe): keep every name for a shared export ordinal
1aac20a 2026-11-23 quaternion fix(loader): stage prepared images in per-writer files
ca362a4 2026-11-23 quaternion fix(hle): read manifest fields past inline XML comments
71e8866 2026-11-23 quaternion fix(hle): honor the requested executable on relaunch
eb8067f 2026-11-24 quaternion fix(hle): keep apostrophes intact when splitting relaunch args
f47ebd8 2026-11-23 quaternion feat(shader): lower buffer cache invalidations as no-ops
52f0a09 2026-11-24 nino-exe policy: resolve a `..` in the not-yet-existing part of a path
fd44561 2026-11-23 Enzo Souza Duarte Config: ignore an empty VICTORY_CONFIG
60d2cf9 2026-11-24 BastienRdz Loader: a clwb in the last bytes of a page is emulated instead of faulting
246ecea 2026-11-24 Luiz Neves HLE: test that XGSuggestSwizzleMode picks linear for a 1D texture
45ed8e3 2026-11-24 Luiz Neves Shaders: hardware test for v_cvt_f16_u16 and v_cvt_f16_i16
f177aae 2026-11-24 Yaekai Shaders: cap BDD memo table and bail out early on overflow
8d67fa5 2026-11-24 Luiz Neves Shaders: keep the v_cvt_f16_u16/i16 test to integers f16 holds exactly
534e1dd 2026-11-24 David Palacios CLI: add AVX2 pre-flight CPU check before executing titles
03a6f22 2026-11-24 heromike405-arch XMA: gather frame payloads one byte at a time
259ba31 2026-11-24 Luiz Neves Shaders: negative-zero ordering test for the f16 min/max forms
8610ffb 2026-11-24 Luiz Neves Shaders: hardware test for the integer v_min3/v_max3/v_med3
e3c9dd0 2026-11-24 给伊莉雅补魔 GPU: guard zero/overflowing stride in expand_rects and add primitive.rs unit tests
ef0c3e9 2026-11-24 给伊莉雅补魔 GPU: validate null args in create_deserializer to prevent d3d12 AV and add root.rs tests
41b2472 2026-11-24 Luiz Neves Shaders: name the pick function type in the integer min3/max3/med3 test
42961e8 2026-11-24 bvtterbroot GPU: log the TDR settings, and warn when a GPU hang would freeze the PC
a7847a5 2026-11-24 Giuseppe Rainone HLE: constrain the title before suspending it
098a26a 2026-11-24 Eduardo Bertozzi README: how to install Rust on Windows (#1087)
44ce067 2026-11-24 BastienRdz XAsync: canceling a call that already completed leaves it alone, as documented
a14a545 2026-11-24 Maledetto Merge remote-tracking branch 'origin/main' into fix/clzero-loops-merge
514927d 2026-11-24 Maledetto Loader: tests for the clzero loop edge cases
47c00d5 2026-11-24 Blitz Studios GPU diagnostics: create missing capture output directories
142ec8d 2026-11-24 DoubleD17 Deduplicate image paths in victory census
344704e 2026-11-24 Rareuncommon Shaders: lower v_mul_hi_u32_u24 and v_mul_hi_i32_i24
5d6889e 2026-11-24 Rareuncommon Shaders: lower v_sad_u8, v_sad_hi_u8, v_sad_u16, v_sad_u32 and v_msad_u8
643fc6d 2026-11-24 divyeshganta HLE: avoid misaligned f32 output slice in XApu
64a33ec 2026-11-24 ashrafsdrop XNetworking: wire UDP port change callbacks and UTF-16 security queries
67c4f53 2026-11-24 bartikele victory-shader: resolve Structured::other and constant masks in v_cndmask_b32
59035de 2026-11-24 Launch GPU: grow per-list upload scratch on demand
4f54c47 2026-11-24 EatCodeSleep2 HLE: complete XPackage progress callback unregistration
1ed26e5 2026-11-24 EatCodeSleep2 Shaders: fold conditional branches to their fallthrough
100de0e 2026-11-24 skl Shader tests: a skipped device says why it was skipped
32eb934 2026-11-24 Omeya Sawant docs: explain Windows shader-test validator prerequisite
5c44562 2026-11-25 fuzeofficial Shaders: compress replaced phi chains in remove_trivial_phis
645391b 2026-11-24 Duff Shaders: a normalized buffer format's ends load as exactly 1.0 and -1.0
97c8783 2026-11-24 AnoHunted GPU: keep A8_UNORM out of the R8 typeless family
78c1267 2026-11-24 AnoHunted Shaders: lower v_cvt_off_f32_i4
e45bacd 2026-11-24 mgunes1311-lab Shaders: preserve mip point filtering in anisotropic static samplers
4e155a3 2026-11-24 mgunes1311-lab Shaders: do not parse VS inputs when reading mesh shader stage I/O
2b13ae4 2026-11-24 BaggyALPHA HLE clears the result of blocked address lookups
23fc8a5 2026-11-24 mgunes1311-lab Shaders: gate anisotropic point-mip host filters
7dfa86f 2026-11-24 mgunes1311-lab GPU: honor anisotropic point-mip device support
c3a04e2 2026-11-24 Luiz Otavio Mello de Campos Shaders: lower scalar quadmask instructions
e3fe5bf 2026-11-24 Luiz Otavio Mello de Campos Test scalar quadmask lowering on D3D12
a7636dd 2026-11-24 fluffyflattershy-afk HLE: allow an explicit maximum title window size
e52777a 2026-11-25 给伊莉雅补魔 GPU list: bind initial pipeline state in create_list and create_list_x
6f36cdf 2026-11-24 Luiz Otavio Mello de Campos GPU: interpolate normalized rectangle vertex attributes
be3202d 2026-11-24 Tex Shaders: fold empty loop latches containing only dead phis
711d651 2026-11-24 Tex Shaders: precompute MD5 round constants and avoid per-block Vec in md5_block
1e923ad 2026-11-24 Tex Shaders: set WAVE_OPS feature flag when lane_bit emits waveGetLaneIndex
2286e03 2026-11-25 给伊莉雅补魔 Shaders: key immediate sampler slots by descriptor words and comparison mode
4ab20fc 2026-11-24 Davide GPU: the indirect budget is charged in record order and no longer wraps
9ccb83c 2026-11-24 Calvix2k HLE: guard dialog commands before initialization
274f698 2026-11-24 Tex Loader: guard clwb_loop shortcut against u64 overflow and wraparound
f078008 2026-11-24 Vyacheslav Venglovsky Formats: test texture layouts against addrlib's own sizes and addresses
bbdc3ca 2026-11-24 BastienRdz GPU: fences answer for ID3D12Fence1 as documented
3f6910a 2026-11-24 Aws CLI: propagate the title entry point exit status
e9cb14a 2026-11-24 AgneyaK run: stop with a clear message when the title's own DLLs are missing
f096774 2026-11-24 Aws GPU: throttle failed swap-chain creation and resize
be824d9 2026-11-25 HailLucifer666 GPU: serialize pipeline library atomic writes with unique temp files
eb7cc2d 2026-11-24 imjustprism flush a kept save blob that had to be copied
4fc9d70 2026-11-25 HailLucifer666 GPU: keep Staging::detile inside the staging buffer and harden committed()
9145d9d 2026-11-24 Vax HLE: show title windows through a stand-in that always reads its messages
6493dd9 2026-11-24 zhaibingye Shader tests: require a device readback in every waterfall test
8d3a025 2026-11-25 给伊莉雅补魔 gpu(list): log initial pipeline bound on reset/reset_x for parity with create_list
40f30be 2026-11-24 davutsimsekk Shaders: GFX10 FLAT offsets ignore bit 11
c0228d9 2026-11-24 tjthebigbearcat-hub victory-shader: mask mesh primitive vertex indices to 9 bits (drop edge flags)
d09f3aa 2026-11-24 Nico Panu DirectStorage: read a page list naming page 0 again before failing its request
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
13185f5 2026-11-24 BastienRdz GPU: GetDeviceRemovedReason reports the host device's state instead of E_NOTIMPL
e29391f 2026-11-24 Calvix2k Shaders: honor op_sel, omod and clamp on v_interp_p2_f16
146dfef 2026-11-24 pes7 gfx10: resolve the opcode name once, at decode time
b8dbcba 2026-11-25 moe-miau XGameSave: a stray or unreadable version folder never costs the save
0601955 2026-11-24 Vax Shaders: a helper lane's v_readfirstlane reads its own value
ed0dbfd 2026-11-24 Kevin Pratap Sidhu GPU: copy title writes under ranges a submission's shaders write
4e60ea3 2026-11-25 pes7 victory-gpu: drop an indirect call before its clamp, not after
b5ac27f 2026-11-25 Marvis Merge pull request #115 from ZekkJJ/codex/census-scan-errors
f8c7498 2026-11-25 Marvis Merge pull request #122 from Dufftwt/fix/xasync-ended-call-reused-block
ed6429e 2026-11-25 Marvis Merge pull request #132 from Dufftwt/fix/terminate-cancels-delayed-callbacks
95191f8 2026-11-25 Marvis Merge pull request #186 from Maraco91/kernel/module-file-name-last-error
5d1fbac 2026-11-25 Marvis Merge pull request #207 from Maraco91/media/unc-forms-are-not-local
65aef25 2026-11-25 Marvis Merge pull request #238 from dpalacios69/feat/cpu-avx2-check
c7f1ae3 2026-11-25 Marvis Merge pull request #335 from marquezinii/fix/store-package-identifier-error
a222fe3 2026-11-25 Marvis Merge pull request #357 from Dufftwt/fix/host-dll-load-outside-cache-lock
2d8cac2 2026-11-25 Marvis Merge pull request #476 from nathanbzrr/fix/census-dedupe-without-hint
a34ac58 2026-11-25 Marvis Merge pull request #538 from coderanonymus/xmem-report-refusals
e82a6ad 2026-11-25 Marvis Merge pull request #615 from KoLolX3/fix/env-switch-zero
ff15f98 2026-11-25 Marvis Merge pull request #636 from Nixitov/hle/allow-rpcrt4
417e3f9 2026-11-25 Marvis Merge pull request #664 from coderanonymus/translate-emit-dump-outside-repo
644ebeb 2026-11-25 Marvis Merge pull request #665 from coderanonymus/policy-escaped-hex
b00c14a 2026-11-25 Marvis Merge pull request #1000 from TG0000/perf/parallel-pipeline-loads
76bcd2b 2026-11-25 Marvis Merge pull request #1080 from giusepperainone/fix/constrained-before-suspend
88de5fc 2026-11-25 Marvis Merge pull request #1611 from AgneyaK/fix/missing-package-dlls
d4ef9eb 2026-11-25 Marvis Merge pull request #1677 from Calvix2k/codex/hle-window-standin-1616
26e03c3 2026-11-25 Marvis Merge pull request #1729 from moe-miau/fix/xgamesave-stray-versions
4bf5100 2026-11-25 Marvis Merge pull request #12 from Raxstreams/codex/startup-breakpoint-diagnostics
1117251 2026-11-25 Marvis Merge pull request #52 from AndyLatimer/fix/gpu-descriptor-mirror-and-query-bounds
c5b83b9 2026-11-25 Marvis Merge pull request #219 from DiegoFR20/investigation/pe-validation
7b96711 2026-11-25 Marvis Merge pull request #247 from Maraco91/xasync/new-block-at-abandoned-address
c8cbe8a 2026-11-25 Marvis Merge pull request #353 from FIocker/shader/show-stage-io
8595259 2026-11-25 Marvis Merge pull request #359 from Dufftwt/fix/manifest-quoted-angle-bracket
897efe0 2026-11-25 Marvis Merge pull request #370 from marquezinii/fix/text-entry-length-limit
ac2525d 2026-11-25 Marvis Merge pull request #373 from marquezinii/fix/module-file-name-null-buffer
e686512 2026-11-25 Marvis Merge pull request #455 from Quat3rnion/victory-relaunch-args
43b7c08 2026-11-25 Marvis Merge pull request #474 from nathanbzrr/fix/gpu-console-sampler-validation
ce2d91f 2026-11-25 Marvis Merge pull request #475 from nathanbzrr/fix/xstore-missing-result-slots
2c67b09 2026-11-25 Marvis Merge pull request #480 from enzoduarte1/hle/percent-decode-hex-digits
805317b 2026-11-25 Marvis Merge pull request #487 from Quat3rnion/victory-manifest-comments
54e7844 2026-11-25 Marvis Merge pull request #504 from broccolize/gpu/depth-stencil-copy-footprints
648a9ff 2026-11-25 Marvis Merge pull request #555 from Quat3rnion/victory-export-aliases
2fb667b 2026-11-25 Marvis Merge pull request #617 from yyc160/perf/reuse-shader-guards
9bd146d 2026-11-25 Marvis Merge pull request #618 from JFIREonFIRE/fix/append-aligned-input-offsets
dcd24fe 2026-11-25 Marvis Merge pull request #649 from askharoun/inspect-file-size
9b6d1c0 2026-11-25 Marvis Merge pull request #957 from LouisRiverstone/hle/swizzle-1d-test
05394a6 2026-11-25 Marvis Merge pull request #993 from TG0000/perf/shared-texture-swizzle-tables
e9f43ed 2026-11-25 Marvis Merge pull request #1082 from bvtterbroot/gpu-tdr-level
357a8bc 2026-11-25 Marvis Merge pull request #1091 from EduBertozzi/docs/install-rust
6d06afb 2026-11-25 Marvis Merge pull request #1098 from BastienRodz/hle/cancel-after-completion
d73d8c9 2026-11-25 Marvis Merge pull request #1109 from divyeshganta/fix/cache-staging-collisions
7582940 2026-11-25 Marvis Merge pull request #1161 from geiyiliyabumo/docs/architecture-bcpack-implemented
fc438a2 2026-11-25 Marvis Merge pull request #1171 from geiyiliyabumo/test/pe-implib-and-layout
69bab30 2026-11-25 Marvis Merge pull request #1173 from geiyiliyabumo/test/gpu-query-and-signature
0fcbee4 2026-11-25 Marvis Merge pull request #1177 from DoubleD17/fix/census-deduplicate-inputs
432b110 2026-11-25 Marvis Merge pull request #1204 from geiyiliyabumo/test/gpu-primitive-audit
02deab1 2026-11-25 Marvis Merge pull request #1206 from geiyiliyabumo/test/gpu-root-audit
5c4f2e9 2026-11-25 Marvis Merge pull request #1248 from ashrafsdrop/net/xnetworking-methods
6355117 2026-11-25 Marvis Merge pull request #1274 from EatCodeSleep2/fix/xpackage-unregister-progress-1249
eb17946 2026-11-25 Marvis Merge pull request #1299 from omeyasawant/docs/windows-shader-test-prerequisites
05f988c 2026-11-25 Marvis Merge pull request #1390 from AnoHunted/gpu-a8-typeless-family
7db1e07 2026-11-25 Marvis Merge pull request #1411 from BaggyALPHA/fix/clear-blocked-name-result
12ee4a1 2026-11-25 Marvis Merge pull request #1453 from Luizbc2/fix/rectangle-normalized-vertices
a985b9d 2026-11-25 Marvis Merge pull request #1455 from fluffyflattershy-afk/feat/window-size-limit
e011e88 2026-11-25 Marvis Merge pull request #1515 from Calvix2k/codex/ui-dialog-command-guard-033
17de445 2026-11-25 Marvis Merge pull request #1539 from Texbirds/fix/loader-clwb-loop-overflow
ebcd90f 2026-11-25 Marvis Merge pull request #1592 from AwsAli05/cli/list-emitted-shaders
cbb3431 2026-11-25 Marvis Merge pull request #1594 from AwsAli05/cli/propagate-title-exit
711e930 2026-11-25 Marvis Merge pull request #1658 from HailLucifer666/fix/texture-staging-oob-and-committed-guard
a9d233f 2026-11-25 Marvis Merge pull request #1660 from HailLucifer666/fix/pipeline-library-atomic-write-race
f69e487 2026-11-25 Marvis Merge pull request #1667 from imjustprism/fix/save-kept-blob-sync
d6e00a8 2026-11-25 Marvis Merge pull request #133 from Dufftwt/fix/bare-exe-name-title-folder
2a45e75 2026-11-25 Marvis Merge pull request #194 from Maraco91/audio/xma-loop-start-skip
280e484 2026-11-25 Marvis Merge pull request #941 from BastienRodz/loader/clwb-near-page-end
4d140e2 2026-11-25 Marvis Merge pull request #1718 from tjthebigbearcat-hub/fix/third-pr
12fa535 2026-11-25 Marvis Merge pull request #96 from tedin7/vic-xma-continuation
c813777 2026-11-25 Marvis Merge pull request #126 from TG0000/perf-pr5
46bde7f 2026-11-25 Marvis Merge pull request #169 from Dufftwt/fix/tbuffer-d16-loads-packed
71ac890 2026-11-25 Marvis Merge pull request #213 from Dufftwt/fix/crash-log-names-host
15de8f9 2026-11-25 Marvis Merge pull request #214 from Makto94/fix/clzero-loops
a9d6f17 2026-11-25 Marvis Merge pull request #322 from jsthope/gpu/title-copies-without-region-queries
1134418 2026-11-25 Marvis Merge pull request #447 from Dufftwt/fix/signed-64-bit-literals
5c0ebce 2026-11-25 Marvis Merge pull request #646 from nathanbzrr/shader/wave-exec-const-and-prefix-validation
ea48bd3 2026-11-25 Marvis Merge pull request #669 from iurysf/dred-breadcrumb-identity
0cffc54 2026-11-25 Marvis Merge pull request #937 from nino-exe/policy-resolve-dotdot
0299812 2026-11-25 Marvis Merge pull request #990 from LouisRiverstone/shaders/cvt-f16-int-test
181825d 2026-11-25 Marvis Merge pull request #1146 from jamesn-xt/fix/capture-output-directory
1c16afd 2026-11-25 Marvis Merge pull request #1214 from divyeshganta/codex/hle-apu-unaligned-output-ub
aa83e3f 2026-11-25 Marvis Merge pull request #1246 from LaunchDoesntCode/gpu/grow-list-upload-scratch
7ae5e17 2026-11-25 Marvis Merge pull request #1474 from Texbirds/fix/shader-lane-bit-wave-ops
41b11b2 2026-11-25 Marvis Merge pull request #1477 from geiyiliyabumo/fix/shader-static-sampler-cmp
fe88382 2026-11-25 Marvis Merge pull request #1510 from rouges78/fix/indirect-budget-in-order
8fb0b28 2026-11-25 Marvis Merge pull request #1549 from venglov/formats/addrlib-reference-layouts
94856e2 2026-11-25 Marvis Merge pull request #1635 from pes7/perf/indirect-skip-clamp
8479c18 2026-11-25 Marvis Merge pull request #1726 from npanu420/dstorage/late-page-lists
aa1bf51 2026-11-25 Marvis Merge pull request #153 from TG0000/perf/gpu-private-data-release
256d7b9 2026-11-25 Marvis Merge pull request #156 from Rareuncommon/shaders/mul-hi-24
83211b9 2026-11-25 Marvis Merge pull request #162 from Rareuncommon/shaders/sad
7d99f63 2026-11-25 Marvis Merge pull request #172 from TG0000/perf/xma-imdct-rotations
8a772ea 2026-11-25 Marvis Merge pull request #174 from AndyLatimer/fix/gpu-frame-wait-reserves-tick
338808e 2026-11-25 Marvis Merge pull request #183 from AnoHunted/xma-noise-tests
a3169a1 2026-11-25 Marvis Merge pull request #197 from Alpaca-exe/fix/victory-keys-with-connected-pad
3dce4ba 2026-11-25 Marvis Merge pull request #221 from AndyLatimer/fix/loader-stack-dump-safety
38011a0 2026-11-25 Marvis Merge pull request #270 from Maraco91/gpu/every-object-answers-iunknown
7448632 2026-11-25 Marvis Merge pull request #288 from Yaekai/fix/shader-bdd-memo-cap
32b9fbf 2026-11-25 Marvis Merge pull request #351 from pedrofort/fix-texture-depth-pitch
e8e494c 2026-11-25 Marvis Merge pull request #361 from booomxd/input/walk-all-devices
d763582 2026-11-25 Marvis Merge pull request #363 from enzoduarte1/config/empty-config-env
9ce9976 2026-11-25 Marvis Merge pull request #368 from Quat3rnion/victory-shader-coverage
13de0cc 2026-11-25 Marvis Merge pull request #536 from AnoHunted/xma-write-block-wrap
e56919d 2026-11-25 Marvis Merge pull request #553 from broccolize/gpu/clear-dxgi-enumeration-outputs
40f5553 2026-11-25 Marvis Merge pull request #560 from Quat3rnion/victory-image-publication
8bf20ad 2026-11-25 Marvis Merge pull request #603 from AwsAli05/formats/detile-copy-runs
9574cee 2026-11-25 Marvis Merge pull request #629 from AlanQiang0418/codex/reject-linear-msaa-layouts
02fcf5b 2026-11-25 Marvis Merge pull request #661 from coderanonymus/sp3-text-operands
c6c244d 2026-11-25 Marvis Merge pull request #974 from pushkarreddyy/fix/descriptor-cursor-range
924457e 2026-11-25 Marvis Merge pull request #1016 from heromike405-arch/perf/xma-frame-bytes
6e6004c 2026-11-25 Marvis Merge pull request #1021 from CaravelaLabs/caravela/smem-atc-probe
fdce5f5 2026-11-25 Marvis Merge pull request #1050 from LouisRiverstone/shaders/f16-minmax-neg-zero-test
19469bd 2026-11-25 Marvis Merge pull request #1052 from LouisRiverstone/shaders/int-min3-max3-med3-test
ef65f88 2026-11-25 Marvis Merge pull request #1125 from EatCodeSleep2/fix/display-deferral-test-727
e2a80cf 2026-11-25 Marvis Merge pull request #1159 from geiyiliyabumo/fix/crash-unwind-test-win10
82494a9 2026-11-25 Marvis Merge pull request #1180 from pes7/perf/instruction-name
6160c9e 2026-11-25 Marvis Merge pull request #1221 from bartikele/shader-cndmask-other-masks
825c045 2026-11-25 Marvis Merge pull request #1282 from EatCodeSleep2/fix/shader-branch-to-fallthrough-1242
87f01ff 2026-11-25 Marvis Merge pull request #1290 from sklsp/shader-tests/show-device-error
170ab23 2026-11-25 Marvis Merge pull request #1337 from mgunes1311-lab/shader-aniso-mip-point
21b4175 2026-11-25 Marvis Merge pull request #1366 from mgunes1311-lab/shader-native-ms-outputs
67bcf47 2026-11-25 Marvis Merge pull request #1371 from fuzeofficlal/fix/shader-phi-path-compression-950
5e3bc84 2026-11-25 Marvis Merge pull request #1391 from AnoHunted/shaders-cvt-off-f32-i4
95f6c6e 2026-11-25 Marvis Merge pull request #1443 from Luizbc2/fix/rdna2-quadmask
24effa8 2026-11-25 Marvis Merge pull request #1446 from geiyiliyabumo/fix/list-create-initial-pipeline
520fefc 2026-11-25 Marvis Merge pull request #1471 from Texbirds/fix/bitcode-fold-dead-phi-latch
f73e8e1 2026-11-25 Marvis Merge pull request #1476 from Texbirds/perf/dxbc-md5-table
3c47fd4 2026-11-25 Marvis Merge pull request #1562 from BastienRodz/gpu/fence1-iid
13d1237 2026-11-25 Marvis Merge pull request #1626 from AwsAli05/gpu/throttle-swap-chain-retries
3688adc 2026-11-25 Marvis Merge pull request #1636 from davutsimsekk/fix/flat-offset-twelve-bits
1bde03c 2026-11-25 Marvis Merge pull request #1695 from zhaibingye/fix/waterfall-require-device
fcdefcf 2026-11-25 Marvis Merge pull request #1322 from KevinPratap/fix/shadow-busy-title-writes
5fa5de3 2026-11-25 Marvis Merge pull request #1733 from moe-miau/fix/shader-helper-lane-readfirstlane
4436fca 2026-11-25 Marvis Merge pull request #1696 from Calvix2k/codex/shader-interp-p2-f16-029
4118f33 2026-11-25 Marvis Merge pull request #928 from BastienRodz/gpu/device-removed-reason
42363ab 2026-11-25 Marvis Tests: hardware-only shader checks skip on a machine without a GPU
ccfb029 2026-11-25 Ivan Tests: a machine with no GPU has no hardware adapter to check shaders on
cd94d35 2026-11-25 Marvis GPU: shaders mark the pages they write through views; copy-backs compare only those
940d32f 2026-11-25 Marvis GPU: a marked page's copy-back does not save the page for its own copy
570a941 2026-11-25 DexNom Publish DexNom Runtime Radeon sources with upstream credits and build instructions
42b6565 2026-11-25 DexNom Enforce the source-only policy for every path on the public branch
ef8f4f1 2026-11-25 DexNom Build and install DexNom Runtime from the source ZIP with Windows Setup
8c7937a 2026-11-25 DexNom Verify setup dependencies with the built-in .NET SHA-256 implementation
ddb7332 2026-11-25 DexNom Restore 40 shader variants and repair integer-target blending
fa007d1 2026-11-25 DexNom Correct Windows setup and attribution link separators
76749ce 2026-11-25 DexNom Recover voxel visibility shader when a wave64 mask is restored from scalar copies
5f4c3df 2026-11-25 DexNom Improve City texture streaming and recover Radeon shadow pipelines
bc3cd4b 2026-11-25 DexNom Restore ray structure building and track guest shader writes precisely
d71f197 2026-11-25 Codex Add an opt-in Windows shortcut for experimental ray tracing
8cca4d2 2026-11-25 DexNom Document experimental RT launch and measured City frame times
921d8d3 2026-11-25 Codex Repair pixel quad sampling, grow ray buffer views, and recover sparse textures
b8ff22e 2026-11-25 Codex Export the tested Radeon renderer for the optional RT preview
4775e8a 2026-11-25 Codex Isolate the material readback test from concurrent frame advancement
2e01ab4 2026-11-25 Codex Keep cached test textures within their reserved guest memory
```

## L. Limitacoes desta analise

- **L-1 - Sem execucao.** Nao compilei nem rodei a suite (`cargo test`) nem o runtime. Nao ha, portanto, nenhuma afirmacao minha
  sobre "passa/falha" que nao venha do proprio repositorio; os numeros de teste (163 passed / 246 passed / 666 `#[test]`) sao
  citacoes, nao medicoes. O estado do CI em `HEAD` e **NAO VERIFICADO**.
- **L-2 - Sem acesso a rede.** Nao consultei GitHub, issues (#871, #908, #950, #1242, #1249, #727 etc. aparecem apenas em
  mensagens de merge do historico) nem os PRs originais. Toda atribuicao de issue e **repeticao do texto do commit**, nao
  verificacao. Nao verifiquei o repositorio `ZSlayerHQ/victory-world-runtime` nem o `xcl-dev/xcl`, citados no README.
- **L-3 - Auditoria de regressao parcial.** Confirmei **5** perdas de fix (D-10, D-11, D-12, D-15, G-3) por inspecao de codigo, e
  **4** nao-regressoes por inspecao de mecanismo (`4ab20fc`, `8a772ea`/`810f442`, `60d2cf9`, `c1c3281`). Os outros 95 arquivos da
  leva nao foram auditados um a um; o numero "0 de 98 identicos" mede o risco, nao o dano.
- **L-4 - Merges sem diff.** 228 dos 547 commits sao merges; `git show` de um merge sem conflito nao mostra diff. Para esses,
  trabalhei pela mensagem, pelo `--stat` do commit de origem (quando identificavel) e pelo estado final do arquivo. Nao examinei
  individualmente as 137 folhas da leva de merges; examinei as que tocam os defeitos catalogados.
- **L-5 - Arvore privada inacessivel.** A origem real de `570a941`/`5f4c3df`/`b8ff22e` e um repositorio privado (o commit cita
  revisoes `c004bfc`/`29649ac` que nao existem aqui). Para os defeitos "introduzidos por importacao", o commit de introducao *no
  historico publico* e a importacao; a origem anterior e **NAO VERIFICADA**.
- **L-6 - Semantica de GPU assumida.** Onde descrevo efeito visual (D-04, D-06, D-07) inferi do codigo e dos comentarios; nao ha
  comparacao de imagem no repositorio alem das descricoes do `BENCHMARK.md`. Marquei esses casos como **PROVAVEL**.
- **L-7 - Divergencia entre fontes.** `_fatos/FORK_DIVERGENCIA.md` afirma que o HEAD da base nao e ancestral dos forks; neste
  clone, `70564ae` **e** ancestral de `HEAD`. Reportei a refutacao na secao A com o comando. Nao consegui determinar a causa da
  discrepancia (metodo ou clone diferente); nao a tratei como erro do autor do pacote de fatos.
- **L-8 - Contagem de linhas e testes.** `120.287` linhas e `666` testes foram contados por `git show` arquivo a arquivo e
  `git grep -c "^\s*#\[test\]"`; testes condicionais (`#[cfg(test)]` sem `#[test]`, testes com `#[ignore]`) e macros de teste nao
  entram na contagem.
