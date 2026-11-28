# Teste de pré-aquecimento de pipelines — PR #1226 (2026-11-27)

## Resultado

A pré-compilação do catálogo inteiro foi concluída e a biblioteca resultante foi
carregada num novo teste. O jogo avançou pela abertura até o prólogo sem
reproduzir o watchdog `+0x23cf683` durante aproximadamente quatro minutos de
observação. Entretanto, a cena ficou em cerca de 0,7–3 FPS e descartou centenas
ou milhares de draws por frame. O resultado é promissor quanto a evitar o
watchdog nesta inicialização, mas não demonstra jogabilidade estável nem uma
correção geral.

## Pré-compilação

- O pack reportou 28.164 registros. Ao final, 20.933 pipelines foram criados e
  6.192 já estavam na biblioteca; 1.039 foram ignorados: 852 estados de
  rasterização não nomeados, 139 estados depth-stencil não nomeados, 3 estados
  blend não nomeados, 8 shaders compute não traduzíveis, 8 shaders ausentes,
  2 registros com estágios não suportados e 27 pipelines recusados pelo host.
- A biblioteca foi salva com aproximadamente 1,08 GB. Embora o comando tenha
  terminado com código de saída 1, emitiu o resumo final e salvou a biblioteca;
  a causa do código de saída ainda não foi isolada. Portanto, o resultado da
  compilação é funcionalmente utilizável, mas o código de saída requer
  investigação.

## Teste de execução

- A execução reconheceu a biblioteca como produzida pelo mesmo tradutor e
  conjunto de switches e carregou aproximadamente 1,08 GB. A lista de pipelines
  prioritários foi reconhecida.
- O jogo avançou pela abertura até o prólogo. Em cerca de quatro minutos, não
  houve ocorrência do watchdog nem aviso de estouro do limite de espera de
  compute.
- Depois da transição para a cena do prólogo, os frames observados variaram
  aproximadamente entre 0,7 e 3 FPS, com cerca de 1.000–7.700 draws por frame e
  aproximadamente 350–7.100 draws descartados. A memória de processo chegou a
  cerca de 14 GB; o indicador do sistema mostrou memória livre próxima de 1 GB.
- Ainda houve avisos de recusa do host para alguns pipelines compute por
  parâmetro inválido. Não foi observado crash durante a janela do teste.

## Ensaio inicial

Antes da compilação completa, um ensaio focado usou um conjunto de 1.468 chaves
prioritárias: 653 pipelines foram criados nessa etapa e 35 vieram da biblioteca
parcial. Esse teste também chegou ao prólogo sem reproduzir o watchdog na janela
observada. Ele foi um resultado preliminar; a execução posterior com o pack
completo é a evidência mais relevante.

## Limitações

O teste completo é uma única execução numa configuração específica; não houve
comparação controlada suficiente para atribuir causalidade exclusiva ao
pré-aquecimento. A biblioteca não cobre os 1.039 registros ignorados e alguns
pipelines ainda são recusados durante a execução. Os PRs #1450 e #1442 não foram
incluídos no build testado; uma execução anterior com essas mudanças apenas
forneceu chaves prioritárias para o ensaio inicial.

## Conclusão

O pré-aquecimento do PR #1226 evitou o watchdog até a chegada ao prólogo nesta
execução e reduziu a necessidade de criar pipelines durante o início. O gargalo
de desempenho, os draws descartados e o código de saída 1 da compilação
permanecem sem solução. É necessária repetição controlada e investigação
separada desses problemas; não se deve tratar o watchdog como corrigido em geral.
