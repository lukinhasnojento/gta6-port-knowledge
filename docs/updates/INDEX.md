# Atualizações horárias

Registro incremental dos treze repositórios monitorados: commits novos, pull requests abertos e
pull requests mesclados, documentados a cada hora pela rotina descrita em
[`docs/automacao/ANALISE_HORARIA.md`](../automacao/ANALISE_HORARIA.md).

As análises históricas completas ficam em [`docs/analises/`](../analises/); aqui só entra o que mudou
depois do fechamento do acervo.

## Como navegar

```
docs/updates/AAAA-MM-DD/HH/<repo>.md    um arquivo por repositório com mudança
docs/updates/AAAA-MM-DD/HH/_falhas.md   erros daquela execução, se houver
```

## Índice

| Data | Hora | Repositórios com mudança | Resumo |
|---|---|---|---|
| 2026-11-26 | 23:00 | 4 | 81 commits documentados de 81 na faixa (25 em `VIctory-7900XTX`, 20 em `victory-fork`, 35 em `TuxBox`, 1 em `VIctory-DexNom`) e 40 PRs abertos de 98 detectados nos quatro + 7 mesclados de 7; `TuxBox` e `victory-fork` retirados do backlog. Seis repositórios com delta ficaram em `backlog` pelo limite de 4 por execução: `VIctory` (50 abertos), `xcl` (50 + 6 merges), `Xbox2Windows` (43 + 16), `VIctory-Belobug-AMD` (50 + 20), `SixNG` (12 commits, 14 abertos, 12 merges) e `xbox-emulator` (1 aberto). `VItoriaBr` saiu do backlog: sem delta |
| 2026-11-26 | 22:00 | 4 | 127 commits na janela, 109 documentados (32 em `Xbox2Windows`, 40 de 58 em `VIctory-7900XTX`, 31 em `TuxBox`, 6 em `SixNG`) e 33 PRs abertos de 63 detectados nos quatro + 19 mesclados em `main` (13 `TuxBox`, 6 `SixNG`); outros seis merges do `Xbox2Windows` foram para `cpp-migration`, não para `main`. Sete repositórios com delta só de PRs e `VIctory-7900XTX` com 18 commits ficam em `backlog` |
| 2026-11-26 | 21:00 | 4 | 130 commits documentados (`Xbox2Windows` 40 de 72, `VIctory-7900XTX` 40 de 85, `TuxBox` 34 de 34, `SixNG` 16 de 16) e 37 PRs abertos documentados de 67 detectados + 75 mesclados de 75. Quatro repositórios retirados do backlog; seis com delta só de PRs ficam em `backlog` |
| 2026-11-26 | 20:00 | 4 | 104 commits (40 em `VIctory-7900XTX`, 40 em `TuxBox`, 22 em `victory-fork`, 2 em `VItoriaBr`) e 72 PRs abertos documentados de 161 detectados + 62 mesclados documentados de 62, nos quatro repositórios retirados do backlog. Sete repositórios com delta ficaram em `backlog` |
| 2026-11-26 | 19:14 | verificacao manual do tracker | Kelvaroq entra no topo; Belobug medido a 22 fps contra 5-7 no proprio README; AvatarManz consolida victory-fork + DexNom + Belobug |
| 2026-11-25 | 22:00 | — | execução de baseline: cursores gravados em `state/watch.json`, nenhum documento escrito |
| 2026-11-25 | 23:00 | 10 | 48 commits e 96 PRs (89 abertos, 7 mesclados) em `VIctory`, `xcl`, `Xbox2Windows`, `VIctory-Belobug-AMD`, `VIctory-DexNom`, `VItoriaBr`, `VIctory-7900XTX`, `victory-fork`, `TuxBox`, `SixNG`. Três repos sem delta: `OpenScarlett`, `xbox-emulator`, `GTA6-Asset-Info` |
| 2026-11-26 | 00:00 | 10 | 60 commits documentados de 63 na faixa (3 ficaram no backlog por teto de 40 em `Xbox2Windows`) e 101 PRs (75 abertos, 26 mesclados) nos mesmos dez repositórios. `VIctory` e `xcl` sem commit novo, com 24 e 9 PRs abertos. Sem delta em `OpenScarlett`, `xbox-emulator` e `GTA6-Asset-Info` |
| 2026-11-26 | 01:00 | 9 | 49 commits (todos documentados, nenhum corte de teto) e 70 PRs abertos + 20 mesclados detectados no cursor, em `Xbox2Windows` (31 commits, 18 mesclados), `victory-fork` (9, 1), `SixNG` (4, 1), `VIctory-7900XTX` (3, 0), `TuxBox` (2, 0) e, sem commit, `VIctory` (28 abertos), `xcl` (11), `VIctory-DexNom` (4), `VIctory-Belobug-AMD` (2). Backlog da hora anterior (`Xbox2Windows`) consumido. Sem delta em `VItoriaBr`, `OpenScarlett`, `xbox-emulator` e `GTA6-Asset-Info` |
| 2026-11-26 | 19:00 | 4 | 120 commits de 200 na janela (teto de 40 cortou 112→40 em `Xbox2Windows` e 48→40 em `SixNG`) e 49 PRs abertos de 257 detectados + 78 mesclados de 158, em `Xbox2Windows` (40 commits), `xcl` (13), `OpenScarlett` (27) e `SixNG` (40). Oito repositórios com delta ficaram em `backlog` pelo limite de 4 por execução |

