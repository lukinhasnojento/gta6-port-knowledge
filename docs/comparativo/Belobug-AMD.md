# Acertos e defeitos de Belobug/VIctory-Belobug-AMD

## Ficha

| Campo | Valor |
|---|---|
| Repositório | `Belobug/VIctory-Belobug-AMD` |
| Commits | `751` (`f4504f6` raiz → `4f62c2a` HEAD; 2026-11-23 a 2026-11-25; 148 de `beloooo` + 189 de `Claude`, o resto upstream re-commitado com hash novo) |
| Licença | `GPL-3.0-or-later` (`Cargo.toml`, `LICENSE`) |
| Papel no ecossistema | Fork do VIctory que faz o bring-up do mundo num único RX 9070 XT e devolve ao ecossistema as correções de AMD: helper lanes, custo de `ExecuteIndirect`, contorno do compilador do driver (`03629f3`, `ccab973`, `1503444`). |
| Estado hoje | Mundo renderiza iluminado com um substituto constante da luz indireta, 5-8 fps, ~22 GB de RAM e mais VRAM que os 16 GB da placa, telefone quebrado, nada testado em NVIDIA ou Intel (`4f62c2a`, README). |

## Placar

| Dimensão | Nota | Justificativa |
|---|---|---|
| Engenharia | 4/5 | As correções caras são pequenas e corretas (`03629f3`, 17 linhas; `ccab973`, 225→72 ms), mas metade dos defeitos foi **contida**, não resolvida (`4f45ab8`, `7263339`, `df71546`). |
| Verificabilidade | 3/5 | Existe CI de clippy+testes (`.github/workflows/ci.yml`, `8e79c80`) e ~592 `#[test]`, mas os três acertos principais não têm teste e o README declara 6 testes falhando na própria máquina (`03629f3`, `b76a71b`). |
| Documentação | 4/5 | README declara o que está quebrado, o que é substituto, "cause not proven" e o número de cada medida; perde ponto por arredondar 72→"about 80" e por "5 to 7" contra "5 to 8" na própria tabela (`b76a71b`, `ccab973`). |
| Sustentabilidade | 2/5 | Defaults perigosos ficaram ligados: sweep de tiles desligado, `crashed.txt` persistente, `view_slots` acima do teto, 114 switches de ambiente e 152 marcas `shortcut:` (`9956958`, `7263339`, `4f62c2a`). |
| Reaproveitamento | 4/5 | Cinco peças são diretamente portáveis (`03629f3`, `1503444`, `ccab973`, `ae75a66`, `0434b3a`); o resto é tuning de um título numa GPU. |

**Média:** `3.4/5`

## Acertos

O que o projeto fez **certo**, com evidência. Ordene por valor.

