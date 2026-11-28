# Execução de 2026-11-28 19:00

## Data e hora

Coleta de 2026-11-28T19:07Z a 19:2xZ (UTC real; 16:07–16:2x local, UTC−3). Diretório de documentos:
`docs/updates/2026-11-28/19/`. Estado lido: `baselined: true`, `runs: 30` (gravado como 31), **15
entradas** em `repos` no início e no fim, nenhuma removida, nenhuma acrescentada.

## Sanidade do estado (passo 1)

`gh api repos/<o>/<r> --jq .default_branch` foi pedido nos 15. Treze responderam com a branch já gravada,
inclusive `master` (`holdmysocks/SixNG`). **Dois continuam sem resposta**: `Belobug/VIctory-Belobug-AMD`
e `avatarmanz/VIctory-7900XTX` devolvem HTTP 404 em `repos/` pela **décima terceira hora** — cursores
parados, triagem NAO VERIFICADO, detalhe em `_falhas.md`. Nenhuma branch divergiu do estado, então nada
precisou ser corrigido antes do delta. Nenhum `last_commit` nulo — não houve baseline. Nenhum `compare`
devolveu `diverged` entre cursor e head.

## Triagem (passo 2)

Os 15 foram triados (13 com medição completa). Arquivo: `_triagem.md`.

