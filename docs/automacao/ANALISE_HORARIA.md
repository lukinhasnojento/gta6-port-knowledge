# Análise horária automática

Rotina agendada que, a cada hora, verifica os treze repositórios monitorados, detecta commits e pull
requests novos ou recém-mesclados, e documenta o que mudou — em arquivos versionados, com hash em cada
afirmação.

Criada em 2026-11-25. Executa no primeiro minuto de cada hora.

## O que cada execução faz

1. **Lê o estado** em `state/watch.json`: último commit visto e último número de PR visto por
   repositório.
2. **Coleta o delta** por repositório:
   - `git fetch` e lista de commits novos na branch default
   - PRs novos (`gh pr list --state open`) e PRs mesclados desde a última execução
   - issues abertas novas, se houver
3. **Se não há delta em nenhum repositório**, atualiza apenas o campo `updated` do estado e termina.
   Não escreve documentação vazia.
4. **Se há delta**, distribui o trabalho em subagentes — um por repositório com mudança, no máximo 4
   por execução. Repositórios que passam do limite ficam para a hora seguinte, registrados em
   `state/watch.json` no campo `backlog`.
5. **Cada subagente** escreve `docs/updates/AAAA-MM-DD/HH/<repo>.md` com:
   - commits novos: hash, autor, assunto, e o que o diff faz de fato
   - PRs abertos: número, título, o que propõem, e se cobrem algum defeito do catálogo
   - PRs mesclados: número, hash do merge, e o que mudou em relação ao que o PR prometia
   - cruzamento com `catalogo/defeitos.csv`: um commit novo corrige algum defeito conhecido? abre
     algum novo?
6. **Consolida** a linha do dia em `docs/updates/INDEX.md` e remove o `backlog` consumido.
7. **Atualiza o estado** com os SHAs e números de PR efetivamente documentados.
8. **Publica** nas duas branches (ver seção Publicação).

## Primeira execução é baseline

Na primeira execução com estado vazio, a rotina **não documenta** nada: apenas grava o estado atual
(último commit e último PR de cada repositório) e termina. Isso evita gerar documentação para os 9.209
commits que já foram analisados nos doze dossiês.

## Estado: `state/watch.json`

```json
{
  "updated": "2026-11-26T03:16:53Z",
  "baselined": true,
  "repos": {
    "Marvisak/VIctory": {
      "default_branch": "main",
      "last_commit": "63bcca43c6a00a758510398fa43096ce7e4f7df0",
      "last_pr_number": 3002,
      "last_run": "2026-11-26T03:16:53Z"
    },
    "R0Y00/Xbox2Windows": {
      "default_branch": "main",
      "last_commit": "d5ea71b2",
      "commit_cursor": "d5ea71b2",
      "head_at_run": "2a801192",
      "last_pr_number": 2706,
      "last_run": "2026-11-26T03:16:53Z"
    }
  },
  "backlog": ["R0Y00/Xbox2Windows"],
  "runs": 3,
  "last_error": null
}
```

Regras de escrita do estado:
- `last_commit` (SHA completo) é o cursor de commits: só avança até o último commit **documentado**.
- `commit_cursor` e `head_at_run` só aparecem quando o teto de 40 commits corta a faixa. O próximo
  ciclo começa em `commit_cursor`, não em `head_at_run`.
- `last_pr_number` é o cursor de **PRs abertos** apenas. Nunca use para PR mesclado.
- `last_run` é o cursor de **PRs mesclados**, comparado com `mergedAt`.
- Nunca apague `repos`; adicione se um repositório novo entrar na lista.

### Quatro regras que nasceram de falhas reais da rotina

Estas quatro foram escritas depois de a rotina errar, não por precaução teórica.

**1. Releia `default_branch` a cada execução; nunca confie no valor gravado.**
`Belobug/VIctory-Belobug-AMD` trocou a branch default de `main` para `a-publier`. O estado continuou
apontando `main`, e por isso o `compare` passou a devolver `diverged` com 638 commits à frente e 550
atrás — número sem sentido que manteve o repositório no backlog indefinidamente. A consulta é barata:

```
gh api repos/<owner>/<repo> --jq .default_branch
```

