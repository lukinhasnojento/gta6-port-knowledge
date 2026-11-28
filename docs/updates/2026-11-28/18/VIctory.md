# Marvisak/VIctory — atualização de 2026-11-28 18:00

## Resumo

A `main` não se moveu (`63bcca43`, `identical`): a hora é toda de PRs abertos, 12 criados entre 16:57Z e
18:05Z. Dois têm sinal: `#3774` recusa descritor uniforme **carregado** que escapa do laço numa prova de
forma de waterfall (área de `D-01`/`D-06`) e `#3771` corrige a resposta de `FORMAT_SUPPORT` para
`DXGI_FORMAT_UNKNOWN`. O resto é otimização de emissão de shader com número medido no próprio corpo e
captura de GPU. Zero commits, zero merges.

## Commits novos (0)

`compare/63bcca43...63bcca43` devolve `identical`, `ahead_by = 0`. O cursor `63bcca43` já era o head na
execução de 17:00 e continua sendo.

## Pull requests abertos (12 novos; 50 lidos, ≥50 acima do cursor #3521)

O cursor `last_pr_number` é 3521 e o menor número lido é #3722, então a janela de 50 não prova cobertura
dos PRs antigos — os 12 abaixo são os criados depois de 16:55Z.

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| #3775 | docs: record RX 5500 XT performance findings and prepared patches | Documento de 70 linhas com os experimentos de Linux/RX 5500 XT: cópia de constantes de GPU (~13%), janelas de malha texturizada em software (~3%), captura de diagnóstico segura em submissão, guarda de descritor carregado, write-watch de alocação grande (crédito `NVIDIA #1038`) e regressão de página suja concorrente. Distingue correção de ganho e registra que a meta de 20 fps não foi atingida | Não |
| #3774 | Shaders: add waterfall shape proof with loaded-descriptor escape refusal | Porta de `VIctory-NVIDIA #458` a prova de forma estrita para waterfall sobre offset de tabela de primeira lane (o traçador por lane não segue a fonte) e acrescenta a recusa própria: descritor uniforme **carregado** que escapa do laço entra no conjunto de escape (valores SSA do descritor), com checagem de leituras fora e de entradas de phi; depois do folding, código após o laço veria outro descritor que a passada escalar final do console. Regressão lê `s12`, dword real de descritor carregado, em wave32 e wave64. `resource.rs` +364/−1 | **Área de `VIctory D-01`** (draw lê o descriptor do título) e `D-06` (descritor lido de `v_readfirstlane_b32` tratado como "o de cada lane"). Hipótese por área, sem hash |
| #3773 | Shaders: fold literals in packed f16 conversion toward zero | Dobra operandos diretos e não modificados de `v_cvt_pkrtz_f16_f32` quando o literal é zero ou tem resultado half normal finito, preservando sinal do zero, truncando significando e saturando overflow para o maior half do mesmo sinal. Mede 4.653 palavras de saída iguais à `main` em 423 shaders e 8 instruções DXIL menos por módulo | Não; toca a classe de `XCL D-06`/`D-07` (arredondamento f16) por ser conversão, mas é ganho de emissão, não correção |
| #3772 | GPU: reuse page buffers in tracking and copy-back | Quatro limpezas de CPU: pool de 256 páginas (16 MiB) em vez de alocar 64 KiB por primeiro fault em `watch.rs`; `PageScratch` por thread em `shadow.rs`; `binary_search` em `take_split` (a lista é construída em ordem de página, com `debug_assert`); `Drop for Tracked` percorre só as páginas detidas | Não. Área de `D-09` (janelas de memória mapeada declaradas escritas) e de `VItoriaBr D-01` (custo de `VirtualQuery`), sem hash |
| #3771 | GPU: report unformatted buffer capabilities for DXGI_FORMAT_UNKNOWN | **Corrige contrato de API:** a consulta `FORMAT_SUPPORT` aplicava as máscaras default ao formato 0, anunciando amostragem de textura e render target para buffer não formatado. Passa a devolver `BUFFER` e `TILED`. Medido: convidado em `63bcca4` devolve `0x7fffffff`/`0x0000ffff`, WARP nativo devolve `0x00000001`/`0x00000200` | Não há linha equivalente no catálogo do `VIctory`; é contrato de API, classe de `Xbox2Windows D-12` (retorno de API divergente do runtime) |
| #3770 | GPU capture: store R32 integer preview samples in 32 bits | Guarda amostras de captura R32 inteira como palavras de 32 bits (para `R32_SINT`, inverte o bit de sinal para manter ordem numérica) e mantém a conta de contraste em aritmética larga. Mede pico de heap de 159,01 → 96,09 MB e mediana 327 → 223 ms num 4K `R32_SINT` gerado | Não |
| #3769 | Shaders: omit redundant masks after high-half reads | Ler a metade alta de um operando i32 já limpa os 16 bits superiores por shift lógico; `half_src`, `packed_src` e `fmac_half` emitiam um AND com `0xFFFF` depois. Mantém a máscara só para leitura de metade baixa. Mede 25 → 23 e 121 → 118 instruções DXIL | Não |
| #3768 | Shaders: skip source masks for wrapping halfword arithmetic | Add/sub/produto baixo/shift esquerdo de meia palavra só precisam dos 16 bits baixos no store final: seleciona as metades pedidas e deixa a máscara de resultado truncar. Mede 418.560 palavras iguais à `main` em 48 shaders e 2 instruções menos nas 32 formas medidas | Não |
| #3767 | Bound SGD5 fallback name scanning | O scanner de fallback do SGD5 andava para trás por toda a corrida imprimível antes do nome, embora o campo de comprimento de 1 byte só identifique candidatos nos últimos 255 bytes (com o NUL). Limita as duas buscas de `name_before` a esse sufixo, preservando ordem e heurística; terminator fora de faixa devolve `None` | Não (custo de CPU) |
| #3765 | Formats: combine XMA pre-rotation and the first FFT stage | Calcula os pesos de pré-rotação em ordem de bits invertida para os dois pontos da primeira borboleta chegarem juntos e aplica a borboleta assim que o par está pré-rotado. Mede 524.160 amostras idênticas bit a bit e mediana de transformada 11–16% menor em 128/256/512 amostras | Não |
| #3764 | Shaders: avoid redundant sign extension in v_ashrrev_i16 | `v_ashrrev_i16` estendia sinal das duas meias-fontes no caminho comum e de novo no braço do shift; a contagem é mascarada em 4 bits, então estender a contagem não muda nada e o valor só precisa da extensão no braço. Pula as extensões comuns para esse opcode. Mede 31 → 27 e 32 → 28 instruções por forma de `op_sel` | Não |
| #3763 | GPU capture: stream compressed PNG chunks to disk | Escreve os chunks PNG por um writer com buffer, mantém o payload IDAT comprimido na alocação original e libera as scanlines antes de abrir o arquivo. Mede pico de alocação viva 31,11 → 18,92 MB no 4K de alta entropia e PNG byte a byte igual em cinco capturas | Não |

