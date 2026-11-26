# Acertos e defeitos de XCL

## Ficha

| Campo | Valor |
|---|---|
| Repositório | `ivall/xcl` (o GitHub redireciona para `xcl-dev/xcl`) |
| Commits | 853 alcançáveis de `HEAD` (`8e87fc7`); 274 merges (32%); 311 arquivos versionados; período 2026-11-20 a 2026-11-25 |
| Licença | `GPL-2.0-only` (`Cargo.toml:8`; `LICENSE` é o texto da GPLv2) |
| Papel no ecossistema | Camada de compatibilidade estilo Wine para jogos Xbox Series X\|S: o código x64 do jogo roda nativo, as bibliotecas do console são reimplementadas (HLE) e o RDNA2 dos shaders é recompilado para HLSL→DXIL — sem emulação de CPU |
| Estado hoje | Minecraft Preview, Unicorn Overlord, Fortnite e XDefiant chegam a menus/UI e um título retail chega ao menu a 30 fps, com três backends de shader (HLSL padrão; DXIL direto e per-lane opt-in por `XCL_DIRECT_DXIL`/`XCL_PER_LANE`) e 24 defeitos catalogados nos 6 dias |

## Placar

| Dimensão | Nota | Justificativa |
|---|---|---|
| Engenharia | 4/5 | As correções atacam causa raiz com invariante (endereço de 64 bits em `f9e5ee9`, dono do valor em `a23b6ff`, identidade de cache em `bfb8133`), mas 20+ defeitos de corretude em 6 dias e um revert de correção (`70f8dc4`) cobram o preço. |
| Verificabilidade | 4/5 | Praticamente toda correção vem com teste que falha sem ela, incluindo testes de **forma** do código gerado (`425b869`), fuzz diferencial (`9230fbc`) e byte-identidade (`b45a6c9`); o furo é o CI sem GPU, onde o backend per-lane nunca executa (`9e27370`, `ci.yml`). |
| Documentação | 3/5 | `README.md` tem coluna "Limits" e diz o que falta, e `HANDOFF.md` (1854 linhas) registra medições reais; mas o diário único é ilegível como referência e o README afirma "works fully" para XDefiant sem nenhum commit que passe do title screen (`507c441`, `58309b9`). |
| Sustentabilidade | 2/5 | Dívida alta e concentrada: `real.rs` com 108 commits (`3885b86`, `c9c8677`), três caminhos de tradução com cobertura incomensurável, limite heurístico em vez de correção estrutural (`69a892c`), 40% dos commits em um autor e uma política de PR que depende de humano (`63b9930`). |
| Reaproveitamento | 4/5 | Loader/HLE, seletor de região por endereço completo, identidade de cache por fonte, barreira via `INDIRECT_ARGUMENT` e o padrão de teste com skip declarado são portáveis quase literalmente. |

**Média:** `3.4/5`

## Acertos

