# holdmysocks/SixNG — atualização de 2026-11-27 22:00

## Resumo

Oito squash entraram na `master`, quase todos no viewer/exportador do jogo: perfis de subsuperfície da pele
lidos do arquivo de perfil do motor, larguras de fio de cabelo das rampas do jogo, textura BC7 por tile de
água interior, tatuagens de braço único no quarto conjunto de UV, culling por mesh para modelos de poucas
instâncias, leitura de adesivos de livery fora da thread de UI, 15 nomes de constante e a força de detalhe
de pele por ped. Nenhum é correção de defeito catalogado; #485 e #488 são os únicos com forma de correção.

## Commits novos (8)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `d8101f48` | Brandon Villar | Skin translucency: the game's subsurface profiles, shaded and exported (#483) | `skinspec.rs` (+165): decodifica o arquivo de perfis de subsuperfície (`SssProfile`) — PSO de `data.rpf`, struct raiz `a0ec2837`, registros `73bc6e91` com `name`, `scatterColor`, `scatterDistance`, faixa de espessura, expoente e enum de tipo; a tabela do motor recebe esses registros pela slot 1 (slot 0 é default vazio). Como a slot que o draw de um ped escreve é escolhida em tempo de execução e não foi rastreada, pele de ped usa o perfil `skin_default`. Emite a espessura `s = lo + (hi-lo)(1-t)^p` e a transmitância por canal, mais textura e extras no GLB. |
| `01f50f8e` | Brandon Villar | Hair export: strand widths from the game's parameter ramps (#482) | `hairexport.rs` (+55): a largura de cada ponto do fio (`_WIDTH`) passa a vir das tabelas de fio do jogo — curva de largura/espessura da rampa do parâmetro da variante (`slot_params`) — quando o conjunto de parâmetros do slot é conhecido; senão cai na largura e no taper guardados no groom. `slot_param_set()` escolhe o conjunto que o asset de cabelo nomeia (`hairsets`) e, na falta dele, os defaults `Long_HighGloss_000`, `Eyebrows_MidGloss_000`, `Eyelashes_HighGloss_002`. |
| `102a534a` | Brandon Villar | water: export each inland tile's BC7 texture (slot e0548ad3) (#484) | `water.rs` (+13/-5): acrescenta `tile_tex` ao `InlandTile` e exporta a textura BC7 do slot sem nome `e0548ad3` por tile interior; `height_texture_check` e o caminho de export passam a incluí-la. README atualizado. |
| `b8d125f4` | Brandon Villar | Names: 15 more shader constant names (SIX-68) (#486) | `names.rs`: 15 nomes derivados por busca prefixo+palavra+sufixo sobre as 748 constantes ainda sem nome (2026-11-27), mantidos só os hits cujo shader casa (fur, grass, terrain, water, ped wrinkle, cutout, slots de textura). |
| `77c5fbc6` | Brandon Villar | Map 3D: cull the meshes of few-placement models one by one (#485) | `render3d.rs` (+134): `GpuMesh` ganha `radius` (raio em espaço de modelo); `GpuModel` guarda `inst_cpu` (transformações e maior escala) quando tem no máximo `CULL_MAX_INST = 8` instâncias. Novo `MeshCull::View` monta os quatro planos laterais de `view_proj` (sem near/far — comentário diz que esfera atrás da câmera falha nos lados e o far é infinito) e `MeshCull::Cascade` reaproveita `shadow::in_cascade`; `MESH_CULLED` conta os meshes pulados em passe principal, fallbacks de cobertura e cascatas. `map3d.rs` passa a usá-lo. |
| `97eca4cb` | Brandon Villar | Liveries panel: read sticker pictures off the UI thread (#488) | `livery.rs` (+71): a leitura de cada imagem (arquivo grande ou textura do jogo ainda fora do cache) vai para uma thread própria — `reading`/`reads`/`read` (`Arc<Mutex<Vec<(String, u64, Read)>>>`) e uma lista `arrived` que o painel consome; `picture_size()` passa a devolver `None` enquanto a leitura não termina. `livery_ui.rs` (+35) acompanha o estado. |
| `4acbe12f` | Brandon Villar | Tattoos: upper-body decorations in its fourth UV set, one arm at a time (#489) | `peddeco.rs` (+207): o corpo superior não é tatuado no UV0 da peça (esse UV é espelhado: os dois braços dividem a mesma ilha, o que não permite tatuar um braço só). O quarto conjunto de UV da peça tem o mesmo layout na metade da largura (`u0/2` e `0.5 + u0/2`) e as decorações são colocadas com `u` dobrado; o bake passa a usar esse `Space::BodyUpper` e `BAKE_RULE` sobe de 6 para 8. Evidência citada: 80 % da área de um decalque de corpo superior cai no corpo nesse espaço, contra 60 % no UV0. Pernas não são espelhadas e continuam no UV0. |
| `9346d0e6` | Brandon Villar | Body skin: the game's per-ped detail strength (distance fade) in the docs and extras (#492) | `skinspec.rs` (+29/-3): dois escalares do registro de instância do ped (`3b316b01`, `per_ped_detail_json`) passam a modular os termos de detalhe — `+0x74` escala cor e normal e desvanece com a distância, `1 - saturate((dist-6)/3)` (6–9 m; 20–21 m quando o byte `+0x22b` do dono tem o bit 0), e `+0x30` escala o blend de tiling da normal, 1 a menos que os bits 1 ou 2 daquele byte estejam ligados. `shader.wgsl` acompanha. |

## Pull requests abertos (8)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 497 | Skin detail: keep the sheet filtering inside each tile | Manter a filtragem da folha de detalhe dentro de cada tile (evitar sangramento entre tiles). Continuação de #492. Lido por título; patch não inspecionado. | NAO VERIFICADO |
| 496 | Liveries: stickers on vehicle glass (zones from _COLOR0) | Adesivos em vidro de veículo, zonas tiradas de `_COLOR0`. Área de D-12/D-13 (cores/tint por peça). | NAO VERIFICADO |
| 495 | Water: place the ocean maps on the water grid | Colocar os mapas de oceano na grade de água. Continuação de #484. | NAO VERIFICADO |
| 494 | Hair export: settle the guides against the body, as the viewer does | Assentar as guias de cabelo contra o corpo como o viewer faz. Continuação de #482. | NAO VERIFICADO |
| 493 | Link subtitles to their speaker and audio stream | Ligar legendas ao falante e ao fluxo de áudio. | NAO VERIFICADO |
| 491 | Map 3D: cull decal and see-through meshes one by one too | Estender o culling por mesh (#485) a decalques e meshes translúcidos. | NAO VERIFICADO |
| 490 | Name 59 more enum values from their named neighbours | Mais 59 valores de enum nomeados pelos vizinhos já nomeados. Continuação de #486. | NAO VERIFICADO |
| 487 | feat(metal4): add configurable MSAA with resolve | MSAA configurável com resolve no backend metal4. Único PR aberto de autor externo (`maple-quartz18`). | NAO VERIFICADO |

## Pull requests mesclados (8)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| 483 | Skin translucency: the game's subsurface profiles | `d8101f48` | Leitura do arquivo de perfis, `SssProfile`, espessura e transmitância, export no GLB. |
| 482 | Hair export: strand widths from the game's parameter ramps | `01f50f8e` | `_WIDTH` das rampas de parâmetro e escolha do conjunto por asset/default. |
| 484 | water: export each inland tile's BC7 texture | `102a534a` | Slot `e0548ad3` no `InlandTile` e no export. |
| 486 | Names: 15 more shader constant names | `b8d125f4` | 15 nomes derivados. |
| 485 | Map 3D: cull the meshes of few-placement models | `77c5fbc6` | `radius`, `inst_cpu`, `MeshCull`, contadores `MESH_CULLED`. |
| 488 | Liveries panel: read sticker pictures off the UI thread | `97eca4cb` | Leitura assíncrona das imagens de livery e estado `arrived`. |
| 489 | Tattoos: upper-body decorations in its fourth UV set | `4acbe12f` | `Space::BodyUpper`, `BAKE_RULE` 6→8, bake por quarto UV. |
| 492 | Body skin: per-ped detail strength and distance fade | `9346d0e6` | Escalares `+0x74`/`+0x30`/`+0x22b` no detalhe de pele. |

Todos são squash de um pai; o `merge_commit_sha` de cada PR é o próprio commit.

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum. Nenhum dos oito commits fecha uma entrada do catálogo deste repositório.
- **D-02 (TOC do RPF9 nunca lido; nomes por carving):** `d8101f48` descreve o arquivo de perfis como "um PSO
  de `data.rpf` que só o índice alcança (`toc-extract`)". É a primeira vez que um commit deste repositório
  cita leitura pelo índice, mas o commit não diz que o TOC do RPF9 passou a ser lido nem mexe em
  `rpf`/`toc`; a afirmação fica como área de D-02, **NAO VERIFICADO** como correção.
- **D-11 (valores de parâmetro de material no offset errado):** `9346d0e6` fixa offsets novos de registro
  de instância (`+0x74`, `+0x30`, `+0x22b`) e uma regra de fade com dois limiares; é a mesma classe de
  risco (offset inferido), aqui com a origem declarada (registro `3b316b01`, cópia do objeto dono em
  `+0x20`). Não é D-11, que trata do offset dos parâmetros de material.
- **D-23 (UV de ground em 64 por unidade em vez de 128):** `4acbe12f` mexe em espaço de UV, mas de
  decoração de corpo (quarto conjunto, `u` dobrado) — não toca o UV de ground.
- **Abre defeito novo?** Dois pontos de atenção, nenhum verificado em execução:
  1. `77c5fbc6`: `MeshCull::View` usa só os quatro planos laterais. Uma esfera grande com centro atrás da
     câmera mas parte visível à frente passa nos quatro planos? Pela conta `dot(c) + w >= -r`, um centro
     atrás do plano lateral mais próximo falha; o risco real é o oposto — esfera que cruza o near plane
     sendo descartada se o centro estiver além de um plano lateral, e o comentário assume que "esfera
     atrás da câmera falha nos lados", o que não vale para toda posição. O raio vem do modelo, então uma
     instância com escala maior que 1 usa `maior escala` (`inst_cpu`), mas só para ≤ 8 instâncias.
  2. `97eca4cb`: a leitura assíncrona devolve resultados por `Arc<Mutex<Vec<...>>>` com um contador `reads`
     por chave; não há, no diff, limite de threads nem descarte de resultado de uma leitura que o painel
     já substituiu. Lido no diff de `livery.rs`; race não reproduzida.

## Confiança

**Verificado por diff:** os oito commits foram baixados e lidos por arquivo e patch; os trechos citados
(`skinspec.rs`, `hairexport.rs`, `peddeco.rs`, `render3d.rs`, `livery.rs`, `water.rs`, `names.rs`) foram
lidos na íntegra ou até o ponto que decide o comportamento. As oito entradas de PR mesclado batem com os
oito commits por `merge_commit_sha` (`parents` = 1 em todos).

**Não verificado:** os oito PRs abertos foram lidos por título e base, não por patch; nenhum teste do
repositório foi executado; nenhuma alegação de qualidade visual (sangramento, água, tatuagem) foi
conferida em imagem. A `default_branch` foi reconferida por API (`master`); o head documentado é
`9346d0e61be1d8299d19f7151a8148c9db95f8b5`.
