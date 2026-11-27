# R0Y00/Xbox2Windows — atualização de 2026-11-27 21:00

## Resumo

Cinco commits entre o cursor `63a30dc1` e o head `429f11d5`, todos mesclados como squash, todos de
PR menos o de documentação. Quatro deles implementam, no caminho direto de DXIL para compute
(`src/gfx1013_direct_compute.inl`), instruções VOP3P/VOP1/VOP3A da gfx1013 que até agora não tinham
lowering: dois dots de nibble empacotado, o `v_sat_pk_u8_i16` e os quatro `v_frexp_*` (f32 e f16).
São adições puras — nenhuma linha removida dos 4 arquivos de código —, cada uma com controle de
compilação (recusa de modificadores não canônicos) e um teste que monta o programa e compara 2.048
palavras mais palavras de guarda. Nenhum defeito do catálogo é corrigido. Três PRs abertos acima de
#3139, nenhum na área dos lowerings.

## Commits novos (5)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `7769a33d` | nanyoudegege | docs: record the Intel Arc A380 hardware report (#3128) | `README.md` +1: acrescenta uma linha à tabela de adaptadores — Intel Arc A380, "16 (16 on direct run)" colunas, `Skipped, lane range 8–32`, 408 queries passadas, driver 32.0.101.9150, debug layer indisponível, link para um comentário da issue #1. Só um registro de medição de terceiro; não altera código |
| `8cf9be9b` | rdx | shader: lower v_dot8_i32_i4 and v_dot8_u32_u4 in the direct compute path (#3079) | `src/gfx1013_direct_compute.inl` +23: no despacho por nome de instrução, com `compute`, aceita os opcodes VOP3P `0x18`/`0x19` e **recusa** (`refuse`) se a codificação não for `Encoding::Mix` com `(i.a & 0x7f00) == 0x4000` e `(i.b >> 27) == 3` — isto é, exige `OP_SEL = 0`, `OP_SEL_HI = [1,1,1]` e `NEG_LO/NEG_HI = 0`. Extrai os oito nibbles: no `i32`, desloca à esquerda até o topo (`28 - shift`) e faz `ashr 28` (extensão de sinal); no `u32`, `lshr`/`and 15`. Soma os oito produtos; com `clamp`, o resultado usa `saturated_add(dot, acc, sign)`, sem `clamp` soma módulo 2³². O doc novo justifica a saturação só na soma final: o dot de oito nibbles cabe em `[-448, 512]` ou `[0, 1.800]`. Testes: `tests/cmake/native_packed_dot8.cmake` registra `native_packed_dot8_controls` (60 s) e `native_packed_dot8_hardware` (`SKIP_RETURN_CODE 77`, 60 s); `tests/native_packed_dot8_check.py` +130 monta as palavras VOP3P à mão (opcode `0x18`/`0x19`, bit 15 = clamp, `operands` com `OP_SEL`s), com 8 casos — entre eles overflow assinado `+512` e `-448`, carry não assinado `+1.800`, as duas fronteiras exatas, máscara `EXEC` esparsa e um terceiro dispatch que zera os bits de `clamp` para provar que o resultado muda |
| `5ebdc581` | rdx | shader: lower v_sat_pk_u8_i16 in the direct compute path (#3095) | `gfx1013_direct_compute.inl` +15: aceita VOP1 `0x62` e VOP3A `0x1e2`; recusa `clamp`, `omod`, `abs`, `neg`, SDWA, DPP, `op_sel` e `src1`/`src2` não-zero na forma VOP3A (`if (i.wide && (i.src[1] != 0 \|\| i.src[2] != 0))` e `(i.a & 0x7800)`). Estende em sinal as duas metades de 16 bits, prende cada uma em `[0, 255]` com dois `select` (`slt 0` → 0, `sgt 255` → 255) e empacota `lo \| hi << 8` — o `D[31:16]` sai zero. `tests/native_sat_pk_u8_i16_check.py` +151 cobre as duas codificações, 16 padrões de entrada, a metade alta zerada e `EXEC` esparso |
| `80668b97` | rdx | shader: lower v_frexp_exp_i32_f32 and v_frexp_mant_f32 in the direct compute path (#3117) | `gfx1013_direct_compute.inl` +36: VOP1 `0x3f`/`0x40` e VOP3A `0x1bf`/`0x1c0`. Recusa `subword`, `dpp_control`, operandos permutados, `scale`, `clamp` no exp e, na forma larga, `op_sel`, `abs`/`neg` fora do primeiro operando e `src1`/`src2`. Separa sinal/expoente/mantissa por máscara; `keep` = expoente 255 (Inf/NaN) ou magnitude zero — nesses casos o exp devolve 0 e o mant devolve os bits originais. Para subnormal, normaliza a mantissa com uma busca binária de deslocamentos (16, 8, 4, 2, 1) contando `steps`; exp = `e - 126`, mant = `sinal \| 0x3F000000 \| frac`. O `clamp` do mant (só existe nele) zera NaN e negativo e prende em `1.0` (`> 1065353215` → `1065353216`) |
| `429f11d5` | rdx | shader: lower v_frexp_mant_f16 and v_frexp_exp_i16_f16 in the direct compute path (#3120) | `gfx1013_direct_compute.inl` +37: VOP1 `0x59`/`0x5a`, VOP3A `0x1d9`/`0x1da`. Mesma estrutura em binário16: expoente de 5 bits, mantissa de 10, normalização com passos 8/4/2/1, exp = `e - 14`, mant = `sinal \| 0x3800 \| frac`; `clamp` trata NaN (`> 31744`) e negativo como 0 e prende em `15360`. O ponto que o commit destaca: **preserva `D[31:16]`** — lê o destino, mascara com `0xFFFF0000` e faz `or` com o resultado de 16 bits. Teste com controle negativo que zera a metade alta e falha |

## Pull requests abertos (3)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #3142 | startup: identify a rotated rectangle corner by its diagonal | base `main`, head `startup/ui-gradient-gap`: escolhe o canto de um retângulo girado pela diagonal na inicialização. Só título e base lidos | Não identificado no catálogo |
| #3141 | runtime: trace UI texture row-pitch bands and fix XG mip-0 and CopyTextureRegion tracking | base `main`, head `runtime/ui-texture-bands`: rastreio de bandas de row-pitch de textura de UI, mais correção de mip 0 de `XG` e do rastreamento de `CopyTextureRegion` | Área de D-08 do catálogo Xbox2Windows (cópia de textura escrita pela CPU antes de `CopyResource`/`CopyTextureRegion`) — **mesma área, defeito diferente**; o patch não foi lido |
| #3140 | native: read fixed-index samplers from the directly indexed sampler heap | base `main`, head `native/sampler-heap-fixed-index`: ler samplers de índice fixo no heap indexado diretamente | Não identificado no catálogo |

## Pull requests mesclados (5)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #3128 | docs: record the Intel Arc A380 hardware report | `7769a33d`, 2026-11-27T17:40:34Z, base `main` | A linha da A380 no README, exatamente como no commit |
| #3079 | shader: lower v_dot8_i32_i4 and v_dot8_u32_u4 in the direct compute path | `8cf9be9b`, 17:43:45Z, base `main` | O commit é squash de 1 pai (`committer` = GitHub): 4 arquivos, +198/-0 |
| #3095 | shader: lower v_sat_pk_u8_i16 in the direct compute path | `5ebdc581`, 17:45:22Z, base `main` | 4 arquivos, +204/-0 |
| #3117 | shader: lower v_frexp_exp_i32_f32 and v_frexp_mant_f32 in the direct compute path | `80668b97`, 17:48:05Z, base `main` | 4 arquivos, +220/-0 |
| #3120 | shader: lower v_frexp_mant_f16 and v_frexp_exp_i16_f16 in the direct compute path | `429f11d5`, 17:48:43Z, base `main` | 4 arquivos, +226/-0; é o head atual |

Os cinco `merge_commit_sha` são iguais aos commits da faixa e todos têm um só pai — merges por squash,
sem commit de merge. Por isso a contagem de merges (5) bate com a de commits (5) nesta hora, ao
contrário da anterior, em que 30 commits vinham de um único PR.

## Cruzamento com o catálogo

- **Nenhum `D-xx` do Xbox2Windows é corrigido.** Os 30 defeitos da seção do repositório em
  `catalogo/defeitos.md` tratam de placeholder/heap (D-02, D-03, D-17), HLE e filas (D-12), debugger
  (D-13, D-14), assinatura de comando (D-15), inventário de PE (D-22), viewport (D-26), texto 2D
  `a16` (D-27), conversão UNORM/SNORM (D-28) e reversão (D-29) — nenhum é "lowering ausente no caminho
  direto de compute".
- **Achado novo, por paralelo de catálogo:** `8cf9be9b` leva ao caminho direto o `v_dot8_i32_i4`, e o
  catálogo do **VItoriaBr** tem exatamente esse buraco registrado — D-15, "`v_dot8_i32_i4` ausente da
  tabela de opcodes", corrigido em `75aa531`. Ou seja: a mesma lacuna existia em dois forks, e o
  Xbox2Windows só a fecha agora. Não é defeito do catálogo deste repositório; é evidência de que a
  classe "instrução sem lowering no caminho direto" não está catalogada aqui.
- **Candidato a entrada nova (não gravei no CSV):** antes de `8cf9be9b`/`5ebdc581`/`80668b97`/
  `429f11d5`, um shader de compute que usasse `v_dot8_*`, `v_sat_pk_u8_i16` ou `v_frexp_*` era
  recusado pelo caminho direto. Não há, no repositório, medição de que um título retail exercite essas
  oito instruções — só o teste sintético. Registro o fato (hash, arquivo, linha do `if` novo), sem
  afirmar impacto em jogo.
- **Área de D-08 em PR aberto:** #3141 mexe em `CopyTextureRegion`, que é o mecanismo de D-08
  (cópia de textura escrita pela CPU não enviada ao host antes da cópia). O patch não foi lido; a
  afirmação é de título, e fica como área, não como correção.

## Confiança

Verificado por mim, pelo diff:

- os cinco hashes, autores, arquivos e contagens de linha vieram da API; li o hunk inteiro de
  `gfx1013_direct_compute.inl` nos quatro commits de lowering e o único hunk de `README.md`;
- conferi no diff que os quatro commits de código são **só adição** (`+N/-0` nos `.inl`, `+/-0` nos
  demais arquivos) e que cada um registra teste no CMake com `SKIP_RETURN_CODE 77` para a parte de
  hardware — o teste de hardware se auto-ignora sem GPU/wave32 e com DXC ausente;
- li o começo de `native_packed_dot8_check.py`: os casos são montados como palavras de instrução
  reais (VOP3P `0x18`/`0x19`, `operands` com `OP_SEL`/`NEG`), com `SENTINEL`/`INACTIVE` e palavras de
  guarda, então o teste exercita o lowering e não uma reimplementação em Python;
- `parents` = 1 e `committer` = GitHub nos cinco, o que sustenta "squash"; `merge_commit_sha` igual ao
  SHA do commit em todos;
- a base dos três PRs abertos foi lida em `base.ref` (`main` nos três).

NÃO VERIFICADO:

- **nada foi executado.** Não rodei `ctest`, `compile_dxil` nem `native_store_replay`; não há
  Windows/DXC/GPU neste ambiente. As afirmações de que os testes passam são do corpo do PR e do
  arquivo de teste, não de execução;
- se a semântica de saturação está certa para todos os pares de nibble: li o código e o argumento de
  faixa do doc (`[-448, 512]`/`[0, 1.800]`), não recalculei caso a caso nem comparei com o ISA além do
  que o próprio diff cita (seção 12.10/12.6 do RDNA 2 ISA);
- o comportamento do `refuse` em produção: quantos shaders reais caem em `unsupported ... operand
  modifiers` não é medido em lugar nenhum do repositório;
- os patches de #3140, #3141 e #3142: li título, base, head; não abri diff, contagem de arquivos nem
  corpo completo (a API não trouxe `changed_files`/`additions` para eles nesta coleta);
- a atribuição do #3128 a "Intel Arc A380 ... 16 lanes": é uma linha de README com link para um
  comentário de issue; não abri o comentário.
