# holdmysocks/SixNG — atualização de 2026-11-27 23:00

## Resumo

9 commits e 9 PRs mesclados acima do cursor `9346d0e6` — todos squash de um pai, casando 1:1 com os
merges da janela. É trabalho de visualizador e exportador, não de decodificação: assentamento de guias de
cabelo contra o corpo, mapas de oceano na grade de água, culling de decalques e malhas see-through,
filtragem do detalhe de pele dentro do tile, MSAA configurável no caminho Metal 4, dois consertos de UI de
livery e o limite do slider de escala da cabeça. 1 PR aberto (`#500`).

## Commits novos (9)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `a31b51a2` | Brandon Villar | Hair export: settle the guides against the body, as the viewer does (#494) | `hairexport.rs` +163/−15: `Options` ganha `settle: bool` (padrão `true`) e `piece_with` recebe `colliders: Option<&Colliders>`; antes de crescer as mechas, assenta as guias com `Groom::settle_in` (gravidade, comprimentos de repouso, retorno à forma guardada) fora dos colisores do corpo (`character_colliders`: peças redondas e caixas do bound do corpo e de cada roupa, na pose de bind, senão os vértices não-cabelo), mantém cabelo de couro cabeludo sob chapéu dentro da copa e amortece o passe de estilo perto dos colisores (`Groom::coat_with`). `--hair-no-settle` mantém as guias guardadas. `model_view.rs` perde 19 linhas (a construção do bound sai de lá) e `main.rs` ganha o switch |
| `89c9fa07` | Brandon Villar | water: place the ocean maps on the water grid (--detail writes ocean/) (#495) | `water.rs` +163: `write_ocean` grava `ocean/` com `mean_sea_level`, `water_quality_control`, `ocean_wave_control`, `ocean_roughness_control`, `ocean_wetness_control` (PNG, 16 bits para o nível do mar) no box de mundo da grade, 512×1056 a 32 m/texel ou 1024×2112 a 16 m; `distance_to` faz chanfro de 8 vizinhos em dois passes para as médias por faixa de distância da terra (0–200 m, 200 m–1 km, 1–5 km, > 5 km) e confere contra o mapa de altura da água e as superfícies interiores exportadas |
| `23964175` | Brandon Villar | Map 3D: cull decal and see-through meshes one by one too (#491) | `render3d.rs`: `MESH_CULLED` passa de 3 para 4 contadores (main, fb, shadow, late) e `record_joined` recebe `shown: bool` por malha; malha escondida que segue a run corrente estende `reach` (fim das escondidas contíguas) para que a próxima visível ainda entre na mesma run — o culling nunca divide um draw. Só vale para modelos colocados ≤ `CULL_MAX_INST` (8) |
| `e607e6d3` | Brandon Villar | Viewer: a switch for the skin detail maps (#498) | `Look.skin_detail` (padrão on, `SIXNG_SKIN_DETAIL=0`/`off` desliga), checkbox na seção de corpo e `--skin-detail off`. No frame, `mode[1] = rma_order \| if skin_detail { 0 } else { 256 }`; no `shader.wgsl` o bit 8 é lido em `skin_det_on`, e **todos** os usos de `F.mode.y` como ordem de canal passam a mascarar `& 255u` (`< 3u`, `== 0u`, `== 1u`) |
| `b2effe97` | Brandon Villar | Skin detail: keep the sheet filtering inside each tile (#497) | `skin_tile` troca `textureSampleGrad(…, q, gx/4, gy/4)` por um LOD isotrópico único: `lod = clamp(log2(max(len(gx),len(gy))*256), 0, 8)`, `h = 0.5*exp2(ceil(lod))/256` e a posição presa em `[h, 1-h]`, com `textureSampleLevel`. A nota de layout em `skinspec.rs` (`detail_json`) passa a dizer o mesmo para outros renderizadores |
| `d2607d4b` | maple-quartz18 | feat(metal4): add configurable MSAA resolve (#487) | `Cargo.toml`/`Cargo.lock` sobem o `patch` do `metal-rust` de `e67e0fe6` para `0aa2a1f6`; `render3d_metal4.rs` ganha `MSAA_SAMPLE_COUNTS [1,2,4,8,16]`, `supported_msaa_sample_count` (maior suportado ≤ pedido, senão 1) e `effective_msaa_sample_count` (1 quando o temporal upscaling está ligado); a UI ganha um combo `MSAA n (Nx)` que mostra "1x fallback"/"Off" e aplica via `set_msaa_level` |
| `0c0e2524` | Brandon Villar | Liveries panel: Ctrl + wheel resizes the sticker again (#499) | `livery_ui.rs`: com Ctrl pressionado o egui trata a roda como zoom e reporta `smooth_scroll_delta` zero; o código reconstrói o scroll de `zoom_delta()` com `ln()*200` (a convenção do egui) e trata o gesto como Ctrl também com a tecla já solta, porque o zoom continua por alguns frames |
| `16d77ec9` | Brandon Villar | Liveries panel: open the panel and the picker clear of the vehicle (#501) | `livery_ui.rs`: `Editor.panel_rect` guarda o retângulo do painel; a janela passa a abrir em `within.left() - 340 - 12` quando a vista tem menos de 900 px de largura e há espaço à esquerda da área, em vez de sempre sobre o veículo, e é restringida a `window_area` |
| `ede2c253` | Brandon Villar | Body shape: head scale stops at half size (#502) | `bodyshape.rs`: `HEAD_SCALE_MIN = -0.5` e `control_min(name)` (0.0 para controles sem lado negativo, −0.5 para `gcheadscale`, −1.0 no resto), usado por `parse_spec` e pelo slider; o comentário documenta que a rig escala a cabeça por `1 + v` (2,0 em +1 e 0,01 em −1). Os testes passam a exigir `0x5ebebbe7 = 0.5` e aceitam `gcheadscale=-0.3` |

## Pull requests abertos (1)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| `#500` | Map 3D: cull placements first, and models placed up to 64 times | Culling por placement antes do por-malha e teto de instâncias de 8 para 64 no culling por malha | Complementa `#491` (`23964175`), que só cobre ≤ `CULL_MAX_INST` = 8. Não é defeito do catálogo; é limite conhecido do culling atual |

## Pull requests mesclados (9)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| `#494` | Hair export: settle the guides against the body, as the viewer does | `a31b51a2` (2026-11-27T19:14:41Z) | `settle_in`/`coat_with` no export, colisores construídos do próprio modelo escrito, `--hair-no-settle`, e o `body_bound` movido do viewer para `hairexport` |
| `#495` | Water: place the ocean maps on the water grid | `89c9fa07` (19:15:45Z) | `write_ocean` + `distance_to` + as cinco entradas de `OCEAN_MAPS`; só roda com `--detail` |
| `#491` | Map 3D: cull decal and see-through meshes one by one too | `23964175` (19:23:52Z) | O quarto contador, `shown` por malha e a lógica de `reach`; a linha de tick vira `[main, fb, shadow, late]` |
| `#498` | Viewer: a switch for the skin detail maps | `e607e6d3` (19:31:50Z) | O bit 8 de `mode.y`, o checkbox, `--skin-detail off` e a máscara `& 255u` em todos os leitores de ordem de canal |
| `#497` | Skin detail: keep the sheet filtering inside each tile | `b2effe97` (19:32:13Z) | `textureSampleLevel` com um LOD isotrópico e a posição presa a meio texel da borda do tile |
| `#487` | feat(metal4): add configurable MSAA with resolve | `d2607d4b` (19:38:51Z) | Bump do `metal-rust` para `0aa2a1f6` e o combo de MSAA no caminho Metal 4, com fallback para 1x |
| `#499` | Liveries panel: Ctrl + wheel resizes the sticker again | `0c0e2524` (19:39:07Z) | Reconstrução do scroll a partir de `zoom_delta()` |
| `#501` | Liveries panel: open the panel and the picker clear of the vehicle | `16d77ec9` (19:56:43Z) | `panel_rect`, posição à esquerda da vista quando ela é estreita, `constrain_to(window_area)` |
| `#502` | Body shape: head scale stops at half size | `ede2c253` (20:04:52Z) | `HEAD_SCALE_MIN`, `control_min`, clamp em `parse_spec` e no slider |

## Cruzamento com o catálogo

- **Nenhum commit ou PR da janela corrige defeito do catálogo.** Os 24 defeitos de SixNG são de
  decodificação de recurso (RAGE/RPF/BC6H/BCPack, UV, cor de configuração) e de fluxo de carregamento; a
  janela inteira é visualizador, exportador e UI.
- **D-22 (contagem de mips subcontada pelo descritor) e D-23 (UV das camadas de ground em 4 camadas,
  64 por unidade em vez de 128) — não tocados.** `b2effe97` mexe em amostragem de atlas de detalhe de
  pele, que não está no catálogo; a classe (filtragem atravessando a borda do tile de um atlas lido como
  sheet em vez de texture array) não tem ID.
- **D-03 (normal map em frame tangente errado) — não tocado**, apesar de `skin_detail` usar a normal do
  detalhe: o código só aplica `strength.y * mask * d` sobre o `xy`, sem mudar o frame.
- **Abre defeito novo?** Provável lacuna, verificada por leitura: o switch `#498` **não tem efeito no
  renderizador Metal 4**. Lido `crates/sixng-gui/src/render3d_metal4.rs` em `ede2c253`, o arquivo não
  referencia `Look.skin_detail` (as duas ocorrências são campos de material `skin_detail: None`, linhas
  4026 e 6608) e tem shader próprio (`MESH_SHADER`), então o checkbox/`--skin-detail off` só muda o
  caminho wgpu. Não é defeito de tradução, é cobertura de renderizador — mas a UI não diz isso.
- **Candidato a regressão, não verificado:** `23964175` mantém a condição de junção em
  `same_pipe && model == mi && mat == me.material` e usa `reach` para atravessar escondidas; se uma
  escondida tiver material diferente no meio, `reach` não avança e a run é dividida — comportamento
  correto, mas não há teste no diff cobrindo o caso de escondida com material diferente entre duas
  visíveis.

## Confiança

- **Verificado por leitura de patch:** os 9 hashes e as afirmações de código acima (`hairexport.rs`,
  `water.rs`, `render3d.rs::record_joined`, `Look`/`shader.wgsl`, `skin_tile`, `render3d_metal4.rs` e a
  UI, `livery_ui.rs` duas vezes, `bodyshape.rs`).
- **Verificado por leitura de arquivo no head:** `render3d_metal4.rs` em `ede2c253` via contents API —
  nenhuma referência a `Look.skin_detail`.
- **Verificado por API:** 9 `merge_commit_sha` idênticos aos 9 commits da faixa (`compare
  9346d0e6...ede2c253`: `ahead_by` 9, `behind_by` 0); 9 merges com `merged_at` 19:14:39Z a 20:04:52Z.
  Commits = merges, 1:1.
- **NÃO VERIFICADO:** nada foi executado (nem `cargo test`, nem o visualizador, nem o exportador); os
  números de culling e de LOD não foram medidos; `#500` e os patches não lidos; o `Fixes`/issues citados
  não foram abertos; não foi verificado se o `metal-rust` `0aa2a1f6` existe de fato — o diff só mostra a
  troca de rev.
