# holdmysocks/SixNG — atualização de 2026-11-26 21:00

## Resumo

**16 commits** em `70cab4c5..8b9fac39`, todos *squash* de PRs (#380 a #402) mesclados depois do cursor
`last_run` de 18:31:19Z. A faixa é quase toda de conteúdo e de formato de exportação: o registrador de
instâncias de prop ganha nome para os bytes `+0x3E/+0x3F` e um arquivo novo
(`batch_props_layer.bin`), o veículo ganha o *tint* da segunda camada de desgaste (`wear_tint`, formato
25), o cache de células do mapa passa a ser chaveado pelo *bake*, e entram `packparts` (peças de ped em
pacotes de conteúdo), a máscara de horas de `CTimeArchetypeDef` e um leitor da tabela de estruturas do
executável (`exeschema`, 233 de 234 estruturas conferidas). Nenhum D-xx do catálogo é corrigido.

## Commits novos (16)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| 8c7feca6 | Brandon Villar | README: the body normal-map change also covers 1,138 heads (#380) | `README.md` (+7/−1), só documentação: cabeças cujo item de roupa nomeia a face passam a usar o normal map **da face** (rugas, sobrancelha, idade) em vez do normal map genérico da malha — 1 138 cabeças; `cs_benedict_halifax` tem no lugar da face algo que não é normal map. 22 cabeças que emprestam face pela tabela de skins mantêm o normal map da malha. Registra o comportamento já implementado em `a0b7fb69` (#378). |
| e9d6f235 | Brandon Villar | Viewer: wear and primer window widths with the game's float arithmetic (#381) | `model_view.rs` (+19/−8): troca a aritmética de byte (`/255`, `clamp(0,1)`, `*255+0.5`, `saturating_sub`) pela do jogo — `f32::from_bits(0x3b80_8081)` para ler o preset, `(x*255.0) as i32 as u8` para embrulhar negativo, largura `trunc((end−start).max(1e-4))` e não a diferença de bytes. Teste fixa `12..44 → start 12, largura 31` e `19..255 → 236`. Antes a largura saía um a mais em janelas curtas. |
| 6f77a46f | Brandon Villar | Batches: name the layer-blend byte of prop instances (#382) | `batches.rs`, `export.rs`, `material.rs`, README (+34/−7): o `raw: (f32, f32, u16, [u8;2])` do registro de 44 B vira `layer_blend` (`+0x3E`, valor `b` do shader de 2 camadas = byte/255) e `layer_aux` (`+0x3F`, desaturação lida ao lado; igual em 96 % das instâncias, 0/0 desliga a camada 2). Grava `batch_props_layer.bin` (2 B por instância, mesma ordem de `batch_props.bin`). Teste com `255−k`/`7`. |
| 9a712b3f | Brandon Villar | Body shape: 11 more preset controls drive the 14 undriven rig inputs (#386) | `bodyshape.rs`, `main.rs`, README (+42/−7): 11 controles do preset PSO (`anim.rpf@106153000`) passam a dirigir 14 entradas de rig que ficavam em 0 — `gcbreastheight`, `gcbreastspacing`, `gcchestheight`, `gccleavage` (só rig masculino), `gchipheight` (−1..1; negativo dirige a entrada oposta pelo tamanho), `gcbellyround`, `gchipwidth`, `gcheadscale`, `gcmusclebiceps`, `gcmuscletriceps` e `gc_b0eae798` (nome não recuperado, −1..1). Nomes são joaat do id do controle; `5ebebbe7` (cabeça/pescoço) fica em 0. Remove a regra de clamp genérica por prefixo. |
| 85d4052e | Brandon Villar | Viewer: no crash on a character-to-animal switch or after a GPU device loss (#390) | `main.rs` +59, `render3d.rs` +27, `model_view.rs` +5/−2: entra `watch_gpu_loss` (`set_device_lost_callback` + `on_uncaptured_error`) — depois de device loss os erros de validação são logados uma vez em vez de *panic*, e uma faixa no topo pede reinício. E `joints_fit(bytes, n)` recusa escrever matrizes de esqueleto em buffer menor: ao trocar personagem→animal, `hair.gpu` ainda tem o cabelo do personagem (buffer de 1 matriz) e o `write_buffer` estourava a validação; agora só modelos cujo `joints_buf` comporta `skin.len()` recebem a escrita. |
| 16be4854 | Brandon Villar | Body shape test: gccleavage is a known control now (#396) | `bodyshape.rs` (+1/−1): o teste de `preset_from_xml("BeachBellies")` passa a esperar `gccleavage=0.5` junto de `gcbutt=0.75` e `gcweightbelly=0.5`. Consequência de `9a712b3f`. |
| 8d9594a3 | Brandon Villar | Map 3D: key the map-cell cache by the bake (#395) | `map_cells.rs` +54, `map2d.rs` +21, README (+79/−9): o diretório de células era fixo (`TAG_FORMAT "v10_…"`); agora há `fingerprint()` (FNV de 64 bits sobre `entities.bin` e `strings.json`, dobrado a u32) e `bake_id = fp(entities) ^ fp(strings).rotate_left(17)`. Células de outro *bake* são apagadas; reexportar não reusa célula antiga. |
| 09cc63a8 | Brandon Villar | Map 3D: far-field blocks wait only for buildings still loading (#397) | `map3d.rs` (+15/−2): o bloco de far-field deixa de esperar qualquer prédio ausente; espera só o que está na fila ou em voo (`queue`+`inflight`), mais `stats.more_queued`. Prédio descartado pelos limites de memória não vai chegar e não segura a área — antes os blocos distantes ficavam por cima dos prédios desenhados. |
| bd1b1456 | Brandon Villar | Characters: draw the nipple and genital inserts from the content pack (SIX-229) (#384) | `packparts.rs` novo (+157), `pedlook.rs` +69, `main.rs` +19, `model_view.rs` +21 (+270/−19): drawables de ped que não estão em `packs.rpf` mas em pacote de conteúdo (`pack/<nome>/<nome>.rpf`, p.ex. `core_restofworld`) passam a ser lidos pelo próprio arquivo (`archive_of`), com `is_pack_key`, `loaded()` e `by_name`; `outfit::dedupe` deixa de descartá-los e o fallback por regex consulta os pacotes. Antes os inserts anatômicos (`bodygenitalsgen_*`) ficavam sem malha. |
| 7451a19f | Brandon Villar | Weapon speculartex2 slot role: wear amount, not a metal mask (#391) | `material.rs` +23, `export.rs` +3/−3: entra `SHADER_SLOTS` — pares (hash de shader, hash de parâmetro) com papel próprio, começando por `e68a4a98`/`739e3d0d` (programa pixel de arma): R é **quantidade de desgaste** (lerp para rampa de desgaste, metalness ×(1−w), rugosidade para um valor de bloco) e G modula brilho; metalness vem de outro mapa. `slot_role_in` consulta a tabela e cai no papel genérico. O papel antigo dizia "provavelmente roughness/metal/AO". |
| e2cb516d | Brandon Villar | LOD links: hour masks of time archetypes (#393) | `lodlinks.rs` +80, `export.rs` +3, README (+84/−7): `CTimeArchetypeDef` (`0x76b0c56c`) = `CBaseArchetypeDef` + `timeFlags` (`0x8609d52c`); `time_hours(flags)` devolve as horas 0..23 com bit ligado no low 24 bits. Entidades desses arquétipos (poucas dezenas: variantes de loja aberta/fechada, espreguiçadeiras) ganham `time {time_flags, hours}` no JSON de placements e no `archetype_lod.jsonl`. |
| 701f59e1 | Brandon Villar | exeschema: meta struct schemas from the executable's parser table (#394) | `crates/sixng-core/examples/exeschema.rs` novo (+439): lê a tabela de estruturas do parser no `.text` — getter `lea rcx,[slot]; lea rdx,..; call <ctor>`, cabeçalho após o slot (`u64` tabela de membros, `u32` hash do nome joaat case-sensitive, `u32` size, `u16` flags, offsets `u32` a partir de header+18), tabela de membros com `u16` contagem/offsets e registros (`u32` hash, `u8` tipo, `u8` subtipo; `Item` `063fa3f2` descreve arrays/maps). Validação declarada no cabeçalho: 233 das 234 estruturas dos meta de `levels` estão na tabela e todos os 1 421 membros aparecem com tipo igual. |
| e59fecec | maple-quartz18 | refactor(metal4): extract build_vertex_index_bytes for shared use (#398) | `render3d_metal4.rs` (+41/−26): o laço que montava vértices/índices e `GpuMesh4` sai do caminho de upload e vira `build_vertex_index_bytes(&glb::Model)`, reusado pelo `upload_shared`/`bake_model_cache`; mesmos limites (`model.materials.len()` ou 1) e o mesmo `bail!` para triângulo incompleto. Refatoração sem mudança de comportamento. |
| e88d5348 | Brandon Villar | Names: 4,114 meta field, struct and enum names from the executable's parser table (#399) | `names_exe.txt` novo (4 139 linhas, 4 114 nomes) + `names.rs` (+22/−1): `exe_names()` inclui o arquivo em tempo de compilação; os nomes entram no mapa `exact` (joaat case-sensitive) como `derived`. Teste exige >3 000 nomes sem espaço e a presença de `priorityLevel`. |
| f6cbf195 | Brandon Villar | Viewer: the first wear layer's tint blend (W2) and 4th wear channel (#401) | `vehcolour.rs` +30, `model_view.rs` +84, `shader.wgsl` +44, `export.rs` +2 (+160/−13): `wear_of` passa a devolver também a cor do registro de cor da superfície (`wear_rgb`, sRGB); `EXPORT_VERSION` 29→30 e `FORMAT` 24→25; sai `wear_tint` por look (256 × 6 dígitos hex) e o WGSL mistura esse *tint* onde o peso de paleta W2 é baixo e a linha de paleta (desgaste) onde é alto. Antes a segunda camada só tinha linha de paleta, rugosidade e metalness. |
| 8b9fac39 | Brandon Villar | Hair colliders: add the box pieces of the body and garment bounds (#402) | `groom.rs` +45, `fragphys.rs` +12, `model_view.rs` +26 (+83/−24): peças `kind 3` do body bound (caixas) deixam de ser descartadas: `bbox` em `+0x30`/`+0x20`, com teste de sanidade (finito, `min ≤ max`, extensão < 2,0), e viram `Cuboid {centre, axes, half}` com `dist()` assinado. `Colliders` guarda `boxes` além de `tapers`; o cabelo colide com corpo e roupa, não só com os cones. |

## Pull requests abertos (8)

Nove PRs abertos na coleta; 8 acima do cursor 402. "Proposta" sai do título e da lista de arquivos do PR.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 403 | Outfits open in their body-shape kits' preset items | `bodyshape.rs`, `pedlook.rs`, `pedset.rs`, `wardrobe.rs`, `model_view.rs`: abre o outfit já no preset do kit (7 arquivos). | Continua `9a712b3f`; não é defeito do catálogo. |
| 404 | Placements: priority names; archetype field doc from the executable's parser table | `meta7a.rs`, `export.rs`, README: nomes de `priority` e documentação dos campos do arquétipo pela tabela do exe. | Não. |
| 405 | Viewer: clear-coat loss and the edge bump on worn paint | `model_view.rs`, `shader.wgsl`: perda de verniz e *bump* de borda na tinta desgastada. | Vizinho de `f6cbf195` (camadas de desgaste); não mapeado. |
| 406 | Inland water: flow texture, water height texture and ripple settings per tile | `water.rs`, README: textura de fluxo, altura e ondulação por tile. | Não. |
| 407 | Viewer: under-layers stay behind the clothes over them on a reshaped body | `model_view.rs`: camadas de baixo atrás da roupa sobre corpo remodelado. | Não. |
| 408 | Viewer: Shape details sliders for the 11 further body-shape controls | `bodyshape.rs`, `pedlook.rs`, `pedset.rs`, `wardrobe.rs`, `model_view.rs`, CLI: sliders para os 11 controles novos. | Continua `9a712b3f`. |
| 409 | Placements: archetype kind (asset type and MapEntityType) per entity | 14 arquivos, inclui `exeschema.rs`, `lodlinks.rs`, `names.rs`: tipo de asset e `MapEntityType` por entidade. | Não; usa `701f59e1`/`e88d5348`. |
| 410 | Viewer: the game's dust and scratch mask on vehicle paint | `shader.wgsl`, `render3d*.rs`, `cells.rs`: máscara de poeira/risco na pintura. | Vizinho de `f6cbf195`; não mapeado. |

## Pull requests mesclados (16)

Todos os 16 são *squash*: `mergeCommit.oid` é o próprio commit da tabela acima.

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| 380 | README: the body normal-map change also covers 1,138 heads | 8c7feca6 | Só README; o código já estava em `a0b7fb69`. |
| 381 | Viewer: wear and primer window widths with the game's float arithmetic | e9d6f235 | Aritmética de float do jogo e teste de largura (31 em 12..44). |
| 382 | Batches: name the layer-blend byte of prop instances | 6f77a46f | `layer_blend`/`layer_aux` + `batch_props_layer.bin`. |
| 384 | Characters: draw the nipple and genital inserts from the content pack (SIX-229) | bd1b1456 | `packparts` e leitura de `core_restofworld`. |
| 386 | Body shape: 11 more preset controls drive the 14 undriven rig inputs | 9a712b3f | 11 controles → 14 entradas de rig. |
| 390 | Viewer: no crash on a character-to-animal switch or after a GPU device loss | 85d4052e | `watch_gpu_loss` e `joints_fit`. |
| 391 | Weapon speculartex2 slot role: wear amount, not a metal mask | 7451a19f | `SHADER_SLOTS` com `e68a4a98`/`739e3d0d`. |
| 393 | LOD links: hour masks of time archetypes | e2cb516d | `CTimeArchetypeDef`/`time_hours`. |
| 394 | exeschema: meta struct schemas from the executable's parser table | 701f59e1 | Exemplo `exeschema` (439 linhas). |
| 395 | Map 3D: key the map-cell cache by the bake | 8d9594a3 | `bake_id` por fingerprint. |
| 396 | Body shape test: gccleavage is a known control now | 16be4854 | Uma linha de teste. |
| 397 | Map 3D: far-field blocks wait only for buildings still loading | 09cc63a8 | Espera só fila/inflight. |
| 398 | refactor(metal4): extract build_vertex_index_bytes for shared use | e59fecec | Extração de função. |
| 399 | Names: 4,114 meta field, struct and enum names from the executable's parser table | e88d5348 | `names_exe.txt` (4 114 nomes). |
| 401 | Viewer: the first wear layer's tint blend (W2) and 4th wear channel | f6cbf195 | `wear_rgb`, formato 25, WGSL com W2. |
| 402 | Hair colliders: add the box pieces of the body and garment bounds | 8b9fac39 | `Cuboid` no `Colliders`. |

## Cruzamento com o catálogo

- **Não corrige nenhum D-xx.** `6f77a46f` mexe exatamente no registro de instância de prop do D-05
  ("instâncias de prop-batch decodificadas como lixo"), mas só nomeia os bytes `+0x3E/+0x3F` e exporta
  um arquivo novo; a correção do D-05 é a de 2026-11-23 citada no README (posição 0/1e10, escala inf),
  que não é tocada aqui.
- **Não corrige D-03** (normal map em frame tangente errado, flip de verde): `8c7feca6` muda *qual*
  normal map é usado (face em vez de malha), não o frame nem o flip.
- **Abre candidato novo (rol de slot de material):** `7451a19f` mostra que `739e3d0d` era documentado
  como "provavelmente roughness/metal/AO" e no shader de arma é desgaste + brilho. Candidato a D-xx em
  `catalogo/defeitos.csv` (repo `SixNG`, arquivo `crates/sixng-core/src/material.rs`).
- **Abre candidato novo (recorte de colisor):** `8b9fac39` acrescenta caixas ao colisor de cabelo; o
  catálogo não tem entrada para o cabelo atravessar corpo/roupa.
- **Limitação declarada no próprio código:** `9a712b3f` deixa `5ebebbe7` (cabeça/pescoço) em 0 e não
  roda os rigs de compensação de escala de cabeça; `6f77a46f` não rastreia o significado de
  `layer_aux`. Não são defeitos do catálogo, são lacunas registradas.

## Confiança

- **Verificado por diff:** os 16 commits tiveram o *patch* lido pela API (`repos/.../commits/<sha>`),
  arquivo por arquivo; os números de linha citados (`+0x3E`, `+0x30`, `0x76b0c56c`, `0x3b80_8081`) e
  os nomes de função saem do diff.
- **Contagem de merges fecha:** 16 PRs com `mergedAt` posterior a `2026-11-26T18:31:19Z` e 16 commits na
  faixa; cada `mergeCommit.oid` é igual ao SHA da tabela (squash), sem divergência a registrar.
- **Não verificado:** os corpos dos PRs abertos (proposta tirada do título e dos caminhos de arquivo,
  sem ler descrição nem patch); a afirmação de validação do `exeschema` (233/234 estruturas, 1 421
  membros) é do comentário do próprio arquivo — não reexecutei o exemplo contra o executável do jogo.
- **Teto de 40 commits não foi atingido** (16 de 16). `commit_cursor` vai a `8b9fac39`, igual ao head.
