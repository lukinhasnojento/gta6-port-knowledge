# R0Y00/Xbox2Windows — atualização de 2026-11-27 18:00

## Resumo

Onze commits desde `801f4c3b`: os sete que a execução das 17:00 deixou acima do teto de 40
(`db28a796`, `ff04a75e`, `246857e4`, `36d2cbeb`, `2972e00a`, `9072230f`, `e1ea515d`) e os quatro que
entraram depois do `last_run` (`2a4bbff5`, `020c7a82`, `ca659396`, `de1595b9`). Cinco dos onze mudam
o tradutor direto de compute (`v_cvt_pk_u8_f32`, `v_dot4`, `v_dot2`, `v_ldexp_f32`, shifts de 64 bits);
três são só teste ou documentação; um é ferramenta nova (manifesto de patch com gate de hash) e um
torna opcional a cópia de root SRV do slot 44. `246857e459` é a correção de uma regressão do próprio
`#2997` medida em driver NVIDIA (#3052), não um ajuste cosmético.

## Commits novos (11)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `db28a79627` | RonLogic | gpu: verify translated compute writes to two texture mips (#2882) | Só teste: `tests/startup_compute_dispatch_texture_check.py` +91/−9 e `tests/startup_stream_consumer.cpp` +47/−25 montam um fixture de dispatch do slot 67 com duas views UAV em mips separados de uma textura, conferem todas as meias palavras depois da fence e exercitam dois controles negativos (tradução ausente e views trocadas) em WARP e em hardware. `CMakeLists.txt` +4/−1 e `docs/experiments.md` +3/−1 registram o alvo. Nenhuma linha de produção. |
| `ff04a75e58` | saii | tooling: add a hash-gated patch manifest loader (#3056) | Novo `tools/patch_manifest.py` (128 linhas): lê TOML com `[target] input_sha256` e uma ou mais `[[patch]]` (`id`, `kind`, `why`, `remove_when`, `rva`, `expect_sha256`, `bytes`), valida chaves desconhecidas e ausentes, exige 64 dígitos hexadecimais minúsculos, confere o `expect_sha256` dos bytes originais e o SHA-256 do arquivo de entrada **antes** de devolver qualquer plano. O único `kind` definido é `memory`, e o próprio docstring diz que nada o aplica ainda. `tests/patch_manifest_check.py` +112 cobre manifesto malformado, a fronteira de 32 bits e hash de entrada errado. |
| `246857e459` | RonLogic | shader: round packed U8 conversions with integer arithmetic (#3058) | Reescreve a emissão de `v_cvt_pk_u8_f32` em `src/gfx1013_direct_compute.inl` (+17/−9). A versão anterior (`#2997`) usava `Round_ne` e `fptoui` do DXIL; o comentário novo registra que o literal `0.5` por `Round_ne` + `fptoui` volta como `1` em alguns drivers NVIDIA (#3052). Passa a fazer tudo em inteiro sobre os bits: `0x3f000000` (0.5) e `0x437f0000` (255.0) como limites, `discard = 150 − (bits >> 23)` (23 + bias), significando com o bit implícito `| 0x800000`, truncamento, viés `halfway − 1` e soma do bit baixo retido — só empates ímpares sobem (ties-to-even) —, preservando os outros bytes do destino. |
| `36d2cbebde` | lukaslw-lab | startup: make unused graphics root SRV copies optional (#3028) | Em `src/startup_stubs/d3d12_xs_device.cpp` +33: `X2W_D3D12_ROOT_SRV_AT_DRAW=1` faz `list_root_srv` pular a cópia de uma janela de 64 KiB para o upload ring quando não há mapeamento (`readable_window(address, 0x10000)` deixa de ser chamado), deixando o parâmetro sem bind e contando em `root_srv_copies_skipped`/`_made`. O comentário justifica: todo pipeline gráfico de desktop é indireto e os draws traduzidos leem o SRV de `root_address`; um pipeline de desktop que lesse root SRV precisaria da cópia. Fixes #3027, refs #2948 e #2949. |
| `2972e00aaf` | pigalot | startup: read single-mip 0x105 volumes in slot 70 (#2871) | `src/startup_stubs/gfx10_layout.h` +91/−16 ganha `supports_volume()` (TEXTURE3D, swizzle mode 5, formato com `texels == 1`, dimensões até 2048, `MipLevels == 1`, uma amostra) e o `Layout` passa a guardar `depth`, `block_depth`, `blocks_high`, `z_bits` e `layer_size` para blocos grossos. `d3d12_draw.inl` +4/−1 exige `Footprint.Depth == layers(i)` no plano e passa o passo de camada (`RowPitch * rows`) ao `detile`, para cada camada de profundidade cair nas suas próprias linhas; `d3d12_memory_srv.inl` +4/−2 passa a contar um volume como uma fatia na checagem de tamanho reportado. Teste novo `tests/startup_gfx10_volume_check.cpp` +362, com controles de layout fino e de volume grande demais. |
| `9072230f86` | RonLogic | shader: lower packed byte dot products in direct compute (#3060) | Adiciona `saturated_add()` (saturação sem sinal por comparação `ult` com a soma; com sinal pelo teste de `(a^sum) & (b^sum) < 0`) e baixa `v_dot4_i32_i8`/`v_dot4_u32_u8` por `dx.op.dot4AddPacked` (163/164), recusando VOP3P que não seja `OP_SEL`/`OP_SEL_HI` canônico e `NEG_LO`/`NEG_HI` não zero. Com `clamp`, satura só a soma final ao acumulador — o dot de quatro produtos 255×255 já cabe em i32. |
| `e1ea515d60` | RonLogic | shader: lower packed halfword dot products in direct compute (#3061) | Baixa `v_dot2_i32_i16`/`v_dot2_u32_u16` sem intrínseco: extrai as metades (`shl 16` + `ashr 16` para com sinal, `and 65535`/`lshr 16` para sem), multiplica, soma e, com `clamp`, rastreia os dois carries e as três extensões de sinal na palavra alta (`carry(result,left)` por `zext (ult ...)`) antes de saturar — a saturação ocorre depois do acumulador. |
| `2a4bbff532` | saii | docs: record DXC GVN pass cost and sampler readback (#3059) | Só documentação: `docs/experiments/dxc-gvn-off-warp.md` +102 registra a fração do tempo de passes do DXC consumida por GVN em `samplebranch_4096/8192/16384` (63,1 %, 77,1 %, 84,9 %), o tamanho do DXIL com GVN desligado (+18,0 %, +15,0 %, +14,0 %) e o resultado do oráculo de sampler em WARP e RTX 4090, separando a execução bloqueada do corpo não medido. Nenhum código. |
| `020c7a8215` | legacy-code-archaeologist | shader: lower v_ldexp_f32 in direct compute (#3057) | `src/gfx1013_direct_compute.inl` +53: baixa `v_ldexp_f32` em inteiro — separa sinal, expoente e mantissa, deixa zero/Inf/NaN passarem (`exponent == 255 \|\| (bits & 0x7fffffff) == 0`), normaliza denormal por cinco deslocamentos condicionais (16, 8, 4, 2, 1), limita S1 a [−512, 512], arredonda resultado denormal ao par mais próximo e, com `clamp`, satura por comparação inteira (NaN, negativo e −0 → +0; ≥ 1.0 → 1.0). Recusa `omod`, `op_sel`, `abs`/`neg` em S1/S2, `src2` não nulo, SDWA e DPP. Fixes #3013. |
| `ca65939687` | legacy-code-archaeologist | shader: lower 64-bit scalar shifts in direct compute (#3067) | `src/gfx1013_compute_native.cpp` +2/−1: `transfer()` passa a limpar também o par de destino de instruções `_i64`, não só `_b64`. Em `gfx1013_direct_compute.inl` +35/−1 entram `s_lshl_b64`, `s_lshr_b64` e `s_ashr_i64`: destino par alinhado até `s104` ou `vcc` (106), fontes validadas (par alinhado, inteiro inline, literal de 32 bits com extensão de sinal só no `ashr`), leitura de todas as fontes antes de escrever o par (destino sobreposto), `SCC = (D != 0)` e `Int64Ops` (bit 1048576) ligado. Fixes #3062. |
| `de1595b96b` | RonLogic | shader: verify derivative-selected gather across a mip view (#3065) | Só teste: fixture diferencial em textura 8×8 com SRV de dois níveis começando no mip 1 do recurso; um estágio de pixel sintético leva o `image_get_lod` a `image_gather4_l` e, com UV fixo em (9/16, 9/16) e passos de 1/4 e 1/2, seleciona mips 0 e 1 relativos à view. Cada draw confere quatro taps contra a referência de CPU; o controle negativo liga a view de dois níveis no gather de mip fixo e discorda nos quatro taps. Arquivos: `docs/experiments/startup-gather-implicit-mip.md` +30, `startup_gather_implicit_mip_check.py` +89, `offscreen_check.cpp` +12/−6, `startup_image_3d.cpp` +6/−6 e `startup_image_get_lod_view_check.py` +5/−3. |

## Pull requests abertos (14)

Todos com base `main`, exceto #3094 (`codex/native-shader-modules`). Janela: números acima de 3071.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #3073 | shader: lower 64-bit scalar bit counts and bit scans in the direct compute path | Baixa `s_bcnt0_i32_b64`, `s_bcnt1_i32_b64`, `s_ff0/ff1_i32_b64`, `s_flbit_i32_b64/i64` com `unaryBits.i32` (31/32/33), espelhando `gfx2hlsl.py`; +346/−3 em 5 arquivos, com CTest novo. Fixes #3072. | Não há D-xx de contagem de bits; é a família #2935 do tradutor direto (área de D-05/D-06). |
| #3079 | shader: lower v_dot8_i32_i4 and v_dot8_u32_u4 in the direct compute path | Extrai oito nibbles (com sinal por `shl`/`ashr 28`, sem por `lshr`/`and 15`), soma oito produtos e satura só a soma final com `clamp`; +198 em 4 arquivos. Fixes #3077. | Área do catálogo `VItoriaBr D-15` (`v_dot8_i32_i4` ausente da tabela de opcodes), mas em outro repositório e outro caminho. |
| #3081 | shader: lower s_cmp_eq_u64, s_cmp_lg_u64, s_bitcmp0_b64 and s_bitcmp1_b64 | Comparações e bitcmp de 64 bits com operações de 32 bits, sem exigir `Int64Ops`; só escreve `SCC`; recusa SGPR ímpar, `vcc`, `exec`, `m0`, `scc`. +314 em 4 arquivos. Fixes #3078. | Área de D-07 (offset imediato SMEM) não; mesma família de escalares de 64 bits do #3067. |
| #3082 | shader: lower v_pk_lshlrev_b16, v_pk_lshrrev_b16 and v_pk_ashrrev_i16 | Shifts 16 bits empacotados com `op_sel`/`op_sel_hi`, contagem mascarada em `& 15`, `ashr` assinado por `ashr (16 + amt)`; recusa `clamp`, `neg_lo/neg_hi`, `op_sel[2]`. Fixes #3080. | Não; família #2935. |
| #3083 | startup: read smpc comparison samplers in published compute manifests (#3075) | `load_compute_translation` deixa de recusar toda linha `smpc` e a lê por `TranslatedSamplers::read_manifest_line` com os mesmos testes de índice e linha inteira de `smp`/`smpl`; `smpheap` continua recusado. +202/−22 em 5 arquivos. Fixes #3075. | Área de amostragem/descritores (D-27 é `image_sample` 2D `a16`); nenhum hash do catálogo. |
| #3084 | runtime: honor Opus activation, trimming and planar output | Passa a honrar contagem de canais na ativação de streams Opus tipados, decodifica a 48 kHz, aplica trim de quadros iniciais e capacidade de saída, publica o registro de conclusão e suporta saída planar e intercalada em estéreo. +169/−26. | Sem D-xx de Opus no catálogo; área de XAudio/XApu. |
| #3087 | startup: add entry-only tracing for offline file operations | `X2W_OFFLINE_ENTRY_ONLY=1` omite o log de conclusão das operações de arquivo offline permitidas, mantendo as checagens de device na entrada; 5 linhas em `tools/startup/debug_launch.py`. | Não. |
| #3088 | shader: lower v_pk_add/sub_i16 and v_pk_add/sub_u16 | Quatro opcodes VOP3P de soma/subtração 16 bits, com `op_sel`/`op_sel_hi`, wrap módulo 2^16 e saturação por lane com `clamp`; recusa `neg_lo/neg_hi`, `op_sel[2]` e `src2` não nulo. Fixes #3085. | Não; família #2935. |
| #3089 | shader: lower s_bfe_u64, s_bfe_i64 and s_brev_b64 | `s_brev_b64` por `unary.i32 30` (reversebits) trocando e invertendo as metades, sem `Int64Ops`; `s_bfe_*_64` com largura de S1[22:16] e offset de S1[5:0], clamp em 64, `SCC = (D != 0)`. Fixes #3086. | Não. |
| #3091 | shader: compile native mesh interfaces and prewarm compute requests | Integra compilação nativa das interfaces fixas mesh/pixel, waterfalls de descritor e layouts revisados; o frontend C++ aceita estado de compute etiquetado e janelas escalares de buffer e publica/valida um pack de frontend versionado; a identidade do cache inclui as fontes de lowering e invalida quando o sidecar de entrada de pixel do slot 57 muda. +3.557/−235 em 35 arquivos. | Área de mesh/pixel (D-21 é de outra base); é o pré-requisito de compilador para a integração de runtime separada (#2981). |
| #3092 | shader: lower v_pk_mul_lo_u16, v_pk_mad_u16 and v_pk_mad_i16 | Multiplicação baixa e mad de 16 bits empacotados, 2 e 3 fontes, wrap módulo 2^16 e saturação por lane; recusa `neg_lo/neg_hi` nos três. Fixes #3090. | Não. |
| #3094 | runtime: bind native shader requests and submit fixed mesh pipelines | Depende de #3091 e mira a branch dele: prepara pares VS/PS suportados no primeiro uso, liga entradas de vértice fixas e pipelines de mesh do slot 57, consome pedidos congelados do slot 67 e packs de frontend; publica tradução gráfica temporária só depois de manifesto e root signature. +2.442/−118 em 21 arquivos. | Área de mesh/root signature (D-21, D-22 em TuxBox). Base não-default: não entra no cursor da `main`. |
| #3095 | shader: lower v_sat_pk_u8_i16 in the direct compute path | VOP1 `0x62` e VOP3A `0x1e2`: estende o sinal das duas metades de `src0`, limita a [0,255], empacota em `D[7:0]`/`D[15:8]` e zera `D[31:16]`; recusa `clamp`, `omod`, `abs`, `neg`, `op_sel`, SDWA, DPP e fontes não usadas não nulas. Fixes #3093. | Não. |
| #3097 | startup: opt-in guarded write of the view token in note_view | `X2W_D3D12_VIEW_TOKEN_GUARDED=1` escreve o token da view sob guarda de exceção (`write_view_token`) em vez de `commit_lazy` + `VirtualQuery` por chamada; fixa o custo por chamada de `note_view`. Fixes #3096. | Área de XMem/página (D-09 é leitura de trap log sem compartilhamento); nenhum hash do catálogo. |

## Pull requests mesclados (4)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #3057 | shader: lower v_ldexp_f32 in the direct compute path | `020c7a82` | O PR prometia exatamente o que entrou: 53 linhas de lowering em `gfx1013_direct_compute.inl`, doc de 44 linhas e 176 linhas de teste com fixture CTest. `merge_commit_sha` é o próprio commit squash. |
| #3059 | docs: record DXC without GVN on WARP and its pass shares | `2a4bbff5` | Um arquivo, +102, só documentação — como o PR declarava ("Docs only, no code change"). |
| #3065 | shader: verify derivative-selected gather across a mip view | `de1595b9` | Entrou a verificação prometida (151/−16 em 7 arquivos), incluindo a mudança de `tests/desktop_gpu.h` +1/−1 e a de `tests/startup_image_3d.cpp` +6/−6; nenhum código de produção, coerente com o corpo do PR. |
| #3067 | shader: lower s_lshl_b64, s_lshr_b64 and s_ashr_i64 in the direct compute path | `ca659396` | 288/−2 em 5 arquivos: os três opcodes, o ajuste de `transfer()` para `_i64` e o fixture de teste. O PR pedia `Int64Ops`; o commit liga o bit 1048576 — conferido no patch. |

## Cruzamento com o catálogo

- **Regressão de `#2997` corrigida** (`246857e459`): a conversão U8 empacotada que o catálogo não
  registrava como defeito passa a usar aritmética inteira porque `Round_ne`/`fptoui` devolvem 1 em
  drivers NVIDIA (#3052). Não é D-28 (esse é `UNORM/SNORM` de store formatado, `f37fe7b8`), mas é a
  mesma classe — arredondamento de conversão de formato — e merece entrada própria se o efeito em
  título for observado.
- **Família #2935** (`9072230f`, `e1ea515d`, `020c7a82`, `ca659396` e os PRs #3073 a #3095): nenhum
  hash de `introduzido`/`corrigido` do catálogo casa. O `refuse()` novo de cada lowering recusa
  modificadores não canônicos — se um shader do título usar `neg_lo` em `v_pk_mad`, o programa cai no
  caminho de recusa em vez de ser traduzido. Não há caso de título citado; fica como risco, não como
  defeito.
- **D-09 / D-17 (área de startup)**: `36d2cbeb` mexe no caminho de `startup_view_heap`/root SRV do
  slot 44 e `2972e00a` no upload do slot 70. Ambos são opt-in ou caminho novo, sem tocar os hashes do
  catálogo.
- **Defeito novo**: nenhum identificado por diff. `ff04a75e` cria um aplicador de patches de memória
  com gate de SHA-256 — mecanismo novo, sem entrada no catálogo.

## Confiança

Verificado: os 11 diffs foram lidos pela API (`commits/<sha>` com patch) e os quatro PRs mesclados
foram conferidos contra o `merge_commit_sha`; a contagem de commits da janela bate com a de merges
(4 PRs = 4 dos 11 commits). Os outros sete commits citam número de PR no assunto mas têm `merged_at`
anterior ao cursor das 13:55:58 — foram os que ficaram acima do teto das 17:00, e não uma divergência
de coleta.

Não verificado: (i) nenhum build, `ctest` ou execução de GPU foi rodado; números de linhas e
asserções vêm do patch; (ii) os 14 PRs abertos foram lidos pelo corpo (até ~1.100 caracteres) e pela
lista de arquivos, nenhum pelo patch linha a linha; (iii) `#3094` tem base `codex/native-shader-modules`
e não entra no cursor de commits da `main` — o texto do PR diz que será retargetado depois de #3091;
(iv) `2972e00a` não cita `Fixes`/`Refs` no assunto além de `(#2871)`; a ligação com issue não foi
conferida.
