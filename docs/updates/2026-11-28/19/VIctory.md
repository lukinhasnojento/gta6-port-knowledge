# Marvisak/VIctory — atualização de 2026-11-28 19:00

## Resumo

Hora inteira de PRs, sem commit: a `main` continua parada em `63bcca43` desde antes do cursor. Sete PRs
novos acima de #3775, todos no tradutor, na captura de GPU ou no formato: `#3780` corrige a máscara
`FORMAT_SUPPORT` que anunciava `IA_INDEX_BUFFER` para **todo** formato, `#3776` dobra constantes half
conhecidas em tempo de tradução e `#3781`/`#3782` tiram recursão e trabalho repetido da análise de
interpolação e de ranges. Zero merges.

## Commits novos (0)

`compare/63bcca43...63bcca43` devolve `identical`, 0 à frente e 0 atrás. Nada a documentar.

## Pull requests abertos (7 acima do cursor #3775)

50 abertos lidos; o menor (#3729) continua acima do cursor anterior #3521, então o total real é ≥50 e o
cursor não prova cobertura dos antigos. Corpos lidos nos sete.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| `#3776` | Shaders: widen known half constants during translation | `emit/lower.rs` +43/-4: `known_half_widen(half)` monta o f32 exato de uma meia-palavra conhecida — sinal em `<<16`, expoente `(half>>10)&0x1F`, fração `half&0x3FF` — e emite o valor direto em vez da chamada de alargamento do DXIL quando o half é normal, zero com sinal ou infinito; subnormal e NaN continuam pela conversão do host. Regressão unitária compara as **65.536** codificações half com modelo numérico independente; 2.970 palavras de saída iguais à `main` em 297 módulos no WARP 1.0.22 | Não catalogado. Classe oposta a D-22 (`f32_to_half` com carry no expoente): aqui é half→f32 |
| `#3778` | Shaders: bound repeated dominance queries | `emit/lower.rs` +11/-23 e `ir.rs` +277/-10: um CFG só-para-frente é redutível **sem** cálculo de dominadores, e as consultas de dominador imediato deixam de ser refeitas por aresta de retorno; a ordem original de blocos e sucessores é mantida ao reportar aresta. O problema declarado é custo quadrático em função com muitas arestas de retorno | Não catalogado |
| `#3779` | GPU capture: use the PNG Up filter for coherent scanlines | `gpu/capture.rs` +74: `filter_png_rows` amostra 32 colunas × 24 linhas, mede variação horizontal e diferença contra a linha anterior, e escolhe o filtro **Up** quando as linhas são parecidas e variam entre colunas; imagem pequena (largura < 32 ou altura < 24) fica com o filtro antigo. O preditor é aplicado linha a linha no buffer cru | Não catalogado (diagnóstico) |
| `#3780` | GPU: report `IA_INDEX_BUFFER` only for unsigned index formats | `gpu/device.rs` +34: no `FORMAT_SUPPORT` do dispositivo convidado, o bit `IA_INDEX_BUFFER` (`0x4`) passa a ser limpo quando o formato não é `R32_UINT` (42) nem `R16_UINT` (57) — antes era ligado para todo formato. Comparação com a consulta nativa do D3D12 no WARP 1.0.22 / Agility 1.620.2, com nove controles (`R8_UINT`, `R32_FLOAT`, typeless, depth, bloco comprimido) | Mesma máscara de `#3771` (documentado às 18:00), que corrigiu `FORMAT_SUPPORT` de `DXGI_FORMAT_UNKNOWN`; nenhum D-xx do catálogo trata de `FORMAT_SUPPORT` |
| `#3781` | Shaders: walk interpolation dependencies iteratively | `emit/io.rs` +84/-31: a análise de interpolação de `io::Halves` deixa de percorrer o grafo de dependência da primeira metade por recursão com vetores crescentes de valores e sítios visitados; passa a caminhada iterativa com `HashSet` de visitados. Cadeias longas de joins estouravam a pilha do host e o custo crescia com os vetores | Não catalogado |
| `#3782` | Shaders: reuse comparison facts across phi inputs | `emit/range.rs` +128/-40: `Ranges::edge_hi` reextraía o suporte das mesmas condições de alcance de lane e rodava as mesmas consultas de implicação BDD para **cada** valor de entrada de um phi; os fatos passam a ser memoizados e compartilhados por `Rc` entre as entradas que dependem das mesmas condições | Não catalogado |
| `#3783` | Formats: apply XMA scale factors in place | `formats/xma.rs` +5/-4: os fatores de escala são aplicados direto em `ch.coeffs[..subframe_len]` antes da transformada inversa, em vez de um `[0f32; FRAME_SAMPLES]` temporário por subquadro. As bandas de fator cobrem o subquadro e a decodificação de coeficientes substitui essas entradas antes do próximo subquadro. O autor compara 1.152 quadros gerados (mono e estéreo, quatro taxas, três comprimentos de subquadro, três padrões estéreo): 1.053.696 amostras PCM e 1.769.472 palavras de saída bit a bit iguais | Área de `VIctory D-09`/`D-10`/`D-11` (XMA), mas não corrige defeito: é troca de buffer com saída declarada idêntica |

## Pull requests mesclados (0)

Nenhum PR fechado com `merged_at > 2026-11-27T13:20:45Z`.

## Cruzamento com o catálogo

- **Nenhum hash do catálogo aparece** nos sete PRs, e nenhum commit existe na janela.
- **`#3780` é a segunda passada na mesma máscara.** `#3771` (documentado às 18:00) corrigiu o valor de
  `FORMAT_SUPPORT` para `DXGI_FORMAT_UNKNOWN`; `#3780` corrige o **bit de índice**, que era ligado para
  todo formato. As duas mudanças compõem a consulta de capacidades do dispositivo, que o catálogo não
  cobre.
- **`#3783` fecha uma dívida de desempenho sem mudar saída.** O catálogo registra três defeitos de XMA
  (`D-09` laço infinito com quadro de comprimento 0, `D-10` soma antes de reduzir em `ADVANCE_XMA_READ`,
  `D-11` aritmética de layout) e nenhum deles é tocado aqui.
- **Abre defeito novo?** Não há sinal no diff. O risco de `#3781`/`#3782` é o de sempre em reescrita de
  análise (a mesma resposta por caminho diferente) e os dois trazem teste próprio; `#3776` declara
  cobertura exaustiva das 65.536 codificações.

## Confiança

- **Verificado:** os sete PRs por `repos/Marvisak/VIctory/pulls/<n>/files` (arquivos, ± linhas e até 7
  linhas de hunk por arquivo) e o corpo de cada um; `compare/63bcca43...63bcca43` `identical`; a lista de
  abertos com 50 itens e máximo #3783 às 19:01Z.
- **NAO VERIFICADO:** se os sete passam CI (nada no diff prova execução); os números medidos nos corpos
  (2.970 palavras, 1.053.696 amostras, nove controles) são do autor e não foram reproduzidos; o texto
  completo de `#3778` (só as duas primeiras linhas do corpo foram lidas).
- **Cursor:** `last_commit` fica em `63bcca43` e `last_pr_number` avança para #3783. A fila antiga (≥50
  abertos com piso em #3729) continua não coberta por inteiro.
