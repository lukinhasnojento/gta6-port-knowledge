# Acertos e defeitos de Marvisak/VIctory

## Ficha

| Campo | Valor |
|---|---|
| Repositório | `Marvisak/VIctory` (original, não é fork; raiz `f4504f6`, HEAD `70564ae`) |
| Commits | 241 alcançáveis de `main`, 91 merges, 88 números de PR, 49 nomes de autor (51 identidades), 174 arquivos, 90.609 linhas de Rust no HEAD |
| Licença | `GPL-3.0-or-later` (`LICENSE`; README:164) — com uma exceção declarada que a licença **não** cobre: `crates/victory-formats/src/bcpack-tables.bin` (`docs/asset-policy.md`) |
| Papel no ecossistema | Tronco do ecossistema: os cinco forks comparados em `FORK_DIVERGENCIA.md` têm `70564ae` como base, e `docs/architecture.md` declara que tudo exceto `victory-game` é camada de plataforma reaproveitável por qualquer título Xbox Series. |
| Estado hoje | Não jogável, por declaração própria em `70564ae`: boota, menus a 30 fps, carrega o mundo em ~50 s e desenha a primeira missão a 1-2 s por frame, superfícies sem iluminação e faixa de geometria ausente. |

## Placar

| Dimensão | Nota | Justificativa |
|---|---|---|
| Engenharia | 4/5 | Correções que fecham a classe de falha (`b1c8306`, `3c81420`, `35b054e`) conviveram com um commit de 42.518 linhas que semeou 14 classes (`2849477`) e três regressões autoinfligidas (`3c81420`→`e8d3b64`, `36eb0b4`→`2e5001c`, `73d776b`/`25b0f88`). |
| Verificabilidade | 4/5 | 482 `#[test]` e testes sintéticos que rodam em hardware **e** WARP, com CI de clippy `-D warnings` + `cargo test` (`8e79c80`) — mas a CI cobre só os últimos 41 de 241 commits e a árvore de testes ficou 54 commits quebrada (`73d776b`→`7102a49`). |
| Documentação | 3/5 | README exemplarmente honesto e numérico (`70564ae`), porém `docs/architecture.md` tem 93 linhas para 90.609 de código, afirma "planned: BCPack" com BCPack já implementado desde `2849477`, e a decisão de `prepare.rs` não tem registro (`c21208c`→`25a3e95`). |
| Sustentabilidade | 2/5 | 125 marcas `shortcut:` sem dono, memória host que só cresce (`2849477`, G-04), um único host de teste e medições que existem apenas em mensagem de commit. |
| Reaproveitamento | 5/5 | É a base dos cinco forks, e a fronteira plataforma/jogo é declarada e respeitada (`docs/architecture.md`): só `victory-game` é específico do título. |

**Média:** `3.6/5`

## Acertos

