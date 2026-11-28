# R0Y00/Xbox2Windows — atualização de 2026-11-28 18:00

## Resumo

Hora de lowerings novos no tradutor direto e de dois consertos com sintoma de renderização: `#3375`
desfaz a máscara de 10 bits aplicada a um campo de 9 bits do índice de mesh (a primitiva com flag de
aresta era culled) e `#3139` recupera o par de disco publicado quando a criação do PSO nativo falha.
Dos 4 PRs mesclados na janela, **um (`#3139`) não entrou na `main`** — foi para
`codex/native-graphics-integration` e não aparece na faixa de commits. Head `f2972840`.

## Commits novos (11)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `58d69281` | R0D4X | startup: lower v_dot4_i32_i8 and v_dot4_u32_u8 in gfx2hlsl | Adiciona os dois `v_dot4` ao tradutor HLSL do caminho de arranque (`tools/startup/gfx2hlsl.py` +56): soma de 4 produtos com sinal e sem sinal sobre bytes empacotados, mais fixture `.cmake` e verificador Python de 299 linhas |
| `eeaa3d78` | R0D4X | shader: lower v_min_f16 and v_max_f16 in the direct compute path | Lowering nativo de mínimo/máximo f16 no compute direto (`src/gfx1013_direct_compute.inl` +29), com doc de experimento e verificador de 158 linhas que confere valor contra IR de CPU |
| `1d395acb` | lewisalex86 | Merge main into feat/gfx2hlsl-v-dot4 | Merge da `main` no ramo do `v_dot4`: 36.358 adições, praticamente todas documentação de experimentos, fixtures e checkers já existentes na `main` — integração, não trabalho novo |
| `8931b511` | R0D4X | tests: add signed-clamp sign-guard lanes for v_dot4_i32_i8 | Nove linhas no verificador do `v_dot4`: lanes de guarda de sinal para o clamp com sinal |
| `1442d8ea` | R0D4X | Merge main and address v_min_f16/v_max_f16 review feedback | Segundo merge da `main` (53.620 adições, de novo quase todo doc/fixture) mais o retrabalho pedido na revisão do `v_min_f16`/`v_max_f16` |
| `58468b35` | R0D4X | tests: add CPU IR value verification and inline 248 coverage for v_min_f16/v_max_f16 | Verificação de valor contra IR de CPU e cobertura "inline 248" para as formas nativas de min/max f16 (+79/−41 no verificador), com a doc do experimento corrigida |
| `67cc213f` | R0D4X | Merge upstream/main into feat/gfx2hlsl-v-dot4 | Terceiro merge da `main` no ramo (27.261 adições): traz `gdk_*` (exports, layouts, knowledge report, shader corpus), BCPack standalone, `xapu`/`acphal` e o checker de CPU `adx_aot` |
| `d418f70e` | Leyukaka | Merge pull request #3031 from R0D4X/feat/gfx2hlsl-v-dot4 | Merge do `#3031` (+376): o `v_dot4` entra com 56 linhas de tradutor e 308 linhas de checker |
| `6685de7c` | Leyukaka | Merge pull request #3125 from R0D4X/feat/native-minmax-f16 | Merge do `#3125` (+326): `v_min_f16`/`v_max_f16` no compute direto, com doc, fixture e 242 linhas de checker |
| `24b895d7` | Roland Edi | shader: keep 9-bit mesh primitive indices | **Corrige o frontend de mesh:** o export primitivo empacota três slots de 10 bits, cada um com índice de 9 bits e o flag de aresta no bit 9; o C++ mascarava com 1023, então índice com flag virava 512..1023 e a guarda `triangle[c] >= vertices` cullava a primitiva. Passa a 511 (`src/gfx1013_direct_mesh.inl` +1/−1) |
| `f2972840` | Leyukaka | Merge pull request #3375 from lambirou/gpu/mesh-edge-mask | Merge do `#3375`: o conserto acima mais o fixture `native_direct_mesh`, que passa a cobrir `0x20280600` (índices 0/1/2 com as três flags de aresta). Controle negativo com a máscara 1023: 0/1 |

## Pull requests abertos (7)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #3379 | runtime: add the input clock to input trace state lines | Acrescenta o relógio de entrada às linhas de estado do trace de input | Não |
| #3380 | Log skipped DrawInstanced when SRV not bound | Registra `DrawInstanced` pulado quando a SRV não está ligada — instrumentação da investigação de tela preta | Área de `Xbox2Windows D-01` (superfície liberada durante cópia) só por proximidade |
| #3382 | shader: decode buffer FORMAT 47 (10_10_10_2 SSCALED) from hardware descriptors | Decodifica o formato 47 de buffer a partir dos descritores de hardware | Não |
| #3383 | shader: decode VOPC DPP16 and lower v_cmp quad swaps in the pixel path | Decodifica VOPC em DPP16 e baixa trocas de quad em `v_cmp` no caminho de pixel | Área de `victory-fork D-14`/`VItoriaBr D-14` (permutas de lane), por classe |
| #3384 | tooling: report compute dispatch refusals in triage summary | Relata recusas de dispatch de compute no resumo de triagem | Não (ferramenta) |
| #3386 | runtime: bind published typed buffer inputs with 32-bit components | Liga entradas de buffer tipado publicadas com componentes de 32 bits | Não |
| #3387 | startup: name each refused slot-19 output view interval | Nomeia cada intervalo de view de saída do slot 19 recusado | Não (diagnóstico) |

