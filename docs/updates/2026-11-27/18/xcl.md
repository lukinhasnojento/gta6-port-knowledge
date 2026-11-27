# ivall/xcl — atualização de 2026-11-27 18:00

## Resumo

A `main` andou dois commits desde `06f90c19`: o commit de branch `b8005022` (perfil de teclado e mouse
por título) e o merge `81693f02` do PR #1824. O commit de conteúdo é datado de 24/11 e só apareceu no
cursor agora porque entrou na `main` por este merge — não é o cursor voltando no tempo. Dez PRs
abertos acima de #3022, três deles com base `amd-dev` (que não entra no cursor de commits); o mais
relevante para o catálogo é #3023, que corrige `V_CMP_LG_F32` no tradutor direto RDNA4 — a mesma
classe do defeito `VIctory D-20` (`v_cmp_lg` baixado como `fcmp one`).

## Commits novos (2)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `b8005022c3` | gefydev | Input: load keyboard and mouse bindings per title | `crates/xcl/src/hle/kbm.rs` +327/−34 tira a tabela de bindings embutida do código e a transforma em `InputProfile` editável por título, com overrides, remoção de binding (lista vazia), troca de saída do wheel e `Default` preservando o mapeamento antigo. `crates/xcl/src/config.rs` +4 acrescenta `pub(crate) input_profile` a `Config`, com `#[serde(skip_serializing_if = "InputProfile::is_default")]`. `crates/xcl/src/hle/gameinput.rs` +1/−3 apaga a tabela fixa de dez teclas de navegação (`GetAsyncKeyState` com `0x0D`, `0x20`, `0x1B`, `0x08`, setas, `0x09`, `0x50`) e passa a chamar `kbm::navigation_buttons(w)`, mantendo a regra de foco. `README.md` +29 documenta `[input_profile]`, os nomes de saída e de entrada e os `physical_*`. |
| `81693f0207` | Prime1997 | Merge pull request #1824 from gefydev/input-profiles | Merge `--no-ff` do PR #1824 sobre `b8005022`; mesmo diff (4 arquivos, +361/−37). |

## Pull requests abertos (10)

Janela: números acima de 3022. Base `main`, exceto onde indicado.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #3023 | Recompiler: fix RDNA4 direct-DXIL f32 ordered compare (V_CMP_LG_F32) and V_DIV_*_F32 NaN/scaling semantics (#2952) | Base `amd-dev`. Baixa `lg` como `(a < b) \|\| (a > b)` (`fcmp olt \|\| fcmp ogt`) e `nlg` como a negação, em vez de `fcmp one`/`fcmp ueq`; em `V_DIV_SCALE_F32` emite guarda de QNaN (`0x7FC0_0000`) e pré-escala de expoente ±64 quando o denominador é minúsculo (`ed < 23`, `en < 191`) ou enorme (`ed ≥ 232`, `en ≥ 65`); `V_DIV_FMAS_F32` ganha pré-arredondamento 2^(±64). +174/−20 em 2 arquivos (`gpu/dxil/lower_valu.rs`, `prelude.rs`). | **Sim, por classe**: `VIctory D-20` (`v_cmp_lg` baixado como `fcmp one` — verdadeiro para NaN no driver AMD) e `XCL D-14` (`v_cmpx` float via bridge descartava o EXEC anterior). Nenhum hash do catálogo. |
| #3024 | D3D12.X: coalesce sparse UpdateTileMappings fence signals until execute/wait (companion to #3010) | Base `amd-dev`. Remove o `self.signal()` imediato de `State::ensure_mapped`; `flush_pending_mappings()` emite um único `queue.Signal` cobrindo todos os mapeamentos enfileirados antes do próximo submit, com escape `XCL_SPARSE_SIGNAL_EACH=1`. O autor mede ~41 mil sinais de fence por run caindo para 700–1.000 no título retail `7952987A`. +59/−8 em `hle/d3d12x/sparse.rs`. | Área de **XCL D-20** (`ExecuteIndirect` lia argumentos antes da barreira que os escreve) — é sincronização de fila, não a mesma barreira. |
| #3025 | D3D12.X: gate vram_small block recycling on GPU fences, evict idle heaps under VRAM pressure, and pack small Texture1D | Base `amd-dev`. Blocos liberados passam a capturar `alloc_pool::pending_gpu_marks()` (marcas de GPU) em vez de esperar só o `COOL = 2s` de relógio — o autor descreve reuso prematuro quando uma compilação de PSO de vários segundos segura trabalho da fila além do cool, causando aliasing/UAF; adiciona evicção de heaps ociosos sob pressão de VRAM e subalocação de 4 KiB para `Texture1D` pequeno. +289/−63 em `alloc_pool.rs` e `vram_small.rs`. | Área de uso-após-liberar (o catálogo tem `Belobug-AMD D-07` nessa classe, outro repositório). Sem hash do catálogo XCL. |
| #3026 | D3D12.X: XCL_RT_LEAVE_OUT leaves out named compute programs while rays cannot hit | `XCL_RT_LEAVE_OUT=<fragmento>[,…]`: programa de compute cujo container contenha o fragmento recebe o programa dummy que um shader não traduzível já recebe, então seus dispatches são pulados; a preparação e `xcl shaders --precompile` o ignoram. Off por padrão, parte de #3011 (C3), port de trabalho de Vice-Tay. +224/−2 em 7 arquivos. | Não há D-xx de ray tracing leave-out. |
| #3027 | xcl run: dump the memory around an unhandled exception's pointers (XCL_FAULT_DUMP) | `XCL_FAULT_DUMP=1`: em exceção não tratada, registra a memória ao redor do endereço faltoso e de cada registrador que aponta para memória legível, depois da linha `unhandled exception`. Off por padrão; parte de #505. +300 em 3 arquivos (`eh.rs`, `eh/fault_dump.rs`). | Não; diagnóstico. |
| #3028 | D3D12.X: a pipeline build that panics fails like a driver error instead of hanging its waiters | `run(job)` em `pso_async.rs` envolve o trabalho em `catch_unwind`, registra aviso com taxa limitada (16 primeiros, depois potências de dois) e devolve 0, o resultado de uma build de driver que falhou — o caminho existente dá pipeline substituto ao bind. Antes, o `recv` do `build_pieces` bloqueava para sempre e o pool encolhia. +70/−4 em 3 arquivos. | Área de **XCL D-02** (regressão que derrubou o `amdxc64`) e do `TuxBox D-01` (estouro de pilha ao criar pipelines): falha de build de pipeline, mas aqui é panic de worker, não crash do compilador. |
| #3029 | XGameRuntime: dispatch serialized task-queue work on the caller | `XTaskQueueDispatch` devolvia `false` para porta `SerializedThreadPool` mesmo com callback na fila; passa a reivindicar um item como porta manual e deixa o worker drenar o resto. Fixes #2948. +101/−20 em `xgameruntime/taskqueue.rs`, com teste. | Não. |
| #3030 | VFS: respect PLS quotas in legacy disk-space queries | `GetDiskFreeSpaceExA/W` já limitava ao quota de armazenamento persistente; `GetDiskFreeSpaceA/W` devolvia os clusters do volume do host. Passa a manter o tamanho de cluster nativo e reportar clusters livres/totais dentro do quota. Fixes #2943. +157/−36, com teste `pls_quota.rs`. | Não. |
| #3031 | XAudio2: reset the Opus decoder at a queued Discontinuity | `Discontinuity` marcava o último buffer enfileirado como fim de stream sem passar pelo flush de fim-de-stream ou de `FlushSourceBuffers`, e o pacote seguinte herdava o pre-skip do stream anterior; passa a dar flush no decoder Opus quando ainda há buffer na fila. Fixes #2942. +232/−7 em `hle/xaudio2.rs`. | Não. |
| #3032 | D3D12.X: forget a released resource in the console heap's resource map | O mapa `RES` de `console_heap.rs` só era preenchido com `XCL_TRACE_FRAME`/`XCL_DUMP_TARGETS` e nada removia a entrada na liberação, então uma busca por endereço podia achar recurso morto e chamar `GetDesc` em ponteiro morto. `res_release` passa a esquecer quando a última referência cai. +32 em 2 arquivos. | Área de uso-após-liberar; sem hash do catálogo. |

