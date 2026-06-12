# 2026 Elixir Bioinformatics Course

This repository contains the materials for the 2026 bioinformatics course, organized by day and project. The course emphasizes reproducible analysis, genome surveillance, antibiotic resistance, and RNA sequencing through hands-on, Makefile-driven workflows based on the Biostar Handbook.

## Course structure (Days 1–5)

Each day builds on prior knowledge and introduces new skills:

| Day | Topic | Project | Key skills |
|-----|-------|---------|-----------|
| 1 | Data acquisition & NCBI | `2026-ebola-surveillance/` | Download genomes, understand FASTA/GFF formats |
| 2 | Genome indexing & annotation | `2026-ebola-surveillance/` | Index genomes, explore genomic data |
| 3 | Read alignment & SAM/BAM | *Future: SNP calling* | Map reads, work with BAM files |
| 4 | RNA-Seq: alignment to counts | `2026-UHR-BHR-rnaseq/` | HISAT2 alignment, feature counting, data formatting |
| 5 | Final project & reporting | `2026-staph-antibio/` + synthesis | Reproducible pipeline, figures, and interpretation |

## How to use this repository

1. Follow the day-by-day guide below.
2. Run `make` targets in each project folder to execute workflows.
3. Inspect outputs and verify results.
4. Document your analysis steps in a final report.

---

## Day 1: Data Acquisition (Adatok)

**Goal:** Learn to find, download, and validate public bioinformatic data from NCBI.

**Project:** `2026-ebola-surveillance/`

**What we did:**

