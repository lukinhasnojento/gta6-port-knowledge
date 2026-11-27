# Execução de 2026-11-27 00:00

## Quando

Início 2026-11-27T00:03:10-03:00 (2026-11-27T03:03:10Z). Hora local do executor: UTC-3. Os carimbos
`updated` e `last_run` do estado são UTC; os diretórios de documento usam a hora local, então esta
execução grava em `docs/updates/2026-11-27/00/`.

## Repositórios com delta

Dez dos treze tinham delta. Quatro foram documentados; seis ficaram no `backlog` pelo limite de 4
repositórios por execução.

| Repositório | Delta detectado | Situação |
|---|---|---|
| `R0Y00/Xbox2Windows` | 2 commits, 45 PRs abertos, 18 mesclados | documentado |
| `holdmysocks/SixNG` | 26 commits, 10 PRs abertos, 26 mesclados | documentado |
| `brcly/TuxBox` | 5 commits, 0 abertos, 2 mesclados | documentado |
| `Belobug/VIctory-Belobug-AMD` | 0 commits, 50 abertos na janela, 20 mesclados | documentado |
| `Marvisak/VIctory` | 50 PRs abertos | backlog |
| `ivall/xcl` | 50 PRs abertos | backlog |
| `DexNom/VIctory` | 24 PRs abertos | backlog |
| `fxceless/victory-fork` | 10 PRs abertos | backlog |
| `avatarmanz/VIctory-7900XTX` | 3 PRs abertos | backlog |
| `omonkeyz/xbox-emulator` | 1 PR aberto (`#395`) | backlog |

Sem delta: `uiliannn/VItoriaBr`, `californiaAlmonds/OpenScarlett`, `walkthestars/GTA6-Asset-Info`.

## Documentos escritos

Quatro, em `docs/updates/2026-11-27/00/`: `Xbox2Windows.md`, `SixNG.md`, `TuxBox.md`,
`VIctory-Belobug-AMD.md`. Cobertura: 33 commits (26 `SixNG`, 5 `TuxBox`, 2 `Xbox2Windows`), 105 PRs
abertos (45 + 10 + 50) e 66 mesclados (18 + 26 + 2 + 20). Nenhum repositório passou do teto de 40
commits.

## Falhas

Nenhuma falha de coleta. `gh api rate_limit` no início: 5.000 de 5.000 no core. Nenhum 403. Nenhum
`_falhas.md` foi escrito porque não houve falha por repositório.

Ferramenta de subagente indisponível nesta sessão: os quatro repositórios foram processados em sequência
pelo próprio executor, não em paralelo.

## Desvios registrados

1. **`Belobug/VIctory-Belobug-AMD`: a branch default da API é `a-publier`, não `main`.** As 20 fusões da
   janela foram todas para `a-publier`. A `main` está parada em `4f62c2a5` desde
   `2026-11-25T20:56:22Z`; `compare/main...a-publier` devolve `diverged`, 638 commits à frente e 550
   atrás, base de merge `8e79c80e`. O estado manteve `default_branch: "main"` e **não** retargetou o
   cursor de commits, porque isso muda a semântica de `last_commit` e precisa de decisão explícita.
   Enquanto isso durar, os commits de `a-publier` são invisíveis para a rotina.
2. **`Belobug/VIctory-Belobug-AMD` tem 239 PRs abertos** (`#3` a `#456`). A janela de 50 traz só os mais
   novos. `last_pr_number: 456` significa "documentei até o 456", **não** "vi todos os abertos": os 189
   mais antigos nunca foram documentados e o cursor por número não consegue expressá-los.
3. **`R0Y00/Xbox2Windows`: 16 dos 18 merges foram para `cpp-migration`**, base não-default. Por isso 18
   PRs mesclados contra 2 commits na `main` (divergência exigida pelo procedimento, registrada na seção
   Confiança do documento). `last_commit` segue `20a497c2`, topo da `main`.
4. **`holdmysocks/SixNG`**: 26 commits e 26 merges casam 1:1, mas vários PRs estavam empilhados e o
   diff declarado no corpo é maior que o commit squash. O que entrou de fato está medido nos dois lados
   na tabela de merges.

## Push

Executado em seguida ao commit de documentos. Resultado registrado no commit seguinte
(`chore(state): registra hashes do push de 2026-11-27 00:00`).
