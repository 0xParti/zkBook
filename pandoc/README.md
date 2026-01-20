# Compiling the Book

From this directory (`markdown/full/`), run:

```bash
pandoc zkBookFull_compilation.md --template=eisvogel -o zkBook.pdf --pdf-engine=xelatex
```

Requires:
- [Pandoc](https://pandoc.org/)
- [Eisvogel template](https://github.com/Wandmalfarbe/pandoc-latex-template)
- XeLaTeX (via TeX Live or MacTeX)
