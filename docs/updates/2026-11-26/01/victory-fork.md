# fxceless/victory-fork — atualização de 2026-11-26 01:00

## Resumo

Nove commits na `main` (`bc6ee68e` → `988aa416`): o `#1255` foi mesclado e impede que o aviso de Auto HDR
do Windows desligue o driver watch no primeiro segundo de cada execução; fora dele, um commit corrige a
busca de slots dinâmicos que não enxergava um run livre atravessando o cursor (porte manual de
`c1c32819e9` do upstream), outro impede que uma versão antiga de save que não pode ser apagada derrube
todos os saves seguintes, e o resto é diagnóstico (três runs novos, contadores de motivo de recusa,
lote de cópias opcional e CI serial).

## Commits novos (9)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `9686ea30` | fxceless | Tests: the card-only depth-bounds test also skips where the adapter is Microsoft's or does not answer | Um pai (`bc6ee68e`), +12/−6 em `host.rs`. `Host::software()` (só em teste) passa a considerar software o adaptador cujo `VendorId` é `0x1414` (Microsoft) ou que não responde a `GetDesc1()`, além do flag `DXGI_ADAPTER_FLAG_SOFTWARE`. Na máquina de CI sem placa o teste de depth-bounds rodava e falhava. |
| `89e41272` | fxceless | CI: the gpu crate's tests run one at a time | Um pai (`9686ea30`), +6/−1 em `.github/workflows/ci.yml`: `cargo test --workspace --locked --exclude victory-gpu` e um passo separado `cargo test -p victory-gpu --locked -- --test-threads=1`, porque o compilador de shader do WARP derrubava o processo (access violation) com vários testes ao mesmo tempo. |
| `db53666c` | fxceless | diagnostics: runs 298 (profiler, standing) and 299 (late-wait lists: refused tables are not held by flights), scrubbed | Um pai (`89e41272`), +62.647 em três arquivos: `diagnostics/run-298-2026-11-26-profiler-standing.txt` (+30.843), `run-299-2026-11-26-late-wait-lists.txt` (+31.802) e duas linhas de índice no README. Run 298: perfil amostrado, 39,5 ms/frame de espera da thread de apresentação num lock e 49,6 ms/frame somados nas seis threads de gravação. Run 299: com `VICTORY_LATE_WAIT_LISTS=128`, 5,0 % de 28.158 tabelas/frame recusadas — as recusas **não** vêm de flights segurando slots. |
| `01e844fe` | fxceless | Merge pull request #1255 from CarverXI/fix/drvwatch-auto-hdr | Merge de dois pais (`db53666c`, `ca8ce8a3`), +14/−6 em `drvwatch.rs`. A assinatura de evento vira a constante `QUERY` com `Provider[...] and (Level=1 or Level=2 or Level=3)`: só crítico, erro e aviso dos drivers de vídeo e da fonte `Display`. O aviso 4125 (Auto HDR, informação) deixa de disparar o driver watch, que antes desligava as janelas de draw no primeiro segundo de toda execução. |
| `da4f2495` | fxceless | Diagnostics: a line of the descriptor tables made a frame (repeats, slots, time by part) and one of the host heap's locks | Um pai (`01e844fe`), +409/−3. `descriptor/host.rs` (+134) cria `LOCKS: [AtomicU64; 15]` e `locked()` (try_lock e, se ocupado, cronometra a espera) para os locks `made`, `kept` e `next`; `draw.rs` (+274) cria `TABLES: [AtomicU64; 26]` com contagem por motivo (mesma gravação, outra lista no frame, frames anteriores, só os constantes mudam), por número de slots, por tipo de view e tempo por parte; `copystat.rs` chama `draw::tables_line(frames)`. Só diagnóstico. |
| `6a42ed27` | fxceless | Saves: an older version that cannot be removed no longer fails the container | Um pai (`da4f2495`), +48/−3 em `victory-hle/src/modules/xgamesave.rs`. `current()` deixa de propagar o erro de `remove_dir_all` de versão `.new` e de versões antigas: registra `debug!`/`info!` e segue, tomando a maior versão como o container. Antes, um arquivo de versão antiga aberto por outro programa sem compartilhamento de exclusão (antivírus, backup) tornava o container ilegível e fazia **todo** save falhar enquanto o arquivo estivesse preso. Teste novo abre a versão 1 sem compartilhamento, salva e lê duas vezes, solta e confere que sobra uma versão. |
| `4d0c6345` | fxceless | Dynamic slots: a free run across the search's starting place is found; refused tables are counted by cause | Um pai (`6a42ed27`), +83/−5. `descriptor/host.rs`: a segunda metade da busca passa a ir de 0 até `(cursor + n - 1).min(end)` em vez de parar no cursor, então um run livre que atravessa o ponto de partida é encontrado inteiro; antes a tabela era recusada com espaço para ela e o atalho `too_long` recusava todas as tabelas do mesmo tamanho até a próxima liberação. É o porte manual de `c1c32819e9` do upstream. `draw.rs` acrescenta `DYNAMIC_WHY[4]` (slots insuficientes, atalho, busca sem run, runs atravessando o cursor) e `DYNAMIC_LOW` (menor número de slots livres visto). |
| `6aed0780` | fxceless | Card copies: the send-and-hash list can record its copies, one state change and its hash passes in three steps (VICTORY_CARD_SYNC_BATCH=1\|alt, off by default) | Um pai (`4d0c6345`), +186/−34 em `host.rs`. Com o switch, a lista de envio e hash copia primeiro todas as faixas alteradas, faz todas as mudanças de estado numa chamada e roda todos os passes de hash depois, em vez de alternar cópia→compute por cópia. Vem do pull request #1227 do fork (lá ligado por padrão), transportado à mão; `alt` alterna 20 s com e 20 s sem. Desligado por padrão. Teste do PR original trazido como `#[ignore]` (precisa da placa). |
| `988aa416` | fxceless | diagnostics: run 300 (why tables are refused: the ring of dynamic slots is full; lock waits; the send-and-hash list in three steps), scrubbed | Um pai (`6aed0780`), +37.398 em `diagnostics/run-300-2026-11-26-slots-by-cause.txt` mais uma linha de índice. Conclusão registrada: o anel de 262.144 slots dinâmicos enche (0 slots livres em toda leitura com recusa; até 2.836 tabelas/frame recusadas de 18.000–31.000 pedidas) e nenhuma tabela se perdeu na busca; as threads de gravação esperam 75–170 ms/frame somados no lock do heap; 12–15 % das tabelas repetem os registros de uma anterior na mesma gravação. |

