# Acertos e defeitos — `omonkeyz/xbox-emulator`

## Ficha

| Campo | Valor |
|---|---|
| Repositório | `omonkeyz/xbox-emulator` |
| Commits | 595 em `HEAD` (`main`), 104 merges, 32 identidades de autor, 5 dias com commit (`2b4bba2` → `a618736`) |
| Licença | `GPL-3.0-or-later` (`ceb2b87`, `LICENSE`; releases anteriores em `MIT OR Apache-2.0`, README.md:11) |
| Papel no ecossistema | Runtime de camada de plataforma para Xbox Series X\|S em Rust que traduz microcódigo RDNA2 pré-compilado para HLSL e recompila com DXC, substituindo `d3d12_xs.dll` sobre o D3D12 do PC. |
| Estado hoje | `a618736` (2026-11-25): Trackmania jogável e Terraria no mundo, 3D incorreto em todos os títulos, 0/8 estágios de shader completos e 117 itens `OPEN` declarados em `docs/BLOCKERS.md`. |

## Placar

| Dimensão | Nota | Justificativa |
|---|---|---|
| Engenharia | 4/5 | Correções atingem causa raiz com prova de execução (`b03b39b`, `7364bb4`, `9b7c93a`), mas os três pontos de maior impacto ainda dependem de heurística (`b03b39b`, `bd185fa`, `4fcfe13`). |
| Verificabilidade | 5/5 | A mais alta do conjunto: conformance contra o `xgameruntime.dll` real (`acfede3`), execução bit-exata em WARP e GPU com mutação que derruba cada fix (`1fc8683`, `e7377c9`), GBV como oráculo (`8c481f4`) e sonda de silício em gfx1036 (`f1e8d7c`). |
| Documentação | 3/5 | README honesto sobre o que não funciona (README.md:40-56) e `BLOCKERS.md` com 117 `OPEN`, mas `c95a909` descreve trabalho que está em outro subsistema e `ff39277` enterrou todo o histórico em um commit. |
| Sustentabilidade | 3/5 | 595 commits em 5 dias, lotes `rNN` multiassunto (`d759b31`, `f89e410`), CI de workspace que não bloqueia (`a618736`) e proveniência de código sem licença registrada (`ceb2b87` + `THIRD_PARTY/CREDITS`). |
| Reaproveitamento | 4/5 | O tradutor ISA→HLSL, o harness de execução e os padrões de conformance são diretamente reutilizáveis para o alvo unificado; ~1.329 arquivos de `crates/compat` são stubs específicos de biblioteca de console. |

**Média:** `3.8/5`

## Acertos

O que o projeto fez **certo**, com evidência. Ordenado por valor.

