# Atualizações horárias

Registro incremental dos treze repositórios monitorados: commits novos, pull requests abertos e
pull requests mesclados, documentados a cada hora pela rotina descrita em
[`docs/automacao/ANALISE_HORARIA.md`](../automacao/ANALISE_HORARIA.md).

As análises históricas completas ficam em [`docs/analises/`](../analises/); aqui só entra o que mudou
depois do fechamento do acervo.

## Como navegar

```
docs/updates/AAAA-MM-DD/HH/<repo>.md    um arquivo por repositório com mudança
docs/updates/AAAA-MM-DD/HH/_falhas.md   erros daquela execução, se houver
```

## Índice

| Data | Hora | Repositórios com mudança | Resumo |
|---|---|---|---|
| 2026-11-28 | 00:00 | 4 | Triagem dos 14 com deep dive em 4: 17 commits documentados de 17 na janela (`Xbox2Windows` 12, `TuxBox` 2, `SixNG` 2, `victory-fork` 1), 14 PRs mesclados (`Xbox2Windows` 12, `SixNG` 2) e 123 PRs abertos acima dos cursores (`VIctory` 76, `xcl` 24, `Xbox2Windows` 14, `TuxBox` 3, `victory-fork` 2, `SixNG` 2, `VItoriaBr` 1, `xbox-emulator` 1). **Achado principal:** em `Xbox2Windows` o commit `98020080` (#3159) publica um teste que compara guardas com `F.SENTINEL`, símbolo que `tests/native_f16_alu_check.py` **não define** nesse commit (conferido nos dois arquivos por API), então a perna de hardware levanta `AttributeError`; o conserto está no PR aberto `#3190`. Na mesma janela `8e8b349b` é só documento, mas registra que com o WARP 1.0.22 app-local três testes passam a falhar (`startup_compute_dispatch_native_warp` com 288 palavras diferentes, `native_startup_request_cache_warp`, `bvh_wave64_warp`), nenhum corrigido. O resto da hora fecha recusas do compute direto: `s_cmov_*`, comparações SOPC de 64 bits, as quatro conversões `f16`↔i16 e `v_sat_pk_u8_i16`, mais `cf3df832` tirando o bit 25 do `MIMG` da máscara de reservados. Em `victory-fork` o commit direto `2eade68c` põe portão em bloco de textura de 64 MiB (que só volta à placa quando esvazia inteiro): sem espaço a textura espera e mostra a view anterior, até ~4 s, e nada no diff é teste — a dívida de D-17 continua. Em `TuxBox` `b1023549` traz um harness que compara tradução contra shaders gravados e `62c7e390` um roadmap que admite 178 das 372 instruções usadas sem check de GPU e zero shaders reais executados contra referência; o harness exige config local fora do Git e não roda em CI. Em `SixNG` `#503` escolhe **um** registro de transmissão por material pelo sub-material que a maioria dos vértices nomeia e o renderizador Metal 4 não consome o registro (`glass: None`). `Belobug` e `avatarmanz` seguem em 404 pela quarta hora |
| 2026-11-27 | 23:00 | 4 | Triagem dos 14 com deep dive em 4: 36 commits documentados de 36 na janela (`TuxBox` 21, `SixNG` 9, `Xbox2Windows` 5, `DexNom` 1), 18 PRs mesclados (`SixNG` 9, `Xbox2Windows` 5, `TuxBox` 4) e 115 PRs abertos acima dos cursores (`VIctory` 67, `xcl` 22, `DexNom` 14, `Xbox2Windows` 5, `victory-fork` 2, `TuxBox` 1, `SixNG` 1, `VItoriaBr` 1, `xbox-emulator` 1). **Achado principal:** em `DexNom` a fila de 14 PRs é o efeito do publish `570a941` (2026-11-25, 170 arquivos): 9 restauram commits derrubados por `cherry-pick -x` e 5 são correções novas com teste de hardware, mas **nenhuma reaplica D-10, D-11, D-12 nem D-15**, que seguem abertos por reversão. Em `TuxBox` o hold de backing sai do `ImportedHeap` e é pendurado no `ID3D12Heap` por `SetPrivateDataInterface` (`2f296416`, mesma janela de uso-após-liberar de D-17), o emissor passa a provar reduções de wave com ballot de máscara e aposentadoria de lanes (`69106310`) e a repetir a tradução sem guardas quando o dispatcher por blocos perde o voto (`60f7cfad`), e o snapshot de upload libera as páginas de rascunho com `MEM_RESET` (`07db0df9`). Em `Xbox2Windows` a linha `x2w-budget` ganha commit do sistema, limite de job e contadores do pool (`6057071b`), `texture_unavailable` vira três motivos (`0f3e8248`) e `pixel_aot`/`gfx2hlsl` ganham `v_cvt_norm_(i16\|u16)_f16` e `s_bitreplicate_b64_b32` (`968ddcdf`, `3c30dc74`). Em `SixNG` o export de cabelo assenta as guias contra o corpo (`a31b51a2`), o detalhe de pele passa a filtrar dentro do tile (`b2effe97`) e o switch novo de detalhe (`e607e6d3`) **não tem efeito no renderizador Metal 4** (lido `render3d_metal4.rs` em `ede2c253`). `Belobug` e `avatarmanz` seguem em 404 pela terceira hora |
| 2026-11-27 | 22:00 | 4 | Triagem dos 14 com deep dive em 4: 39 commits documentados de 39 na janela (`TuxBox` 25, `SixNG` 8, `Xbox2Windows` 5, `victory-fork` 1), 20 PRs mesclados (`SixNG` 8, `TuxBox` 7, `Xbox2Windows` 5) e 99 PRs abertos acima dos cursores (`VIctory` 55, `xcl` 20, `Xbox2Windows` 10, `SixNG` 8, `TuxBox` 2, `DexNom` 2, `VItoriaBr` 1, `xbox-emulator` 1). **Achado principal:** em `victory-fork` o commit `cd935520` corrige as listas de tiles da luz da agua em host de 32 lanes (blocos escuros de 16x16 em volta de barcos e pieres) reescrevendo os dois appends do programa do titulo byte a byte, mas o switch novo `VICTORY_WATER_TILE_SLOTS` **nao entra na chave do cache de shader** (lido em `crates/victory-gpu/src/shader_cache.rs` em `cd935520`), reabrindo a classe de D-02; a mesma lista continua sem `VICTORY_MEMORY_NEAR`, `VICTORY_LDS_START` e `VICTORY_WAVE_LOOP_SKIPS`. Em `TuxBox` a hora mais movimentada da rotina: 17 commits de ramo preenchem do descritor os campos que o programa le (tamanho de textura, contagem de registros de buffer, stride de estruturado), verificam a eleicao de lane por registradores exatos e rodam um grupo de 32 threads como metade de wave de console, com retencao de cache de runtime por `flock`; `1ad86d0b`/`f7e404cd` passam a declarar um buffer usado so pela contagem e `0a5ebfa7` le textura vazia como um texel; o merge `dc92a00e` traz de volta seis commits ja documentados (helper DXC, #312, #313). Em `SixNG` saem os perfis de subsuperficie da pele do arquivo de perfis (`d8101f48`), as larguras de fio das rampas (`01f50f8e`) e as tatuagens de braco unico no quarto UV (`4acbe12f`, `BAKE_RULE` 6->8); `77c5fbc6` culla mesh por mesh com so quatro planos laterais e `97eca4cb` le livery fora da thread de UI sem limite de threads no diff. Em `Xbox2Windows` `88cddb09` carrega provedor invalido com `SEM_FAILCRITICALERRORS` por thread em vez do dialogo modal, `e92776bd` imprime `raw=`/`completed=` com `completed = raw + inflated + bcpack-blank` (contradizendo o comentario de que comprimido fica sem fazer), `0e3ebe36` amplia a tolerancia dos probes lineares para um passo de peso de 1/256 escalado pelo span do footprint e `572ec6f8` da um device D3D12 por teste. `Belobug` e `avatarmanz` seguem em 404 pela segunda hora |
| 2026-11-27 | 21:00 | 4 | Triagem dos 14 com deep dive em 4: 13 commits documentados de 13 na janela (`Xbox2Windows` 5, `TuxBox` 4, `SixNG` 3, `victory-fork` 1), 9 PRs mesclados (`Xbox2Windows` 5, `SixNG` 3, `TuxBox` 1) e 67 PRs abertos acima dos cursores (`VIctory` 38 de 1.675, `xcl` 9, `TuxBox` 8 de 11, `Xbox2Windows` 3 de 113, `victory-fork` 3 de 947, `DexNom` 2 de 122, `SixNG` 2, `xbox-emulator` 1 de 25, `VItoriaBr` 1 de 75). **Dois dos 14 deixaram de existir para a API**: `Belobug/VIctory-Belobug-AMD` e `avatarmanz/VIctory-7900XTX` devolvem HTTP 404 em `repos/<o>/<r>` e em `users/<o>` - a conta desapareceu -; os cursores deles ficaram parados e o caso esta em `_falhas.md`. Achados: em `Xbox2Windows` quatro lowerings novos no caminho direto de compute (`v_dot8_i32_i4`/`u32_u4`, `v_sat_pk_u8_i16`, `v_frexp_*` f32 e f16), todos com controle de compilacao e teste de hardware que se auto-ignora (SKIP 77); o mesmo `v_dot8_i32_i4` esta catalogado como defeito no `VItoriaBr` (D-15, `75aa531`), entao a lacuna existia em dois forks; em `TuxBox` o #312 (`8c082e65`) faz o tradutor DXBC aceitar root SRV de compute depois das tabelas (antes so raster com varias tabelas), com teste que fixa o fail-closed, e `89fb2517` troca a compilacao de shader por um helper DXC que remove o passe `correlated-propagation` (51 s contra 60 s em 1.500 requisicoes) e reescreve a mao a propriedade 11 do entry point para devolver `[WaveSize(N)]`, que o primeiro estagio perde - fora da alca de D-02/D-03, que seguem abertos; em `SixNG` o #476 e correcao de posse (`&Vec<u8>` no staging), o #472 adiciona bake offline de cache de modelo (macOS/metal4) e o #481 leva as chaves de mandibula ao viewer, extraindo `fit_keys`/`add_keys`; em `victory-fork` um unico commit direto de 19 arquivos (+1.914/-108) junta contador de threads por grupo (ported de `VIctory-NVIDIA 854f57f`), skips em volta de loops escalares, LDS inicial do grupo, reuso de consultas ao diretorio pelos ray walkers e um portao de slots dinamicos (`VICTORY_THIN_GATE`), com `VICTORY_WATER_LIGHT=1` por padrao. **Achado verificavel**: a chave do cache de shader passou a incluir `VICTORY_THREAD_TICKET`, mas nao `VICTORY_MEMORY_NEAR`, `VICTORY_LDS_START` nem `VICTORY_WAVE_LOOP_SKIPS`, que mudam o codigo gerado - o comentario de `memory_near` afirma que a chave inclui -, entao a classe de D-02 (traducoes envenenadas por switch) volta a valer por tres switches novos. Ficam para a proxima hora, com cursor parado: `Belobug` e `avatarmanz` (bloqueados por 404), `VIctory` (38 acima do cursor, 1.675 abertos), `xcl` (9), `xbox-emulator` (`#395`), `DexNom` (2) e `VItoriaBr` (1) |
| 2026-11-27 | 20:00 | 4 | Triagem dos 14 com deep dive em 4: 49 commits documentados de 49 na janela (`Xbox2Windows` 37, `TuxBox` 6, `SixNG` 4, `xcl` 2), 9 PRs mesclados (`SixNG` 4, `Xbox2Windows` 3, `TuxBox` 1, `xcl` 1) e 191 PRs abertos acima dos cursores (`Belobug` >=100 de 374, `VIctory-7900XTX` 43 de 240, `VIctory` 26 de 1.663, `Xbox2Windows` 10 de 115, `xcl` 3, `victory-fork` 2 de 946, `SixNG` 3 na coleta, `TuxBox` 1, `xbox-emulator` 1, `VIctory-DexNom` 1, `VItoriaBr` 1). Achados: em `Xbox2Windows` o #2911 (`63a30dc1`, 30 commits) troca falha-por-falha o custo das page runs do XMem sob Wine - split do placeholder antes do map, `stable_call` para leitores que nao geram fault, retry de fault por stamp do break em vez do anel de 64, join entre chamadas atras de `X2W_XMEM_JOIN_CALLS` com teto de 64 paginas, retry de `ReadFile`/`WriteFile` so em arquivo de disco - e o fallback de slot 31 para OS que esconde o bit de placeholder cria uma variante de D-02 restrita a Wine, documentada como limite em `f9391cca` (nao distingue pagina de alocacao privada com `MEM_REPLACE_PLACEHOLDER` e, apos view recusada, corta a pagina da alocacao do chamador); o #3130 foi mesclado em `codex/native-graphics-integration` (`b80513fd`), nao na `main`, entao merge nao aparece na faixa de commits; o #3114 (`2641cc72`) leva `s_quadmask_b32`/`_b64` ao lowering direto de compute. Em `TuxBox` `049e5a43` e `65e3e48c` reescrevem `tb_float_relation` e `tb_f16_cvt` sem branches e refixam as 48 linhas de `host_profiles.txt` - todo cache de traducao invalidado - com equivalencia medida pelo autor, sem teste de referencia no repositorio; `55ec224a` passa a reportar como SKIP a checagem de splash onde a GPU apresenta por software (primeiro frame nao completa, causa desconhecida), o que remove o sinal de falha desse arranjo; `7e2c2408` impede o script do devcontainer de matar Xwayland do host; o #312 aberto le root SRV de compute depois das tabelas, area de D-22 mas nao D-22. Em `SixNG` quatro squash (#475, #478, #477, #479, dois mesclados durante a coleta): a ordem dos indices do buffer compartilhado passa a ser por tipo e material com juncao de draws, o clique do painel de liveries passa a testar o raio contra a malha posada pelas juntas, e um modulo `jawkeys` da aos dentes e a lingua o movimento da mandibula com limiares heuristicos (30% de residuo, 1 mm). Em `xcl` o #880 (`6576d75b`) faz a estatistica e o dump de `xcl shaders` tirarem o tamanho de wave do cabecalho FSSM, como os pipelines; so 2 arquivos entraram porque o PR estava empilhado em #573; os abertos #3040/#3042 sao portes declarados de `Marvisak/VIctory 63bcca4`, que e o cursor atual do `VIctory` monitorado, e o #3039 aponta para `nvidia-dev`. Ficam para a proxima hora `Belobug` (374 abertos, zero commit), `VIctory-7900XTX` (240), `VIctory` (1.663, 26 acima do cursor) e `victory-fork` (946, 2 acima) |
| 2026-11-27 | 19:00 | 4 | Triagem dos 14 com deep dive em 4: 36 commits documentados de 36 atras dos cursores (`Xbox2Windows` 16, `TuxBox` 12, `victory-fork` 6, `xcl` 2), 20 PRs mesclados (`Xbox2Windows` 16, `TuxBox` 3, `xcl` 1) e 19 PRs abertos acima dos cursores (`Xbox2Windows` 10, `xcl` 6, `victory-fork` 3, `TuxBox` 0). Achados: em `Xbox2Windows` a serie #2935 fecha recusas do lowering direto de compute uma por uma (`s_brev_b64`, `s_bfe_*_u64`/`_i64`, `v_cvt_off_f32_i4`, mul24, os quatro `v_pk_*` de 16 bits, `s_lshl1..4_add_u32`, `v_xnor_b32`, bitscan64, `s_sext`, mad32 sem FMad, mad64, add/sub saturado) e o #2919 passa a gravar receitas de estado de PSO em arquivo de 64 registros/64 KiB com handle exclusivo; em `TuxBox` o #310 retenta a recusa de stride IDXEN anexando um root SRV `t1, space32766` a root signature embutida (nao e D-12, que e o stride de descritor inline de 32 bytes) e o #311 desenha o splash do pacote ate 4 frames chegarem em 1 s; em `victory-fork` a `main` passa a ligar ray tracing (`VICTORY_BVH_BUILDER=1`) e `VICTORY_DYNAMIC_WAIT=1`, as paginas de fault viram orcamento reciclavel de 512 MB e a CI morre no Clippy antes dos testes (#1485, a divida de D-17 continua); em `xcl` o #1871 adiciona `xcl pso-precompile` e a pilha #3033-#3035 ordena a espera depois de escrita de CPU adiada na mesma lista, caso que na `main` estoura 2 s e roda sem a escrita (candidato a defeito novo de catalogo). Ficam para a proxima hora `Belobug` (>=100 abertos acima de #456, zero commit), `VIctory-7900XTX` (39), `VIctory` (19), `SixNG` (1 commit), `xbox-emulator` (#395), `DexNom` (#135) e `VItoriaBr` (#132) |
| 2026-11-27 | 18:00 | 4 | 16 commits documentados de 17 na janela (`Xbox2Windows` 11, `TuxBox` 2, `xcl` 2, `victory-fork` 1; o `SixNG`, 1 commit de performance em `render3d`, ficou fora por não ter sinal), 6 PRs mesclados (`Xbox2Windows` 4, `TuxBox` 1, `xcl` 1) e 39 PRs abertos acima dos cursores (`Xbox2Windows` 14, `victory-fork` 13, `xcl` 10, `TuxBox` 2). Achados: em `Xbox2Windows` `246857e459` (#3058) desfaz a conversão U8 empacotada por `Round_ne`/`fptoui` do #2997 — em driver NVIDIA o literal 0,5 voltava como 1 (#3052) — e passa a arredondar ties-to-even em inteiro; `2972e00aaf` (#2871) habilita volume 0x105 de um mip no slot 70, com blocos grossos e passo de camada no `detile`; `36d2cbebde` (#3028) torna opcional a cópia de 64 KiB do root SRV do slot 44; `ff04a75e58` (#3056) cria um carregador de manifesto de patch com gate de SHA-256 que ainda não aplica nada; em `victory-fork` `5dbf5ce226` substitui a árvore inteira pela de `trustno1-coder/VIctory-NVIDIA@959dcaf` — 284 arquivos, +95.231/−913.607 linhas — e os símbolos dos consertos D-01 a D-06 estão presentes no topo novo, mas a equivalência semântica não foi medida; em `TuxBox` `b46c3db9` (#309) fixa a opção do DXC no teste de fallback `-O0`, que passou a depender da versão instalada desde `8a3b6df`, e o #310 em aberto retenta 374 recusas de IDXEN na preparação offline (área de D-12); em `xcl` a `main` ganha perfis de teclado e mouse por título (#1824) e o #3023 em `amd-dev` corrige `V_CMP_LG_F32` para RDNA4, a mesma classe de `VIctory D-20`. Sem delta em `OpenScarlett`, `VItoriaBr`, `DexNom`, `GTA6-Asset-Info` e `kelvaroq`. Ficam para a próxima hora `Belobug` (61 abertos acima de #456), `VIctory-7900XTX` (39), `VIctory` (8), `SixNG` (1 commit) e `xbox-emulator` (#395) |
| 2026-11-27 | 17:00 | 4 | Triagem dos 14 pela primeira vez no formato novo: 55 commits documentados de 62 na janela (`Xbox2Windows` 40 de 47 novos — teto de 40, `TuxBox` 14, `SixNG` 1, `DexNom` 0), 9 PRs mesclados (`Xbox2Windows` 6, `TuxBox` 2, `SixNG` 1) e 32 PRs abertos acima dos cursores (`DexNom` 26, `Xbox2Windows` 5, `TuxBox` 1). Achados: em `Xbox2Windows` o merge `91cbedd6` (#3039) leva o tradutor DXIL nativo em C++ para a `main` (25.403 linhas em 161 arquivos) e, na mesma janela, `f6cdee10` (#3020) devolve `D[31:16]` ao binário16 direto — o contrato de #2964 documentado às 16:00 estava errado — e `1c879506` (#3040) conserta a escala do descritor BVH e os slots de resultado que `0d228580` (#2922) acabara de introduzir; `98007de5` (#2899) reescreve a documentação do ciclo de `.fail` (área de D-19) e `f17a08fd` (#3024) passa a não tratar `UINT64_MAX` como conclusão no sampler de assinatura (área de D-01); em `TuxBox` os merges `497c561a` (#272) e `6ca0d4ac` (#277) trazem a frente de verificação de mesh (fixture NGG real, recibo de worker para mesh/pixel, proibição de tradução sem root) e a junção de esperas de placement, e `18736af5` traduz de novo com a wave junta os programas que leem lane em laço divergente; em `SixNG` o #474 (`2d9c230d`) corrige o base vertex somado duas vezes na máscara de blur do cabelo desde #470 — o commit que era o cursor das 07:00. Sem delta em `OpenScarlett`, `VItoriaBr`, `GTA6-Asset-Info` e `kelvaroq`. Desvio registrado: a "tabela de pistas" que o procedimento cita não existe em `ANALISE_HORARIA.md`; as áreas foram tomadas das seções D do catálogo |
| 2026-11-27 | 16:00 | 4 | 64 commits documentados de 104 na janela (`Xbox2Windows` 40 de 80 — teto de 40, `TuxBox` 19, `xcl` 5, `VIctory` 0 — a `main` de `VIctory` não se moveu desde antes do cursor), 55 PRs mesclados (`Xbox2Windows` 50, `xcl` 3, `TuxBox` 2) e 64 PRs abertos acima dos cursores (`VIctory` 50 de 3456 a 3521, `xcl` 50 de 2962 a 3022, `Xbox2Windows` 14 de 3014 a 3060, `TuxBox` 0). Achados: em `Xbox2Windows` entra inteiro o frontend DXIL nativo em C++ (`4036577020`, 15.618 linhas em 69 arquivos) e, na mesma faixa, `7beb331d` conserta um erro de revisão de chave que fazia o `compile_dxil` recusar *todos* os pedidos do runtime (chave `-14` contra `-28` publicada) e `e97c5c7c` contorna o crash do `amdxc64` na criação de pipeline por falta de `!llvm.ident`; `7f951fd3` reverte deliberadamente os 2.544 linhas de XMem de #2911 fora de `cpp-migration`, o que deixa a branch nativa sem os consertos de run de página; em `TuxBox` o `#306` (`07db52f8`) leva as sombras do ProjectScarlett a ray queries nativas e o `#308` (`acd63597`) retenta raiz raster com a tabela de strides do heap, mas o próprio guia passa a registrar que o jogo chega ao prólogo e cai (`138f3bbf`) e que o `XboxBvhBuilder.dll` do pacote não compila no vkd3d-proton (#307); em `xcl` a `main` ganha `XCL_PREPARE=0`, um grooming de heap de driver proporcional ao tamanho da janela (`f78e05bc`, janela de 512 MB mapeava o heap fora do buraco) e a relicença para GPL-3.0-or-later (`06f90c19`, a partir da issue #3002). Desvios de coleta: `gh pr list --search "merged:>..."` devolveu vazio no `xcl` (repositório transferido) e 49 de 50 no `Xbox2Windows`, perdendo o `#2882` — a coleta de mesclados passou a usar a API REST de pulls fechados; e os timestamps do GitHub ficaram 3 h atrás do UTC real, o que fez a execução de 15:00 gravar cursores de mesclados no relógio errado em quatro repositórios sem merge (`OpenScarlett`, `VItoriaBr`, `victory-fork`, `GTA6-Asset-Info`), corrigidos aqui depois de reconferidos. Ficam em `backlog`: `Xbox2Windows` (os 40 commits acima de `f5204690`, com `commit_cursor`, mais os diffs do `#3055` e do `#2882`), `VIctory-DexNom` (25 acima de 106), `xbox-emulator` (1, #395), `VIctory-7900XTX` (35 acima de 419), `VIctory-Belobug-AMD` (50 acima de 456), `SixNG` (1 acima de 472) e `victory-fork` (4 acima de 1468) |
| 2026-11-27 | 15:00 | 4 | 66 commits documentados de 73 na janela (`TuxBox` 40 de 47 — teto de 40, `SixNG` 24, `VItoriaBr` 1, `victory-fork` 1), 32 PRs mesclados de 33 (`SixNG` 24, `TuxBox` 8 de 9 — o #306 caiu fora do teto de commits) e 36 PRs abertos documentados de 36 detectados (`victory-fork` 35 acima de 1427, `SixNG` 1, `TuxBox` 0 acima de 284, `VItoriaBr` 0 acima de 126). Achados: `TuxBox` fecha a frente do issue #232 (leitura de `ExecuteIndirect` depois da escrita, heap de descritor paginado limpo ao entrar) e do #215 (escrita de textura de programa indexado direto estreitada aos descritores de tabela rastreados — 275 texturas/760 MB por lote antes), mais cinco programas da ProjectScarlett com as caminhadas de BVH inline trocadas por ray queries nativas; `SixNG` sobe o catálogo de enums de 3.253 para 5.982 nomes e leva liveries de veículo para o GLB como decals (SIX-248); `victory-fork` destrava o construtor de estruturas de ray tracing (o watchdog de 30 s do título era a sincronização de todas as cópias em todas as listas) e faz os raios verem as estruturas — mas registra 0,2-3 fps com o construtor ligado e o PR #1466 já corrige um defeito de `PULL_AFTER` aberto contra este commit; `VItoriaBr` só corrige a sonda de adaptador de software em teste. Ficam em `backlog` pelo limite de 4: `xcl` (5 commits, 4 merges, 50 abertos acima de 2708), `Xbox2Windows` (46 commits, 47 merges, 12 abertos acima de 3011), `VIctory` (50 abertos acima de 3057), `VIctory-Belobug-AMD` (50 acima de 456), `VIctory-DexNom` (25 acima de 106), `VIctory-7900XTX` (33 acima de 419), `xbox-emulator` (1, #395) e `TuxBox` (7 commits acima de `90df685a` e o merge #306). `OpenScarlett` e `GTA6-Asset-Info` sem delta |
| 2026-11-27 | 05:00 | 4 | 20 commits documentados de 20 na janela (`Xbox2Windows` 12, `xcl` 6, `SixNG` 2, `VIctory` 0 — a `main` de `VIctory` não se moveu), 21 PRs mesclados de 21 (`Xbox2Windows` 16, `xcl` 3, `SixNG` 2) e 55 PRs abertos documentados de 575 detectados nos quatro (`VIctory` 20 de 380, `xcl` 20 de 180, `SixNG` 10 de 10, `Xbox2Windows` 5 de 5). Achados: no `xcl` a `main` voltou a andar (`8b957bec` → `c2ee10e4`) e 3 dos 6 commits são o mesmo PR #2924 (fusão + o teste de TTPF e a contagem de mips); `#2927` foi mesclado em `nvidia-dev`, não na default; no `Xbox2Windows` 4 dos 16 merges foram para `cpp-migration` (`#2986`, `#2996`, `#2998`, `#3003`) e não entram no cursor de commits, e `923c0ac9` (#2979) rotaciona as duas chaves de shader do runtime (cache invalidado). Desvio de coleta: `gh pr list --state merged --limit 50` devolveu 15 dos 17 merges do `Xbox2Windows` e perdeu o `#2661` (criação antiga, merge às 05:02:20Z); a coleta passou a usar `search/issues` com `merged:>=<last_run>`. Ficam em `backlog` pelo limite de 4: `VIctory` (360 abertos acima do novo cursor 3057), `xcl` (160 acima de 2708), `VIctory-DexNom` (24), `VIctory-Belobug-AMD` (12), `VIctory-7900XTX` (10), `victory-fork` (8), `TuxBox` (4) e `xbox-emulator` (1) |
| 2026-11-27 | 01:00 | 4 | 29 commits documentados de 29 na faixa (`victory-fork` 14, `SixNG` 10, `Xbox2Windows` 5, `xcl` 0 — a `main` do `xcl` não se moveu) e 42 PRs abertos documentados de 223 detectados nos quatro + 26 mesclados de 26 (`SixNG` 10, `Xbox2Windows` 10, `xcl` 6, `victory-fork` 0). `xcl` é o único dos quatro que continua no backlog, com 181 abertos acima do novo cursor 2685. Achados: no `xcl` os 6 merges foram para `nvidia-dev` (nenhum na default); no `Xbox2Windows` 5 dos 10 merges foram para `cpp-migration`, então só 5 commits chegaram à `main`; `4966cd67` em `victory-fork` liga o construtor de BVH por padrão e o README do mesmo ciclo (`67429060`) documenta queda do título escrevendo no endereço 0 entre 30 e 40 s depois, sem correção — candidato a defeito novo; `9e0a3980` no `SixNG` deixa de gravar `archetype_kind` em `placements` enquanto o README reescrito na mesma hora o documenta. Sete repositórios com delta ficam em `backlog` pelo limite de 4: `VIctory` (50 abertos), `xcl` (181), `VIctory-Belobug-AMD` (8), `VIctory-DexNom` (24), `VIctory-7900XTX` (8), `xbox-emulator` (1) e `TuxBox` (2) |
| 2026-11-27 | 00:00 | 4 | 33 commits documentados (`SixNG` 26, `TuxBox` 5, `Xbox2Windows` 2, `VIctory-Belobug-AMD` 0) e 105 PRs abertos de 105 detectados nos quatro + 66 mesclados de 66. `VIctory-Belobug-AMD`, `SixNG` e `Xbox2Windows` saem do backlog. Dois achados estruturais: em `Xbox2Windows` 16 dos 18 merges foram para `cpp-migration`, não para `main`, explicando 18 merges contra 2 commits na branch default; em `VIctory-Belobug-AMD` os 20 merges foram para `a-publier`, que é a branch default real do repositório (a `main` está parada em `4f62c2a5` desde 25/11), e há 239 PRs abertos contra uma janela de 50. Seis repositórios com delta ficam em `backlog` pelo limite de 4: `VIctory` (50 abertos), `xcl` (50), `VIctory-DexNom` (24), `victory-fork` (10), `VIctory-7900XTX` (3) e `xbox-emulator` (1) |
| 2026-11-26 | 23:00 | 4 | 81 commits documentados de 81 na faixa (25 em `VIctory-7900XTX`, 20 em `victory-fork`, 35 em `TuxBox`, 1 em `VIctory-DexNom`) e 40 PRs abertos de 98 detectados nos quatro + 7 mesclados de 7; `TuxBox` e `victory-fork` retirados do backlog. Seis repositórios com delta ficaram em `backlog` pelo limite de 4 por execução: `VIctory` (50 abertos), `xcl` (50 + 6 merges), `Xbox2Windows` (43 + 16), `VIctory-Belobug-AMD` (50 + 20), `SixNG` (12 commits, 14 abertos, 12 merges) e `xbox-emulator` (1 aberto). `VItoriaBr` saiu do backlog: sem delta |
| 2026-11-26 | 22:00 | 4 | 127 commits na janela, 109 documentados (32 em `Xbox2Windows`, 40 de 58 em `VIctory-7900XTX`, 31 em `TuxBox`, 6 em `SixNG`) e 33 PRs abertos de 63 detectados nos quatro + 19 mesclados em `main` (13 `TuxBox`, 6 `SixNG`); outros seis merges do `Xbox2Windows` foram para `cpp-migration`, não para `main`. Sete repositórios com delta só de PRs e `VIctory-7900XTX` com 18 commits ficam em `backlog` |
| 2026-11-26 | 21:00 | 4 | 130 commits documentados (`Xbox2Windows` 40 de 72, `VIctory-7900XTX` 40 de 85, `TuxBox` 34 de 34, `SixNG` 16 de 16) e 37 PRs abertos documentados de 67 detectados + 75 mesclados de 75. Quatro repositórios retirados do backlog; seis com delta só de PRs ficam em `backlog` |
| 2026-11-26 | 20:00 | 4 | 104 commits (40 em `VIctory-7900XTX`, 40 em `TuxBox`, 22 em `victory-fork`, 2 em `VItoriaBr`) e 72 PRs abertos documentados de 161 detectados + 62 mesclados documentados de 62, nos quatro repositórios retirados do backlog. Sete repositórios com delta ficaram em `backlog` |
| 2026-11-26 | 19:14 | verificacao manual do tracker | Kelvaroq entra no topo; Belobug medido a 22 fps contra 5-7 no proprio README; AvatarManz consolida victory-fork + DexNom + Belobug |
| 2026-11-25 | 22:00 | — | execução de baseline: cursores gravados em `state/watch.json`, nenhum documento escrito |
| 2026-11-25 | 23:00 | 10 | 48 commits e 96 PRs (89 abertos, 7 mesclados) em `VIctory`, `xcl`, `Xbox2Windows`, `VIctory-Belobug-AMD`, `VIctory-DexNom`, `VItoriaBr`, `VIctory-7900XTX`, `victory-fork`, `TuxBox`, `SixNG`. Três repos sem delta: `OpenScarlett`, `xbox-emulator`, `GTA6-Asset-Info` |
| 2026-11-26 | 00:00 | 10 | 60 commits documentados de 63 na faixa (3 ficaram no backlog por teto de 40 em `Xbox2Windows`) e 101 PRs (75 abertos, 26 mesclados) nos mesmos dez repositórios. `VIctory` e `xcl` sem commit novo, com 24 e 9 PRs abertos. Sem delta em `OpenScarlett`, `xbox-emulator` e `GTA6-Asset-Info` |
| 2026-11-26 | 01:00 | 9 | 49 commits (todos documentados, nenhum corte de teto) e 70 PRs abertos + 20 mesclados detectados no cursor, em `Xbox2Windows` (31 commits, 18 mesclados), `victory-fork` (9, 1), `SixNG` (4, 1), `VIctory-7900XTX` (3, 0), `TuxBox` (2, 0) e, sem commit, `VIctory` (28 abertos), `xcl` (11), `VIctory-DexNom` (4), `VIctory-Belobug-AMD` (2). Backlog da hora anterior (`Xbox2Windows`) consumido. Sem delta em `VItoriaBr`, `OpenScarlett`, `xbox-emulator` e `GTA6-Asset-Info` |
| 2026-11-26 | 19:00 | 4 | 120 commits de 200 na janela (teto de 40 cortou 112→40 em `Xbox2Windows` e 48→40 em `SixNG`) e 49 PRs abertos de 257 detectados + 78 mesclados de 158, em `Xbox2Windows` (40 commits), `xcl` (13), `OpenScarlett` (27) e `SixNG` (40). Oito repositórios com delta ficaram em `backlog` pelo limite de 4 por execução |

O primeiro ciclo não documenta nada: grava os cursores em `state/watch.json` para que o ciclo
seguinte comece a contar a partir dali.

Documentos de 2026-11-27 21:00: [`2026-11-27/21/`](2026-11-27/21/) - `Xbox2Windows.md`, `TuxBox.md`,
`SixNG.md`, `victory-fork.md`, `_triagem.md` e `_falhas.md`. Os 14 foram triados; 12 puderam ser
medidos e 2 nao: `Belobug/VIctory-Belobug-AMD` e `avatarmanz/VIctory-7900XTX` passaram a devolver 404
na API e na web durante a hora (a conta inteira, nao o repositorio), e por isso a triagem dessas duas
linhas e NAO VERIFICADO, o cursor nao andou e o caso esta em `_falhas.md`. O desempate foi o criterio 2
de novo: os quatro repositorios que tiveram commit na janela ocuparam as quatro vagas, e nenhum commit
casou com defeito do catalogo (criterio 1 vazio). Achado principal da hora, no `victory-fork`: a chave
do cache de shader ganhou `VICTORY_THREAD_TICKET`, mas os outros tres switches novos que mudam o codigo
gerado (`VICTORY_MEMORY_NEAR`, `VICTORY_LDS_START`, `VICTORY_WAVE_LOOP_SKIPS`) nao entraram nela, e o
comentario de `memory_near` afirma que entrou - e a mesma classe de D-02, que o acervo registra como
corrigida em `e9765a7`. No `TuxBox`, o helper DXC escreve a mao a propriedade 11 do entry point para
devolver o `[WaveSize(N)]` que o primeiro estagio do compilador perde: ganho medido de 51 s contra 60 s,
risco novo de largura de wave errada se a insercao falhar de um jeito que nao lance excecao. No
`Xbox2Windows`, quatro instrucoes que nao tinham lowering no caminho direto de compute passam a ter -
uma delas, `v_dot8_i32_i4`, ja estava catalogada como defeito no `VItoriaBr` (D-15).

Documentos de 2026-11-27 17:00: [`2026-11-27/17/`](2026-11-27/17/) — `Xbox2Windows.md`, `TuxBox.md`,
`SixNG.md`, `VIctory-DexNom.md` e `_triagem.md`. Primeira execução no desenho triagem-primeiro: os 14
repositórios foram triados e 4 documentados. O `Xbox2Windows` herdou de novo o corte de teto — 47 commits
novos acima de `f5204690`, documentados os 40 mais antigos, com `commit_cursor` em `801f4c3b` e
`head_at_run` em `e1ea515d`; os 7 restantes ficam para a próxima hora. O `compare` devolveu 79 porque 32
commits da linha `cpp-migration` (`40365770` a `999e17cd`), já documentados às 16:00, não são ancestrais do
cursor e reaparecem até ele passar pelo merge `91cbedd6`; não foram redocumentados. Achados: `f6cdee10`
(#3020) prova que o contrato de #2964 documentado às 16:00 estava errado — as formas não-SDWA de
`v_add_f16`/`v_sub_f16`/`v_subrev_f16`/`v_mul_f16` zeravam `D[31:16]` e agora o preservam; `1c879506`
(#3040) corrige dois erros de `0d228580` (#2922) publicados na mesma janela (base de 40 bits do descritor
BVH tratada como endereço de byte, e slots de resultado compactados com a ordenação desligada), ambos
candidatos a linha nova no catálogo; `98007de5` (#2899) atualiza a documentação do ciclo de `.fail` que
D-19 acusava obsoleta; `f17a08fd` (#3024) deixa de tratar `UINT64_MAX` como conclusão no sampler de
assinatura (área de D-01). No `TuxBox`, 14 commits, dos quais 12 entraram agora pelo merge `497c561a`
(#272) com datas de 26/11 20:53-23:40: a frente de verificação de mesh passa a exigir recibo de worker
para os binários de mesh e pixel carregados e proíbe tradução sem root e stand-in; `6ca0d4ac` (#277)
junta esperas redundantes da mesma timeline no `submit` de textura posicionada; `18736af5` traduz de novo,
com a wave junta, os programas de 32 lanes que leem lane em laço divergente. No `SixNG`, o único commit é o
merge `2d9c230d` de #474, que remove o base vertex do draw da máscara de blur: desde #470 (`790a3c67`, o
próprio cursor das 07:00) os índices endereçam o buffer inteiro e o deslocamento do mesh era somado duas
vezes. O `DexNom` não teve commit nem merge; seus 26 PRs abertos foram documentados com a proposta
declarada pelo autor e a ressalva de que o diff não foi lido. Desvio de procedimento: a "tabela de pistas"
que o passo 2 do procedimento cita não existe em `ANALISE_HORARIA.md`; as áreas foram tomadas das seções D
de `catalogo/defeitos.md`, e o desvio está em `state/last-run.md`. Continuam sem vez, com cursor parado:
`Belobug/VIctory-Belobug-AMD` (50 na janela, piso #551), `VIctory-7900XTX` (37), `victory-fork` (10),
`xbox-emulator` (1), `xcl` (4), `VIctory` (3), `OpenScarlett` (`#60`) e `SixNG` (`#472`).

Documentos de 2026-11-27 16:00: [`2026-11-27/16/`](2026-11-27/16/) — `Xbox2Windows.md`, `xcl.md`,
`TuxBox.md`, `VIctory.md`. `TuxBox` consumiu a faixa que estava no teto em 15:00 (19 commits por `compare`
desde `90df685a`, todos documentados) e o `Xbox2Windows` herdou o corte: 80 commits desde `e62e3a5e`,
documentados os 40 mais antigos, com `commit_cursor` em `f52046900a` e `head_at_run` em `801f4c3b`.
Três métodos de coleta mudaram nesta hora, todos por medição: (1) a listagem de mesclados
`gh pr list --state merged --search "merged:>..."` devolveu **zero** no `ivall/xcl` (repositório
transferido) e 49 dos 50 merges do `Xbox2Windows` (perdeu o #2882, merge `db28a796`), então a coleta passou
a usar a API REST `pulls?state=closed&sort=updated`; (2) a contagem de commits segue em `compare`/`ahead_by`,
nunca na listagem `commits?sha=`; (3) os timestamps do GitHub ficaram 3 h atrás do UTC real — o #3055 do
`Xbox2Windows` traz `mergedAt 2026-11-27T13:06:51Z` e foi visto às 16:06Z —, e por isso os quatro cursores
de mesclados que a execução de 15:00 gravou no horário da execução (`OpenScarlett`, `VItoriaBr`,
`victory-fork`, `GTA6-Asset-Info`, todos em `16:05:00Z`) foram reconferidos contra `2026-11-26T00:00Z`
(zero merges novos) e normalizados para `13:05:00Z`. Achados: no `Xbox2Windows`, `7beb331d` mostra que o
`compile_dxil` recusava **todo** pedido do runtime por comparar a revisão `-14` com a `-28` publicada, e
`7f951fd3` remove de `cpp-migration` os 2.544 linhas de XMem de #2911; no `TuxBox`, `138f3bbf` passa a
documentar que o jogo chega ao prólogo e cai, e que o `XboxBvhBuilder.dll` do pacote não compila no
vkd3d-proton (#307); no `xcl`, `f78e05bc` explica que uma janela de 512 MB deixava um trecho livre entre o
seu tamanho e 1 GiB, e o driver mapeava o heap ali. Nenhuma falha de coleta; nenhum `_falhas.md`; nenhum 403.

Documentos de 2026-11-27 15:00: [`2026-11-27/15/`](2026-11-27/15/) — `TuxBox.md`, `SixNG.md`,
`victory-fork.md`, `VItoriaBr.md`. O `TuxBox` é o único com corte de teto: a faixa desde `9168b2fa` tem 47
commits por `compare` e o teto é 40, então os 7 mais novos (`edf9e0518c`, `6ed257873d`, `c7c84de79e`,
`5995ed341c`, `3894902a29`, `cf47e6874c`, `07db52f84f` — o merge do #306) ficam com `last_commit` e
`commit_cursor` em `90df685ad5` e voltam como delta na próxima hora. `TuxBox` e `SixNG` consumiram o
backlog; `victory-fork` saiu dele com o único commit da janela mais os 35 PRs abertos acima do cursor
`1427`. Método de contagem corrigido nesta hora: a listagem `gh api commits?sha=<branch>&per_page=100`
ordena por data de commit e subconta a faixa quando há commits de pré-merge mais antigos que o cursor (no
`TuxBox` dava 38 em vez de 47); a fonte correta é `compare/<last_commit>...<head>` (`ahead_by`). Filtro de
mesclados por `mergedAt > last_run` (estrito): usar `>=` traria de volta lotes já documentados que
compartilham o `mergedAt` do cursor — 25 no `VIctory-7900XTX` (todos `2026-11-26T08:17:50Z`), 17 no
`VIctory-Belobug-AMD` (todos `22:30:23Z`), 1 no `OpenScarlett` (#50, `07:52:24Z`), 1 no `victory-fork`
(#1312) e 1 no `TuxBox` (#294). Achados: `victory-fork` `9a9091d1` fecha o travamento do construtor de BVH
e, pelo próprio README, deixa o frame em 0,2–3 fps com o builder no padrão (candidato a linha nova),
enquanto o PR #1466 já trata a fome do pull-back que o próprio commit introduziu; `TuxBox` `40eb1a96d0`
mede 1,26× de ganho de compilação offline (3,9× em `ms_6_6`, 1,05× em `cs_6_6`) e mantém **D-02/D-03**
fechados por `e9c8fb6`; `TuxBox` `4c0ac0edf6` (adjacente a **D-22**) troca leitura de lixo por `write_bytes`
por alocação de arena; `SixNG` sobe `BAKE_RULE` de 2 a 6 na mesma hora e renomeia o cache de livery
`f3` → `f4`. Nenhuma falha de coleta; nenhum `_falhas.md`; nenhum 403.

Documentos de 2026-11-26 23:00: [`2026-11-26/23/`](2026-11-26/23/) — `VIctory-DexNom.md`,
`VIctory-7900XTX.md`, `victory-fork.md`, `TuxBox.md`. A faixa de `VIctory-7900XTX` desde `a9400a6d` foi
inteira documentada (25 commits, o cursor `commit_cursor` das 22:00 consumido) e `TuxBox` e `victory-fork`
saíram do backlog. `victory-fork` teve sete PRs mesclados em 22 segundos (`23:06:25` a `23:06:47Z`), cada
um com seu commit de merge; `TuxBox`, doze merges. Dos 98 PRs abertos detectados nos quatro, 40 foram
documentados (20 de 50 no `VIctory-DexNom`, 20 de 28 no `victory-fork`, os 18 do `VIctory-7900XTX` e os 2
do `TuxBox`); os 58 restantes ficam com `last_pr_number` no maior documentado. Cruzamentos que mudam o
catálogo: `victory-fork` **D-12** passa a ter correção com hash (`0a40a00a` e `b1d777a8`, cargas raw
pedindo só as dwords lidas e guardadas no fim da view); `victory-fork` **D-05** tem variante coberta por
PR aberto (#1391); `AvatarManz` **D-17** aparece como PR aberto (#393). Candidatos a linha nova:
`e9428969` (filtro de sondas magenta na AMD esconde dado inválido), `VIctory-7900XTX #104` (crash
`+0x5149aec` em `--no-rt` com placa de 8 GB), `VIctory-DexNom #106`/`#107` (waits de listas liberadas,
backlinks de contador UAV), `victory-fork #1389` (root constants vazios → `0xc0000005`), `TuxBox`
`ab644f02` (execução sem memória gráfica termina em fault de GPU) e `TuxBox e9134b07` (escrita em página
fora do write-watch passa a depender de consulta de `VirtualQuery` que pode envelhecer).

Documentos de 2026-11-26 22:00: [`2026-11-26/22/`](2026-11-26/22/) — `Xbox2Windows.md`,
`VIctory-7900XTX.md`, `TuxBox.md`, `SixNG.md`. Nenhum commit novo entrou em `main` do `Xbox2Windows`
desde 21:00; a faixa documentada (32) é o resto que o teto das 21:00 deixou, com `commit_cursor` consumido.
Em `VIctory-7900XTX` o teto de 40 cortou 58 → 40 e os 18 restantes ficam com `commit_cursor` em
`a9400a6d`. `TuxBox` teve 13 PRs e 31 commits (14 merge commits para 13 PRs: #268 foi mesclado duas
vezes, `6466bcf9` e `e76ff6a2`) e `SixNG` mesclou 6 PRs. Os seis PRs mesclados do `Xbox2Windows`
(`#2914`, `#2918`, `#2917`, `#2931`, `#2928`, `#2932`) têm `base.ref = cpp-migration` e foram registrados
sem contar como mudança de `main`; o par `#2918`/`#2931` repete o padrão do **D-29** (merge e revert no
mesmo dia). Achados para o catálogo: `9ba5661f` do `AvatarManz` refaz a correção de replay que o **D-10**
descreve como revertida; `#266`/`#268` do `TuxBox` cobrem casos de mesh que a correção do **D-21**
(`315b287`) não pegava; `be41e588` do `SixNG` corrige defeito de camada de roupa sem entrada no catálogo.
Tetos aplicados: 40 commits por repositório e 20 PRs abertos por repositório (`Xbox2Windows` 50 → 20,
cursor de abertos em `2887`). Nenhuma falha de coleta; nenhum `_falhas.md`; nenhum 403.

Documentos de 2026-11-26 21:00: [`2026-11-26/21/`](2026-11-26/21/) — `Xbox2Windows.md`,
`VIctory-7900XTX.md`, `TuxBox.md`, `SixNG.md`. Os quatro repositórios com commit pendente foram processados
antes dos que só têm PR aberto; o teto de 4 por execução deixou `Marvisak/VIctory`, `ivall/xcl`,
`Belobug/VIctory-Belobug-AMD`, `DexNom/VIctory`, `omonkeyz/xbox-emulator` e `fxceless/victory-fork` no
`backlog`. Sem ferramenta de subagente no ambiente, processados em sequência. Tetos: 40 commits por
repositório (cortou `Xbox2Windows` 72→40 e `VIctory-7900XTX` 85→40, com `commit_cursor` em `1983228b` e
`f03bfa51`) e 20 PRs abertos por repositório (`Xbox2Windows` 50→20). Divergência registrada: o cursor de
`TuxBox` deixado às 20:00 (`be9055c9`) estava atrás de 22 commits que aquela execução já documentara; a
faixa de 56 foi deduplicada contra os documentos anteriores e só 34 foram escritos. Nenhuma falha de
coleta; nenhum `_falhas.md`; nenhum 403.

Documentos de 2026-11-26 20:00: [`2026-11-26/20/`](2026-11-26/20/) — `VIctory-7900XTX.md`, `TuxBox.md`,
`victory-fork.md`, `VItoriaBr.md`. Os quatro saíram do `backlog` na ordem em que já estavam registrados. Sem
ferramenta de subagente no ambiente, foram processados em sequência. Tetos aplicados: 40 commits por
repositório (cortou `VIctory-7900XTX` 147→40 e `TuxBox` 84→40, com `commit_cursor` gravado em `306722bc` e
`be9055c9`) e 20 PRs abertos por repositório (`VIctory-7900XTX` 60→20, `TuxBox` 29→20, `victory-fork` 60→20,
`VItoriaBr` 12→12). O cursor de mesclados avançou até o `mergedAt` do último merge documentado por
repositório (08:17:50Z em `VIctory-7900XTX`, 19:34:00Z em `TuxBox`, 18:05:17Z em `victory-fork`), não até o
horário da execução. Nenhuma falha de coleta; nenhum `_falhas.md`. Achados que valem revisão do catálogo:
`VICTORY_SHOW_NEW_LATER` (`4cd2d133`+`91018a5d`+`cf9ba084`) ataca o mecanismo do **D-09** do `AvatarManz`,
que segue com `corrigido` vazio; `5cfe32df`+`d792ebd4` fecham só a metade "nome documentado que o código não
lê" do **D-19**; `c6ce497b` do `TuxBox` derruba o teto do cache NVIDIA de 32 GiB para 4 GiB, porque o driver
parou de indexar com ~5 GB — o **D-04** estava corrigido com um teto acima do ponto de quebra.

Documentos de 2026-11-26 19:00: [`2026-11-26/19/`](2026-11-26/19/) — `Xbox2Windows.md`, `xcl.md`,
`OpenScarlett.md`, `SixNG.md`. Execução atrasada: a tentativa do slot 02:00 foi interrompida antes de gravar
qualquer coisa e o estado não avançou, então esta janela cobre 01:15→19:35 local (~18 h) e não uma hora. Sem
ferramenta de subagente disponível, os quatro repositórios foram processados em sequência; os outros oito com
delta (`VIctory`, `VIctory-Belobug-AMD`, `VIctory-DexNom`, `VItoriaBr`, `VIctory-7900XTX`, `victory-fork`,
`xbox-emulator`, `TuxBox`) ficaram em `backlog`. Tetos aplicados e registrados: 40 commits por repositório
(`Xbox2Windows` 112→40, `SixNG` 48→40, `commit_cursor` gravado) e 20 PRs abertos por repositório
(`Xbox2Windows` 63→20, `xcl` 185→20). O cursor de PRs mesclados avançou até o `mergedAt` do último merge
documentado (06:27:26Z em `Xbox2Windows`, 18:31:19Z em `SixNG`), não até o horário da execução, para não pular
merges não documentados. Nenhuma falha de coleta; nenhum `_falhas.md`.

Documentos de 2026-11-26 01:00: [`2026-11-26/01/`](2026-11-26/01/) — `Xbox2Windows.md`,
`victory-fork.md`, `SixNG.md`, `VIctory-7900XTX.md`, `TuxBox.md`, `VIctory.md`, `xcl.md`,
`VIctory-DexNom.md`, `VIctory-Belobug-AMD.md`. Nenhuma falha de coleta. Duas notas de método:
(i) a busca `gh pr list --state merged --search "merged:>=<cursor>"` devolveu lista vazia nos treze
repositórios (índice de busca atrasado) e os mesclados foram lidos por
`gh api .../pulls?state=closed&sort=updated` filtrando `merged_at`, o que recuperou 20 merges;
(ii) 16 merges adicionais (`#333`, `#338`, `#345` em `SixNG` e 13 em `Xbox2Windows`) têm `mergedAt`
anterior ao cursor de `last_run` mas commit posterior ao `last_commit`, e foram documentados como commits
com a divergência registrada em Confiança. Candidatos a linha nova no catálogo: `AvatarManz #97`
(UAV de textura posicionada escrevendo em stand-in, interiores sem luz indireta), `DexNom #66` (slot de
heap lido como CBV, fault de GPU na RDNA 4), `victory-fork #1258` / `DexNom #68` / `AvatarManz #96`
(exaustão do anel de slots dinâmicos com teto de dispositivo), `SixNG #346` (tabela `EDUP` dos pacotes de
conteúdo não lida) e `Xbox2Windows #2679`/`#2685`/`#2709` (regressão de descritor vivo, custo de
`VirtualQuery`, lock de save preso).

Documentos de 2026-11-26 00:00: [`2026-11-26/00/`](2026-11-26/00/) — `VIctory.md`, `xcl.md`,
`Xbox2Windows.md`, `VIctory-Belobug-AMD.md`, `VIctory-DexNom.md`, `VItoriaBr.md`,
`VIctory-7900XTX.md`, `victory-fork.md`, `TuxBox.md`, `SixNG.md`. Nenhuma falha de coleta.
Desvio registrado: PRs mesclados passaram a ser detectados por `mergedAt` (não por
`last_pr_number`), porque a numeração de PR não acompanha a ordem de merge nos repositórios
monitorados; o teto de 40 commits deixou `404446e8`, `41ad992e` e `2a801192` de `Xbox2Windows`
para a próxima execução.

Documentos de 2026-11-25 23:00: [`2026-11-25/23/`](2026-11-25/23/) — `VIctory.md`, `xcl.md`,
`Xbox2Windows.md`, `VIctory-Belobug-AMD.md`, `VIctory-DexNom.md`, `VItoriaBr.md`,
`VIctory-7900XTX.md`, `victory-fork.md`, `TuxBox.md`, `SixNG.md`. Nenhuma falha de coleta na hora.
