# Execução de 2026-11-28 12:00

## Data e hora

Coleta de 2026-11-28T12:13–12:40Z (UTC real). Diretório de documentos: `docs/updates/2026-11-28/12/`.
Estado lido: `baselined: true`, `runs: 23` (gravado como 24); 14 entradas em `repos` no início e no fim,
nenhuma removida, nenhuma acrescentada.

Nesta coleta o relógio do GitHub está **alinhado ao UTC real** — o commit `2b2277a5` do `SixNG`, de
`12:16:23Z`, apareceu às 12:19Z —, ao contrário das horas anteriores, em que os timestamps vinham ~3 h
atrás. Os `last_run` gravados são, como sempre, o maior `merged_at` **documentado** de cada repositório.

## Sanidade do estado (passo 1)

`gh api repos/<o>/<r> --jq .default_branch` foi pedido nos 14. **Dois continuam sem resposta**:
`Belobug/VIctory-Belobug-AMD` e `avatarmanz/VIctory-7900XTX` devolvem HTTP 404 (sexta hora seguida);
`default_branch` não reconferida, cursores parados, triagem NAO VERIFICADO. Detalhe em
`docs/updates/2026-11-28/12/_falhas.md`.

Nos outros 12, `default_branch` idêntico ao gravado, inclusive `master` (`holdmysocks/SixNG`). Nenhum
`last_commit` nulo — não houve baseline. Nenhum `compare` devolveu `diverged` (5 `ahead`, 7
`identical`). Contagem de `repos` igual no início e no fim (14).

## Triagem (passo 2)

Os 14 foram triados (12 com medição completa); 4 entraram no deep dive. Arquivo:
`docs/updates/2026-11-28/12/_triagem.md`.