| # | Acerto | Commit | Por que importa | Reusável? |
|---|---|---|---|---|
| A-01 | Conformance contra o binário real: os mesmos cenários rodam no emulador e no `System32\xgameruntime.dll` carregado com `InitializeApiImpl(0x4a61, 0x2712)`, com `const DIFFERS` listando divergências | `acfede3` (17 cenários), `c3292cc` esvazia `DIFFERS` | Converte "o que o console faz" de conjectura em teste executável; a divergência passa a ser dado, não opinião | sim |
| A-02 | Testes de execução bit-exata de shader (DXC + WARP, `TOMBOX_EXEC_HW=1` para GPU) com mutação que derruba cada correção | `1fc8683`, `26ced2a`, `e7377c9` | É o que torna "o shader traduzido está correto" verificável; sem ele, todo o resto é plausibilidade | sim |
| A-03 | GBV como oráculo de descritor (`TOMBOX_D3D12_GBV=1`, IID conferido por teste) | `8c481f4` | A validação compara a view do título com o tipo do shader traduzido — exatamente a fronteira onde a tradução erra e onde WARP é cego; foi o que expôs D-01/D-02 | sim |
| A-04 | Stride do elemento de `StructuredBuffer` lido do metadado FSSM, não do padrão de acesso da ISA | `b03b39b` (merge `a618736`) | Elimina o `ByteStride` mismatch na origem; testes `buffer_elements_take_the_fssm_stride` e `conflicting_fssm_strides_keep_the_extent_heuristic` | sim |
| A-05 | Guardas IEEE contra o dobramento algébrico do compilador AMD (`tb_pz`, `tb_hz`, `tb_trunc64`, `tb_fsub`) | `65893a4` + `3f1bc0f` (merge `d0fabcb`) | O ISA do console exige o bit exato de zero/Inf; o compilador do host não é obrigado a preservá-lo — 23 casos falham sem a guarda na RX 9060 XT | sim |
| A-06 | Sondas de silício (`tools/rdna2-hwprobe`) como fonte de verdade, em gfx1036/RX 9060 XT | `f1e8d7c`, `bf0f50c` | A referência deixa de ser "o que o WARP faz" e passa a ser o que o hardware faz; `f1e8d7c` prova que o console preserva Inf/NaN no blend | sim |
| A-07 | Recusa nomeada em vez de aproximação (raw+`IDXEN`, MIMG `R128`/`TFE`/`LWE`, `ds_swizzle` por estágio) | `3e9676d`, `d585b94` | Troca corrupção silenciosa por falha alta com motivo; é o que permite rodar 105.506 shaders e saber por que cada um falhou | sim |
| A-08 | Stand-in de objeto COM para API cujo contrato no console é "nunca falha" (`IGameInput` com 22 métodos) | `146286e` | Preserva o contrato sem mentir sobre capacidades: cada método devolve um código real, e o stand-in passa pelo mesmo `patch_object` | sim |
| A-09 | Diagnóstico de teste "flaky" como assincronia real do Windows, com medição antes/depois (42/200 → 0/200 falhas, mutante 10/10) | `ab6d485` | Evita mascarar comportamento de runtime; a consequência de produção ficou declarada em `BLOCKERS.md` em vez de silenciada | com ressalva |
| A-10 | Leitura de memória colocada com recuperação de falha em vez de cópia/snapshot | `5d5bc33` (merge `fe391fe`) | Uma corrida com decommit do título vira *skip* de upload em vez de término do processo | com ressalva (SEH/Windows) |
| A-11 | Um decodificador por classe de campo, não por instrução (`readlane_dst`, `i32_result_of_64`) | `9b7c93a`, `8f71091` | Elimina as tabelas divergentes que produziram M0(124)/NULL(125) e a largura `_b64`; impede a reincidência, não só o sintoma | sim |
| A-12 | Endurecimento de parser de entrada hostil: fuzz de caminho com alfabeto Unicode, colisão de nome em filesystem case-insensitive, RVA limitada no PE | `0284e83`, `cce78b6`, `0b4583d` | Trata dump de terceiro como dado hostil; colisão de entrada vira achado bloqueante, não aviso | sim |
| A-13 | Higiene de dump + gate de PR que recusa binários e dados de jogo | `4291a50`, `.github/workflows/pr-gate.yml` | Política que sobreviveu aos 595 commits e mantém o repositório publicável | sim |
| A-14 | README declara o que não funciona, com título, build, máquina e data | estado em `a618736` (README.md:40-56) | "material errado", "texturas pretas", "<1 fps e crasha", "GTA VI não roda" aparecem ao lado do que funciona — contradiz o verde do `MILESTONES.md`, e o README é o lado honesto | sim |

## Defeitos

O que o projeto fez **errado**. Ordenado por gravidade.

