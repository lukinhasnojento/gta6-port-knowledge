# Verificações independentes

Onze conclusões foram reconferidas no repositório, à mão, depois de escritas por um modelo. Uma
delas corrigiu um erro do próprio orquestrador.

Cada linha traz o comando que reproduz o resultado — nenhuma depende de confiar neste documento.

## Quadro

| # | Repositório | Alegação | Verificação | Resultado |
|---|---|---|---|---|
| 1 | Xbox2Windows | `22989d61` restaurou `d3d12_draw.inl` byte a byte | `git rev-parse` dos blobs | **CONFIRMADO** |
| 2 | Xbox2Windows | existe apenas 1 revert real | `git log --grep="^Revert"` | **CONFIRMADO** |
| 3 | GTA6-Asset-Info | off-by-one na contagem de luzes | soma aritmética | **CONFIRMADO** |
| 4 | GTA6-Asset-Info | 81 ocorrências de "SixNG" em 24 arquivos | `Select-String -AllMatches` | **CONFIRMADO** |
| 5 | GTA6-Asset-Info | `structure/04:59` declara 289, parcelas somam 333 | soma aritmética | **CONFIRMADO** |
| 6 | VIctory | `7102a49` conserta `mod tests` duplicado | `git show --stat` | **CONFIRMADO** |
| 7 | XCL | `STRUCTURED_BYTES_PER_CODE_BYTE = 128` | `git grep` em HEAD | **CONFIRMADO** |
| 8 | SixNG | `gamecipher.rs`/`rpf9toc.rs` só existem em branch | `git ls-tree` em HEAD e na branch | **CONFIRMADO** |
| 9 | VItoriaBr | README contradiz a si mesmo sobre TDR | leitura de `README.md` | **CONFIRMADO** |
| 10 | Belobug-AMD | pipelines legítimos excluídos para sempre | comentário no código | **CONFIRMADO** |
| 11 | AvatarManz | porte verbatim de código GPL-2.0-only | cabeçalho do arquivo + LICENSE | **CONFIRMADO** |
| — | DexNom | *(orquestrador afirmou que nenhum fork descende do VIctory)* | `git merge-base --is-ancestor` | **ERRO DO ORQUESTRADOR — corrigido** |

## Detalhamento

### 1. O revert do viewport restaurou o arquivo exatamente

```
22989d61 (pós-revert)  : 98c69432c7daa93bb2716980c625b76c8831ea0d
1f24626b^ (pré-mutação): 98c69432c7daa93bb2716980c625b76c8831ea0d   ← idêntico
1f24626b  (mutado)     : 9a649fe4f35fb70b52fc0b4f18a020fb29a1537b
```

Confirma que `22989d61` não é uma correção parcial: é restauração byte a byte.

### 2. Cinco "reverts" na lista, um real

A busca inicial por `-i --grep="revert"` retorna cinco commits, mas quatro apenas usam a palavra
no corpo. Filtrando por mensagem que **começa** com `Revert`, sobra um: `22989d61`.

Isso mudou a leitura do projeto: o "MUTATION (temporary): disable default viewport" não era
defeito acidental, era instrumentação de teste — prova *fail-first* de um teste criado 7 minutos
antes. O defeito real estava no commit anterior, `ffb70f6a`, que marcou `viewport_set = true` ao
encaminhar o viewport **default**, confundindo estado sintetizado com estado do título.

### 3. A soma que não fecha

`formats/15-lights.md:38-39`:

> "774,847 placed lights: 447,756 exterior, 326,183 inside interiors, 907 in unplaced prefab groups"

`447.756 + 326.183 + 907 = 774.846`. O documento afirma **774.847**. Um off-by-one de uma luz.

### 4. O rebrand que não propagou

```
arquivos com ocorrência de "SixNG": 24
ocorrências totais:                81
```

Nenhum commit tocou `formats/` ou `tool/` depois do commit inicial `dd592ba`. O rebrand de
"ferramenta" para "notas de formato" chegou a duas páginas e parou.

### 5. Inconsistência interna na camada mais rigorosa

`structure/04-pso-objectgraph.md:59-61` declara "289 slot values resolve" e lista as parcelas:
277 root ids de veículo + 41 de 56 valores do tipo `0xC48AB516` + 10 chaves keyref + 5 de 8 slots
de osso.

`277 + 41 + 10 + 5 = 333`. O total declarado é 289.

