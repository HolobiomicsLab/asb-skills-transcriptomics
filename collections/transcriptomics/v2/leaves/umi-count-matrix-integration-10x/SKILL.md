---
name: umi-count-matrix-integration-10x
description: Use when you have a cellranger count output folder for a 10x scRNAseq
  sample (e.g. HIV donor or SARS-CoV-2 airway data) and need pathogen UMI counts merged
  into features.tsv.gz/matrix.mtx.gz so Seurat or standard single-cell pipelines can
  analyze infected cells downstream.
when_to_use_negative:
- Input is raw FASTQ/SRA data rather than an existing cellranger count output folder
  — cellranger count must be run against the host genome first since -10x points at
  that output folder
- The sample was already processed against a host+pathogen integrated reference (e.g.
  CellRanger on hg38 with the HIV genome integrated), so per-cell pathogen UMI counts
  already exist in the 10x matrices and no host-then-pathogen remapping/integration
  is needed
- Pristine cellranger output must be preserved — scPathoQuant overwrites the original
  10x data provided to include pathogen counts, so work on a copy if unmodified matrices
  are required
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scPathoQuant
  role: 'Performs the integration: extracts host-unaligned reads, remaps them to the
    pathogen genome, and merges pathogen UMI and gene UMI counts into the cellranger
    10x matrix files'
  repo: https://github.com/galelab/scPathoQuant
- name: cellranger
  role: Produces the 10x sample output folder (host-aligned BAM and feature/barcode/matrix
    files) that the integration consumes and overwrites
  repo: https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/latest/using/tutorial_ct
- name: bbmap
  role: Default aligner used to remap host-unaligned reads to the pathogen genome
    before counts are integrated into the 10x matrices
- name: bowtie2
  role: Alternative aligner selectable via -align bowtie2 (e.g. with --very-sensitive
    --non-deterministic) when comparing alignment strategies during integration
- name: htseq
  role: htseq-count quantifies pathogen reads (with the non-default --intersection-nonempty
    parameter) that are written into the integrated 10x matrices
- name: seurat
  role: Downstream consumer that reads the pathogen-integrated filtered_feature_bc_matrix
    via Read10X for further analysis
provenance:
  source_task_ids:
  - task_007
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae145
    title: 'scPathoQuant: a tool for efficient alignment and quantification of pathogen
      sequence reads from 10× single cell sequencing datasets'
  source_content_hash: sha256:8a2b30f5480a18d32eee672e0de108f2b78d008449f33be8ad6803655fe15742
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/galelab/scPathoQuant
  tools:
  - bbmap
  - bowtie2
  - cellranger
  - htseq
  - scPathoQuant
  - seurat
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# umi-count-matrix-integration-10x

## Summary

Integrates per-cell pathogen UMI counts and pathogen gene UMI counts, computed by scPathoQuant from host-unaligned reads, directly into the 10x Genomics cellranger count matrices (filtered_feature_bc_matrix / raw_feature_bc_matrix) so a single command yields Seurat-ready, pathogen-annotated count matrices.

## When to use

Use when you have a cellranger count output folder for a 10x scRNAseq sample (e.g. HIV donor or SARS-CoV-2 airway data) and need pathogen UMI counts merged into features.tsv.gz/matrix.mtx.gz so Seurat or standard single-cell pipelines can analyze infected cells downstream.

## When NOT to use

- Input is raw FASTQ/SRA data rather than an existing cellranger count output folder — cellranger count must be run against the host genome first since -10x points at that output folder
- The sample was already processed against a host+pathogen integrated reference (e.g. CellRanger on hg38 with the HIV genome integrated), so per-cell pathogen UMI counts already exist in the 10x matrices and no host-then-pathogen remapping/integration is needed
- Pristine cellranger output must be preserved — scPathoQuant overwrites the original 10x data provided to include pathogen counts, so work on a copy if unmodified matrices are required

## Inputs

- 10x cellranger count output folder for the sample (containing filtered_feature_bc_matrix/raw_feature_bc_matrix with features.tsv.gz, matrix.mtx.gz, barcodes.tsv.gz, and possorted_genome_bam.bam)
- Pathogen reference folder with the pathogen genome FASTA (simplified header, e.g. >HIV_virus) and optionally a pathogen GTF (e.g. SARS-CoV-2 genome/GTF from NCBI GCF_009858895.2)
- Run parameters: -op results path, -p processor count (e.g. 8), -align bbmap (default) or bowtie2 with --bbmap_params/--bowtie2_params overrides

## Outputs

- Overwritten 10x matrices: filtered_feature_bc_matrix and raw_feature_bc_matrix (features.tsv.gz, matrix.mtx.gz) including pathogen UMI counts and pathogen gene UMI counts, loadable via Seurat Read10X
- pathogen_al_counts.csv — total number of reads mapping to the pathogen in each cell
- pathogen_al_gene_counts.csv — per-cell pathogen gene UMI counts (produced only if a pathogen GTF was provided)
- pathogen_al.bam, pathogen_al_mapped_sort.bam(.bai), pathogen_al_mapped.fq.gz, and htseq outputs pathogen_al_sort_counts.sam / pathogen_genes_al_sort_counts.sam

