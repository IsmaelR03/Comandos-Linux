# Instalación de Dependencias e Instaladores Generales

```bash
sudo apt update
sudo apt install -y ncbi-blast+ fastqc fastp curl ncbi-entrez-direct mafft iqtree figtree jalview
```

```bash
sh -c "$(curl -fsSL https://ftp.ncbi.nlm.nih.gov/entrez/entrezdirect/install-edirect.sh)"
export PATH=${HOME}/edirect:${PATH}
```

```bash
pip install multiqc --break-system-packages
```

# Verificación de Herramientas Instaladas

```bash
blastn -version
makeblastdb -version
efetch -help | head -n 3
fastqc --version
fastp --version
mafft --version
iqtree2 --version || iqtree --version
figtree -version || echo 'FigTree se abre como aplicación gráfica'
jalview -version || echo 'Jalview se abre como aplicación gráfica'
```

# Práctica: BLAST Local con MEGABLAST

```bash
mkdir -p ~/blast_local/{secuencias,db,resultados}
cd ~/blast_local/secuencias
```

```bash
declare -A MITO=(
  ["Homo_sapiens"]="NC_012920.1"
  ["Pan_troglodytes"]="NC_001644.1"
  ["Mus_musculus"]="NC_005089.1"
  ["Equus_caballus"]="NC_001640.1"
  ["Gallus_gallus"]="NC_001323.1"
  ["Danio_rerio"]="NC_002333.2"
  ["Xenopus_laevis"]="NC_001573.1"
  ["Drosophila_melanogaster"]="NC_024511.2"
  ["Caenorhabditis_elegans"]="NC_001328.1"
  ["Saccharomyces_cerevisiae"]="NC_001224.1"
)

for sp in "${!MITO[@]}"; do
  acc="${MITO[$sp]}"
  efetch -db nuccore -id "$acc" -format fasta_cds_na > "${sp}_cds.fasta"
  echo "Descargado: ${sp}_cds.fasta (${acc})"
done
```

```bash
for sp in "${!MITO[@]}"; do
  awk -v RS='>' '/COX1/{print ">"$0}' "${sp}_cds.fasta" > "${sp}_COX1.fasta"
  sed -i "1s/.*/>${sp}/" "${sp}_COX1.fasta"
done
```

```bash
grep -c '^>' *_COX1.fasta
cat *_COX1.fasta > COX1_10especies.fasta
grep -c '^>' COX1_10especies.fasta
```

```bash
cd ~/blast_local/db
cp ../secuencias/COX1_10especies.fasta .
makeblastdb -in COX1_10especies.fasta -dbtype nucl -parse_seqids -out COX1_db
ls -lh COX1_db.*
```

```bash
cd ~/blast_local
blastn -query secuencias/Homo_sapiens_COX1.fasta \
  -db db/COX1_db -task megablast \
  -outfmt "7 qseqid sseqid pident length evalue bitscore" \
  -out resultados/megablast_homo_sapiens.txt
grep -v '^#' resultados/megablast_homo_sapiens.txt
```

```bash
blastn -query secuencias/Homo_sapiens_COX1.fasta \
  -db db/COX1_db -task dc-megablast \
  -outfmt "7 qseqid sseqid pident length evalue bitscore" \
  -out resultados/dcmegablast_homo_sapiens.txt
grep -v '^#' resultados/dcmegablast_homo_sapiens.txt
```

```bash
blastn -query secuencias/Homo_sapiens_COX1.fasta \
  -db db/COX1_db -task blastn \
  -outfmt "7 qseqid sseqid pident length evalue bitscore" \
  -out resultados/blastn_clasico_homo_sapiens.txt
grep -v '^#' resultados/blastn_clasico_homo_sapiens.txt
```

```bash
time blastn -query secuencias/Homo_sapiens_COX1.fasta -db db/COX1_db -task megablast -out /dev/null
time blastn -query secuencias/Homo_sapiens_COX1.fasta -db db/COX1_db -task blastn -out /dev/null
```

# Práctica: Alineamiento Múltiple de Secuencias y Análisis Filogenético

```bash
mkdir -p ~/filogenia/{secuencias,resultados}
cd ~/filogenia/secuencias
```

