# holdmysocks/SixNG — atualização de 2026-11-26 00:00

## Resumo

Seis commits na `master` (`addb0e79` → `f4c9262a`), todos merges de PR do próprio autor na mesma hora:
resolução das texturas de-duplicadas (pele pintada de 2.602 personagens), dois consertos de sombreamento
de pele, os conjuntos de fio de cabelo por asset, seis slots novos na tabela BC1 do BCPack, um conjunto
para os 26 modelos sem dados de outfit e o conjunto de UV da cabeça na exportação de grooms. Sete PRs
abertos, quase todos na frente do mapa 3D.

## Commits novos (6)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `c14ea82c` | Brandon Villar | Textures: resolve de-duplicated textures (painted body skins for 2,602 characters) (#332) | `ttpf.rs` (+86): `TtpfSet` ganha `root`/`dupes` e `parse_dupes()` lê a tabela `EDUP` de `textures.rpf` (magia `EDUP`, u32 contagem, pares `(u64 dropped, u64 kept)` ordenados pelo dropped, filler depois; 17.431 pares na instalação de referência). `TtpfSet::get` cai na tabela quando o hash não está em nenhum store, `get_stored` mantém a busca exata. `export.rs` grava `stored_as` no `model.json` quando a textura veio por essa via, e o PNG leva o nome do hash guardado. Exemplo `bodyskin.rs` novo (+107) com os subcomandos `item`, `dupes` e `survey`. README documenta. |
| `4f16c5a3` | Brandon Villar | Skin shading: no pale band under the jaw, no seam at the pelvis (#308) (#334) | `shader.wgsl` (+6): a parcela de ambiente deixa de ser Fresnel aproximado e passa a usar a tabela de BRDF de ambiente do jogo — `r (F90 - F0) + g F0` em (N·V, roughness), vezes o ganho multi-scatter `1 + (1/g - 1) F0`; o texto diz que em superfície rugosa vista de perfil, como pele sob o maxilar, a estimativa antiga dava ~2,5x o valor da tabela. `glb.rs` (+13): `Mesh::region2` carrega o segundo byte do stream `_REGION`, a camada 2 que as duas metades da pele compartilham (a faixa da cintura); `model_view.rs` (+11) usa isso para cada metade esconder a cópia da outra, e cabeças sem stream de tangente passam a ter frame tangente reconstruído. |
| `e34ebade` | Brandon Villar | Hair: each asset's own strand parameter and colour sets (#335) | `hairsets.rs` novo (+89): cada asset de cabelo/pelo declara nas propriedades de registro `960033653` (conjunto de cor) e `2259149558` (conjunto de parâmetro) o joaat do nome do conjunto, com cache em `<work>/index/hair_asset_sets.json`. `hairlink.rs` (+15) generaliza `registry()` em `registry_props()`. `model_view.rs` (+35) deixa de usar um default adivinhado por slot e passa a ler o conjunto de cada asset; o default fica só para peça cujo asset não é conhecido. Exemplo `hairsets.rs` (+42) confere os 1.421 assets contra as tabelas do jogo. |
| `b7a9ad07` | Brandon Villar | BC1 BCPack: 6 preset window slots from a third mip-consistency round (#337) | `bc1pack_tables.rs`: `OVERRIDES` passa de 44 para 50 entradas, acrescentando os slots 6, 105, 110, 119, 141 e 148. A checagem de mip sobre as 35.533 texturas BC1 BCPack vai de 65 para 60 abaixo de 20 dB e de 474 para 460 abaixo de 25 dB; 44 texturas melhoram mais de 1 dB e nenhuma piora. Desde o #321 o número abaixo de 20 dB caiu de 138 para 60. BC6H fica intocado porque a mesma regra relaxada piorou sete texturas lá. |
| `006565ee` | Brandon Villar | ped-parts: a set for the 26 models with no outfit data (their own drawables) (#340) | `pedset.rs` (+53): `pedset::of` tenta `pedlook::look` e, se falhar, cai em `own_set`, que lista cada drawable do próprio modelo (1 a 8 por modelo) sem outfits. Os conjuntos vêm marcados como incompletos e com nota do motivo, para a contagem `complete` continuar 753 de 779 — antes 26 modelos paravam com erro ("not in the game's outfit list, and no wardrobe names its head"). Teste `own_set_lists_the_models_own_drawables`. |
| `f4c9262a` | Brandon Villar | Hair export: head UV set in grooms --full and the viewer's groom export (#342) | `hairexport.rs` (+23): `tag_uv_set()` grava `head_mesh`, `head_uv_set` e `head_uv_error` nas extras do groom quando as UVs de raiz casam com um conjunto de UV da cabeça. `groom.rs` (+9) passa o `head` adiante em `export_grooms`, o CLI ganha `--head <model.glb>` (+14) e o botão "Export shown grooms" faz o mesmo contra a cabeça do personagem exibido. `scripts/blender_hair_curves.py` (+14) passa a aceitar vários arquivos de cabelo (um por groom). O README explica que o conjunto não se deduz do tipo de peça: em cinco personagens a maioria das peças de couro cabeludo está no set 3 e as de rosto no set 2, mas algumas (rabo de cavalo, cabelo longo) estão no set 0. |

## Pull requests abertos (7)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 336 | Map 3D: each place's own water, and the bed through shallow water | Atlas de névoa d'água (scattering/extinction) e mapa de altura carregados em segundo plano e cacheados; a cor do corpo d'água vem do atlas no lugar do valor único de timecycle, e a profundidade é a superfície menos o terreno (mar usa 4 m abaixo de 0). Duas texturas half-float novas (bindings 24 e 25), `water.rs` +128, `shader.wgsl` +119. SIX-211. | Não. |
| 338 | Map 3D screenshots: don't count a view as loaded before the map data is in | Uma view só conta como carregada quando os dados do mapa (e a pirâmide de altura, com terreno visível) entraram; antes, fila vazia passava por loaded e metade dos runs de screenshot saía com 0 modelos e mar chapado (`map3d.rs` +6). | Não. É falha de harness de captura. |
| 339 | Map 3D: the weather's clouds and their shadows | Camada de cúmulos entre `local_atmosphere_cloud_height` e `_top` (200 e 1500 m), cobertura por ruído derivado da velocidade do vento, profundidade óptica em quatro amostras, luz do sol/lua através da nuvem e sombra projetada na superfície. `atmo.rs` +21, `shader.wgsl` +127. | Não. |
| 341 | Map 3D: made-up ground for the land no map tile covers | Células de terra sem tile de mapa deixam de ser verde chapado: chão sintético por altura, inclinação e ruído (areia logo acima do mar, capim seco e mato no interior, bosque em manchas, rocha em declive), com normais próprias. Cores e escalas de ruído (380, 60 e 9 m) são escolha do autor, não decodificadas. `shader.wgsl` +41. | Não. |
| 343 | Map 3D: GPU frame time by part in the tick lines | Cinco timestamps por frame (início, tabelas de céu, cascatas de sombra, passe principal, post) quando o adaptador suporta timestamp dentro do encoder, lidos alguns frames depois; as linhas de tick ganham `gpu ms/frame sky\|shadows\|main\|post`. `SIXNG_MAP_TICKS=1` mantém os ticks depois do shot. Nada muda no que é desenhado. `gpu_time.rs` +132. | Não. |
| 344 | Hair export: strand colours (COLOR_0) and a coloured Blender material | Cada peça crescida leva a cor que o viewer dá ao slot (conjunto da roupa, senão o do item, senão os presets do próprio item); `COLOR_0` linear por ponto ao longo da rampa de variante da raiz à ponta, e as extras nomeiam `colour_set` e `colour_source`. O script Blender lê `COLOR_0` num atributo de ponto e monta Principled Hair BSDF em parametrização de cor. Constrói sobre `#342`. +226/-64 | Não. |
| 345 | Hair: drawn strands keep the piece's cover under the count rule | A regra de contagem desenha uma fração das hastes conforme a distância (cerca de 7 % num plano de corpo inteiro); o jogo desenha essa fração na espessura-alvo, então a peça mantém a cobertura, enquanto o viewer desenhava as hastes mantidas na largura original e perdia cobertura. Cada material passa a carregar um fator de largura (armazenadas/desenhadas, teto 50) aplicado antes da regra de largura mínima. Corrige o cabelo "nuvem pálida esparsa com couro cabeludo à mostra" relatado na Lucia, que armazena 1.420 e 4.773 hastes contra 16.031 da Mandy. `render3d.rs` +18, `shader.wgsl` +3 | Não é D-xx; defeito novo descrito no PR. |

## Pull requests mesclados (6)

Todos squash: `mergeCommit.oid` é o próprio commit da tabela de commits.

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| 332 | Textures: resolve de-duplicated textures | `c14ea82c` | Tabela `EDUP` de `textures.rpf` lida (17.431 pares) e usada como fallback de busca; `stored_as` no `model.json`. |
| 334 | Skin shading: no pale band under the jaw, no seam at the pelvis | `4f16c5a3` | Tabela de BRDF de ambiente do jogo no lugar do Fresnel aproximado, camada 2 de região na pele e frame tangente para cabeças sem stream de tangente. |
| 335 | Hair: each asset's own strand parameter and colour sets | `e34ebade` | Propriedades `960033653`/`2259149558` lidas por asset, com cache; 171 conjuntos de parâmetro em uso segundo o PR. |
| 337 | BC1 BCPack: 6 preset window slots | `b7a9ad07` | `OVERRIDES` 44 → 50; abaixo de 20 dB 65 → 60 nas 35.533 texturas BC1. |
| 340 | ped-parts: a set for the 26 models with no outfit data | `006565ee` | `own_set` com os drawables próprios, marcados incompletos; `ped-parts` cobre 779 de 779. |
| 342 | Hair export: head UV set in grooms --full and the viewer's groom export | `f4c9262a` | `head_uv_set` por groom, `--head` no CLI, botão do viewer e script Blender com múltiplos arquivos. |

## Cruzamento com o catálogo

- Corrige D-xx? Nenhum dos 24 itens, mas dois ficam na mesma vizinhança:
  - `D-09` ("BC1: seis dos nove slots de janela substituídos eram piores que o default", corrigido em
    `b7fbe3a`) é a tabela que o `#337` (`b7a9ad07`) volta a ajustar. Aqui a régua é diferente: só entra
    slot que melhora a checagem de mip, e o PR publica os números (abaixo de 20 dB 65 → 60; nenhuma
    textura pior). Não é reabertura do defeito.
  - `D-01` ("recurso RAGE dividido em duas entradas de índice", corrigido em `b7fbe3a`) e o `#332` mexem
    em identidade de recurso, mas por motivos distintos: `D-01` era o indexador partindo uma página em
    duas; `#332` é textura guardada uma vez sob outro hash e referenciada por tabela.
- Abre defeito novo? Não evidente. O risco maior é `#334`: mexer em frame tangente de cabeça sem stream
  de tangente mexe exatamente na região de `D-03` (normal map em frame tangente errado, corrigido em
  `839e852`) e de `D-10` (ordem de eixos). O diff não cita nenhum dos dois, e não há comparação contra
  frame do jogo nesta hora.

## Confiança

Verificado por diff: os seis commits lidos em `gh api .../commits/<sha>`, com pai em cadeia
(`addb0e79` → `c14ea82c` → `4f16c5a3` → `e34ebade` → `b7a9ad07` → `006565ee` → `f4c9262a`). `mergeCommit`
de cada PR conferido contra o commit documentado. PRs abertos lidos por `gh pr view`.

Não verificado: as contagens que os PRs declaram (2.602 personagens, 17.431 pares, 35.533 texturas,
1.421 assets, 779 modelos) vêm dos corpos e dos comentários de código, não de execução aqui. Os PRs
abertos 336, 339, 341 e 344 declaram `cargo check` limpo e screenshots em GPU pendentes na segunda
máquina — o estado de validação deles é o que o texto diz, não um resultado observado.