## How to apply

Run cellranger count against the host genome (e.g. GRCh38/hg38) first; the -10x argument must point at that output folder because scPathoQuant pulls reads left unaligned to the host from possorted_genome_bam.bam. Point -p2genome at a folder holding the pathogen FASTA plus an optional pathogen GTF, simplifying the FASTA header (e.g. >HIV_virus) since the header names the quantified total pathogen reads; per-gene counts are only produced when a GTF is supplied. Invoke scpathoquant with -op for the results path and -p for processors (e.g. -p 8); the default aligner is bbmap, with samtools handling BAMs and htseq quantifying counts using the non-default --intersection-nonempty parameter, or rerun with -align bowtie2 and --bowtie2_params set to --very-sensitive --non-deterministic to compare alignment strategies. scPathoQuant then integrates pathogen UMI counts and pathogen UMI gene counts back into the cellranger files, overwriting the original 10x data so filtered_feature_bc_matrix and raw_feature_bc_matrix carry both host and pathogen features. Add --tmp_removal when the large _tmp/ intermediates (unmmaped.bam, barcodes_umi_read_table.csv, unmapped.fq.gz) should be deleted, and confirm integration by loading the matrix with Seurat's Read10X.

## Related tools

- **scPathoQuant** (Performs the integration: extracts host-unaligned reads, remaps them to the pathogen genome, and merges pathogen UMI and gene UMI counts into the cellranger 10x matrix files) — https://github.com/galelab/scPathoQuant
- **cellranger** (Produces the 10x sample output folder (host-aligned BAM and feature/barcode/matrix files) that the integration consumes and overwrites) — https://support.10xgenomics.com/single-cell-gene-expression/software/pipelines/latest/using/tutorial_ct
- **bbmap** (Default aligner used to remap host-unaligned reads to the pathogen genome before counts are integrated into the 10x matrices)
- **bowtie2** (Alternative aligner selectable via -align bowtie2 (e.g. with --very-sensitive --non-deterministic) when comparing alignment strategies during integration)
- **htseq** (htseq-count quantifies pathogen reads (with the non-default --intersection-nonempty parameter) that are written into the integrated 10x matrices)
- **seurat** (Downstream consumer that reads the pathogen-integrated filtered_feature_bc_matrix via Read10X for further analysis)

## Examples

```
scpathoquant -10x path/to/10x/sample -op path/for/results -p 8 -p2genome path/to/pathogen/fastafilefolder
```

## Evaluation signals

- filtered_feature_bc_matrix/features.tsv.gz contains a pathogen feature row named by the simplified FASTA header (e.g. HIV_virus) plus per-gene pathogen rows when a GTF was provided, and the matrix loads cleanly with Seurat Read10X(data.dir='output_scPathoQuant/filtered_feature_bc_matrix')
- Per-cell totals in pathogen_al_counts.csv agree with the pathogen UMI feature values in the overwritten matrix.mtx.gz for the same cell barcodes
- pathogen_al_gene_counts.csv and pathogen_genes_al_sort_counts.sam exist only when a pathogen GTF was supplied; with FASTA-only input expect total per-cell counts but no gene-level output
- pathogen_copy.png and pathogen_genes.png violin plots show nonzero per-cell pathogen UMI distributions consistent with pathogen_al_counts.csv
- Original host feature rows from cellranger are preserved alongside the newly added pathogen rows (integration into, not replacement of, the 10x matrix)

## Limitations

- scPathoQuant overwrites the original 10x data provided to include pathogen counts, so the pristine cellranger output is not retained unless copied beforehand
- Pathogen gene-level counts are not produced if no pathogen GTF is provided; only total per-cell pathogen counts keyed by the FASTA header are integrated
- The _tmp/ working directory (unmmaped.bam, unmmaped.sam, barcodes_umi_read_table.csv, unmapped.fq.gz) can be large; without --tmp_removal it is left in place and may fill disk
- The FASTA header is used to quantify total pathogen reads, so a complicated header name propagates into the outputs — the README recommends simplifying it (e.g. >HIV_virus)

## Evidence

- [abstract] Pathogen counts are integrated into matrix files usable by standard single-cell pipelines: "integrates pathogen sequence counts into matrix files that are used by standard single cell pipelines for downstream analyses"
- [intro] Pathogen UMI and gene UMI counts are merged into the 10x files generated by cellranger count: "This software integrates pathogen UMI counts and pathogen UMI gene counts into 10x files generated by first running [cellranger count]"
- [readme] Unaligned host reads are remapped to the pathogen and written back into the cellranger files: "scPathoQuant then takes the unaligned reads from the host genome and maps them to your pathogen genome of interest and then integrates them back into the cellranger files."
- [readme] Integration overwrites the original 10x data; gene-level rows depend on providing a GTF: "Overwrites original 10x data provided to include pathogen counts and pathogen gene counts (if gtf file is provided)"
- [readme] The integrated filtered_feature_bc_matrix is designed for direct Seurat loading: "filtered_feature_bc_matrix - files that can be integrated into seurat with pathogen counts"
