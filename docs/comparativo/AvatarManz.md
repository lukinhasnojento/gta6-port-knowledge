# Acertos e defeitos de avatarmanz/VIctory-7900XTX

## Ficha

| Campo | Valor |
|---|---|
| Repositório | `avatarmanz/VIctory-7900XTX` |
| Commits | 621 (191 merges; 176 substantivos do fork), 246 arquivos versionados, `HEAD` `0a43566` |
| Licença | `GPL-3.0-or-later` (`Cargo.toml:11`, `LICENSE` = texto GPL v3), **contraditada** pela atribuição de `console_insn.rs:5-6`, que declara port de `GPL-2.0-only` com o decodificador mantido "unchanged" |
| Papel no ecossistema | Fork de desempenho do VIctory especializado em RDNA 3/RX 7900 XTX: é a fonte de conhecimento GPU-específica do ecossistema e o dono do instrumento de replay que os outros forks não têm. |
| Estado hoje | Em `0a43566` (2026-11-25) roda o título entre 6 e 30 fps (6-10 na perseguição de rua) com 40-46 GB de RAM privada, mantém o flicker outdoor aberto (`README.md:19-37`) e carrega um conflito de licença não resolvido. |

## Placar

| Dimensão | Nota | Justificativa |
|---|---|---|
| Engenharia | 4/5 | Correções de causa raiz provada no hardware, com número antes/depois e teste de placa (`20b9834`, `1f58d91`, `31714de`); perde ponto pela higiene de configuração e pela fila de pipelines remendada três vezes. |
| Verificabilidade | 3/5 | Replay reproduzível turno a turno e testes reais em GPU (`8f08131`, `20b9834` com `probe_tests`/`valu_tests`), mas o CI roda só clippy e unidade sem placa (`.github/workflows/ci.yml`) e 46 testes estão `#[ignore]`. |
| Documentação | 2/5 | `docs/findings.md` e `docs/replay.md` são bons e os comentários registram experimento e número, mas o README nega código de XCL que o código declara (`3b30d6f` contra `578d459`) e descreve como padrão o que só `scripts/play.ps1` liga. |
| Sustentabilidade | 2/5 | 297 switches `VICTORY_*` sem registro, três convenções opostas de booleano (`98` × `is_none_or` contra `7` × `is_some_and` contra `map_or`), otimizações ligadas por padrão e desligadas depois (`c651346`→`6ae0696`, `ac11bbb`→`4e99544`, `2c20d7a`→`f4cc8d8`). |
| Reaproveitamento | 0/5 | BLOQUEIO JURÍDICO: `console_insn.rs:5-6` declara port de `GPL-2.0-only` dentro de repositório `GPL-3.0-or-later`, incompatível em regra; **nenhum arquivo pode ser copiado** antes de a licença ser resolvida. |

**Média:** `2.2/5`

## Acertos

