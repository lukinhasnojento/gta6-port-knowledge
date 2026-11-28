# brcly/TuxBox — atualização de 2026-11-28 12:00

## Resumo

169 commits desde `1d0f3fa9` e 50 merges; 40 documentados (teto). A janela é dominada por duas frentes:
o *harness* de GPU — 13 commits de `tools/check_*` novos ou ampliados, mais quatro regenerações do
inventário de instruções — e a forma "dois lanes por thread" do tradutor para waves de 64 lanes, que
recebeu seis correções no emissor. O conserto de maior consequência é `24ebba7e`: `WaveActiveMin` sobre
índices de lane derrubou a GPU (Xid 109) numa RTX 5090 e foi trocado por ballot. Nada foi executado.

## Commits novos (40)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `d868ba13` | Jerkin42 | dxbc: a vertex program's fetch call does not make it write images | `image_writes.rs` lê o estágio do primeiro dword do chunk DXIL (`>> 16 == VERTEX`) e, para programa de vértice, conclui que não há escrita de imagem — antes toda chamada de *fetch* contava como escrita e forçava o caminho conservador. |
| `402249c0` | NickStallman | xshader: number thread-derived lanes two by two where a group takes derivatives | `pipeline.rs` passa a **recusar** grupo de compute que toma derivada e tem mais de 16 threads (o host o dispõe 2×2 e a wave pode não começar em fronteira); `output.rs` numera lanes em blocos 4k..4k+3 quando o grupo tem linhas e `tx`/`ty` pares. |
| `af2cf51b` | NickStallman | tools: check lane numbering of a derivative-taking compute group on the GPU | `check_derivative_lanes_gpu.py` (140 linhas): programa sintético que amostra nas próprias coordenadas com derivada implícita, troca x no quad por `ds_swizzle_b32` e lê o lane 5; registrado em `check_translator_gpu.py`. |
| `ae829f98` | NickStallman | xshader: follow a heap descriptor through its vector copy and waterfall | Módulo novo `hlsl/descriptor_broadcast.rs` (147 linhas): reconhece descriptor copiado para registradores vetoriais (`v_mov_b32 vK, sK` por palavra) e servido por waterfall (`v_readfirstlane_b32`), mantendo a origem por lane. |
| `370756c0` | NickStallman | xshader: read heap slot 0 when a descriptor waterfall has no active lane | `resource_block.rs`: `heap_index_reads` passa a ser reavaliado em cada visita ao bloco (o último vê o estado final) e o índice sai do conjunto quando a palavra deixa de ser descriptor; sem isso um waterfall sem lane ativa lia o slot errado. |
| `1e5ef81c` | NickStallman | xshader: run loop swizzles in whole 64-lane waves two lanes per thread | Fixture novo `shader_loop_wave64` e caso `swizzle_loop` no `check_paired_wave64.py`: troca dentro de laço que aposenta lanes de um EXEC de duas palavras, como só uma wave de 64 mantém. |
| `738f140b` | NickStallman | tools: check a heap descriptor waterfall on the GPU in the two-lane form | Caso `heap_waterfall` no `check_paired_wave64.py` e opção `heap` no exemplo: o programa indexa o heap de recursos (base em s10) e o *check* substitui o heap por uma função do índice e da coordenada. |
| `4a9fc953` | s3rius | Separated actionable from expected refusals | `scarlett_report.py` ganha `expected_at_preparation()`: programa mesh sem RTS0 embutido não é traduzível na preparação (a pipeline o cria contra a raiz do guest depois) e passa a ser listado à parte das recusas acionáveis. |
| `e07bf8a0` | KainanP | verify: count d3d12-guids identifiers imported by ordinal | `guid_receipt(imports, provider)` recebe o provedor e conta símbolos por nome **ou ordinal**, em vez de olhar só `d3d12_xs.dll`. |
| `05942735` | KainanP | present checks: sample the screen in physical pixels | `GetPixel` lê pixel físico e a janela do guest não é *DPI aware*; `screen_pixel()` converte cada ponto amostrado por `PhysicalToLogicalPointForPerMonitorDPI` antes da leitura. |
| `8520d394` | KainanP | tests: skip the Linux-only launcher tests on Windows | Marca os testes de lançador como Linux-only. Só teste. |
| `84d382f9` | KainanP | status: write the tables as UTF-8 whatever the console's code page | `tuxbox.py status` chama `sys.stdout.reconfigure(encoding="utf-8")` — as tabelas vão para Markdown, que é UTF-8 qualquer que seja o code page. |
| `9c2511df` | KainanP | d3d12: give residency priorities only to pageables the host takes | `SetResidencyPriority` só é aplicado a pageables que o host aceita (retorna `S_OK`); `mark_committed(native, flags)` novo em `create_resource` registra o compromisso. |
| `94f3803b` | KainanP | d3d12: name the earlier writes in the indirect argument barrier | `indirect_order::barrier(list_type)` substitui a barreira `ACCESS_COMMON` global: o runtime de desktop recusa `COMMON` antes de um acesso nomeado e o `Close` falhava; agora a barreira nomeia as escritas anteriores que podem gerar argumentos. |
| `5f926353` | KainanP | d3d12: return a null GPU handle for a heap shaders can't see | `gpu_handle` devolve 0 quando o heap não é `SHADER_VISIBLE` — o console devolve nulo e o desktop devolve valor indefinido (frequentemente não nulo). |
| `785b095a` | KainanP | xmem: walk an NT heap for its statistics | `statistics()` detecta heap NT (feito por `HeapCreate`) e faz o *walk*, porque a consulta estendida responde mas deixa os contadores de segmento em zero; heap de Wine continua no caminho antigo. |
| `ec82f400` | NickStallman | tools: check 64-bit scalar instructions and the RDNA 1 multiply-adds on the GPU | `check_basic_alu_gpu.py` ganha modelo de 64 bits (`s64`) e as relações `s_addc/s_subb` de 64 bits; `check_packed_integer_gpu.py` deixa de passar `-mcpu` quando o chamador já passou um. |
| `5324096e` | brcly | docs: regenerate the instruction inventory | `evidence/shader-instructions/gpu-checks.json` recebe `s_ashr_i64` e outras entradas novas. Dado gerado. |
| `55caf3ef` | brcly | tools: check buffer loads on the GPU against buffers of known contents | `check_buffer_loads.py` (149 linhas): lê de volta o que *loads* traduzidos devolvem de buffers de conteúdo conhecido (constante, raw e tipado), a partir de HLSL extraído por `cargo test ... --nocapture`. |
| `52f19fe7` | brcly | docs: regenerate the instruction inventory | `gpu-checks.json` e `recorded-use.json` com `translator_tree` novo (`0bfd591e`), 55 checks contra 54. Dado gerado. |
| `671090f8` | s3rius | Track VccLo/VccHi heap offsets through the s_load dataflow | `resource_block.rs`: `Opnd::VccLo`/`VccHi` passam a ter chave de dados 106/107 (antes caíam em "sem chave"), então um offset de heap que passa por VCC é seguido; `output.rs` deixa de redeclarar registradores que o cabeçalho já declara (`exec_lo/hi`, `vcc_lo/hi`, `scc`, `m0`, `nul`). |
| `ad70d649` | NickStallman | tools: check the half-precision instructions on the GPU | `check_basic_alu_gpu.py` importa `EDGE_F16`, `encode_half`, `f32_value`, `half_value` do check de exatidão de meia precisão e acrescenta as formas f16. |
| `ea532e2a` | brcly | tools: check buffer stores and atomics on the GPU | `check_buffer_loads.py` deixa de ser só leitura: acrescenta stores e atômicos a buffers graváveis; o módulo de teste `buffer_loads.rs` ganha os casos correspondentes. |
| `1445af9d` | NickStallman | tools: check group memory instructions on the GPU | Tabela `GROUP_MEMORY` nova com `ds_write_b32`/`ds_read_b32` e o modelo de resultado (uma thread, endereço em v20). |
| `ab3fb41f` | NickStallman | tools: check 64-bit vector shifts and compares, carries, swap and relative moves on the GPU | Acrescenta `s_wqm_b32` (quads) e as relações de carry/borrow de 64 bits ao modelo escalar do *check* de ALU. |
| `25a62cc3` | brcly | docs: regenerate the instruction inventory | Dado gerado. |
| `d91fe5e5` | NickStallman | xshader: return D16 buffer loads as packed 16-bit values | `buffer.rs`: quando o mnemônico contém `_d16_`, cada canal volta como f16 e é empacotado por `write_d16(vd, …)` — antes a carga D16 era devolvida como f32 por canal. |
| `3943d26c` | NickStallman | tools: check flat loads from a program's own table on the GPU | `source_kind == "code"` entra no caminho raw do *check* (programa que lê a própria tabela não tem buffer), e o fixture deixa de emitir a sequência do descritor nesse caso. |
| `4ede7bc4` | NickStallman | tools: check pixel attribute interpolation on the GPU | `check_pixel_interp.py` (135 linhas): programa de pixel que lê dois atributos com `v_interp_p1/p2_f32`, `v_interp_mov_f32` e as formas `_p1ll_f16`/`_p2_f16` e exporta os bits; registrado no `check_translator_gpu.py`. |
| `c098ce62` | KainanP | xmem guest: take a guard page with a real access where ReadProcessMemory leaves it | No guest de teste, depois de `ReadProcessMemory` falhar num guard, faz um acesso real (que `IsBadReadPtr` captura) antes de exigir que a proteção volte — o Windows mantém o guard, o Wine o consome. |
| `fdcf1069` | NickStallman | xshader: name the level count of a gather so it can't be a register | `check_image_ops.py` novo (213 linhas) e o nível do gather passa a ser nomeado no texto gerado, impedindo que seja lido como registrador. |
| `71168816` | KainanP | d3d12: fall back to null UAV and aliasing barriers for a cache barrier | Quando o *enhanced barrier* global não existe, listas DIRECT/COMPUTE passam a emitir `legacy_full_barrier` em vez de erro de gravação; antes a lista morria com "enhanced global barriers unavailable". |
| `fec681f8` | NickStallman | tools: check the prefetch hint, the wait, s_andn2_wrexec_b64 and v_cvt_pk_u8_f32 on the GPU | Casos novos no *check* de ALU: `s_inst_prefetch` e `s_waitcnt_vscnt` como identidade, e `s_andn2_wrexec_b64`/`v_cvt_pk_u8_f32` com os modelos de máscara e conversão. |
| `72e145df` | NickStallman | tools: run a two-word descriptor fetch in the buffer-loads check | Caso `raw-after-fetch-x2`: `s_load_dwordx2` antes da carga não muda o que a carga lê. Só teste. |
| `af023303` | brcly | docs: regenerate the instruction inventory | Dado gerado. |
| `e6b68c54` | NickStallman | tools: let the arithmetic GPU check run chosen instructions | `--only` repetível no *check* de ALU, para rodar uma instrução por vez. |
| `24ebba7e` | NickStallman | xshader: find the first active lane with a ballot, not a lane minimum | `s_ff1_i32_b64` sobre EXEC deixa de usar `WaveActiveMin(lane)` e passa a usar `first_active_lane()` (ballot); o comentário registra que o mínimo, numa wave com lanes discordantes, **perdeu o dispositivo** numa RTX 5090 (Xid 109, CTX SWITCH TIMEOUT). |
| `38207ed5` | NickStallman | tools: add a GPU check for float compares and v_cmpx | `check_float_compares_gpu.py` (515 linhas): todos os `v_cmp_*_f32`/`v_cmpx_*_f32` e `class` sobre zeros, denormais, infinitos e NaN, nas codificações VOPC e VOP3, com modificadores, literal, fonte escalar e modo denormal; algumas lanes inativas na entrada. |
| `84f3608b` | s3rius | Merge pull request #326 from s3rius/shader-prep/expected-mesh-root-refusals | Merge de dois pais de `#326`. O diff é o mesmo de `4a9fc953` (branch + merge, ambos na faixa). |
| `8420ba03` | s3rius | Merge pull request #336 from s3rius/fix/s-load-vcc-heap-offsets | Merge de dois pais de `#336`. O diff é o mesmo de `671090f8`. |

