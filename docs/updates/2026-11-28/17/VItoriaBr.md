# uiliannn/VItoriaBr — atualização de 2026-11-28 17:00

## Resumo

Sem commit e sem merge na janela: o head `15779f81` está parado desde 27/11. O que há são três PRs
abertos, todos de 27–28/11, e os três de correção ou de robustez de teste: `#132` faz o helper de
teste estrito falhar quando o validador DXIL não carrega, `#133` impede o otimizador de bitcode de
desduplicar `dx.op.getDimensions` (crash do `amdxc64` em 29 pipelines de compute de raytracing, RDNA4)
e `#134` dá handle próprio às consultas de tamanho de buffer. `#133` e `#134` são portes declarados de
outros repositórios.

## Commits novos (0)

`compare(15779f8197fe846b0cdf4509c15a8ef00433e216...15779f8197fe846b0cdf4509c15a8ef00433e216)`:
`status: identical`, `ahead_by: 0`. Nenhum commit novo desde o cursor; nada a documentar por diff de
commit.

## Pull requests abertos (3)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #134 | shader: own handle for buffer size queries | Separado de #133. `Lower::handles` é compartilhado entre consultas de tamanho e acessos de memória; no `amdxc64` (RDNA 4, ex. RX 9070 XT) o compilador estourava quando o mesmo handle alimentava uma consulta de tamanho de buffer raw e uma carga/store. O diff adiciona `size_handles: HashMap<...>` e um `swap_size_caches`, para cada query pegar handle exclusivo. +21/−0, 1 arquivo (`emit/lower.rs`). Porte declarado de `Marvisak/VIctory#2122`. | Área de codegen que derruba o compilador AMD. No catálogo **deste** repositório não há linha para isso; a equivalente está em `XCL D-02` ("o codegen de #123 derrubou o compilador de shader da AMD (amdxc64)"). |
| #133 | shader: do not reuse texture size query (fixes AMD compiler crash) | No otimizador de bitcode, uma mesma consulta `dx.op.getDimensions` servindo usos de outro bloco derrubava o `amdxc64` (RDNA4/RDNA2) em 29 pipelines `CS_RaytracingResolve`. O diff põe `Op::GetDimensions` (ou seja, a consulta) fora da chave de CSE, mantendo-a no ponto em que o lowering a emitiu; teste de regressão `a_size_query_is_not_reused`. +29/−0, 1 arquivo (`emit/bitcode/opt.rs`). Porte declarado de `Belobug #324`. | Mesma área do anterior; mesma observação de catálogo. |
| #132 | tests: enforce DXIL validator loading in strict VALU builds | `valu_tests::build_cs` ignorava em silêncio o erro de carga do validador DXIL mesmo com `VICTORY_STRICT_TESTS=1`, então uma execução estrita podia passar sem validar DXIL nem conferir o hash do contêiner. O diff roteia o erro pela política de skip existente: estrito falha antes da GPU, comum registra validação omitida e segue. +8/−5, 1 arquivo (`emit/valu_tests.rs`). O próprio PR diz que o helper dos testes de wave fica fora. | Área de "teste que não pode rodar reporta sucesso" — catalogada em `OpenScarlett D-09` (`SKIP_RETURN_CODE`), não neste repositório. |

## Pull requests mesclados (0)

Nenhum merge com `mergedAt` acima de `2026-11-27T13:05:00Z`. O cursor de mesclados fica **parado** nesta
hora, para não esconder merge futuro (a lição dos dois relógios registrada em `state/watch.json`).

## Cruzamento com o catálogo

- **Corrige D-xx?** Não há hash para conferir (zero commits, zero merges). Nenhum dos três títulos cita
  id do catálogo.
- **Toca área de D-xx?** `#133` e `#134` estão na área de codegen que quebra o compilador da AMD — a
  linha que descreve esse sintoma está em `XCL D-02` e não em `VItoriaBr`. `#132` é da família de
  "teste que passa sem rodar", descrita em `OpenScarlett D-09`. Se o catálogo deste repositório for
  para cobrir a família de forks, essas três são candidatas a linha nova.
- **Abre defeito novo?** Não verificado. `#133`/`#134` desligam uma otimização (CSE de consulta de
  tamanho, handle compartilhado) — o efeito é custo, não corrupção, mas isso é leitura do diff, não
  medição.

## Confiança

**Verificado por diff.** Os três PRs foram lidos por corpo e `pulls/<n>/files`, com os hunks iniciais
de `emit/bitcode/opt.rs`, `emit/lower.rs` e `emit/valu_tests.rs`. O `compare` foi executado e devolveu
`identical`.

**Não verificado.** Nada foi compilado nem executado; não há GPU nem `amdxc64` neste ambiente, então o
crash em 29 pipelines e a cura são alegações dos PRs (um deles diz ser porte de `Belobug #324`, o outro
de `Marvisak/VIctory#2122` — nenhum dos dois repositórios responde nesta hora: `Belobug` está em 404).
Não li os 74 PRs restantes do repositório nem os 124 anteriores ao cursor.

**Estado.** `last_commit` `15779f81` (parado, correto: nada novo); `last_pr_number` **134** (avançou de
126 para o maior aberto lido); `last_run` mantido em `2026-11-27T13:05:00Z` (sem merge na janela).
