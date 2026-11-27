# DexNom/VIctory — atualização de 2026-11-27 23:00

## Resumo

1 commit acima do cursor `8114ff7` (só o README) e **14 PRs abertos**, nenhum mesclado. A fila é o
efeito de `570a941` ("Publish DexNom Runtime Radeon sources with upstream credits and build instructions",
2026-11-25, 170 arquivos): 9 PRs restauram commits que aquele publish derrubou, com `cherry-pick -x` e
autoria preservada, e 5 são correções novas com teste de hardware. É o único repositório da janela com
sinal de critério 1 sem commit de código — os defeitos D-10, D-11 e D-12 do catálogo continuam abertos
por reversão e nenhum dos 14 PRs os reaplica.

## Commits novos (1)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `4ad86927` | DexNom | Revise README header and project description | Uma linha: o título `# DexNom RayTime (AMD) Join the efforts: fluxer.gg/p9Y0JR0q` perde o convite e fica `# DexNom RayTime (AMD)`. Nada mais muda (1 arquivo, +1/−1) |

## Pull requests abertos (14)

Todos com `created_at` entre 2026-11-27T15:43Z e 20:02Z no relógio do GitHub; `baseRefName` `main` em
todos; corpo lido nos 14, patch não lido.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| `#148` | XAsync: canceling a call that already completed leaves it alone, as documented (restore 44ce067) | Checa `call.done` sob trava em `XAsyncCancel` e devolve antes de invocar o provedor com `OP_CANCEL` | Não está no catálogo; é restauração de `44ce067` derrubado por `570a941` |
| `#147` | GPU: every object answers for IUnknown (restore 92f2c60) | Lista de interfaces de todo objeto gráfico ganha `IUnknown` no `QueryInterface` | Idem, restaura `92f2c60` |
| `#146` | Shaders: a signed 64-bit operand sign-extends its 32-bit literal (restore 7d04943) | Literal de 32 bits em operando de 64 bits passa a ser estendido com sinal | Idem, restaura `7d04943` |
| `#145` | Shaders: lower v_cvt_off_f32_i4 (restore 78c1267) | O opcode estava registrado mas caía no caminho inteiro | Idem, restaura `78c1267` |
| `#144` | GPU: interpolate normalized rectangle vertex attributes (restore 6f36cdf) | Atributos UNORM escalares e de dois canais, e normalizados com sinal, na interpolação | Idem, restaura `6f36cdf` |
| `#143` | GPU: fix CreateSamplerX mip-none base level, LOD range, bias clamp, and invalid enum values (restore e6fe8ec) | `MipLODBias`, faixa de LOD invertida, `MipLODBias` e enums inválidos no `CreateSamplerX` | Idem, restaura `e6fe8ec`. Toca faixa/clamp de LOD, mesma área de **D-04** (LOD e clamp descartado), mas é outro código e outro defeito |
| `#142` | Shaders: preserve mip point filtering in anisotropic static samplers (restore e45bacd) | Sampler estático anisotrópico mantém o filtro point de mip | Idem, restaura `e45bacd` |
| `#141` | GPU: fences answer for ID3D12Fence1 as documented (restore bbdc3ca) | `ID3D12Fence1::GetCreationFlags` no slot 11 | Idem, restaura `bbdc3ca` |
| `#140` | Shaders: v_writelane_b32 takes its lane from S1[4:0] (wave32) ou S1[5:0] (wave64) | Mascara o select com `wave - 1` antes de comparar; select 33 numa wave de 32 passa a escrever a lane 1 | **Correção nova**, não restauração. Não está no catálogo — mas é a mesma classe de **D-03** (máscara de wave64 restaurada não reconhecida) |
| `#139` | Shaders: s_min/s_max set SCC when S0 is chosen, not when the result is non-zero | SCC passa a ser o resultado da comparação que o select já calcula (duas linhas em `lower.rs`), em vez de "resultado ≠ 0" | **Correção nova**. Não está no catálogo |
| `#138` | GPU: GetDeviceRemovedReason reports the host device's state instead of E_NOTIMPL (restore 13185f5) | Slot 37 consulta o `GetDeviceRemovedReason()` do device hospedeiro e devolve o erro, ou `S_OK` | Idem, restaura `13185f5` |
| `#137` | Shaders: a loop whose count the code fixes runs to its end | Novo `emit/trips.rs` conta laços com contador constante (entrada constante em toda aresta, passo constante, uma aresta de volta decidida por `s_cmp` contra constante, latch como única saída) e usa `max(limite do estágio, count + 1)`, até 2^20 voltas; laços dirigidos por dado e com saída antecipada mantêm o limite atual. O arquivo entra em `translation_inputs!` para invalidar tradução em cache | **Correção nova**, com o cuidado de chave de cache que o acervo vem cobrando na classe de D-02 (em `victory-fork`) |
| `#136` | GPU diagnostics: count copy-back waits by fill caller and write declarations by binding | Contadores de espera de copy-back por chamador de fill e de declarações de escrita por binding | **Trabalho novo**, diagnóstico |
| `#135` | GPU: give buffer-to-texture copies slices of the lists' scratch chunks | Cópia buffer→textura passa a fatiar os blocos de rascunho das listas em vez de exigir bloco inteiro | **Trabalho novo**. Medido em RX 6650 XT / Ryzen 5 5600X3D (cabeçalho do corpo) |

