# fxceless/victory-fork — atualização de 2026-11-26 00:00

## Resumo

Nove commits novos (`fc63ce16` → `bc6ee68e`), dos quais cinco são execuções de diagnóstico publicadas
(runs 293 a 297) e quatro são código: a passagem de buffer-views do heap ganhando um modo que olha só
slots marcados, o reaproveitamento de pipeline e cópia de root view dentro de uma lista (ambos atrás de
switch, desligados), conserto de lint que quebrava o CI e três testes que dependiam da máquina. Cinco
PRs abertos; nenhum merge na janela.

## Commits novos (9)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `bcce702f` | fxceless | diagnostics: run 293 (the lookup cache measured: about 14 ms a frame) | Publica `diagnostics/run-293-2026-11-26-lookup-cache.txt` (+35.896 linhas, sem patch textual) e atualiza `README.md` (+22/-2): o estado do fork passa a "2026-11-26, 02:15 UTC". A linha do run diz 203 s na área nova, personagem parado, `VICTORY_LOOKUP_CACHE=alt` sozinho; ajuste sobre 26 leituras de frame-log de 2.000 draws e mais: frame = 84 ms + 13,6 ms por 1.000 draws, 14 ms mais curto com o cache (dispersão de 17 ms, ou seja ±7), e o tempo das threads de gravação cai 57 ms por frame; no bin melhor preenchido (7.000-8.000 draws): 194,5 → 175,8 ms. |
| `53c743d7` | fxceless | diagnostics: run 294 (the light refresh in turn: copies fall, the frame does not) | Publica o run 294 e a linha no `diagnostics/README.md`. O texto: `VICTORY_LIGHT_REFRESH=alt` com `VICTORY_ABSENT_ONCE=1 VICTORY_ABSENT_MS=told` e o cache ligado; os bits respondem ~92 % das perguntas de memória de view (183-212 mil por bits contra 16 mil pelo lock em 20 s), as idas aos guards caem 2.480 por frame, "copies" cai 13,7 ms e "preparing" 10,8 ms — mas o frame não acompanha (+7 ms ±5). Sem ganho na cena; ficou desligado. |
| `18ea6447` | fxceless | diagnostics: run 295 (black surfaces were textures with no room on the card; more room, textured) | Run 295: 153 s com `VICTORY_CARD_WINDOWS_MB=512` em vez de 2048, testando por que superfícies eram pretas. O frame shot do run 292 mostrava a cor base do G-buffer zerada em 37 % da tela com a cadeia de luz intacta; o log dizia que texturas preenchidas pelo título iam a 2 GiB e além só enquanto a placa estava abaixo de nove décimos do orçamento. |
| `895a23bc` | fxceless | The heap's buffer-view pass can look only at marked slots, and a list can reuse a pipeline's keep and a repeated root view copy | O commit de código da faixa (+761/-5, 5 arquivos). `crates/victory-gpu/src/heap_pass.rs` novo (+438): a passagem de buffer-views do heap deixa de reler todos os slots a cada 16 passagens (84.027 slots, 178 ms no run 286) e passa a olhar só slot cuja view foi escrita, cuja entrada de tabela foi liberada (`descriptor::forget` → `note_forgotten`, com teto `FORGOTTEN_MOST = 1 << 20`), cujo registro em memória do título difere do último visto (comparação de 12 bytes em ordem de slot), que não achou cópia do host na última olhada (a cada 16) ou todos a cada `FULL_EVERY`; o span de cada cópia só é refeito quando um slot dela vai. `host.rs` (+84) ganha `first_bind` e `fill_scratch` para a lista reaproveitar pipeline e cópia de root view, com `bound`/`filled` como estado. Tudo atrás de `VICTORY_HEAP_PASS_MARKED` / `VICTORY_LIST_REUSE`; sem o switch o caminho é o de hoje. Origem declarada: snapshot de darkworldhm, que carrega o PR #523 do fork; quatro testes unitários passam; compilado mas **não rodado em jogo**. |
| `be8da788` | fxceless | diagnostics: run 296 (windows on the card at 1024 MB, closed early) | Run 296: janelas na placa a 1.024 MB, encerrado cedo. |
| `ad542076` | fxceless | diagnostics: run 297 (four submit-thread switches in turn together: its work falls, the frame does not) | Run 297: quatro switches de thread de submissão em conjunto; o trabalho da thread cai, o frame não. |
| `97f8dbcc` | fxceless | CI: the lints the checks deny are fixed, and run logs under diagnostics/ get a rule of their own | Conserta onze avisos de clippy que reprovavam o primeiro passo de todo push (cinco `is_multiple_of`, `contains_key` seguido de `insert`, `sort_by` por chave invertida virando `sort_unstable_by_key(Reverse(..))`, `if` colapsável, função com oito argumentos com `#[allow]`, função só do build de diagnóstico, campo de teste nunca lido) em `draw.rs`, `fpsbox.rs`, `queue.rs`, `resource.rs`, `shadow.rs`, `watch.rs`, `indirect.rs`, `pipeline/library.rs`, `heap_pass.rs`. E cria regra para logs de execução: `.txt` até 32 MiB sem dado binário ou `.txt.gz` até 16 MiB, direto sob `diagnostics/`, em `xtask/src/policy.rs` (+66) e `docs/asset-policy.md` (+14), com testes. |
| `0c7807ac` | fxceless | Tests: three that failed on a machine with no card, or by the order tests ran in, hold everywhere | Só testes (+32/-4). (1) O teste de orçamento do clamp indireto passa a afirmar **quantos** draws cabem, não **quais** — a ordem é do escalonador da placa (no adaptador de software dois outros ficaram em cinco de sete execuções); `Host::software()` novo, `#[cfg(test)]`, lê `DXGI_ADAPTER_FLAG_SOFTWARE`. (2) O teste de depth bounds é pulado no software, que declara o recurso e deixa todo pixel passar (255 onde a placa dá 0). (3) O teste de write-watch deixa de exigir journal vazio para alocação nova: o journal é do processo e guarda endereços para sempre, então uma alocação que caiu onde outra liberou falhava às vezes; agora exige que nada seja novo na primeira pergunta e só a página escrita na segunda. |
| `bc6ee68e` | fxceless | Lookup cache: each table change is counted on the right side of the change (fork pull request #1245, by hand) | Reaplicado à mão o PR #1245 do fork: cada mudança de tabela do cache de lookup passa a ser contada no lado certo da mudança (antes/depois), em vez de no lado oposto (+99/-22 num arquivo). |

## Pull requests abertos (5)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 1249 | NVIDIA compatibility: Wave64 control flow and lossless d16 buffer accesses | Integração grande (19 arquivos, +1932/-43): análise de região de laço divergente wave64, sincronização de LDS, opt-in explícito para execução aproximada de meia-wave, acessos d16 lossless com view de descritor unsigned (`VICTORY_SHADER_D16_BITS=1`) e uma variante de PSO R11 que reproduz o erro 613 do D3D12 (declarada já superada por `a35c5573`). Validação em RTX 3070, driver 610.88. | Não é D-xx direto. Encosta em `D-12` (loads escalares agrupados de 4 dwords leem além da view, água preta) pela parte de d16 — o defeito é outro. |
| 1250 | Count a loose window once against the GPU memory budget | Uma janela de 4 MiB num teto de 4 MiB contava como duas recusas, porque a tentativa de upload heap e o fallback de buffer staged pediam espaço cada um. Reserva os bytes uma vez entre as duas tentativas, devolve se nenhuma der certo, checa o teto antes de pedir orçamento ao DXGI e, uma vez falhada a sonda de mapeamento, manda as janelas seguintes direto ao caminho staged (`shadow.rs` +127). Testado em RX 7900 XT com RADV 26.3.1, Wine e vkd3d-proton. | Não é D-xx; é contabilidade de orçamento. `D-05` (regra de staleness anulando slots vizinhos) e `D-08` (índice de view além do fim do heap) são outros defeitos de memória. |
| 1251 | Shader tests: reject device loss before dispatch readback | `Host::dispatch` podia alcançar o readback com o device removido: a checagem está dentro de `while fence.GetCompletedValue() < 1`, e um device removido devolve `UINT64_MAX`, então o laço sai sem checar. Passa a verificar o estado do device depois da espera, antes de mapear saída (`emit/host.rs` +84). Regressão remove um device WARP filho e checa fence nunca sinalizada. | Não. Defeito de harness de teste, não de runtime. |
| 1254 | config: distinguish a non-file VICTORY_CONFIG from a missing one | `load()` dizia "does not exist" para caminho que existe mas não é arquivo (por exemplo um diretório); passa a dizer "is not a file", com `locate_config` testável e quatro testes. +58/-11 | Não. |
| 1255 | The driver watch counts warnings and errors only, not the Auto HDR notice | `drvwatch` assina todo evento da fonte `Display`, inclusive os informativos; o 4125 ("Good news… has been enhanced") do Auto HDR fazia o watch logar erro de driver ~1 s depois de cada início e desligar as janelas de draw pelo resto da execução, então `VICTORY_INDIRECT_WINDOW=1` nunca valia ali. A consulta passa a pedir só níveis 1-3 (critical/error/warning). Parte 1 de `#1253`. +14/-6 | Não é D-xx. É a explicação de por que um switch documentado não tinha efeito — vizinho de `D-14` (alegação de ganho comparando cenas diferentes) só no tema "medição enganosa". |

## Pull requests mesclados (0)

Nenhum PR com `mergedAt` posterior a 2026-11-26T02:11:35Z. O maior número mesclado devolvido é 1014,
abaixo de `last_pr_number` 1248.

## Cruzamento com o catálogo

- Corrige D-xx? Nenhum dos 17 itens. `D-16` ("perdas de device sem causa identificada, runs 204-279 e
  mundo aberto") continua aberto: o `README.md` atualizado em `bcce702f` mantém a linha de estabilidade
  dizendo que a placa é perdida entre 30 e 50 minutos (0x887A0005, runs 277 e 279) e que a causa não é
  conhecida. Os runs 293-297 desta hora não mudam isso.
- Abre defeito novo? Não evidente. `895a23bc` é o candidato: muda o que a passagem de buffer-views
  revisita, mas atrás de switch desligado e com quatro testes unitários; o próprio corpo diz "compilado
  aqui; não rodado em jogo". Enquanto o switch não for ligado por padrão, o risco fica contido.
- `D-14` ("alegação de ganho de 6,0 → 8,9 fps comparava cenas diferentes") tem tratamento correto agora:
  o run 293 mede em contagem de draws igual, por ajuste sobre 26 leituras, e publica a dispersão (±7 ms)
  em vez de um número único.

## Confiança

Verificado por diff: os nove commits foram lidos em `gh api .../commits/<sha>`; o de código
(`895a23bc`) arquivo a arquivo, os de diagnóstico pela linha do `diagnostics/README.md` e pelo cabeçalho
do log. Os cinco runs têm arquivos de 25 a 37 mil linhas — o conteúdo integral NÃO foi lido; o que está
acima é o resumo que o próprio repositório publica no README de diagnóstico.

Não verificado: nenhuma das medições foi reproduzida aqui. `895a23bc` não foi executado em jogo pelo
autor. Os PRs abertos não foram executados.
