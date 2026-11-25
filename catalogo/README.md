# Catálogo

Índice consolidado dos defeitos identificados nas doze análises.

## Arquivos

| Arquivo | Formato | Uso |
|---|---|---|
| `defeitos.csv` | CSV, UTF-8 | fila de trabalho, importação em planilha, filtro por confiança |
| `defeitos.md` | Markdown | leitura, agrupado por repositório |

## Colunas do CSV

| Coluna | Conteúdo |
|---|---|
| `repo` | repositório de origem |
| `id` | identificador dentro da análise (`D-01`, `D-02`, …) |
| `titulo` | descrição curta, como escrita na análise |
| `confianca` | `CONFIRMADA`, `PROVAVEL` ou vazio |
| `introduzido` | hash do commit que introduziu o defeito, quando identificado |
| `corrigido` | hash do commit que corrigiu, quando identificado |

## Números

| Métrica | Valor |
|---|---|
| Entradas | 281 |
| Com hash de introdução | 139 |
| Com hash de correção | 192 |
| `CONFIRMADA` | 91 |
| `PROVAVEL` | 5 |
| Sem confiança extraída | 185 |

## Como ler a coluna de confiança

Vazio **não** significa "não classificado" nem "pouco confiável". Significa que a extração
automática do CSV não localizou o campo no texto — a classificação existe na análise
correspondente, em `docs/analises/<repo>.md`, seção D.

O vocabulário é o mesmo em todos os documentos:

- **CONFIRMADA** — sustentada por evidência direta: diff, execução de teste, ou aritmética reprodutível.
- **PROVÁVEL** — sustentada por evidência parcial.
- **HIPÓTESE** — explicação possível, não confirmada.
- **NÃO VERIFICADA** — não foi possível determinar.

## Como auditar uma entrada

Toda linha tem repositório e ID. Para conferir:

```bash
# o commit de introdução existe e faz o que a análise diz?
git show <hash>

# o commit de correção existe?
git show <hash>

# o problema voltou depois da correção?
git log <hash>..HEAD -S"<trecho alterado>" --format="%h %ad %s" --date=short
```

O índice completo de commits de cada repositório está em `docs/indices/<repo>_log_compact.txt`.
Os dossiês factuais em `docs/fatos/` foram gerados por `git` puro, sem modelo de linguagem, e
servem como referência independente para totais, autores e datas.

## Priorização sugerida

O catálogo não ordena por gravidade — isso depende do seu objetivo. Três filtros úteis:

**Para não repetir erro histórico:** cruze `introduzido` e `corrigido` com as seções `E` e `H` das
análises, que descrevem cadeias de causa e efeito e regressões.

**Para aproveitar código:** seção `I` (conhecimento reutilizável) de cada análise, mais
`docs/fatos/FORK_DIVERGENCIA.md` para saber o que é exclusivo de cada fork.

**Para evitar bloqueio jurídico:** leia `NOTICE.md` antes de tocar em qualquer arquivo do
`AvatarManz`.
