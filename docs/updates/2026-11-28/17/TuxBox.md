# brcly/TuxBox — atualização de 2026-11-28 17:00

## Resumo

Hora de memória de descritor e de upload de textura posicionada. Cinco merges, quatro deles com
conteúdo real: `#373` mantém o heap de render de um alvo colocado em endereço convidado enquanto o
range está commitado (a issue #323 relata leitura do VA de um heap liberado 11 minutos depois e device
loss em 15–18 min); `#391`/`#393` passam a seguir os registros de descritor que o título copia com
stores comuns para dentro do heap indexado, e a guardar as views substituídas que esses registros ainda
nomeiam. No caminho de upload, `a40e5a8c` troca a consulta de endereço XG por tabelas por forma, e
`661ef9cd`/`5de0e62b` reciclam buffers de upload e de bytes de mip alterado, com teto de 64 buffers e
64 MiB.

## Commits novos (10 + 1 de merge posterior ao snapshot)

Janela: `compare(2af42c94...5de0e62b)` = 10 commits, `status: ahead`, `behind_by: 0`. Todos
documentados. O `#404` foi mesclado às 17:12:17Z, **depois** do snapshot do `compare`; o merge dele
(`b8355fa2`) é hoje o head e consta da tabela de mesclados — o conteúdo veio do diff do PR, não do
`compare`.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `ab0751c8` | Jerkin42 | d3d12: keep a render heap while its guest range is committed | `RenderHeaps` guardava só um `Weak` por range convidado, então liberar a última imagem liberava o heap nativo na hora. O diff passa a manter o heap enquanto uma imagem o usa **ou** qualquer página do range convidado continua commitada (`heap.rs`), e libera quando o range é descommitado. `placement_guest.cpp` ganha `run_render_heap` (slot 133, `kept=65536/65536`) e `check_d3d12_regressions.py` a lane `render-heap`. +166/−8, 3 arquivos. |
| `a40e5a8c` | Nick Stallman | d3d12: detile placed mips through cached address tables | Cria `runtime/xg-layout/src/addresses.rs` (154 linhas): em swizzle 2D o endereço é o offset do tile mais os bits internos, combinados por OR/XOR, então os endereços de um mip viram uma tabela por coluna e uma por linha (`MipAddresses`). `placement/addresses.rs` (54 linhas) compartilha a tabela por forma de textura; `upload.rs` detila pela tabela em vez de perguntar o calculador XG a cada run de 16 bytes (0,5–0,8 ms/MiB em `0x105`/`0x10a`). Testes: +204 no layout, +165 no device. +607/−10, 7 arquivos. |
| `661ef9cd` | Nick Stallman | d3d12: reuse placed-texture upload buffers once their copy completes | Cada lote de upload criava buffer commitado, allocator e command list no caminho de submit e os liberava quando a fence passava. Agora o lote concluído vai para um pool; o próximo pega o menor buffer que caiba, reseta allocator e list e grava nele (buffers arredondados em potências de dois até 32 MiB). Teste `completed_uploads_lend_their_buffer_to_the_next_change`. +164/−51. |
| `99d8569d` | MathiewMay | d3d12: follow descriptor records a title copies into an indexed heap | O mapa de índice de recurso (o que programa direct-indexed lê) só preenchia um slot quando a API escrevia aquele índice; um registro copiado com stores comuns do título ficava invisível. O diff trata `InlineDescriptor`/`INLINE_BYTES` no `index_map`, cria `Arena::null_view` (SRV nula válida em vez de descritor não inicializado) e faz `recording.rs`/`submit.rs` registrarem as arenas cujo index map o lote lê, com refresh na submissão. Self-test novo `bindless-raw-copy`. +318/−49, 9 arquivos. |
| `0ae06f52` | MathiewMay | d3d12: keep replaced views for the records a title copied | Empilhado no anterior: no console a cópia de descritor guarda os bytes, então o título reusa o slot do heap de CPU assim que copiou. O diff adiciona `descriptor_paging/history.rs` (108 linhas, views substituídas mantidas para os registros copiados antes) e chama `arena.preserve(dst.ptr, count)` em `descriptor_copy.rs`. Lane nova `bindless-raw-copy-reused` no verify. +145/−5, 6 arquivos. |
| `4cb3347d` | brcly | Merge pull request #373 from Jerkin42/fix/render-heap-lifetime | Merge; conteúdo em `ab0751c8`. |
| `72edad75` | brcly | Merge pull request #391 from MathiewMay/d3d12-index-map-raw-copies | Merge; conteúdo em `99d8569d`. |
| `b90cc37e` | brcly | Merge pull request #393 from MathiewMay/d3d12-descriptor-history | Merge; conteúdo em `0ae06f52`. |
| `19dc351b` | brcly | Merge pull request #385 from NickStallman/d3d12/fast-detile | Merge; conteúdo em `a40e5a8c` e `661ef9cd`. |
| `5de0e62b` | brcly | d3d12: reuse the buffers that hold a changed mip's bytes | Os bytes de cada mip alterado iam para um buffer recém-alocado; plano de vídeo muda a cada frame e o intro do ProjectScarlett alocava e paginava ~12 MiB numa submissão em oito. O diff cria `ByteSpares` (teto `SPARE_BYTES = 64 MiB`, `SPARE_BUFFERS = 64`) e empresta os buffers ao próximo `Change`, com `texture.rs` passando `spares`. Teste `a_changes_bytes_are_lent_to_the_next_change`. +82/−7. |
| `b8355fa2` | brcly | Merge pull request #404 from KainanP (teste) | Merge fora do `compare`; conteúdo lido no PR (1 arquivo, +14/−16). |

