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
conda activate qc

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

Plots the percentage of each of the four nucleotides (T, C, A, G) at each position across all reads in the input sequence file.

<img width="901" alt="Screenshot 2024-08-29 at 12 33 45 PM" src="https://github.com/user-attachments/assets/dea864a4-07a6-47b6-add9-2b68b979ccbf">

### Overrepresented sequences and Adapter Content

Shows the very over-represented sequences. This can mean that it is a highly biologically significant sequence, or indicates that the library is contaminated, or not as diverse as expected.

<img width="1048" alt="Screenshot 2024-08-29 at 12 37 52 PM" src="https://github.com/user-attachments/assets/120574f3-8dbb-4524-bdac-62c5e1915321">

# Trimmomatic 

Trimmomatic performs various useful trimming tasks for Illumina paired-end and single-ended data.

```
#!/bin/bash
#SBATCH --job-name trimmomatic
#SBATCH --nodes=1
#SBATCH --mem=40gb
#SBATCH --tasks-per-node=5 # Number of cores per node
#SBATCH --time=40:00:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=dgarcia@amnh.org
#SBATCH --output=slurm-%j-%x.out

conda init bash
conda activate qc

# Set input and output directories
input_dir="/home/dgarcia/mendel-nas1/short_reads/genomes"
output_dir="/home/dgarcia/mendel-nas1/short_reads/trimmed_sequences"

# Loop through all forward sequence files in the input directory
for forward_file in "$input_dir"/*_R1_*.fastq.gz; do
    # Identify the corresponding reverse file
    reverse_file="${forward_file/_R1_/_R2_}"

    # Generate base name for output files
    base_name=$(basename "$forward_file" _R1_*.fastq.gz)

    # Define output files
    output_forward_paired="$output_dir/${base_name}_R1_paired.fastq.gz"
    output_forward_unpaired="$output_dir/${base_name}_R1_unpaired.fastq.gz"
    output_reverse_paired="$output_dir/${base_name}_R2_paired.fastq.gz"
    output_reverse_unpaired="$output_dir/${base_name}_R2_unpaired.fastq.gz"

    # Run Trimmomatic
    trimmomatic PE -threads 5 "$forward_file" "$reverse_file" \
        "$output_forward_paired" "$output_forward_unpaired" \
        "$output_reverse_paired" "$output_reverse_unpaired" \
        ILLUMINACLIP:/home/dgarcia/mendel-nas1/short_reads/trim/TruSeq3-PE-2.fa:2:30:10:8 LEADING:3 TRAILING:10 SLIDINGWINDOW:4:15 MINLEN:40

    echo "Finished processing $base_name"
done

echo "All files have been processed."

```

# Remove contaminants

```
#!/bin/bash
#SBATCH --job-name rm_contaminants
#SBATCH --nodes=1
#SBATCH --mem=40gb
#SBATCH --tasks-per-node=5 # Number of cores per node
#SBATCH --time=40:00:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=dgarcia@amnh.org
#SBATCH --output=slurm-%j-%x.out

module load Java/jdk-1.8.0_281
input_dir="/home/dgarcia/mendel-nas1/short_reads/trimmed_sequences/paired"
output_dir="/home/dgarcia/mendel-nas1/short_reads/clean_genomes"
CREF="/home/dgarcia/mendel-nas1/short_reads/contam/merged_ref_3032904399224293960.fa.gz"

# Loop through all forward sequence files in the input directory
for forward_file in $input_dir/*_R1_paired.fastq.gz; do
 # Identify the corresponding reverse file
    reverse_file="${forward_file/_R1_paired.fastq.gz/_R2_paired.fastq.gz}"

    # Generate base name for output files  
    filename=$(basename "$forward_file")
    base_name=${filename%_R1.fastq.gz}

    # Define output files
    output_forward_clean="$output_dir/${base_name}_R1_clean.fastq.gz"
    output_reverse_clean="$output_dir/${base_name}_R2_clean.fastq.gz"

    # Run bbsplit .sh to remove contaminants
    /home/dgarcia/mendel-nas1/short_reads/contam/bbmap/bbsplit.sh -Xmx85g \
    in1="$forward_file" \
    in2="$reverse_file" \
    ref="$CREF" minid=0.95 \
    outu1="$output_forward_clean" \
    outu2="$output_reverse_clean"

    echo "Finished processing $base_name"
done

echo "All files have been processed."
```

# FastQC on clean genomes

```
#!/bin/bash
#SBATCH --job-name fastqc_cleaned
#SBATCH --nodes=1
#SBATCH --mem=40gb
#SBATCH --tasks-per-node=5
#SBATCH --time=40:00:00
#SBATCH --mail-type=ALL
#SBATCH --mail-user=dgarcia@amnh.org
#SBATCH --output=slurm-%j-%x.out

# Activate conda environment
source ~/.bash_profile
conda activate fastqc_env

# Define directories
INPUT_DIR="/home/dgarcia/mendel-nas1/short_reads/clean_genomes"
OUTPUT_DIR="/home/dgarcia/mendel-nas1/short_reads/fastqc/results/fastqc_2025may6"

# Make sure output directory exists
mkdir -p "$OUTPUT_DIR"

# Loop through all R1 and R2 cleaned FASTQ files
for file in "$INPUT_DIR"/*_clean.fastq.gz; do
    fastqc "$file" -o "$OUTPUT_DIR"
done
```



# Dylans consideration for short read assembly: 

- Trim
- Call saps
- Filter for low quality calls
- Take the reference and add the variance into the reference
- We do this three times:
- 2 GPU : will take around 16 hours in kubernetes 
- with the final gif you have to make an all sites vcd to have a consensus sequence.  
- depth filter! Need to be around 100?
- maybe do not use CACTUS if we are aligning only with one reference genome. 


