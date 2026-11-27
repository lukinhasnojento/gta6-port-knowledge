# fxceless/victory-fork — atualização de 2026-11-27 18:00

## Resumo

Um único commit desde `9a9091d1`, e ele é o maior evento do dia: `5dbf5ce2` substitui a árvore
inteira do fork pela de `trustno1-coder/VIctory-NVIDIA` no commit `959dcaf`. São 284 arquivos,
+95.231/−913.607 linhas — a linha própria do fork deixa de existir na ponta e permanece só no
histórico, até `9a9091d1`. O autor declara que só mudou redação em 14 arquivos e que nenhum código foi
alterado. Os seis defeitos que o próprio fork havia corrigido (D-01 a D-06 do catálogo) passam a
depender de o upstream carregar equivalentes: conferi os símbolos-chave no novo topo e todos estão
presentes, mas equivalência semântica não foi medida.

## Commits novos (1)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `5dbf5ce226` | fxceless | Rebase this fork on VIctory-NVIDIA by trustno1-coder (959dcaf) | Pai `9a9091d1`. Substitui a árvore pela de `trustno1-coder/VIctory-NVIDIA@959dcaf`: `crates/victory-gpu/src/descriptor/host.rs` +3.910/−1.845, `crates/victory-formats/src/texture.rs` +346/−180, `crates/victory-formats/src/xma.rs` +622/−42, `README.md` +315/−937. Somem `crates/victory-gpu/src/backreads.rs` (−694) e `cardmeter.rs` (−528); entram `ambient.rs` (+861), `book.rs` (+752), `compose.rs` (+325), `cores.rs` (+174), `ab.rs` (+131), mais `apps/victory/src/play.rs` (+493), `replay.rs` (+507), `warm.rs` (+210) e `native_1440p.rs` (+214), e três launchers (`Get-NVIDIA-DLSS.cmd`, `PLAY_FRAMEGEN.cmd`, `Play-VIctory.cmd`). O corpo declara: diferença do upstream só de redação em 14 arquivos (o repositório não nomeia o título; `scripts/play-framegen.ps1` renomeado; variáveis `TITLE_*`), build release passa e `cargo test --workspace` dá 1.046 passando, 46 ignorados e 3 falhando — dois deles reproduzem em `959dcaf` sem modificação e o terceiro passou isolado. |

## Pull requests abertos (13)

