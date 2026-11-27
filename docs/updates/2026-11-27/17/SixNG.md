# holdmysocks/SixNG — atualização de 2026-11-27 17:00

## Resumo

Um commit novo: o merge `2d9c230d` de #474, que conserta um erro de índice na máscara de blur do cabelo introduzido por #470 — o merge `790a3c67`, que era exatamente o cursor deixado pela execução das 07:00. Desde #470 os índices de um modelo endereçam o buffer de vértices inteiro, então `draw_indexed` com `base` somava o deslocamento do mesh uma segunda vez e as peças de cabelo depois do primeiro mesh liam vértices errados. Um PR aberto (#472, abaixo do cursor), nenhum outro merge na janela (`last_run` 07:02:25Z).

## Commits novos (1)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `2d9c230dea` | holdmysocks | Hair blur mask: no base vertex on absolute indices (#474) | Em `crates/sixng-gui/src/render3d.rs`, no laço que monta a máscara de blur das mechas: `first` deixa de ser calculado como `me.ib_range.start / 4` e passa a vir de `me.first()`, a variável `base` (`me.vb_range.start / size_of::<glb::Vertex>()`) é removida, e o `be.draw_indexed(first..first + cnt, base, range.clone())` vira `be.draw_indexed(first..first + cnt, 0, range.clone())` com o comentário "the indices address the whole vertex buffer: no base vertex". Três linhas trocadas; `cnt` continua limitado por `me.n_draw` e `me.n_idx`. |

## Pull requests abertos (1)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #472 | feat(metal4): add offline model cache baking tool | Ferramenta de "bake" offline do cache de modelos para o backend metal4. Aberto em 27/11 07:07:32, atualizado 07:31:49, sem `draft`. Fica abaixo do cursor 472, então não entra na janela desta hora e não foi lido além de título e estado. | Não verificado — o corpo e o diff não foram lidos nesta execução. |

## Pull requests mesclados (1)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #474 | Hair blur mask: no base vertex on absolute indices | `2d9c230d` | Uma alteração de 3 linhas em `render3d.rs`: remove o base vertex do draw da máscara de blur, porque os índices já endereçam o buffer inteiro desde #470. O autor descreve o sintoma no corpo: "strand pieces after a model's first mesh read the wrong vertices in the blur's mask". |

## Cruzamento com o catálogo

- **Nenhum D-xx do catálogo** casa com o hash: `2d9c230d` não aparece nas colunas `introduzido`/`corrigido`, e `790a3c67` (o commit que introduziu o erro, #470 "Map 3D: draw neighbouring meshes of a model as one range", mesclado às 07:02:25Z) também não.
- **Candidato a linha nova**: defeito introduzido por `790a3c67` (#470) e corrigido por `2d9c230d` (#474), no mesmo dia e dentro de duas execuções da rotina — o intervalo é o próprio cursor das 07:00. Sintoma: peças de cabelo depois do primeiro mesh do modelo leem vértices errados na máscara de blur, porque `draw_indexed` recebia um base vertex sobre índices absolutos.
- Não abre defeito novo: o diff só remove a soma dupla; `first` continua derivado do intervalo do index buffer.

## Confiança

Verificado: o patch completo do commit foi lido (é uma única hunks de 3 linhas em um arquivo), e `merged_at`/`merge_commit_sha` de #474 e de #470 vêm da API de pulls. A ligação causal ("introduzido por #470") é a que o próprio corpo do #474 declara; o diff de #470 não foi relido nesta hora.

Não verificado: nenhum build ou execução do SixNG foi rodado — a correção de leitura de vértice não foi reproduzida em runtime. A janela de PRs abertos é trivial (0 novos acima do cursor 472).
