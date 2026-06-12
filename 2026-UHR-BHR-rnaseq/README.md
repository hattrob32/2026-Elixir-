# RNA-Seq Analysis: HBR vs UHR Samples

This project implements a complete **bulk RNA-Seq analysis pipeline** from raw reads to differential gene expression analysis. It demonstrates the workflow covered in **Day 4** of the [2026 Elixir Bioinformatics Course](../README.md).

## Project Overview

**Goal:** Compare transcriptome profiles between two human cell line conditions:
- **HBR** (Human Brain Reference): 3 biological replicates
- **UHR** (Universal Human Reference): 3 biological replicates

**Workflow:** Download reference genome → Index → Align reads → Count features → Normalize & visualize → Test for differential expression

**Tools used:**
- `curl`: Download data
- `HISAT2`: Fast RNA-Seq alignment
- `featureCounts`: Count reads per gene
- `edgeR`: Statistical testing (R/Bioconductor)
- `ggplot2`: Data visualization (R)

---

## Quick Start

Run the complete pipeline end-to-end:

```bash
cd 2026-UHR-BHR-rnaseq

# Execute all workflow steps
make download
make index
make align
make count
make add_names
make edger
make pca
make heatmap

# Or, for a quick summary of available targets:
make help
```

---

## Makefile Targets Explained

### 1. **`make download`**
Downloads and extracts the pre-packaged RNA-Seq data (genome, annotation, and reads).

```bash
make download
```

**Output:**
- `refs/chr22.genome.fa` – Human chromosome 22 reference sequence (FASTA)
- `refs/chr22.gtf` – Chromosome 22 gene annotation (GTF format)
- `reads/` – FASTQ files for all 6 samples (paired-end: `*_R1.fq` and `*_R2.fq`)

**Source:** Biostar Handbook reference datasets

---

### 2. **`make index`**
Builds a HISAT2 index of the reference genome for fast alignment.

```bash
make index
```

**What happens:**
- Calls `src/run/hisat2.mk` to build index files
- Creates `refs/chr22.genome.*.ht2` index files

**Key concept:** Indexing pre-processes the genome so HISAT2 can quickly map millions of reads without scanning the entire sequence every time.

---

### 3. **`make align`**
Maps RNA-Seq reads to the indexed reference genome.

```bash
make align
```

**What happens:**
- Calls `src/run/hisat2.mk` for each sample
- Processes all 6 samples (HBR_1, HBR_2, HBR_3, UHR_1, UHR_2, UHR_3)
- Generates sorted BAM files: `bam/HBR_1.bam`, `bam/UHR_1.bam`, etc.

**Output:** BAM (Binary Alignment Map) files containing:
- Read-to-genome mappings
- Mapping quality scores
- Strand information

---

### 4. **`make count`**
Counts reads overlapping gene features using `featureCounts`.

```bash
make count
```

**What happens:**
- Reads all 6 BAM files
- Overlaps reads with genes in `refs/chr22.gtf`
- Uses `-s 2` flag (reverse-stranded data: second mate pair strand)
- Outputs raw count matrix

**Output:** `csv/counts.txt`
- Rows: genes (ENSEMBL IDs)
- Columns: samples (HBR_1, HBR_2, …, UHR_3)
- Values: read counts per gene per sample

---

### 5. **`make to_csv`**
Reformats the featureCounts output from text to cleaner CSV format.

```bash
make to_csv
```

**Runs:** `src/r/format_featurecounts.r`

**Output:** `csv/counts.csv` – Same count data, easier to import into R or Python

---

### 6. **`make add_names`**
Adds human-readable gene names to count IDs using BioMart.

```bash
make add_names
```

**What happens:**
1. Creates `csv/tx2gene.csv` (ENSEMBL ID ↔ gene name mapping) via BioMart
2. Runs `src/r/format_featurecounts.r` with the mapping
3. Annotates counts with gene symbols (e.g., `BRCA1`, `TP53`, etc.)

**Output:** `csv/counts.csv` with gene names added to each row

---

### 7. **`make edger`**
Performs differential expression analysis using edgeR (R/Bioconductor).

```bash
make edger
```

**What happens:**
- Runs `src/r/edger.r` with:
  - Count matrix: `csv/counts.csv`
  - Sample metadata: `design.csv` (specifies HBR vs. UHR groups)
- Normalizes library sizes (TMM normalization)
- Fits negative binomial model
- Tests for differential expression (HBR vs. UHR)

**Output:** `csv/gene_expression.csv`
- Significantly differentially expressed genes (DE genes)
- Log-fold change (logFC) and adjusted p-values (FDR)
- Genes upregulated in HBR or UHR

---

### 8. **`make pca`**
Generates a principal component analysis (PCA) plot.

```bash
make pca
```

**What happens:**
- Runs `src/r/plot_pca.r`
- Normalizes counts (log scale, variance stabilization)
- Computes PCA on top variable genes
- Creates a scatter plot (PC1 vs. PC2) colored by group

**Output:** `csv/pca.pdf`
- **Interpretation:** Samples from the same group should cluster together
- **Value:** Reveals batch effects, sample quality, and biological separation