| # | Acerto | Commit | Por que importa | Reusável? |
|---|---|---|---|---|
| A-01 | Toda leitura de outro pixel do quad em pixel shader passa por `quad_pick`, helper lanes incluídas | `03629f3` | 17 linhas em `lower.rs`; na AMD leitura de wave devolve 0 de helper lane, então a derivada na borda do triângulo dava o valor inteiro e a textura caía no menor LOD — as linhas em pintura, vidro e pele. `git log 03629f3..HEAD -S"helpers_read"` não devolve nada: nunca foi desfeito. | sim |
| A-02 | Clamps de `ExecuteIndirect` saem do caminho por draw e vão para uma lista anterior | `ccab973` `918ab03` `76387c2` `684a9b5` | Emulação de clamp custava 1 dispatch + 3 barreiras por draw: 9.300 por frame, 190 de 225 ms de GPU. Medido: 225 → 72 ms de GPU e 2,6 → 3,9 fps; `684a9b5` tira 3 chamadas D3D12 por draw indireto. | sim, com ressalva (a invalidação por escrita é a única proteção) |
| A-03 | A saída do resolve de luz indireta é **substituída** por uma constante, não elevada por um piso | `4f45ab8` `4f62c2a` | A primeira tentativa (`0603123`) somava luz sobre uma imagem nunca escrita — garantia que o lixo ficasse visível. Substituir troca magenta nas folhas e blocos de cor em interiores por luz uniforme; `VICTORY_AMBIENT=off` preserva a saída do título e o nível segue o céu por default. | com ressalva (nomes e offsets do título, sem oclusão) |
| A-04 | Contagem de elementos de buffer vem da tabela do host, não de uma size query no shader | `1503444` `1ebf29d` `fa042f6` | `GetDimensions` de structured buffer com certos strides **crasha o compilador da AMD**; `COUNTS_AT = OWN_SLOTS` publica stride e contagem e o shader lê dali. Contrato `victory-shader`↔`victory-gpu`, portável inteiro. | sim (os dois lados juntos) |
| A-05 | Estado do guard de memória por VEH completo: buracos, pins de 2 s, runs que cruzam views, todas as views da página física | `2b6adbb` `1c7231d` `e7e6c00` `579a8d9` `ae90937` `4c1766c` | Quatro falhas independentes no mesmo mecanismo, cada uma derrubava o título (tempestade de 2,5 GB/frame; bloco read-only sem registro; leitura assíncrona do kernel; veículos partidos). É a peça mais delicada do fork e a única com estado explicitamente completo. | sim, com ressalva (área com mais modos de falha) |
| A-06 | Cinco passes exatos de limpeza de bitcode antes do compilador do driver | `ae75a66` | DCE, dobras/identidades, CSE, fatos de branch, CSE de leitura no bloco — exatos por contrato. O compute `39f0e9aeb156a61c` compila em 36 s onde antes nunca terminava; 25.623 shaders emitidos passam no validador da Microsoft. Veio do fork `ba8697aa` e foi adaptado, com o crédito no commit. | com ressalva (escolher **um** modelo de módulo entre os forks) |
| A-07 | A "stretch" de 64 KiB do título é mapeada de novo e o heap do host aberto ali | `0434b3a` | Elimina cópia por bindless: 33 janelas por frame, copy-back 36 → 12 ms, frame 227-266 → 202-216 ms. Padrão aplicável a qualquer título que fragmente heaps em views. | com ressalva (a contiguidade tem de ser verificada — `78028ad` mostra o custo de presumir) |
| A-08 | Esgotamento de slots dinâmicos de descritor: de colapso a degradação | `2aad398` `df71546` `788ea46` `b41f3f3` | Run e1 ficou sem os 524.288 slots em 6 minutos (draws dropados, 0,3 fps); depois: uma cópia por tabela pelo maior comprimento (pico 369.107), busca falha memoizada, ≤8 por frame, hash rápido no lugar de SipHash. Ainda falta slot nas cenas mais pesadas, mas não colapsa. | sim, com ressalva (a arquitetura continua sendo o limite) |
| A-09 | O critério de "compilação sem fim" foi afrouxado com o erro nomeado e medido | `7263339` | 20 s/3 GiB marcava como eternos quatro pipelines que o driver faz em segundos **durante o load**, excluindo-os para sempre; 60 s/8 GiB e o README passa a mandar apagar `pipelines/crashed.txt`. Erro reconhecido, número publicado, consequência documentada. | sim (como política, não como constante) |
| A-10 | Submissão assíncrona final: só as submissões do título retornam antes | `a59f3e5` | 200-238 → 150-186 ms/frame; a versão anterior (`9b89ec2`) liberava o buffer de captura do present com a cópia pendente e removia o device no primeiro frame — a correção separou posse por origem. | com ressalva (medido numa máquina e num lugar) |
| A-11 | Instrumentação que pagou por si | `ccab973` `c28201f` `afca189` `4f45ab8` `cf62e1a` | `VICTORY_GPU_TIMING` por tipo de comando achou os 225 ms; censo de texturas colocadas; `VICTORY_SHADER_PATCH`/`SHADER_SWAP` isolam um shader sem rebuild; `VICTORY_HOIST_FILE`/`AMBIENT_FILE` ajustam em jogo. Foi isso que permitiu achar em uma hora o que contadores próprios não acharam em um dia. | sim |
| A-12 | Integração de PRs de terceiros com crédito e revisão declarada | `04b1a1d` `d6070f1` `64170dd` `14545fa` | Trouxe main oficial até `8e79c80` resolvendo conflitos à mão, mais os PRs #18, #45, #96 e os oito de romainhedouin, cada um com autor nomeado no README. | sim |

