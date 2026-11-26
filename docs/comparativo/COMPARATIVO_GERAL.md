# Comparativo geral — 13 repositórios

Placar consolidado dos treze projetos, com o que cada um acertou e errou, e o que isso significa para
o repositório unificado. Documentos individuais em `docs/comparativo/<repo>.md`.

## 1. Matriz completa

Notas de 0 a 5. **Eng** engenharia · **Ver** verificabilidade · **Doc** documentação ·
**Sus** sustentabilidade · **Reap** reaproveitamento.

| # | Repositório | Eng | Ver | Doc | Sus | Reap | Média |
|---|---|---|---|---|---|---|---|
| 1 | **TuxBox** | 4 | 4 | 4 | **3** | **5** | **4,0** |
| 2 | **xbox-emulator** | 4 | **5** | 3 | **3** | 4 | **3,8** |
| 3 | **SixNG** | 4 | 4 | 4 | 2 | **5** | **3,8** |
| 4 | Xbox2Windows | 4 | **5** | 4 | 2 | 3 | 3,6 |
| 5 | VIctory | 4 | 4 | 3 | 2 | **5** | 3,6 |
| 6 | XCL | 4 | 4 | 3 | 2 | 4 | 3,4 |
| 7 | DexNom | 4 | 4 | 3 | 2 | 4 | 3,4 |
| 8 | Belobug-AMD | 4 | 3 | 4 | 2 | 4 | 3,4 |
| 9 | VItoriaBr | 4 | 3 | 3 | 2 | 4 | 3,2 |
| 10 | victory-fork | 4 | **2** | 4 | 2 | 4 | 3,2 |
| 11 | GTA6-Asset-Info | 4 | 4 | **2** | 2 | 4 | 3,2 |
| 12 | OpenScarlett | 4 | 3 | **2** | 2 | 4 | 3,0 |
| 13 | **AvatarManz** | 4 | 3 | 2 | 2 | **0** | **2,2** |

Amplitude total: **1,8 ponto** entre o primeiro e o último. Nenhum projeto é ruim; nenhum é ótimo.

## 2. O que a matriz revela

### 2.1 Engenharia é 4 em treze de treze

A coluna mais estável do placar é a que mede qualidade técnica das correções. Treze projetos
independentes, autores diferentes, stacks diferentes — e todos chegam ao mesmo 4.

Isso não é coincidência nem complacência da avaliação. É o resultado de um filtro: quem conseguiu
fazer o jogo bootar sabe o que faz. A competência técnica não é o que separa esses projetos.

**O que separa é a coluna seguinte.**

### 2.2 Sustentabilidade nunca passa de 3

| Nota | Quantos |
|---|---|
| 3 | 3 (TuxBox, xbox-emulator) |
| 2 | 10 |
| 4 ou 5 | **0** |

Dez dos treze têm nota 2. E os casos não são abstratos:

- `crashed.txt` persistente cujo dano só se desfaz apagando o arquivo à mão (Belobug)
- *"this merged tree was built; its test suites were not run"* em seis commits (victory-fork)
- cópia crescente até 33 GB commitados (Belobug)
- árvore que não compila testes por 54 commits (VIctory)
- `mod oracle;` sem `oracle.rs`, escondido atrás de feature off-by-default (xbox-emulator)
- esgotamento de descritores que encerra o run mais longo (TuxBox)
- perdas de device sem causa em quatro configurações (victory-fork)

**Ninguém neste ecossistema resolveu manutenção.** É o problema real do conjunto, e é o que um
repositório unificado pode atacar de forma estrutural — nenhum desses projetos consegue, porque
cada um está sozinho correndo atrás de frames.

### 2.3 Verificabilidade varia de 2 a 5 — e é a única com dispersão real

| Nota | Repositórios |
|---|---|
| 5 | xbox-emulator, Xbox2Windows |
| 4 | TuxBox, SixNG, VIctory, XCL, DexNom, GTA6-Asset-Info |
| 3 | Belobug, VItoriaBr, AvatarManz, OpenScarlett |
| 2 | victory-fork |

