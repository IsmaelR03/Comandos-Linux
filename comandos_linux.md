# Guía de Comandos Básicos de Linux y Aplicación en Bioinformática

¡Holis querida amiga Andrea! Esta guía está diseñada para ayudarte a visualizar rápidamente los comandos esenciales de la terminal de Linux, tanto los generales como los específicos que utilizaste en tu práctica de BLAST local con secuencias mitocondriales.

###. Comandos Básicos de Linux (Referencia General)

Aquí tienes una lista de comandos cotidianos para moverte y gestionar archivos en el sistema:

* pwd: Muestra la ruta completa del directorio actual en el que te encuentras (Print Working Directory).
* ls: Lista los archivos y carpetas que están dentro del directorio actual.
* cd: Permite cambiar de directorio o carpeta (ej. `cd Documentos/`).
* mkdir: Crea una nueva carpeta (ej. `mkdir carpeta_nueva`). Usando el modificador `-p` (`mkdir -p`), puedes crear varias carpetas anidadas o múltiples a la vez.
* cp: Copia archivos o carpetas de un lugar a otro.
* mv: Mueve o renombra archivos y carpetas.
* rm: Elimina o borra archivos.
* cat: Muestra el contenido completo de un archivo de texto en la pantalla o permite concatenar varios archivos.
* grep: Busca un patrón de texto específico dentro de archivos o flujos de datos.
* head: Muestra únicamente las primeras líneas de un archivo (por defecto las primeras 10).
* time: Mide el tiempo exacto que tarda en ejecutarse un comando o proceso en la terminal.

---

###. Comandos Utilizados en la Práctica de BLAST Local

Estos son los comandos específicos que empleaste paso a paso durante tu práctica con la profesora Doris Vela:

### Instalación de Herramientas
* sudo apt update: Actualiza la lista de paquetes disponibles en los repositorios del sistema operativo Ubuntu.
* sudo apt install: Instala un programa o herramienta en el sistema (en la práctica se usó para instalar `ncbi-blast+` y `curl`).

### Descarga y Configuración del Entrez Direct
* sh -c: Ejecuta una cadena de comandos como un script de intérprete de shell.
* curl: Herramienta para transferir datos desde o hacia un servidor utilizando protocolos como HTTP o HTTPS (utilizada aquí para descargar el instalador de EDirect).
* export PATH: Añade una ruta específica a la variable de entorno PATH del sistema para que los programas puedan ejecutarse desde cualquier ubicación en la terminal.
* echo: Imprime texto en pantalla o escribe información dentro de un archivo.

### Verificación de Versiones
* blastn -version: Muestra la versión instalada de la herramienta BLAST para nucleótidos.
* makeblastdb -version: Comprueba la versión instalada del creador de bases de datos de BLAST.
* efetch: Herramienta de Entrez Direct para recuperar registros biológicos desde las bases de datos del NCBI (se combinó con `-help | head -n 3` para ver la ayuda inicial).

### Manipulación de Archivos y Directorios
* declare -A MITO: Declara una matriz o arreglo asociativo en Bash para almacenar pares de nombres de especies y sus códigos de accesión del NCBI.
* for ... do ... done: Estructura de control iterativa (en bucle) para repetir un conjunto de instrucciones para varios elementos.
* awk: Un potente lenguaje de procesamiento de patrones y escaneo de texto (utilizado para aislar la secuencia del gen COX1 mediante el delimitador de cabeceras `>`).
* sed: Editor de flujos utilizado para modificar texto de forma automatizada (en la práctica se usó con la opción `-i` para reemplazar o editar las cabeceras de los archivos FASTA).
* wc -l o grep -c '^>': Cuenta la cantidad de líneas que coinciden con un patrón (empleado para verificar que el multi-FASTA final contuviera exactamente las 10 especies).

### Construcción de Base de Datos y Búsquedas BLAST
* makeblastdb: Programa encargado de convertir un archivo FASTA en una base de datos indexada y optimizada para búsquedas locales de BLAST (usando los parámetros `-in`, `-dbtype nucl`, `-parse_seqids` y `-out`).
* blastn: Programa principal para alinear secuencias de nucleótidos contra una base de datos de nucleótidos, especificando tareas (`-task megablast`, `dc-megablast`, o `blastn`), archivos de consulta (`-query`), bases de datos (`-db`), formatos de salida (`-outfmt`) y archivos de resultado (`-out`).
* grep -v '^#': Filtra las líneas que empiezan con `#` (comentarios y encabezados de la tabla de resultados) para mostrar únicamente las filas con los datos de alineamiento.

### Cerrar y abrir nano
* nano: Editor de un archivo dentro de la terminal, para cerrarlo se presiona la tecla `Ctrl + o` luego `Enter`, de esta forma se guarda los cambios realizados dentro de nano.
Para cerrar el editor luego de guardarlo se presiona `Ctrl + x`.

###. Alineamiento Múltiple de Secuencias con MAFFT

Comandos orientados a alinear homologías estructurales y evolutivas entre las distintas especies:

* mafft --auto: Ejecuta el programa MAFFT seleccionando automáticamente la estrategia de alineamiento más adecuada según el tamaño y características del conjunto de datos.
* head -n 2: Muestra las primeras 2 líneas de un archivo, útil para una inspección rápida de las cabeceras y secuencias del archivo alineado resultante.
* mafft --auto --clustalout: Genera el alineamiento múltiple guardándolo en formato Clustal, el cual incluye una línea inferior de asteriscos y dos puntos (`*, :, .`) que facilitan la lectura visual de la conservación de nucleótidos directamente en la terminal.
* less: Permite visualizar archivos de texto largos de manera interactiva, permitiendo desplazarse hacia arriba y hacia abajo mediante las teclas de dirección.

###. Inferencia Filogenética con IQ-TREE

Herramientas para calcular las relaciones evolutivas y construir árboles filogenéticos estadísticos:

* iqtree2 (o variantes como `./iqtree3.exe`): Programa principal de Máxima Verosimilitud para la inferencia de árboles filogenéticos a partir de secuencias alineadas.
* -s: Especifica el archivo de entrada con el alineamiento múltiple en formato FASTA (ej. `-s COX1_alineado.fasta`).
* -m MFP`: Selecciona automáticamente el mejor modelo de sustitución nucleotídica (Model Finder Plus) que se ajusta a los datos.
* -bb 1000: Ejecuta 1000 réplicas de Bootstrap Ultrafast (`UFBoot`) para evaluar estadísticamente el soporte y confiabilidad de cada rama del árbol filogenético.
* -nt AUTO: Permite que IQ-TREE detecte y utilice de manera automática todos los núcleos disponibles del procesador para acelerar el cálculo.
* -pre: Define un prefijo para nombrar y organizar todos los archivos de salida generados por el análisis (ej. guardándolos en la carpeta de resultados).
* cat COX1_tree.treefile: Muestra en la terminal el árbol filogenético resultante codificado en formato estándar Newick (parentescos anidados).
