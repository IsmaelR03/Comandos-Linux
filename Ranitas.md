# Filogenia de Anfibios (Anuro) mediante el Gen Mitocondrial `COX1`

Este repositorio contiene el flujo de trabajo bioinformático reproducible para analizar las relaciones evolutivas entre diversas especies de anfibios (anuros y urodelos), utilizando a *Homo sapiens* como **grupo externo (*outgroup*)** para enraizar el árbol filogenético.

---

## Requisitos Previos (Entorno Linux / WSL)

Asegúrate de tener instaladas las herramientas necesarias antes de ejecutar los comandos:

```bash
sudo apt update
sudo apt install -y ncbi-entrez-direct mafft iqtree figtree jalview
```

---

## Especies Incluidas en el Estudio

| Especie | Grupo Taxonómico | Código de Accesión (NCBI RefSeq) | Rol en el Análisis |
| :--- | :--- | :--- | :--- |
| *Xenopus laevis* | Anura (Pipidae) | `NC_001573.1` | Grupo de estudio (*Ingroup*) |
| *Xenopus tropicalis* | Anura (Pipidae) | `NC_006839.1` | Grupo de estudio (*Ingroup*) |
| *Bufo bufo* | Anura (Bufonidae) | `NC_005329.1` | Grupo de estudio (*Ingroup*) |
| *Rana temporaria* | Anura (Ranidae) | `NC_008068.1` | Grupo de estudio (*Ingroup*) |
| *Pelophylax ridibundus* | Anura (Ranidae) | `NC_002985.1` | Grupo de estudio (*Ingroup*) |
| *Hyla arborea* | Anura (Hylidae) | `NC_015118.1` | Grupo de estudio (*Ingroup*) |
| *Bombina bombina* | Anura (Bombinatoridae) | `NC_007886.1` | Grupo de estudio (*Ingroup*) |
| *Spea multiplicata* | Anura (Scaphiopodidae) | `NC_002621.1` | Grupo de estudio (*Ingroup*) |
| *Pleurodeles waltl* | Urodela (Salamandridae) | `NC_002623.1` | Grupo de estudio (*Ingroup*) |
| ***Homo sapiens*** | Mammalia (Hominidae) | `NC_012920.1` | **Grupo Externo (*Outgroup*)** |

---

### 1. Configuración del Directorio de Trabajo
```bash
mkdir -p ~/proyecto_ranas/{secuencias,resultados}
cd ~/proyecto_ranas/secuencias
```

### 2. Descarga de Genomas Mitocondriales desde NCBI
```bash
declare -A MITO=(
  ["Xenopus_laevis"]="NC_001573.1"
  ["Xenopus_tropicalis"]="NC_006839.1"
  ["Bufo_bufo"]="NC_005329.1"
  ["Rana_temporaria"]="NC_008068.1"
  ["Pelophylax_ridibundus"]="NC_002985.1"
  ["Hyla_arborea"]="NC_015118.1"
  ["Bombina_bombina"]="NC_007886.1"
  ["Spea_multiplicata"]="NC_002621.1"
  ["Pleurodeles_waltl"]="NC_002623.1"
  ["Homo_sapiens_Outgroup"]="NC_012920.1"
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

cat *_COX1.fasta > COX1_ranas.fasta
grep -c '^>' COX1_ranas.fasta
```

### 4. Alineamiento Múltiple con MAFFT y Visualización en Jalview
```bash
mafft --auto COX1_ranas.fasta > COX1_ranas_alineado.fasta

# Abrir alineamiento en interfaz gráfica
jalview COX1_ranas_alineado.fasta &
```

### 5. Inferencia Filogenética con IQ-TREE y Visualización en FigTree
```bash
iqtree2 -s COX1_ranas_alineado.fasta -m MFP -bb 1000 -nt AUTO -pre ../resultados/arbol_ranas

# Abrir árbol filogenético resultante
figtree ../resultados/arbol_ranas.treefile &
```
