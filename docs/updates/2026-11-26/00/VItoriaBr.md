# uiliannn/VItoriaBr — atualização de 2026-11-26 00:00

## Resumo

Um commit novo (`e40fba44`), que torna configurável o número de slots dinâmicos de descritor quando o
heap de views tem teto no host. Três pull requests abertos acima de 108: um relatório de arquitetura
sobre paralelismo de software e localidade de GPU, uma correção de arredondamento em `v_cvt_f32_u32` e
um ganho de inicialização ao passar as leituras de pacote pelo cache de arquivos do Windows.

## Commits novos (1)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `e40fba44` | uiliannn | gpu: VICTORY_DYNAMIC_SLOTS on a capped view heap (trustno1-coder/VIctory-NVIDIA a9632bc) | Em `crates/victory-gpu/src/descriptor/host.rs` (+41/-3), `dynamic_slots_on(null, asked)` decide quantos slots dinâmicos usar: sem teto (`null == 0`) mantém `DYNAMIC = 1 << 18`; com teto, o máximo é `(null / 2).max(DYNAMIC)` e o valor pedido é limitado a `[DYNAMIC, most]`. `dynamic_slots()` lê `VICTORY_DYNAMIC_SLOTS` uma vez em `OnceLock` e emite `tracing::warn!` com `slots` e `title_in_place = null.saturating_sub(slots)` quando o resultado difere do padrão. O comentário do módulo diz que slots acima do teto são refeitos como qualquer outro. |

## Pull requests abertos (3)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 109 | docs: investigate UMA-aware software parallelism and CPU–GPU execution overlap | Relatório em inglês (`docs/uma-aware-software-parallelism.md`, +463) sobre visibilidade de memória do convidado, paralelismo de software e localidade de GPU em D3D12, com dez hipóteses falsificáveis, cinco diagramas e protocolo de benchmark; `docs/architecture.md` ganha 6 linhas. Parte do que a `main` já tem (relay estágio 2, late waits, copy-back largo, sombras GPU-local, PSO workers). | Não. É documento, não correção. `D-21` trata de deriva de documentação (números retirados, contradição sobre TDR) já corrigida em `b37344f`; este PR adiciona análise nova. |
| 110 | shader: round full-width unsigned integer conversions once in v_cvt_f32… | Em `crates/victory-shader/src/emit/lower.rs` (+16/-2), decompõe o operando de `v_cvt_f32_u32` em duas metades exatas de 16 bits e combina por `fmad`, para a soma arredondar uma única vez independentemente de o host fundir FMad (o defeito aparece no WARP). | Não é D-xx deste repositório. `XCL D-06`/`D-07` são arredondamento de f16, outro caso. |
| 111 | hle: read title packs through the host file cache (7x faster launch) | Remove `FILE_FLAG_NO_BUFFERING` de `CreateFileW` com `GENERIC_READ` exato e não sobreposto (`modules/files.rs` +129), deixando as leituras de pacote passarem pelo cache de arquivos do Windows; aberturas sobrepostas e com escrita mantêm as flags. Declara queda de ~37 s para ~5,5 s até a primeira tela, com 4 testes de regressão. `architecture.md` documenta. | Não. |

## Pull requests mesclados (0)

Nenhum PR com `mergedAt` posterior a 2026-11-26T02:11:35Z. O maior número mesclado devolvido é 93,
abaixo de `last_pr_number` 108.

## Cruzamento com o catálogo

- Corrige D-xx? Nenhum diretamente. O commit `e40fba44` é uma **mitigação** na área de `D-04`
  ("varredura de wraparound dos descritores dinâmicos truncada: exaustão falsa", corrigido em `009856f`)
  e de `D-05` ("heap de descriptors esgotado lia o descritor do título como CBV → remoção do device na
  AMD", corrigido em `3c81420`): agora, num heap com teto, dá para reduzir `DYNAMIC` em vez de estourar.
  Não é correção — é um botão, desligado por padrão, e o próprio diff avisa por `warn!` quando atua.
- Abre defeito novo? Não evidente. O clamp `[DYNAMIC, most]` com `most = (null/2).max(DYNAMIC)` garante
  que o valor pedido nunca fica abaixo do padrão nem acima da metade do heap; `null == 0` sai cedo.

## Confiança

Verificado: commit `e40fba44` lido por `gh api .../commits/e40fba449b48...` (um arquivo, patch
completo), pai `85615ad3` igual ao `last_commit` do estado, autor e data da API. PRs lidos por
`gh pr view` (corpo, arquivos, linhas).

Não verificado: o ganho de 7x declarado em `#111` não foi medido aqui, e depende de o cache do Windows
estar quente — o próprio corpo diz "em relançamentos com arquivos já cacheados".
