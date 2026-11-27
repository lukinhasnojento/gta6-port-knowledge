# fxceless/victory-fork — atualização de 2026-11-27 15:00

## Resumo

Um commit na `main` (`67429060..9a9091d1`), grande e de conteúdo: o construtor de estruturas de ray
tracing deixa de travar o título (sincronizava **todas** as cópias numeradas de memória em **todas** as
listas, e o watchdog de 30 s do próprio jogo matava a execução), e os raios passam a ver as estruturas —
uma view de estrutura não tem recurso, e os shaders recebiam endereço 0 e liam um nó vazio, então todo
raio errava. O custo é grave e o próprio README o registra: com o construtor ligado (o padrão) o jogo cai
para 0,2-3 quadros por segundo. Em PRs, 35 abertos acima do cursor 1427, quase todos de desempenho,
diagnóstico e HLE; nenhum mesclado na janela.

## Commits novos (1)

`commits?sha=main&per_page=100`: `674290601fb9dba441e711cc951a8950ff08ddc6` está na posição 2; o topo é
`9a9091d180f044726463d3db819808086daeebae`.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `9a9091d180` | fxceless | Ray tracing runs: the structure builder no longer stalls the title; rays see the structures | 10 arquivos, +522/-59. **Sincronização de memória do título por endereço** (`crates/victory-gpu/src/memory.rs` +347/-37): cada lista passa a atualizar só as cópias que **os seus próprios** dispatches nomeiam, mais as cópias que shaders alcançam por ponteiro (numeradas por uma nota de fault, válidas em toda lista); `VICTORY_MEMORY_SYNC_ALL=1` restaura o comportamento antigo. O teto de cópias numeradas sobe de `COPIES = 1023` para `4095` (o construtor numerava 1023 nos primeiros 3 s e as páginas seguintes liam zero). Novo medidor com linha a cada 5 s (`Meter`: nomes por origem — view de root, par de constantes, constantes em unidades de 256 B, nota de fault —, cópias numeradas, bytes sincronizados, páginas aprendidas). **Páginas escritas** (`crates/victory-shader/src/emit/lower.rs` +28/-5, `shader.rs` +8/-1, `mod.rs` +1/-1): `memory(lo, hi, guard, store)` recebe `store`, e um store ou atomic marca o dword da página na área depois das notas de fault (`MEMORY_NOTES = MEMORY_FAULTS + MEMORY_PAGES`); só páginas marcadas são declaradas escritas, os refreshes as pulam, e elas são puxadas de volta ao título alguns quadros depois (`PULL_AFTER = 3`), em `shadow.rs` (+66, `Shadow::pull_back`, compara em blocos de 256 bytes e só escreve o que difere). `VICTORY_MEMORY_WHOLE_WRITES=1` volta a declarar as cópias inteiras. **Endereço de estrutura** (`descriptor/host.rs` +23, `draw.rs` +2/-2): novo `view_address` — uma view de estrutura de ray tracing (dimensão 11) não tem recurso e nomeia a estrutura pelo endereço nas palavras 4-5 (`RaytracingAccelerationStructure.Location`); antes `buffer_range` devolvia `None` e o shader recebia 0. **Nós 6 e 7** (`victory-shader/src/bvh.rs` +8/-3, `emit/intersect.rs` +8/-4): instância e nó procedural devolvem `[u32::MAX; 4]` como o hardware (AMD GPURT `IntersectCommon.hlsl`, "User defined node. HW returns -1"), não erro. README +31/-6 atualiza o estado de 03:45Z para 06:30Z e registra a queda para 0,2-3 fps com o construtor ligado. |

## Pull requests abertos (35)

