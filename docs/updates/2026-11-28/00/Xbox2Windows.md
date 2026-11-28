# R0Y00/Xbox2Windows — atualização de 2026-11-28 00:00

## Resumo

12 commits e 12 PRs mesclados acima do cursor `3c30dc74`, todos squash de um pai casando 1:1 com os
merges da janela, e 14 PRs abertos. A hora é de **fechamento de recusas do compute direto**: quatro
lowerings novos (`s_cmov_*`, comparações SOPC de 64 bits, as quatro conversões `f16`↔inteiro de 16 bits e
`v_sat_pk_u8_i16` no `gfx2hlsl`), uma correção de falso-recusa no `MIMG` (`cf3df832`) e a leitura de
samplers de comparação (`smpc`) nos manifestos de compute (`a5dad843`). Os outros cinco commits são
ferramenta e teste: três endurecimentos de CLI, o filtro de adaptador do replay e uma nota de medição do
WARP que **nomeia três testes que falham** quando o WARP 1.0.22 é o app-local.

## Commits novos (12)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `a5dad84328` | saii | startup: read smpc comparison samplers in published compute manifests (#3075) (#3083) | `src/startup_stubs/d3d12_compute_dispatch.inl`: a linha `smpc` deixa de ser recusa própria (`return "a comparison sampler"`) e entra no mesmo `sscanf` de `smp`/`smpl` que chama `samplers.read_manifest_line`; só `smpheap` continua recusada, com motivo. O teste ganha o modo `dispatch-sampled-depth` — textura 2x1 `R32_FLOAT` com profundidades 1,0 e 2,5, `image_sample_c_lz` sob sampler point/clamp/LESS escrito como `smpc` — e compara as 320 palavras com um modelo de CPU, tendo como controle a mesma troca de função para GREATER (`tests/startup_compute_dispatch_texture_check.py` +96/−10) |
| `2e74b778af` | rdx | startup: lower v_sat_pk_u8_i16 in gfx2hlsl (#3033) | `tools/startup/gfx2hlsl.py`: entrada nova na tabela de lowering — satura a lane baixa de `S0` (com extensão de sinal) em 0..255 nos bits 7:0 e a alta em 0..255 nos bits 15:8, com `D[31:16] = 0`; a forma VOP3A `v_sat_pk_u8_i16_e64` ganha guarda de codificação que recusa abs/neg e `src1`/`src2` não usados. Teste de 242 linhas em `tests/startup_gfx2hlsl_v_sat_pk_u8_i16_check.py` |
| `3a550d0afc` | John Connor | shader: read inline float sources as binary16 in v_frexp_mant_f16 and v_frexp_exp_i16_f16 (#3147) | `src/gfx1013_direct_compute.inl`: a tabela das nove constantes inline em binary16 (`inline_half_words`) sai de dentro de um caso e vira constante do tipo; o caminho do `v_frexp_*_f16` deixava de ler a constante inline pelo padrão f32 (`operand32` + máscara de 16 bits) e passa a ler o valor binary16, como `v_add_f16` já fazia. As duas ocorrências da tabela passam a usar a mesma constante. O documento da experiência ganha a nota e o check fixa o caso (43 linhas novas) |
| `8e8b349bda` | saii | docs: measure native wave32 compute on app-local WARP 1.0.22 (#3160 stage 1) (#3166) | Só `docs/experiments/native-compute-warp.md` (+120). Medição: Agility 1.620.2 com o WARP de caixa reporta `4..4` lanes; o `d3d10warp.dll` 1.0.22 colocado ao lado dos EXEs reporta `WaveOps=1`, lanes 4 a 128 e SM 6.8. Com ele, três testes `_warp` passam e **três falham**: `startup_compute_dispatch_native_warp` (`word=0 got=c5c5c5c5 expected=ea74e376`, 288 palavras diferem), `native_startup_request_cache_warp` (`compute dispatch output mismatch`) e `bvh_wave64_warp` (#145: `record=0 field=25 expected=f0094d90 actual=f03aeb40`). Nenhuma linha de código muda |
| `671d2f4633` | legacy-code-archaeologist | shader: lower s_cmov_b32, s_cmov_b64 and s_cmovk_i32 in the direct compute path (#3071) | `src/gfx1013_direct_compute.inl`: emite `D = SCC ? S0 : D` (par de palavras em `s_cmov_b64`) com SCC inalterado, com recusa de destino ímpar/fora da faixa e de fonte inválida; `s_cmovk_i32` usa o imediato de 16 bits com sinal. As três instruções entram no conjunto `reads_scc` da varredura de vivacidade — sem isso a passada podia considerar SCC morto antes do `s_cmov`. `src/gfx1013_compute_native.cpp` põe `s_cmovk_i32` na lista de transferência escalar. Teste de 313 linhas |
| `98f7363d13` | legacy-code-archaeologist | shader: lower 64-bit SOPC compares in direct compute path (#3081) | `s_cmp_eq_u64`, `s_cmp_lg_u64`, `s_bitcmp0_b64` e `s_bitcmp1_b64` passam a ser baixadas como comparação de dois pares de 32 bits e `SCC = zext i1`; no `bitcmp`, o bit 5 do índice escolhe a palavra alta (`select upper, a_hi, a_lo`) e recusa fonte de índice inválida e `106`/`126`. Teste de 329 linhas |
| `5fc9f6ff05` | RonLogic | tests: honor the hardware adapter filter in native compute replay (#3155) | `tests/native_store_replay.cpp` passa a criar o device com `xmem_test_device.h` (que ganha um `FILE*` de diagnóstico, `stderr`) em vez de `D3D12CreateDevice(nullptr, …)`, imprime LUID, nome/vendor/device e versão do driver por DXGI, e sai com código 2 quando `XBOX2WINDOWS_ADAPTER` não casa — antes o filtro era ignorado e o replay rodava em qualquer placa. Teste `native_store_replay_adapter_controls` novo |
| `69530cc267` | Ammar Abdelfattah | tooling: harden atomic writes, path guards, and CLI entry in cpu_leaf_aot with synthetic PE32+ suite (#3111) | `tools/cpu_leaf_aot.py`: `translate()` valida `start` (inteiro, não booleano, não negativo) e `code` (bytes-like); saída tem de ter sufixo `.cpp`, não pode ser symlink nem diretório, e cada pai é verificado; escrita passa a ser atômica (tmp irmão + replace). Suíte nova de 691 linhas com PE32+ sintético |
| `98020080c6` | legacy-code-archaeologist | shader: lower v_cvt_f16_u16, v_cvt_f16_i16, v_cvt_u16_f16 and v_cvt_i16_f16 in direct compute (#3159) | As quatro conversões entram em `DirectCompute::float_alu`: preservam `D[31:16]`, usam `sitofp`/`uitofp` + `packed_half_rne` no sentido inteiro→f16 (com `clamp` VOP3 opcional) e `legacyF16ToF32` + truncamento + saturação + NaN→0 no sentido f16→inteiro; contrato de modo binary16 verificado, com recusa de DPP, SDWA, `omod`, `op_sel` alto e modificadores ilegais. 82 casos de conversão e 38 de recusa no arquivo de teste (398 linhas) |
| `9e7eab5b51` | Ammar Abdelfattah | shader: harden import_gfx10_tables CLI, atomic writes, and parser guards with offline unit suite (#3115) | `tools/startup/import_gfx10_tables.py`: valida que a entrada é arquivo regular, recusa saída symlink/diretório e saída igual à entrada, verifica cada pai, escreve por tmp irmão com replace e mantém o `sys.path` idempotente; o SHA-256 fixo de `opcodes.rs` continua sendo a única entrada aceita. Suíte offline de 642 linhas |
| `3d7b3a2692` | Ammar Abdelfattah | tooling: harden atomic writes, path guards, and CLI entry in cpu_call_aot with synthetic PE32+ suite (#3126) | `tools/cpu_call_aot.py`: `--bind` passa a exigir provedor existente em disco e símbolo não vazio; `link_name` vazio recusado; validação de caminhos e escrita atômica iguais às do `cpu_leaf_aot`. Suíte nova de 882 linhas |
| `cf3df832b3` | Leyukaka | compute: keep MIMG sampler index bit 4 out of the reserved-bit check (#3146) | Nos dois emissores (`src/gfx1013_compute_native.cpp` e `src/gfx1013_direct_compute.inl`) a máscara de bits reservados da palavra 1 do `MIMG` muda de `0x3e000000` para `0x3c000000`: o bit 25 deixa de provocar `unsupported image flags`. O commit atribui esse bit ao índice de sampler (4º bit), mas o diff não lê o bit 25 em lugar nenhum — a justificativa vem da mensagem e do teste `native_sample_ssamp` (58 linhas) |

## Pull requests abertos (14)

Os 14 acima do cursor `last_pr_number` 3169, `created_at` entre 20:22Z e 21:03Z no relógio do GitHub;
título e corpo lidos, patch não lido.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| `#3191` | startup: lower wave32 reductions around uniform pixel branches | Aceita redução OR/MIN/MAX wave32 em estágio de pixel com ramo **uniforme** antes e/ou depois da cadeia; ramo não provado uniforme continua recusado, wave64 continua recusado | Não no catálogo |
| `#3190` | tests: define SENTINEL in native_f16_alu_check | Define `SENTINEL = 0x69696969` no módulo que o teste de `#3159` importa — conserta o `AttributeError` que derruba `native_cvt_f16_i16_hardware` na `main` | **Defeito novo desta janela** (ver Cruzamento) |
| `#3189` | compute: classify SCC-neutral SOP1 ops in the direct SCC liveness | Classifica `s_ff0/1`, `s_flbit`, `s_brev`, `s_bitset0/1`, `s_bitreplicate_b64_b32` como neutros de SCC e os que escrevem; `s_cmov` não é tocado | Fixes `#3174`, mesma passada de vivacidade que `671d2f46` alterou — não é D-xx |
| `#3188` | compute: accept DPP16 on binary16 VOP2 add, sub, subrev and mul | DPP16 nas quatro VOP2 binary16, com FI/BOUND_CTRL e máscaras de linha/banco; DPP8, clamp e omod seguem recusados | Não |
| `#3187` | shader: lower v_fma_f16, v_fmac_f16, v_fmamk_f16 and v_fmaak_f16 in the direct C++ path | FMA binary16 de arredondamento único em inteiro 32 bits (evita duplo arredondamento por binary32), preserva metade alta e aplica clamp VOP3 | Não; mesma família de `#3159` |
| `#3185` | test: cover direct compute 32-bit integer min3 max3 and med3 | Só teste: seis formas canônicas e 11 casos contra oráculo independente para `v_min3/max3/med3` u32/i32 | Não |
| `#3180` | compute: accept vcc_lo and vcc_hi as wave32 SMEM offsets and destinations | `vcc_lo`(s106)/`vcc_hi`(s107) como offset e destino de `s_load_*` wave32; limite de SGPR vira 108 em wave32, 106 nos demais | Não |
| `#3179` | compute: accept EXEC as the destination of s_bitreplicate_b64_b32 | `s_bitreplicate_b64_b32` aceita o par `s126/s127` como destino, com `active()` lendo s126 em wave32 | Não; `#3030` cobriu o mesmo opcode no `gfx2hlsl` |
| `#3178` | runtime: count decoded Opus completions and desktop audio submissions | Telemetria saturada de decodificação Opus por motivo de recusa e `accepted`/`not_open` do sink | Não |
| `#3177` | runtime: add opt-in input trace for the virtual gamepad | `X2W_INPUT_TRACE=1` imprime o estado do gamepad virtual depois do merge de teclado/XInput/script | Não |
| `#3176` | tooling: report the startup exit code and last presented frame | `tools/startup/exit_report.py`: código de saída, rótulo fixo, último frame apresentado, sem copiar caminhos nem texto livre | Não |
| `#3175` | tooling: report the first scene-colour write from draw dumps | `tools/startup/scene_color_report.py`: primeira mudança de cor e último escritor por frame/pixel, com classificação da história | Não |
| `#3173` | tooling: add a startup UI regression matrix | Matriz pass/skip/fail sobre os fixtures de UI do startup, mantendo 77 separado de falha | Não; é ferramenta de verificação |
| `#3172` | compute: accept raw buffer loads without offen or with a scalar offset | `buffer_load_dword*` sem `offen` e com `soffset` SGPR/inline numérico no compute direto; `m0` continua recusado; `idxen`/`glc`/`dlc` seguem recusados | Não; Fixes `#3143` |

## Pull requests mesclados (12)

Todos os 12 com `merged_at` na janela casam 1:1 com um commit da faixa; o SHA de merge é o próprio
commit (squash).

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| `#3083` | startup: read smpc comparison samplers in published compute manifests | `a5dad843` (20:09:12Z) | O `smpc` deixa de ser recusa e é lido como sampler estático; o caso `dispatch-sampled-depth` mede o resultado contra modelo de CPU. O PR fecha a issue `#3075` (fechada, não é PR) |
| `#3033` | startup: lower v_sat_pk_u8_i16 in gfx2hlsl | `2e74b778` (20:21:27Z) | A tabela de lowering, a guarda de codificação e o check de 242 linhas |
| `#3147` | shader: read inline float sources as binary16 in v_frexp | `3a550d0a` (20:28:13Z) | A leitura da constante inline pelo padrão binary16 e a tabela compartilhada |
| `#3166` | docs: measure native wave32 compute on app-local WARP 1.0.22 | `8e8b349b` (20:28:54Z) | Só o documento de medição, com as três falhas nomeadas |
| `#3071` | shader: lower s_cmov_b32, s_cmov_b64 and s_cmovk_i32 | `671d2f46` (20:33:06Z) | O `select` sobre SCC, a entrada em `reads_scc` e o `s_cmovk_i32` na transferência escalar |
| `#3081` | shader: lower 64-bit SOPC compares | `98f7363d` (20:35:22Z) | As quatro comparações de 64 bits e o teste de 329 linhas |
| `#3155` | tests: honor the hardware adapter filter in native compute replay | `5fc9f6ff` (20:36:31Z) | O device passa a ser criado pelo filtro, com diagnóstico em stderr e saída 2 quando não casa |
| `#3111` | tooling: harden cpu_leaf_aot | `69530cc2` (20:41:33Z) | As guardas de tipo/caminho e a escrita atômica |
| `#3159` | shader: lower v_cvt_f16_u16, v_cvt_f16_i16, v_cvt_u16_f16, v_cvt_i16_f16 | `98020080` (20:43:02Z) | As quatro conversões e os 82+38 casos do teste |
| `#3115` | shader: harden import_gfx10_tables CLI | `9e7eab5b` (20:45:09Z) | Guardas de caminho, escrita atômica e a suíte offline |
| `#3126` | tooling: harden cpu_call_aot | `3d7b3a26` (20:55:40Z) | Validação de `--bind`/provedor e escrita atômica |
| `#3146` | compute: keep MIMG sampler index bit 4 out of the reserved-bit check | `cf3df832` (21:02:29Z) | A máscara `0x3c000000` nos dois emissores e o teste `native_sample_ssamp` |

## Cruzamento com o catálogo

- **Nenhuma das 30 entradas de `Xbox2Windows` no `catalogo/defeitos.csv` é reaplicada ou corrigida por
  estes 12 commits.** As mais próximas falham por área: D-11 (testes de hardware sem
  `SKIP_RETURN_CODE 77`) não é `5fc9f6ff`, que muda o filtro de adaptador e a saída 2, não o código de
  skip; D-08 (CLI que ignora `--output`, vaza traceback e sai 0) é a mesma **classe** dos três
  endurecimentos de CLI (`69530cc2`, `9e7eab5b`, `3d7b3a26`), mas em outros três scripts — o
  `disasm_rdna2` não é tocado; D-27 (`image_sample` com coordenadas `a16`) não é `cf3df832`, que mexe na
  máscara de bits reservados.
- **Defeito novo N1, verificado no código.** `98020080` adiciona
  `tests/native_cvt_f16_i16_check.py`, que compara as palavras de guarda com `F.SENTINEL`, importando
  `native_f16_alu_check as F`; nesse commit `tests/native_f16_alu_check.py` **não define** `SENTINEL` — ele
  escreve o literal `0x69696969` inline (lido por `contents` na API em `98020080`: `SENTINEL` ausente,
  `0x69696969` presente uma vez; no outro arquivo, `F.SENTINEL` presente). Resultado: a perna
  `native_cvt_f16_i16_hardware` levanta `AttributeError` em vez de asserção de guarda. O conserto está no
  PR aberto `#3190`, desta mesma hora.
- **Achado N2 (não é defeito do catálogo, é falha aberta).** `8e8b349b` registra que, com o WARP 1.0.22
  app-local, três testes passam a **falhar** — `startup_compute_dispatch_native_warp` (288 palavras
  diferem, `word=0 got=c5c5c5c5 expected=ea74e376`), `native_startup_request_cache_warp` e
  `bvh_wave64_warp` (#145). Nenhum conserto entra nesta janela; os três ficam nomeados no documento.
- **Lacuna de vivacidade de SCC.** `671d2f46` altera o conjunto `reads_scc` (para que `s_cmov*` não perca
  o escritor de SCC). A issue `#3174`, aberta às 20:23Z, descreve o problema mais amplo: a passada trata
  todo SOP1 fora de uma lista curta como leitor de SCC, e um SCC escrito por reescrita de descritor fica
  vivo além de instruções neutras. O PR `#3189`, aberto nesta hora, fecha isso sem tocar `s_cmov`.
- **Novo candidato a linha do catálogo** (para o operador decidir): falha de teste por símbolo ausente
  entre módulos de teste (`98020080` → `#3190`), e o trio de falhas sob WARP 1.0.22 (`8e8b349b`).

## Confiança

- **Verificado no diff:** os 9 commits de código e documento desta faixa foram lidos no patch completo
  (`a5dad843`, `2e74b778`, `3a550d0a`, `8e8b349b`, `671d2f46`, `98f7363d`, `98020080`, `cf3df832` e o
  teste de `5fc9f6ff`). Os três de ferramenta (`69530cc2`, `9e7eab5b`, `3d7b3a26`) foram lidos nos
  primeiros ~2.000 caracteres de patch de cada arquivo — as guardas de caminho e a escrita atômica estão
  confirmadas, o resto da suíte sintética não foi lido linha a linha.
- **Verificado por API:** os 12 merges (`merged_at`, `merge_commit_sha` = commit da faixa), a ausência de
  PR nos três commits de `TuxBox`/`victory-fork`, o estado aberto de `#3174` e o conteúdo de
  `tests/native_f16_alu_check.py` e `tests/native_cvt_f16_i16_check.py` em `98020080`.
- **Não verificado:** nenhum teste foi executado nesta hora; as medições de hardware (RTX 5080, RTX 4070,
  Quadro M1000M) são as dos corpos dos PRs e do documento; a atribuição do bit 25 do `MIMG` ao índice de
  sampler não foi conferida contra a ISA; os 14 PRs abertos foram lidos por título e corpo, sem patch.
- **Contagem:** 12 commits na faixa = 12 merges com `merged_at` na janela. Nenhuma divergência.
