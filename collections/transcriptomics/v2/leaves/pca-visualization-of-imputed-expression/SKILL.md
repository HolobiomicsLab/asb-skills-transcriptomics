---
name: pca-visualization-of-imputed-expression
description: Use when comparing raw vs. imputed count matrices (Bfimpute, SCRABBLE,
  netNMF-sc, etc.) and you need a visualization-level and cell-relationship-level
  assessment of whether imputation improved cell-type separation.
when_to_use_negative:
- Input is already a dimensionality-reduced embedding or a normalized log-expression
  matrix prepared with a different scheme than the imputation pipeline — inconsistent
  normalization (e.g., skipping CPM + log10 with bias 1.01) invalidates cross-method
  comparison
- The downstream goal is differential expression or pseudotime rather than assessing
  cell-to-cell relationship recovery — the discussion notes no single imputation method
  wins across all downstream analyses
- The dataset has no cell type labels or clustering available, so the cell-to-cell
  correlation against labeled types cannot be computed
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Bfimpute
  role: Produces the imputed matrices (with or without S_G bulk side information)
    that are projected into PCA space and scored by cell-to-cell correlations
  repo: https://github.com/maiziezhoulab/Bfimpute
- name: Spectrum
  role: Spectral clustering of cells into 7 groups so Bfimpute imputes within cell
    groups before PCA visualization
- name: SCRABBLE
  role: Bulk-constrained comparator imputation method evaluated alongside Bfimpute
    in the PCA/correlation comparison
- name: netNMF-sc
  role: Gene-gene-interaction-based comparator imputation method evaluated alongside
    Bfimpute in the PCA/correlation comparison
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100133
    title: A Bayesian factorization method to recover single-cell RNA sequencing data
  source_content_hash: sha256:718b4ca4debb0e63adc6203715bcf9e7a4aa3ab1ba33ee7b692b467e6c255b50
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/maiziezhoulab/Bfimpute
  repo_url: https://github.com/maiziezhoulab/Bfimpute
  tools:
  - Bfimpute
  - DrImpute
  - MAGIC
  - PBLR
  - SAVER
  - SCRABBLE
  - Spectrum
  - VIPER
  - netNMF-sc
  - scImpute
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9333333333333333
n_claims: 15
---

# pca-visualization-of-imputed-expression

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Projects raw and imputed scRNA-seq expression matrices (e.g., Bfimpute outputs on the Chu et al. 2016 human-ESC dataset) into PCA space and computes cell-to-cell correlations against labeled cell types to judge whether imputation recovers cell-to-cell relationships, reproducing the S1I-S1J comparison pattern.

## When to use

Use when comparing raw vs. imputed count matrices (Bfimpute, SCRABBLE, netNMF-sc, etc.) and you need a visualization-level and cell-relationship-level assessment of whether imputation improved cell-type separation.

## When NOT to use

- Input is already a dimensionality-reduced embedding or a normalized log-expression matrix prepared with a different scheme than the imputation pipeline — inconsistent normalization (e.g., skipping CPM + log10 with bias 1.01) invalidates cross-method comparison
- The downstream goal is differential expression or pseudotime rather than assessing cell-to-cell relationship recovery — the discussion notes no single imputation method wins across all downstream analyses
- The dataset has no cell type labels or clustering available, so the cell-to-cell correlation against labeled types cannot be computed

## Inputs

- QC-passed scRNA-seq count matrix (sc_qc.csv, genes x cells)
- Imputed expression matrices from Bfimpute (Bfimpute.csv/.txt/.rds) and comparator methods (SCRABBLE, netNMF-sc, etc.)
- Cell type label file (cell_type.csv) for the Chu human-ESC dataset
- Bulk RNA-seq matrix (bulk_qc.csv) if testing gene side-information S_G = t(bulk)

## Outputs

- PCA scatter plots of raw and each imputed matrix colored by cell type
- Cell-to-cell correlation summaries against labeled cell types
- Comparison figure reproducing the S1I-S1J pattern (PCA and correlation panels)

## How to apply

Run the candidate imputation methods on the same QC-passed count matrix — e.g., Bfimpute with ccluster=c(7,'Spectrum') and optionally S_G=t(bulk) to test bulk-data integration, alongside bulk-constrained SCRABBLE and gene-gene-interaction-based netNMF-sc. Project every resulting matrix (and the raw counts) into PCA space using cell-type labels for coloring, then compute cell-to-cell correlations within and across labeled cell types. Compare the PCA separation and correlation structure across methods to determine whether a method that integrates extra information (bulk data, gene-gene interactions) actually improves cell-to-cell relationship recovery. On the Chu dataset the expected outcome is that adding S_G to Bfimpute yields no significant performance difference and SCRABBLE/netNMF-sc do not outperform Bfimpute. Note that Bfimpute applies CPM normalization and log10 transform with bias 1.01 when normalized=FALSE, so keep normalization consistent across methods before PCA.

## Related tools

- **Bfimpute** (Produces the imputed matrices (with or without S_G bulk side information) that are projected into PCA space and scored by cell-to-cell correlations) — https://github.com/maiziezhoulab/Bfimpute
- **Spectrum** (Spectral clustering of cells into 7 groups so Bfimpute imputes within cell groups before PCA visualization)
- **SCRABBLE** (Bulk-constrained comparator imputation method evaluated alongside Bfimpute in the PCA/correlation comparison)
- **netNMF-sc** (Gene-gene-interaction-based comparator imputation method evaluated alongside Bfimpute in the PCA/correlation comparison)

## Examples

```
p <- prcomp(t(counts_imputed), scale. = TRUE); plot(p$x[,1], p$x[,2], col = as.numeric(cell_type$CellType))
```

## Evaluation signals

- PCA embeddings of imputed matrices should show recovery of the labeled Chu human-ESC cell populations relative to raw counts
- Adding bulk data via S_G = t(bulk) to Bfimpute should yield no significant performance difference versus Bfimpute without S_G (Figures S1I-S1J)
- SCRABBLE and netNMF-sc should not outperform Bfimpute in the PCA/correlation comparison (Figures S1I-S1J)
- Cell-to-cell correlation structure should improve on imputed matrices relative to raw, consistent with recovering cell-to-cell relationships
- Saved outputs must include all three Bfimpute file types (Bfimpute.csv, Bfimpute.txt, Bfimpute.rds) for downstream PCA projection

## Limitations

- There is no consensus about which imputation method is the best; performance varies across datasets, protocols, and downstream analyses
- It is premature to conclude Bfimpute will achieve better performance across all datasets; results may not generalize beyond benchmarked data
- Imputation can increase false positives in downstream analysis; parameters may need tuning to maximize true positives or minimize false positives

## Evidence

- [other] Project all imputed matrices into PCA space and compute cell-to-cell correlations against the labeled cell types.: [evidence span withheld — anchor status: not_found]
- [supplementary] Comparison expected outcome: SCRABBLE and netNMF-sc did not outperform Bfimpute on the Chu dataset.: [evidence span withheld — anchor status: not_found]
