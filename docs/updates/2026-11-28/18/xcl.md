# ivall/xcl — atualização de 2026-11-28 18:00

## Resumo

A hora é inteira de andaime de pipeline e de estado esparso no `D3D12.X`: 34 dos 40 commits documentados
preparam retenção de identidade exata de PSO (`bind.rs`, `pso_async.rs`), observação de pressão,
conta de submissão e planejamento esparso fora do lock — quase todos declarados como preparação, com o
caminho antigo ainda autoritativo. O que muda comportamento de verdade são quatro commits e um PR: a
espera por célula de valor deixa de assumir conclusão sem prova (`dd84e0ad`, `c7752525`, `40d3a312`) e o
recompilador deixa de substituir por zero um rendezvous wave64 que não chegou (`86a90eeb`). Zero merges
novos; 10 PRs abertos acima do cursor.

## Commits novos (40 de 78 sem documento)

Teto de 40 atingido. Ordem do `compare` a partir de `4701a874`, pulando `2dad32e6` e `abf7e177`
(documentados às 12:00). `4577f09c`, `b0b041fd`, `d85f2c6c` e `f2c0d36f` são merges cujo diff repete o
commit de conteúdo citado na própria linha.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `529fb19c` | bmjask | D3D12.X: keep restoration cleanup off list destruction | Tira a reconciliação do coordenador global de restauração do `Drop` do último bind retido: `pso_async.rs` +66/−8 passa a reconciliar na demanda exata (antes de consumir capacidade) e na emissão (depois de esperar), para não pôr varredura global nem destruição COM na thread que grava a lista |
| `96ef96cc` | bmjask | D3D12.X: report queue device cache reuse | Classifica cada `before_execute` como acerto de cache de dono de device ou `GetDevice` real, com o gate serial da fila tomado; agrega os dois contadores na linha `XCL_ACTIVITY` de VRAM (`sparse.rs` +24, `vram.rs` +13) para medir a eliminação da consulta no caminho de submissão |
| `3f54312c` | bmjask | D3D12.X: shadow exact pipeline eviction transactions | Exercita a transação de evicção em forma de produção sob observação de pressão: retém a geração residente exata, serializa a admissão de restauração, instala o token do ledger, revalida identidade/posse/residência/timestamps e aborta sem desanexar o pipeline nativo (+352 em `pso_async.rs`) |
| `c1c963b7` | bmjask | D3D12.X: attribute pipeline restoration stalls | Expõe demanda exata de restauração, espera bloqueante e reconciliação do coordenador em `XCL_STALL_TRACE` (`stall.rs`, `report.rs`); a espera continua aninhada no total `pipeline_wait` e consumidores já terminais não atribuem trabalho falso |
| `c50fcdfa` | bmjask | D3D12.X: make exact eviction detachment atomic | Torna o desanexo atômico: reivindica os escritores de timestamp, revalida identidade e idade sob os locks de estado e transfere todos os donos nativos como uma transação, para não expor pipeline parcialmente desanexado (+318/−42) |
| `a6dfe2e5` | bmjask | D3D12.X: separate pipeline pressure budgets | Separa o orçamento de admissão de pressão do de evicção: antes a admissão gastava toda a cota antes de qualquer desanexo exato, e candidatos ocupados/velhos/alterados escondiam vítimas úteis |
| `57b766ca` | bmjask | D3D12.X: restrict pipeline eviction to compute | Restringe a evicção a compute: gráficos e reprodução de lista do console ainda têm consumidores de pipeline por endereço, então permitir os builds deles numa evicção futura seria semanticamente inseguro; o registro de candidato passa a carregar o tipo de pipeline |
| `52436c9a` | bmjask | D3D12.X: keep selected pipeline metadata exact | Corrige caminhos que fixavam um pipeline nativo e depois passavam o ponteiro por helpers que o resolviam de novo como endereço visível do título — reuso de endereço podia redirecionar forma de thread, ABI, remap, root signature ou decisão de dois passes (`bind.rs` +34, `real.rs` +34) |
| `5b03933e` | bmjask | D3D12.X: snapshot indirect signature traits | `ExecuteIndirect` classificava a assinatura por três mapas de endereço com locks independentes; recriação concorrente expunha geração mista e assinatura comum reusada em endereço que muda raiz herdava o bit de root velho (+105/−45) |
| `9c0824f2` | bmjask | D3D12.X: retain indirect pipeline wait policy | Preserva a política de espera da resolução retida de `ExecuteIndirect`: mantém o fallback limitado de build inicial sem deixar identidade evictada ou liberada cair num endereço de título reutilizado (+243) |
| `43391a13` | bmjask | D3D12.X: retain indirect pipeline identity | `ExecuteIndirect` resolvia o endereço do título de novo na emissão e na reprodução; agora carrega a identidade do pipeline retido, para reuso de endereço ou evicção não redirecionar trabalho crítico (`bind.rs` +279) |
| `fcbae0ce` | bmjask | D3D12.X: bind selected pipelines without re-resolution | Tira do hook público `SetPipelineState` os PSOs de ABI, cadeia de dispatch, remap e helper indireto: eles eram repassados depois da seleção e podiam ser redirecionados (`bind.rs` +85, `recomp/pso.rs`, `recomp/runner.rs`) |
| `3f91865b` | bmjask | D3D12.X: retain native pipeline bind choices | Captura a escolha de bind nativo da mesma identidade retida que o estado da lista usa, preservando as políticas de espera lazy e de compute `ForBind`, com restauração desanexada falhando fechado (+305) |
| `e5561505` | bmjask | D3D12.X: bind retained pipeline choices | Usa um único snapshot retido para `SetPipelineState` e `ClearState` do título: seleção nativa, posse de geração e estado lógico não podem divergir depois de reuso de endereço; binds exatos pulam os hooks resolvedores e identidade indisponível falha fechado (`bind.rs` +210, `list_x.rs` +50) |
| `db4dd270` | bmjask | D3D12.X: retain initial pipeline choices | Prepara `CreateCommandList` e `Reset` a partir de um snapshot retido, para PSO inicial nativo, dono de geração exato e estado lógico de bind não divergirem (`bind.rs` +158) |
| `01f85586` | bmjask | D3D12.X: track retained pipeline use | Publica um uso sincronizado para binds nativos, resets e emissões críticas, fechando a lacuna entre journal de candidatos e idade antes de a evicção por pressão poder escolher pipeline em uso |
| `5ec65338` | bmjask | D3D12.X: evict old compute pipelines under pressure | Adiciona política de pressão opt-in que desanexa no máximo um pipeline de compute velho e reconstruível a cada amostra de dois segundos, preservando restauração por demanda exata; amostragem só de pressão evita ligar sondas de atividade não relacionadas (+290) |
| `775520be` | bmjask | Tests: benchmark retained pipeline tracking cost | Compara, no `benches/runtime.rs`, resolução `Ready` sem receita contra o caminho quente com receita usado pela evicção, com o journal de candidatos e a sincronização de timestamp reais |
| `1cb6e4c5` | bmjask | Tests: restore observer-evicted pipelines by demand | Exercita o observador de pressão através da restauração exata depois de reuso de endereço do título: prova consumo de candidato, aposentadoria do dono nativo, troca de geração e reconciliação do ledger numa só passada |
| `aeba0477` | bmjask | D3D12.X: report pipeline pressure lifecycle | Separa em `report.json` quatro desfechos — desanexo de publicação, aposentadoria de dono de política, `Release` do dono final de geração e restauração exata — e admite no corpo que os contadores são de caminho raro, aproximados, e que o `Release` não prova devolução de memória do driver |
| `df7b9fd5` | bmjask | D3D12.X: attach sparse coordination to live state | Amarra fronteira de lote, ledger de espera em primeiro plano e reservas do anel de upload a uma geração de estado esparso e a uma identidade de fence; é preparação — o caminho de submissão legado continua autoritativo |
| `dd84e0ad` | bmjask | D3D12.X: fail closed on unproven wait timeouts | **Muda comportamento:** tempo de parede decorrido não prova que produtor de CPU ou GPU terminou. O título é parado antes de trabalho de GPU posterior em vez de sintetizar conclusão de fence ou seguir depois de espera de fallback de CPU não resolvida; quebras de ciclo provadas pelo grafo ficam como estavam (`list_x.rs` +134/−44) |
| `8d6704d9` | bmjask | Tests: emit machine-readable benchmark results | O harness próprio só expunha distribuições como texto formatado; passa a emitir registro JSON versionado sob pedido, preservando a saída padrão |
| `c7752525` | bmjask | D3D12.X: fail closed on unreadable wait cells | **Muda comportamento:** célula `Wait32BitValueX`/`Wait64BitValueX` não mapeada era tratada como já satisfeita, deixando trabalho de GPU posterior rodar sem evidência de que a dependência concluiu; observação ilegível ou inválida passa a ser estado explícito de não-conclusão e alcança o caminho que falha fechado |
| `97e80fbf` | bmjask | D3D12.X: mirror legacy sparse upload work | Espelha cada tentativa legada (zero, retry, dirty) num lote de geração exata e aposenta o lote como obsoleto depois de publicado o journal de retry legado — costura de paridade, sem mudar quem submete |
| `520f3f1c` | bmjask | D3D12.X: reconcile rebased pipeline cache identities | Preserva nomes de root signature e de pipeline library por nó depois do rebase sobre a implementação atual de cache, e atualiza fixtures para o campo de callback de primeiro uso |
| `f0384e0a` | bmjask | Shaders: separate legacy and OS-backed build claims | `main` e a pilha introduziram cada uma um tipo de claim de build de pipeline; mantém os dois protocolos distintos depois do rebase (`shaders.rs` +13/−13) |
| `23c06c9f` | bmjask | Tests: compare benchmark regression thresholds | Cria `bench_compare.rs`: compara medianas observadas contra limite dado e recusa conjuntos de benchmark incompatíveis ou metadados de máquina opcionais divergentes — portão determinístico para baseline salvo |
| `40d3a312` | bmjask | D3D12.X: read wait cells without faulting | **Muda comportamento:** a sonda tipo `VirtualQuery` podia ficar velha antes do dereference atômico se o título descometesse a página; passa a fotografar via `ReadProcessMemory`, deixa o Windows conter o fault e aplica ordenação acquire antes de trabalho posterior observar conclusão. Teste cobre estado publicado por release e página descometida |
| `51cda2f4` | bmjask | D3D12.X: own sparse planning observations | Congela mapeamento, hash enviado e hash de origem numa observação imutável, para a construção do plano não precisar do `State` esparso vivo; geração exata e época de mapeamento continuam obrigatórias na revalidação de commit |
| `687d553c` | bmjask | D3D12.X: keep unlocked sparse planning inert | Máquina de snapshot, validação, reserva e retry fica inerte: prepara o planejamento de upload esparso sem alterar o uploader vivo (+190) |
| `2f49ef5a` | bmjask | Tests: identify benchmark result environments | Grava identidade estável de build e de CPU em cada resultado NDJSON, com override explícito para CI reproduzível, para o comparador recusar medições de ambientes incomparáveis (`build.rs` +13) |
| `86a90eeb` | bmjask | Shaders: fail closed on wave64 rendezvous timeout | **Muda comportamento:** o rendezvous limitado entre metades substituía por zero e deixava o shader seguir quando o parceiro não chegava, podendo publicar estado de registro ou memória meio-local como onda de console bem-sucedida; agora falha fechado (`dxil/lanes.rs` +60, `recomp/emit.rs` +45, `dxil/robustness_tests.rs` +10) |
| `be0c274d` | bmjask | D3D12.X: correlate submissions with GPU completion | Duração de `ExecuteCommandLists` na CPU não mostra quando o trabalho submetido completa; retém recibos de fence de geração de fila, opt-in, para um estágio posterior de timestamp query associar dados de query ao bloco submetido que os produziu (`real.rs` +215) |
| `651867a4` | pr1sm-dev | xcl run: preparation workers count the commit left in MiB, not bytes | **Corrige unidade:** `prepare_workers` passava `host::system_commit()` (bytes) a `workers_for(cores, free_mb)`, que divide por 3072 como MiB — o teto de commit nunca valia e a contagem era sempre `núcleos − 4` (até 32); com 6 GiB de commit livre davam 12 workers em vez de 2. Reportado por pickles976 em #2624 |
| `fdbac840` | John Howard | Shaders: share verified compute blobs across preparation worker counts | Guarda os blobs validados sob a identidade nativa de preparação: importa fatias de workers anteriores, funde atualizações concorrentes sob lock de arquivo e faz checkpoint de validações durante preparação longa (`shaders/warm_blobs.rs` +125/−29) |
| `4577f09c` | Prime1997 | Merge pull request #3085 from Royalflamejlh/fix/prepare-worker-memory-units | Merge de `651867a4`; diff contra o primeiro pai é o mesmo +18/−1 em `shaders.rs` |
| `b0b041fd` | Prime1997 | Merge pull request #2959 from bmjask/stack/308-benchmark-thresholds | Merge do stack de benchmark: 48.456 adições, quase todas `benches/` e fixtures; o conteúdo substantivo é `23c06c9f` |
| `d85f2c6c` | Prime1997 | Merge pull request #2960 from bmjask/stack/309-fault-contained-wait-read | Merge de `40d3a312` (+79/−4 em `list_x.rs`) |
| `f2c0d36f` | Prime1997 | Merge pull request #2961 from bmjask/stack/310-sparse-planning-snapshot | Merge de `51cda2f4` (+77/−3 em `sparse.rs`) |

