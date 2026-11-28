# brcly/TuxBox — atualização de 2026-11-28 16:00

## Resumo

A fila de commits do TuxBox fecha: 18 commits sem documento (de 41 no `compare` a partir de
`99eef7ab`) e nenhum fica pendente. A hora é de memória de vídeo e pipelines — `2ea48307` passa a
liberar pipelines gráficos além dos primeiros 4096 até o primeiro bind, `ce8a7c86` e `b295769f`
seguram texturas amostradas perto do orçamento de VRAM e alocam só os mips que cabem, e
`b244f883` espalha um view heap grande em heaps de CPU menores. `055adb92` faz a espera por label
honrar escrita gravada mas ainda não submetida. Os dois merges da janela (`#401`, `#402`) fecham com
um `Map` de range inválido no selftest e com as receitas Windows 11 atualizadas.

## Commits novos (18)

Janela: `compare(99eef7ab...2af42c94)` = 41 commits; 18 sem documento, todos abaixo.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `055adb92` | MathiewMay | d3d12: hold a label wait for a write that is still to run | O cenário `cross-queue-late` grava no direct queue um `write64`, faz a compute queue esperar esse label e só submete o direct seis segundos depois. Antes, a espera era liberada por time-out de "label que ninguém escreve"; agora o runtime mantém o writer registrado e a compute queue não roda antes da escrita. Também entra `slot_root_srv` e `queue_copy_mappings` no selftest. |
| `2ea48307` | brcly | d3d12: release graphics pipelines past the first 4096 until they are bound | Novo `device/deferred.rs` (+256): passados `TUXBOX_RESIDENT_PIPELINES` (padrão 4096), o pipeline gráfico é compilado — para o cache do driver —, liberado e reconstruído no primeiro bind a partir desse cache. Motivo medido: ProjectScarlett cria mais de 80.000 pipelines e o driver NVIDIA guarda ~150 KB de host por pipeline vivo. `all` desliga. |
| `ce8a7c86` | brcly | d3d12, xmem: spill sampled textures near the video memory budget | `placement/budget.rs` (+224) com um ledger de VRAM: textura amostrada usada quando faltam menos de `TUXBOX_VRAM_RESERVE_MIB` (1536) para o orçamento recebe memória de sistema em vez de vídeo, e o relatório loga para onde a memória foi. Exige `TUXBOX_LAZY_TEXTURES`. |
| `b295769f` | brcly | d3d12: give a sampled placed texture memory only for the mips it holds | `placement/mips.rs` e `overlap.rs`: uma textura 2D amostrada cuja próxima textura começa dentro dela recebe memória só para os mips que terminam antes desse ponto; `TUXBOX_PARTIAL_MIPS=0` volta à cadeia inteira. Texturas escritas no frame forçam `need_whole()`. |
| `b244f883` | KainanP | d3d12: keep a large guest view heap in smaller CPU heaps | Divide o guest heap em vários heaps de CPU quando um driver recusa um único heap grande; `TUXBOX_HOST_PROFILE=cpu-view-heap=1048576` reproduz a recusa de um Intel UHD e o `verify.py` roda o cenário `descriptor-reclaim` de novo nesse modo. |
| `fa7d509c` | KainanP | xcrdapi: build the read test again under MSVC /W4 /WX | Renomeia `size`→`total`, `count`→`pages` e aperta o tipo de `n` para `UINT16` com casts explícitos, matando os avisos que o `/W4 /WX` promovia a erro. |
| `d83d2253` | brcly | Merge branch 'memory/video-budget' | Merge interno de 20 arquivos que junta `2ea48307`, `ce8a7c86`, `b295769f` e `b244f883` na branch do PR #394. |
| `e380da14` | brcly | d3d12: run a list whose structure source got new storage after it was recorded | Cenário e correção: uma lista grava cópia de estrutura BLAS e, antes da submissão, um build é gravado por cima com storage novo; a lista anterior continua copiando a estrutura que gravou e a fila segue. Toca `commands/raytracing.rs`, `device/blas.rs` e o selftest. |
| `0a97a057` | brcly | d3d12: name the draws and dispatches unfinished when the GPU is lost | `TUXBOX_TRACE_GPU_FAULT=1` guarda o nome do pipeline no bind e envolve `draw`/`dispatch` em `diagnostics::gpu_trace::around`, para o relatório de device perdido nomear o que não terminou. Diagnóstico, sem mudança de fluxo. |
| `65785dc7` | brcly | d3d12: scale the package splash to fill the window | `present/splash.rs` e `frame_present.cpp`: o splash do pacote passa a escalar para a janela em vez de desenhar no tamanho fixo. |
| `567bd044` | brcly | tests: keep the shell as the sleep's parent in the zombie guest test | O teste de zumbi usava `sh -c 'sleep …'`, o que fazia o `sleep` virar filho do shell; o comando passa a executar o `sleep` direto, e o teste deixa de medir o processo errado. |
| `3d930b73` | brcly | docs: record that ProjectScarlett renders the opening scene | Atualiza README/FOCUS/PROGRESS/PROJECTSCARLETT/ROADMAP/SHADERS com o estado "renderiza a cena de abertura". |
| `48d5f5c6` | KainanP | lanes: map the later size report within its buffer | O `Map` do buffer de 16 bytes usava o range `{0, 32}` do primeiro relatório; passa a `{0, 8}`, o que a cópia escreve e a checagem lê. É o conteúdo do #401. |
| `926b7642` | brcly | evidence: record the November 28 preparation and the first world render | Evidência de 28/11: preparação e primeiro render de mundo; atualiza FOCUS/PROGRESS/PROJECTSCARLETT/ROADMAP/SHADERS e o JSON de evidência. |
| `40f5c8ed` | KainanP | evidence: refresh the Windows 11 native receipts on both adapters | Regenera as duas receitas Windows 11 (RTX 4060 Laptop e Intel UHD) e `docs/RESULTS.md`/`progress.svg`: 73/76 e 71/76, contra 54/69 e 53/69 do #313. É o conteúdo do #402. |
| `f68c179c` | brcly | docs: remove status and problems that no longer hold | Remove de README/FOCUS/MATRIX_AWAKENS/PROGRESS/PROJECTSCARLETT/SCARLETT_GPU e afins os estados e problemas já resolvidos. |
| `e3f3e153` | brcly | Merge pull request #401 from KainanP/fix/lanes-later-size-map-range | Merge do #401 (o commit de conteúdo é `48d5f5c6`). |
| `2af42c94` | brcly | Merge pull request #402 from KainanP/evidence/windows11-native-refresh | Merge do #402 (o commit de conteúdo é `40f5c8ed`). |

