# Filogenia de Moscas y Mosquitos (Diptera) mediante `COX1`


## Requisitos Previos (Entorno Linux / WSL)

```bash
sudo apt update
sudo apt install -y ncbi-entrez-direct mafft iqtree figtree jalview
```

---

## Especies Incluidas en el Estudio

| Especie | Importancia / Familia | Código de Accesión (NCBI RefSeq) | Rol en el Análisis |
| :--- | :--- | :--- | :--- |
| *Drosophila melanogaster* | Organismo modelo (Drosophilidae) | `NC_024511.2` | Grupo de estudio (*Ingroup*) |
| *Drosophila simulans* | Especie hermana (Drosophilidae) | `NC_005781.1` | Grupo de estudio (*Ingroup*) |
| *Anopheles gambiae* | Vector de malaria (Culicidae) | `NC_002331.1` | Grupo de estudio (*Ingroup*) |
| *Aedes aegypti* | Vector de dengue/zika (Culicidae) | `NC_010200.1` | Grupo de estudio (*Ingroup*) |
| *Culex pipiens* | Mosquito común (Culicidae) | `NC_014559.1` | Grupo de estudio (*Ingroup*) |
| *Musca domestica* | Mosca doméstica (Muscidae) | `NC_007477.1` | Grupo de estudio (*Ingroup*) |
| *Glossina morsitans* | Mosca tsé-tsé (Glossinidae) | `NC_016834.1` | Grupo de estudio (*Ingroup*) |
| *Rhagoletis cerasi* | Mosca de la cereza (Tephritidae) | `NC_021118.1` | Grupo de estudio (*Ingroup*) |
| *Bactrocera oleae* | Mosca del olivo (Tephritidae) | `NC_021430.1` | Grupo de estudio (*Ingroup*) |
| ***Caenorhabditis elegans*** | Nematodo modelo (Rhabditidae) | `NC_001328.1` | **Grupo Externo (*Outgroup*)** |

---

## Pipeline Paso a Paso

### 1. Configuración del Directorio de Trabajo
```bash
mkdir -p ~/proyecto_diptera/{secuencias,resultados}
cd ~/proyecto_diptera/secuencias
```

### 2. Descarga de Genomas Mitocondriales desde NCBI
```bash
declare -A MITO=(
  ["Drosophila_melanogaster"]="NC_024511.2"
  ["Drosophila_simulans"]="NC_005781.1"
  ["Anopheles_gambiae"]="NC_002331.1"
  ["Aedes_aegypti"]="NC_010200.1"
  ["Culex_pipiens"]="NC_014559.1"
  ["Musca_domestica"]="NC_007477.1"
  ["Glossina_morsitans"]="NC_016834.1"
  ["Rhagoletis_cerasi"]="NC_021118.1"
  ["Bactrocera_oleae"]="NC_021430.1"
  ["Caenorhabditis_elegans_Outgroup"]="NC_001328.1"
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

cat *_COX1.fasta > COX1_diptera.fasta
grep -c '^>' COX1_diptera.fasta
```

### 4. Alineamiento Múltiple con MAFFT y Visualización en Jalview
```bash
mafft --auto COX1_diptera.fasta > COX1_diptera_alineado.fasta

# Abrir alineamiento en interfaz gráfica
jalview COX1_diptera_alineado.fasta &
```

### 5. Inferencia Filogenética con IQ-TREE y Visualización en FigTree
```bash
iqtree2 -s COX1_diptera_alineado.fasta -m MFP -bb 1000 -nt AUTO -pre ../resultados/arbol_diptera

# Abrir árbol filogenético resultante
figtree ../resultados/arbol_diptera.treefile &
```
