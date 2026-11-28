# fxceless/victory-fork — atualização de 2026-11-28 15:00

## Resumo

Um único commit pendente, `368967f8` (2026-11-28T13:25:44Z), e ele é grande: **+240/−13.815 em 40
arquivos**, um reset da árvore para o `VIctory-NVIDIA 959dcaf` com os consertos da água por cima. O
diff **remove** todo o trabalho de *ray guide* que a execução de 13:00 documentou — `guide.rs` (566),
`guide/core.rs` (2.194), `livery.rs` (787), `raycounts.rs` (501), `shading.rs` (284), `rays.rs` (2.690)
e `rays_tests.rs` (2.485) — e, junto, o `pipeline/sampler_order.rs` (347), que era o conserto medido de
13:00 para a recompilação de pipelines pelo driver. Ray tracing volta a ficar **ligado por padrão**
(`VICTORY_BVH_BUILDER=1` em `play.rs:75`), agora pela pilha do VIctory-NVIDIA, e `VICTORY_DYNAMIC_WAIT`
deixa de existir no `SETTINGS`.
Sem PR aberto acima do cursor e sem merge na janela.

## Commits novos (1)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `368967f8` | fxceless | The run-403 build with the water fixed: VIctory-NVIDIA `959dcaf`, ray tracing on by default, the water's own light on | Reset da árvore para `VIctory-NVIDIA 959dcaf` (o mesmo que `5dbf5ce2`) mais cinco mudanças do tradutor. Concretamente, o diff **remove** `crates/victory-gpu/src/guide.rs` (566 linhas), `guide/core.rs` (2.194), `livery.rs` (787), `raycounts.rs` (501), `shading.rs` (284), `pipeline/sampler_order.rs` (347), `crates/victory-shader/src/rays.rs` (2.690) e `emit/rays_tests.rs` (2.485), e encolhe `host.rs` (−467/+9), `memory.rs` (−622/+46), `resource.rs` (−270/+35), `pool.rs` (−382/+19), `draw.rs` (−345/+29), `descriptor/host.rs` (−141/+3), `emit/intersect.rs` (−323/+22), `emit/lower.rs` (−304/+6), `list/thin.rs` (−229/+13), `shadow.rs` (−144/+3), `copystat.rs` (−85/+12). `lib.rs` deixa de declarar `guide`, `livery`, `raycounts`, `shading` (gpu) e `rays` (shader). `apps/victory/src/play.rs`: `SETTINGS` passa a ter `VICTORY_WATER_LIGHT=1` e `VICTORY_BVH_BUILDER=1` (linha 75 do arquivo no commit, conferida por conteúdo) e perde `VICTORY_DYNAMIC_WAIT`. `README.md` −143/+28 registra o estado: ~15 fps numa RTX 4070, 1920×1080 interno para 2560×1440, "the sea is teal-green and see-through in the shallows". |

## Pull requests abertos (0 acima do cursor)

`last_pr_number = 1513`; o maior PR aberto é `#1509`. Nenhum PR novo.

## Pull requests mesclados (0)

Nenhum `merged_at > 2026-11-28T01:48:22Z` (cursor de merges). Os últimos merges documentados são
`#1485`/`#1495`, em 2026-11-28 13:00.

## Cruzamento com o catálogo

- **Corrige D-xx? Por hash, nenhum.** `368967f8` foi conferido por prefixo contra as 305 linhas: zero
  interseção.
- **Reabre, por remoção, dois consertos documentados há duas horas.** Este é o achado da hora:
  - `9e907466` (documentado em `docs/updates/2026-11-28/13/victory-fork.md`) criou
    `pipeline/sampler_order.rs` para criar os samplers estáticos numa ordem guardada e fazer o cache do
    driver responder — medição do autor: 406 → 211 → 0 pipelines acima de 1 s. `368967f8` **apaga**
    esse arquivo e as seis linhas de `pipeline/library.rs` que o referenciavam. A classe de **D-02**
    (`victory-fork`: a chave/ordem do cache não reflete o que o tradutor gerou) volta a valer, agora por
    remoção, não por switch novo.
  - `de57b010` (o guia de ray tracing) também sai inteiro. O defeito que ele **expôs** — palavra de
    memória compartilhada nunca escrita antes da primeira leitura, que derrubou o device no primeiro
    dispatch — sai com ele; o `VICTORY_RAY_GUIDE` deixa de existir. Não afirmo que a classe de D-16 do
    XCL (perda de device sem causa) esteja coberta pela pilha nova.
- **Abre defeito novo?** Não comprovado, mas há um fato verificável e desconfortável: o próprio corpo
  do commit diz que "o trabalho do ray-guide dos últimos commits não está nesta árvore (mantido no
  histórico)". Ou seja, o repositório abandonou em `main` duas frentes que estavam medidas e testadas,
  sem reverter o registro no `README.md` além do bloco novo — quem lê o `README.md` anterior a esta
  hora vê um estado que não existe mais.

## Confiança

**Verificado:** o commit foi lido por `commits/368967f8` (40 arquivos, `+240/−13.815`, lista completa de
`status`/`+/-`), com patch de `README.md`, `apps/victory/src/play.rs`, `crates/victory-gpu/src/lib.rs` e
`crates/victory-shader/src/lib.rs`, e o conteúdo de `play.rs` no commit foi baixado por
`contents/...?ref=368967f8` para conferir que `VICTORY_BVH_BUILDER=1` está no `SETTINGS` (linha 75) e que
`VICTORY_DYNAMIC_WAIT` não está.

**Não verificado:** nenhuma build, teste ou execução; os ~15 fps são relato do `README.md` do próprio
commit. Não li o diff de `memory.rs`, `host.rs`, `resource.rs`, `pool.rs` e `draw.rs` linha a linha —
afirmo apenas o tamanho e o sinal das mudanças, não o comportamento. Não sei se a remoção de
`sampler_order.rs` foi deliberada (o corpo não a menciona) ou efeito colateral do reset de árvore.

**Cobertura:** 1 de 1 commit pendente. `last_commit` avança de `6b3ce6ce` para `368967f8`; o
`commit_cursor` que apontava para `368967f8` (gravado às 13:00 como "próximo ciclo começa aqui") é
removido, porque `compare(368967f8...368967f8)` devolve zero e o commit nunca seria documentado.
