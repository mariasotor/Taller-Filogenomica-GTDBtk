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
