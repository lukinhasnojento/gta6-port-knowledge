# Acertos e defeitos de uiliannn/VItoriaBr

## Ficha

| Campo | Valor |
|---|---|
| Repositório | `uiliannn/VItoriaBr` |
| Commits | 373 em `b6e61ab` (107 merges; 218 na cadeia de primeiro pai) |
| Licença | `GPL-3.0-or-later` (`LICENSE`, texto GPLv3; declarada em `6132a9e` e em `README.md:9` de `b6e61ab`) |
| Papel no ecossistema | Fork integrador do `Marvisak/VIctory` que absorveu, em 48 h, o trabalho de cinco forks (`fxceless`, `Belobug`, `myokex`, `DexNom`, `triippywhale`) e ~100 PRs do upstream (`578d459`, `8e46dce`, `4f9a19d`, `b401954`, `4e038bf`), e é a única fonte do conjunto com mundo 3D renderizando ao vivo em NVIDIA Blackwell — RTX 5060 Ti 16 GB, SM 6.6, driver `621.24`, até 6 fps com 1.700–4.400 draws/frame (`README.md:18,26` de `b6e61ab`). |
| Estado hoje | Árvore parada em `b6e61ab` (2026-11-26 02:02 +0300): RTGI não ilumina nada (`docs/fork-ray-tracing.md`), device removal no mundo aberto sem causa conhecida (`0b0074e`), e o README afirma "zero TDR" 49 linhas acima do aviso de que o TDR pode acontecer (`b6e61ab`:26 vs :75). |

## Placar

| Dimensão | Nota | Justificativa |
|---|---|---|
| Engenharia | 4/5 | As grandes correções **removem trabalho estruturalmente errado** em vez de acelerá-lo, e cada uma nomeia a invariante que a torna segura (`9ceb7e7`, `713a095`, `9376fd6`); desconta por três correções que introduziram defeito próprio (`91cd9c9`, `a13a719`, `a91ef70`). |
| Verificabilidade | 3/5 | 686 `#[test]` em 126 arquivos e CI real em dois SOs com `clippy -D warnings` (`d52d84c`, `defaac8`), mas 6 commits declaram "test suites were not run" e nenhum número de medição foi reproduzido fora do autor. |
| Documentação | 3/5 | README bilíngue e denso, com retratação explícita de um benchmark errado (`0b0074e`) — penalizada porque o mesmo README se contradiz sobre TDR (`b6e61ab`:26 vs :75) e publica contagens que os próprios docs desmentem. |
| Sustentabilidade | 2/5 | Histórico reescrito sem merge-base com o upstream, 32.334 linhas num commit (`578d459`), dezenas de `VICTORY_*` no caminho de produto e quatro trocas conscientes sem registro de decisão (`2849477`, `578d459`, `a803c87`, `c931d12`). |
| Reaproveitamento | 4/5 | Mecanismos de cartão, sync de descriptor e instrumentação de frame são os mais portáveis do conjunto — mas só por patch, nunca por cherry-pick (não há merge-base com a base). |

**Média:** `3.2/5`

## Acertos

O que o projeto fez **certo**, com evidência. Ordene por valor.

