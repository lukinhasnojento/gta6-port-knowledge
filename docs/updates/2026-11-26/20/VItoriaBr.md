# uiliannn/VItoriaBr — atualização de 2026-11-26 20:00

## Resumo

**2 commits** na faixa `e40fba44..1361e190`, ambos só em `README.md` — nenhum código mudou. O que entrou é
uma seção de “evidências para o PortWatch” com números medidos pelo autor (30 fps em menus, 4–6 fps no preset
padrão, 22 fps como melhor medição experimental num build do GPT, RTX 5060 Ti 1080p) e uma tabela de campos
que o PortWatch deveria revisar (`playable=1`, `story=0.25`, `render=0.95`, `fps=22`, `stability=0.55`),
declaradamente auto-reportados. Em paralelo, **12 PRs abertos novos** acima do cursor 111, quase todos do
mesmo autor, com mudanças grandes de GPU/shader que ainda não entraram. Nenhum PR mesclado na janela.

## Commits novos (2)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| a3600b5e | uiliannn | docs: add PortWatch evidence for the current NVIDIA build | Só `README.md` (+34/−2): substitui a seção de status de 2026-11-25 por “Status e Progresso (2026-11-26)” com uma tabela de sete campos (Playable, Story, Rendering, Performance, Stability, Hardware, Código). Números: 30 fps em menus/vídeos, 4–6 fps no preset atual da `main`, 22 fps na melhor medição de mundo aberto com build experimental **não integrado** ao preset (RTX 5060 Ti, 1080p), 26 599 containers de shader analisados e 25 344 PSOs aceitos. Diz explicitamente que o PortWatch classifica por leitura de README/commits e pode manter valores anteriores até revisão manual. |
| 1361e190 | uiliannn | docs: record the current render and PortWatch evidence fields | Só `README.md` (+6/−2): troca a linha “Rendering” por “~95 % do console (avaliação do autor)” e acrescenta a lista de campos que a evidência sustenta — `playable=1`, `story=0.25` (primeira missão), `render=0.95`, `fps=22`, `stability=0.55` — com a ressalva “These are self-reported; PortWatch should confirm them against the stream/logs”. |

## Pull requests abertos (12)

