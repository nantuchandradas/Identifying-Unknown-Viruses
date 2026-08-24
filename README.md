# Identifying-Unknown-Viruses
# NGS Project: Identifying Unknown Viruses

**Author:** Nantu Chandra Das (Student ID: 03780564)  
**Group:** Group-04  
**Institution:** TUM School of Life Sciences, Technical University of Munich  
**Course:** Next-Generation Sequencing (NGS) - Project 4  

---

## Executive Summary

This project implements an end-to-end Next-Generation Sequencing (NGS) computational pipeline to process, assemble, identify, and analyze unknown viral genomes from short-read Illumina paired-end data. Out of seven total group samples, samples 4, 5, 6, and 7 were processed and analyzed in this workflow.

Key workflow components:
1. **Quality Control & Filtering**: Adapter trimming, contaminant removal ($\Phi$X174/artifacts), and quality trimming using BBDuk and FastQC.
2. **De Novo Assembly**: Parameter sweep across k-mer sizes using SPAdes, evaluated via QUAST.
3. **Taxonomic Identification**: Homology search against NCBI nucleotide database using `blastn`.
4. **Read Mapping**: Comparative evaluation of Bowtie2 (local) and BBMap (global).
5. **Variant Calling**: Low-frequency variant and indel discovery using FreeBayes v1.3.4.

---

## Identified Viral Genomes

Through de novo assembly and BLAST taxonomic assignment, four distinct viral species were identified at >99% sequence identity:

| Sample | Identified Virus | Reference Accession | Genome Size (bp) | Best K-mer | Assembly N50 | Mapped Reads (%) |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Sample 4** | Equine rhinitis B virus 1 | NC_003983.1 | 8,828 | 55 | 8,828 | 99.99% |
| **Sample 5** | Chikungunya virus | NC_004162.2 | 11,826 | 55 | 11,826 | 99.99% |
| **Sample 6** | Duck hepatitis A virus 1 (R85952) | NC_008250.2 | 7,711 | 33 | 7,711 | 99.99% |
| **Sample 7** | Zika virus | NC_012532.1 | 10,794 | 33 | 10,794 | 99.99% |

---

## Pipeline & Methodology

### 1. Quality Control & Data Cleaning
- **Initial QC**: Raw reads evaluated using `fastqc -o fastqc_raw --nogroup -t 14 *.fastq.gz`.
- **Adapter & Contaminant Trimming**: Processed via `bbduk.sh` using default adapter databases (`adapters.fa`) and $\Phi$X contamination references.
- **Quality Metrics Post-Cleaning**:
  - **Average Read Loss**: ~31.0%
  - **Quality Score Gain ($\Delta$Phred)**: +4.7
  - **Adapter Reduction**: 100.0%

### 2. De Novo Assembly Optimization
- Tested 36 assembly combinations per library running a full-factorial grid search across k-mer lengths (`k = 21, 33, 55, 77, 99, 127`) and coverage cutoffs (`2`, `10`, `auto`).
- **Optimal Settings**: `auto` coverage cutoff with $k=33$ or $k=55$ yielded the highest Assembly Quality Scores (AQS average = 83.7).

### 3. Read Alignment & Mapper Comparison
Alignment performance was evaluated side-by-side using local (`Bowtie2`) and global (`BBMap`) aligners:

| Sample | Mean Depth (Bowtie2) | Mean Depth (BBMap) | Mean Q-Score (Bowtie2) | Mean Q-Score (BBMap) |
| :--- | :--- | :--- | :--- | :--- |
| **Sample 4** | 8,178.19$\times$ | 8,421.94$\times$ | 43.76 | 39.35 |
| **Sample 5** | 6,123.73$\times$ | 6,298.19$\times$ | 43.09 | 39.40 |
| **Sample 6** | 9,365.43$\times$ | 9,641.41$\times$ | 43.78 | 39.42 |
| **Sample 7** | 6,705.05$\times$ | 6,901.08$\times$ | 43.77 | 39.41 |

*Note: Bowtie2 local mapping was retained for downstream variant discovery due to superior mean mapping quality scores.*

### 4. Variant Discovery
Variants were identified using FreeBayes v1.3.4 with strict quality thresholds (Min. Coverage: 10$\times$, Min. Base Quality: Q20, Min. Alternate Fraction: 0.1, Hard Filters: QUAL > 30, DP > 10):

- **Sample 4**: 1 Indel, 0 SNPs
- **Sample 5**: 1 Indel, 2 SNPs
- **Sample 6**: 0 Indels, 2 SNPs
- **Sample 7**: 1 Indel, 1 SNP
- **Total across samples**: 8 variants detected (5 SNPs, 3 Indels), demonstrating high sequence conservation relative to NCBI reference genomes.

---

## Software Dependencies

- [FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) (v0.11.9+)
- [BBTools / BBDuk / BBMap](https://jgi.doe.gov/data-and-tools/software-tools/bbtools/)
- [SPAdes Assembler](https://github.com/ablab/spades)
- [NCBI BLAST+](https://blast.ncbi.nlm.nih.gov/Blast.cgi)
- [Bowtie2](http://bowtie-bio.sourceforge.net/bowtie2/index.shtml)
- [Samtools](http://www.htslib.org/)
- [FreeBayes](https://github.com/freebayes/freebayes) (v1.3.4)
