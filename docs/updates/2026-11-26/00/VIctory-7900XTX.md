# avatarmanz/VIctory-7900XTX — atualização de 2026-11-26 00:00

## Resumo

Sem commit novo: a `main` continua em `e2a28ec70632f5112be085abfa7d0a557b25e8d8`. Sete pull requests
abertos acima de 72, cobrindo orçamento de RAM do sistema para janelas ociosas, consertos no launcher,
três correções na forma wave64 de shaders de compute, sombras em cascata exatas em GPU wave32, LOD de
textura array e duas correções de infraestrutura (escrita fora da árvore e documentação de switches).

## Commits novos (0)

`gh api repos/avatarmanz/VIctory-7900XTX/commits?sha=main&per_page=100` devolve `e2a28ec7` como
primeiro elemento, igual ao `last_commit` do estado.

## Pull requests abertos (7)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 77 | GPU: opt-in system-RAM budget for idle windows | `host::RamBand`/`ram_band()`/`ram_sample()` classificam pressão de RAM do processo e da máquina em Calm/Watch/High por working set, private bytes, RAM física livre e folga de commit, com histerese. `VICTORY_WINDOW_BUDGET_MB` (desligado por padrão) limita os bytes de janelas ociosas únicas em Watch e reduz em High (`host.rs` +139, `shadow.rs` +99). Não mexe em pagefile. | Não é D-xx. `D-13`/`D-04` tratam de consumo de memória no fork Belobug; aqui o alvo é RAM do sistema, e o PR diz explicitamente que o watermark de `#72` só lia VRAM `LOCAL`. |
| 78 | Drop the invalid play.ps1 skip default and fail on missing executables | Remove o preset `VICTORY_SKIP_PIPELINES=none` de `scripts/play.ps1` (o parser Rust já avisa e trata ausente como lista vazia) e faz os dois caminhos de binário ausente saírem com código 1 em vez de 0. Refs `#25`. +3/-3 | Parcialmente: toca `play.ps1`, que é o tema de `D-19` ("a configuração documentada não é a medida: `play.ps1` liga ~20 switches"). Não corrige os defaults de switch; corrige um valor inválido e códigos de saída. |
| 81 | wave64: never emit a barrier a group cannot reach, and cover the whole group in the wave-wide decision | Quatro mudanças na forma `widen` (`emit/lower.rs` +206, `emit/mod.rs` +27): `group_sync` registra `need(block)` para bloco que não é all-lane e `widen` recusa o programa quando nada resta a alargar; branches que saem de laço com barreira também são tomadas wave-wide (`loop_exits_around`). Medido em RTX 3080 Ti, árvore `0a43566`. | Não é D-xx deste repositório, mas é a mesma classe de `VItoriaBr D-20` ("device loss introduzido pela atualização wave64: barreiras dentro de branches/loops") e de `D-11` (helper lanes fora das operações de wave na AMD). O PR descreve a barreira inalcançável como `0x887A0006` garantido. |
| 82 | Translate tiled cascade shadows exactly on wave32 GPUs | Porta o lowering exato de sombra em cascata de Bruno Magalhães: reconhece a forma de salto longo da saída antecipada e segue cópias escalares por joins onde toda entrada resolve para a mesma fonte (`ir.rs` +93, `wave64_tests.rs` +239). Antes, os quatro programas de cascata emitiam DXIL válido mas caíam no fallback aproximado por metade de wave. | Área de `D-11` (sombras com buracos quadrados por causa de helper lanes), defeito diferente: aqui é exatidão da forma de salto, não participação de helper lane. |
| 87 | Shaders: fix LOD queries on array textures | Em `image_get_lod`, baixa só as coordenadas espaciais: `calculateLOD` do DXIL quer uma coordenada para `Texture1DArray` e duas para `Texture2DArray`, e os operandos não usados têm de ser `undef`. O slice continua baixado nas amostras normais. Fixes `#74`. Regressões com programas de pixel sintéticos que exportam LOD com e sem clamp. | Não. `XCL D-16` (máscara wave64 só na metade baixa) e `D-17` (máscaras inline/NULL) são outros defeitos; nenhum D-xx deste repositório trata de `image_get_lod`. |
| 89 | VICTORY_WRITE_SITES: refuse a file inside the source tree (#33) | `titlewrite::learn` resolve o destino uma vez por `victory_core::ensure_outside_repo` e guarda em `OnceLock`; caminho recusado gera um único `warn!` e nenhuma thread nova. `victory-gpu` passa a depender de `victory-core` (sem ciclo). Fixes `#33`. +39/-4 | Não. É a única escrita que faltava passar por `ensure_outside_repo`; o problema é que `cargo xtask policy` aceitaria o `.txt` gerado dentro do checkout. |
| 90 | Docs: switches as the code reads them, and an xtask check for dead names (#49) | Confere os 73 nomes `VICTORY_*` do Markdown rastreado contra o código, corrige as tabelas (`docs/README-fxceless.md` +28, `findings.md`) e adiciona `xtask/src/switches.rs` (+104) que falha em nome morto. Achado maior: vários switches que a documentação chama de "off by default" estão ligados. Fixes `#49`. | Sim, em parte: ataca a raiz de `D-16` ("duas convenções opostas para o mesmo tipo de switch, em 294 variáveis") e de `D-19` (documentação que não reflete a configuração medida). Não é o conserto de nenhum dos dois — o verificador garante que o nome existe, não que o default documentado seja o default real. |

## Pull requests mesclados (0)

Nenhum PR com `mergedAt` posterior a 2026-11-26T02:11:35Z. A API devolve `mergedMax = 0`: os commits
entram direto na `main`.

## Cruzamento com o catálogo

- Corrige D-xx? Nenhum dos 19 itens deste repositório tem hash de correção novo nesta faixa. `#78` e
  `#90` encostam em `D-16`/`D-19`, mas por documentação e validação, não por mudança dos defaults.
- Abre defeito novo? Não verificável. `#77` está desligado por padrão e mantém a varredura atual quando
  a variável não existe; `#81` declara recusar em vez de emitir programa inválido, o que reduz risco.

## Confiança

Verificado: topo da `main`, ausência de delta, números, títulos, corpos, arquivos e linhas dos 7 PRs,
ausência de merge na janela.

Não verificado: as medições citadas nos corpos (`#81` em RTX 3080 Ti árvore `0a43566`, `#82` em RTX
5070 driver 621.24, `#87` em RTX 4090 sob Wine) não foram reproduzidas aqui. `#87` declara uma falha
remanescente em `stub_pipelin…` na suíte Win64 sob Wine — não abri o log, então o estado final da suíte
é NAO VERIFICADO.