| # | Acerto | Commit | Por que importa | Reusável? |
|---|---|---|---|---|
| A-01 | `v_max/v_min`, `v_mul_legacy`, `v_med3` e `v_fract` escritos como a instrução única que o driver compila na nativa, com análise de magnitude `bounded(v, 12)` para decidir quando o produto simples é seguro | `20b9834` | Maior ganho de tradutor medido e reprodutível do ecossistema: 18,41 → 17,58 ms no frame do quintal (12 loops), 0,4 ms só dos produtos legacy, imagem igual a 79 dB; as formas portáveis custavam 5 a 7 operações por instrução | não (licença): reimplementar a técnica |
| A-02 | `crates/victory-gpu/src/replay/*`: 15 arquivos, 13.838 linhas, com `--compare`, `--pixel`, `--diverge`, `--upcopies`, `--retranslate`, `--shaders-b`, `--flow`, `--watch`, `--slots` | `af2baf5`, `6912fb1`, `35155be` | É o instrumento que permite medir GPU sem o jogo e sustenta **todas** as decisões numéricas do fork; mais durável que qualquer otimização isolada | não (licença): reimplementar o desenho |
| A-03 | Raw twins: heaps de 2^22 views, e acima de 2^21 uma view raw dos mesmos bytes para cada view de buffer do slot do título; picks raw/constant leem ali | `1f58d91` | Na AMD um load raw através de view tipada **não tem fim** (lê até 4 GiB adiante) e o descritor de textura lido como buffer é endereço selvagem: isso causou dois device losses em `0452fc2d2e2fc4c3`; custo declarado: 8 min de retradução | não (licença): reimplementar |
| A-04 | Slot nulo no fim do heap de views: todo índice vindo de dados é clampado para um descritor mantido nulo, e o limite entra na chave do cache de shader | `31714de` (branch `fix/amd-view-heap-clamp` de triippywhale), ampliado em `1f58d91` | Converte leitura fora do heap (trava de GPU na NVIDIA, page fault e remoção do dispositivo na RDNA 3) em leitura de descritor nulo definida, sem branch por acesso | não (licença): reimplementar |
| A-05 | Helper lanes: `isHelperLane.i1` e `first_lane()` devolvendo o próprio valor da lane quando ela é helper; programas de pixel passam a ler o quad por operações de quad | `9fea743`, `1c8dd31` | O modo whole-quad do console põe helpers em EXEC e as wave reads do host as excluem: sem isso, buracos quadrados e bordas tracejadas em toda sombra | não (licença): reimplementar |
| A-06 | `monitorx`/`mwaitx`/`clzero`/`rdpru`/`extrq`/`insertq`/`clwb` substituídos **no lugar** no primeiro fault (`monitorx`→nop de 3 bytes, `mwaitx`→`pause`+nop), com laços `clzero` virando stores e cache de região lembrada | `2132366`, `13ffc28` | Num i9-12900K cada instrução dessas levantava `STATUS_ILLEGAL_INSTRUCTION`: **25.000 exceções por segundo** nos laços `clzero` do título | não (licença — é justamente o arquivo com o conflito de F-01) |
| A-07 | Orçamento de VRAM medido por `IDXGIAdapter3::QueryVideoMemoryInfo`, com folga reservada para render targets e limiares de "cópia de buffer vai para a placa" escalonados por VRAM | `17c4717`, `a26859f` (fxceless) | Documenta o fato que quase ninguém mede: numa placa de 24 GB o budget local reportado é ~10,3-11,2 GB; sem folga a fila de texturas enchia a placa a 99-100,5 % e as cópias de buffer eram recusadas | sim (o dado e a forma, não os números) |
| A-08 | Texturas que shaders escrevem e a CPU lê voltam para a memória do título, retiladas no layout do console | `23925a4` (de fxceless `f5c03f3b`, PR 999: Geckomacabre) | Corrige o barco que afunda: os mapas de altura do oceano são escritos por compute e amostrados pela física na CPU; ligado por padrão | não (licença): reimplementar |
| A-09 | Teste de layout de textura contra os tamanhos e endereços da própria addrlib | `8f08131` | É o único ativo que permite confiar no detiling: valida `SW_4KB/64KB_S/D/T/X/Z` contra a referência, não contra a intuição do autor | sim (ideia de teste) |
| A-10 | `Layout::bytes_from(first)`: detilar uma cadeia parcial de mips usando só o prefixo presente | `9526cb2` (de victory-amd `bddfb4a5`) | Resolve o caso real de streaming — só os mips menores estão na memória — com teste que compara byte a byte o prefixo contra o todo | não (licença): reimplementar |
| A-11 | Diagnóstico dentro do buffer de orçamento indireto: palavras com registros recusados e maior custo visto, consumidas pela linha de frame do `Meter` | `449830f` | É o instrumento que falta aos outros forks para provar ou descartar a hipótese de orçamento esgotado; transforma "pisca" em contagem | sim (ideia e formato) |
| A-12 | Corrida entre mapear uma view e abrir os guards das outras views da mesma página, medida e fechada sob o lock | `689df31` (duplicata `2be0515`) | Falhava em 6 de 200 execuções, 4 de 380 com a primeira metade corrigida, 0 de 603 com as duas: é o padrão de corrigir com número, não com opinião | não (licença): reimplementar |
| A-13 | Passes sobre a tela com classificação, workers e orçamento de espera próprios (10 s), na frente dos demais | `a989668` | Esses passes rodam uma vez no primeiro frame do mundo; atrás da fila geral e do orçamento de 0,5 s eram descartados para sempre e a cena ficava sem lâmpadas e sem cor na sombra | não (licença): reimplementar |
| A-14 | Impressão digital do tradutor: BLAKE3 dos fontes de `victory-shader` + `Cargo.lock` + switches que o tradutor lê, para o cache sobreviver a rebuild | `9fea743` | Chave de cache correta é (fonte do tradutor, switches, fatos do dispositivo), não o executável; economiza a retradução de ~2.500 pipelines | sim (desenho) |
| A-15 | Resultados negativos publicados com número: `v_mad` como FMA, recíprocos rápidos, 4 dwords por load (18,47 → 18,45 ms), wave size por programa (+0,1 a +0,26 ms), sem barreiras (0), tudo fora | `20b9834`, `3b30d6f` | Economiza dias de quem for consolidar; quase nenhum fork registra o que já falhou | sim (dado) |

