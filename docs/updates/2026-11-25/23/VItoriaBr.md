# uiliannn/VItoriaBr — atualização de 2026-11-25 23:00

## Resumo

Um commit novo de 305 linhas que porta a composição do segundo plano de present do título (a
interface da Series S sobre a imagem do jogo), atribuído a `fxceless` PR #1189 e a
`trustno1-coder/VIctory-NVIDIA 7997a20`. O mesmo problema aparece hoje em outros dois repositórios
monitorados, com soluções independentes. Dois PRs abertos, ambos de `GaiuLulius`.

## Commits novos (1)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `85615ad3` | uiliannn | gpu: compose the title's second present plane (trustno1-coder/VIctory-NVIDIA 7997a20; fxceless PR 1189; GPL-3.0-or-later) | Cria `crates/victory-gpu/src/compose.rs` (245 linhas) com shader HLSL embutido: amostra o plano 0 (`t0`) e o plano 1 (`t1`) com sampler linear/clamp, escreve `u0` no tamanho do plano do jogo, alpha tratado como straight (`VICTORY_PLANE_PREMULTIPLIED=1` para pré-multiplicado). `SLOTS = 16` (três descritores por frame) para que presents em voo não tenham o descritor reescrito. `lib.rs` registra `mod compose` sob `#[cfg(windows)]`; `host.rs` ganha `present_over(texture, state, over, frame)` e `present_now` passa a receber `over`; `queue.rs` ganha `second_plane(planes, plane_parameters)`, que só devolve algo com `planes >= 2`, pega `plane_parameters.add(1)`, faz `host_copy` do recurso e o deixa em `COPY_SOURCE` (0x800). O comentário do módulo declara duas simplificações: as placement/source rectangles dos planos **não são lidas** (ambos cobrem a tela) e a interface é reduzida por filtragem |

Contexto cruzado: a medição citada no comentário (plano 0 1920x1080 com o jogo, plano 1 3840x2160 com
a interface, run de 2026-11-25) é a mesma família de achado que `R0Y00/Xbox2Windows` publica hoje em
`c99b7b89` — lá o plano 0 fica preto e o 1 tem a imagem, aqui o 0 tem o jogo. Os dois repositórios
descrevem o mesmo present de dois planos e chegam a layouts de registro diferentes; nenhum dos dois
foi verificado contra o outro nesta execução.

## Pull requests abertos (2)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 107 | gpu: enable levels-on-demand textures by default on capable adapters | Ligar por padrão cópias de textura com níveis sob demanda em GPUs com Tiled Resources Tier 2, economizando 2 a 3 GB de VRAM, com opt-out `VICTORY_LEVELS_O…` | Não consta no catálogo `VItoriaBr` (D-01…D-21). Fica na área de D-06 (`SHADER_HEAPS`, uso-após-liberar e contenção) e D-07 (spans de texturas sobrepostas) sem tocar nelas |
| 108 | shader: lower s_quadmask_b32 and s_quadmask_b64 | Reduzir cada grupo consecutivo de quatro bits a um bit e preservar SCC, para GFX10/RDNA2 | Não consta. É a mesma instrução que `R0Y00/Xbox2Windows` propõe hoje no PR #2678 — implementações separadas |

## Pull requests mesclados (0)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| — | — | — | nenhum PR com número > 106 aparece como mesclado |

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum hash desta faixa casa com o catálogo `VItoriaBr` (D-01 `c931d12` … D-21
  `b37344f`). O commit `85615ad3` cobre um comportamento que o catálogo não registrava.
- **Abre defeito novo?** Sim, um candidato claro: as *placement rectangles* e *source rectangles* dos
  planos não são lidas e a interface é sempre escalada para o tamanho do plano do jogo. Em qualquer
  título que use recorte parcial do plano de interface, a composição vai sobrepor a coisa errada — e
  o código não tem caminho de recusa, só o atalho documentado. O alpha também é presumido straight
  por padrão.
- O catálogo `VItoriaBr` continua com **D-17 aberto** (`d16` em buffer store reinterpreta inteiros de
  16 bits como `f16`, marcado SEM CORREÇÃO). Nada desta faixa o toca.

## Confiança

- **Verificado:** o commit `85615ad3`, arquivos, contagens e os trechos de `compose.rs`, `host.rs`,
  `queue.rs` e `lib.rs` citados (patch da API).
- **Verificado:** os 2 PRs abertos, números, títulos e autor.
- **NAO VERIFICADO:** se a composição funciona em execução — o repositório é somente leitura para
  esta rotina e nada foi compilado ou rodado.
- **NAO VERIFICADO:** o diff dos PRs #107 e #108.
