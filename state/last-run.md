# Execução de 2026-11-27 19:00

## Data e hora

Coleta de 2026-11-27T19:07–19:19 UTC real — 16:07–16:19 no relógio do GitHub, que segue 3 h atrás e
é o relógio em que `last_run` é gravado. Diretório de documentos: `docs/updates/2026-11-27/19/`
(UTC). Estado lido: `baselined: true`, `runs: 16` (gravado como 17); 14 entradas em `repos` no
início e no fim, nenhuma removida.

## Sanidade do estado (passo 1)

`gh api repos/<o>/<r> --jq .default_branch` nos 14: idêntico ao gravado em todos, inclusive
`a-publier` (`Belobug/VIctory-Belobug-AMD`) e `master` (`holdmysocks/SixNG`). Nenhum `last_commit`
nulo — não houve baseline nesta execução. Nenhum `compare` devolveu `diverged`. Contagem de `repos`
igual no início e no fim (14); nada foi apagado nem acrescentado.

## Triagem (passo 2)

Os 14 foram triados; 4 entraram no deep dive. Arquivo: `docs/updates/2026-11-27/19/_triagem.md`.

| Repositório | Commits atrás | Abertos acima do cursor | Mesclados na janela | Sinal | Deep dive |
|---|---|---|---|---|---|
| `R0Y00/Xbox2Windows` | 16 | 10 | 16 | SIM (correção) | sim |
| `brcly/TuxBox` | 12 | 0 | 3 | SIM (correção) | sim |
| `fxceless/victory-fork` | 6 | 3 | 0 | SIM (correção/regressão) | sim |
| `ivall/xcl` | 2 | 6 | 1 | SIM (área de D-20/D-21) | sim |
| `holdmysocks/SixNG` | 1 | 1 | 1 | SIM fraco (render, sem termo de correção) | não |
| `Belobug/VIctory-Belobug-AMD` | 0 | ≥100 (357 no total) | 0 | SIM (portas e `fix`) | não |
| `avatarmanz/VIctory-7900XTX` | 0 | 39 (235 no total) | 0 | SIM | não |
| `Marvisak/VIctory` | 0 | 19 (1.656 no total) | 0 | SIM | não |
| `omonkeyz/xbox-emulator` | 0 | 1 (#395) | 0 | SIM | não |
| `DexNom/VIctory` | 0 | 1 (#135) | 0 | SIM fraco | não |
| `uiliannn/VItoriaBr` | 0 | 1 (#132) | 0 | SIM fraco | não |
| `californiaAlmonds/OpenScarlett` | 0 | 0 | 0 | NAO | não |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 | 0 | NAO | não |
| `stellarr-studio/kelvaroq` | 0 | 0 | 0 | NAO | não |

Desempate da quarta vaga: `Belobug`, `avatarmanz` e `Marvisak` empatam no critério 3 com zero commit;
`xcl` tem 2 commits, 1 merge e a pilha #3033–#3035 atacando ordem escrita→espera, o mesmo eixo de
XCL D-20/D-21. Decidido por delta real, como às 18:00.

## Documentos escritos (passo 3)

4 documentos em `docs/updates/2026-11-27/19/`: `Xbox2Windows.md`, `TuxBox.md`, `victory-fork.md`,
`xcl.md`, mais `_triagem.md`. **36 commits documentados de 36** atrás dos cursores (16 + 12 + 6 + 2),
**20 PRs mesclados** (16 + 3 + 0 + 1) e **19 PRs abertos** (10 + 0 + 3 + 6). Nenhum repositório
chegou ao teto de 40 commits.

Achados principais:

- `R0Y00/Xbox2Windows`: quinze dos dezesseis commits fecham, um por um, as recusas
  `direct DXIL lowering for <name>` do tradutor direto de compute (série #2935): `s_brev_b64`/`s_bfe_*64`
  (`0df67295`), `v_cvt_off_f32_i4` (`8929423c`), mul24 (`7a12818c`), os quatro `v_pk_*` de 16 bits
  (`428c08c1`, `4a6302eb`, `c50a85b9`), `s_lshl1..4_add_u32` (`354c6870`), `v_xnor_b32` (`d71f9ed2`),
  bitscan64 (`f6691b67`), `s_sext` (`6c2cddec`), mad32 sem contração em `FMad` (`55669240`), mad64
  (`ca47eb1d`), add/sub saturado (`300b93f0`) e o gather com LOD explícito (`a9fd1488`). O outro é
  `5df098da` (#2919): grava em texto o estado de PSO que o cache de shader não guarda, 64 registros e
  64 KiB por chave, leitura/anexo sob um handle exclusivo, replay antes de
  `CreateGraphicsPipelineState`.
- `brcly/TuxBox`: dos 12 commits, 3 são conteúdo — dedupe de compilação por HLSL com lock de conteúdo
  e snapshot verificado (`e91eedad`, `2057645f`, `2fb96ad7`, `ce7edf2b`, #239), splash de pacote com
  hook de criação de janela e retenção de frames até 4 em 1 s (`7423e478`, #311) e retentativa da
  recusa de stride IDXEN com root SRV `t1, space32766` anexado (`8e641a06`, #310). `d33de277` traz
  116 arquivos, mas o segundo pai está 104 commits atrás do cursor: é main já documentada.
- `fxceless/victory-fork`: a `main` passou a carregar a linha `raytracing` (`bfca0bea`) e liga
  `VICTORY_BVH_BUILDER=1` e `VICTORY_DYNAMIC_WAIT=1` por padrão (`bd6fbce4`, `a0767765`); as páginas
  de fault viraram orçamento reciclável de 512 MB (`3e573d25`). O #1485 mostra que a CI morre no
  Clippy antes de rodar `Tests`, o que mantém D-17 aberto.
- `ivall/xcl`: `#1871` (`f9a0cb80`) adiciona `xcl pso-precompile` para encher o cache de blobs do
  driver antes da execução; a pilha #3033–#3035 corrige a espera em célula escrita por CPU adiada na
  mesma lista, caso que na `main` estoura 2 s e roda sem a escrita.

## Falhas e desvios

- **A "tabela de pistas" citada no passo 2 continua inexistente** em
  `docs/automacao/ANALISE_HORARIA.md`. As áreas do sinal vieram das seções D de
  `catalogo/defeitos.md`, e o sinal dessas linhas é por título/corpo de PR, não por patch. O deep
  dive foi decidido por diff.
- **PRs abertos documentados pelo corpo, não pelo patch.** 19 PRs lidos por título, corpo e contagem
  de arquivos; nenhum patch de PR aberto foi lido linha a linha.
- **16 merges contra 15 commits no `Xbox2Windows`.** O primeiro `compare` (`de1595b9...head`) tinha
  head `ca47eb1d`; o `#3011` (`300b93f0`) foi mesclado um segundo depois e é descendente do cursor
  (`compare` = `ahead 16`), então entrou no documento. Durante a coleta o head foi para `2641cc72`
  (`#3114`), que fica fora de propósito.
- **`victory-fork` sem merge.** Nenhum `merged_at` acima de `2026-11-27T13:05:00Z`; o cursor de
  mesclados ficou parado para não pular merges futuros.
- **Sem subagentes.** A ferramenta de subagente não existe neste ambiente; os quatro repositórios
  foram processados em sequência. O limite de 4 deep dives foi respeitado.
- **Rate limit**: `gh api rate_limit` marcava 4.985 de 5.000 no início; nenhuma chamada devolveu 403.
  `search/issues` recusou `ivall/xcl` com HTTP 422 (repositório transferido) — usado só para total de
  PRs, nada de coleta essencial dependeu dele.
- Nenhum `_falhas.md`: não houve erro de coleta ou de escrita. Nenhuma escrita fora de
  `G:\gta6-port-knowledge`.

## Estado gravado

`runs: 17`, `updated: 2026-11-27T19:19:05Z`, `last_error: null`. Cursores que avançaram:

| Repositório | `last_commit` | `last_pr_number` | `last_run` |
|---|---|---|---|
| `R0Y00/Xbox2Windows` | `300b93f0` (head na hora do snapshot; `2641cc72` pendente) | 3121 | 2026-11-27T16:08:23Z |
| `brcly/TuxBox` | `c2a59af5` (cursor e head) | 311 | 2026-11-27T15:28:30Z |
| `fxceless/victory-fork` | `3e573d25` (cursor e head) | 1487 | inalterado (sem merge) |
| `ivall/xcl` | `f9a0cb80` (cursor e head) | 3038 | 2026-11-27T15:11:50Z |

Os outros 10 mantêm cursor de commit e de merge. `backlog` com 7: `Belobug/VIctory-Belobug-AMD`,
`avatarmanz/VIctory-7900XTX`, `Marvisak/VIctory`, `holdmysocks/SixNG`, `omonkeyz/xbox-emulator`,
`DexNom/VIctory`, `uiliannn/VItoriaBr`. A ordem da próxima hora está no `backlog_note`.

## Push

- Commit na `public`: `12864ca` — `docs(updates): 2026-11-27 19:00` (8 arquivos, 479 inserções, 96
  remoções: os 4 documentos, `_triagem.md`, `docs/updates/INDEX.md`, `state/watch.json` e
  `state/last-run.md`).
- `git push origin public:main`: **sucesso** — `7e0c4ea..12864ca  public -> main`.
- `git switch main` + `git merge --no-ff public`: **conflito em `state/last-run.md`** (a `main`
  carregava a correção do registro das 18:00 em `b98fc99`, a `public` trazia o registro desta hora).
  Resolvido a favor da `public` (`git checkout --theirs`), porque o registro desta execução substitui
  o da anterior; merge `d2ffde3` — `merge updates 2026-11-27 19:00`.
- `git push private main`: **sucesso** — `b98fc99..d2ffde3  main -> main`.
- Este fechamento (registro do push) é um segundo commit na `public`, depois do merge. Para não criar
  um segundo merge na execução (o passo 5 limita a um), a `main` recebe esse commit por cherry-pick,
  como na execução das 18:00.
- Nenhum `git reset --hard`, nenhum merge de `main` na `public`: a `public` só recebeu commits
  próprios e a `main` recebeu a `public`.

