# avatarmanz/VIctory-7900XTX — atualização de 2026-11-26 23:00

## Resumo

A faixa pendente desde `a9400a6d` foi inteira documentada: 25 commits, dos quais três são merges, todos
dentro da janela de um dia, e nenhum cortado pelo teto de 40. O trabalho é uma sequência de diagnóstico
de "pop-in" (árvores, palmeiras, salas com lâmpada) que termina num mecanismo novo — execuções indiretas
cujos argumentos o cartão lê no lugar passam a ser preparadas uma a uma, imediatamente antes de rodar —
e num port de ray tracing com interruptores próprios. Entram também FSR 4.1.2 de PC até 4K, um HUD novo e
as correções de RX 9070 XT de `beloooo`. Dezoito PRs abertos novos, todos documentados.

## Commits novos (25)

Faixa `a9400a6d..014d52de` (o cursor `commit_cursor` era `a9400a6d`, 18 commits pendentes das 22:00 mais
7 desta hora).

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `673be0ea` | local | Renderer port 7 (`VICTORY_PORT_JOURNAL=1`, off): registros dos heaps pedidos no submit; diário em partes com travas próprias | 6 arquivos, +1.159/−60. Cria `port_journal.rs`: as notas das páginas passam a 64 partes, cada uma com seu lock, e uma resposta é anotada segurando o diário compartilhado e as partes das páginas para escrita (`writes`, `landed_beside`, o pedido de uma submissão antes de suas faixas e número serem fixados). Em `descriptor.rs`/`host.rs`/`queue/behind.rs`/`watch.rs`, uma rodada do thread de sincronização dos heaps é pedida quando o título submete e a lane espera essa rodada. Sem medir aqui: o commit afirma 45,8 → 45,1 ms no merge seguinte. |
| `a609a2fc` | local | Renderer port 7: piso da fila levantado só por esperas achadas em memória; a rodada é pedida quando a espera passa | 3 arquivos, +40/−8. Em `descriptor.rs`, `port_journal.rs` e `queue/behind.rs`, uma espera que passa pede sua rodada na hora, e uma espera atendida por escrita do próprio cartão não levanta piso. O corpo mede run p7-1: com a parte 1 sozinha a lane ainda esperava 4,1 ms de 5,3 pelos registros dos heaps; 2,4 de 16 preparings por frame achavam a rodada pedida antes. |
| `a39509f5` | local | Renderer port 7: o comentário do switch como medido (runs p7-1 a p7-5) | 1 arquivo, +6/−0, só `watch.rs`: comentário. |
| `34061929` | local | Merge port/journal: renderer port 7, registros dos heaps no submit, diário em 64 partes com locks próprios (`VICTORY_PORT_JOURNAL=1`, off; lane −2,4 ms, frame 45,8 → 45,1 ms, `reports/PORT-PATCH-7.md`) | Merge de 6 arquivos, +1.197/−60: leva `port_journal.rs` (+1.159 no commit de origem) e os pontos de chamada em `descriptor.rs`, `host.rs`, `lib.rs`, `queue/behind.rs`, `watch.rs`. O switch fica desligado por padrão. |
| `c9c932cb` | local | README: renderer port 7 (`VICTORY_PORT_JOURNAL`), run merge-5: 44,9 ms | 1 arquivo, +4/−2. |
| `e9428969` | beloooo | RX 9070 XT fixes & HUD ported onto upstream c9c932c | 10 arquivos, +1.128/−197. Em `ambient.rs`, filtra sondas magenta de memória não inicializada (`was.g < 0.5 * min(was.r, was.b)` → zero). Em `host.rs`, `vram_keep` passa a cortar em 85%/93% do orçamento (era 88%/95%) e `SHORT_MS` de 1.500 para 3.000, `BATCHES` de 8 para 32 com pool de buffers de lote compartilhado entre listas. Em `descriptor/host.rs`, `keep_shown` só é consultado quando `resource::room_lately()`. Em `pipeline/make.rs`, o build deixa de usar `OnceLock::get_or_init` e pula chaves que já derrubaram o compilador do driver. `panel.rs` é reescrito (+626/−110): FPS grande, gráfico de frame time, GPU/VRAM/RAM e barras por núcleo. `list.rs` ganha `release_tables`; `Cargo.toml`, `lto = "thin"` e `codegen-units = 1` no release. |
| `b0ba5faa` | local | Title memory by address: o limite de páginas tiradas de faults como switch (`VICTORY_BVH_FAULTS_ALL`) e o que fica de fora no log | 1 arquivo, `memory.rs`, +49/−6. Com ray tracing o limite de 2.048 páginas por faults em toda a execução é atingido no primeiro minuto (run play-1126-0921: aos 63 s); dali em diante 130–500 páginas por frame que os programas de caminhada tocam ficam sem cópia. A linha de dez segundos passa a dizer quantas páginas distintas deram fault e quantas por frame o limite deixou sem cópia; as três primeiras frames que deixam alguma listam todas. Sem o switch, o limite é o de antes. |
| `5d6bd841` | beloooo | Upload title textures in submission prelude to fix frame-order & flickering | 4 arquivos, +317/−176. `Step::Texture(Arc<Staging>)` vira `Step::Texture(PreludeUpload)`: o upload da textura passa a ser gravado no *prelude* da lista no momento da submissão. O comentário antigo registrava que toda lista que amostrava uma textura reenviava tudo, mudada ou não — 5.411 cópias de sub-recurso de 1.120 texturas numa frame do run 18. `panel.rs` ganha `uploaded` por buffer. |
| `e8232632` | local | docs: `VICTORY_BVH_FAULTS_ALL`, e por que o limite fica (run rtf-2) | 1 arquivo, `docs/fork-ray-tracing.md`, +1/−0. Registra que toda página comprometida do título em que um shader deu fault ganhou cópia, mas 110–730 páginas por frame continuaram sem — endereços que não são memória do título, ponteiros de estruturas erradas — e que as páginas encheram os 3 GiB em quatro minutos. Conclusão do autor: o flicker com ray tracing não é o limite. |
| `2de99801` | local | README: a meta agora é a versão de Series X em 4K com ray tracing, imagem primeiro (texturas, luz), depois frame rate | 1 arquivo, +29/−7. |
| `4da7c0be` | local | Renderer port 2: as cópias de tabela do alocador de frame marcam seus slots; o fallback olha toda página escrita em todo sync, salvo `VICTORY_PORT_DESCRIPTORS_CHECK/EVERY` | 3 arquivos, +428/−11. Em `descriptor_sites.rs` (+274) fixa `TITLE_MEMCPY = 0x5f4_0560` e a tabela `COPY_CALLS` para os dois copiadores do alocador: `+0x4d9f200` (tabela de exatamente 128 bytes, quatro `vmovups` de `ymm3..ymm0` para `[rbx+60h..rbx]`, senão `memcpy(rbx, r12, r14)`) e `+0x4d9f540` (`memcpy(rax, r15, rbx)`). `port_descriptors.rs` volta o fallback a olhar **toda** página que o write watch diz escrita, uma vez por frame. O corpo registra 150–300 registros de view por frame que nenhum dos seis stores tinha anunciado, em 16 das ~290 páginas escritas; e que 16 páginas por frame custavam 150–300 views mostrando a view anterior. |
| `b26d6871` | local | Pop watch: com `VICTORY_POP_ALL=1`, uma linha por frame com draws, dispatches e execuções indiretas desde a frame anterior | 2 arquivos, `list.rs` e `popwatch.rs`, +35/−2. `list.rs` ganha contadores totais (nunca zerados) para três categorias e `totals()`; `popwatch.rs` guarda os totais na frame apresentada e imprime a diferença. |
| `77137b98` | local | Stand-in para a luz indireta com ray tracing também, numa faixa (`VICTORY_AMBIENT_RT=<least>,<most>[,<level>]`, off) | 1 arquivo, `ambient.rs`, +68/−9. Com o builder rodando a luz difusa traçada do título pisca e fica preta em interiores (o restaurante da abertura: paredes e teto pretos sob lâmpadas acesas), e as estruturas que os raios caminham estão parcialmente erradas (`reports/RT-FLICKER.md`). Com o switch, o stand-in roda **depois** do resolve do título, também com raios, e mantém o que o resolve deixou entre `least` e `most` vezes a estimativa. Sem `VICTORY_AMBIENT`, o nível da faixa é 0,8. |
| `8eaf2dca` | local | Renderer port 2: o hook da cópia de 128 bytes substitui seus dois últimos stores, não o `mov rcx, r15` para onde o caminho do memcpy volta | 1 arquivo, `descriptor_sites.rs`, +15/−9. O run pop-3 parou aos 43 s numa instrução privilegiada em `+0x4d9f272`; o hook passa a cobrir os dois stores finais. |
| `cd8dfc06` | beloooo | PC FidelityFX FSR 4.1.2 to 4K (`-Fsr4K`), portado de `DexNom/VIctory ab34a8b2` | 14 arquivos, +1.844/−13. `apps/victory/src/fsr_4k.rs` (+194) define perfis SDR da build retail `b99f876b…` (Quality 2560×1440, Balanced 2258×1270, Performance 1920×1080, saída 3840×2160). `crates/victory-hle/src/modules/fidelityfx.rs` (+837) serve `amd_fidelityfx_loader_xbox.dll` com a build de PC no dispositivo do host: a descrição de backend do console (tipo `0x0100_0002`) é traduzida para tipo 2 com o `ID3D12Device` do host, e o dispatch nomeia a lista e as cópias do host. `external.rs` (+71) empresta a lista aberta para gravação síncrona; `Config` ganha `fidelityfx_dir`. |
| `4aeb6d08` | local | Renderer port 2: o fallback mantém o orçamento do port (16 páginas, a cada 8 frames) e o hook das cópias de tabela é opt-in (`VICTORY_PORT_COPIES=1`) | 2 arquivos, +35/−38. Olhar toda página custava 15–25 ms por frame e não impedia o flicker das árvores e palmeiras (runs pop-4 a pop-6); o orçamento volta a 16 páginas a cada 8 frames e o hook das cópias, a exigir o switch. |
| `631400f5` | local | Renderer port 1: execuções indiretas cujos argumentos ou contagem o cartão lê no lugar são clampadas sozinhas, logo antes de rodar | 2 arquivos, `descriptor.rs` e `draw.rs`, +76/−1. `VICTORY_PORT_CONSTANTS_ORDER=0` volta ao comportamento antigo; as faixas ficam registradas na linha da frame. É o mecanismo: um lote aberto depois do último shader que pode escrever buffers ainda clampava cedo demais. |
| `3e04ca0a` | local | Renderer port 1: argumentos no lugar podem compartilhar um lote aberto depois do último shader que pode escrever buffers (`VICTORY_PORT_CONSTANTS_ORDER_ALONE=1`: cada um sozinho) | 1 arquivo, `draw.rs`, +20/−5. Primeira tentativa de consertar o pop-in com lote: o run pop-14 usava cada execução sozinha. |
| `15c3d0ea` | local | Renderer port 1 ordering: argumentos no lugar clampados sozinhos por padrão (um lote aberto depois da última escrita não bastou: run pop-15); `..._ALONE=0` dá o lote | 1 arquivo, `draw.rs`, +6/−5. O padrão de `in_place_alone()` vira verdadeiro: o lote não bastou (as palmeiras continuaram indo e vindo no pop-15). |
| `7ab13ed4` | local | Merge fix/popin: draws indiretos cujos argumentos ficam na memória no lugar do port 1 são preparados um a um antes de rodar (`VICTORY_PORT_CONSTANTS_ORDER`, ligado por padrão); árvores e palmeiras não somem mais, salas com lâmpada estáveis | Merge de 7 arquivos, +554/−10. Leva `in_place_indirect_line` e o clamp individual em `draw.rs`, os totais de `list.rs`, os `counts` por frame em `popwatch.rs`, `VICTORY_PORT_COPIES` em `run.rs` e o rastreio de fault por RIP em `port_descriptors.rs`. Corpo cita `reports/POPIN.md`, runs pop-14..17. |
| `126acb35` | local | README: flicker de árvore e de sala com lâmpada corrigido (`VICTORY_PORT_CONSTANTS_ORDER`), salas escuras com ray tracing (`VICTORY_AMBIENT_RT`); o que continua aberto | 1 arquivo, +13/−4. |
| `1e728610` | local | Merge `avatarmanz/VIctory-7900XTX 126acb35` sobre a rx9070xt de Belobug: flicker corrigido, salas escuras com RT, diagnóstico | Merge de 11 arquivos, +710/−32: traz para a árvore de Belobug o README, `run.rs`, `ambient.rs`, `descriptor.rs`, `draw.rs`, `list.rs`, `memory.rs`, `popwatch.rs`, `port_descriptors.rs`, `descriptor_sites.rs` e `docs/fork-ray-tracing.md`. |
| `84917c34` | local | Renderer port RT (`VICTORY_PORT_RT=1`, off), parte 1: a espera da lane por um constant buffer atrás de uma cópia no lugar | 7 arquivos, +532/−5. O corpo mede: com o builder, **uma submissão por frame parava até todas as flights pousarem — 61 ms por frame na cidade (sx4k-rt-w1) e 15–23 ms no barco (merge-6)** — por causa de um constant buffer de 0xb0 bytes lido na CPU sobre bytes que uma cópia no lugar declarara escritos, recusado com `BEHIND_PLACED`. Cria `port_rt.rs` (+205) e `rtstamps.rs` (+237, diagnóstico: timestamp antes/depois de cada dispatch, somado por pipeline e por tipo builder/walkers/outros); `HostList::forward_fill` tira os bytes da origem da cópia quando o fill está atrás dela na mesma lista. |
| `d13e9e0c` | local | Renderer port RT: bench de leituras de memória do título por endereço, onde as cópias caminhadas ficam (memória do PC ou cartão) na linha de memória | 3 arquivos, +281/−0, com `host/rtbench.rs` novo. rts-1 (stamps): os programas que caminham as estruturas gastam 56–60 ms de cartão por frame em 16 dispatches, o builder 1,5 ms em ~275. O bench mede que uma view escolhida por lane não custa nada enquanto as lanes de uma wave a compartilham, e que duas cópias por wave dobram o tempo (pegada de cache). |
| `014d52de` | local | Renderer port RT: teste de que o switch toma a ordem colocada; o bench sem o diretório | 2 arquivos, +19/−3. `card_steps.rs` ganha `#[test] the_port_takes_the_order`: com `PORT_RT_IN_TEST`, um fill à frente de uma cópia colocada passa e um atrás continua `BEHIND_PLACED`; sem o switch, os dois eram recusados. `rtbench.rs` ganha o caso "one view, no directory" (`spread == 99` → `e = uint2(0, (p >> 10) * 1024 * 64)`). Medição no corpo (Series X 4K/1440p, RT, ports 1/2/5/6/7, `-Walk`): cidade parado 145,3 → 126,2 ms, andando 131,1 → 118,7; barco parado 58,2 → 46,2, andando 94,5 → 75,6. |

