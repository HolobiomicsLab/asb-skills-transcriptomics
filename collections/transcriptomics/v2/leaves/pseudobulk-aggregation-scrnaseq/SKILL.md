---
name: pseudobulk-aggregation-scrnaseq
description: Use when raw scRNA-seq counts plus per-cell metadata (cell, individual,
  cell type) must be summarized to individual-by-cell-type pseudobulk expression before
  fitting LMMs such as CTMM's Free model on CTP data.
when_to_use_negative:
- 'The target cell types are rare: CTMM requires >10 cells per individual-cell type
  pair, limiting aggregation to common cell types'
- Genes are very lowly expressed, since lowly-expressed genes can be severely non-Gaussian
  and would need many more cells for a valid Gaussian approximation
- The analysis needs within-cell-type, cell-level resolution, since pseudobulk inherently
  discards cell-level information
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ctmm
  role: Python package whose preprocess module (preprocess.pseudobulk, preprocess.std)
    performs the aggregation and standardization
  repo: https://github.com/Minhui-Chen/ctmm_python
- name: Python
  role: Execution environment for calling the ctmm preprocess functions on counts
    and metadata
- name: rpy2
  role: Bridge used by CTMM routines downstream of aggregation; a working R installation
    is required
  repo: https://rpy2.github.io/
- name: R
  role: Runtime required by CTMM through rpy2 for the fitting routines that consume
    the aggregated pseudobulk files
provenance:
  source_task_ids:
  - task_006
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-49242-9
    title: A robust model for cell type-specific interindividual variation in single-cell
      RNA sequencing data
  source_content_hash: sha256:10f9ba8e6abc40be7775d1a650abbd2a334704537fe2cd72230b85adfaf28e7d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Minhui-Chen/ctmm_python
  tools:
  - Python
  - R
  - ctmm
  - rpy2
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# pseudobulk-aggregation-scRNAseq

## Summary

Aggregate cell-level scRNA-seq counts into cell type-specific pseudobulk (CTP), cell type-specific noise variances (ctnu), and cell type proportions (P) using the ctmm preprocess module, so Gaussian linear mixed models like CTMM can be fit to interindividual variation.

## When to use

Use when raw scRNA-seq counts plus per-cell metadata (cell, individual, cell type) must be summarized to individual-by-cell-type pseudobulk expression before fitting LMMs such as CTMM's Free model on CTP data.

## When NOT to use

- The target cell types are rare: CTMM requires >10 cells per individual-cell type pair, limiting aggregation to common cell types
- Genes are very lowly expressed, since lowly-expressed genes can be severely non-Gaussian and would need many more cells for a valid Gaussian approximation
- The analysis needs within-cell-type, cell-level resolution, since pseudobulk inherently discards cell-level information

## Inputs

- Cell-level scRNA-seq gene-by-cell counts matrix (quality-controlled and normalized)
- Per-cell metadata table with cell ID, individual ID (ind), and cell type label (ct)
- Filter parameters ind_cut (e.g. 100) and ct_cut (e.g. 10)

## Outputs

- CTP: cell type-specific pseudobulk expression, one column per cell type, no header
- ctnu: variance of measurement noise for each individual-cell type pair, one column per cell type
- P: cell type proportions for each individual, one column per cell type
- Optionally standardized OP, nu, CTP, ctnu via preprocess.std

## How to apply

Load the gene-by-cell counts matrix and a metadata table with columns for cell, individual (ind), and cell type (ct), then call ctmm's preprocess.pseudobulk(counts=counts, meta=meta, ind_cut=100, ct_cut=10). The ind_cut=100 filter removes individuals with <=100 cells and ct_cut=10 sets CTP and ctnu to missing for individual-cell type pairs with <=10 cells, which is required because the Gaussian approximation underlying CTMM needs >10 cells per pair. The function returns three objects: CTP (cell type-specific pseudobulk expression, one column per cell type), ctnu (measurement noise variance per individual-cell type pair), and P (cell type proportions per individual). Restrict to individual-cell type pairs passing the cell-count threshold, and optionally apply preprocess.std to rescale CTP/ctnu/P so overall pseudobulk expression has mean 0 and variance 1; remove individuals or cell types so all genes are expressed in all cell types. For OP analyses, the same module produces OP and nu instead of CTP and ctnu.

## Related tools

- **ctmm** (Python package whose preprocess module (preprocess.pseudobulk, preprocess.std) performs the aggregation and standardization) — https://github.com/Minhui-Chen/ctmm_python
- **Python** (Execution environment for calling the ctmm preprocess functions on counts and metadata)
- **rpy2** (Bridge used by CTMM routines downstream of aggregation; a working R installation is required) — https://rpy2.github.io/
- **R** (Runtime required by CTMM through rpy2 for the fitting routines that consume the aggregated pseudobulk files)

## Examples

```
from ctmm import preprocess; CTP, ctnu, P = preprocess.pseudobulk(counts=counts, meta=meta, ind_cut=100, ct_cut=10)
```

## Evaluation signals

- Individual-cell type pairs with <=10 cells must have missing (NaN) CTP and ctnu values after preprocessing
- Individuals with <=100 total cells should be dropped when ind_cut=100 is used
- All genes retained must be expressed in all cell types (no missing pseudobulk entries due to zero expression)
- After preprocess.std, overall pseudobulk expression should have mean 0 and variance 1
- On OneK1K PBMC data the pipeline should retain the 7 primary PBMC cell types and 11,526 testable genes

## Limitations

- Requiring >10 cells per individual-cell type pair limits the analysis to common cell types
- Pseudobulk aggregation inherently discards cell-level information, sacrificing statistical power and resolution within cell types
- Lowly-expressed genes can be severely non-Gaussian, increasing the number of cells needed for the Gaussian approximation
- When cell types are inferred directly from the same scRNA-seq data (as in the PBMC analysis), some circularity is induced that deflates cell type-specific variance estimates

## Evidence

- [methods] compute ctp (cell type-specific pseudobulk) and ctnu (cell type-specific noise variance) and P (cell type proportions): "compute ctp (cell type-specific pseudobulk) and ctnu (cell type-specific noise variance) and P (cell type proportions)"
- [methods] remove individuals with <= 100 cells (ind_cut=100): "preprocess.pseudobulk(counts=counts, meta=meta, ind_cut=100, ct_cut=10) # remove individuals with <= 100 cells"
- [discussion] In practice, we require >10 cells per individual-cell type pair, which limits CTMM to common cell types.: "In practice, we require >10 cells per individual-cell type pair, which limits CTMM to common cell types."
- [discussion] The key idea is to summarize the scRNA-seq data into cell type-specific pseudobulk: "The key idea is to summarize the scRNA-seq data into cell type-specific pseudobulk"
- [readme] CTP: Cell Type-specific Pseudobulk gene expression for each individual.: "CTP: Cell Type-specific Pseudobulk gene expression for each individual. This file should have one column for each cell type and without header."
