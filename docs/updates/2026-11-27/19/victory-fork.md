# fxceless/victory-fork — atualização de 2026-11-27 19:00

## Resumo

Seis commits desde `5dbf5ce2`, e a `main` deixou de ser "a branch sem ray tracing": as duas linhas
(`main` e `raytracing`) foram unidas em `bfca0bea` e o resultado empurrado para a `main`, que agora
liga o ray tracing do título por padrão (`VICTORY_BVH_BUILDER=1`) e a espera sem lock de submissão
(`VICTORY_DYNAMIC_WAIT=1`). Os outros três commits trazem as correções que sustentam isso: fila
própria para os resolves de ray tracing, espera por slots dinâmicos sem o lock de submissão e um
orçamento reciclável de 512 MB para as páginas vindas de faults. Nenhum PR mesclado na janela; três
abertos.

## Commits novos (6)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `bd6fbce4` | fxceless | Ray-tracing build (work in progress): the title's own ray tracing on by default | Três arquivos. `apps/victory/src/play.rs` (+3) entra `("VICTORY_BVH_BUILDER", "1")` na lista `SETTINGS` do launcher; `pipeline/make.rs` (+87/−2) marca `no_wait` para programas cujo nome contém `CS_RaytracingResolve` — enquanto o driver os faz, os dispatches são descartados de imediato em vez de esperar; `memory.rs` (+25/−2) sobe `FAULTS_ALL` de 2.048 para 8.192. |
| `52dfd12e` | fxceless | Ray-tracing fixes and a lock stall fix; README says where ray tracing is | Oito arquivos, a mesma base `5dbf5ce2`. `host.rs` (+96) ganha `wait_for_slots()`: com `VICTORY_DYNAMIC_WAIT=1` e um relay disponível, espera o próximo pouso **sem** o lock de submissão, reconsultando os slots a cada 1 ms, e só cai no `settle_for_fence()` antigo quando não achou; `draw.rs` (+3/−12) troca `settle_for_fence()` por `wait_for_slots()`; `descriptor/host.rs` (+11) publica `DYNAMIC_LOW` (menor número de slots livres); `pipeline/make.rs` e `memory.rs` repetem o conteúdo de `bd6fbce4`; `host/card_steps.rs` (+19) passa a contar **por motivo** as submissões recusadas; `queue/behind.rs` (+28) e o README completam. |
| `bfca0bea` | fxceless | Merge branch 'public' into raytracing | Merge de `52dfd12e` (public) em `bd6fbce4` (raytracing). Resolve a duplicação dos mesmos hunks de `make.rs`/`memory.rs` presentes nas duas linhas; o conteúdo consolidado é o das linhas acima. |
| `a0767765` | fxceless | Ray-tracing branch: the lock stall fix is on by default here (VICTORY_DYNAMIC_WAIT=1) | `play.rs` (+4): entra `("VICTORY_DYNAMIC_WAIT", "1")` nos `SETTINGS`, com o comentário que mede ~48 ms/frame no lock com ray tracing ligado (9.000–12.000 draws) contra ~13 ms sem. |
| `91957800` | fxceless | Ray tracing on by default | Só `README.md` (+21/−19): a documentação passa a dizer que o ray tracing está ligado por padrão desde 27/11/2026, que `play --set VICTORY_BVH_BUILDER=0` desliga, e que a branch `raytracing` agora é o mesmo código da `main`. **Nenhuma mudança de código** — o default já estava em `bd6fbce4`. |
| `3e573d25` | fxceless | Ray tracing: pages for the ray programs are a recycled budget, not a limit for the run | `memory.rs` (+412/−58). `FAULTS_ALL = 8192` sai e entra `FAULT_MB = 512` (`FAULT_PAGES = 512 MiB / 64 KiB`): no limite, a página mais antiga é devolvida — nunca nos primeiros `FAULT_YOUNG = 128` frames nem se uma lista a nomeia — e, se ainda era lida, sua nota volta e ela é refeita no frame seguinte; página devolvida e pedida de novo em até `FAULT_SOON = 4` frames conta como "ainda lida". O refugo cai para uma fila (`VecDeque`) e o log `title memory by address: ... from faults ...` passa a mostrar páginas seguradas, devolvidas e pedidas de novo; o campo `over_the_run` do aviso sai. |

## Pull requests abertos (3)

