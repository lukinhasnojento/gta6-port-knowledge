# ivall/xcl — atualização de 2026-11-27 20:00

## Resumo

Dois commits entre o cursor `f9a0cb80` e o head `6089122a`: um de conteúdo, com data de autor de
2026-11-24, e o merge do #880. A correção é estreita e de contabilidade: `xcl shaders --recomp` e o dump
por shader passam a tirar o tamanho de wave do cabeçalho FSSM, como os pipelines já faziam, em vez de
adivinhar pelo código. Um PR mesclado, três abertos acima de #3038.

## Commits novos (2)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `6576d75b` | throny | Shaders: xcl shaders statistics and the dump take the wave size from the header | `crates/xcl/src/shaders.rs` +18/-3: `RecompAgg::add` deixa de calcular `wave64` internamente por `recomp::detect_wave64(code)` e passa a receber o valor do chamador, que tem o FSSM e usa `recomp::fssm_wave64(f, code)`; em `Agg` o mesmo valor vai para o agregador. `crates/xcl/src/gpu/shader.rs` +1/-1: o dump por programa (`XCL_DUMP_SHADERS`) troca `detect_wave64` por `fssm_wave64`. Novo teste `recompiler_statistics_count_wave32_by_the_header_flag` monta um FSSM com flags `0x20` e espera `(1 programa, 1 wave32)`, e com flags `0` espera `(1, 0)`. O defeito real: um programa wave32 **sem escrita de EXEC** era contado e traduzido como wave64 pelos dois caminhos de relatório, enquanto os pipelines já o traduziam como wave32 pela flag do cabeçalho |
| `6089122a` | throny | Merge pull request #880 from throny/shaders-stats-wave-size-from-header | merge do #880, pais `f9a0cb80` (cursor) + `6576d75b`. Diff contra o primeiro pai: os mesmos 2 arquivos de `6576d75b` (+19/-4), apesar de a lista de arquivos do PR ter 6 entradas |

## Pull requests abertos (3)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #3042 | Recompiler: add opt-in XCL_WAVE_LOCAL=1 compute lowering | base `main`, empilhado em #3040: lowering wave-local de compute wave64 elegível (`NoCrossHalf`, `SingleWave`) nos três backends (HLSL, DXIL direto, per-lane) em hardware de 32 lanes, desligado por padrão. Emite `[WaveSize(32)]` sem troca entre metades de groupshared, ignora `x64_masks::by_pc` quando elegível e põe `wave_local.rs` na lista de fontes que entram nas chaves de cache. Também deixa de tocar o operando `vdata + 1` em atômicos de 32 bits que não são compare-swap/mskor (+1268/-36, 12 arquivos) | Não. Frente nova (issue #3011), sem entrada no catálogo |
| #3040 | Recompiler: add exact wave-local eligibility analysis for wave64 compute | base `main`: análise estática de elegibilidade (`WaveLocal::{NoCrossHalf, SingleWave}`, `Reject`) com `analyze`, `partial_blocks`, `solve_cfg`, `check_exec_in_branch` e `check_masks_and_scalars`, 18 testes de unidade e um censo sobre pacote retail. Proveniência declarada: adaptado de `TaynanGT/Vice-Tay 1f69d67`, herdando de `Marvisak/VIctory 63bcca4` e `fxceless/victory-fork 9a9091d`, com validação de máscara escalar de `Kemerd/VIctory-NVIDIA-RT 0fae809e` (+859/-0, 3 arquivos, `THIRD_PARTY.md` incluído) | Não |
| #3039 | D3D12.X: XCL_TEXTURE_SYSMEM puts streamed texture levels in system memory | base **`nvidia-dev`**: com `XCL_TEXTURE_LEVEL_PAGES=1`, as páginas dos níveis de textura streamada vêm de heaps de sistema (`MEMORY_POOL_L0`, write-back) em vez de memória de vídeo; render targets e as demais texturas continuam em VRAM. Novo teste `a_system_memory_page_is_told_apart_and_cools_in_its_own_pool` (+362/-22, 3 arquivos) | Não. É gestão de memória; o catálogo XCL não tem entrada para pressão de VRAM |

