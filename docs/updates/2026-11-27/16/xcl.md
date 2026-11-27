# ivall/xcl — atualização de 2026-11-27 16:00

## Resumo

Cinco commits na `main` desde `c2ee10e4`, dois deles merges de PR (`#2976` e `#2977`) que trazem um
interruptor de preparação de shaders e um conserto no grooming do heap de driver para janelas menores que
1 GiB. O quinto commit é a relicença para GPL-3.0-or-later, feita direto na `main` a partir da issue #3002.
Três PRs foram mesclados na janela: os dois da `main` e `#2947`, que foi para `amd-dev` e não entra no
cursor de commits.

## Commits novos (5)

Faixa `c2ee10e4e905eb736e56953c52b749e18bcc7ede...06f90c1986f43698d0405eff3c2ab0af230e1ed9`,
`ahead_by` 5. Os cinco estão documentados; nenhum corte de teto.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `8617577b67` | Prime | xcl run: XCL_PREPARE=0 starts the title without the shader preparation | +19/−0 em `README.md` e `crates/xcl/src/hle/d3d12x/present/preparation.rs`. `left_out()` compara `XCL_PREPARE` com `"0"` (com `trim`) e, quando bate, `xlog_info!` e `return Ok(())` antes de entrar na preparação; `None` ou qualquer outro valor prepara como hoje. Traz teste unitário da função. É o primeiro de um par: `83b008188e` é o merge de mesmo diff |
| `f78e05bcc2` | Prime | D3D12.X: the driver heap's grooming reserves stretches a smaller window fits in | +50/−4 em `crates/xcl/src/hle/d3d12x/uma_driver.rs`. Antes o grooming reservava só trechos `>= 1 GiB` (menos os primeiros 256 MiB) para forçar o heap do driver para o buraco; agora usa `least = len.min(1<<30)` e `margin = (256 MiB).min(len/2) & !0xFFFF`, então uma janela de 512 MB (layout de 16 janelas) também cerca os trechos em que ela caberia. Segundo de um par: `623d02e465` é o merge |
| `83b008188e` | Prime1997 | Merge pull request #2976 from Prime1997/contrib/prepare-off-switch | +19/−0, mesmo conteúdo de `8617577b67` (README + `preparation.rs`). Traz o interruptor para a `main` |
| `623d02e465` | Prime1997 | Merge pull request #2977 from Prime1997/contrib/uma-driver-small-window | +50/−4, mesmo conteúdo de `f78e05bcc2`. Traz o grooming por tamanho de janela para a `main` |
| `06f90c1986` | ivall | Docs: relicense XCL to GPL-3.0-or-later (#3002) | +641/−303 em 11 arquivos. `LICENSE` troca o texto GPL-2.0 pelo GPL-3.0 (+617/−281), `Cargo.toml` passa a `license = "GPL-3.0-or-later"`, `README.md`/`SECURITY.md`/`AGENTS.md`/`CONTRIBUTING.md`/`THIRD_PARTY.md` acompanham. A tabela de terceiros reclassifica o que vinha de AnyPS5 como "referência apenas, nenhum código" e remove a exceção de uso sob GPL-2.0 com permissão do autor do VIctory; `crates/xcl/src/gpu/recomp/emit.rs` e `hle/bcpack/victory.rs` perdem as menções de licença antiga. Não é PR: o `#3002` é a issue que pediu a mudança (`gh api repos/ivall/xcl/issues/3002`, `is_pr=false`, fechada em `2026-11-27T12:05:35Z`) |

## Pull requests abertos (50)

Janela de `gh pr list --repo ivall/xcl --state open --limit 50`; a lista traz 50 PRs, números 2962–3022,
todos acima do cursor 2708. A janela está no piso: pode haver mais acima, não vistos.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 2962 | D3D12.X: keep unlocked sparse planning inert | planejamento de sparse desligado fica inerte (`stack/311-bounded-sparse-planning`) | Não consta |
| 2963 | Tests: identify benchmark result environments | carimba ambiente nos resultados de benchmark | Não consta |
| 2964 | Shaders: fail closed on wave64 rendezvous timeout | falha fechada no timeout de rendezvous wave64 | Não consta |
| 2965 | D3D12.X: correlate submissions with GPU completion | correlaciona submissão e conclusão na GPU | Não consta |
| 2966 | Scripts: show descriptor table offsets in BVH summaries | imprime offsets de tabela nos resumos de BVH | Não consta |
| 2967 | D3D12.X: DRED is switched on before the process's first device | DRED ligado antes do primeiro device (base `nvidia-dev`) | Não consta |
| 2968 | D3D12.X: XCL_TEXTURE_DROP_LEVELS leaves large textures' top levels without video memory | variável que descarta níveis superiores de texturas grandes (base `nvidia-dev`) | Não consta |
| 2970 | README: distinguish the two title save directories | doc | Não consta |
| 2971 | README: clarify when the fixed-run timer starts | doc | Não consta |
| 2972 | README: explain adapter selection on multi-GPU systems | doc | Não consta |
| 2973 | README: explain how to disable full-size frame dumps | doc | Não consta |
| 2974 | README: qualify uncapped waits with additional handles | doc | Não consta |
| 2975 | D3D12.X: empty shader slots keep a two-pass mesh pipeline's cache key | slots de shader vazios preservados na chave de pipeline mesh de dois passes (base `nvidia-dev`) | Não consta |
| 2978 | Tests: bind both descriptor heaps before the arena root signature | liga os dois heaps antes da root signature da arena | Não consta |
| 2979 | Tests: join game-save enumeration reader on unwind | junta a thread de enumeração de save no unwind | Não consta (D-13 é perda de saves, outro mecanismo) |
| 2980 | Scripts: repeat smoke runs with separate artifacts | repete smoke runs com artefatos separados | Não consta |
| 2981 | D3D12.X: flush pending tile mappings before submit and fix VRAM unmap/retire races | descarrega mapeamentos de tile antes do submit e fecha corridas de unmap/retire (base `amd-dev`) | Não consta |
| 2982 | VFS: DeleteFileW refuses read-only files instead of hiding them (#2945) | recusa em vez de esconder arquivo somente-leitura | Não consta |
| 2983 | xcl run: skip startup shader preparation when no pipeline device exists (#2958) | pula a preparação quando não há device de pipeline | Relaciona-se a `8617577b67` (mesma preparação, outro gatilho) |
| 2984 | README: document scripted gamepad presses | doc | Não consta |
| 2985 | D3D12.X: pool small fallback placements in placed heaps and deduplicate xg::Texture | pool de placements pequenos e deduplicação de `xg::Texture` (base `amd-dev`) | Não consta |
| 2987 | D3D12.X: sparse tiles outside the GPU heaps can live in system memory | tiles sparse fora dos heaps em memória de sistema (base `nvidia-dev`) | Não consta |
| 2988 | D3D12.X: add opt-in gameplay graphics PSO frame budget, scale workers, and log slow compile hashes | orçamento por frame de compilação de PSO gráfico, opcional (base `amd-dev`) | D-02 do catálogo XCL cobria regressão do codegen que derrubava o `amdxc64`, não tempo de compilação; não é o mesmo |
| 2989 | D3D12.X: pool async upload command lists, raise staging item cap to 32 MiB, and lower parallel detile threshold | pool de listas de upload assíncronas, teto de staging em 32 MiB (base `amd-dev`) | Não consta |
| 2991 | D3D12.X: leave no free granule next to the driver heap's hole, and no one-granule remake | endurece o grooming do heap de driver nas bordas | Mesma área de `f78e05bcc2` (grooming), outro caso |
| 2994 | Shaders: --precompile driver compiles follow cores and free RAM (+44% RT pipelines on 16 GB RAM) | paralelismo de pré-compilação por núcleos e RAM livre | Não consta |
| 2997 | D3D12.X: KickoffX keeps a failed Close's effects and goes on with a closed list | lista fechada continua após `Close` falho | Relaciona-se a OpenScarlett D-14 (listas com `Close()` falho), repositório diferente |
| 2998 | D3D12.X: defer driver cache path hashing on compute RAM hits | adia o hash do caminho de cache do driver | Não consta |
| 2999 | D3D12.X: memory by owner on nvidia-dev: two-pass buffers, GPU-view tiles | memória por dono (base `nvidia-dev`) | Não consta |
| 3000 | D3D12.X: a view-by-address desc's padding is zeroed before it keys a texture | zera padding do descritor antes de virar chave de textura | Não consta |
| 3001 | Shaders: record and precompile nonzero compute PSO flags (#2995) | grava e pré-compila flags de PSO de compute não zero | Não consta |
| 3003 | D3D12.X: coherent UMA windows in a driver heap on NVIDIA (port of #2844 to nvidia-dev) | porta para `nvidia-dev` a janela UMA coerente em heap de driver (base `nvidia-dev`) | Porte do que `f78e05bcc2` ajustou na `main`; não corrige defeito catalogado |
| 3004 | D3D12.X: segments parked as spares stop holding their PC allocator (port of #2936 to nvidia-dev) | segmentos guardados como sobressalentes liberam o alocador (base `nvidia-dev`) | Não consta |
| 3005 | D3D12.X: XCL_UMA_DRIVER=mixed keeps part of the coherent window in VRAM (nvidia-dev) | `mixed` mantém parte da janela em VRAM (base `nvidia-dev`) | Não consta |
| 3006 | D3D12.X: XCL_UMA_DRIVER=mixed falls back without GPU_UPLOAD heaps and keeps console buffers in one segment (nvidia-dev) | fallback do `mixed` (base `nvidia-dev`) | Não consta |
| 3007 | D3D12.X: XCL_UMA_UPLOAD_PRIORITY=1 keeps a mixed window's GPU_UPLOAD segment resident (nvidia-dev) | mantém o segmento `GPU_UPLOAD` residente (base `nvidia-dev`) | Não consta |
| 3008 | D3D12.X: XCL_SMALL_TEXTURES=1 puts small sampled textures on 4 KiB placements | texturas pequenas em placements de 4 KiB (base `nvidia-dev`) | Não consta |
| 3009 | D3D12.X: the count check's buffer stays INDIRECT_ARGUMENT between checks | mantém o buffer de contagem como argumento indireto (base `nvidia-dev`) | D-20/D-21 do catálogo XCL tratam de `ExecuteIndirect` (barreira e root signature); mecanismo diferente |
| 3010 | D3D12.X: the tile-mapping fence is signalled once before the next title work | fence de mapeamento de tile sinalizada uma vez (base `nvidia-dev`) | Não consta |
| 3012 | D3D12.X: two-pass ExecuteIndirect uses plain signatures, with Wine's barrier | assinaturas simples no `ExecuteIndirect` de dois passes (base `nvidia-dev`) | Relaciona-se a D-20/D-21, outro recorte |
| 3013 | Shaders: tile_level leaves out elements past the end instead of panicking | não entra em pânico ao passar do fim | Não consta (D-10 é pânico de underflow em `s_setreg_b32`) |
| 3014 | xcl run: xcl.exe uses the segment heap; the exit is logged before ntdll locks the heap | heap de segmento no executável e log de saída | Não consta |
| 3015 | Recompiler: add an opt-in thread-limit mask for direct compute | máscara de limite de threads opcional | Não consta |
| 3016 | Tests: list_x GPU fixtures use the XCL_ADAPTER adapter | fixtures passam a usar `XCL_ADAPTER` | Não consta |
| 3017 | XGameSave: retry MoveFileExW on transient access/sharing errors (#2941) | repete `MoveFileExW` em erro transitório | Não consta (D-13 é perda de saves por container vazio em outro repositório) |
| 3018 | D3D12.X: DXC compiles XCL's own HLSL when D3DCompile refuses it | fallback para DXC (base `nvidia-dev`) | Não consta |
| 3019 | XGameSave: report Win32 HRESULT when container deletion is blocked (#2944) | devolve o HRESULT real | Não consta |
| 3020 | Memory: XCL's own background loops sleep in-process under Wine | laços de fundo dormem no processo sob Wine (base `nvidia-dev`) | Não consta |
| 3021 | D3D12.X: readable checks remember pages and probe instead of asking Wine | memoriza páginas legíveis em vez de consultar o Wine (base `nvidia-dev`) | Não consta |
| 3022 | Scripts: recognize LLVM AT&T listings of command record writers | reconhece listagens AT&T do LLVM | Não consta |

## Pull requests mesclados (3)

Cursor anterior: `2026-11-27T05:02:20Z`. Coleta pela API REST de pulls fechados (`state=closed`,
`sort=updated`), porque `gh pr list --search "merged:>..."` devolve lista vazia neste repositório
(transferido de `xcl-dev/xcl`).

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| 2976 | xcl run: XCL_PREPARE=0 starts the title without the shader preparation | `83b008188e` (main, 06:41:22Z) | o mesmo diff de `8617577b67`: `preparation.rs` ganha `left_out()` e o curto-circuito, README documenta. Prometia um interruptor; entregou o interruptor sem caminho alternativo de preparação |
| 2947 | Recompiler: keep compute shaders on single loop budget to avoid AMD LLPC crash | `5fbd90f977` (amd-dev, 08:38:33Z) | **fora da `main`**: base `amd-dev`, não entra no cursor de commits, que segue a default. Conteúdo do diff não lido nesta hora |
| 2977 | D3D12.X: the driver heap's grooming reserves stretches a smaller window fits in | `623d02e465` (main, 08:50:20Z) | o mesmo diff de `f78e05bcc2`: `least`/`margin` proporcionais ao tamanho da janela, com a explicação de que a janela de 512 MB deixava um trecho entre 512 MB e 1 GiB livre e o driver mapeava o heap ali |

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum dos cinco commits fecha entrada de `XCL` no catálogo (D-01 a D-24). `f78e05bcc2`
  mexe em janela UMA, mas D-01 é a escolha da janela pelo dword alto da base sem carry/borrow de 32 bits
  (`f3b15b9` → `f9e5ee9`), outro mecanismo: aqui o alvo é o *grooming* de trechos livres fora do buraco.
  D-02 (regressão de codegen que derrubava o `amdxc64`) não é tocado.
- **Abre defeito novo?** Não identificado no diff. Duas observações de risco:
  - `8617577b67`: com `XCL_PREPARE=0` o título "constrói o que usa conforme usa" e fica parado nessas
    construções, o que pode ser lido como travamento em bancada. O texto do README e do `xlog_info!` diz
    isso explicitamente, então é comportamento documentado, não regressão.
  - `f78e05bcc2`: o conserto depende de o trecho reservado ficar grande o bastante; o diff não traz
    medição de título real, só o teste. Efeito em run real: **NAO VERIFICADO.**
- **Licença**: `06f90c1986` alinha o XCL ao VIctory (GPL-3.0) e reclassifica o que vinha do AnyPS5 como
  referência, o que responde à divergência de licença registrada para AvatarManz (D-17) no que toca ao
  XCL. Não há entrada `XCL` no CSV para isso.

## Confiança

- **Verificado pelo diff**: os cinco commits, pelo endpoint `commits/<sha>`. `83b008188e` e `623d02e465`
  foram conferidos como idênticos a `8617577b67` e `f78e05bcc2` (mesmas estatísticas e mesmo patch), o que
  é consistente com merge fast-forward/squash dos dois PRs.
- **Verificado por metadado**: os três merges (número, `mergedAt`, base, `merge_commit_sha`) pela API REST;
  a issue `#3002` foi aberta na API para confirmar que não é PR.
- **Não verificado**: o diff de `#2947` (`amd-dev`) e o de todos os 50 PRs abertos — a coluna "Proposta"
  vem do título e do nome da branch, não do patch. A janela de abertos está no piso de 50 (números
  2962–3022), então o total acima do cursor 2708 é ≥ 50.
- **Contagem de commits**: `ahead_by` de `compare/c2ee10e4...06f90c19` = 5, e a `main` não se moveu
  durante a coleta (reconferida depois: mesmo `06f90c19`).
