# Testes do limite de espera e do lock da pipeline library — 2026-11-27

## Resultado

Dois testes locais combinaram a alteração de limite de espera de pipelines de
computação do PR #1450 com a serialização de loads da pipeline library do PR
#1442. Nenhum dos testes evitou o watchdog `+0x23cf683`.

## Observações

- Com o limite padrão de 8 segundos, o aviso de limite ocorreu, mas um frame
  subsequente durou aproximadamente 15,8 segundos e o watchdog foi atingido.
- Com limite de 1 segundo, houve aviso de limite e descarte de milhares de
  draws enquanto pipelines ainda eram criados. Um frame de aproximadamente
  15,5 segundos terminou novamente no mesmo watchdog; o jogo não chegou à
  etapa de gameplay.
- Os testes usaram caches separados e vazios. A hipótese de que o lock de
  loads não relacionados explicasse, por si só, o travamento não foi sustentada
  por esses resultados.

## Conclusão e limitações

Os resultados indicam que reduzir o limite de espera ou adicionar o lock do
PR #1442 não foi suficiente para evitar o watchdog nessa configuração. A causa
do frame longo permanece sem explicação. São execuções locais específicas, não
uma prova de que essas mudanças sejam ineficazes em todos os ambientes.

Esses testes são contexto para o teste posterior de pré-aquecimento documentado
em [`prebuild-pipelines-pr1226-2026-11-27.md`](prebuild-pipelines-pr1226-2026-11-27.md).
