# holdmysocks/SixNG — atualização de 2026-11-27 00:00

## Resumo

Vinte e seis commits na `master` em cerca de cinco horas (`e0fd6ffa..912287bd`), e **cada um é o squash
de um PR**: os 26 PRs mesclados da janela casam 1:1 com os 26 commits. Quase tudo é o visualizador e o
decodificador de personagem — cabelo (cores por peça, ruído/curl no referencial do jogo, blur
screen-space, simulação de guias), corpo (escala de cabeça de dois lados, sliders das 11 controles
extras, colares e bainhas em corpo moldado) e espaço do jogo (água interior, tinta de props, liveries de
veículo, tatuagens). Fora do visualizador entram três peças de base de dados do executável: tabelas de
enum, tipos de membro do meta e defaults de material. Dez PRs abertos novos.

## Commits novos (26)

Faixa `e0fd6ffa..912287bd`, em ordem cronológica. Todos são squash de PR e a mensagem tem só o assunto —
o mecanismo abaixo vem do diff (`commits/<sha>`) e do corpo do PR correspondente.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `a8d77218` | maple-quartz18 | feat(metal4): add GPU-ready model buffer cache via Metal IO (#414) | 2 arquivos, +254/-2 (`render3d_metal4.rs`, `main.rs`). Cache versionada de buffers de modelo prontos para a GPU (`SXMTLIO2`), no molde da cache de textura: no acerto, `load_buffer` traz os bytes de vértice/índice direto para buffers PRIVATE (`staging=None`) com `commit().wait()`; no erro, cai no caminho de staging e grava a cache em thread de fundo (máx. 2 escritores, teto de 4 GiB, poda LRU). Identidade = nome do arquivo de modelo + versão de origem + CRC dos bytes, então mudança de conteúdo invalida sozinha. |
| `aa4f1451` | holdmysocks | exeschema: enum value tables and more struct references (#400) | 1 arquivo, +129/-10 (`examples/exeschema.rs`). Membro de enum/bitset aponta para um objeto de enum com hash do nome, contagem e ponteiro para tabela de `{hash do nome do valor, valor}` de 12 bytes, terminada em `{0,-1,-1}`; a saída passa de lista de structs para `{"structs":[…],"enums":[…]}`. Uma segunda forma de store nos getters (`mov rcx, rax; …; mov [x], rcx`) sobe as referências de struct de 8.292 para 10.059. `--work <dir>` passa a nomear hashes pelo dicionário da pasta. |
| `29bbf3a0` | holdmysocks | Viewer: Shape details sliders for the 11 further body-shape controls (#408) | 3 arquivos, +50/-10. Seção recolhível "Shape details" com os 11 controles do #386 (altura/espaçamento de seio, altura de peito, clivagem, largura/altura de quadril, redondeza de barriga, escala de cabeça, bíceps, tríceps, seios plásticos); o corpo passa a ser array de 37 controles em vez de descartar os que o preset trazia. Faixa −1..1 nos controles de dois lados, 0..1 nos demais. |
| `d0ced331` | holdmysocks | Inland water: decode the water height texture and check it per tile (#411) | 2 arquivos, +55/-4 (`water.rs`, README). Um texel guarda a altura da superfície de água relativa à origem do tile: `z = origin z + (2v − 1)·H`; texel 0 significa sem água. `H` é a maior distância de um vértice da superfície até a altura da origem e sai como `shading.water_height_scale_m`. Cada tile ganha `shading.water_height_check`, com mediana e percentil 95 do erro, valor mediano e fração de texels com água. |
| `02456c4d` | holdmysocks | Viewer: exit cleanly after a GPU device loss (SIX-234) (#415) | 4 arquivos, +98/-5 (`main.rs` +85). Depois do #390 a perda real de dispositivo ainda saía com código 101, porque o egui-wgpu 0.36.2 entrava em pânico no paint seguinte (`Failed to create staging buffer for index data`) antes de a linha vermelha ser desenhada. Agora a perda mostra caixa de mensagem nativa e sai com código 3; `logic` e `ui` checam a flag `GPU_LOST` antes de qualquer view 3D, screenshot ou leitura de `gpu_time`, e um panic hook converte qualquer pânico posterior na mesma saída limpa. Threads de upload de mapa param quando a flag sobe. |
| `cf173c2b` | holdmysocks | Placements: priority names; archetype field doc from the executable's parser table (#404) | 3 arquivos, +36/-4. Cada `lod` ganha `priority_name` ao lado do número: REQUIRED, OPTIONAL_HIGH, OPTIONAL_MEDIUM, OPTIONAL_LOW (0–3), vindos da tabela de enum do executável. Corrige a doc do arquétipo contra a tabela do parser: **+0x78 é `name` e +0x7C é `clipDictionary`** — a doc dizia dicionário de textura e de drawable; +0x7C vale 0 nos arquétipos conferidos e nenhum código o lê como dicionário. Lista os enums de asset type e MapEntityType (0–7). |
| `e123f84a` | holdmysocks | Placements: archetype kind (asset type and MapEntityType) per entity (#409) | 3 arquivos, +70/-3 (`lodlinks.rs` +66). `sixng lod-links` grava `assetType` (+0x84) e MapEntityType (+0xB0) de cada arquétipo em `archetype_lod.jsonl`; `sixng placements` acrescenta `archetype_kind {asset_type, asset_type_value, entity_type, entity_type_value}` por entidade. Os valores valem para arquétipos simples e temporais (o struct temporal deriva do base). Arquivos antigos carregam sem o campo. |
| `4b2333fd` | holdmysocks | Viewer: the paint shader's HSV colour step (albedo multiply / screen, sun bleach) (#413) | 4 arquivos, +135/-6 (`vehcolour.rs` +59, `shader.wgsl` +62). O visualizador passa a desenhar a etapa de cor da tinta em HSV — escurecer, clarear e desbotar por sol — com os valores de cada superfície. Os bytes lidos são 1, 2 e 3 do registro de superfície +28; o packer (rva 0x37766a0) copia 8 bytes para +24..+31 a partir do objeto de runtime de um registro `307f162c` (objeto +0x28), pela ligação `2cc72f7c` da primeira camada e `174770cc` da segunda. |
| `588c680b` | holdmysocks | Hair noise and curl: the game's frame (in-plane root axis, y = T x axis) (#416) | 2 arquivos, +52/-22 (`hairgen.rs` +31, `groom.rs` +21). Ruído e curl do cabelo deixam de usar a normal da superfície na raiz e passam ao referencial do jogo, por ponto: `x` = vetor A tornado perpendicular à tangente T, `y` = T × A. A vem do referencial da guia levado ao referencial da raiz da mecha (B1, B2, com B2 × B1 = normal da raiz); quando a mecha corre ao longo da guia, A resulta em −B2, direção fixa no plano da superfície. |
| `2461a0e9` | holdmysocks | Textures tab: keep the preview beside the Used by panel (#417) | 1 arquivo, +5/-0 (`texture_view.rs`). O preview da textura deixa de ser dimensionado na mesma área em que o painel "Used by" acabou de ser colocado; passa para um painel central depois da lista e do "Used by", como a aba Jobs faz. O código do preview muda sem alteração para `preview_ui`. |
| `6fe3a138` | holdmysocks | Viewer: a picked hair colour overrides the stored hair colours (#418) | 2 arquivos, +17/-2. Com "stored hair colours" ligado (padrão quando os itens nomeiam um preset de cabelo), clicar numa amostra ou no seletor de cor não mudava nada visível, e `--hair-colour` era sobrescrito ao carregar. Agora escolher uma cor desliga as cores guardadas, e um checkbox "stored" ao lado do seletor as religa. |
| `a11ba3d4` | holdmysocks | Characters: leave out items that name a first-person mesh (SIX-229 follow-up) (#419) | 1 arquivo, +28/-0 (`pedlook.rs`). Item cujo GUID nomeia malha de primeira pessoa (`hand_fr1_000_nude_cull_fp`) passa a ser listado como "first person only" e não é casado pelas texturas. Conserta a regressão do #384 em Lucia: o item caía no fallback de textura, tomava o slot de corpo superior e vestia a textura das mãos em primeira pessoa sobre a blusa. |
| `ed17ecec` | holdmysocks | Viewer: per-piece hair colours that can be linked (#420) | 3 arquivos, +218/-32 (`model_view.rs` +202). Cada peça de cabelo (cabelo, sobrancelha, cílios, barba e qualquer outro slot) ganha linha própria de cor no painel Character, com escolha de vínculo a outra peça; alternativas contam como a peça delas ("other hair" como cabelo, "other beard" como barba). |
| `fae4801a` | holdmysocks | Body shape: head scale is two-sided (#421) | 2 arquivos, +13/-7 (`bodyshape.rs` +7). O input de rig `5ebebbe7` (cabeça e pescoço) é o lado negativo de `gcheadscale`: em 1 ele escala o osso da cabeça para 0,01, o pescoço superior para 0,25 e o inferior para 0,5, espelhando `gcheadscale` em 1 (2,0 / 1,75 / 1,75). `gcheadscale` passa a variar de −1 a 1 no `--body-shape` e no slider do visualizador. |
| `2970355d` | holdmysocks | Meta reader: decode every member type the meta files use (#422) | 3 arquivos, +84/-25 (`meta7a.rs` +82). Usa a tabela de tipos conferida no #400 para decodificar todos os códigos de membro que os arquivos meta usam, agrupados por nibble alto (0x0_: bool, struct, ponteiro, id de 64 bits; 0x1_: inteiros; …). `vehexport.rs` e `vehcolour.rs` passam a ler por esse caminho. |
| `ac4fd64b` | holdmysocks | Viewer: draw the game's own vehicle liveries (#423) | 9 arquivos, +297/-21 (`livery_ui.rs` +171, `vehstickers.rs` +80). O visualizador aplica as liveries do próprio jogo ("brands"), cada adesivo com a transformação que o jogo calcula do layout: registro de camada de 64 bytes escrito pelo packer em rva 0x37766a0 — byte 0 = modo UV do layout, +4 = f16 (ou, ov), +8 = f16 (a, b), +0xc = f16 (c, d), +0x14..0x1c = três cores de máscara, +0x24/+0x28 = duas texturas, +0x30 = flag de máscara. |
| `2b00c691` | holdmysocks | Material library: the game's parameter defaults (SIX-68) (#424) | 4 arquivos, +196/-26 (`matlib.rs` +133, `meta7a.rs` +48). Os defaults de parâmetro de material em camadas que preset e material base deixam em branco passam a vir do jogo — arquivo `$(target)/data/dynamic_art/muv_parameter_default_values` (tipo `DYNAMIC_ART_MUV_PARAMETER_DEFAULTS_FILE`), um PSO dentro de `data.rpf` alcançado só pela TOC, achado como o PSO plano pequeno de root `e5f515b3`. Contém quatro mapas: 80 floats, 10 cores, 6 ints e 5 texturas, por nome de parâmetro; o índice de material os guarda (versão 7) e `sixng materials` os imprime sob `defaults`. |
| `4a3e7d4a` | holdmysocks | Inland water check: list the tiles outside bed and banks, and tell edge slivers from full tiles (#425) | 2 arquivos, +31/-4 (`water.rs` +30). O check passa a listar cada tile fora da cama e das margens em `check_against_height_map.outside`, com altura de superfície, menor e maior terreno, lado e distância da falta, tamanho do tile, contagem de vértices, fração de amostras acima da água, terreno sob os vértices e a fonte da célula. O resumo conta "slivers" (tiles com menos de 6 m ou menos de 30 vértices) e reporta `metres_per_texel`. Nos dados reais: os 81 tiles ficam abaixo do terreno mais baixo e 61 são slivers. |
| `956b603e` | holdmysocks | Meta: enum value names from the executable's enum tables (#426) | 2 arquivos, +3.321/-1 (`enum_values.txt` novo com 3.257 linhas, `meta7a.rs` +64). 3.253 valores nomeados de 919 enums, com os hashes de valor vindos das tabelas de enum do parser (#400) e casados contra `names_exe.txt` só por joaat exato e sensível a caixa. `meta7a::enum_values()` e `enum_value_name(enum_hash, value)` usam o hash do enum no `extra` do membro. |
| `0165c222` | holdmysocks | Viewer: a necklace stays on the neck of a shaped body (#427) | 1 arquivo, +55/-25 (`model_view.rs`). Peças abaixo de 30 cm (relógio, óculos, anel, colar) seguem a pele moldada como peça rígida, deslocada pelo offset da pele no centro — mantêm a forma, mas o colar era movido sem alargar e o pescoço mais grosso o cobria. Agora cada ponto também se afasta para fora onde a pele sob ele inchou mais que o offset, ao longo da linha do vértice de pele mais próximo. |
| `a1e1718a` | holdmysocks | Hair under a hat: fills a strap opening, hangs on below the edge (#428) | 1 arquivo, +69/-4 (`groom.rs`, `Shell`). Abertura que o chapéu **envolve** contava como fechada, então o cabelo ali era prensado logo abaixo da superfície da aba e aparecia como rede esparsa; e quando um segmento de mecha de sob o chapéu até um ponto abaixo da borda cruzava o chapéu, o ponto era puxado de volta, colapsando o resto da mecha na borda. O diff passa a preencher a abertura da alça e a deixar a mecha continuar caindo abaixo da borda. |
| `c58add0c` | holdmysocks | Tattoos: decoration catalogue, placement and full-colour bake (#385, part 1) (#429) | 13 arquivos, +700/-22 (`peddeco.rs` novo, +518; `export.rs` +109). Módulo `peddeco`: lê o catálogo de decorações de `data.rpf` — 7.509 registros, struct `40e79f52`, camadas de struct `77f0121d`, pelos offsets de membro do schema do próprio arquivo — e o guarda em `index/ped_decorations.json`. Assa as camadas na textura de cor da parte, com vínculo de outfit, posicionamento e cobertura. **Tatuagem a tinta fica de fora** porque a codificação de tinta não está decodificada (é o #442, aberto). |
| `9726909f` | holdmysocks | Viewer: screen-space ray march for hair diffuse ambient (#430) | 1 arquivo, +70/-1 (`shader.wgsl`). O ambiente difuso do cabelo deixa de ser céu desobstruído em torno da tangente e passa a ser a GI de sonda em tela do jogo: os pixels de cabelo hospedam sondas orientadas pela tangente da mecha, cada raio é marchado primeiro em tela pelo depth buffer, com comprimento = profundidade de vista limitada a 1 m na distância de personagem, e o gather final é um lobo de cosseno em torno da tangente. |
| `9f89a24f` | holdmysocks | Tint palettes: row layout from the game's mesh shader (SIX-41) (#432) | 3 arquivos, +29/-11 (`texture.rs` +26). A regra de linha da paleta de tinta passa a ser a do shader de mesh `MS_VS_TransformD_c5fd36bb`: com `k = width & 7`, cada linha tem `256 >> k` entradas, há `width >> (8 − k)` linhas e a entrada lida é `min(tint, rows − 1) × (256 >> k) + round(255 × COLOR0.b)`. Onde a regra não dá linha inteira, cai para uma linha da largura toda; entre as larguras em uso só a 32 (7 paletas) cai no fallback. Conferido contra as 127.287 paletas embutidas: só larguras não usadas com `k ≥ 6` mudam. |
| `4daafc38` | holdmysocks | Viewer: the genitals stay under clothes (#433) | 1 arquivo, +139/-38 (`model_view.rs`). O insert anatômico do #384 é malha de pele separada, desenhada sobre o corpo, e a versão masculina **não carrega regiões de vértice** (todas 0), então a máscara de esconder da roupa na camada de pele inferior não o alcança; o fallback só esconde pele logo abaixo da peça. O diff faz o insert seguir a mesma máscara e o mesmo fallback, inclusive sob peça larga (toalha), onde a pele sob ela fica na máscara. |
| `912287bd` | holdmysocks | Head scale: earrings and strand hair follow the scaled head (#434) | 3 arquivos, +144/-7 (`model_view.rs` +89, `bodyshape.rs` +47). `gcheadscale` escala `BM_Head`, e escalpo, orelhas, raiz do rabo de cavalo, óculos e chapéu pendem dele. Brincos e cabelo de mecha não seguiam: em −0,5 ficavam em tamanho cheio em volta de uma cabeça pequena; em +0,5 e +1 afundavam, restando só o pino. O diff leva os dois atrás da escala, com a transformação por osso que o #421 deixou de dois lados. |

## Pull requests abertos (10)

Todos com número acima do cursor `416`.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| `444` | Hair: settle the guides as the game's strand simulation does | Simulação de mechas nas guias a cada frame, com a passada completa perto da câmera; entradas por ponto vêm do groom (complacência a puxão e dobra em +0x34, comprimentos de repouso em +0x38). +281/-5 | Não |
| `443` | Viewer: clothing hems stay smooth on a heavy body | Bainhas de roupa em corpo pesado: a média inversa ao quadrado dos 6 vértices de pele mais próximos dá serrilhado; empilhado no #427. +67/-31 | Não |
| `442` | Tattoos: decode the ink words and bake ink-coloured decorations | Fecha a lacuna declarada no #429: palavra de tinta `0xAAVVSSHH` (hue, saturação, valor, alfa) e três stops em sRGB a partir do `ramp = hue(H)`. +275/-65 | Não |
| `441` | Tint palettes: export how much of the palette the game applies (SIX-245) | Exporta o peso `w = 1 − (1 − speculartex2.a) × strength` (constante `381999f3`) e o segundo controle `73511514`; critério = efeito que liga speculartex2, paleta e os dois controles (217 efeitos). +66/-2 | Não |
| `440` | Game liveries: a sticker on every model with its zones; pictures keep their own colours | Seguimento do #423: adesivo de marca copiado para **todo** modelo que carregue uma das zonas (as quatro popas do Rhode), e adesivo full-colour mantém as próprias cores — 1.899 de 2.444 listam três cores zero e saíam pretos. +20/-13 | Não |
| `439` | Viewer: the game's screen-space hair blur | Porta o `hair_resolve` do jogo (`SingleDrawBlurLow*`): pixel borrado quando há pixel de mecha resolvido na vizinhança 3×3, kernel gaussiano 3×3 com pesos `1 / exp(−1/2s²) / exp(−1/s²)`, sem parada por profundidade. +397/-0 | Não |
| `438` | Head shape keys: face expression networks as glTF morph targets, viewer Expression panel | Segunda metade do #392: rede por face do jogo lida de duas resources sem nome (tipo 95 ComputeGraph, tipo 114 ComputeGraphConstants) ligadas ao nome do drawable da cabeça (`<head>` no LOD0, `<head>_lod1` no LOD1). +767/-19 | Não |
| `437` | Fit vehicle parts whose names are known, and boats' parts | Peças de configuração eram descartadas quando o nome do hash estava resolvido; passa a encaixar nomeadas e hasheadas, e encaixa peças de barco por osso `chassis`. +10/-7 | Não |
| `436` | Water: export pools, hot tubs, fountains, waterfalls and standing water with `sixng water --pools` | Corpos d'água pequenos em `pools/pools.json` e `pools/pools.glb`; critério = nome marca água e geometria LOD0 desenha com shader de água (`water_micro_geom`, `standard_alpha_water`, `standard_alpha_waterfall` ou bucket `water_bucket`). +316/-33 | Não |
| `435` | Body, hand and feet skin spec maps from the wardrobe material assets (#392, part 1) | Mapas de spec de pele vêm dos 47 asset sets de `Material` em `packs.rpf`: chave `179eb44a` = `speculartex`, `4cf41ef4` = `speculartex2`, porque o shader `ped_standard_skin` liga placeholders em toda malha principal. +353/-1 | Não |

## Pull requests mesclados (26)

Todos com `merge_commit_sha` igual ao commit de squash na `master` — não há merge commit de PR criado
pelo GitHub, então "hash do merge" abaixo é o commit squash. **Onde o corpo do PR e o commit divergem de
tamanho**, o PR estava empilhado em outro ainda não mesclado: o diff do PR inclui os commits do de baixo,
e o que entrou de fato é só o commit squash.

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| `434` | Head scale: earrings and strand hair follow the scaled head | `912287bd` | O PR declara +5.364/-222 em 31 arquivos e 31 commits; entrou +144/-7 em 3 arquivos. O resto do diff era o #421/#432/#408 ainda empilhados. Entrou só o ajuste de brincos e mecha à escala de cabeça |
| `433` | Viewer: the genitals stay under clothes | `4daafc38` | Declara +5.520/-241 em 34 arquivos/20 commits; entrou +139/-38 em 1 arquivo (`model_view.rs`) |
| `432` | Tint palettes: row layout from the game's mesh shader (SIX-41) | `9f89a24f` | +29/-11 em 3 arquivos; bate com o commit |
| `430` | Viewer: screen-space ray march for hair diffuse ambient | `9726909f` | +70/-1 em `shader.wgsl`; bate |
| `429` | Tattoos: decoration catalogue, placement and full-colour bake (#385, part 1) | `c58add0c` | +700/-22 em 13 arquivos; bate. Tinta fica para o #442 |
| `428` | Hair under a hat: fills a strap opening, hangs on below the edge | `a1e1718a` | +69/-4 em `groom.rs`; bate |
| `427` | Viewer: a necklace stays on the neck of a shaped body | `0165c222` | +55/-25; bate |
| `426` | Meta: enum value names from the executable's enum tables | `956b603e` | Declara +4.494/-144 em 25 arquivos/15 commits; entrou +3.321/-1 em 2 arquivos (o `.txt` de 3.257 linhas e `meta7a.rs`) |
| `425` | Inland water check: list the tiles outside bed and banks, and tell edge slivers from full tiles | `4a3e7d4a` | +31/-4; bate (o commit tem 2 commits no PR, o corpo diz 81 tiles e 61 slivers) |
| `424` | Material library: the game's parameter defaults (SIX-68) | `2b00c691` | Declara +1.107/-136 em 23 arquivos/14 commits; entrou +196/-26 em 4 arquivos |
| `423` | Viewer: draw the game's own vehicle liveries | `ac4fd64b` | +297/-21 em 9 arquivos; bate |
| `422` | Meta reader: decode every member type the meta files use | `2970355d` | +84/-25 em 3 arquivos; bate |
| `421` | Body shape: head scale is two-sided | `fae4801a` | Declara 16 commits; entrou +13/-7 em 2 arquivos (o #408 vinha junto) |
| `420` | Viewer: per-piece hair colours that can be linked | `ed17ecec` | Declara +755/-79 em 16 arquivos/14 commits; entrou +218/-32 em 3 arquivos |
| `419` | Characters: leave out items that name a first-person mesh (SIX-229 follow-up) | `a11ba3d4` | +28/-0 em `pedlook.rs`; bate |
| `418` | Viewer: a picked hair colour overrides the stored hair colours | `6fe3a138` | +17/-2; bate |
| `417` | Textures tab: keep the preview beside the Used by panel | `2461a0e9` | +5/-0; bate |
| `416` | Hair noise and curl: the game's frame (in-plane root axis, y = T x axis) | `588c680b` | +52/-22; bate |
| `415` | Viewer: exit cleanly after a GPU device loss (SIX-234) | `02456c4d` | +98/-5 em 4 arquivos; bate |
| `414` | feat(metal4): add GPU-ready model buffer cache via Metal IO | `a8d77218` | +254/-2; bate. Único commit da janela de autor externo (maple-quartz18) |
| `413` | Viewer: the paint shader's HSV colour step (albedo multiply / screen, sun bleach) | `4b2333fd` | Declara 11 commits; entrou +135/-6 em 4 arquivos (inclui o #410, já na `master`) |
| `411` | Inland water: decode the water height texture and check it per tile | `d0ced331` | +55/-4; bate (3 commits no PR) |
| `409` | Placements: archetype kind (asset type and MapEntityType) per entity | `e123f84a` | +70/-3 em 3 arquivos; bate (4 commits, empilhado no #404) |
| `408` | Viewer: Shape details sliders for the 11 further body-shape controls | `29bbf3a0` | Declara 13 commits; entrou +50/-10 em 3 arquivos |
| `404` | Placements: priority names; archetype field doc from the executable's parser table | `cf173c2b` | Declara +6.071/-130 em 25 arquivos/18 commits; entrou +36/-4 em 3 arquivos |
| `400` | exeschema: enum value tables and more struct references | `aa4f1451` | +129/-10 em 1 arquivo; bate (o #394 vinha junto e já está na `master`) |

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum dos 26 commits fecha um defeito do catálogo SixNG (D-01 a D-24). Os mais
  próximos:
  - **D-13** (tint de prop aplicado a shaders de veículo, corrigido em `8e8cb9b`): `9f89a24f` fixa a
    **geometria** da paleta (linhas e índice), não a seleção de shader. **Não corrige.**
  - **D-11** (valores de parâmetro de material no offset errado, `6b68f75`) e **D-22/D-24** (mips e cor
    de configuração): `2b00c691` acrescenta os defaults do jogo **por cima** do que preset e material
    base deixam vazio; não relê offsets de parâmetro. **Não corrige.**
  - **D-05** (instâncias de prop-batch decodificadas como lixo, `6b68f75`): `cf173c2b` e `e123f84a`
    corrigem a **documentação** e acrescentam campos de arquétipo; não tocam a decodificação de
    instância. **Não corrige.**
- **Abre defeito novo?**
  - `a11ba3d4` registra uma **regressão** clara, e ela não está no catálogo: o #384 (insert anatômico)
    mudou a resolução do item de roupa de Lucia, e a malha de primeira pessoa tomou o slot de corpo
    superior. Foi corrigido no mesmo commit, mas o mecanismo (fallback de textura casando item por
    textura quando o GUID é de malha de primeira pessoa) continua sendo um caminho frágil.
  - `4daafc38` documenta outro: o insert masculino carrega **todas** as regiões de vértice em 0, então
    nenhuma máscara de roupa consegue alcançá-lo por região; o conserto depende de replicar a máscara da
    camada de pele. Se um insert futuro tiver regiões parciais, o mesmo buraco volta.
  - `02456c4d` registra que a perda de dispositivo ainda terminava em **código 101** por pânico do
    egui-wgpu 0.36.2 no paint seguinte, mesmo depois do #390 — defeito de integração que não estava no
    catálogo e que este commit contorna com panic hook, não com correção da causa.

## Confiança

Verificado: os 26 SHAs em `repos/holdmysocks/SixNG/commits?sha=master&per_page=100`, com o cursor
`e0fd6ffa` na posição 27; cada commit lido em `commits/<sha>` (arquivos, contagens e mensagem completa);
os 26 PRs mesclados em `pulls?state=merged&search=merged:>=2026-11-26T22:01:14Z`, cada um aberto em
`prs view` com `mergeCommit`, `baseRefName`, arquivos, contagem de commits e corpo; os 10 PRs abertos de
`gh pr list --limit 50`, todos acima de 416. O casamento 1:1 entre commit e PR foi conferido pelo número
no assunto do commit e pelo `mergeCommit.oid` do PR.

Não verificado: o mecanismo além do que o corpo do PR e o patch descrevem — os patches foram lidos com
corte de 400 caracteres por arquivo (os 10 maiores), então arquivos menores de commits grandes não foram
abertos; não executei `cargo test` nem o visualizador; não verifiquei nenhuma das capturas de tela,
contagens de paletas (127.287), de enums (919/3.253/34.123) ou de decorações (7.509) além do que os
corpos afirmam. A divergência entre o tamanho declarado e o tamanho do squash nos PRs empilhados foi
medida nos dois lados (corpo e `commits/<sha>`), não inferida. Nenhum teste foi executado aqui; os 13
repositórios monitorados são somente leitura.
