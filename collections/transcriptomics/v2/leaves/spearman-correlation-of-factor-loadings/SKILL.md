---
name: spearman-correlation-of-factor-loadings
description: Use when Tensor-cell2cell (or similar TCA) factor loadings must be compared
  against a reference decomposition or a context-level phenotype such as COVID-19
  severity, and a rank-based, distribution-free agreement metric is needed.
when_to_use_negative:
- The tensors were built with different LR-pair lists or cell-type subsets — the article
  notes the COVID-19 and ASD tensors contained only cell types shared across all samples,
  so loadings are not comparable to the reference decomposition otherwise
- A linear, magnitude-sensitive agreement measure is required rather than a rank-based
  one, or loadings are on incomparable scales after unconstrained (non-nonnegative)
  decomposition
- Only one factor exists or the decomposition rank differs substantially from 10,
  making the reference factor matching (factors 1, 2, 8) meaningless
tools:
- name: Tensor-cell2cell
  role: Produces the factor loadings whose agreement is assessed by Spearman correlation;
    run via the cell2cell.tensor module with rank=10 and init='svd' (falling back
    to init='random').
  repo: https://github.com/earmingol/cell2cell
- name: cell2cell
  role: Python suite that packages Tensor-cell2cell and its downstream analysis functions
    (cell2cell.analysis, cell2cell.stats) used to compute factor-level statistics.
  repo: https://github.com/earmingol/cell2cell
- name: CellChat
  role: Generates the per-context communication scores (using the Human-2020-Jin LR
    pair list) that populate the 4D tensor whose factor loadings are correlated.
  repo: https://github.com/sqjin/CellChat
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-31369-2
    title: Context-aware deconvolution of cell–cell communication with Tensor-cell2cell
  source_content_hash: sha256:65e38a10b833c32376c01f3eb584e30b0a566376f9f726cda59e87258fd2720e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/earmingol/cell2cell
  tools:
  - CellChat
  - Tensor-cell2cell
  - cell2cell
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Spearman correlation of factor loadings

## Summary

Computes Spearman rank correlations between the context (or ligand-receptor) loadings of tensor-decomposition factors and an external reference decomposition or a phenotypic variable (e.g., COVID-19 severity), to quantify how well a factor reproduces or tracks a known pattern.

## When to use

Use when Tensor-cell2cell (or similar TCA) factor loadings must be compared against a reference decomposition or a context-level phenotype such as COVID-19 severity, and a rank-based, distribution-free agreement metric is needed.

## When NOT to use

- The tensors were built with different LR-pair lists or cell-type subsets — the article notes the COVID-19 and ASD tensors contained only cell types shared across all samples, so loadings are not comparable to the reference decomposition otherwise
- A linear, magnitude-sensitive agreement measure is required rather than a rank-based one, or loadings are on incomparable scales after unconstrained (non-nonnegative) decomposition
- Only one factor exists or the decomposition rank differs substantially from 10, making the reference factor matching (factors 1, 2, 8) meaningless

## Inputs

- Context (patient/sample) loading vector per factor from a rank-10 Tensor-cell2cell decomposition of the 12x189x6x6 BALF COVID-19 communication tensor
- Reference factor loadings from the published decomposition (e.g., from the Code Ocean capsule doi:10.24433/CO.0051950.v2)
- Optional: context-level severity labels (3 control, 3 moderate, 6 severe patients from GSE145926) for phenotype correlation

## Outputs

- Spearman rank correlation coefficient per matched factor pair (expected 0.72 for factor 1, 0.61 for factor 2, 0.92 for factor 8 vs. reference)
- Factor-matching map between run and reference decompositions
- Rows appended to tensor_factorization_results.csv containing the Spearman correlations alongside factor loadings and the factor-9 Gini coefficient (~0.09 +/- 0.02)

## How to apply

