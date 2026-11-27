# Execução de 2026-11-27 21:00

## Data e hora

Coleta de 2026-11-27T21:05–21:22 UTC real — ~18:05–18:22 no relógio do GitHub, que segue cerca de 3 h
atrás e é o relógio em que `last_run` é gravado. Diretório de documentos: `docs/updates/2026-11-27/21/`
(UTC). Estado lido: `baselined: true`, `runs: 18` (gravado como 19); 14 entradas em `repos` no início e
no fim, nenhuma removida, nenhuma acrescentada.

## Sanidade do estado (passo 1)

`gh api repos/<o>/<r> --jq .default_branch` foi pedido nos 14. **Dois não respondem**: `Belobug/VIctory-Belobug-AMD`
e `avatarmanz/VIctory-7900XTX` devolvem HTTP 404 em `repos/<o>/<r>` **e** em `users/<o>`; a página
`https://github.com/<owner>` também devolve 404. Não é repositório renomeado nem branch trocada: a conta
inteira saiu da API. Consequência: a `default_branch` desses dois **não pôde ser reconferida** e a
triagem deles é NAO VERIFICADO; os cursores ficaram parados; as entradas continuam em `repos`, com um
campo `unreachable` novo. O caso está em `docs/updates/2026-11-27/21/_falhas.md`.

Nos outros 12, `default_branch` idêntico ao gravado, inclusive `master` (`holdmysocks/SixNG`). Nenhum
`last_commit` nulo — não houve baseline nesta execução. Nenhum `compare` devolveu `diverged`. Contagem
de `repos` igual no início e no fim (14).

## Triagem (passo 2)

Os 14 foram triados (12 com medição completa); 4 entraram no deep dive. Arquivo:
`docs/updates/2026-11-27/21/_triagem.md`.

