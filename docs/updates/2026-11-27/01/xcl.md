# ivall/xcl — atualização de 2026-11-27 01:00

## Resumo

**Nenhum commit novo na `main`**: o cursor `8b957bec` ainda é o topo. Os seis PRs mesclados na janela
foram todos para **`nvidia-dev`**, não para a default — cinco fusões de verdade (dois pais) e um squash.
São a rodada de memória de 8 GB do Prime1997: `CLZERO` em laço do título vira código nativo (#2892),
tile de janela esparsa com memória preguiçosa (#2897), build tardio esperando metade do headroom (#2894),
dispatch indireto de programa dividido construído fora da chamada (#2893), uma linha de log dona das
páginas (#2896) e a contagem de tiles nunca escritos pela CPU (#2895). Na fila de abertos, 201 PRs acima
do cursor: documentados os 20 de menor número (2662..2685), quase todos de ferramentas Python.

## Commits novos (0)

`gh api 'repos/ivall/xcl/commits?sha=main&per_page=100'` devolveu `8b957bec` na posição 0. Nada a
documentar por diff de commit: o delta desta hora está inteiro nos PRs mesclados em `nvidia-dev`.

## Pull requests abertos (20; 181 acima deles ficam para a próxima hora)

`gh pr list --state open --limit 500` devolveu **500** PRs abertos (mínimo #2186, máximo #2951); **201**
têm número acima do cursor 2661. Foram documentados os 20 de menor número, como nas horas anteriores, para
que o cursor avance de forma monotônica; os 181 restantes (2686..2951) ficam para as próximas execuções.
Propostas vêm do corpo de cada PR.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 2662 | Scripts: map PE export RVAs only into file-backed bytes | O inspetor de export mapeia RVA em bytes de disco usando o maior entre `VirtualSize` e `SizeOfRawData`; passa a limitar a `SizeOfRawData`, porque a cauda virtual é zerada pelo carregador e lê bytes da seção seguinte. Fixture PE32+ com a RVA de nome exatamente no fim do raw da primeira seção. | Não. |
| 2664 | Recompiler: eliminate DXIL bitcode double-copy and intermediate allocations in container assembly | `container.rs` (+219/-108) troca `program_part` por `program_header -> [u8; 24]` e um `assemble_container` com capacidade exata (uma cópia do bitcode, zero `Vec` intermediário), e dimensiona os acessores `psv`/`signature`. Os bytes do contêiner continuam idênticos; o corpo mede 533 → 216 µs para 1 MiB. | Não. |
| 2665 | Validate repeated BC7 dictionary widths | O gerador descartava a repetição das larguras de índice dos verbetes 1–254 de BC7, então uma entrada inconsistente podia sobrescrever a tabela gerada e sair com sucesso. Passa a checar antes de escrever e mantém a saída anterior intacta. | Não. |
| 2667 | Report empty XCL smoke logs clearly | O `smoke.ps1` para antes de interpretar log vazio ou só de espaços e informa o caminho, em vez de estourar numa regex com entrada nula. | Não. |
| 2668 | Find complete shader layout pointer pair | O verificador de layout de BVH parava na primeira `s_and_b32` que mascarava `s1`; passa a escolher a primeira máscara de registrador alto com um `move` correspondente de registrador baixo. Fixture com máscara irrelevante antes do par válido. | Não. |
| 2669 | Read saved coverage reports with JSON encoding detection | `coverage_gate.py --run-json` recusava relatório salvo com BOM UTF-8/UTF-16; passa o arquivo em bytes ao decodificador JSON. | Não. |
| 2670 | D3D12.X: the window title counts driver pipeline builds too | O título da janela contava só as compilações do DXC e mostrava "0 running" enquanto o driver montava pipelines frias (2–5 s). `driver_pso` passa a contar cada build na criação e no fim, registra as de ≥1 s sem `XCL_PSO_TRACE`, e o texto vira `DXC N running, M done \| driver N building (W waiting for a slot), D done[, Q queued][, F failed]`. | Não. |
| 2672 | Resolve relative smoke launcher path | Resolve `-Xcl` para caminho absoluto antes de entregar ao lançador, porque `Push-Location` muda de onde o processo resolve a string relativa. | Não. |
| 2673 | Loader: O(1) envelope fast-reject, packed RangeEntry lookup, synchronized registration | `modules.rs` troca as três estáticas separadas por uma `RangeTable` de `RangeEntry { base, size }` atômicos com envelope `[min, max]`, recusa O(1) fora do envelope, serializa registro/desregistro sob `MAPPED.write()`, avisa uma vez ao saturar 512 e troca as somas de contenção por subtração verificada (o teste `contains_handles_base_plus_size_overflow` estourava em `modules.rs:34`). | Não. |
| 2674 | Ignore whitespace in RDNA2 instruction flags | `gen_rdna2.py` descartava a flag de instrução quando o texto no XML tinha espaço em volta; passa a aparar antes de comparar. | Não. |
| 2675 | Clarify required GPU skip modes in README | Documenta quais motivos de pulo cada valor de `XCL_REQUIRE_GPU` (`wave64`, `mesh`, os dois) recusa em `runner::skip_test`. | Não. |
| 2676 | Allow test runner thread overrides | `scripts/test.ps1` sempre acrescentava `--test-threads=4`; passa a pôr o padrão em `RUST_TEST_THREADS` e repassar os argumentos como vieram, porque o libtest do Rust 1.99 recusa a opção duplicada. | Não. |
| 2677 | Clarify Bink skip environment values | Documenta que valor vazio deixa o pulo de filme desligado e que o parser apara espaços e ignora caixa. | Não. |
| 2678 | D3D12.X: copy a GPU-written title texture back into the title's memory | Parte 1 de #2663: `texture::download` copia todas as subresources para linhas lineares com barreiras de ida e volta, `title_textures::write_subresources` re-tileia só as páginas de 4 KiB que mudaram e só as comprometidas e graváveis, e `read_back` junta os dois. Fica `#[allow(dead_code)]` até a parte 2. Recusa formato com mais de um plano, MSAA e textura com página descomprometida no meio (testes CPU e GPU). | Não. |
| 2679 | Use an effective trace import exclusion example | Troca o exemplo de `XCL_TRACE_IMPORTS` de `GetTickCount` (fora da seleção `hle` de qualquer jeito) por uma exclusão `kernelx.dll` de `VirtualAlloc`, que muda de fato o filtro. | Não. |
| 2680 | Document optional UMA window layouts | Corrige a linha de memória do README: quatro janelas de 4 GiB por padrão, oito ou dezesseis com janelas de heap, sem implicar RAM/VRAM comprometida. | Não. |
| 2681 | Ignore commented GDK table rows | `gen_gdk_table.py` apanhava linhas de API dentro de comentários `//` e `/* */`; passa a remover comentários preservando campos entre aspas (um `//` citado engolia a linha ativa seguinte) e mantém o campo `Source`. | Não. |
| 2683 | XAudio2: lock-free hook trampoline lookup, atomic silent-voice fast path, zero-copy decoded PCM | `xaudio2.rs` (+225/-29): espelha as primeiras 32 entradas `(vtable, slot, orig)` numa tabela livre de lock, conta vozes silenciosas vivas num `AtomicUsize` para pular o `XMA.lock()` quando não há nenhuma, entrega o `Vec<i16>` decodificado direto ao XAudio2 (sem segunda alocação nem cópia de 2 bytes por amostra) e troca a alocação de até 32 MiB do silêncio pelo mapeamento compartilhado. | Não. |
| 2684 | Resolve relative game path in smoke runs | Resolve `-Game` contra a localização do chamador antes de calcular o hash do título e lançar, para o mesmo diretório não gerar hash diferente. | Não. |
| 2685 | Accept spaced tiling table terminators | `gen_tiling.py` exigia o `};` de fechamento na coluna 1 e sem espaço; passa a aceitar espaço antes da chave e antes do ponto e vírgula. Saída byte-idêntica conferida contra o `gfx10SwizzlePattern.h` do Mesa. | Não. |

## Pull requests mesclados (6)

Os seis têm base `nvidia-dev`. Cinco são fusões reais (dois pais, com o `+/-` igual ao do PR); o #2895 foi
squash (um pai). Nenhum aparece no histórico da `main`.

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #2892 | PE loader: a title's CLZERO loops run as native code on a CPU without CLZERO | `7e63fc6d` (2 pais, +149/-0, 2 arquivos) | O que promete: `cpu_has_clzero` (CPUID 8000_0008h EBX bit 0), `clzero_loop_thunk` que zera as linhas inteiras com `rep stosb` e deixa `rax`/flags como o laço, `patch_clzero_loops` que acha os laços de 12 bytes e troca por `call rel32` + `nop` de 7 bytes, chamado no passo 9 do mapeamento. `XCL_CLZERO_PATCH=0` mantém a emulação. O corpo mede 459 → 1393 frames num smoke de 60 s na RTX 3060 Ti. |
| #2897 | D3D12.X: XCL_SPARSE_LAZY=1 gives a committed sparse-window tile video memory at its first use | `fb35a759` (2 pais, +440/-12, 2 arquivos) | O que promete: sentinela `LAZY` no pool, `ensure_listed`/`materialize`/`give_memory`/`will_write`, lista `starved`, e `note_write`/`note_imm` em `list_x.rs` para as escritas registradas. Desligado por padrão e deixado de fora em dispositivo com tiled resources abaixo do nível 2. **Lacuna declarada no próprio PR**: um shader que escreve na janela pelo endereço, num tile que nenhum buffer/view/escrita registrada nomeou, não escreve nada enquanto o tile não tem memória — é a razão de vir desligado. |
| #2894 | D3D12.X: a build the title already draws without waits for half the commit headroom | `3eca2912` (2 pais, +30/-1, 2 arquivos) | O que promete: `headroom_for(bytes, late)` usado por `huge_compile_turn` e `build_is_late` em `pso_async.rs` — build de pipeline que o título já desenha sem esperar (tardio) espera 1,5 GB em vez de 3 GB. O corpo mede 44.596 → 35.618 draws pulados e 1.130 → 724 s no portão, com os frames do mundo iguais. |
| #2893 | D3D12.X: an indirect dispatch does not build a split program's whole form in the call | `eff3bcab` (2 pais, +69/-3, 3 arquivos) | O que promete: `Whole { Plain, Ready, Late }` e `whole_ready`, que cria a thread `xcl-pso-whole` uma vez por cadeia; `bind::execute_indirect` usa e passa a pular (contando) o dispatch enquanto a forma inteira não existe; `cs_goes_on` diz se `XCL_CS_PSO_WAIT_MS` está setado. Sem a variável, comportamento antigo. O corpo diz que não há teste unitário do spawn. |
| #2896 | D3D12.X: the activity log says who holds the target pages | `71b9f7b2` (2 pais, +27/-0, 2 arquivos) | O que promete: `vram::page_owners` e uma linha abaixo da linha de memória do `XCL_ACTIVITY`, com texturas e suas páginas, texturas amostradas com páginas por nível (`XCL_TEXTURE_LEVEL_PAGES`) e páginas ociosas. Sem mudança de comportamento; o PR avisa que a soma conta página compartilhada uma vez por textura e pode passar do total. |
| #2895 | D3D12.X: the sparse windows' line counts mapped tiles the CPU never wrote | `f3cd0a5b` (1 pai, +16/-0, 1 arquivo) | Squash do PR: `never_sent` e uma segunda linha junto de "sparse windows: N submits" com quantos tiles mapeados nunca receberam escrita da CPU e quantos MB. Sem mudança de comportamento; base do #2897. |

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum. Os seis merges são de memória de placa e de tradução de programa; nenhum dos
  24 defeitos do catálogo do `xcl` trata de janela esparsa, `CLZERO`, headroom de compilação ou thread de
  forma inteira. Os candidatos próximos não casam: **D-20** (`ExecuteIndirect` lia argumentos indiretos
  antes da barreira) e **D-21** (root signature errada em dispatch indireto de pipeline não traduzido) são
  de `ExecuteIndirect`; o #2893 trata da **construção da forma inteira** de um programa dividido, que
  acontece antes do dispatch existir. **D-06/D-07/D-08** (arredondamento f16, `-0.0`, `-Gis`) são do
  tradutor direto; nenhum arquivo desses caminhos aparece nos seis diffs.
- **Abre defeito novo?** Nenhum demonstrado por diff. Dois pontos ficam registrados:
  - `#2897` (`XCL_SPARSE_LAZY`) muda **onde** os bytes do título aparecem: um tile comprometido sem
    memória não recebe escrita de shader feita pelo endereço, e só o que passa por buffer/view/escrita
    registrada é materializado. O PR declara a lacuna e é por isso que o recurso vem desligado e fora de
    tiled resources nível 1. Vira defeito se o padrão mudar sem fechar isso.
  - `#2892` grava código nativo numa faixa livre a até 64 KiB (ou 1 GiB, conforme `code_near`) do laço
    `CLZERO` do título. Se não houver faixa livre, o laço fica emulado e uma linha é logada — o corpo diz
    isso e não testa esse caso.

## Confiança

Verificado: `commits?sha=main&per_page=100` com `8b957bec` na posição 0 (nenhum commit novo na default);
`commits?sha=nvidia-dev&per_page=100` contendo os seis `merge_commit_sha` (`7e63fc6d`, `fb35a759`,
`3eca2912`, `eff3bcab`, `71b9f7b2`, `f3cd0a5b`), com o topo de `nvidia-dev` em `ecc0bb57`; cada merge lido
em `commits/<sha>` — contagem de pais (2,2,2,2,2,1), `+/-` e número de arquivos iguais aos do PR
respectivo; os 6 PRs em `pulls?state=closed&sort=updated` filtrados por `merged_at >
2026-11-26T22:45:09Z`; os 20 PRs abertos com os corpos em `pulls/<n>` e a lista completa em `gh pr list
--state open --limit 500` (500 linhas, 201 acima do cursor).

Não verificado: a execução dos smokes citados nos corpos (RTX 3060 Ti, Série S, WARP; o `Build and test
(Windows, no GPU)` do GitHub falha em `sparse::tests::a_gpu_write_into_a_gpu_view_lands_in_its_cpu_view`
segundo todos os seis PRs, e não conferi o estado atual da CI); se os seis merges chegam à `main` por
alguma promoção de ramo (até esta hora não chegaram); e a unidade de `count` nos testes do tradutor
quando falam de "grupos de 32/64 lanes". Nenhum teste foi executado aqui; os 13 repositórios monitorados
são somente leitura.
