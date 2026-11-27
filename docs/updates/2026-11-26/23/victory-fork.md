# fxceless/victory-fork — atualização de 2026-11-26 23:00

## Resumo

Hora de absorção em massa: sete PRs mesclados num intervalo de 22 segundos (`23:06:25` a `23:06:47Z`),
cada um com um commit de merge próprio, mais três commits diretos grandes do mantenedor. Os dois eixos do
delta são o tradutor de shader — cargas raw que passam a pedir só as dwords lidas, `s_quadmask`,
`frexp/ldexp` exato, SAD de byte, quad de pixel — e o encanamento de GPU — fence de host validada,
tickets de readback re-registrados, `clzero` que cai no guard em vez de matar o run. Vinte e oito PRs
abertos novos; vinte documentados, oito no `backlog`.

## Commits novos (20)

Faixa `52ac4ec0..81d54b06`. Três commits são de terceiros (Nikita, spk, Arbel) chegados por PR; os demais
são do mantenedor, com cherry-pick indicado no corpo quando é o caso.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `0a40a00a` | fxceless | Raw loads ask only for the dwords read; `VICTORY_SHADER_OPT` bit 128 escreve float com fast-math (off); hash de cartão um grupo por vez e 512 MB de tiles pequenos por padrão | 5 arquivos, +148/−11. `emit/bitcode/opt.rs` ganha `FAST = 128` (fora de `ALL`, para `127` continuar "todos os passes") e, em `narrow_raw_loads`, um `dx.op.rawBufferLoad` vivo passa a pedir só as dwords realmente lidas. `host.rs`: `VICTORY_CARD_HASH_WIDE` passa a 1 (um grupo por vez) por padrão — run 349 mediu −10 ms de cartão; `tiles.rs`: `small_when_full_mb` passa de 0 para 512, porque sem isso o run 341 mostrou 612 texturas pretas. Cherry-pick de `5d4a0265`. |
| `0e006f00` | fxceless | Masked patch e mesh launch ligados por padrão (`=0` desliga; o payload mesh do tradutor segue o mesmo padrão) | 3 arquivos, +9/−7 (`launch.rs`, `shadow.rs`, `emit/shader.rs`). Corpo: run 362 espera pelo cartão 21 → 2 ms e a frame usual −10 a −15 ms; run 361 cartão −5 ms; imagens conferidas nas fases ligadas. |
| `1f5d17a6` | fxceless | README: status de 2026-11-27 01:20 (masked patch, mesh launch) | 1 arquivo, +16/−5. |
| `30c14669` | fxceless | Spare chunks de scratch ficam limitados a 128 MiB por padrão aqui (`VICTORY_SCRATCH_SPARE_MB` sobe) | 1 arquivo, `host.rs`, +3/−1. Motivo: o PC fica sem memória ao lado do título. Cherry-pick de `c6c8b8f7`. |
| `3221ac42` | fxceless | `VICTORY_MASKED_PATCH` (off, alt; fps47): página com mais runs mudados que o teto de merge é remendada no cartão por máscaras de byte depois das flights, em vez de assentar | 2 arquivos, `host.rs` e `shadow.rs`, +460/−12. Substitui o caminho de "settle" por um patch por máscara quando os runs passam do teto. |
| `34bcce8d` | fxceless | README: status da noite de 2026-11-26 (hash de cartão um grupo por vez por padrão, PRs mesclados) | 1 arquivo, +25/−5. |
| `4c604661` | fxceless | Pipelines: o teste de pipeline pendente espera um pipeline de computação feito na hora, agora que dispatches esperam pelo seu (#1312) | 2 arquivos, +5/−3 (`README.md`, `pipeline/make.rs`). Ajusta o teste à mudança do #1312. |
| `5a83e5a2` | Ym1r1710 | GPU: o primeiro chunk pequeno de scratch de uma lista pode ser um de 4 MiB mantido; 512 MiB de spare chunks (#1367) | 1 arquivo, `host.rs`, +52/−15. Spares ficavam no teto de 128 MiB o run inteiro, todos de 4 MiB; o primeiro chunk (256 KiB) era feito novo toda vez. Numa RTX 5080 eram ~35 chunks de 256 KiB (~1 ms cada para liberar) e 12 de 4 MiB (2–2,5 ms cada) por frame, na thread que os soltasse. `VICTORY_SCRATCH_ANY_SIZE` ligado por padrão. |
| `62a8d51e` | fxceless | Second outside list: fast-math por padrão com bit 24 (testes de NaN/infinito mantidos), merge runs 4096, join copies/plain fills/step index ligados, 512 MiB de spares, joins 16/128, refresh de textura 8 vias, listas do heap worker 128 MiB, workers de pipeline lógico-4; segmentos de gather abertos sobrevivem a dispatches até barreira do título ou próximo draw indireto (`VICTORY_SEGMENTS_ACROSS`) | 11 arquivos, +133/−31. É a segunda "lista de fora" (conjunto de switches padrão) aplicada em `draw.rs`, `heap_worker.rs`, `host.rs`, `indirect.rs`, `list.rs`, `pipeline/make.rs`, `queue.rs`, `shadow.rs`, `emit/bitcode.rs`, `bitcode/opt.rs`, `emit/mod.rs`. O body registra que segue `VICTORY_INDIRECT_BATCH=2` do avatarmanz. |
| `81d54b06` | fxceless | Fork PR #1407 (PiloteCode): o corte da primeira fatia de um compare grande satura no início da cópia (dava a volta: a primeira fatia nunca era comparada) | 1 arquivo, `shadow.rs`, +38/−1. |
| `83a5593b` | Nikita Romanovich | GPU/texture: cair para `copy_mapped` em texturas parcialmente mapeadas nunca preenchidas sob `no_wait` (#923, #1284) (#1392) | 1 arquivo, `texture.rs`, +85/−24. |
| `86d446b7` | spk | GPU: re-registrar tickets de readback de textura invalidados (#1376) | 4 arquivos, +123/−4 (`host.rs`, `list.rs`, `resource.rs`, `texback.rs`). |
| `a42b627e` | Exopus | Pipelines: um dispatch espera pelo pipeline de computação até ele ser feito (`VICTORY_PIPELINE_COMPUTE_WAIT=0`: dentro do orçamento da frame, como antes) (#1312) | 1 arquivo, `pipeline/make.rs`, +26/−1. Um draw dropado é redesenhado na frame seguinte; um dispatch não. Corpo: no corredor da prisão um programa de computação novo levou 4,4 s para ser feito numa máquina de 16 GB, seu único dispatch foi dropado depois do orçamento de meio segundo, e as janelas da sala anterior ficaram misturadas sobre a imagem inteira por minutos (run 195). |
| `a7ee5dfa` | groster | Spread dispatches: `VICTORY_SPREAD_LEAN` roda os registros dentro de 65.535 grupos pela assinatura simples (off) (#1352) | 3 arquivos, +510/−37 (`draw.rs`, `host.rs`, `indirect.rs`). Um dispatch indireto que usa as palavras de spread passa por uma assinatura que fixa dois root constants e despacha; numa RTX 3060 isso custa ~13 ms de cartão por frame para ~1.000 execuções, das quais ~1 por frame pede mais de 65.535 grupos em x. O switch clampeia lotes de até 64 registros em dois arrays (um para a assinatura simples, com os que passam do teto vazios; outro `[row, asked x, x, y, z]` para a de spread) e um predicado entre eles. |
| `b1d777a8` | fxceless | Wide raw loads guardados no fim da view e ligados por padrão; `ds_bpermute` dentro da sua metade; leituras de quad de pixel; first-lane de helper lane; testes de NaN nos bits; `VICTORY_SHADER_FAST_BITS`; testes de nó de raio respondem "sem acerto" por padrão; PRs #1397 (wave64 pareado, off), #1399 e #1400 mesclados à mão; `VICTORY_CARD_FILLS` e `VICTORY_MESH_LAUNCH` (off); threads mantidas para checagem de textura | 34 arquivos, +4.686/−910. O maior commit da faixa. Mexe em `emit/bitcode`, `emit/container`, `emit/intersect`, `emit/loop_regions`, `emit/lower`, `emit/paired`, `emit/shader`, `emit/wave64_*`, `launch.rs`/`launch.hlsl`/`launch_code.rs`, `pipeline/host.rs`/`library.rs`/`make.rs`, `shader_cache.rs`, `shadow.rs`, `texture.rs`, `tiles.rs`, `watch.rs` e `descriptor/host.rs`. |
| `b3b65aa2` | Arbel | `clzero` numa linha guardada dá fault no guard em vez de encerrar o run (#1380) | 1 arquivo, `console_insn.rs`, +151/−13. A emulação de `clzero` (uma isolada, o laço de 12 bytes e o de 15) recusa linha que `writable` nega, com o argumento de que "a instrução real também dá fault ali" — mas no console o fault é violação de acesso, que o handler vetorizado do write-watch resolve; aqui era instrução ilegal, e o guard nunca a via. Corpo: `0xc000001d` no laço de 12 bytes, 22 minutos dentro da abertura, numa CPU sem CLZERO. |
| `c005a245` | fxceless | Região de ramo por lane que só levanta flag de LDS roda como duas waves de host (depois de `DrTardis1963 1b5c3a1e`; `VICTORY_LDS_FLAGS=0` desliga) | 1 arquivo, `emit/lower.rs`, +256/−1. Medido sobre os 25.679 programas do pacote de shaders para host de 32 lanes: exatamente um muda — o tiling de reflexão `2cc4c197ee29e291` (`sr_tracing CS_Tiling_Generate_Opaque`), recusado antes por só existir em forma de 64 lanes, agora duas waves exatas de 32. |
| `d036cea8` | spk | GPU: rejeitar fence de host com conclusão falha em workers de fundo (#1366) | 1 arquivo, `host.rs`, +35/−11. Preserva o diagnóstico síncrono e rejeita o sentinela de device removed antes da publicação. |
| `d154dcb1` | fxceless | README: status de 2026-11-27 01:00; a lista de hashes `CS_RaytracingResolve` para `VICTORY_DENY_PIPELINES_FILE` | 2 arquivos, +1.021/−4 (`README.md` e `diagnostics/deny-raytracing-resolve.txt` novo). |
| `f9e0371a` | fxceless | `VICTORY_DENY_PIPELINES_FILE`: hashes de um arquivo também (um conjunto); para os 1.000 programas `CS_RaytracingResolve`, construídos 30–80 ms antes das perdas de cartão nos runs 281, 318 e 359, recusados pela pipeline library cada vez | 1 arquivo, `pipeline/make.rs`, +15/−5. |

## Pull requests abertos (20 de 28)

Acima de 1380. Os oito restantes (1412, 1413, 1414, 1415, 1416, 1419, 1421, 1422) ficam no `backlog` com
`last_pr_number` em 1411. Propostas lidas do corpo de cada PR.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 1381 | GPU: indirect mesh records launched by an amplification stage (`VICTORY_MESH_LAUNCH=1`, off) | Porto do patch 0041 de Kelvaroq (`stellarr-studio/kelvaroq`, GPL-3.0-or-later), só a parte de mesh launch; o hash de cartão largo fica de fora porque `main` já tem `VICTORY_CARD_HASH_WIDE`. | Não. Complementa o `0e006f00`, que liga mesh launch por padrão. |
| 1384 | GPU: use HashSet for `VICTORY_SKIP_PIPELINES` and `VICTORY_DENY_PIPELINES` | Troca `OnceLock<Vec<String>>` por `HashSet` e compartilha `parse_shader_hashes`; item 4 do #1379 (varredura linear em `pipeline::skipped_shader`). | Não. Perf. |
| 1385 | victory-gpu: propagate relay merges to current sibling shadows | `watch::wrote_own` passa a checar `guards().clean` e devolver o stamp pré-merge por página de alias; `Shadow::put_back_own` trata cruzamento de fronteira de página e propaga só os bytes mesclados aos shadows irmãos correntes. Fixes #1346. | Candidato a defeito novo (#1346), mesma área do `81d54b06`. |
| 1386 | Shaders: fast-path zero and square factors in legacy_mul/legacy_fmad | Fator `±0.0` devolve `+0.0` sem `mul`/`fma`/`fcmp`/`select`; fatores iguais (quadrado) emitem caminho próprio. Item 1a do #1383. | Não. |
| 1388 | GPU: record per-span stamps for partial watched-page refreshes | `Shadow::copy_stale` só marcava `copied[page-base]` em refresh de página inteira; refresh parcial deixava a página sem stamp e recopiava bytes de fronteira a cada frame. Fixes #1344. | Candidato (#1344). |
| 1389 | Shaders/host: skip Set\*Root32BitConstants when constants list is empty | `Root::Constants` com `Vec` vazio chamava `Set*Root32BitConstants` com contagem 0 e ponteiro para alocação vazia, o que dá `STATUS_ACCESS_VIOLATION` (0xc0000005) nos drivers NVIDIA. Fixes #1343. | **Candidato forte**: crash de driver por chamada com contagem zero. Sem id. |
| 1390 | GPU: reuse own-byte copy buffers (`VICTORY_EXACT_REUSE`), log card-cap turn-downs, split bind time | Base `52ac4ec`; três commits (`06b1fa4`, `515e421` e um terceiro de tempo de bind). | Não. Perf. |
| 1391 | GPU/descriptor: cap Rule::Start head block at texture size | Sob `Rule::Start` (padrão), `Spans::note` tratava qualquer view posicionada começando dentro de `HEAD` (4.096 bytes) de outra como substituta, sem olhar o tamanho real; com texturas pequenas ou paletas de tint empacotadas em 4 KiB, cada view nova marcava vizinhas como stale e anulava as views do host. | **Sim, variante do D-05 do victory-fork** ("regra de staleness por sobreposição total anulava slots de texturas vizinhas", corrigido em `310c6eb`): o PR corrige o caso "começa dentro da janela" que a correção de `310c6eb` não cobriu. Ainda aberto. |
| 1393 | GPU/draw: create D3D12 command signatures outside cache mutex | `host_spread_signature` e `host_signature` seguravam o `Mutex` de `MADE` durante `CreateCommandSignature`; no Intel Arc a chamada levava segundos e travava todas as threads de gravação. Item 1 do #1370. | Não. Perf/contenção. |
| 1394 | GPU/profile: bound saved-register offsets and Rip range in frame_fits | `frame_fits` validava só parte dos casos de unwind antes de `RtlVirtualUnwind` num slice de pilha copiado; `UWOP_SAVE_NONVOL`(4), `_FAR`(5), `_XMM12` etc. ficavam fora. Questão 7 do #1379. | Candidato: leitura fora dos limites no stack walk com `VICTORY_PROFILE`. |
| 1395 | GPU/texture: replace BLAKE3 texture content hash with 128-bit AVX2/scalar hash | BLAKE3 era usado só como detector de mudança em memória e respondia por ~27% do perfil da thread de submissão (#1375); troca por digest não criptográfico de 128 bits. | Não. Perf. |
| 1396 | Working shader translation + precompilation (Belobug) + v7-v10 RTX 5070 perf + main fixes | Traz de `Alpacher/victory-fork:pr/fxceless-full-v10` (head `fe703a0`) o que o autor diz fazer o jogo renderizar em RTX 5070, onde `main` (`52ac4ec`) renderiza preto. | Candidato: divergência de árvore em que `main` não compila shaders na RTX 5070. |
| 1398 | Shaders/opt: push Bitcast through Select/Phi and fold constant chains/masks | `lower.rs` guarda registradores como `i32` e `Op::Phi` como `i32`, então operações float guardadas emitem `bitcast` sistematicamente (`bitcast` e `select` são ~12% das instruções emitidas cada). | Não. Perf de tradução. |
| 1402 | Diagnostics: validate process-scoped PresentMon display captures | Validador offline em PowerShell para capturas PresentMon 2.x: identidade de processo/swapchain, lotes de mundo, continuidade de telemetria, erros do título, isolamento de warm pipeline; portado de `spk-c/victory-fork#36` sobre `4c604661`. | Não. |
| 1403 | GPU: bound conventional table CBV scratch fills and validate host ranges | O host limita CBV convencional a 64 KiB, mas o fallback de tabela ainda reservava e preenchia o tamanho inteiro declarado pelo título; limita o payload, inicializa padding, usa CBV nulo tipado para faixa inválida. Adaptação de `spk-c/victory-fork#30`. | Candidato (alocação/preenchimento acima do limite). |
| 1404 | Diagnostics: measure title submission driver and descriptor stages | `VICTORY_SUBMIT_STAGES=1` (off) reporta sync de descritor, realização e join, e duração de `ExecuteCommandLists`; adaptação de Kelvaroq 0053 via `spk-c/victory-fork#40`. | Não. |
| 1405 | Diagnostics: add opt-in title no-draw mode | `VICTORY_NO_DRAWS=1` suprime lookup/criação de pipeline do título e registro do builder de BVH; adaptação de Kelvaroq 0061 via `spk-c/victory-fork#39`. | Não. |
| 1406 | GPU: keep bounded sparse refresh spans for known unwatched buffers | `VICTORY_SPARSE_REFRESH=1` evita copiar lacunas grandes entre faixas de um shadow conhecido como não observado, mantendo até oito spans mesclados com coalescência de 64 KiB. Adaptação de `spk-c/victory-fork#38`. | Não. Perf. |
| 1408 | Shaders: correct pixel quad, DPP8 and whole-quad execution semantics | Corrige DPP8 recusado, DPP ignorando lanes inativas e execução de quad inteiro; base no approach nativo de quad do `DexNom/VIctory 921d8d3b`. | Não catalogado; mesma família do `DexNom D-07` (operações de quad em pixel shader), que o DexNom já corrigiu em `921d8d3`. |
| 1411 | Tests: exercise Wine watch capabilities and placeholder fallback | Dois testes de watch falhavam no GE-Proton11-14 antes de exercitar o comportamento suportado: stub de Event Log do Wine e `STATUS_INVALID_INFO_CLASS` na classe de informação de placeholder. | Não. Teste. |

## Pull requests mesclados (7)

Todos com `merged_at` entre `23:06:25` e `23:06:47Z`; cada `mergeCommit.oid` bate com o commit de merge
na lista de commits da `main`.

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| 1366 | GPU: validate host fence completion before background publication | `d036cea8` | `host.rs` +35/−11: sentinela de device removed rejeitado antes de publicar. |
| 1376 | GPU: re-register invalidated texture readback tickets | `86d446b7` | `host.rs`, `list.rs`, `resource.rs`, `texback.rs` +123/−4. |
| 1380 | clzero on a guarded line faults into the guard instead of ending the run | `b3b65aa2` | `console_insn.rs` +151/−13: `clzero` em linha guardada passa pelo handler do write-watch. |
| 1367 | GPU: a list's small first scratch chunk may be a kept 4 MiB one; 512 MiB of spare chunks | `5a83e5a2` | `host.rs` +52/−15, com `VICTORY_SCRATCH_ANY_SIZE` ligado. |
| 1392 | GPU/texture: fall back to copy_mapped for never-filled partly mapped textures under no_wait | `83a5593b` | `texture.rs` +85/−24. |
| 1352 | Spread dispatches: VICTORY_SPREAD_LEAN… | `a7ee5dfa` | `draw.rs`, `host.rs`, `indirect.rs` +510/−37; switch desligado. |
| 1312 | Pipelines: a dispatch waits for its compute pipeline until it is made | `a42b627e` | `pipeline/make.rs` +26/−1; `VICTORY_PIPELINE_COMPUTE_WAIT=0` volta ao comportamento antigo. |

Os PRs **#1397, #1399 e #1400** foram, segundo o corpo de `b1d777a8`, "merged by hand": não aparecem na
listagem de `state=merged` desta coleta (nem entre os 15 fechados com merge nos últimos 100 por
`updated`). Não têm hash de merge próprio a citar.

## Cruzamento com o catálogo

- **D-12** ("loads escalares agrupados de 4 dwords leem além do fim da view (água preta)", `corrigido`
  vazio no catálogo): **corrigido por `0a40a00a`**, que acrescenta `narrow_raw_loads` — uma carga raw viva
  passa a pedir só as dwords lidas —, e por `b1d777a8`, que liga por padrão as "wide raw loads guardadas
  no fim da view". São os dois hashes que faltavam na coluna `corrigido`; nenhum outro commit da faixa
  mexe nesse caminho.
- **D-05** (regra de staleness por sobreposição total anulava slots de texturas vizinhas, `310c6eb`):
  nenhum commit mesclado reabre ou refaz. O PR aberto **#1391** cobre a variante que sobra ("começa dentro
  de `HEAD` = 4 KiB, sem olhar o tamanho"), ainda não mesclada.
- **D-16** (perdas de device sem causa identificada, runs 204–279 e mundo aberto): `f9e0371a` e
  `d154dcb1` dão **evidência e mitigação**, não correção — 1.000 programas `CS_RaytracingResolve`
  construídos 30–80 ms antes das perdas dos runs 281, 318 e 359, agora recusáveis por arquivo de hashes.
  O D-16 continua aberto com um suspeito nomeado.
- **D-02** (chave do cache de shader sem os switches de wave, `e9765a7`): `0a40a00a` acrescenta um switch
  novo (`VICTORY_SHADER_OPT` bit 128) e um passe novo (`FAST`) — não há, no diff, mudança de
  `translation_fingerprint`. **Não reabre o D-02, mas também não o estende ao switch novo**; se `FAST`
  altera o resultado, a chave precisa dele. Fica como candidato de conferência.
- **D-11** (texturas escritas por compute não chegavam à CPU do título, `f5c03f3`): `86d446b7`
  (re-registrar tickets de readback invalidados) mexe no mesmo caminho de tickets; é correção de um caso
  de invalidação, não do D-11. Não reabre.
- **Abre defeito novo?** Dois candidatos com hash: `81d54b06` conserta um bug que estava na árvore — o
  corte da primeira fatia de um compare grande saturava no início da cópia e a primeira fatia nunca era
  comparada (o sintoma não tem entrada no catálogo). E `f9e0371a` nomeia uma classe de programa
  (`CS_RaytracingResolve`) que derruba o cartão — é o suspeito do D-16, não uma entrada nova.

## Confiança

Verificado: os 20 SHAs e metadados em `commits?sha=main&per_page=100`, com o cursor `52ac4ec0` na posição
20; cada commit lido em `commits/<sha>` (nome, mensagem, arquivos, linhas e trechos de patch de `opt.rs`,
`tiles.rs`, `host.rs`, `shadow.rs`, `console_insn.rs`, `lower.rs`); os 7 merges em
`pulls?state=closed&sort=updated&per_page=100`, com `merge_commit_sha` igual ao commit na `main`; os 28
PRs abertos de `gh pr list --limit 50` acima de 1380.

Não verificado: as medições (`−10 ms`, `21 → 2 ms`, `4,4 s`, `13 ms`, `22 minutos`) são do corpo dos
commits e dos PRs. Não verificado se `narrow_raw_loads` cobre todos os caminhos que o D-12 descreve — a
leitura foi de trecho de patch, não de execução. Não verificado o estado dos PRs #1397/#1399/#1400: os
números vêm do corpo de `b1d777a8`, e a API não os devolveu como mesclados nesta janela. As propostas dos
20 PRs abertos vêm do corpo, sem diff lido.
