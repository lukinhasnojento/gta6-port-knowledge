# ivall/xcl — atualização de 2026-11-27 19:00

## Resumo

Um único conteúdo novo na `main`: `xcl pso-precompile` (#1871), que traduz os programas de compute
de um pacote de shader e manda o driver compilar os pipelines antes da execução, para encher o cache
de blobs que o runtime lê. O commit de conteúdo (`062158ea`) tem data de autor de 24/11 e só agora
entrou na `main` — não é cursor retrocedendo. Seis PRs abertos acima do cursor, três deles uma pilha
sobre `main` que corrige a ordem escrita→espera dentro de uma command list.

## Commits novos (2)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `062158ea` | bmjask | PSO: xcl pso-precompile warms the driver-blob cache before a run | Três arquivos. `crates/xcl/src/main.rs` (+7) registra o subcomando `pso-precompile <pasta do jogo \| shaders.rpf>`; `crates/xcl/src/shaders.rs` (+194/−23) acrescenta `pso_precompile()` (resolve o pacote, monta `--precompile --pipelines` e delega a `run`), `--records FILE` (hashes/pipeline keys em hexa) e o alias `--used`, e liga `--threads` ao `precompile()` pela nova `pipeline_threads()`: `XCL_PIPELINE_THREADS` se definido, senão `min(cores, RAM_GB/2)` limitado a 1..32 por `default_pipeline_threads()`; `crates/xcl/tests/shaders_e2e.rs` (+27) cobre o caminho. O help diz para rodar com o mesmo `XCL_DIRECT_DXIL`/`XCL_HEAP_WINDOWS` da execução, porque mudam as chaves. |
| `f9a0cb80` | Prime1997 | Merge pull request #1871 from bmjask/perf/pso-aot — PSO: xcl pso-precompile builds a title's compute pipelines before its run | Merge de `062158ea` na `main`. O diff contra o primeiro pai é idêntico ao commit de conteúdo (mesmos 3 arquivos, +228/−23): é merge normal, sem resolução. |

## Pull requests abertos (6)

Lidos por corpo; nenhum patch lido linha a linha.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #3033 | D3D12.X: track exact command-list write intervals | Primeiro de três PRs empilhados (item 3 de #2787). As faixas aproximadas e limitadas de escrita por lista viram intervalos exatos numa árvore, para as partes seguintes saberem **qual produtor** escreveu um intervalo e preservarem lacunas. "Muda nenhum comportamento". +143/−16, 1 arquivo (`hle/d3d12x/list_x.rs`, por cherry-pick de `62de4d55`). | Não é D-20 (que é `ExecuteIndirect` lendo argumentos antes da barreira); é a infraestrutura que permite ordenar espera depois de escrita. |
| #3034 | D3D12.X: snapshot command-list effects and generalize segmentation | Empilhado em #3033. Esperas, sinais de `Write64`, escritas esparsas e escritas de CPU adiadas são capturadas uma vez num plano imutável antes da submissão; motivos de split ganham nome e o estado de continuação anda num objeto de carry. Também "muda nenhum comportamento". +530/−128, 3 arquivos. | Não. |
| #3035 | D3D12.X: a list wait after a deferred CPU write waits for that store | O caso concreto: fora de memória unificada, uma escrita de CPU adiada (`cpu_writes`) acontece depois de a GPU terminar a lista; se um `Wait32/64BitValueX` posterior na **mesma** lista espera por essa célula, a `main` avalia a espera antes de submeter a lista, estoura o timeout de 2 s e a lista roda **sem a escrita**. Com o PR a lista divide nessa espera: submete o segmento produtor, deixa as escritas adiadas pousarem e só então avalia a continuação (`ExecuteCommandLists` e `KickoffX`). +957/−187, 5 arquivos, com teste e2e novo que falha na `main` e passa no head. | **Candidato a defeito novo de catálogo**, não catalogado: nenhum D-xx de XCL descreve essa ordem. É o mais próximo do eixo de D-20 (ordem escrita→leitura dentro da lista), com mecanismo diferente. |
| #3036 | D3D12.X: stencil, topology and scissor sets go into deferred two-pass runs | Com `XCL_TWO_PASS_BATCH=2` (desligado por padrão), `OMSetStencilRef` era o motivo mais frequente de fim de run (470.956 de 1.048.576), à frente de buffer cheio e `IASetPrimitiveTopology`; esses sets passam para dentro do run e os runs vão de 12,7 para 24,3 draws. Base `nvidia-dev`. +158/−15, 2 arquivos. | Não. É desempenho de batching. |
| #3037 | D3D12.X: gate texture uploads on OOM back-off and commit headroom (XCL_UPLOAD_BACKOFF=1) | Porta do `Vice-Tay` `57deba5c` o back-off de OOM: cooldown de 500 ms após `E_OUTOFMEMORY`, amostragem de commit disponível a cada 32 ms, pisos de reserva de 512 MiB (reuso de staging) e 1.024 MiB (novo `CreateCommittedResource`), deferimento por thread e contadores agregados no relatório. Desligado por padrão. Base `nvidia-dev`. +722/−27, 8 arquivos. | Não. É admissão de memória/estabilidade. |
| #3038 | Recompiler: watertight ray/triangle test for BVH triangle nodes, behind XCL_BVH_WATERTIGHT | Troca o Möller-Trumbore do nó de triângulo por um teste watertight (Woop/Benthin/Wald, JCGT 2013) na referência de CPU, no prelúdio HLSL e no lowering DXIL direto: eixo dominante vira z, triângulo cisalhado para o espaço do raio, três funções de borda com sinal — dois triângulos que compartilham aresta dão valores exatamente negados. Traz um caso fixo em que o teste antigo erra os dois triângulos. Desligado por padrão; com a chave ligada, muda a chave do cache DXC e a identidade do cache traduzido. +535/−28, 8 arquivos. | Não é D-24 (ordem `getDimensions`→`textureLoad`); é o buraco em aresta compartilhada. Nenhum D-xx de XCL descreve isso. |

## Pull requests mesclados (1)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #1871 | PSO: xcl pso-precompile builds a title's compute pipelines before its run | `f9a0cb80` | Subcomando `pso-precompile` (aceita pasta do jogo ou `shaders.rpf`), `--records`/`--used` para restringir por hash de record ou pipeline key, `--threads` ligado ao número de compilações do driver (`XCL_PIPELINE_THREADS` ou `min(cores, RAM/2)`, 1..32), teste e2e do subcomando. O PR prometia "aquece o cache de blobs do driver antes da execução"; o diff entrega isso e **não** aplica nada no runtime — só enche caches que o runtime já consulta. |

## Cruzamento com o catálogo

- **Não corrige D-20.** D-20 é `ExecuteIndirect` lendo argumentos indiretos antes da barreira que os
  escreve (NVIDIA). Nada nesta janela toca `ExecuteIndirect`. O que a pilha #3033–#3035 ordena é a
  **espera** (`Wait32/64BitValueX`) em relação a uma escrita de CPU adiada na mesma lista — outro
  consumidor, outra barreira.
- **Não corrige D-21** (`ExecuteIndirect` de compute não traduzido com root signature errada) nem
  **D-11** (chave de cache per-lane sem as fontes do lowering). O #1871 apenas pré-compila; não muda
  chave de cache.
- **Possível defeito catalogável, encontrado no corpo do #3035:** na `main`, uma espera em célula
  escrita por CPU adiada na mesma lista expira em 2 s e a lista roda **sem a escrita**. Não existe
  D-xx para isso; fica registrado como candidato, com a ressalva de que só o corpo do PR o descreve —
  não reproduzi.
- **Abre defeito novo?** #3038, com a chave ligada, muda chaves de cache (DXC e identidade do
  traduzido). Se a tag de identidade não cobrir todos os consumidores, recebe entrada envenenada —
  mesmo formato de D-02/D-11, mas o autor diz ter conferido as chaves lendo o código e afirma que
  nenhum teste exercita a troca pelos caches de disco. **NÃO VERIFICADO**.

