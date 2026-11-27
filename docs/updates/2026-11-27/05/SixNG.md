# holdmysocks/SixNG — atualização de 2026-11-27 05:00

## Resumo

Dois commits novos no `master` acima do cursor `aa6345c8`, ambos merges de PR: `fba2b4e8` (#445) torna
determinística a escolha de cor de lâmpada quando um osso está em mais de um lamp set — o `HashMap`
decidia por ordem de iteração, e duas execuções davam cores diferentes; `e380c30d` (#446) leva os mapas de
spec de pele para canais glTF (`metallicRoughnessTexture` = `speculartex`, `KHR_materials_specular`) e para
o sombreamento de pele do viewer. Merges na janela: 2 (#444 cai exatamente no cursor e já foi documentado).
Abertos acima de 450: 10 (#451..​#460), todos do próprio mantenedor.

## Commits novos (2)

`commits?sha=master&per_page=100`: `aa6345c8` está na posição 2; o topo é `e380c30d`.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `e380c30d` | Brandon Villar | Skin spec maps as glTF channels and in the viewer's skin shading (#392, part 2) (#446) | +172/-6 em 10 arquivos: `crates/sixng-core/src/skinspec.rs` +94/-1 (o mapa de spec da pele deixa de ser só decodificado e passa a ser publicado por canal), `crates/sixng-core/src/gltf.rs` +6 e `export.rs` +8 (material glTF recebe `metallicRoughnessTexture` = `speculartex` com `metallicFactor` 0 e `roughnessFactor` 1, mais `KHR_materials_specular`), `crates/sixng-gui/src/glb.rs` +24/-1, `render3d.rs` +7, `render3d_metal4.rs` +2, `shader.wgsl` +13/-1 (GGX com alpha = rugosidade², F0 = 0,04 × (1 + `fresnel`)), `world3d.rs` +2, `cells.rs` +1, `README.md` +15/-3. |
| `fba2b4e8` | Brandon Villar | Vehicle lamp lights: same lamp colours on every run (#445) | +9/-3 em `crates/sixng-core/src/vehlights.rs`: `Index::for_vehicle` percorria `p.lamp_sets.values()` de um `HashMap`, então quando um osso aparece em mais de um lamp set do pacote, `entry(..).or_insert_with(..)` deixava vencer o set que a iteração trouxe primeiro — e a ordem mudava entre processos (exemplo do corpo: osso `d418625e` saía `ffb626` numa execução e `ebf88c` em outra). Os lamp sets passam a ser visitados ordenados pelo hash do nome; **vence o de menor hash**, para `lamp_lights` e para `archetypes`. |

## Pull requests abertos (10)

`gh pr list --state open` devolveu **14** abertos; os 10 abaixo têm número acima do cursor 450.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 451 | Outfit export: the genitals stay under clothes | Leva ao export o que o #433 fez só no viewer: em `sixng outfit cs_jason --look 0` o insert de genitais atravessava o short tático. Causa declarada: a malha de genitais masculinos não carrega regiões de vértice (todas 0), então a máscara de esconder do short na camada de pele do corpo inferior nunca a alcançava; e uma peça solta (a toalha) deixa a pele sob ela na máscara. +222/-8, 8 arquivos. | Não. |
| 452 | Liveries panel: undo and redo (SIX-249) | Undo/Redo no painel de Liveries, com Ctrl+Z, Ctrl+Y e Ctrl+Shift+Z enquanto o painel está aberto e nenhum texto está sendo digitado; um passo é o que mudou entre dois momentos de repouso (solta do ponteiro ou 0,35 s sem mudança). +217/-1, 3 arquivos. | Não. |
| 453 | Facial control names and garment pose shape keys (#392) | Módulo `facecontrols` novo: os clipes faciais movem 91 controles, cada um uma trilha float de tipo `joaat("facialtranslation")` cujo id é o joaat do nome; 79 dos 91 nomes recuperados por busca de joaat sobre combinações de palavras estilo FACS. Acrescenta shape keys de pose de peça. +269/-15, 7 arquivos. | Não. |
| 454 | Enum value names: 303 more from a prefix brute force | Segue o #426: +305 valores de enum (303 nomes em 39 enums) achados por força bruta de palavras dentro do prefixo compartilhado de cada enum parcialmente nomeado (`INPUT_`, `EVENT_`, `VEH_`, `PCF_`, `TYPE_`, `EASE_`, `RC_`, `HS_`, `HBIL_`, `CT_`); a tabela vai de 3.253 para 3.558 valores. +311/-2, 2 arquivos. | Não. |
| 455 | Skin F0: the engine's albedo window, decoded | Parte de #392, empilhado no #446. Os dois programas de pixel de pele (studio e deferred) escalam F0 por uma janela no brilho do albedo: `saturate((max(r,g,b) - lo) / (hi - lo))` sobre o albedo linear, com `lo`/`hi` em `+0x6c0`/`+0x6c4` do buffer por frame `6e8725e4`, preenchidos por `fn ff0120` a partir de duas constantes em `.data` (sRGB 5 e 20 de 255 → 0,0015176 e 0,0069954 linear). **Base `people-skin-shading`.** +46/-6, 3 arquivos. | Não; é a mesma família de conversão de espaço de cor do D-24, em outro caminho. |
| 456 | Characters: a legacy outfit gets no second head or body | O outfit legado de Lucia ("outfit 148") é o modelo de corpo inteiro de baixa detalhe do jogo; o resolvedor do outfit somava a própria cabeça dela e a pele nua do corpo, e o corpo real aparecia através da malha legada. Passa a não incluir segunda cabeça nem corpo. +10/-3, 1 arquivo. | Não. |
| 457 | Hair: hold the settled guides on the game's leash | Segue o #444. A simulação de fios de perto não usa puxão fixo para o guia guardado: escala a compliance do puxão por uma coleira `c' = c · m · (R / |x − B|)^e`, com R no `graphics header +0x3c` (byte 4 / 600 m, ou `4·a·c` sem ele) e `e = 3E/255` (`+0x70` byte 2). +64/-15, 1 arquivo. | Não. |
| 458 | Ped decorations: bake head makeup toward the skin tone | Parte de #385, após #442: make-up de cabeça passa a ser assado. Uma camada de make-up tem tinta cuja linha `p3 (dL, da, db, w)` com `w ≠ 0` passa pela função do jogo (HSV→RGB do jogo, cor-alvo = bytes R,G,B do tom de pele / 255 levados a CIELAB) antes de empacotar. +291/-56, 12 arquivos. | Não. |
| 459 | Game liveries: one option per brand slot, as the game fits it | O slot de uma marca pode listar várias peças com probabilidade e o jogo encaixa **uma**; o export guardava todas e as marcas desenhavam as alternativas umas sobre as outras (604 de 1.898 marcas têm esses slots, com 2 a 40 variantes; a marca de polícia Rhode herda um slot de seis conjuntos de decalque a 16,7 % cada). `vehstickers` (formato 4) passa a dar slot, índice de opção e a escolha. +82/-42, 2 arquivos. | Não; parente de **D-14** ("Cores de peca do veiculo tiradas da primeira configuracao"), mas o defeito aqui é desenhar todas as opções, não pegar a primeira. |
| 460 | Ped decorations: the game's Lab white point for makeup | Parte de #385, empilhado no #458: o branco de referência da conversão sRGB→Lab passa a ser o do próprio exe — cópia da entrada D65 da tabela de iluminantes `(0,95047, 1,0, 1,08883)` — em vez das somas de linha da matriz sRGB que o #458 usou. **Base `claude/tattoo-makeup-385`.** +4/-3, 1 arquivo. | Não. |

## Pull requests mesclados (2)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #445 | Vehicle lamp lights: same lamp colours on every run | `fba2b4e8` (1 pai, +9/-3, 1 arquivo) | O que promete e o que entrou coincidem: `vehlights.rs` só, ordenação por hash do nome do set e a regra "menor hash vence" declarada no corpo e aplicada a `lamp_lights` e `archetypes`. Nenhuma mudança de formato de saída além da cor estável. |
| #446 | Skin spec maps as glTF channels and in the viewer's skin shading (#392, part 2) | `e380c30d` (1 pai, +172/-6, 10 arquivos) | O corpo pede que o #435 seja mesclado antes (já estava: `e985401a`, abaixo do cursor); o diff traz `skinspec.rs` +94/-1, a ponte glTF em `gltf.rs`/`export.rs`, o viewer em `glb.rs`/`render3d.rs`/`shader.wgsl` e o `README.md`. `metallicRoughnessTexture` = `speculartex`, metal 0, rugosidade 1, `KHR_materials_specular` com F0 = 0,04 × (1 + `fresnel`). A decodificação de `speculartex` (G = rugosidade, R = translucidez, B = oclusão) vem do programa `PS_StudioPedTextured_Eight` segundo o corpo. |

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum. Os dois hashes não são `corrigido` de nenhuma das 24 entradas do `SixNG`
  (`b7fbe3a`, `839e852`, `b7fbe3a`, `2ef8b8c`, `08e7457`, `2f67255` e as entradas sem `corrigido`). O
  parente mais próximo é **D-24** (cor da configuração tratada como linear quando é valor de display): o
  #446 e o #455 aberto mexem em espaço de cor de pele e de make-up, mas por leitura de canais de textura e
  janela de F0, não pela cor de configuração do veículo. **D-14** (cores de peça tiradas da primeira
  configuração) fica perto do #459 aberto, sem casar.
- **Abre defeito novo?** Nenhum demonstrado por diff. Dois pontos ficam registrados:
  - `e380c30d` publica rugosidade só no canal G de `metallicRoughnessTexture` e deixa B (oclusão) e R
    (translucidez) fora do glTF, indo para o G-buffer do jogo. Um consumidor glTF que leia oclusão de
    `metallicRoughnessTexture` B — ou que ignore `KHR_materials_specular` — vê um material diferente do
    viewer; o corpo declara a escolha, não um teste contra outro importador.
  - `fba2b4e8` fixa a escolha por **menor hash de nome de set**, uma regra arbitrária e determinística. Ela
    é estável entre execuções, mas muda a cor de qualquer osso que já estivesse vencendo por ordem de
    `HashMap` — ou seja, corrige a não-determinação trocando o resultado médio de ~5 lâmpadas citadas no
    corpo.

## Confiança

Verificado: `commits?sha=master&per_page=100` com `aa6345c8` na posição 2 e `e380c30d` no topo; os dois
commits lidos em `commits/<sha>` (um pai, `+172/-6` em 10 arquivos e `+9/-3` em 1); `search/issues` com 3
PRs mesclados desde 03:44:53Z — #444 cai no cursor (merge `aa6345c8`) e sobram #445 (04:36:51Z) e #446
(04:37:14Z); `pulls/<n>` para os 10 abertos acima de 450 (corpo, `+/-`, arquivos, base declarada).

Não verificado: as medições e decodificações citadas nos corpos (programas `PS_StudioPedTextured_Eight`,
`fn ff0120`, offsets `+0x6c0`/`+0x6c4` do buffer `6e8725e4`, tabela de iluminantes do exe); se os PRs
empilhados (#455 sobre #446, #460 sobre #458) foram rebaseados depois dos merges desta hora; a execução de
qualquer teste ou do viewer. Nenhum teste foi executado aqui; o repositório monitorado é somente leitura.
