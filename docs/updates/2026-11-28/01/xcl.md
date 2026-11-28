# ivall/xcl — atualização de 2026-11-28 01:00

## Resumo

Dois commits acima do cursor `6089122a`: o commit de trabalho `d8dae08d` e o merge real `2c717a04` que o
traz. É uma mudança só, de 30 linhas em `crates/xcl/src/shaders.rs`: a passada offline de pipelines
deixa de manter **todos** os `PipelineJob` vivos até a última compilação de driver terminar. Não há PR
mesclado além do `#3043` que carrega esse commit, e 25 PRs abertos acima do cursor — a maioria é a pilha
de bibliotecas nativas de pipeline (`#3045`–`#3057`), que ainda está empilhada, não mesclada.

## Commits novos (2)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `d8dae08d7f` | gefydev | Shaders: release offline DXIL after each pipeline build | `make_pipelines` troca o `AtomicUsize` + `jobs.get(index)` (que mantinha o `Vec<PipelineJob>` inteiro vivo durante toda a passada) por uma função nova `run_pipeline_jobs<T: Send>(jobs, workers, build)`: os jobs vão para um `Mutex<IntoIter<T>>`, cada worker tira o próximo **por valor** (`jobs.lock().unwrap().next()`) e o `build(&job)` roda fora do lock, de modo que o `PipelineJob` — e o DXIL/pedaços que ele carrega — é liberado quando a compilação daquele job retorna. `workers` passa a ser `pipelines.most.min(jobs.len())`, o que também evita `spawn` de thread à toa quando a lista é curta. Teste novo `offline_pipeline_jobs_drop_their_inputs_after_each_build`: três `Arc` viram `Weak` e, a cada build, o teste exige que os anteriores já estejam derrubados, os seguintes ainda vivos e a ordem de prioridade preservada |
| `2c717a043c` | Prime1997 | Merge pull request #3043 from gefydev/release-completed-offline-dxil | Merge real de dois pais (não squash): traz `d8dae08d` para a `main`. O conteúdo é idêntico ao commit acima — o `compare` lista os dois porque o merge não foi achatado |

## Pull requests abertos (25)

