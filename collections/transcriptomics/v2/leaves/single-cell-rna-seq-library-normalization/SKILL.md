---
name: single-cell-rna-seq-library-normalization
description: Decision support for choosing between RAW, TC, TMM, or FQ normalization
  before scRNA-seq dimensionality reduction (PCA, ZIFA, ZINB-WaVE), when the analysis
  goal is to quantify how normalization choice affects clustering quality.
when_to_use_negative:
- The counts are already normalized (e.g., TC-scaled or log-transformed) — re-applying
  library normalization is redundant and the comparison design assumes RAW as one
  arm
- The downstream model is ZINB-WaVE with a column of ones in the gene-level covariate
  matrix — the paper shows its cell-level intercept acts as a global-scaling normalization
  factor, so no prior normalization is needed
- The data are bulk RNA-seq where a standard edgeR TMM differential-expression workflow
  suffices without a single-cell low-dimensional benchmark
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ZINB-WaVE (zinbwave)
  role: Fits the zero-inflated negative binomial model on raw counts; its sample-level
    intercept serves as the global-scaling normalization factor, making ZINB-WaVE
    the no-normalization arm of the comparison
  repo: https://github.com/drisso/zinbwave
- name: edgeR
  role: Performs TMM normalization to build the TMM arm of the four normalization
    variants per data set
- name: ZIFA
  role: Zero-inflated dimensionality reduction applied on each normalized matrix (after
    log2(1 + count) transformation) to test whether normalization choice affects it
    like PCA
  repo: https://github.com/epierson9/ZIFA
- name: scRNAseq
  role: Loads the Tasic V1 mouse visual cortex count matrix used in the silhouette/QC-correlation
    benchmark
- name: BiocManager
  role: Installs the zinbwave package (R >=3.4, Bioconductor >=3.6)
provenance:
  source_task_ids:
  - task_008
  - task_002
  source_papers:
  - doi: 10.1038/s41467-017-02554-5
    title: A general and flexible method for signal extraction from single-cell RNA-seq
      data
  source_content_hash: sha256:19aa807ea5ca55ec1f0c225f436e6ee7dd4dbcc61089aaf0c80d8cb131b97cda
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drisso/zinbwave
  tools:
  - BiocManager
  - Bioconductor (>=3.6)
  - R (>=3.4)
  - ZIFA
  - ZINB-WaVE (zinbwave)
  - edgeR
  - scRNAseq
  - scater
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# single-cell-rna-seq-library-normalization

## Summary

Constructs comparable normalization variants (RAW, TC, TMM, FQ) of scRNA-seq count matrices and benchmarks their effect on downstream low-dimensional representations, testing whether ZINB-WaVE's built-in global-scaling intercept removes the need for a preliminary normalization step.

## When to use

Decision support for choosing between RAW, TC, TMM, or FQ normalization before scRNA-seq dimensionality reduction (PCA, ZIFA, ZINB-WaVE), when the analysis goal is to quantify how normalization choice affects clustering quality.

## When NOT to use

- The counts are already normalized (e.g., TC-scaled or log-transformed) — re-applying library normalization is redundant and the comparison design assumes RAW as one arm
- The downstream model is ZINB-WaVE with a column of ones in the gene-level covariate matrix — the paper shows its cell-level intercept acts as a global-scaling normalization factor, so no prior normalization is needed
- The data are bulk RNA-seq where a standard edgeR TMM differential-expression workflow suffices without a single-cell low-dimensional benchmark

## Inputs

- Raw scRNA-seq count matrices (V1, S1/CA1, glioblastoma, mESC) from the drisso/zinb_analysis repository
- Author-provided cell-type labels for each data set
- R/Bioconductor environment with edgeR, scRNAseq, zinbwave, scater installed via BiocManager

## Outputs

- Four expression matrices per data set (RAW, TC, TMM, FQ)
- First two low-dimensional components from PCA, ZIFA, and ZINB-WaVE per matrix
- silhouette_widths.csv with one row per data set per normalization per method
- Tabulated average silhouette widths supporting the normalization-vs-method comparison (e.g., tasic_v1_benchmark_results.csv)

