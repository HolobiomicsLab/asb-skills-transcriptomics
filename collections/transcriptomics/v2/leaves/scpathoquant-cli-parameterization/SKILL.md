---
name: scpathoquant-cli-parameterization
description: Use when a user has a completed cellranger count output for 10x scRNAseq
  data and wants pathogen (viral or bacterial) reads aligned, quantified per cell,
  and appended to the filtered/raw feature-barcode matrices without leaving the host
  pipeline.
when_to_use_negative:
- The sequencing data are not 10x scRNAseq cellranger outputs — scPathoQuant is built
  specifically around 10x files (features.tsv.gz, matrix.mtx.gz, possorted_genome_bam.bam).
- Reads have already been aligned to the pathogen genome and quantified by another
  tool (e.g. ViralTrack, Venus, PathogenTrack); re-running SPQ would duplicate the
  integration step.
- Only host-cell transcriptomic analysis is needed with no pathogen genome reference
  available — the -p2genome fasta is mandatory.
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0749
tools:
- name: scPathoQuant
  role: Python package whose CLI parameters (-10x, -op, -p, -p2genome, -align, --tmp_removal)
    define the single-command pathogen alignment and quantification run
  repo: https://github.com/galelab/scPathoQuant
- name: cellranger
  role: 'Upstream prerequisite: aligns reads to the host genome and produces the 10x
    output folder supplied via -10x'
- name: samtools
  role: Pulls host-unmapped reads from possorted_genome_bam.bam and records barcodes/UMIs
    into barcodes_umi_read_table.csv
- name: bbmap
  role: Default aligner for mapping extracted unmapped reads to the pathogen reference
    (parameterized via --bbmap_params)
- name: bowtie2
  role: Alternative aligner selected with -align bowtie2 and tuned via --bowtie2_params
    (e.g. --very-sensitive --non-deterministic)
- name: htseq
  role: Quantifies per-cell total and per-gene pathogen reads (htseq-count with --intersection-nonempty)
- name: docker
  role: Alternative runtime via docker build -t scpathoquant, with bind mounts for
    input, genome, and output
- name: seurat
  role: Downstream validation/consumption of the pathogen-integrated filtered_feature_bc_matrix
    via Read10X
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae145
    title: 'scPathoQuant: a tool for efficient alignment and quantification of pathogen
      sequence reads from 10× single cell sequencing datasets'
  source_content_hash: sha256:ed5c6c7de756267528c2eb1944b65faef534b78321c26c826a340aa5c692a921
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
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0749
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# scpathoquant-cli-parameterization

## Summary

Configures and launches the scPathoQuant Python package with its CLI parameters (-10x, -op, -p, -p2genome, -align, aligner-specific params, --tmp_removal) to align host-unaligned reads to a pathogen genome and integrate pathogen UMI counts back into 10x cellranger output in a single command.

## When to use

Use when a user has a completed cellranger count output for 10x scRNAseq data and wants pathogen (viral or bacterial) reads aligned, quantified per cell, and appended to the filtered/raw feature-barcode matrices without leaving the host pipeline.

## When NOT to use

- The sequencing data are not 10x scRNAseq cellranger outputs — scPathoQuant is built specifically around 10x files (features.tsv.gz, matrix.mtx.gz, possorted_genome_bam.bam).
- Reads have already been aligned to the pathogen genome and quantified by another tool (e.g. ViralTrack, Venus, PathogenTrack); re-running SPQ would duplicate the integration step.
- Only host-cell transcriptomic analysis is needed with no pathogen genome reference available — the -p2genome fasta is mandatory.

## Inputs

- 10x cellranger count output folder (possorted_genome_bam.bam, filtered_feature_bc_matrix, raw_feature_bc_matrix)
- Pathogen reference genome FASTA file (header name used for total-read quantification)
- Optional pathogen GTF annotation file for per-gene counts

## Outputs

- pathogen_al_counts.csv — per-cell total pathogen read counts
- pathogen_al_gene_counts.csv — per-cell pathogen gene read counts
- pathogen_al.bam, pathogen_al_mapped.bam, pathogen_al_mapped_sort.bam (+ .bai) — mapped pathogen read artifacts
- pathogen_al_mapped.fq.gz — mapped reads fastq for phylogenetic/clade analyses
- pathogen_al_sort_counts.sam and pathogen_genes_al_sort_counts.sam — htseq outputs (latter only when gtf provided)
- Overwritten filtered_feature_bc_matrix and raw_feature_bc_matrix including pathogen counts
- pathogen_copy.png, pathogen_genes.png, coveragemap.png violin/coverage plots
- _tmp/ directory with unmmaped.bam, barcodes_umi_read_table.csv, unmapped.fq.gz (removed if --tmp_removal)

## How to apply

