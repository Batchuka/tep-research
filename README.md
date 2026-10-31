# Supervisão do Processo Tennessee Eastman com Kubernetes

Documentos da pesquisa de Matheus Corteletti Delfino sobre supervisão declarativa de plantas industriais com Kubernetes, usando o Processo Tennessee Eastman (TEP): o TCC (Engenharia de Controle e Automação, Ifes campus Serra, defendido em 29/10/2026), os artigos derivados e o plano do mestrado. O código fica em outro repositório.

Cada documento LaTeX tem o próprio `main.tex` e compila sozinho.

## Documentos

| Pasta | O que é | Arquivo de entrada |
| --- | --- | --- |
| [`tcc-monografia/`](tcc-monografia/) | A monografia do TCC (ABNT, classe `ifes8`). Capítulos e apêndices em `trabalho/`, figuras em `imagens/`. O [README](tcc-monografia/README.md) traz o mapa da monografia e o inventário de referências. | `main.tex` |
| [`tcc-slide/`](tcc-slide/) | Slides da defesa (Beamer, modelo institucional do Ifes), roteiro da fala e transcrição da defesa. | `main.tex`, [`roteiro.md`](tcc-slide/roteiro.md) |
| [`paper-arxiv/`](paper-arxiv/) | Artigo derivado da monografia, no estilo arXiv. Ainda é o esqueleto do modelo. Ver o [README](paper-arxiv/README.md). | `main.tex` |
| [`paper-ieee/`](paper-ieee/) | Artigo no modelo de conferência do IEEE. Ainda é o modelo. | `conference-latex-template/` |
| [`plano-pos/`](plano-pos/) | Plano de pesquisa do mestrado: Kubernetes na supervisão de uma planta industrial. | `main.tex` |

## Material de apoio

- [`references.bib`](references.bib): bibliografia única, usada por todos os documentos.
- [`referencial/`](referencial/): vault Obsidian com as notas de fonte (`notes/`) e os PDFs (`articles/`, `books/`, `standard/`). Em `new/` ficam os PDFs que ainda não têm nota.
- [`docs/`](docs/):
  - **Modelos:** [`templates/`](docs/templates/), só os usados: LaTeX (Ifes, arXiv, IEEE) e o modelo de slides do Ifes. Ver o [README](docs/templates/README.md).
  - **Ambiente:** [`doc_setup-latex-monografia.md`](docs/doc_setup-latex-monografia.md) (instalação e compilação) e [`doc_obsidian_setup.md`](docs/doc_obsidian_setup.md) (plugins do vault).
  - **Normas resumidas:** [`IEC_62264.md`](docs/IEC_62264.md), [`IEC_62541.md`](docs/IEC_62541.md), [`IEC_63303.md`](docs/IEC_63303.md).
  - **Narrativa da monografia:** [`br_narrativa.md`](docs/br_narrativa.md) e [`br_narrativa_08_10_26.md`](docs/br_narrativa_08_10_26.md).
  - **Leitura de fontes:** os prompts `doc_prompt_*.md` e as respostas em [`respostas/`](docs/respostas/).
  - **Diagramas:** [`miscellanea.drawio`](docs/miscellanea.drawio). O mapa geral da monografia está no [README da monografia](tcc-monografia/README.md#big-picture).
  - **Planta Festo do Ifes:** manuais e exercícios em [`planta-festo-ifes/`](docs/planta-festo-ifes/), para o mestrado.

## Compilação

Em cada pasta de documento:

```
latexmk -pdf main.tex
```

Os PDFs e arquivos auxiliares não são versionados (ver `.gitignore`). Detalhes do ambiente em [`docs/doc_setup-latex-monografia.md`](docs/doc_setup-latex-monografia.md).