Se divergir do estado, corrija o estado **antes** de calcular o delta.

**2. Nunca reduza a lista de `repos`.** Se o número de entradas for menor que o do estado lido,
aborte a escrita e registre o erro. A rotina já apagou o `stellarr-studio/kelvaroq` que havia sido
adicionado à mão, porque reconstruiu o estado a partir do próprio conjunto de trabalho em vez de
mesclar.

**3. `compare` com `status = "diverged"` não é faixa de commits.** Quando o histórico foi reescrito,
`ahead_by` conta tudo que não está no ancestral comum e não tem relação com o que falta documentar.
Nesse caso: grave o novo `head` como cursor, documente apenas os últimos `N` commits (pelo teto) com
uma linha dizendo que o histórico foi reescrito, e siga.

**4. Cursor nulo significa baseline daquele repositório, não histórico inteiro.** Ao encontrar
`last_commit: null`, grave o head atual e não documente nada naquela execução. Sem isso, um
repositório novo entra com 50 commits de um dia documentados como se fossem novidade do dia.

### Por que PR mesclado não usa número

A numeração de PR não acompanha a ordem de merge nos repositórios monitorados. Medição de
2026-11-26: `Xbox2Windows` mesclou 18 PRs com números de 2630 a 2706 enquanto o cursor de abertos
estava em 2688; `VIctory` tinha `last_pr_number` 2978 e o maior PR mesclado 2493. Filtrar por
`numero > last_pr_number` perdeu **todos os 26 merges** daquela hora.

Regra correta:

```
gh pr list --repo <owner>/<repo> --state merged --limit 50 --search "merged:>=<last_run>"
```

e conferir cada `mergeCommit.oid` contra o commit documentado. Se a contagem de PRs mesclados não
bater com a de commits da janela, registre a divergência na seção Confiança do documento — não
invente a linha que falta.

## Repositórios monitorados

Treze, com o nome exato usado em `state/watch.json` e na API. A coluna de nome de arquivo importa:
as análises e os comparativos usam um rótulo curto, e as atualizações horárias usam o nome do
repositório. Não invente nomes — siga esta tabela.

| # | Repositório | Branch | Arquivo em `docs/analises/` | Arquivo em `docs/comparativo/` | Arquivo em `docs/updates/` |
|---|---|---|---|---|---|
| 1 | `Marvisak/VIctory` | main | `VIctory.md` | `VIctory.md` | `VIctory.md` |
| 2 | `ivall/xcl` | main | `XCL.md` | `XCL.md` | `xcl.md` |
| 3 | `R0Y00/Xbox2Windows` | main | `Xbox2Windows.md` | `Xbox2Windows.md` | `Xbox2Windows.md` |
| 4 | `californiaAlmonds/OpenScarlett` | main | `OpenScarlett.md` | `OpenScarlett.md` | `OpenScarlett.md` |
| 5 | `Belobug/VIctory-Belobug-AMD` | main | `Belobug-AMD.md` | `Belobug-AMD.md` | `VIctory-Belobug-AMD.md` |
| 6 | `DexNom/VIctory` | main | `DexNom.md` | `DexNom.md` | `VIctory-DexNom.md` |
| 7 | `uiliannn/VItoriaBr` | main | `VItoriaBr.md` | `VItoriaBr.md` | `VItoriaBr.md` |
| 8 | `avatarmanz/VIctory-7900XTX` | main | `AvatarManz.md` | `AvatarManz.md` | `VIctory-7900XTX.md` |
| 9 | `fxceless/victory-fork` | main | `victory-fork.md` | `victory-fork.md` | `victory-fork.md` |
| 10 | `omonkeyz/xbox-emulator` | main | `xbox-emulator.md` | `xbox-emulator.md` | `xbox-emulator.md` |
| 11 | `brcly/TuxBox` | main | `TuxBox.md` | `TuxBox.md` | `TuxBox.md` |
| 12 | `holdmysocks/SixNG` | **master** | `SixNG.md` | `SixNG.md` | `SixNG.md` |
| 13 | `walkthestars/GTA6-Asset-Info` | main | `GTA6-Asset-Info.md` | `GTA6-Asset-Info.md` | `GTA6-Asset-Info.md` |

