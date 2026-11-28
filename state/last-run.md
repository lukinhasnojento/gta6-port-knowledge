# Execução de 2026-11-28 14:00

## Data e hora

Coleta de 2026-11-28T14:07Z a 14:22Z (UTC real). Diretório de documentos:
`docs/updates/2026-11-28/14/`. Estado lido: `baselined: true`, `runs: 25` (gravado como 26), 14
entradas em `repos` no início e no fim, nenhuma removida, nenhuma acrescentada.

O relógio do GitHub estava alinhado ao UTC real: o merge `#3157` do `xcl` (`14:06:59Z`) e o `#3094` do
`Xbox2Windows` (`14:07:49Z`) caíram dentro da janela. Os `last_run` gravados são, como sempre, o maior
`merged_at` **efetivamente coberto** de cada repositório, no relógio do GitHub.

## Sanidade do estado (passo 1)

`gh api repos/<o>/<r> --jq .default_branch` foi pedido nos 14. **Dois continuam sem resposta**:
`Belobug/VIctory-Belobug-AMD` e `avatarmanz/VIctory-7900XTX` devolvem HTTP 404 (oitava hora seguida);
`default_branch` não reconferida, cursores parados, triagem NAO VERIFICADO, detalhe em `_falhas.md`.

Nos outros 12, `default_branch` idêntico ao gravado, inclusive `master` (`holdmysocks/SixNG`). Nenhum
`last_commit` nulo — não houve baseline. Nenhum `compare` devolveu `diverged` (4 `ahead`, 8
`identical`). Contagem de `repos` igual no início e no fim (14).

## Triagem (passo 2)

Os 14 foram triados (12 com medição completa); 4 entraram no deep dive. Arquivo: `_triagem.md`.

