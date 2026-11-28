# R0Y00/Xbox2Windows — atualização de 2026-11-28 19:00

## Resumo

Uma hora curta e inteiramente verificável: `s_bfm_b64` passa a aceitar **VCC** como destino no tradutor
direto de compute (`2e460b26`), com teste de controle e de hardware novo; `f81feb16` conserta a
codificação do próprio teste. O merge `#3357` (`eeb52277`, 19:07:04Z) entrou logo depois do primeiro
snapshot da coleta. Seis PRs abertos acima do cursor, todos pequenos e na mesma frente de manifesto e
de tópicos de runtime.

## Commits novos (3)

Faixa `compare/f2972840...eeb52277`, `ahead` 3, 0 atrás.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `2e460b26` | Ammmae0x | compute: accept VCC as destination in `s_bfm_b64` | `src/gfx1013_direct_compute.inl` +1/-1: a condição que recusa destino vira `(i.dst & 1) || (i.dst > 104 && i.dst != 106 && i.dst != 126)`. VCC é par (106) e agora passa; `VCC_HI` (107) continua recusado pelo bit 0, `EXEC` (126) já era aceito e `M0` (124) segue recusado. Antes, VCC caía na recusa e o programa não baixava pelo caminho direto. Entram `tests/native_s_bfm_b64_vcc_check.py` (+137, com pernas `controls` e `hardware`), `tests/cmake/native_s_bfm_b64_vcc.cmake` (+12, `TIMEOUT 60`, condicionado a `compile_dxil` + `dxcompiler.dll`/`dxil.dll` e a `native_store_replay`), `docs/experiments/native-salu64-ops.md` (+7/-1) e, em `tests/native_wave64_exec_salu_check.py` (+3/-2), o controle negativo deixa de ser VCC e passa a ser M0 mais o destino fora de faixa 23 |
| `f81feb16` | Ammmae0x | compute: fix v9 store encoding in `s_bfm_b64` VCC test | `tests/native_s_bfm_b64_vcc_check.py` +9/-3: o template copiado de `s_not_b64` estreitava `EXEC` a uma máscara e depois restaurava; em `s_bfm_b64` `EXEC` nunca é estreitado, então a restauração sai. E o `v9` passa a ser guardado direto, porque `vmove(1, 9)` codifica `v_mov_b32 v1, s9` (lê o escalar 9, não o VGPR 9) e o readback lia outro registrador |
| `eeb52277` | Leyukaka | Merge pull request #3357 from Ammmae0x/compute/s-bfm-b64-vcc | Merge de dois pais do `#3357` sobre `f2972840`. Diff contra o primeiro pai = os dois commits acima; nada além deles |

## Pull requests abertos (6 acima do cursor #3387)

50 abertos lidos; #3390 e #3392 não aparecem na janela de 50. Nenhum corpo lido — a coluna vem do
título.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| `#3388` | Clean replacement for #2853 | Substituição limpa do PR #2853 | Não verificado (título não diz o escopo) |
| `#3389` | compute: publish `packed_d16_x_uav` runtime manifest (Part of #3224) | Publica manifesto de runtime para o UAV `packed_d16_x`; declarado parte de #3224 | Não |
| `#3391` | startup: opt-in table lock so resource and heap lookups stop queueing on `note_lock` | Lock de tabela opt-in para as buscas de recurso e heap pararem de enfileirar no `note_lock` | Área de D-09 (`startup_view_heap` lê o trap log sem compartilhamento) — não é o mesmo defeito |
| `#3393` | runtime: assert that physical page tokens fit the caller's 21-bit entry | Assertiva de que o token de página física cabe na entrada de 21 bits do chamador | Não |
| `#3394` | tests: expect the output-token refusal for a texture token in the output check | O teste de saída passa a esperar a recusa de token de textura | Não |
| `#3395` | compute: name the packed D16 and indexed raw view kinds in manifest reasons | Nomeia os tipos `packed D16` e `indexed raw view` nas razões do manifesto | Não |

## Pull requests mesclados (1)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| `#3357` | compute: accept VCC as destination in `s_bfm_b64` | `eeb52277` (19:07:04Z) | Os dois commits de conteúdo da tabela acima: a recusa de destino VCC sai do caminho direto e o teste ganha perna de controle e de hardware. `merged_at` 19:07:04Z bate com o `pushed_at` do repositório lido às 19:07Z, então o relógio do GitHub continua sem desvio |

## Cruzamento com o catálogo

- **Corrige defeito catalogado?** Não. Nenhum dos 29 defeitos de `catalogo/defeitos.csv` para
  `Xbox2Windows` trata de `s_bfm_b64` nem do conjunto de destinos válidos de SALU. O que o diff fecha é
  uma **recusa** do tradutor direto — a mesma frente dos lowerings de 12:00–14:00, não um defeito
  catalogado.
- **Abre defeito novo?** Nada no diff sugere. O risco do lado do teste é o oposto: o readback corrigido
  em `f81feb16` mostra que a primeira versão do teste lia o registrador errado, isto é, o teste do
  commit anterior não provava o que dizia provar.
- **Verificação de coerência:** a mudança do controle negativo em `native_wave64_exec_salu_check.py`
  (VCC → M0) é evidência de diff de que VCC era tratado como erro antes deste commit.

## Confiança

- **Verificado:** os 3 commits por `repos/R0Y00/Xbox2Windows/commits/<sha>` (arquivos, ± linhas e
  patch); o `compare/f2972840...eeb52277` (`ahead` 3); o `merged_at` de `#3357`; a lista de 6 PRs
  abertos acima de #3387 pelo `gh pr list --state open --limit 50`.
- **NAO VERIFICADO:** se o teste de hardware (`native_s_bfm_b64_vcc_hardware`) realmente rodou — o diff
  mostra o registro no CMake e o `TIMEOUT`, não a execução; os corpos de `#3388`–`#3395`; se `#3388`
  ("Clean replacement for #2853") toca as áreas de D-01/D-02, porque o corpo não foi lido.
- **Coleta:** o merge chegou às 19:07:04Z, depois do primeiro snapshot; a faixa foi refeita. Nada ficou
  sem documento neste repositório (3 de 3).