## Pull requests abertos (10)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #3198 | Shaders: report bounded preparation timing | Telemetria do tempo da preparação limitada de histórico (continuação de #3195) | Não |
| #3197 | Shaders: prepare exact graphics history at launch | Prepara a história gráfica exata no arranque do título | Não |
| #3196 | D3D12.X: identify exact graphics recipes in manifests | Identifica receitas gráficas exatas nos manifestos, para o preparo bater a chave | Não |
| #3195 | Shaders: bound history-backed launch preparation | Limita a preparação de arranque apoiada em histórico (memória/tempo) | Não |
| #3194 | D3D12.X: release large texture upload buffers after idle | Libera buffers grandes de upload de textura depois de ociosos | Não (controle de memória; área de `XCL D-20` só por proximidade) |
| #3193 | Tests: verify the finest timer request without timing Sleep | Verifica a requisição de timer mais fina sem medir `Sleep` — tira o teste de relógio do caminho dependente de agendador | Não |
| #3192 | D3D12.X: a superseded target the title binds, clears or moves is mapped again | Remapeia alvo substituído que o título ainda liga, limpa ou move | Não; é o mesmo sintoma de "escrita em alvo velho" que a série de aposentadoria de 17:00 tentou fechar |
| #3191 | README: show --pso-manifest in the --through-present bullet | Documentação do passo offline | Não |
| #3190 | Tests: alloc_pool checks for STATE held by the caller, not by a parallel test | Corrige o teste que dependia de `STATE` de outro teste paralelo | Não (teste) |
| #3189 | Recompiler: 2-D and 3-D compute groups take the direct DXIL route again | Devolve grupos de compute 2-D/3-D ao tradutor direto; o desvio para HLSL era contorno de recusa | Sim por área: `XCL D-24`/`D-02` são recusas e crashes do `amdxc64` no caminho direto |

