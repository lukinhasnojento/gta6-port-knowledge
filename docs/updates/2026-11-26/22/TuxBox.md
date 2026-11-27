# brcly/TuxBox — atualização de 2026-11-26 22:00

## Resumo

Um lote de treze PRs mesclado às 21:27:17Z, quase todo em tradução e hosts D3D12: roots de compute que
a reflexão não descreve, tabelas corroboradas em tabelas de compute posteriores, *literal samplers* de
pixel em pipelines mesh, `image_atomic_cmpswap` de 32 bits em UAV tipado, f64 (`cvt`, `min/max`,
`frexp`, `fract`, `ldexp`) e importação de backings XMem não contíguos. Junto vêm os commits dos autores
que compõem esses merges e dois consertos de robustez no host (device removal durante espera de fence e
retenção de extensões de tabela). A branch `main` avançou de `68e79b7e` para `e76ff6a2`.

## Commits novos (31)

Faixa `68e79b7e..e76ff6a2`, 31 commits — 17 de autores e 14 merges (treze PRs; #268 aparece duas vezes,
ver Confiança).

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `b141cbd3` | Valdi320 | xshader: resolve inline compute root SRVs by the actual root parameter | Novo `compute_root_raw_srv` mapeia root SRV inline pelo argumento físico de usuário (SGPR), não pelo registrador que o compilador nomeou: exige linha de usuário independente no FSSM corroborando a linha usada e a PSV antes de o root do pipeline renomear. `hlsl/emitter/resources.rs` (+38), teste `raw_width.rs` (+67), `check_raw_index.py --root-srv-remap`. |
| `f9b630db` | DrArroway | d3d12: import non-contiguous XMem backings through a reserved buffer | Importação de range de 64 KiB cujo backing XMem não é contíguo: cria heaps extras (`extra_heaps`) atrás de um buffer reservado, com ordem de drop carregada (`native` e `extra_heaps` antes de `_shadow` e `_backing`) e `wait_for_queue` com evento de fence próprio. `allocation.rs` (+88/−30), `backing.rs` (+156/−29), `import.rs` (+132/−6), teste de 255 linhas. |
| `54c2ce3e` | Valdi320 | dxbc: recover corroborated descriptors in later compute tables | `compute_table.rs` recupera descritores corroborados em tabelas de compute posteriores (+14/−3), com 50 linhas de teste e a sonda `check_third_table_atomic.py` (+89) registrada no verificador. |
| `eaea46d9` | Valdi320 | launcher: identify staged provider before comparing revisions | Novo `launch_provenance.py`: calcula SHA-256 da DLL D3D12 estagiada e de `build/d3d12-smoke/d3d12_xs.dll`, avisa se diferem e, com `--require-current-provider`, recusa lançar quando não batem. `docs/PROJECTSCARLETT.md` diz explicitamente o que isso não prova (revisão de build, módulos carregados pelo Wine). +138. |
| `a23c6e9f` | Valdi320 | xshader: permit proven bounded pixel code-table offsets | O emissor passa a aceitar dados de código embutidos em estágio de **pixel** além de compute (`hlsl/emitter/buffer.rs`), desde que a prova cubra todo o intervalo de endereços possível — não um valor de lane nem a largura da wave do host. Novo `check_pixel_code_offset.py` (+87) e teste `pixel_embedded_table_offset_is_bounded_and_aligned`. |
| `f806dd6c` | gtb-sbe | d3d12: translate root-less compute programs against a root reflection cannot describe | Para root que `compute_table::reflected` não descreve, o programa sem root embutido caía na tradução root-less e seus registradores vinham da reflexão do próprio programa, que o root não precisa declarar — o host rejeitava ("Failed to remap"). Passa a traduzir só contra o root do pipeline e, se recusado, usa stand-in como o braço refletido; programa com root embutido mantém a ordem. `compute.rs` (+19/−3), `shader_cache.rs` (+7), cenário `lanes-compute-root`, 7 arquivos. |
| `6623e3e2` | Valdi320 | d3d12: check device removal during prefix fence waits | Espera de fence de prefixo passa a sondar remoção de dispositivo a cada `PREFIX_POLL_MS = 100` ms, sem prazo novo para trabalho lento saudável; `GetDeviceRemovedReason` substitui o retorno cego de `Ok(())`. Novo módulo `fence_poll.rs` com `poll()` e teste de wakeup obsoleto. +91/−17. |
| `8f10bd13` | yefan911 | launcher: discover Proton in Flatpak and secondary Steam libraries | `host.py` inclui `~/.var/app/com.valvesoftware.Steam/.local/share/Steam` e lê `libraryfolders.vdf` de cada instalação (não só da primeira); teste cobre Flatpak com e sem sufixo `-x86_64`. +49/−4. |
| `ddb9e5dd` | Jeffrey C | d3d12: give a mesh pipeline's pixel stage the literal samplers its root leaves out | Um programa de pixel pode construir sampler de palavras literais que nenhum static sampler do root mesh casa; a tradução contra o root recusa e o caminho mesh caía no root-less, que lê samplers privados no espaço 32765 e o vkd3d-proton rejeita ("Failed to remap sampler 32765:0", `E_INVALIDARG`) — 204 pipelines mesh/pixel distintos no RADV. Monta a tabela mesh de novo com os estados faltantes como static samplers privados. `mesh_table.rs` (+15), `stream.rs` (+22), `compute_table.rs` (visibilidade). |
| `e09b1959` | Jeffrey C | lanes: draw through a mesh pipeline whose pixel sampler the root lacks | Cenário `mesh-samplers` no caminho mesh de #227: programa de pixel declara s0/s1 lineares e o root tem point; o mesh é container escrito à mão só com registros FSSM, nomeando `b501` onde o root declara `b500`. Fixture e verificação (+234/−47). |
| `c6a4887d` | Jeffrey C | d3d12: add a mesh pixel program's unmatched literal samplers when it translates too | Quando o programa de pixel declara exatamente um sampler, a tradução liga o literal àquele registrador qualquer que seja o estado do root, então a tradução **tem sucesso** e o retry nunca roda — o mesh amostra com o estado do root (linear clamp sob s0 point desenha point-filtered). Passa a acrescentar os estados literais não casados também nesse caso, como os roots de vértice/pixel já faziam (#241). +24/−13. |
| `1e3e6b46` | Nick Stallman | xshader: translate 32-bit image_atomic_cmpswap on typed UAVs | Os últimos "unresolved image" do ProjectScarlett eram dois programas de compute que fazem CAS em texel de UAV 3D uint: `image_atomic_cmpswap v[18:19], [v13,v14,v20], dmask:0x3, glc`. O emissor não conhecia a forma e caía no caminho de sampling, que não aceita UAV. DMASK 0x3 = 32 bits, DATA[0] valor gravado, DATA[1] comparado, GLC devolve o anterior em DATA[0]; mapeia para `InterlockedCompareExchange` com a ordem invertida do HLSL. `emitter/image.rs` (+34) e 137 linhas de teste. |
| `c4840113` | Nick Stallman | tools: check translated image_atomic_cmpswap on the GPU | Sonda em GPU real: dois programas de 32 lanes (llvm-mc gfx1030) num root cuja única tabela tem UAV 3D uint, rodando num guest novo com textura 4×4×4 `R32_UINT` e buffer de resultado; um caso faz CAS-retry concorrente (todos os bits ao fim, cadeia de anteriores), o outro compara acerto/erro. +354, 4 arquivos. |
| `e3262e02` | Nick Stallman | xshader: a guarded mesh half result does not read the launch bits no reader takes | 99 programas mesh recusados com "unsupported mesh launch register v7" (59) ou v6 (40): cada um escreve o registrador com `v_cvt_f16_f32`, que no RDNA2 grava 15:0 e mantém 31:16, então `live_in` conta a escrita como leitura do valor de launch. `kept_half_unread` reconhece que todo leitor posterior lê só 15:0 (`v_cvt_f32_f16`/`v_pack_b32_f16` sem `op_sel`) e deixa de recusar. `hlsl/mesh.rs` (+231), `mesh_exports.rs` (+4, `vertex_start`), teste (+200). |
| `99ffcf2f` | JohnVonWells | xshader: lower f64 conversions, extrema, frexp, fract and ldexp | Adiciona `v_cvt_f64_f32`, `v_cvt_f32_f64`, `v_min_f64`, `v_max_f64`, `v_frexp_exp_i32_f64`, `v_frexp_mant_f64`, `v_fract_f64`, `v_ldexp_f64` e o par f32 `v_frexp_*_f32` como operações de bits em pares uint2, para VGPR sem modificadores em compute, seguindo `MODE` (round, denormais, quieting de NaN); corrige `v_trunc/floor/ceil/rndne_f64`, que quietavam todo NaN sinalizante. `hlsl/float64.rs` (+262/−33), evidência nativa RDNA2 e 408 linhas de sonda. 14 arquivos, +4501/−196. |
| `e2e27dee` | Valdi320 | d3d12: retain original table extents when appending corroborated ranges | Ao anexar ranges corroborados a uma tabela de compute, preserva as extensões originais em vez de substituí-las (`compute_table.rs` +9/−1), com cenário de fixture e verificação (5 arquivos). |
| `210b28b4` | brcly | Merge pull request #270 from JohnVonWells/xshader/f64-conversions-extrema | Merge de `99ffcf2f`: f64 e frexp. +4501/−196 em 14 arquivos. |
| `0c51a552` | brcly | Merge pull request #265 from NickStallman/fix/xshader-unresolved-image-2 | Merge de `1e3e6b46`+`c4840113`: CAS 32 bits em UAV tipado e sonda de GPU. +528, 7 arquivos. |
| `21dbd0f6` | brcly | Merge pull request #260 from gtb-sbe/d3d12-compute-root-first | Merge de `f806dd6c`: compute root-less traduzido contra o root do pipeline. +406/−9, 7 arquivos. |
| `83abffda` | brcly | Merge pull request #253 from DrArroway/feat/d3d12-scattered-backing-reserved | Merge de `f9b630db`: backings XMem não contíguos por buffer reservado. +631/−72. |
| `1cab1a14` | brcly | Merge pull request #254 from Valdi320/work/critical-binding-proof-20261126 | Merge de `54c2ce3e`: descritores corroborados em tabelas posteriores + sonda `check_third_table_atomic`. +192/−3. |
| `04bba7cd` | brcly | Merge pull request #259 from Valdi320/work/device-aware-fence-wait-20261126 | Merge de `6623e3e2`: remoção de dispositivo durante espera de fence de prefixo. +91/−17. |
| `c1df9c1a` | brcly | Merge pull request #255 from Valdi320/work/launch-provider-provenance-20261126 | Merge de `eaea46d9`: proveniência do provider estagiado no launcher. +138. |
| `eed7c65c` | brcly | Merge pull request #262 from yefan911/fix/portable-launcher | Merge de `8f10bd13`: Proton em Flatpak e bibliotecas secundárias. +49/−4. |
| `6466bcf9` | brcly | Merge pull request #268 from NickStallman/fix/xshader-mesh-launch-2 | Primeiro merge de #268 (conflito resolvido em `tests/hlsl/mesh.rs`): `kept_half_unread` + `vertex_start` + 200 linhas de teste. +435. |
| `2c4a3b93` | brcly | Merge pull request #266 from JeffreyBytes/fix/mesh-literal-samplers | Merge de `ddb9e5dd`+`e09b1959`+`c6a4887d`, com conflitos em `examples/lanes/*` e `project_tuxbox/verify.py`: samplers literais no pixel de pipeline mesh. +283/−50, 9 arquivos. |
| `5979f8d8` | brcly | Merge pull request #252 from Valdi320/work/root-srv-register-resolution-20261126 | Merge de `b141cbd3`, com conflitos em `check_translator_gpu.py` e `raw_width.rs`: root SRV inline pelo argumento físico. +136/−14. |
| `fcfcabdc` | brcly | Merge pull request #269 from Valdi320/work/retain-reflected-root-extents-20261126 | Merge de `e2e27dee`: extensões originais da tabela preservadas. +206/−3. |
| `60497284` | brcly | Merge pull request #256 from Valdi320/work/pixel-buffer-offset-20261126 | Merge de `a23c6e9f`, com conflito em `check_translator_gpu.py`: offsets de code-table em pixel provados. +115/−1. |
| `e56f9ee4` | Nick Stallman | xshader: a guarded mesh half result does not read the launch bits no reader takes | Commit do autor para #268: mesmo conteúdo de `6466bcf9` nos fontes, com 263 linhas de teste (inclui a fixture `ONE_COUNT` de 70 palavras). Fica na história da branch antes do merge. |
| `e76ff6a2` | brcly | Merge pull request #268 from NickStallman/fix/xshader-mesh-launch-2 | Segundo merge de #268: apenas `tests/hlsl/mesh.rs` (+63/−14), levando a fixture `ONE_COUNT` que a resolução de conflito do primeiro merge deixou de fora. É o novo `main`. |

