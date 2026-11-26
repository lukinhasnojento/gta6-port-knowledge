# brcly/TuxBox — atualização de 2026-11-25 23:00

## Resumo

Cinco commits novos, um deles enorme: `04a560c1` (+1987/-310) introduz o *host profile* — a camada
passa a escolher caminho de shader e de memória pelo que o driver declara poder fazer, não pelo
fabricante — junto com a tradução de programas wave64 para hosts de 32 lanes e a cópia de memória para
o RADV. Os outros quatro são dois merges (#200, #207), a regra de que código compartilhado precisa de
check em AMD **e** NVIDIA, e o pin do GE-Proton11-14 no shell de Nix. Um PR aberto (#208).

## Commits novos (5)

Ordem de entrada na `main` (cadeia de pais verificada): `4af37a4c` → `04a560c1` → `13033e67` →
`4f7fa84c`; `813740a1` é o commit da branch do #207, criado sobre `cb8c38bb` antes dos merges.

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `4af37a4c` | brcly | Merge pull request #200 from DrArroway/fix/radv-mapped-buffer-shadow | Merge de dois pais (`cb8c38bb`, `42af6743`), +748/-26 em 7 arquivos. Cria `runtime/d3d12/src/device/heap/shadow.rs` (+535): cópia anônima de páginas quando o host rejeita import de memória de seção, com `CreateFileMappingW`/`MapViewOfFile`, fila de cópia, fence e `VirtualQuery` para achar guardas. `ImportedHeap` ganha `_shadow: Option<Arc<ShadowPages>>` com **ordem de campo load-bearing** (o `native` precisa cair antes do `_shadow` desmapear) e construtores `direct`/`shadowed`; `pages.rs` ganha `supports_mapped_import()` com `OnceLock` e `probe_mapped_import`; `import.rs` +27; `page_mapping.rs` passa a mapear `HashMap<usize, (Range<u64>, Arc<AtomicBool>)>`; `submit.rs` chama `crate::device::sync_shadow_imports()` a cada batch. Teste `heap_shadow.rs` +124 |
| `04a560c1` | brcly | choose shader and memory paths by what the host driver can do | +1987/-310 em ~20 arquivos. **Host profile** (`docs/USAGE.md` +28): `wave` (faixa de lanes), `section-import` (`aliased`/`substituted`/`refused`/`unknown`), `descriptor-bytes`, com override `TUXBOX_HOST_PROFILE` (`wave=32`; uma configuração inválida rejeita todas). **Tradução para 32 lanes** (`tools/shader-compiler/src/hlsl/paired_wave64.rs`, +464): cada thread do host roda duas lanes do console (`n` e `n+32`), escalares e máscaras mantêm o valor de 64, vetoriais rodam uma vez por lane; `translate_for_wave32_host` só troca quando a tradução comum exige 64 lanes e o estágio é compute; recusa `v_movrel`; cache `<hash>_w32` ao lado do normal. Emissores tocados: `instruction.rs`, `output.rs` (prólogo `paired_wave64::prologue`), `scalar.rs`, `setup.rs`, `wave_cull.rs`. Mais `tools/check_paired_wave64.py` (+383, 18 programas sintéticos contra modelo do console, 256 waves cada), `tools/reference/independent-shaders/wave_pairs.cpp` (+103), `tests/hlsl/host_profiles.txt` (48 linhas de hashes FNV-1a por perfil), `examples/lanes/selftest.cpp` (+82, cenário `wave64-shader`). Documenta também a **cópia de memória** que o merge #200 implementa, com as limitações conhecidas: escritas da GPU ficam na cópia, e o custo é encontrar e copiar memória alterada antes de cada submissão |
| `13033e67` | brcly | require AMD and NVIDIA checks for code both vendors run | +24/-0. `CONTRIBUTING.md` ganha a seção *AMD and NVIDIA*: mudança em `runtime/` ou `tools/shader-compiler/` só entra depois de check nas duas marcas, as duas execuções podem vir de pessoas diferentes, e caminho que difere por host **deve** ler o host profile. `tests/test_repository_hygiene.py` +9 acrescenta `test_paths_are_chosen_by_host_capability_not_vendor`, que falha se aparecer `0x10de`/`0x1002`/`0x8086`, `.VendorId` ou `vendor_id` em `runtime/` ou `tools/shader-compiler/src/`, exceto em `device/dxgi.rs` e `device/host_profile`. `pull_request_template.md` +2 |
| `4f7fa84c` | brcly | Merge pull request #207 from gtb-sbe/nix-ge-proton | Merge de `13033e67` e `813740a1`, +33/-1 em 3 arquivos; o conteúdo é o do commit abaixo |
| `813740a1` | gtb-sbe | nix: pin GE-Proton11-14 in the development shell | `flake.nix` +13: `proton-ge = pkgs.fetchzip` de `GE-Proton11-14-x86_64.tar.gz` com hash `sha256-Bru3l0AqrvDl+XYC1dYhmy5XqKP49yFjWVJwZzel60Q=`, exposto como `packages.proton-ge`, e `PROTONPATH` no `devShells.default`. `tests/test_flake.py` novo (19 linhas) exige que o nome pinado no flake seja exatamente `PINNED_PROTON` de `project_tuxbox/host.py` e que `PROTONPATH` apareça; `examples/lanes/README.md` atualizado |

O diff de `4f7fa84c` contra o primeiro pai é idêntico ao de `813740a1` — é o comportamento normal de um
merge commit, não um patch duplicado.

## Pull requests abertos (1)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 208 | lanes: draw through the second of two raster root tables | Cenário `raster-tables` no Lanes: desenhar um triângulo de tela cheia pelo **segundo** de dois root tables de raster, com dois programas de console escritos à mão (`examples/lanes/raster_fixture.py`). Declara ser a cobertura de GPU que o #168 deixou para depois | **Área direta de D-22** — "`Failed to remap` (-3) em roots raster com várias tabelas de descritor", registrado como corrigido em `95b242b`. Não é o mesmo defeito: aqui é cobertura de teste do caminho que o #168 refletiu, e o catálogo não tem linha para ele |

## Pull requests mesclados (2)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| 200 | fix/radv-mapped-buffer-shadow | `4af37a4c` | Entrou exatamente o que o título diz: cópia-sombra das páginas importadas quando o host rejeita o import de seção, com sonda em `OnceLock`, sincronização por batch em `submit.rs` e ordem de destruição de campo documentada como crítica. +748/-26 |
| 207 | nix: pin GE-Proton11-14 in the development shell | `4f7fa84c` | Pin do GE-Proton11-14 no `flake.nix` com `PROTONPATH` no shell, teste que amarra o pin ao `PINNED_PROTON` do launcher, e README do Lanes atualizado. +33/-1 |

## Cruzamento com o catálogo

- **Corrige D-xx?** O catálogo `TuxBox` não tem linha para import de memória recusado pelo RADV — e é
  exatamente esse o defeito que `4af37a4c` e a seção *Memory copy* de `04a560c1` tratam (o
  vkd3d-proton relata sucesso e a GPU lê outra memória). **Candidato a D-30 de TuxBox**: defeito novo,
  descoberto e corrigido na mesma hora. Os D-11 (desligar `VK_EXT_descriptor_buffer` corrompe heap no
  RADV) e D-12 (stride inline de 32 bytes no RADV) são vizinhos de plataforma, não este defeito.
- **Abre defeito novo?** Dois:
  - `04a560c1` + `4af37a4c`: a cópia de memória é tomada quando o perfil diz `substituted` ou
    `refused`, e o próprio documento declara duas limitações estruturais — **escritas da GPU ficam na
    cópia** (a CPU e outros endereços GPU das mesmas páginas não as veem) e há custo por submissão.
    É um caminho que troca correção parcial por funcionamento: qualquer shader que escreva nessas
    páginas e a CPU do título leia depois recebe dado velho. Nada no catálogo cobre isso.
  - `04a560c1`: `TUXBOX_HOST_PROFILE` deixa forçar o perfil de outro driver (`wave=32`,
    `section-import=substituted`) numa máquina que não precisa. O documento avisa que o override não
    reproduz o comportamento do outro driver — mas o aviso está em prosa, não no código.
- **Dívida aberta que continua:** D-09 (toolchain Linux: `synchronization.lib` ausente e `clang 19` com
  `/W4 /WX`), D-14 (tabela de slots da lista Scarlett deslocada), D-19 (asserts de tile-heap remendados
  por RVA) e D-20 (instruções exclusivas de AMD em CPUs Intel) seguem sem correção registrada. Nada
  desta faixa os toca.
- **Observação de processo:** `13033e67` institucionaliza no CI uma regra que o catálogo cobra de outro
  repositório — o `AvatarManz` D-16 ("duas convenções opostas para o mesmo tipo de switch, em 294
  variáveis"). Aqui a escolha por capacidade do host passou a ser verificada por teste.

## Confiança

- **Verificado:** os 5 commits, a cadeia de pais de cada um (inclusive que `813740a1` nasceu em
  `cb8c38bb` e foi mesclado por `4f7fa84c`), e os diffs contra o primeiro pai.
- **Verificado:** os dois merges (#200, #207) por `parents` com dois SHAs e por `mergedAt`.
- **NAO VERIFICADO:** o PR #208 (diff não lido) e as afirmações de desempenho do `docs/SHADERS.md`
  ("133 compute programs differem; 111 compilam com DXC e DXV; 22 recusados") — são medições do autor.
- **NAO VERIFICADO:** se a cópia-sombra funciona em RADV real; nada foi compilado nem executado.