## Pull requests mesclados (0)

A consulta de pulls fechados (`state=closed`, ordenada por `updated`) devolveu 50 PRs, nenhum com
`merged_at` maior que `2026-11-27T13:20:45Z`. `last_run` de mesclados fica parado no valor anterior, de
propósito: avançar para o horário da execução esconderia merge futuro (a lição dos dois relógios).

## Cruzamento com o catálogo

- **Corrige D-xx?** Por hash, nenhum: os 12 PRs não carregam hash do `catalogo/defeitos.csv` e não há
  commits na janela.
- **Por área (hipótese):** `#3774` é o mais próximo de uma linha do catálogo — `VIctory D-06` descreve
  "descritor lido de `v_readfirstlane_b32` tratado como o de cada lane em qualquer lugar → amostragem
  errada", e a prova de forma que o PR porta é exatamente o que decide quando o offset de tabela é
  uniforme e quando cada lane tem o seu. `D-01` (draw lê o descriptor do título) fica na mesma família.
  Marcar como **área**, não como correção: o PR não cita nenhum dos hashes de correção do catálogo.
- **Abre defeito novo?** Nenhum identificado. Observação de processo, não de defeito: `#3774` porta
  código de `trustno1-coder/VIctory-NVIDIA #458` preservando o autor em commit separado, e o
  repositório monitorado 15 (`VIctory-NVIDIA`) teve, nesta mesma hora, só um workflow de issues — a
  procedência do patch não pôde ser conferida contra a origem nesta execução (**NAO VERIFICADO**).
- **Divergência declarada:** `#3775` admite que três implementações de renderizador precisam de
  fundamentos ausentes da `main` atual e que a meta de 20 fps não foi atingida; o PR é documentação, não
  ganho.

## Confiança

Verificado: `compare/63bcca43...63bcca43` = `identical`; a lista de 50 PRs abertos com número, data de
criação, branch e título; para os 12 documentados, corpo (resumo), lista de arquivos com contagem de
linha e `headRefName`, via `gh pr view`. **NAO VERIFICADO:** todos os números medidos nos corpos (palavras
DXIL, MB de heap, ms, contagem de shaders) — são afirmação de autor, não reprodução; que os testes das
branches passem; que `#3774` cubra de fato a linha `D-06` (a área bate, o hash não); e a existência do
`#458` de origem no `VIctory-NVIDIA`. Contagem: 0 commits, 0 merges, 12 PRs abertos documentados de ≥50
acima do cursor. Head `63bcca43`; `last_pr_number` avança para 3775.