After running non-negative tensor factorization of the 4D BALF COVID-19 communication tensor (12 contexts x 189 LR pairs x 6 sender x 6 receiver cell types) at rank=10 with Tensor-cell2cell, extract the context loadings for each factor. Align factors between your run and the reference decomposition, then compute Spearman rank correlations of the matched loadings; in the reference workflow, factor 1, 2, and 8 context loadings were expected to correlate at 0.72, 0.61, and 0.92 respectively against the paper's decomposition, with factor 8 most correlated with disease severity (Spearman 0.92, Fig. 4c). Because tensor decomposition is unsupervised, factor order is arbitrary, so the correlation step doubles as factor-matching: the pair of factors with the highest Spearman coefficient is treated as equivalent. Acceptance uses a +/-0.05 tolerance around the reference coefficients, so re-run with init='random' if init='svd' fails (e.g., on memory errors) and check that correlations remain within tolerance.

## Related tools

- **Tensor-cell2cell** (Produces the factor loadings whose agreement is assessed by Spearman correlation; run via the cell2cell.tensor module with rank=10 and init='svd' (falling back to init='random').) — https://github.com/earmingol/cell2cell
- **cell2cell** (Python suite that packages Tensor-cell2cell and its downstream analysis functions (cell2cell.analysis, cell2cell.stats) used to compute factor-level statistics.) — https://github.com/earmingol/cell2cell
- **CellChat** (Generates the per-context communication scores (using the Human-2020-Jin LR pair list) that populate the 4D tensor whose factor loadings are correlated.) — https://github.com/sqjin/CellChat

## Examples

```
pip install cell2cell
```

## Evaluation signals

- Spearman correlations of factor 1, 2, and 8 context loadings vs. reference fall within +/-0.05 of 0.72, 0.61, and 0.92 respectively
- Factor 8 is the top-correlated factor with disease severity, with a Spearman coefficient near 0.92
- Correlation values are stable enough that switching init from 'svd' to 'random' still lands within the acceptance tolerance
- Result file tensor_factorization_results.csv is populated with the correlations, the ~0.09 factor-9 Gini coefficient, and the expected top LR pairs (APP/MDK/MIF/CD99, CCL2/CCL3/CCL8–CCR1/CCR2/CCR5, MIF–CD74/CD44)

## Limitations

- Tensor decomposition is unsupervised, so factor indices are not guaranteed to match between runs; correlations must be interpreted as factor matching, and moderate factors (e.g., 0.61) reflect run-to-run variability
- The COVID-19 and ASD case-study tensors only include cell types shared across all samples, potentially missing rare cell types and biasing loadings
- The implemented tensor decomposition algorithm is not optimized for missing values, so LR pairs or contexts absent in some contexts cannot be handled directly
- Reproducible reference loadings depend on exact code/software versions, which were deposited in a Code Ocean capsule (doi:10.24433/CO.0051950.v2)

## Evidence

- [abstract] Tensor-cell2cell uncovers context-driven patterns of communication associated with different phenotypic states: [evidence span withheld — anchor status: not_found]
- [abstract] Tensor-cell2cell identifies multiple communication modules linked to severities of Coronavirus Disease 2019: "Tensor-cell2cell can identify multiple modules associated with distinct communication processes (e.g., participating cell-cell and ligand-receptor pairs) linked to severities of Coronavirus Disease"
- [discussion] Downstream analyses include statistical comparison of factors: "such analyses included GSEA, clustering, visualization and statistical comparison of factors"
- [methods] cell2cell provides downstream analysis functions for Tensor-cell2cell: "cell2cell.analysis : Includes simplified pipelines for running the analyses, and functions for downstream analyses of Tensor-cell2cell"
- [discussion] Reproducibility depends on deposited code and software versions: "we deposited all our analyses including data and exact versions of code and software in a Code Ocean capsule"
- [readme] CellChat DB provides curated LR pairs used to build communication scores: [evidence span withheld — anchor status: external_doc]
