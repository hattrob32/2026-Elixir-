# Ebolavirus genome


The Makefile operates on URLS obatined from NCBI datasets:

https://www.ncbi.nlm.nih.gov/datasets/


```
# The URL to the genome.
FASTA_URL=https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/848/505/GCF_000848505.1_ViralProj14703/GCF_000848505.1_ViralProj14703_genomic.fna.gz

# The URL to the annotation.
GFF_URL=https://ftp.ncbi.nlm.nih.gov/genomes/all/GCF/000/848/505/GCF_000848505.1_ViralProj14703/GCF_000848505.1_ViralProj14703_genomic.gff.gz

```

First step download the genome (automatically indexes it)

```
make genome
```

second step to index the genmome

```
make fastq
```
