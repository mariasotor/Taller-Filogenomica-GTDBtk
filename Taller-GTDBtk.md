# Taller Filogenómica - GTDBtk

### Overview de los flujos de trabajo classify_wf y de_novo_wf de GTDB-Tk
GTDB-Tk es una herramienta diseñada para clasificar genomas bacterianos y arqueales mediante la taxonomía estandarizada del Genome Taxonomy Database (GTDB). En este taller utilizaremos la release r226 de GTDB y trabajaremos con dos flujos de trabajo principales de GTDB-Tk: `classify_wf`, orientado a la clasificación taxonómica de MAGs con base en el árbol de referencia de GTDB, y `de_novo_wf`, destinado a generar un árbol filogenético personalizado que integre tanto MAGs propios como genomas de referencia.

#### classify_wf: clasificación taxonómica basada en genes marcadores
La clasificación de un genoma de interés se determina combinando su ubicación en el árbol de referencia de GTDB, su Divergencia Evolutiva Relativa (RED) y su Identidad Nucleotídica Promedio (ANI) con genomas de referencia. En la mayoría de los casos, la topología del árbol define la asignación taxonómica, pero el valor RED resulta clave para resolver ambigüedades en los niveles taxonómicos.

Este flujo asigna una taxonomía a cada MAG utilizando un conjunto de 120 genes marcadores de copia única, un enfoque ideal para MAGs incompletos o fragmentados. El proceso incluye:

1. Predicción de genes con Prodigal.
2. Identificación de genes marcadores mediante HMMs de HMMER.
3. Alineamiento de cada marcador con su modelo (HMM) correspondiente.
4. Concatenación de los alineamientos para generar una MSA de ~5.000 posiciones.
5. Ubicación del MAG en el árbol de referencia de GTDB y asignación de taxonomía usando topología, RED y ANI.

El resultado es una clasificación taxonómica estandarizada según la release de GTDB utilizada.

#### de_novo_wf: construcción de un árbol filogenético *de novo*
Este flujo emplea el mismo conjunto de 120 genes marcadores, pero en lugar de ubicar los MAGs en el árbol de referencia de GTDB, genera un árbol filogenético independiente construido exclusivamente a partir de los MAGs del estudio. En la etapa de inferencia filogenética, GTDB-Tk utiliza FastTree junto con el modelo WAG+GAMMA para estimar árboles bacterianos y arqueales *de novo* por separado, optimizando la estimación de relaciones evolutivas entre los genomas incluidos. Una vez inferido el árbol, este puede ser enraizado utilizando un outgroup definido por el usuario.

### Configuracion del espacio de trabajo
Cree una nueva carpeta en su espacio de trabajo llamada `taller-gtdbtk`. Dentro de esta carpeta, cree los siguientes subdirectorio:

📂 `taller-gtdbtk`/ <br>
├── 📁 `gtdbtk_classify`/ <br>
├── 📁 `gtdbtk_tree`

### Generar el batchfile
Un batch file es un archivo de texto con dos columnas que contiene las rutas de los MAGs junto con sus identificadores correspondientes (ID del MAG), lo que facilita el procesamiento de múltiples archivos de manera estructurada. Este archivo se utilizará como entrada para los analisis con GTDB-Tk.

Los MAGs que se utilizarán durante este taller están ubicados en:
`/hpcfs/home/cursos/bcom4101/Filogenomica2025/alejandra_soto/taller-GTBtk/mags`

Para automatizar la creación del batchfile, dispone del script `generate_batchfile.sh`, el cual se encuentra en:
`/hpcfs/home/cursos/bcom4101/Filogenomica2025/alejandra_soto/taller-GTBtk/`

Copie este script a su directorio `taller-gtdbtk` y ejecútelo usando: 
`bash generate_batchfile.sh`

Esto generará un archivo `batchfile.txt` en el directorio `taller-gtdbtk`, listo para los análisis posteriores.

### Crear y ejecutar el script en Bash para correr GTDB-Tk (classify_wf y de_novo_wf)