First run cellranger count on the 10x data so that possorted_genome_bam.bam and the filtered_feature_bc_matrix/raw_feature_bc_matrix folders exist. Prepare a -p2genome folder containing at most two files: the pathogen reference fasta (simplify the fasta header, e.g. > HIV_virus, since the header is used to quantify total pathogen reads) and an optional gtf for per-gene quantification. Invoke scpathoquant with -10x pointing at the cellranger output folder, -op for the results directory, -p for processor count (default 1), and choose -align bbmap (default; e.g. --bbmap_params="--semiperfectmode") or bowtie2 (e.g. --bowtie2_params="--very-sensitive --non-deterministic"). Internally the tool uses samtools to pull host-unmapped reads and their barcodes/UMIs into a _tmp/ directory, aligns them with the selected aligner, quantifies per-cell totals and per-gene counts with htseq (--intersection-nonempty), and overwrites the 10x matrix files with pathogen counts. Add --tmp_removal when disk space is an issue, or run the identical parameterization inside Docker by bind-mounting /app/input, /app/genome, and /app/output.

## Related tools

- **scPathoQuant** (Python package whose CLI parameters (-10x, -op, -p, -p2genome, -align, --tmp_removal) define the single-command pathogen alignment and quantification run) — https://github.com/galelab/scPathoQuant
- **cellranger** (Upstream prerequisite: aligns reads to the host genome and produces the 10x output folder supplied via -10x)
- **samtools** (Pulls host-unmapped reads from possorted_genome_bam.bam and records barcodes/UMIs into barcodes_umi_read_table.csv)
- **bbmap** (Default aligner for mapping extracted unmapped reads to the pathogen reference (parameterized via --bbmap_params))
- **bowtie2** (Alternative aligner selected with -align bowtie2 and tuned via --bowtie2_params (e.g. --very-sensitive --non-deterministic))
- **htseq** (Quantifies per-cell total and per-gene pathogen reads (htseq-count with --intersection-nonempty))
- **docker** (Alternative runtime via docker build -t scpathoquant, with bind mounts for input, genome, and output)
- **seurat** (Downstream validation/consumption of the pathogen-integrated filtered_feature_bc_matrix via Read10X)

## Examples

```
scpathoquant -10x path/to/10x/sample -op path/for/results -p 8 -p2genome path/to/pathogen/fastafilefolder -align bowtie2 --bowtie2_params="--very-sensitive --non-deterministic" --tmp_removal
```

## Evaluation signals

- pathogen_al_counts.csv exists and contains per-cell pathogen read counts keyed by cell barcodes.
- The overwritten filtered_feature_bc_matrix loads via Seurat Read10X with pathogen counts appended as extra features.
- pathogen_al_mapped_sort.bam and its .bai index are present and readable in IGV; pathogen_al_mapped.fq.gz contains only mapped pathogen reads.
- pathogen_genes_al_sort_counts.sam is produced only when a gtf was supplied via -p2genome.
- _tmp/ directory is absent (or absent after --tmp_removal) while the violin PNGs pathogen_copy.png, pathogen_genes.png, and coveragemap.png exist in -op.

## Limitations

- Temporary _tmp/ files can be large; --tmp_removal must be passed explicitly to delete them when space is an issue.
- A complicated FASTA header inflates/obscures total-read quantification — simplify it (e.g. > HIV_virus) because the header is used for quantification.
- htseq-count uses non-default --intersection-nonempty; per-gene outputs are skipped entirely if no pathogen gtf is provided.
- raw_feature_bc_matrix integration is 'not always needed'.

## Evidence

- [abstract] scPathoQuant extracts sequences that were not aligned to the primary host genome: "scPathoQuant extracts sequences that were not aligned to the primary host genome"
- [abstract] integrates pathogen counts into matrix files with one command: "integrates pathogen sequence counts into matrix files that are used by standard single cell pipelines for downstream analyses with only one command"
- [intro] conda install: "conda env create -f condaenv.yml -n scpathoquant"
- [intro] downstream Seurat analysis: "This allows softwares such as seurat to be used for easy analysis of the data"
- [methods] -10x parameter meaning: "-10x = Path/to/10x/sample/ (this the path to the output folder generated by first running cellranger count )"
- [methods] --tmp_removal flag: "--tmp_removal = if specified will remove the temporary directory (_tmp/) of files used by scpathoquant"
- [readme] fasta header guidance: "In the fasta file the header will be used to quantify the number of pathogen total reads, it is recommended that if the fasta header is a complicated name it be simplified"
- [readme] htseq non-default parameter: "non default htseq-count include using the --intersection-nonempty parameter"
- [readme] tmp folder contents: "barcodes_umi_read_table.csv - table of unmapped reads and corresponding barcodes and UMIs"
