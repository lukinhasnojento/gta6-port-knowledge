# brcly/TuxBox — atualização de 2026-11-26 00:00

## Resumo

Quatro commits na `main` (`4f7fa84c` → `bc4ba79f`), que são dois merges: `#206` faz o tradutor baixar
trocas de quad DPP16 convergentes em shaders de pixel, e `#208` acrescenta o cenário de self-test
`raster-tables`, que cobre o que o `#168` deixou pendente. Um PR aberto (`#209`) passa a registrar a causa
de stand-ins de compute traduzidos.

## Commits novos (4)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `5cd728f8` | user | xshader: lower converged pixel quad swaps | Commit de branch do `#206`, criado sobre `cb8c38bb` (+582/-5, 11 arquivos). `pixel_quad::straight_prefix()` novo (+67): só admite a otimização quando instruções decodificadas e os campos crus de DPP16 provam movimentos de eixo convergentes, com o último swap e as arestas de retorno verificados. O emissor passa a gerar `QuadReadAcrossX/Y` direto na entrada WQM provada, em vez de exigir wave de 32 lanes (`control_flow.rs` +5, `scalar.rs` +6, `pipeline.rs` +4). Traz `tools/check_straight_pixel_quads.py` (+124), o guest C++ de referência (+112), o exemplo sintético (+88) e 172 linhas de teste. |
| `ff9d54cf` | Lucas Boyles | lanes: draw through the second of two raster root tables | Commit de branch do `#208` (+509/-8, 8 arquivos). `examples/lanes/raster_fixture.py` novo (+122): par de programas de console escritos à mão (vertex fullscreen e pixel que lê um dword do slot 2 da segunda tabela do root, com FSSM e binding PSV `t3, space2`). `selftest.cpp` +146, `project_tuxbox/verify.py` +65 com a suíte `lanes-raster-tables` em `BASE_SUITES`, `tests/test_lanes_raster.py` +138, e um teste em `runtime/dxbc/src/tests/compute_table.rs` que fixa `Binding { table: 1, register: 3, space: 2, slot: 2 }` e exige que sem a linha de user-data o descritor fique "unconfirmed". O README do Lanes documenta o cenário. |
| `c2d7bbc8` | brcly | Merge pull request #206 from nztom/fix/xshader-straight-pixel-quad-swaps | Merge de dois pais com o conteúdo de `5cd728f8`. |
| `bc4ba79f` | brcly | Merge pull request #208 from postblink/test/lanes-raster-tables | Merge de dois pais com o conteúdo de `ff9d54cf`. |

## Pull requests abertos (1)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 209 | d3d12: record the cause for translated compute stand-ins | A TSV de falhas de pipeline mostrava "stand-in after translation (cause not recorded)". Em `runtime/d3d12/src/device/compute.rs` (+3), grava uma nota de recusa em cada ponto de retorno de stand-in: `driver compile deadline` no caminho de prazo de compilação grande (nas duas tentativas, traduzida e bytes originais) e `driver compile failed: 0x{código}` quando o driver rejeita o pipeline traduzido, para a razão aparecer na string de estágios (`CS=<hash> CS.used=<key> CS.refused=driver compile failed: 0x80004005`). `tests/failures.rs` +17 cobre. Declara `tuxbox.py check` verde no branch. | Não é D-xx. É instrumentação de diagnóstico; `D-02`/`D-03` (otimizador do DXC estourando tempo, conversão DXIL→SPIR-V de minutos) são as causas que essa nota vai passar a nomear. |

## Pull requests mesclados (2)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| 206 | xshader: lower converged pixel quad swaps | `c2d7bbc8` | O lowering de `QuadReadAcrossX/Y` para swaps DPP16 FI=1 com máscara completa, mais três shaders montados independentemente que recusavam em `cb8c38b` e passam a traduzir. O PR relata 1.024 pixels comparados contra oráculo de CPU em quatro draws (alvo inteiro e recortado) e que um swap com prefixo guardado continua recusando. |
| 208 | lanes: draw through the second of two raster root tables | `bc4ba79f` | O cenário `raster-tables` do Lanes: desenha só sob o root refletido dos dois programas; sob o root do título o pipeline vira stand-in e o alvo mantém a cor de limpeza, e tabela ou slot errado lê um decoy que nomeia sua view. A suíte roda com `TUXBOX_TRACE_RASTER_ROOTS=1` e exige as linhas de trace, nenhum stand-in, e nenhuma tradução sem root. |

## Cruzamento com o catálogo

- Corrige D-xx? `D-22` ("`Failed to remap` (-3) em roots raster com várias tabelas de descritor",
  corrigido em `95b242b`) é a área direta do `#208`, mas não é o mesmo defeito: o PR é **cobertura de
  teste** do caminho que o `#168` refletiu, e o catálogo não tem linha para "faltava cenário de duas
  tabelas". Os outros 28 itens do catálogo seguem com os hashes antigos, sem novo corretor nesta faixa.
- Abre defeito novo? Não evidente. `#206` recusa explicitamente o caso guardado em vez de traduzir
  errado; `#208` só acrescenta teste e fixture. Ambos exigem `tuxbox.py check` verde na GPU antes do
  merge, segundo o template do repositório.

## Confiança

Verificado por diff: os quatro commits foram lidos em `gh api .../commits/<sha>`, com pais conferidos
(`5cd728f8` sobre `cb8c38bb`; `ff9d54cf` sobre `4f7fa84c`; os dois merges de dois pais). Números de
`mergeCommit.oid` conferidos contra os commits documentados. PRs lidos por `gh pr view`.

Não verificado: as execuções de `tuxbox.py check` e os desenhos em GPU não foram reproduzidos aqui; os
resultados citados são os que os PRs declaram. Não abri o log de CI do repositório.