| # | Acerto | Commit | Por que importa | Reusável? |
|---|---|---|---|---|
| A-01 | Endereço efetivo de 64 bits calculado **antes** de escolher a janela de memória, com `window_read`/`window_write` separados e range-check contra a base da UMA (leitura fora da UMA = 0, escrita = descartada) | `f9e5ee9` | Elimina a classe inteira de miscompile silencioso que existia desde `f3b15b9`; o `default` que não apelida uma janela viva é a parte que ninguém escreve | sim |
| A-02 | Identidade de cache derivada por hash das fontes do produtor: `build.rs` hasheia `PER_LANE_SOURCES` em `XCL_PER_LANE_SOURCE_ID` e a chave vira `xcl-per-lane-<hash>` | `bfb8133` | Impede servir DXIL obsoleto depois de mudar o lowering, sem invalidar o cache do caminho HLSL; 20 linhas resolvem uma classe de bug que só aparece em campo | sim |
| A-03 | Limite heurístico de saída contra entrada: se `code.len() > 128 * program.len()`, re-traduz com `pc` blocks (`STRUCTURED_BYTES_PER_CODE_BYTE`, `mod.rs:641`) | `69a892c` | Troca "prever quais programas explodem" por "medir e mudar de estratégia"; 148 MB → 13,8 MB sem perder o ganho de `b8b1f49` (1 670 → 170 programas em `pc` blocks) | sim |
| A-04 | Barreira portável onde o driver ignora UAV barrier: duas transições de um buffer próprio da XCL para `INDIRECT_ARGUMENT` em volta de cada `ExecuteIndirect` | `4e3515e` | Resolve dependência real de dados sem sincronização grosseira, e sem tocar nos recursos do título (+196 linhas em `bind.rs`) | sim |
| A-05 | Correção mínima e completa: `Entry::Kernel => lw.setup_lanes()` incondicional mais o SGPR superior no bridge, para carry-ins e `v_cndmask` lerem o bit certo | `188ee17` | 6 inserções/1 remoção consertam um erro que corrompia 31 de 32 lanes em silêncio (`82c875d` documentou o sintoma antes de alguém consertar) | sim |
| A-06 | Invalidar o CSE block-local de `dx.op.wave*`/`dx.op.quad*` ao ver `dx.op.discard`, limpando o mapa do bloco | `50228bb` | Correção grosseira de propósito (perde CSE, não correção) e com teste de 1 chamada sem discard contra 2 com discard | sim |
| A-07 | Testes de GPU que **declaram** o skip: `SKIPPED <test>: <reason>` em stderr, contagem no fim de `scripts/test.ps1` e `XCL_REQUIRE_GPU=wave64\|mesh` para transformar skip em falha | `ff9430e` (e `9e27370`) | Torna visível a diferença entre "passou" e "não rodou" — o defeito mais comum em suíte com hardware heterogêneo; nunca `#[ignore]` | sim |
| A-08 | Teste de determinismo: tradução byte-idêntica entre execuções e entre threads | `b45a6c9` | Cache em disco e paralelismo de DXC só são seguros se a saída for determinística; é a invariante que sustenta `pso.rs` | sim |
| A-09 | Fuzz diferencial aleatório do emissor de DXIL, com sementes e contador de divergência | `9230fbc` | 1 253 linhas de gerador de casos acham o que teste escrito à mão não acha — e acharam defeitos reais (`ecb7547`) | sim |
| A-10 | Banco de stubs que **falha** em vez de devolver `S_OK` falso para DirectStorage e `xg_x` | `c2e1f8d` | Falha honesta no lugar de sucesso mentiroso: converte "jogo age estranho" em erro localizável, com 4 linhas de dado | sim |
| A-11 | Remover a exclusão do teste em vez de deixar o defeito escondido: `if !split` vira comparação sempre, comentário passa a "fixed by #1689" | `2a2d54b` (com `87d421b`) | Fecha o ciclo: o teste que mascarava D-19 passa a cobrir o caso; o diff é 6+/12− | sim |
| A-12 | Revert cirúrgico por merge (`git revert -m 1 778e329`), isolando #123 de #124, que entrou onze minutos depois, com o motivo registrado no PR e em `HANDOFF.md:957-959` | `70f8dc4` | É a decisão certa sob pressão: `13+/93−` remove exatamente um PR e deixa o resto do lote; num lote de 100+ merges por dia, é a única forma de separar | com ressalva (o custo está em F-01) |
| A-13 | Repositório traduzido para inglês de uma vez (98 arquivos de comentário/log/teste/docs) e recusa explícita de `rustfmt` global, mantendo as mudanças | `98d32ec`, `bb363d1` | Padroniza a língua sem perder o diff útil, e assume que a árvore não é `rustfmt`-clean em vez de fingir que é | sim (a decisão; o diff não) |
| A-14 | Cobertura declarada e fechada do pack: 26 698 de 26 698 programas retail traduzem, com amostragem que compila para DXIL sem erro | `e80ce8d` | Número verificável no lugar de "funciona"; é o que permite dizer no README "100% do pack" sem mentir | com ressalva (mede tradução, não execução) |

## Defeitos

