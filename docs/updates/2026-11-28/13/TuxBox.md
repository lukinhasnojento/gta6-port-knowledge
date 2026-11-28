# brcly/TuxBox — atualização de 2026-11-28 13:00

## Resumo

Quarenta commits novos a partir do primeiro commit não documentado (`39fa7de8`, índice 19 da página do
`compare`): o ciclo das 12:00 já havia documentado os 19 primeiros, e a janela desta hora começa
onde ele parou. A hora tem três blocos: o fechamento da frente *two-lane* para waves de 64 do
console (oito commits de tradução, com a forma de derivadas duas a duas), o **decodificador BCPack**
do DirectStorage (rANS, BC1, BC3, BC4, BC5, BC7 — quatro commits e ~5.000 linhas), e uma leva de
consertos de host (cache da NVIDIA apagado por processos auxiliares, barreira indireta recusada
pelo runtime de desktop, handle de heap invisível, heap NT no XMem, barreira de cache sem *enhanced
barriers*). O `last_commit` vai a `26d0f6df` e o cursor fica em `02fceb26` (108 commits restantes).

## Commits novos (40)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `39fa7de8` | brcly | Merge pull request #343 | Merge de dois pais do conserto `d868ba13` (já documentado às 12:00): em `runtime/dxbc/src/image_writes.rs`, um programa de vértice que chama o programa de fetch por `s_swappc_b64` deixa de contar como escrita de imagem, porque a tradução materializa as entradas do layout e nunca executa a chamada. O merge não acrescenta diff. |
| `c1a7488b` | brcly | Merge pull request #325 | Merge do `402249c0`: grupo de compute que toma derivadas e tem linhas é numerado dois a dois (`lane = ((gtid.z*ty/2 + gtid.y/2)*tx/2 + gtid.x/2)*4 + (gtid.y&1)*2 + (gtid.x&1)`), com recusa de grupo mais largo que 16 threads que atravessa waves. |
| `c560ed68` | brcly | Merge pull request #327 | Merge do `1e5ef81c`: troca de lane (`ds_swizzle_b32`) dentro de laço em grupos de waves de 64 inteiras passa a usar a forma two-lane em todo host (`paired_wave64::takes_over`), porque com uma lane por thread a troca exige barreira que todo thread tem de alcançar igual. |
| `e63a5bf8` | brcly | Merge pull request #328 | Merge do `ae829f98` + `370756c0`: descritor de heap copiado para VGPRs e lido de volta por `v_readfirstlane` num waterfall; quando a wave não tem lane ativa, o índice lido passa a ser 0 em vez do que a lane 0 tiver. |
| `6f2c1fea` | Nick Stallman | xcrdapi: log refused requests and count requests by option word | `runtime/xcrdapi/src/diag.rs` novo (154 linhas): a primeira recusa de cada palavra de opção distinta é sempre registrada; `TUXBOX_XCRD_LOG=1` conta cada requisição por opção (zlib, BCPack, swizzle, páginas, memória) com origem, destino e bytes intermediários, e despeja totais a cada 5 s e ao fechar a fila. O guest ganha o caso de um BCPack BC7 que falha com `E_NOTIMPL` e deixa o destino com `0xa5`. |
| `01132561` | Nick Stallman | xcrdapi: add a platform-independent rANS fragment decoder | Crate novo `runtime/xcrdapi/codec` com `src/rans.rs` (228 linhas): modela uma lane do decodificador rANS do sample público da Microsoft (`mdu_minirans.v`, MIT) — cabeçalho de 2/3 bytes, redução de alfabeto, reset de estado, renormalação abaixo de 2^24, drenagem entre fragmentos. Sem dependência de plataforma; entra no CI (`cargo test -p tuxbox-xcrdapi-codec`). |
| `09df7152` | Nick Stallman | xcrdapi: decode BCPack BC1 requests | 1.725 linhas em 21 arquivos: requisição com `BcpackMode` BC1 deixa de falhar com `E_NOTIMPL` e é decodificada no worker; com zlib junto, a fonte é inflada para `IntermediateSize` primeiro. Stream de 12 lanes, modelos de probabilidade do silicon-rans e reconstrução de bloco BC1 portados de `bcpack-rs 5bf5cc5` (Apache-2.0). O corpo declara 400 chunks do título byte a byte idênticos ao bcpack-rs. |
| `8f449025` | Nick Stallman | xcrdapi: decode BCPack BC3, BC4, BC5 and BC7, and use the format's BC1 tables | 2.969 linhas: BC3/BC4/BC5/BC7 decodificados como BC1; BC2 e BC6H seguem recusados (sem dado de título / sem descrição de rebuild). Tabelas geradas por `gen_tables.py` de `bcpack-rs 2d6333b`, e parte delas vem do `bcpack-tables.bin` do VIctory (tratado como constante de formato, com a ressalva de licença escrita em `docs/REFERENCES.md`). Declara 15% dos blocos BC7 iguais ao decodificador do VIctory e os nossos "mais suaves" em 39 de 40 chunks. |
| `5a221d25` | brcly | xshader: take a logarithm close to 1 from a series | `v_log_f32` deixa de chamar `log2()` direto: `tb_log2(x)` usa a série de `2 atanh((x-1)/(x+1))` para `0.75 < x < 1.5`, porque o erro absoluto do `log2` do host (≈1e-7 junto de 1) é milhões de ULP de um resultado pequeno. O check de ALU passa a exigir 4 ULP em vez de 16 e a gravar a entrada do maior erro. `host_profiles.txt` inteiro regenerado (49 linhas). |
| `3e33cbbe` | brcly | Merge main into xshader/first-lane-ballot | Merge de dois pais que traz a `main` para a branch do ballot. O diff é o do `24ebba7e` já documentado às 12:00 (`first_active_lane` por ballot) mais a resolução de conflito com o índice de heap vazio. |
| `84a1f92d` | brcly | Merge #348: float compares over whole waves | Merge de dois pais do `38207ed5` (check de `v_cmp`/`v_cmpx` de f32, 624 linhas, já documentado). Não acrescenta diff próprio. |
| `0b082aa0` | brcly | Merge pull request #347 | Merge de dois pais da branch do primeiro-lane por ballot — o mesmo conteúdo do `24ebba7e`, agora na `main`. |
| `10b48fd4` | brcly | Merge pull request #348 | Merge de dois pais do check de comparações de float (`38207ed5`). Duplica o `84a1f92d` por outro caminho. |
| `381a9296` | Nick Stallman | xshader: leave out what a lane does not use in the two-lane form | Na forma two-lane, o emissor deixa de escrever o que uma lane não lê: `used`/`written` são calculados a partir dos identificadores do corpo e cada lane só sombreia a máscara que o código nomeia. O HLSL gerado encolhe; `host_profiles.txt` muda em 24 linhas. |
| `8d8075b3` | Nick Stallman | tools: measure where a compute group's threads sit in a wave and its quads | Ferramenta nova `tools/measure_compute_quads.py`: mede, numa GPU real, em que lane da wave e em que quad cada thread de um grupo de compute cai, para grupos até 32×32 e com profundidade. É a base empírica da numeração dois a dois. |
| `574589be` | Nick Stallman | tools: run the compute quad measurement on the adapter asked for | Acrescenta seleção de adaptador à ferramenta acima (`--adapter`), para medir numa placa específica em host com mais de uma. |
| `3e4f706a` | Nick Stallman | xshader: lower a 64-lane prefix scan whose scratch only looked alive | Um scan de prefixo de 64 lanes cujo scratch só *parecia* vivo passa a ser traduzido: a análise de vivacidade deixa de considerar viva a memória que nenhuma lane lê de fato. `tools/check_mask_ballots_gpu.py` perde a lista `NATIVE_ONLY` que excluía `scan` da forma de 32 lanes. |
| `e1714696` | Nick Stallman | xshader: write each instruction of the two-lane form once | A forma two-lane passa a nomear o registrador vetorial como array (`vN__[0]`, `vN__[1]`) e escrever a instrução uma vez por lane, em vez de duplicar o texto por lane. `lane_registers()` reescreve cada identificador; os checks de máscara acompanham a mudança de nome. |
| `b2f39b3a` | Nick Stallman | xshader: read inactive lanes in a two-lane row permute as its modifiers say | `v_permlane16_b32`/`v_permlanex16_b32` passam a respeitar `OP_SEL[0]` (busca de lane inativa) e `OP_SEL[1]` (lane inativa fornece zero); sem modificador o destino mantém o valor. O check do modelo de referência ganha os dois casos e o controle de `op_sel:[1,0]`. |
| `56513f59` | Nick Stallman | xshader: lower register-relative vector moves in the two-lane form | Movimentos vetoriais com índice de registrador relativo passam a ser traduzidos na forma two-lane, em vez de recusados. |
| `25848e09` | Nick Stallman | xshader: choose a one-of-N buffer once for both lanes in the two-lane form | A escolha de um buffer entre N (cadeia de comparações sobre o índice) é feita uma vez para as duas lanes, e não uma vez por lane. |
| `b1e1d85e` | Nick Stallman | xshader: take derivatives in the two-lane form, two by two in a group with rows | A forma two-lane recusava qualquer programa que tomasse derivada implícita. Agora, para grupo de **uma linha**, o grupo do host é uma linha e o quad do host são quatro threads em linha — as mesmas quatro lanes de cada metade —, então nada mais é preciso; grupos com linhas continuam exigindo a numeração dois a dois. `check_paired_wave64.py` ganha `QUAD_GROUPS` (8×8, 16×8, 4×16, 64×1) e o modelo de derivadas por quad. |
| `87116da5` | Nick Stallman | tools: run the mask ballot check's 64-lane scan in the two-lane form | Remove `NATIVE_ONLY = {"scan"}` e o `continue` que pulava o scan na forma de 32 lanes: a forma paired agora segue o permute do scan, que busca lanes fora de EXEC. −7 linhas, sem diff de tradução. |
| `0e2e6ef3` | Nick Stallman | launcher: keep helper processes from cleaning an exported NVIDIA shader cache | `project_tuxbox/host.py` ganha `keep_nvidia_shader_cache(env)`: quem recebe `__GL_SHADER_DISK_CACHE_PATH` sem `SKIP_CLEANUP` faz o driver, no arranque, comparar a pasta inteira com o teto padrão e apagar todo arquivo que nenhum processo segura. O `pipeline_prep` passa a chamar a função para `vulkaninfo` e o helper, e o `verify` remove a variável do ambiente do Xwayland privado. Dois testes novos. |
| `80fcabe8` | brcly | xshader: a program that only handles 32-bit masks has one-word compares | `operand_fields::one_word_masks`: se alguma instrução escalar de 32 bits toma EXEC/VCC e nada trata máscara como 64 bits nem nomeia a metade alta, um `v_cmp` VOP3 em SGPR par escreve **um** registrador e não zera o seguinte. O check de float compares deixa de esperar o defeito (`s25`) e passa em RTX 4070 Ti SUPER e RADV. |
| `79bb224e` | brcly | xshader: let the instruction inventory test write its translations | Com `TUXBOX_INSTRUCTION_HLSL` apontando para uma pasta, cada tradução que emite instrução é gravada com o alvo DXC que o `xshader` usaria (`target()` deriva `vs_6_x`/`cs_6_x` do perfil e dos recursos), para o inventário conseguir compilar cada uma. |
| `edfbcf0c` | brcly | xshader: declare the upper registers a 64-bit scalar compare reads | `s_cmp_eq_u64` e `s_cmp_lg_u64` leem o segundo registrador de cada par sem declará-lo; num programa em que nada mais o nomeava, o DXC não compilava. Correção de 6 linhas em `emitter/scalar.rs` (`touch(&hi(operand))`). Foi achada compilando cada instrução isolada. |
| `89587e41` | brcly | status: map RDNA 2 instruction coverage instead of shader archive counts | O mapa de progresso deixa de contar entradas de arquivo que produziram saída e passa a contar **instruções RDNA 2 convertidas para HLSL que compila para DXIL** (compilou sozinha via DXC/DXV ou um check de GPU a executou): 689 de 1.085 (63,5%), barra geral 76,2% (era 94,2%). `evidence/shader-instructions/dxil.json` novo com 919 linhas; `shader_harness.py inventory --compile`; todos os 911 encodings que o tradutor emite compilam. |
| `2d7e536a` | brcly | Merge pull request #349 | Merge do `0e2e6ef3` (cache da NVIDIA). Sem diff próprio. |
| `71ad0fd4` | brcly | Merge pull request #330 | Merge do `05942735`: as checagens de tela amostram em pixel físico (`LogicalToPhysicalPointForPerMonitorDPI`), porque `GetPixel` lê pixel físico e a janela não é DPI-aware. |
| `f8f79f2d` | brcly | Merge pull request #331 | Merge do `8520d394`: testes que só rodam no Linux (flock, network namespace, lançamento offline) passam a ser pulados no Windows em vez de falhar. |
| `9d81680d` | brcly | Merge pull request #332 | Merge do `84d382f9`: `tuxbox.py status` reconfigura `stdout` para UTF-8, porque o console do Windows escrevia em cp1252 e os travessões viravam `0x97`. |
| `a905f69d` | brcly | Merge pull request #335 | Merge do `9c2511df`: `SetResidencyPriority` só recebe heaps e recursos *committed*; recurso placed/reserved/display, descriptor heap e query heap são omitidos (o runtime de desktop devolve `E_INVALIDARG` e remove o device). Marca `COMMITTED` por `SetPrivateData`. |
| `1d5c847c` | brcly | Merge pull request #337 | Merge do `94f3803b`: a barreira global antes de `ExecuteIndirect` deixa de usar `ACCESS_COMMON` e nomeia `UNORDERED_ACCESS|COPY_DEST` (mais `STREAM_OUTPUT|RESOLVE_DEST` em lista direta), porque o runtime de desktop recusa `COMMON` antes de acesso nomeado e o `Close` falha. |
| `5af445df` | brcly | Merge pull request #338 | Merge do `5f926353`: `GetGPUDescriptorHandleForHeapStart` devolve 0 para heap sem `SHADER_VISIBLE`, como o console; o runtime de desktop devolvia valor indefinido. |
| `886376ee` | brcly | Merge pull request #339 | Merge do `785b095a`: `XMemGetWin32HeapStatistics` detecta heap NT (feito por `HeapCreate`) pelos contadores zerados e percorre o heap em vez de confiar no item de segmento. |
| `f36b59b3` | brcly | Merge pull request #340 | Merge do `c098ce62`: no guest do XMem, uma página `PAGE_GUARD` é consumida com acesso real (`IsBadReadPtr`) no Windows, que recusa o `ReadProcessMemory` com `ERROR_PARTIAL_COPY` e mantém a guarda. |
| `903567aa` | brcly | Merge pull request #341 | Merge do `71168816`: as barreiras de cache do console (slots 131/132 e flush de cache de textura) caem para barreira legada (UAV nula + aliasing nula) quando o host não tem *enhanced barriers*, em vez de deixar a lista falhar no `Close`. |
| `9fba2d51` | brcly | Merge pull request #346 | Merge do `e1714696`/`381a9296`: HLSL menor na forma two-lane (`vN__[2]` em vez de `vN__0`/`vN__1`, instrução escrita uma vez). |
| `26d0f6df` | brcly | Merge pull request #350 | Merge do `b2f39b3a`: semântica de `OP_SEL` em `v_permlane*` (bit 0 busca inativa, bit 1 zera); o bit 3 que o compilador do console liga é aceito e não seleciona nada. |

