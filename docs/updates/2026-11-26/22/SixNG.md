# holdmysocks/SixNG — atualização de 2026-11-26 22:00

## Resumo

Seis PRs mesclados em 48 minutos e cinco abertos, todos do mesmo autor principal, quase todos no viewer
e no decodificador de mundo. O bloco de pintura de veículo avança em três frentes (perda de clear coat e
*edge bump* em `63a69321`, HSV e *sun bleach* ainda em PR, máscara de poeira/risco em `e0fd6ffa`), o
vestuário ganha as kits de *body-shape* do jogo (`4f2e560a`) e a ordem de camadas sobre corpo remodelado
(`be41e588`), e a água interiorana passa a levar textura de fluxo e de altura por tile (`2f8763ff`).
Nenhum commit novo desde `e0fd6ffa`, que é o merge do PR #410.

## Commits novos (6)

Faixa `8b9fac39..e0fd6ffa`, 6 commits, todos merges de PR.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `4f2e560a` | Brandon Villar | Outfits open in their body-shape kits' preset items (#403) | O outfit deixa de abrir pelo rótulo (`Average`, `Skinny`) e passa a abrir pelas **kits de body-shape** que a variação `BodyShapePreset` inclui, na ordem de include: `PRESET_ITEM_KEYS` (5 chaves, lista estática em `.rdata 0x651eab4`, colhidas por `fn 0x3e47f90`) nomeiam itens do PSO de preset em `anim.rpf` (struct `890b5009`), aplicados kit a kit, com valor posterior vencendo no mesmo controle (`fn 0x4f95310`). `bodyshape.rs` +121 (`outfit_preset`), `pedlook.rs` +21 (`body_preset_items`), `wardrobe.rs` mantém as chaves como hash, CLI e `model_view.rs` consomem. Mandy abre com músculo 0,2 e quadris/pernas/seios mais cheios; Lucia com quadris/pernas/glúteos; Jason musculoso. 7 arquivos, +181/−22. |
| `63a69321` | Brandon Villar | Viewer: clear-coat loss and the edge bump on worn paint (#405) | Preenche as colunas 69–72 da linha de *coat* do parte com intensidade/início/fim do registro de borda (`487f0f4c`, empacotado em parte+40 por `rva 0x1a0d7a0`) e o byte de perda de clear coat do registro de idade (`aa7d4fbd` +0xf0: `trunc(25,5 × valor)` em parte+56 byte 3). No shader, `wear_level_sum` soma os níveis da primeira camada de desgaste (canais 16+c, 34+c, 37+c, com gama) e abre a janela de borda sobre a soma. `model_view.rs` +32/−3, `shader.wgsl` +53. |
| `2f8763ff` | Brandon Villar | Inland water: flow texture, water height texture and ripple settings per tile (#406) | Cada tile de água interiorana passa a carregar sua própria textura de fluxo (BC3 256×256 em 944 tiles, 512×512 em 7; 7 tiles parados com placeholder 4×4) e de altura d'água (R16F, 128 ou 256 px, 0–1), com UV0 = `world x / 64` e `-world y / 64` — o que fecha exatamente nos 952 tiles com vértices de superfície. Ripple scale 0,04 e bumpiness 1 em todos; `inland.glb` mantém UV0. `water.rs` +86/−10 (`shader_json`, `flow`, `height_tex`), README reescrito na linha do sistema de água. |
| `be41e588` | Brandon Villar | Viewer: under-layers stay behind the clothes over them on a reshaped body (#407) | Cada peça segue a pele por conta própria, então num corpo pesado ou musculoso uma peça de baixo pode sair **na frente** da de cima (camiseta atravessando o bolso de uma camisa aberta). Guarda as posições por mesh antes de seguir a pele e reordena as camadas por `layered` para devolver a peça de baixo para trás. `model_view.rs` +103. |
| `8ea8bea8` | Brandon Villar | Part sets: a legacy whole-body mesh makes a complete character (#412) | `is_whole_body` reconhece mesh de personagem inteiro legado (`legacyped..._fullbody_...`: `cs_ricky_diaz_legacypedcs_mr1_000_fullbody_ws00_full`, `legacypedworld_fr1_005_fullbody_ws00_full`) e o trata como parte de corpo, marcando o conjunto `complete` e extraindo o `fit` do próprio mesh quando não há slot de cabeça. `pedset.rs` +29/−5. |
| `e0fd6ffa` | Brandon Villar | Viewer: the game's dust and scratch mask on vehicle paint (#410) | Liga a máscara de poeira e risco do jogo (`dusttexture`, parâmetro `fb03ae36`) como binding 8, com bit 4 no flags; as colunas 73–88 da linha de parte levam os bytes da máscara e as 8 curvas do registro de idade (`fe5d32e7`, `rva 0x37766a0`: min, max, start, end, expoente). No shader, a máscara entra no tiling da peça, vermelho/azul (riscos) pelas curvas e verde (poeira) pelo verde do vértice, somando rugosidade (k3) e tirando clear coat (k1); quando o modelo não tem a máscara, usa a de desgaste. 7 arquivos, +137/−2. |

## Pull requests abertos (5)

Todos com número acima de 410.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 411 | Inland water: decode the water height texture and check it per tile | Explica o conteúdo da textura de altura (`z = origem z + (2v − 1) × H`, texel 0 = sem água, `H` = maior distância de vértice à altura da origem) e checa tile a tile, com `shading.water_height_scale_m` e `shading.water_height_check`. +55/−4. | Não. Complementa `2f8763ff`. |
| 413 | Viewer: the paint shader's HSV colour step (albedo multiply / screen, sun bleach) | Desenha o passo de cor do shader de pintura do jogo: escurecer, clarear e *sun bleach* em HSV, com os valores por superfície, lidos dos bytes 1–3 de surface record +28 e empacotados por `rva 0x37766a0` do objeto `307f162c`. Inclui a branch do #410 (merge primeiro). +272/−8, 9 arquivos. | Borda o **D-24** (cor tratada como linear onde o jogo usa valor de display): aplica o passo que o jogo aplica, mas o defeito do catálogo é sobre a conversão de espaço de cor da configuração. |
| 414 | feat(metal4): add GPU-ready model buffer cache via Metal IO | Cache versionado de buffers de modelo prontos para GPU (`SXMTLIO2`), espelhando o cache de textura: hit faz DMA direto para buffer `PRIVATE` com `commit().wait()`, miss cai no staging e escreve em background (2 escritores, teto de 4 GiB, LRU), identidade por nome + versão + CRC. +254/−2. | Não. Backend metal4, não o viewer wgpu. |
| 415 | Viewer: exit cleanly after a GPU device loss (SIX-234) | Depois do #390 a perda de dispositivo ainda saía com código 101, porque `egui-wgpu` 0.36.2 entrava em panic no paint seguinte (`renderer.rs:984`) e a linha vermelha nunca aparecia; passa a mostrar caixa nativa e sair com código 3, checando `GPU_LOST` antes de qualquer view 3D, screenshot ou `gpu_time`. +98/−5. | Não no catálogo. |
| 416 | Hair noise and curl: the game's frame (in-plane root axis, y = T × axis) | O ruído e o cacho do cabelo (#310) passam a usar o referencial do jogo: `x = A` tornado perpendicular a `T`, `y = T × A`, com `A` vindo do referencial da guia levado ao root da mecha (`B1`, `B2`, `B2 × B1` = normal). +52/−22. | Não. |

## Pull requests mesclados (6)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| 403 | Outfits open in their body-shape kits' preset items | `4f2e560a` | `outfit_preset` em `bodyshape.rs` (+121) e `body_preset_items` em `pedlook.rs`; README reescrito para dizer que a palavra é só rótulo e que Lucia/Jason abrem neutros. |
| 405 | Viewer: clear-coat loss and the edge bump on worn paint | `63a69321` | Colunas 69–72 (edge intensity/start/end, coat loss) e `wear_level_sum` no WGSL; sem mudança de decodificador. |
| 406 | Inland water: flow texture, water height texture and ripple settings per tile | `2f8763ff` | `shader_json` com efeito, texturas (parâmetro, hash de store, tamanho, formato) e constantes por tile; texturas gravadas em `inland/textures/`. |
| 407 | Viewer: under-layers stay behind the clothes over them on a reshaped body | `be41e588` | Reordenação das camadas por posições pré-reshaping; só `model_view.rs`. |
| 412 | Part sets: a legacy whole-body mesh makes a complete character | `8ea8bea8` | `is_whole_body`, marcação `complete` e `fit` do mesh inteiro; só `pedset.rs`. |
| 410 | Viewer: the game's dust and scratch mask on vehicle paint | `e0fd6ffa` | Binding 8 com fallback para a máscara de desgaste, colunas 73–88 e bit 4 no flags; `glb.rs` exporta a máscara, `render3d.rs`/`render3d_metal4.rs`/`world3d.rs`/`cells.rs` acompanham. |

## Cruzamento com o catálogo

- **D-24** (cor da configuração tratada como linear quando é valor de display, pintura escura): a faixa
  trabalha a pintura de veículo, mas pelo lado do **jogo** — `e0fd6ffa` soma rugosidade e retira clear coat
  pela máscara de poeira e `63a69321` aplica a perda de clear coat e a janela de borda do registro de
  idade. Não altera a conversão de espaço de cor da configuração; o PR #413 é o que chega perto (passo HSV
  e *sun bleach*) e também não corrige a conversão. **Não corrige.**
- **D-12** (carroceria do veículo aplicada a interior, carcaças de lâmpada e peças mecânicas): `e0fd6ffa`
  condiciona a máscara a `classification.stand_in == "parts"` e a um limiar de largura ≥32 px, o que é o
  mesmo eixo de classificação de peça que originou o D-12, mas não muda a atribuição de carroceria.
- **D-11** (valores de parâmetro de material lidos no offset errado): `4f2e560a` traz offsets novos e
  verificados (`.rdata 0x651eab4`, PSO `890b5009`, struct `91315b01`, `fn 0x3e47f90`/`0x4f95310`) para
  outro sistema (body-shape); não revisa os offsets de material do D-11.
- **D-01/D-04** (modelos vazios; modelo antigo sob nome novo): `8ea8bea8` trata mesh legado de corpo
  inteiro, o que evita "conjunto incompleto" em personagens legados, mas não é a divisão de recurso RAGE
  nem a corrida de carregamento assíncrono. **Não corrige.**
- **Abre defeito novo?** Um candidato com hash e arquivo:
  - `be41e588` corrige um defeito real do viewer (peça de baixo atravessando a de cima em corpo
    remodelado) — `crates/sixng-gui/src/model_view.rs`. Não há entrada para ele no catálogo; seria a
    **D-25** do `SixNG`.
  - Observação de dados, não defeito de código: `2f8763ff` registra que **81 de 958** superfícies
    interioranas ficam fora do leito/margem medidos (59 rios, 8 reservatórios, 5 canais) e que 7 tiles
    parados levam placeholder 4×4 — o próprio commit diz que não checou a causa.

## Confiança

Verificado: os 6 hashes e metadados em `gh api .../compare/8b9fac39...e0fd6ffa`; cada commit lido em
`.../commits/<sha>` (7, 2, 2, 1, 1 e 7 arquivos, com trechos de patch de `bodyshape.rs`, `model_view.rs`,
`pedset.rs`, `water.rs`, `glb.rs` e `shader.wgsl`); os 6 merges em `--state merged --limit 50` com
`mergeCommit.oid` igual ao hash do commit; os 5 PRs abertos lidos em `.../pulls/<n>` (`draft=false`).

Não verificado: os endereços e RVAs citados (`.rdata 0x651eab4`, `rva 0x1a0d7a0`, `rva 0x37766a0`) vêm
do texto dos commits, não de leitura do binário; se o viewer abre os corpos com os valores declarados
(Mandy 0,2 etc.); se o passo HSV do PR #413 reproduz o shader do jogo; e se os 81 tiles fora do leito
são erro de dados ou de amostragem — o commit não conclui.
