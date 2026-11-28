# Execução de 2026-11-28 01:00

## Data e hora

Coleta de 2026-11-28T01:07–01:17 UTC real — ~22:07–22:17 no relógio do GitHub, que segue cerca de 3 h
atrás e é o relógio em que `last_run` é gravado. Diretório de documentos: `docs/updates/2026-11-28/01/`
(UTC). Estado lido: `baselined: true`, `runs: 22` (gravado como 23); 14 entradas em `repos` no início e no
fim, nenhuma removida, nenhuma acrescentada.

## Sanidade do estado (passo 1)

`gh api repos/<o>/<r> --jq .default_branch` foi pedido nos 14. **Dois continuam sem resposta**:
`Belobug/VIctory-Belobug-AMD` e `avatarmanz/VIctory-7900XTX` devolvem HTTP 404 (quinta hora seguida);
a `default_branch` deles não pôde ser reconferida, os cursores ficaram parados e a triagem deles é
NAO VERIFICADO. Detalhe em `docs/updates/2026-11-28/01/_falhas.md`.

Nos outros 12, `default_branch` idêntico ao gravado, inclusive `master` (`holdmysocks/SixNG`) e
`a-publier` (`Belobug`, valor antigo, inverificável agora). Nenhum `last_commit` nulo — não houve
baseline nesta execução. Nenhum `compare` devolveu `diverged` (4 `ahead`, 8 `identical`). Contagem de
`repos` igual no início e no fim (14).

## Triagem (passo 2)

Os 14 foram triados (12 com medição completa); 4 entraram no deep dive. Arquivo:
`docs/updates/2026-11-28/01/_triagem.md`.

| Repositório | Commits atrás | Abertos acima do cursor | Mesclados na janela | Sinal | Deep dive |
|---|---|---|---|---|---|
| `R0Y00/Xbox2Windows` | 29 | 13 (100 no total) | 29 | SIM (critério 2; lowerings e duas correções de runtime) | sim |
| `holdmysocks/SixNG` | 6 | 1 (5 no total) | 6 | SIM (critério 2; máscara de tatuagem, índices de imagem no merge) | sim |
| `brcly/TuxBox` | 5 | 9 (15 no total) | 0 | SIM (critério 2; inventário por instrução achou recusa falsa) | sim |
| `ivall/xcl` | 2 | 25 (100 no total) | 1 | SIM (critério 2; liberação de DXIL offline) | sim |
| `Marvisak/VIctory` | 0 | 81 (100 no total) | 0 | SIM (critério 3; `GPU:`/`Shaders:`/`CLI:`) | não |
| `uiliannn/VItoriaBr` | 0 | 1 (75 no total) | 0 | SIM fraco (critério 3; `#132`) | não |
| `fxceless/victory-fork` | 0 | 2 (100 no total) | 0 | SIM fraco (critério 3; `#1499`, `#1500`) | não |
| `omonkeyz/xbox-emulator` | 0 | 1 (25 no total) | 0 | SIM fraco (critério 3; `#395`) | não |
| `DexNom/VIctory` | 0 | 0 (100 total, máx. 148 = cursor) | 0 | NAO | não |
| `californiaAlmonds/OpenScarlett` | 0 | 0 (1 no total) | 0 | NAO | não |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 | 0 | NAO | não |
| `stellarr-studio/kelvaroq` | 0 | 0 | 0 | NAO | não |
| `Belobug/VIctory-Belobug-AMD` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (404) | bloqueado |
| `avatarmanz/VIctory-7900XTX` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (404) | bloqueado |

Desempate: quatro repositórios tinham commit na janela e o critério 2 ocupou as quatro vagas. O critério
1 (defeito do catálogo) não selecionou ninguém — os 42 hashes foram conferidos contra as 306 linhas do
`catalogo/defeitos.csv` e nenhum aparece nas colunas `introduzido`/`corrigido`. Nenhum repositório passou
de 200 commits de atraso (máximo 29), então o critério 4 não foi acionado.

## Documentos escritos (passo 3)

