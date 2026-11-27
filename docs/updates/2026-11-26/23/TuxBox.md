# brcly/TuxBox — atualização de 2026-11-26 23:00

## Resumo

Trinta e cinco commits e doze PRs mesclados em cerca de cinco horas, de nove autores. O grosso é
tradutor de shader (`s_quadmask`, `s_bitreplicate`, saveexec de nor/xnor/orn1, SAD de byte e quad,
`frexp/ldexp` de f16 exato, packs normalizados, interpolação de pixel, heaps de sampler de mesh) e
verificação de D3D12 (roots bindless de computação, CBV em cada offset de 16 bytes, publicação de escrita
de textura). Entra também o primeiro registro de execução da fase ProjectScarlett: chegou a Game Running e
morreu em falta de memória gráfica com fault de GPU.

## Commits novos (35)

Faixa `e76ff6a2..f48ce028`. Doze são commits de merge; os demais, commits de PR chegados à `main`.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `8d1ffb3a` | Nick Stallman | launcher: report an NVIDIA shader cache that has outgrown its index | 2 arquivos, +133/−2 (`project_tuxbox/host.py`, `docs/USAGE.md`, `tests/test_host.py`). O índice do driver (`.toc`) endereça o `.bin` com offsets de 32 bits: cabeçalho de 32 bytes e entradas de 24 bytes (hash de 16, offset, tamanho). Com o limite de 4 GiB, um `.bin` chegou a **8.304.209.601 bytes** com a última entrada terminando 11 KB depois de 4 GiB; os 3,7 GiB seguintes nunca eram achados e os pipelines eram recompilados a cada launch sem aviso. Na partida, compara cada `.bin` sob `GLCache` com o fim da última entrada do índice e relata o arquivo. |
| `680573c5` | Nick Stallman | d3d12 checks: read a root CBV at every 16-byte offset under the launcher's settings | 4 arquivos, +168/−10. Controle `root-cbv-offsets`: liga um root CBV em cada passo de 16 bytes de 0 a 256 e compara 24 palavras em cada; um host que arredonda o endereço para baixo lê outras palavras e o convidado nomeia os offsets errados. Motivo declarado: nada testava a leitura que `force_raw_va_cbv` (`5c5677b`) conserta — o controle `root-cbv-window` só liga em múltiplos de 256 e o teste do launcher checa a variável, não a leitura. |
| `2f0473bd` | Nick Stallman | launcher: wake the live shader worker for a request while jobs run | 2 arquivos, +71/−16 (`runtime_shaders.py`, teste). O worker "live" esperava nos futures com timeout de 50 ms e não escutava a notificação de diretório enquanto um job rodava, então um pedido publicado ao lado de uma compilação em curso esperava o próximo rescan. Um job terminado passa a escrever num pipe que a espera de notificação consulta junto com o descritor do inotify. |
| `49371604` | Pavel Kirilin | xshader: accept dead f16 interpolator position reads of lane-spill registers | 2 arquivos, +84/−1 (`rewrite/lanes.rs`, testes). |
| `5cbce05c` | Pavel Kirilin | Created lanes example. | 5 arquivos, +115/−1: `examples/lanes/` (README, `selftest.cpp`, `spill_fixture.py`), `build.py`. |
| `d26d8978` | Valdi320 | Merge commit 'd49808d6' into work/review-bindless-235-20261126 | Merge de 3 arquivos, +71/−1 (`runtime/d3d12/src/commands/compute_bindings.rs`, `device/compute_table.rs` e teste). É o hash que a API dá como `merge_commit_sha` do PR #235 — um merge interno da própria branch do PR, não um `Merge pull request #235`. |
| `2f7e0f8a` | Valdi320 | verify: exercise direct-indexed compute paging and close review gaps | 6 arquivos, +168/−8: `examples/lanes/` (README, `selftest.cpp`, `shader_fixture.py`), `project_tuxbox/verify.py`, `device/compute.rs`, `device/compute_table.rs`. |
| `6ce0aac8` | brcly | d3d12: run the bindless compute scenarios in the check, and cover their sampler | 7 arquivos, +206/−23. `lanes-bindless-compute` era opcional e `tuxbox.py check` o pulava; passa a rodar e também despacha `bindless-sampler`, um programa de console que amostra textura de heap com sampler literal que o root deixa de fora. Um root indexado direto com menos de dois DWORDs livres não cabe no root SRV do mapa de índices e passa a manter o root do convidado em vez de falhar a criação do pipeline. |
| `c97bfaca` | Manuel Smith Lazarte Alcocer | xshader: preserve pixel interpolation attributes and exports | 3 arquivos, +166/−11 (`rdna2/operands.rs`, `hlsl/params.rs`, testes). |
| `21ac7364` | Manuel Smith Lazarte Alcocer | xshader: resolve mesh sampler heaps along reaching paths | 3 arquivos, +52/−7 (`hlsl/emitter/resources.rs`, `hlsl/mesh.rs`, testes). |
| `ec4e0844` | Manuel Smith Lazarte Alcocer | xshader: prove guards derived from wave reductions | 5 arquivos, +45/−20: `check_translator_gpu.py`, `check_wave_guard.py`, `examples/wave_guard.rs`, `hlsl/wave_sum.rs`, testes. |
| `e9134b07` | Nick Stallman | d3d12: publish texture writes to plain data pages with a store | 6 arquivos, +302/−7 (`device/placement.rs`, `placement/guarded_copy.rs`, `placement/writeback.rs`, `survey.rs`, dois testes). Motivo: no Wine a escrita nativa é um pedido ao servidor para cada corrida de texels, mesmo no processo do provedor. Páginas comprometidas de leitura/escrita que **não** estão sob write-watch passam a receber uma cópia que relata fault em vez de levantar exceção — o título pode desmapear as páginas depois de consultadas. Write-watch, copy-on-write e páginas executáveis mantêm a escrita antiga. |
| `71cc7244` | Nick Stallman | lanes: check that a published texture write leaves padding, closed pages and memory past it alone | 3 arquivos, +197/−2 (`examples/lanes/` e `verify.py`). O autoteste posiciona uma textura RGBA8 tiled de 100×60 sobre memória comum e sob write-watch, aprende onde cada texel mora por leitura, fecha uma página e manda a GPU escrever a textura quatro vezes enquanto uma fila de computação espera o label da última. Se a escrita publicada aparece no write watch depende do host, então só as páginas observadas depois da textura têm de ficar sem registro. |
| `158e08db` | JohnVonWells | xshader: lower s_quadmask, s_bitreplicate and nor/xnor/orn1 saveexec | 9 arquivos, +784/−11. `s_quadmask_b32/b64` e `s_bitreplicate_b64_b32` entram nas operações escalares comuns, usando a prova de reaching-definition existente: só para valores escalares comuns; aplicadas a EXEC ou VCC são operações de quad de pixel e continuam recusadas. `s_quadmask` liga SCC pelo resultado; `s_bitreplicate` o preserva; `wide_bit_result` trata `s_bitreplicate_b64_b32` como resultado par. `s_nor_saveexec`, `s_xnor_saveexec` e `s_orn1_saveexec` entram em b3x. |
| `215b5b1a` | brcly | Merge pull request #280 from NickStallman/launcher/nvidia-cache-index-check | Merge de 3 arquivos, +133/−2: leva `8d1ffb3a`. |
| `e2cad28e` | brcly | Merge pull request #282 from NickStallman/tools/root-cbv-offsets-control | Merge de 4 arquivos, +168/−10: leva `680573c5`. |
| `9b90087f` | brcly | Merge pull request #283 from NickStallman/perf/live-worker-wake | Merge de 2 arquivos, +71/−16: leva `2f0473bd`. |
| `8c56463a` | brcly | Merge pull request #281 from smith-3/codex/shader-fixes-upstream-1126 | Merge de 10 arquivos, +263/−38: "xshader: fix pixel interpolation, mesh sampler heaps, and reduction guards" — leva `c97bfaca`, `21ac7364` e `ec4e0844`. |
| `87be1861` | brcly | Merge pull request #278 from s3rius/mesh/lane-spill-dead-position-use | Merge de 6 arquivos, +124/−6. Mantém a cena `lane-spill-interp` e o fixture de Pavel Kirilin (que exercitam na GPU a decodificação de interpolador que o #281 corrige) e a põe em `lanes-raster-tables`; **não** mantém a mudança de tradutor, porque o #281 corrige a causa no decodificador — o campo SRC0 do interpolador é seletor de atributo, não registrador de posição. |
| `12e4b064` | brcly | lanes: match the spill scenario's pixel input to the vertex output | 1 arquivo, +1/−1. O programa de pixel declarava `TEXCOORD1`, que o programa de raster não escreve, e o pipeline não podia ser criado; com `TEXCOORD0` falha antes do #281 e passa depois. |
| `a682eb81` | brcly | Enhance README with Windows compatibility details | 1 arquivo, +5/−5: tabela de compatibilidade com o progresso em Windows. |
| `ab644f02` | brcly | docs: record the latest ProjectScarlett preparation and run, and open work | 6 arquivos, +108/−25, incluindo `docs/progress.svg` e dois `evidence/projectscarlett-*.json`. A preparação de 26 de novembro compilou **36.653 de 41.064** pedidos registrados; um launch em 27 de novembro chegou a **Game Running**, ficou sem memória gráfica e terminou em fault de GPU. `FOCUS.md` ganha o trabalho de memória (orçamento de 13,5 GB do console, posse, transbordo para disco, memória gráfica), velocidade de compilação no primeiro launch, como atacar shaders não suportados e as regras para Windows e Proton. |
| `dd9a9082` | JohnVonWells | xshader: lower masked and quad byte SADs | 17 arquivos, +971/−9. Nova `quad_sad`: `v_qsad_pk_u16_u8`, `v_mqsad_pk_u16_u8` e `v_mqsad_u32_u8`, mais as formas inteiras. A janela k do par S0 são os bits `[8k+31:8k]`; o SAD de cada janela contra S1 vai para o campo acumulador de 16 ou 32 bits correspondente de S2. Toda palavra de resultado é calculada antes de qualquer escrita, então destino pode sobrepor S0 ou S2. Só VGPR simples sem clamp/op_sel/negação, e só computação. A análise de live-in passa a conhecer as palavras de fonte extras e os resultados mais largos. Execução nativa em RDNA2 em `tools/native_rdna2`. |
| `e712c537` | JohnVonWells | xshader: lower exact f16 frexp/ldexp and normalized 16-bit packs | 16 arquivos, +3.313/−8. `v_frexp_mant_f16`, `v_frexp_exp_i16_f16`, `v_ldexp_f16`, `v_cvt_pknorm_i16_f16`, `v_cvt_pknorm_u16_f16` e `v_cvt_pknorm_i16_f32` numa `half_exact` nova, sobre os bits armazenados e seguindo o MODE (campo de arredondamento f16, denormais f16, quieting de NaN IEEE). Os packs f16 reusam `tb_norm_f16`; `tb_norm_f32` arredonda o produto exato uma vez. **Corrige `v_cvt_pknorm_u16_f32`**, que arredondava `x * 65535` para float antes de arredondar para inteiro e podia errar por um perto do empate. |
| `6fe69f44` | brcly | Merge pull request #286 from JohnVonWells/xshader/masked-sad | Merge de 15 arquivos, +971/−9: leva `dd9a9082`; conflito em `tools/check_translator_gpu.py` resolvido no merge. |
| `ba7ea033` | brcly | Merge pull request #285 from JohnVonWells/xshader/scalar-quad-masks | Merge de 12 arquivos, +784/−11: leva `158e08db`. |
| `bde76ad4` | brcly | Merge pull request #287 from JohnVonWells/xshader/exact-half | Merge de 15 arquivos, +3.311/−6: leva `e712c537`; conflitos em `check_translator_gpu.py`, `native-rdna2/README.md`, `hlsl.rs` e `emitter/vector.rs`. |
| `d8c43276` | brcly | Merge pull request #288 from smith-3/fix/xshader-vop3-s0-and-mesh-readlane | Merge de 5 arquivos, +62/−2: leva `60454d7b`, `3cf39196`, `6ca88fbe`. |
| `60454d7b` | Manuel Smith Lazarte Alcocer | xshader: ignore unused VOP3 s0 padding in lower_first_lane_reads across all pixel shaders | 3 arquivos, +42/−2 (`driver/pipeline.rs`, `rewrite/lanes.rs`, testes). |
| `3cf39196` | Manuel Smith Lazarte Alcocer | xshader: treat v_readlane_b32 destination as SGPR in mesh launch validation | 1 arquivo, +4/−0 (`hlsl/mesh.rs`). |
| `6ca88fbe` | Manuel Smith Lazarte Alcocer | xshader: add regression test for v_readlane_b32 destination in mesh validate_launch | 1 arquivo, +16/−0 (teste). |
| `8c0773b6` | Jeffrey C | d3d12: recover a mesh pixel program's literal samplers under a root with no static samplers | 2 arquivos, +18/−5 (`device/compute_table.rs`, `device/stream.rs`). Um programa de pixel de console monta samplers de palavras literais, então o root do título pode omiti-los por completo; contra um root sem samplers estáticos, a tradução lê o estado literal de um sampler privado (espaço 32765). Os roots de vértice e pixel acrescentam esses samplers quando refletidos (`inline_sampler::pixel`, depois `inline_sampler::private`), mas o caminho de mesh pedia só `inline_sampler::unmatched`, que deixa um root sem estáticos para aqueles dois. |
| `f3ed8250` | Jeffrey C | lanes: draw through mesh pipelines whose root has no static samplers | 5 arquivos, +67/−10. Nova cena `mesh-sampler-rootless`: desenha pelo programa `mesh-samplers` sob o root de duas tabelas sem samplers estáticos, com dois programas de pixel que amostram com o estado linear literal — o `raster-samplers`, que declara só s0, e o de dois samplers com feature part vazia (SFI0), que mostra que ele não lê sampler do heap. |
| `025a0f38` | brcly | Merge pull request #290 from NickStallman/fix/215-writeback | Merge de 9 arquivos, +499/−9: leva `e9134b07` e `71cc7244`, mais `examples/lanes` e `verify.py`. |
| `f48ce028` | brcly | Merge pull request #291 from JeffreyBytes/fix/mesh-rootless-literal-samplers | Merge de 7 arquivos, +85/−15: leva `8c0773b6` e `f3ed8250`. |