## Pull requests abertos (1)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 1258 | GPU: a title heap a capped host heap does not hold whole keeps its lower 393,215 slots; the rest are dynamic | notquiteog, +68/−5 em `descriptor/host.rs`. Com o heap do host limitado a 1.000.000 de views (NVIDIA), o heap do título (1.245.184) mais os dinâmicos não cabe: hoje ficam 737.855 slots de título e 262.144 dinâmicos, e um frame do mundo aberto pede ~200.000 dinâmicos, o que esgota e dropa draws. O PR entrega ao título só os primeiros 393.215 slots e torna o resto (606.784) dinâmico, com `VICTORY_DYNAMIC_SLOTS=<n>`. Mede 13.000–60.000 tabelas recusadas por 5 s → 0, e 350.000 dinâmicos livres no pior caso. | **Não há linha em `victory-fork` para isso.** É a mesma classe de `VIctory` `D-01` (`edf64e2`) e de `Belobug-AMD` `D-05` (sem corretor): exaustão de slots dinâmicos de descritor, com draws dropados. Candidato a defeito novo. O PR mede numa árvore derivada de `avatarmanz/VIctory-7900XTX`, não nesta. |

## Pull requests mesclados (1)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| 1255 | The driver watch counts warnings and errors only, not the Auto HDR notice | `01e844fe` (conteúdo em `ca8ce8a3`; mergedAt 2026-11-26T03:41:50Z) | Exatamente o que o PR prometia, e só: +14/−6 em `drvwatch.rs`, a `QUERY` com `Level=1 or 2 or 3` e o teste que compara a assinatura. Nenhuma outra mudança no watch. O efeito medido pelo autor (o switch disparando ~1 s depois do início, com `VICTORY_INDIRECT_WINDOW=1` sem efeito em máquina com Auto HDR) é o que a linha de comentário no código passa a explicar. |

## Cruzamento com o catálogo

- Corrige D-xx? `4d0c6345` corrige, neste fork, o defeito que o catálogo registra como **`DexNom` `D-16`**
  ("falsa exaustão de descritores dinâmicos quando o run livre cruza o cursor", corrigido lá em
  `c1c3281`). O hash do upstream citado no próprio commit é `c1c32819e9`. Não existe linha equivalente em
  `victory-fork`; o catálogo deste fork tem 17 linhas e nenhuma trata da busca de slots. Vale acrescentar
  a linha com `corrigido = 4d0c6345`, remetendo a `DexNom D-16`.
- Abre defeito novo? Dois candidatos, ambos corrigidos nesta faixa e ausentes do catálogo:
  - `01e844fe` (`#1255`): o driver watch assinava **todas** as entradas da fonte `Display`, incluindo
    informações como o 4125 (Auto HDR), e desligava as janelas de draw no primeiro segundo de cada
    execução em máquina com Auto HDR — `VICTORY_INDIRECT_WINDOW=1` nunca valia ali. É um defeito de
    instrumentação com efeito de desempenho, não catalogado.
  - `6a42ed27`: um arquivo de versão antiga de save preso por outro processo (sem exclusão de
    compartilhamento) tornava o container ilegível e fazia **todo** save falhar. Ausente do catálogo.
- Não mexe em linha aberta: `D-12` (loads escalares agrupados lendo além da view), `D-15` (page mapping
  revertido) e `D-16` deste fork (perdas de device sem causa) seguem sem corretor nesta faixa.
- Nota de catálogo: `D-17` ("dívida de verificação: *this merged tree was built; its test suites were not
  run*") fica mais estreita — `89e41272` põe a suíte do `victory-gpu` para rodar serializada no CI, mas
  num runner sem placa (WARP). Não muda o texto de `D-17`; apenas reduz o motivo.

## Confiança

Verificado por diff: os nove commits foram lidos em `gh api .../commits/<sha>`, com pais conferidos em
cadeia (`bc6ee68e → 9686ea30 → 89e41272 → db53666c → 01e844fe → da4f2495 → 6a42ed27 → 4d0c6345 →
6aed0780 → 988aa416`), e `01e844fe` de dois pais (`db53666c`, `ca8ce8a3`). `mergeCommit.oid` do `#1255`
confere com o commit documentado; `merged_at` 2026-11-26T03:41:50Z é posterior ao cursor, então a
detecção de mesclados e a de commits concordam (1 = 1).

Não verificado: os números dos runs 297–300 (profiler, slots, locks) não foram reproduzidos; são as
transcrições que o repositório versiona. Não rodei a suíte nem o teste `#[ignore]` do
`VICTORY_CARD_SYNC_BATCH`. O efeito do Auto HDR só é verificável em máquina com o recurso ligado — o
commit é a fonte da afirmação.
