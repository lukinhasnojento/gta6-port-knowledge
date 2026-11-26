# californiaAlmonds/OpenScarlett — atualização de 2026-11-26 19:00

## Resumo

**27 commits** desde `7220ce94`, todos documentados (abaixo do teto de 40). O bloco é contínuo e gira em torno de
quatro frentes: DirectStorage/RDS5 (resolvedor exato da tabela de chunks, spans entre chunks, cache de slices),
memória física (transferência *scatter* sem syscall, mapeamento de tiles em lote, PFNs livres mais baixos),
o lifter RDNA 2 (DPP16, `writelane` wave64, BFM/QUADMASK/SAD/MIMG, VOP3 `0x347`/`0x371`) e uma série de guardas
de VEH para travamentos de título retail. Entra um commit de build (`/utf-8` no MSVC) via PR #50, o único PR
mesclado na janela, e três merges de ramo de trabalho cujo conteúdo é idêntico ao commit que agregam.

## Commits novos (27)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| eb6234d3 | leon | fix(build): force MSVC UTF-8 source & execution charset | Em `CMakeLists.txt`, dentro de `if(MSVC)`, adiciona `add_compile_options(/utf-8)`: 15 arquivos sob `src/` têm bytes não-ASCII e, sem a flag, o MSVC decodifica pelo code page do host (C4819 com `/W4`, C2001 quando o byte cai em literal). |
| dedf0a8b | californiaAlmonds | feat(dstorage,xmem,d3d12_xs): add exact RDS5 chunk-table resolver, XMem fail logs, and hash scan guard | `DStorageFileImpl::ResolveRds5Request` resolve o pedido por três passos — `comp_off` exato, *shift* constante memorizado (`last_rds5_shift_`) e varredura para trás de 16 MiB — devolvendo `true_off`, tamanho descomprimido e *slice*; `xmem_shim` loga as 16 primeiras falhas de `XMemMapPhysicalPages` com `err=`; o VEH do `d3d12_xs` intercepta a varredura linear de tabela hash em RVA `+0x28CD253` e devolve `0xFFFF` ("não achado") pulando para `+0x28CD265`. |
| 2d9d3dc0 | californiaAlmonds | fix(dstorage,loader): handle shifted RDS5 prefix/tail sub-chunks and preserve OOB chunk completion | O caminho de *shift* aceita `uncomp_sz >= dst_sz` (sub-chunk deslocado de prefixo/cauda) e guarda um `fallback_match`; no `hook_dll`, os resgates estruturados de epílogo do INT3 passam a rodar **antes** do `Rip += 1` genérico também quando `OPENSCARLETT_CONTINUE_INT3` está ligado, para não cair em sentinelas fora da tabela de chunks. |
| d9388800 | californiaAlmonds | feat(kernel,d3d12_xs,dstorage): add large sparse XMem backing, direct __XMemAllocationHooks__ guard, and RDS5 multi-slice cache | `OPENSCARLETT_FORCE_SPARSE_COMMIT=l` passa a exigir ≥ 4 MiB (em vez de forçar tudo); o proxy de device chama **direto** o par `__XMemAllocationHooks__` exportado pelo título (`tbl[0]`/`tbl[1]`, allocatorId `0x80`, código de página 1/2 e `log2(alignment)`), sem cair em `XMemAllocDefault`, e prende um `TitleHookReleaseGuard` ao recurso para chamar o `free` na destruição; o `DStorageFileImpl` ganha um anel de rastreio de slices RDS5 e cache do último chunk descomprimido. |
| 9871a6e9 | californiaAlmonds | fix(dstorage,d3d12_xs): assemble cross-chunk RDS5 spans and generalize watchdog trap filter | Monta spans RDS5 que atravessam mais de um chunk; o filtro do *watchdog* de título passa de duas assinaturas fixas (`0x4D8BD421`) para `C7 04 25 00 00 00 00 <imm32>` e escolhe o comprimento por `rip[11] == 0xCC && rip[12] != 0xCC` (12) ou 11. |
| 378b7105 | californiaAlmonds | feat(kernel,dstorage,d3d12_xs): add zero-syscall physical scatter transfer, VirtualQuery cache, and empty functor guard | Novo export `OpenScarlettTransferTitlePhysicalPages` (abi + implementação em `kernelx_shim`): resolve todos os PFNs sob um único `mutex_`, aponta para uma **view R/W persistente** de cada seção física (`ResolvePhysicalViewPtr`, `MapViewOfFile` uma vez por pool) e copia fora do lock; o DirectStorage usa isso primeiro e só cai em `MapTitleVirtualMemory` se o retorno for `-1`. Em `command_queue_x`, `IsWritableCpuVaCached` guarda o intervalo `VirtualQuery` por lote de escritas diferidas. No VEH do `d3d12_xs`, um AV em `Rip == 0` cujo endereço de retorno está depois de `FF 50 50` (`call [rax+0x50]`), com `rax != 0` e `[rax+0x50] == 0`, é resolvido com `al = 1` e retorno ao chamador. |
| ddb07818 | californiaAlmonds | docs(readme): condense README, update live status, and remove UE5 claims | Só `README.md` (+55/−157): condensa o texto e remove as afirmações sobre UE5. |
| 4ff81865 | californiaAlmonds | feat(kernel,xmem,d3d12_xs): implement GPU tile page-batch mapping, lowest-free PFNs, and async unmap slack | `kernelx_shim` (+400) implementa mapeamento de tiles de GPU em lote por página, mantém um conjunto `free_pfns_` (o menor PFN livre é reutilizado) e adiciona folga assíncrona ao desmapear; `xmem_shim` perde 68 linhas do caminho antigo e o ABI ganha três declarações. |
| b11868a4 | californiaAlmonds | fix(d3d12_xs): guard empty streaming ring bucket pop race and extend R9G9B9E5/packed RTV stand-ins | VEH para o *pop* de bucket circular (`48 8B 01 / 48 8B 50 18 / 48 89 11 / C3` em `+0x5298EE3` e `+0x188013`): quando um worker desregistra sem lock e esvazia `[rcx]` depois de o chamador ler `[rcx+8]`, reconcilia a contagem e desenrola para o epílogo do chamador. Em `device_x`, +36/−7 para *stand-ins* de RTV `R9G9B9E5` e formatos empacotados. |
| 18280a0b | californiaAlmonds | fix(loader): enforce structured INT3 bounds-exit guards under OPENSCARLETT_CONTINUE_INT3 | Reordena `Zen2IllegalInstructionVeh`: os dois resgates de epílogo (contador de chunk e `EnqueueChunksForRead`) rodam mesmo com o modo de continuação ligado; só depois o `allow_continue` genérico é considerado. Evita que `OPENSCARLETT_CONTINUE_INT3=1` pule para dentro de sentinela fora da tabela. |
| ee7153c7 | californiaAlmonds | feat(d3d12x,kernelx): implement 3D-world streaming remap VEH, retail slots 76/148/74/31, Mesh57 wg, RTAS SRV & 4K tex | +1 556/−189 em nove arquivos: `SanitizeRetailResourceDesc` passa a substituir `B5G6R5`, `B5G5R5A1` e `B4G4R4A4` por `R8G8B8A8` quando há RT/UAV (antes só `B4G4R4A4` → `BGRA8`); entram o VEH de *remap* de streaming do mundo 3D, os slots retail 76/148/74/31, o *wavegroup* de Mesh57, SRV de RTAS e textura 4K, com trabalho correspondente em `command_list_x`, `command_queue_x` e `device_x`. |
| d58bfd51 | californiaAlmonds | Merge branch 'wt/d3d12x-3d-world' | Merge do ramo `wt/d3d12x-3d-world`; mesmos nove arquivos e mesmas estatísticas de `ee7153c7` (+1 556/−189). Não acrescenta conteúdo próprio. |
| 26c9d8a6 | californiaAlmonds | docs(readme): group hardware reports by GPU generation and credit new testers | Só `README.md` (+29/−21): reorganiza os relatórios de hardware por geração de GPU e credita testadores. |
| 6360d416 | californiaAlmonds | Merge pull request #50 from leonredfish/fix/msvc-utf8-charset | Merge do PR #50; conteúdo igual a `eb6234d3` (`CMakeLists.txt` +8, `add_compile_options(/utf-8)`). |
| 9c6aa503 | californiaAlmonds | docs(readme): credit @leonredfish via GitHub profile | Só `README.md` (+2/−2): troca o crédito pelo perfil do GitHub. |
| 19b24905 | californiaAlmonds | feat(gpu): complete RDNA2 DPP16, writelane Wave64 spills, BFM/QUADMASK/SAD/MIMG atomics, and V# UFMT | +2 947/−187 no lifter: `WaveDpp16`/`WaveDpp16Mask`/`WaveWriteLane` entram em `IsCrossLaneWaveOp` e em `ModuleUsesWaveSwizzle` (o caminho de *swizzle* por lane passa a valer para eles); `rdna2_lifter` (+1 060) emite as formas, `rdna2_ir` (+805) e `rdna2_disasm` (+307) ganham os opcodes, e `descriptor_sgpr_bridge` (+136) trata `UFMT` do V#. Teste `test_s16_shader_completeness` +520 linhas. |
| 7e6dcdf7 | californiaAlmonds | docs: update Discord invite link | Troca o convite do Discord em nove arquivos (README, `docs/index.html`, `tools/generate_status.py`, seis páginas de wiki). |
| 9c84659e | californiaAlmonds | Merge branch 'wt/rdna2-3d-isa' | Merge do ramo `wt/rdna2-3d-isa`; mesmos sete arquivos e estatísticas de `19b24905`. Não acrescenta conteúdo próprio. |
| c5c95f9d | californiaAlmonds | fix(kernelx,d3d12_xs): accept PFN=0 scatter unmap sentinel and reconcile RGBA8/BGRA8 stand-in views | O *scatter unmap* trata `pfn == 0` como sentinela de fim de lista (libera as entradas anteriores e para), e `ResolveCompatibleSrvFormat` passa a aceitar view `RGBA8` sobre tipeless `RGBA8` para pedidos `BGRA8`/`BGRA8_SRGB` e vice-versa, mais os formatos empacotados de 16 bits. |
| 2902ad33 | californiaAlmonds | fix(d3d12_xs): preserve stencil plane SRV formats on typeless depth-stencil textures | `ResolveCompatibleSrvFormat` devolve o formato de plano pedido para `R32G8X24_TYPELESS` (`R32_FLOAT_X8X24`, `X32_TYPELESS_G8X24_UINT`) e `R24G8_TYPELESS` (`R24_UNORM_X8`, `X24_TYPELESS_G8_UINT`), em vez de cair no formato default e perder o plano de stencil. |
| 60743d27 | californiaAlmonds | fix(kernel,d3d12_xs): invalidate TitleVAMirror on XMem remaps and unmap stale GPU tile PFNs on free | `title_va_mirror` ganha invalidação por intervalo; o `kernelx_shim` agrupa os intervalos remapeados e chama `OpenScarlettD3D12InvalidateActiveVirtualRange` exportado por `d3d12_xs.dll` (resolvido por `GetProcAddress` e cacheado), e ao liberar um bloco desmapeia os PFNs de tile que ficaram apontando para ele. |
| b1e5ee09 | californiaAlmonds | fix(xmem,kernel,d3d12_xs): keep Scarlett allocations <40-bit, handle 3-arg null unmap, and heal fwPool ABA | `AllocVirtualWriteWatched` ganha `require_below_40bit`: tenta primeiro uma colocação abaixo de 2^40 (ponteiros tagueados de 40 bits e VAs de GPU/BVH do retail truncariam com ASLR alto), mantendo `MEM_WRITE_WATCH`; o desmapeamento aceita a forma de 3 argumentos com `nullptr`; `fwPool` ganha cura de ABA. Teste `test_mine_d3d12_xmem` +49. |
| 4e67f0cb | californiaAlmonds | fix(loader,d3d12_xs): expand Scarlett PRT pool, guard out-of-band descriptor pools and unbound shader vars | `hook_dll` +306: resgate do alocador PRT (*free-list* vazia em `+0x55CC218`, `CC EB 84`) que dá `SwitchToThread` e rebobina para `+0x55CC180` até 512 vezes antes de desistir, em vez de andar +1 e corromper o slot sentinela 0. Em `d3d12_xs_shim`, +107 para pools de descritor fora da faixa e variáveis de shader não ligadas. |
| 0bc6c5d1 | californiaAlmonds | fix(d3d12_xs): guard unplaced polymorphic extension vtable clones at +0x17ABCA1 | VEH *case E*: quando um sub-objeto de extensão *streamed* tem vtable nula (`rax == 0` no `call [rax+0x10]`), avança 3 bytes até `mov [rsi], rax` (`48 89 06`) para gravar `nullptr` e liberar a seção crítica `+0x80122E8`, em vez de estourar. Loga até 8 ocorrências. |
| 30e2405d | californiaAlmonds | fix(kernelx,d3d12_xs,loader): reclaim unmapped placed VRAM, enable <40-bit sparse commit and guard paged fwPool | `CreatePlacedShaderResourceViewX` deixa de soltar a textura *lazy* ao achá-la em `unified_textures_`: marca `is_lazy_placed`/`gpu_alias_va` e chama `MarkLazyPlacedTexture`, para que o VRAM colocado desmapeado seja recuperável; `xmemory` e `kernelx_shim` habilitam *sparse commit* abaixo de 2^40; `hook_dll` +98 guarda o `fwPool` paginado. |
| d33419c0 | californiaAlmonds | fix(gpu,d3d12_xs): add VOP3 0x347/0x371, acyclic DAG & mesh epilogue HLSL and CmdList Slot 85 | Acrescenta `v_sad_hi_u8`/`v_msad_u8` (VOP3 `0x347`/`0x371`) ao disassembler/IR/lifter do GPU, troca `hoisted_mesh_counts_` por `mesh_stage_slots_` com `groupshared uint _gs_mesh_counts[2]` para o epílogo de mesh, e trata o slot 85 de `ID3D12GraphicsCommandList`. Teste de segurança de CFG +153. |
| 48a98fcd | californiaAlmonds | fix(d3d12_xs): fix Mesh PSO root sig v1.0/1.2/fallback, bound 4GB VRAM/lazy textures, and guard streaming RVAs | +1 081/−423, quase tudo em `device_x.cpp`: `CreateRootSignature` negocia versão 1.0/1.2 e cai num fallback quando o blob serializado não é aceito; o caminho de textura *lazy* passa a respeitar o teto de 4 GiB de VRAM; `command_list_x` e `d3d12_xs_shim` ganham guardas para RVAs de streaming. Teste de CFG +165. |