| # | Acerto | Commit | Por que importa | Reusável? |
|---|---|---|---|---|
| A-01 | Prova de satisfabilidade antes de colapsar o waterfall por lane (e nomear cada recusa) | `b1c8306` + `6c63dea` (merge `ca48fbe`) | Transforma uma otimização que supunha "cada lane lê o seu" num teorema verificável sobre o CFG; quatro formas inválidas viraram teste. É o modelo para qualquer lowering que dependa de convergência de lanes. | sim |
| A-02 | Recurso escasso falha local, nunca fallback silencioso | `3c81420` (merge `edf64e2`) | `bind() -> bool` + `table_failed` descartam o draw em vez de deixar o shader ler descritor alheio como constant buffer; a unidade de trabalho é descartável, o device não. | sim |
| A-03 | HRESULT do console como contrato funcional | `f0ee35e`, `28c1b07` (merge `f0894c2`); `376b3cf` (merge `40b7ed1`) | O título ramifica no código de erro: separar cota de save de cota de disco (`E_GS_OUT_OF_LOCAL_STORAGE` vs `E_GS_QUOTA_EXCEEDED`) evita mandar o jogador apagar saves por um disco cheio. | sim |
| A-04 | Copy-back por instantâneo + diff, com o instantâneo em pool | `35b054e` | Separa o que a GPU escreveu do que a CPU do título escreveu durante a submissão; o pool evita ~2,3 M page faults por frame do mundo. Padrão clássico de memória compartilhada com escrita dos dois lados. | sim |
| A-05 | Teste sintético de GPU em hardware **e** WARP como gate de fidelidade | `2e5001c`, `51f1627`, `bf84134` | É o único mecanismo que separa "host" de "driver": pegou a regressão de `fract` em NVIDIA e a semântica de `fcmp one` na AMD sem relato de usuário. | sim |
| A-06 | Heap dimensionado pela capacidade real do host, com o offset traduzido | `3827767` + `e8d3b64` (merge `edf64e2`) | Toda tabela que passa dos slots de título é refeita em slots dinâmicos e o shader recebe o índice novo; nenhum índice inexistente chega ao hardware. | sim (com ressalva: G-01) |
| A-07 | Política de assets com hook de commit, allowlist e guarda em runtime | `f4504f6` (`docs/asset-policy.md`, `xtask/src/policy.rs`, `.githooks/pre-commit`) | Impede que material do jogo entre no repositório, com a única exceção registrada, justificada e com instrução de reescrita de histórico. Raro num projeto de 2 dias. | sim |
| A-08 | Política final de imagem PE preparada | `c21208c` + `25a3e95` | Realocação do ponteiro TLS zerado vira `IMAGE_REL_BASED_ABSOLUTE` e o diretório de import aponta para o terminador vazio, com teste que mapeia uma DLL real e confere o slot de TLS. Reusável por qualquer loader que injete imports. | sim |
| A-09 | Limite de entrada derivado do hardware, não de número redondo | `e6a2c54`, `e158d95` | `MAX_EXTENT = 1 << 14` é o máximo que um descritor GFX10 endereça, então nenhum dado válido é recusado; `checked_*` fecham os sítios vizinhos. | sim |
| A-10 | Registro de `shortcut:` no ponto exato do código | `2849477` (prática; 125 ocorrências no HEAD `70564ae`) | Separa "não implementado de propósito" de bug num codebase que precisa rodar um título real em dias — e produz o mapa do que falta. | com ressalva (virou passivo sem dono) |
| A-11 | README que declara o estado real, com números por estágio | `70564ae` | "Não jogável", cobertura mesh 8.765/8.765, vertex/pixel 12.550/12.592, compute 2.579/2.637, 1-2 s por frame, um único host testado. É o oposto de README promocional. | sim |
| A-12 | Crash report que nomeia a instrução recusada e o conteúdo do endereço | `babe2db`, `3390e47` | Falha de CPU deixa de ser endereço solto e passa a ser diagnóstico acionável; somado a `5f670bb` (import não implementado no log), encurta o ciclo de bring-up. | sim |

## Defeitos