## Defeitos

O que o projeto fez **errado**. Ordene por gravidade.

| # | Defeito | Gravidade | Commit | Estado |
|---|---|---|---|---|
| F-01 | Sweep de tiles desligado por default: cópias de memória do título nunca são devolvidas e o processo chega a ~33 GB commitados em 15 minutos | ALTO | `9956958` (default em `shadow.rs:1642`, HEAD `4f62c2a`) | aberto (escolha declarada: segurança > memória) |
| F-02 | `pipelines/crashed.txt` persistente + watchdog de 20 s/3 GiB marcou quatro pipelines legítimos como eternos e os excluiu **para sempre** | ALTO | `95e935d` `2655edb` → `7263339` | mitigado (o arquivo envenenado tem de ser apagado à mão) |
| F-03 | Slots dinâmicos de descritor acabam nas cenas mais pesadas: draws do frame são dropados e um frame chegou a 7-11 s | ALTO | `df71546` `788ea46` | aberto (README, "Known problems") |
| F-04 | A luz indireta é uma constante global: sem oclusão, interiores com paredes/pisos/tetos pretos, 611 de 611 dispatches de resolve vazios sem rays | ALTO | `0603123` → `4f45ab8` `4f62c2a` | mitigado (stand-in estável, não é a correção) |
| F-05 | 266 programas do pack não traduzem (195 usam endereços de console em registradores escalares; 71 mesh shaders passam de 32 KiB de saída) e os draws caem | ALTO | `7e480f6` (recusa explícita), `b76a71b` (número) | aberto |
| F-06 | `view_slots()` pede `max(count+SLOTS, 2_294_784)`, acima do teto tier-2 de 1.000.000 documentado pelo próprio fork: o heap nasce cortado, slots são perdidos e o warning "a view heap shorter than shaders keep their indices in" dispara a cada heap | MÉDIO | `9956958` `6cf5090`; teto em `host.rs:514` (`PORTABLE_VIEWS`) | mitigado por `3827767` (tabelas acima do teto são refeitas em slots dinâmicos); o invariante do comentário continua falso |
| F-07 | Revert em bloco de 26 PRs do upstream (`a62995f`) deixou o fork **abaixo** do upstream em loader, saves e XAsync por horas | MÉDIO | `a62995f` | mitigado (reabsorvido em `6269b57`, `9e44f38`, `56d5032` e outros) |
| F-08 | Ligar o hoist por default introduziu flicker de objetos mesh; a correção final é o modo 7, sequencial e mais lento que o modo 1 | MÉDIO | `918ab03` → `afca189` `76387c2` | corrigido (com custo residual aceito) |
| F-09 | Telefone in-game deixa o mundo sem luz; não há diagnóstico nem hipótese no repositório | MÉDIO | `71652a2` (README, HEAD `4f62c2a`) | aberto |
| F-10 | Depois de reconstruir o cache de shaders, a água desaparece até o jogo ser iniciado de novo; causa não provada | MÉDIO | `ae75a66` `b76a71b` | aberto |
| F-11 | Dívida estrutural: `lower.rs` com 14.678 linhas / 630 KB (acima dos 512 KiB de `xtask/src/policy.rs`), 152 marcas `shortcut:` e 114 switches `VICTORY_*` | MÉDIO | `4f62c2a` | aberto (declarado no README) |
| F-12 | A segunda tentativa de submissão assíncrona removia o device no primeiro frame (buffer de captura liberado com cópia pendente) | MÉDIO | `9b89ec2` | corrigido em `a59f3e5` |
| F-13 | README arredonda 72 ms para "about 80 ms" e diz "5 to 7 frames per second" no cabeçalho contra "5 to 8 fps" na tabela | BAIXO | `ccab973` `b76a71b` | aberto |

