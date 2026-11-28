# fxceless/victory-fork — atualização de 2026-11-28 13:00

## Resumo

Oito commits e dois merges desde o cursor `2eade68c`, e o cursor de commit chega a `6b3ce6ce`. Um
nono commit (`368967f8`, o build "run-403" com a água corrigida) chegou às 13:25Z, **depois** da
janela da coleta, e não está documentado. A hora
tem um conserto medido — a ordem dos samplers estáticos, que era a causa de o driver recompilar
centenas de pipelines a cada arranque —, o primeiro uso do guia de ray tracing por hardware numa
placa, e a correção imediata do próprio autor de que os fps daquela sessão **não** são ganho, porque
~590 de ~2.000 instâncias ficaram fora da cena. O guia também expôs um `device hung` por uma palavra
de memória compartilhada nunca escrita antes da primeira leitura.

## Commits novos (8)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `38f404ac` | CwWwWwW | Formats: parenthesize the shift in a texture test | Uma linha em `crates/victory-formats/src/texture.rs:1346`: `(i * 7 >> 3)` vira `((i * 7) >> 3)`. O passo de CI `cargo clippy --all-targets -- -D warnings` parava em `clippy::precedence` e não chegava ao build. Nada de comportamento muda (a precedência de `*` sobre `>>` já era a pretendida). |
| `8ddcf358` | t90 | shader: handle scalar codes beyond physical registers in unspill | `crates/victory-shader/src/ir.rs`: `unspill` reserva `used: [false; 256]` em vez de 128; 18 linhas, mais um teste de regressão `scalar_codes_past_the_registers_do_not_panic` que monta três programas (`v_writelane_b32`, wave64 e wave32) e chama `build`. Backport declarado de Vice-Tay `00eaf47c` e de `avatarmanz/VIctory-7900XTX#325`, com a descrição do panic no índice 215 com a tabela de 128. |
| `7353d144` | fxceless | README: status of 27 November evening | Trinta e três linhas no README. Registra o diagnóstico dos samplers estáticos (0 de 1.303 pipelines acima de 1 s numa sessão, 406 em outra, quase todos com seis ou mais samplers estáticos), o guia de ray tracing como "exato num rig, não rodou em placa", o verde-limão como valor de dado (camada de decalque escolhida por máscara) e a regra de medir cada mudança sozinha. |
| `12550f58` | fxceless | Merge pull request #1485 | Merge de dois pais do `38f404ac`; o único arquivo tocado é o mesmo `texture.rs`, +1/−1. Não traz nada além do commit já listado. |
| `2f3f3993` | fxceless | Merge pull request #1495 | Merge de dois pais do `8ddcf358`; `ir.rs` +18/−1. Idem: é o mesmo diff, já descrito. |
| `9e907466` | fxceless | Pipelines: static samplers take their places in the driver's table in one kept order | O commit grande da hora: 394 linhas, arquivo novo `crates/victory-gpu/src/pipeline/sampler_order.rs` (347 linhas). O driver NVIDIA dá a cada sampler estático um lugar na tabela quando a primeira root signature que o contém é criada, e o pipeline compilado carrega esses lugares; como as threads de pipeline chegavam em outra ordem a cada arranque, o cache em disco não respondia. Agora todo sampler estático que um pipeline já teve é listado num arquivo ao lado da pipeline library e criado nessa ordem antes da primeira root signature que o contenha. `VICTORY_SAMPLER_ORDER=0` volta ao comportamento antigo. Medição declarada no RTX 4070: 406 pipelines acima de 1 s num arranque antes, 211 no arranque que compilou na ordem guardada, 0 de 1.314 no seguinte. Contadores novos (`SLOW_NOW`, `NEW`) entram na linha de log de `library.rs`. |
| `de57b010` | fxceless | Ray guide: the card's ray-tracing hardware finds candidates for the title's walkers | 10.546 linhas em 25 arquivos, desligado por padrão (`VICTORY_RAY_GUIDE`). `guide.rs` (566) e `guide/core.rs` (2.194) constroem estruturas de ray tracing do host a partir das do título (bottom levels decodificados da memória do título para um pool, um top level antes de cada caminhada), o hardware acha candidatos e o teste de triângulo do título decide. `descriptor/host.rs` ganha `structure_at()` (view SRV sem recurso) e `slot_probe()` para a sonda de livery. `livery.rs` novo (787 linhas). O corpo registra o defeito encontrado no caminho: o teste de nó guiado guardava uma palavra por thread na memória compartilhada do grupo e nada a escrevia antes da primeira leitura — o adaptador de software zera essa memória e todo teste passava, a placa travou o primeiro dispatch (device hung); o programa agora escreve onde começa. |
| `6b3ce6ce` | fxceless | README: the ray guide's first session left objects out of the rays' scene; its frame rate is not a gain | Corrige o próprio commit anterior no README: ~590 de ~2.000 instâncias por frame ficaram fora do top level (a checagem de sanidade recusou 3.598 de 4.007 bottom levels, e a instância de um deles é omitida), então a luz e a sombra traçadas desses objetos desapareceram e os raios que acham menos custam menos. "O número não diz nada até a imagem estar inteira"; deixar o switch desligado para jogar. |

