# Falhas e desvios — 2026-11-28 14:00

## Repositórios inacessíveis (HTTP 404)

- `Belobug/VIctory-Belobug-AMD` — `repos/Belobug/VIctory-Belobug-AMD` devolveu `Not Found (HTTP 404)`
  nesta coleta (oitava hora seguida). `default_branch` não reconferida, cursores parados em
  `4eb6cfdf` / `a-publier`, triagem NAO VERIFICADO.
- `avatarmanz/VIctory-7900XTX` — o mesmo 404, oitava hora. Cursores parados em `014d52de`.

Nenhum dos dois saiu de `repos`; nenhuma escrita foi feita neles.

## Desvios

- **Ordem do `compare` não é estável.** Em `xcl` o `commit_cursor` gravado (`e5266254`) apareceu no
  índice 2 da página atual, não no 40. As quatro janelas desta hora foram escolhidas por **hash**
  (primeiro commit cuja SHA não aparece em documento anterior), não por índice. O efeito colateral é
  que o novo cursor não é o 41º da ordem antiga; é o 41º da ordem atual.
- **`last_run` antigo em três repositórios.** `xcl` (04:12:54Z), `TuxBox` (01:16:56Z) e
  `Xbox2Windows` (05:24:33Z) tinham cursores de merge mais antigos que merges já documentados nos
  ciclos de 12:00/13:00. A deduplicação foi feita por hash de merge contra todo
  `docs/updates/**/*.md`; os merges reconhecidos não foram reescritos. O novo `last_run` de cada um
  avança até o maior `merged_at` efetivamente coberto.
- **A "tabela de pistas" citada no passo 2 do procedimento não existe** em
  `docs/automacao/ANALISE_HORARIA.md`. As áreas do critério 3 foram tomadas das seções D do
  `catalogo/defeitos.md` e do corpo dos PRs, como nos ciclos anteriores.
- **Nenhum subagente disponível** neste ambiente; os quatro repositórios foram processados em
  sequência.
- **Rate limit:** nenhuma resposta 403. Consumo da coleta registrado no `state/last-run.md`.
- **Escrita:** somente em `G:\gta6-port-knowledge`. Os 14 monitorados foram apenas leitura de API.