| # | Defeito | Gravidade | Commit | Estado |
|---|---|---|---|---|
| F-01 | `XGameSave` com configuration id vazio: a pasta do usuário era tomada por container, um container vazio era gravado no lugar e a pasta era removida **com todos os saves** das outras configurações | CRÍTICO | `c88b7ac` (merge `27f9b76`) | corrigido |
| F-02 | `#[cfg(test)] mod tests` duplicado em `victory-gpu/src/host.rs` (E0428): nenhum teste do workspace rodava, por 54 commits | CRÍTICO | `73d776b` → `7102a49` (merge `d2a4bae`) | corrigido |
| F-03 | `lowers()` mudou de aridade em um PR e o chamador antigo ficou: `cargo test` não compilava no `main` | CRÍTICO | `25b0f88` → `8e79c80` | corrigido |
| F-04 | Esgotamento dos slots dinâmicos de descritor fazia o draw ler a tabela do título como constant buffer → `DXGI_ERROR_DEVICE_REMOVED` ~60 s após entrar no mundo | CRÍTICO | `3c81420` + `e8d3b64` (merge `edf64e2`) | corrigido (custo: draws descartados) |
| F-05 | Heap de views limitado pelo host (teto de 1.000.000 na NVIDIA) deixava o anel de tabelas do título aliasar slots dinâmicos ou passar do fim → GPU hung em 3 de 3 execuções | CRÍTICO | `3827767` (merge `97c78d1`) | corrigido, com residual aberto (F-14/G-01) |
| F-06 | Produto não jogável: mundo a 1-2 s por frame, superfícies sem iluminação, faixa de geometria distante ausente, personagens quebrados, 195 programas com estágios fundidos sem tradução | CRÍTICO | `70564ae` (declarado pelo próprio projeto) | aberto |
| F-07 | `prepare.rs`: ponteiro de callbacks TLS zerado mas realocação mantida (Wine faultava em `ntdll`), e três políticas de import/IAT em quatro commits | ALTO | `c21208c`, `d85097a`, `4da3d63`, `25a3e95` | corrigido |
| F-08 | Caminho narrow reescrito duas vezes (wrapper wide chamando wrapper wide): toda abertura narrow de arquivo do título falhava | ALTO | `27b530e` (merge `b0328ba`) | corrigido |
| F-09 | `v_readfirstlane_b32` de descritor tomado como "o de cada lane" em qualquer lugar, sem prova de que o compare estreita EXEC → amostragem por lanes puladas | ALTO | `ffa6635` → `6285e19` → `b1c8306` | corrigido, com restrição aberta (G-03) |
| F-10 | Copy-back devolvia o range inteiro declarado e sobrescrevia o que a CPU do título escreveu durante a submissão → crash do título | ALTO | `35b054e` | corrigido |
| F-11 | Correção de `fract()` em `36eb0b4` foi dobrada para `minNum` pelo compilador DXIL da NVIDIA e passou a engolir NaN (regressão em hardware real) | ALTO | `36eb0b4` → `2e5001c` (merge `237f2e8`) | corrigido |
| F-12 | `CopyResource`/`CopyTextureRegion` de textura escrita pela CPU não passavam pelo caminho de upload: a GPU recebia conteúdo velho | ALTO | `0064c08` + `4f53d6f` (merge `6db02ea`) | corrigido |
| F-13 | Descritores nulos do tipo/dimensão errados geravam índice absurdo no heap → device removido segundos depois do mundo | ALTO | `8c26061`, `071487f`, `6af0fae`, `4051149`, `d71c388` | corrigido em cinco camadas |
| F-14 | Nada é liberado no lado host: uma textura host por endereço e formato, views antigas mantidas, cópias de janela nunca liberadas — memória cresce monotonicamente | ALTO | `2849477` (`resource.rs`, `shadow.rs`) | aberto |
| F-15 | Overflow e laço infinito com dados do título: XMA de comprimento 0, `ADVANCE_XMA_READ` somando em `u32`, quatro sítios de layout de textura, `XMemMapPhysicalPages` com contagem 0 (panic dentro de `extern "system"`) | MÉDIO | `005ad49`, `257d8bb`, `e6a2c54`, `e158d95`, `02fcf72` | corrigido |
| F-16 | Códigos de erro do HLE divergentes do Xbox: disco cheio reportado como cota, buffer pequeno como `E_NOT_SUFFICIENT_BUFFER`, path curto como `ERROR_INSUFFICIENT_BUFFER` | MÉDIO | `f0ee35e`, `28c1b07`, `376b3cf` | corrigido |
| F-17 | `XTaskQueueDispatch(..., INFINITE)` não acordava no terminate (thread e join travados) e `XAsync` só aceitava `S_OK`, perdendo payload de `S_FALSE` | MÉDIO | `36d6a79`, `3e39eac` (merge `fd92558`) | corrigido |
| F-18 | Lote de fidelidade de shader: `v_cmp_lg` como `fcmp one`, min/max sem ordenar `-0`, carry de half-float com `or`, `s_addc`/`s_subb` sem SCC, far jump lido sem sinal, `s_ttracedata_imm` em 0x26 | MÉDIO | `51f1627`, `bf84134`, `63df363`, `a2f5f43`, `c963b71`, `79fb722` | corrigido |
| F-19 | Testes VALU de GPU travavam `cargo test` (3 de 30 execuções) por uso concorrente do mesmo device/fila | MÉDIO | `1106f43` (merge `0ddbce8`), `74a9d33` | mitigado (lock de processo; causa raiz não investigada) |
| F-20 | Page table limitada a 512 regiões de 256 MiB por tabela: endereço fora do limite lê o que a página tiver | MÉDIO | `27c3dc9` (mecanismo), `6d021fe` (`CAPACITY = 512`) | aberto |
| F-21 | `docs/architecture.md` afirma "planned: BCPack" com BCPack implementado e usado desde `2849477` | BAIXO | `70564ae` (`architecture.md:43`) | aberto |