| # | Defeito | Gravidade | Commit | Estado |
|---|---|---|---|---|
| F-01 | Checkout limpo não compila: `#[cfg(test)] mod oracle;` em `xg_xs` sem `src/oracle.rs` — `E0583`, 15 erros e 19 testes inexequíveis; o job com `--all-targets` nomeia cinco crates e não inclui `xg_xs` | CRÍTICO | import (`ff39277`); mitigado em `706a2b7` (merge `2813b2a`) | mitigado |
| F-02 | Árvore Rust inteira importada em um commit (1.131 arquivos, 266.033 linhas) e movida para a raiz em `410e01f`: 19 dos 22 defeitos catalogados só podem ser datados como "import" | ALTO | `ff39277`, `410e01f` | aberto |
| F-03 | CI não bloqueia o check de workspace: `rust-workspace-check` com `continue-on-error: true` e `cargo check --workspace` sem `--all-targets`; o job agregador `rust-core` só exige `rust-core-tests` e `xshader-exec` | ALTO | `6741468` (estado em `a618736`) | aberto |
| F-04 | `MODE` numérico preso em `0xf0`: `PGM_RSRC1_AT: &[(u32, usize)] = &[]` (tabela vazia), localização no container FSSM não provada; denormal de f32 nem é reportado em `Translated.unknowns` | ALTO | `bd185fa` (`mode.rs`) | aberto |
| F-05 | Elemento de `StructuredBuffer` ainda por heurística quando a tabela FSSM falta, conflita ou traz stride 0/1 — o próprio código marca `/// A heuristic (TECHNICAL_DEBT)` | ALTO | import; parcial em `b03b39b` | mitigado |
| F-06 | Estágios de shader incompletos: `MILESTONES.md` marca "All shader stages translated 0/8 open"; tesselação tem só o parser PSV0, e o encoding 2D/3D de `DispatchMesh` nunca foi verificado | ALTO | import | aberto |
| F-07 | `DispatchMesh` lia Z do dword do ponteiro de payload: todo grupo de malha lançado duas vezes, invisível na comparação de pixels (2x de trabalho de GPU) | ALTO | `df91928` | corrigido (`53bb998`) |
| F-08 | `cl_edition::after_call` lia um byte além da janela de 8 bytes em candidato SIB: panic em pleno despacho de comandos | ALTO | `6d19943` | corrigido (`68f2cf2`) |
| F-09 | `GameInputCreate` falho devolvia ponteiro nulo a título que não checa `HRESULT`: access violation em host sem serviço GameInput | ALTO | import | corrigido (`146286e`) |
| F-10 | Remoção de device interpretada como fence concluída (`UINT64_MAX` satisfaz qualquer comparação): uso de device morto | ALTO | import | corrigido (`d550bc0`) |
| F-11 | `M0` (124) e `SGPR_NULL` (125) trocados em quatro sítios (readlane, spill, VOP3B, carry-out); dois testes existentes codificavam o bug | ALTO | import | corrigido (`9b7c93a`) |
| F-12 | SOP1 com resultado de 32 bits de fonte de 64 bits escrevia dois SGPRs, apagando ponteiro de root table e não semeando user-data | ALTO | import | corrigido (`8f71091`) |
| F-13 | Buffer raw emitido como `StructuredBuffer` de 16 bytes contra view de `StructureByteStride 0`: todo índice escalado errado | ALTO | import | corrigido (`3e9676d`) |
| F-14 | AS escolhia ABI de root por posição textual do primeiro `s_load`: shader sem nenhum grupo lançado e sem recusa | ALTO | import | corrigido (`3d90f6f`) |
| F-15 | `XMemVirtualAlloc` mascarava `MEM_RESERVE_PLACEHOLDER`/`MEM_REPLACE_PLACEHOLDER`: divisão de placeholder falhava e abortava a inicialização do título | ALTO | import | corrigido (`2e8e2b8`) |
| F-16 | Teste de conformidade afrouxado: `assert_eq!(cur, mine)` → `assert_ne!(cur, 0); assert_ne!(cur, mine)` — remove a garantia que o teste existia para dar, sem declarar medição do runtime real | ALTO | `ba8e0e5` | aberto |
| F-17 | Base de user-data deduzida do estágio da API (VS s8) errava em vertex shader ES mergeado sem `s_swappc` | MÉDIO | import | corrigido (`4fcfe13`, com fallback) |
| F-18 | `v_mad_i64_i32` escrevia o flag de overflow em `sdst`; em gfx1036 `sdst` é o bit 64 (sinal) do resultado exato | MÉDIO | import | corrigido (`7364bb4`) |
| F-19 | Resíduo do dobramento do host: `v_add/sub/mul_f16` com SDWA byte-select + `abs` em src0 devolve 0 no AMD em compute (passa em pixel) | MÉDIO | `3f1bc0f` (seção "Still failing on AMD") | aberto |
| F-20 | Clamp de blend menos exato que o console para Inf/NaN: o console preserva, os modos `eager`/`ps` mapeiam Inf→max e NaN→+max | MÉDIO | medido em `f1e8d7c` | aberto |
| F-21 | Ciclo de vida do `XAsync` divergente em quatro pontos (Cancel, cancelamento por fila, bloco nunca iniciado, identidade de `GetResult`) | MÉDIO | import; corrigido em `e7377c9` | corrigido, 1 cenário aberto |
| F-22 | Escrita byte/short no payload de amplificação via `InterlockedAnd/Or` em `groupshared` não chegava ao estágio de malha em NVIDIA | MÉDIO | import | corrigido (`d2cc33e`) |
| F-23 | `VirtualFree` de título pode falhar até ~50 ms após o release do último buffer aliasado: o wrapper de `xmem` passa direto | MÉDIO | `ab6d485` | aberto |
| F-24 | `gather4_l` ignora bias/clamp quando o S# vem de tabela ou é calculado: só emite aviso (`GATHER4_L: hash=...`) | MÉDIO | `d2dad0c` | aberto |
| F-25 | `tombox-archive::validate` aceitava entradas com nomes colidentes em filesystem case-insensitive e reportava "seguro para extrair" | MÉDIO | import | corrigido (`cce78b6`) |
| F-26 | `console-sandbox::fs::resolve` panicava em caminho controlado pelo título com caractere multi-byte cruzando o índice 4 | MÉDIO | import | corrigido (`0284e83`) |
| F-27 | Flags de fila só-do-console repassadas ao D3D12 do PC: `E_INVALIDARG` em filas de compute | MÉDIO | import | corrigido (`979615b`) |
| F-28 | Blendclamp "eager" prendia alvos `R32G32B32A32_FLOAT` a ±65504 (faixa de fp16) | MÉDIO | import | corrigido (`ced7994`) |
| F-29 | `XGameGetXboxTitleId` com código e ordem de checagem diferentes do runtime do PC (`E_FAIL` em vez de `E_NOT_FOUND`; ponteiro nulo checado antes do título) | BAIXO | import | corrigido (`c3292cc`) |
| F-30 | BOM UTF-8 inserido em `tests/gdk_minecraft_slots.rs` por rebase/cherry-pick | BAIXO | `ba8e0e5` | aberto |

