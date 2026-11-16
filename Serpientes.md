# Filogenia de Serpientes mediante el Gen Mitocondrial `COX1`

Este flujo de trabajo permite descargar, alinear y reconstruir la historia evolutiva de distintas familias de serpientes (Boidae, Colubridae, Viperidae, Elapidae), empleando a *Gallus gallus* como **grupo externo (*outgroup*)** para polarizar las ramas del árbol.

---

## Requisitos Previos (Entorno Linux / WSL)

```bash
sudo apt update
sudo apt install -y ncbi-entrez-direct mafft iqtree figtree jalview
```

---

## Especies Incluidas en el Estudio

| Especie | Familia | Código de Accesión (NCBI RefSeq) | Rol en el Análisis |
| :--- | :--- | :--- | :--- |
| *Python bivittatus* | Pythonidae | `NC_007579.1` | Grupo de estudio (*Ingroup*) |
| *Pantherophis guttatus* | Colubridae | `NC_001322.1` | Grupo de estudio (*Ingroup*) |
| *Thamnophis sirtalis* | Colubridae | `NC_026040.1` | Grupo de estudio (*Ingroup*) |
| *Crotalus atrox* | Viperidae | `NC_011585.1` | Grupo de estudio (*Ingroup*) |
| *Vipera berus* | Viperidae | `NC_025514.1` | Grupo de estudio (*Ingroup*) |
| *Notechis scutatus* | Elapidae | `NC_001325.1` | Grupo de estudio (*Ingroup*) |
| *Laticauda semifasciata* | Elapidae | `NC_000845.1` | Grupo de estudio (*Ingroup*) |
| *Ophiophagus hannah* | Elapidae | `NC_020625.1` | Grupo de estudio (*Ingroup*) |
| *Naja naja* | Elapidae | `NC_028424.1` | Grupo de estudio (*Ingroup*) |
| ***Gallus gallus*** | Phasianidae (Aves) | `NC_001323.1` | **Grupo Externo (*Outgroup*)** |

---

## Pipeline Paso a Paso

### 1. Configuración del Directorio de Trabajo
```bash
mkdir -p ~/proyecto_serpientes/{secuencias,resultados}
cd ~/proyecto_serpientes/secuencias
```

### 2. Descarga de Genomas Mitocondriales desde NCBI
```bash
declare -A MITO=(
  ["Python_bivittatus"]="NC_007579.1"
  ["Pantherophis_guttatus"]="NC_001322.1"
  ["Thamnophis_sirtalis"]="NC_026040.1"
  ["Crotalus_atrox"]="NC_011585.1"
  ["Notechis_scutatus"]="NC_001325.1"
  ["Laticauda_semifasciata"]="NC_000845.1"
  ["Ophiophagus_hannah"]="NC_020625.1"
  ["Naja_naja"]="NC_028424.1"
  ["Vipera_berus"]="NC_025514.1"
  ["Gallus_gallus_Outgroup"]="NC_001323.1"
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

cat *_COX1.fasta > COX1_serpientes.fasta
grep -c '^>' COX1_serpientes.fasta
```

### 4. Alineamiento Múltiple con MAFFT y Visualización en Jalview
```bash
mafft --auto COX1_serpientes.fasta > COX1_serpientes_alineado.fasta

# Abrir alineamiento en interfaz gráfica
jalview COX1_serpientes_alineado.fasta &
```

### 5. Inferencia Filogenética con IQ-TREE y Visualización en FigTree
```bash
iqtree2 -s COX1_serpientes_alineado.fasta -m MFP -bb 1000 -nt AUTO -pre ../resultados/arbol_serpientes

# Abrir árbol filogenético resultante
figtree ../resultados/arbol_serpientes.treefile &
```
