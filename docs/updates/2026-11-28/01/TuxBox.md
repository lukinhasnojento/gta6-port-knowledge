# brcly/TuxBox — atualização de 2026-11-28 01:00

## Resumo

Cinco commits acima do cursor `62c7e390`, todos push direto na default (`commits/<sha>/pulls` vazio nos
cinco), nenhum PR mesclado e 9 PRs abertos. A hora é a da **cobertura medida do tradutor**: um teste
novo dá ao tradutor cada instrução RDNA 2 sozinha e compara o resultado com um registro versionado, e
foi isso que achou o defeito real da janela — o tradutor tratava **toda** instrução SMEM que não fosse
carga de descritor como carga de buffer, e `s_scratch_*`/`s_get_waveid_in_workgroup` estouravam o
cálculo de tamanho. Em cima disso, duas levas de checks de GPU (65 instruções) e uma correção de
semântica de seno/cosseno e de FMA; o inventário final mede **241 das 372 instruções usadas** com check
de GPU (eram 188).

## Commits novos (5)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `aa1f4028c1` | brcly | xshader: record what the translator does with every instruction alone | Novo módulo de teste `tools/shader-compiler/src/tests/hlsl/instruction_inventory.rs` (+124): `encode(enc, op)` fabrica **uma** instrução por família/opcode com registradores baixos e sem modificador, o tradutor roda sobre ela sozinha e o resultado é comparado com `instruction_inventory.txt` (1.398 linhas, uma por encoding); `TUXBOX_BLESS_INSTRUCTIONS=1` regrava o arquivo, e o teste falha se qualquer instrução **derrubar** o tradutor. O teste achou o defeito: em `hlsl/emitter/resource_block.rs` e `hlsl/emitter/scalar_memory.rs` entra a guarda `Enc::Smem if !(8..=12).contains(&ins.op) => Err("unsupported scalar memory instruction")` antes do braço de carga. Sem ela, `s_scratch_load_*`, controles de cache, timers e sondas caíam no caminho `s_buffer_load_*` e o `n = 1 << (op - 8)` estourava o tamanho lido |
| `fa7333b8f0` | brcly | docs: generate an inventory of every RDNA 2 instruction | `docs/INSTRUCTIONS.md` (+1709) gerado por `project_tuxbox/instruction_inventory.py` a partir de três entradas versionadas: o registro do tradutor, `evidence/shader-instructions/gpu-checks.json` (por placa) e `recorded-use.json` (44.799 traduções de ProjectScarlett 1.0.2.1). `tools/shader_harness.py` ganha o subcomando `inventory [--from-run]`, que só grava evidência de uma execução **passando e de árvore limpa** (checa `report["problems"]`, `report["dirty"]` e que o `HEAD:tools/shader-compiler` é o mesmo da execução), e o teste Python `tests/test_instruction_inventory.py` falha se o documento ficar velho. Resumo publicado: 1.085 instruções em 1.398 encodings; 372 usadas pelos shaders gravados; **188 delas rodadas por check de GPU**; 210 sem tradução e 186 "não provadas". `docs/ROADMAP.md` passa a apontar T1 como feito e T2 com 184 instruções usadas sem check |
| `2a22586959` | brcly | xshader: reduce sine and cosine angles, and ask the host to fuse multiply-adds | `hlsl/emitter/vector.rs`: `v_sin_f32`/`v_cos_f32` recebem ângulo em **voltas** e passam a calcular `sin((a - round(a)) * 6.28318530718)`; antes era `sin(a * 2π)`, que perdia a fração de um ângulo grande ou estourava (na NVIDIA o seno do maior float dava NaN e o cosseno de 2^23 voltas dava −1). `v_fma_f32`, `v_fmac_f32`, `v_fmaak_f32` e `v_fmamk_f32` passam a emitir `mad(...)` (que pede ao host a fusão e arredonda uma vez); `v_mac_f32`, `v_madmk_f32`, `v_madak_f32` e `v_mad_f32` continuam como produto + soma. `tools/check_basic_alu_gpu.py` ganha 42 instruções, entre elas `v_mul_legacy_f32`, `v_rsq_f32`, `v_fract_f32`, `v_exp_f32`, `v_log_f32`, `v_perm_b32` (com o modelo `byte_permute` novo), as `v_cvt_f32_ubyte*` e as comparações `v_cmpx` float; o recibo passa a registrar o maior erro de cada instrução aproximada e quais resultados alternativos o host aceitou |
| `9fb505bd9d` | brcly | tools: check the cube map, class, constant and carry instructions on the GPU | Mais 23 checks no mesmo arquivo: `v_cubeid/sc/tc/ma_f32` (modelo `cube()` escolhe o eixo de maior magnitude como o manual), `v_cmp_class_f32` (modelo `number_class()` com o bit por classe, incluindo sNaN/qNaN), `s_addk_i32`, `s_mulk_i32`, as doze `s_cmpk_*`, `s_bitset0/1_b32`, `s_addc_u32` e `s_subb_u32` (SCC = 1 antes da instrução, então carry/borrow entram). `v_fma_f32` passa a aceitar o resultado **não fundido** que o RADV devolve, como as outras formas fundidas já faziam |
| `1d0f3fa9fa` | brcly | docs: regenerate the instruction inventory after the new GPU checks | Regenera `docs/INSTRUCTIONS.md` e as duas evidências: a linha de resumo passa de "188 de 372" para **"241 de 372"** com check de GPU, e a tabela por família cai em "usadas sem check" (SOP2 5→4, SOPK 9→1, SOP1 7→4, VOP2 14→11, VOP1 21→10, VOPC 34→16, VOP3 94→54). O `ROADMAP.md` passa T2 de 184 para **131** instruções usadas ainda sem check. O `translator_tree` das duas evidências muda para `117ac4b0`, que é o commit em que os checks rodaram |

