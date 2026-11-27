# brcly/TuxBox — atualização de 2026-11-27 20:00

## Resumo

Seis commits entre o cursor `c2a59af5` e o head `7953f0a5`: quatro de conteúdo, um de merge e um de
evidência. Dois são reescritas do tradutor de shader para eliminar branches (`tb_float_relation` e
`tb_f16_cvt`/`tb_cvt_u32_f32`), um conserta o script do devcontainer para não matar Xwayland do host e
um faz o `verify` reportar como **skip** a checagem de splash onde a GPU apresenta por software. Um PR
mesclado (#313, só evidência de Windows físico), um aberto (#312, root SRV de compute depois das
tabelas).

## Commits novos (6)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `049e5a43` | brcly | xshader: compare floats without branches | `tools/shader-compiler/src/hlsl/compare.rs` +16/-12: `tb_float_relation` deixa de ter quatro `return` antecipados e passa a calcular as mesmas quatro relações por seleção aritmética — `unordered*8`, `equal*2` e `(4 - 3*less)` para 1/4 — com `flush` derivado de `!keepDenormals`. O motivo é de compilação: um programa da ProjectScarlett inlineia o helper centenas de vezes e cada `return` dava blocos extras ao DXC |
| `65e3e48c` | brcly | xshader: convert to f16 and unsigned integers without branches | `float_to_integer.hlsl` +4/-3 e `helper_code.rs` +24/-1: `tb_cvt_u32_f32` usa máscara `ok`/`big` em vez de retornos, e `tb_f16_cvt` passa a calcular todos os intervalos de magnitude e escolher por `value ^= (value ^ alvo) & (0 - condição)`, na ordem denormal → normal → overflow → infinito → NaN (o último vence). Os limiares são os mesmos de antes (`0x477fe000` com `rtz`, `0x477ff000` com arredondamento par). `tests/hlsl/host_profiles.txt` tem as 48 linhas reescritas: **todos os hashes de tradução dos perfis de host mudam**, porque o texto do helper é a chave |
| `55ec224a` | brcly | verify: skip the splash check where the GPU presents through software | `project_tuxbox/verify.py` +22, `docs/ENVIRONMENT.md` +1/-1, `tests/test_verify.py` +7/-1: nova `splash_skip_reason()` devolve motivo quando o ambiente do display privado tem `MESA_VK_WSI_DEBUG` (GPU que apresenta ao servidor privado por software). Nesse caso `guest-present-4k-splash` não roda, entra em `report["skipped"]` e sai como `SKIP` na saída. O commit declara que o primeiro frame depois do splash nunca completa nesse arranjo e que **a causa não é conhecida**; os outros checks de janela passam |
| `7e2c2408` | brcly | devcontainer: leave the host's X servers alone | `.devcontainer/start-display.sh` +24/-2: o script recusa rodar fora de contêiner (`/.dockerenv`, `/run/.containerenv`), grava pidfile e, na retentativa, para só o Xwayland que ele mesmo subiu (confere `/proc/<pid>/cmdline`), em vez de `pkill -x Xwayland`. Novo `tests/test_devcontainer_display.py` verifica que não há `pkill`/`killall` no código e que a recusa acontece antes de qualquer programa ser chamado |
| `e3cf2096` | KainanP | evidence: first physical Windows 11 runs on the native D3D12 backend | `evidence/windows11-rtx4060laptop-native.json` e `evidence/windows11-uhd-a78b-native.json` (+1054 cada), `docs/RESULTS.md` +81/-73, `docs/progress.svg` +1/-1: primeiros recibos de Windows físico, 54/69 gates no RTX 4060 Laptop e 53/69 no Intel UHD. As falhas são registradas como falhas, sem correção |
| `7953f0a5` | brcly | Merge pull request #313 from KainanP/evidence/windows11-native-rtx4060-uhd | merge do #313. O diff do merge é exatamente o de `e3cf2096` (4 arquivos, +2190/-74): o PR carregava um commit só |

