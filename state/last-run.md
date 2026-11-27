# Execução de 2026-11-27 22:00

## Data e hora

Coleta de 2026-11-27T22:05–22:36 UTC real — ~19:05–19:36 no relógio do GitHub, que segue cerca de 3 h
atrás e é o relógio em que `last_run` é gravado. Diretório de documentos: `docs/updates/2026-11-27/22/`
(UTC). Estado lido: `baselined: true`, `runs: 19` (gravado como 20); 14 entradas em `repos` no início e no
fim, nenhuma removida, nenhuma acrescentada.

## Sanidade do estado (passo 1)

`gh api repos/<o>/<r> --jq .default_branch` foi pedido nos 14. **Dois continuam sem resposta**:
`Belobug/VIctory-Belobug-AMD` e `avatarmanz/VIctory-7900XTX` devolvem HTTP 404 (segunda hora seguida);
a `default_branch` deles não pôde ser reconferida, os cursores ficaram parados e a triagem deles é NAO
VERIFICADO. O caso está em `docs/updates/2026-11-27/22/_falhas.md`.

Nos outros 12, `default_branch` idêntico ao gravado, inclusive `master` (`holdmysocks/SixNG`). Nenhum
`last_commit` nulo — não houve baseline nesta execução. Nenhum `compare` devolveu `diverged`. Contagem de
`repos` igual no início e no fim (14); 14 chaves relidas do arquivo gravado.

## Triagem (passo 2)

Os 14 foram triados (12 com medição completa); 4 entraram no deep dive. Arquivo:
`docs/updates/2026-11-27/22/_triagem.md`.

