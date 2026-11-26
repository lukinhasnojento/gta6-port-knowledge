# avatarmanz/VIctory-7900XTX — atualização de 2026-11-26 01:00

## Resumo

Três commits na `main` (`e2a28ec7` → `b0cfe2ba`), todos de `local` e nenhum vindo de PR: o merge da
branch `fps/fills`, que move a escrita das linhas do medidor de quadros para uma thread própria
(`VICTORY_METER_THREAD`, ligado por padrão) e acrescenta spans e contadores de diagnóstico; e um par
"merge GitHub main → remoção" que fez a linha `ZETTROX` do README entrar e sair em três minutos. Onze
PRs abertos, cinco deles da mesma série de diagnóstico de performance do `SousaTrashBin`.

## Commits novos (3)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `022ee74d` | local | Merge fps/fills: the frame meter's lines on a thread of their own (VICTORY_METER_THREAD), one hold for asking, timeline spans | Merge de dois pais (`f7fb366c`, `86de5cdd`), 4 arquivos, +244/−146. `crates/victory-gpu/src/host.rs` (+178/−125) adiciona `meter_lines()`, que envia o fechamento das linhas de 5 s ao canal de uma thread chamada `victory-meter` (mpsc, sem espera; se o envio falhar a closure roda na thread chamadora). O switch é lido uma vez em `OnceLock`: `VICTORY_METER_THREAD != "0"`, ou seja, **ligado por padrão**; em builds `diagnostics` há também `crate::ab::pick`. O comentário novo registra medições A/B: 88,8 ms/frame com o título submetendo contra 100,5 desligado (run fills-ab3) e 60–71 contra 64–65 com `VICTORY_THIN=1` (run fills-ab4). `host/card_steps.rs` (+16/−1) envolve a checagem de leitura em `crate::timeline::span("steps on the card: whether in the way")` e loga `place::READERS` por frame. `host/place.rs` (+30/−14) passa a classificar quem leu (outra cópia no lugar, cópia de buffer, fill, textura, passo nosso) em `READERS: [AtomicU64; 5]`. `watch.rs` (+20/−6) acrescenta o span `asking refused` e junta `ask_ahead` e `ask_rest` sob um único `Asking::hold_all` (`ask_ahead_held(ranges, held)`, `ask_rest(ranges, held)`), com o comentário de que o segundo hold esperava de novo as perguntas das outras threads (0,5 ms por vez, run fills-ab3). |
| `a6ba0a8d` | local | Merge GitHub main (README) | Merge de dois pais (`022ee74d`, `e2a28ec7`), +2/−0 em `README.md`: traz a linha `ZETTROX` que estava na `main` do GitHub (`e2a28ec7`, o cursor da execução anterior). |
| `b0cfe2ba` | local | README: ZETTROX line taken out | Um pai (`a6ba0a8d`), +0/−2 em `README.md`: remove a linha `ZETTROX` recém-mesclada. O resultado é que o README local volta a divergir da `main` do GitHub; a próxima sincronização traz a linha de novo e ela precisa ser removida outra vez. |

## Pull requests abertos (11)

