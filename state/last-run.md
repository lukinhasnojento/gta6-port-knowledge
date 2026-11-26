# Execução de 2026-11-26 20:00 (-03:00)

- **Início da coleta:** 2026-11-26T23:06Z (20:06 local).
- **Fim da coleta:** 2026-11-26T23:21Z.
- **Janela:** cursores de `state/watch.json`, que iam de `2026-11-26T04:15:00Z` (01:15 local) a
  `2026-11-26T22:45:09Z` (19:45 local), até a coleta. Para os quatro repositórios documentados a janela era de
  ~19 h, não de uma hora: eles estavam no `backlog` desde a execução de 19:00 e nenhum cursor deles avançou.

## Repositórios com delta (11)

| Repositório | Commits na janela | PRs abertos novos | PRs mesclados na janela | Situação |
|---|---|---|---|---|
| avatarmanz/VIctory-7900XTX | 147 | 60 (lista limitada a 60) | 32 | documentado: 40 commits e 20 PRs abertos |
| brcly/TuxBox | 84 | 29 | 24 | documentado: 40 commits e 20 PRs abertos |
| fxceless/victory-fork | 22 | 60 (lista limitada a 60) | 6 | documentado: 22 commits e 20 PRs abertos |
| uiliannn/VItoriaBr | 2 | 12 | 0 | documentado: 2 commits e 12 PRs abertos |
| R0Y00/Xbox2Windows | 72 | 45 | 66 | backlog |
| holdmysocks/SixNG | 9 | 5 | 9 | backlog |
| ivall/xcl | 0 | 60 (lista limitada a 60) | 0 | backlog (só PRs abertos) |
| Marvisak/VIctory | 0 | 60 (lista limitada a 60) | 0 | backlog (só PRs abertos) |
| Belobug/VIctory-Belobug-AMD | 0 | 60 (lista limitada a 60) | 0 | backlog (só PRs abertos) |
| DexNom/VIctory | 0 | 34 | 0 | backlog (só PRs abertos) |
| omonkeyz/xbox-emulator | 0 | 1 (#395) | 0 | backlog (só PRs abertos) |

**Sem delta (2):** `californiaAlmonds/OpenScarlett` (head `48a98fcd` = cursor; 1 PR com `mergedAt` posterior ao
`last_run` mas anterior ao cursor, nada a documentar) e `walkthestars/GTA6-Asset-Info` (head `bad5bc21` =
cursor, sem PRs).

## Documentos escritos

Quatro, em `docs/updates/2026-11-26/20/`: `VIctory-7900XTX.md`, `TuxBox.md`, `victory-fork.md`,
`VItoriaBr.md`. `docs/updates/INDEX.md` ganhou uma linha.

Tetos aplicados:
- **4 repositórios por execução** (limite do procedimento). Não há ferramenta de subagente neste ambiente;
  os quatro foram processados em sequência. Os quatro saíram do `backlog`, na ordem em que já estavam
  registrados ali.
- **40 commits por repositório.** Cortou `VIctory-7900XTX` (147→40, `commit_cursor` `306722bc`, 107 restantes,
  `head_at_run` `2de99801`) e `TuxBox` (84→40, `commit_cursor` `be9055c9`, 44 restantes, `head_at_run`
  `b3184d35`).
- **20 PRs abertos por repositório** (limite desta execução, herdado da anterior; não é regra do
  procedimento). Aplicado a `VIctory-7900XTX` (60→20), `TuxBox` (29→20) e `victory-fork` (60→20);
  `VItoriaBr` tinha 12 e coube inteiro.
- **PRs mesclados:** 62 documentados (32 + 24 + 6 + 0). `last_run` avançou até o `mergedAt` do último merge
  documentado de cada repositório, não até o horário da execução.

## Falhas

- **Nenhuma falha de coleta.** As 13 consultas de commits (`gh api .../commits`), as 13 comparações
  `compare/<cursor>...<branch>`, as 13 listas de PRs abertos e as 13 buscas de mesclados responderam. Nenhum
  403; `rate_limit` estava em 4 822/5 000 no começo.
- **Nenhum `_falhas.md`.** Todo repositório com delta que não foi documentado entrou em `backlog`, não em
  falha.
- **Desvio de método registrado:** a busca de PRs mesclados com `--search "merged:>=<timestamp completo>"`
  devolveu 0 em oito dos treze repositórios na primeira passada (formato de data com hora não é aceito de
  forma confiável pelo índice de busca). Refiz com `merged:>=AAAA-MM-DD` e filtrei `mergedAt` localmente;
  aí os números fecharam: 32 em `VIctory-7900XTX`, 24 em `TuxBox`, 6 em `victory-fork`, 66 em
  `Xbox2Windows`, 9 em `SixNG`.
- **Desvio de leitura registrado:** o clone local foi `--filter=blob:none`, o que dá commits e árvores de
  graça e busca blobs sob demanda. Para 40 commits por repositório isso foi rápido, mas um `git show` em
  commit antigo pode demorar; a primeira tentativa de rodar `--shortstat` em 32 merges foi movida para
  background e cancelada, e a tabela de merges foi feita por `git diff --name-status` (sem blobs).

## Achados que pedem revisão do catálogo

- **D-09 do `AvatarManz`** (`Device loss a cada ~16 min: textura mostrada antes de o upload rodar`) tem
  `corrigido` vazio, e a faixa traz `4cd2d133` + `91018a5d` + `cf9ba084`, que implementam
  `VICTORY_SHOW_NEW_LATER` **ligado por padrão** exatamente para esse mecanismo, mais `c278f0d5` (textura
  ≤ 64 KiB em múltiplos de 64 KiB). Não gravei hash: falta execução sem device loss.
- **D-19 do `AvatarManz`** fica meio corrigido: `5cfe32df` remove `VICTORY_SKIP_PIPELINES='none'` do
  `play.ps1` e `d792ebd4` cria checagem no `xtask` de que todo `VICTORY_*` do Markdown é lido pelo código.
- **D-04 do `TuxBox`** estava corrigido com teto de 32 GiB; `c6ce497b` mostra, com medição, que o driver
  para de indexar com ~5 GB e baixa o teto para 4 GiB.
- **Candidato novo no `TuxBox`:** teto fixo de 90 s de compilação `-O0` recusava os kernels de ray tracing
  (~2,5 MB de HLSL, ~77 s cada) em todo boot — corrigido no mesmo commit (`7e0ad65c`).
- **Candidato novo no `victory-fork`:** CBV de zero bytes virou 256 bytes da cópia do host por causa de
  `2ba6e1f`; corrigido em `619b2dc5`. E `52ac4ec0` (HEAD) é o squash do PR #1324 com **diff vazio**: o
  conteúdo já estava em `e4245f3c`.

## Publicação

- `git switch public`, `git add docs/updates state`, commit `docs(updates): 2026-11-26 20:00` = **713cce9**.
- `git push origin public:main` → **ok** (`613147f..713cce9 public -> main`).
- `git switch main`, `git merge --no-ff public -m "merge updates 2026-11-26 20:00"` = **b2a0dcb**.
- `git push private main` → **ok** (`e9862a2..b2a0dcb main -> main`).
- Nenhum merge de `main` em `public`.
- A diferença entre as duas branches continua sendo só `docs/plano/PROMPT_WINDOWS_PHASE.md` (na `main`).
- Este arquivo é escrito **depois** da publicação e vai num commit `chore(state)` na `public`, mesclado na
  `main` e empurrado para os dois remotes — mesmo padrão da execução de 19:00.

## Rodapé: o segundo par de commits

- `chore(state)` de 2026-11-26 20:00 = **7e91b83** — `git push origin public:main` ok
  (`713cce9..7e91b83 public -> main`); inclui este arquivo e a correção da tabela de merges do
  `VIctory-7900XTX.md` feita depois da primeira publicação (`git diff` do merge confirmou +202/−22 em #147,
  +199/−21 em #63 etc.).
- `git merge --no-ff public -m "merge updates 2026-11-26 20:00 (last-run)"` = **504a921**;
  `git push private main` ok (`b2a0dcb..504a921 main -> main`).
- Estado final: `public` = `origin/main` (0 commits de diferença); árvore limpa.

