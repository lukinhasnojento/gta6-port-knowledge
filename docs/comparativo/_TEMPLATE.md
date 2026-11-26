# Template — Acertos e defeitos de <REPO>

**Preencha exatamente esta estrutura, sem inventar seções.** Cada afirmação precisa de hash de
commit. Onde não houver evidência, escreva `NÃO VERIFICADO`.

---

## Ficha

| Campo | Valor |
|---|---|
| Repositório | `<owner/repo>` |
| Commits | `<n>` |
| Licença | `<SPDX>` |
| Papel no ecossistema | uma frase |
| Estado hoje | uma frase |

## Placar

Cinco dimensões, nota de 0 a 5, com a justificativa em uma linha cada.

| Dimensão | Nota | Justificativa |
|---|---|---|
| Engenharia | /5 | qualidade técnica das correções |
| Verificabilidade | /5 | existência de testes, provas, reprodutibilidade |
| Documentação | /5 | honestidade e precisão do README e docs |
| Sustentabilidade | /5 | dívida, defaults perigosos, manutenção |
| Reaproveitamento | /5 | quanto do código serve ao repositório unificado |

**Média:** `x.x/5`

## Acertos

O que o projeto fez **certo**, com evidência. Ordene por valor.

| # | Acerto | Commit | Por que importa | Reusável? |
|---|---|---|---|---|
| A-01 | | `<hash>` | | sim / não / com ressalva |

## Defeitos

O que o projeto fez **errado**. Ordene por gravidade.

| # | Defeito | Gravidade | Commit | Estado |
|---|---|---|---|---|
| F-01 | | CRÍTICO / ALTO / MÉDIO / BAIXO | `<hash>` | aberto / corrigido / mitigado |

Escala de gravidade:
- **CRÍTICO** — quebra o build, perde dados do usuário, ou impede o uso por qualquer pessoa.
- **ALTO** — corrupção de resultado, crash frequente, regressão de correção anterior.
- **MÉDIO** — comportamento errado em caso específico, dívida que trava evolução.
- **BAIXO** — higiene, nomenclatura, inconsistência documental sem efeito funcional.

## Erros de processo

Fatos sobre **como** o projeto trabalha, não sobre o código. Exemplos: mensagem de commit que não
corresponde ao diff, histórico reescrito, README que contradiz o código, benchmark medido em
cenário não representativo, CI que não bloqueia.

| # | O que aconteceu | Evidência |
|---|---|---|

## O que copiar

Lista curta e concreta: arquivos, commits ou padrões que devem ir para o repositório unificado.

## O que não copiar

Lista curta: o que deve ser evitado ou reescrito, e por quê.

## Veredito

Um parágrafo. O que este projeto é, em uma avaliação honesta, sem elogio nem demolição.
