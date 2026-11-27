# ivall/xcl — atualização de 2026-11-27 05:00

## Resumo

Seis commits novos na `main` acima do cursor `8b957bec`: o antigo `8b957bec` deixou de ser o topo
(`c2ee10e4`). Dos seis, três são a mesma coisa — o PR **#2924** entrou por fusão de verdade (`c2ee10e4`,
dois pais) trazendo o teste de detile de todo entry de TTPF (`34161433`) e a leitura da contagem de mips
nos dois campos da tabela (`fd0cc07d`). Os outros três: **#2844** (`fa671a2f`) põe a janela coerente de
4 GiB num heap de driver no endereço do título na NVIDIA, com `uma_driver.rs` novo (+305), e `b84e0378`
manda as leituras de textura do título fora da janela pelo caminho com `ReadProcessMemory` em vez de ler
no lugar. PRs mesclados na janela: 3 (#2924 e #2844 na `main`, #2927 na `nvidia-dev`). Na fila de abertos,
**180** acima do cursor 2685; documentados os 20 de menor número (2686..2708), os 160 restantes ficam para
a próxima hora.

## Commits novos (6)

`gh api 'repos/ivall/xcl/commits?sha=main&per_page=100'`: o cursor antigo `8b957bec` está na posição 6;
`c2ee10e4` é o topo. Os seis abaixo.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `c2ee10e4` | thehollowedone (merge #2924) | Merge pull request #2924 from thehollowedone/tests/ttpf-generic-707 | Fusão de dois pais, +290/-77 em 2 arquivos: `crates/xcl/src/gpu/texture.rs` (+287/-74) e `README.md` (+3/-3). É o conteúdo de `34161433` mais `fd0cc07d` abaixo. |
| `fa671a2f` | Kalmarv (merge #2844) | Merge pull request #2844 from Kalmarv/pr/uma-driver-window | Fusão de dois pais, +575/-59 em 7 arquivos. Arquivo novo `crates/xcl/src/hle/d3d12x/uma_driver.rs` (+305/-0); `title_textures.rs` (+200/-36), `real.rs` (+25/-7), `uma.rs` (+14/-0), `host.rs` (+14/-12), `d3d12x/mod.rs` (+14/-4), `tests/e2e.rs` (+3/-0). Conteúdo descrito em `f2248f06`. |
| `34161433` | — | Tests: detile every TTPF archive entry and check mip chains | +290/-78 em `texture.rs` (+287/-75) e `README.md` (+3/-3): o teste de TTPF passa a ler os spans empacotados exatos com suas tags de codec, verificar o stream inteiro e comparar os tiers duplicados de cubemap contra a cadeia completa — antes parava em 909 de 912 entries porque o leitor pulava blocos taggeados, o último entry e tiers de mip de cubemap. Mantém os controles de conteúdo de máscara alpha, caudas uniformes e volumes pequenos. |
| `fd0cc07d` | — | Tests: read the TTPF texture table's mip count from both of its fields (#1121) | +2/-1 em `texture.rs`: `retail_ttpf_textures_detile_with_their_mip_chains` tirava os níveis só de `dword 0` bits 28-31; passa a somar `dword 1` bits 0-2 (abaixo do formato DXGI), porque os níveis extras só cabem no bloco de mip tail e o tamanho batia no arquivo contra o qual o teste foi escrito. Medição citada: 351.970 de 352.461 texturas casam com `(a >> 28) + (b & 7)` contra 294.726 só com `a >> 28`. Cherry-pick de `830bf789`. |
| `f2248f06` | — | D3D12.X: coherent UMA windows in a driver heap at the title's address on NVIDIA | +381/-25, 7 arquivos (mesmo conjunto do `fa671a2f`, com `title_textures.rs` +6/-2 em vez de +200/-36): em `open_uma` as páginas escritas da janela são copiadas, a janela é liberada, todo trecho livre de 1 GiB ou mais é reservado por um instante para que o mapeamento de CPU do novo heap CUSTOM L0 só caia no buraco, e as páginas voltam; o título mantém os ponteiros. Se o mapeamento cair em outro lugar ou um passo falhar, a janela é refeita no endereço e importada como antes; se o snapshot falhar, a janela não é liberada. Buffers nela perdem `ALLOW_CROSS_ADAPTER`; o heap é criado com `D3D12_HEAP_FLAG_ALLOW_WRITE_WATCH`. Ligado por padrão em adaptador NVIDIA fora do Wine; `XCL_UMA_DRIVER=0` desliga. |
| `b84e0378` | — | D3D12.X: title texture reads outside the coherent window use the safe copy | +196/-36 em `title_textures.rs`: `with_title_bytes` substitui `title_bytes`; faixa dentro da janela coerente comprometida continua lida no lugar (`VirtualFree` da XCL nunca descompromete aquela janela), o resto passa pela cópia de `read_committed` num buffer por thread descartado acima de 64 MiB. `read_committed_into` passa a escrever todo byte (copiado, ou zero para página ilegível), então o buffer reusado dispensa `memset`. Dois testes novos: `title_bytes_survive_concurrent_decommit` e `title_bytes_leave_guard_pages_alone`. |

## Pull requests abertos (20 documentados; 160 acima deles ficam para a próxima hora)

`gh pr list --state open --limit 500` devolveu **500** PRs abertos (mínimo #2185, máximo #2957); **180**
têm número acima do cursor 2685. Foram documentados os 20 de menor número, como nas horas anteriores, para
que o cursor avance de forma monotônica: 2686..2708. Propostas vêm do corpo de cada PR; o `+/-` e a
contagem de arquivos vêm de `pulls/<n>`.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 2686 | Ignore rotated runtime reports | `.gitignore` passa a ignorar o basename exato `report.prev.json` em qualquer profundidade, casando com a regra já existente de `report.json`, para o relatório rotacionado não aparecer como não rastreado. +1/-0, 1 arquivo. | Não. |
| 2687 | Document shared cache history for used-only precompile | README: `--used-only` lê `used-pipelines.txt` do cache DXIL compartilhado entre execuções, não o histórico de um título só; documenta o requisito de cache compartilhado e o que acontece com arquivo ausente ou vazio. +5/-1, 1 arquivo. | Não. |
| 2691 | D3D12.X: forget a command list before the runtime frees it | `real::cl_release` apagava o que a XCL guarda pelo endereço da lista **depois** do `Release` final do runtime; uma lista criada em outra thread no intervalo podia pegar o endereço e perder bind state, allocator, último submit e marcas DRED. Passa a esquecer a lista antes, como `res_release` e #2422 já fazem. Corrige #2690. +70/-5, 1 arquivo. | Não está no catálogo; é o #2690, aberto por outro autor. |
| 2692 | D3D12.X: check writability in note_signaled and remove per-frame allocations in WaitFrameEventX | `note_signaled` troca `readable(out, 4)` por `writable_with_uma(...)`, mantendo o atalho `Uma::in_committed_window` antes de `resident_and_writable`/`VirtualQuery`; no caminho quente `WaitFrameEventX`/`mark_flipped` remove alocações por frame, `Arc::clone` e deslocamento O(N) do ring buffer. +284/-20, 1 arquivo (`frame.rs`). | Não. |
| 2693 | D3D12.X: keep the FLAGS subobject of a console compute stream (#2607) | `stream_pipeline` no ramo de CS de console projetava o `PIPELINE_STATE_STREAM` num `D3D12_COMPUTE_PIPELINE_STATE_DESC` zerado mapeando só root signature (tipo 0), shader (6) e node mask (18): um subobjeto FLAGS (tipo 20) era descartado, então o PSO criado por stream chegava com `Flags` 0 enquanto o mesmo título pelo desc legado mantinha as flags — e `Flags` entra na chave do cache de PSO (`compute_blob_key` / `driver_cache::path_for`), permitindo alias entre PSO de stream e legado. +44/-0, 1 arquivo. | Não; corrige #2607, parte de #2508. |
| 2694 | VFS: route MAXIMUM_ALLOWED opens through the overlay | Inclui `MAXIMUM_ALLOWED` na máscara de acesso gravável do classificador de VFS: um `CreateFileW` só com esse bit ia para o arquivo original do jogo, onde o Windows pode conceder escrita, furando o overlay copy-on-write. Fixture abre arquivo existente só com esse bit e confere caminho do overlay e bytes originais intactos. Corrige #2682. +16/-0, 1 arquivo. | Não; ver #2708, que propõe o mesmo conserto. |
| 2695 | D3D12.X: XCL_GPU_TEXTURE_TOUCH logs CPU accesses to textures the GPU owns | `XCL_GPU_TEXTURE_TOUCH=1` registra quais texturas que a GPU possui (render targets, depth, UAVs, que `vram::place` põe em memória de vídeo) a CPU do título lê ou escreve no endereço de console — na PC nada as copia de/para a memória do título, então a leitura devolve bytes velhos. Desligado por padrão, sem mudança de comportamento. Parte de #2663. +358/-1, 5 arquivos. | Não. |
| 2696 | Opus: reuse multistream scratch buffers, validate sub-packets atomically, and bulk-append PCM16 | `hle/opus.rs`: `split_self_delimited_into` com buffer reusado, `split_multistream_into` que exige `2 * stream_count - 1` bytes e valida todos os sub-pacotes antes de decodificar, cópia em bloco de PCM16 LE e `decode_buffer` pré-alocado. +421/-57, 1 arquivo. | Não. |
| 2697 | D3D12.X: a memory line by owner, against DXGI's budget | Linha de memória por dono sob `XCL_ACTIVITY`: commit do processo e uso local/não-local da DXGI contra o orçamento, partido nas partes da XCL e no que a XCL não conta como sua. Renomeia após review (@throny em #2688) as linhas `target pages` → `placed texture pages` e `target pools free` → `placed texture pools free`. Parte de #2688. +280/-0, 6 arquivos. | Não. |
| 2698 | xcl run: the end of a run ends the process past XCL's own exit hook | O watchdog de 15 s do `end_title` terminava o processo por `TerminateProcess`, isto é, pelo hook `NtTerminateProcess` da própria XCL (`eh.rs`), que formata stack e log antes — e aloca; em metade das execuções numa máquina Linux o watchdog esperava ali e o processo não terminava. Passa a chamar o `NtTerminateProcess` real. +53/-1, 2 arquivos. | Não. |
| 2699 | README: document xcl shaders --direct --seed and --validate | Uma linha de exemplo para `xcl shaders --direct --seed 7 --validate` (amostra determinística com validação DXIL), que faltava ao lado de `--sample/--hash/--dump/--compare`. Draft, empilhado em #2657. +3/-0, 1 arquivo. | Não. |
| 2700 | Tests: tell a leaked XApu buffer from another allocation at its address | `xapu_connect_returns_oom_and_cleans_up_on_failure` era flaky em execução paralela: perguntava por `VirtualQuery` se o endereço estava livre e outra thread podia alocar ali no intervalo. O alocador do teste reserva `0x53000` bytes (tamanho que nada mais no processo aloca) e compromete no começo; só falha se uma alocação exatamente desse tamanho ainda começar ali. Só teste. +28/-6, 1 arquivo. | Não. |
| 2701 | D3D12.X: drop a heap's, queue's and allocator's books before last Release | `heap_release`, `mem_heap_release`, `q_release` e `alloc_release` chamavam o `Release` do driver primeiro e só apagavam o que a XCL guarda por endereço quando ele voltava 0 — outro objeto podia já estar no mesmo endereço com suas próprias entradas, e a limpeza removia as do novo; num `ID3D12Heap` isso libera o bloco de memória unificada (`HEAP_UMA`) em uso. Passa a esquecer antes do `Release` final (mesmo padrão de `cl_release`/`res_release`). +203/-49, 2 arquivos. | Não. |
| 2702 | BCPack: replace bit-packing loops with compile-time LUTs and precomputed masks in victory | `hle/bcpack/victory.rs`: LUTs de compile-time `SPREAD4`/`SPREAD8`/`NIBBLES` com `interleave`/`wide_interleave`, máscaras de largura pré-computadas no lugar dos deslocamentos dinâmicos, âncoras de slice em bitmask `u16` e helpers de empacotamento em array fixo no caminho de descompressão BCPack da CPU. +316/-59, 1 arquivo. | Não. |
| 2703 | D3D12.X: bypass title hooks for own_work ExecuteCommandLists | Submissões internas da XCL dentro de `title_textures::own_work()` vão direto ao `ExecuteCommandLists` da PC (mais `stop_if_device_removed()`), pulando os hooks só-de-título (`title_textures::refresh`, tradução de lista de console/split, `present::note_execute` → `late_release::drain_known` → `vram::recycle_ready`), para não reentrar em mutex internos (`vram::PAGES` em `before_execute`/`flush_zero_tiles`, `sparse::STATE`) e travar. +76/-1, 1 arquivo. | Não. |
| 2704 | Shaders: include HEAP_FLAT in offline precompile runtime_windows | Extrai `runtime_windows_flags(windows, heap_windows, heap_flat)` para que `--precompile`/`--export-hlsl`/`--measure-pso` incluam `recomp::HEAP_FLAT` quando `windows > 1 && heap_windows && heap_flat`, casando com `recomp::pso::mem_windows()` do runtime; mascara `HEAP_WINDOWS\|HEAP_FLAT` na impressão da contagem (antes podia imprimir `536870916 memory windows`) e acrescenta `(flat)` só no `--precompile`. +25/-8, 1 arquivo. | Não. |
| 2705 | xcl run: preparation workers count the commit left in MiB, not bytes | `prepare_workers` passava `host::system_commit()` (bytes) a `workers_for(cores, free_mb)`, que divide por 3072 como se fosse MiB — o limite de 3 GB por worker nunca valia e a contagem era sempre núcleos lógicos menos quatro (até 32): 16 threads com 6 GiB de commit davam 12 workers em vez de 2. Novo `workers_for_commit` converte para MiB. Introduzido em `b09acbc` (#2624). +18/-1, 1 arquivo. | Não está no catálogo; é defeito de upstream ainda aberto, candidato a entrada nova. |
| 2706 | D3D12.X: preserve the NVIDIA world-streaming integration | Draft: preserva a integração local histórica que aceitou perfil offline de Série X numa RTX 4060 Laptop e sustentou streaming de mundo sem reset no mais longo run on-demand; parte do ancestral `cbc296a` e carrega os commits anteriores necessários para reconstruir o experimento. Smokes emparelhados contra a `main` atual seguem abertos segundo o próprio corpo. +7379/-567, 35 arquivos, draft. | Não. |
| 2707 | D3D12.X: detach retired segments and reclaim idle aside allocators | `alloc_pool.rs`: `retire_recording(list)` desliga um segmento split aposentado ou lista primária substituída de `Backing.users` preservando `marks`/`last_submit`; `Backing.already_reset` devolve backings ociosos postos de lado em `title_reset`; solta `STATE` antes de `reset_pc_allocator`/`CreateCommandAllocator`. +942/-61, 5 arquivos. | Não. |
| 2708 | VFS: treat a MAXIMUM_ALLOWED open as a write open (#2682) | Mesmo alvo do #2694: `WRITE_ACCESS` sem `MAXIMUM_ALLOWED` (0x0200_0000) fazia `is_write` cair no ramo de leitura, `open_decision_for` devolver o caminho original e o open real acontecer com todos os direitos da ACL — em pasta gravável, `WriteFile` altera a pasta do jogo que o README promete não modificar. +21/-0, 1 arquivo. | Não; duplica o #2694 para o mesmo #2682 (dois PRs concorrentes). |

## Pull requests mesclados (3)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #2924 | Tests: detile every TTPF archive entry and check mip chains | `c2ee10e4` (2 pais, +290/-77, 2 arquivos, base `main`) | O que promete: `texture.rs` reescrito em 287 linhas para ler spans empacotados por tag de codec e conferir o stream inteiro, mais a soma de `dword 1` bits 0-2 na contagem de mips (`fd0cc07d`). Os dois commits citados no corpo estão na `main` (`34161433`, `fd0cc07d`). Saída conferida pelo autor em cinco amostras de arquivo licenciado (912 a 10.013 entries). |
| #2844 | D3D12.X: UMA windows in a driver heap on NVIDIA, 4.9x FPS (0.62 -> 3.03) | `fa671a2f` (2 pais, +575/-59, 7 arquivos, base `main`) | O que promete e o que está no diff batem: `uma_driver.rs` novo com o remapeamento da janela para heap CUSTOM L0, reserva temporária dos trechos livres ≥1 GiB, snapshot/restore das páginas escritas, `ALLOW_WRITE_WATCH` e fallback para a importação; `title_textures.rs` +200/-36 é a parte que lê os bytes do título região a região sob Wine. Números do corpo (172 ms → 1.427 ms de tempo de GPU numa replay de frame capturado; 0,62 → 3,03 FPS) não foram reproduzidos aqui. |
| #2927 | D3D12.X: a texture level that lost its texels gives its pages back | `a5e2d851` (2 pais, +134/-5, 2 arquivos, base **`nvidia-dev`**) | Com `XCL_TEXTURE_LEVEL_PAGES=1`, um nível que perdia os texels (título de streaming tirando a memória) voltava a ser enviado todo zero e as páginas ficavam com os zeros. O diff em `vram.rs` (+103/-1) e `title_textures.rs` (+31/-4) devolve as páginas de um grupo de níveis que está todo zero antes do upload; tile sem memória lê zero em tiled resources tier 2. **Não está na `main`**: o merge `a5e2d851` não aparece em `commits?sha=main`. |

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum dos seis commits nem dos três merges casa com os 24 defeitos do `XCL`. Nenhum
  hash novo é um dos `corrigido` do CSV (`f9e5ee9`, `70f8dc4`, `69a892c`, `188ee17`, `ac34cb6`, `50228bb`,
  `ecb7547`, `bfb8133`, `66879ef`, `9dcf2f6`, `fe78c81`, `a23b6ff`, `13ac196`, `15fff3b`, `27d28ce`,
  `f3242b2`, `4e3515e`, `6c89285`, `4143a09`, `9e27370`, `425b869`). Os candidatos próximos não casam:
  **D-01** (janela UMA escolhida pelo dword alto da base) é aritmética de endereço em `uma.rs`, e o
  +#2844 só acrescenta 14 linhas ali (`in_committed_window`/constantes do heap de driver); **D-20/D-21**
  são `ExecuteIndirect`, e #2924/#2927 tratam de TTPF e de páginas de nível; **D-06/D-07/D-08** são do
  tradutor direto, sem arquivo desses caminhos nos diffs.
- **Abre defeito novo?** Nenhum demonstrado por diff. Três pontos ficam registrados:
  - `f2248f06`/`#2844` muda **quem possui** a janela coerente de 4 GiB: ela passa a ser um heap CUSTOM L0
    do driver no endereço do título, com reserva de todos os trechos livres ≥1 GiB por um instante. Se
    outra alocação do processo pedir memória nesse intervalo, ou se o heap não puder ser refeito no
    endereço, o corpo diz que a XCL loga e **encerra o processo** em vez de rodar com janela sem lastro.
    Também depende de `GetWriteWatch` conseguir ler o mapeamento (validado numa RTX 5070 Ti, driver 610.74).
  - `b84e0378` põe a leitura de bytes do título fora da janela num buffer por thread que cresce até ser
    descartado acima de 64 MiB; um buffer reusado por leitura maior que a anterior fica residente na thread.
    O caminho `read_committed_into` agora **escreve todo byte** (zero para página ilegível) — é o que
    dispensa `memset`, e também o que apaga qualquer expectativa de "buraco preservado".
  - `fd0cc07d` e `34161433` mudam o que o **teste** de TTPF aceita: 491 texturas do pacote 1.0.2.1 seguem
    fora do modelo de tamanho (445 cubos com mip e a última textura de cada arquivo). Se o teste passar a
    exigir igualdade estrita, essas 491 são o lugar onde ele quebra primeiro — não verifiquei a asserção.

## Confiança

Verificado: `commits?sha=main&per_page=100` com `8b957bec` na posição 6 e `c2ee10e4` no topo — os seis
commits acima do cursor, cada um lido em `commits/<sha>` (contagem de pais 2/2/1/1/1/1, `+/-` e arquivos
como na tabela; `f2248f06` e `fa671a2f` compartilham o mesmo conjunto de arquivos, com `title_textures.rs`
em +6/-2 e +200/-36 respectivamente); `search/issues?q=repo:xcl-dev/xcl+type:pr+is:merged+merged:>=2026-11-26T23:00:02Z`
com 4 resultados, dos quais #2844 (04:50:32Z), #2924 (05:02:20Z) e #2927 (04:20:44Z) são novos e #2893 cai
exatamente no cursor (já documentado na hora 01:00); `commits/a5e2d851` com dois pais e os dois arquivos do
#2927; `gh pr list --state open --limit 500` com 500 linhas, máximo #2957, mínimo #2185 e 180 acima do
cursor 2685; os 20 PRs documentados com `pulls/<n>` (arquivos, `+/-`, `draft`).

Não verificado: a execução dos smokes e benchmarks citados nos corpos (RTX 5070 Ti driver 610.74, RTX 3060
Ti, 7600X, máquina Linux); se `XCL_UMA_DRIVER` e o caminho de encerramento do processo se comportam como
descrito em hardware sem `GetWriteWatch` legível; o estado da CI do repositório; e se os merges em
`nvidia-dev` (#2927 e anteriores) chegam à `main` por alguma promoção de ramo — até esta hora não chegaram.
Nenhum teste foi executado aqui; o repositório monitorado é somente leitura.
