# Execução de 2026-11-26 00:00

- **Data/hora local:** 2026-11-26 00:00 (America/Sao_Paulo) — início da coleta 2026-11-26T03:03Z
- **Execução:** 3 de N (22:00 foi o baseline; 23:00 a primeira documentação)
- **Estado:** `baselined=true` na entrada; baseline não foi refeito

## Repositórios com delta (10)

| Repositório | Commits novos | PRs abertos | PRs mesclados | Documento |
|---|---|---|---|---|
| Marvisak/VIctory | 0 | 24 | 0 | `docs/updates/2026-11-26/00/VIctory.md` |
| ivall/xcl | 0 | 9 | 0 | `docs/updates/2026-11-26/00/xcl.md` |
| R0Y00/Xbox2Windows | 43 (40 documentados) | 14 | 18 | `docs/updates/2026-11-26/00/Xbox2Windows.md` |
| Belobug/VIctory-Belobug-AMD | 0 | 3 | 0 | `docs/updates/2026-11-26/00/VIctory-Belobug-AMD.md` |
| DexNom/VIctory | 0 | 2 | 0 | `docs/updates/2026-11-26/00/VIctory-DexNom.md` |
| uiliannn/VItoriaBr | 1 | 3 | 0 | `docs/updates/2026-11-26/00/VItoriaBr.md` |
| avatarmanz/VIctory-7900XTX | 0 | 7 | 0 | `docs/updates/2026-11-26/00/VIctory-7900XTX.md` |
| fxceless/victory-fork | 9 | 5 | 0 | `docs/updates/2026-11-26/00/victory-fork.md` |
| brcly/TuxBox | 4 (2 merges) | 1 | 2 | `docs/updates/2026-11-26/00/TuxBox.md` |
| holdmysocks/SixNG | 6 | 7 | 6 | `docs/updates/2026-11-26/00/SixNG.md` |

Sem delta: `californiaAlmonds/OpenScarlett` (`main` em `7220ce94`, 2 PRs abertos todos ≤ 60),
`omonkeyz/xbox-emulator` (topo em `a618736d`, 25 PRs abertos todos ≤ 394), `walkthestars/GTA6-Asset-Info`
(topo em `bad5bc21`, 1 PR aberto e zero mesclados).

## Números da hora

- **60 commits** documentados, de 63 na faixa real (40 de `Xbox2Windows` + 9 de `victory-fork` +
  6 de `SixNG` + 4 de `TuxBox` + 1 de `VItoriaBr`).
- **101 PRs** documentados: **75 abertos** (24 + 9 + 14 + 3 + 2 + 3 + 7 + 5 + 1 + 7) e **26 mesclados**
  (18 de `Xbox2Windows`, 6 de `SixNG`, 2 de `TuxBox`).
