# R0Y00/Xbox2Windows — atualização de 2026-11-27 19:00

## Resumo

Dezesseis commits na `main` desde `de1595b9`, quinze deles o mesmo trabalho: fechar as recusas
`direct DXIL lowering for <name>` do tradutor direto de compute, instrução por instrução, cada uma
com controle de compilação e um teste de hardware que compara a leitura de volta com uma referência
escrita à parte (série #2935). O décimo sexto é de outra natureza: `#2919` grava em disco o estado de
draw que o cache de shader não guarda e recria o PSO a partir dele na execução seguinte. Dezesseis
PRs mesclados, todos em squash (o `merge_commit_sha` é o próprio commit). Head avançou para
`2641cc72` durante a coleta (#3114, quadmask escalar), que fica para o próximo ciclo.

## Commits novos (16)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `a9fd1488` | RonLogic | shader: lower explicit-LOD 2D gather in direct DXIL (#3068) | Em `src/gfx1013_direct_compute.inl` (+88/−7): novo `texture_dimensions()` via `dx.op.getDimensions` (opcode 72) com mip relativo à view; `explicit_gather_2d()` exige sampler constante nos 4 dwords de SGPR e recusa qualquer forma que não seja clamp normalizado com filtro de mip point; `lod = floor(lod + 0.5)` limitado ao máximo do sampler e ao último mip visível; quatro loads tipados ordenados LL/LR/UR/UL; coordenada ou LOD não finito grava zero nas lanes ativas. Teste `native_gather_lod_check.py` cobre pixel e compute, SRV de 2 mips começando no mip 1 do recurso, quatro lanes com LODs diferentes num dispatch, e um controle forçado a mip 0 que falha. Fixes #3066, refs #1802. |
| `5df098da` | Roland Edi | startup: persist bounded translated PSO recipes (#2919) | Novo `src/startup_stubs/pso_recipe.h` (653 linhas) e +231/−38 em `d3d12_draw.inl`. O cache de shader guarda bytecode traduzido, root signature e samplers, mas não o estado de draw que produz o PSO; `X2W_PSO_RECIPE_DIR` (desligado por padrão) grava, após um `CreateGraphicsPipelineState` bem-sucedido, o blend efetivo (depois das máscaras de escrita dos exports), depth-stencil, rasterizer, formatos RTV/DSV, sample mask/count e topologia. Um arquivo texto por chave de tradução, teto de 64 registros e 64 KiB, leitura/checagem/append sob um único handle exclusivo (sem ponteiros nem endereços de processo). O parser recalcula a chave de estado e ignora registro cuja chave recomputada não bate, cujo formato/DSV não é o do pass atual ou cuja versão é desconhecida; o escritor aplica o mesmo parser antes de anexar e recusa o 65º registro. O replay chama `CreateGraphicsPipelineState` — é reconstrução de estado, não pipeline library. |
| `0df67295` | legacy-code-archaeologist | shader: lower 64-bit scalar bitfield extract and reverse (#3089) | `s_brev_b64` vira `@dx.op.unary.i32` opcode 30 (`reversebits`) nas duas metades, com troca de low/high antes de escrever o par SGPR par, sem tocar SCC; `s_bfe_u64`/`s_bfe_i64` passam a ser lowering só de 32 bits. `src/gfx1013_compute_native.cpp` (+3/−1) inclui `_u64` na regra simbólica de limpeza de par em `transfer()`. Fixes #3086. |
| `8929423c` | rdx | shader: lower v_cvt_off_f32_i4 in direct compute (#3099) | `v_cvt_off_f32_i4` nas duas codificações (VOP1 `0x0e`, VOP3A `0x18e`): estende `src0[3:0]` com sinal para `[-8,+7]` e multiplica por `0.0625`, dando valores exatos em `[-0.5,+0.4375]`; `omod` e `clamp` do VOP3A aplicam-se ao float; recusa `abs`/`neg`, `op_sel` e `src2` não usado. Fixes #3098. |
| `7a12818c` | rdx | shader: lower 24-bit integer multiplies in direct compute (#3102) | `v_mul_i32_i24`, `v_mul_hi_i32_i24`, `v_mul_hi_u32_u24`, em VOP2 (`0x09/0x0a/0x0c`) e VOP3A (`0x109/0x10a/0x10c`): lê só os 24 bits baixos de cada fonte (estende conforme `i24`/`u24`), devolve `[31:0]` do produto ou `[63:32]` via split 12+12 em aritmética de 32 bits. Fixes #3101. |
| `428c08c1` | rdx | shader: lower packed 16-bit multiply and MAD in direct compute (#3092) | VOP3P `v_pk_mul_lo_u16` (`0x01`), `v_pk_mad_u16` (`0x09`) e `v_pk_mad_i16` (`0x00`): duas lanes de 16 bits independentes selecionadas por `op_sel` e `op_sel_hi`; produto wrap mod 2^16; o mul recusa `clamp`, `op_sel[2]` e `src2` não zero. Fixes #3090. |
| `4a6302eb` | rdx | Lower packed 16-bit shifts in the direct compute shader path (#3082) | VOP3P `v_pk_lshlrev_b16` (`0x04`), `v_pk_lshrrev_b16` (`0x05`), `v_pk_ashrrev_i16` (`0x06`): desloca as duas lanes de 16 bits de `src1` pelo valor `& 15` das lanes correspondentes de `src0`, por half de saída. Fixes #3080. |
| `c50a85b9` | rdx | shader: lower packed 16-bit add and subtract in direct compute (#3088) | VOP3P `v_pk_add_i16` (`0x02`), `v_pk_sub_i16` (`0x03`), `v_pk_add_u16` (`0x0a`), `v_pk_sub_u16` (`0x0b`): sem `clamp` a lane dá a volta mod 2^16; com `clamp` satura no intervalo da signedness. Fixes #3085. |
| `354c6870` | rdx | shader: lower scalar shift-add and carry in direct compute (#3104) | SOP2 `s_lshl1..4_add_u32` (`0x2e`–`0x31`): `temp = ((u64)S0 << k) + S1`, `D = temp[31:0]`, `SCC = temp >= 2^32` — verdadeiro quando `S0 >> (32-k) != 0` ou quando a soma de 32 bits dá carry. Fixes #3103. |
| `d71f9ed2` | rdx | shader: lower v_xnor_b32 in direct compute (#3107) | `v_xnor_b32` em VOP2 `0x1e`, VOP3A `0x11e` e VOP2 SDWA: `~(S0 ^ S1)`; o `.cpp` acompanha (+1/−1) na tabela de opcodes. Fixes #3106. |
| `f6691b67` | legacy-code-archaeologist | shader: lower 64-bit scalar bit counts and scans in direct compute (#3073) | Seis SOP1 de fonte 64 bits (`s_bcnt0_i32_b64`, `s_bcnt1_i32_b64`, `s_ff0_i32_b64`, `s_ff1_i32_b64`, `s_flbit_i32_b64`, `s_flbit_i32_i64`) via `unaryBits.i32` 31/32/33; destino é SGPR de 32 bits (`s[dst+1]` intocado), fonte é par SGPR alinhado ou literal inline `-16..64`. Fixes #3072. |
| `811c4ceb` | nanyoudegege | docs: record the RTX 3090 Ti hardware report (#3113) | Uma linha no `README.md` (+1/−0). Nada executável. |
| `6c2cddec` | rdx | shader: lower scalar sign-extension in direct compute (#3110) | `s_sext_i32_i8` (SOP1 `0x19`) e `s_sext_i32_i16` (`0x1a`) como `ashr(shl(a,24),24)` e `ashr(shl(a,16),16)`, preservando `SCC`; o `.cpp` (+2) acompanha. Fixes #3108. |
| `55669240` | legacy-code-archaeologist | shader: lower unfused f32 multiply-adds in direct compute (#3007) | `v_mad_f32`, `v_mad_legacy_f32`, `v_mac_f32` e `v_mac_legacy_f32` deixam de ser recusados: produto e soma viram `fmul` e `fadd` separados, sem fast-math, para o DXIL não poder contrair em `FMad` (opcode terciário 46); denormal é zerado por máscara inteira que preserva o bit de sinal; as formas legacy fazem `0 * x = +0`; as `v_mac_*` somam o destino; `abs`/`neg` no acumulador é recusado. Fixes #3005. |
| `ca47eb1d` | legacy-code-archaeologist | shader: lower signed and unsigned 64-bit vector MAD in direct compute (#3004) | `v_mad_u64_u32` e `v_mad_i64_i32` (recusados até aqui): `D = S0*S1 + S2 mod 2^64`, fatores de 32 bits zero/sign-estendidos e `S2` de 64 bits; o carry-out do add vai para o SDST VOP3B via `write_ballot` (caminho de máscara de lanes já usado pelas outras instruções de carry). Fixes #3000. |
| `300b93f0` | legacy-code-archaeologist | shader: lower signed and saturated 32-bit add/subtract in direct compute (#3011) | `v_add_nc_i32`/`v_sub_nc_i32` (VOP3-only, antes recusados) dão a volta mod 2^32; as formas com `clamp` de `v_add_nc_u32`, `v_sub_nc_u32` e `v_subrev_nc_u32` saturam (i32 satura para o sinal de S0, u32 para 0 ou `0xffffffff`), com detecção de overflow por testes de sinal em 32 bits — sem inteiros de 64 bits no módulo; `abs`/`neg`/`omod`/`op_sel`/SDWA/DPP recusados. Fixes #3009. |

## Pull requests abertos (10)

Lidos por título, corpo e contagem de arquivos; **nenhum patch de PR aberto foi lido linha a linha**.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #3100 | shader: lower pixel structured-to-raw buffer loads | Desde #2910 um load MUBUF não indexado em estágio pixel era recusado como `buffer load through a modified descriptor`; consulta `raw_descriptor_alu()` também no pixel e abaixa o load como leitura de byte-address limitada a `NUM_RECORDS * stride`, com tag própria quando o stride foi clampado (+467/−6, 8 arquivos). | Não é defeito do catálogo; é a mesma classe de lacuna do lowering que os commits desta hora fecham. |
| #3105 | startup: lower triangle nodes in image_bvh_intersect_ray | Nó de triângulo de 64 bytes (kinds 0..3), mapa de IDs de 8 bits, teste raio/triângulo watertight em binary32, modo de retorno 1 `{t,det,v,w}`, miss `{+inf,1,0,0}`; empilhado em #3032 (box node binary16) e com `main` mesclada (+972/−94). | Área de ray tracing; nenhum D-xx de Xbox2Windows descreve BVH. |
| #3109 | shader: keep an accumulated lane mask live at a compute execz exit | Aceita a saída por esgotamento de lanes que mantém uma máscara acumulada viva (`s_and_b32 sA,sA,exec_lo` + `s_or_b32 sB,sB,sA`), com `sA` morto na saída e todo uso posterior de `sB` por lane; o loop contado com latch para trás continua recusado. `mergeable=blocked`. | Não. Refs #2645 (T4). |
| #3111 | tooling: harden atomic writes, path guards, and CLI entry in cpu_leaf_aot | `main(argv)` explícito, `_validate_paths` (sufixo `.cpp`, symlink, alias por `samefile`), escrita atômica por temporário irmão, guarda de `start` booleano/negativo, suíte sintética PE32+ (+763/−4). | Não. |
| #3114 | shader: lower s_quadmask_b32 and s_quadmask_b64 | `s_quadmask_b32` (SOP1 `0x2c`) e `_b64` (`0x2d`) por colapso de nibbles e empacotamento, `SCC = (D != 0)`, par SGPR par validado. Fixes #3112. **Já mesclado como `2641cc72` depois do snapshot** — fica para o próximo ciclo. | Não. |
| #3115 | shader: harden import_gfx10_tables CLI, atomic writes, and parser guards | `expected_sha256` parâmetro, validação de fonte/saída, escrita atômica, suíte offline (+733/−15). | Não. |
| #3117 | shader: lower v_frexp_exp_i32_f32 and v_frexp_mant_f32 | `v_frexp_exp_i32_f32` (VOP1 `0x3f`/VOP3A `0x1bf`) e `v_frexp_mant_f32` (`0x40`/`0x1c0`) por manipulação inteira de IEEE-754 binary32; denormal normalizado sem flush; `±0`, `±Inf`, `NaN` preservados. Fixes #3116. | Não. |
| #3119 | tooling: extend the patch manifest loader (nesting refusal, shader kind, load_data) | Recusa aninhamento profundo (`RecursionError` virava traceback), adiciona `kind = "shader"` com checagem de forma de `key`/`pipeline`/`revision`/`files`. Refs #3037; ainda sem aplicador. | Não. |
| #3120 | shader: lower v_frexp_mant_f16 and v_frexp_exp_i16_f16 | As versões f16 (`0x59/0x1d9`, `0x5a/0x1da`) por manipulação inteira de binary16 preservando `D[31:16]`. Fixes #3118. | Não. |
| #3121 | cpu: pass bounded frame-slot pointers to internal callees in cpu_switch_aot | Resumo de parâmetro-ponteiro e prova de frame para passar `LEA reg,[rsp+disp]` a callees internos que leem/escrevem escalar de largura fixa sem escape; 313 pares de entrada em `/Od` e `/O2`, 26 controles negativos (+1390/−4). | Não. |

## Pull requests mesclados (16)

Todos em squash: o SHA do merge é o commit único na `main`.

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #3068 | shader: lower explicit-LOD 2D gather in direct DXIL | `a9fd1488` | Gather 4 com LOD explícito e sampler constante, quatro loads ordenados; teste compara com referência independente de mip/tap. |
| #2919 | startup: persist and reuse translated PSO recipes | `5df098da` | Receitas de estado de PSO em texto, 64/64 KiB, handle exclusivo, replay antes da criação; teste de dois processos com 32 bytes RGBA idênticos. |
| #3089 | shader: lower s_bfe_u64, s_bfe_i64 and s_brev_b64 | `0df67295` | Extração de bitfield e reversão de 64 bits só com opcodes de 32 bits; regra de par em `transfer()` estendida. |
| #3099 | shader: lower v_cvt_off_f32_i4 | `8929423c` | Nibble com sinal vezes 1/16, `omod`/`clamp` aplicados ao float. |
| #3102 | shader: lower v_mul_i32_i24 and the two hi forms | `7a12818c` | Multiplicações de 24 bits, incluindo o produto alto por split 12+12. |
| #3092 | shader: lower v_pk_mul_lo_u16, v_pk_mad_u16, v_pk_mad_i16 | `428c08c1` | Multiplicação e MAD empacotados de 16 bits com seleção por half. |
| #3082 | shader: lower v_pk_lshl/lshr/ashrrev_b16 | `4a6302eb` | Deslocamentos empacotados de 16 bits por half. |
| #3088 | shader: lower v_pk_add/sub_i16 and v_pk_add/sub_u16 | `c50a85b9` | Soma e subtração empacotadas com saturação opcional. |
| #3104 | shader: lower s_lshl1..4_add_u32 | `354c6870` | Shift-add escalar com SCC de carry calculado sem 64 bits. |
| #3107 | shader: lower v_xnor_b32 | `d71f9ed2` | `~(S0^S1)` em VOP2, VOP3A e SDWA. |
| #3073 | shader: lower 64-bit scalar bit counts and bit scans | `f6691b67` | Seis instruções de contagem/varredura de bits via `unaryBits`. |
| #3113 | docs: record the RTX 3090 Ti hardware report | `811c4ceb` | Uma linha de README; nada de código. |
| #3110 | shader: lower s_sext_i32_i8 and s_sext_i32_i16 | `6c2cddec` | Extensão de sinal por dois shifts aritméticos. |
| #3007 | shader: lower unfused f32 multiply-adds | `55669240` | Quatro formas de MAD não fusionado como `fmul`+`fadd` sem fast-math, com flush de denormal e regra legacy. |
| #3004 | shader: lower v_mad_u64_u32 and v_mad_i64_i32 | `ca47eb1d` | MAD de 64 bits com carry para SDST pelo caminho de máscara. |
| #3011 | shader: lower signed and saturated 32-bit add/subtract | `300b93f0` | Formas `_nc_i32` e as `clamp` das `_nc_u32`, saturação detectada com testes de sinal de 32 bits. |

## Cruzamento com o catálogo

- **Não corrige nenhum D-xx de Xbox2Windows.** Os 30 defeitos catalogados são de superfície
  apresentada, save, debugger, heap, CLI e validação; nenhum descreve recusa de lowering no
  tradutor direto de compute. A série desta hora fecha lacunas de cobertura (#2935), não defeitos
  registrados.
- **D-27** (`image_sample`/`image_sample_lz` 2D `a16` lidas como dois registradores binary32) é
  vizinho de `a9fd1488`, que mexe em MIMG: o commit novo trata `image_gather4_l`, lê U/V/LOD de três
  VGPRs contíguos e exige sampler constante — **não toca o caminho `a16`**. Se a mudança altera o
  sintoma de D-27, **NÃO VERIFICADO**.
- **D-06** (comparações f32 erradas sob FXC/DXC sem `-Gis`): `55669240` emite `fmul`/`fadd` sem
  fast-math, o que impede contração em `FMad`, mas no caminho direto de compute, não no FXC/DXC. Não
  é correção de D-06.
- **Abre defeito novo?** Não medido. O próprio `#3007` registra que a leitura de volta foi pulada
  (skip 77) na única GPU em que foi medida, por exigir shader model 6.6 com wave32.

## Confiança

- Os 16 diffs foram lidos: cabeçalho de arquivos alterados, `docs/experiments/*` acrescentados no
  mesmo commit e os trechos de `src/gfx1013_direct_compute.inl` / `pso_recipe.h` /
  `d3d12_draw.inl`. As descrições acima vêm do código, não do assunto do commit.
- **Contagem de merges x commits.** O primeiro `compare` (`de1595b9...head`) devolveu 15 commits com
  head `ca47eb1d` (16:05:36 no relógio do GitHub), mas a lista de mesclados trazia 16 PRs. A
  diferença é o `#3011` (`300b93f0`, `merged_at` 16:08:23), que entrou cerca de um segundo depois do
  snapshot; conferi que é descendente do cursor (`compare` = `ahead 16`) e o incluí. Janela
  documentada: `de1595b9` → `300b93f0`.
- Durante a coleta o head foi para **`2641cc72`** (`#3114`, quadmask, 16:11:30). Esse commit está
  fora deste documento de propósito: o cursor fica em `300b93f0` e o próximo ciclo começa nele.
- PRs abertos: corpo e contagem de arquivos lidos; **nenhum patch lido**. `#3109` está
  `mergeable=blocked`; os outros, `clean` ou `unknown`.
- Testes alegados pelos commits (controles de compilação e leitura de volta) **não foram
  executados** aqui. Onde o próprio commit diz que a leitura de hardware foi pulada (`#3007`,
  `#3011` na Quadro M1000M), isso está registrado como não medido, não como aprovado.
- `last_pr_number` sobe para 3121. `#3114` consta como aberto nesta coleta e foi mesclado em
  `2641cc72` às 16:11:30 — depois do cursor de merges (16:08:23) —, então o próximo ciclo o pega
  pelo relógio de `merged_at`, não pelo número.
