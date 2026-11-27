# fxceless/victory-fork — atualização de 2026-11-27 22:00

## Resumo

Um único commit direto na `main`, e é a correção dos blocos de água escura: as duas listas de tiles da luz
da água recebiam slots por wave de 64 lanes no console; num host de 32 lanes a segunda metade da wave
sobrescrevia slots da primeira, e os tiles sobrescritos ficavam sem luz naquele frame. O commit reescreve
os dois appends do programa do título (byte a byte, com digest e hash de código como guarda) para que cada
lane tome o próprio slot, e é ligado por `VICTORY_WATER_TILE_SLOTS`. **O switch novo não entra na chave do
cache de shader** — a mesma classe de D-02.

## Commits novos (1)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `cd935520` | fxceless | Open water without dark patches: the water light's tile lists take a slot a lane | Cria `crates/victory-shader/src/emit/water_tile_slots.rs` (164 linhas) e o chama de `emit/mod.rs` (+16). O programa `water_shading:CS_TransmissionVolumetrics_GenerateTiles` (título 1.0.2.1) distribui cada tile de água em duas listas — sem lâmpadas, do início; com lâmpadas, do fim — e o console reserva os slots pela wave de 64 lanes inteira: a primeira lane soma a contagem da wave ao contador da lista e o slot de cada lane é a posição dela entre as lanes da wave sobre o que o contador tinha. Em duas host waves, a posição de uma lane da segunda metade precisa da máscara da primeira, trocada por group memory. No RTX 4070 (run 421, frame 2822) os tiles da segunda metade sobrescreveram os slots da primeira a poucos slots do começo: esses tiles ficaram sem luz e a água ficou escura em blocos de 16 × 16 pixels em volta de barcos e píeres. O `rewrite()` reconhece o programa por digest do container (`f5d10cde 7433 3782 f30f 9bb5 56c0 73d4`), `BYTES = 2112` e FNV-1a do código (`0x377a8d974fb8631f`); em cada append copia para o começo as instruções que ficam — o `buffer_atomic_add` (com `glc`), os `s_waitcnt`, o load do tamanho da lista e a reversão do slot — e preenche o resto do corpo com `s_nop`. O `v_mov_b32 v5, 0` em `0x6c4` tem o campo de fonte trocado de `0x80` (constante 0) para `0x81` (constante 1), de modo que cada lane soma 1 e usa o valor anterior do contador como slot. Alvos de branch e o código depois dos appends mantêm o lugar. `VICTORY_WATER_TILE_SLOTS=0` desliga; `host_form()` só age quando `lower::host_max_wave() == 32`, e é chamado por `translate()` e por `translate_plain()` antes de montar o programa. Dois testes: programa desconhecido não é tocado; o append vira um add por lane e mantém o comprimento. `README.md` documenta o switch. |

## Pull requests abertos (0)

Nenhum PR aberto novo acima do cursor (`last_pr_number` 1493). A listagem de abertos continua batendo no
teto de 100; nada novo entrou nesta janela.

## Pull requests mesclados (0)

Nenhum `merged_at` maior que `last_run` (2026-11-27T18:10:00Z). O commit desta janela é direto na `main`,
não veio de PR.

## Cruzamento com o catálogo

- **Corrige D-xx?** Nenhum identificador `D-xx` é fechado: o defeito corrigido (slots das listas de tiles
  da luz da água em host de 32 lanes) não está catalogado. O mais próximo é **D-12** ("loads escalares
  agrupados de 4 dwords leem além do fim da view — água preta"), que tem o mesmo sintoma visual e
  mecanismo diferente: ali é leitura fora da view por load agrupado; aqui é colisão de slot entre as duas
  metades da wave. D-12 continua aberto (sem hash de correção no catálogo).
- **D-02 (chave do cache de shader sem os switches de wave):** **reaberta por um switch novo.** Lido o
  `crates/victory-gpu/src/shader_cache.rs` no commit `cd935520`: a chave é montada com
  `env!("VICTORY_TRANSLATOR")`, `emit::group_sync()`, o limite de loop, `emit::shader_opt()` e uma lista
  fixa de variáveis de ambiente que termina em `VICTORY_THREAD_TICKET`. `VICTORY_WATER_TILE_SLOTS` **não
  está nessa lista**, e o switch muda o código gerado (`host_form()` altera o programa antes da tradução).
  Duas execuções do mesmo build do tradutor, uma com o switch ligado e outra com `=0`, geram shaders
  diferentes com a mesma chave: a segunda recebe a tradução da primeira. É a mesma classe de D-02
  ("traduções envenenadas por switch"), catalogada como corrigida em `e9765a7`.
- **Achado anterior confirmado:** a mesma lista de `shader_cache.rs` continua sem `VICTORY_MEMORY_NEAR`,
  `VICTORY_LDS_START` e `VICTORY_WAVE_LOOP_SKIPS`, os três já apontados no deep dive de 21:00 como
  switches que mudam o código gerado e ficaram fora da chave. Nada os acrescentou nesta hora.
- **Abre defeito novo?** Um ponto de atenção: a reescrita é por offset fixo (`MOVE_ZERO = 0x6c4`,
  `NOP = 0x6d8`, `FRONT = 0x6a8..0x704`, `BACK = 0x70c..0x774`) com guarda de digest + hash do código. Se
  o título mudar o programa, a guarda recusa e nada é alterado — falha segura. O risco está no meio: um
  programa com o mesmo digest/hash mas com outra disposição de instruções não existe (a guarda é forte);
  o risco real é o bake de slots mudar a ordem dentro das listas, que o comentário afirma que ninguém lê
  — não verificado em execução.

## Confiança

**Verificado por diff:** o commit `cd935520` foi baixado e lido por arquivo e patch
(`water_tile_slots.rs` completo, `emit/mod.rs` completo, `translate.rs`, `README.md`). A chave do cache foi
lida no arquivo do repositório em `cd935520` (`crates/victory-gpu/src/shader_cache.rs`), não inferida de
mensagem.

**Não verificado:** nenhuma execução do título; a alegação de que os blocos escuros somem e a de que a
ordem dentro das listas não é lida não foram conferidas na GPU. O `run 421, frame 2822` do comentário é
evidência do autor, não medida aqui. Nenhum teste do repositório foi rodado. A `default_branch` foi
reconferida por API (`main`); o head documentado é `cd9355206de35baeab822ce9bb2bbac7e8354eb6`.
