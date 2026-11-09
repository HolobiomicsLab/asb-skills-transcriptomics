---
name: trajectory-accuracy-benchmarking
description: Use when you must decide which 2D embedding (t-SNE/UMAP/MDS over 20/40/100%
  HVGs) to feed Slingshot or Monocle3, or when benchmarking inferred pseudotime against
  ground truth on simulated scRNA-seq data.
when_to_use_negative:
- The trajectory method does not use two-dimensional representations — Escort presently
  only considers trajectory inference approaches that use two-dimensional representations
- You rely on RNA velocity-derived dimension reduction — support for these is not
  yet implemented in Escort
- You need a guaranteed recommendation — Escort is not guaranteed to identify a recommended
  embedding among the choices being evaluated
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Escort
  role: Computes connectivity (DCcheck), same-group preservation (SimiRetain), coverage
    area (GOF), and ambiguous-cell (USHAPE) metrics and combines them via calcScore
    into the embedding suitability score
  repo: https://github.com/xiaorudong/Escort
- name: Slingshot
  role: Fits trajectories and pseudotime on each embedding for accuracy evaluation;
    its pseudotime standard deviations over the closest 10% of neighbors feed the
    USHAPE ambiguous-cell metric
- name: Monocle3
  role: Alternative trajectory inference method fit on every embedding in the benchmark
- name: Seurat
  role: LogNormalize normalization via NormalizeData() during preprocessing
- name: scran
  role: modelGeneVar() and getTopHVGs identify highly variable genes at 20/40/100%
    proportions
- name: Rtsne
  role: Builds t-SNE embeddings from an initial PCA retaining the top 50 dimensions
- name: SCORPIUS
  role: Builds MDS embeddings using the Spearman distance metric
- name: mclust
  role: Basis for the hierarchical clustering used to generate clusters before trajectory
    fitting
- name: Scaffold
  role: R package used to simulate the benchmark scRNA-seq datasets from a human pancreas
    reference
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1093/bib/bbae216
    title: Data-driven selection of analysis decisions in single-cell RNA-seq trajectory
      inference
  source_content_hash: sha256:1d4862f5626d8205a85e28b2ce5388fa69ce98dd25d4993e9eb7044f35a9fcbb
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-4.0
    url: https://github.com/xiaorudong/Escort
  repo_url: https://github.com/xiaorudong/Escort
  tools:
  - Escort
  - Monocle3
  - R
  - Rtsne
  - SCORPIUS
  - Scaffold
  - Seurat
  - Slingshot
  - mclust
  - scran
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6785714285714286
n_claims: 28
---

# trajectory-accuracy-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks single-cell RNA-seq trajectory inference accuracy across embeddings built from varying highly variable gene (HVG) proportions and dimension reduction techniques, scoring each embedding with Escort and, when ground-truth pseudotime exists, Kendall rank correlation and mean squared error.

## When to use

Use when you must decide which 2D embedding (t-SNE/UMAP/MDS over 20/40/100% HVGs) to feed Slingshot or Monocle3, or when benchmarking inferred pseudotime against ground truth on simulated scRNA-seq data.

## When NOT to use

- The trajectory method does not use two-dimensional representations — Escort presently only considers trajectory inference approaches that use two-dimensional representations
- You rely on RNA velocity-derived dimension reduction — support for these is not yet implemented in Escort
- You need a guaranteed recommendation — Escort is not guaranteed to identify a recommended embedding among the choices being evaluated

## Inputs

- Normalized scRNA-seq expression matrix (Seurat LogNormalize), e.g. Scaffold-simulated datasets from a human pancreas reference (500 cells, 15,894 genes, 20% dynamic genes)
- Highly variable gene subsets (top 20%, 40%, 100% of genes, or 2000/4000/10,000 HVGs) identified by scran modelGeneVar/getTopHVGs
- 2D embeddings per dataset/HVG-proportion combination from t-SNE (Rtsne, initial PCA with top 50 dimensions), UMAP (defaults), and MDS (SCORPIUS, Spearman distance)
- Ground-truth pseudotime per cell (simulated datasets) for Kendall correlation and MSE computation

