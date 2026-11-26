# holdmysocks/SixNG — atualização de 2026-11-26 01:00

## Resumo

Quatro commits na `master` (`f4c9262a` → `4859e1ff`), todos merges de PR: as luzes posicionadas do mapa
passam a iluminar a cena noturna (`#333`), o registro de screenshots do Map 3D deixa de contar uma vista
como carregada antes dos dados (`#338`), os fios desenhados do cabelo ganham a largura que compensa a
contagem reduzida (`#345`) e as tabelas de deduplicação de textura passam a ser lidas também dentro dos
pacotes de conteúdo (`#346`, que resolve as texturas de gore). Três PRs abertos.

## Commits novos (4)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `2f635c8c` | Brandon Villar | Map 3D: the placed lights shine at night (#333) | Um pai (`f4c9262a`), 7 arquivos, +441/−6. `crates/sixng-gui/src/lights.rs` é novo (+137): `N = 64` células de `CELL = 12,0` m (768 m em volta da câmera), `PER_CELL = 32`, `MAX_LIGHTS = 16384`, `MAX_INDICES = 262144`; `pack()` filtra por alcance e luminância e ordena por `lum * range`. `atmo.rs` (+46) acrescenta os bindings 22/23 (storage, fragmento) e buffers zerados; `render3d.rs` (+44) cria o pipeline `glow` (blend aditivo, `depth(false, GreaterEqual)`) e `set_lights`; `map3d.rs` (+68) tem `gather_lights`, que só recalcula quando a câmera anda 8 m ou a hora muda, aplica `mask & (1 << hour)` e o bit noturno; `shader.wgsl` (+128) acrescenta `local_e()` (inverso do quadrado com janela até o alcance, `LIGHT_K = 0,65` lux a 1 m) e `map_finish_added()` para as coroas (`GLOW_K = 0,12`); `main.rs` aceita `--no-placed-lights`; `Map3D.placed_lights` é **true** por padrão e o README documenta as escolhas. |
| `5ef4ea2f` | Brandon Villar | Map 3D screenshots: don't count a view as loaded before the map data is in (#338) | Um pai (`2f635c8c`), +6/−2 em `map3d.rs`: `frames_ok()` passa a exigir `self.data.is_some()` e, com terreno visível, `self.world.ready()` antes de `self.sectors.idle()`. Antes, uma fila vazia antes de os dados chegarem contava como vista carregada e a captura saía sem o mapa. |
| `c5830e3c` | Brandon Villar | Hair: drawn strands keep the piece's cover under the count rule (#345) | Um pai (`5ef4ea2f`), +21/−2. `render3d.rs` ganha o campo `strand: [f32; 4]` em `MatU` e, em `render`, para cada malha com `strand_lod`, calcula `k = (n_idx / indices(...)).clamp(1,0, 50,0)` e grava em `MatU.strand.x`; a visibilidade do binding 0 sobe para `VERTEX_FRAGMENT`. `shader.wgsl` usa `max(M.strand.x, 1.0)` como fator de largura do fio, exceto no passe de sombra do sol (`F.frame.y > 1.0e8`). Assim um pedaço de cabelo que desenha menos fios mantém a cobertura. |
| `4859e1ff` | Brandon Villar | Textures: read every de-duplication table (gore textures), cached (#346) | Um pai (`c5830e3c`), 3 arquivos, +131/−24. `ttpf.rs` (+73/−22): `parse_dupes()` passa a exigir tabela ordenada e não vazia (rejeita o magic `EDUP` que aparece por acaso em outros dados) e `find_dupes()` procura o magic **dentro** de arquivos pequenos (bin/xml/text < 1 MiB) de `textures.rpf` e dos pacotes de conteúdo, somando as tabelas; `core_restofworld` traz a segunda (33 pares, texturas de gore) apendada ao XML de setup do pacote. O resultado é cacheado em `<work>/index/texture_dupes.json`, reconstruído quando o índice muda. `bodyskin.rs` (+57) ganha os subcomandos `missing` e `heads` para listar o que continua sem resolução. README: 17.464 pares no total, 7.789 dos 391.040 hashes referenciados resolvidos pelas tabelas (16 a mais que `#332`), 620 restantes. |

## Pull requests abertos (3)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 350 | Viewer: the primer's colour from the material palette | Só `holdmysocks`, +187/−17 em 6 arquivos (`vehcolour.rs` +106, `model_view.rs` +62, `shader.wgsl` +10/−6). O primer deixa de ser cinza de stand-in e vem da paleta do material: seis entradas por registro de cor (2 tipos × 3 conjuntos), tipo lido do `primerType` do registro de superfície (`78b545dd`, u8 em +0x3c; 563 superfícies com 0, 194 com 1); conjunto com linha −1 cai no conjunto 0 do mesmo tipo; a janela de gradiente usa colunas 46 e 47 da rampa. Exporta `part_colours.primer` (formato 24, versão 29) e adiciona o slider "primer set" e `--primer-set`. Declara fora de escopo roughness/metalness do primer e o uso da linha global `primerPalette`. | Não é D-xx. A área é pintura de veículo; `D-24` ("cor da configuração tratada como linear quando é valor de display — pintura escura", corrigido `2f67255`) é conversão de cor, não escolha de paleta. |
| 349 | Cutscene characters: clothing sets on the model that wears them, names confirmed by scene peds, outfit for models without own parts (#306) | +171/−14 em 3 arquivos (`pedlook.rs` +140). Prefixos de personagem de cutscene sem cabeça passam a herdar os outfits do modelo que veste suas peças (só conta se a peça do próprio prefixo entra, ou duas peças num conjunto sem cabeça; um acessório compartilhado não basta); nomes são confirmados contra os peds das cenas (novos `cs_felixtheten`, `g_m_m_cashcity_00`, `g_f_m_polillanegra_00` etc.); `sixng outfit` exporta outfit 0 para modelo sem peças próprias. Contagens: 585 → 596 modelos de outfit nomeados, 2.452 → 2.435 entradas de elenco sem modelo. | Não é D-xx. |
| 348 | Hair export: scalp hair pressed under the outfit's hat | **Draft**, +257/−39 em 6 arquivos (`hairexport.rs` +172, `scripts/blender_hair_curves.py` +33/−10). `outfit --hair` pressiona as peças de couro cabeludo sob o chapéu do outfit (3 mm abaixo do forro) usando `groom::Shell` do viewer, antes e depois de crescer os fios; `hairexport::hat_shell` constrói a casca a partir do GLB do modelo. Mede 480.455 de 1,19 M pontos movidos no look 1 do Jason e ~0,1 % de pontos até 12 mm fora do forro. | Não é D-xx. |

## Pull requests mesclados (4 nesta faixa, 1 com `mergedAt` posterior ao cursor)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| 333 | Map 3D: the placed lights shine at night | `2f635c8c` (mergedAt 2026-11-26T03:09:33Z) | O commit squash, com os sete arquivos descritos acima. O PR prometia luzes posicionadas com máscara de hora e coroas; entrou isso e o `--no-placed-lights`. Não entrou nada de spot cone nem expoente de queda — o próprio PR declara que todas as luzes brilham como pontos. |
| 338 | Map 3D screenshots: don't count a view as loaded before the map data is in | `5ef4ea2f` (mergedAt 2026-11-26T03:12:24Z) | Exatamente o `frames_ok()` com as duas condições novas (dados presentes, pirâmide de altura pronta com terreno visível). |
| 345 | Hair: drawn strands keep the piece's cover under the count rule | `c5830e3c` (mergedAt 2026-11-26T03:15:15Z) | O fator de largura por peça, gravado por `queue.write_buffer` em `MatU.strand.x`, aplicado no vértice e excluído do passe de sombra do sol. |
| 346 | Textures: read every de-duplication table (gore textures), cached | `4859e1ff` (mergedAt 2026-11-26T03:22:55Z) | `find_dupes()` sobre arquivos pequenos de todos os pacotes, validação de tabela ordenada, cache em `index/texture_dupes.json` e os subcomandos `bodyskin missing`/`heads`. O PR prometia as texturas de gore; entrou a leitura de qualquer tabela em qualquer arquivo pequeno, que é mais amplo, e a contagem declarada (7.789 de 391.040). |

## Cruzamento com o catálogo

- Corrige D-xx? Nenhum. As 24 linhas de `SixNG` tratam de RAGE/RPF, normal map, BC6H/BC1, eixos de
  posição, cores de peça e de pneu, strings de text table, meshlets, mips, UV de ground, cor de display.
  Nenhuma descreve luz posicionada do mapa, screenshot do Map 3D, largura de fio de cabelo ou tabela de
  deduplicação de textura.
- Abre defeito novo? Um candidato real: a segunda tabela `EDUP` (`core_restofworld`, 33 pares) ficava
  fora do índice porque estava apendada ao XML de setup e era classificada como XML — o sintoma eram
  texturas de gore ausentes. O `#346` descreve isso como "o que faltava" e o corrige; o catálogo não tem
  linha. Vale entrada nova com introduzido/corrigido em `4859e1ff`.
- Corrige parcialmente uma linha? `#346` também reduz a lista de 620 hashes ainda sem resolução, mas
  declara que quase todo o "111 K faltando" de `#332` eram paletas de tint embutidas nos modelos. Não é
  `D-02` (TOC do RPF9 nunca lido) nem `D-22` (contagem de mips): é insumo de textura, não índice.
- Material de cor: `#350` (aberto) mexe em cor de veículo, área de `D-24` (`2f67255`), e declara deixar
  roughness/metalness do primer para depois — se mesclado, `D-24` continua valendo e o texto do catálogo
  não muda.

## Confiança

Verificado por diff: os quatro commits foram lidos em `gh api .../commits/<sha>`, todos de um pai, em
cadeia `2f635c8c → 5ef4ea2f → c5830e3c → 4859e1ff`, com `f4c9262a` (o `last_commit` anterior) como pai
do primeiro. PR `#346` conferido com `merge_commit_sha = 4859e1ff`.

Divergência registrada conforme o procedimento: a API de PR mesclados com `merged:>=<last_run>`
(2026-11-26T03:16:53Z) devolveu **um** merge (`#346`), mas a janela de commits tem **quatro**. Os outros
três (`#333` 03:09:33Z, `#338` 03:12:24Z, `#345` 03:15:15Z) mesclaram antes do cursor `last_run` e depois
do `last_commit` anterior, porque a execução de 00:00 gravou o estado em 03:16:53Z sem ter visto esses
commits. Conferi cada `merged_at` por `gh api repos/holdmysocks/SixNG/pulls/<n>`; nenhum merge ficou sem
linha, então a contagem fecha em 4 = 4.

Não verificado: nenhuma execução do viewer, do Map 3D ou do exportador foi reproduzida; as contagens
(17.464 pares, 7.789 hashes, 596 outfits, 480.455 pontos) são as que os PRs declaram. Não abri o log de
CI nem o `sixng texture-gaps`.
