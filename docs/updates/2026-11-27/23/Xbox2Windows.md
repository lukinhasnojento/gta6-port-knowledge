# R0Y00/Xbox2Windows — atualização de 2026-11-27 23:00

## Resumo

5 commits e 5 PRs mesclados acima do cursor `88cddb09` — todos squash de um pai, casando 1:1 com os
merges da janela. Duas frentes: diagnóstico de memória do `startup` (linha `x2w-budget` com commit do
sistema, limite de job e contadores do pool; causa nomeada para cada `texture_unavailable`) e lowering de
shader no tradutor (`v_cvt_norm_i16/u16_f16` em 10 formas, `s_bitreplicate_b64_b32`). 5 PRs abertos.

## Commits novos (5)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `6057071b` | tedin7 | startup: system commit, job limit and page-pool counters in the budget line (#2833) | Novo `src/startup_stubs/commit_state.h` (+96, header-only): `SystemCommit` de `GlobalMemoryStatusEx` (`ullTotalPageFile`/`ullAvailPageFile` → `commit=<used>/<limit> avail=<free>` em MiB), `JobMemory` de `QueryInformationJobObject` (`JobMemoryLimit`/`PeakJobMemoryUsed`, `job_peak`/`job_limit`/`none`), `job_limit_reached(slack=256 MiB)` e `format_exhausted`, que nomeia o limite de job quando `peak+256 ≥ limit` e imprime `sections/slabs/slots`. Em `d3d12_xs_device.cpp::report_budget`, o `log("x2w-budget …")` ganha os três campos; os contadores vêm de `x2w_xmem_page_stats`/`x2w_xmem_slab_stats` por `GetProcAddress` em `xmem.dll`. Em `xmem_mapped_pages.cpp`, as **três primeiras** falhas de commit do pool (erro 1455/8) imprimem a mesma linha. Teste novo `startup_commit_state_check.cpp` (+107) |
| `0f3e8248` | tedin7 | startup: name the cause of a slot 70 texture_unavailable refusal in the memory SRV census (#2836) | `memory_srv_census.h` separa o que `texture_unavailable` juntava em `texture_no_resource`, `texture_unreadable` e `texture_over_64mib` (o limite `max_published_texture_bytes = 64 MiB` sai do literal em `publish_tiled_texture`), com `texture_unavailable_cause(published, bytes, readable)` — o `readable()` só roda para layout dentro do limite, então layout enorme não paga varredura de página. `d3d12_memory_srv.inl` chama a função nos dois pontos de recusa e passa a decidir `published` por `tiled_bc_srv || bc64::supports || console_layout_srv`. `triage_run.py` e os checks ganham o motivo. Log e contagem apenas — nenhuma view é publicada ou recusada de forma diferente |
| `968ddcdf` | KomaGuy | pixel_aot: lower 10 binary16 cvt_norm forms across VOP1/VOP3/DPP16/DPP8/SDWA (#2857) | `tools/pixel_aot.py` ganha dois helpers HLSL (`cvt_norm_i16_f16`, `cvt_norm_u16_f16`) que fazem a conversão em **inteiro 32 bits** com arredondamento RNE explícito (`rem > mid`, ou `rem == mid` e `q` ímpar), cobrindo subnormais (`shift = 24`) e saturação em `0x3c00`; e `f16_cvt_norm_result`, o oráculo em Python que recusa operando não finito. Os quatro caminhos novos são SDWA, DPP16/DPP8, e o `v_cvt_norm_(i16\|u16)_f16[_e32\|_e64]` simples, todos produzindo `("low16", alu)`. Testes: `pixel_aot_check.py` +311 e `form_coverage_check.py` +43 |
| `454fc0ed` | tedin7 | startup: X2W_LOW_ADDRESSES keeps the title's own memory below 2^40 (#2859) | `tools/startup/launch.py` ganha `STARTUPINFOEXW`, `PROC_THREAD_ATTRIBUTE_MITIGATION_POLICY` (0x20007) e `PROCESS_CREATION_MITIGATION_POLICY_BOTTOM_UP_ASLR_ALWAYS_OFF` (0x2<<16); `startup_for_create(si, env)` monta a lista de atributos e devolve `(estrutura, EXTENDED_STARTUPINFO_PRESENT, objetos a manter vivos, nota)`. Se a lista ou o atributo falharem, devolve `si` **inalterado** e só imprime a nota — o run continua sem a política. `debug_launch.py` usa o mesmo caminho; `run.ps1` ganha `-LowAddresses`. `startup_launch_probe.cpp` (+17) e `startup_launch_check.py` (+40) testam que controles de `startup_launch` não herdam `X2W_LOW_ADDRESSES` |
| `3c30dc74` | rdx | startup: lower s_bitreplicate_b64_b32 in gfx2hlsl (#3030) | `tools/startup/gfx2hlsl.py` ganha `op_s_bitreplicate_b64_b32` (SOP1 0x3b): exige destino `s[D:D+1]` com `D` par e `D+1 ≤ 105`, recusa máscara de lane no destino ou na fonte e símbolo que não seja `const`/`ud`/`fixed_buffer_word`; dobra constante com `_rep16`, e no caso de `S0` ser a palavra baixa do próprio destino escreve **primeiro** a palavra alta. Emite `bitreplicate16(x)` e `bitreplicate16(x >> 16u)`, com o helper só no cabeçalho quando `uses_bitreplicate`. O check novo (`startup_gfx2hlsl_s_bitreplicate_check.py`, 311 linhas) exercita as formas |

## Pull requests abertos (5)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| `#3169` | compute: accept dword buffer atomics through a byte-window raw UAV | Aceitar atômicos de dword de buffer vindos por UAV raw de janela de bytes (o caso que o código atual recusa por tipo) | Não identificado no catálogo por título; é lowering de compute, área de D-06/D-27 mas sem casamento |
| `#3166` | docs: measure native wave32 compute on app-local WARP 1.0.22 (#3160 stage 1) | Documenta medição de compute wave32 nativo no WARP local | Não |
| `#3164` | shader: lower multi-DWORD GLOBAL loads in direct Compute | Lowering de carga `GLOBAL` multi-dword no compute direto | Não |
| `#3162` | shader: lower typed 2D image atomics in direct Compute DXIL | Atômicos de imagem 2D tipada no DXIL de compute direto | Não |
| `#3161` | compute: carry descriptor spills through v_writelane_b32 / v_readlane_b32 | Levar descritores derramados por `v_writelane_b32`/`v_readlane_b32` | Não |

Os 5 foram lidos por título, base e head (`created_at` 19:17–19:58 no relógio do GitHub); nenhum patch de
PR aberto foi lido.

## Pull requests mesclados (5)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| `#2833` | startup: system commit, job limit and page-pool counters in the budget line | `6057071b` (2026-11-27T19:31:49Z) | O commit é o merge (squash): `commit_state.h`, a linha de budget com 3 grupos novos e o log das 3 primeiras falhas de commit. O PR também diz "cherry picked from ffaa4643/24d42107/3c7b2a56"; o diff confirma os três pedaços (contadores, docs, job limit) |
| `#2836` | startup: name the cause of a slot 70 texture_unavailable refusal in the memory SRV census | `0f3e8248` (19:36:26Z) | Os três motivos e o limite de 64 MiB movido para constante, mais o teste `startup_memory_srv_census_check` que exercita cada ramo e garante que layout acima do limite não toca páginas |
| `#2857` | pixel_aot: lower 10 binary16 cvt_norm forms across VOP1/VOP3/DPP16/DPP8/SDWA | `968ddcdf` (19:50:37Z) | Os helpers, o oráculo e os quatro caminhos; o teste roda os draws `f16_cvt_norm` com `-Od` e fixa o mapeamento quad-intrínseco de DPP16/DPP8. O PR diz "10 formas"; o diff tem 4 sítios de emissão (VOP1/VOP3, SDWA, DPP, e o simples) |
| `#2859` | startup: X2W_LOW_ADDRESSES keeps the title's own memory below 2^40 | `454fc0ed` (19:52:33Z) | `Fixes #2858`: a política de mitigação passada na criação do processo, o switch em `run.ps1` e o teste de que o controle não herda a variável |
| `#3030` | startup: lower s_bitreplicate_b64_b32 in gfx2hlsl | `3c30dc74` (20:04:37Z) | A forma SOP1 0x3b com dobra de constante e o helper `bitreplicate16`; o PR é de terceiro (`R0D4X`) com coautoria de `tedin7` |

## Cruzamento com o catálogo

- **D-28 (conversão UNORM/SNORM do store formatado usava `round()` em vez da regra do Direct3D) —
  consistente, não regressão.** O conserto do catálogo é `f37fe7b8`. `968ddcdf` aplica a mesma regra de
  arredondamento (par mais próximo, empate para o par) nas conversões `cvt_norm`, e o comentário do
  oráculo diz explicitamente por que não usar binário32: duplo arredondamento em `0x3801..0x3804` e perda
  de subnormal sob `-denorm ftz`. Não é o mesmo código (store formatado × conversão de instrução), mas é a
  mesma classe de erro evitada.
- **D-27 (coordenadas `a16` de `image_sample` lidas como dois registradores) — não tocado.** Nenhum
  commit da janela mexe em `image_sample`.
- **D-11/D-25/D-29 (`s_ttracedata`, `seg_inspect_check`, mutação revertida) — não tocados.**
- **Abre defeito novo?** Não verificado. Dois candidatos para o próximo ciclo, ambos por leitura do diff:
  (1) `454fc0ed` falha em silêncio útil — se `InitializeProcThreadAttributeList` ou
  `UpdateProcThreadAttribute` falharem, `startup_for_create` devolve o `si` original e imprime uma nota,
  mas o run continua e pode voltar a bater em `#1758` sem que o teste perceba (o teste cobre o controle,
  não o caminho de falha); (2) `968ddcdf` levanta `ValueError` para operando binário16 não finito, e
  `ValueError` é o sinal genérico de "não suportado" do arquivo (usado em ~20 sítios) — o efeito é recusa
  daquele shader, não tradução errada, mas não foi verificado que a CLI o converte em recusa e não em
  traceback.
- **D-06 (f32 `nlt/nle/…` sob FXC/DXC) — sem relação** com os `cvt_norm` de binary16 desta janela.

## Confiança

- **Verificado por leitura de patch:** os 5 hashes e suas afirmações: `commit_state.h` inteiro,
  `report_budget`, `memory_srv_census.h` + `texture_unavailable_reason`, os dois helpers de
  `pixel_aot.py` e os quatro sítios, `startup_for_create`/`STARTUPINFOEXW` em `launch.py` e `run.ps1`,
  e `op_s_bitreplicate_b64_b32` com o helper `bitreplicate16`.
- **Verificado por API:** os 5 `merge_commit_sha` são idênticos aos 5 commits da faixa
  (`compare 88cddb09...3c30dc74`: `ahead_by` 5, `behind_by` 0, status `ahead`); 5 PRs mesclados na janela
  com `merged_at` 19:31:49Z a 20:04:37Z — contagem de commits e de merges bate.
- **NÃO VERIFICADO:** nenhuma execução do título, nenhuma GPU; a medição de `RssAnon` do `#324` é de
  outro repositório; os 311+43+107 linhas de teste não foram executadas, só lidas; os patches dos 5 PRs
  abertos não foram lidos (só título, base, head e data); o `Fixes #2858` e o `#1758` citados nos commits
  e no código não foram abertos.
