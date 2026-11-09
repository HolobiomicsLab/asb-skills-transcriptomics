---
name: dual-virus-read-differentiation-within-sample
description: Use when a 10x scRNAseq sample may harbor a co-infection and you need
  to detect, differentiate, and quantify reads for more than one pathogen genome within
  the same cellranger-processed output.
when_to_use_negative:
- Only a single pathogen is of interest and a single-fasta -p2genome reference suffices
  — the dual-fasta setup is only needed for differentiating co-infections within one
  sample
- Input reads have not been run through cellranger count against the host genome first
  — scPathoQuant takes the unaligned reads from the host genome and cannot operate
  on raw FASTQ alone
- You need per-strain/clade resolution beyond a two-genome reference — the -p2genome
  folder accepts at most 2 fasta files
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3174
- http://edamontology.org/topic_3308
tools:
- name: scPathoQuant
  role: 'core tool: extracts cellranger-unaligned reads, aligns them to the dual-virus
    reference (default bbmap), and quantifies per-cell per-virus read and UMI counts'
  repo: https://github.com/galelab/scPathoQuant
- name: cellranger
  role: 'preprocessing: aligns reads to the host genome so scPathoQuant can operate
    on the unaligned fraction'
- name: Python
  role: scPathoQuant is a python package; used for aggregating per-patient per-virus
    counts into a detection table
  repo: https://github.com/galelab/scPathoQuant
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae145
    title: 'scPathoQuant: a tool for efficient alignment and quantification of pathogen
      sequence reads from 10× single cell sequencing datasets'
  source_content_hash: sha256:5d17854e4b632ff61e72e1f6489ee9ab5276f423704beb03619705c16a6b9bfa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/galelab/scPathoQuant
  tools:
  - Python
  - bowtie2
  - cellranger
  - scPathoQuant
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3174
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# dual-virus-read-differentiation-within-sample

## Summary

Detect and separately quantify two co-infecting viruses (e.g. SARS-CoV-2 and metapneumovirus) within a single 10x scRNAseq sample by pointing scPathoQuant at a pathogen reference folder containing two fasta entries, yielding per-cell per-virus read counts that differentiate each virus.

## When to use

Use when a 10x scRNAseq sample may harbor a co-infection and you need to detect, differentiate, and quantify reads for more than one pathogen genome within the same cellranger-processed output.

## When NOT to use

- Only a single pathogen is of interest and a single-fasta -p2genome reference suffices — the dual-fasta setup is only needed for differentiating co-infections within one sample
- Input reads have not been run through cellranger count against the host genome first — scPathoQuant takes the unaligned reads from the host genome and cannot operate on raw FASTQ alone
- You need per-strain/clade resolution beyond a two-genome reference — the -p2genome folder accepts at most 2 fasta files

## Inputs

- cellranger count output folder per 10x sample (filtered_feature_bc_matrix, possorted_genome_bam with unmapped reads)
- Pathogen reference folder (-p2genome) containing up to 2 fasta files, one per virus (e.g. SARS-CoV-2 GCF_009858895.2 and a metapneumovirus genome), with simplified fasta headers; optional pathogen gtf files

## Outputs

- pathogen_al_counts.csv — total number of reads mapping to each pathogen in each cell
- pathogen_copy.png — violin plot of cells with pathogen and UMI counts per virus
- pathogen_al_gene_counts.csv and pathogen_genes.png (if gtf provided)
- pathogen_al_mapped_sort.bam/.bai and pathogen_al_mapped.fq.gz for IGV visualization and phylogenetic analysis
- Aggregated per-patient per-virus detection table (e.g. patient_virus_detection_table.csv)

## How to apply

Assemble a pathogen reference folder holding at most two fasta entries — e.g. the SARS-CoV-2 reference genome (NCBI GCF_009858895.2) and a second viral genome such as metapneumovirus — with simplified fasta headers (e.g. >HIV_virus style), because scPathoQuant uses the fasta header to quantify total pathogen reads. For each sample that has already been processed with cellranger count, run scPathoQuant with -10x pointing at the cellranger output folder, -op for a results folder, -p for processors, -p2genome pointing at the dual-virus reference folder, keeping the default bbmap aligner (or switch to bowtie2 with -align bowtie2). scPathoQuant extracts reads left unaligned by cellranger, maps them to the multi-genome pathogen reference, and reports per-virus results in pathogen_al_counts.csv (total reads mapping to each pathogen in each cell) plus pathogen_copy.png violin plots of infected-cell and UMI counts. Aggregate the per-patient, per-cell, per-virus counts into a detection table listing infected-cell counts and read abundances separately for each virus, exploiting SPQ's ability to differentiate multiple viruses in a single sample. Validate by checking that the table reproduces the expected detection pattern, e.g. both viruses detected in the co-infected patient and only SARS-CoV-2 in the others.

## Related tools

- **scPathoQuant** (core tool: extracts cellranger-unaligned reads, aligns them to the dual-virus reference (default bbmap), and quantifies per-cell per-virus read and UMI counts) — https://github.com/galelab/scPathoQuant
- **cellranger** (preprocessing: aligns reads to the host genome so scPathoQuant can operate on the unaligned fraction)
- **Python** (scPathoQuant is a python package; used for aggregating per-patient per-virus counts into a detection table) — https://github.com/galelab/scPathoQuant

## Examples

```
scpathoquant -10x path/to/10x/sample -op path/for/results -p 8 -p2genome path/to/pathogen/fastafilefolder
```

## Evaluation signals

- pathogen_al_counts.csv must contain separate per-virus read counts for both entries in the multi-fasta reference (differentiation succeeded, not merged under one header)
- Co-infection detection pattern reproduced: both SARS-CoV-2 and metapneumovirus detected in the co-infected patient, SARS-CoV-2 only in the other patients (Fig. 4A/B pattern)
- pathogen_copy.png violin plots show non-zero infected-cell counts and UMI distributions per virus
- 10x filtered_feature_bc_matrix output overwritten with pathogen UMI counts and pathogen UMI gene counts integrated so Seurat can read them via Read10X

## Limitations

- The -p2genome folder should contain at most 2 fasta files, so this approach is limited to differentiating up to two viruses per run
- Fasta headers are used to quantify total pathogen reads, so complicated headers must be simplified or quantification is misleading
- Gene-level output (pathogen_genes.png / pathogen_al_gene_counts.csv) is not produced if a pathogen gtf is not provided
- Temporary _tmp/ files can be large and should be removed with --tmp_removal if space is an issue

## Evidence

- [abstract] can differentiate and define multiple viruses in a single sample scRNAseq output: "can differentiate and define multiple viruses in a single sample scRNAseq output"
- [readme] in this folder should be at most 2 files ... Note: In the fasta file the header will be used to quantify the number of pathogen total reads: "Note: In the fasta file the header will be used to quantify the number of pathogen total reads, it is recommended that if the fasta header is a complicated name it be simplified"
- [readme] Data to test scPathoQuants ability to quantify multiple infections in a single cell sample: [evidence span withheld — anchor status: external_doc]
- [readme] pathogen_al_counts.csv - total number of reads mapping to the pathogen in each cell: "pathogen_al_counts.csv - total number of reads mapping to the pathogen in each cell"
- [intro] scPathoQuant then takes the unaligned reads from the host genome and maps them to your pathogen genome of interest and then integrates them back into the cellranger files: "scPathoQuant then takes the unaligned reads from the host genome and maps them to your pathogen genome of interest and then integrates them back into the cellranger files"