## Pull requests abertos (2 acima do cursor; 1 segue aberto)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #405 | d3d12 checks: keep the owned and splash windows above others while sampling | `guest-present-4k-owned` e `guest-present-4k-splash` falham no Windows (RTX 4060 Laptop e UHD) porque amostram janelas que não são topmost e outro processo tem o foreground. Diff: `frame_present.cpp` ganha `coverage(HWND)` — nomeia o que a tela mostra nos pontos de amostragem — (31 linhas) e `splash.rs` passa a tratar `hook.is_invalid()` (11 linhas). | Não; é falha de teste, não do port. |
| #404 | d3d12 tests: order the alias copies by submission, not a UAV barrier | Aberto às 16:24 e **mesclado às 17:12**. `aliases_import_their_private_backing` falha no RTX 4060 Laptop: lê write de GPU por um segundo alias com só uma UAV barrier no meio, e UAV barrier ordena trabalho de acesso desordenado, não cópias. Diff reordena por submissão (+14/−16). | Não; infra de teste (ver mesclados). |

## Pull requests mesclados (5)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #373 | d3d12: keep a render heap while its guest range is committed | `4cb3347d` (16:55:02Z) | `RenderHeap` deixa de morrer com a última imagem: fica enquanto o range convidado tem página commitada. O PR cita a issue #323 — shader lendo o VA de um heap liberado 11 minutos depois, com device loss em 15–18 min de jogo. Corresponde a `ab0751c8`, incluindo a lane `render-heap` em `check_d3d12_regressions.py`. |
| #391 | d3d12: follow descriptor records a title copies into an indexed heap | `72edad75` (16:55:02Z) | Index map passa a seguir registros copiados por store comum; `null_view` centralizada; submit refresca as arenas indexadas que o lote lê. Corresponde a `99d8569d` (9 arquivos). |
| #393 | d3d12: keep replaced views for the records a title copied | `b90cc37e` (16:55:02Z) | Histórico das views substituídas (`history.rs`) e `preserve()` na cópia de descritores, para o registro copiado continuar válido depois que o título reescreve o slot de CPU. Empilhado no #391; corresponde a `0ae06f52`. |
| #385 | d3d12: detile placed mips through address tables and reuse upload buffers | `19dc351b` (16:55:02Z) | Duas metades: tabelas de endereço por forma (`MipAddresses`) e pool de buffers/allocator/command list de upload. Correspondem a `a40e5a8c` e `661ef9cd`. Não mediu ganho em jogo: os números do PR são de bancada. |
| #404 | d3d12 tests: order the alias copies by submission, not a UAV barrier | `b8355fa2` (17:12:17Z) | Um arquivo (`tests/device/heap_backing.rs`): a ordem passa a vir da submissão em vez de uma UAV barrier. O merge chegou depois do snapshot do `compare`; o conteúdo foi lido no diff do PR, e a alteração não toca runtime. |

