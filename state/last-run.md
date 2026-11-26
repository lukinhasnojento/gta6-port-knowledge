# Execução de 2026-11-25 23:00

- **Data/hora local:** 2026-11-25 23:00 (America/Sao_Paulo) — 2026-11-26T02:11:35Z
- **Execução:** 2 de N (a de 22:00 foi o baseline)
- **Estado:** `baselined=true` na entrada; baseline não foi refeito

## Repositórios com delta (10)

| Repositório | Commits novos | Documento |
|---|---|---|
| Marvisak/VIctory | 0 | `docs/updates/2026-11-25/23/VIctory.md` |
| ivall/xcl | 0 | `docs/updates/2026-11-25/23/xcl.md` |
| R0Y00/Xbox2Windows | 14 | `docs/updates/2026-11-25/23/Xbox2Windows.md` |
| Belobug/VIctory-Belobug-AMD | 0 | `docs/updates/2026-11-25/23/VIctory-Belobug-AMD.md` |
| DexNom/VIctory | 2 | `docs/updates/2026-11-25/23/VIctory-DexNom.md` |
| uiliannn/VItoriaBr | 1 | `docs/updates/2026-11-25/23/VItoriaBr.md` |
| avatarmanz/VIctory-7900XTX | 1 | `docs/updates/2026-11-25/23/VIctory-7900XTX.md` |
| fxceless/victory-fork | 19 | `docs/updates/2026-11-25/23/victory-fork.md` |
| brcly/TuxBox | 5 | `docs/updates/2026-11-25/23/TuxBox.md` |
| holdmysocks/SixNG | 6 | `docs/updates/2026-11-25/23/SixNG.md` |

Sem delta: `californiaAlmonds/OpenScarlett` (2 PRs abertos, todos ≤ 60), `omonkeyz/xbox-emulator`
(nenhum commit, nenhum PR > 394), `walkthestars/GTA6-Asset-Info` (5 commits, cursor no topo, nenhum
PR).

## Números da hora

- **48 commits** documentados (14 + 2 + 1 + 1 + 19 + 5 + 6).
- **96 PRs** documentados: 89 abertos (33 + 11 + 24 + 1 + 1 + 2 + 5 + 9 + 1 + 2) e 7 mesclados
  (`R0Y00` #2662/#2663, `SixNG` #329/#330/#331, `TuxBox` #200/#207).
- **10 documentos** escritos, um por repositório com delta.
- **0 documentos** de falha: nenhum repositório falhou na coleta.

## Decisões e desvios do procedimento

1. **Subagentes: nenhum.** A ferramenta de subagente não está disponível nesta sessão, então os dez
   repositórios foram processados **em sequência**, como o passo 3 autoriza. O limite de 4 subagentes
   simultâneos não chegou a ser exercido (0 usados) e por isso **nada foi para `backlog`** — não havia
   excedente de concorrência. `backlog` ficou `[]`.
2. **Teto de 40 commits por repositório:** não foi atingido. O maior foi `victory-fork`, com 19.
   Nenhum `commit_cursor` precisou ser gravado.
3. **Duas passadas de commit, e por quê.** O passo 6 manda commitar `docs/updates state` e publicar, e
   o passo 7 manda gravar este arquivo com "o resultado do push". O resultado do push não existe antes
   do push. Este arquivo é gravado **depois** de publicado o commit de documentação e entra num
   segundo commit, com a mesma sequência `public` → `push origin public:main` → `main` →
   `merge --no-ff` → `push private main`. É o mesmo formato da execução de baseline, que também tem
   dois commits (`34c1e75` documentação, `7accafe` resultado).
4. **Cursor de PR é cego para merge de PR antigo.** `holdmysocks/SixNG` mesclou `#326`, `#327` e
   `#328` nesta janela; os três têm número ≤ 328, o cursor do baseline, então a consulta
   `--state merged` filtrada por `numero > last_pr_number` não os enxerga. Eles ficaram documentados
   pelos commits `6c90bfd6`, `8c54c5b6` e `f2cce109`. O mesmo pode acontecer em qualquer hora: um PR
   aberto há tempo pode ser mesclado com número abaixo do cursor. **Sugestão para o procedimento:**
   filtrar PRs mesclados por `mergedAt > last_run` além do número, ou guardar `last_run` por
   repositório e usá-lo como corte.

## Resultado do push

Sequência executada exatamente como no procedimento, sem merge de `main` na `public`:

1. `git switch public` → commit `207527b` "docs(updates): 2026-11-25 23:00" (12 arquivos, +686/-17).
2. `git push origin public:main` → **ok**, `7accafe..207527b  public -> main`.
3. `git switch main` → `git merge --no-ff public -m "merge updates 2026-11-25 23:00"` → **ok**,
   merge `2fef96e`.
4. `git push private main` → **ok**, `6bcd1f7..2fef96e  main -> main`.

Este arquivo e a atualização de `updated` entram no commit seguinte, pelo mesmo caminho.

## Achados que valem acompanhamento

- **`victory-fork` `025b042a`** (alvo inteiro não faz blending no host) corrige o que o catálogo do
  repositório irmão registra como **DexNom D-01**. Falta a linha no catálogo `victory-fork`.
- **`victory-fork` `0fc6b066`** descarta uma hipótese do **D-16** do catálogo (perdas de device sem
  causa): os registros reescritos além dos slots de título do host são tabelas, não pista. D-16
  continua aberto.
- **`victory-fork` `36bdd515`** marca o switch `gather_by_copy` como **NOT PROVEN EXACT** após revisão
  independente, com uma forma de escrita não coberta.
- **`victory-fork` `32be36d5`** entrou com uma corrida de invalidação que o próprio repositório abre
  como PR #1245 três commits depois.
- **`TuxBox` `4af37a4c`/`04a560c1`** corrige import de memória recusado pelo RADV (o vkd3d-proton
  relata sucesso e a GPU lê outra memória) — defeito não catalogado, **candidato a D-30 de TuxBox** —
  mas a cópia-sombra admitidamente deixa as escritas da GPU invisíveis para a CPU.
- **`TuxBox` PR #1242 de `victory-fork`** afirma que a `main` (`03f0002`) está vermelha no Clippy em
  13 erros; se for verdade, é o **D-17** do catálogo `victory-fork` se repetindo.
- **`SixNG` `f2cce109`** volta a mexer na janela de presets do BC1 — região do **D-09**, aberto — e o
  arquivo continua declarando `Fitted, not extracted`. Ajuste empírico sem medição no diff.
- **`avatarmanz` `e2a28ec7`** é um commit de duas linhas que só acrescenta `ZETTROX` ao fim do README,
  sem contexto. Provável crédito que perdeu a frase.

## O que não foi verificado

Nenhum repositório monitorado foi escrito (somente leitura, como manda o limite). Nada foi compilado
ou executado. O corpo dos 89 PRs abertos foi lido, **os diffs não**: toda afirmação sobre o que um PR
faz é declaração do autor, marcada como `NAO VERIFICADO` em cada documento. Dos commits, os diffs
foram lidos pela API (arquivos, contagens, trechos de patch); os `.txt.gz` das runs 281 e 282 de
`victory-fork` aparecem como binários e não foram descompactados.
