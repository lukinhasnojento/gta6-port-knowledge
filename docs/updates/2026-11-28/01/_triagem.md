# Triagem de 2026-11-28 01:00

Coleta entre 01:07 e 01:2x UTC real — ~22:07–22:2x no relógio do GitHub, que segue cerca de 3 h atrás e
é o relógio em que `last_run` é gravado. Cursor de commits de cada repositório = `last_commit` do
`state/watch.json` lido no início (`runs: 22`). 14 entradas em `repos` no início e no fim; nenhuma
removida, nenhuma acrescentada.

## Sanidade (passo 1)

- `gh api repos/<o>/<r> --jq .default_branch` pedido nos 14. **Dois sem resposta**:
  `Belobug/VIctory-Belobug-AMD` e `avatarmanz/VIctory-7900XTX` devolvem HTTP 404 em `repos/` (quinta hora
  seguida) — cursor parado, triagem NAO VERIFICADO. Detalhe em `_falhas.md`.
- `default_branch` idêntico ao gravado nos 12 restantes, inclusive `master` (`holdmysocks/SixNG`) e
  `a-publier` (`Belobug`, gravado de antes do 404 e inverificável agora).
- Nenhum `last_commit` nulo: não houve baseline nesta execução.
- Nenhum `compare` devolveu `diverged` (12 medições: 4 `ahead`, 8 `identical`).
- Nenhum repositório passa de 200 commits de atraso; o maior atraso é 29.

## Triagem (passo 2)

Janela de PRs abertos: número maior que `last_pr_number`, contado sobre a primeira página de 100
(`per_page=100`, `sort=created`, `direction=desc`), com cobertura provada só quando o menor número da
página é menor que o cursor. Janela de mesclados: `merged_at` maior que `last_run`, por
`repos/<o>/<r>/pulls?state=closed&sort=updated` paginado até `updated_at` cair abaixo do cursor (a busca
`merged:>=` não é usada — perde merges em repositório transferido).

| repo | commits atrás | PRs abertos | PRs mesclados | sinal | prioridade |
|---|---|---|---|---|---|
| `Marvisak/VIctory` | 0 | 81 (100 na 1ª página, mín. 3494 < cursor 3521) | 0 | SIM — critério 3, `GPU:`/`Shaders:`/`CLI:`; `#3611` slots dinâmicos devolvidos ao heap de origem, `#3613` gathers do primeiro nível, `#3609` `v_exp_f16` | média, sem vaga |
| `ivall/xcl` | **2** | 25 (100, mín. 2955 < 3042) | 1 | SIM — critério 2, liberação de DXIL offline por pipeline (`d8dae08d`), e critério 3 na pilha de bibliotecas nativas de `#3045`–`#3075` | **deep dive** |
| `R0Y00/Xbox2Windows` | **29** | 13 (100, mín. 1334 < 3191) | 29 | SIM — critério 2, 20 lowerings/recusas novas no compute direto, 2 correções de runtime (sampler de menu, footprints de mip XG) e o conserto do `SENTINEL` | **deep dive** |
| `californiaAlmonds/OpenScarlett` | 0 | 0 (1 total) | 0 | NAO | — |
| `Belobug/VIctory-Belobug-AMD` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (HTTP 404) | bloqueado |
| `DexNom/VIctory` | 0 | 0 (100, máx. 148 = cursor) | 0 | NAO | — |
| `uiliannn/VItoriaBr` | 0 | 1 (75 total) | 0 | SIM fraco — critério 3, `#132` validador DXIL estrito | baixa |
| `avatarmanz/VIctory-7900XTX` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (HTTP 404) | bloqueado |
| `fxceless/victory-fork` | 0 | 2 (100, mín. 1274 < 1496) | 0 | SIM fraco — critério 3, `#1500` tetos de textura sob pressão de RAM/VRAM e `#1499` tabela de lotes indiretos | baixa, sem vaga |
| `omonkeyz/xbox-emulator` | 0 | 1 (25 total) | 0 | SIM fraco — critério 3, `#395` validação do bloco de relocação PE | baixa |
| `brcly/TuxBox` | **5** | 9 (15 total) | 0 | SIM — critério 2, inventário por instrução do tradutor que achou uma recusa falsa, redução de ângulo de seno/cosseno e FMA (`mad`) | **deep dive** |
| `holdmysocks/SixNG` | **6** | 1 (5 total) | 6 | SIM — critério 2, camadas de máscara de tatuagem (`#507`), índices de imagem no merge de personagem (`#508`), adesivo plano (`#509`) | **deep dive** |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 | 0 | NAO | — |
| `stellarr-studio/kelvaroq` | 0 | 0 | 0 | NAO | — |

## Seleção do deep dive (passo 3)

Vagas: 4. Exatamente quatro repositórios tiveram commit na janela e todos vencem pelo critério 2
(correção, regressão ou reversão): `Xbox2Windows` (29), `SixNG` (6), `TuxBox` (5) e `xcl` (2). As quatro
vagas ficam com eles; nenhum outro repositório entra por atraso (o maior atraso entre os demais é 0).

Critério 1 (defeito do catálogo) **não** selecionou ninguém nesta hora: os 42 commits foram conferidos
contra as 306 linhas do `catalogo/defeitos.csv` — nenhum dos 42 hashes aparece nas colunas `introduzido`
ou `corrigido`, e nenhum reaplica um D-xx por área.

`Marvisak` (81 PRs abertos) e os demais com PR aberto perdem a vaga porque o critério 3 vem depois do 2.
Os cursores deles ficam parados; a triagem registra o atraso.

## Números da janela

- **42 commits** acima dos cursores: `Xbox2Windows` 29, `SixNG` 6, `TuxBox` 5, `xcl` 2.
- **133 PRs abertos** acima dos cursores de número: `VIctory` 81, `xcl` 25, `Xbox2Windows` 13,
  `TuxBox` 9, `victory-fork` 2, `SixNG` 1, `VItoriaBr` 1, `xbox-emulator` 1.
- **36 PRs mesclados** com `merged_at` na janela: `Xbox2Windows` 29, `SixNG` 6, `xcl` 1.
- **5 commits sem PR**: os 5 de `TuxBox` foram empurrados direto na default
  (`repos/brcly/TuxBox/commits/<sha>/pulls` devolve lista vazia nos cinco).
- `xcl` é o único caso em que commits (2) ≠ merges (1): o merge `2c717a04` é um merge real de dois pais,
  não squash, e traz `d8dae08d` junto na faixa.

## Observação sobre a "tabela de pistas"

O passo 2 do procedimento manda usar "arquivo das áreas da tabela de pistas", mas essa tabela não existe
em `docs/automacao/ANALISE_HORARIA.md` — ela só é citada na linha 231. Na ausência dela, as áreas do
sinal vieram das seções D do `catalogo/defeitos.md` (tradutor DXIL/`gfx2hlsl`, bind de descritor e heap,
memória e XMem, textura e tiling, present e save) e do assunto de cada PR. O desvio está registrado em
`state/last-run.md`.