Nota de correção sobre F-06: a análise de origem descreve o fallback como `create(count + 1024)` com warning a cada heap; em HEAD o caminho é um bloco dedicado (`host.rs:692-707`) que cria `PORTABLE_VIEWS - SLOTS` e reporta `made`/`lost`. O defeito de fundo — dois números incompativeis no mesmo desenho — permanece (`4f62c2a`).

Escala de gravidade:
- **CRÍTICO** — quebra o build, perde dados do usuário, ou impede o uso por qualquer pessoa.
- **ALTO** — corrupção de resultado, crash frequente, regressão de correção anterior.
- **MÉDIO** — comportamento errado em caso específico, dívida que trava evolução.
- **BAIXO** — higiene, nomenclatura, inconsistência documental sem efeito funcional.

## Erros de processo

Fatos sobre **como** o projeto trabalha, não sobre o código.

| # | O que aconteceu | Evidência |
|---|---|---|
| P-01 | Revert e reaplicação do **mesmo** merge no mesmo dia, três vezes, durante a campanha `integrate/all` | `e1dd56b`/`d48de53` (pr/12), `5da8aaf`/`8302f6a` (pr/213), `2645fcd`/`50a3f1e` (pr/150) |
| P-02 | Decisão de risco tomada por uma máquina só: 26 PRs removidos em bloco porque "nenhum deles muda o que o RX 7600 mostra" | `a62995f` |
| P-03 | Mensagem de commit não descreve o efeito: `a873b24` junta correção de use-after-free **e** "coalesce VirtualProtect", e a coalescência criou o hang que precisou de `e7e6c00` | `a873b24`, `e7e6c00` |
| P-04 | Commit de correção carrega mudança de default não relacionada (fix do fault storm + cap de 240 FPS) | `2b6adbb` |
| P-05 | "Work in progress" commitado direto na main | `afca189`, `3a839d4` |
| P-06 | Ganho medido num ponto e generalizado: "225 to 72 ms ... 2.6 to 3.9 fps" em `ccab973` vira "225 to about 80 ms ... in a scene of 9,000 draws" no README, e o fps de mundo continua 5-8 | `ccab973`, `b76a71b`, `4f62c2a` |
| P-07 | Histórico do upstream re-commitado com hashes novos: as duas histórias só compartilham `f4504f6` e não há merge-base com o HEAD do VIctory, então comparação por hash entre repositórios é inválida | `f4504f6`, `12caefe`, `FORK_DIVERGENCIA.md` |
| P-08 | CI de clippy+testes existe e roda em PR e main, mas o README declara 6 testes falhando na máquina do autor e os três acertos principais não têm teste (o diff de `03629f3` é só `lower.rs`; hoist foi validado por olho em 6 modos, luz por runs) | `.github/workflows/ci.yml`, `8e79c80`, `b76a71b`, `03629f3`, `afca189` |
| P-09 | Referência a PR de terceiro como parte do argumento sem reproduzir a medição: `3827767` foi testado numa RTX 3080 e é o que sustenta o cap de views, mas o fork nunca rodou NVIDIA | `3827767`, `4f62c2a` (README: "Nothing here has been run on an NVIDIA or Intel card") |

## O que copiar