## Pull requests abertos (9)

Os 9 acima do cursor `last_pr_number` 328, `created_at` entre 21:14Z e 21:39Z no relógio do GitHub;
título e corpo lidos, patch não lido. Todos nascem de uma rodada de testes em Windows físico descrita
como "#313", e vários são correções de **fixture**, não do tradutor.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| `#339` | xmem: walk an NT heap for its statistics | `XMemGetWin32HeapStatistics` devolve 0/0 para heap de `HeapCreate` (NT heap) no desktop Windows; passa a andar o heap para obter os contadores | Não; vizinho de D-01/D-02 na área de XMem, defeito diferente |
| `#338` | d3d12: return a null GPU handle for a heap shaders can't see | `GetGPUDescriptorHandleForHeapStart` devolve 0 para heap sem `SHADER_VISIBLE`, como no console, em vez de repassar o valor indefinido do host (medido: `0x5678a00000000` na RTX 4060 e `0x13ae086300000140` na Intel UHD) | Não |
| `#337` | d3d12: name the earlier writes in the indirect argument barrier | A barreira global antes de ler argumentos indiretos usa `AccessBefore = COMMON` e o `Close` falha com `E_INVALIDARG` na RTX 4060; nomeia as escritas anteriores | Não; área vizinha de D-15 (fence) e D-29 (esperas de fence), defeito diferente |
| `#336` | Track VccLo/VccHi heap offsets through the s_load dataflow | O marcador de offset de heap só reconhecia `Opnd::Sgpr`; shaders que calculam o índice em `VccHi` e usam `soff=107` eram recusados como "dynamic soffset". Receipt: 0 falhas (eram 22), acionáveis 907 → 758 | Não; é a mesma família de `#3180` do `Xbox2Windows`, mas no tradutor Rust — **candidato a defeito novo** se o operador quiser registrar a recusa falsa |
| `#335` | d3d12: give residency priorities only to pageables the host takes | `SetResidencyPriority` em heap, superfície de display, heap de RTV e heap de query falha com `0x80070057`; só pageáveis que o host aceita recebem prioridade | Não |
| `#332` | status: write the tables as UTF-8 whatever the console's code page | `python tuxbox.py status` grava na code page do console (cp1252) e o travessão vira `0x97`, quebrando `test_results_table_matches_recorded_evidence`; passa a reconfigurar stdout para UTF-8 | Não; é falha de ferramenta em Windows |
| `#331` | tests: skip the Linux-only launcher tests on Windows | `unittest discover` no Windows falha com 1 falha e 17 erros em testes que usam `fcntl` e backends Wine/Proton; passam a pular | Não |
| `#330` | present checks: sample the screen in physical pixels | `GetPixel` lê pixel físico, mas os pontos de amostragem são calculados em pixel lógico; com 175 % de escala as amostras caem no desktop. Move cada ponto para o pixel físico que ele cobre | Não |
| `#329` | verify: count d3d12-guids identifiers imported by ordinal | `guid_receipt()` montava a linha esperada só com imports nomeados (`identifiers=0`) enquanto o guest importa 32 por ordinal; passa a resolver ordinais na tabela de exportação | Não |

