# Execução de 2026-11-26 19:00 (-03:00)

- **Início da coleta:** 2026-11-26T22:03Z (19:03 local) — retomada após reinício do servidor.
- **Fim da coleta:** 2026-11-26T22:45Z.
- **Janela:** cursores de `state/watch.json` em `2026-11-26T04:15:00Z` (01:15 local) até a coleta. São ~18 h, não
  uma hora: a tentativa do slot 02:00 foi interrompida pelo reinício do servidor antes de gravar qualquer arquivo,
  e o estado em disco não avançou.

## Repositórios com delta (12)

| Repositório | Commits na janela | PRs abertos novos | PRs mesclados na janela | Situação |
|---|---|---|---|---|
| R0Y00/Xbox2Windows | 112 | 63 | 110 | documentado, 40 commits e 20 PRs abertos |
| ivall/xcl | 13 | 185 (lista de abertos com 500 entradas) | 0 | documentado, 13 commits e 20 PRs abertos |
| californiaAlmonds/OpenScarlett | 27 | 0 | 1 | documentado, 27 commits |
| holdmysocks/SixNG | 48 | 9 | 47 | documentado, 40 commits e 9 PRs abertos |
| avatarmanz/VIctory-7900XTX | 147 | 100 | 32 | backlog |
| brcly/TuxBox | 83 | 26 | 0 | backlog |
| fxceless/victory-fork | 22 | 61 | 0 | backlog |
| uiliannn/VItoriaBr | 2 | 12 | 0 | backlog |
| Marvisak/VIctory | 0 | 100 | 0 | backlog (só PRs abertos) |
| Belobug/VIctory-Belobug-AMD | 0 | 53 | 0 | backlog (só PRs abertos) |
| DexNom/VIctory | 0 | 28 | 0 | backlog (só PRs abertos) |
| omonkeyz/xbox-emulator | 0 | 1 | 0 | backlog |

`walkthestars/GTA6-Asset-Info` não teve delta (0 commits, 0 PRs abertos novos, 0 mesclados).

## Documentos escritos

4 documentos em `docs/updates/2026-11-26/19/`: `Xbox2Windows.md`, `xcl.md`, `OpenScarlett.md`, `SixNG.md`.
Mais `docs/updates/INDEX.md` (uma linha e um parágrafo) e `state/watch.json`.

- **Commits documentados:** 120 de 200 na janela (40/112 em `Xbox2Windows`, 13/13 em `xcl`, 27/27 em
  `OpenScarlett`, 40/48 em `SixNG`). Os dois cortes de teto gravaram `commit_cursor` e `head_at_run`.
- **PRs abertos documentados:** 49 de 257 detectados (20/63 em `Xbox2Windows`, 20/185 em `xcl`, 9/9 em
  `SixNG`). Os demais ficam para a próxima execução; `last_pr_number` avança só até o último documentado.
- **PRs mesclados documentados:** 78 de 158 (38 em `Xbox2Windows`, 0 em `xcl`, 1 em `OpenScarlett`, 39 em
  `SixNG`). O cursor `last_run` de cada repositório avançou até o `mergedAt` do último merge documentado, não até
  o horário da execução.

## Limites aplicados

- **4 repositórios por execução.** Não havia ferramenta de subagente no ambiente; os quatro foram processados em
  sequência. Os oito restantes com delta entraram em `backlog`, na ordem em que aparecem em `state/watch.json`.
- **40 commits por repositório.** Cortou `Xbox2Windows` (112) e `SixNG` (48).
- **20 PRs abertos por repositório (limite desta execução, não do procedimento).** Aplicado a `Xbox2Windows`
  (63) e `xcl` (185), porque a fila de PRs cresceu uma ordem de grandeza acima dos commits; sem esse teto a
  tabela teria mais de 250 linhas de PR por hora e nenhuma verificação real.

## Falhas

- Nenhuma falha de coleta: as 13 consultas de commits, as listas de abertos e as buscas de mesclados
  responderam. Nenhum 403 de rate limit.
- A tentativa anterior (slot 02:00) morreu por reinício do servidor sem gravar arquivos; nada ficou pela metade
  no repositório.
- **Não verificado:** os candidatos a defeito novo apontados nos quatro documentos (marcados `NAO VERIFICADO`
  em cada um); o comportamento em hardware real dos commits que dependem de prova de fluxo; o efeito do novo
  poller de GameInput do `xcl` em títulos reais.

## Publicação

- `git switch public`, `git add docs/updates state`, commit `docs(updates): 2026-11-26 19:00` = **99907b4**.
- `git push origin public:main` → **ok** (`1625ff2..99907b4 public -> main`).
- `git switch main`, `git merge --no-ff public -m "merge updates 2026-11-26 19:00"` = **36a9798**.
- `git push private main` → **ok** (`ee82d73..36a9798 main -> main`).
- Nenhum merge de `main` em `public`.
- A diferença entre as duas branches continua sendo só `docs/plano/PROMPT_WINDOWS_PHASE.md` (presente na `main`,
  ausente na `public`), como antes desta execução.

