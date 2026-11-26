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
| — | — | — | rotina criada em 2026-11-25; primeira execução fará apenas o baseline |

O primeiro ciclo não documenta nada: grava os cursores em `state/watch.json` para que o ciclo
seguinte comece a contar a partir dali.