## Pull requests abertos (2)

Acima de 275, ambos documentados.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 277 | d3d12: coalesce redundant placed texture queue waits | `draft=true`. Reduz esperas duplicadas na mesma fence privada monotônica sem mudar sincronização exigida: um helper e três fases de espera separadas, guardando o maior tick elegível por fence; os filtros de self/completed/label-blocked ficam. Evidência declarada: head `6ea8f39` sobre `e76ff6a`, 3 testes do helper, seis grupos sem GPU e 17/17 gates de GPU NVIDIA. | Mesma área do **D-16** (deadlock entre filas por espera de placement atrás de um label), mas o PR reduz esperas redundantes; não é o deadlock. |
| 284 | xshader: accept uniform loops around compute swizzles for single-wave | O tradutor recusava qualquer swizzle de memória compartilhada dentro de laço ("compute swizzle inside a loop"), classe que atinge ~36 pedidos registrados do ProjectScarlett. Um grupo de wave única (32/64 lanes) com back-edge uniforme — SCC, EXEC ou incondicional — faz a troca rodar o mesmo número de vezes para toda invocação, e passa a traduzir; laços VCC divergentes e grupos multi-wave continuam recusados. | Não há id para essa classe de recusa no catálogo. |

## Pull requests mesclados (12)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| 235 | d3d12: page bindless compute roots through the resource index map | `d26d8978` | O hash da API é o merge interno da branch (`Merge commit 'd49808d6' into work/review-bindless-235`), com `commands/compute_bindings.rs` e `device/compute_table.rs`; o commit `2f7e0f8a` ("exercise direct-indexed compute paging") vem logo depois e é do mesmo autor. |
| 280 | launcher: report an NVIDIA shader cache that has outgrown its index | `215b5b1a` | `8d1ffb3a`: `host.py` compara `.bin` com o fim do índice e reporta. |
| 282 | d3d12 checks: read a root CBV at every 16-byte offset | `e2cad28e` | `680573c5`: controle `root-cbv-offsets`. |
| 283 | launcher: wake the live shader worker for a request while jobs run | `9b90087f` | `2f0473bd`: pipe + inotify numa espera só. |
| 281 | xshader: fix pixel interpolation, mesh sampler heaps, and reduction guards | `8c56463a` | `c97bfaca` + `21ac7364` + `ec4e0844`. |
| 278 | shader-compiler: mesh/lane spill dead position use | `87be1861` | Cena e fixture mantidos; mudança de tradutor descartada em favor do #281. |
| 287 | xshader: lower exact f16 frexp/ldexp and normalized 16-bit packs | `bde76ad4` | `e712c537`. |
| 288 | xshader: fix VOP3 operand handling in lower_first_lane_reads and mesh validate_launch | `d8c43276` | `60454d7b` + `3cf39196` + `6ca88fbe`. |
| 285 | xshader: lower s_quadmask, s_bitreplicate and nor/xnor/orn1 saveexec | `ba7ea033` | `158e08db`. |
| 286 | xshader: lower masked and quad byte SADs | `6fe69f44` | `dd9a9082`. |
| 290 | d3d12: publish texture writes to plain data pages with a store | `025a0f38` | `e9134b07` + `71cc7244`. |
| 291 | d3d12: recover a mesh pixel program's literal samplers under a root with no static samplers | `f48ce028` | `8c0773b6` + `f3ed8250`. |

