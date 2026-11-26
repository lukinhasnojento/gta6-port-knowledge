# fxceless/victory-fork — atualização de 2026-11-25 23:00

## Resumo

Dezenove commits novos — a maior faixa da hora em código. Oito são pacotes de log de execução
(cru `diagnostics/run-281…292`); onze mudam código de verdade, todos no `victory-gpu`. Três achados
merecem destaque: a correção de blending em alvo inteiro (`025b042a`) replica um defeito que o
catálogo do repo irmão **DexNom** já registra; o instrumento que investigava a perda de placa foi
retirado e sua hipótese **descartada** (`0fc6b066`); e o autor marca o switch de *gather by copy* como
**não provado exato** depois de revisão independente (`36bdd515`). Há nove PRs abertos, dois deles já
importados na `main` (`#1237`, `#1241`).

## Commits novos (19)

Ordem cronológica.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `bbf8de9c` | fxceless | Instrument: how high the title's view records reach, and those at slots the host heap has no title slot for | `descriptor.rs` +63: novo `PAST_THE_HOST[4]` (maior slot em página alterada; registros não-nulos em slots sem slot de título no host; maior slot; contagem de slots do host) e `past_the_host()`, chamado de dentro de `sync_heaps`. `host.rs` +10 imprime os quatro números no relatório de remoção do device. **Só log**; o que é espelhado não muda |
| `025b042a` | fxceless | Integer render targets do not blend on the host | `pipeline/host.rs` +109: `integer_target(format)` reconhece `_UINT`/`_SINT`; novo `GraphicsState::integer_targets_do_not_blend()`. Em `pipeline.rs`, ao ler o estado inteiro, o blending é desligado **só** para esses alvos e só na cópia usada no host (estado sem alvo inteiro fica bit a bit igual, preservando o nome na library). Loga uma vez. A origem é o run 283, onde a debug layer acusou `R8_UINT` com blending no slot 2 |
| `f38b4274` | fxceless | Instrument: what the view records past the host's title slots name, and how many the title writes during play | `descriptor.rs` +79: os registros contados por `past_the_host` passam a ser classificados em quatro tipos (view vivo, view constante, view de memória colocada, nada conhecido), com os dezesseis primeiros nomeados e uma linha a cada 5 s. Medição registrada: 30.882 registros não-nulos nos slots 737.855–1.245.183 durante o boot, em grupos ao fim de cada 4.096 slots, e **nenhum** em 160 s de jogo |
| `fabf3c4c` | fxceless | diagnostics: the logs of runs 281 to 284, scrubbed | Cinco arquivos: `diagnostics/README.md` +4, `run-281-2026-11-25.txt.gz` e `run-282-2026-11-26.txt.gz` (binários, 0 linhas contadas pela API), `run-283-2026-11-26-debug-layer.txt` (26.193 linhas) e `run-284-2026-11-26.txt` (24.199) |
| `0fc6b066` | fxceless | The pass over the title's view heap can leave alone the part the host has no title slots for (VICTORY_HEAP_SYNC_CUT=1\|alt, off by default); the instrument that found it removed | `descriptor.rs` +45/-131: com o switch, `sync_heaps` só toma e lê as páginas até `start + mirrored_count * step`; o miolo do instrumento sai. `host.rs` perde as dez linhas que imprimiam `PAST_THE_HOST` na remoção. O comentário registra a conclusão medida: ~100.000 registros reescritos por frame em slots além dos do host (um terço views vivas, um quinto views constantes), cerca de metade das páginas alteradas por frame, e que **esses registros não são pista para a perda de placa** — são tabelas que a camada serve de slots próprios |
| `c9092ec4` | fxceless | diagnostics: run 285 (what the view records past the host's title slots name), scrubbed | `diagnostics/README.md` +1 e `run-285-2026-11-26.txt` (22.921 linhas) |
| `249f2c20` | fxceless | Frame shots: how much of each picture's texture holds anything (the share of texels with a bit set, the highest packed-light value) | `shot.rs` +41: `holds(format, w, h, pitch, rows)` calcula quantos texels têm qualquer bit posto e, para `R32_UINT` (luz empacotada lida como R9G9B9E5), o maior valor de canal. Duas colunas novas em `snaps.txt`, só em build de diagnóstico. O motivo declarado: a imagem é escalada pelo maior valor, então um texel branco deixa o resto preto (caso shot-2705) |
| `03f0002a` | fxceless | diagnostics: run 286 (the heap pass's cut measured: no gain), scrubbed | `diagnostics/README.md` +1 e `run-286-2026-11-26.txt` (26.389 linhas). O título registra o resultado negativo do corte de `0fc6b066` |
| `36bdd515` | fxceless | Gathering copy by copy: still ends whole for programs that can store through a picked view or by address; the switch marked not proven exact | `draw.rs` +19/-1: `gather_ends` passa a encerrar o gather inteiro também quando algum estágio tem `t.stores` e `root_srvs` contém `SLOT_STRIDES` ou `SLOT_MEMORY`. `indirect.rs` +10 documenta **NOT PROVEN EXACT (revisão independente, 2026-11-26)**: três formas de escrita não declaradas — buffer escolhido por índice calculado no view heap, memória do título por endereço via cópia numerada por outra lista, e entrada de array de descritores além da lista de recursos usados; as duas primeiras agora encerram inteiro, a terceira **não está coberta**. Exige `VICTORY_INDIRECT_WINDOW_CHECK=1` e uma execução antes de usar o switch para jogar |
| `e3cdeff5` | fxceless | diagnostics: runs 287 (frame shot) and 288 (closed at boot), scrubbed | `diagnostics/README.md` +2, `run-287-2026-11-26-frame-shot.txt` (23.322) e `run-288-2026-11-26-closed-at-boot.txt` (1.538) |
| `ac255498` | fxceless | Instruments: why the merge under flights gives up, a submission's preparing by part, the heap pass by part | `host.rs` +47 cria `PREP_PARTS[6]`; `descriptor.rs` passa a cronometrar `SYNC_PARTS` (pedido, leitura, varredura, espelhamento); `shadow.rs` +16 cria `MERGE_GAVE_UP[4]` com os quatro motivos de desistência do merge (memória não observada, sem resposta do rastreio, mais runs que `VICTORY_MERGE_RUNS`, cópia em RAM sem snapshot) e `merge_gave_up()`. Números de origem: settle de 6,5–30 ms/frame a 64–94 ms cada, 19–23 ms de "preparing" sem parte contada, heap pass de 8–13 ms. **Nenhuma mudança de comportamento** |
| `32be36d5` | fxceless | A thread remembers what its asks for a buffer's host copy found (fork PR #1237; aqui VICTORY_LOOKUP_CACHE=1\|alt, off by default) | `shadow.rs` +116: `tables_changed()` e contador de geração; a thread guarda o que cada consulta achou enquanto as tabelas não mudam; `LOOKUPS` conta acertos por thread e por tabela. `host.rs` +8 imprime as duas médias por frame na linha de frame. O PR de origem está aberto (#1237) e **já foi importado aqui**, com o switch desligado por padrão (lá ligado) |
| `7496a4ce` | fxceless | Frame shots: VICTORY_SHOT_LISTS, so a shot can hold the frame's later lists | `shot.rs` +13/-1: `closing()` compara com `frame_lists()` em vez de `FRAME_LISTS`; `VICTORY_SHOT_LISTS` permite um limite que nenhum frame alcança (o shot passa a guardar três presents). Motivo declarado: shot-1785 (run 287) tinha 34 listas submetidas antes de as listas de iluminação do frame começarem e não guardava nada delas |
| `19a45d19` | fxceless | A view of a texture the card wrote is copied from it even if the list already uploaded that view from title memory (fork PR #1241) | `host.rs` +8: `Deferred::texture_aliases: HashMap<usize, usize>` e `alias_source_changed(view, source)`. `texture.rs` +5/-2: `alias()` deixa de usar `first_use` como prova e passa a exigir cópia da **mesma** fonte GPU. `resource.rs` +92: teste que faz upload da memória do título e depois lê o atlas escrito pela GPU. O autor do PR leu zero antes e o valor de luz empacotada depois numa RTX 5080 |
| `64f3a82e` | fxceless | diagnostics: run 289 (a play session under the debug layer: a draw with a pipeline declared for another target format), scrubbed | `diagnostics/README.md` +1 e `run-289-2026-11-26-debug-layer-play.txt` (85.387 linhas) |
| `a35c5573` | fxceless | A draw into a render target of another format than its pipeline declares gets a host pipeline for the bound format (VICTORY_TARGET_VARIANTS=0: as before) | `draw.rs` +31: `State::target_formats` guarda os formatos das views ligadas por `set_targets` (nove words); `pipeline.rs` ganha `host_pipeline_for(this, bound)`; `pipeline/host.rs` +214 traz `TargetFormats`, `formats_differ` e `with_formats`; `make.rs` +89 faz `Pending::formats` e uma cadeia de variantes `OnceLock<Arc<Pending>>` por conjunto de formatos. Só o caso medido (alvo com formato diferente do declarado, `R10G10B10A2_UNORM` contra `R11G11B10_FLOAT` na run 289); depth e slots não declarados ficam como estavam. Quatro testes de unidade |
| `e5b80dd7` | fxceless | diagnostics: run 290 (closed in the menu), scrubbed | `diagnostics/README.md` +1 e `run-290-2026-11-26-closed-in-menu.txt` (5.863) |
| `c74e1738` | fxceless | diagnostics: run 291 (a new area: frame times, pipelines made, shadows still black), scrubbed | `diagnostics/README.md` +1 e `run-291-2026-11-26-new-area.txt` (30.073) |
| `fc63ce16` | fxceless | diagnostics: run 292 (lookup cache in turn, a three-frame shot in the new area), scrubbed | `diagnostics/README.md` +1 e `run-292-2026-11-26-frame-shot-new-area.txt` (33.150) |

## Pull requests abertos (9)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 1236 | Allow timed input holds in release benchmarks | Porta a sintaxe `at:key:hold` de `Marvisak/VIctory#2205` para pular a cena de abertura do benchmark da primeira sala | Não |
| 1237 | A thread remembers what its asks for a buffer's host copy found: 245.000 lookups a frame off the tables' lock, 117 → 104 ms | Cache por thread das consultas de cópia host, com invalidação; mede 117,2 → 104,2 ms/frame numa RTX 3090 Ti | Não. **Já está na `main` deste repositório** como `32be36d5`, com o switch desligado e um follow-up aberto (#1245) |
| 1240 | GPU: opt-in mip cap (VICTORY_TEXTURE_MAX_DIM), low-RAM pool trimmer (VICTORY_RAM_TRIM_MB), batched descriptor mirror & bind cache | Quatro otimizações opt-in para 8 GB VRAM + 16 GB RAM; base declarada `006c353` | Área de D-05 (staleness de textura por sobreposição) e D-09 (cópia in-place e sondagens), sem declarar correção |
| 1241 | Copy GPU texture aliases after an upload from title memory | Rastrear a última fonte GPU copiada por textura, para que um upload da memória do título não anule a cópia | Não. **Também já está na `main`**: `19a45d19` |
| 1242 | victory-gpu: Clippy passes again in both CI builds | O Clippy fixo 1.98.1 parou a `main` (`03f0002`) em 10 erros no build simples e 3 mais no de diagnóstico, então **todo PR falha o check antes das próprias mudanças** | Não é um D-xx nominal, mas é exatamente o **D-17** do catálogo (`victory-fork`: "dívida de verificação — this merged tree was built; its test suites were not run"). O texto descreve árvore mesclada que não passa no próprio lint |
| 1245 | Invalidate cached buffer lookups at registration and idle sweep | Segue o cache importado em `32be36d5`: o registro podia publicar a geração nova enquanto uma thread de gravação ainda lia a tabela antiga, e uma consulta podia reerguer uma janela solta depois do sweep | Expõe uma corrida **no commit `32be36d5` desta mesma faixa**: o corpo daquele commit afirma que "cada lugar que muda as tabelas incrementa o contador", e o #1245 diz que não bastava |
| 1246 | GPU: textures picked by index: looked-at and fresh slots in turns; slots without memory looked at again after up to 20 s | Rostos com textura escolhida por índice (garçonete, close-ups da Lucia, figurantes) saem pretos, vermelhos ou ruidosos de forma não determinística; propõe alternar slots e reexaminar slots sem memória | **Cobre o D-06 do catálogo** ("Textura escolhida por índice era enviada antes de o título preencher a memória — rosto preto"), registrado como corrigido em `a803c87`. É recorrência ou resto do mesmo defeito — só o diff decide, **NAO VERIFICADO** |
| 1247 | Keep AMD's shader compiler (RX 9060 XT) from failing on 13 translated programs | Mesmo texto e mesmo problema do PR #62 de `DexNom/VIctory`: 13 programas traduzidos derrubam o compilador da AMD numa RX 9060 XT | Não há linha no catálogo `victory-fork`; é a mesma classe do D-06 de Belobug-AMD, outro repositório |
| 1248 | Wave-wide form for three wave64 programs seen to run in it; dynamic slots found by settling flights; later texture rechecks; a shader watch and slowed clocks | Três programas wave64 de compute que o host de 32 lanes recusa passam a usar a forma wave-wide por padrão; mais slots dinâmicos por settle e rechecagem de textura | Área de **D-08** (índice de view além do fim do heap) e **D-01** (forma wave64 "wide" derrubando o device) do catálogo; este último está registrado como corrigido em `e9765a7` e o PR mexe justamente na escolha da forma wide |

## Pull requests mesclados (0)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| — | — | — | nenhum PR com número > 1234 aparece como mesclado. Os conteúdos de #1237 e #1241 entraram por commit direto na `main` (`32be36d5`, `19a45d19`), não por merge |

## Cruzamento com o catálogo

- **Corrige D-xx conhecido?** `025b042a` — alvo de renderização inteiro com blending ligado derrubando
  a criação do pipeline. Esse defeito **não está no catálogo `victory-fork`**, mas está no catálogo do
  repo irmão **DexNom como D-01**, com correção registrada em `ddb7332`. É o mesmo defeito, resolvido
  aqui por implementação independente (aqui o blending é desligado por alvo, preservando o nome do
  pipeline na library). Vale acrescentar a linha ao catálogo `victory-fork`.
- **Elimina hipótese de D-16?** Sim, parcialmente. O D-16 do catálogo diz "perdas de device sem causa
  identificada (runs 204-279 e mundo aberto)". `0fc6b066` mede os registros reescritos além dos slots
  de título do host (~100.000/frame, metade das páginas alteradas) e conclui que **não são pista**
  para a perda: são tabelas servidas de slots próprios. D-16 continua aberto — um suspeito a menos,
  nenhum culpado.
- **Registra defeito novo?** Sim, dois:
  - `36bdd515`: o switch `gather_by_copy` fica marcado **NOT PROVEN EXACT** por revisão independente.
    A terceira forma de escrita (entrada de array de descritores além da lista de recursos usados) não
    é coberta e pode ser omitida do gather. Antes do commit o switch se apresentava como exato.
  - `32be36d5` + PR #1245: o cache de consultas foi mesclado com uma corrida de invalidação que o
    próprio repositório já reconhece no PR seguinte.
- **Dívida de verificação, D-17:** o PR #1242 afirma que a `main` (`03f0002`) está vermelha no Clippy
  em 13 erros. Se for verdade, a árvore mesclada desta hora não passa no próprio check — exatamente o
  que o D-17 do catálogo descreve.

## Confiança

- **Verificado:** os 19 commits, seus diffs e as contagens de linha por arquivo (patch da API); as
  medições citadas são as que o próprio diff/comentário registra, não medições desta rotina.
- **Verificado:** os 9 PRs abertos, números, títulos e autores; a ausência de merge na faixa.
- **NAO VERIFICADO:** o diff de qualquer PR. Em particular não sei se o #1246 repete o D-06 de fato,
  se o #1248 reintroduz o problema de D-01, nem se o #1242 descreve o estado real de `03f0002` — só o
  corpo do PR diz.
- **NAO VERIFICADO:** os `.txt.gz` das runs 281 e 282 aparecem como binários para a API; não foram
  descompactados.
