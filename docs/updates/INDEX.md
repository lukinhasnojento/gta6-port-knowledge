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
| 2026-11-26 | 19:14 | verificacao manual do tracker | Kelvaroq entra no topo; Belobug medido a 22 fps contra 5-7 no proprio README; AvatarManz consolida victory-fork + DexNom + Belobug |
| 2026-11-25 | 22:00 | — | execução de baseline: cursores gravados em `state/watch.json`, nenhum documento escrito |
| 2026-11-25 | 23:00 | 10 | 48 commits e 96 PRs (89 abertos, 7 mesclados) em `VIctory`, `xcl`, `Xbox2Windows`, `VIctory-Belobug-AMD`, `VIctory-DexNom`, `VItoriaBr`, `VIctory-7900XTX`, `victory-fork`, `TuxBox`, `SixNG`. Três repos sem delta: `OpenScarlett`, `xbox-emulator`, `GTA6-Asset-Info` |
| 2026-11-26 | 00:00 | 10 | 60 commits documentados de 63 na faixa (3 ficaram no backlog por teto de 40 em `Xbox2Windows`) e 101 PRs (75 abertos, 26 mesclados) nos mesmos dez repositórios. `VIctory` e `xcl` sem commit novo, com 24 e 9 PRs abertos. Sem delta em `OpenScarlett`, `xbox-emulator` e `GTA6-Asset-Info` |
| 2026-11-26 | 01:00 | 9 | 49 commits (todos documentados, nenhum corte de teto) e 70 PRs abertos + 20 mesclados detectados no cursor, em `Xbox2Windows` (31 commits, 18 mesclados), `victory-fork` (9, 1), `SixNG` (4, 1), `VIctory-7900XTX` (3, 0), `TuxBox` (2, 0) e, sem commit, `VIctory` (28 abertos), `xcl` (11), `VIctory-DexNom` (4), `VIctory-Belobug-AMD` (2). Backlog da hora anterior (`Xbox2Windows`) consumido. Sem delta em `VItoriaBr`, `OpenScarlett`, `xbox-emulator` e `GTA6-Asset-Info` |
| 2026-11-26 | 19:00 | 4 | 120 commits de 200 na janela (teto de 40 cortou 112→40 em `Xbox2Windows` e 48→40 em `SixNG`) e 49 PRs abertos de 257 detectados + 78 mesclados de 158, em `Xbox2Windows` (40 commits), `xcl` (13), `OpenScarlett` (27) e `SixNG` (40). Oito repositórios com delta ficaram em `backlog` pelo limite de 4 por execução |

O primeiro ciclo não documenta nada: grava os cursores em `state/watch.json` para que o ciclo
seguinte comece a contar a partir dali.

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
