# Marvisak/VIctory — atualização de 2026-11-27 16:00

## Resumo

Nenhum commit e nenhum merge na `main` nesta rodada: o head continua em `63bcca43c6`, parado desde antes do
cursor, e a coleta REST de pulls fechados não achou merge com `mergedAt` maior que `2026-11-26T04:15:00Z`.
O delta do repositório é só de PRs abertos: 50 acima do cursor 3057 (números 3456–3521), o que é o piso da
janela de 50 — há mais acima, não vistos.

## Commits novos (0)

`compare/63bcca43...63bcca43` não se aplica: o head da `main` é igual ao `last_commit` do estado. Nada a
documentar.

## Pull requests abertos (50)

Janela de `gh pr list --repo Marvisak/VIctory --state open --limit 50` (50 PRs, 3456–3521), filtrada por
número maior que 3057. A coluna "Proposta" vem do título e do nome da branch; nenhum patch foi lido.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 3456 | GPU: store HostHeap::kept sparsely by slot index to save unpopulated slot RAM and speed up evict | guarda `HostHeap::kept` de forma esparsa por índice de slot | Não consta |
| 3457 | GPU: add opt-in LRU eviction for unreferenced shadow buffer copies (VICTORY_SHADOW_BUDGET_MB) | despejo LRU opcional de cópias de shadow buffer sem referência | Relaciona-se à família de D-07/D-08 (espelho de recursos), outro mecanismo |
| 3458 | HLE: reject null out pointers before allocating in DirectStorage factory | recusa ponteiro de saída nulo antes de alocar | Não consta |
| 3459 | Shaders: sort SDK dxil.dll versions numerically and ignore empty VICTORY_DXIL | ordena versões do `dxil.dll` numericamente | Não consta |
| 3460 | feat(shader,gpu): add wave64 lowering, own-bits support, and bitcode shader optimizations | lowering wave64 e otimizações de bitcode | Não consta (D-01..D-27 não cobrem wave64) |
| 3461 | GPU: report staged texture bytes in the frame line | reporta bytes de textura em staging na linha de frame | Não consta |
| 3462 | fix(shader,gpu): add universal DXIL sampler/signature fixes and Wave32-safe wave policy | consertos de sampler/assinatura DXIL e política segura para wave32 | Não consta |
| 3463 | Game: use checked arithmetic in shaderpack::Pack::parse and input_layout | aritmética checada ao analisar o pacote de shaders | Não consta |
| 3464 | HLE: match Windows executable paths case- and separator-insensitively on relaunch | compara caminho do executável sem diferenciar caixa/separador no relançamento | Não consta |
| 3466 | GPU: echo requested format and report required count in DXGI GetDisplayModeList | devolve o formato pedido e a contagem exigida em `GetDisplayModeList` | Não consta |
| 3467 | GPU: support D3D12_FEATURE_ARCHITECTURE1 and reject empty feature_levels | suporta `FEATURE_ARCHITECTURE1` e recusa lista de níveis vazia | Não consta |
| 3468 | GPU: publish numeric descriptor bases in 16-byte view-address records | publica bases numéricas de descritor em registros de 16 bytes | Relaciona-se a D-02 (slot de tabela aliasando slot dinâmico), outro recorte |
| 3469 | fix(gpu): add integer RT blend bypass and merged readback run handling | bypass de blend em RT inteiro e readback de run mesclado | Não consta |
| 3470 | perf(gpu,hle,loader,shader): add opt-in in-place heap, descriptor marks, raw CBV, and sharded journal | heap in-place opcional, marcas de descritor, CBV cru e journal fragmentado | Não consta |
| 3471 | fix(hle,loader,gpu): add storage/save safety guards, loader helpers, and bounded DRED diagnostics | guardas de segurança de storage/save e diagnóstico DRED limitado | Relaciona-se a D-13 (perda de saves), sem provar que fecha o caso de configuration id vazio |
| 3472 | perf(gpu): add texture staging throttles, ripe-block guards, and post-upload slot visibility | limita staging de textura e torna visível o slot após o upload | Relaciona-se a D-08 (cópia escrita pela CPU não enviada antes de `CopyResource`) |
| 3473 | GPU: add opt-in VICTORY_BURY_LAG for delayed resource retirement | aposentadoria atrasada de recurso, opcional | Não consta |
| 3475 | GPU: name the title slots a capped host heap could not hold | nomeia os slots que um heap limitado não pode manter | Diagnóstico de D-02 (teto de 1.000.000 do host), não a correção |
| 3476 | GPU: clamp view-heap indices below what the host device takes (#3330) | limita índices do heap de views ao que o host aceita | D-02 é exatamente o teto de views do host; este PR trata o índice, não o slot de tabela — **provável sobreposição, não verificada** |
| 3477 | Formats/coalesce linear texture copies | junta cópias lineares de textura | Não consta |
| 3478 | gpu: a texture placement reaps the pending ranges only when no heap has room | só colhe faixas pendentes quando não há espaço | Não consta |
| 3479 | Shaders: probe v_cvt_f32_i32 at its rounding midpoints | testa `v_cvt_f32_i32` nos pontos médios de arredondamento | Família de D-22 (arredondamento em conversão f16), operação diferente |
| 3482 | Capture: avoid overflow in world-relative frame matching | evita overflow no casamento de frames relativo ao mundo | Não consta |
| 3483 | perf(shader): group signature elements by row for interpolation normalization | agrupa elementos de assinatura por linha | Não consta |
| 3484 | GPU: fast-skip clean/idle bindless textures and lazy-reset upload list | pula texturas bindless limpas e reseta a lista de upload sob demanda | Não consta |
| 3485 | GPU: experiment with allocation-level write-watch classification (FIND-44) | classificação de write-watch por alocação, experimental | Não consta |
| 3487 | GPU: batch submission heap/memory steps and fast-scan bindless and page-table bitmaps | agrupa passos de heap/memória e varre bitmaps mais rápido | Não consta |
| 3488 | HLE: a view mapped during a physical write joins it | uma view mapeada durante escrita física entra nela | Relaciona-se a D-07/D-08 (escritas da CPU e cópia de ida e volta) |
| 3489 | Shaders: lower full-mask pos1 layer and viewport exports | lowering de exports de layer/viewport em máscara cheia | Não consta |
| 3490 | GPU: stop advertising BC textures as render targets | deixa de anunciar BC como destino de render | Não consta (D-03 é descritor nulo com tipo errado) |
| 3493 | Shaders: lower gradient samples with an offset and LOD clamp | samples com gradiente, offset e clamp de LOD | Não consta |
| 3494 | HLE/GPU: mouse look (right stick + triggers) and cursor hiding | mouse look e esconder cursor | Não consta |
| 3495 | gpu: promote unsupported RT/UAV texture carrier formats and filter uncastable DXGI formats | promove formatos de transporte para RT/UAV e filtra formato inconversível | Não consta |
| 3496 | shader: lower compute constant lane masks via flat group index without WaveSize(64) | máscaras de lane constantes por índice de grupo | Não consta |
| 3499 | Game: name disabled blend states with separate alpha factors | nomeia estados de blend desligados com fatores alfa separados | Não consta |
| 3500 | Shader: a branch-only mask SCC is the console wave's "any lane" (#3491) | SCC de máscara usada só em branch vira "qualquer lane" | Não consta (D-24 é carry de `s_addc`/`s_subb`) |
| 3501 | Shader tests: check Wave64 binary64 sums after one half exits a loop | teste de soma binary64 wave64 após metade sair do laço | Não consta |
| 3503 | GPU: reuse the matched entry when finding a shadow buffer | reaproveita a entrada casada ao procurar shadow buffer | Família de D-07/D-08 |
| 3504 | HLE: a failed zlib request says why in its warning | o aviso de falha do zlib diz o motivo | Não consta |
| 3506 | Report compute thread-group dimensions in shader refusals and offline listings | reporta dimensões do grupo de threads nas recusas de shader | Não consta |
| 3507 | docs: index inline decisions, shortcuts and environment variables | índice de decisões, atalhos e variáveis | Não consta |
| 3508 | GPU: zero pipeline stream padding before library comparison | zera padding do stream de pipeline antes de comparar com a biblioteca | Relaciona-se a DexNom D-11 (escrita atômica da pipeline library), repositório diferente |
| 3511 | GPU: variable-rate shading (RSSetShadingRate) reaches the host | shading rate variável chega ao host | Não consta |
| 3512 | GPU: a heap sync leaves out the slots past the host heap | sincronização de heap ignora slots além do heap do host | D-02 (teto de views do host), outro recorte — **provável sobreposição, não verificada** |
| 3516 | GPU: takes make pages read-only outside the guard lock (VICTORY_GUARD_OUTSIDE, opt-in, draft for A/B) | páginas ficam somente-leitura fora do lock de guarda | Não consta |
| 3517 | GPU: report single-sample quality for BC formats | reporta qualidade single-sample de BC | Não consta |
| 3518 | gpu: remap title view slots past capped host heaps | remapeia slots de view além de heap limitado | D-02 (teto de views), outro recorte — **provável sobreposição, não verificada** |
| 3519 | GPU: retry prelude reset with a fresh allocator on E_OUTOFMEMORY | repete o reset do prelúdio com alocador novo em `E_OUTOFMEMORY` | Não consta |
| 3520 | Loader: a fault while reporting one is not reported again | uma falha durante o relatório de outra não é reportada de novo | Não consta |
| 3521 | Match the primary shader container by database hash during prebuild | casa o container primário de shader pelo hash da base na pré-construção | Não consta |

## Pull requests mesclados (0)

A coleta REST (`pulls?state=closed&sort=updated&direction=desc`, primeira página de 100) não devolveu
nenhum PR com `mergedAt` maior que `2026-11-26T04:15:00Z`. O repositório não mescla há mais de um dia,
apesar de manter 50+ PRs abertos.

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum merge, logo nenhuma correção entra. Entre os abertos, os únicos que tocam
  defeitos catalogados são `#3476`, `#3512` e `#3518`, todos na área de D-02 ("heap de views limitado pelo
  host, teto de 1.000.000 da NVIDIA"): eles contornam o teto (clamp, remap, sync), enquanto a correção
  registrada em `3827767` mudou a forma como o slot de tabela do título é endereçado. Se os três entrarem,
  a entrada D-02 precisa ser relida — **NAO VERIFICADO**, nenhum patch lido.
- **Abre defeito novo?** Não há commit para avaliar. Dois PRs merecem atenção quando entrarem:
  `#3476` (clamp de índice) e `#3518` (remap) podem mascarar em vez de corrigir o estouro de heap, e
  `#3473`/`#3457` (aposentadoria atrasada e despejo de shadow) mexem em ciclo de vida de recurso, área de
  D-07 (uso-após-liberar na GPU).
- **Pendência estrutural mantida**: D-03 (descritores nulos do tipo/dimensão errados, sem correção no
  catálogo) não é tocada por nenhum dos 50 títulos.

## Confiança

- **Verificado por metadado**: head da `main` igual ao `last_commit` do estado (`63bcca43c6`), checado
  pela API de branches; zero merges na janela, pela API REST de pulls fechados; 50 PRs abertos acima do
  cursor, pela listagem com `--limit 50` (números 3456–3521).
- **Não verificado**: todos os 50 diffs. A coluna "Proposta" é leitura do título e do nome da branch, e a
  coluna do catálogo marca "não consta" por ausência de correspondência no texto do título — não por
  inspeção do patch. Onde escrevi "provável sobreposição", é leitura de título.
- **Janela no piso**: a listagem de abertos traz exatamente 50, então o número real acima de 3057 é ≥ 50;
  o cursor avança para 3521 e os 50 foram catalogados um a um.
- **`last_run`**: sem merges, o cursor de mesclados avançou para o instante em que a consulta foi feita
  (`2026-11-27T13:20:45Z`), depois de a página de fechados ser lida inteira até passar do cursor antigo
  (o PR mais antigo da página tinha `updated_at` de `2026-11-26T05:51:18Z`, anterior ao cursor de
  `04:15:00Z`).
