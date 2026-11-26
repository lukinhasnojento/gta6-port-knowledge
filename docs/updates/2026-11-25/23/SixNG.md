# holdmysocks/SixNG — atualização de 2026-11-25 23:00

## Resumo

Seis commits novos, todos squash de PRs (`#326`–`#331`), num total de +926/-79. O trabalho da hora é
quase todo de aparência de personagem e cabelo: UV de raiz de fio amarrada ao conjunto de UV correto da
cabeça, roupas escondidas pelas máscaras do próprio item, strand tables lidas dos arquivos do jogo
(no lugar da variável de pesquisa), luz de rebote do chão no visor e roupa que não fica dentro de pele
inchada. Um commit mexe na tabela de presets do BC1 BCPack — a mesma região do defeito **D-09** do
catálogo, que continua aberto. Dois PRs novos.

## Commits novos (6)

Ordem cronológica.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `6c90bfd6` | Brandon Villar | Hair export: root UVs in the head UV set they belong to (#326) | +197/-20. `hairexport.rs` ganha `HeadUvs` (`name`, `pos`, `sets`) e `head_from_glb`, que acha a maior mesh com nome de cabeça (não os olhos) no GLB do modelo; `root_uv_set` compara as UV de raiz dos fios contra **todos** os conjuntos da cabeça nos pontos de guia. O `main.rs` passa a gravar `head_mesh` e `head_uv_set` nos extras do GLB, e `scripts/blender_hair_curves.py` usa esse conjunto como mapa de superfície. **Corrige uma afirmação errada do próprio README**: antes dizia que `TEXCOORD_0` era o layout do groom; agora é coordenada em um dos conjuntos da cabeça (no Jason, escalpo no conjunto 3, sobrancelhas/cílios/barba no 2, mediana de erro 0,000–0,023 contra 0,06 ou mais do segundo melhor) |
| `8c54c5b6` | Brandon Villar | Outfits: hide clothes worn under other clothes by the items' hide masks (#327) | +66/-15 em `model_view.rs`. Sai o par `under_meshes`/`outer_meshes` alimentado por `f.layer` (1 e 2, com `_ => {}` engolindo o resto); entra `layered: Vec<(Range<usize>, u8)>` com `levels` por garçom vindo de `garment_levels(parts.iter().map(|e| e.off), fx)`. Peça sem nível deixa de ser classificada em silêncio |
| `f2cce109` | Brandon Villar | BC1 BCPack: 7 preset window slots from a second mip-consistency round (#328) | +8/-3 em `bc1pack_tables.rs`: `OVERRIDES` passa de 39 para 44 entradas — slots novos 39, 48, 71 e 73, e os valores dos slots 4 e 69 mudam (slot 4 B0 14→13; slot 69 G1 26→25). O comentário do arquivo continua dizendo **"Fitted, not extracted: see bc1pack.rs"** |
| `084d845b` | Brandon Villar | Viewer: ground bounce in the model view's ambient light (#329) | +41/-7. `daylight.rs` acrescenta `Light.bounce` lido de `ambient_bounce_intensity` com `clamp(0.0, 4.0)` (padrão 0,5); `render3d.rs` acrescenta `Look.bounce`, a constante `BOUNCE = 0.5` e o campo no `Default`; `shader.wgsl` separa `sky_light(n)` e reconstrói `ambient(n)` como o cubo ambiente do mapa (`atmo.wgsl cs_ambient`): céu, chão de `GROUND_ALBEDO = 0.15`, e o rebote do sol por `sun_col.w × albedo`. O efeito declarado: modelo iluminado por sol baixo recebe luz do lado sombreado, como no jogo |
| `14a44598` | Brandon Villar | Viewer body shape: clothes are not left inside a swelling skin (#331) | +19/-5 em `model_view.rs`: `Wrap::at` ganha o parâmetro `n: Option<[f32;3]>` e, quando o normal é unitário (tolerância 0,1 na norma), desloca o offset no eixo do normal até pelo menos o maior dos seis vértices de pele mais próximos. Antes, uma barriga em peso máximo crescia além da média inverso-quadrado e a roupa ficava dentro da pele |
| `addb0e79` | Brandon Villar | Hair: strand tables from the game files, still-camera accumulation, viewer sun shadows (#330) | +595/-28. **`hairtables.rs` novo (+294)**: lê os XML binários `RBF0` do arquivo de packs (raízes `rage__crsmHairStrandColorDatabase` …) e produz `colour_ramps`, `colour_sets`, `parameter_ramps`, `parameter_sets` e `lod_presets` no formato que `hairtone::StrandData::from_json` já aceitava — substitui o `SIXNG_HAIR_DATA=<json>` de pesquisa. **`accum.rs` novo (+155)**: acumulação de frames com câmera parada, `CAP = 32`, média corrida em 16 bits float para o dither de fios assentar. `render3d.rs` +106 e `model_view.rs` +38 acrescentam `accumulate`, `accum_key`, `accum_bump` e `DEFAULT_PARAMSETS` (`Long_HighGloss_000` para `hair`, `Eyebrows_MidGloss_000`, `Eyelashes_MidGloss_000`), com a ressalva de que a propriedade `2259149558` do catálogo de assets ainda não é lida por asset |

## Pull requests abertos (2)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 332 | Textures: resolve de-duplicated textures (painted body skins for 2,602 characters) | A maioria das texturas de pele pintada faltava e o corpo pegava a cor do rosto no pescoço. Texturas idênticas são guardadas uma vez e uma tabela em `textures.rpf` (entrada `bin` com cabeçalho `EDUP`) mapeia os outros hashes para a cópia guardada | Não consta no catálogo `SixNG` (D-01…D-24). É classe nova: textura existente no install e invisível por deduplicação |
| 333 | Map 3D: the placed lights shine at night | Luzes externas do mapa assado (`lights.bin`) acesas na hora, por máscara horária e flag noturna, com atenuação a ~500 m da câmera | Não consta. Vizinho de D-23 (UV das camadas de ground em 4 camadas), outro subsistema |

## Pull requests mesclados (3)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| 329 | Viewer: ground bounce in the model view's ambient light | `084d845b` | Squash de um commit; entrou o rebote por `ambient_bounce_intensity` com clamp 0–4, `sky_light` separado e o cubo ambiente equivalente ao do mapa. Bate com o que o título promete |
| 330 | Hair: strand tables from the game files, still-camera accumulation, viewer sun shadows | `addb0e79` | Squash de +595/-28 com três peças: parser `RBF0` (`hairtables.rs`), acumulação de frames (`accum.rs`) e sombras de sol no visor. O título cita as três; o diff entrega as três |
| 331 | Viewer body shape: clothes are not left inside a swelling skin | `14a44598` | Squash de +19/-5: deslocamento no eixo do normal até o maior dos seis vizinhos de pele. Bate com o título |

Observação de cursor: **`#326`, `#327` e `#328` também foram mesclados nesta janela** (seus commits
`6c90bfd6`, `8c54c5b6`, `f2cce109` estão na tabela acima), mas têm número ≤ 328 — o cursor
`last_pr_number` do baseline era 328, então o filtro `numero > last_pr_number` não os alcança. Eles
ficam documentados por hash de commit, não pela tabela de mesclados.

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum hash desta faixa casa com correção registrada no catálogo `SixNG` (D-01
  `b7fbe3a` … D-24 sem hash). Mas dois commits tocam regiões catalogadas:
  - `6c90bfd6` corrige uma premissa errada que o repositório publicava (UV de raiz "no layout do
    groom"). O catálogo registra defeitos dessa natureza em outros repos (D-26 de GTA6-Asset-Info, D-17
    de AvatarManz); em SixNG não havia linha. **Candidato a linha nova.**
  - `f2cce109` mexe de novo na janela de presets do BC1 — a região do **D-09** ("BC1: seis dos nove
    slots de janela substituídos eram piores que o default"), que está **aberto** no catálogo. O
    commit muda valores dos slots 4 e 69 e acrescenta quatro slots. O comentário do arquivo continua
    declarando `Fitted, not extracted`, ou seja: a tabela segue sendo ajuste empírico, sem extração
    do dado original. **Não fecha D-09** e não há medição no diff que prove melhora — só a frase
    "second mip-consistency round" no assunto.
- **Abre defeito novo?** Um candidato declarado pelo próprio autor em `addb0e79`: o parâmetro de fio
  por peça ainda não vem do asset (`DEFAULT_PARAMSETS` fixo por slot, "property 2259149558 is not read
  per asset yet"). Enquanto isso, toda cabeça usa `Long_HighGloss_000` como conjunto de parâmetros.
- Ficam abertos no catálogo, sem toque nesta hora: D-02 (TOC do RPF9 nunca lido — nomes e offsets por
  carving), D-04 (modelo antigo sob nome novo), D-06/D-07/D-08 (BC6H), D-16 (strings de text tables),
  D-22 (mips subcontados pelo descritor, ~44,9 mil texturas).

## Confiança

- **Verificado:** os 6 commits e os diffs (arquivos, contagens, trechos); as três fusões por
  `mergedAt` + `mergeCommit.oid`.
- **Verificado:** os 2 PRs abertos, títulos e autoria; o cabeçalho `EDUP` citado é o que o corpo do PR
  descreve, não foi conferido em arquivo.
- **NAO VERIFICADO:** se o resultado visual descrito (luz de rebote, roupa fora da pele, cabelo mais
  suave) aparece em execução — nenhum binário foi rodado.
- **NAO VERIFICADO:** o diff de #332 e #333.
