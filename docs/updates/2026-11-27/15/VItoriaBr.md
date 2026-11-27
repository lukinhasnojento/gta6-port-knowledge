# uiliannn/VItoriaBr — atualização de 2026-11-27 15:00

## Resumo

Um commit na `main` desde o cursor `1361e190`: a sonda de teste que decide se o adaptador é de software
passa a tratar adaptador **ausente** como software. É uma correção de CI, não de runtime — em runners
WARP o handle DXGI não é retido, `is_none_or` devolvia falso e o teste de depth bounds tentava afirmar
contagens de draw de GPU real sobre software. Nenhum PR aberto ou mesclado varia nesta hora.

## Commits novos (1)

`commits?sha=main&per_page=100`: `1361e190` está na posição 2; o topo é `15779f81`.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `15779f8197` | uiliannn | victory-gpu: treat None adapter as software in is_software_adapter | 1 arquivo, +9/-6, `crates/victory-gpu/src/host.rs` linhas ~4000-4015. A função `is_software_adapter` (sob `#[cfg(test)]`, `#[allow(dead_code)]`) troca a forma `self.adapter.as_ref().is_none_or(...)` (que fechava o adaptador com `GetDesc1()`) por um `match` explícito com `None => true` e `Some(a)` mantendo o teste anterior (`GetDesc1()`, flag `DXGI_ADAPTER_FLAG_SOFTWARE` ou `VendorId == 0x1414`). Com `Option::is_none_or` o caso `None` devolvia **falso**, então num runner sem adaptador DXGI retido o teste de depth bounds deixava de rodar no caminho de software e tentava afirmar contagens de draw de GPU real em WARP. O corpo diz manter o teste de PSO/cache rodando em WARP e os asserts de draw presos a GPU real. |

## Pull requests abertos (0)

`gh pr list --repo uiliannn/VItoriaBr --state open --limit 50` devolveu 50 PRs abertos e **nenhum** com
número acima do cursor 126 — os 50 já estavam cobertos pela execução que fixou o cursor. Nada a
documentar.

## Pull requests mesclados (0)

`search/issues` com `is:merged merged:>=2026-11-26T04:15:00Z` devolveu `total_count=0` para este
repositório. Nenhum merge na janela.

## Cruzamento com o catálogo

- **Corrige D-xx?** Não. O único candidato próximo é **D-11** ("Depth bounds test não traduzido: draws
  passavam em toda profundidade", corrigido em `c96b4f2`) — este commit mexe na **sonda de teste** que
  decide se o adaptador é software, não na tradução do depth bounds. O efeito é fazer o teste de depth
  bounds voltar a pular os asserts de draw em WARP, como pretendia; a tradução continua a de `c96b4f2`.
- **Abre defeito novo?** Não. Duas linhas de verificação:
  - A mudança está sob `#[cfg(test)]`; não entra em build de release, então não pode alterar
    comportamento de runtime.
  - O caminho `Some(a)` é byte a byte o anterior (mesma chamada `GetDesc1`, mesmas duas condições). Só o
    caso `None` muda de falso para verdadeiro — o que é o objetivo declarado.

## Confiança

Verificado: o commit em `repos/uiliannn/VItoriaBr/commits/15779f8197` (pai único `1361e190b7f46df996f00ea1b540ecf66785127f`,
+9/-6, patch completo lido); a lista de commits com o cursor na posição 2; `gh pr list --state open`
(50, nenhum acima de 126); `search/issues` de mesclados com `total_count=0`.

Não verificado: não rodei a suíte de testes nem `cargo test` neste repositório, então não confirmei que
o teste de depth bounds passa agora em WARP. A afirmação de que o runner WARP é o afetado é do corpo do
commit, não medida aqui.