## Defeitos

| # | Defeito | Gravidade | Commit | Estado |
|---|---|---|---|---|
| F-01 | BLOQUEIO JURÍDICO: o README afirma que nenhum código do XCL está no repositório, mas `console_insn.rs` declara "Ported from XCL (GPL-2.0-only): `insn.rs` (decoder, as `console_insn_decode.rs`, **unchanged**) and `eh.rs`", e a licença do projeto é `GPL-3.0-or-later` | CRÍTICO | código: `578d459`, `6a2d9f9`; afirmação: `3b30d6f` | aberto |
| F-02 | Um registro indireto não preenchido esgota o orçamento de 2^27 da lista e **todos os draws seguintes daquela lista rodam vazios**; qual registro é o culpado muda de frame a frame (ordem vem de atomics) | ALTO | introduzido em `8c26061`; instrumentado em `449830f` | aberto — `rest()` continua `map_or(true, \|v\| v != "0")`, ou seja, `REST=1` por padrão (`indirect.rs:302-304`) |
| F-03 | `VICTORY_SEND_CHANGED` ligado por padrão mandava só as dwords alteradas de uma página e deixava o que a placa escrevera para trás: na run `tile1-0` a sala perdeu a luz indireta e a geometria explodiu | ALTO | `c651346`; desligado em `6ae0696` | mitigado por default invertido, **causa raiz não encontrada**; o caminho rápido está morto (`shadow.rs:2029-2030`) |
| F-04 | 16 threads de mapeamento por padrão: no dormitório os grupos seguram o lock de mapeamento e **não mapeiam mais páginas** — 260 ms de frame contra 125 ms | ALTO | `ac11bbb`; revertido em `4e99544` | corrigido (8 threads por padrão, `auto` opt-in; `play.ps1` fixa `1`) |
| F-05 | A correção de ordem do tamanho de BVH foi **desligada** cinco commits depois de medida: com ela, 33 de 154 cópias ainda liam o tamanho velho (21 %) contra 53 de 230 sem ela (23 %), ao custo de ~20 ms/frame | ALTO | `2c20d7a` (correção), desligada em `f4cc8d8` | aberto — a cópia que transborda sobre estruturas vizinhas volta a ser possível por padrão (`memory.rs:799-820`) |
| F-06 | Reversões mudas: `a50c7fd` (replay recriar buffers em heaps abertos da memória) é revertido integralmente e `9820bdc` (stand-in de janela HLE) também, ambas sem justificativa no corpo | ALTO | `95037ba`, `d1c8acd` | revertido; limitação volta sem documentação (a lacuna nem aparece em `docs/replay.md` "Limits") |
| F-07 | 297 nomes `VICTORY_*` lidos por `env::var`/`var_os`, sem registro central, com três convenções opostas de booleano: 98 × `is_none_or(\|v\| v != "0")`, 7 × `is_some_and(\|v\| v != "0")`, mais `map_or(true, ..)` | MÉDIO | crescimento incremental; a convenção que o fork segue em parte vem de `77582c3` | aberto |
| F-08 | O README apresenta como padrão features que o binário não liga; a configuração real é `scripts/play.ps1`, com 22 switches aplicados (+4 variáveis de caminho) | MÉDIO | `3b30d6f` (README e script no mesmo commit) | aberto — `victory run` puro mede outro programa |
| F-09 | CI verde não diz nada sobre o produto: `cargo clippy` + `cargo test` sem GPU, com 46 testes `#[ignore]`, incluindo o de replay grande e testes que exigem a placa | MÉDIO | `.github/workflows/ci.yml`; testes de hardware em `8f08131`, `20b9834` | estrutural |
| F-10 | Constantes de layout amarradas a um título: `ambient.rs` (`LIGHT = (1,176)`, `DEPTH = (0,464)`, `RADIANCE = (1,16)`, presos a `CS_ScreenProbes_TemporalResolve`) e `texback.rs` (`DEFAULT = "256x256f41,256x256f16,64x64f16,128x128f36"`) | MÉDIO | `82ca134`, `23925a4` | por design; quebra silenciosa em outro build/patch do jogo |
| F-11 | Depois de `84eae35` liberar dispatch indireto largo em compute (2^20 testado na 7900 XTX), o limite de **mesh** foi endurecido: acima de 65.535 rodam vazios e o mesh direto é cortado | MÉDIO | `a9c2e99`, `ef556e4` | aberto — regressão de capacidade para conter risco de crash |
| F-12 | O mesmo patch commitado duas vezes com bases diferentes, ambos ancestrais de `HEAD`: `689df31`/`2be0515` e `9c2168a`/`160d222` (191 merges em 3 dias) | BAIXO | `689df31`, `2be0515`, `9c2168a`, `160d222` | árvore limpa hoje; qualquer rebase que reaplique a versão antiga reintroduz a corrida corrigida em `689df31` |
| F-13 | Device loss a cada ~16 min por textura mostrada antes de o upload rodar: a correção existe, mas é opt-in e o autor a considera "likely fixed, not proven" | BAIXO | `3bab23c` | mitigado só em `play.ps1` (`VICTORY_SHOW_AFTER='1'`); custo aceito: texturas aparecem tarde |
| F-14 | `VICTORY_WAVE_WIDE` (forma wave-wide) perdeu a placa em submissão de dois programas que acabavam de migrar para ela (`ptfx_CS_lightAtlas_Sprite`, `_Trail`, runs 121-124) | BAIXO | NÃO VERIFICADO (forma herdada; comentário em `emit/mod.rs:509-514`) | desligado por padrão |

