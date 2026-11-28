# R0Y00/Xbox2Windows — atualização de 2026-11-28 15:00

## Resumo

Treze commits e um merge (`#3365`) desde o cursor `cdce92da`: a leva é quase toda **sincronização**
das pilhas nativas de shader e de mesh contra a `main` (três merges internos, três commits "sync"), o
que explica a maior parte do volume de linhas sem trabalho novo. O trabalho novo com consequência é
`7a163fc5`: a aposentadoria dos *sampler heaps* traduzidos passa a ser feita pelo allocator da lista
(`clear_draw_sampler_snapshot`) em vez de um mapa global por `ID3D12CommandList*`; e `090de8f2`
(`#3365`) normaliza a realocação do descritor fixo de Mesh57 de `0x300` bytes na chave do perfil — sem
isso, dois estados idênticos de processos diferentes não casavam.
Quatro PRs abertos, todos novos, três deles lowerings de `v_cvt_f64` no caminho direto de compute.

## Commits novos (13)

Janela: todos os 13 commits sem documento anterior em `cdce92da...f5a6f6ba` (72 à frente, 59 já
documentados em ciclos anteriores).

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `6b9db222` | Ammar Abdelfattah | shader: address review feedback on `compile_lds_request` guards and error paths | `tools/startup/compile_lds_request.py` +51/−26 e o check `tests/startup_compile_lds_request_cli_check.py` +166/−50: guardas e caminhos de erro do pedido de compilação LDS, cobertos pelo teste de CLI. |
| `4133425c` | Roland Edi | shader: merge `codex/native-shader-modules` into the refresh branch | `src/shader_compute_request.cpp` +23/−10 e `.h` +2: o portão do watch request mantém a invalidação `sidecar_changed` de `#3138` **e** as checagens `dependency_changed`/`lost`; `tools/startup/compile_dxil.cpp` +45/−20. Testes novos em `native_compile_dxil_watch_check.py` (+103). |
| `86cf5aa8` | Roland Edi | shader: merge `main 3926a289` into the native shader stack | `CMakeLists.txt` +31, `CMakePresets.json` +3/−2, `src/shader_compute_request.*` e cinco documentos novos em `docs/experiments/` (`gdk-knowledge-report.md` 97, `gdk-layouts.md`, `gdk-platform-profiles.md`, `gdk-runtime-oracle.md`, `gdk-shader-corpus.md`). É integração: mantém os helpers de chave gráfica/compute e o leitor de pixel input `.ps57in`. |
| `48b2027c` | Roland Edi | Merge remote-tracking branch `origin/main` into `merge/native-shader-main-current` | `src/gfx1013_compute_native.cpp` +8, `tools/startup/gfx2hlsl.py` +56 e o lowering `v_dot2` com teste novo (`startup_gfx2hlsl_v_dot2_check.py` 308 linhas, `native_exec_symbol_b64_check.py` 47). Integração de `main`. |
| `b30d8f1f` | R0Y00 | Sync the native shader compiler with current main | `src/gfx1013_compute_native.cpp` +23/−7 e `src/gfx1013_direct_compute.inl` +20/−5; novos `tests/cpu/adx_aot.asm` (336), `tests/cpu/adx_aot_check.py` (742), `tests/cmake/native_carry_vcc_hi.cmake`, `native_scalar_mask64.cmake`, `tests/gdk_exports_check.py` (251) e `docs/experiments/gdk-exports.md`. Sincronização com resolução de conflito declarada no corpo. |
| `c8e37cc6` | R0Y00 | Keep mesh transfer calls on the full-width graphics path | Uma linha em `src/gfx1013_compute_native.cpp`: `State transfer(..., bool narrow_wave64 = false)` — o parâmetro ganha valor padrão, então as chamadas de transferência de mesh que não o passam caem no caminho gráfico de largura cheia. |
| `7a163fc5` | R0Y00 | runtime: retire translated sampler heaps with their allocator | `src/startup_stubs/d3d12_draw.inl` −48/+29: sai o mapa global `recorded_draw_sampler_heaps` por `ID3D12CommandList*` e a lista `submitted_draw_sampler_heaps` com fence; `d3d12_xs_device.cpp` +41/−9 troca `release_upload_recording` por `release_list_recording` + `clear_draw_sampler_snapshot`. `tests/startup_list_resubmit.cpp` +45: snapshot de samplers gravado, submetido a **duas** filas, a segunda presa num fence liberado pela CPU; soltar a lista e resetar o allocator mantém o snapshot vivo enquanto a segunda submissão está bloqueada. |
| `303ecdd7` | R0Y00 | Sync fixed mesh runtime with the current compiler base | `CMakeLists.txt` +49, `CMakePresets.json`, `docs/experiments.md` +94/−12 e os mesmos documentos novos de `86cf5aa8` mais `direct-compute-exact-width-image-store.md`. Conflito declarado em seis arquivos. |
| `0cdcfd00` | R0Y00 | Sync the fixed mesh runtime with main after sampler retirement | Repete `7a163fc5` sobre a base nova (`d3d12_draw.inl` +29/−48, `d3d12_xs_device.cpp` +41/−9, `startup_list_resubmit.cpp` +45): é a resolução do sync depois da aposentadoria de samplers, não trabalho novo. |
| `cb81bd81` | R0Y00 | tests: check the exact shader pair in the missing DXC fallback | `tests/startup_native_graphics_draw_check.py` +3/−2: o teste do fallback sem DXC passa a conferir o **par** exato de shaders, não só a ausência do compilador. |
| `c23cbc36` | R0Y00 | tooling: accept optional fixed rectangle geometry in shader manifests | `tools/patch_manifest.py` +3/−4 e `tests/patch_manifest_check.py` +13/−8: geometria de retângulo fixa passa a ser opcional no manifesto; o doc `patch-manifests.md` acompanha. |
| `090de8f2` | R0Y00 | gpu: normalize relocations in extended Mesh57 descriptors | `src/startup_stubs/mesh57_profile.h` +47/−1: para descritor de `0x300` bytes, além de zerar os índices 0/1/3/5/13, o código lê a seção em `+0x38` e trata como ponteiro todo valor em `[0x10000, 0x800000000000)`, preservando as distâncias entre os ponteiros internos dos dois blocos de shader record e removendo a realocação do processo (o endereço de alocação repetido em `+0x160`/`+0x2c0` varia de forma independente). `tests/startup_indirect_draw_indexed.cpp` +48 cobre. |
| `f5a6f6ba` | Royoo | Merge pull request #3365 from `R0Y00/codex/normalize-mesh57-color-descriptors` | Merge de dois pais do `090de8f2`; 3 arquivos, +103/−3, exatamente o commit já descrito (doc, `mesh57_profile.h`, teste). Nada além. |

