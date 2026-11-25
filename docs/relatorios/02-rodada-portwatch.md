# Relatório — Lote PortWatch VI (7 repositórios)

**Data:** 2026-11-26
**Fonte do ranking:** `portwatch-vi.xyz/data/tracker.json` (atualizado 2026-11-25T22:09Z, 43 projetos rastreados)
**Modelo:** DeepSeek V4.1 Flash, 7 subagentes em paralelo
**Método:** índice de commits + dossiê factual gerado por git puro + pacote de 45 diffs reais por repositório + verificação independente pelo orquestrador

---

## 1. Seleção

Top 7 repositórios **públicos e acessíveis** do ranking, excluindo os quatro já analisados na rodada anterior (VIctory, XCL, Xbox2Windows, OpenScarlett):

| # no site | Repositório | Score/stars | Analisado |
|---|---|---|---|
| 1 | `Belobug/VIctory-Belobug-AMD` | 77,4% — líder | ✅ |
| 2 | `DexNom/VIctory` | 940 ⭐ | ✅ |
| 3 | `Zettrox/ztrx` | 1.499 ⭐ | ❌ **404 — repositório inacessível** |
| 4 | `uiliannn/VItoriaBr` | 192 ⭐ | ✅ |
| 5 | `avatarmanz/VIctory-7900XTX` | 362 ⭐ | ✅ |
| 6 | `fxceless/victory-fork` | 3.037 ⭐ | ✅ |
| 7 | `omonkeyz/xbox-emulator` | 1.849 ⭐ | ✅ |
| 8 | `brcly/TuxBox` | 371 ⭐ | ✅ |

Fora do escopo por falta de código público: `NotNozy`, `Doomsday`, `darkworldhm`, `suri`, `omonkey` (build privado) e `CGBJ`.
Fora do ar: `realdody/americas-recomp` (404) e `ItsNotRexed/Leonida` (DMCA + remoção pelo autor).
**Total novo analisado: 3.731 commits.**

## 2. Entregas

| Arquivo | Tamanho | Linhas | Seções | Defeitos | Hashes |
|---|---|---|---|---|---|
| `FINAL/xbox-emulator_deepseek.md` | 150 KB | 1.053 | 12/12 | 22 | 851 |
| `FINAL/Belobug-AMD_deepseek.md` | 140 KB | 1.994 | 12/12 | 17 | 1.145 |
| `FINAL/AvatarManz_deepseek.md` | 134 KB | 1.232 | 12/12 | 19 | 850 |
| `FINAL/DexNom_deepseek.md` | 132 KB | 1.636 | 12/12 | 16 | 851 |
| `FINAL/TuxBox_deepseek.md` | 128 KB | 1.227 | 12/12 | 29 | 941 |
| `FINAL/VItoriaBr_deepseek.md` | 122 KB | 1.004 | 12/12 | 21 | 765 |
| `FINAL/victory-fork_deepseek.md` | 111 KB | 857 | 12/12 | 17 | 511 |
| **Total** | **917 KB** | **9.003** | **84/84** | **141** | **5.914** |

Somado ao lote anterior: **12 repositórios, 9.209 commits, 273 defeitos catalogados, 1,7 MB de documentação.**

## 3. Achados estruturais

### 3.1 A genealogia real do ecossistema

Cinco dos sete compartilham a raiz `f4504f6` (commit inicial do VIctory). O teste de ancestralidade, porém, separa dois mundos:

| Fork | HEAD do Marvisak é ancestral? | Commits à frente |
|---|---|---|
| DexNom | **SIM** | 306 |
| Belobug-AMD | NÃO | — (sem merge-base) |
| VItoriaBr | NÃO | — |
| AvatarManz | NÃO | — |
| victory-fork | NÃO | — |

Para quatro deles o único ancestral comum é a raiz: houve reescrita de histórico. **Nesses, comparação por hash de commit é inútil** — só comparação por blob funciona. Para o DexNom, ao contrário, cherry-pick conceitual é viável.