| # | Defeito | Gravidade | Commit | Estado |
|---|---|---|---|---|
| F-01 | A correção `a787c52` (merge `778e329`, #123) foi mergeada e revertida 11 minutos depois porque o codegen de 64 bits por acesso derrubava o `amdxc64.dll` da AMD em XDefiant (`0xc0000005`, write em `0x2000000c`, CS `8e4af296b9d7fd9c`); o revert devolveu o defeito de janela por ~12 h (15:12 de 23/11 a 00:16 de 24/11), apagou o teste `gpu_memory_window_carry` (55 linhas) e esse teste nunca voltou com o nome | ALTO | `a787c52` → `70f8dc4` → `f9e5ee9` | corrigido (funcional); cobertura equivalente sob outro nome (`tests.rs:3326`), teste original perdido |
| F-02 | Janela UMA escolhida pelo dword alto da base, somando o offset em 32 bits: qualquer acesso que cruzasse 4 GiB lia/escrevia em outro recurso, silenciosamente, em `s_load`, `s_buffer_load`, MUBUF e FLAT | ALTO | `f3b15b9` | corrigido (`f9e5ee9`); `self.window(` não existe mais em `emit.rs`, só `window_read`/`window_write` (`emit.rs:1579`, `1590`) |
| F-03 | Estruturar saltos para cabeças de loop externo (`b8b1f49`) fez as guardas crescerem com o quadrado dos flags: 148 MB de HLSL a partir de 334 KB, 3 861 flags, guardas de 597 termos, DXC consumindo commit a ~1 GB/s até esgotar a máquina | ALTO | `b8b1f49` | corrigido por heurística (`69a892c`), não por repensar a forma estruturada |
| F-04 | O per-lane foi ligado no runtime (`ddfbf39`) com carry-ins e `v_cndmask` lendo sempre o bit 0 da máscara; a correção veio 30 min depois | MÉDIO | `77db49c` (causa) / `ddfbf39` (exposição) / `188ee17` (fix) | corrigido; janela opt-in, sem release no meio |
| F-05 | Typo de campo (`"OPSEL"` em vez de `"OP_SEL"`): `field_value` devolve `None`, `unwrap_or(0)` engole, e a guarda **nunca disparava** — VOP3 com `op_sel` era baixado como se não tivesse modificador | MÉDIO | `b3c0b37` → `ac34cb6` | corrigido em ~22 min, antes de o per-lane estar ligado |
| F-06 | Semântica de máscara errada na análise per-lane: `s_cbranch_vccz` e leituras escalares do SCC de uma máscara tratadas como por-lane (7 onde o console dá 0), e `s_setreg` de MODE virando no-op silencioso | MÉDIO | `1bab757` / `f5d3e05` → `66879ef` | corrigido por recusa (`uniform_over`); programas caem para HLSL |
| F-07 | f16 do tradutor direto: produtos arredondam para zero em vez de nearest-even (D-06) e `V_CVT_F16_F32` devolve 0 abaixo do menor denormal (D-07); mitigation é recusar, não corrigir | MÉDIO | `746f7dd`/`a8569bf` (origem), `fe8dc84`/`878afda` (recusa) | **aberto**: `DIFFERS_ON_1426` ainda lista `V_CVT_F16_F32`, `V_MUL_F16`, `V_PK_MUL_F16` (`lower.rs:1592-1599`) |
| F-08 | Caminho HLSL de runtime compila sem `-Gis`, então o driver AMD dobra `-0.0` em `+0.0`; só a referência de teste usa a flag — a referência ficou mais precisa que o produto | MÉDIO | `b1bfd1e` (só teste) | **aberto** no runtime (`dxc.rs` sem `-Gis`) |
| F-09 | Cache do per-lane com identidade fixa `xcl-per-lane`, sem as fontes do lowering: DXIL obsoleto servido do disco depois de uma correção de semântica (o "conserto" não aparecia) | MÉDIO | `ddfbf39` → `bfb8133` | corrigido; o mesmo padrão já existia correto para o HLSL, o que mostra que a lição era conhecida |
| F-10 | Teste que escondia defeito: `geometry_tests.rs` excluía a comparação com a forma split do caminho HLSL, com comentário descrevendo exatamente o sintoma (fase ES invadindo o GS, records vazios em GPU só-wave32) | MÉDIO | `8c6f1c3`/`a21d3df` (origem) → `f3242b2`/`87d421b` (fix) → `2a2d54b` (exclusão removida) | corrigido; o commit de introdução exato é `NÃO VERIFICADO`, o de fix é |
| F-11 | `ExecuteIndirect` lia argumentos indiretos antes da barreira que os escreve: 40 de 40 tentativas rodaram a contagem velha na RTX 5090 | ALTO | origem `NÃO VERIFICADO` (comportamento de título) → `4e3515e` | corrigido; `git log 4e3515e..HEAD -- bind.rs` vazio, sem regressão conhecida |
| F-12 | Classe `then_some` com aritmética ainda presente: `upper_lanes.rs:322` avalia `lane + n` mesmo quando a condição é falsa, a mesma família do panic de `dd566c0` | BAIXO | `55156f0` (linha), `dd566c0` (caso irmão corrigido) | aberto (sem overflow demonstrado: os operandos vêm de campos pequenos) |
| F-13 | `xcl shaders --export-hlsl` pulava registros não-compute **sem contagem nem aviso** e aceitava flags sem valor, o que fazia varredura de cobertura mentir | BAIXO | `4143a09` | corrigido |
| F-14 | `README.md` afirma XDefiant "works fully, as far as the game allows without its servers", mas nenhum commit do repositório passa do title screen e do texto da UI | BAIXO | `507c441`, `58309b9` (evidência mais profunda) | aberto (documental); gameplay além dos menus: `NÃO VERIFICADO` |
| F-15 | Forma de 64 bits por acesso, que o mantenedor culpou pelo crash do `amdxc64`, voltou em `f9e5ee9` no caminho switch, e não há no repositório nenhum registro de re-teste do CS `8e4af296b9d7fd9c` | MÉDIO (risco, não fato) | `f9e5ee9` vs `70f8dc4` | aberto — **HIPÓTESE**: os relatos de #792 são em RDNA4, sem menção a XDefiant |

## Erros de processo

| # | O que aconteceu | Evidência |
|---|---|---|
| P-01 | Dois PRs grandes (#123 e #124) mergeados no mesmo minuto, sem smoke de título entre eles; onze minutos depois um precisou ser revertido por merge. O lote de ~100 merges/dia foi o que tornou o revert caro | `778e329` e `828d9db` (15:01), `70f8dc4` (15:12); `HANDOFF.md:957-959` |
| P-02 | Um teste foi escrito para **não** comparar o caso que falhava, com o comentário descrevendo o sintoma, e ficou assim até o fix — o defeito ficou invisível na suíte | `geometry_tests.rs` antes de `2a2d54b`; remoção em `git show 2a2d54b` |
| P-03 | O CI roda em `windows-2025` com "no GPU e no game data": os testes per-lane pulam por construção (WARP não tem waves de 32 lanes), então a metade mais nova do backend **nunca** é exercitada automaticamente — a cobertura depende de GTX 1650/RTX 4090/RX 9070 de contribuidores | `ci.yml` (comentário de cabeçalho) + `9e27370` ("`XCL_ADAPTER=warp`: 876 passed, 0 failed, 22 ignored") |
| P-04 | Política de elegibilidade por **modelo de IA** ("no PRs written with small or open-weight models", com lista nominal), aplicada por script, enquanto o próprio histórico mescla commits com `Co-Authored-By: Claude Opus 5.5` e seis commits de `Claude <noreply@anthropic.com>` | `9ab7d76`, `pr-check.yml`, `CONTRIBUTING.md`; `69a892c` (trailer), autor `Claude` em `c858190`, `608a79b`, `8adb9dd` |
| P-05 | Documentação por apêndice: `HANDOFF.md` com 1854 linhas e 94 commits é o único lugar de várias decisões e medições; os specs (`docs/specs/`) e o README não o substituem, e o próprio `AGENTS.md` manda "search it rather than reading it all" | `HANDOFF.md` (1854 linhas; 94 toques, o 2º arquivo mais alterado), `AGENTS.md` |
| P-06 | 274 merges com mensagem sem corpo ("Merge PR #212", "Merge PR #1037"): o contexto fica só no commit de branch, que não é o que `git log` da `main` mostra | `163b55e`, `dc6a283`, `010e1ec` e outros 200+; `git log --format=%s` só devolve `Merge PR #N` |
| P-07 | Bot de stale fechando PR após 7 dias sem atividade, com `operations-per-run: 300` e `sort-by: updated`, num repositório com "Nearly 1000 PRs are open" | `1d027e5` + `e9ac9f0`; `.github/workflows/stale.yml` (`days-before-pr-stale: 3`, `days-before-pr-close: 4`) — nenhum fechamento indevido observável: `NÃO VERIFICADO` |
| P-08 | Descompasso entre o número anunciado e o medido em documento: README diz "370+ automated tests" enquanto o clone tem 1 035 `#[test]`. O erro é na direção conservadora, mas mostra que a contagem não é gerada do código | `README.md:29`; contagem de `#[test]`/`#[tokio::test]` em `*.rs` |
| P-09 | Regra de commit ("um teste que falha sem o fix") é cumprida, mas a regra de "cada commit compila e passa" foi violada pelo lote: o per-lane entrou no runtime 30 min antes da correção de carry-ins | `AGENTS.md` ("every commit builds and passes the tests") vs `ddfbf39` → `188ee17` |

## O que copiar

- **`crates/xcl/build.rs`** — hash FNV das fontes **por caminho de tradução** (`TRANSLATOR_SOURCES`, `PER_LANE_SOURCES`, `cargo:rerun-if-changed`), usado como chave de cache (`bfb8133`, `242a6ea`).
- **`gpu/recomp/emit.rs`** — `window_read`/`window_write` com endereço efetivo de 64 bits, base da UMA por slot de I/O e `default` que não apelida janela viva (`f9e5ee9`).
- **`hle/d3d12x/bind.rs`** — barreira via transição `INDIRECT_ARGUMENT` em buffer próprio em volta de `ExecuteIndirect` (`4e3515e`).
- **Padrão de teste**: `skip_test(reason)` + `SKIPPED` em stderr + contagem no runner + variável que transforma skip em falha (`ff9430e`, `9e27370`); teste de **forma** do código gerado desmontando o DXIL (`425b869`); teste de byte-identidade (`b45a6c9`); fuzz diferencial com sementes (`9230fbc`).
- **`lower.rs::DIFFERS_ON_1426`** — lista de divergências conhecidas por opcode, com sintoma e relato no comentário (`878afda`, `fe8dc84`), desde que acompanhada de teste que falhe quando o item for corrigido.
- **`69a892c`** — limite saída/entrada com re-tradução em outra forma, como padrão de defesa contra codegen patológico.
- **`data/stubdb.toml` + `c2e1f8d`** — stub que falha em vez de devolver `S_OK`.
- **`CONTRIBUTING.md` + `scripts/pr_check.py`** — exigir bloco `xcl-smoke` medido, com commit e hardware nomeados, e recusar números escritos à mão (`63b9930`, `0e946cf`, `1d83fd1`).
- **`a23b6ff`** — `own(b, v, bit)`: materializar escrita em bloco parcial (`or v, 0`) para que a leitura posterior seja detectada, em vez de ajustar o detector.

## O que não copiar

- **Reverter correção sem re-testar o caso que falhou.** `70f8dc4` desfez o PR correto e o bug de janela voltou por ~12 h; `f9e5ee9` reintroduziu a forma acusada e nenhum commit prova re-teste do CS do XDefiant (F-15). Revert precisa de issue de acompanhamento com o caso concreto.
- **Teste que exclui a comparação para passar** (`if !split` em `geometry_tests.rs`, removido só em `2a2d54b`). Teste que documenta o sintoma no comentário e não falha é pior que teste ausente.
- **Três backends simultâneos com cobertura incomensurável** (HLSL, DXIL direto, per-lane): dois opt-in, sem número final de cobertura e com `DIFFERS_ON_1426` apodrecendo. Um backend por vez, com métrica de aceitação publicada.
- **`HANDOFF.md` como fonte primária** (1854 linhas, 94 toques). O conteúdo é bom; o formato não escala e some do `git log`.
- **Política de PR por fornecedor de modelo** (`9ab7d76`, `pr-check.yml`) — critério não técnico, aplicado por script, contra o próprio histórico do repositório.
- **Bot de stale agressivo** com centenas de PRs abertos (`.github/workflows/stale.yml`), e **merges "Merge PR #N" sem corpo** (274 ocorrências).
- **Heurística como solução final** quando o problema é de representação: `69a892c` é uma boa mitigação, mas o limite `128` continua sendo um número mágico que ninguém derivou (`git log -S` do símbolo devolve só ele).

## Veredito

O XCL é um projeto de seis dias que fez o que quase ninguém faz: rodar jogos de console sem emular CPU, com HLE por API, recompilador RDNA2→HLSL e um tradutor direto para DXIL escrito em paralelo. A qualidade do **diagnóstico** é a parte fora do comum — quase toda correção cita quem achou, em que GPU, com que número, e traz um teste que falha sem ela; `f9e5ee9`, `bfb8133` e `69a892c` são correções de representação, não de sintoma, e valem ser copiadas. O que não está à altura é o **processo**: 853 commits e 274 merges em cinco dias significam lotes grandes, CI sem GPU onde justamente o backend novo nunca roda, um teste que escondeu um defeito por dois dias, um revert que devolveu um bug de memória por meio dia e uma política de contribuição baseada em fornecedor de modelo. O resultado é um código de engenharia acima da média dentro de um ritmo que garante regressões: nada aqui sugere má-fé, e muito sugere pressa. Como base para o repositório unificado, o valor está nos invariantes e nos padrões de teste; o que deve ficar de fora é o modo de operação.