O primeiro ciclo não documenta nada: grava os cursores em `state/watch.json` para que o ciclo
seguinte comece a contar a partir dali.

Documentos de 2026-11-26 23:00: [`2026-11-26/23/`](2026-11-26/23/) — `VIctory-DexNom.md`,
`VIctory-7900XTX.md`, `victory-fork.md`, `TuxBox.md`. A faixa de `VIctory-7900XTX` desde `a9400a6d` foi
inteira documentada (25 commits, o cursor `commit_cursor` das 22:00 consumido) e `TuxBox` e `victory-fork`
saíram do backlog. `victory-fork` teve sete PRs mesclados em 22 segundos (`23:06:25` a `23:06:47Z`), cada
um com seu commit de merge; `TuxBox`, doze merges. Dos 98 PRs abertos detectados nos quatro, 40 foram
documentados (20 de 50 no `VIctory-DexNom`, 20 de 28 no `victory-fork`, os 18 do `VIctory-7900XTX` e os 2
do `TuxBox`); os 58 restantes ficam com `last_pr_number` no maior documentado. Cruzamentos que mudam o
catálogo: `victory-fork` **D-12** passa a ter correção com hash (`0a40a00a` e `b1d777a8`, cargas raw
pedindo só as dwords lidas e guardadas no fim da view); `victory-fork` **D-05** tem variante coberta por
PR aberto (#1391); `AvatarManz` **D-17** aparece como PR aberto (#393). Candidatos a linha nova:
`e9428969` (filtro de sondas magenta na AMD esconde dado inválido), `VIctory-7900XTX #104` (crash
`+0x5149aec` em `--no-rt` com placa de 8 GB), `VIctory-DexNom #106`/`#107` (waits de listas liberadas,
backlinks de contador UAV), `victory-fork #1389` (root constants vazios → `0xc0000005`), `TuxBox`
`ab644f02` (execução sem memória gráfica termina em fault de GPU) e `TuxBox e9134b07` (escrita em página
fora do write-watch passa a depender de consulta de `VirtualQuery` que pode envelhecer).

Documentos de 2026-11-26 22:00: [`2026-11-26/22/`](2026-11-26/22/) — `Xbox2Windows.md`,
`VIctory-7900XTX.md`, `TuxBox.md`, `SixNG.md`. Nenhum commit novo entrou em `main` do `Xbox2Windows`
desde 21:00; a faixa documentada (32) é o resto que o teto das 21:00 deixou, com `commit_cursor` consumido.
Em `VIctory-7900XTX` o teto de 40 cortou 58 → 40 e os 18 restantes ficam com `commit_cursor` em
`a9400a6d`. `TuxBox` teve 13 PRs e 31 commits (14 merge commits para 13 PRs: #268 foi mesclado duas
vezes, `6466bcf9` e `e76ff6a2`) e `SixNG` mesclou 6 PRs. Os seis PRs mesclados do `Xbox2Windows`
(`#2914`, `#2918`, `#2917`, `#2931`, `#2928`, `#2932`) têm `base.ref = cpp-migration` e foram registrados
sem contar como mudança de `main`; o par `#2918`/`#2931` repete o padrão do **D-29** (merge e revert no
mesmo dia). Achados para o catálogo: `9ba5661f` do `AvatarManz` refaz a correção de replay que o **D-10**
descreve como revertida; `#266`/`#268` do `TuxBox` cobrem casos de mesh que a correção do **D-21**
(`315b287`) não pegava; `be41e588` do `SixNG` corrige defeito de camada de roupa sem entrada no catálogo.
Tetos aplicados: 40 commits por repositório e 20 PRs abertos por repositório (`Xbox2Windows` 50 → 20,
cursor de abertos em `2887`). Nenhuma falha de coleta; nenhum `_falhas.md`; nenhum 403.

Documentos de 2026-11-26 21:00: [`2026-11-26/21/`](2026-11-26/21/) — `Xbox2Windows.md`,
`VIctory-7900XTX.md`, `TuxBox.md`, `SixNG.md`. Os quatro repositórios com commit pendente foram processados
antes dos que só têm PR aberto; o teto de 4 por execução deixou `Marvisak/VIctory`, `ivall/xcl`,
`Belobug/VIctory-Belobug-AMD`, `DexNom/VIctory`, `omonkeyz/xbox-emulator` e `fxceless/victory-fork` no
`backlog`. Sem ferramenta de subagente no ambiente, processados em sequência. Tetos: 40 commits por
repositório (cortou `Xbox2Windows` 72→40 e `VIctory-7900XTX` 85→40, com `commit_cursor` em `1983228b` e
`f03bfa51`) e 20 PRs abertos por repositório (`Xbox2Windows` 50→20). Divergência registrada: o cursor de
`TuxBox` deixado às 20:00 (`be9055c9`) estava atrás de 22 commits que aquela execução já documentara; a
faixa de 56 foi deduplicada contra os documentos anteriores e só 34 foram escritos. Nenhuma falha de
coleta; nenhum `_falhas.md`; nenhum 403.

Documentos de 2026-11-26 20:00: [`2026-11-26/20/`](2026-11-26/20/) — `VIctory-7900XTX.md`, `TuxBox.md`,
`victory-fork.md`, `VItoriaBr.md`. Os quatro saíram do `backlog` na ordem em que já estavam registrados. Sem
ferramenta de subagente no ambiente, foram processados em sequência. Tetos aplicados: 40 commits por
repositório (cortou `VIctory-7900XTX` 147→40 e `TuxBox` 84→40, com `commit_cursor` gravado em `306722bc` e
`be9055c9`) e 20 PRs abertos por repositório (`VIctory-7900XTX` 60→20, `TuxBox` 29→20, `victory-fork` 60→20,
`VItoriaBr` 12→12). O cursor de mesclados avançou até o `mergedAt` do último merge documentado por
repositório (08:17:50Z em `VIctory-7900XTX`, 19:34:00Z em `TuxBox`, 18:05:17Z em `victory-fork`), não até o
horário da execução. Nenhuma falha de coleta; nenhum `_falhas.md`. Achados que valem revisão do catálogo:
`VICTORY_SHOW_NEW_LATER` (`4cd2d133`+`91018a5d`+`cf9ba084`) ataca o mecanismo do **D-09** do `AvatarManz`,
que segue com `corrigido` vazio; `5cfe32df`+`d792ebd4` fecham só a metade "nome documentado que o código não
lê" do **D-19**; `c6ce497b` do `TuxBox` derruba o teto do cache NVIDIA de 32 GiB para 4 GiB, porque o driver
parou de indexar com ~5 GB — o **D-04** estava corrigido com um teto acima do ponto de quebra.

Documentos de 2026-11-26 19:00: [`2026-11-26/19/`](2026-11-26/19/) — `Xbox2Windows.md`, `xcl.md`,
`OpenScarlett.md`, `SixNG.md`. Execução atrasada: a tentativa do slot 02:00 foi interrompida antes de gravar
qualquer coisa e o estado não avançou, então esta janela cobre 01:15→19:35 local (~18 h) e não uma hora. Sem
ferramenta de subagente disponível, os quatro repositórios foram processados em sequência; os outros oito com
delta (`VIctory`, `VIctory-Belobug-AMD`, `VIctory-DexNom`, `VItoriaBr`, `VIctory-7900XTX`, `victory-fork`,
`xbox-emulator`, `TuxBox`) ficaram em `backlog`. Tetos aplicados e registrados: 40 commits por repositório
(`Xbox2Windows` 112→40, `SixNG` 48→40, `commit_cursor` gravado) e 20 PRs abertos por repositório
(`Xbox2Windows` 63→20, `xcl` 185→20). O cursor de PRs mesclados avançou até o `mergedAt` do último merge
documentado (06:27:26Z em `Xbox2Windows`, 18:31:19Z em `SixNG`), não até o horário da execução, para não pular
merges não documentados. Nenhuma falha de coleta; nenhum `_falhas.md`.

Documentos de 2026-11-26 01:00: [`2026-11-26/01/`](2026-11-26/01/) — `Xbox2Windows.md`,
`victory-fork.md`, `SixNG.md`, `VIctory-7900XTX.md`, `TuxBox.md`, `VIctory.md`, `xcl.md`,
`VIctory-DexNom.md`, `VIctory-Belobug-AMD.md`. Nenhuma falha de coleta. Duas notas de método:
(i) a busca `gh pr list --state merged --search "merged:>=<cursor>"` devolveu lista vazia nos treze
repositórios (índice de busca atrasado) e os mesclados foram lidos por
`gh api .../pulls?state=closed&sort=updated` filtrando `merged_at`, o que recuperou 20 merges;
(ii) 16 merges adicionais (`#333`, `#338`, `#345` em `SixNG` e 13 em `Xbox2Windows`) têm `mergedAt`
anterior ao cursor de `last_run` mas commit posterior ao `last_commit`, e foram documentados como commits
com a divergência registrada em Confiança. Candidatos a linha nova no catálogo: `AvatarManz #97`
(UAV de textura posicionada escrevendo em stand-in, interiores sem luz indireta), `DexNom #66` (slot de
heap lido como CBV, fault de GPU na RDNA 4), `victory-fork #1258` / `DexNom #68` / `AvatarManz #96`
(exaustão do anel de slots dinâmicos com teto de dispositivo), `SixNG #346` (tabela `EDUP` dos pacotes de
conteúdo não lida) e `Xbox2Windows #2679`/`#2685`/`#2709` (regressão de descritor vivo, custo de
`VirtualQuery`, lock de save preso).

Documentos de 2026-11-26 00:00: [`2026-11-26/00/`](2026-11-26/00/) — `VIctory.md`, `xcl.md`,
`Xbox2Windows.md`, `VIctory-Belobug-AMD.md`, `VIctory-DexNom.md`, `VItoriaBr.md`,
`VIctory-7900XTX.md`, `victory-fork.md`, `TuxBox.md`, `SixNG.md`. Nenhuma falha de coleta.
Desvio registrado: PRs mesclados passaram a ser detectados por `mergedAt` (não por
`last_pr_number`), porque a numeração de PR não acompanha a ordem de merge nos repositórios
monitorados; o teto de 40 commits deixou `404446e8`, `41ad992e` e `2a801192` de `Xbox2Windows`
para a próxima execução.

Documentos de 2026-11-25 23:00: [`2026-11-25/23/`](2026-11-25/23/) — `VIctory.md`, `xcl.md`,
`Xbox2Windows.md`, `VIctory-Belobug-AMD.md`, `VIctory-DexNom.md`, `VItoriaBr.md`,
`VIctory-7900XTX.md`, `victory-fork.md`, `TuxBox.md`, `SixNG.md`. Nenhuma falha de coleta na hora.
