# Execução de 2026-11-28 00:00

## Data e hora

Coleta de 2026-11-28T00:07–00:17 UTC real — ~21:07–21:17 no relógio do GitHub, que segue cerca de 3 h
atrás e é o relógio em que `last_run` é gravado. Diretório de documentos: `docs/updates/2026-11-28/00/`
(UTC). Estado lido: `baselined: true`, `runs: 21` (gravado como 22); 14 entradas em `repos` no início e no
fim, nenhuma removida, nenhuma acrescentada.

## Sanidade do estado (passo 1)

`gh api repos/<o>/<r> --jq .default_branch` foi pedido nos 14. **Dois continuam sem resposta**:
`Belobug/VIctory-Belobug-AMD` e `avatarmanz/VIctory-7900XTX` devolvem HTTP 404 (quarta hora seguida),
também em `users/<login>`; a `default_branch` deles não pôde ser reconferida, os cursores ficaram parados
e a triagem deles é NAO VERIFICADO. Detalhe em `docs/updates/2026-11-28/00/_falhas.md`.

Nos outros 12, `default_branch` idêntico ao gravado, inclusive `master` (`holdmysocks/SixNG`). Nenhum
`last_commit` nulo — não houve baseline nesta execução. Nenhum `compare` devolveu `diverged`. Contagem de
`repos` igual no início e no fim (14).

## Triagem (passo 2)

Os 14 foram triados (12 com medição completa); 4 entraram no deep dive. Arquivo:
`docs/updates/2026-11-28/00/_triagem.md`.

| Repositório | Commits atrás | Abertos acima do cursor | Mesclados na janela | Sinal | Deep dive |
|---|---|---|---|---|---|
| `R0Y00/Xbox2Windows` | 12 | 14 (100 no total) | 12 | SIM (critério 2; lowerings e `MIMG`) | sim |
| `brcly/TuxBox` | 2 | 3 (6 no total) | 0 | SIM (critério 2; harness de regressão do tradutor) | sim |
| `holdmysocks/SixNG` | 2 | 2 (5 no total) | 2 | SIM (critério 2; vidro de lente, adesivo sem textura) | sim |
| `fxceless/victory-fork` | 1 | 2 (100 no total) | 0 | SIM (critério 2; orçamento de VRAM) | sim |
| `Marvisak/VIctory` | 0 | 76 (100 no total) | 0 | SIM (critério 3; `GPU:`/`Shaders:`/`CLI:`) | não |
| `ivall/xcl` | 0 | 24 (100 no total) | 0 | SIM (critério 3; bibliotecas nativas de pipeline) | não |
| `uiliannn/VItoriaBr` | 0 | 1 (75 no total) | 0 | SIM fraco (critério 3; `#132`) | não |
| `omonkeyz/xbox-emulator` | 0 | 1 (25 no total) | 0 | SIM fraco (critério 3; `#395`) | não |
| `DexNom/VIctory` | 0 | 0 (100 no total, máx. 148 = cursor) | 0 | NAO | não |
| `californiaAlmonds/OpenScarlett` | 0 | 0 (1 no total) | 0 | NAO | não |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 | 0 | NAO | não |
| `stellarr-studio/kelvaroq` | 0 | 0 | 0 | NAO | não |
| `Belobug/VIctory-Belobug-AMD` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (404) | bloqueado |
| `avatarmanz/VIctory-7900XTX` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (404) | bloqueado |

Desempate: havia exatamente quatro repositórios com commit na janela e o critério 2 ocupou as quatro
vagas. O critério 1 (defeito do catálogo) não selecionou ninguém — os 12 commits foram conferidos contra
as 100 entradas dos quatro repositórios no `catalogo/defeitos.csv` e nenhum reaplica um D-xx. Nenhum
repositório passou de 200 commits de atraso, então o critério 4 não foi acionado.

## Documentos escritos (passo 3)

4 documentos em `docs/updates/2026-11-28/00/`: `Xbox2Windows.md`, `victory-fork.md`, `TuxBox.md`,
`SixNG.md`, mais `_triagem.md` e `_falhas.md`. **17 commits documentados de 17** atrás dos cursores
(12 + 2 + 2 + 1), **14 PRs mesclados** (12 + 2) e **21 PRs abertos lidos** (14 + 2 + 3 + 2 — o total de
123 abertos novos dos 14 está na triagem). Nenhum repositório chegou ao teto de 40 commits.

Achados principais:

- `R0Y00/Xbox2Windows` (`a5dad843` … `cf3df832`): a hora fecha recusas do compute direto — `s_cmov_b32/b64`
  e `s_cmovk_i32` (`671d2f46`, com as três entrando no conjunto `reads_scc`), as quatro SOPC de 64 bits
  (`98f7363d`), as quatro conversões `f16`↔inteiro de 16 bits (`98020080`), `v_sat_pk_u8_i16` no
  `gfx2hlsl` (`2e74b778`) — e `cf3df832` tira o bit 25 da máscara de bits reservados do `MIMG`, que
  provocava `unsupported image flags` em instrução válida. **Defeito verificável no código:**
  `98020080` publica `tests/native_cvt_f16_i16_check.py` comparando guardas com `F.SENTINEL`, e
  `tests/native_f16_alu_check.py` nesse commit não define `SENTINEL` (usa `0x69696969` inline) — a perna
  `native_cvt_f16_i16_hardware` levanta `AttributeError`; o conserto está no PR aberto `#3190`.
  `8e8b349b` é só documento e registra três testes que passam a falhar com o WARP 1.0.22 app-local.
