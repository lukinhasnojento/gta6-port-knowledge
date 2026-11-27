# brcly/TuxBox — atualização de 2026-11-27 18:00

## Resumo

Dois commits desde `76d1abe0`: o do PR #309 (`b46c3db9`) e o merge `cf0a0824`. É uma correção de
teste de três linhas, não de comportamento: o teste do fallback `-O0` do DXC contava chamadas ao
compilador e passou a depender da versão instalada do DXC — com 1.9.2609 o `compile_dxil` retenta sem
`-fnew-inlining-behavior` e o teste via três chamadas onde esperava duas. O teste agora fixa a opção
procurada, sem tocar o caminho de compilação. Os dois PRs abertos (#310 e #311) são bem maiores: um
retenta recusas de `IDXEN` na fase de preparação offline, o outro desenha o splash do pacote até o
título apresentar de forma estável.

## Commits novos (2)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `b46c3db9f5` | Haolin Xue | shaders: pin the DXC option lookup in the -O0 fallback test | `tests/test_runtime_shader_links.py` +3/−1: `test_cs_6_6_falls_back_to_no_optimisation` passa a fixar `inlining_flags` sem a opção, de modo que o teste exercite só o caminho para o qual foi escrito; a retentativa com a opção continua coberta por `test_rejected_compute_output_is_compiled_again_without_new_inlining`. O corpo do commit mede o motivo: desde `8a3b6df`, `compile_dxil` retenta um `cs_6_6` recusado sem `-fnew-inlining-behavior` quando o DXC instalado lista a opção, então a contagem de chamadas variava com a versão (3 ≠ 2 em DXC 1.9.2609, e passa em DXC que não lista a opção). |
| `cf0a0824c6` | brcly | Merge pull request #309 from tzbkk/pin-dxc-lookup | Merge `--no-ff` do PR #309 sobre `b46c3db9`; mesmo diff, 1 arquivo, +3/−1. |

## Pull requests abertos (2)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #310 | shader-prep: retry a IDXEN stride refusal with the appended stride table | A fase de preparação em tempo de launch nunca retentava a recusa "raw buffer IDXEN needs the console descriptor stride" com a tabela de strides anexada, embora o runtime já o fizesse. O PR mede 504 recusas IDXEN e leva a 130 (374 resgatadas) em `run --prepare-only`; das 130 restantes, 127 são shaders raster sem RTS0 embutido (tratados no launch pela retentativa de raster já mesclada) e 3 falham por outro motivo. +488/−6 em 8 arquivos: `project_tuxbox/runtime_shaders.py`, `scarlett_report.py`, `runtime/dxbc/src/rootsig.rs` e testes. | **D-12** (`Stride de descritor inline incompativel com o RADV (32 bytes)`) e D-24 (resolução de merge que apagou lowering): mesma tabela de strides e mesmo caminho de preparação; o PR não muda a raiz do problema, só faz a preparação usar a tabela. |
| #311 | d3d12: show the package splash until the title presents steadily | No console o `SplashScreenImage` do `ShellVisuals` aparece do launch até o título apresentar; no desktop aparecia a janela do título sem pintura e mais larga que o monitor. O projeto cria a janela para 3840×2160 e apresenta um quadro ~2 s depois e outro em até 26 s, sem bombear mensagens no intervalo, então o `SWP_ASYNCWINDOWPOS` do primeiro present esperava ~26 s. O PR ajusta a janela na criação, instala um hook de WinEvent in-context e desenha o splash até os frames ficarem estáveis. +1.110/−42 em 11 arquivos (`runtime/d3d12/src/present.rs`, `present/splash.rs`, `present/window.rs`, `frame_present.cpp`, `frame_guest.cpp`, `build.py`, `verify.py`). | Área de **D-06** (`Present unico nao chega a tela no GE-Proton11-14`) — ambos são o caminho de present/janela, mas D-06 é sobre o quadro do título e este PR é sobre a janela antes do primeiro quadro. Sem hash do catálogo. |

## Pull requests mesclados (1)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| #309 | shaders: pin the DXC option lookup in the -O0 fallback test | `cf0a0824` | Exatamente o prometido: +3/−1 em `tests/test_runtime_shader_links.py`, sem código de produção. O `merged_at` é 14:11:02Z, contra a data de commit 13:28:02Z de `b46c3db9`; o merge commit `cf0a0824` é de 14:09:02Z. |

## Cruzamento com o catálogo

- **D-02** (`Otimizador do DXC estoura o tempo; shaders viram pipeline nulo`) e **D-03** (`Conversao
  DXIL->SPIR-V leva minutos em cs_6_6 compilado em -O0`): `b46c3db9` existe por causa do caminho de
  fallback `cs_6_6`/`-O0`, mas o diff só fixa a opção no teste — não altera tempo, retentativa nem
  escolha de otimização.
- **D-12** e **D-24**: #310 toca a tabela de strides do descritor na preparação e o relatório de
  recusas; é a área exata de D-12 (stride inline de 32 bytes no RADV) e do tipo de resolução de merge
  que gerou D-24. Em aberto, sem merge.
- **D-06**: #311 é o caminho de present/janela antes do primeiro quadro; D-06 trata de um present que
  não chega à tela. Área, não correção.
- **Defeito novo**: nenhum identificado. O commit e o merge não introduzem caminho novo de produção.

## Confiança

Verificado: os dois diffs foram lidos pela API (`commits/<sha>`), e o merge #309 confere com o
`merge_commit_sha` `cf0a0824c6`; a contagem de 2 commits contra 1 merge é explicada pelo commit de
branch (`b46c3db9`) mais o merge.

Não verificado: (i) os dois PRs abertos foram lidos pelo corpo e pela lista de arquivos, não pelo
patch; os números de recusas (504 → 130) são os que o autor declara; (ii) nenhum teste foi executado
aqui; (iii) `#310` e `#311` não citam `Fixes` de issue, então a ligação com issues não foi conferida;
(iv) o corpo do commit cita `8a3b6df` como origem da retentativa — não reconferi esse hash no
histórico do repositório nesta hora.
