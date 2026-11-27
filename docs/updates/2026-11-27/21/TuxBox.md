# brcly/TuxBox — atualização de 2026-11-27 21:00

## Resumo

Quatro commits entre o cursor `7953f0a5` e o head `89fb2517`. Dois deles são a branch do #312
(`f5233880`, `49fad5f7`), que o merge `8c082e65` traz para a `main`: o tradutor DXBC passa a reconhecer
um root SRV de **compute** colocado depois das tabelas de descritor — antes a regra só valia para
estágio raster com mais de uma tabela. O quarto, `89fb2517`, é um commit direto de 12 arquivos que
troca o caminho de compilação de shader por um helper DXC próprio, rodando os estágios do compilador com
o passe `correlated-propagation` removido: mede 51 s contra 60 s numa compilação fria de 1.500
requisições, 16 em paralelo. Oito PRs abertos acima de #312, todos de `xshader` ou `d3d12`, nenhum
mesclado nesta janela além do #312.

## Commits novos (4)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `f5233880` | Nick Stallman | dxbc: confirm a compute root SRV after the tables | `runtime/dxbc/src/compute_table.rs` +6/-4. A condição que marcava um parâmetro como root SRV era `multi_table.is_some() && class == 0x100` — só raster com várias tabelas. Passa a `root_srv = (multi_table.is_some() \|\| stage == 5 && root.is_some()) && class == 0x100`, e o braço `(0x100, _) if root_srv && location & 0xff == 4 && shape == 1` devolve `(ROOT_SRV, 4)`. Isto é: um root de **compute** (estágio 5) com root signature estendida pode carregar um root SRV depois das tabelas, e o registro do programa nomeia esse parâmetro como os estágios raster nomeiam o deles. `tests/compute_table.rs` +18: `compute_root_srv_carries_the_shader_register` monta o root de três tabelas + root SRV, põe o endereço do parâmetro 3 em `s7-s8` e o shader declarando `t5 space7`, e espera `Binding { table: 3, kind: ROOT_SRV, register: 5, space: 7, slot: 0 }`. Também fixa o **fail-closed**: sem a linha de user-data, com o parâmetro 3 declarado como `Constants`, ou em chamador root-less (`bindings(&srv, 5)`), a resposta é `None` |
| `49fad5f7` | Nick Stallman | lanes: self-test a compute root with a root SRV after its tables | `examples/lanes/selftest.cpp` +111: cenário `compute-root-srv` — root com três tabelas (UAV no slot 0 da tabela 0, **SRV** de 4 no espaço 1 da tabela 1, UAV no slot 0 da tabela 3) mais `ROOT_PARAMETER_TYPE_SRV` no parâmetro 3 declarado `{ShaderRegister: 1, RegisterSpace: 3}` (o shader o nomeia `t5 space7`). O programa do fixture grava no slot 2 da tabela 1 (view 10), onde o root do título declara um SRV; 16 views de 64 palavras começam com um decoy, e o teste exige que **só** a view 10 mude e que as palavras de guarda fiquem intactas — se a view 10 mantiver o decoy, a mensagem acusa "stand-in pipeline or a skipped dispatch". `examples/lanes/shader_fixture.py` +20 gera `root_srv_shader` (PSV com `t5 space7` como raw buffer e a linha de user-data do root SRV); `project_tuxbox/verify.py` refatora a suíte de root refletido em `lanes_reflected_compute_root(..., slots, heap_limit)` e registra `lanes-compute-root-srv` em `BASE_SUITES`; `tests/test_lanes_compute_root.py` +14 confere as duas linhas de user-data e a tabela PSV |
| `8c082e65` | brcly | Merge pull request #312 from NickStallman/d3d12/compute-root-srv-after-tables | Merge de 2 pais (`7953f0a5` = cursor, `49fad5f7`). Contra o primeiro pai: os 7 arquivos da branch, +212/-13 — exatamente `f5233880` + `49fad5f7`, sem conteúdo adicional |
| `89fb2517` | brcly | shaders: compile through a DXC helper that skips value propagation | 12 arquivos, +564/-106. `tools/dxc_worker.cpp` vira um compilador de verdade: recebe quatro linhas (HLSL, saída, perfil, flags), roda o front end com `-fcgl`, obtém a lista de passes com `-Odump` e **remove `-correlated-propagation`**, roda `IDxcOptimizer::RunOptimizer` e monta o container com `IDxcAssembler`. Duas correções que o próprio código justifica: (a) o primeiro estágio não entrega `[WaveSize(N)]` aos passes, então o helper desmonta o módulo, acha `!dx.entryPoints`, o nó de propriedades e insere `, i32 11, !999999999` mais o nó com as lanes — sem isso um programa de 64 lanes "computes wrong sums"; (b) `lib_*` não recebe `-E main`. Falha de qualquer etapa devolve `no` e o chamador cai para a linha de comando do `dxc`; `-O0` sempre vai para a CLI. `project_tuxbox/persistent_dxc.py` +109/-9 compila o helper uma vez em `build/dxc-helper/<sha256>/dxc-worker` (exige release DXC cuja string contenha `1.9.2609`, `dxc/dxcapi.h` e `libdxcompiler.so`), recusa em não-Linux, e a chave de cache do shader inclui o binário do helper; `TUXBOX_PERSISTENT_DXC=0` desliga. `chase_warmup.py`, `runtime_shaders.py`, `scarlett_prep.py`, `shader_prep.py`, `docs/PROJECTSCARLETT.md` e `docs/ENVIRONMENT.md` seguem a troca de assinatura. Testes: `tests/test_dxc_worker.py` +68 e `tests/test_persistent_dxc.py` +103, mais ajustes em três suítes |