## Pull requests abertos (0)

Nenhum acima do cursor #398: os 7 abertos têm número máximo #393. Os números #394–#402 já foram
mesclados.

## Pull requests mesclados (2)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #401 | lanes: map the later size report within its buffer | `e3f3e153` (2026-11-28T15:29:35Z) | Uma linha trocada no selftest: o `Map` do readback de 16 bytes passa a usar `D3D12_RANGE{0, 8}` em vez de `{0, 32}`. O runtime Windows recusa range que passa do fim do recurso com `E_INVALIDARG`; o vkd3d-proton não checa, e era por isso que o cenário passava sob Proton e falhava em Windows nativo com `map the later size report failed: 0x80070057`. Um arquivo, +3/−1. |
| #402 | evidence: refresh the Windows 11 native receipts on both adapters | `2af42c94` (2026-11-28T15:29:35Z) | Sem código: quatro arquivos. `evidence/windows11-rtx4060laptop-native.json` e `windows11-uhd-a78b-native.json` (+278/−61 cada) com o `verify` de hoje (73/76 RTX 4060 Laptop, 71/76 Intel UHD, contra 54/69 e 53/69 do #313), mais `docs/RESULTS.md` e `progress.svg`. Treze gates que falhavam no #313 passam nos dois adaptadores. |

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum hash por linha do catálogo: os 41 commits e os 2 merges não casam com
  `introduzido`/`corrigido`. Por área, **D-16** ("deadlock entre filas por espera de placement atrás
  de um label") é vizinho exato de `055adb92`/`#398`; é hipótese de área, não correção registrada.
  `#401` não é defeito de emulação — é range inválido no próprio teste.
- **Abre defeito novo?** Não verificado. `2ea48307` troca pipeline residente por reconstrução no
  primeiro bind: se o cache do driver não devolver um pipeline idêntico, o primeiro draw paga a
  compilação ou recebe outro PSO; o diff guarda o necessário para recriar e o PR declara o cache como
  fonte, mas nenhuma medição de acerto/erro do cache está no documento.
- **Pressão de memória:** `ce8a7c86` e `b295769f` mudam onde a textura mora; o critério é o orçamento
  reportado pelo adapter e o `TUXBOX_VRAM_RESERVE_MIB`, não um teste de conteúdo de imagem.

## Confiança

- **Verificado por diff:** os 18 commits (patch por `commits/<sha>`), incluindo os quatro de memória, o
  de label e os dois merges; arquivos e linhas conferidos acima.
- **Verificado por API:** `merged_at`, `merge_commit_sha` e arquivos dos PRs #401 e #402.
- **NÃO VERIFICADO:** execução de `lanes-selftest` ou do título nesta máquina; as contagens 73/76 e
  71/76 são o que os JSONs de evidência dizem, não medição própria; o efeito de
  `TUXBOX_RESIDENT_PIPELINES`/`TUXBOX_VRAM_RESERVE_MIB` em fps não foi medido.
- **Cobertura:** 18 de 18 commits sem documento. O cursor de commit zera (`commit_cursor` e
  `head_at_run` saem do estado); o novo `last_commit` é `2af42c94`.
- **Merges x commits:** 2 merges contra 41 commits; os merges #394, #395–#400 e #371 já tinham sido
  documentados às 15:00, e seus commits continuam no `compare` porque a página não ordena por merge.