`xbox-emulator` e `TuxBox` **não** são o mesmo projeto apesar dos 1.652 arquivos de cada: só **7 caminhos coincidem** (`.gitignore`, `Cargo.toml`, `Cargo.lock`, `LICENSE`, `README.md`, `CONTRIBUTING.md`, template de PR). Têm o mesmo upstream (`drycheetah84/TomBox`) e linhagens paralelas — o xbox-emulator importou do TuxBox apenas 3 commits de patch.

### 3.2 Perda de trabalho por sobrescrita (DexNom)

Os 287 commits / 137 PRs entre `70564ae` e `4118f33` tocaram 98 arquivos. A importação privada posterior (`570a941`) tocou os mesmos 98 e **nenhum ficou byte-idêntico ao estado pós-merge** (0/98). Três consertos de terceiros se perderam e estão **abertos em HEAD**:

- `4fc9d70` — out-of-bounds / UB em `Staging::detile`
- `be824d9` — corrida no arquivo `.tmp` de nome fixo da biblioteca de pipelines
- `274f698` — overflow de `u64` no `clwb_loop`

**É a justificativa mais concreta para o repositório unificado existir:** um fork com 940 estrelas está rodando com três bugs já corrigidos por terceiros.

### 3.3 O mecanismo que permite o ecossistema divergir (victory-fork)

`crates/victory-shader/build.rs:51` varre o próprio código-fonte por literais `"VICTORY_` e publica a lista como `VICTORY_SHADER_SWITCHES`; `translation_fingerprint()` (`emit/mod.rs:199`) entra na chave do cache de shader (`shader_cache.rs:75`). **Mudar qualquer flag invalida o cache automaticamente.** Existe porque `6a2d9f9` ligou wave64 wide sem entrar na chave e serviu tradução errada do cache.

Contagem independente: **248** literais distintos `"VICTORY_*"` (o relatório aponta 219 switches / 246 pontos de leitura).

Disciplina documentada: default off até rodar em jogo, `=alt` para A/B no mesmo run, log ao ligar.

### 3.4 A correção mais barata é a mais definitiva (Belobug)

`03629f3`, **17 linhas**: derivadas calculadas por `quad_perm` liam lanes comuns e recebiam zero de helper lanes na AMD; na borda do triângulo os três vizinhos do quad são helpers, a derivada dava valor inteiro errado e a textura caía no menor LOD. Fix: `quad_pick` sempre no pixel shader. Nenhum commit posterior desfez.

Contraste: o substituto da luz indireta precisou de **6 tentativas** e continua stand-in.

Correção que assume ambiente errado — cometida em casa: `pipeline/library.rs:244-252` documenta que o watchdog de compilação com 20 s / 3 GiB marcou **quatro pipelines legítimos como travados permanentemente** (*"run amd1, 2026-11-25"*), porque durante o carregamento o processo já ocupa gigabytes. Subiu para 60 s / 8 GiB em `7263339`.

### 3.5 Bloqueios jurídicos (AvatarManz)

`README.md:108` afirma: *"XCL (GPL-2.0-only) was read for how the same title behaves under another layer; **no code of it is here**"*.

`console_insn.rs` declara: *"**Ported from XCL** (GPL-2.0-only): `insn.rs` (decoder, as `console_insn_decode.rs`, **unchanged**)"*.

`LICENSE` do projeto: **GPL v3**.

GPL-2.0-**only** não é compatível com GPL-3.0 — só a variante "or later" permite atualização. Código copiado verbatim não pode ser relicenciado sem autorização do titular. **Este arquivo não pode entrar num repositório unificado GPL-3.0 sem resolver a licença antes.**

### 3.6 Qualidade de prova: a melhor e a pior do lote

**Melhor (xbox-emulator):** conformance contra o `xgameruntime.dll` real (`InitializeApiImpl(0x4a61,0x2712)`), sondas em silício (gfx1036 / RX 9060 XT), GBV como oráculo de descritor, testes bit-exatos com mutação que derruba cada fix. Guardas IEEE contra dobramento do compilador AMD (`tb_pz`/`tb_hz`/`tb_trunc64`): 23 casos falham sem elas.

**Pior (xbox-emulator, ironicamente):** commits cuja mensagem aponta para outro subsistema. `c95a909` diz *"Exec mask: keep full exec... PSO use-after-free fix in register_pso"* e o diff é **100% `xtfeventserver`** (ETW, 1.294 linhas). Nenhum arquivo de `register_pso`.

