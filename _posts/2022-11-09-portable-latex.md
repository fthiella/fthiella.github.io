---
layout: post
title: "Portable LaTex"
date: 2022-11-09 22:40:00 +0200
categories: latex
---
Dato che ho l'esigenza di creare diversi documenti in formato LaTex
nella mia macchina Windows 11. Esistono varie soluzioni disponibili,
ma mi è sembrato che la versione [MikTex](https://miktex.org/)
fosse la più vicina alle mie esigenze.

Tra l'altro si trova disponibile anche in versione Portable che ho deciso
di installare sotto la mia cartella usuale:

     c:\Users\[username]\Documents\sw\

Nella sezione [MikTex Downloads](https://miktex.org/download) si trova
la versione Windows, ed è possibile scegliere l'installer standard,
la versione "portable" oppure la versione installabile a riga di comando,
utile per distribuire il software in automatico su numerosi computer aziendali.

La versione "portable" in realtà non richiede un installer separato: va semplicemente
scaricato l'installer per windows e rinominato in `miktex-portable.exe`. Una volta
eseguito è necessario assicurarsi di scegliere la seguente cartella di destinazione:

    c:\Users\[username]\Documents\sw\miktex\

al cui interno si troverà lo script `miktex-portable.cmd` pronto per essere lanciato.

Tale script una volta lanciato apre una piccola icona nella barra di windows, dalla quale
è possibile lanciare il terminale TeXworks. Oppure, in alternativa, è possibile
eseguire ogni programma che si troverà nel seguente percorso:

    c:\Users\[username]\Documents\sw\miktex\texmfs\install\miktex\bin\x64\

Un file di partenza che ho incominciato ad utilizzare è il seguente:

````latex
\documentclass{article}

\usepackage{graphicx}
\usepackage{amsmath}

\input{./pandoc_syntax.tex}

\begin{document}
    \input{./title.tex}
    \tableofcontents
    \newpage

    \section{Hello World!} % creates a section
    
    \textbf{Hello World!} Today I am learning \LaTeX. %notice how the command will end at the first non-alphabet charecter such as the . after \LaTeX
     \LaTeX{} is a great program for writing math. I can write in line math such as $a^2+b^2=c^2$ %$ tells LaTexX to compile as math
     . I can also give equations their own space: 
    \begin{equation} % Creates an equation environment and is compiled as math
    \gamma^2+\theta^2=\omega^2
    \end{equation}
    If I do not leave any blank lines \LaTeX{} will continue  this text without making it into a new paragraph.  Notice how there was no indentation in the text after equation (1).  
    Also notice how even though I hit enter after that sentence and here $\downarrow$
     \LaTeX{} formats the sentence without any break.  Also   look  how      it   doesn't     matter          how    many  spaces     I put     between       my    words.
    
    For a new paragraph I can leave a blank space in my code.
\end{document}
````

dove ho importato la seguente definizione dei colori:

````latex
% color syntax from pandoc
\usepackage{color}
\usepackage{fancyvrb}
\newcommand{\VerbBar}{|}
\newcommand{\VERB}{\Verb[commandchars=\\\{\}]}
\DefineVerbatimEnvironment{Highlighting}{Verbatim}{commandchars=\\\{\}}
\newenvironment{Shaded}{}{}
\newcommand{\AlertTok}[1]{\textcolor[rgb]{1.00,0.00,0.00}{\textbf{#1}}}
\newcommand{\AnnotationTok}[1]{\textcolor[rgb]{0.38,0.63,0.69}{\textbf{\textit{#1}}}}
\newcommand{\AttributeTok}[1]{\textcolor[rgb]{0.49,0.56,0.16}{#1}}
\newcommand{\BaseNTok}[1]{\textcolor[rgb]{0.25,0.63,0.44}{#1}}
\newcommand{\BuiltInTok}[1]{#1}
\newcommand{\CharTok}[1]{\textcolor[rgb]{0.25,0.44,0.63}{#1}}
\newcommand{\CommentTok}[1]{\textcolor[rgb]{0.38,0.63,0.69}{\textit{#1}}}
\newcommand{\CommentVarTok}[1]{\textcolor[rgb]{0.38,0.63,0.69}{\textbf{\textit{#1}}}}
\newcommand{\ConstantTok}[1]{\textcolor[rgb]{0.53,0.00,0.00}{#1}}
\newcommand{\ControlFlowTok}[1]{\textcolor[rgb]{0.00,0.44,0.13}{\textbf{#1}}}
\newcommand{\DataTypeTok}[1]{\textcolor[rgb]{0.56,0.13,0.00}{#1}}
\newcommand{\DecValTok}[1]{\textcolor[rgb]{0.25,0.63,0.44}{#1}}
\newcommand{\DocumentationTok}[1]{\textcolor[rgb]{0.73,0.13,0.13}{\textit{#1}}}
\newcommand{\ErrorTok}[1]{\textcolor[rgb]{1.00,0.00,0.00}{\textbf{#1}}}
\newcommand{\ExtensionTok}[1]{#1}
\newcommand{\FloatTok}[1]{\textcolor[rgb]{0.25,0.63,0.44}{#1}}
\newcommand{\FunctionTok}[1]{\textcolor[rgb]{0.02,0.16,0.49}{#1}}
\newcommand{\ImportTok}[1]{#1}
\newcommand{\InformationTok}[1]{\textcolor[rgb]{0.38,0.63,0.69}{\textbf{\textit{#1}}}}
\newcommand{\KeywordTok}[1]{\textcolor[rgb]{0.00,0.44,0.13}{\textbf{#1}}}
\newcommand{\NormalTok}[1]{#1}
\newcommand{\OperatorTok}[1]{\textcolor[rgb]{0.40,0.40,0.40}{#1}}
\newcommand{\OtherTok}[1]{\textcolor[rgb]{0.00,0.44,0.13}{#1}}
\newcommand{\PreprocessorTok}[1]{\textcolor[rgb]{0.74,0.48,0.00}{#1}}
\newcommand{\RegionMarkerTok}[1]{#1}
\newcommand{\SpecialCharTok}[1]{\textcolor[rgb]{0.25,0.44,0.63}{#1}}
\newcommand{\SpecialStringTok}[1]{\textcolor[rgb]{0.73,0.40,0.53}{#1}}
\newcommand{\StringTok}[1]{\textcolor[rgb]{0.25,0.44,0.63}{#1}}
\newcommand{\VariableTok}[1]{\textcolor[rgb]{0.10,0.09,0.49}{#1}}
\newcommand{\VerbatimStringTok}[1]{\textcolor[rgb]{0.25,0.44,0.63}{#1}}
\newcommand{\WarningTok}[1]{\textcolor[rgb]{0.38,0.63,0.69}{\textbf{\textit{#1}}}}
% end of color syntax

````

e la seguente prima pagina:

````latex
\begin{titlepage}
    \newcommand{\HRule}{\rule{\linewidth}{0.4mm}}

    \center
    \textsc{\LARGE Marjor heading}\\[0.5cm] % Major heading such as course name
    \textsc{\large Main heading}\\[0.5cm] % Main heading such as the name of your university/college
    \textsc{\large Course title}\\[1.5cm] % Minor heading such as course title
    \includegraphics[width=4cm]{logo.jpg}\\[1.5cm]

    
    \HRule\\[0.4cm]
    {\huge\bfseries Project}\\[0.4cm]   
    \HRule\\[2.5cm]

    {\large\textit{Author}}\\
    Federico \textsc{Thiella}
    
    \vfill\vfill\vfill
    
    {\large\today}
    \vfill
\end{titlepage}

````

Per inserire del codice sorgente correttamente colorato e formattato,
ho pensato di scrivere il codice in formato Markdown e di tradurlo in
LaTex con [Pandoc](https://pandoc.org/)

    pandoc codice.md -t latex

anche Pandoc sarà oggetto di un prossimo articolo!
