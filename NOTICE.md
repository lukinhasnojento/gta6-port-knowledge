# Avisos legais, licenças e atribuição

## O que este repositório não contém

- Nenhum arquivo do jogo: sem executáveis, sem `.rpf`, sem assets, sem áudio, sem shaders originais.
- Nenhuma chave de descriptografia.
- Nenhum link para download de build vazada.
- Nenhum código-fonte copiado de projeto de terceiro.

Todo o conteúdo foi escrito a partir da leitura de **histórico git público**: `git log`,
`git show`, `git diff`, `git blame`. Trechos de diff aparecem como citação curta, para
evidência, sob os usos permitidos pelas licenças dos respectivos projetos.

## Projeto relacionado, não afiliado

Este acervo não é afiliado, endossado ou patrocinado pela Rockstar Games, Take-Two Interactive,
Microsoft ou AMD. *Grand Theft Auto*, *GTA VI* e marcas relacionadas pertencem aos seus titulares.

Este repositório é distribuído sob **GNU General Public License v3.0** (veja `LICENSE`).
Copyright (C) 2026 lukinhasnojento.

Os doze repositórios analisados são projetos independentes de terceiros. Suas licenças estão
listadas abaixo, conforme declaradas nos próprios repositórios no momento da análise.

## Licenças dos projetos analisados

| Repositório | Licença declarada |
|---|---|
| `Marvisak/VIctory` | GPL-3.0 |
| `ivall/xcl` | GPL-2.0 |
| `R0Y00/Xbox2Windows` | MIT |
| `californiaAlmonds/OpenScarlett` | MIT |
| `Belobug/VIctory-Belobug-AMD` | GPL-3.0 |
| `DexNom/VIctory` | GPL-3.0 (herdada) |
| `uiliannn/VItoriaBr` | GPL-3.0 (herdada) |
| `avatarmanz/VIctory-7900XTX` | GPL-3.0 |
| `fxceless/victory-fork` | GPL-3.0 (herdada) |
| `omonkeyz/xbox-emulator` | GPL-3.0-or-later |
| `brcly/TuxBox` | verificar no repositório |
| `holdmysocks/SixNG` | GPL-3.0 |
| `walkthestars/GTA6-Asset-Info` | GPL-3.0 |

## Bloqueio encontrado — leia antes de consolidar código

`avatarmanz/VIctory-7900XTX` apresenta uma incompatibilidade de licença que **impede o uso direto
do código dele** em qualquer obra derivada GPL-3.0:

- O `README.md:108` afirma: *"XCL (GPL-2.0-only) was read for how the same title behaves under
  another layer; **no code of it is here**."*
- O cabeçalho de `console_insn.rs` afirma: *"**Ported from XCL** (GPL-2.0-only): `insn.rs`
  (decoder, as `console_insn_decode.rs`, **unchanged**)"*.
- O `LICENSE` do repositório é **GNU GPL v3**.

GPL-2.0-**only** e GPL-3.0 são incompatíveis: a segunda não permite relicenciamento de código que
a primeira fixou na versão 2. Somente a forma "GPL-2.0-or-later" permite a atualização. Como o
arquivo foi portado sem modificação, ele permanece sob GPL-2.0-only, e o restante do projeto sob
GPL-3.0 — o que torna a distribuição combinada problemática.

**Consequência prática para quem for construir o repositório unificado:** não copie
`console_insn.rs` nem `console_insn_decode.rs`. Se a funcionalidade for necessária — ela é
necessária, porque emula `CLZERO`, `MONITORX`, `MWAITX` e as instruções SSE4a que o executável do
jogo usa —, reimplemente do zero a partir do comportamento documentado (o `STATUS_ILLEGAL_INSTRUCTION`
que essas instruções geram em CPU Intel), sem consultar a implementação GPL-2.0-only.

O agravante é que **este acervo e o repositório unificado pretendido são GPL-3.0**. Incorporar
código GPL-2.0-only violaria a licença em dois sentidos: pela incompatibilidade entre as versões e
pelo relicenciamento indevido. A regra vale para qualquer arquivo, de qualquer fork, cujo cabeçalho
declare `GPL-2.0-only` — não só para este.

Isto é uma observação técnica sobre compatibilidade de licenças, não assessoria jurídica.

## Atribuição

As análises foram produzidas com **DeepSeek V4.1 Flash** em subagentes isolados, sobre artefatos
gerados por `git` puro. O método completo está em `METODO.md`.

O acervo se apoia no trabalho dos autores dos doze projetos. Cada análise cita os hashes e
arquivos que permitem rastrear qualquer conclusão até o commit original. Se este material for útil,
o crédito pelo código é deles; os erros de análise são nossos.

Dados de ranking e contexto de ecossistema: **PortWatch VI** (`portwatch-vi.xyz`), tracker
independente não afiliado à Rockstar nem à Take-Two.

## Correções

Se uma conclusão deste acervo estiver errada, ela deve ser corrigida com o mesmo padrão que foi
usado para escrevê-la: hash do commit, caminho de arquivo e comando que reproduz a evidência.
Discordância sem evidência reprodutível não é correção.