Os dois com 5 chegaram lá pelo mesmo método, independente: **conformance contra a implementação de
referência**. O xbox-emulator roda os mesmos cenários contra o `xgameruntime.dll` real e contra o
seu próprio shim, com `const DIFFERS` explícito. O Xbox2Windows testa 510 formas em FXC **e** DXC
porque o defeito é invisível no compilador que ele usa.

O victory-fork com 2 é o caso mais instrutivo: tem CI desde `8e79c80` e faz push direto em `main`,
com "test suites were not run" registrado no próprio commit. **A infraestrutura existe e é
contornada** — o que é pior que não ter, porque cria a aparência de verificação.

## 3. Achados transversais

### 3.1 Um caminho de apagar o save do usuário continua aberto em três dos seis forks

Verificação por `git log --grep` em todos os clones:

| Repositório | `c88b7ac` (configuration id vazio) | `b8dbcba`/`d1da85a` (`v01` residual) |
|---|---|---|
| VIctory | ✅ `27f9b76`, `c88b7ac` | ❌ **ausente** |
| Belobug-AMD | ✅ `27f9b76`, `c88b7ac` | ❌ **ausente** |
| victory-fork | ✅ `27f9b76`, `c88b7ac` | ❌ **ausente** |
| DexNom | ✅ `27f9b76`, `c88b7ac` | ✅ `26e03c3`, `b8dbcba` |
| VItoriaBr | ✅ `27f9b76`, `c88b7ac` | ✅ `d1da85a` |
| AvatarManz | ✅ `27f9b76`, `c88b7ac` | ✅ `70868f8` |

Ambos os defeitos vivem em `crates/victory-hle/src/modules/xgamesave.rs`. A primeira correção
(configuration id vazio) propagou para todos os seis forks — a segunda não.

**O `#1` do ranking (Belobug), o fork mais popular (victory-fork, 3.037 ⭐) e o projeto-base
(VIctory) continuam podendo apagar um save já commitado** quando existe uma pasta `v01` residual ao
lado do `v1` válido. Três dos seis forks consertaram; três não.

Este é o achado mais acionável do estudo. Não é questão de escolher a base certa: é que metade dos
candidatos apaga save em alguma condição, e a correção pronta está no outro lado.

### 3.2 A correção que assume o ambiente é a fonte nº 1 de regressão

Cinco casos, cinco projetos, cinco ambientes assumidos errado:

| Commit | Assumiu | Quebrou em |
|---|---|---|
| VIctory `3c81420` | host aceita slots do título + 2^18 | heap limitado (sobraram 737.856 para 917.140) |
| VIctory `36eb0b4` | `select(OGT())` sobrevive ao DXIL | NVIDIA, que dobrou para `minNum` e engoliu NaN de `fract` |
| XCL PR #123 / `70f8dc4` | somar `uint64_t` por acesso é seguro | `amdxc64.dll`, `0xc0000005` determinístico |
| Belobug `7263339` | 20 s / 3 GiB é tempo demais | carregamento do mundo, que já usa gigabytes |
| AvatarManz `c651346` | dwords alteradas bastam | `tile1-0`, onde a sala perdeu luz e a geometria explodiu |

Padrão comum: **a lógica está certa, a premissa sobre a máquina está errada.** E em quatro dos
cinco, o autor não teria visto o bug testando no próprio hardware.

### 3.3 Três implementações independentes do mesmo invariante

O problema é idêntico nos três: *cache servindo tradução feita com configuração antiga*.

| Projeto | Mecanismo | Automação |
|---|---|---|
| victory-fork | `build.rs:51` varre literais `VICTORY_` → `translation_fingerprint()` → chave do cache (`shader_cache.rs:75`) | **automática** — impossível esquecer |
| Xbox2Windows | string `x2w-startup-shader-6` em `d3d12_draw.inl:201` **e** `gfx2hlsl.py:2176` | manual, mas nos dois lados |
| XCL | `XCL_PER_LANE_SOURCE_ID` por hash das fontes do lowering (`bfb8133`) | automática, escopo local |