## Pull requests abertos (11)

Lidos por título e corpo; **nenhum patch de PR aberto foi lido**.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #394 | bvh, d3d12: rehydrate console structures as host builds (stacked on #376) | Empilhado em #376→#375→#320: refaz as estruturas de BVH do console como builds do host. | Área de D-18 (aliasing/BLAS), não o defeito. |
| #393 | d3d12: keep replaced views for the records a title copied | Empilhado em #391: guarda as views substituídas quando o título copiou o registro de 32 B para o heap indexado. | Não. |
| #391 | d3d12: follow descriptor records a title copies into an indexed heap | Percorre o registro copiado por store comum da CPU para o heap visível. | Não. |
| #390 | d3d12: give mesh pipelines the resource index map | **Rascunho declarado:** expõe um travamento de GPU em ProjectScarlett que o autor não resolveu; com ele o jogo roda os draws de terreno que a `main` pula. | Candidato a defeito novo (hang), admitido no corpo. |
| #389 | d3d12: map title memory far past root SRVs and UAVs | Root SRV/UAV em memória do título era importado por **uma página** de 64 KiB; passa a mapear o intervalo inteiro. | Não. |
| #385 | d3d12: detile placed mips through address tables and reuse upload buffers | Detile por tabelas de endereço em cache (0,4–0,x ms por run de 16 B) e reuso de buffers de upload. | Não. |
| #376 | d3d12: decode packed positions in title-memory BLAS builds (stacked on #375) | Posições empacotadas nos builds de BLAS do próprio título. | Não. |
| #375 | d3d12: decode meshlet-coded indices in title-memory BLAS builds (stacked on #320) | Índices codificados como meshlet. | Não. |
| #373 | d3d12: keep a render heap while its guest range is committed | `RenderHeaps` guardava só `Weak` por faixa do guest, e um alvo colocado via slot 133 perdia o heap; fecha `#323`. | Não é `D-xx`; área de D-17/D-18. |
| #371 | tools: rank actionable refusals by pipeline impact | `tools/refusal_impact.py` separa falha de driver/stand-in/lista e ordena as recusas acionáveis por pipelines afetados. | Não. |
| #344 | d3d12: append the descriptor slots a raster root leaves out | Root raster que omite slots da tabela mantinha a raiz do guest e o pipeline não oferecia elemento para as SRVs nomeadas. | Área de **D-22** (`Failed to remap` em root raster com várias tabelas), não o mesmo caso. |

