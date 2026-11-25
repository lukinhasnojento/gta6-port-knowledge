# Método

Como o acervo foi produzido, e por que ele pode ser usado sem confiar na memória de um modelo.

## 1. Princípio

Toda afirmação factual sobre um repositório precisa de uma referência que o leitor possa
reproduzir: hash de commit, caminho de arquivo e o comando que produz a evidência. Onde isso não
foi possível, o documento diz `NÃO VERIFICADO` em vez de preencher a lacuna.

Isso não é formalidade. As análises encontraram commits cuja mensagem aponta para um subsistema e
cujo diff altera outro completamente — o caso mais direto é `c95a909` em `xbox-emulator`: a
mensagem fala de *"Exec mask"* e *"PSO use-after-free fix in register_pso"*, e o diff é
integralmente o servidor de eventos ETW, sem um único arquivo de `register_pso`. Nenhuma conclusão
pôde ser tirada de mensagem de commit sem abrir o diff.

## 2. Ingestão

Para cada repositório, antes de qualquer análise:

1. `git clone` completo, verificado com `git rev-parse --is-shallow-repository` → `false`
2. `git fetch --all --prune --tags` e comparação do `HEAD` local com o `sha` que a API do GitHub
   reporta para a branch default
3. contagem comparada com a do GitHub (`Link: rel="last"` na API de commits)

O passo 3 não é zelo excessivo: três dos clones estavam desatualizados no momento da verificação
— `XCL` com 835 de 853 commits, `SixNG` com 99 de 303, `Xbox2Windows` com 1.145 de 4.076. Um
desses projetos recebeu ~2.000 commits em uma hora e meia.

## 3. Artefatos preparados por git puro

Nenhum modelo participou desta etapa. É o alicerce que impede a alucinação.

| Artefato | Comando de origem | Função |
|---|---|---|
| `docs/indices/*_log_full.txt` | `git log --all --format="COMMIT %H%nDATE %ad%nAUTHOR %an%nSUBJECT %s%n%n%b"` | histórico completo, com corpos de mensagem |
| `docs/indices/*_log_compact.txt` | `git log --all --format="%h\|%ad\|%an\|%s"` | índice verificável |
| `docs/fatos/*_fatos.md` | `rev-list --count`, `log --max-parents=0`, `--name-only`, `--merges` | dossiê factual: totais, autores, commits por dia, arquivos mais tocados |
| pacote de evidências | `git show --unified=2` em 45 commits candidatos | diffs reais por commit |
| `docs/fatos/FORK_DIVERGENCIA.md` | `git merge-base --is-ancestor` + comparação de blob | genealogia e divergência entre forks |

O filtro de candidatos usa `git log -i --grep=` com os termos `fix`, `bug`, `regress`, `crash`,
`revert`, `hotfix`, `workaround`, `broken`, `fail`, `leak`, `race`, `hang`, `error`, `overflow`,
`wrong`, `stale`, `mismatch`. Esse filtro **só seleciona candidatos** — não decide nada.

## 4. Regra central da análise

Mensagem de commit não é evidência. As instruções dadas a cada análise foram:

- decidir **pelo diff** se o commit corrigiu defeito, introduziu defeito, ou é ruído;
- não aceitar `fix:` na mensagem como prova de correção;
- verificar se o problema voltou: `git log <hash>..HEAD -S"<trecho alterado>"`;
- reconstruir a cadeia completa `problema → causa → tentativa → resultado → correção definitiva`;
- classificar cada conclusão em `CONFIRMADA`, `PROVÁVEL`, `HIPÓTESE` ou `NÃO VERIFICADA`;
- nunca inventar hash, issue, PR ou resultado de teste.

Resultado prático: das 281 entradas do catálogo, 139 têm hash de introdução e 192 têm hash de
correção. Onde o autor do projeto não registrou a causa, o campo diz `SEM CORREÇÃO` ou
`NÃO IDENTIFICADO` em vez de uma suposição.

## 5. Formato de saída

Cada análise segue as mesmas 12 seções, para que os documentos sejam comparáveis:

`A` Identificação · `B` Resumo técnico · `C` Linha do tempo · `D` Catálogo de defeitos ·
`E` Relação de causa e efeito · `F` Correções importantes · `G` Problemas ainda existentes ·
`H` Regressões · `I` Conhecimento reutilizável · `J` Conclusão técnica · `K` Índice de commits ·
`L` Limitações

Em `D`, cada defeito recebe um ID (`D-01`, `D-02`, …) e responde: descrição, sintomas, causa raiz,
commit de introdução, tentativas de correção, correção definitiva, arquivos, evidência, estado
atual e confiança.

## 6. Escolha do modelo

Seis modelos foram executados na primeira rodada, com as mesmas instruções: `Iceman Contributor
Free`, `Hy4 flash Free`, `HungerG Free`, `Beam Free`, `Muse Spark 1.3 Contributor` e `DeepSeek
V4.1 Flash`.

Os cinco primeiros foram descartados. Não por velocidade: por **rastreabilidade**. Nenhum
entregou a estrutura exigida de forma autônoma, e as contagens de hashes citados ficaram entre 0
e 21 por documento, contra 511–1.145 do modelo escolhido. Sem hash, uma conclusão não é
auditável — e um acervo não auditável é pior que nenhum, porque parece confiável.

Todas as 12 análises finais foram produzidas por **DeepSeek V4.1 Flash**, rodando em subagentes
isolados, um por repositório, em paralelo. Cada um recebeu apenas o próprio repositório e seus
artefatos — nenhum viu o trabalho dos outros.

## 7. Verificação independente

Depois de cada análise entregue, o orquestrador reconferiu conclusões no git, à mão. Onze
passaram; uma apontou um erro do próprio orquestrador, corrigido em `docs/fatos/FORK_DIVERGENCIA.md`.

O protocolo completo está em `VERIFICACOES.md`. Ele existe porque a pergunta certa não é "o
modelo parece confiável?", e sim "esta afirmação específica se sustenta contra o repositório?".

## 8. Limitações conhecidas

- **Issues e pull requests não foram varridos em massa.** Apenas os números citados em mensagens
  de merge. São 1.830 PRs em `Xbox2Windows` e 1.395 em `VIctory` de discussão inexplorada.
- **Commits fora da branch principal** foram contados e amostrados, não analisados um a um
  (755 no total, incluindo 332 do `SixNG` e 423 do `Xbox2Windows`).
- **`Zettrox/ztrx`**, 3º colocado no ranking do PortWatch no momento da coleta, retornou 404 e
  ficou sem cobertura.
- **Profundidade é desigual.** O documento do `Xbox2Windows` tem 4.618 linhas em parte porque
  inclui o índice completo de 4.076 commits; a análise interpretativa é comparável às demais.
