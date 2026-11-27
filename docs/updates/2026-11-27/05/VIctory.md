# Marvisak/VIctory — atualização de 2026-11-27 05:00

## Resumo

Nenhum commit novo e nenhum PR mesclado na janela: o topo da `main` continua `63bcca43`, o mesmo cursor, e
`search/issues` com `merged:>=2026-11-26T04:15:00Z` devolveu **0**. Todo o delta é de PRs abertos: **380**
acima do cursor 3032 (a listagem de 500 corta em #2890, então a contagem acima do cursor está completa).
Foram documentados os 20 de menor número, 3033..3057, para o cursor avançar de forma monotônica; os 360
restantes ficam para as próximas horas. O lote é dominado por D3D12 na GPU (bind de heaps, read-marks,
footprints, write faults), com quatro PRs de shader/lowering e três de ferramenta/loader.

## Commits novos (0)

`gh api 'repos/Marvisak/VIctory/commits?sha=main&per_page=100'` devolveu `63bcca43c6a00a758510398fa43096ce7e4f7df0`
na posição 0 — o próprio cursor. Nada a documentar por diff de commit.

## Pull requests abertos (20 documentados; 360 acima deles ficam para a próxima hora)

`gh pr list --state open --limit 500` devolveu **500** PRs (mínimo #2890, máximo #3473); **380** acima do
cursor 3032. Documentados os 20 de menor número. Propostas vêm do corpo de cada PR.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 3033 | GPU: load an optional D3D12 Agility SDK | Carrega um Agility SDK app-local opcional para um Windows 10 com Turing ter Shader Model 6.6 sem trocar o SO, já que o `D3D12Core.dll` de caixa para no 6.5. Ordem de busca de `D3D12Core.dll`: `%VICTORY_D3D12_DIR%` (caminho inválido é reportado, não ignorado), `<exe_dir>\D3D12`, `<exe_dir>`, `%LOCALAPPDATA%\Victory\D3D12`; achando um SDK válido, `ID3D12SDKConfiguration1::CreateDeviceFactory` cria a fábrica e `ApplyToGlobalState`. +380/-26, 9 arquivos. | Não. |
| 3034 | GPU: require texture copy footprints to fit a complete row | `CopyTextureRegion` usa `PlacedFootprint` para dimensionar o staging de upload e ler as linhas pedidas; cada cálculo passa a exigir que a linha inteira sem padding caiba em `RowPitch` (em compressão de bloco, blocos horizontais × bytes por bloco). O teste sintético cita que `63bcca4` aceita pitch pequeno para RGBA8 e BC1 nos dois cálculos. +52/-4, 1 arquivo. | Não. |
| 3035 | GPU: write faults read the bytes they save without the guard lock | O issue #400 mede o lock de guarda de `crates/victory-gpu/src/watch.rs` preso ~600 ms por frame de mundo numa RTX 2080, com faltas de escrita de ~450 µs cada, quase tudo esperando o lock: o handler salvava os 64 KiB da página (alocação + `ReadProcessMemory`) **dentro** do lock, e toda outra falta, `take` e escrita de kernel fazia fila atrás da cópia. Passa a ler os bytes fora do lock. +388/-19, 1 arquivo. | Não está no catálogo; é o #400 do próprio repositório. |
| 3038 | Shaders: keep spilled compute root parameters from claiming s16+ system SGPRs | Em compute (`HwStage::Cs`) o hardware dá no máximo 16 SGPRs de usuário (`s0..s15`); parâmetro de root com `user_data >= 16` derrama para memória e `s16+` guarda entradas de sistema (`SV_GroupID`, `GroupInfo`). `spill_loads`, `spilled_param` e `param_word` em `crates/victory-shader/src/resource.rs` já excluem `user_data >= 16` para CS, mas `Tracer::param_of` não — passa a excluir. +195/-0, 1 arquivo. | Não. |
| 3039 | GPU: a lost host list's heaps are not a wanted pair | Corrige #2881: `host_state_lost` marcava `host_heaps` como `[usize::MAX; 2]`, mas uma lista que não liga heaps e lê memória do título quer `[MEMORY_HEAP, SAMPLERS_OURS]` — o mesmo par; depois de um cut (`Wait32/64BitValueX`, 0–21 por frame de mundo) ou de um `external::record` (FidelityFX), `bind_heaps` achava iguais e não ligava nada, e a próxima root signature de indexação direta rodava sem heap — debug layer 1419 e remoção de device na NVIDIA, o que #1778 existia para evitar (#1754). `host_heaps` vira `Option<[usize; 2]>`. +29/-6, 1 arquivo. | Parente de **D-02** (heap de views limitado pelo host → slot aliasado ou fora do fim → device hung) e de **D-03** (descritores nulos), mas o caminho é outro: par de heaps não religado depois de `host_state_lost`. |
| 3041 | GPU: refresh heap textures with untracked pages (FIND-17) | Uma textura pode ocupar memória rastreada e não rastreada; o refresh na submissão usava só as faixas sujas do tracker, então com elas limpas uma escrita tardia na parte não rastreada deixava a textura de heap nos zeros iniciais. Passa a incluir faixas não rastreadas na escolha de subresources a refrescar, e subresource que toca memória não rastreada mantém hash de conteúdo para não repetir a cópia de GPU quando os bytes não mudam. FIND-17, coordenado em #1633. +72/-15, 2 arquivos. | Não. |
| 3043 | Shaders, GPU: clamp immediate static sampler MipLODBias to D3D12 bounds | `StaticSampler::from_words` decodifica o campo `LOD_BIAS` de 14 bits sinalizado em 6.8 fielmente em `[-32,0; +31,996]`, e os dois sítios de conversão para sampler estático (`victory-gpu/src/pipeline/host.rs`, `victory-shader/src/emit/host.rs`) copiavam `mip_lod_bias` direto para `D3D12_STATIC_SAMPLER_DESC::MipLodBias`, enquanto sampler dinâmico em `sampler.rs` limita a `[-16,0; 15,99]`. Passa a limitar nos dois sítios. +89/-2, 3 arquivos. | Não. |
| 3044 | CLI: refuse VICTORY_PREBUILD_KEYS paths inside the repository | `apps/victory/src/prebuild.rs` lê `VICTORY_PREBUILD_KEYS=<file>` e grava chaves de pipeline e hashes de shader derivados do jogo via `std::fs::write` sem passar por `victory_core::ensure_outside_repo`; um caminho relativo ao repositório escrevia saída derivada do jogo na árvore (e truncava arquivo existente), ao contrário de `VICTORY_EMIT_DUMP` em `translate.rs`. Passa a validar. +34/-1, 1 arquivo. | Não. |
| 3046 | Run, GPU: save the pipeline cache on normal title exit | `pipeline::library::save(false)` limita escrita a cada 20 s (`SAVE_EVERY`) e `lifecycle::end()` descarrega no fechar de janela / suspender, mas quando a thread principal do título retorna normalmente (`main.join()` em `run.rs`) o `run` devolve `Ok(code)` sem `save_pipeline_cache()`: pipelines compilados e chaves de early-start do último intervalo se perdem. Passa a chamar no retorno normal. +9/-2, 2 arquivos. | Não. |
| 3047 | Game: name rasterisers from the pack's register images | Fecha a metade rasteriser do #2930: `translate --cache --pipelines` pulava 852 pipelines do pacote como `rasteriser state not named` — 6 dos 18 hashes de rasteriser têm slope bias (e 5 um `DepthBiasClamp`) que `raster_space` não lista, mas as imagens de registrador mantêm: os 7 words de `+0x28` são `PA_CL_CLIP_CNTL`, `PA_SU_SC_MODE_CNTL`, dois não usados, `PA_SU_POLY_OFFSET_CLAMP`, `_FRONT_SCALE` e `_FRONT_OFFSET`. `raster_from_registers` lê cull e o resto dali. +135/-5, 1 arquivo. | Não. |
| 3048 | Game: try what depth-stencil registers do not keep (stacked on #3047) | Metade depth-stencil do #2930, empilhado no #3047 (o primeiro commit é do #3047; o próprio é `1d5afa0`). Depois dos rasterisers, a maior lacuna de estado nomeado é de 139 registros (13 hashes) com `depth-stencil state not named`; o diff aponta três coisas que os registradores não guardam, entre elas b14/b15 (depth-bounds test) ligados em várias descrições com `DEPTH_BOUNDS_ENABLE` limpo. +222/-21, 1 arquivo. | Não. |
| 3049 | Game: recover SGD5 records from unparsed databases alongside intact ones | `crates/victory-game/src/sgd5.rs::records` condiciona a recuperação por marcador a `dbs.is_empty()`: num arquivo com vários bancos SGD5, se um parseia e um irmão falha, `dbs` não está vazio e todo registro recuperável do irmão é omitido em silêncio, embora varrer aquele banco sozinho os recupere. Fatora `scan_databases(file)` devolvendo `(Vec<Database>…)` e passa a recuperar por marcador também quando há banco intacto. +80/-9, 1 arquivo. | Não. |
| 3050 | GPU: our view heap is bound for a list that binds none (stacked on #3039) | `State::wanted` devolvia `[0, SAMPLERS_OURS]` para lista que não liga view heap e não lê memória do título, e `bind_heaps` ligava só o heap de samplers — mas toda root signature gerada indexa os dois heaps de descritor diretamente; D3D12 exige um heap de cada tipo antes de setar a signature (debug layer 1419) e a NVIDIA remove o device. `wanted` passa a pedir `MEMORY_HEAP` sempre que `title[0] == 0`. Empilhado no #3039. +45/-8, 1 arquivo. | Parente de **D-02**/**D-03** pelo sintoma (1419 → remoção de device), por outro caminho. |
| 3051 | GPU: mesh pipelines without a pixel shader get an empty one on WARP | O adaptador de software (WARP) perde o device no primeiro draw de um pipeline mesh sem pixel shader, como passes só de profundidade: um mesh shader trivial que desenha um triângulo mostra isso, direto e por `ExecuteIndirect`; com um pixel shader que não lê nem escreve nada ligado, os mesmos draws rodam e escrevem a mesma profundidade. Em build de diagnóstico com `VICTORY_D3D12_WARP`, o host passa a dar esse pixel shader vazio a tais pipelines; hardware não é afetado. +59/-3, 4 arquivos. | Não. |
| 3052 | GPU: read-mark scans stop at the end of the view slots in use | Depois de cada submissão, `take_reads` varre as marcas de leitura de view de todo slot que um shader pode indexar: `HEAP_VIEWS` são 2^21 slots, 64 Ki words em cada um de dois bitmaps mais o registro de slots vistos, independentemente do que os heaps de view do título têm; `reads_since`, que alimenta os stamps de evicção, faz o mesmo. `descriptor::view_slots()` passa a dar o fim dos slots em uso (só cresce) e as duas varreduras param ali. +55/-4, 4 arquivos. | Não. |
| 3053 | xmem: large graphics blocks get write-watched memory of their own | Para título sem hooks de alocação, `XMemAllocDefault` serve todo bloco do heap do host, memória gráfica inclusive; a memória de recursos comprometidos (alocador 128, de `victory_gpu::resource`) cai assim em memória privada que o lado da GPU não consegue write-watch, e as cópias do host são refrescadas inteiras a cada submissão. A decisão 0007 é copiar só as páginas escritas desde a última cópia, o que as reservas write-watched de `XMemVirtualAlloc` permitem; blocos gráficos de 64 KiB ou mais, alinhados a 64 KiB no máximo, passam a ter memória própria. +67/-1, 1 arquivo. | Não. |
| 3054 | Loader: unload already-mapped title DLLs when a dependency fails | `LoadedImage::unload` (que tira a imagem de `threads::IMAGES` e libera o `HMODULE`) não era chamado quando uma dependência transitiva ou irmã falhava em `map_dll`, `load_title_dll` ou `load_title_dlls`; como `LoadedImage` não tem `Drop` de propósito (imagens mapeadas seguem vivas depois de entregues ao runtime), as já mapeadas ficavam registradas. Passa a descarregar as já mapeadas no caminho de falha. +112/-3, 2 arquivos. | Não. |
| 3055 | GPU: batch and parallelize descriptor heap syncs | **Draft**, parte 3 de um split de 3 pedido pela triagem: a parte 1 é o #3139 (faixas de página em lote e diff de lock único sem alocação), a parte 2 está na fila atrás do #3139 (`shadow::find_existing_batch` + `PreparedBuffer`/`bindless::set_with`), e esta parte 3 paraleliza a sincronização. +1154/-69, 5 arquivos. | Não. |
| 3056 | Shaders/Lower: guard global_atomic_fmin and global_atomic_fmax against NaN operands | Em `crates/victory-shader/src/emit/lower.rs` (`Lower::flat_write`) as duas atômicas baixam para dois atômicos inteiros separados pelo bit de sinal (`SLT(x, 0)`) sem checar NaN: um NaN positivo tem `negative == false`, então `global_atomic_fmax` executa `SMAX(mem, +NaN)` e sobrescreve valor finito ou `+Inf` com `+NaN` (0x7FC0_0000 > 0x7F80_0000 como `i32`); um NaN negativo faz o mesmo do outro lado. Passa a guardar os operandos NaN. +170/-7, 2 arquivos. | Não; mesma família de **D-20** (comparação float com semântica de NaN errada), em atômicos, não em `v_cmp_lg`. |
| 3057 | GPU/Resource: support 2-plane D16_UNORM_S8_UINT formats (118..=120) in GetCopyableFootprints | Em `crates/victory-gpu/src/resource.rs`, `subresource()` (usado por `GetCopyableFootprints`) só reconhece os formatos 2-planos `19..=22` (família `D32_FLOAT_S8X24_UINT`) e `44..=47` (`D24_UNORM_S8_UINT`), omitindo os de console 16 bits de profundidade + 8 de stencil `118..=120` (`DXGI_FORMAT_D16_UNORM_S8_UINT`, `R16_UNORM_X8_TYPELESS`, `X16_TYPELESS_G8_UINT`), que `twin.rs:101` e `victory-formats/src/texture.rs:766` já tratam como 2-planos `[R16, R8]`; o plano 0 cai no `(desc.format, 25…)` errado. +55/-5, 1 arquivo. | Não. |

## Pull requests mesclados (0)

`search/issues?q=repo:Marvisak/VIctory type:pr is:merged merged:>=2026-11-26T04:15:00Z` devolveu
`total_count` 0. Nenhum merge na janela — o cursor `last_run` de 2026-11-26T04:15:00Z é o mais antigo dos
treze repositórios e faz a janela cobrir 25 horas sem nenhum merge.

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum. Sem commit nem merge na janela, não há hash novo para comparar com os
  `corrigido` de `VIctory` (`edf64e2`, `3827767`, `25a3e95`, `27b530e`, `b1c8306`, `35b054e`, `6db02ea`,
  `005ad49`, `257d8bb`, `e158d95`, `02fcf72`, `c88b7ac`, `f0894c2`, `36d6a79`, `3e39eac`, `7102a49`,
  `8e79c80`, `077ec8b`, `51f1627`, `bf84134`, `63df363`, `2e5001c`, `a2f5f43`, `1106f43`, `c963b71`,
  `79fb722`). Dos PRs abertos, dois merecem nota de proximidade e **não** são o defeito do catálogo:
  - **#3039** e **#3050** produzem o mesmo sintoma de **D-02** (debug layer 1419 e remoção de device na
    NVIDIA por heap faltando), mas D-02 é o teto de 1.000.000 da NVIDIA no heap de views com alias ou
    índice fora do fim; aqui o par de heaps deixa de ser religado depois de `host_state_lost` (#3039) e a
    lista sem view heap só recebe o heap de samplers (#3050).
  - **#3056** é a mesma classe de **D-20** (`v_cmp_lg` baixado como `fcmp one`, verdadeiro para NaN), mas em
    `global_atomic_fmin/fmax` com o truque de dois atômicos inteiros, não no lowering de comparação.
- **Abre defeito novo?** Nenhum demonstrado por diff. Ponto registrado: **#3053** passa a dar memória
  write-watched própria a blocos gráficos de 64 KiB ou mais e depende de "copiar só as páginas escritas
  desde a última cópia" (decisão 0007). Se alguma escrita de GPU não passar por `XMemVirtualAlloc` nem for
  rastreada pelo write watch, a cópia parcial perde a página — é o mesmo tipo de risco do #3041 no sentido
  inverso, e nenhum dos dois traz teste de escrita de GPU fora do caminho rastreado.

## Confiança

Verificado: `commits?sha=main&per_page=100` com `63bcca43c6a00a758510398fa43096ce7e4f7df0` na posição 0 —
nenhum commit novo; `search/issues` com `merged:>=2026-11-26T04:15:00Z` devolvendo 0; `gh pr list --state
open --limit 500` com 500 linhas, mínimo #2890, máximo #3473 e 380 acima do cursor 3032 (como a lista vem
ordenada por criação decrescente e o mínimo é anterior ao cursor, todos os PRs acima de 3032 estão dentro
das 500); os 20 documentados com `pulls/<n>` (base, `+/-`, arquivos, `draft`).

Não verificado: se o repositório tem mais de 500 PRs abertos (a janela de 500 cortou em #2890, e a contagem
acima do cursor não é afetada, mas o total real não foi medido); a exatidão das medições citadas nos corpos
(RTX 2080, 600 ms de lock, `852` pipelines pulados, contagem de 604 de 1.898 marcas); se o #3055 deixou de
ser draft ou se o #3139 citado como parte 1 existe e está aberto; e se os empilhamentos (#3048 sobre #3047,
#3050 sobre #3039) continuam válidos. Nenhum teste foi executado aqui; o repositório monitorado é somente
leitura.
