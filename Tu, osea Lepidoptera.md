# Filogenia de Mariposas y Polillas (Lepidoptera) mediante `COX1`


## Requisitos Previos (Entorno Linux / WSL)

```bash
sudo apt update
sudo apt install -y ncbi-entrez-direct mafft iqtree figtree jalview
```

---

## Especies Incluidas en el Estudio

| Especie | Nombre Común / Familia | Código de Accesión (NCBI RefSeq) | Rol en el Análisis |
| :--- | :--- | :--- | :--- |
| *Bombyx mori* | Gusano de seda (Bombycidae) | `NC_003310.1` | Grupo de estudio (*Ingroup*) |
| *Danaus plexippus* | Mariposa monarca (Nymphalidae) | `NC_009848.1` | Grupo de estudio (*Ingroup*) |
| *Heliconius melpomene* | Mariposa cartera (Nymphalidae) | `NC_015949.1` | Grupo de estudio (*Ingroup*) |
| *Bicyclus anynana* | Mariposa marrón (Nymphalidae) | `NC_013589.1` | Grupo de estudio (*Ingroup*) |
| *Pieris rapae* | Blanca de la col (Pieridae) | `NC_014600.1` | Grupo de estudio (*Ingroup*) |
| *Papilio xuthus* | Cola de golondrina (Papilionidae) | `NC_014272.1` | Grupo de estudio (*Ingroup*) |
| *Manduca sexta* | Esfinge del tabaco (Sphingidae) | `NC_004193.1` | Grupo de estudio (*Ingroup*) |
| *Spodoptera frugiperda* | Cogollero del maíz (Noctuidae) | `NC_023223.1` | Grupo de estudio (*Ingroup*) |
| *Plutella xylostella* | Polilla dorso de diamante (Plutellidae) | `NC_012648.1` | Grupo de estudio (*Ingroup*) |
| ***Drosophila melanogaster*** | Mosca de la fruta (Diptera) | `NC_024511.2` | **Grupo Externo (*Outgroup*)** |

---

## Pipeline Paso a Paso

### 1. Configuración del Directorio de Trabajo
```bash
mkdir -p ~/proyecto_lepidoptera/{secuencias,resultados}
cd ~/proyecto_lepidoptera/secuencias
```

### 2. Descarga de Genomas Mitocondriales desde NCBI
```bash
declare -A MITO=(
  ["Bombyx_mori"]="NC_003310.1"
  ["Danaus_plexippus"]="NC_009848.1"
  ["Heliconius_melpomene"]="NC_015949.1"
  ["Pieris_rapae"]="NC_014600.1"
  ["Manduca_sexta"]="NC_004193.1"
  ["Spodoptera_frugiperda"]="NC_023223.1"
  ["Plutella_xylostella"]="NC_012648.1"
  ["Bicyclus_anynana"]="NC_013589.1"
  ["Papilio_xuthus"]="NC_014272.1"
  ["Drosophila_melanogaster_Outgroup"]="NC_024511.2"
)

for sp in "${!MITO[@]}"; do
  acc="${MITO[$sp]}"
  efetch -db nuccore -id "$acc" -format fasta_cds_na > "${sp}_cds.fasta"
  echo "Descargado: ${sp}_cds.fasta (${acc})"
done
```

### 3. Extracción y Limpieza del Gen `COX1`
```bash
for sp in "${!MITO[@]}"; do
  awk -v RS='>' '/COX1/{print ">"$0}' "${sp}_cds.fasta" > "${sp}_COX1.fasta"
  sed -i "1s/.*/>${sp}/" "${sp}_COX1.fasta"
done

cat *_COX1.fasta > COX1_lepidoptera.fasta
grep -c '^>' COX1_lepidoptera.fasta
```

### 4. Alineamiento Múltiple con MAFFT y Visualización en Jalview
```bash
mafft --auto COX1_lepidoptera.fasta > COX1_lepidoptera_alineado.fasta

# Abrir alineamiento en interfaz gráfica
jalview COX1_lepidoptera_alineado.fasta &
```

### 5. Inferencia Filogenética con IQ-TREE y Visualización en FigTree
```bash
iqtree2 -s COX1_lepidoptera_alineado.fasta -m MFP -bb 1000 -nt AUTO -pre ../resultados/arbol_lepidoptera

# Abrir árbol filogenético resultante
figtree ../resultados/arbol_lepidoptera.treefile &
```