## Pull requests abertos (0)

Nenhum PR aberto com número acima do cursor 60 no momento da coleta: `gh pr list --state open --limit 100`
devolveu 2 PRs, ambos com número ≤ 60 e já contabilizados. `last_pr_number` permanece 60.

## Pull requests mesclados (1)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| 50 | fix(build): force MSVC UTF-8 source & execution charset | 6360d416 | Exatamente `add_compile_options(/utf-8)` sob `if(MSVC)` em `CMakeLists.txt` (+8). O commit que o PR prometia é `eb6234d3`; o merge é vazio de conteúdo próprio. `mergedAt` = `2026-11-26T07:52:24Z`. |

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum dos 27 hashes é `corrigido` de um defeito de `OpenScarlett` (D-01…D-24). Proximidades
  que valem registro:
  - **D-01** (DirectStorage lia o pedido no layout de desktop) foi corrigido em `76adeff`, antes da janela. Todo
    o trabalho RDS5 desta faixa (`dedf0a8b`, `2d9d3dc0`, `9871a6e9`, `d9388800`) é camada seguinte sobre aquele
    conserto, não uma revisão dele.
  - **D-22** (`notify_one`/`notify_all` fora do lock no DirectStorage, use-after-free) foi corrigido em `a208562`.
    Os novos caminhos (`rds5_slice_mutex_`, `heap_lock`, `mutex_` do `KernelXState`) mantêm a notificação e a
    cópia fora do lock, coerentes com a correção.
  - **D-21** (`ds_sub` em LDS emitido como `InterlockedAdd` de valor negado, recusado pelo driver NVIDIA) é do
    lifter de shader; `d33419c0` não toca LDS. O mesmo padrão (`Interlocked*` com valor negado) reaparece no
    `compute_store_aot.py` do Xbox2Windows nesta mesma hora (`343dc04f`), o que sugere que o padrão é aceito
    pelo driver em compute e recusado em pixel — não verificado.
  - **D-15** (orçamento de residência cumprido evictando heaps vivos) é vizinho de `30e2405d`, que recupera VRAM
    colocada desmapeada; são mecanismos distintos.