### 3.7 "Primeiro frame 4K" é apresentação, não imagem (TuxBox)

`evidence/projectscarlett-startup.json`:
```json
"present_dimensions": [3840, 2160],
"startup_presents_recorded_by_mangohud": 4618,
"desktop_image": "black with MangoHud after presentation began",
"logo_seen": false,
"game_report": { "state": "Game Init", "gameplay_frame": 0 }
```
Houve present em 4K — e a captura de tela é **preta**, o logo não apareceu e o frame de gameplay é 0. A primazia mundial não é verificável por este registro.

### 3.8 Contradições internas documentadas

| Repo | Fato |
|---|---|
| VItoriaBr | `README:26` promete "zero TDR"; `README:75` avisa que "ainda pode disparar reset do driver" |
| VItoriaBr | BVH: `README:164` diz 50/53; `docs/fork-ray-tracing.md:38` diz 49 de 52 |
| VItoriaBr | `parse_group_sync` retorna 3 por padrão (`emit/mod.rs:181`); docs dizem 2 |
| AvatarManz | README nega código do XCL; o arquivo declara porte verbatim |
| Belobug | `view_slots()` pede além do teto de 1.000.000 (`host.rs:514` `PORTABLE_VIEWS`) |

### 3.9 Defeitos abertos mais caros

| Repo | Defeito | Consequência |
|---|---|---|
| AvatarManz | registro indireto lixo esgota orçamento de 2²⁷ da lista de draws | derruba todos os draws seguintes; autor: *"whether ours are refused is not known"* |
| VItoriaBr | `VICTORY_FIRSTLANE_OWN=1` | 53/53 shaders, mas corrompe offsets de AABB; crash após ~18 s |
| TuxBox | stalls de fence no intro (D-29) | sem correção |
| TuxBox | esgotamento de descritores | encerrou o run mais longo |
| Belobug | sweep de tiles desligado | cópia cresce até 33 GB commitados |
| victory-fork | perdas de device sem causa em 4 configurações; *"this merged tree was built; its test suites were not run"* | dívida de verificação |

## 4. Conhecimento reutilizável para o repositório unificado

Por ordem de valor:

1. **Cache auto-invalidante por varredura de literais** (victory-fork) — `build.rs` + `translation_fingerprint()`. Resolve a classe inteira de "cache servindo tradução errada".
2. **Comparar forks por blob, não por commit** (DexNom, Belobug, VItoriaBr, AvatarManz, victory-fork) — quatro dos cinco não têm merge-base com a base.
3. **`quad_pick` no pixel shader** (Belobug, 17 linhas) — helper lanes na AMD, LOD mínimo em arestas de triângulo.
4. **Tradutor RDNA com análise de magnitude** (AvatarManz `20b9834`) — `bounded(v, 12)` prova quando o produto simples é seguro.
5. **Stride do metadado FSSM, não da ISA** (xbox-emulator) + `ByteAddressBuffer` para views raw.
6. **Guardas IEEE contra dobramento do compilador AMD** — `tb_pz`/`tb_hz`/`tb_trunc64`.
7. **Proteção de todas as views de uma página física** (victory-fork `8dd30ab`) — não só o endereço da cópia.
8. **Hash por grupo de 64 threads com dobra em ordem fixa** (victory-fork `5557224`) — 35 → 10 ms, opt-in por ser outro hash.
9. **Replay como ferramenta** (AvatarManz, 15 arquivos / 13.838 linhas) — mais durável que qualquer patch.
10. **Lição de auditoria:** mensagem de commit não é evidência (xbox-emulator `c95a909`), nem README de licença (AvatarManz).

## 5. Limitações

- `Zettrox/ztrx` (#3 do ranking) inacessível — a análise do 3º colocado ficou sem cobertura.
- Issues e PRs consultadas apenas quando citadas em mensagens; nenhuma varredura em massa.
- Números de estrelas variam entre execuções (API viva).
- Contagem de switches do victory-fork: 248 literais medidos vs 219 declarados no relatório (inclui fixtures).
- `TuxBox` e `xbox-emulator` foram analisados por subagentes distintos que convergiram na mesma conclusão de linhagem — o que aumenta a confiança.
