# Execução de 2026-11-26 21:00 (-03:00)

- **Início da coleta:** 2026-11-26T23:03Z (20:03 local). **Gravação do estado:** 2026-11-27T00:14Z.
- **Estado lido:** `baselined=true`, `runs=6`, `updated=2026-11-26T23:21:11Z`. Execução de delta, não de
  baseline.
- **Janela:** os cursores de `state/watch.json`, cada um no seu horário (de `04:15:00Z` a `22:45:09Z`),
  até a coleta. Não é janela de uma hora: repositórios que estavam no `backlog` tinham cursor de várias
  horas atrás.

## Repositórios com delta (10)

| Repositório | Commits na faixa | PRs abertos novos | PRs mesclados na janela | Situação |
|---|---|---|---|---|
| R0Y00/Xbox2Windows | 72 | 50 (lista limitada a 50) | 57 | documentado: 40 commits e 20 PRs abertos; 30 PRs abertos ficaram |
| avatarmanz/VIctory-7900XTX | 149 (85 não documentados) | 4 | 0 | documentado: 40 commits e 4 PRs abertos |
| brcly/TuxBox | 56 (34 não documentados) | 5 | 2 | documentado: 34 commits (faixa inteira) e 5 PRs abertos |
| holdmysocks/SixNG | 16 | 8 | 16 | documentado: 16 commits e 8 PRs abertos |
| Marvisak/VIctory | 0 | 50 | 0 | backlog |
| ivall/xcl | 0 | 50 | 0 | backlog |
| Belobug/VIctory-Belobug-AMD | 0 | 50 | 0 | backlog |
| DexNom/VIctory | 0 | 39 | 0 | backlog |
| fxceless/victory-fork | 0 | 1 (#1381) | 0 | backlog |
| omonkeyz/xbox-emulator | 0 | 1 (#395) | 0 | backlog |

**Sem delta (3):** `californiaAlmonds/OpenScarlett` (head `48a98fcd` = cursor; o único PR aberto é o
#60, já no cursor, e o merge de 26/11 tem `mergedAt` anterior a `last_run`), `uiliannn/VItoriaBr` (head
`1361e190` = cursor, sem PR acima de 126) e `walkthestars/GTA6-Asset-Info` (head `bad5bc21` = cursor, sem
PRs).

## Documentos escritos

Quatro, em `docs/updates/2026-11-26/21/`: `Xbox2Windows.md`, `VIctory-7900XTX.md`, `TuxBox.md`,
`SixNG.md`. `docs/updates/INDEX.md` ganhou uma linha e um parágrafo.

Tetos aplicados:
- **4 repositórios por execução.** Não há ferramenta de subagente neste ambiente; os quatro foram
  processados em sequência. Critério: primeiro os quatro que tinham commit pendente (o backlog acumula
  commit, PR aberto não muda), depois os que só têm PR aberto.
- **40 commits por repositório.** Cortou `Xbox2Windows` (72→40, `commit_cursor` `1983228b`, 32 restantes,
  `head_at_run` `6ee878d8`) e `VIctory-7900XTX` (85 não documentados→40, `commit_cursor` `f03bfa51`, 48
  restantes, `head_at_run` `126acb35`). `TuxBox` (34) e `SixNG` (16) couberam inteiros; o
  `commit_cursor` de `TuxBox` foi removido porque o cursor iguala o head.
- **20 PRs abertos por repositório** (limite herdado da execução anterior, não é regra do procedimento).
  Aplicado só a `Xbox2Windows` (50→20, os 20 de número mais alto). `VIctory-7900XTX` (4), `TuxBox` (5) e
  `SixNG` (8) couberam inteiros.
- **PRs mesclados:** 75 documentados (57 + 0 + 2 + 16), todos os que a API devolveu na janela de cada
  repositório. `last_run` avançou até o `mergedAt` do último merge documentado por repositório
  (`17:08:05Z` em `Xbox2Windows`, `20:15:58Z` em `TuxBox`, `20:59:44Z` em `SixNG`; `VIctory-7900XTX`
  ficou em `08:17:50Z`, que é o valor exato dos merges daquele lote).

## Divergências e correções de método

- **Cursor de `TuxBox` estava atrás do que a execução de 20:00 documentou.** A faixa
  `be9055c9..68e79b7e` tem 56 commits; 22 deles (`e4fdfe4c`, `4461c8fb`, `c6ce497b`, `be9055c9` e
  vizinhos) já constavam do documento de 20:00. Deduplicado contra `docs/updates/2026-11-26/*/TuxBox.md`
  e `2026-11-25/*/TuxBox.md` antes de escrever: 34 commits novos, nenhum repetido. O "44 restantes" de
  20:00 não se confirmou; eram 34.
- **Contagem de PR mesclado refeita.** A primeira passada comparava `datetime` do PowerShell; o
  `Z` do `mergedAt` foi lido como hora local (−03:00), o que trouxe 3 PRs de antes do cursor em
  `Xbox2Windows` (#2751, #2752, #2756) e dava 60 merges. Refiz com comparação de string ISO-8601 UTC:
  57 merges, todos com `mergeCommit.oid` dentro da faixa de commits.
- **`last_pr_number` de `Xbox2Windows` não avançou** (ficou em 2 861): os 50 PRs abertos novos foram
  documentados só nos 20 de número mais alto, e avançar o cursor para 2 925 perderia os números
  2 862–2 905. O repositório ficou no `backlog` por esses 30 PRs; os commits estão cobertos por
  `commit_cursor`.

## Falhas

- **Nenhuma falha de coleta.** As 13 consultas de commits, as 13 comparações `compare/<cursor>...<branch>`,
  as 13 listas de PRs abertos e as 13 buscas de mesclados responderam. Nenhum 403; `rate_limit` estava em
  4 835/5 000 ao fim.
- **Nenhum `_falhas.md`.** Todo repositório com delta não documentado entrou em `backlog`.
- **Nenhum hash, PR ou número inventado.** Todo fato nas quatro tabelas de commits sai do `patch` da API
  (`repos/.../commits/<sha>`); onde não deu para verificar, o documento diz "não verificado".

## Push

- Comando executado: `switch public` → `add docs/updates state` → `commit -m "docs(updates): 2026-11-26 21:00"`
  → `push origin public:main` → `switch main` → `merge --no-ff public` → `push private main`.
- Resultado: **sucesso nas duas branches.**
  - `public`: commit `be35454212d50cdad68aeec07fcb99bb6fd68306`; `git push origin public:main` avançou
    `85a67d4..be35454`.
  - `main`: merge `--no-ff` `3e5e977c2ecdea86b8c3ea70e2f8392db19674c9` ("merge updates 2026-11-26 21:00");
    `git push private main` avançou `8b572c1..3e5e977`.
- O `main` nunca foi mesclado na `public`; a `public` foi mesclada na `main`, na ordem do procedimento.
- Este parágrafo e os dois hashes foram gravados no commit de estado seguinte, sobre a `public`, e
  mesclados na `main` pelo mesmo caminho.
