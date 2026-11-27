# R0Y00/Xbox2Windows — atualização de 2026-11-27 20:00

## Resumo

37 commits entre o cursor `300b93f0` e o head `63a30dc1`. Trinta deles são a série do #2911 (page
runs do XMem sob Wine, mesclada às 17:03:14 no relógio do GitHub), um é o #3114 (`s_quadmask` no
lowering direto de compute, squash em `2641cc72`), dois são merges e cinco são commits de revisão que
só tocam comentário, documentação ou fixture. Três PRs mesclados na janela — um deles (#3130) foi
mesclado em `codex/native-graphics-integration`, não na `main`, e por isso não aparece na faixa.
Dez PRs abertos acima de #3121.

## Commits novos (37)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `17ee827d` | JohnConnorNPC | xmem: split slot 31 placeholders when the OS hides the placeholder bit | `src/xmem_mapped_pages.cpp` +48/-6: sonda única no primeiro uso liga um fallback quando o SO não reporta o bit `0x100` do placeholder (Wine), porque todo split ou coalesce de slot 31 cuja run não era exatamente um placeholder falhava com `ERROR_INVALID_ADDRESS`. Onde o bit existe, o caminho é o de antes |
| `364d3b4b` | JohnConnorNPC | tests: slot 31 split inside an XMem reservation, with a committed-target control | `tests/xmem_placeholder_split_check.cpp` +109 e 15 linhas de registro no `CMakeLists.txt`: fixture do split de slot 31 dentro de reserva XMem, com controle cujo alvo já está comprometido |
| `e44e3cd5` | JohnConnorNPC | docs: placeholder fallback for slot 31 splits under Wine | README +1 e `docs/experiments.md` +2: nota do fallback |
| `f1f90abf` | JohnConnorNPC | xmem: copy DirectStorage destinations under the mapping mutex | `xmem_mapped_pages.cpp` +75/-1, `xcrdapi.cpp` +8/-21, `local_log.h` +26, `xmem_shim.def` +1: a checagem e a cópia do destino passam para dentro do `xmem.dll`, sob `memory.mutex`. Antes, o pré-`VirtualQuery` do XCRDAPI podia rodar com a página desmapeada por um break e recusar destino que estava mapeado antes e depois |
| `b7b1552e` | JohnConnorNPC | startup: copy DirectStorage data through x2w_xmem_copy_to_destination | `xcrdapi.cpp` +23/-5: XCRDAPI passa a chamar o export novo em vez de checar e copiar por conta própria |
| `4de5c09e` | JohnConnorNPC | tests: DirectStorage copy during an XMem run break | `startup_dstorage_break_window_check.cpp` +240 e 33 linhas de CMake: mantém um break aberto e submete cópia para página que lê `MEM_RESERVE` dentro dele; a requisição tem de esperar e completar com todos os bytes |
| `06dfe20e` | JohnConnorNPC | xmem: split an owned placeholder before mapping a page into it | `xmem_mapped_pages.cpp` +46/-5: `map_split` ganha `exact_placeholder()` e faz o split **antes** do `MapViewOfFile3` quando o destino é o meio de um placeholder maior; se split ou map falham, roda o caminho antigo (map primeiro). Novo export de teste `x2w_xmem_map_counts`; `tests/xmem_map_split_first_check.cpp` +93 |
| `be3f918c` | JohnConnorNPC | xmem: answer non-faulting readers outside run breaks | novo export `x2w_xmem_stable_call(fn, ctx)`: roda `fn` uma vez quando nenhum break começa ou termina em volta e, senão, uma segunda vez sob o mutex. Passam a usar: `XMemVirtualQuery`, `x2w_local::writable`, `measure_placed_buffer` (slot 133) e `snapshot_samplers` |
| `e6e7d9b9` | JohnConnorNPC | xmem: answer the main module's VirtualQueryEx outside run breaks | `xmem_mapped_pages.cpp` +38: com shared views ligadas, o import `VirtualQueryEx` do módulo principal vai para um wrapper que usa `stable_call`; o módulo só é pinado quando algum import foi trocado |
| `0ebef8a7` | JohnConnorNPC | xmem: retry a fault in the break after the one the handler saw end | correção de defeito anterior ao PR: um handler que lia a geração do break depois do fim nomeava o break **seguinte**, e o fault na mesma página nesse break não era retentado. Fixture `xmem_run_break_readers_check.cpp` +96 |
| `284f05ec` | JohnConnorNPC | xmem: run readers once without shared views, and write their answers after a break | `xmem_mapped_pages.cpp` +29/-10, `xmem_shim.cpp` +9/-4, `placed_buffer_range.h` +6/-1: sem o switch `stable_call` chama `fn` direto; respostas de query vão para cópia local e são escritas no buffer do chamador só depois (o buffer pode estar em página lazy) |
| `4408f03a` | JohnConnorNPC | xmem: join a page mapped on its own call to the view before it | `xmem_mapped_pages.cpp` +71/-2 e `tests/xmem_cross_call_merge_check.cpp` +329: uma chamada cujas páginas formam uma run junta-se à run registrada que termina no endereço, só quando a primeira página é o slot seguinte da mesma seção e ambos estão na mesma reserva do shim; a run é desmapeada dentro de um break, os placeholders são juntados e uma view é mapeada sobre os dois. Recusa antes do unmap: nada muda; depois: layout antigo restaurado |
| `a4dc3f9c` | JohnConnorNPC | xmem: stop a joined run at 64 pages | `constexpr SIZE_T max_merged_run_pages = 64` e `merged_run_cap()` (override `X2W_XMEM_TEST_MERGE_CAP` só no build de teste); `merge_with_previous` devolve `Merge::separate` quando `(size + bytes)/page_bytes > cap`. Fixture `capped` |
| `1bf82dd2` | JohnConnorNPC | xmem: retry a fault from any of the last 64 breaks, not only the last | anel de 64 breaks com ranges por break (substituído depois por `c8391709`) |
| `3985646e` | JohnConnorNPC | xmem: run title ReadFile/WriteFile again after a run break refused the buffer | `xmem_mapped_pages.cpp` +84: imports `ReadFile`/`WriteFile` do módulo principal viram wrapper; repetição até 4 tentativas só em `ERROR_NOACCESS`/`ERROR_INVALID_USER_BUFFER`, com break sobreposto e buffer dentro de uma run |
| `f415ffcd` | JohnConnorNPC | xmem: write back only what XMemVirtualQuery's query writes, and pass lines at the end | `xmem_shim.cpp` +5/-5: o write-back cobre os bytes que a query escreve e nada além |
| `4a43a173` | JohnConnorNPC | xmem: say the run cap bounds joined runs only, and test the break ring's edge | docs de `startup-runtime.md` +4/-2, 4 linhas em `xmem_mapped_pages.cpp` e teste de borda do anel: o teto vale só para runs juntados; view de uma chamada ou páginas remapeadas por um break continuam podendo ser maiores |
| `ad1980cb` | JohnConnorNPC | tests: map single-page XMem views from high to low | `xmem_page_runs_device_check.cpp` +3/-3: mapeia as páginas de alta para baixa (mesmo patch-id de `f6020249`, do #2918) |
| `238ed78a` | JohnConnorNPC | tests: preserve the XMem linear buffer view boundary | `linear_buffer_backing_check.cpp` +9/-8: mapeia a página superior do buffer `split` primeiro e limpa o switch de join (mesmo patch-id de `27778f4f`, #2918) |
| `14a9589f` | JohnConnorNPC | tests: move the XMem page-run registrations to tests/cmake/xmem_page_runs.cmake | move 118 linhas de registro verbatim para `tests/cmake/xmem_page_runs.cmake`; `CMakeLists.txt` volta a ser o da `main` |
| `e9e0b441` | JohnConnorNPC | xmem: put the slab accounting comment back above x2w_xmem_slab_stats | comentário de contabilidade de slab volta ao lugar; comentários de fixture deixam de apontar notas fora do repositório |
| `d4f1344e` | JohnConnorNPC | xmem: put joins across mapping calls behind X2W_XMEM_JOIN_CALLS | joins passam a exigir `X2W_XMEM_JOIN_CALLS=1` além de shared views (`run.sh` liga, `run.ps1` não); novos fixtures `_no_join` e `xmem_page_runs_device_join_warp` |
| `cb8cc808` | JohnConnorNPC | xmem: honour the two split test switches only in the test-controls build | `X2W_XMEM_TEST_MAP_FIRST` e `X2W_XMEM_TEST_NO_PLACEHOLDER_FALLBACK` só são lidos sob `X2W_XMEM_TEST_CONTROLS`; a DLL publicada não contém os nomes (linhas `_deployed` novas) |
| `1615efc5` | JohnConnorNPC | xmem: check every region of a view the placeholder fallback accepts | `xmem_mapped_pages.cpp` +47/-11: o fallback passa a checar **todas** as regiões que a view cobre, não só a primeira de cada alocação; fixture `_partial` nova. Sob Wine, com a caminhada antiga, o lote era aceito e a view substituía a página comprometida (byte `0x53` em vez da tag `0x7a`) |
| `c8391709` | JohnConnorNPC | xmem: stamp each run with the break that mapped it instead of a 64-break history | `PhysicalMemory::runs` vira `map<base, Run{bytes, stamp}>`; `begin_break()` devolve o id e cada view mapeada dentro do break grava o stamp. `retry_remapped_page` retenta por (página, stamp) por thread, e um fault que não pode ser o transitório passa sem pegar `memory.mutex` (checagem de geração par + ausência de view mapeada). Remove `RemapWindow::kept`, o anel e `newest_holding` |
| `4f7d47de` | JohnConnorNPC | xmem: run a failed ReadFile/WriteFile again only on a disk file | `xmem_mapped_pages.cpp` +9/-7: o retry só vale para handle de disco (no pipe, o Wine mantém os dados na fila; caso `ReadFile_pipe` novo) |
| `f19890e4` | JohnConnorNPC | xmem: take the mapping mutex for a DirectStorage copy only when it touches a run | `xmem_mapped_pages.cpp` +75/-11, novo `resident_pages.h` +31: o mutex só é tomado com shared views ligadas e destino ou origem sobrepostos a uma run registrada; antes disso, checa páginas residentes RW como o XCRDAPI sem xmem.dll (#2685). Export de teste `x2w_xmem_copy_counts` |
| `302618bd` | JohnConnorNPC | dstorage: log the destination copy's outcome by its error code | `xcrdapi.cpp` +17/-3: quatro desfechos nomeados (destino não gravável, cópia com fault, requisição inválida, falha com código); fixture `_outcomes` |
| `0cfdd607` | JohnConnorNPC | docs, ci: move the page-run note to docs/experiments/ and add the fixtures to the pr preset | `docs/experiments/xmem-page-runs.md` +63; `CMakePresets.json` +6/-2 faz o preset `pr` construir as fixtures e rodar 28 testes que não rodava |
| `f9391cca` | JohnConnorNPC | gh #2911 review round 1: state the placeholder fallback's limit for MEM_REPLACE_PLACEHOLDER allocations | 7 linhas em `xmem_mapped_pages.cpp` e 28 no fixture: registra que o fallback não distingue página reservada de alocação privada feita com `VirtualAlloc2(MEM_REPLACE_PLACEHOLDER)` e que, após view recusada, o retry página a página corta a página da alocação do chamador |
| `f4cc2900` | JohnConnorNPC | gh #2911 review round 1: say which half of the pipe retry reason is measured | 2 linhas: qual metade da razão do retry de pipe está medida |
| `863c0459` | JohnConnorNPC | gh #2911 review round 1: describe the readers fixture's handler by its stamp lookup | comentário do handler no fixture (15/-14) |
| `bf9e448f` | JohnConnorNPC | gh #2911 review round 1: list no-join in the cross-call merge fixture's usage comment | comentário de uso do fixture de merge (1/-1) |
| `0c6be971` | JohnConnorNPC | gh #2911 review round 1: put the page-run note link next to the row's first link | README (1/-1) |
| `104617d6` | JohnConnorNPC | Merge main into memory/merged-page-runs | merge de `main` na branch. Pais `0c6be971` + `215934bd`; `215934bd` está **105 commits atrás** do cursor `300b93f0`, ou seja, o conteúdo que entra (83 arquivos, +5835/-221, incluindo `gfx2hlsl.py` +438/-39 e `d3d12_xs_device.cpp` +208/-11) é o da `main` já documentada, reentrando na branch — não é trabalho novo da janela |
| `2641cc72` | rdx | shader: lower scalar quadmask operations in direct compute (#3114) | `src/gfx1013_direct_compute.inl` +34: `s_quadmask_b32` (SOP1 `0x2c`) e `s_quadmask_b64` (`0x2d`) entram em `mask_scc`, com colapso de nibbles sem branch (`and i32` com `286331153`), `bits[7:0]` no b32 e `bits[15:0]` com dword alto zerado no b64, `SCC = (D != 0)` e par SGPR par exigido no b64. Mais `docs/experiments/native-s-quadmask.md` e `tests/native_s_quadmask_check.py` (+142) |
| `63a30dc1` | John Connor | Merge pull request #2911 from JohnConnorNPC/memory/merged-page-runs | merge do #2911. Pais `2641cc72` (`main`) + `104617d6`. Diff contra o primeiro pai: 23 arquivos (+3211/-164), com `xmem_mapped_pages.cpp` +677/-75 |

## Pull requests abertos (10)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #3139 | runtime: recover the published disk pair when native candidate PSO creation fails | base `codex/native-graphics-integration`: se `CreateGraphicsPipelineState` do candidato nativo falha, recarrega o par publicado em disco para a mesma chave e variante RTV, refaz o PSO com revisão incrementada e recusa com diagnóstico se o par não existe (+121/-12) | Não. É correção de caminho do pipeline nativo; sem entrada no catálogo |
| #3138 | shader: invalidate watch manifests when sidecar content changes with an older mtime | base `codex/native-shader-modules`: compara o conteúdo canônico de `.psin`/`.ps57in` contra `<key>.psdep` via `read_pixel_input_description()` em vez de comparar mtime (+143/-14, 4 arquivos) | Não. Área de cache de manifesto; nenhum D-xx trata de sidecar |
| #3137 | compile_dxil: invalidate a published manifest when sidecar content changes with an older timestamp | base `codex/native-shader-modules`: grava digest FNV-1a do sidecar em `<key>.psinh` e o watcher retira o manifesto quando o digest difere (+130/-2) | Não |
| #3136 | shader: harden compile_lds_request CLI, path guards, and replay checks with offline unit suite | base `main`: `main(argv)` com `SystemExit`, `_validate_paths` para `.c67` e cinco arquivos companheiros, erros viram `error:` e código 1, suíte sintética com 8 mutações (+693/-48) | Não |
| #3135 | shader: lower v_floor_f16, v_ceil_f16, v_trunc_f16 and v_rndne_f16 in the direct compute path | base `main`: quatro opcodes de arredondamento binário16 em inteiro de 32 bits, preservando `D[31:16]` e zero com sinal; recusa `omod`, `op_sel` e fontes VOP3 não usadas (+250/-0) | Não corrige; mesma frente do #3114, que fechou a issue #3112 |
| #3129 | cpu: harden cpu_chained_aot CLI validation and atomic writes | base `main`: validação de caminhos, `callback_offset` só `int` (recusa `bool`), escrita atômica por arquivo irmão e suíte PE32+ sintética (+535/-6) | Não |
| #3128 | docs: record the Intel Arc A380 hardware report | base `main`: uma linha de cobertura de hardware no README para o relatório do Arc A380 (+1/-0) | Não |
| #3126 | tooling: harden atomic writes, path guards, and CLI entry in cpu_call_aot with synthetic PE32+ suite | base `main`: mesma frente de #3129 para `cpu_call_aot.py` (+988/-6) | Não |
| #3125 | shader: lower v_min_f16 and v_max_f16 in the direct compute path | base `main`: `v_min_f16`/`v_max_f16` com ordenação inteira de binário16, `-0.0h < +0.0h`, `minNum`/`maxNum` e NaN canônico `0x7e00` (+224/-0) | Não |
| #3123 | shader: lower v_ldexp_f16 in the direct compute path | base `main`: `v_ldexp_f16` com escala inteira exata de binário16, expoente limitado a `[-64, 64]`, denormal promovido sem flush e subnormal com arredondamento par (+262/-0) | Não |

Três dos dez (#3137, #3138, #3139) apontam para bases que não são a `main`. Mesclar qualquer um deles
não move o cursor de commits nem aparece na faixa do `compare`.

## Pull requests mesclados (3)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #3114 | shader: lower s_quadmask_b32 and s_quadmask_b64 in the direct compute path | `2641cc72`, 2026-11-27T16:11:31Z, base `main` | Squash de um commit só. 4 arquivos, +212/-0: o lowering em `gfx1013_direct_compute.inl`, a nota de experimento e o teste (controla DXIL emitido e recusa de par SGPR ímpar, 8 casos b32 e 4 b64 no hardware). O PR cita verificação em RTX 5080 com `2/2 passed` nos presets `dev` e `release` |
| #3130 | shader: preserve helper-lane inputs and texture dimensions on native pipelines | `b80513fd`, 2026-11-27T16:22:59Z, base **`codex/native-graphics-integration`** | +83/-16 em 6 arquivos. Não está na `main`: `b80513fd` não aparece na faixa de 37 commits e o `compare` do cursor ao head não o contém. Conteúdo: liveness de registrador de helper lane, carga exata com máscara de imagem e operações half empacotadas intercaladas, mais as dimensões 2D de entrada e 3D de saída de textura no manifesto de compute |
| #2911 | xmem: make page runs cheaper under Wine | `63a30dc1`, 2026-11-27T17:03:14Z, base `main` | 30 commits de trabalho + revisão, agrupados em três frentes: (1) fallback de split de slot 31 onde o SO esconde o bit de placeholder; (2) cópia de DirectStorage dentro do `xmem.dll`, sob o mutex só quando toca uma run; (3) runs de página — split antes do map, `stable_call` para leitores que não geram fault, retry de fault por stamp do break, joins entre chamadas atrás de `X2W_XMEM_JOIN_CALLS` com teto de 64 páginas, retry de `ReadFile`/`WriteFile` só em disco. O corpo do PR registra que nenhuma execução de título foi feita no head final e que as medições de frames são de um head anterior |

Divergência de contagem, como o procedimento manda registrar: 3 merges na janela contra 37 commits. Dos
37, 2 são merges e 35 não são; 30 desses 35 pertencem ao #2911, 1 é o `2641cc72` do #3114 e os outros 4
são commits de fixture/comentário do próprio #2911. O #3130 não tem commit na `main` porque foi
mesclado na base empilhada.

## Cruzamento com o catálogo

- **D-02 (split de placeholder atinge placeholder do chamador):** o diff **preserva** a garantia.
  `map_split` só divide reserva que o shim possui (`owned_reservation`) e o comentário fixa que
  placeholder do chamador tem de já casar com a view (`06dfe20e`). Não corrige D-02; mantém.
- **D-02, variante nova sob Wine:** o fallback de `17ee827d` mais `1615efc5` cria um caminho em que a
  distinção não existe — quando o SO não reporta o bit `0x100`, uma página reservada de alocação privada
  feita com `VirtualAlloc2(MEM_REPLACE_PLACEHOLDER)` é tratada como placeholder. `f9391cca` registra o
  limite: depois de uma view recusada, o retry página a página corta a página da alocação do chamador, e
  a página comprometida passa a ter alocação começando uma página depois. É a mesma **classe** de D-02
  (escrita fora do contrato do chamador), restrita a Wine, e está documentada como limite conhecido, não
  corrigida. Candidata a linha nova no catálogo.
- **D-03 (rollback do commit de slab deixa slab aberto):** não tocado. `e9e0b441` só devolve o comentário
  de contabilidade ao lugar.
- **D-05 (loads pendentes não unidos nas fronteiras dos braços mascarados):** não tocado.
- **D-09 (`startup_view_heap` lê o trap log sem compartilhamento):** não tocado.
- **D-27 (coordenadas 2D `a16` de `image_sample` lidas como dois binary32):** #3130 mexe em helper lane e
  dimensões de textura no pipeline nativo, que é área vizinha, mas o mecanismo é outro e o merge nem está
  na `main`. Não corrige D-27.
- **#3114:** não há entrada de catálogo para `s_quadmask`; o PR fecha a issue #3112 do próprio
  repositório e é um passo da frente #2935 (lowering direto de compute), a mesma que as horas anteriores
  documentaram fechando recusas uma por uma.

## Confiança

Verificado por mim, pelo diff:

- os 37 hashes, autores, arquivos e contagens de linha vieram da API de commits; li o patch inteiro de
  `06dfe20e` (`map_split`/`exact_placeholder`), de `a4dc3f9c` (teto de 64 páginas e `merge_with_previous`)
  e de `c8391709` (struct `Run{bytes, stamp}`, remoção do anel, novo `retry_remapped_page`);
- pais de `104617d6` (`0c6be971` + `215934bd`), de `63a30dc1` (`2641cc72` + `104617d6`) e de `2641cc72`
  (`300b93f0`); `compare/215934bd...300b93f0` devolve `ahead`, `behind_by: 0`, `ahead_by: 105`, o que
  sustenta a afirmação de que o merge traz conteúdo já documentado;
- base de cada PR aberto veio do campo `base.ref`; `#3130` tem base `codex/native-graphics-integration` e
  seu `merge_commit_sha` `b80513fd` não está entre os 37 commits da faixa;
- a lista de mesclados foi recolhida por `pulls?state=closed&sort=updated` paginando até `updated_at`
  cair abaixo de `last_run`, e reconferida ao fim da coleta: 3 merges, o mais recente `#2911`.

NÃO VERIFICADO:

- qualquer medição do PR: os tempos de DXC, os frames de 168-243 s, os `2/2 passed` do #3114, as 4.985
  linhas de teste, o CI do Windows em `0c6be971` (o head do PR, não o merge `63a30dc1`, que ainda
  incorporou a `main`);
- o diff do #3130 — li só título, corpo, arquivos e base; ele não está na `main`;
- o interior dos cinco commits de revisão: classifiquei-os por arquivo tocado (README, doc, fixture,
  comentário) e pelos títulos, sem ler cada hunk;
- quais páginas o título lê de fato: o PR mede 45.595 de 45.602 runs de uma página, mas isso é contagem
  do autor.