## Pull requests mesclados (4)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #3031 | startup: lower v_dot4_i32_i8 and v_dot4_u32_u8 in gfx2hlsl | `d418f70e` | Só `tools/startup/gfx2hlsl.py` (+56), fixture e checker; os três merges de `main` do ramo trazem doc/fixtures, não código novo |
| #3125 | shader: lower v_min_f16 and v_max_f16 in the direct compute path | `6685de7c` | `gfx1013_direct_compute.inl` +33, doc do experimento, fixture e 242 linhas de checker; três commits do ramo passam por dois merges de `main` (97 mil adições, quase todas documentação) |
| #3139 | runtime: recover the published disk pair when native candidate PSO creation fails | `7291aef2` **em `codex/native-graphics-integration`** | `d3d12_draw.inl` +99/−40 e 194 linhas de teste: candidato nativo marcava a tradução pronta antes de `pipeline_for` criar o PSO de desktop; quando `CreateGraphicsPipelineState` falhava, o draw devolvia nulo e cacheava o nulo, então o alvo ficava limpo mesmo havendo par de disco válido (medido: alvo 0 vermelho/32 limpos contra 16/16 com o par). Agora o candidato guarda `native_candidate`, `revision` e `rtv_request`; na falha recarrega o par publicado da mesma chave e variante RTV, recompila com revisão incrementada e recusa com diagnóstico se o par faltar/for inválido. `X2W_NATIVE_SHADERS_ONLY=1` mantém recusa sem fallback. **Não está na `main`:** `compare/7291aef2...main` devolve `diverged` (16 à frente, 1 atrás) |
| #3375 | shader: keep 9-bit mesh primitive indices | `f2972840` | Máscara 1023 → 511 em `gfx1013_direct_mesh.inl` (1 linha) e o fixture `0x20280600` com as três flags de aresta; o PR declara que `gfx2hlsl.py` já usava 511, então o desvio era só do frontend C++ |

## Cruzamento com o catálogo

- **Corrige D-xx?** Por hash, nenhum: os 11 hashes da faixa e os 4 de merge não intersectam os 270
  tokens de `catalogo/defeitos.csv`.
- **Abre defeito novo?** Candidato a linha, com evidência no diff e no corpo: `24b895d7`/`#3375` — o
  export primitivo de mesh tem três slots de 10 bits que carregam índice de **9 bits** com o flag de
  aresta no bit 9; o frontend C++ mascarava com `1023` e a guarda `triangle[c] >= vertices` descartava a
  primitiva. O defeito é de **contrato de campo**, não de valor: o formato está documentado em
  `docs/experiments.md` (linha de export primitivo) e o tradutor Python já usava 511. Não existe linha
  equivalente em `Xbox2Windows D-01..D-29` nem em `victory-fork`/`VItoriaBr`.
- **Observação de catálogo:** `#3139` trata exatamente a diferença entre "tradução pronta" e "PSO de
  desktop criado" — o mesmo par que `D-18` (`b87a99a3`/`b87a99a3` "compilação não retomada depois de
  rodada manifest-only") descreve por outro caminho. Não é a mesma linha e não a corrige.

## Confiança

Verificado: os 11 hashes, autores, assuntos, arquivos e contagens de linha (`repos/R0Y00/Xbox2Windows/commits/<sha>`);
o corpo e a lista de arquivos dos 4 PRs mesclados via `gh pr view`; a branch de base de cada merge
(`baseRefName`) e o teste `compare/7291aef2...main`, que prova que `#3139` não está na default.
**NAO VERIFICADO:** os números de teste citados nos corpos dos PRs (16 vermelhos/16 limpos, 104 testes
Debug, 4/4 Release, camada de debug indisponível em `0x887a002d`) — são afirmação do autor, não
reproduzidos aqui; que os três merges de `main` nos ramos do `v_dot4` (97 mil adições) não carreguem
mudança de código além de `gfx1013_direct_compute.inl`/`direct_compute_cache.cpp`, porque só li a lista
de arquivos com contagem, não o conteúdo. Contagem de merges x commits: 4 merges, 11 commits; a
diferença é explicada por 3 merges de ramo interno (conteúdo já presente como commit) e 1 merge fora da
`main`. Head no fim da coleta: `f2972840`.