- `03629f3` inteiro: `helpers_read(quad) = quad && self.s.pixel` e `quad_pick` em toda leitura de quad de pixel shader. Melhor custo/benefício do ecossistema.
- O desenho do hoist de clamps (`ccab973`, `State::hoists_into`/`hoist`/`end_hoist`, `Hoist { clamps, into, writes }`) **com** o teste de invalidação por escrita que faltou na primeira versão.
- `1503444` + `1ebf29d` + `fa042f6`: contagem e stride pela tabela do host (`COUNTS_AT`, `StrideTable`), portando os dois lados do contrato.
- `watch.rs` completo (`2b6adbb`, `1c7231d`, `e7e6c00`, `579a8d9`, `ae90937`, `4c1766c`) — com os testes de região/alias/pin que o fork não escreveu.
- `crates/victory-shader/src/emit/bitcode/opt.rs` (`ae75a66`), depois de escolher um único modelo de módulo entre os forks que o têm.
- `0434b3a` (stretch mapping) com a verificação de contiguidade que `78028ad` provou necessária.
- `7263339` como **política**: timeout por tempo *e* por memória, com número e consequência escritos no commit.
- O conjunto de diagnóstico: `VICTORY_GPU_TIMING`, censo de texturas colocadas (`c28201f`), `VICTORY_SHADER_PATCH`/`SHADER_SWAP`, arquivos de ajuste ao vivo.
- `docs/view-heap-limit.md`: o limite de view heap do D3D12 explicado com as três soluções e o custo de cada uma.

## O que não copiar

- `ambient.rs` como correção de luz indireta: é stand-in global sem oclusão, com nome de programa, root parameter e índice (`LIGHT = (1, 176)`) presos ao título. Portar só o padrão (identificar o passe cuja entrada nunca é produzida e substituir a saída), com a tabela em config.
- O default de `shadow::sweeps()` = 0 (`9956958`): troca hang por crescimento sem limite. Um repositório unificado herda os 33 GB junto.
- `crashed.txt` como cache de decisão permanente, sem escopo, sem expiração e sem comando de limpeza automática.
- `view_slots()` com `max(VIEW_SLOTS)`: o invariante do comentário não vale em nenhum host tier-2 e o warning virou ruído.
- Modo 7 como default herdado sem re-medir em outra GPU: foi escolhido por corretude, é mais lento que o modo 1 e o mecanismo fino do flicker é hipótese (`76387c2`).
- Os 114 switches de ambiente e os arquivos `VICTORY_HOIST_FILE`/`VICTORY_AMBIENT_FILE` como API pública — são ferramentas de bancada.
- A prática de revert em bloco (`a62995f`) e o ciclo revert/reapply (`e1dd56b`/`d48de53`).
- O README inteiro como está: o estado de uma máquina não é o estado do projeto, e a seção do upstream colada no fim exige que o leitor saiba onde o fork termina.

## Veredito

Este é um laboratório de bring-up de um título real num único sistema AMD, feito em três dias com 148 commits próprios e 189 de agente sobre um upstream re-commitado, e não "o VIctory afinado para AMD". O que ele tem de melhor é engenharia barata e correta: 17 linhas que consertam toda aresta de triângulo porque entenderam helper lanes (`03629f3`), e um hoist que corta 153 dos 225 ms de GPU porque mediu antes de mexer (`ccab973`). O que ele tem de pior não é o que falta — é o que ficou ligado por default: sweep desligado, `crashed.txt` envenenado, view heap pedindo acima do teto, modo 7 mais lento por corretude. A documentação é a mais honesta do conjunto (declara telefone quebrado, luz substituta, `cause not proven`, memória, testes falhando e a própria dívida), e é isso que torna o repositório utilizável como fonte mesmo com 22 GB de RAM, 5-8 fps e zero teste em NVIDIA ou Intel. A posição de líder no PortWatch (77,4%) é alegação externa, e as 1.633 estrelas não são verificáveis no clone; o que o repositório sustenta é "mundo livre renderizando iluminado a 5-8 fps", não "missões a ~4 fps" — essa parte não tem suporte em nenhum commit (`4f62c2a`).