## Pull requests mesclados (2)

Dos 50 merges na janela, apenas `#326` e `#336` têm o merge commit dentro dos 40 documentados; os outros
48 estão além do teto de commits e ficam para o próximo ciclo.

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #326 | shader-translatro: Separated actionable from expected refusals | `84f3608b` | Exatamente o prometido: o recibo de preparação passa a separar a recusa esperada de mesh sem RTS0 das acionáveis. O commit de branch `4a9fc953` está na mesma faixa. |
| #336 | shader-translator: Fixed the s_load related errors | `8420ba03` | O diff não é o que o título diz: não há correção de `s_load` — são duas coisas, (a) `VccLo/VccHi` com chave 106/107 no dataflow de offset de heap e (b) o emissor deixando de redeclarar registradores especiais. O título ficou genérico demais. Commit de branch `671090f8`. |

## Cruzamento com o catálogo

- **Nenhum dos 40 hashes está no catálogo.** Os 169 hashes da janela também não (conferência por
  prefixo contra 262 hashes distintos).
- **Área de D-21** ("assinatura mesh/pixel incompatível derruba o draw", `315b287`): nesta janela há
  `#356` na lista de merges ("refuse mesh and ray signatures the host cannot run"), mas o commit
  correspondente (`d8ad4773`) está **fora** dos 40 documentados. Não documentado aqui.