## Pull requests abertos (2)

Ambos em rascunho e ambos só de verificação.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 273 | verify: isolate the deliberate compute stand-in policy | `lanes-compute-root` exige exatamente um pipeline nulo intencional, mas herda `TUXBOX_NULL_PIPELINES` do chamador; com 0 o shader inválido é recusado e a fixture falha porque espera stand-in. Liga null pipelines só nesse gate e restaura o valor do chamador; testes unset/0/1. Sem mudança de runtime. +15/−3. | Não. Falha de verificação exposta pelo lote #252/#254/#260/#269/#266/#268 em `e76ff6a`. |
| 275 | verify: repeat descriptor reclamation lifecycles on one device | Gate opt-in que repete o ciclo completo de descritor oito vezes num único dispositivo D3D12, recriando heaps, filas, listas e fences entre ciclos, com 2.097.152 descritores de giro lógico sob limite residente de 1.000.000, Reset em voo, ClearState, rebind, OOM deliberado e recuperação. +53/−7. | Não; segue #172 (recuperação de descritor), que não está no catálogo como D-xx. |

## Pull requests mesclados (13)

Todos com `mergedAt = 2026-11-26T21:27:17Z` — o lote inteiro foi mesclado no mesmo instante.

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| 270 | xshader: lower f64 conversions, extrema, frexp, fract and ldexp | `210b28b4` | `float64.rs` +262/−33 com as novas formas e a correção de NaN sinalizante em trunc/floor/ceil/rndne; 14 arquivos e +4501 no total, incluindo `tools/native_rdna2/` (host.c, kernels.hip, check_model.py) e três JSON de evidência. |
| 265 | xshader: translate 32-bit image_atomic_cmpswap on typed UAVs | `0c51a552` | `emitter/image.rs` reconhece a forma DMASK 0x3/GLC e emite `InterlockedCompareExchange` com DATA[0]/DATA[1] invertidos; sonda de GPU com caso concorrente e caso acerto/erro. +528. |
| 260 | d3d12: translate root-less compute programs against a root reflection cannot describe | `21dbd0f6` | `compute.rs`/`shader_cache.rs`: programa sem root embutido só traduz contra o root do pipeline, com stand-in na recusa; cenário `lanes-compute-root` e teste de 143 linhas. |
| 253 | d3d12: import non-contiguous XMem backings through a reserved buffer | `83abffda` | `extra_heaps` no import, `wait_for_queue` com evento, ordem de drop documentada; teste de heap backing com mapas de runs e erros `E_NOTIMPL`/`ERROR_MORE_DATA`. +631/−72. |
| 254 | dxbc: recover corroborated descriptors in later compute tables | `1cab1a14` | `compute_table.rs` recupera descritores corroborados em tabelas posteriores; sonda `check_third_table_atomic.py` e teste em `raw_width.rs`. |
| 259 | d3d12: check device removal during prefix fence waits | `04bba7cd` | Sondagem de remoção a cada 100 ms com `fence_poll::poll`; teste garante que wakeup obsoleto não libera prefixo inacabado. |
| 255 | launcher: identify staged provider before comparing revisions | `c1df9c1a` | `launch_provenance.py` + `--require-current-provider`; doc diz que compara arquivos, não a escolha do loader. |
| 262 | launcher: discover Proton in Flatpak and secondary Steam libraries | `eed7c65c` | `host.py` varre `libraryfolders.vdf` de cada instalação Steam, incluindo Flatpak; teste com `-x86_64`. |
| 268 | xshader: a guarded mesh half result does not read the launch bits no reader takes | `6466bcf9`, `e76ff6a2` | `kept_half_unread` em `mesh.rs`, `vertex_start` em `mesh_exports.rs` e a fixture `ONE_COUNT`. **Dois merge commits** para o mesmo PR: o primeiro (`6466bcf9`) trouxe os fontes, o segundo (`e76ff6a2`) só o teste que a resolução de conflito perdeu. |
| 266 | d3d12: give a mesh pipeline's pixel stage the literal samplers its root leaves out | `2c4a3b93` | `mesh_table.rs`/`stream.rs` remontam a tabela com os estados literais faltantes; cenários `mesh-samplers` e `mesh-sampler-single`; conflitos resolvidos em `examples/lanes/*` e `verify.py`. |
| 252 | xshader: resolve inline compute root SRVs by the actual root parameter | `5979f8d8` | `compute_root_raw_srv` com corroboração FSSM/PSV; `--root-srv-remap` na sonda. |
| 269 | d3d12: retain original table extents when appending corroborated ranges | `fcfcabdc` | `compute_table.rs` mantém as extensões originais; cenário de fixture correspondente. |
| 256 | xshader: permit proven bounded pixel code-table offsets | `60497284` | `buffer.rs` aceita `Stage::Pixel` com prova de intervalo completo; `check_pixel_code_offset.py` e teste. |