| Repositório | Commits atrás | Abertos acima do cursor | Mesclados na janela | Sinal | Deep dive |
|---|---|---|---|---|---|
| `ivall/xcl` | 376 | 36 (100 lidos, máx. #3125) | 48 (29 em `nvidia-dev`) | SIM (critérios 2 e 4) | sim |
| `brcly/TuxBox` | 169 | 11 (13 no total) | 50 | SIM (critério 2) | sim |
| `R0Y00/Xbox2Windows` | 121 | 39 (100 lidos, máx. #3352) | 106 (2 em `codex/…`) | SIM (critério 2) | sim |
| `holdmysocks/SixNG` | 33 (36 na lista) | 5 (7 no total) | 34 (+#518 depois do recorte) | SIM (critério 2) | sim |
| `fxceless/victory-fork` | 8 | 8 (máx. #1513) | 2 | SIM (critério 2) | não (limite) |
| `Marvisak/VIctory` | 0 | 181 (200 lidos, máx. #3728) | 0 | SIM fraco (critério 3) | não |
| `DexNom/VIctory` | 0 | 4 (máx. #152) | 0 | SIM fraco (critério 3) | não |
| `uiliannn/VItoriaBr` | 0 | 3 (máx. #134) | 0 | SIM fraco (critério 3) | não |
| `omonkeyz/xbox-emulator` | 0 | 1 (#395) | 0 | SIM fraco (critério 3) | não |
| `californiaAlmonds/OpenScarlett` | 0 | 1 (#66) | 0 | NAO | não |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 | 0 | NAO | não |
| `stellarr-studio/kelvaroq` | 0 | 0 | 0 | NAO | não |
| `Belobug/VIctory-Belobug-AMD` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (404) | bloqueado |
| `avatarmanz/VIctory-7900XTX` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (404) | bloqueado |

Desempate: cinco repositórios tinham sinal por correção e o limite é 4. Ficou fora o de menor janela,
`fxceless/victory-fork` (8 commits, 2 merges). O critério 1 (defeito do catálogo) **não selecionou
ninguém**: os 710 hashes da janela dos cinco repositórios com commit foram conferidos contra as 306
linhas de `catalogo/defeitos.csv` (262 hashes distintos) sem interseção. O critério 4 só acionou em
`xcl` (376 > 200).

## Documentos escritos (passo 3)

4 documentos em `docs/updates/2026-11-28/12/`: `xcl.md`, `TuxBox.md`, `Xbox2Windows.md`, `SixNG.md`,
mais `_triagem.md` e `_falhas.md`. **156 commits documentados de 702** na janela (376 + 169 + 121 + 36),
**80 PRs mesclados** documentados (35 + 35 + 2 + 8) e **89 PRs abertos** lidos por título e corpo.

Achados principais:

- **`xcl` é o maior atraso já medido:** 376 commits, 40 documentáveis por hora, 336 restantes — cerca de
  9 horas de deep dive exclusivo. Os 40 são a série de *pipeline library* nativa (receita de PSO em
  `used-graphics-pipelines.bin`, corpus em `.xcl/pso-corpus` com `pso_archive.py`/`stage.py` novos,
  biblioteca serializada e ativada no arranque) e o núcleo de *stall* (`stall.rs`, `build_queue.rs`
  declarado **não ligado**). Consertos reais: `2dad32e6` tira o laço `[unroll]` do `ballot_px_quad` que
  o DXC desenrolava e quebrava o `amdxc64`; `abf7e177` limita a janela coerente a RAM/8 abaixo de 24 GB.
- **`TuxBox`:** a hora é do *harness* de GPU (13 commits de `tools/check_*`, quatro regenerações do
  inventário) e da forma dois-lanes. O achado com evidência é `24ebba7e`: `WaveActiveMin` sobre índices
  de lane, usado para achar a primeira lane ativa de `s_ff1_i32_b64`, **derrubou a GPU** numa RTX 5090
  (Xid 109, CTX SWITCH TIMEOUT); virou ballot. Também `94f3803b`: a barreira global de argumentos
  indiretos usava `ACCESS_COMMON`, que o runtime de desktop recusa antes de acesso nomeado, e o
  `Close` da lista falhava.
- **`Xbox2Windows`:** maior leva de lowerings do DXIL direto até agora (27 dos 40 em
  `gfx1013_direct_compute.inl`/`gfx1013_compute_native.cpp`/manifesto) e três consertos de runtime:
  `4b0350ea` recusa o draw quando o ring de upload não dá espaço (causa nova `ring_full`) em vez de
  ligar zeros; `fad97f47` troca `__try/__except` por leitura guardada antes da cópia; `2e12dd8e`
  identifica o canto do retângulo girado pela diagonal (a região ficava sem pintura, #3131). O merge
  `fd41d5a7` traz a `main` inteira para a branch de slot 70 (26 arquivos novos; `direct_compute_cache.cpp`
  925 linhas) e **não** é trabalho novo da hora.
- **`SixNG`:** o bake de tatuagem mudou de regra duas vezes (`BAKE_RULE` 10→12); `a565fd9f` (#530)
  troca a UV do tronco da quarta para a segunda **sem bump no mesmo commit**, então um cache de bake
  antigo é aceito pela regra nova; a tabela de cabelo inferida passou a casar por nome de peça
  (`ac0d9608`, `CACHE_VERSION` 6) e o mapa 3D ganhou renderizador próprio (`gpu_draw.rs`, 1.181
  linhas) que evoluiu em cinco commits na mesma hora.

## Falhas e desvios

- **Dois repositórios inacessíveis (404, sexta hora).** Não avançaram cursor, não foram documentados,
  não saíram de `repos`.
- **A "tabela de pistas" citada no passo 2 continua inexistente.** As áreas do sinal vieram das seções
  D do `catalogo/defeitos.md` e do assunto/corpo de commits e PRs.
- **Três repositórios bateram o teto de 40** e ficam com `commit_cursor`. O `compare` a partir dele não
  devolve exatamente o resto da janela (338 vs 336 em `xcl`, 148 vs 129 em `TuxBox`, 86 vs 81 em
  `Xbox2Windows`), porque o 40º commit é um merge cuja linhagem contém commits já documentados. Medido
  e registrado em cada documento.
- **Sem subagentes.** A ferramenta não existe neste ambiente; os quatro repositórios foram processados
  em sequência.
- **Rate limit:** 4.999 no início, 4.678 no fim, nenhum 403, nenhuma repetição em laço.
- **Escrita:** somente em `G:\gta6-port-knowledge`. Os 14 monitorados foram só leitura de API.
- **Arquivos de terceiros no diretório de trabalho** (`docs/analises/victory-fork.md` modificado e dois
  relatórios não rastreados em `docs/relatorios/`) ficaram fora do `git add` desta rotina e intactos.

## Estado gravado

`runs: 24`, `updated: 2026-11-28T12:44:00Z`, `last_error` com os dois 404 (sexta hora). Cursores que
avançaram:

| Repositório | `last_commit` | `commit_cursor` | `last_pr_number` | `last_run` |
|---|---|---|---|---|
| `holdmysocks/SixNG` | `2b2277a5` (head) | — | 541 | 2026-11-28T12:14:29Z |
| `ivall/xcl` | `2c717a04` (parado) | `3dbac13a` | 3075 | 2026-11-28T01:14:26Z |
| `brcly/TuxBox` | `1d0f3fa9` (parado) | `8420ba03` | 339 | 2026-11-28T00:05:08Z |
| `R0Y00/Xbox2Windows` | `886a5a28` (parado) | `50e4f763` | 3215 | 2026-11-28T01:42:49Z |

Os outros 10 mantêm cursor de commit e de merge. `head_at_run` gravado nos três com teto
(`e429936c`, `6f89916a`, `052abe6f`). `backlog` com 10: os dois bloqueados, os três com teto e
`fxceless/victory-fork`, `Marvisak/VIctory`, `DexNom/VIctory`, `uiliannn/VItoriaBr`,
`omonkeyz/xbox-emulator`; a ordem e os achados estão no `backlog_note`.

## Push

O registro abaixo foi escrito depois das duas publicações, no fechamento de 2026-11-28T12:29Z.

- Commit na `public`: `cff0570` — `docs(updates): 2026-11-28 12:00` (9 arquivos, 827 inserções, 116
  remoções: os 4 documentos, `_triagem.md`, `_falhas.md`, `docs/updates/INDEX.md`, `state/watch.json`
  e `state/last-run.md`).
- `git push origin public:main`: **sucesso** — `763903a..cff0570  public -> main` (a saída do git vai
  para stderr e o PowerShell a reporta como erro; o intervalo de refs confirma o envio).
- `git switch main` + `git merge --no-ff public`: **conflito em `state/last-run.md`** (a `main` tinha o
  registro de push da execução das 01:00, cherry-pickado; a `public` trazia o arquivo novo). Resolvido
  tomando a versão da `public` (`git checkout --theirs`) e conferindo o resto do merge: os outros 8
  arquivos entraram sem conflito. Merge `06938b2` — `merge updates 2026-11-28 12:00`.
- `git push private main`: **sucesso** — `4c0f8cf..06938b2  main -> main`.
- Este fechamento (registro do push) é um segundo commit na `public`, depois do merge. Para não criar
  um segundo merge na execução (o passo 5 limita a um), a `main` recebe esse commit por cherry-pick,
  como nas execuções das 18:00 às 01:00.
- Nenhum `git reset --hard`, nenhum merge de `main` na `public`: a `public` só recebeu commits próprios
  e a `main` recebeu a `public`.
- Ficaram fora do commit, intactos: `docs/analises/victory-fork.md` modificado e os dois relatórios não
  rastreados em `docs/relatorios/`, que não são desta rotina.
