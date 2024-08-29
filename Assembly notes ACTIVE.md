# Samples
Samples were sequenced in AZENTA Life and Science and Arlington North Texas Genome center

![image](https://github.com/user-attachments/assets/b286a232-30f3-4df6-854c-5fb791406bd6)

Short-read whole genome sequences (payed for ~30X coverage)
All the raw read of the genomes are located within the Mendel Cluster of AMNH under: 
```
/home/dgarcia/mendel-nas1/short_reads/genomes
```

# FastQC 

First I will do a quality check of the raw reads I obtained from AZENTA and Arlington. For this I installed the _fastqc_ program using Bioconda in a conda environment called fastqc

```
#!/bin/sh
#SBATCH --job-name fastqc_H.danieli_DGC5
#SBATCH --nodes=1
#SBATCH --mem=40gb
#SBATCH --tasks-per-node=5 # Number of cores per node
#SBATCH --time=40:00:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=dgarcia@amnh.org
#SBATCH --output=slurm-%j-%x.out

conda init bash
conda activate fastqc

fastqc /home/dgarcia/mendel-nas1/short_reads/genomes/DG1_R1_001.fastq.gz \
/home/dgarcia/mendel-nas1/short_reads/genomes/DG1_R2_001.fastq.gz \
/home/dgarcia/mendel-nas1/short_reads/genomes/DG2_R1_001.fastq.gz \
/home/dgarcia/mendel-nas1/short_reads/genomes/DG2_R2_001.fastq.gz \
/home/dgarcia/mendel-nas1/short_reads/genomes/DG3_R1_001.fastq.gz \
/home/dgarcia/mendel-nas1/short_reads/genomes/DG3_R2_001.fastq.gz \
/home/dgarcia/mendel-nas1/short_reads/genomes/DG4_R1_001.fastq.gz \
/home/dgarcia/mendel-nas1/short_reads/genomes/DG4_R2_001.fastq.gz \
-o /home/dgarcia/mendel-nas1/short_reads/fastqc/results/fastqc_1

```
## QC results of raw reads

Unfortunately, our results seem to have issues with the adaptor content (this occurred for both the sequences in Arlington (all 