## Outputs

- Escort composite scores (from DCcheck, SimiRetain, GOF, USHAPE) per embedding, classified as recommended (score > 0) or not-recommended
- Kendall rank correlation and mean squared error (MSE) of inferred versus ground-truth pseudotime for each embedding-method pair
- Accuracy table / escort_score_correlation.csv correlating Escort total scores with Kendall correlations and MSE across all embeddings (e.g. 450 embeddings for 50 datasets x 9 embeddings)

## How to apply

Preprocess each dataset (remove cells with duplicated ground truth time, filter low-quality cells by outlier unique gene/total molecule counts and high mitochondrial percentage, drop genes with fewer than 3 total counts, apply Seurat 'LogNormalize', select HVGs with scran modelGeneVar/getTopHVGs at 20/40/100% proportions). Build embeddings per dataset and HVG proportion: t-SNE via Rtsne (initial PCA retaining top 50 dimensions), UMAP with default settings, and MDS via SCORPIUS using the Spearman distance metric. Generate clusters with mclust-based hierarchical clustering and fit trajectories with both Slingshot and Monocle3. Run Escort's evaluation: DCcheck excludes any embedding with newly disconnected clusters, then SimiRetain (3-NN same-group preservation), GOF (alpha-convex hull area proportion), and USHAPE (ambiguous cells from Slingshot pseudotime SDs over the closest 10% of neighbors exceeding the skew-adjusted upper fence) are combined via calcScore into a score from negative infinity to 2. On simulated data with ground-truth pseudotime, additionally compute Kendall rank correlation and MSE of inferred versus true pseudotime, and correlate Escort scores against these across all embeddings.

## Related tools

- **Escort** (Computes connectivity (DCcheck), same-group preservation (SimiRetain), coverage area (GOF), and ambiguous-cell (USHAPE) metrics and combines them via calcScore into the embedding suitability score) — https://github.com/xiaorudong/Escort
- **Slingshot** (Fits trajectories and pseudotime on each embedding for accuracy evaluation; its pseudotime standard deviations over the closest 10% of neighbors feed the USHAPE ambiguous-cell metric)
- **Monocle3** (Alternative trajectory inference method fit on every embedding in the benchmark)
- **Seurat** (LogNormalize normalization via NormalizeData() during preprocessing)
- **scran** (modelGeneVar() and getTopHVGs identify highly variable genes at 20/40/100% proportions)
- **Rtsne** (Builds t-SNE embeddings from an initial PCA retaining the top 50 dimensions)
- **SCORPIUS** (Builds MDS embeddings using the Spearman distance metric)
- **mclust** (Basis for the hierarchical clustering used to generate clusters before trajectory fitting)
- **Scaffold** (R package used to simulate the benchmark scRNA-seq datasets from a human pancreas reference)

## Examples

```
if (!requireNamespace("remotes", quietly=TRUE)) install.packages("remotes"); remotes::install_github("xiaorudong/Escort"); Escort::shinyEscort()
```

## Evaluation signals

- Embeddings flagged 'good' must satisfy Kendall rank correlation > 0.85 and MSE < 0.01, the benchmark criterion used to calibrate Escort's scoring cutoffs
- Embeddings classified as good should receive Escort total scores above the score > 0 recommendation threshold
- Any embedding with newly disconnected clusters detected by DCcheck must be excluded immediately and classified not recommended
- Escort score should reproduce the reported positive association with Kendall correlation and inverse association with MSE across embeddings (Figure 4B)
- No single dimension reduction algorithm or HVG percentage should consistently win across all datasets — a uniform winner indicates a pipeline bug

## Limitations

- No dimension reduction algorithm, HVG percentage, or combination showed a clear performance advantage across datasets, so the best choice cannot be pre-selected; performance is highly dataset-specific
- Escort only considers trajectory inference approaches using two-dimensional representations; methods outside this restriction require further evaluation
- RNA velocity-derived dimension reduction methods are not yet implemented (future work)
- Escort is not guaranteed to identify a recommended embedding among the choices being evaluated
- Quality control steps such as normalization and outlier removal may significantly impact Escort's embedding evaluations

## Evidence
