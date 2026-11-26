# Acertos e defeitos de R0Y00/Xbox2Windows

## Ficha

| Campo | Valor |
|---|---|
| Repositório | `R0Y00/Xbox2Windows` |
| Commits | 4.076 em `HEAD` (`6218beb2`); 4.499 em todas as refs; 423 existem só em branches nunca mergeadas |
| Licença | MIT |
| Papel no ecossistema | Pesquisa de recompilação estática Xbox Series X\|S (gfx1013) para x64 Windows, com tradutor de shader, lifters de CPU, shims de runtime e o GTA VI como estudo de caso declarado |
| Estado hoje | Pesquisa em andamento, não produto: prova propriedades de fixtures e ferramentas, declara o que não provou e cresce a ~800 commits/dia |

## Placar

| Dimensão | Nota | Justificativa |
|---|---|---|
| Engenharia | 4/5 | Correções mínimas e na causa raiz, mas `0478a386` e `9727c306` regrediram o próprio caminho que consertavam e `ffb70f6a` criou o defeito que `07f2464f` teve de desfazer 22 minutos depois |
| Verificabilidade | 5/5 | 855 `add_test` no `CMakeLists.txt` em `6218beb2`, 359 deles com `SKIP_RETURN_CODE 77`, CI que quebra com `--no-tests=error`, e cada correção examinada traz um controle que **deve** falhar (`583178d4`, `2a898640`, `f37fe7b8`) |
| Documentação | 4/5 | O README declara o que não está provado (`6218beb2:README.md:7`) e `docs/experiments.md` amarra resultado a adaptador e driver, mas o arquivo tem 1,7 MB e já ficou desatualizado depois de `b87a99a3` até `5b29fe0f` |
| Sustentabilidade | 2/5 | 4.076 commits em cinco dias, 44,7% de merge, 313 merges de sincronização, 423 commits órfãos em branches e 142 identidades de autor: o custo de manter o repositório cresce mais rápido que o código |
| Reaproveitamento | 3/5 | As ferramentas (`tools/disasm_rdna2.py`, `tools/startup/gfx2hlsl.py`, `tools/pe_map.cpp`, `src/xmem_mapped_pages.cpp`) e ~15 padrões de correção são portáveis; shims e fixtures são específicos de Scarlett e do título |

**Média:** `3.6/5`

## Acertos

| # | Acerto | Commit | Por que importa | Reusável? |
|---|---|---|---|---|
| A-01 | Controle negativo obrigatório: a correção vem junto com uma prova que precisa falhar | `583178d4` | 464 linhas de teste para 10 linhas de produto, exercitando 510 formas em FXC **e** DXC sem `-Gis`; sem o controle o defeito é invisível no compilador que o projeto realmente usa | sim |
| A-02 | Recusa por nome em vez de tradução aproximada | `c3c3ab3c` | `unorm`, `r128`, `tfe`, `lwe`, `d16`, `invalid_nsa`, `dmask` vazio e contagem errada de registradores de endereço viram `Unsupported`; 2D `a16` é o único caso traduzido, e o resto fica recusado por decisão escrita | sim |
| A-03 | Revert provado por blob, não por mensagem | `22989d61` | `98c69432` é o blob antes (`1f24626b^`) e depois (`22989d61`), e o mutado `9a649fe4` não existe em HEAD: a limpeza é auditável em dois comandos | sim |
| A-04 | Revisão de cache bumpada nos dois lados (tool e runtime) | `c3c3ab3c` | `d3d12_draw.inl:201` e `gfx2hlsl.py:2176` concordam em `x2w-startup-shader-6`; sem isso um bytecode pré-fix sobreviveria ao fix, porque `shader_cache/` é o caminho normal | sim |
| A-05 | Evidência externa com adaptador, driver e data | `995fc999`, `b40db732`, `d0e6f977` | relatórios de terceiros em hardware real acumulados em série (quatro, sete, quinze por lote), não só WARP — o único teste que o CI não consegue fazer | sim |
| A-06 | Skip de capacidade com código dedicado e documentado | `0b649c69` | 359 testes com `SKIP_RETURN_CODE 77` em HEAD; sem isso um PC sem adapter reporta **Failed** e a suíte perde o sinal | sim |
| A-07 | Estado sintetizado separado do estado do chamador | `07f2464f` | `default_viewport_w/h` guarda o que a camada encaminhou e `viewport_set` volta a significar só "o título definiu"; é o antídoto exato do defeito de `ffb70f6a` | sim |
| A-08 | Seam de injeção de falha para provar rollback | `21e8f5c0` | `X2W_XMEM_TEST_COMMIT_FAILURE` (`src/xmem_mapped_pages.cpp:85`, `CMakeLists.txt:223`) recusa o primeiro commit sem consumir commit do sistema, então o caminho de falha é testável | sim |
| A-09 | Cobertura reversa de tabela de classificação | `13745b1b` | 16 amostras, cada uma casada de volta ao índice, mais a exigência de que **toda** entrada tenha amostra: entrada nova sem teste passa a falhar | sim |
| A-10 | Correção que reduz o escopo do shim | `e50a72bc` | reconhece só `0x69` e deixa o fallback **depois** da recusa real do desktop, em vez de sequestrar todo tipo indireto futuro | sim |
| A-11 | Documentação tratada como contrato de medição | `5b29fe0f` | a seção de custo do loop é reescrita quando `b87a99a3` muda o ciclo do `.fail`; o commit declara que está alinhando doc e código, não "melhorando o texto" | sim |

