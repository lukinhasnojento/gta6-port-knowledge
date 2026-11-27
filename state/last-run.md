# Execução de 2026-11-27 18:00

## Data e hora

Coleta de 2026-11-27T15:07–15:45 no horário local do operador (UTC−3) — 18:07–18:45 UTC real, com
os timestamps do GitHub ainda 3 h atrás do UTC real (relógio em que `last_run` é gravado). Diretório
de documentos: `docs/updates/2026-11-27/18/` (UTC). Estado lido: `baselined: true`, `runs: 15`
(gravado como 16); 14 entradas em `repos` no início e no fim, nenhuma removida.

## Triagem (passo 2)

Os 14 foram triados; 4 entraram no deep dive. Arquivo: `docs/updates/2026-11-27/18/_triagem.md`.

| Repositório | Commits atrás | Abertos acima do cursor | Mesclados na janela | Sinal | Deep dive |
|---|---|---|---|---|---|
| `R0Y00/Xbox2Windows` | 11 | 14 | 4 | SIM | sim |
| `brcly/TuxBox` | 2 | 2 | 1 | SIM | sim |
| `ivall/xcl` | 2 | 10 | 1 | SIM | sim |
| `fxceless/victory-fork` | 1 (284 arquivos) | 13 | 0 | SIM | sim |
| `holdmysocks/SixNG` | 1 | 0 | 1 | SIM (área de render) | não |
| `Belobug/VIctory-Belobug-AMD` | 0 | ≥61 acima de #456 | 0 | SIM | não |
| `avatarmanz/VIctory-7900XTX` | 0 | 39 | 0 | SIM | não |
| `Marvisak/VIctory` | 0 | 8 | 0 | SIM | não |
| `omonkeyz/xbox-emulator` | 0 | 1 (#395) | 0 | SIM | não |
| `DexNom/VIctory` | 0 | 0 | 0 | NAO | não |
| `californiaAlmonds/OpenScarlett` | 0 | 0 | 0 | NAO | não |
| `uiliannn/VItoriaBr` | 0 | 0 | 0 | NAO | não |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 | 0 | NAO | não |
| `stellarr-studio/kelvaroq` | 0 | 0 | 0 | NAO | não |

Nenhum `compare` devolveu `diverged`. `default_branch` reconferido nos 14 por `gh api repos/<o>/<r>
--jq .default_branch` e igual ao gravado, inclusive `a-publier` e `master`. Nenhum `last_commit`
nulo: não houve baseline nesta execução.

## Documentos escritos (passo 3)

4 documentos em `docs/updates/2026-11-27/18/`: `Xbox2Windows.md`, `TuxBox.md`, `xcl.md`,
`victory-fork.md`, mais `_triagem.md`. **16 commits documentados de 17** atrás dos cursores,
**6 PRs mesclados** (4 + 1 + 1 + 0) e **39 PRs abertos** (14 + 2 + 10 + 13). Nenhum repositório
chegou ao teto de 40 commits; o único commit não documentado é o `SixNG` (performance, sem sinal).

Achados principais:

- `R0Y00/Xbox2Windows`: `246857e459` (#3058) substitui `Round_ne`/`fptoui` por aritmética inteira na
  conversão U8 empacotada porque o literal 0,5 voltava como 1 em driver NVIDIA (#3052) — é a correção
  de uma regressão do #2997. `2972e00aaf` (#2871) habilita volume 0x105 de um mip no slot 70;
  `36d2cbebde` (#3028) torna opt-in o pulo da cópia de 64 KiB do root SRV do slot 44; `ff04a75e58`
  (#3056) cria um carregador de manifesto de patch com gate de SHA-256 que ainda não aplica nada.
- `fxceless/victory-fork`: `5dbf5ce226` troca a árvore inteira pela de
  `trustno1-coder/VIctory-NVIDIA@959dcaf` (284 arquivos, +95.231/−913.607). Conferi por download no
  topo novo os símbolos dos consertos D-01 a D-06 (`VICTORY_WAVE_WIDE`, `WINDOW_MOST`,
  `RECHECK_FRAMES`, `staged_memory`, `unspill`/`parks`, `V_HOST_MAX_WAVE`, `Spans`/`overlap`): todos
  presentes, equivalência semântica **não medida**.
- `brcly/TuxBox`: `b46c3db9` (#309) fixa a opção do DXC no teste de fallback `-O0`, que desde
  `8a3b6df` contava chamadas conforme a versão instalada; #310 (aberto) retenta 374 recusas de IDXEN
  na preparação offline (área de D-12).
- `ivall/xcl`: a `main` ganha perfis de teclado/mouse por título (#1824, `81693f02`); #3023 em
  `amd-dev` corrige `V_CMP_LG_F32` no tradutor direto RDNA4 — mesma classe de `VIctory D-20`.

## Falhas e desvios

- **A "tabela de pistas" citada no passo 2 continua inexistente.** `docs/automacao/ANALISE_HORARIA.md`
  cita "arquivo de área listada na tabela de pistas" na linha 231, mas nenhuma tabela existe no
  procedimento nem em outro arquivo do acervo. As áreas do sinal foram tomadas das seções D de
  `catalogo/defeitos.md`, como nas duas horas anteriores; o critério 3 ficou subsumido pelo 2.
- **PRs abertos documentados pelo corpo, não pelo patch.** Os 39 foram lidos por título, corpo (até
  ~1.100 caracteres) e lista de arquivos; nenhum patch de PR aberto foi lido linha a linha. Onde o
  corpo não foi lido (`victory-fork` #1475 a #1483), a tabela diz "corpo e arquivos não lidos".
- **A coleta de mesclados não via `gh pr list --search`.** Usei `pulls?state=closed&sort=updated`,
  paginando até o `updated_at` mais antigo cair abaixo de `last_run`, como o estado manda. Em
  `Xbox2Windows`, 4 merges contra 11 commits: os outros 7 citam número de PR no assunto mas têm
  `merged_at` anterior ao cursor das 13:55:58 — são os que a execução das 17:00 deixou acima do teto,
  não uma divergência de coleta. Registrado na seção Confiança do documento.
- **`victory-fork` sem merge e com `last_run` parado.** Não houve merge na janela; o cursor de
  mesclados ficou em `2026-11-27T13:05:00Z` de propósito.
- **Sem subagentes.** A ferramenta de subagente não existe neste ambiente; os quatro repositórios
  foram processados em sequência. O limite de 4 deep dives foi respeitado.
- **Rate limit**: `gh api rate_limit` marcava 4.997 de 5.000 no início; nenhuma chamada devolveu 403.
- Nenhum `_falhas.md`: não houve erro de coleta ou de escrita em repositório monitorado. Nenhuma
  escrita fora de `G:\gta6-port-knowledge`.

## Estado gravado

`runs: 16`, `updated: 2026-11-27T18:19:18Z`, `last_error: null`. Cursores que avançaram:

| Repositório | `last_commit` | `last_pr_number` | `last_run` |
|---|---|---|---|
| `R0Y00/Xbox2Windows` | `de1595b9` (cursor e head; `commit_cursor`/`head_at_run` removidos) | 3097 | 2026-11-27T15:07:31Z |
| `brcly/TuxBox` | `cf0a0824` | 311 | 2026-11-27T14:11:02Z |
| `ivall/xcl` | `81693f02` | 3032 | 2026-11-27T14:21:57Z |
| `fxceless/victory-fork` | `5dbf5ce2` | 1483 | inalterado (sem merge) |

Os outros 10 mantêm cursor de commit e de merge. `backlog` com 5: `Belobug/VIctory-Belobug-AMD`,
`avatarmanz/VIctory-7900XTX`, `Marvisak/VIctory`, `holdmysocks/SixNG`, `omonkeyz/xbox-emulator`. A
ordem da próxima hora está no `backlog_note`. Também corrigi o `watch.json`, que tinha **duas chaves
`backlog_note`** (a das 16:00 e a das 17:00 no mesmo objeto); ficou uma só, a desta execução, e as
pendências estruturais antigas que ainda valem foram reescritas nela.

## Push

- Commit na `public`: `f0f0141` — `docs(updates): 2026-11-27 18:00` (8 arquivos, 516 inserções, 171
  remoções: os 4 documentos, `_triagem.md`, `docs/updates/INDEX.md`, `state/watch.json` e
  `state/last-run.md`).
- `git push origin public:main`: **sucesso** — `d546e96..f0f0141  public -> main`; `origin/main` =
  `f0f0141e5cac5d36cbf9a04ac57c677dff2b5af0`.
- `git switch main` + `git merge --no-ff public`: **sucesso** — `3379744..dabe290`; `git push private
  main`: **sucesso** — `3379744..dabe290  main -> main`, `private/main` =
  `dabe290` (merge `merge updates 2026-11-27 18:00`).
- Este fechamento (registro do push) é um segundo commit na `public`, depois do merge; para não
  criar um segundo merge na execução (o passo 5 limita a um), a `main` recebeu esse commit por
  fast-forward de `public`, não por um segundo `--no-ff`.
- Nenhum `git reset --hard`, nenhum merge de `main` na `public`: a `public` só recebeu commits próprios
  e a `main` recebeu a `public`.