## Pull requests abertos (4)

Todos abertos entre 14:22Z e 14:53Z, nenhum em rascunho.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #3366 | Runtime: add present-path diagnostic logging | Com `X2W_D3D12_RENDER=1` e tela preta, loga nas saídas silenciosas do caminho de present (`queue_present` com zero planos, `present_to_window` recusando, falhas de `QueryInterface`/`CreateDescriptorHeap`/`GetBuffer`/`CreateRenderTargetView`), no máximo 4 vezes cada. Declara 13 testes `startup_present` passando. | Não. É diagnóstico. |
| #3367 | shader: lower `v_cvt_f64_i32` and `v_cvt_f64_u32` in direct compute path | `gfx1013_direct_compute.inl`: constrói o par binary64 exato com aritmética inteira de 32 bits e `dx.op.unaryBits.i32(33)` (`FirstbitHi`), sem exigir `DoublePrecisionFloatShaderOps` nem `Int64ShaderOps`; aceita VOP1/VOP3A com VGPR/SGPR/inline/literal e `D[0] == S0`, recusa estágio de pixel, `abs`/`neg`, `clamp`, `omod`, `op_sel`, `subword`, `dpp`. Registra o par de 64 bits em `clear_vector_spills`. Declara 3/3 testes com 2.064 palavras de readback batendo em RTX 5080. | Não. Fecha recusa do tradutor. |
| #3369 | shader: lower `v_cvt_f64_f32` in direct compute path | Mesmo desenho para `v_cvt_f64_f32` (`0x10`/`0x190`), com normais, subnormais binary32 normalizados, zero com sinal, `±Inf` e quieting de NaN preservando payload; aceita `abs`/`neg` do VOP3A. Declara 3/3 testes, 2.064 palavras conferidas. | Não. |
| #3371 | shader: lower `v_cvt_i32_f64` and `v_cvt_u32_f64` in direct compute path | Converte par binary64 (VGPR ou SGPR par) para inteiro de 32 bits saturado, truncando para zero, `NaN → 0`, saturação `[INT32_MIN, INT32_MAX]`/`[0, UINT32_MAX]`, com `abs`/`neg` aplicados ao sinal da fonte. Declara 3/3 testes. | Não. |

