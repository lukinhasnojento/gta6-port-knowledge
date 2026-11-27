# brcly/TuxBox — atualização de 2026-11-27 23:00

## Resumo

21 commits acima do cursor `d37f092c`, dos quais 12 de conteúdo e 9 merges (4 de PR, 5 de `main` em
ramo), e 4 PRs mesclados. Três frentes: o backing de páginas do `xmem` passa a ser preso ao tempo de vida
do `ID3D12Heap` (classe de D-17), o emissor de shader passa a provar reduções de wave atrás de guardas
com contagem de máscara e aposentadoria de lanes (`xshader`), e o diagnóstico de compilação de compute
ganha identificação exata da requisição (`request=<digest>`). 1 PR aberto (`#325`).

## Commits novos (21)

Cursor `d37f092ce5ff49871f5196f00e3bb32e28459848` → head `c05ce004b1eb18435b6e92b7239d156619afa6e9`
(`compare` status `ahead`, `ahead_by` 21, `behind_by` 0).

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `67508417` | Valdi320 | diagnostics: identify compute compile requests and sample host memory pressure | `shader_cache.rs` passa a devolver `ComputeTranslation { code, request }` em vez de `Vec<u8>`: `prepared_compute`/`compute_with_root`/`compute_for_original_root` calculam o digest da requisição (`source_digest`) e `translate_keyed` centraliza leitura do portátil + `compile::on_demand`. `compute.rs` usa `&translated.code` (não move mais o `Vec`), e o label do pipeline vira `CS <hash8> request=<digest> bytes=<n>`, com `request=original` no retry com os bytes do convidado; `narrow_translation` devolve também `"<source>_w32"`. O diff acrescenta ainda o teste `compute_compile_labels_distinguish_rooted_narrow_and_original_attempts`. Em `big_stack.rs`, `request_trace` liga log de `CreatePipelineState` curto **só** com `TUXBOX_TRACE_PIPELINE_REQUESTS=1` e só para label `CS ` com ` request=`; o resto continua reportando apenas chamadas ≥ 5 s |
| `5352c3ca` | Valdi320 | tools: replay Wave32 variants exactly and audit compute loop guards | Novo `tools/audit_shader_guards.py` (124 linhas): fixa emissor (`pin_emitter`), roda `command(emitter, source, work)` por requisição arquivada, exige `profile` `cs_*` e conta sítios de guarda no HLSL **sem comentários** — `++tb_guard < 65536u` (estruturado) e `while (tb_pc != N u [&& tb_guard++ < 1000000u])` (dispatch). O relatório não inclui bytes de shader, HLSL bruto, diagnóstico nem caminhos; digest inválido nunca executa o emissor (testado). `scarlett_replay_integrity.py` muda 1 linha |
| `4d7fba03` | Valdi320 | Merge main and report paired shader guard-shape transitions | Merge de `main` no ramo `work/pipeline-request-attribution-20261126` (75 arquivos, +8275/−454). O conteúdo novo do ramo é o par `67508417`/`5352c3ca`; o resto é `main` recebida |
| `795e78b6` | Valdi320 | Merge main preserving rooted compute translation and exact request attribution | Merge de `main` no mesmo ramo (105 arquivos, +12744/−682), preservando o `ComputeTranslation` com root |
| `e98ed363` | Valdi320 | Merge remote-tracking branch 'origin/main' into work/pipeline-request-attribution | Merge com **conflito em `runtime/d3d12/src/device/compute.rs`** (9 arquivos, +427/−14) |
| `d5b85162` | Valdi320 | Merge commit '4d7fba0' into work/pipeline-request-attribution | Merge de linha (7 arquivos, +288/−1), conflito em `docs/PROJECTSCARLETT.md` |
| `0640e507` | DrArroway | xmem: back physical pages with private memory and export backing holds | `physical_pages.rs` ganha `PageBacking` por página (`Unbacked`/`PrivateView`/`PrivateArena`/`SharedSection`), `cpu_views`, e o `Pool` ganha `private_views`, `holds`/`view_holds`, `next_hold` e `deferred_unmaps`; novos módulos `backing.rs` (+519) e `private_views.rs` (+337). `unmap` de página com hold passa a limpar o mapeamento e registrar `DeferredKind::SectionPlaceholder` em vez de coalescer a abertura. `shared_pages.rs` ganha `map_private_range`/`unmap_private_range` (`MEM_REPLACE_PLACEHOLDER`/`MEM_PRESERVE_PLACEHOLDER`). Exporta `TuxBoxAcquirePhysicalBacking`, `...AcquirePhysicalBackingRuns`, `TuxBoxReleasePhysicalBacking` em `xmem.def` |
| `db70b21d` | DrArroway | xmem, d3d12: fix eager section promotion hang and resolve XMem backings for CPU reads | Promoção de seção passa a ser preguiçosa: `placement.rs` deixa de chamar a promoção antecipada (−15/+3) e `heap/backing.rs` resolve backings para leitura de CPU (+124/−2). `private_views.rs` reescrito em parte (+461/−128): vistas privadas contíguas, snapshots por run. `virtual_memory.rs` +19/−3 |
| `2f296416` | DrArroway | d3d12, xmem: pin backing holds to ID3D12Heap lifetime and guard held slots | `heap.rs` cria `BackingGuard { _shadow, _backing }` e o pendura no heap nativo com `SetPrivateDataInterface(&IMPORTED_BACKING, …)`, via um `Proxy` de 3 slots (`QueryInterface` só devolve `IUnknown`); `ImportedHeap` **perde** os campos `_shadow`/`_backing` e a ordem de drop deixa de ser load-bearing. `resolve_backings` passa a devolver `ERROR_INVALID_ADDRESS` (não `ERROR_NOT_FOUND`) para faixa não mapeada dentro de abertura do XMem e a não coalescer entre `BackingOrigin` distintos. `place_gpu_target` aceita a faixa se `backing::is_backed_range` |
| `8db05fcc` | DrArroway | xmem, d3d12: track arena holds, retire promoted backings, and reduce alias memory | `PrivateArena` ganha `bytes` e contagem de holds (`Pool::arena_holds`); `PagePools::pieces` cacheia heaps de import espalhado como `Arc<PieceHeap>` (substitui `extra_heaps: Vec<ID3D12Heap>`); `page_mapping.rs` ganha `retire_range`/`retire_from_xmem`, registrado em `TuxBoxSetBackingInvalidationCallback`, que invalida import cujo intervalo cruza a faixa aposentada; páginas `GpuAlias` deixam de ser populadas/copiadas; `read-only TuxBoxIsPhysicalBacked` evita `PrivateArena` fantasma; `TUXBOX_TRACE_XMEM_BACKING=1` loga holds/arenas/unmaps adiados |
| `f4772cdd` | Nick Stallman | xshader: keep swizzles in the full-mask resolve mode | Uma linha em `wave_loops.rs::full_mask_program`: a recusa de `Enc::Ds` passa a excetuar `ds_swizzle_b32`. O teste novo `a_swizzle_reads_lanes_that_stay_together` monta 2 dwords (`0xd8d4_8000`, `0x1100_001a`) e confere `WaveReadLaneAt`/`exec_lo`/`tb_guard < 65536u` no HLSL |
| `69106310` | Nick Stallman | xshader: branch with the whole wave through mask counts and lane retirement | `lockstep::prove` devolve `Proof { votes, ballots }` (antes `BTreeMap`). O analisador aceita: cópia de registrador `Conflict` propagando `Conflict` (`s_mov_b32/b64`) em vez de recusar; `s_bcnt1`/`s_ff1` sobre máscara como ballot da wave (EXEC sempre; outro registrador só quando `size == width`); `s_andn2_wrexec_b32` e irmãos escrevendo EXEC **e** destino; `tb_indexed_readlane`/`GROUP_READ` com lane de valor como uniforme; `v_mbcnt_*` com posição 0 como ballot; leitura de buffer com offset de **um** registrador (`words(position)` = 1 em `Mubuf`/`Mtbuf` pos. 2) e exigência de EXEC máscara. `emitter/exchange.rs` emite `WaveReadLaneAt` direto quando `lockstep` provou e a wave exigida é `mask+1`. `scalar.rs`/`vector.rs` usam `lockstep::word(...)` → `WaveActiveBallot(...)` para contar/primeiro-bit de máscara. +222 linhas de teste em `lockstep.rs` |
| `60f7cfad` | Nick Stallman | xshader: branch with the whole wave where the block dispatcher loses a guard's vote | `translate_outputs` passa a repetir a tradução dentro de `lockstep::without_guards(|| …)` quando a primeira falha com `lockstep::DISPATCHED_VOTE`, mantendo a recusa se a segunda também falhar. `pipeline.rs` transforma guardas de redução não vazias do `wave_sum::check_control_flow` em `DISPATCHED_VOTE` quando `instead_of_guards()`. `control_flow.rs` só troca a string do erro pela constante |
| `fcb58490` | DrArroway | xmem, d3d12: restore GpuAlias CPU readability and skip unmapped XMem pages in writeback | `copy_mappings.rs` popula os bytes da página **antes** de soltar o token anterior quando não há hold e o backing não é `SharedSection`; `private_views.rs` copia runs vivos em um único buffer por run em vez de um `Vec` por página e volta a publicar os bytes preservados; `try_write_xmem_bytes` extrai `write_host_slice` e pula página não mapeada do XMem em vez de falhar. O teste `is_physical_backed_...` passa a esperar `0x1234_5678` (conteúdo preservado) em vez de `0` |
| `07db0df9` | proof-by-vibes | d3d12: release the scratch pages past a stored texture snapshot | `placement.rs` ganha `Contents::store(level, bytes)`: depois de guardar o snapshot, chama `VirtualAlloc(MEM_RESET, PAGE_READWRITE)` nas páginas inteiras entre `ptr+len` (arredondado) e `ptr+capacity` (truncado), **somente** se sobrarem ≥ 64 KiB; `std::mem::replace` sai de `upload.rs`. 3 testes novos usam `QueryWorkingSetEx` para provar que a cauda deixa de estar residente e que o buffer pode crescer de volta |
| `4907e110` | brcly | Merge pull request #258 from DrArroway/feat/xmem-physical-backing | Merge de PR (19 arquivos, +2995/−146): `0640e507` + `db70b21d` + `2f296416` + `8db05fcc` + `fcb58490` |
| `ba780200` | proof-by-vibes | Merge branch 'main' into fix/snapshot-scratch-pages | Merge de `main` no ramo do #324 (57 arquivos, +5876/−172) |
| `41ddf163` | brcly | Merge pull request #324 from proof-by-vibes/fix/snapshot-scratch-pages | Merge de PR: `07db0df9` |
| `1f86cb9c` | brcly | Merge pull request #322 from NickStallman/xshader/wave-reduction-whole-wave | Merge de PR (12 arquivos, +474/−72): `f4772cdd` + `69106310` + `60f7cfad` |
| `db99a763` | brcly | Refresh README with terminal-style banner | `README.md` +21/−7 e `docs/assets/tuxbox-banner.svg` |
| `c05ce004` | brcly | Merge pull request #249 from Valdi320/work/pipeline-request-attribution-20261126 | Merge de PR (15 arquivos, +486/−31): `67508417` + `5352c3ca` e os 4 merges de `main` do ramo |

