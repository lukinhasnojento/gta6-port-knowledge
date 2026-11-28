# holdmysocks/SixNG — atualização de 2026-11-28 12:00

## Resumo

Janela de 33 commits medida às 12:13Z, 36 na lista paginada (o head avançou durante a coleta) e 35 PRs
mesclados. A hora é quase toda de conteúdo de jogo e de ferramenta de exportação: o bake de tatuagem
mudou duas vezes de regra (`BAKE_RULE` 10→12) e passou a usar o segundo conjunto de UV, a tabela de
cabelo inferida passou a casar por nome de peça, e o mapa 3D ganhou um renderizador próprio com tabela
de material (`gpu_draw.rs`, 1.181 linhas) que evoluiu em cinco commits na mesma hora. Nada foi
executado: as afirmações abaixo vêm do diff.

## Commits novos (36)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `767c2572` | holdmysocks | Names: 8 more shader constant names (SIX-68) (#510) | Acrescenta 8 nomes ao vetor `DERIVED` de `names.rs` (`vertexdisplacementnoisetiling`, `tintroughnessmod`, `grassdecallayer`, `grassdecaluvoffset`, `verttintblendamount`, `tangentuvset`, `clearcoatuvset`, `heightfilter`), achados por prefixo de vizinho nomeado; o comentário declara ~0,14 acertos aleatórios esperados no conjunto. |
| `941bdf97` | holdmysocks | Skin detail: read the region as the high nibble, and test the merge (#514) | Muda a string `layout` de `detail_json` em `skinspec.rs`: a região deixou de ser `u32(16 G + 0.5)` e passou a ser o **nibble alto** do byte (`u32(255 G + 0.5) >> 4`), ou seja, 0–15 em vez de 0–15 por multiplicação — a regra antiga saturava acima de 15. Só a documentação da regra muda; nenhum código de amostragem. |
| `059aed3c` | holdmysocks | Hair link: read the registries by field hash, and retry the stored link (#517) | `hairlink.rs`: novo `FIELD_TYPE = 0x2d4958cf` (campo `asset type`, chamado `shaderId` na tabela do executável) e leitura dos registros pelo hash de campo em vez do offset fixo, com retentativa do link guardado. `meta7a.rs` passa a procurar `tint_row` pelo nome do campo (`field_name(0x3c852527)`) com queda para a chave hexadecimal antiga. |
| `e578e1f6` | holdmysocks | 3D map: material table with shared pools and texture arrays (#519) | Arquivo novo `crates/sixng-gui/src/gpu_draw.rs` (1.181 linhas): desenho dirigido pela GPU do mapa, com geometria em pools compartilhados de vértice/índice, texturas em arrays e uma tabela de material por modelo; `main.rs` registra o módulo. É caminho de render novo, não substituição do antigo no mesmo commit. |
| `ccb4b1a4` | holdmysocks | 3D map: the material-table shader without the sticker part (#520) | No mesmo `gpu_draw.rs`, a reescrita do programa de pixel remove o trecho dos adesivos do shader da tabela de material; o comentário mede o passe principal ~um décimo mais longo com ele, mesmo com a lista vazia. |
| `ea766ef1` | holdmysocks | Link subtitles to their speaker and audio stream (#493) | Arquivo novo `crates/sixng-cli/src/dialogue_links.rs` (562 linhas): subcomando `sixng dialogue-links`, que liga cada linha de legenda ao falante e ao fluxo de áudio, conferindo contra o índice de áudio; `main.rs` declara o módulo. |
| `90e34f9a` | holdmysocks | Names: flipbook and region tint shader constants (SIX-68) (#521) | Mais 7 nomes em `DERIVED`: as cinco constantes consecutivas de `standard_flipbook` (`flipbookrows/cols/count/freq/phase`) e o par `regiontintamount`/`regiontintintensity`. |
| `aecd8349` | holdmysocks | Water: export the small water bodies inside placed interiors (#524) | `water.rs`: `PoolSurface` ganha `interior: Option<PoolInterior>`; quando um interior posiciona a superfície, posição e rotação passam a ser do espaço do interior e a geometria vai para o mundo pela instância do interior. |
| `5b3da3a3` | holdmysocks | Hair export: keep settled hair just outside the body and garments (#525) | `groom::Colliders` ganha `surface: Option<Surface>` — as superfícies do próprio corpo e das roupas; `hairexport.rs` importa `Surface` e usa o *clear* da superfície depois do *settle* para manter o fio logo fora da pele e da peça. |
| `25f6dc79` | holdmysocks | Hair settle: push strands out of colliders only to their stored depth (#526) | `push_out` passa a delegar para um novo `push_out_to(p, 0.0)`; o deslocamento de um fio para fora de um colisor agora para na profundidade guardada, em vez de projetar até a superfície. |
| `9559a01d` | holdmysocks | Dialogue links: name speakers whose hash a known name gives (#522) | `dialogue_links.rs`: usa `BTreeSet` e o dicionário de nomes já resolvidos para nomear um falante cujo hash coincide com um nome conhecido; sem isso a linha ficava com hash hexadecimal. |
| `0377edc6` | holdmysocks | 3D map: the shared mesh shader without the sticker part too (#527) | `gpu_draw.rs` e `render3d.rs`: o shader de malha compartilhado do mapa ganha a variante sem o trecho de adesivos, escolhida quando `hdr`; o mesmo motivo medido do #520, agora no programa de malha. |
| `1417d210` | holdmysocks | Cutscenes: audio banks and master timelines (#516) (#528) | `sixng cutscenes` passa a resolver, por cena, o banco de áudio que contém dois ou mais dos fluxos `joaat("<nome>.<canal>") & 0x1FFFFFFF` (c, l, r, ad, ls, rs, music.l, music.r); entram as estruturas da linha do tempo mestra e o aviso de terceiros `vihook`. |
| `cba86b04` | holdmysocks | Liveries: a sticker laid on flat can be projected from a side of the vehicle (#529) | `livery::Sticker` ganha `from: Side` (serializado só quando difere do padrão `panels`); `--livery-flat` aceita a lateral de projeção e o editor guarda `auto_from`. |
| `34bcb4b9` | holdmysocks | Names: UV transform and blend mask constants the exporters read (#532) | Seis nomes novos em `DERIVED` (`uv0scalebias`, `uv1scalebias`, `uvset`, `controltexturetex`, `controluvset`, `controlfilter`) e os comentários de `material.rs` passam a nomear os hashes `59eb15c5`, `f50070cc` como `controlfilter`/`controluvset`. |
| `a565fd9f` | holdmysocks | Tattoo bake: lay the upper body out in its second UV set (#530) | `peddeco.rs`: o espaço do tronco deixa de ser o **quarto** conjunto de UV (meia largura, `u0/2` e `0.5 + u0/2`, `u` dobrado) e passa a ser o **segundo**; `export.rs` muda o comentário e a chamada de `bake_decorations`. A regra nova não reusa a quarta UV. |
| `71768e8c` | holdmysocks | Water: take the inland height texture's scale from the drawable's stored bounds (#534) | `InlandTile` ganha `bounds: Option<[[f32; 3]; 2]>` (bbox do drawable em espaço de modelo) e a escala da textura de altura da água interior passa a sair dela, em vez de constante. |
| `df295f3b` | holdmysocks | Looks: a cap worn backwards picks the hair's capbackwardshairalt state (#533) | `pedlook.rs`: novo `hair_states_under` que consulta o estado que a peça de cabeça pede; um boné ao contrário (`hatcapbackwards`) pede `capbackwardshairalt` (hashes `1465a09b`, `b411d8df`). `wardrobe.rs` acrescenta `capbackwardshairalt` e `visorhairalt` à lista documentada. |
| `564e93c2` | holdmysocks | 3D map: short vertex program for the material table's shadows (#531) | `gpu_draw.rs`: programa de vértice curto para os sombreadores da tabela de material, com as opções `shadowold`, `vsfull` e `shadowfull` no `watch`; `map3d.rs` passa a imprimir também os milhares de triângulos de modelo por quadro. |
| `9508a81f` | holdmysocks | Cutscenes: master state machines (#516) (#535) | `cutscene.rs` (357 linhas) lê as máquinas de estado das linhas do tempo mestras (tempos de início, transições, alvo, tipos de condição) e `CATALOG_VERSION` sobe de 5 para 6; o CLI ganha `--masters`. |
| `9c526e7a` | holdmysocks | Version 0.14.0 (#536) | `Cargo.toml` e `Cargo.lock`: 0.13.0 → 0.14.0 em `workspace.package` e no pacote `sixng-cli`. |
| `46e14f18` | holdmysocks | Liveries: a sticker projected from a side is dragged as seen from that side (#537) | `ZoneSet::uv_at` deixa de projetar ao longo da normal do painel e passa a projetar pela lateral escolhida; o arrasto roteirizado imprime a zona e a posição atingidas. |
| `f4105aae` | holdmysocks | Tattoo bake: limit a region-flagged record to the part's mesh (#539) | `export.rs` passa um `decoration_corners` a `bake_decorations`, e `peddeco.rs` limitado ao mesh da peça quando o registro traz região; `BAKE_RULE` 10→11. |
| `e12df728` | holdmysocks | Names: 12 shader constant names carried over from the previous engine (SIX-68) (#540) | `names.rs`: nomes da geração anterior de shaders (`umglobalparams`, `usetessellation`, `specularintensitymult`, `specularfalloffmult`, `diffuse2specmod`, `reflectivepower`, …) casados direto contra as ~370 constantes sem nome. |
| `94012be9` | holdmysocks | Looks: tint a colour slot that binds the jewellery tint data map (#542) | `model_view::colour_part` deixa de iterar só `ped_shade` e passa a tingir também material cujo slot de cor liga o *tint data map* de joia. |
| `4bb79778` | holdmysocks | Cutscenes: subtitle lines with text, speaker and bank time (#516) (#543) | `CATALOG_VERSION` 6→7; cada cena lista as linhas de legenda (rótulo, falante, início/fim na cena e o tempo no relógio do banco quando a cena está colocada uma vez), com texto. |
| `7dee3840` | holdmysocks | Tattoo bake: the 0x40 vertex-colour region cap (#544) | `decoration_corners` passa a ler os dois fluxos de cor de vértice (ausente = 255) e aplica o teto de região `0x40`; `BAKE_RULE` 11→12. Terceira regra de bake em três commits. |
| `ac0d9608` | holdmysocks | Inferred hair table: match assets by piece names, clean up pieces (#545) | `hairlink.rs` (173/44): `CACHE_VERSION` 5→6, a tabela de cabelo inferida passa a casar o ativo pelo **nome da peça** e a limpar nomes de peça que não existem. |
| `5bbf3ac7` | holdmysocks | Cutscenes: subtitle times per composite section (#546) | `CATALOG_VERSION` 7→8; o tempo de legenda passa a ser por **seção composta**, não por cena colocada, e a descrição do comando muda no mesmo sentido. |
| `9387fbc9` | holdmysocks | Dialogue links: place generic speech lines by the speaker's speech voice (#547) | `dialogue_links.rs` consome o novo `Voices` do `audiometa` (arquivos de fala, schema 18, em `<work>/index/audio_voices_<archive>.json`) e posiciona linhas de fala genéricas pela voz do falante. |
| `03a2e239` | holdmysocks | 3D map: material table draws decals and see-through surfaces (#548) | `render3d.rs`: `gd_pipes` passa de 4 para 7 pipelines (opaco, fallback, sombra recortada, sombra sólida, decal, decal misturado, translúcido) e `gpu_draw` desenha decals e superfícies translúcidas. |
| `06866d04` | holdmysocks | Dialogue links: report generic-role lines as voice chosen at run time (#549) | Um caso novo em `link_lines`: quando as vozes são conhecidas e nenhuma é do falante (papel genérico — atendente, policial, inimigo), a linha é reportada como voz escolhida em tempo de execução, em vez de "não ligada". |
| `0bb6fa5d` | holdmysocks | Cutscenes: what the state machine's string tests check, and its start transitions (#551) | `CATALOG_VERSION` 8→9; a listagem de máquinas passa a mostrar o que os testes de string checam (nome da cena, seção) e as transições de início. |
| `700627d9` | holdmysocks | Name 963 more enum values (#490) | `enum_values.txt` recebe 966 linhas (963 nomes), de casamento exato das tabelas de enum do executável contra `names_exe.txt`, prefixos de vizinhos nomeados e listas de nome inteiro (input, event, ped-flag, vehicle-bone, weapon-flag); há teste novo para `INPUT_VEH_MOVE_UP_ONLY`. |
| `08137817` | holdmysocks | Vehicles: merge a model's records filed under its handlingId (#515) | `vehicles.rs`: `FORMAT` 18→19; os registros de configuração/paleta/livery arquivados sob o `handlingId` passam a ser fundidos com os do modelo (`guardiantx2`, `dump`, `blazer4`, …) em vez de perdidos. |
| `2b2277a5` | holdmysocks | Liveries: tyre-sidewall stickers (zones 226/227) draw on the fitted tyre (#518) | `vehlivery::decal_prim`: o grupo de uma roda passa a levar o nome do modelo **e o osso** (`tyre_car_001_hi@wheel_lf`), então o adesivo de flanco de pneu vai para as quatro rodas; `livery::Zones::of_with` aceita as peças montadas e `model_view` só libera o editor quando as rodas exibidas terminaram de carregar. |

## Pull requests abertos (3)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #541 | Vehicles: merge a chassis-named record into its model's empty record (tankercar, freight) | Estende o #515/#523: um registro arquivado sob o nome do chassi entra no registro vazio do modelo (trem-tanque, vagão de carga), liberando 403 adesivos de marca. | Não é `D-xx`; é a mesma classe do que `08137817` acabou de fazer para `handlingId`. |
| #538 | Name 1,328 more enum values from gta6-namehash matches | Acrescenta 1.328 nomes a `enum_values.txt` (8.273 linhas), por casamento exato contra o dicionário `gta6-namehash`. | Não. Área de enum, sem catálogo próprio no `SixNG`. |
| #523 | Vehicles: name a record with no model after its chassis model (the semi-trailer trailers and 8 more) | Nove veículos sem configuração/paleta/livery (o `trailers` semirreboque entre eles) porque o registro estava sob um nome sem modelo; renomeia pelo chassi. | Não. |

Os três foram lidos por título e corpo; **nenhum patch de PR aberto foi lido**.

## Pull requests mesclados (35)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #510 | Names: 8 more shader constant names | `767c2572` | 8 nomes em `DERIVED`, como o título promete. |
| #514 | Skin detail: read the region as the high nibble, and test the merge | `941bdf97` | A regra de região (nibble alto) e o módulo de teste do merge em `glb.rs`; a mudança é de documentação da regra, não de amostragem. |
| #517 | Hair link: read the registries by field hash, and retry the stored link | `059aed3c` | Leitura por hash de campo, `FIELD_TYPE` e retentativa; `tint_row` com queda para a chave antiga. |
| #519 | 3D map: material table with shared pools and texture arrays | `e578e1f6` | O renderizador novo entra, mas ainda não é ligado ao caminho antigo neste commit. |
| #520 | 3D map: the material-table shader without the sticker part | `ccb4b1a4` | Remoção do trecho de adesivos do shader da tabela. |
| #493 | Link subtitles to their speaker and audio stream | `ea766ef1` | Comando `dialogue-links` completo (562 linhas). |
| #521 | Names: flipbook and region tint shader constants | `90e34f9a` | 7 nomes. |
| #524 | Water: export the small water bodies inside placed interiors | `aecd8349` | `interior: Option<PoolInterior>` em `PoolSurface`. |
| #525 | Hair export: keep settled hair just outside the body and garments | `5b3da3a3` | `surface` em `Colliders` + uso no export. |
| #526 | Hair settle: keep strands stored inside a collider where they are stored | `25f6dc79` | `push_out_to(p, depth)`. |
| #522 | Dialogue links: name speakers whose hash a known name gives | `9559a01d` | Nomeação por dicionário. |
| #527 | 3D map: the shared mesh shader without the sticker part too | `0377edc6` | Variante sem adesivos do shader de malha, atrás de `hdr`. |
| #528 | Cutscenes: audio banks and master timelines (#516) | `1417d210` | Banco de áudio por cena + estruturas da linha do tempo mestra. |
| #529 | Liveries: a sticker laid on flat can be projected from a side | `cba86b04` | `Sticker::from: Side`. |
| #532 | Names: UV transform and blend mask constants the exporters read | `34bcb4b9` | 6 nomes e comentários de `material.rs`. |
| #530 | Tattoo bake: lay the upper body out in its second UV set | `a565fd9f` | Terceira→segunda UV; **muda o layout do bake**, sem bump de `BAKE_RULE` neste commit (ver Confiança). |
| #534 | Water: inland height texture's scale from the drawable's stored bounds | `71768e8c` | `InlandTile::bounds` e escala derivada. |
| #533 | Looks: a cap worn backwards picks the hair's capbackwardshairalt state | `df295f3b` | Estado `capbackwardshairalt`/`visorhairalt`. |
| #531 | 3D map: short vertex program for the material table's shadows | `564e93c2` | Programa curto + opções de `watch`. |
| #535 | Cutscenes: master state machines (#516) | `9508a81f` | `CATALOG_VERSION` 5→6 e `--masters`. |
| #536 | Version 0.14.0 | `9c526e7a` | Só versão. |
| #537 | Liveries: a sticker projected from a side is dragged as seen from that side | `46e14f18` | `uv_at` pela lateral + log do arrasto. |
| #539 | Tattoo bake: limit a region-flagged record to the part's mesh | `f4105aae` | `decoration_corners` na assinatura do bake; `BAKE_RULE` 11. |
| #540 | Names: 12 shader constant names carried over from the previous engine | `e12df728` | 12 nomes. |
| #542 | Looks: tint a colour slot that binds the jewellery tint data map | `94012be9` | `colour_part` deixa de exigir `ped_shade`. |
| #543 | Cutscenes: subtitle lines with text, speaker and bank time (#516) | `4bb79778` | `CATALOG_VERSION` 7; legendas com texto e falante. |
| #544 | Tattoo bake: the 0x40 vertex-colour region cap | `7dee3840` | Dois fluxos de cor de vértice; `BAKE_RULE` 12. |
| #545 | Inferred hair table: match assets by piece names, clean up pieces | `ac0d9608` | `CACHE_VERSION` 6 e casamento por nome de peça. |
| #546 | Cutscenes: subtitle times per composite section (#516) | `5bbf3ac7` | `CATALOG_VERSION` 8. |
| #547 | Dialogue links: place generic speech lines by the speaker's speech voice | `9387fbc9` | Uso de `Voices` (schema 18). |
| #548 | 3D map: material table draws decals and see-through surfaces | `03a2e239` | 7 pipelines e mais dois tipos de desenho. |
| #549 | Dialogue links: report generic-role lines as voice chosen at run time | `06866d04` | Caso de papel genérico. |
| #551 | Cutscenes: what the state machine's string tests check, and its start transitions | `0bb6fa5d` | `CATALOG_VERSION` 9. |
| #490 | Name 963 more enum values | `700627d9` | 963 nomes em `enum_values.txt`. |
| #515 | Vehicles: merge a model's records filed under its handlingId | `08137817` | `FORMAT` 19 e fusão por `handlingId`. |
| #518 | Liveries: tyre-sidewall stickers (zones 226/227) draw on the fitted tyre | `2b2277a5` | Roda com osso no grupo; adesivo nas quatro rodas. Mesclado às 12:16:23Z, **depois** do recorte de mesclados da coleta (12:14Z) — o merge foi lido pelo commit, não pela lista de pulls fechados. |

## Cruzamento com o catálogo

- **Nenhum hash do catálogo aparece na janela.** Os 36 hashes foram conferidos contra as colunas
  `introduzido`/`corrigido` das 306 linhas; a lista de PRs mesclados também não cita `D-xx`.
- **Hipótese de área, não provada:** `#518` mexe no slot do pneu, área de **D-15** ("slot de cor do
  pneu aplicado: pneu inteiro vermelho ou cinza claro", introduzido em `4c2fe2a`). O diff não trata de
  cor de pneu e sim de posição de adesivo de flanco — classes diferentes.
- **Hipótese de área, não provada:** `#532`/`#545` nomeiam constantes que o exportador lê, área de
  **D-11** (valores de parâmetro de material no offset errado, `6b68f75`). Nomear a constante não
  muda o offset de leitura em `material.rs`.
- **Mudança de layout sem invalidação declarada:** `a565fd9f` (#530) troca a UV do bake do tronco e
  **não** incrementa `BAKE_RULE` (que só sobe em `f4105aae` e `7dee3840`, dois commits depois). Entre
  os dois, um cache de bake feito com a UV antiga é aceito como válido pela regra nova. Isso é
  candidato a linha de catálogo, e não está provado como defeito em jogo — ver Confiança.
- Nenhum defeito novo do catálogo fica demonstrado por este diff.

## Confiança

- **Verificado por API:** os 36 commits (sha, data, autor), os 35 mesclados e os 7 PRs abertos
  (título e corpo), os diffs de todos os 36 commits — arquivo a arquivo, com patch lido. O head
  `2b2277a5` foi conferido no fim da coleta (`compare` de `08137817...HEAD` = 1 commit).
- **NÃO VERIFICADO:** nada foi compilado nem executado. `BAKE_RULE` 10→11→12 e `CATALOG_VERSION`
  5→9 afirmam invalidação de cache, mas nenhum teste do repositório foi rodado para provar que a
  recompilação acontece. O efeito visual do nibble alto da região (`941bdf97`) e do segundo UV
  (`a565fd9f`) não foi medido em nenhum ativo.
- **Divergência de contagem, registrada:** o `compare` inicial deu `ahead_by=33`; a lista paginada
  trouxe 36 commits porque o head avançou durante a coleta (`700627d9` 12:14Z, `08137817` 12:15Z,
  `2b2277a5` 12:16Z). Os 34 PRs mesclados do recorte não incluem o `#518`, mesclado às 12:16Z; o
  merge dele foi lido pelo commit `2b2277a5`. Total real da janela: 35 merges.
- **Teto de commits:** não foi atingido (36 < 40). O cursor `last_commit` vai para o head
  `2b2277a5`; `compare` a partir dele devolve `identical`. Nada fica para a próxima hora.
- **`last_run`** gravado em `2026-11-28T12:14:29Z`, o maior `merged_at` do recorte (#490), no relógio
  do GitHub.