## Pull requests mesclados (0)

Nenhum merge novo: o `compare` a partir de `4701a874` não traz merge posterior aos três de 17:00
(`db3921dc`, `59e4be57`, `10d27297`), e a consulta de pulls fechados devolveu zero `merged_at` maior que
`2026-11-28T16:59:49Z`.

## Cruzamento com o catálogo

- **Corrige D-xx?** Por hash, nenhum. Os 80 hashes da faixa e os 5 merges da hora foram conferidos
  contra os 270 tokens de `catalogo/defeitos.csv`: zero interseção.
- **Por área (hipótese, não correção):** `40d3a312` troca a sonda `VirtualQuery` por
  `ReadProcessMemory` — mesma classe de custo/corrida da caminhada de `VirtualQuery` de
  `VItoriaBr D-01`, mas outro repositório e outro defeito; `86a90eeb` falha fechado no rendezvous wave64,
  vizinho de `XCL D-16` (máscara wave64 obsoleta) sem ser a mesma coisa; `c7752525` e `dd84e0ad`
  aproximam-se de `XCL D-20` (trabalho dependente rodando antes da evidência), também sem hash.
- **Abre defeito novo?** Nenhum identificado nos 40. `fdbac840` escreve em cache sob lock de arquivo e
  faz checkpoint de validação parcial: se a validação for retomada de um checkpoint, um blob cortado
  poderia ser aceito como verificado — **NAO VERIFICADO**, o diff lido não mostra o critério de
  invalidação do checkpoint (`shaders/warm_blobs.rs`), só a fusão sob lock.

## Confiança

Verificado pelo diff (mensagem + lista de arquivos com contagem de linhas + corpo do commit, via
`repos/ivall/xcl/commits/<sha>`): os 40 hashes, autores, assuntos e o que cada um toca. Para os commits
de `bmjask` o corpo descreve a intenção e a lista de arquivos confirma onde ela cai; não abri o conteúdo
integral de `pso_async.rs`/`bind.rs`, então afirmações sobre *como* a transação é serializada
(quais locks, qual ordem) vêm do corpo do commit, não de leitura linha a linha. **NAO VERIFICADO:** que
os testes citados (`775520be`, `1cb6e4c5`) passam; que a política de pressão `5ec65338` está realmente
desligada por padrão (o corpo diz opt-in, o diff de `host.rs` +29/−14 não foi lido); e a crítica de
`fdbac840` acima. Contagem de merges x commits: 0 merges, 0 commits de conteúdo novos; a faixa de 80
inclui 4 merges cujo conteúdo já aparece como commit próprio, então os commits únicos são 76 e os sem
documento, 74 (não 78). O cursor novo é `f2c0d36f` e restam 38 commits na fila pela contagem do
`compare`.
