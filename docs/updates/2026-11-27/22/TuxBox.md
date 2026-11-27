# brcly/TuxBox — atualização de 2026-11-27 22:00

## Resumo

A hora mais movimentada do `TuxBox` desde o início da rotina: 25 commits na janela, dos quais 17 são
commits de ramo novos, 7 são merges de PR e 1 é o merge da linha de `main` no ramo do #319. O trabalho é
quase todo no tradutor `xshader`: os campos de descritor que um programa lê (tamanho de textura, contagem
de registros de buffer) passam a ser preenchidos a partir do descritor em vez de adivinhados, a eleição de
lane passa a ser verificada por registradores exatos, um grupo de 32 threads passa a rodar como metade de
uma wave de console, e a contagem de chaves por lane troca a forma de wave inteira por um add por lane. O
cache de shader de runtime ganha retenção (três builds) com trava `flock`.

## Commits novos (25)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `80c57b38` | Nick Stallman | xshader: share which source fields an instruction reads | Cria `operand_fields.rs` (108 linhas): `memory()` (Smem/Flat/Mubuf/Mtbuf/Mimg/Ds/Exp), `run_start()` (início da corrida reta antes de uma instrução, pelos alvos de branch), `unused_field()` (no VOP3 o terceiro campo não lido decodifica como `s0`; tabela de quantos campos cada instrução lê, inclusive `v_cndmask_b32`, `*_co_ci_` e `v_pk_*`) e `constant_before()` (literal que um SGPR carrega, só quando todo caminho até o uso passou pelo `s_mov`). Testes decodificam palavras reais de `v_sqrt_f32`, `v_pack_b32_f16`, `v_lshlrev_b64`, `v_add_co_u32` e um `s_cbranch_scc0` que pula o move. |
| `e8fb84e6` | Nick Stallman | xshader: fill a texture's size fields where a program reads them from its descriptor | Em `descriptor_size.rs` (+198) segue os campos de tamanho do descritor de textura até o uso e emite `GetDimensions` + escrita dos SGPRs de largura/altura nos bits que o console usa; anota os pontos de emissão em `resources.rs`, `scalar_memory.rs`, `setup.rs` e `resource_block.rs`. Cria `tools/check_descriptor_size.py` (127 linhas) e o registra em `check_translator_gpu.py`; 103 linhas de teste no tradutor. |
| `ce07c554` | Nick Stallman | tools: initialize the descriptor-size check's heap properties field by field | Duas linhas no verificador de GPU: `D3D12_HEAP_PROPERTIES` deixa de ser inicializado por agregado e passa a ser preenchido campo a campo (o agregado deixava campos indefinidos no teste, não no tradutor). |
| `1ad86d0b` | Nick Stallman | xshader: fill a buffer's record count where a program reads it from its descriptor | `descriptor_size.rs` (+79): a palavra 2 do descritor de buffer (`SQ_BUF_RSRC_WORD2`) é a contagem de registros que `GetDimensions` devolvia ao fonte; `note_buffer_count()` registra a leitura dessa palavra e `emit_buffer_count()` a preenche, exigindo que o load esteja na mesma corrida reta, que nada tenha escrito o registrador desde então (`clobbers_scalar`) e que o campo lido seja realmente lido (`unused_field`). Testes: o load preenche a contagem, um buffer usado só como buffer não recebe `GetDimensions`, e uma contagem sem buffer no host é recusada. |
| `63ad67ee` | Nick Stallman | tools: check translated buffer record counts on the GPU | `tools/check_buffer_count.py` (149 linhas) compara, na GPU, a contagem de registros que o programa traduzido lê com a do descritor de staging; registrado em `check_translator_gpu.py`; 42 linhas de teste no tradutor. |
| `f7e404cd` | Nick Stallman | xshader: count a structured buffer only by its recorded stride, and declare a buffer asked only for its count | Estruturado só é contado quando a stride vem de um registro (`table_structured_stride` ou `fssm_bindings::stride_words`), porque o host pode dividir o tamanho da view pelo elemento declarado; declara `tbuf`/`raw`/`sbuf` para um buffer que o programa só consulta pela contagem. Testes: stride adivinhada é recusada; stride de FSSM produz `sbuf1.GetDimensions(n0, st0)`. |
| `aebffea0` | Nick Stallman | xshader: refuse a scalar load through a root descriptor's address | `abi.rs` ganha `root_address_sgprs()`: linhas de user-data FSSM de classe 1 com largura 2 são endereços de 2 SGPR. `resource_block.rs` recusa um `s_load` cuja base seja um desses endereços, com o offset na mensagem, em vez de tratar o endereço como ponteiro de tabela e deixar o destino sem escrita. Teste em `sampler_heap.rs`. |
| `5f3bb04e` | Nick Stallman | xshader: read scalar loads by their exact registers when verifying an election | `election.rs` ganha `smem_writes_sgpr()`: um load escalar escreve uma faixa de registradores (uma palavra por `dwordxN`). Na verificação da eleição, instruções `Smem` passam a ser julgadas pelos registradores exatos (`reads_sgpr`/`smem_writes_sgpr`) em vez de `may_touch_sgpr`. |
| `6da26fe5` | Nick Stallman | xshader: ignore the unused third source of v_writelane when verifying an election | Uma linha: `v_writelane_b32` entra na lista de instruções de dois operandos, cujo terceiro campo decodifica como `s0` e não deve contar como leitura. |
| `5fea7f7e` | Nick Stallman | xshader: run a 32-thread group as the lower half of a 64-lane console wave | `HALF_WAVE = 32` passa a ser aceito em `paired_wave64`; as lanes 32..64 nunca começam e só participam onde o programa alarga EXEC (reduções sobre a wave inteira), com o thread ID continuando a numeração da metade de baixo. `pipeline.rs` repete a tradução na forma pareada quando a forma de uma lane por thread recusa com "wave operation requires complete 64-lane compute waves". `check_paired_wave64.py` e os perfis de host acompanham. |
| `ea19a35c` | Nick Stallman | xshader: count lanes by key with one add per lane | Cria `rewrite/key_counts.rs` (595 linhas): substitui a contagem por chave na forma de wave inteira (que recusa) por um add por lane, sem máscara nem troca entre metades; exemplo `examples/key_counts.rs` e fixture de 161 linhas, verificador de GPU `check_key_counts_gpu.py` (142), 187 linhas de teste. |
| `990b1bfb` | csj0oe | shader-cache: keep the caches of the last three compiler builds | `runtime_shaders.py` (+117): retenção de caches de runtime — cada build tem cache próprio, nada os removia (dois builds de ramo somavam 2,1 GB); passam a ficar o build em execução e os dois outros usados mais recentemente, com um arquivo `last-used` (o mtime do diretório muda quando um arquivo interno é reescrito) e um `in-use` mantido com `flock`. `rank_pipeline_failures.py` passa a consultar a retenção; 141 linhas de teste. |
| `8ad601d5` | csj0oe | xshader: lower a first-lane read once its result is consumed | `lanes.rs`: a busca da eleição deixa de ser limitada às 64 instruções anteriores e passa a exigir que o `s_ff1_i32_b` escreva o registrador da lane a partir de máscara escalar ou EXEC, que nenhum alvo de branch entre no intervalo e que EXEC não seja escrito entre a eleição e o `v_readlane`. Testes em `hlsl.rs` e `rewrite_lanes.rs`. |
| `ffffb751` | brcly | shader-cache: let launchers of one build share its in-use claim | A trava do `in-use` vira `LOCK_SH`: launchers do mesmo build compartilham o cache; só a retenção pede a trava exclusiva. Teste de retenção cobre o caso. |
| `0a5ebfa7` | brcly | xshader: read an empty texture's size fields as one texel | Uma view nula não reporta texels; o descritor vazio do console tem zero nesses campos, que lido direto daria largura -1. Passa a emitir `(max(w,1) - 1)` e `(max(h,1) - 1)`. Verificador de GPU ajustado. |
| `db6d4e88` | brcly | docs: record the half-wave GPU results | Uma linha em `docs/SHADERS.md` com o resultado do half-wave na GPU. |
| `2253a87b` | brcly | Merge pull request #316 (xshader/refuse-load-through-root-descriptor-address) | Merge de 2 pais; entra `aebffea0` (recusa de `s_load` por endereço de root descriptor). |
| `04667020` | brcly | Merge pull request #317 (xshader/election-exact-registers) | Merge de 2 pais; entram `5f3bb04e` e `6da26fe5` (eleição por registradores exatos). |
| `fec1627c` | brcly | Merge pull request #318 (xshader/half-wave-group) | Merge de 2 pais; entram `5fea7f7e` e `db6d4e88` (half-wave de 32 threads). |
| `47daef42` | brcly | Merge pull request #315 (xshader/descriptor-words-standalone) | Merge de 2 pais; entram 7 commits — `80c57b38`, `e8fb84e6`, `ce07c554`, `1ad86d0b`, `63ad67ee`, `f7e404cd`, `0a5ebfa7` (campos de descritor). |
| `8ef909bb` | brcly | Merge pull request #314 (shader-cache/branch-runtime-caches) | Merge de 2 pais; entram `990b1bfb` e `ffffb751` (retenção de cache de runtime). |
| `dc92a00e` | brcly | Merge main: run the key-count retry inside the half-wave fallback | Merge de 2 pais (`ea19a35c` ← `8ef909bb`) que traz a linha de `main` para dentro do ramo da contagem de chaves: 25 commits, entre eles os seis já documentados às 19:00–21:00 (`f5233880`, `49fad5f7`, `8c082e65` do #312, `7953f0a5` do #313, `e3cf2096` e `89fb2517`, o helper DXC). O diff deste merge é conteúdo já documentado, não trabalho novo; a resolução junta o retry da contagem ao fallback half-wave. |
| `5d5a4d08` | brcly | Merge pull request #319 (xshader/dynamic-lane-operands) | Merge de 2 pais; entram `ea19a35c` (contagem de chaves por lane) e `dc92a00e`. |
| `db31dd09` | brcly | xshader: keep a first-lane read whose index a branch or saveexec still needs | `lanes.rs` (+29/-14): novo `index_dead_after()`, um dataflow que só deixa baixar a leitura de primeira lane quando todo caminho a partir dela escreve o registrador ou termina o programa antes de outro uso; `saveexec` passa a contar como escrita de EXEC. Testes cobrem branch e `saveexec`. |
| `d37f092c` | brcly | Merge pull request #321 (xshader/first-lane-read-after-branch) | Merge de 2 pais; entram `8ad601d5` e `db31dd09` (leitura de primeira lane depois de branch). |

Os 25 commits são 17 de ramo + 7 merges de PR + 1 merge de `main`; nenhum passa do teto de 40.

## Pull requests abertos (2)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 324 | d3d12: release the scratch pages past a stored texture snapshot | Liberar as páginas de scratch depois de um snapshot de textura armazenado — o snapshot guarda a textura e as páginas de staging ficam presas. Área de D-17/D-18 (vida do backing de descritor e alias de textura posicionada). Lido por título e base; patch não inspecionado. | NAO VERIFICADO |
| 322 | xshader: branch with the whole wave where a guard's vote or lane reads would otherwise refuse | Ramificar com a wave inteira quando o voto de um guard ou leituras de lane fariam a tradução recusar — extensão da família de `5fea7f7e`/`db31dd09`. Lido por título; patch não inspecionado. | NAO VERIFICADO |

## Pull requests mesclados (7)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| 314 | shader-cache: keep the caches of the last three compiler builds | `8ef909bb` | Retenção de caches de runtime + trava compartilhada (`990b1bfb`, `ffffb751`). |
| 315 | xshader: descriptor words standalone | `47daef42` | Os 7 commits de campos de descritor (tamanho de textura, contagem de registros, stride, textura vazia). |
| 316 | xshader: refuse load through root descriptor address | `2253a87b` | `aebffea0`: recusa fail-closed de `s_load` cuja base é endereço de root descriptor. |
| 317 | xshader: election exact registers | `04667020` | `5f3bb04e`, `6da26fe5`: verificação da eleição por registradores exatos e terceiro operando de `v_writelane` ignorado. |
| 318 | xshader: half-wave group | `fec1627c` | `5fea7f7e`, `db6d4e88`: grupo de 32 threads como metade de wave de console, com fallback no tradutor. |
| 319 | xshader: dynamic lane operands | `5d5a4d08` | `ea19a35c` + merge de `main`: contagem de chaves por lane e o retry dentro do fallback half-wave. |
| 321 | xshader: first-lane read after branch | `d37f092c` | `8ad601d5`, `db31dd09`: leitura de primeira lane depois de branch, com dataflow de uso e `saveexec` contando como escrita de EXEC. |

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum. Nenhum dos 17 commits de ramo toca um defeito catalogado deste repositório.
- **D-02/D-03 (tempo do otimizador DXC / `-O0`):** não há commit novo nesta janela; o helper DXC
  (`89fb2517`, `dxc_worker.cpp`) reaparece neste diff apenas porque `dc92a00e` mergeou `main` no ramo.
  O helper continua sendo usado com as opções que a CLI passa (inclusive `-O0`), então D-02/D-03 seguem
  abertos, como registrado às 21:00. O que é verificável no arquivo agora: a lista de passes vem de
  `-Odump` com `-correlated-propagation` removido, e o fallback para a CLI existe e é testado — o worker
  responde `no` e `PersistentDxc.compile` devolve `False`, apagando a saída parcial
  (`test_refused_program_goes_to_the_cli_and_the_helper_serves_on`).
- **D-28 (`WaveSize(N)` recusado só no driver):** `dxc_worker.cpp` reescreve à mão a propriedade 11 do
  entry point para repor `[WaveSize(N)]` depois do front end (`with_wave_size`, `i32 11, !999999999`).
  Se o formato da metadata não for o esperado, ele lança `unexpected metadata` e o programa vai para a
  CLI — falha segura, mas o caminho rápido deixa de valer para aquele shader. É a área de D-28, não uma
  correção dele.
- **Abre defeito novo?** Dois pontos de atenção, ambos de área e nenhum verificado em execução:
  1. `1ad86d0b`/`f7e404cd` passam a preencher a contagem de registros do descritor de buffer a partir de
     `GetDimensions` e **declaram** um buffer que o programa só consulta pela contagem. O teste cobre o
     caso de stride desconhecida (recusa) e o de PSV sem buffer (recusa), mas a decisão de declarar
     `tbuf`/`raw`/`sbuf` para uma consulta de contagem muda o conjunto de recursos refletidos — se um
     caminho de emissão não passar por `unused_field`, a contagem pode ser preenchida onde o programa
     não a lê. Commit para revisão: `f7e404cd`.
  2. `0a5ebfa7` troca `(w - 1)` por `(max(w,1) - 1)`; para uma textura de largura 0 o console grava 0 no
     campo, que agora vira 0 em vez de estourar para `0x3FFFFFFF` no shift. É correção, mas nenhum teste
     de GPU foi rodado aqui.
- **Observação de contagem:** o `compare` do `TuxBox` deu `ahead_by = 25` (17 commits de ramo + 7 merges
  de PR + 1 merge de `main`) e os PRs mesclados na janela são 7; cada `merge_commit_sha` da listagem de
  PRs aparece na lista de commits do `compare` (#314 `8ef909bb`, #315 `47daef42`, #316 `2253a87b`, #317
  `04667020`, #318 `fec1627c`, #319 `5d5a4d08`, #321 `d37f092c`), então as duas contagens batem.

## Confiança

**Verificado por diff:** todos os 25 commits foram lidos por arquivo e patch (39 objetos de commit
baixados na coleta). Os 7 merges foram confirmados por `compare <primeiro pai>...<segundo pai>`, com a
lista de commits que cada um traz (`ahead_by` 1, 2, 2, 7, 2, 2, 2). Os patches citados foram lidos na
íntegra ou até o trecho que decide o comportamento: `operand_fields.rs`, `descriptor_size.rs`,
`election.rs`, `lanes.rs`, `paired_wave64.rs`, `pipeline.rs`, `runtime_shaders.py`, `dxc_worker.cpp`,
`persistent_dxc.py`.

**Não verificado:** nenhuma verificação de GPU foi executada; os números de tempo e de ganho que
aparecem em mensagens de commit (51 s contra 60 s, 2,1 GB de cache) não foram medidos aqui. Os PRs #322 e
#324 foram lidos por título e base, não por patch. Os testes do repositório não foram rodados. A
`default_branch` foi reconferida por API (`main`); o head documentado é
`d37f092ce5ff49871f5196f00e3bb32e28459848`.
