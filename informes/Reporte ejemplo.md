```latex
\documentclass[a4paper]{article} % Comentario

\usepackage[utf8]{inputenc}
\usepackage[spanish]{babel}

\usepackage[margin=2cm, top=2cm, includefoot]{geometry} % Margenes

\usepackage{graphicx} % Para inserción de imágenes
\usepackage[table,xcdraw]{xcolor} % Para colores
\usepackage{fancyhdr} % Para el estilo de página
\usepackage[hidelinks]{hyperref} % Para gestión de hipervínculos
\usepackage{setpace} % Para modificar el espaciado entre lineas
\usepackage{parskip} % Elimina el tabulado al principio de un párrafo
\usepackage{listings} % Para los bloques de código

\setstretch{1.2} % Para asignar el espaciado entre lineas
\setlength{\parindent}{0pt}
\setlength{parskip}{0.8em plus 0.5em minus 0.2em} % Establece el espacio entre párrafos en 1em (espacio de fuente) más 0.5em de espacio extra permitido, con una reducción máxima de 0.2em
\setlength{\parfillskip}{\parindent plus 1fill}

\setlength{\headheight}{40.2pt}
\pagestyle{fancy}
\fancyhf
% Logo en la esquina superior izquierda y derecha
\lhead{\includegraphics[height=1cm]{<logo>}}\rhead{\includegraphics[height=1cm]{<logo>}} 
\renewcommand{\headrulewidth}{3pt} % Aumenta el tamaño de la barra superior
\renewcommand{\headrule}{\hbox to\headwidth{\color{<color>}\leaders\hrule height \headrulewidth\hfill}} % Cambiar color de la barra superior

% Para los bloques de código
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
\renewcommand{\lstlistingname}{Código}

\newcomand{\logoportada}{<image>}
\newcomand{\plataforma}{<plataforma>}
\newcomand{\machineName}{<maquina>}
\newcomand{\machineLogo}{<logoMaquina>}
\newcomand{\startDate}{<fecha>}

\definecolor{tituloPortada}{HTML}{146c8a}

\addto\captionsspanish{\renewcommand{\contentsname}{Tabla de contenidos}} % Cambiar encabezado del indice

\begin{document}

	\cfoot{\thepage} % Para ver la numeración en las páginas

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

	% INDICE ========================================================================
	\clearpage
	\tableofcontents
	\clearpage

	% ANTECEDENTES ========================================================================

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

	\begin{figure}[h] % La h es para que no salte a una nueva página
		\centering
		\setlength{\fboxrule}{0.8pt}
		\fbox{\includegraphics[width=\textwidth]{<Imagen>}} % Añade un marco
		\caption{Página principal del servicio web de la máquina.}
	\end{figure}

	% OBJETIVOS ========================================================================

	\section{Objetivos}
	Los objetivos de la presente auditoría de seguridad se enfocan en la identificación de posibles vulnerabilidades y debilidades en la máquina \textbf{\color{tituloPortada} \machineName}, con el propósito de garantizar la integridad y confidencialidad de la información almacenada en ella.

	Con este fin se ha llevado a cabo un análisis exhaustivo de todos los servicios que se encontraban expuestos en dicho servidor, recopilando información detallada de aquellos que representan un riesgo potencial desde el punto de vista de la seguridad.
	
	\subsection{Alcance}
	A continuación se representan los objetivos a cumplir para esta auditoría:

	\begin{itemize}
		\item Identificar los puertos y servicios vulnerables.
		\item Realizar una explotación de las vulnerabilidades encontradas.
		\item Conseguir acceso al servidor mediante la explotación de los servicios vulnerables encontrados.
		\item Enumerar vías potenciales de elevar privilegios en el sistema una vez este ha sido vulnerado.
	\end{itemize}

	\subsection{Impedimentos y limitaciones}

	Durante el proceso de auditoría, está terminantemente prohibido realizar alguna de las siguientes actividades:

	\begin{itemize}
		\item Realizar tareas que puedan ocasionar una \textbf{denegación de servicios} ó afectar a la disponibilidad de los servicios expuestos.
		\item Borrar archivos residentes en el servidor una vez este haya sido vulnerado.
	\end{itemize}

	\subsection{Resumen general}

	\clearpage

	% RECONOCIMIENTO ========================================================================
	\section{Reconocimento}
	\subsection{Enumeración de servicios expuestos}
	
	A continuación se presenta una evidencia de los puertos y servicios identificados durante el reconocimiento aplicado con la herramienta \textbf{nmap}:

	\begin{figure}[h] % La h es para que no salte a una nueva página
		\centering
		\setlength{\fboxrule}{0.8pt}
		\fbox{\includegraphics[width=\textwidth]{<Imagen>}}
		\caption{Enumeración de puertos con nmap.}
	\end{figure}

	\vspace{0.3cm}

	Asimismo no se encontraron servicios corriendo bajo otro protocolo, por lo que se priorizará la evaluación de los puertos identificados en el primer escaneo efectuado.

	\clearpage
	\subsection{Enumeración de servidores web}
	A continuación se representan los resultados obtenidos con la herramienta \textbf{WhatWeb}, una herramienta de reconocimiento web que se utiliza para identificar tecnologías web específicas que están siendo empleadas en un sitio web. Tras aplicar un reconocimiento sobre el servicio HTTP que corre sobre el puerto 80:

	\begin{figure}[h] % La h es para que no salte a una nueva página
		\centering
		\setlength{\fboxrule}{0.8pt}
		\fbox{\includegraphics[width=\textwidth]{<Imagen>}}
		\caption{Enumeración del servicio HTTP.}
	\end{figure}

	En los resultados obtenidos es posible identificar las versiones para algunas de las tecnologías existentes:

	\vspace{0.4cm}
	\centering
	\begin{tabular}{ c | c }
		\textbf{Tecnología} & \textbf{Versión} \\
		\hline
		PHP & 5.5.38 \\
		Apache & 2.4.6 \\
	\end{tabular}
	\vspace{0.4cm}

	\justifying

	Dentro de la información representada, también es posible identificar 2 correos electrónicos, los cuáles podrían ser utilizados de cara a un ataque de \textbf{Phishing}:

	\vspace{0.3cm}
	\begin{center}
		\texttt{contact@example.com} \qquad \texttt{contact@votenow.local}
	\end{center}
	\vspace{0.3cm}
	
	El \textbf{Phishing} es ...

	\subsection{Enumeración de subdominios}
	Una vez identificado el dominio '\textbf{votenow.local}', se procedió a realizar un ataque de fuerza bruta para enumerar subdominios válidos.

	Al finalizar el ataque éstos fueron los resultados obtenidos:

	\begin{figure}[h]
		\centering
		\setlength{\fboxrule}{0.8pt}
		\makebox[\textwidth]{
			\fbox{\includegraphics[width=0.9\paperwidth]{<Imagen>}}
		}
		\caption{Enumeración de subdominios.}
		\label{fig:identifiedSubdomains}
	\end{figure}

	Se identificó el subdominio \textbf{datasafe.votenow.local}. Este subdominio representó un punto crucial en la auditoría dado que ...

	Cabe destacar que para que estos subdominios fueran accesibles, fue necesario incorporar la siguiente información en el archivo \textbf{/etc/hosts}:

	\begin{figure}[h]
		\centering
		\setlength{\fboxrule}{0.8pt}
		\fbox{\includegraphics[width=0.9\paperwidth]{<Imagen>}}
		\caption{Archivo \textbf{/etc/hosts}.}
		\label{fig:etcHosts}
	\end{figure}
	\vspace{0.3cm}

	Esto es así dado que se está aplicando \textbf{virtual hosting}, una técnica ...

	\clearpage
	\subsection{Enumeración de paneles de autenticación}

	Una vez descubierto el subdominio \textbf{datasafe.votenow.local}, representado en la imagen \ref{fig:identifiedSubdomains} de la página \pageref{fig:identifiedSubdomains}, se encontró el siguiente panel de \textbf{PHPMyAdmin}:

	\begin{figure}[h]
		\centering
		\setlength{\fboxrule}{0.8pt}
		\fbox{\includegraphics[width=0.9\paperwidth]{<Imagen>}}
		\caption{Archivo \textbf{/etc/hosts}.}
		\label{fig:phpmyadmin}
	\end{figure}
	\vspace{0.3cm}

	% EXPLOTACION ========================================================================
	\section{Identificación y explotación de vulnerabilidades}
	\subsection{Archivo de backup expuesto}

	Durante un reconocimiento con la herramienta \textbf{gobuster}, <descripción>, se identificó un archivo de backup expuesto en el servidor:

	 \begin{figure}[h]
		\centering
		\setlength{\fboxrule}{0.8pt}
		\fbox{\includegraphics[width=0.9\paperwidth]{<Imagen>}}
		\caption{Archivo \textbf{Archivo de backup expuesto}.}
		\label{fig:backupFile}
	\end{figure}
	\vspace{0.3cm}

	El script utilizado para la explotación es:

	\begin{lstlisting}[language=Python, caption=Exploit para phpmyadmin]
	<Codigp>
	\end{lstlisting}

	\section{Escalada de privilegios}
	
	\section{Contramedidas y buenas prácticas}

	\section{Conclusiones}

\end{document}
```