Todos acima do cursor `last_pr_number` 3042, `created_at` entre 17:52Z e 21:43Z no relógio do GitHub;
título e corpo lidos, patch não lido. Vários são declaradamente empilhados ("stacked on"), e por isso o
corpo de um descreve o commit de topo, não o PR inteiro.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| `#3075` | kernelx: check section page liveness for DirectStorage | Empilhado em `#3074`/`#2307`; relê listas de páginas de destino elegíveis quando o request roda, depois de habilitar o alias físico | Não; área de memória/DirectStorage |
| `#3074` | kernelx: preserve null scatter unmaps with physical aliases | Preserva unmaps nulos de scatter com aliases físicos; pede revisão do commit final contra o pai de integração limpo | Não |
| `#3072` | README: document xcl pso-precompile | Só documentação: exemplo de uso de `xcl pso-precompile`, que o HELP lista e o README omitia | Não |
| `#3071` | XAudio2: reset the Opus decoder at a queued Discontinuity | `IXAudio2SourceVoice::Discontinuity` passa a limpar o decoder Opus contado por revisão, para o pré-skip reiniciar no próximo buffer; guarda de corrida para decoder que já falhou/avançou. Fixes `#2942`; revive `#2904`, retirado sem revisão | Não |
| `#3068` | UMA: cap the coherent window at RAM/8 below 24 GB of RAM | Em máquinas com menos de 24 GB, a janela coerente (fixada em RAM do sistema) cai para 1/8 da RAM física, mínimo 1 GiB; `XCL_COHERENT_MB` continua vencendo e `XCL_COHERENT_AUTO_CAP=0` desliga | Não; vizinho de D-01 (janela UMA escolhida pelo dword alto), mas o defeito é outro |
| `#3066` | D3D12.X: build masked pipelines for partial DispatchX groups on workers | `DispatchX` com dimensão parcial seleciona o pipeline mascarado de compute direto; grupos inteiros ficam no pipeline original. Medição citada: 2.999 de 3.009 chamadas parciais numa amostra | Não |
| `#3065` | README: document xcl rpf --extract matching and defaults | Só documentação: `--extract NAME` casa por substring, saída padrão `rpf_out`, arquivos acima de 1 GB exigem `--full` | Não |
| `#3063` | D3D12.X: a sparse readback its wait gave up on is not copied into title memory | Se o `wait` de 5 s desiste, o buffer ainda tem o readback anterior, de outros endereços; o commit de topo deixa de copiá-lo para a memória do título | Não |
| `#3062` | D3D12.X: a sparse list still in flight after its bounded wait is held, not reset | Depois do timeout de 5 s, `begin_slot`/readback deixam de `Reset` no allocator/lista que a GPU pode ainda estar executando (D3D12 proíbe); a lista fica retida | Não; área de fence/reset |
| `#3061` | D3D12.X: coherent UMA windows in a driver heap on NVIDIA (port of #2844 to amd-dev) | Porta o backend UMA de `#2844` para `amd-dev`, para `XCL_UMA_DRIVER` funcionar lá; default inalterado (ligado em NVIDIA fora do Wine) | Não |
| `#3060` | recomp: add XCL_HLSL_CS to force compute shaders onto HLSL path | `XCL_HLSL_CS=<hash>,...` mantém compute específico no caminho HLSL mesmo com `XCL_DIRECT_DXIL`/reroteamento `enormous`, com entradas de cache separadas | Não |
| `#3059` | added logging and tracing for precompilation | Só logging no estágio de pré-compilação | Não |
| `#3057` | Shaders: skip recipe walks for warm libraries | Separa metadados de corpus de descoberta completa e adia a varredura O(N) até um *miss* do recibo; empilhado em `#3055`, `#3054`, `#3053`, `#3051`, `#3046`, `#3045` | Não |
| `#3056` | D3D12.X: two-pass indirect commands with groups D3D12 forbids run nothing | Sob `XCL_TWO_PASS_BATCH=2`, comando cujos grupos o D3D12 não permite para `DispatchMesh` (>65.535 numa dimensão, ou >2^22 no total) deixa de rodar grupos clampados sobre registros que ninguém escreveu | Não; área vizinha de D-20 (`ExecuteIndirect` antes da barreira), mas o defeito é o clamp de grupos |
| `#3055` | Tests: measure distinct pipeline library scaling | Fixture sintética com 32 programas distintos para medir serialização e reuso preguiçoso sem dados do título | Não |
| `#3054` | Shaders: load prepared pipelines at runtime | Ativa as bibliotecas nativas validadas de `#3053` no device de runtime e carrega os pipelines preparados nos mesmos pontos que chamam o compilador do driver | Não |
| `#3053` | Shaders: load prepared pipelines at runtime | **Corpo idêntico ao de `#3054`**, inclusive a frase "as bibliotecas validadas de `#3053` devem ser usadas pelo device de runtime". O título não distingue os dois; a proposta de `#3053` não é legível pelo corpo | Não |
| `#3052` | D3D12.X: two-pass mesh stages never bind a memory window at GPU VA 0 | `uma_window_va(k).unwrap_or(first)` devolve `Some(0)` para janela sem buffer e a janela é vinculada como view nula em u2–u5; passa a não vincular VA 0 | Não |
| `#3051` | Shaders: collect prepared PSOs into native libraries | Coleta os PSOs nativos já criados na preparação offline em bibliotecas D3D12, com nomes determinísticos, deduplicação e substituição atômica; bibliotecas separadas para graphics e compute | Não |
| `#3050` | D3D12.X: XCL_UPLOAD_KEEP_MB sets the idle texture upload buffers kept | `XCL_UPLOAD_KEEP_MB` (0–4096, 512 quando ausente) define quantos bytes de buffers de upload ociosos ficam para os próximos lotes | Não |
| `#3049` | Memory: XCL_HEAP_COUNT=1 counts XCL's own heap by size and by place | Com `XCL_ACTIVITY`, loga a cada 20 s quanto do commit do processo é o heap do próprio XCL, por tamanho de alocação e por local | Não |
| `#3048` | VFS: XCL_DRIVER_SHADER_CACHE=0 refuses the driver its shader cache file | Faz as aberturas do driver do próprio arquivo de cache de shader falharem com acesso negado, para rodar sem cache; desligado por padrão | Não |
| `#3047` | D3D12.X: a split compute program keeps its console code, not its whole DXIL | Um programa de compute dividido só constrói a forma inteira quando um dispatch indireto precisa dela; deixa de manter o DXIL do programa inteiro por corrente | Não |
| `#3046` | Shaders: publish semantic corpus identities | Identidade SHA-256 semântica do corpus, publicada atomicamente com o corpus preparado, mais identidade de layout de PSO que exclui o build do executável | Não |
| `#3045` | Shaders: prove native pipeline library round trips | Sonda as bibliotecas de pipeline do D3D12 antes de integrá-las à preparação offline; dono dos bytes serializados durante a vida da biblioteca, biblioteca vazia com blob nulo | Não |

## Pull requests mesclados (1)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| `#3043` | Shaders: release offline DXIL after each pipeline build | `2c717a04` (21:27:54Z) | `d8dae08d`: consumo por valor dos jobs no `run_pipeline_jobs`, liberação do DXIL a cada build e o teste de `Weak` que prova a queda dos inputs. Merge real de dois pais, não squash |

## Cruzamento com o catálogo

- **Nenhum dos dois commits corrige ou reaplica um D-xx.** Nenhum dos hashes aparece em `introduzido` ou
  `corrigido` no `catalogo/defeitos.csv`.
- **Área vizinha, defeito diferente: D-11.** D-11 ("entradas de cache do per-lane não incluíam as fontes
  do lowering: DXIL obsoleto servido do disco", corrigido em `bfb8133`) também vive no caminho de cache
  de shader, mas é chave de cache; `d8dae08d` é **tempo de vida de memória** dos jobs offline. O diff não
  toca em chave, hash nem `stale()`.
- **Risco introduzido pelo diff, não defeito de catálogo.** O novo `run_pipeline_jobs` exige
  `impl Fn(&T) + Sync` e um `Mutex` compartilhado: a serialização do `next()` é curta, mas se `build`
  entrar em pânico dentro de uma thread, o `Mutex` fica envenenado e os outros workers falham no
  `unwrap()` seguinte. O teste cobre a ordem e a queda dos inputs, não o caminho de pânico. Não há
  tratamento de `PoisonError` no diff.
- **Nada novo para o catálogo do lado de `xcl` nesta hora**: a janela foi pequena e não há correção de
  comportamento observável.

## Confiança

- **Verificado no diff:** os dois commits foram lidos por inteiro (30 linhas de produção e o teste,
  arquivo único `crates/xcl/src/shaders.rs`). O merge `2c717a04` foi conferido por API como merge real
  (`merge_commit_sha` igual ao commit e pai `d8dae08d` na faixa).
- **Verificado por API:** o `merged_at` do `#3043` (21:27:54Z), a ausência de qualquer outro merge na
  janela e os 25 PRs abertos (título e corpo). A duplicação de corpo entre `#3053` e `#3054` foi lida
  nos dois corpos, não inferida.
- **Não verificado:** nenhum teste executado. Não medi consumo de memória antes/depois — a alegação de
  "libera o DXIL" é a do teste com `Weak`, não uma medição de RSS. Não li o patch de nenhum PR aberto;
  em particular, o empilhamento `#3045`→`#3057` foi tomado do texto dos corpos, e `#3053`/`#3054`
  podem ser duplicatas acidentais ou uma divisão mal descrita — os corpos idênticos não permitem dizer
  qual.
- **Contagem:** 2 commits, 1 merge — bate, porque o merge não é squash. Nenhuma divergência.
