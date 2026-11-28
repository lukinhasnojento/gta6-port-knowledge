# Falhas e desvios — 2026-11-28 16:00

## Repositórios inacessíveis (404, décima hora)

`Belobug/VIctory-Belobug-AMD` e `avatarmanz/VIctory-7900XTX` devolvem HTTP 404 em `repos/` e em
`users/`. `default_branch` não reconferida, cursores parados, triagem NAO VERIFICADO. Não saíram de
`repos` e não interromperam os outros.

## Contagem de repositórios: 15, não 14

O estado tem 15 entradas em `repos`. A 15ª, `trustno1-coder/VIctory-NVIDIA`, foi acrescentada às
15:50Z (commits `0eac98c` e `bc6c64b`, tabela do procedimento corrigida para 15 linhas), depois da
execução das 15:00 que ainda falava em 14. Ela foi triada nesta hora com `last_pr_number: null`:
100 PRs abertos lidos, máximo #1226. O cursor de PRs não avançou porque nenhum foi lido por diff —
o `last_run` de 15:50:07Z é o da inclusão. Nada foi removido: 15 no início e 15 no fim.

## "Tabela de pistas" continua inexistente

`docs/automacao/ANALISE_HORARIA.md` não tem a tabela de pistas que o passo 2 e o critério 3 citam. As
áreas foram tiradas das seções D do catálogo e do corpo dos PRs, como nas horas anteriores. O desvio
está registrado desde 2026-11-27 17:00 e não foi corrigido.

## Ordem do `compare` instável (duas ocorrências)

1. **`xcl`:** o novo `commit_cursor` é o 40º dos 128 commits documentáveis, `3cde0cc3`.
   `compare(3cde0cc3...bb9a6b4b)` devolve 114 commits e re-inclui `2dad32e6` e `abf7e177`, que já
   foram documentados às 12:00 (o primeiro com o `[unroll]` do ballot de pixel, o segundo com o teto
   de RAM/8). O próximo ciclo tem de pular hash já documentado, ou vai duplicar dois commits por hora
   até o cursor passar deles.
2. **`TuxBox`:** a fila zerou nesta hora; não há duplicata pendente.

## `#3100` do `xcl` mesclado como draft

O corpo do PR diz "This remains a draft until the required clean main/head smoke blocks are
available" e lista como não testados o smoke main/head, o build release no commit rebaseado e
adaptadores AMD/Intel. O merge `bb9a6b4b` entrou mesmo assim. Está registrado no documento, não é
falha da rotina.

## Cobertura de diff em `DexNom`

Dos 52 PRs abertos, 4 foram lidos por diff (`#183`, `#194`, `#156`, `#200`); os outros 48 foram
classificados por título, corpo, lista de arquivos e `+/-` da API. A restauração de hash só foi
conferida linha a linha no `#183`, que é o que aciona o critério 1.

## `#173` do `DexNom` fora de escopo

O PR "Vanviro 16gb" adiciona 1.016.414 linhas (vendoriza `cold-project-vi/` inteiro). Não foi lido e
não deve ser mesclado; está listado como "não é para merge", junto com `#155` ("Findings (not for
merge)").

## Outros

- **Nenhum subagente disponível** neste ambiente; os quatro repositórios foram processados em
  sequência.
- **Rate limit:** nenhum 403 nesta coleta.
- **Escrita:** somente em `G:\gta6-port-knowledge`; os 15 monitorados foram só leitura de API.
- **`Marvisak/VIctory`:** a leitura de 100 PRs devolve só o topo; 100 acima de #3521 é piso, não
  total. O cursor ficou parado.
- **Coleta de mesclados:** feita por
  `pulls?state=closed&sort=updated&direction=desc&per_page=100`, paginando até `updated_at` cair
  abaixo do `last_run`, e filtrando `merged_at > last_run`, como manda a nota do estado. Não se usou
  `gh pr list --search`.