- Explored [NCBI Datasets](https://www.ncbi.nlm.nih.gov/datasets/) to find Ebolavirus reference genomes.
- Located the reference URLs for Ebolavirus genome (FASTA) and annotation (GFF).
- Documented URLs in the Makefile:

```makefile
FASTA_URL=https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/848/505/GCF_000848505.1_ViralProj14703/GCF_000848505.1_ViralProj14703_genomic.fna.gz

GFF_URL=https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/848/505/GCF_000848505.1_ViralProj14703/GCF_000848505.1_ViralProj14703_genomic.gff.gz
```

**Key concepts:**

- **Data sources:** NCBI, GenBank, RefSeq
- **Formats:** FASTA (sequences), GFF/GTF (annotations)
- **Automation:** Use `curl` and shell variables to standardize downloads

**Hands-on:**

```bash
cd 2026-ebola-surveillance
make genome
```

---

## Day 2: Genome Indexing & Preprocessing

**Goal:** Index reference genomes and prepare them for downstream analysis.

**Project:** `2026-ebola-surveillance/`

**What we did:**

- Verified downloaded genome FASTA files for correct format and headers.
- Created index files for fast random access (e.g., `.fai` index with `samtools`).
- Explored the GFF annotation file to understand genomic features.

**Key concepts:**

- **Genome indexing:** Makes large sequences searchable by position.
- **Validation:** Check file integrity, sequence length, and header format.
- **Annotation exploration:** Review gene, CDS, and repeat regions.

**Hands-on:**

```bash
make fastq
# or manually:
samtools faidx GCF_000848505.1_ViralProj14703_genomic.fna.gz
```

---

## Day 3: Read Alignment & Sequence Variation

**Goal:** Map reads to a reference and understand BAM/SAM formats.

**Project:** *Reserved for variant calling workflows*

**What we learned (theory):**

- **Aligners:** BWA, Bowtie2, HISAT2 for different data types.
- **SAM/BAM formats:** How to store, filter, and manipulate alignments.
- **Quality metrics:** Coverage depth, mapping quality, clipping.

**Key concepts:**

- **Global vs. local alignment:** Trade-offs in sensitivity and speed.
- **SAM headers and flags:** Interpret alignment records.
- **Filtering:** Keep only high-confidence alignments.

**Future implementation:**

This day's content prepares for SNP calling and variant annotation pipelines, which will be implemented in `2026-staph-antibio/`.

---

## Day 4: RNA-Seq Analysis (Transcriptomics)

**Goal:** Process bulk RNA-Seq data from raw reads to gene expression counts and differential expression.

**Project:** `2026-UHR-BHR-rnaseq/`

**What we did:**

1. **Genome indexing with HISAT2:**
   ```bash
   make index
   ```

2. **Read alignment:**
   ```bash
   make align
   ```
   Produced BAM files from paired-end FASTQ reads.

3. **Feature counting:**
   ```bash
   make count
   ```
   Used `featureCounts` to count reads overlapping genes.

4. **Data formatting & annotation:**
   ```bash
   make to_csv
   make add_names
   ```
   Converted raw counts to annotated CSV.

5. **Exploratory analysis:**
   ```bash
   make pca
   make heatmap
   ```
   Generated visualizations of sample clustering and differential expression.

6. **Differential expression:**
   ```bash
   make edger
   ```
   Used edgeR to compare HBR vs. UHR samples.

**Sample design (`design.csv`):**

```csv
sample,group
HBR_1,HBR
HBR_2,HBR
HBR_3,HBR
UHR_1,UHR
UHR_2,UHR
UHR_3,UHR
```

**Key skills:**

- Index construction and read mapping
- Count table creation and normalization
- Basic statistical testing (edgeR)
- Visualization (PCA, heatmaps)

**Hands-on:**

```bash
cd 2026-UHR-BHR-rnaseq
make download
make index
make align
make count
make edger
make pca
make heatmap
```

---

## Day 5: Final Project (Záróprojekt – Applied Bioinformatics)

**Goal:** Synthesize course skills into a complete, reproducible analysis with results and interpretation.

**Project:** `2026-staph-antibio/` (or extend existing projects)

**What a strong final project includes:**

1. **Research question:**
   - A clear, testable question about microbial epidemiology, resistance, or evolution.
   - Example: "Which Staphylococcus isolates carry β-lactam resistance genes?"

2. **Data acquisition:**
   - Download sequences from NCBI or a public repository.
   - Document all URLs and accession numbers.
   - Verify data integrity.

3. **Reproducible pipeline:**
   - Implement using `Makefile` or shell scripts.
   - Include all commands used for download, preprocessing, analysis, and visualization.
   - Document parameters and software versions.

4. **Results:**
   - Summary tables (e.g., count tables, variant lists, enrichment results).
   - Publication-quality figures (PCA plots, heatmaps, phylogenetic trees).
   - Interpretation of biological findings.

5. **Report:**
   - Short written summary explaining methods, results, and limitations.
   - Reference to final project notebooks or scripts.
   - Reproducibility statement: "All analyses can be reproduced by running `make`."

**Suggested project extensions:**

- **Staphylococcus antibiotic resistance:**
  - Query NCBI for *S. aureus* or *S. epidermidis* genomes.
  - Use BLAST to search for resistance genes from CARD database.
  - Summarize resistance profiles by species or isolation source.

- **Ebola genome expansion:**
  - Download multiple Ebolavirus species/strains.
  - Perform multiple sequence alignment (MSA).
  - Build a phylogenetic tree.
  - Annotate variants relative to the reference.

- **RNA-Seq enhancement:**
  - Add additional sample conditions or tissues.
  - Perform gene set enrichment analysis (GSEA).
  - Generate pathway or GO term enrichment plots.

**Hands-on workflow:**

```bash
mkdir -p final_project
cd final_project

# Set up a reproducible pipeline
cat > Makefile << 'EOF'
# Your final project Makefile
all: download align count analyze report

download:
	# Download data from NCBI or other source
	echo "Download step"

analyze:
	# Run your analysis
	Rscript analysis.r

report:
	# Generate summary tables and figures
	echo "Generate report"
EOF

make all
```

---

## Software and environment

**Required tools:**

- `bash` and `make`
- `curl` for downloading data
- `samtools` for genomic file manipulation
- `HISAT2` or another short-read aligner
- `featureCounts` for read counting
- `Rscript` for statistical analysis and visualization

**Install on Ubuntu/WSL:**

```bash
sudo apt update
sudo apt install -y curl make samtools hisat2 subread r-base
```

**Recommended practices:**

- Keep work in version control (`git`).
- Use relative paths within projects.
- Quote paths with spaces:

```bash
cd "/mnt/c/home/hattrob32/2026 Elixir/2026-ebola-surveillance"
```

- Use shell safety settings in scripts:

```bash
set -eu -o pipefail
```

---

## Using the Biostar Handbook as a guide

This course is informed by the [Biostar Handbook](https://www.biostarhandbook.com/), which emphasizes:

- Real biological data and realistic workflows
- Reproducible, well-documented commands
- Practical problem-solving and troubleshooting
- Steadily building competence from basics to advanced topics

The Handbook is a living reference that updates monthly with new content and corrections, making it both a course text and a career-long resource.

---

## Notes for Windows and WSL users

Windows paths in WSL appear under `/mnt/c/...`. For example:

```bash
# Navigate to the course folder
cd "/mnt/c/home/hattrob32/2026 Elixir/2026-ebola-surveillance"

# Or use escaped spaces
cd /mnt/c/home/hattrob32/2026\ Elixir/2026-ebola-surveillance
```

---

## Summary

This course teaches end-to-end bioinformatics workflows:

- **Day 1:** Locate and download reference genomes from public databases.
- **Day 2:** Index and explore genomic annotation.
- **Day 3:** Understand read alignment and sequence variation (theory + future implementation).
- **Day 4:** Complete RNA-Seq pipeline from reads to differential expression.
- **Day 5:** Combine all skills into a final reproducible research project.

Good luck! Use this repository as your foundation and extend it with your own research questions.
