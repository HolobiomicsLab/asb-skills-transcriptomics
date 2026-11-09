---
name: quantile-normalization-of-merged-expression-matrices
description: Use when pseudo-bulk profiles computed from scRNA-seq (e.g. per pre-implantation
  stage from GSE38495/GSE45719) must be compared against bulk RNA-seq profiles from
  a different platform, where column-wise expression distributions differ and must
  be forced onto a common scale.
when_to_use_negative:
- Analysis stays at the single-cell level and Seurat's per-cell normalization/integration
  (e.g. SCTransform or log-normalization) already provides a common scale — quantile-normalizing
  the merged matrix is a cross-platform pseudo-bulk vs bulk step
- Input is already quantile-normalized or the comparison is within a single platform/batch
  where distribution shifts do not confound the correlation
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: preprocessCore
  role: Provides normalize.quantiles (v1.48.0) used to quantile-normalize the merged
    pseudo-bulk/bulk expression matrix
- name: Seurat
  role: Builds the Seurat objects (CreateSeuratObject, min.features = 2000, min.cells
    = 0) from which pseudo-bulk per-stage expression is computed prior to normalization
- name: TimeTalk
  role: Custom R scripts from the TimeTalk Figshare project deposit calculate the
    pseudo-bulk expression for an average single cell in each stage
  repo: https://github.com/ChengLiLab/TimeTalk
- name: kBET
  role: Assesses, in PCA space, whether data points from different batches are well-mixed
    within random neighborhoods of a fixed size after normalization
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-023-05283-2
    title: TimeTalk uses single-cell RNA-seq datasets to decipher cell-cell communication
      during early embryo development
  source_content_hash: sha256:c52ffdf6a3aec27dc8cd50a36c1ab7631f2264cf7abcca0230b82441f6a373f7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChengLiLab/TimeTalk
  tools:
  - R
  - Seurat
  - TimeTalk
  - kBET
  - preprocessCore
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# Quantile normalization of merged expression matrices

## Summary

Applies quantile normalization (preprocessCore::normalize.quantiles, v1.48.0) to the merged pseudo-bulk scRNA-seq and low-input bulk RNA-seq expression matrix so stage-wise profiles become directly comparable for Pearson correlation and kBET batch-effect assessment.

## When to use

Use when pseudo-bulk profiles computed from scRNA-seq (e.g. per pre-implantation stage from GSE38495/GSE45719) must be compared against bulk RNA-seq profiles from a different platform, where column-wise expression distributions differ and must be forced onto a common scale.

## When NOT to use

- Analysis stays at the single-cell level and Seurat's per-cell normalization/integration (e.g. SCTransform or log-normalization) already provides a common scale — quantile-normalizing the merged matrix is a cross-platform pseudo-bulk vs bulk step
- Input is already quantile-normalized or the comparison is within a single platform/batch where distribution shifts do not confound the correlation

## Inputs

- Merged gene-by-sample expression matrix combining per-stage pseudo-bulk scRNA-seq profiles (GSE38495 MII-oocyte, GSE45719 E1-E4.0) and matching low-input bulk RNA-seq profiles from the TimeTalk Figshare deposit
- Seurat objects built with CreateSeuratObject(min.features = 2000, min.cells = 0) for pseudo-bulk computation

## Outputs

- Quantile-normalized merged expression matrix with identical empirical distributions across columns
- Stage-wise Pearson correlation coefficients between pseudo-bulk scRNA-seq and bulk RNA-seq profiles
- kBET batch-mixing statistics on the PCA-space representation, reproducing Fig. 1a-b and Supplementary Figs. 1-3

## How to apply

First compute pseudo-bulk expression for an average single cell in each pre-implantation stage from the scRNA-seq count matrices (Seurat CreateSeuratObject with min.features = 2000, min.cells = 0, plus the custom R scripts shipped with the TimeTalk Figshare deposit). Merge the pseudo-bulk matrix with the low-input bulk RNA-seq profiles into a single gene-by-sample matrix, then run preprocessCore::normalize.quantiles (version 1.48.0) so every column shares the same empirical distribution — the rationale being that pseudo-bulk and bulk profiles are cross-platform and quantile normalization is required before their Pearson correlation is meaningful. On the normalized matrix, compute Pearson correlations between each stage's pseudo-bulk profile and the matching bulk profile (expect ~0.91 for the late 2-cell stage) and run kBET in PCA space with a fixed neighborhood size to test whether batches are well-mixed within random neighborhoods.

## Related tools

- **preprocessCore** (Provides normalize.quantiles (v1.48.0) used to quantile-normalize the merged pseudo-bulk/bulk expression matrix)
- **Seurat** (Builds the Seurat objects (CreateSeuratObject, min.features = 2000, min.cells = 0) from which pseudo-bulk per-stage expression is computed prior to normalization)
- **TimeTalk** (Custom R scripts from the TimeTalk Figshare project deposit calculate the pseudo-bulk expression for an average single cell in each stage) — https://github.com/ChengLiLab/TimeTalk
- **kBET** (Assesses, in PCA space, whether data points from different batches are well-mixed within random neighborhoods of a fixed size after normalization)

## Examples

```
norm.matrix <- preprocessCore::normalize.quantiles(merged.pseudobulk.bulk.expr.matrix)
```

## Evaluation signals

- Post-normalization, every column of the merged matrix has the same sorted expression values (quantile-normalization invariant)
- Pearson correlation between the late 2-cell pseudo-bulk profile and its matching low-input bulk profile reaches ~0.91, with high agreement for other matching stages
- kBET in PCA space reports that data points from different batches are well-mixed within random neighborhoods of the fixed size
- Results are reproducible against Fig. 1a-b and Supplementary Figs. 1-3 of the TimeTalk paper using data from the Figshare deposit

## Limitations

- Quantile normalization forces identical distributions across all samples, which can remove genuine global expression differences between stages if applied in the wrong context
- The step depends on the upstream pseudo-bulk definition ('average single cell' per stage); a different aggregation changes the normalized matrix and downstream correlations
- The paper's workflow is specific to the curated early-embryo scRNA-seq (GSE38495/GSE45719) vs low-input bulk comparison; transfer to other datasets should be re-validated

## Evidence

- [discussion] Quantile normalization was performed with the function “normalize.quantiles” from R package preprocessCore (version 1.48.0).: "Quantile normalization was performed with the function “normalize.quantiles” from R package preprocessCore (version 1.48.0)."
- [discussion] Custom R scripts were used for the pre-implantation scRNA-seq data to calculate the pseudo-bulk expression for an average single cell in each stage.: "Custom R scripts were used for the pre-implantation scRNA-seq data to calculate the pseudo-bulk expression for an average single cell in each stage."
- [discussion] The kBET analysis assessed whether the data points with different batches in the PCA space were well-mixed within random neighborhoods of a fixed size.: "The kBET analysis assessed whether the data points with different batches in the PCA space were well-mixed within random neighborhoods of a fixed size."
- [results] The pseudo-bulk scRNA-seq profiles and low-input bulk RNA-seq profiles of matching stages are highly correlated; for the late 2-cell stage the Pearson correlation coefficient is 0.91.: [evidence span withheld — anchor status: not_found]