## Pull requests abertos (14)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #344 | d3d12: append the descriptor slots a raster root leaves out | Um root raster que deixa slots de descritor fora da tabela mantinha o root do guest quando não era heap-indexado; o pipeline oferecia nenhum elemento de tabela para os SRVs que o tradutor nomeia. O PR anexa os slots. | **Sim, por área: D-22** ("`Failed to remap` (-3) em roots raster com várias tabelas de descritor", corrigido em `95b242b`). Não provei que é o mesmo caso. |
| #371 | tools: rank actionable refusals by pipeline impact | `tools/refusal_impact.py` separa linhas de falha que não são recusa (driver deadline, stand-ins, não listados) das recusas do tradutor e ordena os grupos acionáveis por pipelines. | Não; ferramenta de triagem. |
| #373 | d3d12: keep a render heap while its guest range is committed | Fixes #323: um render target colocado pelo título num endereço do guest vive num render heap device-local, e `RenderHeaps` guardava só um `Weak` por faixa — o heap podia sumir com a faixa ainda comprometida. | Área de **D-17** (liberação imediata de backing). |
| #375 | d3d12: decode meshlet-coded indices in title-memory BLAS builds | Empilhado em #320: decodifica índices codificados por meshlet em BLAS construídas na memória do título. | Não. |
| #376 | d3d12: decode packed positions in title-memory BLAS builds | Empilhado em #375: posições empacotadas nas mesmas BLAS. | Não. |
| #385 | d3d12: detile placed mips through address tables and reuse upload buffers | Detila mips colocados por tabelas de endereço em cache (em vez de perguntar o calculador XG a cada run de 16 bytes) e reusa buffers de upload. | Não. |
| #389 | d3d12: map title memory far past root SRVs and UAVs | Um root SRV/UAV em memória do título era ligado por import de uma página; o PR mapeia além disso. | Não. |
| #390 | d3d12: give mesh pipelines the resource index map | Draft declarado: com a mudança o jogo roda os draws de mesh de terreno, e em três runs com #386 a **GPU travou**; o autor não resolveu. | Candidato a **D-16/D-21** (perda de device / assinatura mesh), sem prova. |
| #391 | d3d12: follow descriptor records a title copies into an indexed heap | No console a memória do heap contém os descritores, e o título copia registro de 32 bytes para o heap visível com stores comuns; o PR segue esses registros. | Não. |
| #393 | d3d12: keep replaced views for the records a title copied | Empilhado em #391: mantém views substituídas para os registros que o título copiou. | Não. |
| #394 | bvh, d3d12: rehydrate console structures as host builds | Empilhado em #376: reidrata as estruturas de BVH do console como builds do host. | Não. |
| #395 | d3d12: evict descriptor tables in one pass and keep submission off the table lock | A evicção de tabela de descritor varria todas as tabelas residentes por tabela expulsa e repetia até abrir espaço contíguo; o PR faz numa passada e tira a submissão do lock da tabela. | Não. |
| #396 | xshader: decide a loop's back edge for the whole wave too | `8724e8f` transformou branches forward em voto `WaveActiveAnyTrue`, mas a **borda de trás** de um laço estruturado ficou decidida por lane; o PR decide a borda para a wave. | Não; é continuação de trabalho já documentado. |
| #397 | tools: per-stage baseline of translation and pipeline compile cost | T14 do `docs/ROADMAP.md`: mede tempo e pico de memória por estágio; não muda compilação. | Não. |

