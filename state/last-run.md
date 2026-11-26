# Execução de 2026-11-26 01:00

- **Data/hora local:** 2026-11-26 01:00 (America/Sao_Paulo) — início da coleta 2026-11-26T04:03Z
- **Execução:** 4 de N (22:00 foi o baseline; 23:00 a primeira documentação)
- **Estado na entrada:** `baselined=true`, `runs=3`, `backlog=["R0Y00/Xbox2Windows"]`, `updated=2026-11-26T03:16:53Z`

## Repositórios com delta (9)

| Repositório | Commits novos | PRs abertos | PRs mesclados (no cursor) | Documento |
|---|---|---|---|---|
| R0Y00/Xbox2Windows | 31 | 9 | 18 | `docs/updates/2026-11-26/01/Xbox2Windows.md` |
| fxceless/victory-fork | 9 | 1 | 1 | `docs/updates/2026-11-26/01/victory-fork.md` |
| holdmysocks/SixNG | 4 | 3 | 1 | `docs/updates/2026-11-26/01/SixNG.md` |
| avatarmanz/VIctory-7900XTX | 3 | 11 | 0 | `docs/updates/2026-11-26/01/VIctory-7900XTX.md` |
| brcly/TuxBox | 2 | 1 | 0 | `docs/updates/2026-11-26/01/TuxBox.md` |
| Marvisak/VIctory | 0 | 28 | 0 | `docs/updates/2026-11-26/01/VIctory.md` |
| ivall/xcl | 0 | 11 | 0 | `docs/updates/2026-11-26/01/xcl.md` |
| DexNom/VIctory | 0 | 4 | 0 | `docs/updates/2026-11-26/01/VIctory-DexNom.md` |
| Belobug/VIctory-Belobug-AMD | 0 | 2 | 0 | `docs/updates/2026-11-26/01/VIctory-Belobug-AMD.md` |

Sem delta: `uiliannn/VItoriaBr` (`main` em `e40fba44`), `californiaAlmonds/OpenScarlett` (`7220ce94`),
`omonkeyz/xbox-emulator` (`a618736d`) e `walkthestars/GTA6-Asset-Info` (`bad5bc21`). Nesses quatro a
varredura de abertos (números ≤ 111, 60, 394 e 0) e a de mesclados por `merged_at` não acharam nada.

## Números da hora

- **49 commits** documentados, de 49 na faixa — nenhum corte de teto. `Xbox2Windows` sozinho tem 31
  (abaixo do teto de 40), e os três commits que a hora anterior deixou no backlog (`404446e8`,
  `41ad992e`, `2a801192`) estão entre eles.
- **70 PRs abertos** (28 + 11 + 9 + 4 + 3 + 2 + 1 + 11 + 1) e **20 PRs mesclados** detectados no cursor
  (18 em `Xbox2Windows`, 1 em `SixNG`, 1 em `victory-fork`).
- **9 documentos** escritos, um por repositório com delta; **0 documentos de falha** (`_falhas.md` não
  existe nesta hora porque nenhuma coleta falhou).
- **Backlog:** esvaziado. `R0Y00/Xbox2Windows` foi consumido e os cursores `commit_cursor`/`head_at_run`
  foram removidos do estado, como manda a regra (só existem quando o teto corta a faixa).
- `runs` passou de 3 para 4.

## Método: o que mudou em relação ao procedimento escrito

1. **PR mesclado continuou por `mergedAt`** (mudança adotada na hora anterior, já registrada no próprio
   procedimento). O filtro por `numero > last_pr_number` teria perdido 20 merges: em `Xbox2Windows` os
   números vão de 1821 a 2726 com cursor de abertos em 2710; em `SixNG` o `#346` é maior que o cursor 345,
   mas `#333`/`#338`/`#345` não.
2. **A busca de mesclados falhou nesta hora e foi trocada.** `gh pr list --state merged --search
   "merged:>=<cursor>"` devolveu **lista vazia nos treze repositórios** (índice de busca atrasado: PRs
   mesclados há minutos ainda não estavam indexados). Troquei por
   `gh api 'repos/<owner>/<repo>/pulls?state=closed&sort=updated&direction=desc&per_page=100'` e filtrei
   `merged_at > last_run` no cliente. Só assim os 18 merges de `Xbox2Windows` apareceram. Registrei o
   desvio no `INDEX.md` e no documento de cada repositório afetado. Se a busca voltar a funcionar, as duas
   rotas concordam; se não voltar, a rota por API é a que deve ficar.
3. **Divergência entre commits e merges, registrada e não "consertada".** A janela de commits tem 31
   merges squash em `Xbox2Windows` e 4 em `SixNG`, mas a API de mesclados (com o cursor
   `2026-11-26T03:16:53Z`) devolveu 18 e 1. Os 16 restantes (`#333`/`#338`/`#345` de `SixNG` e 13 de
   `Xbox2Windows`) mesclaram **antes** do cursor `last_run` e **depois** do `last_commit` anterior: a
   execução de 00:00 gravou o estado às 03:16:53Z sem ter visto esses commits. Conferi cada `mergedAt` por
   `gh api repos/<owner>/<repo>/pulls/<n>` e a contagem fecha (18+13 = 31; 1+3 = 4). O documento de cada
   repositório traz a divergência na seção Confiança, com os números e as horas.
4. **Subagentes: nenhum.** A ferramenta de subagente não está disponível nesta sessão; os nove
   repositórios foram processados **em sequência**, como o passo 3 autoriza. O limite de 4 simultâneos não
   foi exercido e nada foi para o backlog por concorrência.
