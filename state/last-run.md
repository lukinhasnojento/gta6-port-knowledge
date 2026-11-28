# Execução de 2026-11-28 13:00

## Data e hora

Coleta de 2026-11-28T13:07–13:52Z (UTC real). Diretório de documentos: `docs/updates/2026-11-28/13/`.
Estado lido: `baselined: true`, `runs: 24` (gravado como 25); 14 entradas em `repos` no início e no
fim, nenhuma removida, nenhuma acrescentada.

Nesta coleta o relógio do GitHub está alinhado ao UTC real: o merge `#541` do `SixNG`, de
`13:05:41Z`, e o commit `3b6201a4`, de `13:03:30Z`, caíram dentro da janela. Os `last_run` gravados
são, como sempre, o maior `merged_at` **documentado** de cada repositório.

## Sanidade do estado (passo 1)

`gh api repos/<o>/<r> --jq .default_branch` foi pedido nos 14. **Dois continuam sem resposta**:
`Belobug/VIctory-Belobug-AMD` e `avatarmanz/VIctory-7900XTX` devolvem HTTP 404 (sétima hora seguida);
`default_branch` não reconferida, cursores parados, triagem NAO VERIFICADO. Detalhe em
`docs/updates/2026-11-28/13/_falhas.md`.

Nos outros 12, `default_branch` idêntico ao gravado, inclusive `master` (`holdmysocks/SixNG`). Nenhum
`last_commit` nulo — não houve baseline. Nenhum `compare` devolveu `diverged` (5 `ahead`, 7
`identical`). Contagem de `repos` igual no início e no fim (14).

## Triagem (passo 2)

Os 14 foram triados (12 com medição completa); 4 entraram no deep dive. Arquivo:
`docs/updates/2026-11-28/13/_triagem.md`.

