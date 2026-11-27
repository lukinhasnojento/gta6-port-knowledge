# holdmysocks/SixNG — atualização de 2026-11-27 15:00

## Resumo
24 commits na `master` desde `e380c30d` (`2026-11-27T04:37:14Z`), todos do mesmo autor e todos casados 1:1 com
24 PRs mesclados (#447 a #470). A hora é de **conteúdo de personagem e veículo**: decoração de ped (makeup com
ponto branco Lab, UV do próprio registro, camadas de makeup por tom de pele), cabelo (guia com "leash" do
jogo, coroa do chapéu como volume de contenção, ordem embaralhada das mechas em LOD), pele (atlases de
detalhe, janela de albedo do F0), véu de exportação (inserts anatômicos sob a roupa, acessório no osso de
attach, livery como malha de decalque) e 2.729 nomes de enum. Fecha com **0.12.0**.

## Commits novos (24)
| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `790a3c673d` | Brandon Villar | Map 3D: draw neighbouring meshes of a model as one range (#470) | `render3d.rs` (+91/−20): o índice de cada mesh passa a endereçar o **vertex buffer inteiro** (sem base vertex), o que permite desenhar vizinhos como uma faixa — no passe principal juntando meshes de mesmo material, no passe de sombra juntando todos os sólidos vizinhos. `map3d.rs` (+56) acrescenta `SIXNG_MAP_DRAWLOG=1`, que imprime por tick os 10 modelos com mais draws (principal e sombra). |
| `d416eca356` | Brandon Villar | Version 0.12.0 (#469) | `Cargo.toml` e `Cargo.lock`: `0.11.0` → `0.12.0`. Só versão. |
| `724ad49408` | Brandon Villar | Ped decorations: place layers in the record's own UV (#468) | `peddeco.rs` (+50/−6): a camada passa a ser posicionada no UV **do próprio registro** (UV da parte × matriz do registro `6e8249bf`), como o `MSNC_VS_BakeDecorations` do jogo rasteriza; fora de `0..1` o registro não desenha. Sobe `VERSION` 3→5 e `BAKE_RULE` 6, o que invalida caches de bake antigos. |
| `dc0fdfd1f3` | Brandon Villar | Strand hair: no bare patches when a groom is drawn at a distance (#467) | `groom.rs` (+60): `Coat::interleaved()` reordena as mechas em ordem embaralhada determinística (hash multiplicativo sobre o índice), porque o corte por distância pegava só o começo da lista — que eram membros de clump — e deixava áreas carecas; agora todo corte mantém distribuição uniforme. Teste em `hairexport.rs` passa a afirmar 120 pontos soltos + 30 de clump, não mais "os 120 primeiros". |
| `27faca7f7a` | Brandon Villar | Hair under hats: keep it inside the crown; sunglasses and the hat fit follow the head scale (#466) | `groom.rs` (+107): `Shell::with_crown(hat)` e `Crown::fit(hat, centro, 0.006)` dão à coroa do chapéu o papel de volume de contenção do settle (elipsoide, tipo 16 do jogo); `model_view.rs` (+38/−20) passa a coroa só para cabelo sob chapéu e trata acessório preso sob a cabeça (`under_head`) como parte da cabeça para o scale. |
| `dc99b51d6f` | Brandon Villar | Facial controls: face_tonguethicknesstip (#392) (#465) | `facecontrols.rs` +1 nome (`0x2e37b26c`) e comentário: passam 11 ids sem nome (quatro são controles 2-D que o rig de dentes/língua lê, sete nenhum rig lê). README: 79 → 80 nomes. |
| `9096558e45` | Brandon Villar | Enum value names: prefix-less enums, data file types, vehicle parts (1,659 values) (#464) | `enum_values.txt` +1659 linhas; `meta7a.rs` (+3/−1) com teste `enum_value_name(0x3ff19213, 6) == "HANDLING_FILE"` e o piso de contagem subindo de 4.000 para 5.500. |
| `e25f88a7f3` | Brandon Villar | Skin detail atlases and occlusion; translucency recorded (#392) (#463) | 10 arquivos, +341/−23. `skinspec.rs` (+175): os atlases de detalhe são **engine-wide** (nenhuma mesh os liga) e vêm de dois arrays 2-D de 48 tiles 256×256 em `data/ttdb/graphics/ttdb_graphics.ttpf`, tiles 16–31 pele (poros/rugas) e 0–15 tecido; a tabela de região do material (`a6b9e97a`) converte a região em `speculartex2` G em tile. `shader.wgsl` (+52) implementa as duas fórmulas do jogo (cor `*= saturate(1 + str.x*mask*(2c−1))`, normal tangente `+= str.y*mask*detail.xy`) e `glb.rs` (+36) grava `extras.skin_spec.detail`. |
| `dbcc5ae04a` | Brandon Villar | Names: three library and shader constant names (SIX-68) (#462) | `names.rs` +4: `sheenfallbackroughness`, `sheencutoffthreshold`, `tintsplitcolor`, com a nota de que os dois primeiros ficam a dois slots dos constantes de sheen de tecido. |
| `163958933c` | Brandon Villar | Water: export the run-time water textures (--detail) (#461) | `water.rs` (+98): opção `detail` grava as texturas que o jogo liga em runtime (`DETAIL_TEXTURES`: `waterbump`, `waternoise`, espuma, caustics, perfis de onda de costa, mapas de controle de oceano) — os drawables de água carregam placeholders 4×4 nesses slots. `main.rs` (+7) expõe `--detail`, escrevendo `detail/*.png`, `detail/*.dds` e `detail/detail.json`. |
| `9d113b2985` | Brandon Villar | Ped decorations: the game's Lab white point for makeup (#460) | `peddeco.rs` (+4/−3): o branco de referência da conversão Lab deixa de ser a soma das linhas da matriz `XYZ` e passa a ser a entrada D65 da tabela de iluminantes do jogo copiada no start (`[0.95047, 1.0, 1.08883]`); `BAKE_RULE` 4→5. |
| `2746d9f524` | Brandon Villar | Skin F0: the engine's albedo window, decoded (#455) | `skinspec.rs` (+34): os dois programas de pele escalam F0 por `saturate((max(r,g,b) − lo)/(hi − lo))` sobre o albedo linear, com limites no buffer por frame (`6e8725e4` +0x6c0/0x6c4) preenchidos a partir de sRGB 5 e 20, isto é `lo = 0.0015176`, `hi = 0.0069954`; só albedo quase preto perde especular. `shader.wgsl` (+4) aplica a janela; o GLB grava `extras.skin_spec.albedo_window`. |
| `0205b9ef56` | Brandon Villar | Liveries panel: undo and redo (SIX-249) (#452) | `livery.rs` (+153): `History` por veículo, com passo = mudança entre dois momentos de repouso (arrasto, giro de roda ou rajada de digitação é um passo), tomado por `History::REST` segundos sem mudança. `livery_ui.rs` (+62) com `--livery-undo N` / `--livery-redo N` para passo roteirizado. |
| `42cc029646` | Brandon Villar | Outfit export: the genitals stay under clothes (#451) | `outfit.rs` (+192): `hide_inserts` esconde os inserts anatômicos (genitais, mamilos — meshes de pele desenhadas sobre a pele do corpo) por três critérios contados em `InsertHiding`: região herdada do vértice de pele mais próximo, máscaras de hide da camada e roupa cobrindo a maior parte. `export.rs` (+17) carrega `part_layers` (layer, insert, garment, hides). |
| `c7f018c9bb` | Brandon Villar | Game liveries: one option per brand slot, as the game fits it (#459) | `vehstickers.rs` (+42/−31): um slot de marca pode ter várias opções com probabilidade — o jogo encaixa **uma**, então os adesivos são alternativas, não camadas. `livery_ui.rs` (+40) cria `variants(brand)` = maior número de opções de um slot, e a variante k pega a opção k (com wrap) de cada slot; cache renomeado `vehicle_liveries_f4.json`. |
| `3b2333673c` | Brandon Villar | Ped decorations: bake head makeup toward the skin tone (#458) | 13 arquivos, +292/−56. `peddeco.rs` (+238): makeup de cabeça passa a ser misturado (em sRGB, peso `w`) na direção do tom de pele deslocado por `(dL, da, db)` em CIELAB D65 e reempacotado em HSV; `pedlook.rs`/`export.rs` carregam `skin_tone` e gravam `makeup_rule`/`skin_tone` no JSON; `BAKE_RULE` e `VERSION` sobem. |
| `df7945833c` | Brandon Villar | Hair: hold the settled guides on the game's leash (#457) | `groom.rs` (+64): lê a "leash" dos guides do header de gráficos (`+0x3c`, B3: 4 bytes por segmento, quarto byte / 600 m = raio por ponto, 0 nas raízes) e o expoente do header `+0x70` byte 2 (`3E/255`, 0 = sem leash); sem B3 o jogo usa `4 a c`. |
| `30ea1f3b3d` | Brandon Villar | Characters: a legacy outfit gets no second head or body (#456) | `pedlook.rs` (+10/−3): quando a parte é `legacyped…_fullbody` (corpo vestido inteiro) ou `legacyheadcs_…`, a cabeça própria e a pele nua do corpo deixam de ser acrescentadas ao outfit — apareciam através da malha legada. |
| `47250388a5` | Brandon Villar | Enum value names: 1,070 more from prefix brute force and structured candidates (#454) | `enum_values.txt` +1073: força bruta de palavras dentro do prefixo comum de cada enum, com controle de hash aleatório; um valor com vários nomes (alias, como `KEY_PRIOR`/`KEY_PAGEUP`) ganha uma linha por nome. `meta7a.rs`: aliases passam a manter o **primeiro** nome listado (`or_insert`), com testes `TYPE_QUADRATIC_OUT`, `KEY_F1`, `L3`. |
| `b7c3f1d579` | Brandon Villar | Facial control names and garment pose shape keys (#392) (#453) | `facecontrols.rs` novo (+142): os 91 ids que os clipes faciais usam vêm de joaat de nomes FACS (`face_[l_ ou r_]<região>_<ação>`), com espelho esquerda/direita como checagem. `facegraph.rs` (+103): rede de pose corretiva do personagem (uma por modelo de ped; `player_story000_00` tem 294 entradas em 8 grupos de 36×7+42 e 4.675 pontos), amostrada pela stream 8 da roupa (tag `0xb210`/`0xb220`, três pontos por vértice, deslocamento pela média); exporta como shape keys `pose_000…`. |
| `f9ab2bf070` | Brandon Villar | Vehicle export with its livery: stickers as decal meshes in the GLB (SIX-248) (#450) | `vehlivery.rs` novo (+425): recorta os triângulos dentro do retângulo UV do adesivo e escreve uma primitiva por adesivo e modelo, no nó do modelo em que ele cai e skinnada como os faces abaixo (segue a porta). `livery_ui.rs` (+94) com `export_glb` para a configuração do viewer; `--livery` no CLI (+21) com arquivo salvo por padrão. |
| `282d337322` | Brandon Villar | Names: eleven more material parameter and shader constant names (SIX-68) (#449) | `names.rs` +25: 11 nomes de parâmetro (`thinfilmior`, `thinfilmmaxthickness`, `tintpaletteselector`, `hboheightscale/bias`, `translucency`, `dustintensity`, `sheenfadealbedo`, …); `material.rs` (+2/−2) fecha `emissivecolormultiplier` = `7ab0fd74` (a cor do neon); `export.rs` atualiza a nota de `materials_note`. |
| `af60984a2b` | Brandon Villar | Outfit export: an attached accessory goes on its attach bone (#448) | `outfit.rs` (+133): `accessory_attach` reconhece relógio/anel (todas as ossos usados com um prefixo só, raiz no topo da hierarquia desse prefixo, vértices a ≤25 cm da raiz, osso de attach não usado a ≥30 cm dela — esquerdo quando há dois) e move a raiz para o osso de attach; `model_view.rs` (−33/+8) passa a usar a função em vez da cópia local. |
| `280f553ef1` | Brandon Villar | Liveries panel: fit a new sticker to the largest UV island of its zones (SIX-247) (#447) | `livery.rs` (+177): calcula por UV set a ilha (patches de faces conectados no UV) e a área de cada zona; `livery_ui.rs` (+4) avisa quando as zonas estão em mais de uma ilha (`main_share < 97 %`) e o adesivo é disposto na maior. |

## Pull requests abertos (1 novo)
| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 472 | `feat(metal4): add offline model cache baking tool` (branch `p2-offline-model-cache-bake`, base `master`, aberto `2026-11-27T07:07:32Z`, autor maple-quartz18) | Pelo PR: subcomando `--bake-model-cache` que varre os `.glb` do diretório de trabalho e pré-gera caches `SXMTLIO2`, reusando `build_vertex_index_bytes()` e os escritores `model_identity`/`write_model_cache_entry` para os bytes ficarem idênticos aos do upload em runtime; depois de cozinhar, relê cada `.buf` em blocos de 64 KiB e confere o CRC de vértice/índice contra o cabeçalho. 4 arquivos: `crates/sixng-gui/src/main.rs`, `modelbakejob.rs` (novo), `pages.rs`, `render3d_metal4.rs`. | Não identificado no catálogo (24 itens para `SixNG`; nenhum cita cache de modelo ou metal4). **Diff não inspecionado:** a lista de arquivos e o texto vêm de `gh pr view 472`; nenhum hunk foi lido. |

## Pull requests mesclados (24)
Todos são merges squash na `master`: o `mergeCommit.oid` de cada PR é o próprio commit da tabela acima.

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| 470 | Map 3D: draw neighbouring meshes of a model as one range | `790a3c673d` | Faixa única de índices sobre o vertex buffer compartilhado + `SIXNG_MAP_DRAWLOG`. |
| 469 | Version 0.12.0 | `d416eca356` | `Cargo.toml`/`Cargo.lock` para 0.12.0. |
| 468 | Ped decorations: place layers in the record's own UV | `724ad49408` | UV do registro aplicado à camada; `VERSION` 5, `BAKE_RULE` 6. |
| 467 | Strand hair: no bare patches when a groom is drawn at a distance | `dc0fdfd1f3` | Ordem embaralhada determinística das mechas. |
| 466 | Hair under hats: keep it inside the crown | `27faca7f7a` | `Crown::fit` como contenção do settle; acessórios sob a cabeça seguem o scale. |
| 465 | Facial controls: face_tonguethicknesstip | `dc99b51d6f` | 80º nome de controle facial. |
| 464 | Enum value names: prefix-less enums, data file types, vehicle parts | `9096558e45` | +1.659 nomes de valor. |
| 463 | Skin detail atlases and occlusion; translucency recorded | `e25f88a7f3` | Atlases de detalhe por região, oclusão e translucidez no GLB e no viewer. |
| 462 | Names: three library and shader constant names | `dbcc5ae04a` | 3 nomes. |
| 461 | Water: export the run-time water textures (--detail) | `163958933c` | `--detail` com bump, espuma, caustics, perfis de costa e controle de oceano. |
| 460 | Ped decorations: the game's Lab white point for makeup | `9d113b2985` | Branco D65 explícito; `BAKE_RULE` 5. |
| 455 | Skin F0: the engine's albedo window, decoded | `2746d9f524` | Janela `0.0015176..0.0069954` aplicada ao especular da pele. |
| 452 | Liveries panel: undo and redo | `0205b9ef56` | Histórico por repouso + `--livery-undo/--livery-redo`. |
| 451 | Outfit export: the genitals stay under clothes | `42cc029646` | `hide_inserts` com região herdada, máscaras e cobertura. |
| 459 | Game liveries: one option per brand slot | `c7f018c9bb` | Variantes por slot; cache `f4`. |
| 458 | Ped decorations: bake head makeup toward the skin tone | `3b2333673c` | Makeup movido ao tom de pele em CIELAB. |
| 457 | Hair: hold the settled guides on the game's leash | `df7945833c` | Raio de leash por ponto (B3) + expoente. |
| 456 | Characters: a legacy outfit gets no second head or body | `30ea1f3b3d` | Outfit legado não recebe cabeça nem pele extra. |
| 454 | Enum value names: 1,070 more | `47250388a5` | +1.070 nomes; aliases mantêm o primeiro. |
| 453 | Facial control names and garment pose shape keys | `b7c3f1d579` | `facecontrols.rs` + `facegraph.rs` (pose corretiva por shape key). |
| 450 | Vehicle export with its livery | `f9ab2bf070` | Adesivos como malhas de decalque no GLB. |
| 449 | Names: eleven more material parameter and shader constant names | `282d337322` | 11 nomes + `emissivecolormultiplier`. |
| 448 | Outfit export: an attached accessory goes on its attach bone | `af60984a2b` | `accessory_attach` compartilhado. |
| 447 | Liveries panel: fit a new sticker to the largest UV island of its zones | `280f553ef1` | Ilhas de UV por zona e encaixe na maior. |

## Cruzamento com o catálogo
- **Corrige D-xx?** Nenhum dos 24 itens do CSV para `SixNG` é fechado por esta hora, e nenhum commit cita
  `D-nn`. Três territórios encostam em itens existentes, sem fechar:
  - **D-24** ("cor da configuração tratada como linear quando é valor de display") é sobre cor de pintura
    de veículo; `9d113b2985` e `3b2333673c` mexem em espaço de cor, mas no caminho CIELAB da **decoração de
    ped**, com o branco D65 agora explícito. Não alteram o tratamento das cores de configuração.
  - **D-12/D-14/D-15** (carroceria, cor de peça, slot de cor do pneu) não são tocados; `c7f018c9bb` e
    `f9ab2bf070` tratam de adesivo de marca e export de livery, outro caminho.
  - **D-23** (UV das camadas de ground, 64 por unidade) é de terreno; `724ad49408` trata do UV do registro
    de decoração de ped.
- **Abre defeito novo?** Nenhum identificado. Duas mudanças de cache merecem acompanhamento, não são
  defeito: `724ad49408` sobe `VERSION` (3→5) e `BAKE_RULE` (2→4→5→6 na hora) — todo bake de decoração
  antigo é regenerado, e `3b2333673c` inclui o `skin_tone` na chave de cache; `c7f018c9bb` renomeia o cache
  do viewer para `vehicle_liveries_f4.json`, então o arquivo `f3` fica órfão em `cache/`.
- Limitação declarada pelo próprio código, não defeito novo: `e25f88a7f3` registra que translucidez e
  oclusão alimentam **só** o passe diferido, "que não é decodificado"; `b7c3f1d579` registra que o que
  alimenta as entradas da rede de pose não é decodificado, e por isso as entradas saem como shape keys.

## Confiança
- 24 commits conferidos por `gh api repos/holdmysocks/SixNG/commits/<sha>`, com arquivos, status e hunks.
  `e380c30d7b49b6f59af2820aa3eeb6ed78e43738` (cursor anterior) estava na posição 24 da listagem de 100 da
  `master`, então a faixa é exatamente 24 commits.
- **Contagem commits × merges**: 24 merges com `mergedAt > 2026-11-27T04:37:14Z` e 24 commits; cada
  `mergeCommit.oid` foi casado com o commit correspondente, sem sobra. O `#446` (`mergedAt` idêntico ao
  cursor) foi documentado na execução anterior e ficou fora por filtro estrito.
- O commit `dc99b51d6f` não aparece na ordem cronológica do topo (é de `06:06:31Z`, entre itens de
  `06:06:47Z` e `06:06:14Z`); a lista segue a ordem do grafo, não a data.
- Não verificado: se as contagens de README (1.659 + 1.070 nomes de enum, 80 controles faciais, 147 cabeças
  com rede nomeada) se sustentam em execução — vêm dos testes e do texto do diff. Não rodei o CLI nem o
  viewer.
- Não verificado: o PR #472 por diff (só título e branch).
