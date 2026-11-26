# holdmysocks/SixNG — atualização de 2026-11-26 19:00

## Resumo

A faixa desde `4859e1ff` tem **48 commits**; os **40 mais antigos** foram documentados e os 8 restantes ficam no
`commit_cursor`. É um dia de ferramenta de conteúdo: exportação de cabelo com cor por fio (`COLOR_0`), elenco de
personagens de cutscene, primer do veículo lido da paleta de material, refinamento das tabelas BCPack BC3/BC4/BC5,
mapa 3D (terreno, nuvens, água, células, tempo de GPU), exportação de roupa com máscaras de esconder, e um par
introduzido-e-revertido no mesmo dia (nuvem pré-cozida em textura repetida, `cca88805` → `444327fe`).

## Commits novos (40 documentados de 48)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| f96b3f08 | Brandon Villar | Hair export: strand colours (COLOR_0) and a coloured Blender material (#344) | `hairexport.rs` ganha `HairColour` (`Set` das quatro rampas de `hairtone::StrandData`, ou `Presets` do item) e `slot_colour`, que escolhe o conjunto pela categoria do slot (`SLOT_CATEGORY`) e cai no preset do item; `gltf.rs` emite `COLOR_0` por ponto; `scripts/blender_hair_curves.py` monta material Principled Hair com a cor elevada ao quadrado por nó Gamma. |
| 6694896a | Brandon Villar | Cutscene characters: clothing sets on the model that wears them, names confirmed by scene peds, outfit for models without own parts (#306) (#349) | `pedlook.rs` (`CACHE_VERSION` 4→5) ganha o mapa `parts`: nome de drawable de prefixo fora da lista de outfits → recursos de wardrobe que o nomeiam, achado varrendo os blobs em passos de 8 bytes; o CLI passa a exportar outfit 0 quando o modelo não tem peças próprias (`cs_mandy`) e a aceitar prefixos que vestem outro modelo (`cs_roxy`). |
| 30202d89 | Brandon Villar | Viewer: the primer's colour from the material palette (#350) | `vehcolour.rs` (`EXPORT_VERSION` 28→29, `FORMAT` 23→24) acrescenta `primer` (três linhas de paleta por conjunto 0..2, tabela `PRIMER_ENTRIES` por `primerType`) e `primer_set` no look; `shader.wgsl` sobe `RAMP_ROWS` de 1025 para 1281 e amostra a cor do primer na janela `primerGradientStart/End` em vez da constante cinza `PRIMER_COLOUR`. |
| ef255ad9 | Brandon Villar | BC3/BC4/BC5 BCPack: 44 preset slots fitted on mip consistency (#352) | Só tabelas: 43 entradas de `BC4_PRESET` em `bc45pack_tables.rs` e uma em `bc3pack_tables.rs` (slot 226) reajustadas para consistência entre mips. |
| c06f5b97 | Brandon Villar | Hair: each strand its own coverage dither; curls per guide; count rule in 1440 lines (#353) | `groom.rs` (+7/−5) muda a contagem/ondulação dos fios, `shader.wgsl` dá a cada fio seu próprio dither de cobertura, e `render3d.rs`/`glb.rs` acompanham o atributo. |
| 20d1d191 | Brandon Villar | Map 3D: the land drawn as made-up ground, with the tiles' relief (#341) | `shader.wgsl` +70: o terreno do mapa 3D passa a ser desenhado como chão sintético, usando o relevo dos tiles em vez da textura aérea. |
| d187e24e | Brandon Villar | Map 3D: GPU frame time by part in the tick lines (#343) | Novo `gpu_time.rs` (132 linhas) mede o tempo de GPU por parte e o imprime nas linhas de tick, ligado por `map3d.rs`/`render3d.rs`. |
| 7943fda9 | Brandon Villar | Viewer: clothes under clothes drawn as their _cull_00 twins and cut by the hide masks (#354) | `model_view.rs` +41/−11: peças sobpostas são desenhadas pela variante `_cull_00` e recortadas pelas máscaras de esconder, em vez de ficarem ocultas. |
| 54f96b2d | Brandon Villar | Hair export: scalp hair pressed under the outfit's hat (#348) | `groom.rs` +206 e `hairexport.rs` +119: cabelo do couro cabeludo é comprimido sob o chapéu do outfit (detecção de chapéu, deslocamento dos fios), com `model_view.rs` aplicando o mesmo na prévia. |
| 51b0a7ae | Brandon Villar | Head skins: cs_ heads without a face preset take their character's face (cs_lucia) (#355) | `headskin.rs` +77: cabeça `cs_` sem preset de rosto herda o rosto do personagem correspondente, com fallback explícito. |
| 0a850d5b | Brandon Villar | Body shape: full rig program; clothing helper bones follow the garment surface rig (#308) (#356) | `bodyshape.rs` +462/−159: programa completo de rig (ossos de ajuda da roupa seguem o rig da superfície da peça), com `export.rs` carregando os dados no GLB. |
| 72303f5b | Brandon Villar | Map 3D: the weather's clouds and their shadows (#339) | `shader.wgsl` +158 e `atmo.rs` +21: nuvens do clima e suas sombras no mapa 3D, alimentadas pelo estado de atmosfera. |
| 0cc6f6c4 | Brandon Villar | Map 3D: map cells draw from 0.3 of the HD distance (25 to 44 fps downtown) (#351) | `map_cells.rs` +16: as células do mapa passam a desenhar a partir de 0,3 da distância HD, reduzindo o custo no centro (número de fps no assunto, medido pelo autor). |
| 58d22b38 | Brandon Villar | Outfit export: clothes under clothes as their _cull_00 twins, skin and clothes cut by the hide masks (#358) | `pedlook.rs` +55, `outfit.rs` +17, `main.rs` +19: a exportação aplica a mesma regra da prévia — peça sobposta vira `_cull_00`, pele e roupa recortadas pelas máscaras de esconder. |
| 99fae714 | Brandon Villar | BC4/BC5 BCPack: 6 preset slots refined in a second mip-consistency round (#359) | Só `bc45pack_tables.rs` (+6/−6): seis entradas de preset reajustadas numa segunda rodada. |
| fc530fd5 | Brandon Villar | Map 3D: each place's own water, and the bed through shallow water (#336) | Novo `water.rs` +128 no core e +189 em `shader.wgsl`: água por local (cor/profundidade) e leito visível sob água rasa; `atmo.rs`/+133 e `render3d.rs` +82 passam os parâmetros. |
| d0cc7e06 | Brandon Villar | Viewer: the primer's metalness and roughness blended by the material map (#362) | `model_view.rs` +16/−11 e `shader.wgsl` +5/−2: metalness e roughness do primer são misturados pelo mapa de material, em vez de constantes. |
| 1bb8aec4 | Brandon Villar | Viewer: distance test as a backstop under the hide masks; second mask list findings (#363) | `model_view.rs` +15/−11: teste de distância como rede de segurança sob as máscaras de esconder, mais achados da segunda lista de máscaras. |
| 7aa8ef88 | Brandon Villar | Skin shading: the body kept out of its own mirrored surroundings (last jaw rim, #308) (#364) | Só `shader.wgsl` (+42/−1): o corpo deixa de ser excluído do próprio entorno espelhado no sombreamento de pele (último contorno de mandíbula). |
| 94852942 | Brandon Villar | Painted hair: find a hair item's layer set by its own tag, not by its drawable name (#360) | `pedlook.rs`, `pedset.rs`, `wardrobe.rs`: o conjunto de camadas de cabelo pintado é procurado pela tag do item, não pelo nome do drawable. |
| cca88805 | Brandon Villar | Map 3D: cloud noise baked once into a repeating texture (cheaper clouds) (#361) | `atmo.rs` +126: o ruído de nuvem é cozido uma vez numa textura repetida, substituindo o cálculo por pixel em `shader.wgsl` (−15 linhas de ruído). |
| b666920a | Brandon Villar | Hair colliders from the character's body bound (#365) | `fragphys.rs` +64 e `groom.rs` +64: colisores de cabelo derivados do bound do corpo do personagem, com `model_view.rs` usando-os na prévia. |
| 97ac6c2b | Brandon Villar | Body skins: approximate skin for whole-body meshes that no body set names (#366) | `pedlook.rs` +228: pele aproximada para malhas de corpo inteiro que nenhum body set nomeia. |
| ebad7ee6 | Brandon Villar | Cutscene characters: stand-in clothes for the eight head-only characters (#306) (#367) | `pedlook.rs` +159: os oito personagens que só têm cabeça recebem roupa substituta. |
| 97d5cce9 | Brandon Villar | Version 0.11.0 (#368) | Só versão: `Cargo.toml` e `Cargo.lock`. |
| 444327fe | Brandon Villar | Map 3D: back to the procedural cloud noise (revert #361) (#369) | Reverte `cca88805`: apaga as 126 linhas de `atmo.rs` que coziam o ruído e devolve o cálculo no shader. |
| ebf09a0c | Brandon Villar | Map 3D: water-plant tiles and far SLOD tiles no longer draw flat squares near the camera (#370) | `map3d.rs` +25 e `map_cells.rs` +7: tiles de planta aquática e SLOD distantes deixam de desenhar quadrados chapados perto da câmera. |
| bac66cca | Brandon Villar | Decode the BC5 cloud volume: 3D layout for 16-byte blocks and its mip chain (#372) | `gfx10.rs` +37 e `texture.rs` +18: decodificação do volume BC5 de nuvem, com layout 3D dos blocos de 16 bytes e a cadeia de mips. |
| cc73e1a8 | Brandon Villar | Viewer: pick animations from dropdowns instead of typing the name (#371) | Novos `clip_picker.rs` (224 linhas) e `anim.rs` +143: seletor de animações por dropdown, substituindo o campo de texto em `model_view.rs` (−58). |
| b2698c0e | Brandon Villar | Hair colliders: the worn garments' bounds join the body bound (#374) | `model_view.rs` +14/−3: os bounds das roupas vestidas entram no bound usado para os colisores. |
| 4a90d613 | Brandon Villar | Meta dump: fixed-capacity arrays (type 0x53) list every element (#373) | `meta7a.rs` +13: arrays de capacidade fixa (tipo `0x53`) passam a listar todos os elementos no dump. |
| 37f7942b | Brandon Villar | README: the index's size on disk today (#377) | Só `README.md` (+12/−2): atualiza o tamanho do índice em disco. |
| 7cb78e82 | Brandon Villar | Outfit export: --parts hides covered skin and under-layers, like --look (#375) | `main.rs` +33: `--parts` aplica as mesmas regras de esconder pele e subcamadas que `--look`. |
| a0b7fb69 | Brandon Villar | Body skins use the character's own normal map (#378) | `export.rs` +6 e README: a pele do corpo usa o normal map do próprio personagem. |
| 77ec8c5f | Brandon Villar | Outfit export: parts added with --add hide what they cover (#379) | `main.rs` +44/−45: peças adicionadas por `--add` escondem o que cobrem, e o caminho de `--parts` é reorganizado para isso. |
| 492b0400 | Brandon Villar | Export inland water surfaces (rivers, canals, creeks, reservoirs) with `sixng water --inland` (#383) | `water.rs` +241 e novo subcomando `water --inland`: exporta superfícies de água interior (rios, canais, córregos, reservatórios). |
| 7e3eb450 | Brandon Villar | Map bake: read the newer of the two placements exports (#376) | `mapbake.rs` +20: o bake do mapa lê o mais novo dos dois exports de placements. |
| 9de14109 | Brandon Villar | Viewer: Breasts, Butt and region body sliders; outfits open in their body-shape preset (#388) | `bodyshape.rs` +100 e `model_view.rs` +119: sliders de corpo (peito, glúteo, regiões) e abertura do outfit no preset de forma do corpo. |
| 8c6e2312 | Brandon Villar | Animation picker: animal species from the model list, short button text, screenshots wait for the lists (#389) | `anim.rs` +15 e `clip_picker.rs` +28: espécies de animal vindas da lista de modelos, texto de botão curto e captura que espera as listas. |
| 70cab4c5 | Brandon Villar | Map 3D: settle views that want more models than the count cap (#387) | `map3d.rs` +52 e `map_cells.rs` +9: uma vista que pede mais modelos que o teto de contagem assenta em vez de ficar incompleta; `mem.rs` ajusta o contador. |

## Pull requests abertos (9)

Todos os 9 PRs abertos do repositório têm número acima do cursor 350 e estão listados. `last_pr_number` → 402.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 384 | Characters: draw the nipple and genital inserts from the content pack (SIX-229) | Resolve os GUIDs de peças de inserção (mamilo/genital) contra o `core_restofworld`, não só contra `packs.rpf`, para que peitoral e partes íntimas deixem de sair em branco. | Não; fecha SIX-229. |
| 391 | Weapon speculartex2 slot role: wear amount, not a metal mask | Tabela `SHADER_SLOTS` e `slot_role_in` em `material.rs`: em shader de arma, `speculartex2` R é quantidade de desgaste (lerpa cor para rampa de wear, metalness × (1−wear)), não máscara de metal. | Não; corrige papel de slot, sem ID no catálogo. |
| 393 | LOD links: hour masks of time archetypes | `lod-links` passa a ler `CTimeArchetypeDef` (`76b0c56c`) com a máscara de horas em `timeFlags` (`8609d52c`), bit *h* = hora *h*. | Não. |
| 394 | exeschema: meta struct schemas from the executable's parser table | Novo exemplo `exeschema` que lê a tabela de parser do executável e emite, por struct, hash do nome, tamanho, flags, struct base e membros com tipo/subtipo, em JSON. | Não. |
| 398 | refactor(metal4): extract build_vertex_index_bytes for shared use | Refatoração pura: move a montagem de vértice/índice/mesh de `upload_shared()` para `build_vertex_index_bytes()`, pré-requisito de um cache de buffer de modelo pronto para GPU. | Não. Declara não mudar comportamento. |
| 399 | Names: 4,114 meta field, struct and enum names from the executable's parser table | `names_exe.txt` embutido com 4.114 nomes (field/struct/enum) casados por joaat contra os hashes da tabela de parser. | Não. |
| 400 | exeschema: enum value tables and more struct references | Estende #394: membros de enum/bitset apontam para um objeto de enum com tabela `{hash do nome, valor}`; saída vira `{"structs": [...], "enums": [...]}`. | Não. |
| 401 | Viewer: the first wear layer's tint blend (W2) and 4th wear channel | Implementa no shader as duas partes restantes da primeira camada de desgaste: `lerp(tint, cor da paleta, W2)` e o 4º canal (D4); declara depender do PR 381. | Não. |
| 402 | Hair colliders: add the box pieces of the body and garment bounds | Usa também as peças tipo 3 (caixas, bbox em +0x30/+0x20) dos bounds de corpo e roupa no amortecimento do cabelo; até agora só as peças redondas. | Não. |

## Pull requests mesclados (39 dentro da faixa documentada)

A busca `merged:>=2026-11-26T04:15:00Z` devolveu **47 merges**. Desses, **39** têm `mergeCommit.oid` igual a um
dos commits documentados e estão listados; os **8** restantes (`#380`, `#381`, `#382`, `#386`, `#390`, `#395`,
`#396`, `#397`) pertencem aos 8 commits que ficam para a próxima execução. O commit `f96b3f08` (PR #344) foi
mesclado antes do cursor e por isso não aparece nem na busca nem na tabela.

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| 336 | Map 3D: each place's own water, and the bed through shallow water | fc530fd5 | commit 16 |
| 339 | Map 3D: the weather's clouds and their shadows | 72303f5b | commit 12 |
| 341 | Map 3D: the land drawn as made-up ground, with the tiles' relief | 20d1d191 | commit 6 |
| 343 | Map 3D: GPU frame time by part in the tick lines | d187e24e | commit 7 |
| 348 | Hair export: scalp hair pressed under the outfit's hat | 54f96b2d | commit 9 |
| 349 | Cutscene characters: clothing sets on the model that wears them, names confirmed by scene peds, outfit for models without own parts (#306) | 6694896a | commit 2 |
| 350 | Viewer: the primer's colour from the material palette | 30202d89 | commit 3 |
| 351 | Map 3D: map cells draw from 0.3 of the HD distance (25 to 44 fps downtown) | 0cc6f6c4 | commit 13 |
| 352 | BC3/BC4/BC5 BCPack: 44 preset slots fitted on mip consistency | ef255ad9 | commit 4 |
| 353 | Hair: each strand its own coverage dither; curls per guide; count rule in 1440 lines | c06f5b97 | commit 5 |
| 354 | Viewer: clothes under clothes drawn as their _cull_00 twins and cut by the hide masks | 7943fda9 | commit 8 |
| 355 | Head skins: cs_ heads without a face preset take their character's face (cs_lucia) | 51b0a7ae | commit 10 |
| 356 | Body shape: full rig program; clothing helper bones follow the garment surface rig (#308) | 0a850d5b | commit 11 |
| 358 | Outfit export: clothes under clothes as their _cull_00 twins, skin and clothes cut by the hide masks | 58d22b38 | commit 14 |
| 359 | BC4/BC5 BCPack: 6 preset slots refined in a second mip-consistency round | 99fae714 | commit 15 |
| 360 | Painted hair: find a hair item's layer set by its own tag, not by its drawable name | 94852942 | commit 20 |
| 361 | Map 3D: cloud noise baked once into a repeating texture (cheaper clouds) | cca88805 | commit 21 |
| 362 | Viewer: the primer's metalness and roughness blended by the material map | d0cc7e06 | commit 17 |
| 363 | Viewer: distance test as a backstop under the hide masks; second mask list findings | 1bb8aec4 | commit 18 |
| 364 | Skin shading: the body kept out of its own mirrored surroundings (last jaw rim, #308) | 7aa8ef88 | commit 19 |
| 365 | Hair colliders from the character's body bound | b666920a | commit 22 |
| 366 | Body skins: approximate skin for whole-body meshes that no body set names | 97ac6c2b | commit 23 |
| 367 | Cutscene characters: stand-in clothes for the eight head-only characters (#306) | ebad7ee6 | commit 24 |
| 368 | Version 0.11.0 | 97d5cce9 | commit 25 |
| 369 | Map 3D: back to the procedural cloud noise (revert #361) | 444327fe | commit 26 |
| 370 | Map 3D: water-plant tiles and far SLOD tiles no longer draw flat squares near the camera | ebf09a0c | commit 27 |
| 371 | Viewer: pick animations from dropdowns instead of typing the name | cc73e1a8 | commit 29 |
| 372 | Decode the BC5 cloud volume: 3D layout for 16-byte blocks and its mip chain | bac66cca | commit 28 |
| 373 | Meta dump: fixed-capacity arrays (type 0x53) list every element | 4a90d613 | commit 31 |
| 374 | Hair colliders: the worn garments' bounds join the body bound | b2698c0e | commit 30 |
| 375 | Outfit export: --parts hides covered skin and under-layers, like --look | 7cb78e82 | commit 33 |
| 376 | Map bake: read the newer of the two placements exports | 7e3eb450 | commit 37 |
| 377 | README: the index's size on disk today | 37f7942b | commit 32 |
| 378 | Body skins use the character's own normal map | a0b7fb69 | commit 34 |
| 379 | Outfit export: parts added with --add hide what they cover | 77ec8c5f | commit 35 |
| 383 | Export inland water surfaces (rivers, canals, creeks, reservoirs) with `sixng water --inland` | 492b0400 | commit 36 |
| 387 | Map 3D: settle views that want more models than the count cap | 70cab4c5 | commit 40 |
| 388 | Viewer: Breasts, Butt and region body sliders; outfits open in their body-shape preset | 9de14109 | commit 38 |
| 389 | Animation picker: animal species from the model list, short button text, screenshots wait for the lists | 8c6e2312 | commit 39 |

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum dos 40 hashes é `corrigido` de um defeito de `SixNG` (D-01…D-24). Três frentes
  encostam em defeitos conhecidos, sem fechá-los:
  - **D-08** (tabelas de preset BCPack com deslocamento constante: speckle em áreas planas). `ef255ad9` (43
    entradas de BC4, 1 de BC3) e `99fae714` (6 entradas de BC4/BC5) reajustam presets por consistência de mips.
    É a mesma família de defeito (tabela de preset errada ⇒ speckle), em BC3/BC4/BC5 e não em BC6H; os commits
    não citam D-08 nem trazem imagem antes/depois.
  - **D-11** (parâmetros de material lidos no offset errado). `30202d89` e `d0cc7e06` acrescentam leitura de
    `primer`/`primer_set` e mistura de metalness/roughness por mapa — offsets novos, não correção dos antigos.
  - **D-03** (normal map em frame tangente errado). `a0b7fb69` faz a pele do corpo usar o normal map do
    personagem; não muda frame tangente nem flip de verde.
- **Abre defeito novo?** Dois candidatos, **NAO VERIFICADO**:
  1. `cca88805` (nuvem pré-cozida em textura repetida) foi **revertido** por `444327fe` no mesmo dia, cinco
     horas depois. É exatamente o padrão que o catálogo já registra em outros repositórios (D-20 e D-29 de
     `Xbox2Windows`: defeito introduzido e corrigido no mesmo dia). O par não gera ID aqui porque a reversão
     veio antes de qualquer release; fica como candidato se o release 0.11.0 (`97d5cce9`, commit 25, anterior
     ao revert) tiver saído com a nuvem pré-cozida.
  2. `30202d89` sobe `EXPORT_VERSION` 28→29 e `FORMAT` 23→24 e `6694896a` sobe `CACHE_VERSION` 4→5. Quem tiver
     `vehicle_parts.json`/cache de outfits de versão anterior perde o cache silenciosamente (recomputa). Não é
     erro, mas o custo de reconstrução do índice não está medido em nenhum dos dois commits.
- **Sem cruzamento:** os commits de mapa 3D (`20d1d191`, `72303f5b`, `fc530fd5` etc.) e os de cabelo
  (`f96b3f08`, `c06f5b97`, `54f96b2d`) não têm defeito correspondente no catálogo — são funcionalidade nova.

## Confiança

- **Verificado por diff:** os 40 hashes foram lidos em `gh api repos/holdmysocks/SixNG/commits/<sha>`; para os
  quatro primeiros li hunks completos, para os demais a lista de arquivos com estatísticas e o assunto. Onde a
  descrição depende do assunto (números de fps de `0cc6f6c4`, "1440 linhas" de `c06f5b97`), isso está dito no
  texto como afirmação do autor.
- **Cobertura:** 40 de 48 commits. `last_commit` = `commit_cursor` = `70cab4c5` (40º); head na coleta =
  `09cc63a8b510026e8d03562c571c083465b1c8ac`. Os 8 restantes ficam para a próxima execução.
- **Merges:** 47 na janela; 39 casados por `mergeCommit.oid`. `mergedAt` máximo entre os 39 é
  `2026-11-26T18:31:19Z` (#387, `70cab4c5`), novo cursor de PRs mesclados.
- **PRs abertos:** 9 de 9 documentados; `last_pr_number` → 402. Um PR (#398) é de outro autor
  (`maple-quartz18`), os demais de `holdmysocks`.
- **NAO VERIFICADO:** os dois candidatos acima; o efeito visual das mudanças de shader (não há captura nem
  comparação numérica no repositório); se o release 0.11.0 saiu entre `97d5cce9` e o revert `444327fe`.