## Erros de processo

Fatos sobre **como** o projeto trabalha, não sobre o código.

| # | O que aconteceu | Evidência |
|---|---|---|
| P-01 | Código publicado sem o histórico que explica as decisões: `ff39277` traz 1.131 arquivos e 266.033 linhas de uma vez sob `source/`, `410e01f` move para a raiz. Não há commit por subsistema, então a autoria pré-import é irrecuperável | `git show --shortstat ff39277`; `git show --stat 410e01f` (1.133 arquivos); 19 de 22 defeitos datados como "import" |
| P-02 | Mensagem de commit que aponta para outro subsistema: `c95a909` diz "Exec mask ... PSO use-after-free fix in register_pso" e o diff é 100% `xtfeventserver` (COM, ETW, dispatcher, 1.294 linhas + `Cargo.lock`). A mudança de exec mask não é datável por mensagem — `git log -S"wqm"` em `hlsl.rs` só encontra o import | `git show --stat c95a909` |
| P-03 | `d585b94` anuncia "Fix #109: recover poisoned shader_xlat mutexes" e **não toca** `shader_xlat.rs` (o stat lista `hlsl.rs`, `gcn.rs`, `meta.rs`, testes e scripts); as 27 ocorrências de `poison`/`into_inner` no arquivo em `HEAD` não têm commit correspondente localizável | `git show --stat d585b94` |
| P-04 | CI cego por construção: `rust-workspace-check` tem `continue-on-error: true` e roda sem `--all-targets`; o agregador `rust-core` exige apenas `[rust-core-tests, xshader-exec]`. Foi esse furo que deixou o `E0583` de `xg_xs` passar por 300+ commits, e ele continua aberto | `.github/workflows/rust.yml` em `a618736`; `706a2b7` descreve o furo |
| P-05 | Métrica verde contra produto vermelho: `MILESTONES.md` marca 200/200 métodos D3D12.x e 105506/105506 shaders como atingidos enquanto o README registra material errado, texturas pretas, Minecraft crashando e XDefiant a <1 fps. "Traduzido" significa "o DXC aceitou", não "renderiza certo" | `docs/MILESTONES.md` e `README.md:51-55` em `a618736` |
| P-06 | Lotes `rNN` multiassunto do mantenedor, com correções de subsistemas distintos na mesma mensagem (`d759b31` mistura tradutor, command-list, PSO library e net_guard; `f89e410` mistura profiling, captura e registries de XUser) | `d759b31`, `2be7211`, `f89e410` |
| P-07 | Histórico reescrito com cherry-pick de PRs: merges declaram "cherry-picked onto rewritten main" e a autoria se dispersa — o merge `894e339` é literalmente 1 linha deletada porque o resto já entrara por cherry-pick | `a0c9fd5`, `81a1252`, `99f48e3`, `894e339` |
| P-08 | Hotfix que admite regressão própria sem apontar o culpado: `3de1b1f` ("o prewarm de game-scan esgotou a memória de título do Trackmania e abortou o carregamento do mapa") torna o prewarm opt-in, mas `git log -S"GAME_SCAN_PREWARM"` só encontra o próprio hotfix | `3de1b1f`, `a1ff5be` (candidato, não provado) |
| P-09 | Conformidade tratada como detalhe ajustável: `ba8e0e5` troca igualdade por desigualdade em dois asserts de identidade de handle, num projeto cujo método é "igualar o runtime do PC medido", sem citar a medição que justifique | `ba8e0e5` (`gdk_process_queue.rs`) |
| P-10 | Proveniência não resolvida sob GPL: `THIRD_PARTY/CREDITS` registra TomBox de drycheetah84 como "publishes no licence" e o fork brcly/TomBox como "no longer reachable", depois da relicença em `ceb2b87` | `ceb2b87`, `THIRD_PARTY/CREDITS`, `d59ed23` |
| P-11 | Linhagem declarada não é a linhagem real: o README afirma continuação de `drycheetah84/TomBox` (base `7e8064f`), mas o reuso verificável do TuxBox são 3 commits Apache-2.0 (`5a25b00`, merge `baadb80`, crédito `05f7422`). A igualdade de 1.652 arquivos com o TuxBox clonado é coincidência numérica — só 7 caminhos coincidem | `5a25b00`, `baadb80`, `05f7422`; ancestralidade TomBox **NÃO VERIFICADO** (repositório indisponível) |