Cree dentro de su carpeta `taller-gtdbtk` un script en Bash llamado `run_gtdbtk_classify.sh`, copie en él el código mostrado a continuación y actualice la variable `batchfile` con la ruta correcta al archivo `batchfile.txt` generado previamente. Este script enviará un trabajo a SLURM para procesar cada MAG listado en el batchfile y generará los resultados de la clasificación taxonómica en la carpeta `gtdbtk_classify`.

```bash
#!/bin/bash

#SBATCH -J gtdbtk_classify
#SBATCH -D .
#SBATCH -e gtdbtk_classify_%j.err
#SBATCH -o gtdbtk_classify_%j.out
#SBATCH --cpus-per-task=8
#SBATCH --time=4:00:00	
#SBATCH --mem=100000

source /hpcfs/apps/conda4.12.0/bin/activate
conda activate gtdbtk-2.5.2

batchfile="/path/to/batchfile.txt"

gtdbtk classify_wf --batchfile ${batchfile} -x fasta --skip_ani_screen --cpus 8 --out_dir gtdbtk_classify
 
```
**Nota:** Se utiliza la opción `--skip_ani_screen`. Según los desarrolladores de GTDB-Tk, los resultados son prácticamente idénticos con o sin esta opción, con diferencias que afectan a menos del 0.1% de los genomas. Dado que el filtrado por ANI requiere recursos computacionales adicionales y no ofrece una ventaja significativa en la mayoría de los casos, se omite.

Después de crear y guardar el script, debe otorgarle permisos de ejecución y enviarlo al clúster:

```
chmod +x run_gtdbtk_classify.sh
sbatch run_gtdbtk_classify.sh
```

Ahora cree un script en Bash llamado `run_gtdbtk_tree.sh` tambien dentro de su carpeta `taller-gtdbtk`, copie en él el código mostrado a continuación y actualice la variable batchfile con la ruta correcta al archivo batchfile.txt generado previamente. Este script enviará un trabajo a SLURM para generar un árbol filogenético *de novo* con los MAGs incluidos en el batchfile, enraizado usando p__Chloroflexota. Todos los archivos generados se guardaran en la carpeta `gtdbtk_tree`.

```bash
#!/bin/bash

#SBATCH -J gtdbtk_tree
#SBATCH -D .
#SBATCH -e gtdbtk_tree_%j.err
#SBATCH -o gtdbtk_tree_%j.out
#SBATCH --cpus-per-task=8
#SBATCH --time=10:00:00	
#SBATCH --mem=100000

source /hpcfs/apps/conda4.12.0/bin/activate
conda activate gtdbtk-2.5.2

batchfile="/path/to/batchfile.txt"

gtdbtk de_novo_wf --batchfile ${batchfile} --bacteria --outgroup_taxon p__Chloroflexota --out_dir gtdbtk_tree -x fasta --cpus 8
 
```

Después de crear y guardar el script, debe otorgarle permisos de ejecución y enviarlo al clúster:

```
chmod +x run_gtdbtk_tree.sh
sbatch run_gtdbtk_tree.sh
```

### Descripción del output

#### classify_wf
Una vez que la clasificación de MAGs haya finalizado, podrá visualizar la siguiente estructura dentro de la carpeta `gtdbtk_classify`:

📂 `gtdbtk_classify`/ <br>
│── 📂 `align`/ <br>
│── 📂 `classify`/ <br>
│── 📂 `identify`/  <br>
│── 📄 `gtdbtk.ar53.summary.tsv` <br>
│── 📄 `gtdbtk.bac120.summary.tsv` <br>
│── 📄 `gtdbtk.log` <br>
│── 📄 `gtdbtk.warnings.log`

