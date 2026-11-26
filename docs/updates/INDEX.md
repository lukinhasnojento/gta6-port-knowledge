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
| 2026-11-25 | 22:00 | — | execução de baseline: cursores gravados em `state/watch.json`, nenhum documento escrito |
| 2026-11-25 | 23:00 | 10 | 48 commits e 96 PRs (89 abertos, 7 mesclados) em `VIctory`, `xcl`, `Xbox2Windows`, `VIctory-Belobug-AMD`, `VIctory-DexNom`, `VItoriaBr`, `VIctory-7900XTX`, `victory-fork`, `TuxBox`, `SixNG`. Três repos sem delta: `OpenScarlett`, `xbox-emulator`, `GTA6-Asset-Info` |
| 2026-11-26 | 00:00 | 10 | 60 commits documentados de 63 na faixa (3 ficaram no backlog por teto de 40 em `Xbox2Windows`) e 101 PRs (75 abertos, 26 mesclados) nos mesmos dez repositórios. `VIctory` e `xcl` sem commit novo, com 24 e 9 PRs abertos. Sem delta em `OpenScarlett`, `xbox-emulator` e `GTA6-Asset-Info` |

O primeiro ciclo não documenta nada: grava os cursores em `state/watch.json` para que o ciclo
seguinte comece a contar a partir dali.

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
