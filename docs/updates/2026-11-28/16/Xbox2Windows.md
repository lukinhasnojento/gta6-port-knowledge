# R0Y00/Xbox2Windows — atualização de 2026-11-28 16:00

## Resumo

Sem commits novos: o head `f5a6f6ba` está parado desde 14:15Z de hoje (o `compare` desta hora devolve
`identical`). A hora é só de PRs: quatro abertos acima do cursor #3371, todos com diff pequeno e
verificável. O de maior consequência é `#3373`, que compõe todos os planos de apresentação quando há
mais de um — antes, um plano 0 preto deixava a janela inteira preta com áudio e render funcionando.
`#3375` corrige o índice de primitiva de mesh de 10 bits para 9, `#3372` deixa o compute direto
aceitar `image_store` a16 com dados de 32 bits, e `#3374` só troca caminhos locais de GDK por um
placeholder.

## Commits novos (0)

`compare(f5a6f6ba...f5a6f6ba)` = `identical`. Nada entrou na branch default desde a coleta anterior;
os quatro PRs abaixo são trabalho ainda não mesclado.

## Pull requests abertos (4)

| # | Título | Proposta (diff verificado) | Cobre defeito conhecido? |
|---|---|---|---|
| #3375 | shader: keep 9-bit mesh primitive indices | `gfx1013_direct_mesh.inl`: a máscara do índice de primitiva passa de `"1023"` para `"511"`. A exportação de mesh empacota três slots de 10 bits, mas cada slot é um índice de 9 bits com o flag de aresta no bit 9; com 1023, um índice com o flag virava 512..1023 e a guarda `triangle[c] >= vertices` cullava a primitiva. `gfx2hlsl.py` já usava 511. O teste novo (`native_direct_mesh_check.py`) monta `0x20280600` (três vértices com as três arestas) e exige `and i32 %t22, 511` e a ausência de `1023`. | Não por hash. Área de mesh/índices; o catálogo de Xbox2Windows não tem linha de índice de primitiva. |
| #3373 | Fix black screen on multi-plane presents | `startup_stubs/d3d12_xs_device.cpp`, `presented_planes`: a condição do plano único vira `present_composite_enabled() \|\| planes > 1`. Com mais de um plano e sem `X2W_PRESENT_PLANE`, todos são compostos; antes só o plano 0 era mostrado e um plano 0 vazio deixava a janela preta. +3/−1. | Área de apresentação/planos; D-01 do repositório trata de superfície apresentada liberada durante a cópia da assinatura — vizinho, sem hash. |
| #3372 | compute: lower 2D a16 image_store with 32-bit data in direct compute | `gfx1013_direct_compute.inl`: a recusa `store && !half` vira `graphics_input && store && !half`. A16 seleciona endereço de 16 bits independentemente da largura do dado (MIMG A16 no RDNA 2); o store 2D com dado de 32 bits passa a usar as coordenadas empacotadas (x na metade baixa, y na alta do primeiro VGPR) como o load a16 já faz, e os estágios gráficos continuam recusando. +3 no código e dois testes novos (`tests/native_image_store_a16_check.py`, +96, e o alvo CMake). | Área de **D-27** (coordenadas 2D a16 lidas como dois registradores), mas o defeito registrado é de `image_sample`/`image_sample_lz`; aqui é `image_store`. Sem hash. |
| #3374 | docs: use a neutral GDK root placeholder | Troca caminhos absolutos (`D:\gdk-oracle\…`) por `$gdk = '<out-of-repo GDK root>'` nos `docs/experiments/gdk-*.md`; as tabelas medidas e os argumentos das ferramentas não mudam. | Não. |

## Pull requests mesclados (0)

Nenhum `merged_at` acima de `2026-11-28T14:15:27Z`.

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum hash: os PRs ainda não têm merge. Por área, `#3372` mexe nas coordenadas
  `a16` (família de D-27) e `#3373` mexe em `presented_planes` (família de D-01). Nenhum dos dois é a
  linha registrada.
- **Abre defeito novo?** `#3373` muda o padrão de apresentação para **todos** os títulos com mais de
  um plano e sem `X2W_PRESENT_PLANE`: quem dependia de mostrar só o plano 0 passa a compor. É
  mudança de comportamento pretendida, mas nenhum teste de título com plano 0 preto está no diff — a
  verificação é a descrição do PR. `#3375` não abre nada: a guarda `triangle[c] >= vertices` continua
  descartando índice fora da faixa, agora com o valor certo de 9 bits.
- **Sem verificação de execução:** nenhum dos quatro foi rodado nesta coleta.

## Confiança

- **Verificado por diff:** os quatro PRs (`gh pr diff`), incluindo a linha exata alterada em cada um e
  os arquivos de teste de `#3372`/`#3375`.
- **Verificado por API:** números, títulos, `created_at`, `+/-` e lista de arquivos.
- **NÃO VERIFICADO:** o efeito em jogo real — tela preta corrigida, primitiva de mesh deixando de ser
  cullada, store a16 com 32 bits — é o que os PRs afirmam e o que o diff torna plausível; nenhuma
  execução foi feita. Também não se conferiu se `#3373` interage com `X2W_PRESENT_PLANE` definido para
  um plano inexistente.
- **Cobertura:** 0 commits, 4 PRs lidos por diff, 0 merges.
