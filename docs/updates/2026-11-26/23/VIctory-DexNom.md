# DexNom/VIctory — atualização de 2026-11-26 23:00

## Resumo

Um único commit novo em `main`, e ele é de documentação: o título do README ganha o convite ao servidor
de comunidade. O resto do delta são PRs abertos — cinquenta acima do cursor 68, dos quais vinte estão
documentados aqui e trinta ficam no `backlog`. Nenhum PR mesclado na janela. Vários dos PRs abertos são
da mesma família de correções que o fork `victory-fork` já mesclou nesta hora (readback de textura,
identidade de cache, limites de cópia), o que dá o que cruzar com o catálogo.

## Commits novos (1)

Faixa `64bd58f5..8114ff77`, 1 commit.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `8114ff77` | DexNom | Update README title with community link | Uma linha do `README.md` (+1/−1): o `#` de título passa de `DexNom RayTime (AMD)` para `DexNom RayTime (AMD) Join the efforts: fluxer.gg/p9Y0JR0q`. Nada de código, nada de configuração; o parágrafo seguinte ("community fork of Marvisak's VIctory…") fica igual. |

## Pull requests abertos (20 de 50)

Todos com número acima de 68. Os trinta restantes (107–128) ficam no `backlog` com `last_pr_number` em
106. Propostas lidas do corpo de cada PR pela API, não de diff.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 87 | Fix the card_limits doc comment's defaults | Corrige o comentário de `card_limits()` em `crates/victory-gpu/src/shadow.rs`: ainda diz 16 MiB e 512 MB, mas o código usa 1 MiB e 1.024 MB desde a mudança que o PR cita. | Não. Dívida de documentação, sem id no catálogo. |
| 88 | Add TODO.md as a shared contributor task list | Cria `TODO.md`, porque `crates/victory-hle/src/modules/display.rs` cita "the presentation items in `TODO.md`" e o arquivo não existia. | Não. |
| 89 | Docs: BCPack and XMA2 are implemented in victory-formats | `bcpack.rs` e `xma.rs` existem e são exportados, mas o doc do crate e `docs/architecture.md` ainda os listam como planejados. | Não. |
| 90 | Docs: document fidelityfx_dir and data_dir | `victory.example.toml` passa a listar `[paths].fidelityfx_dir` (env `VICTORY_FIDELITYFX_DIR`) e `data_dir`; `Config` já os aceita. | Não. |
| 91 | Docs: add CONTRIBUTING.md | Reúne numa página o que um PR precisa passar (política de assets, `cargo xtask install-hooks`, build). | Não. |
| 92 | GPU: convert copies between 16-bit textures and their RGBA8 stand-ins | Em hosts que recusam render target B5G5R5A1 (86) e B4G4R4A4 (115) — NVIDIA — `target_format()` dá a esses alvos uma cópia RGBA8; cópias entre o original e o stand-in eram recusadas. | Não há entrada para o stand-in de 16 bits no catálogo do DexNom. Lacuna de cobertura, não correção. |
| 93 | Gate Windows-only self-exe tests and the crash module; Linux test lane goes 227/4 to 230/0 | Porta para Linux o que é assumidamente Windows: testes de self-exe e o módulo de crash; mede 182/182 do tradutor em duas máquinas Linux. | Não. |
| 94 | Add opt-in runtime telemetry and frame-time HUD | `--telemetry` para CPU/RAM/IO/VRAM por sessão, `--telemetry-full-log`, HUD com tempos de frame e taxas de RAM/VRAM, arquivos em `logs/`. | Não. |
| 95 | GPU: add AVX2 changed_runs, persistent wide_each pool, and async heap_buffers offload | Otimizações de CPU/memória portadas de `VIctory-lnx` sobre `64bd58f5`, preservando `BufferPass` incremental, `device_removed`, ray tracing e SparseTexture. | Não. Perf, não correção. |
| 96 | Audio: retain XMA frames after continuation-only packets | Porta o PR #96 do `Marvisak/VIctory` (commit `ba42db64`): quando um frame cruza pacotes e termina num pacote sem início de frame, o leitor atual devolve uma posição no padding. `draft=true`. | Não no catálogo do DexNom. Mesma família do D-05 do `victory-fork` só em "leitura de áudio". |
| 97 | Findings: NVIDIA RTX 4090 run of the Experimental NVIDIA mode | Documento em `docs/research/`; 26.148 de 26.148 programas traduzem e estruturam, 25.632 de 25.748 emitem, em RTX 4090 com driver 621.24. | Não. Relato de execução. |
| 98 | Make compute pipelines from programs the title compiled itself (device method 0x218) | O método 0x218 do `ID3D12Device` devolvia `E_NOTIMPL`; o PR implementa o decodificador em `native_build.rs` e monta pipelines de computação a partir dos programas do próprio título. Empilhado em #64/#61. | Não catalogado. É capacidade nova (o título deixa de ficar sem pipelines que ele mesmo compila). |
| 99 | Lane masks and whole-wave sections of wave64 programs on 32-lane hosts | Complemento de #98: `s_bfm_b64 exec, 32, 0`, `s_bitreplicate_b64_b32`, `s_pack_*`, `s_movk_i32` e `s_brev` em programas 0x218 wave64 num host de 32 lanes. | Não. |
| 100 | Addresses and descriptors of 0x218 programs | Complemento de #98: recusa por dominância em vez de panic quando o lowering lê valor não dominado (visto em `m083`), dobra `s_bfm_b64` de constantes. | Não. |
| 101 | Lower the operations 0x218 programs use that were refused | Complemento de #98: `s_sext_i32_i8/i16`, `v_mad_u32_u16`, `buffer_atomic_sub`, `s_absdiff_i32` e `v_pk_fmac_f16` com arredondamento único (`0x3e02`). | Não. Fecha lacunas de tradução de uma família de programas. |
| 102 | Views guarded while they are mapped stay writable | Corrige corrida: um refresh em outra thread podia abrir os guards dos blocos novos de uma view entre o mapeamento e a notificação, e a notificação os esquecia read-only. Portado de `avatarmanz/VIctory-7900XTX@160d222`. | Sim, em outro acervo: é a mesma corrida do **AvatarManz D-06** ("corrida entre mapear uma view e abrir os guards das outras views da mesma página"), aqui no DexNom. Não há id DexNom. |
| 103 | Setup: warn early when no GPU supports mesh shaders and SM 6.6 | `Setup-DexNomRuntime.ps1` consulta o D3D12 antes do build e avisa quando a placa não tem mesh shaders/SM 6.6, em vez de deixar o usuário descobrir depois do build Rust inteiro. | Não. |
| 104 | Findings: RTX 5070 Laptop 8 GB — current main crashes in --no-rt with the +0x5149aec pool-exhaustion breakpoint | Relato reproduzível: `main` (`64bd58f`) quebra em `--no-rt` com `0x80000003`, `rax=0x1fffe` (lista de instâncias vazia), 7 threads, sem exceção tratada; o build de 25/11 chegava ao mundo no mesmo modo. | **Abre candidato**: regressão de exaustão de pool (`+0x5149aec`) entre 25/11 e `64bd58f`. Não há id no catálogo. |
| 105 | docs: reorganize RayTime documentation | Encurta o README e move instalação, modos gráficos, diagnóstico e contribuição para um índice em `docs/`. Mantém a baseline Radeon e os problemas conhecidos visíveis. | Não. |
| 106 | GPU: discard unsubmitted waits when command lists are released | `Drop for List` remove só os waits daquela lista de `VALUE_WAITS`; listas criadas/gravadas/liberadas em ciclo deixavam registros para endereços mortos, e um endereço reutilizado herdava waits. | **Abre candidato**: vazamento de waits por lista liberada, com reuso de endereço herdando espera alheia. Sem id. |