Nenhum commit desta faixa veio de PR; os onze estavam abertos antes e continuam abertos.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 109 | Shaders: guard divergent single-console-wave barriers on narrow-wave (NVIDIA) GPUs | Rivaldodev, +245/−15 em 5 arquivos (`emit/lower.rs` +117/−11, `emit/mod.rs` +31/−4, `descriptor/host.rs` +26, `wave64_tests.rs` +58). Restringe a omissão automática de sincronização de barreira divergente a grupos de uma única wave de console (`group_threads <= console_wave`), preservando `TGSM_FENCE`/`UAV_FENCE` e `VICTORY_GROUP_SYNC`; estende o teste de slot nulo do heap com alocações pequenas, exatas e transbordando. | Não é D-xx. A área é wave/barreira; `D-11` (helper lanes fora das operações de wave na AMD) trata de `v_*` com lanes auxiliares, não de `s_barrier` divergente em wave32. |
| 108 | Loop limits of their own for named shaders (grid march 39f0e9: 12.3 -> 0.4 ms) | SousaTrashBin, +144/−3. `victory_shader::emit::with_loop_limit(Some(n), …)` limita as iterações por tradução na thread; `shader_cache::host_shader` tira o limite do hash de 16 dígitos do container em `VICTORY_SHADER_LOOP_LIMITS` (`hash:iterations,…`) e, sem a variável, aplica `39f0e9aeb156a61c:512`. Mede 12,3 → 0,3–0,5 ms/frame no shader de marcha de grade. | Não é D-xx. `D-08` (compilação de pipelines travando o título) é compilação, não execução; aqui é tempo de GPU por iteração de loop que nunca sai. |
| 107 | Read placed R9G9B9E5 views from the R32_UINT texture itself (no alias copies) | SousaTrashBin, +552/−2 (`resource.rs` +517). Com `RelaxedFormatCastingSupported`, texturas `R32_UINT` 2D com UAV passam a ser criadas por `CreateCommittedResource3` listando `R9G9B9E5_SHAREDEXP` como formato conversível, marcadas com private data; views posicionadas desse formato recebem o descritor na própria textura e o draw a move para estado de SRV, sem cópia. `VICTORY_ALIAS_CAST=0` desliga. Mede 8,7 → 0,1 ms/frame de cópias de alias (87 → 10 cópias). | Não é D-xx. É cópia por incompatibilidade de formato (alias), não o caminho de `Staging`/detile de `D-05`/`D-09`. |
| 106 | Timing runs: the card's time by kind of list | SousaTrashBin, +122/−14. Com `VICTORY_GPU_TIMING`, cada lista é cronometrada sozinha (incluindo preludes e o compare de cópias) e uma linha nova soma o tempo da placa por tipo: comandos do título, preludes, passos, compares, "o mínimo que uma lista levou". Só diagnóstico. | Não é D-xx. |
| 105 | Card steps: read fills through placed copies, gather reads, bound the pool | SousaTrashBin, +733/−45 (`host/card_steps.rs` +727). Quatro mudanças em `VICTORY_CARD_STEPS=2`: fills atrás de cópia no lugar leem a fonte da cópia (`VICTORY_PLACED_FORWARD`, ligado por padrão) e as submissões recusadas caem de 0,9 para 0 por frame; duas recusas novas fecham buracos de leitura velha (`PLACED_ON_CPU`, `PLACED_STALE`); leituras de um mesmo buffer são reunidas num dispatch (`VICTORY_CARD_GATHER`, ~2.100 comandos de cópia → 66 + ~18 dispatches); o pool de listas é dimensionado pela submissão. | Não é D-xx. `D-10` é "correção do replay feita e revertida" e `D-09` é device loss por textura mostrada antes do upload; este PR é o caminho de *card steps*, que nem está no catálogo. |
| 104 | Decode packed 16-bit BGR targets in diagnostic captures | Yoshi-233, +50/−4 em `capture.rs`. O dumper de alvos decodifica B5G6R5, B5G5R5A1 e B4G4R4A4 como palavras little-endian de 16 bits na faixa completa de cada canal, alfa opaco; teste 2×2 com padding de linha. Porta `fxceless/victory-fork#910`. | Não é D-xx. É ferramenta de captura, não renderização. |
| 100 | Open the window at 1280x720 by default (VICTORY_WINDOW_MAX_*) | kalijerry, +23/−0 em `host.rs`. Abre a janela em no máximo 1280×720 mantendo o aspecto, configurável por `VICTORY_WINDOW_MAX_WIDTH`/`_HEIGHT` (`0` = sem limite). Declara 326 testes e quadro inalterado num RTX 3080. | Não é D-xx. `D-19` (configuração documentada ≠ medida, `play.ps1` liga ~20 switches) é o padrão que este PR reforça: mais um par de variáveis de ambiente com padrão novo. |
| 98 | Streaming stores for the big copies of the preparing thread (snapshots, sends to card copies, refreshes) | Joly0, +277/−7. `VICTORY_STREAM_COPY` (padrão ligado) usa stores AVX2 para snapshots (~90 MB/frame) e envios de cópias ≥ 4 KiB; `VICTORY_STREAM_REFRESH` faz o mesmo em `safecopy::copy_streaming` para `Shadow::copy`. Mede mediana 34,2 contra 37,3 ms num cenário de ~900 draws. | Não é D-xx. Toca o caminho de snapshot/cópia de `watch`, área de `D-06` (corrida entre mapear uma view e abrir guards) e `D-09`, sem alterar a lógica de guarda. |
| 97 | Placed texture UAVs: land writes on the title resource, not a stand-in | kalijerry, +244/−25. Um UAV de textura posicionada era espelhado num stand-in que ninguém lia: as escritas caíam no stand-in e o construtor de estrutura de ray tracing não produzia nada, deixando os interiores pretos (só luz direta). Passa a manter um recurso por endereço, com o recurso do título ganhando de uma cópia sintética, `View::Unordered` para descritor feito do recurso do título e `ResourceRef` segurando referência. | **Não está no catálogo.** O sintoma ("interiores pretos, só luz direta, sem bounce") não tem linha em `AvatarManz`; é candidato a defeito novo, com correção neste PR ainda não mesclado. `D-03` (perda de luz indireta por `VICTORY_SEND_CHANGED` ligado por padrão) é outro caminho e já tem corretor. |
| 96 | Search free runs in eight shards, best fit, and size the pool for a capped heap | kalijerry, +381/−94 (`descriptor/host.rs` +355/−92). Troca o scan único sob um lock por oito pools de runs livres, cada um com lock próprio, escolhe o menor run que cabe e junta vizinhos ao devolver; dimensiona o heap para adaptador com teto (1.835.008 views num 3080 em vez do fallback de 1.000.000). Mede 2.800 draws/frame dropados → dígitos, e 0,30–0,40 s/frame dentro da busca → não reportado. | Não é D-xx **deste** repositório, mas é a mesma classe de `VIctory` `D-01` ("esgotamento de slots dinâmicos … remoção de device", corrigido `edf64e2`) e de `Belobug-AMD` `D-05` ("esgotamento de slots dinâmicos: draws dropados, frames de segundos", sem corretor). O `AvatarManz` não tem linha para exaustão de slots dinâmicos. |
| 92 | Run on a 16 GB PC: windows on the card, relief on out-of-memory, idle lists by memory | TheHoracio, +406/−12 (`host.rs` +293, `list.rs` +87). Janelas de memória do título vão para heap de upload da GPU enquanto a placa está abaixo de 85 % do orçamento, depois para memória compartilhada, depois para a placa até 92 %; em `E_OUTOFMEMORY` libera rascunhos, staging sem lista, listas ociosas e o mapeamento da pipeline library, e repete; pool de listas ociosas escala com a memória compartilhada (256/96/32/0). Mede estabilidade em 9,5–11,4 fps num RX 7800 XT 16 GB. | Não é D-xx. `D-01` (orçamento indireto esgotado) e `D-13` de `Belobug-AMD` (consumo de memória) são as linhas vizinhas; nenhuma descreve o estouro de memória compartilhada na entrada do mundo. |

