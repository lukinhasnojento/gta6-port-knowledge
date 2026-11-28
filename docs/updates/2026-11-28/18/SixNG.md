# holdmysocks/SixNG — atualização de 2026-11-28 18:00

## Resumo

Um commit e um merge na janela: `#555` resolve nomes de decoração que são **conjuntos** (struct de
catálogo `0x91501997`) para os registros que os compõem. Antes, esse tipo de nome não achava registro e
era descartado, então personagem cuja tatuagem de tronco vem por conjunto ficava sem nenhuma. Diff de 60
linhas em dois arquivos, com bump de cache de catálogo para 6.

## Commits novos (1)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `cff3f31b` | Brandon Villar | Decorations: resolve decoration set names to their records (#555) | Em `crates/sixng-core/src/peddeco.rs` (+51/−2) o nome de decoração passa a ser procurado primeiro como registro e, se for conjunto, vira o corpo de registros que ele nomeia, cada um na parte da própria categoria; os registros de cabeça do conjunto ficam de fora (a maioria repete uma camada de corpo posta em espaço de corpo, cujo lugar na malha da cabeça não é conhecido). `pedlook.rs` (+4/−3) acompanha a nova forma de retorno. `CATALOGUE_CACHE_VERSION` sobe para 6, invalidando o cache anterior. O corpo do commit é explícito sobre o efeito: nome de conjunto "encontrava nenhum registro e era descartado, então um personagem cuja tatuagem de tronco vem por conjunto não recebia nenhuma" |

## Pull requests abertos (0)

Nenhum. `gh pr list --state open` devolve lista vazia.

## Pull requests mesclados (1)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #555 | Decorations: resolve decoration set names to their records | `cff3f31b` | O commit acima é o **squash** do PR (a mensagem termina em "(#555)" e o histórico tem um pai só, então 1 merge = 1 commit de conteúdo). O diff contra `67fc2742`, o cursor anterior, é exatamente o de `peddeco.rs` + `pedlook.rs`; os 55 adicionados são a resolução do conjunto e a construção dos registros de corpo |

## Cruzamento com o catálogo

- **Corrige D-xx?** Por hash, não: `cff3f31b` não intersecta os 270 tokens de `catalogo/defeitos.csv`.
- **Abre defeito novo?** Candidato a linha, sem hash de defeito anterior: "nome de decoração que é
  conjunto (struct `0x91501997`, campos `Name`/`Values`) não resolvia para registro e era descartado, e
  a tatuagem de tronco aplicada por conjunto sumia". Fica próximo de `SixNG D-04` (modelo antigo
  exibido sob nome novo) e `D-12` (carroceria do veículo aplicada a interior) apenas como classe
  "registro errado aplicado à peça errada"; não é nenhum dos dois.
- **Limite declarado no próprio diff:** registros de **cabeça** do conjunto são descartados de
  propósito, com a justificativa de que repetem uma camada de corpo em espaço de corpo. Isso significa
  que um conjunto cujo único conteúdo útil seja de cabeça continua sem efeito — o commit escolhe
  omissão em vez de aplicar no lugar errado. **NAO VERIFICADO:** se existe conjunto nessa situação no
  corpus do título; o diff não traz contagem.
- **Cache:** `CATALOGUE_CACHE_VERSION` 5 → 6 no mesmo commit. Ao contrário do caso registrado em
  12:00 (mudança de UV de tronco sem bump de `BAKE_RULE`), aqui o bump veio junto.

## Confiança

Verificado pelo payload de `repos/holdmysocks/SixNG/commits/cff3f31b`: mensagem completa, dois
arquivos com contagem de linha, `+55/−5`, e o `compare` `67fc2742...cff3f31b` com `ahead_by = 1` — não
houve outro commit na janela. Verificado também que o PR `#555` está mesclado (`merged_at`
`2026-11-28T17:41:36Z`) e que a lista de abertos está vazia. **NAO VERIFICADO:** que o teste do
repositório passe; que o conjunto de decoração realmente apareça no corpus do título (o corpo do commit
afirma o sintoma, mas não cita um recurso concreto com nome); e se os registros de cabeça descartados
deixam algum personagem sem decoração de cabeça — o commit diz que a maioria repete camada de corpo, não
que todos. Head no fim da coleta: `cff3f31b`.
