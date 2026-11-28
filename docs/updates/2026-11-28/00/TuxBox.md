# brcly/TuxBox — atualização de 2026-11-28 00:00

## Resumo

Dois commits acima do cursor `c05ce004`, os dois de `brcly` no mesmo segundo (20:56:48Z) e empurrados
direto na `main` — nenhum PR mesclado na janela. É trabalho de **verificação**: um harness que compara uma
mudança do tradutor com uma linha de base sobre os shaders gravados do título, e um roadmap que torna
explícito o buraco que o acervo vem cobrando — nenhum shader real é executado e comparado com referência,
e 178 das 372 instruções que os shaders gravados usam não têm check de GPU.

## Commits novos (2)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `b102354950` | brcly | tools: add a harness that tests translator changes over recorded shaders | 9 arquivos, +774/−2. `tools/shader_harness.py` (+361) ganha seis estágios: **code checks** (`tuxbox.py check --no-gpu`), **survey** (traduz cada request com as duas builds; compute é traduzido duas vezes, para host de 64 e de 32 lanes), **determinism** (traduz uma amostra de novo e compara saída e resultado), **compile** (programas alterados e recuperados passam por DXC/DXV; bloca quando um que compilava deixa de compilar), **GPU** (builda, roda os checks do tradutor e as suítes Lanes em cada placa) e **coverage** (lista instruções usadas que nenhum programa de check cobre e as maiores recusas; nunca bloqueia). Configuração em `build/shader-harness.json` (ignorado pelo Git), relatório em `build/shader-harness/<run>/report.md` + `report.json`, sem bytes de shader nem HLSL. Opções `--baseline REF`, `--identical`, `--no-gpu`/`--allow-no-gpu`, `--compile-limit 0`. `tools/shader-compiler/src/hlsl/coverage.rs` (+17) registra, com `TUXBOX_XSHADER_COVERAGE` apontando um arquivo, uma linha por instrução de cada programa emitido (uma escrita só, para não partir linha com traduções concorrentes); `emitter/control_flow.rs` (+7) separa `emit_program` de `emit_body` para chamar o registro sem mudar a tradução. `project_tuxbox/shader_survey.py` (+204) e `tests/test_shader_harness.py` (+127) acompanham |
| `62c7e390242d` | brcly | docs: add a roadmap for shader translation and the project audit | Só documento, 3 arquivos +143/−1: `docs/ROADMAP.md` (+140), com um `README.md` +2 e um ajuste em `docs/FOCUS.md`. Fixa os números de 2026-11-27 na `main` sobre os shaders gravados de um título (ProjectScarlett 1.0.2.1): **38.724 de 42.859** shaders traduzem em host de 64 lanes; **6.075 de 7.683** compute traduzem em host de 32 lanes; ~1.090 instruções nomeadas pelo decodificador, **372 usadas** pelos shaders gravados e **194 rodadas por algum check de GPU**; até 575 nomes do decodificador que não aparecem no tradutor (busca textual, a ser substituída por lista exata na tarefa T1); **nenhum** shader real executado e comparado com referência; quase todo o tempo de traduzir-e-compilar é DXC, com o tradutor em ~30 ms por shader; HLSL mediano de 97 KB, 1 em 100 acima de 1,8 MB |

## O que o harness deliberadamente não cobre

O próprio `docs/SHADER_HARNESS.md` diz, em "What a pass does not show": os shaders gravados são traduzidos
e compilados, **não executados** — uma saída alterada que compila ainda pode calcular errado; os checks de
GPU usam programas sintéticos, e ter uma instrução num check não prova todas as formas de operando nem
todos os estados de lane; o harness não roda nem mede a compilação de pipeline dos drivers.

## Pull requests abertos (3)