Três equipes resolveram o mesmo problema de três formas. A do victory-fork é a única que
**estruturalmente** impede o erro: a lista de flags é gerada do código, então adicionar uma flag nova
já entra na chave sem ninguém lembrar. É a peça de infraestrutura mais valiosa do ecossistema.

### 3.4 Mensagem de commit não é evidência — três casos confirmados

| Repositório | Commit | Mensagem diz | Diff faz |
|---|---|---|---|
| xbox-emulator | `c95a909` | "Exec mask ... PSO use-after-free fix in register_pso" | 100% `xtfeventserver`, 1.294 linhas |
| TuxBox | `c217aa4` | "Now displays GTA VI properly on linux." | adiciona JSON com `logo_seen: false`, `gameplay_frame: 0`, `desktop_image: "black"` |
| AvatarManz | `README.md:108` | "no code of [XCL] is here" | `console_insn.rs`: "Ported from XCL ... unchanged" |

O segundo é o mais grave, porque a contradição está **dentro do próprio commit**: a mensagem afirma
uma coisa e o artefato versionado no mesmo ato registra o contrário.

### 3.5 Import monolítico destrói a auditabilidade

| Projeto | Commit | Escala | Consequência |
|---|---|---|---|
| xbox-emulator | `ff39277` | 1.131 arquivos / **266.033 linhas** | 19 de 22 defeitos só datáveis como "import" |
| VItoriaBr | `578d459` | 61 arquivos / **+32.334 linhas** | "commit introdutor" = "primeiro commit onde o código aparece" |
| DexNom | `570a941` | 170 arquivos | **0 de 98** arquivos byte-idênticos ao estado pós-merge; 3 correções de terceiros perdidas |

O caso do DexNom é qualitativamente diferente dos outros dois: não é só perda de granularidade, é
**perda de trabalho de terceiros**. A importação privada apagou o efeito de 287 commits e 137 PRs.

### 3.6 Capacidade pronta e não mergeada

**SixNG**: 1.294 linhas de leitura do TOC do RPF9 (`gamecipher.rs` + `rpf9toc.rs`) existem apenas em
`origin/claude/rpf9-toc`. O `README.md:42` de `master` ainda diz *"the TOC is encrypted and SixNG
does not decrypt it"* — porque o commit que implementa a funcionalidade **também apaga essa linha do
README**, e nunca foi mergeado. A documentação está tecnicamente correta sobre um repositório que
não tem a capacidade que o próprio autor já escreveu.

### 3.7 Teste maior que o produto, quando o produto é invisível

Xbox2Windows `583178d4`:

```
tests/startup_gfx2hlsl_f32_nan_check.py   +464
tools/startup/gfx2hlsl.py                    +7  -3
```

**464 linhas de teste para 3 linhas de produto**, porque o defeito não aparece no compilador que o
projeto usa. A regra que se extrai: *teste no compilador e no fornecedor que você não usa*, porque o
usuário pode não usar o seu.

É a mesma família do `quad_perm` do Belobug (bug só na AMD) e do `select(OGT())` do VIctory (bug só
na NVIDIA).

### 3.8 Default inseguro como escolha deliberada

**AvatarManz** `indirect.rs:294-304` — o próprio comentário descreve o risco:

> *"A list whose budget one garbage record empties loses all its later draws, and which record that
> is may differ from frame to frame."*
> ```rust
> fn rest() -> u32 {
>     u32::from(std::env::var("VICTORY_INDIRECT_REST").map_or(true, |v| v != "0"))
> }
> ```

Default `true` = "toma o resto". O modo seguro existe (`=0`) e vem desligado. A escolha é defensável
— recusar tudo é mais seguro que executar registro corrompido — mas o custo é geometria ausente sem
aviso, e há testes WARP extensos para o comportamento. O defeito é o default, não a falta de
pensamento.