O repositório `ivall/xcl` responde também por `xcl-dev/xcl` (transferência); use `ivall/xcl`, que
redireciona.
normalmente.

### Verificar cobertura

Antes de concluir que falta um repositório, rode esta checagem — ela compara `state/watch.json` com os
arquivos em disco:

```bash
cd gta6-port-knowledge
jq -r '.repos | keys[]' state/watch.json | while read repo; do
  printf '%-36s watch=sim' "$repo"
  for d in docs/analises docs/comparativo; do
    n=$(ls "$d" 2>/dev/null | wc -l)
    printf ' %s=%s' "$d" "$n"
  done
  echo
done
```

A saída esperada é 13 linhas, uma por repositório, todas com `watch=sim`.

## Formato de cada documento de atualização

```markdown
# <repo> — atualização de AAAA-MM-DD HH:00

## Resumo
Uma a três frases.

## Commits novos (N)
| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|

## Pull requests abertos (N)
| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|

## Pull requests mesclados (N)
| # | Título | Merge | O que entrou de fato |
|---|---|---|---|

## Cruzamento com o catálogo
- Corrige D-xx? (hash)
- Abre defeito novo? (hash, arquivo, linha)

## Confiança
O que foi verificado e o que não foi.
```

## Publicação

O acervo tem duas branches com um commit de diferença:

- `public` → repositório público (sem o prompt de implementação)
- `main` → repositório privado (`public` + o prompt)

A rotina grava **na branch `public`** e depois mescla `public` na `main`. Nunca o contrário: mesclar
`main` na `public` vazaria o prompt.

```
git switch public
git add docs/updates state
git commit -m "docs(updates): <data> <hora>"
git push origin public:main
git switch main && git merge --no-ff public -m "merge updates <data> <hora>"
git push private main
```

Se o push falhar, a rotina **não** avança o estado — a próxima execução reprocessa o mesmo delta.

## Limites e falhas

O desenho original era "documentar tudo a cada hora". Ele não se sustenta: com 14 repositórios e
repositórios que recebem 80 commits por hora, o teto de 4 repositórios por execução nunca alcança a
fila. Medição de 2026-11-27: 714 commits pendentes, 638 deles em um único repositório.

O desenho correto é **triagem caro, documentação barata**:

**Toda execução, barato, para os 14:**
- `gh api repos/<owner>/<repo> --jq .default_branch` (e corrige o estado se mudou)
- head atual e contagem de commits desde o cursor via `compare`
- PRs abertos e mesclados desde `last_run`
- grava uma linha de triagem por repositório em `docs/updates/AAAA-MM-DD/HH/_triagem.md`

**Deep dive só quando houver sinal**, até 4 repositórios por execução, priorizando:
1. commit ou PR mesclado que casa com um defeito do `catalogo/defeitos.csv`
2. commit com sinal de correção, reversão ou regressão
3. PR aberto que toca arquivo de área listada na tabela de pistas
4. nenhum dos acima e atraso maior que 200 commits

Repositórios que não entram no deep dive ficam com o cursor **parado** — a triagem registra o atraso e
o próximo ciclo decide. O importante é que a fila pare de crescer: a triagem cobre os 14 sempre, e o
deep dive é seletivo por mérito, não por ordem de chegada.

- **Máximo 4 deep dives por execução.** O que não entrar fica na triagem, com o atraso medido.
- **Máximo de 40 commits documentados por repositório por deep dive.**
- **Falha em um repositório não interrompe os outros.** Registre em
  `docs/updates/AAAA-MM-DD/HH/_falhas.md` e siga.
- **Rate limit do GitHub**: se `gh api` retornar 403 por limite, pare e aguarde a próxima hora. Não
  faça retry em laço.
- **Nada de escrita em repositório de terceiros.** Somente leitura neles.
- **Nunca reduza `repos` no estado.** Se a contagem cair, aborte a escrita.

## Como desligar

A rotina é uma automação do OpenCode. Para pausar: `automation` → listar → pausar pelo id. Para
remover de vez: `automation` → delete. Nada no repositório depende dela para estar correto — os
documentos gerados ficam.
