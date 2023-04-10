# docs: linux/latex

## Examples

```latex
\documentclass{article}

\usepackage[utf8]{inputenc}
\usepackage{listings}
\usepackage{xcolor}

\definecolor{codegreen}{rgb}{0,0.6,0}
\definecolor{codegray}{rgb}{0.5,0.5,0.5}
\definecolor{codepurple}{rgb}{0.58,0,0.82}
\definecolor{backcolour}{rgb}{0.95,0.95,0.92}

\lstdefinestyle{code}{
    backgroundcolor=\color{backcolour},   
    commentstyle=\color{codegreen},
    keywordstyle=\color{magenta},
    numberstyle=\tiny\color{codegray},
    stringstyle=\color{codepurple},
    basicstyle=\ttfamily\footnotesize,
    breakatwhitespace=false,         
    breaklines=true,                 
    captionpos=b,                    
    keepspaces=true,                 
    numbers=left,                    
    numbersep=5pt,                  
    showspaces=false,                
    showstringspaces=false,
    showtabs=false,                  
    tabsize=2
}

\lstset{style=code}

\begin{document}

\title{Exercise Sheet 1}
\author{Jan Fuhrer}
\maketitle

\thispagestyle{empty}
\tableofcontents

\clearpage
\setcounter{page}{1}

\section{Section}

\subsection{Subsection}

\begin{itemize}
    \item Item1
\end{itemize}

\begin{lstlisting}
text
\end{lstlisting}    

\section{Implementation in Go}
\subsection{Go}
\lstinputlisting[language=go]{main.go}

\end{document}
```