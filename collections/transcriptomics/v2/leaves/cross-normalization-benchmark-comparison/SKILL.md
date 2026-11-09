---
name: cross-normalization-benchmark-comparison
description: Use when scRNA-seq raw count matrices with known cell-type labels are
  available and the question is how much normalization method (RAW/TC/TMM/FQ) vs.
  dimensionality-reduction method (PCA/ZIFA/ZINB-WaVE) affects downstream clustering
  quality.
when_to_use_negative:
- Input is already normalized or log-transformed (e.g. log2(1 + counts) for ZIFA)
  and the RAW/TC/TMM/FQ crossing cannot be constructed from raw counts
- No ground-truth or author-provided cell-type labels exist, so the silhouette criterion
  cannot be computed
- The goal is differential expression or batch-effect correction with covariates rather
  than comparing normalization impact on clustering of labeled cells
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: zinbwave
  role: Runs ZINB-WaVE directly on unnormalized raw counts, providing the normalization-free
    arm of the benchmark; v0.1.1 required for exact reproduction
  repo: https://github.com/drisso/zinbwave
- name: ZIFA
  role: Zero-inflated dimensionality-reduction arm applied to each normalization variant
    (log2(1 + counts) input recommended by its README)
  repo: https://github.com/epierson9/ZIFA
- name: edgeR
  role: Supplies TMM normalization factors for the TMM arm of the normalization comparison
- name: drisso/zinb_analysis
  role: Source of the four real data sets, author-provided cell-type labels, and the
    paper's reproduction scripts
  repo: https://github.com/drisso/zinb_analysis
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-017-02554-5
    title: A general and flexible method for signal extraction from single-cell RNA-seq
      data
  source_content_hash: sha256:786da82070eb8f5ae5ac850eea5c53d0a826295c90d481b4ecc448f59b2caae7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drisso/zinbwave
  tools:
  - BiocManager
  - Bioconductor (>=3.6)
  - R (>=3.4)
  - ZIFA
  - drisso/zinb_analysis
  - edgeR
  - zinbwave
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# cross-normalization-benchmark-comparison

## Summary

Benchmarking skill that crosses between-sample normalization choices (RAW, TC, TMM, FQ) with dimensionality-reduction methods (PCA, ZIFA, ZINB-WaVE) and scores each representation by the average silhouette width of author-provided cell-type labels, to test whether normalization choice dominates method choice in scRNA-seq signal extraction.

## When to use

Use when scRNA-seq raw count matrices with known cell-type labels are available and the question is how much normalization method (RAW/TC/TMM/FQ) vs. dimensionality-reduction method (PCA/ZIFA/ZINB-WaVE) affects downstream clustering quality.

## When NOT to use

- Input is already normalized or log-transformed (e.g. log2(1 + counts) for ZIFA) and the RAW/TC/TMM/FQ crossing cannot be constructed from raw counts
- No ground-truth or author-provided cell-type labels exist, so the silhouette criterion cannot be computed
- The goal is differential expression or batch-effect correction with covariates rather than comparing normalization impact on clustering of labeled cells

## Inputs

- scRNA-seq raw count matrices for the V1, S1/CA1, glioblastoma, and mESC data sets from drisso/zinb_analysis
- Author-provided cell-type label vector per data set (silhouette criterion reference)
- R (>=3.4) with Bioconductor (>=3.6): zinbwave, edgeR; Python environment with ZIFA (epierson9/ZIFA)

## Outputs

- silhouette_widths.csv with one row per data set per normalization per method (average silhouette width on Euclidean distances)
- Tabulated normalization-by-method comparison confirming the paper's reported pattern (normalization effect > PCA-vs-ZIFA effect; ZINB-WaVE on raw counts comparable to best normalization)

## How to apply

