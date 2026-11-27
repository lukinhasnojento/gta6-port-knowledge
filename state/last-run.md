# Execução de 2026-11-26 22:00 (-03:00)

- **Início da coleta:** 2026-11-27T01:03Z (22:03 local). **Gravação do estado:** 2026-11-27T01:11Z.
- **Estado lido:** `baselined=true`, `runs=7`, `updated=2026-11-27T00:14:04Z`. Execução de delta, não de
  baseline.
- **Janela:** os cursores de `state/watch.json`, cada um no seu horário (de `2026-11-26T04:15:00Z` a
  `22:45:09Z`), até a coleta. Não é janela de uma hora: os repositórios em `backlog` tinham cursor de
  várias horas atrás.

## Repositórios com delta (11)

| Repositório | Commits na faixa | PRs abertos novos | PRs mesclados na janela | Situação |
|---|---|---|---|---|
| R0Y00/Xbox2Windows | 32 (resto das 21:00) | 50 | 6 (todos em `cpp-migration`) | documentado: 32 commits (faixa inteira) e 20 PRs abertos; 30 PRs abertos ficaram |
| avatarmanz/VIctory-7900XTX | 58 | 6 | 0 | documentado: 40 commits e 6 PRs abertos; 18 commits ficaram |
| brcly/TuxBox | 31 | 2 | 13 | documentado: faixa e PRs inteiros |
| holdmysocks/SixNG | 6 | 5 | 6 | documentado: faixa e PRs inteiros |
| Marvisak/VIctory | 0 | 50 | 0 | backlog |
| ivall/xcl | 0 | 50 | 0 | backlog |
| Belobug/VIctory-Belobug-AMD | 0 | 50 | 0 | backlog |
| DexNom/VIctory | 0 | 50 | 0 | backlog |
| uiliannn/VItoriaBr | 0 | 1 (#128) | 0 | backlog |
| fxceless/victory-fork | 0 | 1 (#1381) | 0 | backlog |
| omonkeyz/xbox-emulator | 0 | 1 (#395) | 0 | backlog |

**Sem delta (2):** `californiaAlmonds/OpenScarlett` (head `48a98fcd` = cursor; o único PR aberto é o #60,
já coberto pelo cursor de abertos, e nenhum merge depois de `07:52:24Z`) e
`walkthestars/GTA6-Asset-Info` (head `bad5bc21` = cursor; sem PRs).

## Documentos escritos

Quatro, em `docs/updates/2026-11-26/22/`: `Xbox2Windows.md`, `VIctory-7900XTX.md`, `TuxBox.md`,
`SixNG.md`. `docs/updates/INDEX.md` ganhou uma linha e um parágrafo.

Tetos aplicados:
- **4 repositórios por execução.** Não há ferramenta de subagente neste ambiente; processados em
  sequência. Critério: os quatro com commit pendente, porque commit acumula e PR aberto não muda o diff.
- **40 commits por repositório.** Só cortou `VIctory-7900XTX` (58→40, `commit_cursor` `a9400a6d`, 18
  restantes, `head_at_run` `126acb35`). `Xbox2Windows` (32), `TuxBox` (31) e `SixNG` (6) couberam
  inteiros; o `commit_cursor`/`head_at_run` de `Xbox2Windows` foi removido porque o cursor iguala o head.
- **20 PRs abertos por repositório** (limite herdado das execuções anteriores, não é regra do
  procedimento). Aplicado só a `Xbox2Windows` (50→20, os 20 de menor número, 2 867–2 887, para que o
  cursor de abertos avance sem deixar buraco). `VIctory-7900XTX` (6), `TuxBox` (2) e `SixNG` (5)
  couberam inteiros.
- **PRs mesclados:** 19 documentados em `main` (13 `TuxBox` + 6 `SixNG`). Os 6 do `Xbox2Windows` estão no
  documento, mas não contam como mudança de `main` (ver divergência abaixo). `last_run` avançou até o
  `mergedAt` do último merge examinado por repositório: `21:59:23Z` em `Xbox2Windows`, `21:27:17Z` em
  `TuxBox`, `22:01:14Z` em `SixNG`; `VIctory-7900XTX` ficou em `08:17:50Z` (sem merges).

## Divergências e correções de método

- **PRs mesclados do `Xbox2Windows` não vão para `main`.** Os seis (`#2914`, `#2918`, `#2917`, `#2931`,
  `#2928`, `#2932`) têm `base.ref = cpp-migration` e `merge_commit_sha` fora da faixa de `main`
  (`6ee878d8` continua sendo o head). Ficaram registrados no documento como merges de branch de trabalho,
  não como mudança da branch monitorada. Registro do par `#2918` (`ee76dfa8`, 21:23:40Z) → `#2931`
  (`7f951fd3`, 21:47:04Z): merge revertido em 23 min, mesma classe do **D-29**.
- **`TuxBox`: 14 merge commits para 13 PRs.** O PR #268 foi mesclado duas vezes — `6466bcf9` (fontes e
  teste) e `e76ff6a2` (só `tests/hlsl/mesh.rs`, +63/−14, levando a fixture `ONE_COUNT` que a resolução de
  conflito do primeiro merge perdeu). Todos os 13 `mergedAt` são idênticos (`21:27:17Z`), o que indica
  merge em lote. Registrado na Confiança do documento.
- **Cursor de mesclados usa `mergedAt`, não número de PR**, como manda o procedimento (seção "Por que PR
  mesclado não usa número"). O pedido de execução sugeria filtrar por `numero > last_pr_number`; seguir
  isso teria perdido os 13 merges do `TuxBox` (números 252–270 contra cursor de abertos 272) e os 6 do
  `SixNG` (#403–#412 abaixo do cursor 410). `last_pr_number` foi usado só para PRs **abertos**.
- **`last_pr_number` de `Xbox2Windows` avançou para 2 887**, o maior dos 20 documentados em ordem
  crescente. Os 30 restantes (2 888–2 930) serão re-detectados na próxima execução; o repositório voltou
  ao `backlog` por eles.

## Falhas

- **Nenhuma falha de coleta.** As 13 consultas de head, as 13 listas de PRs abertos, as 13 de mesclados,
  4 `compare/<cursor>...<branch>` e 109 `commits/<sha>` responderam. Nenhum 403.
- **Nenhum `_falhas.md`.** Todo repositório com delta não documentado entrou em `backlog`.
- **Nenhum hash, PR ou número inventado.** Todo fato das tabelas de commits sai do `patch` da API
  (`repos/.../commits/<sha>`); o que não deu para verificar está marcado como não verificado.

## Push

- Comando executado: `switch public` → `add docs/updates state` → `commit -m "docs(updates): 2026-11-26 22:00"`
  → `push origin public:main` → `switch main` → `merge --no-ff public` → `push private main`.
- Resultado: **sucesso nas duas branches.**
  - `public`: commit `3dd1ee537268698f5a7c13a674bcdd575dfa00f0`; `git push origin public:main` avançou
    `77386e9..3dd1ee5`.
  - `main`: merge `--no-ff` `e11b843918e703858580c35c8d5a8242e1acff63` ("merge updates 2026-11-26 22:00");
    `git push private main` avançou `fb7c1b1..e11b843`.
- O `main` nunca foi mesclado na `public`; a `public` foi mesclada na `main`, na ordem do procedimento.
- Este parágrafo e os hashes foram gravados no commit de estado seguinte, sobre a `public`, e mesclados na
  `main` pelo mesmo caminho.
- `rate_limit` ao fim: 4 832/5 000; nenhum 403 em nenhuma chamada.
