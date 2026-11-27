# Execução de 2026-11-27 17:00

## Data e hora

Coleta de 2026-11-27T14:00–14:25 no relógio dos dados do GitHub e no horário local do operador
(UTC−3) — 17:00–17:25 UTC real. Diretório de documentos: `docs/updates/2026-11-27/17/` (UTC).
Estado lido: `baselined: true`, `runs: 14` (gravado como 15); 14 entradas em `repos`, nenhuma removida.

## Triagem (passo 2)

Primeira execução no desenho **triagem-primeiro**: os 14 foram triados, sempre, e 4 entraram no deep dive.
Arquivo: `docs/updates/2026-11-27/17/_triagem.md`.

| Repositório | Commits atrás | Abertos acima do cursor | Mesclados na janela | Sinal | Deep dive |
|---|---|---|---|---|---|
| `R0Y00/Xbox2Windows` | 79 (47 novos) | 5 | 6 | SIM | sim |
| `brcly/TuxBox` | 14 | 1 | 2 | SIM | sim |
| `holdmysocks/SixNG` | 1 | 0 | 1 | SIM | sim |
| `DexNom/VIctory` | 0 | 26 | 0 | SIM | sim |
| `Belobug/VIctory-Belobug-AMD` | 0 | 50 (piso #551) | 0 | SIM | não |
| `avatarmanz/VIctory-7900XTX` | 0 | 37 | 0 | SIM | não |
| `fxceless/victory-fork` | 0 | 10 | 0 | SIM | não |
| `ivall/xcl` | 0 | 4 | 0 | SIM | não |
| `Marvisak/VIctory` | 0 | 3 | 0 | SIM | não |
| `omonkeyz/xbox-emulator` | 0 | 1 (#395) | 0 | SIM | não |
| `californiaAlmonds/OpenScarlett` | 0 | 0 (#60 abaixo do cursor) | 0 | NAO | não |
| `uiliannn/VItoriaBr` | 0 | 0 | 0 | NAO | não |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 | 0 | NAO | não |
| `stellarr-studio/kelvaroq` | 0 | 0 | 0 | NAO | não |

Nenhum `compare` devolveu `diverged`. `default_branch` reconferido nos 14 e igual ao gravado.

## Documentos escritos (passo 3)

4 documentos em `docs/updates/2026-11-27/17/`: `Xbox2Windows.md`, `TuxBox.md`, `SixNG.md`,
`VIctory-DexNom.md`, mais `_triagem.md`. Total: **55 commits** (40 + 14 + 1 + 0) de **62** na janela,
**9 PRs mesclados** (6 + 2 + 1) e **32 PRs abertos** (5 + 1 + 0 + 26).

`R0Y00/Xbox2Windows` é o único com corte de teto: 47 commits novos desde `f5204690`, documentados os 40
mais antigos (até `801f4c3b`), com `commit_cursor = 801f4c3b` e `head_at_run = e1ea515d`; os 7 restantes
(`db28a796`, `ff04a75e`, `246857e4`, `36d2cbeb`, `2972e00a`, `9072230f`, `e1ea515d`) voltam na próxima hora.
Dos 6 merges dele, 5 estão acima do teto e foram registrados **só pelo assunto**, marcados na tabela.

## Falhas e desvios

- **A "tabela de pistas" citada no passo 2 não existe.** `docs/automacao/ANALISE_HORARIA.md` cita, na
  linha 231, "arquivo de área listada na tabela de pistas", mas nenhuma tabela de pistas existe no
  procedimento nem em outro arquivo do acervo (`grep` por "pista" devolve só essa linha e duas menções
  em documentos de atualização). Na ausência dela, as áreas do sinal foram tomadas das seções D de
  `catalogo/defeitos.md` (tradutor DXIL/`gfx2hlsl`, descritor e heap, memória e XMem, textura e tiling,
  present e save). O critério 3 do passo 3 ficou, na prática, subsumido pelo critério 2 nesta hora.
- **Reaparecimento de commits já documentados no `compare`.** No `Xbox2Windows`, `compare` desde
  `f5204690` devolveu 79, mas 32 (`40365770` a `999e17cd`) são commits da linha `cpp-migration` que já
  estavam no documento das 16:00 e **não são ancestrais de `f5204690`** (medido: `compare/40365770...f5204690`
  dá `diverged`, `merge_base 6ee878d8`; `compare/40365770...e1ea515d` dá `ahead` com `behind_by 0`). Eles
  reaparecem até o cursor passar pelo merge `91cbedd6` (10:13), que os traz para a `main`. Não foram
  redocumentados; a contagem "atrás" na triagem traz os dois números. Registrado como pendência estrutural 3.
- **`pulls?state=open` com um único PR não é lista.** Em `OpenScarlett` (#60) e `SixNG` (#472) a resposta é
  um objeto, não um array; a contagem inicial devolveu 0 e foi corrigida por reconferência antes de escrever
  a triagem. Os dois estão abaixo do cursor e não foram triados além de título e estado.
- **PRs abertos documentados sem diff.** Os 26 do `DexNom/VIctory` foram documentados pela proposta que o
  autor declara no corpo (lido até 700 caracteres) e pela lista de arquivos; nenhum patch de PR aberto foi
  lido linha a linha. A tabela diz isso na coluna e na seção Confiança do documento.
- **Sem subagentes.** A ferramenta de subagente não existe neste ambiente; os quatro repositórios foram
  processados em sequência. O limite de 4 deep dives foi respeitado.
- **Rate limit**: `gh api rate_limit` marcava 4.919 de 5.000 no início; nenhuma chamada devolveu 403.
- Nenhum `_falhas.md`: não houve erro de coleta ou de escrita em repositório monitorado. Nenhuma escrita
  fora de `G:\gta6-port-knowledge`.

## Estado gravado

`runs: 15`, `updated: 2026-11-27T17:22:16Z`, `last_error: null`. Cursores que avançaram:

| Repositório | `last_commit` | `last_pr_number` | `last_run` |
|---|---|---|---|
| `R0Y00/Xbox2Windows` | `801f4c3b` (cursor), `e1ea515d` (head) | 3071 | 2026-11-27T13:55:58Z |
| `brcly/TuxBox` | `76d1abe0` | 309 | 2026-11-27T14:07:09Z |
| `holdmysocks/SixNG` | `2d9c230d` | 472 (inalterado) | 2026-11-27T13:55:15Z |
| `DexNom/VIctory` | inalterado | 134 | inalterado (sem merge) |

Os outros 10 mantêm cursor de commit e de merge; `OpenScarlett`, `VItoriaBr`, `GTA6-Asset-Info` e
`kelvaroq` não tiveram delta. `backlog` com 7: `R0Y00/Xbox2Windows` (7 commits), `Belobug/VIctory-Belobug-AMD`,
`avatarmanz/VIctory-7900XTX`, `fxceless/victory-fork`, `omonkeyz/xbox-emulator`, `ivall/xcl`,
`Marvisak/VIctory`. A ordem da próxima hora está no `backlog_note`.

## Push

- Commit na `public`: `083ddf7` — `docs(updates): 2026-11-27 17:00` (8 arquivos, 412 inserções, 86
  remoções: os 4 documentos, `_triagem.md`, `docs/updates/INDEX.md`, `state/watch.json` e
  `state/last-run.md`).
- `git push origin public:main`: **sucesso** — `000bfb8..083ddf7  public -> main`; `origin/main` =
  `083ddf71096d7632b143e8b4bb9ca5d3e9c959d1`.
- Este fechamento (registro do push) é o segundo commit da mesma execução; o merge `--no-ff` de `public`
  na `main` é **um só** e vem depois dele, como o passo 5 limita ("um merge por execução, no máximo").
- Nenhum `git reset --hard`, nenhum merge de `main` na `public`: a `public` só recebeu commits próprios e
  a `main` recebeu a `public`.

