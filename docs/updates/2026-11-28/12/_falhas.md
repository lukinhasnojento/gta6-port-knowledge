# Falhas e desvios — 2026-11-28 12:00

## Repositórios inacessíveis (não interrompeu os outros)

| Repositório | Falha | Horas seguidas | Efeito |
|---|---|---|---|
| `Belobug/VIctory-Belobug-AMD` | `gh api repos/Belobug/VIctory-Belobug-AMD` e `gh api users/Belobug` devolvem **HTTP 404** ("Not Found"). | 6 | `default_branch` não reconferida (o estado grava `a-publier`), `compare` não calculado, triagem NAO VERIFICADO, cursor parado. |
| `avatarmanz/VIctory-7900XTX` | Idem: `repos/` e `users/` devolvem HTTP 404. | 6 | Idem (o estado grava `main`). |

Ambos continuam em `repos` no estado: a rotina não remove entrada. Se voltarem, os cursores antigos
(`4eb6cfdf` e `014d52de`) fazem o delta dos últimos dias ser recalculado e a triagem seguinte mede o
atraso real.

## Desvios do procedimento

1. **A "tabela de pistas" citada no passo 2 não existe** em `docs/automacao/ANALISE_HORARIA.md`. O
   arquivo tem as seções "Estado", "Limites e falhas" e "Publicação", mas nenhuma tabela de áreas por
   repositório. O desvio já está no `INDEX.md` desde 2026-11-27 17:00. As áreas do sinal vieram das
   seções D do `catalogo/defeitos.md` e do assunto/corpo de cada commit e PR; o sinal dos repositórios
   sem commit é por título e corpo de PR, não por patch.
2. **Sem subagentes.** A ferramenta não existe neste ambiente; os quatro repositórios foram processados
   em sequência. O limite de 4 deep dives foi respeitado.
3. **Descrição de PR aberto sem patch.** Os 39 PRs abertos de `Xbox2Windows`, 36 de `xcl`, 11 de
   `TuxBox` e 3 de `SixNG` (89 no total, além de 6 em repositórios sem deep dive) foram lidos por
   título e corpo. Nenhum patch de PR aberto foi baixado.
4. **`xcl` em violação estrutural do teto.** 376 commits de atraso contra 40 documentáveis por hora;
   336 ficam. Não é falha desta execução, é o desenho: o repositório precisa de ~9 horas de deep dive
   exclusivo. Registrado no documento dele e no `backlog_note`.
5. **Cursor de commits com fronteira aproximada.** Nos três repositórios que bateram o teto, o
   `commit_cursor` é o 40º commit documentado, e `compare` a partir dele não devolve exatamente o
   resto da janela (86 contra 81 em `Xbox2Windows`, 148 contra 129 em `TuxBox`, 338 contra 336 em
   `xcl`). A causa é o 40º ser, em dois casos, um merge cuja linhagem contém commits já documentados.
   Medido e registrado; o próximo ciclo deve reconferir com essa diferença em mente.

## Coleta

- **Rate limit:** 4.999 restantes no início, 4.678 no fim (13:13Z de reset), nenhum 403, nenhuma
  repetição em laço.
- **Escrita:** somente em `G:\gta6-port-knowledge`. Os 14 monitorados foram só leitura de API.
- **Relógio:** nesta coleta os timestamps do GitHub estão **alinhados ao UTC real** — commit
  `2b2277a5` do `SixNG` com data `12:16:23Z` apareceu às 12:19Z locais. Os `last_run` gravados são o
  maior `merged_at` documentado de cada repositório, não o horário da execução.
- **Arquivos de terceiros no diretório de trabalho** (`docs/analises/victory-fork.md` modificado e os
  dois relatórios não rastreados em `docs/relatorios/`) ficam fora do `git add` desta rotina.
