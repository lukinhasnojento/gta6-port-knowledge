# Xbox2Windows - analise de DeepSeek V4.1 Flash

## A. Identificacao

- **Nome:** Xbox2Windows
- **URL:** https://github.com/R0Y00/Xbox2Windows.git (remoto `origin`, fetch e push)
- **Fork ou original:** original. O historico comeca em `7e28f98c` "Initialize Xbox2Windows project", sem ancestral externo; nao ha sinal de fork (nenhum merge de `upstream` de outro repositorio, apenas `upstream/main` do proprio projeto em mensagens de merge -- `git remote -v` mostra so `origin`).
- **Branch analisada:** `main`, HEAD = `6218beb2` ("Merge pull request #1947 from tedin7/x2w-frontier-triage-endpoint").
- **Commit 0:** `7e28f98c33fda0f15e4c2d632c8c168efef0451a`, 2026-11-21, autor R0Y00, "Initialize Xbox2Windows project".
- **Ultimo commit:** `6218beb279185aa21a7ee97f0970e238739b87f7`, 2026-11-25, autor Royoo.
- **Periodo coberto:** 2026-11-21 a 2026-11-25 (5 dias distintos com commit).
- **Total de commits:** 4076 alcancaveis de HEAD; 4499 alcancaveis de todas as refs; 423 existem apenas em outras branches (nao mergeadas).
- **Commits examinados em profundidade:** 45 do pacote de evidencias (diff integral) + cerca de 38 hashes adicionais abertos com `git show`/`git log -S` (autores das regressoes, correcoes nao empacotadas, pais de merge, commits de revert). Os 4076 estao no indice da secao K.
- **Dias com commit:** 2026-11-21 (414), 2026-11-22 (722), 2026-11-23 (634), 2026-11-24 (1996), 2026-11-25 (310).
- **Merges:** 1823 de 4076 (44,7%).
- **Autores:** 142 identidades, com forte concentracao: Royoo 1803, tedin7 400 (duas identidades: 317 + 83), Vonsek 149, Leyukaka 97, ChazY11 80, rasptum 78, Asim Mirza Ciftler 77, ammar 77, Jess-Doit 79 (75 + 4).
- **Arquivos versionados:** 1471 (`tests/` 1202, `tools/` 127, `src/` 106, `docs/` 18).
- **Modelo responsavel / data:** DeepSeek V4.1 Flash, 2026-11-25.
- **Correcao de premissa:** o contexto que acompanha esta tarefa fala em "5 reverts". O repositorio tem **um** revert de verdade (`git log --all --grep='^Revert'` retorna apenas `22989d61`). Os outros quatro hashes que apareceram numa lista anterior (`b5df5287`, `3d0618a2`, `a72a0929`, `7b9d561c`) sao commits normais cujo **corpo** usa a palavra "revert" ao descrever um experimento local desfeito a mao (por exemplo "a manual mutant ... recorded, reverted" em `b5df5287`). A distincao importa: nao houve 5 reversoes de codigo em main, houve 1.

## B. Resumo tecnico

**Proposito.** Pesquisa de recompilacao estatica Xbox Series X|S (Scarlett, CPU x64 Zen 2 + GPU RDNA 2 "gfx1013") para x64 Windows. O projeto traduz codigo de maquina x64 selecionado para C++ e shaders gfx1013 selecionados para DXIL de desktop, e mede o resultado com fixtures construidos a partir do codigo-fonte, sem distribuir dado de jogo. O estudo de caso declarado no README e GTA VI (Xbox Series X): o README afirma que um EXE de diagnostico modificado, com continuacao automatica de assert, tocou a intro, mostrou texto legivel de setup e chegou ao carregamento num RTX 4060 Laptop; startup do EXE original, menu principal e gameplay seguem **nao verificados**.

**Arquitetura em seis camadas.**

1. **Decodificacao e traducao de shader.** `tools/disasm_rdna2.py` (decoder/disassembler RDNA 2, com comparacao contra LLVM 15), `tools/startup/gfx2hlsl.py` (gfx1013 -> HLSL para o translator de startup, com manifesto de bindings e cache por chave FNV), `tools/pixel_aot.py` (pixel shader -> HLSL AOT), `tools/compute_store_aot.py` (compute stores/atomics), `tools/spirv_address_window.py`. A traducao e conservadora por construcao: forma nao modelada e **recusada por nome**, nunca aproximada.
2. **Traducao de CPU.** `tools/cpu_*_aot.py` (`cpu_switch_aot`, `cpu_seg_inspect_aot`, `cpu_exe_aot`, `cpu_chained_zero`, lifters de x87/SSE/AVX/CFG/jump-table), `tools/pe_map.cpp`, `tools/pe_inventory.py`, `tools/x87_lower.py`. A estrategia e differential testing: o mesmo fixture roda nativo, mapeado e recompilado, e os tres devem concordar byte a byte.
3. **Shims de runtime.** `src/d3d12_xs_shim*`/`src/d3d12x_*` (device/shadow por slot de vtable, IIDs, retorno de struct), `src/era_*` (alias de GPU, heap, paginas fisicas, query, title VA), `src/xmem*.{h,cpp}`, `src/dstorage_rfc1950.cpp`.
4. **Stand-ins de startup.** `src/startup_stubs/` e onde o projeto cresce mais rapido: D3D12 por slot (`d3d12_xs_device.cpp`, `d3d12_draw.inl`, `d3d12_present.inl`), GameInput e script de entrada, XApu/audio, save store, DirectStorage, trap log (`trap.h`), log local (`local_log.h`), assinatura da imagem apresentada.
5. **Orquestracao e medida.** `tools/startup/{debug_launch.py,run.ps1,launch.py,triage_run.py}` e 1202 arquivos em `tests/`, registrados em `CMakeLists.txt` (912 alteracoes no historico). A convencao central e **return code 77 = skip de capacidade** (sem adapter, sem camada de debug, sem compilador).
6. **Documentacao como contrato.** `docs/experiments.md` (1,7 MB) e `docs/startup-runtime.md` registram, por experimento, o que foi medido, em qual adaptador e driver, e o que continua recusado. Praticamente todo commit de comportamento vem acompanhado de diff em doc; onde isso falhou, gerou defeito (D-19).

**Estado atual.** Pesquisa, nao produto. O que esta provado sao propriedades de fixtures e de ferramentas; nenhum resultado aqui estabelece execucao confiavel do titulo. As maiores massas de defeitos corrigidos estao em: tempo de vida de recurso de GPU atraves de command list, posse/rollback em alocadores reserva+commit, semantica de float dependente de compilador, e oraculos de teste que nunca recebiam uma linha propria.

## C. Linha do tempo

**Fase 0 -- bootstrap (2026-11-21, primeiras horas).** `7e28f98c` cria o projeto, `ce546383` publica os experimentos de compatibilidade e o fluxo de contribuicao, `f8951def` abre as tarefas de contribuidor em ingles para o caso GTA VI Xbox Series X. Em horas o repositorio ja tem decoder RDNA 2, lifters de CPU e fixtures: `644aa51c` (operandos SRC0 de 9 bits no `disasm_rdna2`), `d10ab7f4` (rejeicao de operandos EIP-relativos), `4d2835d5` (dispatch por jump table recompilado), `6151171a` (comparacoes float gfx1013 -> HLSL), `b67071a5`/`2ce2b96f` (leitura de secao PE em setores de 0x200 e zero-fill, "como o Windows faz"), `ba5e808e` (xmem.dll de desktop para imports nomeados). O padrao que vai durar todo o projeto aparece aqui: cada capacidade nova entra com um fixture source-built e um controle negativo.

**Fase 1 -- 2026-11-22, amplitude (722 commits).** Correcoes de decodificacao em massa: `eefc3f86` (PR #517: EXP, VINTRP, SMEM, MIMG e literais no `disasm_rdna2`; e o commit que introduz o decode de SMEM sem sign-extend que vira D-07), `c7659e6c` (classificacao VOPC no scanner), `f6e0184b` (MASM macros/prefixos no form coverage). Comecam os primeiros merges de PR externo (#53, #62, #69, #72, #99, #106) -- o projeto ja e multi-autor e o custo de merge passa a ser estrutural.

**Fase 2 -- 2026-11-23, runtime (634 commits).** O shim deixa de ser so traducao e vira runtime: `256d0b39` (paginas fisicas com secoes compartilhadas e uma view para paginas adjacentes), `51e7e618`/`3647cce7`/`c4a42d26` (reserva e fechamento da faixa de GPU configuravel), `2134f9db`/`a472aba2`/`440de93c` (threads do main image, TLS no slot 0, enrolamento explicito em vez de terminar o processo), `419e859f` (recusa de release com participante vivo), `9149048b`/`49776ef6` (emulacao wave64 por group-shared memory), `e28b6077`/`7c0910ab` (compute lists e descriptor heaps do D3D12X).

**Fase 3 -- 2026-11-24, o dia grande (1996 commits, 898 deles merges).** Duas coisas acontecem ao mesmo tempo.

- *Nascem os stand-ins completos*: `e658fe24` (runtime experimental para EXEs de console -- e o commit que introduz `nlt`/`nle`/`ngt`/`nge` como `!(a OP b)` e `o`/`u` como `a == a`/`a != a`, origem de D-06), `d73ab13a`/`658e6eb4` (fila XApu), `8c2d82f6` (script de entrada com espera pela imagem apresentada -- origem de D-01), `3795037e`/`d6c9a773`/`738bfc48`/`50f56203`/`bb35f045` (DirectStorage e o decoder RFC 1950, com comprimento e capacidade separados, overflow de `get_slice`, destino final curto), `e1416e9e` -> `ca92741e` -> `d4de15eb` -> `8b82ced3` -> `bb4272f5` (save store, do zero ao lock cross-process; `ca92741e` introduz D-04), `ef9a0798` (secao "run end-point" do relatorio de triagem -- origem de D-13), `ac539650` (hooks de chamada do debugger viram opt-in).
- *Nascem os alocadores com reserva/commit*: `9727c306` (slabs `SEC_RESERVE`, commit por slot sob demanda -- introduz D-03), `0478a386` (split de placeholder maior -- introduz D-02), `21e8f5c0` corrige os dois no dia seguinte.
- *Nasce o fix de NaN*: `583178d4` reescreve as seis condicoes f32 com o teste de bits `ORDERED` (D-06).
- *Nasce o gate de log saturante*: `f360d30a` troca `InterlockedIncrement <= N` por CAS que para em N (D-10).
- *Nasce a pratica fail-first explicita*: `1f24626b` (17:38:50) commita em main uma mutacao de uma linha, `false && !pass.viewport_set`, para provar que o teste novo de viewport pega a regressao; `22989d61` (17:40:18) a reverte 88 segundos depois.

**Fase 4 -- 2026-11-25, consolidacao (310 commits em main nesse dia, dos quais **214 sao merges** -- 69%; 423 commits seguem em 100+ branches nao mergeadas).** O dia e dominado por merges de PR (1332 assuntos `Merge pull request` no historico inteiro), cada um trazendo um fix pequeno e um controle negativo: `8ec25533` (#2008, retomada de compilacao de shader depois de rodada manifest-only = D-18), `481a1c62` (#2014, `s_pack_hh`), `e63721f9` (#1988, join de loads pendentes nas fronteiras de braco mascarado = D-05), `454af100` (#1976, container de save fechado = D-04), `34472a58` (#1967, XApu vazio = D-12), `889cd3f4` (#1964, debug layer indisponivel = D-23), `1e7436db` (#1970, `pe_inventory` = D-22), `06a494c8` (#1982, CLI do disasm = D-08), `66423e45` (#1980, tokens MASM, com o fix de conflito `14d6023e` = D-16), `442d289e` (#2021, trace-calls opt-in = D-14), `86c225cc` (resolve conflito e re-escopa o skip 77 para hardware). HEAD fecha em `6218beb2` (#1947, endpoint de triagem), que carrega `313803bc` (D-13).

## D. Catalogo de defeitos

Escopo e metodo: os 45 blocos de `_evidencias\Xbox2Windows_evid.md` foram julgados pelo **diff**, nao pela mensagem; cada cadeia foi fechada com `git show` do commit introdutor, `git log <hash>..HEAD -S"<trecho>"` e leitura do arquivo em HEAD. Para blocos de merge sem diff no pacote, o segundo pai foi aberto (`git log -1 --format='%p'`). Nenhum hash abaixo foi inferido. Classes de confianca conforme o SPEC.

- **D-01** Superficie apresentada liberada durante a copia da assinatura
  - Sintomas: o sampler da assinatura de imagem submetia uma copia de 18 linhas da superficie apresentada sem deter referencia propria. Command lists de D3D12 nao retem os recursos que referenciam; o titulo pode liberar a superficie logo apos o Present, e a copia enfileirada passa a ler memoria liberada. Sob debug layer: uso de recurso ja liberado.
  - Causa raiz: o sampler introduzido em `8c2d82f6` nao tomava referencia e nao havia fence que garantisse a vida do recurso.
  - Introduzido em: `8c2d82f6` (2026-11-24).
  - Tentativas de correcao: `2a898640`.
  - Correcao definitiva: `2a898640` (2026-11-24) -- `surface->AddRef()` antes de `ExecuteCommandLists`, `retire_source()` quando o fence completou; se `Signal` falha depois da submissao, `stranded_ = failed_ = true` e tudo fica retido ate o fim do processo.
  - Arquivos: `src/startup_stubs/present_signature_sampler.h`, `present_signature.h`, `d3d12_xs_device.cpp`, `gameinput.cpp`, `tests/startup_present_signature.cpp`.
  - Evidencia: `git show 2a898640 -- src/startup_stubs/present_signature_sampler.h` (as tres linhas citadas); `git log --format=... -- src/startup_stubs/present_signature_sampler.h` mostra so `8c2d82f6`, `f0451ea5` e `2a898640`, e nenhum commit posterior toca o arquivo (a correcao nao foi desfeita).
  - Estado atual: corrigido. O controle `startup_present_signature_unheld_source_control_warp` so passa quando a debug layer acusa a superficie liberada -- e a prova de que o caminho positivo realmente segura.
  - Confianca: CONFIRMADA.

- **D-02** Split de placeholder atinge placeholder do chamador
  - Sintomas: mapear uma view menor dentro de um placeholder maior pertencente ao **chamador** dividia e liberava um placeholder que o shim nao possuia, destruindo o estado de reserva do chamador; o erro reportado tambem podia ser o da primeira tentativa mesmo quando o retry falhava por outro motivo.
  - Causa raiz: `0478a386` removeu a checagem `owned_reservation(address)` e passou a chamar `VirtualFree(target, bytes, MEM_RELEASE | MEM_PRESERVE_PLACEHOLDER)` em qualquer placeholder maior.
  - Introduzido em: `0478a386` (2026-11-24).
  - Tentativas de correcao: `21e8f5c0`.
  - Correcao definitiva: `21e8f5c0` (2026-11-25): exige `reservation != memory.reservations.end()`, exige que `bytes <= reservation->second - (address - reservation->first)` e devolve o resultado do retry.
  - Arquivos: `src/xmem_mapped_pages.cpp`, `tests/xmem_mapped_pages_commit_check.py`.
  - Evidencia: diff de `0478a386` (`if (VirtualFree(...) && map()) return true;` sem ownership) contra `git show 21e8f5c0 -- src/xmem_mapped_pages.cpp` (o `map_split` final).
  - Estado atual: corrigido. Placeholder do chamador precisa ter exatamente o tamanho da view (ver G-03).
  - Confianca: CONFIRMADA.

- **D-03** Rollback do commit de slab deixa slab aberto
  - Sintomas: quando o commit do primeiro slot falhava, a chamada retornava erro mas o slab ja inserido permanecia aberto e contabilizado -- contrariando o "leaving no state changed" da propria mensagem.
  - Causa raiz: `9727c306` chamava `take_fresh` (que ja registra o slab) e, ao falhar `commit_slot`, chamava `unwind_fresh`, que so ajusta `fresh_used_` e nao fecha o handle nem remove o slab.
  - Introduzido em: `9727c306` (2026-11-24).
  - Tentativas de correcao: `21e8f5c0`.
  - Correcao definitiva: `21e8f5c0` (2026-11-25): o commit do primeiro slot passa para dentro de `make_slab`, que fecha o handle em falha; `first_slot_committed` evita commitar duas vezes.
  - Arquivos: `src/xmem_mapped_pages.cpp`, `src/xmem_slab_slots.h`, `tests/xmem_mapped_pages_commit_check.py`.
  - Evidencia: diff de `21e8f5c0` (bloco `make_slab` + `first_slot_committed`) e o seam de teste `X2W_XMEM_TEST_COMMIT_FAILURE`, que recusa o primeiro commit sem consumir commit do sistema.
  - Estado atual: corrigido.
  - Confianca: CONFIRMADA.

- **D-04** Container de save aceita update depois de fechado
  - Sintomas: `begin_update` iniciava um update em container sem referencia aberta enquanto um update anterior mantinha o nome na tabela; o titulo recebia slot valido para container fechado.
  - Causa raiz: `ca92741e` passou a manter `names_` durante updates e a reusar slots por refcount, mas `begin_update` testava `names_[container].empty()` em vez de `refs_[container] == 0`.
  - Introduzido em: `ca92741e` (2026-11-24).
  - Tentativas de correcao: `2260b7a4`.
  - Correcao definitiva: `2260b7a4` (2026-11-25), com o teste "a closed container cannot start an update while another update keeps its entry".
  - Arquivos: `src/startup_stubs/save_store.h`, `tests/startup_gamesave_persist.cpp`.
  - Evidencia: `git show 2260b7a4 -- src/startup_stubs/save_store.h` (a linha trocada) e o arquivo em HEAD (`refs_[container] == 0`); `git log -S'refs_[container]' -- src/startup_stubs/save_store.h` nao retorna nada depois do fix.
  - Estado atual: corrigido.
  - Confianca: CONFIRMADA.

- **D-05** Loads pendentes nao unidos nas fronteiras dos bracos mascarados
  - Sintomas: em regiao if/else com skip em um braco, o estado de loads pendentes e `sample_dest` no fechamento voltava ao snapshot do **opener** em vez do snapshot do inicio do ultimo braco. Efeito: num braco que executou e ja resolveu seus waits, o analisador voltava a trata-los como pendentes (recusa ou hazard espurio); no braco que pulou, o estado restaurado podia nao corresponder ao caminho realmente tomado.
  - Causa raiz: `6b8ad31c` introduziu o segundo braco (else) e, na transicao `else`, fez `region["skip"] = None` sem atualizar o snapshot `outstanding`; `close_region` restaurava `region["outstanding"]` (do opener).
  - Introduzido em: `6b8ad31c` (2026-11-24).
  - Tentativas de correcao: `05ef4d92`.
  - Correcao definitiva: `05ef4d92` (2026-11-25): renomeia para `arm_outstanding`, atualiza na transicao do else e so faz o join quando `region["skip"] is not None` (o braco final tem skip).
  - Arquivos: `tools/pixel_aot.py`, `tests/pixel_aot_check.py`.
  - Evidencia: `git show 05ef4d92 -- tools/pixel_aot.py`; o teste novo exige "Constant read before load completion" quando o braco pode pular o wait.
  - Estado atual: corrigido; cobre `then`, `else`, `both` e os dois restores.
  - Confianca: CONFIRMADA para o mecanismo; PROVAVEL para o efeito em shader real (nao ha caso de titulo citado).

- **D-06** f32 `nlt/nle/ngt/nge/o/u` errados sob FXC e DXC sem `-Gis`
  - Sintomas: com NaN, `nlt/nle/ngt/nge` davam 0 onde a ISA da 1; `u` virava constante 0 e `o` constante 1.
  - Causa raiz: o lowering emitia `!(a OP b)` e `a != a`/`a == a`; FXC sem `/Gis` dobra `!(a < b)` em `ge a, b` e DXC dobra `a != a`.
  - Introduzido em: `e658fe24` (2026-11-24, stand-ins iniciais; `git log -S"'nlt'" -- tools/startup/gfx2hlsl.py` aponta para ele).
  - Tentativas de correcao: `583178d4`.
  - Correcao definitiva: `583178d4` (2026-11-24): `!(ORDERED && a OP b)`, `o` = `ORDERED`, `u` = `!ORDERED`, com `ORDERED` sendo o teste de bits das duas palavras de fonte que `lg` ja usava desde #1920.
  - Arquivos: `tools/startup/gfx2hlsl.py`, `tests/startup_gfx2hlsl_f32_nan_check.py`, `tests/startup_gfx2hlsl_f32_lg_check.py`, `docs/experiments.md`.
  - Evidencia: diff das seis condicoes e a remocao de `nlt/nle/ngt/nge/o/u` de `UNCHANGED` no check de `lg`; `git log '583178d4..HEAD' -S'ORDERED' -- tools/startup/gfx2hlsl.py` vazio (nao regrediu).
  - Estado atual: corrigido; o teste avalia 510 formas e desenha 13 condicoes com FXC e DXC.
  - Confianca: CONFIRMADA.

- **D-07** Offset imediato SMEM impresso sem sinal
  - Sintomas: `-1` codificado aparecia como `+0x1fffff`.
  - Causa raiz: `disasm_rdna2` mascarava `OFFSET[20:0]` e imprimia unsigned, embora em GFX9+/RDNA 2 o campo seja um byte offset signed de 21 bits.
  - Introduzido em: `eefc3f86` (2026-11-22, PR #517).
  - Tentativas de correcao: `83bb752d`.
  - Correcao definitiva: `83bb752d` (2026-11-24): `if imm_offset & 0x100000: imm_offset -= 0x200000` e `f"-0x{-imm_offset:x}"`, nos dois `forms` (offset puro e `soffset + offset:`).
  - Arquivos: `tools/disasm_rdna2.py`, `tests/rdna2_tools_check.py`.
  - Evidencia: quatro vetores de regressao (-0x1, -0x100000, 0xfffff, `s0 offset:-0x1`); `git log '83bb752d..HEAD' -S'0x200000' -- tools/disasm_rdna2.py` vazio.
  - Estado atual: corrigido.
  - Confianca: CONFIRMADA.

- **D-08** CLI do `disasm_rdna2` ignora `--output` com `--hex`, vaza traceback e sai 0 em alvo ausente
  - Sintomas: `--hex ... --output` imprimia so em stdout; hex de tamanho impar ou nao-hex levantava `ValueError`; alvo que nao era arquivo nem diretorio era pulado com exit 0.
  - Causa raiz: o caminho de hex retornava antes do writer compartilhado; `bytes.fromhex` sem tratamento; `else` ausente no loop de alvos.
  - Introduzido em: NAO IDENTIFICADO (nasceu com o proprio CLI, no inicio do historico; nao isolei o commit).
  - Tentativas de correcao: `20fa94a9`.
  - Correcao definitiva: `20fa94a9` (2026-11-24): `parser.error` para `--hex` (exit 2), `error: <target>: not found` em stderr, exit 1 apos processar os demais, hex incluido em `all_listings`.
  - Arquivos: `tools/disasm_rdna2.py`, `tests/rdna2_tools_check.py`.
  - Evidencia: `test_cli_hex_output_and_input_errors` cobre os tres casos (stdout com `--output`, exit 2 sem Traceback, exit 1 com "not found").
  - Estado atual: corrigido.
  - Confianca: CONFIRMADA.

- **D-09** `startup_view_heap` le o trap log sem compartilhamento
  - Sintomas: `startup_view_heap_hardware` reportava "shortcut logged -1 times" e falhava exatamente nos adapters que recusam o heap completo; WARP faz skip, entao o CI nao via.
  - Causa raiz: desde #1757 (`99f097ad`, merge `447c38df`) os stand-ins mantem `X2W_TRAP_LOG` aberto para append com `FILE_SHARE_READ|WRITE|DELETE`; o teste usava `_wfopen_s`, que abre sem share, e o open falhava enquanto o handle existia.
  - Introduzido em: `99f097ad` (2026-11-24) -- a mudanca que passou a manter o handle aberto.
  - Tentativas de correcao: `2c1dcdd6`.
  - Correcao definitiva: `2c1dcdd6` (2026-11-24): `_wfsopen(path, L"rb", _SH_DENYNO)`.
  - Arquivos: `tests/startup_view_heap_check.cpp`.
  - Evidencia: diff de uma linha mais o comentario que cita `src/startup_stubs/trap.h`; o arquivo em HEAD mantem `CreateFileW(..., FILE_APPEND_DATA, FILE_SHARE_READ|FILE_SHARE_WRITE|FILE_SHARE_DELETE, ...)`.
  - Estado atual: corrigido.
  - Confianca: CONFIRMADA.

- **D-10** Contador de "primeiras N chamadas" estoura 2^31 e volta a logar
  - Sintomas: o gate de log reabria depois de ~2^31 incrementos; um consumidor que consulta status em loop dispara o caso.
  - Causa raiz: `InterlockedIncrement(&counter) <= N` conta todas as chamadas; o contador vira negativo e a comparacao volta a ser verdadeira.
  - Introduzido em: `e658fe24` (2026-11-24, stand-ins iniciais) -- o padrao aparece em `xcrdapi.cpp`, `xg_xs.cpp`, `runtime_proxy.cpp`, `d3d12_draw.inl` e no dump de pipeline.
  - Tentativas de correcao: `f360d30a`.
  - Correcao definitiva: `f360d30a` (2026-11-24): `x2w_local::first_calls` com CAS que para em `limit`; os tres contadores que tambem alimentam estatistica (`render_draws`, `shown_frames`, contagens por slot do runtime proxy) continuam contando e passam a exigir valor positivo.
  - Arquivos: `src/startup_stubs/local_log.h`, `xcrdapi.cpp`, `xg_xs.cpp`, `runtime_proxy.cpp`, `d3d12_draw.inl`, `d3d12_xs_device.cpp`, `tests/startup_local_log_check.cpp`.
  - Evidencia: o `first_calls` em HEAD (laco `InterlockedCompareExchange` com `while (seen < limit)`); o controle `startup_local_log_wrap_control` exige a string "FAIL first_calls passes no call at the wrap point".
  - Estado atual: corrigido. Sobra o padrao antigo em dois pontos protegidos por guarda (ver G-06).
  - Confianca: CONFIRMADA (mecanismo); frequencia real em titulo NAO VERIFICADA.

- **D-11** Testes de hardware sem `SKIP_RETURN_CODE 77`
  - Sintomas: PC sem adapter D3D12 reportava **Failed** em vez de **Skipped** para `wave64_emulate`, `startup_cb_blend`, `startup_cb_blend_swapped`, `startup_composite_provenance` e `compute_store_aot_scratch_base`.
  - Causa raiz: a propriedade nao foi registrada nesses testes, embora `offscreen_check` retorne 77 sem adapter.
  - Introduzido em: NAO IDENTIFICADO (omissao distribuida ao longo de varios PRs).
  - Tentativas de correcao: `0b649c69`, depois `86c225cc`.
  - Correcao definitiva: `0b649c69` (2026-11-24) para os testes existentes; `86c225cc` (2026-11-25) reaplica a propriedade **so nas variantes `_hardware`** (ver H-03).
  - Arquivos: `CMakeLists.txt`.
  - Evidencia: o diff de `0b649c69` (6 insercoes, 4 remocoes) e, em HEAD, `set_tests_properties(startup_cb_blend_hardware startup_cb_blend_swapped_hardware startup_composite_provenance_hardware startup_range_provenance_hardware PROPERTIES SKIP_RETURN_CODE 77)` com as variantes `_warp` sem a propriedade.
  - Estado atual: corrigido para hardware; WARP fica sem skip por decisao explicita (WARP sempre acha adapter).
  - Confianca: CONFIRMADA.

- **D-12** `XApuDequeueResult` em fila vazia retorna `E_FAIL`
  - Sintomas: fila vazia ou drenada devolvia `E_FAIL`, sem distinguir "nenhum resultado ainda"; o chamador nao recebia o codigo especifico.
  - Causa raiz: o stub usava `hr_fail` (0x80004005) no caminho sem `pending_results`.
  - Introduzido em: NAO IDENTIFICADO (nasceu com o stub XApu; nao isolei o commit).
  - Tentativas de correcao: `b3a90888`.
  - Correcao definitiva: `b3a90888` (2026-11-25): `XAPU_E_PENDING_RESULTS = 0x8AC81001`; o teste envia buffer sentinela e exige o codigo novo com o buffer preservado.
  - Arquivos: `src/startup_stubs/xapu.cpp`, `tests/startup_xapu_queue_check.cpp`, `docs/startup-runtime.md`.
  - Evidencia: `git show b3a90888 -- src/startup_stubs/xapu.cpp` (a constante substitui `hr_fail`).
  - Estado atual: corrigido.
  - Confianca: CONFIRMADA.

- **D-13** Endpoint de triagem conta first-chance step-over como "debugger stop"
  - Sintomas: o relatorio exibia "First/Last stop event" para o breakpoint de loader (0x80000003) que `debug_launch.py` pula, sugerindo uma parada do debugger que nunca ocorreu.
  - Causa raiz: `ef9a0798` derivava os stop events de **todo** evento de excecao em `events.txt`, incluindo os que o launcher step over.
  - Introduzido em: `ef9a0798` (2026-11-24).
  - Tentativas de correcao: `313803bc`.
  - Correcao definitiva: `313803bc` (2026-11-25): o launcher grava `debugger stop 0x... first|second-chance`, e o relatorio separa "Confirmed debugger stop" de "First/Last observed exception" + "Observed exceptions".
  - Arquivos: `tools/startup/triage_run.py`, `tools/startup/debug_launch.py`, `tests/startup_triage_run_check.py`, `tests/startup_debug_check.py`, `docs/startup-runtime.md`.
  - Evidencia: em HEAD, `tools/startup/triage_run.py:490` ("Confirmed debugger stop") e `:493` ("Observed exceptions"); o teste novo exige `debugger stop 0x80000003 first-chance` exatamente uma vez.
  - Estado atual: corrigido; logs antigos sem `.out` do launcher nao tem parada confirmada (G-04).
  - Confianca: CONFIRMADA.

- **D-14** Call hooks do debugger armados por padrao
  - Sintomas: toda chamada hookada de gaming runtime ou movie player parava o processo sob o debugger; como file tracing e padrao (o modo offline precisa dele), uma rodada normal ficava lenta ou travava.
  - Causa raiz: os dois conjuntos de hooks eram armados sempre que file tracing estava ligado.
  - Introduzido em: NAO IDENTIFICADO (o commit que acoplou os hooks ao file tracing nao esta isolado no pacote).
  - Tentativas de correcao: `ac539650` (merge `442d289e`, #2021).
  - Correcao definitiva: `ac539650` (2026-11-24): hooks de runtime e movie player so com `--trace-calls 1` (`run.ps1 -TraceCalls`); hooks de arquivo, negacao offline, continuacao de assert e trap dumps inalterados; o resumo do launcher declara `call tracing: on|off`.
  - Arquivos: `tools/startup/debug_launch.py`, `tools/startup/run.ps1`, `tests/startup_debug_check.py`, `tests/startup_debug_calls.cpp`, `tests/startup_debug_probe.cpp`, `CMakeLists.txt`.
  - Evidencia: o teste copia a DLL source-built sob o nome do modulo de movie player e conta 7 chamadas: 0 por padrao, exatamente 7 com a flag -- distingue "nao armou" de "armou e nao contou".
  - Estado atual: corrigido; o relatorio de movie player so existe com a flag.
  - Confianca: CONFIRMADA.

- **D-15** `CreateCommandSignature` sequestra qualquer tipo acima de `DISPATCH_MESH` e le ponteiros sem validar
  - Sintomas: tipos indiretos futuros do desktop eram tratados como "console-only"; `desc`/`pArgumentDescs` podiam ser lidos sem checagem e `out` escrito sem checagem.
  - Causa raiz: `71d4bff1` comparava `Type > D3D12_INDIRECT_ARGUMENT_TYPE_DISPATCH_MESH` e devolvia placeholder **antes** do runtime, sem `readable`/`writable`.
  - Introduzido em: `71d4bff1` (2026-11-24).
  - Tentativas de correcao: `e50a72bc`.
  - Correcao definitiva: `e50a72bc` (2026-11-25): o filtro reconhece apenas `0x69`; o device valida `readable`/`writable`; o fallback para tipos desconhecidos so ocorre **depois** de o desktop recusar o pedido real.
  - Arquivos: `src/startup_stubs/command_signature_filter.h`, `d3d12_xs_device.cpp`, `tests/startup_command_signature.cpp`, `tests/startup_command_signature_filter_check.cpp`.
  - Evidencia: `has_console_indirect_arg_type` em HEAD tem assinatura sem `last_desktop_type` e compara `== 0x69`; `d3d12_xs_device.cpp:914` exige `output && readable_args && ... && writable`.
  - Estado atual: corrigido.
  - Confianca: CONFIRMADA.

- **D-16** Fixture MASM isolado trocado por programa de pseudo-ops
  - Sintomas: o caso de parsing isolado passou a medir o programa com `REPT`/`IFIDNI` em vez do fixture original; a cobertura do caso isolado caiu sem falhar nada.
  - Causa raiz: `5b2dadd7` reutilizou a variavel `asm` para o programa de pseudo-ops antes de `fc.measure({}, sample_repo(tmp, {"branch": asm}))`.
  - Introduzido em: `5b2dadd7` (2026-11-25).
  - Tentativas de correcao: `14d6023e`.
  - Correcao definitiva: `14d6023e` (2026-11-25): renomeia a segunda cadeia para `pseudo_ops_asm`, deixando `asm` para o fixture isolado.
  - Arquivos: `tests/form_coverage_check.py`.
  - Evidencia: `git show 14d6023e` (3 linhas) e o arquivo em HEAD, onde `fc.measure(..., {"branch": asm})` volta a receber o fixture de `PROC FRAME`/`IFDEF`.
  - Estado atual: corrigido; `14d6023e` e o segundo pai do merge `66423e45` (#1980), isto e, o fix entrou junto com o PR que causou o problema.
  - Confianca: CONFIRMADA.

- **D-17** Caso de heap em placed-buffer assumia prefixo reservado
  - Sintomas: o caso "heap block" esperava pagina base reservada e recusava buffers no prefixo com expectativa fixa; se o alocador commitasse o prefixo, a expectativa mascarava regressao.
  - Causa raiz: premissa sobre o layout do heap que a API nao garante.
  - Introduzido em: NAO IDENTIFICADO.
  - Tentativas de correcao: `6ef541f3` (merge `c514f20b`, #1937).
  - Correcao definitiva: `6ef541f3` (2026-11-24): `place(..., -1, ...)` compara as duas medicoes sem fixar expectativa e o caso deterministico ("committed suffix after a reserved prefix") passa a ser construido com `VirtualAlloc`.
  - Arquivos: `tests/startup_placed_buffer_range_check.cpp`.
  - Evidencia: `git show 6ef541f3 --stat` (22 insercoes, 11 remocoes, so teste).
  - Estado atual: corrigido.
  - Confianca: CONFIRMADA.

- **D-18** Compilacao de shader nao retomada depois de rodada manifest-only
  - Sintomas: depois de `X2W_FXC=` (manifest only), rodadas seguintes com fxc continuavam sem `.cso`, porque `--pending` pulava qualquer request com `<key>.txt`; e podia publicar bytecode de um par que falhou ao lado de manifesto antigo.
  - Causa raiz: o criterio de completude era so a existencia do `.txt`; a publicacao escrevia `.cso` direto, antes do manifesto.
  - Introduzido em: NAO IDENTIFICADO (nasceu com o modo manifest-only; nao isolei o commit).
  - Tentativas de correcao: `b87a99a3` (merge `8ec25533`, #2008).
  - Correcao definitiva: `b87a99a3` (2026-11-25): compila os dois estagios em `tempfile.TemporaryDirectory`, exige `has_bytecode` nos dois antes de publicar, retira o manifesto antigo quando as bindings mudam, remove o `.fail` no sucesso; o criterio de retomada passa a ser `.txt` + ambos `.cso`, salvo manifest-only.
  - Arquivos: `tools/startup/gfx2hlsl.py`, `tests/startup_translator_check.py`, `docs/startup-runtime.md`, `docs/experiments.md`.
  - Evidencia: `git show b87a99a3 -- tools/startup/gfx2hlsl.py` (bloco de staging, `withdraw the old manifest`, `manifest_only = os.environ.get('X2W_FXC') == ''`).
  - Estado atual: corrigido.
  - Confianca: CONFIRMADA.

- **D-19** Documentacao do ciclo de `.fail` ficou obsoleta
  - Sintomas: a doc dizia que um `.fail` nunca era removido; depois de `b87a99a3`, com fxc disponivel, um request que traduz perde o `.fail`.
  - Causa raiz: mudanca de comportamento sem atualizar a secao "Cost of the loop".
  - Introduzido em: `b87a99a3` (2026-11-25).
  - Tentativas de correcao: `5b29fe0f` (merge `e1aa07e0`, #2043).
  - Correcao definitiva: `5b29fe0f` (2026-11-25): a frase passa a exigir `<key>.txt` + ambos `.cso` (salvo `X2W_FXC=""`) e explica que, em manifest-only, o `.fail` antigo permanece.
  - Arquivos: `docs/startup-runtime.md`.
  - Evidencia: os dois commits sao **paralelos** (`git merge-base --is-ancestor` falha nos dois sentidos); `b87a99a3` tem data 01:57+0300 e `5b29fe0f` 00:34+0100, isto e, a doc foi escrita 37 min depois do codigo, num ramo separado, com a mensagem "match the iteration section to the retry and .fail rules on main".
  - Estado atual: corrigido.
  - Confianca: PROVAVEL (a cadeia e inferida pela ordem temporal e pelo texto novo descrever exatamente o comportamento introduzido; nao ha referencia cruzada entre os dois commits).

- **D-20** Tabela `SHADER_REASONS` nunca exercitada pela suite
  - Sintomas: um pattern que nunca casa, ou um label renomeado, dobrava uma causa conhecida em `other` sem falhar em lugar nenhum.
  - Causa raiz: o teste nao alimentava `SHADER_REASONS` com uma linha propria nem verificava a cobertura reversa (toda entrada precisa de amostra).
  - Introduzido em: `71ec5842` (2026-11-24, o commit que cria a tabela).
  - Tentativas de correcao: `13745b1b`.
  - Correcao definitiva: `13745b1b` (2026-11-25): 16 amostras, cada uma confrontada com a tabela, conferencia de que todo indice foi coberto e fallback obrigatorio para `other`.
  - Arquivos: `tests/startup_triage_run_check.py`.
  - Evidencia: `git show 13745b1b` (40 linhas em um arquivo, nenhum codigo de produto).
  - Estado atual: corrigido para as entradas atuais; entrada nova sem amostra passa a falhar.
  - Confianca: CONFIRMADA.

- **D-21** Checagem device-free das selecoes SDWA `movrel` nao detectava selecao removida
  - Sintomas: remover a selecao de fonte ainda passava no check sem GPU; so o draw pegava a regressao.
  - Causa raiz: o check verificava apenas que `SDWA_MOVREL_BODY` traduzia, sem fixar as expressoes geradas.
  - Introduzido em: `bc03f191` (2026-11-24, o lowering que criou o corpo).
  - Tentativas de correcao: `2e5fdca4`.
  - Correcao definitiva: `2e5fdca4` (2026-11-24): `SDWA_MOVREL_PINS` fixa expressao e contagem (uma por move estatico, uma por destino alcancavel de um dinamico); cinco mutacoes de uma linha passam a falhar.
  - Arquivos: `tests/pixel_aot_check.py`, `docs/experiments.md`.
  - Evidencia: `git show 2e5fdca4 -- tests/pixel_aot_check.py` (10 padroes com contagem) e o texto do commit "tools/pixel_aot.py is unchanged".
  - Estado atual: corrigido.
  - Confianca: CONFIRMADA.

- **D-22** `pe_inventory` quebra com inventario malformado e nao valida a CLI
  - Sintomas: baseline com schema valido mas `imports`/`images` de tipo errado gerava `KeyError`/`TypeError`; arquivo inexistente e saida invalida vazavam traceback; o temporario podia ficar no disco em falha.
  - Causa raiz: `changes()` confiava na estrutura; `main()` sem captura de `OSError`/`ValueError`/`JSONDecodeError` e sem limpeza do temporario.
  - Introduzido em: NAO IDENTIFICADO.
  - Tentativas de correcao: `65d07996` (suite + hardening) e `9a2b9cd8` ("Validate inventory entries before comparing baselines"), ambos no PR #1970 (merge `1e7436db`).
  - Correcao definitiva: `9a2b9cd8` (2026-11-25) -- `git log -1 --format='%p' 1e7436db` mostra `2deaa030 9a2b9cd8`, e `65d07996` e ancestral do primeiro pai.
  - Arquivos: `tools/pe_inventory.py`, `tests/pe_inventory_cli_check.py`, `CMakeLists.txt`.
  - Evidencia: validacao de dict/list e de campos string; `parser.exit(1, ...)`; `tmp_path.unlink(missing_ok=True)`; suite com PE sintetico construido em `.rdata`.
  - Estado atual: corrigido.
  - Confianca: CONFIRMADA.

- **D-23** Draw `attr0.z/w` falhava em host sem debug layer
  - Sintomas: em host sem Graphics Tools, `pixel_aot_attribute_sampled_{hardware,warp}` falhava no primeiro draw mesmo com `max_absolute_error=0`, porque exigia a linha `debug_errors=0 debug_warnings=0`.
  - Causa raiz: sem a camada, o renderer imprime `debug_layer=unavailable` e nenhum contador; o teste tratava a ausencia como falha.
  - Introduzido em: NAO IDENTIFICADO.
  - Tentativas de correcao: `d28732b6` (merge `889cd3f4`, #1964).
  - Correcao definitiva: `d28732b6` (2026-11-24): `debug_status()` aceita `unavailable` apenas sem contadores, exige zero com a camada ligada e recusa qualquer outra saida; seis casos sinteticos sao fixados antes de qualquer trabalho de device.
  - Arquivos: `tests/pixel_attribute_sampled_check.py`.
  - Evidencia: `git show d28732b6` (o helper e a tabela de seis casos).
  - Estado atual: corrigido.
  - Confianca: CONFIRMADA.

- **D-24** `<initializer_list>` ausente no teste de tipo de heap
  - Sintomas: a compilacao dependia de inclusao transitiva do cabecalho.
  - Causa raiz: uso direto do tipo sem include proprio.
  - Introduzido em: NAO IDENTIFICADO.
  - Correcao definitiva: `5d6d5f52` (2026-11-24), uma linha.
  - Arquivos: `tests/startup_descriptor_heap_type_check.cpp`.
  - Estado atual: corrigido.
  - Confianca: PROVAVEL (o diff prova o include; nao ha log de falha de compilacao no repositorio).

- **D-25** `seg_inspect_check` decodifica stderr do compilador sem `errors="replace"`
  - Sintomas: em falha do compilador com bytes fora de UTF-8, `subprocess.run(..., text=True)` pode levantar `UnicodeDecodeError` e esconder o diagnostico original.
  - Causa raiz: falta de `errors="replace"` na captura.
  - Introduzido em: NAO IDENTIFICADO.
  - Correcao definitiva: `6855f425` (2026-11-25), junto com a documentacao do perfil fixo de GDT (LAR/LSL/VERR/VERW para `0x20`/`0x28`/`0x30`).
  - Arquivos: `tests/cpu/seg_inspect_check.py`, `tools/cpu_seg_inspect_aot.py`, `docs/experiments.md`.
  - Estado atual: corrigido.
  - Confianca: PROVAVEL.

- **D-26** Viewport/scissor default confundido com o estado do titulo (defeito introduzido e corrigido no mesmo dia, em torno do revert)
  - Sintomas: uma lista que nunca chamou `RSSetViewports` recebia o default full-target uma vez, e `pass.viewport_set` era marcado como `true` por esse default. Quando o render target 0 mudava de tamanho (`OMSetRenderTargets` para outro alvo), o default **nao** era regenerado: o viewport/scissor antigo continuava valendo e o draw cobria so parte do alvo novo (ou recortava demais).
  - Causa raiz: `ffb70f6a` usou a mesma flag (`viewport_set`) para "o titulo definiu" e "o default ja foi encaminhado". `07f2464f` separa os dois: `default_viewport_w/h` guardam o tamanho ja encaminhado e so o titulo seta `viewport_set`.
  - Introduzido em: `ffb70f6a` (2026-11-24 17:31:05 +0100, "fix: default viewport and scissor for translated draws").
  - Tentativas de correcao: nenhuma intermediaria funcional; `1f24626b`/`22989d61` sao experimento e revert (ver D-29).
  - Correcao definitiva: `07f2464f` (2026-11-24 17:53:01 +0100), 22 minutos depois, com o caso "grow" adicionado a `startup_stream_viewport`.
  - Arquivos: `src/startup_stubs/d3d12_draw.inl`, `src/startup_stubs/d3d12_xs_device.cpp`, `tests/startup_stream_draw.cpp`, `tests/startup_stream_setup.cpp`, `tests/startup_stream_viewport_check.py`.
  - Evidencia: `git show 07f2464f -- src/startup_stubs/d3d12_draw.inl` troca `pass.viewport_set = true` por `pass.default_viewport_w/h = ...` e adiciona `default_viewport_w/h` em `struct Pass` com o comentario "Kept apart from the title flags above so a later render target 0 of another size gets a fresh default"; o estado em HEAD mantem exatamente esse desenho.
  - Estado atual: corrigido. O default e regenerado por tamanho, nao por identidade de recurso (ver G-10).
  - Confianca: CONFIRMADA.

- **D-27** Coordenadas 2D `a16` de `image_sample`/`image_sample_lz` lidas como dois registradores binary32
  - Sintomas: nas formas `a16` (duas coordenadas binary16 empacotadas num unico VGPR), o lowering lia `p` e o VGPR seguinte como dois floats -- UV errado, amostragem no texel errado. Como o cache de shader nao mudava de revisao, bytecode pre-fix podia ser reutilizado.
  - Causa raiz: o decoder marcava `a16` nos mods, mas o caminho de sample em `gfx2hlsl.py` nao validava modificadores nem desempacotava metades (`float2(F(cs[0]), F(cs[1]))` incondicional); nao havia verificacao de contagem de registradores de endereco por forma.
  - Introduzido em: o commit que passou a aceitar `a16` no caminho de sample (NAO IDENTIFICADO com precisao; `56bd3bd1` fez o equivalente para `image_load`, e o caminho de sample ficou para tras).
  - Tentativas de correcao: `c3c3ab3c`.
  - Correcao definitiva: `c3c3ab3c` (2026-11-24): decodifica **um** VGPR de endereco para 2D `a16`, sinaliza `invalid_nsa`, emite `float2(f16tof32(p), f16tof32(p >> 16))`, recusa modificadores nao modelados/`dmask` vazio/contagem errada de registradores e **avanca a revisao do cache para `x2w-startup-shader-6`** em `gfx2hlsl.py` e em `d3d12_draw.inl`.
  - Arquivos: `tools/startup/gfx2hlsl.py`, `tools/disasm_rdna2.py`, `src/startup_stubs/d3d12_draw.inl`, `tests/startup_image_sample_a16_check.py`, `docs/experiments.md`, `docs/startup-runtime.md`.
  - Evidencia: `git show c3c3ab3c -- tools/startup/gfx2hlsl.py` (o `packed_2d` e a troca de `revision`); em HEAD, `d3d12_draw.inl:201` e `gfx2hlsl.py:2176` concordam em `x2w-startup-shader-6`; o teste roda seis controles mutantes do texto pre-fix e exige mismatch de texel.
  - Estado atual: corrigido para 2D; `a16` em 3D/2D-array e `image_sample_l` com `a16` seguem recusados por decisao (G-11).
  - Confianca: CONFIRMADA para o fix e a revisao de cache; PROVAVEL para o sintoma pre-fix (a descricao "p and the next VGPR" vem da doc adicionada no proprio commit, nao de um caso medido).

- **D-28** Conversao UNORM/SNORM do store formatado usava `round()` (ties-to-even) em vez da regra do Direct3D
  - Sintomas: em empate exato (o caso fixado e um binary32 cujo produto por 255 da exatamente 2.5), o store formatado gravava 2, porque `round()` empata para o par; a regra do Direct3D manda 3 (afasta de zero). O desvio e de 1 LSB e so em empates exatos.
  - Causa raiz: `descriptor_store_hlsl()` emitia `(uint)round(scaled)` e `(uint)(int)round(scaled)`; `round()` em HLSL e ties-to-even, enquanto a conversao FLOAT->UNORM/SNORM do Direct3D e "somar/subtrair 0.5 e truncar".
  - Introduzido em: `6177df41` (2026-11-24, "compute: lower buffer_store_format_* from the descriptor format").
  - Tentativas de correcao: `f37fe7b8`.
  - Correcao definitiva: `f37fe7b8` (2026-11-24): `floor(scaled + 0.5)` e `scaled >= 0 ? floor(scaled+0.5) : ceil(scaled-0.5)`; o teste ganha `UNORM8_TIE = 0x3C20A0A1` e um modelo com as duas regras (`_round_half_away` vs `_round_nearest_even`), de modo que trocar uma pela outra falha.
  - Arquivos: `tools/compute_store_aot.py`, `tests/compute_store_check.py`, `docs/experiments.md`.
  - Evidencia: `git show f37fe7b8 -- tools/compute_store_aot.py` (duas linhas de HLSL gerado + o comentario que declara o tie-break da unidade RDNA como nao verificado).
  - Estado atual: corrigido segundo a regra do Direct3D; o comportamento real da unidade de formato da RDNA 2 continua **nao verificado** (G-12).
  - Confianca: CONFIRMADA para a mudanca de codigo; a semantica de `round()` do FXC/DXC e a escolha "correta" para gfx1013 sao PROVAVEL/HIPOTESE declarada no proprio diff.

- **D-29** Mutacao temporaria commitada em `main` (e revertida 88 s depois)
  - Sintomas: `src/startup_stubs/d3d12_draw.inl` passou a conter `if (false && !pass.viewport_set)`, desligando o viewport default; todo draw traduzido numa lista sem viewport proprio deixaria de cobrir o alvo.
  - Causa raiz: pratica fail-first levada ao extremo -- a mutacao foi commitada no ramo de trabalho e entrou em `main` para provar que `startup_stream_viewport_check.py` (criado 7 minutos antes em `ffb70f6a`) detecta a regressao.
  - Introduzido em: `1f24626b` (2026-11-24 17:38:50 +0100, autor tedin7, assunto "MUTATION (temporary): disable default viewport"), 1 arquivo, 1 linha.
  - Tentativas de correcao: nenhuma.
  - Correcao definitiva: `22989d61` (2026-11-24 17:40:18 +0100, "Revert \"MUTATION (temporary): disable default viewport\"").
  - **Verificacao de completude (pedido explicito desta analise):**
    - O revert e **byte-exato**: `git rev-parse 22989d61:src/startup_stubs/d3d12_draw.inl` = `98c69432...`, identico a `git rev-parse 1f24626b^:src/startup_stubs/d3d12_draw.inl`; o blob mutado era `9a649fe4...`. Nao sobrou nenhum resquicio de contexto, comentario ou linha vizinha alterada (o commit toca 1 arquivo, 1 linha).
    - `git log 1f24626b..HEAD -S'false && !pass.viewport_set'` retorna **apenas** o proprio `22989d61` (a remocao). Nenhum commit posterior reintroduziu o texto.
    - Varredura no HEAD: `git grep -n -i 'MUTATION' HEAD` so encontra controles de mutacao **de teste** (`CMakeLists.txt` e docs, `--mutate ...`); `git grep -n 'false &&' HEAD` retorna **uma** ocorrencia, em `tests/cpu/x87_fprem.py:179`, que e o texto esperado de um controle de mutacao do fixture de x87 -- nao codigo de produto.
    - Nao ha outro commit com assunto `MUTATION`/`temporary`/`WIP`/`do not merge` no historico (`git log --all -i --grep=...`): `1f24626b` e caso unico.
    - A mutacao **nao reapareceu em outro arquivo**: o mecanismo que ela desligava foi reescrito 13 minutos depois em `07f2464f` (D-26), e o estado em HEAD mantem `if (!pass.viewport_set && (pass.default_viewport_w != target.Width || ...))` com `RSSetViewports` ativo.
  - Arquivos: `src/startup_stubs/d3d12_draw.inl`.
  - Estado atual: revertido e evoluido; nao ha risco residual de comportamento.
  - Confianca: CONFIRMADA.

### Cobertura dos 45 blocos do pacote de evidencias

Classes: `a` = corrigiu defeito (pelo diff); `b` = introduziu defeito ou regressao; `c` = merge, feature, refactor ou teste sem relacao com defeito.

| hash | assunto (curto) | classe | ref. |
|---|---|---|---|
| 313803bc | triage: stop confirmado separado | a | D-13 |
| 6855f425 | cpu: perfil de inspecao de segmento | a | D-25 |
| 442d289e | merge #2021 (trace-calls) | c (carrega D-14) | D-14 |
| 5b29fe0f | docs: regra de retry/.fail | a | D-19 |
| e50a72bc | command signature: tipos indiretos | a | D-15 |
| 8ec25533 | merge #2008 (resume shader) | c (carrega D-18) | D-18 |
| 481a1c62 | merge #2014 (s_pack_hh) | c | — |
| ac539650 | debugger: call hooks opt-in | a | D-14 |
| f37fe7b8 | compute store: UNORM/SNORM + pins | a | D-28 |
| c168cf80 | gfx2hlsl: `s_pack_hh_b32_b16` | c (feature) | — |
| 3ff6e9d4 | gfx1013: image atomics | c (feature) | — |
| 86c225cc | merge: retem range fixture, skip so hardware | b (deliberado) | H-03 |
| 66423e45 | merge #1980 (MASM tokens) | c (carrega D-16) | D-16 |
| 2a898640 | assinatura: segura a superficie | a | D-01 |
| 13745b1b | triage: fixa reason tokens | a | D-20 |
| 06a494c8 | merge #1982 (disasm CLI) | c (carrega D-08) | D-08 |
| 14d6023e | MASM: preserva fixture isolado | a | D-16 |
| 2c1dcdd6 | view_heap: trap log com share | a | D-09 |
| 63d30a50 | NGG: modo `binary` com llvm-mc | c (teste) | — |
| 1ebfe511 | merge #1991 (range provenance) | c | — |
| f0451ea5 | script: sensibilidade/grupos | c (feature; toca o sampler de D-01) | — |
| 8c2d82f6 | script: espera pela imagem | b | D-01 |
| c3c3ab3c | startup: `a16` 2D empacotado | a | D-27 |
| e63721f9 | merge #1988 (masked waits) | c (carrega D-05) | D-05 |
| 1e7436db | merge #1970 (pe_inventory) | c (carrega D-22) | D-22 |
| 9dcf1ae1 | merge #1969 (class f32 SDWA) | c (feature) | — |
| e2a6ef57 | gpu: range provenance | c (teste) | — |
| 454af100 | merge #1976 (save fechado) | c (carrega D-04) | D-04 |
| 5d6d5f52 | teste: `initializer_list` | a | D-24 |
| 2e5fdca4 | teste: pin SDWA movrel | a | D-21 |
| 0b649c69 | build: `SKIP_RETURN_CODE 77` | a | D-11 |
| 83bb752d | disasm: SMEM com sinal | a | D-07 |
| 20fa94a9 | disasm: CLI/`--output` | a | D-08 |
| 889cd3f4 | merge #1964 (debug layer) | c (carrega D-23) | D-23 |
| b1715e1e | merge #1965 (MUBUF refusals) | c (mensagem de recusa) | — |
| 34472a58 | merge #1967 (XApu vazio) | c (carrega D-12) | D-12 |
| d8b51b5f | pixel_aot: DPP16/DPP8 f16 | c (feature) | — |
| f360d30a | `first_calls` saturante | a | D-10 |
| 65d07996 | pe_inventory: hardening + suite | a | D-22 |
| d28732b6 | attr sampled: sem debug layer | a | D-23 |
| 0478a386 | xmem: split de placeholder | b | D-02 |
| 583178d4 | gfx2hlsl: NaN exato | a | D-06 |
| aadcbd2c | merge #1789 (emit_variants) | c | — |
| 9727c306 | xmem: slabs/commit sob demanda | b | D-03 |
| c514f20b | merge #1937 (placed buffer) | c (carrega D-17) | D-17 |

Leitura do conjunto: **20 dos 45** blocos corrigem defeito; **4** introduzem defeito (e 3 desses 4 sao corrigidos no mesmo dia ou no dia seguinte: `8c2d82f6`->`2a898640`, `0478a386` e `9727c306`->`21e8f5c0`); **21** sao merge, feature, refactor ou teste sem relacao com defeito. Os 21 blocos "sem relacao" incluem 15 merges -- reflexo do processo: 44,7% do historico e merge e boa parte dos fixes chega em main por um merge sem diff proprio.

## E. Relacao de causa e efeito

1. `8c2d82f6` cria o sampler da assinatura sem referencia propria -> o titulo pode liberar a superficie apos o Present -> `2a898640` toma a referencia ate o fence e mantem o recurso se `Signal` falha.
2. `ca92741e` passa a manter o nome do container durante updates -> `begin_update` testa `names_` e aceita container fechado -> `2260b7a4` testa `refs_` e recusa.
3. `71d4bff1` trata todo tipo indireto acima de `DISPATCH_MESH` como console-only -> tipos futuros viram placeholder e ponteiros nao sao validados -> `e50a72bc` restringe a `0x69`, valida leitura/escrita e deixa o fallback para depois da recusa real.
4. `0478a386` remove a checagem de ownership no split -> `VirtualFree(MEM_PRESERVE_PLACEHOLDER)` atinge placeholder do chamador -> `21e8f5c0` volta a exigir reservation do shim e devolve o erro do retry.
5. `9727c306` faz `take_fresh` antes de `commit_slot` -> falha de commit deixa slab aberto porque `unwind_fresh` so ajusta `fresh_used_` -> `21e8f5c0` commita dentro de `make_slab` e fecha o handle em falha.
6. `e658fe24` baixa `nlt/nle/ngt/nge` como `!(a OP b)` e `o`/`u` como `a == a`/`a != a` -> FXC e DXC reescrevem sem `-Gis` -> `583178d4` usa o teste de bits `ORDERED` e recupera a semantica da ISA.
7. `ffb70f6a` (17:31) marca `viewport_set = true` ao encaminhar o **default** -> RT0 de outro tamanho herda viewport/scissor antigo -> `1f24626b` (17:38) desliga o default de proposito para provar o teste novo e entra em main -> `22989d61` (17:40) reverte em 88 s, byte-exato -> `07f2464f` (17:53) separa `default_viewport_w/h` do estado do titulo e resolve o defeito original.
8. `b87a99a3` passa a remover o `.fail` quando a traducao conclui -> a secao de custo do loop ainda diz o contrario -> `5b29fe0f` atualiza a doc (cadeia PROVAVEL: ramos paralelos, doc 37 min depois).
9. `5b2dadd7` reutiliza `asm` para o programa de pseudo-ops -> `fc.measure` deixa de medir o fixture isolado -> `14d6023e` separa `pseudo_ops_asm` e restaura a medicao; os dois entram em main pelo mesmo PR (`66423e45`).
10. `ef9a0798` deriva "stop event" de todo evento de excecao -> breakpoint de loader first-chance vira parada falsa -> `313803bc` separa "observed exception" de "confirmed debugger stop" e faz o launcher gravar o registro explicito.
11. `eefc3f86` decodifica SMEM sem sign-extend -> `-1` vira `0x1fffff` -> `83bb752d` estende o sinal e fixa vetores negativos e a forma `soffset + offset:`.
12. `99f097ad` passa a manter `X2W_TRAP_LOG` aberto com share -> o leitor do teste abre sem share e falha -> `2c1dcdd6` le com `_SH_DENYNO`; o defeito so aparecia em hardware (WARP faz skip), por isso o CI nao o viu.
13. `6177df41` emite `round()` no helper de store formatado -> empate exato grava 1 LSB abaixo -> `f37fe7b8` troca por `floor(x+0.5)`/`ceil(x-0.5)` e adiciona um caso de empate exato ao fixture.
14. `6b8ad31c` acrescenta o braco `else` e zera `region["skip"]` na transicao sem atualizar o snapshot -> `close_region` restaura o estado do opener -> `05ef4d92` usa `arm_outstanding` e so faz o join quando o braco final tem skip.
15. `1f24626b` (mutacao) -> `startup_stream_viewport_check.py` falha como esperado -> `22989d61` reverte; a prova de que o teste pega a regressao fica no historico, nao no codigo.

## F. Correcoes importantes

- **F-01 -- `583178d4` (NaN em comparacoes f32).** O erro era depender da **forma fonte** `!(a < b)`, que FXC e DXC reescrevem por algebra de comparacao sem `-Gis`. A correcao expressa a condicao como `!(ORDERED && a OP b)` e `o`/`u` como teste dos bits das duas palavras, o que nao depende de flag de compilador. Funciona porque `ORDERED` e o mesmo teste de bits que `lg` ja usava desde #1920. Reutilizavel em qualquer lowering que precise de semantica IEEE sob FXC/DXC. Risco: assumir que "not ordered" e o complemento de "ordered" -- so vale com o teste de bits, nunca com `==`/`!=` de float.
- **F-02 -- `2a898640` (tempo de vida da superficie).** Command lists nao seguram recursos. A correcao toma referencia da superficie ao submeter a copia e solta depois do fence; se `Signal` falha, o sampler entra em "stranded" e mantem os objetos ate o processo terminar em vez de arriscar use-after-free. A parte reutilizavel e o **controle negativo**: `unheld_source_control_warp` so passa quando a debug layer acusa o recurso liberado -- prova que o caminho positivo segura de verdade.
- **F-03 -- `21e8f5c0` (placeholder e rollback de slab).** Duas correcoes no mesmo arquivo: (a) so dividir placeholder que o shim reservou, com checagem de tamanho; (b) commitar o primeiro slot dentro da factory para que a falha feche o handle e nao deixe slab contabilizado. Padrao: "adquira e registre no mesmo ponto; em falha, desfaca antes de publicar estado". Reutilizavel em qualquer alocador com reserva/commit separados (`SEC_RESERVE` + `MEM_COMMIT`).
- **F-04 -- `b87a99a3` (publicacao atomica de shader).** Compila os dois estagios em diretorio temporario, so publica `.cso` depois de ambos terem bytecode, retira o manifesto antigo quando as bindings mudam e apaga o `.fail` no sucesso. O criterio de retomada passa a exigir `.txt` + ambos `.cso` (ou manifest-only). Elimina a mistura manifesto/bytecode de revisoes diferentes e o bloqueio permanente apos uma rodada sem fxc.
- **F-05 -- `ac539650` (hooks opt-in).** Separa hooks de arquivo (necessarios ao offline) de hooks de chamada (que param o processo). Os segundos passam a exigir `--trace-calls 1`, mantendo o comportamento antigo sob a flag. O teste com DLL source-built sob o nome do movie player conta 0 chamadas por padrao e exatamente 7 com a flag -- distingue "nao armou" de "armou e nao contou".
- **F-06 -- `2260b7a4` (container de save).** O nome pode sobreviver ao close enquanto um update o mantem; logo "aberto" e `refs_ > 0`, nao `name != ""`. O teste cobre o caso exato: update aberto + close + segundo `begin_update`.
- **F-07 -- `e50a72bc` (command signature).** Reconhecer so o valor observado (`0x69`) e deixar o resto para o runtime torna o shim resistente a ampliacao da enumeracao do desktop; validar `readable`/`writable` evita excecao em descritor invalido; o fallback depois da falha real preserva compatibilidade sem sequestrar tipos validos. E a unica correcao do pacote que **reduz** o escopo de um shim em vez de aumenta-lo.
- **F-08 -- `07f2464f` (default viewport por tamanho).** O estado "o titulo definiu" e o estado "ja encaminhei o default" sao coisas diferentes. A correcao guarda o **tamanho** ja encaminhado (`default_viewport_w/h`, `default_scissor_w/h`, zerados em `reset_viewport_state`) e regenera o default quando o alvo muda de tamanho. Reutilizavel em qualquer camada que sintetize estado de pipeline ausente: nunca use a flag do chamador para registrar o que voce mesmo sintetizou.
- **F-09 -- `c3c3ab3c` (cache invalidado por revisao).** O fix de `a16` e mecanico, mas a parte que generaliza e a **revisao de cache**: `x2w-startup-shader-5` -> `-6`, escrita nos dois lados (tool e runtime). Qualquer mudanca de semantica de traducao que possa reutilizar bytecode antigo exige bump nos dois arquivos; em HEAD os dois concordam (`d3d12_draw.inl:201`, `gfx2hlsl.py:2176`).
- **F-10 -- `22989d61` (revert disciplinado).** O valor aqui e processual: uma mutacao commitada para provar um teste foi revertida em 88 s, com o corpo do commit citando o hash original, e o revert e byte-exato (mesmo blob). Verificar "o revert foi completo" e barato: comparar o blob antes da mutacao com o blob do revert (`git rev-parse <rev>:<path>`).

## G. Problemas ainda existentes

- **G-01** O sampler "stranded" de `2a898640` mantem superficie, allocator, lista e readback ate o fim do processo quando `Signal` falha depois da submissao. E deliberado (evita use-after-free), mas e vazamento permanente; nao ha recuperacao nem contagem desse estado nos relatorios.
- **G-02** Em `9727c306`/`21e8f5c0`, um slot liberado continua commitado; `x2w_xmem_slab_stats` reporta slabs/slots, nao o commit retido. A doc diz isso, mas ferramentas que somam commit podem subestimar o consumo real.
- **G-03** `21e8f5c0` recusa dividir placeholder do chamador maior que a view. E a correcao segura, porem muda o contrato: chamadores que dependiam do split agora falham e precisam reservar no tamanho exato.
- **G-04** `313803bc` so confirma a parada quando o log tem o registro explicito ou o `.out` do launcher. Logs antigos, sem esse arquivo, continuam sem "confirmed debugger stop" -- o relatorio diz `not reported`.
- **G-05** `b87a99a3` considera bytecode presente por `os.path.getsize(path) > 0`; um `.cso` truncado ou corrompido passa no criterio de completude e so falha no consumo. HIPOTESE: nao ha teste com cso invalido no repositorio.
- **G-06** Em HEAD, `src/startup_stubs/d3d12_xs_device.cpp:527` e `:557` ainda usam `shape_checks < 32 && InterlockedIncrement(&shape_checks) <= 32`. A guarda `shape_checks < 32` limita o incremento, entao nao ha risco de wrap em 2^31, mas o padrao nao foi unificado com `first_calls`; uma remocao futura da guarda reabre D-10.
- **G-07** A convencao "77 = capability skip" so funciona se o teste nunca retornar 77 por outro motivo; `0b649c69` e `86c225cc` dependem da convencao, nao de uma checagem do harness.
- **G-08** `13745b1b` e `2e5fdca4` fecham lacunas especificas (tabela de reasons, pins de selecao SDWA), mas nao ha varredura sistematica de outras tabelas de classificacao ou de outros `*_PINS` que nunca recebem uma linha propria. HIPOTESE.
- **G-09** Nenhum resultado deste pacote estabelece startup do titulo ou execucao de codigo do jogo; os testes provam apenas o comportamento dos fixtures e ferramentas citados.
- **G-10** O default viewport/scissor de `07f2464f` e regenerado por **tamanho** de RT0, nao por identidade de recurso. Dois alvos distintos de mesmo tamanho nao recebem um novo `RSSetViewports` -- o que e correto para o par viewport/scissor (o estado nao depende do recurso), mas significa que uma mudanca de formato/orientacao que exija outro retangulo nao e coberta. HIPOTESE de risco, nao defeito observado.
- **G-11** `a16` em 3D/2D-array e `image_sample_l` com `a16` continuam **recusados** apos `c3c3ab3c`; qualquer shader de titulo que use essas formas cai no caminho de recusa por nome, nao em traducao degradada.
- **G-12** O tie-break da unidade de formato de buffer da RDNA 2 nao esta medido: `f37fe7b8` adota a regra do Direct3D e o proprio comentario declara "unverified on gfx1013". Se o hardware usar ties-to-even (como as conversoes `v_cvt_pk_*` do LLVM, que sao outra unidade), o store formatado erra 1 LSB em empates exatos.
- **G-13** O cache de shader tem uma unica revisao global (`x2w-startup-shader-6`). Um `shader_cache/` antigo continua valido para tudo o que nao mudou de semantica, mas qualquer correcao futura que esqueca o bump reutiliza bytecode pre-fix -- foi exatamente o risco que `c3c3ab3c` teve de resolver. Nao ha teste que force a igualdade das duas constantes; a coincidencia e verificada a mao.

## H. Regressoes

- **H-01** `0478a386` regrediu o comportamento anterior ao remover `owned_reservation` do caminho de split; passou a dividir placeholder de qualquer dono. Corrigido em `21e8f5c0` (D-02).
- **H-02** `9727c306` introduziu estado parcial em falha de commit (slab aberto), contrariando a promessa de "leaving no state changed". Corrigido em `21e8f5c0` (D-03).
- **H-03** `86c225cc` (merge que resolve `0b649c69` com `de2ca7ea`) remove `SKIP_RETURN_CODE 77` das variantes **WARP** de `startup_cb_blend`, `startup_cb_blend_swapped` e `startup_composite_provenance`, e o mantem so nas `_hardware`. Nao e regressao de comportamento: o proprio assunto do commit e "scope adapter skips to hardware", WARP sempre encontra adapter e a propriedade nao teria efeito. E, tecnicamente, um revert parcial de D-11 -- vale registrar porque um leitor do diff de `0b649c69` isolado concluiria o contrario.
- **H-04** `5b2dadd7` degradou o teste do fixture MASM isolado ao reutilizar `asm`; a regressao e de cobertura, nao de produto. Corrigida em `14d6023e` (D-16), dentro do mesmo PR.
- **H-05** `b87a99a3` mudou o ciclo de vida do `.fail` sem atualizar a doc; a inconsistencia so foi corrigida em `5b29fe0f` (D-19), 37 minutos depois e em ramo paralelo.
- **H-06** `ffb70f6a` corrigiu um defeito (draw sem viewport nao pinta) e criou outro no mesmo bloco: passou a marcar `viewport_set` ao encaminhar o default, impedindo a regeneracao quando RT0 mudava de tamanho. Corrigido em `07f2464f` (D-26), 22 minutos depois. E o caso mais claro do repositorio de "fix que introduz regressao no proprio caminho que conserta".
- **H-07** `1f24626b` colocou em `main`, por 88 segundos, uma mutacao que desliga o viewport default. Nao ha evidencia no repositorio de build publicado entre os dois commits (o intervalo e de 88 s e o revert e byte-exato, `NAO VERIFICADO` se algum job de CI chegou a rodar em `1f24626b`), mas e uma regressao deliberada no historico e o unico caso do tipo (D-29).
- **H-08** `c3c3ab3c` mudou a semantica de traducao de `a16` e, corretamente, bumpou a revisao do cache. Se o bump tivesse sido esquecido, a regressao seria silenciosa: bytecode antigo (UV errado) continuaria sendo servido a partir de `shader_cache/`. Nao houve regressao, mas o risco e estrutural (G-13).

## I. Conhecimento reutilizavel

- **I-01 -- Placeholder: so divida o que voce possui.** Antes de `MEM_RELEASE | MEM_PRESERVE_PLACEHOLDER`, confirme que a faixa pertence a uma reserva do proprio componente e que o tamanho cabe nela. Preserve o erro original apenas se o split **nao** ocorreu; se ocorreu e o retry falhou, reporte o erro do retry. Arquivos: `src/xmem_mapped_pages.cpp`; commits `0478a386`, `21e8f5c0`. Risco: dividir placeholder alheio corrompe o estado do chamador.
- **I-02 -- Reserva/commit sob demanda com rollback simetrico.** Crie o slab com `SEC_RESERVE`, commite o slot no primeiro uso e, em falha, feche o handle e nao registre o slab. Nunca insira o objeto numa colecao antes da operacao que pode falhar sem caminho de remocao. Arquivos: `src/xmem_mapped_pages.cpp`, `src/xmem_slab_slots.h`; commits `9727c306`, `21e8f5c0`.
- **I-03 -- Recurso de GPU: referencia ate o fence.** Command lists nao retem recursos; quem enfileira copia precisa de referencia propria ate o fence. Se o sinal falhar, mantenha o recurso e marque o subsistema como parado em vez de liberar. Teste reutilizavel: controle que so passa quando a debug layer acusa o recurso liberado. Arquivos: `src/startup_stubs/present_signature_sampler.h`; commits `8c2d82f6`, `2a898640`.
- **I-04 -- "Aberto" e refcount, nao nome.** Em tabelas que mantem o nome apos o close para nao invalidar updates, a condicao de abertura deve ser `refs > 0`. Arquivos: `src/startup_stubs/save_store.h`; commits `ca92741e`, `2260b7a4`.
- **I-05 -- Semantica IEEE nao sobrevive a `!(a OP b)`.** Expresse ordenado/nao-ordenado com teste de bits e nunca com `a == a`/`a != a`. Valide com FXC **e** DXC sem `-Gis`; a flag muda o resultado. Arquivos: `tools/startup/gfx2hlsl.py`, `tests/startup_gfx2hlsl_f32_nan_check.py`; commit `583178d4`.
- **I-06 -- Campo assinado: sign-extend antes de imprimir.** Para `OFFSET[20:0]` de SMEM, mascare e estenda o bit 20; imprima `-0x...` para negativos e fixe vetores em -1, minimo e maximo positivo. Arquivos: `tools/disasm_rdna2.py`; commits `eefc3f86`, `83bb752d`. Risco: automacao que compara desmontagem ve offsets diferentes do LLVM.
- **I-07 -- Gate de log atomico deve saturar.** `InterlockedIncrement <= N` estoura e reabre o log; use CAS com `seen < limit` e pare em `limit`. Arquivos: `src/startup_stubs/local_log.h`; commit `f360d30a`. Risco: consumidor que faz poll de status reativa o log depois de ~2^31 chamadas.
- **I-08 -- Publicacao de artefato de build: staging + manifesto por ultimo.** Compile tudo em diretorio temporario, publique bytecode so com o par completo, retire o manifesto antigo quando as bindings mudam e apague o `.fail` no sucesso. Arquivos: `tools/startup/gfx2hlsl.py`; commit `b87a99a3`. Risco: manifesto e bytecode de revisoes diferentes; retomada bloqueada apos rodada manifest-only.
- **I-09 -- CLI de ferramenta: modos uniformes.** Todo modo (inclusive `--hex`) deve passar pelo mesmo writer de `--output`; entrada malformada usa `parser.error` (exit 2) sem traceback; alvo ausente vai para stderr e exit 1 **depois** de processar os demais. Arquivos: `tools/disasm_rdna2.py`; commit `20fa94a9`.
- **I-10 -- Teste de fixture: nunca reutilize o nome da variavel.** Um `asm` reutilizado troca silenciosamente o programa medido; separe `asm` e `pseudo_ops_asm` e fixe a medicao no fixture original. Arquivos: `tests/form_coverage_check.py`; commits `5b2dadd7`, `14d6023e`.
- **I-11 -- Estado sintetizado nao compartilha flag com o estado do chamador.** Se a camada cria um default (viewport, scissor, blend, root signature), registre o que **ela** encaminhou (e de que tamanho/versao) numa variavel separada da flag do chamador; caso contrario o default nunca e regenerado quando o contexto muda. Arquivos: `src/startup_stubs/d3d12_draw.inl`, `d3d12_xs_device.cpp`; commits `ffb70f6a`, `07f2464f`.
- **I-12 -- Mudanca de semantica de traducao exige bump de cache nos dois lados.** Tool e runtime devem carregar a mesma string de revisao; um `shader_cache/` mantido entre execucoes e o caminho normal do projeto, entao o bump e o que impede bytecode pre-fix de sobreviver ao fix. Arquivos: `src/startup_stubs/d3d12_draw.inl`, `tools/startup/gfx2hlsl.py`; commit `c3c3ab3c`.
- **I-13 -- Mutacao temporaria: revert comprovado por blob, nao por mensagem.** Se for commitada uma mutacao para provar um teste, o revert deve citar o hash original **e** restaurar o blob identico (`git rev-parse <rev>:<path>` antes e depois). Verifique tambem `git log <mutacao>..HEAD -S'<texto mutado>'` para garantir que ninguem reintroduziu o texto noutro arquivo. Commits `1f24626b`, `22989d61`.
- **I-14 -- Cobertura de tabela de classificacao: amostra por entrada + verificacao reversa.** Uma tabela `pattern -> token` sem amostra por linha acumula causas silenciosamente em `other`; o teste deve alimentar cada entrada, casar a amostra de volta ao indice e exigir que toda entrada tenha amostra. Arquivos: `tests/startup_triage_run_check.py`; commit `13745b1b`.
- **I-15 -- Skip de capacidade precisa de codigo dedicado e documentado.** A convencao 77 do CTest funciona, mas exige que o binario so retorne 77 por ausencia de capacidade; qualquer outro motivo contamina o relatorio. Commits `0b649c69`, `86c225cc`; arquivos `CMakeLists.txt`, `tests/desktop_gpu.h`.

## J. Conclusao tecnica

O Xbox2Windows e um projeto de pesquisa com disciplina de engenharia incomum para cinco dias e 4076 commits: quase toda correcao entra acompanhada de um **controle negativo que deve falhar**, e o vocabulario dos commits ("fail-first", "must mismatch", "the control must fail") e sustentado pelo codigo dos testes, nao apenas pela mensagem. Isso torna a arqueologia barata: em quase todos os casos o diff do teste diz exatamente qual era o defeito.

Os defeitos reais concentram-se em cinco familias, e todas tem a mesma forma -- **estado que atravessa uma fronteira**:

1. **Fronteira de comando GPU** (D-01): a lista nao segura o recurso; quem enfileira precisa segurar.
2. **Fronteira de transacao de alocador** (D-02, D-03): reserva e commit separados exigem rollback simetrico e checagem de posse.
3. **Fronteira de compilador** (D-06, D-28): a semantica do HLSL emitido nao e a semantica da ISA, e flags mudam o resultado; a unica defesa e expressar a condicao em termos que nenhum compilador reescreve.
4. **Fronteira de braco de controle** (D-05): dataflow com skip por `exec` precisa de snapshot por braco, nao por regiao.
5. **Fronteira de camada sintetizada** (D-26): o que a camada inventa nao pode ser registrado na flag do chamador.

A mutacao temporaria (`1f24626b`, revertida em `22989d61`) e o caso-limite dessa cultura: uma regressao deliberada, em main, por 88 segundos, para provar um teste criado 7 minutos antes. O revert foi completo -- blob identico ao pre-mutacao, uma linha, um arquivo -- e a mutacao nao reapareceu em nenhum outro arquivo em HEAD; o codigo que ela desligava foi reescrito 13 minutos depois (`07f2464f`) para corrigir um defeito real que o proprio fix anterior (`ffb70f6a`) havia introduzido. Esse encadeamento -- fix, mutacao de prova, revert, fix do fix -- e, em miniatura, o metodo do repositorio inteiro.

O que **nao** esta provado e o que importa para o objetivo declarado: nada aqui mostra o titulo rodando. O README e honesto quanto a isso ("startup of the original EXE, main-menu navigation and gameplay remain unverified"), e a maior parte do volume diario e cobertura de formas de instrucao e de slots de API, nao integracao de ponta a ponta. A maturidade esta na instrumentacao (trap log, log local, heartbeat, triagem, relatorios de endpoint) e na recusa explicita: o projeto prefere falhar por nome a traduzir errado, o que e a decisao certa para um recompilador estatico -- e a razao pela qual os defeitos que sobrevivem sao de fronteira, nao de traducao.

## K. Indice de commits

Todos os 4076 commits alcancaveis de `HEAD`, em ordem cronologica (commit 0 primeiro), no formato `hash|data|assunto`. Autoria nao repetida aqui (ver secao A); a atribuicao de cada commit aparece em `git log`. Os 423 commits que existem apenas em outras refs nao entram neste bloco -- ver L.
```text
7e28f98c|2026-11-21|Initialize Xbox2Windows project
ce546383|2026-11-21|Publish PC compatibility experiments and contributor workflow
f8951def|2026-11-21|Open English contributor tasks for the GTA VI Xbox Series X case study
6c38c5ce|2026-11-21|Merge pull request #3 from R0Y00/community/english-contributor-entry
2eb5a816|2026-11-21|Package the Windows GPU checks for independent hardware reports
def386ab|2026-11-21|Merge pull request #4 from R0Y00/community/gpu-check-release
bf39ff64|2026-11-21|Verify desktop BVH root roundtrips, memory predicates and PE dependency paths
c63caed5|2026-11-21|Merge pull request #5 from R0Y00/engineering/intake-root-roundtrip
6a6ab2a2|2026-11-21|Verify concurrent BVH bounds, numeric operations and in-place DXR updates
17d9e281|2026-11-21|Merge pull request #6 from R0Y00/engineering/bvh-build-and-update
0b254503|2026-11-21|Verify packed BVH leaf expansion through ray traversal and GPU bounds initialization
127fa864|2026-11-21|Merge pull request #7 from R0Y00/engineering/packed-bvh-leaves
11fca329|2026-11-21|Measure device creation for BVH and upscaler root signatures
2da28c83|2026-11-21|Merge pull request #8 from R0Y00/codex/root-signature-device-coverage
7d22c5a3|2026-11-21|Exercise pre-entry GPU workloads, archive metadata and import gaps
077eb184|2026-11-21|Merge pull request #10 from R0Y00/codex/pre-exe-workloads
ba3e72cb|2026-11-21|Recompile single-texture gfx1013 pixel programs for desktop D3D12
07a835cc|2026-11-21|Resolve length-prefixed records by their logical names
95b65576|2026-11-21|Merge pull request #11 from R0Y00/codex/pixel-aot
741e9efc|2026-11-21|Lower constant buffer reads and pixel color arithmetic to DXIL
90b422ed|2026-11-21|Merge pull request #12 from R0Y00/codex/pixel-constants
0494b7af|2026-11-21|Read paged resource ranges and validate tagged section streams
44144ea3|2026-11-21|Merge pull request #13 from R0Y00/codex/asset-ranges
15915570|2026-11-21|Draw detiled BC7 surfaces and validate implicit mip selection
e153b5c7|2026-11-21|Merge pull request #14 from R0Y00/codex/bc7-texture-draw
8afdb845|2026-11-21|Read packed BC7 mip tails and probe fixed-symbol BCPack prefixes
9c0480f7|2026-11-21|Merge pull request #16 from R0Y00/codex/bcpack-mip-tail
b1e02a00|2026-11-21|Verify BC7 mip fallback and first BCPack refill predictions
52fed119|2026-11-21|Merge pull request #17 from R0Y00/codex/bc7-resident-tail
d46339e5|2026-11-21|Verify state-linked camera parameters in script metadata
66eb87ba|2026-11-21|Merge pull request #18 from R0Y00/codex/script-state-evidence
894a13a0|2026-11-21|Replay context-scoped script state selection on Windows
29e016cd|2026-11-21|Merge pull request #20 from R0Y00/codex/script-state-path-replay
757c21e4|2026-11-21|Align contributor entry points with current Windows porting work
d1bd2975|2026-11-21|Merge pull request #21 from R0Y00/codex/community-project-status
f97be9b1|2026-11-21|Recompile and verify both branches of an x64 PE leaf function
fe78c98e|2026-11-21|Merge pull request #23 from R0Y00/codex/cpu-leaf-aot
663d9761|2026-11-21|Recompile a direct x64 call and verify its Windows ABI
75d98a30|2026-11-21|Merge pull request #25 from R0Y00/codex/cpu-direct-call
37fd03c0|2026-11-21|Recompile a named x64 PE import and verify Windows loader outcomes
83c7d777|2026-11-21|Keep expected loader failures noninteractive in the test subprocess
b5675831|2026-11-21|Merge pull request #27 from R0Y00/codex/cpu-import-call
0de5cd71|2026-11-21|Recover chained x64 function ranges and lift guarded return paths
c5ff6ccc|2026-11-21|Merge pull request #30 from R0Y00/codex/chained-cpu-ranges
5710009d|2026-11-21|Recompile ordinal-only PE import calls and verify loader failures
8b55e069|2026-11-21|Merge pull request #31 from R0Y00/codex/cpu-ordinal-import
a80b15c7|2026-11-21|Lift explicitly bound x64 object-table callbacks
f4596277|2026-11-21|Merge pull request #32 from R0Y00/codex/typed-object-callback
3044173e|2026-11-21|Trace and recompile the FidelityFX provider destroy contract
b3759d83|2026-11-21|Merge pull request #34 from R0Y00/codex/ffx-provider-contract
f8f4d539|2026-11-21|Run the statically translated FidelityFX version query on Windows
af17557e|2026-11-21|Merge pull request #35 from R0Y00/codex/ffx-version-query
7542bc3b|2026-11-21|Connect FidelityFX context creation to the recompiled destroy path
1389d735|2026-11-21|Merge pull request #36 from R0Y00/codex/ffx-context-lifecycle
248b759e|2026-11-21|Recompile the upscaler backend's first D3D12 device calls
cb99c449|2026-11-21|Merge pull request #38 from R0Y00/codex/upscaler-device-init
c4645bc3|2026-11-21|Translate and verify the upscaler backend context slot scan
23715e19|2026-11-21|Merge pull request #40 from R0Y00/codex/ffx-backend-slots
2c1091c9|2026-11-21|Record two community Radeon wave64 hardware reports
71bb9f4f|2026-11-21|Merge pull request #41 from R0Y00/codex/amd-wave64-reports
c8065c50|2026-11-21|Recompile and create the upscaler recurrent texture on D3D12
e0825372|2026-11-21|Merge pull request #43 from R0Y00/codex/ffx-recurrent-resource
962bdb4d|2026-11-21|Recompile recurrent texture views and verify UAV writes through SRV readback
24658375|2026-11-21|Merge pull request #45 from R0Y00/codex/ffx-recurrent-views
2e678b35|2026-11-21|Translate the FidelityFX provider resource loop and verify context handles (#47)
7c175f23|2026-11-21|Document community GPU runs and independent Windows build (#48)
965f374d|2026-11-21|Report layout chunks and launch marker in verify_build
861cde58|2026-11-21|Record GPU driver retest and refresh community report routes (#50)
fadc3cd5|2026-11-21|Accept unordered DXContainer part offsets and reject physical overlaps
0117a01d|2026-11-21|Merge pull request #51 from dcastell1/codex/layout-chunk-summary
bbdaba73|2026-11-21|Pack and verify RGB9E5 texels on the provider RCAS intermediate (#55)
e226d7a0|2026-11-21|Map source-built images without LoadLibrary and compare with the lifted calls
f6fe2dbe|2026-11-21|Record community GPU runs and Windows signature cross-check
10c4136c|2026-11-21|Reject undersized import directories and empty export slots in pe_map
fd7a1804|2026-11-21|Merge pull request #49 from Leyukaka/native-pe-map
e69639dd|2026-11-21|Reject duplicate FourCC tags in DXIL framing scanner
ec05dab5|2026-11-21|Merge pull request #59 from R0Y00/codex/community-coverage-provenance
6c820f31|2026-11-21|tools: add a summary and #1 submit path to the GPU check report
59959547|2026-11-21|Raise and catch SEH exceptions inside a mapped image
995fc999|2026-11-21|Record four independent Windows GPU reports
d88528c5|2026-11-21|Merge pull request #63 from R0Y00/codex/four-windows-gpu-reports
5682b094|2026-11-21|Merge pull request #53 from Joacoorandom/fix/dxcontainer-unordered-parts
6c495f59|2026-11-21|Use platform-focused wording in README.md
bceb2291|2026-11-21|Use platform-focused wording in CONTRIBUTING.md
7600adde|2026-11-21|Check a fixed-symbol lane's second refill against a supplied round mask
31936f7e|2026-11-21|Use platform-focused wording in docs/experiments.md
d6a8262d|2026-11-21|Accept the comma after exp targets in newer LLVM disassembly
c320c7bd|2026-11-21|Recompile the first FSR4 writer and reader of resource 16 (#66)
2731c0e4|2026-11-21|Merge pull request #67 from R0Y00/codex/generic-project-documentation
7725ea77|2026-11-21|Cover both exp spellings in the pixel translator controls
26d851f4|2026-11-21|Merge pull request #58 from pedro-anthony/pixel-aot-llvm-exp-comma
5f912f5f|2026-11-21|Keep import names that pefile rejects in the PE inventory
8289615d|2026-11-21|Bind system ordinal imports by their stored ordinal
f59e6842|2026-11-21|Compare sign-extended immediates at operand width in the query lowerer
6151171a|2026-11-21|Lower gfx1013 float comparisons and lane selects to HLSL
c263f5fd|2026-11-21|Merge branch 'main' into mapped-seh
6968c42e|2026-11-21|Merge pull request #65 from Leyukaka/mapped-seh
85114658|2026-11-21|Merge branch 'main' into contribution/bcpack-second-refill
d0e6f977|2026-11-21|Record 15 Windows GPU runs and capability skips
fe9603f7|2026-11-21|Merge pull request #69 from descent81/contribution/bcpack-second-refill
83229f5b|2026-11-21|Merge branch 'main' into fix/gpu-check-report-submit-path
5e8b62c5|2026-11-21|Merge branch 'main' into codex/gpu-community-batch-64
89262232|2026-11-21|Recompile and trace both FSR4 scratch writers feeding pass 26 (#72)
019f4f02|2026-11-21|Merge branch 'main' into codex/gpu-community-batch-64
b7f7e3fa|2026-11-21|Merge pull request #71 from R0Y00/codex/gpu-community-batch-64
6464ce93|2026-11-21|Link newcomer discussions from README.md
75fed518|2026-11-21|Link newcomer discussions from CONTRIBUTING.md
0b42a44a|2026-11-21|Merge pull request #79 from R0Y00/codex/discussions-entry-points
036f08b3|2026-11-21|Run platform-independent Python checks on Linux
61e94169|2026-11-21|Recompile and verify the default Bink heap callers on Windows (#88)
195042ef|2026-11-21|Accept only label-kind state IDs in the script state probe and cover more synthetic trees
ba26c1ac|2026-11-21|Check ERA heap zeroing on recycled blocks
3e763a56|2026-11-21|Merge branch 'main' into linux-python-checks
42608828|2026-11-21|Support absolute-value operands in pixel shader translation
ef22a29a|2026-11-21|Merge pull request #80 from Vonsek/linux-python-checks
b40db732|2026-11-21|Record seven Windows hardware reports from issue 1
bb677389|2026-11-21|Add a source-built BC7 block reference and field fixture
88fd8c59|2026-11-21|Recompile Bink custom heap callbacks and verify release ownership (#92)
27567fa2|2026-11-21|Skip cs_6_5 BVH fixtures on adapters below Shader Model 6.5
d4fcf032|2026-11-21|docs: add AGENTS.md for coding agents
b113c629|2026-11-21|bcpack: add a stride-6 lane diagnostic
43f23353|2026-11-21|meta: align issue forms and pull request template
64ea7149|2026-11-21|Follow export forwarders in package DLLs
60dde88a|2026-11-22|Lower and render a two-texture gfx1013 pixel shader
2ce2b96f|2026-11-21|Zero-fill a mapped section that has no file offset, as Windows does
73f7166d|2026-11-21|Call TLS callbacks and the entry point from pe_map in the Windows order
cbbf67b9|2026-11-21|Allow floating-point tolerance for ray distance in check_dxr_update
e7ff764a|2026-11-21|Add RDNA 2 shader disassembly, extraction, and coverage scanning tools
482115ad|2026-11-21|Correct the note on nonzero unaligned section file offsets
fed0e8ec|2026-11-21|Note the DXC version that shader_metadata_probe needs
b67071a5|2026-11-21|Read section file data in whole 0x200-byte sectors, as Windows does
c5ed0cfa|2026-11-22|Round-trip the dual-texture fixture through gfx1013 machine code
611d87c4|2026-11-21|Record the next batch of Windows GPU reports from the hardware thread
b483ebe6|2026-11-22|Merge pull request #78 from Leyukaka/tls-callbacks
a9bc2280|2026-11-22|Merge branch 'main' into parser-robustness
a46f1ff2|2026-11-22|Merge pull request #83 from Vonsek/parser-robustness
ce6fa1da|2026-11-22|Merge branch 'main' into pe-imports-callers
4bdfe5da|2026-11-22|Verify sample order and descriptor reuse against the blend reference
f633956d|2026-11-22|Merge remote-tracking branch 'origin/main' into shader-abs-operand
0f5f6ec2|2026-11-22|Merge pull request #82 from Vonsek/pe-imports-callers
ee1b7483|2026-11-22|Merge branch 'main' into fix/sm65-capability-skip
d45e6748|2026-11-21|tools: use ASCII punctuation in the GPU check runner for Windows PowerShell 5.1
8d3793ee|2026-11-22|Merge remote-tracking branch 'origin/main' into pe-map-zero-raw-pointer
5053c2a9|2026-11-22|Merge branch 'main' into contribution/dual-texture-pixel
a57a4af6|2026-11-22|Merge branch 'pe-map-zero-raw-pointer' into pe-map-sector-reads
ed242e5d|2026-11-22|Merge pull request #99 from rasptum/fix/sm65-capability-skip
0995ff07|2026-11-22|Merge branch 'main' into docs/dxc-module-validation
0a1a4044|2026-11-22|Merge commit '5053c2a9d4c61808453bc35b9c2d26afefa3a0d5' into pixel-aot-instructions
657ecaef|2026-11-22|Merge branch 'pixel-aot-instructions' into shader-abs-operand
2d96d6d3|2026-11-22|Merge pull request #110 from Krille2004zzz/docs/dxc-module-validation
9c323f86|2026-11-22|Merge commit 'b40db7321551e42beba17295351b6292dc540301' into docs/hardware-batch-b580-and-more
0c95de51|2026-11-22|Merge branch 'main' into docs/hardware-seven-reports
1ab002f0|2026-11-22|Record the measured Windows block reuse count
72306fa7|2026-11-22|Merge commit '5f912f5f237de6e716c37131151ccb069b2d3f5c' into pe-forwarded-exports
63fc71cb|2026-11-22|Merge pull request #90 from R0Y00/docs/hardware-seven-reports
b8bd6e80|2026-11-22|Merge branch 'main' into pe-map-zero-raw-pointer
03e665fa|2026-11-22|Keep the agent guide aligned with existing contribution requirements
3a248235|2026-11-21|Reject duplicate state ID attributes in the script state probe
34142824|2026-11-21|Report a named skip when the SDK validator refuses module-only validation
6c6446ab|2026-11-22|Merge pull request #107 from danielradosa/pe-map-zero-raw-pointer
1d9dbca1|2026-11-22|Merge remote-tracking branch 'origin/main' into bcpack-reference
3074f34a|2026-11-22|Merge branch 'main' into pe-map-sector-reads
be12d43d|2026-11-22|Merge pull request #111 from danielradosa/pe-map-sector-reads
f5e63dde|2026-11-22|Merge branch 'main' into contribution/dual-texture-pixel
9c9523f6|2026-11-21|Run the module-only validation controls as their own test
babb56ee|2026-11-22|Check valid and malformed absolute comparison operands
f30ca0dc|2026-11-22|Merge remote-tracking branch 'origin/main' into fix/gpu-check-report-submit-path
84a82f47|2026-11-22|Merge pull request #100 from descent81/contribution/dual-texture-pixel
1a648594|2026-11-22|Merge branch 'main' into pixel-aot-instructions
5fce3f3b|2026-11-21|Lower RDNA2 32-bit bitfield extracts with masked HLSL shifts
40b0d1c4|2026-11-22|Merge remote-tracking branch 'origin/main' into docs/hardware-batch-b580-and-more
fd1ff2a1|2026-11-22|Merge pull request #84 from Vonsek/pixel-aot-instructions
3f1cd41d|2026-11-22|Merge branch 'main' into pe-inventory-imports
0668f667|2026-11-22|Merge pull request #81 from Vonsek/pe-inventory-imports
0bec695c|2026-11-22|Merge branch 'main' into pe-forwarded-exports
024585c8|2026-11-21|Merge branch 'main' into fix/dispatch-aot-bfe
67b7f08d|2026-11-21|Accept round-toward-zero RGB9E5 stores on the native RCAS path
ee395360|2026-11-22|Merge pull request #105 from Vonsek/pe-forwarded-exports
b8f48fdb|2026-11-21|Merge branch 'main' into fix/dispatch-aot-bfe
44651f1d|2026-11-22|Merge branch 'main' into bcpack/stride-6-diagnostic
2a3f6ec6|2026-11-21|Validate DXContainers via resource_inventory.parts and mapped section bounds in call_evidence
9b966513|2026-11-22|Merge pull request #96 from raulmoracode/bcpack/stride-6-diagnostic
2dbc2642|2026-11-22|Merge branch 'main' into script-state-synthetic
2524d743|2026-11-22|Verify packed RCAS neighbor coordinates by GPU readback
a2a62ee2|2026-11-22|Merge pull request #102 from Vonsek/script-state-synthetic
8d6ae0aa|2026-11-22|Merge branch 'main' into docs/agents-md
4d2835d5|2026-11-21|Recompile x64 switch dispatch through jump tables
eb0f7fbe|2026-11-22|Merge pull request #94 from raulmoracode/docs/agents-md
4229a277|2026-11-22|Merge branch 'main' into meta/contribution-workflow
d603dd69|2026-11-21|feat(cpu): recompile the grouped Bink allocation caller
3f1e504b|2026-11-22|Merge pull request #95 from raulmoracode/meta/contribution-workflow
04b8b48f|2026-11-22|Merge branch 'main' into shader-abs-operand
4cac5e65|2026-11-21|Walk import descriptors to the terminator in pe_map, as Windows does
47e990bf|2026-11-22|Merge pull request #89 from RonLogic/shader-abs-operand
15adb728|2026-11-22|Merge branch 'main' into docs/hardware-batch-b580-and-more
4c4da0d8|2026-11-21|Add source-owned automated checks and opcode recognition split for RDNA 2 tooling
a4fa8235|2026-11-21|tooling: add a measured progress dashboard
4e3d535b|2026-11-22|Merge pull request #113 from Krille2004zzz/docs/hardware-batch-b580-and-more
3de8f5f8|2026-11-22|Merge branch 'main' into era-heap-allocation
87d14b56|2026-11-21|Test each console root-signature bit separately
2be03d84|2026-11-22|Merge remote-tracking branch 'origin/main' into bcpack-reference
01aa32f8|2026-11-21|Find TLS and delay-import directories by address in pe_map
72db3c87|2026-11-22|Merge pull request #104 from Vonsek/era-heap-allocation
dc08688b|2026-11-21|feat(cpu): trace grouped Bink allocation descriptors
c97f66f4|2026-11-22|Merge branch 'main' into fix/gpu-check-report-submit-path
5febbc34|2026-11-22|Merge remote-tracking branch 'origin/main' into native-audit
a2f1f4e3|2026-11-22|Merge pull request #62 from Jawschamp/fix/gpu-check-report-submit-path
da357bdd|2026-11-22|Merge branch 'main' into bcpack-reference
d6429798|2026-11-22|Merge pull request #106 from Vonsek/bcpack-reference
a15097a1|2026-11-21|Apply x86 count masking to byte and word memory shifts
c6124641|2026-11-22|Merge branch 'native-audit' into pe-map-tls-delay-dirs
a0d995fe|2026-11-22|Merge branch 'main' into native-audit
10d404eb|2026-11-21|Accept version-2 chained unwind headers in function_ranges
839bf855|2026-11-21|Count coplanar rays' box candidates in the BVH hit reference
17cf485d|2026-11-21|Compare DXR face culling with a CPU reference under mirrored instances
4a92b446|2026-11-22|Merge pull request #85 from Vonsek/native-audit
5623001c|2026-11-22|Merge branch 'main' into pe-map-tls-delay-dirs
4d7dcc4b|2026-11-22|Merge remote-tracking branch 'origin/main' into cpu-aot-coverage
c58fec4b|2026-11-22|Merge pull request #103 from Vonsek/pe-map-tls-delay-dirs
4aa2de80|2026-11-22|Merge branch 'main' into cpu-aot-coverage
a5d9a7d2|2026-11-21|Validate remaining sibling links before returning ambiguous in resolve_script_path
5b023b37|2026-11-21|Disambiguate duplicate shader extraction names, resolve LLVM library, and add CLI checks
bc8c6b49|2026-11-21|Document version-2 chained unwind trailer support in docs/experiments.md
a190e4e6|2026-11-21|chore: merge upstream contribution updates
409b8680|2026-11-22|Merge pull request #86 from Vonsek/cpu-aot-coverage
4607f002|2026-11-21|build: warn when fixture Python is missing
1e25edd1|2026-11-22|ffx: replay the first pass 26 LDS gate and tagged stores
b73874f9|2026-11-21|Merge branch 'main' into fix/module-validation-skip
d92f364f|2026-11-22|Merge branch 'main' into fix/dxr-update-tolerance
c385beb3|2026-11-22|Merge branch 'main' into bvh-cpu-reference
2d9dda3b|2026-11-22|Merge branch 'main' into script-duplicate-state-ids
41b12fe3|2026-11-22|Merge branch 'main' into rcas-native-toward-zero
fad34b82|2026-11-22|Merge branch 'main' into fix/call-evidence-dxcontainer-validation
e0a57d13|2026-11-22|Merge branch 'main' into test/root-signature-single-bits
347468d4|2026-11-22|Merge branch 'main' into ag/unwind-v2-chained-ranges
462bcbeb|2026-11-22|Merge branch 'main' into ag/script-path-validate-after-ambiguous
9a0ac663|2026-11-22|Merge pull request #123 from Vonsek/script-duplicate-state-ids
4cdd2644|2026-11-21|Merge branch 'main' into fix/module-validation-skip
cf3c3d2a|2026-11-21|Reject non-canonical integers, lengths and unsorted keys in decode_torrent
06c874a4|2026-11-21|tooling: split form coverage from README graphics
86c927ac|2026-11-22|Merge pull request #114 from bosseurdinternet/fix/module-validation-skip
737dec01|2026-11-22|Merge branch 'main' into rcas-native-toward-zero
ca7504f2|2026-11-22|Merge branch 'main' into codex/warn-missing-fixture-python
7f93f27d|2026-11-21|Pin relocation-free mapping away from a reserved preferred base
245b207f|2026-11-22|Merge pull request #148 from Neygen8/codex/warn-missing-fixture-python
62936492|2026-11-22|Merge branch 'main' into fix/bencode-canonical-validation
516faaf4|2026-11-22|Report candidate paths for ambiguous camera state labels
087e0ad6|2026-11-21|chore(cpu): sync the grouped allocation and jump-table experiments
7df4dfb0|2026-11-21|Assemble LLVM comparison fixture with llvm-mc for gfx1013
e4f54310|2026-11-21|Merge upstream main into tooling/progress-dashboard
c1a65f0c|2026-11-22|Merge pull request #150 from KomaGuy/fix/bencode-canonical-validation
e32a4ca2|2026-11-21|tooling: classify the jump-table and two-import CPU fixtures
b72a8730|2026-11-21|Add a minimal WaveReadLaneAt shuffle check at native and forced widths
c1533d8e|2026-11-21|Accept a git checkout of the SimpleTriangle sample in reference_triangle
03c61dca|2026-11-21|Preserve SCC updates on scalar BFE and test SCC-dependent consumers
5b1bef63|2026-11-21|Merge remote-tracking branch 'upstream/main' into tools/rdna2-tooling
a72720ee|2026-11-22|Merge branch 'main' into fix/dxr-update-tolerance
f659ac7c|2026-11-21|docs: add a contributor guide for non-Windows hosts
6a6d3019|2026-11-22|Merge pull request #112 from Holastor/fix/dxr-update-tolerance
e5b26052|2026-11-21|Check live page protection in pe_map_export
891e1962|2026-11-21|Reuse the pipeline and resident texture across BC7 mip-tail draws
c47f7286|2026-11-22|Run the source-owned pass 26 LDS fixture on WARP in CTest
99052e50|2026-11-22|Register RCAS positive and changed-coordinate WARP readbacks in CTest
801431d6|2026-11-22|Merge branch 'main' into bvh-cpu-reference
e6c944b7|2026-11-22|Merge pull request #117 from Vonsek/bvh-cpu-reference
a4866f6d|2026-11-21|Compare mapped DLL draw with a Windows load
62cfefff|2026-11-22|Merge branch 'main' into rcas-native-toward-zero
50b0c810|2026-11-22|Merge pull request #130 from danielradosa/rcas-native-toward-zero
dcb079ce|2026-11-22|Merge branch 'main' into fix/call-evidence-dxcontainer-validation
624f0987|2026-11-22|shader: complete gfx1013 floating-point comparison predicate coverage
7f50eb8e|2026-11-21|Reject trailing relocation bytes, odd block sizes and unaligned .pdata directories in pe_map
35c4287d|2026-11-22|Merge pull request #152 from rasptum/fix/reference-triangle-crlf
f078d1e8|2026-11-22|Merge pull request #142 from Vonsek/ag/unwind-v2-chained-ranges
8ffd0bca|2026-11-21|Skip unresolved None provider hops when tracing packaged forwarders
e5a519dc|2026-11-21|Add a desktop XMemVirtualQuery handler for the BVH call sites
71161bf2|2026-11-22|Merge pull request #140 from yummancito/test/root-signature-single-bits
734dfb0e|2026-11-22|Merge pull request #146 from Vonsek/ag/script-path-validate-after-ambiguous
0e9b0af3|2026-11-22|Merge pull request #131 from alexander-src/fix/call-evidence-dxcontainer-validation
345d919b|2026-11-22|Merge pull request #151 from rasptum/test/wave-shuffle-widths
4a7ae553|2026-11-22|Merge pull request #154 from sof9816/test/mapped-guest-draw
c7725d0e|2026-11-21|fix(shader): reject overlapping primary texture and constant descriptors
8ac95cf2|2026-11-22|Merge pull request #153 from Neoplayer/docs/non-windows-hosts
6d0dd7c3|2026-11-22|Merge pull request #156 from thegl0w/xmem/virtual-query-desktop
74a37958|2026-11-21|Lower equality, jbe/ja and 32-bit immediate operations in cpu_leaf_aot
b71fa93a|2026-11-22|Merge pull request #157 from Vonsek/codex/bink-group-descriptors
023d8dbe|2026-11-22|Merge pull request #158 from descent81/shader/fp32-comparison-predicates
b1ab7ccd|2026-11-22|Merge pull request #159 from KomaGuy/fix/forwarded-ambiguous-hop
583fee5f|2026-11-22|Merge pull request #160 from Vonsek/codex/pixel-binding-overlap
b62eab5b|2026-11-21|Lower x86 high-byte registers to bits 8..15 in ffx_version_query
ef2802b2|2026-11-21|feat(ffx): verify post-query working-set counter updates
0a016ecd|2026-11-21|Batch buffer_ops dispatches and avoid duplicate D3D12 device creation
f6952a3c|2026-11-21|chore(build): retain the merged descriptor test registrations
aedfc9a9|2026-11-21|chore(cpu): sync the merged descriptor producer and allocation caller
eef1069e|2026-11-21|Add synthetic (id, type) state dispatch fixture
8c774edf|2026-11-21|Register the dispatch fixture in CTest
939394ba|2026-11-21|Bind delay imports through the resolver in pe_map
31cbe52f|2026-11-21|Reject high bits in named delay-import thunks
a8c92e3a|2026-11-21|Handle the full ERA attribute word in era_heap on the desktop
25d03f50|2026-11-21|Lower JNE after TEST in cpu_chained_aot
d4dc8705|2026-11-21|Cache System32 PE export tables by content digest across intake scans
39d0b644|2026-11-21|feat(cpu): trace Bink frame plane allocation requests
b5723a85|2026-11-22|shader: track texture sample readiness across partial VM waits
a94d7037|2026-11-22|Merge pull request #119 from PrivacyOnion/bc7-tail-pipeline-reuse
2b1b9e01|2026-11-22|Merge pull request #109 from Holastor/tools/rdna2-tooling
f88346d1|2026-11-22|Merge pull request #128 from KomaGuy/fix/dispatch-aot-bfe
58de3b32|2026-11-22|Merge pull request #120 from descent81/contribution/report-ambiguous-camera-states
29fa4bf9|2026-11-21|Locate the TLS directory by VirtualAddress in intake_exe
dcd68fce|2026-11-22|Merge pull request #136 from Vonsek/codex/bink-allocation-caller
ad394e29|2026-11-22|Merge pull request #127 from dcastell1/delay-imports
3f280ddd|2026-11-22|Merge pull request #149 from descent81/ffx/pass26-first-lds
acf56908|2026-11-22|Merge pull request #138 from PrivacyOnion/cache-system32-pe-exports
777b1fb5|2026-11-22|Merge pull request #133 from MarlomMedeiros/script-dispatch-fixture
ea14ed47|2026-11-21|Assign BCPack refill bytes by each lane's read count
a7fcbfa2|2026-11-22|Merge pull request #118 from Vonsek/dxr-ray-flags
314c507e|2026-11-21|Lower gfx1013 add, mul, sqrt, rsq, floor and fract in pixel shader translation
e44af731|2026-11-22|Merge pull request #115 from Leyukaka/reloc-free-mapping
fb4a2a55|2026-11-22|Merge pull request #129 from PrivacyOnion/cpu-leaf-aot-comparisons
d41d1454|2026-11-22|Merge pull request #135 from descent81/contribution/rcas-coordinate-readback
ecb9160e|2026-11-22|Merge pull request #134 from thegl0w/era-heap-full-attributes
e601b707|2026-11-22|Merge pull request #121 from PrivacyOnion/pixel-aot-alu-ops
ccfa7038|2026-11-22|Merge pull request #137 from PrivacyOnion/batch-buffer-ops-and-single-device
ba5e808e|2026-11-21|Build a desktop xmem.dll for the package DLLs' named xmem imports
d2d8c636|2026-11-21|chore(docs): sync the working-set and grouped allocation notes
7e97af38|2026-11-21|Lower reversed float subtract and vector moves in pixel shaders
e8651294|2026-11-22|Merge main and retain both README coverage updates
bd788c01|2026-11-21|Map mutated images in child processes and require named results
fdff5e4a|2026-11-21|Merge upstream main into tooling/progress-dashboard
a3522920|2026-11-21|tooling: keep form coverage independent of fixture changes on main
68e1bac1|2026-11-21|cpu: reject FS/GS and LOCK prefixes in the shared lowering
4a128624|2026-11-22|Merge pull request #144 from bosseurdinternet/shader/basic-arithmetic
99990ef4|2026-11-22|Merge pull request #167 from raulmoracode/pe/export-page-readability
2ebbd84c|2026-11-21|test(runtime): verify page boundaries across partial decommit
178eb86d|2026-11-21|perf(textures): read only requested BC7 mip-tail blocks
16ecc0f9|2026-11-22|Merge pull request #161 from Vonsek/ag/ffx-lower-high-byte-regs
e733f0c9|2026-11-22|Merge pull request #172 from Vonsek/ag/intake-tls-by-address
7e970d49|2026-11-22|Merge pull request #168 from Vonsek/ag/cpu-chained-jne
60800558|2026-11-21|fix(script): require matching root schema and instance sizes
1e17d4ff|2026-11-22|Merge pull request #162 from Vonsek/codex/ffx-working-set-deltas
d4acadeb|2026-11-21|Serve the Bink heap shim from era_heap
1e394fe6|2026-11-22|Merge pull request #170 from Vonsek/codex/bink-frame-plane-plan
59c413e2|2026-11-21|Use versioned feature queries under the app-local Agility SDK
a07738d9|2026-11-21|fix(shader): lower unordered equality with explicit NaN checks
e27cfe97|2026-11-21|Recompile a caller that repeats one direct call
e9e61188|2026-11-21|Validate shader records and output stems in extract_shaders
509db2ab|2026-11-22|Merge pull request #181 from raulmoracode/pe/mutation-harness
b41a07cd|2026-11-21|chore(shader): sync the upstream arithmetic fixtures
c195ed59|2026-11-22|Merge pull request #177 from denn0x/cpu/repeated-direct-call
193a502b|2026-11-22|Merge pull request #184 from Vonsek/codex/runtime-heap-contract
4eda8cda|2026-11-22|Merge pull request #185 from Vonsek/codex/bc7-block-fixtures
ebbf30af|2026-11-22|Merge pull request #186 from Vonsek/codex/script-tree-validation
714defac|2026-11-22|Merge pull request #189 from raulmoracode/runtime/xmem-provider
83031b5b|2026-11-21|test(textures): compare all eight BC7 modes with D3D12
b065e3ea|2026-11-22|Merge pull request #190 from Spaghetih/cpu/lower-reject-prefixes
4329df92|2026-11-21|Lower self-XOR register zeroing in cpu_chained_aot
55f91be9|2026-11-21|fix(shader): require DXIL program magic in scene records
7232663b|2026-11-22|Merge pull request #187 from Vonsek/codex/shader-unordered-equality
551108f9|2026-11-21|fix(shader): skip symbol directives in assembly scans
ccd6ad6c|2026-11-22|Merge pull request #191 from rasptum/fix/agility-versioned-feature-queries
c0bfb547|2026-11-22|Merge pull request #193 from Vonsek/codex/bc7-all-mode-draw
633e24ec|2026-11-22|Merge pull request #195 from Vonsek/codex/scene-record-validation
d1f7c33a|2026-11-21|test(gpu): verify DXR hits on a shared triangle edge (#196)
18d57d92|2026-11-22|Merge pull request #198 from Vonsek/codex/rdna2-assembly-directives
5c661fae|2026-11-21|test(runtime): distinguish copy-on-write pages in memory queries
204e1d1c|2026-11-21|Run the D3D12 WARP checks on Windows 10 with app-local Agility SDK and WARP
ae985a38|2026-11-21|Remove the app-local WARP when OFFSCREEN_WARP_DIR is cleared
d8dc318a|2026-11-21|Keep unrelated files in WORK_DIR in the WARP toggle test
6ff9f7e2|2026-11-21|fix(tooling): accept short reads in paged resources
cc1c1a92|2026-11-21|fix(textures): draw BC7 mip chains through the final texel
cccd89c3|2026-11-21|Truncate 32-bit effective addresses in ffx_version_query
43d5842a|2026-11-22|Merge pull request #175 from Vonsek/bcpack-refill-read-order
1a88110a|2026-11-22|Merge pull request #200 from Vonsek/codex/runtime-mapped-memory
0b18bd96|2026-11-21|Require a nonnegative offset and DXBC magic in dxil_program (#201)
ca67fe57|2026-11-21|fix(shader): round negative half-export references toward zero (#202)
d5c6f9fc|2026-11-22|Merge pull request #203 from Vonsek/codex/resource-short-reads
9eb4dbe6|2026-11-22|Merge pull request #204 from Vonsek/codex/bc7-small-mips
e0bc47ee|2026-11-21|Merge upstream main into tooling/progress-dashboard
4e90a617|2026-11-21|tooling: count only gfx programs the translator accepted
40cce1fc|2026-11-21|Make 64-bit self-XOR zeroing observable in cpu_chained_zero
760593ea|2026-11-21|fix(gpu): preserve precision in scaled DXR reference rays
f02c2379|2026-11-22|Merge pull request #183 from thegl0w/xmem/shim-dll
7cd18a0d|2026-11-21|fix(tooling): normalize layout destination directories
926ca8aa|2026-11-21|Reject unknown offscreen GPU operations
1430f363|2026-11-21|feat(textures): verify fixed lanes using supplied BCPack refills
3941e9ef|2026-11-21|Report an unrecognized loader hash in dependency_paths
671a9e57|2026-11-21|Advance extract_shaders cursor by consumed deflate bytes only
88ccc525|2026-11-21|Identify failing pixel predicate cases
9f001ed7|2026-11-21|docs(contributing): link the installed build failure form
331eb8c9|2026-11-21|Allow repeated calls to the same IAT slot in cpu_call_aot
5b1df6d0|2026-11-22|cpu: verify Bink frame allocation through the shared Windows heap (#221)
216df144|2026-11-21|fix(runtime): report the native error for null query output
48fff165|2026-11-21|fix(tooling): recognize smaller zlib windows in section probes
8cd88b72|2026-11-21|fix(textures): preserve implicit LOD for rounded mip dimensions
1aa05c6a|2026-11-22|Merge pull request #220 from RonLogic/test/pixel-predicate-context
2f32b9c1|2026-11-22|Merge pull request #217 from Vonsek/codex/build-report-template
7b2cebdd|2026-11-22|Merge pull request #194 from Vonsek/ag/cpu-chained-xor-zero
f776b918|2026-11-22|Merge pull request #192 from alexander-src/fix/extract-shaders-validation
ba360f5e|2026-11-22|Merge pull request #147 from rasptum/feat/win10-agility-warp
f998ebed|2026-11-22|Merge pull request #212 from raulmoracode/tooling/unverified-loader-hash
eb7b830c|2026-11-22|Merge pull request #213 from Vonsek/codex/layout-destination-components
8f3861d5|2026-11-22|Merge pull request #225 from Vonsek/codex/section-zlib-window-sizes
dea3580f|2026-11-22|Merge pull request #205 from Vonsek/ag/ffx-lower-addr32
2882bc62|2026-11-22|Merge pull request #209 from Vonsek/codex/dxr-ray-scale-precision
6941bf6c|2026-11-22|Merge pull request #214 from Vonsek/codex/bcpack-complete-refills
0bd2ec71|2026-11-22|Merge pull request #141 from Kha73k/tooling/progress-dashboard
e8895adf|2026-11-22|Merge pull request #224 from Vonsek/codex/runtime-query-errors
6fb24041|2026-11-21|pe_inventory: name import providers as the loader looks them up
f6e0184b|2026-11-21|Ignore MASM macros/prefixes and inspect JUnit children in form_coverage
59e29ed8|2026-11-21|Lower 32-bit immediate moves to any GPR in cpu_chained_aot
c7659e6c|2026-11-21|Fix VOPC classification and inline comments in scan_unsupported_shaders
c038bc09|2026-11-21|Sign-extend SOPK immediates and preserve SALU SCC semantics in ffx_dispatch_aot
d10ab7f4|2026-11-21|cpu: reject EIP-relative operands and cover 32-bit loads and stores
c04e65a8|2026-11-22|cpu: verify a translated EXE entry and DLL call end to end (#228)
a9382b82|2026-11-21|Advance scene_records cursor by consumed deflate bytes only
6254a96e|2026-11-22|Merge pull request #219 from Vonsek/ag/cpu-repeated-import-call
f00853e2|2026-11-21|tooling: draw form coverage as a README graphic
971ae91a|2026-11-21|Recompile a caller with two different direct callees
3a54f336|2026-11-21|fix(tooling): diagnose missing or ambiguous integrity manifests
e238a653|2026-11-21|pe: start a mapped EXE with pe_map_run
39119956|2026-11-22|docs: identify shared interfaces and dependencies when claiming work (#237)
69400dba|2026-11-21|Guard short data-directory tables and require IMAGE_FILE_EXECUTABLE_IMAGE in cpu_exe_aot
f85723ee|2026-11-22|Merge pull request #223 from denn0x/cpu/distinct-direct-callees
9e82c7e9|2026-11-21|Check local Markdown links in CTest and the Linux job
a9948d4d|2026-11-21|fix(shader): exclude printable length bytes from record names
bb15e034|2026-11-21|Skip docs_links outside a Git checkout
41669e2b|2026-11-21|fix(shader): keep the NaN case explicit for NGE, NGT, NLE and NLT
644aa51c|2026-11-21|Decode 9-bit VOP SRC0 operands in disasm_rdna2 per RDNA 2 ISA
62688a5b|2026-11-22|Merge pull request #215 from RonLogic/fix/offscreen-invalid-option
9156edd9|2026-11-22|Merge pull request #226 from Vonsek/codex/bc7-npot-lod
64e8d0e4|2026-11-22|Merge pull request #229 from Vonsek/ag/cpu-chained-mov-r32-imm
48433276|2026-11-21|fix(textures): accept logical dimensions for late BC7 tails
829d6e02|2026-11-22|Merge pull request #230 from Vonsek/import-names-without-extension
7c0d4c7c|2026-11-22|Merge pull request #231 from alexander-src/fix/shader-scanner-vopc-comments
5cb4994d|2026-11-21|Translate distinct direct call targets at EXE entry
9e1f9157|2026-11-22|Merge pull request #233 from Spaghetih/cpu/lower-eip-and-addr32-memory
4f16f91f|2026-11-22|Merge pull request #235 from alexander-src/fix/scene-records-consumed-cursor
3fe72f9b|2026-11-21|fix(shader): make length-prefixed name handling explicit
ec156a0d|2026-11-22|Merge pull request #239 from Vonsek/codex/integrity-manifest-diagnostic
771e7936|2026-11-21|cpu: wrap 32-bit effective addresses in the jump-table translator
6ee1e6cd|2026-11-22|Merge pull request #240 from Vonsek/ag/cpu-exe-header-guards
72876b02|2026-11-21|Merge remote-tracking branch 'origin/main' into cpu/exe-multiple-call-targets
affd0612|2026-11-22|Merge pull request #222 from Leyukaka/docs-links
35701a63|2026-11-22|Merge pull request #236 from Kha73k/tooling/coverage-dashboard
c29c8cba|2026-11-22|Merge pull request #244 from rasptum/fix/pixel-aot-unordered-compares
238cb6b1|2026-11-22|Merge pull request #245 from Vonsek/ag/rdna2-disasm-vop-src0
1485f17a|2026-11-22|Merge pull request #234 from KomaGuy/fix/ffx-salu-sopk-scc
8ffdf5c7|2026-11-21|cpu: zero-extend 32-bit destinations of zero-count shifts
8f7457d3|2026-11-21|Scope repo unmatched-form check in form_coverage_check
3928f003|2026-11-21|Commit only payload pages for ERA heap alignments above a page
5c488798|2026-11-21|Align VOP2 opcode table with GFX10/RDNA 2 in disasm_rdna2
d9ac3b41|2026-11-22|pe: verify refused EXE startups and document caller lifetime
23d954d5|2026-11-22|Merge pull request #238 from yummancito/feat/pe-map-run-entry
79bb63d2|2026-11-21|Fix GFX10 opcodes for v_ashrrev_i32_e32 and v_lshlrev_b32_e32
7233cecc|2026-11-22|cpu: compare one EXE under Windows, pe_map and AOT (#249)
24305fe8|2026-11-21|tooling: classify the address_size, call_pair and exe_start CPU fixtures
ccf1794a|2026-11-21|tooling: classify the shift_count fixture for form coverage
6106f884|2026-11-21|tooling: classify the switch_addr32 fixture for form coverage
825cb2eb|2026-11-21|Forward thread attach and detach to mapped images
fb436695|2026-11-21|fix(textures): preserve implicit LOD across rounded BC7 views
72c5a797|2026-11-21|Translate two named DLL imports from an EXE caller
7a4e0dfe|2026-11-21|Restore 0x2f v_cvt_pkrtz_f16_f32_e32 in VOP2_OPS and LLVM fixture
8c9ee086|2026-11-21|test(gpu): name the lane a wrong WaveReadLaneAt read came from
251724e8|2026-11-21|feat(shader): lower gfx1013 floating-point truncation
02892524|2026-11-21|Report PSO readiness from shader manifests
2e514445|2026-11-21|docs: run the FSR4 dispatch translation natively on Windows
56d58331|2026-11-21|cpu: compare an EXE and two dependent DLLs under Windows and pe_map
1d271928|2026-11-21|test(shader): exercise both truncation clamp bounds
be2f0f8f|2026-11-21|shader: decode GFX10 null and ttmp scalar operands in disasm_rdna2
49a9a86e|2026-11-21|Normalize the shader manifest directory before checking paths
b90d1a0b|2026-11-21|Test release ownership, concurrent release and in-flight teardown
dad7776e|2026-11-21|feat(textures): inventory bounded ETPF descriptor tables
294bf7ce|2026-11-21|shader: clamp v_fract_f32 below 1.0 in the pixel translator
55cea41e|2026-11-21|runtime: give title graphics allocations desktop GPU addresses
ccb4018b|2026-11-21|docs(shader): describe truncation coverage and controls
05a6fe39|2026-11-21|docs: record desktop GPU addresses for title allocations
275043a9|2026-11-21|shader: lower gfx1013 floating-point ceiling
64c6e787|2026-11-22|Merge pull request #242 from Spaghetih/cpu/lower-zero-count-shift
c761f78e|2026-11-22|Merge pull request #227 from alexander-src/fix/form-coverage-masm-junit
748725c0|2026-11-22|Merge pull request #252 from Spaghetih/tooling/classify-cpu-fixtures
e5ccb54e|2026-11-22|Merge pull request #246 from Spaghetih/cpu/switch-addr32
cead20ef|2026-11-22|Merge pull request #260 from Spaghetih/docs/ffx-dispatch-native-llvm
e8549b36|2026-11-22|Merge pull request #264 from LooperSalty/shader/gfx10-null-ttmp-operands
2e448f56|2026-11-22|Merge pull request #243 from Vonsek/codex/shader-record-name-prefix
3646fdc4|2026-11-22|Merge pull request #267 from Spaghetih/shader/fract-clamp
cc19c0b5|2026-11-21|fix(shader): reject arithmetic writes to pending sample registers
7a74edf0|2026-11-22|Merge pull request #248 from Vonsek/codex/bc7-clamped-tail-extents
88bcb74a|2026-11-22|Merge pull request #272 from rasptum/test/wave-shuffle-block-misroute
1e9fa36d|2026-11-22|Merge pull request #266 from Vonsek/codex/texture-descriptor-inventory
977e0434|2026-11-22|Merge pull request #259 from RonLogic/tooling/pso-readiness
496fc28a|2026-11-22|Merge pull request #182 from Vonsek/era-heap-large-alignment
ceb55e2f|2026-11-21|runtime: reject non-base pointers before OpenExistingHeapFromAddress
9dd7a236|2026-11-21|Lift SETcc and CMOVcc in the jump-table translator
8642b844|2026-11-21|Keep the memory read of a CMOVcc whose condition folds to false
34f9eea5|2026-11-21|Condition the stale-pointer case on actual address reuse
9e5e4ded|2026-11-21|Merge remote-tracking branch 'upstream/main' into codex/pixel-straight-line-instruction
91dcc710|2026-11-21|feat(textures): map descriptor pages in equal-size raw ranges
3996e1e3|2026-11-21|shader: align scalar and VOP1 opcode tables with GFX10/RDNA 2 in disasm_rdna2
a284cb6f|2026-11-21|feat(shader): lower single-word constant buffer reads
17e73819|2026-11-21|Reject explicit bidirectional controls in tracked sources
aa7edaf7|2026-11-21|cpu: fail the startup fixture on a worker that cannot run to completion
da2bdd12|2026-11-22|Merge pull request #139 from Leyukaka/thread-notify
5203a831|2026-11-22|Merge pull request #278 from Leyukaka/bidi-check
e50afc4e|2026-11-22|Merge pull request #262 from yummancito/test/two-dll-startup-order
c352c42a|2026-11-22|Merge pull request #173 from Vonsek/cpu-switch-setcc-cmov
d77cefce|2026-11-22|Merge pull request #268 from thegl0w/gpu-alias/title-addresses
49bf7100|2026-11-22|Merge pull request #273 from Vonsek/codex/pixel-alu-destination-wait
1853669a|2026-11-22|Merge pull request #275 from Vonsek/codex/texture-raw-page-map
7cb3c9bf|2026-11-22|Merge pull request #277 from Vonsek/codex/pixel-scalar-buffer-dword
978186ba|2026-11-21|feat(textures): report verified zlib segments for descriptor pages
c832696c|2026-11-22|Merge pull request #263 from LooperSalty/shader/gfx10-scalar-vop1-opcodes
18ece112|2026-11-21|Merge remote-tracking branch 'upstream/main' into codex/pixel-straight-line-instruction
55db13d5|2026-11-21|feat(shader): track helper-lane affine sample coordinates
c3842fd7|2026-11-21|cpu: start a translated function at its entry when code below it is reached
985f0abf|2026-11-21|shader: look up 64-bit title addresses in the aliased ranges
9439902e|2026-11-21|docs: record the shader-side title address lookup
8bf76665|2026-11-21|shader: store and update with atomics through 64-bit title addresses
66131698|2026-11-21|docs: record stores and atomics through title addresses
35b03f95|2026-11-21|tests: keep gpu_alias_lookup_writes store addresses distinct
c4195752|2026-11-21|docs: record the hardware run of gpu_alias_lookup_writes
134b0d8b|2026-11-22|Add binary shader scanning and stage progress dashboard to RDNA 2 scanner
57a07b01|2026-11-22|scan_unsupported_shaders: retain failed inputs with diagnostics and separate file-level denominator tests
cf77eda5|2026-11-21|shader: add dxil-ms-xb2 to dxil-ms-dx retarget tool and fixtures
1c2755c6|2026-11-21|Lower 32-bit and 64-bit BMI1 bit-manipulation instructions
63fe9e99|2026-11-22|Merge pull request #241 from Holastor/tools/scan-progress-dashboard
0aedc4d5|2026-11-22|Merge pull request #250 from MarlomMedeiros/dxil-xb2-retarget
31cc5c56|2026-11-22|Merge pull request #258 from Vonsek/codex/pixel-straight-line-instruction
fc0f5d57|2026-11-22|Merge pull request #282 from bosseurdinternet/cpu/entry-order
d311159b|2026-11-22|Merge pull request #280 from Vonsek/codex/texture-zlib-segment
a809eb23|2026-11-21|fix(textures): honor declared zero padding in raw page maps
b3cf7839|2026-11-21|Merge remote-tracking branch 'upstream/main' into codex/pixel-helper-coordinates
1456950a|2026-11-22|tooling: keep small dashboard groups inside their boxes
6acdd903|2026-11-21|cpu: accept narrow register writes and return widths in the jump-table translator
e2f31cb8|2026-11-21|pixel_aot: lower three-operand min, max and median
b252f57d|2026-11-21|cpu: report no reachable normal return and count defined return bytes
112b6f0e|2026-11-21|feat(shader): select a named record by container digest
b5df54e4|2026-11-21|cpu: compare the lifecycle of two DLLs with the C runtime under Windows and pe_map
99193fd0|2026-11-21|feat(textures): report all descriptor indices for a texture key
84cf6c41|2026-11-21|Merge upstream main into shader/lower-pixel-ceil
0c0d00f4|2026-11-21|shader: lower gfx1013 cube coordinate instructions in pixel_aot
1c19cb63|2026-11-21|shader: validate the whole aligned DWORD before a title-address access
4dc87309|2026-11-21|docs: record whole-DWORD validation of title-address accesses
02dbbd49|2026-11-21|shader: derive the scanner's pixel ALU forms from pixel_aot
5ec7c051|2026-11-22|Merge pull request #285 from bosseurdinternet/cpu/narrow-returns
8f9c84f4|2026-11-22|Merge pull request #294 from Kha73k/tooling/dashboard-small-groups
477fa068|2026-11-22|Merge pull request #290 from Vonsek/codex/texture-raw-page-padding
110fcbb6|2026-11-22|Merge pull request #270 from lambirou/shader/lower-pixel-ceil
36bd4282|2026-11-22|Merge pull request #295 from Vonsek/codex/pixel-container-digest
094860e9|2026-11-22|Merge pull request #284 from thegl0w/gpu-alias/shader-lookup
2fa9be83|2026-11-22|Merge pull request #296 from yummancito/test/crt-dll-lifecycle
b53bfb0b|2026-11-22|Merge pull request #299 from Its3rr0rsWRLD/shader/cube
661a539c|2026-11-21|chore(textures): merge upstream into texture key lookup
345855b4|2026-11-21|Merge remote-tracking branch 'upstream/main' into codex/pixel-helper-coordinates
462287a4|2026-11-21|shader: lower gfx1013 v_fmaak_f32 and v_fmamk_f32 in pixel_aot
99b69d24|2026-11-21|shader: add the fmaak/fmamk program to the gfx1013 binary fixtures
41474a47|2026-11-21|shader: lower gfx1013 v_cvt_f32_f16 in pixel_aot
d8bf0c05|2026-11-21|pixel_aot: lower 64-bit scalar ALU and 32-bit BFE operations
77b2bbe1|2026-11-21|feat(tooling): inventory bounded AWC sound-effect metadata
3e55ad07|2026-11-21|docs: record 51 hardware coverage runs from #1
a08f34d9|2026-11-21|shader: inventory candidate programs for a named SGD5 effect
03d32119|2026-11-21|feat(tooling): validate relocated resource roots against their schemas
e0356904|2026-11-22|Merge pull request #279 from MarlomMedeiros/pixel-aot/sop1-sop2-b64
888100bf|2026-11-22|Merge pull request #298 from Vonsek/codex/texture-key-matches
1e935e83|2026-11-22|Merge pull request #300 from Vonsek/pso-scanner-pixel-forms
90136cab|2026-11-22|Merge pull request #311 from Its3rr0rsWRLD/docs/hardware-batch-51-runs
846be36d|2026-11-22|Merge pull request #292 from Its3rr0rsWRLD/shader/fmaak-fmamk
1292d15a|2026-11-22|Merge pull request #297 from Its3rr0rsWRLD/shader/cvt-f32-f16
dc21141e|2026-11-21|shader: accept padded DXIL parts in effect inventories
830da2d1|2026-11-21|shader: lower gfx1013 v_mad_f32 and v_mac_f32 as unfused, denormal-flushing multiply-add
2843f2e7|2026-11-21|shader: add the mad/mac program to the gfx1013 binary fixtures
6ee2406e|2026-11-22|Merge pull request #315 from Vonsek/codex/material-effect-evidence
a823f000|2026-11-21|Merge remote-tracking branch 'upstream/main' into codex/pixel-helper-coordinates
27a06264|2026-11-21|chore: merge upstream texture lookup documentation
61ae89e9|2026-11-21|chore: merge upstream texture lookup documentation
6543f1e5|2026-11-21|chore: merge upstream effect inventory checks
e4906ced|2026-11-21|fix(shader): preserve scalar helpers across vector arithmetic
1473e1eb|2026-11-21|Check CPUID BMI1 and register capability skip 77 in cpu_aot_bmi1
de229bb2|2026-11-21|shader: lower gfx1013 32-bit integer comparisons in pixel_aot
8f240582|2026-11-21|shader: lower gfx1013 32-bit integer min, max and med3 in pixel_aot
a51742bc|2026-11-21|shader: lower gfx1013 v_frexp_mant_f32 and v_ldexp_f32 in pixel_aot
9801b2ba|2026-11-21|textures: accept fixed-point texel snapping in magnified BC7 tail draws
4d65fc31|2026-11-21|textures: limit the tail-draw neighbor to one texel per axis and add band controls
7ead4cf6|2026-11-21|cpu: accept a fourth integer argument in R9 in the jump-table translator
e1445650|2026-11-21|cpu: make the fourth argument in R9 an explicit per-entry choice
391f932a|2026-11-22|Merge pull request #309 from bosseurdinternet/cpu/fourth-argument
542c478c|2026-11-22|Merge pull request #286 from Vonsek/ag/cpu-lower-bmi1
b860dd51|2026-11-22|Merge pull request #313 from rasptum/shader/bc7-draw-win10-warp
f0912600|2026-11-21|Lower 32-bit and 64-bit BMI2 instructions in ffx_version_query
d40f12ed|2026-11-21|Check CPUID BMI2 and register capability skip 77 in cpu_aot_bmi2
d903b2cd|2026-11-21|Lower 32-bit and 64-bit ROL, ROR, SHLD, and SHRD instructions
f9fdea07|2026-11-21|Lower CBW, CWDE, CDQE, CQO, MOVSX, and ADC in ffx_version_query
c2ccfa6d|2026-11-21|Observe the CF that ANDN and BEXTR clear, and the BEXTR length mask
e4f504fb|2026-11-21|ci: also reject the Arabic letter mark U+061C
a8f7a9aa|2026-11-21|shader: draw the s_movk_i32 sign extension in pixel_aot
33abca19|2026-11-21|Observe BLSMSK clearing ZF and the BEXTR field edges
93501779|2026-11-21|shader: re-sign retargeted DXIL containers and verify compute PSO creation
62262394|2026-11-22|Merge pull request #336 from Leyukaka/bmi1-flag-controls
60240726|2026-11-22|Merge pull request #342 from Its3rr0rsWRLD/shader/movk-fixture
e2d9f02e|2026-11-21|tests: allow 4-arg --compute-root-file in offscreen_check
bc24dfd7|2026-11-22|Merge pull request #345 from thepatrickworld/patch-1
0f2466fc|2026-11-21|gpu: skip the title-address write fixture where array atomics are inexact
55adbe09|2026-11-21|gpu: fail the title-address write fixture on inexact array atomics
35e6d0a5|2026-11-22|Observe SAR sign fill and the CF a zero-count shift keeps
c26af89d|2026-11-22|Add masked-zero immediate and ZF=1/CF=0 seed controls to rotates_shifts
44ef4d0e|2026-11-22|Emit direct write for masked-zero immediate ROL/ROR/SHLD/SHRD
d7ae01da|2026-11-22|Merge remote-tracking branch 'origin/main' into cpu/exe-multiple-call-targets
2d7323c2|2026-11-22|Reject invalid C++ import names before translation
40bf87d4|2026-11-22|Merge remote-tracking branch 'origin/main' into cpu/exe-two-named-imports
a1c75b7d|2026-11-21|docs: record RX 5600 XT hardware coverage from #1
2fc9ee43|2026-11-21|shader: accept gfx10 scheduling hints in pixel_aot
7908ece7|2026-11-22|Merge pull request #352 from Leyukaka/shift-flag-controls
9bdbb203|2026-11-22|Merge pull request #332 from rasptum/gpu/alias-lookup-win10-warp
2b93466b|2026-11-21|shader: keep s_code_end rejected and update the coverage control
06b7bd98|2026-11-22|Cover eight jump-table translator semantics that no test observed
04f48146|2026-11-21|tests: describe the rejected-form controls in check_shader_records
67fd2437|2026-11-21|docs: remove unrelated README image embeds
9ce4ecd1|2026-11-21|cpu: lower RCL, RCR, CLC, STC and CMC in the shared lifter
c1cf1dbe|2026-11-21|docs: remove extra README whitespace
0437fb92|2026-11-22|Merge pull request #378 from ghostluke69/docs/rx-5600-xt-coverage
7f7cb40e|2026-11-22|Merge pull request #372 from Leyukaka/jump-table-controls
4f6c39d5|2026-11-22|Merge pull request #362 from denn0x/shader/sopp-hints
85cd5e09|2026-11-21|cpu: lower XCHG in the shared lifter, atomically for memory
970a0d0d|2026-11-22|cpu: lower baseline 128-bit SSE/SSE2 packed conversions, compares, half moves, and scalar sqrt/min/max
ce4bb367|2026-11-22|docs: record the RDNA 4 array-atomics probe measurements
6c0c52ef|2026-11-21|build: link the CRT-free startup and lifetime fixtures at /O2
adbc413a|2026-11-22|cpu: lower AVX 128-bit and 256-bit packed floating-point instructions
a0881a15|2026-11-22|cpu: lower AVX broadcast, permute, shuffle, blend, dot, round, and vzeroall instructions
c32318a4|2026-11-22|cpu: terminate the native EXE after the entry returns
eb126f7d|2026-11-22|cpu: lower AVX conversions, compares, flag tests, and aligned/streaming moves
0fd139b9|2026-11-22|cpu: lower VEX AVX packed integer ALU instructions
09e9f509|2026-11-21|docs: fix the macOS Python quick start
091a0b45|2026-11-21|textures: check BC7 raster placement at an odd block width
5700d1ec|2026-11-22|cpu: lower VEX AVX packed saturating, min/max, avg, multiply, and sad instructions
5704bd31|2026-11-22|cpu: lower VEX AVX packed sign/zero-extend, horizontal integer, and phminposuw
1050542a|2026-11-22|Merge pull request #383 from denn0x/docs/rdna4-array-atomics
a6bf91dc|2026-11-22|Merge pull request #385 from grahamster2/build/crt-free-byte-copies
d9365370|2026-11-22|Merge pull request #396 from BZ110/docs/macos-python-quick-start
65492448|2026-11-22|Merge pull request #399 from BZ110/textures/bc7-rectangular-raster
9e27641b|2026-11-22|docs: note that SDK 10.0.26100.8249 ships a DXC that passes module-only validation
c2a56e89|2026-11-22|cpu: lower VEX AVX packed shifts, packs, unpacks, shuffles, blends, and insert/extract
66693255|2026-11-21|tooling: treat pixel disassembly failures as PSO blockers
f7932cb5|2026-11-22|cpu: lower VEX AVX scalar FP, half-register moves, vinsertps/vextractps, and vcomis*
bdd167a4|2026-11-21|runtime: give xmem graphics allocations desktop GPU addresses on attach
5e33c6a9|2026-11-21|docs: record the hardware run of gpu_alias_registry
f847956c|2026-11-21|tests: walk regions to place the registry's interior block below the first allocation
d8c6c256|2026-11-22|cpu: lower XLATB in the shared lifter
b6b3fd19|2026-11-21|docs: record the registry run at f847956 with the debug layer
5636d6ae|2026-11-21|cpu: label a conditional branch target that is its own fallthrough
33c68185|2026-11-22|Merge pull request #405 from JB2WIN/docs/sdk-26100-8249-dxc
b5a420ec|2026-11-22|Merge pull request #410 from aadi-varc/tooling/pso-disassembly-blocker
eab82c20|2026-11-22|Merge pull request #413 from grahamster2/cpu/jcc-to-next-label
e5e00797|2026-11-22|Merge pull request #394 from RonLogic/cpu/native-exe-termination
90231925|2026-11-22|shader: lower quad-local DPP16 and DPP8 binary16 ops in pixel_aot
ad423acc|2026-11-22|cpu: accept stack frames of call-free functions in the jump-table translator
3b88c14f|2026-11-22|shader: lower quad-local DPP16 and DPP8 conversion ops in pixel_aot
dd963354|2026-11-22|cpu: lower an opt-in x87 leaf profile with FADDP
ac48a50a|2026-11-22|Merge pull request #415 from denn0x/cpu/lower-xlatb
b7bb9d1e|2026-11-22|shader: lower quad-local DPP16 and DPP8 extra f32, carry, and dot ops in pixel_aot
7cf82212|2026-11-21|cpu: treat a three-operand IMUL destination as written, not read
cace6d20|2026-11-22|cpu: save the full x87 state around the x87_add harness call
086e446c|2026-11-21|cpu: translate direct calls and tail jumps between functions in the jump-table translator
a75b1a99|2026-11-21|docs: record the GTX 1060 and Win10 WARP array-atomics measurements
5654af9e|2026-11-22|shader: lower quad-local DPP16 and DPP8 v_movrel ops in pixel_aot
acef834f|2026-11-22|Merge pull request #422 from denn0x/cpu/lower-x87-add-profile
db3cb504|2026-11-22|Merge pull request #424 from grahamster2/cpu/imul3-dest-write
c2fc84cd|2026-11-22|Merge pull request #427 from abilliontokens/docs/array-atomics-gtx1060
649e4608|2026-11-22|cpu: lower FMUL forms in the x87 leaf profile
165a66f8|2026-11-21|cpu: accept only primary runtime functions and code exports as call entries
ed260fab|2026-11-22|cpu: lower LOOP, LOOPE, LOOPNE, JRCXZ and their ECX forms
e0081e8f|2026-11-21|shader: decode SOPC scalar comparisons in disasm_rdna2
55d5f5f8|2026-11-22|Merge pull request #431 from denn0x/cpu/lower-x87-mul
97016d96|2026-11-22|Merge pull request #432 from Jawschamp/shader/decode-sopc
b9935bc8|2026-11-21|cpu: treat frame bytes below a call's home space as overwritten by the call
4b2e125f|2026-11-21|cpu: bind call targets without unwind data through an opt-in range manifest
6335f4ab|2026-11-22|cpu: lower FDIV and FDIVR forms in the x87 leaf profile
2fcf6d36|2026-11-22|Merge pull request #247 from RonLogic/cpu/exe-multiple-call-targets
84d30233|2026-11-22|Merge pull request #305 from Vonsek/codex/audio-format-metadata
b3d90658|2026-11-22|Merge pull request #316 from Vonsek/codex/resource-root-schema
a7836ce0|2026-11-22|Merge pull request #283 from Vonsek/codex/pixel-helper-coordinates
01b58df0|2026-11-22|Merge pull request #291 from Vonsek/ag/cpu-lower-rotates-shifts
20a1b92f|2026-11-22|Merge pull request #288 from Its3rr0rsWRLD/shader/mad-mac-unfused
992306bd|2026-11-21|runtime: test every graphics memory type at the granularity boundary (#365)
70bb4199|2026-11-22|cpu: lower MOVBE in the shared lifter (#412)
1606f7ba|2026-11-22|docs: align non-Windows host checks with CI workflow (#411)
5b2fd02d|2026-11-21|cpu: lower x64 string instructions in the shared lifter
73ed4b7d|2026-11-22|shader: preserve pixel hardware capability skips (#322)
3f77ffd4|2026-11-21|cpu: lower overflow and parity conditions
0b2e8b30|2026-11-21|cpu: record decoded condition forms for coverage
292281d9|2026-11-22|gpu: check that the built array-atomics probe keeps nonUniformIndex
58174084|2026-11-21|Merge origin/main into cpu/switch-direct-calls-420
5f7a0e01|2026-11-21|Merge cpu/switch-direct-calls-420 into cpu/switch-pdata-less-callees
8e3158c1|2026-11-21|shader: lower vector bitfield, population count and bit scans
e9ab5b6a|2026-11-22|ci: run the BC7 block reference check on Linux (#348)
e2a30e34|2026-11-22|Merge pull request #306 from alexander-src/shader/int-compare-pixel-aot
7fb6a9ad|2026-11-22|cpu: forget stack bytes released by ADD RSP and POP
5ec180f8|2026-11-22|Merge remote-tracking branch 'origin/main' into cpu/stack-frames
1088c1bc|2026-11-22|Reject non-label state ID attributes in the script probe and replay (#174)
2a0c9777|2026-11-22|cpu: translate RIP-relative data against an explicitly bound image mapping
a094c970|2026-11-22|Cover five pixel translator semantics that no draw observed (#384)
680ff0e3|2026-11-22|shader: lower s_nand_b64, s_nor_b64 and s_xnor_b64 in pixel_aot (#400)
7739726b|2026-11-22|Let the hardware checks select a GPU by name on multi-GPU PCs (#116)
3f02f4d3|2026-11-22|Merge pull request #330 from Joacoorandom/cpu/overflow-parity-conditions
fb4fb85b|2026-11-22|Merge pull request #433 from denn0x/cpu/lower-x87-div
68fd84ef|2026-11-21|shader: make the pixel_aot f32 denormal mode explicit and accept NaN by class
fa924981|2026-11-21|shader: assert mode independence for fixtures recorded outside the preserve assumption
38270b45|2026-11-21|shader: export only the enabled channels in pixel_aot compressed exports
4b394044|2026-11-21|shader: accept partial channel masks on uncompressed pixel_aot exports
bf498cd1|2026-11-21|shader: pin compressed EXP enable-mask decoding across all 16 raw EN values
4d26cc4b|2026-11-22|cpu: lower FSUB, FSUBR and the remaining FADD forms in the x87 profile
a0945766|2026-11-21|Merge #420 (7fb6a9a frame lifetime) into cpu/switch-direct-calls-420
843154f6|2026-11-21|cpu: clear only the callee's home space at a call again
1ecd897b|2026-11-22|Merge remote-tracking branch 'origin/main' into cpu/stack-frames
01fb2566|2026-11-21|Merge cpu/switch-direct-calls-420 into cpu/switch-pdata-less-callees
bf446a22|2026-11-21|shader: lower gfx1013 v_cmp_class_f32 in pixel_aot
8afb68c5|2026-11-22|Merge pull request #379 from denn0x/cpu/lower-carry-rotates
a40fa0e7|2026-11-22|Merge pull request #403 from denn0x/cpu/lower-rcx-branches
b70c3de7|2026-11-22|Merge pull request #420 from bosseurdinternet/cpu/stack-frames
d4708725|2026-11-22|Merge remote-tracking branch 'origin/main' into HEAD
6916d9a4|2026-11-22|Merge commit 'd470872' into HEAD
71ddca95|2026-11-22|Merge pull request #293 from Its3rr0rsWRLD/shader/cmp-class
1ec51e0f|2026-11-22|Merge pull request #368 from denn0x/cpu/lower-string-instructions
d3cd043a|2026-11-21|shader: lower two-word constant buffer reads in the pixel translator
def98584|2026-11-22|Merge remote-tracking branch 'origin/main' into HEAD
490a419e|2026-11-22|Merge commit 'def9858' into HEAD
93c5256b|2026-11-22|Merge pull request #440 from denn0x/cpu/lower-x87-sub
1768ae07|2026-11-22|Merge pull request #425 from grahamster2/cpu/switch-direct-calls-420
fb7547cf|2026-11-22|Merge pull request #430 from grahamster2/cpu/switch-pdata-less-callees
dec705fc|2026-11-22|cpu: exercise a framed call to a bound leaf in the three-way EXE comparison (#441)
22366680|2026-11-22|cpu: lower x87 register moves, FCHS and FABS in the x87 profile
3a0f33b8|2026-11-22|Merge pull request #386 from denn0x/cpu/lower-xchg
10060ecf|2026-11-21|cpu: lift RDX:RAX multiplies, divides and sign extensions in the jump-table translator
b5f9042b|2026-11-22|cpu: lower 32-bit immediate RSP adjustments in cpu_chained_aot (#255)
bec2521b|2026-11-22|cpu: lower signed 32-bit comparison branches in cpu_leaf_aot (#261)
318d9c3f|2026-11-21|tools: report the attribute word at each xmem call site (#318)
41bc5405|2026-11-22|textures: check BC7 4 KiB surface linearization against a bit-order oracle
c1ca45bb|2026-11-22|Merge pull request #346 from alexander-src/shader/dxil-retarget-signing
9c86fa1f|2026-11-22|coverage: register new CPU fixtures in their own MASM sources (#444)
2e7a3a95|2026-11-22|Merge main and register sign-extension coverage in its MASM fixture
7cd97d9b|2026-11-21|ci: build and test the PC suite at /O2 as well
4876b058|2026-11-22|Read export names as stored in the PE inventory (#124)
ba7d6b75|2026-11-22|cpu: confirm the worker is alive before the entry returns (#388)
a38f3297|2026-11-22|pe_map: treat a relocation directory at address 0 as absent (#171)
a0c01137|2026-11-22|Merge pull request #326 from Vonsek/ag/cpu-lower-sign-ext-adc
93e5f8e6|2026-11-22|ci: redraw the coverage dashboard on each push to main (#302)
aef19a78|2026-11-22|Resolve KnownDLL names from System32 before package images (#178)
a2ecabb6|2026-11-22|runtime: add desktop d3d12_xs.dll shim for root-signature imports (#351)
1f31fec4|2026-11-22|Merge pull request #304 from rasptum/shader/scalar-buffer-loads
0b2a565e|2026-11-22|Reconcile partial VM waits with per-register move hazards
9105a0da|2026-11-22|Merge remote-tracking branch 'origin/main' into codex/review-308
467bb501|2026-11-22|ffx: define float-to-integer conversion at NaN and saturation boundaries
46f8ab07|2026-11-22|Merge pull request #445 from roansystemlabs/textures/bc7-tile-swizzle
6fa88783|2026-11-22|Merge pull request #447 from grahamster2/ci/release-o2
c29586d1|2026-11-22|Merge pull request #443 from denn0x/cpu/lower-x87-moves
718d6838|2026-11-22|Merge pull request #308 from alexander-src/shader/int-minmax-med3-pixel-aot
d38d2602|2026-11-22|cpu: lower the x87 constant loads in the x87 profile
fb19768c|2026-11-22|textures: verify the published mode-6 BCPack vector through GPU readback
78829ab9|2026-11-22|Merge remote-tracking branch 'origin/main' into HEAD
2e7dfa47|2026-11-22|Merge pull request #450 from denn0x/cpu/lower-x87-constants
56f01c61|2026-11-22|cpu: lower FILD, FIST, FISTP and FISTTP in the x87 profile
401e2368|2026-11-22|Merge pull request #416 from bosseurdinternet/cpu/image-data
04790b3a|2026-11-22|Merge remote-tracking branch 'origin/main' into codex/review-310
9f1fb5c9|2026-11-22|cpu: run a relocated global read/write through the three-way EXE chain (#451)
ab5b8d4c|2026-11-22|Merge pull request #310 from alexander-src/shader/frexp-ldexp-pixel-aot
ea6cb2f7|2026-11-22|ffx: validate and dispatch the folded positive-infinity UAV store
b7cce635|2026-11-22|Merge pull request #449 from descent81/ffx/defined-float-to-int
029faed7|2026-11-22|Merge pull request #452 from denn0x/cpu/lower-x87-integers
7cca3d15|2026-11-22|Merge pull request #453 from descent81/textures/mode6-fixed-block
21a65e50|2026-11-21|runtime: create the GPU registry without throwing, and test allocation failure
75543e19|2026-11-22|cpu: raise a divide error even when quotient and remainder are unused
6804d0dd|2026-11-22|cpu: lower FCOMI, FCOMIP, FUCOMI and FUCOMIP in the x87 profile
44b37078|2026-11-22|cpu: lower FSQRT in the x87 profile
54da9ddc|2026-11-22|cpu: lower FRNDINT in the x87 profile
348b7855|2026-11-21|docs: record the registry's allocation-failure run at 21a65e5
3a806d38|2026-11-22|cpu: recompile a multi-unit UCRT console program from its PE entry (#455)
c4716a9e|2026-11-22|Exercise VOP3 fract source modifiers in pixel translation and readback
8fc1205d|2026-11-22|Merge pull request #327 from thegl0w/gpu-alias/device-attach
5131d992|2026-11-22|Merge pull request #456 from denn0x/cpu/lower-x87-compare
5d776752|2026-11-22|tooling: validate relocated resource root block-map pointer
efffd904|2026-11-22|cpu: lower FXTRACT in the x87 profile
2bdb504d|2026-11-22|cpu: translate a multi-record archive reader from standard startup (#459)
c2a3d6c5|2026-11-22|cpu: lower FSCALE in the x87 profile
6c20087e|2026-11-22|Merge pull request #321 from bosseurdinternet/cpu/wide-arithmetic
b4384e80|2026-11-22|Merge pull request #454 from agilesh08/fix/resource-root-block-map-overlap
917b52d9|2026-11-22|Merge pull request #457 from descent81/shader/coverage-fract-e64
d5af443c|2026-11-22|Merge pull request #460 from denn0x/cpu/lower-x87-sqrt
0f52c782|2026-11-22|cpu: translate the blast decoder and dispatch its callbacks (#461)
96099843|2026-11-22|docs: record strict and fitted TTPF exports
97179859|2026-11-22|docs: list BCPack format coverage and decoder tests
5ee73e68|2026-11-22|Merge pull request #462 from denn0x/cpu/lower-x87-rndint
de7c8ed2|2026-11-22|Merge pull request #435 from roansystemlabs/gpu/369-nonuniform-atomic-guard
9dc9dd6e|2026-11-22|docs: preserve the twelve synthetic BCPack checks and fixed byte pairs
aa1ade85|2026-11-22|shader: reject DXIL VBR operands above 64 bits
17810653|2026-11-22|shader: check FP32 exponent edge cases on Vulkan
3bf8312a|2026-11-22|tooling: normalize malformed zlib errors in paged resource reader
79630430|2026-11-22|docs: define CPU and GPU review responsibilities (#464)
e3508f5c|2026-11-22|docs: make the measured BCPack exports reproducible offline
01930572|2026-11-22|docs: identify the coverage snapshot and link current CI charts
8258d2db|2026-11-22|Merge pull request #465 from denn0x/cpu/lower-x87-xtract
3cfa73f7|2026-11-22|Merge pull request #466 from agilesh08/fix/dxil-vbr-64bit-bound
f714e34d|2026-11-22|Merge pull request #468 from agilesh08/fix/paged-resource-zlib-error
2d1449cf|2026-11-22|docs: record RX 7900 XTX and Intel Arc B390 hardware runs from #1
25a00a73|2026-11-22|Merge pull request #469 from R0Y00/codex/chart-caption
945919b9|2026-11-22|pe: reject PE32+ import-name thunks with nonzero reserved bits
3b366ec5|2026-11-22|shader: lower gfx1013 FREXP exponent instructions
be79f380|2026-11-22|docs: limit the TTPF report to two measured exports
86dff9f0|2026-11-22|cpu: compare blast window continuation and callback refusal (#470)
80a4acb9|2026-11-22|tooling: report individual texture descriptors
33007145|2026-11-22|Merge pull request #463 from wish131400/docs/ttpf-dds-measurements
80bbc07a|2026-11-22|Merge pull request #467 from agilesh08/fix/pe32plus-import-reserved-bits
18d9ea0b|2026-11-22|Merge pull request #471 from denn0x/cpu/lower-x87-scale
64e4e4eb|2026-11-22|Merge pull request #472 from stan495/docs/record-7900xtx-b390-coverage
ff6841f9|2026-11-22|Merge pull request #473 from venetanji/shader/vulkan-fp32-exponent-check
13be1841|2026-11-22|Merge pull request #475 from venetanji/textures/etpf-descriptor-detail
3310cb8e|2026-11-22|shader: bound and verify two-exit FSSM pixel programs (#477)
5a2c7822|2026-11-22|docs: surface verified progress and hardware report intake (#479)
cbfc7ce7|2026-11-22|Merge remote-tracking branch 'origin/main' into codex/review-251
e193712f|2026-11-22|shader: translate unsampled pixel colors and masked MRT0 exports (#480)
8da230cb|2026-11-22|Merge remote-tracking branch 'origin/main' into codex/clear-169
abbc80d3|2026-11-22|Merge remote-tracking branch 'origin/main' into codex/clear-176
01e2539d|2026-11-22|Merge remote-tracking branch 'origin/main' into codex/inspect-122
dc61f113|2026-11-22|Merge pull request #169 from Vonsek/pixel-aot-min3-max3-med3
4e42575a|2026-11-22|Merge pull request #251 from Vonsek/ag/rdna2-disasm-vop2-opcodes
5bbaf853|2026-11-22|Merge remote-tracking branch 'origin/main' into codex/clear-287
601b4f78|2026-11-22|Merge remote-tracking branch 'origin/main' into codex/clear-176
fae747ae|2026-11-22|Merge pull request #122 from PrivacyOnion/pe-map-reloc-pdata-bounds
9c4b9afb|2026-11-22|Merge pull request #287 from Vonsek/ag/cpu-lower-bmi2
7efa513e|2026-11-22|Merge pull request #176 from descent81/shader/partial-texture-waits
39d7de63|2026-11-22|Check gfx1013 VOP3 reciprocal and reciprocal square root readback (#482)
4f34df63|2026-11-22|shader: translate and dispatch fixed and indexed compute UAV stores (#483)
933cbad0|2026-11-22|shader: translate four-component entry interpolation in unsampled pixels (#484)
04a260a3|2026-11-22|build: select a VS instance with the x64 C++ toolset among all installs (#485)
9f76745c|2026-11-22|cpu: lower the ZF and CF forms of FCMOVcc in the x87 profile (#476)
a8e2d398|2026-11-21|tests: discard frexp result explicitly in FFX scratch check (#489)
3ecb8ed5|2026-11-22|cpu: lower 32-bit and 64-bit integer bit-scan, bit-count, byte-swap, and bit-test instructions (#289)
c41967da|2026-11-22|Integrate current main and register the verified CPU fixture at its source
fceadbb3|2026-11-22|Integrate current main and register the verified CPU fixture at its source
afc57733|2026-11-22|Integrate current main and register the verified CPU fixture at its source
a44f2db0|2026-11-22|Integrate current main and register the verified CPU fixture at its source
c90ab211|2026-11-22|Integrate current main and register the verified CPU fixture at its source
19190386|2026-11-22|Integrate current main and register the verified CPU fixture at its source
4c29ddec|2026-11-22|Integrate current main and register the verified CPU fixture at its source
59e7dac6|2026-11-22|Integrate current main and register the verified CPU fixture at its source
235a108b|2026-11-22|Integrate current main and register the verified CPU fixture at its source
be67e6b4|2026-11-22|Translate two-table depth-only pixel programs (#492)
3b07743c|2026-11-22|Specialize a wave64 mesh entry through D3D12 export and draw (#491)
d3678618|2026-11-22|Resolve forwarded exports through the host resolver (#208)
7f13b7fe|2026-11-22|Compare DXR opacity and non-opaque candidate handling with a CPU reference (#216)
0f6a7577|2026-11-21|cpu: terminate the native CRC line fixture (#487)
f85b3270|2026-11-22|Integrate current pixel lowering and retain both draw suites
185b9d6c|2026-11-22|docs: record Linux translation-layer RADV and NVIDIA runs from #1 (#486)
587fbd8d|2026-11-21|cpu: compare startup TLS values inside callbacks (#426)
a8b130df|2026-11-22|cpu: keep memory reads whose value is unused in the jump-table translator (#437)
8566ecc2|2026-11-22|Preserve pixel entry signatures and sample waits across the current render paths
c23f493e|2026-11-22|Preserve pixel entry signatures and sample waits across the current render paths
088eb485|2026-11-22|Preserve pixel entry signatures and sample waits across the current render paths
11fbd0f0|2026-11-21|docs: describe an msvc-wine compile check for Linux hosts (#490)
83ff4f1b|2026-11-22|Integrate current pixel lowering and retain both draw suites
3c042c92|2026-11-22|Integrate current pixel lowering and retain both draw suites
2d165984|2026-11-22|Integrate current pixel lowering and retain both draw suites
82efaaa1|2026-11-22|Integrate current pixel lowering and retain both draw suites
0df85756|2026-11-22|pixel_aot: support export counter synchronization and verify subrev ALU lowering (#271)
329510f1|2026-11-22|shader: lower gfx1013 v_cvt_f32_ubyte0-3 in pixel_aot (#353)
2424584d|2026-11-22|Translate indexed texture coverage and verify per-pixel survival (#497)
85cf18a2|2026-11-21|docs: record 21 hardware runs from #1 after #311 (#481)
de331f4f|2026-11-22|Integrate current pixel lowering and retain both draw suites
c834f300|2026-11-22|cpu: add AVX2 leaf AOT translator covering all 28 forms (#323)
a00bfb8b|2026-11-22|Translate indexed mesh vertex fetches and verify visible geometry (#498)
5a16ee95|2026-11-22|pixel_aot: lower 32-bit bitwise, shift and integer conversion operations (#232)
6f48e010|2026-11-22|pe: keep section protections from faulting pe_map during the map (#418)
d3ddbe23|2026-11-22|docs: record that no code in the inspected Bink image produces odd tagged strides (#496)
643b14b9|2026-11-22|cpu: add AVX packed-integer and bit-move leaf AOT translators (#373)
f3b0d191|2026-11-22|cpu: lower the integer-operand arithmetic of the x87 profile (#499)
d24bb44e|2026-11-22|shader: lower RDNA2 scalar comparison instructions (#494)
7e4525d1|2026-11-22|Integrate the two-provider EXE comparison with the current startup and frame profiles
4bf3dc09|2026-11-22|shader: lower RDNA2 integer VOP2 operations (#495)
a72e2e7d|2026-11-22|cpu: lower all 7 SHA-NI instructions (#381)
265e2250|2026-11-22|Merge pull request #502 from R0Y00/codex/integrate-pr-256
3c5802a2|2026-11-22|shader: lower SDWA forms of the f32 comparisons in pixel_aot (#337)
3f59c6a1|2026-11-22|shader: draw the e64 forms that pixel_aot accepts without a fixture (#340)
40fb465a|2026-11-22|cpu: translate import tail jumps through an explicit binding (#343)
a46dac37|2026-11-22|shader: lower packed binary16 VOP3P arithmetic in pixel_aot (#376)
31b0834d|2026-11-22|List uncovered reference forms with their fixture evidence (#504)
10b891df|2026-11-22|shader: lower partial dmask image_sample and non-zero texture offsets in pixel_aot
14e54563|2026-11-22|docs: record four hardware runs from #1 after #481 (#501)
aeb616a1|2026-11-22|cpu: test EXE startup through desktop shims
fb0315a0|2026-11-22|shader: lower image_sample_lz, image_sample_l and image_sample_b in pixel_aot
f3963bed|2026-11-22|docs: record RX 7900 XTX array-atomics measurement in #369 (#507)
58898762|2026-11-22|Make the README a guide to verified results and first runs (#509)
72f27a60|2026-11-22|scan_unsupported_shaders: track 32-bit integer operations from #232 (#505)
a4ab3a31|2026-11-22|shader: lower mixed-precision VOP3P FMA in pixel_aot (#389)
221eae31|2026-11-22|shader: lower 32-bit scalar ALU, comparisons and SCC selects in pixel_aot (#257)
4f48746a|2026-11-22|cpu: lift VEX.128 16-bit packed-integer leaf functions (#325)
4237a898|2026-11-22|cpu: lift scalar SSE/SSE2 leaf functions (#269)
80a5dccf|2026-11-22|docs: show coverage and contributor guidance on the homepage (#519)
830a7202|2026-11-22|cpu: lower 32-bit and 64-bit MUL, DIV, IDIV, XADD, and CMPXCHG instructions (#331)
1aafb0f8|2026-11-22|shader: lower VOP3P dot-product instructions in pixel_aot (#391)
90e70aed|2026-11-22|finally pushing omfg (#333)
d4ec9426|2026-11-22|cpu: lower unsigned and equality CMOVcc and SETcc forms (#328)
81bbbf6e|2026-11-22|Merge pull request #511 from Jess-Doit/test/runtime-shim-startup
f94c2549|2026-11-22|shader: lower gfx1013 v_rcp_iflag_f32 in pixel_aot (#350)
184b84d3|2026-11-22|shader: lower packed 16-bit integer VOP3P instructions in pixel_aot (#397)
a1f57a91|2026-11-22|shader: lower RGB-only pixel texture samples (#303)
2620000f|2026-11-22|cpu: lower 32-bit and 64-bit ADX instructions (ADCX, ADOX) and SETO (#335)
026460ad|2026-11-22|cpu: lower FNSTCW, FSTCW and WAIT in the x87 profile (#513)
39cf4c4b|2026-11-22|cpu: drive D3D12 compute from the linked shim startup entry (#525)
cfa7fcee|2026-11-22|shader: lower gfx1013 v_cvt_off_f32_i4 in pixel_aot (#356)
26544c69|2026-11-22|shader: lower SDWA moves and half widening in pixel_aot (#329)
d2628abb|2026-11-22|shader: lower gfx1013 v_ceil_f32 and v_rndne_f32 in pixel_aot (#274)
6db5f2bf|2026-11-22|cpu: lower AVX2 integer broadcasts and variable per-lane shifts (#339)
7e9d42ea|2026-11-22|Batch e64 pixel readbacks per shader and isolate case failures (#508)
0b20572b|2026-11-22|Measure fixed-profile shader coverage and packed RGB readback (#527)
914d2412|2026-11-22|shader: lower gfx1013 v_cvt_f16_f32 in pixel_aot (#324)
cd430642|2026-11-22|pe: cover mixed-case forwarder targets (#516)
54e82b05|2026-11-22|cpu: add malformed legacy-prefix VEX negative controls for BMI2
0ba2636c|2026-11-22|shader: lower gfx1013 16-bit integer comparisons in pixel_aot (#524)
c849f7aa|2026-11-22|feat(shader): lower explicit level-zero texture samples (#320)
119bdd68|2026-11-22|cpu: add a checked x87 entry that follows FLDCW (#530)
62c965c1|2026-11-22|shader: lower SDWA forms of exact f32 arithmetic in pixel_aot (#334)
1181d44c|2026-11-22|cpu: compare implicit TLS values on main and worker threads (#523)
f486c5e9|2026-11-22|shader: lower gfx1013 v_sin_f32 and v_cos_f32 in pixel_aot (#357)
eefc3f86|2026-11-22|shader: fix RDNA2 EXP, VINTRP, SMEM, MIMG and literal decoding in disasm_rdna2 (#517)
aba0555c|2026-11-22|shader: lower gfx1013 v_rndne_f32 in pixel_aot (#512)
1fda3aca|2026-11-22|shader: add dxil_retarget_probe_hardware test for GPU signing and dispatch (#521)
690c8d8a|2026-11-22|Lower scalar byte and halfword sign extension and verify preserved SCC (#533)
4063e396|2026-11-22|shader: trace the first unresolved pixel constant-buffer read (#536)
92e80ed9|2026-11-22|cpu: clear OF in the x87 FCOMI lowering (#534)
1ecdc953|2026-11-22|scan_unsupported_shaders: track lowered scalar, VOP3P, SDWA and conversion ops (#538)
64dbda26|2026-11-22|shader: support multi-store and constant-SGPR compute kernels in compute_store_aot (#539)
f68dd404|2026-11-22|shader: make the pixel_aot f32 denormal mode explicit and accept NaN by class (#363)
22064113|2026-11-22|Lower 14 128-bit SSE4.1 integer vector instructions in ffx_version_query (#359)
af977503|2026-11-22|cpu: lower FCOM, FUCOM and FNSTSW AX with a status-word consumer check (#535)
bcfd4567|2026-11-22|Lower all 12 SSE4.1 pmovsx/pmovzx vector extension instructions (#364)
075413d7|2026-11-22|shader: lower gfx1013 v_cvt_f32_u32 and v_cvt_f32_i32 in pixel_aot (#392)
7d682b53|2026-11-22|shader: lower plain e32 pixel float-to-uint and verify all result bits (#545)
4dde5ff7|2026-11-22|cpu: lower baseline 128-bit SSE2 saturating, multiply, min/max, avg, and sad instructions (#367)
e55c593a|2026-11-22|shader: lower the e64 integer-to-f32 conversions with clamp in pixel_aot (#417)
7217bf5e|2026-11-22|cpu: lower FNINIT and FINIT in the checked x87 entry
b48a7c6d|2026-11-22|Link translated indexed mesh and buffer-color pixel stages in one verified draw (#544)
e6cc8195|2026-11-22|Translate mesh PARAM0 and verify interpolated runtime vertex colors (#546)
5460a9a7|2026-11-22|shader: lower gfx1013 v_cvt_u32_f32 and v_cvt_i32_f32 in pixel_aot (#438)
a0c5b7bc|2026-11-22|tests: describe the SOPK support test instead of addressing AI agents (#547)
0a9f4b4e|2026-11-22|pixel_aot: lower integer-to-float conversions v_cvt_f32_u32 and v_cvt_f32_i32 (#529)
b515663d|2026-11-22|disasm_rdna2: decode GFX10 VOP3P packed instructions and sync scanner (#537)
aefed61b|2026-11-22|shader: lower quad-local DPP16 and DPP8 f32/move/select ops in pixel_aot (#408)
1e46451a|2026-11-22|Translate nested direct calls from EXE entry (#253)
96e836d3|2026-11-22|Lower all three SOP2 scalar halfword pack instructions (#550)
670734ad|2026-11-22|cpu: lower legacy and 128-bit VEX AES-NI and PCLMULQDQ instructions (#380)
37ef2c82|2026-11-22|shader: map a bounded SPIR-V address window to D3D12 storage buffers (#553)
0f8e7faa|2026-11-22|cpu: lower all 24 VEX-encoded scalar FMA3 instructions (ss/sd) (#347)
3ba3472b|2026-11-22|shader: lower quad-local DPP16 and DPP8 32-bit integer/bitwise/shift ops in pixel_aot (#414)
f318cc2b|2026-11-22|Batch floating-point predicate readback cases per shader (#556)
72ea0158|2026-11-22|cpu: lower baseline 128-bit SSE2 packed integer ALU instructions (#366)
579f58f0|2026-11-22|Verify desktop pixel positions in a shifted viewport (#552)
9a773f36|2026-11-22|Merge origin/main (579f58f) into shader/dpp-quad-f16-ops
c4d5e8ee|2026-11-22|test: verify exchanged CBVs in a translated mesh and pixel draw (#559)
915a9eee|2026-11-22|shader: record the pixel_aot binary16 denormal mode and draw an f16 subnormal control (#549)
b1be506d|2026-11-22|cpu: lower SSSE3/SSE4.1/SSE4.2 insert/extract, ptest, misc integer, and crc32 (#377)
d3586dd1|2026-11-22|Cover eight VEX word translator semantics that no test observed (#554)
3f745ea2|2026-11-22|Verify mesh and pixel readback after swapping CBV descriptors (#557)
d8968679|2026-11-22|cpu: skip native ADOX in cpu_aot_x87_compare on hosts without ADX (#563)
96fa78f1|2026-11-22|tests: use mode="any" for dpp_f16 draw fixture
f9fb4a2f|2026-11-22|shader: lower RDNA2 32-bit scalar complement logic in pixel_aot (#561)
b351edcd|2026-11-22|Verify a GPU compute buffer consumed by translated pixels (#555)
d8c6e90c|2026-11-22|pixel_aot: lower vector no-op v_nop and sync scanner (#565)
16cd504d|2026-11-22|shader: batch the integer and SDWA predicate draws in pixel_aot (#564)
14f2d304|2026-11-22|textures: reject invalid BC7 mode and negative partition indices (#566)
909b5b08|2026-11-22|ffx: reject unsupported ALU and packed modifiers in dispatch AOT (#568)
079120cc|2026-11-22|cpu: model PF of FCOMI and lower FCMOVU and FCMOVNU in the x87 profile (#548)
046d033c|2026-11-22|shader: lower RDNA2 vector reverse subtract v_subrev_nc_u32 in pixel_aot (#570)
b7a05976|2026-11-22|shader: hoist DPP quad swizzles and use mode="any" for dpp fixture (#569)
d4c76565|2026-11-22|Translate bounded runtime constant loads in compute stores (#558)
68723abb|2026-11-22|Cover eight scalar SSE translator semantics that no test observed (#577)
bbebcad0|2026-11-22|shader: lower gfx1013 24-bit integer multiplies in pixel_aot (#542)
2b8bdc6b|2026-11-22|tooling: inventory bounded AWC music bank metadata
81afe09d|2026-11-22|disasm_rdna2: decode 32-bit VOPC comparison instructions (#571)
475ee7de|2026-11-22|Cover the s_pack low-half and high-half masks and the DPP16 src1 neg modifier (#578)
70730155|2026-11-22|docs: record Intel Core Ultra 7 358H measurement for cpu_aot_x87_add (#581)
1a4c7288|2026-11-22|shader: lower gfx1013 SOPK s_cmpk and s_cmovk_i32 in pixel_aot (#573)
f73522f8|2026-11-22|pe: validate forwarded export ordinals in pe_inventory (#567)
6a8bab31|2026-11-22|cpu: translate ordinary calls of a bound import (#572)
19f5e428|2026-11-22|shader: lower gfx1013 SOP1 s_abs_i32, s_cmov_b32 and s_bitset ops in pixel_aot (#574)
f8d11ad6|2026-11-22|Cover five integer and float conversion paths that no draw observed (#585)
f3aeede1|2026-11-22|Merge main and preserve vector-bit coverage alongside current shader tests
192a6429|2026-11-22|docs: record Intel Core Ultra 7 358H measurements for x87 mul, div and sub (#586)
8ae13778|2026-11-22|docs: record Intel Core Ultra 7 358H measurements for x87 moves, constants and integers (#588)
c8a3fd96|2026-11-22|docs: record Intel Core Ultra 7 358H measurements for x87 sqrt, rndint, xtract and scale (#591)
d9382c3e|2026-11-22|cpu: lower all 60 VEX FMA3 scalar and packed instructions
997518d2|2026-11-22|shader: lower the remaining no-carry 32-bit add and sub forms in pixel_aot (#582)
d1f74e56|2026-11-22|docs: record Intel Core Ultra 7 358H measurements for x87 compare, fcmov, iarith and fnstcw (#592)
5d36745b|2026-11-22|docs: record Intel Core Ultra 7 358H measurements for x87 fldcw, status and parity (#593)
3ffc193b|2026-11-22|cpu: terminate the exe_start fixtures after the body returns (#589)
abac7f28|2026-11-22|disasm_rdna2: decode 64-bit MUBUF and MTBUF buffer instructions (#579)
2e57d9f5|2026-11-22|shader: lower RDNA2 scalar shifted additions in pixel_aot (#587)
3f398b1f|2026-11-22|shader: lower gfx1013 v_madak_f32 and v_madmk_f32 in pixel_aot (#595)
3059fe8d|2026-11-22|Cover ten AVX integer translator semantics that no test observed (#603)
e3946dda|2026-11-22|disasm_rdna2: decode 64-bit FLAT, GLOBAL and SCRATCH memory instructions (#580)
75e4701a|2026-11-22|cpu: add SSE2 leaf AOT translator covering 23 forms
7cfe3b03|2026-11-23|shader: lower scalar multiply-high instructions in pixel_aot (#604)
7b55abcd|2026-11-22|shader: lower gfx1013 s_bfm_b32 in pixel_aot (#601)
6c905a22|2026-11-22|textures: decode bounded multi-lane BCPack rANS streams
1a9173a0|2026-11-22|ci: publish the latest Windows coverage chart (#610)
9b0df709|2026-11-22|cpu: lower repeated string copies in control-flow functions (#597)
a7418050|2026-11-22|disasm_rdna2: decode 64-bit DS local data share instructions
d4a31cff|2026-11-22|cpu: lower memory fences, PAUSE and cache hints
79c93c8d|2026-11-22|cpu: lower CMPXCHG8B and CMPXCHG16B without LOCK
c91340be|2026-11-22|cpu: lower FTST, FXAM, FICOM, FICOMP, FNOP and FNCLEX in the x87 profile
e7e9133f|2026-11-23|shader: lower scalar bit-count instructions in pixel_aot (#605)
8fedbcc1|2026-11-23|shader: lower scalar first-bit scans in pixel_aot
7ca812ba|2026-11-22|Check section raw data against the end of the file as Windows does (#143)
efecacfc|2026-11-23|shader: report xbox.* intrinsics as declarations, calls and metadata names (#608)
2f1f8322|2026-11-22|Lower 128-bit legacy MOVDQU and 14 SSSE3 integer vector instructions (#355)
3b8fb59b|2026-11-22|pe: reject TLS images with an unwritable AddressOfIndex (#543)
84c1506f|2026-11-22|disasm_rdna2: decode v_madmk_f32 and v_madak_f32 with their literal (#615)
1947e972|2026-11-22|cpu: lower AVX2 128-bit lane ops, dword blends, and cross-lane permutes (#344)
14daa49f|2026-11-22|cpu: lower baseline 128-bit SSE2 shifts, packs, unpacks, shuffles, and movemask (#371)
c9431ca3|2026-11-22|shader: lower gfx1013 v_mad_legacy_f32 in pixel_aot (#358)
b03f7545|2026-11-23|shader: lower scalar leading-bit scans in pixel_aot (#622)
40560a6e|2026-11-22|cpu: lower all 36 VEX-encoded packed FMA3 instructions (ps/pd) (#349)
a4ad8bb3|2026-11-22|docs: point the form_coverage example at the dev build's records (#636)
a97ba9f0|2026-11-22|shader: find VkRunner probe failures on stdout in the FP32 check (#634)
c3747274|2026-11-22|tooling: write dependency_paths JSON as ASCII (#635)
c0334140|2026-11-22|Reject eight compute-store programs the guards should refuse (#638)
076b1d05|2026-11-22|pe: record malformed intake JSON as an error report (#637)
840b2745|2026-11-23|shader: draw textures through translated indexed mesh UVs (#633)
bcdce962|2026-11-22|gpu: add DXR Tier 1.0 fixture coverage (#641)
2fa38a16|2026-11-22|Cover eight depth-mask lowerings that no draw observed (#630)
9d71655e|2026-11-22|shader: lower s_cselect_b64 in pixel_aot (#631)
5e279b15|2026-11-22|Cover four mesh-entry lowerings that no draw observed (#632)
824370fd|2026-11-23|shader: lower scalar add-with-carry and subtract-with-borrow in pixel_aot (#629)
dd31c484|2026-11-22|shader: accept scratch-base and binary16 rounding-mode s_setreg_imm32_b32 writes in pixel_aot (#640)
c0612cde|2026-11-22|Cover eight AVX move translator semantics that no test observed (#611)
8f573661|2026-11-23|shader: verify depth occlusion between translated mesh draws (#639)
2bb9ee0f|2026-11-23|cpu: preserve packed FMA sources until writeback (#649)
5fc014d6|2026-11-22|disasm_rdna2: decode 64-bit VOP3A and VOP3B instructions
fa21e29d|2026-11-22|shader: verify unsigned pixel comparisons on raw bit patterns (#616)
fddc4ff4|2026-11-22|docs: record hardware runs from #1 after #501
7b8b6c41|2026-11-22|shader: lower s_bfe_u64 and s_bfe_i64 in pixel_aot (#642)
7009a7e9|2026-11-22|pe: support width-checked HIGHLOW base relocations (#607)
03ece067|2026-11-22|shader: lower single forward exec-masked regions in pixel_aot
1a252c81|2026-11-22|shader: validate region pair elements and tighten masked controls
719552ef|2026-11-23|Lower named SPIR-V ballot extraction for the HLSL bridge (#656)
3be2e29e|2026-11-22|cpu: keep memory reads whose value is unused in the shared lowering
6b33fbf7|2026-11-22|shader: lower gfx1013 binary16 v_cmp and v_cmp_class_f16 in pixel_aot
32dd77be|2026-11-22|pixel_aot: decide binary16 comparison ordering on the encoding
c90d3f50|2026-11-22|shader: lower gfx1013 v_cvt_u32_f32_e64 and v_cvt_i32_f32_e64 in pixel_aot
080fd5ce|2026-11-22|tooling: reject deeply nested bencode in decode_torrent (#643)
7f7802a3|2026-11-22|tooling: validate section_probe --first and --last as section tags (#647)
680babed|2026-11-22|Cover four v_mad_legacy_f32 and literal multiply-add lowerings (#660)
4c3bbd09|2026-11-22|shader: reject a non-positive --limit in extract_shaders (#645)
bd5bef84|2026-11-22|Reject four import bindings the call translator should refuse (#655)
e7860e3c|2026-11-22|textures: report bcpack_probe packet errors without a traceback (#648)
9b2b05a2|2026-11-22|meta: link Development Q&A from the new-issue chooser (#659)
58dfa51d|2026-11-22|gpu: run BVH buffer operations at Shader Model 6.0 (#619)
b962800a|2026-11-22|cpu: lower repeated string fills in control-flow functions (#598)
f1f95cf6|2026-11-22|Verify translated compute texture through mesh and pixel stages (#658)
fa4a28c4|2026-11-22|textures: validate flushed rANS home states and exact decode bounds
d7c4db7a|2026-11-22|docs: fix the non-Windows Python check lists (#662)
742a2463|2026-11-22|tooling: report unreadable images in xmem_attribute_scan without a traceback (#663)
2d243b9a|2026-11-22|tooling: report damaged inputs in dxil_retarget without a traceback (#664)
4808cfd9|2026-11-22|tooling: report unreadable torrents in verify_build as usage errors (#646)
dc94dc83|2026-11-22|tooling: require exactly one layout file in resource_inventory (#644)
edc2e54e|2026-11-22|cpu: track string direction across branches and ABI boundaries (#599)
8606b038|2026-11-22|cpu: lower baseline 128-bit SSE3 and SSE4.1 floating-point instructions
324748ba|2026-11-23|docs: name the shader census tiers and the key behind each count (#625)
07d10b45|2026-11-22|shader: lower composite s_waitcnt forms in pixel_aot (#665)
6caa1ca7|2026-11-22|Cover eight AVX2 lane, blend and cross-lane permute lowerings (#668)
e2526e17|2026-11-22|cpu: accept vendor-defined NaN orderings in dpps/dppd check
09cea87a|2026-11-22|Cover seven SSE2 shift, pack, unpack, shuffle and movemask lowerings (#667)
a334126a|2026-11-22|tooling: report damaged inputs in the script state tools without a traceback (#672)
ff7f68a3|2026-11-22|shader: bound extracted shader file names in extract_shaders (#673)
899785ea|2026-11-22|cpu: render one EXE-entry raster draw across native, mapped and AOT runners (#560)
a4538d8b|2026-11-22|Cover seven sampled-coverage lowerings that no draw observed (#623)
e7e1ba73|2026-11-23|test: extend RDNA2 SOP2 decoding vectors (#518)
0a9cc046|2026-11-22|Cover eight SOPK, SOP1 and no-carry lowerings that no draw observed (#620)
b93399de|2026-11-22|shader: lower scalar quadmask and bit replication
24936e2d|2026-11-22|shader: lower scalar 64-bit bit-set instructions
de736171|2026-11-22|Cover four DPP lowering semantics that no draw observed
149fc21b|2026-11-22|Merge main and preserve integer export provenance
5748b02c|2026-11-23|Separate depth-reference sampler views in the HLSL bridge (#676)
55b06d90|2026-11-22|gpu: replace offscreen_check dispatch chain with capability table
09a6c49d|2026-11-22|cpu: lower single-element string transfers without REP (#600)
b5509fe8|2026-11-22|Cover seven jump-table dispatch lowerings (#675)
3b9bb1aa|2026-11-22|Cover six packed FMA3 lowerings (#677)
64c4329d|2026-11-22|Merge origin/main (b5509fe) into shader/dpp-quad-f16-ops
df675d76|2026-11-22|Merge origin/main (b5509fe) and add plain VOP2 e32 dot2c/dot4c forms
e6071736|2026-11-22|shader: lower flat_load_dword through TitleLookup in pixel_aot
1e30f793|2026-11-22|disasm_rdna2: consume the second dword of DPP, SDWA and literal-K forms
5a54c779|2026-11-22|shader: batch remaining pixel_aot draw cases per shader
24011857|2026-11-22|Merge remote-tracking branch 'upstream/main' into gpu/offscreen-capability-table
11987cd8|2026-11-22|pe: serve implicit TLS data templates through a carrier DLL
2b033596|2026-11-22|Cover six scalar carry, borrow and 64-bit select lowerings
8a891a17|2026-11-22|disasm_rdna2: decode s_waitcnt with the GFX10 counter fields
26464b53|2026-11-22|Merge remote-tracking branch 'upstream/main' into shader/batch-pixel-aot-draws
26b501dc|2026-11-22|shader: split shared comparison samplers in the SPIR-V bridge
a3cf6894|2026-11-22|Cover five 64-bit scalar bit-field extract lowerings (#691)
2d29904b|2026-11-23|tooling: list hardware reports the coverage table does not link
7ab772bb|2026-11-22|gpu: run the translated compute-mesh-pixel chain from the shim EXE entry (#682)
89b86ed7|2026-11-22|tooling: distinguish shader census populations (#617)
1dc97d90|2026-11-22|Cover three repeated string-copy lowerings that no test observed (#689)
b6c2fbcc|2026-11-22|cpu: keep memory reads whose value is unused in the chained-range translator (#674)
343a3a48|2026-11-23|docs: install python3-venv before the Linux quick start (#696)
47a840ab|2026-11-23|docs: record seven more hardware runs from #1
e20a8df1|2026-11-22|Cover six mesh-entry lowerings that no draw observed
a712fb6c|2026-11-22|Merge main and preserve scalar bit-set coverage documentation
b8150d0c|2026-11-22|shader: accept 2D/3D and wider thread groups in compute_store_aot
7d929158|2026-11-23|Merge pull request #702 from romanv9/tooling/hardware-report-helper
ac8e6160|2026-11-23|Merge pull request #699 from roansystemlabs/disasm/waitcnt-gfx10-vmcnt
4e956b7e|2026-11-22|Verify GPU resource reuse across EXE-driven draws
706100f9|2026-11-22|shader: run SPIR-V sampler views at Shader Model 6.0
279d30cb|2026-11-23|Merge pull request #693 from roansystemlabs/disasm/vop-extension-dwords
b49216fb|2026-11-22|cpu: cover AVX2 broadcast, variable-shift and alias controls
ba329ec2|2026-11-23|Merge pull request #657 from romanv9/docs/hardware-runs-after-501
bc0dff05|2026-11-22|Merge remote-tracking branch 'origin/main' into HEAD
b961a891|2026-11-22|Merge remote-tracking branch 'origin/main' into textures/bcpack-rans
33a24a03|2026-11-22|Merge remote-tracking branch 'origin/main' into shader/scalar-bitset64
ab2de473|2026-11-22|Cover five repeated string-fill lowerings that no test observed
d8684761|2026-11-23|shader: refuse unresolved xbox.* calls before a PSO submission
fa52b377|2026-11-23|Merge pull request #703 from Leyukaka/mesh2-controls
4c3b5731|2026-11-23|Merge pull request #697 from ammmae2312/shader/batch-pixel-aot-draws
5e56e6a1|2026-11-23|shader: gate every xbox.* call and refuse to submit without an inventory
efc986ba|2026-11-23|docs: separate the read-only inventory from the submission gate
e718b4ed|2026-11-22|Cover eight depth-mask lowerings that no draw observed
5fdbd444|2026-11-22|Cover four multiply-high and bit-count lowerings that no draw observed
cf51bdde|2026-11-22|Add a signed-only multiply-high row and a 63-bit count row
67918c4b|2026-11-23|Merge pull request #709 from Leyukaka/stos-controls
d9f1b747|2026-11-23|Merge pull request #706 from ammmae2312/cpu/avx2-shift-controls
33524e3a|2026-11-22|shader: pass through modules without a mixed standalone sampler
4b10089f|2026-11-22|test: add sampler descriptor controls to the comparison-sampler readback
6f620f73|2026-11-22|shader: count an undecorated descriptor set as set 0 in the sampler split
8571007b|2026-11-23|Merge pull request #681 from rwZephyr/shader/comparison-sampler-split
0cfdda1a|2026-11-23|Merge pull request #690 from MoonYueya/shader-xbox-intrinsic-gate
b42888dd|2026-11-23|Merge pull request #705 from RonLogic/test/guest-texture-reuse-upstream
cf8ab8b9|2026-11-23|Merge pull request #708 from roansystemlabs/compute/thread-group-dimensions
b6ec4ae0|2026-11-23|Merge main and preserve both sampler experiment results
e22289df|2026-11-22|cpu: lower 51 vector shuffle, unpacking, and move instructions
6f48648e|2026-11-23|Merge pull request #576 from Holastor/shader/vop3
3c209380|2026-11-22|Reject seven sampled-coverage programs the guards should refuse
4c3035f4|2026-11-23|tools/disasm_rdna2: format 64-bit SOP1 and SOP2 operands as register pairs
aa109532|2026-11-23|Merge pull request #707 from Jess-Doit/test/spirv-sampler-sm60
0cadad79|2026-11-23|Merge pull request #712 from Leyukaka/depth2-controls
38e0a43a|2026-11-23|docs: add Intel UHD Graphics (device 468b) to hardware coverage
5c74dabc|2026-11-23|shader: expand 64-bit scalar bitfield extraction in pixel_aot and scanner
64c1f556|2026-11-23|Preserve reciprocal estimates and legacy XMM lowering
94bbacd7|2026-11-23|docs: name the reporter on the Intel UHD Graphics row
d5757430|2026-11-23|Merge pull request #612 from Neoplayer/shader/v-cmp-f16
b4b82e0c|2026-11-23|Merge pull request #714 from Leyukaka/sc2-controls
f46dacd4|2026-11-23|Merge pull request #686 from MrSkelee/feature/cpu-100-percent-zen2-coverage
787ace7c|2026-11-23|Merge pull request #716 from lambirou/shader/gfx1013-quadmask-bitreplicate
d3bfa875|2026-11-22|Cover four scalar leading-bit scan lowerings
08f83072|2026-11-22|Fold the leading-bit SCC case into the main draw
5bf70ce3|2026-11-22|shader: specify raw MUBUF binding contract
dfc76233|2026-11-22|Cover seven string direction and single-transfer lowerings
9c05e76f|2026-11-23|Merge remote-tracking branch 'upstream/main' into gpu/offscreen-capability-table
1f222442|2026-11-22|shader: lower 64-bit scalar bitset forms
d11dd0fc|2026-11-23|Merge pull request #719 from romanv9/docs/intel-uhd-coverage-row
7c256c3e|2026-11-23|Merge pull request #683 from Leyukaka/sop2-controls
089afa5d|2026-11-23|Sync main and isolate optional fence fixture capabilities
642b1165|2026-11-23|Merge pull request #687 from Leyukaka/flbit-controls
cb382f81|2026-11-23|Merge pull request #688 from Holastor/shader/salu64-wave32
45559f9e|2026-11-23|Merge pull request #717 from lambirou/shader/gfx1013-raw-mubuf-contract
121fdc3a|2026-11-23|Merge pull request #722 from Leyukaka/strdir-controls
837fd524|2026-11-22|shader: bind further constant buffers by table register and offset in pixel_aot
a0726e66|2026-11-22|shader: lower v_xnor_b32 and draw the integer e64 forms in pixel_aot
ab7c5284|2026-11-23|cpu: lower 56 packed and scalar float/double instructions (SSE, SSE2, AVX)
d3e22396|2026-11-22|shader: draw v_exp_f32_e64, v_log_f32_e64 and v_mac_f32_e64 in pixel_aot
b0fa5451|2026-11-23|shader: lower three-axis compute stores with a folded plane offset
d9c34c82|2026-11-23|Merge pull request #626 from danielradosa/cpu/fences-cache-hints
bf31e19a|2026-11-23|Merge pull request #724 from Not-Saii/shader/compute-three-axis-store
a87adb67|2026-11-23|Merge pull request #700 from Holastor/shader/salu-bfe64
434fabed|2026-11-23|Merge pull request #685 from ammmae2312/gpu/offscreen-capability-table
c7070e9e|2026-11-23|cpu: validate operands and add signed NaN/zero controls for float arithmetic
bb6ad6fc|2026-11-23|shader: lower v_alignbit_b32 and v_alignbyte_b32 in pixel_aot
8bfb6563|2026-11-22|shader: verify observable bit and byte alignment results
c956ef7c|2026-11-23|shader: reject invalid e32 encoding on three-operand integer operations
4656a421|2026-11-23|cpu: complete NEG, ADD and INC/DEC flag conditions in switch lifter
a7868f17|2026-11-23|shader: isolate bit and byte alignment into dedicated fixture
0377e42b|2026-11-23|shader: extract two-exit compute programs with the pixel check from #477
87d3677d|2026-11-22|ci: run the SPIR-V bridge WARP tests on both Windows jobs
6b0e0b02|2026-11-23|Merge pull request #726 from rwZephyr/ci/spirv-bridge-windows
87b861df|2026-11-23|Merge pull request #727 from rasptum/shader/cb-bindings
d3b69b0f|2026-11-23|Merge pull request #728 from rasptum/shader/e64-integer-draws
5bbfe125|2026-11-23|Merge pull request #725 from MrSkelee/feature/cpu-float-packed
89e80971|2026-11-23|Merge pull request #729 from rasptum/shader/e64-approx-draws
fd2486f4|2026-11-23|Merge pull request #731 from ammmae2312/cpu/switch-arithmetic-flag-conditions
f007ac45|2026-11-23|Keep both alignment and integer e64 draw registrations
da81e511|2026-11-23|ci: wait for SPIR-V version probes to exit
417827b2|2026-11-22|shader: lower image_sample_c_lz to SampleCmpLevelZero in pixel_aot
3823ac72|2026-11-23|shader: bind TRUNC_COORD and the depth range in the comparison contract
617374a1|2026-11-23|shader: accept every constant buffer as a comparison sample operand
1ab37569|2026-11-23|Merge pull request #740 from R0Y00/codex/fix-spirv-cache-probe
2165bb88|2026-11-23|Sync the SPIR-V cache probe fix from main
8fdc3770|2026-11-23|cpu: lower 34 packed, scalar, and half-precision float conversion instructions (SSE, SSE2, AVX)
c9f96062|2026-11-23|gpu: avoid an indexed constant array in the texture-init shader
f5530a4a|2026-11-23|Sync the SPIR-V cache probe fix from main
28bbc421|2026-11-22|shader: add a compute fallback for indexed mesh draws
cfb26005|2026-11-23|Merge pull request #627 from danielradosa/cpu/x87-examine-compare
2f4d918b|2026-11-23|cpu: gate host AVX/F16C and limit vcvtps2ph to immediate 0
67edabed|2026-11-23|Merge pull request #695 from Leyukaka/carry-controls
f3901d6d|2026-11-23|Merge pull request #718 from roansystemlabs/shader/pixel-aot-sample-c
4d7eb03f|2026-11-23|Merge pull request #692 from Holastor/shader/valu-alignbit
356a7be2|2026-11-23|Merge pull request #738 from roansystemlabs/compute/two-exit-extraction
4b218255|2026-11-23|gpu: light a translated mesh draw with an interpolated normal and N.L
20f0c32f|2026-11-23|shader: accept the LLVM 15 group-base form of a compute byte offset
80113dfa|2026-11-22|tests: run the carry-chain constant-operand draw by its command
d766349a|2026-11-23|cpu: support NEG flags and SETcc/CMOVcc in cpu_sse_aot
632e60f9|2026-11-23|Merge pull request #752 from rasptum/tests/carry-chain-draw-name
aeb45daf|2026-11-23|Merge pull request #748 from lukaslw-lab/cpu/texture-init-win10-warp
95776941|2026-11-23|shader: lower RDNA2 s_absdiff_i32 in pixel_aot
401dea07|2026-11-23|pe: bound RVA reads and TLS slots by Misc.VirtualSize in intake_exe.py
25a949f9|2026-11-23|Sync the predicate-rounding test fix from main
d5f5d3d1|2026-11-23|Sync the predicate-rounding test fix from main
25d049e5|2026-11-23|Sync the predicate-rounding test fix from main
5f5077fe|2026-11-22|shader: lower SDWA forms of 32-bit integer operations in pixel_aot
407e3557|2026-11-22|shader: lower the SDWA forms of ten unary f32 operations in pixel_aot
2dc5e13e|2026-11-22|shader: accept the e64 spellings of v_interp_p1/p2_f32 and v_nop in pixel_aot
248ba247|2026-11-22|tooling: keep the entry register when resolving xmem attribute callers
afe23570|2026-11-23|Merge pull request #741 from MrSkelee/feature/cpu-float-conversions
ed5b7dcf|2026-11-22|pe: bound intake section spans by VirtualSize
4069725d|2026-11-23|Merge pull request #749 from roansystemlabs/gpu/mesh-lit-fixture
37019005|2026-11-23|Merge pull request #750 from roansystemlabs/compute/llvm-three-axis-store
37ed46d9|2026-11-23|Merge pull request #679 from Leyukaka/dpp-controls
2018706b|2026-11-23|Sync the predicate-rounding test fix from main
405e9920|2026-11-23|shader: lower v_mad_u32_u24 row-pitch compute stores
9e1032dc|2026-11-23|shader: reject row pitches whose group rows wrap the address
b1e36157|2026-11-23|gpu: gate the SPIR-V ballot readback on SM 6.6
532c9512|2026-11-23|shader: check row pitches with an LLVM 15 group-base column offset
69a9f8ba|2026-11-23|shader: lower s_brev_b32 in pixel_aot
a455ccf2|2026-11-23|shader: decode v_fma_mix* neg/abs modifiers in disasm_rdna2
d3649785|2026-11-23|shader: lower the plain bfrev, ffbh and ffbl forms in pixel_aot
55c10e01|2026-11-23|Merge pull request #753 from ammmae2312/cpu/sse-aot-neg-setcc-cmovcc
c51284d8|2026-11-23|Merge pull request #757 from rasptum/shader/e64-leftover-spellings
1538fe63|2026-11-23|Merge pull request #762 from ammmae2312/shader/disasm-fma-mix-modifiers
8d2e7f01|2026-11-23|Merge pull request #761 from FZNull/fix/xmem-attribute-entry-register
f77b8292|2026-11-23|Merge pull request #613 from Joacoorandom/textures/bcpack-rans
f714d2fc|2026-11-23|Merge pull request #763 from lukaslw-lab/gpu/spirv-ballot-sm66-gate
d9c1f23c|2026-11-23|pe: require file-backed bytes for RVA and TLS reads in intake_exe.py
5b7b7b5f|2026-11-23|pe: keep sections with no raw pointer zero-filled during TLS intake
18fa56f2|2026-11-23|Merge pull request #755 from JB2WIN/shader/plain-bitscan
fdd71584|2026-11-23|Merge pull request #754 from Not-Saii/shader/compute-mad-row-pitch
65d27a10|2026-11-23|cpu: update scalar SSE docs and 32-bit cmovcc zero-extension check
a4ade303|2026-11-23|Merge pull request #756 from rasptum/shader/sdwa-f32-unary
deb02d69|2026-11-23|shader: lower disassembled v_fma_mix* fixture and check LLVM parity
086ad3fa|2026-11-23|Merge main into the SDWA integer pixel fixture
761e3236|2026-11-23|Merge pull request #760 from FZNull/fix/intake-virtual-size
2192cbe6|2026-11-23|gpu: cast a shadow with a translated depth pass and SampleCmpLevelZero
99589581|2026-11-22|shader: compose ballot lowering with the SPIR-V address window
d8840b5f|2026-11-23|shader: lower a per-lane compute guard as control flow around its stores
44329f55|2026-11-23|shader: block every form that reaches an xbox.* builtin, not only a direct call
bd302096|2026-11-23|Merge pull request #758 from rasptum/shader/sdwa-int-alu
6eefa31d|2026-11-23|shader: separate debug-only uses from executable xbox.* references
16ee123e|2026-11-23|shader: accept the gfx10 SOPK waitcnt forms with null in pixel_aot
6f00b62c|2026-11-23|shader: lower non-packed 16-bit integer VOP3 ops as low-half-only values
3cd9df39|2026-11-23|Merge pull request #624 from stan495/shader/sop2-absdiff-i32
c47e1e58|2026-11-23|shader: lower v_swap_b32 in pixel_aot
03d4fbc5|2026-11-23|Merge pull request #773 from roansystemlabs/gpu/mesh-shadow-fixture
9bba2d23|2026-11-23|Merge pull request #774 from Jess-Doit/fix/spirv-window-ballot
2bf21616|2026-11-23|shader: decide debug-only from the callee, not from a line-wide match
39d989ce|2026-11-23|Merge upstream/main (#760) and preserve mapped-image TLS tests
4c8fd70e|2026-11-23|Merge main and preserve adjacent fixtures
fd34062c|2026-11-23|Merge pull request #770 from Ammmae0x/cpu/sse-aot-followup-controls
c426252b|2026-11-23|Merge pull request #772 from Ammmae0x/shader/disasm-fma-mix-e2e-controls
a02593a9|2026-11-23|shader: lower 8 ternary integer ALU instructions in pixel_aot (VOP3)
42c3d167|2026-11-23|Merge main and preserve adjacent documentation
9f4b6ff7|2026-11-23|Merge main and preserve adjacent documentation
5d3a859a|2026-11-22|shader: format the v_readfirstlane_b32 SGPR and reject the VOP3A word
c1965294|2026-11-23|Merge pull request #500 from agilesh08/feat/cpu-bmi2-followup
a4c6c06c|2026-11-23|Merge pull request #764 from JB2WIN/shader/s-brev-b32
dc4fc49e|2026-11-23|Merge pull request #583 from legacy-code-archaeologist/audio/awc-music-header
1ae97419|2026-11-23|shader: lower v_mac_legacy_f32 in pixel_aot
8fef4a7f|2026-11-23|docs: add bcpack_symbols_check and shader_census_check to non-Windows and agent check lists
efa95561|2026-11-23|Merge main and retain both pixel test modes
bb00cf13|2026-11-23|docs: record four GPU hardware runs from #1
7d827d7b|2026-11-23|Merge pull request #777 from JB2WIN/shader/sopk-waitcnt
e861e7ed|2026-11-23|Merge pull request #776 from roansystemlabs/compute/per-lane-guard
81cf1e38|2026-11-23|Merge pull request #734 from MoonYueya/shader-xbox-intrinsic-gate
522e0a4f|2026-11-23|cpu: bind two ordinary import slots in one jump-table translation
d5cc461b|2026-11-23|cpu: lift ROL and ROR in cpu_switch_aot.py
eac0d290|2026-11-23|Merge pull request #778 from FZNull/fix/readfirstlane-sgpr
8d21a20c|2026-11-23|Merge pull request #766 from MrSkelee/feature/shader-vop3-ternary-int
8ed77f6c|2026-11-23|Merge pull request #759 from Ammmae0x/pe/intake-virtual-size-rva
3c49d706|2026-11-23|Merge main and isolate the FREXP exponent draw
f083fdc9|2026-11-23|Merge remote-tracking branch 'upstream/main' into review
7dd70a2c|2026-11-22|tooling: read 0xFA sound format tags in AWC sound-effect banks
72c7ae59|2026-11-23|shader: copy compute buffer reads from a caller-bound source to the store
749131f3|2026-11-23|cpu: lower RDTSC and RDTSCP in the shared lowering
e724a323|2026-11-23|Merge main and retain the swap and ternary draw modes
003d85d3|2026-11-23|Merge pull request #474 from venetanji/shader/frexp-exp-f32
e690ab86|2026-11-23|Check EXE cleanup after compute adapter selection fails (#787)
0dc025c8|2026-11-22|pe: keep each TLS carrier in a private directory and dispose of it by handle
7fdd7ac9|2026-11-23|cpu: lower SSE4.2 packed string compare instructions
049d5878|2026-11-22|shader: add a source-built subgroup compaction bridge fixture
224d6fd5|2026-11-22|Add wave-based UAV atomic waterfall experiment
b9d0b258|2026-11-23|Merge pull request #606 from Jess-Doit/shader/mesh-compute-fallback
20561568|2026-11-23|Merge pull request #789 from legacy-code-archaeologist/cpu/lower-rdtsc
195cdebf|2026-11-23|Merge pull request #784 from Ammmae0x/cpu/switch-rotates
001caeb5|2026-11-23|Merge pull request #786 from legacy-code-archaeologist/audio/awc-sfx-fa-header
3cb23b9e|2026-11-23|Merge pull request #780 from stan495/docs/sync-python-checks
80142711|2026-11-23|Merge remote-tracking branch 'upstream/main' into review
03a4fc3a|2026-11-23|Merge remote-tracking branch 'upstream/main' into review
6282b896|2026-11-22|tooling: read big-endian TADA AWC banks
7db2a2bf|2026-11-23|Merge pull request #771 from JB2WIN/shader/v-swap
5f4513ac|2026-11-23|Reject a non-IAT target with multiple import bindings
f618e68b|2026-11-22|tests: refuse the carrier by integrity label instead of by signature
287c0810|2026-11-23|Merge pull request #782 from stan495/docs/record-four-gpu-reports
b809ba43|2026-11-23|tools: add batch DDS exporter for TTPF texture pools
027afbac|2026-11-23|Merge main and isolate the scalar first-bit CTest
20197512|2026-11-23|Document the stable disjoint source buffer binding
f53830ce|2026-11-23|Merge pull request #785 from lukaslw-lab/cpu/two-import-bindings
00860ec6|2026-11-23|Merge pull request #698 from rasptum/pe/tls-templates
b5e2c5c2|2026-11-23|Merge pull request #788 from roansystemlabs/compute/vector-reads
04da1f4f|2026-11-23|Merge pull request #792 from legacy-code-archaeologist/audio/awc-big-endian
aa34b183|2026-11-23|Merge pull request #790 from Ammmae0x/cpu/sse42-pcmpstr
7b554557|2026-11-23|Merge pull request #618 from descent81/shader/scalar-first-bit
333562f7|2026-11-23|Merge pull request #791 from FZNull/shader/spirv-compaction-fixture
8b8557cb|2026-11-23|Merge main and retain all pixel draw modes
436a7fb9|2026-11-23|Merge main and isolate the scalar bitset draws
d508188f|2026-11-23|shader: accept s_version as a no-op marker in pixel_aot
8b4574d3|2026-11-23|Merge pull request #781 from JB2WIN/shader/mac-legacy
d70002c1|2026-11-23|Merge pull request #715 from lambirou/shader/gfx1013-bitset-b64
37112f68|2026-11-23|Merge main and isolate the e64 conversion draws
e6f0d6d8|2026-11-23|Merge main and retain both DS and VOP3 decoding
7458f5e0|2026-11-23|Merge remote-tracking branch 'upstream/main' into review
07c2c951|2026-11-23|Retain the scalar bitset draw alongside the e64 conversion test
16b433c1|2026-11-23|Compare two translated shadow frames from the EXE entry (#797)
f13f026b|2026-11-23|shader: refuse DPP spellings of VOP3-only ALU ops in pixel_aot
8863f1f5|2026-11-23|Merge pull request #796 from JB2WIN/shader/s-version
c6be0e2a|2026-11-23|Merge remote-tracking branch 'upstream/main' into review
a0554e51|2026-11-23|Merge pull request #621 from Neoplayer/disasm/ds-instructions
0a9da40d|2026-11-22|shader: lower v_add_co_u32, v_sub_co_u32 and v_subrev_co_u32 in pixel_aot
bed88fb6|2026-11-22|shader: lower v_cvt_pk_u16_u32 and v_cvt_pk_i16_i32 in pixel_aot
732ee6ba|2026-11-22|shader: lower v_sad_hi_u8, v_sad_u32 and v_msad_u8 in pixel_aot
40ec7cdb|2026-11-23|Preserve wide CMPXCHG writes on compare mismatch
c555eeba|2026-11-23|tools: enforce strict-by-default and atomic export in ttpf_batch_export
65c6b007|2026-11-23|Merge pull request #540 from alexander-src/shader/cvt-f32-to-int-pixel-aot
cbf7fa04|2026-11-23|Integrate current pixel and coverage fixtures
7d463b35|2026-11-23|Create the EXE shadow device through the Xbox shim (#799)
a46bc2bc|2026-11-23|tooling: measure DXC compile time on a generated mixed-sampler shader
198aa492|2026-11-23|Add explicit RT0 masks to normalized compressed exports
38e7c8e3|2026-11-23|tests: add real-backend verification with synthetic TTPF pool and decoded pixels
2614f223|2026-11-23|Merge remote-tracking branch 'upstream/main' into review
fdd05cf8|2026-11-23|Keep the current conversion and low16 scanner checks
d9986658|2026-11-23|Merge pull request #628 from danielradosa/cpu/cmpxchg8b-16b
837a404c|2026-11-23|shader: lower the 64-bit integer compares in pixel_aot
d0445318|2026-11-22|shader: verify textured indexed mesh fallback
ac66ea1c|2026-11-23|tools: purge stale DDS on export failure and test reused directory
d8819fe4|2026-11-22|test: cover pixel constant buffers in the textured mesh fallback
1f83267b|2026-11-23|Merge pull request #398 from abilliontokens/shader/exp-enable-mask
899366b0|2026-11-23|shader: choose the suffix-less 64-bit compare encoding like LLVM 15
747b21a7|2026-11-23|shader: preserve SPIR-V minimum sample LOD in HLSL
186711d6|2026-11-23|Merge pull request #800 from JB2WIN/shader/dpp-vop3-only
f2dc003c|2026-11-23|Merge remote-tracking branch 'upstream/main' into review
a0f28fae|2026-11-23|cpu: lower RDRAND and RDSEED in an opt-in host-entropy profile
8d933646|2026-11-23|Preserve explicit title address and pixel binding contracts
1e268ab7|2026-11-23|shader: refuse e32 compares with an SGPR destination in pixel_aot
77bca2c1|2026-11-23|docs: the f16 comparisons are no longer rejected
90f615b4|2026-11-23|Keep compressed export and low16 draw modes together
d2f7e7ec|2026-11-23|Distinguish the DXC profile from separate pipeline measurements
eca72b31|2026-11-23|Merge remote-tracking branch 'upstream/main' into review
ab5d3a51|2026-11-23|Merge pull request #804 from R0Y00/codex/spirv-minlod
7ec6337b|2026-11-23|Keep title ranges and compressed export cases independent
56b428b9|2026-11-23|tests: split the compute store draw into two CTest entries
721dc1d9|2026-11-23|Preserve both DXC timing and minimum LOD experiment notes
20e4357e|2026-11-23|Merge pull request #802 from Jess-Doit/shader/mesh-textured-fallback
df6a9c0a|2026-11-23|Merge pull request #807 from legacy-code-archaeologist/cpu/lower-rdrand
ee861c35|2026-11-23|Integrate the complete raw compressed EXP mask check
fd0d8cc6|2026-11-23|Merge remote-tracking branch 'upstream/main' into review
7d3940bd|2026-11-23|Preserve prior DDS files when an export fails
d795395d|2026-11-23|Merge pull request #732 from JB2WIN/shader/int16-vop3
2959baa3|2026-11-23|tests: cover TTPF export failure cleanup
a81b55a5|2026-11-23|Merge pull request #808 from JB2WIN/tests/compute-store-split
28385648|2026-11-23|cpu: lower signed condition codes, Jcc, CMOVcc, SETcc, NEG and CWD
24bb4752|2026-11-23|Require explicit per-channel masks for partial uncompressed exports
143479fb|2026-11-23|Keep title address and low16 draw modes together
774e6df8|2026-11-23|cpu: share the shim draw device selection in a header
70d34e90|2026-11-23|cpu: sample CPU-uploaded RGBA8 and BC7 textures from the shim-startup EXE entry
869a8251|2026-11-23|Merge pull request #801 from FZNull/tooling/dxc-compile-time
26d2f411|2026-11-23|Merge pull request #434 from abilliontokens/shader/exp-en-decoding
66669bbf|2026-11-23|Merge pull request #798 from stan495/textures/ttpf-batch-export
b6390017|2026-11-23|Keep raw compressed mask checks and uncompressed draw controls
1901dd2c|2026-11-23|cpu: preserve SF across all signed flag producers
bc1fce63|2026-11-23|Keep low16 ALU and VOPC destination scanner rules
f68fdf2c|2026-11-23|Record two AMD GPU reports
d9a9f25b|2026-11-23|tooling: discover the local Leonida backend in TTPF tests
ea370edf|2026-11-23|Merge pull request #694 from Its3rr0rsWRLD/flat-load-dword
2e8eef82|2026-11-22|shader: draw a GPU-produced texture through the mesh fallback
4825fa32|2026-11-23|Merge pull request #806 from JB2WIN/shader/vopc-e32-dst
50db1301|2026-11-23|Keep title load and uncompressed export draw controls together
d2ac4155|2026-11-23|Merge pull request #812 from R0Y00/codex/gpu-coverage-nov23
40e0bec7|2026-11-23|Keep 64-bit compare tests and current VOPC rules together
af942385|2026-11-23|shader: measure MinLod mip selection and blending against an HLSL control
a5176cc1|2026-11-23|shader: skip the MinLod draws below SM 6.6
06c8bf8b|2026-11-23|Merge pull request #803 from JB2WIN/shader/cmp64
f39f85d0|2026-11-23|Merge pull request #813 from stan495/codex/ttpf-test-tools-path
adae6b7c|2026-11-23|Merge pull request #814 from yummancito/exp/shim-textured-rebase
5a8333d7|2026-11-23|Gate optional signed-condition native instructions on CPUID
463bf9f7|2026-11-23|Keep 64-bit comparisons and uncompressed export draw modes
129191fd|2026-11-23|Merge pull request #815 from Jess-Doit/shader/mesh-produced-fallback
23faba66|2026-11-23|docs: record Ryzen 7 7800X3D results for four x87 fixtures
6cd0f158|2026-11-23|Merge pull request #816 from lukaslw-lab/textures/minlod-readback
6de5738e|2026-11-23|Merge pull request #805 from agilesh08/feat/cpu-signed-conditions
e860c8c1|2026-11-23|shader: test depth occlusion through the SM 6.0 mesh fallback
c1a65897|2026-11-23|Merge pull request #429 from abilliontokens/shader/exp-enable-mask-uncompressed
579d13fc|2026-11-23|Merge pull request #817 from stan495/codex/x87-7800x3d-measurements
95c8172c|2026-11-23|Merge main and preserve independent pixel draw fixtures
f16ef398|2026-11-23|Keep partial uncompressed exports alongside integer draw fixtures
1e0cecca|2026-11-23|gpu: test acceleration structure updates on DXR Tier 1.0
4ace7968|2026-11-23|shader: run SPIR-V subgroup compaction at Shader Model 6.0
dda25ebe|2026-11-23|Merge pull request #819 from Jess-Doit/test/mesh-depth-pair-fallback
dc893933|2026-11-23|Merge pull request #779 from rasptum/shader/new-integer-forms
325fe1ee|2026-11-23|Merge pull request #820 from Jess-Doit/test/dxr-tier-1-0-update
68603ab1|2026-11-23|Integrate DPP binary16 draws and correct trigonometric endpoints
449217d5|2026-11-23|textures: add optional Rust BCPack support to the batch exporter
352077cb|2026-11-23|tooling: report unusable JUnit input in form_coverage instead of a traceback
bf29cae6|2026-11-23|shader: run the SPIR-V MinLod smoke test at Shader Model 6.0
16d23561|2026-11-23|Merge pull request #821 from Jess-Doit/test/spirv-compaction-sm60
63a46c56|2026-11-23|Reject JUnit durations that cannot delimit a CTest run
e2ade14a|2026-11-23|gpu: test triangle facing and culling on DXR Tier 1.0
10fa426e|2026-11-23|Pin the optional Rust decoder checkout in the reproduction
ddc841fc|2026-11-23|gpu: create the shadow device from the EXE import
79ea3e44|2026-11-23|Merge pull request #824 from Jess-Doit/test/spirv-minlod-sm60
3ddce573|2026-11-23|Merge pull request #823 from alvaroolt/tooling/form-coverage-junit-diagnostic
c81164d0|2026-11-23|shader: run the standalone SPIR-V sampler split at Shader Model 6.0
4b606cd7|2026-11-23|Integrate vector bit operations with the shared integer lowerer
ea966619|2026-11-23|gpu: submit and wait for each shadow frame from the EXE
f80b24b8|2026-11-23|Keep JUnit coverage diagnostics with the batched shader controls
6cd84a07|2026-11-23|Merge pull request #822 from wish131400/textures/optional-rust-bcpack
00eeacb5|2026-11-23|Merge pull request #825 from Jess-Doit/test/dxr-tier-1-0-facing
929e6598|2026-11-23|Merge pull request #826 from R0Y00/codex/exe-shadow-runtime-chain
7269fd3a|2026-11-23|tests: run the comparison draws as their own pixel_aot CTest entry
8bebb5dc|2026-11-23|shader: run the SPIR-V address window at Shader Model 6.0
e8fd5507|2026-11-23|shader: lower a small integer ALU subset between a compute read and its store
e3beac54|2026-11-23|Merge pull request #829 from R0Y00/codex/exe-shadow-frame-submissions
1c90a28e|2026-11-23|Merge pull request #827 from Jess-Doit/test/spirv-compare-sampler-sm60
efb834ed|2026-11-23|Merge pull request #436 from Joacoorandom/shader/vector-bit-operations
06ee1951|2026-11-23|Merge pull request #831 from roansystemlabs/tests/pixel-aot-comparisons-entry
523d9ac9|2026-11-23|Merge pull request #832 from Jess-Doit/test/spirv-address-window-sm60
87df1714|2026-11-23|gpu: test triangle opacity on DXR Tier 1.0
bb578b14|2026-11-23|shader: lower v_readfirstlane_b32 in pixel_aot
1956565d|2026-11-23|shader: support Wave32 exec_lo execution mask operations in pixel_aot
76a922d7|2026-11-23|shader: lower the 16-bit three-operand integer VOP3 ops in pixel_aot
0294d19a|2026-11-23|Merge partial sample masks with current descriptor and wait tracking
3b5bf265|2026-11-23|Merge pull request #833 from Jess-Doit/test/dxr-tier-1-0-opacity
0306f884|2026-11-23|Merge pull request #765 from Holastor/shader/valu-readfirstlane
8370c17e|2026-11-23|Merge pull request #506 from PrivacyOnion/shader/image-sample-dmask-offset
5e936856|2026-11-23|Merge pull request #835 from roansystemlabs/compute/alu-subset
f7d0e897|2026-11-23|shader: lower v_mul_lo_u32 in pixel_aot
141ccff3|2026-11-23|Merge pull request #834 from JB2WIN/shader/int16-vop3-ternary
bb1eb948|2026-11-23|Merge current texture sampling and preserve explicit LOD controls
efd54611|2026-11-23|Preserve texture bindings and verify the Wave32 mask encoding
3c0d6c2c|2026-11-23|shader: lower gfx1013 v_cvt_flr_i32_f32 and v_cvt_rpi_i32_f32 in pixel_aot
cea1d927|2026-11-23|Merge pull request #510 from PrivacyOnion/shader/image-sample-lod-bias
93c853fa|2026-11-23|Merge current main and correct the full-width multiply readback
e874fece|2026-11-23|shader: restrict flr/rpi f32-to-i32 lowering to proven finite in-range inputs
87dd0908|2026-11-23|Merge pull request #684 from Holastor/shader/wave32-exec-mask
ee98bc68|2026-11-23|Merge pull request #838 from alexander-src/shader/cvt-flr-rpi-pixel-aot
527a2557|2026-11-23|Merge pull request #678 from Holastor/shader/valu-lshl-add
fcabab64|2026-11-23|gpu: measure default and -Od mesh pipeline creation across adapters
0ba78a58|2026-11-23|docs: keep public project wording title-neutral
856061c5|2026-11-23|test(shader): extend int16 ternary matrix for all orderings, ties and med3 chain
c3db3352|2026-11-23|Merge pull request #841 from R0Y00/codex/title-neutral-public-wording
5264daab|2026-11-23|docs: record the RTX 3070 Ti BVH report
b358dc1d|2026-11-23|Merge pull request #840 from alexander-src/shader/min3-max3-med3-int16-pixel-aot
930905bc|2026-11-23|Keep compile failures visible in mesh capability checks
fa96c5cc|2026-11-23|Merge pull request #843 from R0Y00/codex/record-rtx-3070-ti
68d849a8|2026-11-23|shader: decode the RDNA 2 scanner's CLI output as UTF-8
275c3997|2026-11-23|Merge pull request #842 from alvaroolt/gpu/mesh-od-pipeline
e97090a9|2026-11-23|Merge pull request #845 from lukaslw-lab/shader/rdna2-scanner-utf8
4c780733|2026-11-22|runtime: report process GPU memory from XMemGetWorkingSetStatistics
a35f8f0b|2026-11-23|runtime: select the working-set test adapter as offscreen_check does
1ca4ad14|2026-11-23|runtime: read every node of an adapter for the working-set total
629eb613|2026-11-23|runtime: compare adapter keys ignoring case and check the PnP key length
fea0d47e|2026-11-23|shader: lower quad-local DS swizzle in pixel_aot
046a2bdb|2026-11-23|Merge main and preserve integer pixel constant bits
402e8efe|2026-11-23|docs: generalize the README affiliation disclaimer
a99e8dad|2026-11-23|shader: name special source operands in disasm_rdna2
c0f4d9ce|2026-11-23|shader: compile the log2 pixel fixtures with -denorm any
06ca5c54|2026-11-23|Merge pull request #847 from R0Y00/codex/837-neutral-readme-disclaimer
ca43ae24|2026-11-23|Merge pull request #846 from stan495/codex/gfx1030-ds-swizzle
f73206b5|2026-11-23|Merge pull request #522 from lukaslw-lab/runtime/xmem-working-set
c0b36756|2026-11-23|Merge pull request #680 from Joacoorandom/shader/scalar-bitset64
a67576dc|2026-11-23|docs: link the RTX 5060 Ti BVH 621.14 run
386e737a|2026-11-23|Merge pull request #848 from marquezinii/shader/disasm-special-scalar-operands
1dce507a|2026-11-23|Merge main into FMA3 leaf fixture and reuse the host capability probe
9b537599|2026-11-23|cpu: publish RGBA8 textures from completed archive records
85c504d8|2026-11-23|Merge main into SSE2 leaf fixture and enforce its aligned-buffer contract
c422cd7e|2026-11-23|Merge pull request #849 from R0Y00/codex/rtx5060ti-bvh-62114
851b68c5|2026-11-23|docs: record MinLod mip selection and blend-weight rounding per adapter
facfdcfb|2026-11-23|Merge pull request #503 from PrivacyOnion/cpu/lower-fma3-all
9fff1ee7|2026-11-23|shader: lower v_div_fmas_f32, reject unverified v_fma_f16 and v_div_scale_f32
6c3eb6c2|2026-11-23|Keep the SSE2 and FMA3 coverage checks after merging main
ed2476f7|2026-11-23|Merge main and keep masked EXEC lowering within proven operations
34d03c79|2026-11-23|Merge pull request #850 from descent81/cpu/archive-texture-publication
3d84719f|2026-11-23|shader: lower ds_nop in pixel_aot
bd863378|2026-11-23|Merge pull request #851 from alvaroolt/shader/log2-denorm-any
0dc4d79b|2026-11-23|Merge pull request #852 from lukaslw-lab/docs/minlod-warp-builds
5d80e549|2026-11-23|Merge pull request #651 from MarlomMedeiros/shader/pixel-aot-exec-regions
3007320d|2026-11-23|Merge main and preserve legacy SSE m128 alignment faults
766276f3|2026-11-23|Recognize midpoints throughout the binary32 subnormal range
097afafe|2026-11-23|Merge pull request #609 from MarlomMedeiros/cpu-sse2-28
6292c355|2026-11-23|Merge main and verify divide FMA in an isolated GPU check
7e0be98b|2026-11-23|shader: lower plain binary16 add, sub, subrev and mul in pixel_aot
b99e8b58|2026-11-23|Merge main and retain DS NOP binary coverage
293b157d|2026-11-23|Merge pull request #704 from Holastor/shader/valu-fma-div
978273ba|2026-11-23|Keep the waterfall diagnostic opt-in and preserve strict readback failures
d4620fd1|2026-11-23|Merge pull request #853 from stan495/codex/gfx1030-ds-nop
227dd719|2026-11-23|Merge main and retain both divide FMA and binary16 test entries
58408d11|2026-11-23|shader: lower bounded DS lane rotations in compute
c1394473|2026-11-23|shader: lower private LDS word accesses in compute
c608fc1a|2026-11-23|Merge current main after DS NOP and preserve binary16 coverage
bdc76df2|2026-11-23|shader: lower and verify RDNA2 binary32 floating-point vector comparisons
11a233b7|2026-11-23|test: bound zero-count RCX branch mutation scans
393b68d7|2026-11-23|Merge pull request #793 from potatoeking5150-commits/experiment/gpu-alias-atomic-waterfall
8307b9ac|2026-11-23|Merge pull request #854 from JB2WIN/shader/f16-vop2-arith
b5541874|2026-11-23|shader: lower v_add_co_ci_u32, v_sub_co_ci_u32 and v_subrev_co_ci_u32 in pixel_aot
58da18a0|2026-11-23|Merge pull request #856 from stan495/codex/gfx1030-ds-bpermute-b32
b7ec9c27|2026-11-23|shader: lower 8 scratch load and store instructions in pixel_aot (FLAT)
1c0fd914|2026-11-23|Merge current main and retain both compute DS fixtures
e114e893|2026-11-23|Respect raw FP immediates and NaN ordering in f32 predicates
223a9081|2026-11-23|Merge pull request #858 from R0Y00/codex/bound-rcx-branch-mutants
35372ec5|2026-11-23|Keep scratch load results pending through partial VM waits
ec7d5d96|2026-11-23|Merge current main with the bounded RCX branch fixture
7adf1f9e|2026-11-23|Merge current main and retain guest source reads and their fault controls
3fb24dce|2026-11-23|cpu: draw each completed archive record through D3D12
e7c9b9b6|2026-11-23|docs: link the development Discord
688aa85d|2026-11-23|shader: decide v_dot4_i32_i8 and v_dot8_i32_i4 saturation in 32 bits
b6415ae2|2026-11-23|Merge pull request #857 from stan495/codex/gfx1030-ds-read-b32
f682ee46|2026-11-23|Merge pull request #855 from agilesh08/feat/shader-vopc-f32-compares
ee5535a4|2026-11-23|test: distinguish both compute MAD24 source truncations
b4b2ade8|2026-11-23|shader: decode s_and_saveexec_b32, v_mac_f32, v_cvt_f32_ubyte and v_mad_f32 in disasm_rdna2
c7ecfe0b|2026-11-23|Merge pull request #859 from alvaroolt/shader/carry-in-add-sub
3ccc6f9f|2026-11-23|Merge pull request #783 from MrSkelee/feature/shader-flat-scratch
4f4d03d0|2026-11-23|Restrict archive GPU capability skips to the initial probe
708f4ab3|2026-11-23|Merge pull request #602 from MarlomMedeiros/cpu-sse34-fp
4ff09e3d|2026-11-23|Retain full-width guest reads alongside the SSE alignment checks
3cbbd84e|2026-11-23|Merge remote-tracking branch 'origin/main' into codex/666-refresh
0cb42fca|2026-11-23|Merge pull request #860 from R0Y00/codex/docs-discord-community
0c609b6a|2026-11-23|shader: lower private LDS subword accesses in compute
c639a015|2026-11-23|Merge pull request #861 from romanv9/shader/dot-bounded-saturation
f761de17|2026-11-23|Merge shared vector lowering and verify the packed integer memory form
ff2b40e9|2026-11-23|Merge pull request #864 from Holastor/shader/disasm-decode-alu
654395ea|2026-11-23|Merge pull request #862 from R0Y00/codex/check-compute-mad24-truncation
2dd0bf30|2026-11-23|Merge pull request #863 from lukaslw-lab/cpu/archive-texture-gpu
8adda2b6|2026-11-23|Merge pull request #666 from grahamster2/cpu/lower-keep-dead-reads
301bfae1|2026-11-23|shader: verify lit mesh fallback at Shader Model 6.0
12fdf14c|2026-11-23|shader: lower private LDS byte and halfword accesses in compute
2617bed2|2026-11-23|Keep both shared CPU fixtures and full-width operand reads
39010d49|2026-11-23|Merge remote-tracking branch 'origin/main' into maint-395
32e2fd00|2026-11-23|Keep the first AVX min/max slice and pin DAZ bit behavior
25a33745|2026-11-23|docs: put Discord and newcomer paths first
041ca688|2026-11-23|Merge pull request #865 from stan495/codex/gfx1030-ds-write-b8
e8f31c20|2026-11-23|Keep a private LDS regression for overlapping partial writes
3c34d0bf|2026-11-23|Retain guest source read checks with the first AVX min/max slice
bdbb5749|2026-11-23|cpu: convert a console EXE with one command or report the first refusal
eab58b52|2026-11-23|shader: run portable MinLod readbacks at Shader Model 6.0
fc89b9e2|2026-11-23|Merge pull request #395 from evilclanker9000/ag/cpu-lower-avx-packed-int
4a19cec2|2026-11-23|shader: avoid double greater-than in the v_div_fmas_f32 helper
e74f6bd7|2026-11-23|shader: lower private LDS word updates in compute
8872308d|2026-11-23|shader: lower the binary16 rounding ops in pixel_aot
713c3ba4|2026-11-23|runtime: export the default XMem allocation pair
2d3a69e2|2026-11-23|Merge pull request #866 from MrSkelee/feature/shader-ds-read-write
063ffb5e|2026-11-23|Integrate packed SIMD integer fixture registrations
cedf64fe|2026-11-23|Merge branch 'main' of https://github.com/R0Y00/Xbox2Windows into maint-387
27a728f0|2026-11-23|Merge pull request #867 from R0Y00/codex/docs-readme-start-here
027b65fb|2026-11-23|docs: record a second RTX 3050 Ti Laptop result
9cdc1e13|2026-11-23|Merge pull request #869 from Jess-Doit/test/mesh-lit-fallback
fd38ead0|2026-11-23|Keep XMem consumer imports by name with stable export ordinals
df8edcdc|2026-11-23|Merge pull request #873 from Jess-Doit/test/spirv-minlod-levels-sm60
a8fc57f0|2026-11-23|Merge pull request #877 from lukaslw-lab/shader/div-fmas-win10-warp
9fa011ca|2026-11-23|Protect the EXE converter input from output-path collisions
2aa81a26|2026-11-23|Merge pull request #878 from stan495/codex/gfx1030-ds-add-u32
5fb3161c|2026-11-23|Integrate packed integer min/max and multiply fixtures with current lowerer
02cd2ac8|2026-11-23|pe: check a page-spanning TLS template and two callbacks in table order
2a661ee3|2026-11-23|Keep both packed SIMD CPU fixtures registered
ff6eaf20|2026-11-23|Merge pull request #387 from evilclanker9000/ag/cpu-lower-avx-packed-fp
e2a1f8c9|2026-11-23|Merge branch 'main' of https://github.com/R0Y00/Xbox2Windows into maint-401
b8646f5b|2026-11-23|Merge pull request #881 from JB2WIN/shader/f16-round
daf87ee4|2026-11-23|shader: return old values from private LDS word updates
76b65b03|2026-11-23|shader: decode scalar ALU, lane ops, and integer multiply in disasm_rdna2
a2da0896|2026-11-23|Merge pull request #879 from R0Y00/codex/xmem-default-startup
1e8c6a64|2026-11-23|Merge pull request #882 from R0Y00/codex/record-rtx-3050-ti-9597
71ad6ef3|2026-11-23|textures: verify strict two-block BCPack decode through GPU readback
54b6781e|2026-11-23|Use the first TLS page boundary strictly inside the array
330d8b9b|2026-11-23|Merge pull request #872 from rm-NoobInCoding/cpu/exe-aot-driver
ff3d85c6|2026-11-23|Integrate AVX extension and horizontal fixtures with the shared lowerer
07a03e27|2026-11-23|Merge pull request #401 from evilclanker9000/ag/cpu-lower-avx-int-minmax-mul
5fbc4ab0|2026-11-23|Merge branch 'main' of https://github.com/R0Y00/Xbox2Windows into codex/maint-402
71987272|2026-11-23|Merge pull request #884 from Holastor/shader/disasm-decode-sop-vop
2629d412|2026-11-24|Merge pull request #886 from stan495/codex/bcpack-mode6-multiblock
169289b3|2026-11-24|cpu: prove fixed RBP frame anchors
406e14af|2026-11-24|Merge pull request #883 from rasptum/pe/tls-page-spanning-template
22ad3d8b|2026-11-24|Integrate AVX packed shift and lane fixtures with the shared lowerer
ddac8a73|2026-11-23|pe: inspect package import closure without a Windows loader
729bfa1d|2026-11-23|runtime: define the XMem physical-page allocation pair
bcc1068b|2026-11-23|cpu: lower FPREM and FPREM1 in the checked x87 entry
6eb3341c|2026-11-23|cpu: record the D3D12ConfigureVirtualMemoryX call shape under Windows and pe_map
70ccbfe0|2026-11-24|Merge pull request #885 from stan495/codex/gfx1030-ds-add-rtn-u32
ca0cc0ea|2026-11-24|Merge pull request #402 from evilclanker9000/ag/cpu-lower-avx-int-extend-hadd
afaa07e2|2026-11-24|Merge branch 'main' of https://github.com/R0Y00/Xbox2Windows into codex/maint-404
a1739cbf|2026-11-24|Carry frame anchors through the archive codec adapter
638e95ed|2026-11-23|tooling: add a deterministic CPU differential fuzz smoke
b40df0d2|2026-11-24|cpu: bind synchronous stack outputs through ordinary imports
8e85283b|2026-11-23|runtime: add a source-built tools-memory availability fixture
80c4d6e0|2026-11-24|shader: lower private LDS min, max and reverse subtract updates
21371181|2026-11-23|shader: lower s_load_dword and s_load_dwordx2 descriptor fetches in pixel_aot
320474df|2026-11-24|Keep preflight output separate from inputs and retain forwarding paths
d0bf29ad|2026-11-24|Document caller buffers and reuse of physical page numbers
f5c35c22|2026-11-24|docs: record three independent BVH retests
9adf97c8|2026-11-24|Use the shared native termination wrapper for VM recording fixtures
62c7536b|2026-11-24|pe: initialize the default x64 security cookie before startup
b3f00766|2026-11-23|tooling: reject empty CPU fuzz filters
279a7fb2|2026-11-24|Merge pull request #889 from R0Y00/codex/cpu-rbp-frame-anchor
369ad504|2026-11-24|Merge pull request #893 from denn0x/cpu/lower-x87-fprem
d3a0aae7|2026-11-23|runtime: define the physical-page mapping contracts for startup
1f6c38a5|2026-11-24|Merge pull request #404 from evilclanker9000/ag/cpu-lower-avx-int-shifts-pack
c8de0e42|2026-11-24|Merge branch 'main' into codex/cpu-stack-output-import
fab50380|2026-11-24|Merge pull request #890 from Joacoorandom/pe/offline-package-import-closure
1a5148ec|2026-11-24|Check the XMem tools-memory trace in both loaders
d03a4401|2026-11-24|Merge pull request #899 from R0Y00/codex/pe-load-security-cookie
7f74a1f5|2026-11-24|Merge pull request #897 from R0Y00/codex/record-three-bvh-retests
a8d0ecfc|2026-11-24|Merge pull request #891 from lukaslw-lab/gpu/configure-virtual-memory
3dfe3a9e|2026-11-24|Merge pull request #892 from rasptum/runtime/xmem-physical-pages
cc09358e|2026-11-23|shader: lower guarded compute v_mul_lo_u32
431db896|2026-11-24|Keep archive frame analysis on the proven control-flow edges
d172f0ac|2026-11-24|Merge pull request #898 from stan495/codex/gfx1030-ds-min-i32
02db28b9|2026-11-24|Merge pull request #894 from triggered02/tooling/cpu-differential-fuzz
16533766|2026-11-24|Merge main and keep both XMem fixture notes
24f42978|2026-11-23|tests: start the XMem default pair EXE natively and under pe_map
b68ef35f|2026-11-23|gpu: check the CPU/GPU range translation against a desktop D3D12 heap
24e58eb7|2026-11-23|runtime: add startup wrapper, live-worker control and independent controls to map-pages fixture
0fa712a6|2026-11-24|Merge pull request #904 from denn0x/cpu/lower-x87-fninit
a329153d|2026-11-24|shader: return old LDS words from min, max and reverse subtract
ff3c29fa|2026-11-23|cpu: reproduce load-config dispatch callback behavior
adb3101a|2026-11-24|Integrate AVX shuffle support with the current shared lifter
f19bcfc6|2026-11-24|Merge pull request #895 from R0Y00/codex/cpu-stack-output-import
86c5ae79|2026-11-22|cpu: lower FBSTP and FBLD in the x87 profile
4fbf91d1|2026-11-24|Merge pull request #909 from Jess-Doit/shader/compute-guarded-mul-lo
dc7118f9|2026-11-24|shader: lower private LDS reads into a VGPR half
da9bc9c0|2026-11-24|Merge pull request #896 from Holastor/xmem/tools-mem-available-fixture
40a19174|2026-11-23|shader: lower the 16-bit integer to binary16 conversions in pixel_aot
af0590ae|2026-11-23|Release D3D12 objects before freeing the wrapped range
03a101a9|2026-11-24|shader: lower private LDS counter updates
47be65f8|2026-11-23|tooling: isolate the DXC GVN cost to conditional UAV stores
d9320ba2|2026-11-24|Preserve the native Win64 ABI around the VZEROALL fixture
41b1de6c|2026-11-23|pe: notify a started EXE's TLS callbacks on thread attach and detach
20a9ac08|2026-11-24|Merge pull request #910 from rasptum/shader/smem-load-dword
379b0d62|2026-11-23|runtime: record trace, ordered addresses and changed-argument controls in map-pages fixture
67b21b4c|2026-11-24|shader: lower private LDS word replacements
80ca079e|2026-11-24|Merge pull request #913 from rasptum/tests/xmem-default-pair-native-mapped
f9ecbb5c|2026-11-24|Resolve physical-page fixture registration and documentation conflicts
cba69690|2026-11-24|runtime: report desktop tools-memory availability
20ff8eb8|2026-11-23|docs: add Fedora setup to the non-Windows host guide
2c38a115|2026-11-24|Merge pull request #915 from stan495/codex/gfx1030-ds-min-rtn-i32
ec5d0046|2026-11-24|Merge pull request #916 from denn0x/cpu/lower-x87-bcd
8f1786e2|2026-11-24|shader: lower wide private LDS transfers
2d6a1273|2026-11-24|Clarify Fedora LLVM 15 versioned aliases
a8a88e1f|2026-11-24|Merge pull request #917 from stan495/codex/gfx1030-ds-read-u8-d16
4673aeed|2026-11-23|cpu: lower AVX horizontal floating-point add and subtract instructions
c6319981|2026-11-23|pe: make EXE thread notifications eligible after process attach
c3d6ce1c|2026-11-24|Merge pull request #912 from lukaslw-lab/gpu/configure-vm-translation
1591a014|2026-11-24|Keep both pixel test modes when updating main
fba08922|2026-11-24|Merge remote-tracking branch 'origin/main' into codex/gfx1030-ds-read-b64
9a71aec6|2026-11-24|Merge remote-tracking branch 'origin/main' into codex/gfx1030-ds-cmpst-b32
d686d24f|2026-11-24|Merge remote-tracking branch 'origin/main' into codex/gfx1030-ds-read-b64
c9403fa3|2026-11-24|Merge remote-tracking branch 'origin/main' into codex/gfx1030-ds-cmpst-b32
94bc632f|2026-11-24|Merge pull request #390 from evilclanker9000/ag/cpu-lower-avx-shuffle-blend
ec5221dc|2026-11-24|Integrate LDS counters with current return and half-read fixtures
4b8f5d43|2026-11-23|cpu: add linker-produced CFG callback fixture
6e7ee7e2|2026-11-24|Merge pull request #902 from Holastor/xmem/map-physical-pages-fixture
f9ad5523|2026-11-23|tooling: add fixed-frame generators to the CPU fuzz test
bbdaa771|2026-11-24|Merge pull request #918 from rasptum/tooling/dxc-compile-blowup
9fea3a77|2026-11-24|Merge pull request #923 from R0Y00/codex/xmem-tools-memory-query
56ef0bfc|2026-11-24|Integrate AVX horizontal floating-point tests with current main
e1c2b527|2026-11-24|shader: lower remaining private LDS subword and thread-ID accesses
fdaecd6c|2026-11-24|Merge pull request #924 from MasterDomino/docs/fedora-host-setup
af8fcba9|2026-11-24|Merge pull request #920 from rasptum/pe/exe-tls-startup
a33eff82|2026-11-24|pe: start EXEs with bundled DLL dependencies
8e3d647b|2026-11-23|cpu: accept LEAVE on a proven RBP anchor in cpu_switch_aot
70286f55|2026-11-24|Merge pull request #919 from stan495/codex/gfx1030-ds-inc-u32
756bc6fb|2026-11-23|pixel_aot: accept partial uncompressed integer export masks
edb179ad|2026-11-24|Merge remote-tracking branch 'origin/main' into codex/gfx1030-ds-read-b64
dc491ddb|2026-11-23|cpu: allow stricter CFG handling in the load-config callback fixture
49400f2a|2026-11-24|Merge pull request #927 from denn0x/tooling/fuzz-cpu-frames
7716a09b|2026-11-24|runtime: share physical page backing across XMem views
be8624e7|2026-11-24|Merge pull request #922 from stan495/codex/gfx1030-ds-cmpst-b32
3c2018ba|2026-11-24|shader: lower private strided paired LDS accesses
1fada95f|2026-11-24|Merge pull request #926 from Ammmae0x/cpu/avx-horiz-fp
8ea6be10|2026-11-24|Preserve scalar AVX floating-point state in the shared lowerer
fca61339|2026-11-23|runtime: define the memory-transfer contract for startup
0d53c3d4|2026-11-24|Merge remote-tracking branch 'origin/main' into codex/gfx1030-ds-write-b8-d16-hi
60ebf5c7|2026-11-24|Merge remote-tracking branch 'origin/main' into codex/gfx1030-ds-read2st64-b32
25149031|2026-11-24|Merge pull request #921 from JB2WIN/shader/f16-cvt-int16
e8c5886d|2026-11-24|Keep binary16 DPP half-turn sine results stable across hosts
dba3dc62|2026-11-24|Merge remote-tracking branch 'origin/main' into codex/gfx1030-ds-read-b64
ece4bdd7|2026-11-24|Merge pull request #930 from R0Y00/codex/pe-package-startup
22084b39|2026-11-24|shader: lower private LDS quadword and paired transfers
1347311c|2026-11-24|shader: lower private qword arithmetic and bitwise LDS updates
a9720b95|2026-11-24|Retain binary16 conversion and DPP draw registrations
30483cfe|2026-11-24|shader: lower private qword LDS replacement operations
9d4aa5d8|2026-11-23|Merge remote-tracking branch 'origin/main' into shader/exp-mask-ordinary
53561dac|2026-11-24|Require an explicit unsupported CFG result in the fixture
5bc4456b|2026-11-24|shader: lower private qword comparisons and XOR
d2573563|2026-11-24|shader: return the old pair from private qword LDS updates
4ab09d3f|2026-11-24|shader: return the old qword from LDS comparisons and XOR
82a1eedb|2026-11-24|Reject non-qword LEAVE operands in the frame model
51ad45bb|2026-11-23|runtime: record ordered arguments and test changed first transfer request
1c13defd|2026-11-24|Merge pull request #934 from R0Y00/codex/xmem-shared-physical-pages
eb9910cb|2026-11-24|Keep the SSE negative control outside the tracked VEX register set
f14e3f6f|2026-11-23|runtime: inventory audio imports and add an ApuAlloc caller fixture
3fd852ae|2026-11-24|shader: lower private LDS floating-point updates in normal mode
2e236d9e|2026-11-24|runtime: reserve XMem address space as mappable placeholders
88fec3c8|2026-11-24|shader: lower private LDS qword counters and conditional exchange
d9926ac9|2026-11-24|Merge branch 'codex/gfx1030-ds-read-b64' into codex/gfx1030-ds-min-rtn-f32
c0287fb5|2026-11-24|Require a CLI opt-in for uncompressed RT0 write masks
30d9c988|2026-11-24|shader: lower private LDS floating-point compares with old values
af7df661|2026-11-24|Merge pull request #925 from stan495/codex/gfx1030-ds-read-b64
26f9be7b|2026-11-24|Keep ADDTID accesses separate from wide LDS layouts
d7c4d6e7|2026-11-24|Merge pull request #935 from Holastor/xmem/transfer-memory-contract-fixture
e943cca2|2026-11-24|Merge pull request #914 from triggered02/cpu/load-config-callback
e3272939|2026-11-24|Integrate AVX conversions and comparisons with current main
71273b44|2026-11-24|Merge pull request #933 from legacy-code-archaeologist/cpu/switch-leave
810acb05|2026-11-24|Merge pull request #407 from evilclanker9000/ag/cpu-lower-avx-scalar-fp
8265d3ff|2026-11-24|Merge remote-tracking branch 'origin/main' into codex/gfx1030-ds-write-b8-d16-hi
182b4df3|2026-11-24|Merge remote-tracking branch 'origin/main' into codex/gfx1030-ds-read2st64-b32
b84b096b|2026-11-24|Merge remote-tracking branch 'origin/main' into codex/gfx1030-ds-add-f32
49581310|2026-11-24|Merge remote-tracking branch 'origin/main' into codex/gfx1030-ds-write-b8-d16-hi
9ba58388|2026-11-24|shader: lower private LDS binary64 selection and paired exchange
6063ede8|2026-11-24|Merge remote-tracking branch 'origin/main' into codex/gfx1030-ds-read2st64-b32
1c1ef3d7|2026-11-24|Merge remote-tracking branch 'origin/main' into codex/gfx1030-ds-add-f32
fdaba2fb|2026-11-24|Preserve scalar AVX support alongside conversions and comparisons
56cad548|2026-11-24|Merge branch 'codex/addtid-maintainer' into codex/gfx1030-ds-write-b8-d16-hi
d6f428bd|2026-11-24|Merge branch 'codex/gfx1030-ds-add-f32' into codex/gfx1030-ds-min-rtn-f32
87252c3e|2026-11-24|Merge branch 'codex/gfx1030-ds-min-rtn-f32' into codex/gfx1030-ds-min-f64
78858c91|2026-11-24|Allow package child to pin external providers until exit
7586d327|2026-11-23|tooling: print hwreg operands in disasm_rdna2 as LLVM 15 does
b3de0f4a|2026-11-24|Merge pull request #945 from R0Y00/codex/xmem-placeholder-reservations
0dbf5829|2026-11-24|shader: lower paired LDS exchanges and full-wave counters
0f0c3eb1|2026-11-24|Merge pull request #931 from triggered02/shader/exp-mask-ordinary
39643ec9|2026-11-24|Merge remote-tracking branch 'origin/main' into codex/pe-pin-external-providers
84934a05|2026-11-24|Keep both source-built startup provider fixtures registered
5ccf7128|2026-11-23|cpu: lower AVX VEX.128 packed string compare instructions
06831afc|2026-11-23|runtime: test memory transfer prerequisite and failure chain
c48b644b|2026-11-23|Pin the ballot lowering's operand, matcher and prefix checks
7d70539a|2026-11-23|Pin the sampler view split's gather, multi-comparison and rejection paths
cc926a61|2026-11-24|Keep AVX regression controls on the supported boundary
e69db648|2026-11-24|Run private LDS cases in bounded test groups
a8901c04|2026-11-23|shader: open pixel_aot masked regions with s_mov_b64 exec save and VOPC e32 v_cmpx
2137d997|2026-11-23|Check that the renamed ballot variant is lowered
e5b396d0|2026-11-23|Check the bridge pointer and fix the gather comment
48bb0d05|2026-11-24|Verify transfer recorder failure and missing-import traces
cbe4fcb7|2026-11-23|pe: resolve bundled ordinal-only imports during EXE startup
9b2fe942|2026-11-24|Merge pull request #950 from R0Y00/codex/pe-pin-external-providers
750cd33b|2026-11-24|Merge pull request #951 from MasterDomino/tooling/disasm-hwreg
c8b88076|2026-11-24|Merge pull request #947 from rasptum/runtime/audio-import-inventory
45593dee|2026-11-24|Preserve discarded packed string reads and require a defined return
e1b0930f|2026-11-24|Keep transfer chain and APU fixtures after merging main
b8a70524|2026-11-24|Combine strided LDS cases with the bounded private suites
cf3c75fe|2026-11-23|tooling: compare raw UAV output of condstore_256 with and without GVN
134b5be8|2026-11-23|tests: disable MSVC runtime checks in CRT-free ordinal fixtures
185ffbef|2026-11-24|Merge pull request #929 from stan495/codex/gfx1030-ds-write-b8-d16-hi
8e8bf201|2026-11-23|tests: compare XG-style swizzle selection with AMD addrlib
306c02d3|2026-11-23|dstorage: test console factory and queue adapter with delayed submit
76c828d0|2026-11-23|shader: lower the 64-bit VOP3 shifts in pixel_aot
27220796|2026-11-23|Pin compute_store's PSV, group, literal, offset, clause and wait bounds
672ac62f|2026-11-23|Name the expected diagnostics in the compute_store bound checks
788fc9eb|2026-11-23|tooling: recognize SOPK hwreg opcodes in scanner and add boundary tests
a52f8528|2026-11-24|Merge pull request #393 from evilclanker9000/ag/cpu-lower-avx-cvt-cmp-test
2138e2d6|2026-11-24|Merge pull request #936 from stan495/codex/gfx1030-ds-read2st64-b32
02ebdbea|2026-11-23|runtime: test aligned committed CPU memory on Windows
8a2f0f16|2026-11-23|gpu: export D3D12ConfigureVirtualMemoryX and check the startup sequence
a48c2b99|2026-11-24|Merge pull request #953 from Ammmae0x/cpu/avx-pcmpstr
d10c6fe9|2026-11-23|cpu: lower CPUID and XGETBV in an opt-in host-CPU profile
6f111ed8|2026-11-24|Merge pull request #955 from Holastor/xmem/transfer-prerequisite-chain
f65d5b79|2026-11-24|dstorage: add PC IID refusal, eager-provider control, and IID provenance labeling
1c9667eb|2026-11-24|Keep name-only package startup compatible with older mappers
00cc3c36|2026-11-23|tooling: decode the VOP3 binary16 interpolation forms in disasm_rdna2
a5325f52|2026-11-24|Merge pull request #957 from MasterDomino/shader/cmpx-region
ac1ac85b|2026-11-24|Merge pull request #959 from Leyukaka/ballot-controls
48782111|2026-11-24|Merge pull request #960 from Leyukaka/sampler-controls
67d54aa6|2026-11-24|Keep wide and strided LDS forms in the bounded private suites
d121c1de|2026-11-23|Pin the SPIR-V address window's end bound, direct casts and profile rejections
3a795d08|2026-11-23|Validate the direct-cast variant and tighten the comments
cfd5403c|2026-11-24|Merge pull request #956 from Batushn/disasm/hwreg-symbolic-spelling
2028e603|2026-11-24|Bound both generated dword shift counts explicitly
d5381340|2026-11-24|Merge pull request #962 from rasptum/tooling/dxc-gvn-output-parity
b3af6288|2026-11-24|Merge pull request #965 from Leyukaka/cs3-controls
1a2577f1|2026-11-23|shader: admit per-lane binary32 VALU forms in pixel_aot masked regions
4b30f253|2026-11-23|tooling: decode VOP2 carry ops and GFX10 MIMG image_atomic_* in disasm_rdna2
e715c932|2026-11-24|Merge pull request #961 from Joacoorandom/pe/package-ordinal-startup
f9a4c090|2026-11-23|runtime: test scoped guest content-root mapping
ea9f60b9|2026-11-24|dstorage: align fixture with public sources and add exact readback
0bcda168|2026-11-24|Merge pull request #970 from MasterDomino/tooling/disasm-interp-f16
5750ee72|2026-11-23|Pin the SPIR-V sampler split's profile, interface, set and ID checks
453a251a|2026-11-23|Group the sampler split cases with their neighbours
f560749a|2026-11-24|Merge main while retaining shift64 and CMPX fixture coverage
95b161df|2026-11-24|Merge pull request #939 from stan495/codex/gfx1030-ds-write-b128
72b296f5|2026-11-24|dstorage: adopt fresh fixture-owned IDs and recorder convention terminology
51e7e618|2026-11-23|gpu: reserve the configured GPU range at device creation
4f65184b|2026-11-23|tooling: decode GFX10 VOPC/VOPCX SDWA instructions in disasm_rdna2
631a1298|2026-11-24|Merge pull request #971 from Leyukaka/spw-controls
9d4bc953|2026-11-24|Integrate qword replacement with current LDS fixtures
23d2e93b|2026-11-24|Merge pull request #974 from Jess-Doit/test/xmem-committed-memory
778101b6|2026-11-23|tests: relocate VOP2 carry and MIMG atomic tests to resolve #970 merge conflict
b22457d2|2026-11-23|cpu: separate the supported XGETBV contract from what the passthrough executes
a6f7fdca|2026-11-23|xmem: check protection and shared-view behavior for the second commit case
e909bc7d|2026-11-23|tooling: reject LLVM-refused reserved and mismatched VOPC SDWA modifier bits
e41a09ac|2026-11-23|Merge remote-tracking branch 'upstream/main' into gpu/configure-device-sequence
8668d742|2026-11-23|Pin pcmpstr length saturation, flags, singleton ranges and indexed memory
d62e61f2|2026-11-23|Update the pcmpstr case and control counts
73fd5871|2026-11-23|tests: avoid LLVM 15 printSDWASel llvm_unreachable on sel==7 in round-trip check
e2f39444|2026-11-23|tooling: match LLVM 15 don't-care bits Inst{54}/Inst{62} in VOPC SDWA
a9980a5d|2026-11-24|Merge pull request #964 from MasterDomino/shader/shift64
783bb9b8|2026-11-24|Document enabling AddrLib after an initial configure
e307b664|2026-11-24|Preserve all startup fixtures in DirectStorage test integration
597f945f|2026-11-24|Merge pull request #978 from Leyukaka/sps-controls
6d5072a5|2026-11-24|Keep masked ALU and shift64 test registrations together
b1b779ae|2026-11-24|Merge pull request #940 from stan495/codex/gfx1030-ds-cmpst-b64
5223a82c|2026-11-23|cpu: lower legacy SSE FP compare and partial/non-temporal move instructions
f8e80e56|2026-11-24|Merge pull request #977 from Batushn/disasm/vop2-carry-and-mimg-atomics
cd9ff65b|2026-11-24|gpu: verify tile remaps across copy and direct queues
e58a231a|2026-11-24|Merge pull request #969 from legacy-code-archaeologist/cpu/lower-cpuid
79e25a9f|2026-11-24|Merge pull request #981 from rasptum/antigravity/xmem-second-commit-protection
3647cce7|2026-11-23|gpu: close the configured range before the device's final release
85302634|2026-11-24|Integrate qword arithmetic without replacing existing LDS fixtures
a472aba2|2026-11-23|pe: install a main image's TLS block in slot 0 of every thread
8b0213f3|2026-11-24|Merge pull request #982 from Leyukaka/pcmp-controls
b92dc801|2026-11-24|Merge pull request #980 from Batushn/disasm/vopc-sdwa
88e454d4|2026-11-24|runtime: export XMemTransferMemory with narrow desktop refusal and accounting verification
b3311bf3|2026-11-24|Merge pull request #979 from rasptum/gpu/xg-swizzle-selection
09658fe1|2026-11-23|Pin AVX integer palignr overflow, pmaddwd overflow, wide-source extension, vzeroupper and blend-mask checks
f5179c9f|2026-11-23|Update the AVX integer export and case counts
9ecdc945|2026-11-23|Model the three new AVX integer exports and update the rejection counts
64beb550|2026-11-23|Merge main, tighten DPP guards, and add DPP_CVT semantic probes
d7c52d23|2026-11-24|Merge pull request #983 from Jess-Doit/test/guest-content-root
9a353a74|2026-11-24|Merge pull request #963 from Holastor/dstorage/factory-queue-adapter
ace6a3c2|2026-11-24|Merge pull request #976 from MasterDomino/shader/region-body
2f2893f0|2026-11-24|Merge pull request #986 from R0Y00/codex/gpu-copy-queue-tile-order
d80138ed|2026-11-23|pe: reject mixed x86 machine bindings in package preflight
664e411a|2026-11-24|Preserve executed SSE guest reads after optimization
aaf8ab08|2026-11-24|Merge pull request #941 from stan495/codex/gfx1030-ds-add-u64
0ac58f64|2026-11-23|Pin x87 status-word AX tracking across joins, EAX reads and direct calls
a925424d|2026-11-23|d3d12_xs: export standard IID and WKPDID data symbols
c4a42d26|2026-11-23|gpu: own the configured GPU range through an explicit handle
dae1ca25|2026-11-24|Merge pull request #984 from Holastor/xmem/transfer-runtime-export
b6d4151a|2026-11-23|shader: lower v_pack_b32_f16 in pixel_aot
a399fe3c|2026-11-23|Merge main, route sin/cos/mac through sin32/cos32/mad32, and add semantic probes
fc26b58f|2026-11-24|Integrate private LDS f32 without replacing existing fixtures
8d6b5c44|2026-11-23|Pin AVX move broadcast width, 16-byte alignment and REP-prefix rejection
cda177ca|2026-11-23|Pin vperm2f128 zeroing priority and vmovhlps upper-lane clearing
23cd3221|2026-11-23|Check the new AVX move cases against the reference model and keep the prefix control in its group
2a8fade1|2026-11-23|Pin chained test width, 32-bit register writes and callback binding rejections
3118ca58|2026-11-23|Merge origin/main (dae1ca2) into shader/dpp-quad-extra-f32-co
bca9bbd4|2026-11-23|Pin the callback rejection addresses and update the chained counts and docs
907058ee|2026-11-23|tests: route the XG linear-fallback control through addrlib
12574172|2026-11-23|Update the x87 status rejection count and list
ffbbf526|2026-11-24|runtime: verify native Windows interior placeholder splits
25e9451c|2026-11-23|Merge origin/main (dae1ca2) into shader/dpp-quad-cvt-ops
2134f9db|2026-11-23|pe: install main-image slot 0 only on participating threads
aabbedd8|2026-11-24|Merge pull request #992 from Leyukaka/avxi-controls
08fe5f45|2026-11-23|cpu: lower MMX/SSE2 double-to-packed-int and quadword bridge ops
d9f7d29f|2026-11-24|Reject invalid GPU range sizes before reserving windows
54b0a666|2026-11-24|Check the exact AddrLib mutation result and preserve local edits
46b3137d|2026-11-23|shader: lower v_pk_fmac_f16 in pixel_aot
91312663|2026-11-24|Merge pull request #994 from Joacoorandom/pe/preflight-machine-compatibility
5f0aa847|2026-11-24|runtime: test ACP address and memory mapping sequence
986e7f6e|2026-11-24|Merge pull request #985 from Ammmae0x/cpu/sse-fp
77008e5a|2026-11-23|shader: lower private LDS qword min, max and xor updates
0715b380|2026-11-24|Use DPP trigonometry inputs that detect omitted range reduction
da1c4473|2026-11-24|Merge pull request #946 from stan495/codex/gfx1030-ds-add-f32
1d7954ff|2026-11-24|gpu: verify cross-page copy provenance for mapped sections
1f9c6af0|2026-11-24|Integrate legacy packed comparisons and half moves with the shared lowerer
5816bf67|2026-11-23|Merge remote-tracking branch 'upstream/main' into cpu/mmx-sse2
3174eb3a|2026-11-24|Merge pull request #997 from R0Y00/codex/runtime-placeholder-interior
eddd3e17|2026-11-23|shader: compute s_add_i32 overflow SCC from the result sign
ac0acd1a|2026-11-24|Merge pull request #968 from lukaslw-lab/gpu/configure-device-sequence
e9439d98|2026-11-24|Merge pull request #1000 from Holastor/audio/acp-address-memory
40ea2434|2026-11-24|Exercise rounding and DAZ in the shared SSE2 differential
d5ec5d84|2026-11-24|Document the floating-point state required by the MMX leaf
1ccc3260|2026-11-24|Merge pull request #1001 from rasptum/opencode/shader-pack-b32-f16
9d28f086|2026-11-23|cpu: lower legacy SSE MMX-domain conversion, shuffle, and non-temporal store ops
8084aeea|2026-11-24|Merge pull request #1003 from R0Y00/codex/gpu-page-source-provenance
419e859f|2026-11-23|pe: refuse a main-image release while a participant may run its code
2fca753b|2026-11-23|cpu: lower LDMXCSR and STMXCSR in an opt-in host-MXCSR profile
7c04aa0a|2026-11-24|Merge pull request #998 from Leyukaka/avxm-controls
d7ad9a25|2026-11-24|Link the RX 9070 XT source-built wave64 report
d763b204|2026-11-24|Document SSE/MMX leaf state and MOVNTQ output scope
d2938082|2026-11-23|shader: alternate qword min/max fixture controls
8868f7c1|2026-11-24|Keep the verified DPP and integer dot forms
2067f54a|2026-11-24|Merge pull request #1005 from mumba17/shader/add-i32-overflow-form
5246da9b|2026-11-24|cpu: lower VEX vaddsubps and vaddsubpd
e06e8c1d|2026-11-24|Merge pull request #999 from Ammmae0x/cpu/mmx-sse2
1f34ba14|2026-11-24|Merge pull request #996 from rasptum/tests/xg-swizzle-review-fixes
bda89d7b|2026-11-24|Pin aligned SSE faults in the existing shuffle regression
ca52bbc1|2026-11-24|Merge pull request #1006 from Leyukaka/chain-controls
2892db6d|2026-11-23|pixel_aot: use binary16 encodings for inline float constants in v_pack_b32_f16
87c69f68|2026-11-23|pixel_aot: refuse other decimal floats on v_pack_b32_f16 and hedge the encoding note
422cd8be|2026-11-23|docs: record RX 9060 XT MinLod point selection and blend rounding
bd39381a|2026-11-23|Pin signed imm8 arithmetic, call-clobbered R8, import name case and link-name checks
a8d5f062|2026-11-23|Describe the new call-pair and ordinal-import controls
fd805a22|2026-11-24|Merge main and keep both D3D12 configuration and SDK GUID exports
6832a9dd|2026-11-24|Clear MXCSR refusal status when a later guest invocation faults
20b51132|2026-11-24|Merge pull request #1002 from lambirou/shader/lds-qword-minmax-xor
e2fed935|2026-11-24|Merge pull request #1008 from R0Y00/codex/rx9070-31044-hardware-report
b4211ed5|2026-11-24|Merge pull request #1010 from Leyukaka/x87c-controls
b7b63548|2026-11-23|cpu: lower AVX alternating add/sub, quadword move, and byte-masked store
abdb8fb5|2026-11-24|Merge pull request #1007 from Ammmae0x/cpu/sse-mmx
dd8735bd|2026-11-24|Merge pull request #382 from evilclanker9000/ag/cpu-lower-sse2-packed-cvt
2d8a0cc9|2026-11-24|Merge pull request #423 from KomaGuy/shader/dpp-quad-extra-f32-co
685dce48|2026-11-23|pe: keep pe_map_release 0/1 and notify the main image only on participants
2b977cd0|2026-11-24|gpu: test Xbox frame events and pipeline token wait semantics
ea66b8cd|2026-11-24|docs: record the GTX 1660 SUPER and in-box WARP dxr_opacity TMin cross-check
be947e1d|2026-11-24|docs: record that the in-box WARP cannot reach the wave lane-block misroute
a6728e06|2026-11-24|Merge main and adopt shared low-16-only contract for DPP_F16
ea777873|2026-11-24|Synchronize frame-event fixture ticks with pending waits
965c3eb1|2026-11-24|Tie the DXR cross-check to its reported adapter coverage
44c7a7b4|2026-11-24|Merge pull request #1009 from legacy-code-archaeologist/cpu/lower-mxcsr
c89aba51|2026-11-24|Keep the WARP misroute observation scoped to the tested setup
21b88684|2026-11-24|Merge pull request #1015 from zohaiblazuli/cpu/vex-addsub
23ab53d2|2026-11-24|Exercise inline half constants in the binary and pixel readbacks
62206622|2026-11-24|Merge pull request #1012 from lantiumBot/docs/minlod-rx9060xt
f1c515df|2026-11-24|Merge origin/main into shader/dpp-quad-f16-ops
cd879076|2026-11-24|Merge pull request #1016 from Leyukaka/calla-controls
5b2b07b8|2026-11-23|shader: round v_pk_fmac_f16 once as a fused binary16 result
26d19479|2026-11-24|Merge origin/main into shader/dpp-quad-cvt-ops
4648502f|2026-11-23|Merge remote-tracking branch 'origin/main' into antigravity/shader-vop2-pk-fmac-f16
79c0be85|2026-11-23|shader: admit s_nop and satisfied waits in pixel_aot masked regions
d185cab5|2026-11-24|Limit DPP conversion support to verified 32-bit operations
ff4b71b9|2026-11-23|cpu: split PR #1018 around vmovq and vmaskmovdqu, merge upstream/main, fix RDI save definedness and page-edge semantics
6a09e878|2026-11-24|Merge pull request #1014 from Holastor/gpu/frame-events
ee730d22|2026-11-24|gpu: validate a raw SRV address against its buffer
d9c3609f|2026-11-24|gpu: store through two raw UAV address windows
ea243ffd|2026-11-24|Merge pull request #1013 from Leyukaka/pack-inline-f16
fcb34088|2026-11-24|Refuse an explicit SGPR export wait inside masked regions
44b525f8|2026-11-24|Merge pull request #1019 from triggered02/docs/dxr-opacity-tmin-crosscheck
ee04692c|2026-11-24|Merge pull request #1020 from triggered02/docs/warp-wave-lane-inbox
957f9923|2026-11-23|tests: make the xmem second-commit refusals reach the path they claim
9e227b40|2026-11-23|Pin x87 integer store and integer-arithmetic slot selection at depth two
d3bd50c6|2026-11-23|Update the x87 integer and integer-arithmetic counts
0c6083ac|2026-11-23|Update the x87 integer-arithmetic table row
64ecc873|2026-11-24|Merge pull request #1004 from rasptum/antigravity/shader-vop2-pk-fmac-f16
a6399891|2026-11-24|Validate raw UAV window offsets and make readback expectations independent
b2fa7973|2026-11-23|tooling: refuse VOPC SDWA scalar codes 209-234 in disasm_rdna2
c4b5bef0|2026-11-23|tooling: print neg on a VOPC SDWA inline constant as neg(c)
838baf78|2026-11-23|cpu: lower INT3 and UD2 to the host exceptions they raise
364eb996|2026-11-24|Use integer masks after quad reads for general DPP permutations
ee36f45c|2026-11-24|cpu: lower Integer flag-stack and LAHF instructions
c262d28f|2026-11-23|textures: test the swizzle layout handoff with aliased output
5e3ae8f4|2026-11-24|cpu: restrict vmaskmovdqu lifted helper to selected bytes
d8829992|2026-11-24|Merge main and retain both pixel controls
b6a59ad1|2026-11-23|Update the x87 summary total
750457f4|2026-11-24|Preserve decimal SDWA comparisons with explicit source negation
9db7a26b|2026-11-24|Separate native masked-store page observations from the lifted contract
f6636f1c|2026-11-23|gpu: report the shader-visible view heap limit
87229b1a|2026-11-24|shader: narrow PR #419 to v_cvt_f16_f32_dpp under explicit mode and low-16 contract
9a1c1e9a|2026-11-24|gpu: check signed-zero float min and max through title addresses
4bee1430|2026-11-24|Merge pull request #1023 from MasterDomino/shader/region-nop-waits
addd9537|2026-11-24|Merge pull request #1028 from MasterDomino/tooling/disasm-sdwa-scalar-subset
94cb2a33|2026-11-24|Merge pull request #1032 from Jess-Doit/test/xg-swizzle-factory
d0d6f4d3|2026-11-24|cpu: test XG swizzle mode selection and texture computer layout
4bc40d05|2026-11-24|Merge pull request #1030 from R0Y00/codex/quad-dpp-bit-mask
6e155ee3|2026-11-24|Reserve a fresh preferred base for each relocation-free fixture
2e761c0e|2026-11-24|cpu: narrow integer-flags PR to same-block ADD/SUB/CMP -> LAHF slice
42452cd4|2026-11-24|Merge pull request #1033 from legacy-code-archaeologist/cpu/lower-trap
d1ea7d50|2026-11-24|Merge verified quad routing fix into the half conversion candidate
4e7336f3|2026-11-24|shader: export signed compute DXIL by input digest
996881ab|2026-11-24|Merge pull request #1029 from MasterDomino/tooling/disasm-sdwa-neg-inline
c6c23378|2026-11-23|docs: add pinned XCL reference and evidence map
c766c32b|2026-11-24|Merge pull request #1035 from Leyukaka/x87i-controls
56206562|2026-11-24|Merge pull request #1038 from zohaiblazuli/gpu/alias-signed-zero
61e5a099|2026-11-24|Merge pull request #1037 from FIocker/gpu/report-view-heap-limit
49f6a252|2026-11-24|docs: record Intel first full-query diagnostic
139892a9|2026-11-23|docs: record the RDNA 4 guarded-compare denormal mode switch
e2276063|2026-11-24|cpu: lower 64-bit explicit-length packed string compare instructions
d1673dd2|2026-11-24|Keep XCL licensing note scoped to repository policy
e569b52e|2026-11-24|Link the Intel diagnostic instrumentation and run log
7e8c1a1d|2026-11-24|docs: record divergent array-atomic results (Refs #369)
68ec1db6|2026-11-23|tooling: add deterministic PE compatibility report
a982c112|2026-11-23|tooling: clarify import route candidates
ef2189a2|2026-11-24|Merge pull request #1034 from Holastor/xg/suggest-swizzle-mode
4291c73a|2026-11-24|Merge pull request #1040 from R0Y00/codex/reloc-free-preferred-base
ee2d05eb|2026-11-24|Merge the verified PE fixture fix before rerunning xmem CI
62b55f96|2026-11-24|Merge pull request #419 from KomaGuy/shader/dpp-quad-f16-ops
0bf13820|2026-11-24|Merge origin/main into shader/dpp-quad-cvt-ops
8e9393a5|2026-11-23|runtime: test ACP HAL create, shape-context and heap sequence
440de93c|2026-11-23|pe: enroll main-image guest threads explicitly instead of ending the process
f3f83000|2026-11-24|Attribute array-atomic counts to their measured drivers
031e1a2f|2026-11-24|Merge pull request #1042 from R0Y00/codex/signed-dxil-catalogue
7baa8543|2026-11-24|Merge pull request #1018 from Ammmae0x/cpu/avx-fp-move
5969579f|2026-11-24|docs: state the RX 9070 XT provenance and separate ISA from draws
ef2f6035|2026-11-24|Merge pull request #1045 from Ammmae0x/cpu/pcmpstr64
e91f60b3|2026-11-24|Merge pull request #1036 from lambirou/tools/compat-report
4af2ab09|2026-11-24|Reclaim terminated suspended TLS enrollments
43cbd9ec|2026-11-24|Merge pull request #1043 from romn4rch/romn4rch-docs-737-first-full-query
a4e76515|2026-11-24|Merge pull request #1026 from rasptum/tests/xmem-second-commit-review-fixes
0e326266|2026-11-23|shader: add deterministic corpus report
8285b10d|2026-11-23|shader: fail on native extraction errors
0254713b|2026-11-24|Merge pull request #1027 from lambirou/docs/xcl-reference
b1d041cf|2026-11-24|Merge pull request #1046 from romn4rch/romn4rch-docs/369-array-atomics-experiment
df4cb1be|2026-11-24|tests: split the pixel_aot draw-core CTest
45ad903c|2026-11-24|Merge pull request #421 from KomaGuy/shader/dpp-quad-cvt-ops
9926b9f6|2026-11-24|Merge current main and resolve LDS translator and fixture conflicts
96b9d9da|2026-11-24|Merge current main and resolve LDS translator and fixture conflicts
93dca075|2026-11-24|Merge current main and resolve LDS translator and fixture conflicts
096c61ed|2026-11-24|Merge remote-tracking branch 'upstream/main' into cpu/integer-flags
f02da958|2026-11-24|Integrate updated LDS prerequisite and preserve existing coverage
6bcd2d01|2026-11-24|Integrate updated LDS prerequisite and preserve existing coverage
8b477a4c|2026-11-24|Integrate updated LDS prerequisite and preserve existing coverage
03029a27|2026-11-24|Merge pull request #1047 from rasptum/acphal/create-shape-heap-fixture
1fa981e2|2026-11-23|pe: route imports through explicit candidate policy
3dc27c33|2026-11-24|Merge main and keep both pixel CTest registrations
d82d5b6c|2026-11-24|docs: attribute the four-channel probe observation and confirm the WARP build
1bcfc42e|2026-11-23|pe: keep xgameruntime stub reason accurate
d0bede8c|2026-11-24|Merge origin/main into shader/dpp-quad-movrel-ops and split DPP movrel CTest
dc7bd046|2026-11-23|pe: refuse a main-image thread whose ResumeThread fails
969b7f72|2026-11-24|Keep ownership of the failed worker handle while waiting
d4cbfcd4|2026-11-24|Merge pull request #1039 from lambirou/shader/corpus-report
2cbd121f|2026-11-24|cpu: lower FXSAVE and FXRSTOR state save and restore instructions
7239f860|2026-11-23|gpu: test descriptor windows across two submissions
2a7f1b77|2026-11-24|cpu: accept FTZ and DAZ in the host-MXCSR profile's denormals opt-in
c74d708c|2026-11-24|Merge pull request #942 from stan495/codex/gfx1030-ds-add-rtn-u64
bb3d4d9f|2026-11-24|tooling: scan records after s_endpgm
239d3633|2026-11-23|runtime: add XGameRuntime thunk skeleton
af712152|2026-11-23|runtime: isolate xgameruntime fixture shim
4ad7b0a2|2026-11-24|Merge pull request #1049 from romn4rch/romn4rch-draw-core-split
d295488e|2026-11-24|Register the generated XGameRuntime fixture import library
248a32b6|2026-11-24|Retain both qword return and binary64 compare LDS cases
21b5a63d|2026-11-24|Merge pull request #1044 from lantiumBot/docs/rdna4-guarded-compare-denorm
e012a3ac|2026-11-24|Merge pull request #1050 from lambirou/pe/import-routing
884e5df5|2026-11-24|Merge remote-tracking branch 'upstream/main' into cpu/integer-flags
9a839a47|2026-11-24|Merge pull request #428 from KomaGuy/shader/dpp-quad-movrel-ops
f551f97c|2026-11-24|cpu: lower LAHF after a register ADD, SUB or CMP
b0fd0513|2026-11-24|Merge pull request #988 from rasptum/pe/main-image-tls-slot0
a8c82b4e|2026-11-24|cpu: route integer_flags LAHF fixture through ffx_version_query
edb4eb20|2026-11-24|tooling: fix SOPP marker and zero word decoding
083c95cf|2026-11-23|pe: cover delay-load and ordinal imports
1a6425e4|2026-11-24|Merge pull request #1054 from Jess-Doit/test/descriptor-window-submissions
24cadd43|2026-11-23|gpu: test CPU-copied descriptor records across submissions
c36b663f|2026-11-24|Report FTZ independently of the host DAZ capability
fad4eb05|2026-11-24|shader: add exact MinLod readback fixture
bfe43124|2026-11-24|Refuse LAHF flags from an overridden instruction
44ba0c3a|2026-11-24|Merge pull request #949 from stan495/codex/gfx1030-ds-min-rtn-f32
9c3ab31e|2026-11-24|Merge pull request #1051 from lambirou/runtime/xgameruntime-thunks
de19af87|2026-11-24|docs: record RDNA subnormal compare mismatch
cefbc19b|2026-11-24|shader: admit modeled scalar ALU in masked regions
97074f32|2026-11-24|Keep the paired exchange and existing LDS readback cases
ec0e7d63|2026-11-24|cpu: lower XSAVE and XRSTOR state save and restore instructions
8b97622c|2026-11-24|d3d12x: add device creation slots
96727873|2026-11-24|Merge pull request #1055 from romn4rch/romn4rch-rdna2-disasm-scan-938
53dd0402|2026-11-24|pe: route winhttp, mfplat, mfreadwrite and gameinput as Windows
8531dd26|2026-11-24|Initialize descriptor probe output from COMMON
3955ae4c|2026-11-24|Merge pull request #1057 from legacy-code-archaeologist/cpu/mxcsr-denormals
d33845b9|2026-11-24|Merge pull request #1058 from lambirou/pe/delay-load-ordinals
a8910975|2026-11-24|runtime: add XUser and XGameSave contracts
389fe728|2026-11-24|Decode six gfx1013 VOP1 and VOP2 opcodes
7ee4880f|2026-11-24|Merge pull request #1059 from romn4rch/romn4rch-minlod-readback-809
d73ab13a|2026-11-23|Add XApu command-queue caller and recording provider fixture
658e6eb4|2026-11-23|Record refused-output sentinels in the XApu queue provider
02cd56c6|2026-11-24|Keep the comparison note scoped to the original observation
995f4e17|2026-11-24|Merge pull request #1031 from Ammmae0x/cpu/integer-flags
d68a310d|2026-11-24|docs: link the live startup status and GPU tasks
8e45488a|2026-11-24|Integrate masked scalar lowering with current main
232103a0|2026-11-23|gpu: test descriptor paging across two logical heaps
b3357355|2026-11-24|tooling: decode the remaining GFX1013 MIMG sampling, gather, packed-load and BVH ops
110e1860|2026-11-24|Merge pull request #952 from stan495/codex/gfx1030-ds-append
6187b066|2026-11-24|pe: add a --trap-missing diagnostic mode to pe_run_package
c28350e0|2026-11-24|cpu: enforce FXSAVE entry-state contract and probe host FIP/FDP preservation
f3aebeb4|2026-11-24|runtime: test the GameInput ordinal 1 caller with a recording provider
040d4963|2026-11-24|Expect unsupported device IID in shadow rejection trace
580adbcc|2026-11-24|tooling: print a packed MIMG address longer than eight registers as a 16-register tuple
6ef8952a|2026-11-24|shader: lower finite in-range binary16 to 16-bit integer conversions in pixel_aot
e89e6fc4|2026-11-24|tooling: decode the 13 missing GFX10 SOP2 carry, inverted-logic and pack opcodes
e7a90c15|2026-11-24|shader: split shared comparison samplers on Texture2DArray images
2c49c0c5|2026-11-24|shader: lower scalar and vector 32-bit ALU ops in the mesh-entry translator
0938f6f0|2026-11-24|cpu: enforce XSAVE component entry-state contract and probe host FOP/FIP/FDP preservation
b75ff646|2026-11-24|texture: add gfx10 tiling round-trip
9bf0d06e|2026-11-23|rdna2: decode past s_endpgm with after_endpgm field
e0c577e7|2026-11-23|tests: fix unused scan_source in endpgm boundary test
73abcf11|2026-11-24|Mark records after the first endpgm
5c7e4dd6|2026-11-24|shader: lower u32 add, max, or and xor in the compute store profile
f01259c9|2026-11-24|Merge pull request #1060 from Jess-Doit/test/descriptor-record-copy
3cbccd38|2026-11-24|tests: include FOP in host FXSAVE/FXRSTOR error-pointer capability probe
4d605f08|2026-11-24|Merge pull request #1067 from mumba17/pe/routing-desktop-system-dlls
b51372f1|2026-11-24|Transition the paging output buffer from COMMON
d246daf4|2026-11-24|Merge remote-tracking branch 'origin/main' into HEAD
9fadea43|2026-11-24|tooling: compare packed MIMG and BVH addresses of 9 to 15 registers as LLVM 15 spells them
c859144a|2026-11-24|pe: start the pe_run_package entry as the main image
bdace855|2026-11-24|Merge pull request #1063 from romn4rch/romn4rch-rdna-compare-denorm-958
1f33420b|2026-11-24|Merge remote-tracking branch 'upstream/main' into cpu/sse-fxsave
3694ef01|2026-11-24|Merge pull request #1070 from tedin7/x2w-disasm-gfx1013-vop12
e5bfa0c9|2026-11-24|Merge pull request #1071 from rasptum/audio/xapu-queue-fixture
96e03500|2026-11-24|tooling: print BVH tuples as LLVM 15 does, give gathers four texels, decode non-sequential BVH addresses
60467c27|2026-11-24|Merge remote-tracking branch 'upstream/main' into cpu/xsave-state
e6361ca2|2026-11-24|Merge upstream/main into runtime/gameinput-caller-fixture
71719df0|2026-11-24|test: keep the VOP2 compute-store fixture out of the runtime case list
80d11300|2026-11-24|pe: trap only listed callable imports and terminate if the report fails
f7c9f579|2026-11-24|Merge pull request #1064 from romn4rch/romn4rch-pixel-aot-scalar-masking
d162e9d7|2026-11-24|test: make the mesh-entry unsigned min/max fixture detect swapped results
7182b787|2026-11-24|runtime: fix XGameSave data lifetimes
c738c7e7|2026-11-24|cpu: lower MASKMOVQ and MASKMOVDQU instructions
ed983c1c|2026-11-24|Merge pull request #1072 from R0Y00/codex/main-startup-progress-entry
0cfb44ce|2026-11-24|Preserve explicit CreateThread providers for package entry
d92d01d3|2026-11-24|gpu: lower primitive topology 0x11 (rectangle list) to triangle list (Refs #870)
e7ab28b4|2026-11-24|test: keep binary16 to integer draw expectations out of signaling NaN patterns
0b45e7f2|2026-11-24|Merge upstream main into qword LDS return forms
a4e2e47c|2026-11-24|Merge pull request #1066 from lambirou/d3d12x/device-slots
24d5a0c6|2026-11-24|docs: record the Zig cache workaround for the two archive fixtures
c85d40e3|2026-11-24|tooling: scanner counts the plain binary16 e32 forms pixel_aot lowers to the low half
6dad87b0|2026-11-24|shader: lower DWORD atomics through the SPIR-V address window
4f32ab2a|2026-11-24|Merge upstream main into binary64 LDS selection
45d21ef7|2026-11-24|Merge pull request #1079 from tedin7/x2w-spirv-sampler-split-texture2d-array
1fbba342|2026-11-24|Merge upstream main into qword LDS counters
0afcdfa1|2026-11-24|Merge pull request #1082 from lambirou/texture/gfx10-tiling
5a5edf2c|2026-11-24|Record Arc 140T and RTX 5080 Laptop coverage
23fbc5b0|2026-11-24|shader: lower scalar add, sub, and, or, xor and shift on known words in the compute store profile
16df191a|2026-11-24|docs: escape the pipe in the scalar OR table row
f8df2770|2026-11-24|pso: add static sampler root signature
94044c6a|2026-11-24|Decode 29 gfx1013 SOPK and SOPP opcodes
b19516a5|2026-11-24|Draw the actual rectangle-list expansion
cbf3c8bb|2026-11-24|Describe the verified rectangle corner order
31f9ebb0|2026-11-24|Merge pull request #1078 from tedin7/x2w-disasm-sop2-ops
72270be3|2026-11-24|Merge pull request #1062 from KelboLive/hardware/rtx4070tisuper
4c3c11b1|2026-11-24|test: add a dedicated chained-arithmetic readback for the VOP2 compute-store fixture
bd86292b|2026-11-24|docs: record the RTX 3070 inline dxr_opacity TMin scan (#1011)
a92a5831|2026-11-24|merge: update with main
ee082f52|2026-11-24|Size the native XSAVE test buffers from CPUID
4921f154|2026-11-23|gpu: keep the alias registry test independent of bottom-up ASLR
8f49205e|2026-11-23|tooling: accept named SM 6.5 skips in the GPU check smoke test
20fbd9bb|2026-11-23|docs: note the Windows path-length limit for the first build
b9beb159|2026-11-24|Merge pull request #1074 from Jess-Doit/test/descriptor-heap-paging
7ccdb7a4|2026-11-24|Test graphics pipeline stream draws through a root SRV
1ecbfbbb|2026-11-24|test: add a dedicated scalar-chain readback with an independent reference
d9d453e3|2026-11-24|docs: document the SOPK and SOPP decoding and make the scanner recognize it
2c04bf85|2026-11-24|shader: decode the remaining GFX10 SMEM operations in disasm_rdna2
c526be17|2026-11-24|Merge pull request #1086 from Ammmae0x/cpu/sse-maskmov
ded2ce7f|2026-11-24|Merge pull request #1076 from lukaslw-lab/runtime/gameinput-caller-fixture
aa88e37b|2026-11-24|Probe native FXSAVE selector retention independently
ea0e838e|2026-11-24|Merge pull request #1073 from tedin7/x2w-disasm-gfx1013-mimg
1f567822|2026-11-24|shader: split shared samplers on 1D and cube images
065d7662|2026-11-24|tooling: assess mesh lowering in PSO readiness
b89f9ec6|2026-11-24|tooling: decode 18 gfx1013 binary16 VOP1 and VOP2 opcodes
fba92c8f|2026-11-24|shader: keep invalid addresses from issuing a window atomic
d15536fd|2026-11-23|gpu: test both D3D12 device struct-return argument orders
f38773dc|2026-11-23|shader: read the split Texture2DArray comparison sampler back on the GPU
c70ee1cc|2026-11-23|cpu: load the app-local WARP in the mapped shim host
4480a13e|2026-11-24|texture: add BC and volume readback
78a6b31e|2026-11-24|Merge pull request #1069 from lambirou/runtime/xuser-xgamesave
423c2e0a|2026-11-24|test: verify SPIR-V address-window atomic readback
9f23be2b|2026-11-24|Refresh CPU-backed shader inputs after GPU readers retire
be033d97|2026-11-24|Merge pull request #1087 from tedin7/x2w-scan-f16-low16-sync
7a73f8c7|2026-11-24|Merge pull request #1091 from lambirou/pso/root-signatures
c77687bc|2026-11-24|Merge pull request #1085 from mumba17/pe/run-package-main-image
9149048b|2026-11-23|gpu: add wave64 emulation through group-shared memory
48de0f32|2026-11-24|Merge pull request #1075 from mumba17/pe/trap-missing
01c992ae|2026-11-24|fix: contain unexpected mesh lowering errors in PSO readiness
7c0910ab|2026-11-24|d3d12x: add descriptor heap fixture
b8869f5d|2026-11-24|fix: reject the nonexistent VOP3 form of the packed binary16 fused multiply-accumulate
305de800|2026-11-24|fix: keep named 64-bit timer destinations in SMEM decode and document the coverage
da6e42fe|2026-11-24|Merge pull request #1080 from tedin7/x2w-mesh-entry-scalar-vector-u32
46a7298c|2026-11-24|Merge pull request #1089 from R0Y00/codex/arc-140t-rtx-5080-laptop-gpu-reports
2db96ba5|2026-11-24|Merge pull request #1068 from Holastor/gpu/rectlist-topology
92c4ddee|2026-11-24|tooling: decode the five SOP1 relative scalar moves
58da41c3|2026-11-24|Merge pull request #944 from stan495/codex/gfx1030-ds-min-rtn-i64
9866ee6a|2026-11-24|shader: lower binary32 add, sub, subrev and mul encodings in the mesh-entry translator
4a2740d7|2026-11-24|shader: lower vector shifts and subtracts in the compute store profile
d8880573|2026-11-24|Merge main and keep the scalar decoder evidence
c3a14857|2026-11-24|tests: place CRT-free EXE data in the static TLS template
2e7e9306|2026-11-23|test(python-linux): run gpu_report_format_check.py
32f63f9e|2026-11-24|Merge pull request #1100 from roansystemlabs/docs-1011-nvidia-inline
b9e0110a|2026-11-24|runtime: add PLS and XPackage contracts
00d71ef5|2026-11-24|docs: note the carrier temp directory requirement of the TLS fixtures
5b5728c6|2026-11-24|Merge pull request #1083 from tedin7/x2w-compute-store-u32-add-max-or-xor
b66f869b|2026-11-24|shader: lower binary32 add, sub, subrev, mul, min and max in the depth-mask translator
f066e620|2026-11-24|Merge main and preserve the FXSAVE experiment evidence
a7710b73|2026-11-24|Merge pull request #1077 from tedin7/x2w-pixel-aot-cvt-i16-f16
c579c564|2026-11-24|tests: skip the preflight symlink case when Windows refuses symlinks
a02c48bb|2026-11-24|shader: lower plain e32 binary16 v_fmac_f16 and v_ldexp_f16 in pixel_aot
c4369fad|2026-11-24|runtime: match documented PLS and package ABI
8d4767bc|2026-11-24|Merge main and retain both device fixture sets
995b3a31|2026-11-24|Merge remote-tracking branch 'origin/main' into x2w-disasm-vop1-vop2-f16
792ab4cd|2026-11-24|chore: merge origin/main into the scalar compute store branch
593d4295|2026-11-24|docs: make the scanner recognize the decoded scalar-memory mnemonics
48c8b712|2026-11-24|tooling: scanner recognizes the decoded binary16 VOP1 and VOP2 mnemonics
92f3377f|2026-11-24|Validate RFC 1950 file reads through a DirectStorage custom queue
b63a8aa5|2026-11-24|Merge main and keep the returning qword cases
f8407ade|2026-11-24|tooling: add call chains, shared callees and tail jumps to the CPU fuzz test
d410e423|2026-11-24|Merge main and measure native XSAVE selector behavior
c9679bf1|2026-11-24|chore: merge origin/main into the vector shift and subtract branch
d1791fe3|2026-11-24|cpu: lower VLDMXCSR and VSTMXCSR in the host-MXCSR profile
6e29a805|2026-11-24|Merge pull request #1095 from ChazY11/docs/zig-cache-note
6fa152e9|2026-11-24|Merge pull request #1105 from lambirou/texture/bc-volume
45b3c5d2|2026-11-24|Merge pull request #1102 from tedin7/x2w-spirv-sampler-split-1d-cube
2f580b01|2026-11-24|shader: cover wave64 and wave32
4a76439d|2026-11-24|shader: lower waits and scalar loads in pixel_aot masked regions
37d4aaed|2026-11-24|Merge main and preserve both Windows build notes
9369db1a|2026-11-24|docs: record the sampler-views readback difference seen on Intel Xe2
e658fe24|2026-11-24|startup: experimental runtime stand-ins for console executables
c205cffd|2026-11-24|Merge pull request #1114 from archielcqp-a11y/gpu/wave64-emulation-v3
a8692e52|2026-11-24|Merge pull request #1108 from archielcqp-a11y/cpu/shim-map-app-local-warp-v2
18ef8ce7|2026-11-24|Merge main and keep sampler dimension evidence
e672bd10|2026-11-24|gpu: check the desktop sampler contract for CreateSamplerX
71a53aa6|2026-11-24|Merge pull request #1111 from archielcqp-a11y/gpu/alias-registry-first-fit-v2
21d963d3|2026-11-24|Merge pull request #1112 from archielcqp-a11y/tooling/gpu-check-win10-smoke-v2
6278d4fe|2026-11-24|Merge pull request #1088 from tedin7/x2w-spirv-address-window-physical-atomics
add11b9c|2026-11-24|Merge pull request #1113 from archielcqp-a11y/docs/windows-path-length-v2
edee8d07|2026-11-24|Merge pull request #1117 from tedin7/x2w-mesh-entry-f32-sub-mul-add64
7798f7bf|2026-11-24|shader: lower binary32 integer conversions, min and max in the mesh-entry translator
1d46d3e2|2026-11-24|Merge pull request #1092 from tedin7/x2w-disasm-sopk-sopp-ops
5dd14562|2026-11-24|Merge pull request #1104 from tedin7/x2w-pso-readiness-mesh-stage
1f172a06|2026-11-24|Merge main and retain native and emulated wave checks
0a4bd9da|2026-11-24|Merge pull request #1125 from tedin7/x2w-depth-mask-aot-f32-ops
072e85b6|2026-11-24|Merge pull request #1124 from KelboLive/hardware/rtx4070tisuper
20bd771f|2026-11-24|shader: lower IEEE multiply and lt, le and ge kills in the sampled coverage translator
3f8a40bb|2026-11-24|shader: lower bounded 64-bit loads and stores in the SPIR-V address window
026f9abd|2026-11-24|tooling: report the e64 spellings of binary16 arithmetic and rounding as supported
e126db40|2026-11-24|tooling: follow pixel_aot scalar ALU, waits and scratch writes in the cb read trace
2ae62ad2|2026-11-24|gpu: record forced-wave pipeline creation at startup
cdbe4280|2026-11-24|chore: merge origin/main into the 'x2w-disasm-smem-ops' branch
a430e970|2026-11-24|merge: resolve against the merged binary32 add, sub and mul lowering
9e89b22d|2026-11-24|chore: merge origin/main into the 'x2w-disasm-vop1-vop2-f16' branch
699ae547|2026-11-24|chore: merge origin/main into the 'x2w-disasm-sop1-movrel' branch
924198ff|2026-11-24|Merge main and place FXSAVE with the CPU experiments
ace0b8ba|2026-11-23|runtime: commit startup blocks through XMemVirtualAlloc
96309447|2026-11-23|runtime: align large-page startup blocks to 2 MiB
60d9c513|2026-11-24|Merge main and place XSAVE with the CPU experiments
b6802852|2026-11-24|shader: lower the ordered binary32 compares in the depth discard exit
38af829c|2026-11-24|Merge pull request #1109 from lambirou/d3d12x/descriptor-heaps
8174bcbd|2026-11-24|Merge pull request #1090 from tedin7/x2w-compute-store-scalar-alu
619ef878|2026-11-24|Merge pull request #948 from stan495/codex/gfx1030-ds-inc-u64
41608457|2026-11-24|Merge pull request #1127 from tedin7/x2w-pixel-aot-fmac-ldexp-f16
c199a8ed|2026-11-24|tooling: decode v_mul_lo_i32, the quad SAD pair, v_interp_mov_f32_e64 and the d16_hi format buffer ops
e9f7b2ab|2026-11-24|Merge pull request #1126 from lunisahdidouch/tests/preflight-symlink-skip
debca5ec|2026-11-24|Merge pull request #1119 from R0Y00/codex/static-crt-free-tls
1568435d|2026-11-24|Merge pull request #1106 from Jess-Doit/test/d3d12x-struct-return
e9165845|2026-11-24|tooling: add direct-call generators to the CPU fuzz test
f92c2923|2026-11-24|Merge main and preserve binary64 and qword counter controls
063df2cc|2026-11-24|Merge pull request #1133 from ChazY11/docs/sampler-views-xe2
a702ab4a|2026-11-24|docs: record the subgroup width change between dispatches on Intel Xe2
16c60586|2026-11-24|Merge pull request #1122 from lambirou/runtime/pls-xpackage
abe00ca2|2026-11-24|Merge origin/main into x2w-scan-f16-e64-sync
11b5168d|2026-11-24|Merge pull request #1134 from legacy-code-archaeologist/cpu/lower-vmxcsr
4b69ea7f|2026-11-24|gpu: compare the sampler descriptor index source on the same tables
5739a7ab|2026-11-24|Merge pull request #1123 from ChazY11/docs/carrier-temp-directory
ae83dc94|2026-11-24|docs: restore the cpu_aot_fuzz_calls paragraph and remove the duplicated document tail
ccbfbde8|2026-11-24|Merge remote-tracking branch 'origin/main' into x2w-fuzz-cpu-calls
4a477936|2026-11-24|Merge pull request #1121 from Not-Saii/gpu/startup-sampler-x
5691fe81|2026-11-24|startup: register startup_stubs after INTAKE_PYTHON is defined
062b3c5e|2026-11-24|shader: lower binary32 add, subtract and multiply in the compute store profile
e91c53ed|2026-11-24|cpu: produce flags from SHL, SHR and SAR by an immediate in cpu_switch_aot
2a175a8d|2026-11-24|cpu: lower BT, BTS, BTR and BTC in cpu_switch_aot
f1d2c956|2026-11-24|cpu: lower POPCNT, LZCNT and TZCNT in cpu_switch_aot
086d96bb|2026-11-24|Merge pull request #1107 from Jess-Doit/test/claude-1079-array-readback
4edb6dc3|2026-11-24|Merge pull request #1118 from tedin7/x2w-compute-store-vector-shifts-sub
b173cede|2026-11-24|cpu: lower scalar SQRT, MIN and MAX in the scalar SSE translator
3ebaaddb|2026-11-24|test: verify DirectStorage stays pending before manual submit
91fae9e2|2026-11-24|docs: use the source-built XMem allocation evidence
1ced9065|2026-11-24|cpu: accept an inverted unsigned bound check in the jump-table translator
4e0f704f|2026-11-24|Merge pull request #1101 from tedin7/x2w-disasm-smem-ops
f05369ec|2026-11-24|Merge pull request #1132 from lambirou/shader/wave64-coverage
d8fe7a69|2026-11-24|shader: lower indexed gfx1030 tbuffer_load_format_x
6a8130bc|2026-11-24|fix: clear conditional provenance when an unskippable masked-region scalar load redefines an SGPR
95c4e420|2026-11-24|Merge remote-tracking branch 'origin/main' into x2w-pixel-aot-masked-region-waits-loads
89e00bf8|2026-11-24|tooling: decode GFX10 VOP1 and VOP2 SDWA instructions
4a1bccf2|2026-11-24|runtime: add XSystem and XGame contracts
86fd6a33|2026-11-24|test: cover ds dec u64 borrow below the limit
1b09ae0b|2026-11-24|Merge origin/main into x2w-disasm-vop1-vop2-f16
5e775dda|2026-11-24|startup: fail a DirectStorage request instead of the process on an allocation error
fe4fab25|2026-11-24|shader: lower indexed gfx1030 UINT typed-buffer stores
0de06040|2026-11-24|Merge main and keep D3D12 device and SDK GUID sources
e483ad58|2026-11-24|Merge pull request #1154 from tedin7/x2w-sampled-coverage-aot-mul-cmpx
744089ac|2026-11-24|Report six graphics pipeline creations in the WARP startup diagnostic (#1097)
2721db85|2026-11-24|test: isolate sampler selection from output indexing
024338fa|2026-11-23|build: enter an x64 shell and compile C++14 sources with VS 2019 Build Tools
1311db90|2026-11-24|tooling: decode GFX10 VOP1 and VOP2 DPP instructions
1783d78f|2026-11-24|Merge pull request #1053 from Ammmae0x/cpu/sse-fxsave
a02cef83|2026-11-24|Merge pull request #1116 from tedin7/x2w-disasm-sop1-movrel
7017f831|2026-11-24|Merge pull request #1065 from Ammmae0x/cpu/xsave-state
ec6a3163|2026-11-24|test: run the lt compare on the same Y-sampling fixture as the other compares
6e5ed453|2026-11-24|fix: follow decimal float immediates in 32-bit scalar ALU lines of the cb read trace
ec46cda5|2026-11-24|Merge remote-tracking branch 'origin/main' into x2w-pixel-cb-read-salu-waitcnt-sync
2da8071b|2026-11-24|Merge pull request #1161 from roansystemlabs/gpu/870-wave32-startup
41123162|2026-11-24|refactor: share 32-bit scalar operand validation with pixel_aot in the cb read trace
5623244a|2026-11-24|shader: keep successor phi parents valid when a QWORD access splits a block
b30c7f80|2026-11-24|Merge origin/main into x2w-disasm-vop1-vop2-f16
4de52b05|2026-11-24|test: pin malformed VEX MXCSR refusal and per-instruction controls
0718b6b3|2026-11-24|Merge origin/main into x2w-compute-store-aot-f32-alu
4e35649e|2026-11-24|Merge pull request #1162 from tedin7/x2w-disasm-vop3-mubuf-d16
e65d57b3|2026-11-24|Merge pull request #1164 from roansystemlabs/test/446-fuzz-direct-calls
a318037d|2026-11-24|Merge pull request #954 from stan495/codex/gfx1030-ds-min-f64
4e5c1a79|2026-11-24|shader: keep successor phi parents valid after splitting a window atomic
4b740f9c|2026-11-24|Merge pull request #1165 from ChazY11/docs/compaction-width-flip-xe2
a5e9ae5f|2026-11-24|Merge origin/main into x2w-fuzz-cpu-calls: keep only the distinct chain generators as --area chains
33f80eae|2026-11-23|tooling: validate through dxcompiler.dll when the SDK ships no dxil.dll
e132e457|2026-11-24|cpu: lower PF consumers after VEX scalar compares
e28b6077|2026-11-24|d3d12x: add compute list fixture
0ec94d5b|2026-11-24|Merge pull request #1152 from tedin7/x2w-mesh-entry-cvt-minmax
9b6d9dad|2026-11-24|runtime: add title buffer window fixture
7456a93c|2026-11-24|Merge pull request #1128 from R0Y00/codex/dstorage-rfc1950
7d0f3b91|2026-11-24|test: keep the LDS decrement borrow control focused
645b9697|2026-11-24|Merge pull request #1098 from rasptum/runtime/xmem-reserve-commit
da5884af|2026-11-24|startup: describe the DirectStorage option bits by what was measured
c42b1ef0|2026-11-24|Merge main and retain both LDS readback controls
ea81bfdf|2026-11-24|tooling: reject duplicate CTest names in form coverage
416df060|2026-11-24|cpu: lower SETP, SETNP, CMOVP and CMOVNP where the reaching PF definition is modeled
e4284f12|2026-11-24|Run a focused Windows build and test gate on pull requests (#1185)
e85989aa|2026-11-23|gpu: test runtime indexing of a bounded sampler table
95d6aba9|2026-11-24|Merge pull request #1175 from lambirou/runtime/xsystem-xgame
717f1a9e|2026-11-24|Merge pull request #1176 from stan495/codex/gfx1030-tbuffer-load-format-x
b30ac9e5|2026-11-24|Merge pull request #1131 from tedin7/x2w-pixel-aot-masked-region-waits-loads
568f35ad|2026-11-24|shader: track the final predecessor for mixed guarded accesses
b45fb54b|2026-11-24|cpu: keep scalar SQRT, MIN and MAX under the host MXCSR
2d5e7fa2|2026-11-24|docs: count the new scalar fixture exports and record the MXCSR check
044b39c8|2026-11-24|Merge origin/main into x2w-compute-store-aot-f32-alu
e60b3509|2026-11-24|Merge pull request #1155 from tedin7/x2w-scan-f16-e64-sync
f9a634c0|2026-11-24|Merge pull request #1166 from ChazY11/tests/sampler-index-control
d1ea005d|2026-11-24|Merge pull request #1178 from henriazz10/build/msvc-2019-compat
6dd32215|2026-11-24|Merge pull request #1171 from tedin7/x2w-cpu-switch-aot-inverted-unsigned-bound
68b0d836|2026-11-24|Merge pull request #1103 from tedin7/x2w-disasm-vop1-vop2-f16
d5cc87c4|2026-11-24|Merge pull request #1179 from stan495/codex/gfx1030-tbuffer-store-format
008a7bad|2026-11-24|tooling: assess compute stages by ISA recognition in the PSO readiness report
d61477b3|2026-11-24|cpu: lower 256-bit packed integer add, subtract and compare in the AVX translator
1dfb59ef|2026-11-24|feat: split several mixed comparison/ordinary samplers per SPIR-V module
e56af416|2026-11-24|Merge origin/main into x2w-compute-store-aot-f32-alu
de3b1b26|2026-11-24|shader: lower indexed gfx1030 multichannel UINT typed-buffer loads
a34ceea6|2026-11-24|cpu: lower legacy SSE2 cmpsd with predicates 0..7
4e1775cb|2026-11-24|shader: lower aligned gfx1030 packed D16 typed-buffer loads
c02af631|2026-11-24|Merge pull request #1135 from denn0x/startup/runtime-stand-ins
1e34566e|2026-11-24|shader: lower one-source binary32 math and rounding in the mesh entry translator
f9a01b33|2026-11-24|shader: lower the e64 spellings of the sampled coverage multiply and kill compare
10da0549|2026-11-24|Merge main and keep both sampler and QWORD test entry points
832abf98|2026-11-24|test: record the LLVM 15 SDWA decode boundary
beccc1ed|2026-11-24|cpu: lower FNSTSW m16 where every status-word bit is known
d12a6b96|2026-11-24|shader: define float helpers used only by a DS step in the compute store profile
806ee0df|2026-11-24|docs: preserve failures in non-Windows quick starts
401c464f|2026-11-24|Merge pull request #1129 from tedin7/x2w-fuzz-cpu-calls
9dee245d|2026-11-24|Merge pull request #1191 from nanyoudegege/fix/unique-ctest-coverage-names
0dc3c4b7|2026-11-24|build: build the MXCSR harness with the scalar float translations
e03a299c|2026-11-24|cpu: lower packed SSE arithmetic in the SSE FP translator
963ffe5b|2026-11-24|Merge pull request #1183 from lambirou/d3d12x/compute-lists
6c2e7b2a|2026-11-24|Merge pull request #1184 from lambirou/d3d12x/console-buffers
a7539075|2026-11-24|Merge pull request #1120 from lambirou/shader/ds-inc-dec-u64
738bfc48|2026-11-24|startup: track decode length and destination capacity separately, support chunked zlib streams
6a4d18ed|2026-11-24|tooling: flag repeated test sections in GPU report intake
b5cb8800|2026-11-24|Merge pull request #1163 from tedin7/x2w-depth-mask-aot-cmpx-variants
5c66f08d|2026-11-24|Merge pull request #1193 from legacy-code-archaeologist/cpu/lower-parity
59eb7812|2026-11-24|Merge pull request #1186 from Batushn/disasm/vop1-vop2-sdwa
5c6a9564|2026-11-24|test: record the LLVM 15 DPP decode boundary
9259ac84|2026-11-24|Merge pull request #1157 from tedin7/x2w-spirv-address-window-64bit-loads
50f56203|2026-11-24|startup: prevent unsigned overflow in get_slice and add near-wrap control
726d232e|2026-11-24|build: run the FNSTSW m16 controls in the x87 control-word CMake test
c5fd3ff7|2026-11-24|Merge main and keep both sampler-index checks
86dce1f4|2026-11-24|Merge pull request #1190 from MisaXz/tooling/validate-without-dxil-dll
5e04e8aa|2026-11-24|Merge main and retain DPP and SDWA decoding controls
aa639bf8|2026-11-24|Capture every visible render-target pixel after the submitting queue fence (#1172)
39d56ab9|2026-11-24|Merge pull request #1192 from Jess-Doit/test/startup-dynamic-sampler-sm60
6cb34885|2026-11-24|Merge pull request #1187 from Batushn/disasm/vop1-vop2-dpp
56387003|2026-11-24|cpu: export the new cmpsd fixture procedures from the native definition file
16d2031f|2026-11-24|merge: bring in main; keep the successor phi regression test on the shared lowering
e9d79b70|2026-11-24|test: keep the unary mesh draw depth inside the clip range
81c377e5|2026-11-24|Adapt two unsupported null Texture2D views on desktop devices (#1194)
61c14219|2026-11-23|textures: add an opt-in BC7 tiled layout query
9b54fdd7|2026-11-24|cpu: keep the SSE FP MXCSR driver out of translated-form coverage
3835fe22|2026-11-23|startup: translate run-time sampler heap indexing into a bounded table
f38599ca|2026-11-24|Translate VS/PS pipeline stream requests through the startup cache
2f1fc946|2026-11-24|Verify pixels submitted by translated stream pipelines on WARP and hardware
6e07b1bf|2026-11-24|Register stream checks after resolving the fixture interpreter
2aee967d|2026-11-24|Merge pull request #1156 from tedin7/x2w-pixel-cb-read-salu-waitcnt-sync
5d057c49|2026-11-24|Merge pull request #1215 from nanyoudegege/fix/flag-duplicate-gpu-report-sections
704a4236|2026-11-24|Merge main and preserve startup and SDK GUID shim sources
c9c30ea0|2026-11-24|Merge pull request #1212 from nanyoudegege/docs/non-windows-quick-start-failures
40e42aa3|2026-11-24|Merge pull request #1195 from tedin7/x2w-pso-readiness-compute-stage
8cbc3f05|2026-11-24|docs: clarify non-Windows quick-start test coverage
90a58664|2026-11-24|Merge pull request #1197 from tedin7/x2w-cpu-avx-int-aot-avx2-ymm-add-sub
3de5147a|2026-11-24|Merge pull request #1196 from tedin7/x2w-spirv-sampler-split-multi-sampler
aa8a76d4|2026-11-24|Merge pull request #1174 from R0Y00/codex/stream-stage-draw
4d721ad0|2026-11-24|Merge pull request #1208 from dicnunz/cpu/switch-shift-flags
01b955a9|2026-11-24|Merge pull request #1181 from tedin7/x2w-cpu-mxcsr-vex-extra-tests
5ad75b5c|2026-11-24|startup: scope chunk cache to queue and live file sources, isolate standalone streams
a25fb55f|2026-11-24|startup: upload both planes of single-image NV12 textures
bd2fd808|2026-11-24|shader: lower indexed gfx1030 raw subword loads
3ef6ea11|2026-11-24|docs: report the RTX 4060 Laptop intro readback
74001fdc|2026-11-24|Merge main and retain both bit-test and shift flag checks
4b2d84e9|2026-11-24|Merge pull request #1177 from R0Y00/codex/nv12-startup-texture
5a0dbd5c|2026-11-24|Merge pull request #993 from EvilOpus/d3d12xs/iid-exports
65b0bb42|2026-11-24|shader: translate readfirstlane for a uniform scalar-buffer conversion
1d90a8cf|2026-11-24|shader: lower binary32 subrev, sqrt, rsq, floor, trunc, fract and clamp in the dispatch translator
771498b4|2026-11-24|shader: split depth-reference views of any sampler table in the SPIR-V bridge
392ba968|2026-11-24|d3d12x: render a triangle through a graphics list
4919b51a|2026-11-24|Merge pull request #1209 from dicnunz/cpu/switch-bit-tests
27780101|2026-11-23|startup: decode colour-blend registers into a desktop blend state
209745a3|2026-11-24|shader: cover mesh amplification forms
4d1c117f|2026-11-24|docs: distinguish local and CI SPIR-V bridge checks
3c37997f|2026-11-24|Merge main and preserve the NV12 and BC7 layout profiles
3eb1db63|2026-11-24|Merge main and retain bit counts, bit tests and shift flags
1b990574|2026-11-24|docs: advance startup status to the two setup pages
a7ce0018|2026-11-23|startup: guard the sampler heap index in the generated shader
d6c9a773|2026-11-24|startup: keep decode errors distinct from continuations, reject damaged streams
962e979d|2026-11-24|Merge pull request #1206 from stan495/codex/gfx1030-tbuffer-load-format-wide-upstream
06131ae3|2026-11-24|Merge pull request #1218 from Jess-Doit/textures/startup-xg-bc7-layout
fa338d90|2026-11-24|Merge pull request #1210 from dicnunz/cpu/switch-bit-counts
f39021c6|2026-11-24|cpu: lower CLZERO in counted x64 loops
8a3068d3|2026-11-24|Merge pull request #1221 from R0Y00/codex/document-rtx-intro
a2f907dd|2026-11-24|shader: refuse overlapping sampler table views and ignore comments in the sampler bridge
4926f34a|2026-11-24|Merge pull request #1201 from tedin7/x2w-sampled-coverage-aot-e64-variants
349b84e5|2026-11-24|docs: report paired intro decoder timing
02b2b2ed|2026-11-24|shader: keep v_fract_f32 strictly below 1.0 in the dispatch translator
e5f4acf2|2026-11-24|shader: lower aligned gfx1030 packed D16 typed-buffer stores
bb35f045|2026-11-24|startup: support short final destinations in multi-piece streams, guard physical pages
f7ec1944|2026-11-23|startup: check the colour-blend words reported from captured state blocks
3267e4db|2026-11-24|Merge pull request #1231 from R0Y00/codex/intro-loop-timing
29210868|2026-11-24|Merge pull request #1220 from nanyoudegege/docs/clarify-non-windows-check-coverage
5f509209|2026-11-24|Merge pull request #1227 from nanyoudegege/docs/clarify-pr-spirv-testing
f1fa4355|2026-11-23|gpu: bind translated startup sampler heaps
cfff54a1|2026-11-24|Merge pull request #1230 from R0Y00/codex/static-clzero-loops
828a0723|2026-11-24|shader: find the reserved descriptor set from parsed decorations in the sampler bridge
288b1bf2|2026-11-24|tooling: scan the XMemAllocDefault and XMemFreeDefault call sites
3a156358|2026-11-24|Merge pull request #1226 from Jess-Doit/startup/cb-blend-decode
1e0a9834|2026-11-24|Merge main and retain startup sampler and graphics fixtures
7b10acb8|2026-11-24|tooling: decode gfx10 VOP3 interp e64 at 0x200-0x202
857d9731|2026-11-24|Merge pull request #1228 from lambirou/d3d12x/graphics-lists
66ea51ca|2026-11-24|docs: record RTX 2070 Max-Q hardware coverage
80804587|2026-11-24|Merge main and preserve typed and raw buffer loads
f4a6caf4|2026-11-24|Merge pull request #1217 from Jess-Doit/translator/dynamic-sampler-heap
52c67518|2026-11-24|cpu: take the scalar square root from the processor so DAZ is honoured
f84292e7|2026-11-24|Merge main and keep startup shader regressions registered
2cf332e3|2026-11-23|Merge updated #1217 into startup sampler heap binding
ca1c6cd0|2026-11-24|tooling: refuse non-VGPR sources in the gfx10 interp e64 decode
0806ef57|2026-11-24|shader: keep quote state across lines when stripping comments in the sampler bridge
c59161c3|2026-11-24|test: feed the unary mesh draw through runtime inputs and round approximate results
5b1ba999|2026-11-24|Merge pull request #1222 from R0Y00/codex/startup-uniform-buffer-lane
0f496ab6|2026-11-24|startup: refresh CPU-written R8 planes on frame reuse
0e995c19|2026-11-24|Merge pull request #1234 from Alexofbulgaria/docs/rtx2070-maxq-20261124
e7c8bfd2|2026-11-24|startup: treat s_setprio as a scheduling hint in gfx2hlsl
7fb791a4|2026-11-24|Merge pull request #1232 from triggered02/tooling/xmem-scan-default-aliases
fa414483|2026-11-24|Merge pull request #1237 from Jess-Doit/gpu/startup-sampler-heap-binding
ac4bd0cc|2026-11-24|startup: keep fallback GDI off the swap-chain window
4043af1e|2026-11-24|Merge pull request #1182 from tedin7/x2w-spirv-window-atomic-phi-parent
a2baada4|2026-11-23|textures: copy BC7 blocks into the opt-in tiled layout
908c34bf|2026-11-24|Merge pull request #1207 from stan495/codex/gfx1030-tbuffer-store-format-d16-upstream
bd558d64|2026-11-24|shader: keep v_fract_f32 strictly below 1.0 in the mesh entry translator
e3d2eb48|2026-11-24|Merge main and preserve packed D16 load and store
7a4043e4|2026-11-24|Merge pull request #1169 from tedin7/x2w-compute-store-aot-f32-alu
e784374c|2026-11-24|startup: zero-extend unsigned byte vertex fetches
6d9935be|2026-11-23|translator: check 3D image sampling with a source-built draw
709ecf63|2026-11-24|Merge main and retain D16 loads and float lowering
b74f9b91|2026-11-23|Merge remote-tracking branch 'origin/main' into translator/image-3d-fixture
c49b4c24|2026-11-23|Merge remote-tracking branch 'origin/main' into textures/startup-xg-bc7-copy
4f67e704|2026-11-24|startup: scope zlib stream cache to queued continuation entries
55a975ab|2026-11-24|Merge pull request #1211 from stan495/codex/gfx1030-tbuffer-load-format-d16-upstream
4daa4a81|2026-11-24|shader: lower signed DWORD loads, stores and SMin/SMax atomics in the SPIR-V address window
eb9269c9|2026-11-24|shader: lower typed gfx1030 2D image loads
a8be8889|2026-11-24|shader: lower packed-address gfx1030 2D image loads
9b8d55a3|2026-11-24|Merge pull request #1241 from triggered02/startup/setprio-hint
eb4c67f6|2026-11-24|Merge pull request #1245 from R0Y00/codex/startup-ubyte-vertex
692aa6ca|2026-11-24|gpu: emulate a shader-visible view heap larger than the driver accepts
be9c5cb8|2026-11-24|Merge pull request #1242 from R0Y00/codex/presenter-window-selection
372408ad|2026-11-24|shader: keep a NaN input a NaN in the mesh entry v_fract_f32 cap
8526d1f7|2026-11-24|Gate the worker in the separate DirectStorage submission control
ce4f8c7e|2026-11-24|Merge pull request #1204 from tedin7/x2w-c3-sse2-cmpsd
6df746bd|2026-11-24|Merge main and retain raw subword and typed D16 checks
10e03f12|2026-11-24|Merge pull request #1205 from tedin7/x2w-x87-lower-fnstsw-m16
0fa43ad6|2026-11-24|Merge pull request #1214 from Holastor/startup/dstorage-chunked-zlib
9e1bb715|2026-11-24|Merge pull request #1225 from stan495/codex/gfx1030-mubuf-subword-loads
57e35f1b|2026-11-23|shader: lower v_sat_pk_u8_i16 in pixel_aot
d8a478b3|2026-11-24|docs: justify the full v_sat_pk_u8_i16 destination
1aa7aed2|2026-11-24|Merge main and retain both startup R8 experiments
1c2c4b3e|2026-11-24|Merge pull request #1233 from triggered02/tooling/disasm-vintrp-e64
be828a03|2026-11-24|docs: update the offline setup and loading milestone
b429d365|2026-11-24|shader: legal failure semantics, loop-header guard and disjoint stems for signed address-window accesses
c74485f5|2026-11-24|gpu: cover and document the oversized view heap shortcut
9af7d9f5|2026-11-24|Merge pull request #1238 from R0Y00/codex/cpu-plane-refresh
aab473fb|2026-11-24|Merge pull request #1248 from R0Y00/codex/setup-menu-progress
8b5a7124|2026-11-24|shader: keep a NaN input a NaN in the dispatch v_fract_f32 cap
c51633e8|2026-11-24|build: add opt-in sccache arguments to dev.ps1
89d860cf|2026-11-24|startup: add a WASAPI render sink and the startup_audio_output fixture
4d100c9c|2026-11-24|shader: lower indexed gfx1030 raw vector loads
e6f1a2cd|2026-11-24|shader: seed the signed compare-exchange loop without a plain read and scan only physical atomics
af36254c|2026-11-24|shader: keep a NaN input a NaN in the depth-mask v_fract_f32 cap
f9aa4380|2026-11-24|Resolve translated graphics tables against the bound logical view heap
313440a6|2026-11-23|Merge remote-tracking branch 'origin/main' into translator/image-3d-fixture
83d4e18e|2026-11-24|tooling: translate VOP1/VOP2 sub-dword addressing in gfx2hlsl
138aa5a6|2026-11-24|Merge upstream main into gfx1030 typed image loads
16a5a077|2026-11-24|startup: register the audio fixture in the docs, reject failed padding queries, release sinks before COM teardown
fba6b978|2026-11-24|Merge pull request #1168 from tedin7/x2w-cpu-sse-aot-scalar-sqrt-minmax
fbd51285|2026-11-24|Merge updated typed image loads into packed-address image loads
9c39e2be|2026-11-24|build: clear compiler launchers when sccache is missing; Windows test stub
c056fbdb|2026-11-24|Merge pull request #1247 from tedin7/x2w-spirv-address-window-signed-int-psb
5275bb33|2026-11-24|startup: lower explicit-LOD image_sample_l to SampleLevel in gfx2hlsl
4182af8f|2026-11-24|Merge pull request #1243 from Jess-Doit/textures/startup-xg-bc7-copy
64b4a53b|2026-11-24|test: read SystemRoot from os.environ in the dev.ps1 sccache check
4a41ce3a|2026-11-24|Emit image and LDS temporaries in instruction order
48b31812|2026-11-24|Merge pull request #1236 from stan495/codex/gfx1030-mimg-load-2d
3b2032bb|2026-11-24|Merge main and retain both overlay and logical view bindings
544a0cb6|2026-11-24|startup: expose the MIMG a16 modifier so image_sample_l refuses packed addresses
5efd401f|2026-11-23|textures: detile opt-in BC7 backing for startup draws
7b9d561c|2026-11-24|tests: check the launch tool's containment on a source-built probe
33234dd9|2026-11-24|test: keep the pwsh directory out of the isolated sccache check PATH
2c2ca4db|2026-11-24|Merge pull request #1256 from triggered02/tooling/gfx2hlsl-sdwa
ef57a64d|2026-11-24|Merge pull request #1244 from Jess-Doit/translator/image-3d-fixture
3c7b78fb|2026-11-24|Merge pull request #1250 from tedin7/x2w-c3-audio-output
be022bb1|2026-11-24|Exercise placed R8 texture SRVs by guest address on hardware and WARP
30fa57b4|2026-11-24|startup: expose the MIMG tfe modifier so image_sample_l refuses status results
9d05702e|2026-11-24|tests: use wide-character path APIs in the launch probe
588e8777|2026-11-23|translator: keep integer words in flat parameters
d4ecad81|2026-11-24|startup: refuse every unmodelled MIMG bit in image_sample_l and decode lwe from its real bit
7fb2c893|2026-11-24|Merge origin/main into x2w-gfx2hlsl-image-sample-l-explicit-lod
ab1ea74a|2026-11-24|Preserve unsupported interface errors when retrying oversized view heaps
f0741f16|2026-11-24|Merge main and retain ordered image and LDS lowering
33121855|2026-11-24|Merge pull request #1249 from tedin7/x2w-dev-ps1-sccache-opt-in
2cf96392|2026-11-24|Merge pull request #1240 from triggered02/gpu/emulated-view-heap
aa9d605e|2026-11-24|Publish matching placed R8 texture views by their guest address
e28dd285|2026-11-24|Merge pull request #1239 from stan495/codex/gfx1030-mimg-a16
f1b16abd|2026-11-24|Merge pull request #1258 from tedin7/x2w-gfx2hlsl-image-sample-l-explicit-lod
b5810ee6|2026-11-24|shader: lower typed gfx1030 2D image mip loads
bb206279|2026-11-24|runtime: accept the 0x283/0x2004 pool block in the startup commit profile
4c272ccb|2026-11-24|Merge upstream main and retain raw vector load coverage
a578b0b8|2026-11-23|Merge remote-tracking branch 'origin/main' into translator/flat-param-bits
136ed589|2026-11-23|startup: preserve typed pixel colour exports
c6dcaee5|2026-11-24|cpu: lower xsavec and xsavec64 in the XSAVE state translator
3d03af04|2026-11-24|docs: name the xsavec ctest exactly
ceeed757|2026-11-24|docs: record visible language selection during offline startup
2d96a44e|2026-11-24|Merge main into logical descriptor tail branch
99cde8d6|2026-11-24|Merge pull request #1261 from triggered02/x2w-launch-containment
d5f83f7e|2026-11-24|shader: lower v_mad_u64_u32 and v_mad_i64_i32 in pixel_aot
11e948c3|2026-11-24|Draw tiled BC3 and BC7 from two guest addresses with exchanged-tile controls
c3b18721|2026-11-24|tooling: check the frame capture reader against synthetic captures and reject damaged files
b9234523|2026-11-24|Merge pull request #1255 from R0Y00/codex/bound-view-tail
06673a11|2026-11-24|Upload tiled BC3 and BC7 address views on first translated draw
943a9ccd|2026-11-24|Merge pull request #1269 from R0Y00/codex/language-selector-readback
aae7a50c|2026-11-24|Apply typed blend patches to derived graphics pipelines
db080a02|2026-11-24|Preserve logical view and BC7 draw checks when merging main
a39c5849|2026-11-23|shader: test folded inline sampler masks
1e744970|2026-11-24|Retain logical descriptor table draw regression on current main
56f81dd9|2026-11-24|cpu: rewrite instructions the host processor lacks in the startup debugger
c641d1b6|2026-11-23|docs: list the inline sampler check in AGENTS.md
598313ac|2026-11-24|feat: accept non-arrayed Texture3D ordinary sampling in the SPIR-V sampler split
05ed719f|2026-11-24|Merge pull request #1260 from Jess-Doit/textures/startup-xg-bc7-detile
f93ae14f|2026-11-23|Merge branch 'main' of https://github.com/R0Y00/Xbox2Windows into translator/startup-1259-mrt
910607df|2026-11-24|docs: note CMake 3.25 requirement for cached Debug builds
84d500d8|2026-11-24|Reject frame chunks with incomplete headers
7a691883|2026-11-24|runtime: report peak commit and physical-page sections in the startup heartbeat
146e133b|2026-11-24|Preserve descriptor and BC7 draw tests when merging main
95a9cce3|2026-11-24|Merge pull request #1254 from stan495/codex/gfx1030-mubuf-vector-upstream
79f0f84a|2026-11-24|Merge pull request #1264 from rasptum/runtime/xmem-pool-commit-2004
3e814490|2026-11-24|startup: show live_view file opens under the run's package root
3709e10a|2026-11-24|Merge pull request #1263 from R0Y00/codex/startup-memory-srv
f61e4f34|2026-11-24|shader: preserve the other half in gfx1030 raw D16 loads
fac6a522|2026-11-24|Merge pull request #1271 from tedin7/x2w-c3-capture-check
3e48d345|2026-11-23|Merge branch 'main' of https://github.com/R0Y00/Xbox2Windows into translator/startup-1259-mrt
22709a63|2026-11-24|Sample the live page provider and report current-only commit changes
afbcbcff|2026-11-24|cpu: gate native xsavec on CPUID and keep active x87/SSE payloads compared
cf632d50|2026-11-24|Share 4 KiB tile uploads across placed and address-only texture views
02f2f3dc|2026-11-24|Merge pull request #1270 from triggered02/runtime/startup-commit-heartbeat
f203f4c5|2026-11-24|build: register explicit mip texture load checks
03f8980b|2026-11-24|shader: lower explicit mip texture loads at startup
c776675e|2026-11-24|test: verify explicit mip texture loads
fb0c39a4|2026-11-24|docs: record explicit mip texture load verification
bf47f225|2026-11-24|ci: check explicit mip texture fixtures
774265ed|2026-11-24|Synchronize address-view candidate with current main
882d2aa0|2026-11-24|startup: resolve image descriptors inside wider scalar loads
9e4b2818|2026-11-24|build: preserve repository line endings
3303687e|2026-11-24|shader: preserve repository line endings
364fc0ce|2026-11-24|docs: preserve repository line endings
ee3574ca|2026-11-24|ci: preserve repository line endings
db7a3e68|2026-11-24|build: normalize CMake upload to LF
fd8283cf|2026-11-24|shader: normalize translator upload to LF
ac07e11b|2026-11-24|docs: normalize experiment notes upload to LF
c7afbf0e|2026-11-24|ci: normalize workflow upload to LF
ccd18f16|2026-11-24|cpu: lower 256-bit packed integer min and max in the AVX translator
06e764bf|2026-11-24|shader: lower indexed gfx1030 raw subword stores
6e23abc6|2026-11-24|Merge upstream main and retain both gfx1030 experiment rows
a440dabb|2026-11-24|Isolate corrected startup translations from stale shader cache entries
de0754bf|2026-11-24|Reject DirectStorage reads past EOF before allocating staging
62f86d55|2026-11-24|Keep address-view experiment notes valid UTF-8
3212a9c2|2026-11-24|Merge pull request #1279 from nanyoudegege/docs/sccache-debug-cmake-minimum
582d6932|2026-11-24|docs: record readable English legal text during offline startup
93cb6a2a|2026-11-24|Merge pull request #1272 from R0Y00/codex/address-bc-4ks
a846f371|2026-11-24|Merge pull request #1276 from tedin7/x2w-spirv-sampler-split-texture3d
3303f2a1|2026-11-24|Merge pull request #1285 from R0Y00/codex/dstorage-bounded-file-read
40e7766f|2026-11-24|shader: cover SOP1 and SOP2 scalar bit-search, bit-count, sign-extension and arithmetic operations
f83fe017|2026-11-24|Merge pull request #1253 from tedin7/x2w-depth-mask-aot-fract-nan
b2d72f09|2026-11-24|Merge pull request #1287 from R0Y00/codex/english-legal-text-progress
3adc6f38|2026-11-24|Merge pull request #1286 from R0Y00/codex/wide-srv-offsets-menu
1afefd65|2026-11-24|Merge pull request #1282 from stan495/codex/gfx1030-mubuf-d16-upstream
b316ad12|2026-11-24|gpu: answer console allocation sizes for depth-stencil and multisampled textures
5228a3f7|2026-11-24|Keep blend, placed BC7, and address-view draw regressions together
e1416e9e|2026-11-24|startup: persist game-save containers and blobs under the storage path
ca92741e|2026-11-24|startup: per-write staging files, reserved suffix case-insensitively, reusable container table
5b05c67a|2026-11-24|startup: serialise the staging-file rename and retry it on Windows sharing errors
50430a15|2026-11-24|shader: lower the zero-level sample in the sampled coverage translator
8bacaa17|2026-11-24|Keep spaces in paths displayed by the live viewer
64985368|2026-11-24|gpu: create 16-bit packed targets the adapter cannot use as R8G8B8A8
91a470c8|2026-11-24|Merge pull request #1273 from R0Y00/codex/typed-derived-blend
e0b2e867|2026-11-24|gpu: read slot 57 blend state from the colour-blend register words
9bc96316|2026-11-24|Merge pull request #1280 from MasterDomino/startup/live-view-package-root
d5508e72|2026-11-24|Merge pull request #1288 from stan495/codex/gfx1030-mubuf-subword-stores
9c4442fd|2026-11-24|Merge pull request #1289 from agilesh08/feat/shader-sop1-sop2-scalar-ops
b1e812cc|2026-11-24|Merge pull request #1252 from FAJIFBEFJHA/shader/pack-sat-u8
a80d0d9b|2026-11-23|Merge branch 'main' of https://github.com/R0Y00/Xbox2Windows into translator/flat-param-bits
4786e8b2|2026-11-23|Merge branch 'main' of https://github.com/R0Y00/Xbox2Windows into test/startup-1149-inline-sampler
e8f1a64e|2026-11-24|cpu: keep non-default MXCSR live in compacted xsavec saves
c05767dd|2026-11-24|shader: lower indexed gfx1030 formatted UINT buffer loads
924e414b|2026-11-24|Merge pull request #1283 from tedin7/x2w-c3-avx2-ymm-minmax
16326d8a|2026-11-23|Merge branch 'main' of https://github.com/R0Y00/Xbox2Windows into translator/startup-1259-mrt
78ccdd91|2026-11-24|shader: accept any ordinary LOD in the SPIR-V sampler split
e6133013|2026-11-24|Merge main and keep both new startup checks
bab8473f|2026-11-24|Keep gfx1030 image-mip and D16 read documentation when merging main
992116ec|2026-11-24|gpu: validate the description before answering a console size, and check it through the device
c35c3c62|2026-11-24|Merge pull request #1265 from stan495/codex/gfx1030-mimg-load-mip
47e91d85|2026-11-24|Round-trip six 64 KiB standard BC mips through XG and an address SRV
8cccacdf|2026-11-24|tooling: stop at a chosen assert, dump threads mid-run and log large allocations in the startup debugger
d2f014be|2026-11-24|Refresh cached shader pairs for integer flat parameters
6c6b9f2d|2026-11-24|Describe the compacted save fields the implementation writes
4abd9db3|2026-11-24|Merge branch 'main' into shader/startup-image-load-mip
932423c7|2026-11-24|docs: cross-compiling with clang-cl and xwin, and the startup debugger under Wine
1c3dd0ad|2026-11-24|shader: lower one-source binary32 math in the depth mask translator
d61d02ed|2026-11-24|docs: clarify the scope of local startup evidence
d64e72da|2026-11-24|docs: record three GPU coverage retests
6b8b7fa9|2026-11-24|gpu: check live aliases with colliding low 40 address bits
2d60414d|2026-11-24|shader: pack sparse gfx1030 image load masks
f9710f64|2026-11-24|Merge pull request #1262 from Jess-Doit/translator/flat-param-bits
38822019|2026-11-24|Merge pull request #1267 from tedin7/x2w-cpu-xsave-aot-xsavec
256d0b39|2026-11-24|runtime: back physical pages with shared sections and map adjacent pages as one view
22815c09|2026-11-24|Merge pull request #1303 from R0Y00/codex/clarify-diagnostic-startup-scope
a78c3322|2026-11-24|docs: drop the blank line at the end of the startup runtime page
dc73c29e|2026-11-24|Merge pull request #1300 from R0Y00/codex/xg-six-levels
6e029d1a|2026-11-24|Trace colliding low-40-bit guest AS addresses through real DXR geometry
5bcba0cf|2026-11-24|cpu: lift VEX.128 saturating, multiply, unsigned min/max and compare word forms
0ba01c03|2026-11-23|Merge branch 'main' of https://github.com/R0Y00/Xbox2Windows into test/startup-1149-inline-sampler
da6ef404|2026-11-23|Merge branch 'main' of https://github.com/R0Y00/Xbox2Windows into translator/startup-1259-mrt
6c1b5710|2026-11-24|Merge pull request #1305 from R0Y00/codex/gpu-va-high-bits
1c8ad413|2026-11-24|cpu: give every archive texture runner the same app-local WARP
66f373a1|2026-11-24|Merge main and keep the slot 57 blend hookup after the derived-pipeline blend
976540cc|2026-11-24|Report verified BC array tile ranges for XG slot 10
4f3a1320|2026-11-24|test: stop a local variable from shadowing the reference model in the vex words check
b8767e29|2026-11-24|cpu: handle SHA, RDPID, RDPRU and non-temporal scalar stores the host processor lacks
0049a351|2026-11-24|gpu: report the adapter's wave lane range in wave64_emulate
45841d7c|2026-11-24|Merge remote-tracking branch 'origin/main' into runtime/xmem-shared-section-runs
f464f34f|2026-11-24|Merge pull request #1304 from Evan-Lowry/docs/gpu-coverage-retests
0adfe857|2026-11-24|Merge pull request #1307 from R0Y00/codex/dxr-guest-address-readback
854a7ea4|2026-11-24|Merge pull request #1311 from R0Y00/codex/xg-slot10-bc64-arrays
ce9dc423|2026-11-24|runtime: keep the page record consistent when an unmap fails, and bound the fault retry
067968d5|2026-11-24|Merge pull request #1266 from Jess-Doit/translator/startup-1259-mrt
55d7178e|2026-11-24|Merge pull request #1313 from archielcqp-a11y/gpu/wave64-emulate-lane-range
a73e3362|2026-11-24|Merge pull request #1296 from tedin7/x2w-c3-split-any-lod
5ef546e9|2026-11-23|Merge branch 'main' of https://github.com/R0Y00/Xbox2Windows into test/startup-1149-inline-sampler
fc547dd7|2026-11-24|Bind translated vertex views to bounded live placed buffers
d4de15eb|2026-11-24|startup: publish a game-save update only at submit and refuse redirected containers
931de892|2026-11-24|Merge remote-tracking branch 'origin/main' into cpu/startup-host-cpu-rewrites
9bbc1af7|2026-11-24|cpu: log and skip a site whose replacement cannot be planned
87088c1e|2026-11-24|Merge branch 'cpu/startup-host-cpu-rewrites' into cpu/startup-host-cpu-sha-rdpid
687312db|2026-11-24|Merge pull request #1308 from tedin7/x2w-c3-vex-words2
3338f5ec|2026-11-24|tooling: accept an empty X2W_VALLOC_MIN and name the logged address as requested
c0aed1de|2026-11-24|Merge remote-tracking branch 'origin/main' into tooling/startup-debugger-stop-dump-valloc
2b547199|2026-11-24|Merge pull request #1310 from stan495/codex/gfx1030-mimg-sparse-dmask
f5d83c31|2026-11-24|Merge pull request #1309 from archielcqp-a11y/cpu/archive-texture-app-local-warp
1e95fcae|2026-11-24|tests: distinguish MinLod translation from adapter rounding
d6b54dfc|2026-11-24|docs: explain fractional MinLod verification
ffc92c06|2026-11-24|Share nine-mip BC layout across XG copies and array SRV sampling
0ae25b34|2026-11-24|Merge pull request #1224 from tedin7/x2w-spirv-sampler-views-general-table
429fb8f7|2026-11-24|docs: quote the SDK paths inside the clang-cl toolchain flags
d6225375|2026-11-24|Merge remote-tracking branch 'origin/main' into docs/linux-clang-cl-wine-startup
3185186e|2026-11-24|docs: say which directories were tried with spaces
7cb28638|2026-11-24|Merge main and keep both startup shortcut notes
a970e1a0|2026-11-24|cpu: accept the 8087/287 x87 controls as FNOP in the x87 profile
1a2b32b7|2026-11-24|Merge main and keep both test registrations
32fc727d|2026-11-24|Merge main and keep both test registrations
580299da|2026-11-24|shader: lower packed gfx1030 2D image loads
a0e6ca94|2026-11-24|Roll back placed-buffer registration on allocation failure
4d7509aa|2026-11-24|Merge pull request #1318 from R0Y00/codex/xg-nine-bc-copy
985e8e33|2026-11-24|cpu: submit a placed vertex draw from a lifted PE entry
f208f3b6|2026-11-24|runtime: remove the fault-retry handler when xmem.dll unloads
1988aa97|2026-11-24|Merge remote-tracking branch 'origin/main' into runtime/xmem-shared-section-runs
3437253a|2026-11-24|tooling: decode v_permlane16_b32 and v_permlanex16_b32 in disasm_rdna2
29757986|2026-11-24|runtime: verify absolute archive offsets in DirectStorage batches
dce4ce6d|2026-11-24|startup: trace declared provider outputs behind an opt-in switch
06b11c2e|2026-11-24|Merge pull request #1319 from archielcqp-a11y/cpu/x87-legacy-controls
8cbb4c03|2026-11-24|Merge pull request #1302 from tedin7/x2w-depth-mask-aot-unary-f32-math
04db799b|2026-11-24|Merge pull request #1297 from stan495/codex/gfx1030-mubuf-format-uint-upstream
436ac1bf|2026-11-24|Merge pull request #1025 from zohaiblazuli/gpu/raw-uav-windows
54ecfced|2026-11-24|Merge remote-tracking branch 'origin/main' into codex/materialize-placed-vertex-view
641be458|2026-11-24|Resolve slot-133 destination and scratch ranges for a desktop DXR build
e3193607|2026-11-24|Merge pull request #1323 from R0Y00/codex/dstorage-directory-offsets
cd953f74|2026-11-24|Merge pull request #1317 from Evan-Lowry/tests/minlod-adapter-reference
0ecae4ec|2026-11-24|cpu: lower xsaveopt and xsaveopt64 in the XSAVE state translator
8ce2a037|2026-11-24|shader: accept the s_mov_b64 close of an s_and_saveexec_b64 masked region
a166e42f|2026-11-24|Merge pull request #1316 from R0Y00/codex/materialize-placed-vertex-view
bb6e42e3|2026-11-24|Merge pull request #1322 from MasterDomino/tooling/disasm-permlane
632b8939|2026-11-24|tooling: add a jump-table switch area to the CPU differential fuzz
60ddf539|2026-11-24|shader: lower signed atomic add, exchange, and, or, xor and compare-exchange in the address window
fcbe10bc|2026-11-24|Merge remote-tracking branch 'origin/main' into HEAD
47dfb267|2026-11-24|startup: count null descriptor bindings by cause
d9d8b46a|2026-11-24|startup: leave bound identity fallbacks out of the null counts and read the totals under the draw lock
b4227960|2026-11-24|cpu: draw BC7 pages requested by a recompiled archive reader
17e2a04c|2026-11-24|Merge pull request #1213 from tedin7/x2w-cpu-sse-fp-aot-packed-arith
3795037e|2026-11-24|startup: decode BCPack DirectStorage requests into destination memory (Refs #1139)
d5ea3ed4|2026-11-24|Merge pull request #1202 from tedin7/x2w-mesh-entry-aot-f32-unary-math-round
1cdaf442|2026-11-24|Merge pull request #1275 from Jess-Doit/test/startup-1149-inline-sampler
23ff04e2|2026-11-24|shader: lower gfx1030 formatted UINT buffer stores
8d6c5477|2026-11-24|startup: bound the API trace by each declared output's width
184637f4|2026-11-24|Merge remote-tracking branch 'origin/main' into runtime/xmem-shared-section-runs
68f23ea3|2026-11-24|Merge pull request #1321 from stan495/codex/gfx1030-mimg-packed-loads
0f09ca93|2026-11-24|Merge pull request #1326 from R0Y00/codex/slot133-dxr-build
2cb2312a|2026-11-24|shader: accept gathers in the SPIR-V sampler split
012e142a|2026-11-24|tooling: name unbound operand reads in pixel_aot and count them separately in the shader scan
9d0e43ee|2026-11-24|shader: lower integer v_cmpx region openers
769c83e1|2026-11-24|runtime: pin xmem.dll while the fault handler is registered instead of removing it at detach
4a518b2e|2026-11-24|Merge pull request #1257 from tedin7/x2w-c3-null-bindings
395be5b6|2026-11-24|Merge remote-tracking branch 'origin/main' into gpu/startup-slot57-blend-hookup
1faaeada|2026-11-24|Merge pull request #1223 from tedin7/x2w-ffx-dispatch-aot-vop3-f32-math-and-mods
ce8d9d00|2026-11-24|test: skip the GatherCmp HLSL checks when SPIRV-Cross cannot write them
b1382a66|2026-11-24|gpu: slot 57 blend draw fixture; a refused target leaves the registers unused
23376f94|2026-11-24|cpu: track MXCSR definition separately in the XSAVE translator
73cddb3f|2026-11-24|Merge pull request #1330 from tedin7/x2w-spirv-address-window-signed-atomics
c263c4a6|2026-11-24|cpu: lift BSF and BSR in the switch translator
bb90a3d0|2026-11-24|shader: lower gfx1030 formatted D16 UINT buffer loads
728e41db|2026-11-24|Merge pull request #1331 from R0Y00/codex/archive-directory-gpu
115516fc|2026-11-24|Merge pull request #1329 from tedin7/x2w-c3-fuzz-switches
bec0c399|2026-11-24|Merge pull request #1328 from MasterDomino/shader/region-mov-close
c919ad07|2026-11-24|Merge pull request #1278 from JB2WIN/shader/mad-u64-u32
9731fae2|2026-11-24|startup: report every changed output and cover the API trace with a control
44a862fe|2026-11-24|Merge remote-tracking branch 'origin/main' into gpu/startup-packed-target-formats
78dbf135|2026-11-24|Merge remote-tracking branch 'origin/main' into gpu/startup-console-plane-sizes
4f819364|2026-11-24|Merge pull request #1333 from stan495/codex/gfx1030-mubuf-format-stores
8ad61a6e|2026-11-24|Merge pull request #1291 from tedin7/x2w-c3-coverage-sample-lz
5bbfbc01|2026-11-24|Merge remote-tracking branch 'origin/main' into codex/pe-placed-view-draw
f6f4146c|2026-11-24|Merge remote-tracking branch 'origin/main' into runtime/xmem-shared-section-runs
97a63a54|2026-11-24|Reserve each async save block before committing its staged files
a31b7088|2026-11-24|Merge pull request #1327 from tedin7/x2w-cpu-xsave-aot-xsaveopt
1238d7db|2026-11-24|docs: distinguish startup, archive drawing and shader compilation
45e246e5|2026-11-24|cpu: lift VEX.128 vpsraw, vpsllw and vpsrlw with an XMM or memory count
9e35fe29|2026-11-24|cpu: lower 256-bit packed abs, sign and word shuffle forms
dde8afd2|2026-11-24|Merge pull request #1301 from JohnConnorNPC/docs/linux-clang-cl-wine-startup
9f608935|2026-11-24|runtime: make shared views opt-in with X2W_XMEM_SHARED_VIEWS
dce00071|2026-11-24|Merge pull request #1320 from R0Y00/codex/pe-placed-view-draw
d6cbd5d7|2026-11-24|Merge pull request #1340 from R0Y00/codex/readme-archive-to-pixels
6ebbcc58|2026-11-24|shader: lower gfx1030 whole-texel D16 UINT buffer stores
f26519a5|2026-11-24|test: require DXC and read back the four gathered values of a plain and a comparison gather
8b82ced3|2026-11-24|startup: publish a multi-blob game-save update under a cross-process lock
4f3f79a2|2026-11-24|cpu: select BC7 storage ranges from a bounded guest directory
07817325|2026-11-24|Merge remote-tracking branch 'origin/main' into gpu/startup-slot57-blend-hookup
b27f4eed|2026-11-24|shader: lower float loads and stores in the SPIR-V address window
46d20eab|2026-11-24|shader: lower binary64 comparisons on their source words
b5804f60|2026-11-24|startup: wait on the GPU for console queue Wait (slot 17)
90e84561|2026-11-24|bcpack: enforce flushed-fragment home state and add test coverage
a43e9f8f|2026-11-24|startup: treat every not-granted lock request as busy and report the real status
30c48924|2026-11-24|Merge pull request #1339 from tedin7/x2w-c3-switch-bsf-bsr
eca20d64|2026-11-24|Merge pull request #1324 from triggered02/tooling/api-trace
07f0d40b|2026-11-24|shader: lower gfx1030 fixed-address integer atomic reductions
8ca2c58d|2026-11-24|Merge pull request #1306 from JohnConnorNPC/runtime/xmem-shared-section-runs
3d48314d|2026-11-24|Merge pull request #1335 from archielcqp-a11y/shader/vcmpx-integer
b82fe73f|2026-11-24|Merge pull request #1343 from stan495/codex/gfx1030-mubuf-format-d16-loads
7f3b070f|2026-11-24|shader: lower image_sample_l with a buffered level in the sampled coverage translator
c4966c64|2026-11-24|Merge pull request #1294 from JohnConnorNPC/gpu/startup-slot57-blend-hookup
929f6b13|2026-11-24|shader: accept plain VOP3 spellings in the depth mask translator
5f994492|2026-11-24|cpu: draw a selected page from a sixteen-page zlib record
2f9c55d9|2026-11-24|test: give every game-save test run its own directory (process id plus clock)
91aefe89|2026-11-24|cpu: submit two placed DXR build requests from a lifted PE
f8ae5eb2|2026-11-24|Track queue wait state without a global allocation
15087752|2026-11-24|Merge pull request #1345 from R0Y00/codex/guest-directory-selection
843f8d4b|2026-11-24|Merge pull request #1246 from tedin7/x2w-c3-xgamesave-persist
7eead4da|2026-11-24|Merge pull request #1341 from tedin7/x2w-c3-vex-xmm-count-shifts
d2f4165d|2026-11-24|Merge pull request #1342 from tedin7/x2w-cpu-avx-int-aot-ymm-abs-sign-shuf
f35c8d25|2026-11-24|Merge pull request #1332 from tedin7/x2w-c3-split-gather
143a6bbe|2026-11-24|Merge remote-tracking branch 'origin/main' into gpu/startup-console-plane-sizes
874463fd|2026-11-24|Merge remote-tracking branch 'origin/main' into gpu/startup-packed-target-formats
30e1943e|2026-11-24|Merge remote-tracking branch 'origin/main' into cpu/startup-host-cpu-sha-rdpid
704f5445|2026-11-24|Merge pull request #1347 from tedin7/x2w-c3-spirv-address-float
4dd0f73e|2026-11-24|Merge remote-tracking branch 'origin/main' into HEAD
90a707ef|2026-11-24|tooling: label unbound operands as unbound under the scanned inputs
1e8b0570|2026-11-24|Merge pull request #1356 from R0Y00/codex/aot-placed-dxr-batch
623e2112|2026-11-24|Merge pull request #1348 from Jess-Doit/startup/queue-wait-slot
b274877b|2026-11-24|cpu: accept FFREEP ST(0) on a nonempty stack in the x87 profile
fc6d25db|2026-11-24|startup: keep closed lists submittable across their allocator's Reset
597f1710|2026-11-24|shader: lower v_cvt_f64_i32 and v_cvt_f64_u32 in pixel_aot
1cfd4962|2026-11-24|tooling: add a scalar float and double area to the CPU differential fuzz
f008456f|2026-11-24|gpu: compare dxr_address_ranges with a direct-address DXR control
a53a750d|2026-11-24|cpu: lower SSE2 packed double arithmetic in the SSE FP leaf translator
253ce62e|2026-11-24|build: define INTAKE_PYTHON before its first use
1fd5cfba|2026-11-24|tests: add BC7 Mode 1 and wrong-destination-size DirectStorage cases
e9a84c0d|2026-11-24|shader: treat OpImageQueryLod as an ordinary sampler use in the sampler bridge
4b166b7d|2026-11-24|Merge pull request #1353 from R0Y00/codex/guest-sixteen-page-record
e99dfccd|2026-11-24|shader: lower subgroup ballot find-lowest-bit in the SPIR-V bridge
f2082414|2026-11-24|cpu: start the load-config /guard:cf fixtures with an explicit CFG policy
c7d6b4c2|2026-11-24|Merge pull request #1290 from JohnConnorNPC/gpu/startup-console-plane-sizes
2e2a90e8|2026-11-24|Merge pull request #1360 from MasterDomino/gpu/dxr-address-direct-control
f23753c2|2026-11-24|Merge pull request #1346 from stan495/codex/gfx1030-mubuf-format-d16-stores
64e7b152|2026-11-24|Merge pull request #1350 from tedin7/x2w-sampled-coverage-image-sample-l
a75c252d|2026-11-24|Merge pull request #1337 from MasterDomino/tooling/scan-unbound-operand
f38881cd|2026-11-24|Merge pull request #1351 from tedin7/x2w-c3-depth-mask-e64
7cdb2c4d|2026-11-24|Match ID3D12Device11 by IID instead of a header type
dad66916|2026-11-24|tooling: decode v_mqsad_u32_u8 with four-register destination and accumulator
d6e557bf|2026-11-24|Compile the D3D12.X PSO shader without the unused strip pass
8d062f33|2026-11-24|startup: add a steady-versus-bursty playback-timing control
b9c9f624|2026-11-24|Merge pull request #1361 from legacy-code-archaeologist/cpu/x87-ffreep
8e2162de|2026-11-24|Merge main into startup/allocator-pool; cover a Reset while the GPU waits
6b8ad31c|2026-11-24|shader: lower one structured wave64 else arm in pixel_aot masked regions
ef9b7d6d|2026-11-24|Merge branch 'main' of https://github.com/R0Y00/Xbox2Windows into startup/allocator-pool
e1316e1c|2026-11-24|Merge pull request #1354 from stan495/codex/gfx1030-mubuf-atomic-reductions
b70f516a|2026-11-24|test: round-trip only the mqsad vectors LLVM 15 assembles
98d2c3e8|2026-11-24|shader: lower gfx1030 fixed-address 64-bit atomic reductions
5e221a3d|2026-11-24|tooling: record the gfx1030 DWORD load in D16 fixtures
7fc6a118|2026-11-24|Merge remote-tracking branch 'origin/main' into gpu/startup-packed-target-formats
a14064ec|2026-11-24|startup: never reuse an allocator whose submit could not be tracked
69155425|2026-11-24|Merge remote-tracking branch 'origin/main' into cpu/startup-host-cpu-rewrites
aa529520|2026-11-24|cpu: fall back to a NOP when the host lacks CLFLUSHOPT too
77396bf6|2026-11-24|Merge branch 'cpu/startup-host-cpu-rewrites' into cpu/startup-host-cpu-sha-rdpid
0972768a|2026-11-24|﻿startup: match the pacing control's totals and report per-phase deltas
994823a3|2026-11-24|Merge pull request #1364 from RenzoWit35/build/intake-python-before-use
384ee4ac|2026-11-24|cpu: lift BSWAP in the switch translator
028efdfe|2026-11-24|cpu: lift memory-source forms of the AVX logical ops
d40de849|2026-11-24|Merge pull request #1358 from tedin7/x2w-c3-fuzz-scalar-fp
f303097e|2026-11-24|Merge pull request #1362 from tedin7/x2w-c3-spirv-query-lod
6827017d|2026-11-24|Merge pull request #1365 from tedin7/x2w-spirv-ballot-findlsb
cb0ae6b1|2026-11-24|Merge pull request #1359 from tedin7/x2w-cpu-sse-fp-pd
52c8486f|2026-11-24|Merge pull request #1369 from FoxxoOwO/build/windows-sdk-22621
a1a0c149|2026-11-24|Resolve f64 comparison and cmpx region overlap
6201ab60|2026-11-24|gpu: tag packed stand-in textures, refuse views that do not fit, add a draw fixture
9845a1a8|2026-11-24|Merge remote-tracking branch 'origin/main' into gpu/startup-packed-target-formats
6518a75a|2026-11-24|feat: add ceil, exp2 and log2 to the depth-mask translator
10009a94|2026-11-24|﻿startup: pair CoInitializeEx with CoUninitialize in the pacing control
8d1c18cf|2026-11-24|Merge pull request #1336 from Holastor/bcpack/dstorage-decode
a819b80c|2026-11-24|shader: diagnose unsupported gfx1030 MUBUF cache invalidations
afaa702f|2026-11-24|Merge pull request #1368 from triggered02/startup/audio-pacing-control
ea5884d3|2026-11-24|Merge pull request #1349 from archielcqp-a11y/shader/cmp-f64
7ea035ce|2026-11-24|shader: lower four-float loads and stores in the SPIR-V address window
6963edab|2026-11-24|shader: lower image_sample_b with a buffered bias in the sampled coverage translator
382d1db2|2026-11-24|shader: lower typed gfx1030 2D image stores
3cad6b96|2026-11-24|Merge pull request #1374 from tedin7/x2w-cpu-switch-bswap
81818136|2026-11-24|cpu: limit the CFG-policy child to its own handles and record a hung launch
be57eb2d|2026-11-24|Merge pull request #1372 from stan495/codex/gfx1030-mubuf-atomic-x2-upstream
a34a97dc|2026-11-24|Merge pull request #1373 from stan495/codex/gfx1030-mubuf-dword-coverage
7261ab67|2026-11-24|Merge pull request #1357 from Jess-Doit/startup/allocator-pool
85523e79|2026-11-24|shader: lower gfx1030 fixed-address atomic exchanges
34f03603|2026-11-24|gpu: translate 2D array images in the startup shader translator
098cf79a|2026-11-24|cpu: end the ordinal-package and startup fixture EXEs with their entry result
f5c0be11|2026-11-24|cpu: lower 256-bit packed integer saturating add and subtract in the AVX translator
e4112744|2026-11-24|shader: restore outstanding loads at an else transition after a then-arm skip
5e94fe47|2026-11-24|Use exact green in the B5G6R5 draw control
c4350165|2026-11-24|Merge pull request #1274 from JohnConnorNPC/cpu/startup-host-cpu-rewrites
166a6a9a|2026-11-24|cpu: lower scalar CVTSS2SI and CVTSD2SI by the MXCSR rounding control
d20fef65|2026-11-24|Merge pull request #1312 from JohnConnorNPC/cpu/startup-host-cpu-sha-rdpid
d2a1778c|2026-11-24|Merge pull request #1292 from JohnConnorNPC/gpu/startup-packed-target-formats
844cc9ec|2026-11-24|Integrate binary64 conversions with main comparisons
f4996ca1|2026-11-24|Merge pull request #1376 from tedin7/x2w-c3-avx-move-mem
57cdfceb|2026-11-24|Merge pull request #1377 from tedin7/x2w-depth-mask-ceil-exp-log
0d68d6db|2026-11-24|Merge main and retain startup regression checks for #1381
d926b1f9|2026-11-24|Merge main and retain startup regression checks for #1284
1b00fb9e|2026-11-24|startup: handle partial pointer reads in allocation diagnostics
d72ee8b9|2026-11-24|Merge current main into startup diagnostic review
748b9cdb|2026-11-24|cpu: lift vmovupd and vmovapd in the AVX quadword move translator
23319ff2|2026-11-24|Merge pull request #1383 from Maxim-Root/cpu/fixture-entry-exit
816fb71b|2026-11-24|Merge pull request #1371 from MasterDomino/shader/region-else
3696ec96|2026-11-24|Reconcile array and mip translations at their verified dimensions
998ee7da|2026-11-24|Merge pull request #1381 from JohnConnorNPC/gpu/startup-translator-2d-array
9d8acc73|2026-11-24|Merge pull request #1363 from JB2WIN/shader/cvt-f64-int
f236a41b|2026-11-24|Reject raw atomic reductions on typed image UAV bindings
5084945b|2026-11-24|Merge pull request #1299 from JohnConnorNPC/tooling/startup-debugger-stop-dump-valloc
a2e2a905|2026-11-24|cpu: keep the jump-table unsupported-instruction control unsupported
ddc9fce4|2026-11-24|docs: record the per-adapter MinLod mip rule, including the Arc floor case
8a6d5854|2026-11-24|Merge pull request #1386 from R0Y00/codex/priority-integrate-1284
54838ce6|2026-11-24|shader: lower ceil and the mul:4 and div:2 output modifiers in the dispatch translator
695805a3|2026-11-24|Merge pull request #1344 from stan495/codex/gfx1030-mimg-typed-stores
2e43a268|2026-11-24|docs: tell contributors to run the adapter tests serially
ba073fca|2026-11-24|Skip the compile-time check on a DXC without -ftime-report
de21f76c|2026-11-24|docs: clarify the Windows SDK choice for a first build
f59ce4e4|2026-11-24|Merge pull request #1388 from Maxim-Root/cpu/jump-tables-unsupported-control
eebdd91d|2026-11-24|Merge pull request #1389 from ChazY11/docs/minlod-clamp-floor-xe2
f1b20566|2026-11-24|docs: scope the "only the Windows 10 in-box WARP" MinLod sentence
7a147016|2026-11-24|cpu: lift legacy SSE2 cmppd in the SSE FP translator
1f0244de|2026-11-24|tooling: record the forms of the CLZERO fixture for form coverage
57659afc|2026-11-24|docs: note that the developer environment is per PowerShell session
e6071428|2026-11-24|﻿startup: add a voice-lifecycle control with explicit diagnostic and system providers
da8bfc74|2026-11-24|Merge pull request #1370 from tedin7/x2w-c3-disasm-mqsad
4875082d|2026-11-24|Merge pull request #1379 from stan495/codex/gfx1030-mubuf-cache-diagnostics
33237f61|2026-11-24|Merge pull request #1382 from tedin7/x2w-c3-avx-int-ymm-sat
35d9d346|2026-11-24|Merge pull request #1387 from tedin7/x2w-c3-avx-fp-vmovapd-upd
abaf5549|2026-11-24|Merge pull request #1385 from tedin7/x2w-cpu-sse-rounded-cvt
052f3c51|2026-11-24|Merge pull request #1380 from tedin7/x2w-sampled-coverage-image-sample-bias
b80149a8|2026-11-24|Merge pull request #1378 from tedin7/x2w-c3-spirv-address-vec4
5b64197e|2026-11-24|Merge pull request #1384 from stan495/codex/gfx1030-mubuf-atomic-exchanges-upstream
b898d318|2026-11-24|shader: lower packed gfx1030 2D image stores
f78ff728|2026-11-24|docs: add six new adapters to the hardware coverage table
06395b06|2026-11-24|cpu: restore the compacted XSAVE format in XRSTOR
a8e757a1|2026-11-24|Merge pull request #1394 from nanyoudegege/docs/clarify-first-build-windows-sdk
19efd4da|2026-11-24|docs: add two missing startup checks to the AGENTS.md Python list
d9509b57|2026-11-24|Describe local GPU test contention precisely
a8946ec0|2026-11-24|Merge pull request #1395 from ChazY11/docs/minlod-only-clause
1ac40729|2026-11-24|Clarify which shell receives the developer environment
c4fb0f7f|2026-11-24|runtime: check the DirectStorage stand-in's zlib decoder against zlib
04bc596e|2026-11-24|Bound PCM submissions and synchronize audio completion recording
ea978727|2026-11-24|gpu: give a descriptor heap plain memory when its desktop start is not an address
237d4b85|2026-11-24|tooling: reject unnamed CTest cases in coverage reports
0588fd7c|2026-11-24|tests: pass quirk keyword argument to cmp_reference in sse_fp_pd_check
3b6f3e6d|2026-11-24|docs: record the RX 7800 XT inline dxr_opacity TMin scan (#1011)
829d3208|2026-11-24|Merge pull request #1393 from FoxxoOwO/test/dxc-ftime-report-skip
34ea6548|2026-11-24|Merge pull request #1396 from Maxim-Root/tooling/clzero-form-coverage
a32349de|2026-11-24|Merge pull request #1392 from ChazY11/docs/serial-gpu-tests
70210624|2026-11-24|Merge pull request #1398 from ChazY11/docs/fresh-shell-note
07211552|2026-11-24|Merge pull request #1399 from triggered02/startup/audio-provider-control
dd73bde1|2026-11-24|tooling: add mid-body RSP generators to the CPU fuzz test
429a8b92|2026-11-24|Merge pull request #1397 from tedin7/x2w-sse-fp-cmppd
dc16b29b|2026-11-24|startup: read the whole destination page array when a storage request runs
7b6e5896|2026-11-24|shader: accept v_cmpx_class_f32 as a pixel_aot region opener
639ef5a1|2026-11-24|cpu: require SSE RFBM bit for MXCSR liveness in XRSTOR AOT
a953c41a|2026-11-24|runtime: check the zlib decoder's distance bound on bare deflate data
85c6183e|2026-11-24|Reject descriptor heaps when required CPU backing cannot be allocated
04e307cf|2026-11-24|Merge pull request #1400 from RenzoWit35/docs/hardware-coverage-new-adapters
10ab5f9b|2026-11-24|docs: record gpu_alias_window_hardware on an RDNA 2 adapter
226461aa|2026-11-24|Merge pull request #1402 from RenzoWit35/docs/agents-python-checks-sync
908a72ad|2026-11-24|Merge pull request #1406 from wasonidas/tooling/require-ctest-case-names
4957e609|2026-11-24|gpu: retry a pending pipeline manifest on later draws
76745be2|2026-11-24|Merge pull request #1405 from stan495/codex/gfx1030-mimg-packed-stores
9ce46ef8|2026-11-24|Merge pull request #1407 from trick-ye/docs/dxr-opacity-rx7800xt-tmin
73e2110f|2026-11-24|cpu: add non-default entry-MXCSR control for AVX-only compacted XRSTOR
77d7a36c|2026-11-24|gpu: translate image_load_mip on 2D arrays
0a679c06|2026-11-24|Merge pull request #1391 from tedin7/x2w-c3-ffx-ceil-scale
b448ad2a|2026-11-24|tests: bind SSE compacted MXCSR control outside single-input xsave_pairs loop
bb3d5bd4|2026-11-24|tests: include new xsaveopt controls in host gating check
153bb5f6|2026-11-24|Merge pull request #1403 from JohnConnorNPC/gpu/startup-tagged-descriptor-handles
220f060f|2026-11-24|docs: refresh the fractional MinLod consequence after the fixture changed
c6571ee9|2026-11-24|cpu: accept FFREE of a statically empty register in the x87 profile
b70aa2ed|2026-11-24|gpu: log the exact fields of a refused slot 70 memory SRV call
64b6d2d3|2026-11-24|fix: return ApuFree blocks to the APU heap stand-in through a coalescing free list
f04313cf|2026-11-24|docs: record the comparison-sampler split readback on Intel Xe2
dc70f46b|2026-11-24|shader: lower gfx1030 2D image resource queries
2e8eebe3|2026-11-24|docs: record the ballot bit extraction readback on Intel Xe2
2dfe4df1|2026-11-24|test: rename small to tiny_buf for MSVC rpcndr.h compatibility
07329433|2026-11-24|Merge pull request #1416 from R0Y00/codex/startup-reload-published-shader
7c826335|2026-11-24|Merge pull request #1414 from Asimmimx/docs/gpu-alias-window-rdna2
3571ef42|2026-11-24|Merge pull request #1417 from tedin7/x2w-frontier-gpu3-gfx2hlsl
eebc6c80|2026-11-24|Merge pull request #1404 from RenzoWit35/runtime/startup-inflate-check
82680176|2026-11-24|gpu: compare the placed DXR hit distance within 1e-5 of 1
22961a53|2026-11-24|docs: record the address-window and atomic readbacks on Intel Xe2
4e69fd96|2026-11-24|runtime/inflate: return structured InflateStatus via inflate_zlib_ex
ff03ffc4|2026-11-24|d3d12_xs: wire slot 50 DSV binding, view_slot<21> tracking, and stream PSO depth-stencil subobjects
8f9faa92|2026-11-24|shader: accept binary16 and binary64 v_cmpx forms as pixel_aot region openers
7dbe5161|2026-11-24|tests/startup_stream_depth: use correct guest vtable slots 9/10/14/8 for list/queue/fence
49776ef6|2026-11-24|gpu: add reductions, prefixes and quad reads to wave64 emulation
4c4c9795|2026-11-24|Merge pull request #1418 from ChazY11/docs/fractional-contract-refresh
7502a3f2|2026-11-24|Merge pull request #1422 from ChazY11/docs/compare-sampler-arc
bb4272f5|2026-11-24|runtime: read a persisted blob through async save slots 21 and 22
18632ead|2026-11-24|Merge pull request #1423 from ChazY11/docs/ballot-arc
491bf25d|2026-11-24|Merge pull request #1427 from ChazY11/docs/address-window-arc
b55de3c9|2026-11-24|Merge pull request #1409 from RenzoWit35/tooling/fuzz-cpu-mid-body-rsp
98c1b943|2026-11-24|cpu: keep standard-only XRSTOR default and gate compacted path on --compacted-xrstor
45324bcb|2026-11-24|Merge pull request #1411 from Maxim-Root/shader/cmpx-class-f32
92bd3362|2026-11-24|Merge pull request #1424 from stan495/codex/gfx1030-mimg-resinfo
f4442ee8|2026-11-24|tests: test compacted_xrstor=True explicitly for AVX-only MXCSR refusal
88dd9484|2026-11-24|shader: lower v_cvt_f64_f32 for statically known sources in pixel_aot
f9acc69d|2026-11-24|docs: record the independent BVH differential run on Intel Xe2
71fbc403|2026-11-24|tests/docs: address Codex P2 findings on #1421
ff2297cc|2026-11-24|shader: lower a gfx1030 four-sample image_msaa_load
35a913a4|2026-11-24|startup_stubs: add GameInput keyboard readings and GetPreviousReading history
a6879558|2026-11-24|startup_stubs: add R32G32B32A32/R16G16/R32/R16/R8 MSAA and D24_UNORM_S8_UINT console plane sizing
ebb3ab79|2026-11-24|docs: record the BC7 draw fixture on Intel Xe2
1f434220|2026-11-24|shader: lower a forward s_cbranch_scc0/scc1 skip over vector ALU in pixel_aot
d2b9bcd6|2026-11-24|docs: distinguish local slot 70 logs from report summaries
d1873624|2026-11-24|shader: lower gfx1030 2D image LOD queries
59ed0a5a|2026-11-24|cpu: lift SSE4a extrq, insertq, movntsd, and movntss leaf functions
81446f94|2026-11-24|Merge pull request #1419 from legacy-code-archaeologist/cpu/x87-ffree-empty
76de549a|2026-11-24|gfx2hlsl: lower per-lane bit operations (bfe_i32, bcnt, ffbl, ffbh, bfrev)
f9033a9b|2026-11-24|fix: publish the APU heap under one lock, keep the 16-byte alignment fallback, test a guaranteed no-fit
5bfafce2|2026-11-24|Merge pull request #1426 from Asimmimx/gpu/placed-dxr-distance-tolerance
910809bb|2026-11-24|Merge main and preserve both mesh validation paths
85f9dd83|2026-11-24|Merge pull request #1420 from tedin7/x2w-frontier-slot70-buf
75908557|2026-11-24|test: define each bit-op mutation independently and subtract the bcnt addend
772ac100|2026-11-24|shader: lower four gfx1030 pixel sample offset forms
12864bdb|2026-11-24|startup: fail page-destination requests whose page array is unreadable at run time
e8003928|2026-11-24|Hold 1 ms timer resolution while the desktop audio connection is active
795a18bc|2026-11-24|Merge pull request #1203 from lambirou/shader/mesh-as-forms
eba22f70|2026-11-24|docs: document R24G8 and added MSAA colour console plane sizing and its verification status
ea475f35|2026-11-24|Merge pull request #1440 from MasterDomino/shader/scc-skip
99624a8d|2026-11-24|fix: view typeless presented surfaces through a typed format
4dc61c82|2026-11-24|docs: keep the presenter note in experiments.md only
23a3533d|2026-11-24|gpu/tests: preserve typed formats above V408 and reject NaN readbacks (#1425)
e84829de|2026-11-24|Merge pull request #1421 from tedin7/x2w-frontier-acphal-apufree
d32b7fea|2026-11-24|startup: check that the last derived blend patch wins
014f2c47|2026-11-24|d3d12_xs: resolve DSVs from shadow records, refuse wrong-kind and mixed read-only views, keep DEPTH_READ when sampled, refuse DepthBoundsTestEnable; depth test emits adapter metadata and controls
bb82d0a6|2026-11-24|cpu: lift RDFSBASE, RDGSBASE, WRFSBASE, WRGSBASE, and RDPID in cpu_switch_aot
1a361114|2026-11-24|d3d12_xs: stencil-less depth formats have no second read-only plane
1c3842a6|2026-11-24|startup: check the logical view table bounds
5a71e682|2026-11-24|docs: state what the device run verified for the new plane-size formats
e7504862|2026-11-24|textures: check the BC7 4 KiB standard detile order
41c0852c|2026-11-24|Merge pull request #1425 from tedin7/x2w-frontier-present-typeless
22458aea|2026-11-24|shader: lower gfx1030 fixed-address atomic counters
5dfcb3c6|2026-11-24|shader: lower experimental gfx1030 packed D16 XYZ stores
c19508dd|2026-11-24|startup_stubs: filter GameInput history by device, keep extended scan codes, skip modifier aliases, separate handle lookups
fb2a4c08|2026-11-24|startup_stubs/audio_sink: add multi-channel downmix and linear resampling
054c159d|2026-11-24|startup_stubs: prefix navigation-cluster scan codes with E0
daf09b2a|2026-11-24|runtime: hand the executable a storage root that ends in a separator and is not cut off
3fcfd9ff|2026-11-24|docs: record the mesh draw fallback on Intel Xe2
bf512234|2026-11-24|d3d12_xs: support buffer and RT-AS memory SRVs on slot 70
d1dae4b4|2026-11-24|Merge pull request #1433 from ChazY11/docs/bvh-arc
bb6c9a0c|2026-11-24|Merge pull request #1438 from ChazY11/docs/bc7-draw-arc
5187d4dc|2026-11-24|d3d12_xs: fix namespace include and MSVC alignment in buffer SRV test
c4e6f4e6|2026-11-24|startup_stubs: stand in for B5G6R5 and array packed render targets
f6399bda|2026-11-24|docs: record RTX 4070 BVH retest
f998aad1|2026-11-24|docs: record the exact-mip MinLod target on a physical Intel adapter
4ec5dae1|2026-11-24|tests: fix DXGI_FORMAT_R32_TYPELESS enum value (39) in buffer SRV check
0968f732|2026-11-24|docs: record the compute-store readback on Intel Xe2
e1cb790c|2026-11-24|Merge pull request #1447 from Asimmimx/startup/derived-blend-check
f859099e|2026-11-24|Merge pull request #1401 from tedin7/x2w-c3-xrstor-compacted
1f07f846|2026-11-24|Merge pull request #1450 from Asimmimx/startup/logical-view-table-check
206dffcc|2026-11-24|Merge pull request #1442 from Ammmae0x/cpu/sse4a-aot
8f51708e|2026-11-24|Merge pull request #1446 from JB2WIN/shader/cvt-f64-f32
c44bd0ad|2026-11-24|tools/startup: add redacted 1-run telemetry summarizer (triage_run.py)
14dd94de|2026-11-24|Merge remote-tracking branch 'upstream/main' into cpu/switch-fsgsbase-rdpid
d27560cf|2026-11-24|tooling: report frame heartbeats without guessing draw status
50b872da|2026-11-24|Merge pull request #1444 from stan495/codex/gfx1030-mimg-msaa-load
767a0783|2026-11-24|docs: record the compute-produced mesh texture on Intel Xe2
1e788c6f|2026-11-24|tooling: add counted-loop generators to the CPU fuzz test
7e0e95c4|2026-11-24|Merge pull request #1443 from tedin7/x2w-frontier-gfx1-alu
c85b52ff|2026-11-24|Merge pull request #1448 from R0Y00/codex/audio-mixer-timer
76cc6a7c|2026-11-24|Merge pull request #1437 from tedin7/x2w-h-console-plane
93672b99|2026-11-24|Merge pull request #1415 from denn0x/startup/storage-token-reread
66677341|2026-11-24|startup_stubs/audio_sink: fold to mono before clamping and keep resampler state across blocks
310ad973|2026-11-24|Contain read-context allocation failures before XAsyncBegin
883bc706|2026-11-24|tests/startup_stream_depth: depth-only PSO has no pixel shader so zero-RTV pass raises no SV_Target0 warning
1df9604d|2026-11-24|Merge main and retain the root SRV draw evidence
d8ec13dd|2026-11-24|Merge pull request #1428 from tedin7/x2w-h-inflate-ex
3892513b|2026-11-24|shader: compare image LOD readback with direct HLSL
41c6e603|2026-11-24|tests/startup_stream_depth: depth-only PSO uses a pixel shader that exports nothing (no SV_Target0)
32d2c87e|2026-11-24|Merge pull request #1099 from R0Y00/codex/stream-root-srv-draw
569b5882|2026-11-24|gfx2hlsl: translate an export-less pixel shader as a void depth-only main
03d4c69d|2026-11-24|Merge pull request #1439 from SimSalabimse/x2w-gamesave-read-async
7d019d80|2026-11-24|Use typed element sizes for slot-70 buffer SRVs and refuse unsupported typed formats
06edf01a|2026-11-24|startup_stubs: keep per-kind GameInput sequence numbers and per-kind history rings
addc99c2|2026-11-24|tests/startup_stream_depth: request run ends after pipeline creation with the extra shader argument
d2c8c504|2026-11-24|runtime: fail a DirectStorage request on allocation failure without extra stack work
79270eb0|2026-11-24|runtime: roll back queue work if worker startup fails
fa7735b2|2026-11-24|runtime: defer queue failure effects to Submit and survive a throwing queue construction
3c158313|2026-11-24|runtime: fail every single allocation point of DirectStorage Enqueue and Submit without an exception
f3e07e49|2026-11-24|runtime: keep the DirectStorage queue entry free of allocating moves and trace the allocation sweep
f80e341b|2026-11-24|runtime: keep DirectStorage queue entries in order when an allocation fails during Enqueue
bf1b5b86|2026-11-24|triage_run: redact paths with spaces, de-duplicate mirrored stderr/trap logs, ignore successful forwarded slots
941510be|2026-11-24|tooling: show startup storage and render summaries
f5b7c737|2026-11-24|Merge pull request #1455 from ChazY11/docs/mesh-fallback-arc
9e056e3c|2026-11-24|tests: check previous keyboard sequence against the keyboard kind
399a56fc|2026-11-24|Merge pull request #1460 from ChazY11/docs/minlod-exact-arc
8adfac8a|2026-11-24|Merge pull request #1461 from SuperCraft3M/docs/rtx-4070-2114-bvh-report
06e2ff78|2026-11-24|Merge pull request #1467 from ChazY11/docs/compute-mesh-texture-arc
7ed364a1|2026-11-24|Split acceleration-structure SRVs out of slot 70; require structured stride multiple of 4
71ec5842|2026-11-24|triage_run: allowlist-only structured summary; never echo free-form log text
be3ebcf6|2026-11-24|docs: record the indexed mesh draw on Intel Xe2
8b2df3dc|2026-11-24|test: bound the gfx1030 upper raw LOD comparison
adf1d255|2026-11-24|tests: slot 70 refuses an acceleration-structure view and keeps the previous descriptor
e4454d0c|2026-11-24|docs: record the compute-produced palette draw on Intel Xe2
ae3dbdef|2026-11-24|Merge pull request #1463 from ChazY11/docs/compute-store-arc
b1d43427|2026-11-24|Preserve existing channel mappings and reset resampling on reopen
9c40d92e|2026-11-24|Merge pull request #1454 from JohnConnorNPC/runtime/startup-storage-path-separator
4ca3c951|2026-11-24|Merge main and preserve shader test registrations
ce36d731|2026-11-24|Merge pull request #1452 from Asimmimx/startup/bc7-4k-s-check
bd40dc96|2026-11-24|Merge pull request #1468 from roansystemlabs/test/446-fuzz-loops
204e11a1|2026-11-24|docs: record the XCL-08 compute list on Intel Xe2
af93500d|2026-11-24|Merge pull request #1451 from stan495/codex/gfx1030-mimg-sampling
eeeea64c|2026-11-24|Merge pull request #1459 from tedin7/x2w-frontier-packed-target
2ae8a9e4|2026-11-24|Merge pull request #1430 from tedin7/x2w-h-gpu1a-dsv
3f445536|2026-11-24|cpu: restrict --host-cpu in cpu_switch_aot to read-only RDFSBASE, RDGSBASE, and RDPID
3ed5928a|2026-11-24|runtime: verify local_log env_on, logf, and page probes without GPU
ecf042f5|2026-11-24|runtime: complete the failed DirectStorage suffix only after the worker finished the submitted prefix
d0b04e85|2026-11-24|startup: add storage fallback, stale cache warning and fast-fail diagnostics in launch.py
2842678d|2026-11-24|startup: resolve the cache check against the child cwd and keep it non-fatal
a8a5d992|2026-11-24|docs: record the gpu alias window run on Intel Xe2
d037cec0|2026-11-24|gfx2hlsl: lower 2D depth-comparison image sampling (image_sample_c, image_sample_c_lz)
01ec6db6|2026-11-24|Merge pull request #1445 from stan495/codex/gfx1030-mimg-get-lod
53497d10|2026-11-24|startup_stubs/xapu: track handles and retire permissive command completions
d953dc6a|2026-11-24|tests: count only buffer-sized worker allocations in the all-thread allocation sweep
3b858850|2026-11-24|Use stream position for filtered previous GameInput reading
8ae3c180|2026-11-24|gfx2hlsl: keep a sampler MAX_LOD limit on implicit depth comparisons
12a05a52|2026-11-24|Merge pull request #1110 from R0Y00/codex/cpu-srv-refresh
1b9f9088|2026-11-24|Merge pull request #1453 from tedin7/x2w-h-audio-sink
d043bd91|2026-11-24|startup: check the translated sampler bindings
f500398e|2026-11-24|tests: keep the suffix-order fences alive until the worker has signaled them
a8815ee6|2026-11-24|shader: lower gfx1030 image atomic arithmetic and bitwise operations
e1c8e922|2026-11-24|Merge pull request #1462 from tedin7/x2w-h-launch-diag
bbdd7a1a|2026-11-24|docs: mark xapu zero result as placeholder, note handle and memory behavior
717b617a|2026-11-24|Merge pull request #1436 from tedin7/x2w-h-gameinput
0400187a|2026-11-24|runtime: preflight Windows system commit headroom before PE launch
cec885b2|2026-11-24|Merge pull request #1474 from ChazY11/docs/compute-mesh-texture-plain-arc
6df679bf|2026-11-24|Merge pull request #1475 from ChazY11/docs/d3d12x-compute-list-arc
d34ee740|2026-11-24|shader: lower gfx1030 2D comparison sampling offsets
04686634|2026-11-24|startup: check the slice footprint plan and re-read rules
974b3888|2026-11-24|docs: record the virtual-memory device cases on Intel Xe2
d05eb21a|2026-11-24|startup: check the stand-in switch and trap-log contract
0b47a18a|2026-11-24|Bound exported slots and HRESULT encodings
d3ddcf0c|2026-11-24|docs: record the queue-ordered and raw view cases on Intel Xe2
5065b574|2026-11-24|Merge pull request #1472 from ChazY11/docs/mesh-indexed-draw-arc
08d689e0|2026-11-24|Merge pull request #1456 from tedin7/x2w-h-wall1b-srv
e706f731|2026-11-24|runtime: check the device creation IID inventory
c694231a|2026-11-24|Merge pull request #1464 from tedin7/x2w-h-triage-run
db7d106d|2026-11-24|startup_stubs/inflate: fix fixed_block static-init data race
b9b60f6b|2026-11-24|startup: keep shadow-only descriptor table entries in the capture summary
96bef2c1|2026-11-24|startup_stubs: guard window presenter surface and close state
ac0b5c8a|2026-11-24|runtime: answer store method 3 with an offline context
5b4e926f|2026-11-24|Merge pull request #1479 from ChazY11/docs/gpu-alias-window-arc
b6522d19|2026-11-24|startup: check the DirectStorage file read above 4 GiB
6b7a2033|2026-11-24|Merge pull request #1483 from ChazY11/docs/vm-translation-arc
3c6273ac|2026-11-24|Merge pull request #1484 from ChazY11/docs/vm-raw-xe2
b61a2359|2026-11-24|startup_stubs: refuse zero-extent present surfaces
7ba209aa|2026-11-24|startup: check the null binding cause labels
271af14e|2026-11-24|docs: record the array-atomics probe on Intel Arc Xe2
8a03ab30|2026-11-24|tests: remove the startup_stubs trap-log folder on exit
c0ade401|2026-11-24|Mock the non-Windows commit-budget test path
4da7bd7f|2026-11-24|fix: guard slice footprint byte spans and uploads against invalid indices
53bbb442|2026-11-24|runtime: check the per-adapter GPU usage total in either order
d03a76c4|2026-11-24|docs: complete the DXR command group for Intel Xe2
edcba82c|2026-11-24|docs: record descriptor heap paging on Intel Xe2
928d3fee|2026-11-24|Merge pull request #1449 from Ammmae0x/cpu/switch-fsgsbase-rdpid
2ba386bf|2026-11-24|Merge pull request #1486 from Asimmimx/startup/slice-footprints-check
b76c34ed|2026-11-24|Merge pull request #1485 from stan495/codex/gfx1030-mimg-atomic-rmw
3e23ce2c|2026-11-24|Merge pull request #1477 from Ammmae0x/runtime/startup-local-log-check
28cba308|2026-11-24|Merge pull request #1480 from Asimmimx/startup/translated-sampler-bindings-check
8a8e7e39|2026-11-24|docs: record the static-sampler PSO on Intel Xe2
9bb7e5ec|2026-11-24|Snapshot failed Queue::Submit entries at the cutoff and complete them on the worker behind the prefix
ca4a9830|2026-11-24|Merge pull request #1487 from stan495/codex/gfx1030-mimg-compare-sampling-upstream
1a1d58e2|2026-11-24|Merge pull request #1481 from Joacoorandom/runtime/host-commit-preflight
aa0e90b2|2026-11-24|Handle allocation failure before taking the XApu connection lock
1b7b2c06|2026-11-24|shader: lower gfx1030 image atomic integer min and max
2993c12c|2026-11-24|startup: check the placement publish lock and width
04d78f3d|2026-11-24|tests: constant-initialize the TLS value template on MSVC 19.42
d9e27f68|2026-11-24|gfx2hlsl: refuse unrestricted implicit image_sample_c without representable derivatives
ccd6fb04|2026-11-24|Merge pull request #1431 from Maxim-Root/shader/cmpx-f64
1fd9d20f|2026-11-24|Merge main and combine the local logging checks
bc43e665|2026-11-24|Merge pull request #1478 from tedin7/x2w-h-xapu-queue
6edd861d|2026-11-24|shader: lower gfx1030 explicit-gradient pixel sampling
0ae499c7|2026-11-24|startup: check the memory SRV report fields
4bbbbdc1|2026-11-24|ci: run the platform-independent Python checks in parallel
2f933a45|2026-11-24|﻿audio: report pacing error against the ideal cadence in the pacing control
7536843c|2026-11-24|shader: lower gfx1030 non-comparison 2D gathers
dbb1906f|2026-11-24|Merge pull request #1490 from tedin7/x2w-frontier-w3-capture
e2294b07|2026-11-24|Merge pull request #1496 from ChazY11/docs/gpu-alias-atomics-xe2
6a87f6a0|2026-11-24|runtime: refuse store method 3 for a non-local user
c99e4474|2026-11-24|Merge pull request #1498 from ChazY11/docs/dxr-five-ray-arc
69a3e0ef|2026-11-24|﻿audio: calibrate the steady pacing bound against scheduler jitter
62dc0124|2026-11-24|acphal: roll back partial shape-context allocations and expand tests
93ba3563|2026-11-24|Merge pull request #1500 from ChazY11/docs/descriptor-heap-xe2
7b0d7954|2026-11-24|Merge pull request #1501 from ChazY11/docs/pso-static-sampler-xe2
b86bceaa|2026-11-24|runtime: check the GPU range windows without a device
d4a99171|2026-11-24|docs: pair the Intel Xe2 atomics failure with the passing lookup read path
09b3e8ae|2026-11-24|docs: record a third behaviour for a non-visible heap GPU start
d21e3d6e|2026-11-24|gfx2hlsl: lower image_sample_c only with a MAX_LOD of zero; refuse implicit mip selection
4d8a0a0b|2026-11-24|Merge pull request #1505 from SimSalabimse/x2w-tls-value-msvc1942
04f94893|2026-11-24|Merge pull request #1504 from stan495/codex/gfx1030-mimg-atomic-minmax-upstream
29e6e94b|2026-11-24|Merge pull request #1491 from tedin7/x2w-h-w3-present
de3894f6|2026-11-24|Merge pull request #1488 from tedin7/x2w-h-w3-inflate
0b1f880b|2026-11-24|xcrdapi: complete deferred queue entries with one owner and no lock held across a fence
72f9fa62|2026-11-24|docs: record the descriptor window cases on Intel Xe2
cd186e9e|2026-11-24|Merge pull request #1495 from Asimmimx/tests/startup-stubs-temp-cleanup
e5af69c7|2026-11-24|Merge pull request #1493 from Asimmimx/startup/dstorage-file-read-check
ef39d41e|2026-11-24|Merge pull request #1489 from Asimmimx/runtime/device-slots-check
615ebe52|2026-11-24|Merge pull request #1497 from Asimmimx/startup/null-binding-cause-check
90fbe417|2026-11-24|Merge pull request #1499 from Asimmimx/runtime/xmem-gpu-usage-check
982e0303|2026-11-24|Reject overflowing footprint spans and subresource indices
e221849f|2026-11-24|shader: lower gfx1030 non-returning image exchanges
f1bf687a|2026-11-24|Merge pull request #1503 from Asimmimx/startup/placement-publish-check
d21fdf5a|2026-11-24|tests: tighten acphal negative controls and require release trap export
72e4e3e3|2026-11-24|Merge pull request #1506 from Asimmimx/startup/memory-srv-report-check
6fe5def5|2026-11-24|Merge pull request #1482 from Asimmimx/startup/local-log-check
191de955|2026-11-24|docs: record the d3d12_xs device creation on Intel Xe2
86f29e63|2026-11-24|startup_stubs: reject invalid ranges in publish_placement
309cfede|2026-11-24|Merge pull request #1457 from stan495/codex/gfx1030-mubuf-d16-xyz
0103dd53|2026-11-24|tests: cast read-only page pointer to void** for AcpHalCreate
60c72330|2026-11-24|Merge pull request #1458 from stan495/codex/gfx1030-mubuf-atomic-counters
136370be|2026-11-24|runtime: cap the permissive XMemMapPhysicalPages page count
9fd77e96|2026-11-24|startup: reject a writable range whose size wraps the address space
1ca0dec1|2026-11-24|Merge pull request #1494 from tedin7/x2w-frontier-w3-slice
9c38a5ed|2026-11-24|startup: make the save lock wait without overflow and name a failed open
f4d2a92f|2026-11-24|startup: guard min/max macros and negative-clock deadlines in file_lock
7d73370a|2026-11-24|era_gpu_alias: validate full committed span before calling D3D12
5f57a8d7|2026-11-24|era_gpu_alias: reject PAGE_NOACCESS and PAGE_GUARD regions in span walk
8045169e|2026-11-24|tooling: match scan shader stage tags as whole tokens
a8591c77|2026-11-24|tooling: report malformed PSO pipeline JSON as validation errors
3ff56553|2026-11-24|textures: report malformed --indices as a usage error in the batch exporter
08f102dd|2026-11-24|tooling: make the docs link check platform-independent
3626c7d1|2026-11-24|startup: make gfx2hlsl --capture terminate on wrapper cycles and refuse short descs
48a04876|2026-11-24|startup: run the debugger with the repository Python
f2c6524d|2026-11-24|Merge pull request #1512 from ChazY11/docs/gpu-alias-lookup-xe2
ff375c2f|2026-11-24|Merge pull request #1517 from ChazY11/docs/descriptor-window-xe2
d998e0ca|2026-11-24|Merge pull request #1521 from ChazY11/docs/d3d12x-device-create-xe2
d4f7a3ff|2026-11-24|startup: check the overlay window choice between two guests
b13a293b|2026-11-24|Separate observed non-visible heap handles from the D3D12 contract
cbb300e2|2026-11-24|runtime: check the PnP key length against the buffer capacity
efd0c471|2026-11-24|runtime: check the XMemVirtualQuery tail flag when it is clear
0e1ad704|2026-11-24|runtime: check that adjacent title ranges pack for the shader lookup
cbf54406|2026-11-24|gpu: check the raw UAV window validation and touching windows
dc24e35d|2026-11-24|Merge pull request #1471 from tedin7/x2w-frontier-gfx2-sample-c
a3a6c318|2026-11-24|startup: check the draw dump budget and removal fences
c07629d1|2026-11-24|Merge pull request #1514 from Asimmimx/runtime/gpu-range-check
399077a3|2026-11-24|Merge pull request #1522 from stan495/codex/gfx1030-mimg-atomic-exchange-upstream
9a586657|2026-11-24|Merge pull request #1507 from stan495/codex/gfx1030-mimg-gradients-upstream
8f438025|2026-11-24|Merge pull request #1513 from stan495/codex/gfx1030-mimg-gather-upstream
ca82d82e|2026-11-24|shader: lower the VOP3 (e64) v_cmpx region openers
d8af3c18|2026-11-24|Require the exact store provider GUID
64d451e3|2026-11-24|Merge main and retain the independent VEX parity differential
6e452960|2026-11-24|runtime: encode the persistent storage root as UTF-8
b50dfca3|2026-11-24|startup: check the packed stand-in tag round trip
e451c80d|2026-11-24|runtime: verify startup_stubs trap header switches and exit codes
be98528a|2026-11-24|startup: answer XG slot 10 for layout 0x10a from a 64 KiB tile plan
f9fc9800|2026-11-24|startup: check the title heap allocation that succeeds
f0379d4e|2026-11-24|textures: check which descriptions take the 64 KiB BC chain
329db8fa|2026-11-24|Merge pull request #1432 from archielcqp-a11y/gpu/wave64-emulate-folds
a8bd39b0|2026-11-24|Merge pull request #1516 from ChazY11/docs/descriptor-nonvisible-xe2
4bf6dd45|2026-11-24|Merge pull request #1470 from SimSalabimse/x2w-store-method-3
a3633d4d|2026-11-24|Merge pull request #1188 from Batushn/cpu/vex-pf-consumers
32fa3dcc|2026-11-24|startup: keep GameInput timestamps from wrapping after 21 days
47cc3f70|2026-11-24|tooling: do not report a partial-register xor as a constant xmem attribute word
c859515d|2026-11-24|runtime: reject a null blob name in XGameSaveReadBlobData
7f067c56|2026-11-24|ci: require every SPIR-V bridge WARP test in the result check
15e42bef|2026-11-24|Use p95 for steady and bursty pacing acceptance
b4fd34ff|2026-11-24|Merge main and preserve the full Python check list
af72b13d|2026-11-24|shader: lower gfx1030 2D comparison gathers
db40f92a|2026-11-24|shader: lower gfx1030 bounded image counters
255c328c|2026-11-24|docs: record Intel UHD coverage baseline
8c3696d6|2026-11-24|Merge pull request #1502 from tedin7/ci/parallel-python-checks
249a0719|2026-11-24|Merge pull request #1508 from triggered02/audio/pacing-error
b61eeb75|2026-11-24|shader: lower gfx1030 conditional atomic subtract with discarded result
863395d3|2026-11-24|Merge pull request #1509 from tedin7/x2w-h-w4-placed
edc59e54|2026-11-24|Hold the worker while a deferred completion is ready or owned
0565a910|2026-11-24|Merge pull request #1515 from tedin7/x2w-frontier-w4-storage
2b9536b1|2026-11-24|Merge pull request #1558 from RenzoWit35/startup/gameinput-timestamp-wrap
e83e225b|2026-11-24|Merge pull request #1559 from RenzoWit35/tooling/xmem-scan-partial-xor
bbd40029|2026-11-24|Define deferred_pending after Unrecorded
56ff9b20|2026-11-24|Allow the status entry before the held deferred fence to complete
b2691135|2026-11-24|Merge pull request #1561 from RenzoWit35/runtime/xgamesave-read-null-blob-name
6c0e5380|2026-11-24|Merge pull request #1560 from RenzoWit35/ci/spirv-warp-required-list
c3ff2308|2026-11-24|Merge pull request #1554 from JohnConnorNPC/runtime/xg-slot10-tile64-plan
6139aa46|2026-11-24|Merge pull request #1529 from RenzoWit35/tooling/scan-shader-stage-names
2efc4c3f|2026-11-24|Merge pull request #1531 from RenzoWit35/tooling/ttpf-export-indices-errors
9e7bed97|2026-11-24|Merge pull request #1532 from RenzoWit35/tooling/docs-links-portable-paths
6a77f06a|2026-11-24|Merge pull request #1538 from Asimmimx/tests/adapter-identity-check
800a4500|2026-11-24|Merge pull request #1518 from tedin7/x2w-h-p3-era-mem
f90f1c3c|2026-11-24|shader: lower v_cvt_i32_f64 and v_cvt_u32_f64 on proven pairs in pixel_aot
ba9ce578|2026-11-24|docs: record RX 6600 array-atomics readbacks
02fc3d04|2026-11-24|startup: print the reason when the logical view draw skips
987257ba|2026-11-24|Merge pull request #1530 from RenzoWit35/tooling/pso-readiness-input-validation
900c16b3|2026-11-24|Merge pull request #1539 from Asimmimx/tests/era-query-check
842e4886|2026-11-24|Merge pull request #1563 from JustinDCosta/docs/intel-uhd-coverage-baseline
a6f0602a|2026-11-24|Merge pull request #1552 from archielcqp-a11y/shader/vcmpx-e64
78907a8e|2026-11-24|Merge pull request #1562 from stan495/codex/gfx1030-mimg-compare-gather-upstream
22e0bba5|2026-11-24|Merge pull request #1524 from RenzoWit35/startup/local-log-probe-range
5042ab75|2026-11-24|Merge pull request #1525 from RenzoWit35/runtime/xmem-permissive-page-count
79f41db8|2026-11-24|tests: fix the DirectStorage file read check build on MSVC 19.42
b9a3aabb|2026-11-24|shader: lower compute v_xor3_b32
0fa48906|2026-11-24|shader: lower compute v_max3_u32 and v_min3_u32
0479b8de|2026-11-24|shader: lower gfx1030 uniform coarse-gradient sampling
b7b17d1f|2026-11-24|Merge main and reconcile storage-root documentation
e09b3302|2026-11-24|Merge pull request #1510 from tedin7/x2w-h-w3-acphal
9cea0b18|2026-11-24|shader: lower compute v_med3_u32 and v_med3_i32
be484d0f|2026-11-24|Merge pull request #1565 from stan495/codex/gfx1030-mimg-atomic-counters-upstream
2db1df38|2026-11-24|tooling: print one VGPR for the f64-source VOP1 conversions in disasm_rdna2
aec7d276|2026-11-24|startup: keep a character split between two log reads in the live view
a5633ab6|2026-11-24|shader: lower gfx1030 finite-normal image atomics
285cbc6c|2026-11-24|shader: lower compute s_bitreplicate_b64_b32 lane masks
906fc90a|2026-11-24|Merge remote-tracking branch 'origin/main' into startup/gfx2hlsl-capture-wrapper-cycle
dadee3b6|2026-11-24|tests: run repeated CLI checks in-process in resource_ranges_check
3f3047c1|2026-11-24|shader: lower compute s_absdiff_i32
6aa50eca|2026-11-24|Merge pull request #1551 from SuperCraft3M/runtime/storage-root-utf8
d76813a7|2026-11-24|startup_stubs: fix GameInput single-kind history step-back and init race
e7e1e928|2026-11-24|tooling: decode v_cvt_off_f32_i4, v_sat_pk_u8_i16 and the two swaps in disasm_rdna2
24322555|2026-11-24|Merge pull request #1540 from Asimmimx/tests/era-title-va-check
db0ea4b4|2026-11-24|Merge pull request #1548 from Asimmimx/tests/raw-buffer-uav-check
990c3593|2026-11-24|Merge pull request #1556 from Asimmimx/tests/title-heap-memory-check
328bd892|2026-11-24|Merge pull request #1557 from Asimmimx/tests/bc64-chain-check
2f0e3220|2026-11-24|docs: record the descriptor table and negative cases on Intel Xe2
eaf9b5ee|2026-11-24|Merge pull request #1570 from RenzoWit35/startup/logical-view-skip-reason
a600acaf|2026-11-24|Merge pull request #1528 from RenzoWit35/startup/gfx2hlsl-capture-wrapper-cycle
5239cb04|2026-11-24|docs: complete the vm_translation range cases on Intel Xe2
8bc460c5|2026-11-24|Merge pull request #1569 from RenzoWit35/docs/array-atomics-rx6600-readbacks
a3ac47a9|2026-11-24|Merge pull request #1564 from stan495/codex/gfx1030-mubuf-atomic-csub-upstream
2a59b44d|2026-11-24|docs: record the desktop memory queries on Windows 11 26300
8e57fe2d|2026-11-24|﻿ci: parse every Python file under tools/startup
9ac12472|2026-11-24|﻿ci: select the startup tools lint in the pull request preset
ab913560|2026-11-24|runtime: refuse forwarding the offline store context to desktop methods
5fad2caf|2026-11-24|test: inject worker creation failures (error 1455) and report crashes in the storage fixture
004609ac|2026-11-24|pe: read exports of an image without data directories as empty
4c15ea83|2026-11-24|script: report path replay failures without a traceback
b7352824|2026-11-24|startup: return the debugger's exit code from run.ps1
7c34e797|2026-11-24|Merge pull request #1576 from stan495/codex/gfx1030-mimg-coarse-gradient
47cc44bd|2026-11-24|docs: record the struct-return orderings on Intel Xe2
c5a13907|2026-11-24|Merge pull request #1573 from Asimmimx/shader/compute-xor3
d5987959|2026-11-24|Merge pull request #1589 from SimSalabimse/x2w-dstorage-c2664
88d49d51|2026-11-24|startup: report all slot70 refusal reasons and quote launch arguments
cdf90e15|2026-11-24|Merge pull request #1575 from JB2WIN/shader/cvt-int-f64
ded0fa0e|2026-11-24|Use Python source encoding rules in startup syntax check
cc3d4b83|2026-11-24|tooling: refuse bitcode arrays longer than the stream and runaway block nesting
e9ba20ed|2026-11-24|ci: cache pip/sccache and parallelize pixel-binary checks
3ca76c4f|2026-11-24|shader: lower gfx1013 float image stores and mip loads
ce996213|2026-11-24|tests: cache and parallelize form_coverage_check control runs
38cf3c00|2026-11-24|tests: fall back to serial controls when worker startup fails in submit
0f99e8b6|2026-11-24|Merge pull request #1578 from triggered02/ci/startup-tools-lint
dc1fd752|2026-11-24|Merge pull request #1600 from RenzoWit35/script/path-replay-clean-errors
afccd831|2026-11-24|shader: place compute entry registers from COMPUTE_PGM_RSRC2
18961289|2026-11-24|Merge remote-tracking branch 'origin/main' into docs/descriptor-table-xe2
ca678390|2026-11-24|Merge remote-tracking branch 'origin/main' into docs/vm-translation-range-xe2
af21c4bb|2026-11-24|Merge remote-tracking branch 'origin/main' into docs/desktop-memory-query-win11
1776e658|2026-11-24|Merge remote-tracking branch 'origin/main' into docs/device-struct-return-xe2
633b6181|2026-11-24|Merge pull request #1602 from RenzoWit35/pe/exports-without-data-directories
e92566a0|2026-11-24|Merge pull request #1603 from RenzoWit35/startup/run-ps1-exit-code
be147f17|2026-11-24|tests: run repeated CLI checks in-process in sgd5_effects_check
6eca98d7|2026-11-24|tooling: accept empty fragments on Markdown file links
81df4051|2026-11-24|Merge pull request #1593 from vladbondarenko97/tooling/disasm-vop1-narrow-dst
0be4f8a7|2026-11-24|Retain a texture descriptor process-level smoke check
3329b4cd|2026-11-24|Preserve the prior pad event for combined GameInput references
79226e74|2026-11-24|shader: lower compute quad DPP OR
7d4b477d|2026-11-24|cpu: report a named skip for the pe_map blast longjmp cases when CFG is active
62826689|2026-11-24|Merge pull request #1590 from ChazY11/docs/descriptor-table-xe2
3376850a|2026-11-24|tooling: decode 32-bit src0 for SOP1 s_bitset0_b64 and s_bitset1_b64 in disasm_rdna2
ed72e619|2026-11-24|Merge pull request #1595 from ChazY11/docs/vm-translation-range-xe2
abc89fca|2026-11-24|Merge pull request #1597 from ChazY11/docs/desktop-memory-query-win11
4aefce63|2026-11-24|Merge pull request #1601 from ChazY11/docs/device-struct-return-xe2
af31a689|2026-11-24|docs: record the gpu alias registry case on Intel Xe2
8650c725|2026-11-24|docs: record the offscreen check on Intel Xe2
8e0eb6cd|2026-11-24|gpu: compare dxr_placed_build with a direct-address DXR reference
24f16e5e|2026-11-24|Merge pull request #1535 from lambirou/startup/run-venv-python
ae21de70|2026-11-24|Merge main and preserve both storage compatibility notes
b4fa1c1b|2026-11-24|Merge pull request #1583 from tedin7/x2w-opt-devspeed-w2
71be6f0f|2026-11-24|Merge pull request #1581 from tedin7/x2w-h-w8-audio
4334bc13|2026-11-24|docs: record the descriptor heap limits on Intel Xe2
ec5ee282|2026-11-24|pe_map: reject reserved bits in ordinary named import thunks
047a3f19|2026-11-24|pe_run_package: report a package image that is not a PE as an error line
117ebb8b|2026-11-24|shader: lower a forward SCC if/else diamond over vector ALU in pixel_aot
8ca79203|2026-11-24|Merge pull request #1585 from tedin7/x2w-h-w8-mesh-ffx
26ede1d3|2026-11-24|docs: complete the descriptor paging run on Intel Xe2
56c438fb|2026-11-24|Merge pull request #1609 from nanyoudegege/tooling/accept-empty-doc-fragments
4ce5e761|2026-11-24|Merge pull request #1604 from tedin7/x2w-opt-devspeed-w3
f9002c4b|2026-11-24|Merge pull request #1605 from tedin7/x2w-frontier-w9-shadertools
d31605c7|2026-11-24|Merge pull request #1617 from ChazY11/docs/gpu-alias-registry-xe2
956c7b46|2026-11-24|Merge pull request #1618 from ChazY11/docs/offscreen-view-heap-xe2
5bfd55a4|2026-11-24|Merge pull request #1614 from MasterDomino/cpu/pe-map-cfg-longjmp-skip
767f43c1|2026-11-24|Merge pull request #1620 from ChazY11/docs/descriptor-heap-limits-xe2
60572183|2026-11-24|Merge pull request #1619 from MasterDomino/gpu/dxr-placed-direct-control
b9397c73|2026-11-24|Merge pull request #1410 from tedin7/x2w-frontier-1199-oom
7c5dac78|2026-11-24|Merge pull request #1599 from SuperCraft3M/runtime/offline-store-context-guard
aa87ce9a|2026-11-24|shader: restrict gfx1013 image stores to full DMASK
2bdd20db|2026-11-24|shader: refuse TG_SIZE_EN and test noncontiguous entry axes
d90dd76b|2026-11-24|Merge pull request #1594 from vladbondarenko97/tooling/disasm-vop1-cvt-sat-swap
9c40e65f|2026-11-24|Translate, bind and upload one-dimensional textures in startup draws
158626c9|2026-11-24|Merge main and retain the XOR3 and min/max3 forms
ffce6d1e|2026-11-24|cpu: refuse a PE with no runtime-function directory cleanly
c8147541|2026-11-24|cpu: report EXE translation failures as a message, not a traceback
b1669d91|2026-11-24|docs: add the RTX 5080 view-heap limit to the startup section
17b28a03|2026-11-24|docs: record which adapter needs the null-view fallback
9d931069|2026-11-24|tests: run repeated CLI checks in-process in layout_chunks_check
0a1a01ad|2026-11-24|docs: record the slot 17 queue-wait run
7b774833|2026-11-24|docs: record the descriptor record copy cases on Intel Xe2
845eddbe|2026-11-24|tests: run repeated CLI checks in-process in dxil_retarget_check
141617bb|2026-11-24|startup: hand off a packed screen-position gradient fixture (#1634)
5c599b99|2026-11-24|shader: index compute DPP quads by group thread
4147ef23|2026-11-24|gpu: add a composite provenance fixture for retained clear colours
2a97c577|2026-11-24|Merge pull request #1520 from tedin7/x2w-opt-ci-cache
cb94b8ca|2026-11-24|Merge pull request #1527 from tedin7/x2w-opt-form-coverage
eb98d9b4|2026-11-24|Merge branch 'main' of https://github.com/R0Y00/Xbox2Windows into shader/compute-entry-layout
cdebca53|2026-11-24|Merge branch 'main' of https://github.com/R0Y00/Xbox2Windows into shader/compute-image-store
126ce023|2026-11-24|startup: recheck single-mip A8 atlases every frame
2469f688|2026-11-24|Check the explicit-LOD refusal for 1D textures
734217e3|2026-11-24|Merge pull request #1574 from Asimmimx/shader/compute-max3-min3
d673eb38|2026-11-24|Merge pull request #1626 from ChazY11/docs/descriptor-paging-swapped-xe2
0aa6522c|2026-11-24|Merge pull request #1632 from ChazY11/docs/view-heap-limit-rtx5080
94c5fd3f|2026-11-24|shader: pin the COMPUTE_PGM_RSRC2 field widths
457cc779|2026-11-24|shader: pin gfx1013 image instruction and SRV binding refusals
7355cc6e|2026-11-24|Merge main and register SCC diamond controls in both test modes
8a068d50|2026-11-24|Merge pull request #1630 from R0Y00/codex/startup-texture1d-on-windows
e200cbaf|2026-11-24|Merge pull request #1636 from ChazY11/docs/queue-wait-slot17
dd76fd1c|2026-11-24|Merge pull request #1637 from ChazY11/docs/descriptor-record-copy-xe2
b976cf5e|2026-11-24|Merge pull request #1642 from MasterDomino/gpu/composite-provenance
874cc3fa|2026-11-24|Merge pull request #1625 from MasterDomino/shader/scc-diamond
e3360c65|2026-11-24|cpu: lower UD1 to the host exception it raises; keep UD0 refused
35ea852e|2026-11-24|startup: start the debugged process without the debug heap
6eb74ba4|2026-11-25|Merge pull request #1615 from tedin7/x2w-frontier-w10-pemap
e2c24edd|2026-11-25|Merge pull request #1633 from tedin7/x2w-frontier-w11-cpuaot
d7c0ac63|2026-11-25|Merge pull request #1627 from tedin7/x2w-opt-devspeed-w4
bb80aa6e|2026-11-25|Merge pull request #1640 from tedin7/x2w-opt-devspeed-w5
6177df41|2026-11-24|compute: lower buffer_store_format_* from the descriptor format
20d2ef06|2026-11-25|Merge main and preserve the MED3, MIN3, MAX3 and XOR3 forms
e484e62d|2026-11-25|Merge pull request #1608 from Jess-Doit/shader/compute-entry-layout
0ccde933|2026-11-25|Merge pull request #1571 from tedin7/x2w-frontier-w6-liveview
9c1d24a6|2026-11-25|Merge pull request #1537 from Asimmimx/tests/overlay-window-check
405ee00d|2026-11-25|Merge pull request #1645 from rm-NoobInCoding/startup/a8-atlas-recheck
2f177cc3|2026-11-25|Merge pull request #1577 from Asimmimx/shader/compute-med3
578d1e1c|2026-11-25|Merge pull request #1635 from ChazY11/docs/null-views-fallback-split
ec7bfff0|2026-11-25|Merge main and preserve the image binding and RSRC2 CLI options
852ab34b|2026-11-24|startup: count BCPack requests by format
315e6d5e|2026-11-24|startup: record root CBV addresses for translated draws (slot 42)
df488f52|2026-11-25|Merge pull request #1611 from Jess-Doit/shader/compute-image-store
16c9abd6|2026-11-25|Merge pull request #1650 from rm-NoobInCoding/startup/no-debug-heap
c259252e|2026-11-24|﻿compute: accept a scratch-base register write that nothing can observe
f0c0ed86|2026-11-25|Merge pull request #1553 from Asimmimx/tests/packed-target-format-check
d6624c0e|2026-11-24|shader: lower v_cvt_f32_f64 for exact static pairs in pixel_aot
ed874cfe|2026-11-25|Merge pull request #1550 from Asimmimx/tests/draw-dump-tracking-check
109e7dc8|2026-11-24|startup: clear stale root CBV address on null or unreadable bind
743a8b4c|2026-11-24|tests: add a disp32 UD1 memory form to the trap fixture
4b84235f|2026-11-25|Merge pull request #1622 from lambirou/startup/bcpack-format-census
d383358b|2026-11-25|Merge main and document logical DPP quads
7ff2fbc1|2026-11-25|Merge upstream main into gfx1030 finite-normal image atomics
87e48ad7|2026-11-24|shader: lower v_sad_u8 and v_sad_u16
ffb70f6a|2026-11-24|fix: default viewport and scissor for translated draws
1a6e8cc3|2026-11-24|startup: refresh a frame copy when the title rewrites its memory
30ef44f3|2026-11-24|﻿compute: accept the LLVM float spellings of the scratch-base immediate
c864f138|2026-11-25|Merge pull request #1613 from JustinDCosta/compute/quad-perm-or
e6d94835|2026-11-24|﻿tests: cover the accepted scratch-base write positively in every mode
50aaa702|2026-11-24|test: avoid the Windows small macro in the viewport case
9e4488cd|2026-11-24|docs: link 8 GPU coverage reports from #1
49d15fc2|2026-11-24|Document startup root CBV fixture in experiments
99f097ad|2026-11-24|startup: open the trap log once per module
1f24626b|2026-11-24|MUTATION (temporary): disable default viewport
4bbfee25|2026-11-24|tooling: report a missing fxc instead of publishing a shader manifest without bytecode
a19dc4a3|2026-11-24|startup: check that refused storage requests leave the destination and later requests intact
44fc72c4|2026-11-24|docs: add the RX 7700 XT to the hardware coverage table
22989d61|2026-11-24|Revert "MUTATION (temporary): disable default viewport"
1ca92d57|2026-11-24|tooling: decode the VOP1 rounding, frexp and operand-free opcodes in disasm_rdna2
6ee522e4|2026-11-24|Merge origin/main into x2w-compute-store-format
9b49e548|2026-11-24|startup: cache placed-buffer allocation extents (slot 133)
0647a981|2026-11-25|docs: run the startup experiments under Wine and Proton on Linux
61a55f63|2026-11-25|Invalidate recorded root addresses on root signature changes
d7128cdd|2026-11-25|docs: record which mip the Arc reads at the MinLod boundary
b894acb1|2026-11-24|Merge pull request #1659 from Makto94/docs/link-hardware-reports
d53d2386|2026-11-25|Merge pull request #1654 from tedin7/x2w-frontier-b2-rootcbv
79899b7d|2026-11-24|startup: refuse a storage request whose destination is not writable to its capacity
78fc5c08|2026-11-24|tests: expect the scanner to accept v_sad_u8 and v_sad_u16
2751cbd0|2026-11-25|Merge pull request #1655 from triggered02/compute/flat-scr-write
abb24b67|2026-11-24|Merge pull request #1666 from Makto94/docs/rx7700xt-row
50cd06c8|2026-11-24|tooling: mark the PATH stub executable in the fxc discovery check
950c0578|2026-11-24|tests: tighten exit code masking assertions and terminate child on timeout in startup_trap_header_check
f121513e|2026-11-24|Merge pull request #1671 from ChazY11/docs/minlod-fractional-arc
07f2464f|2026-11-24|Regenerate the default viewport and scissor when render target 0 changes
e4e47add|2026-11-24|Merge pull request #1668 from JohnConnorNPC/docs/wine-linux
f674edfd|2026-11-25|Merge pull request #1664 from RenzoWit35/startup/storage-queue-refusal-order
cd4ea26a|2026-11-24|startup: check every field of a buffer memory SRV
04711708|2026-11-24|startup: translate the SMEM offset 'off' as the literal 0 in gfx2hlsl
1f5d0bac|2026-11-24|startup: check the plane size families and shape bounds
4bbfec62|2026-11-24|docs: link 4 more GPU coverage reports from #1
8bf16dff|2026-11-24|shader: lower compute v_mad_i32_i24
d03f2264|2026-11-24|shader: lower compute v_cvt_pk_u8_f32
3e895d23|2026-11-24|test: keep the small target alive until the grow list has executed
6c03c3c9|2026-11-25|test: cover leading and trailing spaces in save names
f2cb67f8|2026-11-25|Merge pull request #1662 from RenzoWit35/tooling/gfx2hlsl-missing-fxc
a1455727|2026-11-24|﻿cpu: lower the reserved/hint NOP space in the shared lowering
a72a0929|2026-11-24|﻿cpu: compare native execution against the lowered body for the hint NOPs
d5ca939c|2026-11-24|﻿docs: record the reserved hint NOP support and how to reproduce it
7368c1b6|2026-11-24|﻿cpu: select the reserved NOP encodings by iced code, not by mnemonic
ab820b47|2026-11-24|﻿tests, docs: pin the operand sizes and keep the doc diff to this change
80f413a7|2026-11-24|Merge pull request #1678 from Makto94/docs/link-hardware-reports-2
4fb81a5f|2026-11-24|﻿tests: run the prefixed and 16/64-bit hint NOPs natively
9ebcb5e1|2026-11-24|cpu: select a host-disabled XCOMP_BV bit for native compacted XRSTOR #GP checks
19c19eb6|2026-11-24|Merge origin/main into x2w-compute-store-format
3c684a32|2026-11-24|startup: check the cb_blend decoder without a device
1a4557ce|2026-11-24|tests: cover PAGE_WRITECOPY destination in startup_dstorage_destination_check
a9852263|2026-11-24|startup: check that a reused or destroyed save lock frees its lock
80f48e3d|2026-11-24|shader: accept more result-neutral SOPP hints like s_nop 0
90793ffa|2026-11-25|Merge pull request #1676 from Asimmimx/startup/gfx2hlsl-smem-off
daa65230|2026-11-25|Merge pull request #1657 from tedin7/x2w-frontier-p04-viewport
d81928db|2026-11-25|test: check the bc64 chain offsets and address interleave
69fd3cb3|2026-11-24|shader: lower v_sat_pk_u8_i16 in DPP16 and DPP8
e01e5bd4|2026-11-25|Keep translated frame uploads usable without a CPU snapshot
7fdfe6d2|2026-11-25|Merge pull request #1672 from tedin7/x2w-frontier-xcrd-writable
0a0dbf82|2026-11-24|tests: verify module overflow masking with bit 19 in startup_trap_header_check
18a2942a|2026-11-25|Merge pull request #1677 from Asimmimx/tests/memory-srv-buffer-check
f2b10057|2026-11-25|tests: use an encodable scratch-store refusal control
c525af74|2026-11-25|Merge pull request #1679 from Asimmimx/tests/plane-size-check
685dcc3a|2026-11-25|Merge pull request #1652 from legacy-code-archaeologist/cpu/lower-ud1
2a3dd3f1|2026-11-24|Merge pull request #1682 from ChazY11/test/save-name-space-cases
da4bf085|2026-11-25|Merge pull request #1658 from tedin7/x2w-frontier-p07-framecopy
c327903b|2026-11-25|Merge pull request #1547 from triggered02/cpu/hint-nop
bc65bb1b|2026-11-24|Merge pull request #1367 from Maxim-Root/cpu/load-config-cfg-policy
4d0a436e|2026-11-25|Merge pull request #1692 from stan495/codex/compute-scratch-store-control
1e2bbb5b|2026-11-24|startup: run every gfx2hlsl vertex-fetch buffer format
f9bbb4cb|2026-11-24|tools: count Device::slot133 placements by outcome instead of as unhandled slots
7a5650b3|2026-11-24|Merge remote-tracking branch 'origin/main' into startup/trap-log-single-open
df97a04b|2026-11-24|tests: keep the slot133 check clear of neighbouring test hunks
7955a4f1|2026-11-24|startup: let the kept trap log be deleted while it is open
cb6f71d4|2026-11-24|Merge pull request #1687 from Asimmimx/tests/cb-blend-check
a4a0b1c7|2026-11-24|shader: allow several samples of one bound texture in pixel_aot
c3fa1e7d|2026-11-25|Merge pull request #1691 from Asimmimx/tests/file-lock-release-check
f5fbc214|2026-11-25|Merge pull request #1690 from ChazY11/test/bc64-chain-arithmetic
409b6f2c|2026-11-25|Merge pull request #1686 from R0D4X/fix/issue-1647-xsave-xcr0-bit
99e2e324|2026-11-24|gpu: check the raw SRV bounds and 64-bit offsets without a device
37608bfc|2026-11-24|runtime: check the struct-return adapter's refusals and heap type
4b6d887e|2026-11-25|test: cover the save-name rules that no case exercised
68969ed8|2026-11-25|Merge pull request #1598 from tedin7/x2w-frontier-w8-tools
ceccbcae|2026-11-24|startup: accept output modifiers on float-result conversions
5338a26c|2026-11-24|startup: check the draw-dump BMP writer's limits and header fields
9d47aec5|2026-11-24|startup: draw the gfx2hlsl conversion operations
62758977|2026-11-24|runtime: check the CPU-SRV refresh refusals without a device
a8a84c39|2026-11-24|startup: lower image_sample_c and image_sample_c_lz on a 2D array
6dbe718d|2026-11-24|shader: drop the redundant no-binding clause from the sample check
4ab6a47a|2026-11-24|Merge pull request #1596 from Asimmimx/shader/compute-mad-i32-i24
86ffc079|2026-11-24|Merge pull request #1656 from rm-NoobInCoding/shader/sad-u8-u16
bfe9158d|2026-11-24|Merge pull request #1660 from JB2WIN/shader/cvt-f32-f64
d301f498|2026-11-25|docs: record a second adapter on the null-view fallback
d7346d7e|2026-11-24|startup: apply the output modifier before the clamp
17866fd4|2026-11-24|startup: refuse a binary32 vector operation with the wrong operand count
e4ac2c78|2026-11-25|Merge pull request #1696 from Asimmimx/startup/vertex-fetch-formats-check
eb913fc7|2026-11-24|runtime: check the RFC 1950 codec's refusals
e048bd08|2026-11-24|Merge pull request #1661 from vladbondarenko97/tooling/disasm-vop1-gap-blocks
b6e525f0|2026-11-24|Merge pull request #1704 from ChazY11/test/save-name-rules
107a0466|2026-11-24|Merge pull request #1688 from rm-NoobInCoding/shader/timing-hints
1715ef09|2026-11-24|docs,tests: drop stale 2D-array not-covered note and format fixture dict
fd4358fd|2026-11-25|docs: give the null-view command a directory that exists
3d0618a2|2026-11-24|startup: flip the sign bit for a negated source on the vector select
b019275d|2026-11-24|startup: refuse an ambiguous negated integer constant on the select
41839f26|2026-11-24|Merge pull request #1592 from Asimmimx/shader/compute-cvt-pk-u8-f32
36ace502|2026-11-24|Merge pull request #1702 from henriazz10/shader/pixel-aot-multitap
669282ec|2026-11-24|startup: check the API trace's declared contracts and second-output reports
515d6a35|2026-11-25|Merge pull request #1697 from tedin7/x2w-frontier-triage-slot133
cea8af5b|2026-11-25|Merge pull request #1713 from Asimmimx/tests/raw-buffer-srv-check
7fb32427|2026-11-24|runtime: check era_gpu_alias refusals without a GPU
eaa2024b|2026-11-25|Merge pull request #1714 from Asimmimx/tests/cpu-srv-upload-check
53cee64c|2026-11-25|docs: run the two native CPU blocks in the documented build tree
deeef352|2026-11-24|Merge pull request #1694 from rm-NoobInCoding/shader/dpp-sat-pk-u8
1b0e6148|2026-11-25|Merge pull request #1621 from tedin7/x2w-h-w11-pemap-thunk
f76093b6|2026-11-24|Merge pull request #1715 from ChazY11/docs/null-view-second-adapter
a1f2625d|2026-11-25|Merge pull request #1716 from Asimmimx/startup/gfx2hlsl-vec-f-operand-count
ad410f54|2026-11-24|startup: check the 64 KiB tile plan without a device
dfc73a1b|2026-11-25|Preserve unmodified negative select immediates
7941c488|2026-11-24|Merge pull request #1587 from stan495/codex/gfx1030-mimg-float-atomics-main
c7ea6059|2026-11-24|runtime: check era_physical page ownership
49634a12|2026-11-25|Merge pull request #1707 from triggered02/startup/vcvt-omod
b877efa6|2026-11-24|Merge pull request #1720 from ChazY11/docs/native-cpu-build-tree
0ceece32|2026-11-24|startup: defer heap-indexed root signatures and refuse real draws under them
ed9689bc|2026-11-25|test: refuse a GPU virtual address floor of 16 bits
65e758b0|2026-11-25|Merge pull request #1588 from Asimmimx/shader/compute-absdiff
e6a0cd79|2026-11-24|startup: translate the SDWA forms of the float-result conversions again
ab586947|2026-11-25|Merge main and retain both startup shader checks
50625c8c|2026-11-25|test: check the range's QueryInterface contract
9ffb8529|2026-11-24|Merge pull request #1582 from Asimmimx/shader/compute-bitreplicate
321ba502|2026-11-24|startup: check the null Texture2D view fallback without a GPU
30c332cb|2026-11-24|startup: decode short Huffman codes in inflate.h with a lookup table
a4d17cf5|2026-11-25|Merge pull request #1730 from triggered02/startup/sdwa-cvt
4baa9002|2026-11-25|test: cover a BC7 layout whose padded extent is not square
27125e43|2026-11-24|startup: drop bound descriptor tables on a root signature change and log dropped root constants
7e9af999|2026-11-24|startup: keep the root signature reset and its notes clear of the open root signature changes
360bbf31|2026-11-25|test: refuse a BC7 copy from an unreadable source
9bed7ffc|2026-11-24|startup: refuse output modifiers on the vector select and cover the immediate range edges
11f3bb65|2026-11-25|test: refuse a null layout destination
a992dbaa|2026-11-24|runtime: check every field of an ERA attribute word
48d9571d|2026-11-25|Merge pull request #1732 from MasterDomino/startup/root-heap-indexed
909ec206|2026-11-25|Merge pull request #1718 from Asimmimx/tests/d3d12x-struct-return-check
312f5613|2026-11-25|Merge pull request #1717 from Asimmimx/tests/draw-bmp-check
8a07e812|2026-11-24|build: link the dev preset with fast debug info and compile the device file once for two test targets
1f9aeec7|2026-11-24|docs: record the gfx2hlsl bit-operation draws on WARP and an RX 6600
0f3dd062|2026-11-25|Merge pull request #1727 from ChazY11/test/range-va-bits-floor
f58d2425|2026-11-25|Merge pull request #1731 from ChazY11/test/range-query-interface
74811186|2026-11-24|startup: refuse an op_sel the float lowering does not model
5550ec2b|2026-11-24|startup: list save containers and blobs under the shared publication lock
229f7b13|2026-11-24|tests: bind placed-allocation-cache controls to their stated reason
171c7213|2026-11-24|fix: bind single-range render targets, resolve copied views in clears, count unresolved handles
f7b7297d|2026-11-24|shader: preserve non-fused gfx1013 multiply-add rounding
a1011264|2026-11-24|shader: lower s_setreg_b32 and s_getreg_b32 for the MODE rounding field in pixel_aot
110f18c4|2026-11-24|test: list the extra containers out of order so an unsorted listing fails
89dc7fe9|2026-11-24|docs: place the render target section away from the file tail
dc89ee60|2026-11-24|fix: guard single-range handle overflow, avoid stale RTV fallback, sync totals label
ad0deab4|2026-11-24|test: reset command list before recording in startup_rtv_bind
82b1bf94|2026-11-24|test: use command list slot 139 for Reset in startup_rtv_bind
d81c36ef|2026-11-24|docs: record WARP verification results for startup_rtv_bind
9f20c573|2026-11-24|fix: reject unmaterialized shadow RTVs/DSVs on clear and bind
cada2606|2026-11-25|startup: preserve explicit pixel depth exports
f6ea28fb|2026-11-25|tooling: align audio packet levels with input timestamps
f9e815d5|2026-11-24|Merge remote-tracking branch 'origin/main' into startup/cndmask-nits
25fbf27b|2026-11-24|Merge branch 'main' of https://github.com/R0Y00/Xbox2Windows into startup/gfx2hlsl-nonfused-mad
771e8de2|2026-11-25|docs: measure early window and movie milestones under the ordinary heap
50df4c2d|2026-11-24|tests: create an alternate root signature directly in startup_root_reset
8373ae8f|2026-11-25|test: cover a descriptor key from a root parameter
6604fab7|2026-11-25|gpu: recover startup command lists after failed Close
a511dbb0|2026-11-25|test: refuse a numeric read of a descriptor register
d1daca3a|2026-11-24|startup: pass the modifiers into the comparison path instead of dropping them
787f284f|2026-11-25|Merge pull request #1710 from triggered02/startup/cndmask-neg
c7fa63c4|2026-11-24|startup: copy stored blocks in inflate.h with one memcpy
a5997c1e|2026-11-25|test: pin the two sampler-descriptor refusals
6f2ee280|2026-11-25|Merge pull request #1762 from R0Y00/codex/synthetic-depth-export-control
c0e5059c|2026-11-24|startup: copy inflate.h matches 8 bytes at a time
4c4ea4aa|2026-11-25|startup: exercise upload arena allocation contract
383e567b|2026-11-24|docs: link the RX 9060 XT 32.0.32015.16001 report from #1
1b9e8284|2026-11-24|shader: lower the 46 non-cmpx SDWA comparisons
8f333a8a|2026-11-24|tests: pin the absolute-value and negated-mask refusals on the select
e6ac2d05|2026-11-24|runtime: hold the unmapped window for the reader-without-retry control
fc2f1ebb|2026-11-24|Merge pull request #1748 from ChazY11/test/bc7-layout-null-output
587688e9|2026-11-24|startup: decode inflate.h symbols from a 64-bit buffer away from the ends
c197c188|2026-11-24|shader: translate mesh allocations with several vertices and triangles
bfea5baa|2026-11-24|Merge pull request #1746 from ChazY11/test/bc7-copy-unreadable-source
a8f581c4|2026-11-24|startup: translate the fetch call of stream vertex shaders from the input layout
67319a17|2026-11-24|Merge pull request #1742 from ChazY11/test/bc7-layout-nonsquare
0ad04b91|2026-11-24|startup: count failed DirectStorage requests by cause in the heartbeat
f0202ea5|2026-11-24|startup: check only fetched layout elements; keep translating with an unreadable layout
af162999|2026-11-24|Merge origin/main into startup/stream-input-layout-fetch
65660899|2026-11-24|shader: lower compute v_mbcnt_lo/hi_u32_b32 with a uniform mask
58a796ce|2026-11-25|test: pin two integer-op refusals in the translator
cc0712a4|2026-11-24|startup: check the inflate.h distance bound in the fast loop
58de1589|2026-11-24|Merge pull request #1737 from Asimmimx/tests/null-texture-view-check
b4c961cd|2026-11-24|Merge pull request #1775 from Makto94/docs/link-rx9060xt-32015
9ce0edb8|2026-11-24|Merge pull request #1777 from JB2WIN/shader/sopk-setreg
8bbf8206|2026-11-24|startup: name the command list slots the noop log can state (#870)
ccba9fcf|2026-11-24|startup: re-check container state under the lock in list_blobs
f3649ec0|2026-11-25|Merge pull request #1701 from tedin7/x2w-frontier-p01-rtv
b9f355f1|2026-11-25|Merge pull request #1770 from R0Y00/codex/recover-failed-command-list
4e47decc|2026-11-24|startup: include the slot names header with the sorted local headers
8c77885b|2026-11-25|Use one failure sample for both storage heartbeat lines
c23e3ffe|2026-11-25|Merge pull request #1765 from R0Y00/codex/startup-heap-timing-notes
4e732d53|2026-11-25|Merge pull request #1781 from MasterDomino/startup/storage-failure-counters
5b294ca9|2026-11-24|Merge pull request #1780 from ChazY11/test/gfx2hlsl-op-refusals
a9b2ecf6|2026-11-24|Merge pull request #1782 from henriazz10/shader/compute-mbcnt
7a1acb7b|2026-11-25|Keep conversion draw controls alongside the current startup checks
96a02243|2026-11-24|Merge pull request #1738 from Asimmimx/tests/tile64-plan-check
ed12196d|2026-11-25|startup: lower the binary32 class comparison
3dc78117|2026-11-25|Merge pull request #1768 from Jess-Doit/startup/gfx2hlsl-nonfused-mad
3c924362|2026-11-24|Merge pull request #1776 from triggered02/startup/cndmask-controls
330df7f0|2026-11-25|Merge pull request #1744 from tedin7/x2w-frontier-p08-rootsig
40f7a3ba|2026-11-24|Merge pull request #1773 from ChazY11/test/sampler-descriptor-refusals
1c3777c0|2026-11-24|startup: pass OMSetStencilRef (slot 24) to the desktop list
6efdd359|2026-11-24|startup: stop the slot 133 walk at the buffer end instead of caching extents
41f46c4b|2026-11-24|Merge pull request #1751 from Asimmimx/runtime/era-heap-decode-fields
d4fb8113|2026-11-24|test: verify GPU-written indirect dispatch arguments
ecc6e6ce|2026-11-25|Merge pull request #1708 from Asimmimx/startup/gfx2hlsl-cvt-alu-check
c3649d4b|2026-11-24|runtime: keep the physical-page slab bookkeeping in a portable header and report slab counts
dca7a4ac|2026-11-24|docs,tests: include xmem_slab_slots_check in build snippet and cover slot-0 unwind
439333ac|2026-11-24|tests: draw 2D-array depth comparison with slice controls
05bce5c0|2026-11-25|startup: fix the class-compare operand order, exec mask and e32 form
d7ee9475|2026-11-24|startup: check inflate.h's fast_bits range and byte order at compile time
350619f0|2026-11-25|Keep upload arena checks alongside startup runtime regressions
574f65fa|2026-11-24|shader: accept v_clrexcp and v_pipeflush as no-ops in pixel_aot
5d3b3c85|2026-11-24|startup: keep the byte-wise stored copy in the inflate.h reference build
9170a4e1|2026-11-24|startup: end the inflate probe's destination at a no-access page
49eea1e7|2026-11-24|startup: invalidate cached constants when the graphics root changes
91fb38a4|2026-11-24|shader: take MIN3 of the operands when a v_med3_f32 operand is NaN
757cea99|2026-11-24|tests: report the failed load in the relocation-free control instead of a TypeError
4b48f09b|2026-11-24|startup: add the inflate fast loop's input margin and distance controls
6f224486|2026-11-24|startup: limit the inflate boundary case's stored-block exception
f48a9a8b|2026-11-25|test: pin the gather offset refusal and its signed decode
1ce038f8|2026-11-24|startup: lower combined idxen+offen and SGPR soffset buffer loads in gfx2hlsl
fe375f43|2026-11-24|startup: pin the single 1-bit distance code in the inflate check
e709d716|2026-11-24|startup: measure inflate.h throughput with startup_inflate_check.py --bench
971a9eb2|2026-11-25|Merge pull request #1774 from R0Y00/codex/command-list-upload-arena
5ddcdf8d|2026-11-25|Merge pull request #1788 from tedin7/x2w-frontier-slot-names
2a8ed34e|2026-11-25|Merge pull request #1761 from tedin7/x2w-frontier-1334a-enum
813045b3|2026-11-25|Merge pull request #1763 from R0Y00/codex/align-audio-input-timeline
d819085a|2026-11-25|startup: lower the 64-bit vector shifts
a95e7f19|2026-11-24|Merge startup/inflate-table-fast-path (fe375f43) into startup/inflate-second-pass
eb78b95e|2026-11-24|Merge origin/main into startup/stream-input-layout-fetch
ddeaa5eb|2026-11-25|Merge pull request #1711 from tedin7/x2w-frontier-gfx-samplec2da
90d39cad|2026-11-24|gpu: make DXR opacity TMin configurable
25b0b650|2026-11-24|startup: source-built NGG identity and contrast cases with a reference model
c7db80b0|2026-11-25|Match the stencil fixture optimized clear to each case
4004c33b|2026-11-25|Keep indirect dispatch and upload arena fixtures registered
5d0dce30|2026-11-25|startup: lower a typed buffer load of BUF_FMT_16_16_16_16_FLOAT (#1699)
dae82379|2026-11-24|test: pin commit headroom rounding and request cap in host_commit_check
f7226c22|2026-11-24|test: pin rectlist placement and winding refusal in rectlist_topology check
6b69c720|2026-11-25|Merge pull request #1787 from tedin7/x2w-frontier-slot24-stencil
26943da1|2026-11-24|test: pin padding, tight-length and zero-depth controls in texture_bc_volume_check
9b5ec25b|2026-11-24|test: pin failure reporting in run_checks_parallel_check
5a6bb608|2026-11-25|Merge pull request #1805 from SuperCraft3M/test/gpu-produced-indirect-dispatch
64869e8e|2026-11-24|test: pin call evidence code refs and mapped extent in dxcontainer_part_offsets_check
317ce645|2026-11-24|test: pin word masks and page and graphics bounds in xmem_attribute_scan_check
4ee1780f|2026-11-24|test: pin evidence label and failed cell wording in hardware_reports_check
26537e29|2026-11-24|test: pin conditional finding causes in conditional_import_paths
99230d69|2026-11-24|test: pin header, empty-root and budget boundaries in resource_root_check
dae6ddc5|2026-11-24|test: pin bcpack_symbols input type, result keys and CLI failures
0eb24969|2026-11-24|Merge pull request #1807 from legacy-code-archaeologist/shader/vop1-nops
f5a41a2b|2026-11-24|test: pin cache index bounds and header digest span in shader_census_check
107b6410|2026-11-25|startup: check the class helper by value, not by text
6dfabe09|2026-11-24|tests: pin candidate container field refusals in sgd5_effects_check
9314b151|2026-11-24|test: pin stream limits, 29-bit ids and empty tags in awc_metadata_check
82749f0f|2026-11-24|Merge pull request #1810 from ChazY11/test/gather-offset-refusal
1da587c7|2026-11-24|test: pin section probe and paged read bounds in resource_ranges_check
8741e6d0|2026-11-24|test: pin mode DC, the terminal state and input bounds in bcpack_check
46f11252|2026-11-24|test: pin compute catalogue refusals in dxil_retarget_check
15ce041b|2026-11-24|startup: refuse fetch slots the stand-in does not bind; test base vertex and two slots
6ff39343|2026-11-24|test: pin unsupported form ranking and decoder result refusals in shader_corpus_report_check
028edd03|2026-11-24|test: pin module-name and descriptor-table forms in import_forms_check
f3363374|2026-11-24|test: pin window arithmetic and refusals in startup_audio_timeline_check
bca2702c|2026-11-24|shader: round-trip the SDWA comparison fixture
8e6db198|2026-11-24|test: pin which file opens startup_live_view shows
178daa9b|2026-11-24|Merge origin/main (82749f0f) into startup/placed-allocation-cache
9b7775c8|2026-11-24|Merge pull request #1823 from default-cybe/tests/reloc-free-none-diagnostic
cf547ed4|2026-11-24|shader: lower the plain VOP1/VOP3 v_movrel* forms in pixel_aot
7a75c9ab|2026-11-24|shader: lower v_swaprel_b32 with the movrel m0 indexing in pixel_aot
f55b76f4|2026-11-25|Merge pull request #1808 from SuperCraft3M/startup/invalidate-stale-root-constants
58f182a3|2026-11-25|Merge pull request #1824 from Asimmimx/tests/host-commit-rounding
f738af81|2026-11-24|runtime: lower REX-prefixed and register-form SSE4a EXTRQ/INSERTQ stubs in host_cpu
ed39e1d0|2026-11-25|Merge pull request #1832 from Asimmimx/tests/intake-paths-gap
efcf9e27|2026-11-25|Merge pull request #1838 from Asimmimx/tests/section-probe-bounds
e1bc6007|2026-11-24|tests: add the reviewed gap-before-buffer repro to startup_placed_buffer_range
ae56e87c|2026-11-25|Merge pull request #1839 from Asimmimx/tests/bcpack-probe-bounds
cbf56af7|2026-11-24|Merge origin/main into startup/stream-input-layout-fetch
48a2859c|2026-11-25|Merge pull request #1813 from consolelogkyo/startup/gfx2hlsl-buffer-address
939e5142|2026-11-24|Merge pull request #1822 from default-cybe/shader/pixel-aot-med3-f32-nan-min3
466b9fe5|2026-11-24|startup: NGG cases report real interpretation outputs; decode control; docs
5ccdbbec|2026-11-24|Merge remote-tracking branch 'origin/main' into startup/ngg-first-slice
f8101db1|2026-11-24|startup: start DirectStorage status-array slots complete with S_OK
bbadc76d|2026-11-25|Merge pull request #1818 from triggered02/startup/b64-shift
37d58798|2026-11-24|Merge pull request #1825 from Asimmimx/tests/rectlist-topology-placement
ddd6db71|2026-11-24|Merge origin/main into startup/stream-input-layout-fetch
fb8947f0|2026-11-24|Merge pull request #1826 from Asimmimx/tests/texture-blocks-controls
f5ca2686|2026-11-24|startup: remove the unread SAMPLER_CLAMP table from gfx2hlsl
305d284f|2026-11-24|Merge pull request #1827 from Asimmimx/tests/parallel-runner-report
759e9063|2026-11-24|shader: harden DXIL/PSV0 validation and add unit suite for shader_profile_audit
dfd6bef2|2026-11-24|Merge pull request #1828 from Asimmimx/tests/call-evidence-code-refs
658dd726|2026-11-24|Merge pull request #1830 from Asimmimx/tests/xmem-word-boundaries
553f7322|2026-11-24|Merge pull request #1831 from Asimmimx/tests/bcpack-symbols-gaps
fd35dae0|2026-11-24|Merge pull request #1833 from Asimmimx/tests/shader-census-boundaries
5f16bc20|2026-11-24|Merge pull request #1835 from Asimmimx/tests/dxil-catalogue-refusals
733ecc40|2026-11-24|xmem: opt-in placement of 0x283 startup blocks below 2^40
8e2923ae|2026-11-24|Merge pull request #1836 from Asimmimx/tests/sgd5-effects-bounds
bd528414|2026-11-24|runtime: report DirectStorage queue errors and signal the event
2f729116|2026-11-24|Merge pull request #1837 from Asimmimx/tests/awc-sfx-limits
106555ab|2026-11-24|Merge pull request #1834 from Asimmimx/tests/resource-root-header-budget
b466e23c|2026-11-24|tooling: count ML64 fsetpm as the no-wait fnsetpm form in form coverage
56bd3bd1|2026-11-24|shader: unpack A16 coordinates for startup image loads
cd2b333c|2026-11-25|test: refuse a site too far from the host_cpu stub
5b6f3c51|2026-11-24|startup: NGG cases pin encoder words to golden vectors; null bit stated as an assumption
f767d83d|2026-11-24|Merge remote-tracking branch 'origin/main' into startup/ngg-first-slice
8a61ae06|2026-11-24|Merge pull request #1841 from Asimmimx/tests/import-forms-check
5e9a631a|2026-11-24|startup-runtime: report long or truncated X2W_INPUT_SCRIPT values instead of dropping them
f1872e5f|2026-11-24|Test the entry limit separately and follow the sibling tests' conventions
e8b3a510|2026-11-24|Correct the character count in the test comment
7c896330|2026-11-24|shader: lower compute v_readlane_b32
6ff0afbb|2026-11-24|cpu: cover FSTSW AX in the x87 examine fixture
dffd7f56|2026-11-24|shader: keep non-negative unknown stages as no-stage-profile and tighten audit guards
0631667d|2026-11-24|Merge pull request #1842 from Asimmimx/tests/audio-timeline-window-check
b2abbcc3|2026-11-24|docs: update startup-runtime host_cpu SSE4a limits and case count
5b5d9260|2026-11-24|startup: count DirectStorage page requests by where they end in the heartbeat
7aa033d5|2026-11-25|test: name a missing probe line when the debug-heap check fails
b1ab8a67|2026-11-25|test: name the shim target when the layout check cannot load it
45cf6bc5|2026-11-24|docs: link 5 GPU coverage retests from #1
5a6ad936|2026-11-24|Merge pull request #1844 from legacy-code-archaeologist/shader/swaprel
afddddbd|2026-11-25|Merge pull request #1849 from winne42/startup/drop-unused-sampler-clamp
aa5eed1a|2026-11-25|test: print the usage when binary mode gets the wrong argument count
6e16e21a|2026-11-24|runtime: forward a GPU-written indirect dispatch
ab4d8c9a|2026-11-24|test: use supported D3D12 fixture interfaces
749009ff|2026-11-24|docs: record WARP indirect-dispatch results
3e754f5d|2026-11-24|Merge origin/main into startup/stream-input-layout-fetch
7e13965b|2026-11-25|Give the status fixture a complete queue descriptor
7c96def1|2026-11-25|Merge pull request #1848 from Makto94/runtime/xmem-reader-window
51b3e05f|2026-11-25|Merge pull request #1753 from Asimmimx/docs/bit-alu-draw-results
4e6f2c81|2026-11-24|runtime: validate chad permissive out-handle and add lifecycle checks
abd992d2|2026-11-24|Merge branch 'main' of https://github.com/R0Y00/Xbox2Windows into startup/dstorage-error-record
5b46e98e|2026-11-24|shader: refuse an unexpected NSA extension on packed loads
be9862b6|2026-11-24|ci: run the startup audio timeline check in CTest and the Python job
8be6ef00|2026-11-25|Merge pull request #1733 from Asimmimx/tests/dstorage-rfc1950-check
b509a670|2026-11-25|Merge pull request #1847 from Jess-Doit/startup/dstorage-status-initial
9d9c5a0a|2026-11-24|runtime: validate GameInput COM pointers and add contract checks
dd753176|2026-11-24|Merge pull request #1852 from Asimmimx/tooling/fsetpm-coverage-spelling
b7854f13|2026-11-24|shader: lower compute s_bcnt1_i32_b32 of exec_lo as a known word
0e324e95|2026-11-24|Merge pull request #1866 from Makto94/docs/link-hardware-reports-3
d6489798|2026-11-25|Merge pull request #1860 from MasterDomino/runtime/startup-blocks-below-40
fb2be030|2026-11-25|Merge pull request #1867 from avt0m4t48/fix/startup-indirect-dispatch
150a01d3|2026-11-25|Merge pull request #1864 from MasterDomino/startup/storage-page-shape-counts
a2f24b6a|2026-11-25|Merge pull request #1854 from avt0m4t48/fix/startup-image-load-layout
397caf1f|2026-11-24|gpu: select DXR opacity backends in the operation table
b5e765d4|2026-11-24|startup: prove fetched registers defined only in the entry block; pin the layout refusals
df40e4bc|2026-11-24|Merge pull request #1858 from JustinDCosta/shader/compute-v-readlane
e90a7561|2026-11-24|docs: record the RDNA 4 subnormal-immediate result and its controls
0c32ee16|2026-11-25|test: refuse a comparison sample without one dim and dmask
f8b39a5b|2026-11-25|test: refuse image_sample_l with the wrong address count
9bba6a77|2026-11-25|test: refuse a comparison sample whose sampler is not constant
60ee4c20|2026-11-24|tests: assert permissive chad log entries and clarify header comment
2857ddc4|2026-11-24|startup: guard and verify 64-bit VOP3 shifts in gfx2hlsl
f13a7a7b|2026-11-24|test: isolate readlane guarded refusal
122e2db2|2026-11-24|shader: define v_cvt_rpi/flr_i32_f32 for NaN, infinity and overflow in pixel_aot
ea3b6378|2026-11-24|startup: check inflate matches up to the 32 KiB window
f77a6a97|2026-11-25|Merge pull request #1859 from Leyukaka/input-script
46b75a30|2026-11-25|Merge pull request #1735 from Asimmimx/tests/era-gpu-alias-check
b560c3be|2026-11-25|Merge pull request #1736 from Asimmimx/tests/era-physical-check
38e79ae6|2026-11-24|startup: end the unconditional region at any exec write after a fetch call
59c9e234|2026-11-25|Merge pull request #1878 from ChazY11/test/image-compare-modifiers
c9253969|2026-11-24|Merge origin/main into startup/stream-input-layout-fetch
b5df5287|2026-11-24|startup: track a refused page request through the consumer view
3041aab1|2026-11-24|Merge pull request #1872 from henriazz10/shader/compute-bcnt-exec
dfa9b9f5|2026-11-24|Merge pull request #1880 from ChazY11/test/image-compare-const-sampler
04291bc1|2026-11-24|tests: pin the SDWA comparison models without a GPU
8c82879f|2026-11-24|Merge pull request #1879 from ChazY11/test/image-sample-l-address-count
d1d4c28b|2026-11-24|startup: keep one() usable on unit probes that build a Stage without __init__
4117f0c0|2026-11-24|Merge branch 'main' of https://github.com/R0Y00/Xbox2Windows into startup/dstorage-error-record
cf00afe3|2026-11-24|startup: record CommandType in the DirectStorage error record
e60e4e17|2026-11-24|gpu: rotate desktop allocator after failed list Close before recovery
d2b55bd8|2026-11-24|shader: lower v_writelane_b32 on known words in compute_store_aot
14ce7e79|2026-11-24|shader: cover runtime mesh geometry inputs
18133b5a|2026-11-24|Merge pull request #1883 from JustinDCosta/test/readlane-guard-refusal
c57ef28e|2026-11-24|Merge remote-tracking branch 'upstream/main' into runtime/gameinput-pointer-and-contract-check
99266a58|2026-11-24|runtime: forward one GPU-written indirect indexed draw
b80058f0|2026-11-24|Merge pull request #1862 from FAJIFBEFJHA/gpu/dxr-opacity-tmin
a1aa1d98|2026-11-24|startup: NGG cases anchor field layouts to public sources; wave order, exec and shared-LDS checks
cf27add8|2026-11-24|runtime: harden save_blob_pack size checks and add unit fixture
5307fc60|2026-11-24|Merge remote-tracking branch 'origin/main' into startup/ngg-first-slice
5f2edd46|2026-11-24|startup: pin the error event's SetEvent and the request-name cut in tests
cc98d047|2026-11-24|shader: lower the scalar s_movrel* moves in pixel_aot
9a06df39|2026-11-24|pe: validate package plan edges and add pe_run_package unit checks
8a03d2ad|2026-11-24|Merge pull request #1886 from MasterDomino/startup/storage-consumer-pair
09b125ca|2026-11-24|startup: lower 32-bit scalar writes to m0 and track symbolic m0 state
4c5afa8e|2026-11-24|Merge pull request #1829 from Asimmimx/tests/hardware-reports-cell-wording
bedd5229|2026-11-24|ffx: validate FSPX and PSV0 bounds and add dispatch profile checks
2d5be580|2026-11-24|startup: prove slot 133 placements with region information first
2cd369aa|2026-11-24|docs: link 3 Windows 10 GPU coverage reports from #1
e8718f9f|2026-11-25|test: keep a skipped test out of the failed list
b8dfc207|2026-11-25|Merge pull request #1874 from Ammmae0x/runtime/gameinput-pointer-and-contract-check
69077aa2|2026-11-25|Merge pull request #1871 from Ammmae0x/runtime/chad-permissive-check
e3bd57c7|2026-11-25|Merge pull request #1851 from NiiRoZz/startup/stream-input-layout-fetch
b180e9c4|2026-11-25|Merge pull request #1783 from R0D4X/fix/issue-1723-gfx2hlsl-shift64
8ed64724|2026-11-25|Cover untracked command-list recovery without the debug layer
44136651|2026-11-24|Merge pull request #1840 from Asimmimx/tests/shader-corpus-form-ranking
4eeaf3b0|2026-11-24|Merge pull request #1885 from Asimmimx/shader/cvt-rpi-flr-exceptional
23d9d12d|2026-11-24|Merge pull request #1894 from legacy-code-archaeologist/shader/scalar-movrel
2de7fe7f|2026-11-24|test: require the debug summary in mesh_multi_check only with the layer on
b98e69fa|2026-11-25|Merge pull request #1888 from R0D4X/fix/issue-1876-list-resubmit-nvidia-crash
b77050ee|2026-11-25|Merge pull request #1891 from avt0m4t48/fix/startup-indirect-indexed
3ad0069d|2026-11-25|Merge pull request #1873 from Jess-Doit/startup/dstorage-error-record
3cc12064|2026-11-24|startup: lower combined register+immediate SMEM offsets in gfx2hlsl
3556bc28|2026-11-24|startup: create committed upload and readback resources in their required initial states
407f8b92|2026-11-24|Merge pull request #1767 from ChazY11/test/root-descriptor-key
9948d80d|2026-11-24|Merge pull request #1719 from thepatrickworld/panam/958-matrix-doc
94dbdbf0|2026-11-24|Merge pull request #1555 from Ammmae0x/runtime/startup-trap-header-check
06d5d72c|2026-11-25|startup: fail XDspActivate and XDspSubmitCommand instead of succeeding without outputs
5d97a2bb|2026-11-24|test: exercise a GPU-written indirect Draw record
07548688|2026-11-24|startup: input script holds, stick and trigger values, a process-start time base
d33f2d18|2026-11-24|Merge remote-tracking branch 'origin/main' into startup/trap-log-single-open
fd8dfcf1|2026-11-24|tooling: harden bounds in bink_heap_aot and add synthetic PE32+ unit and refusal suite
bbf89779|2026-11-24|shader: read attr0.z/w as plain values in sampled pixel programs
fa4a3098|2026-11-24|Merge pull request #1904 from henriazz10/shader/compute-writelane
ba21b680|2026-11-24|runtime: forward one GPU-written indirect Draw
bfcb8a10|2026-11-24|Merge pull request #1899 from ChazY11/test/hardware-reports-skip-not-failed
db924b76|2026-11-24|Merge pull request #1898 from Makto94/docs/link-hardware-reports-4
55dcda96|2026-11-24|shader: lower bounded runtime mesh geometry comparisons
1d1f638b|2026-11-24|ffx: add clean CLI error handling in ffx_create_context and unit and refusal suite
2d68c156|2026-11-24|tests: decision cases for write-watch, top-down, executable, heap, AWE and large-page allocations
df41938b|2026-11-24|Merge origin/main (db924b76) into startup/placed-allocation-cache
c3c6ce02|2026-11-24|docs: record indirect Draw verification on Windows WARP
c3b05e45|2026-11-25|Canonicalize named trap ordinals before matching routes
a88f30e1|2026-11-25|Merge pull request #1743 from NiiRoZz/startup/inflate-table-fast-path
cdd188ec|2026-11-24|tooling: add a read-only Authenticode SHA-256 digest tool
f8f9bd7f|2026-11-25|Merge pull request #1819 from NiiRoZz/startup/inflate-second-pass
8c447809|2026-11-25|Merge pull request #1889 from avt0m4t48/tests/dynamic-mesh-geometry
e263314d|2026-11-25|Merge pull request #1892 from Asimmimx/runtime/save-blob-pack-check
6a41a3e9|2026-11-25|Validate m0 sources before clearing their symbolic state
20f8e1d5|2026-11-24|tests: heap-block and AWE expectations in startup_placed_buffer_range
e6e327d6|2026-11-24|test: verify runtime mesh lowering pixels on D3D12
07c262a4|2026-11-24|shader: fold s_ff1_i32 of exec outside guarded regions
309d80f4|2026-11-25|Merge pull request #1881 from consolelogkyo/startup/gfx2hlsl-smem-offset-mods
2d42df74|2026-11-25|Merge pull request #1893 from Ammmae0x/pe/run-package-plan-validation
0c9be0f1|2026-11-24|test: exercise GPU-written indirect mesh group counts
66f70135|2026-11-24|Merge remote-tracking branch 'upstream/main' into tooling/bink-heap-aot-validation
0ef61543|2026-11-25|runtime: forward one GPU-written indirect DispatchMesh
352b62c5|2026-11-25|runtime: confine mesh stream admission to native pipelines
73d34ceb|2026-11-25|Merge pull request #1901 from lukaslw-lab/startup/committed-heap-initial-state
925198a1|2026-11-24|startup: lower VOPC SDWA compares in gfx2hlsl
d30004d4|2026-11-25|Merge pull request #1909 from avt0m4t48/fix/startup-indirect-draw
3e62f716|2026-11-25|Keep both descriptor controls when merging main
f3d96d31|2026-11-25|Document the XDsp failure-only startup behavior
46bf486e|2026-11-24|Merge pull request #1903 from henriazz10/shader/compute-ff1-exec
015b8d89|2026-11-25|docs: record indirect mesh verification on Windows WARP
498dcf58|2026-11-25|Keep the symbolic m0 guard for SMEM offsets
ba0449cc|2026-11-24|Merge pull request #1734 from Asimmimx/tests/api-trace-check
87f280f3|2026-11-25|Merge pull request #1907 from ChazY11/startup/xdsp-failure-only
c681e34a|2026-11-25|Merge pull request #1769 from ChazY11/test/numeric-symbol-refusal
82ceaf3f|2026-11-25|Merge pull request #1895 from consolelogkyo/startup/gfx2hlsl-scalar-m0
7f06265c|2026-11-25|Merge pull request #1870 from Asimmimx/ci/register-audio-timeline-check
d9c5b926|2026-11-25|Merge pull request #1868 from ChazY11/test/pixel-aot-binary-usage
27cf223d|2026-11-25|Merge pull request #1865 from ChazY11/test/xg-layout-load-hint
0c560c21|2026-11-24|compute_store_aot: lower v_cvt_u32_f32_e32
be88b353|2026-11-25|Merge pull request #1863 from ChazY11/test/debug-heap-flag-diagnostic
df15e925|2026-11-25|Merge pull request #1912 from roansystemlabs/pixel/730-attr0-zw
67a240ba|2026-11-25|Merge pull request #1916 from avt0m4t48/fix/startup-indirect-mesh
a6514901|2026-11-25|Merge pull request #1908 from Ammmae0x/tooling/bink-heap-aot-validation
435a15f2|2026-11-24|pe: harden call_evidence CLI and import guards and add unit suite
7f71875f|2026-11-25|Merge pull request #1897 from Ammmae0x/ffx/dispatch-profile-validation
4ab5e018|2026-11-25|Merge pull request #1845 from Asimmimx/tests/live-view-file-opens
2cfaea43|2026-11-25|shader: lower fixed-mip 2D texture gathers at startup
800a8e34|2026-11-24|startup: keep VOPC SDWA compares integer-only in gfx2hlsl
e4697f3a|2026-11-25|Merge pull request #1913 from avt0m4t48/tests/runtime-mesh-lowering
8730bc12|2026-11-25|test: name the stale fixture DLL when an export is missing
9881fef0|2026-11-25|Merge pull request #1857 from ChazY11/test/host-cpu-plan-range
11a7de79|2026-11-25|Clarify which PE header values the digest tool validates
5c243aba|2026-11-24|tests: pin the swaprel selects and the plain movrel entry-mask refusal
987faa5a|2026-11-25|Keep both new CLI validation tests when merging main
ecce2d6b|2026-11-24|tests: run the SDWA comparison controls on Linux
3b94f17d|2026-11-24|shader: align not_checked schema and add CLI and contract suite for xbox_intrinsic_gate
492b900d|2026-11-25|test: match the gather renderer vertex signature
3a149841|2026-11-24|Merge pull request #1905 from henriazz10/shader/compute-cvt-u32-f32
cf120552|2026-11-25|Merge pull request #1915 from MasterDomino/startup/gfx2hlsl-vopc-sdwa
9b6cfcbf|2026-11-25|Merge pull request #1911 from Makto94/tools/pe-authentihash
8b29d095|2026-11-25|Merge pull request #1910 from Ammmae0x/ffx/create-context-validation
1c8260ae|2026-11-25|Merge pull request #1921 from Ammmae0x/pe/call-evidence-validation
a8fa733b|2026-11-25|test: index gather fixture levels relative to the SRV
7ca344dc|2026-11-25|Merge pull request #1861 from Asimmimx/cpu/x87-fstsw-fixture
c580d904|2026-11-25|Merge pull request #1922 from ChazY11/test/x87-missing-export-diagnostic
aeb6d280|2026-11-24|startup: lower s_nand_b32, s_nor_b32 and s_xnor_b32 in gfx2hlsl
6d0a7885|2026-11-25|Merge pull request #1875 from legacy-code-archaeologist/tests/movrel-swaprel-pins
91f6bf69|2026-11-24|startup: refuse the p10 and p20 flat-interpolation selectors
682dd4f9|2026-11-25|docs: describe the startup mip-zero gather profile
b96960b7|2026-11-25|Merge pull request #1924 from Ammmae0x/shader/xbox-intrinsic-gate-cli-and-contract
9b620015|2026-11-24|startup: refuse lane-mask forms of the 32-bit complement ops in gfx2hlsl
e557d74d|2026-11-25|test: register the seven documented device-free checks in CTest
fc9517aa|2026-11-24|startup: lower gfx2hlsl f32 lg/nlg compares as ordered
bc03f191|2026-11-24|shader: lower the SDWA v_movrel* forms in pixel_aot
3ef8e9bd|2026-11-24|startup: lower NEG on an immediate source of v_cndmask_b32_e64 from its encoding
3920220f|2026-11-25|Merge pull request #1756 from tedin7/x2w-frontier-1199a-slabs
8b0f41a3|2026-11-24|Merge remote-tracking branch 'origin/main' into startup/trap-log-single-open
e6e3fd25|2026-11-24|startup: prove lane-pure v_readfirstlane_b32 sources uniform in straight-line code
edd281c9|2026-11-24|Close uniformity gaps: aliased scalar-load offsets, WQM, saved exec, 64-bit high words
23a2cffa|2026-11-24|Refuse proofs across restored exec copies, lane masks and overlapping or symbolic 64-bit moves
cf71ddb6|2026-11-25|Merge pull request #1850 from Ammmae0x/shader/profile-audit-validation
5a031940|2026-11-25|Integrate op_sel refusals with integer SDWA compares
9b8e6253|2026-11-24|docs: close the scalar m0 section's code block before the readfirstlane section
6bd3e3d2|2026-11-24|ffx: harden device-init dispatch lookup and CLI error handling with unit suite
2a27507a|2026-11-25|Merge pull request #1779 from roansystemlabs/mesh/730-multi-primitive
93f8cee9|2026-11-24|shader: harden probe_root_devices validation and add offline unit suite
34105de4|2026-11-25|Refuse symbolic descriptor inputs to scalar complements
56a40e77|2026-11-24|Merge remote-tracking branch 'origin/main' into startup/ngg-first-slice
3876a14d|2026-11-25|Keep the default debug linker flags
148d6daa|2026-11-24|startup: retry a failed trap log open instead of keeping the failure
a20fc744|2026-11-25|Merge pull request #1790 from lambirou/shader/sdwa-cmp
8c662534|2026-11-25|Merge pull request #1766 from rasptum/startup/placed-allocation-cache
89598708|2026-11-25|Merge pull request #1925 from MasterDomino/startup/gfx2hlsl-sop2-complement-b32
ea82122f|2026-11-25|Merge pull request #1755 from triggered02/startup/vf-opsel
fc32ceb5|2026-11-25|Merge binary32 class compares with current modifier handling
18fe83d5|2026-11-25|Merge pull request #1752 from tedin7/x2w-cmake-fastlink-obj
c04c27f8|2026-11-24|build: write the variants of each x87 fixture in one Python process
0bf9c546|2026-11-24|build: allow a single variant pair in emit_variants.py
0e1a22c5|2026-11-25|Merge pull request #1798 from triggered02/startup/cmp-class
57cb5fe9|2026-11-25|Merge pull request #1932 from Leyukaka/shader/cndmask-neg-immediate
57bc15b5|2026-11-25|Merge pull request #1929 from ChazY11/test/register-documented-device-free-checks
bfb8d132|2026-11-25|Merge pull request #1933 from Ammmae0x/ffx/device-init-validation
6ef541f3|2026-11-24|test: construct a reserved prefix for placed buffer checks
6098b735|2026-11-25|Merge pull request #1926 from roansystemlabs/shader/1890-interp-mov-selector
746060de|2026-11-25|Remove the temporary report when publish fails
82f0db65|2026-11-25|Merge pull request #1927 from avt0m4t48/fix/startup-gather-lz
b464076c|2026-11-24|build: compile four checks' shared sources once
ef9a0798|2026-11-24|tooling: add a run end-point section to the startup triage report
f6c377b9|2026-11-25|shader: lower quad-local DPP16 and DPP8 cvt_flr, cvt_rpi, and cvt_pkrtz in pixel_aot
5e2aa88d|2026-11-24|startup: NGG cases credit the s3 location of the wave info to mesh_entry_aot
0e814819|2026-11-24|gpu: compare one linear buffer through sharing, copying and refused sharing
37061481|2026-11-24|Merge remote-tracking branch 'origin/main' into startup/ngg-first-slice
4d0fdea0|2026-11-24|ffx: harden ffx_working_set_delta validation and add unit suite
36cdc213|2026-11-25|Keep both startup compare controls in CI
09b21dfc|2026-11-24|Merge commit '82f0db65' into startup/ngg-first-slice
71d4bff1|2026-11-24|startup: recognise a console indirect argument type before CreateCommandSignature (slot 41)
f2775cfd|2026-11-25|tooling: harden bink_descriptors_aot CLI and add unit suite
387bdaf5|2026-11-25|Merge pull request #1934 from legacy-code-archaeologist/shader/sdwa-movrel
999875ad|2026-11-25|Merge pull request #1846 from Ammmae0x/runtime/host-cpu-sse4a-reg-rex
fd999952|2026-11-25|test: refuse a gather destination that runs past v255
4fe59c25|2026-11-24|compute_store_aot: lower v_add_nc_i32 and v_sub_nc_i32 on known words
c514f20b|2026-11-24|Merge pull request #1937 from SuperCraft3M/test/placed-buffer-heap-layout
9727c306|2026-11-24|fix: reserve physical-page slabs and commit slots on first use
ff76a552|2026-11-24|Merge pull request #1941 from KomaGuy/shader/dpp-quad-cvt-flr-rpi-pkrtz
53d7989a|2026-11-24|feat: implement GetDescriptorHandleIncrementSize (slot 15) in startup device
ea0cf33a|2026-11-25|Merge pull request #1930 from MasterDomino/startup/gfx2hlsl-f32-lg-ordered
aadcbd2c|2026-11-25|Merge pull request #1789 from tedin7/x2w-cmake-batch-py
cd5508e9|2026-11-25|Merge pull request #1942 from MasterDomino/gpu/unified-buffer-paths
583178d4|2026-11-24|startup: keep gfx2hlsl f32 n-forms, o and u exact under FXC and DXC
447c38df|2026-11-25|Merge pull request #1757 from rasptum/startup/trap-log-single-open
2db78fe2|2026-11-25|Merge upstream/main and address PR #1935 review feedback
87c120bc|2026-11-24|Merge commit '999875a' into pr1931-update
ed88a09e|2026-11-24|Track SCC written through m0 in the uniformity proof
0478a386|2026-11-24|fix: split a larger placeholder before mapping a physical-page view into it
11f11e84|2026-11-24|Merge pull request #1953 from ChazY11/test/gather-dest-refusal
d28732b6|2026-11-24|shader: accept an unavailable debug layer in the attr0.z/w sampled draw
e366d03a|2026-11-24|Invalidate the SCC proof where the SCC result is not modelled
5a77e32c|2026-11-25|startup: explain formatted MUBUF descriptor refusals
f6856564|2026-11-25|tooling: harden bink_frame_aot anchor validation and add unit suite
57ba8d8e|2026-11-25|Merge pull request #1935 from Ammmae0x/shader/probe-root-devices-validation
b3a90888|2026-11-25|startup: report pending results for an empty XApu queue
3682b34f|2026-11-24|Merge pull request #1957 from henriazz10/shader/compute-vadd-nc-i32
65d07996|2026-11-25|pe: harden pe_inventory changes and CLI validation with unit suite
f360d30a|2026-11-24|startup: stop the stand-ins' first-calls log counters at their limit
4c4502b8|2026-11-25|Merge pull request #1949 from MasterDomino/startup/gfx2hlsl-f32-nan-forms
d8b51b5f|2026-11-25|shader: lower quad-local DPP16 and DPP8 binary16 arithmetic and rounding in pixel_aot
89694c00|2026-11-24|ci: register slot 15 descriptor increment tests
1d825c18|2026-11-25|Merge pull request #1961 from NiiRoZz/startup/ngg-first-slice
f1bc5eca|2026-11-25|startup: lower binary32 class SDWA compares
150c625a|2026-11-25|Merge pull request #1962 from rasptum/startup/input-script-holds-axes
f0e33436|2026-11-25|Merge upstream/main into ffx/working-set-delta-validation
34472a58|2026-11-25|Merge pull request #1967 from avt0m4t48/fix/startup-xapu-empty-results
b1715e1e|2026-11-25|Merge pull request #1965 from avt0m4t48/fix/startup-mubuf-format-refusal
889cd3f4|2026-11-25|Merge pull request #1964 from alvaroolt/shader/attribute-sampled-debug-layer
20fa94a9|2026-11-24|tooling: report disasm_rdna2 CLI input errors and honor --output with --hex
7205fcd8|2026-11-24|tooling: consume DPP extension dword in disassembled VOPC
83bb752d|2026-11-24|tooling: sign-extend the SMEM immediate offset in disasm_rdna2
24e5a171|2026-11-24|tooling: skip BOM and commented labels in scan_unsupported_shaders assembly parsing
8194c91e|2026-11-25|Merge upstream/main into tooling/bink-descriptors-aot-validation
2260b7a4|2026-11-25|startup: reject new updates on a closed save container
0b649c69|2026-11-24|build: add SKIP_RETURN_CODE 77 to hardware tests that skip on a missing adapter
5510deca|2026-11-24|startup: add a read-only --refusals census of gfx2hlsl refusal sites
b4d1cf01|2026-11-24|Keep the census faithful to the translator; cover --refusals=DIR and raw encoding names
425cd53b|2026-11-24|Keep the refusals check independent of fxc
2e5fdca4|2026-11-24|tests: pin the SDWA v_movrel* selections without a GPU
5d6d5f52|2026-11-24|fix: include initializer_list in heap type check
cc171feb|2026-11-25|tests: pin pe_inventory structure and directory validation
5b2dadd7|2026-11-25|tooling: ignore MASM labels, directives and assignments in form coverage
454af100|2026-11-25|Merge pull request #1976 from avt0m4t48/fix/startup-save-closed-update
4ca8d33c|2026-11-25|Merge pull request #1968 from Ammmae0x/tooling/bink-frame-aot-validation
5ac154f9|2026-11-24|ci: run slot 15 increment test in permissive mode
89b8230d|2026-11-25|Merge pull request #1950 from Ammmae0x/tooling/bink-descriptors-aot-validation
bf0c42c9|2026-11-25|pe: add cpu_stdio_exe_aot CLI entry and synthetic validation suite
99eb6e0f|2026-11-25|Merge remote-tracking branch 'origin/main' into codex/startup-telemetry
e2a6ef57|2026-11-24|gpu: add a range provenance fixture for bound-range writers (#870)
4fe9e20e|2026-11-25|ffx: add ffx_dispatch_aot CLI entry and offline validation suite
9dcf1ae1|2026-11-25|Merge pull request #1969 from avt0m4t48/fix/startup-class-f32-sdwa
f558d4c2|2026-11-24|startup: lower three-source integer min, max and median in gfx2hlsl
ba2df377|2026-11-24|Check constant and literal sources, and state what the med3 check does not cover
5fdb913a|2026-11-25|Merge main and retain both synthetic AOT checks
05ef4d92|2026-11-25|shader: join pending loads at each masked arm boundary
6b50e3d7|2026-11-25|Merge remote-tracking branch 'origin/main' into codex/startup-telemetry
9a2b9cd8|2026-11-25|Validate inventory entries before comparing baselines
80bde65a|2026-11-25|Merge pull request #1948 from Ammmae0x/ffx/working-set-delta-validation
ac6814ad|2026-11-24|Merge pull request #1983 from RenzoWit35/tooling/disasm-vopc-dpp
2deaa030|2026-11-25|Merge pull request #1978 from Leyukaka/startup/gfx2hlsl-refusals
9ff8ee1a|2026-11-24|Merge remote-tracking branch 'origin/main' into x2w-compute-store-format
46303c0a|2026-11-25|test: pin the hex redaction outside operand positions in a refusal reason
b2c99ea2|2026-11-25|Merge remote-tracking branch 'upstream/main' into pe/stdio-exe-aot-validation
1e7436db|2026-11-25|Merge pull request #1970 from Ammmae0x/pe/inventory-changes-and-cli-validation
93e4753a|2026-11-24|Merge pull request #1986 from RenzoWit35/tooling/scan-assembly-bom-labels
e63721f9|2026-11-24|Merge pull request #1988 from avt0m4t48/fix/pixel-aot-masked-scalar-waits
f018831b|2026-11-24|startup: lower v_swap_b32 in gfx2hlsl
82c07000|2026-11-25|Merge pull request #1945 from tedin7/x2w-cmake-shared-objs
4a948312|2026-11-25|Merge pull request #1959 from tedin7/x2w-frontier-dl646-slot15-v2
e893cba4|2026-11-25|Merge pull request #1993 from ChazY11/test/pin-refusal-hex-redaction
6139c664|2026-11-24|Merge commit '2deaa030' into pr1931-update
4b8d92e0|2026-11-25|Merge pull request #1979 from legacy-code-archaeologist/tests/sdwa-movrel-pins
c3c3ab3c|2026-11-24|startup: unpack 2D A16 coordinates for image_sample and image_sample_lz
8c2d82f6|2026-11-24|startup: input script waits for the presented picture, marks and relative times
f0451ea5|2026-11-24|startup: per-wait change sensitivity, regions, a fine signature grid and groups
1ebfe511|2026-11-25|Merge pull request #1991 from thepatrickworld/panam/870-range-provenance
63d30a50|2026-11-24|startup: NGG cases gain a binary mode that reassembles them with llvm-mc
2c1dcdd6|2026-11-24|tests: read the kept trap log with sharing in startup_view_heap
36a9a06f|2026-11-25|Merge pull request #1989 from sydney-0o/codex/startup-telemetry
b87a99a3|2026-11-25|tooling: resume startup shader compilation after manifest-only runs
14d6023e|2026-11-25|Preserve the existing isolated MASM parser fixture
5b7947e5|2026-11-25|Merge pull request #1990 from Ammmae0x/ffx/dispatch-aot-cli-validation
de2ca7ea|2026-11-25|Merge pull request #1981 from Ammmae0x/pe/stdio-exe-aot-validation
06a494c8|2026-11-25|Merge pull request #1982 from RenzoWit35/tooling/disasm-rdna2-cli
13745b1b|2026-11-25|test: pin every shader reason token in the triage classifier
2a898640|2026-11-24|startup: picture signature holds its surface until the copy ends; sampling is opt-in
03de07ef|2026-11-25|Merge pull request #1987 from rasptum/startup/log-limiter-saturate
9d29375c|2026-11-25|Merge pull request #1992 from Leyukaka/shader/int-med3
66423e45|2026-11-25|Merge pull request #1980 from davutsimsekk/fix/form-coverage-masm-tokens
f1ade5b3|2026-11-25|tests: make all DPP f16 arith/round mutations numerically distinct
86c225cc|2026-11-25|Retain the range fixture and scope adapter skips to hardware
3ff6e9d4|2026-11-24|shader: lower gfx1013 image_atomic_add, umax and cmpswap
21e8f5c0|2026-11-25|Preserve placeholder ownership and test slab commit rollback
c168cf80|2026-11-25|shader: lower startup scalar high-halfword packing
f37fe7b8|2026-11-24|Pin descriptor-store words and match UNORM tie rounding
b19772fc|2026-11-25|ci: put the swap check on its own line in the pure-Python list
ac539650|2026-11-24|startup: arm the debugger's runtime and movie-player call hooks only with --trace-calls 1
5d63a269|2026-11-25|Merge pull request #1958 from tedin7/x2w-frontier-1199-commit
ed7d09a4|2026-11-25|Preserve SMEM refusals with signed decoder offsets
7f149ac9|2026-11-25|Merge pull request #1985 from RenzoWit35/build/hardware-skip-return-codes
a0bc0f9a|2026-11-25|Merge pull request #1984 from RenzoWit35/tooling/disasm-smem-signed-offset
1744710c|2026-11-25|Merge pull request #2009 from ChazY11/test/pin-shader-reason-table
70fdf29a|2026-11-25|Merge pull request #2000 from NiiRoZz/startup/ngg-slice-llvm-mc
032faab4|2026-11-25|Merge pull request #1995 from JB2WIN/startup/gfx2hlsl-swap
481a1c62|2026-11-25|Merge pull request #2014 from avt0m4t48/fix/startup-s-pack-hh
7ea49908|2026-11-25|startup: lower s_bitcmp1_b32 and s_bitcmp0_b32 in gfx2hlsl
d2f0e1df|2026-11-25|startup: lower s_lshl1..4_add_u32 in gfx2hlsl
a8b2ab85|2026-11-25|Merge pull request #1997 from consolelogkyo/startup/gfx2hlsl-image-sample-a16
8ec25533|2026-11-25|Merge pull request #2008 from avt0m4t48/fix/startup-resume-shader-compilation
47fa55b5|2026-11-24|cpu: lower register-indexed memory bit tests
e99a7876|2026-11-25|Merge pull request #1683 from SimSalabimse/x2w-compute-store-format
e50a72bc|2026-11-25|Preserve desktop indirect argument types and isolate the debug control
ab9b0283|2026-11-25|Merge pull request #1977 from rasptum/startup/input-script-wait
2ee7627b|2026-11-25|Merge pull request #2013 from Jess-Doit/shader/gfx1013-image-atomics
9bc7c297|2026-11-25|Merge pull request #1931 from Leyukaka/shader/readfirstlane-uniform-straight-line
0d20c5e2|2026-11-25|Merge pull request #2001 from JB2WIN/tests/view-heap-shared-read
88767ebc|2026-11-25|cpu: lower LAR, LSL, VERR, and VERW segment inspection instructions
06cdc47d|2026-11-25|Merge pull request #2018 from tedin7/x2w-frontier-dl743-cmdsig-v2
476ab123|2026-11-25|docs: describe iterating on start-up runs (isolated VM, warm shader cache, refusal census)
5b29fe0f|2026-11-25|docs: match the iteration section to the retry and .fail rules on main
442d289e|2026-11-25|Merge pull request #2021 from rasptum/startup/trace-calls-opt-in
ca1ed1a9|2026-11-25|shader: reject projected masks in startup scalar bit tests
489d5797|2026-11-25|Merge pull request #2031 from louisdibiasi/louis/cpu-bit-string
68d04fd5|2026-11-24|compute_store_aot: lower s_add_i32, s_sub_i32 and s_abs_i32
6855f425|2026-11-25|cpu: document the fixed Windows segment inspection profile
127f5f80|2026-11-25|Merge pull request #1971 from KomaGuy/shader/dpp-quad-f16-arith-round
e1aa07e0|2026-11-25|Merge pull request #2043 from Leyukaka/docs/startup-iteration-method
313803bc|2026-11-25|tooling: report confirmed debugger stops separately
9383b531|2026-11-25|Merge pull request #2024 from tedin7/x2w-r20-s-bitcmp
85a2aa65|2026-11-25|shader: refuse unsupported startup shift-add destinations
05092b6e|2026-11-25|Merge pull request #2042 from Ammmae0x/cpu/seg-inspect-aot
b80a3a5b|2026-11-25|Merge main and retain SCC uniformity in startup scalar lowering
928a19e3|2026-11-25|Merge main and retain startup triage controls
316945cf|2026-11-25|Merge pull request #2027 from tedin7/x2w-r20-s-lshl-add
2fd3f696|2026-11-25|Merge pull request #1956 from henriazz10/shader/compute-salu-i32
6218beb2|2026-11-25|Merge pull request #1947 from tedin7/x2w-frontier-triage-endpoint
```


## L. Limitacoes desta analise

- **Sem rede.** Nao consultei issues, PRs, discussoes nem o Discord do projeto. Todo numero de issue citado (#870, #1011, #1199, #1545, #1699, #1723, #1876, #1917, #1919, #1920, #1934, #1936, #1996, #2021, #2042) vem da mensagem do commit ou do diff, nunca de leitura da issue. Onde a issue seria a fonte autoritativa (por exemplo o estado de #870 "progress and current blockers" ou o resultado do PR #1135 no RX 9070 XT), a afirmacao e `NAO VERIFICADA`.
- **Sem execucao.** Nao compilei, nao rodei CTest e nao executei nenhuma fixture. Toda afirmacao sobre "o teste passa/falha" e sobre comportamento em WARP/hardware vem do texto do commit e de `docs/`. Onde o repositorio registra o par (adaptador, driver) -- por exemplo RX 9070 XT `1002:7550` driver `32.0.32029.2010`, WARP `10.0.26100.9818` -- reproduzo o registro, nao o medi.
- **Historico dominado por merge.** 1823 de 4076 commits sao merges; um fix que chega por merge aparece duas vezes no indice (o commit do ramo e o merge) e o merge normalmente nao tem diff proprio. Para evitar dupla contagem, as cadeias foram fechadas no commit do ramo (segundo pai), e os merges foram classificados como `c` salvo quando carregam um fix identificado.
- **Arquivos gerados e documentacao.** `docs/experiments.md` tem 1,7 MB e e o registro de medicao do proprio projeto; tratei-o como **evidencia citada**, nao como fonte independente. Onde ele afirma um resultado de hardware, a confianca e a do projeto, nao desta analise.
- **423 commits fora de main.** Existem em 100+ branches (`codex/*`, `startup/*`, `shader/*`, `tools/*`, `fix/*`) e **nao** foram percorridos um a um; o indice da secao K cobre os 4076 de HEAD. A enumeracao completa esta em `analise\indices\Xbox2Windows_log_full.txt` (1585 KB) e o indice compacto de todas as refs em `analise\indices\Xbox2Windows_log_compact.txt`. Consequencia: um defeito que exista apenas num ramo nao mergeado nao aparece aqui -- e, pelo proprio desenho do projeto, esses ramos sao experimentos.
- **Introducoes nao identificadas.** Dez defeitos (D-08, D-11, D-12, D-14, D-17, D-18, D-22, D-24, D-25, e a introducao de D-27) tem commit introdutor `NAO IDENTIFICADO`. Em todos esses casos o defeito e anterior ou difuso no recorte do pacote de evidencias (que prioriza os commits de correcao); dizer "introduzido em X" exigiria varrer todo o historico de `CMakeLists.txt` (912 alteracoes) ou de arquivos que nasceram ja incompletos. Preferi marcar do que inferir.
- **D-05 (o efeito), D-19, D-24, D-25 e o sintoma de D-27 sao PROVAVEL.** Em D-05 o efeito em shader real nao tem caso de titulo citado; em D-19 a cadeia e inferida por ordem temporal (ramos paralelos, sem referencia cruzada); em D-24/D-25 o diff prova o conserto mas nao ha log de falha no repositorio; em D-27 a descricao do comportamento pre-fix vem da doc escrita no proprio commit de correcao.
- **Fuso horario.** Os dias do projeto sao 2026-11-21 a 2026-11-25, mas os commits usam fusos diferentes (+0100, +0300, -0300, +0800). Comparacoes de "minutos depois" (D-19, D-26, D-29) foram feitas sobre o timestamp absoluto do commit, nao sobre a data local do autor. `1f24626b` -> `22989d61` da 88 s em qualquer fuso.
- **O indice da secao K e gerado por `git log --reverse --format='%h|%ad|%s' --date=short`,** sem autoria, para manter o arquivo utilizavel. A atribuicao por autor esta na secao A e no `git log` do clone; nao a repeti em 4076 linhas.
- **Escopo do julgamento.** Classifiquei cada um dos 45 blocos pelo **diff**, como pedido. Onde o diff do pacote vinha vazio (blocos de merge), abri o commit no repositorio; se ainda assim nao houvesse diff (merge fast-forward de conteudo identico), a classe e `c` por ausencia de efeito, nao por ausencia de investigacao.