---

### 9. **`make heatmap`**
Generates a heatmap of top differentially expressed genes.

```bash
make heatmap
```

**What happens:**
- Runs `src/r/plot_heatmap.r`
- Selects top DE genes (by p-value or fold change)
- Creates a clustered heatmap with:
  - Rows: genes
  - Columns: samples
  - Color: normalized expression level

**Output:** `csv/heatmap.pdf`
- **Interpretation:** Blue = low expression, red = high expression
- **Pattern:** Genes upregulated in HBR vs. UHR are visually apparent

---

### 10. **`make design`**
Displays or generates the sample metadata file.

```bash
make design
```

**Output:** `design.csv` file contents

**Format:**
```csv
sample,group
HBR_1,HBR
HBR_2,HBR
HBR_3,HBR
UHR_1,UHR
UHR_2,UHR
UHR_3,UHR
```

This file tells R which samples belong to which experimental group for statistical comparisons.

---

## Sample Design

The experiment compares two human reference RNA sources:

| Sample  | Group | Type | Replicates |
|---------|-------|------|------------|
| HBR_1–3 | HBR   | Brain reference | 3 |
| UHR_1–3 | UHR   | Universal reference | 3 |

Each group has 3 biological replicates, allowing edgeR to estimate variance and test for reliable differences.


---

## Output Files

After running the full pipeline, you'll have:

```
csv/
├── counts.txt              # Raw featureCounts output
├── counts.csv              # Reformatted, annotated counts
├── tx2gene.csv             # ENSEMBL ID → gene name mapping
├── gene_expression.csv     # Differential expression results
├── pca.pdf                 # PCA plot (sample clustering)
└── heatmap.pdf             # Heatmap of top DE genes

bam/
├── HBR_1.bam               # Aligned reads
├── HBR_2.bam
├── HBR_3.bam
├── UHR_1.bam
├── UHR_2.bam
└── UHR_3.bam

refs/
├── chr22.genome.fa         # Reference genome
├── chr22.genome.*.ht2      # HISAT2 index files
└── chr22.gtf               # Gene annotation
```

---

## How to Interpret Results

### PCA Plot (`csv/pca.pdf`)
- **Good result:** HBR and UHR samples cluster separately
- **Red flag:** Samples from the same group scattered apart (batch effect or poor quality)

### Heatmap (`csv/heatmap.pdf`)
- **Pattern:** A clear block of red genes in one column group, blue in the other
- **Interpretation:** Those genes are consistently upregulated in one condition

### Differential Expression (`csv/gene_expression.csv`)
- **logFC:** Log2 fold-change (positive = higher in HBR, negative = higher in UHR)
- **FDR:** Adjusted p-value (threshold usually FDR < 0.05 for significance)
- **Example:** BRCA1 with logFC=2.5, FDR=0.001 means it's ~5.6× upregulated in HBR, highly significant

---

## Customizing the Makefile

### Change which samples to process
Edit the `count` target to include different samples:

```makefile
count:
	featureCounts -a ${GTF} -s 2 -o ${COUNT_TXT} \
		bam/YOUR_SAMPLE_1.bam \
		bam/YOUR_SAMPLE_2.bam \
		bam/YOUR_SAMPLE_3.bam
```

### Adjust strandedness
The `-s 2` flag in `featureCounts` specifies reverse-stranded data. Change to:
- `-s 0` for unstranded data
- `-s 1` for forward-stranded data

### Change the reference genome
Update the `REF` and `GTF` variables:

```makefile
REF = refs/my_genome.fa
GTF = refs/my_genome.gtf
```

---

## Linking to the Course

This project implements **Day 4** of the [2026 Elixir Bioinformatics Course](../README.md):

> **Day 4: RNA-Seq Analysis (Transcriptomics)**
> Process bulk RNA-Seq data from raw reads to gene expression counts and differential expression.

For more context on:
- Data acquisition principles: see Day 1 in the course README
- Genome indexing: see Day 2
- Read alignment concepts: see Day 3
- Final project application: see Day 5

---

## Running with Make

**Quick reference:**

```bash
# Show all available targets
make help

# Run one target at a time
make download
make index
make align

# Run all targets in sequence
make download && make index && make align && make count && \
  make add_names && make edger && make pca && make heatmap

# Clean up intermediate files (if a `clean` target exists)
# make clean
```

---

## Troubleshooting

**Problem:** `featureCounts: command not found`
- **Solution:** Install subread: `sudo apt install subread`

**Problem:** `Rscript: command not found`
- **Solution:** Install R: `sudo apt install r-base`

**Problem:** BAM files not created
- **Solution:** Check that `make index` and `make align` completed successfully. Review `samtools` availability.

**Problem:** edgeR fails with "design.csv not found"
- **Solution:** Run `make design` first to generate the metadata file.

---

## References

- **Biostar Handbook:** https://www.biostarhandbook.com/ (RNA-Seq guide and tutorials)
- **HISAT2:** https://daehwankimlab.github.io/hisat2/
- **featureCounts:** http://subread.sourceforge.net/
- **edgeR:** https://bioconductor.org/packages/edgeR/