| Repositório | Commits atrás | Abertos acima do cursor | Mesclados na janela | Sinal | Deep dive |
|---|---|---|---|---|---|
| `ivall/xcl` | 232 | 2 (máx. #3158) | 7 (3 novos) | SIM (critérios 2 e 4) | sim |
| `brcly/TuxBox` | 99 | 1 (#398) | 30 (24 novos) | SIM (critérios 2 e 3) | sim |
| `R0Y00/Xbox2Windows` | 84 | 6 (máx. #3364) | 38 (33 novos) | SIM (critérios 2 e 3) | sim |
| `holdmysocks/SixNG` | 8 | 0 | 10 (7 novos) | SIM (critério 2) | sim |
| `Marvisak/VIctory` | 0 | 195 (300 lidos, máx. #3743) | 0 | SIM fraco (3) | não |
| `DexNom/VIctory` | 0 | 5 (máx. #153) | 0 | SIM fraco (3) | não |
| `uiliannn/VItoriaBr` | 0 | 3 (máx. #134) | 0 | SIM fraco (3) | não |
| `omonkeyz/xbox-emulator` | 0 | 1 (#395) | 0 | SIM fraco (3) | não |
| `fxceless/victory-fork` | 1 | 0 | 0 | NAO | não |
| `californiaAlmonds/OpenScarlett` | 0 | 1 (#66) | 0 | NAO | não |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 | 0 | NAO | não |
| `stellarr-studio/kelvaroq` | 0 | 0 | 0 | NAO | não |
| `Belobug/VIctory-Belobug-AMD` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | 404 | bloqueado |
| `avatarmanz/VIctory-7900XTX` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | 404 | bloqueado |

**Critério 1 (defeito do catálogo) não selecionou ninguém**: os hashes de todos os commits e merges
da janela (423 no total) foram conferidos contra as 305 linhas de `catalogo/defeitos.csv` (262 hashes
distintos) sem interseção. O critério 4 só acionou em `xcl` (232 > 200).

## Documentos escritos (passo 3)

4 documentos em `docs/updates/2026-11-28/14/`: `xcl.md`, `TuxBox.md`, `Xbox2Windows.md`,
`SixNG.md`, mais `_triagem.md` e `_falhas.md`. **127 commits documentados de 395 não documentados**
(211 + 85 + 71 + 7 + o commit pendente do `victory-fork`), **71 merges novos** (33 + 24 + 7 + 3) e
**9 PRs abertos** lidos por título e corpo.

Achados principais:

- **`xcl`**: a série de identidade de cache de pipeline — samplers estáticos (`ba44b52c`), `NodeMask`
  em memória (`fbcd70e7`) e em disco (`efdebc82`), dispositivo (`5cd0c4ae`), root signature
  (`65704ff1`), `HEAP_FLAT` na chave (`2f523330`, com separação só para compute em `e8f7164b`) — e a
  de estado esparso fora do lock: criação de recurso reservado (`a97dbe9b`), unmap adiado até as
  marcas de todas as filas (`c22c0ac2`) com o `Signal` fora do `PAGES` (`f043b319`), ring de upload
  que desiste em vez de repetir para sempre (`711ff3ef`), allocator que aborta quando o fence não
  chega (`79cf5d9b`). O merge `#3149` (`b937fbab`) é o conserto com sintoma: o copy-back descartava a
  escrita da GPU num chunk que a CPU também alterou, e os três testes e2e do título falhavam com
  `post-Wait32/64 write failed (0x0)` (3 de 5 na `main`, 5 de 5 no head, segundo o corpo).
- **`TuxBox`**: o decodificador BCPack no `xcrdapi` (`#345`, 39 arquivos, rANS + tabelas do
  `bcpack-rs 2d6333b0`) e quatro consertos com sintoma no diff: `#362` (clear de UAV com CPU handle de
  outra arena), `#367` (`SEC_RESERVE` e `VirtualQuery` no wineserver), `#377` (matriz de páginas
  preenchida depois do enqueue), `#386` (carga raw fora de EXEC). `#380` para de reconstruir VCC em
  `v_cmpx` no GFX10, `#357` divide dispatches de mais de 65.535 grupos com `group base` em
  `b0, space32766` e `#372`/`44ee2537` faz laços que encontram o grupo em barreiras iterarem como
  grupo. O **T2 do roadmap fecha em 372/372** instruções usadas com check de GPU (`df4f27c3`).
- **`Xbox2Windows`**: a integração da pilha nativa — `#3091` (`221ed27b`, compilador de mesh/pixel e
  prewarm de compute), `#3241` (merge-only de `main`), `#3094` (`06cfb07f`, runtime que consome os
  requests e submete mesh fixo) — com sete commits de sincronização entre eles; e o fechamento de
  recusas (`s_lshl/s_lshr/s_ashr_b64`, 12 `*_saveexec_*`/`*_wrexec_*`, `v_cvt_norm_*` binário16,
  clamp VOP3P f16, `v_cvt_pknorm_*`, D16 `bufhalf`, SRV 1D/2D-array/3D). Três consertos reais:
  `#3285` (`17f112b2`, teto do `fract`), `#3307` (`ca165ed9`, UB em `unpin_slot`), `#3293`
  (`bbf4687b`, BC2/BC6H em branco e BC1 ajustado).
- **`SixNG`**: veículos e biblioteca — `#523` (`2a744586`) nomeia nove registros pelo modelo do
  chassi, `#541` (`5ef1224e`) funde `tankercar`/`freight` no registro vazio do modelo, `#496`
  (`076d1986`) lê a zona do adesivo do vidro em `_COLOR0` byte 0, `#552` (`11004e6a`) exporta o
  personagem como um GLB com o traje e a forma mostrados e `#553` (`add169df`) rotula slot padrão do
  motor em vez de "missing". `#554` sobe para 0.15.0.

## Falhas e desvios

- **Dois repositórios inacessíveis (404, oitava hora).** Não avançaram cursor, não foram documentados,
  não saíram de `repos`.
- **Ordem do `compare` instável.** O `commit_cursor` do `xcl` (`e5266254`) apareceu no índice 2 da
  página, não no 40. As quatro janelas foram escolhidas por hash (primeiro commit sem documento
  anterior) e o novo cursor é o 41º dessa ordem atual — não há garantia topológica de que ele seja
  ancestral de tudo o que ficou para trás.
- **`last_run` antigo em três repositórios.** `xcl` (04:12:54Z), `TuxBox` (01:16:56Z) e
  `Xbox2Windows` (05:24:33Z) tinham cursores de merge anteriores a merges já documentados nos ciclos
  de 12:00/13:00. A deduplicação foi por hash de merge contra todo `docs/updates/**/*.md`; os merges
  reconhecidos não foram reescritos.
- **A "tabela de pistas" do passo 2 continua inexistente** em `docs/automacao/ANALISE_HORARIA.md`.
  As áreas vieram das seções D do catálogo e do corpo dos PRs.
- **`Xbox2Windows` andou depois da coleta:** head `06cfb07f` às 14:07Z, `f5a6f6ba` às 14:21Z. O
  `head_at_run` registra o novo; os commits que chegaram depois não foram documentados.
- **Nenhum subagente disponível** neste ambiente; os quatro repositórios foram processados em
  sequência.
- **Rate limit:** nenhum 403; a contabilidade ficou em 4.851 no fim da coleta.
- **Escrita:** somente em `G:\gta6-port-knowledge`. Os 14 monitorados foram só leitura de API.

## Estado gravado

`runs: 26`, `updated: 2026-11-28T14:22:00Z`, `last_error` com os dois 404 (oitava hora). Cursores que
avançaram:

| Repositório | `last_commit` | `commit_cursor` | `head_at_run` | `last_pr_number` | `last_run` |
|---|---|---|---|---|---|
| `ivall/xcl` | `e9389076` | `be9d4057` | `b937fbab` | 3158 | 2026-11-28T14:07:15Z |
| `brcly/TuxBox` | `94cabfa8` | `1f44aec1` | `6f89916a` | 398 | 2026-11-28T11:19:37Z |
| `R0Y00/Xbox2Windows` | `1f07b2c1` | `cdce92da` | `f5a6f6ba` | 3364 | 2026-11-28T14:07:49Z |
| `holdmysocks/SixNG` | `67fc2742` | — | — | 541 | 2026-11-28T13:17:57Z |

Os outros 10 mantêm cursor de commit e de merge. `backlog` com 10 entradas (os 3 com trabalho
restante, os 5 com sinal fraco ou 1 commit pendente e os 2 bloqueados; `SixNG` zerou e sai); a ordem,
os achados e a prioridade de 15:00 estão no `backlog_note`.

## Push

Pendente no momento em que este arquivo foi escrito; o resultado é registrado no fechamento, em
commit próprio na `public` e cherry-pick na `main` (para não criar um segundo merge na execução).
