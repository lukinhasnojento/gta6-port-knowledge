# Acertos e defeitos de `fxceless/victory-fork`

> Medido no repositório local, HEAD `6d20eed6ae8afe37c52f1911b46537b0ea8bcd0d` (2026-11-25 18:02:01 -0500).
> Todo número tem o commit de onde veio. Onde só existe a mensagem do commit, isso é dito na própria
> linha. Onde não há evidência, está escrito `NÃO VERIFICADO`.

---

## Ficha

| Campo | Valor |
|---|---|
| Repositório | `fxceless/victory-fork` |
| Commits | 253 em `main` (160 não-merge, 93 merge): 213 rehospedados do VIctory + 40 próprios do fork, de `578d459` a `6d20eed` |
| Licença | `GPL-3.0` (`LICENSE` = GNU GPL v3, 29 June 2007; API do GitHub: `spdx_id: GPL-3.0`) |
| Papel no ecossistema | A camada Xbox→PC com a instrumentação e a disciplina de switches mais completas entre os forks: 3.057 estrelas e 877 forks na API do GitHub em 25/11/2026, o repositório mais observado do conjunto. Ser "a base da maioria dos builds privados de topo" e o resultado de 59,4% do build líder no PortWatch: **NÃO VERIFICADO** dentro deste repositório (nenhum placar, log externo ou identificador de build) |
| Estado hoje | `6d20eed`: a primeira sala renderiza com paredes, texturas, pessoas e luz de lâmpada a 8–10 fps — mas medido com um conjunto de 13 switches que **não** é o default, com sombra preta, memória de 24 GB por processo e perdas de device sem causa aberta |

---

## Placar

Cinco dimensões, nota de 0 a 5, com a justificativa em uma linha cada.

| Dimensão | Nota | Justificativa |
|---|---|---|
| Engenharia | 4/5 | Quatro consertos de causa raiz de nível alto — chave de cache derivada do código (`8dd30ab`), guarda em todas as views de uma página (`8dd30ab`), cópia in-place com limiar e quarentena (`0b0074e`), hash por grupo de 64 threads com dobra ordenada (`5557224`) — contra duas tentativas de page mapping que quebraram o streaming (`d510ee4`) e uma otimização barrada por defeito não compreendido (`c36971b`) |
| Verificabilidade | 2/5 | Existem 619 `#[test]` em 162 módulos de teste e CI com clippy `-D warnings` desde `8e79c80`, mas os commits de integração declaram literalmente "this merged tree was built; its test suites were not run" (`a26859f`, `9376fd6`, `f85101b`, `a253706`, `c36971b`) e o número de default nunca foi remedido (`6d20eed`, README:45) |
| Documentação | 4/5 | README de 48 KB que registra fracassos tanto quanto ganhos, publica uma retratação (`0b0074e`), mantém uma seção de "medido e não deu nada" e avisa "Not playable yet" (README:33) — mas os 8–10 fps de capa vêm de switches não-default, há números de duas máquinas sem rótulo por linha e uma referência a `TODO.md` inexistente (`display.rs:62`) |
| Sustentabilidade | 2/5 | 247 nomes de switch distintos em 260 sítios de leitura (minha contagem; 219/246 na análise), 164 marcadores `shortcut:`, lints afrouxados em `6a2d9f9` (`undocumented_unsafe_blocks = "allow"` mais 8 lints de estilo), push direto em `main` e perdas de device abertas |
| Reaproveitamento | 4/5 | Cinco módulos de instrumentação e quatro padrões diretamente transferíveis para uma camada unificada, mas tudo fortemente acoplado a um título e a uma GPU, e a configuração não tem schema para herdar |

**Média:** `3.2/5`

---

## Acertos

O que o projeto fez **certo**, com evidência. Ordene por valor.