- `align` – Contiene los alineamientos de secuencias múltiples de los genes marcadores utilizados para la clasificación taxonómica.
- `classify` – Incluye los archivos que describen la ubicación de los genomas en el árbol de referencia de GTDB y sus asignaciones taxonómicas.
- `identify` – Contiene los resultados de identificación de genes marcadores, donde GTDB-Tk detecta los genes conservados y evalúa la calidad del genoma antes de clasificarlo.
- `gtdbtk.ar53.summary.tsv` – Archivo con las clasificaciones taxonómicas de los genomas arqueales, basadas en la base de datos arqueal de GTDB (ar53).
- `gtdbtk.bac120.summary.tsv` – Archivo con las clasificaciones taxonómicas de los genomas bacterianos, basadas en la base de datos bacteriana de GTDB (bac120).
- `gtdbtk.log` y `gtdbtk.warnings.log` – Archivos de log que documentan la ejecución de GTDB-Tk, incluyendo información del proceso y advertencias o problemas encontrados.

#### de_novo_wf
Una vez que este proceso haya finalizado, podrá visualizar la siguiente estructura dentro de la carpeta `gtdbtk_tree`:

📂 `gtdbtk_tree`/ <br>
│── 📂 `align`/ <br>
│── 📂 `identify`/ <br>
│── 📂 `infer`/  <br>
│── 📄 `gtdbtk.bac120.decorated.tree` <br>
│── 📄 `gtdbtk.bac120.decorated.tree-table` <br>
│── 📄 `gtdbtk.log` <br>
│── 📄 `gtdbtk.warnings.log`

- `align` – Contiene los alineamientos múltiples (MSA) de los genes marcadores utilizados para la construcción del árbol *de novo*. Estos archivos pueden emplearse como entrada para programas de inferencia filogenética externos, como IQ-TREE, en caso de que desee realizar tu propio análisis filogenético y no utilizar directamente el árbol generado por GTDB-Tk.
- `identify` – Incluye los resultados de la etapa de identificación de genes marcadores, donde GTDB-Tk detecta los genes correspondientes y evalúa la calidad de los MAGs antes de la inferencia del árbol.
- `infer` – Contiene los archivos generados durante la fase de inferencia filogenética, en la que GTDB-Tk utiliza FastTree (con el modelo WAG+GAMMA) para construir el árbol de novo a partir de los MAGs analizados.
- `gtdbtk.bac120.decorated.tree` – Árbol filogenético de novo para los MAGs bacterianos analizados, decorado con la taxonomía GTDB. Este es el archivo principal del análisis *de novo*: puede descargarse para visualizarlo en programas como FigTree, iTOL, R (phytools, ggtree) o seguir procesándose según las necesidades del usuario.
- `gtdbtk.bac120.decorated.tree-table` – Tabla asociada al árbol anterior que resume la asignación taxonómica.

### Procesamiento del archivo gtdbtk.bac120.decorated.tree
El archivo `gtdbtk.bac120.decorated.tree` contiene el árbol filogenético generado por el flujo `de_novo_wf` de GTDB-Tk, decorado con la taxonomía GTDB. Este árbol está en formato Newick, pero incluye múltiples caracteres `;` que algunos programas de visualización filogenética interpretan incorrectamente como el final del árbol y genera errores de lectura.

Para evitar errores al cargar el archivo en programas como FigTree o iTOL, es necesario realizar una corrección previa:

Abrir el archivo `.tree` en un editor de texto y reemplazar todos los caracteres `;` por `:`, excepto el último `;` que marca el final del árbol.

Una vez corregido, el archivo puede usarse como entrada para análisis adicionales, por ejemplo rerooting, edición o anotación avanzada.

Además, debido a que el flujo `de_novo_wf` requiere la adición de un grupo extenso como outgroup, este puede eliminarse o redefinirse posteriormente utilizando herramientas de manipulación de árboles, como el paquete `ape` de `R`. Por ejemplo:

```R
library(ape)

tree <- read.tree("gtdbtk.bac120.decorated.tree")
ids_to_keep <- readLines("genomes_for_tree.txt") # Archivo de texto que contiene la lista de IDs de los MAGs que se desea conservar en el árbol filogenético.

tree_no_outgroup <- keep.tip(tree, ids_to_keep) 
write.tree(pruned_tree, file = "tree_no_outgroup.nwk")

```
