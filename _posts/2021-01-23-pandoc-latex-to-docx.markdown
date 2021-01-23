---
layout: post
title:  "Pandoc: convert latex file to docx"
date:   2021-01-23 19:00:00 +0100
categories: pandoc latex docx
---

I want to convert latex files to docx so that I can share them with my supervisor. 
For this I use [pandoc][pandoc].

I use the following command to convert the .tex document (from latex) to .docx, I specify the bibliography and the output file:
``` 
pandoc MAThesisToLatex.tex -f latex -t docx --bibliography=../../bibliography.bib -s -o test.docx
```

The output contains some errors: some figures were not found, some captions are at the wrong place and the bibliography is not perfect, but it's a good start.

[pandoc]:https://pandoc.org/getting-started.html