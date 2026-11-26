# Belobug/VIctory-Belobug-AMD — atualização de 2026-11-26 00:00

## Resumo

Sem commit novo: o topo da `main` continua em `4f62c2a5e27e7beebf3e8d68f77898e43dc7b77c`. Três pull
requests abertos acima de 215, todos pequenos e cirúrgicos: guarda de panic no worker de pipeline,
invalidação do plano bindless para escritas e suporte a Wave64 com LDS em computação multi-wave.

## Commits novos (0)

`gh api repos/Belobug/VIctory-Belobug-AMD/commits?sha=main&per_page=100` devolve `4f62c2a5` como
primeiro elemento, igual ao `last_commit` do estado.

## Pull requests abertos (3)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 219 | A panic while a pipeline is made refuses it, the worker goes on | Envolve a criação de pipeline do host (`pipeline/make.rs` +28/-1): captura o panic, registra shaders e mensagem quando o payload é string, marca a pipeline como recusada e acorda os waiters. Declara ser o segundo item de `#217`; o teto de espera por pipeline ficou de fora por ser mudança de comportamento. Propõe o mesmo patch upstream em `Marvisak/VIctory#2980`. | Não é D-xx. `D-06` ("Compilador AMD: pipelines que nunca terminam ou crasham, 16 GB em dois minutos") descreve sintoma vizinho, mas a causa ali é o compilador, não um panic do Rust. |
| 220 | Restore bindless plan invalidation for writes | Reverte parte de `705a2d0` (`descriptor/host.rs` +3/-7): mudança de faixa escrita e desaparecimento de slot escrito voltam a ser motivo de reconstruir o plano bindless. O bisect do `#143` aponta esse commit como origem. Validação declarada: `cargo check` para `x86_64-pc-windows-gnu` e `git diff --check`; nenhum runtime ou benchmark de GPU. | Não é D-xx deste repositório. `D-05` é exaustão de slots dinâmicos; aqui o plano bindless deixa de ser reconstruído, o que produz leitura de slot velho — comportamento parecido com `D-07` (use-after-free na GPU), causa diferente. |
| 221 | victory-shader: support multi-wave and LDS Wave64 compute shaders | Em alvo de 32 lanes, `console_cf` deixa de exigir grupo de uma wave e `lds_dwords == 0`: a troca aloca `threads.div_ceil(64) * 2` dwords de shared memory e indexa o par de troca de cada wave do console por `flat_tid >> …` (`lower.rs` +32, `emit/mod.rs` +4, `split_tests.rs` +58). Declara fechar `#149` e recuperar ~17 dispatches de compute por frame no prólogo da prisão. | Não é D-xx. `D-06` (compilador) e `D-13` (memória) são outros sintomas; `AvatarManz D-11` (helper lanes em operações de wave na AMD) é a mesma família de risco, outro repositório. |

## Pull requests mesclados (0)

Nenhum PR com `mergedAt` posterior a 2026-11-26T02:11:35Z. O maior número mesclado devolvido é 96,
abaixo de `last_pr_number` 215 — numeração de PR não acompanha ordem de merge aqui.

## Cruzamento com o catálogo

- Corrige D-xx? Nenhum dos 17 itens de `catalogo/defeitos.csv` deste repositório. Os que continuam
  abertos — `D-05` (esgotamento de slots dinâmicos), `D-06` (compilador AMD), `D-12` (crash/hang ao
  devolver tiles de cópia), `D-13` (memória), `D-14` (telefone sem luz), `D-15` (água sumindo) e `D-17`
  (`VIEW_SLOTS` acima do tier-2) — não são tocados por nenhum dos três PRs.
- Abre defeito novo? Não verificável sem merge. `#220` é reversão parcial: se entrar, muda o
  comportamento de invalidação que hoje está em `main`, e o próprio corpo admite não ter rodado
  runtime.

## Confiança

Verificado: topo da `main`, ausência de delta, números, corpos, arquivos e linhas dos 3 PRs, ausência de
merge na janela.

Não verificado: qualquer execução dos três PRs. Para `#221`, a alegação de "~17 dispatches por frame no
próximo da prisão" é do autor; não há log no repositório desta hora que a confirme.