## Pull requests abertos (18)

Todos acima de 387, todos documentados. Propostas lidas do corpo de cada PR.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 390 | GPU: keep textures the title rewrites in place in step (TV and phone video) | O noticiário da TV mostrava um fantasma colorido da frame anterior (braço e cabeça do repórter em rosa ao lado). As imagens de vídeo são texturas do título reescritas no lugar: luma 2048×1088 (2.228.224 bytes) e croma 1024×544 (557.056); a passada de textura recheca 4, 12, 40, 120 e 400 frames depois do upload. | Parcialmente o **D-09** (textura mostrada antes do upload): aqui a causa é reescrita no lugar, não upload atrasado. Sem id. |
| 391 | Windows in mapped card memory: how full the card may be (`VICTORY_WINDOWS_BAR_PCT`) | `VICTORY_WINDOWS_BAR_MB` só põe janelas em memória mapeada (ReBAR) abaixo de 80% do orçamento; com Series X e FSR de PC o orçamento passa de 80% antes do mundo carregar e nenhuma janela vai. O switch move a linha (10..99). | Não. |
| 393 | Loader: descriptor and write sites reach their stubs without the GPL-2.0-only `console_insn` helpers | `descriptor_sites.rs` e `write_sites.rs` pegam emprestado o codificador `jmp rel32` de `console_insn.rs`, que se declara portado do XCL (GPL-2.0-only) num repo GPL-3.0-or-later. Cria `reach.rs`. | **Sim: é o D-17 do próprio AvatarManz** ("README nega código de XCL; dois arquivos declaram port de XCL (GPL-2.0-only) num repo GPL-3.0-or-later"), ainda aberto no catálogo. |
| 401 | Descriptors: dynamic slots go back only to the heap that took them (#392) | Uma lista guardava os slots dinâmicos pelo endereço do heap do título e os devolvia a qualquer heap registrado naquele endereço ao rodar/reiniciar/liberar; o título pode liberar um heap com slots ainda em uso e criar outro no mesmo endereço. | Candidato a defeito novo (issue #392 do próprio repo). Sem id. |
| 402 | HLE: skip the title's Bink movies on request (`VICTORY_SKIP_BINK`, `play.ps1 -SkipIntro`) | Pula logos e filme de abertura (450 frames). Port de `fxceless/victory-fork#613`, com código e testes do autor original. | Não. |
| 403 | Shader tests: differential fuzzing against an RDNA2 interpreter ("oracle") | Fuzzer diferencial: programas aleatórios contra um modelo da ISA RDNA2 que não compartilha código com o tradutor, na GPU. Um commit sobre `014d52d`, sem mudança de tradutor. | Não. Teste. |
| 404 | Shaders: s_min/s_max set SCC from the comparison; v_readlane/v_writelane lanes wrap | Duas falhas de lowering achadas pelo fuzzer: `s_min_{i,u}32`/`s_max_{i,u}32` não geravam SCC próprio (usavam `D != 0`) e `v_readlane/v_writelane` não faziam a lane dar a volta. | Não catalogado; defeito de tradutor real. |
| 405 | Shaders: scalars written in lane-divergent branches reach every lane after the join | Escalares escritos em ramos divergentes por lane precisam chegar a todas as lanes depois do join. Conflita com #81 (mesmo `widen()`/`branches_around`) e cita `Belobug/VIctory-Belobug-AMD#352`. | Não. |
| 408 | Diagnostics: renderer port 1, CPU reads too early and memory changed under the card (#379) | Dois diagnósticos (só build `diagnostics`, `draft=true`): passos de CPU que leem a memória do port 1 antes das escritas dos nossos shaders, e memória mudada sob o cartão. | Não. Instrumentação do **D-01/D-03**? Não — do artefato do #379, sem id. |
| 409 | Clippy clean again, with and without --all-features | Os dois passos de clippy do CI falham em `main` (`126acb3`): 8 erros sem features, 6 com `--all-features`; corrige na origem sem `#[allow]`. Inclui gate de `totals()` em `list.rs` sob `cfg(all(windows, feature = "diagnostics"))`. | Não. CI. |
| 410 | Decode source-less VOP1 length and the EXEC destination of VOP3 CMPX | `v_nop`, `v_pipeflush`, `v_clrexcp` têm bits de fonte ignorados, mas cinco codificações faziam o decodificador consumir a instrução seguinte; VOP3 CMPX escreve EXEC e o destino decodificado seguia o VDST codificado. Sobre `014d52d`. | Não. |
| 412 | Formats/BCPack: BC7 channel rotation only in modes 4 and 5 (port of `Marvisak/VIctory#3111`) | `bc7_precision()`/`bc7_pack()` leem `ctrl >> 3 & 3` como rotação de canal em todo modo BC7, mas só os modos 4 e 5 têm esse campo: modos 0/1/3 com rotação 1 dão panic no shift, modos 0–3 com 2/3 perdem canal, 6/7 pulam o p-bit do vermelho. | Candidato a defeito novo (issue #411 do repo). Mesmo tipo do **D-18** (constantes de formato fixas), mas não é ele. |
| 413 | Renderer port 5 on NVIDIA: constant buffers in place through views from the 256 bytes below (`VICTORY_PORT_CBV=2`) | Na NVIDIA, `VICTORY_PORT_CBV=1` deixa todo constant buffer de tabela copiado: os twins raw precisam de heaps de 2^22 e a NVIDIA limita CBV/SRV/UAV a 1.000.000 views (`raw_twins=false`, 0 lidos no lugar, ~7.600 copiados por frame). O modo 2 lê CBV como CBV através de views dos 256 bytes abaixo. | Não. |
| 414 | Stand-in fix for too-bright reflections (`VICTORY_AMBIENT_REFLECT`, off) | `CS_RayMiss_de6b91a6` preenche todo raio de reflexão que não acerta nada com a luz da sonda de céu; sem RT e sondas, quase todo raio erra e olhos, pele molhada e piso glossy refletem com força de céu aberto em interiores. O switch limita a reflexão pela luz ao redor. | Não. Mesma família do `77137b98` (stand-in de ambiente), mas reflexões, não difusa. |
| 415 | Clarify which settings the safe launcher preset keeps | Explica o que `-Safe` **ainda** liga (`VICTORY_TEXTURE_NO_WAIT=1`), referência #240. | Não. Doc. |
| 416 | Add a runtime bug report template | Template de issue para falha de runtime, com seção 6 pedindo contexto para comparar builds e máquinas. Referência #358. | Não. |
| 417 | Show a directory path for the texture store setting | O README passa a mostrar um caminho para `VICTORY_TEXTURE_STORE` e explica que `1` seleciona `./1`; confere `store()` em `texstore.rs`. | Não. Doc. |
| 419 | Retry texture detiling after an incomplete conversion | O staging gravava o digest da origem antes de detilar o primeiro mip; se um mip posterior falhasse, o refresh seguinte aceitava a mesma entrada como cache hit, e o caminho de write-watch podia avançar a marca d'água. Passa a gravar o digest depois do último sub-recurso. | Não catalogado; candidato (cache de detile aceitando resultado incompleto). |

## Pull requests mesclados (0)

Nenhum `merged_at` posterior a `2026-11-26T08:17:50Z` entre os 100 fechados mais recentes por `updated`.
A atividade desta faixa é toda de commits diretos e merges locais, não de PR mesclado pela interface.

## Cruzamento com o catálogo

- **D-05** (tamanho de estrutura BVH lido antes de a submissão pousar — cópias sobrescreviam vizinhos;
  corrigido em `2c20d7a`): `84917c34` **não é** o D-05, mas é o mesmo mecanismo — leitura na CPU de bytes
  que uma cópia colocada declarou escritos — aplicado a um constant buffer de 0xb0 bytes, não ao tamanho
  da BVH. O commit **generaliza** o conserto: `HostList::forward_fill` passa a tirar os bytes da origem da
  cópia, e o D-05 continua fechado. Não reabre.
- **D-03** (`VICTORY_SEND_CHANGED` ligado por padrão fez a sala perder luz indireta e explodir a
  geometria): `77137b98` trata do mesmo sintoma visível — interiores escuros com ray tracing — com um
  stand-in de luz indireta em faixa, **desligado por padrão**. Não muda o switch do D-03 nem o motivo do
  defeito. **Não corrige.**
- **D-09** (device loss a cada ~16 min: textura mostrada antes de o upload rodar): `5d6bd841` move o
  upload de textura para o prelude da submissão, e o comentário do próprio diff diz que "toda lista que
  amostrava uma textura reenviava tudo, mudada ou não". É o mesmo eixo (textura exibida fora de ordem com
  o upload), mas o commit **não menciona device loss nem mede 16 minutos**, e o D-09 segue aberto no
  catálogo. **Não confirmado como correção.**
- **D-08** (compilação de pipelines travava o título por dezenas de segundos): `e9428969` troca
  `OnceLock::get_or_init` em `pipeline/make.rs` por um caminho que pula chaves que já derrubaram o
  compilador do driver — ataca o caso "worker preso para sempre", não o tempo de compilação. **Não
  corrige o D-08; fecha uma espera infinita vizinha.**
- **D-17** (código de XCL sob GPL-2.0-only num repo GPL-3.0-or-later): aparece como PR aberto **#393**, não
  como commit. Continua sem correção mesclada.
- **Abre defeito novo?** Três candidatos com hash de arquivo e linha:
  - `e9428969` — filtro que zera sondas magenta de "memória não inicializada/corrompida na AMD" em
    `crates/victory-gpu/src/ambient.rs`. É um curativo que esconde uma origem não identificada: sonda
    lida como magenta é dado inválido, não cor.
  - `4aeb6d08` registra que olhar todas as páginas custou 15–25 ms por frame; `4da7c0be` havia medido
    150–300 registros de view por frame sem marca. Fica como dívida de desempenho, não como defeito de
    correção.
  - `8eaf2dca` conserta um bug introduzido na própria faixa: o hook da cópia de 128 bytes substituía
    `mov rcx, r15` e o run pop-3 morreu aos 43 s com instrução privilegiada em `+0x4d9f272`. Os dois
    commits têm 6 minutos de diferença; o catálogo poderia ganhar a entrada "hook de patch cai no meio da
    instrução" (introduzido em `4da7c0be`, corrigido em `8eaf2dca`).

## Confiança

Verificado: os 25 SHAs, autores e datas vieram de `repos/avatarmanz/VIctory-7900XTX/commits?sha=main&per_page=100`,
com o cursor `a9400a6d` na posição 25; cada commit foi lido em `commits/<sha>` (arquivos, adições e
remoções, e trechos de patch de `port_journal.rs`, `draw.rs`, `port_rt.rs`, `rtstamps.rs`, `rtbench.rs`,
`descriptor_sites.rs`, `ambient.rs`, `host.rs`, `texture.rs`, `fsr_4k.rs`, `fidelityfx.rs`); os 18 PRs
abertos, em `gh pr list --state open --limit 50` com número acima de 387; nenhum merge na janela pelos 100
fechados por `updated`.

Não verificado: as medições de frame citadas (`45,8 → 45,1 ms`, `61 ms`, `145,3 → 126,2 ms`,
`15–25 ms`) são as do corpo dos commits, não reproduzidas aqui. Não verificado que
`ab34a8b2` do DexNom contenha o mesmo FSR que `cd8dfc06` porta — o commit só o nomeia. Os PRs abertos não
tiveram diff lido: as propostas vêm do corpo. Não verificado se `reports/POPIN.md` e `reports/RT-FLICKER.md`
existem no repositório; são citados pelos commits.