## How to apply

From each scRNA-seq count matrix (e.g., the V1, S1/CA1, glioblastoma, and mESC data sets in drisso/zinb_analysis), build four versions: unnormalized RAW counts, total-count (TC) scaling, TMM normalization via edgeR, and FQ (full-quantile) normalization. Run PCA and ZIFA on each normalized version — note ZIFA requires the log2(1 + count) transformation per its README — while ZINB-WaVE is fit with the Bioconductor zinbwave package (R >=3.4, Bioconductor >=3.6) directly on raw counts, relying on the sample-level intercept acting as a global-scaling normalization factor. Compare the methods by computing average silhouette widths of author-provided cell-type labels over Euclidean distances in each low-dimensional space. The decision point is whether normalization choice or method choice (PCA vs ZIFA) dominates: the expected pattern is that normalization matters more than the PCA-vs-ZIFA choice, and that ZINB-WaVE on raw counts is comparable to the best normalization, eliminating the preliminary normalization step.

## Related tools

- **ZINB-WaVE (zinbwave)** (Fits the zero-inflated negative binomial model on raw counts; its sample-level intercept serves as the global-scaling normalization factor, making ZINB-WaVE the no-normalization arm of the comparison) — https://github.com/drisso/zinbwave
- **edgeR** (Performs TMM normalization to build the TMM arm of the four normalization variants per data set)
- **ZIFA** (Zero-inflated dimensionality reduction applied on each normalized matrix (after log2(1 + count) transformation) to test whether normalization choice affects it like PCA) — https://github.com/epierson9/ZIFA
- **scRNAseq** (Loads the Tasic V1 mouse visual cortex count matrix used in the silhouette/QC-correlation benchmark)
- **BiocManager** (Installs the zinbwave package (R >=3.4, Bioconductor >=3.6))

## Examples

```
BiocManager::install("zinbwave")
```

## Evaluation signals

- silhouette_widths.csv exists and contains exactly one row per data set per normalization (RAW, TC, TMM, FQ) per method (PCA, ZIFA, ZINB-WaVE)
- Relative silhouette ordering reproduces the paper's pattern: the spread across normalizations exceeds the spread between PCA and ZIFA on the same normalization
- ZINB-WaVE on RAW counts achieves average silhouette widths comparable to PCA/ZIFA with the best-performing normalization
- For the Tasic V1 benchmark, ZINB-WaVE achieves the highest silhouette width and the lowest absolute correlations of its two components with detection rate and total reads
- zinbwave version 0.1.1 installed when exact reproduction of the paper's analyses is required

## Limitations

- There is no guarantee that the low-dimensional signal extracted by ZINB-WaVE is biologically relevant, and unwanted technical variation not accounted for in the model (or prior normalization) will be captured by the low-rank matrix W
- ZINB-WaVE's better performance relative to PCA comes at a computational cost from numerically optimizing a non-convex likelihood function
- ZIFA input must be log-transformed (Y = log2(1 + count_data)) and for large gene sets block_ZIFA is recommended, so the RAW-vs-normalized comparison is not symmetric across methods
- The original paper's analyses require zinbwave version 0.1.1, not the current Bioconductor release

## Evidence

- [intro] Sample-level intercept as normalization factor: "includes a sample-level intercept, which serves as a global-scaling normalization factor, and gives the user the ability to include both gene-level and sample-level covariates"
- [discussion] Raw counts without prior normalization: "the corresponding cell-level intercept acts as a global-scaling normalization factor, allowing the modeling of raw count data, with no need for prior normalization"
- [discussion] Unaccounted technical variation caveat: "the low-rank matrix W inferred by ZINB-WaVE will capture such confounding effects"
- [readme] zinbwave version pin for reproducibility: [evidence span withheld — anchor status: external_doc]
- [methods] Runtime requirements for zinbwave: "Note that `zinbwave` requires R (>=3.4) and Bioconductor (>=3.6)."