## Erros de processo

| # | O que aconteceu | Evidência |
|---|---|---|
| P-01 | README que contradiz o código em assunto com consequência jurídica, escrito no dia seguinte a o código já estar na árvore | `README.md:108-109` ("no code of it is here") contra `console_insn.rs:5-6`; código em `578d459` (2026-11-24), frase em `3b30d6f` (2026-11-25) |
| P-02 | Medição de um cenário promovida a padrão universal: 16 threads de mapeamento valiam para o carregamento e foram aplicadas ao dormitório, dobrando o frame | `ac11bbb` → `4e99544` (260 ms contra 125 ms, runs `textures-d4`, `-d2`) |
| P-03 | Reversão sem justificativa: o corpo de `95037ba` é só "This reverts commit a50c7fd…", e `d1c8acd` reverte 322 linhas de `lifecycle.rs` sem dizer por quê; nenhuma das duas tem substituta | `95037ba`, `d1c8acd` |
| P-04 | Correção desligada com base em diferença dentro do ruído, descrita como "mended nothing": 33/154 (21 %) contra 53/230 (23 %), ao custo de 20 ms | `f4cc8d8` contra `2c20d7a`; números no próprio comentário de `memory.rs:802-808` |
| P-05 | O mesmo patch feito em `day/*` e `night/*`, cherry-picked dos dois lados e depois mergeado: dois pares de commits com assunto idêntico, ambos no histórico final | `689df31`/`2be0515`, `9c2168a`/`160d222`; 191 merges em 621 commits |
| P-06 | Configuração oficial fora do binário: os números do README só se reproduzem com um hashtable PowerShell, e qualquer medição com `victory run` puro mede outro programa | `scripts/play.ps1` (22 switches, `VICTORY_THIN='1'` exige literal `"1"` em `list/thin.rs`), `3b30d6f` |
| P-07 | CI que não bloqueia o que importa: clippy `-D warnings` e unidade sem GPU, enquanto os testes que validam o produto são `#[ignore]` ou exigem uma RX 7900 XTX | `.github/workflows/ci.yml`; 46 ocorrências de `#[ignore]` |

