# brcly/TuxBox — atualização de 2026-11-27 16:00

## Resumo

A `main` andou 19 commits desde `90df685a` — os 7 que a execução de 15:00 deixou no teto de 40, mais 12
novos. Os dois PRs mesclados na janela são `#306` (sombras do ProjectScarlett em ray queries nativas) e
`#308` (retentativa de raiz raster com a tabela de strides do heap), e a hora também trouxe o painel de
desempenho (`--overlay`), a aceleração dos checks por concorrência e um aviso novo nos guias: o prólogo
aparece e o jogo cai logo depois.

## Commits novos (19 de 19)

Faixa `90df685ad5dc542802a43183b58aad4b2de4cd56...76b7bafebb9623b6b4bca3c0bb4faf8b704c3e46`,
`ahead_by` 19. Nenhum corte de teto: os 19 estão abaixo.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `580a3bdb5b` | Jerkin42 | xshader: lower ProjectScarlett's sun shadow walks to ray queries | +50 em 2 arquivos. `tools/shader-compiler/src/hlsl/scarlett_rays.rs` ganha `Walk::RaytraceShadows` e faz o casamento por nome e código exato: `"CS_RaytraceShadows_Mixed_22f3acbc_Wrapped"` com `matches_program(&ex.isa, 7728, 0x49e8da1f58f8e753)`. O teste correspondente afirma `identify(&shadows) == None` para um container de bytes zerados |
| `4ecaa4fc2a` | Jerkin42 | xshader: lower ProjectScarlett's laser light shadow walk to a nearest-hit query | +109/−15. `Walk::LaserLightShadow` (`CS_LaserLightShadowRaytracing_c3ab3042_Wrapped`, 3332 bytes, `0xab98828801484e03`); em `driver/pipeline.rs` a substituição da caminhada passa a acontecer **antes** da análise de lanes, e o comentário registra o motivo: a pilha compartilhada e o tráfego entre lanes vão embora com a caminhada |
| `2f341e3c99` | Jerkin42 | tools: replay ProjectScarlett's laser light shadows on the GPU | +211/−5. `tools/check_scarlett_rays.py` deixa de excluir o programa laser; `tools/reference/scarlett/rays_guest.cpp` (+207) declara as duas tabelas que o programa lê (`t47`; `t371`, `t354`, `u48` no espaço 1), com o layout de registro de lâmpada de 512 bytes documentado no código e checagem de distância ao triângulo mais próximo (mais 0,04) ou `FLT_MAX` |
| `00247bb277` | Jerkin42 | tools: replay ProjectScarlett's sun shadow program on the GPU | +268/−22. O `rays_guest.cpp` troca o parâmetro `bool tables` por uma root signature declarada pelo convidado (`const D3D12_ROOT_SIGNATURE_DESC* own`), e o fixture passa a incluir `CS_RaytraceShadows_Mixed` (`641ecfc4956cc4c0`) |
| `156d00199d` | Jerkin42 | xshader: test that the laser light shadow walk needs its own program | +2. Só teste: `identify` do laser com bytes zerados deve devolver `None`, provando que o casamento não é genérico |
| `f9765f385c` | brcly | check: run the translator and D3D12 GPU checks several at a time | +72/−14 em 5 arquivos. Novo `project_tuxbox/jobs.py` com `check_jobs()` (metade das threads, teto 8, `TUXBOX_CHECK_JOBS`); `check_d3d12_regressions.py` e `check_translator_gpu.py` passam a rodar em `ThreadPoolExecutor`, com um `Lock` para a construção da camada Vulkan. O commit mede: RTX 4070 Ti SUPER, tradutor 298 s → 49 s, controles 83 s → 17 s |
| `8a3b6df38f` | brcly | shaders: compile with DXC's new inlining where it is available | +88/−13. `runtime_shaders.py` consulta o `dxc` com `lru_cache` e usa `-fnew-inlining-behavior` quando listado, para todo alvo menos `lib_6_3`; saída recusada é recompilada sem a flag. `docs/SHADERS.md` justifica: promoção de locais antes do inlining, que o DXC chama de experimental |
| `7540bbef37` | brcly | Merge branch 'perf/faster-checks': faster GPU checks | +72/−14, mesmo conteúdo de `f9765f385c` (merge local da branch `perf/faster-checks`) |
| `edf9e0518c` | brcly | Merge branch 'perf/new-inlining': faster shader compilation | +88/−13, mesmo conteúdo de `8a3b6df38f` |
| `6ed257873d` | brcly | Merge branch 'feature/overlay': performance overlay | +1.021/−82 em 22 arquivos. Painel de desempenho: `runtime/d3d12/src/performance.rs` e os módulos `performance/{gpu,host,stats}.rs`, `present/overlay/performance.rs`, `examples/lanes/panel.cpp` para o teste, `TUXBOX_OVERLAY`/`TUXBOX_OVERLAY_POS`/`TUXBOX_FRAME_LOG` documentados em `docs/ENVIRONMENT.md` e `docs/USAGE.md`. Sensores: NVML (`nvml.dll`) e arquivos de sensor do kernel Linux; em AMD/Intel no Windows, carga/temperatura/clock mostram `-` |
| `c7c84de79e` | brcly | check: let the D3D12 control test accept either start order | +2/−1 em `tests/test_d3d12_regressions.py`: ordena os comandos por presença de `--offsets` antes de comparar, porque os casos passaram a rodar em paralelo e qualquer convidado pode começar primeiro |
| `eb2a500062` | Pavel Kirilin | d3d12: retry a raster root with the heap stride table for IDXEN | +43 em `runtime/d3d12/src/commands/compute_bindings.rs` e `src/device/stream.rs`. Quando a tradução de um estágio raster é recusada por falta de stride no acesso indexado ao heap (`IDXEN`), o provider retenta a tradução contra uma raiz com a tabela de strides anexada — a mesma tabela do caminho de compute — e a liga com `SetGraphicsRootShaderResourceView`; `log_once` registra `paged`/`identity`. O retry só vale se o motivo da recusa casar e a tradução vencer |
| `5995ed341c` | brcly | check: run independent Lanes and fixture gates side by side | +192/−55. `project_tuxbox/prefix_pool.py` novo: cada worker pega uma cópia do prefixo Wine (convidados no mesmo prefixo travam ou falham ao criar device); `verify.py` separa gates que compartilham cache ou medem tempo/tela (ficam em fila) dos independentes (rodam juntos), só no backend Proton. `TUXBOX_CHECK_DISPLAY=private` faz os checks de janela amostrarem um Xwayland privado por padrão |
| `3894902a29` | brcly | docs: merge on the checks relevant to a change | +48/−20 em 7 arquivos. Muda a política de merge: o template de PR pede "the checks relevant to this change", `CONTRIBUTING.md` passa a exigir `python tuxbox.py check --no-gpu` para qualquer mudança de código e os suítes de GPU correspondentes; o check completo fica para quem mexe nos runners e para mudanças de alcance incerto |
| `cf47e6874c` | brcly | Merge branch 'perf/faster-lanes': faster checks and per-change merge testing | +240/−75 em 11 arquivos: reúne `5995ed341c` e `3894902a29` (mesmos arquivos e hunks — `prefix_pool.py` +55, `verify.py` +91/−54, template de PR, `CONTRIBUTING.md`), mais `docs/ENVIRONMENT.md` e `tests/test_check.py`. Não inclui `eb2a500062` |
| `07db52f84f` | brcly | Merge pull request #306 from Jerkin42/xshader/scarlett-raytrace-shadows | +616/−18 em 5 arquivos: junta `580a3bdb5b`…`156d00199d` ao `main` (rays_guest +453, `scarlett_rays.rs` +150/−10, `pipeline.rs` +4/−2, `check_scarlett_rays.py` +5/−3). É o merge commit do PR #306 |
| `138f3bbf36` | brcly | docs: bring the guides up to date | +79/−31 em 10 arquivos. `PROGRESS.md` passa a dizer que **o prólogo aparece e o jogo cai**; `PROJECTSCARLETT.md` instrui substituir o `XboxBvhBuilder.dll` do pacote (`cp --remove-destination`), cujos programas de compute o vkd3d-proton não compila (issue #307); `SHADERS.md` registra que o tradutor sozinho traduziu 38.263 de 42.672 pedidos gravados e documenta a seção "ProjectScarlett shadow rays", inclusive que no jogo esses dispatches ainda não fazem nada porque as cenas de ray tracing não estão construídas; `FOCUS.md` atualiza a memória (28,5 GB após ~3,5 min, 22,8 GB em blocos de 64 MB) e registra que uma execução sem out-of-memory de VRAM teve o mesmo fault |
| `acd6359755` | brcly | Merge pull request #308 from s3rius/fix/raster-idxen-stride-retry | +43, mesmo conteúdo de `eb2a500062`. Merge commit do PR #308 |
| `76b7bafebb` | brcly | lanes: test the raster retry with the heap stride table | +170/−20. Novo cenário `raster-heap-stride` em `selftest.cpp`/`raster_fixture.py`: programa de vértice pega o descritor de um buffer estruturado de 32 bytes no slot 100 do heap e lê a posição pelo número do vértice; sem stride no programa, o pipeline só constrói se o provider retentar contra a raiz com a tabela. `verify` exige o bind da tabela paginada e a tradução feita contra a raiz do pipeline |

## Pull requests abertos (0)

Janela de `gh pr list --state open --limit 50`: 6 PRs, números 236–277, **todos abaixo do cursor 284**.
Nenhum PR aberto novo nesta hora. Persiste a pendência estrutural já registrada no estado: o cursor
`last_pr_number` não reexamina PR aberto de número antigo, então esses 6 nunca voltam à fila.

## Pull requests mesclados (2)

Cursor anterior: `2026-11-27T09:37:36Z` (merge do #305).

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| 306 | xshader: lower ProjectScarlett's sun and lamp shadow walks to ray queries | `07db52f84f` (main, 12:36:16Z) | Prometia trocar as caminhadas de sombra por ray queries nativas; entregou quatro programas tratados (`RaytraceShadows` para sol, `LaserLightShadow` para lâmpada e os dois de irradiância que já existiam), mais os replays de GPU e o teste que impede casamento genérico. Ressalva escrita no próprio `docs/SHADERS.md` do merge seguinte (`138f3bbf36`): no jogo os dispatches ainda não fazem nada, porque as cenas de ray tracing não estão construídas |
| 308 | d3d12: retry a raster root with the heap stride table for IDXEN | `acd6359755` (main, 13:14:10Z) | Prometia retentar a raiz raster; entregou o retry em `stream.rs` restrito a recusas por stride ausente, o bind da tabela em `compute_bindings.rs` e o cenário `raster-heap-stride` em `76b7bafebb` |

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhuma entrada de `TuxBox` no catálogo (D-01 a D-29) é fechada nesta faixa.
  `8a3b6df38f` ataca a mesma dor de D-02 (o otimizador do DXC estoura o tempo) e de D-03 (DXIL→SPIR-V
  caro em `cs_6_6`), mas por outra via — `-fnew-inlining-behavior` no DXC, e não o teto de tempo com queda
  para `-O0` que `e9c8fb6` já tinha posto. As duas entradas seguem com correção registrada; o commit novo
  é ganho de velocidade, medido em 1,31× em 800 programas (offline, sem ganho de launch medido, conforme
  `docs/FOCUS.md`).
- **Abre defeito novo?** Um candidato e uma ressalva:
  - `138f3bbf36`: o guia passa a afirmar que o jogo **chega ao prólogo e cai**. É sintoma novo em relação
    ao que `PROGRESS.md` registrava (fault de GPU ao encher a VRAM); a nota de `FOCUS.md` no mesmo commit
    diz que uma execução sem out-of-memory teve o mesmo fault, o que enfraquece a explicação por VRAM.
    Sem stack nem captura no repositório: **NAO VERIFICADO**.
  - `138f3bbf36` também documenta a issue #307 (o `XboxBvhBuilder.dll` do pacote não compila no
    vkd3d-proton), que é limitação conhecida, não regressão desta faixa.
  - `eb2a500062`: o retry raster só dispara quando o motivo da recusa casa com stride ausente; um caso
    vizinho (outro motivo, mesma raiz sem tabela) continua recusando. O cenário `76b7bafebb` cobre o caso
    feliz, não o vizinho — cobertura parcial, registrada aqui.

## Confiança

- **Verificado pelo diff**: os 19 commits, um a um, pelo endpoint `commits/<sha>`. Os três merges locais
  (`7540bbef37`, `edf9e0518c`, `6ed257873d`, `cf47e6874c`) foram conferidos como idênticos em estatística
  e patch aos commits que trazem, o que é o esperado de merge de branch.
- **Verificado por metadado**: os dois merges de PR (número, `mergedAt`, base `main`, `merge_commit_sha`)
  pela API REST de pulls fechados.
- **Não verificado**: se o painel de desempenho funciona em AMD/Intel no Windows (o próprio `USAGE.md`
  diz que mostra `-`), e qualquer medição de jogo real desta faixa. Nada foi executado.
- **Mudança durante a coleta**: o primeiro `compare` desta hora devolveu 16 commits; a reconferência
  pegou 19, com `eb2a500062` inserido no meio da lista (data de commit anterior à posição em que chegou) e
  os dois commits do #308 no topo. A contagem usada é a do segundo `compare`, 19, e o head gravado é
  `76b7bafe`.
- **Contagem de commits**: `ahead_by` de `compare/90df685a...76b7bafe` = 19.