| Repositório | Commits atras | Abertos acima do cursor | Mesclados na janela | Sinal | Deep dive |
|---|---|---|---|---|---|
| `ivall/xcl` | 45 no `compare` (**10** sem documento) | 2 (máx. #3200) | 0 | SIM (critério 2) | sim |
| `R0Y00/Xbox2Windows` | 3 | 6 (máx. #3395) | 1 (#3357 `eeb52277`) | SIM (critério 2) | sim |
| `trustno1-coder/VIctory-NVIDIA` | 70 | 50 lidos (cursor nulo, máx. #1409) | 30 (#33–#132) | SIM (critério 2) | sim |
| `Marvisak/VIctory` | 0 | 7 (máx. #3783) | 0 | SIM (critério 3) | sim |
| `brcly/TuxBox` | 0 | 1 (#408) | 0 | SIM fraco (2/3) | não |
| `fxceless/victory-fork` | 0 | 0 (máx. #1519) | 0 | SIM fraco (2) | não |
| `DexNom/VIctory` | 0 | 0 (máx. #201) | 0 | SIM fraco (2/3) | não |
| `uiliannn/VItoriaBr` | 0 | 0 (máx. #134) | 0 | SIM fraco (3) | não |
| `omonkeyz/xbox-emulator` | 0 | 0 (máx. #395) | 0 | SIM fraco (3) | não |
| `californiaAlmonds/OpenScarlett` | 0 | 1 (#66) | 0 | NAO | não |
| `holdmysocks/SixNG` | 0 | 0 | 0 | NAO | não |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 | 0 | NAO | não |
| `stellarr-studio/kelvaroq` | 0 | 0 | 0 | NAO | não |
| `Belobug/VIctory-Belobug-AMD` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | 404 | bloqueado |
| `avatarmanz/VIctory-7900XTX` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | 404 | bloqueado |

**Critério 1 não acionou.** Os 262 tokens hexadecimais de `catalogo/defeitos.csv` foram conferidos
contra os 118 hashes das três janelas com commit (`xcl` 45, `Xbox2Windows` 3, `NVIDIA` 70): **zero
interseções**, inclusive por prefixo de 7 caracteres.

**Critério 2 acionou em `xcl`, `Xbox2Windows` e `NVIDIA`**: em `xcl` `bba710e2` é guarda de regressão da
classe de D-24 e `2de1544d` estende o limite `s_setpc`/`gs_start` ao wave64 nativo; em `Xbox2Windows` o
merge `#3357` fecha uma recusa de destino em `s_bfm_b64`; em `NVIDIA` os 30 merges são consertos e
desempenho com número medido. **Critério 3** acionou em `Marvisak`, `TuxBox`, `victory-fork`,
`omonkeyz` e `VItoriaBr`; a "tabela de pistas" continua inexistente em `docs/automacao/ANALISE_HORARIA.md`
(a única menção é a linha 233, que a referencia), então as áreas vieram das seções D do catálogo e do
corpo dos PRs. **Critério 4** não acionou (maior atraso 70, em `NVIDIA`).

## Documentos escritos (passo 3)

4 documentos em `docs/updates/2026-11-28/19/`: `VIctory-NVIDIA.md`, `xcl.md`, `Xbox2Windows.md`,
`VIctory.md` (`Marvisak`), mais `_triagem.md` e `_falhas.md`.

**53 commits descritos um a um** (`NVIDIA` 40, `xcl` 10, `Xbox2Windows` 3) **mais 30 merges em tabela de
PR** (`NVIDIA`), **77 PRs abertos lidos** (`NVIDIA` 50, `xcl` 14, `Marvisak` 7, `Xbox2Windows` 6) e
**31 mesclados** (`NVIDIA` 30, `Xbox2Windows` 1). Janela de commits dos quatro: 118 (`xcl` 45, `NVIDIA`
70, `Xbox2Windows` 3, `Marvisak` 0).

Achados principais:

- **`VIctory-NVIDIA`: 30 PRs parados mesclados em dois lotes.** 24 entre 18:12:15Z e 18:12:20Z e 6 entre
  18:20:42Z e 18:22:18Z. A ordem de merge não acompanha o número do PR (`#124`/`#126`/`#127`/`#132`
  entraram antes de `#35`/`#52`; `#68`, o maior número do primeiro lote, entrou com ele). Conteúdo
  substantivo: `00d5623c` (+4.665 em `bitcode/opt.rs` — dobras algébricas, LICM puro, forwarding
  store→load em espaço 0, reuso load→load em espaço 3, dedup de barreira; `DEFAULT` 447); `075785eb`
  (+3.482 em `emit/lds.rs` — alias estático e perigo lane-private no lugar da checagem grosseira, com
  oráculo de 48.212 casos); `05a269d8` (o teto de VRAM dos alvos de render vira 1,5× o pico ocupado em
  vez de 3.072 MiB fixos — numa 4060 Laptop de 8 GB, 2.675 MB presos recusavam textura nova); `a162ae7f`
  (a biblioteca de pipeline inteira deixa de ser re-salva em rajada: 180 s em burst, 600 s sob pressão);
  `3f6ba011` (o upload de prelude reconfere `host_written` na submissão — antes a memória crua do título
  cobria o rosto pintado); `ba360f5f` (duas funções do build devolvem 6 e a Series S usa o upscaler do
  título, não FSR 4; 28,5 → 30,9 fps medidos pelo autor). **Dois commits da faixa são `Merge main`**
  (`0eb34b84`, `76e3c697`) e o diff deles contra o primeiro pai é o que a `main` já tinha — está marcado
  na tabela.
- **`xcl`: a segunda ocorrência da classe de D-24, agora com guarda.** `56237a2d` pôs `getDimensions`
  antes dos loads de `Texture2DMS`; a simplificação de CFG re-fundiu o bloco e o padrão de crash do
  `amdxc64` voltou (registrado às 15:00 para o merge `309c3ec5`). `bba710e2` marca o bloco
  (`preserve_block_boundary: Vec<bool>`) e recusa a fusão, com teste. `2de1544d` aplica ao wave64 nativo
  os limites de `s_setpc`/`gs_start` que só valiam no split (classe de D-19). `5f284cc2` faz as escritas
  da própria lista pousarem num chunk esparso que a CPU também alterou — o e2e falhava com
  `post-Wait32/64 write failed (0x0)`. Quatro dos 10 commits são o **mesmo** `E0451` rebasado
  (`15b1845a`) e três corrigem assertivas que nunca podiam passar (`99e2b1b3` com o `ResBind` de memória
  do título, `1ecc3035` com o root UAV que dá a volta, `1d9a6424` com as recusas do direto respondidas
  por HLSL).
- **`Xbox2Windows`: VCC como destino de SALU.** `2e460b26` (`gfx1013_direct_compute.inl` +1/-1) aceita
  VCC (106): a condição vira `(i.dst & 1) || (i.dst > 104 && i.dst != 106 && i.dst != 126)`, `VCC_HI`
  (107) continua recusado pelo bit 0 e `M0` (124) segue recusado — o controle negativo do teste muda de
  VCC para M0, o que é evidência de diff de que VCC era erro antes. `f81feb16` conserta a codificação do
  próprio teste (`vmove(1, 9)` codifica `v_mov_b32 v1, s9`). Merge `#3357` = `eeb52277`, 19:07:04Z.
- **`Marvisak`: sete PRs e nenhum commit.** `#3780` limpa o bit `IA_INDEX_BUFFER` (`0x4`) do
  `FORMAT_SUPPORT` para todo formato que não seja `R16_UINT`/`R32_UINT` — segunda passada na mesma
  máscara de `#3771`; `#3776` dobra 65.536 codificações half conhecidas em tempo de tradução; `#3778` e
  `#3782` tiram dominadores repetidos e fatos de comparação recomputados; `#3781` troca recursão por
  caminhada iterativa; `#3783` aplica escala XMA no lugar com saída declarada bit a bit igual.

## Falhas e desvios

- **Dois repositórios inacessíveis (404, décima terceira hora).** Não avançaram cursor, não foram
  documentados, não saíram de `repos`.
- **Desvio de cursor no `xcl`, e é o achado operacional da hora.** O `last_commit` gravado às 18:00
  (`f2c0d36f`) **não contém** `86a90eeb`, que já estava documentado: `compare/86a90eeb...f2c0d36f`
  devolve `diverged` (5 à frente, 3 atrás). O `compare` a partir do cursor devolve 45 commits, dos quais
  35 já estão em `docs/updates/*/xcl.md`; o "38 restantes" daquela hora (78 − 40) não se reproduz. Os 35
  foram pulados **por hash**, os 10 sem documento foram documentados e o cursor avança para o head
  `10d27297`, zerando a fila de commits do repositório.
- **A "tabela de pistas" continua inexistente** em `docs/automacao/ANALISE_HORARIA.md`.
- **`#3199` do `xcl` fechado durante a coleta**, sem merge; não foi contado como mesclado.
- **`#3390` e `#3392` do `Xbox2Windows` não aparecem** na janela de 50 abertos; registrados só os que a
  API devolveu.
- **Repository em movimento durante a coleta.** `#3202` do `xcl` (19:08Z) e dez PRs do `NVIDIA` chegaram
  depois do primeiro snapshot; a lista foi lida duas vezes e o documento usa o segundo snapshot.
- **Nenhum subagente disponível** neste ambiente; os quatro repositórios foram processados em sequência.
- **Rate limit:** nenhum 403 (4.833 restantes no início, 4.590 no fim).
- **Escrita:** somente em `G:\gta6-port-knowledge`. Os 15 monitorados foram só leitura de API.

## Estado gravado

`runs: 31`, `updated: 2026-11-28T19:22:00Z`, `last_error` com os dois 404 (décima terceira hora),
**15 entradas** em `repos`. Cursores que avançaram:

| Repositório | `last_commit` | `last_pr_number` | `last_run` |
|---|---|---|---|
| `ivall/xcl` | `f2c0d36f` → **`10d27297`** (fila de commits zerada; `commit_cursor`/`head_at_run` removidos) | 3198 → **3200** | 2026-11-28T16:59:49Z (sem merge) |
| `R0Y00/Xbox2Windows` | `f2972840` → **`eeb52277`** | 3387 → **3395** | **2026-11-28T19:07:04Z** |
| `trustno1-coder/VIctory-NVIDIA` | `93008750` → **`32ecacfa`** | `null` → **1409** (piso abaixo de #1265 em backlog) | **2026-11-28T19:01:28Z** |
| `Marvisak/VIctory` | `63bcca43` (parado, `identical`) | 3775 → **3783** | 2026-11-27T13:20:45Z (sem merge) |
| `californiaAlmonds/OpenScarlett` | `48a98fcd` (parado) | 66 | 2026-11-27T13:05:00Z |
| `DexNom/VIctory` | `4ad86927` (parado) | 201 | 2026-11-26T04:15:00Z |
| `uiliannn/VItoriaBr` | `15779f81` (parado) | 134 | 2026-11-27T13:05:00Z |
| `fxceless/victory-fork` | `368967f8` (parado) | 1519 | 2026-11-28T01:48:22Z |
| `omonkeyz/xbox-emulator` | `a618736d` (parado) | 395 | 2026-11-26T04:15:00Z |
| `brcly/TuxBox` | `b8355fa2` (parado) | 407 (cursor parado; #408 na fila) | 2026-11-28T17:12:17Z |
| `holdmysocks/SixNG` | `cff3f31b` (parado) | 541 | 2026-11-28T17:41:36Z |
| `walkthestars/GTA6-Asset-Info` | `bad5bc21` (parado) | 0 | 2026-11-27T13:05:00Z |
| `stellarr-studio/kelvaroq` | `92606ea7` (parado) | `null` | 2026-11-27T16:52:39Z |

`backlog` com 6 entradas: `xcl` (#3202 acima do novo cursor), `Marvisak` (piso de ≥50 abertos, cursor
novo 3783), `NVIDIA` (piso abaixo de #1265), `TuxBox` (#408) e os dois bloqueados por 404.

## Push

A executar logo depois deste arquivo, na ordem do procedimento, com o resultado conferido por
`git ls-remote` e registrado no commit de fechamento desta execução:

```
git switch public
git add docs/updates state
git commit -m "docs(updates): 2026-11-28 19:00"
git push origin public:main
git switch main
git merge --no-ff public -m "merge updates 2026-11-28 19:00"
git push private main
```

Nenhum `git reset --hard`, nenhum merge de `main` na `public`. Se o push falhar, o estado **não** avança
e a próxima execução reprocessa o mesmo delta; nesse caso o cursor de `xcl` volta a `f2c0d36f` e os de
`Xbox2Windows` e `NVIDIA` voltam aos valores lidos no início da hora.