## Pull requests abertos (1)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| `#325` | xshader: number thread-derived lanes two by two where a compute group takes derivatives | Ordena as lanes derivadas de thread em pares quando o grupo de compute toma derivadas (o preço da ordem de lane do console numa wave hospedeira). Lido por título/base; patch não lido | Não identificado no catálogo por título; área é `xshader` (mesma de `#322`, recém-mesclado) |

## Pull requests mesclados (4)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| `#258` | xmem: back physical pages with private memory and export backing holds | `4907e110` (2026-11-27T19:22:37Z) | Exatamente os 5 commits do ramo (`0640e507`, `db70b21d`, `2f296416`, `8db05fcc`, `fcb58490`), conferidos pela lista de commits do PR na API. O commit de merge tem 19 arquivos e +2995/−146 |
| `#324` | d3d12: release the scratch pages past a stored texture snapshot | `41ddf163` (19:35:32Z) | `07db0df9` + o merge `ba780200` de `main`. Diff de 3 arquivos, +81/−1: só `Contents::store` e o teste de resident set. O commit promete RssAnon 6,35–6,8 GiB → 2,35 GiB medido em ProjectScarlett com RADV 24 GB — **medida não reproduzida aqui** |
| `#322` | xshader: branch with the whole wave where a guard's vote or lane reads would otherwise refuse | `1f86cb9c` (19:35:49Z) | `f4772cdd`, `69106310`, `60f7cfad`, 12 arquivos, +474/−72. O PR prometia recuperar requisições sem regredir nenhuma; o diff não traz contador de regressões, só os testes |
| `#249` | diagnostics: attribute compute pipeline failures and replay exact shader variants | `c05ce004` (20:01:33Z) | `67508417` + `5352c3ca` + 4 merges de `main`. 15 arquivos, +486/−31. Ferramenta de replay e label, não mudança de tradução |