São 12 PRs e 12 commits de merge na faixa (`d26d8978`, `215b5b1a`, `e2cad28e`, `9b90087f`, `8c56463a`,
`87be1861`, `bde76ad4`, `d8c43276`, `ba7ea033`, `6fe69f44`, `025a0f38`, `f48ce028`), sem divergência de
contagem.

## Cruzamento com o catálogo

- **D-21** ("assinatura mesh/pixel incompatível derruba o draw", `315b287`): `8c0773b6`/`f3ed8250` e
  `21ac7364` atuam no **caminho de mesh com root sem samplers estáticos** — samplers literais de programa
  de pixel de mesh recuperados em registrador privado. Não tocam a compatibilidade de assinatura que o
  D-21 descreve. **Não corrige o D-21.**
- **D-04** ("cache de pipeline da NVIDIA no lugar errado e com teto de tamanho", `9b18f7c`/`d10b369`):
  `8d1ffb3a` só **relata** o `.bin` que passou do fim do índice; não move o cache nem muda o teto.
  **Não corrige.**
- **D-05** (varredura O(n) case-insensitive do Wine na cache de shaders): nenhum commit da faixa toca
  `host.py` nesse caminho — o único commit de `host.py` é o do índice NVIDIA.
- **D-27** (slot 80 trata dimensões em threads como grupos, issue #164): `2f7e0f8a`/`6ce0aac8` exercitam
  paginação de compute indexada direto e roots bindless de computação; é o mapeamento de roots por mapa de
  índices, não a interpretação de dimensão do slot 80. **Não corrige.**
- **D-28** (`WaveSize(N)` incompatível recusado só no driver) e **D-29** (filas param em esperas de fence
  no story intro): as mudanças de wave são de semântica (`s_quadmask`, `v_readlane`, saveexec), não de
  `WaveSize`; `#277` mexe em esperas de fence de textura, não na parada do story intro.
- **Abre defeito novo?** Dois candidatos:
  - `e9134b07` troca a exceção por uma cópia que **relata fault** nas páginas comprometidas de
    leitura/escrita fora do write-watch. A justificativa é que o título pode desmapear páginas depois de
    consultadas — ou seja, o código passa a depender de as consultas de `VirtualQuery` envelhecerem de
    forma segura; se uma página for desmapeada entre a consulta e a escrita, o caminho novo é o único que
    não levanta exceção. Não há teste para esse intervalo.
  - `ab644f02` registra a execução que **ficou sem memória gráfica e terminou em fault de GPU** depois de
    chegar a Game Running. É evidência de defeito aberto com arquivo e números (36.653 de 41.064 pedidos
    compilados), não uma correção.

## Confiança

Verificado: os 35 SHAs e metadados em `repos/brcly/TuxBox/commits?sha=main&per_page=100`, com o cursor
`e76ff6a2` na posição 35; cada commit lido em `commits/<sha>` (arquivos, linhas e mensagem completa); os
12 PRs mesclados em `pulls?state=closed&sort=updated&per_page=100` com `merged_at` acima de
`2026-11-26T21:27:17Z`, e cada `merge_commit_sha` encontrado na lista de commits da `main`; os 7 PRs
abertos de `gh pr list --limit 50`, dos quais 2 acima de 275.

Não verificado: o conteúdo dos patches de `dd9a9082`, `e712c537`, `158e08db`, `e9134b07` e `71cc7244`
além de mensagem e lista de arquivos — as descrições de mecanismo saem do corpo do commit. Não verificado
que o hash `d26d8978` (PR #235) seja o commit de merge criado pelo GitHub: a mensagem é de um merge
interno da branch do PR, e a lista de commits da `main` não traz um `Merge pull request #235`. Não
verificado o número "~36 pedidos" do PR #284 nem os gates de GPU citados no #277: são afirmações dos
corpos. Nenhum teste foi executado aqui; os 13 repositórios monitorados são somente leitura.