Escala de gravidade:
- **CRÍTICO** — quebra o build, perde dados do usuário, ou impede o uso por qualquer pessoa.
- **ALTO** — corrupção de resultado, crash frequente, regressão de correção anterior.
- **MÉDIO** — comportamento errado em caso específico, dívida que trava evolução.
- **BAIXO** — higiene, nomenclatura, inconsistência documental sem efeito funcional.

## Erros de processo

| # | O que aconteceu | Evidência |
|---|---|---|
| P-01 | O suporte ao título entrou num único commit de 107 arquivos e 42.518 inserções, sem revisão possível — e é a origem de 14 das classes de defeito catalogadas (slots dinâmicos, descritores nulos, XGameSave, XMA, overflows, predicados) | `git show --stat 2849477`: `107 files changed, 42518 insertions(+), 860 deletions(-)` |
| P-02 | CI só existiu nos últimos 41 de 241 commits; antes disso nada impedia `main` vermelho | `.github/workflows/ci.yml` criado em `8e79c80`; `git rev-list --count 8e79c80^` = 200 |
| P-03 | A árvore de testes ficou 54 commits sem compilar, e um segundo build vermelho durou 118 commits até a mesma CI o pegar | `git rev-list --count 73d776b..7102a49` = 54; `git rev-list --count 25b0f88..8e79c80` = 118 |
| P-04 | Três políticas para o mesmo campo em quatro commits, incluindo uma reversão manual (`d85097a` desfeita por `4da3d63`) e a volta da ideia em `25a3e95` — sem commit de revert e sem registro de decisão | `git log -- crates/victory-loader/src/prepare.rs`: `25a3e95`, `4da3d63`, `d85097a`, `c21208c`, `f4504f6`; "Reverts explícitos" vazio em `VIctory_fatos.md` |
| P-05 | Mensagem de commit que esconde o alcance: "Test probe: compare direct and joined reads of a shader timestamp" muda a assinatura de `fn lowers` e quebra o build de testes do `main` | `git show 25b0f88 -- crates/victory-shader/src/emit/lower.rs` (`- fn lowers(words: &[u32])` → `+ fn lowers(words: &[u32], rsrc2: u32)`), 33 inserções, cherry-pick de `4b2ea70` |
| P-06 | Ganhos de desempenho e taxas de falha existem apenas no texto do commit: "11x", "3.5 → 7.8 GiB/s", "3 de 30", "0 de 60". Não há `benches/`, nenhum artefato de medição, e 7 testes `#[ignore]` | `git ls-files \| grep bench` vazio; 7 `#[ignore]` no HEAD `70564ae` |
| P-07 | Validação num host só, com 49 autores e 91 merges em ~20 h: os dois builds vermelhos vieram de PRs paralelos que se cruzaram | README `70564ae`: "Tested on one machine: Windows 11, AMD RX 6700 XT"; `73d776b`/`0b607b1` e `a2f5f43`/`25b0f88` |
| P-08 | Documentação que não acompanha o código, no documento que orienta quem chega | `docs/architecture.md:43` ("planned: BCPack") contra `crates/victory-formats/src/bcpack.rs`, presente desde `2849477` |
| P-09 | Duas fontes secundárias usadas como especificação: tabela SOPP do Mesa (`s_ttracedata_imm` em 0x26) e equivalência de `fcmp one` para `v_cmp_lg` — os dois errados no alvo | `79fb722` (opcode 0x28), `51f1627` (`olt \| ogt`) |

## O que copiar