## Pull requests mesclados (1)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #880 | Shaders: xcl shaders statistics and the dump take the wave size from the header | `6089122a`, 2026-11-27T16:16:53Z, base `main` | Só o commit de topo. O PR foi aberto em 2026-11-24 e é declarado empilhado em #573, já mesclado; por isso a lista do PR tem 6 arquivos (`recomp/mod.rs`, `pso.rs`, `gpu/xbox.rs`, `gpu_compute_e2e.rs` além dos dois do commit) e o merge contra `f9a0cb80` tem 2. O que entrou de novo em `main` é exatamente `6576d75b`: a troca de `detect_wave64` por `fssm_wave64` nos dois relatórios e o teste |

## Cruzamento com o catálogo

- **D-22 (`xcl shaders`: `--export-hlsl` pulava registros não-compute em silêncio e flags sem valor eram
  aceitas):** `6576d75b` mexe no mesmo subcomando, mas não nesse defeito. D-22 é validação de CLI e
  exportação; aqui é de onde sai o tamanho de wave usado na contagem e na tradução. **Não corrige D-22.**
- **D-16/D-17 (máscara wave64 obsoleta reportada só na metade baixa; máscaras inline/NULL recusadas no
  bridge):** área de wave size, mecanismo diferente. Não tocados.
- **D-20/D-21 (ExecuteIndirect antes da barreira; root signature errada em compute não traduzido):** a
  triagem das 19:00 apontou a pilha #3033-#3035 nessa área; esses PRs não estão entre os três novos
  abertos (ficam abaixo do cursor de abertos #3038) e nenhum commit da janela os toca.
- **Proveniência cruzada, fato do repositório:** #3040 e #3042 declaram-se adaptados de
  `Marvisak/VIctory 63bcca4` — que é **exatamente o `last_commit` do `Marvisak/VIctory` no estado desta
  rotina** — e de `fxceless/victory-fork 9a9091d`. O que o `xcl` está portando veio, em parte, do topo
  atual de um dos repositórios monitorados.
- **Estrutural:** #3039 aponta para `nvidia-dev`, como #3036/#3037 antes dele. Merge nessa base não move
  o cursor de commits nem o de merges deste repositório (o `last_run` aqui é o de `main`). Pendência
  registrada nas horas anteriores e ainda aberta.

## Confiança

Verificado por mim, pelo diff:

- os dois hashes, autores, arquivos e contagens de linha vieram da API; `6089122a` tem pais
  `f9a0cb80` + `6576d75b` e o diff contra o primeiro pai são os 2 arquivos de `6576d75b` — a afirmação de
  "só o commit de topo entrou" é do diff, não do texto do PR;
- li o hunk inteiro de `shaders.rs` (assinatura nova de `add`, remoção do cálculo interno, chamada em
  `Agg`) e o de `gpu/shader.rs`, e o teste novo;
- o teste fixa os dois lados (`flags 0x20` → 1 wave32; `flags 0` → 0), então o parâmetro do cabeçalho é
  o que decide; não verifiquei o conteúdo de `recomp::fssm_wave64` nem se ele lê o mesmo dword que o
  teste escreve (`0xD4..0xD8`), só que o teste passa pelo caminho de produção;
- bases dos três PRs abertos conferidas em `base.ref`.

NÃO VERIFICADO:

- os números do PR: 1.130 testes passando, 26.698 programas no pacote retail, wave32 de 2.608 para 2.662,
  e o controle negativo que falha ao voltar `detect_wave64`. Não rodei `cargo test` nem o pacote;
- a data de autor de `6576d75b` é 2026-11-24T07:18:01Z, três dias antes da janela: o commit entrou agora
  pelo merge, e não há como saber pelo diff por que ficou parado — registrado como fato, sem explicação;
- os diffs de #3039, #3040 e #3042: li título, corpo, base, arquivos e contagens; não abri os patches.
