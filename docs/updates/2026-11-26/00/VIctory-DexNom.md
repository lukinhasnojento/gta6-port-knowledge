# DexNom/VIctory — atualização de 2026-11-26 00:00

## Resumo

Sem commit novo: a `main` continua em `64bd58f5fb794216894bc0e33e03a3a64d19c347`. Dois pull requests
abertos acima de 62, ambos grandes: um corrige o alvo do copy-back de textura em relação ao mapeamento
gravado, o outro porta para 32 lanes a forma emparelhada dos programas wave64 de compute.

## Commits novos (0)

`gh api repos/DexNom/VIctory/commits?sha=main&per_page=100` devolve `64bd58f5` como primeiro elemento,
igual ao `last_commit` do estado.

## Pull requests abertos (2)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 63 | GPU: bind texture readback to recorded mapping | `texback::record` passa a capturar `ResourceRef`, layout, faixa do convidado e `ReadbackMapping` no momento em que o primeiro escritor é gravado, em vez de só no `Close` da lista; o copy-back adiado usa esse mesmo alvo e a checagem de mapeamento existente recusa pouso obsoleto (`resource.rs` +104, `texback.rs` +31, `host.rs` +8). Sem mudar a política de sobreposição. | Não é D-xx deste repositório. `D-07` (operações de quad em pixel rebaixadas) e `D-10` (`Staging::detile` escrevendo além do buffer) são outros defeitos de cópia; este é janela de vida entre gravação e fechamento da lista. |
| 64 | Run wave64 compute programs in a paired form on 32-lane hosts (port of fxceless#854, stacked on #61) | Porta `26600c6` de `fxceless/victory-fork#854`: cada wave do console roda como uma wave de 32 lanes do host e cada thread faz o trabalho de duas lanes (`n` e `n + 32`), com EXEC como guarda por lane, tentado depois de todas as outras formas. `VICTORY_WAVE_PAIRS=0` desliga. Diff de 34 arquivos (+5563/-1433), empilhado sobre `#61`. O corpo diz que `#61` levou 22 de 31 pipelines wave64-only a funcionar. | Não é D-xx. `D-03` (máscara wave64 restaurada por dois `s_mov_b32`) é outro defeito de wave64. `VItoriaBr D-20` e `AvatarManz D-11` cobrem perdas de device por forma wave64 em outros forks. |

## Pull requests mesclados (0)

Nenhum PR com `mergedAt` posterior a 2026-11-26T02:11:35Z. A API devolve `mergedMax = 0`, ou seja, este
repositório não tem nenhum PR mesclado registrado — os commits entram direto na `main`.

## Cruzamento com o catálogo

- Corrige D-xx? Nenhum dos 21 itens de `catalogo/defeitos.csv` deste repositório.
- Abre defeito novo? Não verificável. `#64` é o candidato de risco: a forma emparelhada em 32 lanes é
  exatamente a classe de mudança que, em `VItoriaBr D-20`, terminou em perda de device por barreiras
  dentro de branches. Nada no PR mede isso — não há log de execução no repositório desta hora.

## Confiança

Verificado: topo da `main`, ausência de delta, números, corpos, arquivos e linhas dos 2 PRs, ausência de
merge na janela (`mergedMax = 0`).

Não verificado: se `#64` realmente roda em jogo. O diff foi lido pela lista de arquivos e pelos corpos;
o commit de revisão indicado (`40c2f29`) não foi aberto.