## Pull requests mesclados (0)

Nenhum PR com `merged_at` maior que `last_run` (2026-11-27T20:01:33Z). Os cinco commits da faixa são
push direto; `repos/brcly/TuxBox/commits/<sha>/pulls` devolve lista vazia nos cinco.

## Cruzamento com o catálogo

- **Nenhum dos cinco hashes aparece em `introduzido` ou `corrigido`** no `catalogo/defeitos.csv`. Nenhum
  dos 29 D-xx de `TuxBox` é reaplicado.
- **Defeito real corrigido em `aa1f4028`, fora do catálogo (candidato a linha nova).** Antes do commit,
  o tradutor aceitava **qualquer** SMEM que não fosse carga de descritor como `s_buffer_load_*`: o braço
  `Enc::Smem` calculava `n = 1 << (op - 8)` e `b` como base de descritor. Para `s_scratch_load_*` e
  `s_get_waveid_in_workgroup` isso produz um `n` enorme/negativo e estoura o cálculo de tamanho — o
  commit diz "overflowed the size computation" e o diff confirma a guarda `!(8..=12)` nos dois
  emissores. Isso é um defeito de tradutor que estava **vivo na `main`** e foi encontrado e fechado na
  mesma janela, sem nunca ter sido reportado como issue. O catálogo de `TuxBox` não tem entrada dessa
  classe (D-24 é resolução de merge que apagou lowering).
- **D-28 não é tocado.** D-28 ("`WaveSize(N)` incompatível recusado só no driver") é o caso oposto — uma
  forma que o tradutor deixa passar e o driver recusa; o inventário desta hora cataloga recusas **do
  tradutor**, e não altera a regra de `WaveSize`.
- **O que o inventário ainda não prova.** `docs/INSTRUCTIONS.md` diz explicitamente que "Translates"
  significa que alguma instrução foi emitida, **não** que o resultado esteja correto, e que a coluna de
  GPU não mostra que toda forma de operando ou estado de lane foi testada. O `ROADMAP.md` mantém
  "Recorded shaders executed and compared with a reference: **none**" (T4) e T5 (referência de console)
  em aberto. Isto é a mesma limitação que a execução anterior registrou no achado (4): o harness agora
  mede cobertura de **instrução**, não correção de shader real.

## Confiança

- **Verificado no diff:** os cinco commits foram lidos no patch por arquivo. `aa1f4028` foi lido nos
  três pontos que mudam produção (`resource_block.rs`, `scalar_memory.rs`) e no começo do teste novo;
  o `instruction_inventory.txt` de 1.398 linhas não foi lido linha a linha, só o começo e o cabeçalho do
  teste. `2a225869` e `9fb505bd` foram lidos no lowering (`vector.rs`) e nos modelos novos do check
  (`byte_permute`, `cube`, `number_class`). `fa7333b8` e `1d0f3fa9` foram lidos nos números de resumo e
  no `ROADMAP.md`.
- **Aritmética derivada, não uma linha do repositório:** 65 checks novos (42 + 23) contra 53 instruções
  usadas a mais cobertas (188→241) implica que 12 das 65 não estão entre as 372 usadas. O documento não
  diz isso; é conta minha.
- **Verificado por API:** os 9 PRs abertos (título e corpo) e a ausência de PR nos cinco commits
  (`commits/<sha>/pulls` vazio nos cinco).
- **Não verificado:** nenhum teste executado; os números de erro de GPU (RTX 4060, Intel UHD, RADV,
  NVIDIA) são os das mensagens e dos corpos. Não conferi os 42/23 modelos contra a ISA RDNA 2 — em
  particular `v_perm_b32` (ordem de bytes S0/S1 e seletores 12/13) e `v_cmp_class_f32` (bits de classe)
  foram lidos no código, não validados contra hardware. Não sei se `s_scratch_load_*` aparece nos
  shaders gravados: o inventário marca SMEM como "seen translating 5", mas não cruzei quais cinco.
- **Contagem:** 5 commits, 0 merges — bate, são pushes diretos.