## Pull requests mesclados (17 de 47 na janela)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #325 | xshader: number thread-derived lanes two by two where a compute group takes derivatives | `c1a7488b` | A numeração dois a dois em grupo com linhas e a recusa de grupo >16 threads que atravessa waves. Confere com o título. |
| #327 | xshader: keep a loop's swizzles in the two-lane form for whole console waves | `c560ed68` | `paired_wave64::takes_over` para `"compute swizzle inside a loop"` com `threads % 64 == 0` e EXEC de duas palavras. Confere. |
| #328 | xshader: read a heap descriptor through the copy it was loaded from, one wave at a time | `e63a5bf8` | `descriptor_broadcast.rs` e a regra de heap slot 0 para wave sem lane ativa. Confere. |
| #347 | xshader: find the first active lane with a ballot, not a lane minimum | `0b082aa0` | `first_active_lane()` com `WaveActiveBallot` e `firstbitlow` da segunda palavra. O `WaveActiveMin` sobre índices de lane, que derrubou a GPU numa RTX 5090, sai. Confere. |
| #348 | tools: check float compares and v_cmpx over whole waves with a partly active EXEC | `10b48fd4` | `check_float_compares_gpu.py` (515 linhas) e `test_float_compares_reference.py` (108). Confere. |
| #349 | launcher: keep helper processes from cleaning an exported NVIDIA shader cache | `2d7e536a` | `keep_nvidia_shader_cache`, chamado no `pipeline_prep` e com o Xwayland privado sem a variável. Confere. |
| #330 | present checks: sample the screen in physical pixels | `71ad0fd4` | `LogicalToPhysicalPointForPerMonitorDPI` nos dois pontos de amostragem (`panel.cpp`, `frame_present.cpp`). Confere. |
| #331 | tests: skip the Linux-only launcher tests on Windows | `f8f79f2d` | `skipUnless(sys.platform == "linux")` em cinco arquivos de teste e `create=True` nos patches de `sched_getaffinity`. Confere. |
| #332 | status: write the tables as UTF-8 whatever the console's code page | `9d81680d` | `sys.stdout.reconfigure(encoding="utf-8")` + teste com `PYTHONIOENCODING=cp1252`. Confere. |
| #335 | d3d12: give residency priorities only to pageables the host takes | `a905f69d` | Lista filtrada, `levels` paralelo, `mark_committed` em `create_resource`. Confere com o título e com o corpo (o device era removido com `DXGI_ERROR_INVALID_CALL`). |
| #337 | d3d12: name the earlier writes in the indirect argument barrier | `1d5c847c` | `barrier(list_type)` com `UNORDERED_ACCESS|COPY_DEST`, mais `STREAM_OUTPUT|RESOLVE_DEST` em lista direta. Confere. |
| #338 | d3d12: return a null GPU handle for a heap shaders can't see | `5af445df` | `cpu_handle` devolve 0 sem `SHADER_VISIBLE`. Confere. |
| #339 | xmem: walk an NT heap for its statistics | `886376ee` | `native.bytes[0] != 0` como condição para aceitar os contadores de segmento. Confere. |
| #340 | xmem guest: take a guard page with a real access where ReadProcessMemory leaves it | `f36b59b3` | `IsBadReadPtr` + `VirtualQuery` no guest. Confere. |
| #341 | d3d12: fall back to null UAV and aliasing barriers for a cache barrier | `903567aa` | `legacy_full_barrier` com UAV nula, aliasing nula e `log_once`. Confere. |
| #346 | xshader: write smaller HLSL for the two-lane form of whole console waves | `9fba2d51` | `vN__[2] = {0u, 0u}` e instrução escrita uma vez. Confere. |
| #350 | xshader: read inactive lanes in a two-lane row permute as its modifiers say | `26d0f6df` | `flags & 4` recusado, bit 0 busca inativa, bit 1 zera; bit 3 aceito. Confere. |

