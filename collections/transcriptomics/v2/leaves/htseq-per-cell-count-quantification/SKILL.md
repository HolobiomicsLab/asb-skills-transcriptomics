---
name: htseq-per-cell-count-quantification
description: 'Use when processing 10x cellranger count output from an infected sample:
  you have extracted host-unmapped reads (barcodes_umi_read_table.csv) and need per-cell
  pathogen read and gene counts integrated into filtered_feature_bc_matrix for Seurat.'
when_to_use_negative:
- The host reference already integrates the pathogen genome (e.g. the human+hIV combined
  genome in the testSets 2a scripts), so pathogen reads align in cellranger count
  and there are no host-unaligned reads to extract
- Only a pathogen FASTA is available without a GTF and per-gene pathogen counts are
  required — htseq gene quantification cannot be produced
- Input is not 10x cellranger count output (scPathoQuant expects the filtered/raw_feature_bc_matrix
  folder structure and possorted_genome_bam.bam)
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: scPathoQuant
  role: Python package that orchestrates unmapped-read extraction, alignment, htseq
    per-cell counting, and integration into 10x matrices with one command
  repo: https://github.com/galelab/scPathoQuant
- name: htseq
  role: Performs the per-cell total-pathogen and per-gene read counting (htseq-count
    with --intersection-nonempty)
- name: samtools
  role: Extracts host-unaligned reads from possorted_genome_bam.bam and generates
    the SAM used to pull barcodes and UMIs
- name: bbmap
  role: Default aligner for mapping extracted unaligned reads to the pathogen reference
    genome (e.g. --semiperfectmode)
- name: bowtie2
  role: Alternative aligner, e.g. with --very-sensitive --non-deterministic
- name: seurat
  role: Downstream R analysis loading the pathogen-integrated filtered_feature_bc_matrix
    via Read10X
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae145
    title: 'scPathoQuant: a tool for efficient alignment and quantification of pathogen
      sequence reads from 10× single cell sequencing datasets'
  source_content_hash: sha256:604d9ed448aabfd577a96db3c11c14bc6a11e4bed513b5072980e03ca10c9299
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/galelab/scPathoQuant
  tools:
  - Python
  - bbmap
  - bowtie2
  - docker
  - htseq
  - samtools
  - scPathoQuant
  - seurat
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# htseq-per-cell-count-quantification

## Summary

Within the scPathoQuant single-command pipeline for 10x scRNAseq data, htseq-count quantifies host-unaligned reads that were re-mapped to a pathogen genome into per-cell total pathogen read counts and, when a pathogen GTF is supplied, per-cell pathogen gene counts that are then integrated back into the cellranger count matrices.

## When to use

Use when processing 10x cellranger count output from an infected sample: you have extracted host-unmapped reads (barcodes_umi_read_table.csv) and need per-cell pathogen read and gene counts integrated into filtered_feature_bc_matrix for Seurat.

## When NOT to use

- The host reference already integrates the pathogen genome (e.g. the human+hIV combined genome in the testSets 2a scripts), so pathogen reads align in cellranger count and there are no host-unaligned reads to extract
- Only a pathogen FASTA is available without a GTF and per-gene pathogen counts are required — htseq gene quantification cannot be produced
- Input is not 10x cellranger count output (scPathoQuant expects the filtered/raw_feature_bc_matrix folder structure and possorted_genome_bam.bam)

## Inputs

- possorted_genome_bam.bam from cellranger count (host-unaligned reads extracted to _tmp/unmmaped.bam)
- barcodes_umi_read_table.csv mapping unmapped reads to cell barcodes and UMIs
- Pathogen reference genome FASTA (simple header recommended, e.g. > HIV_virus)
- Optional pathogen GTF annotation file for per-gene counting

## Outputs

