---
name: co-expression-estimation-bias-curve-generation
description: Use when benchmarking co-expression estimators on permuted null UMI count
  data (z_i ~ F_p(mu, Sigma), x_ij | z_ij ~ Poisson(s_i * z_ij)) to visualize depth-induced
  bias, e.g. mean-expression vs. average estimated co-expression curves.
when_to_use_negative:
- Data contain true co-expression signal (non-null data) — the curve's deviation from
  zero is only interpretable as bias when no gene pairs are truly co-expressed
- Input is normalized or transformed expression rather than raw UMI counts — several
  methods in the benchmark require raw counts and normalization can itself induce
  the depth bias being tested
- Goal is testing type-I error calibration via p-value Q-Q plots rather than estimation
  bias — use the Q-Q / BH-correction workflow (Fig. 1b) instead
edam_operation: http://edamontology.org/operation_3438
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: CS-CORE / CSCORE (R package)
  role: Primary unbiased co-expression estimator run on raw UMI counts of each permuted
    null dataset; its flat, zero-centered curve is the reference against which other
    methods' bias is judged
  repo: https://github.com/ChangSuBiostats/CS-CORE
- name: Seurat
  role: Loads and subsets the PBMC COVID Seurat object (blish_covid.seu.rds) used
    to generate permuted null count datasets
- name: locCSN
  role: One of the comparison co-expression estimators whose bias curve is plotted
    against CS-CORE's
  repo: https://github.com/xuranw/locCSN
- name: Noise Regularization
  role: Comparison preprocessing/estimation method whose bias curve is evaluated on
    the null datasets
  repo: https://github.com/RuoyuZhang/NoiseRegularization
- name: Normalisr
  role: Comparison normalization/co-expression method whose bias curve is evaluated
    on the null datasets
  repo: https://github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-40503-7
    title: Cell-type-specific co-expression inference from single cell RNA-sequencing
      data
  source_content_hash: sha256:e08c17c5c92ceebeab79f07db3c4142bfd659a56d681c9f3e14a79cf38a1cb64
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ChangSuBiostats/CS-CORE
  tools:
  - CS-CORE
  - CS-CORE / CSCORE (R package)
  - CSCORE
  - Noise Regularization
  - Normalisr
  - Seurat
  - locCSN
  edam_operation: http://edamontology.org/operation_3438
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# co-expression-estimation-bias-curve-generation

## Summary

Generates sequencing-depth bias curves (Fig. 1a of Su et al. 2023, Nature Communications) by averaging a method's co-expression estimates over gene pairs binned by mean expression on permuted null scRNA-seq data, revealing whether estimates drift away from zero as sequencing depth varies.

## When to use

Use when benchmarking co-expression estimators on permuted null UMI count data (z_i ~ F_p(mu, Sigma), x_ij | z_ij ~ Poisson(s_i * z_ij)) to visualize depth-induced bias, e.g. mean-expression vs. average estimated co-expression curves.

## When NOT to use

- Data contain true co-expression signal (non-null data) — the curve's deviation from zero is only interpretable as bias when no gene pairs are truly co-expressed
- Input is normalized or transformed expression rather than raw UMI counts — several methods in the benchmark require raw counts and normalization can itself induce the depth bias being tested
- Goal is testing type-I error calibration via p-value Q-Q plots rather than estimation bias — use the Q-Q / BH-correction workflow (Fig. 1b) instead

## Inputs

- Raw UMI count matrix from a Seurat object (e.g. blish_covid.seu.rds, PBMC COVID from Wilk et al.)
- Permuted null count datasets simulated under the CS-CORE generative model with varying sequencing depths s_i
- Co-expression estimate matrices from CS-CORE and the ten comparison methods (baredSC, locCSN, Noise Regularization, Normalisr, Pearson, propr, Spearman, SpQN, rho-analytic PR, rho-sctransform)

## Outputs

- Per-method bias curve: gene-pair mean expression bins vs. average estimated co-expression (Fig. 1a)
- Bias assessment: whether average co-expression estimates are centered around zero across sequencing-depth/expression bins