5. **Duas passadas de commit, e por quê.** O passo 6 publica `docs/updates state`; o passo 7 manda este
   arquivo conter "o resultado do push", que não existe antes do push. Segui o formato da execução
   anterior: commit de documentação e estado primeiro, publicado nas duas branches; este arquivo depois,
   num segundo commit com a mesma sequência. O primeiro commit é o que o passo 6 pede, com a mensagem
   exata `docs(updates): 2026-11-26 01:00`.

## Resultado do push

Sequência executada exatamente como o procedimento manda, sem merge de `main` na `public`:

1. `git -C G:\gta6-port-knowledge switch public` → **ok**.
2. `git add docs/updates state` → 11 arquivos (9 documentos novos, `INDEX.md`, `state/watch.json`).
3. `git commit -m "docs(updates): 2026-11-26 01:00"` → **ok**, commit `740bd83`, 11 arquivos,
   +671/−16.
4. `git push origin public:main` → **ok**, `a1bb51d..740bd83  public -> main`.
5. `git switch main` + `git merge --no-ff public -m "merge updates 2026-11-26 01:00"` → **ok**, merge
   `91f3d3d`.
6. `git push private main` → **ok**, `32d94e5..91f3d3d  main -> main`.

Verificado depois: `git ls-remote origin main` devolve `740bd83d…` e o `HEAD` local da `main` é
`91f3d3d`. Nenhum push falhou, então o estado **pôde** avançar (a regra "se o push falhar, não avance o
estado" não foi acionada).

Este arquivo entra num commit seguinte, pelo mesmo caminho: `public` → `push origin public:main` →
`main` → `merge --no-ff` → `push private main`.

## Achados que valem acompanhamento

- **`Xbox2Windows` `#2679` (`293e77f6`) reverte parte de um recurso próprio.** O commit remove a reescrita
  de descritores que o `#2579` tinha acabado de introduzir, e converte `X2W_D3D12_RELEASE_RETIRED_VIEWS`
  em diagnóstico legado. O `#2719`, aberto na mesma hora, propõe a fatia seguinte de `#2379` (slots 19/20/21
  na classe de aposentadoria). Se ele mesclar sem revisão, o mesmo tipo de regressão pode voltar.
- **`Xbox2Windows` `#2685`**: o corpo do PR descreve `probe_writable` (sonda de escrita sob SEH), mas o
  código que entrou usa `QueryWorkingSetEx` e mantém `VirtualQuery` como fallback — a revisão do próprio PR
  trocou a abordagem. Documento registra a divergência entre texto e diff.
- **`Xbox2Windows` `#2044`**: o corpo cita `x2w-startup-shader-9`, o código entrou em `-18`. É só número
  defasado no texto, mas qualquer auditoria que compare corpo do PR com código vai tropeçar nisso.
- **`AvatarManz`**: `#97` (UAV de textura posicionada escrevendo em stand-in → interiores sem luz indireta)
  e `#96` (exaustão de slots dinâmicos em heap com teto) descrevem defeitos que o catálogo `AvatarManz` não
  tem. `#96` é a mesma classe de `VIctory D-01` e `Belobug-AMD D-05`.
- **`DexNom` `#66`**: slot de heap sem stride lido como constant buffer → fault de página de GPU e remoção
  do dispositivo num dispatch indireto de culling, em RX 9070 (RDNA 4). É a falha mais grave da hora na
  fila aberta e não tem linha no catálogo.
- **`victory-fork` `4d0c6345`** porta `c1c32819e9` (upstream) e corrige, neste fork, o que o catálogo
  registra como `DexNom D-16`. Vale acrescentar a linha em `victory-fork` com `corrigido = 4d0c6345`.
- **`TuxBox` `ac4f0993`** sobe as vagas de compilação em jogo de `min(4, núcleos)` para
  `max(2, núcleos/2)` (máx. 32) e as de preparação de 8 para os núcleos. É a alavanca de `D-02`/`D-03`
  (tempo e memória do otimizador do DXC) e o commit não traz medição de memória com o novo padrão.
- **`SixNG` `#346`**: a segunda tabela `EDUP` (`core_restofworld`, 33 pares, texturas de gore) ficava fora
  do índice porque estava apendada ao XML de setup do pacote. Defeito novo, corrigido, sem linha no
  catálogo.
- **README de `AvatarManz`**: a linha `ZETTROX` entra no merge da `main` do GitHub (`a6ba0a8d`) e sai três
  minutos depois num commit direto (`b0cfe2ba`). A `main` local fica divergindo da remota nesse arquivo; a
  próxima sincronização traz a linha de volta e a rotina terá de documentar o mesmo vai-e-vem.

## O que não foi verificado

Nenhum dos treze repositórios monitorados foi escrito — somente leitura, como manda o limite. Nada foi
compilado nem executado. Os 49 diffs foram lidos pela API (lista de arquivos, contagens, trechos de patch
truncados em 3.000 caracteres por arquivo e 25 arquivos por commit), e os corpos dos 70 PRs abertos e dos
20 mesclados foram lidos; os **diffs dos PRs abertos não** — toda afirmação sobre o que um PR aberto faz é
declaração do autor. Os números de teste citados (320 palavras de readback, 1.024 pixels, 35.533 texturas,
17.464 pares de deduplicação, 2.800 draws dropados) são os que os PRs e commits declaram. Os quatro logs
de diagnóstico de `victory-fork` têm 30 a 37 mil linhas cada e não foram lidos integralmente; o resumo é o
que o `diagnostics/README.md` do próprio repositório publica.
