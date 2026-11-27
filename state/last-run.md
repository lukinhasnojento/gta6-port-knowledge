# Execução de 2026-11-27 23:00

## Data e hora

Coleta de 2026-11-27T23:05–23:20 UTC real — ~20:05–20:20 no relógio do GitHub, que segue cerca de 3 h
atrás e é o relógio em que `last_run` é gravado. Diretório de documentos: `docs/updates/2026-11-27/23/`
(UTC). Estado lido: `baselined: true`, `runs: 20` (gravado como 21); 14 entradas em `repos` no início e no
fim, nenhuma removida, nenhuma acrescentada.

## Sanidade do estado (passo 1)

`gh api repos/<o>/<r> --jq .default_branch` foi pedido nos 14. **Dois continuam sem resposta**:
`Belobug/VIctory-Belobug-AMD` e `avatarmanz/VIctory-7900XTX` devolvem HTTP 404 (terceira hora seguida),
também em `users/<login>`; a `default_branch` deles não pôde ser reconferida, os cursores ficaram parados
e a triagem deles é NAO VERIFICADO. Detalhe em `docs/updates/2026-11-27/23/_falhas.md`.

Nos outros 12, `default_branch` idêntico ao gravado, inclusive `master` (`holdmysocks/SixNG`). Nenhum
`last_commit` nulo — não houve baseline nesta execução. Nenhum `compare` devolveu `diverged`. Contagem de
`repos` igual no início e no fim (14).

## Triagem (passo 2)

Os 14 foram triados (12 com medição completa); 4 entraram no deep dive. Arquivo:
`docs/updates/2026-11-27/23/_triagem.md`.

| Repositório | Commits atrás | Abertos acima do cursor | Mesclados na janela | Sinal | Deep dive |
|---|---|---|---|---|---|
| `brcly/TuxBox` | 21 | 1 (3 no total) | 4 | SIM (critério 2 e 1; `xmem`/`xshader`, D-17) | sim |
| `holdmysocks/SixNG` | 9 | 1 (4 no total) | 9 | SIM (critério 2; culling, pele, cabelo, MSAA) | sim |
| `R0Y00/Xbox2Windows` | 5 | 5 (≥100 no total) | 5 | SIM (critério 2; `startup:`, `f16_cvt_norm`, `s_bitreplicate`) | sim |
| `DexNom/VIctory` | 1 | 14 (≥100 no total) | 0 | SIM (critério 1/2; 9 restores de `570a941`, D-10/11/12 abertos) | sim |
| `Marvisak/VIctory` | 0 | 67 (≥100 no total) | 0 | SIM (critério 3; lote `Shaders:`/`GPU:`) | não |
| `ivall/xcl` | 0 | 22 (≥100 no total) | 0 | SIM (critério 3; bibliotecas nativas de pipeline) | não |
| `fxceless/victory-fork` | 0 | 2 (≥100 no total) | 0 | SIM (critério 3; dois backports de Vice-Tay) | não |
| `omonkeyz/xbox-emulator` | 0 | 1 (25 no total) | 0 | SIM fraco (critério 3; `#395`) | não |
| `uiliannn/VItoriaBr` | 0 | 1 (75 no total) | 0 | SIM fraco (critério 3; `#132`) | não |
| `californiaAlmonds/OpenScarlett` | 0 | 0 (1 no total) | 0 | NAO | não |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 | 0 | NAO | não |
| `stellarr-studio/kelvaroq` | 0 | 0 | 0 | NAO | não |
| `Belobug/VIctory-Belobug-AMD` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (404) | bloqueado |
| `avatarmanz/VIctory-7900XTX` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (404) | bloqueado |

Desempate: o critério 2 selecionou os quatro repositórios com commit na janela e ocupou as quatro vagas.
`Marvisak` (67 PRs novos) e `xcl` (22) perdem a vaga porque o critério 3 vem depois do 2. Nenhum
repositório passou de 200 commits de atraso, então o critério 4 não foi acionado.

## Documentos escritos (passo 3)

