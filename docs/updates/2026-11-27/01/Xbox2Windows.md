# R0Y00/Xbox2Windows — atualização de 2026-11-27 01:00

## Resumo

Cinco commits na `main` (`20a497c2..8895045b`), todos o resultado em squash de PRs abertos contra a
própria `main`. Três são do tradutor de startup (`tools/startup/gfx2hlsl.py`): `image_get_lod` passa a
aceitar 2D-array consumindo o dword da fatia e consultando o LOD só pelas coordenadas espaciais (#2947);
um diamante SCC provado pode selecionar **dois** samplers inline de quatro palavras em `image_sample_l`/
`_lz` (#2970); e `ds_sub_u32` em LDS vira duas `InterlockedAdd` porque um `InterlockedAdd` de `sub i32 0, x`
em groupshared derruba a criação do PSO de computação na RTX 5080 (#2954). Os outros dois só endurecem
validação de entrada de ferramentas Python (`import_routing` #2879, `host_commit` #2885) e trazem suítes
unitárias. Na janela foram mesclados dez PRs, mas cinco deles têm base `cpp-migration` e **não** estão no
histórico da `main`. Nenhum dos cinco commits toca defeito do catálogo.

## Commits novos (5)

Cadeia linear conferida pelos pais: `20a497c2 → b80ed067 → a4fae199 → fb955a51 → e05b3ced → 8895045b`,
um pai cada. Todos os assuntos trazem o número do PR entre parênteses, o que confirma o squash.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `b80ed067` | Ammmae0x | pe: harden import routing input validation (#2879) | 3 arquivos, +266/-0 (2 novos). `tools/import_routing.py` (+15) ganha `_validate_module`: recusa com `ValueError` nome que não é `str`, string vazia, qualquer `ch.isspace()` e qualquer caractere `< 0x20` ou em `0x7f..0x9f`; a chamada entra como primeira linha de `classify`, antes do `module.lower()`. `annotate` passa a exigir `MutableMapping` (`TypeError`) e a presença de `module_key` (`KeyError`) **antes** de tocar na linha. `tests/import_routing_check.py` (novo, 245 linhas) cobre as quatro rotas, maiúsculas misturadas, prefixo `api-ms-win-`/`ext-ms-` contra substring, 22 nomes inválidos, e seis mutações aplicadas a cópias temporárias do módulo (derrubar cada guarda tem de falhar a suíte). |
| `a4fae199` | Ammmae0x | runtime: validate host commit budget inputs (#2885) | 3 arquivos, +266/-4 (1 novo). `tools/host_commit.py` (+21/-4): `_is_int` exclui `bool`; `snapshot_from_pages` recusa `page_size`/`limit_pages` não inteiros ou `<= 0` e `used_pages` não inteiro ou negativo (antes só testava `not page_size or not limit_pages`, aceitando `True` e `-1`); `require_commit_budget` passa a exigir um `Mapping` com as quatro chaves (`available_bytes`, `limit_mib`, `used_mib`, `available_mib`) e cada valor inteiro não negativo. O clamp `max(limit_bytes - used_bytes, 0)` fica. `tests/host_commit_unit_check.py` (novo, 239 linhas) cobre matemática exata em MiB, clamp, recusas de bool/float/negativo/zero, bordas de `minimum_mib` (0, 6 de 6, 6 menos 1 byte, 2^20, 2^20+1), `GetPerformanceInfo` mockado com sucesso e falha, e seis mutações (inclusive remover `winmode=0x800`). |
| `fb955a51` | chadhyatt | shader: lower two immutable samplers at uniform explicit-LOD joins (#2970) | 10 arquivos, +368/-9. Em `gfx2hlsl.py` (+81) entra `sampler_join_shape`: exige um único diamante para a frente (`s_cbranch_scc0/scc1` → `s_branch` → junção), nenhuma escrita de EXEC nem `s_endpgm` dentro dele, nenhum destino escalar em "any" (arquivo inteiro) nos braços; em cada braço as **quatro** palavras do sampler têm de ser escritas como literais (`s_mov_b32` imediato). O `image_sample_l`/`_lz` com `dim:SQ_RSRC_IMG_2D` e registro `sN..sN+3` (N+3 ≤ 105) junta o par; `joined_sampler_call` valida as palavras (só wrap/clamp normalizado nos bits 0-5 de w0, w1 sem bits acima de 0xffffff, w2 == `1<<26` para mip filter pontual, w3 == 0) e emite `(sN+k == 0x…u ? A.SampleLevel(…) : B.SampleLevel(…))` escolhendo pela primeira palavra que difere. Qualquer escrita posterior em um dos quatro registradores apaga o fato. No C++: `tests/startup_image_3d.cpp` passa a ler **duas** linhas de manifesto obrigatoriamente ordenadas (`smp 0` e `smp 1`, sem heap, sem comparação, `statics.size()==2`) e `offscreen_check --startup-image-2d-view` aceita o seletor `0\|1` como segundo root constant. Fixture novo `startup_image_sample_l_join_check.py` (219 linhas, 10 variantes de assembly e 7 recusas). |
| `e05b3ced` | alvaroolt | shader: lower group-shared ds_sub_u32 as two additions (#2954) | 3 arquivos, +47/-8. `tools/compute_store_aot.py`: `ds_sub_u32` deixa de ser `InterlockedAdd(lds[i], (0u - v))` e vira duas chamadas no mesmo dword, `~(v)` e depois `1u` (igual a `mem - v` módulo 2^32); nenhuma outra atômica muda. `tests/compute_store_check.py` passa a exigir as duas linhas e a ausência de `0u - ` no programa contador, e acrescenta uma mutação do HLSL que remove a segunda adição: tem de divergir na palavra 1 (64 curto, uma por lane). A documentação registra o gatilho: na RTX 5080 (driver 32.0.16.2124) `CreateComputePipelineState` devolvia `E_OUTOFMEMORY` para o programa com `InterlockedAdd` de `sub i32 0, x` em groupshared, e o DXC dobra `~x + 1u`, `x * 0xffffffffu`, `-(int)x` e `-Od` para a mesma instrução; o mesmo xadrez com UAV, `InterlockedExchange`, `Xor` ou `Max`, ou com `5u - x`, não falha. |
| `8895045b` | chadhyatt | shader: lower startup 2D-array image LOD queries (#2947) | 7 arquivos, +175/-48. Em `gfx2hlsl.py` o ramo `get_lod` de `image()` aceita `dim:SQ_RSRC_IMG_2D_ARRAY`, exige `want_addr = 3` dwords de endereço (2 no 2D) e mantém todas as recusas (3D, `s_cbranch_vccz/execz` antes da leitura, bias de LOD do sampler, mipmapping desligado). A fatia é consumida mas descartada: `elif three and how != 'get_lod'` deixa `uv` em `float2`, o que casa com `CalculateLevelOfDetail(float2)` de `Texture2DArray`. No C++, `--startup-image-2d-array-view` reusa `run_startup_image_2d_view` com `array=true`, e `check_startup_image_3d` passa a montar SRV `D3D12_SRV_DIMENSION_TEXTURE2DARRAY` (`Texture2DArray.MipLevels`, `ArraySize`) quando há janela de mips. Fixtures: variante NSA com endereço não sequencial (`[v0, v1, v3]`), duas fatias, duas escalas de derivada, recusa de array decodificado com dois registradores de endereço. |

## Pull requests abertos (5 novos)

`gh pr list --state open` devolveu 5 PRs acima do cursor 2984, todos com base `cpp-migration`; nenhum
deles entra no histórico da `main`.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 2986 | shader: lower v_alignbit_b32, v_alignbyte_b32 and v_bfm_b32 in the direct C++ compute path | Baixa as três instruções no frontend direto de computação com as expressões da tabela VOP3 do `gfx2hlsl.py`, mascarando o deslocamento a 5 bits e resolvendo o caso zero por `select`. O readback de hardware ficou pulado (77: precisa de SM 6.6 com wave32). | Não. |
| 2993 | runtime: bind table buffers, 2D texture inputs and texture outputs on the in-process compute route | A rota in-process (`X2W_NATIVE_SHADERS` → `translate_compute_natively`) passa a preencher `buffer_srv`, `texture_srv` 2D e `texture_uav`/`half4_texture_uav` com as mesmas regras de slot do manifesto; samplers e `byte_window_srv` continuam recusados. Diz depender do #2975. | Não. |
| 2996 | shader: lower v_cvt_pk_u16_u32 and v_cvt_pk_i16_i32 in the direct C++ compute path | Empacota S0/S1 saturando a 16 bits (unsigned e com clamp assinado), recusando SRC2 não zero, `op_sel` e modificadores; notas admitem que a saturação não é medida no console. Readback de hardware pulado. | Não. |
| 2998 | shader: lower v_cvt_pk_u8_f32 in the direct C++ compute path | Converte com `FMin` 255 + `Round_ne` + `fptoui` (round-to-nearest-even, saturando; NaN, zeros e negativos dão 0, +inf dá 255) e funde o byte em S2. O próprio corpo registra que a revisão pegou uma primeira versão com `FMax`. | Não. |
| 3002 | tooling: canonicalize script paths when checking duplicates | `run_checks_parallel.py` passa a deduplicar por `(Path(argv[0]).resolve(), *args)`, mantendo o caminho exibido; cobre variantes como `tests/a.py` e `./tests/a.py`. | Não. |

## Pull requests mesclados (10)

Cinco fecham na `main` e são exatamente os commits acima; os outros cinco têm base `cpp-migration`.

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #2947 | shader: lower startup 2D-array image LOD queries | `8895045b` | O que o PR promete, e nada além: 2D-array em `image_get_lod` com a fatia consumida e descartada, SRV 2D-array para janela de mips, fixture de duas fatias e duas derivadas. O corpo declara que DXC/WARP/GPU não rodaram no host (Apple Silicon) e que só a checagem portátil passou. |
| #2954 | shader: lower group-shared ds_sub_u32 as two additions | `e05b3ced` | O workaround do `E_OUTOFMEMORY` da RTX 5080 mais o teste de mutação e a nota de medição por adaptador (WARP, RTX 5080, iGPU Radeon). O PR declara `+47/-8`; o commit tem `+47/-8`. |
| #2970 | shader: lower two immutable samplers at uniform explicit-LOD joins | `fb955a51` | O diamante de dois samplers com as três provas (diamante para a frente, palavras literais nos dois braços, sem escrita de EXEC) e o fixture de 10 variantes. O corpo fala em "eight exact assembly variants" no texto e "ten" no documento novo; o diff tem as duas contagens — registro a divergência sem resolver qual vale. |
| #2885 | runtime: harden bool/integer and page bounds in host_commit with pure-Python unit suite | `a4fae199` | Exatamente o descrito: guardas de tipo/borda e a suíte `tests/host_commit_unit_check.py` com seis mutações de cópia temporária. |
| #2879 | pe: harden module name and row schema guards in import_routing with standalone unit suite | `b80ed067` | Exatamente o descrito: `_validate_module`, exigência de `MutableMapping`/`module_key` e a suíte com seis mutações. |
| #2989 | shader: resolve and cache RTV variant requests in compile_dxil --requests | `e0d83759` (base `cpp-migration`) | 13 arquivos, +635/-22. `read_graphics_request` aceita `req_<key>.rtv` (`rtv1 <base> <formato>…`, FNV-1a do texto no nome), tipa os alvos do estágio de pixel pelos formatos ordenados (`u32`/`i32` para inteiros, `f32` caso contrário; formatos 7 e 8 recusados por nome), põe os tipos na chave do `.pairir` e faz o manifesto terminar com o texto do pedido; recusa variante sem manifesto de runtime e contêiner cuja assinatura de saída discorda do formato. |
| #2992 | tooling: add --watch mode to compile_dxil for live startup shader requests | `79a8db6c` (base `cpp-migration`) | 4 arquivos, +562/-0: modo contínuo e `--once` com N workers, retomada de `.cso` truncado, nova checagem quando o `.psin` é mais novo, retry de `.fail` na primeira passada e `.fail` para pedido sem manifesto. |
| #2994 | shader: lower v_ashrrev_i64 and v_mad_i32_i24 in the direct C++ compute path | `c6908cce` (base `cpp-migration`) | 4 arquivos, +228/-3: `ashr i64` com extensão de sinal de literal de 32 bits (os deslocamentos lógicos continuam zero-extendendo) e `sext(S0[23:0]) * sext(S1[23:0]) + S2` módulo 2^32. Readback de hardware pulado. |
| #2990 | shader: lower the SAD operations in the direct C++ compute path | `b69c111f` (base `cpp-migration`) | 4 arquivos, +216/-0: `v_sad_u8`, `v_sad_hi_u8`, `v_sad_u16`, `v_sad_u32` e `v_msad_u8` com as somas de diferenças absolutas da tabela do `gfx2hlsl.py`; modificadores de operando seguem recusados. |
| #2984 | tooling: write .fail diagnostics and add read-only --refusals summary to compile_dxil | `4ff5e36d` (base `cpp-migration`) | 8 arquivos, +692/-19: ciclo `.fail` por pedido em `--requests`, `--refusals` somente-leitura com agregação normalizada (PCs, offsets, índices e registradores trocados por marcadores) e classificação *allowlist* em `triage_run.py`. |

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum. Os dois candidatos plausíveis já constam corrigidos antes desta janela, e
  verifiquei os hashes: **D-19** ("documentação do ciclo de `.fail` ficou obsoleta") foi corrigido em
  `5b29fe0f` (2026-11-24T23:34:30Z, "docs: match the iteration section to the retry and .fail rules on
  main"); **D-20** ("tabela `SHADER_REASONS` nunca exercitada") em `13745b1b` (2026-11-24T22:59:56Z,
  "test: pin every shader reason token in the triage classifier"). O #2984, mesclado nesta janela na
  `cpp-migration`, toca os dois arquivos (`docs/experiments/native-compile-dxil-refusals.md` +37 e
  `tests/startup_triage_run_check.py` +19), mas é o PR que originou esse ciclo — não é correção nova.
  **D-27** (coordenadas 2D `a16` lidas como dois registradores binary32) não tem hunk correspondente no
  #2947 nem no #2970; `a16` não aparece nos fixtures novos. **D-06/D-28** (arredondamento f32 e conversão
  UNORM/SNORM) são de shader de pixel/`store` e não são tocados pelos cinco commits.
- **Abre defeito novo?** Nenhum demonstrado por diff. Dois pontos ficam registrados:
  - `e05b3ced` troca **uma** atômica por **duas** no mesmo dword. Não há retorno de valor e o passo termina
    em `GroupMemoryBarrierWithGroupSync()`, então nenhuma lane lê a palavra intermediária; mas o par deixa
    de ser indivisível para qualquer leitor que não seja o próprio passo. É a razão declarada de o perfil
    baixar só a forma não-retornante.
  - `fb955a51` aceita dois samplers inline num diamante, com validação restrita (`w0 & ~0x3f`, `w1 &
    ~0xffffff`, `w2 == 1<<26`). Um sampler com endereçamento fora de wrap/clamp pontual é recusado, não
    traduzido errado; é lacuna declarada, não regressão.

## Confiança

Verificado: os 5 SHAs e metadados em `repos/R0Y00/Xbox2Windows/commits?sha=main&per_page=100`, com o
cursor `20a497c2` na posição 5; cada commit lido em `commits/<sha>` (lista de arquivos, contagens e patch);
a cadeia de pais, um por commit, sem merge commit; o patch completo de `tools/import_routing.py`,
`tools/host_commit.py`, `tools/compute_store_aot.py`, `tools/startup/gfx2hlsl.py` (os dois hunks do
`image()` e o bloco `sampler_join_shape`/`joined_sampler_call`) e de `tests/startup_image_3d.cpp`; os 10
PRs em `pulls?state=closed&sort=updated` filtrados por `merged_at > 2026-11-27T03:01:12Z`, com
`merge_commit_sha` igual ao SHA do commit da `main` nos cinco de base `main`; a existência e a data de
`5b29fe0f` e `13745b1b`.

Não verificado: a execução de `ctest` e das cenas de GPU citadas nos corpos (WARP/hardware, DXC `1.9.2609`);
se os 5 merges de base `cpp-migration` já estão em algum ramo publicado (a `main` não os contém — pendência
estrutural já registrada no estado); qual das duas contagens do #2970 (oito ou dez variantes) vale; e se um
leitor concorrente das palavras LDS em `e05b3ced` existe fora do passo barrado. Nenhum teste foi executado
aqui; os 13 repositórios monitorados são somente leitura.
