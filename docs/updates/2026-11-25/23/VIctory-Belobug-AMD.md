# Belobug/VIctory-Belobug-AMD — atualização de 2026-11-25 23:00

## Resumo

Nenhum commit novo: `4f62c2a5e27e7beebf3e8d68f77898e43dc7b77c` continua no topo da `main`. O único
delta é o PR #215, que propõe preparar uma submissão enquanto a anterior ainda corre — opt-in, com
ganho medido em cima de outros dois PRs ainda abertos.

## Commits novos (0)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| — | — | — | nenhum commit novo desde `4f62c2a5e27e7beebf3e8d68f77898e43dc7b77c` |

## Pull requests abertos (1)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 215 | A submission is prepared while the one before still runs (opt-in; median frame time 12 % lower on a Radeon 8060S on top of #178 and #179) | Preparar a submissão seguinte enquanto a anterior ainda está na GPU, atrás de dois switches desligados por padrão. O autor declara que **sozinho na `main` não conseguiu mostrar ganho** — o número de 12 % é medido sobre #178 e #179 | Não. Fica perto de D-08 (tempestade de faults de 2,5 GB/frame no `watch::guard`) e D-13 (22 GB de RSS), mas não é correção de nenhum dos dois |

## Pull requests mesclados (0)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| — | — | — | nenhum PR com número > 214 aparece como mesclado |

## Cruzamento com o catálogo

- Sem commit novo, nenhum D-xx de `Belobug-AMD` pode ter sido fechado por hash nesta hora. Os 17
  defeitos do catálogo continuam com o mesmo estado, incluindo D-05, D-06, D-12, D-13, D-14, D-15 e
  D-17, que estão sem correção registrada.
- O PR #215 não declara corrigir defeito; declara ganho de tempo de frame. Se ele reduz a janela em
  que o `watch::guard` enxerga páginas sujas (D-08), é consequência, não objetivo — **NAO
  VERIFICADO**.

## Confiança

- **Verificado:** ausência de commit novo (cursor `4f62c2a…` no índice 0 de 100).
- **Verificado:** existência do PR #215, título e autor (`SPFCChris`).
- **NAO VERIFICADO:** o diff do PR #215; a proposta é o que o corpo declara, inclusive a ressalva de
  que o ganho não aparece na `main` isolada.