4 documentos em `docs/updates/2026-11-27/23/`: `TuxBox.md`, `SixNG.md`, `Xbox2Windows.md`,
`VIctory-DexNom.md`, mais `_triagem.md` e `_falhas.md`. **36 commits documentados de 36** atrás dos
cursores (21 + 9 + 5 + 1), **18 PRs mesclados** (9 + 5 + 4) e **20 PRs abertos** lidos (1 + 1 + 5 + 14 — o
total de 115 abertos novos dos 14 está na triagem). Nenhum repositório chegou ao teto de 40 commits.

Achados principais:

- `DexNom/VIctory` (`4ad86927`, só README): a fila de 14 PRs abertos é o efeito do publish `570a941`
  (2026-11-25T12:18:03Z, 170 arquivos, pai `940d32f6`). Nove restauram commits que ele derrubou
  (`44ce067`, `92f2c60`, `7d04943`, `78c1267`, `6f36cdf`, `e6fe8ec`, `e45bacd`, `bbdc3ca`, `13185f5`),
  com `cherry-pick -x` e autoria preservada; cinco são correções novas com teste de hardware (`#135`,
  `#137`, `#139`, `#140`, e o diagnóstico `#136`). **Nenhum dos 14 reaplica D-10 (`subresource_len`),
  D-11 (escrita atômica com `.tmp` fixo), D-12 (overflow do `clwb_loop`) nem D-15** — os quatro seguem
  abertos por reversão. `#137` promete listar `emit/trips.rs` em `translation_inputs!` (classe de chave de
  cache), afirmação do corpo não conferida no diff.
- `brcly/TuxBox`: 21 commits = 12 de conteúdo + 4 merges de PR (`#258`, `#324`, `#322`, `#249`) + 5 merges
  de `main` em ramo. `2f296416` tira `_shadow`/`_backing` do `ImportedHeap` (cuja ordem de drop era
  comentada como load-bearing) e prende os dois no `ID3D12Heap` via `SetPrivateDataInterface`, com o
  comentário nomeando o `vkd3d-proton` destruindo `resource->heap` depois do `private_store` — mesma
  janela de uso-após-liberar de D-17; `resolve_backings` passa a devolver `ERROR_INVALID_ADDRESS` dentro
  de abertura do XMem (área de D-22). `69106310` troca `prove()` por `Proof { votes, ballots }` e passa a
  contar/primeiro-bit de máscara com `WaveActiveBallot`, mais `s_andn2_wrexec_b32` escrevendo EXEC e
  destino e offset de buffer como um registrador; `60f7cfad` repete a tradução com `without_guards`
  quando o dispatcher por blocos perde o voto de guarda; `f4772cdd` admite `ds_swizzle_b32` na prova de
  máscara cheia. `07db0df9` libera com `MEM_RESET` as páginas de rascunho acima do snapshot (`Contents::store`),
  com teste de resident set. `67508417` dá identidade à requisição de compilação (`CS <hash8>
  request=<digest> bytes=<n>`) e um trace opt-in (`TUXBOX_TRACE_PIPELINE_REQUESTS=1`).
- `holdmysocks/SixNG`: 9 squash, 1:1 com 9 merges. Export de cabelo assenta as guias contra o corpo
  (`a31b51a2`), mapas de oceano na grade de água (`89c9fa07`), culling de decalques e malhas see-through
  que nunca divide um draw (`23964175`), detalhe de pele com LOD isotrópico dentro do tile (`b2effe97`),
  MSAA configurável no Metal 4 (`d2607d4b`), dois consertos de UI de livery (`0c0e2524`, `16d77ec9`) e o
  slider de escala da cabeça parando em −0,5 (`ede2c253`). **Achado verificável:** o switch de detalhe de
  pele (`e607e6d3`, bit 8 de `mode.y`) não é lido pelo renderizador Metal 4 — lido
  `render3d_metal4.rs` em `ede2c253`, as duas ocorrências de `skin_detail` são campos de material
  (`None`), e o arquivo tem shader próprio.
