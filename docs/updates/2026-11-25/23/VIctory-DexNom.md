# DexNom/VIctory — atualização de 2026-11-25 23:00

## Resumo

Dois commits novos, ambos só de `README.md` e ambos de autoria `Codex`: documentam os modos
experimentais de NVIDIA e as prioridades de contribuição do RayTime. Nenhuma linha de código mudou.
Há um PR aberto (#62) sobre 13 programas traduzidos que derrubam o compilador da AMD numa RX 9060 XT.

## Commits novos (2)

| Hash | Autor | Assunto | O que o diff faz |
|---|---|---|---|
| `64bd58f5` | Codex | Document experimental NVIDIA test modes | `README.md` +17/-1: o parágrafo sobre NVIDIA passa a apontar para uma seção nova `## Experimental NVIDIA`, que dá o comando `--no-rt` com `VICTORY_FSR_4K=0` e `VICTORY_AMBIENT_FALLBACK=1`, descreve o preenchimento ambiente difuso do `--no-rt`, o modo `--rt-static-bvh` e o que reportar (GPU, driver, tempos de frame, log `logs\latest.txt`). O diff não toca código: nenhum `crates/` aparece nos arquivos |
| `f08fee57` | Codex | Document RayTime contribution priorities in the README | `README.md` +12/-0: acrescenta a frase do baseline FSR Quality 4K e uma seção `## Things to Be Done` com três frentes (reduzir copy-back rastreando páginas escritas pela GPU; reduzir VRAM e estabilizar streaming de textura; estabilizar tempo de frame), cada uma com links para arquivos do código, mais o protocolo de medição antes/depois. Só documentação |

Ordem cronológica: `64bd58f5` antes de `f08fee57`.

## Pull requests abertos (1)

| # | Título | Proposta | Cobre defeito conhecido? |
|---|---|---|---|
| 62 | Keep AMD's shader compiler (RX 9060 XT) from failing on 13 translated programs | Na RX 9060 XT (driver `32.0.32029.2010`, `amdxc64.dll` `8.18.10.0514`), criar os pipelines de 13 programas traduzidos faz o compilador da AMD falhar — access violation lendo `0x8` dentro de `amdxc64.dll`, `victory warm` morre com `0xC0000005` | Não há linha para isso no catálogo `DexNom` (D-01…D-16). É a mesma classe do **D-06 de Belobug-AMD** ("Compilador AMD: pipelines que nunca terminam ou crasham"), em outro repositório e com outro driver — não é o mesmo defeito nem o mesmo hash |

## Pull requests mesclados (0)

| # | Título | Merge | O que entrou de fato |
|---|---|---|---|
| — | — | — | nenhum PR com número > 61 aparece como mesclado |

## Cruzamento com o catálogo

- **Corrige D-xx?** Não. Os dois commits mexem só em `README.md`; nenhum dos defeitos `DexNom`
  D-01…D-16 tem correção por estes hashes.
- **Abre defeito novo?** Candidato documental: `f08fee57` promete que o `--no-rt` usa "a lightweight
  diffuse ambient fill based on their base color" (`64bd58f5`) e manda o leitor a
  `crates/victory-shader/src/emit/mod.rs`. Nenhum dos dois commits verifica que esse caminho existe
  no código — o README passa a afirmar comportamento que esta rotina **não conferiu**.
- Contexto útil para o catálogo: o README agora declara publicamente que a configuração NVIDIA é
  **não testada** e que o modo RT padrão chega a "frame times … several seconds". Os defeitos
  abertos D-10, D-11 e D-12 (reversões e overflow do `clwb_loop`) continuam sem correção.

## Confiança

- **Verificado:** os dois commits, seus diffs completos (+17/-1 e +12/-0), arquivo único `README.md`,
  e que nenhum arquivo fora dele entrou.
- **Verificado:** o PR #62, título, autor (`MrToucann`) e driver/versões citados no corpo.
- **NAO VERIFICADO:** se o comportamento descrito no README corresponde ao código — não abri
  `crates/victory-shader/src/emit/mod.rs` nem `crates/victory-gpu/src/pipeline/ray_tracing.rs`, e a
  rotina não executa nada do repositório monitorado.
- **NAO VERIFICADO:** o diff do PR #62.
