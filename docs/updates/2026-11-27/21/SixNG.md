# holdmysocks/SixNG — atualização de 2026-11-27 21:00

## Resumo

Três commits squash entre o cursor `67bcc0ec` e o head `1c8672d8`, todos com `committer` GitHub. O
`fix(metal4)` do #476 é de duas linhas e é o único com sinal de correção: passa a **emprestar** os
bytes de staging em vez de move-los para `new_buffer_with_bytes`, o que mantém os vetores vivos depois
da chamada. O #472 acrescenta uma ferramenta de bake offline do cache de modelo (CLI + botão na GUI,
processo filho, verificação CRC em streaming), e o #481 traz para o viewer o que o export já fazia: as
chaves de mandíbula da cabeça passam a mover dentes e língua. Dois PRs abertos acima de #479, nenhum
merge novo além dos três.

## Commits novos (3)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `5871a8cb` | maple-quartz18 | fix(metal4): borrow model staging bytes (#476) | `crates/sixng-gui/src/render3d_metal4.rs` +2/-2: `new_buffer_with_bytes(vertex_bytes, …)` e `(index_bytes, …)` viram `new_buffer_with_bytes(&vertex_bytes, …)` e `(&index_bytes, …)` nos dois buffers de staging (`ResourceOptions::SHARED`). Nenhuma mudança de conteúdo de byte, de layout ou de sincronização: os `Vec` deixam de ser consumidos na chamada e continuam disponíveis para o resto do escopo. É correção de posse/compilação, não de comportamento de render |
| `eec52ca0` | maple-quartz18 | feat(metal4): add offline model cache baking tool (#472) | Quatro arquivos, +524. `main.rs` +54 registra `--bake-model-cache` e `--bake-log`; `crates/sixng-gui/src/modelbakejob.rs` +128 (novo) roda o subcomando como **processo filho** (`SIXNG_WORK`, stdout/stderr em `null`, progresso lido de `<work>/model_cache_bake.log` a partir do offset inicial), com `cancel()` que mata o filho e `Drop` que mata bake órfão; `pages.rs` +41 põe o botão e a linha de resultado na página de Settings; `render3d_metal4.rs` +301 gera os caches `SXMTLIO2` com o **mesmo** `build_vertex_index_bytes()` e os mesmos escritores do upload em runtime, verifica CRC em blocos (memória limitada), pula diretórios de cache e conta found/baked/skipped/failed/verified. Tudo com `#[cfg(macos + metal4-poc)]` |
| `1c8672d8` | Brandon Villar | Viewer: teeth and tongue open with the jaw shape keys (#392) (#481) | `crates/sixng-core/src/jawkeys.rs` ±: extrai de `follow_jaw` duas funções públicas — `fit_keys(pivot, head, head_jaw, morphs, lo, hi)` devolve `(nº de vértices de queixo, Vec<JawKey>)` e `add_keys(morphs, keys, pivot, positions, jaw)` aplica o deslocamento ponderado; o export passa a chamá-las, com o mesmo resultado de antes. O teste de queixo (`< 50` vértices) e o limiar de resíduo continuam iguais. `model_view.rs` +58 (novo `teeth_follow_jaw`): acha a junta `FB_Jaw`, calcula o peso de cada vértice sob ela pela cadeia de pais, acha a malha com morfos `face_*`, ajusta os morfos no queixo como giro em torno do pivô ou como translação (o de menor resíduo, abaixo de `MAX_RESIDUAL`), converte Z-up/-Y-forward ↔ Y-up/Z-forward nas duas direções e acrescenta às malhas de dentes/língua/interior da boca chaves **com o mesmo nome**, para o painel Expression abrir a boca inteira. `README.md` +1/-1 atualiza a linha de shape keys faciais |

## Pull requests abertos (2)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #483 | Skin translucency: the game's subsurface profiles, shaded and exported | base `master`, head `people-skin-translucency`: perfis de subsuperfície do jogo, sombreados e exportados | Não no catálogo |
| #482 | Hair export: strand widths from the game's parameter ramps | base `master`, head `people/mandy-export`: largura de fios a partir das ramps de parâmetro | Não no catálogo |

## Pull requests mesclados (3)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #476 | fix(metal4): borrow model staging bytes | `5871a8cb`, 2026-11-27T17:32:57Z, base `master` | O diff de 2 linhas; nada além |
| #472 | feat(metal4): add offline model cache baking tool | `eec52ca0`, 17:33:10Z, base `master` | Os 4 arquivos (+524); o corpo do PR descreve dois commits (ferramenta e botão) e o squash traz os dois |
| #481 | Viewer: teeth and tongue open with the jaw shape keys (#392) | `1c8672d8`, 17:50:50Z, base `master` | 3 arquivos, +117/-31: a extração `fit_keys`/`add_keys` em `jawkeys.rs`, o `teeth_follow_jaw` no viewer e a linha do README. O título indica que o squash veio do PR #392 |

Os três `merge_commit_sha` são os próprios commits da faixa, com um pai cada: squash, sem commit de
merge. Contagem de merges (3) = contagem de commits (3).

## Cruzamento com o catálogo

- **Nenhum `D-xx` do SixNG é corrigido.** Os 24 defeitos da seção do repositório tratam de RAGE/RPF
  (D-01, D-02), normal map (D-03), corrida de carregamento de modelo (D-04), prop-batch (D-05), BC6H e
  BC1 (D-06 a D-09), eixos e materiais (D-10 a D-16), meshlets (D-17), lista de modelos (D-18),
  `extract --force` (D-19), expressão de sticker (D-20), R16F (D-21), mips (D-22), UV de ground (D-23)
  e cor de configuração (D-24).
- **O `fix` do #476 não é um defeito do catálogo.** É mudança de posse em Rust (`&Vec<u8>` em vez de
  `Vec<u8>`), sem efeito sobre pixels, cache ou formato. Não abre nem fecha `D-xx`.
- **A ferramenta de bake toca o cache de modelo, área de D-04** (modelo antigo exibido sob o nome novo,
  corrida de carregamento assíncrono) — mas D-04 é corrida de carregamento no viewer, e o bake escreve
  caches validados por CRC na leitura. Mesma área de dados, mecanismo diferente; **não corrige D-04**.
- **Dentes e língua não têm entrada no catálogo.** O achado do #481 é novo para o acervo: o export já
  movia dentes e língua com as chaves de mandíbula, e o viewer não; agora os dois usam o mesmo
  `fit_keys`. A documentação do defeito (se ele for um) teria de nascer de uma análise, não desta
  rotina.

## Confiança

Verificado por mim, pelo diff:

- os três hashes, autores, arquivos e contagens; `parents` = 1 e `committer` = GitHub nos três;
- li o diff completo do #476 (é o arquivo inteiro, 2 linhas) e conferi que a assinatura de
  `new_buffer_with_bytes` recebe `&[u8]` — a mudança é de empréstimo, não de conteúdo;
- li `modelbakejob.rs` inteiro (start/cancel/poll/Drop, leitura do log a partir do offset, `try_wait`);
- li o diff de `jawkeys.rs` (a extração de `fit_keys`/`add_keys` e a troca em `follow_jaw`) e o
  `teeth_follow_jaw` do viewer, incluindo a conversão de eixos e o peso por cadeia de pais.

NÃO VERIFICADO:

- **nada foi executado.** Não rodei `cargo test`, nem o bake, nem o viewer. As "6 unit tests" do bake e
  a contagem de 14 testes citadas no corpo do #472 são afirmação do autor;
- **o bake é macOS + `metal4-poc`.** `modelbakejob.rs` e o subcomando estão sob `cfg` de macOS; neste
  ambiente (Windows) nenhuma linha é compilada, então não há como exercitar;
- **a equivalência byte a byte entre o cache assado e o de runtime.** O PR afirma que usa as mesmas
  funções; li a chamada no diff de `render3d_metal4.rs` só por nome, não comparei os dois caminhos de
  escrita linha a linha;
- **o efeito visual do #481.** O ajuste tem limiar (`MAX_RESIDUAL`) e o teste de queixo é
  "pelo menos 50 vértices"; não há imagem, medição ou teste automatizado do resultado no diff. O
  número "Lucia: 16,6° em `face_039`" é do README;
- os patches de #482 e #483: li número, título, base e head; não abri corpo nem diff.
