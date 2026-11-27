# R0Y00/Xbox2Windows — atualização de 2026-11-27 05:00

## Resumo

Doze commits novos na `main` acima do cursor `8895045b`, e os doze são merges de PR: a maior parte é
cobertura de `gfx2hlsl` (selects simbólicos em dados de descritor, `v_pk_min/max_f16`, `v_ldexp_f32`,
`v_alignbit/alignbyte/bfm`, `v_cvt_pk_*`, `s_cmp_eq/lg_u64`, `s_bitcmp*_b64`, sparse `image_store` como
escritas de componente) mais três de runtime/teste (retain de readback apresentado até a fence, fixture de
pixel-history, canonicalização de caminhos no `run_checks_parallel`). Na janela, **16** PRs mesclados: 12
caíram na `main` e 4 na **`cpp-migration`**, que não é rastreada pelo cursor de commits. Abertos: 5 acima
do cursor 3002 (#3004, #3006, #3007, #3010, #3011), todos de lowering de ALU no caminho direto de compute.

## Commits novos (12)

`commits?sha=main&per_page=100`: `8895045b` está na posição 12; o topo é `e62e3a5e`. Todos com um pai
(squash de PR).

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `e62e3a5e` | saii | startup: keep descriptor-data scalar selects symbolic in compute (#2903) | +535/-0 em 6 arquivos: `tools/startup/gfx2hlsl.py` +64 (mantém `s_cselect_b32` sobre dados de descritor simbólico no compute, permite select opaco morto e recusa uso numérico, de descritor emendado e de par alto), `docs/experiments/startup-gfx2hlsl-cselect-descnum.md` novo (+87), `docs/startup-runtime.md` +7, teste `tests/startup_gfx2hlsl_cselect_descnum_check.py` novo (+363) e seu `.cmake` (+13). |
| `1bedde06` | Leyukaka | startup: lower packed binary16 minimum and maximum (#2927) | +254/-8 em 7 arquivos: `gfx2hlsl.py` +40/-8 (escolhe a codificação half original de cada resultado empacotado, preservando zero com sinal e a regra de um único NaN), teste novo `startup_gfx2hlsl_pk_minmax_check.py` +168, `.cmake` +13, `docs/experiments/startup-gfx2hlsl-pk-minmax.md` +27, `AGENTS.md` +1. |
| `18d2adfe` | SimSalabimse | startup: lower sparse pixel image stores as component writes (#2661) | +364/-64 em 12 arquivos: `gfx2hlsl.py` +25/-13 (store de pixel esparso vira escrita de componente para canais não tocados sobreviverem quando stores disjuntos miram o mesmo texel; máscara cheia de array e o comportamento de compute ficam), `src/startup_stubs/d3d12_draw.inl` +21/-3, `typed_uav_load.h` novo (+14), `tests/startup_pixel_image_store_check.py` +185/-22, `offscreen_check.cpp` +17/-5, `startup_image_store.cpp` +43/-8, `docs/startup-runtime.md` +22/-12. |
| `923c0ac9` | Chad Hyatt | shader: track both VCC halves for unsigned MAD64 carry (#2979) | +44/-19 em 12 arquivos: `gfx2hlsl.py` +5/-4 invalida a prova de VCC alta quando há carry VOP3B unsigned vivo e **rotaciona as duas chaves de requisição de shader do runtime** (`d3d12_draw.inl` +2/-2 e os testes que embutem a chave); `tests/startup_gfx2hlsl_mad64_check.py` +19. |
| `57f9e07f` | Chad Hyatt | runtime: retain presented readbacks until fence completion (#2968) | +344/-16 em 8 arquivos: `src/startup_stubs/d3d12_xs_device.cpp` +176/-10 mantém a posse do readback diagnóstico até a fence correspondente, aposenta trabalho estourado de tempo de forma assíncrona e desliga o reuso na remoção de device ou em signal falho; `fence_completion.h` (+16) e `present_readback_state.h` (+41) novos; teste `startup_present_readback_check.cpp` (+89); `CMakePresets.json` +2/-2. |
| `228f6044` | rdx | startup: lower 64-bit scalar equality comparisons in gfx2hlsl (#2891) | +403/-0 em 4 arquivos: `gfx2hlsl.py` +23 (compara as duas palavras escalares de `s_cmp_eq_u64`/`s_cmp_lg_u64`, mantém a recusa de máscara de lane), teste novo `startup_gfx2hlsl_s_cmp64_check.py` (+368) cobrindo igualdade, desigualdade e cada metade diferindo isolada. |
| `d4fd6cc1` | rdx | startup: lower 64-bit scalar bit compares in gfx2hlsl (#2880) | +441/-0 em 4 arquivos: `gfx2hlsl.py` +26 (seleciona palavra alta ou baixa pelo bit cinco para `s_bitcmp0_b64`/`s_bitcmp1_b64`, preserva recusa de máscara de lane), teste novo `startup_gfx2hlsl_s_bitcmp64_check.py` (+403). |
| `7add7a93` | Chad Hyatt | gpu: isolate the pixel-history fixture from inherited diagnostics (#2869) | +2/-0 em `tests/startup_pixel_history_draw_check.py`: limpa as três chaves de dump herdadas antes de aplicar cada caso explícito da fixture. Só teste. |
| `50488e1c` | lukaslw-lab | startup: opt in to placed buffer aliasing for table SRVs (#2977) | +376/-1 em 8 arquivos: `src/startup_stubs/d3d12_draw.inl` +31/-1 liga SRVs de tabela crus elegíveis direto a um alias de placed buffer quando habilitado, preservando índice combinado e estados de leitura do shader e validando a faixa arredondada inteira; `CMakeLists.txt` +4; `docs/experiments/startup-table-srv-alias.md` novo (+104); `tests/startup_address_srv.cpp` +100; `startup_table_alias_check.py` novo (+109). |
| `9436a181` | Chad Hyatt | shader: admit proven calculated far early-outs (#2956) | +699/-13 em 5 arquivos: `tools/startup/gfx2hlsl.py` +81 e `tools/pixel_aot.py` +41/-13 aceitam um branch calculado limitado cuja continuação é pura e terminante ("far early-out"), validando extração byte-exata com LLVM 15; teste `startup_gfx2hlsl_far_early_out_check.py` novo (+555); `.github/workflows/windows.yml` +2. |
| `34b2f58d` | Rachel Amber | startup: lower v_ldexp_f32 in gfx2hlsl (#2913) | +397/-6 em 4 arquivos: `gfx2hlsl.py` +68/-6 baixa `v_ldexp_f32` pelo contrato de escala binário bit a bit já existente; teste novo `startup_gfx2hlsl_ldexp_check.py` (+322) com controles de decodificação e de recusa. |
| `be58ec5a` | ashishdhillon | tooling: canonicalize script paths when checking duplicates (#3002) | +29/-6 em 3 arquivos: `tools/run_checks_parallel.py` +13/-5 compara caminhos canônicos de script ao checar duplicatas, preservando os argumentos de comando e os caminhos de execução originais; testes `run_checks_parallel_check.py` +6 e `run_checks_fragments_check.py` +10/-1. |

## Pull requests abertos (5)

`gh pr list --state open --limit 200` devolveu **114** abertos; os 5 abaixo têm número acima do cursor 3002.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 3004 | shader: lower v_mad_u64_u32 and v_mad_i64_i32 in the direct C++ compute path | Baixa `v_mad_u64_u32` e `v_mad_i64_i32` no frontend direto de compute (antes recusados com `direct DXIL lowering for <name>`), seguindo o contrato `mad64` do `gfx2hlsl.py`: D = S0*S1 + S2 módulo 2^64, fatores de 32 bits zero/sign-estendidos e S2 de 64 bits; o carry da soma vai ao SDST VOP3B pelo caminho de máscara de lane (`carry_destination` em `src/gfx1013_compute_native.cpp` passa a nomear esse SDST). Refs #3000, parte de #2935. +223/-0, 5 arquivos. | Não. |
| 3006 | shader: read the wave size of slot 67 compute requests from .sa | `read_compute_request` tira `ComputeRequest::wave32` do bit 20 do dword 4 do `.sa` (set = wave32, clear = wave64), `.sa` que não tenha 28 bytes segue recusado; a rota DXIL direta de `translate_compute` compila pedido wave64 pelo caminho que `translate_compute_container` já usa. **Base `cpp-migration`**; o corpo diz que corrige #2999 e que o issue precisa ser fechado à mão por isso. +180/-23, 22 arquivos. | Não. |
| 3007 | shader: lower unfused f32 multiply-adds in the direct compute path | Baixa `v_mad_f32`, `v_mad_legacy_f32`, `v_mac_f32` e `v_mac_legacy_f32` em `float_alu` (antes recusados), com a semântica `mad32` do `gfx2hlsl.py`: entradas denormais levadas a zero com sinal, produto arredondado sozinho, formas legacy fazendo `0 * x + 0`, adendo já flushado somado e soma flushada. +244/-1, 4 arquivos. | Não. |
| 3010 | shader: lower DPP16 row_shl, row_shr and row_ror in direct compute | Estende o DPP16 do compute direto (que só baixava `quad_perm` e os dois espelhos) a `row_shl:N`, `row_shr:N` e `row_ror:N` para N = 1..15, com máscaras de linha e de banco: `row_shl` lê `r + N`, `row_shr` lê `r - N`, `row_ror` lê `(r - N) mod 16` dentro da linha de 16 lanes. **Base `cpp-migration`**; corpo diz que corrige #3008, a fechar à mão. +254/-18, 8 arquivos. | Não. |
| 3011 | shader: lower signed and clamped 32-bit add/sub in the direct compute path | Baixa `v_add_nc_i32`/`v_sub_nc_i32` (só VOP3) e as formas com `clamp` de `v_add_nc_u32`, `v_sub_nc_u32` e `v_subrev_nc_u32` em `arithmetic()`; sem clamp o resultado inteiro dá a volta módulo 2^32, com clamp `_i32` satura para a faixa i32 e `_u32` satura em 0 / 0xffffffff; overflow por testes de sinal de 32 bits, sem `Int64Ops`. Recusa neg/abs, omod, op_sel, SDWA e DPP. +215/-0, 4 arquivos. | Não. |

## Pull requests mesclados (16)

`search/issues?q=repo:R0Y00/Xbox2Windows type:pr is:merged merged:>=2026-11-27T03:57:04Z` devolveu **17**;
#2947 cai exatamente no cursor (merge `8895045b`, já documentado na hora 03:00) e os 16 restantes estão
abaixo. Doze entraram na `main` e são os doze commits acima; quatro entraram na **`cpp-migration`**.

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| 2661 | startup: pixel image_store sparse DMASK as component stores | `18d2adfe` (1 pai, +364/-64, 12 arquivos, `main`) | O store de pixel esparso vira escritas de componente: canais fora da máscara ficam como estavam quando dois stores disjuntos miram o mesmo texel. Fecha a lacuna que o próprio diff testa em `startup_image_store.cpp` (+43/-8) e no `startup_pixel_image_store_check.py` (+185/-22). |
| 2869 | gpu: isolate pixel-history fixture from inherited dump diagnostics | `7add7a93` (1 pai, +2/-0, 1 arquivo, `main`) | Três `clear` de chaves de dump antes de cada caso da fixture; nenhum código de produto. |
| 2880 | startup: lower s_bitcmp0_b64 and s_bitcmp1_b64 in gfx2hlsl | `d4fd6cc1` (1 pai, +441/-0, 4 arquivos, `main`) | O lowering novo em `gfx2hlsl.py` (+26) e o controle Python de 403 linhas; recusa de máscara de lane preservada. |
| 2891 | startup: lower s_cmp_eq_u64 and s_cmp_lg_u64 in gfx2hlsl | `228f6044` (1 pai, +403/-0, 4 arquivos, `main`) | O lowering (+23) compara as duas palavras; o controle cobre igualdade, desigualdade e cada metade diferindo sozinha. |
| 2903 | startup: keep s_cselect_b32 on descriptor data symbolic in compute | `e62e3a5e` (1 pai, +535/-0, 6 arquivos, `main`) | Só a doc de experimento e o teste entram junto com `gfx2hlsl.py` +64; não toca runtime nem stubs. |
| 2913 | startup: lower v_ldexp_f32 to ldexp32 in gfx2hlsl | `34b2f58d` (1 pai, +397/-6, 4 arquivos, `main`) | Reusa o contrato de escala binário existente; o teste (+322) traz controles de decodificação e de recusa. |
| 2927 | startup: lower v_pk_max_f16 and v_pk_min_f16 in gfx2hlsl | `1bedde06` (1 pai, +254/-8, 7 arquivos, `main`) | Regra de um-NaN e zero com sinal aplicadas à escolha da codificação half de cada resultado empacotado; `AGENTS.md` ganha uma linha. |
| 2956 | shader: admit proven calculated far early-outs | `9436a181` (1 pai, +699/-13, 5 arquivos, `main`) | Dois scripts mudam (`gfx2hlsl.py` +81, `pixel_aot.py` +41/-13) e o fluxo do Windows ganha duas linhas; a prova é a extração byte-exata conferida com LLVM 15 no teste de 555 linhas. |
| 2968 | runtime: retain presented readback ownership until fence completion | `57f9e07f` (1 pai, +344/-16, 8 arquivos, `main`) | `d3d12_xs_device.cpp` +176/-10 é o grosso; dois headers novos (`fence_completion.h`, `present_readback_state.h`), um teste C++ novo e `CMakePresets.json` +2/-2 (preset de build). |
| 2977 | startup: opt-in alias binding for table buffer views over aliased placed buffers | `50488e1c` (1 pai, +376/-1, 8 arquivos, `main`) | Ligado por opção (`CMakeLists.txt` +4 provavelmente o define): sem a opção o `d3d12_draw.inl` +31/-1 mantém o caminho anterior; com ela, valida a faixa arredondada inteira antes de apontar o SRV ao alias. |
| 2979 | shader: track both VCC halves for unsigned MAD64 carry | `923c0ac9` (1 pai, +44/-19, 12 arquivos, `main`) | 9 dos 12 arquivos são testes que embutem a chave de requisição de shader: a mudança **invalida caches de shader do runtime** (duas chaves rotacionadas), não só o lowering. |
| 3002 | tooling: canonicalize script paths when checking duplicates | `be58ec5a` (1 pai, +29/-6, 3 arquivos, `main`) | Correção de ferramenta de CI (`run_checks_parallel.py`) com dois testes ajustados. |
| 2986 | shader: lower v_alignbit_b32, v_alignbyte_b32 and v_bfm_b32 in the direct C++ compute path | `24d92609` (base **`cpp-migration`**, +166/-1, 4 arquivos) | `src/gfx1013_direct_compute.inl` mais doc de experimento e teste. **Não está na `main`**: `24d92609` não aparece no histórico de `commits?sha=main`. |
| 2996 | shader: lower v_cvt_pk_u16_u32 and v_cvt_pk_i16_i32 in the direct C++ compute path | `4701650d` (base **`cpp-migration`**, +175/-0, 4 arquivos) | Satura S0/S1 a 16 bits e empacota S0 na metade baixa, conforme a tabela VOP3 do `gfx2hlsl.py`. Fora da `main`. |
| 2998 | shader: lower v_cvt_pk_u8_f32 in the direct C++ compute path | `af6d255f` (base **`cpp-migration`**, +197/-0, 4 arquivos) | `D = (S2 & ~(0xff << 8*S1[1:0])) \| (flt32_to_uint8(S0) << 8*S1[1:0])`, com a conversão medida em RDNA 2. Fora da `main`. |
| 3003 | tooling: support .rtv variant requests in compile_dxil --watch and --refusals | `999e17cd` (base **`cpp-migration`**, +156/-24, 5 arquivos) | `tools/startup/compile_dxil.cpp` com `dedupe_graphics_markers` compartilhado e precedência de `.rtv`; dois testes Python e duas docs. Corrige #3001. Fora da `main`. |

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum. Os 12 merges da `main` não usam nenhum hash `corrigido` do CSV
  (`2a898640`, `21e8f5c0`, `2260b7a4`, `05ef4d92`, `583178d4`, `83bb752d`, `20fa94a9`, `2c1dcdd6`,
  `f360d30a`, `0b649c69`, `b3a90888`, `313803bc`, `ac539650`, `e50a72bc`, `14d6023e`, `6ef541f3`,
  `b87a99a3`, `5b29fe0f`, `13745b1b`, `2e5fdca4`, `9a2b9cd8`, `d28732b6`, `5d6d5f52`, `6855f425`,
  `07f2464f`, `c3c3ab3c`, `f37fe7b8`, `22989d61`). Candidato próximo que **não** casa: **D-06** (f32
  `nlt/nle/ngt/nge/o/u` errados sem `-Gis`, corrigido em `583178d4`) é de comparação float; os #2880/#2891
  baixam comparação **escalar inteira de 64 bits** (`s_bitcmp*_b64`, `s_cmp_eq/lg_u64`), outra família de
  opcodes e outro tipo. **D-28** (UNORM/SNORM do store formatado usando `round()`) não aparece no
  `v_cvt_pk_u8_f32` de #2998, que está na `cpp-migration` e empacota byte, não formato de store.
- **Abre defeito novo?** Nenhum demonstrado por diff. Dois pontos ficam registrados:
  - `923c0ac9` (#2979) **rotaciona as duas chaves de requisição de shader do runtime**. Qualquer cache
    local de shader feito com a chave anterior deixa de ser encontrado — é o comportamento pretendido
    (a prova de VCC alta era inválida), mas explica recompilação em massa na primeira execução depois do
    merge, e não verifiquei se algum consumidor externo fixa a chave.
  - `50488e1c` (#2977) aponta um SRV de tabela cru direto a um alias de placed buffer, opcional e
    validando a faixa arredondada inteira. Se o título escrever no placed buffer entre a ligação e o draw,
    o SRV vê o alias — o desenho é esse — e não há teste no diff para escrita concorrente durante o draw.

## Confiança

Verificado: `commits?sha=main&per_page=100` com `8895045b` na posição 12 e `e62e3a5e` no topo; cada um dos
12 lido em `commits/<sha>` (um pai, arquivos e `+/-` como na tabela); `search/issues` com 17 PRs mesclados
desde 03:57:04Z, dos quais #2947 é o cursor; `pulls/<n>` para 2986/2996/2998/3003 (base `cpp-migration`,
`+/-` e lista de arquivos) e confirmação de que `18d2adfe` é `#2661` com `merged_at` 05:02:20Z e um pai.

Desvio registrado: `gh pr list --state merged --limit 50` devolveria 15 PRs e **perderia o #2661**, cujo
merge é de 05:02:20Z mas cuja criação é antiga (a listagem corta por criação, não por merge). A coleta de
mesclados desta hora usou a API de busca por `merged:>=<last_run>`, como o procedimento manda.

Não verificado: se `24d92609`, `4701650d`, `af6d255f` e `999e17cd` chegam à `main` por alguma promoção da
`cpp-migration` (até esta hora não chegaram, e o cursor de commits segue a `main`); a execução dos testes
citados nos corpos (LLVM 15.0.7, RDNA 2, LLVM 15 byte-exato); o estado da CI do repositório; e o efeito
prático da rotação das chaves de shader em caches já gravados. Nenhum teste foi executado aqui; o
repositório monitorado é somente leitura.
