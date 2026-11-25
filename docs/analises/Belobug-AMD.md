# Belobug-AMD - analise de DeepSeek V4.1 Flash

## A. Identificacao

| Campo | Valor |
|---|---|
| Nome | Belobug-AMD (diretorio local `VIctory-Belobug-AMD`) |
| URL | https://github.com/Belobug/VIctory-Belobug-AMD |
| Natureza | **Fork** de `Marvisak/VIctory` (nao e original) |
| Branch analisada | `main` (unica branch local; `origin/HEAD -> origin/main`; nenhuma branch remota nao mergeada) |
| Commit 0 (raiz) | `f4504f6` - 2026-11-23 01:15:26 +0100 - Marvis - "Initial commit" |
| Ultimo commit | `4f62c2a` - 2026-11-25 15:56:22 -0500 - beloooo - "The ambient light follows the sky by default" |
| Periodo coberto | 2026-11-23 a 2026-11-25 (3 dias com commit: 107 / 574 / 70) |
| Total de commits | **751** em HEAD e 751 em todas as refs |
| Merges | 287 (`git rev-list --merges --count HEAD`); 183 com "pull request" ou `pr/NN` no assunto |
| Arquivos versionados | 186 |
| Commits examinados em profundidade | 45 com diff integral (pacote `_evidencias`), ~35 adicionais abertos por `git show`/`git log -S`/`--grep`; os 751 estao no indice (secao K) |
| Modelo / data | DeepSeek V4.1 Flash - 2026-11-25 |

Autoria (por `git log --format=%an`):

```
189 Claude      148 beloooo     100 Marvis       43 mich0206
 20 Duff         19 Manuel Perez  19 r4.          14 TG0000
 13 Marvis(noreply) 12 tedin7     11 wrothgaril    9 Maraco91
  8 romainhedouin  7 Nikita Romanovich  7 Yaekai
```

Interpretacao: "Claude" e o agente que executou merges e integracoes em nome de
beloooo; "Marvis" e o autor do upstream cujos commits foram **re-commitados**
neste fork (mesmos assuntos e datas, hashes diferentes). O trabalho proprio do
fork e o par beloooo (148) + Claude (189).

**Aviso de arqueologia.** O HEAD do `Marvisak/VIctory` nao e ancestral deste
fork: as duas historias compartilham apenas a raiz `f4504f6` e divergem
imediatamente depois dela (`FORK_DIVERGENCIA.md`, secao de abertura; nao
reproduzido localmente porque o clone da base nao esta em `repos/portwatch`).
Consequencia pratica, confirmada por inspecao: os commits de Marvis aparecem
aqui com hash re-gerado (o segundo commit do fork e `12caefe`, filho direto de
`f4504f6`, identico em assunto ao segundo commit do upstream). Nao ha
cherry-pick possivel; toda comparacao neste documento e por conteudo de
arquivo e por mensagem/diff, nunca por hash entre repositorios.

## B. Resumo tecnico

### B.1 O que e o VIctory

Reimplementacao nativa, para Windows, da plataforma Xbox Series X|S, suficiente
para executar Grand Theft Auto VI a partir dos arquivos do usuario. O codigo
x86-64 do titulo roda sem alteracao dentro do processo do emulador; o VIctory
fornece a plataforma por baixo (HLE) e converte os dados do titulo - shaders sob
demanda, resto antecipadamente.

Workspace Rust com 186 arquivos versionados: nove crates em `crates/` mais
`apps/victory` e `xtask`.

| Crate | Papel |
|---|---|
| `victory-loader` | PE, imports/exports/TLS, VEH, threads, crash handler |
| `victory-pe` | imagem PE, import libs, layout |
| `victory-hle` | modulos de plataforma: xmem, xasync, xgamesave, audio/XMA, dstorage, gameinput, kernel, lifecycle |
| `victory-shader` | GFX10 (RDNA2) -> IR -> DXIL: `ir`, `structure`, `resource`, `io`, `emit/lower`, `emit/bitcode`, `emit/bitcode/opt` |
| `victory-gpu` | D3D12 no host: `host`, `draw`, `descriptor{,/host}`, `list`, `pipeline{,/host,/library,/make}`, `shadow`, `watch`, `resource`, `texture`, `texpool`, `indirect`, `ambient`, `twin`, `capture` |
| `victory-pipeline` | cache de pipeline (lado compartilhado) |
| `victory-game`, `victory-formats`, `victory-core` | SGD5/RPF, texturas/XMA, config |
| `apps/victory` | CLI: `run`, `translate`, `shader-info`, `census`, `inspect` |

Modelo de dados central: a **memoria do titulo** (guest, enderecada por
ponteiros de 40 bits) e espelhada em copias do host (`Shadow`, tiles de 64 KiB
sob `PAGE`/`BLOCK`), com protecao `PAGE_READONLY` e handler VEH para detectar
escritas; descritores sao espelhados em heaps D3D12 do host; shaders GFX10 sao
traduzidos para DXIL e compilados pelo driver do host.

### B.2 O que este fork acrescenta

1. **Integracao de comunidade**: ~90 pull requests de terceiros trazidos para
   uma branch `integrate/all` (com 26 deles revertidos em bloco, `a62995f`).
2. **Bring-up do mundo num unico AMD RX 9070 XT**: o README registra o mundo
   renderizando iluminado, com texturas, pessoas, veiculos, vegetacao, agua e
   ceu; 5 a 8 fps; ~22 GB de RAM e mais VRAM do que os 16 GB da placa.
3. **Substituto da iluminacao indireta ray-traced** (modulo novo `ambient.rs`):
   o passe `CS_ScreenProbes_TemporalResolve` tem a saida **substituida** por uma
   constante, com tinta, sombreamento por profundidade, acompanhamento do ceu e
   contencao dos reflexos.
4. **Correcao das linhas em arestas de triangulo** (pintura de carro, vidro,
   pele): leituras de quad em pixel shader passam por `quad_pick`, incluindo
   helper lanes.
5. **Corte de 225 ms para ~72-80 ms de GPU por frame** numa cena de ~9.000
   draws: clamps de `ExecuteIndirect` retirados do caminho por draw e levados
   para uma lista anterior.
6. **Guarda de memoria do titulo por VEH** endurecida: todas as views de uma
   pagina fisica guardadas juntas, buracos nao mapeados memorizados, blocos
   pinados durante escritas assincronas do kernel.
7. **Contencao do compilador AMD**: pipelines que crasham ou nunca terminam
   custam apenas o proprio pipeline.

### B.3 Estado atual (README, 2026-11-25)

```
Boot, menus ............... funcionam, 30 fps
Mundo .................... renderiza iluminado, com texturas
Frame rate no mundo ...... 5 a 8 fps (7.2-7.7 parado numa cena de 8.800 draws)
Luz na sombra ............ substituto constante que segue o ceu
Reflexos ................. os do jogo, contidos
Veiculos em movimento .... inteiros
Telefone in-game ......... QUEBRADO (mundo sem luz com o telefone aberto)
Memoria .................. ~22 GB de RAM; mais VRAM que 16 GB
Driver timeout ........... aconteceu uma vez; "expect it"
```

## C. Linha do tempo

### Fase 0 - Replay do upstream + primeira luz no AMD (2026-11-23, 107 commits)

