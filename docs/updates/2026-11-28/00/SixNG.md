# holdmysocks/SixNG — atualização de 2026-11-28 00:00

## Resumo

Dois commits acima do cursor `ede2c253`, ambos squash de um pai casando 1:1 com os merges `#503` e `#504`,
e dois PRs abertos. O `#503` traz o vidro das lentes de pedestre: registro de transmissão por
sub-material lido da biblioteca de materiais e o blend dual-source do jogo no visualizador. O `#504`
conserta a lista de adesivos: registro cuja textura não existe em nenhuma store do jogo deixa de ser
oferecido, e uma livery sem adesivo posicionável não apaga mais o que está no veículo.

## Commits novos (2)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `ff17c3f7ce` | Brandon Villar | Ped lenses: glass sub-material per vertex and the game's tinted-glass blend (#503) | 11 arquivos, +336/−8. `sixng-core/src/matlib.rs` (+107) ganha `Transmission` (absorção σA rgb + wA, espalhamento σB rgb + wB, `d_base`, `d_mask`, `d_max`, anisotropia `p`, âncora `h0`, divisor de alfa `k`, limiar opaco `t`, id do perfil) e `Library::transmission(preset, sub)`, que escolhe o perfil do sub-material, senão o do material base, senão o registro todo-em-defaults. `fabric.rs` (+47/−4) guarda a textura `material_map` índice 3 como `cover` (alfa do tile), amostra em UV com escala/rotação do sub-material, e o bake passa a receber `by_vertex` (materiais sem `materialid` próprio, como a lente que compartilha a textura da armação) e `glass`. `model_view.rs` (+24/−1) escolhe, para cada material com `shader_hash == "1c43a65b"` e `alpha_mode == "BLEND"`, **o sub-material que a maioria dos vértices nomeia** em `(v.col[1] >> 2) & 7` (desempate pelo menor índice) e grava o registro no material. `main.rs` pede `Features::DUAL_SOURCE_BLENDING` ao adaptador. `render3d.rs` (+74/−2) acrescenta `MatU::glass[[f32;4];4]`, `GpuModel::mat_glass` e `pipe_glass`, construído só quando `hdr == false` **e** o device tem dual-source blending; o blend é `src_factor One` / `dst_factor Src1`, e `glass.wgsl` (+26) escreve `col` e `trans` como duas fontes (`@blend_src(0)`/`(1)`) com o blend do jogo `dst' = src0 + dst × src1`. `shader.wgsl` (+43) calcula em `mesh_colour`: `a0` = alfa do material map do tile × alfa do speculartex (assado no alfa da base), `alpha = sat(a0/k)`, `o = sat((a0−t)/(1−t))`, `s_c = σA_c(1−b_c·wA) + σB_c(1+(b_c−1)·wB)`, caminho de vista `d = max(0, d0 + |p|·(min(d_max, r·d0) − d0))` com `r = 1/c` se `p>0` senão `c`, `c = max(1e−4, |N·V|)`, `T = exp(−s·d)`, `RT1 = (1−o)(1−alpha + alpha·T·(1−alpha·F))`, `RT0 = o·(luz difusa) + alpha·(luz especular)`. Nas views de depuração (`mode.x != 0` e `!= 7`) cai no alfa simples |
| `1b419021e8` | Brandon Villar | Liveries panel: leave out stickers without a picture; keep stickers when a game livery places none (#504) | 1 arquivo, +18/−7, em `livery_ui.rs`. `GameSticker` ganha `missing: bool` (`serde(default)`), o cache da lista sobe de `sticker_list_v1.json` para `v2`, e `read_list` consulta `sixng_core::ttpf::global(game)`: registro cujo id de textura existe mas não está em nenhuma store é mantido e marcado `missing`. O filtro da lista passa a exigir `!s.missing` e a linha de status informa quantos registros foram deixados de fora por não terem textura em store alguma. No `place`, livery cujo conjunto de adesivos fica vazio **não** mexe no veículo: escreve "none of its stickers can be placed, so the stickers on the vehicle were kept" e retorna |

## Pull requests abertos (2)

Acima do cursor `last_pr_number` 500; título e corpo lidos, patch não lido.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| `#506` | refactor(metal4): extract egui painter crate | Extrai o painter egui privado do Metal 4 para o crate reutilizável `egui-mtl4`, troca o módulo privado pela dependência sob `metal4-poc`, substitui a checagem linear de texture id por `Painter::set_linear_texture_ids` e documenta licença GPL-3.0-only, pin do fork Metal-Rust (`0aa2a1f6`) e limites | Não; refatoração |
| `#505` | Names: 18 more shader constant names (SIX-68) | 18 nomes de constante de shader a mais em `names.rs`, obtidos por busca de palavra acrescentada a nome conhecido; o corpo discute o ruído esperado da busca (0,6 e 0,8 acertos aleatórios) | Não |

## Pull requests mesclados (2)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| `#503` | Ped lenses: glass sub-material per vertex and the game's tinted-glass blend | `ff17c3f7` (2026-11-27T20:23:12Z) | O `Transmission` e o `Library::transmission`, a textura `cover`, a escolha do sub-material por maioria dos vértices, o pedido de `DUAL_SOURCE_BLENDING`, `pipe_glass` e o blend dual-source, e o cálculo de transmissão em `mesh_colour`. O commit é o merge (squash) |
| `#504` | Liveries panel: leave out stickers without a picture; keep stickers when a game livery places none | `1b419021` (2026-11-27T20:41:30Z) | O campo `missing`, o cache v2, a consulta às stores e o retorno sem alterar o veículo quando nada é posicionável |

## Cruzamento com o catálogo

- **Nenhuma das 24 entradas de `SixNG` é corrigida por estes dois commits.** O `#503` é a mesma **classe**
  de D-12 (carroceria do veículo aplicada a interior, carcaças de lâmpada e peças mecânicas) e D-13 (tint
  de prop em shader de veículo): escolher o sub-material errado para um grupo de materiais. Aqui a escolha
  é explícita e por vértice, mas **não corrige D-12/D-13**, que são atribuição de material no carregador.
- **Aproximação registrada no próprio código.** O registro de transmissão é **por material**, escolhido
  pelo sub-material que a maioria dos vértices nomeia (`model_view.rs`, laço sobre `v.col[1] >> 2 & 7` com
  `max_by_key((count, 8 − s))`). Um material cujos vértices nomeiem sub-materiais diferentes — lente com
  armação na mesma malha, por exemplo — usa um registro só para todos os fragmentos. O código diz "the
  sub-material most of its vertices name"; não há teste no diff que exercite o caso dividido.
- **O caminho Metal 4 não consome o registro.** `render3d_metal4.rs` preenche `glass: None` nos dois
  literais de `MatU` que o diff toca, e `pipe_glass` só existe no renderer wgpu. É a mesma forma do achado
  da hora anterior (`e607e6d3`, detalhe de pele sem efeito no Metal 4): um switch novo que só vale num
  renderizador. NAO VERIFICADO se a build Metal 4 desenha a lente com `pipe_blend` ou sem blend.
- **Blend limitado ao renderer não-HDR.** `pipe_glass` é construído com `!hdr && DUAL_SOURCE_BLENDING`;
  `new_map` (o renderer HDR do mapa 3D) fica com `pipe_blend`. Como só o visualizador de modelo grava
  registros de vidro, o efeito prático é o esperado, mas o pipeline não existe no mapa.
- **`#504` mexe na lista de adesivos, área de D-20** (expressão da lista de cores do sticker, defeito de
  branch que nunca chegou ao master). O diff não toca a expressão de cores: filtra por textura ausente e
  preserva o veículo. Não é D-20.

## Confiança

- **Verificado no diff:** os dois patches foram lidos por inteiro em todos os arquivos (11 e 1). O
  consumo de `missing` foi conferido no arquivo em `1b419021` (filtro da lista e contagem na linha de
  status, `livery_ui.rs` linhas 1204–1211), e os construtores de `Renderer3D` foram lidos em
  `render3d.rs` (`new`/`new_viewer` com `hdr = false`, `new_map` com `hdr = true`).
- **Verificado por API:** `#503` e `#504` são squash cujo `merge_commit_sha` é o próprio commit; os dois
  PRs abertos estão acima do cursor 500.
- **Não verificado:** nenhum shader foi compilado nem rodado; os valores de transmissão vêm de "shared
  notes 42es" citadas no código, não de uma captura conferida aqui; o caso de material com sub-materiais
  divididos não foi exercitado; nenhum patch de PR aberto foi lido. O `#503` não traz teste novo (os
  arquivos `cells.rs`, `render3d_metal4.rs` e `world3d.rs` só acrescentam `glass: None` em literais de
  teste/estrutura).
- **Contagem:** 2 commits na faixa = 2 merges na janela. Coerente.