- `R0Y00/Xbox2Windows`: 5 squash, 1:1 com 5 merges. `6057071b` acrescenta à linha `x2w-budget` o commit do
  sistema (`commit=usado/limite avail=livre`), o limite/pico do job e `sections/slabs/slots`, e imprime a
  mesma linha nas três primeiras falhas 1455/8 do pool; `0f3e8248` separa `texture_unavailable` em
  `texture_no_resource`/`texture_unreadable`/`texture_over_64mib`; `968ddcdf` faz `cvt_norm_i16/u16_f16` em
  aritmética inteira com arredondamento RNE explícito (mesma regra do conserto de D-28); `454fc0ed` liga
  `X2W_LOW_ADDRESSES` (ASLR bottom-up desligado na criação do processo) e **falha em silêncio** se a lista
  de atributos não puder ser montada; `3c30dc74` implementa `s_bitreplicate_b64_b32` do SOP1 0x3b.

## Falhas e desvios

- **Dois repositórios inacessíveis (404 de conta, terceira hora).** Registrado em `_falhas.md`. Não
  avançaram cursor, não foram documentados, não saíram de `repos`.
- **A "tabela de pistas" citada no passo 2 continua inexistente** em `docs/automacao/ANALISE_HORARIA.md`.
  As áreas do sinal vieram das seções do `catalogo/defeitos.md` e do assunto de cada PR; o sinal dos
  repositórios sem commit é por título/corpo de PR, não por patch.
- **Contagem de commits × merges:** bate nos quatro documentados. `SixNG` e `Xbox2Windows` são squash de
  um pai (9=9, 5=5); `TuxBox` tem 4 merges de PR de dois pais dentro da faixa de 21; `DexNom` tem commit
  direto e zero merges.
- **PRs abertos documentados pelo título e pelo corpo, não pelo patch.** 20 corpos lidos (14 de `DexNom`,
  5 de `Xbox2Windows`, 1 de `SixNG`, 1 de `TuxBox` — este último só título); nenhum patch de PR aberto foi
  lido linha a linha.
- **Sem subagentes.** A ferramenta não existe neste ambiente; os quatro repositórios foram processados em
  sequência e o limite de 4 deep dives foi respeitado.
- **Rate limit:** 4.838 restantes no início, 4.921 depois do reset da janela, nenhum 403, nenhuma
  repetição em laço.
- **Escrita:** somente em `G:\gta6-port-knowledge`. Os 14 monitorados foram só leitura de API.
- **Arquivos de terceiros no diretório de trabalho** (`docs/analises/victory-fork.md` modificado e dois
  relatórios não rastreados em `docs/relatorios/`) ficaram fora do `git add` desta rotina e intactos.

## Estado gravado

`runs: 21`, `updated: 2026-11-27T23:20:00Z`, `last_error` com os dois 404 (terceira hora). Cursores que
avançaram:

| Repositório | `last_commit` | `last_pr_number` | `last_run` |
|---|---|---|---|
| `R0Y00/Xbox2Windows` | `3c30dc74` (head) | 3169 | 2026-11-27T20:04:37Z |
| `brcly/TuxBox` | `c05ce004` (head) | 325 | 2026-11-27T20:01:33Z |
| `holdmysocks/SixNG` | `ede2c253` (head) | 500 | 2026-11-27T20:04:52Z |
| `DexNom/VIctory` | `4ad86927` (head) | 148 | 2026-11-26T04:15:00Z (sem merge) |

`last_run` dos três com merge é o maior `merged_at` documentado, no relógio do GitHub — não o horário da
execução, para não esconder merge entre o fim da coleta e a próxima hora. Os outros 10 mantêm cursor de
commit e de merge. `backlog` com 7: os dois bloqueados e `Marvisak/VIctory`, `ivall/xcl`,
`fxceless/victory-fork`, `omonkeyz/xbox-emulator`, `uiliannn/VItoriaBr`; a ordem e os achados estão no
`backlog_note`.

## Push

Resultado registrado no fechamento desta execução (segundo commit na `public`, como nas horas
anteriores).