## Pull requests abertos (8)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #321 | xshader: lower a first-lane read once its result is consumed | base `main`: baixar leitura de primeira lane quando o resultado é consumido | Área de D-23/D-24 (lowering apagado em resolução de merge; verificação quebrada) — não é o defeito |
| #320 | d3d12: build acceleration structures that titles place in their own memory | base `main`: construir AS onde o título as coloca | Não no catálogo |
| #319 | xshader: count lanes by key with one add per lane where the whole-wave form refuses | base `main`: contagem por chave quando a forma whole-wave recusa | Área de D-28 (`WaveSize(N)` recusado só no driver) |
| #318 | xshader: run a 32-thread group as the lower half of a 64-lane console wave | base `main`: grupo de 32 threads como metade baixa de wave de 64 | Área de D-20/D-21 (instruções/lane shape) |
| #317 | xshader: verify an election by the exact registers its instructions read | base `main`: verificar eleição pelos registradores exatos | Não no catálogo |
| #316 | xshader: refuse a scalar load through a root descriptor's address | base `main`: recusar load escalar por endereço de root descriptor | Área do #312/D-22 (roots e descritores) |
| #315 | xshader: fill texture size and buffer record count fields read from a descriptor | base `main`: preencher campos de tamanho/contagem lidos de descritor | Não no catálogo |
| #314 | shader-cache: keep the caches of the last three compiler builds | base `main`: manter cache das três últimas builds do compilador | Área do commit `89fb2517` (identidade de cache inclui o helper) |

Todos os oito são de autoria `NickStallman`/`csj0oe`/`gtb-sbe`, base `main`. Nenhum patch lido.

## Pull requests mesclados (1)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #312 | dxbc: confirm a compute root SRV after the tables | `8c082e65`, 2026-11-27T17:21:38Z, base `main`, pais `7953f0a5` + `49fad5f7` | O que entrou é o par `f5233880` + `49fad5f7`: a condição `root_srv` generalizada em `compute_table.rs`, o teste unitário de binding, o cenário `compute-root-srv` do selftest com fixture e a suíte `lanes-compute-root-srv` ligada em `BASE_SUITES`. O merge não trouxe nada além da branch: os 7 arquivos do diff contra o primeiro pai são os mesmos dois commits |

## Cruzamento com o catálogo