Os outros 30 merges da janela (`#351`–`#357`, `#329`, `#345` e a série de 12:xx) ficaram **fora** do
recorte de 40 commits: são merge commits posteriores ao 40º documentado e o cursor não os cobre.

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum hash do catálogo na janela (148 commits + 47 merges conferidos por prefixo
  contra os 262 hashes das 305 linhas).
- **Por área, com prova no diff:**
  - **D-04** (cache de pipeline da NVIDIA no lugar errado e com teto): `0e2e6ef3` não é D-04 — D-04 é
    sobre onde e com que teto o cache vive; o commit é sobre **quem apaga** o cache. É a mesma
    superfície (`__GL_SHADER_DISK_CACHE_*`) e o diff mostra o mecanismo de deleção, mas não fecha D-04.
  - **D-22** (root raster com várias tabelas): o aberto **#344** propõe exatamente anexar os slots que
    o root raster deixa de fora. Sem merge na janela.
  - **D-28** (`WaveSize(N)` incompatível só recusado no driver): a forma two-lane emite
    `[WaveSize(32)]` e a série desta hora amplia quando ela é usada (`takes_over`). `80fcabe8` também
    decide largura de máscara por análise. Não há recusa nova no diff; não afirmo correção.
- **Abre defeito novo?** Um candidato com evidência: `89587e41` rebaixa a barra de progresso de
  94,2% para 76,2% porque a métrica mudou de "entradas que produziram saída" para "instruções que
  compilam para DXIL". Não é defeito de código; é mudança de régua e está declarada no diff.
  `8f449025` admite que o decodificador BC7 do TuxBox difere do VIctory em 85% dos blocos — é limite
  declarado, não defeito do catálogo.