## Pull requests mesclados (0)

Nenhum `merged_at` posterior a `2026-11-26T04:15:00Z` nos 100 PRs fechados mais recentes por `updated`.

## Cruzamento com o catálogo

- **D-15** (detecção de "hardware" nos testes de shader: WARP passava por GPU): o único commit da faixa é
  README, `8114ff77`. **Não corrige, não reabre.**
- **D-10, D-11, D-12** (abertos por reversão: `subresource_len`, escrita atômica da pipeline library,
  overflow de `clwb_loop`): nenhum commit novo toca `shadow.rs`, `pipeline/library.rs` ou
  `console_insn.rs`. Continuam abertos. **Nada na faixa os reaplica ou corrige.**
- **AvatarManz D-06** (corrida entre mapear view e abrir guards): o PR aberto #102 é exatamente essa
  correção, portada de `160d222`, mas o catálogo não tem id DexNom para ela e o PR ainda não foi mesclado.
- **victory-fork D-02** (chave de cache de shader sem os switches de wave): o PR aberto #112 ("Include
  shader-lowering switches in cache identity") é a mesma classe de defeito — `VICTORY_FIRST_HALF` e
  `VICTORY_FIRSTLANE_OWN` afetam a tradução e não entravam na identidade do cache. Aparece no `backlog`
  deste documento (PR 112 entre 107–128), não foi lido o diff.
- **Abre defeito novo?** Dois candidatos, ambos de PR aberto e não de commit mesclado: o crash
  `+0x5149aec` em `--no-rt` com placa de 8 GB (#104, regressão entre o build de 25/11 e `64bd58f`) e o
  vazamento de waits em listas liberadas (#106). Nenhum hash de commit nosso: são relatos contra
  `64bd58f5`/`64bd58f`, não diffs mesclados.

## Confiança

Verificado: `8114ff77` lido em `repos/DexNom/VIctory/commits/8114ff77` (1 arquivo, `README.md`, patch de
uma linha); head de `main` igual a `8114ff77` em `commits?sha=main&per_page=1`; os 50 PRs abertos vieram
de `gh pr list --state open --limit 50` e os números acima do cursor 68 são 20 de 50; a lista de fechados
(`pulls?state=closed&sort=updated&per_page=100`) não trouxe nenhum `merged_at` depois de
`2026-11-26T04:15:00Z`.

Não verificado: o conteúdo dos diffs dos 20 PRs abertos — as colunas "Proposta" saem do corpo de cada PR,
não de leitura de patch, e por isso os defeitos que um PR diz cobrir não foram conferidos no código. Não
verificado se #104 e #106 têm reprodução independente; são relatos de terceiros. Não verificado se
`fluxer.gg/p9Y0JR0q` é o convite certo — só que o README aponta para ele.
