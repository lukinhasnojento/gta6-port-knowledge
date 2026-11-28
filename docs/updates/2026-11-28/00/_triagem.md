# Triagem de 2026-11-28 00:00

Coleta entre 00:07 e 00:17 UTC real — ~21:07–21:17 no relógio do GitHub, que segue cerca de 3 h atrás
e é o relógio em que `last_run` é gravado. Cursor de commits de cada repositório = `last_commit` do
`state/watch.json` lido no início (`runs: 21`). 14 entradas em `repos` no início e no fim; nenhuma
removida.

## Sanidade (passo 1)

- `gh api repos/<o>/<r> --jq .default_branch` pedido nos 14. **Dois sem resposta**: `Belobug/VIctory-Belobug-AMD`
  e `avatarmanz/VIctory-7900XTX` devolvem HTTP 404 em `repos/` (quarta hora seguida), e `users/<login>`
  também devolve 404 nos dois — cursor parado, triagem NAO VERIFICADO.
- `default_branch` idêntico ao gravado nos 12 restantes, inclusive `master` (`holdmysocks/SixNG`);
  `Belobug` continua com `a-publier` gravado de antes do 404, inverificável.
- Nenhum `last_commit` nulo: não houve baseline nesta execução.
- Nenhum `compare` devolveu `diverged` (12 medições: 4 `ahead`, 8 `identical`).
- A coleta inicial de `compare` falhou nos 12 por `\n` no SHA devolvido por `--jq .sha` (URL inválida);
  refeita com o SHA aparado. Nenhum número desta triagem vem da primeira tentativa.

## Triagem (passo 2)

Janela de PRs abertos: número maior que `last_pr_number`, contado sobre a primeira página de 100
(`per_page=100`, `sort=created`, `direction=desc`), com cobertura provada só quando o menor número da
página é menor que o cursor. Janela de mesclados: `merged_at` maior que `last_run`, por
`repos/<o>/<r>/pulls?state=closed&sort=updated` paginado até `updated_at` cair abaixo do cursor (a busca
`merged:>=` não é usada — perde merges em repositório transferido).

| repo | commits atrás | PRs abertos | PRs mesclados | sinal | prioridade |
|---|---|---|---|---|---|
| `Marvisak/VIctory` | 0 | 76 (100 na 1ª página, mín. 3487 < cursor 3521) | 0 | SIM — critério 3, `GPU:`/`Shaders:`/`CLI:`; `#3611` slots dinâmicos devolvidos ao heap de origem, `#3609` arredondamento de `v_exp_f16`, `#3595` switch do cache do driver | média, sem vaga |
| `ivall/xcl` | 0 | 24 (100, mín. 2953 < 3042) | 0 | SIM — critério 3, pilha de bibliotecas nativas de pipeline (`#3051`–`#3057`) e `D3D12.X` | média, sem vaga |
| `R0Y00/Xbox2Windows` | **12** | 14 (100, mín. 1939 < 3169) | 12 | SIM — critério 2, lowerings novos no compute direto e `MIMG` falso-recusado; fecha refusals | **deep dive** |
| `californiaAlmonds/OpenScarlett` | 0 | 0 (1 total) | 0 | NAO | — |
| `Belobug/VIctory-Belobug-AMD` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (HTTP 404) | bloqueado |
| `DexNom/VIctory` | 0 | 0 (100, máx. 148 = cursor) | 0 | NAO — a fila restore dos 14 PRs está toda abaixo do cursor; nenhum PR novo | — |
| `uiliannn/VItoriaBr` | 0 | 1 (75 total) | 0 | SIM fraco — critério 3, `#132` validador DXIL estrito | baixa |
| `avatarmanz/VIctory-7900XTX` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (HTTP 404) | bloqueado |
| `fxceless/victory-fork` | **1** | 2 (100, mín. 1266 < 1493) | 0 | SIM — critério 2, orçamento de VRAM das texturas (`2eade68c`) | **deep dive** |
| `omonkeyz/xbox-emulator` | 0 | 1 (25 total) | 0 | SIM fraco — critério 3, `#395` validação do bloco de relocação PE | baixa |
| `brcly/TuxBox` | **2** | 3 (6 total) | 0 | SIM — critério 2, harness que compara tradução contra shaders gravados | **deep dive** |
| `holdmysocks/SixNG` | **2** | 2 (5 total) | 2 | SIM — critério 2, `#503` vidro de lente com blend dual-source, `#504` adesivo sem textura | **deep dive** |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 | 0 | NAO | — |
| `stellarr-studio/kelvaroq` | 0 | 0 | 0 | NAO | — |

## Seleção do deep dive (passo 3)

Vagas: 4. Exatamente quatro repositórios tiveram commit na janela e todos venceram pelo critério 2
(correção, regressão ou reversão): `Xbox2Windows`, `fxceless/victory-fork`, `TuxBox` e `SixNG`. As quatro
vagas ficam com eles; nenhum outro repositório entra por atraso (o maior atraso entre os demais é 0).

Critério 1 (defeito do catálogo) **não** selecionou ninguém nesta hora: os 12 commits e os 14 merges foram
conferidos contra as 100 entradas dos quatro repositórios no `catalogo/defeitos.csv` e nenhum reaplica um
D-xx. Em compensação, dois defeitos novos saem desta janela e estão registrados no cruzamento de
`Xbox2Windows.md` e `victory-fork.md`.

`Marvisak` (76 PRs abertos) e `xcl` (24) perdem a vaga porque o critério 3 vem depois do 2. Os cursores
deles ficam parados; a triagem registra o atraso.

## Números da janela

- **17 commits** acima dos cursores: `Xbox2Windows` 12, `TuxBox` 2, `SixNG` 2, `victory-fork` 1.
- **123 PRs abertos** acima dos cursores de número: `VIctory` 76, `xcl` 24, `Xbox2Windows` 14,
  `TuxBox` 3, `victory-fork` 2, `SixNG` 2, `VItoriaBr` 1, `xbox-emulator` 1.
- **14 PRs mesclados** com `merged_at` na janela: `Xbox2Windows` 12, `SixNG` 2. Os 12 merges de
  `Xbox2Windows` e os 2 de `SixNG` casam 1:1 com os commits da faixa (squash de um pai, `merge_commit_sha`
  igual ao SHA do commit).
- **3 commits sem PR**: os 2 de `TuxBox` e o 1 de `victory-fork` foram empurrados direto na default
  (`repos/<o>/<r>/commits/<sha>/pulls` devolve lista vazia nos três), o que explica 17 commits contra 14
  merges.
