# holdmysocks/SixNG — atualização de 2026-11-28 14:00

## Resumo

O repositório saiu da fila desta vez: 8 commits desde o cursor `2b2277a5` e 10 merges na janela, dos
quais 3 já estavam documentados e 7 entram aqui. A hora é de veículos e biblioteca: o `#523` nomeia
nove registros de veículo pelo modelo do chassi (715 adesivos de marca passam a ter onde ser
colocados), o `#541` funde o registro do `tankercar`/`freight` no registro vazio do modelo (mais 403
adesivos), o `#496` lê a zona do adesivo do vidro em `_COLOR0` byte 0 em vez de `_COLOR1` byte 3 e o
`#552` exporta um personagem como um único GLB com o traje e a forma de corpo que o visualizador
mostra. O `#554` sobe a versão para 0.15.0. Nenhum hash da janela casa com o catálogo de defeitos.

## Commits novos (7 não documentados, 8 desde o cursor)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `4d9a8e87` | maple-quartz18 | refactor(metal4): extract egui painter crate (#506) | Extrai o pintor egui do Metal 4 para o crate de workspace `crates/egui-mtl4` (624 linhas novas em `lib.rs`, `#![cfg(target_os = "macos")]`, shader `metal_stdlib` embutido, `ArgumentTable`, `RenderPipelineDescriptor`, `ResidencySet`), com `sixng-gui` dependendo por `path`. `metal4_app.rs` troca `metal4_egui::Painter` por `egui_mtl4::Painter` e passa a registrar `MAP_TEXTURE_ID` como textura linear; `THIRD_PARTY_NOTICES.md` é enxugado (−17/+8) porque o código deixa de ser cópia vendorizada. |
| `2a744586` | Brandon Villar | Vehicles: name a record with no model after its chassis model (#523) | `vehicles.rs` (+67/−3) sobe `FORMAT` de 19 para 20 e acrescenta `name_by_chassis`: para registros com `configurations` não vazias e `model` nulo, olha as peças do slot `SLOT_CHASSIS`, exige que todas nomeiem o mesmo modelo conhecido e que esse modelo não tenha registro próprio, e então move o registro para o hash do modelo, mantendo o antigo em `records_from` (as liveries continuam lá). Devolve `named_by_chassis`; `main.rs` imprime os renomeados junto com `merged_by_handling`. O corpo mede 715 adesivos que passam a ser colocáveis. |
| `076d1986` | Brandon Villar | Liveries: vehicle glass keeps its sticker zone in _COLOR0 byte 0 (#496) | `vehlivery.rs` ganha `zone_in_colour0(shader)` (prefixo `standard_vehicle_glass`), `decal_prim` passa a receber um predicado `glass` e escolhe `(&p.colors, 0)` para vidro e `(&p.colors1, 3)` para o resto; `glb.rs` calcula `zone_in_colour0` por material a partir de `extras.shader_name`/`classification.shader` e usa a mesma escolha. O corpo documenta as zonas do Buffalo (93/94 para-brisa, 97/98 traseiro, 101–108 laterais) e que `_COLOR1` byte 3 só tem 0–3 no vidro. |
| `11004e6a` | Brandon Villar | Library: a character exports as one GLB of the outfit and body shape the viewer shows (#550) (#552) | `model_view.rs` (+70) acrescenta `body_shape_spec` (controles `nome=valor` separados por vírgula), `look_of`/`shows_look_of` (reconhece o id com `#look=`) e `outfit_export`, que enfileira `sixng outfit` com o traje e a forma de corpo mostrados; `library.rs` (+18) mostra o botão "Export (glTF + textures)" só para ids `#outfit:`; `jobs.rs` trata um destino `-o x.glb` pelo diretório pai. |
| `add169df` | Brandon Villar | Viewer: label engine-default texture slots instead of calling them missing (#553) | `glb.rs` (+21): `slot_status` passa a devolver `"engine default (run time)"` quando `texture_fallback.source` começa com `engine default`, e só `"missing"` nos demais casos; teste `an_engine_default_slot_is_not_missing` cobre o caso do padrão do motor, o do valor típico e o sem fallback. |
| `5ef1224e` | Brandon Villar | Vehicles: merge a chassis-named record into its model's empty record (tankercar, freight) (#541) | `vehicles.rs` (+50/−21): extrai `merge_into(dst, src, hid)`, que copia o que `dst` deixa vazio e concatena listas (paletas, grupos de peças) sem duplicar, preserva nome e model info de `dst` e grava `records_from`; `merged` passa a registrar o hash de origem. Faz o `tankercar`/`freight` abrirem com configurações, paletas, peças e liveries (o corpo cita 303 + 100 adesivos). |
| `67fc2742` | Brandon Villar | Version 0.15.0 (#554) | Só versão: `Cargo.toml` de 0.14.0 para 0.15.0 e as três ocorrências em `Cargo.lock`. |

O oitavo commit da faixa, `3b6201a4` ("Name 1,328 more enum values from gta6-namehash matches",
#538), já está documentado no ciclo das 13:00 e não foi reescrito.

## Pull requests abertos (0)

Nenhum PR aberto acima de #541 na coleta.

## Pull requests mesclados (10 na janela, 7 novos)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #554 | Version 0.15.0 | `67fc2742` (13:17:57Z) | Bump de versão do workspace (22 PRs desde 0.14.0); dois arquivos, +4/−4. |
| #553 | Viewer: label engine-default texture slots instead of 'missing' | `add169df` (12:40:56Z) | O rótulo novo e o teste; nenhuma mudança de render (`glb.rs` +21/−1). |
| #552 | Library: export a character as its outfit and body shape, in one GLB (#550) | `11004e6a` (12:38:26Z) | Os três arquivos descritos acima: `model_view.rs` +70, `library.rs` +18, `jobs.rs` +6/−1. O corpo explica que o botão antigo rodava `extract` nas peças `cs_*` do próprio personagem, sem o traje escolhido, sem a forma de corpo e sem seguir o visualizador quando havia `#look=`. |
| #496 | Liveries: stickers on vehicle glass (zones from _COLOR0) | `076d1986` (12:34:13Z) | `vehlivery.rs` +41/−12, `glb.rs` +25/−2, `livery.rs` +2/−1. Corpo: ~100 marcas põem adesivo em vidro, a maioria no traseiro (zonas 97/98). |
| #523 | Vehicles: name a record with no model after its chassis model | `2a744586` (12:33:35Z) | `vehicles.rs` +67/−3 e `main.rs` +2/−2; nove veículos (o semi-reboque `trailers` entre eles) ganham configurações/paletas/liveries no visualizador. |
| #506 | refactor(metal4): extract egui painter crate | `4d9a8e87` (12:28:54Z) | 10 arquivos, +712/−26, com o crate novo `crates/egui-mtl4`. |
| #541 | Vehicles: merge a chassis-named record into its model's empty record (tankercar, freight) | `5ef1224e` (13:05:41Z) | `vehicles.rs` +50/−21; empilhado no #523 e no #515. |
| #538 | Name 1,328 more enum values from gta6-namehash matches | `3b6201a4` (13:03:30Z) | Já documentado às 13:00; não reescrito. |
| #518 | (já documentado às 12:00) | `2b2277a5` | Cursor anterior. |
| #515 | (já documentado às 12:00) | `08137817` | Base do #523/#541. |

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum. Os hashes dos 7 commits e dos 10 merges da janela foram conferidos contra
  `catalogo/defeitos.csv` (305 linhas): zero interseção; nenhum assunto cita `D-xx`.
- **Abre defeito novo?** Sem evidência no diff. Os quatro consertos de dados (`#523`, `#541`, `#496`,
  `#552`) mudam o que é exportado, não o caminho de render. Duas observações de área, como hipótese:
  - `#552` faz o visualizador e a biblioteca compartilharem a escolha de traje; se o id do visualizador
    não tiver `#look=`, a exportação cai no primeiro traje da lista do jogo. É comportamento declarado
    no corpo, não um defeito.
  - `#496` depende do nome do shader (`standard_vehicle_glass*`) para escolher o stream; qualquer
    variação de nome fora desse prefixo continua lendo `_COLOR1` byte 3. Não há teste no diff para
    nomes alternativos.

## Confiança

**Verificado no diff:** os 7 commits (arquivo por arquivo) e os 7 merges novos, com hash de merge e
corpo do PR; as contagens de arquivos e linhas vêm da API.

**Não verificado:** execução de qualquer teste (nada foi rodado nesta coleta); o número de adesivos
(715 e 403) é o que os corpos dos PRs declaram, não foi recalculado; a alegação de que o vidro
"desenhava em lugar nenhum" antes do `#496` não foi reproduzida.

**Limites de cobertura:** 1 dos 8 commits da faixa já estava documentado (`3b6201a4`); 3 dos 10
merges (#538, #518, #515) também. `last_run` avança até `#554` (13:17:57Z), o maior `merged_at`
coberto. Nenhum subagente neste ambiente; leitura apenas na API do repositório.
