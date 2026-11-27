# brcly/TuxBox — atualização de 2026-11-27 00:00

## Resumo

Cinco commits na `main` em cerca de quarenta minutos (`f48ce028..9168b2fa`): dois merges de PR, dois
commits de PR e o commit de correção de revisão do #284. As duas frentes são D3D12 — um root de
computação indexado direto que **também** tem tabela de descritores passa a receber o mapa de índices de
recursos (#294, de Kayotic1337) — e o tradutor de shader, que deixa de recusar um `ds_swizzle_b32`
dentro de um laço uniforme de grupo de uma onda (#284, de s3rius). Nenhum dos dois toca os defeitos do
catálogo.

## Commits novos (5)

Faixa `f48ce028..9168b2fa`. Dois são commits de merge (`5444f388`, `9168b2fa`); os outros três são os
commits de PR que eles carregam.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `7349d832` | s3rius | xshader: accept uniform loops around compute swizzles for single-wave groups | 3 arquivos, +56/-0. Em `hlsl/driver/pipeline.rs` o laço que recusa troca de memória compartilhada dentro de um laço ganha uma exceção: com `single_wave = count <= 64`, um salto para trás cujo opcode **não** é 6 nem 7 (os de VCC, por lane) sai do filtro antes da checagem `compute swizzle inside a loop`. O fixture ganha `shader_loop()`, que embrulha o `ds_swizzle_b32` num `s_cbranch_scc1`/`s_cbranch_vccnz` para 0x00, e o teste ganha três casos: laço SCC traduz, laço incondicional traduz, laço VCC continua recusado. |
| `a27ca0ae` | s3rius | Fixed PR comments. | 11 arquivos, +380/-19. Troca a exceção solta do commit anterior por uma prova explícita, `exchange_loop_safe()` (`hlsl/driver/pipeline.rs`, +53/-8): recusa grupo com `count > 32`, aceita só back-edge incondicional (op 2) ou SCC (4, 5), e recusa se houver **qualquer** outro desvio dentro do corpo do laço (`s_branch`, condicionais SOPP, `s_setpc`). Recusa explicitamente as polaridades invertidas (`s_cbranch_execz`/`s_cbranch_vccz`): o console só desvia quando a máscara da onda inteira é zero, e um voto sobre condição por lane repete enquanto qualquer lane estiver limpa. No emissor, `control_flow.rs` (+31/-2) faz o laço aceito votar a condição de saída na onda inteira em vez de por thread, e `exchange.rs` (+11/-2) mascara a publicação do `ds_swizzle_b32` a zero para as lanes que já saíram do laço — sem isso as lanes restantes leriam o valor de uma lane terminada. `hlsl.rs`/`setup.rs`/`bindings.rs` acrescentam o conjunto `accepted_swizzle_loops` e a pilha de aninhamento. Entra também um verificador de GPU novo, `tools/check_compute_swizzle_loop.py` (+150) com o exemplo `compute_swizzle_loop.rs`. |
| `5444f388` | brcly | Merge pull request #284 from s3rius/xshader/compute-swizzle-loop | Merge de 11 arquivos, +423/-6; leva `7349d832` e `a27ca0ae` na forma final (o diff do merge é idêntico ao de `a27ca0ae`, que já contém o primeiro). |
| `341a9fd3` | Kayotic1337 | d3d12: give direct-indexed compute roots with tables the resource index map | 7 arquivos, +69/-12. `device/compute_table.rs` ganha `heap_indexed_tabled_root()` (+19): um root com `CBV_SRV_UAV_HEAP_DIRECTLY_INDEXED` (flag 0x400) **e** tabelas de descritores passa a ser tratado como bindless, desde que o custo declarado caiba nos 64 DWORDs do root — tabela custa 1, descritor de root custa 2, constantes custam `num32`. Em `reflected_root()`, esse root recebe `scalar_index = Some((0,0))` e visibilidade ALL, o mesmo que o #235 fazia só para roots sem tabela. Sem o mapa, `paged_layout_gap` pulava **todo** dispatch do pipeline com a mensagem `skipped compute dispatch: pipeline has no paged descriptor layout`. O teste `tabled_compute_roots_that_index_the_heap_take_the_index_map` cobre tabela única, duas tabelas + constantes, flag ausente e o caso de 15 constantes + 2 tabelas (cabe) contra 3 tabelas (não cabe). No Lanes entra a cena `bindless-table-compute` (`selftest.cpp`, `shader_fixture.py`, `verify.py`, README) — o mesmo store de UAV por índice de heap do `bindless-compute`, sob um root que também tem tabela de quatro UAVs que o programa não lê, nas variantes folgada e restrita a 64 descritores residentes. |
| `9168b2fa` | brcly | Merge pull request #294 from Kayotic1337/fix/direct-indexed-tabled-compute | Merge de 7 arquivos, +69/-12; leva `341a9fd3`. |

## Pull requests abertos (0)

Nenhum PR aberto novo: `gh pr list --state open` devolveu lista vazia para `brcly/TuxBox` nesta hora.

## Pull requests mesclados (2)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #294 | d3d12: give direct-indexed compute roots with tables the resource index map | `9168b2fa` | O que o PR promete, e o merge não acrescenta nada além: o mapa de índices para roots de computação indexados direto que também têm tabela, mais a cena `bindless-table-compute` no check de GPU e um teste unitário. O corpo declara que **não** conserta sozinho a falha do intro em AMD (#232): com este PR e o #258 o intro ainda termina em `gfx ring timeout`, com `Faulty page starting at address: 0x0`, status `0x601431` (SQC data read, permission fault), e parte dos dispatches continua pulada porque os roots caem fora das formas simple/extended. |
| #284 | xshader: accept uniform loops around compute swizzles for single-wave | `5444f388` | Traduz o `ds_swizzle_b32` em laço uniforme de grupo de uma onda e mantém a recusa para laço VCC divergente e grupo multi-onda. O corpo alega ~36 pedidos de ProjectScarlett na classe de recusa "compute swizzle inside a loop"; o merge traz as três provas no emissor (voto de onda, máscara de publicação, prova de back-edge) e o verificador de GPU, não só a relaxação do filtro. **Divergência registrada:** o corpo fala em grupos de "32/64 lanes", mas `exchange_loop_safe` recusa `count > 32`; o teste de `7349d832` descreve o caso de 64 lanes como "single wave". Não verifiquei qual unidade `count` mede no código final (lanes ou ondas do host). |

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum. O candidato natural, **D-22** ("`Failed to remap` (-3) em roots raster com
  várias tabelas de descritor", corrigido em `95b242b`), é caminho **raster**; `341a9fd3` mexe no ramo
  simple/extended de **computação** (`reflected_root`) e o próprio corpo do PR diz que "draws are not
  changed: raster roots with the flag still go through the existing pixel-stage scalar-index path".
  **Não corrige o D-22.** **D-27** (slot 80 trata dimensões em threads como grupos, issue #164) é
  interpretação de dimensão de dispatch; a cena nova usa o slot lógico 80 apenas como índice de heap.
  **Não corrige.** **D-28** (`WaveSize(N)`) não é tocado: as mudanças de onda do #284 são de ordenação de
  barreira e voto, não de `WaveSize`.
- **Abre defeito novo?** Nenhum demonstrado por diff. Dois pontos ficam registrados:
  - `341a9fd3` só cobre a forma "simple/extended"; o corpo do #294 afirma que dispatches com "more
    tables, or tables after constants" continuam pulados. É lacuna conhecida e declarada, não regressão.
  - `a27ca0ae` muda a semântica de emissão para laços aceitos (voto de onda na saída e publicação
    mascarada). O teste unitário cobre a decisão de aceitar/recusar, não o valor observado na GPU; a
    cobertura de execução é o `check_compute_swizzle_loop.py`, que não foi executado aqui.

## Confiança

Verificado: os 5 SHAs e metadados em `repos/brcly/TuxBox/commits?sha=main&per_page=100`, com o cursor
`f48ce028` na posição 5; cada commit lido em `commits/<sha>` (lista de arquivos, contagens e patch dos
arquivos de código); `341a9fd3` conferido como pai único de `9168b2fa` e `a27ca0ae` como pai único de
`5444f388`; os 2 PRs mesclados em `pulls?state=merged&search=merged:>=2026-11-26T21:54:42Z`, com
`merge_commit_sha` (`9168b2fa`, `5444f388`) encontrado na lista de commits da `main`; `gh pr list
--state open` vazio.

Não verificado: a execução de `python tuxbox.py check` e das cenas de GPU citadas nos corpos (#294 diz
"all six trials pass", #284 diz 335 s em "Lanes and fixtures"); o teste em placa NVIDIA, que o #294
marca como pendente e o #284 não marca; a unidade de `count` em `exchange_loop_safe`; e o número "~36
pedidos" do #284. Nenhum teste foi executado aqui; os 13 repositórios monitorados são somente leitura.