## Pull requests abertos (8)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #1500 | Add memory-pressure safeguards, timing safety and frame-time testing | Limita uso especulativo de memória para hosts de 16 GiB de RAM / 8 GiB de VRAM, mantendo o retry de textura e as salvaguardas de descritor; tetos explícitos de textura se interceptam. | Não diretamente; é a classe de D-15 (page mapping que quebrou o streaming), sem hash no diff. |
| #1503 | GPU: defer host replacements for private translator lists | Adia a aquisição de allocator/lista nativa vazia para listas privadas do tradutor, que hoje são adquiridas logo após o desanexo da parte do host. | Não. |
| #1505 | Add exhaustive development plan + agent rules | Documento: plano de desenvolvimento e regras de agente para o fork. | Não (só documentação). |
| #1506 | Shaders: 64-bit integer compares on their dwords | Backport de `Marvisak/VIctory#1599`: `v_cmp_{lt,le,gt,ge,eq,ne}_{i64,u64}` e `v_cmpx_` comparam por dword em vez de um `icmp` em i64 — o compilador AMD erra a ordem quando o par de VGPRs não é o esperado. Reproduzido num RX 7900 XT. | Classe de `XCL D-20`/`D-24` (ordem de comparação/leitura em AMD), sem linha própria em `victory-fork`. |
| #1507 | Shaders: the carry of v_mad_u64_u32 on its dwords | O carry de `v_mad_u64_u32` é calculado nas dwords; o lowering atual faz `icmp ult` em i64 e erra quando o addendo compartilha registrador com um fator (RX 7900 XT). | Área de `XCL D-04` (carry-ins lidos da máscara errada), hipótese. |
| #1508 | Tests: the indirect budget tests assert both REST modes | Os dois testes de orçamento indireto falham na `main` `6b3ce6c` e só passam com `VICTORY_INDIRECT_REST=1`; o PR faz os testes afirmarem os dois modos. | Não; é dívida de teste (classe de D-17). |
| #1509 | Audio: the XMA decoder runs at time-critical priority | O estalo de áudio cresce ao longo da run porque cada voz XMA escreve o anel de saída na memória do título; o PR sobe a prioridade do decodificador. | Não. |
| #1513 | Fork improvements: shader cache key, 64-bit compares/carry, reading callbacks, password hiding, clippy cleanup, docs | Traz para o fork upstream as mudanças de `ors33/victory-fork`; o primeiro item é **#1504: a chave do cache de shader inclui todos os switches do tradutor** (`WATER_TILE_SLOTS`, `WAVE_LOOP_SKIPS`, `MEMORY_NEAR`, `LDS_S…`). | **Sim, por área: é a classe de D-02** — a chave de cache que não incluía os switches de wave, reaberta em 2026-11-27 22:00 por três switches novos. O corpo do PR é a única evidência lida; o diff de #1504 não foi aberto nesta execução. |