## Defeitos

| # | Defeito | Gravidade | Commit | Estado |
|---|---|---|---|---|
| F-01 | Cópia da assinatura de imagem libera a superfície antes do fence; command list de D3D12 não retém recurso, então a cópia enfileirada pode ler memória liberada | ALTO | `8c2d82f6` → `2a898640` | corrigido |
| F-02 | Split de placeholder sem checagem de posse: `VirtualFree(MEM_RELEASE\|MEM_PRESERVE_PLACEHOLDER)` atinge reserva do chamador e destrói o estado dele | ALTO | `0478a386` → `21e8f5c0` | corrigido |
| F-03 | Falha no commit do primeiro slot deixa o slab aberto e contabilizado, contra o "leaves the slab state unchanged" da própria mensagem | ALTO | `9727c306` → `21e8f5c0` | corrigido |
| F-04 | f32 `nlt/nle/ngt/nge/o/u` dependiam da forma-fonte: FXC e DXC sem `-Gis` dobram `!(a OP b)` e `a != a`, e o resultado sai errado em silêncio sob NaN | ALTO | `e658fe24` → `583178d4` | corrigido |
| F-05 | Coordenadas 2D `a16` lidas como dois binary32 (UV errado) com o cache de shader sem mudança de revisão, permitindo servir bytecode pré-fix | ALTO | `c3c3ab3c` | corrigido; commit introdutor NÃO IDENTIFICADO |
| F-06 | Correção que regride o próprio caminho: marcar `viewport_set` ao encaminhar o default impede a regeneração quando o render target 0 muda de tamanho | ALTO | `ffb70f6a` → `07f2464f` | corrigido |
| F-07 | Snapshot por região em vez de por braço: em if/else com skip, o estado de loads pendentes volta ao do opener | MÉDIO | `6b8ad31c` → `05ef4d92` | corrigido |
| F-08 | Container de save aceita `begin_update` depois de fechado porque testa `names_[container].empty()` e não `refs_[container] == 0` | MÉDIO | `ca92741e` → `2260b7a4` | corrigido |
| F-09 | Gate de log por `InterlockedIncrement(&c) <= N` estoura em 2^31 e reabre o log; um consumidor que faz poll reativa o caso | MÉDIO | `e658fe24` → `f360d30a` | corrigido em parte: `d3d12_xs_device.cpp:527` e `:557` mantêm o padrão sob guarda |
| F-10 | `CreateCommandSignature` sequestrava qualquer tipo acima de `DISPATCH_MESH` e lia `desc`/`pArgumentDescs`/`out` sem validar | MÉDIO | `71d4bff1` → `e50a72bc` | corrigido |
| F-11 | Call hooks do debugger armados por padrão: com file tracing ligado, rodada normal para o processo | MÉDIO | `ac539650` | corrigido; commit introdutor NÃO IDENTIFICADO |
| F-12 | Publicação de shader não atômica e retomada travada depois de uma rodada manifest-only; bytecode podia sair ao lado de manifesto antigo | MÉDIO | `b87a99a3` | corrigido; commit introdutor NÃO IDENTIFICADO |
| F-13 | Endpoint de triagem contava first-chance step-over como parada do debugger — evidência falsa num projeto cujo produto é evidência | MÉDIO | `ef9a0798` → `313803bc` | corrigido; log antigo sem `.out` do launcher segue sem parada confirmada |
| F-14 | Fixture MASM isolado trocado por programa de pseudo-ops ao reutilizar a variável `asm`; a cobertura caiu sem falhar nada | MÉDIO | `5b2dadd7` → `14d6023e` | corrigido dentro do mesmo PR (`66423e45`) |
| F-15 | Tabela `SHADER_REASONS` nunca alimentada pela própria suíte: pattern que não casa dobra uma causa em `other` em silêncio | MÉDIO | `71ec5842` → `13745b1b` | corrigido |
| F-16 | Estado "stranded" do sampler mantém superfície, allocator, lista e readback até o fim do processo, sem contagem nem relatório | MÉDIO | `2a898640` | aberto (deliberado, evita use-after-free) |
| F-17 | Testes de hardware sem `SKIP_RETURN_CODE 77` reportavam falha em vez de skip em PC sem adapter | BAIXO | `0b649c69`, `86c225cc` | corrigido para `_hardware`; `_warp` fica sem a propriedade por decisão |
| F-18 | Leitor do trap log abre sem compartilhamento e falha exatamente nos adapters que mantêm o handle aberto; WARP faz skip, então o CI não viu | BAIXO | `99f097ad` → `2c1dcdd6` | corrigido |
| F-19 | Critério de completude do shader é `os.path.getsize(path) > 0`: um `.cso` truncado passa e só falha no consumo | BAIXO | `b87a99a3` | aberto; consequência prática NÃO VERIFICADO |
| F-20 | Nenhum teste força a igualdade das duas constantes de revisão de cache; a coincidência é conferida à mão | BAIXO | `c3c3ab3c` | aberto |

