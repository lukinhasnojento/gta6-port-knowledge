# Falhas e desvios — 2026-11-28 15:00

## Repositórios inacessíveis (nona hora)

`Belobug/VIctory-Belobug-AMD` e `avatarmanz/VIctory-7900XTX` devolvem **HTTP 404** em `repos/<o>/<r>`
e em `users/<o>`. `default_branch` não reconferida, cursores de commit e de merge parados, triagem
NAO VERIFICADO, nenhum documento escrito. Não saíram de `repos` no estado. Nono ciclo consecutivo
(primeiro registro: 2026-11-27 21:00).

## Tabela de pistas inexistente

O passo 2 do procedimento manda usar a "tabela de pistas", que **não existe** em
`docs/automacao/ANALISE_HORARIA.md`. As áreas usadas para o critério 3 vieram das seções D de
`catalogo/defeitos.csv` e do corpo de cada PR. Desvio registrado desde 2026-11-27 17:00; continua sem
correção no procedimento.

## Ordem do `compare` instável

A ordem de `repos/<o>/<r>/compare/<cursor>...<head>` não é cronológica nem topológica estável: em
`xcl`, o commit `8eb35ad7` (40º da janela) tem data `2026-11-26T11:46:22Z`, anterior a `8fd21391`
(11:47Z) e `85c26b1e` (11:48Z), que aparecem depois. As quatro janelas foram escolhidas por **hash**
(o primeiro commit cuja SHA não aparece em nenhum documento de `docs/updates/**/*.md`), e o novo
`commit_cursor` é o 41º dessa ordem atual. Não há garantia topológica de que ele seja ancestral de tudo
o que ficou para trás — o mesmo desvio do ciclo anterior.

## Cursor preso em `victory-fork` (corrigido nesta execução, com desvio)

O ciclo de 13:00 gravou `commit_cursor = 368967f8` em `fxceless/victory-fork`, com a nota de que
aquele era "o nono commit, que chegou depois da coleta e não está documentado", e `head_at_run = 368967f8`.
Como o cursor de commits é usado em `compare(<cursor>...<head>)`, e o cursor **é** o commit pendente,
todo ciclo seguinte devolve zero e o commit nunca entraria em documento nenhum. O estado do ciclo de
14:00 já mostrava `1 commit pendente` na triagem e `0` no `compare`, sem que ninguém notasse a causa.

Nesta execução o commit foi documentado (`docs/updates/2026-11-28/15/victory-fork.md`) e o cursor
avançou para `368967f8`, com `commit_cursor`/`head_at_run` removidos. Isso consumiu o **4º slot** de
deep dive, que o critério de mérito não daria a esse repositório (sinal fraco). Registrado como desvio,
não como seleção por sinal.

**Correção sugerida para o procedimento:** `commit_cursor` deve apontar para o primeiro commit
**não documentado** e o `compare` deve ser feito a partir do **pai** dele, ou o `compare` do ciclo
seguinte deve incluir o próprio cursor. Do jeito que está escrito, o cursor perde exatamente o commit
que ele nomeia.

## Hash documentado que não existe (`TuxBox` `#377`)

`docs/updates/2026-11-28/14/TuxBox.md` e `state/last-run.md` registram o merge do `#377` como
`3eb76bd5`. `repos/brcly/TuxBox/commits/3eb76bd5` devolve **404**; o merge real é
`3eb76bd901b39f9121f0524ddf4dd0e00392b96b`, com `merged_at = 2026-11-28T09:22:22Z` e assunto
"Merge pull request #377 from NickStallman/xcrdapi/page-array-at-run". O hash correto está registrado
em `docs/updates/2026-11-28/15/TuxBox.md` (Confiança). Não reescrevi o documento publicado de 14:00 —
corrigir o acervo é decisão à parte desta rotina.

## Leitura truncada de PRs abertos em `Marvisak/VIctory`

`gh pr list --state open --limit 400` devolveu 400 PRs com 203 acima do cursor `#3521` e máximo
`#3753`. O repositório pode ter mais de 400 abertos; a contagem "203" é um **piso**, não o total. A
triagem registra o piso.

## Nenhum subagente disponível

Este ambiente não expõe a ferramenta de subagente; os quatro repositórios foram processados em
sequência, como nos ciclos anteriores.
