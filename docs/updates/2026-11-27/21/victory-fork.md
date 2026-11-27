# fxceless/victory-fork — atualização de 2026-11-27 21:00

## Resumo

Um único commit entre o cursor `3e573d25` e o head `f299fba1`, e ele é tudo menos estreito: 19
arquivos, +1.914/-108, envio direto na `main` (um pai, sem PR, `committer` = o próprio autor). O
assunto é "water light on 32-lane cards, cheaper ray walkers, no dropped draws", e o diff entrega cinco
frentes: um contador de threads por grupo no tradutor de shader (ported de `VIctory-NVIDIA 854f57f`,
que por sua vez porta `Vice-Tay c125c0c`), a preservação de *skips* em volta de loops com teste escalar,
o valor inicial do LDS do grupo, o reaproveitamento de consultas ao diretório de memória do título pelos
ray walkers, e um portão que segura a tradução de listas quando os slots dinâmicos de descritor estão
curtos. O launcher passa a ligar `VICTORY_WATER_LIGHT=1` por padrão. Nenhum PR mesclado; três abertos
acima de #1487.

## Commits novos (1)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `f299fba1` | fxceless | Water light on 32-lane cards, cheaper ray walkers, no dropped draws; README updated | Ver a decomposição por arquivo abaixo. Um pai (`3e573d25` = cursor), 19 arquivos, +1.914/-108 |

Decomposição por área, pelo diff:

- **Contador de threads por grupo (`crates/victory-shader/src/emit/lower.rs`, função de ticket e o
  ponto em `Lower` que a usa):** com lanes de console num grupo de várias waves do host (o caso que
  não é `halves`), as waves passam a tirar seus threads de console de um contador em LDS, e não do id
  plano arredondado. O comentário do código dá o motivo: "o id plano arredondado não é a base de uma
  wave do host em todo host (#1243)", e cita que a NVIDIA preenche grupos 2-D a partir de tiles 16×2 —
  com isso alguns threads de um grupo rodavam duas vezes e outros nunca, o que deixava buracos em
  bloco no passe de transmissão da água. `VICTORY_THREAD_TICKET=0` volta ao comportamento anterior.
- **Skips em volta de loops com teste escalar (`ir.rs`, `without_skips(p, deep, loops_kept)`):** a
  varredura que remove `skips` ganha um terceiro parâmetro e uma regra: se dentro da região há um
  `s_cbranch_scc0/scc1` cujo SCC vem de `s_cmp_*`/`s_bitcmp` e cujo destino volta para o próprio bloco,
  a região **não** é removida. O comentário explica o dano que a remoção causava: andado com `EXEC`
  zerado, o laço usa limites lidos dos registros de lanes que nunca escreveram, roda até o limite de
  iteração (4.096 voltas de barreira de grupo) e o grupo inteiro abandona o programa — é o laço sobre os
  slices de um tile do volume light da água. O `emit/mod.rs` tenta primeiro a forma "without skips,
  loops kept" e só depois a anterior, e a escolhe por programa (`VICTORY_WAVE_LOOP_SKIPS=0` desliga).
- **LDS inicial do grupo (`lower.rs`, `lds_start`):** acha o primeiro store de constantes em endereço
  constante que precede todo outro acesso a LDS e o repete no começo de cada grupo. A justificativa é
  que o console não zera o LDS: um programa que inicializa em `thread 0` atrás de early-outs não
  inicializa num grupo cujo thread 0 não está vivo, e esse grupo trabalhava com o que a memória do host
  tinha. `VICTORY_LDS_START=0` desliga.
- **Ray walkers (`lower.rs` `memory_near`/`memory_page`/`memory_entry`/`memory_words`/`look_before`,
  `emit/intersect.rs`):** um load por endereço de uma página de 64 KiB que um load anterior **já
  consultou** reaproveita a view e o offset daquela consulta em vez de refazer clamp, duas leituras de
  dword no diretório, o teste de página presente e a escolha de view no heap. A escolha da consulta
  anterior é restrita a blocos que dominam o load (ou ao mesmo turno do laço interno), e a comparação
  de página continua sendo feita em runtime — o comentário afirma que "a mesma página é a mesma entrada
  do diretório, que nenhum shader escreve". O segundo metade de um nó de 128 bytes passa a usar a view
  da primeira metade; um passo de triângulo faz uma consulta em vez de duas, e entra em instância uma
  vez em vez de nove. `VICTORY_MEMORY_NEAR=0` desliga.
- **Slots dinâmicos de descritor (`crates/victory-gpu/src/list/thin.rs` +217/-11, `list.rs`,
  `descriptor/host.rs`, `queue/behind.rs`, `draw.rs`, `host.rs`):** novo portão `VICTORY_THIN_GATE`
  (`gate()`/`gated()`/`park()`/`unparked()`): uma gravação de lista que está sendo traduzida é
  **estacionada** entre dois fluxos de palavras enquanto os slots dinâmicos estão curtos, exceto se for
  das listas que serão enviadas a seguir (`submitted && presents >= frame`), que nunca são seguradas.
  Limiares: livre < 1/4 do total sempre; livre < 5/8 enquanto houver lista submetida não traduzida. Teto
  de 250 ms por gravação segurada, e quem chama `wait()` nunca é segurada. O comentário mede o defeito
  numa execução: 15 de 106 segundos com draws sem slot, 15 a 340 draws por frame perdidos, com 247.000
  a 256.000 slots presos em listas que ainda não tinham sido enviadas. `list.rs` passa a marcar
  `slots_at(1)`/`slots_at(2)` para a linha de contadores. `VICTORY_THIN_GATE=0` desliga;
  `VICTORY_DYNAMIC_WAIT=alt` alterna ligado/desligado na mesma execução, para comparar.
- **Instrumentos (só `diagnostics`):** `capture.rs` (F9 salva um frame com todas as texturas que os
  passes escreveram), `copystat.rs` (tempo de draw por registro e por onde os buffers estão),
  `shadow.rs`, `draw.rs` (censo de janelas de memória do título), `queue/behind.rs` (`presents()`).
- **Launcher (`apps/victory/src/play.rs` +2):** `VICTORY_WATER_LIGHT=1` entra na tabela `SETTINGS`, com
  o comentário "runs (it is left out on 32-lane hosts otherwise, make.rs)".

## Pull requests abertos (3)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #1493 | Fix formatting issue in texture.rs blocking Actions | base `main`, head `patch-1`: formatação em `texture.rs` que bloqueia o CI | Não |
| #1491 | Audio DSP: compute mono convolutions, so TVs, phone calls and reverb are heard | base `main`, head `dsp-convolution`: convolução mono no DSP de áudio | Não. Nenhuma entrada de áudio no catálogo deste repositório |
| #1489 | GPU: restore indirect and card-copy shaders under Proton | base `main`, head `proton-indirect-card-compiler-20261127`: restaurar shaders indiretos e de cópia de card sob Proton | Área de D-02/D-03 do catálogo (janelas de indirect draw; forma wave64 derrubando o device) — **área, não o defeito**; patch não lido |

## Pull requests mesclados (0)

Nenhum `merged_at` acima de `last_run` (`2026-11-27T13:05:00Z`) — o commit da janela é envio direto,
não merge. O cursor de commits avança; o de merges não.

## Cruzamento com o catálogo

- **D-01 ("forma wave64 *wide* derruba o device") e D-04 (reversão e reaplicação da mudança de
  lane-mask em um minuto):** o contador por grupo mexe exatamente na correspondência entre lanes do
  console e waves do host — a área de D-01/D-04. Não é a mesma falha: D-01 era a forma wide escolhida
  para um programa; aqui é a origem do id de thread dentro de um grupo de várias waves. **Não corrige
  D-01 nem D-04.**
- **D-12 ("loads escalares agrupados de 4 dwords leem além do fim da view — água preta"):** a mudança de
  `memory_near` também reaproveita views em loads de walker, mas o mecanismo é outro (reúso de consulta
  ao diretório, não agrupamento escalar). **Não corrige D-12.**
- **Achado verificável — a chave do cache de shader não inclui três dos switches novos.**
  `crates/victory-gpu/src/shader_cache.rs` lista, no `build()` da chave, 14 variáveis e **acrescenta
  `VICTORY_THREAD_TICKET`** (+1 linha no diff), mas não lista `VICTORY_MEMORY_NEAR`,
  `VICTORY_LDS_START` nem `VICTORY_WAVE_LOOP_SKIPS` — os três mudam o código gerado (o próprio
  comentário de `memory_near` em `emit/mod.rs` afirma "Part of the translation: the shader cache keys on
  it"), e `host_settings()` devolve só `[host_max_wave, view_heap_limit, raw_views, host_rdna]`. É
  exatamente o mecanismo de **D-02** ("chave do cache não incluía os switches de wave — traduções
  envenenadas", corrigido em `e9765a7`): ligar/desligar `VICTORY_MEMORY_NEAR` (ou `LDS_START`, ou
  `WAVE_LOOP_SKIPS`) entre duas execuções pode servir a tradução cacheada da configuração anterior.
  **Reabre a classe de D-02 por três switches novos.** Arquivos: `shader_cache.rs` (lista da chave),
  `emit/mod.rs` (`memory_near`), `emit/lower.rs` (`lds_start`, `ticket_on`).
- **`VICTORY_WATER_LIGHT`:** é default do launcher (`play.rs`), citado como pertencente a `make.rs` no
  comentário; não aparece em `emit/mod.rs` nem em `emit/lower.rs` e não está na chave do cache. Se ele
  muda o código traduzido, cai na mesma lacuna acima. **NÃO VERIFICADO** qual é o efeito dele.
- **D-16 ("perdas de device sem causa identificada"):** o commit não alega corrigir; o README continua
  listando "foam on the water is drawn dark" e "as páginas dos programas de ray podem ficar para trás em
  viagem rápida".

## Confiança

Verificado por mim, pelo diff e pelo código no ref:

- hash, autor, data, contagem de arquivos e de linhas vieram da API; `parents` = 1 (`3e573d25`), ou
  seja, envio direto, sem merge e sem PR;
- li os hunks de `list/thin.rs` (portão, `parked`, `held_back`, `unparked`, limiares e o teste
  `the_gate_holds_back_all_but_the_lists_sent_next`), de `ir.rs` (regra `loops_kept` e os testes
  reescritos com o terceiro parâmetro), de `shader_cache.rs` (+1 na lista da chave), de
  `emit/mod.rs` (tentativa "loops kept" antes da forma anterior, `memory_near`) e os cabeçalhos de
  hunk de `lower.rs` com os comentários de `memory_near`, `lds_start`, `ticket` e `look_before`;
- **a lacuna da chave de cache foi conferida nos dois lados**: li a lista completa de variáveis em
  `shader_cache.rs` no ref `f299fba1` e a definição de `host_settings()` em `emit/lower.rs`; nenhuma das
  três aparece em nenhuma das duas;
- li `README.md` e o diff de `play.rs`, que fixam o default `VICTORY_WATER_LIGHT=1` e o texto do que
  ainda está errado.

NÃO VERIFICADO:

- **nada foi executado.** Não compilei nem rodei `cargo test`; não há GPU, driver nem cena aqui. Os
  números do commit — ~15 fps a 3.000-5.000 draws contra ~6 antes; 8 a 5.000-12.000; 320 leituras sem
  draw sem slot; "86 testes de victory-gpu, 14 de wave com 1 falha que também falha na árvore
  inalterada" — são do autor;
- **a premissa do `memory_near`.** O código afirma que nenhum shader escreve o diretório de memória do
  título, então a mesma página é sempre a mesma entrada; não há no diff uma verificação disso, e é a
  premissa da qual depende não ler dado errado. Se ela falhar num título que escreva o diretório, o
  walker lê pela view antiga;
- **o efeito real do contador de ticket em hardware NVIDIA.** O commit diz que a forma anterior rodava
  threads duas vezes e deixava outros de fora; não há teste de hardware no diff que compare as duas
  formas num grupo de várias waves — os testes de `wave64_tests.rs` (+207) são de unidade;
- **os 18% de instruções e o custo dos ray walkers** (30 de 85 ms por frame, execução 410): medição do
  autor, não reproduzida;
- **os três PRs abertos**: li número, título, base e head; não abri patch nem corpo;
- **se `VICTORY_THREAD_TICKET` na chave basta**: a chave inclui o valor da variável, mas o commit não
  diz o que acontece com entradas gravadas antes desta mudança (a chave muda porque a lista mudou, então
  o conjunto inteiro é novo — verifiquei a lista, não a versão da chave).
