# Supervisão Declarativa do Processo Tennessee Eastman com Kubernetes

Trabalho de conclusão de curso de Matheus Corteletti Delfino, Engenharia de Controle e Automação, Ifes campus Serra. Defesa em 29/10/2026.

Este repositório reúne a monografia e os documentos ligados a ela. Cada documento LaTeX tem o próprio `main.tex` e compila sozinho.

## Documentos

| Pasta                            | O que é                                                                                                                                                            | Arquivo de entrada                                  |
| -------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------- |
| [`monografia/`](monografia/)     | O TCC (ABNT, classe `ifes8`). Capítulos e apêndices em `trabalho/`, figuras em `imagens/`. O [README](monografia/README.md) explica como as fontes foram fichadas. | `main.tex`                                          |
| [`apresentacao/`](apresentacao/) | Slides da defesa (Beamer, modelo institucional do Ifes), roteiro da fala e transcrição da defesa.                                                                  | `main.tex`, [`roteiro.md`](apresentacao/roteiro.md) |
| [`plano-pos/`](plano-pos/)       | Plano de pesquisa do mestrado: Kubernetes na supervisão de uma planta industrial.                                                                                  | `main.tex`                                          |
| [`paper/`](paper/)               | Artigo derivado da monografia (estilo arXiv). Ainda é o esqueleto do modelo. Ver o [README](paper/README.md).                                                      | `main.tex`                                          |

## Material de apoio

- [`references.bib`](references.bib): bibliografia única, usada pela monografia, pelo artigo e pelo plano do mestrado.
- [`referencial/`](referencial/): vault Obsidian com as notas de fonte (`notes/`) e os PDFs (`articles/`, `books/`, `standard/`). Em `new/` ficam os PDFs que ainda não têm nota.
- [`docs/`](docs/):
  - **Ambiente:** [`doc_setup-latex-monografia.md`](docs/doc_setup-latex-monografia.md) (instalação e compilação) e [`doc_obsidian_setup.md`](docs/doc_obsidian_setup.md) (plugins do vault).
  - **Normas resumidas:** [`IEC_62264.md`](docs/IEC_62264.md), [`IEC_62541.md`](docs/IEC_62541.md), [`IEC_63303.md`](docs/IEC_63303.md).
  - **Narrativa da monografia:** [`br_narrativa.md`](docs/br_narrativa.md) e [`br_narrativa_08_10_26.md`](docs/br_narrativa_08_10_26.md).
  - **Leitura de fontes:** os prompts `doc_prompt_*.md` e as respostas em [`respostas/`](docs/respostas/).
  - **Diagramas:** [`miscellanea.drawio`](docs/miscellanea.drawio). O mapa geral da monografia está no [README da monografia](monografia/README.md#big-picture).
  - **Modelos do Ifes:** [`modelos-apresentacao-institucional-odp/`](docs/modelos-apresentacao-institucional-odp/), [`modelos-apresentacao-institucional-ppt/`](docs/modelos-apresentacao-institucional-ppt/) e [`modelo-padroes-acessibilidade/`](docs/modelo-padroes-acessibilidade/).
  - **Planta Festo do Ifes:** manuais e exercícios em [`planta-festo-ifes/`](docs/planta-festo-ifes/), para o mestrado.

## Compilação

Em cada pasta de documento:

```
latexmk -pdf main.tex
```

Os PDFs e arquivos auxiliares não são versionados (ver `.gitignore`). Detalhes do ambiente em [`docs/doc_setup-latex-monografia.md`](docs/doc_setup-latex-monografia.md).
