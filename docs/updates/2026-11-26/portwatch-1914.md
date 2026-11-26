# PortWatch — mudanças de 2026-11-26T19:14Z

Verificação manual do tracker (`portwatch-vi.xyz/data/tracker.json`) contra o estado registrado em
2026-11-25T22:09Z. **44 projetos** rastreados (eram 43). Nenhuma notícia nova publicada desde
2026-11-25T11:02Z — o que mudou foram os cartões de progresso, não o noticiário.

## Entrou um projeto novo, direto no topo

| Campo | Valor |
|---|---|
| Nome | **Kelvaroq** |
| Repositório | `stellarr-studio/kelvaroq` |
| Licença | GPL-3.0 |
| Estrelas | 194 |
| Último push | 2026-11-26T17:59Z |
| HEAD | `e062a82e` |

Runtime em Rust construído **sobre o VIctory e o victory-fork**, com série de patches documentada e
revisões do upstream fixadas. Roda o prólogo e a primeira missão a ~10 fps (7-8 em vistas pesadas).
Um playtest externo de 36 minutos numa RTX 4090 chegou à primeira missão sem crash.

Bloqueios declarados: submissão e sincronização de CPU continuam o gargalo; faltam pipelines de
shader; 7% dos draws são descartados; pressão de VRAM.

Já foi adicionado a `state/watch.json` — a rotina horária passa a monitorá-lo. São **14 repositórios**.

## Belobug: de 4 para 22 fps, e o README não acompanhou

| Componente | 2026-11-25 | 2026-11-26 |
|---|---|---|
| fps | 4 | **22** |
| stability | 0,7 | **1,0** |
| story | 0,5 | **0,8** |
| render | 0,85 | 0,8 |
| RAM | ~22 GB | ~30 GB |
| VRAM | >16 GB | 12 GB |

O cartão diz "very playable", sem crashes, com o mundo iluminado e as missões correndo. Observação do
próprio PortWatch: **o README continua reportando 5-7 fps e chamando o build de "not playable yet"**.
A distância entre o que o projeto declara e o que o tracker mediu é de quatro vezes.

Isso é material para o acervo porque é a mesma classe que já documentamos três vezes (mensagem de
commit que não corresponde ao diff, README que contradiz o código): **documentação que subestima o
próprio projeto**. A nota de Documentação do Belobug no comparativo (`docs/comparativo/Belobug-AMD.md`,
nota 4) foi dada quando o README era o mais preciso do conjunto; hoje ele é o mais desatualizado.

Ainda em aberto, segundo o cartão: ray tracing funciona na AMD segundo o dono, mas o README diz que
ligar o caminho de ray tracing ainda termina em remoção de device; e continua testado numa única
máquina AMD.

## AvatarManz já fez a consolidação que o plano recomendava

O cartão descreve algo que muda a leitura do `docs/plano/PLANO_REVISADO.md`:

> "It brings victory-fork's, DexNom's and Belobug's work together on VIctory and adds its own
> speed-ups: the game's draw calls translated on eight threads of its own, frames prepared on a
> separate thread, and textures uploaded on their own queue."

E, nos destaques: *"history and credits kept; builds from source, no prebuilt downloads;
GPL-3.0-or-later"*.

Métricas: ~16 fps em jogo (13-20 em interiores, 19-22 na primeira missão de barco, 30 em menus),
41 minutos sem crash, som funcionando, 40-46 GB de memória. Bloqueios: perseguições em rua movimentada
caem para 6-10 fps; árvores e prédios distantes cintilam; salas escuras ficam escuras demais sem ray
tracing.

**Consequência para o plano:** a abordagem "VIctory como base + cherry-pick do DexNom" deixou de ser
uma proposta e virou um repositório em produção. Vale ler o histórico dele antes de repetir o
trabalho — e vale checar se a contaminação GPL-2.0-only do `console_insn*.rs` se propagou para os
trechos vindos de victory-fork, DexNom e Belobug.

Duas ressalvas continuam de pé para ele: `Reaproveitamento` seguia 0 no comparativo por causa da
licença, e 40-46 GB de RAM não é um requisito democrático.

## ZETTROX: o repositório continua fora do ar

`Zettrox/ztrx` responde **404** — o tracker mantém o cartão (2 fps, estabilidade 0,8, "first kill
with a gun" verificado), mas o código não está acessível. Segue como o único do top 6 sem código
público.

## Estado dos 14 monitorados no tracker

| Nome | Repositório | world | playable | story | render | fps | stab |
|---|---|---|---|---|---|---|---|
| Kelvaroq | `stellarr-studio/kelvaroq` | 1 | 1 | 0,25 | 0,25 | 10 | 0,8 |
| Belobug | `Belobug/VIctory-Belobug-AMD` | 1 | 1 | 0,8 | 0,8 | **22** | **1** |
| DexNom | `DexNom/VIctory` | 1 | 1 | 0,5 | **0,85** | 3 | 0,7 |
| ZETTROX | `Zettrox/ztrx` | 1 | 1 | 0,1 | 0,3 | 2 | 0,8 |
| Uilian | `uiliannn/VItoriaBr` | 1 | 1 | 0,1 | 0,2 | 4 | 0,4 |
| AvatarManz | `avatarmanz/VIctory-7900XTX` | 1 | 1 | 0,5 | 0,7 | 16 | 0,9 |
| hyphennn | `fxceless/victory-fork` | 1 | 1 | 0 | 0,7 | 3 | — |
| VIctory | `Marvisak/VIctory` | 1 | 1 | 0 | 0 | 1 | — |
| xbox-emulator | `omonkeyz/xbox-emulator` | 0 | 0 | 0 | 0 | 0 | — |
| XCL | `xcl-dev/xcl` | 0,5 | 0 | 0 | 0 | 0 | — |

`OpenScarlett`, `TuxBox`, `SixNG` e `GTA6-Asset-Info` aparecem no tracker nos tiers de watchlist ou
toolkit, sem mudança de cartão nesta verificação.

## O que observar nas próximas horas

1. **Se o README do Belobug for atualizado** para 22 fps — indicaria reconciliação; se continuar em
   5-7, é a divergência documental mais cara do conjunto.
2. **Se o Kelvaroq sustenta o topo** — 194 estrelas em menos de um dia e playtest externo verificado
   é o começo mais forte de qualquer projeto até agora.
3. **O histórico do AvatarManz** — é a primeira consolidação real do ecossistema e a melhor fonte
   para o repositório unificado.
4. **Se o ZETTROX volta** — era o único build NVIDIA com estabilidade de 1h30 e continua inacessível.
