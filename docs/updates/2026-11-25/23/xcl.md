# ivall/xcl — atualização de 2026-11-25 23:00

## Resumo

Nenhum commit novo: `b49be1836844fe0b0cdb3211dce3cb7fb4e21ac7` continua no topo da `main`. O delta são
11 PRs abertos, quase todos fechando lacunas de *liveness* de máscara no recompilador split-wave64 e
no emissor compacto de DXIL, além de três correções de `D3D12.X` em torno de `ExecuteIndirect`.

## Commits novos (0)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| — | — | — | nenhum commit novo desde `b49be1836844fe0b0cdb3211dce3cb7fb4e21ac7` |

## Pull requests abertos (11)

Propostas declaradas pelo autor no corpo do PR; nenhum diff foi lido nesta execução.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 2596 | README: document xcl rpf --full | Documentar `--full` (probe de header com memória limitada em arquivos > 1 GB) | Não |
| 2597 | Recompiler: mark SRC_VCCZ/SRC_EXECZ whole-live in x64_masks S_MOVREL and VOP3 mask ops | `S_MOVREL*` com `SSRC0` 251/252 e ops de máscara VOP3 passam a ser whole-live em `uses_with_table` e `uses_fallback` | A mesma família de D-16 (máscara wave64 obsoleta reportada só na metade baixa) e D-12 (branch por lane ignorando escritas de M0); não são os mesmos defeitos |
| 2598 | D3D12.X: ExecuteIndirect skips a stand-in whose build ends during the call | Corrigir corrida entre `resolve_wait` e `is_pending` na resolução do PSO | Área de D-20/D-21 (`ExecuteIndirect`), defeito diferente |
| 2603 | Tests: isolate the streamed-texture recheck fixture | Isolar chave inteira pendente publicada na mesma tabela de texturas dos testes paralelos | Não; é isolamento de teste |
| 2604 | DXIL: fix compact SGPR liveness for RMW/CMOV SALU and Wave32 masks | `inst_sreg_scc_use_def` só tratava `S_CMOVK_I32` como escrita condicional; corrigir `use`/`def` de RMW e CMOV | Vizinho de D-04 (carry-ins/`v_cndmask` lendo só o bit 0) e D-13 (SCC de máscara virando uniforme), não os mesmos |
| 2605 | Recompiler: match all VOP3 encoding names in ps_masks | `starts_with("ENC_VOP3")` perde `VOP3_SDST_ENC(_INST_LITERAL)` e `VOP3_INST_LITERAL` | Não; mesma classe de D-05 (nome de campo errado causando recusa silenciosa) |
| 2610 | D3D12.X: a DispatchX with thread dimensions launches whole groups | `DispatchX(x,y,z,flags)` recompõe grupos inteiros a partir de dimensões em threads | Não consta no catálogo XCL; nota que o TuxBox D-27 descreve o mesmo erro de contagem no slot 80, repo diferente |
| 2611 | D3D12.X: a graphics ExecuteIndirect waits for its pipeline at the submit | Adiar a espera do PSO gráfico para o submit em vez de dentro da chamada | Área de D-20/D-21; defeito diferente |
| 2612 | Scripts: reject non-RDNA2 decoder table inputs | `gen_rdna2.py` aceitava o XML do RDNA1 (1.149 vs 1.153 instruções) e mudava a ISA em silêncio | Não consta (integridade da tabela do decodificador) |
| 2613 | Shaders: warm runtime-selected graphics DXIL variants in --precompile | Rotear VS `hw == 4` e PS `hw == 5` por `console_to_dxil_in` no `--precompile` | Vizinho de D-11 (entradas de cache sem as fontes do lowering), não o mesmo |
| 2614 | D3D12.X: preserve replaced driver blobs after stale rejection | Fixes #2601: serializar publish/invalidação por entrada e apagar só se o arquivo ainda tem os bytes rejeitados | Não é D-11 (DXIL obsoleto servido do cache), mas é o mesmo tipo de erro de cache |

## Pull requests mesclados (0)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| — | — | — | nenhum PR com número > 2594 aparece como mesclado |

## Cruzamento com o catálogo

- Sem commit novo, nenhum defeito catalogado pode ter sido corrigido por hash nesta hora.
- Nenhum PR aberto declara corrigir um dos D-01..D-24 de XCL. Os mais próximos: #2597 e #2604
  (máscaras/liveness, família de D-04/D-13/D-16) e #2614 (cache obsoleto, família de D-11). Só o
  diff pode dizer — **NAO VERIFICADO**.
- Candidato a defeito novo: #2612 — `gen_rdna2.py` aceitar o XML do RDNA1 e gerar 1.149 instruções
  em vez de 1.153, sem checar `ARCH`. Não há linha no catálogo.

## Confiança

- **Verificado:** ausência de commit novo (cursor `b49be18…` no índice 0 de 100).
- **Verificado:** os 11 PRs, números, títulos e autores (API `pulls?state=open`).
- **NAO VERIFICADO:** conteúdo dos diffs dos PRs; os arquivos e linhas citados no corpo do PR não
  foram abertos.
