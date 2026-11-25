# Relatório Final — Análise Histórica dos Repositórios

**Data:** 2026-11-25
**Modelo único:** DeepSeek V4.1 Flash (`opencode-go/deepseek-v4.1-flash`), 5 subagentes em paralelo
**Método:** arqueologia de git (log, show, diff, blame, `log -S`) + pacotes de evidência pré-extraídos + verificação independente pelo agente orquestrador

---

## 1. Escopo

Cinco repositórios clonados e sincronizados com o GitHub (`git fetch --all`, HEAD comparado com a API):

| Repositório | Commits | Branch | Raiz (commit 0) | Fora do main |
|---|---|---|---|---|
| `Marvisak/VIctory` | 241 | main | `f4504f6` | 0 |
| `ivall/xcl` | 853 | main | `8fa7fa0` | 0 |
| `walkthestars/GTA6-Asset-Info` | 5 | main | `dd592ba` | 0 |
| `holdmysocks/SixNG` | 303 | master | `6b68f75` | **332** |
| `R0Y00/Xbox2Windows` | 4.076 | main | `7e28f98` | **423** |
| **Total** | **5.478** | | | 755 |

Todos os clones são completos (`shallow=false`), com raiz única preservada e HEAD idêntico ao do GitHub.

---

## 2. Entregas

| Arquivo | Tamanho | Seções | Defeitos | Hashes citados |
|---|---|---|---|---|
| `FINAL/Xbox2Windows_deepseek.md` | 408 KB | 12/12 | 29 | 4.459 |
| `FINAL/XCL_deepseek.md` | 147 KB | 12/12 | 24 | 1.194 |
| `FINAL/SixNG_deepseek.md` | 95 KB | 12/12 | 24 | 675 |
| `FINAL/VIctory_deepseek.md` | 93 KB | 12/12 | 27 | 592 |
| `FINAL/GTA6-Asset-Info_deepseek.md` | 79 KB | 12/12 | 28 | 151 |
| **Total** | **822 KB** | **60/60** | **132** | **7.071** |

Todos com a estrutura A–L exigida, índice de commits, catálogo com IDs `D-xx` e classificação de confiança.

---

## 3. Verificação independente

Oito conclusões foram reconferidas por mim, diretamente no git, depois de escritas:

| # | Alegação | Verificação | Resultado |
|---|---|---|---|
| 1 | `22989d61` restaurou `d3d12_draw.inl` byte a byte | `git rev-parse` dos blobs | **CONFIRMADO** — `98c69432` antes e depois; mutado `9a649fe4` |
| 2 | Só existe 1 revert real no Xbox2Windows | `git log --grep="^Revert"` | **CONFIRMADO** — os outros 4 apenas citam "revert" no corpo |
| 3 | `format/15-lights.md` tem off-by-one | soma 447.756+326.183+907 | **CONFIRMADO** — 774.846 vs 774.847 declarado |
| 4 | 81 ocorrências de "SixNG" em 24 arquivos | `Select-String -AllMatches` | **CONFIRMADO** — 24 arquivos, 81 ocorrências |
| 5 | `structure/04:59` declara 289, parcelas somam 333 | 277+41+10+5 | **CONFIRMADO** — inconsistência real |
| 6 | `7102a49` conserta `mod tests` duplicado | `git show --stat` | **CONFIRMADO** — "rename the second test module", 2 linhas |
| 7 | `STRUCTURED_BYTES_PER_CODE_BYTE = 128` | `git grep` em HEAD | **CONFIRMADO** — `mod.rs:641` |
| 8 | `gamecipher.rs`/`rpf9toc.rs` só existem na branch | `git ls-tree` em HEAD e em `origin/claude/rpf9-toc` | **CONFIRMADO** — ausentes em master, presentes na branch |

Uma imprecisão encontrada: o relatório do XCL descreve o revert `70f8dc4` como "51 linhas de emit.rs, 55 de tests.rs". O total de 93 deleções está certo; 51 é o total de linhas tocadas em `emit.rs` (13 adicionadas + 38 removidas).

---

## 4. Achados transversais

### 4.1 A fonte nº 1 de regressão é "correção que assume algo do ambiente"

