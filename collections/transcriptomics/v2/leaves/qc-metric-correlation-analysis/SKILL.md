---
name: qc-metric-correlation-analysis
description: Use when scRNA-seq low-dimensional components (e.g., ZINB-WaVE W factors,
  PCA or ZIFA scores) need to be benchmarked for residual technical effects by correlating
  them with QC measures like detection rate and total reads.
when_to_use_negative:
- The input is not count-based scRNA-seq data — the zero-inflated negative binomial
  model underlying ZINB-WaVE accounts for zero inflation, over-dispersion, and the
  count nature of the data, so bulk or non-count matrices are out of scope.
- QC measures are already included as model covariates — the paper advises including
  highly correlated QC measures as covariates in the model, in which case a post hoc
  correlation audit of the components is largely redundant.
- The goal is supervised differential expression — the paper notes no such capability
  is yet available in zinbwave (only envisioned for a future version), so this QC-benchmarking
  skill cannot substitute for it.
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: zinbwave
  role: Produces the ZINB-WaVE low-dimensional components whose correlation with QC
    measures is audited
  repo: https://github.com/drisso/zinbwave
- name: scater
  role: Bioconductor package for the QC workflow that motivates correlating latent
    factors with QC measures
- name: ZIFA
  role: Zero-inflated factor analysis comparator whose components are also correlated
    with QC measures
  repo: https://github.com/epierson9/ZIFA
- name: scRNAseq
  role: Bioconductor package supplying the Tasic V1 dataset used in the benchmark
- name: drisso/zinb_analysis
  role: Reference analysis code reproducing the ZINB-WaVE paper's results and figures
  repo: https://github.com/drisso/zinb_analysis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-017-02554-5
    title: A general and flexible method for signal extraction from single-cell RNA-seq
      data
  source_content_hash: sha256:578562743fa2ec086c53efd2948e73d71893eaadc6c214987382b8bf02b090a3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drisso/zinbwave
  tools:
  - Bioconductor (>=3.6)
  - R (>=3.4)
  - ZIFA
  - drisso/zinb_analysis
  - scRNAseq
  - scater
  - zinbwave
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# qc-metric-correlation-analysis

## Summary

Quantifies how much technical variation (detection rate, sequencing depth/total reads) drives low-dimensional representations of scRNA-seq data by computing absolute correlations of inferred latent factors with QC measures, as used to benchmark ZINB-WaVE against PCA and ZIFA on the Tasic V1 visual cortex dataset.

## When to use

Use when scRNA-seq low-dimensional components (e.g., ZINB-WaVE W factors, PCA or ZIFA scores) need to be benchmarked for residual technical effects by correlating them with QC measures like detection rate and total reads.

## When NOT to use

- The input is not count-based scRNA-seq data — the zero-inflated negative binomial model underlying ZINB-WaVE accounts for zero inflation, over-dispersion, and the count nature of the data, so bulk or non-count matrices are out of scope.
- QC measures are already included as model covariates — the paper advises including highly correlated QC measures as covariates in the model, in which case a post hoc correlation audit of the components is largely redundant.
- The goal is supervised differential expression — the paper notes no such capability is yet available in zinbwave (only envisioned for a future version), so this QC-benchmarking skill cannot substitute for it.

## Inputs

- Raw scRNA-seq count matrix (e.g., Tasic V1 mouse visual cortex data loaded via the scRNAseq Bioconductor package, subset to the 1000 most variable genes)
- TC (total-count) normalized count matrix used to fit ZINB-WaVE, PCA, and ZIFA
- Per-cell QC measures: detection rate and total reads
- Cell-level biological labels (e.g., layer of origin) for silhouette-width computation

## Outputs

- Absolute correlation table of each method's first two components with detection rate and total reads
- Per-cluster average silhouette widths against layer-of-origin labels for ZINB-WaVE, PCA, and ZIFA spaces
- Combined side-by-side results table (tasic_v1_benchmark_results.csv)

## How to apply

Fit each dimensionality-reduction method on the normalized count matrix and extract the first two low-dimensional components (ZINB-WaVE via the zinbwave package with its zero-inflated negative binomial model, plus PCA and ZIFA as comparators). Compute per-cluster average silhouette widths against biological labels (e.g., layer of origin in the Tasic V1 data) from cell pairwise distances in each method's low-dimensional space, following the paper's practice of deriving distances from the inferred factors. Compute the absolute correlation of each method's first two components with QC measures (detection rate and total reads), in the spirit of the scater QC workflow. Tabulate silhouette widths and QC correlations side by side (written to tasic_v1_benchmark_results.csv in the reproduction task) and accept the result if ZINB-WaVE attains the highest silhouette width and the lowest absolute QC correlations, consistent with the paper's claim of a more stable and accurate low-dimensional representation than PCA and ZIFA. If a latent factor is highly correlated with a QC measure, the paper recommends including that QC measure as a model covariate rather than discarding the analysis.

## Related tools

- **zinbwave** (Produces the ZINB-WaVE low-dimensional components whose correlation with QC measures is audited) — https://github.com/drisso/zinbwave
- **scater** (Bioconductor package for the QC workflow that motivates correlating latent factors with QC measures)
- **ZIFA** (Zero-inflated factor analysis comparator whose components are also correlated with QC measures) — https://github.com/epierson9/ZIFA
- **scRNAseq** (Bioconductor package supplying the Tasic V1 dataset used in the benchmark)
- **drisso/zinb_analysis** (Reference analysis code reproducing the ZINB-WaVE paper's results and figures) — https://github.com/drisso/zinb_analysis

## Examples

```
qc_cor <- abs(cor(W_zinbwave, qc[, c("detection_rate", "total_reads")])); write.csv(qc_cor, "tasic_v1_benchmark_results.csv")
```

## Evaluation signals

- ZINB-WaVE's first two components show the lowest absolute correlation with detection rate and total reads among ZINB-WaVE, PCA, and ZIFA.
- ZINB-WaVE achieves the highest per-cluster average silhouette width against layer-of-origin labels computed from cell distances in each method's low-dimensional space.
- Silhouette widths and QC correlations are tabulated side by side and written to tasic_v1_benchmark_results.csv.
- All three methods are fit on the same input: TC-normalized counts of the 1000 most variable genes from the Tasic V1 dataset.
- Results are consistent with the paper's finding that ZINB-WaVE yields a more stable and accurate low-dimensional representation than PCA and ZIFA.

## Limitations

- There is no guarantee that the low-dimensional signal extracted by ZINB-WaVE is biologically relevant — if unwanted technical variation is not accounted for in the model, the inferred W will capture such confounding effects.
- ZINB-WaVE's better performance versus PCA comes at a computational cost because a non-convex likelihood must be numerically optimized.
- The zinbwave version matters for exact reproduction: the paper's analysis code requires zinbwave version 0.1.1.

## Evidence

- [discussion] QC-correlation motivation: "explore the correlation between the latent factors estimated by our procedure and known measures of quality control"
- [discussion] scater as QC workflow tool: "using, for instance, the Bioconductor R package scater43 (see Fig. 2f)"
- [discussion] Remediation decision point: "it may be beneficial to include these QC measures as covariates in the model."
- [abstract] Stability claim underpinning the benchmark: "the model and its associated estimation procedure are able to give a more stable and accurate low-dimensional representation of the data than principal component analysis (PCA) and zero-inflated"
- [readme] Reproducibility codebase: [evidence span withheld — anchor status: external_doc]