### 3.9 Um bloqueio jurídico que contamina o conjunto

`avatarmanz/VIctory-7900XTX` porta `console_insn.rs` verbatim de um projeto GPL-2.0-**only** para um
repositório GPL-3.0, enquanto o README nega. Ver `NOTICE.md`. Consequência: **Reaproveitamento 0** —
nada do projeto pode ser copiado, apesar da Engenharia 4.

## 4. Ranking por propósito

A média geral ordena os projetos, mas não responde "qual usar para quê". Quatro leituras diferentes:

| Pergunta | Resposta | Por quê |
|---|---|---|
| Qual roda melhor hoje? | **Belobug-AMD** | único com mundo iluminado e missões; 77,4% no PortWatch |
| Qual serve de base? | **TuxBox** (Linux) ou **VIctory / Xbox2Windows** (Windows) | 4,0 de média; Reaproveitamento 5 |
| Qual tem a melhor verificação? | **xbox-emulator** e **Xbox2Windows** (5) | conformance contra implementação de referência |
| Qual tem mais código aproveitável? | **TuxBox** e **SixNG** (5) | plataforma única e especificação de formato |
| Qual **não** usar como base? | **AvatarManz** | GPL-2.0-only contaminando obra GPL-3.0 |

Duas surpresas frente ao que eu havia recomendado antes:

**TuxBox passou o Belobug e o DexNom.** Não por ser mais avançado — é o menos avançado dos três em
progresso de jogo. Passou porque documenta o que não sabe (`crash_cause_established: false`,
`memory_waits_released_after_5s_at_least: 16`), porque cada defeito de Wine/Proton está isolado e
corrigido, e porque nada nele precisa ser reescrito para ser reusado.

**DexNom caiu para 3,4.** Os ganhos são reais e grandes — 3.469 → 969 ms, 99,94% dos shaders — mas o
`570a941` apagou o trabalho de 137 PRs, e isso é uma dívida que custa mais que os ganhos para quem
vai construir em cima.

## 5. Recomendação

A conclusão que a matriz sustenta é que **nenhum desses projetos deve ser a base sozinho**. A razão
principal é técnica: cada um carrega, isoladamente, pelo menos um defeito que outro já resolveu — e
no caso dos três mais usados, o defeito em questão apaga dados do usuário.

Ordem de trabalho sugerida:

1. **Fundação:** VIctory (Windows) + TuxBox (Linux/Proton), pela média e pelo Reaproveitamento 5.
2. **Correção obrigatória antes de qualquer coisa:** o `b8dbcba`/`d1da85a` do save, que falta
   justamente em VIctory, Belobug e victory-fork. Sem isso, o repositório unificado nasce podendo
   apagar dados do usuário.
3. **Infraestrutura:** o `build.rs` + `translation_fingerprint()` do victory-fork. É o que impede a
   classe de erro mais comum do conjunto.
4. **Verificação:** adotar o padrão do xbox-emulator e do Xbox2Windows — conformance contra
   referência e controle negativo no fornecedor que você não usa.
5. **Não copiar:** nada do AvatarManz (licença); nenhum default do Belobug (`crashed.txt`, 33 GB,
   contagem de slots); nenhum import monolítico.
6. **Sustentabilidade:** é a única dimensão em que ninguém passou de 3. Se o repositório unificado
   não atacar isso, herdará a mesma falha de todos os treze.

## 6. Método

Cada documento individual foi produzido por um subagente isolado, que recebeu apenas o próprio
repositório, sua análise completa, os fatos gerados por `git` e a divergência entre forks. Nenhum viu
o trabalho dos outros. O placar consolidado foi extraído dos treze arquivos por script, não por
transcrição manual.

Onze conclusões foram reconferidas à mão no `git` — ver `VERIFICACOES.md`. Duas nesta rodada:
o comentário de `rest()` no AvatarManz e a mensagem do `c217aa4` no TuxBox.
