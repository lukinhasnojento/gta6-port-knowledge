# Triagem de 2026-11-27 18:00

Coleta dos 14 repositórios de `state/watch.json`, sem documentação por diff. Cursor de commits:
`compare/<last_commit>...<head>`, campo `ahead_by`. Cursor de mesclados: `last_run` comparado com
`merged_at`, lido por `pulls?state=closed&sort=updated&direction=desc` paginando até o `updated_at`
mais antigo cair abaixo de `last_run` (a busca `--search "merged:>="` continua descartada). Janela de
PRs abertos: `pulls?state=open&per_page=100` ordenada por `updated` desc; "abertos" abaixo conta só
os que têm número acima de `last_pr_number`.

Nenhum `compare` devolveu `diverged`. `default_branch` reconferido nos 14 por
`gh api repos/<owner>/<repo> --jq .default_branch` e igual ao gravado — inclusive
`Belobug/VIctory-Belobug-AMD` em `a-publier` e `holdmysocks/SixNG` em `master`. 14 entradas em
`repos` no início e no fim da coleta; nenhuma removida.

| repo | commits atrás | PRs abertos | PRs mesclados | sinal | prioridade |
|---|---|---|---|---|---|
| `R0Y00/Xbox2Windows` | 11 | 14 (#3073 a #3097) | 4 (#3057, #3059, #3065, #3067) | SIM | 1 |
| `brcly/TuxBox` | 2 | 2 (#310, #311) | 1 (#309) | SIM | 1 |
| `ivall/xcl` | 2 | 10 (#3023 a #3032; #3023, #3024 e #3025 com base `amd-dev`) | 1 (#1824) | SIM | 2 |
| `fxceless/victory-fork` | 1 | 13 (#1469 a #1483) | 0 | SIM | 2 |
| `holdmysocks/SixNG` | 1 | 0 (#472 aberto abaixo do cursor) | 1 (#475) | SIM (área de render, sem termo de correção) | 4 |
| `Belobug/VIctory-Belobug-AMD` | 0 | 61 acima de #456 na janela dos 100 mais recentes | 0 | SIM | 3 |
| `avatarmanz/VIctory-7900XTX` | 0 | 39 (#420 a #470) | 0 | SIM | 3 |
| `Marvisak/VIctory` | 0 | 8 (#3522, #3523, #3525 a #3529, #3531) | 0 | SIM | 3 |
| `omonkeyz/xbox-emulator` | 0 | 1 (#395) | 0 | SIM | 4 |
| `DexNom/VIctory` | 0 | 0 (100 abertos, todos até #134) | 0 | NAO | — |
| `californiaAlmonds/OpenScarlett` | 0 | 0 (#60 abaixo do cursor) | 0 | NAO | — |
| `uiliannn/VItoriaBr` | 0 | 0 (74 abertos, todos até #126) | 0 | NAO | — |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 | 0 | NAO | — |
| `stellarr-studio/kelvaroq` | 0 | 0 | 0 | NAO | — |

## Deep dives escolhidos (4)

1. **`R0Y00/Xbox2Windows`** — critério 1/2: 11 commits e 4 merges reais, e `246857e459` (#3058)
   corrige a conversão U8 empacotada que `#2997` introduziu (regressão medida em driver NVIDIA, #3052).
2. **`brcly/TuxBox`** — critério 1/2: `b46c3db9` (#309) fixa a opção do DXC no teste de fallback `-O0`,
   área de D-02/D-03; os dois PRs abertos (#310 IDXEN/stride, #311 splash) são áreas do catálogo.
3. **`ivall/xcl`** — critério 3: #3023 corrige `V_CMP_LG_F32`/`V_DIV_*_F32` no tradutor direto RDNA4,
   mesma classe de defeito do catálogo `VIctory D-20` (`v_cmp_lg` como `fcmp one`); 10 abertos e 1 merge.
4. **`fxceless/victory-fork`** — critério 2: um único commit que substitui a árvore inteira pela de
   `trustno1-coder/VIctory-NVIDIA` em `959dcaf` (284 arquivos). Risco de regressão dos defeitos D-01 a
   D-06 que o próprio fork havia corrigido na linha antiga.

Ficam fora, com cursor parado: `Belobug/VIctory-Belobug-AMD` (61 abertos, sem commit), `avatarmanz`
(39 abertos), `Marvisak` (8), `SixNG` (1 commit de performance em `render3d`), `omonkeyz` (#395). O
critério 3 empata `xcl` com `Belobug`; o desempate foi o delta real (2 commits + 1 merge contra zero
commit), e `Belobug` entra primeiro na próxima hora.
