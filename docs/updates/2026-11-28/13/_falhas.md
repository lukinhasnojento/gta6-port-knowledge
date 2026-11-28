# Falhas e desvios — 2026-11-28 13:00

## Repositórios inacessíveis (HTTP 404, sétima hora)

`Belobug/VIctory-Belobug-AMD` e `avatarmanz/VIctory-7900XTX` continuam devolvendo HTTP 404 em
`repos/<owner>/<repo>` (e em `users/<owner>`). `default_branch` não reconferida, cursores de commit,
de PR aberto e de merge parados, nenhuma linha documentada, nenhum dos dois removido de `repos`.
Terceira instrução do procedimento (releia `default_branch`) não pôde ser cumprida neles.

## A "tabela de pistas" do procedimento não existe (sexto dia)

O passo 2 manda marcar sinal quando um PR aberto "tocar arquivo das áreas da tabela de pistas do
procedimento". `docs/automacao/ANALISE_HORARIA.md` não tem essa tabela — foi verificado de novo nesta
execução, nas 251 linhas do arquivo. As áreas do critério 3 vieram das seções D do
`catalogo/defeitos.md` e do corpo de cada PR, como nas horas anteriores, e cada cruzamento está
marcado como área ou hipótese, nunca como prova.

## `compare` do `xcl` truncado em 250 de 338

`gh api repos/ivall/xcl/compare/3dbac13a...e429936c` devolve `ahead_by=338` mas `commits.length=250`.
Os 40 documentados são os 40 primeiros da página devolvida e a interseção com o documento das 12:00 é
zero, mas **88 commits da faixa não vêm na resposta**. Eles não foram documentados nem descartados em
silêncio: a lacuna está registrada na seção Confiança de `xcl.md` e o `ahead_by` continua a ser a
medida de atraso (338). Não há como cobri-los pelo mesmo `compare`; uma coleta por outro âncora seria
necessária e não foi tentada nesta execução.

## Falha de coleta isolada, refeita

O `fetch` do commit `4b01fde4` do `xcl` falhou na primeira passada (resposta vazia do `gh api`) e foi
refeito com sucesso; o documento descreve o diff real (+73/−4 em `vram.rs`). Nenhum outro `fetch` de
commit falhou entre os 128 pedidos.

## Sobreposição de cursor medida antes de escrever

Nos três repositórios com `commit_cursor`, o cursor gravado apontava para o 40º commit do ciclo
anterior, mas a ordem do `compare` não é a ordem em que aquele ciclo escreveu. Antes de documentar,
medi a interseção entre a página do `compare` e os hashes dos documentos das 12:00:

| repo | índice do primeiro commit não documentado | interseção com o doc das 12:00 |
|---|---|---|
| `ivall/xcl` | 0 | 0 de 40 |
| `brcly/TuxBox` | 19 | 19 de 40 (descartados) |
| `R0Y00/Xbox2Windows` | 5 | 5 de 40 (descartados) |

As janelas desta hora começam nesses índices, e por isso os três documentos têm 40 commits novos cada
sem repetir linha. O desvio — cursor gravado não corresponde ao primeiro não documentado — é o mesmo
que o estado já anotava em `backlog_note` e continua valendo para o próximo ciclo.

## Nenhum `diverged`, nenhum `last_commit` nulo

Os 12 repositórios acessíveis devolveram `ahead` (5) ou `identical` (7); nenhum `diverged`, nenhum
histórico reescrito. Nenhum `last_commit` nulo: não houve baseline nesta execução.

## Rate limit

4.677 no início, 3.192 no fim (≈1.485 chamadas, incluindo 128 leituras de commit, 19 de merge e ~130
de PR). Nenhum 403, nenhuma repetição em laço.

## Escrita

Somente em `G:\gta6-port-knowledge`. Os 14 monitorados foram apenas leitura de API. Os arquivos de
terceiros no diretório de trabalho (`docs/analises/victory-fork.md` modificado e os dois relatórios
não rastreados em `docs/relatorios/`) ficaram intactos e fora do `git add`.