## Cruzamento com o catálogo

- **D-17 (liberação imediata do backing de descritor causa falha de MMU) — mesma classe, tratada por
  outro caminho.** O catálogo registra o conserto em `98043df`. `2f296416` ataca a mesma janela de
  uso-após-liberar: o hold de backing e o `ShadowPages` deixam de viver no `ImportedHeap` (cuja ordem de
  drop era comentada como load-bearing) e passam a viver no heap nativo via
  `SetPrivateDataInterface`, com o comentário nomeando o motivo — `vkd3d-proton` destrói
  `resource->heap` **depois** de derrubar o `private_store`. Se isso cobre D-17 ou apenas a variante de
  heap importado com `ShadowPages` não foi verificado contra o caso do catálogo.
- **D-22 (`Failed to remap` −3 em roots raster) — mudança de código de erro na mesma área.**
  `2f296416` faz `resolve_backings` devolver `ERROR_INVALID_ADDRESS` para faixa dentro de abertura do
  XMem sem página mapeada, e `ERROR_NOT_FOUND` só fora dela. É semântica de `remap`; não há, no diff,
  teste que reproduza o `-3` com várias tabelas de descritor.
- **D-24 (resolução de merge apagou lowering e quebrou o verificador) — a janela tem merges com
  conflito exatamente no arquivo do caso.** `e98ed363` e `d5b85162` resolvem conflito em
  `runtime/d3d12/src/device/compute.rs` e `docs/PROJECTSCARLETT.md`. Lido o `compute.rs` resultante em
  `67508417`, `compute_with_root(...)` continua como segunda tentativa depois de `prepared_compute(...)`,
  e `translate_keyed` não perde o `read_portable`/`compile::on_demand` — o lowering com root não foi
  apagado nesta resolução. O merge `c05ce004` traz 15 arquivos; não foi conferido arquivo por arquivo
  contra `main` antes do merge.
