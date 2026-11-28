# holdmysocks/SixNG — atualização de 2026-11-28 01:00

## Resumo

Seis commits acima do cursor `1b419021`, todos squash 1:1 com os seis PRs mesclados da janela, e um PR
aberto. A hora fecha três defeitos de **aparência de personagem** (camadas de máscara de tatuagem,
índices de imagem no merge de personagem, adesivo plano em painel) e nomeia 30 hashes de constante de
shader; fecha também com o bump de versão para 0.13.0. O `#508` é o mais direto: o merge de personagem
deixava o `speculartex2` e as duas folhas de tile do detalhe de pele — e as máscaras de desgaste, primer
e poeira do veículo — nos índices de imagem da **primeira** parte, então a pele do corpo lia a textura
de outra parte (manchas planas inclinadas e costura no pescoço da Lucia e da Connie).

## Commits novos (6)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `946524cb16` | Brandon Villar | Names: 18 more shader constant names (SIX-68) (#505) | `crates/sixng-core/src/names.rs`: dezoito nomes entram em `DERIVED` a partir de busca por palavra ao lado de nomes já conhecidos — o conjunto `layer1*`/`layer2*` de cachoeira (tiling, offset, intensity, opacitythreshold, opacitycontrast), `dirttintchannel`, `furlengthjitter`, `normalmaptiling2`, `specularfresnelselector`, `waterfadestartrandom`, `stickermetalness`, `stickerroughness`, `stickerbackopacity`. Só nomes; nenhum leitor de offset muda |
| `6def09f1ba` | Brandon Villar | Tattoos: mask layers fade or cut the layer before them (#507) | `peddeco.rs`: entram `MASK = 0x4` e `MASK_INVERTED = 0x8000_0000` e a função `masked(&[Layer])`, que percorre o registro pareando cada camada com a **máscara que a segue** (e pulando a máscara, que nunca é desenhada); `BAKE_RULE` sobe de 8 para 9, forçando re-bake das texturas. `export.rs` passa a usar `masked()` no lugar de `flat_map(|d| &d.layers)`, ajusta o `debug` de `layers` para `(l, mask)` e reescreve a regra publicada: camada com flag `0x4` não é desenhada, seu alfa de decal no próprio UV (invertido com `0x80000000`) escala a cobertura da camada anterior por `1 - (1 - a) × opacity`. Uma máscara que abre o registro não tem camada anterior e é desenhada como camada. `Skipped::uncoloured_masks` deixa de contar máscara que é ela mesma máscara |
| `66f606ac54` | Brandon Villar | Characters: shift skin detail images when parts are merged (#508) | `crates/sixng-gui/src/glb.rs` `merge()`: a lista que desloca índices de imagem pela base da parte (`img0`) ganha `wear_mask`, `primer_mask` e `dust_mask` ao lado de `ramp_u`/`ramp`; e um bloco novo desloca `skin_detail.spec2`, `skin_detail.normal` e `skin_detail.control`. Eram exatamente os campos que ficavam apontando para o atlas da primeira parte — daí as manchas e a costura de pescoço em personagem montado |
| `2a605002e1` | Brandon Villar | Liveries: a sticker laid on flat runs across panels and onto other models (#509) | Recurso novo, não correção. `vehlivery.rs`: `Decal` ganha `flat: Option<[f32; 11]>` (A, ou, B, ov, N no espaço RAGE Z-up) e `also: Vec<(String, Vec<u8>)>` (zonas em outros modelos), com `flat_to_rage()` convertendo do espaço do viewer (`glb(x,y,z) = rage(x,z,-y)`) e `prepare()` lendo o campo. `livery.rs`: `Sticker` ganha `flat`, `also` e `projection` (números gravados para referência, recalculados na carga) e o struct `Also`. `livery_ui.rs`: `toggle_zone` num adesivo plano com zonas acrescenta o modelo a `also` em vez de mover o adesivo, e `--livery-flat` (`main.rs`) aciona o modo no passo scriptado. `shader.wgsl`: bit 16 em `flags`, três vec4 novos (`pa`, `pb`, `pn`) e os varying `rest`/`rest_n` (posição e normal no espaço armazenado, antes de juntas e instância) para projetar o adesivo na face que olha ao longo de N |
| `58665d043d` | Brandon Villar | Water: name 12 of the water tile shader's parameter hashes (#512) | `names.rs` ganha os doze nomes do `water_micro_geom` (busca meet-in-the-middle sobre palavras de água e shader; ~16 acertos aleatórios esperados em 45 hashes, então só ficaram nomes que encaixam no valor ou num irmão): `useflowmap`, `useheighttexture`, `usesurfacemap`, `watersurfacemaptexture`, `usesurfacedetailmap`, `surfacedetailfadenear/far`, `surfacedetailtintcoloura/b`, `surfacedetailtintmultiplier`, `surfacedetailcontrol`, `microgeomcontrol`. `water.rs`: `TILE_TEX_PARAM` deixa de ser o hash `"e0548ad3"` e passa a `"watersurfacemaptexture"` nas duas consultas, e o JSON deixa de chamar o campo de `tile_texture_e0548ad3` e passa a `surface_map_texture` — **renomeação de chave de saída**, não mudança de conteúdo |
| `b42d09a9da` | Brandon Villar | Version 0.13.0 (#513) | `Cargo.toml` (workspace) e `Cargo.lock`: `0.12.0` → `0.13.0` nos três crates (`sixng-cli`, `sixng-core`, `sixng-gui`) |

## Pull requests abertos (1)

Acima do cursor `last_pr_number` 506, `created_at` 21:25:22Z no relógio do GitHub; título e corpo lidos,
patch não lido.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| `#510` | Names: 8 more shader constant names (SIX-68) | Oito nomes a mais, cada um com o hash, o nome e "onde fica": `vertexdisplacementnoisetiling` (084b2443), `tintroughnessmod` (08e5959c), `verttintblendamount` (896d1e47), `grassdecallayer`/`grassdecaluvoffset` (692e0a42/a56b5ccc), entre outros. Método declarado: começo de cada vizinho nomeado dentro de quatro slots + palavra de domínio + sufixo opcional, ~0,14 acerto aleatório esperado em 422 constantes | Não; mesma série de `#505`/`#512` |

## Pull requests mesclados (6)

Todos os seis com `merged_at` na janela casam 1:1 com um commit da faixa; o SHA de merge é o próprio
commit (squash de um pai).

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| `#505` | Names: 18 more shader constant names (SIX-68) | `946524cb` (21:14:49Z) | As 18 entradas de `DERIVED`, só nomes |
| `#507` | Tattoos: mask layers fade or cut the layer before them | `6def09f1` (21:19:29Z) | `masked()`, as duas flags, o bump de `BAKE_RULE` para 9 e a regra de tinta reescrita |
| `#508` | Characters: shift skin detail images when parts are merged | `66f606ac` (21:34:24Z) | Os três campos de `skin_detail` e as três máscaras de veículo somados em `img0` |
| `#509` | Liveries: a sticker laid on flat runs across panels and onto other models | `2a605002` (21:35:32Z) | O campo `flat`/`also` no arquivo de livery, a conversão de espaço, o modo `--livery-flat`, o caminho novo no shader e o bit 16 de `flags` |
| `#512` | Water: name 12 of the water tile shader's parameter hashes | `58665d04` (21:42:10Z) | Os 12 nomes, `TILE_TEX_PARAM` por nome e a chave `surface_map_texture` no `inland.json` |
| `#513` | Version 0.13.0 | `b42d09a9` (22:03:13Z) | O bump dos três crates |

## Cruzamento com o catálogo

- **Nenhum dos seis hashes aparece em `introduzido` ou `corrigido`** no `catalogo/defeitos.csv`; nenhum
  dos 24 D-xx de `SixNG` é reaplicado.
- **Áreas vizinhas, defeitos diferentes:**
  - **D-11** (valores de parâmetro de material lidos no offset errado) é sobre **offset**; `58665d04`
    dá **nome** a 12 hashes e troca a chave de um campo no JSON. O valor lido não muda.
  - **D-12** (carroceria do veículo aplicada a interior, carcaças e peças) é sobre qual material recebe
    o quê; `66f606ac` é sobre **índice de imagem** depois de juntar partes. Nenhum dos dois toca o
    outro.
  - **D-19** (`extract --force` não reescrevia texturas compartilhadas) não é tocado.
- **`#508` é um defeito real que não estava no catálogo** — o par de campos `speculartex2` e as duas
  folhas de tile do `skin_detail` (mais `wear_mask`/`primer_mask`/`dust_mask`) ficavam fora da lista de
  deslocamento em `glb::merge`, e o sintoma (manchas planas inclinadas, costura no pescoço) é nomeado no
  próprio commit. Candidato a linha nova em `SixNG`, com hash `66f606ac`.
- **`#509` acrescenta um recurso que o jogo não tem.** O próprio doc do `livery.rs` diz "The game has no
  such layer": o adesivo plano é do viewer, projetado pela normal, e pode atravessar painéis e modelos.
  Isso não é defeito, mas uma livery salva com `flat` não é reproduzível pelo jogo — vale registrar para
  quem for comparar export do SixNG com o resultado in-game.
- **Mudança de formato de saída em `58665d04`.** A chave `tile_texture_e0548ad3` do `inland.json` vira
  `surface_map_texture`. Qualquer consumidor do JSON que leia a chave antiga quebra silenciosamente; o
  commit não versiona o schema nem deixa alias.

## Confiança

- **Verificado no diff:** os seis commits foram lidos no patch por arquivo. `6def09f1` foi lido em
  `peddeco.rs` (flags, `masked()`, bump de `BAKE_RULE`) e em `export.rs` (a troca de `flat_map` e a
  regra publicada); a parte final de `peddeco.rs` (o bake em si) foi truncada no patch e **não** foi
  lida linha a linha. `2a605002` foi lido em `vehlivery.rs`, `livery.rs`, `livery_ui.rs` e nos primeiros
  45 linhas de `shader.wgsl`; a parte do shader que aplica `pa`/`pb`/`pn` por face foi truncada.
  `66f606ac` e `946524cb` foram lidos por inteiro.
- **Verificado por API:** os seis merges (`merged_at`, `merge_commit_sha` = commit da faixa) e o `#510`
  aberto (título e corpo).
- **Não verificado:** nada foi executado. Não há teste no diff de `66f606ac` (9 linhas, nenhum teste) —
  a correção é plausível pelo código, mas o sintoma visual não foi reproduzido. Não medi o efeito do
  `BAKE_RULE` 9 (quantas texturas são refeitas). A semântica de `flag 0x4` vem da mensagem e do
  comentário do código, não de um shader do jogo desmontado nesta hora.
- **Contagem:** 6 commits = 6 merges, 1:1. Nenhuma divergência.
