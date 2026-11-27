# Execução de 2026-11-27 20:00

## Data e hora

Coleta de 2026-11-27T20:07–20:21 UTC real — 17:07–17:21 no relógio do GitHub, que segue 3 h atrás e
é o relógio em que `last_run` é gravado. Diretório de documentos: `docs/updates/2026-11-27/20/`
(UTC). Estado lido: `baselined: true`, `runs: 17` (gravado como 18); 14 entradas em `repos` no
início e no fim, nenhuma removida.

## Sanidade do estado (passo 1)

`gh api repos/<o>/<r> --jq .default_branch` nos 14: idêntico ao gravado em todos, inclusive
`a-publier` (`Belobug/VIctory-Belobug-AMD`) e `master` (`holdmysocks/SixNG`). Nenhum `last_commit`
nulo — não houve baseline nesta execução. Nenhum `compare` devolveu `diverged`. Contagem de `repos`
igual no início e no fim (14); nada foi apagado nem acrescentado.

## Triagem (passo 2)

Os 14 foram triados; 4 entraram no deep dive. Arquivo: `docs/updates/2026-11-27/20/_triagem.md`.

| Repositório | Commits atrás | Abertos acima do cursor | Mesclados na janela | Sinal | Deep dive |
|---|---|---|---|---|---|
| `R0Y00/Xbox2Windows` | 37 | 10 (115 no total) | 3 | SIM (correção; variante de D-02) | sim |
| `brcly/TuxBox` | 6 | 1 | 1 | SIM (correção em tradução de shader) | sim |
| `holdmysocks/SixNG` | 4 | 1 (#476) | 4 | SIM (correção) | sim |
| `ivall/xcl` | 2 | 3 | 1 | SIM (correção em `xcl shaders`) | sim |
| `Marvisak/VIctory` | 0 | 26 (1.663 no total) | 0 | SIM (PRs `fix` em GPU/shader) | não |
| `Belobug/VIctory-Belobug-AMD` | 0 | ≥100 (374 no total) | 0 | SIM | não |
| `avatarmanz/VIctory-7900XTX` | 0 | 43 (240 no total) | 0 | SIM | não |
| `fxceless/victory-fork` | 0 | 2 (946 no total) | 0 | SIM fraco | não |
| `omonkeyz/xbox-emulator` | 0 | 1 (#395) | 0 | SIM | não |
| `DexNom/VIctory` | 0 | 1 (#135) | 0 | SIM fraco | não |
| `uiliannn/VItoriaBr` | 0 | 1 (#132) | 0 | SIM fraco | não |
| `californiaAlmonds/OpenScarlett` | 0 | 0 | 0 | NAO | não |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 | 0 | NAO | não |
| `stellarr-studio/kelvaroq` | 0 | 0 | 0 | NAO | não |

Desempate: o procedimento põe o critério 2 (commit com sinal de correção/regressão/reversão) acima do
critério 3 (PR aberto em área de defeito), e nesta hora quatro repositórios tiveram commit na janela
com esse sinal. `Belobug`, `avatarmanz` e `Marvisak` seguem com sinal apenas de PR aberto e ficaram
fora pelo terceiro ciclo; o passivo medido é 374, 240 e 1.663.

## Documentos escritos (passo 3)

4 documentos em `docs/updates/2026-11-27/20/`: `Xbox2Windows.md`, `TuxBox.md`, `SixNG.md`, `xcl.md`,
mais `_triagem.md`. **49 commits documentados de 49** atrás dos cursores (37 + 6 + 4 + 2), **9 PRs
mesclados** (3 + 1 + 4 + 1) e **15 PRs abertos** (10 + 1 + 1 + 3). Nenhum repositório chegou ao teto de
40 commits.

Achados principais:

- `R0Y00/Xbox2Windows`: os 30 commits do #2911 (`63a30dc1`) trocam, falha por falha, o custo das page
  runs do XMem sob Wine — split do placeholder antes do map (`06dfe20e`), `stable_call` para leitores
  que não geram fault (`be3f918c`, `e6e7d9b9`), retry de fault por stamp do break substituindo o anel
  de 64 (`c8391709`), join entre chamadas atrás de `X2W_XMEM_JOIN_CALLS` com teto de 64 páginas
  (`a4dc3f9c`, `d4f1344e`), retry de `ReadFile`/`WriteFile` só em arquivo de disco (`4f7d47de`).
  **Achado novo:** o fallback para SO que esconde o bit `0x100` do placeholder (`17ee827d` +
  `1615efc5`) cria uma variante de D-02 restrita a Wine, registrada como limite em `f9391cca` — não
  distingue página de alocação privada com `MEM_REPLACE_PLACEHOLDER` e, após view recusada, corta a
  página da alocação do chamador. O #3130 foi mesclado em `codex/native-graphics-integration`
  (`b80513fd`), não na `main`; o #3114 (`2641cc72`) leva `s_quadmask_b32`/`_b64` ao lowering direto.
- `brcly/TuxBox`: `049e5a43` e `65e3e48c` reescrevem `tb_float_relation` e `tb_f16_cvt` sem branches e
  refixam as 48 linhas de `host_profiles.txt` — todo cache de tradução invalidado, e a equivalência é
  medição do autor, sem teste de referência no repositório; `55ec224a` passa a reportar como `SKIP` a
  checagem de splash onde a GPU apresenta por software (primeiro frame não completa, causa
  desconhecida); `7e2c2408` impede o script do devcontainer de matar Xwayland do host. O #312 aberto lê
  root SRV de compute depois das tabelas — área de D-22, não D-22.
- `holdmysocks/SixNG`: quatro squash, dois mesclados durante a coleta (#477 às 17:10:02, #479 às
  17:14:27). A ordem dos índices do buffer compartilhado passa a ser por tipo e material com junção de
  draws (#475, #477); o clique do painel de liveries passa a testar o raio contra a malha posada pelas
  juntas (#478); novo módulo `jawkeys` dá aos dentes e à língua o movimento da mandíbula, com limiares
  heurísticos de 30% de resíduo e 1 mm (#479).
- `ivall/xcl`: `#880` (`6576d75b`) faz a estatística e o dump de `xcl shaders` tirarem o tamanho de
  wave do cabeçalho FSSM, como os pipelines; só 2 arquivos entraram porque o PR está empilhado em #573.
  Os abertos #3040/#3042 declaram-se portes de `Marvisak/VIctory 63bcca4` — o cursor atual de um
  repositório monitorado — e o #3039 aponta para `nvidia-dev`.

## Falhas e desvios

- **A "tabela de pistas" citada no passo 2 continua inexistente** em
  `docs/automacao/ANALISE_HORARIA.md`. As áreas do sinal vieram das seções D de
  `catalogo/defeitos.md`, e o sinal dessas linhas é por título/corpo de PR, não por patch. O deep dive
  foi decidido por diff.
- **Coleta que mudou durante a execução.** `holdmysocks/SixNG` recebeu dois merges entre a primeira
  coleta e a reconferência (#477 e #479). O `compare` foi refeito até o head novo `67bcc0ec`, os dois
  commits entraram no documento e a reconferência de mesclados fechou em 4. Sem isso, o cursor teria
  ficado dois commits atrás.
- **Contagem de merges contra a de commits no `Xbox2Windows`**: 3 merges contra 37 commits, porque 30
  commits são de um único PR (#2911), um é squash (#3114) e um merge (#3130) nem está na `main`. A
  divergência está registrada na seção Confiança do documento.
- **PRs abertos documentados pelo corpo, não pelo patch.** 15 PRs lidos por título, corpo, base e
  contagem de arquivos; nenhum patch de PR aberto foi lido linha a linha.
- **Sem subagentes.** A ferramenta de subagente não existe neste ambiente; os quatro repositórios
  foram processados em sequência, e o limite de 4 deep dives foi respeitado.
- **Rate limit**: `gh api rate_limit` marcava 4.999 de 5.000 no início e 4.794 depois da coleta;
  nenhuma chamada devolveu 403. `search/issues` funcionou para contar abertos; `ivall/xcl` ficou sem
  total (HTTP 422 na execução das 19:00, não pedido de novo).
- Nenhum `_falhas.md`: não houve erro de coleta ou de escrita. Nenhuma escrita fora de
  `G:\gta6-port-knowledge`; os 14 monitorados foram só leitura de API.

## Estado gravado

`runs: 18`, `updated: 2026-11-27T20:21:00Z`, `last_error: null`. Cursores que avançaram:

| Repositório | `last_commit` | `last_pr_number` | `last_run` |
|---|---|---|---|
| `R0Y00/Xbox2Windows` | `63a30dc1` (head) | 3139 | 2026-11-27T17:20:00Z |
| `brcly/TuxBox` | `7953f0a5` (head) | 312 | 2026-11-27T17:20:00Z |
| `holdmysocks/SixNG` | `67bcc0ec` (head) | 479 | 2026-11-27T17:20:00Z |
| `ivall/xcl` | `6089122a` (head) | 3042 | 2026-11-27T17:20:00Z |

Os outros 10 mantêm cursor de commit e de merge. `backlog` com 7: `Belobug/VIctory-Belobug-AMD`,
`avatarmanz/VIctory-7900XTX`, `Marvisak/VIctory`, `fxceless/victory-fork`, `omonkeyz/xbox-emulator`,
`DexNom/VIctory`, `uiliannn/VItoriaBr` — `SixNG` foi consumido e `victory-fork` entrou por ter 2 PRs
acima do cursor e 946 abertos no total. A ordem da próxima hora está no `backlog_note`.

## Push

- Commit na `public`: `63b3104` — `docs(updates): 2026-11-27 20:00` (8 arquivos, 519 inserções, 94
  remoções: os 4 documentos, `_triagem.md`, `docs/updates/INDEX.md`, `state/watch.json` e
  `state/last-run.md`).
- `git push origin public:main`: **sucesso** — `70fb975..63b3104  public -> main`.
- `git switch main` + `git merge --no-ff public`: **conflito em `state/last-run.md`** (a `main`
  carregava o registro do push das 19:00 em `3489092`, a `public` trazia o registro desta hora).
  Resolvido a favor da `public` (`git checkout --theirs`), porque o registro desta execução substitui o
  da anterior; merge `f018882` — `merge updates 2026-11-27 20:00`.
- `git push private main`: **sucesso** — `3489092..f018882  main -> main`.
- Este fechamento (registro do push) é um segundo commit na `public`, depois do merge. Para não criar um
  segundo merge na execução (o passo 5 limita a um), a `main` recebe esse commit por cherry-pick, como
  nas execuções das 18:00 e 19:00.
- Nenhum `git reset --hard`, nenhum merge de `main` na `public`: a `public` só recebeu commits próprios
  e a `main` recebeu a `public`.
- Ficaram fora do commit, intactos: `docs/analises/victory-fork.md` modificado e dois relatórios não
  rastreados em `docs/relatorios/`, que não são desta rotina.