4 documentos em `docs/updates/2026-11-28/01/`: `Xbox2Windows.md`, `SixNG.md`, `TuxBox.md`, `xcl.md`,
mais `_triagem.md` e `_falhas.md`. **42 commits documentados de 42** atrás dos cursores (29 + 6 + 5 + 2),
**36 PRs mesclados** (29 + 6 + 1) e **133 PRs abertos lidos** por título e corpo. Nenhum repositório
chegou ao teto de 40 commits.

Achados principais:

- `R0Y00/Xbox2Windows` (`cf3df832` → `886a5a28`): 20 dos 29 commits fecham recusas do compute direto
  (`vcc_hi` como máscara SDWA, atômicos por janela de bytes, sampler heap em índice fixo, SCC neutro,
  EXEC como destino de `s_bitreplicate`, `s_mov_b32 exec_hi`, `v_pk_add/mul/fma_f16`,
  `v_pk_min/max_{i16,u16,f16}`, `vcc_lo/hi` como offset SMEM, GLOBAL `x2/x3/x4`, spill de descritor por
  lane, SDWA pkrtz/add-co-ci, VOPC de 64 bits). `afe70bd7` (#3190) **fecha o defeito N1 da hora
  anterior**: define `SENTINEL` em `tests/native_f16_alu_check.py`, o símbolo que
  `tests/native_cvt_f16_i16_check.py` importa como `F.SENTINEL` desde `98020080`.
- `R0Y00/Xbox2Windows`, promessa maior que o diff: `23b6f7b6` (#3141) tem título "trace UI texture
  row-pitch bands" e o diff não amostra pitch nenhum — são `full_mips()` para `MipLevels == 0` em
  `xg_xs.cpp` (duas funções), um `mark_gpu_written` na cópia de sub-recurso e o teste de 1.402 linhas.
  A correção das bandas de `#3132` é a hipótese do teste, não algo que o diff prove.
- `R0Y00/Xbox2Windows`, defeito dentro do PR: o squash `6cc46a3c` (#1408) carrega três correções de
  defeitos que o próprio PR introduziu (`returns.get` sobre `ReturnBook`, segunda atribuição de
  `file_log` quebrando `X2W_LOG_AT`, retorno tail-called). Não chegaram à `main` porque o squash é um
  commit, mas são candidatos a linha do catálogo sobre integração de PR.
- `brcly/TuxBox` (`aa1f4028`): o inventário de 1.398 encodings achou um defeito **vivo na `main`** —
  toda SMEM que não fosse carga de descritor era tratada como `s_buffer_load_*`, e
  `s_scratch_load_*`/`s_get_waveid_in_workgroup` estouravam o cálculo de tamanho. Os 65 checks de GPU
  novos levam a cobertura de 188 para **241 das 372 instruções usadas**; o roadmap segue admitindo zero
  shaders reais executados contra referência (T4/T5).
- `holdmysocks/SixNG` (`66f606ac`, #508): `glb::merge` deixava `speculartex2`, `normal` e `control` do
  `skin_detail` — e `wear_mask`/`primer_mask`/`dust_mask` — nos índices de imagem da primeira parte;
  era isso que produzia manchas planas e costura de pescoço em Lucia e Connie. `#507` pareia camada e
  máscara de tatuagem e sobe `BAKE_RULE` de 8 para 9; `#509` acrescenta adesivo plano que o próprio doc
  diz que o jogo não tem; `58665d04` renomeia a chave `tile_texture_e0548ad3` para
  `surface_map_texture` no `inland.json`.
- `ivall/xcl` (`d8dae08d`): os jobs de pipeline offline passam a ser consumidos por valor, então o DXIL
  de um job é liberado assim que o build daquele job retorna; merge real `2c717a04` (não squash), o que
  explica 2 commits para 1 merge. `#3053` e `#3054`, abertos, têm corpo idêntico.

## Falhas e desvios

- **Dois repositórios inacessíveis (404, quinta hora).** Registrado em `_falhas.md`. Não avançaram
  cursor, não foram documentados, não saíram de `repos`.
- **A "tabela de pistas" citada no passo 2 continua inexistente** em
  `docs/automacao/ANALISE_HORARIA.md` (o `INDEX.md` já registra o desvio desde as 17:00). As áreas do
  sinal vieram das seções D do `catalogo/defeitos.md` e do assunto de cada PR; o sinal dos repositórios
  sem commit é por título/corpo de PR, não por patch.
- **Contagem de commits × merges:** bate nos quatro documentados. `Xbox2Windows` 29 = 29 (squash de um
  pai), `SixNG` 6 = 6, `xcl` 2 commits para 1 merge (merge real de dois pais), `TuxBox` 5 commits e 0
  merges (push direto; `commits/<sha>/pulls` vazio nos cinco).
- **PRs mesclados lidos por API REST, não pela busca.** `merged:>=` continua sem uso; os 36 merges vieram
  de `pulls?state=closed&sort=updated`, paginado até `updated_at` cair abaixo do cursor.
- **PRs abertos documentados pelo título e pelo corpo, não pelo patch.** 133 corpos lidos; nenhum patch
  de PR aberto foi lido.
- **Sem subagentes.** A ferramenta não existe neste ambiente; os quatro repositórios foram processados
  em sequência e o limite de 4 deep dives foi respeitado.
- **Rate limit:** 4.833 restantes no início, nenhum 403, nenhuma repetição em laço.
- **Escrita:** somente em `G:\gta6-port-knowledge`. Os 14 monitorados foram só leitura de API.
- **Arquivos de terceiros no diretório de trabalho** (`docs/analises/victory-fork.md` modificado e dois
  relatórios não rastreados em `docs/relatorios/`) ficaram fora do `git add` desta rotina e intactos.

## Estado gravado

`runs: 23`, `updated: 2026-11-28T01:17:11Z`, `last_error` com os dois 404 (quinta hora). Cursores que
avançaram:

| Repositório | `last_commit` | `last_pr_number` | `last_run` |
|---|---|---|---|
| `R0Y00/Xbox2Windows` | `886a5a28` (head) | 3215 | 2026-11-27T22:07:09Z |
| `holdmysocks/SixNG` | `b42d09a9` (head) | 510 | 2026-11-27T22:03:13Z |
| `brcly/TuxBox` | `1d0f3fa9` (head) | 339 | 2026-11-27T20:01:33Z (sem merge) |
| `ivall/xcl` | `2c717a04` (head) | 3075 | 2026-11-27T21:27:54Z |

`last_run` dos que tiveram merge é o maior `merged_at` documentado, no relógio do GitHub — não o horário
da execução, para não esconder merge entre o fim da coleta e a próxima hora. Os outros 10 mantêm cursor de
commit e de merge. `backlog` com 6: os dois bloqueados e `Marvisak/VIctory`, `uiliannn/VItoriaBr`,
`fxceless/victory-fork`, `omonkeyz/xbox-emulator`; a ordem e os achados estão no `backlog_note`.

## Push

- Commit na `public`: `148285b` — `docs(updates): 2026-11-28 01:00` (9 arquivos, 607 inserções, 106
  remoções: os 4 documentos, `_triagem.md`, `_falhas.md`, `docs/updates/INDEX.md`, `state/watch.json` e
  `state/last-run.md`).
- `git push origin public:main`: **sucesso** — `993b54e..148285b  public -> main`.
- `git switch main` + `git merge --no-ff public`: **conflito em `state/last-run.md`** (a `main` tinha o
  registro de push da execução das 00:00, cherry-pickado; a `public` trazia o arquivo novo). Resolvido
  tomando a versão da `public` (`git checkout --theirs`) e conferindo o resto do merge: os outros 8
  arquivos entraram sem conflito. Merge `cbfee23` — `merge updates 2026-11-28 01:00`.
- `git push private main`: **sucesso** — `5d98b01..cbfee23  main -> main`.
- Este fechamento (registro do push) é um segundo commit na `public`, depois do merge. Para não criar um
  segundo merge na execução (o passo 5 limita a um), a `main` recebe esse commit por cherry-pick, como
  nas execuções das 18:00, 19:00, 20:00, 21:00, 22:00, 23:00 e 00:00.
- Nenhum `git reset --hard`, nenhum merge de `main` na `public`: a `public` só recebeu commits próprios e
  a `main` recebeu a `public`.
- Ficaram fora do commit, intactos: `docs/analises/victory-fork.md` modificado e os dois relatórios não
  rastreados em `docs/relatorios/`, que não são desta rotina.