Acima do cursor `last_pr_number` 325; título e corpo lidos, patch não lido.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| `#328` | xshader: read a heap descriptor through the copy it was loaded from, one wave at a time | `CS_ProjDecalResolve_Scalarized` carrega descritor de textura em índice de heap calculado em tempo de execução e o lê de volta com `v_readfirstlane` num laço waterfall; o dataflow perde os oito words na cópia e a amostra é recusada (`s12 not an image descriptor`). Propõe provar, por caminho de volta, que o primeiro write do registrador é a cópia e que o EXEC só estreitou desde então (`hlsl/descriptor_broadcast.rs`), em forma empilhada sobre `#327` | Área de **D-22** (root raster com várias tabelas) e do `#232` citado; mecanismo diferente — não é D-22 |
| `#327` | xshader: keep a loop's swizzles in the two-lane form for whole console waves | `ds_swizzle_b32` dentro de laço era recusado ("compute swizzle inside a loop") a menos que o grupo fosse uma wave de 32 lanes com volta uniforme; a forma de duas lanes por thread não tem esse limite e passa a ser usada quando as formas de uma lane recusam, o grupo contém waves inteiras de 64 e o programa guarda EXEC em duas palavras | Não |
| `#326` | shader-translatro: Separated actionable from expected refusals | Separa, no recibo de preparação, as 8.756 recusas de assinatura de mesh root das 907 recusas acionáveis, para o recibo mostrar o que precisa de correção por shader em vez do que é esperado na criação do pipeline | Não; é ferramenta de triagem de recusas |

`#327` e `#328` são empilhados (o `#328` declara base `1e5ef81c` do `#327`), ambos referenciando o issue
`#232`. `#326` não foi executado em AMD e NVIDIA segundo o próprio corpo (a caixa da tabela de GPU ficou
desmarcada).

## Pull requests mesclados (0)

Nenhum `merged_at` maior que `last_run` (2026-11-27T20:01:33Z). O `#249` foi mesclado exatamente às
20:01:33Z, que é o cursor — fica fora por não ser estritamente maior, e foi documentado na execução
anterior. Os dois commits desta hora não vieram de PR (`commits/<sha>/pulls` devolve lista vazia nos dois).

## Cruzamento com o catálogo

- **Nenhuma das 29 entradas de `TuxBox` no catálogo é corrigida por estes dois commits.** O harness ataca a
  **classe** que D-24 nomeia (resolução de merge apagou lowering e quebrou o verificador): a partir daqui,
  uma mudança do tradutor que pare de traduzir um shader gravado, que mude a saída de forma não
  determinística ou que quebre a compilação de um programa que compilava é bloqueada antes do merge. Mas o
  harness não é o verificador que D-24 cobra, e nada no diff reaplica o lowering perdido.
- **O roadmap confirma, com número, a dívida estrutural do repositório:** 178 das 372 instruções usadas
  não têm check de GPU e nenhum shader real é executado e comparado com referência. É a mesma lacuna que o
  acervo registra em D-17 de outro repositório (dívida de verificação), sem entrada equivalente em
  `TuxBox`.
- **Candidato a linha nova do catálogo, se o operador quiser:** "cobertura de GPU incompleta no tradutor —
  178/372 instruções usadas sem check e zero shaders reais comparados com referência", medido em
  `62c7e390` e verificável pelo `docs/ROADMAP.md`.
- **Limite operacional do harness:** ele exige Linux, uma pasta de requests gravados do próprio título,
  DXC e Proton, com caminhos num `build/shader-harness.json` que o Git ignora. Nada disso está no
  repositório, então o harness **não roda em CI** — só o teste unitário `tests/test_shader_harness.py`
  (+127) pode rodar. NAO VERIFICADO se algum workflow o invoca; não há workflow no diff.

## Confiança

- **Verificado no diff:** os dois patches foram lidos por inteiro nos arquivos de documento
  (`docs/SHADER_HARNESS.md`, `docs/ROADMAP.md`) e nos de código pequenos (`coverage.rs`,
  `control_flow.rs`, `hlsl.rs`); `tools/shader_harness.py` (+361) e `project_tuxbox/shader_survey.py`
  (+204) foram lidos no patch até o teto de 6.000 caracteres — os estágios vêm da documentação do próprio
  commit, não de cada função.
- **Verificado por API:** os dois commits são pushes diretos (sem PR), não há merge na janela, e os três
  PRs abertos estão acima do cursor.
- **Não verificado:** nenhum shader foi traduzido nem compilado aqui; os números do roadmap (38.724/42.859,
  6.075/7.683, 372/194) são os declarados pelo commit e não foram remedidos. Não foi checado se o harness
  roda em CI. Os três PRs abertos foram lidos por título e corpo, sem patch.
- **Contagem:** 2 commits na faixa, 0 merges. Coerente com push direto.