## Pull requests mesclados (0)

Nenhum. Nesta faixa a `main` recebeu só commits diretos de `local` (acima).

## Cruzamento com o catálogo

- Corrige D-xx? Nenhum. Nenhum dos três commits toca um caminho com linha aberta no catálogo
  `AvatarManz` (`D-01`…`D-19`).
- Abre defeito novo? Dois candidatos, ambos de PR ainda aberto, não de commit:
  - `#97`: escritas em UAV de textura posicionada iam para um stand-in e sumiam, com o construtor de
    estrutura de ray tracing ativo — interiores sem luz indireta. Não existe linha correspondente no
    catálogo; o sintoma é próximo de `D-03`, mas a causa é outra.
  - `#96`: exaustão de slots dinâmicos de descritor em adaptador com teto de 1.000.000 de views, com
    2.800 draws dropados por frame. É `VIctory` `D-01` e `Belobug-AMD` `D-05` no código deste fork; o
    catálogo deste repositório não tem a linha.
- Padrão de switch: `022ee74d` liga `VICTORY_METER_THREAD` **por padrão**. É mais um switch novo com
  padrão ativo, exatamente o que `D-19` cobra ("a configuração documentada não é a configuração medida")
  e o que `D-16` descreve ("duas convenções opostas para o mesmo tipo de switch, em 294 variáveis"). O
  commit não atualiza `play.ps1` nem a lista de variáveis documentadas; NÃO VERIFICADO se o switch
  aparece em `docs/` ou em `scripts/play.ps1`.
- Divergência de README: `a6ba0a8d` mescla a `main` do GitHub (que tem a linha `ZETTROX`) e `b0cfe2ba`
  a remove de novo em três minutos. O repositório fica com a `main` local divergindo da remota nesse
  arquivo; a cada sincronização futura a linha volta. Não é defeito de catálogo, é risco operacional
  para a próxima execução desta rotina (o commit de merge não "corrige" nada e será documentado de novo).

## Confiança

Verificado por diff: os três commits foram lidos em `gh api .../commits/<sha>`, com pais conferidos —
`022ee74d` de dois pais `f7fb366c`/`86de5cdd`; `a6ba0a8d` de dois pais `022ee74d`/`e2a28ec7`;
`b0cfe2ba` sobre `a6ba0a8d`. `e2a28ec7` é o `last_commit` do estado anterior, então a faixa está
completa (3 commits, nenhum corte de teto). PRs lidos por `gh pr view`.

Não verificado: as medições A/B citadas nos comentários do código (runs fills-ab3/fills-ab4) e as dos
PRs não foram reproduzidas; são o que os autores declaram. Não verifiquei se `docs/` ou `play.ps1`
listam `VICTORY_METER_THREAD` — os arquivos alterados no commit são só os quatro de código. Não rodei
`cargo test`.