## Pull requests mesclados (2)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #1485 | Formats: parenthesize the shift in a texture test | `12550f58` | Exatamente o que o título diz: um parêntese em `texture.rs:1346`. O merge não trouxe mais nada. |
| #1495 | Shaders: the scalar-code table of the parking search covers all 256 codes | `2f3f3993` | `ir.rs` +18/−1: tabela de 128 → 256 e o teste de regressão. Confere com o prometido no título ("all 256 codes"). |

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum hash de `catalogo/defeitos.csv` aparece na janela (8 commits + 2 merges,
  conferidos por prefixo contra os 262 hashes das 305 linhas). `38f404ac` e `8ddcf358` são consertos,
  mas de defeitos que não estão no catálogo.
- **Abre defeito novo?** Não comprovado, mas dois candidatos com evidência no próprio diff:
  - `de57b010` (guia de ray tracing): a palavra de memória compartilhada não escrita antes da
    primeira leitura **derrubou o device** na placa. É a mesma classe de **D-16** (perdas de device
    sem causa identificada). Aqui a causa está identificada e corrigida no mesmo commit; o que fica
    sem resposta é se o caminho guiado ainda perde o device por outros motivos.
  - `6b3ce6ce` admite que a checagem de sanidade recusa 3.598 de 4.007 bottom levels e que instâncias
    de estruturas recusadas são omitidas da cena — ou seja, o guia desenha menos e mede mais rápido.
    É a classe de **D-14** (alegação de ganho comparando cenas diferentes), e o autor a evita
    explicitamente em vez de repeti-la.
- **Área de D-02 (chave de cache sem os switches):** o aberto #1513/#1504 propõe exatamente a
  correção; não há commit na janela que a aplique. O switch novo `VICTORY_RAY_GUIDE` (e
  `VICTORY_SAMPLER_ORDER`) **não** foi conferido contra a chave do cache neste diff — não afirmo que
  entra ou deixa de entrar.
- **Área de D-17 (dívida de verificação):** `9e907466` traz medições de uma placa e um número
  (406 → 211 → 0), mas é o autor medindo; `de57b010` não tem medição de fps válida por admissão
  própria.

## Confiança

**Verificado:** os 8 commits e os 2 merges foram lidos pelo diff (`repos/fxceless/victory-fork/commits/<sha>`,
13:30–13:40Z), incluindo nomes de arquivo, contagens e trechos de patch; a numeração dos PRs veio do
assunto do merge e da lista `pulls?state=merged`; os 8 PRs abertos acima de #1496 foram lidos por
título e corpo (primeiros ~200 caracteres) da API.

**Não verificado:** o diff de #1504 (só o corpo de #1513), o conteúdo de `sampler_order.rs` além dos
primeiros 80 trechos de patch, e qualquer execução — nenhuma build, nenhum teste e nenhuma medição
foram feitos nesta rotina; os números de fps e de pipelines acima de 1 s são os que o autor relata no
commit e no README, não medidos aqui. O `device hung` do guia é relato do corpo do commit.

**Cobertura:** `last_commit` = `6b3ce6ce54288df514585caa6880f91468c8bc54` (último documentado); sem
`commit_cursor` antes desta hora, agora `368967f8fea2a982cea134c5a051b2afe7d9fc8f` porque um nono
commit (`368967f8`, 2026-11-28T13:25:44Z, "The run-403 build with the water fixed: VIctory-NVIDIA
959dcaf, ray tracing on by default, the water's own light on") chegou depois da coleta e **não** foi
documentado. `head_at_run` = `368967f8…`. `last_run` = `2026-11-28T01:48:22Z` (maior `merged_at`
documentado, #1485/#1495).
