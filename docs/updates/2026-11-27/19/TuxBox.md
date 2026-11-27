# brcly/TuxBox — atualização de 2026-11-27 19:00

## Resumo

Doze commits na `main` desde `cf0a0824`, mas só quatro mudanças de conteúdo: um pacote de dedupe e
endurecimento do cache DXIL do launcher (#239), o splash de pacote no `d3d12` (#311) e a retentativa
offline da recusa de stride IDXEN (#310), mais um ajuste de teste. Os outros oito são merges — três
deles repetem o diff do commit de branch correspondente, e um (`d33de277`) mescla `origin/main` na
branch de trabalho e por isso aparece com 116 arquivos que já estavam documentados. Três PRs
mesclados, nenhum PR aberto acima do cursor.

## Commits novos (12)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `e91eedad` | Valdi320 | launcher: deduplicate concurrent DXIL compilation by HLSL content | Em `project_tuxbox/runtime_shaders.py` (+30/−15): o lock por fonte exata não deduplica pedidos diferentes que emitem HLSL idêntico. Depois de um miss, adquire um lock de conteúdo (`compile_lock(store.root, content, timeout=O0_TIMEOUT_MAX + COMPUTE_TIMEOUT + 60)`), sempre fonte antes de conteúdo, e reconsulta o cache sob o lock antes de compilar. Um teste novo usa `ThreadPoolExecutor` e duas `Barrier(2)` para forçar as duas buscas a errarem antes de qualquer publicação e verifica que só uma compilação acontece. `docs/ENVIRONMENT.md` ganha o parágrafo correspondente. |
| `2057645f` | Valdi320 | launcher: reuse one bounded verified DXIL snapshot | Extrai para `project_tuxbox/shader_blob.py` (+39) os utilitários `MAX_SHADER_BYTES`, `bounded_shader`, `exact_marker` e o novo `bounded_regular`; `dxil_store.fetch()` passa a ler o `.cso` **uma vez** como bytes limitados, calcular o SHA-256 desses bytes e só então comparar com o marcador e gravar no destino — antes copiava o arquivo e relia para conferir. `save()` recebe bytes já validados e recusa acima de 64 MiB. `IDENTITY_FILE` também passa a ser lido por `bounded_regular`. |
| `2fb96ad7` | Valdi320 | launcher: hash generated shader content incrementally | `dxil_store.key()` monta o digest com `hashlib.sha256(tag).update(hlsl)` em vez de concatenar `tag + hlsl` numa única `bytes`; o chamador de `runtime_shaders.py` acompanha. Evita a cópia do blob inteiro só para calcular a chave. |
| `ce7edf2b` | Valdi320 | launcher: reject wrong-stage snapshots before counting reuse | Em `dxil_store.py` (+3/−2) e `runtime_shaders.py` (+1/−1): a checagem de estágio portátil passa a acontecer **antes** de incrementar o contador de reuso; snapshot de estágio errado é descartado sem contar como acerto. |
| `b460d1ce` | Valdi320 | test: preserve snapshot validator in concurrent cache control | `tests/test_dxil_store.py` (+2/−2): o controle concorrente mantém o validador de snapshot no lugar do mock que o substituía. Só teste. |
| `845a4029` | — | Merge pull request #239 from Valdi320/shader-dedupe | Merge do pacote acima: o diff contra o primeiro pai é o conjunto consolidado (+319/−62 em `dxil_store.py`, `runtime_shaders.py`, `shader_blob.py`, dois testes e `docs/ENVIRONMENT.md`). |
| `d6d40659` | — | Merge commit 'ce7edf2' into work/content-cache-single-flight-20261126 | Resolve conflito da branch com `ce7edf2b` em `dxil_store.py`, `runtime_shaders.py`, `shader_blob.py` e dois testes (+127/−45). Sem conteúdo novo além da resolução. |
| `d33de277` | — | Merge remote-tracking branch 'origin/main' into work/content-cache-single-flight-20261126 | Merge de `origin/main` na branch de trabalho: 116 arquivos e +45.889/−726 **contra o primeiro pai**. O segundo pai é `6ce0aac8`, que está 104 commits **atrás** do cursor `cf0a0824` (`compare` = `behind`, `ahead 0`), ou seja: o conteúdo é trabalho de `main` já documentado em ciclos anteriores, trazido para dentro da branch. Não é delta novo. |
| `7423e478` | MathiewMay | d3d12: show the package splash until the title presents steadily | Novo `runtime/d3d12/src/present/splash.rs` (733 linhas) e +178/−28 em `frame_present.cpp`. O hook `SetWinEventHook` in-context roda na thread que cria a janela do título, antes de ela aparecer: ajusta a janela ao tamanho que o primeiro present pediria (16:9, 3840×2160) e a deixa transparente; o splash é uma janela sem borda do próprio TuxBox no tamanho da imagem, com o alfa dela. Frames do título completam sem chegar à tela até `STEADY_FRAMES = 4` chegarem dentro de `STEADY_WINDOW = 1 s`, ou no máximo `LONGEST = 60 s`; o estilo é devolvido **antes** de criar a swap chain (tirar o layering com swap chain viva deixava os presents do Wine lentos por segundos, com espera de `UNLAYER_WAIT = 5 s`); o splash fecha no primeiro frame do título. `build.py` passa a gerar o fixture `splash/MicrosoftGame.Config` + PNG 64×36 e `verify.py` ganha o caso `guest-present-4k-splash`. |
| `8e641a06` | Pavel Kirilin | shader-prep: retry a IDXEN stride refusal with the appended stride table | +488/−6 em 8 arquivos. `runtime_shaders.py` ganha `stride_table_retry()`: quando o `xshader` recusa com a linha `raw buffer IDXEN needs the console descriptor stride`, roda de novo com `--stride-table`, `--hlsl-out` e `--profile-out` e publica a variante sob o digest `stride-digest=<sha256>`, mantendo os marcadores de recusa da chave original para a retentativa do runtime. `runtime/dxbc/src/rootsig.rs` (+141) ganha `stride_table_variant()`: lê o chunk RTS0 embutido, valida cabeçalhos/corpos/samplers estáticos e anexa um root SRV volátil (`t1, space32766`, um u32 por slot do guest) antes dos samplers, deslocando o resto do corpo em `12 + 8` (v1) ou `12 + 12` (v2) bytes. `scarlett_report.py` classifica a recusa como `heap_stride`; `tools/shader-compiler` ganha a flag `--stride-table` e imprime o digest. |
| `d35d3bb3` | — | Merge pull request #310 from s3rius/stride-table-prep-retry | Merge de #310; diff contra o primeiro pai idêntico ao de `8e641a06` (mesmos 8 arquivos, +488/−6). |
| `c2a59af5` | — | Merge pull request #311 from MathiewMay/splash-screen | Merge de #311; diff contra o primeiro pai idêntico ao de `7423e478` (mesmos 11 arquivos, +1.110/−42). |

## Pull requests abertos (0)

Nenhum PR acima de `last_pr_number` (311). O repositório tem 3 PRs abertos, todos com número ≤ 311,
já contabilizados em ciclos anteriores.

## Pull requests mesclados (3)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #239 | launcher: deduplicate shader compilation and verify one bounded cache snapshot | `845a4029` | Lock de conteúdo depois do miss (fonte antes de conteúdo), reconsulta sob o lock, `fetch()` como leitura única limitada e verificada por SHA-256, `save()` de bytes já validados, chave calculada incrementalmente, rejeição de snapshot de estágio errado antes de contar reuso. Testes de concorrência com barreira. |
| #310 | shader-prep: retry a IDXEN stride refusal with the appended stride table | `d35d3bb3` | Retentativa offline com tabela de stride anexada: variante de root signature em `rootsig.rs`, publicação sob digest próprio, marcadores originais preservados; classificação `heap_stride` no relatório; flag `--stride-table` no compilador. |
| #311 | d3d12: show the package splash until the title presents steadily | `c2a59af5` | Janela de splash própria com hook de criação de janela, ajuste de tamanho e transparência, retenção de frames até estabilizar (4 em 1 s ou 60 s), devolução de estilo antes da swap chain, fixture e caso de verificação 4K. |

## Cruzamento com o catálogo

- **Não corrige D-02 nem D-03.** O catálogo registra "Otimizador do DXC estoura o tempo; shaders viram
  pipeline nulo" (D-02) e "conversão DXIL→SPIR-V leva minutos em `cs_6_6` compilado em `-O0`" (D-03).
  O #239 ataca o **trabalho repetido**: pedidos distintos que emitem HLSL idêntico passam a dividir uma
  compilação, e o snapshot é lido uma vez só. O timeout do otimizador e o custo de uma compilação
  individual não mudam. É mitigação de custo, não correção dos dois defeitos.
- **Não corrige D-12, apesar da palavra "stride".** D-12 é o *stride de descritor inline* de 32 bytes
  incompatível com o RADV. O `8e641a06` trata de outra coisa: o *stride do heap* que o descritor de
  buffer do console carrega na palavra 1, sem o qual um acesso raw IDXEN não consegue escalar o
  índice; a solução é anexar um root SRV volátil `t1, space32766` com um u32 por slot e traduzir o
  programa contra ele. Mecanismo e camada diferentes — **não fecha D-12**.
- **D-27/D-28** (issues abertas sobre slot 80 tratar threads como grupos e `WaveSize(N)` recusado só no
  driver) não são tocados por nenhum dos 12 commits.
- **Abre defeito novo?** O splash muda o caminho de apresentação (estilo de janela, layering, retenção
  de frames). O caso `guest-present-4k-splash` cobre a janela e os frames, mas uma regressão em jogo
  real não foi medida aqui. **NÃO VERIFICADO**.

## Confiança

- "Doze commits" é a contagem do `compare`, não doze mudanças. `d35d3bb3` repete `8e641a06`,
  `c2a59af5` repete `7423e478` e `d6d40659` é resolução de conflito da mesma branch; `d33de277`
  traz conteúdo anterior ao cursor. As três mudanças de conteúdo estão em `e91eedad`/`2057645f`/
  `2fb96ad7`/`ce7edf2b` (dedupe e cache), `7423e478` (splash) e `8e641a06` (stride IDXEN).
- Li os diffs de `runtime_shaders.py`, `dxil_store.py`, `shader_blob.py`, `rootsig.rs` (a função
  `stride_table_variant` inteira na parte de validação e crescimento), o cabeçalho de `splash.rs`, o
  preâmbulo de `frame_present.cpp`, `build.py` e `verify.py`. **Não li** as 733 linhas de `splash.rs`
  nem o diff de 116 arquivos de `d33de277` linha a linha.
- Nenhum teste foi executado; as contagens de teste citadas acima são do que o diff acrescenta, não
  de uma execução minha. O commit `b460d1ce` é um ajuste de teste que corrige um mock quebrado.
- O par `stride-digest` publicado em `8e641a06` depende da retentativa do runtime ler os marcadores
  originais; li o caminho no launcher, **não** li o lado do runtime (`compute_table`) que consome a
  variante.
- Head `c2a59af5` = cursor novo; `last_run` sobe para `2026-11-27T15:28:30Z` (o maior `merged_at` da
  janela, os três no mesmo segundo).
