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
  "updated": "2026-11-25T23:00:00Z",
  "baselined": true,
  "repos": {
    "Marvisak/VIctory": {
      "default_branch": "main",
      "last_commit": "70564ae",
      "last_pr_number": 1395,
      "last_run": "2026-11-25T23:00:00Z"
    }
  },
  "backlog": ["repo/que/ficou/para/depois"]
}
```

Regras de escrita do estado:
- `last_commit` só avança depois que os commits daquela faixa foram documentados.
- `last_pr_number` só avança para o maior PR **processado**, não para o maior existente.
- Nunca apague `repos`; adicione se um repositório novo entrar na lista.

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
