# R0Y00/Xbox2Windows — atualização de 2026-11-28 17:00

## Resumo

Hora de PRs: dois commits apenas, um deles o merge `#3332`, que faz os launchers (`run.ps1`, `run.sh`)
passarem `--suspend-others 0` ao depurador de startup — hoje é impossível desligar, pela linha de
comando, a retenção de todas as outras threads enquanto um hook dá um passo, e ela custa 3–4× no load
depois das páginas de primeira execução. Os três PRs abertos atacam o present multi-plano (`#3376`),
o crescimento sem limite dos registros de view (`#3377`) e um preset compartilhado de execução
(`#3378`).

## Commits novos (2)

Janela: `compare(f5a6f6ba...aec2e084)` = 2 commits, `status: ahead`, `behind_by: 0`. Ambos
documentados.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `539c3ad5` | lukaslw-lab | startup: let run.ps1 and run.sh turn off the hook thread holding | `run.ps1` ganha `[ValidateSet(0, 1)] [int]$SuspendOthers = 1` e repassa `--suspend-others N`; `run.sh` valida 0/1 (exit 64 fora disso), atualiza o `usage()` e repassa. Testes: `startup_run_ps1_check.py` +3 e `startup_run_sh_check.py` +5 cobrem 0 e 1. Docs: `startup-runtime.md` +7 e uma linha corrigida em `experiments.md`. +28/−6, 6 arquivos. |
| `aec2e084` | John Connor | Merge pull request #3332 from lukaslw-lab/startup/run-suspend-others | Merge; conteúdo em `539c3ad5`. |

## Pull requests abertos (3)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #3376 | runtime: compose presented planes by default | Um present do perfil Series S pode carregar dois planos: plano 0 a 1920×1080 preto e plano 1 a 3840×2160 com a imagem; a janela mostrava o plano 0 e ficava preta. O diff faz o present multi-plano compor todos os planos desenháveis por padrão, mantém `X2W_PRESENT_PLANE=<n>` como escolha diagnóstica (vence o default, e `n` além da contagem volta ao plano 0), deixa present de um plano igual a antes e transforma `X2W_PRESENT_COMPOSITE=1` em forçar o caminho de todos os planos. Também fecha a lacuna do `#3373`: trocar só a seleção não bastava, `present_to_windows` também precisava. +90/−40, 6 arquivos, com controle do teste (`0x40`-byte stride) atualizado. | Mesma área da tela preta de `#3373` (documentado às 16:00), que não tem linha no catálogo; nenhum D-xx cita planos apresentados. |
| #3377 | startup: opt-in content view tokens so view records can be freed | Ref. #1199 (um registro por view distinta, nunca removido). Sob `X2W_D3D12_VIEW_CONTENT_TOKENS=1`, `note_view` grava no descritor do título um token com a **própria view** (CBV: endereço e tamanho; SRV: recurso, `FirstElement`, `NumElements`, stride, formato, flags, component mapping) em vez de um id de registro; assim um registro só nomeado por shadow slots pode ser liberado, porque a cópia com `memcpy` do título continua resolvendo. Unset, nada muda. +859/−49, 10 arquivos (inclui `view_content_token.h`, 113 linhas, e dois testes novos). | Não. Área de heap/registro de view; o catálogo trata de view heap em `D-09` (`startup_view_heap` lendo trap log sem compartilhamento), que é outro mecanismo. Candidato a linha nova (#1199). |
| #3378 | startup: run a package with shared presets (console, OS, GPU) | `tools/startup/run_preset.py` (296 linhas) com três escolhas — console `x`/`s`, GPU `nvidia`/`amd`, OS `win11`/`win10`/`linux` — e todo o resto em `run_presets.json` (70 linhas) aplicado em camadas base → OS → console → GPU, cada camada trazendo `why` e a execução medida. Valida que cada camada nomeia só switches que `run.ps1`/`run.sh` realmente têm; staging só da pasta do executável. +640, 7 arquivos, com `tests/startup_run_preset_check.py` e entrada em `AGENTS.md`. | Não (ferramenta). |

## Pull requests mesclados (1)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #3332 | startup: let run.ps1 and run.sh turn off the hook thread holding | `aec2e084` (16:13:45Z) | Conserta a issue #3331: `--suspend-others 0` já existia em `debug_launch.py` desde o #1408, mas os launchers não o expunham. Entrou o parâmetro `-SuspendOthers`/`--suspend-others` com validação 0/1, passagem adiante, testes nos dois shells e documentação. O default continua 1 (reter as threads). Corresponde ao commit `539c3ad5` (+28/−6, 6 arquivos). |

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum por hash: os 2 hashes da janela e o merge (`aec2e084`) não têm interseção
  com os 270 tokens de `catalogo/defeitos.csv`.
- **Toca área de D-xx?** `#3332`/`539c3ad5` mexe no depurador de startup e em como os hooks retêm
  threads — mesma região de **D-14** ("call hooks do debugger armados por padrão") e do par
  `unprotected`/`#1281` citado na doc. O diff **não** muda o que é armado por padrão; só permite
  desligar a retenção pela linha de comando. `#3376` e `#3377` não correspondem a nenhuma linha do
  catálogo deste repositório.
- **Abre defeito novo?** Não verificado. Dois candidatos a linha nova: (1) present multi-plano com
  plano 0 preto — `#3373`/`#3376`, arquivo `src/startup_stubs/present_planes.h`; (2) registros de view
  que crescem sem limite (#1199), arquivo `src/startup_stubs/d3d12_capture.inl`. Nenhuma das duas tem
  hash no catálogo.

## Confiança

**Verificado por diff.** Os 2 commits foram lidos pelo patch de `repos/R0Y00/Xbox2Windows/commits/<sha>`.
Os 4 PRs (`#3332`, `#3376`, `#3377`, `#3378`) foram lidos por corpo e `pulls/<n>/files`. O diff do
`#3376` foi conferido linha a linha nos pontos de seleção de plano (`present_planes.h`,
`d3d12_present.inl`, `d3d12_xs_device.cpp`).

**Não verificado.** Nada foi compilado nem executado; nenhum teste do repositório foi rodado aqui. A
afirmação do `#3376` de que "fecha a lacuna do #3373" e a do `#3332` de que a retenção custa 3–4× no
load são do corpo dos PRs, não medições reproduzidas. Não abri o `#3373` nesta hora (foi documentado às
16:00) para comparar hunk a hunk com o `#3376`.

**Estado.** `last_commit` `aec2e084`; `last_run` `2026-11-28T16:13:45Z`; `last_pr_number` 3378. Sem
`commit_cursor` pendente.
