# ivall/xcl — atualização de 2026-11-28 19:00

## Resumo

Hora de conserto de teste e de fronteira de tradução. Dos 10 commits sem documento, quatro são o
**mesmo** conserto de `E0451` rebasado (`15b1845a` e três duplicatas) e os outros seis corrigem
assertivas que não podiam passar e fronteiras que o tradutor não respeitava: `bba710e2` impede a fusão
de CFG de re-fundir o bloco `getDimensions`/`TextureLoad` de `Texture2DMS` (**segunda ocorrência da
classe de D-24**), `2de1544d` aplica ao wave64 nativo os limites de `s_setpc`/`gs_start` que só valiam
no split (classe de D-19) e `5f284cc2` faz as escritas da própria lista pousarem num chunk esparso que a
CPU também alterou. Zero merges; 14 PRs abertos novos, dois acima do cursor (#3200, #3202).

## Commits novos (10 de 45 no `compare`)

O `compare` de `f2c0d36f` até o head `10d27297` devolve 45 commits, mas o cursor não é ancestral de
tudo o que já foi documentado: `compare/86a90eeb...f2c0d36f` devolve `diverged` (5 à frente, 3 atrás),
então 35 hashes voltam a aparecer e foram pulados **por hash** contra `docs/updates/*/xcl.md`. Restam os
10 abaixo. `93ea614b`, `f52da002` e `dc684f21` são o mesmo patch de `15b1845a` rebasado.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `15b1845a` | thehollowedone | Tests: construct benchmark config without private fields | `benches/runtime.rs` +2/-4: troca `Config { mounts, ..Config::default() }` por `Config::default()` seguido de `resolved.mounts = mounts`. O struct update a partir de fora da crate lia `input_profile` privado e o alvo de benchmark não compilava (`E0451`) |
| `93ea614b` | thehollowedone | (idêntico a `15b1845a`) | Mesmo `benches/runtime.rs` +2/-4, mesma mensagem: rebase do mesmo conserto. Não é achado novo |
| `f52da002` | thehollowedone | (idêntico a `15b1845a`) | Idem |
| `dc684f21` | thehollowedone | (idêntico a `15b1845a`) | Idem |
| `99e2b1b3` | KOLESNiii | Tests: the split-piece binding check looks for title memory's own ResBind | `dxil/module/split.rs` +1/-1: a string procurada passa de `ResBind { i32 0, i32 2, i32 2, i8 1 }` para `ResBind { i32 2, i32 2, i32 0, i8 1 }`. A forma do DXIL é `{lower, upper, space, class}`; memória do título é `u2, space 0`, então a assertiva antiga não casava com peça nenhuma e falhava em toda GPU que rodasse o fixture |
| `1ecc3035` | KOLESNiii | Tests: one root UAV wraps an outside address in the heap flat-read test | `recomp/heap_window_tests.rs` +5/-1: com **uma** janela (root UAV) o endereço fora da arena **dá a volta** e o valor esperado na lane 7 passa de `0` para `1000 + lane`; com mais de uma janela continua `0`. Antes o teste esperava `0` nos dois casos |
| `1d9a6424` | KOLESNiii | Tests: direct-DXIL comparisons take the HLSL path where the runtime does | `recomp/tests.rs` +35/-13: novo `routed_to_hlsl()` reconhece as duas recusas que o runtime responde pelo caminho HLSL (`s_barrier in a group of several hardware waves`, `split wave64 LDS access needs the HLSL path`); `direct_or` e os dois testes que traduziam DXIL direto à mão passam a usar HLSL nesses casos e pulam só a comparação do backend direto. Antes panicavam na recusa |
| `2de1544d` | bmjask | Recompiler: geometry ES's `s_setpc` and `gs_start` bound structured stretches in every wave form | `recomp/mod.rs` +13/-7: o campo `phase_split: bool` vira `geometry: bool` mais um segundo limite para a entrada do GS. O `s_setpc` não resolvido do ES termina o ES **em qualquer forma de onda** (antes só no split wave64, e no wave64 nativo um `s_setpc` adiantado caía dentro do outro ramo); e um stretch não pode começar no padding entre ES e GS e atravessar `gs_start`, que é aresta que `edges` não carrega. `recomp/tests.rs` +54/-0 traz `geometry_with_es_tail` |
| `5f284cc2` | KOLESNiii | D3D12.X: a list's own GPU writes land in a sparse chunk the CPU changed | `hle/d3d12x/sparse.rs` +40/-11: `land_page` ganha `gpu_written`. Desde `04de3234` o copy-back preserva os bytes que a **CPU** mudou num chunk de 64 B e descarta os da GPU — certo quando a CPU reescreveu o chunk inteiro, errado quando os dois escreveram bytes diferentes dele. Agora os bytes que os comandos da própria lista escreveram (`sparse_writes`, registrados por `note_write`) também pousam no chunk alterado, e o hash antigo faz a próxima olhada de write-watch subir o chunk inteiro. A leitura por `Map` (`readback_now`) continua preferindo os bytes da CPU |
| `bba710e2` | bmjask | Recompiler: preserve MSAA load block boundary | `dxil/lower_image.rs` +1 marca o bloco que separa `GetDimensions` de `TextureLoad`; `dxil/module.rs` +7 guarda `preserve_block_boundary: Vec<bool>` com setter; `dxil/module/opt.rs` +26 recusa fundir bloco marcado na simplificação de CFG, com o teste `simplify_cfg_keeps_preserved_block_boundary`. É a guarda de regressão da classe de D-24: a simplificação re-fundia o bloco e o padrão de crash do `amdxc64` voltava |

## Pull requests abertos (14)

Criados depois de `last_run` (`2026-11-28T16:59:49Z`) ou acima do cursor #3198. Nenhum corpo foi lido,
exceto onde indicado; a coluna "Proposta" vem do título e, nos quatro marcados, do corpo.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| `#3202` | README: document `XCL_LAZY_PSO` | Documenta o switch de PSO lazy no README | Não |
| `#3200` | Recompiler: patch console FP16 PS exports for 32-bit render targets | Corpo lido: sequências `v_cvt_pkrtz_f16_f32` + `exp mrtK ... compr` do epílogo comprimido de PS de console são remendadas para export não comprimido de 32 bits logo após o decode RDNA2, nos dois backends, chaveadas por máscara de RTs de 32 bits | Não — capacidade nova |
| `#3199` | Tests: fine timer test takes the median of time-critical sleeps | Mede o timer pelo mediano das esperas time-critical em vez de cronometrar `Sleep` | Não (fechado sem merge às 19:2xZ) |
| `#3198` | Shaders: report bounded preparation timing | Reporta o tempo da preparação limitada | Não |
| `#3197` | Shaders: prepare exact graphics history at launch | Prepara no arranque o histórico exato de gráficos | Não |
| `#3196` | D3D12.X: identify exact graphics recipes in manifests | Identifica receitas exatas de gráficos no manifesto | Não |
| `#3195` | Shaders: bound history-backed launch preparation | Limita a preparação de arranque apoiada em histórico | Não |
| `#3194` | D3D12.X: release large texture upload buffers after idle | Devolve buffers grandes de upload de textura depois de ociosos | Não |
| `#3193` | Tests: verify the finest timer request without timing `Sleep` | Verifica o pedido de timer mais fino sem cronometrar `Sleep` | Não |
| `#3192` | D3D12.X: a superseded target the title binds, clears or moves is mapped again | Corpo lido: alvo colocado que um placement mais novo substitui tem os tiles desmapeados; `placements::note_use` passa a remapear também em `OMSetRenderTargets` com RTV/DSV, `ClearRenderTargetView`, `ClearDepthStencilView` e barreira de textura no grupo da lista #146, tudo gravado na lista antes de usar | Não catalogado; mesma classe do VIctory D-01 (slot/alvo usado depois de aposentado) |
| `#3191` | README: show `--pso-manifest` in the `--through-present` bullet | Documentação | Não |
| `#3190` | Tests: `alloc_pool` checks for STATE held by the caller, not by a parallel test | Torna o teste de `alloc_pool` independente de teste paralelo | Não |
| `#3189` | Recompiler: 2-D and 3-D compute groups take the direct DXIL route again | Corpo lido: remove a regra de `d9403543` que mandava grupo 2-D/3-D para o HLSL ("reorder console-linear"); o tradutor direto já tem o reorder desde `746f7ddc`, e `TRANSLATION_GLUE_REVISION` sobe para `pso-glue-v5` para o cache não servir as traduções HLSL antigas | Não — remove recusa introduzida |
| `#3188` | D3D12.X: disable lazy level pages when a sampled texture aliases a live GPU page | Corpo lido: port de `amd-dev dece6dd9`; sob `XCL_TEXTURE_LEVEL_PAGES=1`, `vram::place_pages` marcava `lazy = true` para toda textura 2-D de fatia única, adiando o mapeamento de tiles até `map_levels_with_texels` ver texels não zero — errado quando a textura aliasa página de GPU viva | Não catalogado |

## Pull requests mesclados (0)

Nenhum PR fechado com `merged_at > 2026-11-28T16:59:49Z`. `#3199` foi **fechado sem merge** durante a
coleta.

## Cruzamento com o catálogo

- **Corrige D-24? Parcialmente, como guarda de regressão.** `bba710e2` (hash) não é o conserto de D-24
  (`425b869`), é a segunda vez que a **mesma causa** reaparece: `56237a2d` pôs `getDimensions` antes dos
  loads de `Texture2DMS` e a simplificação de CFG re-fundiu o bloco, restaurando o crash do `amdxc64`
  (registrado às 15:00 para o merge `309c3ec5`). O diff agora marca o bloco e recusa a fusão, com teste
  próprio. Nenhum hash do catálogo aparece na janela.
- **Classe de D-19 (geometry split).** `2de1544d` estende ao wave64 nativo o limite de stretch que D-19
  descreve para a fase split: o ES caía dentro do código do GS e os records saíam vazios.
- **Classe do VIctory D-07 (copy-back contra escrita da CPU).** `5f284cc2` é o caso espelhado: aqui o
  copy-back descartava a escrita da **GPU** num chunk que a CPU também tocou. O e2e do título falhava com
  `post-Wait32/64 write failed (0x0)`.
- **Abre defeito novo?** Nada no diff desta janela abre defeito; três dos commits apenas corrigem
  assertivas que nunca podiam passar (`99e2b1b3`, `1ecc3035`, `1d9a6424`), o que é dívida de teste
  paga, não defeito novo.

## Confiança

- **Verificado:** os 10 commits, um a um, por `repos/ivall/xcl/commits/<sha>` (arquivos e ± linhas) e
  pelo patch de cada um; o `compare` de `f2c0d36f` a `10d27297` (45, `ahead`, 0 atrás); a divergência
  `86a90eeb...f2c0d36f` (5 à frente, 3 atrás) que explica os 35 hashes repetidos; o estado de `#3199`
  (fechado, não mesclado), `#3200` e `#3202`; os corpos de `#3188`, `#3189`, `#3192` e `#3200`.
- **NAO VERIFICADO:** os corpos dos outros 10 PRs abertos (só título); se algum dos commits desta
  janela foi testado em GPU (as mensagens citam `cargo test`/WARP, o diff não prova execução); a
  contagem real de PRs abertos do repositório — 50 lidos, menor #3135, então o total é ≥50 e o cursor
  #3200 não prova cobertura dos antigos.
- **Desvio de cursor registrado:** o `last_commit` gravado às 18:00 (`f2c0d36f`) não contém
  `86a90eeb`, que já estava documentado. O número "38 restantes" daquela hora não se reproduz; a
  medição desta hora é 10 sem documento. O cursor desta execução avança para o head `10d27297`, o que
  zera a fila de commits do repositório.
