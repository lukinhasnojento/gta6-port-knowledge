# ivall/xcl — atualização de 2026-11-26 19:00

## Resumo

**13 commits** desde `b49be183`, nove deles de conteúdo e quatro merges de PR. O tema é preparação de shaders
antes da entrada no título (janela de progresso no mesmo HWND do swap chain) e persistência de blobs de PSO
"two-pass" (mesh/geometry) no cache de driver, com tolerância a drivers que não exportam blob. Entram também
GameInput com histórico de leituras e duas correções de identidade de stream de descritores. A fila de PRs
abertos do repositório é enorme (**500** listados, mínimo #2128) e **185** têm número acima do cursor 2638;
documentamos os 20 mais antigos.

## Commits novos (13)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| 7d9e8111 | bmjask | xcl run: prepare supported shader records before starting the title | Novo `crates/xcl/src/launcher/shader_window.rs` (196 linhas) abre uma janela Win32 própria com barra de progresso e chama `shaders::prepare_for_launch` antes de criar o guest; `shaders.rs` ganha `LaunchPreparation` (contadores de registros suportados/recusados) e o launcher passa a aguardar a preparação. |
| defe7ef1 | bmjask | D3D12.X: show shader preparation in the presentation window | Move a preparação para `src/hle/d3d12x/present/preparation.rs` (230 linhas), que desenha o progresso no **mesmo HWND** que depois é dono do swap chain DXGI, e remove `launcher/shader_window.rs`; `present.rs` chama a tela antes do primeiro Present. |
| 5bec5877 | bmjask | Scripts: allow shader preparation time before the smoke guest timer | `scripts/smoke.ps1` ganha `-StartupTimeoutSeconds` (padrão 120, faixa 1..86400), somado à espera antes de iniciar o timer do guest, para que a preparação de shaders não seja contada como lentidão do título. |
| d61ffdc4 | Bloxive | D3D12.X: canonicalize inactive two-pass stream descriptors | `two_pass_stream_key` deixa de recusar o stream inteiro quando os sub-objetos 7 (`STREAM_OUTPUT`), 12 (`INPUT_LAYOUT`) e 22 (`VIEW_INSTANCING`) contêm ponteiros inertes/padding: lê a estrutura e só recusa se `NumEntries`/`NumStrides`/`RasterizedStream`, `NumElements` ou o descritor de view instancing forem não vazios. |
| f9e6ac5f | Bloxive | GameInput: retain recent gamepad readings between polls | `src/hle/gameinput.rs` passa de leitura "no momento do pedido" (250 Hz por callback) para **um poller** de ~4 ms que alimenta um histórico de 0,5 s; `GetCurrentReading`/callbacks servem do histórico. Também reescreve o cabeçalho do módulo para documentar o modelo. |
| bdef4480 | Bloxive | D3D12.X: persist both PSOs for translated mesh and geometry | `driver_cache` ganha `keep`/`KEEP_FIXTURES` e passa a guardar os **dois** PSOs do caminho two-pass (mesh e geometry) em vez de só um; `real.rs` reestrutura `driver_compile` em `driver_compile_identified` (hash `fnv64` do container, tempo só no driver) e `gpu/xbox.rs` perde 101 linhas de caminho antigo. Novo `tests/gpu_two_pass_cache_e2e.rs` (565 linhas). |
| 7ca01644 | Bloxive | D3D12.X: preserve cold mesh cache retention and retry ownership | Extrai `worth_keeping(took)` e usa em `keep()`; corrige a posse no retry de `create_mesh` (não zera mais `*out` antes de tentar) para que um blob frio de mesh continue elegível e o PSO adotado não perca o dono. |
| f9c9df37 | Bloxive | D3D12.X: tolerate drivers that cannot export two-pass blobs | `read_pso_cached_blob` unifica os dois caminhos (`is_adopted` via `GetCachedBlob` do proxy e `ID3D12PipelineState::GetCachedBlob`) e devolve `None` quando o driver falha, em vez de propagar erro; o teste two-pass cobre driver que não exporta blob. |
| 76631abb | Prime1997 | Merge pull request #2448 from ZekkJJ/codex/two-pass-empty-descriptors | Merge do PR #2448; o conteúdo é exatamente `d61ffdc4` (mesmos 64+/2− em `two_pass_identity.rs`). |
| 8d5fde3c | Prime1997 | Merge pull request #565 from ZekkJJ/codex/gameinput-reading-history | Merge do PR #565; conteúdo igual a `f9e6ac5f` (407+/73− em `gameinput.rs` e uma linha de README). |
| e120925e | Prime1997 | Merge pull request #1942 from ZekkJJ/upstream/two-pass-driver-cache | Merge do PR #1942; agrega `bdef4480` + `7ca01644` + `f9c9df37` (935+/256− em seis arquivos). |
| 813302f3 | bmjask | xcl run: merge main and retain both shader preparation notes | Merge de `main` no ramo de trabalho: 2 597+/400− em 22 arquivos. Traz o trabalho que estava em `main` (compactação DXIL, `recomp/dxc.rs` +229, `pso/offline_cold_profile.rs` novo com 296 linhas, `link_tests` +242, `dxil/module/split.rs` +73) e resolve os dois textos de preparação de shader; a autoria das partes herdadas é de `main`, não deste ramo. |
| 8b957bec | Prime1997 | Merge pull request #2555 from bmjask/shaders/startup-preparation | Merge do PR #2555; agrega `7d9e8111`+`defe7ef1`+`5bec5877` (439+/45− em 11 arquivos, incluindo `present/preparation.rs`). |

## Pull requests abertos (185 novos; 20 documentados)

`gh pr list --state open --limit 500` devolveu **500** PRs abertos (mínimo #2128, máximo #2860); **185** têm
número acima de 2638. Foram documentados os 20 de menor número para manter o cursor monotônico; os 165 restantes
ficam para a próxima execução. Títulos e propostas vêm do corpo do PR.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 2640 | Recompiler: prune unused module constants and dead-aggregate globals in direct DXIL bitcode | Worklist de liveness (`live_values`) sobre blocos alcançáveis e metadados para descartar constantes e globais de agregado mortos na serialização direta de bitcode DXIL. | Não. |
| 2641 | Recompiler: simplify direct DXIL frexp_mant, frexp_exp, and alignbit helpers | Substitui intrínsecos redundantes e round-trips int/float nos helpers `frexp_mant`/`frexp_exp`/`alignbit` do prelúdio DXIL direto. | Não. |
| 2642 | Recompiler: eliminate BTree/Vec allocations and skip dominator pass on acyclic CFGs in direct DXIL cfg | Troca `BTreeMap`/`BTreeSet` por `binary_search_by_key` sobre PCs monotônicos e pula o passe de dominadores em CFG acíclico. | Não. |
| 2643 | DXC: unload a candidate folder that is not a usable dxcompiler (#2413) | `load()` passa a `FreeLibrary` da pasta candidata quando `dxcompiler.dll` não carrega ou não exporta `DxcCreateInstance`, para não deixar `dxil.dll` obsoleto mapeado. | Não; corrige #2413. |
| 2644 | Scripts: match smoke report headers on their own lines | Exige que o cabeçalho `xcl-smoke v1` ocupe a linha inteira, para que um `-Title` contendo esse texto não seja lido como próximo relatório. | Não. |
| 2645 | Scripts: replace coverage baselines atomically | Grava o baseline de cobertura em arquivo temporário irmão e substitui o destino só depois do sucesso, em vez de truncar o existente. | Não. |
| 2646 | Scripts: let local PR checks use an upstream main ref | Adiciona `--main-ref` ao verificador local (padrão `origin/main`) para checar contra `upstream/main` num fork com `origin` velho. | Não. |
| 2649 | D3D12.X: drop global FENCES lock across GetCompletedValue and scope CPU-signal refreshes | Tira o `Mutex` global de `FENCES` de volta das chamadas `GetCompletedValue()` e limita o refresh de sinal de CPU à fence sinalizada. | Não. |
| 2650 | Scripts: decode coverage sweeps as UTF-8 | Passa `encoding="utf-8"` ao subprocesso do sweep, para nome de shader com byte inválido no code page local não gerar `UnicodeDecodeError`. | Não. |
| 2651 | Scripts: allow smoke artifacts in source log directory | Normaliza `-OutDir` e não copia `xcl.log`/`report.json` sobre si mesmos quando o destino é o diretório de log do título. | Não. |
| 2652 | Recompiler: serialize dxc::wait_for_memory slot admission (#2463) | Serializa a admissão de slot em `dxc::wait_for_memory`, onde workers liam `WORKING` sem sincronização antes de `fetch_add(1)` e furavam `WANTED_AT_ONCE`. | Não; corrige corrida TOCTOU. |
| 2653 | Scripts: keep Unicode separators inside coverage JSON lines | Divide o relatório só em `\n` físico, para que NEL/separador de parágrafo dentro de nome de shader não parta um registro JSON válido. | Não. |
| 2654 | Scripts: mark untracked crate inputs in smoke reports | Inclui arquivos não rastreados e não ignorados sob `crates` no teste de "sujo" do bloco de smoke, porque o build script do DXIL inclui `.rs` recursivamente. | Não. |
| 2655 | Scripts: print unbounded BVH ranges in ASCII | Imprime `unbounded` em vez do símbolo de infinito quando o descritor de BVH não tem limite, evitando `UnicodeEncodeError` em console CP1252. | Não. |
| 2656 | Recompiler: stream gpu::effects without Vec<Decoded>/HashSet and avoid copying compute blobs | Analisa `gpu/effects.rs` em passada única sobre `&code[off..]`, sem `Vec<Decoded>`/`HashSet<u64>` e sem copiar blob de compute; early-exit para shader que só escreve memória. | Não. |
| 2657 | README: document xcl shaders --direct --hash and --dump | Uma linha de exemplo no README para `--direct --hash/--dump/--compare`, conferida contra a ajuda do `main.rs` em `b49be18`. | Não; só documentação. |
| 2658 | Scripts: reject GDK vtable slots outside u8 | `gen_gdk_table.py` valida o slot antes de abrir a saída e recusa >255, em vez de gravar valor que não cabe no campo `u8`. | Não. |
| 2659 | Scripts: read BOM-marked record-writer listings | Reconhece BOM UTF-16 (listing salvo por redirecionamento do PowerShell 5.1) e mantém UTF-8 sem BOM, em vez de achar zero sites de escrita. | Não. |
| 2660 | Scripts: follow reachable record-writer fallback calls | A varredura de fallback segue só instruções alcançáveis a partir do alvo, em vez de pegar a primeira chamada indireta nas próximas 12 linhas. | Não. |
| 2661 | Scripts: parse C integer spellings in tiling masks | `gen_tiling.py` aceita octal (`010`) e hexadecimal em qualquer caixa (`0X10`) nos campos de máscara de swizzle. | Não. |

## Pull requests mesclados (0)

`gh pr list --state merged --search "merged:>=2026-11-26T04:15:00Z"` devolveu lista vazia, e a varredura
`gh api repos/ivall/xcl/pulls?state=closed&sort=updated&per_page=50` não trouxe nenhum `merged_at` posterior ao
cursor. Os quatro merges da faixa (`#2448`, `#565`, `#1942`, `#2555`) têm `mergedAt` anterior a
`2026-11-26T04:15:00Z`; aparecem como **commits de merge** documentados acima, não como PR mesclado novo.

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum dos 13 hashes consta como `corrigido` de um defeito de `XCL` (D-01…D-24 têm hashes
  dos dossiês). Duas proximidades:
  - `d61ffdc4`/`76631abb` mexem na chave do stream two-pass; **D-11** (entradas de cache per-lane sem as fontes
    do lowering) é outro cache (DXIL por lane, não blob de driver) e não é tocado.
  - `bdef4480`/`7ca01644` ampliam o cache de blob de driver; o commit não registra versão de tradutor na chave
    do blob, então a pergunta de invalidação entre revisões continua em aberto. O Xbox2Windows abriu o PR
    #2820 exatamente para avisar sobre cache e tradutor de revisões diferentes — problema análogo, outro
    repositório.
- **Abre defeito novo?** Dois candidatos, **NAO VERIFICADO**:
  1. `f9e6ac5f` — `GetCurrentReading` deixa de amostrar no instante da chamada e passa a servir um histórico de
     0,5 s alimentado por um poller de ~4 ms. Um título que leia o estado "agora" (por exemplo, para detectar
     soltura de botão) pode receber uma amostra de até um período de poll atrás. Não há caso de título nem
     teste de latência no diff.
  2. `813302f3` — o merge de `main` traz 2 597 linhas de trabalho herdado (compactação DXIL, perfil frio
     offline, split de módulo) para dentro do ramo de preparação de shaders. O merge foi feito pelo autor do
     ramo; se alguma dessas partes exigia ordem de aplicação, o histórico não mostra um teste de integração
     rodado nesse commit.
- **Mantido:** os quatro merges não introduzem conteúdo próprio; o conteúdo é o dos commits que agregam.

## Confiança

- **Verificado por diff:** os 13 hashes foram lidos em `gh api repos/ivall/xcl/commits/<sha>`. Nos merges,
  comparei a lista de arquivos e as estatísticas com as dos commits agregados: `76631abb` e `d61ffdc4` têm a
  mesma lista (1 arquivo, +64/−2) e `8d5fde3c` e `f9e6ac5f` também (2 arquivos, +408/−73).
- **Cobertura:** os 13 commits da faixa foram todos documentados (abaixo do teto de 40). `last_commit` vai para
  `8b957bec6f82495aec940944c6bf47faa02db590`, head no momento da coleta.
- **PRs abertos:** 185 novos; 20 documentados, `last_pr_number` → 2661. A lista de abertos tem 500 entradas no
  limite da consulta; pode haver mais de 500 abertos e o universo real de "novos" ser maior que 185.
- **NAO VERIFICADO:** os dois candidatos a defeito novo; se `813302f3` foi testado em Windows (o repositório é
  Windows-only e o commit não traz resultado de CI); o efeito do poller de GameInput em títulos reais.