## O que copiar

- **`crates/tombox-xshader/tests/exec*.rs` + `tests/isa_sweep`** (`1fc8683`, `26ced2a`): harness que compila o HLSL gerado com DXC e executa comparando bits, com sharding no CI (`xshader-exec`, 4 shards) e `TOMBOX_EXEC_HW=1` para hardware real. É o ativo mais valioso do repositório.
- **Padrão de conformance contra o binário de referência** (`acfede3`, `c3292cc`, `e7377c9`): carregar o DLL real, rodar os mesmos cenários, listar divergências em `const DIFFERS` e pular com motivo quando o runtime não existe.
- **Guardas IEEE `tb_pz`/`tb_hz`/`tb_hadd`/`tb_hmul`/`tb_trunc64`** (`hlsl.rs:187-192`, `hlsl.rs:791`; `65893a4`, `3f1bc0f`): qualquer tradutor ISA→HLSL herda o dobramento do compilador do host.
- **`fssm_bind` + `fssm_buffer_strides` + `fssm_user_base`** (`b17ecef`, `b03b39b`, `4fcfe13`): a regra "layout vem do metadado do driver, não do padrão de acesso" resolve três classes inteiras de bug.
- **`hlsl/bufref.rs`** (`3e9676d`): tradução de view raw para `ByteAddressBuffer`, com recusa explícita de raw+`IDXEN`.
- **`rootmem.rs` / `FORCE_MEMORY_BASED_ABI`** (`3d90f6f`): regra medida (todo PSO com estágio de amplificação usa ABI de root memory) no lugar de detecção heurística.
- **`guard.rs`** (`5d5bc33`, 242 linhas): leitura de memória que o convidado pode desmapear, com falha recuperável em vez de crash.
- **`present_wait.rs` + teste de remoção de device** (`d550bc0`): predicado `current != u64::MAX && current >= value` e injeção de `UINT64_MAX` na vtable sintética.
- **`tools/rdna2-hwprobe`** (`f1e8d7c`, `bf0f50c`) e o padrão `ab6d485` (medir 200 execuções, comparar taxas, provar com mutante).
- **`pr-gate`** (`.github/workflows/pr-gate.yml`, `4291a50`): recusa binários e dados de jogo em PR.