Load the V1, S1/CA1, glioblastoma, and mESC count matrices plus author-provided cell-type labels from the drisso/zinb_analysis repository. Build four expression matrices per data set — RAW counts, TC (total-count) scaling, TMM normalization via edgeR, and FQ (full-quantile) normalization — then run PCA and ZIFA (epierson9/ZIFA, which expects log2(1 + count) input) on each of the four versions. Separately run ZINB-WaVE with default settings directly on the unnormalized raw counts using the Bioconductor zinbwave package (R >= 3.4, Bioconductor >= 3.6; pin zinbwave 0.1.1 to reproduce the paper exactly), because its cell-level intercept acts as a global-scaling normalization factor so no preliminary normalization is needed. For every low-dimensional representation, compute the average silhouette width of the author-provided labels using Euclidean distances, tabulate one row per data set × normalization × method (silhouette_widths.csv), and inspect the ordering: normalization choice should matter more than PCA-vs-ZIFA, and ZINB-WaVE on raw counts should be comparable to the best-performing normalization.

## Related tools

- **zinbwave** (Runs ZINB-WaVE directly on unnormalized raw counts, providing the normalization-free arm of the benchmark; v0.1.1 required for exact reproduction) — https://github.com/drisso/zinbwave
- **ZIFA** (Zero-inflated dimensionality-reduction arm applied to each normalization variant (log2(1 + counts) input recommended by its README)) — https://github.com/epierson9/ZIFA
- **edgeR** (Supplies TMM normalization factors for the TMM arm of the normalization comparison)
- **drisso/zinb_analysis** (Source of the four real data sets, author-provided cell-type labels, and the paper's reproduction scripts) — https://github.com/drisso/zinb_analysis

## Examples

```
library(devtools); install_github("drisso/archive-zinbwave@v0.1.1")
```

## Evaluation signals

- silhouette_widths.csv exists and contains exactly one row per data set × normalization × method combination
- All average silhouette widths fall in the valid range [-1, 1] and were computed on Euclidean distances of the low-dimensional representations
- Relative ordering reproduces the paper's pattern: the spread across normalizations (RAW/TC/TMM/FQ) exceeds the spread between PCA and ZIFA within each normalization
- ZINB-WaVE on unnormalized raw counts achieves average silhouette widths comparable to PCA/ZIFA run on the best-performing normalization
- zinbwave version is pinned to 0.1.1 when exact reproduction of the published figures is required

## Limitations

- ZINB-WaVE fits a non-convex likelihood, so it carries a computational cost relative to PCA
- There is no guarantee that the low-dimensional signal extracted by ZINB-WaVE is biologically relevant; if unwanted technical variation is unaccounted for, W will capture confounding effects
- ZIFA expects log2(1 + count_data) input and for datasets with more than a few thousand genes the README recommends block_ZIFA subsampling, which yields less reliable results
- Exact reproduction requires zinbwave version 0.1.1, not the current Bioconductor release

## Evidence

- [discussion] the corresponding cell-level intercept acts as a global-scaling normalization factor, allowing the modeling of raw count data, with no need for prior normalization: "the corresponding cell-level intercept acts as a global-scaling normalization factor, allowing the modeling of raw count data, with no need for prior normalization"
- [intro] We demonstrate, with simulated and real data, the model and its associated estimation procedure are able to give a more stable and accurate low-dimensional representation than PCA: [evidence span withheld — anchor status: not_found]
- [readme] This repository is designed to allow interested people to reproduce the results and figures of the paper: [evidence span withheld — anchor status: external_doc]
- [readme] To exactly reproduce the analyses of the paper, version `0.1.1` of the `zinbwave` package is required.: [evidence span withheld — anchor status: external_doc]
- [readme] If you are using count data, we recommend taking the log (ie, Y = log2(1 + count_data)) prior to using ZIFA.: [evidence span withheld — anchor status: external_doc]
- [methods] Note that `zinbwave` requires R (>=3.4) and Bioconductor (>=3.6).: "Note that `zinbwave` requires R (>=3.4) and Bioconductor (>=3.6)."