- **D-22 ("`Failed to remap` (-3) em roots raster com várias tabelas de descritor", corrigido em
  `95b242b`):** o #312 é o mesmo mecanismo — tabelas de descritor múltiplas com um parâmetro extra —
  mas em **compute** e com um **root SRV**, não um root raster. Não corrige D-22 e não é a mesma falha:
  em D-22 o root não remapeava; aqui o binding de um parâmetro que o shader nomeia era descartado.
  **Não corrige D-22.**
- **D-02/D-03 (otimizador do DXC estoura o tempo e shaders viram pipeline nulo; `cs_6_6 -O0` leva
  minutos):** `89fb2517` ataca o **tempo** de compilação, que é o sintoma de D-02, mas por outro
  caminho: remove um passe caro em vez de tornar o pipeline nulo visível. Não há, no diff, nada que
  trate o caso de estouro de tempo — o helper só tem prazo e cai para a CLI. E `-O0` continua indo
  para a CLI: **D-03 não é tocado.**
- **D-28 (`WaveSize(N)` incompatível recusado só no driver):** o helper **reescreve à mão** a
  propriedade 11 do entry point para devolver `[WaveSize(N)]` que o primeiro estágio perdeu. É a mesma
  área (largura de wave declarada no DXIL) e um risco novo: se a inserção no nó de propriedades estiver
  errada para um programa, o DXIL roda na largura que a GPU escolher, sem erro. O código lança exceção
  quando não acha o nó, o que manda a requisição para a CLI; não há teste no diff que compare o DXIL do
  helper com o do `dxc` num programa com `[WaveSize(64)]` — o teste novo é de persistência/erro.
- **D-24 ("resolução de merge apagou lowering"):** nada aqui; o merge `8c082e65` não conflitou.

## Confiança

Verificado por mim, pelo diff:

- os quatro hashes, autores, arquivos e contagens; `8c082e65` tem pais `7953f0a5` + `49fad5f7` e o diff
  contra o primeiro pai são exatamente os 7 arquivos dos dois commits de branch;
- li o hunk inteiro de `compute_table.rs` (condição nova + braço de binding) e o teste novo completo,
  incluindo os três casos de fail-closed;
- li o cenário `compute_root_srv` do selftest e o fixture `root_srv_shader`, e conferi que o teste
  compara **todas** as 16 views e as 63 palavras de guarda de cada uma;
- li `tools/dxc_worker.cpp` quase inteiro (front end, lista de passes, `with_wave_size`, assembler,
  protocolo `ok`/`no`) e o começo de `persistent_dxc.py` (build, versão verificada, cache por hash do
  fonte, fallback);
- a afirmação "51 s em vez de 60 s" é do corpo do commit; o código não mede, e eu não medi.

NÃO VERIFICADO:

- **nada foi executado.** Não rodei o selftest, as suítes de `verify.py`, `cargo test` do `runtime/dxbc`
  nem o helper; não há Linux, DXC 1.9.2609 nem GPU Proton neste ambiente;
- **o teste de hardware depende de Proton + GPU.** `lanes-compute-root-srv` só roda com
  `host.backend == "proton"`; o `test_lanes_compute_root.py` valida o fixture, não o dispatch;
- **a equivalência do DXIL gerado pelo helper com o do `dxc`.** O commit cita checagem no tradutor em
  uma RTX 4070 Ti SUPER e numa iGPU AMD Raphael e "250 shaders gravados: tudo igual menos as
  instruções, à parte os nomes mantidos". Não reproduzi. O risco que fica registrado é o oposto do que
  o commit afirma: o helper mantém nomes e **não** grava digest de container, então qualquer detecção
  de corrupção por digest a jusante deixa de valer para esses arquivos;
- **o efeito dos 15% no caso patológico de D-02.** A medição é de 1.500 requisições de ProjectScarlett
  em 16 paralelos; não há número para o shader que hoje estoura o tempo do otimizador;
- os 8 PRs abertos: li número, título, base e head; não abri nenhum patch nem corpo completo.