## Pull requests mesclados (1)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #3365 | gpu: normalize relocations in extended Mesh57 descriptors | `f5a6f6ba` (2026-11-28T14:15:27Z) | 3 arquivos, +103/−3, e o diff é exatamente o prometido no corpo: normaliza a realocação no descritor fixo reconhecido de `0x300` bytes ao formar a chave do perfil capturado e ao comparar um perfil com o descritor corrente, preservando metadados e os deslocamentos relativos dos ponteiros de shader record; layouts estendidos não reconhecidos mantêm os campos de seção. O teste de criação verifica que dois registros com endereços diferentes casam. |

## Cruzamento com o catálogo

- **Corrige D-xx? Por hash, nenhum.** Os 13 hashes da janela e o hash de merge `f5a6f6ba` foram
  conferidos por prefixo de 12 contra as 305 linhas do catálogo: zero interseção.
- **Classe de defeito de chave de cache, sem linha em `Xbox2Windows`.** `090de8f2`/`#3365` corrige uma
  chave de identidade que carregava o endereço de alocação do processo: dois estados idênticos de
  execuções diferentes geravam chaves diferentes (e o inverso também era possível ao comparar). É a
  mesma classe de **XCL D-11** (entradas de cache sem as fontes do lowering) e de **victory-fork D-02**
  (chave do cache de shader sem os switches), mas o catálogo de `Xbox2Windows` não tem linha para
  isso. **Hipótese minha, marcada como tal**: candidato a linha nova, sem hash de catálogo.
- **`7a163fc5` — área de liberação de recurso.** O mapa global por `ID3D12CommandList*` mantinha heaps
  de sampler vivos sem relação com a vida do allocator; o commit amarra a aposentadoria ao allocator e
  adiciona o cenário de duas filas com a segunda presa. É a janela de **Xbox2Windows D-01** ("superfície
  apresentada liberada durante a cópia da assinatura") em outro recurso; o diff **não** prova que o
  caso de D-01 esteja coberto.
- **Abre defeito novo?** Não comprovado. `c8e37cc6` muda o caminho de transferência de mesh por um
  valor padrão de parâmetro — uma linha, sem teste no mesmo commit, e é exatamente o tipo de mudança
  que só aparece em runtime; registro como risco, não como defeito.

## Confiança

**Verificado:** os 13 commits foram lidos por API com lista de arquivos, `+/-`, cabeçalhos de hunk e
primeiras linhas alteradas; `090de8f2`, `7a163fc5` e `c8e37cc6` tiveram patch lido em extensão. O
merge #3365 veio de `pulls/3365` (corpo, 3 arquivos) e o `merge_commit_sha` (`f5a6f6ba`) confere com o
head do repositório. Os 4 PRs abertos vieram de `pulls/<n>` com corpo completo.

**Não verificado:** nenhuma build nem teste; os números dos PRs (2.064 palavras de readback, 13 testes
de present, RTX 5080) são relato dos autores. Não abri o diff dos commits de sync (`4133425c`,
`86cf5aa8`, `48b2027c`, `303ecdd7`) além dos cabeçalhos — o que afirmo sobre eles é integração e nomes
de arquivo, não comportamento. Não sei se `0cdcfd00` reaplicou o `7a163fc5` sem perda: são commits de
resolução de conflito e o diff é idêntico em números, mas não comparei linha a linha com a `main`.

**Cobertura:** 13 de 13 commits não documentados. `last_commit` = `f5a6f6ba` (head); sem
`commit_cursor` — a fila deste repositório está zerada. `head_at_run` = `f5a6f6ba`. O head não andou
depois da coleta.