- **D-02 (otimizador do DXC estoura o tempo) — não corrigido, agora observável.** `67508417` só liga
  trace de chamadas curtas (`TUXBOX_TRACE_PIPELINE_REQUESTS=1`) e nomeia a requisição exata no label;
  o teto de 5 s para logar continua. Nenhum commit da janela mexe no deadline nem no cache de pipeline.
- **D-28 (`WaveSize(N)` incompatível recusado só no driver) — não tocado.** A checagem
  `unsupported_wave_size(this, code)` em `compute.rs` já existia; o diff só muda o que é passado a ela
  (`&translated.code`).
- **Abre defeito novo?** Não verificado. Candidato a revisão no próximo ciclo: `07db0df9` mexe em
  memória de uma `Vec` por `VirtualAlloc` direto no intervalo `[len_arredondado, capacity_truncada]`;
  se a alocação do `Vec` não começar em fronteira de página, `start` é arredondado **para cima** e `end`
  para baixo, o que é conservador — mas depende de o alocador não devolver bloco cujo `capacity` cruze
  página não pertencente ao `Vec`. Não há teste com `capacity` desalinhada.

## Confiança

- **Verificado por leitura de patch:** todos os 21 hashes, seus arquivos e as afirmações acima sobre
  `compute.rs`, `shader_cache.rs`, `big_stack.rs`, `placement.rs`, `copy_mappings.rs`, `private_views.rs`,
  `heap.rs`, `heap/import.rs`, `lockstep.rs`, `wave_loops.rs`, `exchange.rs`, `scalar.rs`, `vector.rs`.
  A atribuição de cada commit ao PR (`#258`, `#324`, `#322`, `#249`) foi conferida na lista de commits do
  PR na API, não por mensagem.
- **Verificado por medição:** `compare d37f092c...c05ce004` devolve `ahead_by` 21, `behind_by` 0, status
  `ahead`. Contagem de PRs mesclados na janela: 4, com `merged_at` 19:22:37Z, 19:35:32Z, 19:35:49Z e
  20:01:33Z. Commits da faixa = 12 de conteúdo + 5 merges de `main`/linha + 4 merges de PR; o número de
  merges de PR bate com o de commits de PR na faixa.
- **NÃO VERIFICADO:** conteúdo das resoluções dos merges `4d7fba03`, `795e78b6`, `ba780200` arquivo por
  arquivo (só o resultado em `compute.rs`); a medição de RssAnon do `#324`; qualquer execução em GPU — os
  três commits de `xshader` dizem explicitamente que passaram pelo emissor, DXC e validador, e não por
  GPU. O patch do PR aberto `#325` não foi lido.