## O que copiar

Nada como código, enquanto F-01 estiver aberto: a licença declarada em `console_insn.rs:5-6` contamina o repositório inteiro para efeito de incorporação. O que deve ir para o repositório unificado é conhecimento, reimplementado do zero:

1. As regras de float da RDNA da A-01, incluindo a análise de magnitude (`bounded(v, 12)`) que decide quando o produto simples é seguro — o item de maior valor do fork.
2. O desenho dos raw twins (A-03) e do slot nulo (A-04), que precisam ser projetados **juntos** na consolidação: um ocupa a faixa 2^21..2^22, o outro o último slot.
3. A regra de helper lanes (A-05): wave reads do host excluem helpers, o console não.
4. O desenho do replay (A-02) como ferramenta de medida, não como feature.
5. Os fatos de VRAM de A-07 e o formato de diagnóstico de A-11 (palavras de registros recusados e maior custo no buffer de orçamento).
6. O teste contra a addrlib (A-09) e `bytes_from` (A-10) como padrão de validação de detiling.
7. A lista de A-15: o que já foi medido e descartado, para não ser repetido.

## O que não copiar

1. Qualquer arquivo, antes de resolver F-01 — inclusive os 15 arquivos de `replay/*` e todo `victory-shader`, que são a parte mais valiosa.
2. O modelo de switches (F-07): 297 variáveis sem registro e três convenções de booleano; consolidar isso multiplica a dívida por fork.
3. `ambient.rs` e `texback.rs` como estão (F-10): constantes de layout de um título específico que falham em silêncio em outro build.
4. O rateio do orçamento indireto de F-02, que ainda derruba todos os draws seguintes de uma lista por um registro lixo.
5. O endurecimento do limite de mesh de F-11: não acompanhou o de compute e corta trabalho legítimo.
6. O par `THIN` + `ASYNC_SUBMIT` + landing como peças soltas: é um modelo de ordem fechado, e foi ele que criou a classe de bugs de F-05.
7. `scripts/play.ps1` como configuração oficial e o README como especificação de comportamento padrão (F-08).

## Veredito

Um laboratório de desempenho de três dias feito por um agente, com disciplina experimental rara e higiene de projeto ruim. Do lado bom, é o único fork com conhecimento GPU-específico medido no hardware: as regras de float da RDNA escritas como instrução nativa com prova de magnitude, o load raw sem fim da AMD fechado por raw twins, o page fault da RDNA 3 fechado por slot nulo, helper lanes, 25.000 exceções por segundo eliminadas por auto-modificação de código, e um replay de 13.838 linhas que é mais valioso que qualquer otimização que ele mediu. Do lado ruim, três correções e otimizações centrais foram desligadas por padrão em vez de consertadas (`c651346`, `ac11bbb`, `2c20d7a`), duas reversões não têm justificativa, 297 switches não têm registro nem convenção única, o README descreve um programa que o binário não é, e o projeto carrega um conflito de licença que o próprio README nega. Não cumpre a meta que declara (30 fps): entrega 6-10 fps na perseguição de rua com 40-46 GB de RAM privada. Como referência técnica de RDNA 3 é a melhor fonte entre os cinco forks; como código a incorporar, hoje, é zero — não por falta de qualidade, mas porque a licença não permite.