`gh pr list --repo fxceless/victory-fork --state open --limit 50` devolveu 50 abertos; os **35** abaixo
têm número acima do cursor 1427 (o maior é 1468).

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 1428 | GPU: reject device-removed fence values on copy lanes | `GetCompletedValue` devolve `UINT64_MAX` após remoção de device; a lane de cópia podia ler isso como conclusão e reciclar um alocador. Confere o valor antes de reuso, polling e esperas e devolve `DXGI_ERROR_DEVICE_REMOVED` para o sentinela. Autor: gingerninja85. | Não. |
| 1429 | GPU: batch retirement of same-heap descriptor spans | Agrupa spans consecutivos por endereço e identidade de heap e libera cada grupo sob um único lock, em vez de repetir busca no registro e lock por span. | Não. |
| 1430 | GPU tests: verify four queued copies stay distinct behind a fence | Teste: quatro lotes de cópia pendentes atrás de uma fence, identidades e estado conferidos, liberação e comparação destino-fonte; oito rodadas para exercitar reuso de alocador. Só teste. | Não. |
| 1431 | GPU: merge dirty upload ranges in place | Ordena as faixas sujas e grava os intervalos unidos no prefixo do mesmo vetor, sem alocar um segundo. | Não. |
| 1432 | diagnostics: analyse frame logs by draw count | Comando offline de xtask que lê linhas de quadro, filtra por contagem mínima de draws e reporta percentis e estimativa linear por draw. | Não. |
| 1433 | Rect tests: skip validation on a validator older than DXC 1.8 (#1337) | `Validator::version()` por `IDxcVersionInfo`; os dois testes de `emit::rect` pulam a validação quando o `dxil.dll` é anterior a 1.8 (Windows SDK 10.0.22621 traz 1.6). Declara corrigir #1337. | Não (issue #1337, não catálogo). |
| 1434 | Frame pacing record behind VICTORY_PACING (#1347) | `VICTORY_PACING=1` registra por período p50/p95/p99/max do intervalo entre `Present` do título, entre presents do host e da latência até a conclusão na GPU. Declara atender #1347. | Não. |
| 1435 | Allow the compute detile readback test on the runtime GPU | `VICTORY_TEST_GPU_DETILE_HARDWARE=1` exige o device do renderizador em vez de WARP; sem o switch, o teste segue em WARP. Relata 19 de 19 testes de readback passando em RX 7900 XT (RADV 26.3.1). | Não. |
| 1436 | GPU: avoid per-draw Vec allocation in stage_spans | `State::bind` chamava `stage_spans` a cada draw e alocava um `Vec` novo; com raiz limitada a 6 parâmetros, cabe em pilha/`SmallVec`. Autor: DoktorTK. | Não. |
| 1437 | GPU: avoid per-call allocs/locks in merge_under_writes and redundant refresh scans | Reusa buffer `thread_local` do tamanho de página em `merge_under_writes` e remove varreduras de refresh redundantes. Autor: n-romanovich. | Não. |
| 1440 | Shaders: fix workspace clippy lints in wave64_after_loop and valu_tests | Parênteses em `((i / 16) ^ 1) * 16` para `clippy::precedence` e ajustes em `valu_tests`. Declara corrigir #1439. | Não. |
| 1441 | PE: reject TLS callback lists above the supported limit | O parser de TLS parava em 1.024 entradas sem verificar se havia outra; lê uma entrada a mais e devolve `Unsupported` se for não nula. Autor: pwu2775-dev. | Não. |
| 1442 | GPU: keep unrelated pipeline library loads from blocking each other | Serializa cargas de biblioteca só quando o **nome** do pipeline é o mesmo; os 64 slots compartilhados hoje serializam nomes alheios e uma carga em primeiro plano pode esperar segundos. Autor: Filip722. | Não. |
| 1443 | GPU: reuse descriptor table scratch buffers and stack roots in bind | `State::bind` alocava vários `Vec` por estágio/draw (`made`, `entries`, `selected`, `memory_ready`); reusa buffers de rascunho e roots de pilha. Autor: DoktorTK. | Não. |
| 1444 | HLE/files: a title path's ".." stops at the mounted drive's root | A montagem mapeia pasta do host num drive virtual e o `..` do título era resolvido contra a **raiz do volume do host**; passa a parar na raiz do drive montado. Autor: CwWwWwW. | Não. |
| 1445 | Loader/HLE: run the process-detach notices when a title calls ExitProcess | `ExitProcess` estava ligado ao do host, que termina o processo sem o caminho de detach do loader; imagens do título são preparadas sem entry point nem callbacks TLS, então só esse caminho entrega `DLL_PROCESS_DETACH`. | Não. |
| 1446 | Loader: stage each prepared image under its own name, then publish | `write_if_changed` usava um `.tmp` determinístico por destino: dois processos carregando o mesmo título truncavam o staging um do outro. Passa a preparar por nome próprio e publicar por rename. | Não. |
| 1447 | GPU/watch: give a remapped view's pages their write access back | `title_memory_remapped` esquecia os blocos de 64 KiB postos como somente-leitura por um guard e limpava `PUBLISHED` **sem restaurar `PAGE_READWRITE`**, então um refresh que volta a proteger as páginas escrevia em página sem permissão. | Não. |
| 1448 | HLE/net: a blocked name lookup clears the caller's result slot | As quatro rotinas que devolvem lista de resultados por parâmetro saíam com `WSAHOST_NOT_FOUND` **antes** de escrever o slot do chamador, deixando lixo dele. | Não. |
| 1449 | GPU: pad resource groups to their largest member alignment | `place` devolvia o fim do último membro sem alinhar o grupo ao maior alinhamento; uma textura MSAA de 4 MiB seguida de dois buffers de 64 KiB terminava em 4 MiB + 128 KiB em vez de 8 MiB. Autor: pwu2775-dev. | Não. |
| 1450 | GPU: cap the time a frame waits for compute pipelines made at their dispatch (#1420) | Com `VICTORY_PIPELINE_COMPUTE_WAIT` ligado (padrão), o primeiro cutscene criava 457 pipelines de computação num quadro; limita a espera sem desligar o caminho. Declara corrigir o crash em `+0x23cf683` do #1420. | Não. |
| 1453 | Shaders: the legacy multiply and multiply-add as the IEEE operation alone where what is known of the operands makes it exact (VICTORY_LEGACY_PLAIN) | Três commits: multiplicação legada como produto puro onde exato (`emit/bitcode/facts.rs`, `emit/legacy.rs`), o mesmo para `legacy_fmad`, sob a opção. Base `9a9091d1`. Autor: PiloteCode. | Não. |
| 1454 | Diagnostics: isolate scripted GameInput for repeatable benchmarks | `VICTORY_KEYS_ONLY=1`: ignora teclado GameInput e controle XInput físicos, mantendo a entrada roteirizada por `VICTORY_KEYS`. Autor: spk-c. | Não. |
| 1455 | GPU: opt-in exact conventional table CBV upload reuse | `VICTORY_CBV_REUSE_UPLOAD=1` mantém cache local de lista para uploads de CBV de tabela convencional, só com fonte e tamanho exatos; qualquer passo que não seja `Fill` invalida. | Não. |
| 1456 | GPU: opt-in shared indexed texture CPU frame budget | `VICTORY_TEXTURE_FRAME_BUDGET=1` compartilha um orçamento de tempo de CPU entre streaming inicial, passes normais e worker da heap, com `VICTORY_TEXTURE_SLICE_MS` (padrão 64 ms) escalado pelo backlog observado. | Não. |
| 1457 | Diagnostics: split indexed texture queue CPU phases | `VICTORY_TEXTURE_CPU_PROFILE=1` separa manutenção da fila, seleção/contabilidade de vencimento e processamento de slot (com setup pós-seleção); passes de bake fonte apenas ficam de fora. | Não. |
| 1458 | victory-gpu: add diagnostics counters per branch of refresh_timed | Sob `feature = "diagnostics"` em `texture.rs`, contadores por ramo de `Staging::refresh_timed`, extraídos do re-teste do #768 a pedido de KyleS98. | Não. |
| 1460 | HLE/xmem: a remap that cannot serve a page leaves the buffer it showed | `XMemMapPhysicalPages` (`victory-hle/src/modules/xmem.rs:853`) desfaz as views do destino (`unmap`) **antes** de olhar os números de página (`:861`); se a página não pode ser servida, o buffer fica sem as views que mostrava. | Não. |
| 1461 | Pipelines: static samplers are first made in the same order in every launch | Nova `pipeline/samplers.rs` mantém a ordem em que o host vê cada sampler estático igual em toda execução — o quadro dependia de quais launches tinham enchido a biblioteca de pipelines. Declara corrigir #1459. | Não. |
| 1462 | HLE/gameinput: the system button registration gets its own six arguments | Os slots `0x50` (RegisterSystemButtonCallback) e `0x58` (RegisterKeyboardLayoutCallback) da vtable `IInput` apontavam para o mesmo handler de cinco argumentos; `0x50` tem assinatura própria de seis. | Não. |
| 1464 | HLE/gameinput: a pad an all-zero receiver answers for is not a device | Receptor XInput sem fio responde `XInputGetState` com sucesso e estado todo zero com o controle desligado; `Host::pad` e `State::poll` passam a não contar isso como controle presente. | Não. |
| 1465 | HLE/gameinput: a reading-callback registration is refused like the platform | `register_reading` (slot `0x40`) devolvia `S_OK` e um token fabricado sem registrar nada; passa a recusar como a plataforma. | Não. |
| 1466 | GPU memory: a page written every frame is still pulled back (#1438) | `Memory::take_written` colapsava duplicatas de `(copy, page)` na **última** aparição, então uma página escrita em quadros consecutivos nunca satisfazia `frame < seen + PULL_AFTER`. Declara corrigir #1438. | Não no catálogo — corrige #1438, aberto contra o `PULL_AFTER` introduzido por `9a9091d1` nesta mesma hora. |
| 1467 | HLE/xgameui: a password text entry hides what is typed | O chamado de entrada de texto ignorava o escopo (`victory-hle/src/modules/xgameui.rs:60`); `XGameUiTextEntryInputScope::Password` é 31 e o HLE abria um `ES_AUTOHSCROLL` comum, mostrando o que era digitado. | Não. |
| 1468 | GPU: title buffer copies out of memory flights write are done on the card in stream order and not settled for | `CopyBufferRegion` e afins que leem memória que um voo em curso escreve hoje são liquidados (espera o voo, busca as escritas, copia na CPU); passa a copiar na placa em ordem de stream, sob `VICTORY_CARD_READ_COPY=1/2/3`. Autor: brunofgmag. | Não. |

## Pull requests mesclados (0)

`search/issues` com `is:merged merged:>=2026-11-26T23:06:47Z` devolveu `total_count=1`, e o único item
tinha `mergedAt` anterior ao cursor — filtrado. Nenhum merge na janela.

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum. Conferido contra as 17 linhas de `catalogo/defeitos.csv` para `victory-fork`:
  - **D-16** ("Perdas de device sem causa identificada, runs 204-279 e mundo aberto") continua aberto: o
    commit trata do travamento pelo **watchdog do próprio título** (store deliberado no endereço 0 depois
    de 30 s sem quadro apresentado) e dos raios que erravam; o README novo continua dizendo que a placa
    é perdida de 30 a 50 minutos depois, causa desconhecida.
  - **D-11** ("texturas escritas por compute não chegavam à CPU do título") é o inverso do que este
    commit faz: ele **limita** o que é declarado escrito (só páginas marcadas) e puxa de volta 3 quadros
    depois. Não é a mesma correção nem a mesma falha.
  - **D-12** ("loads escalares agrupados de 4 dwords leem além do fim da view — água preta") não é
    tocado por nenhum dos 10 arquivos.
  - **D-17** (dívida de verificação: "this merged tree was built; its test suites were not run") piora:
    o corpo deste commit afirma que as suítes de shader e de memória/descritor passam, mas a árvore é de
    conteúdo novo grande e nada foi executado aqui. Registro como não verificado, não como resolvido.
- **Abre defeito novo?** Sim, um demonstrado **pelo próprio repositório**, e não pelo diff sozinho:
  - **Página escrita em quadros consecutivos nunca volta ao título** — o `PULL_AFTER = 3` desta hora
    combinado com o colapso de duplicatas em `take_written` faz `frame < seen + PULL_AFTER` nunca
    valer. O defeito está descrito e corrigido no PR aberto **#1466** (`fix/1438-pull-back-starvation`,
    aberto contra o issue #1438). Candidato a linha nova com `introduzido` = `9a9091d1`.
  - Candidato de desempenho: com o construtor de BVH ligado (padrão), 0,2-3 quadros por segundo porque
    cada quadro ainda sincroniza cerca de 8 GB de cópias (~150 ms de cópia na CPU; o README registra o
    cartão ocioso 73% de um quadro de 515 ms). O README trata como trabalho pendente, não como defeito
    fechado; registro como candidato com a ressalva de que é medição do autor.
  - Observação que **não** afirmo como relação causal: o PR #1450 fala do mesmo endereço de watchdog
    (`+0x23cf683`) atingido por outro caminho (457 pipelines de computação criados num quadro). Dois
    caminhos para o mesmo watchdog, causas distintas segundo os textos; não verifiquei se o endereço é
    literalmente o mesmo código.

## Confiança

Verificado: o commit em `repos/fxceless/victory-fork/commits/9a9091d180` (pai único `674290601fb9dba441e711cc951a8950ff08ddc6`,
+522/-59, 10 arquivos, patches lidos até 6.000 caracteres por arquivo nos maiores); a lista de commits
com o cursor na posição 2; os 35 PRs abertos acima de 1427 por `gh pr list --state open --limit 50`
(50 no total, maior 1468) com os corpos lidos em até 300 caracteres cada; `search/issues` de mesclados
(`total_count=1`, filtrado por `mergedAt`).

Não verificado: nada foi compilado nem executado. As medições do corpo e do README (0,2-3 fps, 8 GB de
cópias por quadro, 150 ms de CPU, cartão ocioso 73%, runs 371-375) são dos autores. Os patches de
`memory.rs` (+347) e `lower.rs` foram lidos truncados, então partes do caminho de sincronização não
foram conferidas linha a linha. A relação entre #1466 e o `PULL_AFTER` desta hora é inferida do texto do
PR (#1438), não de reprodução.