## How to apply

First generate permuted null count datasets from a Seurat object such as the PBMC COVID data (blish_covid.seu.rds) under the CS-CORE generative model, where sequencing depth s_i varies across cells but no gene pairs are truly co-expressed. Run each estimator on the raw UMI counts — CS-CORE via CSCORE(), plus comparison methods including baredSC, locCSN, Noise Regularization, Normalisr, Pearson, propr, Spearman, SpQN, rho-analytic PR, and rho-sctransform. For each method, bin gene pairs by mean expression and average the co-expression estimates within each bin. Plot mean expression (x-axis) against average estimated co-expression (y-axis) to form the bias curve. Rationale: on null data all curves should center at zero; a method whose curve trends away from zero across expression bins has depth-induced bias, as most existing methods did, whereas CS-CORE's estimates remained unbiased and centered around zero.

## Related tools

- **CS-CORE / CSCORE (R package)** (Primary unbiased co-expression estimator run on raw UMI counts of each permuted null dataset; its flat, zero-centered curve is the reference against which other methods' bias is judged) — https://github.com/ChangSuBiostats/CS-CORE
- **Seurat** (Loads and subsets the PBMC COVID Seurat object (blish_covid.seu.rds) used to generate permuted null count datasets)
- **locCSN** (One of the comparison co-expression estimators whose bias curve is plotted against CS-CORE's) — https://github.com/xuranw/locCSN
- **Noise Regularization** (Comparison preprocessing/estimation method whose bias curve is evaluated on the null datasets) — https://github.com/RuoyuZhang/NoiseRegularization
- **Normalisr** (Comparison normalization/co-expression method whose bias curve is evaluated on the null datasets) — https://github.com/lingfeiwang/normalisr

## Examples

```
library(CSCORE); library(Seurat); CSCORE_result <- CSCORE(null_counts, genes = genes_selected)  # run on each permuted null dataset, then bin pairs by mean expression and average co-expression estimates
```

## Evaluation signals

- On truly null permuted data, the bias curve for a calibrated method should be flat and centered around zero co-expression across all mean-expression bins — CS-CORE satisfies this in Fig. 1a
- A curve trending away from zero at higher sequencing depth/expression bins indicates depth-induced estimation bias, as observed for most existing methods
- Bias curves must be computed from estimates on raw UMI counts, not normalized matrices, to reflect the intended confounding mechanism
- The simulation must ensure no gene pairs are truly co-expressed (zero off-diagonal correlation in Sigma) so that any non-zero average estimate is attributable to bias
- Curves should be averaged over gene pairs within bins, and results should be reproducible across independently permuted null replicates

## Limitations

- The null simulation uses only a Poisson measurement model x_ij | z_ij ~ Poisson(s_i * z_ij); CS-CORE's measurement model does not cover other count distributions such as the negative binomial, which the authors defer to future work
- Bias-curve interpretation assumes the CS-CORE generative model (cells of the same cell type share one expression distribution), which may not hold across individuals with different genetic, demographic and clinical characteristics
- Conclusions are specific to sequencing-depth confounding; other confounders (e.g. batches, covariates) are handled separately via covariate adjustment

## Evidence

- [methods] Permuted null generation and bias-curve protocol: "we use the single cell RNA-sequencing data on Peripheral blood mononuclear cells (PBMC) from COVID patients and healthy controls from Wilk et al."
- [intro] Depth confounding rationale: "gene co-expressions measured via correlations of UMI counts across cells can be seriously confounded by varying sequencing depths, resulting in inflated false positive findings"
- [abstract] Bias finding: "most existing methods suffered from inflated false positives as well as biased co-expression estimates and clustering analysis"
- [methods] CS-CORE estimator invocation: "CSCORE_result <- CSCORE(pbmc_B_healthy, genes = genes_selected)"
- [abstract] CS-CORE accuracy claim: "whereas CS-CORE gave accurate estimates in these experiments"