```bash
declare -A MITO=(
  ["Homo_sapiens"]="NC_012920.1"
  ["Pan_troglodytes"]="NC_001644.1"
  ["Mus_musculus"]="NC_005089.1"
  ["Equus_caballus"]="NC_001640.1"
  ["Gallus_gallus"]="NC_001323.1"
  ["Danio_rerio"]="NC_002333.2"
  ["Xenopus_laevis"]="NC_001573.1"
  ["Drosophila_melanogaster"]="NC_024511.2"
  ["Caenorhabditis_elegans"]="NC_001328.1"
  ["Saccharomyces_cerevisiae"]="NC_001224.1"
)

for sp in "${!MITO[@]}"; do
  acc="${MITO[$sp]}"
  efetch -db nuccore -id "$acc" -format fasta_cds_na > "${sp}_cds.fasta"
  echo "Descargado: ${sp}_cds.fasta (${acc})"
done
```

```bash
for sp in "${!MITO[@]}"; do
  awk -v RS='>' '/COX1/{print ">"$0}' "${sp}_cds.fasta" > "${sp}_COX1.fasta"
  sed -i "1s/.*/>${sp}/" "${sp}_COX1.fasta"
done
```

```bash
grep -c '^>' *_COX1.fasta
cat *_COX1.fasta > COX1_10especies.fasta
grep -c '^>' COX1_10especies.fasta
```

```bash
cd ~/filogenia/secuencias
mafft --auto COX1_10especies.fasta > COX1_alineado.fasta
grep -c '^>' COX1_alineado.fasta
head -n 2 COX1_alineado.fasta
```

```bash
mafft --auto --clustalout COX1_10especies.fasta > COX1_alineado.clu
less COX1_alineado.clu
```

```bash
cd ~/filogenia/secuencias
iqtree2 -s COX1_alineado.fasta -m MFP -bb 1000 -nt AUTO -pre ../resultados/COX1_tree
```

```bash
cd ~/filogenia/resultados
cat COX1_tree.treefile
less COX1_tree.iqtree
```

```bash
figtree ~/filogenia/resultados/COX1_tree.treefile
```

# Práctica: Análisis de Calidad de Secuencias Illumina Paired-End

```bash
mkdir -p ~/qc_illumina/{raw,trimmed,fastqc_raw,fastqc_trimmed}
cd ~/qc_illumina
```

```bash
cp DRR016723_1.fastq.gz DRR016723_2.fastq.gz ~/qc_illumina/raw/
```

```bash
cd ~/qc_illumina/raw
echo $(zcat DRR016723_1.fastq.gz | wc -l) / 4 | bc
echo $(zcat DRR016723_2.fastq.gz | wc -l) / 4 | bc
ls -lh *.fastq.gz
```

```bash
seqkit head -n 100000 DRR016723_1.fastq.gz -o DRR016723_sub_1.fastq.gz
seqkit head -n 100000 DRR016723_2.fastq.gz -o DRR016723_sub_2.fastq.gz
```

```bash
echo $(zcat DRR016723_sub_1.fastq.gz | wc -l) / 4 | bc
echo $(zcat DRR016723_sub_2.fastq.gz | wc -l) / 4 | bc
```

```bash
cd ~/qc_illumina
fastqc raw/DRR016723_sub_1.fastq.gz raw/DRR016723_sub_2.fastq.gz -o fastqc_raw/
```

```bash
cd fastqc_raw
unzip -o DRR016723_sub_1_fastqc.zip
xdg-open DRR016723_sub_1_fastqc.html
```

```bash
cd ~/qc_illumina
fastp \
  -i raw/DRR016723_sub_1.fastq.gz -I raw/DRR016723_sub_2.fastq.gz \
  -o trimmed/DRR016723_sub_1.trimmed.fastq.gz -O trimmed/DRR016723_sub_2.trimmed.fastq.gz \
  --detect_adapter_for_pe \
  --qualified_quality_phred 20 \
  --length_required 36 \
  --json trimmed/fastp_report.json --html trimmed/fastp_report.html
```

```bash
fastqc trimmed/DRR016723_sub_1.trimmed.fastq.gz trimmed/DRR016723_sub_2.trimmed.fastq.gz -o fastqc_trimmed/
```

```bash
multiqc ~/qc_illumina -o ~/qc_illumina/multiqc_report
```