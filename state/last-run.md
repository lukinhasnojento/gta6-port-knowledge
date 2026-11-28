# Execução de 2026-11-28 16:00

## Data e hora

Coleta de 2026-11-28T16:07Z a 16:23Z (UTC real; 13:07–13:23 local, UTC−3). Diretório de documentos:
`docs/updates/2026-11-28/16/`. Estado lido: `baselined: true`, `runs: 27` (gravado como 28), **15
entradas** em `repos` no início e no fim, nenhuma removida, nenhuma acrescentada.

## Sanidade do estado (passo 1)

`gh api repos/<o>/<r> --jq .default_branch` foi pedido nos 15. Treze responderam com a branch já
gravada, inclusive `master` (`holdmysocks/SixNG`) e `a-publier` (`Belobug`, não lido nesta hora).
**Dois continuam sem resposta**: `Belobug/VIctory-Belobug-AMD` e `avatarmanz/VIctory-7900XTX`
devolvem HTTP 404 pela **décima hora**; cursores parados, triagem NAO VERIFICADO, detalhe em
`_falhas.md`. Nenhum `last_commit` nulo — não houve baseline. Nenhum `compare` devolveu `diverged`
(2 `ahead`, 11 `identical`, 2 sem resposta).

A contagem é 15, não 14: `trustno1-coder/VIctory-NVIDIA` foi acrescentado ao estado às 15:50Z, depois
da execução das 15:00. Nada foi reduzido.

## Triagem (passo 2)

Os 15 foram triados (13 com medição completa). Arquivo: `_triagem.md`.