## Pull requests abertos (1)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #312 | dxbc: confirm a compute root SRV depois das tabelas | `runtime/dxbc/src/compute_table.rs` +6/-4: o leitor de binding passa a ler o registro root SRV (classe `0x100`) também para programas de compute, com as mesmas três checagens dos estágios raster (linha PSV bruta para o registrador, registro de user-data para os SGPRs e parâmetro root SRV no índice). Se qualquer uma faltar, continua falhando fechado e mantendo o root do título. O autor mede, sobre 52 pedidos da ProjectScarlett com esse registro, 19 que só traduzem contra o root refletido e 33 que traduzem diferente (+212/-13, 7 arquivos, inclui `examples/lanes` e teste) | Área de **D-22** (`Failed to remap (-3)` em roots raster com várias tabelas de descritor), mas não é o mesmo defeito: D-22 é o root **raster** com várias tabelas; #312 é o root SRV de **compute** depois das tabelas. Não fecha D-22 |

## Pull requests mesclados (1)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #313 | evidence: first physical Windows 11 runs (RTX 4060 Laptop and Intel UHD, native D3D12) | `7953f0a5`, 2026-11-27T16:55:39Z, base `main` | Só evidência: dois JSON de recibo, `RESULTS.md` e o SVG de progresso. Nenhum arquivo de código. O merge não altera comportamento; dá pela primeira vez conteúdo à coluna "Physical Windows x64" de todas as gates |

## Cruzamento com o catálogo

- **D-24 (resolução de merge apagou lowering e quebrou o verificador):** a reescrita de `049e5a43` e
  `65e3e48c` não é merge, mas tem o mesmo risco de classe — mudar o texto do helper muda a tradução de
  todo programa. Aqui o repositório refixa as 48 linhas de `host_profiles.txt`, então o verificador
  continua pinando o que é emitido; o que ele **não** pina é a semântica nova do helper: a equivalência
  alegada (11.520 comparações em duas GPUs, 6.336 entradas em duas GPUs) é medição do autor, feita fora
  do repositório, e nenhum teste do commit compara o helper novo com um modelo de referência.
- **D-22:** ver #312 acima — área vizinha, mecanismo diferente, não corrigido.
- **D-29 (filas de GPU ainda param em esperas de fence no story intro):** não tocado.
- **Candidata a linha nova de catálogo (cobertura, não produto):** `55ec224a` transforma uma falha real e
  não explicada (primeiro frame depois do splash não completa quando a GPU apresenta por software) em
  `SKIP`. O sintoma deixa de aparecer como falha em qualquer recibo de `verify` nesse arranjo. É correto
  do ponto de vista de ruído de CI e é uma perda de sinal do ponto de vista de regressão.
- **#313:** o `RESULTS.md` passa a mostrar gates falhando em Windows físico (16 no RTX 4060, 16 no UHD).
  O PR não mapeia essas falhas para D-xx do catálogo; não inventei correspondência.

## Confiança

Verificado por mim, pelo diff:

- os seis hashes, autores, arquivos e contagens de linha vieram da API de commits; li o patch inteiro de
  `049e5a43`, `65e3e48c`, `55ec224a` e `7e2c2408`;
- o conteúdo de `7953f0a5` é idêntico ao de `e3cf2096` (mesmos 4 arquivos, mesmas contagens), o que
  confirma que o merge não trouxe trabalho extra;
- `host_profiles.txt`: 48 linhas trocadas, todas de hash, sem linha nova ou removida — a leitura é que o
  texto do helper entra na chave de tradução. Deduzido do diff, não de leitura do código que monta a
  chave;
- o único PR aberto acima do cursor é #312; base `main` conferida no campo `base.ref`.

NÃO VERIFICADO:

- os números de desempenho e de equivalência dos dois commits de shader (8,6%/11,7%/3,0% de tempo de DXC,
  11.520 e 6.336 comparações em RTX 4070 Ti SUPER e iGPU Raphael). Não rodei DXC nem GPU;
- o conteúdo dos dois recibos de evidência: são JSON de 1054 linhas cada, li apenas nome, tamanho e a
  descrição do PR (54/69 e 53/69 gates);
- a alegação de que os outros checks de janela passam no arranjo de apresentação por software — vem do
  commit, não de execução;
- não li `project_tuxbox/verify.py` fora dos hunks do commit, então não sei se algum outro gate usa o
  mesmo `private[2]` e deveria ter o mesmo tratamento.