Sessenta PRs abertos na coleta (`newest` = #126); os 12 acima do cursor 111 estão abaixo. “Proposta” lida no
título e no conjunto de arquivos (GraphQL), sem leitura de patch. Todos com pelo menos um arquivo de código.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 126 | Tests: add structured Cargo test reports with explicit skip classification | 6 arquivos, +724: `scripts/test-report*.ps1` novos e `docs/test-report.md`; relatório estruturado de testes com classificação explícita de skip. | Não; é infraestrutura de verificação (vizinho de D-17 do `victory-fork`, dívida de verificação). |
| 125 | feat(gpu): pré-drena texturas na fila durante telas de loading (low-draw) e adiciona VICTORY_TEXTURE_PASS_MB | `draw.rs` +459/−46, `descriptor/host.rs` +10, `resource.rs` +22: passagem de textura durante telas de loading com orçamento por MB. | Vizinho de D-08 (textura indexada subia zerada) pelo caminho de upload; não é a mesma correção. |
| 124 | feat(hle,gpu,formats): I/O concorrente overlapped no DirectStorage com sequential scan e sincronização adaptativa de áudio/vídeo (VICTORY_AV_SYNC) | 7 arquivos, +415/−73: `frame.rs`, `queue.rs`, `apu.rs`, `bcpack.rs`; I/O concorrente e sincronia A/V adaptativa. | Não mapeado. |
| 123 | feat(shader,gpu): decodifica shaders ACO pré-compilados ACCC (FSPX) e destrava os 29/29 passes FFX_FSR4 no Series S | 9 arquivos, +1 327/−56: `emit/image.rs` +151, `emit/lower.rs` +106, `emit/mod.rs` +379, `dxbc.rs`; decodifica containers ACCC/FSPX. | Não; a alegação “29/29 passes” é do autor, não verificada. |
| 122 | feat(gpu,shader): implementa ClearUnorderedAccessViewUint/Float (0x1a8/0x1b0) e fallback de contêiner XDAT sem DXIL | 9 arquivos, +540/−56: `descriptor/host.rs` +221, `draw.rs` +77, `list.rs` +129; implementa os dois clears e o fallback XDAT. | Não. |
| 121 | feat(gpu): corrige layout de 72 bytes do PresentX em Series S e adiciona upscaling espacial AMD FSR 1.0 | 4 arquivos, +1 643/−205: `compose.rs` +1 312, `queue.rs` +258; corrige o layout do PresentX e adiciona FSR 1.0. | Vizinho de D-06 (present único não chega à tela é do `TuxBox`, não deste catálogo). |
| 120 | Ports: correções de shader, GPU e áudio de fxceless/victory-fork e do fork VIctory do AnyXboxX (16 commits, GPL-3.0) | 25 arquivos, +3 092/−185: `formats/xma.rs` +341, `descriptor/host.rs` +307, `draw.rs` +127, `bcpack.rs` +33. | Vizinho de D-03 (laço infinito no XMA) — o PR toca `xma.rs`, mas o diff não foi aberto. |
| 119 | feat(gpu): adiciona presets VICTORY_QUALITY (low/medium/high/ultra) para orçamento de texturas e iluminação ambiente | 5 arquivos, +220/−23: `lib.rs` +192, `ambient.rs` +2/−17, `resource.rs`; presets de qualidade. | Não. |
| 117 | fix(gpu): corrige null CBV no create_cbv para evitar DXGI_ERROR_DEVICE_HUNG (0x887A0006) no AMD RDNA2 | `descriptor.rs` +29/−4, `resource.rs` +8: evita CBV nulo na criação. | **Vizinho direto de D-05** (heap de descritores esgotado lia o descritor do título como CBV → remoção do device na AMD, corrigido em `3c81420`). Mesma classe de falha, caminho de criação. |
| 116 | feat(cli): adiciona atalho VICTORY_PRESET=8gb para GPUs de 8 GB VRAM + HDD | `main.rs` +188, README +18/−24: preset de 8 GB. | Não. |
| 114 | docs: add English keyboard controls table | Só `README.md` +11. | Não. |
| 112 | docs: document per-stage shader loop limits | Só `README.md` +2/−2. | Não. |

## Pull requests mesclados (0)

Nenhum. `gh pr list --state merged --limit 100 --search "merged:>=2026-11-26"` devolveu zero para este
repositório, e nenhum merge aparece na faixa de commits. `last_run` permanece `2026-11-26T04:15:00Z`.

## Cruzamento com o catálogo

- **Corrige defeito do catálogo?** Nenhum. Os dois commits são documentação; um deles (`1361e190`) apenas
  publica autoavaliação.
- **Abre defeito novo?** Um candidato de **documentação**, **NAO VERIFICADO** como defeito de código:
  `a3600b5e` + `1361e190` colocam no README números que o próprio texto declara auto-reportados
  (`render=0.95`, `stability=0.55`, `fps=22`) como “evidência pública para auditoria” de um tracker externo.
  O catálogo já tem **D-21** (“Deriva de documentação: números retirados, contradição sobre TDR, contagens
  divergentes”, introduzido em `0603123`, corrigido em `b37344f`). A forma é a mesma: número no README sem
  medição reproduzível atrás. A diferença é que aqui o número vem rotulado como autoavaliação — o que reduz
  o risco de ser lido como fato, mas não o elimina se o tracker consumir o campo sem o rótulo.
- **Nota de consistência com D-21:** o commit anterior de documentação (`0603123` → corrigido em `b37344f`)
  tratava de números retirados e contagens divergentes. `1361e190` mantém a contagem de containers
  (26 599) e de PSOs (25 344) separada por hardware (RTX 5060 Ti), o que é coerente com a correção de
  D-21 — não encontrei contradição entre `a3600b5e` e o texto anterior, só substituição da seção de status.

## Confiança

- **Verificado por diff:** os 2 hashes em clone local `--filter=blob:none` de `uiliannn/VItoriaBr` (commit
  `1361e190`). Ambos tocam **apenas** `README.md` (34+/2− e 6+/2−): nenhuma alteração em `crates/`, `apps/`
  ou `scripts/`.
- **Cobertura:** 2 de 2 commits. `last_commit` avança para `1361e190` (= head).
- **PRs:** 12 de 60 abertos documentados (`last_pr_number` 111 → 126). Nenhum mesclado; `last_run` inalterado.
- **NAO VERIFICADO:** todos os números do README (fps, “~95 % do console”, contagem de PSOs) — são
  auto-reportados e não há log, captura ou comando reproduzível na faixa; o conteúdo real dos 12 PRs abertos
  (só título e arquivos foram lidos); se o PR #117 trata do mesmo caminho de D-05.