- **Abre defeito novo?** Três candidatos, **NAO VERIFICADO**:
  1. `dedf0a8b` — o VEH de `+0x28CD253` devolve `0xFFFF` ("não achado") sempre que a varredura linear da tabela
     hash toca página não mapeada. Se o dicionário parcial for legítimo (carregamento incremental), a busca
     passa a **perder** entradas em vez de falhar de forma visível (`src/shims/d3d12_xs_shim.cpp`, guarda
     `LookupByHash`).
  2. `378b7105` — a guarda de functor vazio trata `call [rax+0x50]` com `[rax+0x50] == 0` como predicado
     verdadeiro (`al = 1`) em qualquer ponto do executável principal. Isso converte um ponteiro de invocação
     nulo em "predicado satisfeito" sem distinguir o *call site* previsto (`+0x4E53906`) de outro com a mesma
     forma; um caminho que devesse recusar passaria a executar o ramo.
  3. `9871a6e9` — o comprimento do pulo do *watchdog* é decidido por `rip[11] == 0xCC && rip[12] != 0xCC`.
     Um `int3` de 12 bytes seguido imediatamente de outro `0xCC` seria classificado como 11 bytes e deixaria o
     segundo `int3` para ser tratado como exceção real; não há teste com essa sequência.
- **Não é defeito, mas limita:** `d9388800` chama o hook de alocação do título **sem** fallback para
  `XMemAllocDefault`; se o hook devolver nulo, nenhum guard de orçamento é preso ao recurso. O diff não trata
  esse caso explicitamente.

## Confiança

- **Verificado por diff:** os 27 hashes foram lidos em `gh api repos/californiaAlmonds/OpenScarlett/commits/<sha>`
  (arquivos, estatísticas e hunks). Os três merges (`d58bfd51`, `9c84659e`, `6360d416`) foram comparados com os
  commits que agregam: mesmos arquivos, mesmas estatísticas, conteúdo idêntico.
- **Cobertura:** 27 de 27 commits; `last_commit` vai para `48a98fcd31a9f5d6c8ba344592215c1539d6e7dc` (head na
  coleta). Nenhum backlog de commits neste repositório.
- **PRs:** 0 abertos novos; 1 mesclado na janela, casado por `mergeCommit.oid` com `6360d416`. O cursor de
  mesclados vai para `2026-11-26T07:52:24Z`.
- **NAO VERIFICADO:** os três candidatos a defeito novo; o efeito real das guardas de VEH em títulos que não os
  casos citados nos comentários (`steps-ab2`, `start-3`, `play-1125-2107`); e se `4ff81865`/`ee7153c7` foram
  exercitados em hardware além do relatório do autor.