## Erros de processo

| # | O que aconteceu | Evidência |
|---|---|---|
| P-01 | Uma mutação que desliga o viewport default foi commitada em `main` e ficou lá 88 segundos, para provar que um teste criado 7 minutos antes pegava a regressão | `1f24626b` (17:38:50) e `22989d61` (17:40:18); `ffb70f6a` (17:31:05) criou o teste; 1 arquivo, 1 linha em cada ponta. Não há registro de build publicado no intervalo (NÃO VERIFICADO), mas o histórico de `main` contém uma regressão deliberada |
| P-02 | 44,7% do histórico é merge, e 313 desses merges são apenas re-sincronização de branch (`Merge remote-tracking branch 'origin/main' into …`), sem contribuição própria | `6218beb2`: 1.823 merges de 4.076, 1.332 com assunto `Merge pull request`, 313 de sincronização; os assuntos mais repetidos são `… into review` (9) e `… into HEAD` (7) |
| P-03 | 423 commits vivem em 100+ branches nunca mergeadas, e o mesmo autor aparece sob várias identidades | `6218beb2`: 4.499 commits em todas as refs contra 4.076 em `HEAD`; 142 identidades, com `R0Y00` em 3 e-mails distintos e `tedin7` em 3, e `1245206726@qq.com` em duas grafias de nome (65 + 60 commits) |
| P-04 | Um PR entrou em `main` com o defeito e o conserto de teste no mesmo merge, o que apaga o sinal de revisão | `66423e45` tem `5b2dadd7` (degrada o fixture) e `14d6023e` (restaura) na mesma linhagem |
| P-05 | A doc ficou dizendo o contrário do código, e o conserto veio em ramo paralelo, sem referência cruzada | `b87a99a3` (01:57+0300) muda o ciclo do `.fail`; `5b29fe0f` (00:34+0100) corrige a seção. `git merge-base --is-ancestor` falha nos dois sentidos: a cadeia é inferida por tempo e texto, não declarada |
| P-06 | Mensagem de commit promete garantia que o diff não entrega | `9727c306` escreve "fails the call with the system error and leaves the slab state unchanged", mas o slab ficava aberto; `21e8f5c0` é o commit que torna a frase verdadeira |
| P-07 | A manchete de shaders usa um número de outro objeto que o da tabela logo abaixo, sem marcador que impeça a leitura conjunta: "compilation route" 26.500/26.503 contra 851/12.101 que compilaram, montaram pipeline e deram pixel controlado | `6218beb2:README.md:11` contra `:30` (razão de ~31×) |