## Confiança

- Os dois commits foram lidos por diff completo. `062158ea` tem `commit.author.date` de
  `2026-11-24T21:17:27Z` e pai `81693f02` (o cursor anterior): entrou na `main` agora por merge, o
  que explica a data antiga sem cursor retrocedendo. `f9a0cb80` é merge de dois pais e o diff contra
  o primeiro pai repete o conteúdo do commit de branch.
- "2 commits" não são 2 mudanças: é 1 mudança (`062158ea`) e seu merge (`f9a0cb80`).
- PRs abertos: corpo e contagem de arquivos lidos; **nenhum patch de PR aberto lido**. A divisão de
  base é real e importa: #3033–#3035 têm base `main`, #3036–#3037 têm base `nvidia-dev`, que **não**
  está coberta pelo cursor de commits nem pelo de merges (a rotina só enxerga a `main`) — pendência
  estrutural já registrada no estado.
- `#3033`/`#3034`/`#3035` são empilhados: cada um declara carregar só os N commits do topo (1, 2 e
  3). Se forem mesclados fora de ordem, o `changed_files` de cada um não corresponde ao que entra.
- Testes citados nos corpos (1.453/1.459/1.463 passed) e no commit **não foram executados** aqui.
- Head `f9a0cb80` = cursor novo; `last_run` sobe para `2026-11-27T15:11:50Z`; `last_pr_number` vai
  para 3038.
