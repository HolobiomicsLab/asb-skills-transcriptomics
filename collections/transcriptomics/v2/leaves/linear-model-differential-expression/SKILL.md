---
name: linear-model-differential-expression
description: Use when performing differential expression or guide-gRNA association
  testing on single-cell UMI count data (e.g. Perturb-seq, CROP-seq) after Normalisr
  normalization, especially when unbiased null P values or control of gRNA cross-association
  FPR matters.
when_to_use_negative:
- Hypothesis testing of arbitrary nonlinear associations — the article explicitly
  states Normalisr is not designed for this
- Full-length (non-UMI) scRNA-seq platforms, which were not validated and may require
  revalidating the nonlinear cellular summary covariates
- Integrative analyses spanning multiple datasets needing explicit batch-effect correction,
  which Normalisr does not address
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Normalisr
  role: Provides the linear-model differential expression function (de / normalisr.de)
    applied after its own QC and normalization steps (qc_reads, qc_outlier, lcpm,
    normcov, scaling_factor, compute_var, normvar)
  repo: github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_002
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:4bd4d446a4d44a858286080f59393d1b212bf8b43c09fe3cf2b28e81ee8ce05d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Normalisr
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# linear-model-differential-expression

## Summary

Runs Normalisr's linear-model differential expression (the 'de' module) on normalized single-cell UMI counts to test case-control or gRNA-gene associations with exact, permutation-free P values, optionally including untested gRNAs as fixed-effect covariates to model gRNA competition.

## When to use

Use when performing differential expression or guide-gRNA association testing on single-cell UMI count data (e.g. Perturb-seq, CROP-seq) after Normalisr normalization, especially when unbiased null P values or control of gRNA cross-association FPR matters.

## When NOT to use

- Hypothesis testing of arbitrary nonlinear associations — the article explicitly states Normalisr is not designed for this
- Full-length (non-UMI) scRNA-seq platforms, which were not validated and may require revalidating the nonlinear cellular summary covariates
- Integrative analyses spanning multiple datasets needing explicit batch-effect correction, which Normalisr does not address

## Inputs

- Normalized (lcpm/normvar) UMI expression matrix in tsv or gzipped tsv format
- gRNA assignment matrix or cell grouping/case-control label vector (text file, one name per line or tsv)
- Raw UMI count matrix (tsv or Cell Ranger sparse mtx) plus gRNA assignment matrix from GEO for full pipelines

## Outputs

- Per-gene (or per-gRNA-gene pair) table of log fold-changes, P values, and Q values (e.g. de_associations.tsv)
- Null-calibration summary such as ks_uniformity_results.csv with per-bin, per-split KS statistics and P values
- FPR estimates from Storey's method on NTC P-value histograms and association-inflation ratios at Q<=0.2

## How to apply

First complete Normalisr QC (qc_reads, qc_outlier with prior outlier-proportion bound 2% and Bonferroni P <= 1e-10) and the full normalization pipeline (lcpm, normcov, scaling_factor, compute_var, normvar), which removes nonlinear library-size confounders at both mean and variance levels. Then run the linear-model de function (normalisr.de / 'normalisr de') on the normalized expression matrix against the grouping or gRNA assignment matrix; in competition-aware CRISPR screens, include all untested gRNAs as fixed-effect covariates, reducing them to the top-500 principal components when their number exceeds 10,000, whereas the naive configuration omits these covariates. Extract log fold-changes, P values, and Q values for every tested pair; Normalisr provides exact P-value estimation without permutation and natively removes covariates as fixed effects. For null calibration, permute group labels (e.g. 100 random splits of 4,622 gRNA-free K562 cells), stratify genes into 10 expression bins, and test P values against uniformity with two-sided Kolmogorov-Smirnov tests. Estimate empirical FPR from NTC P-value histograms via Storey's method (expected ~4.8% naive vs ~1.5% competition-aware) and association inflation at Q<=0.2.

## Related tools

- **Normalisr** (Provides the linear-model differential expression function (de / normalisr.de) applied after its own QC and normalization steps (qc_reads, qc_outlier, lcpm, normcov, scaling_factor, compute_var, normvar)) — github.com/lingfeiwang/normalisr

## Examples

```
normalisr de -h  # see submodule help; python: import normalisr.normalisr as norm; ?norm.de
```

## Evaluation signals

- Null P values from label-permuted splits are uniformly distributed in two-sided KS tests within each of the 10 expression-level bins
- Storey-estimated FPR on non-targeting-control (NTC) gRNAs is low in the competition-aware run (~1.5%) versus inflated naive run (~4.8%)
- Output association table (de_associations.tsv) contains logFC, P value, and Q value for every gRNA-gene pair (13,186 gRNAs x 10,877 genes on the full-scale K562 CROP-seq screen)
- Association-inflation ratio computed at Q<=0.2 matches or improves on the published competition-aware benchmark
- Input matrices were preprocessed with the full Normalisr QC and normalization chain before running de (not raw counts)

## Limitations

- Only UMI-based datasets were analyzed; full-length platforms may produce a different read-count distribution and were not validated
- ScVI could not be compared because it does not recommend P-value-based FDR estimation; SCEPTRE NTC P values were unavailable for sensitivity comparison
- Requires enough (> 100) cells per test for robustness against read-count distribution distortions
- Does not reconstruct causal regulatory networks (v-structures, regularized regression, Granger causality) or handle multi-dataset batch effects

## Evidence

- [readme] exact P-value estimation without permutation: "linear association testing provides a unified inferential framework with several advantages: (i) exact P-value estimation without permutation, (ii) native removal of covariates (*e.g.* batches,"
- [intro] association step uses linear models: "The association step utilizes linear models to unify case–control DE, single-cell CRISPR screen analysis as multivariate DE, and gene co-expression network inference"
- [readme] robustness requires >100 cells: "robustness against read count distribution distortions with enough (> 100) cells, and (iv) computational efficiency"
- [abstract] unbiased P-value estimation across expression levels: "Normalisr achieves high sensitivity, specificity, speed, and generalizability across multiple scRNA-seq protocols and experimental conditions with unbiased p-value estimation"
- [discussion] not designed for nonlinear association testing: "Normalisr is not designed for hypothesis testing of arbitrary nonlinear associations54"
