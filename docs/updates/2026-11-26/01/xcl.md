# ivall/xcl — atualização de 2026-11-26 01:00

## Resumo

Sem commit novo na `main`: o cursor continua em `b49be183` e nenhum PR mesclou entre 03:16Z e 04:03Z.
Onze PRs abertos, quase todos de saneamento do `d3d12x` — referências COM de fila em quarentena de tiles
esparsos, tabela de alias que crescia sem limite, escrita de `used-pipelines.txt` sob o lock de bind,
locks globais de heap de descritor mantidos durante `CopyDescriptorsSimple` — mais um PR grande de regras
de input por quadro e dois de split program.

## Commits novos (0)

Nenhum. `last_commit` `b49be1836844fe0b0cdb3211dce3cb7fb4e21ac7` é o `head` da `main`.

## Pull requests abertos (11)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 2638 | D3D12.X: stop retaining strong ID3D12CommandQueue refs in sparse tile quarantine and prune stale queue fences | Decodere, +165/−51 em `hle/d3d12x/sparse.rs`. `queue_fences` guardava `(ID3D12CommandQueue, ID3D12Fence, u64)` e nunca removia: cada clone deixava uma referência COM forte a mais, mantendo filas liberadas vivas (`refcount >= 2`) e reacendendo o limite de 63 filas submetidas de #1668 (`E_OUTOFMEMORY`) mesmo com `present::QUEUES` descartando. | Não é D-xx. `D-20` (ExecuteIndirect lendo argumentos antes da barreira) é a linha vizinha de fila/barreira; aqui é vazamento de referência COM. |
| 2637 | GameInput: match presented frames to scripted input rules | KDenKC, +419 em 7 arquivos (`input_rules.rs` +259, `gameinput.rs` +60, `examples/retail-input-rules.toml` +21). `XCL_INPUT_RULES=<toml>` aplica regras de controle virtual casadas ao número do quadro apresentado e emite eventos JSON que scripts de lançamento podem esperar, para runs automáticos passarem telas de primeira execução e termos sem depender de `sleep`. Head `de2292ea` sobre `b49be183`. | Não é D-xx. |
| 2636 | D3D12.X: drop CreatePlacedResourceX alias table once logged and avoid hot-path formatting | Decodere, +128/−9 em `device_x.rs`. `note_alias` inseria em `Mutex<Option<HashMap<usize, String>>>` para sempre, mesmo depois das 16 linhas de diagnóstico; ganha fast-path atômico no teto e limpa `SEEN` ao logar o 16º alias, liberando o mapa. Também limita os VAs rastreados. | Não é D-xx. `D-11` (`VDPAU`/cache de DXIL per-lane) não é; a linha de vazamento mais próxima é `D-08` do Xbox2Windows (CLI que ignora `--output`), nada a ver. |
| 2635 | D3D12.X: batch used-pipelines.txt writes off the bind lock | Decodere, +234/−20 em `pso_async.rs`. `used::note` rodava `create_dir_all`/`open` e um `writeln!` síncrono por hash novo **segurando o mutex global**, no primeiro bind de cada conjunto de shader; em frames de carregamento de mundo isso serializa as threads. Passa a escrever em lote fora do lock. | Não é D-xx. |
| 2634 | xcl run: report log write failures once | luckyteagirl, +108/−6 (`log.rs` +106). Falha de `write_all`/flush em `xcl.log` depois de aberto era engolida e o resto do diagnóstico desaparecia; passa a reportar a primeira falha em stderr uma vez por processo e a continuar tentando. Follow-up de #2286; parte de #2277. | Não é D-xx. |
| 2633 | D3D12.X: the pieces of a split program read their slab from a root CBV | Kalmarv, **draft**, +1075 em 9 arquivos. Um `ExecuteIndirect` encadeado (#2404) com assinatura de comando custava 15,3 µs de GPU por rodada (vazio ou não) num RTX 5070 Ti, 4,9 ms a cada 32 rodadas de 10 peças, contra 0,41 µs com assinatura só de Dispatch. A cadeia passa a dar o slab (tamanho e origem, iguais para todas as peças da rodada) como root CBV `b0, space2`, o registrador que as peças já leem, em vez de root constants. | Não é D-xx. `D-20`/`D-21` são ordem de barreira e root signature errada em `ExecuteIndirect`; este é custo de assinatura. |
| 2631 | D3D12.X: drop shared heap locks during CopyDescriptorsSimple and clamp publish_range to heap bounds | Decodere, +188/−31 em `console_heap.rs`. `publish()` e `publish_range()` largam os locks `GLOBAL`/`GLOBAL_SAMPLER`/`HEAPS` antes de chamar `CopyDescriptorsSimple`, e `publish_range` recusa `n == 0` e limita a faixa aos limites de cada heap. Antes, a chamada de dispositivo ficava sob lock global, com risco de reentrância com hooks de dispositivo adotados. | Não é D-xx. `D-01` (janela UMA escolhida pelo dword alto sem carry) e `D-12` (stride de descritor inline) são descritores, mas não o lock de publicação. |
| 2630 | D3D12.X: the sparse windows' pool heaps are 4 MiB, so freed runs of tiles go back | imnicho, +45/−9 em `sparse.rs`. `POOL_TILES` cai de 1024 (64 MiB) para 64 (4 MiB): um heap de pool só volta quando todos os tiles estão livres, e o título libera runs espalhados por todos os heaps, então 1,5–2,0 GB de VRAM ficavam em heaps vivos como tiles livres depois do carregamento do mundo. Também marca os pools que voltam em vez de procurar em lista. Parte de #1773 (ponto 3). | Não é D-xx. `D-20` (ExecuteIndirect antes da barreira) é a linha de memória de GPU mais próxima; esta é fragmentação de heap esparso. |
| 2629 | D3D12.X: a split program's failed whole build is not tried again | Royalflamejlh, +28/−11 em `bind.rs`/`dispatch_chain.rs`. `Chain::whole` guardava o pipeline inteiro quando construído e **não** guardava a falha: todo `ExecuteIndirect` de um programa dividido (ou todo dispatch sem scratch) recompilava o programa inteiro, na thread do título e sob o lock da cadeia, podendo levar segundos e falhar de novo. A falha passa a ficar registrada. Parte de #2088. | Não é D-xx. `D-02` ("regressão: o codegen de #123 derrubou o compilador de shader da AMD") é falha de compilação no caminho per-lane, não recompilação repetida. |
| 2628 | D3D12.X: a worker waiting for a big compile turn leaves the queue to a helper | KOLESNiii, **draft**, +179/−9 em `pso_async.rs`/`real.rs`. Um compile grande espera em `big_compile_turn` por uma das poucas vagas (uma por vez por padrão, 3 GB de commit livre cada) e segura a thread do worker; no início do mundo do título de #505 todos os workers ficam esperando e a fila fica sem ninguém. Um worker que precisa esperar ganha uma thread auxiliar (até o número de workers) que serve a fila e sai quando ele volta; as vagas e a checagem de commit não mudam. | Não é D-xx. `D-02` é a mesma área (compilador AMD), mas ali o problema é o codegen derrubar o compilador. |
| 2627 | README: document xcl --version | romeawayfromhome-rgb, **draft**, +1 em `README.md`. Só acrescenta `xcl --version` ao bloco de exemplos; HELP e `main.rs` já tratavam. Docs apenas; declara não ter compilado (preparado em Linux, onde XCL não builda). | Não é D-xx. |

## Pull requests mesclados (0)

Nenhum. `gh pr list --state merged` (e a varredura por `merged_at` em `pulls?state=closed`) não encontrou
merge posterior a 2026-11-26T03:16:53Z.

## Cruzamento com o catálogo

- Corrige D-xx? Nenhum. As 24 linhas de `XCL` seguem com os corretores registrados; `D-06`, `D-07` e
  `D-08` (arredondamento f16, `-0.0` no caminho HLSL) continuam sem corretor, e nenhum PR aberto os
  declara.
- Abre defeito novo? Nenhum evidente nos diffs lidos. Dois candidatos de robustez, sem sintoma de jogo
  demonstrado: `#2636` (tabela de alias crescia sem limite e formatava string no caminho quente) e
  `#2635` (escrita síncrona de `used-pipelines.txt` sob o mutex global no primeiro bind). Ambos são
  vazamento/serialização de diagnóstico, não defeito funcional; ficam como nota, não como linha.
- Padrão de catálogo: `#2629` documenta um caso de "falha não memorizada → trabalho repetido", que não
  existe como linha em `XCL`. `#2630`, `#2638`, `#2631` e `#2633` são todos do mesmo bloco `d3d12x` e
  nenhum tem linha; o catálogo de `XCL` é fortemente orientado a tradução de shader e não cobre o
  subsistema de heap/fila do D3D12.X. Se a rotina for consolidar, esta é a lacuna.

## Confiança

Verificado por API: `gh api repos/ivall/xcl/commits?sha=main&per_page=100` devolve `b49be183` como `head`,
igual ao `last_commit` do estado — zero commits. `gh pr list --state open --limit 50` devolveu 11 PRs,
todos com número > `last_pr_number` 2625. A varredura de mesclados por `pulls?state=closed&sort=updated`
não achou `merged_at` posterior ao cursor.

Não verificado: nenhum PR foi compilado ou testado aqui; as medições (15,3 µs por `ExecuteIndirect`,
1,5–2,0 GB de VRAM em heaps vivos, `refcount >= 2`) são declarações dos autores. `#2627` declara
explicitamente não ter sido compilado pelo autor.