- **10 documentos** escritos, um por repositório com delta; **0 documentos de falha**.
- **3 commits em backlog:** `404446e8` (#2668), `41ad992e` (#2671) e `2a801192` (#2667), todos de
  `Xbox2Windows`, pelo teto de 40 commits.

## Decisões e desvios do procedimento

1. **Subagentes: nenhum.** A ferramenta de subagente não está disponível nesta sessão, então os dez
   repositórios com delta foram processados **em sequência**, como o passo 3 autoriza. O limite de 4
   simultâneos não foi exercido e nada foi para o `backlog` por concorrência. O `backlog` tem uma
   entrada — `R0Y00/Xbox2Windows` — apenas por causa do teto de commits.
2. **PRs mesclados passaram a ser detectados por `mergedAt`, não por `last_pr_number`.** Esta é a
   mudança de método mais importante da hora. Nos treze repositórios, a numeração de PR **não acompanha
   a ordem de merge**: `Marvisak/VIctory` tem `last_pr_number = 2978` e o maior PR mesclado devolvido é
   2493; `ivall/xcl` tem 2614 e o maior mesclado é 2466; `Xbox2Windows` mesclou 18 PRs nesta hora com
   números de 2630 a 2706, abaixo do cursor 2688. O filtro `numero > last_pr_number` do procedimento
   teria perdido **18 merges de `Xbox2Windows`, 6 de `SixNG` e 2 de `TuxBox`** — ou seja, todos. Usei
   `mergedAt > last_run` (02:11:35Z) e conferi que cada `mergeCommit.oid` corresponde ao commit
   documentado. O `last_pr_number` continua sendo o cursor de PRs **abertos**.
3. **Teto de 40 commits em `Xbox2Windows`.** A faixa tinha 43. Documentei os 40 mais antigos
   (`c1e4c709` … `d5ea71b2`) e gravei `last_commit = d5ea71b2` (mais `commit_cursor` e `head_at_run` em
   `state/watch.json`), de modo que a próxima execução comece exatamente nos três que faltam. Não
   avancei o cursor até o topo para não perder commit sem documento.
4. **Divergência não resolvida em `Xbox2Windows`: 44 merges contra 43 commits.** A busca
   `search/issues?is:pr+is:merged+merged:>=2026-11-26T02:11:35Z` conta **44** PRs mesclados, e o
   `compare` `572723f9...2a801192` tem **43** commits. Um PR mesclado na janela não aparece como commit
   próprio na lista (merge de branch, ou squash ausente). Não identifiquei qual pela API de commits e
   registrei a divergência na seção Confiança do documento, em vez de inventar a linha que falta.
5. **Duas passadas de commit, e por quê.** O passo 6 manda publicar `docs/updates state` e o passo 7
   manda gravar este arquivo com "o resultado do push", que não existe antes do push. Segui o mesmo
   formato da execução anterior: primeiro o commit de documentação e estado, publicado nas duas
   branches; depois este arquivo, num segundo commit com a mesma sequência `public` → `push origin
   public:main` → `main` → `merge --no-ff` → `push private main`.
6. **Commits de merge contados como commits.** Em `TuxBox` a faixa tem 4 commits, mas dois são os merges
   `c2d7bbc8` (#206) e `bc4ba79f` (#208) e dois são os commits de branch (`5cd728f8`, `ff9d54cf`). Eles
   aparecem na tabela de commits e os PRs correspondentes na tabela de mesclados, sem duplicar a
   descrição do conteúdo.

## Resultado do push

Sequência executada sem merge de `main` na `public`:

1. `git -C G:\gta6-port-knowledge switch public` → **ok**.
2. `git add docs/updates state` → 12 arquivos (10 documentos novos, `INDEX.md`, `state/watch.json`).
3. `git commit -m "docs(updates): 2026-11-26 00:00"` → **ok**, commit `97834af`, 12 arquivos, +631/-17.
4. `git push origin public:main` → **ok**, `24a5ac1..97834af  public -> main`.
5. `git switch main` + `git merge --no-ff public -m "merge updates 2026-11-26 00:00"` → **ok**, merge
   `2d3c449`.
6. `git push private main` → **ok**, `1f77d9c..2d3c449  main -> main`.

Este arquivo entra no commit seguinte, pelo mesmo caminho (public → main público → merge na main →
main privado).

## Achados que valem acompanhamento

- **`Xbox2Windows` `c452959f` (#2706)** remove quatro linhas que declaravam `bound_targets[8]` /
  `bound_depth` duas vezes no `Pass`; as duplicatas vieram da integração de **#2448 (`a3b923e1`)** e
  **#2417 (`73621923`)** e derrubavam a build MSVC (C2086) antes dos testes. Defeito introduzido e
  corrigido em ~40 minutos. O PR **#2707** propõe a mesma correção e continua aberto contra uma `main`
  já corrigida — vai precisar de rebase.
- **`Xbox2Windows` #2710** diz que `startup_compute_dispatch_texture_*` (da #2532) **falha na `main`
  atual** em `tests/startup_compute_dispatch_texture_check.py:101`, por causa da reescrita de recusa de
  view do #2649. Se for verdade, a `main` está vermelha nesse teste agora.
- **`Xbox2Windows` #2701 e #2707** declaram a `main` quebrada por motivos diferentes (logs de Windows de
  #2454 e #2704 mostrando as declarações duplicadas antes dos testes). Os dois achados apontam para a
  mesma janela de integração.
- **`TuxBox` #208** é cobertura de teste do caminho que o `#168` refletiu (roots raster com duas tabelas
  de descritor), não correção do `D-22`. O catálogo não tem linha para "faltava cenário de duas
  tabelas" — candidato a item novo.
- **`SixNG` b7a9ad07 (#337)** volta à tabela BC1 do BCPack, que é a região de `D-09`. Aqui a régua é
  declarada (checagem de mip sobre 35.533 texturas: abaixo de 20 dB 65 → 60, nenhuma pior), diferente do
  ajuste empírico que originou `D-09`.
- **`victory-fork 895a23bc`** muda o que a passagem de buffer-views revisita e está atrás de switch
  desligado; o autor declara "compilado aqui; não rodado em jogo". Se o switch virar padrão sem medição,
  é a classe de mudança que já produziu perda de device neste fork (`D-16`, aberto).
- **`xcl` #2624** (83 arquivos, +12.712/-687) é a maior proposta da fila e mistura upstream, preparação
  de shader em processos worker e mudanças em `dxil/`/`recomp/`. Quando mesclar, merece documento só
  dele.

## O que não foi verificado

Nenhum dos treze repositórios monitorados foi escrito — somente leitura, como manda o limite. Nada foi
compilado nem executado. Os 40 diffs de `Xbox2Windows`, os 9 de `victory-fork`, os 6 de `SixNG`, os 4 de
`TuxBox` e o 1 de `VItoriaBr` foram lidos pela API (arquivos, contagens, trechos de patch), mas os
números de cobertura que os testes declaram (1.024 words, 320 words, 35.533 texturas, 17.431 pares) são
declaração dos próprios testes. Os corpos dos 75 PRs abertos foram lidos, **os diffs não**: toda
afirmação sobre o que um PR faz é declaração do autor. Os cinco logs de diagnóstico de `victory-fork`
têm 25 a 37 mil linhas cada e **não** foram lidos integralmente; o resumo é o que o próprio
`diagnostics/README.md` publica.