Lidos por corpo; nenhum patch lido.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #1485 | Formats: parenthesize the shift in a texture test (main's clippy step fails on it) | Uma linha: `(i * 7 >> 3)` → `((i * 7) >> 3)` em `crates/victory-formats/src/texture.rs:1346`. O corpo mostra a CI `46021947845` falhando no **primeiro** passo (Clippy, `-D clippy::precedence`) e explica que o arquivo é byte-idêntico a `trustno1-coder/VIctory-NVIDIA@959dcaf`, que não tem Actions — por isso o lint herdado passou despercebido, e os passos seguintes (`Clippy, diagnostics build`, `Tests`) nunca rodam. | Não é D-xx; é o **D-17** (dívida de verificação) se manifestando: a CI morre antes de testar. |
| #1486 | GPU: initialize the Series S compositor on Proton | Serializa a tabela de dois SRVs e o sampler estático do compositor de dois planos com D3D12 e compila o triângulo/pixel shader como SM5.0, para o `Composer::new` não falhar com `0x80004005` no GE-Proton11-14 (janela preta, plano de interface perdido). +48/−6, 1 arquivo. | Área de TuxBox D-07/D-06 (apresentação no Proton), não de victory-fork. |
| #1487 | GPU: pin the texture a placed view aliases while the copy uses it | `alias_of` devolve ponteiro cru depois de soltar o lock de `TARGETS_AT`; se o título solta a última referência, `texture::alias` lê `Box<Resource>` liberado (use-after-free). Reintroduz `object::add_ref_alive` (CAS em `refs`) e `resource::Held` (pino com `Drop`), com `alias_of` fixando a fonte sob o lock antes de ler. Forward-port de `a463dcb`/PR #1336, ausente na árvore `959dcaf`. Teste reproduz a falha antes e passa depois. | Área de D-18 do catálogo de TuxBox (aliasing de textura posicionada), não de victory-fork; aqui não há D-xx equivalente. |

## Pull requests mesclados (0)

Nenhum `merged_at` maior que `last_run` (`2026-11-27T13:05:00Z`). Conferido paginando
`pulls?state=closed&sort=updated` até o `updated_at` mais antigo da página cair em
`2026-11-26T07:04` local (10:04Z de 26/11), bem abaixo do cursor.

## Cruzamento com o catálogo

- **D-17 (dívida de verificação) continua aberto, e agora com prova.** O catálogo registra
  "this merged tree was built; its test suites were not run" para o rebase em `5dbf5ce2`. Nesta
  janela, os commits afirmam "release build passes; unit tests of the changed modules pass (77)" e
  "unit tests of the two changed modules pass (9)" — módulo, não suíte. O corpo do #1485 mostra que a
  CI do repositório morre no Clippy antes de rodar `Tests`, e que dois testes de `victory-gpu` falham
  também na árvore de origem. Nenhum desses dois commits roda a suíte inteira.
- **D-15 (page mapping do título revertido duas vezes)** não é tocado: o commit de memória trata das
  páginas do *arena* feitas a partir de notas de fault dos shaders de ray tracing, não do mapeamento
  de páginas do título.
- **D-16 (perdas de device sem causa)** não é tocado.
- **Risco novo, não defeito catalogado:** virar o default de dois switches (`VICTORY_BVH_BUILDER`,
  `VICTORY_DYNAMIC_WAIT`) muda o que todo usuário vê sem uma medição de título publicada. O catálogo
  tem um caso análogo — D-03 de `victory-fork`/irmãos: "`VICTORY_SEND_CHANGED` ligado por padrão fez
  a sala perder luz indireta". Aqui não há evidência de regressão, só de que a decisão foi tomada com
  medição parcial. **NÃO VERIFICADO** em jogo.

## Confiança

- Topologia conferida por `parents` de cada commit: `bd6fbce4` e `52dfd12e` saem ambos de
  `5dbf5ce2`; `bfca0bea` é merge dos dois; `a0767765`, `91957800` e `3e573d25` seguem em cadeia
  linear. Ou seja, a `main` atual contém a linha `raytracing` — o "duas branches" do README anterior
  deixou de existir.
- `bd6fbce4` e `52dfd12e` carregam hunks **idênticos** de `memory.rs` e `pipeline/make.rs` (mesmos
  +25/−2 e +87/−2). Não é leitura dupla: são os mesmos patches aplicados nas duas linhas antes do
  merge. O texto do commit de `52dfd12e` já descreve o estado em que "ray tracing está desligado por
  padrão aqui", que `bd6fbce4` contradiz na mesma janela — os dois foram escritos antes do merge.
- Li os diffs de `play.rs`, `host.rs` (`wait_for_slots`), `draw.rs`, `descriptor/host.rs`,
  `card_steps.rs`, `pipeline/make.rs` (struct `Pending`, `no_wait`) e `memory.rs` (`FAULT_MB`,
  `FAULT_YOUNG`, `FAULT_SOON` e a doc do módulo). **Não li** a implementação inteira de
  `give_back()` nem as 412 linhas somadas de `memory.rs` na íntegra.
- Nenhum teste foi executado. As contagens "77" e "9" são o que os commits afirmam; a falha de
  Clippy é o que o corpo do #1485 afirma, com o log da CI.
- Cursor de commits vai para `3e573d25`; `last_run` fica em `2026-11-27T13:05:00Z` de propósito —
  não houve merge, e avançar o relógio de merges sem merge é o erro que já escondeu merges por três
  horas em 27/11.