Aparece em 3 dos 5 repositórios, sempre com a mesma forma: o patch está correto na lógica e errado na premissa sobre a máquina.

- **VIctory `3c81420`** assumiu que todo host aceita "slots do título + 2^18". Num heap limitado sobraram 737.856 slots — menos que os 917.140 que o título indexa. Reparado por `e8d3b64` no mesmo PR.
- **VIctory `36eb0b4`** assumiu que `select(OGT(...))` sobreviveria ao compilador DXIL. A NVIDIA dobrou para `minNum` e engoliu o NaN de `fract`. Reparado por `2e5001c`.
- **XCL PR #123 (`a787c52`)** estava correto — somar o endereço em `uint64_t` a cada acesso resolve o carry de 32 bits. Mas derrubava `amdxc64.dll` de forma determinística. O revert `70f8dc4` devolveu o bug original por ~12h até `f9e5ee9` refazer a mesma ideia com range-check e descriptor heap.
- **Videogame/GPU**: a lição é que "compila e passa no teste" não cobre o backend real do driver.

### 4.2 Instrumentação temporária não é defeito — e o defeito real fica um passo antes

O caso `Xbox2Windows`: `ffb70f6a` (17:31) marcou `viewport_set = true` ao encaminhar o viewport **default**, confundindo estado sintetizado com estado do título. `1f24626b` (17:38) desligou o default **de propósito**, como prova fail-first de um teste escrito 7 minutos antes. `22989d61` (17:40) reverteu em 88 s. A correção de verdade veio em `07f2464f` (17:53), separando `default_viewport_w/h`.

Quem lê o histórico procurando "commit com MUTATION" acha um falso positivo. O defeito estava no commit anterior.

### 4.3 Capacidade escrita e não mergeada é a maior perda documentada

**SixNG**: a leitura do TOC do RPF9 inteira (`gamecipher.rs` com mapeamento do executável e chamadas às rotinas de cifra do próprio jogo, `rpf9toc.rs`, CLI `toc`/`toc-extract`, 1.294 linhas) existe **apenas** em `origin/claude/rpf9-toc`. O `README.md:42` de master afirma: *"not read: the TOC is encrypted and SixNG does not decrypt it"*. Todo o carving de zlib, o `INDEX_VERSION` 4→7 e o conserto de split de recurso existem para compensar uma funcionalidade pronta que ninguém mergeou.

Dos 332 commits fora de master, apenas duas branches trazem arquivos inexistentes (`rpf9-toc` e `people-hair-lighting`); o resto é squash já aplicado.

### 4.4 Em engenharia reversa de formato, o defeito dominante é leitura de dado

Quatro dos maiores defeitos do SixNG são de offset/índice, não de lógica:

- `2cf005b` (PR #163) — offset de constante de material lido como índice de float: *"nearly every value came from the wrong place"*, invalidando todo `materials.json` anterior.
- `38de2e5` — `Meta::ptr` tentava ponteiro real antes da referência de pool: 130.066 instâncias com posição 1e10 e escala `inf`.
- `e7aae6b` — ordem de eixos escolhida pelos normais em vez do header: 114 malhas erradas.
- `3bfa416` — frame tangente vindo de UV1 enquanto o normal map é amostrado em UV0.

Padrão reutilizável: cada conserto substitui uma suposição pela fonte de verdade **e invalida o cache por versão**. Quem replica o formato precisa versionar cache junto.

### 4.5 Correção medida melhora a métrica e piora o invariante

- **SixNG `18bc31c`**: down-mip de 25,1 → 32,7 dB, mas violações no-wrap de 221 → 288 (recuperado para 85 em `783eaa4`).
- **SixNG `9ce31d3`**: substituiu 9 slots BC1, 6 ficaram piores que o default (um −13 dB), revertidos em `4406256`.

Lição: otimização de qualidade precisa de invariante medido junto, senão troca um erro visível por um invisível.

### 4.6 Defesa em profundidade tem custo explícito

**VIctory** fechou três famílias de device removal (slots dinâmicos esgotados `2849477`/`3c81420`; teto de 1.000.000 de views da NVIDIA `3827767`; descritor nulo `8c26061`+`071487f`+`6af0fae`+`4051149`+`d71c388`) com clamp, tabela de strides, nulos tipados e `bind() -> bool`. O custo está no código: draws descartados, geometria ausente. Não há crash — há silêncio.

### 4.7 O pior defeito do conjunto não é de renderização

**VIctory `c88b7ac`**: `XGameSave` com configuration id vazio tomava a pasta do usuário por pasta de configuração; a conversão de containers antigos gravava um container vazio no lugar e **apagava a pasta com todos os saves**. É o único defeito do corpus com potencial de perda de dados do usuário.

---

## 5. Estado por repositório

| Repositório | Estado | Risco aberto |
|---|---|---|
| **VIctory** | 27 defeitos, 3 famílias de device removal fechadas com custo de silêncio | salvamento (`XGameSave`), política de `prepare.rs` trocada 4× |
| **XCL** | 24 defeitos; per-lane atrás de flag opt-in | 2 defeitos f16 sem correção (mitigados por recusa); caso XDefiant nunca re-testado |
| **Xbox2Windows** | 29 defeitos; 1 revert real (instrumentação) | 4 famílias de estado atravessando fronteira (command list, rollback, float FXC/DXC, snapshot por braço) |
| **SixNG** | 24 defeitos; TOC pronto e não mergeado | 6 defeitos sem correção; 2 regressões de qualidade |
| **GTA6-Asset-Info** | 28 defeitos documentais | rebrand incompleto, `structure/` órfã e contraditória |

---

## 6. Base para o repositório unificado

O que vale levar, por ordem de valor:

1. **Contratos de device (VIctory)** — clamp de slots, tabela de strides, nulos tipados, `bind() -> bool`. Reutilizável em qualquer camada de compatibilidade D3D12.
2. **Aritmética de janela de memória (XCL)** — carry em `uint64_t` **com range-check contra a base** e acesso por descriptor heap, não soma por acesso. Esta é a forma que sobreviveu ao driver AMD.
3. **Limite heurístico de geração de código (XCL)** — `STRUCTURED_BYTES_PER_CODE_BYTE = 128` com fallback para `pc` blocks: 148 MB → 13,8 MB sem afetar nenhum outro programa.
4. **Política de fonte de verdade (SixNG)** — header antes de heurística; nunca inferir ordem de eixos pelos normais.
5. **Regra de versionamento de cache (SixNG)** — toda mudança de leitura de offset invalida cache; carimbar versão no artefato.
6. **Separação estado sintetizado vs. estado do título (Xbox2Windows)** — viewport default com `w/h` próprios, nunca compartilhando flag com o viewport do título.
7. **Prova fail-first isolada** — instrumentação temporária deve viver em teste, não em produção, e nunca ser reportada como defeito histórico.

---

## 7. Limitações

- **Issues e pull requests não foram consultados** em massa. Apenas números de PR citados em mensagens de merge. Xbox2Windows tem 1.830 PRs e VIctory 1.395; o conteúdo de discussão continua inexplorado.
- **Commits fora da branch principal** foram contados e amostrados (755), não analisados um a um.
- **Profundidade variável**: o Xbox2Windows tem 4.618 linhas porque inclui o índice completo dos 4.076 commits; a análise interpretativa propriamente dita é comparável aos demais.
- Cinco modelos foram executados antes e descartados por não seguirem a estrutura nem citarem hashes verificáveis. Os 28 documentos deles permanecem em `analise\` e o backup em `analise\_v1\`.

---

## 8. Arquivos de referência

| Caminho | Conteúdo |
|---|---|
| `analise\FINAL\*.md` | 5 documentos finais |
| `analise\_fatos\*.md` | fatos por repositório, gerados por git puro (sem LLM) |
| `analise\indices\*_log_full.txt` | histórico completo com corpos de mensagem |
| `analise\indices\*_fixcandidates.txt` | candidatos a defeito filtrados |
| `analise\_evidencias\*_evid.md` | 45 commits por repositório com diff real |
| `analise\SPEC.md` | especificação de formato e regras anti-invenção |
| `analise\_logs\*.ndjson` | execução bruta de cada modelo |