| Repositório | Commits atrás | Abertos acima do cursor | Mesclados na janela | Sinal | Deep dive |
|---|---|---|---|---|---|
| `brcly/TuxBox` | 25 | 2 (6 no total) | 7 | SIM (critério 2; 17 commits de ramo em `xshader`) | sim |
| `holdmysocks/SixNG` | 8 | 8 (8 no total) | 8 | SIM (critério 2 fraco; culling e leitura fora da thread de UI) | sim |
| `R0Y00/Xbox2Windows` | 5 | 10 (≥100 no total) | 5 | SIM (critério 2; loader sem diálogo, contadores de storage) | sim |
| `fxceless/victory-fork` | 1 | 0 (≥100 no total) | 0 | SIM (critério 2; água escura em host de 32 lanes) | sim |
| `Marvisak/VIctory` | 0 | 55 (≥100 no total) | 0 | SIM (critério 3; lote `Shaders:`/`GPU:`) | não |
| `ivall/xcl` | 0 | 20 (≥100 no total) | 0 | SIM (critério 3; bibliotecas nativas de pipeline) | não |
| `DexNom/VIctory` | 0 | 2 (≥100 no total) | 0 | SIM fraco (critério 3) | não |
| `omonkeyz/xbox-emulator` | 0 | 1 (25 no total) | 0 | SIM (critério 3; #395) | não |
| `uiliannn/VItoriaBr` | 0 | 1 (75 no total) | 0 | SIM fraco (critério 3) | não |
| `californiaAlmonds/OpenScarlett` | 0 | 0 (1 no total) | 0 | NAO | não |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 | 0 | NAO | não |
| `stellarr-studio/kelvaroq` | 0 | 0 | 0 | NAO | não |
| `Belobug/VIctory-Belobug-AMD` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (404) | bloqueado |
| `avatarmanz/VIctory-7900XTX` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (404) | bloqueado |

Desempate: o critério 1 (defeito do catálogo) não selecionou nenhum dos 39 commits; o critério 2
selecionou os quatro repositórios com commit na janela, que ocuparam as quatro vagas. `Marvisak` (55 PRs
novos) e `xcl` (20) perdem a vaga porque o critério 3 vem depois do 2 e as vagas acabaram.

## Documentos escritos (passo 3)

4 documentos em `docs/updates/2026-11-27/22/`: `TuxBox.md`, `SixNG.md`, `Xbox2Windows.md`,
`victory-fork.md`, mais `_triagem.md` e `_falhas.md`. **39 commits documentados de 39** atrás dos cursores
(25 + 8 + 5 + 1), **20 PRs mesclados** (8 + 7 + 5) e **17 PRs abertos** (2 + 8 + 10 + 0). Nenhum
repositório chegou ao teto de 40 commits.

Achados principais:

- `fxceless/victory-fork` (`cd935520`): o programa `water_shading:CS_TransmissionVolumetrics_GenerateTiles`
  distribuía slots entre as duas listas de tiles assumindo wave de 64 lanes; num host de 32 lanes a segunda
  metade sobrescrevia slots da primeira e os tiles ficavam sem luz (blocos de 16 × 16 px em volta de barcos
  e píeres, `run 421`, frame 2822). O commit reescreve os dois appends byte a byte, com guarda de digest do
  container, comprimento 2112 e FNV do código, trocando `v_mov_b32 v5, 0` por `v5, 1` e enchendo o resto
  com `s_nop`. **Achado verificável:** lido `crates/victory-gpu/src/shader_cache.rs` no commit, a chave do
  cache inclui `VICTORY_THREAD_TICKET` (acrescentado às 21:00) mas **não** `VICTORY_WATER_TILE_SLOTS`, o
  switch novo que muda o código gerado — a classe de D-02 volta a valer por mais um switch; a mesma lista
  continua sem `VICTORY_MEMORY_NEAR`, `VICTORY_LDS_START` e `VICTORY_WAVE_LOOP_SKIPS`.
- `brcly/TuxBox`: 17 commits de ramo, 7 merges de PR e 1 merge de `main` (`dc92a00e`, que traz de volta
  seis commits já documentados — helper DXC `89fb2517`, `8c082e65` do #312, `7953f0a5` do #313 e a
  evidência `e3cf2096`). O trabalho novo preenche do descritor os campos que o programa lê (tamanho de
  textura `e8fb84e6`, contagem de registros de buffer `1ad86d0b`/`f7e404cd`, textura vazia como um texel
  `0a5ebfa7`), verifica a eleição de lane por registradores exatos (`5f3bb04e`, `6da26fe5`, `8ad601d5`,
  `db31dd09`), roda grupo de 32 threads como metade de wave (`5fea7f7e`) e conta chaves por lane
  (`ea19a35c`); o cache de runtime passa a guardar três builds com trava `in-use` (`990b1bfb`,
  `ffffb751`). O `dxc_worker.cpp` reescreve à mão a propriedade 11 do entry point para repor
  `[WaveSize(N)]` e, se a metadata não for a esperada, cai para a CLI — falha segura, mas o caminho rápido
  se perde.
- `holdmysocks/SixNG`: perfis de subsuperfície da pele lidos do arquivo de perfis do motor (`d8101f48`,
  struct `a0ec2837`/`73bc6e91`, `skin_default`), larguras de fio das rampas (`01f50f8e`), BC7 por tile de
  água interior (`102a534a`), tatuagens de braço único no quarto UV com `BAKE_RULE` 6→8 (`4acbe12f`),
  culling por mesh para ≤ 8 instâncias com **só quatro planos laterais** (`77c5fbc6`) e leitura de livery
  fora da thread de UI sem limite de threads no diff (`97eca4cb`).
- `R0Y00/Xbox2Windows`: `88cddb09` põe `SEM_FAILCRITICALERRORS` no modo de erro da thread em volta do
  `LoadLibraryExW` e restaura depois; `e92776bd` imprime `raw=`/`completed=` com
  `completed = raw + inflated + bcpack-blank`, o que contradiz o comentário do mesmo arquivo de que
  requisição comprimida fica sem fazer; `0e3ebe36` amplia a tolerância dos probes lineares para um passo
  de peso de 1/256 escalado pelo span do footprint; `572ec6f8` dá um device por teste com servidor
  `--serve`; `7832b98b` captura a janela a cada N segundos com teto de 120 arquivos.

## Falhas e desvios

- **Dois repositórios inacessíveis (404 de conta, segunda hora).** Registrado em `_falhas.md`. Não
  avançaram cursor, não foram documentados, não saíram de `repos`.
- **A "tabela de pistas" citada no passo 2 continua inexistente** em `docs/automacao/ANALISE_HORARIA.md`.
  As áreas do sinal vieram das seções D de `catalogo/defeitos.md`, e o sinal das linhas sem commit é por
  título/corpo de PR, não por patch. O deep dive foi decidido por diff.
- **Contagem de commits x de merges:** bate nos quatro documentados. `SixNG` e `Xbox2Windows` são squash
  de um pai; `TuxBox` tem 7 merges de PR de dois pais + 1 merge de linha (`dc92a00e`) e 17 commits de
  ramo; `victory-fork` tem commit direto e zero merges.
- **PRs abertos documentados pelo título, não pelo patch.** 17 PRs lidos por título, base e head; nenhum
  patch de PR aberto foi lido linha a linha.
- **Sem subagentes.** A ferramenta não existe neste ambiente; os quatro repositórios foram processados em
  sequência e o limite de 4 deep dives foi respeitado.
- **Rate limit:** 4.889 restantes no início, nenhum 403, nenhuma repetição em laço. O gasto total não foi
  medido ao fim para não consumir a janela.
- **Escrita:** somente em `G:\gta6-port-knowledge`. Os 14 monitorados foram só leitura de API.

## Estado gravado

`runs: 20`, `updated: 2026-11-27T22:36:00Z`, `last_error` com os dois 404 (sem mudança). Cursores que
avançaram:

| Repositório | `last_commit` | `last_pr_number` | `last_run` |
|---|---|---|---|
| `R0Y00/Xbox2Windows` | `88cddb09` (head) | 3159 | 2026-11-27T18:51:00Z |
| `brcly/TuxBox` | `d37f092c` (head) | 324 | 2026-11-27T19:05:19Z |
| `holdmysocks/SixNG` | `9346d0e6` (head) | 497 | 2026-11-27T19:07:18Z |
| `fxceless/victory-fork` | `cd935520` (head) | 1493 (sem PR novo) | 2026-11-27T18:10:00Z (sem merge) |

`last_run` dos três com merge é o maior `merged_at` observado na janela, no relógio do GitHub — não o
horário da execução, para não esconder merge entre o fim da coleta e a próxima hora. Os outros 10 mantêm
cursor de commit e de merge. `backlog` com 7: os dois bloqueados e `Marvisak/VIctory`, `ivall/xcl`,
`omonkeyz/xbox-emulator`, `DexNom/VIctory`, `uiliannn/VItoriaBr`. A ordem e o achado do cache de shader
estão no `backlog_note`.

## Push

- Commit na `public`: `35637f9` — `docs(updates): 2026-11-27 22:00` (9 arquivos, 548 inserções, 111
  remoções: os 4 documentos, `_triagem.md`, `_falhas.md`, `docs/updates/INDEX.md`, `state/watch.json` e
  `state/last-run.md`).
- `git push origin public:main`: **sucesso** — `d9dc319..35637f9  public -> main`.
- `git switch main` + `git merge --no-ff public`: **conflito em `state/last-run.md`** (a `main` carregava
  o registro do push das 21:00 em `954060e`, a `public` trazia o registro desta hora). Resolvido a favor
  da `public` (`git checkout --theirs`), porque o registro desta execução substitui o da anterior; merge
  `22b6683` — `merge updates 2026-11-27 22:00`.
- `git push private main`: **sucesso** — `954060e..22b6683  main -> main`.
- Este fechamento (registro do push) é um segundo commit na `public`, depois do merge. Para não criar um
  segundo merge na execução (o passo 5 limita a um), a `main` recebe esse commit por cherry-pick, como nas
  execuções das 18:00, 19:00, 20:00 e 21:00.
- Nenhum `git reset --hard`, nenhum merge de `main` na `public`: a `public` só recebeu commits próprios e
  a `main` recebeu a `public`.
- Ficaram fora do commit, intactos: `docs/analises/victory-fork.md` modificado e os dois relatórios não
  rastreados em `docs/relatorios/`, que não são desta rotina.