| Repositório | Commits atrás | Abertos acima do cursor | Mesclados na janela | Sinal | Deep dive |
|---|---|---|---|---|---|
| `R0Y00/Xbox2Windows` | 5 | 3 (113 no total) | 5 | SIM (critério 2; quatro lowerings novos) | sim |
| `brcly/TuxBox` | 4 | 8 (11 no total) | 1 | SIM (critério 2; root SRV de compute e helper DXC) | sim |
| `holdmysocks/SixNG` | 3 | 2 (2 no total) | 3 | SIM (critério 2; `fix(metal4)` + bake + mandíbula) | sim |
| `fxceless/victory-fork` | 1 | 3 (947 no total) | 0 | SIM (critério 2; commit direto de 19 arquivos) | sim |
| `Marvisak/VIctory` | 0 | 38 (≥, 1.675 no total) | 0 | SIM (lote de `Shaders:`/`GPU:`) | não |
| `ivall/xcl` | 0 | 9 | 0 | SIM (#3052, #3047, #3049) | não |
| `DexNom/VIctory` | 0 | 2 (122 no total) | 0 | SIM fraco | não |
| `uiliannn/VItoriaBr` | 0 | 1 (75 no total) | 0 | SIM fraco (#132, só teste) | não |
| `omonkeyz/xbox-emulator` | 0 | 1 (25 no total) | 0 | SIM (#395) | não |
| `californiaAlmonds/OpenScarlett` | 0 | 0 (1 no total) | 0 | NAO | não |
| `walkthestars/GTA6-Asset-Info` | 0 | 0 | 0 | NAO | não |
| `stellarr-studio/kelvaroq` | 0 | 0 | 0 | NAO | não |
| `Belobug/VIctory-Belobug-AMD` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (404) | bloqueado |
| `avatarmanz/VIctory-7900XTX` | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO | NAO VERIFICADO (404) | bloqueado |

Desempate: o critério 1 (defeito do catálogo) não selecionou nenhum dos 13 commits; o critério 2
selecionou os quatro repositórios com commit na janela, que ocuparam as quatro vagas. `xcl` não teve
commit nesta hora e perdeu a vaga que tivera às 20:00; `victory-fork` entrou no lugar dele.

## Documentos escritos (passo 3)

4 documentos em `docs/updates/2026-11-27/21/`: `Xbox2Windows.md`, `TuxBox.md`, `SixNG.md`,
`victory-fork.md`, mais `_triagem.md` e `_falhas.md`. **13 commits documentados de 13** atrás dos
cursores (5 + 4 + 3 + 1), **9 PRs mesclados** (5 + 1 + 3 + 0) e **16 PRs abertos** (3 + 8 + 2 + 3).
Nenhum repositório chegou ao teto de 40 commits.

Achados principais:

- `R0Y00/Xbox2Windows`: quatro lowerings novos no caminho direto de compute — `v_dot8_i32_i4`/`u32_u4`
  (`8cf9be9b`), `v_sat_pk_u8_i16` (`5ebdc581`), `v_frexp_exp_i32_f32`/`mant_f32` (`80668b97`) e
  `mant_f16`/`exp_i16_f16` (`429f11d5`) —, todos com recusa explícita de modificadores não canônicos e
  teste registrado com `SKIP_RETURN_CODE 77` para a parte de hardware. O `v_dot8_i32_i4` já estava
  catalogado como defeito no `VItoriaBr` (D-15, corrigido em `75aa531`): a mesma lacuna existia em dois
  forks. Nenhum `D-xx` do próprio repositório é corrigido.
- `brcly/TuxBox`: o #312 (`8c082e65`) generaliza a regra de root SRV em `compute_table.rs` —
  `(multi_table.is_some() || stage == 5 && root.is_some()) && class == 0x100` —, com teste que fixa o
  fail-closed em três casos; é área de D-22, não D-22. `89fb2517` troca o caminho de compilação por um
  helper DXC que remove `correlated-propagation` do *pass list* (51 s contra 60 s em 1.500 requisições)
  e **reescreve à mão** a propriedade 11 do entry point para repor `[WaveSize(N)]`; `-O0` continua na
  CLI, então D-02/D-03 seguem abertos. A chave do cache inclui o binário do helper, e a saída mantém
  nomes (~38% maior) sem digest de container.
- `holdmysocks/SixNG`: o `fix(metal4)` do #476 é uma troca de posse (`&vertex_bytes`/`&index_bytes`),
  não de comportamento; o #472 acrescenta bake offline de cache de modelo (processo filho, CRC em
  streaming, macOS + `metal4-poc`); o #481 extrai `fit_keys`/`add_keys` de `follow_jaw` e leva as chaves
  de mandíbula ao viewer, para dentes e língua abrirem com a boca no painel Expression.
- `fxceless/victory-fork`: um commit direto na `main`, 19 arquivos, +1.914/-108, com cinco frentes —
  contador de threads por grupo (ported de `VIctory-NVIDIA 854f57f` / `Vice-Tay c125c0c`), skips em
  volta de loops com teste escalar, LDS inicial do grupo, reúso de consultas ao diretório pelos ray
  walkers e o portão `VICTORY_THIN_GATE` para slots dinâmicos —, mais `VICTORY_WATER_LIGHT=1` como
  default do launcher. **Achado verificável:** a chave do cache (`shader_cache.rs`) ganhou
  `VICTORY_THREAD_TICKET`, mas não `VICTORY_MEMORY_NEAR`, `VICTORY_LDS_START` nem
  `VICTORY_WAVE_LOOP_SKIPS`, que mudam o código gerado, e `host_settings()` devolve só quatro números
  de host. É a classe de D-02 (traduções envenenadas por switch), catalogada como corrigida em
  `e9765a7`, de volta por três switches novos.

## Falhas e desvios

- **Dois repositórios inacessíveis (404 de conta).** Registrado em `_falhas.md`, com as tentativas
  feitas. Não avançaram cursor, não foram documentados, não saíram de `repos`.
- **A "tabela de pistas" citada no passo 2 continua inexistente** em
  `docs/automacao/ANALISE_HORARIA.md`. As áreas do sinal vieram das seções D de `catalogo/defeitos.md`,
  e o sinal das linhas sem commit é por título/corpo de PR, não por patch. O deep dive foi decidido por
  diff.
- **Contagem de merges x commits:** nesta hora batem nos quatro documentados — os 5 do `Xbox2Windows` e
  os 3 do `SixNG` são squash de um pai (`committer` = GitHub) e o `merge_commit_sha` é o próprio commit;
  o `TuxBox` tem 1 merge de dois pais; `victory-fork` tem commit direto e zero merges.
- **PRs abertos documentados pelo título, não pelo patch.** 16 PRs lidos por título, base e head;
  nenhum patch de PR aberto foi lido linha a linha.
- **Sem subagentes.** A ferramenta não existe neste ambiente; os quatro repositórios foram processados
  em sequência e o limite de 4 deep dives foi respeitado.
- **Rate limit:** 4.783 restantes no início; ao fim a janela já tinha reiniciado (4.890 restantes, 110
  usados), então o gasto total não é legível pelo contador. Nenhum 403. `search/issues` continua
  devolvendo HTTP 422 para `ivall/xcl` (repositório transferido) — o total de abertos desse repositório
  fica sem número pela terceira hora.
- **Escrita:** somente em `G:\gta6-port-knowledge`. Os 14 monitorados foram só leitura de API.

## Estado gravado

`runs: 19`, `updated: 2026-11-27T21:22:00Z`, `last_error` com os dois 404. Cursores que avançaram:

| Repositório | `last_commit` | `last_pr_number` | `last_run` |
|---|---|---|---|
| `R0Y00/Xbox2Windows` | `429f11d5` (head) | 3142 | 2026-11-27T18:10:00Z |
| `brcly/TuxBox` | `89fb2517` (head) | 321 | 2026-11-27T18:10:00Z |
| `holdmysocks/SixNG` | `1c8672d8` (head) | 483 | 2026-11-27T18:10:00Z |
| `fxceless/victory-fork` | `f299fba1` (head) | 1493 | 2026-11-27T18:10:00Z |

Os outros 10 mantêm cursor de commit e de merge; `Belobug` e `avatarmanz` ganharam o campo
`unreachable` e seguem no `backlog` como **bloqueados**. `backlog` com 6: `Belobug/VIctory-Belobug-AMD`,
`avatarmanz/VIctory-7900XTX`, `Marvisak/VIctory`, `omonkeyz/xbox-emulator`, `DexNom/VIctory`,
`uiliannn/VItoriaBr` — `victory-fork` foi consumido. A ordem e o achado da chave de cache estão no
`backlog_note`.

## Push

(Preenchido após a publicação.)
