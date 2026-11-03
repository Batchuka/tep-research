## Artigo WiP (IEEE)

Artigo de trabalho em andamento (Work in Progress) para o ICIT 2027 (IEEE International Conference on Industrial Technology, Florianópolis, 17 a 19/03/2027), no modelo de conferência do IEEE (classe `IEEEtran`, opção `conference`). Limite de 4 páginas. Prazos: submissão em 3/11/2026, aceite em 15/12/2026, versão final e inscrição em 18/01/2027. A submissão é pelo sistema da IES, num trilho que começa com `WiP-`; a revisão é *single blind* e a apresentação é presencial.

## Build

Entrada: `main.tex`. A bibliografia vem de `../referencial/references.bib` (não duplicar citações aqui; adicionar direto nesse arquivo), com o estilo `IEEEtran.bst`. As figuras são desenhadas no próprio `main.tex`; os dados das Figs. 3 e 4 ficam em `dados/`, extraídos de `tep-lab/data/experiment_82` e `experiment_89`.

```
latexmk -pdf main.tex
```

## Arquivos

- **`main.tex`**: o artigo.
- **`versoes/`**: PDF e fonte de cada versão guardada (`main-v1.0.0`).

A classe `IEEEtran.cls` e o estilo `IEEEtran.bst` vêm da instalação do LaTeX (pacote `ieeetran` do MiKTeX). O modelo oficial do IEEE e os manuais ficam em [`../docs/templates/latex/ieee/`](../docs/templates/latex/ieee/).