- **Área de D-28** (`WaveSize(N)` recusado só no driver): os commits de forma dois-lanes tratam de
  onda de 64 lanes no tradutor, não de `WaveSize`. Sem relação demonstrada.
- **Candidato a linha de catálogo (novo, com evidência no próprio diff):** `24ebba7e`. O emissores
  usava `WaveActiveMin` sobre índices de lane para achar a primeira lane ativa de `s_ff1_i32_b64`, e
  esse mínimo, em wave com lanes discordantes, **derrubou o dispositivo** numa RTX 5090 (Xid 109,
  CTX SWITCH TIMEOUT). O conserto é ballot. É a primeira vez que este acervo registra perda de
  dispositivo por intrínseca de wave escolhida errado; a classe não existe em `catalogo/defeitos.csv`
  para `TuxBox`.
- **Candidato a linha de catálogo (novo, sem prova em jogo):** `94f3803b`. A barreira global de
  argumentos indiretos usava `ACCESS_COMMON` e o runtime de desktop **recusa** `COMMON` antes de um
  acesso nomeado, fazendo o `Close` da lista falhar. O commit troca por barreira que nomeia as
  escritas anteriores. O `catalogo` tem a classe em `XCL D-20` (ExecuteIndirect lendo argumentos antes
  da barreira), mas para `TuxBox` não há linha.