| # | Acerto | Commit | Por que importa | Reusável? |
|---|---|---|---|---|
| A-01 | **A chave do cache de shader é derivada do próprio código**: `build.rs` varre `crates/victory-shader/src/**/*.rs` atrás de todo literal `"VICTORY_`, emite `VICTORY_SHADER_SWITCHES`, e `translation_fingerprint()` (nome, valor e NUL de cada switch + hash FNV da fonte + `host_max_wave`, `view_heap_limit`, `loop_limit`, `group_sync`, `shader_opt`) entra na chave em `shader_cache.rs:75` | `8dd30ab` (versão manual dos quatro switches de wave em `e9765a7`) | Trocar qualquer flag invalida o cache sozinho: nenhum experimento "desligado" pode rodar o artefato "ligado". Nasceu porque `6a2d9f9` ligou wave64 wide sem entrar na chave e serviu tradução errada do cache (F-03). Verificado no código: `build.rs:51`, `emit/mod.rs:199`, `shader_cache.rs:75`. Medida independente: **247 literais distintos** de `VICTORY_...`, 260 sítios `env::var`/`env::var_os` (a análise registra 219/246 — a diferença é de método de contagem, não de código) | sim |
| A-02 | **Guarda sobre todas as views de uma página física**, não só o endereço que a cópia tomou; o handler devolve acesso a todas. `VICTORY_VIEW_GUARD=0` desliga | `8dd30ab` | Fecha uma classe inteira de corrupção silenciosa: **0,8 a 1,0 milhão** de escritas passando por outra view em cinco minutos de run, invisíveis para a cópia host. Código creditado a myokex/victory-amd `c6dc4d8`, PR #115 e VIctory #1122 | sim |
| A-03 | **Cópia in-place em assembly com handler somente-leitura**: `victory_copy_guarded` (`rep movsb`, sem pilha, `rsi`/`rdi` preservados em `r9`/`r10`), dois pontos de retorno rotulados; o handler só toma leitura com `RIP` exatamente no ponto, e escrita que falha **não** é tomada; `LEAST = 256` bytes e `RECENT[1024]` com validade de 2 s por bloco | `0b0074e` (módulo criado em `a253706`) | Separa "copiar" de "perguntar se a página existe": 1.000–1.300 faults/frame → **4 a 9**, kernel da thread de submit **44 → 20 ms/frame**, imagem igual. O par (limiar de tamanho, quarentena por bloco) é a parte transferível | com ressalva (x86-64 + handler vetorizado de Windows) |
| A-04 | **Hash de peça por grupo de 64 threads com dobra em ordem fixa de lane**: thread `t` hasheia os passos `t, t+64, t+128…`, a thread 0 dobra os 64 hashes em ordem de lane | `5557224` (opt-in em `6d20eed`) | Transforma latência serial em paralelismo: passes de **35 → ~10 ms de placa**; frame 197/231 → 163/183 ms. Mantido opt-in porque é **outro hash** ("another hash with another rare pattern of changes it cannot see") — decisão correta: somar hashes na GPU só vale se store e compare usarem a mesma forma | sim (padrão canônico de hash paralelo com redução ordenada) |
| A-05 | **Driver watch como canário externo**: assina o event log do Windows (`EvtSubscribe`), conta e loga; o primeiro evento marca `TRIPPED`, e `draw::windows()` consulta `tripped()` **fora do `OnceLock`**, a cada chamada | `a253706` | Troca "perder a placa" por "andar mais devagar", usando um sinal que existe **fora do processo**: entradas `nvlddmkm` 153 de 2,7 a 6,3 s antes da remoção (`drvwatch.rs:5-6`) | sim |
| A-06 | **Regra de staleness medida, não intuída**: `Rule::Start` invalida a view antiga só quando a nova começa no mesmo endereço ou dentro dos primeiros 4 KiB (`HEAD = 4096`); modos antigos ficam como `VICTORY_STALE_TEXTURES=overlap` e `=0` | `310c6eb` | Corrige paredes ausentes e dezenas de milhares de slots nulos por run, com o racional numérico no diff: "of 129 pairs of overlapping neighbours in the listings of runs 188 and 189, none starts closer than 0x2000". Mudar o default **e** deixar o antigo selecionável é o que permite A/B depois | sim |
| A-07 | **Todo heap de views termina em um slot nulo**: `UNCAPPED_VIEWS = 1 << 21` quando o host aceita, senão `PORTABLE_VIEWS = 1_000_000`; o limite escolhido entra na chave do cache | `31714de` | Índice de view além do fim do heap travava a GPU na NVIDIA e removia o device na AMD (slot 16.777.088 de um heap de 1.507.328). Clampar "from the first shader on" elimina a classe, e pôr o limite na chave impede shader traduzido com limite antigo | sim |
| A-08 | **Reexame de textura enviada inteira** nos frames 4, 12, 40, 120 e 400, por fingerprint de 48 KiB da memória; se mudou, reenvia e reinicia a série (`VICTORY_HEAP_RECHECK=0` desliga) | `a803c87` | Conserta rosto preto e cabelo sem cor em ~3 de 4 lançamentos: a textura era enviada quando o título escrevia a view, e o título preenche a memória depois. Um lançamento reenviou **mais de 6.000** texturas assim. O autor declara o limite da própria amostragem | com ressalva (amostragem incompleta, declarada) |
| A-09 | **Guard de orçamento no indirect**: registro de 2^23 vértices por instância ou mais roda vazio e não cobra orçamento da lista, mais o checker `VICTORY_INDIRECT_WINDOW_CHECK` | `f85101b` | Fecha a regressão mais cara do fork: janela copiava registros de cauda não desenhados (`0xFFFFFFFF`) e a placa era perdida 2–3 vezes por run; o checker do próprio PR achou "one list drawing 90 records of 2^24 vertices". Depois do guard: cinco runs com janelas, sem perda | sim |
| A-10 | **Texturas escritas por shader voltam para a memória do título**, tileadas no layout do console | `895c0b8` (cherry-pick), `f5c03f3` (merge do PR #999, `VICTORY_TEXTURE_BACK=1`, off) | É o que faz o barco flutuar: as passes de oceano escrevem height/displacement map e o código CPU do título **amostra** os dois; sem o retorno, a primeira missão falhava de imediato (run 275). 22.900 read-backs em seis minutos no run 276 | sim |
| A-11 | **Retratação de número publicado + troca de método**: "6,0 → 8,9 fps" é retirado e substituído por reta de tempo de frame contra draws por frame | `0b0074e` (número publicado em `a253706`) | Em arqueologia de performance, retirar o próprio número vale mais que acertá-lo: as duas execuções estavam em cenas diferentes (4.300 contra 1.550 draws/frame). O método novo (145 ms a 4.500 draws, 271 ms a 10.000) é reutilizável e sobrevive à comparação seguinte | sim |
| A-12 | **Instrumento antes da otimização**: `framelog.rs` (composição de cada frame), `cardmeter.rs` (tempo da placa por tipo de lista), `profile.rs` (profiler amostral), `shot.rs` (captura dentro da camada), `backreads.rs` (quem leu o que o copy-back gravou) | `a253706`, `f85101b`, `79cfdd9`, `c36971b` | Foi o instrumento que achou que o "dip" periódico era o próprio harness (`a253706`) e que 497 GB de copy-back por 49 min tinham 60% de stores sem leitor (`c36971b`). Sem esses cinco arquivos, todos os outros números do fork seriam opinião | sim |
| A-13 | **Versão ingênua mantida como oráculo testado**: `changed_runs_bytes` (byte a byte) ao lado de `changed_runs_wide` (máscara de 64 bytes), com dois testes de equivalência | `6d20eed` | É a forma mais barata de otimizar um diff sem perder a referência: a implementação lenta continua no código e é a especificação executável da rápida | sim |
| A-14 | **Warm-up offline com chave por fonte do tradutor**: `victory translate --warm <hashes> --jobs N` (4,5–6,5 min) enche o cache antes do run | `8dd30ab` | Separa "o que já foi traduzido" de "qual binário está rodando"; um build que não toca o tradutor começa quente. Depende de A-01 estar correto — os dois foram projetados juntos | sim |
| A-15 | **Log de diagnóstico real, higienizado e publicado**: `diagnostics/timing-run-2026-11-25.txt` (60 s) com nomes de usuário, pastas, título, title id, rede e shaders substituídos por placeholders; hashes preservados | `44bff61` | Log anonimizado vale mais que tabela de números: permite a terceiro reconstruir a configuração do run a partir dos switches que logam ao ligar | sim |

---

## Defeitos

O que o projeto fez **errado**. Ordene por gravidade.

| # | Defeito | Gravidade | Commit | Estado |
|---|---|---|---|---|
| F-01 | **Perdas de device sem causa identificada**, em pelo menos quatro configurações: run 276 (6 min, dois eventos de driver e hang), run 277 (49 min, `device removed` 8 ms depois de um sweep liberar 150 cópias de textura), a primeira sessão no mundo aberto (9,2 de 10,3 GB de orçamento, 2.073 passes sem textura) e o conjunto com `VICTORY_TEXTURE_BACK=1` (run 275 sem o switch jogou 20 min sem erro; n=1 de cada lado). Três hipóteses registradas, nenhuma fechada | ALTO | sem commit atribuível; evidência em `a253706` (doc de `drvwatch.rs`), `f5c03f3` e README:98-99 | aberto |
| F-02 | **Dívida de verificação**: a árvore mergeada é compilada e as suítes não rodam nela; os testes citados passaram na árvore de onde as mudanças vieram | ALTO | `a26859f`, `9376fd6`, `f85101b`, `a253706`, `0b0074e`, `c36971b` (literais) e `8dd30ab` ("its test suites were run on the tree the changes come from (gpu 129, hle 97)") | aberto |
| F-03 | **Chave do cache de shader não incluía os switches de wave**: como a chave era o BLAKE3 do **próprio executável**, um run com `VICTORY_WAVE_WIDE=0` recebia as traduções que um run anterior do mesmo binário fizera com o switch ausente — o experimento "desligado" rodava o código "ligado" | ALTO | introduzido `6a2d9f9`, corrigido `e9765a7` | corrigido |
| F-04 | **Forma wave64 "wide" ligada por default** (`is_none_or(\|v\| v != "0")`) põe barreiras de grupo dentro de branches/loops: 4 de 4 execuções perdem o device logo após o início; `ptfx_CS_lightAtlas_Sprite`/`_Trail` travam a placa | ALTO | introduzido `6a2d9f9`, invertido para opt-in em `e9765a7`; sintoma documentado em `004bf69` | corrigido |
| F-05 | **Regra de invalidez por sobreposição total anulava texturas vizinhas**: `Spans` assumia "spans never overlap", mas o título empacota os níveis pequenos de uma textura onde estariam os grandes da anterior — cada view escrita tirava as vizinhas da fila de upload e zerava seus slots (paredes ausentes, dezenas de milhares de slots nulos por run) | ALTO | herdado em `578d459`, corrigido em `310c6eb` | corrigido; resíduo aberto (o primeiro shot da sala ainda às vezes vem sem paredes, README de `8dd30ab`) |
| F-06 | **Registros de descritor copiados como bytes crus não eram espelhados**: ~40.000 registros por frame com slot host vazio ou velho, contra "a few hundred views made by calls". O conserto (`a26859f`) criou uma regressão de 70–220 ms/frame, fechada em `9376fd6` (~3 ms) | ALTO | lacuna herdada em `578d459`; conserto `a26859f` → `9376fd6` | corrigido |
| F-07 | **Janelas de indirect serviam registros de lixo** (2^24 vértices) porque cobravam o orçamento no clamp por registro e não por lista; placa perdida 2–3 vezes por run | ALTO | código do PR #220 entrou em `f85101b` (o PR "is not in" em `9376fd6`), guard no mesmo `f85101b` | mitigado (guard + `drvwatch`); janelas seguem off por default |
| F-08 | **Page mapping do título: duas tentativas que quebraram o streaming e foram revertidas.** (a) "deixar em paz páginas que já mostram o que se pede" — contagem mostrou que não existem tais páginas; (b) mapear páginas consecutivas como uma view — remap 140 → 79 ms/frame, mas faces pretas ou vermelhas e lixo em 3 de 3 lançamentos. Custo atual: 32 ms de cada frame de cutscene e 246 ms de um frame de streaming | MÉDIO | `d510ee4` (README-only, "no code change in this commit") | aberto |
| F-09 | **Loads escalares agrupados de 4 dwords leem além do fim da view**: água preta no run 277 com `VICTORY_SHADER_OPT=127`; a causa é hipótese do autor ("the lead, not proven on the card"). Com o bit fora do default (63), 25.635 de 25.635 shaders são byte-idênticos ao build anterior | MÉDIO | `c36971b` | aberto — o bit 64 está fora do default e a otimização existe sem conserto |
| F-10 | **O default nunca foi remedido**: "The plain defaults were last measured at 420 to 450 ms a frame and not since". Os 8–10 fps da capa vêm de um conjunto de 13 switches que não é o default — quem compila e roda recebe algo até 4,5× mais lento que o número anunciado | MÉDIO | `6d20eed` (README:45) | aberto |
| F-11 | **Ray tracing do título não ilumina nada**: sombra preta; 59 de 64 programas de trace traduzem e os testes ray/box e ray/triangle rodam em DXIL contra referência, mas "Nothing is lit by it yet" | MÉDIO | `6a2d9f9` (`docs/fork-ray-tracing.md`) | aberto |
| F-12 | **Reexame de textura por três amostras de 48 KiB**: uma textura cujas partes chegam fora do começo/meio/fim continua errada — limite declarado pelo próprio autor no commit | MÉDIO | `a803c87` | aberto (parcial por desenho) |
| F-13 | **164 marcadores `shortcut:`** no código Rust — 164 pontos em que o código declara aproximar o console em vez de reproduzi-lo, sem índice do que cada um custa em fidelidade | MÉDIO | `6d20eed` (contagem minha, `Select-String -CaseSensitive 'shortcut:'`) | aberto |
| F-14 | **Lints afrouxados**: `undocumented_unsafe_blocks = "allow"` "until the newer code is tidied", mais 8 lints de estilo (`type_complexity`, `needless_range_loop`, `chunks_exact_to_as_chunks`, `sliced_string_as_bytes`, `collapsible_match`, `manual_clamp`, `implicit_saturating_sub`, `min_max`) | BAIXO | `6a2d9f9` (`Cargo.toml`) | aberto |
| F-15 | **Referência a documento inexistente**: `crates/victory-hle/src/modules/display.rs:62` remete a "the presentation items in `TODO.md`", e `TODO.md` não existe em `git ls-files` | BAIXO | `6d20eed` (HEAD) | aberto |
| F-16 | **Sem teste dedicado para a auto-cópia proibida** (duas texturas do título que compartilham um recurso host), num caminho de cópia que foi reescrito depois em `queue2` e no window upload | BAIXO | `a253706` | **NÃO VERIFICADO** (a ausência foi procurada por `git log -S`, não provada) |

---

## Erros de processo

Fatos sobre **como** o projeto trabalha, não sobre o código.

| # | O que aconteceu | Evidência |
|---|---|---|
| P-01 | **"This merged tree was built; its test suites were not run"** virou fórmula. Seis commits de integração dizem isso com essas palavras ou equivalentes, e um sétimo diz que as suítes rodaram "on the tree the changes come from". Consequência: os números de **desempenho** são confiáveis (medidos em jogo), os de **corretude** são menos | literais em `a26859f`, `9376fd6`, `f85101b`, `a253706`, `0b0074e`, `c36971b`, `8dd30ab` |
| P-02 | **O gate existe e não é usado.** `.github/workflows/ci.yml` roda clippy `-D warnings` nos dois feature sets e `cargo test --workspace` em pull request **e** em push para `main` desde `8e79c80`; mas os commits do fork entram por push direto (só dois merges de PR próprios em 40 commits: `f5c03f3`, `1391409`). O único hook instalável (`.githooks/pre-commit` → `cargo xtask policy --staged`) verifica política de assets, não build nem testes | `.github/workflows/ci.yml`, `8e79c80`, `.githooks/pre-commit` |
| P-03 | **Número publicado comparando cenas diferentes**: "draw windows measured at 6.0 → 8.9 frames a second" (uma execução de cada lado, cenas com 4.300 contra 1.550 draws/frame), retirado no dia seguinte | publicado em `a253706`, retratado em `0b0074e` ("compared two runs that stood in different scenes … and are withdrawn") |
| P-04 | **"Ganho de placa" virou "ganho de frame" três vezes** sem que a diferença fosse dita na hora: PR #1014 (35 → 10 ms de placa, frame 197/231 → 163/183), `CARD_BUFFERS_MIN_KB=64` (−18 ms de placa, −5 de frame), window upload (−1,9 GB de memória, "A memory gain; no speed gain shown") | `5557224`, `c36971b`, `79cfdd9` |
| P-05 | **Reverter e reaplicar a mudança #11 em ~60 segundos**: `9e49b8a` a tira ("the lighting gain could not be shown, it costs about 40 ms of GPU time per frame"), `2119b0e` a devolve ("The previous commit took it out; that was not wanted"). A "árvore boa conhecida" citada no plano de verificação de imagem muda de definição duas vezes na mesma sessão | `9e49b8a`, `2119b0e`, `90228a7` (versão com look-ahead) |
| P-06 | **História reescrita e delta comprimido**: 213 commits do VIctory rehospedados abaixo de um único commit de squash do fork (`578d459`, 61 arquivos, +32.334/−1.349), sem merge-base com o HEAD do Marvisak — o HEAD do Marvisak não é ancestral deste repositório. Custo: nenhuma comparação ou cherry-pick por hash é possível contra a base; toda integração é por blob | `578d459`, `FORK_DIVERGENCIA.md` (`git merge-base` falha), `git diff --shortstat ffe310e HEAD` = 117 arquivos, +83.615/−1.778 |
| P-07 | **README de um commit se autocorrige dentro do próprio commit**: `79cfdd9` remove "This push is the README only: the code is not in this tree yet" e, no mesmo diff, afirma que o código está na árvore. A mudança de estado real fica sem commit próprio e sem registro de quando o código entrou | `git show 79cfdd9 -- README.md` |
| P-08 | **Números de duas máquinas intercalados sem rótulo por linha**: o baseline declarado é RTX 4070 12 GB, mas o PR #1014 foi medido em "an RTX 3060 12 GB (Ryzen 5 5600GT)" e os 35 → 10 ms entram no README ao lado de números de 4070 | `5557224` (mensagem), README:45 |
| P-09 | **Medição de switches nunca normalizada**: 247 nomes distintos e 260 sítios de leitura, sem schema, sem `--help` gerado e sem validador; pares que precisam andar juntos (`changed_kept`/`changed_list`, `card_hash_wide`/`card_compare_groups`) só existem como conhecimento tácito | contagem minha sobre `*.rs` de `6d20eed`; módulos `shadow.rs`, `texback.rs` |
| P-10 | **Escopo declarado vs. escopo real**: o README de HEAD diz "the title" e não nomeia o jogo (mudança cosmética de `4e17478`), mas mantém 1112 issues abertas no GitHub e uma tabela de status com números de 23/11 que o fork nunca atualizou (mesh 8.191 de 8.506, vertex/pixel 10.617 de 12.346, compute 1.643 de 2.539) | `4e17478`, README (tabela herdada), API do GitHub em 25/11/2026 |

---

## O que copiar

- **`crates/victory-shader/build.rs` + `translation_fingerprint()` + o uso em `shader_cache.rs:75`** (`8dd30ab`). É o acerto de maior valor do conjunto: a lista de flags que mudam a saída é gerada a partir do código, então um switch novo entra na chave sem ninguém lembrar. Levar junto a ressalva do próprio desenho: só cobre literais `"VICTORY_` dentro de `crates/victory-shader/src`, e por isso `view_heap_limit()` precisou ser listado à mão.
- **A guarda em todas as views de uma página física** (`watch.rs`, `resource.rs`, `8dd30ab`). Vale para qualquer emulação de memória compartilhada com alias.
- **`crates/victory-gpu/src/plaincopy.rs`** (`0b0074e`): `rep movsb` sem pilha, handler só de leitura com `RIP` exato, `LEAST = 256`, quarentena de 2 s por bloco.
- **`crates/victory-gpu/src/drvwatch.rs` + a consulta por chamada em `draw.rs::windows()`** (`a253706`): canário externo que degrada antes do crash.
- **O hash de peça por grupo de 64 threads** (`5557224`) como padrão, com a regra que o commit impõe: um processo usa **uma** forma de hash para tudo que armazena e compara.
- **`crates/victory-gpu/src/descriptor/host.rs` `Rule::Start` + `HEAD = 4096`** (`310c6eb`), com os modos antigos preservados para A/B.
- **`tiles.rs` (levels on demand)** (`8dd30ab`, `a26859f`): alinhar a alocação ao que o título mantém, não ao tamanho declarado da cadeia.
- **Os cinco instrumentos**: `framelog.rs`, `cardmeter.rs`, `profile.rs`, `shot.rs`, `backreads.rs` (`a253706`, `f85101b`, `79cfdd9`, `c36971b`). `backreads.rs` é a técnica mais transferível do repositório: amostrar o que a CPU leu via `VirtualUnlock` + `QueryWorkingSetEx`, sem tocar em proteção e sem mudar um byte.
- **A regra de switch, não a forma**: default off até rodar em jogo, `=alt` para A/B dentro de um run, log ao ligar, flag de emissão entra na chave do cache, switch morto fica desligado e documentado.
- **O formato do log higienizado** (`44bff61`, `diagnostics/README.md`): um log real anonimizado vale mais que uma tabela de números.

## O que não copiar

- **Os 247 switches e 260 sítios de leitura sem schema.** Triplicação de parsers (`match env::var(...) { Ok("1") => 1, Ok("alt") => 2, _ => 0 }`), pares que quebram se ligados pela metade. Funciona para um operador único; num repositório unificado é configuração sem contrato.
- **Integrar sem rodar a suíte na árvore integrada** (P-01). Se o número de desempenho é medido em jogo, o de corretude tem que ser medido onde o código está.
- **Números de fps de mensagem de commit** como base de decisão: três "ganhos" eram de placa e um foi retratado (P-03, P-04).
- **A história reescrita e o squash total do delta** (P-06): destrói atribuição e impede cherry-pick por hash contra a base.
- **O default não medido** (F-10) como referência de viabilidade: 420–450 ms por frame é o que o usuário recebe ao compilar, e não foi remedido.
- **As 164 aproximações `shortcut:`** sem contexto: várias são específicas deste título e desta GPU; herdar o texto sem herdar a condição engana.
- **Os lints afrouxados** (`6a2d9f9`): `undocumented_unsafe_blocks = "allow"` num crate com `global_asm!` e handlers de exceção é dívida que não deve ser herdada.
- **A dupla tentativa de otimizar page mapping por conta própria** (F-08): cada tentativa custou um ciclo de streaming quebrado; o problema é do título e segue sem causa conhecida.

---

## Veredito

Este é o fork mais bem instrumentado do conjunto e o que documenta o próprio método com mais honestidade, inclusive os fracassos: 40 commits próprios em ~29 horas, +83.615 linhas contra a base `ffe310e`, cinco módulos de medição escritos antes das otimizações e uma retratação publicada no dia seguinte a um número errado. A engenharia de maior valor não é um algoritmo, é uma propriedade: a chave do cache de shader é derivada do próprio código, de modo que nenhum experimento pode rodar o artefato de outra configuração — exatamente o erro que `6a2d9f9` cometeu e que `e9765a7` e `8dd30ab` fecharam. Ao lado disso há três consertos de causa raiz que se pagam em qualquer camada (guarda em todas as views de uma página física, cópia in-place com limiar e quarentena, hash paralelo com redução ordenada) e uma disciplina de switches que explica por que vários builds privados podem divergir do mesmo código. O que impede uma nota maior é duplo e concreto: **estabilidade** — perdas de device sem causa em pelo menos quatro configurações, com sombra preta e 24 GB de processo — e **verificação** — commits de integração que declaram não ter rodado a suíte na árvore integrada, 247 switches sem schema, 164 aproximações declaradas e um default que ninguém remede desde 420–450 ms. É um repositório de pesquisa de altíssima densidade com um placar honesto e um piso frágil: copie os instrumentos e as quatro correções, não a configuração.
