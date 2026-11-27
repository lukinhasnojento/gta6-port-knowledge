# R0Y00/Xbox2Windows — atualização de 2026-11-27 22:00

## Resumo

Cinco merges na janela, todos de infraestrutura de teste e diagnóstico, nenhum no tradutor de shader: um
servidor único de device para os casos de GPU, tolerância de peso de filtro linear justificada pela
especificação do D3D, contadores `raw=`/`completed=` nas linhas de status do DirectStorage, captura
periódica da janela do título e carga de provedores de `pe_run_package` sem o diálogo de erro crítico do
Windows.

## Commits novos (5)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `572ec6f8` | tedin7 | tests: serve GPU draw cases from one offscreen_check device (#2766) | `tests/gpu_case_runner.py` (227 linhas): `run(renderer, adapter, args)` fala com um processo longo `offscreen_check <adapter> --serve` em vez de um processo por caso — um device D3D12 para todos os casos do teste. Devolve o mesmo `CompletedProcess` do caminho de uma chamada (0 passa, 1 falha, 77 skip, 2 argumentos), com o banner do adaptador repetido em **todo** caso; só um servidor que não sobe cai para a chamada única. Depois do primeiro caso, morte, estouro de `CASE_TIMEOUT_S = 300` ou resposta com número de caso errado dão falha do caso (nunca reexecução que possa virar passe); o próximo caso começa um servidor novo. `offscreen_check.cpp` ganha o modo `--serve` (+57), `gpu_case_runner_check.py` (243) testa o runner, e cinco checagens existentes passam a usá-lo. |
| `0e3ebe36` | tedin7 | test: allow one 8-bit filter weight step in the linear sampler probes (#2794) | `tests/startup_sampler_x.cpp` (+28/-3): a tolerância dos probes lineares deixa de ser só arredondamento de float e passa a `weight_step * footprint_span`, com `weight_step = 1/256` — o D3D12 só exige 8 bits fracionários para os pesos bilinear e de mip, então um sampler pode mover cada peso um passo. `footprint_span()` mede a maior diferença entre os quatro texels do footprint do mip. Probes de point filtering continuam exatos. |
| `e92776bd` | tedin7 | startup: print raw= and completed= in the DirectStorage status lines (#2817) | `src/startup_stubs/xcrdapi.cpp` (+26/-8): novo `format_request_counts()` imprime `raw=R inflated=I failed=F bcpack-blank=B completed=C` nas linhas de status, com `completed = raw + inflated + bcpack-blank`; o trecho `inflated=%ld failed=%ld` fica byte a byte porque `tools/startup/live_view.py` o parseia, e o `live_view.py` ganha os campos novos. `requests_done` documentado como só as cópias não comprimidas. Testes em `startup_storage_failure_counts.cpp` (+70) e `startup_live_telemetry_check.py`. |
| `7832b98b` | tedin7 | startup-tools: save the title's window every N seconds (--capture-every) | `tools/startup/window_capture.py` (96 linhas): conversão BGRA→PNG sem API do Windows (fatias de buffer, filtro 0 por linha, `zlib`), nome de arquivo por frame, pulo de frame idêntico ao anterior, teto de `FILE_CAP = 120` arquivos e uma linha de resumo. `debug_launch.py` (+88/-56) ganha `--capture-every`, `run.ps1`/`run.sh` repassam a opção, `docs/startup-runtime.md` documenta, e `startup_window_capture_check.py` (145) + `startup_window_probe.cpp` (63) testam. |
| `88cddb09` | Álvaro | pe: load pe_run_package providers without a critical-error dialog (#2829) | `tools/pe_run_package.py` (+29/-3): `quiet_loader()` põe `SEM_FAILCRITICALERRORS` no modo de erro **da thread** em volta de cada `LoadLibraryExW` e restaura o modo anterior no `finally`. Um provedor que não é imagem carregável (vazio, truncado, não-PE) passa a falhar a carga e reportar símbolos ausentes em vez de abrir o diálogo modal "Bad Image" e bloquear a execução. `tests/pe_run_package_cli_check.py` (87 linhas) cobre a CLI e o caso. |

## Pull requests abertos (10)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 3159 | shader: lower v_cvt_f16_u16, v_cvt_f16_i16, v_cvt_u16_f16 and v_cvt_i16_f16 in the direct compute path | Quarto lote de lowerings de conversão f16/i16 no caminho direto de compute, com controle de compilação e teste de hardware. Continua `429f11d5`/`80668b97`. | NAO VERIFICADO |
| 3157 | compute: accept vcc_hi as the wave32 mask of SDWA compares | Aceitar `vcc_hi` como máscara wave32 nas comparações SDWA. Área de D-21 (seleções SDWA). | NAO VERIFICADO |
| 3155 | tests: honor hardware adapter selection in native Compute replay | Respeitar a seleção de adaptador nos replays nativos de compute. | NAO VERIFICADO |
| 3154 | runtime: bind constant static samplers on the in-process compute route | Ligar samplers estáticos constantes na rota de compute em processo. | NAO VERIFICADO |
| 3153 | runtime: restore menu sampler bindings and BC7 display uploads | Restaurar bindings de sampler do menu e uploads de display BC7. | NAO VERIFICADO |
| 3152 | 'Fix DPP row shift direction logic' | Corrigir a direção do shift de linha do DPP. Área de D-21. | NAO VERIFICADO |
| 3151 | shader: lower packed binary16 arithmetic in direct DXIL | Aritmética binary16 empacotada no DXIL direto. | NAO VERIFICADO |
| 3147 | shader: read inline float sources as binary16 in v_frexp_mant_f16 and v_frexp_exp_i16_f16 | Ler fontes float inline como binary16 nos dois `v_frexp_*` f16 — correção do próprio `429f11d5` documentado às 21:00. | NAO VERIFICADO |
| 3146 | compute: keep MIMG sampler index bit 4 out of the reserved-bit check | Manter o bit 4 do índice de sampler do MIMG fora da checagem de bits reservados. | NAO VERIFICADO |
| 3144 | shader: lower packed 16-bit minimum and maximum in direct DXIL | min/max de 16 bits empacotados no DXIL direto. | NAO VERIFICADO |

## Pull requests mesclados (5)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| 2766 | tests: serve GPU draw cases from one offscreen_check device | `572ec6f8` | Runner de device único, modo `--serve`, teste do runner. |
| 2794 | test: allow one 8-bit filter weight step in the linear sampler probes | `0e3ebe36` | Tolerância por passo de peso de 1/256 escalada pelo span do footprint. |
| 2817 | startup: print raw= and completed= in the DirectStorage status lines | `e92776bd` | Contadores novos nas linhas de status e no `live_view.py`. |
| 2822 | startup-tools: save the title's window every N seconds | `7832b98b` | `window_capture.py`, `--capture-every`, teto de 120 arquivos. |
| 2829 | pe: load pe_run_package providers without a critical-error dialog | `88cddb09` | `SEM_FAILCRITICALERRORS` por thread em volta do `LoadLibraryExW`. |

Todos são squash de um pai; o `merge_commit_sha` de cada PR é o próprio commit.

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum. Os cinco commits são de teste/diagnóstico e de carga de provedor.
- **D-22 (`pe_inventory` quebra com inventário malformado e não valida a CLI):** `88cddb09` mexe em
  `pe_run_package.py`, não em `pe_inventory`, e o que ele corrige é o diálogo modal do loader, não a
  validação do inventário; adiciona `pe_run_package_cli_check.py`, que é a mesma lacuna de teste de D-22,
  em outra ferramenta. Área, não correção.
- **D-11 (testes de hardware sem `SKIP_RETURN_CODE 77`):** `572ec6f8` preserva o contrato de 77 no runner
  novo (servidor que morre dá falha do caso, nunca 77), e `7832b98b` não toca o contrato. Área, não
  correção.
- **D-28 (conversão UNORM/SNORM usava `round()`):** `0e3ebe36` amplia a tolerância dos probes lineares de
  amostragem, não a conversão de store formatado. Área vizinha, sem relação com a correção de `f37fe7b8`.
- **Abre defeito novo?** Dois pontos de atenção, nenhum verificado em execução:
  1. `e92776bd`: `completed = raw + inflated + bcpack-blank`, mas o comentário do mesmo arquivo diz que
     requisições comprimidas "são registradas e deixadas sem fazer" e `requests_bcpack_blank` conta
     exatamente uma requisição comprimida cujo BCPack não foi decodificado. Se uma requisição
     `bcpack-blank` não escreveu o destino, contá-la como `completed` superestima o progresso de storage —
     exatamente o número que a linha passou a mostrar. O commit justifica o contrário ("a run whose
     requests are all compressed still shows storage progressing"). Divergência de semântica a conferir
     em `xcrdapi.cpp`; não reproduzida.
  2. `0e3ebe36`: a tolerância de um passo de peso é correta pela especificação, mas ela é aplicada por
     diferença de texels do footprint (`footprint_span`), então numa textura de alto contraste a
     tolerância cresce com o conteúdo e pode aceitar um erro de peso que antes falharia. O teste cobre o
     caso de borda (textura com valores ≥ 2, borda branca); não há caso de contraste máximo no diff.
- **Observação sobre o #3147 (aberto):** é correção do próprio `429f11d5`, cujo teste de hardware ficou
  atrás de `SKIP_RETURN_CODE 77` (documentado às 21:00). Enquanto aberto, o lote de lowerings f16 de
  21:00 não está corrigido para fonte inline.

## Confiança

**Verificado por diff:** os cinco commits foram baixados e lidos por arquivo e patch; os trechos citados
(`gpu_case_runner.py`, `startup_sampler_x.cpp`, `xcrdapi.cpp`, `window_capture.py`, `pe_run_package.py`)
foram lidos na íntegra ou até o ponto que decide o comportamento. As cinco entradas de PR mesclado batem
com os cinco commits por `merge_commit_sha` (`parents` = 1 em todos).

**Não verificado:** os dez PRs abertos foram lidos por título e base, não por patch; os testes do
repositório e os casos de GPU não foram executados; os contadores do DirectStorage não foram observados
em uma execução real. A `default_branch` foi reconferida por API (`main`); o head documentado é
`88cddb0919fd4044ff45211c0919c3aefda73c35`.
