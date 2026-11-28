# fxceless/victory-fork — atualização de 2026-11-28 00:00

## Resumo

Um commit acima do cursor `cd935520`, empurrado direto na `main` (sem PR), e nenhum merge na janela. O
commit `2eade68c` é a correção do estouro de orçamento de VRAM por blocos de textura de 64 MiB: o portão
que admitia textura nova pesava o uso da placa **menos** a parte livre dos blocos, então um bloco novo era
criado sempre que a textura não cabia num pedaço livre — e o bloco só volta à placa quando esvazia por
inteiro. Passa a haver portão por bloco, espera da textura que pode esperar, e o orçamento dos arenas de
ray tracing sai do que ainda falta dos render targets.

## Commits novos (1)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `2eade68c` | fxceless | The card stays under its memory budget: no new texture block past the texture line | 8 arquivos, +698/−69. `pool.rs` (+382/−19) ganha o portão: bloco novo só se `used + 64 MiB + card_room_kept + targets_to_come <= room_line(budget)` (`block_room`/`block_fits`, com cache de um décimo de segundo em `may_grow`); sem esse espaço a textura procura pedaço num bloco existente e, se não houver, **espera** — o slot continua devido, mostra a view anterior e é reexaminado em 1, 2, 4, … frames até ~4 s (`draw.rs` +26; `waiting`/`waits` marcam a thread, e `resource.rs` deixa de marcar `refused` para quem só espera). Textura que não pode esperar (leitura por tabela no draw, bake) ainda ganha bloco, contada em `PAST`. `resource.rs` (+208/−35) muda o que a fila e a varredura pesam: `weighed()` usa `blocks_full(line, free) = line + 64 MiB − free` quando nenhum bloco pode ser criado, mantendo 2–5 % do orçamento e um bloco livres para o portão não travar; `by_address_room()` passa a contar `targets_to_come` (arenas de ray tracing não podem tomar o espaço que os render targets vão ocupar — na run 423, 44 arenas, 2,8 GB, 14 couberam e os targets cresceram +850 MB depois). `memory.rs` e `shadow.rs` passam a usar a mesma regra e a contar por onde a memória por endereço ficou (placa × PC). `host.rs` (+32) acrescenta duas linhas de diagnóstico (enchimento dos blocos por faixa, blocos feitos/não feitos/passados, texturas esperando). `play.rs` fixa `VICTORY_POOL_FULLEST=0` |

### O que o commit afirma (não medido nesta hora)

- Antes: run 423 ficou 0,8–1,0 GB acima do orçamento por sete minutos (11,3 GB em uso contra 10,3) com
  1,5–2,6 GB livres **dentro** dos blocos; frames de ~162 ms contra ~90 ms em sessões dentro do orçamento,
  com travadas de mais de um segundo.
- Depois: RTX 4070, sessão de 4,5 min, 8.078–8.707 MB contra orçamento de ~10.000 em todas as 49 leituras,
  163 blocos não criados; 12,6 fps com 5.000–7.000 draws, ~10 com 7.000–12.000; uma leitura acima de
  300 ms; nenhuma textura preta ou faltando.
- Verificação declarada: build release passa; testes das módulos alterados de `victory-gpu`, 100 passaram.

## Pull requests abertos (2)

Ambos acima do cursor 1493, `created_at` 2026-11-27, base `main`; título e corpo lidos, patch não lido.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| `#1496` | GPU: backport placed-page write-watch stamps from Vice-Tay | Páginas posicionadas na submissão guardam `Held::taken = 0`; o refresh seguinte as trata como nunca enviadas e pode sobrescrever com bytes de CPU o que o construtor de estruturas já escreveu na GPU. Lembra o pedido de write-watch anterior à cópia inicial; `VICTORY_PLACE_TAKEN=0` mantém o comportamento antigo | Área de **D-06** (textura enviada antes de o título preencher a memória), mecanismo diferente — D-06 é ordem de upload, este é carimbo de write-watch. Não está no catálogo com esse nome |
| `#1495` | Shaders: backport the scalar-code bounds fix from Vice-Tay | `ir.rs`: uma VOP3 pode expor código de destino escalar acima de 127 e `unspill` varre uma tabela `used` de 128 entradas — pânico no índice 215 antes de a tradução devolver resultado ou erro. Passa a cobrir os 256 códigos | Não é D-xx do `victory-fork`. Backport de Vice-Tay `00eaf47`, original de `avatarmanz/VIctory-7900XTX#325` |

Os dois são backports de Vice-Tay e se declaram independentes entre si. Nenhum patch de PR foi lido.

## Pull requests mesclados (0)

Nenhum `merged_at` maior que `last_run` (2026-11-27T18:10:00Z). O único commit da janela não veio de PR:
`repos/fxceless/victory-fork/commits/2eade68c/pulls` devolve lista vazia.

## Cruzamento com o catálogo

- **D-17 (dívida de verificação: "this merged tree was built; its test suites were not run") não fecha.**
  O commit declara build release e 100 testes dos módulos alterados, não a árvore inteira, e **não
  acrescenta teste nenhum** — os 8 arquivos do diff não incluem `tests/`. A mudança mexe no caminho de
  admissão de toda textura e no que a varredura pesa; sem teste, a alegação de que "nada é movido e nenhum
  bloco é devolvido que não era antes" fica sem verificação no repositório.
- **D-06 não é reaplicado nem corrigido.** O `#1496` aberto mexe em `memory.rs` na mesma vizinhança
  (`Held::taken`, refresh de página), mas o defeito do catálogo é textura escolhida por índice enviada
  antes de o título preencher a memória — outro caminho.
- **Candidato a defeito novo do catálogo:** estouro de orçamento de VRAM por blocos de 64 MiB que só
  retornam inteiros, com a fila pesando o uso menos a parte livre desses blocos (commit `2eade68c`
  corrige; números da run 423 na mensagem). Não existe linha correspondente nas 17 entradas de
  `victory-fork`.
- **Divergência de convenção de switch, no mesmo commit.** `pool.rs` liga `VICTORY_POOL_FULLEST` por
  padrão (`is_none_or(|v| v != "0")`) e `apps/victory/src/play.rs` o fixa em `0` nas `SETTINGS`; o
  comentário diz por quê (mudaria a frequência com que bloco vazio volta à placa, não contada ainda).
  Quem roda fora do launcher vê o caminho mais agressivo. Não é D-xx do `victory-fork`, mas é a mesma
  classe de convenção dupla que o acervo registra em D-16 de outro repositório.

## Confiança

- **Verificado no diff:** `pool.rs` (portão, `fullest`, `waiting`/`waits`, `Grow`, contadores) lido até o
  teto de 6.000 caracteres do patch; `host.rs`, `memory.rs`, `shadow.rs`, `draw.rs` e `play.rs` lidos por
  inteiro; `resource.rs` lido até o teto (`weighed`, `blocks_full`, `POOL_BLOCK`, `block_room`,
  `block_fits`, `by_address_room`) — o fim de `resource.rs` e de `pool.rs` não foi lido, então
  `targets_to_come`/`title_fills` são conhecidos só pelo trecho e pelos comentários.
- **Não verificado:** nenhum número de memória, fps ou travada foi medido aqui — são os da mensagem do
  commit. Nenhum teste foi executado; não há teste novo no diff. O comportamento de deadlock do portão
  (a alegação de que a varredura mantém folga) não foi exercitado.
- **Contagem:** 1 commit na faixa, 0 merges, 0 PRs abertos mesclados fora de janela. Coerente.