## Cruzamento com o catálogo

- **D-22** (`Failed to remap` (-3) em roots raster com várias tabelas de descritor, corrigido em `95b242b`):
  a faixa ataca a **mesma mensagem** em outro caminho — `f806dd6c`/`#260` (compute sem root embutido) e
  `ddb9e5dd`/`#266` ("Failed to remap sampler 32765:0", 204 pipelines mesh/pixel no RADV). A entrada
  deve registrar que a correção de `95b242b` cobria roots raster e que compute e mesh só foram cobertos
  aqui. Não é reabertura; é escopo maior.
- **D-21** (assinatura mesh/pixel incompatível derruba o draw, corrigido em `315b287`): `c6a4887d` mostra
  que a correção de samplers de #241 não cobria o caso de **um** sampler declarado no pixel do mesh —
  a tradução tinha sucesso e amostrava com o estado do root. `e3262e02` cobre os 99 programas recusados
  por `v7`/`v6`. A entrada fica incompleta: três formas novas em `mesh_table.rs`/`stream.rs`/`mesh.rs`.
- **D-29** (filas de GPU ainda param em esperas de fence no story intro, corrigido em `2d887b3`):
  `6623e3e2`/`#259` só acrescenta detecção de remoção de dispositivo durante a espera de prefixo; não
  mexe no travamento do story intro. Não corrige.
