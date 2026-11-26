# ivall/xcl — atualização de 2026-11-26 00:00

## Resumo

Sem commit novo: a `main` continua em `b49be1836844fe0b0cdb3211dce3cb7fb4e21ac7`. Nove pull requests
abertos acima de 2614, a maioria endurecendo o caminho D3D12.X (cache de PSO, segmentos adiados,
streams de PSO gráfico estendido) e um deles trazendo a preparação de shaders em processos worker.

## Commits novos (0)

`gh api repos/ivall/xcl/commits?sha=main&per_page=100` devolve `b49be183` como primeiro elemento, igual
ao `last_commit` do estado. Nada novo.

## Pull requests abertos (9)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 2615 | D3D12.X: a bind never builds a cold pipeline on the title's thread | Com `XCL_PSO_WAIT_MS` não definido, um bind de pipeline fria passa a receber o stand-in na hora e a mandar a compilação para os workers, em vez de compilar no thread do título (`pso_async.rs` +134, `bind.rs`, `list_x.rs`, `console_list.rs`). O corpo relata um thread do título preso 124 s no compilador da AMD e o último frame em ~125-126 s. | Não é D-xx. `XCL D-02` foi uma regressão que derrubou o `amdxc64`; aqui é espera de compilação no thread errado. |
| 2616 | D3D12.X: preserve replacement driver blobs after refusal | O leitor de cache de PSO retém o handle do arquivo que forneceu os bytes e invalida exatamente esse objeto após recusa do driver, para não apagar um blob de substituição publicado no mesmo caminho (`real.rs` +143, `shaders.rs`). Fixes #2601; alternativa ao `#2614`. | Não. Corrige #2601, que não está no CSV. |
| 2618 | Console CPU instructions: monitorx / mwaitx are rewritten in place at their first fault | Reescreve `monitorx` para nops e `mwaitx` para `pause` + nops na primeira falta, com um único compare-exchange atômico de 8 bytes quando a instrução cai em palavra alinhada (`eh.rs` +86). Mesmo tratamento que o `clwb` já recebia. | Não. Cobertura nova de instrução; o catálogo de XCL não tem item de CPU. |
| 2620 | Tests: f32 VOP1 math and bit scans against the CPU on random inputs | Teste `gpu_f32_math_and_bit_scans_random` com 22 operações VOP1 f32 e os bit scans, 50 dispatches por padrão, wave64 e wave32, comparando bit a bit onde a ISA é exata (`gpu/recomp/tests.rs` +145). É o teste que encontrou o `#580` (`v_frexp_mant_f32` perdia o sinal), corrigido em `#587`. | Não abre nem fecha D-xx; é a cobertura que faltava para `D-06`/`D-07` (arredondamento f16) na família f32. |
| 2621 | D3D12.X: preserve extended state in translated graphics PSO streams | Detecta subobjetos de sufixo desconhecido e cauda truncada (`known_subobjects_checked`) e passa a marcar o stream como não-cacheável, além de projetar o estado estendido (depth bias exato, stream de ...) em vez de rebaixar em silêncio para `D3D12_GRAPHICS_PIPELINE_STATE_DESC` (`real.rs` +325). Fixes #2606. | Não. |
| 2622 | D3D12.X: a put-off call's segment that does not close is not submitted | `segment_close`/`deferred_segment` passam a olhar o `Close` nativo: segmento cuja gravação foi recusada não entra na submissão. O corpo cita um teste em que submeter um segmento assim removeu o device (0x887A0001) (`list_x.rs` +174, `title_list.rs` +10). | Não é D-xx; corrige #2088/#2118. `D-20`/`D-21` são sobre `ExecuteIndirect`, sintoma diferente. |
| 2623 | XGameRuntime: forward every IID of a forwarding class through its own clone (#2399) | Cada IID de classe de forwarding passa por `forward_alias_obj`, que faz clone por IID sob o lock do alias map; o `Obj` compartilhado fica com `pc` vazio, fechando a corrida de refcount e o over-read de vtable latente (`api.rs` +17). Fixes #2399. | Não. |
| 2624 | xcl run: shader preparation in worker processes, only what earlier runs used; upstream main; sampler fix | Traz o `nvidia-dev` até a `main` upstream (`b49be183`), prepara shaders na janela antes do título rodar e compila pipelines em processos worker (processadores lógicos menos quatro, 3 GB de commit livre cada, `XCL_PREPARE_WORKERS`). Diff enorme: 83 arquivos, +12.712/-687. | Não. É a maior proposta da fila; inclui correção de sampler e mudanças em `dxil/`, `recomp/` e `hle/d3d12x/`. |
| 2625 | D3D12.X: count skipped draws and dispatches by cause | Mantém os totais `DRAWS_SKIPPED`/`DISPATCHES_SKIPPED` e acrescenta contador por causa, anexado à linha periódica "console code so far" (`bind.rs` +278, `list_x.rs` +88, `pso_async.rs` +23). Sem mudança de comportamento. | Não. Instrumentação para diagnosticar o stall de carregamento de mundo. |

## Pull requests mesclados (0)

Nenhum PR com `mergedAt` posterior a 2026-11-26T02:11:35Z. O maior número mesclado devolvido é 2466,
abaixo de `last_pr_number` 2614, o que confirma que numeração de PR não acompanha ordem de merge neste
repositório.

## Cruzamento com o catálogo

- Corrige D-xx? Nenhum dos 24 itens de `catalogo/defeitos.csv` deste repositório (`XCL D-01` a `D-24`).
  Os que ainda estão abertos — `D-06`, `D-07`, `D-08` (arredondamento f16 e `-0.0` no caminho HLSL) — não
  são tratados por nenhum PR desta fila, e `#2620` só cobre f32.
- Abre defeito novo? Não verificável. `#2622` declara ter reproduzido remoção de device (0x887A0001) por
  segmento submetido sem `Close` válido; se o caminho existir na `main`, é defeito catalogável ali.

## Confiança

Verificado: topo da `main` e ausência de delta; números, títulos, corpos, arquivos e linhas dos 9 PRs;
ausência de merge na janela.

Não verificado: nenhum PR foi compilado ou executado aqui. `#2624` toca 83 arquivos e o resumo acima é
o que o corpo declara — não conferi arquivo por arquivo se o diff faz exatamente isso.
