---
name: r-bioconductor-package-execution
description: Use when a task requires installing an R/Bioconductor package and reproducing
  its quick-start example, e.g. scoring gene signatures with UCell's ScoreSignatures_UCell
  on a bundled sample matrix.
when_to_use_negative:
- The input is already a table of signature scores or a feature/abundance table rather
  than a gene-by-cell expression matrix
- The task requires dataset-level differential expression modeling rather than per-cell
  signature enrichment scoring
- A Python-native pipeline is mandated and R is unavailable — note carmonalab/pyucell
  exists but this skill covers the R/Bioconductor execution path
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: UCell
  role: Bioconductor R package being executed; ScoreSignatures_UCell computes Mann-Whitney
    U-based signature scores from an expression matrix and gene set list
  repo: https://github.com/carmonalab/UCell
- name: data.table
  role: Provides the frank function UCell uses internally for efficient gene rank
    computations
- name: SingleCellExperiment
  role: Supported Bioconductor container for the gene-vs-cell matrix input to UCell
    scoring functions
- name: future
  role: Enables parallel batch processing of large datasets to minimize execution
    time
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2021.06.043
    title: 'UCell: Robust and scalable single-cell gene signature scoring'
  source_content_hash: sha256:38a227f8b8665ff8626eef61f3fbadb7a2e8520288dae45c9c9bf924bd44df1a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/carmonalab/UCell
  tools:
  - R
  - SingleCellExperiment
  - UCell
  - data.table
  - future
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# r-bioconductor-package-execution

## Summary

Install and run an R/Bioconductor package (here UCell) to execute its documented quick-start workflow on bundled sample data, from BiocManager installation through function calls on in-memory data objects to per-cell score tables.

## When to use

Use when a task requires installing an R/Bioconductor package and reproducing its quick-start example, e.g. scoring gene signatures with UCell's ScoreSignatures_UCell on a bundled sample matrix.

## When NOT to use

- The input is already a table of signature scores or a feature/abundance table rather than a gene-by-cell expression matrix
- The task requires dataset-level differential expression modeling rather than per-cell signature enrichment scoring
- A Python-native pipeline is mandated and R is unavailable — note carmonalab/pyucell exists but this skill covers the R/Bioconductor execution path

## Inputs

- Gene-vs-cell expression matrix (e.g. the bundled sample.matrix from UCell), as dense matrix, sparse matrix, SingleCellExperiment, or Seurat object
- Named list of gene sets (features), e.g. list(Tcell_signature = c('CD2','CD3E','CD3D'), Myeloid_signature = c('SPI1','FCER1G','CSF1R'))

## Outputs

- Per-cell signature score table: one row per cell, one column per gene set (e.g. Tcell_signature, Myeloid_signature), written to ucell_scores.csv

## How to apply

Install the package from Bioconductor with BiocManager::install('UCell'), falling back to remotes::install_github('carmonalab/UCell', ref='v2.2') for a tagged release. Load the bundled sample data (e.g. data(sample.matrix)) and define the gene.sets list exactly as in the vignette — Tcell_signature = c('CD2','CD3E','CD3D') and Myeloid_signature = c('SPI1','FCER1G','CSF1R'). Run scores <- ScoreSignatures_UCell(sample.matrix, features=gene.sets), which internally ranks genes per cell using the data.table frank function, caps ranks at rmax = 1500 to truncate the uninformative tail of zero-expression genes, and computes scores via the Mann-Whitney U statistic normalized as 1 - U/U_max. Inspect head(scores) and persist the full table with write.csv to ucell_scores.csv. The scoring depends only on relative gene ranks within individual cells, so results are robust to dataset composition.

## Related tools

- **UCell** (Bioconductor R package being executed; ScoreSignatures_UCell computes Mann-Whitney U-based signature scores from an expression matrix and gene set list) — https://github.com/carmonalab/UCell
- **data.table** (Provides the frank function UCell uses internally for efficient gene rank computations)
- **SingleCellExperiment** (Supported Bioconductor container for the gene-vs-cell matrix input to UCell scoring functions)
- **future** (Enables parallel batch processing of large datasets to minimize execution time)

## Examples

```
library(UCell); data(sample.matrix); gene.sets <- list(Tcell_signature=c('CD2','CD3E','CD3D'), Myeloid_signature=c('SPI1','FCER1G','CSF1R')); scores <- ScoreSignatures_UCell(sample.matrix, features=gene.sets)
```

## Evaluation signals

- Output table schema: exactly one row per cell in sample.matrix and one column per signature (Tcell_signature, Myeloid_signature)
- All scores are normalized U statistics equivalent to AUC and must fall in the range 0-1
- head(scores) matches the quick-start vignette output for the bundled sample.matrix and default rmax = 1500
- uCell version >= 2.7.6 uses normalization 1 - U/U_max with U_max = n*maxRank - n(n+1)/2; verify scores are consistent with this formula rather than the older n*maxRank normalization
- Package loads cleanly after BiocManager::install('UCell') and ?UCell help is accessible

## Limitations

- Rank cap rmax = 1500 truncates the uninformative tail of zero-expression genes; changing maxRank alters scores
- Since UCell >= 2.7.6 the U_max normalization changed (U_max = n*maxRank - n(n+1)/2); results differ slightly from earlier implementations, particularly for large gene sets
- Single-cell data are sparse; SmoothKNN k-nearest-neighbor smoothing may be needed to impute scores and partially correct sparsity
- Very large datasets (>10^5 cells) are split into batches processed serially or in parallel; serial processing minimizes memory but increases runtime

## Evidence

- [readme] Load sample data and test your installation: "Load sample data and test your installation:
r
library(UCell)

data(sample.matrix)
gene.sets <- list(Tcell_signature = c("CD2","CD3E","CD3D"),"
- [readme] BiocManager installation: "**[UCell is on Bioconductor](https://bioconductor.org/packages/release/bioc/html/UCell.html)** To install the package from Bioc run:
r
if (!requireNamespace("BiocManager", quietly=TRUE))"
- [methods] Mann-Whitney U statistic and rank robustness: "This score depends only on the gene expression ranks of individual cell, and therefore is robust across datasets."
- [methods] rmax rank capping default: "we set rg,c = rmax + 1 for all rg,c > rmax, with rmax = 1500 by default"
- [methods] data.table frank usage: "Internally, UCell uses the frank function from the data.table package [9] for efficient ranks computations."
- [readme] New normalization formula: "${UCell} = 1 - {U} / {U_{max}}$

where

$U = \sum_{i=1}^{n} r_i - \frac{n(n+1)}{2}$"
