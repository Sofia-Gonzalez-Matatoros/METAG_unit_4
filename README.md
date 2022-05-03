# METAG_unit_4. Sofía González Matatoros
## Fecha: 20/05/2022
## Ejercicio
Follow the workflow of this tutorial for the taxonomic binning of the virome reads and contigs used as homework in the unit_3.

Reads from unit_3 homework are paired_end reads. To perform this task you can do it in several ways; joining all decontaminated and QF reads and then run DIAMOND or run DIAMOND using both files and then join DIAMOND outputs, etc..

Write a brief summary describing the bioinformatic pipeline you have followed (trimming, decontamination, improve in quality, number of reads remove in each step, etc.) and the most relevant results (two figures maximun) with the taxonomy assessment of the virome at family level.

*Optional: run the same analysis but using the assembled contigs and try to compare the results using MEGAN (File/Compare and load both rma6 files created while making the individual analysis).*

### 1.1. Create a folder in /home/bgm/Documents/:
```
mkdir -p ~/Documents/unit_4_tarea
cp ~/Documents/unit_3_tarea/virome.zip ~/Documents/unit_4_tarea
cd ~/Documents/unit_4_tarea
unzip virome.zip
```
### 1.2. Perform quality filtering and decontamination
```
#--FASTQ quality assessment
mkdir virome_fastqc
fastqc virome_R1.fastq -o virome_fastqc/
fastqc virome_R2.fastq -o virome_fastqc/

#--Trimmomatic quality filtering
trimmomatic PE -phred33 virome_R1.fastq virome_R2.fastq virome_R1_QF.fq virome_R2_QF.fq SLIDINGWINDOW:4:20 MINLEN:70
trimmomatic PE -phred33 virome_R1.fastq virome_R2.fastq virome_R1_QF.fq virome_R2_QF.fq SLIDINGWINDOW:4:15 MINLEN:36
trimmomatic PE -phred33 virome_R1.fastq virome_R2.fastq virome_R1_QF.fq virome_R2_QF.fq SLIDINGWINDOW:5:20 MINLEN:150

trimmomatic SE -phred33 Virome1.fq Virome1_QF.fq SLIDINGWINDOW:5:20 MINLEN:150

#--FASTQ quality assessment
mkdir virome_QF_fastqc
fastqc virome_R1_QF_paired.fq -o virome_QF_fastqc/
fastqc virome_R2_QF_paired.fq -o virome_QF_fastqc/

#--Decontamination
for f in /home/bgm/Documents/unit_3/*bt2; do ln -s $f .; done
bowtie2 -x human-phix174 -q Virome1_QF.fq --un Virome1_QF_clean.fq -S tmp.sam
```
2. Alignment of decontaminated high-quality reads with a viral protein database
2.1. Download RefSeq viral proteins available at NCBI and prepare the database for Diamond.
Download viral protein database files from NCBI: https://ftp.ncbi.nlm.nih.gov/refseq/release/viral/

viral.1.protein.faa.gz
viral.2.protein.faa.gz
viral.3.protein.faa.gz
viral.4.protein.faa.gz
