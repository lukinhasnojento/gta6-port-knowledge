# holdmysocks/SixNG — atualização de 2026-11-27 01:00

## Resumo

Dez commits na `master` (`912287bd..aa6345c8`), todos squash de PRs, num intervalo de 33 minutos. É a
maior rajada de recursos de gente e água do acervo: mapas de spec de pele (#435), `sixng water --pools`
(#436), peças de veículo com nome e de barco (#437), redes de expressão facial como morph targets do
glTF (#438), o desfoque de cabelo em espaço de tela (#439), livery em todo modelo com as zonas (#440),
`tint_weight` (#441), tatuagem com as palavras de tinta decodificadas (#442), bainha de roupa lisa em
corpo pesado (#443) e o assentamento dos fios pela simulação do jogo (#444). Um dos commits mexe no
formato de saída `placements` sem mexer na documentação que o descreve — registrado abaixo como
candidato a defeito.

## Commits novos (10)

Cadeia linear conferida pelos pais: `912287bd → e985401a → 55412aa6 → 461455a7 → a4cff8d4 → 2dfd75f1 →
b9bf3cd3 → 41359d77 → 9e0a3980 → 399ca89a → aa6345c8`, um pai cada.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `e985401a` | holdmysocks | Body, hand and feet skin spec maps from the wardrobe material assets (#392, part 1) (#435) | 5 arquivos, +353/-1. `crates/sixng-core/src/skinspec.rs` novo (281 linhas): monta do índice de `packs.rpf` a tabela mapa de cor → `Material` → mapas de spec, cacheada em `<work>/index/skin_spec.json`, e resolve as chaves `179eb44a` (`speculartex`) e `4cf41ef4` (`speculartex2`) nos 47 conjuntos de assets PSO. `export.rs` (+32) põe os mapas nos slots que os meshes `hand_*`/`feet_*` deixavam com placeholder e grava `spec` no registro de pele da peça em `model.json`; `material.rs` (+7) declara os papéis dos slots dos dois shaders de pele. README (+32) documenta os canais lidos no `PS_DeferredPedTexturedNC`: R de `speculartex` é máscara de translucidez (vai a um alvo próprio do G-buffer), G é rugosidade. Mapa de cor que nomeia mais de um material (10 casos, defaults compartilhados) fica de fora em vez de ser adivinhado. |
| `55412aa6` | holdmysocks | Water: export pools, hot tubs, fountains, waterfalls and standing water with `sixng water --pools` (#436) | 3 arquivos, +316/-33. `water.rs` (+308) ganha `Options::pools`, `pool_surfaces`, `check_pools`, `write_pools` e `placed_mesh` (LOD 0 do drawable, posicionado pela entidade do mapa, filtrando as geometrias pelo shader: `water_micro_geom`, `standard_alpha_water`, `standard_alpha_waterfall` ou qualquer shader do *bucket* de água). Escreve `pools/pools.json` e `pools/pools.glb`. `main.rs` (+7/-2) acrescenta `--pools`. Contagem declarada no corpo: 2.283 posicionamentos de 2.259 drawables — 1.353 piscinas, 500 águas paradas, 119 efeitos, 94 banheiras, 54 tanques, 41 fontes, 17 cachoeiras, 16 valas, 3 lagoas. |
| `461455a7` | holdmysocks | Fit vehicle parts whose names are known, and boats' parts (#437) | 2 arquivos, +10/-7. Em `vehexport.rs` sai o filtro `hex_name` (só peças de nome hash de 8 dígitos eram encaixadas): agora toda peça da configuração entra no laço principal e nos modelos de slot, e rodas/pneus/discos/cáliper continuam indo para as rodas. Em `model_view.rs` o encaixe passa a valer para qualquer fragmento com osso `chassis`, não só os que têm `wheel_mounts` — é o que faz o barco receber os quatro motores de popa. |
| `a4cff8d4` | holdmysocks | Head shape keys: face expression networks as glTF morph targets, viewer Expression panel (#438) | 21 arquivos, +768/-20 (dos quais `facegraph.rs` novo, 502 linhas). Lê o par de recursos anônimos do `packs.rpf` (tipo 95 `ComputeGraph` + tipo 114 `ComputeGraphConstants`, casados pelo id em sys+0x48 / sys+0x20), os registros de op (`0x0b`, `0x2e`, `0x2f`, `0x35`, `0x36`, `0x3c`, `0x43`, `0x44`, com `record_len` fixo por op) e exige exatamente dois `Gather` (entradas e saídas) e ao menos um `MatMul`/`Transpose_MatMul`; sem ativações nem vieses, a rede é linear e cada controle é uma shape key exata, avaliada como mapa linear `basis[i * outputs + k]`. O mapa vértice→saída vem do stream 8 (formato `0x8c`) do mesh LOD0. `gltf.rs` (+41) escreve os alvos como acessores VEC3 **esparsos**, com `weights` zerados e `extras.targetNames = face_000…`; `glb.rs` (+49) ganha `vec3_maybe_sparse` para ler de volta; `model_view.rs` (+99) traz o painel Expression, `--expr nome=peso` e `morph_vertices`, que reescreve os vértices e reconstrói a normal a partir da variação das normais por face; `render3d.rs` (+14) passa o buffer de vértice a `COPY_DST` e ganha `write_vertices`. Os outros 14 arquivos são inicializadores de `TriPrim`/`Mesh` com `morphs: Vec::new()` em testes e construtores. |
| `2dfd75f1` | holdmysocks | Viewer: the game's screen-space hair blur (#439) | 4 arquivos, +397/-0. `hairblur.rs` novo (297 linhas): WGSL com gaussiana 3×3 (pesos 1 / `exp(-1/2s²)` / `exp(-1/s²)` para centro/aresta/canto), `s = mix(s_far, s_near, sat(0.0002·H/z))` com `z` a profundidade reversa da amostra de fio mais próxima, `s_near = clamp(x,0.1,2)` e `s_far = clamp((0.3+0.2·sat(H/720-1))·x, 0.1, s_near)`. O pixel só é borrado se algum vizinho 3×3 for fio. `render3d.rs` (+91) cria `pipe_hair_mask` com `depth_compare: Equal`, sem escrita de profundidade, alvo `Rgba16Float` MSAA, e `record_strand_mask` redesenha exatamente as peças de fio do passe principal; `shader.wgsl` (+8) devolve `(1, p.z, 0, 1)`. `SIXNG_HAIR_BLUR=0` desliga e `=mask` mostra a máscara. |
| `b9bf3cd3` | holdmysocks | Game liveries: a sticker on every model with its zones; pictures keep their own colours (#440) | 1 arquivo, +20/-13, em `livery_ui.rs`. `brand_stickers` só copia as três cores do layout quando `s.mask` é verdadeiro (1.899 dos 2.444 stickers listam três cores zero e saíam pretos). `put_on_brand` deixa de escolher o modelo com mais zonas e passa a clonar cada sticker para **todo** modelo que carregue uma de suas zonas (zonas 0, 63 e 255 não contam; sticker sem outra zona fica como está). |
| `41359d77` | holdmysocks | model.json: tint_weight, how much of the tint palette the game applies (#441) | 2 arquivos, +66/-2. `export.rs` ganha `tint_weight_json` com as quatro constantes lidas no shader (`739e3d0d` = `speculartex2`, `381999f3` = força, `73511514` = suavização, `b1b16aaf` = cor alvo) e publica `status`, `strength`, `smooth`, `smooth_target`, `mask` (PNG e hash da máscara, canal alpha) e `full_palette`; a fórmula vai em texto: `w = 1 - (1 - a)·strength`, com o termo de `smoothstep` e a mistura para a cor alvo. Sem paleta devolve `null`; com paleta mas sem os controles devolve `status` dizendo que não foi decodificado para aquele shader. Um teste novo fixa os três casos. |
| `9e0a3980` | holdmysocks | Tattoos: decode the ink words and bake ink-coloured decorations (#442) | 2 arquivos, +275/-65. `peddeco.rs`: `VERSION` vai a 2 e entra `BAKE_RULE = 2` (chave do cache de textura assada); a palavra de tinta é lida como `0xAAVVSSHH` (hue, saturação, valor, alpha) e `ink_stops` produz três paradas em sRGB — `0.1V·lerp(1,ramp,S)`, `V·lerp(1,ramp,S)` com alpha A e `(0.45+0.55V)·lerp(1,ramp,0.55S)` —, cada uma convertida a linear e quantizada a byte; `gradient` interpola as paradas e o blend vem dos bits `0x04000000`/`0x08000000`/`0x10000000` (multiply/screen/overlay, senão normal). `is_makeup` (quarta componente de `p3`, tabela em `0x6eea8588`) separa maquiagem de cabeça, que continua de fora, de máscara sem tinta, que o jogo colore em tempo de execução. `export.rs` (+10/-14) ajusta os textos `rule`/`ink_rule`, conta `makeup_layers_left_out`/`uncoloured_masks_left_out` e põe `BAKE_RULE` na chave do nome do arquivo assado. O mesmo hunk **remove** `surface_tone`/`surface_tone_note` das peças de veículo e o campo `archetype_kind` de cada entidade de `placements`. |
| `399ca89a` | holdmysocks | Viewer: clothing hems stay smooth on a heavy body (#443) | 1 arquivo, +15/-9, em `model_view.rs`. `Wrap` troca os seis vizinhos de pele mais próximos por `NEAR = 16` e o peso `1/(d² + 1e-5)` por `1/(d² + SOFT²)` com `SOFT = 0.02` m, e o truncamento acompanha; o empurrão pela normal (mínimo do maior offset) fica. |
| `aa6345c8` | holdmysocks | Hair: settle the guides as the game's strand simulation does (#444) | 3 arquivos, +281/-5. `groom.rs` (+272): `SimData` lido do groom (B1 em +0x34 com complacência e dobra em half floats, B2 em +0x38 com o comprimento de repouso, flag de dobra em +0x50 bit 7, amortecimento em +0xF6), com validação de faixa e `None` quando falta; `Colliders::push_out` empurra o ponto para fora das formas/caixas em até 4 rodadas por gradiente numérico; `Groom::settle` parte da estimativa do jogo (`x_k = x_(k-1) + L_k·unit((B_k − B_(k-1)) + c·a·g)`), roda com `DT = 0.05` s e `g = (0,0,−9,8)`, aplica `v = damp(v + g·dt/2)`, a mola para a pose `f = 1/(1 + a·c/dt²)`, o look-ahead do passo e a restrição de dobra (`h = 0,5/(1 + a·c_b/dt²)`, `c_b` = metade alta de B1 + `0,3937·127·d⁸`), e para quando nenhum ponto anda 0,05 mm num passo ou em 300 passos. `model_view.rs` (+7) chama no carregamento, com `SIXNG_HAIR_NO_SIM=1` para desligar. |

## Pull requests abertos (6 novos)

`gh pr list --state open` devolveu 6 PRs acima do cursor 444.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 445 | Vehicle lamp lights: same lamp colours on every run | Torna determinística a escolha de cor das luzes de lampada (ordem estável na geração). | Não. |
| 446 | Skin spec maps as glTF channels and in the viewer's skin shading (#392, part 2) | Publica os mapas de spec da parte 1 como canais do glTF e os usa no sombreamento de pele do visualizador. | Não. |
| 447 | Liveries panel: fit a new sticker to the largest UV island of its zones (SIX-247) | Ao adicionar sticker novo pelo painel, encaixa na maior ilha de UV das zonas em vez do retângulo inteiro. | Não. |
| 448 | Outfit export: an attached accessory goes on its attach bone | Peça anexada (boné, óculos) passa a ir para o osso de encaixe em vez da origem. | Não. |
| 449 | Names: eleven more material parameter and shader constant names (SIX-68) | Acrescenta onze nomes de parâmetro de material/constante de shader à tabela de nomes. | Não. |
| 450 | Vehicle export with its livery: stickers as decal meshes in the GLB (SIX-248) | Exporta os stickers de livery como malhas de decalque no GLB do veículo. | Não. |

## Pull requests mesclados (10)

Todos os dez são squash: o `merge_commit_sha` de cada PR é o próprio commit da `master`, com um pai.

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #435 | Body, hand and feet skin spec maps from the wardrobe material assets (#392, part 1) | `e985401a` | O PR promete os mapas de spec vindos dos assets de material; o commit traz `skinspec.rs`, o cache `<work>/index/skin_spec.json`, os papéis de slot dos dois shaders, `spec` no `model.json` e a leitura dos canais de `PS_DeferredPedTexturedNC`. O corpo diz 6.793 materiais / 2.639 mapas de cor e 10 mapas deixados de fora; o diff não traz essas contagens, só o mecanismo — **NAO VERIFICADO** o número. |
| #436 | Water: export pools, hot tubs, fountains, waterfalls and standing water with `sixng water --pools` | `55412aa6` | O que promete e mais o `check_pools` (contra a height map). O corpo lista 2.283/2.259 e as proporções por tipo; o diff não contém as contagens. |
| #437 | Fit vehicle parts whose names are known, and boats' parts | `461455a7` | Exatamente o descrito: sai o filtro de nome hash, entra o critério de osso `chassis`. O corpo cita o Rhode e o Buffalo config 4; não há fixture nova no diff, só a mudança de dois filtros. |
| #438 | Head shape keys: face expression networks as glTF morph targets, viewer Expression panel | `a4cff8d4` | O que promete, com duas diferenças de escopo em relação ao texto do PR: o PR diz "Second half of #392" mas o diff é 21 arquivos (o corpo fala em 35 arquivos e 15 commits), e a rede é **provada linear** (recusa grafo sem 2 `Gather` + 1 `MatMul`), não "todo grafo". |
| #439 | Viewer: the game's screen-space hair blur | `2dfd75f1` | O WGSL do diff implementa os quatro pontos do corpo (vizinhança 3×3 sem parada por profundidade, gaussiana, `s` com `0.0002·H/z`, sigmas de uma força x=1). A variante 1× é a única implementada, como o corpo avisa. |
| #440 | Game liveries: a sticker on every model with its zones; pictures keep their own colours | `b9bf3cd3` | Exatamente o descrito nos dois pontos. O corpo cita "1.899 of 2.444" stickers com cores zero; o número só aparece no comentário do diff, não há contagem derivada no código. |
| #441 | Tint palettes: export how much of the palette the game applies (SIX-245) | `41359d77` | O que promete, com a ressalva declarada de que os shaders de duas camadas usam `381999f3` para outra coisa e ficam com `status` de não decodificado. O teste `tint_weight_reads_the_mask_and_strength` cobre paleta+controles, sem máscara e sem paleta. |
| #442 | Tattoos: decode the ink words and bake ink-coloured decorations | `9e0a3980` | O corpo descreve as três paradas e o blend; o diff traz isso e mais duas remoções de campo (`surface_tone` em peças de veículo, `archetype_kind` em `placements`) que o corpo não menciona. |
| #443 | Viewer: clothing hems stay smooth on a heavy body | `399ca89a` | O PR descreve `NEAR`/`SOFT` e o commit traz exatamente isso — mas **divergência registrada**: a API do PR conta 27 arquivos e 11 commits, e o commit que entrou na `master` mexe em 1 arquivo (+15/-9). O corpo diz que o PR está empilhado sobre o #427; não verifiquei por qual merge os outros arquivos entraram. |
| #444 | Hair: settle the guides as the game's strand simulation does | `aa6345c8` | O que promete: `SimData` lido do groom e o passo até o repouso, com os itens deixados de fora declarados (amortecimento extra do follow-the-leader, raio do ponto contra colisores, vento e a escala por distância do `smh_sim_1`). |

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum demonstrado. **D-20** ("expressão da lista de cores do sticker — defeito de
  branch, nunca chegou ao master") é o mesmo assunto do `b9bf3cd3`, mas o próprio catálogo diz que o
  defeito nunca esteve no master: o commit é onde o código chega ao master **já com** a guarda `s.mask`,
  não uma correção de defeito publicado. **D-13** (tint de prop aplicado a shaders de veículo) e **D-24**
  (cor de configuração tratada como linear) não são tocados: `tint_weight` lê como o jogo aplica a paleta
  em `prop_batch_dirt`/`standard*`, e nada no diff mexe em shader de veículo ou na conversão da cor de
  configuração. **D-12/D-14/D-15** (cor de peça, carroceria, slot do pneu) são de `vehexport`/`material`;
  `461455a7` muda só **quais** peças são encaixadas, não qual cor cada uma recebe.
- **Abre defeito novo?** Um candidato, verificado por diff e não por execução:
  - `9e0a3980` remove de `export_placements` o bloco que gravava `j["archetype_kind"]` por entidade
    (`export.rs`, hunk `@@ -2687,9 +2686,6 @@`), enquanto o README do mesmo dia — reescrito em
    `a4cff8d4`, o commit imediatamente anterior na cadeia — descreve `archetype_kind {asset_type,
    entity_type}` como campo de cada entidade de `placements`. O README não foi tocado no `9e0a3980`. O
    mesmo hunk remove `surface_tone`/`surface_tone_note` das peças de veículo, sem menção no corpo do PR.
    **Não verifiquei** se outro escritor (o `archetype_lod.jsonl`, que o README também cita) ainda emite
    o campo, nem se a remoção foi intencional.
  - `aa6345c8` muda **o que o visualizador desenha** por padrão (fios assentados) sem flag de
    comparação: `SIXNG_HAIR_NO_SIM=1` é o caminho de volta. Não há teste no diff para `settle`; a
    verificação é o comentário de pesquisa do próprio código.

## Confiança

Verificado: os 10 SHAs e metadados em `repos/holdmysocks/SixNG/commits?sha=master&per_page=100`, com o
cursor `912287bd` na posição 10; cada commit lido em `commits/<sha>` (lista de arquivos, contagens e
patch); a cadeia de pais, um por commit; os 10 PRs em `pulls?state=closed&sort=updated` filtrados por
`merged_at > 2026-11-27T03:03:18Z`, com `merge_commit_sha` igual ao commit da `master` e `commits` do PR
maior que 1 em #435, #436, #437, #438, #439, #442, #443 e #444 (squash de vários commits num só).

Não verificado: a execução de `cargo test`, do visualizador e de `sixng water --pools` (as contagens de
pools, materiais, heads e stickers vêm dos corpos dos PRs e dos comentários do código, não de execução
aqui); se a remoção de `archetype_kind`/`surface_tone` foi intencional; por qual merge entraram os 26
arquivos que faltam do #443; e o comportamento do `settle` em fios reais (o diff não traz teste). Nenhum
teste foi executado aqui; os 13 repositórios monitorados são somente leitura.