- **D-17** (liberação imediata do backing de descritor, falha de MMU): `f9b630db` muda a importação de
  backings não contíguos e a ordem de drop (`native`/`extra_heaps` antes de `_shadow`/`_backing`), o que
  é a mesma classe de risco de vida do backing, mas não o caminho do descritor; não corrige.
- **D-04** (cache NVIDIA no lugar errado/teto): `8f10bd13` mexe em descoberta de Proton, não em cache.
- **Abre defeito novo?** Dois achados de processo, com hash:
  - `#268` tem **dois** merge commits (`6466bcf9` e `e76ff6a2`), e `e76ff6a2` leva só o teste que o
    primeiro merge perdeu na resolução de conflito — a árvore ficou, por alguns minutos, com fonte novo
    e fixture antiga.
  - `e56f9ee4` e `6466bcf9` carregam o mesmo conteúdo nos fontes; a diferença está só no tamanho do
    teste (+263/−14 contra +200). Duplicata de conteúdo na história, como o **D-15** do `AvatarManz`.

## Confiança

Verificado: 31 hashes e metadados em `gh api .../compare/68e79b7e...e76ff6a2`; cada commit lido em
`.../commits/<sha>` com lista de arquivos e contagens; os 13 merges listados em `--state merged --limit 50`
com `mergedAt` na janela e `mergeCommit.oid` conferido; os 2 PRs abertos lidos em `.../pulls/<n>`
(`draft=true` em ambos).

Divergência registrada: 14 merge commits para 13 PRs (#268 duas vezes) e todos os 13 `mergedAt`
idênticos a 21:27:17Z, o que indica merge em lote; a soma dos commits da janela (31) não é comparável
ao número de PRs por causa disso. Não verificado: se a árvore de `e76ff6a2` compila e passa as suítes
citadas; as medições e contagens de pipelines recusados vêm dos corpos dos commits; o conteúdo dos
PRs #273/#275 além do corpo.