Janela: números acima de 1468. Cinco corpos lidos (#1469, #1471, #1472, #1473, #1474); os demais só
pelo título e pela lista de arquivos, e estão marcados assim.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #1469 | perf(gpu): VRAM RT directory, 64 KiB page arenas, and Use::Reads/Builds split | Move o diretório de 16 MiB de endereço bruto do `host.shared_buffer` (PCIe) para uma cópia em VRAM, para o sobremapeamento falso-positivo de 8 MiB de célula solta do root constant e troca varredura volátil de página inteira + `pull_back` por arenas de página de 64 KiB em VRAM e separação `Use::Reads`/`Use::Builds`. +2.903/−939 em 14 arquivos. O autor liga a necessidade ao construtor de BVH por padrão em `9a9091d`. | Área de D-05 (regra de staleness por sobreposição total) e D-06 (textura enviada antes do preenchimento): mexe em `resource.rs`, `shadow.rs`, `upload.rs`, `descriptor.rs`. Sem hash do catálogo. |
| #1471 | GPU/texture: a subresource's staging slice stops at the buffer's end | `Staging::detile` monta a fatia de cada subrecurso como `pitch * rows * depth` a partir de `data + Offset`, mas `GetCopyableFootprints` não preenche a última linha do último subrecurso (`pTotalBytes = (rows−1)*pitch + última linha`), então a última fatia reivindicava bytes além do upload mapeado — `from_raw_parts_mut` fora do mapping é UB mesmo sem ler nem escrever fora; com `VICTORY_LEVELS_ON_DEMAND` o efeito aparece na tela. +48/−6 em `texture.rs`. | Corrige exatamente a área de D-06 (textura enviada antes de o título preencher): o PR cita runs 72/73, buffer de 1.920 bytes e fatia de 2.048. |
| #1472 | Formats: a depth pitch that overlaps its depth slices is refused | `Layout::walk` calcula o buffer linear de `row_pitch`/`depth_pitch` do título sem checar que uma fatia de profundidade termina antes da próxima; com `depth_pitch` menor que uma fatia de linhas, `tile`/`detile` leem ou escrevem endereços sobrepostos e ainda devolvem `Ok`, perdendo em silêncio as linhas da fatia anterior. O PR recusa esse layout. +43 em `victory-formats/src/texture.rs`. | Área de tiling/textura do catálogo; sem hash. |
| #1473 | Formats: reject multisampled linear (row-major) texture layouts | `Layout::new` aceitava `samples > 1` com `Swizzle::LINEAR`; a matemática linear é de uma amostra, então o tamanho reportado discordava dos próprios mips (4 amostras 256×256 reportava 262.144 bytes, contra 4× isso no tiled) e só a amostra 0 se movia nos caminhos de cópia. +31/−1 em `victory-formats/src/texture.rs` e `victory-hle/src/modules/xg.rs`. | Área de formato/tiling; sem hash. |
| #1474 | GPU: a recording let go without running lets go its resolved query slots | `query::resolve` (caminho `VICTORY_OCCLUSION`) aloca buffer de leitura durante a gravação e insere um slot por query; os slots só voltavam quando a lista executava. Uma lista fechada que o título reseta ou libera sem `ExecuteCommandLists` é legal em D3D12, e as escritas adiadas eram descartadas sem remover os slots — cada `ResolveQueryData` vazava um buffer. +59 em `host.rs`, `host/abandoned_recordings.rs` e `query.rs`. | Sem D-xx equivalente; área de recursos/query. |
| #1475 | HLE/tests: the read-in-place test's reader reads before the remapping loop | Só teste (título apenas): corrige a ordem de leitura no teste de leitura no lugar antes do laço de remapeamento. | Não. |
| #1476 | GPU: what shaders touch on the card's side in one switch, sizes by the card's budget (VICTORY_CARD_SIDE=1, off by default) | Diagnóstico opt-in do que os shaders tocam do lado da placa, com tamanhos pelo orçamento da placa. Corpo e arquivos não lidos nesta hora. | Não verificado. |
| #1478 | GPU/tests: the alias-source pin test runs in default builds | Faz o teste de pin de fonte de alias rodar em builds padrão (antes só em configuração de diagnóstico). | Não; teste. |
| #1479 | GPU/diagnostics: count the pieces that differ on the relay and card merges | Contadores de diagnóstico para as peças que divergem nas junções relay/placa. | Área de `shadow.rs`/merge; não verificado. |
| #1480 | GPU: the retirement path drops without a list and splits outside its lock | Aposentadoria de listas: descarta sem lista e divide fora do lock. Corpo e arquivos não lidos nesta hora. | Não verificado. |
| #1481 | gpu: VICTORY_LINE_WAITS, what the submitting thread waits on and who ends it (diagnostics, off by default) | Diagnóstico das esperas da thread de submissão. | Não. |
| #1482 | gpu: VICTORY_SAMPLE_LINE, a stack sampler for the submit thread and named thread groups (off by default) | Amostrador de pilha da thread de submissão. | Não. |
| #1483 | xmem: VICTORY_XMEM_WHOLE_MB, join adjacent mapping calls into one section (off by default) | Junta chamadas de mapeamento adjacentes numa seção, atrás de switch. | Área de XMem; não verificado. |

## Pull requests mesclados (0)

Nenhum merge na janela: `pulls?state=closed&sort=updated` não devolveu `merged_at` acima de
`2026-11-27T13:05:00Z` para este repositório. Os 13 PRs acima seguem abertos.

## Cruzamento com o catálogo

Os defeitos D-01 a D-06 do `victory-fork` foram todos corrigidos na linha que o rebase substituiu
(`e9765a7`, `f85101b`, `2119b0e`, `310c6eb`, `a803c87`). Como o commit troca a árvore inteira, a
pergunta é se o upstream carrega equivalentes. Conferência por símbolo no topo `5dbf5ce2`:

- **D-01/D-02** (forma wave64 "wide" derrubando o device; chave do cache sem os switches de wave):
  `VICTORY_WAVE_WIDE` aparece em `crates/victory-shader/src/emit/mod.rs` e em
  `crates/victory-gpu/src/shader_cache.rs` — os dois pontos que `e9765a7` alterou. **Presente por
  símbolo; semântica não comparada.**
- **D-03** (janelas de indirect draw servindo lixo): `WINDOW_MOST` aparece três vezes em
  `crates/victory-gpu/src/draw.rs` no topo novo, o mesmo arquivo e o mesmo nome do conserto de
  `f85101b`. **Presente por símbolo; não comparado.**
- **D-04** (reativação da mudança de lane-mask, `2119b0e`): `unspill` (7 ocorrências) e `parks` (23)
  em `crates/victory-shader/src/ir.rs`, e `VICTORY_HOST_MAX_WAVE` em `apps/victory/src/translate.rs`.
  **Presente por símbolo; não comparado.**
- **D-05** (staleness por sobreposição total): `descriptor/host.rs` do topo novo tem 23 ocorrências de
  `Spans`, 182 de `span` e 16 de `overlap` — o arquivo e o vocabulário de `310c6eb`. **Presente por
  símbolo; não comparado.**
- **D-06** (textura escolhida por índice enviada antes do preenchimento): `RECHECK_FRAMES` em
  `draw.rs` e `staged_memory` em `texture.rs` no topo novo, contra os mesmos nomes em `a803c87`.
  **Presente por símbolo; não comparado.**
- **Risco de regressão nos PRs abertos**: #1471, #1472, #1473 e #1474 foram escritos contra a linha
  antiga (citam `texture.rs:1352-1360`, `query.rs`, `host.rs`) e agora têm base `main` já rebaseada;
  mergeabilidade não foi verificada. #1469 já se refere a `9a9091d`, que agora é ancestral.
- **Defeito novo**: nenhum identificado. O volume do diff (913.607 linhas removidas) impede afirmar
  por diff que nada do fork se perdeu; a conferência acima é de símbolo, não de comportamento.

## Confiança

Verificado: o commit foi lido pela API com `stats` e a lista completa de arquivos (`parents` =
`9a9091d1`); os 13 PRs foram coletados por `pulls?state=open`; cinco corpos foram lidos (#1469 a
#1474) e os outros oito só têm título, o que está dito na tabela. Os símbolos de D-01 a D-06 foram
buscados por download do arquivo no `ref` `5dbf5ce2`, não por leitura do histórico.

Não verificado: (i) nenhum `cargo test`/build foi executado; a contagem 1.046/46/3 é declaração do
autor; (ii) não comparei semanticamente o código do upstream `959dcaf` com os consertos `e9765a7` a
`a803c87` — a presença dos símbolos não prova que a lógica seja a mesma; (iii) o commit `959dcaf` está
em outro repositório (`trustno1-coder/VIctory-NVIDIA`) e não foi inspecionado nesta hora; (iv) os
oito PRs sem corpo lido (#1475 a #1483) podem propor mais do que o título diz.