O agravante: o próprio `structure/00-INDEX.md:24-28` estabelece a regra *"a claim that does not
close arithmetically is a hypothesis, and any conclusion drawn from it is void until it does"*.

### 6. Árvore que não compilava

`73d776b` adicionou um segundo `#[cfg(test)] mod tests {` em `crates/victory-gpu/src/host.rs`.
Corrigido por `7102a49` — *"rename the second test module in host.rs so the tests compile"* — 2
linhas alteradas, **54 commits depois**.

### 7. Constante do limite heurístico

```
crates/xcl/src/gpu/recomp/mod.rs:641:  const STRUCTURED_BYTES_PER_CODE_BYTE: usize = 128;
crates/xcl/src/gpu/recomp/mod.rs:1497: if code.len() > STRUCTURED_BYTES_PER_CODE_BYTE * program.len() {
```

O commit associado é `69a892c`, *"oversized structured code falls back to pc blocks (#1307)"*.

### 8. A maior capacidade do projeto está fora da branch principal

```
git ls-tree -r --name-only HEAD                  → nada
git ls-tree -r --name-only origin/claude/rpf9-toc → crates/sixng-core/src/gamecipher.rs
                                                    crates/sixng-core/src/rpf9toc.rs
```

E `README.md:42` de `master` afirma: *"not read: the TOC is encrypted and SixNG does not decrypt
it"*. A funcionalidade existe, não está mergeada, e a documentação oficial diz que não existe.

### 9. Contradição sobre TDR no mesmo README

- `README.md:26`: *"até `6 fps`, **zero TDR**, `1.700-4.400 draws/frame`"*
- `README.md:75`: *"**Aviso de GPU (TDR):** entrar no mundo 3D em uma GPU real **ainda pode
  disparar reset** do driver de vídeo"*

Quarenta e nove linhas de distância.

### 10. Correção que assume o ambiente errado

`crates/victory-gpu/src/pipeline/library.rs:244-252`, comentário acima de `fn endless()`:

> *"while the world loads the process takes gigabytes by itself and every core is busy, and
> **four pipelines the driver makes in seconds were left out for good (20 s and 3 GiB; run amd1,
> 2026-11-25)**. A compile that never ends takes 8 GiB a minute."*

```rust
const ENDLESS: Duration = Duration::from_secs(60);
const GROWTH: u64 = 8 << 30;
```

O watchdog marcava pipelines como travados permanentemente; como o arquivo é persistente
(`crashed.txt`), a exclusão era definitiva. `7263339` subiu para 60 s / 8 GiB.

Este é o exemplo mais didático do acervo: a mesma classe de erro que o projeto corrigia em
terceiros, cometida em casa.

### 11. Porte verbatim de código incompatível com a licença

`README.md:108`:

> "[XCL](https://github.com/ivall/xcl) (GPL-2.0-only) was read for how the same title behaves
> under another layer; **no code of it is here**."

`crates/victory-core/src/console_insn.rs`, cabeçalho:

> "**Ported from XCL** (github.com/ivall/xcl, **GPL-2.0-only**): `insn.rs` (decoder, as
> `console_insn_decode.rs`, **unchanged**) and `eh.rs` `emulate_console_only`."

`LICENSE` do repositório: **GNU GPL v3**.

GPL-2.0-**only** não é compatível com GPL-3.0 — só a variante "or later" permite atualização.
Código copiado sem modificação não pode ser relicenciado sem autorização do titular. Ver
`NOTICE.md`.

### Erro do orquestrador, corrigido

Havia sido afirmado que o `HEAD` do `Marvisak/VIctory` não era ancestral de nenhum fork. O teste
por fork:

```
baseHead = 70564ae08c82d43df5fad97b08d34628b07b1d9d
Belobug-AMD    base é ancestral: NAO   merge-base: nenhum
DexNom         base é ancestral: SIM   306 commits à frente
VItoriaBr      base é ancestral: NAO   merge-base: nenhum
AvatarManz     base é ancestral: NAO   merge-base: nenhum
victory-fork   base é ancestral: NAO   merge-base: nenhum
```

Verdadeiro para quatro, falso para o `DexNom`. A generalização foi corrigida em
`docs/fatos/FORK_DIVERGENCIA.md`, onde a tabela substituiu a afirmação original.

Consequência prática, que continua válida: no `DexNom` existe proveniência por commit e
cherry-pick conceitual é viável; nos outros quatro, a única comparação confiável é por conteúdo
de blob.
