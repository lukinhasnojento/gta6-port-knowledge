# avatarmanz/VIctory-7900XTX — atualização de 2026-11-26 22:00

## Resumo

A noite de ports do renderer: 58 commits na faixa, 40 documentados (teto), todos de `main`. Entram os
ports 1, 2, 5, 6 e 7 (argumentos indiretos lidos no lugar, *guards* com lock compartilhado, constantes
como bytes crus, journal de heaps), o desbloqueio de frame rate (`VICTORY_FRAME_INTERVAL_MS`),
tamanho de saída e escala de render, o *stand-in* de luz que assenta por tempo, o repositório de
texturas em disco e a composição de dois planos — este último ligado **por padrão** em `0ce46d5a`.
O commit `9ba5661f` refaz a correção de replay que o **D-10** do catálogo descreve como feita e
revertida sem explicação.

## Commits novos (40 de 58)

Faixa `f03bfa51..126acb35`, 58 commits. Documentados os 40 primeiros na ordem do `compare`; os 18
restantes (`673be0ea` … `126acb35`) ficam para a próxima execução com `commit_cursor` em `a9400a6d`.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `85a663d3` | local | coolfox 641835a4+581b0cb9: compose a second presented plane over the first (VICTORY_PLANES=1) | Novo `compose.rs`: dois planos desenhados como um, o segundo premultiplicado sobre o primeiro e cada um escalado à tela; `Host::present_planes` lê o segundo plano no stride de 72 bytes do `PresentX` de retalho e confere que o recurso é nosso; overlay passa pelo job de present da lane de submit. Barreiras de/para `COPY_SOURCE` são omitidas quando a textura já está nesse estado. Fora do fork: `WakeByAddressAll` em escritas de memória. +638/−36, 6 arquivos. |
| `46e32978` | local | coolfox 641835a4: tables planned once per pipeline (TablePlan); fallback heaps (VICTORY_FALLBACK_HEAPS=1) | `TablePlan` agrupa por pipeline o que as tabelas de descritor de um shader contêm (slots, CBV/UAV, último dword, entradas por descritor), em vez de varrer as declarações a cada bind; mesmas tabelas, mesma ordem. `VICTORY_FALLBACK_HEAPS=1` (off) garante view heap e sampler heap sempre ligados por lista, com heap host de um slot (view nula, sampler point) para o que o título não setou. +251/−69. |
| `cee78fa7` | gomeaw | Trace where pipeline workers spend time | Separa no trace a tradução de shader e a aquisição de lock das chamadas de load/create do D3D12, e instrumenta esperas de primeiro uso e serialização de library, para atribuir tempo de worker num trace de dependência de Present. +23/−3. |
| `706305b4` | local | coolfox residency budgets (VICTORY_RESIDENCY=1); pool growth backs off after a refused block | `residency.rs` consulta a memória do sistema uma vez por segundo: sob pressão (3 GiB livres / 4 GiB de commit) limita prefetch de textura a 64 MiB por frame, solta chunks ociosos e mantém menos blocos vazios; abaixo de 1,5 GiB cai a 16 MiB e nenhum bloco de staging sobrando; abaixo de 512 MiB o passe de textura para. Com ≤32 GiB de RAM ou `VICTORY_RESIDENCY_LIMITED=1`, orçamentos diferentes. +435/−14, 9 arquivos. |
| `6d172c42` | local | coolfox c64fc9b4: the heap buffer pass keeps its ranges on the heap; per-heap passes (VICTORY_SWEEP_PER_HEAP=1) | O que o passe de buffer bindless guarda de um view heap (passes, ranges dos slots, último refresh de cópias não observadas) passa a viver no heap host, então um heap criado no endereço de outro liberado começa limpo. Com intervalo padrão 1 nada muda. `VICTORY_SWEEP_PER_HEAP=1` (off): passe por heap, tabela de strides por chave própria, passe adiado descartado se o heap no endereço já tem outra tabela. +60/−32. |
| `8a0700ae` | local | coolfox fabb872a/641835a4: XMA voices refilled after their end (VICTORY_XMA_REFILL=1); DirectStorage staging floor (VICTORY_DSTORAGE_STAGING_MB) | `VICTORY_XMA_REFILL=1` (off): voz XMA sem entrada volta quando o título dá buffer válido de novo, e o pump publica só blocos inteiros (cauda acolchoada) após no máximo 8 passos — o fork limpa 6 bits onde o campo lido tem 5. `VICTORY_DSTORAGE_STAGING_MB=<n>` fixa piso do staging (padrão: o que o título pede, 32 MiB; o fork usa 256 MiB). +43/−4. |
| `a6a6ef73` | gomeaw | Merge remote-tracking branch 'upstream/main' into fix/trace-pipeline-workers | Merge de `upstream/main` na branch do trace de workers: 43 arquivos, +4774/−441, trazendo `apps/victory`, `victory-gpu` (book, copystat, cores, descriptor) e o restante da árvore para a base da PR #144. |
| `773d902f` | local | coolfox 641835a4: one barrier per card buffer after its copies, however many copies lie in it | As transições `COPY_DEST -> NON_PIXEL_SHADER_RESOURCE` depois das cópias de ranges sujos são feitas uma vez por buffer; duas sombras no mesmo buffer geravam duas transições do mesmo sub-recurso numa só chamada de `ResourceBarrier`. `host.rs` (+12/−5). |
| `39031cb4` | local | README: the goal … where it stands, the renderer patches | README declara o alvo (30 fps ou mais sem ray tracing, depois mais; portar, não emular), o estado medido e a lista de patches do renderer. +32/−17. |
| `7c3d839c` | Jake | Our threads prefer the cores of the largest last-level cache where all cores are of one class (VICTORY_CACHE_CORES, on; 0: not) | Em Ryzen dual-CCD com cache empilhado (7900X3D/7950X3D/9900X3D/9950X3D) todos os núcleos reportam a mesma classe de eficiência e `fast_of` não achava núcleos rápidos. Onde as classes são iguais e os LLC diferem, o conjunto do maior LLC (≥8 núcleos) passa a contar como rápido; dois tipos de núcleo seguem por classe. Medição em 9950X3D + R9700: mediana −1,3 % (ruído). +170/−5. |
| `022db78e` | local | Merge PR #144: Trace shader and driver work on pipeline workers | Merge de `cee78fa7`: trace separado de tradução/lock e de load/create nos workers. +23/−3. |
| `4c3f311a` | local | Merge PR #147: Our threads prefer the big-cache cores on dual-CCD X3D Ryzens (VICTORY_CACHE_CORES) | Merge de `7c3d839c` mais README: `VICTORY_CACHE_CORES` documentado. +202/−22. |
| `49cb83b4` | local | Frame rate switch: VICTORY_FRAME_INTERVAL_MS … (off by default) | O título pede 33.333 µs (`SetFrameIntervalX`) porque o settings guarda 30 fps (+0x210); o commit identifica os três consumidores (+0x1e0fb30, +0x22434a0 → +0x98f84ac, +0x247ab10) e o limitador próprio em +0x120530. `VICTORY_FRAME_INTERVAL_MS=<ms>` faz `frame.rs` tiquetaquear nesse intervalo e o loader troca as duas cargas do setting por `mov eax, rate; nop`, com bytes originais conferidos; `VICTORY_TITLE_FPS` sobrepõe. +466/−4, 6 arquivos. |
| `e46151e0` | local | Merge fixup: clippy -D warnings for the code PRs #77, #106 and #69 brought | Corrige `collapsible if`, índice de `enumerate` usado só em diagnóstico e cast para o mesmo tipo; `-D warnings` volta a passar. 3 arquivos, +17/−16. |
| `79fac1ac` | local | Merge triage/prs: 33 pull requests … reviewed and merged (reports/TRIAGE-PRS-ISSUES.md) | Merge de 33 PRs triadas do próprio fork: 70 arquivos, +3877/−511, incluindo CI, `Cargo.lock`, census e `run.rs`. |
| `a12870aa` | local | Merge eval/coolfox: the AMD-safe parts of codec-d/cool_fox … (reports/COOLFOX-EVAL.md) | Traz o que é seguro em AMD do `codec-d/cool_fox`: compose de dois planos, TablePlan, correção de barreira; residency, fallback heaps e refill de XMA ficam atrás de switch, desligados. 16 arquivos, +1438/−159. |
| `0ce46d5a` | local | Two-plane compose on by default (VICTORY_PLANES=0 turns it off) | Muda o padrão de `VICTORY_PLANES` para ligado; a composição só age quando há segundo plano apresentado. Duas linhas em `compose.rs`. |
| `08b88b82` | local | Frame rate switch: the timers' shortest step follows the rate too (+0x2241fd0) | No run fps-1, com o relógio nosso a 16,667 ms e o título a 30, cinco dos sete timers andavam 33,3 ms por frame e o tempo de jogo corria ao dobro do relógio de parede. +0x2241fd0 é a terceira leitura do setting; passa a devolver a taxa pedida (1000 = sem limite), com bytes originais conferidos. +101/−33. |
| `b4ba4bbb` | local | Light and texture probes; the stand-in's slow mean by the clock (VICTORY_AMBIENT_SETTLE, off) | `lightprobe.rs` (`VICTORY_LIGHT_PROBE=<csv>`) emite uma linha por frame do que `PS_CalculateExposure` e `ComputeLuxCompressionInfo[Smooth]` leem/escrevem e da média lenta do stand-in; `texlate.rs` mede quanto a textura leva para aparecer (slot escrito → upload, primeiro uso → nível 0, última busca com memória ausente → upload); `ambient.rs` faz a média lenta (20 % da estimativa por pixel) seguir por constante de tempo. 9 arquivos, +740/−43. |
| `bf7833a9` | local | Light probe: the lux program only with VICTORY_LIGHT_PROBE_LUX=1 (run light-21 lost the device in that pass); texture times split by waits for memory | O probe do programa de luxo fica atrás de switch porque o run light-21 perdeu o dispositivo nesse passe; tempos de textura separados por espera de memória. +18/−28. |
| `722c0e1c` | local | Merge port/fps: frame rate unlock (VICTORY_FRAME_INTERVAL_MS, off by default) … (reports/FPS-UNLOCK.md) | Merge de `49cb83b4` + `08b88b82`: pacing nosso e as três leituras do alvo de 30 fps do título, byte a byte conferidas; relógio de jogo permanece 1,00×. +534/−4. |
| `9386b828` | local | Textures: a store of larger levels by the bytes of the smaller ones (VICTORY_TEXTURE_STORE=<folder>, off); looks at slots waiting for memory by the clock (VICTORY_TEXTURE_LOOK_MS, off) | `texstore.rs` guarda textura subida inteira do nível 0 (memória até 4 GB e pasta) sob chave de forma e digest dos níveis menores (≥32 KiB); quando ela é vista primeiro por um nível menor com esses bytes, recebe cópia host inteira e as views mostram do nível 0. O backoff de buscas em slots com memória ausente, que era ~4 s contados em passes (até 256 frames, 13–18 s), passa a ser limitado por `VICTORY_TEXTURE_LOOK_MS`. 7 arquivos, +477/−2. |
| `7e82cf51` | local | Texture store: detile, digest and write on its own thread | Detile, digest e escrita do store saem das threads de refresh para uma thread própria — com isso o run light-23 andou ~10 ms mais lento por frame; preenchimentos do store contam como mostrados por inteiro. +83/−45. |
| `50f72024` | local | Merge port/light: the light stand-in settles by time … (reports/LIGHT-POPIN.md) | Merge de `b4ba4bbb`+`bf7833a9`+`9386b828`+`7e82cf51`, com conflito resolvido em `draw.rs`: stand-in por tempo, store em disco, probes. 11 arquivos, +1245/−45. |
| `9576c45f` | local | Light stand-in settles by time by default (VICTORY_AMBIENT_SETTLE unset = 250,3; 0 = per frame as before) | Muda o padrão: sem a variável, o stand-in assenta por tempo (250 ms, salto 3) e salas deixam de ficar claras/escuras por segundos após entrar. Uma linha em `ambient.rs` (+4/−2). |
| `e151e71f` | local | Output size and render scale switches (VICTORY_OUTPUT, VICTORY_RENDER_SCALE, VICTORY_UPSCALER; off by default), byte-checked | O título tira o tamanho de saída do settings e aplica em +0x224352b via setter +0x2478130, com back buffers em +0x247ad10. `VICTORY_OUTPUT=<w>x<h>` escreve o tamanho no settings e segue o caminho do título; `VICTORY_RENDER_SCALE=0.5..1.0` troca os 2/3 do upscaler próprio (+0x28f1e78) e a escala do FidelityFX (+0x28ef680, PR #53); `VICTORY_UPSCALER=<6\|7>` aplica o picker do PR #53 (+0x7e93f0). Cada site confere bytes antes. +395, 3 arquivos. |
| `45cad4c5` | local | VICTORY_OUTPUT: jump over the back buffer pool's left-over check (+0x247af25) for sizes other than 4K | O título faz back buffers num pool de 64 MB (Series X) e para num `int3` se sobrar ≥2 MB; a 1920×1080 sobram 49 MB (run res-1 parou 2 s depois). Pula a checagem. +25/−1. |
| `9ba5661f` | local | Replay: buffers placed in heaps opened from title memory are made again; detailed timing sums the commands of our own lists apart | Heaps são criados sem compartilhamento entre adaptadores, então recursos posicionados neles não podem pedi-lo: os buffers que o port 1 posiciona no heap aberto da memória gráfica do título eram recusados ("parameter incorrect", 1.326–3.576 por captura). Passam a ser criados de novo. Com `--timing detailed`, lista sem shader do título tem comandos somados em "(lists of ours)". +41/−2. |
| `3920dc26` | local | Renderer port 5 (VICTORY_PORT_CBV=1, off by default): constant buffers read as raw bytes, the title's in place given to the card where they are | O console alinha CB em 16 bytes e o D3D12 em 256, então cada CB nomeado pelas tabelas era copiado a cada submissão (~23.000 por frame, ~9 MB); raiz fora de 256 bytes, 64 KiB cada. Com o switch, shaders traduzidos leem CB como bytes crus: tabela pelo gêmeo raw do slot, raiz como root SRV; CB de tabela em memória do título vira view raw no lugar (`constant_raw_at`), sem cópia nem passo de submissão. 9 arquivos, +477/−7. |
| `4d7185ed` | local | Render scale: VICTORY_RENDER_SCALE for the title's own upscaler only; FidelityFX's as VICTORY_FFX_SCALE (not safe) | Series S renderiza por FidelityFX: com escala 0,5 (run res-6) o jitter phase count voltou 0 e o título dividiu por ele em +0x28f03e8, 49 s depois. A escala do FidelityFX fica atrás de switch próprio e marcada como insegura; o upscaler do título (Series X) rodou a 1,0 no run res-4. +26/−6. |
| `f63b4b12` | local | Merge port/res: VICTORY_OUTPUT, VICTORY_RENDER_SCALE, PR #53's FidelityFX scale/picker patches; all off by default (reports/RESOLUTION.md) | Merge de `e151e71f`, `45cad4c5` e `4d7185ed`: saída 1080p/1440p, escala de render e picker. +439 em 3 arquivos. |
| `0dec5dcc` | local | Merge port/gpupass: renderer port 5 … ~22,000 fills a frame -> 0 (reports/GPU-PASSES.md) | Merge de `3920dc26` + `9ba5661f`: CB lidos como bytes crus e replay criando recursos em heaps da memória do título. 10 arquivos, +518/−9. |
| `9d723e88` | local | Renderer port 6 (VICTORY_PORT_GUARD=1, off by default): the guards' lock shared by faults and refreshes … | Os conjuntos de blocos guardados são palavras atômicas e `GUARDS` vira lock leitor-escritor: fault de escrita, refresh do range e bloco tornado conhecido tomam o lock compartilhado e os claims (bits BUSY) dos blocos que mudam; faults esperam só quem muda o mesmo bloco, em ordem de bloco, refresh só tenta o seu. O que muda mais de um bloco (mapeamentos, leituras de kernel, memória liberada, buracos) mantém o lock exclusivo. Memória de view vai direto aos guards em vez de perguntar ao kernel. +683/−98. |
| `2edf8446` | local | Renderer port 6: the guards' first looks of an asking (a view? all clean?) under the shared lock too | As primeiras olhadas de um pedido passam ao lock compartilhado; +20/−9 em `watch.rs`. |
| `65f9707d` | local | Renderer port 6: view memory a caller asks of the kernel alone (textures) is not asked: the kernel refuses it | Memória de view pedida só ao kernel (texturas) deixa de ser pedida aos guards; o kernel recusa. +6. |
| `00ab2acf` | local | Renderer port 6: a submission's look at which refreshes copy nothing goes to the threads beside the submitting one … | Uma submissão por frame gastava até 6 ms nessa olhada sozinha (run guard-3: 5,8 ms sem nada mais na lane) enquanto as seis threads ao lado esperavam; acima de 256 refreshes já era feita ao lado. Passa a ir para as threads vizinhas; spans do planejamento entram na timeline. +32/−3. |
| `8f02bf74` | local | Renderer port 6: a submission's planning of its refreshes in front made per list on the threads beside the submitting one and joined in order | Run guard-4: a submissão gastava 2,9 ms juntando passos e 3,9 ms nos spans. Cada lista dobra seus passos ao lado (`fold_touches`: span dos refreshes antes da primeira escrita por shaders, e se escreve) e as dobras são juntadas em ordem (`join_keys`), igual a `front_spans`, provado em 3.000 submissões aleatórias. +220. |
| `88b63a2c` | local | Renderer port 6: what the switch does, as measured (VICTORY_GUARD_HOT off: worse in run guard-6) | Comentário/doc do switch com a medição: `VICTORY_GUARD_HOT` desligado foi pior no run guard-6. +6/−3. |
| `cc761bf3` | local | Merge port/guard: renderer port 6 … 47.2 -> 45.1 ms (reports/PORT-PATCH-6.md) | Merge de `9d723e88`…`88b63a2c`: lock compartilhado com claims por bloco, view memory sem kernel, planejamento por lista. 2 arquivos, +964/−111. |
| `a9400a6d` | local | README: where it stands after renderer ports 5 and 6 (45.6 ms, about 22 fps, Series S, run merge-4) | README atualiza o estado (45,6 ms, ~22 fps, Series S) e lista os switches da noite. +20/−4. |

## Pull requests abertos (6)

Todos os seis com número acima de 379; nenhum PR mesclado na janela (`last_run` de 08:17:50Z).

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 380 | Prepare time on an 8-thread CPU: per-frame benchmark, A/B of the renderer ports, #55 merged for RDNA 4 | Mede tempo de preparo na thread de submissão num 9800X3D com SMT off e RX 9070 XT (RDNA 4), com benchmark por frame e A/B de switches; nenhum padrão muda. +1010/−61, 19 arquivos. | Não. Instrumentação; toca o **D-08** (compilação de pipeline travando o título) só ao medir. |
| 383 | victory-gpu: record texture copy-back ranges as card-steps writers | `Fixes #320`: ranges de copy-back de textura entram em `card_steps::writers` como `Writer { copy: false }`, então `holder()` recusa leitura/escrita de card-step que sobreponha copy-back pendente em vez de ler cópia host velha; teste com ranges sobrepostos e disjuntos. +43. | Não no catálogo; é a classe do **D-07** (copy-back sobrescrevendo escrita da CPU). |
| 384 | Renderer port 2: a copy into a record's later words marks its slot (missed view records 101 -> 6 a frame) | Com `VICTORY_PORT_DESCRIPTORS=1`, o `memcpy`/`memmove` importado só marcava o slot se a cópia tocasse a primeira palavra (tag); cópia que reescreve endereço/tamanho não marcava e o registro ficava com a view antiga até o fallback passar pela página. Marca também palavras posteriores: 101 → 6 registros perdidos por frame. +28/−6. | Não. |
| 385 | Diagnostics: renderer port 2 trace watches a missed record's writers (VICTORY_PORT_DESCRIPTORS_TRACE=1) | Empilhado em #359: o trace nomeia o escritor de um registro não marcado por escritas no bloco de 64 KiB, com `place()` (módulo+offset); traz o commit 8436b3f de #359 e o próprio 5c1c9be. +675/−29. | Não. Diagnóstico. |
| 386 | Linux under Wine: 23.5-25.5 fps on an RX 7900 XTX; pipeline workers' stack, thread names in the log, docs/linux-wine.md | Relato medido em Wine 11.22 + RADV: 23,5 fps release na cela (25,5 ao fim de 5 min), 17 em diag, contra 7,3 do `fxceless/victory-fork`; inclui stack dos workers, nomes de thread e doc. +74. | Não. |
| 387 | Diagnostics: the card's clock counts list-translation waits on their own, and copystat gives bytes copied beside bytes looked at | `Fixes #366`: com `VICTORY_THIN=1` (padrão do `play.ps1`) a submissão espera na fila até as listas serem traduzidas, e essa espera era contada como tempo do título; passa a contar à parte. `copystat` passa a dar bytes copiados ao lado de bytes olhados. +59/−16. | Não. Diagnóstico; confirma que `play.ps1` já difere do documentado (**D-19**). |

## Pull requests mesclados (0)

Nenhum PR mesclado desde `2026-11-26T08:17:50Z`. Os merges desta faixa são de branches internas
(`Merge PR #144`, `Merge PR #147`, `Merge triage/prs`, `Merge eval/coolfox`, `Merge port/*`), não de PRs
do GitHub com `mergedAt` na janela.

## Cruzamento com o catálogo

- **D-10** (correção do replay feita e revertida, buffers em heaps abertos da memória): `9ba5661f`
  **refaz** a correção — buffers posicionados em heaps abertos da memória do título passam a ser criados
  de novo, com o motivo da recusa nomeado ("parameter incorrect", 1.326–3.576 por captura). O catálogo
  deve passar `corrigido` para `9ba5661f`, com a ressalva de que o commit é de 2026-11-26 e não há
  evidência de run que confirme a ausência da recusa.
- **D-09** (device loss a cada ~16 min, textura mostrada antes do upload): `b4ba4bbb`/`bf7833a9` medem
  quando a textura aparece e `7e82cf51` aponta que o store atrasou ~10 ms por frame; **nada corrige** a
  ordem upload→exibição. `9386b828` melhora o caminho, não o invariante.
- **D-16** (duas convenções opostas para switch, 294 variáveis): a faixa **agrava** — convivem
  `VICTORY_PLANES` (ligado por padrão, `=0` desliga), `VICTORY_PORT_CBV=1`/`PORT_GUARD=1`/`PORT_JOURNAL=1`
  (opt-in), `VICTORY_AMBIENT_SETTLE` (não definido = 250,3; `0` = por frame) e `VICTORY_TEXTURE_STORE=<pasta>`.
  Quatro convenções distintas em `compose.rs`, `ambient.rs`, `texstore.rs` e nos ports.
- **D-18** (constantes de layout fixas por título): `e151e71f` e `45cad4c5` fixam RVA e offsets
  (`+0x224352b`, `+0x247af25`, `+0x28ef680`, `+0x7e93f0`) com checagem de bytes — mitiga, mas amplia a
  superfície de constantes por build.
- **D-19** (configuração documentada ≠ medida): `0ce46d5a` muda um padrão (planos ligados) e o PR #387
  registra que `VICTORY_THIN=1` é padrão do `play.ps1`; o README de `a9400a6d` lista switches, não os
  padrões efetivos do script.
- **Abre defeito novo?** Dois candidatos:
  - `4d7185ed` — escala 0,5 no FidelityFX do Series S produz `jitter phase count` 0 e divisão por zero em
    `+0x28f03e8` aos 49 s (run res-6); `crates/victory-loader/src/resolution.rs`.
  - `a6064651`-like não se aplica aqui; o outro é `9386b828`/`7e82cf51`: o store em disco pode atrasar
    frames em ~10 ms quando ligado (`crates/victory-gpu/src/texstore.rs`).

## Confiança

Verificado: 58 hashes e metadados via `gh api .../compare/f03bfa51...126acb35`; os 40 documentados lidos
em `.../commits/<sha>` com arquivos e contagens; os 6 PRs abertos lidos em `.../pulls/<n>` (corpo,
`additions`, `changed_files`, `draft`); ausência de mesclados conferida em `--state merged --limit 50`
filtrada por `mergedAt > 08:17:50Z`.

Não verificado: as medições citadas nos corpos (fps, ms, contagens de recusa) — são relato do autor, não
reproduzidas; se o merge de `upstream/main` (`a6a6ef73`) deixou a árvore compilando; o conteúdo dos 18
commits fora do teto (`673be0ea`…`126acb35`), que ficam no `backlog`.
