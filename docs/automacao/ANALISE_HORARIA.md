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

Camada e portes: `Marvisak/VIctory`, `ivall/xcl`, `R0Y00/Xbox2Windows`, `californiaAlmonds/OpenScarlett`.

Forks do VIctory: `Belobug/VIctory-Belobug-AMD`, `DexNom/VIctory`, `uiliannn/VItoriaBr`,
`avatarmanz/VIctory-7900XTX`, `fxceless/victory-fork`.

Independentes: `omonkeyz/xbox-emulator`, `brcly/TuxBox`.

Ferramentas: `holdmysocks/SixNG`, `walkthestars/GTA6-Asset-Info`.

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

- **Máximo 4 subagentes por execução.** Excedente vai para `backlog`.
- **Máximo de 40 commits documentados por repositório por execução.** O restante fica no backlog com
  a marca `commit_cursor`.
- **Falha em um repositório não interrompe os outros.** Registre o erro em
  `docs/updates/AAAA-MM-DD/HH/_falhas.md` e siga.
- **Rate limit do GitHub**: se `gh api` retornar 403 por limite, pare e aguarde a próxima hora. Não
  faça retry em laço.
- **Nada de escrita em repositório de terceiros.** Somente leitura neles.

## Como desligar

A rotina é uma automação do OpenCode. Para pausar: `automation` → listar → pausar pelo id. Para
remover de vez: `automation` → delete. Nada no repositório depende dela para estar correto — os
documentos gerados ficam.
