# Execução de 2026-11-28 15:00

## Data e hora

Coleta de 2026-11-28T15:07Z a 15:26Z (UTC real; 12:07–12:26 local, UTC−3). Diretório de documentos:
`docs/updates/2026-11-28/15/`. Estado lido: `baselined: true`, `runs: 26` (gravado como 27), 14
entradas em `repos` no início e no fim, nenhuma removida, nenhuma acrescentada.

O relógio do GitHub está alinhado ao UTC real: o merge `#3159` do `xcl` (`14:48:56Z`), o `#3365` do
`Xbox2Windows` (`14:15:27Z`) e a leva de `TuxBox` das 15:06–15:07Z caíram dentro da janela. Os
`last_run` gravados são o maior `merged_at` **efetivamente coberto** de cada repositório, no relógio do
GitHub.

## Sanidade do estado (passo 1)

`gh api repos/<o>/<r> --jq .default_branch` foi pedido nos 14. **Dois continuam sem resposta**:
`Belobug/VIctory-Belobug-AMD` e `avatarmanz/VIctory-7900XTX` devolvem HTTP 404 (nona hora seguida);
`default_branch` não reconferida, cursores parados, triagem NAO VERIFICADO, detalhe em `_falhas.md`.

Nos outros 12, `default_branch` idêntico ao gravado, inclusive `a-publier` (`Belobug`, não lido) e
`master` (`holdmysocks/SixNG`). Nenhum `last_commit` nulo — não houve baseline. Nenhum `compare`
devolveu `diverged` (4 `ahead`, 8 `identical`). Contagem de `repos` igual no início e no fim (14).

## Triagem (passo 2)

Os 14 foram triados (12 com medição completa); 4 entraram no deep dive. Arquivo: `_triagem.md`.

