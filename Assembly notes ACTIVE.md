# Samples
Samples were sequenced in AZENTA Life and Science and Arlington North Texas Genome center (Illumina nova seq)

![image](https://github.com/user-attachments/assets/b286a232-30f3-4df6-854c-5fb791406bd6)

Short-read whole genome sequences (paid for ~30X coverage)
All the raw read of the genomes are located within the Mendel Cluster of AMNH under: 
```
/home/dgarcia/mendel-nas1/short_reads/genomes
```

# FastQC 

First I will do a quality check of the raw reads I obtained from AZENTA and Arlington. For this, I installed the _fastqc_ program using Bioconda in a conda environment called fastqc

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

Unfortunately, our results have issues with the adaptor content. This occurred for all the genome sequences in Arlington and most of the genome sequences in AZENTA. Here I will show the overall results were I am getting a warning and the possible reason behind these results. 

### Per tile sequence quality

Encoded in these is the flowcell tile from which each read came. This graph allows me to look at the quality scores from each tile across all the bases to see if there was a loss in quality associated with only one part of the flowcell. 

<img width="878" alt="Screenshot 2024-08-29 at 12 14 08 PM" src="https://github.com/user-attachments/assets/75457329-420c-48b8-9fb2-ed8f4b61aeb4">

### Per base sequence content

Explanation

<img width="901" alt="Screenshot 2024-08-29 at 12 33 45 PM" src="https://github.com/user-attachments/assets/dea864a4-07a6-47b6-add9-2b68b979ccbf">

### Overrepresented sequences and Adapter Content

Explanation

<img width="1048" alt="Screenshot 2024-08-29 at 12 37 52 PM" src="https://github.com/user-attachments/assets/120574f3-8dbb-4524-bdac-62c5e1915321">

