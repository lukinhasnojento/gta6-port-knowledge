# brcly/TuxBox — atualização de 2026-11-26 01:00

## Resumo

Dois commits na `main` (`bc4ba79f` → `ac4f0993`): o `#209`, que na hora anterior ainda estava aberto,
foi mesclado e passa a nomear a causa dos stand-ins de compute; e um commit direto muda os padrões de
lançamento — ProjectScarlett passa a ser sempre offline, o cache de pipeline do vkd3d-proton vai para o
prefixo, o teto do cache NVIDIA sobe de 8 para 32 GiB, as vagas de compilação passam a seguir os núcleos
da CPU e o log do jogo só é gravado com `--log-file`.

## Commits novos (2)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `ac4f0993` | brcly | launch ProjectScarlett offline and settle launcher defaults | Commit direto, um pai (`27b90b05`), 17 arquivos, +269/−97. `host.py` ganha `OFFLINE_EXECUTABLES = ("game_scarlett_master.exe",)` e `requires_offline()`, que compara o nome sem diferenciar caixa (`tests/test_host.py` cobre `Game_Scarlett_Master.exe`); `tuxbox.py:launch_game` levanta `ValueError` quando o alvo exige offline e o backend é `native`, senão monta `Host(..., offline=args.offline or forced)`. O mesmo `host.py` cria `configure_pipeline_cache()`, que grava `VKD3D_SHADER_CACHE_PATH=<prefixo>/shadercache` só se a variável não estiver definida, e troca `__GL_SHADER_DISK_CACHE_SIZE` de `8 << 30` por `NVIDIA_CACHE_BYTES = 32 << 30`. `runtime_shaders.py` sobe `RUNTIME_JOBS_LIMIT` de 8 para 32 e muda o padrão de `min(4, cpus)` para `min(cap, max(2, cpus // 2))`; `scarlett_prep.py` muda o padrão de `max(1, min(8, cpus))` para `max(1, min(PREP_JOBS_LIMIT, cpus))`. `game_log.py` deixa de criar arquivo de log por padrão (`capture_directory()`/`capture_parent()` substituem `log_directory()`), e `mangohud.py`/`pipeline_profile.py` passam a usar `log.capture_parent()`. `docs/ENVIRONMENT.md` é novo (+95 linhas) com as tabelas de opções, variáveis definidas pelo launcher e variáveis de comportamento. Testes: `tests/test_cli.py` (+26), `test_game_log.py` (+16/−27), `test_host.py` (+22/−3), `test_runtime_shader_workers.py` (+5/−2), `test_shader_pack.py` (+5/−3), `test_performance.py` (+2). |
| `27b90b05` | brcly | Merge pull request #209 from s3rius/d3d12/compute-stand-in-refusal | Merge de dois pais (`bc4ba79f`, `6640e457`) com o conteúdo do `#209`: `runtime/d3d12/src/device/compute.rs` (+3) chama `crate::failures::refusal("CS", "driver compile deadline")` nos dois caminhos `PipelineWait::Deferred` (shader traduzido e bytes originais) e `refusal("CS", &format!("driver compile failed: {:#010x}", …))` quando o driver rejeita o pipeline; `tests/failures.rs` (+17) fixa a string `stand-in\tslot63\tCS=len6 CS.used=key CS.refused=driver compile failed: 0x80004005`. |

## Pull requests abertos (1)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 210 | shader-compiler: ignore v_nop in IDXEN stride inference | Draft de s3rius, +43/−3 em dois arquivos: `tools/shader-compiler/src/hlsl/emitter/buffer.rs` (+5/−3) e `tests/shader-compiler/src/tests/hlsl/raw_idxen.rs` (+38). A inferência de stride para cargas com `idxen` passa a pular `v_nop` no cálculo em vez de tratá-lo como uso do índice. O corpo traz o checklist de GPU do repositório com `tuxbox.py check` marcado como passado e a caixa "AMD e NVIDIA" desmarcada. | Não é D-xx. `D-14` (tabela de slots deslocada) e `D-22` (`Failed to remap`) são de root signature/descritores; este PR é a heurística de emissão de leitura de vértice. |

## Pull requests mesclados (1)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| 209 | d3d12: record the cause for translated compute stand-ins | `27b90b05` (conteúdo em `6640e457`; mergedAt 2026-11-26T03:07:29Z) | Exatamente o que o PR prometia e nada além: +20/−0 em dois arquivos. Três chamadas a `failures::refusal` e um teste. A linha de estágios passa a render `CS.refused=driver compile deadline` ou `CS.refused=driver compile failed: 0x<hr>`, que é a causa que faltava na TSV de falhas de pipeline. Não mexe em recusa, aceitação ou tradução. |

## Cruzamento com o catálogo

- Corrige D-xx? Nenhum nesta faixa. `27b90b05` só instrumenta: `D-02` e `D-03` (otimizador do DXC
  estourando o tempo e conversão DXIL→SPIR-V de minutos, ambos com corretor `e9c8fb6`) são as causas que
  a nova nota passa a nomear por shader, mas o defeito em si não muda.
- Toca a área de um defeito já corrigido, sem reabri-lo: `ac4f0993` reescreve o bloco de cache NVIDIA e
  introduz o cache do vkd3d-proton no prefixo — exatamente `D-04` ("cache de pipeline da NVIDIA no lugar
  errado e com teto de tamanho", corrigido em `d10b369`). O caminho continua no prefixo; o que muda é o
  teto (8 → 32 GiB) e o cache do vkd3d, que antes caía na pasta do jogo. Não é regressão do `D-04`
  (o cache segue por prefixo e `VKD3D_SHADER_CACHE_PATH=0` continua respeitado, coberto por teste), mas
  é uma segunda mudança de política de cache em cima da correção — vale reavaliar o texto do `D-04`.
- Abre defeito novo? Não evidente. `ac4f0993` troca o padrão de vagas de compilação em jogo de
  `min(4, núcleos)` para `max(2, núcleos/2)` (32 no máximo) e o de preparação de 8 para o número de
  núcleos. É a mesma alavanca de `D-02`/`D-03` (memória e tempo de compilação), e o commit não traz
  medição de memória com o novo padrão; `docs/ENVIRONMENT.md` documenta o valor, e
  `tests/test_runtime_shader_workers.py` fixa 1→1, 2→2, 3→2, 8→4, 32→16, 128→32. Sem execução de
  `tuxbox.py check` reproduzida aqui, NÃO VERIFICADO o custo de memória em 32 vagas.

## Confiança

Verificado por diff: os dois commits foram lidos em `gh api .../commits/<sha>`, com pais conferidos
(`ac4f0993` sobre `27b90b05`; `27b90b05` de dois pais `bc4ba79f`/`6640e457`). `mergeCommit.oid` do `#209`
confere com o commit documentado. O estado anterior (`bc4ba79f`) é o último commit da execução de 00:00,
que listava o `#209` como aberto — a promoção a mesclado foi confirmada pela API (`merged_at`
2026-11-26T03:07:29Z). PR `#210` lido por `gh pr view`.

Não verificado: `tuxbox.py check`, os desenhos em Lanes e o consumo de memória com 32 vagas de
compilação não foram reproduzidos; o checklist do `#210` é o que o autor declara. Não abri o log de CI.