## O que copiar

- **A convenção de skip 77** (`CMakeLists.txt`, 359 usos em `6218beb2`, introduzida em `0b649c69` e reescopada em `86c225cc`): código dedicado a "sem capacidade", separado de falha.
- **O seam de injeção de falha por variável de ambiente** (`X2W_XMEM_TEST_COMMIT_FAILURE`, `src/xmem_mapped_pages.cpp:85` + `CMakeLists.txt:223`, de `21e8f5c0`): é o que torna rollback testável sem mock de kernel.
- **O lowering `ORDERED` para semântica IEEE** (`tools/startup/gfx2hlsl.py`, `583178d4`): teste de bits em vez de `a == a`, validado em FXC **e** DXC sem `-Gis`.
- **O padrão de pin de expressão gerada** (`2e5fdca4`, `SDWA_MOVREL_PINS`): cinco mutações de uma linha passam a falhar, o que fixa o texto emitido e não só "compila".
- **A cobertura reversa de tabela** (`13745b1b`): toda entrada de uma tabela de classificação precisa de amostra, e a amostra precisa casar de volta.
- **O par de constantes de revisão de cache bumpado nos dois lados** (`c3c3ab3c`, `d3d12_draw.inl:201` / `gfx2hlsl.py:2176`).
- **A separação estado-do-título vs estado-sintetizado** (`07f2464f`): qualquer camada que invente default registra o que ela encaminhou e de que tamanho.
- **O protocolo de evidência externa** (`995fc999`, `b40db732`): adaptador, driver, build e data em todo relatório de hardware.
- **O formato de triagem com "confirmed debugger stop" separado de "observed exception"** (`313803bc`): evidência de parada exige registro explícito, não inferência.

## O que não copiar

- **O ritmo**: 4.076 commits em cinco dias com 44,7% de merge produz um histórico em que arqueologia custa caro e revisão vira carimbo. `6218beb2`.
- **Merges de sincronização**: 313 `Merge remote-tracking branch … into <branch>` são puro ruído de deriva de branch. `6218beb2`.
- **Branches órfãs**: 423 commits em 100+ branches nunca mergeadas, sem marcador de abandono. `6218beb2`.
- **Um único doc de 1,7 MB**: `docs/experiments.md` recebeu 1.239 commits não-merge; é a maior fonte de conflito de merge do repositório.
- **Um `CMakeLists.txt` de 578 KB com 855 `add_test`**: registro de teste como texto manual garante conflito em quase todo PR.
- **Testes 3,2× maiores que o produto**: 243.915 linhas em `tests/` contra 77.300 em `src/`+`tools/` (em `6218beb2`). Justificável em pesquisa, insustentável como base de produto.
- **A mutação em `main`** (`1f24626b`): o objetivo era legítimo, o veículo não; o mesmo teste podia ser provado em branch ou com `--mutate`.
- **O critério de completude por tamanho de arquivo** (`b87a99a3`): `getsize > 0` aceita `.cso` corrompido.
- **A conclusão de que os fixtures provam o título**: nada no repositório estabelece startup do EXE original nem gameplay (`6218beb2:README.md:7`).

## Veredito

O Xbox2Windows é um projeto de pesquisa com disciplina de prova acima do normal: quase toda correção examinada chega com um controle que precisa falhar, o vocabulário dos commits é sustentado pelo código dos testes, e o README declara em vez de esconder o que não foi verificado. Os defeitos reais são poucos e têm uma forma única — estado que atravessa uma fronteira (command list, transação de alocador reserva/commit, compilador de shader sem `-Gis`, braço mascarado, camada que sintetiza default) — e quase todos foram achados e fechados no mesmo dia, alguns em menos de meia hora, como `ffb70f6a` → `07f2464f`. O que pesa contra não é a qualidade técnica e sim a sustentabilidade: cinco dias, 4.076 commits, 44,7% de merge, 313 merges de sincronização, 423 commits em branches abandonadas e 142 identidades de autor para um punhado de pessoas, com um único documento de 1,7 MB como memória do projeto. É um repositório excelente para extrair padrões e péssimo para herdar como base de código.
