# Teste de ray tracing bloqueado e comparação de FPS entre forks — 2026-11-27

## Resultado

O teste local bloqueou os dispatches que percorrem estruturas de ray tracing
(`VICTORY_BVH_HOLD=1`) e deixou o construtor de BVH desligado
(`VICTORY_BVH_BUILDER=0`). O log confirma dispatches bloqueados, mas a cena
observada continuou em aproximadamente 1–2 FPS. Portanto, ray tracing não foi
identificado como causa única da baixa taxa de quadros desta execução. O teste
não reproduziu a cena específica em que as texturas pretas haviam sido relatadas.

Há, sim, medições documentadas acima de 20 FPS em outro fork: `AvatarManz/VIctory`
registra 19–22 FPS na primeira missão de barco e 17–24 FPS voando. São resultados
de outra máquina, GPU, configuração e cenas; não constituem comparação A/B com
este teste nem demonstram que o mesmo desempenho será obtido neste sistema.

## Execução local

- Fingerprint do build executado: `b99f876b74d4a77f03af5f878223b27bd647b5c35a79d5decd9f716504239e93`
  (o log identifica a revisão como `unknown`; o fingerprint não é apresentado como commit).
- Variáveis relevantes: `VICTORY_BVH_BUILDER=0` e `VICTORY_BVH_HOLD=1`.
- O log registrou cerca de 15 dispatches por frame bloqueados por
  `VICTORY_BVH_HOLD=1`; também registrou estágios do builder descartados porque
  o builder estava desligado.
- Na cena junto à grade, a imagem estava visível e não mostrou texturas pretas
  evidentes no quadro observado. A cena problemática original não foi isolada,
  então isso não prova correção de textura.
- A execução chegou a aproximadamente 1–2 FPS. Um resumo de frame do log
  registrou 716 ms por frame em média, 422 ms no thread de renderização,
  365,2 ms de preparação no thread de submissão, 286,8 ms em listas da placa e
  cerca de 4.184 draws. Os componentes de tempo são métricas do log e não devem
  ser somados como se fossem intervalos independentes.
- No mesmo resumo havia aproximadamente 1.799 MB livres no contador reportado
  pelo processo/sistema. A memória privada do processo chegou a cerca de
  25,1 GB numa amostra anterior desta execução.

**Interpretação:** bloquear os dispatches de ray tracing não restaurou o
desempenho nesta cena. O perfil observado é compatível com carga substancial na
preparação/submissão e processamento de listas, além de milhares de draws; os
dados não isolam uma causa única nem excluem efeitos de memória, streaming ou
compilação. É necessária comparação controlada na mesma cena, build e hardware.

## Comparação com resultados documentados de outros forks

| Fork e evidência | Máquina/configuração registrada | Resultado registrado | Limitação |
|---|---|---|---|
| `AvatarManz/VIctory`, análise baseada no README | i9-12900K, RX 7900 XTX 24 GB, 64 GB RAM, Windows 11, 1440p, perfil Series X; script `scripts/play.ps1` ativa cerca de 20 switches | 19–22 FPS na primeira missão de barco; 17–24 FPS voando | Outra GPU, configuração e cenas; são resultados relatados pelo fork, não medição reproduzida neste teste. |
| `fxceless/victory-fork`, README | i7-14700KF, RTX 4070 12 GB, 32 GB RAM, Series X | Cerca de 8–11 FPS em cenas do mundo com builder desligado; 0,2–3 FPS com builder ligado | As medidas são de 26–27/11 e de cenas específicas. |
| `VItoriaBr`, análise do acervo | RTX 5060 Ti 16 GB | Cerca de 1,7–6 FPS no mundo | Build, configuração e cenas diferem; não é comparação controlada com os outros forks. |

Os resultados acima de 20 FPS são plausíveis e estão registrados, mas não
significam que todos os forks, builds ou cenas atinjam esse nível. Menus a
30 FPS também não são evidência de 30 FPS durante gameplay.

## Fontes e artefatos

- Log desta execução:
  `G:\irineu6-logs\pr1450-1442-rayhold-run-20261127.err.log`
- Saída padrão:
  `G:\irineu6-logs\pr1450-1442-rayhold-run-20261127.out.log`
- `docs/analises/AvatarManz.md`, seção **Estado atual medido** e configuração.
- `G:\fxtexture-latest\README.md`, estado do fork e medições de frame.
- `docs/analises/VItoriaBr.md`, estado atual medido.
- `docs/relatorios/prebuild-pipelines-pr1226-2026-11-27.md`, resultado de
  0,7–3 FPS com draws descartados após pré-aquecimento.

## Limitações

O teste foi uma única execução, sem baseline simultâneo na mesma cena com
`VICTORY_BVH_HOLD=0`. A captura visual foi observada na janela, mas não foi
arquivada como screenshot comparativo. Os números entre forks vêm de máquinas,
builds, switches e cenários diferentes; servem como contexto, não como ranking
controlado. Não foi verificado se os resultados documentados de `AvatarManz`
foram reproduzidos independentemente.
