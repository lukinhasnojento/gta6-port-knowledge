# avatarmanz/VIctory-7900XTX — atualização de 2026-11-25 23:00

## Resumo

Um único commit novo, de duas linhas, que acrescenta a palavra `ZETTROX` ao fim do `README.md` — sem
contexto, sem seção, sem link. Cinco PRs abertos, todos de otimização de memória, guard pages e
contagem de threads em hosts de 32 lanes.

## Commits novos (1)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `e2a28ec7` | avatarmanz | Add ZETTROX to README | `README.md` +2/-0. O patch inteiro é uma linha em branco e a linha `ZETTROX`, anexadas depois do parágrafo de licença (GPL-3.0-or-later / `docs/asset-policy.md`). Não há seção, título, link, crédito com função ou qualquer texto em volta. Co-autoria declarada: `Claude Opus 5.5`. É o único arquivo tocado |

Esse é o diff completo; não há mais nada no commit. O efeito no documento é uma linha solta no fim do
arquivo.

## Pull requests abertos (5)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 63 | Retry incomplete pipeline book writes and flush on orderly exit | **Fixes #41.** Se um dos arquivos de cache falhar ao abrir, o `book` lembrava a chave mas só o segundo pipeline chegava ao disco; propõe retentar e descarregar na saída ordenada | Não consta no catálogo `AvatarManz` (D-01…D-19). Fica na área de D-08 (compilação de pipelines travando o título) sem ser o mesmo defeito |
| 66 | Shaders: count threads of wave64 builder programs on 32-lane hosts (NVIDIA) | `counted_in_threads` pula os programas wave64 unidimensionais do builder; a proposta é contá-los em threads no host de 32 lanes | Não consta. Vizinho de D-12 (contagem de dispatch de ray tracing tomada como grupos em vez de threads) — mesma classe, outro caminho |
| 68 | GPU: keep a title guard page inside a range; change guards in runs | Guard page do título dentro de bloco protegido perdia a proteção; propõe manter a guarda e mudar guardas em execuções. Declara não depender do PR #14 | Não consta. Área de D-09 (device loss a cada ~16 min), defeito diferente |
| 69 | GPU: add optional memory category log | Linha `VICTORY_MEMORY_CATEGORIES=1` desligada por padrão, para julgar reduções de memória contra working set e commit headroom | Não |
| 72 | victory-gpu: fence-tracked LRU VRAM GC and upload buffer pooling | Mapa reverso `Weak` em `HostHeap::kept` para despinar descritores bindless antes de `let_go_idle`; pooling de upload/staging por fence; GC LRU de VRAM | Não consta. Área de D-09/D-14 (device loss e índice de view além do heap), sem declarar correção |

## Pull requests mesclados (0)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| — | — | — | nenhum PR com número > 58 aparece como mesclado |

## Cruzamento com o catálogo

- **Corrige D-xx?** Não. `e2a28ec7` toca só `README.md`; nenhum D-01…D-19 tem correção por este hash.
- **Abre defeito novo?** Um defeito documental menor, e vale registrar porque o catálogo já tem
  entradas dessa natureza (D-17 registra o README que nega código de XCL; D-19 registra configuração
  documentada ≠ medida): o README termina agora com uma linha `ZETTROX` sem explicação, provavelmente
  um crédito que perdeu o contexto. Quem lê o arquivo não sabe o que é.
- Continua aberto no catálogo: **D-01** (orçamento indireto esgotado por registro lixo), **D-08**
  (compilação de pipelines travando dezenas de segundos), **D-09** (device loss a cada ~16 min por
  textura mostrada antes do upload), **D-15** (patch duplicado por cherry-pick), **D-18** (constantes
  de layout fixas por título). Nada desta faixa os toca.

## Confiança

- **Verificado:** o commit `e2a28ec7` inteiro (patch de 2 linhas, `README.md` único).
- **Verificado:** os 5 PRs abertos, números, títulos e autores.
- **NAO VERIFICADO:** o diff dos 5 PRs; em particular se o #66 realmente corrige a contagem de threads
  ou só desloca o critério, e se o #72 mexe no caminho de D-14.
