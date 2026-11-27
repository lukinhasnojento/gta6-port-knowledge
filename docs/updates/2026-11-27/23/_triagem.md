# Triagem de 2026-11-27 23:00

Coleta entre 23:05 e 23:20 UTC real — ~20:05–20:20 no relógio do GitHub, que segue cerca de 3 h atrás
e é o relógio em que `last_run` é gravado. Cursor de commits de cada repositório = `last_commit` do
`state/watch.json` lido no início (`runs: 20`). 14 entradas em `repos` no início e no fim; nenhuma
removida.

## Sanidade (passo 1)

- `gh api repos/<o>/<r> --jq .default_branch` pedido nos 14. **Dois sem resposta**: `Belobug/VIctory-Belobug-AMD`
  e `avatarmanz/VIctory-7900XTX` devolvem HTTP 404 em `repos/` (terceira hora seguida) — cursor parado,
  triagem NAO VERIFICADO.
- `default_branch` idêntico ao gravado nos 12 restantes, inclusive `master` (`holdmysocks/SixNG`) e
  `a-publier` (`Belobug`, gravado antes do 404).
- Nenhum `last_commit` nulo: não houve baseline nesta execução.
- Nenhum `compare` devolveu `diverged` (12 medições: 4 `ahead`, 8 `identical`).

## Triagem (passo 2)

Janela de PRs abertos: número maior que `last_pr_number`. Janela de mesclados: `merged_at` maior que
`last_run`, por `repos/<o>/<r>/pulls?state=closed&sort=updated` paginado até `updated_at` cair abaixo
do cursor (a busca `merged:>=` não é usada — perde merges em repositório transferido).

| repo | commits atrás | PRs abertos | PRs mesclados | sinal | prioridade |
|---|---|---|---|---|---|
| `Marvisak/VIctory` | 0 | 67 (100 na 1ª página) | 0 | SIM — critério 3, lote `Shaders:`/`GPU:`; `#3538` inclui opções de tradução na chave do cache | média, sem vaga |
| `ivall/xcl` | 0 | 22 (100) | 0 | SIM — critério 3, bibliotecas nativas de pipeline (`#3051`–`#3057`) | média, sem vaga |
| `R0Y00/Xbox2Windows` | 5 | 5 (100) | 5 | SIM — critério 2, `startup:` e lowering de `f16`/`s_bitreplicate` | **deep dive** |
| `californiaAlmonds/OpenScarlett` | 0 | 0 (1 total) | 0 | NAO | — |
| `Belobug/VIctory-Belobug-AMD` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (HTTP 404) | bloqueado |
| `DexNom/VIctory` | 1 | 14 | 0 | SIM — critério 2 e 1: 13 PRs `restore`/`T-00xx` do que `570a941` derrubou; D-10/D-11/D-12 seguem abertos | **deep dive** |
| `uiliannn/VItoriaBr` | 0 | 1 (75) | 0 | SIM fraco — critério 3, `#132` validador DXIL | baixa |
| `avatarmanz/VIctory-7900XTX` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (HTTP 404) | bloqueado |
| `fxceless/victory-fork` | 0 | 2 (100) | 0 | SIM — critério 3, dois backports de Vice-Tay | baixa |
| `omonkeyz/xbox-emulator` | 0 | 1 (25) | 0 | SIM fraco — critério 3, `#395` validação do bloco de relocação PE | baixa |
| `brcly/TuxBox` | 21 | 1 (3 total) | 4 | SIM — critério 2, 12 commits de conteúdo em `xmem`/`xshader`/`diagnostics` + 4 merges | **deep dive** |
| `holdmysocks/SixNG` | 9 | 1 (4 total) | 9 | SIM — critério 2, culling, pele, cabelo, MSAA | **deep dive** |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 | 0 | NAO | — |
| `stellarr-studio/kelvaroq` | 0 | 0 | 0 | NAO | — |

## Seleção do deep dive (passo 3)

Vagas: 4. Os quatro repositórios com commit novo na janela venceram pelo critério 2 (correção,
regressão ou reversão) e ocuparam as quatro vagas; `TuxBox` também casa o critério 1 (D-24, resolução de
merge que apaga lowering — os merges da janela tocam exatamente `compute.rs` com conflito) e `DexNom` o
critério 1 (D-10/D-11/D-12 abertos por reversão).

`Marvisak` (67 PRs abertos) e `xcl` (22) perdem a vaga porque o critério 3 vem depois do 2 e as vagas
acabaram. Os cursores deles ficam parados; a triagem registra o atraso.

## Números da janela

- **36 commits** acima dos cursores: `TuxBox` 21, `SixNG` 9, `Xbox2Windows` 5, `DexNom` 1.
- **115 PRs abertos** acima dos cursores de número.
- **18 PRs mesclados** com `merged_at` na janela: `SixNG` 9, `Xbox2Windows` 5, `TuxBox` 4.
- Contagem de commits × merges bate nos quatro documentados (ver cada documento; `TuxBox` tem merges de
  dois pais na faixa).