| Repositório | Commits atrás | Abertos acima do cursor | Mesclados na janela | Sinal | Deep dive |
|---|---|---|---|---|---|
| `ivall/xcl` | 193 | 4 (máx. #3165) | 1 (#3159) | SIM (critério 2; `#3159` é a 2ª ocorrência de D-24) | sim |
| `brcly/TuxBox` | 92 | 0 (máx. #393) | 12 (11 novos) | SIM (critério 2: #395, #396, #398, #399, #400) | sim |
| `R0Y00/Xbox2Windows` | 72 | 4 (máx. #3371) | 1 (#3365) | SIM (critérios 2 e 3) | sim |
| `fxceless/victory-fork` | 0 (1 commit preso no cursor) | 0 | 0 | SIM (critério 2, por reversão) | sim |
| `Marvisak/VIctory` | 0 | 203 (máx. #3753) | 0 | SIM fraco (3) | não |
| `DexNom/VIctory` | 0 | 7 (máx. #155) | 0 | SIM fraco (3) | não |
| `uiliannn/VItoriaBr` | 0 | 3 (máx. #134) | 0 | SIM fraco (3) | não |
| `omonkeyz/xbox-emulator` | 0 | 1 (#395) | 0 | SIM fraco (3) | não |
| `californiaAlmonds/OpenScarlett` | 0 | 1 (#66) | 0 | NAO | não |
| `holdmysocks/SixNG` | 0 | 0 | 0 | NAO | não |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 | 0 | NAO | não |
| `stellarr-studio/kelvaroq` | 0 | 0 | 0 | NAO | não |
| `Belobug/VIctory-Belobug-AMD` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | 404 | bloqueado |
| `avatarmanz/VIctory-7900XTX` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | 404 | bloqueado |

**Critério 1 (defeito do catálogo por hash) não selecionou ninguém**: 358 hashes de commit e 14 hashes
de merge das quatro janelas foram conferidos por prefixo de 12 contra as 305 linhas de
`catalogo/defeitos.csv` (262 hashes distintos), sem interseção. **Critério 4** não acionou em ninguém:
`xcl` caiu de 232 para 193 e é o maior atraso.

## Documentos escritos (passo 3)

4 documentos em `docs/updates/2026-11-28/15/`: `xcl.md`, `TuxBox.md`, `Xbox2Windows.md`,
`victory-fork.md`, mais `_triagem.md` e `_falhas.md`. **94 commits documentados de 241 não documentados**
(`xcl` 40 de 168, `TuxBox` 40 de 59, `Xbox2Windows` 13 de 13, `victory-fork` 1 de 1), **13 merges
novos** (`TuxBox` 11, `xcl` 1, `Xbox2Windows` 1) e **8 PRs abertos** lidos por título e corpo.

Achados principais:

- **`xcl`**: quarenta commits de `bmjask` em três frentes ainda não ligadas a política — classes de
  alocação no `TilePool` (`c580dd31`, `e009ba2c`, `3b5bc7b8`), gerações nativas de PSO com receita de
  rebuild (`1b937474`, `dcfbb49c`, `9da465f1`, `11613fb6`, `c1be67ec`, `c1bdf10e`, `8eb35ad7`) e épocas
  de descriptor heap (`9d7bea0f`, `c524972c`, `a3b43590`, `00dfb305`) — mais a única frente com
  consequência visível: o estado esparso fora do lock, em que `86bad822` espera o fence com `STATE`
  liberado, `f5ce20f3` libera o gate durante a espera e `cfeb2817` **encerra o título** quando o upload
  não pode ser ordenado, porque a API de submissão é `void`. Cinco commits (`60a90b12`, `3caff9ef`,
  `bff80f9b`, `976faf72`, `9d8af31c`) são diagnóstico de `Wait64` para o `XCL_THREAD_DUMP_AT`, escritos a
  partir de um travamento real com `rec 0x619, sub 0x0@qNone` sem time-out. O merge `#3159`
  (`309c3ec5`, 3 arquivos, +34/−1) é a **segunda ocorrência de D-24**: `56237a2d` pôs `getDimensions`
  antes dos loads de `Texture2DMS`, a simplificação de CFG re-fundiu o bloco de predecessor único e
  restaurou o padrão que derruba o `amdxc64`; agora o alvo de bloco é marcado e não pode ser fundido.
- **`TuxBox`**: a pilha de ray tracing em memória do título entrou junta às 14:41:16Z (`#320`
  `561b8059`, `#375` `c800b907`, `#376` `470b749a`, `#394` `dd432e6f`) — BLAS em endereço cru,
  índices no formato 53 decodificados, posições `R32G32_FLOAT` de 8 bytes com stride `0x8008` lido pelos
  14 bits, e a rehidratação das estruturas que o `XboxBvhBuilder` recebe prontas. Às 15:06–15:07Z
  entrou a leva de runtime: `#395` (`ff2c9f41`) evicção de tabelas em um passe com `pins:
  Arc<AtomicU32>`, `#398` (`05c691da`) label que não é mais liberado em 5 s sem olhar o título, `#399`
  (`ba02a79c`) heap de 2.097.152 descritores em heaps CPU menores (o `0x8007000e` que falha no UHD),
  `#400` (`0f845250`) o `C4456` do MSVC em `/W4 /WX`, `#396` (`ddbbc1c8`) aresta de volta do laço
  decidida pela wave. Fora do ray tracing, `a5891254` impede o *paired-wave64* de reconstruir VCC em
  `v_cmpx` e apagar os bits das lanes fora da comparação; `d828a00d` faz todo load de raw buffer
  testar EXEC.
- **`victory-fork`**: o commit preso `368967f8` (+240/−13.815 em 40 arquivos) é um reset para o
  `VIctory-NVIDIA 959dcaf` com os consertos da água. O diff **apaga** `pipeline/sampler_order.rs` — o
  conserto medido às 13:00 (406→211→0 pipelines acima de 1 s) — e todo o guia de ray tracing (`guide.rs`,
  `guide/core.rs`, `livery.rs`, `raycounts.rs`, `shading.rs`, `rays.rs`, `rays_tests.rs`). A classe de
  D-02 volta a valer **por remoção**. Ray tracing fica ligado por padrão pela pilha antiga
  (`VICTORY_BVH_BUILDER=1` em `play.rs:75`) e `VICTORY_DYNAMIC_WAIT` sai do `SETTINGS`.
- **`Xbox2Windows`**: a hora é integração (três merges internos e três "sync" das pilhas nativa e de
  mesh). O trabalho novo com consequência é `7a163fc5`, que amarra a aposentadoria dos *sampler heaps*
  traduzidos ao allocator da lista em vez de um mapa global por `ID3D12CommandList*`, e `#3365`
  (`f5a6f6ba`, 3 arquivos, +103/−3), que normaliza a realocação do descritor Mesh57 de `0x300` bytes na
  chave do perfil preservando as distâncias entre os ponteiros internos. A fila de commits do
  repositório zerou.

## Falhas e desvios

- **Dois repositórios inacessíveis (404, nona hora).** Não avançaram cursor, não foram documentados,
  não saíram de `repos`.
- **Cursor preso em `victory-fork`.** O ciclo de 13:00 gravou `commit_cursor = 368967f8`, que é o
  **próprio** commit pendente; `compare(368967f8...368967f8)` devolve zero e o commit nunca seria
  documentado. Foi documentado nesta execução e o cursor avançou. Isso consumiu o **4º slot** de deep
  dive, que o critério de mérito não daria a esse repositório. Desvio registrado em `_falhas.md`, com a
  correção sugerida para o procedimento.
- **Documento publicado com hash inexistente.** O ciclo de 14:00 registra o merge do `TuxBox` `#377`
  como `3eb76bd5`; `commits/3eb76bd5` devolve 404 e o merge real é `3eb76bd9`
  (`merged_at 2026-11-28T09:22:22Z`). Registrado em `TuxBox.md` e em `_falhas.md`; o documento de 14:00
  não foi reescrito.
- **Ordem do `compare` instável.** As quatro janelas foram escolhidas por hash (primeiro commit sem
  documento anterior) e o novo `commit_cursor` é o 41º dessa ordem — não há garantia topológica de que
  seja ancestral de tudo o que ficou para trás.
- **A "tabela de pistas" do passo 2 continua inexistente** em `docs/automacao/ANALISE_HORARIA.md`. As
  áreas vieram das seções D do catálogo e do corpo dos PRs.
- **`Marvisak/VIctory`:** `gh pr list --limit 400` devolveu 400 PRs com 203 acima do cursor; a contagem
  é um piso, não o total.
- **Nenhum subagente disponível** neste ambiente; os quatro repositórios foram processados em
  sequência.
- **Rate limit:** nenhum 403 nesta coleta.
- **Escrita:** somente em `G:\gta6-port-knowledge`. Os 14 monitorados foram só leitura de API.

## Estado gravado

`runs: 27`, `updated: 2026-11-28T15:26:00Z`, `last_error` com os dois 404 (nona hora). Cursores que
avançaram:

| Repositório | `last_commit` | `commit_cursor` | `head_at_run` | `last_pr_number` | `last_run` |
|---|---|---|---|---|---|
| `ivall/xcl` | `8eb35ad7` | `97583afd` | `309c3ec5` | 3165 | 2026-11-28T14:48:56Z |
| `brcly/TuxBox` | `b9972eba` | `99eef7ab` | `ff2c9f41` | 398 | 2026-11-28T15:07:14Z |
| `R0Y00/Xbox2Windows` | `f5a6f6ba` | — | — | 3371 | 2026-11-28T14:15:27Z |
| `fxceless/victory-fork` | `368967f8` | — | — | 1513 | 2026-11-28T01:48:22Z |

`Xbox2Windows` e `victory-fork` perderam `commit_cursor`/`head_at_run` porque a fila zerou. Os outros 10
mantêm cursor de commit e de merge. `backlog` com 8 entradas (os 2 com trabalho restante, os 4 com sinal
fraco e os 2 bloqueados); `SixNG`, `OpenScarlett`, `GTA6-Asset-Info` e `kelvaroq` saíram. A ordem, os
achados e a prioridade de 16:00 estão no `backlog_note`.

## Push

Executado às 15:2xZ e registrado depois das duas publicações.

- Commit na `public`: `cbfb0ef` — `docs(updates): 2026-11-28 15:00` (8 arquivos: os 4 documentos,
  `_triagem.md`, `_falhas.md`, `docs/updates/INDEX.md` e `state/watch.json`; 560 inserções, 23
  remoções).
- `git push origin public:main`: **sucesso** — `f115623..cbfb0ef  public -> main`.
- `git switch main` + `git merge --no-ff public`: **sem conflito nesta hora** (ao contrário de 14:00) —
  merge `5db6f17` — `merge updates 2026-11-28 15:00`.
- `git push private main`: **sucesso** — `eb245d7..5db6f17  main -> main`.
- Este fechamento é um segundo commit na `public`, depois do merge. Para não criar um segundo merge na
  execução, a `main` o recebe por cherry-pick, como nos ciclos anteriores.
- Nenhum `git reset --hard`, nenhum merge de `main` na `public`.
- Ficaram fora do commit, intactos: `docs/analises/victory-fork.md` (modificado) e os dois relatórios
  não rastreados em `docs/relatorios/`, que não são desta rotina.