`f4504f6` (raiz) -> `12caefe` (GFX10 operand fields / FSSM) -> `754619f`
(Minecraft no HLE) -> `2849477` ("Run Grand Theft Auto VI from first boot into
the world") -> `015d45b` (README). Sao os commits do upstream, re-commitados.

O trabalho proprio comeca nesse mesmo dia, ainda sobre a plataforma:
`d852b24` (filas seguram trabalho atras de waits de valor de memoria; "not yet
checked in the world"), `4d34cef` (corta lista em wait que segue trabalho),
`fd1bf62` (listas de retangulo sem vertex buffer via geometry shader gerado),
`eed372` (objetos aposentados retidos 120 frames), `774a50f` (liberar objetos do
host alguns frames apos o ultimo uso), `2a000c1` (root views em memoria solta),
`754f401` (rodar no adaptador mais rapido), `b02980d` (SGPR spillado em VGPR),
`e0c582e` (**primeiro revert**: PR 185 do upstream, "copies gigabytes per frame
in the world"), `161dc7f` (diagnostico de teclas e capturas).

### Fase 1 - A campanha "integrate/all" (2026-11-24, madrugada, ~90 PRs)

Entre `a75e290` (primeiro merge da serie `pr/NN`) e `a62995f` o fork absorve
dezenas de PRs da comunidade num intervalo de horas, com churn de revert/reapply:

- `e1dd56b` Revert pr/12 -> `d48de53` Reapply pr/12;
- `5da8aaf` Revert pr/213 -> `8302f6a` Reapply pr/213;
- `2645fcd` Revert pr/150 -> `50a3f1e` Reapply pr/150;
- mais 10 reverts de merges (`d0f97ef`, `4270e8f`, `13d385d`, `fe72473`,
  `011e7de`, `9b32471`, `4eb9643`, `7dfc31b`, `b79192f`);
- `a62995f` (**Revert em bloco de 26 PRs**: audio/XMA, saves, XAsync/XTaskQueue,
  paths, PE, loader hardening) com a justificativa "nenhum deles muda o que o
  RX 7600 mostra ou se o jogo fica de pe, e cada um e mais uma coisa que pode
  quebrar, como o #234 quebrou";
- `85f90f1` Revert do #234 (xmem section views / DirectStorage sobreposto).

No meio disso, `04b1a1d` traz o **main oficial ate `8e79c80`** com 41 PRs,
resolvendo conflitos a mao e registrando o que ficou de fora (#566 per-lane phi
e o waterfall loop, que reescrevem o mesmo tracking de descritor por lane).

### Fase 2 - Traducao de shaders e o primeiro mundo "de pe" (2026-11-24, dia)

- `8c26061` "the world stays up": descritores nulos com o tipo certo, listas
  seguram o que nomeiam, `ExecuteIndirect` com clamp por compute, **off by
  default** (`VICTORY_INDIRECT=1`), mundo preto.
- `35b054e` (Marvis, upstream): pipelines em threads de trabalho + cache de
  pipeline library; **execucao indireta vira default**; mundo ainda sem
  geometria e morrendo em page fault em ~55 s.
- Traducao: `02564c5`/`1f7f7d2`-familia de PRs, `7463484` (scratch como array
  por lane), `e0865d2` (wave sizes do host), `42ad74e` (phis de descritor por
  lane), `b67d15d` (escalares saindo de loops divergentes), `3b27a86` (fp16
  packed), `a8bfe78` (whole-wave sections), `01c900d` (waterfall em um passe) e
  as correcoes de VALU de `d3a5f2d`, `a5af45f`, `db20637`, `fa5189d`.
- Rays como experimento: `e63e820` (memoria por endereco), `d567f6b`, `36a19d1`
  (`VICTORY_RAYS=1`), `72e748e` (ordem de atomics dos builders), `0fe13af`
  (rays nao acertam nada onde o BVH nao pode ser lido).
- Material iluminado: `e52106f` (depth twins para o classificador de material)
  -> `8044d21` (twins off por branco estourado) -> `7eb9545` (twins on).
- Memoria e estabilidade: `e136a55` (orcamento de staging 1 GiB, pool de
  snapshots <= 512 MiB), `5e72a8f` (log do commit charge), `a873b24`
  (use-after-free de GPU), `2b6adbb` (holes em `watch::guard`), `9956958`
  (sweep de tiles desligado), `1b0c6b5`/`2655edb`/`95e935d`/`0d2f6a6`
  (compilador AMD).

### Fase 3 - Arqueologia de desempenho com PIX (2026-11-25, madrugada)

- `ccab973` mede GPU por tipo de comando e descobre que os clamps de indiretos
  custam 190 de 225 ms; corta a lista no primeiro draw indireto e leva os
  clamps para uma lista anterior -> **72 ms**.
- `918ab03` liga o hoist por default; `afca189` instrumenta modos 2-7 para cacar
  flicker; `76387c2` fixa o modo 7 (clamps em sequencia) e elimina o flicker.
- `0434b3a` mapeia de novo a "stretch" de 64 KiB do titulo num endereco proprio
  e abre o heap ali: copy-back 36 -> 12 ms/frame.
- `ff920f2` descobre, por PIX, cinco threads de gravacao dormindo no mutex de
  `descriptor::view` (6.600 trocas de contexto/s cada) e troca por `RwLock`;
  sem ganho de frame time - o gargalo e a thread que submete.
- Copias esparsas em paralelo: `a0a4591`/`64170dd` (copy-back 48 -> 32 ms).
- PRs de romainhedouin `13d6b73`, `ed273ef`, `684a9b5`, `f53ae3d`, `58655f9`,
  `0145ca7`, `a1ef2dd`, `127830e`: alocacoes por draw, hash de tabela,
  descriptores repetidos, clamps hoisted, staging de copia, ranges declarados
  uma vez. Nenhum mexeu no frame time; nenhum quebrou.
- `a23d1b4`/`14545fa`: thin LTO e uma codegen unit.

### Fase 4 - A imagem (2026-11-25, tarde)

- `f520772` helper lane de `v_readfirstlane` le o proprio valor.
- `03629f3` leituras de quad em pixel shader passam por `quad_pick`: **fim das
  linhas nas arestas de triangulo**.
- `4f45ab8` substitui a saida do resolve de luz indireta por uma constante
  (**fim do magenta nas folhas e dos blocos de cor em interiores**).
- `e48eb83`, `705a2d0`, `cd9f78c` (sombreamento por profundidade),
  `c3bd561` (piso dos reflexos), `4f62c2a` (**ceu como default**).
- `ae75a66` traz os passes de limpeza de um outro fork (`ba8697aa`), `7263339`
  afrouxa o timeout de compilacao, `6986a57` deixa de fora o programa que
  ativa os raios dos screen probes.
- Guarda de memoria: `e7e6c00`, `579a8d9`, `1c7231d`, `ae90937`, `f2cc100`.
- Submissao assincrona: `bdecda3` -> `9b89ec2` -> `a59f3e5` (default).

## D. Catalogo de defeitos

### D-01 - Folhas magenta e blocos de cor em interiores (luz indireta ausente)

- **Descricao / sintomas**: vegetacao magenta, superficies translucidas
  magenta, "blocos de cores" em interiores, interior preto onde o sol nao bate.
  O README atual ainda descreve o estagio anterior: "bright magenta on leaves,
  cloth and paper, and blocks of colours indoors".
- **Causa raiz**: a luz indireta difusa do jogo vem de probes preenchidos por
  ray tracing. Os passes de ray tracing **nao rodam** no host, e o programa que
  resolve os probes (`CS_ScreenProbes_TemporalResolve_*`) le uma imagem nunca
  escrita - "it holds whatever their memory did". Nao e regressao: e subsistema
  ausente desde o primeiro bring-up (`2849477`, upstream).
- **Commit que introduziu**: nao aplicavel (ausencia de implementacao). O
  consumo do passe aparece com a arquitetura de RT do upstream e com os
  experimentos `36a19d1`/`d567f6b`, mas o defeito visivel existe antes deles.
- **Tentativas de correcao**:
  1. `0603123` - "piso" **aditivo** aplicado depois do resolve: eleva o que o
     programa escreveu. Nao funciona porque o que ele escreveu e lixo; o
     proprio modulo passa a dizer "it used to be a floor under what the program
     wrote, but that is garbage".
  2. `4d63a09` - desliga por default ("the user finds the added light wrong",
     `VICTORY_AMBIENT=0.6` para ligar). Reacao a (1), nao correcao.
  3. `cd9f78c` - sombreamento por profundidade (menos luz em dobras e sob
     objetos), `VICTORY_AMBIENT_SHADE`.
  4. `c3bd561` - piso para reflexos (superficies polidas pretas sem rays).
  5. `e48eb83` + `705a2d0` - seguir o sky dome em vez de constante fixa.
- **Correcao definitiva**: `4f45ab8` - **substituir** a saida do resolve por
  `tint * level` (default 0.6), em vez de somar sobre ela; `VICTORY_AMBIENT=off`
  preserva a saida do titulo. Depois `4f62c2a` torna o seguimento do ceu o
  default.
- **Arquivos**: `crates/victory-gpu/src/ambient.rs` (novo em `0603123`,
  reescrito em `4f45ab8`), `crates/victory-gpu/src/draw.rs`,
  `crates/victory-gpu/src/shader_cache.rs`, `crates/victory-shader/src/native.rs`.
- **Evidencia** (`git show 4f45ab8 -- crates/victory-gpu/src/ambient.rs`):
  ```diff
  -    float3 least = tint * level;
  -    float3 c = unpack(light[at.xy]);
  -    float missing = saturate(1.0 - dot(c, weights) / dot(least, weights));
  -    if (missing > 0.0) light[at.xy] = pack(c + least * missing);
  +    // What the resolve left is not kept: without rays its inputs are never
  +    // written and it holds whatever their memory did (bright magenta on
  +    // translucent surfaces, blocks of colours indoors: run v6, 2026-11-25).
  +    light[at.xy] = pack(tint * level);
  ```
- **Estado atual**: mitigado e estavel (constante + ceu + sombreamento +
  contencao de reflexos). **Nao e a correcao real**: sem rays a luz e a mesma em
  todo o frame e nao ha oclusao; paredes, pisos e tetos de interior continuam
  pretos ("611 of 611 resolve dispatches are empty without rays", `7eb9545`).
- **Confianca**: CONFIRMADA (codigo, diffs e mensagens do autor). A atribuicao
  do magenta a leitura de memoria nao escrita e do proprio autor no comentario
  do shader.

### D-02 - Linhas em todas as arestas de triangulo (pintura de carro, vidro, pele)

- **Descricao / sintomas**: linhas finas acompanhando cada aresta de triangulo
  em superficies com pintura/reflexo; visivel em carroceria, vidro e pele.
- **Causa raiz**: o lowering calculava a derivada de textura "a mao" com
  `quad_perm`, que le outros pixels do quad por leitura de lane. Na AMD, leitura
  de wave devolve **0** para helper lanes. Na borda do triangulo os outros tres
  pixels do quad sao helpers, entao a derivada dava o valor inteiro, a leitura
  de textura caia no menor nivel de detalhe e a aresta aparecia como linha.
- **Commit que introduziu**: NAO VERIFICADO. `git log -S"quad_perm" -- 
  crates/victory-shader/src/emit/lower.rs` so encontra o fix (`03629f3`), o que
  indica que o caminho veio junto com o lowering herdado do upstream, sem
  commit isolavel neste fork.
- **Tentativas de correcao**:
  1. `f520772` - helper lane de `v_readfirstlane` le o proprio valor (upstream
     `0601955`): "in a waterfall loop they read no lane, took whatever
     descriptor that gave and ran to the loop limit. Triangle edges on skin and
     car paint; time lost per draw". Corrige uma causa vizinha, nao esta.
- **Correcao definitiva**: `03629f3` - nova funcao `helpers_read(quad)` que
  devolve `quad && self.s.pixel`; em pixel shader toda leitura de outro pixel do
  quad passa a usar `quad_pick` (que inclui helpers), tanto nas derivadas quanto
  no caminho de `quad_perm`.
- **Arquivos**: `crates/victory-shader/src/emit/lower.rs`.
- **Evidencia** (`git show 03629f3`):
  ```rust
  +    fn helpers_read(&self, quad: bool) -> bool {
  +        quad && self.s.pixel
  +    }
  ...
  -            let r = if mesh_quad {
  +            let r = if mesh_quad || self.helpers_read(quad) {
                   self.quad_pick(x, src)
  ```
  e o comentario: "the texture read that used it took its smallest level, and
  the triangles' edges showed as lines on car paint, glass and skin. Upstream
  does this after discards only (77053a0)."
- **Estado atual**: corrigido; `git log 03629f3..HEAD -S"helpers_read"` nao
  devolve nada, ou seja, nenhum commit posterior desfez a mudanca.
- **Confianca**: CONFIRMADA.

### D-03 - 225 ms de GPU por frame em 9.300 dispatches de clamp

- **Descricao / sintomas**: cena pesada (stilt house / ~9.000 draws) roda a
  2,6 fps; GPU saturada. `VICTORY_GPU_TIMING` mostra 190 de 225 ms por frame
  gastos em clamps de desenho indireto.
- **Causa raiz**: o console faz o clamp dos argumentos de `ExecuteIndirect` em
  hardware; no host isso foi emulado (`8c26061`, Marvis) como um **compute
  dispatch de tres barreiras por draw indireto**: "Arguments are clamped and
  charged to a per-list budget by a small compute pass first, so garbage cannot
  hang the GPU". Com `35b054e` a execucao indireta passou a ser o default, e o
  custo virou 9.300 dispatches + ~27.900 barreiras por frame.
- **Commit que introduziu**: `8c26061` (mecanismo), agravado por `35b054e`
  (default). Refinamentos que mantiveram o custo: `1f7f7d2` (clamp
  count-buffer-aware e nao destrutivo), `6b867e6` (budget sem wrap),
  `43a4ce4` (merge do PR #242).
- **Tentativas de correcao**:
  1. `1f7f7d2` - reduz o risco de budget estourado, nao o numero de dispatches.
  2. `6b867e6` - impede que o budget de 32 bits de wrap (listas de milhares de
     registros lixo rodavam por segundos), idem.
  3. `ccab973` - primeiro corte real: no primeiro draw de uma corrida indireta,
     a gravacao e cortada e os clamps vao para uma lista propria antes dela,
     escrevendo em buffers lidos por offset.
- **Correcao definitiva**: `ccab973` + `918ab03` (default) + `76387c2` (modo 7,
  ver D-04) + `684a9b5`/`ab5ece8` (PR #28: root signature, pipeline e budget
  ligados uma vez por lista, "3 fewer D3D12 calls per indirect draw").
- **Arquivos**: `crates/victory-gpu/src/draw.rs` (struct `Hoist`,
  `State::hoists_into`/`hoist`/`end_hoist`), `indirect.rs` (`hoist_mode`),
  `host.rs`, `list.rs`, `shader_cache.rs`.
- **Evidencia** (`git show ccab973`, mensagem):
  > "A run of indirect draws had a dispatch of ours and three barriers between
  > each two draws: 9,300 a frame in the world, 190 of 225 ms of GPU time. ...
  > Measured at one place: 225 to 72 ms of GPU time a frame, 2.6 to 3.9 fps."

  O README arredonda para "225 to about 80 ms ... in a scene of 9,000 draws".
- **Estado atual**: fechado. `hoist_mode()` devolve `VICTORY_HOIST_CLAMPS` ou 7
  por default (modo 7 = clamps em sequencia).
- **Confianca**: CONFIRMADA.

### D-04 - Flicker de objetos mesh (regressao do hoist de clamps)

- **Descricao / sintomas**: objetos mesh piscando entre frames depois que o
  hoist virou default; `afca189` cria modos 2-7 e um arquivo de controle ao vivo
  (`VICTORY_HOIST_FILE`) para isolar a causa.
- **Causa raiz**: com os clamps hoisted correndo intercalados com os draws (ou
  entre si, sem barreira), a ordem de escrita/leitura dos buffers de argumentos
  clampados nao era garantida para todos os draws de uma corrida. HIPOTESE
  quanto ao mecanismo fino: a mensagem diz apenas "run one after another
  (a barrier between them)".
- **Commit que introduziu**: `918ab03` (hoist on por default) sobre `ccab973`.
- **Tentativas**: `afca189` (instrumentacao, 6 modos).
- **Correcao definitiva**: `76387c2` - modo 7: clamps em sequencia, com barreira
  entre eles; vira o default.
- **Arquivos**: `crates/victory-gpu/src/indirect.rs` (5 linhas alteradas).
- **Evidencia** (`git show 76387c2 -- indirect.rs`; modo default em
  `indirect::hoist_mode`):
  ```rust
  let set = std::env::var("VICTORY_HOIST_CLAMPS").ok().and_then(|v| v.parse().ok()).unwrap_or(7);
  ```
- **Estado atual**: fechado; custo residual aceito (sequencial em vez de
  paralelo).
- **Confianca**: CONFIRMADA (regressao e fix); causa fina HIPOTESE.

### D-05 - Esgotamento de slots dinamicos de descritor: draws dropados, frames de segundos

- **Descricao / sintomas**: em cenas carregadas faltam slots; draws sao
  descartados ("fog", imagem incompleta); em casos extremos o frame leva **7 a
  11 segundos**; uma corrida chegou a **0,3 fps** apos seis minutos.
- **Causa raiz**: toda tabela que o host nao pode bindar direto (CBV com
  alinhamento 256, tabela que passa do heap, tabela com registro nulo) e
  **copiada para a area dinamica da lista**. Tabelas grandes eram copiadas para
  **cada comprimento** que um shader lia; e a busca por um run livre era linear
  e refeita a cada tentativa falha.
- **Commit que introduziu**: mecanismo herdado do upstream (`Heaps`/`tables` em
  `draw.rs`); a pressao aparece quando o mundo comeca a desenhar com indiretos
  (`35b054e`).
- **Tentativas de correcao**:
  1. `6cf5090` / `3c81420` - mais slots dinamicos (2^18) e pular draws cuja
     tabela nao acha slots; `edf64e2` (PR #336) no upstream.
  2. `2aad398` - dobra os slots dinamicos.
  3. `df71546` - tabelas grandes (>= 64 entradas) feitas **uma vez pelo maior
     comprimento ja visto** (`longest`): "Run e1 ran out of the 524288 dynamic
     slots after six minutes (draws dropped, fog, 0.3 fps) ... Run e2: peak
     369107, falling again (no leak)".
  4. `788ea46` - busca falha memoizada por tamanho (`no_run`) e no maximo 8
     buscas falhas por frame: "a heavy place took 7 to 11 seconds a frame".
  5. `b41f3f3`/`127830e` (PR #34) - hash rapido da tabela + comparacao dos
     registros, no lugar de SipHash (colisao custa copia, nao descritor errado).
- **Correcao definitiva**: **SEM CORRECAO COMPLETA**. As quatro mudancas acima
  transformam um colapso em degradacao; o README mantem o item aberto.
- **Arquivos**: `crates/victory-gpu/src/descriptor/host.rs`, `draw.rs`.
- **Evidencia**: mensagens de `df71546`, `788ea46`, `b41f3f3` (citadas acima) e
  README, "Known problems": "The per-draw descriptor slots are few for the
  heaviest places: when they run out, some draws of a frame are dropped."
- **Estado atual**: aberto; a arquitetura (uma area dinamica por lista, por
  frame) e o limite.
- **Confianca**: CONFIRMADA.

### D-06 - Compilador AMD: pipelines que nunca terminam ou crasham, 16 GB em dois minutos

- **Descricao / sintomas**: o compilador do driver nao termina certos programas
  (ray tracing do mundo), consumindo memoria sem parar; outros o crasham na
  hora; workers de compilacao morrem; o mundo trava em "black world" por
  minutos.
- **Causa raiz**: bugs do compilador do driver em programas GFX10 traduzidos
  grandes (ray tracing, um compute especifico) + um caminho especifico que
  crasha o compilador: **size query de structured buffer** (`GetDimensions`)
  com certos strides. Diagnostico do proprio fork: "a size query of a structured
  buffer of some strides crashes the AMD driver's compiler" (`1503444`).
- **Commit que introduziu**: NAO VERIFICADO (comportamento do driver; nao ha
  commit do fork que introduza).
- **Tentativas de correcao**:
  1. `1b0c6b5` - pipelines que o compilador nao termina sao deixados de fora.
  2. `2655edb` - parar compilacoes lentas pela memoria que tomam ("16 GB in two
     minutes").
  3. `95e935d` - workers perdidos sao substituidos, compilacoes penduradas
     suspensas por `SuspendThread`+`GetThreadContext` (so quando a thread esta no
     codigo do driver, nunca em funcao de heap), e nao se espera por perdidos.
  4. `0d2f6a6` - VEH `compiler_crashed` + `pipelines/crashed.txt`: o pipeline
     que crashou o compilador e lembrado e pulado nas proximas execucoes.
  5. `1503444` + `1ebf29d` + `fa042f6` - contagem de elementos de buffer vinda
     da **tabela do proprio host** (dwords depois dos strides, em
     `COUNTS_AT = OWN_SLOTS`), nao de uma size query no shader.
  6. `ae75a66` - passes de limpeza reduzem o programa: "the RX 9070 XT's driver
     makes the pipeline of compute 39f0e9aeb156a61c in 36 s; without, it never
     ended it".
  7. `7263339` - afrouxa o timeout de 20 s/3 GiB para 60 s/8 GiB, porque o
     criterio apertado **excluia para sempre** quatro pipelines que o driver faz
     em segundos.
  8. `18e57db` - workers e budget de espera proprios para screen passes (sem
     isso, os draws que preenchem as tabelas de iluminacao eram dropados e o
     mundo saia cinza/sepia).
- **Correcao definitiva**: **SEM CORRECAO** para a classe. Contencao completa
  (custo = um pipeline, nao a execucao). Um compute ainda crasha o compilador.
- **Arquivos**: `crates/victory-gpu/src/pipeline/library.rs`, `pipeline/make.rs`,
  `descriptor/host.rs`, `victory-shader/src/emit/lower.rs`,
  `victory-shader/src/emit/bitcode/opt.rs`.
- **Evidencia**: README ("the AMD driver's compiler crashes on one compute
  program"); `ae75a66` ("Compute f16876929fa64f9f still crashes the driver's
  compiler at once, with and without"); `library.rs` com
  `ENDLESS = 60 s`, `EVEN_SO = 300 s`, `GROWTH = 8 GiB`, `crashed.txt`.
- **Estado atual**: contido; `crashed.txt` e um estado persistente que pode
  envenenar um cache antigo - o README manda apagar `pipelines/crashed.txt`
  depois de mudar o criterio.
- **Confianca**: CONFIRMADA (mecanismos); causa raiz do hang e do crash e do
  driver AMD, NAO VERIFICADA em detalhe.

### D-07 - Uso-apos-liberar na GPU (texpool / Staging / HostHeap) -> device removed

- **Descricao / sintomas**: host GPU removido; page fault "inside freed" staging
  buffers ao desenhar malhas indiretas; partes de cena desaparecendo.
- **Causa raiz**: um slot de descritor shader-visible continuava apontando para
  uma textura cujo `texpool::Slot`/`ID3D12Heap` ja tinha sido liberado; o
  `HostHeap` retinha o `ID3D12Resource` mas nao o **dono** da memoria.
- **Commit que introduziu**: arquitetura herdada (pool de texturas + heap
  espelhado); a falha aparece com o mundo desenhando por indiretos.
- **Tentativas**:
  1. `a873b24` - `kept_textures: Mutex<HashMap<u32, Arc<HostTexture>>>` por slot
     e `retire()` fora do lock; `keep_with_view` descobre o dono pelo `View`.
  2. `7dddec2` (fork de AntoninMichelq, merge `d105484`) - liberar texturas e
     slots so depois das submissoes em voo.
  3. `eebcf07` - esperar trabalho assincrono em voo antes de sinalizar fences do
     titulo.
- **Correcao definitiva**: `a873b24` (mecanismo) + `7dddec2` (outra ponta).
- **Arquivos**: `crates/victory-gpu/src/descriptor/host.rs`, `texpool.rs`,
  `texture.rs`, `host.rs`, `shadow.rs`, `watch.rs`.
- **Evidencia** (`git show a873b24`):
  ```rust
  +    /// Owning `HostTexture` (and its `texpool::Slot` / `ID3D12Heap`) for any
  +    /// slot viewing a pooled texture, so the backing heap cannot be freed while
  +    /// a shader-visible descriptor slot still points into it.
  +    kept_textures: Mutex<HashMap<u32, Arc<crate::resource::HostTexture>>>,
  ```
- **Estado atual**: corrigido no caminho conhecido; o README ainda registra um
  driver timeout eventual, entao a classe nao esta provadamente extinta.
- **Confianca**: CONFIRMADA (mecanismo e correcao); exaustao da classe
  PROVAVEL.

### D-08 - Tempestade de faults de 2,5 GB por frame no `watch::guard`

- **Descricao / sintomas**: o frame gastava tempo protegendo/desprotegendo
  blocos; leituras de 2,5 GB por frame; "fault storm".
- **Causa raiz**: `VirtualProtect` falha em blocos nao mapeados (buracos na
  memoria do titulo) e o guard retentava e copiava esses blocos a cada
  verificacao, para sempre.
- **Commit que introduziu**: guarda por VEH herdada do upstream; regride com o
  volume de views do mundo.
- **Tentativas**: `2b6adbb` - `holes: BTreeSet<usize>` com os blocos que falharam,
  limpos por `title_memory_remapped`; `written` so e estendido para blocos
  realmente protegidos; + cap default de 240 FPS (`VICTORY_FPS`).
- **Correcao definitiva**: `2b6adbb`, refinada por `1c7231d` (view tornada
  read-only entre o mapeamento e o aviso do runtime e devolvida; bloco
  read-only sem guard conhecido e aberto no fault) e `4c1766c` (fault de escrita
  nao pode ser recusado numa corrida).
- **Arquivos**: `crates/victory-gpu/src/watch.rs`, `frame.rs`.
- **Evidencia** (`git show 2b6adbb`):
  ```diff
  +    /// Unmapped blocks (`VirtualProtect` failed) within guarded ranges;
  +    /// cleared by [`title_memory_remapped`] when the runtime maps them.
  +    holes: BTreeSet<usize>,
  ```
  e o assunto: "fix 2.5GB/frame fault storm".
- **Estado atual**: fechado.
- **Confianca**: CONFIRMADA.

### D-09 - Run de blocos guardados que cruza views falhava inteiro e deixava blocos read-only

- **Descricao / sintomas**: uma leitura do kernel para dentro de um bloco
  "guardado de novo" falhava e o titulo parava.
- **Causa raiz**: `VirtualProtect` de um run contiguo que atravessa duas views
  (o titulo mapeia uma view por pagina) falha por inteiro; os blocos ficavam
  `PAGE_READONLY` **sem estar registrados** como protegidos, entao ninguem os
  abria de volta.
- **Commit que introduziu**: a coalescencia de `VirtualProtect` em um run
  contiguo, introduzida em `a873b24` ("coalesce VirtualProtect";
  `git log -S"run_start" -- watch.rs` confirma que as variaveis do run nasceram
  ali).
- **Tentativas / correcao definitiva**: `e7e6c00` - quando o run falha, devolver
  **bloco a bloco** (`give(from,to)` com fallback de um `VirtualProtect` por
  bloco).
- **Arquivos**: `crates/victory-gpu/src/watch.rs`.
- **Evidencia** (`git show e7e6c00 -- watch.rs`): "One call cannot span two
  views: the title's page mappings are one view a page, and a run over several
  failed whole, leaving blocks read-only that were no longer noted guarded (a
  kernel read into them then failed and the title stopped)."
- **Estado atual**: fechado.
- **Confianca**: CONFIRMADA.

### D-10 - Escrita assincrona do kernel em bloco re-guardado

- **Descricao / sintomas**: o titulo para quando uma leitura assincrona do
  runtime cai num bloco que voltou a ser read-only.
- **Causa raiz**: o kernel nao faz fault para o nosso handler: ele **falha**. Uma
  leitura anunciada por `watch::writing(at, len)` pode pousar depois que a
  chamada retornou; ate la o bloco ja tinha sido protegido de novo.
- **Commit que introduziu**: modelo de guarda herdado (protecao imediata no fim
  da escrita).
- **Tentativas / correcao definitiva**: `579a8d9` - `pins: Vec<(from,to,until)>`
  com janela de 2 s: blocos em `[from,to)` ficam gravaveis, sao reportados como
  escritos em toda checagem e `bump_region_epochs` invalida os ranges que os
  contem; as outras views das mesmas paginas tambem sao abertas.
- **Arquivos**: `crates/victory-gpu/src/watch.rs`.
- **Evidencia** (`git show 579a8d9`):
  ```rust
  +    /// Blocks the kernel is about to write for the runtime ([`writing`]),
  +    /// from, to, and until when they are left writable: a read may be
  +    /// asynchronous and land after the call that asked for it returned, and
  +    /// the kernel does not fault into our handler, it fails.
  +    pins: Vec<(usize, usize, std::time::Instant)>,
  ```
- **Estado atual**: fechado (com custo: 2 s de "sempre escrito" por escrita
  anunciada).
- **Confianca**: CONFIRMADA.

### D-11 - Partes de veiculos em movimento ficavam para tras

- **Descricao / sintomas**: veiculos em movimento perdiam ou esticavam partes;
  quads de arvores distantes cruzando a tela sumiam.
- **Causa raiz**: a guarda de pagina so cobria a **view** pela qual o titulo
  escrevia; como a mesma pagina fisica aparece em varias views, uma escrita por
  outro endereco nao era vista e as copias do host nao eram atualizadas.
- **Commit que introduziu**: modelo "uma view guardada" herdado do upstream.
- **Tentativas**: `1c7231d` (view read-only entre mapeamento e aviso),
  `f2cc100` (views olhadas uma vez por run de blocos, paginas com uma view nao
  fazem lista - desempenho, nao correcao).
- **Correcao definitiva**: `ae90937` - toda view de uma pagina fisica guardada
  junto com ela; `VICTORY_ALWAYS_COPY` para diagnostico.
- **Arquivos**: `crates/victory-gpu/src/watch.rs`.
- **Evidencia**: README, "Vehicles in motion: whole (they lost and stretched
  parts until the copies of title memory saw writes through every view of a
  page)"; commit `ae90937` ("parts of moving vehicles were left behind").
- **Estado atual**: fechado para veiculos. O retorno dos quads de arvores
  distantes e atribuido ao mesmo periodo mas **a causa nao esta provada**
  (README: "cause not proven").
- **Confianca**: CONFIRMADA (veiculos); HIPOTESE (arvores).

### D-12 - Crash/GPU hang 120 frames dentro do mundo ao devolver tiles de copia

- **Descricao / sintomas**: GPU hang ou fault ~120 frames apos entrar no mundo;
  o host GPU era removido.
- **Causa raiz**: o sweep desmapeava e reusava tiles de 64 KiB que ninguem tinha
  pedido por 120 frames, inclusive tiles ainda referenciados por submissoes em
  voo.
- **Commit que introduziu**: sweep de tiles (arquitetura de `shadow`/`texpool`).
- **Tentativas**:
  1. `9956958` - desliga o sweep (`VICTORY_SWEEP_TILES=1` traz de volta):
     "1203 world frames without a fault, memory steady".
  2. `0a32e89` - reintroduz como `VICTORY_SWEEP_TILES=2` (so memoria que o
     titulo nao tem mais), ainda off por default.
  3. `e11fb06` - pergunta ao sistema por regiao e fora do lock dos tiles, uma vez
     por frame, e loga o que levou.
- **Correcao definitiva**: **SEM CORRECAO** (default e nao devolver: verify em
  `shadow.rs`, `sweeps()` -> `unwrap_or(0)`), com o custo de memoria crescendo.
- **Arquivos**: `crates/victory-gpu/src/shadow.rs`,
  `crates/victory-gpu/src/descriptor/host.rs`.
- **Evidencia**: mensagem de `9956958`; `shadow.rs`:
  ```rust
  std::env::var("VICTORY_SWEEP_TILES").ok().and_then(|v| v.parse().ok()).unwrap_or(0)
  ```
- **Estado atual**: aberto por escolha (seguranca > memoria). Alimenta D-13.
- **Confianca**: CONFIRMADA.

### D-13 - Consumo de memoria: 22 GB de RSS, 33 GB de commit, mais VRAM que a placa

- **Descricao / sintomas**: processo chega a ~22 GB de RAM no mundo e a ~33 GB
  commitados depois de 15 minutos dirigindo; VRAM estoura os 16 GB e as texturas
  sao reduzidas para 1024, 512, 256 px de lado.
- **Causa raiz**: soma de (a) copias de toda a memoria do titulo em tiles
  (`Shadow`), (b) pool de snapshots, (c) staging, (d) compilacoes infinitas que
  crescem GB/min (`7263339`: "A compile that never ends takes 8 GiB a minute"),
  (e) copias que nunca sao devolvidas (D-12), (f) texturas do host.
- **Commit que introduziu**: nao aplicavel (arquitetura).
- **Tentativas**:
  1. `e136a55` - orcamento de staging (`VICTORY_STAGING_BUDGET_MB`, 1 GiB) e
     pool de snapshots <= 512 MiB ("it could keep 8 snapshots of the world's
     174 MB heap, about 1.4 GB, for good").
  2. `5e72a8f` - log de onde vai o commit charge a cada 5 s.
  3. `f78d243`, `1343a0b`, `cf1f4d4`, `c4bb8d`-familia - merges do ramo
     `memory/world-load-commit` (reduzir commit durante o load).
  4. `687e7c9` - compartilhar uma copia ja feita em vez de copiar de novo ("a
     buffer inside another, the world's views of its 174 MB heap").
  5. `7263339` - o timeout de compilacao afrouxado troca memoria por tempo;
     `2aad398`/`df71546` trocam CPU por slots; `e136a55` limita.
- **Correcao definitiva**: **SEM CORRECAO**; aberto e declarado no README.
- **Arquivos**: `crates/victory-gpu/src/shadow.rs`, `texpool.rs`, `resource.rs`,
  `memstat.rs`, `memwait.rs`.
- **Evidencia**: README ("about 22 GB of RAM and more video memory than a 16 GB
  card has"; "the process commits about 33 GB after a quarter of an hour's
  drive").
- **Estado atual**: aberto; e o principal impedimento pratico junto com o fps.
- **Confianca**: CONFIRMADA (sintoma e orcamentos); a decomposicao exata do
  consumo e PROVAVEL.

### D-14 - Telefone in-game deixa o mundo sem luz

- **Descricao / sintomas**: com o telefone aberto, o mundo fica preto e o
  telefone aparece so como contorno.
- **Causa raiz**: NAO VERIFICADA. Nao ha commit, issue ou diagnostico no
  repositorio sobre o caso; o README apenas registra o fato.
- **Commit que introduziu**: NAO VERIFICADO.
- **Tentativas de correcao**: nenhuma registrada.
- **Correcao definitiva**: **SEM CORRECAO**.
- **Arquivos**: NAO VERIFICADO.
- **Evidencia**: README, tabela de estado: "In-game phone | **broken**: with the
  phone out the world is unlit (black) and the phone is an empty outline".
- **Estado atual**: aberto.
- **Confianca**: NAO VERIFICADA (o sintoma e declarado pelo autor; a causa nao).

### D-15 - Agua desaparece ate reiniciar depois de reconstruir o cache de shaders

- **Descricao / sintomas**: a primeira execucao depois de mudar os passes de
  limpeza (ou qualquer coisa que entre na chave do cache) reconstroi todos os
  pipelines e **a agua nao aparece**; a segunda execucao mostra agua.
- **Causa raiz**: NAO VERIFICADA. Hipotese: variantes de shader de agua cujo
  pipeline e criado tarde (workers de pipeline) e cujo draw e dropado ate o
  fim da sessao, ou `skipped_mips`/`streamed textures` ainda sem os mips que a
  view le.
- **Commit que introduziu**: PROVAVEL associacao com `ae75a66` (os passes de
  limpeza entram na chave do cache via `shader_cache::key`), que e exatamente
  quando o README passou a documentar o sintoma.
- **Tentativas**: nenhuma registrada.
- **Correcao definitiva**: **SEM CORRECAO**.
- **Arquivos**: `crates/victory-gpu/src/shader_cache.rs`,
  `crates/victory-gpu/src/pipeline/*` (suspeitos).
- **Evidencia**: README: "The first launch after such a change rebuilds every
  pipeline (minutes of black world) and **the water is missing until the game is
  started again**."
- **Estado atual**: aberto.
- **Confianca**: HIPOTESE (associacao); causa NAO VERIFICADA.

### D-16 - Apenas o material 0 iluminado (quadrados) e depois imagem branca estourada

- **Descricao / sintomas**: manchas quadradas sem luz onde o classificador de
  material nao escreveu; depois de corrigir, imagem final estourada em branco e
  ruidosa (runs w94/w95).
- **Causa raiz**: o classificador escreve numeros de material no plano de
  profundidade de uma textura depth16+stencil via UAV; o host recusava essas
  views, entao so o material 0 era iluminado. A correcao (twins) expos um
  problema de exposicao: "final image now overexposed".
- **Commit que introduziu**: modelo de depth sem twin (upstream).
- **Tentativas**:
  1. `e52106f` - depth twins: o compute escreve numa textura gemea, copiada para
     o depth depois de cada dispatch. Quadrados e rosa somem; imagem estoura.
  2. `8044d21` - twins off por default ("worse than without").
  3. `7eb9545` - twins **on** por default, apos os merges de shader do dia
     resolverem o branco: "no squares, people and rooms lit, exposure as the
     title sets it per shot".
- **Correcao definitiva**: `7eb9545` (default), com `e52106f` como mecanismo.
- **Arquivos**: `crates/victory-gpu/src/twin.rs`.
- **Evidencia** (`git show 7eb9545`): a mudanca e a inversao do default
  (`is_ok_and(|v| v == "1")` -> `!is_ok_and(|v| v == "0")`) e o comentario "The
  white picture of w94/w95 is gone with today's shader merges."
- **Estado atual**: fechado. Paredes, pisos e tetos de interior seguem pretos
  (luz so ray-traced) - isso e D-01.
- **Confianca**: CONFIRMADA (sequencia e correcao); qual merge de shader
  consertou o branco e HIPOTESE.

### D-17 - `VIEW_SLOTS` (2.294.784) acima do limite tier-2 do D3D12 (1.000.000): o padding nunca e criado

- **Descricao / sintomas**: o host tenta criar view heaps com
  `max(count + 1024, 2_294_784)` descritores "para que os indices dos shaders,
  que ficam abaixo de `VIEW_SLOTS`, caibam no heap". Em qualquer host tier-2 o
  `CreateDescriptorHeap` desse tamanho falha (limite 1.000.000, documentado pelo
  proprio fork em `docs/view-heap-limit.md`), cai no fallback
  `create(count + 1024)` e emite o warning "a view heap shorter than shaders
  keep their indices in" a cada heap.
- **Causa raiz**: dois numeros incompativeis no mesmo desenho: `emit::VIEW_SLOTS`
  = 2.294.784 e o teto real do D3D12; o padding so teria efeito num driver que
  reportasse mais de 2,29 M views.
- **Commit que introduziu**: `9956958` (funcao `view_slots`).
- **Tentativas**: o fallback (`or_else`) e a propria mitigacao; `3827767`
  (upstream, PR #21) remapeia tabelas que passam do heap.
- **Correcao definitiva**: **SEM CORRECAO** (o fallback evita o crash; a
  garantia anunciada no comentario nao vale).
- **Arquivos**: `crates/victory-gpu/src/descriptor/host.rs`,
  `crates/victory-shader/src/emit/mod.rs`, `docs/view-heap-limit.md`.
- **Evidencia**:
  ```rust
  fn view_slots(count: u32) -> u32 {
      let slots = count + crate::memory::SLOTS;
      if cfg!(test) { slots } else { slots.max(victory_shader::emit::VIEW_SLOTS) }
  }
  ```
  com `pub const VIEW_SLOTS: u32 = 2_294_784;` e o doc do fork dizendo que o
  teto e 1.000.000 views.
- **Estado atual**: latente; o warning e esperado em toda criacao de heap nao
  sampler.
- **Confianca**: CONFIRMADA quanto a contradicao de constantes (leitura de
  codigo + doc); consequencia em runtime NAO VERIFICADA (nao executado).

## E. Relacao de causa e efeito

**E-01 - O clamp de indiretos que virou 225 ms**
`8c26061` (Marvis) emula o clamp de `ExecuteIndirect` com um compute dispatch +
3 barreiras por draw -> `35b054e` torna a execucao indireta o default ->
9.300 dispatches e ~27.900 barreiras por frame, 190 de 225 ms de GPU ->
`1f7f7d2` e `6b867e6` tornam o clamp correto (count buffer, sem wrap) e mantem o
custo -> `ccab973` corta a lista no primeiro indireto e leva os clamps para uma
lista anterior -> 72 ms -> `918ab03` liga por default -> flicker -> `afca189`
isola -> `76387c2` (modo 7) elimina -> `684a9b5` tira 3 chamadas D3D12 por draw.

**E-02 - O magenta que nao era luz**
Rays nao rodam -> `CS_ScreenProbes_TemporalResolve` le imagem nunca escrita ->
saida = lixo (magenta, blocos) -> `0603123` aplica **piso aditivo** sobre o lixo
("raised, not added") -> continua errado -> `4d63a09` desliga por default ->
`4f45ab8` **substitui** a saida por constante -> `e48eb83`/`705a2d0` fazem a
constante seguir o ceu -> `4f62c2a` default.

**E-03 - A linha na aresta do triangulo**
Derivada calculada com `quad_perm` -> leitura de wave devolve 0 para helper
lanes na AMD -> na borda do triangulo a derivada e o valor inteiro -> textura no
menor LOD -> linha em toda aresta -> `f520772` corrige um vizinho
(`v_readfirstlane` de helper) -> `03629f3` faz toda leitura de quad em pixel
shader passar por `quad_pick`.

**E-04 - Slots dinamicos: de colapso a degradacao**
Tabelas "fresh" copiadas por lista + tabelas grandes copiadas por comprimento ->
esgotamento dos 524.288 slots em 6 min (0,3 fps) -> `df71546` (uma copia pelo
maior comprimento, pico 369.107) -> `788ea46` (busca falha memorizada, <= 8 por
frame) -> `b41f3f3`/`127830e` (hash rapido) -> ainda faltam slots nas cenas mais
pesadas (README).

**E-05 - O timeout que matou quatro pipelines**
Compilador AMD nao termina ray tracing -> `1b0c6b5`/`2655edb` deixam de fora
pipelines lentos com criterio de 20 s / 3 GiB -> quatro pipelines que o driver
faz em segundos sao excluidos **para sempre** durante o load do mundo ->
`7263339` afrouxa para 60 s / 8 GiB e documenta o caso -> custo: load mais longo
e ate 8 GiB de crescimento por compilacao infinita.

**E-06 - Sweep de tiles: hang -> memoria**
Sweep devolve tiles de 64 KiB sem uso por 120 frames -> GPU hang/fault 120
frames dentro do mundo -> `9956958` desliga (1203 frames sem fault) -> copias
crescem ->
`0a32e89` reintroduz so para memoria que o titulo nao tem mais (`=2`), ainda off
por default -> `e11fb06` consulta o sistema por regiao, fora do lock.

**E-07 - Submissao assincrona: tres tentativas**
`2bfe95a` introduz submissao assincrona -> instabilidade ->
`a049b48` torna opt-in -> `bdecda3` reimplementa (off por default) -> `9b89ec2`
descobre que o buffer de captura do present era liberado com a copia pendente
(device removed no primeiro frame) e faz so as submissoes **do titulo**
retornarem antes -> `a59f3e5` liga por default: 200-238 -> 150-186 ms/frame.

**E-08 - A pagina fisica com varias views**
Uma view guardada por pagina -> escrita por outro endereco nao vista -> copias
do host desatualizadas -> partes de veiculos para tras -> `ae90937` guarda
todas as views da pagina fisica -> veiculos inteiros.

**E-09 - O material que so existia no depth**
Classificador escreve material no depth via UAV -> host recusa -> so material 0
iluminado -> quadrados -> `e52106f` twins -> branco estourado -> `8044d21` off
-> merges de shader corrigem exposicao -> `7eb9545` on.

## F. Correcoes importantes

### F-01 - Substituir a luz indireta em vez de elevar o que esta la (`4f45ab8`)

- **O que estava errado**: o passe de resolve de probes tem saida invalida sem
  rays; a primeira correcao somava um piso a esse valor, o que apenas garantia
  que o lixo fosse claro o bastante para ser visto.
- **Por que funciona**: a imagem R9G9B9E5 e **historia** do proximo frame; basta
  escrever o valor desejado (constante tingida) para que a iluminacao do titulo o
  aplique com as cores das superficies. Nao ha dependencia dos dados do passe.
- **Dependencias**: nome do programa (`CS_ScreenProbes_TemporalResolve`), root
  parameter e indice na tabela (`LIGHT = (1, 176)`), formato de empacotamento
  R9G9B9E5 implementado em HLSL no proprio modulo, `D3DCompile` em runtime,
  `shader_cache` para nao recompilar.
- **Reutilizavel?** Sim, como **padrao**: identificar por prefixo de nome o passe
  cuja entrada nunca e produzida e substituir sua saida; manter o valor num
  arquivo recarregavel a quente para calibrar. Especifico do titulo nos numeros.
- **Efeitos colaterais**: sem oclusao; interiors sem RT continuam pretos;
  reflexos precisaram de contencao propria (`c3bd561`/`REFLECTED =
  CS_UpdateHistory`).

### F-02 - Leituras de quad em pixel shader (`03629f3`)

- **O que estava errado**: derivadas por `quad_perm` liam lanes comuns e
  recebiam 0 de helper lanes.
- **Por que funciona**: `quad_pick` le explicitamente os quatro pixels do quad
  (via leitura de quad do hardware/`QuadReadLaneAt`), que **incluem** helpers;
  helper lane e justamente quem carrega a informacao nas bordas.
- **Dependencias**: nenhuma alem de `self.s.pixel` (estagio) e do caminho
  `quad_pick` ja existente para mesh shaders.
- **Reutilizavel?** Sim, e a mudanca mais barata e mais geral do fork: 17 linhas
  em `lower.rs`. Upstream faz isso apenas depois de `discard`; a AMD exige sempre
  em pixel shader.
- **Risco**: em hardware onde leitura de quad de helper lane nao e barata, ha
  custo; em NVIDIA o comportamento ja era outro (o fork nao testou NVIDIA).

### F-03 - Hoisting dos clamps de indiretos (`ccab973`, `918ab03`, `76387c2`, `684a9b5`)

- **O que estava errado**: trabalho de emulacao (clamp) intercalado com o
  trabalho real (draw), um dispatch + 3 barreiras por draw.
- **Por que funciona**: os argumentos de todos os draws de uma corrida sao
  clampados **antes** de qualquer draw da corrida, em buffers lidos por offset;
  a lista e cortada no primeiro indireto (`Hoist { clamps, into, writes }`) e
  invalidada se um dispatch/copia posterior escrever os argumentos
  (`writes` comparado). Modo 7 executa os clamps em sequencia para eliminar a
  corrida que causava flicker.
- **Dependencias**: `indirect.rs` (`bound`, `record_clamp`, `hoist_mode`),
  `draw.rs` (`State::hoist`/`hoists_into`/`end_hoist`), `list.rs`
  (`keep_until_run`, `count_write`), `host.rs` (tempo de GPU por comando).
- **Reutilizavel?** Sim: e o padrao classico "emulacao por comando -> lote".
  Exige que a emulacao seja **pura** e que a invalidadacao por escrita seja
  correta - e onde o flicker nasceu.
- **Risco**: qualquer escrita nos argumentos entre a lista de clamps e a lista
  de draws invalida o lote; o teste de `writes` e a unica protecao.

### F-04 - Contagem de elementos pela tabela do host (`1503444`)

- **O que estava errado**: `GetDimensions`/size query de structured buffer com
  certos strides **crasha o compilador da AMD**.
- **Por que funciona**: o host ja sabe o `NumElements` da view do titulo
  (`view_count`); publica strides e contagens numa tabela de constantes
  (`COUNTS_AT = OWN_SLOTS`) que os shaders leem em vez de perguntar ao recurso.
- **Dependencias**: `victory_shader::emit::OWN_SLOTS` (contrato entre crates),
  `StrideTable` alocada com `COUNTS_AT + len` dwords, `set_stride_count` em todo
  caminho que cria view.
- **Reutilizavel?** Sim, sempre que um driver crashar em reflection de recurso:
  mover a informacao para uma tabela alimentada pelo host.
- **Risco**: tabela e visao do host; se um shader criar a view dinamicamente
  sem passar por `keep_with_view`, a contagem fica 0 (o proprio fork tem
  `fa042f6` "Raw-viewed structured buffers keep their size query" para o caso
  oposto).

### F-05 - Passes de limpeza no bitcode antes do driver (`ae75a66` + `opt.rs`)

- **O que estava errado**: o lowering emite uma instrucao por instrucao do
  console, sem nada entre ele e o compilador do driver; programas enormes
  (upscaler com 10.000 instrucoes, ray tracing) faziam o compilador nao terminar.
- **Por que funciona**: cinco passes **exatos** (DCE, fold/identidades, CSE de
  instrucoes sem efeito, fatos de branch, CSE de leituras dentro do bloco), cada
  um preservando valores e efeitos para toda entrada; resultado: o compute
  `39f0e9aeb156a61c` compila em 36 s onde antes nunca terminava.
- **Dependencias**: `emit::shader_opt()` (`VICTORY_SHADER_OPT`, mascara de bits)
  e a **chave do cache de shader** (mudar os passes invalida o cache - e a origem
  provavel de D-15).
- **Reutilizavel?** Sim: 2.155 linhas de `opt.rs` vieram do fork `ba8697aa`
  (GPL-3.0-or-later) e foram adaptadas a outro modelo de modulo (arrays locais,
  doubles, joins de registrador que ninguem escreve - que **nao** podem virar
  `undef`, senao o validador recusa).
- **Risco**: os passes sao exatos por contrato; qualquer pass novo tem de ser
  provado, e o validador da Microsoft foi usado como rede (25.623 shaders do
  pack passam).

### F-06 - Guarda de memoria por VEH endurecida (`2b6adbb`, `e7e6c00`, `579a8d9`, `ae90937`)

- **O que estava errado**: tres falhas independentes no mesmo mecanismo
  (buracos nao mapeados, runs que cruzam views, escritas assincronas do kernel) e
  uma premissa errada (uma view por pagina).
- **Por que funciona**: o estado do guard passa a ser explicito e completo -
  `holes` (nao mapeado), `protected`/`clean` (read-only e sem escrita), `pins`
  (kernel pode escrever por 2 s), `aliased` (views da mesma pagina), epocas por
  regiao para invalidar copias.
- **Dependencias**: VEH, `VirtualProtect`, `VirtualQuery`, mapeamento de
  secoes do `xmem`, `shadow` (copias) e `resource` (views).
- **Reutilizavel?** Sim para qualquer emulacao de memoria de console com copia
  no host; e a peca mais delicada do fork.
- **Risco**: cada falha dessa classe se manifesta como "o titulo para" sem
  stack util; o fork gastou ~6 commits so nisso em 24 h.

### F-07 - Mapear a "stretch" do titulo num endereco proprio (`0434b3a`)

- **O que estava errado**: o titulo mapeia heaps de streaming em views de 64 KiB,
  cada uma sua alocacao; o host nao consegue abrir isso como um heap unico, e
  cada bindless custava copia.
- **Por que funciona**: as paginas sao **contiguas na secao**; mapear a mesma
  secao de novo no nosso espaco de enderecos cria uma janela contigua e o heap do
  host e aberto ali - zero copia. 33 janelas por frame: copy-back 36 -> 12 ms,
  frame 227-266 -> 202-216 ms.
- **Dependencias**: conhecimento do alocador do `xmem` (views de 64 KiB), secao
  subjacente, `VICTORY_STRETCH=0` como escape.
- **Reutilizavel?** Sim para qualquer titulo que fracione heaps; exige que a
  contiguidade seja **verificada**, nao presumida (`78028ad` mostra o custo de
  errar: leitura 2,3 GB alem do buffer, page fault).

### F-08 - Contencao do compilador do driver (`95e935d`, `0d2f6a6`, `7263339`)

- **O que estava errado**: um pipeline problematico derrubava a execucao inteira
  (crash do compilador) ou a travava por minutos (compilacao sem fim, 16 GB).
- **Por que funciona**: (a) VEH dedicado que registra o nome do pipeline em
  `crashed.txt` e o pula nas proximas execucoes; (b) detector de compilacao sem
  fim por tempo **e** por crescimento de memoria, com `SuspendThread` apenas
  quando o RIP esta no modulo do driver (nunca em ntdll/ucrtbase, cujos locks a
  thread suspensa poderia segurar); (c) workers perdidos sao substituidos e
  ninguem espera por eles.
- **Dependencias**: nomes estaveis de pipeline (hash do codigo traduzido + root
  signature + estado), `pipelines/crashed.txt` no cache, `GetThreadContext`,
  `GetModuleHandleExW`.
- **Reutilizavel?** Sim, e uma das partes mais uteis para qualquer tradutor
  GFX10->DXIL em AMD.
- **Risco**: `crashed.txt` e estado persistente; um criterio de timeout mal
  calibrado (20 s/3 GiB) exclui pipelines legitimos **para sempre** (foi o que
  `7263339` corrigiu).

### F-09 - Reflexos contidos na luz ambiente (`c3bd561`, `REFLECTED = CS_UpdateHistory`)

- **O que estava errado**: sem rays, todo material glossy refletia o ceu aberto,
  inclusive em interiores; pisos de madeira, balcao e vidros ganhavam um brilho
  branco 5-10x mais claro que a sala.
- **Por que funciona**: depois do ultimo passe que escreve reflexos
  (`CS_UpdateHistory`), o valor e limitado a uma fracao da luz ambiente
  (`VICTORY_AMBIENT_REFLECT`, 0.5 default), e um decimo numero no arquivo de
  ambiente da um piso para reflexos escuros (`LEAST`).
- **Reutilizavel?** Como padrao de "conter o que o RT deveria ocluir": identificar
  o ultimo produtor do buffer e aplicar um limite fisico plausivel.

## G. Problemas ainda existentes

| # | Problema | Evidencia | Confianca |
|---|---|---|---|
| G-01 | Telefone in-game deixa o mundo preto (D-14) | README, tabela de estado | CONFIRMADA (sintoma) |
| G-02 | Agua ausente ate reiniciar depois de reconstruir o cache de shaders (D-15) | README | CONFIRMADA (sintoma) / causa NAO VERIFICADA |
| G-03 | 22 GB de RAM e ~33 GB commitados; VRAM acima de 16 GB; texturas limitadas a 1024/512/256 px | README | CONFIRMADA |
| G-04 | Slots dinamicos de descritor insuficientes nas cenas mais pesadas -> draws dropados | README; D-05 | CONFIRMADA |
| G-05 | Sem luz RT: luz indireta igual em todo o frame, sem oclusao; paredes/pisos/tetos de interior pretos (611/611 dispatches de resolve vazios) | `7eb9545`; README | CONFIRMADA |
| G-06 | 266 programas do pack nao traduzem (195 usam enderecos de console em registradores escalares; 71 mesh shaders passam de 32 KiB de saida) -> draws dropados | README | CONFIRMADA (por declaracao do autor) |
| G-07 | Um compute crasha o compilador da AMD na hora (`f16876929fa64f9f`) | `ae75a66`; README | CONFIRMADA |
| G-08 | Driver timeout do Windows aconteceu uma vez; "expect it" | README | CONFIRMADA (sintoma) |
| G-09 | `VIEW_SLOTS` inalcancavel no D3D12 tier-2: padding nunca criado, warning por heap (D-17) | codigo + `docs/view-heap-limit.md` | CONFIRMADA (codigo) |
| G-10 | Dois testes falham na maquina do autor antes e depois das mudancas: `list::tests::lists_are_cut_at_waits_that_follow_work` (victory-gpu) e cinco de `xmem` (victory-hle) | README | CONFIRMADA (declarado) |
| G-11 | Windows 10 nao passa do start-up sem runtime D3D12 mais novo (issue #9 / PR #44, nao mergeado nem testado aqui) | README | NAO VERIFICADA (issue externa) |
| G-12 | `crates/victory-shader/src/emit/lower.rs` acima do limite de tamanho do `cargo xtask policy` | README | CONFIRMADA |
| G-13 | Muitas mudancas marcadas `shortcut:` no codigo aproximam o console (luz, reflexos, depth twins, memory-by-address) | grep `shortcut:` no repo | CONFIRMADA |
| G-14 | O sweep de tiles esta desligado por default: copias crescem sem limite (D-12) | `shadow.rs` `sweeps()` default 0 | CONFIRMADA |

## H. Regressoes

### H-01 - Revert em bloco de 26 PRs do upstream (`a62995f`)

Removeu audio/XMA, saves, XAsync/XTaskQueue, checagens de path e PE, loader
hardening e o relatorio de headroom de commit, com a justificativa de reduzir
superficie de risco depois do #234. Efeito: o fork ficou **abaixo** do upstream
em robustez de loader e saves por varias horas, e teve de reabsorver parte disso
depois (`6269b57` VEH/PE/TLS/oleaut32, `9e44f38`, `db89843`, `89e1031`,
`56d5032` etc. via merges de main). Risco: um revert em bloco esconde conflitos
semanticos; o proprio autor registra que "cada um e mais uma coisa que pode
quebrar".
Confianca: CONFIRMADA (diff de 31 arquivos, -4.888 linhas).

### H-02 - Hoist de clamps ligado por default causou flicker (`918ab03` -> `76387c2`)

Mudanca de 3 linhas em `indirect.rs` mudou o default e introduziu flicker de
objetos mesh; precisou de 4 commits (`afca189`, `76387c2` e os modos 2-7) para
voltar atras parcialmente (modo 7 e mais lento que o modo 1).
Confianca: CONFIRMADA.

### H-03 - Default de luz ambiente: 0.6 -> 0 -> 0.6 (`0603123` -> `4d63a09` -> `4f45ab8`)

A primeira correcao de D-01 ligou 0.6 por default; o autor considerou o
resultado errado ("the user finds the added light wrong") e desligou; depois a
abordagem mudou e o default voltou. Duas mudancas de comportamento visivel em
24 h.
Confianca: CONFIRMADA.

### H-04 - Timeout de compilacao apertado demais excluiu pipelines legitimos (`95e935d`/`2655edb` -> `7263339`)

O criterio de 20 s / 3 GiB marcou quatro pipelines como "endless" **durante o
load do mundo** (quando o processo legitimamente cresce GB e todos os nucleos
estao ocupados) e os deixou de fora permanentemente, escrevendo em
`pipelines/crashed.txt`. O fix following afrouxa para 60 s / 8 GiB e o README
manda apagar o arquivo de cache antigo - sinal de que o estado persistente
envenenado e um problema real.
Confianca: CONFIRMADA.

### H-05 - Sweep de tiles: correcao que removeu funcionalidade (`9956958`)

O fix do GPU hang desligou o sweep; a memoria das copias passou a crescer
(D-13). Reintroduzido como opcional (`0a32e89` `=2`), mas o default continua
"nao devolver". Regressao de recurso disfarcada de fix.
Confianca: CONFIRMADA.

### H-06 - Submissao assincrona: introduzir, desligar, reimplementar, religar

`2bfe95a` (submissao assincrona) -> `a049b48` (opt-in por estabilidade do
driver) -> `bdecda3` (reimplementacao off) -> `9b89ec2` (o buffer de captura do
present era liberado com copia pendente: **device removed no primeiro frame**) ->
`a59f3e5` (default). A regressao do `9b89ec2` e o caso classico de posse de
recurso num caminho assincrono.
Confianca: CONFIRMADA.

### H-07 - `view_slots` com padding inalcancavel (`9956958`, D-17)

Junto com o fix de indices ficou um `max(count+1024, 2_294_784)` que nenhum host
tier-2 consegue criar; o fallback mascara, mas o warning "a view heap shorter
than shaders keep their indices in" passa a ser ruido permanente e a garantia do
comentario e falsa.
Confianca: CONFIRMADA (codigo); impacto NAO VERIFICADO.

### H-08 - `a873b24` coalesceu `VirtualProtect` e criou D-09

A otimizacao (um `VirtualProtect` por run contiguo) era correta em teoria, mas
runs que cruzam views falham por inteiro; o fork precisou de `e7e6c00` para
devolver bloco a bloco. Cluster de causa e efeito: otimizacao de syscall ->
premissa de contiguidade invalida -> titulo para.
Confianca: CONFIRMADA.

### H-09 - Depth twins: ligar, desligar, religar (`e52106f` -> `8044d21` -> `7eb9545`)

A correcao dos quadrados expos exposicao estourada; o default foi revertido por
"pior de olhar" e so voltou quando merges de shader resolveram o branco. Nao e
bem uma regressao de codigo, mas de **default**: por ~1 dia o mundo ficou com
quadrados sem luz porque o fix estava desligado.
Confianca: CONFIRMADA.

## I. Conhecimento reutilizavel

Prioridade para o que e **exclusivo deste fork** (nao existe no upstream) e
aproveitavel num repositorio unificado.

### I-01 - Substituicao do passe de luz indireta (EXCLUSIVO)

| Campo | Conteudo |
|---|---|
| Problema resolvido | RT indireto ausente -> saida invalida do resolve -> magenta/blocos/preto |
| Implementacao | hook por prefixo de nome do programa; reescreve a tabela de descritores; compute HLSL proprio (`D3DCompile` em runtime) que escreve constante tingida em R9G9B9E5; sombreamento por profundidade; nivel seguindo o ceu (`gain*mean^power+least`); piso de reflexos |
| Origem | este fork, modulo novo |
| Commits | `0603123`, `4f45ab8`, `cd9f78c`, `c3bd561`, `e48eb83`, `705a2d0`, `4f62c2a` |
| Arquivos | `crates/victory-gpu/src/ambient.rs` (592 linhas), `draw.rs`, `shader_cache.rs`, `victory-shader/src/native.rs` |
| Dependencias | nome/indice do programa no titulo; `HostList` para emitir o compute; cache de shader |
| Conflitos | qualquer outro fork que tambem reescreva `CS_ScreenProbes_*`; `VICTORY_AMBIENT*` pode colidir com `VIToriaBr`/`AvatarManz`, que tambem tem `ambient.rs` |
| Riscos | constante global sem oclusao; nao substitui RT; nomes e offsets sao do titulo |
| Recomendacao | portar como modulo isolado com a lista de programas/offsets em tabela de configuracao, nao hardcoded; manter o arquivo de ajuste a quente |

### I-02 - Leituras de quad em pixel shader (EXCLUSIVO, 17 linhas)

Problema: derivadas por leitura de lane devolvem 0 de helper lanes na AMD.
Implementacao: `helpers_read(quad) = quad && self.s.pixel` e uso de `quad_pick`
em `deriv`/`ds_swizzle`-like. Commit `03629f3`; arquivo
`victory-shader/src/emit/lower.rs`.
Dependencias: `quad_pick` existente. Conflitos: qualquer fork que reescreva
`lower.rs` (todos os cinco divergem nesse arquivo). Riscos: custo de quad reads
em hardware onde helper lane nao e barato; upstream so faz depois de `discard`.
Recomendacao: **portar primeiro**, e o melhor custo/beneficio do repositorio.

### I-03 - Hoisting de emulacao por comando (EXCLUSIVO no desenho, upstream na ideia)

Problema: emulacao de `ExecuteIndirect` (clamp) custando um dispatch + 3
barreiras por draw. Implementacao: `Hoist { clamps, into, writes }`, lista
cortada no primeiro indireto, buffers lidos por offset, invalidacao por
`writes`, modos 0-7 para A/B, modo 7 (sequencial) como default por causa do
flicker. Commits `ccab973`, `afca189`, `918ab03`, `76387c2`, `684a9b5`.
Arquivos `draw.rs`, `indirect.rs`, `list.rs`, `host.rs`.
Dependencias: `set_budget`/`indirect_scratch` por lista; `keep_until_run`.
Conflitos: PR #242 upstream (`1f7f7d2`) reescreve o mesmo `record_clamp`.
Riscos: invalidacao incompleta = descritores/argumentos errados (flicker).
Recomendacao: portar com o teste de invalidacao explicito.

### I-04 - Contagem de elementos pela tabela do host (EXCLUSIVO)

Problema: size query crasha o compilador AMD. Implementacao: `COUNTS_AT =
OWN_SLOTS`, `StrideTable` dimensionada com `COUNTS_AT + len`, `view_count()`
para SRV/UAV buffer, leitura pelos shaders. Commit `1503444` (+ `1ebf29d`,
`fa042f6`). Contrato entre `victory-shader` e `victory-gpu` - portar os dois
lados juntos. Riscos: manter contagem sincronizada em todo caminho de criacao
de view.

### I-05 - Passes de limpeza de bitcode (de TERCEIRO, adaptado aqui)

Origem: fork `ba8697aa` (GPL-3.0-or-later), trazido em `ae75a66` como
`victory-shader/src/emit/bitcode/opt.rs` (2.155 linhas + testes). Cinco passes
exatos + trace (`VICTORY_SHADER_OPT_TRACE`). Ganho medido: compute
`39f0e9aeb156a61c` compila em 36 s (antes nunca terminava). Dependencias: modelo
de modulo (`Op`, `Attr`, `for_each_operand`, `successors`) e a chave do cache.
Conflitos: `DexNom` e `VItoriaBr` tambem tem `bitcode/opt.rs` (mesma origem,
adaptacoes diferentes) - unificar exige escolher **um** modelo de modulo.
Riscos: um pass que transforme join em `undef` faz o validador recusar
programas (aconteceu com seis programas do mundo, corrigido no proprio commit).

### I-06 - Guarda de memoria por VEH completa (EXCLUSIVO na completude)

Problema: copia no host de memoria de console escrita por CPU e por GPU.
Implementacao: `Guards { protected, clean, holes, aliased, pins }` + epocas por
regiao + `writing()` com pins de 2 s + guarda de todas as views de uma pagina
fisica + devolucao bloco a bloco. Commits `2b6adbb`, `a873b24`, `1c7231d`,
`e7e6c00`, `579a8d9`, `ae90937`, `f2cc100`, `4c1766c`. Arquivo `watch.rs`.
Riscos: e a area com mais modos de falha (o titulo simplesmente para); qualquer
port precisa dos testes de regiao/alias/pin.

### I-07 - Stretch mapping para heaps fragmentados (EXCLUSIVO)

Problema: views de 64 KiB, uma alocacao cada, nao abrem como heap unico.
Implementacao: mapear a mesma secao de novo num endereco proprio e abrir o heap
ali; 36 -> 12 ms de copy-back. Commit `0434b3a`; `host.rs`/`memory.rs`.
Riscos: presumir contiguidade sem verificar leva a leitura fora do buffer
(`78028ad`).

### I-08 - Contencao de compilacao do driver (EXCLUSIVO na combinacao)

`pipelines/crashed.txt` + VEH + detector de compilacao sem fim (tempo **e**
memoria) + `SuspendThread` so no modulo do driver + substituicao de workers +
workers proprios para screen passes (`18e57db`). Commits `1b0c6b5`, `2655edb`,
`95e935d`, `0d2f6a6`, `7263339`, `18e57db`. Riscos: criterio de timeout errado
exclui pipelines legitimos (H-04).

### I-09 - Diagnostico que pagou por si

- `VICTORY_GPU_TIMING` (tempo de GPU por tipo de comando e por pipeline) foi o
  que revelou os 225 ms (`ccab973`).
- Hash DXIL por shader, o mesmo que o **Radeon GPU Detective** usa para nomear um
  shader em voo (README).
- `VICTORY_SHADER_PATCH` (muda uma palavra do codigo da maquina antes de
  traduzir) e `VICTORY_SHADER_SWAP` (troca containers por arquivo) - permitem
  isolar um shader culpado sem rebuild.
- `VICTORY_SHADER_OPT_TRACE` (quais chamadas o DCE descarta).
- `placed_census` (`c28201f`): quantas texturas colocadas existem, quantos bytes
  do titulo cobrem, quantos enderecos tem varias, quantas estao sobre memoria que
  o titulo nao tem mais.
- `VICTORY_HOIST_FILE`, `VICTORY_AMBIENT_FILE`: ajuste ao vivo de modulo em jogo.
Recomendacao: portar o conjunto inteiro para o repo unificado; e o que permitiu
achar em uma hora o que contadores proprios nao acharam em um dia.

### I-10 - Descartado/nao recomendado

- `RwLock` na tabela de views (`ff920f2`): correta, mas **sem ganho** de frame
  time - o gargalo e a thread que submete; nao portar esperando desempenho.
- Os oito PRs de romainhedouin (`13d6b73`, `ed273ef`, `684a9b5`, `f53ae3d`,
  `58655f9`, `0145ca7`, `a1ef2dd`, `127830e`): "None of the eight moved the frame
  time measurably here; nothing broke" - portar por higiene, nao por fps.
- Thin LTO (`a23d1b4`): sem medicao de ganho no documento.

## J. Conclusao tecnica

1. **O que este fork e, tecnicamente**: nao e "o VIctory afinado para AMD". E um
   **laboratorio de bring-up de um titulo real num unico sistema AMD**, feito em
   ~48 horas de trabalho intenso (148 commits proprios + 189 de agente),
   construido sobre um upstream re-commitado, com uma campanha de integracao de
   ~90 PRs de terceiros e uma segunda campanha de diagnostico por captura (PIX,
   Radeon GPU Detective).

2. **As tres conquistas que o ranking do PortWatch resume estao confirmadas no
   codigo**:
   - substituto da luz indireta RT (`4f45ab8` + `ambient.rs`): troca de "elevar
     o lixo" por "substituir o lixo"; o magenta e os blocos de cor sao
     consequencia direta de ler uma imagem nunca escrita;
   - arestas de triangulo (`03629f3`): helper lanes tem de ser lidas por quad
     reads em pixel shader na AMD; 17 linhas;
   - 225 ms -> ~72-80 ms (`ccab973` + `918ab03` + `76387c2` + `684a9b5`):
     emulacao de `ExecuteIndirect` por comando -> por lote.

3. **O padrao de engenharia dominante e "medir, isolar, conter"**: quase nenhum
   defeito foi resolvido por insight arquitetural; foram resolvidos por
   instrumentacao (GPU timing, PIX, censo, dumps) seguida de uma mudanca pequena
   e verificada. Os melhores exemplos sao `03629f3` (17 linhas) e `ccab973`
   (400 linhas).

4. **O custo estrutural e alto e esta declarado**: 22-33 GB de memoria, VRAM
   acima da placa, 266 programas sem traducao, telefone quebrado, sem RT, 5-8
   fps. O fork e uma **prova de viabilidade**, nao um port jogavel. A afirmacao
   de "roda missoes a ~4 fps" nao tem suporte no repositorio (README fala de
   mundo livre e 5-8 fps); trata-se de alegacao externa, NAO VERIFICADA aqui.

5. **A divida mais perigosa nao e o que falta, e o que ficou ligado por
   default**: sweep de tiles desligado (memoria cresce), `crashed.txt`
   persistente (pipeline bom excluido para sempre), `VIEW_SLOTS` inalcancavel,
   hoist em modo 7 (mais lento que o modo 1, escolhido por corretude). Um
   repositorio unificado herda esses defaults e herda os problemas junto.

6. **Veredito de arqueologia**: o historico e legivel e honesto - mensagens
   longas, numeros medidos, "not verified" explicito em varios commits
   (`d852b24`, `3a839d4`, `95e935d`), nomes de run citados (`a2`, `w94`, `v6`,
   `mem1`, `pr1`, `ciel3`, `opt1b`). Isso torna este fork a melhor fonte
   disponivel sobre **como fazer um GFX10->DXIL rodar num driver AMD real**.

## K. Indice de commits

751 commits, ordem cronologica inversa (`git log --format="%h %ad %s" --date=short`).

```
4f62c2a 2026-11-25 The ambient light follows the sky by default
b76a71b 2026-11-25 README: the sky following, the clean-up passes, what is refused now
6986a57 2026-11-25 The program that activates the screen probes' rays is left out without rays
7263339 2026-11-25 A compile is not held endless as soon; a trace of what the clean-up drops
ae75a66 2026-11-25 Clean-up passes over translated shaders, from the fork at ba8697aa
e48eb83 2026-11-25 The ambient light follows the sky dome only, and holds it
5d7bd16 2026-11-25 A placed texture's view may hold its detail back with a least level of detail
f1ba837 2026-11-25 The texture census also tells what the pool and the host hold
c28201f 2026-11-25 Shaders of a refused pipeline are saved; a census of placed textures
705a2d0 2026-11-25 The ambient light follows the sky; a write alone does not rebuild the plan
000ffdd 2026-11-25 The bindless plan looks at direct slots only when a window died
c3bd561 2026-11-25 A least light for reflections, set from the ambient file
50a9fa4 2026-11-25 Show the frame rate and frame time in the picture
a59f3e5 2026-11-25 Title submissions return before the host GPU is done by default; README with today's measurements
ff920f2 2026-11-25 The views table is a read-write lock: every list recorder reads it for every descriptor it binds
14545fa 2026-11-25 Merge pull request #96 from Key-rs: thin LTO and one codegen unit for the release profile
0434b3a 2026-11-25 Title memory shown page by page is used in place through a view of ours on the same stretch of its section
127830e 2026-11-25 Merge pull request #34 from romainhedouin: A table made afresh is looked up by a quick hash and its record words are compared, instead of a SipHash over them (a collision now costs a copy, not wrong descriptors)
a1ef2dd 2026-11-25 Merge pull request #33 from romainhedouin: A draw keeps its root constants on the stack, its fresh-table slots in a bit set and the tables it made in a buffer the state reuses, so a draw no longer allocates for them (up to 2 + 1 + 1 allocations per draw)
0145ca7 2026-11-25 Merge pull request #24 from romainhedouin: A root constant buffer not 256-byte aligned is copied into list scratch once for the draws of a list that bind it, until a title copy or run intervenes (up to 64 KiB less copied per repeat)
58655f9 2026-11-25 Merge pull request #29 from romainhedouin: A range shaders write is declared once until the title next writes memory, and the submission finds the unique ranges with a set, so a list of thousands of draws no longer scans and re-snapshots the same ranges per step
f53ae3d 2026-11-25 Merge pull request #30 from romainhedouin: Buffer-to-texture copies take their staging bytes from the list's pooled scratch chunks instead of creating and mapping a committed upload buffer each, which is much cheaper
684a9b5 2026-11-25 Merge pull request #28 from romainhedouin: Clamps hoisted into one list bind their root signature, pipeline and budget once, not before every clamp (3 fewer D3D12 calls per indirect draw)
ed273ef 2026-11-25 Merge pull request #25 from romainhedouin: A pipeline a list binds again is not named, queried and kept again (one lookup by address instead of two QueryInterface calls, three locks and a name search per switch)
13d6b73 2026-11-25 Merge pull request #27 from romainhedouin: The host heap behind a descriptor heap is returned without taking the pending lock once it is made, since nothing is queued after that
1edd766 2026-11-25 Root views into sparse copies reach to the end of the title's buffer (64 MiB at most); sparse copies are logged with their host address
40b8b9b 2026-11-25 A title buffer registered again at the same place retires its old copy instead of dropping it at once
9ff6beb 2026-11-25 README: reflections, lifetimes, the ray passes' state after the runs with Radeon GPU Detective
44dcd43 2026-11-25 Reflections held to the ambient light; copies and retired objects outlive the lists that name them
01aadd1 2026-11-25 README: upstream's status section left out (it named upstream's test machine, an RX 6700 XT, and a warning that is upstream's), Windows 10 as reported
f870d80 2026-11-25 README: pull requests #45 and #18 merged, the ambient light's shading and tuning, what the hours of the day need, the ray passes' state
a23d1b4 2026-11-25 build: thin LTO and one codegen unit for the release profile
cd9f78c 2026-11-25 Ambient light: shaded in creases and under objects from the scene's depth (VICTORY_AMBIENT_SHADE), tuned live from the ambient file
64170dd 2026-11-25 Merge pull request #18 from PikPakPik/perf-copies-paralleles: sparse copies snapshot and copy back their tiles on several threads
d6070f1 2026-11-25 Merge pull request #45 from Arvi23/pr/integer-target-blend: no blending on integer render targets in host pipelines
8629211 2026-11-25 GPU: no blending on integer render targets in host pipelines
e6b6386 2026-11-25 A draw keeps its root constants on the stack, its fresh-table slots in a bit set and the tables it made in a buffer the state reuses, so a draw no longer allocates for them (up to 2 + 1 + 1 allocations per draw)
b41f3f3 2026-11-25 A table made afresh is looked up by a quick hash and its record words are compared, instead of a SipHash over them (a collision now costs a copy, not wrong descriptors)
b022d86 2026-11-25 The host heap behind a descriptor heap is returned without taking the pending lock once it is made, since nothing is queued after that
dd799aa 2026-11-25 A range shaders write is declared once until the title next writes memory, and the submission finds the unique ranges with a set, so a list of thousands of draws no longer scans and re-snapshots the same ranges per step
5e08eee 2026-11-25 Buffer-to-texture copies take their staging bytes from the list's pooled scratch chunks instead of creating and mapping a committed upload buffer each, which is much cheaper
ab5ece8 2026-11-25 Clamps hoisted into one list bind their root signature, pipeline and budget once, not before every clamp (3 fewer D3D12 calls per indirect draw)
4ffc476 2026-11-25 The ambient light's colour can be set (VICTORY_AMBIENT_TINT), and level and colour tuned in a running game from a file (VICTORY_AMBIENT_FILE, diagnostics)
92d353f 2026-11-25 A pipeline a list binds again is not named, queried and kept again (one lookup by address instead of two QueryInterface calls, three locks and a name search per switch)
84ea9c1 2026-11-25 A root constant buffer not 256-byte aligned is copied into list scratch once for the draws of a list that bind it, until a title copy or run intervenes (up to 64 KiB less copied per repeat)
cf57ca3 2026-11-25 A queue held on a wait first finishes the unwaited submission, whose bottom-of-pipe writes the wait may be for (it stood two seconds a frame)
bdecda3 2026-11-25 An unwaited submission is finished when it is waited for (its shaders' writes, read-backs and bottom-of-pipe writes), so every submission of the title's may go unwaited; the next one's buffer copies are refreshed meanwhile (VICTORY_ASYNC_SUBMIT=1, off by default)
9b89ec2 2026-11-25 Only the title's submissions may return before the host GPU is done; ours wait (the present's capture buffer was freed with its copy pending: device removed at the first frame with VICTORY_ASYNC_SUBMIT=1)
a0a4591 2026-11-25 Sparse copies: snapshots and copy-back of shader-written tiles on several threads, ranges already declared in the submission skipped, tiles found under one lock
71652a2 2026-11-25 README: vehicles and distant trees work, the phone does not; memory and descriptor slots among the known problems
788ea46 2026-11-25 A search for free dynamic slots that failed is not walked again until a release, and at most eight times a frame (frames of seconds when the slots ran out)
e11fb06 2026-11-25 The tile sweep asks the system by region and outside the tiles' lock, once a frame, and logs what it took
0a32e89 2026-11-25 Tiles of title memory that is no longer there can be given back (VICTORY_SWEEP_TILES=2), so the copies stop growing; off by default
f2cc100 2026-11-25 The views are looked at once for a run of guarded blocks, and pages with one view make no list (less time under the guards' lock)
579a8d9 2026-11-25 Blocks the kernel is about to write stay writable for two seconds, at every view of their pages, and are reported written meanwhile (an asynchronous read into a block guarded again failed, and the title stopped)
e7e6c00 2026-11-25 Guarded blocks given back in a run that spans views are given back one by one when the run fails
1c7231d 2026-11-25 A view made read-only between its mapping and the runtime's notice is given back; a read-only view block no guard knows is opened on its fault
ae90937 2026-11-25 Every view of a guarded physical page is guarded with it: a write of the title's through another address is seen (parts of moving vehicles were left behind); VICTORY_ALWAYS_COPY modes (diagnostics)
4cbe9d7 2026-11-25 README: what this fork is, its state on one AMD machine, how it is run, credits; VICTORY_ALWAYS_COPY (diagnostics)
cf62e1a 2026-11-25 VICTORY_SHADER_SWAP replaces shader containers by files: the distant trees' variants without wind stand in for those with wind
03629f3 2026-11-25 A pixel's quad is read through quad reads, helper lanes included: a derivative taken by hand at a triangle's edge was the whole value (lines on car paint, glass and skin)
76387c2 2026-11-25 Hoisted clamps run one after another by default (mode 7): no flicker of mesh objects
4f45ab8 2026-11-25 The resolve's indirect diffuse light is replaced by a constant while rays do not run (magenta foliage, blocks of colours indoors); VICTORY_SHADER_PATCH changes a word of a shader's code (diagnostics)
afca189 2026-11-25 Work in progress: switches to find the flicker of hoisted clamps (modes 2 to 7, VICTORY_HOIST_FILE)
918ab03 2026-11-25 Clamps hoisted before a run of indirect draws by default (VICTORY_HOIST_CLAMPS=0 turns it off)
ccab973 2026-11-25 Clamps of a run of indirect draws go to a list before it (VICTORY_HOIST_CLAMPS=1); GPU time per kind of command; shader cache of another build
f520772 2026-11-25 A helper lane's v_readfirstlane reads its own value (upstream 0601955)
7de1751 2026-11-25 Diagnostics: a peeked view's bytes as a file; a test that detiles a peeked 256x16 texture
236ecce 2026-11-25 Diagnostics: a draw's root buffers, index buffer and whole indirect record in the pass text
18e57db 2026-11-25 Screen passes get pipeline workers and a wait budget of their own
b1105d1 2026-11-24 A host pipeline's state in its log line (diagnostics)
2aad398 2026-11-24 Twice the dynamic descriptor slots: tables of heavy frames no longer fit and their draws were dropped
64e7c9f 2026-11-24 Memory the runtime fills from disk counts as written for host copies of buffers
fa042f6 2026-11-24 Raw-viewed structured buffers keep their size query
1503444 2026-11-24 Buffer element counts by slot from the host's table, not a size query (AMD compiler crash)
1ebf29d 2026-11-24 A descriptor's size words are asked of the host only when read as data
2823ba1 2026-11-24 A first half of an interpolation other instructions read evaluates on its own
c98fac0 2026-11-24 Why a draw was not prepared, in the drop notes (diagnostics)
ffedb7d 2026-11-24 Tile snapshots reused from their own pool; a sampling profile and GPU wait counts (diagnostics)
3afed01 2026-11-24 victory-gpu: place large buffers in VRAM with Wave64 GPU page hashing
df71546 2026-11-24 Large descriptor tables made once for every length shaders read them to
78028ad 2026-11-24 Reads through root views stay inside the memory behind them
72e748e 2026-11-24 Rays: builders' writes ordered around atomics, their root buffers coherent
4d63a09 2026-11-24 Ambient light off by default: the title's own light only
0603123 2026-11-24 A least amount of indirect light where the title's ray-traced probes have none
beffaea 2026-11-24 Views the title wrote last are made at each submission; memory by address clamped
7eb9545 2026-11-24 Depth twins on by default: every material is lit
9956958 2026-11-24 Tiles of copies no longer given back (crash 120 frames into the world); view indices kept inside the heap
a049b48 2026-11-24 victory-gpu: make experimental async submit and GPU_UPLOAD opt-in for driver stability
8202cfc 2026-11-24 victory-gpu: zero-syscall VEH safe_copy, texture watch/backoff fast path, and batched copy timing
eebcf07 2026-11-24 victory-gpu: wait for in-flight async GPU work before signalling title fences
c5948a7 2026-11-24 victory-shader: keep parked lanes across v_movrels_b32 and gate BVH user-data unspill behind VICTORY_RAYS
a873b24 2026-11-24 victory-gpu: fix GPU use-after-free in texpool/Staging/HostHeap and coalesce VirtualProtect
2bfe95a 2026-11-24 victory-gpu: per-region guard epochs, faster bindless streaming, and async GPU submissions
0506e78 2026-11-24 victory-gpu: guard-epoch O(1) watch fast path, pooled sparse snapshots, parallel copy_back, and Resizable BAR GPU_UPLOAD
520cfb9 2026-11-24 victory-shader + victory-gpu: implement raw root UAVs (D3D12_ROOT_PARAMETER_TYPE_UAV)
2b6adbb 2026-11-24 victory-gpu: cache unmapped holes in watch::guard (fix 2.5GB/frame fault storm) and set 240 FPS default cap
fa07481 2026-11-24 victory-gpu: uncap frame pacing by default (optional VICTORY_FPS override)
18e3849 2026-11-24 victory-gpu: keep page-guarding active on sparse shadows and speed up copy_back with 64-bit dirty-span merge
522d2bc 2026-11-24 victory-shader: lower RDNA2 POPS/ROV fast path and ray-tracing CBV-as-data
56fad1c 2026-11-24 victory-shader: resolve wrexec SCC, high-half phi, fma_mixlo old-half, multi-mask lane_select, and pack2 spills
d3a5f2d 2026-11-24 Shaders: fix f16 inline constants/EXEC init/wave restore; add f64 ops, f16/f64 wave sums, and mesh MaskData::Host
04b1a1d 2026-11-24 Merge official main up to 8e79c80 (CI), before the per-lane phi and waterfall changes
8044d21 2026-11-24 Depth twins off unless VICTORY_DEPTH_TWINS=1; peeks read constant and buffer views
e52106f 2026-11-24 Depth textures written as images: twins the title's compute writes, copied into the depth after each dispatch
36a19d1 2026-11-24 Rays: dispatches that count threads, coherent builders, off unless VICTORY_RAYS=1
d567f6b 2026-11-24 WIP: rays: memory by address on the host, ray intersections
e63e820 2026-11-24 Shaders: memory by address (the ray tracing builders translate)
2655edb 2026-11-24 Shaders: lane permutations follow the console's lanes; pipelines: slow compiles are stopped by the memory they take
5559960 2026-11-24 Shaders: SCC as a number, lane permutations, one lane read of a section, packed half FMA, atomics with 16-bit addresses
7df4a50 2026-11-24 Shaders: ACCC containers (AMD's ACO compiler), descriptors made raw
8b2c36f 2026-11-24 Textures bindless slots begin to view are uploaded before the submission's lists
4284939 2026-11-24 Streamed textures: only the mips the title's view reads, padding ignored
a549d67 2026-11-24 Pipelines: full waiting again; empty title memory looked at each frame and when committed
3a839d4 2026-11-24 WIP: no tiles for unmapped title memory
95e935d 2026-11-24 Pipelines: workers lost to the driver's compiler are replaced, hung compiles stopped, no waiting for lost ones
4e23b53 2026-11-24 Textures uploaded again as soon as streaming rewrites them; VICTORY_TEXTURES=keep|always
d88566e 2026-11-24 Structured buffers of 16-byte multiples as raw views (AMD reads element 0 otherwise)
fd3b8ce 2026-11-24 GPU: scratch and tile memory given back, planes put together, memory counters
8e79c80 2026-11-24 CI: clippy and tests on pull requests and main
c7a5f49 2026-11-24 Merge pull request #93 from TG0000/perf/bcpack-bucket-lookup
2005e35 2026-11-24 Merge pull request #88 from AndyLatimer/fix/title-size-overflows
c810215 2026-11-24 Diagnostics: what placed buffers hold
1b0c6b5 2026-11-24 GPU: pipelines the compiler does not end are left out
4c0409f 2026-11-24 Diagnostics: snapshots within a frame, indirect arguments
58fed37 2026-11-24 Merge pull request #706 from Lyrnc/fix/cmp-lg-ordered
a51bdfa 2026-11-24 Merge pull request #696 from catminou4/devin/formats-doc-micro-link
02f404e 2026-11-24 Merge pull request #685 from catminou4/devin/constant-color-cfg-windows
5cd7421 2026-11-24 Merge pull request #681 from fuzeofficlal/fix/vertex-buffer-null-unbind
8d23f07 2026-11-24 Merge pull request #680 from EatCodeSleep2/fix/wine-staging-upload-128
22700b9 2026-11-24 Merge pull request #675 from lameloball2813-ui/docs/rustdoc-private-links
6db02ea 2026-11-24 Merge pull request #671 from fuzeofficlal/fix/copyresource-cpu-texture-source
16bd577 2026-11-24 Merge pull request #666 from lameloball2813-ui/fix/load-probe-target-dir
b4b4110 2026-11-24 Merge pull request #654 from SantannaCarlos/fix/uav-describe
cf0bb43 2026-11-24 Merge pull request #637 from AlanQiang0418/codex/gfx10-packed-inline-constants
6f0a10a 2026-11-24 Merge pull request #634 from SAitken91/shader/image-sample-o
9a92ccf 2026-11-24 Merge pull request #612 from KoLolX3/fix/phi-liveness-transitive
204bdcc 2026-11-24 Merge pull request #611 from AdvaitPalve11/fix/config-show-data-dir
a4d7b16 2026-11-24 Merge pull request #565 from SAitken91/shader/conservative-depth-position
fd92558 2026-11-24 Merge pull request #520 from Quat3rnion/victory-xasync-payload
dd54de2 2026-11-24 Merge pull request #448 from Dufftwt/fix/min-max-signed-zeros
0ddbce8 2026-11-24 Merge pull request #433 from r4tedx/tests/serialise-valu-gpu-tests
ce42e77 2026-11-24 Merge pull request #432 from Proudnoob4/fix/uninitialized-sgprs
f0894c2 2026-11-24 Merge pull request #429 from marquezinii/fix/xgamesave-documented-errors
9f18eb4 2026-11-24 Merge pull request #423 from pedrofort/shader-perm-no-int64
a1ca60f 2026-11-24 GPU: 32-bit targets read as shared-exponent colors through placed views
dd273df 2026-11-24 Merge pull request #419 from itsRevela/fix/swallow-keyboard-menu
3c86238 2026-11-24 Merge pull request #418 from enzoduarte1/gpu/half-subnormal-output
2e4e280 2026-11-24 Merge pull request #384 from LouisRiverstone/shader-cvt16-saturate
faba64c 2026-11-24 Merge pull request #382 from Dufftwt/fix/stencil-ref-shader-flag
c30fc99 2026-11-24 Merge pull request #381 from Dufftwt/fix/psv-sample-frequency
ebf3615 2026-11-24 Merge pull request #376 from LouisRiverstone/shader-cvt-norm-f16
f67c50c 2026-11-24 Merge pull request #372 from marquezinii/fix/capture-unorm16-formats
40b7ed1 2026-11-24 Merge pull request #371 from marquezinii/fix/pls-path-small-buffer
63af596 2026-11-24 GPU: copies keep what shaders wrote where the title changed nothing
7374bfb 2026-11-24 Merge pull request #365 from FIocker/gpu/publish-initialized-descriptor-heap
42361b0 2026-11-24 Merge pull request #356 from FIocker/gpu/stable-drop-rank
2745589 2026-11-24 Merge pull request #355 from enzoduarte1/clippy/clean-workspace-lints
533b7e2 2026-11-24 Merge pull request #341 from enzoduarte1/config/empty-test-titles-env
edf64e2 2026-11-24 Merge pull request #336 from hophaver/fix/dynamic-descriptor-exhaustion
5a16e66 2026-11-24 Merge pull request #313 from Abdellah-Khanani/loader-aligned-unwind
93c0852 2026-11-24 Merge pull request #285 from Dufftwt/fix/name-title-dll-that-fails-to-prepare
db89843 2026-11-24 Merge pull request #276 from Dufftwt/fix/s-ttracedata-imm-opcode
89e1031 2026-11-24 Merge pull request #260 from Dufftwt/fix/f32-to-half-rounding-carry
88b6658 2026-11-24 Merge pull request #249 from Maraco91/gpu/get-device-newer-levels
56d5032 2026-11-24 Merge pull request #245 from Dufftwt/fix/import-trap-in-crash-log
9e44f38 2026-11-24 Merge pull request #239 from Maraco91/xtaskqueue/terminate-ends-dispatch
7c8cecb 2026-11-24 Merge pull request #229 from Dufftwt/fix/gamertag-modern-suffix-empty
27f9b76 2026-11-24 Merge pull request #209 from Maraco91/xgamesave/empty-configuration-id
6af0fae 2026-11-24 Merge pull request #205 from asdaasdsaddf/fix/null-uav-dimension
27c960b 2026-11-24 Merge pull request #204 from Dufftwt/fix/crash-report-fault-memory
3dd37e6 2026-11-24 Merge pull request #198 from Dufftwt/fix/crash-report-instruction-bytes
13ac0e6 2026-11-24 Merge pull request #167 from Maraco91/xmem/zero-length-unmap
025e6c5 2026-11-24 Shaders: uniform descriptors joined in a former use
787f665 2026-11-24 Merge pull request #161 from lordon1a/fix/pipeline-staging-cleanup
ba7d873 2026-11-24 Merge pull request #152 from Makto94/fix/net-test-win10
80c0d12 2026-11-24 Merge pull request #140 from lordon1a/feat/cmp-class-f16
1dc1b4f 2026-11-24 Shaders: programs that trace rays, SCC joins
3d77d5e 2026-11-24 Merge pull request #117 from tedin7/vic-s-bitcmp-2
c0fc5d3 2026-11-24 Shaders: 64-bit sums in whole-wave sections
b0328ba 2026-11-24 Merge pull request #111 from Dufftwt/fix/narrow-paths-translated-twice
0e990e4 2026-11-24 Merge pull request #104 from TG0000/perf/bitcode-worklist-liveness
ec476ff 2026-11-24 Shaders: samplers a branch picks, per-lane descriptors joined in several blocks
c2e7254 2026-11-24 Merge pull request #87 from jeremygold02/codex/copy-stat-frame-window
1bb4e98 2026-11-24 Shaders: registers joined in a former use (masks, descriptors)
1f858b0 2026-11-24 Merge pull request #25 from artor410/packed-buffer-formats
ac36741 2026-11-24 GPU: unchanged textures looked at less often
ebd990d 2026-11-24 GPU: streamed textures without their largest mips as video memory fills up
6db1472 2026-11-24 GPU: unchanged tiles looked at less often
6e2a299 2026-11-24 GPU: texture and staging memory in slabs, parallel copies, plans whole once a frame
3e39eac 2026-11-23 fix(hle): return XAsync payloads for nonnegative results
51f1627 2026-11-24 Shaders: v_cmp_lg is ordered on AMD hosts too (olt | ogt, not fcmp one)
6bca461 2026-11-24 Shaders: tables in the code read through computed pointers; hair strands left out
0f0c872 2026-11-23 Docs: drop the private intra-doc link to MICRO in texture.rs
4f53d6f 2026-11-24 GPU: upload CPU-written sources before texture region copies
0fe13af 2026-11-24 Shaders: rays hit nothing where the BVH cannot be read; GPU: sparse copies in 2 GiB chunks
ab3eb92 2026-11-23 Shaders: gate the constant_color test helper to Windows
31ce30f 2026-11-24 GPU: unbind vertex buffer slots for null view arrays
5864aab 2026-11-23 Docs: rustdoc builds the loader, GPU and shader crates without warnings
cc4d18d 2026-11-24 GPU: copies of title memory in sparse buffers
af22e5d 2026-11-23 GPU: use the host upload path for texture staging copies
0064c08 2026-11-24 GPU: upload CPU-written textures before CopyResource
0723474 2026-11-24 GPU: placed textures only where their pages are, cheaper refreshes and checks
764fa19 2026-11-23 Loader tests: build the probe fixture under CARGO_TARGET_TMPDIR
41b9207 2026-11-24 GPU: the title's GPU heap shown whole and used in place; far less work per bindless submission
d74558c 2026-11-24 GPU diagnostics: describe a buffer UAV by its range, not as uav None
a0efd2d 2026-11-24 Shaders: decode GFX10 packed inline constants by operand type
260611a 2026-11-23 Shaders: image_sample_o with constant offsets
85340e8 2026-11-24 GPU: drop the memory probes (heaps of file mappings leak in the driver)
3dd467c 2026-11-23 Shaders: a phi that only another read phi consumes is live too
25b0f88 2026-11-24 Test probe: compare direct and joined reads of a shader timestamp
440589e 2026-11-24 GPU: title memory used in place by the host GPU
9a8908c 2026-11-24 core: show data_dir in config and check its asset policy
e4e4904 2026-11-24 Shaders: lane permutations through LDS, SGPRs no one set; more time for bindless slots
c3f8407 2026-11-24 Shaders: whole-wave sections fetch inactive lanes through permlanex16 with fi
0d2f6a6 2026-11-24 Survive driver compiler crashes; diagnostics for target dumps on demand
c249f02 2026-11-23 Shaders: centroid position with conservative depth
e136a3f 2026-11-23 Merge PR 413 (bindless heap materialization) into the local branch
e0c582e 2026-11-23 Revert the integration's old PR 185 (copies gigabytes per frame in the world)
161dc7f 2026-11-23 Diagnostics: keys and captures on demand; test fixes after the merge
da7a7f8 2026-11-23 Merge PR 414 (community integration) into the local branch
126388e 2026-11-24 GPU: keep bindless record-time work off other recording threads
efc045e 2026-11-24 GPU: bound bindless slot work while the world streams in
4d34cef 2026-11-23 GPU: cut a list at a wait that follows work
d852b24 2026-11-23 GPU: queues hold work back for the memory values lists wait for
e8d3b64 2026-11-24 GPU: the larger dynamic range only where the heap still fits
7afcbff 2026-11-24 GPU: make bindless plans incremental and throttle bindless texture uploads
cc9e86d 2026-11-23 GPU: count queue waits that are not met (diagnostics)
fd1bf62 2026-11-23 GPU: rectangle lists without vertex buffers through a generated geometry shader, indirect draws included
bf84134 2026-11-24 Shaders: maximums and minimums order -0 below +0, as RDNA2 does
1106f43 2026-11-24 Shader tests: run the VALU GPU tests one at a time
f4438b3 2026-11-23 Shaders: reads of uninitialized SGPRs read as zero, as VGPRs do
d3c29f1 2026-11-23 GPU: name the pipeline and topology of dropped indirect draws (diagnostics)
f0ee35e 2026-11-24 XGameSave: a full host disk is E_GS_OUT_OF_LOCAL_STORAGE, not the quota
d5704aa 2026-11-24 fix(shader): lower dynamic v_perm_b32 without INT64
28c1b07 2026-11-24 XGameSave: a read result into a small buffer is E_GS_PROVIDED_BUFFER_TOO_SMALL
eb30a0d 2026-11-24 GPU: write half subnormals from f32 rectangle outputs
bcc5f60 2026-11-24 GPU: materialize bindless heap views and refresh only what changed
d3917df 2026-11-24 Merge PR #342 (Shaders: lower partial-channel typed buffer stores) into the integration
e510ba7 2026-11-23 Shaders: image sizes read through sibling loads; quad derivatives for compute samples
3287834 2026-11-24 Merge PR #333 (Shaders: native blobs must place root parameters, vertex inputs and thread ids in real registers) into the integration
9a7c323 2026-11-24 Merge PR #332 (Shaders: lower partial-channel image_store) into the integration
d9e7d31 2026-11-24 Merge PR #329 (Query record counts for structured buffers read only through their descriptor) into the integration
ff8694a 2026-11-23 Window: keep Alt and F10 from freezing the title in menu mode
e0a596a 2026-11-24 Merge PR #328 (Shaders: lower MSAA image_load and image_get_resinfo) into the integration
4a8d5ec 2026-11-24 Merge PR #320 (Shaders: translate for the host's wave sizes; console flow for lane reads where lanes branch differently) into the integration
efd7063 2026-11-24 Merge PR #317 (Shaders: support cube and MSAA dimensions in descriptor_data) into the integration
eb962e1 2026-11-24 Merge PR #306 (Shader: fix biased 16-bit image samples and DXIL validator failures) into the integration
e186586 2026-11-24 Merge PR #305 (Shaders: lower 64-bit image atomics) into the integration
2af66ff 2026-11-23 Shaders: v_cvt_u16_f16 and v_cvt_i16_f16 saturate at 16 bits
60303c3 2026-11-24 Merge PR #288 (Shaders: cap BDD memo table and bail out early on overflow) into the integration
c8b1a03 2026-11-24 Merge PR #286 (Shaders: accept known D3D_MIN_PRECISION signature elements) into the integration
302285d 2026-11-24 Merge PR #284 (Shaders: RDNA2's legacy multiply-adds are the fused ones) into the integration
80c2eb9 2026-11-24 Shaders: drop a doubled blank line in the spill tests
bd1110d 2026-11-24 Merge PR #276 (Shaders: s_ttracedata_imm is SOPP opcode 0x28, not 0x26) into the integration
511a8bb 2026-11-24 Merge PR #269 (Shaders: scalars leave divergent loops and branches as the wave's last path left them) into the integration
c2f60d6 2026-11-24 Shaders: a pixel shader writing SV_StencilRef declares StencilRef
404d7fd 2026-11-24 Shaders: PSV0 says a pixel shader runs per sample when an input does
ded334b 2026-11-23 Shaders: lower v_cvt_norm_i16_f16 and v_cvt_norm_u16_f16
2f39239 2026-11-24 Merge PR #265 (Shaders: private (scratch) memory as a per-lane array) into the integration
fa3efd1 2026-11-24 Merge PR #259 (Shaders: query a descriptor's level count only when word 3 is read) into the integration
b02980d 2026-11-23 Shaders: follow SGPRs spilled to VGPR lanes; diagnostics save untranslated shaders
a62995f 2026-11-24 Take out upstream PRs that do not serve the 3D picture or stability
0f369d6 2026-11-23 Capture: R16G16B16A16_UNORM targets are integers, format 89 is 10:10:10:2
a4e7f1d 2026-11-24 GPU: replay pending descriptors before publishing the host heap
536aa29 2026-11-23 Shaders: a native vertex input must load into the VGPRs
376b3cf 2026-11-23 Storage: a path buffer too small fails with E_FAIL, as on Xbox
efb3e3d 2026-11-24 GPU diagnostics: order tied drop counts consistently
40c127c 2026-11-23 Chore: clear two workspace Clippy errors
85f90f1 2026-11-24 Revert PR #234 (xmem section views, overlapped DirectStorage) from the integration
fabe8bc 2026-11-24 Shaders: validate the split and console-flow test programs
6cdedbe 2026-11-24 Fix the three Windows tests that failed on integrate/all
765712f 2026-11-23 Shaders: a thread-id count field of 3 still means three components
94a8607 2026-11-23 Shaders: lower partial-channel typed buffer stores
cc63aa2 2026-11-23 Config: ignore an empty VICTORY_TEST_TITLES
3c81420 2026-11-24 GPU: drop draws whose fresh tables cannot be made; 2^18 dynamic slots
fb7d023 2026-11-23 Shader tests: name the reason each native blob is refused
808e921 2026-11-23 Shaders: a native program's root parameters must sit in its SGPRs
00fc4a7 2026-11-24 gpu: materialize and refresh bindless ResourceDescriptorHeap views
5cc9a7e 2026-11-24 Shaders: console flow joins masks that start as constants
81ef5a3 2026-11-23 Shaders: lower partial-channel image_store
02564c5 2026-11-23 Query record counts for structured buffers read only through their descriptor
6d7602d 2026-11-23 Shaders: lower MSAA image_load and image_get_resinfo
f636134 2026-11-24 Shaders: per-lane descriptors fetched with buffer_load over a table or heap
ee1f683 2026-11-24 Merge fix/packed-half: keep #176's half_inputs and add its end-to-end test
451f2c6 2026-11-24 Shaders: lane reads and writes where lanes branch differently run in the console flow
4a3f8a8 2026-11-23 Shaders: support cube and MSAA dimensions in descriptor_data
0758d0a 2026-11-24 Merge fix/validate (DXIL validator fixes) into the integration
2d43710 2026-11-24 Merge fix/firstlane (per-lane descriptors through phis) into the integration
2cf1b15 2026-11-24 Merge integrate/shaders (upstream shader PRs) into the integration
b5c6678 2026-11-24 loader: unwind a 16-byte aligned copy of the context
eedd372 2026-11-23 GPU: hold retired objects 120 frames; VICTORY_KEEP_RETIRED holds them for good
038a36f 2026-11-23 Shaders: lower 64-bit image atomics
4d838a9 2026-11-24 Merge PR #25 (shader: decode the packed typed-buffer formats) into the integration
cc39174 2026-11-24 Merge PR #226 (Shaders: test that an image read as float and used by an atomic is annotated twice) into the integration
6b0f69d 2026-11-24 Shader: fix biased 16-bit image samples and DXIL validator failures
e4c7c8a 2026-11-24 Merge PR #200 (Shaders: lower v_div_scale_f32, v_div_fmas_f32 and v_div_fixup_f32) into the integration
80a3884 2026-11-24 Merge PR #179 (Shaders: index interpolation halves on demand) into the integration
8fda153 2026-11-24 Merge PR #169 (Shaders: tbuffer_load_format_d16 loads pack two 16-bit channels per VGPR) into the integration
4576c50 2026-11-24 Merge PR #168 (Shaders: SCC of s_min, s_max and s_lshl1-4_add_u32 as the ISA sets it) into the integration
703aa0e 2026-11-24 Merge PR #166 (Shaders: lower s_bitset{0,1}_b64, s_brev_b64 and s_bitreplicate_b64_b32) into the integration
d81cfaf 2026-11-24 GPU: frame wait's diagnostics log no longer names a removed tick
6148054 2026-11-24 Merge PR #162 (Shaders: lower v_sad_u8, v_sad_hi_u8, v_sad_u16, v_sad_u32 and v_msad_u8) into the integration
7e480f6 2026-11-24 Shaders: refuse mesh outputs past the host's 32 KiB
fec4d02 2026-11-24 Shaders: leave the array slice of a level-of-detail query undef
c30b9c3 2026-11-24 Shaders: give each packed signature row one interpolation mode
b3e1e23 2026-11-24 Shaders: declare the position centroid under conservative depth
09a673b 2026-11-24 Merge PR #157 (Shaders: lower 24-bit mul_hi, saturating 16-bit packs, integer dot products and v_ldexp_f32) into the integration
d48de53 2026-11-24 Reapply "Merge remote-tracking branch 'pr/12' into integrate/all"
50a3f1e 2026-11-24 Reapply "Merge remote-tracking branch 'pr/150' into integrate/all"
8302f6a 2026-11-24 Reapply "Merge remote-tracking branch 'pr/213' into integrate/all"
56037e9 2026-11-24 Merge integrate/runtime into the integration
d158fdb 2026-11-24 Merge PR #156 (Shaders: lower v_mul_hi_u32_u24 and v_mul_hi_i32_i24) into the integration
176c766 2026-11-24 Shaders: cap BDD memo table and bail out early on overflow
9356bee 2026-11-24 Shaders: test a scalar counted in nested divergent loops
b05196d 2026-11-24 Merge PR #154 (Shaders: use DXIL IsNaN for unordered/ordered and self float compares) into the integration
42ad74e 2026-11-24 Shaders: trace per-lane descriptors through the phis that join them
05d6d18 2026-11-24 Merge PR #145 (Shaders: read both ballot halves for wave32 host-wave mask ops) into the integration
192a82d 2026-11-24 Merge PR #237 (HLE: offload async saves, reuse audio buffers, pin APU heap, and decouple input rumble/dialogs) into the integration
9fe73ba 2026-11-23 shader: decode the packed typed-buffer formats
6afdb58 2026-11-23 Shaders: accept known D3D_MIN_PRECISION signature elements
16ac1b3 2026-11-24 Merge PR #139 (Shaders: reuse SSA sums when tracing memory addresses) into the integration
0bbfc31 2026-11-24 Merge PR #138 (Shaders: support image_sample_b with a16 and _cl LOD-clamp variants; gradient samples on cube images) into the integration
5cd3c93 2026-11-24 Merge PR #234 (HLE: overlapped DirectStorage I/O and persistent xmem section views) into the integration
393ca51 2026-11-24 Merge PR #180 (Shaders: twin descriptor loads and quad gradients for compute samples) into the integration
2a000c1 2026-11-23 GPU: root views in loose memory get a copy reaching past the view's start
3b27a86 2026-11-24 Shaders: pair packed fp16 pixel inputs whose row2 repeats their row with the unlisted signature row
9a9b3dd 2026-11-24 Merge PR #224 (HLE: report the host's commit headroom before the title starts) into the integration
f653c1c 2026-11-24 Merge PR #216 (Loader & HLE: harden VEH crash handler, PE prep/exports/TLS, and allow oleaut32) into the integration
d5d0970 2026-11-24 Merge PR #177 (Shader: key heap handles by element type too) into the integration
d18cd26 2026-11-24 Shaders: write signature parts as the validator rebuilds them
187472f 2026-11-24 Merge PR #193 (Shaders, GPU: add runtime stride table for structured buffers (#155)) into the integration
964aa13 2026-11-24 Merge PR #182 (Files: `..` in a title path stops at the virtual drive's root) into the integration
3febd4a 2026-11-24 Merge PR #132 (XTaskQueue: terminating a queue cancels its delayed callbacks too) into the integration
fb0a797 2026-11-24 Merge PR #159 (GPU: an occlusion query around a draw the host skipped keeps its stand-in) into the integration
7660587 2026-11-24 Merge PR #158 (GPU: the depth bounds test reaches the host) into the integration
3a653c1 2026-11-24 Merge PR #24 (Run indirect mesh dispatches through indirect execution) into the integration
43a4ce4 2026-11-24 Merge PR #242 (GPU: make ExecuteIndirect clamping countBuffer-aware and non-destructive) into the integration
fa5189d 2026-11-24 Shaders: RDNA2's legacy multiply-adds are the fused ones
774a50f 2026-11-23 GPU: free host objects some frames after their last use
e1dd56b 2026-11-24 Revert "Merge remote-tracking branch 'pr/12' into integrate/all"
02744d5 2026-11-24 Merge PR #176 (Shaders: pair packed fp16 pixel inputs with unlisted signature rows) into the integration
d0f97ef 2026-11-24 Revert "Merge remote-tracking branch 'pr/245' into integrate/all"
5da8aaf 2026-11-24 Revert "Merge remote-tracking branch 'pr/213' into integrate/all"
4270e8f 2026-11-24 Revert "Merge remote-tracking branch 'pr/204' into integrate/all"
13d385d 2026-11-24 Revert "Merge remote-tracking branch 'pr/198' into integrate/all"
fe72473 2026-11-24 Revert "Merge remote-tracking branch 'pr/207' into integrate/all"
011e7de 2026-11-24 Revert "Merge remote-tracking branch 'pr/133' into integrate/all"
9b32471 2026-11-24 Revert "Merge remote-tracking branch 'pr/26' into integrate/all"
4eb9643 2026-11-24 Revert "Merge remote-tracking branch 'pr/87' into integrate/all"
7dfc31b 2026-11-24 Revert "Merge remote-tracking branch 'pr/115' into integrate/all"
b79192f 2026-11-24 Revert "Merge remote-tracking branch 'pr/244' into integrate/all"
2645fcd 2026-11-24 Revert "Merge remote-tracking branch 'pr/150' into integrate/all"
050b9ac 2026-11-24 Merge PR #134 (GPU: drop a draw whose table found no dynamic slots; more dynamic slots) into the integration
cb0cc5f 2026-11-24 Merge PR #160 (Shaders: a16 packs an image's coordinates, not its derivatives) into the integration
65d73aa 2026-11-24 Merge PR #230 (GPU: skip unchanged texture uploads via prelude copy recording) into the integration
b67d15d 2026-11-24 Shaders: scalars leave divergent loops and branches as the wave's last path left them
cf5af29 2026-11-24 Merge PR #164 (Shaders: lower integer memory atomics and sub-dword buffer accesses) into the integration
b3b27b8 2026-11-24 Merge PR #215 (GPU: reuse the read buffer for watched textures) into the integration
5f29f3a 2026-11-24 Loader: a title DLL that fails to prepare is named in the error
8d3b701 2026-11-24 Merge PR #143 (GPU: hash and detile unwatched textures from a kernel-read copy) into the integration
e232d3d 2026-11-23 Shaders: query a descriptor's level count only when word 3 is read
b302c71 2026-11-24 Merge PR #206 (GPU: make a structured buffer view off its host copy's elements a raw view) into the integration
44cbbb1 2026-11-24 Merge PR #185 (GPU: maintain shadow-buffer coherency for directly indexed SRVs) into the integration
7463484 2026-11-24 Shaders: private (scratch) memory as a per-lane array
b16b81f 2026-11-24 Merge PR #163 (Shaders: lower scalar and vector arithmetic and compares) into the integration
f5e18de 2026-11-24 Merge PR #135 (GPU: age retired host resources in presented frames, not the title's 60 Hz clock) into the integration
3f8a1de 2026-11-24 Merge PR #136 (Shader: support s_buffer descriptor/structured loads, dead descriptor phis, PSV0/ISG1 fixes, calculateLOD and image_gather4_l) into the integration
4c9a922 2026-11-24 Shaders: handle both-huge operands (-1e35 / 1e35) in v_div_scale_f32
717f322 2026-11-24 GPU: GetDevice and the DXGI device hand out the device at every level
8986e24 2026-11-24 Merge remote-tracking branch 'pr/247' into integrate/all
b94ba80 2026-11-24 Merge remote-tracking branch 'pr/245' into integrate/all
b0d137b 2026-11-24 Merge remote-tracking branch 'pr/244' into integrate/all
cfd7736 2026-11-24 Merge remote-tracking branch 'pr/240' into integrate/all
13c88ff 2026-11-24 Merge remote-tracking branch 'pr/239' into integrate/all
3b30332 2026-11-24 Merge remote-tracking branch 'pr/236' into integrate/all
e6fc838 2026-11-24 Merge remote-tracking branch 'pr/229' into integrate/all
beeffa2 2026-11-24 Merge remote-tracking branch 'pr/228' into integrate/all
e94f123 2026-11-24 Merge remote-tracking branch 'pr/221' into integrate/all
e60e6b7 2026-11-24 Merge remote-tracking branch 'pr/219' into integrate/all
393b973 2026-11-24 Merge remote-tracking branch 'pr/218' into integrate/all
1d6039c 2026-11-24 Merge remote-tracking branch 'pr/213' into integrate/all
c7deab0 2026-11-24 Merge remote-tracking branch 'pr/209' into integrate/all
c811c22 2026-11-24 Merge remote-tracking branch 'pr/207' into integrate/all
edeb2e1 2026-11-24 Merge remote-tracking branch 'pr/205' into integrate/all
9e47523 2026-11-24 Merge remote-tracking branch 'pr/204' into integrate/all
aadd87a 2026-11-24 Merge remote-tracking branch 'pr/202' into integrate/all
d8f3a6c 2026-11-24 Merge remote-tracking branch 'pr/198' into integrate/all
7013788 2026-11-24 Merge remote-tracking branch 'pr/196' into integrate/all
26292a1 2026-11-24 Merge remote-tracking branch 'pr/194' into integrate/all
15649f6 2026-11-24 Merge remote-tracking branch 'pr/188' into integrate/all
89f70c3 2026-11-24 Merge remote-tracking branch 'pr/186' into integrate/all
9f4f3b4 2026-11-24 Merge remote-tracking branch 'pr/183' into integrate/all
032e863 2026-11-24 Merge remote-tracking branch 'pr/174' into integrate/all
2a2dc7d 2026-11-24 Merge remote-tracking branch 'pr/172' into integrate/all
744707c 2026-11-24 Merge remote-tracking branch 'pr/171' into integrate/all
6bb3375 2026-11-24 Merge remote-tracking branch 'pr/167' into integrate/all
4456829 2026-11-24 Merge remote-tracking branch 'pr/161' into integrate/all
d3a6297 2026-11-24 Merge remote-tracking branch 'pr/153' into integrate/all
a05662a 2026-11-24 Merge remote-tracking branch 'pr/151' into integrate/all
86fd469 2026-11-24 Merge remote-tracking branch 'pr/150' into integrate/all
0a59417 2026-11-24 Merge remote-tracking branch 'pr/148' into integrate/all
8227afa 2026-11-24 Merge remote-tracking branch 'pr/141' into integrate/all
f71de9e 2026-11-24 Merge remote-tracking branch 'pr/140' into integrate/all
97e170c 2026-11-24 Merge remote-tracking branch 'pr/137' into integrate/all
ef71bc8 2026-11-24 Merge remote-tracking branch 'pr/133' into integrate/all
231213c 2026-11-24 Merge remote-tracking branch 'pr/126' into integrate/all
5eee1ae 2026-11-24 Merge remote-tracking branch 'pr/124' into integrate/all
d803006 2026-11-24 Merge remote-tracking branch 'pr/115' into integrate/all
be19d01 2026-11-24 Merge remote-tracking branch 'pr/111' into integrate/all
3f11e34 2026-11-24 Merge remote-tracking branch 'pr/96' into integrate/all
032fa09 2026-11-24 Merge remote-tracking branch 'pr/93' into integrate/all
5397ded 2026-11-24 Merge remote-tracking branch 'pr/88' into integrate/all
8570386 2026-11-24 Merge remote-tracking branch 'pr/87' into integrate/all
fd4b2fc 2026-11-24 Merge remote-tracking branch 'pr/76' into integrate/all
f5a81dd 2026-11-24 Merge remote-tracking branch 'pr/72' into integrate/all
798e671 2026-11-24 Merge remote-tracking branch 'pr/31' into integrate/all
431f0a4 2026-11-24 Merge remote-tracking branch 'pr/26' into integrate/all
a75e290 2026-11-24 Merge remote-tracking branch 'pr/12' into integrate/all
63df363 2026-11-24 GPU: a half-float rounding carry raises the exponent, not halves the value
f0e7745 2026-11-24 Shaders: address PR #176 review comments on half_inputs
01c3a5a 2026-11-24 Merge branch 'fix/spill' into test/nuit
48b6a5c 2026-11-24 Shaders: leave SV_Barycentrics unpacked; tally the new descriptor and interpolation kinds
5ff639e 2026-11-23 Shaders: descriptor dwords at joins and images read only for their size
db20637 2026-11-24 Shaders: fix v_div_scale_f32 src1/src2 order (den, num) and zero->NaN
5cff000 2026-11-24 XAsync: a new block where a result was left behind begins a call
7f95b87 2026-11-24 Shaders: share and serialize WARP host operations across threads
79fb722 2026-11-24 Shaders: s_ttracedata_imm is SOPP opcode 0x28, not 0x26
754f401 2026-11-23 GPU: run on the fastest adapter and log it; name buffers in unordered views
44ab54d 2026-11-24 Shaders: forward SGPRs reloaded from VGPR lanes as the spilled value
1d5e1bf 2026-11-24 Merge the night's shader fixes into the test stack
36d6a79 2026-11-24 XTaskQueue: terminating a queue ends a dispatch waiting on it
c7d3f7d 2026-11-24 Shaders: pass the non-uniform flag in the descriptor size handle
f78d243 2026-11-24 Merge fork branch memory/world-load-commit (bb1d5d6)
f435ff5 2026-11-24 Merge branch 'fix/waterfall' into stack/shader-fixes-on-d2a4bae
34e0a70 2026-11-24 Merge branch 'fix/interp' into stack/shader-fixes-on-d2a4bae
7265593 2026-11-24 Shaders: a16 packs an image's coordinates, not its derivatives
01c900d 2026-11-24 Shaders: lower waterfall loops over per-lane descriptors in one pass
9f77a9a 2026-11-23 feat(hle): convolve DSP reverb streams on the host
bb1d5d6 2026-11-24 Memory log: host textures and placed textures
1f7f7d2 2026-11-24 GPU: make ExecuteIndirect clamping countBuffer-aware and non-destructive
203d9f3 2026-11-24 HLE: offload async saves, reuse audio buffers, pin APU heap, and decouple input rumble/dialogs
1343a0b 2026-11-24 Merge fork branch memory/world-load-commit (e63b96a)
d105484 2026-11-24 Merge fork branch fix/gpu-hang-texture-lifetime (7dddec2)
27d18d5 2026-11-24 gpu: expand rectangle lists of indirect draws on the GPU
5f670bb 2026-11-24 Loader: a trapped unimplemented import goes into the crash log
a5af45f 2026-11-24 Shaders: fix v_mad_i64_i32 65-bit carry, refuse clamp, add -3*5+0 test
97cdea7 2026-11-24 Shaders: interpolate with computed barycentrics from the vertex values
7dddec2 2026-11-24 GPU: free textures and slot resources only after running submissions
727e884 2026-11-24 Shaders: DPP row masks name the console's rows in split programs
5eac4fa 2026-11-24 Shaders: wave64 compute whose halves exchange lanes, on NVIDIA
6ccf47b 2026-11-24 Shaders: run in-half wave64 compute as two 32-lane waves
d2715a1 2026-11-24 Shaders: pick the test adapter by name
f43117c 2026-11-24 Docs: plan for console wave64 compute on 32-lane hosts
e63b96a 2026-11-24 Memory log: split private commit by title range and allocation size
c153ae6 2026-11-24 HLE: overlapped DirectStorage I/O and persistent xmem section views
c7bd0dc 2026-11-24 Shaders: match a descriptor's size load through a rebuilt table pointer
f0393dd 2026-11-24 XAsync: a canceled pending call's work touches neither freed state nor the next call
6a96adb 2026-11-24 GPU: skip unchanged texture uploads via prelude copy recording
b87a968 2026-11-23 HLE: report the host's commit headroom before the title starts
acdcc41 2026-11-24 Loader: suspend a thread before reading its TEB; skip bad dump times
e0865d2 2026-11-24 Shaders: translate for the host's wave sizes
d46b078 2026-11-24 XUser: a gamertag without a suffix has an empty modern suffix
e57e76a 2026-11-23 fix(pe): validate section raw data bounds
200943e 2026-11-23 feat(hle): support APU decode-convert and PCM rate conversion
09ff84b 2026-11-24 XGameSave: a failed create leaves a null handle, as documented
d38b73f 2026-11-24 Merge fork branch stack/shader-fixes-on-d2a4bae (2a10e49)
18b2716 2026-11-24 net: rustfmt the presence assertion
afc4641 2026-11-24 net: skip newer-Windows-only exports in only_loopback_is_reachable
79bb95e 2026-11-24 Merge remote-tracking branch 'origin/main' into pr31-merge
2f3c4b2 2026-11-24 shader-info: print the runtime's hash per container, filter with VICTORY_SHADER_HASHES; diag: name indirect rectangle-list drops
2a10e49 2026-11-24 Merge branch 'fix/wholewave' into stack/shader-fixes-on-d2a4bae
c263e28 2026-11-24 Merge fix/gather onto the shader stack
0f7052e 2026-11-23 GPU: reuse the read buffer for watched textures
a8bfe78 2026-11-24 Shaders: follow lane writes and scalar steps in whole-wave sections
6269b57 2026-11-24 Loader & HLE: harden VEH crash handler, PE prep/exports/TLS, and allow oleaut32
c88b7ac 2026-11-24 Game saves: an empty configuration id is the default, not every configuration's folder
80883b7 2026-11-24 Shaders: trace scalar addresses kept in phis
4a3c9f1 2026-11-24 GPU: an occlusion query around a draw the host skipped keeps its stand-in
56d3a5d 2026-11-24 GPU: occlusion, statistics and timestamp queries run on the host
728a8e3 2026-11-24 GPU: the depth bounds test reaches the host
11ca3ce 2026-11-24 Media: a share is refused however it is spelled
6e01ea0 2026-11-23 Shaders: test that an image read as float and used by an atomic is annotated twice
05981dd 2026-11-24 GPU: make a structured buffer view off its host copy's elements a raw view
cbb50b8 2026-11-23 GPU: give null texture UAVs the title's view dimension
7f00a62 2026-11-24 Crash reports: name the host's CPU and Windows build
dcf6664 2026-11-23 GPU: skip unmapped regions after a failed bulk read into host memory
416ac79 2026-11-24 Shaders: read pixel barycentrics as values through SV_Barycentrics
6cf5090 2026-11-23 GPU: larger dynamic descriptor space; skip draws whose tables have no slots
cf1f4d4 2026-11-24 Merge fork branch memory/world-load-commit (687e7c9)
3e8f2d3 2026-11-24 Shaders: lower v_div_scale_f32, v_div_fmas_f32 and v_div_fixup_f32
42ac20c 2026-11-24 diag: VICTORY_PRESENT_TARGET presents a chosen render target in place of the title's image
687e7c9 2026-11-24 GPU: share a made copy of title memory instead of copying its bytes again
c63262f 2026-11-24 Merge upstream main into startup breakpoint diagnostics
5e72a8f 2026-11-24 Log where the commit charge goes every 5 seconds
3b039fd 2026-11-24 translate: a selected descriptor source is no leaf either (stack fix for the shader-fixes branch)
bccb4d9 2026-11-24 Merge fork branch fix/gpu-hang-split-submissions (082e9a2)
5df733f 2026-11-24 Merge fork branch stack/shader-fixes-on-d2a4bae (9707daf)
3390e47 2026-11-24 Crash reports: say what an access violation's address holds
082e9a2 2026-11-24 GPU: submit each host list on its own, wait once at the end
3c0b609 2026-11-24 GPU: specialize packed position and color rectangle expansion
985d8d4 2026-11-24 Audio: an XMA loop's skip is of its first frame, not of its last
9707daf 2026-11-24 Merge descriptor select tracing onto the shader stack
538d2de 2026-11-24 Merge PR #109 onto the shader stack
655d996 2026-11-24 Merge remote-tracking branch 'pr/120' into stack/approx
8c19657 2026-11-24 Merge PR #116 onto the shader stack
2a6e6a4 2026-11-24 Loader: find TLS-array load candidates sixteen offsets at a time
ca4e0f9 2026-11-24 Merge PR #92 onto the shader stack
48dcdde 2026-11-24 Merge PR #91 onto the shader stack
3c7d845 2026-11-24 Kernel: a title image's file name that fits clears the last error
aa369b1 2026-11-24 Merge PR #78 onto the shader stack
48b6f5c 2026-11-24 Merge PR #69 onto the shader stack
4322b61 2026-11-24 Shaders, GPU: add runtime stride table for structured buffers (#155)
ce9c884 2026-11-24 Merge remote-tracking branch 'fork/fix/sgpr-address-not-root' into stack/approx
9a0c483 2026-11-24 Merge remote-tracking branch 'pr/20' into stack/approx
d503d4c 2026-11-24 Merge remote-tracking branch 'pr/86' into stack/approx
da79a33 2026-11-24 Merge remote-tracking branch 'pr/123' into stack/approx
ac53b1b 2026-11-24 Merge remote-tracking branch 'pr/114' into stack/approx
c1f430f 2026-11-24 Merge remote-tracking branch 'pr/122' into stack/approx
8137ca0 2026-11-24 Merge remote-tracking branch 'pr/119' into stack/approx
4225b64 2026-11-24 Merge remote-tracking branch 'pr/104' into stack/approx
13e61a2 2026-11-24 Merge remote-tracking branch 'pr/117' into stack/approx
fe602fd 2026-11-24 Merge remote-tracking branch 'pr/110' into stack/approx
9520a19 2026-11-24 Merge remote-tracking branch 'pr/105' into stack/approx
622dd40 2026-11-24 Merge remote-tracking branch 'pr/101' into stack/approx
18f3165 2026-11-24 Merge remote-tracking branch 'pr/98' into stack/approx
c8b6aa3 2026-11-24 Merge remote-tracking branch 'pr/73' into stack/approx
6f475a9 2026-11-24 Merge remote-tracking branch 'pr/67' into stack/approx
145faad 2026-11-24 Merge remote-tracking branch 'pr/35' into stack/approx
0406c6c 2026-11-24 Merge remote-tracking branch 'pr/52' into stack/approx
9f6e765 2026-11-24 Merge remote-tracking branch 'pr/48' into stack/approx
c47b922 2026-11-24 Merge remote-tracking branch 'pr/55' into stack/approx
e1d9f29 2026-11-24 Merge remote-tracking branch 'fork/fix/shader-gaps-combined' into stack/approx
d4d1822 2026-11-24 XMA: noise tests, and end a frame at its data instead of looping forever
1827532 2026-11-24 Files: `..` in a title path stops at the virtual drive's root
babe2db 2026-11-24 Crash reports: show the bytes of an instruction the CPU refused
0434f3d 2026-11-24 Merge fix/sgpr-address-not-root
383bcb5 2026-11-24 Merge origin/main (d2a4bae)
83963c5 2026-11-23 GPU: maintain shadow-buffer coherency for directly indexed SRVs
d83f807 2026-11-24 shader: read root parameters a program loads from the user-data spill table
c78056c 2026-11-23 Shaders: twin descriptor loads and quad gradients for compute samples
3248948 2026-11-24 Shaders: index interpolation halves on demand
24d7c6c 2026-11-24 Shaders: trace descriptors chosen by s_cselect and s_cmov
f5f4ad9 2026-11-24 Shaders: benchmark the I/O raising pass
a823dfb 2026-11-24 Shaders: pair packed fp16 pixel inputs with unlisted signature rows
e55eb69 2026-11-24 Shader: key heap handles by element type too
810f442 2026-11-24 GPU: a frame wait takes its tick before it sleeps
60b63b4 2026-11-24 Shaders: lower v_mad_u64_u32 and v_mad_i64_i32
4eb0898 2026-11-24 XMA: build the inverse transform's rotations once per subframe size
02fcf72 2026-11-24 xmem: mapping or unmapping no pages changes nothing
318082f 2026-11-23 Shaders: lower v_sad_u8, v_sad_hi_u8, v_sad_u16, v_sad_u32 and v_msad_u8
04acee2 2026-11-24 Shaders: lower s_bitset{0,1}_b64, s_brev_b64 and s_bitreplicate_b64_b32
800227d 2026-11-23 Merge main: keep both test helpers in the lowering tests
7d73e2b 2026-11-24 pipeline: remove staging files when cache publication fails
13d984a 2026-11-24 Shaders: lower 24-bit mul_hi, saturating 16-bit packs, integer dot products and v_ldexp_f32
8142a94 2026-11-23 Shaders: lower v_mul_hi_u32_u24 and v_mul_hi_i32_i24
d5fca30 2026-11-24 Shaders: use DXIL IsNaN for unordered/ordered and self float compares
03a9a15 2026-11-24 GPU: seek directly to private data on final release
ff95daf 2026-11-23 Shaders: test gradient samples on planes and cubes
3bbc9ba 2026-11-24 Shaders: lower integer memory atomics and sub-dword buffer accesses
ce1f138 2026-11-24 Shaders: lower scalar and vector arithmetic and compares
be7369a 2026-11-24 Shaders: texel offsets, comparison and gather forms of image samples
f96b197 2026-11-24 Shaders: an audit of which opcodes lower
d2a4bae 2026-11-24 Merge pull request #129 from Luislariosss/fix/duplicate-test-module
d77408b 2026-11-23 Link victory.exe with a 16 MiB default thread stack
4c5545a 2026-11-24 Merge fork branch fix/gpu-hang-with-pr32 (new tip)
34a0823 2026-11-24 Input: read DualSense pads over HID when XInput has none
9dac364 2026-11-24 GPU: age retired host resources in presented frames, not the title's 60 Hz clock
d566e33 2026-11-24 GPU: drop dispatches that write through a null unordered access view
ecc4eb6 2026-11-24 Shaders: read both ballot halves for wave32 host-wave mask ops
a038004 2026-11-23 Shaders: lower s_bcnt0_i32_b32/_b64, s_ff0_i32_b32/_b64 and s_flbit_i32_b64/_i64
4f49a25 2026-11-24 list: pass the host list by reference to texture upload in readbacks (stack fix: #67 on top of the PR32 branch)
652f060 2026-11-24 GPU: hash and detile unwatched textures from a kernel-read copy
8fb5435 2026-11-24 Merge branch 'fix/shader-gaps-combined' of https://github.com/AntoninMichelq/VIctory into victory-stack
2f6efc1 2026-11-24 HLE: find an import's implementation by name, not by scanning every module
6d8d197 2026-11-23 GPU: drop a draw whose table found no dynamic slots; more dynamic slots
a103937 2026-11-24 Shaders: tbuffer_load_format_d16 loads pack two 16-bit channels per VGPR
c74bb8d 2026-11-24 Merge fork branch memory/world-load-commit
78b3463 2026-11-24 Merge fork branch fix/gpu-hang-with-pr32
8507e54 2026-11-24 Merge branch 'pr20' into victory-stack
1823079 2026-11-24 Shaders: lower v_cmp_class_f16 and v_cmpx_class_f16
92f7127 2026-11-24 Merge branch 'pr116' into victory-stack
aa31d2b 2026-11-24 Merge branch 'pr92' into victory-stack
ae8967c 2026-11-24 Shaders: SCC of s_min, s_max and s_lshl1-4_add_u32 as the ISA sets it
492e948 2026-11-23 Shaders: gradient samples on cube images
dea90cc 2026-11-24 xmem: map a run of physical pages over several placeholders
efe8a76 2026-11-24 Merge branch 'pr91' into victory-stack
a6731dd 2026-11-24 Merge branches 'fix/address-from-root-param' and 'fix/scalar-read-as-data' into combo-test
2fd2a92 2026-11-24 Merge branch 'pr78' into victory-stack
1c5493f 2026-11-24 Shaders: reuse SSA sums when tracing memory addresses
0214df8 2026-11-24 Shaders: read scalar loads from root views and repeated descriptor loads as data
8ae232f 2026-11-24 Shader: support s_buffer descriptor/structured loads, dead descriptor phis, PSV0/ISG1 fixes, calculateLOD and image_gather4_l
d112889 2026-11-23 Shader: support a16 biased/clamped/compare image samples and conservative depth centroid position
6cb36b7 2026-11-24 GPU: expand dynamic descriptor slots, use typed nulls, and drop draws missing fresh tables or root views
a78c323 2026-11-24 GPU: defer dropping replaced host resources across frames
e136a55 2026-11-24 Reduce host memory committed while loading the world
26556d2 2026-11-24 Shaders: trace addresses offset by root constants, s_addk_i32 and s_sub
1c96414 2026-11-24 Merge branch 'pr86' into victory-stack
f352a03 2026-11-24 Merge branch 'pr123' into victory-stack
d29f899 2026-11-24 Merge branch 'pr114' into victory-stack
baa8c70 2026-11-24 Merge branch 'pr122' into victory-stack
5c7de66 2026-11-24 Shaders: index buffers built around root descriptors by their own stride
45407c7 2026-11-24 Merge branch 'pr119' into victory-stack
6d45a26 2026-11-24 Merge branch 'pr104' into victory-stack
b98c32d 2026-11-24 Merge branch 'pr89' into victory-stack
09c970d 2026-11-24 Merge branch 'pr117' into victory-stack
50ab30c 2026-11-24 Merge branch 'pr110' into victory-stack
663953c 2026-11-24 Merge branch 'pr105' into victory-stack
908b511 2026-11-24 Merge branch 'pr101' into victory-stack
430b0a8 2026-11-24 Merge branch 'pr98' into victory-stack
b658e5c 2026-11-24 Merge branch 'pr73' into victory-stack
bd3b654 2026-11-24 Merge branch 'pr69' into victory-stack
6692d75 2026-11-24 Merge branch 'pr67' into victory-stack
72af95d 2026-11-24 Merge branch 'pr35' into victory-stack
9b46bde 2026-11-24 Merge branch 'pr52' into victory-stack
f853f34 2026-11-24 Merge branch 'pr48' into victory-stack
3a5786d 2026-11-24 Merge branch 'pr55' into victory-stack
6b867e6 2026-11-24 GPU: keep the indirect budget from wrapping; drop dispatches past the group limit
cf42807 2026-11-24 GPU: pool detached host lists and preserve pending list lifetimes
a8dcf2f 2026-11-24 GPU: wake fence events and held queues on direct memory writes
40d8ca5 2026-11-24 GPU: order CopyPageMappingsBatchX behind a queue's timeline
b85061f 2026-11-24 GPU: order Present behind a queue's timeline
f1ffee9 2026-11-24 GPU: hold a queue's work back behind a fence wait
e969948 2026-11-23 Fences: block on a null-event wait, wake waits on bottom-of-pipe writes
7102a49 2026-11-23 GPU: rename the second test module in host.rs so the tests compile
219e3ce 2026-11-23 fix(gpu): preserve pixels outside texture readback regions
da580e4 2026-11-23 gpu: packed 16-bit render targets are R8G8B8A8 where the GPU cannot render to them
01ae55a 2026-11-24 Files: a title run by its file name alone gets its own folder as G:
9aef3d9 2026-11-24 Merge pull request #51 from BaggyALPHA/fix/windows-commit-limit-diagnostic
8a8a38d 2026-11-24 Merge pull request #53 from a55uka/fix/wine-physical-pages
4c0bb89 2026-11-24 Merge pull request #41 from LaunchDoesntCode/codex/proton-warp-safety
91e9bc9 2026-11-24 Merge pull request #46 from 9hvn9yt2vk-netizen/shader-loop-limit
b9ff219 2026-11-24 XTaskQueue: terminating a queue cancels its delayed callbacks too
e5b7159 2026-11-24 Shaders: walk a lane-mask diagram once in vars and given
6168fc6 2026-11-24 Merge pull request #70 from FIocker/gpu/diagnostic-pipeline-filter
0322732 2026-11-24 Merge remote-tracking branch 'origin/main' into fix/med3-dxil-isnan
97464ba 2026-11-24 Merge pull request #61 from TG0000/hle/physical-page-runs
93bc978 2026-11-24 Merge pull request #60 from ifBars/perf/reuse-texture-row-spans
fa5ef73 2026-11-24 Merge pull request #79 from TG0000/perf/texture-check-once-per-submission
0f35dfc 2026-11-24 Merge pull request #66 from AndyLatimer/fix/gpu-rect-element-past-stride
fc0c4dd 2026-11-24 Merge pull request #77 from Exorion1er/fix/guard-fault-race
828e3cd 2026-11-24 GPU: refuse raw buffer views that do not start on 16 bytes
095e44a 2026-11-23 Query allocation type directly when guarding mapped memory
683278b 2026-11-24 Merge pull request #36 from jeremygold02/codex/root-parameter-fixture
cb40cdc 2026-11-24 Shaders: lower s_absdiff_i32, s_sext_i32_i8/i16 and s_pack_ll/lh/hh_b32_b16
a6deb46 2026-11-24 GPU: reuse unchanged page checks across shadow copy-back ranges
eda38aa 2026-11-24 Merge pull request #57 from Yaekai/fix/null-buffer-views
3c86053 2026-11-24 Shaders: lower s_bitcmp0/1_b32 and s_bitcmp0/1_b64
237f2e8 2026-11-24 Merge pull request #37 from alessiodeia/fix/valu-med3-fract-trig
b1bdd89 2026-11-23 Census: report directory traversal failures
0b729de 2026-11-24 XAsync: run a failed call's cleanup without the completion lock
4926f72 2026-11-24 Merge pull request #80 from tedin7/vic-far-jump-backward
46610eb 2026-11-24 Merge pull request #81 from tedin7/vic-scalar-carry
d01b170 2026-11-24 XAsync: end the call that finished, not whatever call the block holds now
8504777 2026-11-23 Shaders: byte and halfword LDS reads and writes
a5837b8 2026-11-24 Shaders: use DXIL IsNaN in med3 so Intel Xe does not fold fcmp uno
4a8a058 2026-11-24 Merge pull request #58 from Yaekai/feat/shader-xnor-swap
8fe0b74 2026-11-24 Merge pull request #56 from 9hvn9yt2vk-netizen/find-input-underflow
a6888a6 2026-11-24 Merge pull request #43 from 9hvn9yt2vk-netizen/shader-mask-cycle
062ad14 2026-11-24 Merge pull request #50 from Vanthys/fix/format-offset-overflow
bf6c557 2026-11-24 Merge pull request #75 from tedin7/vic-pe-export-holes
cc51927 2026-11-24 Merge pull request #30 from AndyLatimer/fix/pe-rva-offset-overflow
cbd71a9 2026-11-24 Merge pull request #38 from raffclar/emulation-tests
24480a7 2026-11-24 Shaders: find the written instructions with a worklist, not repeated passes
c78ccf7 2026-11-24 Shaders: lower sub-dword LDS reads and writes
27e6442 2026-11-24 Merge pull request #28 from ifBars/fix/windows10-import-directory
27b530e 2026-11-24 Files: send rewritten narrow paths to the host, not to our wide wrappers
743b1e2 2026-11-24 Shaders: rebuild a cube image descriptor for code that reads its dwords
7d33c46 2026-11-24 Merge pull request #45 from HenriqueSBueno/clwb
f468457 2026-11-24 fix: model v_permlane16_b32 in whole-wave sections
ba42db6 2026-11-24 xma: start the frame after a continuation at the next packet that has frames
df40f86 2026-11-23 GPU: report the view heap sizes when the host caps one
6e6d3fc 2026-11-24 fix: accept shader loops that branch to the first instruction
3e982f7 2026-11-24 BCPack: find a slot's bucket from a window table
4f2d5db 2026-11-24 shader: lower v_add_nc_i32, v_sub_nc_i32, v_mul_lo_i32 and v_add_nc_i16, v_sub_nc_i16
0e5953d 2026-11-23 Merge branch 'main' into fix/windows10-import-directory
38ae55d 2026-11-24 Keep the diagnostic skip outside indirect argument validation
ed5f98f 2026-11-24 Shaders: lower s_cmov_b32, s_cmov_b64 and s_cmovk_i32
b5c38fa 2026-11-24 GPU: skip texture-to-buffer readback when host texel size differs
ec0d155 2026-11-24 Watch: saturate the end of announced writes and remaps
257d8bb 2026-11-24 Audio: reduce the title's XMA read advance before adding it
b304b7d 2026-11-23 fix(gpu): average copy statistics over the full reporting interval
a80eb17 2026-11-24 GPU: cap a sampler heap past the host's limit instead of losing it
a2f5f43 2026-11-24 Shaders: set SCC from the carry of s_addc_u32 and s_subb_u32
c963b71 2026-11-24 fix: read a far jump's offset as signed so backward far jumps decode
73d776b 2026-11-24 GPU: check a sampled texture once per submission, not once per list
e6faad2 2026-11-24 Shaders: inline float constants as f16 in v_fma_mix and v_pack_b32_f16
4c1766c 2026-11-24 GPU: keep a guarded block's write fault from being declined in a race
9a5cbbe 2026-11-24 fix: skip unused export address slots instead of reporting them at RVA 0
c4e915f 2026-11-23 loader: a clwb write-back loop runs in one exception
6c8af09 2026-11-24 Shaders: support image_sample_b with a16 and _cl LOD-clamp variants
521cecc 2026-11-24 GPU: retry pipeline cache saves after write failures
0eca412 2026-11-24 GPU diagnostics: skip pipelines by shader hash
18f3937 2026-11-24 Translate compute selects with numeric lane masks
56f1d41 2026-11-23 GPU: create host textures without the uses the host refuses
78b8404 2026-11-23 xmem: commit physical-page sections whole under Wine
25a3e95 2026-11-23 Keep an empty import descriptor so Windows assigns static TLS
f3c7f0f 2026-11-24 GPU: leave rectangle vertex components past the stride alone
56f0072 2026-11-24 Translate 64-bit texture atomics and specialize proven shader branches
1ac256c 2026-11-24 GPU: implement CopyTextureRegion texture-to-buffer readbacks
35b054e 2026-11-24 GPU: pipelines off the render thread and cached; indirect execution on
fc807b6 2026-11-23 Reuse texture row spans when converting Xbox pixel layouts
4e94c2b 2026-11-24 xmem: write DirectStorage page destinations one view per run
0c56091 2026-11-24 Shaders: lower v_xnor_b32 and v_swap_b32
071487f 2026-11-24 GPU: emit null buffer SRV/UAV when descriptor names a buffer with null resource
5fbea47 2026-11-24 shader: look up an input channel without underflowing
4da3d63 2026-11-23 Clear the IAT directory alongside the import directory
2e5001c 2026-11-24 shader: keep fract(NaN) as NaN on NVIDIA and add synthetic GPU VALU tests
628b95d 2026-11-24 GPU: hold SHADER_HEAPS across descriptor mirror and check query offsets
ef22723 2026-11-24 PE: remove leftover dbg_tls test and check layout offsets
7e42fef 2026-11-23 Report Windows commit-limit failures in crashes.log
4cca767 2026-11-24 Reject empty XMA test fixture directories
e158d95 2026-11-24 Guard texture and XMA offsets against overflow
15b0091 2026-11-24 shader: VICTORY_SHADER_LOOP_LIMIT to find what makes world frames slow
71cfe84 2026-11-23 loader: emulate clwb on Intel CPUs before Ice Lake
ea24ea1 2026-11-24 shader: a lane mask that needs its own value fails instead of overflowing the stack
0b607b1 2026-11-23 GPU: verify WARP adapter is software before using it
f00caec 2026-11-23 Loader: test 64-bit extrq at index 0, not in AMD's undefined range
97c78d1 2026-11-24 Merge pull request #21 from Nostyxx/fix/nvidia-view-heap-cap
d222b87 2026-11-24 Merge pull request #19 from raffclar/exit-watchdog
0c91c03 2026-11-23 Loader: test the AMD-only instruction emulation on any CPU
9e04419 2026-11-24 Merge pull request #15 from 9hvn9yt2vk-netizen/fix-texture-extents
29b558c 2026-11-23 test(gpu): use the SDK root parameter in the serializer fixture
d85097a 2026-11-23 Keep an empty import descriptor for the Windows 10 loader
b3ecede 2026-11-24 HLE: leave a released-out COM count at zero
e98d3ca 2026-11-23 PE: checked file offset in rva_to_file_offset
1dd7afd 2026-11-24 Merge pull request #14 from 9hvn9yt2vk-netizen/fix-xma-zero-length
4b76fff 2026-11-23 Run indirect mesh dispatches through indirect execution
886d15b 2026-11-23 Input: pass the trigger motor levels on to the host
3827767 2026-11-24 GPU: make tables past a capped host view heap afresh
a68ac08 2026-11-24 Keep SRV descriptor tables stable until command list reset
8c26061 2026-11-24 GPU: the world stays up; indirect execution behind a switch
0b499d7 2026-11-24 Shaders: sine and cosine in turns, exact zeros at half and quarter turns
36eb0b4 2026-11-23 Shaders: med3 with NaN, fract below 1.0, signed bit fields past bit 31
e6a2c54 2026-11-24 texture: refuse extents past 16384 texels instead of overflowing
d348b9a 2026-11-24 Merge pull request #10 from ismailchaka1/emulate-amd-only-instructions
d4ed2de 2026-11-23 Process exit: start the watchdog before other threads end
005ad49 2026-11-24 xma: stop at a zero-length frame instead of looping forever
a3acc87 2026-11-23 Check the required startup archive before executing Series 1.0.2.1
f5cb5ff 2026-11-23 Capture startup breakpoint diagnostics
934e67f 2026-11-23 loader: emulate AMD-only instructions on Intel CPUs
e8f3b1c 2026-11-23 Merge pull request #4 from Exorion1er/fix/tls-callbacks-reloc
c21208c 2026-11-23 Loader: keep the cleared TLS callback pointer out of base relocations
015d45b 2026-11-23 README: status, subsystem table, how to run, debugging switches
2849477 2026-11-23 Run Grand Theft Auto VI from first boot into the world
bc30709 2026-11-23 Thread notices: leave out threads Rust's std starts
4b25269 2026-11-23 Thread notices for title images
d032b8a 2026-11-23 Title DLLs at run time: find by name, map on first LoadLibrary
754619f 2026-11-23 Run Minecraft: runtime HLE, D3D12.X device on the host GPU, shader IR
12caefe 2026-11-23 GFX10 operand fields and SP3 text; FSSM wave size
f4504f6 2026-11-23 Initial commit
```

## L. Limitacoes desta analise

1. **Sem execucao**: nada foi compilado ou rodado. Nenhum numero deste documento
   foi medido por mim; todos vem de mensagens de commit e do README do proprio
   autor. Onde o autor mediu, eu cito; onde ele nao mediu, marco.
2. **Sem acesso a rede**: as issues e PRs citados nas mensagens (#2, #8, #9,
   #11, #16, #23, #44, #234, #566, #706, #88 e os PRs de romainhedouin/PikPakPik/
   Key-rs) **nao foram abertos**. O que sei deles vem apenas do texto dos commits
   e do README. A lista de PRs mergeados e inferida dos assuntos de merge, nao de
   uma pagina do GitHub.
3. **Historico reescrito**: o clone da base (`Marvisak/VIctory`) nao esta
   disponivel em `repos/portwatch`, entao a afirmacao "o HEAD do Marvisak nao e
   ancestral" vem de `FORK_DIVERGENCIA.md` e nao foi reproduzida. Nao comparei
   hashes entre repos (seria invalido).
4. **Atribuicao de "commit que introduziu"**: em varios defeitos a causa e
   arquitetural (herdada do upstream re-commitado) e nao ha commit isolavel. Usei
   `git log -S` para tentar isolar (`quad_perm`, `VICTORY_HOIST_CLAMPS`,
   `VICTORY_ASYNC_SUBMIT`, `VICTORY_SWEEP_TILES`); quando o resultado foi vazio
   ou ambiguo, escrevi NAO VERIFICADO em vez de escolher um hash.
5. **Profundidade por amostragem**: 45 commits tem diff integral no pacote de
   evidencias; outros ~35 foram abertos por mim. Os 671 restantes entram no
   indice mas nao foram lidos linha a linha - em particular os ~180 merges de
   PRs de terceiros (DexNom, VItoriaBr, victory-fork e a integracao comunitaria),
   cujo conteudo eu so conheco pelo assunto e pelas mensagens de merge.
6. **Estado de runtime nao verificado**: D-17 (`VIEW_SLOTS` vs limite tier-2) e
   a associacao D-15 (`opt.rs` -> agua ausente) sao conclusoes de leitura de
   codigo; nao vi a falha acontecer.
7. **Ranking do PortWatch (77,4%) e "roda missoes"**: externos ao repositorio,
   NAO VERIFICADOS. O que o repositorio sustenta e: mundo iluminado com texturas,
   pessoas, veiculos, vegetacao, agua e ceu; 5-8 fps; saves carregam e salvam.
8. **Autoria**: "Claude" e "beloooo" assinam 337 dos 751 commits, mas muitos
   commits de agente declaram `Co-Authored-By: Claude ...` e `Claude-Session:`;
   nao tentei reconstruir a divisao real de trabalho humano/agente.
