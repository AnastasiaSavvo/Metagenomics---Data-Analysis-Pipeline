# Metagenomics-Data-Analysis-Pipeline 
## 1. Downoalding and preprocessing the data
First I chose 2 SRR accession numbers and downloaded them with the help of sratoollkit. Then I made a txt file to use it in the loops in the downstream analysis. Also, in this step I do the quality control with fastqc
```bash
module spider sra
module load gcc/14.2.0
module load sra-tools/3.0.3
nano list.txt
while read SRR; do  prefetch $SRR;  fasterq-dump $SRR --split-files; done < list.tx
module load gcc/14.2.0 fastqc/0.12.1
fastqc SRR8397893_1.fastq
fastqc SRR8397897_1.fastq
```
## 2. Trimming 
After the QC, I use trimmomatic to trim the adapters and low-quality reads.
```bash
module load gcc/9.4.0-eewq4j6 trimmomatic/0.39-dlgljoz
for sample in *_1.fastq.gz
do
    base=$(basename $sample _1.fastq.gz)

    trimmomatic PE -threads 4 -phred33 \
        ${base}_1.fastq ${base}_2.fastq \
        ${base}_1_P.fastq ${base}_1_U.fastq \
        ${base}_2_P.fastq ${base}_2_U.fastq \
        ILLUMINACLIP:TruSeq3-PE.fa:2:30:10 \
        LEADING:3 TRAILING:3 SLIDINGWINDOW:4:15 MINLEN:36
done
```
## 3. Assembly of the data into contigs
I use metaspades to assemble the reads into contigs.
```bash 
module load spades/4.0.0
for sample in SRR8397893 SRR8397897
do
    metaspades.py \
        -1 ${sample}_1_paired.fastq \
        -2 ${sample}_2_paired.fastq \
        -o assembly_${sample} \
        -t 16 -m 128
done
```


