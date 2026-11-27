# holdmysocks/SixNG — atualização de 2026-11-27 20:00

## Resumo

Quatro commits entre o cursor `2d9c230d` e o head `67bcc0ec`, todos squash de PR mesclado, dois deles
com merge durante esta coleta (#477 às 17:10:02, #479 às 17:14:27 no relógio do GitHub). A janela é
inteira de render e geometria: agrupar batches de célula de mapa, juntar desenhos por material, fazer o
clique do painel de liveries acertar o veículo como ele está desenhado, e dar aos dentes e à língua o
movimento da mandíbula. Um PR aberto (#476).

## Commits novos (4)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `bab14f08` | Brandon Villar | Map 3D cells: join neighbouring batches into one draw (#475) | `crates/sixng-gui/src/cells.rs` +48: novo `CellCpu::draw_order()` ordena os batches por (pass, chunk, lo, hi do grupo de distância, grupo) e **reescreve o index buffer** nessa ordem, recalculando `Batch.first`/`count`. `render3d.rs` +32/-7: o upload usa essa ordem; no laço de desenho, batches consecutivos do mesmo chunk que se tocam viram um `draw_indexed` só, e a run também engole um batch fora da faixa quando o vão custa no máximo metade dos índices da run (`(b.first - r.end)*2 <= (r.end - r.start) + b.count`), o que é seguro porque o vertex shader já descarta placement fora da própria faixa. O bind group do chunk é setado só quando o chunk muda. Batch com `chunk` fora de `chunk_bg` passa a ser ignorado em vez de indexar |
| `3f9c8513` | Brandon Villar | Liveries panel: clicks land on the vehicle as it is drawn (doors open) (#478) | `livery.rs` +98/-10: novo `posed_vertex()` aplica a soma ponderada das matrizes de skin (peso/255, normalizada pelo peso total, limiar 0,001; vértice sem peso ou mesh não skinned fica parado) e `Posed::of()` gera posições e caixas por mesh com zona de sticker. `pick()` passa a receber `Option<&Posed>` e testa o raio contra as posições desenhadas. `livery_ui.rs` +25/-23: `Env` ganha `skin: &[Mat4]` e o cache de pose vira `(caminho, triângulos, hash das matrizes, Posed)`, refeito só quando uma junta se move. `model_view.rs` +4/-2: passa `joint_mats()` quando o painel está com portas abertas, `&[]` caso contrário. Teste novo: porta girada 90° sobre a dobradiça não é mais atingida de frente e é atingida pela lateral |
| `2b57afd1` | Brandon Villar | Map 3D: lay out model meshes by kind and material, join decal and see-through draws (#477) | `render3d.rs` +156/-44: `GpuModel` ganha `order: Vec<usize>` e `ordered()`; `mesh_kind()` classifica material em 0 sólido, 1 cut-out, 2 decal, 3 see-through (`alpha == 2` ou classe 5/6/8 vira see-through, `alpha == 1` cut-out); `draw_order(mats, kind)` ordena por (kind, material, índice). O upload grava os vértices e índices já nessa ordem, mantendo `meshes` na ordem do arquivo. O passe principal e as sombras passam a caminhar `ordered()`, e as sombras desenham os sólidos de um modelo em um call. Novo `record_joined()` desenha as listas de decal e see-through ordenadas por (pipeline, modelo, mesh), ligando grupo e buffers só na mudança e juntando mesh que segue no mesmo modelo/material/pipeline; decal opaco passa a ser ordenado de longe para perto **por modelo** (meshes do mesmo modelo ficam juntos). Modelo cujas instâncias são todas fallback de cobertura sai das listas. `late_draws` novo conta decal e see-through no campo de draws |
| `67bcc0ec` | Brandon Villar | Teeth and tongue follow the head's jaw shape keys (#392) (#479) | `crates/sixng-core/src/jawkeys.rs` +296 (módulo novo): para cada chave de mandíbula, ajusta aos deslocamentos do queixo rígido um giro em torno de `FB_Jaw` por mínimos quadrados (equações normais 3x3, regra de Cramer, `Rodrigues` para aplicar) e, quando não há giro que sirva, um deslocamento. Constantes: `MIN_MOTION = 0.001` m, `MAX_RESIDUAL = 0.3`, `CHIN_DEPTH = 0.06` m. Chave aceita dá o mesmo movimento a todo vértice sob `FB_Jaw`, escalado pelo peso de skin. `outfit.rs` +3 e `lib.rs` +1 ligam o módulo: os dentes e a língua ganham shape key de mesmo nome e os vértices do interior da boca somam o movimento à chave da cabeça |

## Pull requests abertos (1)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #476 | fix(metal4): borrow model staging bytes | base `master`: passa slices emprestados a `new_buffer_with_bytes` do Metal-Rust nos buffers de staging de vértice e índice do modelo, consertando o build de debug com a feature `metal4-poc` contra a API fixada (+2/-2, 1 arquivo). Validação declarada: `cargo build --locked -p sixng-gui --features metal4-poc` em macOS arm64 | Não. Build de back-end experimental; sem entrada no catálogo |

#477 e #479 estavam abertos quando a coleta começou e foram mesclados durante ela; entram como commits
e como merges desta hora, e o cursor vai até o head `67bcc0ec`.

## Pull requests mesclados (4)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #475 | Map 3D: join map-cell batches into fewer draws | `bab14f08`, 2026-11-27T14:54:27Z, base `master` | O que o PR prometia, e um pouco mais: além da ordem de upload e da junção de draws, o diff também ignora batch cujo chunk não existe no bind group (guarda que o texto do PR não menciona). 3 arquivos, +86/-8, com teste de unidade da ordem, dos offsets e dos índices reordenados |
| #478 | Liveries panel: clicks land on the vehicle as it is drawn (doors open) | `3f9c8513`, 2026-11-27T16:59:46Z, base `master` | Picking contra a malha posada pelas juntas, com cache por hash das matrizes. O diff entrega também um detalhe que o PR não detalha: o limiar de peso total (0,001) e a normalização pela soma dos pesos, que é o que faz o resultado bater com o shader |
| #477 | Map 3D: group model meshes by material so they draw as one call | `2b57afd1`, 2026-11-27T17:10:02Z, base `master` | Layout por tipo e material, junção no passe principal e nas sombras, ordenação de decal opaco por modelo, descarte de modelos só-fallback e contagem de draws tardios. 3 arquivos, +166/-46 |
| #479 | Teeth and tongue follow the head's jaw shape keys (#392) | `67bcc0ec`, 2026-11-27T17:14:27Z, base `master` | Módulo `jawkeys` com o ajuste giro/deslocamento e os limiares de aceitação, ligado ao export de outfit. 4 arquivos, +301/-1. O PR foi aberto às 17:04:29 e mesclado dez minutos depois; `README.md` do mesmo commit cita #392 |

## Cruzamento com o catálogo

- Nenhum dos quatro commits corrige entrada do catálogo SixNG (D-01 a D-24 são todas de conteúdo de
  pacote, formato, textura ou extração; a janela é de render e UI).
- **D-18 (lista de modelos pulando e roubando o foco do teclado):** não tocado — é UI de lista, não
  picking de viewport.
- **Candidata a linha nova, `bab14f08` + `2b57afd1`:** a ordem dos índices no buffer compartilhado deixou
  de ser a do arquivo. Qualquer consumidor de `Batch.first`/`count` fora do laço de desenho (export,
  ferramenta, outro passe) passa a precisar do `order`; o diff mantém `meshes` na ordem do arquivo e cria
  `order` justamente para isso, mas **nada nos dois commits enumera os consumidores** — é risco de
  contrato implícito, não defeito observado.
- **Candidata a linha nova, `67bcc0ec`:** o ajuste da mandíbula é heurístico e silencioso — chave cujo
  ajuste deixa mais de 30% do movimento do queixo sem explicação, ou cujo queixo anda menos de 1 mm, é
  ignorada sem aviso. Um key de mandíbula que caia fora desse critério volta ao comportamento antigo
  (dentes parados) sem que nada registre isso.
- **`3f9c8513`:** o `skin` só é passado quando `livery.ed.open`; com o painel fechado o picking usa a
  pose de repouso. Não é defeito, é o contrato do commit, e está no código (`model_view.rs`).

## Confiança

Verificado por mim, pelo diff:

- os quatro hashes, autores, pais e contagens de linha vieram da API; `2b57afd1` tem pai `3f9c8513` e
  `67bcc0ec` tem pai `2b57afd1`, ou seja, a sequência da `master` é linear nesta janela e não há merge
  commit — os quatro são squash;
- li o patch inteiro de `bab14f08` (células) e de `3f9c8513` (livery/UI/view), o começo do de
  `2b57afd1` (`GpuModel.order`, `mesh_kind`, `draw_order`, `record_joined`) e o começo de `jawkeys.rs`
  (cabeçalho, constantes, `apply`, `rotate`, `solve3`, `fit_both`);
- a lista de mesclados foi recolhida por `pulls?state=closed&sort=updated` e reconferida ao fim da
  coleta: 4 merges, o último #479 às 17:14:27;
- o head conferido ao fim da coleta é `67bcc0ec`, igual ao último commit documentado.

NÃO VERIFICADO:

- nada foi compilado nem executado; os ganhos de draw call (345-676 batches por frame nos views de
  cidade, 30-70 calls por modelo) são medições do autor no corpo dos PRs;
- li só o primeiro trecho do diff de `2b57afd1` (~6 KB de 156 linhas adicionadas em `render3d.rs`): a
  parte de `record_joined` e a ordenação de decal foi lida parcialmente; o resto do arquivo não;
- de `jawkeys.rs` li ~4,5 KB de 296 linhas: não verifiquei o resto do ajuste (montagem das equações
  normais além do primeiro bloco, critério de escolha entre giro e deslocamento, testes);
- não confirmei se há teste que compare `posed_vertex` com o shader de skin do viewer; o commit alega
  "a mesma soma ponderada", e o diff mostra a fórmula, mas não há verificação cruzada no repositório que
  eu tenha visto.
