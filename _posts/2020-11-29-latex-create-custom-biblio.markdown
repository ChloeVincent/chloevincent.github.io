---
layout: post
title:  "Create a custom bibliography in LaTeX"
date:   2020-11-29 19:00:00 +0100
categories: latex bibtex custom-bib
---


In this article I will report on my experience of creating a custom BibTeX style. 


I had previously switch from BibTeX to biblatex and Biber because it offers more styles. 
However, for my MA dissertation, the style had to follow the [Linguistic Society style][lingSocStyle], and I could not find the exact same style in the predefined option of biblatex.


## Creating a custom style

I used the tool [custom-bib][custom-bib]. After downloading it and unzipping it, the only thing to do is to run the following commands: 

```sh
latex makebst.ins
latex makebst.tex
```

The tool will then ask many questions about the desired style and it will create a .dbj file. 

If the job to create the .bst is not run right at the end of the questions, run the following: 
```sh
latex <dbjfile.dbj> 
```

## Using the newly created style

In my tex file, in the header I add (cf [LaTeX Wikibook][latex-wiki] for details on natbib options)
```latex
\usepackage[round, comma]{natbib}
```
(instead of `\usepackage[backend=biber,style=apa,natbib=true]{biblatex}`)


And at the end of my document, where I want my bibliography to appear, I add:
```latex
\bibliographystyle{MAling}
\bibliography{/home/<fullPathToBiblio>/bibliography.bib}
```
(instead of `\printbibliography`, where `MAling` is the bst file previously created and `<fullPathToBiblio>` is the full path to where my .bib file is located)


## Citing references
To get the desired output in the text body, I used `\citet` which outputs "Author (year)" and a new command `\ctp` (for CiteT Parenthesis):
```latex
\newcommand{\ctp}[1]{(\citealt{#1})} % outputs "(Author year)"
```


## Installing the style for further use
I have not tried this, since the style I created was a one time thing. But this might be useful for later. ([source][install-style])
```sh
mkdir -p ~/texmf/bibtex/bst
cp mystyle.bst ~/texmf/bibtex/bst/
texhash ~/texmf # Make TeX aware of what you just did
```


## Note on manually changing the bst
After all this I realised I need the year was in parenthesis where I actually wanted it to be without. 
So I looked into the bst file and searched for "(", but it did not seem to match.
So I looked for the keyword year, which was surrounded by `\harvardyearleft` and `\harvardyearright`, I tried removing those and it worked. 

Before:
```
  " \harvardyearleft " swap$ * "\harvardyearright{}" *
```
After:
```
  " " swap$ * "" *
```




It is clearly not perfect (there are colon instead of points, editors are not properly referenced), but it's a start!

[lingSocStyle]:https://www.linguisticsociety.org/sites/default/files/style-sheet_0.pdf
[custom-bib]:https://www.ctan.org/pkg/custom-bib
[install-style]:http://gabrielelanaro.github.io/blog/2014/12/01/latex-bibliography-in-5-minutes.html
[latex-wiki]:https://en.wikibooks.org/wiki/LaTeX/Bibliography_Management#Customization