1. `crates/victory-shader/src/resource.rs` + `emit/lower.rs` no estado de `b1c8306`/`6c63dea`: a prova de EXEC antes de colapsar waterfall, com as quatro formas inválidas como teste.
2. O padrão `bind() -> bool` / `table_failed` / contador de descarte de `3c81420` e `crates/victory-gpu/src/draw.rs`, mais o dimensionamento de heap de `3827767` + `e8d3b64`.
3. `crates/victory-gpu/src/shadow.rs` como está em `35b054e`: instantâneo prévio do range, diff, pool.
4. `crates/victory-hle/src/modules/xgamesave.rs` e `storage.rs` com a tabela de HRESULT do console (`f0ee35e`, `28c1b07`, `376b3cf`, `c88b7ac`).
5. `crates/victory-loader/src/prepare.rs` + `crates/victory-pe/src/layout.rs` (`c21208c`, `25a3e95`), incluindo o teste que mapeia uma DLL real e confere o slot de TLS.
6. `docs/asset-policy.md`, `xtask/src/policy.rs` e `.githooks/pre-commit` (desde `f4504f6`): allowlist, limite de 512 KiB, exceção única registrada, guarda `ensure_outside_repo` em runtime.
7. Os módulos de teste sintéticos `emit/valu_tests.rs`, `synthetic_tests.rs`, `waterfall_tests.rs`, `stride_tests.rs`, `memory_tests.rs` e o lock de processo de `1106f43`.
8. `.github/workflows/ci.yml` de `8e79c80`: clippy `-D warnings` em duas configurações (normal e `--all-features`) e `cargo test --workspace --locked` em `windows-latest`.
9. `crates/victory-formats/src/xma.rs` e `texture.rs` com limite derivado do descritor e aritmética checada (`e6a2c54`, `e158d95`).
10. A convenção `shortcut:` como vocabulário — mas com relatório gerado por `xtask` e teste que falha quando o comportamento mudar.

## O que não copiar

1. `2849477` como modelo de commit: 107 arquivos e 42.518 linhas num só diff é a causa direta da densidade de defeitos dos dois dias seguintes.
2. A camada de GPU como um todo sem reescrever o ciclo de vida: 125 marcas `shortcut:` e nenhuma liberação de textura, view ou cópia de janela (`2849477`, F-14) garantem crescimento monotônico de memória.
3. `crates/victory-formats/src/bcpack-tables.bin`: o próprio `docs/asset-policy.md` diz que a licença do repositório **não cobre** essas tabelas. Não redistribuir sem antes decidir a proveniência.
4. `victory-game` (formatos, identidades de build e hooks do título): é a única crate específica do jogo e não pertence ao núcleo.
5. `docs/architecture.md` como documento de arquitetura: 93 linhas, afirmação desatualizada sobre BCPack. Reescrever a partir do código.
6. O estado do waterfall anterior a `b1c8306` — reconhecer a forma sem provar a semântica — mesmo que o código pareça mais simples.
7. Testes de GPU sem serialização e sem diagnóstico de causa (F-19/G-08): o lock resolve o sintoma, não a interação com o driver.
8. Limites fixos no host (`HEAP_VIEWS`, `CAPACITY = 512`, `MIN_DYNAMIC`) como constantes sem aviso em runtime quando o host não os comporta (G-01, F-20).

## Veredito

VIctory é, em dois dias e 241 commits, o esqueleto completo de um port de plataforma — loader PE com política de imagem preparada, HLE com fidelidade de HRESULT, tradutor GFX10→DXIL e uma camada D3D12 bindless — e é a base real dos cinco forks, não por acaso: a fronteira entre plataforma e título está declarada e respeitada. Tecnicamente, o que merece ser herdado são as correções que converteram suposição implícita em invariante verificada (`b1c8306`, `3c81420`, `35b054e`, `c88b7ac`) e o hábito de testar fidelidade de shader em hardware e WARP. O que não merece é o método de entrega: um commit de 42.518 linhas semeou 14 classes de defeito, 200 dos 241 commits entraram sem CI, a árvore de testes ficou 54 commits sem compilar, a política de `prepare.rs` foi trocada três vezes sem registro, e 125 `shortcut:` documentam um passivo que ninguém acompanha — inclusive uma textura host por endereço que nunca é liberada. O produto é o que o próprio README diz: boota, mostra o mundo, e não é jogável a 1-2 s por frame. Como fundação a aproveitar, é o melhor material do ecossistema; como código a colar sem revisão, é uma dívida com data marcada.