Contagem: 5 merges contra 4 commits de conteúdo na janela (o `#404` é só teste, e entrou depois do
snapshot). Nenhuma linha faltando.

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum por hash: os 10 hashes da janela e os 5 de merge (`4cb3347d`, `72edad75`,
  `b90cc37e`, `19dc351b`, `b8355fa2`) não têm interseção com os 270 tokens de `catalogo/defeitos.csv`.
- **Toca área de D-xx?** Sim, em três frentes, sem fechar linha:
  - `#391`/`#393` (`99d8569d`, `0ae06f52`) atuam exatamente onde **D-17** ("liberação imediata do
    backing de descritor causa falha de MMU na GPU") e **D-22** ("`Failed to remap` (−3) em roots
    raster com várias tabelas de descritor") descrevem sintoma: registro de descritor que continua
    nomeando view liberada. O diff ataca a **cópia feita pelo título** (store comum no heap indexado),
    que é um mecanismo diferente do descrito nas duas linhas; não há hash do catálogo no PR e não
    verifiquei se D-17/D-22 desaparecem.
  - `ab0751c8` (#373) é o inverso de **D-18** ("aliasing de textura posicionada derrubando a fila"):
    ali o problema era aliasing no mesmo VA; aqui é liberar cedo o heap de um alvo colocado. A linha
    correspondente ao sintoma do #323 (leitura de VA de heap liberado, device loss em ~15 min) **não
    existe** no catálogo — candidata a linha nova.
  - `5de0e62b` e `661ef9cd` são custo/alocação no caminho de upload; não há linha de orçamento de
    upload no catálogo.
- **Abre defeito novo?** Não verificado. `0ae06f52` mantém views substituídas vivas — o teto dessa
  retenção não aparece no hunk lido, e sem teto seria crescimento monotônico (arquivo
  `descriptor_paging/history.rs`, linha não verificada).

## Confiança

**Verificado por diff.** Os 10 commits da janela foram lidos pelo patch de
`repos/brcly/TuxBox/commits/<sha>`, arquivo por arquivo. Os 4 merges com conteúdo foram conferidos
contra o commit de conteúdo correspondente (mesmos arquivos e mesmas contagens). `#405` e `#404` foram
lidos por `pulls/<n>/files`; o `#404` não passou pelo `compare` porque foi mesclado depois do snapshot.

**Não verificado.** Nada foi compilado nem executado: sem `cargo test`, sem GPU, sem rodar
ProjectScarlett. As alegações de desempenho do `#385` (0,5–0,8 ms/MiB de detile; ~12 MiB por submissão
no intro) vêm do corpo do PR, não de medição reproduzida. A relação entre o #373 e a issue #323 é
afirmação do PR. O `#404` foi mesclado às 17:12:17Z no relógio do GitHub, e é possível que outro merge
tenha entrado entre a leitura do head (`5de0e62b`) e o fechamento desta execução — o `last_commit`
gravado inclui `b8355fa2`.

**Estado.** `last_commit` `b8355fa2`; `last_run` `2026-11-28T17:12:17Z`; `last_pr_number` 405. A fila
de commits do repositório zera nesta hora (não há `commit_cursor` pendente).
