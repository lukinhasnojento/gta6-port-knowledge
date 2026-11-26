# GTA VI PC port — acervo de conhecimento histórico

Documentação técnica acumulada sobre os doze projetos públicos que tentam rodar a build de
Xbox Series de GTA VI no PC: o que cada um corrigiu, o que quebrou, o que continua quebrado e
por quê.

O objetivo é servir de base para um repositório unificado que nasça com os defeitos conhecidos
já corrigidos e sem repetir os erros históricos. Este repositório contém **apenas documentação**:
nenhum arquivo de jogo, nenhuma chave, nenhum executável.

## O que há aqui

| Caminho | Conteúdo |
|---|---|
| `docs/analises/` | 12 análises históricas, uma por repositório, em 12 seções (A–L) |
| `docs/comparativo/` | 12 documentos de acertos e defeitos + `COMPARATIVO_GERAL.md` com a matriz |
| `docs/relatorios/` | 2 relatórios consolidados, um por rodada de análise |
| `docs/fatos/` | 15 dossiês factuais gerados **por git puro**, sem nenhum modelo de linguagem |
| `docs/indices/` | 14 índices de commits (hash, data, autor, assunto) — a evidência bruta |
| `catalogo/defeitos.csv` | 281 defeitos catalogados, com hashes de introdução e correção |
| `METODO.md` | Como a análise foi feita e quais regras impediram invenção |
| `VERIFICACOES.md` | 11 conclusões reconferidas à mão, com o comando e o resultado |
| `NOTICE.md` | Licenças, atribuição e um bloqueio jurídico encontrado |

## Números

| Métrica | Valor |
|---|---|
| Repositórios analisados | 12 |
| Commits cobertos | 9.209 |
| Defeitos catalogados | 281 |
| Defeitos com hash de introdução identificado | 139 |
| Defeitos com hash de correção identificado | 192 |
| Defeitos com confiança declarada | 96 (91 CONFIRMADA, 5 PROVÁVEL) |
| Páginas de análise | 44 arquivos, 3,4 MB |

Os 185 defeitos sem confiança declarada no CSV não são de qualidade inferior: são os casos em que
a extração automática não conseguiu localizar o campo no texto original. A classificação está
no documento de análise correspondente.

## Repositórios cobertos

**Camada de compatibilidade e portes**

| Repositório | Commits | O que é |
|---|---|---|
| `Marvisak/VIctory` | 241 | Camada Rust que roda o executável Xbox nativamente no Windows |
| `ivall/xcl` | 853 | Camada estilo Wine, traduz shaders RDNA2 para DXIL |
| `R0Y00/Xbox2Windows` | 4.076 | Pesquisa de recompilação estática, estudo de caso GTA VI |
| `californiaAlmonds/OpenScarlett` | 347 | Camada C++20 D3D12/Vulkan para jogos Xbox Series |

**Forks do VIctory**

| Repositório | Commits | Posição no PortWatch |
|---|---|---|
| `Belobug/VIctory-Belobug-AMD` | 751 | 1º — 77,4% |
| `DexNom/VIctory` | 547 | 2º |
| `uiliannn/VItoriaBr` | 373 | 4º |
| `avatarmanz/VIctory-7900XTX` | 621 | 5º |
| `fxceless/victory-fork` | 253 | base da maioria dos builds privados |

**Projetos independentes**

| Repositório | Commits | O que é |
|---|---|---|
| `omonkeyz/xbox-emulator` | 595 | Runtime por DLL, conformance contra o `xgameruntime.dll` real |
| `brcly/TuxBox` | 591 | Camada Linux-first, testada via Proton |

**Ferramentas e formatos**

| Repositório | Commits | O que é |
|---|---|---|
| `holdmysocks/SixNG` | 303 | Extrator de assets RPF9 |
| `walkthestars/GTA6-Asset-Info` | 5 | Notas de formato de asset (só documentação) |

## Como usar

Se você quer entender **um** projeto: comece em `docs/analises/<repo>.md`, seção D (catálogo de
defeitos) e seção I (conhecimento reutilizável).

Se você quer **construir** o repositório unificado: leia `docs/relatorios/` primeiro, depois
`docs/fatos/FORK_DIVERGENCIA.md` (a genealogia entre os forks) e use `catalogo/defeitos.csv`
como fila de trabalho.

Se você quer **auditar** uma conclusão: cada defeito tem hash; `docs/indices/<repo>_log_compact.txt`
permite conferir cada um com `git show`.

## Avisos

**Nada aqui é código de jogo.** Não há executáveis, arquivos `.rpf`, chaves de descriptografia nem
assets. Todo o conteúdo foi derivado de repositórios públicos, lendo histórico e diffs.

**As análises são texto nosso, não código de terceiros.** Trechos de diff são citados para
evidência. Ver `NOTICE.md` para o detalhamento de licenças e para um **bloqueio jurídico real**
encontrado em um dos forks que precisa ser resolvido antes de qualquer consolidação de código.

**A precisão é variável por natureza.** Cada conclusão traz o grau de confiança que tinha quando
foi escrita. Onde não havia evidência, está escrito `NÃO VERIFICADO` — isso é intencional e é a
razão pela qual o acervo pode ser usado.

## Licença

**GNU General Public License v3.0** — veja [`LICENSE`](LICENSE).

```
GTA VI PC port — acervo de conhecimento histórico
Copyright (C) 2026 lukinhasnojento

Este programa é software livre: você pode redistribuí-lo e/ou modificá-lo
sob os termos da GNU General Public License, versão 3, conforme publicada
pela Free Software Foundation.

Este programa é distribuído na esperança de ser útil, mas SEM NENHUMA
GARANTIA; sem sequer a garantia implícita de COMERCIALIZAÇÃO ou ADEQUAÇÃO
A UM PROPÓSITO ESPECÍFICO. Veja a GNU General Public License para mais
detalhes.

Você deve ter recebido uma cópia da GNU General Public License junto com
este programa. Se não, veja <https://www.gnu.org/licenses/>.
```

A escolha é coerente com o ecossistema: os projetos analisados são majoritariamente GPL-3.0, e
o repositório unificado que este acervo pretende alimentar também será. Usar a mesma licença evita
atrito na hora de incorporar correções vindas deles.

O texto em `LICENSE` é cópia verbatim do original da Free Software Foundation
(35.149 bytes, 674 linhas), sem modificação — a própria licença proíbe alterá-la.
