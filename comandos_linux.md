# Guía de Comandos Básicos de Linux y Aplicación en Bioinformática

¡Holis querida amiga Andrea! Esta guía está diseñada para ayudarte a visualizar rápidamente los comandos esenciales de la terminal de Linux, tanto los generales como los específicos que utilizaste en tus prácticas de bioinformática guiadas por la profesora Doris Vela.

---

### Comandos Básicos de Linux (Referencia General)

Aquí tienes una lista de comandos cotidianos para moverte y gestionar archivos en el sistema:

- **pwd**: Muestra la ruta completa del directorio actual en el que te encuentras (Print Working Directory).
- **ls**: Lista los archivos y carpetas que están dentro del directorio actual (ej. `ls -lh` para ver tamaños de archivos en formato legible).
- **cd**: Permite cambiar de directorio o carpeta (ej. `cd Documentos/`).
- **mkdir**: Crea una nueva carpeta (ej. `mkdir carpeta_nueva`). Usando el modificador `-p` (`mkdir -p`), puedes crear varias carpetas anidadas o múltiples a la vez.
- **cp**: Copia archivos o carpetas de un lugar a otro.
- **mv**: Mueve o renombra archivos y carpetas.
- **rm**: Elimina o borra archivos.
- **cat**: Muestra el contenido completo de un archivo de texto en la pantalla o permite concatenar varios archivos.
- **grep**: Busca un patrón de texto específico dentro de archivos o flujos de datos (ej. `grep -c '^>'` para contar cabeceras FASTA).
- **head**: Muestra únicamente las primeras líneas de un archivo (por defecto las primeras 10).
- **time**: Mide el tiempo exacto que tarda en ejecutarse un comando o proceso en la terminal.
- **echo**: Imprime texto en pantalla o escribe información dentro de un flujo o archivo.
- **unzip**: Descomprime archivos en formato `.zip` (utilizado para reportes de FastQC).

---

### Instalación de Herramientas y Configuración Inicial

- **sudo apt update**: Actualiza la lista de paquetes disponibles en los repositorios del sistema operativo Ubuntu.
- **sudo apt install**: Instala un programa o herramienta en el sistema (empleado para paquetes como `ncbi-blast+`, `ncbi-entrez-direct`, `mafft`, `iqtree`, `fastqc`, `fastp`, etc.).
- **sh -c**: Ejecuta una cadena de comandos como un script de intérprete de shell (usado para la instalación manual de Entrez Direct mediante script remoto).
- **curl**: Herramienta para transferir datos desde o hacia un servidor utilizando protocolos como HTTP o HTTPS.
- **export PATH**: Añade una ruta específica a la variable de entorno PATH del sistema para que los programas instalados manualmente queden accesibles desde cualquier ubicación.

---

### Análisis de Calidad de Secuencias (FastQC y FastP)

Comandos para la evaluación, control y filtrado de calidad de lecturas crudas Illumina paired-end:

- **fastqc**: Genera un reporte detallado en formato HTML y ZIP sobre métricas de calidad de secuencias FASTQ (ej. `fastqc raw/archivo_1.fastq.gz -o fastqc_raw/`).
- **seqkit head**: Permite extraer un subconjunto rápido de un archivo FASTQ conservando el orden y emparejamiento (ej. `seqkit head -n 100000 archivo_1.fastq.gz -o sub_1.fastq.gz`).
- **fastp**: Herramienta rápida que combina el recorte de adaptadores por solapamiento en pares, filtrado por calidad Phred y control de longitud en un solo paso. Parámetros principales utilizados:
  - `-i` / `-I`: Archivos de entrada R1 y R2.
  - `-o` / `-O`: Archivos de salida limpios (trimmed).
  - `--detect_adapter_for_pe`: Detección automática de adaptadores para lecturas paired-end.
  - `--qualified_quality_phred 20`: Umbral mínimo de calidad Phred aceptable.
  - `--length_required 36`: Longitud mínima de lectura requerida tras el recorte.

---

### BLAST Local con MEGABLAST

Comandos específicos para la construcción de bases de datos locales y búsquedas comparativas de secuencias:

- **declare -A MITO**: Declara una matriz o arreglo asociativo en Bash para almacenar pares de nombres de especies y sus códigos de accesión del NCBI.
- **for ... do ... done**: Estructura de control iterativa (en bucle) para automatizar la descarga y procesamiento masivo de archivos FASTA.
- **efetch**: Herramienta de Entrez Direct para recuperar registros biológicos desde las bases de datos del NCBI (ej. `-format fasta_cds_na` para secuencias codificantes).
- **awk**: Un potente lenguaje de procesamiento de patrones y escaneo de texto (utilizado para aislar la secuencia del gen COX1 mediante el delimitador de cabeceras `>`).
- **sed**: Editor de flujos utilizado para modificar texto de forma automatizada (en la práctica se usó con la opción `-i` para renombrar las cabeceras FASTA con el nombre de la especie).
- **makeblastdb**: Programa encargado de convertir un archivo FASTA en una base de datos indexada y optimizada para búsquedas locales de BLAST (usando los parámetros `-in`, `-dbtype nucl`, `-parse_seqids` y `-out`).
- **blastn**: Programa principal para alinear secuencias de nucleótidos, especificando tareas de búsqueda con el parámetro `-task` (`megablast`, `dc-megablast`, o `blastn` clásico), archivos de consulta (`-query`), bases de datos (`-db`), y formatos de salida tabulares (`-outfmt`).
- **grep -v '^#'**: Filtra las líneas que empiezan con `#` (comentarios y encabezados de la tabla de resultados de BLAST) para mostrar únicamente los datos de alineamiento.

---

### Alineamiento Múltiple de Secuencias con MAFFT

Comandos orientados a alinear homologías estructurales y evolutivas entre secuencias homólogas:

- **mafft --auto**: Ejecuta el programa MAFFT seleccionando automáticamente la estrategia de alineamiento óptima según las características del conjunto de datos.
- **mafft --auto --clustalout**: Genera el alineamiento múltiple guardándolo en formato Clustal, el cual incluye una línea inferior de conservación (`*, :, .`) para inspección rápida en texto.
- **less**: Permite visualizar archivos largos de texto de manera interactiva y desplazarse verticalmente dentro de la terminal.

---

### Inferencia Filogenética con IQ-TREE

Herramientas estadísticas de Máxima Verosimilitud para calcular relaciones evolutivas y construir árboles filogenéticos:

- **iqtree2**: Programa principal para la inferencia de árboles filogenéticos a partir de alineamientos múltiples.
- `-s`: Especifica el archivo de entrada con el alineamiento en formato FASTA (ej. `-s COX1_alineado.fasta`).
- `-m MFP`: Selecciona automáticamente el mejor modelo de sustitución nucleotídica (ModelFinder Plus) adecuado para los datos.
- `-bb 1000`: Ejecuta 1000 réplicas de Bootstrap Ultrafast (`UFBoot`) para evaluar estadísticamente el soporte de cada rama.
- `-nt AUTO`: Detecta y utiliza automáticamente todos los núcleos disponibles del procesador para acelerar el procesamiento.
- `-pre`: Define un prefijo para organizar todos los archivos de salida generados (como el archivo con extensión `.treefile` en formato Newick).

---

### Edición de Archivos con Nano

- **nano**: Editor de texto interactivo dentro de la terminal. 
  - Para guardar los cambios realizados, se presiona `Ctrl + o` seguido de la tecla `Enter`.
  - Para cerrar el editor después de guardar, se presiona `Ctrl + x`.
