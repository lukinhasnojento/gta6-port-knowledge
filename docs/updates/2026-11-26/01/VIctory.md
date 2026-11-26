# Marvisak/VIctory — atualização de 2026-11-26 01:00

## Resumo

Sem commit novo na `main`: o cursor continua em `63bcca43` e o repositório não recebeu nada entre 03:16Z e
04:03Z. Vinte e oito PRs abertos, todos posteriores ao último número processado (`3002`) — a maioria
endurecimento de tradução de shader (recusar em vez de ignorar modificador, alinhar carga sub-dword em
CBV, formas de `mul_hi` por `UMul`/`IMul`) e correções de borda em CLI, PE e loader.

## Commits novos (0)

Nenhum. `last_commit` `63bcca43c6a00a758510398fa43096ce7e4f7df0` é o `head` da `main`; nada a documentar
por diff nesta faixa.

## Pull requests abertos (28)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 3032 | Shaders/Lower: reject unmodeled non-VOP3P modifiers in Lower::packed16 | RVZO6, +144/−2. `v_pk_fmac_f16` decodificado como VOP2 (`0x03C`) e VOP3 (`0x13C`, não VOP3P) tinha `v.neg` aplicado só à metade baixa e `v.abs`/`v.opsel`/`v.sdwa` silenciosamente ignorados. Passa a recusar (`Err`) em vez de traduzir errado. | Não é D-xx. É a classe "fail closed" da área de `D-20`/`D-24` (baixar `v_cmp_lg` como `fcmp one`; `s_addc`/`s_subb` sem carry) — modificador não modelado, não instrução. |
| 3031 | GPU/Shadow: advance saved before snapshot after merge_writeback in copy_back_or_not | RVZO6, +61/−3 em `shadow.rs`. `compared` só deduplicava pedaços inteiros de 64 KiB (`PIECE`); com duas faixas sobrepostas e uma parcial, o segundo `copy_back()` recompunha os mesmos bytes contra um `before` velho, porque `merge_writeback` copia escritas concorrentes da CPU para o host sem atualizar `old`. | Não é D-xx, mas é a mesma área de `D-07` ("copy-back sobrescrevia o que a CPU do título tinha escrito durante a submissão · crash", corrigido `35b054e`). Aqui a direção é a oposta: o snapshot fica velho depois do merge. |
| 3030 | HLE/Media: translate `\??\<drive>:\` NT object-manager paths through virtual drive mounts | RVZO6, +33/−2 em `media.rs`. `\??\G:\...` era devolvido como veio, sem passar por `files::translate`, enquanto `\\?\G:\...` logo acima já reescrevia drives virtuais. Converte para `\\?\` e chama `translate`, com fallback para o texto original quando o drive não é montado. | Não é D-xx. `D-05`/`D-06` são caminho narrow e descritor de `v_readfirstlane`; nada de VFS. |
| 3029 | CLI: keep distinct DXIL dumps when shader labels collide | passerbyyhx, +38/−8 em `apps/victory/src/translate.rs`. `victory translate --list` dizia dois shaders emitidos e deixava um `.dxil` só: `pass-a.bin` e `pass_a.bin` no offset 0 escreviam o mesmo nome. Mantém o radical de 100 bytes e prefixa com 128 bits de BLAKE3 do rótulo completo mais os bytes emitidos. | Não é D-xx. |
| 3027 | Shaders: lower v_min3/v_max3/v_med3 to DXIL integer min/max | Ingenioeren, +56/−15. Cada escolha de `v_min3_*32`/`v_max3_*32`/`v_med3_*32` deixa de ser `icmp`+`select` e vira `dx.op.binary.i32` (`IMax` 37, `IMin` 38, `UMax` 39, `UMin` 40); mede 22 → 20 instruções por forma num shader sintético. | Não é D-xx. `D-21` é ordem de `-0` em min/max **de ponto flutuante** (corrigido `bf84134`); esta é a forma inteira. |
| 3026 | xtask: bound policy file reads and report read errors | fuzeofficlal, +258/−22 em `xtask/src/policy.rs`. Para de ler arquivo inteiro na memória antes de aplicar o limite de 512 KiB (um ativo grande staged podia dar OOM no `cargo xtask policy` e no hook de pré-commit) e passa a reportar erro de leitura que não seja `NotFound`, em vez de pular o arquivo em silêncio. | Não é D-xx. |
| 3025 | GPU: graphics pipelines are made before compute ones, and compute never takes every worker | Luislariosss, +116/−13 em `pipeline/make.rs`. As filas de workers viram quatro, atendidas na ordem "gráfico que um draw espera", "compute que um dispatch espera", "iniciados cedo"; compute nunca ocupa todos os workers (deixa um). Motivo: ~180 pipelines de compute a 3,7 s cada (até 24 s) seguravam todos os workers e os draws de gráfico, que compilam em milissegundos, ficavam sem pipeline. | **Não há linha em `VIctory`** para isso. O catálogo `AvatarManz` tem `D-08` ("compilação de pipelines travava o título por dezenas de segundos", **sem corretor**) exatamente com este sintoma. Candidato a linha nova ou a correção cruzada de `AvatarManz D-08`. |
| 3024 | PE: check SizeOfOptionalHeader and slice bounds in data_directory_offset | fuzeofficlal, +77/−4 em `victory-pe/src/layout.rs`. `data_directory_offset` confiava em `NumberOfRvaAndSizes` sem verificar `SizeOfOptionalHeader` nem o comprimento da fatia, e podia devolver offset dentro da tabela de seções ou além do fim do arquivo para `victory-loader/src/prepare.rs`. | Não é D-xx. `D-04` é o ponteiro de callbacks TLS da imagem preparada; mesmo subsistema (PE/loader), defeito distinto. |
| 3023 | Shaders: s_cbranch_vccz/vccnz on a scalar VCC decide once for the wave (#2780, on #2836) | wwwwwwww3333333, +296/−54 em 5 arquivos. Um VCC que o passe de estrutura não enxerga vira `Atom::Nonzero(lo, hi)` no branch, baixado como `(lo \| hi) != 0`, uniforme; o branch passa a ser tratado como uniforme nas contagens. Vem em cima do `#2836` (dois commits de Devin rebaseados em `63bcca4`). | Não é D-xx. `D-24` (`s_addc`/`s_subb` sem carry, SCC ausente) é o vizinho mais próximo — SCC/condição escalar — mas ali o problema é a cadeia de 64 bits. |
| 3022 | GPU: read D24S8 and D32S8 planes into guest buffers | andrewpozdnakov7-jpg, +202/−11 em `list.rs`/`resource.rs`. `CopyTextureRegion` do guest descartava leitura de D24S8/D32S8; passa a copiar um plano inteiro single-sampled no mip e na fatia escolhidos, transferindo as linhas para o buffer do guest sem sobrescrever padding. | Não é D-xx. |
| 3021 | Tests: the conformance title's GPU frame, cargo conformance and cargo demo (with a stress scale), on Windows and macOS | 0xkn0wh3r3, +7348 em 24 arquivos. Depende do `#2057`. Traz o quadro de GPU do título de conformidade (shaders GFX10 montados à mão, comparados com a mesma cena em D3D12 nativo), mais `cargo conformance` e um demo com escala de estresse, em Windows e macOS. | Não é D-xx. |
| 3020 | CLI: count all SGD5 hashes and stripped-bitcode XDAT/xbop markers in shader-info | fuzeofficlal, +149/−37 em `apps/victory/src/shader.rs`. `Totals::engine` passa a ser preenchido direto de `sgd5::databases()`, e os marcadores `XDAT` (em +0x18) e `xbop` são detectados a partir do cabeçalho da parte `DXIL`, sem exigir `DxilProgram::parse` (que pede magic de bitcode). Corrige subcontagem do `victory shader-info`. | Não é D-xx. |
| 3019 | App: roll back re-entrant dependent DLLs on LateDlls attach failure | fuzeofficlal, +158/−8 em `apps/victory/src/run.rs`. Se `A.dll` recarrega `B.dll` no `DllMain` e `A.dll` devolve `FALSE`, o rollback descarregava só o lote externo e deixava `B.dll` registrada em `LateDlls::mapped`/`runtime::IMAGES` com ponteiros de IAT para memória desmapeada; `reentrant_dependents()` calcula o fecho transitivo e desfaz o lote todo. | Não é D-xx. `D-04` é a imagem preparada com ponteiro de callbacks TLS zerado; mesmo loader, defeito distinto. |
| 3018 | HLE: share DirectStorage file opens and keep the host's error code | Halfect, +47/−4 em `modules/dstorage.rs`. `OpenFile` pedia só `FILE_SHARE_READ` e falhava com `ERROR_SHARING_VIOLATION` quando as ferramentas do título ou um mod mantinham o arquivo aberto para escrita; passa a compartilhar leitura, escrita e exclusão. Também captura `last_os_error()` antes de formatar o caminho (a alocação podia sobrescrever o código thread-local). Fixes VIC-BT-037 de #1249. | Não é D-xx. `D-08` e `D-09` são copy-back e cópia de textura, não VFS/DirectStorage. |
| 3017 | Fix #2413: use saturating_add for view end bounds in aliases | tharun-37, +23/−7 em `xmem.rs` (+2/−2) e `lower.rs` (+21/−5). Em `xmem::aliases`, o fim usava `saturating_add` mas as checagens de faixa usavam soma crua (`*base + size`), que estoura e entra em pânico. | Não é D-xx. `D-11` ("aritmética de layout de textura/XMA estourava com valores do título") é a mesma classe de overflow, em outro caminho e já corrigido (`e158d95`). |
| 3016 | Shaders: the SCC a waterfall loop's s_andn2_wrexec leaves is false after the loop | Anzymanz, +55 em `lower.rs`/`valu_tests.rs`. No laço waterfall do compilador do console, `s_andn2_wrexec_b32` deixa `SCC = EXEC != 0`; depois do laço o valor tem de ser falso, e o lowering passa a fixá-lo. | Não é D-xx. `D-24` (carry de `s_addc`/`s_subb`) é a outra linha de SCC; aqui é o SCC de uma instrução de máscara dentro de laço. |
| 3015 | App: check transitive static imports of mapped title DLLs for missing files | fuzeofficlal, +49/−24 em `run.rs`. `absent_title_dlls` passa a inspecionar as tabelas de importação estática de **toda** DLL de título mapeada, não só as do executável, no mesmo caminho recursivo de `load_title_dlls`, para nomear `exe -> A.dll -> missing_B.dll` na validação de início em vez de estourar no attach. | Não é D-xx. |
| 3014 | CLI: deduplicate lib-exports entries per case-normalized DLL | fuzeofficlal, +85/−16 em `inspect.rs`. Agrupa por `(&ImportSymbol, LibImportKind)` dentro de um balde com o nome da DLL em ASCII minúsculo e calcula o total a partir dos grupos, para `Example.dll`/`example.dll` não contarem duas vezes. | Não é D-xx. |
| 3013 | GPU: keep untracked pages writable in Tracked::taken | fuzeofficlal, +141/−3 em `watch.rs`. `Tracked::taken` filtrou `pages` contra `g.holders` antes de chamar `aliases`/`guard_aliases`/`protect`: uma página não rastreada que caísse numa janela de 4 MiB do `Shadow` ia para `VirtualProtect(PAGE_READONLY)` sem entrada em `g.holders` e ficava somente-leitura para sempre, mesmo depois do `Tracked::drop`. | Não é D-xx. `D-06` ("corrida entre mapear uma view e abrir os guards das outras views da mesma página") é a linha vizinha de proteção de página; o defeito aqui é vazamento de proteção, não corrida. |
| 3011 | Build: thin LTO, x86-64-v3 on Windows, xxh3 for texture change detection | 0xkn0wh3r3, +34/−5. Thin LTO no release (e no `diag`), `target-cpu=x86-64-v3` só para `x86_64-pc-windows-msvc`, e `xxh3` no lugar do hash anterior na detecção de mudança de textura. Declara não mudar comportamento. | Não é D-xx. |
| 3010 | Shaders/Lower: return 6 faces for channel 2 of image_get_resinfo on SQ_RSRC_IMG_CUBE | RVZO6, +203 em `lower.rs` e teste. `image_get_resinfo` de cubemap devolvia 1 no canal 2 (`depth`); passa a devolver 6 (número de faces), citando GCN3 §12.16, RDNA3 §10.1, `nir_lower_tex.c` e `ImageBuilder.cpp`. | Não é D-xx. |
| 3009 | Shaders/Lower: reject non-zero soffset/built_offset on typed buffer_store_format_xyzw | RVZO6, +118 em `lower.rs`. O caminho de store tipado só recusava `count != 4`, `offen` e `offset != 0`; com `soffset` escalar ou constante inline não-zero (ou `built_offset` somando base) a instrução descartava o offset e escrevia no elemento `index` em offset 0. Passa a recusar como o caminho de load. | Não é D-xx. |
| 3008 | Shaders/Lower: align and extract sub-dword buffer_load_{ubyte,sbyte,ushort,sshort} on CBVs | RVZO6, +345/−21. No caminho de `ResourceType::Cbv`, `sub = Some((8\|16, signed))` era ignorado: em tabela CBV o offset ia cru para `rawBufferLoad.f32` com `align = 4`, e em root CBV devolvia os 32 bits do dword sem deslocar por `(byte & 3) * 8`. Fatora `Lower::extract_sub` compartilhado com o caminho SRV/UAV. | Não é D-xx. |
| 3007 | Shaders: lower the 32-bit mul_hi forms to one UMul/IMul | Ingenioeren, +129/−9. As formas escalar e vetorial de `mul_hi` de 32 bits (e as duas de 24 bits, após extensão de sinal/zero) passam por um helper que emite `UMul`/`IMul` do DXIL e extrai o dword alto, em vez de expandir em produtos 16×16 — remove a exigência de `INT64`. Alternativa ao `#2148`. | Não é D-xx. |
| 3006 | GPU: read single-plane depth textures into guest buffers | andrewpozdnakov7-jpg, +178/−57. O caminho de readback de `CopyTextureRegion` descartava qualquer recurso de profundidade; passa a aceitar D16 e D32 single-sampled inteiros com footprint de plano único, checando sub-recurso de origem, origem de destino e a regra de sub-recurso inteiro. | Não é D-xx. |
| 3005 | GPU: make already-used pipeline lookups read-only | AwsAli05, +24/−5 em `pipeline/make.rs`. Evita troca atômica incondicional e segunda checagem de prontidão a cada draw/dispatch que usa pipeline pronta; a troca atômica continua escolhendo uma thread para registrar o primeiro uso. | Não é D-xx. Mesma área do `#3025` (fila de pipelines), que também não tem linha. |
| 3004 | CLI: discover compressed shader packs in archives without SGD5 or header magic | fuzeofficlal, +242/−18 em `prebuild.rs`. Separa descoberta de pacote de shader da indexação de container e passa a chamar `shaderpack::find` em arquivos só-DXBC e em arquivos de arquivo não-shader dentro de `MAX_FILE`, para `victory translate --cache --pipelines` não pular pacote dentro de stream zlib alinhado a 16 bytes sem cabeçalho `PS\0\x09` nem `SGD5`. | Não é D-xx. |
| 3003 | CLI: deduplicate translate containers by declared container bytes instead of DXIL payload alone | fuzeofficlal, +72/−1 em `translate.rs`. O hash passa a cobrir o container DXBC pelo tamanho declarado validado, não só a parte `DXIL`/`ACCC`, para não pular containers distintos com payload igual e metadados/signaturas diferentes (`ISG1`, `OSG1`, `PSV0`), que `emit::emit_with` lê. | Não é D-xx. |

