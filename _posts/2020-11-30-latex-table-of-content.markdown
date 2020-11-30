---
layout: post
title:  "Customize the Table of Content in LaTeX"
date:   2020-11-30 19:00:00 +0100
categories: latex toc appendix
---


In this article I described how I customized my Table of Content (ToC) for my MA dissertation. 

To print the ToC in LaTeX I simply use the command `\tableofcontents`. 

I want my bibliography to appear in it. I got 2 choices: 

### Bibliography 1
Use the package `tocbibind`:
```latex
\usepackage[nottoc,numbib]{tocbibind} % adds the bibliography to the Table of Content
```

I did not like this solution, because it adds a bibliography section, which is numbered. Removing the numbib option resolve the issue: 

```latex
\usepackage[nottoc]{tocbibind} % adds the bibliography to the Table of Content
```

Note that this changes to name of the bibliography section to "Bibliography" instead of "Reference"

### Bibliography 2

The other option is to add the following line where the bibliography is printed: 
```latex
\addcontentsline{toc}{section}{References} % add biblio to ToC
```
Apparently this can cause issue in case the bibliography starts on a new page, so the previous solution is probably better.

### Appendices

To change how the appendices are presented in the ToC, I used the package [appendix][appendix]:
```latex
\usepackage[titletoc, page, toc]{appendix} % to add 'Appendix' before the appendix in the Table of Content (to use with \begin{appendices})
```
`titletoc` changes the appendix entries to "Appendix A ... " (instead of "A ..."), `page` adds a "Appendices" title at the start of the appendices, and `toc` adds the title before the list of appendices in the ToC.

The different strings can be changed thanks to the following options in the header: 
```latex
\renewcommand{\appendixpagename}{Appendices} % Section name in the body
\renewcommand{\appendixtocname}{List of appendices} % name of section in ToC
```
and right before the appendices:
```latex
\renewcommand{\appendixname}{Annex} % name of entry in ToC

```

Also I had to change from `\appendix` to `\begin{appendices}...\end{appendices}` to make it work.

### Ignore some sections
I did not want some section to appear in the ToC (Abstract, Introduction, Conclusion), so I added a ' \* ' after the command `\section`: 
```latex
\section*{Introduction}
```

Those section are not numbered and do not appear in the ToC.

[appendix]:http://tug.ctan.org/macros/latex/contrib/appendix/appendix.pdf