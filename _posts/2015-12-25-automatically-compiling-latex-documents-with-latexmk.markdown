---
title: "Automatic Compilation of LaTeX"
date: 2015-12-25 21:09:34 +0100
comments: true
categories: latex
---

Recently, I found the [latexmk](https://www.ctan.org/pkg/latexmk) tool, which can automatically figure out the order of LaTeX and BibTeX compiles, making sure you never get ``undefined references`` in the LaTeX log file or ``??`` in the PDF due to the wrong compile order.

On top of that, latexmk can recompile LaTeX documents upon file changes, in any LaTeX source file that makes up your final output document. It can easily be integrated into a Makefile, as I show here.

The example Makefile below configures latexmk to compile a document using XeLaTeX. If you are using pdfTeX or LuaTeX, use the ``-pdf`` or ``-luatex`` flags instead of ``-xelatex`` in the ```latexmkopts``` variable.

It defines 3 tasks along with the default ```make``` task, which takes care of creating the final output PDF, after a couple of LaTeX and bibliography compile calls. The ```watch``` task keeps a continuous process running, watching for source file changes, and recompiling if necessary.

```make
# Example Makefile for a XeLaTeX project
root = master # name of root .tex file
latexmkopts = -xelatex -shell-escape

all: $(root).pdf

$(root).pdf: $(root).tex
	latexmk $(latexmkopts) $(root)

# Remove auxiliary files
clean:
	rm -rf `biber --cache`
	rm -rf _minted-$(root)
	latexmk -bibtex -CA

# Auto-compile on file changes
watch: clean
	latexmk $(latexmkopts) -pvc $(root)

# Quick and dirty compile task
quick: clean
	xelatex -shell-escape $(root)
```

For packages with auxiliary files that aren't automatically removed by ```make clean```, you can define a ```.latexmkrc``` file next to the ```Makefile```, containing extensions that are to be auto-removed. For example, the excellent [todonotes](https://www.ctan.org/pkg/todonotes) package creates a bunch of filler files, which are removed by the below line in the ```.latexmkrc``` file.

```bash
# .latexmkrc
push @generated_exts, "run.xml", "tdo";
```

Hope this helps someone!
