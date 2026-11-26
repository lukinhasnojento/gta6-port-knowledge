# Catalogo de defeitos - 305 entradas

Gerado a partir de `defeitos.csv`. O detalhamento completo de cada defeito esta em `docs/analises/<repo>.md`, secao D.

## VIctory (27)

| ID | Titulo | Confianca | Introduzido em | Corrigido em |
|---|---|---|---|---|
| D-01 | · Esgotamento de slots dinamicos de descritor fazia o draw ler o descriptor do titulo · remocao de device (#59... | - | `2849477` | `edf64e2` |
| D-02 | · Heap de views limitado pelo host (teto de 1.000.000 da NVIDIA) · slot de tabela do titulo aliasava slot dina... | - | `2849477` | `3827767` |
| D-03 | · Descritores nulos do tipo/dimensao errados · remocao de device segundos depois de entrar no mundo | - | `2849477` | - |
| D-04 | · Imagem preparada da DLL: ponteiro de callbacks TLS zerado mas realocacao mantida; ida e volta no diretorio d... | - | `2849477` | `25a3e95` |
| D-05 | · Caminho narrow reescrito duas vezes · toda abertura narrow de arquivo do titulo falhava | - | `754619f` | `27b530e` |
| D-06 | · Descritor lido de `v_readfirstlane_b32` tratado como "o de cada lane" em qualquer lugar · amostragem errada | - | `ffa6635` | `b1c8306` |
| D-07 | · Copy-back sobrescrevia o que a CPU do titulo tinha escrito durante a submissao · crash do titulo | - | `2849477` | `35b054e` |
| D-08 | · Copia de textura escrita pela CPU nao era enviada ao host antes de `CopyResource` / `CopyTextureRegion` | - | `2849477` | `6db02ea` |
| D-09 | · Pacote XMA danificado com frame de comprimento 0 · laco infinito (`next_frame`) | - | `2849477` | `005ad49` |
| D-10 | · `ADVANCE_XMA_READ` somava antes de reduzir · overflow de u32 (panic em debug) e bloco errado | - | `2849477` | `257d8bb` |
| D-11 | · Aritmetica de layout de textura/XMA estourava com valores do titulo (4 sitios + offsets) | - | `2849477` | `e158d95` |
| D-12 | · `XMemMapPhysicalPages` com contagem 0 · panic de `BTreeMap` dentro de `extern "system"` · abort do processo | - | `2849477` | `02fcf72` |
| D-13 | · `XGameSave` com configuration id vazio · conversao de containers apagava os saves de todas as configuracoes ... | - | `2849477` | `c88b7ac` |
| D-14 | · Codigos de erro do HLE divergentes do Xbox (`XGameSave`, `storage`) | - | `2849477` | `f0894c2` |
| D-15 | · `XTaskQueueDispatch(queue, WORK, INFINITE)` nao terminava quando a fila era terminada · thread e join travad... | - | `754619f` | `36d6a79` |
| D-16 | · `XAsync` tratava so `S_OK` como sucesso · payload de `S_FALSE` nunca voltava e a chamada ficava registrada p... | - | `754619f` | `3e39eac` |
| D-17 | · `#[cfg(test)] mod tests` duplicado em `victory-gpu/src/host.rs` · `cargo test --workspace` nao compila (E042... | - | `73d776b` | `7102a49` |
| D-18 | · `lowers()` mudou de aridade em um PR e o chamador antigo ficou · build de testes do `main` quebrado | - | `25b0f88` | `8e79c80` |
| D-19 | · `match` nao exaustivo em `apps/victory/src/translate.rs` apos novos `Source` · `cargo check -p victory` falh... | - | `ffa6635` | `077ec8b` |
| D-20 | · `v_cmp_lg` baixado como `fcmp one` · verdadeiro para NaN no driver AMD | - | `2849477` | `51f1627` |
| D-21 | · `min`/`max` nao ordenavam `-0` abaixo de `+0` · sinal errado propagado (ex.: `rsqrt`) | - | `2849477` | `bf84134` |
| D-22 | · `f32_to_half` somava o carry de arredondamento no expoente com `or` · valor caia para a potencia anterior | - | `2849477` | `63df363` |
| D-23 | · `fract()` corrigido em `36eb0b4` quebrou em NVIDIA (dobrado para `minNum`, engolia NaN) · regressao conserta... | - | `36eb0b4` | `2e5001c` |
| D-24 | · `s_addc_u32`/`s_subb_u32` nao devolviam carry · SCC ausente · cadeia de 64 bits nao traduzia | - | `2849477` | `a2f5f43` |
| D-25 | · Testes VALU de GPU travavam `cargo test` (dispatches nunca completavam) | - | - | `1106f43` |
| D-26 | · Offset de `s_branch`/far jump lido como unsigned · saltos para tras nao decodificavam | - | `12caefe` | `c963b71` |
| D-27 | · `s_ttracedata_imm` com opcode SOPP errado (0x26 em vez de 0x28) · shader com marcador de trace nao traduzia | CONFIRMADA | `12caefe` | `79fb722` |

## XCL (24)

| ID | Titulo | Confianca | Introduzido em | Corrigido em |
|---|---|---|---|---|
| D-01 | Janela de memoria UMA escolhida pelo dword alto da base, sem carry/borrow de 32 bits | - | `f3b15b9` | `f9e5ee9` |
| D-02 | Regressao: o codegen de #123 derrubou o compilador de shader da AMD (amdxc64) | - | `a787c52` | `70f8dc4` |
| D-03 | Guardas da forma estruturada crescem com o quadrado dos flags: 148 MB de HLSL e OOM no DXC | - | `b8b1f49` | `69a892c` |
| D-04 | No backend per-lane, carry-ins e `v_cndmask` liam sempre o bit 0 da mascara | - | `77db49c` | `188ee17` |
| D-05 | Typo de campo (`OPSEL` em vez de `OP_SEL`): o modificador `op_sel` do VOP3 nunca era recusado | - | `b3c0b37` | `ac34cb6` |
| D-06 | Produtos f16 do tradutor direto arredondam para zero em vez de para o par mais proximo | - | `746f7dd` | - |
| D-07 | `V_CVT_F16_F32` do tradutor direto devolve 0 abaixo do menor denormal f16 | - | `746f7dd` | - |
| D-08 | O caminho HLSL perde `-0.0` no driver AMD e o runtime compila sem `-Gis` | - | - | - |
| D-09 | CSE block-local reaproveitava `dx.op.wave*`/`dx.op.quad*` atraves de `dx.op.discard` | - | `223311f` | `50228bb` |
| D-10 | Panic de underflow em `s_setreg_b32` de `HW_REG_MODE` bit 3 (tradutor direto) | - | `746f7dd` | `ecb7547` |
| D-11 | Entradas de cache do per-lane nao incluiam as fontes do lowering: DXIL obsoleto servido do disco | - | `ddfbf39` | `bfb8133` |
| D-12 | A analise per-lane tratava `s_cbranch_vccz` como branch por lane e ignorava escritas de MODE | - | `1bab757` | `66879ef` |
| D-13 | SCC de uma operacao de mascara carregado em volta de um loop virava valor uniforme | - | `1bab757` | `9dcf2f6` |
| D-14 | `v_cmpx` float via bridge descartava o EXEC anterior | - | `372be9d` | `fe78c81` |
| D-15 | `stale()`: valor copiado num bloco parcial mantinha o bloco de origem | - | `b3c0b37` | `a23b6ff` |
| D-16 | Mascara wave64 obsoleta reportada so na metade baixa | - | `b3c0b37` | `13ac196` |
| D-17 | Mascaras inline/NULL recusadas no bridge do per-lane | - | `878afda` | `15fff3b` |
| D-18 | `s_wqm` nao era seguido: todo PS mantinha guarda em toda instrucao vetorial | - | `d2cc6ff` | `27d28ce` |
| D-19 | Geometry na forma split: a fase ES caia dentro do codigo do GS e os records saiam vazios | - | `a21d3df` | `f3242b2` |
| D-20 | ExecuteIndirect lia argumentos indiretos antes da barreira que os escreve (NVIDIA) | - | - | `4e3515e` |
| D-21 | `ExecuteIndirect` de um pipeline de compute nao traduzido rodava com a root signature errada | - | - | `6c89285` |
| D-22 | `xcl shaders`: `--export-hlsl` pulava registros nao-compute em silencio; flags sem valor eram aceitas | - | - | `4143a09` |
| D-23 | Testes GPU falhavam (em vez de pular) em adaptadores sem waves de 32 lanes; e o teste de IR falhava sem `dxil.... | - | - | `9e27370` |
| D-24 | Ordem `dx.op.getDimensions` -> `dx.op.textureLoad` em Texture2DMS derrubava o compilador da AMD | - | `2dcc926` | `425b869` |

## Xbox2Windows (30)

| ID | Titulo | Confianca | Introduzido em | Corrigido em |
|---|---|---|---|---|
| D-01 | ** Superficie apresentada liberada durante a copia da assinatura | CONFIRMADA | `8c2d82f6` | `2a898640` |
| D-02 | ** Split de placeholder atinge placeholder do chamador | CONFIRMADA | `0478a386` | `21e8f5c0` |
| D-03 | ** Rollback do commit de slab deixa slab aberto | CONFIRMADA | `9727c306` | `21e8f5c0` |
| D-04 | ** Container de save aceita update depois de fechado | CONFIRMADA | `ca92741e` | `2260b7a4` |
| D-05 | ** Loads pendentes nao unidos nas fronteiras dos bracos mascarados | CONFIRMADA | `6b8ad31c` | `05ef4d92` |
| D-06 | ** f32 `nlt/nle/ngt/nge/o/u` errados sob FXC e DXC sem `-Gis` | CONFIRMADA | `e658fe24` | `583178d4` |
| D-07 | ** Offset imediato SMEM impresso sem sinal | CONFIRMADA | `eefc3f86` | `83bb752d` |
| D-08 | ** CLI do `disasm_rdna2` ignora `--output` com `--hex`, vaza traceback e sai 0 em alvo ausente | CONFIRMADA | - | `20fa94a9` |
| D-09 | ** `startup_view_heap` le o trap log sem compartilhamento | CONFIRMADA | `99f097ad` | `2c1dcdd6` |
| D-10 | ** Contador de "primeiras N chamadas" estoura 2^31 e volta a logar | CONFIRMADA | `e658fe24` | `f360d30a` |
| D-11 | ** Testes de hardware sem `SKIP_RETURN_CODE 77` | CONFIRMADA | - | `0b649c69` |
| D-12 | ** `XApuDequeueResult` em fila vazia retorna `E_FAIL` | CONFIRMADA | - | `b3a90888` |
| D-13 | ** Endpoint de triagem conta first-chance step-over como "debugger stop" | CONFIRMADA | `ef9a0798` | `313803bc` |
| D-14 | ** Call hooks do debugger armados por padrao | CONFIRMADA | - | `ac539650` |
| D-15 | ** `CreateCommandSignature` sequestra qualquer tipo acima de `DISPATCH_MESH` e le ponteiros sem validar | CONFIRMADA | `71d4bff1` | `e50a72bc` |
| D-16 | ** Fixture MASM isolado trocado por programa de pseudo-ops | CONFIRMADA | `5b2dadd7` | `14d6023e` |
| D-17 | ** Caso de heap em placed-buffer assumia prefixo reservado | CONFIRMADA | - | `6ef541f3` |
| D-18 | ** Compilacao de shader nao retomada depois de rodada manifest-only | CONFIRMADA | - | `b87a99a3` |
| D-19 | ** Documentacao do ciclo de `.fail` ficou obsoleta | PROVAVEL | `b87a99a3` | `5b29fe0f` |
| D-20 | ** Tabela `SHADER_REASONS` nunca exercitada pela suite | CONFIRMADA | `71ec5842` | `13745b1b` |
| D-21 | ** Checagem device-free das selecoes SDWA `movrel` nao detectava selecao removida | CONFIRMADA | `bc03f191` | `2e5fdca4` |
| D-22 | ** `pe_inventory` quebra com inventario malformado e nao valida a CLI | CONFIRMADA | - | `9a2b9cd8` |
| D-23 | ** Draw `attr0.z/w` falhava em host sem debug layer | CONFIRMADA | - | `d28732b6` |
| D-24 | ** `<initializer_list>` ausente no teste de tipo de heap | PROVAVEL | - | `5d6d5f52` |
| D-25 | ** `seg_inspect_check` decodifica stderr do compilador sem `errors="replace"` | PROVAVEL | - | `6855f425` |
| D-26 | ** Viewport/scissor default confundido com o estado do titulo (defeito introduzido e corrigido no mesmo dia, e... | CONFIRMADA | `ffb70f6a` | `07f2464f` |
| D-27 | ** Coordenadas 2D `a16` de `image_sample`/`image_sample_lz` lidas como dois registradores binary32 | CONFIRMADA | - | `c3c3ab3c` |
| D-28 | ** Conversao UNORM/SNORM do store formatado usava `round()` (ties-to-even) em vez da regra do Direct3D | CONFIRMADA | `6177df41` | `f37fe7b8` |
| D-29 | ** Mutacao temporaria commitada em `main` (e revertida 88 s depois) | CONFIRMADA | `1f24626b` | `22989d61` |
| D-05 | (o efeito), D-19, D-24, D-25 e o sintoma de D-27 sao PROVAVEL.** Em D-05 o efeito em shader real nao tem caso ... | - | - | - |

## OpenScarlett (24)

| ID | Titulo | Confianca | Introduzido em | Corrigido em |
|---|---|---|---|---|
| D-01 | ** DirectStorage le o pedido do titulo no layout de **desktop**, nao no layout Xbox | CONFIRMADA | `a429535` | `76adeff` |
| D-02 | ** `GameInputGamepadButtons` deslocado em dois bits | CONFIRMADA | `a429535` | `3112404` |
| D-03 | ** `XMemGetWorkingSetStatistics` com assinatura de um argumento e deteccao por `VirtualQuery` | CONFIRMADA | `669f0b9` | `675d9f4` |
| D-04 | ** O duplo de teste de XAudio2 declarava uma vtable de 8 slots onde a real tem 19 | CONFIRMADA | `59bd45a` | `3cc1984` |
| D-05 | ** A chave do cache de shader em disco nao contem a versao do tradutor | CONFIRMADA | `538e904` | - |
| D-06 | ** O gerador de status le o ultimo `LastTest.log` e publica "6/6 testes" | CONFIRMADA | - | - |
| D-07 | ** As familias do dashboard somam 1.211 "feitos" contra 779 "totais", e o numero de opcodes verificados e um l... | CONFIRMADA | `7ba25f7` | - |
| D-08 | ** O cabecalho escrito a mao do README diverge do bloco auto-gerado | CONFIRMADA | - | - |
| D-09 | ** Teste que nao pode rodar reporta `Passed`, e nao existe `SKIP_RETURN_CODE` em lugar nenhum | CONFIRMADA | - | - |
| D-10 | ** `PresentX` degrada para headless mascarando falha de criacao de swapchain (e a mensagem fala de outra coisa... | CONFIRMADA | `d7f1b7d` | - |
| D-11 | ** Um merge derrubou o wrap do `ScarlettRetailDeviceProxy` e ele teve de ser restaurado | CONFIRMADA | `cbfd4f1` | `f2c0610` |
| D-12 | ** Espera sincrona de fence no caminho de submissao (quatro paradas de 50 ms por frame) | CONFIRMADA | - | `022a8b2` |
| D-13 | ** Footprint nao alinhado do `CopyTextureRegion` do Xbox era passado direto ao D3D12 de desktop | CONFIRMADA | `d58f0b6` | `f7566aa` |
| D-14 | ** Listas com `Close()` falho eram descartadas sem honrar as escritas de CPU diferidas | CONFIRMADA | `f7566aa` | `022a8b2` |
| D-15 | ** O orcamento de residencia era cumprido evictando heaps vivos | CONFIRMADA | `a92359c` | `d71ed46` |
| D-16 | ** A regra de passthrough de caminho de host foi aplicada API a API, nao como regra | CONFIRMADA | `990c3ff` | - |
| D-17 | ** `E:\OpenScarlett` compilado no produto e testes que exigem "fora de `C:`" | CONFIRMADA | `7e2dd9b` | `438fdb0` |
| D-18 | ** Habilitacao de `ctest --parallel 4` quebrou o benchmark de wall-clock | CONFIRMADA | `7e2dd9b` | `1ffd939` |
| D-19 | ** Overflow de microssegundos do QPC congela timestamps depois de ~10,67 dias de uptime | CONFIRMADA | `a429535` | `05a24bc` |
| D-20 | ** Topologia primitiva console-only entregue ao runtime de desktop trava o pushbuffer | CONFIRMADA | `d58f0b6` | `0cbc742` |
| D-21 | ** `ds_sub` em LDS emitido como `InterlockedAdd` de valor negado — driver NVIDIA recusa o pipeline | CONFIRMADA | - | `5d3809b` |
| D-22 | ** `notify_one`/`notify_all` fora do lock no DirectStorage — use-after-free | CONFIRMADA | `a429535` | `a208562` |
| D-23 | ** Mutex da telemetria mantido durante a chamada para a hook DLL | CONFIRMADA | `acbdb7c` | `2b42f72` |
| D-24 | ** `XApuGetResultCount` devolvia a contagem em `EAX` e travava a inicializacao | CONFIRMADA | - | `ef763eb` |

## Belobug-AMD (17)

| ID | Titulo | Confianca | Introduzido em | Corrigido em |
|---|---|---|---|---|
| D-01 | Folhas magenta e blocos de cor em interiores (luz indireta ausente) | - | - | `4f45ab8` |
| D-02 | Linhas em todas as arestas de triangulo (pintura de carro, vidro, pele) | - | - | `03629f3` |
| D-03 | 225 ms de GPU por frame em 9.300 dispatches de clamp | - | - | `ccab973` |
| D-04 | Flicker de objetos mesh (regressao do hoist de clamps) | - | - | `76387c2` |
| D-05 | Esgotamento de slots dinamicos de descritor: draws dropados, frames de segundos | - | - | - |
| D-06 | Compilador AMD: pipelines que nunca terminam ou crasham, 16 GB em dois minutos | - | - | - |
| D-07 | Uso-apos-liberar na GPU (texpool / Staging / HostHeap) -> device removed | - | - | `a873b24` |
| D-08 | Tempestade de faults de 2,5 GB por frame no `watch::guard` | - | - | `2b6adbb` |
| D-09 | Run de blocos guardados que cruza views falhava inteiro e deixava blocos read-only | - | `a873b24` | `e7e6c00` |
| D-10 | Escrita assincrona do kernel em bloco re-guardado | - | - | `579a8d9` |
| D-11 | Partes de veiculos em movimento ficavam para tras | - | - | `ae90937` |
| D-12 | Crash/GPU hang 120 frames dentro do mundo ao devolver tiles de copia | - | - | - |
| D-13 | Consumo de memoria: 22 GB de RSS, 33 GB de commit, mais VRAM que a placa | - | - | - |
| D-14 | Telefone in-game deixa o mundo sem luz | - | - | - |
| D-15 | Agua desaparece ate reiniciar depois de reconstruir o cache de shaders | - | - | - |
| D-16 | Apenas o material 0 iluminado (quadrados) e depois imagem branca estourada | - | - | `7eb9545` |
| D-17 | `VIEW_SLOTS` (2.294.784) acima do limite tier-2 do D3D12 (1.000.000): o padding nunca e criado | CONFIRMADA | `0a32e89` | - |

## DexNom (21)

| ID | Titulo | Confianca | Introduzido em | Corrigido em |
|---|---|---|---|---|
| D-01 | Alvo de renderizacao inteiro com blending ligado derruba a criacao do pipeline | - | `570a941` | `ddb7332` |
| D-02 | Sampler estatico escolhido por desvio uniforme: "an untracked sampler" | - | `2849477` | `ddb7332` |
| D-03 | Mascara wave64 restaurada por dois `s_mov_b32` nao era reconhecida | - | `2849477` | `76749ce` |
| D-04 | LOD calculado com logaritmo natural vezes log2(e), e clamp descartado | - | `570a941` | `921d8d3` |
| D-05 | `USER_MEMORY_BASE` respondia a base do buffer para as duas dwords; shader lia descritores fora do bind | - | `570a941` | `921d8d3` |
| D-06 | Textura com apenas mips intermediarios residentes era considerada ausente | - | `570a941` | `921d8d3` |
| D-07 | Operacoes de quad em pixel shader rebaixadas por leitura de lane da wave inteira | - | `2849477` | `921d8d3` |
| D-08 | BVH: slots vazios entrando na ordenacao, grupo de 4 dwords cruzando a metade do no, e pagina ausente lida como... | - | `570a941` | `bc3cd4b` |
| D-09 | Toda janela de memoria mapeada era declarada "escrita": 12,8 GB comparados por frame | - | `570a941` | `bc3cd4b` |
| D-10 | `Staging::detile` escreve alem do buffer mapeado (UB) - **corrigido e depois reintroduzido** | - | `2849477` | - |
| D-11 | Corrida na escrita atomica da pipeline library (`.tmp` de nome fixo) | - | `35b054e` | - |
| D-12 | `clwb_loop`: overflow de u64 no calculo do `base` final | - | `c4e915f` | - |
| D-13 | Testes novos de material demand: recurso posicionado fora da reserva e corrida com o relogio global de frames | - | `b8ff22e` | `2e01ab4` |
| D-14 | Tabelas BCPack fora do repositorio: `build.rs` removido e verificacao de hash dependente do host | - | `570a941` | `8c7937a` |
| D-15 | Deteccao de "hardware" nos testes de shader: WARP passava por GPU | - | - | `ccfb029` |
| D-16 | Falsa exaustao de descritores dinamicos quando o run livre cruza o cursor | - | - | `c1c3281` |
| D-10 | , D-11, D-12 estao **abertos** por reversao. Nao encontrei nenhum commit posterior que os reaplique, e nao ha,... | - | - | - |
| D-10 | `subresource_len` \| `4fc9d70` / `711e930` \| a linha `let len = pitch * rows as usize * f.Depth as usize;` es... | - | - | - |
| D-11 | escrita atomica \| `be824d9` / `a9d233f` \| `write_atomically` tem `path.with_extension("tmp")` e nenhum mutex... | - | - | - |
| D-12 | overflow do `clwb_loop` \| `274f698` / `17de445` \| expressao original presente; `clwb_loop_end` ausente \| | - | - | - |
| D-15 | skip de hardware \| `ccfb029` \| `synthetic_tests.rs` **apagado** por `570a941` \| | - | `570a941` | - |

## VItoriaBr (21)

| ID | Titulo | Confianca | Introduzido em | Corrigido em |
|---|---|---|---|---|
| D-01 | — Caminhada `VirtualQuery` em `mapped_bytes` derruba o frame para 752 ms | - | - | `c931d12` |
| D-02 | — Corrida no `relay_land`: ranges liberados antes de `line.written` + `notify_all` -> 0xc0000005 | - | - | `aa85bb0` |
| D-03 | — XMA: laco infinito com contagem de bits alem dos dados | - | - | `bfa5f49` |
| D-04 | — Varredura de wraparound dos descritores dinamicos truncada: exaustao falsa | - | - | `009856f` |
| D-05 | — Heap de descriptors esgotado lia o descritor do titulo como CBV -> remocao do device na AMD | - | - | `3c81420` |
| D-06 | — `SHADER_HEAPS`: use-after-free no espelho e, depois, 48,5 ms/frame de contencao | - | - | `a253706` |
| D-07 | — Spans de texturas sobrepostas: escrever uma view anulava os slots das vizinhas | - | - | `310c6eb` |
| D-08 | — Textura indexada subia zerada: pele preta e cabelo claro em 3 de 4 execucoes | - | - | `a803c87` |
| D-09 | — Paredes ausentes na primeira sala: escritas por outra view da mesma pagina fisica | - | - | `8dd30ab` |
| D-10 | — Janelas de draw indireto derrubavam o device (3 vezes) | - | - | `f85101b` |
| D-11 | — Depth bounds test nao traduzido: draws passavam em toda profundidade | - | - | `c96b4f2` |
| D-12 | — LDS fora de faixa: trap na NVIDIA e, depois, limite errado (usava o total do host) | - | - | `40beb26` |
| D-13 | — `SLOT_VIEWS`: tabela de 16 bytes lida com stride de 8 | - | - | `5603b45` |
| D-14 | — `v_permlane16`/`v_permlanex16` nao pediam as lanes do console | - | - | `448416c` |
| D-15 | — `v_dot8_i32_i4` ausente da tabela de opcodes | - | - | `75aa531` |
| D-16 | — MIMG: variantes ausentes e LOD com conversao de log2 dupla | - | - | `6dcd8cd` |
| D-17 | — `d16` em buffer store reinterpreta inteiros de 16 bits como `f16` (**SEM CORRECAO**) | - | - | - |
| D-18 | — XGameSave podia apagar o save commitado | - | - | `d1da85a` |
| D-19 | — Deadlock do loader lock no cache de modulos HLE | - | - | `708b450` |
| D-20 | — Device loss introduzido pela atualizacao wave64 (barreiras dentro de branches/loops) | - | - | `e9765a7` |
| D-21 | — Deriva de documentacao: numeros retirados, contradicao sobre TDR, contagens divergentes | - | `0603123` | `b37344f` |

## AvatarManz (19)

| ID | Titulo | Confianca | Introduzido em | Corrigido em |
|---|---|---|---|---|
| D-01 | Orcamento indireto esgotado por um registro lixo derruba todos os draws seguintes da lista | - | - | - |
| D-02 | Corte em 65.535 grupos perdia a ultima faixa de tiles da imagem | - | - | `84eae35` |
| D-03 | `VICTORY_SEND_CHANGED` ligado por padrao fez a sala perder luz indireta e explodir a geometria | - | - | - |
| D-04 | 16 threads de mapeamento por padrao pioraram o dormitorio de 125 para 260 ms | - | - | `4e99544` |
| D-05 | Tamanho de estrutura BVH lido antes de a submissao pousar: copias sobrescreviam vizinhos | - | - | `2c20d7a` |
| D-06 | Corrida entre mapear uma view e abrir os guards das outras views da mesma pagina | - | - | `689df31` |
| D-07 | Passes sobre a tela perdiam pipelines e a cena ficava sem luz | - | - | `a989668` |
| D-08 | Compilacao de pipelines travava o titulo por dezenas de segundos | - | - | - |
| D-09 | Device loss a cada ~16 min: textura mostrada antes de o upload rodar | - | - | - |
| D-10 | Correcao do replay feita e revertida sem explicacao (buffers em heaps abertos da memoria) | - | - | - |
| D-11 | Helper lanes nao participam das operacoes de wave na AMD: buracos quadrados e bordas tracejadas nas sombras | - | - | `9fea743` |
| D-12 | Contagem de dispatch de ray tracing tomada como grupos em vez de threads: toda ray errava | - | - | `810efe5` |
| D-13 | Leitura raw atraves de view tipada nao tem fim na AMD: dois device losses no meshlet culling | - | - | `1f58d91` |
| D-14 | Indice de view alem do heap: a RDNA 3 remove o dispositivo | - | - | `31714de` |
| D-15 | O mesmo patch commitado duas vezes, com bases diferentes (duplicatas de cherry-pick) | - | - | - |
| D-16 | Duas convencoes opostas para o mesmo tipo de switch, em 294 variaveis | - | - | - |
| D-17 | README nega codigo de XCL; dois arquivos declaram port de XCL (GPL-2.0-only) num repo GPL-3.0-or-later | - | - | - |
| D-18 | Constantes de layout fixas por titulo (slots de descritor e formatos de textura) | - | - | - |
| D-19 | A configuracao documentada nao e a configuracao medida: `play.ps1` liga ~20 switches | - | `bddfb4a5` | - |

## victory-fork (17)

| ID | Titulo | Confianca | Introduzido em | Corrigido em |
|---|---|---|---|---|
| D-01 | Forma wave64 "wide" derruba o device no inicio da execucao | - | - | `e9765a7` |
| D-02 | Chave do cache de shader nao incluia os switches de wave (traducoes envenenadas) | - | - | `e9765a7` |
| D-03 | Janelas de indirect draw serviam registros de lixo (2^24 vertices) e derrubavam a placa | - | - | `f85101b` |
| D-04 | Reversao e reaplicacao da mudanca de lane-mask (#11) em um minuto | - | - | `2119b0e` |
| D-05 | Regra de staleness por sobreposicao total anulava slots de texturas vizinhas (paredes ausentes) | - | - | `310c6eb` |
| D-06 | Textura escolhida por indice era enviada antes de o titulo preencher a memoria (rosto preto, cabelo sem cor) | - | - | `a803c87` |
| D-07 | Registros de descritor copiados como bytes nao eram espelhados (40.000 por frame com slot nulo/velho) | - | - | `9376fd6` |
| D-08 | Indice de view alem do fim do heap: hang na NVIDIA, remocao do device na AMD | - | - | `31714de` |
| D-09 | Copia in-place tomava as sondagens de 1 byte da camada (1.000-1.300 faults por frame) | - | - | `0b0074e` |
| D-10 | Copia de textura sobre ela mesma (auto-copia proibida pelo host) | - | - | `a253706` |
| D-11 | Texturas escritas por compute nao chegavam a CPU do titulo (o barco afundava) | - | - | `f5c03f3` |
| D-12 | Loads escalares agrupados de 4 dwords leem alem do fim da view (agua preta) | - | - | - |
| D-13 | O "dip" periodico de frame era o proprio harness de captura (235 ms por frame salvo) | - | - | `a253706` |
| D-14 | Alegacao de ganho de 6,0 -> 8,9 fps comparava cenas diferentes | - | - | `0b0074e` |
| D-15 | Page mapping do titulo: duas tentativas que quebraram o streaming de textura e foram revertidas | - | - | - |
| D-16 | Perdas de device sem causa identificada (runs 204-279 e mundo aberto) | - | - | - |
| D-17 | Divida de verificacao: "this merged tree was built; its test suites were not run" | - | - | `e9765a7` |

## xbox-emulator (24)

| ID | Titulo | Confianca | Introduzido em | Corrigido em |
|---|---|---|---|---|
| D-01 | — Elemento de StructuredBuffer dimensionado por heuristica do ISA | CONFIRMADA | - | `b03b39b` |
| D-02 | — Raw buffer declarado como `StructuredBuffer` com elemento de 16 bytes | CONFIRMADA | - | `3e9676d` |
| D-03 | — Base de user-data deduzida do estagio da API (VS s8), errada em merged ES | CONFIRMADA | - | `4fcfe13` |
| D-04 | — `v_mad_i64_i32` escrevia o flag de overflow em `sdst`, nao o bit de sinal | CONFIRMADA | - | `7364bb4` |
| D-05 | — M0 (124) e SGPR_NULL (125) trocados no decode escalar | CONFIRMADA | - | `9b7c93a` |
| D-06 | — Operacoes SOP1 com resultado de 32 bits de fonte de 64 bits escreviam dois SGPRs | CONFIRMADA | - | `8f71091` |
| D-07 | — `DispatchMesh` lia Z do dword do ponteiro de payload: cada grupo de malha lancado duas vezes | CONFIRMADA | - | `53bb998` |
| D-08 | — Escritas byte/short no payload de amplificacao sumiam em hardware NVIDIA | CONFIRMADA | - | `d2cc33e` |
| D-09 | — AS escolhia o ABI de root por *memory* ou *register* conforme a posicao do primeiro `s_load` | CONFIRMADA | - | `3d90f6f` |
| D-10 | — O compilador AMD dobra expressoes que a ISA nao dobra (sinal de zero, Inf-Inf, trunc f64) | CONFIRMADA | - | `65893a4` |
| D-11 | — `cl_edition`: leitura de um byte alem da janela de 8 bytes no scan de `call` com SIB | CONFIRMADA | - | `68f2cf2` |
| D-12 | — Build limpo quebrado: `mod oracle` declarado sem arquivo; CI cego ao problema | CONFIRMADA | - | `706a2b7` |
| D-13 | — Flags de fila so-do-console repassadas ao D3D12 do PC: `E_INVALIDARG` | CONFIRMADA | - | `979615b` |
| D-14 | — Blendclamp "eager" prendia alvos `R32G32B32A32_FLOAT` a ±65504 | CONFIRMADA | - | `ced7994` |
| D-15 | — Remocao de device D3D12 era interpretada como fence concluida | CONFIRMADA | - | `d550bc0` |
| D-16 | — `GameInputCreate` falhando devolvia ponteiro nulo para um titulo que nao checa HRESULT | CONFIRMADA | - | `146286e` |
| D-17 | — `XGameGetXboxTitleId` com codigo e ordem de checagem diferentes do runtime do PC | CONFIRMADA | - | `c3292cc` |
| D-18 | — `XAsyncCancel`/`XAsyncGetResult` divergindo do `xgameruntime.dll` | CONFIRMADA | - | `e7377c9` |
| D-19 | — `XMemVirtualAlloc` descartava `MEM_RESERVE_PLACEHOLDER` / `MEM_REPLACE_PLACEHOLDER` | CONFIRMADA | - | `2e8e2b8` |
| D-20 | — `console-sandbox::fs::resolve` panicava em caminho com caractere multi-byte | CONFIRMADA | - | `0284e83` |
| D-21 | — `tombox-archive::validate` aceitava entradas com nomes colidentes | CONFIRMADA | - | `cce78b6` |
| D-22 | — Teste "flaky" era sintoma de comportamento assincrono real do Windows (e o problema de runtime ficou aberto) | CONFIRMADA | - | `ab6d485` |
| D-20 | a mensagem de `0284e83` fala em "drive-letter escape"; o diff **so** mostra a correcao de fronteira de caracte... | - | - | - |
| D-08 | a segunda mudanca no caminho (`9101363`, "Prototype staging atomic amplification payload writes", 2026-11-24) ... | - | - | - |

## TuxBox (29)

| ID | Titulo | Confianca | Introduzido em | Corrigido em |
|---|---|---|---|---|
| D-01 | Estouro de pilha dentro do vkd3d-proton ao criar pipelines | - | - | `c679f13` |
| D-02 | Otimizador do DXC estoura o tempo; shaders viram pipeline nulo | - | - | `e9c8fb6` |
| D-03 | Conversao DXIL->SPIR-V leva minutos em `cs_6_6` compilado em `-O0` | - | - | `e9c8fb6` |
| D-04 | Cache de pipeline da NVIDIA no lugar errado e com teto de tamanho | - | `9b18f7c` | `d10b369` |
| D-05 | Varredura O(n) case-insensitive do Wine na cache de shaders | - | - | `4b4c08c` |
| D-06 | Present unico nao chega a tela no GE-Proton11-14 | - | - | `1b0dfaa` |
| D-07 | Amostragem de tela falha em XWayland rootless (`BadMatch`) | - | - | `00200ba` |
| D-08 | Pacote DXC da distribuicao (Arch) aborta em todo shader | - | - | `baa315b` |
| D-09 | Toolchain Linux: `synchronization.lib` ausente e `clang 19` com `/W4 /WX` | - | - | - |
| D-10 | Jogo nao abre arquivos do pacote sem o drive `G:` | - | - | `c561d03` |
| D-11 | Desligar `VK_EXT_descriptor_buffer` em todas as GPUs corrompe heap no RADV | - | - | `72766e7` |
| D-12 | Stride de descritor inline incompativel com o RADV (32 bytes) | - | - | `cf427e8` |
| D-13 | Slot 62 interpretado como upload de CPU em vez de `CopyMemoryToMemoryX(dst, src)` | - | `3086e0b` | `97b8ad3` |
| D-14 | Tabela de slots da lista Scarlett deslocada (61/62/131/132/133) | - | - | - |
| D-15 | Escritas de fence do titulo perdidas e um comando derrubando a lista inteira | - | `4ae23ab` | `e8af9fd` |
| D-16 | Deadlock entre filas por espera de placement atras de um label | - | - | `faf8b19` |
| D-17 | Liberacao imediata do backing de descritor causa falha de MMU na GPU | - | - | `98043df` |
| D-18 | Aliasing de textura posicionada derrubando a fila, e o contorno que matou o watchdog | - | - | `524cd21` |
| D-19 | Asserts de tile-heap remendados por RVA e bytes | - | - | - |
| D-20 | Instrucoes exclusivas de AMD em CPUs Intel | - | - | - |
| D-21 | Assinatura mesh/pixel incompativel derruba o draw | - | - | `315b287` |
| D-22 | `Failed to remap` (-3) em roots raster com varias tabelas de descritor | - | - | `95b242b` |
| D-23 | `clang 19` vetoriza errado o clear RGB10 e a sonda de present escreve em ponteiro | - | - | `40e1565` |
| D-24 | Resolucao de merge apagou lowering e quebrou o verificador | - | - | `1a50eb9` |
| D-25 | Profundidade 16 bits com stencil sem plano UAV gravavel | - | `fe6b0e1` | `0e00f83` |
| D-26 | `R9G9B9E5_SHAREDEXP` como UAV | - | `fe6b0e1` | `cda1912` |
| D-27 | Slot 80 trata dimensoes em threads como grupos (issue #164, aberta) | - | `18e4e08` | `2edd4a9` |
| D-28 | `WaveSize(N)` incompativel recusado so no driver (issue #151, aberta) | - | - | `6161046` |
| D-29 | Filas de GPU ainda param em esperas de fence no story intro | - | `c679f13` | `2d887b3` |

## SixNG (24)

| ID | Titulo | Confianca | Introduzido em | Corrigido em |
|---|---|---|---|---|
| D-01 | — Recurso RAGE dividido em duas entradas de indice (modelos vazios) | CONFIRMADA | `6b68f75` | `b7fbe3a` |
| D-02 | — TOC do RPF9 nunca lido: nomes e offsets vem de carving (limitacao estrutural) | CONFIRMADA | - | - |
| D-03 | — Normal map iluminado em frame tangente errado e flip duplo de verde | CONFIRMADA | `6b68f75` | `839e852` |
| D-04 | — Modelo antigo exibido sob o nome novo (corrida de carregamento assincrono) | CONFIRMADA | `6b68f75` | - |
| D-05 | — Instancias de prop-batch decodificadas como lixo (posicoes 1e10-1e35, escalas inf) | CONFIRMADA | `6b68f75` | - |
| D-06 | — BC6H decodificado errado por tres bugs da dependencia `texture2ddecoder` | CONFIRMADA | `d7430ce` | - |
| D-07 | — BC6H: estado herdado de bloco one-region para two-region produz speckle | CONFIRMADA | `d7430ce` | - |
| D-08 | — Tabelas de preset BCPack (BC6H) com deslocamento constante: speckle em areas planas | CONFIRMADA | `d7430ce` | - |
| D-09 | — BC1: seis dos nove slots de janela substituidos eram piores que o default | CONFIRMADA | `08e7457` | - |
| D-10 | — Ordem de eixos da posicao escolhida pelos normais em vez do header | CONFIRMADA | `6b68f75` | - |
| D-11 | — Valores de parametro de material lidos no offset errado (quase todos errados) | CONFIRMADA | `6b68f75` | - |
| D-12 | — Carroceria do veiculo aplicada a interior, carcasas de lampada e pecas mecanicas | CONFIRMADA | `6b68f75` | - |
| D-13 | — Tint de prop aplicado a shaders de veiculo: cacamba em faixas de arco-iris | CONFIRMADA | `8e8cb9b` | - |
| D-14 | — Cores de peca do veiculo tiradas da primeira configuracao | CONFIRMADA | `3b50a33` | - |
| D-15 | — Slot de cor do pneu aplicado: pneu inteiro vermelho ou cinza claro | CONFIRMADA | `4c2fe2a` | - |
| D-16 | — Strings de text tables lidas sempre na primeira pagina do blob | CONFIRMADA | `6b68f75` | - |
| D-17 | — Meshlets com contagem de 32 bits lida como u16: moto desenhada em espinhos | CONFIRMADA | `6b68f75` | - |
| D-18 | — Lista de modelos pulando e roubando o foco do teclado | CONFIRMADA | `6b68f75` | - |
| D-19 | — `extract --force` nao reescrevia texturas compartilhadas | CONFIRMADA | `6b68f75` | - |
| D-20 | — Expressao da lista de cores do sticker (defeito de branch, nunca chegou ao master) | CONFIRMADA | - | - |
| D-21 | — Array R16F lido no dobro do tamanho (cada 1 KiB alternado sem uso) | CONFIRMADA | `6b68f75` | - |
| D-22 | — Contagem de mips subcontada pelo descritor (~44,9 mil texturas embaralhadas) | CONFIRMADA | `6b68f75` | - |
| D-23 | — UV das camadas de ground em 4 camadas: 64 por unidade em vez de 128 | CONFIRMADA | `2ef8b8c` | - |
| D-24 | — Cor da configuracao tratada como linear quando e valor de display (pintura escura) | CONFIRMADA | `2f67255` | - |

## GTA6-Asset-Info (28)

| ID | Titulo | Confianca | Introduzido em | Corrigido em |
|---|---|---|---|---|
| D-01 | — O rebrand parou nas paginas de entrada: o repositorio tem duas identidades** | CONFIRMADA | `dd592ba` | - |
| D-02 | — `structure/` e uma arvore orfa: zero links de entrada** | CONFIRMADA | `bad5bc2` | - |
| D-03 | — O rebrand nunca chegou aos metadados do repositorio** | CONFIRMADA | `dd592ba` | - |
| D-04 | — `0a52a67` apagou as unicas metricas de cobertura de arquivo inteiro** | CONFIRMADA | `0a52a67` | - |
| D-05 | — Instrucao de build aponta para um placeholder de URL que nunca existiu** | CONFIRMADA | `dd592ba` | - |
| D-06 | — Header RPF9: `0x24`/`0x28` com significados incompativeis, e tres contagens para o mesmo `levels.rpf`** | CONFIRMADA | `dd592ba` | - |
| D-07 | — `formats/17` declara "planned / not decoded" o que `structure/03` decodificou ate o framing** | CONFIRMADA | `bad5bc2` | - |
| D-08 | — Shader cache: "one compiled cache, no names; not decoded" vs layout por regiao e ISA identificada** | CONFIRMADA | `bad5bc2` | - |
| D-09 | — Familia de tiling de 4 KiB: `SW_4KB_S` ou `SW_4KB_D`?** | - | `bad5bc2` | - |
| D-10 | — Pergunta aberta ja respondida na arvore canonica (tile mode x header)** | CONFIRMADA | `bad5bc2` | - |
| D-11 | — Largura do campo DXGI: `bits 3–10` contra mascara de 7 bits** | - | `dd592ba` | - |
| D-12 | — `PSIN` e big-endian numa arvore e little-endian na outra — e nas duas dentro do mesmo arquivo** | CONFIRMADA | `dd592ba` | - |
| D-13 | — "Geometria e meshlets, nao arrays de vertice" contra o mesh blob documentado** | - | `bad5bc2` | - |
| D-14 | — A descoberta de que a chave do `ttpf` e joaat nao foi propagada** | CONFIRMADA | `dd592ba` | - |
| D-15 | — Duas contagens do mesmo universo de texturas, sem denominador declarado: 357.115 x 352.461** | PROVAVEL | `dd592ba` | - |
| D-16 | — "34 de 51 node flags", mas a tabela nomeia 32** | CONFIRMADA | `dd592ba` | - |
| D-17 | — Total de luzes nao fecha: 774.847 declarado, 774.846 somado** | CONFIRMADA | `dd592ba` | - |
| D-18 | — Coluna "Shaders" dos buckets soma 389; o texto declara 344 (e as instancias somam 1,82 M, nao 1,83 M)** | - | `dd592ba` | - |
| D-19 | — A conta dos archetipos sem nome nao fecha: faltam 56.267** | CONFIRMADA | `dd592ba` | - |
| D-20 | — `priorityLevel = 84799847` nao e o joaat do proprio nome** | - | `dd592ba` | - |
| D-21 | — Type bytes do chunk AWC: a lei vale para 3 de 8 casos diretos** | - | `bad5bc2` | - |
| D-22 | — `structure/00-INDEX` promete rotulo em toda afirmacao e uma secao de conflitos que nao existem** | CONFIRMADA | `bad5bc2` | - |
| D-23 | — `structure/04`: "289 slot values resolve", mas as parcelas somam 333** | - | `bad5bc2` | - |
| D-24 | — `formats/08` afirma que nomes de struct nao seguem joaat; 10 structs do proprio corpus provam o contrario** | - | `dd592ba` | - |
| D-25 | — `README` promete que "toda pagina" termina em open questions e da confianca por campo; `formats/17` nao faz ... | CONFIRMADA | `dd592ba` | - |
| D-26 | — `formats/13`: as tres faixas de confianca somam 98,2 %, nao 98,3 %** | PROVAVEL | `dd592ba` | - |
| D-27 | — Ancora do README apontava para um slug inexistente (unico defeito corrigido do historico)** | CONFIRMADA | `0a52a67` | `4b3beb1` |
| D-28 | — `formats/07`: o modo de eixo 0 e um espelhamento, mas o paragrafo seguinte jura que so existem permutacoes c... | - | `dd592ba` | - |
