# Plano de consolidação — versão revisada

Documento de decisão para o repositório unificado. Substitui a recomendação anterior, escrita antes
do comparativo dos doze projetos estar completo. O que mudou e por quê está na seção 6.

## 1. Decisão

| Papel | Repositório | Por quê |
|---|---|---|
| **Fundação Windows** | `Marvisak/VIctory` | upstream canônico, Reaproveitamento 5, média 3,6 |
| **Fundação Linux** | `brcly/TuxBox` | melhor média (4,0), Reaproveitamento 5, único com camada Wine/vkd3d |
| **Linha de suprimento** | `DexNom/VIctory` | único fork com descendência linear — permite cherry-pick seletivo |
| **Não usar como base** | `avatarmanz/VIctory-7900XTX` | contaminado por licença GPL-2.0-only |

Nenhum repositório é base sozinho. A fundação é o upstream; os forks são fontes.

## 2. Por que a base é o VIctory e não o DexNom

A recomendação anterior dizia: *"Base: DexNom, porque é o único mergeável."* O raciocínio estava
invertido — **"mergeável" é uma qualidade da fonte, não da base**.

Como o DexNom descende linearmente do VIctory (`git merge-base --is-ancestor 70564ae HEAD` retorna
sucesso, 306 commits à frente), seus commits podem ser aplicados **seletivamente** sobre a base
canônica. O contrário não é possível: com base no DexNom, o dano já está assado.

| | Base DexNom | **Base VIctory + cherry-pick do DexNom** |
|---|---|---|
| Histórico | 306 commits de trabalho | 241 canônicos **+** os 306 do DexNom |
| `570a941` (170 arquivos sobrescritos) | **baked in** | aplicado commit a commit, com o dano pulado |
| Reaproveitamento (placar) | 4 | **5** |
| Correção de save `b8dbcba` | presente | vem junto no cherry-pick |
| Posição no ecossistema | fork de terceiro | upstream de que todos derivam |

Os outros quatro forks não aceitam cherry-pick em nenhuma direção: o HEAD do VIctory não é ancestral
deles e o único ancestral comum é a raiz `f4504f6`. Deles só se aproveita conteúdo de arquivo, nunca
histórico.

## 3. Ordem de trabalho

### Passo 0 — antes de qualquer feature: a correção de save

Sem isso o repositório nasce podendo apagar dados do usuário. Verificação por `git log --grep` em
todos os clones:

| Repositório | `c88b7ac` (config id vazio) | `b8dbcba` (`v01` residual) |
|---|---|---|
| VIctory | ✅ `27f9b76`, `c88b7ac` | ❌ **ausente** |
| Belobug-AMD | ✅ | ❌ **ausente** |
| victory-fork | ✅ | ❌ **ausente** |
| DexNom | ✅ | ✅ `26e03c3`, `b8dbcba` |
| VItoriaBr | ✅ | ✅ `d1da85a` |
| AvatarManz | ✅ | ✅ `70868f8` |

Ambos em `crates/victory-hle/src/modules/xgamesave.rs`. A primeira correção propagou para os seis
forks; a segunda, não — e é a que apaga um save já commitado quando existe uma pasta `v01` residual ao
lado do `v1` válido.

O commit `d1da85a` do VItoriaBr é o modelo de correção a seguir: enuncia o novo invariante (só `v<N>`
exatamente como `commit` escreve é versão) e declara o pior caso (sem versão legível, leitura e
atualização falham e nada é removido). Falha fechado.

### Passo 1 — infraestrutura de cache

Portar de `fxceless/victory-fork`: `crates/victory-shader/build.rs:51` varre os literais
`"VICTORY_` do próprio código-fonte, publica `VICTORY_SHADER_SWITCHES`, e
`translation_fingerprint()` (`emit/mod.rs:199`) entra na chave do cache (`shader_cache.rs:75`).

Portar **o mecanismo, não as 247 flags**. O valor está em a lista ser gerada do código: adicionar
uma flag nova já entra na chave sem ninguém lembrar.

Três equipes resolveram o mesmo invariante de formas diferentes — `build.rs` automático
(victory-fork), string `x2w-startup-shader-6` sincronizada em C++ e Python (`Xbox2Windows`,
`d3d12_draw.inl:201` e `gfx2hlsl.py:2176`), e hash de fontes com escopo local (`XCL`,
`XCL_PER_LANE_SOURCE_ID`). Só a primeira é estruturalmente à prova de esquecimento.

### Passo 2 — enxertos de correção

| Origem | O que | Ressalva |
|---|---|---|
| `Belobug` `03629f3` | `quad_pick` em toda leitura de quad no pixel shader — 17 linhas, helper lanes incluídas | correção **AMD**; verificar no lado NVIDIA antes de declarar resolvido |
| `DexNom` `bc3cd4b` | posse temporal dos bytes do convidado (`writeback.rs`) | depende do cherry-pick linear |
| `DexNom` BVH | emulação das instruções de BVH com referência Rust bit a bit, 311 casos | idem |
| `DexNom` `cd94d35` | marca de escrita por página emitida pelo tradutor, bitmap no host | idem |
| `DexNom` `texback.rs` | retilagem de textura escrita por shader | idem |
| `AvatarManz` `20b9834` | tradutor RDNA float: `v_max`/`v_min`, `v_mul_legacy`, `v_med3`, `v_fract` como instrução única do driver, com `bounded(v, 12)` | **portável** — ver seção 5 |

### Passo 3 — reaplicar o que a importação sobrescreveu

Três correções de terceiros que o `570a941` do DexNom apagou e que estão **abertas em HEAD**:

- `4fc9d70` — out-of-bounds / UB em `Staging::detile` (`texture.rs:925`)
- `be824d9` — corrida no `.tmp` de nome fixo da biblioteca de pipelines (`library.rs:678`)
- `274f698` — overflow de `u64` no `clwb_loop` (`crash.rs:176`)

### Passo 4 — reescrever do zero

`crates/victory-loader/src/console_insn.rs` e `console_insn_decode.rs`. A funcionalidade é
necessária: emulam `CLZERO`, `MONITORX`, `MWAITX`, `CLWB` e as instruções SSE4a que o executável do
jogo usa e que geram `STATUS_ILLEGAL_INSTRUCTION` em CPU Intel. A implementação é
GPL-2.0-**only** (porte verbatim do XCL) e não pode entrar numa obra GPL-3.0. Ver `NOTICE.md`.

Reimplementar a partir do comportamento observável, sem consultar o original.

### Passo 5 — regras de sustentabilidade, no primeiro PR

É a única dimensão em que nenhum dos doze projetos passou de 3 (nove têm 2). Sem isso, o repositório
unificado herda a falha de todos.

1. **Nunca excluir permanentemente por timeout.** O `crashed.txt` do Belobug continua excluindo
   pipelines depois de o limiar ser corrigido (`7263339` subiu 20 s/3 GiB para 60 s/8 GiB), e a
   única saída é o usuário apagar o arquivo. Exclusão precisa de expiração ou reavaliação.
2. **CI que bloqueia.** O victory-fork tem CI desde `8e79c80` e faz push direto em `main`, com
   *"this merged tree was built; its test suites were not run"* em seis commits de integração
   (`a26859f`, `9376fd6`, `f85101b`, `a253706`, `0b0074e`, `c36971b`). Infraestrutura contornada é
   pior que ausente: cria aparência de verificação.
3. **Controle negativo no fornecedor que você não usa.** `Xbox2Windows` `583178d4` gastou **464
   linhas de teste para 3 de produto** porque o defeito de `f32`/NaN só aparece em FXC sem `-Gis`, e
   o projeto usa DXC.

## 4. O que não herdar

| Origem | Item | Motivo |
|---|---|---|
| Belobug | sweep de tiles desligado (`9956958`) | cópia cresce até ~33 GB commitados |
| Belobug | `pipelines/crashed.txt` sem expiração | exclusão permanente de pipeline legítimo |
| Belobug | `view_slots()` acima de `PORTABLE_VIEWS` (`host.rs:514`, 1.000.000) | padding nunca criado, fallback mascara |
| Belobug | substituto da luz indireta | 6 tentativas, continua stand-in — não é correção |
| VItoriaBr | `VICTORY_FIRSTLANE_OWN=1` | 53/53 shaders mas corrompe offsets de AABB; crash em ~18 s |
| AvatarManz | `rest()` default `true` (`indirect.rs:294-304`) | um registro lixo esvazia a lista inteira e derruba todos os draws seguintes |
| AvatarManz | `console_insn*.rs` | licença |
| victory-fork | perdas de device sem causa em 4 configurações | abertas, sem causa estabelecida |
| todos | import monolítico de árvore | xbox-emulator `ff39277` (266.033 linhas), VItoriaBr `578d459` (32.334), DexNom `570a941` (170 arquivos) — destrói a auditabilidade |

## 5. Correção de escopo: a licença não bloqueia o tradutor

A recomendação anterior dizia que, para aproveitar `20b9834`, seria preciso reescrever
`console_insn*.rs` "antes". Isso confundia dois subsistemas distintos:

```
20b9834 toca:   crates/victory-shader/src/emit/{lower,mod,opt,probe_tests,valu_tests}.rs
                ← nenhum cabeçalho GPL/XCL

contaminado:    crates/victory-loader/src/console_insn.rs
                crates/victory-loader/src/console_insn_decode.rs
```

`console_insn` emula **instrução de CPU**; o tradutor converte **shader de GPU**. Arquivos diferentes,
subsistemas diferentes.

**O tradutor RDNA é portável.** O bloqueio atinge dois arquivos, e atinge também VIctory e XCL — a
contaminação vem do XCL original, sob GPL-2.0-only.

## 6. Histórico de revisão

Duas decisões da versão anterior foram corrigidas por evidência, não por opinião:

**Base trocada (DexNom → VIctory).** A primeira versão escolhia o DexNom por ser "o único mergeável".
O comparativo mostrou que a linearidade serve para **extrair** trabalho dele, não para servir de
fundação — e que basear nele assa o dano do `570a941` na árvore.

**Escopo da licença reduzido (projeto inteiro → dois arquivos).** A primeira versão dizia que pegar o
tradutor exigia reescrever `console_insn*.rs` antes. Verificação nos caminhos dos commits mostrou que
são subsistemas separados; o tradutor nunca esteve contaminado.

O que **não** mudou: `03629f3` continua sendo a melhor razão valor/linha do conjunto; o
`build.rs`/`translation_fingerprint` continua sendo a peça de infraestrutura mais valiosa; os três
consertos sobrescritos continuam precisando ser reaplicados; e nenhum default do Belobug deve ser
herdado.

Duas ressalvas foram **acrescentadas**: `quad_pick` é correção de helper lane AMD e precisa de
verificação no lado NVIDIA; e o mecanismo de fingerprint deve ser portado sem as 247 flags que o
acompanham no fork de origem.

## 7. Referências

- Placar e achados transversais: `docs/comparativo/COMPARATIVO_GERAL.md`
- Documentos individuais: `docs/comparativo/<repo>.md`
- Análises completas: `docs/analises/<repo>.md`
- Verificações manuais: `VERIFICACOES.md`
- Licenças e bloqueios: `NOTICE.md`
- Catálogo de 281 defeitos: `catalogo/defeitos.csv`