| # | Acerto | Commit | Por que importa | Reusável? |
|---|---|---|---|---|
| A-01 | Funde bytes da CPU **no cartão** por máscara de 1 bit por byte, em vez de esperar a GPU | `9ceb7e7` | O cap `VICTORY_MERGE_RUNS=512` sempre estourava (centenas de milhares de runs de poucos bytes/frame): ~1 settle/frame, 58-61 ms de espera → 0,00 settle e 13 ms, frame 162-172 → 139-140 ms (~10%). A operação é idempotente por construção (`card = (card & ~m) \| (title & m)`). | sim, com ressalva (levar máscara + `CARD_MERGE` + exclusão byte a byte no relay) |
| A-02 | Hash de peça de 64 KiB com **grupo de 64 threads** em vez de 1 thread e 4.096 passos dependentes | `713a095` | "our card lists ~30 ms -> ~5 ms a frame"; cada passo é bijeção do estado, então qualquer mudança confinada a uma lane muda o hash da peça. Remove o gargalo invisível que fazia o passe custar uma peça inteira. | sim, com ressalva (a forma é escolhida **uma vez por processo**; hashes antigos ficam incomparáveis) |
| A-03 | Espelha registros de descriptor **copiados como bytes** pelo título, com fingerprint barato e comparação real | `a26859f`, `9376fd6`, `213fa54` | ~40.000 registros/frame que nenhuma chamada de API anunciava; o passe sobre o heap caiu de **70-220 ms para ~3 ms**, e a colisão de hash custa uma cópia, nunca um descriptor errado. É a resposta geral para "o título escreve memória que a API não vê". | sim |
| A-04 | Regra `Rule::Start` para spans de textura sobrepostos, com `HEAD = 4096` medido | `310c6eb` | Invalidar por sobreposição anulava **toda** textura empacotada (spans se sobrepõem por construção). A regra nova usa um bloco de 4 KiB não compartilhado como identidade de reuso, e cita 129 pares medidos em 2 runs. | sim, com ressalva (o número 4096 é empírico deste título) |
| A-05 | Ordem de publicação no `relay_land`: estado → `notify_all` → libera recurso | `aa85bb0` | Corrige o 0xc0000005 do loading com `VICTORY_RELAY=2` (que é o padrão). Instância limpa do padrão que qualquer pool com espera deve seguir. | sim |
| A-06 | Recuperação do XGameSave reescrita para não apagar o save commitado | `d1da85a` | `version_number()` passou a exigir `v<N>` exato e `current()` pega a maior versão **que lê**, sem tocar em entradas mais novas ilegíveis. Antes, um `v01` residual apagava o `v1` commitado (issue #870). | sim |
| A-07 | Deadlock do loader lock no cache de módulos HLE, com teste em processo filho | `708b450` | `host::module` segurava o lock do cache enquanto chamava `LoadLibraryExA` — cache→loader lock e loader lock→cache. O teste usa `DLL_THREAD_ATTACH` dormindo 500 ms e mata o filho, porque um deadlock impediria o processo de sair. | sim |
| A-08 | Bounds check de LDS com o tamanho do **programa**, não o total do host | `40beb26` | Separar `lds_dwords` do console dos scratch slots do host reproduz a semântica OOB do RDNA2 (lê 0, descarta escrita) e evita trap de SRAM na NVIDIA. | sim |
| A-09 | Cache de shaders chaveado pela **tradução**, não pelo executável | `a6a8d39`, `ef30575`, `a393a19` | `translation_fingerprint()` = fontes do `victory-shader` + flags + valores em runtime de todos os `VICTORY_*` lidos pelo tradutor + capacidades do host. Sem isso, aquecer cache (`victory warm`) é inútil. | sim |
| A-10 | Reverteu a própria correção em 22 minutos, com contador que provou 0 acertos | `c931d12` (reverte `89434b7`) | "0 textures … while walking uncommitted ranges via VirtualQuery spiked the heap texture pass to 752 ms/frame (5 s frame stalls)". Deixou o resto do commit (coalescência de remaps, prioridade de pipelines) e a caminhada atrás de `VICTORY_COMMITTED_QUERY=1`. | sim (a prática, não o código) |
| A-11 | Retratou publicamente um benchmark errado | `0b0074e` | Retirou o "6.0 against 8.9 frames a second" de `a253706` porque as duas execuções mediram cenas diferentes (~4.300 vs ~1.550 draws) e publicou **reta de ms vs draws** (145 ms off / 124 ms on a 4.500 draws). Uma retratação em 373 commits vale mais que dez números. | sim (a prática) |
| A-12 | CI duplo (Windows completo + Linux portátil) e `cargo xtask ci` | `d52d84c`, `defaac8` | Job `portable` em `ubuntu-latest` com `clippy -D warnings` e `cargo test` cobre o tradutor, que é a parte mais ativa e não precisa de Windows. Foi o que pegou a quebra de `213beb9`. | sim |
| A-13 | `test_skip` com `VICTORY_STRICT_TESTS=1` falhando em vez de pular | `b4180ea` (via `#32`/`#43`) | Resolve o "verde que não testou nada": teste que exige GPU imprime `VICTORY TEST SKIPPED` e, com a variável, **falha**. | sim |
| A-14 | Nomeou o stride no ponto de contrato shader↔host (`SLOT_VIEW_BYTES`) | `5603b45` (corrige `b62bbc0`) | Uma constante de 16 bytes usada nos dois lados + `read_view_slot()` centralizando clamp e offset — em vez do `slot << 3` que lia o slot vizinho. | sim |
| A-15 | Forma wave-wide perigosa **desligada por padrão**, e os switches de wave entraram na chave do cache | `e9765a7` | Corrigir o emissor sem corrigir a chave teria mantido o defeito vivo em toda máquina com cache. O elo escondido foi fechado no mesmo commit. | sim |

## Defeitos

O que o projeto fez **errado**. Ordene por gravidade.

| # | Defeito | Gravidade | Commit | Estado |
|---|---|---|---|---|
| F-01 | Import monolítico destruiu a granularidade histórica: 61 arquivos e +32.334/−1.349 linhas num único commit. Para a maioria dos 21 defeitos catalogados, "commit introdutor" significa apenas "primeiro commit onde o código aparece" — é o caso de `copy_back_done`, do `LOG2_E` e do clamp de 65.535 grupos | CRÍTICO | `578d459` | aberto (irreversível) |
| F-02 | XGameSave podia **apagar o save commitado**: um `v01` residual parseava como versão 1 e a limpeza reconstruía o nome como `v1`, apagando-o | CRÍTICO | `d1da85a` | corrigido |
| F-03 | Device removal no mundo aberto, causa desconhecida: "lost the device there with no driver entry … Cause of that removal not known either" | ALTO | `0b0074e` | aberto |
| F-04 | Dispatches indiretos acima de 65.535 grupos são **truncados ou zerados**, não divididos: trabalho perdido silenciosamente (`indirect.rs:129`, `:142-146`: `if (ahead && g.x > 65535u) → 0`, senão `min(g.x, 65535u)` com `y=z=1`) | ALTO | `578d459` (estado em `b6e61ab`) | aberto |
| F-05 | `VICTORY_FIRSTLANE_OWN=1` chega a 53/53 programas do builder de BVH mas **corrompe offsets de AABB na memória** — crash após ~18 s; o builder fica em 50/53 e não ativa por padrão | ALTO | `18924c7` (texto em `README.md:164` de `b6e61ab`) | aberto |
| F-06 | Corrida no `relay_land`: ranges liberados antes de `line.written` + `notify_all` → 0xc0000005 no loading com `VICTORY_RELAY=2` (o padrão) | ALTO | `578d459`, corrigido em `aa85bb0` | corrigido |
| F-07 | A **correção** de F-06 (`91cd9c9`) deduplicou ranges entre flights, mas `copy_back_done` decrementa `Flights` por flight: ranges ficavam pendentes para sempre | ALTO | `91cd9c9`, corrigido em `aa85bb0` | corrigido |
| F-08 | Bounds check de LDS comparava com o total alocado pelo host (inclui scratch de troca de metades): leituras fora da faixa **do programa** liam scratch do host | ALTO | `a13a719`, corrigido em `40beb26` | corrigido |
| F-09 | `SLOT_VIEWS`: host escreve 16 bytes por slot, emissor lia com stride de 8 (`slot << 3`) — descritor do slot vizinho em qualquer índice dinâmico | ALTO | `b62bbc0`, corrigido em `5603b45` | corrigido |
| F-10 | Heap de descriptors esgotado fazia a tabela cair no offset do **título**, e o shader lia textura/nulo como CBV → endereço base selvagem e remoção do device na AMD (issue #59) | ALTO | `3c81420` (política: descartar em vez de ler lixo) | corrigido |
| F-11 | `VirtualQuery` em `mapped_bytes` levou o passe de texturas a **752 ms/frame** com 0 texturas recuperadas | ALTO | `89434b7`, revertido em `c931d12` | revertido (limitação original permanece) |
| F-12 | Merge sem build: `ef30575` (PR #6, `victory warm`) deixou **duas definições** de `warm_shader` | ALTO | `ef30575`, corrigido em `06161b9` | corrigido |
| F-13 | Atualização wave64 emitiu barreiras de grupo **dentro de branches/loops**: dois programas de atlas de luz travavam o cartão até o reset do device | ALTO | `6a2d9f9`, mitigado em `e9765a7` | mitigado (forma off por padrão) |
| F-14 | `d16` em buffer store interpreta inteiros de 16 bits como `f16`: `0x0001` grava ~6e-8 em vez de 1 | MÉDIO | `2849477` (comentário em `lower.rs:10108`) | aberto (sem correção; impacto NÃO VERIFICADO) |
| F-15 | Rechecagem de textura indexada faz fingerprint de apenas 48 KiB (início/meio/fim): textura cujo miolo chega depois continua errada; o próprio commit declara o gap | MÉDIO | `a803c87` | aberto |
| F-16 | O primeiro frame da primeira sala ainda podia vir sem paredes; o README de `8dd30ab` dizia "the room's very first shot still comes up without walls in some runs" — e a frase **saiu** do README sem que nenhum commit afirme a correção (não existe em `b6e61ab`) | MÉDIO | `8dd30ab` | aberto |
| F-17 | Ray tracing não ilumina nada: "nothing is lit by ray tracing yet … Everything here is off by default", compensado por `ambient.rs` portado de Belobug | MÉDIO | `6a2d9f9` (`docs/fork-ray-tracing.md`) | aberto |
| F-18 | README afirma **"zero TDR"** (`:26`) e 49 linhas depois avisa que entrar no mundo 3D "ainda pode disparar reset do driver" (`:75`); o aviso é de `6132a9e` e o "zero TDR" entrou depois, em `b3d2f13` | MÉDIO | `b3d2f13` (aviso em `6132a9e`) | aberto |
| F-19 | `SHADER_HEAPS`: use-after-free no espelho (lock solto dentro de `shader_slot()`); a correção (`32d7848`) segurou o mutex e criou 48,5 ms/frame de contenção nas seis threads de gravação | MÉDIO | `32d7848`, corrigido em `dfd22d1` | corrigido |
| F-20 | Teste novo do POPS usava `Container`/`FourCC` inexistentes fora do Windows e **quebrou o CI portátil** | MÉDIO | `213beb9`, corrigido em `ca4759f` | corrigido |

Escala de gravidade:
- **CRÍTICO** — quebra o build, perde dados do usuário, ou impede o uso por qualquer pessoa.
- **ALTO** — corrupção de resultado, crash frequente, regressão de correção anterior.
- **MÉDIO** — comportamento errado em caso específico, dívida que trava evolução.
- **BAIXO** — higiene, nomenclatura, inconsistência documental sem efeito funcional.

## Erros de processo

Fatos sobre **como** o projeto trabalha, não sobre o código. Exemplos: mensagem de commit que não
corresponde ao diff, histórico reescrito, README que contradiz o código, benchmark medido em
cenário não representativo, CI que não bloqueia.

| # | O que aconteceu | Evidência |
|---|---|---|
| P-01 | Integração por **squash-import**: forks inteiros entraram como um commit cada, sem PR rastreável nem atribuição verificável por linha | `git show --stat 578d459` → "61 files changed, 32334 insertions(+), 1349 deletions(-)", assunto "victory-gta6: this fork's changes on top of VIctory main (ffe310e)"; mesma forma em `8e46dce`, `b401954`, `dfd22d1`, `4e038bf` |
| P-02 | Histórico **reescrito** em relação à base: não existe merge-base com o HEAD do Marvisak (`70564ae`), só a raiz `f4504f6`. Consequência: nenhum cherry-pick é possível, e comparação por hash entre repositórios é inválida | `git merge-base` falha entre as duas linhas; `f4504f6` é a única raiz comum (`docs/fatos/FORK_DIVERGENCIA.md`) |
| P-03 | README que **contradiz a si mesmo** sobre estabilidade: o aviso de TDR foi escrito em `6132a9e` (11-24 07:05) e o "zero TDR" entrou 10 h depois em `b3d2f13` (11-24 17:52), e os dois textos convivem no HEAD | `b6e61ab`:26 vs `b6e61ab`:75; `git log -S"zero TDR" -- README.md` → `b3d2f13`; `git log -S"ainda pode disparar reset" -- README.md` → `6132a9e` |
| P-04 | Contagens de BVH divergentes entre o README e o doc técnico: README diz **50/53** (53/53 com `FIRSTLANE_OWN`, que corrompe AABB), `docs/fork-ray-tracing.md` diz o builder tem **52** programas e **49 de 52** traduzem | `b6e61ab`:164 vs `git show b6e61ab:docs/fork-ray-tracing.md` |
| P-05 | Doc técnico descreve o padrão do código errado: "The current barrier mode defaults to 2", mas `parse_group_sync` devolve **3** no caso `_` | `git show b6e61ab:docs/performance-architecture.md` vs `git show b6e61ab:crates/victory-shader/src/emit/mod.rs` (`_ => 3`) |
| P-06 | Benchmark medido em **cenário não representativo** e publicado assim mesmo — corrigido depois | `a253706` publicou 6,0 → 8,9 fps; `0b0074e`: "compared runs in different scenes and is withdrawn" |
| P-07 | Testes **não rodados** em merges declarados prontos | 6 commits com "test suites were not run" ou equivalente: `9376fd6` ("This merged tree was built; its test suites were not run"), `a26859f`, `4e038bf`, `a253706`; `8dd30ab` admite rodar só a árvore de origem ("gpu 129, hle 97") |
| P-08 | Merge **sem build**: PR #6 entrou com duas definições da mesma função | `ef30575` → `06161b9` remove a duplicata em `crates/victory-hle/src/modules/d3d12x.rs` (−8 linhas) |
| P-09 | Oscilação de decisão sem medição registrada: a mudança de lane-masks entra (`90228a7`), sai (`9e49b8a`, "costs about 40 ms of GPU time per frame") e volta no dia seguinte (`2119b0e`, "that was not wanted") | `git log -1 --format=%B` dos três commits |
| P-10 | Números de PR de **três repositórios diferentes** misturados no mesmo histórico, o que torna "o que o PR #N resolveu" uma pergunta sem resposta | `58fed37` (`Merge pull request #706`, upstream), `9ceb7e7` (`(#37)`, fxceless), `ef30575` (`merge: PR #6`, este fork) |
| P-11 | Identidade de autor inconsistente: 73 nomes distintos em 373 commits, incluindo um `Marvis <marvis@noreply.invalid>` com 15 commits | `git log --format="%an" \| sort -u` → 73; contagem por e-mail em `docs/fatos/VItoriaBr_fatos.md` |
| P-12 | Instrumentação e diagnóstico são robustos, mas os **artefatos de medição** não são versionados: não há diretório de traços, ao contrário de `victory-fork` | `git ls-tree -r b6e61ab \| grep diagnostics` → vazio; `crates/victory-gpu/src/{framelog,cardmeter,drvwatch,profile,shot,fpsbox}.rs` existem, os dados que os alimentaram não |

## O que copiar

- `crates/victory-gpu/src/shadow.rs` + `host.rs`: o par **máscara de bytes / `CARD_MERGE`** de `9ceb7e7`, com o teste WARP de 3.000 runs aleatórios de 1-9 bytes cruzando dwords. Copiar o par inteiro ou nada.
- `crates/victory-gpu/src/host.rs` + `indirect.rs`: `CARD_COMPARE_WIDE` de `713a095`, com a decisão por processo (`card_hash_wide()`) mantida como mecanismo — não como configuração de usuário.
- O trio de sync de descriptor: `descriptor.rs` (`sync_heaps`, `RwLock` de `dfd22d1`), `draw.rs` (`fresh_tables`, hash de tabela + comparação de 3 dwords de `213fa54`), `descriptor/host.rs` (`Spans`/`Rule::Start` de `310c6eb`).
- `SLOT_VIEW_BYTES` e `read_view_slot()` (`5603b45`): o padrão de **nomear o stride no ponto de contrato**, entre emissor e host.
- `crates/victory-shader/src/emit/test_skip.rs` (`b4180ea`): `VICTORY_STRICT_TESTS=1` transformando "pulado" em falha.
- `.github/workflows/ci.yml` + `xtask/src/ci.rs` (`d52d84c`, `defaac8`): CI duplo com `clippy -D warnings` e `cargo test` nos dois SOs.
- `crates/victory-shader/src/emit/image.rs` (`6dcd8cd`) e o clamp de LOD de `a290636`: cobertura de variantes MIMG e a remoção do `LOG2_E` sobre um `Log` que já devolve log2.
- `crates/victory-gpu/src/shader_cache.rs` + `build.rs` (`a6a8d39`, `e9765a7`): chave de cache = fingerprint da tradução + switches de wave + capacidades do host.
- `crates/victory-hle/src/modules/xgamesave.rs` (`d1da85a`) e `crates/victory-hle/src/host.rs` (`708b450`): correções de perda de save e de deadlock, ambas com teste de regressão honesto.
- `ambient.rs` (`d0af132`, `692bce4`) como **paliativo nomeado** de RTGI, com o switch de desligar e a obrigação de remoção quando o RT funcionar.
- Como ferramenta, não como produto: `shot.rs`, `framelog.rs`, `cardmeter.rs`, `drvwatch.rs`, `profile.rs`, `fpsbox.rs`.
- O hábito de `c931d12` e `0b0074e`: reverter e retratar com o número na mão, no mesmo dia.

## O que não copiar

- O **modelo de integração**: importar fork inteiro como commit único (`578d459`, 61 arquivos, +32.334 linhas). É a razão pela qual 21 defeitos deste repositório não têm commit introdutor identificável.
- O README como fonte de verdade sobre estabilidade. `"zero TDR"` (`b6e61ab`:26) contradiz `:75`; `50/53` (`:164`) contradiz `docs/fork-ray-tracing.md`. Qualquer número copiado do README precisa ser remedido.
- O `docs/performance-architecture.md` como especificação de padrões: afirma barrier mode padrão 2 onde `parse_group_sync` devolve 3.
- As quatro trocas conscientes sem registro de decisão: `d16` como `f16` (`2849477`), truncamento em 65.535 grupos (`indirect.rs:129`), fingerprint de 48 KiB (`a803c87`), e mips interiores invisíveis (`c931d12`). Cada uma vira uma pergunta que alguém refará do zero.
- O preset com 16 variáveis `VICTORY_*` (`README.md:141-164` de `b6e61ab`) como configuração de usuário: cada switch é um caminho de teste não coberto.
- `VICTORY_FIRSTLANE_OWN=1`, `VICTORY_INDIRECT_WINDOW=1` e `VICTORY_CARD_FETCH_INFLIGHT=1`: um corrompe AABB (`:164`), outro perdeu o device três vezes (`f85101b`), o terceiro reduz o FPS (`:163`).
- `docs/comparative-performance-hypotheses.md` e `docs/performance-architecture.md` como documentação de produto: são notas de revisor externo, escritas contra uma árvore que se movia, e já nasceram divergentes do código (`b6e61ab`).

## Veredito

Este repositório é uma sessão de três dias que produziu 135 mil linhas de Rust e fez um título de Xbox renderizar o mundo aberto num PC — e é, tecnicamente, o membro mais avançado do conjunto em desempenho medido: as suas maiores vitórias não são otimizações, são **remoções de trabalho estruturalmente errado** (hashear 64 KiB com um thread, comparar 300 mil registros com BLAKE3, esperar o cartão para fundir bytes), cada uma com número antes/depois e uma invariante escrita no código que a torna segura. É também o único que **retratou um benchmark** (`0b0074e`) e o único que reverteu a própria correção em 22 minutos com o contador provando 0 acertos (`c931d12`) — disciplina real, não retórica. O que não se sustenta é o resto: a arqueologia foi destruída por decisão tática (`578d459`, +32.334 linhas num commit), o README se contradiz sobre a única coisa que um usuário precisa saber antes de ligar o jogo (`:26` vs `:75`), as contagens divergem entre README e docs, e quatro trocas conscientes ficaram sem registro de decisão. Serve muito bem como **fonte de patches** — máscara de bytes no cartão, hash em grupo, sync de descriptor por fingerprint, `test_skip`, CI duplo — e serve mal como fonte de história ou como referência de estabilidade. Para o repositório unificado: copiar os mecanismos por conteúdo, nunca por commit, e remedir tudo o que o README afirma.
