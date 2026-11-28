# Falhas e desvios — 2026-11-28 18:00

## Repositórios inacessíveis (2)

`Belobug/VIctory-Belobug-AMD` e `avatarmanz/VIctory-7900XTX` devolvem **HTTP 404** em
`repos/<owner>/<repo>` pela **décima segunda hora consecutiva** (desde 2026-11-27T21:05Z). Também
devolvem 404 em `users/<owner>`, o que indica conta removida ou renomeada, não apenas repositório
apagado. Consequências nesta execução:

- `default_branch` não pôde ser conferido em nenhum dos dois (o estado grava `a-publier` para `Belobug` e
  `main` para `avatarmanz`; ambos os valores ficam **NAO VERIFICADOS**).
- Nenhum `compare`, nenhuma lista de PRs, nenhuma linha de triagem com medição: as duas linhas saem como
  `NAO VERIFICADO` em `_triagem.md`.
- Cursores parados (`Belobug` em `4eb6cfdf`/`last_run` `2026-11-27T16:52:39Z`; `avatarmanz` em
  `014d52de`/`last_run` `2026-11-26T08:17:50Z`). Nada foi removido de `repos`.
- As duas entradas continuam no `backlog` com a marca `unreachable`.

Se a conta reaparecer, o próximo ciclo reprocessa do cursor gravado — não há baseline a fazer, porque
`last_commit` não é nulo em nenhum dos dois.

## Desvios de procedimento

- **A tabela de pistas continua ausente.** `docs/automacao/ANALISE_HORARIA.md` descreve o critério 3 como
  "PR aberto que toca arquivo de área listada na tabela de pistas", mas essa tabela não existe no
  documento. As áreas usadas nesta hora vieram das seções D de `catalogo/defeitos.csv` e do corpo dos
  PRs, e todo cruzamento por área está marcado como **hipótese** nos documentos.
- **Ordem do `compare` instável no `xcl`.** `2dad32e6` e `abf7e177`, documentados às 12:00, voltaram a
  aparecer dentro da faixa do `compare` a partir de `4701a874` e foram pulados **por hash**. O novo
  cursor `f2c0d36f` provavelmente re-inclui alguns dos 40 documentados nesta hora; a regra de pular hash
  documentado continua obrigatória.
- **Merge fora da branch default no `Xbox2Windows`.** O `#3139` foi mesclado em
  `codex/native-graphics-integration` (`merged_at` `2026-11-28T17:58:35Z`, merge `7291aef2`), não na
  `main`. O `compare` `7291aef2...main` devolve `diverged` (16 à frente, 1 atrás), então o merge não
  aparece na faixa de commits. O conteúdo foi documentado pelo diff do PR e o merge consta da tabela de
  mesclados; o cursor de commits avança para `f2972840`, que **não** contém `#3139`.
- **PR mesclado entre o snapshot e o fim da coleta no `Xbox2Windows`.** O `compare` pegou
  `aec2e084...6685de7c` (9 commits); `24b895d7` e `f2972840` chegaram às 18:08Z, depois do snapshot. Os
  dois foram lidos por `compare/6685de7c...f2972840` e o merge `#3375` foi documentado pelo diff do PR,
  para o próximo ciclo não reprocessar.
- **Snapshot de mesclados no `Marvisak`.** 50 PRs fechados lidos; nenhum com `merged_at` posterior a
  `2026-11-27T13:20:45Z`. Como o repositório tem mais de 3.700 PRs, a paginação parou quando o
  `updated_at` mais antigo da página caiu abaixo de `last_run` — o que é seguro para merges, porque
  mesclar atualiza `updated_at`, mas **NAO VERIFICADO** para PRs mesclados e depois comentados.
- **Nenhum subagente disponível** neste ambiente; os quatro deep dives foram processados em sequência.

## Rate limit

Nenhum 403. Saldo no início da coleta: 4.984 de 5.000. Saldo ao fim da coleta (18:57Z): 4.835.
