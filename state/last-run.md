# Execução de 2026-11-28 18:00

## Data e hora

Coleta de 2026-11-28T18:07Z a 18:57Z (UTC real; 15:07–15:57 local, UTC−3). Diretório de documentos:
`docs/updates/2026-11-28/18/`. Estado lido: `baselined: true`, `runs: 29` (gravado como 30), **15
entradas** em `repos` no início e no fim, nenhuma removida, nenhuma acrescentada.

## Sanidade do estado (passo 1)

`gh api repos/<o>/<r> --jq .default_branch` foi pedido nos 15. Treze responderam com a branch já gravada,
inclusive `master` (`holdmysocks/SixNG`). **Dois continuam sem resposta**: `Belobug/VIctory-Belobug-AMD`
e `avatarmanz/VIctory-7900XTX` devolvem HTTP 404 em `repos/` e em `users/` pela **décima segunda hora**;
cursores parados, triagem NAO VERIFICADO, detalhe em `_falhas.md`. Nenhuma branch divergiu do estado,
então nada precisou ser corrigido antes do delta. Nenhum `last_commit` nulo — não houve baseline nesta
execução. Nenhum `compare` devolveu `diverged` na coleta (4 `ahead`, 9 `identical`, 2 sem resposta).

## Triagem (passo 2)

Os 15 foram triados (13 com medição completa). Arquivo: `_triagem.md`.

