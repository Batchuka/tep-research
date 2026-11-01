## Artigo TEP

Artigo derivado da monografia (`../tcc-monografia/`), usando o estilo arXiv-preprint ([arxiv-latex-template](https://github.com/kourgeorge/arxiv-latex-template), baseado em `nips_2018.sty`). Ver `License.txt` para a licença do estilo.

## Build

Entry point: `main.tex`. Bibliografia compartilhada com a monografia via `../referencial/references.bib` (não duplicar citações aqui; adicionar direto nesse arquivo).

```
latexmk -pdf -cd main.tex
```

## Arquivos

- **`main.tex`** — o artigo.
- **`arxiv.sty`** — estilo (não reimportar `geometry`/`fancyheader`, já usados internamente).
- **`orcid.pdf`** — ícone usado no cabeçalho de autoria.

## Submetendo ao arXiv

O ambiente TeX do arXiv não roda BibTeX. Antes de submeter:
1. `latex main` / `bibtex main` (gera `main.bbl`)
2. Colar o conteúdo de `main.bbl` dentro de `\begin{thebibliography}` em `main.tex`
3. Comentar a linha `\bibliography{../referencial/references}`