| Repositório | Commits atrás | Abertos acima do cursor | Mesclados na janela | Sinal | Deep dive |
|---|---|---|---|---|---|
| `ivall/xcl` | 338 | 62 (400 lidos, máx. #3155) | 19 | SIM (critérios 2 e 4) | sim |
| `brcly/TuxBox` | 148 | 14 (máx. #397) | 47 | SIM (critérios 2 e 3) | sim |
| `R0Y00/Xbox2Windows` | 86 | 41 (máx. #3354) | 71 | SIM (critérios 2 e 3) | sim |
| `fxceless/victory-fork` | 8 | 8 (máx. #1513) | 2 | SIM (critérios 2 e 3) | sim |
| `holdmysocks/SixNG` | 7 | 1 (#554) | 9 | SIM (critério 2) | não (limite) |
| `Marvisak/VIctory` | 0 | 187 (300 lidos, máx. #3734) | 0 | SIM fraco (critério 3) | não |
| `DexNom/VIctory` | 0 | 4 (máx. #152) | 0 | SIM fraco (critério 3) | não |
| `uiliannn/VItoriaBr` | 0 | 3 (máx. #134) | 0 | SIM fraco (critério 3) | não |
| `omonkeyz/xbox-emulator` | 0 | 1 (#395) | 0 | SIM fraco (critério 3) | não |
| `californiaAlmonds/OpenScarlett` | 0 | 1 (#66) | 0 | NAO | não |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 | 0 | NAO | não |
| `stellarr-studio/kelvaroq` | 0 | 0 | 0 | NAO | não |
| `Belobug/VIctory-Belobug-AMD` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (404) | bloqueado |
| `avatarmanz/VIctory-7900XTX` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (404) | bloqueado |

Desempate: cinco repositórios tinham sinal por correção e o limite é 4. Ficou fora o de menor janela,
`holdmysocks/SixNG` (7 commits, 9 merges), cujo cursor está no head `2b2277a5` e portanto não perde
nada por esperar. O critério 1 (defeito do catálogo) **não selecionou ninguém**: 570 hashes distintos
na janela dos quatro escolhidos foram conferidos contra as 305 linhas de `catalogo/defeitos.csv` (262
hashes distintos) sem interseção. O critério 4 só acionou em `xcl` (338 > 200).

## Documentos escritos (passo 3)

4 documentos em `docs/updates/2026-11-28/13/`: `xcl.md`, `TuxBox.md`, `Xbox2Windows.md`,
`victory-fork.md`, mais `_triagem.md` e `_falhas.md`. **128 commits documentados de 580** na janela
(338 + 148 + 86 + 8), **75 PRs mesclados** documentados (37 + 19 + 17 + 2) e **125 PRs abertos** lidos
por título e corpo.

Achados principais:

- **`xcl`**: a série D3D12.X de lock/ownership. Vinte dos 40 commits tiram uma chamada de driver de
  dentro de um lock global (fence, `CopyDescriptorsSimple`, `Signal`, `Reset`, `ExecuteCommandLists`)
  e a trocam por snapshot de dono forte + consulta fora do lock + revalidação de geração exata; o
  `queue_submit.rs` novo dá a cada fila um lease reentrante por geração. Quatro consertos
  independentes: `5ee15471` retém objetos submetidos quando a espera de fallback falha (em vez de
  liberar memória que a GPU ainda referencia), `15aa7f36` dobra `select(exec,novo,antigo)` só onde o
  valor é observado sob a mesma condição, `fb5e7d15` apaga o blob de driver em volume sem
  `FileDispositionInfoEx` e `8e3755a1` faz o journal de pipelines usados sobreviver à morte do worker.
- **`TuxBox`**: fecha a frente *two-lane* para waves de 64 do console (numeração dois a dois para
  derivadas, HLSL menor com `vN__[2]`, `OP_SEL` de `v_permlane*`, scan de 64 lanes antes excluído) e
  entrega o decodificador BCPack do DirectStorage: rANS do `silicon-rans`, 12 lanes, BC1 com 400
  chunks byte a byte contra `bcpack-rs 5bf5cc5`, e BC3/BC4/BC5/BC7 com tabelas de `2d6333b` e do
  `bcpack-tables.bin` do VIctory. `0e2e6ef3` impede que `vulkaninfo` e o Xwayland privado apaguem o
  cache de shader da NVIDIA; `94f3803b` tira `ACCESS_COMMON` da barreira de argumentos indiretos.
- **`Xbox2Windows`**: maior leva de lowerings do DXIL direto (26 dos 40) — `v_dot2_f32_f16`,
  `v_dot2c`, `v_pk_fmac_f16`, EXEC/VCC em SALU de 64 bits, atomics 2D (`swap/umin/and/or/xor`),
  `GLC/DLC/SLC`, derivadas implícitas no compute com validador 1.8, rounds de f16, `div_fixup`,
  `madmk/madak`, `rcp_iflag` — e cinco consertos de runtime: `4d7265c0` (MSVC), `a0b45d43` (caminhos
  resolvidos), `c827d8ba` (primeiro draw gráfico nativo no processo), `24eb18b9` (stderr do debugger
  no PowerShell 5.1) e `20ef1f72` (BC4_UNORM devolvia 79, o formato typeless).
- **`victory-fork`**: `9e907466` faz todos os samplers estáticos de um pipeline entrarem na tabela do
  driver numa ordem guardada, e mede 406 → 211 → 0 pipelines acima de 1 s; `de57b010` roda o guia de
  ray tracing por hardware numa placa pela primeira vez e acha um `device hung` por palavra de
  memória compartilhada nunca escrita; `6b3ce6ce` corrige o próprio número: ~590 de ~2.000 instâncias
  ficaram fora da cena, então os fps não são ganho.

## Falhas e desvios

- **Dois repositórios inacessíveis (404, sétima hora).** Não avançaram cursor, não foram documentados,
  não saíram de `repos`.
- **A "tabela de pistas" citada no passo 2 continua inexistente.** As áreas do critério 3 vieram das
  seções D do `catalogo/defeitos.md` e do corpo de commits e PRs.
- **Cursor não apontava para o primeiro commit não documentado.** Medi a sobreposição antes de
  escrever: em `TuxBox` os 19 primeiros da página do `compare` já estavam no documento das 12:00, e em
  `Xbox2Windows` os 5 primeiros. As janelas começam nesses índices (0, 19, 5) e os 128 commits
  documentados são todos novos.
- **`compare` do `xcl` truncado:** `ahead_by=338` com 250 commits na resposta. Os 88 restantes não vêm
  na chamada e estão registrados como lacuna de cobertura em `xcl.md`.
- **Heads mudaram durante a execução.** `xcl` passou de `e429936c` para `e13d6250` às 13:23–13:24Z
  (13 commits, quatro merges: #1809, #3102, #1873, #1908) e `victory-fork` ganhou `368967f8` às
  13:25Z. Nenhum foi documentado — chegaram depois da janela e o teto de 40 já estava consumido — e o
  `head_at_run` de cada um os registra. `SixNG` também se moveu (`67fc2742`, v0.15.0 #554, 13:17Z) e
  não foi documentado por limite.
- **Um `fetch` de commit falhou e foi refeito:** `4b01fde4` do `xcl`; o documento descreve o diff real.
- **Sem subagentes.** A ferramenta não existe neste ambiente; os quatro repositórios foram processados
  em sequência.
- **Rate limit:** 4.677 no início, 3.192 no fim, nenhum 403, nenhuma repetição em laço.
- **Escrita:** somente em `G:\gta6-port-knowledge`. Os 14 monitorados foram só leitura de API.
- **Arquivos de terceiros no diretório de trabalho** (`docs/analises/victory-fork.md` modificado e os
  dois relatórios não rastreados em `docs/relatorios/`) ficaram fora do `git add` e intactos.

## Estado gravado

`runs: 25`, `updated: 2026-11-28T13:30:00Z`, `last_error` com os dois 404 (sétima hora). Cursores que
avançaram:

| Repositório | `last_commit` | `commit_cursor` | `head_at_run` | `last_pr_number` | `last_run` |
|---|---|---|---|---|---|
| `ivall/xcl` | `4e59a814` | `e5266254` | `e13d6250` | 3155 | 2026-11-28T04:12:54Z |
| `brcly/TuxBox` | `26d0f6df` | `02fceb26` | `6f89916a` | 397 | 2026-11-28T01:16:56Z |
| `R0Y00/Xbox2Windows` | `74867491` | `d649e470` | `052abe6f` | 3354 | 2026-11-28T05:24:33Z |
| `fxceless/victory-fork` | `6b3ce6ce` | `368967f8` | `368967f8` | 1513 | 2026-11-28T01:48:22Z |

Os outros 10 mantêm cursor de commit e de merge. `backlog` com 10: os dois bloqueados, os três com
teto, `holdmysocks/SixNG`, `Marvisak/VIctory`, `DexNom/VIctory`, `uiliannn/VItoriaBr` e
`omonkeyz/xbox-emulator`; a ordem, os achados e a prioridade de 14:00 estão no `backlog_note`.

## Push

Este registro é escrito antes da publicação; o resultado do push entra no fechamento desta execução,
num segundo commit na `public` (o passo 5 limita a **um merge** por execução; o registro do push vai
por cherry-pick para a `main`, como nas execuções das 18:00 às 01:00).