| Repositório | Commits atrás | Abertos acima do cursor | Mesclados na janela | Sinal | Deep dive |
|---|---|---|---|---|---|
| `ivall/xcl` | 154 (128 sem documento) | 9 (máx. #3177) | 1 (#3100) | SIM (critério 2) | sim |
| `brcly/TuxBox` | 41 (18 sem documento) | 0 (máx. #393) | 2 (#401, #402) | SIM (critério 2) | sim |
| `DexNom/VIctory` | 0 | 52 (máx. #200) | 0 | SIM (**critério 1**: `#183`/`274f698` = D-12) | sim |
| `R0Y00/Xbox2Windows` | 0 | 4 (#3372–#3375) | 0 | SIM (critério 3) | sim |
| `Marvisak/VIctory` | 0 | ≥100 (piso, máx. #3759) | 0 | SIM fraco (2/3) | não |
| `uiliannn/VItoriaBr` | 0 | 3 (máx. #134) | 0 | SIM fraco (3) | não |
| `omonkeyz/xbox-emulator` | 0 | 1 (#395) | 0 | SIM fraco (3) | não |
| `trustno1-coder/VIctory-NVIDIA` | 0 | 100 lidos (máx. #1226; cursor nulo) | 0 | SIM fraco (3) | não |
| `californiaAlmonds/OpenScarlett` | 0 | 1 (#66) | 0 | NAO | não |
| `fxceless/victory-fork` | 0 | 0 | 0 | NAO | não |
| `holdmysocks/SixNG` | 0 | 0 | 0 | NAO | não |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 | 0 | NAO | não |
| `stellarr-studio/kelvaroq` | 0 | 0 | 0 | NAO | não |
| `Belobug/VIctory-Belobug-AMD` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | 404 | bloqueado |
| `avatarmanz/VIctory-7900XTX` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | 404 | bloqueado |

**Critério 1 acionou pela primeira vez por hash**: `DexNom` PR `#183` restaura `274f698`, e a linha de
prosa de D-12 do catálogo registra `274f698` / `17de445` como conserto/reintrodução do overflow de u64
no `clwb_loop`. O diff lido devolve `clwb_loop_end` com `checked_mul(64)` e `checked_add(step)` e
`None` quando o `add base, 0x40` estouraria. Nos outros repositórios, os 195 hashes de commit e os 3
de merge não têm interseção com o catálogo.

**Critério 4** não acionou: o maior atraso é `xcl`, com 154.

## Documentos escritos (passo 3)

4 documentos em `docs/updates/2026-11-28/16/`: `xcl.md`, `TuxBox.md`, `VIctory-DexNom.md`,
`Xbox2Windows.md`, mais `_triagem.md` e `_falhas.md`.

**58 commits documentados de 146 sem documento** (`xcl` 40 de 128, `TuxBox` 18 de 18), **65 PRs
abertos** lidos (`DexNom` 52, `xcl` 9, `Xbox2Windows` 4) e **3 merges** (`xcl` #3100, `TuxBox` #401 e
#402).

Achados principais:

- **`DexNom`: o publish `570a941` derrubou 42 mudanças.** O commit
  `570a941e9b4e945121c26eb30dfc10c126845207` ("Publish DexNom Runtime Radeon sources…",
  2026-11-25T12:18:03Z, 170 arquivos, +49.715/−36.762) é ancestral da `main` e 42 dos 52 PRs abertos
  são `cherry-pick -x` de commits que ele descartou. O catálogo aponta esse publish como introdutor de
  D-01, D-04, D-05, D-06, D-08, D-09 e D-14 do repositório e registra D-10/D-11/D-12 como abertos por
  reversão. `#183` fecha D-12 **se mesclado**; `#180` (cache `.tmp`) e `#176` (stride) tocam as áreas de
  D-11 e D-10, mas com hashes diferentes dos registrados.
- **`xcl`: andaime e um merge de importação.** Quarenta commits de `bmjask`: fila de candidatos à
  evicção (`b4625a4d`, `48286fdd`, `379aa29d`), ledger de upload esparso
  (`a3fdbde4`, `bed14445`, `5674db9d`, `4b0c6ad7`, `fdd9df7b`, `c5d854d7`, `e3fb670c`, `8f3998f1`,
  `cfb7374c`), classificador de restauração dormente (`f19a07de`, `38452a17`), bancos sintéticos
  (`1d942ea8`, `c35d24a8`, `1120fdc9`, `622c5e61`) — quase tudo `#[allow(dead_code)]` ou `#[cfg(test)]`.
  O que tem consequência: o merge `#3100` (`bb9a6b4b`) abre a biblioteca de pipeline anterior como
  fonte de importação e copia PSOs exatos em vez de recompilar tudo na invalidação semântica; foi
  **mesclado como draft**, sem smoke main/head nem build release. `XCL_GPU_TIMING=1` (`cfc11ab6`,
  `3cf40d1c`) mede `ExecuteCommandLists` no CPU, não GPU.
- **`TuxBox`: memória de vídeo e pipelines.** `2ea48307` libera pipelines gráficos além dos primeiros
  4096 e os reconstrói no primeiro bind pelo cache do driver (o título cria mais de 80.000; ~150 KB de
  host por pipeline na NVIDIA). `ce8a7c86` desvia textura amostrada para memória de sistema quando
  faltam menos de `TUXBOX_VRAM_RESERVE_MIB`; `b295769f` aloca só os mips que cabem antes da próxima
  textura; `b244f883` divide o guest view heap em heaps de CPU menores. `055adb92` faz a espera por
  label honrar uma escrita gravada mas ainda não submetida. `#401` corrige o `Map` de um buffer de 16
  bytes que usava range de 32; `#402` atualiza as receitas Windows 11 para 73/76 e 71/76.
- **`Xbox2Windows`: quatro PRs, um com sintoma.** `#3373` compõe todos os planos quando há mais de um
  (plano 0 preto deixava a janela preta); `#3375` troca a máscara do índice de mesh de 1023 para 511,
  porque o slot é de 9 bits com o flag de aresta no bit 9 e a guarda cullava a primitiva; `#3372`
  permite `image_store` a16 com dados de 32 bits no compute direto; `#3374` tira caminhos locais de GDK.

## Falhas e desvios

- **Dois repositórios inacessíveis (404, décima hora).** Não avançaram cursor, não foram documentados,
  não saíram de `repos`.
- **15 entradas, não 14.** `trustno1-coder/VIctory-NVIDIA` entrou às 15:50Z; triado com
  `last_pr_number: null`, 100 PRs lidos e nenhum por diff.
- **A "tabela de pistas" continua inexistente** em `docs/automacao/ANALISE_HORARIA.md`; as áreas vieram
  das seções D do catálogo e do corpo dos PRs.
- **Ordem do `compare` instável:** o novo `commit_cursor` do `xcl` (`3cde0cc3`) re-inclui `2dad32e6` e
  `abf7e177`, já documentados às 12:00. O próximo ciclo tem de pular hash documentado.
- **`xcl` #3100 mesclado como draft**, com a suíte em 1.999 pass / 7 fail herdadas da `main`.
- **`DexNom`:** só 4 dos 52 PRs foram lidos por diff (`#183`, `#194`, `#156`, `#200`); os outros 48
  por título, corpo, arquivos e `+/-`. `#173` ("Vanviro 16gb", +1.016.414 linhas) está fora de escopo.
- **Nenhum subagente disponível** neste ambiente; os quatro repositórios foram processados em
  sequência.
- **Rate limit:** nenhum 403 nesta coleta.
- **Escrita:** somente em `G:\gta6-port-knowledge`. Os 15 monitorados foram só leitura de API.

## Estado gravado

`runs: 28`, `updated: 2026-11-28T16:23:31Z`, `last_error` com os dois 404 (décima hora). Cursores que
avançaram:

| Repositório | `last_commit` | `commit_cursor` | `head_at_run` | `last_pr_number` | `last_run` |
|---|---|---|---|---|---|
| `ivall/xcl` | `3cde0cc3` | `3cde0cc3` | `bb9a6b4b` | 3177 | 2026-11-28T15:37:54Z |
| `brcly/TuxBox` | `2af42c94` | — | — | 398 | 2026-11-28T15:29:35Z |
| `DexNom/VIctory` | `4ad8692` (parado) | — | — | 200 | 2026-11-26T04:15:00Z (sem merge) |
| `R0Y00/Xbox2Windows` | `f5a6f6ba` (parado) | — | — | 3375 | 2026-11-28T14:15:27Z (sem merge) |

`TuxBox` perdeu `commit_cursor`/`head_at_run` porque a fila zerou. Os demais mantêm cursor de commit e
de merge. `last_run` só avançou onde houve merge coberto — nos dois repositórios sem merge ele ficou
parado, para não esconder merge futuro. `backlog` com 7 entradas: `xcl` (114 commits na fila),
`Marvisak` (piso de 100 abertos), `VItoriaBr` (3), `omonkeyz` (1), `NVIDIA` (baseline de PRs) e os
dois bloqueados.

## Push

Executado ao fim desta execução; o resultado fica no commit de fechamento (`state/last-run.md`
atualizado depois das duas publicações, como nos ciclos anteriores).