- Nenhum defeito do catálogo é fechado por esta janela.

## Confiança

- **Verificado por API:** 40 commits (sha, data, autor), o `patch` de cada um nos arquivos de código,
  e os 50 merges da janela por `pulls?state=closed` filtrado em `merged_at > 2026-11-27T20:01:33Z`.
- **NÃO VERIFICADO:** nada foi compilado nem executado. Os *checks* de GPU citados exigem placa e
  drivers e **não** rodaram nesta coleta; os arquivos `evidence/shader-instructions/*.json` que os
  commits de documentação atualizam foram lidos só como dado gerado, e eu não os reexecutei. A
  afirmação de que `WaveActiveMin` derruba a RTX 5090 é do comentário do autor em `24ebba7e`, não uma
  medição desta rotina.
- **Commits duplicados na faixa:** `4a9fc953` e `84f3608b` têm o mesmo diff (branch + merge de `#326`);
  `671090f8` e `8420ba03`, idem (`#336`). São 40 entradas para 38 mudanças distintas.
- **Teto de 40 atingido:** 40 de 169. `last_commit` fica em `1d0f3fa9`, `commit_cursor = 8420ba03`
  (40º documentado) e `head_at_run = 6f89916a`. `compare` a partir de `8420ba03` devolve **148**
  commits, contra 129 restantes da janela original: 19 dos 40 documentados são ancestrais do 40º (que
  é um merge), então a próxima janela é maior que o resto. A diferença está registrada para que o
  próximo ciclo a veja.
- **Commits × merges:** 50 merges contra 2 merge commits documentados. Não é divergência de coleta:
  os 48 restantes têm o merge commit além do 40º.
- **`last_run`** gravado em `2026-11-28T00:05:08Z` (o `merged_at` de `#336`, o mais recente entre os
  dois merges documentados). Os outros 48 merges, até 12:07:42Z, ficam para o próximo ciclo.
