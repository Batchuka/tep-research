---
type: doc
componente: latex
assunto: setup e estrutura da monografia
status: atualizado
relates-to: []
tags: []
data: 2026-06-06
---

# doc_setup-latex-monografia

## Contexto
Registro do ambiente de escrita da monografia — o que foi instalado, como o projeto está organizado e como compilar.

## Instalação do ambiente LaTeX

1. **LaTeX Workshop** — extensão do VS Code (editor + compilador integrado)
2. **MiKTeX** — distribuição LaTeX para Windows (instala pacotes sob demanda)
3. **Perl** — obrigatório para scripts internos do LaTeX Workshop
   - Depois de instalar, adicionar o binário do Perl ao `PATH` nas variáveis de ambiente do sistema
   - Sem isso, a compilação falha silenciosamente em certas etapas

**Compilar:** `Ctrl+Alt+B` no VS Code

## Estrutura do projeto

O repo hospeda documentos LaTeX autocontidos (a monografia, os slides da defesa, dois artigos derivados e o plano do mestrado), que compartilham o `referencial/` (vault Obsidian), onde fica o `references.bib` único.

```
tep-research/
├── tcc-monografia/
│   ├── main.tex           ← entry point LaTeX da monografia
│   ├── ifes8.cls          ← classe da IFES (precisa ficar junto ao main.tex)
│   ├── latexindent.yaml
│   ├── trabalho/          ← capítulos .tex (Cap0 a Cap5)
│   └── pdf/               ← PDFs incluídos (folha, aprovacao)
├── tcc-slide/             ← slides da defesa (Beamer)
├── paper-arxiv/
│   ├── main.tex           ← entry point LaTeX do artigo (arxiv-style)
│   ├── arxiv.sty
│   └── orcid.pdf
├── paper-ieee/            ← artigo no modelo de conferência do IEEE
├── plano-pos/             ← plano de pesquisa do mestrado
├── docs/templates/        ← modelos originais (LaTeX e slides)
└── referencial/           ← vault Obsidian (notas + PDFs + config)
    ├── references.bib     ← bibliografia compartilhada
    ├── .obsidian/
    ├── articles/
    ├── books/
    ├── notes/
    └── standard/
```

## Capítulos

| Arquivo | Capítulo | Status |
|---------|----------|--------|
| `Cap0-resumo.tex` | Resumo (PT) | escrito |
| `Cap0-abstract.tex` | Abstract (EN) | escrito |
| `Cap1-introducao.tex` | Introdução | escrito — revisar |
| `Cap2-referencialteorico.tex` | Referencial Teórico | escrito — lapidar |
| `Cap3-desenvolvimento.tex` | Desenvolvimento | escrito — lapidar |
| `Cap4-resultados.tex` | Resultados | escrito — lapidar |
| `Cap5-conclusao.tex` | Conclusão | escrito — lapidar |

Para ativar um capítulo comentado no `main.tex`: remover o `%` antes do `\include{}` correspondente.

## Decisões tomadas

| Decisão | Alternativa descartada | Motivo |
|---------|------------------------|--------|
| Capítulos em `tcc-monografia/trabalho/` | raiz do projeto | separar LaTeX de notas Obsidian |
| `ifes8.cls` junto ao `main.tex` em `tcc-monografia/` | classe solta na raiz do repo | LaTeX exige classe no mesmo diretório do `main.tex`; mover `main.tex` para `tcc-monografia/` (revertendo a decisão anterior de nunca movê-lo) deixa o documento simétrico com os artigos e permite buildar cada um isoladamente |
| `references.bib` compartilhado, em `referencial/` | um `.bib` por documento | monografia e artigo citam a mesma literatura — evita divergência ao adicionar referências |
| `main.pdf` no `.gitignore` | versionar o PDF | evita conflito de merge a cada compilação (casa `main.pdf` em qualquer profundidade, cobre o `main.pdf` de cada documento) |

## Problemas em aberto / Próximos passos

- [ ] Arquivo `nota_setup_monografia.md` está desatualizado — este o substitui

## Referências internas

- [[]]