## Pull requests mesclados (0)

Nenhum. A `main` não tem commit novo; nenhum PR mesclou nesta faixa.

## Cruzamento com o catálogo

- Corrige D-xx? Nenhum PR aberto declara correção de linha do catálogo `VIctory`. As 27 linhas seguem com
  os corretores registrados; `D-03` (descritores nulos do tipo/dimensão errados), `D-25` (testes VALU de
  GPU travando `cargo test`) e `D-26`… continuam sem corretor.
- Abre defeito novo? Dois candidatos em PR aberto:
  - `#3025`: a fila de pipelines deixava gráficos atrás de compute de 3,7 s (até 24 s cada) e os draws
    ficavam sem pipeline. É `AvatarManz` `D-08` com outro código, e não há linha em `VIctory`.
  - `#3032`: modificador não modelado em `v_pk_fmac_f16` era ignorado em silêncio (`v.abs`, `v.opsel`,
    `v.sdwa`) e `v.neg` negava só a metade baixa. Tradução errada silenciosa, sem linha no catálogo; o PR
    fecha a recusa.
- Áreas de linha existente tocadas sem mudar o veredito: `#3031` (copy-back, área de `D-07`),
  `#3017` (overflow de aritmética, área de `D-11`), `#3013` (proteção de página, área de `D-06`),
  `#3024` e `#3019` (PE/loader, área de `D-04`). Nenhum reabre o defeito original.

## Confiança

Verificado por API: `gh api repos/Marvisak/VIctory/commits?sha=main&per_page=100` devolve `63bcca43` como
`head`, igual ao `last_commit` do estado — zero commits na faixa. `gh pr list --state open --limit 50`
devolveu 28 PRs, todos com número > `last_pr_number` 3002; `gh pr list --state merged` com
`merged:>=2026-11-26T03:16:53Z` devolveu lista vazia (e a coleta por `gh api .../pulls?state=closed`
confirmou: nenhum `merged_at` posterior ao cursor).

Não verificado: nenhum diff de PR foi lido além dos arquivos e do corpo que a API devolve; as medições
citadas (instruções por forma no `#3027`, tempos de compilação no `#3025`, 3,7 s/24 s) são declarações dos
autores. Não rodei `cargo test` nem `cargo check` em nenhum branch.
