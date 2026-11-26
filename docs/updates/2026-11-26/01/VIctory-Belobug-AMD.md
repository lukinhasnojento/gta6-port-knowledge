# Belobug/VIctory-Belobug-AMD — atualização de 2026-11-26 01:00

## Resumo

Sem commit novo na `main`: o cursor continua em `4f62c2a5` e nenhum PR mesclou nesta faixa. Dois PRs
abertos: um corrige a histerese do controle de VRAM de 16 GB (as faixas de admissão e de despejo usavam
percentuais diferentes do mesmo cálculo, deixando 1,24 GiB presos num RX 9070 XT), o outro paraleliza a
atualização das texturas novas de uma submissão para reduzir pop-in.

## Commits novos (0)

Nenhum. `last_commit` `4f62c2a5e27e7beebf3e8d68f77898e43dc7b77c` é o `head` da `main`.

## Pull requests abertos (2)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 223 | GPU: align texture room_percent below vram_keep and fix >=90% eviction target | zaixiakongyiji, +115/−25 em `host.rs` (+36/−12) e `resource.rs` (+79/−13). O commit `81ebb7b` da branch `rx9070xt` baixou a marca d'água de despejo de `vram_keep` (88→85 %, 82→78 %) e condicionou a retenção de slots do `keep_shown` a `resource::room_lately()`, mas deixou duas incoerências no laço de VRAM de 16 GB: `resource::room()` e `resource::let_go_idle()` calculam o mesmo uso efetivo, e `room_percent()` continuava em 90 % (13.905 MB do orçamento local de 15.450 MB do RX 9070 XT) — uma faixa de admissão 82–90 % contra despejo em 82 %, 1,24 GiB que nunca volta; e o alvo de despejo `>=90 %` não era atingido porque o teste comparava o percentual errado. O PR alinha os dois e corrige a comparação. | Não é D-xx. `D-05` (esgotamento de slots dinâmicos de descritor) e `D-13` (consumo de memória: 22 GB de RSS, 33 GB de commit, mais VRAM que a placa) são as linhas de memória do catálogo; esta é a histerese do teto de texturas, sem linha. |
| 222 | Refresh the textures new to a submission on several threads (less texture pop-in on a Radeon 8060S; no frame-rate change shown) | SPFCChris, +30/−5 em `host.rs`. `Early::take` lia, calculava hash e detilava uma a uma, na thread que submete, as texturas que ficaram visíveis a slots bindless de uma vez durante streaming; numa Radeon 8060S é onde as texturas chegam atrasadas depois de corte de cena. Coleta as texturas devidas e processa em várias threads. O autor declara explicitamente que julgou o resultado na tela e por contagem de slots, **não** por frame rate, e mediu em cima de `#148`, `#178` e `#179`, não na `main`. | Não é D-xx. `D-15` ("água desaparece até reiniciar depois de reconstruir o cache de shaders") e `D-09` ("run de blocos guardados que cruza views falhava inteiro") são as linhas de upload/refresh mais próximas; nenhuma descreve atraso de textura por detile serial. |

## Pull requests mesclados (0)

Nenhum. A `main` não tem commit novo; a varredura por `merged_at` em `pulls?state=closed` não achou merge
posterior a 2026-11-26T03:16:53Z.

## Cruzamento com o catálogo

- Corrige D-xx? Nenhum. As 17 linhas seguem como estão; `D-05`, `D-06`, `D-12`, `D-13`, `D-14`, `D-15` e
  `D-17` continuam sem corretor, e `D-17` é **CONFIRMADA** com introdução em `0a32e89` (`VIEW_SLOTS`
  2.294.784 acima do limite tier-2 de 1.000.000: o padding nunca é criado). `#223` mexe em orçamento de
  VRAM, não no tamanho do heap de views; não toca `D-17`.
- Abre defeito novo? Nenhum evidente. `#223` corrige uma incoerência introduzida pelo próprio `81ebb7b`
  (branch `rx9070xt`, não a `main`) — se mesclar, é correção de regressão de branch, sem linha no
  catálogo.
- Nota de método: `#222` não mostra ganho de frame rate e mede sobre três outros PRs. Se entrar, o
  catálogo não deve ganhar linha de "correção" — o próprio autor não afirma melhora mensurável.

## Confiança

Verificado por API: `gh api repos/Belobug/VIctory-Belobug-AMD/commits?sha=main&per_page=100` devolve
`4f62c2a5` como `head`, igual ao `last_commit` do estado — zero commits. `gh pr list --state open
--limit 50` devolveu 2 PRs (#222, #223), ambos com número > `last_pr_number` 221. A varredura de
mesclados não achou `merged_at` posterior ao cursor.

Não verificado: os números de VRAM (13.905 / 15.450 MB, 1,24 GiB) e o pop-in na Radeon 8060S são
declarações dos autores; não rodei nada. Não li o commit `81ebb7b` (está numa branch, não na `main`).
