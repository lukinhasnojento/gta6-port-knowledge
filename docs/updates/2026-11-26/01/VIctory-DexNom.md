# DexNom/VIctory — atualização de 2026-11-26 01:00

## Resumo

Sem commit novo na `main`: o cursor continua em `64bd58f5` e nenhum PR mesclou nesta faixa. Quatro PRs
abertos, e três deles são correções de defeito real: um slot de heap lido como constant buffer quando não
é (fault de página de GPU na RDNA 4), cópia de textura de volume aceitando pitch de profundidade que
sobrepõe fatias, e um heap de título que não cabe no heap limitado da NVIDIA. O quarto separa espera de
CPU de tempo de GPU nas medições.

## Commits novos (0)

Nenhum. `last_commit` `64bd58f5fb794216894bc0e33e03a3a64d19c347` é o `head` da `main`.

## Pull requests abertos (4)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 68 | GPU: a title heap a capped host heap does not hold whole keeps its lower 393,215 slots; the rest are dynamic | notquiteog, +61 em `descriptor/host.rs`. Com teto de 1.000.000 de views (NVIDIA), `HostHeap::new` dá 737.855 slots ao título e deixa 262.144 dinâmicos; o mundo aberto pede ~200.000 dinâmicos por frame e o anel seca, recusando tabelas e dropando draws com o log `no free dynamic descriptor slots: a table is not made`. Passa a dar 393.215 slots ao título e 606.784 dinâmicos, com `VICTORY_DYNAMIC_SLOTS=<n>` (mínimo 262.144, deixando ao menos 65.536 de título). Mede 13.000–60.000 recusas por 5 s → 0, com 350.000 dinâmicos livres no pior caso. | **Não há linha em `DexNom` para isso.** É a classe de `D-16` ("falsa exaustão de descritores dinâmicos quando o run livre cruza o cursor", corrigido `c1c3281`) — mesmo sintoma, causa diferente: lá a busca perdia um run; aqui o anel é pequeno por construção no adaptador com teto. Candidato a linha nova, e a mesma classe de `VIctory D-01` (`edf64e2`) e `Belobug-AMD D-05` (sem corretor). |
| 67 | fix(gpu): distinguish CPU waits from GPU timestamp intervals | LZJ-Developer, +578 em 4 arquivos (`gpu_timing.rs` +390 novo, `host.rs` +111/−42, `docs/timing.md` +75). Legendas de janela e log de frame derivavam "GPU ocupada" de esperas de conclusão do lado da CPU, então espera longa de CPU aparecia como utilização alta do dispositivo. Passa a reportar a espera de conclusão em ms/frame separada, mede a espera bloqueante do evento à parte da submissão e adiciona marcas de timestamp de fila (`VICTORY_GPU_QUEUE_TIMING=1`, desligado por padrão), com duas listas de marcadores por lote da fila direta. | Não é D-xx. Instrumentação; o catálogo não tem linha de medição. |
| 66 | Read a heap slot as a constant buffer only when it holds one | ExMoha, +30/−7 em 4 arquivos (`descriptor/host.rs` +8, `lower.rs` +11/−5, `emit/mod.rs`, `shader.rs`). Num RX 9070 (RDNA 4) o dispositivo era removido com fault de página de GPU no mesmo ponto do primeiro mundo aberto, com o endereço mudando entre execuções (`0x3000_0270_3000`, `0x9000_02d9_8000`, `0x5c2d000`); o DRED apontava um dispatch indireto de culling. Causa: o programa escolhe um buffer do view heap por índice lido dos próprios registros, e o tradutor decidia pelo stride da tabela do host — stride não-zero virava buffer view, qualquer outro era lido como constant buffer. Um slot com textura, ou vazio, não tem stride e era lido como CBV. | **Não está no catálogo.** O sintoma é remoção de dispositivo por fault de página em dispatch indireto; `D-10` (escrita de `Staging::detile` além do buffer) e `D-11` (corrida no `.tmp` da pipeline library) são as linhas vizinhas de memory-safety e não descrevem isso. Candidato a linha nova, com correção neste PR. |
| 65 | Reject overlapping depth slices before copying a volume texture | Yoshi-233, +47/−5 em `victory-formats/src/texture.rs`. Um volume R8 2×2×2 com dois bytes ativos por linha e pitch de 4 precisa de 6 bytes antes da fatia seguinte; pitches 0 e 5 eram aceitos nas duas direções, e uma fatia sobrescrevia ou reaproveitava texels da anterior. Passa a checar a extensão ocupada até o último byte ativo da fatia antes de copiar, preservando o limite de `bytes_from(mip)` para mips em streaming; mip de fatia única continua aceitando pitch 0. Traz o caso de `Belobug/VIctory-Belobug-AMD#21`. | Não é D-xx diretamente. A área é validação de layout de textura; `D-10` ("`Staging::detile` escreve além do buffer mapeado (UB) — corrigido e depois reintroduzido") é o vizinho mais próximo, mas a operação e o arquivo são outros. |

## Pull requests mesclados (0)

Nenhum. A `main` não tem commit novo; a varredura por `merged_at` em `pulls?state=closed` não achou merge
posterior a 2026-11-26T03:16:53Z.

## Cruzamento com o catálogo

- Corrige D-xx? Nenhum PR fecha linha existente. As 16 linhas de `DexNom` seguem com os corretores
  registrados; `D-10`, `D-11` e `D-12` continuam **abertos por reversão** (o catálogo registra que
  `subresource_len`, `write_atomically` e `clbw_loop` voltaram à forma original), e nenhum PR desta hora
  os reaplica.
- Abre defeito novo? Dois candidatos em PR aberto, ambos com correção proposta e nenhum com linha:
  - `#66`: slot de heap sem stride lido como constant buffer → fault de página de GPU e remoção do
    dispositivo num dispatch indireto de culling (RDNA 4). É a falha mais grave da hora neste
    repositório.
  - `#68`: exaustão do anel de slots dinâmicos quando o heap do título não cabe inteiro no heap do host
    limitado — mesma classe de `D-16`, causa diferente.
- Nota para consolidação: `D-16` ("falsa exaustão … run livre cruza o cursor") e `#68` ("anel pequeno por
  construção") produzem o mesmo log e o mesmo sintoma (draws dropados). Se `#68` mesclar, o catálogo
  precisa de duas linhas distintas ou de uma linha com duas causas.

## Confiança

Verificado por API: `gh api repos/DexNom/VIctory/commits?sha=main&per_page=100` devolve `64bd58f5` como
`head`, igual ao `last_commit` do estado — zero commits. `gh pr list --state open --limit 50` devolveu 4
PRs, todos com número > `last_pr_number` 64. A varredura de mesclados não achou `merged_at` posterior ao
cursor.

Não verificado: o fault de GPU do `#66` e os números do `#68` são as medições que os autores declaram (RX
9070 / RTX 5060 Ti); não reproduzi nenhuma. Não li o DRED nem o dump citado. Não rodei teste algum.