## O que não copiar

- **O import maciço de árvore** (`ff39277`, `410e01f`): nada do valor arqueológico sobrevive. Se o código vem de fora, entra por subsistema e com o histórico que o explica.
- **Lotes `rNN` multiassunto e mensagens que descrevem outro subsistema** (`d759b31`, `c95a909`, `d585b94`): um commit por assunto, e a mensagem precisa corresponder ao diff.
- **O CI como está** (`a618736`): `continue-on-error: true` e check de workspace sem `--all-targets` transformam o gate em decoração. `cargo check --workspace --all-targets` bloqueante, ou nada.
- **`MILESTONES.md` como termômetro de qualidade**: cobertura de tradução não é correção de renderização (`P-05`). Se a métrica for publicada, precisa de rótulo explícito.
- **Heurística como fallback silencioso** (`sbuf_words`, `detect_rootmem` pré-`3d90f6f`, `PGM_RSRC1_AT` vazio): onde o metadado não foi provado, recusar com nome é melhor que adivinhar — o próprio projeto já faz isso para raw+`IDXEN`.
- **Defaults presos** (`MODE_DEFAULT = 0xf0` em `mode.rs`) sem instrumentação de quantos shaders realmente dependem do valor: acerta por acidente e esconde a dívida.
- **Merges com main reescrita e cherry-pick de autoria** (`a0c9fd5`, `81a1252`, `99f48e3`): preservar autor e commits originais, não squashed.

## Veredito

Um runtime de camada de plataforma Xbox Series que ataca o problema mais difícil do domínio — microcódigo RDNA2 pré-compilado, sem fonte — e o faz com a melhor cultura de prova do conjunto: conformance contra o `xgameruntime.dll` real, validação de GPU como oráculo de descritor, sondas de silício em gfx1036 e testes de execução bit-exata com mutação que derruba cada correção. A engenharia é sólida onde é medida (`b03b39b`, `7364bb4`, `9b7c93a`, `8f71091`) e honesta onde não é: `BLOCKERS.md` termina com 117 itens abertos declarados e o README afirma que o 3D está errado. Contra isso pesa o preço de como o código foi publicado: 1.131 arquivos e 266 mil linhas em um commit apagaram toda a arqueologia, e mensagens como `c95a909` — que descreve um fix de PSO e entrega 1.294 linhas de `xtfeventserver` — corroem justamente a auditabilidade que o resto do projeto construiu. O produto está em estágio inicial de fidelidade (Trackmania joga, nenhum título renderiza 3D correto), com três pontos de inferência ainda heurísticos (`b03b39b`, `bd185fa`, `4fcfe13`) e um CI que não bloqueia o check de workspace que já falhou uma vez. Vale copiar o tradutor, o harness de execução e os padrões de conformance; não vale copiar o fluxo de importação, os lotes de commit nem o gate.
