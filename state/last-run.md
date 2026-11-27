# Execução de 2026-11-27 01:00

## Quando

Início 2026-11-27T01:03:08-03:00 (2026-11-27T04:03:08Z). Hora local do executor: UTC-3. Os carimbos
`updated` e `last_run` do estado são UTC; os diretórios de documento usam a hora local, então esta
execução grava em `docs/updates/2026-11-27/01/`.

## Repositórios com delta

Dez dos treze tinham delta. Quatro foram documentados; sete ficaram no `backlog` pelo limite de 4
repositórios por execução.

| Repositório | Delta detectado | Situação |
|---|---|---|
| `ivall/xcl` | 0 commits na `main`, 201 PRs abertos acima do cursor, 6 mesclados (em `nvidia-dev`) | documentado (20 abertos + 6 merges) |
| `R0Y00/Xbox2Windows` | 5 commits, 5 PRs abertos, 10 mesclados | documentado |
| `fxceless/victory-fork` | 14 commits, 11 PRs abertos, 0 mesclados | documentado |
| `holdmysocks/SixNG` | 10 commits, 6 PRs abertos, 10 mesclados | documentado |
| `Marvisak/VIctory` | 50 PRs abertos (teto da janela) | backlog |
| `Belobug/VIctory-Belobug-AMD` | 8 PRs abertos | backlog |
| `DexNom/VIctory` | 24 PRs abertos | backlog |
| `avatarmanz/VIctory-7900XTX` | 8 PRs abertos | backlog |
| `omonkeyz/xbox-emulator` | 1 PR aberto (`#395`) | backlog |
| `brcly/TuxBox` | 2 PRs abertos | backlog |

Sem delta: `uiliannn/VItoriaBr`, `californiaAlmonds/OpenScarlett`, `walkthestars/GTA6-Asset-Info`.

Prioridade usada: repositório com commit ou merge na janela primeiro; depois repositório só com PR aberto;
dentro de cada grupo, o cursor `last_run` mais antigo. `xcl` (22:45Z de 26/11), `victory-fork` (23:06Z),
`Xbox2Windows` (03:01Z) e `SixNG` (03:03Z) eram os quatro mais antigos do grupo com código novo.

## Documentos escritos

Quatro, em `docs/updates/2026-11-27/01/`: `xcl.md`, `Xbox2Windows.md`, `victory-fork.md`, `SixNG.md`.
Cobertura: **29 commits** de 29 na faixa (`victory-fork` 14, `SixNG` 10, `Xbox2Windows` 5, `xcl` 0),
**42 PRs abertos** de 223 detectados nos quatro (`xcl` 20 de 201, `Xbox2Windows` 5, `victory-fork` 11,
`SixNG` 6) e **26 mesclados** de 26 (`SixNG` 10, `Xbox2Windows` 10, `xcl` 6, `victory-fork` 0). Nenhum
repositório chegou perto do teto de 40 commits.

## Falhas

Nenhuma falha de coleta. `gh api rate_limit` no início: 4.945 de 5.000 no core; nenhum 403 em nenhuma
chamada. Nenhum `_falhas.md` foi escrito porque não houve falha por repositório.

Ferramenta de subagente indisponível nesta sessão: os quatro repositórios foram processados em sequência
pelo próprio executor. O limite de 4 foi respeitado como limite de repositórios por execução.

## Desvios registrados

1. **PR mesclado não é detectado por número.** O procedimento manda comparar `merged_at` com o cursor
   `last_run`, não `numero > last_pr_number`; foi o que fiz
   (`pulls?state=closed&sort=updated&per_page=100`, filtrado por `merged_at` **estritamente** maior que o
   cursor). Isso importa: `Belobug` devolve 17 PRs com `mergedAt` igual ao cursor, `VIctory-7900XTX` 25,
   `TuxBox` 1 e `OpenScarlett` 1 — todos já documentados nas horas anteriores. Com `>` eles ficam fora;
   com `>=` seriam recontados a cada hora.
2. **`ivall/xcl`: os 6 merges foram para `nvidia-dev`, não para a default.** A `main` continua em
   `8b957bec` (cursor na posição 0, nenhum commit novo) e o topo de `nvidia-dev` é `ecc0bb57`, à frente
   dos seis merges documentados. Cinco dos seis são fusões reais (dois pais, `+/-` igual ao do PR); o
   `#2895` foi squash (um pai, `+16/-0`).
3. **`R0Y00/Xbox2Windows`: 5 dos 10 merges foram para `cpp-migration`.** Por isso 10 merges contra 5
   commits na `main`. Os cinco da `main` (`#2947`, `#2954`, `#2970`, `#2885`, `#2879`) casam com os cinco
   commits por `merge_commit_sha`; os cinco de `cpp-migration` estão documentados como merges, com os
   arquivos e contagens do PR, sem diff de commit na default.
4. **`ivall/xcl`: 20 dos 201 PRs abertos acima do cursor.** A janela de 50 do `gh pr list` só traz os mais
   novos (2951..2868, todos abaixo do que restava). Documentei os 20 de **menor** número acima de 2661
   (2662..2685), como nas horas anteriores, para o cursor avançar de forma monotônica; restam 181
   (2686..2951). O repositório continua no `backlog` por isso.
5. **`fxceless/victory-fork`: `last_run` não avançou.** Não houve merge na janela, e o cursor de mesclados
   é `mergedAt`, não o horário da execução. Fica em `2026-11-26T23:06:47Z`.
6. **`fxceless/victory-fork`: o PR #1419 está aplicado no fork e continua aberto.** O commit `8a32d0ba`
   aplica o PR externo como commit comum (pai único), resolvendo conflito à mão, e o #1419 segue na lista
   de abertos. O mesmo vale para o "PR #36 recut" em `1170216f`/`1320dfd7`.
7. **Dois candidatos a defeito novo, ambos por diff e não por execução.**
   `4966cd67` (`victory-fork`) liga o construtor de BVH por padrão; o README de `67429060` documenta que o
   título morre escrevendo no endereço 0 entre 30 e 40 s depois de o construtor começar, com e sem
   `VICTORY_BVH_QUARANTINE`, sem correção na janela. `9e0a3980` (`SixNG`) deixa de gravar
   `archetype_kind` por entidade em `placements` e remove `surface_tone`/`surface_tone_note` das peças de
   veículo, enquanto o README reescrito em `a4cff8d4`, na mesma hora, documenta `archetype_kind`.

## Push

Sucesso nas duas branches, na ordem do procedimento (nunca `main` na `public`).

- commit na `public`: `d942d1d` — `docs(updates): 2026-11-27 01:00` (7 arquivos, +456/-51)
- `git push origin public:main`: `7154018..d942d1d  public -> main`; `origin/main` = `d942d1d`
- `git switch main && git merge --no-ff public`: merge `d5e6b34` — `merge updates 2026-11-27 01:00`
- `git push private main`: `955718c..d5e6b34  main -> main`; `private/main` = `d5e6b34`

Conferido com `git ls-remote` nos dois remotes depois do push. Nenhum force, nenhum rebase.

Os hashes acima só existem depois do push, então este parágrafo foi gravado no commit seguinte —
`chore(state): registra hashes do push de 2026-11-27 01:00` —, também publicado na `public` e mesclado
na `main`, como na execução anterior.
