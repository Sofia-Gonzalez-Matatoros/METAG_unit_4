# METAG_unit_4. Sofía González Matatoros
## Fecha: 20/05/2022
## Ejercicio

Follow the workflow of this tutorial for the taxonomic binning of the virome reads and contigs used as homework in the unit_3.

Reads from unit_3 homework are paired_end reads. To perform this task you can do it in several ways; joining all decontaminated and QF reads and then run DIAMOND or run DIAMOND using both files and then join DIAMOND outputs, etc..

Write a brief summary describing the bioinformatic pipeline you have followed (trimming, decontamination, improve in quality, number of reads remove in each step, etc.) and the most relevant results (two figures maximun) with the taxonomy assessment of the virome at family level.

*Optional: run the same analysis but using the assembled contigs and try to compare the results using MEGAN (File/Compare and load both rma6 files created while making the individual analysis).*

### 1. Nos descargamos los datos
```
mkdir -p ~/Documents/unit_4_tarea
cp ~/Documents/unit_3_tarea/virome.zip ~/Documents/unit_4_tarea
cd ~/Documents/unit_4_tarea
unzip virome.zip
```

### 2. Preprocesado
#### 2.1. Comprobamos la calidad inicial de los datos
```
#--FASTQ quality assessment
mkdir virome_fastqc
fastqc virome_R1.fastq -o virome_fastqc/
fastqc virome_R2.fastq -o virome_fastqc/
```
#### 2.2. Filtramos con Trimmomatic
```
 En la práctica de la unidad 3 se explica por qué se eligen estos parámetros
trimmomatic PE -phred33 virome_R1.fastq virome_R2.fastq virome_R1_QF_paired.fq virome_R1_QF_unpaired.fq virome_R2_QF_paired.fq virome_R2_QF_unpaired.fq SLIDINGWINDOW:4:20 MINLEN:149
```
#### 2.3. Comprobamos la calidad de los datos filtrados
```
mkdir virome_QF_fastqc
fastqc virome_R1_QF_paired.fq -o virome_QF_fastqc/
fastqc virome_R2_QF_paired.fq -o virome_QF_fastqc/
```
#### 2.4. Descontaminación
```
for f in /home/bgm/Documents/unit_3_tarea1/*bt2; do ln -s $f .; done #hacemos los hipervínculos
bowtie2 -x human-phix174 -1 virome_R1_QF_paired.fq -2 virome_R2_QF_paired.fq --un-conc virome_clean.fq -S tmp.sam
cat virome_clean.1.fq virome_clean.2.fq > virome_clean.fq
```
### 2. Alineamineto de las lecturas descontaminadas con la base de datos viral
#### 2.1. Preparación de la base de datos para Diamond
Nos descargamos los ficheros del NCBI: https://ftp.ncbi.nlm.nih.gov/refseq/release/viral/

> viral.1.protein.faa.gz
> viral.2.protein.faa.gz
> viral.3.protein.faa.gz
> viral.4.protein.faa.gz

```
mv ~/Downloads/viral.*.protein.faa.gz ~/Documents/unit_4_tarea
gunzip viral.*.protein.faa.gz
cat viral.*.protein.faa > viral.protein.faa
grep -c ">" *faa
```
> viral.1.protein.faa:51652

> viral.2.protein.faa:219237

> viral.3.protein.faa:290246

> viral.4.protein.faa:18116

> viral.protein.faa:579251

```
diamond makedb --in viral.protein.faa -d viralproteins
diamond blastx -d viralproteins.dmnd -q virome_R1_QF_paired.fq -o virome_R1_QF_paired_vs_viralprotein.m8
diamond blastx -d viralproteins.dmnd -q virome_R2_QF_paired.fq -o virome_R2_QF_paired_vs_viralprotein.m8
```
> Para R1

>> Total time = 11.18s

>> Reported 133163 pairwise alignments, 133163 HSPs.

>> 13533 queries aligned.

> Para R2

>> Total time = 11.684s

>> Reported 154832 pairwise alignments, 154832 HSPs.

>> 15236 queries aligned.

#### 2.2. Descomprimimos el fichero de acceso proporcionado en Moodle
```
unzip prot_acc2tax-Jul2019X1.abin.zip
```
#### 3. Análisis taxonómico con MEGAN6
Para importar los ficheros se ha utilizado la interfaz gráfica, de forma que:
- En *Specify the Blast files to import* se han seleccionado los ficheros: virome_R1_QF_paired_vs_viralprotein.m8, virome_R2_QF_paired_vs_viralprotein.m8
- En *Specify the READ files to import* se han seleccionado virome_clean.1.fq, virome_clean.2.fq
- En *Load Accession mapping file* se ha indicado prot_acc2tax-Jul2019X1.abin