## Pull requests mesclados (1)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #1824 | Input: load keyboard and mouse bindings per title | `81693f02` | Confere com o prometido: o perfil por título em `xcl.toml` (`config.rs`), a tabela de bindings movida para `kbm.rs` e a navegação por teclado passando pelo perfil (`gameinput.rs`), mais 29 linhas de README. O PR declara `cargo test --workspace` com 865 passando, 42 ignorados, e o commit `b8005022` é o único commit de conteúdo. |

## Cruzamento com o catálogo

- **`VIctory D-20`** (`v_cmp_lg` baixado como `fcmp one` — verdadeiro para NaN no driver AMD) e
  **`XCL D-14`** (`v_cmpx` float via bridge descartava o EXEC anterior): #3023 corrige exatamente a
  mesma família semântica no tradutor direto RDNA4 de outro projeto. É o achado mais forte da hora e
  reforça que a classe `fcmp one` reaparece a cada novo caminho de tradução.
- **`XCL D-02`** (regressão que derrubou o `amdxc64`): #3028 trata panic de build de pipeline sem
  deixar devedores pendurados — área de robustez de build, sem tocar o compilador.
- **`XCL D-20`** (barreira de `ExecuteIndirect` no NVIDIA): #3024 junta sinais de fence de
  `UpdateTileMappings`; é sincronização, não a barreira de argumentos indiretos.
- **Nenhum hash** das colunas `introduzido`/`corrigido` do catálogo aparece nos dois commits.
- **Defeito novo**: nenhum identificado por diff. `#3025` descreve reuso prematuro de bloco quando o
  cool de 2 s expira com trabalho em voo — se confirmado em título, é candidato a entrada nova.

## Confiança

Verificado: os dois diffs foram lidos pela API (`commits/<sha>` com patch) e o merge #1824 confere com
`merge_commit_sha` `81693f0207`; `b8005022` tem data de autor 24/11 e entrou na `main` agora pelo
merge — o `compare` o lista porque não é ancestral de `06f90c19`, não porque o cursor retrocedeu.

Não verificado: (i) os 10 PRs abertos foram lidos pelo corpo (até ~1.100 caracteres) e pela lista de
arquivos, nenhum pelo patch; números como "41 mil sinais → 700–1.000" são declaração do autor;
(ii) `#3023`, `#3024` e `#3025` têm base `amd-dev`, então não entram no cursor de commits da `main`
nem no de merges — se forem mesclados na `amd-dev`, esta rotina não os vê; (iii) nenhum teste foi
executado aqui.