## Confiança

**Verificado:** os 40 commits foram lidos pelo diff (`repos/brcly/TuxBox/commits/<sha>`, 13:15–13:35Z),
com arquivos, contagens e trechos de patch; os merges foram conferidos pelo assunto e pelo diff de
dois pais (quando o merge só repete o commit já descrito, isso está dito). Os 14 PRs abertos acima de
#339 e os 47 mesclados desde `2026-11-28T00:05:08Z` vieram da API REST (`pulls?state=…`), com 17
casando por `merge_commit_sha` com a janela.

**Não verificado:** nenhuma build, nenhum teste e nenhuma execução — os números de ULP, de pipelines e
de cobertura de instrução são os que os commits e o README declaram. O corpo de #1504/#1513 do
`victory-fork` não tem relação com este repositório. Os 400 chunks de BC1 e as amostras de BC3/4/5/7
não foram reproduzidos.

**Sobreposição e teto:** o `commit_cursor` gravado (`8420ba03`) apontava para o 40º commit do ciclo
anterior, mas a ordem do `compare` não é a ordem em que aquele ciclo escreveu. Medi a interseção: os
19 primeiros da página já estavam no documento das 12:00; a janela desta hora começa no índice 19
(`39fa7de8`) e vai até o índice 58 (`26d0f6df`), com **zero** repetição. O próximo cursor é
`02fceb26` (índice 59) e restam 108 commits dos 148.

**Divergência de contagem:** 47 PRs foram mesclados desde `last_run`, mas só 17 têm `merge_commit_sha`
dentro da janela de 40 commits. `last_run` avança para o maior `merged_at` **documentado**
(`2026-11-28T01:16:56Z`, #350); os 30 restantes continuam pendentes e o próximo ciclo os vê de novo.
