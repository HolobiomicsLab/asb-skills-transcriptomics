---
name: 10x-cellranger-matrix-integration
description: Use when starting from a completed cellranger count run on 10x scRNAseq
  data from an infected sample and you need pathogen UMI/gene counts appended into
  the 10x matrix files for Seurat/Scanpy downstream analysis.
when_to_use_negative:
- Input is raw FASTQ data — cellranger count must be run first to align host reads
  and generate the 10x output folder this skill consumes
- Data were not produced by the 10x platform / cellranger count (the integration targets
  features.tsv.gz and matrix.mtx.gz in filtered_feature_bc_matrix and raw_feature_bc_matrix
  folders)
- You need de novo viral transcript assembly or multi-host alignment strategies rather
  than remapping cellranger-unaligned reads to a single pathogen reference genome
edam_operation: http://edamontology.org/operation_0362
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPathoQuant
  role: Python package that performs the single-command integration of pathogen UMI
    and gene counts into the 10x cellranger matrix files
  repo: https://github.com/galelab/scPathoQuant
- name: cellranger
  role: Upstream step whose count output (possorted_genome_bam.bam, filtered/raw_feature_bc_matrix)
    is the substrate into which pathogen counts are integrated
- name: samtools
  role: Extracts host-unaligned reads from possorted_genome_bam.bam so barcodes and
    UMIs can be recorded for remapping
- name: htseq
  role: htseq-count with --intersection-nonempty quantifies per-cell reads mapping
    to individual pathogen genes
- name: bowtie2
  role: Alternative aligner (with -align bowtie2) for mapping extracted unaligned
    reads to the pathogen genome
- name: seurat
  role: 'Downstream validation and analysis: Read10X loads the integrated filtered_feature_bc_matrix
    with appended pathogen counts'
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae145
    title: 'scPathoQuant: a tool for efficient alignment and quantification of pathogen
      sequence reads from 10× single cell sequencing datasets'
  source_content_hash: sha256:af246ce1e36e63fcb96ab30a254f8e5a72fdacb02ff70036f0df792c7ba78792
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/galelab/scPathoQuant
  tools:
  - Python
  - bbmap
  - bowtie2
  - cellranger
  - docker
  - htseq
  - samtools
  - scPathoQuant
  - seurat
  edam_operation: http://edamontology.org/operation_0362
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# 10x-cellranger-matrix-integration

## Summary

A scPathoQuant skill that integrates pathogen UMI counts and per-cell pathogen gene counts back into the 10x cellranger output matrices (filtered_feature_bc_matrix and raw_feature_bc_matrix), so that standard single-cell pipelines such as Seurat can analyze host and pathogen reads together from a single command.

## When to use

Use when starting from a completed cellranger count run on 10x scRNAseq data from an infected sample and you need pathogen UMI/gene counts appended into the 10x matrix files for Seurat/Scanpy downstream analysis.

## When NOT to use

- Input is raw FASTQ data — cellranger count must be run first to align host reads and generate the 10x output folder this skill consumes
- Data were not produced by the 10x platform / cellranger count (the integration targets features.tsv.gz and matrix.mtx.gz in filtered_feature_bc_matrix and raw_feature_bc_matrix folders)
- You need de novo viral transcript assembly or multi-host alignment strategies rather than remapping cellranger-unaligned reads to a single pathogen reference genome

## Inputs

- 10x cellranger count output folder for one sample (possorted_genome_bam.bam, filtered_feature_bc_matrix, raw_feature_bc_matrix)
- Pathogen reference genome FASTA file (header used as the pathogen total-reads identifier, e.g. > HIV_virus)
- Optional pathogen GTF file (required only for per-gene pathogen counts)
- Output directory path and processor count (-op, -p)

## Outputs

- Overwritten filtered_feature_bc_matrix and raw_feature_bc_matrix with pathogen counts and pathogen gene counts appended
- pathogen_al_counts.csv — per-cell total pathogen read counts
- pathogen_al_gene_counts.csv and pathogen_genes_al_sort_counts.sam — per-cell per-pathogen-gene htseq counts (gtf-dependent)
- pathogen_al.bam, pathogen_al_mapped.bam, pathogen_al_mapped_sort.bam (+ .bai), pathogen_al_mapped.fq.gz for IGV visualization and phylogenetic/clade analyses
- pathogen_copy.png, pathogen_genes.png, coveragemap.png violin plots
- _tmp/ intermediates (unmmaped.bam, unmmaped.sam, barcodes_umi_read_table.csv, unmapped.fq.gz), optionally deleted with --tmp_removal