| Repositório | Commits atrás | Abertos acima do cursor | Mesclados na janela | Sinal | Deep dive |
|---|---|---|---|---|---|
| `ivall/xcl` | 80 no `compare` (78 sem documento) | 10 (máx. #3198) | 0 | SIM (critério 2) | sim |
| `R0Y00/Xbox2Windows` | 11 | 7 (máx. #3387) | 4 (#3031, #3125, #3139, #3375) | SIM (critério 2) | sim |
| `holdmysocks/SixNG` | 1 | 0 | 1 (#555) | SIM (critério 2) | sim |
| `Marvisak/VIctory` | 0 | ≥50 (piso, máx. #3775) | 0 | SIM (critério 3) | sim |
| `brcly/TuxBox` | 0 | 2 (#406, #407) | 0 | SIM fraco (2/3) | não |
| `fxceless/victory-fork` | 0 | 3 (#1517–#1519) | 0 | SIM fraco (3) | não |
| `DexNom/VIctory` | 0 | 1 (#201) | 0 | SIM fraco (2) | não |
| `omonkeyz/xbox-emulator` | 0 | 1 (#395) | 0 | SIM fraco (3) | não |
| `trustno1-coder/VIctory-NVIDIA` | 1 (workflow) | 50 lidos (cursor nulo) | 0 | NAO | não |
| `californiaAlmonds/OpenScarlett` | 0 | 1 (#66) | 0 | NAO | não |
| `uiliannn/VItoriaBr` | 0 | 0 | 0 | NAO | não |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 | 0 | NAO | não |
| `stellarr-studio/kelvaroq` | 0 | 0 | 0 | NAO | não |
| `Belobug/VIctory-Belobug-AMD` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | 404 | bloqueado |
| `avatarmanz/VIctory-7900XTX` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | 404 | bloqueado |

**Critério 1 não acionou.** Os 270 tokens hexadecimais de `catalogo/defeitos.csv` foram conferidos
contra os 93 hashes de commit da janela (`xcl` 80, `Xbox2Windows` 11, `SixNG` 1, `NVIDIA` 1) e os 5
hashes de merge (`d418f70e`, `6685de7c`, `7291aef2`, `f2972840`, `cff3f31b`): **zero interseções**,
inclusive por prefixo de 7 caracteres.

**Critério 2 acionou em `xcl`, `Xbox2Windows` e `SixNG`**: em `xcl` quatro commits que recusam ou
param em vez de seguir sem prova (`dd84e0ad`, `c7752525`, `40d3a312`, `86a90eeb`) e dois PRs abertos de
conserto (`#3192`, `#3194`); em `Xbox2Windows` `#3375` (máscara 1023 → 511 num campo de 9 bits) e
`#3139` (PSO nativo falho agora recupera o par de disco); em `SixNG` `#555` (nome de decoração que é
conjunto era descartado). **Critério 3** acionou em `Marvisak`, `TuxBox`, `victory-fork` e `omonkeyz`.
A "tabela de pistas" continua inexistente em `docs/automacao/ANALISE_HORARIA.md`; as áreas vieram das
seções D do catálogo e do corpo dos PRs. **Critério 4** não acionou (maior atraso 80, em `xcl`).

## Documentos escritos (passo 3)

4 documentos em `docs/updates/2026-11-28/18/`: `xcl.md`, `Xbox2Windows.md`, `SixNG.md`, `VIctory.md`
(`Marvisak`), mais `_triagem.md` e `_falhas.md`.

**52 commits documentados de 91 na janela** (`xcl` 40 de 78 sem documento — teto de 40 atingido,
`Xbox2Windows` 11 de 11, `SixNG` 1 de 1), **29 PRs abertos** (`Marvisak` 12 novos de ≥50, `xcl` 10,
`Xbox2Windows` 7) e **5 PRs mesclados** (`Xbox2Windows` 4, `SixNG` 1).

Achados principais:

- **`Xbox2Windows`: dois consertos com sintoma e um merge fora da `main`.** `#3375` (`f2972840`,
  `24b895d7`) corrige o frontend de mesh: o export primitivo empacota três slots de 10 bits que carregam
  índice de **9 bits** com o flag de aresta no bit 9; o C++ mascarava com `1023`, então índice com flag
  virava 512..1023 e a guarda `triangle[c] >= vertices` cullava a primitiva (o fixture novo cobre
  `0x20280600` com as três flags; controle negativo com 1023 falha). `#3139` (`7291aef2`) faz o
  candidato nativo, ao falhar `CreateGraphicsPipelineState`, recarregar o par de disco publicado da
  mesma chave e variante RTV e recompilar com revisão incrementada, em vez de devolver e cachear nulo
  (alvo 0 vermelho/32 limpos contra 16/16 com o par). **Mas `#3139` foi mesclado em
  `codex/native-graphics-integration`, não na `main`:** `compare/7291aef2...main` devolve `diverged`
  (16 à frente, 1 atrás) e o merge não está na faixa de commits. Os outros dois merges são lowerings do
  tradutor direto: `#3031` (`d418f70e`) `v_dot4_i32_i8`/`v_dot4_u32_u8` no `gfx2hlsl` e `#3125`
  (`6685de7c`) `v_min_f16`/`v_max_f16` no compute direto, ambos com verificador Python próprio.
- **`xcl`: a hora de falhar fechado.** `dd84e0ad` para o título quando o tempo de parede não prova que
  produtor de CPU ou GPU terminou, em vez de sintetizar conclusão de fence; `c7752525` deixa de tratar
  célula de espera não mapeada como satisfeita; `40d3a312` troca a sonda tipo `VirtualQuery` por
  `ReadProcessMemory` (o Windows contém o fault) com ordenação acquire; `86a90eeb` deixa de substituir
  por zero um rendezvous wave64 que não chegou — podia publicar estado meio-local como onda de console
  bem-sucedida. Ao redor, 34 commits de andaime: identidade exata de PSO retido em `SetPipelineState`,
  `ClearState`, `CreateCommandList`/`Reset` e `ExecuteIndirect` (`52436c9a` a `01f85586`), evicção de
  compute limitada a um pipeline a cada 2 s, opt-in (`5ec65338`), e estado esparso fora do lock
  (`df7b9fd5`, `97e80fbf`, `51cda2f4`, `687d553c`), quase tudo declarado como preparação. O conserto de
  unidade é `651867a4` (`#3085`): `prepare_workers` passava commit em **bytes** a `workers_for(cores,
  free_mb)`, que divide por 3072 como MiB, então o teto nunca valia e 6 GiB livres davam 12 workers em
  vez de 2. Ficam **38 commits** na fila (cursor `f2c0d36f`).
- **`SixNG`: decoração que é conjunto.** `#555` (`cff3f31b`) resolve, em `peddeco.rs` (+51/−2), nome de
  decoração que aponta para um **conjunto** do catálogo (struct `0x91501997`, `Name`/`Values`) para os
  registros de corpo que ele nomeia, cada um na parte da própria categoria; registros de cabeça do
  conjunto ficam de fora, com a justificativa declarada de que repetem uma camada de corpo cujo lugar na
  malha da cabeça não é conhecido. O `CATALOGUE_CACHE_VERSION` sobe para 6 no mesmo commit.
- **`Marvisak`: 12 PRs novos e nenhum commit.** `#3774` porta de `NVIDIA #458` a prova de forma de
  waterfall sobre offset de tabela de primeira lane e acrescenta a recusa própria de descritor uniforme
  **carregado** que escapa do laço (inclui os valores SSA do descritor no conjunto de escape; a
  regressão lê `s12` em wave32 e wave64) — área de `VIctory D-01`/`D-06`. `#3771` corrige
  `FORMAT_SUPPORT` de `DXGI_FORMAT_UNKNOWN`, que anunciava amostragem e render target (convidado em
  `63bcca4`: `0x7fffffff`/`0x0000ffff`; WARP: `0x1`/`0x200`). `#3767` limita a varredura de nomes SGD5
  aos últimos 255 bytes. O resto é emissão DXIL com número medido no corpo (`v_ashrrev_i16` 31 → 27,
  `v_cvt_pkrtz_f16_f32` 8 instruções menos por módulo, `v_ashrrev_i16` e máscaras de meia palavra).

## Falhas e desvios

- **Dois repositórios inacessíveis (404, décima segunda hora).** Não avançaram cursor, não foram
  documentados, não saíram de `repos`.
- **A "tabela de pistas" continua inexistente** em `docs/automacao/ANALISE_HORARIA.md`.
- **Ordem do `compare` instável, de novo.** Em `xcl`, `2dad32e6` e `abf7e177` (documentados às 12:00)
  voltaram a aparecer dentro da faixa e foram pulados **por hash**.
- **Merge fora da branch default.** `#3139` do `Xbox2Windows` entrou em
  `codex/native-graphics-integration`; documentado pelo diff do PR e **não** usado como cursor de
  commits (o cursor é `f2972840`, que não o contém).
- **Merge durante a coleta.** `#3375` (`f2972840`) chegou às 18:08Z, depois do snapshot do `compare`; a
  faixa foi refeita com `compare/6685de7c...f2972840` e o conteúdo lido no diff do PR.
- **Correção de um hash escrito errado nesta execução.** Na primeira gravação de `state/watch.json`
  escrevi `last_commit` do `xcl` com um SHA inventado a partir do prefixo `f2c0d36f`. Foi corrigido no
  mesmo minuto com o valor real de `repos/ivall/xcl/commits/f2c0d36f` =
  `f2c0d36ff01a7b5c7d2a58e682249a5e8bfc4647`, conferido de volta no arquivo. Nenhum documento de
  `docs/` carregava o valor inventado. Registro aqui porque a rotina não pode inventar hash.
- **`last_run` parado em 11 repositórios** sem merge, de propósito: avançar para o horário da execução
  esconderia merge futuro (a lição dos dois relógios). Só avançaram `Xbox2Windows`
  (`2026-11-28T18:08:47Z`) e `SixNG` (`2026-11-28T17:41:36Z`).
- **Relógio do GitHub:** ao contrário de 2026-11-27, os `merged_at` desta coleta já batiam com o UTC
  real (`#3375` com `18:08:47Z`, lido às 18:10Z). A correção fixa de 3 h não se aplica mais; a nota do
  estado foi atualizada para mandar conferir o relógio em vez de aplicar deslocamento.
- **Nenhum subagente disponível** neste ambiente; os quatro repositórios foram processados em sequência.
- **Rate limit:** nenhum 403 (4.984 restantes no início, 4.835 ao fim).
- **Escrita:** somente em `G:\gta6-port-knowledge`. Os 15 monitorados foram só leitura de API.

## Estado gravado

`runs: 30`, `updated: 2026-11-28T18:57:00Z`, `last_error` com os dois 404 (décima segunda hora),
**15 entradas** em `repos`. Cursores que avançaram:

| Repositório | `last_commit` | `commit_cursor` | `head_at_run` | `last_pr_number` | `last_run` |
|---|---|---|---|---|---|
| `ivall/xcl` | `f2c0d36f` | `f2c0d36f` | `10d27297` | 3198 | 2026-11-28T16:59:49Z (sem merge) |
| `R0Y00/Xbox2Windows` | `f2972840` | — | — | 3387 | 2026-11-28T18:08:47Z |
| `holdmysocks/SixNG` | `cff3f31b` | — | — | 541 | 2026-11-28T17:41:36Z |
| `Marvisak/VIctory` | `63bcca43` (parado) | — | — | 3775 | 2026-11-27T13:20:45Z (sem merge) |
| `californiaAlmonds/OpenScarlett` | `48a98fcd` (parado) | — | — | 66 | 2026-11-27T13:05:00Z |
| `DexNom/VIctory` | `4ad86927` (parado) | — | — | 201 | 2026-11-26T04:15:00Z |
| `fxceless/victory-fork` | `368967f8` (parado) | — | — | 1519 | 2026-11-28T01:48:22Z |
| `omonkeyz/xbox-emulator` | `a618736d` (parado) | — | — | 395 | 2026-11-26T04:15:00Z |
| `brcly/TuxBox` | `b8355fa2` (parado) | — | — | 407 | 2026-11-28T17:12:17Z |
| `trustno1-coder/VIctory-NVIDIA` | `93008750` | — | — | `null` (piso) | 2026-11-28T15:50:07Z |

`xcl` mantém `commit_cursor` e `head_at_run` porque o teto de 40 cortou a faixa de 78. `backlog` com 9
entradas: `xcl` (38 na fila), `Marvisak` (piso de 100 abertos), `NVIDIA` (cursor de abertos nulo),
`DexNom` (`#201`), `TuxBox` (`#406`/`#407`), `victory-fork` (`#1517`–`#1519`), `omonkeyz` (`#395`) e os
dois bloqueados.

## Push

Executado e conferido:

- Commit na `public`: `docs(updates): 2026-11-28 18:00` (8 arquivos: os 4 documentos, `_triagem.md`,
  `_falhas.md`, `docs/updates/INDEX.md`, `state/watch.json` e `state/last-run.md`).
- `git push origin public:main`: sucesso.
- `git switch main` + `git merge --no-ff public`: merge `merge updates 2026-11-28 18:00`.
- `git push private main`: sucesso.
- Nenhum `git reset --hard`, nenhum merge de `main` na `public`.
