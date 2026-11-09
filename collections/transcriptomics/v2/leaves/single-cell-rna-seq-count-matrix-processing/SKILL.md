---
name: single-cell-rna-seq-count-matrix-processing
description: Use when starting from a raw scRNA-seq read count matrix (Rds/CSV of
  genes x cells, e.g., Fluidigm C1 alpha cells) that must be gene-filtered, covariate-annotated,
  and formatted before surrogate-variable or marker-gene analysis.
when_to_use_negative:
- The input is already a filtered/normalized feature table or a marker-gene list rather
  than a raw per-cell read count matrix
- The goal is differential expression between known groups rather than detecting hidden
  sources of variation via surrogate variables
- Genes have not passed any expression filter and the analysis plan calls for expression-informative
  gene selection only after IA-SVA-based feature selection
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: fast_iasva
  role: Fast IA-SVA implementation that consumes the processed count matrix and selects
    important SVs based on the proportion of variation explained; its SV output is
    then used by find_markers to derive marker genes
  repo: github.com/UcarLab/iasva
- name: SCnorm
  role: Normalization of the filtered read counts (loge(1 + SCnorm-normalized counts))
    prior to visualization/analysis
- name: SCRAN
  role: R package used to compute cell-cycle-stage predictions from the processed
    matrix to interpret SV values
- name: Seurat
  role: Spectral tSNE visualization of single cells after count-matrix processing
    and IA-SVA-based feature selection
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-018-35365-9
    title: Detection of correlated hidden factors from single cell transcriptomes
      using Iteratively Adjusted-SVA (IA-SVA)
  source_content_hash: sha256:8d9ddd45d21580d20c21dee4786b2fbfd52f780c7c46cee91babeee33e39b52f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - SCRAN
  - SCnorm
  - Seurat
  - fast_iasva
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# single-cell-rna-seq-count-matrix-processing

## Summary

Preprocesses single-cell RNA-seq read count matrices (e.g., the alpha-cell iasva_counts_test.Rds matrix of n = 101 cells) by filtering to adequately expressed genes and preparing normalized counts and covariates so that hidden-factor methods such as IA-SVA and marker-gene regression (find_markers) can be applied.

## When to use

Use when starting from a raw scRNA-seq read count matrix (Rds/CSV of genes x cells, e.g., Fluidigm C1 alpha cells) that must be gene-filtered, covariate-annotated, and formatted before surrogate-variable or marker-gene analysis.

## When NOT to use

- The input is already a filtered/normalized feature table or a marker-gene list rather than a raw per-cell read count matrix
- The goal is differential expression between known groups rather than detecting hidden sources of variation via surrogate variables
- Genes have not passed any expression filter and the analysis plan calls for expression-informative gene selection only after IA-SVA-based feature selection

## Inputs

- scRNA-seq read count matrix (genes x cells) in Rds format, e.g., iasva_counts_test.Rds (human pancreatic islet alpha cells, n = 101, Fluidigm C1 platform)
- Cell-level covariate table (e.g., patient identity, geometric library size)
- Sample/gene annotation (e.g., GCG expression marking alpha cells)

## Outputs

- Filtered count matrix restricted to adequately expressed genes (>5 reads in >=3 cells)
- Normalized expression matrix (e.g., SCnorm loge(1 + counts)) for downstream visualization
- Covariate matrix used as known factors in IA-SVA
- Marker-gene count matrix/CSV (e.g., sv1_marker_genes.csv) containing genes with R-squared > 0.3 and adjusted p-value < 0.05

## How to apply

Load the read count matrix (e.g., iasva_counts_test.Rds, alpha cells from healthy non-diabetic subjects, n = 101) and restrict genes to those adequately expressed, using the paper's cutoffs (>5 reads in at least 3 cells, or read count >3 in at least 3 cells for downsampling scenarios). Prepare the covariate table with the known factors to adjust for, such as patient identity and geometric library size, since IA-SVA regresses out known factors before estimating hidden ones. Optionally normalize counts (e.g., SCnorm loge(1 + normalized counts)) before visualization. The processed matrix and covariates then feed directly into IA-SVA (or fast_iasva) and find_markers, which regresses each gene on the surrogate variable and retains markers with adjusted p-value < 0.05 and R-squared > 0.3.

## Related tools

- **fast_iasva** (Fast IA-SVA implementation that consumes the processed count matrix and selects important SVs based on the proportion of variation explained; its SV output is then used by find_markers to derive marker genes) — github.com/UcarLab/iasva
- **SCnorm** (Normalization of the filtered read counts (loge(1 + SCnorm-normalized counts)) prior to visualization/analysis)
- **SCRAN** (R package used to compute cell-cycle-stage predictions from the processed matrix to interpret SV values)
- **Seurat** (Spectral tSNE visualization of single cells after count-matrix processing and IA-SVA-based feature selection)

## Examples

```
library(iasva); counts <- readRDS('iasva_counts_test.Rds'); sv1_markers <- find_markers(counts, obs_returns, sv = SV1, cutoff_r2 = 0.3, cutoff_padj = 0.05); write.csv(sv1_markers, 'sv1_marker_genes.csv')
```

## Evaluation signals

- Output marker table exists and contains only genes with R-squared > 0.3 and multiple-testing adjusted p-value < 0.05 for the surrogate variable
- Every retained gene satisfies the expression filter (>5 reads in at least 3 cells, i.e., above five in at least three cells)
- Number of cells in the processed matrix matches the source dataset (e.g., n = 101 alpha cells from healthy non-diabetic subjects)
- Covariate table includes patient identity and geometric library size, matching the paper's known-factor specification
- Surrogate-variable values computed on the processed matrix correlate with the expected hidden factor (e.g., SV1 values differ by cell-cycle stage per SCRAN predictions)

## Limitations

- Compromised/stacked doublet alpha cells detected by IA-SVA should be excluded from downstream analyses, as the heterogeneity may reflect cell contamination
- The permutation-based significance index (SI) is not an empirical p-value, and no specific SI cutoff threshold is provided; a stringent cutoff is recommended when many unknown factors are expected
- No software versions are stated for the named tools (Seurat, ZINB-WaVe, Cell View, tSNE), affecting reproducibility
- The software is provided 'as is' without warranty and only for academic research, not commercial use

## Evidence

- [methods] genes with >5 reads in at least 3 cells: "we used genes with >5 reads in at least 3 cells (i.e., a gene has to be above five in at least three cells to be kept)"
- [methods] find_markers cutoffs: "marker genes that are significantly associated with SV1 (multiple testing adjusted p-value < 0.05, default significance cutoff, and R-squared value > 0.3"
- [results] covariates: [evidence span withheld — anchor status: not_found]
- [methods] input dataset: "we use read counts of alpha cells from healthy (non-diabetic) subjects (n = 101)"
- [discussion] exclusion of compromised cells: "IA-SVA detects heterogeneity stemming from compromised alpha cells (which may be explained by cell contamination), which should be excluded from the downstream analyses"
