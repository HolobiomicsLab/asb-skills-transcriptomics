---
name: logcounts-pca-baseline-embedding
description: Use when benchmarking a dimension-reduction method (e.g., CA via corral)
  on scRNA-seq counts and you need the canonical log(x+1)-then-PCA baseline embedding
  on the same datasets for ARI comparison.
when_to_use_negative:
- When the goal is to embed raw counts directly without a log-transformation distortion
  — use CA (corral) on counts instead, since CA is 'based on decomposition of a chi-squared
  residual matrix, avoiding distortive log-transformation'
- When the input matrix is not raw counts (e.g., already normalized or log-transformed)
  — re-applying log(x + 1) would double-transform and invalidate the baseline
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: corral
  role: Companion Bioconductor package whose CA-on-counts embeddings this logcounts-PCA
    baseline is benchmarked against in the NNGraph/ARI comparison
  repo: github.com/laurenhsu1/corral
- name: R
  role: Language in which the log(x+1) transformation, PCA, NNGraph clustering, and
    ARI computation are executed
- name: Bioconductor
  role: Ecosystem providing SingleCellExperiment classes that hold counts, logcounts,
    reducedDims, and cluster labels
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:35c825ae44321d9a3f1be2c016508590c4124e23adfb993b9e1f8facb269d751
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - corral
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8666666666666667
n_claims: 15
---

# logcounts-pca-baseline-embedding

## Summary

Computes the standard single-cell baseline embedding: a log(x + 1) transformation of the raw counts matrix followed by PCA on the resulting 'logcounts', serving as the comparator against which CA (corral) embeddings are benchmarked by NNGraph clustering and Adjusted Rand Index.

## When to use

Use when benchmarking a dimension-reduction method (e.g., CA via corral) on scRNA-seq counts and you need the canonical log(x+1)-then-PCA baseline embedding on the same datasets for ARI comparison.

## When NOT to use

- When the goal is to embed raw counts directly without a log-transformation distortion — use CA (corral) on counts instead, since CA is 'based on decomposition of a chi-squared residual matrix, avoiding distortive log-transformation'
- When the input matrix is not raw counts (e.g., already normalized or log-transformed) — re-applying log(x + 1) would double-transform and invalidate the baseline

## Inputs

- Raw counts matrix (genes x cells) from a benchmark scRNAseq dataset (SCMixology or Zhengmix)
- Ground-truth cell-type annotations from the original dataset publications
- SingleCellExperiment or similar count container in R/Bioconductor

## Outputs

- Cell embeddings from PCA on log(x+1)-transformed counts (logcounts)
- NNGraph cluster assignments for each cell
- ARI values per dataset for the PCA-on-logcounts arm (e.g., rows of ari_comparison.csv)

## How to apply

Start from each benchmark dataset's raw counts matrix (e.g., the three SCMixology CellBench datasets and three Zhengmix DuoClustering2018 datasets with published ground-truth labels). Apply the log(x + 1) transformation to the counts matrix to produce logcounts, mirroring typical single-cell workflows in which 'most single cell workflows begin with a log(x + 1) transformation of the counts matrix, and then use PCA to decompose the resulting logcounts data'. Run PCA on the logcounts (PCA is computed by SVD of column-centered or Z-score normalized data), retaining the cell embeddings. Cluster cells in the PCA embedding with NNGraph clustering and compute the Adjusted Rand Index against the annotated cell types, so results are directly comparable to the CA-on-counts arm of the Fig. 1D ARI table. Keep preprocessing identical between the CA and PCA arms so the only difference is the embedding step.

## Related tools

- **corral** (Companion Bioconductor package whose CA-on-counts embeddings this logcounts-PCA baseline is benchmarked against in the NNGraph/ARI comparison) — github.com/laurenhsu1/corral
- **R** (Language in which the log(x+1) transformation, PCA, NNGraph clustering, and ARI computation are executed)
- **Bioconductor** (Ecosystem providing SingleCellExperiment classes that hold counts, logcounts, reducedDims, and cluster labels)

## Examples

```
logcounts <- log1p(counts(sce)); reducedDim(sce, "PCA") <- prcomp(t(logcounts), center = TRUE)$x[, 1:30]
```

## Evaluation signals

- Verify logcounts equal log1p(counts) elementwise on the input matrix before running PCA
- Confirm the PCA is fit on logcounts (not raw counts) and embeddings have the same cells as the CA arm for a fair ARI comparison
- Check NNGraph clustering is run on the PCA embedding with the same clustering settings used for the CA embedding arm
- Compare resulting ARI against the CA-on-counts ARI per dataset — in the paper's benchmark CA outperformed or matched PCA in every counts and logcounts pipeline comparison across the six datasets
- ARI must be computed against the same annotated cell-type labels used for the CA arm

## Limitations

- Ground-truth labels come from original dataset publications and were not independently validated, so ARI performance is limited by the original cell type assignments
- ARI evaluation may penalize methods that better distinguish sub-populations or rare cell types relative to reference labels
- Benchmark datasets (SCMixology pseudo-cell mixtures, Zhengmix cell-type pools) are 'by design simpler than data from complex tissues' and do not reflect real-world tissue complexity
- PCA on logcounts applies a 'distortive log-transformation' to count data, per the article's critique of typical pipelines

## Evidence

- [intro] log(x + 1) transformation of the counts matrix followed by PCA in typical single cell workflows: "most single cell workflows begin with a log(x + 1) transformation of the counts matrix, and then use PCA to decompose the resulting “logcounts” data"
- [abstract] Switching from PCA to CA is achieved through a simple pipeline substitution: "Switching from PCA to CA is achieved through a simple pipeline substitution and improves dimension reduction of scRNAseq datasets."
- [abstract] CA avoids the log-transformation that the logcounts-PCA baseline requires: "CA is based on decomposition of a chi-squared residual matrix, avoiding distortive log-transformation"
- [readme] corral interfaces with Bioconductor single-cell classes: "corral is a package to perform correspondence analysis on count data, particularly for single-cell (e.g., RNA-seq)."
- [discussion] Benchmark dataset simplicity caveat: "Except for SCMixology and Zhengmix (both comprising well-defined cell clusters and by design simpler than data from complex tissues)"
