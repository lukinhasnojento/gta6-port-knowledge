# Marvisak/VIctory — atualização de 2026-11-25 23:00

## Resumo

Nenhum commit novo na `main` desde o cursor (`63bcca43c6a00a758510398fa43096ce7e4f7df0`, ainda o
topo da branch). O delta da hora são 33 pull requests abertos, quase todos de desempenho de
`victory-gpu`/`victory-shader` (RAM, pipelines, marcação de escrita) e de conformidade da camada
D3D12. Duas propostas vêm de fora do próprio fork (`#2960` portado de `fxceless/victory-fork`).

## Commits novos (0)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| — | — | — | nenhum commit novo: `63bcca43c6a00a758510398fa43096ce7e4f7df0` continua sendo o topo de `main` na API |

## Pull requests abertos (33)

Todas as linhas vêm do título e do corpo do PR. O diff de nenhum deles foi lido nesta execução —
o que está na coluna "Proposta" é o que o autor declara, não o que o código faz.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 2944 | GPU: cap default pipeline worker count on <=24 GiB RAM hosts | Limitar `(cores/2).clamp(2,8)` workers de pipeline conforme RAM física instalada | Não consta no catálogo (pressão de commit memory) |
| 2945 | GPU: title memory in place by default where host GPU shares memory | Memória do título in place em APU/GPU compartilhada; empilhado em #2112, #2912, #2921 | Não |
| 2946 | GPU tests: the work run while a view is prepared is per thread | Corrigir teste que falha aleatoriamente em CI (`both made`) | Não; é flakiness de teste, não D-25 (testes VALU travando) |
| 2947 | GPU: reject unknown shader model feature requests | `CheckFeatureSupport(SHADER_MODEL)` devolver `E_INVALIDARG` para modelos indefinidos | Não consta |
| 2948 | Shaders, GPU: a root UAV a shader only reads is not written back | Não declarar escrito um root UAV só lido: evita snapshot + comparação de até 1 MiB | Não é D-07 (copy-back sobrescrevia escrita da CPU); é ganho de banda |
| 2950 | Game: the shader pack in an archive inflates through zlib-rs | Trocar `miniz_oxide` por `zlib-rs` na inflação do shader pack do arquivo | Não consta |
| 2951 | victory: translate --cache and the pre-build say where their time went | Instrumentar `translate --cache --pipelines`: shaders/pipelines mais lentos, saves da library | Não consta |
| 2952 | GPU: disable implicit stream depth test without a DSV format | Seguir o parser de stream do D3D12: `DepthEnable` default só com `DSVFormat` | Não consta (conformidade) |
| 2953 | Shaders: lower binary64 division fixup in DXIL | VOP3 0x160 (`v_div_fixup_f64`) com seleção de dwords por inteiro | Não consta |
| 2954 | Shaders: lower v_bfe_u32/v_bfe_i32 to one Ubfe/Ibfe | Uma chamada DXIL por instrução vetorial, mais teste de contagem | Não consta |
| 2955 | GPU/Placement: defer free() reap, on-demand place() reap, O(1) heap metadata, TLS fits cache | Remover reaping O(P²), varreduras O(H·F) e `GetResourceAllocationInfo` duplicado | Não; toca a área de D-01/D-02 (descritores/heap de views) sem declarar correção delas |
| 2956 | Loader/TLS: use memchr::memmem SIMD scan in tls::patch | Varredura SIMD do padrão `mov r64, gs:[0x58]` em `.text`, saída bit-idêntica | Não é D-04 (ponteiro de callback TLS zerado), mas é o mesmo módulo `tls.rs` |
| 2957 | Shaders/IR: pruned SSA via bitset IDF + dominator-tree renaming | SSA podada por bitsets e renomeação por árvore de dominadores, saída estruturalmente igual | Não consta |
| 2958 | Shaders: one lane of a wave sets a mark its wave shares (411 → 283 ms em RTX 4070 SUPER) | Marca de escrita/leitura por wave em vez de por lane, com OR atômico único | Não consta (anoata que mexe em `mark_view_write`/`mark_view_read`) |
| 2959 | GPU: report pipeline creation and loads after they succeed | Só logar `built … loaded` quando o create/load realmente sucedeu | Não consta |
| 2960 | Cut the 17 s world frames with asynchronous submission and a copy queue | Porta de `fxceless/victory-fork`: submissão assíncrona + fila de cópia; 17 s → 0,3–2 s (RTX 4060 8 GB) | Não; é a mesma linha de trabalho dos runs 281–292 de `victory-fork` |
| 2961 | GPU: reject shader-visible RTV and DSV heap creation | `CreateDescriptorHeap` com RTV/DSV + `SHADER_VISIBLE` devolve `E_INVALIDARG` | Não consta |
| 2962 | GPU: sample packed 10-bit frames for black-frame diagnostics | Amostrar R10G10B10A2 no probe de frame preto (antes só 8 bits) | Não consta |
| 2963 | Shaders: allow abs/neg source modifiers on v_cvt_{u16,i16}_f16 | Aceitar `abs`/`neg` no VOP3 de conversão f16→int, com teste sintético | Não consta |
| 2964 | Shader tests: serialize remaining Host::new tests with ONE_AT_A_TIME | Serializar os `Host::new` restantes em testes (#1882/#1892) | Não |
| 2965 | GPU: reject mismatched command list allocators | Exigir allocator do mesmo tipo da lista em create/Reset | Não consta |
| 2966 | GPU watch: ignore zero-length writes and remaps on unaligned addresses (#2011) | Curto-circuitar `len == 0` em `watch::writing`, `Drop for Writing` e `title_memory_remapped` | Aborda a issue #2011; não há linha correspondente no catálogo |
| 2967 | Shaders: track spilled lane masks, Atom::Phi back-edges, and carry masks | Fechar lacunas de lane-mask do item 4 da #2268 (writelane/readlane, phi) | Não é D-06 (descritor de `v_readfirstlane`), é máscara de lane |
| 2968 | Shaders: f32 floor, ceil and fract flush denormal operands | Emular flush de denormal do console em `v_floor/ceil/fract_f32` | Vizinho de D-23 (`fract()` corrigido e quebrado na NVIDIA), defeito diferente |
| 2969 | GPU: shaders of our own that Wine's compiler builds too | Evitar métodos em `RWByteAddressBuffer` e `[RootSignature]` que o `d3dcompiler_47` do Wine não aceita | Não consta (caminho Wine) |
| 2970 | Shaders: keep v0 as the leading vertex of both rectangle triangles | Provoking vertex do segundo triângulo do strip, como o console | Não consta |
| 2971 | GPU: store HostHeap kept/made sparsely and allocate strides/marks lazily | `HashMap` esparso no lugar de Vecs de ~1,26 M por heap de views | Toca D-02 (teto de 1.000.000 do heap de views); não declara corrigi-lo |
| 2972 | GPU: reclaim idle staging, scratch, and arena buffers and evict on system memory pressure | Trimar chunks de staging após 64 submissões ociosas e evictar sob pressão | Não; mesma família de D-13 do catálogo Belobug-AMD (consumo de memória), repo diferente |
| 2973 | Tests and fixes so the suite runs under Wine | Correções reais + skips para `cargo xtask wine-test` (#1218), empilhado em #2969 | Não |
| 2974 | GPU: reduce shader and pipeline RAM usage (Weak cache, deflate, 32 MiB DXIL cache) | Cache `Weak<Shader>` com poda, liberar `Pending::graphics`, teto de 32 MiB no cache DXIL | Não |
| 2975 | Shaders: lower v_div_scale_f64 and its lane mask | VOP3B `v_div_scale_f64`, expoentes medidos, quatro modos de arredondamento f64 | Não consta |
| 2977 | Shaders: one back edge per block, for the dxil-spirv in Proton | Evitar latch que volta ao header do próprio loop e ao externo (`LoopMergeTracer`) | Não consta; é travamento na criação de pipeline no Proton |
| 2978 | Shaders: whole-wave sections follow binary64 sums (WaveActiveSum of a double) | Seções whole-wave para somas f64, empilhado em #2348 | Não consta |

Observação de numeração: a faixa aberta hoje vai de #2944 a #2978, com lacunas em #2949 e #2976. O
maior número aberto é **2978**.

## Pull requests mesclados (0)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| — | — | — | nenhum PR com número > 2943 aparece como mesclado nesta consulta |

## Cruzamento com o catálogo

- Não há commit novo, então **nenhum defeito do catálogo pode ter sido corrigido por hash nesta
  hora** para este repositório.
- Os PRs abertos que tocam áreas com defeito catalogado: #2955 (D-01/D-02, sem declarar correção),
  #2971 (D-02, alocação esparsa do heap de views), #2968 (vizinho de D-23), #2967 (vizinho de D-06),
  #2956 (mesmo `tls.rs` de D-04). Nenhum desses pode ser dado como correção: o diff não foi lido.
- Possível defeito novo declarado pelo próprio autor: #2959 diz que `VICTORY_PIPELINE_LOG` gravava
  `built … loaded` para pipeline que o driver acabara de criar — log que mentia. Não existe linha
  para isso no catálogo (candidato a D-28 de VIctory, se confirmado no diff).

## Confiança

- **Verificado:** ausência de commit novo (comparação da lista de `repos/Marvisak/VIctory/commits`
  com o cursor `63bcca4…`, âncora no índice 0 de 100).
- **Verificado:** existência e numeração dos 33 PRs abertos (API `pulls?state=open`, faixa
  #2944–#2978), títulos e autores.
- **NAO VERIFICADO:** o conteúdo dos diffs dos PRs. As colunas "Proposta" e "Cobre defeito
  conhecido?" são o que o corpo do PR declara. Nenhum arquivo, linha ou hash de commit de PR foi
  inspecionado — os PRs não têm merge commit.
- **NAO VERIFICADO:** se algum dos 33 PRs modifica arquivo citado em `catalogo/defeitos.csv`.