- pathogen_al_counts.csv (per-cell total pathogen read counts)
- pathogen_al_gene_counts.csv (per-cell pathogen gene counts, GTF required)
- pathogen_al_sort_counts.sam and pathogen_genes_al_sort_counts.sam (htseq output)
- pathogen_genes.png violin plot of cells with pathogen gene UMI counts
- filtered_feature_bc_matrix and raw_feature_bc_matrix overwritten with pathogen counts appended

## How to apply

After samtools pulls all reads unaligned to the host genome from possorted_genome_bam.bam into _tmp/ (recording barcodes and UMIs in barcodes_umi_read_table.csv), align them to the pathogen FASTA reference with bbmap (default, e.g. --bbmap_params="--semiperfectmode") or bowtie2 (--very-sensitive --non-deterministic). Run htseq on the pathogen-mapped reads to emit pathogen_al_counts.csv (total reads mapping to the pathogen per cell) and, if a pathogen gtf file was placed in the -p2genome folder, pathogen_genes_al_sort_counts.sam for per-gene counts; note htseq runs with the non-default --intersection-nonempty parameter. scPathoQuant then overwrites the filtered_feature_bc_matrix and raw_feature_bc_matrix to append pathogen UMI and pathogen gene counts so Seurat Read10X can load them. A pathogen GTF is mandatory for gene-level quantification; with FASTA-only input only total per-cell counts are produced.

## Related tools

- **scPathoQuant** (Python package that orchestrates unmapped-read extraction, alignment, htseq per-cell counting, and integration into 10x matrices with one command) — https://github.com/galelab/scPathoQuant
- **htseq** (Performs the per-cell total-pathogen and per-gene read counting (htseq-count with --intersection-nonempty))
- **samtools** (Extracts host-unaligned reads from possorted_genome_bam.bam and generates the SAM used to pull barcodes and UMIs)
- **bbmap** (Default aligner for mapping extracted unaligned reads to the pathogen reference genome (e.g. --semiperfectmode))
- **bowtie2** (Alternative aligner, e.g. with --very-sensitive --non-deterministic)
- **seurat** (Downstream R analysis loading the pathogen-integrated filtered_feature_bc_matrix via Read10X)

## Examples

```
scpathoquant -10x path/to/10x/sample -op path/for/results -p 8 -p2genome path/to/viral/fastafilefolder -align bowtie2 --bowtie2_params="--very-sensitive --non-deterministic"
```

## Evaluation signals

- pathogen_al_counts.csv exists and contains per-cell total pathogen read counts
- pathogen_genes_al_sort_counts.sam is produced only when a pathogen gtf was supplied with the FASTA
- pathogen_genes.png violin plot is generated showing cells with pathogen gene UMI counts
- filtered_feature_bc_matrix files load via Seurat Read10X with pathogen counts appended to the feature matrix
- Alignment ran with htseq's non-default --intersection-nonempty parameter for gene counting

## Limitations

- Per-gene pathogen counts are only produced when a pathogen/viral gtf is provided alongside the FASTA; otherwise only total pathogen read counts are quantified
- The FASTA header is used to quantify total pathogen reads, so complicated headers should be simplified (e.g. > HIV_virus)
- htseq-count runs with the non-default --intersection-nonempty parameter, differing from default settings
- The _tmp/ working directory of intermediate files can be large; pass --tmp_removal to delete it when space is an issue

## Evidence

- [intro] scPathoQuant quantifies total pathogen reads and per-gene counts with htseq: "The software uses outside software samtools, bowtie2, and htseq to quantify pathogen reads"
- [abstract] Quantification covers whole pathogen and individual genes: "quantifies total reads mapping to the entire pathogen"
- [readme] htseq parameterization: "Default parameters are used when aligning reads to the pathogen genome and and non default htseq-count include using the --intersection-nonempty parameter."
- [readme] GTF optional for gene-level output: "pathogen_genes_al_sort_counts.sam - htseq output reads mapping to individual pathogen genes (will not be produced if pathogen/viral gtf is not provided)"
- [intro] Integration into cellranger matrices: "This software integrates pathogen UMI counts and pathogen UMI gene counts into 10x files generated by first running [cellranger count]"
