# Execução de 2026-11-28 17:00

## Data e hora

Coleta de 2026-11-28T17:07Z a 17:41Z (UTC real; 14:07–14:41 local, UTC−3). Diretório de documentos:
`docs/updates/2026-11-28/17/`. Estado lido: `baselined: true`, `runs: 28` (gravado como 29), **15
entradas** em `repos` no início e no fim, nenhuma removida, nenhuma acrescentada.

## Sanidade do estado (passo 1)

`gh api repos/<o>/<r> --jq .default_branch` foi pedido nos 15. Treze responderam com a branch já
gravada, inclusive `master` (`holdmysocks/SixNG`) e `a-publier` (`Belobug`, não lido nesta hora).
**Dois continuam sem resposta**: `Belobug/VIctory-Belobug-AMD` e `avatarmanz/VIctory-7900XTX`
devolvem HTTP 404 pela **décima primeira hora**; cursores parados, triagem NAO VERIFICADO, detalhe em
`_falhas.md`. Nenhum `last_commit` nulo — não houve baseline. Nenhum `compare` devolveu `diverged`
(3 `ahead`, 10 `identical`, 2 sem resposta).

## Triagem (passo 2)

Os 15 foram triados (13 com medição completa). Arquivo: `_triagem.md`.

| Repositório | Commits atrás | Abertos acima do cursor | Mesclados na janela | Sinal | Deep dive |
|---|---|---|---|---|---|
| `ivall/xcl` | 120 (118 sem documento) | 7 (máx. #3188) | 3 (#3175, #3181, #3182) | SIM (critério 2) | sim |
| `brcly/TuxBox` | 10 (+1 merge após o snapshot) | 2 (#404, #405) | 5 (#373, #385, #391, #393, #404) | SIM (critério 2) | sim |
| `R0Y00/Xbox2Windows` | 2 | 3 (#3376–#3378) | 1 (#3332) | SIM (critério 2/3) | sim |
| `uiliannn/VItoriaBr` | 0 | 3 (#132–#134) | 0 | SIM (critério 3) | sim |
| `Marvisak/VIctory` | 0 | ≥100 (piso, máx. #3764) | 0 | SIM fraco (2/3) | não |
| `DexNom/VIctory` | 0 | 1 (#201) | 0 | SIM fraco (2) | não |
| `omonkeyz/xbox-emulator` | 0 | 1 (#395) | 0 | SIM fraco (3) | não |
| `trustno1-coder/VIctory-NVIDIA` | 0 | 200 lidos (máx. #1299; cursor nulo) | 0 | SIM fraco (3) | não |
| `californiaAlmonds/OpenScarlett` | 0 | 1 (#66) | 0 | NAO | não |
| `fxceless/victory-fork` | 0 | 0 | 0 | NAO | não |
| `holdmysocks/SixNG` | 0 | 0 | 0 | NAO | não |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 | 0 | NAO | não |
| `stellarr-studio/kelvaroq` | 0 | 0 | 0 | NAO | não |
| `Belobug/VIctory-Belobug-AMD` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | 404 | bloqueado |
| `avatarmanz/VIctory-7900XTX` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | 404 | bloqueado |

**Critério 1 não acionou.** Os 270 tokens hexadecimais de `catalogo/defeitos.csv` foram conferidos
contra os 132 hashes de commit da janela e os 9 hashes de merge: zero interseção, inclusive por
prefixo de 7 caracteres.

**Critério 2 acionou em `xcl`, `TuxBox` e `Xbox2Windows`**: `xcl` tem três merges de ramos `fix/`
(`#3157` `DispatchX` wave-uniforme, `#3159` fronteira do bloco MSAA, `#3085` unidades de MiB) e seis
commits de recompilador que recusam ou adiam caminho inseguro; `TuxBox` mescla cinco PRs com sintoma
declarado (`#373` cita a issue #323: leitura do VA de heap liberado 11 min depois, device loss em
15–18 min); `Xbox2Windows` mescla `#3332` consertando `#3331`.

**Critério 3** acionou em `Xbox2Windows`, `VItoriaBr`, `Marvisak`, `DexNom`, `omonkeyz` e `NVIDIA`. A
"tabela de pistas" continua inexistente em `docs/automacao/ANALISE_HORARIA.md`; as áreas vieram das
seções D do catálogo e do corpo dos PRs. **Critério 4** não acionou (maior atraso 120, em `xcl`).

## Documentos escritos (passo 3)

4 documentos em `docs/updates/2026-11-28/17/`: `xcl.md`, `TuxBox.md`, `Xbox2Windows.md`,
`VItoriaBr.md`, mais `_triagem.md` e `_falhas.md`.

**53 commits documentados** (`xcl` 40 de 118 sem documento, `TuxBox` 11 de 11 contando o merge
posterior ao snapshot, `Xbox2Windows` 2 de 2), **23 PRs lidos** (9 mesclados, 14 abertos).

Achados principais:

- **`xcl`: identidade exata de pipeline.** `8cb7b0a0` a `c51cfd39` substituem o "endereço visível do
  título" por pin de geração exata em bind, draw e dispatch (`ef9ec838`, `ed5e9e47`, `077a0afa`,
  `056fc43b`, `e7567b1b`), com aposentadoria por recibo de submissão consultado **fora** do lock
  (`4a7f1175`: `SubmissionReceipt.marker: Arc<QueueFence>` no lugar da fence crua) e índice por
  identidade de alocação. No recompilador, a hora é de **recusar o que não se ordena**: `2ca91276`
  recusa `s_barrier` em grupo com mais de uma onda de hardware, `dbe6b30b` recusa LDS real no split
  wave64, `50e85c5b` recusa leitura de lane superior em wave64 gráfico dividido; `fb0931da` reconstrói
  reduções de onda inteira sobre as lanes lançadas. Os três merges (`#3175` `db3921dc`, `#3181`
  `59e4be57`, `#3182` `10d27297`) trazem o manifesto ordenado de primeiro uso de PSO e o passo offline
  `--through-present N`, com teto de 500 pipelines. Ficam **78 commits** na fila (cursor `4701a874`).
- **`TuxBox`: memória de descritor.** `#373` (`4cb3347d`, `ab0751c8`) mantém o heap de render de um
  alvo colocado em VA convidado enquanto o range está commitado — a issue #323 mostra o shader lendo o
  VA de um heap liberado 11 minutos depois e o device caindo em 15–18 min. `#391`/`#393`
  (`99d8569d`, `0ae06f52`) seguem os registros de descritor que o título copia com stores comuns para
  o heap indexado e guardam as views substituídas que esses registros ainda nomeiam (`history.rs`).
  `#385` (`19dc351b`) troca a consulta XG por tabelas de endereço por forma (`a40e5a8c`, 0,5–0,8 ms/MiB
  segundo o PR) e recicla buffers de upload (`661ef9cd`); `5de0e62b` recicla os bytes de mip alterado
  (teto de 64 buffers / 64 MiB). `#404` (`b8355fa2`) entrou **depois** do snapshot do `compare`.
- **`Xbox2Windows`: launchers.** `539c3ad5`/`#3332` (`aec2e084`) expõem `-SuspendOthers` /
  `--suspend-others` com validação 0/1, que antes só existia em `debug_launch.py` (issue #3331). Dos
  abertos, `#3376` compõe todos os planos de um present multi-plano por padrão (plano 0 preto deixava a
  janela preta, continuação do `#3373`), `#3377` introduz token de conteúdo de view (#1199) e `#3378`
  um preset compartilhado de execução.
- **`VItoriaBr`: só PRs.** `#133` põe `dx.op.getDimensions` fora da chave de CSE (o `amdxc64` cai em 29
  pipelines de raytracing, RDNA4), `#134` dá handle próprio às consultas de tamanho de buffer e `#132`
  faz o helper de teste estrito falhar quando o validador DXIL não carrega. Zero commits e zero merges:
  cursor de mesclados fica parado de propósito.

## Falhas e desvios

- **Dois repositórios inacessíveis (404, décima primeira hora).** Não avançaram cursor, não foram
  documentados, não saíram de `repos`.
- **A "tabela de pistas" continua inexistente** em `docs/automacao/ANALISE_HORARIA.md`.
- **Ordem do `compare` instável.** Em `xcl`, `2dad32e6` e `abf7e177` (documentados às 12:00) voltaram a
  aparecer dentro da faixa e foram pulados **por hash**; `2de72223` (conteúdo do `#3100`, documentado
  às 16:00) apareceu na posição 35 e não entrou nos 40. O novo cursor `4701a874` volta a re-incluir os
  dois primeiros — a regra de pular hash documentado continua obrigatória.
- **`TuxBox` mesclou o `#404` no meio da coleta** (17:12:17Z, depois do `compare`): o conteúdo foi lido
  no diff do PR e o merge `b8355fa2` gravado como `last_commit`, para o próximo ciclo não reprocessar.
- **`VItoriaBr` sem merge**: `last_run` mantido em `2026-11-27T13:05:00Z` (avançar para o horário da
  execução esconderia merge futuro — a lição dos dois relógios).
- **Nenhum subagente disponível** neste ambiente; os quatro repositórios foram processados em sequência.
- **Rate limit:** nenhum 403 (4.846 restantes no início, 4.777 no meio).
- **Escrita:** somente em `G:\gta6-port-knowledge`. Os 15 monitorados foram só leitura de API.

## Estado gravado

`runs: 29`, `updated: 2026-11-28T17:41:00Z`, `last_error` com os dois 404 (décima primeira hora).
Cursores que avançaram:

| Repositório | `last_commit` | `commit_cursor` | `head_at_run` | `last_pr_number` | `last_run` |
|---|---|---|---|---|---|
| `ivall/xcl` | `4701a874` | `4701a874` | `10d27297` | 3188 | 2026-11-28T16:59:49Z |
| `brcly/TuxBox` | `b8355fa2` | — | — | 405 | 2026-11-28T17:12:17Z |
| `R0Y00/Xbox2Windows` | `aec2e084` | — | — | 3378 | 2026-11-28T16:13:45Z |
| `uiliannn/VItoriaBr` | `15779f81` (parado) | — | — | 134 | 2026-11-27T13:05:00Z (sem merge) |

`TuxBox` sai da fila de commits (não tem `commit_cursor`). `xcl` mantém `commit_cursor` e `head_at_run`
porque o teto de 40 cortou a faixa de 118. `backlog` com 8 entradas: `xcl` (78 commits na fila),
`Marvisak`, `DexNom` (`#201`), `VItoriaBr` (3 abertos), `omonkeyz`, `NVIDIA` (cursor de abertos nulo) e
os dois bloqueados.

## Push

Executado e conferido:

- Commit na `public`: `adacc0c` — `docs(updates): 2026-11-28 17:00` (9 arquivos: os 4 documentos,
  `_triagem.md`, `_falhas.md`, `docs/updates/INDEX.md`, `state/watch.json` e `state/last-run.md`).
- `git push origin public:main`: **sucesso** — `4d2e96c..adacc0c  public -> main`; conferido por
  `git ls-remote origin refs/heads/main` = `adacc0c`.
- `git switch main` + `git merge --no-ff public`: **um conflito** em `state/last-run.md` (a `main`
  carregava o fechamento da execução de 16:00). Resolvido tomando o arquivo da `public` — o desta
  execução substitui o anterior; merge `e8d422a` — `merge updates 2026-11-28 17:00`.
- `git push private main`: **sucesso** — `b39ad4e..e8d422a  main -> main`; conferido por
  `git ls-remote private refs/heads/main` = `e8d422a`.
- Nenhum `git reset --hard`, nenhum merge de `main` na `public`.
- Este fechamento é um segundo commit na `public`, depois do merge; para não criar um segundo merge na
  execução, a `main` o recebe por cherry-pick, como nos ciclos anteriores.
