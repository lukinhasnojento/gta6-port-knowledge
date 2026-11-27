# fxceless/victory-fork — atualização de 2026-11-27 01:00

## Resumo

Catorze commits na `main` (`81d54b06..67429060`), sem nenhum PR mesclado na janela. O eixo é o
tradutor de shaders: entra o PR externo #1419 (ByFxbian) com as reduções FP32 ordenadas para wave64 em
hosts de 32 lanes, resolvido à mão (`8a32d0ba`) e depois corrigido em três commits — só campos de MODE
que tocam arredondamento/denormal de FP32 barram a redução, somas FP64 exatas viram `contagem × termo`, e
programas pareados passam a ser escritos sem as flags de fast-math. Os prefix sums do construtor de BVH
passam a traduzir num par de commits e o construtor fica **ligado por padrão** — com uma queda conhecida
documentada no README: o título morre escrevendo no endereço 0 entre 30 e 40 s depois de o construtor
começar. Quatro recursos novos ficam desligados por padrão: destile de textura na placa, heap de
constantes no lugar, patch magro e dump de shaders recusados.

## Commits novos (14)

Cadeia linear conferida pelos pais: `81d54b06 → 8a32d0ba → 3ab5c12b → b63b465f → 50c79c35 → e3307c77 →
a9181638 → 4966cd67 → ff1776b7 → 17533c5c → 3461f52f → bcc54215 → 1170216f → 1320dfd7 → 67429060`.
Nenhum é commit de merge: os dois com assunto igual (`1170216f` e `1320dfd7`) são o corpo e o recorte de
uma mesma mudança.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `8a32d0ba` | fxceless | Fork pull request #1419 (ByFxbian): ordered paired FP32 reductions for wave64 on 32-lane hosts (the two water programs' refusal in #1397); conflict: lower.rs's packing helpers moved to signature_packing.rs | 6 arquivos, +1022/-584. Aplica o PR externo **sem** commit de merge e resolve o conflito movendo os ajudantes de empacotamento de assinatura de `lower.rs` (que perde 547 linhas) para `signature_packing.rs` (novo, 167). `lds_regions.rs` novo (384) e `paired_fp32.rs` novo (160) saem do mesmo desmembramento; `wave64_after_loop.rs` recebe +279/-29 (a redução ordenada e o laço depois dela) e `paired.rs` +11/-8. É a maior mudança da hora e a base de todos os commits seguintes. |
| `3ab5c12b` | fxceless | Paired FP32: only MODE writes that touch FP32 rounding or denormals (bits 0-1, 4-5) stop the ordered reduction; the water programs set hwreg(MODE, 2, 2) only | 1 arquivo, +49/-10. `mode_change_touches_fp32` passa a decodificar o imediato de `s_setreg_imm32_b32` (id, deslocamento e tamanho) e só barra a redução quando o campo toca os bits 0-1 ou 4-5 de `HW_REG_MODE`; `s_setreg_b32`, `s_round_mode`, `s_denorm_mode` e `s_swappc_b64` continuam barrando. É o que deixa o `hwreg(MODE, 2, 2)` dos programas de água passar. |
| `b63b465f` | fxceless | fps48 01-prep-meter: card_before by part and the lists steps by kind (meters only) | 1 arquivo, +140/-0, só instrumentação. `CARD_BEFORE_PARTS` (7 contadores) e `card_part` medem as fases de `Host::card_before`; sob a feature `diagnostics`, `STEP_KINDS`/`STEP_FLUSHES` e os guardas `StepTimed`/`FlushTimed` medem os passos das listas por tipo; duas linhas `tracing::info!` publicam ms por frame. Nenhuma mudança de comportamento. |
| `50c79c35` | fxceless | fps48 02-patch-lean: VICTORY_PATCH_LEAN (off, alt): busy pages go up as patch words under the cap, patched pieces rehashed one span a piece, pooled word buffers | 2 arquivos, +206/-10. `shadow.rs` (+152) e `host.rs` (+54/-8): `CardWork` ganha `patch_pool` de no máximo `PATCH_POOL_MOST = 4` buffers de palavras de patch já submetidas, reaproveitados no patch seguinte; páginas ocupadas sobem como palavras de patch abaixo do teto e as peças remendadas são re-hasheadas um trecho por peça. Desligado por padrão. |
| `e3307c77` | fxceless | Water programs in paired form: their wave64 sums replayed exactly (gfx48: unseen partial sums, exact FP64 constant sums as count x term), paired FP32 programs written without fast-math flags; MODE gate: only the 64/16-bit fields (bits 2-3, 6-7) pass | 4 arquivos, +404/-20. `paired.rs` (+64) ganha `paired_f64_constant_sum`: uma soma FP64 de seção cujo termo é uma constante com somas exatas vira `n × termo` (n = lanes no máscara, as duas metades somadas), com `makeDouble`/`splitDouble` e `fptosi→fpext` da contagem; qualquer outra soma FP64 é recusada (`a paired FP64 reduction`). `paired_fp32.rs` (+202) acrescenta `unseen_after` (as somas parciais que o compilador deixa em VGPR são consideradas não vistas quando toda leitura posterior é sobrescrita por inteiro em toda a onda) e `exact_f64_term`; o gate de MODE passa a recusar também os bits 8-9 (DX10_CLAMP/IEEE). `bitcode.rs` (+8) traz `exact_floats`, que desliga as flags de fast-math por módulo. `wave64_after_loop.rs` (+130) traz o fixture `water_sums` com as duas somas FP32 da água, os três termos aceitos e as recusas. |
| `a9181638` | fxceless | Shared-memory slots that every write sets to one constant and every atomic treats as its identity are set by every thread at the start, behind a group barrier (gfx48; the water tiles' min/max start values the corner thread skipped when it returned early; VICTORY_LDS_INIT_HOIST=0 off) | 3 arquivos, +229/-1. `lds_regions.rs` (+134): `lds_init_hoists` acha os dwords de LDS cujo **único** valor escrito é uma constante (por `ds_write_b32`/`ds_write2_b32` de endereço e dado constantes do próprio bloco) e cujas atômicas usam essa constante como identidade (`ds_min_u32`/`ds_and_b32` → `u32::MAX`, `ds_max_u32`/`ds_add_u32`/`ds_or_b32`/`ds_xor_b32` → 0, mín/máx assinados → `0x7fffffff`/`0x80000000`); se qualquer escrita ou atômica tem endereço/dado não constante, a lista vem vazia. `lower.rs` (+27) chama `hoist_lds_inits` na entrada do programa, grava esses dwords e põe uma barreira de grupo antes de qualquer acesso. O fixture de `wave64_after_loop.rs` reproduz o programa de tile (thread 0 grava o máximo/mínimo inicial, toda thread acumula `ds_min`/`ds_max`) e confirma `[(0,0), (1,u32::MAX)]`, a recusa de um valor inicial que não é a identidade e a recusa com escrita de endereço variável. |
| `4966cd67` | fxceless | The ray-tracing structure builder runs by default (VICTORY_BVH_BUILDER=0 off); ray node tests answer no hit only while it is off | 2 arquivos, +6/-3. `builder_runs` inverte: ligado salvo `VICTORY_BVH_BUILDER=0`. `intersect.rs` inverte a condição gêmea: o teste de nó de raio só responde "sem acerto" com o construtor **desligado**. |
| `ff1776b7` | fxceless | VICTORY_DUMP_REFUSED=<dir>: a shader that does not translate is saved as <hash>.cdxil for offline work (the structure builder's three refused programs live in its DLL, not in the game's archives) | 1 arquivo, +15/-6. No ramo de erro da tradução, cria o diretório e grava o contêiner recebido do título como `<hash>.cdxil`; o aviso de "shader não traduz" continua. Erros de escrita são ignorados (`let _ =`). |
| `17533c5c` | fxceless | VICTORY_WAVE_PAIRS_SPILLED=1: the paired form is tried on spilled DispatchX programs too (off; the structure builder's three wave64 programs) | 1 arquivo, +5/-1. A recusa "paired Wave64 for spilled DispatchX programs is not validated" passa a ser ignorada quando a variável vale `"1"`. Desligado por padrão, e o comentário diz que é para conferência offline. |
| `3461f52f` | fxceless | VICTORY_GPU_DETILE (0/1/alt, off; detile50): staging buffers hold the title's tiled bytes and a compute pass in the list's prelude detiles them on the card (byte-exact against the CPU detile on WARP; CPU falls back for what it cannot take); per-frame line with MB by swizzle mode | 5 arquivos, +1089/-6. `crates/victory-gpu/src/gpudetile.rs` novo (714 linhas) mais 168 em `victory-formats/src/texture.rs`, 158 em `victory-gpu/src/texture.rs` e 47 em `host.rs`: os bytes tileados do título vão para um buffer de staging e um passe de computação no prelúdio da lista os destila na placa; o que a placa não assume volta à destilação na CPU. Desligado por padrão, com `alt` para alternar. |
| `bcc54215` | fxceless | VICTORY_INPLACE_CONSTANTS=1 (off; inplace50): the title's 174 MB constants heap is opened for the card where the title writes it (a second view of its section, OpenExistingHeapFromAddress1): no copies, hashes, compares or patches for it; alt is turned down (a slot keeps the form it was made with) | 3 arquivos, +430/-4. `inplace.rs` (+393): `constants_on`/`mode_of` (`1` liga, `alt` é recusado com aviso), `constants_min` (`VICTORY_INPLACE_CONSTANTS_MIN_MB`, 64), `constants_wanted` (abaixo de 4 GiB, memória de view que é um trecho de seção), `Region`/`REGIONS` alinhados a páginas de 64 KiB, `place_constants` e `constants_forget` (chamado em `title_memory_freeing` para o título que remapeia as páginas). `shadow.rs` (+33): com o recurso ligado a faixa entra na conta de escritas da GPU das flights mesmo sem cópia, e uma sombra só abre região se nenhum buffer do título nas mesmas páginas já tem cópia (`fresh`). `host.rs` (+4/-4): `carded() || in_place()` pula a cópia de volta. Só um buffer casa na prática: o heap de 174 MB em `0x3de00000`, dentro de uma seção de 185 MB (runs 115 e 363). |
| `1170216f` | fxceless | Structure builder's prefix sums translate (bvh51): a shared-memory read at a uniform address is lane-uniform (PR #36 recut); the first-lane read takes the first lane within the DispatchX count | 2 arquivos, +201/-19. `lane_uniform` é reescrito: o teste de guarda passa para o começo (`st.guards[b][i].is_some()` → `continue`), a leitura de vetor passa a olhar só os dwords do operando (`reg..reg+dwords`, exceto `v_movrel`/`v_swap`, que mantêm "deste registro em diante"), e um `Fields::Ds` de `ds_read*` **sem** `_d16`, endereço vetorial e `gds: false` conta como uniforme quando a leitura do endereço é o mesmo valor em todas as lanes. O fixture `prefix_sum_words` reproduz as palavras 0x14c..0x210 dos três programas de prefix sum do construtor (última lane de cada onda grava o total em LDS, barreira, as ondas seguintes somam os totais anteriores com `s17 + lds[s18*4]` lido por `v_readfirstlane_b32`). |
| `1320dfd7` | fxceless | Structure builder's prefix sums translate (bvh51): a shared-memory read at a uniform address is lane-uniform (PR #36 recut); the first-lane read takes the first lane within the DispatchX count | 2 arquivos, +67/-1. Corrige o commit anterior: em `v_readfirstlane_b32`, quando as metades trocam (`halves`) e o valor é igual em toda lane, a guarda do `first_lane` passa a ser `guard AND partial_active` — as threads além do `DispatchX` pularam as leituras de LDS e seguram outro valor, então a primeira lane é a primeira das outras. Teste novo valida o programa como `DispatchX` derramado, exige o user data `USER_DISPATCH_THREADS` no dword 0 e passa o validador de DXIL quando ele existe. |
| `67429060` | fxceless | README: status of 2026-11-27 03:45 (structure builder runs; its known crash; water) | 1 arquivo, +18/-2. Só documentação: sobe o carimbo de status, registra que o construtor de BVH roda por padrão e que **o título morre escrevendo no endereço 0 entre 30 e 40 s depois de o construtor começar (duas execuções, com e sem `VICTORY_BVH_QUARANTINE`)**, que a água tem a cor verde com os dois programas no formato pareado, e que `VICTORY_GPU_DETILE`, `VICTORY_INPLACE_CONSTANTS`, `VICTORY_PATCH_LEAN` e `VICTORY_DUMP_REFUSED` estão desligados por padrão. |

## Pull requests abertos (11 novos)

Nenhum PR foi mesclado nesta janela; os onze abaixo estão acima do cursor 1411 e continuam abertos.
Três dos onze são de autores de fora e citam adaptação de outro fork, com atribuição.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 1412 | GPU: opt-in ordered single-group indirect batches | `VICTORY_INDIRECT_BATCH=1` usa a janela de tabela contada em grupos de 64 lanes para registros `Draw`/`DrawIndexed` homogêneos e só-leitura, excluindo buffers de contagem, mesh e dispatches; cada execução ganha entrada própria. Adaptação de Kelvaroq 0051. | Não. |
| 1413 | GPU: cache exact buffer span unions per heap | `VICTORY_DESCRIPTOR_BUFFER_SPANS=1` guarda a faixa envolvente exata por buffer de base no passe não marcado, com invalidação em inconsistência, varredura completa e destruição do heap. Adaptação de Kelvaroq 0055. | Não. |
| 1414 | Descriptors: every slot of a table a draw binds counts as a table's slot | `VICTORY_TABLED_ALL=1` deixa todo slot de uma tabela ligada a um draw para os draws que a leem, qualquer que seja o conteúdo, por `VICTORY_TABLED_ALL_FRAMES` (padrão 60). O corpo mede 80–91 → 56–62 ms na célula do prólogo numa RTX 3090 Ti e registra um par **mais lento** numa RTX 5060 Laptop sob Proton. | Não. |
| 1415 | Diagnostics: VICTORY_FRAME_HEALTH=1 writes a per-frame CSV | Uma linha CSV por frame apresentado (draws/dispatches executados, pulados por compilação, recusados, texturas não prontas, VRAM), mais resumo no stderr a cada 5 s. | Não. |
| 1416 | Warm-up: the run's log names the view heap's null slot | A linha do log passa a nomear `null_slot=views-1` e a ajuda de `--warm` manda usar esse valor, não `views`. O corpo mede 0 acertos de cache com o valor errado e 3.097 com o certo numa RX 9070 XT. | Não; é o defeito de uso que o próprio PR descreve, sem entrada no catálogo. |
| 1419 | Shaders: ordered paired FP32 reductions for Wave64 on Wave32 hosts | Reduções FP32 ordenadas para árvores wave64 completas em hosts de 32 lanes, o que destrava os dois shaders de `TransmissionVolumetrics`. | Não. **Já está no fork**: `8a32d0ba` aplica este PR localmente, e o PR continua aberto no repositório de origem. |
| 1422 | GPU: a skin bake's own sources are uploaded while the card's budget holds them | As duas portas de `resource::room()` do bake passam a usar `within_budget(bytes)` com 95 % do orçamento, sem contar o que está guardado para cópias futuras; mede 106 → 0 pulados e 309 de 336 fontes não feitas. | Não. |
| 1424 | processor side: the copies a submission makes, and the pages it looks at, on more than one thread | Cópias de 4 MiB ou mais são cortadas em peças de 2 MiB lidas pelos threads guardados, e as páginas do `sync_heaps` são lidas/comparadas em paralelo sob o mesmo gate. Marcado como **draft**, sem números de frame. | Não. |
| 1425 | GPU: preserve texture retry deadlines after card memory holds | Preserva o prazo de nova tentativa de textura quando a memória da placa volta depois de uma retenção; adaptado de TaynanGT/Vice-Tay#81. | Não. |
| 1426 | GPU: allow disabling saved pipeline scheduling history | `VICTORY_PIPELINE_HISTORY=0` desliga o histórico de escalonamento salvo; porta de TaynanGT/Vice-Tay#43. | Não. |
| 1427 | GPU: avoid Vec allocations in Shadow::card_merge_relay when skip is empty | Evita o `Vec` de `skip` e a alocação de `outside()` quando não há escrita concorrente, mantendo o caminho exato no caso não vazio. | Não. |

## Pull requests mesclados (0)

`gh api repos/fxceless/victory-fork/pulls?state=closed&sort=updated&per_page=100` não trouxe nenhum PR com
`merged_at` posterior a `2026-11-26T23:06:47Z`. O único PR de fora que entrou no código nesta janela
(#1419) foi aplicado como commit comum, e continua aberto — ver a tabela acima.

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum do catálogo. **D-12** ("loads escalares agrupados de 4 dwords leem além do fim
  da view — água preta") é outra causa: `a9181638` conserta os valores iniciais de mínimo/máximo em LDS
  que a thread de canto pulava num retorno antecipado, o que o README liga a "tiles escuros piscando" —
  sintoma parecido, mecanismo distinto. **D-01/D-02** (forma wave64 larga, chave de cache) não são
  tocados: o wave64 desta hora é a redução ordenada, e o `178...`/`e9765a7` do catálogo continua sendo a
  referência. **D-16** (perdas de device sem causa identificada) fica **agravado**, não corrigido: ver
  abaixo.
- **Abre defeito novo?** Um candidato, e ele é o mais importante da hora:
  - `4966cd67` liga o construtor de BVH por padrão e `67429060` documenta a consequência no README:
    **"about 30-40 s after the builder starts, the title crashes on a write to address 0 (two runs, with
    and without `VICTORY_BVH_QUARANTINE`)"**. Isto é um crash de título, não uma perda de device, e o
    README só oferece `VICTORY_BVH_BUILDER=0` como saída. Não há correção em nenhum commit desta janela.
    Candidato a linha nova no catálogo, ligado a `4966cd67` (o que liga) e `67429060` (o que documenta).
  - `1170216f` amplia o que conta como "mesmo valor em toda lane" (leitura de LDS em endereço uniforme) e
    `1320dfd7` já corrige um caso do próprio commit: a primeira lane podia estar fora do `DispatchX`. O
    par fica registrado como mudança em dois tempos, não como defeito aberto — o teste
    `a_first_lane_read_of_a_per_lane_shared_memory_read_is_still_refused` mantém a recusa no endereço
    por lane.
  - `3461f52f` e `bcc54215` movem bytes do título para a placa e para um heap aberto **por cima** da
    memória do título; ambos vêm desligados por padrão e o comentário de `bcc54215` declara os limites
    (descritor guarda a forma com que foi feito; cópia no lugar não vê escrita da placa). Lacuna
    declarada, não regressão demonstrada.

## Confiança

Verificado: os 14 SHAs e metadados em `repos/fxceless/victory-fork/commits?sha=main&per_page=100`, com o
cursor `81d54b06` na posição 14; cada commit lido em `commits/<sha>` (lista de arquivos, contagens e
patch — inclusive o diff completo de `lane_uniform`, `lds_init_hoists`, `mode_change_touches_fp32`,
`paired_f64_constant_sum`, `constants_on`/`place_constants` e do README de status); a cadeia de pais,
um por commit, sem merge commit; os 11 PRs abertos com os corpos em `pulls?state=open`; a ausência de
merge na janela por `pulls?state=closed&sort=updated` filtrado por `merged_at`.

Não verificado: a execução do título e dos testes (as medidas de frame, os "runs 115/363", a queda no
endereço 0 e o "1383 frames contra 459" vêm dos textos do commit e do README); se o crash do construtor
de BVH já tem causa identificada fora do README; se `8a32d0ba` é exatamente o PR #1419 ou o PR com as
correções locais somadas (o assunto diz "conflict: lower.rs's packing helpers moved"); e o efeito do par
`VICTORY_GPU_DETILE`/`VICTORY_INPLACE_CONSTANTS` em jogo real, já que o primeiro não foi medido aqui e o
segundo, segundo o próprio README, "did not open in its first run". Nenhum teste foi executado aqui; os 13
repositórios monitorados são somente leitura.
