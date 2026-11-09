---
name: single-cell-count-matrix-correlation-analysis
description: Use when an alternative scRNA-seq quantification workflow must be benchmarked
  against a standard pipeline like Cell Ranger by comparing gene-level UMI count matrices
  across the same cells and genes.
when_to_use_negative:
- The two count matrices quantify different feature sets (e.g. 3'UTR isoforms vs genes)
  or different cells, so shared-identifier alignment is impossible
- The goal is differential expression or 3'UTR length analysis rather than quantification-pipeline
  benchmarking
- Data are full-length or non-3'-end tag-based libraries that scUTRquant does not
  support
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0203
tools:
- name: scUTRquant
  role: Produces the alternative gene-level UMI count matrices from raw FASTQs via
    kallisto-bustools quantification on the truncated UTRome
  repo: https://github.com/Mayrlab/scUTRquant
- name: Snakemake
  role: Executes the scUTRquant pipeline with --use-conda --configfile to generate
    the SCE count objects being benchmarked
- name: Cell Ranger
  role: Supplies the reference gene count matrices (version 3.0.0 filtered HDF5 files)
    against which correlations are computed
- name: kallisto-bustools
  role: Underlying quantification engine of scUTRquant (kallisto bus pseudoalignment,
    bustools correct barcode correction)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:560202ae666dbef67c602600adafc0da93db35ab9d6e1043bfc0b99c84e58aa0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Mayrlab/scUTRquant
  tools:
  - Cell Ranger
  - Conda
  - Snakemake
  - kallisto-bustools
  - scUTRquant
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# single-cell-count-matrix-correlation-analysis

## Summary

Validates that a custom scRNA-seq quantification pipeline (scUTRquant on a truncated UTRome) produces gene counts concordant with a reference pipeline (Cell Ranger) by computing per-cell and per-gene Spearman rank correlations on raw UMI count matrices.

## When to use

Use when an alternative scRNA-seq quantification workflow must be benchmarked against a standard pipeline like Cell Ranger by comparing gene-level UMI count matrices across the same cells and genes.

## When NOT to use

- The two count matrices quantify different feature sets (e.g. 3'UTR isoforms vs genes) or different cells, so shared-identifier alignment is impossible
- The goal is differential expression or 3'UTR length analysis rather than quantification-pipeline benchmarking
- Data are full-length or non-3'-end tag-based libraries that scUTRquant does not support

## Inputs

- Gene-level SingleCellExperiment RDS objects from scUTRquant (data/sce/utrome_mm10_v2/)
- Cell Ranger 3.0.0 filtered HDF5 gene-barcode UMI count matrices
- Sample FASTQ files and sample_sheet.csv for the 10x Genomics demonstration datasets

## Outputs

- Per-sample table of per-cell and per-gene Spearman rho values (spearman_correlations.csv)
- Filtered raw UMI count matrices restricted to shared cells and genes

## How to apply

Run the alternative pipeline (scUTRquant with Snakemake --use-conda --configfile) on the same FASTQ inputs the reference pipeline processed, requesting output_type 'genes' against the 'utrome_mm10_v2' target with the '--fr-stranded' setting required for 10x 3'-end libraries, and collect gene-level SingleCellExperiment objects (RDS) from data/sce/utrome_mm10_v2/. Extract raw UMI count matrices from the reference Cell Ranger 3.0.0 filtered HDF5 files for the identical datasets. Restrict both matrices to the shared cell and gene identifiers, then compute per-cell and per-gene Spearman correlations sample-wise and tabulate rho values (e.g. spearman_correlations.csv). Agreement is judged against the manuscript's benchmarks: per-cell Spearman rho > 0.99 and per-gene Spearman rho around 0.92 on the 10x Genomics demonstration datasets.

## Related tools

- **scUTRquant** (Produces the alternative gene-level UMI count matrices from raw FASTQs via kallisto-bustools quantification on the truncated UTRome) — https://github.com/Mayrlab/scUTRquant
- **Snakemake** (Executes the scUTRquant pipeline with --use-conda --configfile to generate the SCE count objects being benchmarked)
- **Cell Ranger** (Supplies the reference gene count matrices (version 3.0.0 filtered HDF5 files) against which correlations are computed)
- **kallisto-bustools** (Underlying quantification engine of scUTRquant (kallisto bus pseudoalignment, bustools correct barcode correction))

## Examples

```
snakemake --use-conda --configfile examples/<dataset>/config.yaml
```

## Evaluation signals

- Per-cell Spearman rho between scUTRquant and Cell Ranger UMI counts exceeds 0.99 for each 10x sample
- Per-gene Spearman rho between the two pipelines is approximately 0.92
- Both count matrices align on the same cell barcodes and gene identifiers before correlation is computed
- scUTRquant run completed through data/sce/utrome_mm10_v2/ with HTML QC reports and BUS/MTX intermediates retained
- Results written to spearman_correlations.csv cover all six samples

## Limitations

- Per-gene correlation (~0.92) is lower than per-cell correlation (>0.99), so gene-level discrepancies are expected and acceptable
- Benchmarks are established on 10x 3'-end demonstration datasets; other chemistries (10xv2 vs 10xv3) require matching tech configuration
- scUTRquant requires '--fr-stranded' for all 10X 3'-end libraries; omitting strand settings will invalidate counts

## Evidence

- [other] Spearman correlations exceed 0.99 for UMI counts per cell and reach 0.92 for UMI counts per gene: [evidence span withheld — anchor status: not_found]
- [methods] Extract the raw UMI count matrices from the Cell Ranger 3.0.0 filtered HDF5 files: [evidence span withheld — anchor status: not_found]
- [methods] strand '--fr-stranded' (required for all 10X 3'-end libraries): [evidence span withheld — anchor status: not_found]
- [discussion] Builds on kallisto-bustools for compatible 3'-end quantification: "By building on the kallisto-bustools workflow, we provide a fast and direct means of quantification that is compatible with most 3′-end tag scRNA-seq libraries"