- `fxceless/victory-fork` (`2eade68c`, push direto): portão em bloco de textura de 64 MiB — bloco novo só
  se `used + 64 MiB + buffers + targets a vir <= linha`; sem espaço, a textura espera e mostra a view
  anterior por até ~4 s, e o que a fila e a varredura pesam passa a ser `line + 64 MiB − livre` quando
  nenhum bloco pode ser criado. Os arenas de ray tracing passam a descontar os render targets a vir. Sem
  teste no diff; D-17 segue aberto.
- `brcly/TuxBox` (`b1023549`, `62c7e390`, pushes diretos): harness de seis estágios que compara uma
  mudança do tradutor com `origin/main` sobre shaders gravados e roda os checks de GPU em cada placa, mais
  um roadmap que admite **178 das 372 instruções usadas sem check de GPU** e **zero shaders reais
  executados e comparados com referência**. O harness precisa de `build/shader-harness.json` (fora do
  Git), Linux, DXC e Proton — não roda em CI.
- `holdmysocks/SixNG` (`#503`, `#504`): registro de transmissão do vidro de lente lido da biblioteca de
  materiais, com blend dual-source `dst' = src0 + dst × src1`; a escolha do sub-material é **por
  material**, pela maioria dos vértices (`(v.col[1] >> 2) & 7`), e `render3d_metal4.rs` grava
  `glass: None` — o Metal 4 não consome o registro. `#504` deixa de oferecer adesivo cuja textura não
  existe em store alguma e não apaga mais os adesivos do veículo quando a livery não tem nada posicionável.

## Falhas e desvios

- **Dois repositórios inacessíveis (404 de conta, quarta hora).** Registrado em `_falhas.md`. Não
  avançaram cursor, não foram documentados, não saíram de `repos`.
- **Falha de coleta corrigida na hora:** `compare` com o SHA devolvido por `--jq .sha` (com `\n`) montou URL
  inválida nos 12; refeita com o SHA aparado. Nenhum número desta execução vem da tentativa que falhou.
- **A "tabela de pistas" citada no passo 2 continua inexistente** em `docs/automacao/ANALISE_HORARIA.md`
  (o `INDEX.md` já registra o desvio desde as 17:00). As áreas do sinal vieram das seções do
  `catalogo/defeitos.md` e do assunto de cada PR; o sinal dos repositórios sem commit é por título/corpo de
  PR, não por patch.
- **Contagem de commits × merges:** bate nos quatro documentados. `Xbox2Windows` 12 = 12 merges e `SixNG`
  2 = 2 merges (squash de um pai, `merge_commit_sha` igual ao commit); `TuxBox` 2 e `victory-fork` 1 são
  push direto sem PR (`commits/<sha>/pulls` vazio nos três).
- **PRs abertos documentados pelo título e pelo corpo, não pelo patch.** 21 corpos lidos; nenhum patch de
  PR aberto foi lido linha a linha.
- **Três commits de ferramenta do `Xbox2Windows`** (`69530cc2`, `9e7eab5b`, `3d7b3a26`) foram lidos nos
  primeiros ~2.000 caracteres de patch por arquivo, não por inteiro.
- **Sem subagentes.** A ferramenta não existe neste ambiente; os quatro repositórios foram processados em
  sequência e o limite de 4 deep dives foi respeitado.
- **Rate limit:** 4.999 restantes no início, nenhum 403, nenhuma repetição em laço.
- **Escrita:** somente em `G:\gta6-port-knowledge`. Os 14 monitorados foram só leitura de API.
- **Arquivos de terceiros no diretório de trabalho** (`docs/analises/victory-fork.md` modificado e dois
  relatórios não rastreados em `docs/relatorios/`) ficaram fora do `git add` desta rotina e intactos.

## Estado gravado

`runs: 22`, `updated: 2026-11-28T00:17:42Z`, `last_error` com os dois 404 (quarta hora). Cursores que
avançaram:

| Repositório | `last_commit` | `last_pr_number` | `last_run` |
|---|---|---|---|
| `R0Y00/Xbox2Windows` | `cf3df832` (head) | 3191 | 2026-11-27T21:02:29Z |
| `brcly/TuxBox` | `62c7e390` (head) | 328 | 2026-11-27T20:01:33Z (sem merge) |
| `holdmysocks/SixNG` | `1b419021` (head) | 506 | 2026-11-27T20:41:30Z |
| `fxceless/victory-fork` | `2eade68c` (head) | 1496 | 2026-11-27T18:10:00Z (sem merge) |

`last_run` dos que tiveram merge é o maior `merged_at` documentado, no relógio do GitHub — não o horário
da execução, para não esconder merge entre o fim da coleta e a próxima hora. Os outros 10 mantêm cursor de
commit e de merge. `backlog` com 6: os dois bloqueados e `Marvisak/VIctory`, `ivall/xcl`,
`uiliannn/VItoriaBr`, `omonkeyz/xbox-emulator`; a ordem e os achados estão no `backlog_note`.

## Push

Registrado no commit de fechamento desta execução (abaixo).