## Pull requests mesclados (0)

Nenhum PR com `merged_at` maior que `last_run` (`2026-11-26T04:15:00Z`) na varredura de `pulls?state=closed`
paginada. O cursor de merge fica parado nesta execução.

## Cruzamento com o catálogo

- **O evento que o catálogo já nomeia é o que esta fila está consertando.** D-01 a D-09, D-13 e D-14 têm
  "introduzido em `570a941`" na coluna do catálogo. Verificado por API: `570a941` existe, é
  `570a941e9b4e945121c26eb30dfc10c126845207`, pai `940d32f6`, autor `DexNom`, 2026-11-25T12:18:03Z, 170
  arquivos, assunto "Publish DexNom Runtime Radeon sources with upstream credits and build instructions".
  Nove dos 14 PRs abertos restauram commits derrubados por ele (`44ce067`, `92f2c60`, `7d04943`,
  `78c1267`, `6f36cdf`, `e6fe8ec`, `e45bacd`, `bbdc3ca`, `13185f5`), todos por `cherry-pick -x` com o
  autor original — os corpos dos 9 dizem o mesmo e citam o SHA.
- **D-10, D-11 e D-12 seguem abertos — e nenhum PR os cobre.** O catálogo os marca como abertos por
  reversão (`subresource_len` ausente, `write_atomically` com `.tmp` de nome fixo, overflow do
  `clwb_loop`). Os 14 títulos/corpos não incluem nenhum deles; a fila restore ataca outra lista de
  commits. Isto é a lacuna da hora: o publish derrubou mais do que os 9 PRs restauram.
- **D-15 (`synthetic_tests.rs` apagado por `570a941`) — continua sem restauração** na fila.
- **D-04 (LOD com `ln * log2(e)` e clamp descartado) — não corrigido**, apesar de `#143` mexer em faixa
  de LOD de sampler: são caminhos diferentes (sampler × cálculo de LOD no shader).
- **Abre defeito novo?** Não verificado. `#137` mexe em limite de laço e promete `translation_inputs!`;
  `translation_inputs!` é o mecanismo de chave de cache usado por este repositório, então o risco de D-02
  (código gerado muda sem mudar a chave) depende de o arquivo novo estar mesmo listado — afirmação do
  corpo do PR, **não verificada no diff**.

## Confiança

- **Verificado por API:** head `4ad869278a348826516e9c3255ff3fd5a947c7c7`; `compare
  8114ff77...4ad86927` com status `ahead`, `ahead_by` 1; zero merges na janela; 14 PRs abertos com
  número > 134; `570a941` com pai, data, autor e 170 arquivos lidos da API.
- **Verificado por leitura:** o diff inteiro do único commit (README, +1/−1); título, base, data e corpo
  dos 14 PRs.
- **NÃO VERIFICADO:** nenhum patch de PR foi lido linha a linha; as asserções de "os testes falham em
  `main` e passam com a mudança" vêm dos corpos, e os links de CI apontam para um fork de terceiro
  (`11run11/VIctoryDex-11-run`), não para o repositório monitorado; as medições de hardware de `#135`,
  `#139` e `#140` (RX 6650 XT / RX 9070 XT) são declarações do corpo; não foi executado nada; não foi
  conferido no código do head se `570a941` de fato removeu cada um dos 9 commits restaurados — a
  evidência é a presença do commit no histórico e a ausência da mudança, afirmada no corpo.