## How to apply

Run scPathoQuant against the sample's cellranger output folder (-10x) after cellranger count has produced possorted_genome_bam.bam: the tool uses samtools to pull all host-unmapped reads into the _tmp/ working directory, recording barcodes and UMIs in barcodes_umi_read_table.csv, then realigns them to the pathogen reference genome folder (-p2genome, fasta plus optional gtf) with bbmap (default, e.g. --bbmap_params="--semiperfectmode") or bowtie2 (e.g. --bowtie2_params="--very-sensitive --non-deterministic"). Per-cell totals are quantified into pathogen_al_counts.csv, and when a pathogen gtf is supplied htseq-count with the non-default --intersection-nonempty parameter produces per-gene counts (pathogen_genes_al_sort_counts.sam). The final integration step overwrites the original 10x files so pathogen counts and pathogen gene counts become additional features in filtered_feature_bc_matrix and raw_feature_bc_matrix, alongside violin plot PNGs (pathogen_copy.png, pathogen_genes.png, coveragemap.png). Pass --tmp_removal if disk space is limited since _tmp/ files can be large. Validation is done by loading the integrated matrix via Seurat Read10X and confirming the appended pathogen features appear.

## Related tools

- **scPathoQuant** (Python package that performs the single-command integration of pathogen UMI and gene counts into the 10x cellranger matrix files) — https://github.com/galelab/scPathoQuant
- **cellranger** (Upstream step whose count output (possorted_genome_bam.bam, filtered/raw_feature_bc_matrix) is the substrate into which pathogen counts are integrated)
- **samtools** (Extracts host-unaligned reads from possorted_genome_bam.bam so barcodes and UMIs can be recorded for remapping)
- **htseq** (htseq-count with --intersection-nonempty quantifies per-cell reads mapping to individual pathogen genes)
- **bowtie2** (Alternative aligner (with -align bowtie2) for mapping extracted unaligned reads to the pathogen genome)
- **seurat** (Downstream validation and analysis: Read10X loads the integrated filtered_feature_bc_matrix with appended pathogen counts)

## Examples

```
seurat.object.data <- Read10X(data.dir ="output_scPathoQuant/filtered_feature_bc_matrix")
```

## Evaluation signals

- pathogen_al_counts.csv exists and contains per-cell pathogen read counts
- filtered_feature_bc_matrix files load via Seurat Read10X with pathogen counts (and pathogen gene counts when a gtf was provided) appended as extra features
- pathogen_al_mapped_sort.bam is present with its .bai index and can be visualized in IGV
- pathogen_genes_al_sort_counts.sam is produced only when a pathogen/viral gtf was provided
- Violin plot PNGs (pathogen_copy.png, pathogen_genes.png, coveragemap.png) are generated showing cells with pathogen and UMI counts

## Limitations

- The _tmp/ directory of intermediate files can be very large; use --tmp_removal if disk space is an issue
- The pathogen fasta header is used as the total-reads identifier, so complicated headers should be simplified (e.g. > HIV_virus) before running
- A pathogen gtf is not mandatory but per-gene pathogen counts are not produced without one
- raw_feature_bc_matrix integration 'is not always needed' per the README

## Evidence

- [readme] This software integrates pathogen UMI counts and pathogen UMI gene counts into 10x files generated by first running: "This software integrates pathogen UMI counts and pathogen UMI gene counts into 10x files generated by first running [cellranger count]"
- [abstract] integrates pathogen sequence counts into matrix files that are used by standard single cell pipelines for downstream analyses with only one command: "integrates pathogen sequence counts into matrix files that are used by standard single cell pipelines for downstream analyses with only one command"
- [readme] Overwrites original 10x data provided to include pathogen counts and pathogen gene counts (if gtf file is provided): "Overwrites original 10x data provided to include pathogen counts and pathogen gene counts (if gtf file is provided)"
- [readme] non default htseq-count include using the --intersection-nonempty parameter: "Default parameters are used when aligning reads to the pathogen genome and and non default htseq-count include using the --intersection-nonempty parameter."
- [readme] filtered_feature_bc_matrix - files that can be integrated into seurat with pathogen counts: "filtered_feature_bc_matrix - files that can be integrated into seurat with pathogen counts"
