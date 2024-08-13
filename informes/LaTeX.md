#tech 

# ¿Qué es?

Es un lenguaje de marcado utilizado para crear documentos científicos y técnicos de alta calidad.

```bash
sudo apt install latexmk zathura texlive-full
xdg-mime query default application/pdf
xdg-mime default zathura.desktop application/pdf 
```

# Document

```latex
\documentclass[a4paper]{article} % Comentario

\usepackage[utf8]{inputenc}
\usepackage[spanish]{babel}

\usepackage[margin=2cm, top=2cm, includefoot]{geometry} % Margenes

[...]

\begin{document}
	Esto es el cuerpo del documento.
\end{document}
```

```bash
latexmk -pdf <document> -pvc
```
> Con **`-pvc`** podemos previsualizar el documento.

## Módulos

```latex
\usepackage{graphicx} % Para inserción de imágenes
\usepackage[table,xcdraw]{xcolor} % Para colores
\usepackage{fancyhdr} % Para el estilo de página
\usepackage[hidelinks]{hyperref} % Para gestión de hipervínculos
\usepackage{setpace} % Para modificar el espaciado entre lineas
\usepackage{parskip} % Elimina el tabulado al principio de un párrafo
```

## Portada

```latex
\newcomand{\logoportada}{<image>}
\newcomand{\plataforma}{<plataforma>}
\newcomand{\machineName}{<maquina>}
\newcomand{\machineLogo}{<logoMaquina>}
\newcomand{\startDate}{<fecha>}

\definecolor{tituloPortada}{HTML}{146c8a}

\begin{document}
	\begin{titlepage}
		\centering
		\includegraphics[width=0.5\textwidth]{\logoportada}\par\vspace{0.4cm}
		{\scshape\LARGE \textbf{Informe Técnico}}
		{\Huge\textcolor{tituloPortada}{\textbf{Máquina: \machineName}}}
		\vfill
		\includegraphics[width=\textwidth, keepaspectratio]{\machineLogo}
		\vfill
		{\large \startDate\par}
		\vfill
	\end{titlepage}
\end{document}
```

## Indice

```latex
\addto\captionsspanish{\renewcommand{\contentsname}{Tabla de contenidos}} % Cambiar encabezado del indice

\begin{document}

[...]

\clearpage
\tableofcontents
\clearpage
```

## Páginas

### Estilo de página

```latex
\usepackage{fancyhdr}
\usepackage{setpace} % Para modificar el espaciado entre lineas
\usepackage{ragged2e} % Para poder utilizar \justifying después de los \centering

\setstretch{1.2} % Para asignar el espaciado entre lineas
\usepackage{parskip} % Elimina el tabulado al principio de un párrafo
\setlength{\parindent}{0pt}
\setlength{parskip}{0.8em plus 0.5em minus 0.2em} % Establece el espacio entre párrafos en 1em (espacio de fuente) más 0.5em de espacio extra permitido, con una reducción máxima de 0.2em
\setlength{\parfillskip}{\parindent plus 1fill}
[...]

\setlength{\headheight}{40.2pt}
\pagestyle{fancy}
\fancyhf
% Logo en la esquina superior izquierda y derecha
\lhead{\includegraphics[height=1cm]{<logo>}}\rhead{\includegraphics[height=1cm]{<logo>}} 
\renewcommand{\headrulewidth}{3pt} % Aumenta el tamaño de la barra superior
\renewcommand{\headrule}{\hbox to\headwidth{\color{<color>}\leaders\hrule height \headrulewidth\hfill}} % Cambiar color de la barra superior

[...]

\begin{document}
	\cfoot{\thepage} % Para ver la numeración en las páginas
```

### Secciones

```latex
\section{Antecedentes}
El presente documento recoge los resultados de la auditoría realizada sobre la máquina \textbf{\machineName}, enumerando los vectores de ataque encontrados, así como la explotación realizada sobre cada uno de ellos.

Esta máquina ha sido descargada de la plataforma \href{<enlace>}{\textbf{\color{\tituloPortada} \plataforma}}. A continuación se proporciona el enlace de descarga de la máquina: \href{<enlace>}{\textbf{\color{\tituloPortada} \machineName}}

\vspace{0.2cm}

\begin{tcolorbox}[enhanced,attach boxed title to top center={yshift=-3mm,yshifttext=-1mm},
	colback=blue!5!white,colframe=blue!75!black,colbacktitle=tituloPortada!80!black,
	title=Dirección URL,fonttitle=\bfseries,
	boxed title style={size=small,colframe=red!50!black} ]
	\centering
	\href{<enlace>}{\textbf{\color{\tituloPortada} <enlace>}}
\end{tcolorbox}
```


## Elementos
### Figures

```latex
\usepackage[figurename=Imagen]{caption} % Para cambiar el nombre de las figures

[...]

\begin{figure}[h] % La h es para que no salte a una nueva página
	\centering
	\setlength{\fboxrule}{0.8pt}
	\fbox{\includegraphics[width=\textwidth]{<Imagen>}} % Añade un marco
	\caption{Descripcion de la imagen}
\end{figure}
```

### Cuadros de texto

Podemos hacerlo con *tcolorbox*, un ejemplo:

```latex
\usepackage{tikz,lipsum,lmodern}
\usepackage[most]{tcolorbox}


\begin{tcolorbox}[colback=red!5!white,colframe=red!75!black]
  My box.
\end{tcolorbox}
```

### Tablas

```latex
\begin{tabular}{ c | c }

	\textbf{Tecnología} & \textbf{Versión} \\
	\hline
	PHP & 5.5.38 \\
	Apache & 2.4.6 \\

\end{tabular}
```

### Código

```latex
\usepackage{listings}
\usepackage{xcolor}

\definecolor{codegreen}{rgb}{0,0.6,0}
\definecolor{codegray}{rgb}{0.5,0.5,0.5}
\definecolor{codepurple}{rgb}{0.58,0,0.82}
\definecolor{backcolour}{rgb}{0.95,0.95,0.92}

\lstdefinestyle{mystyle}{
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

\lstset{style=mystyle}
```