---
name: monocle2-generalized-additive-model-fitting
description: Use when a Monocle2-style single-sample pseudotime analysis must be extended
  to covariate-associated differential expression (XDE), testing whether a sample-level
  group covariate shifts gene expression intercepts or pseudotime trends.
when_to_use_negative:
- When analyzing multi-sample scRNA-seq where inference must account for cross-sample
  variability — Lamian draws statistical inference after accounting for sample-level
  variation, which this Monocle2-style adaptation does not
- When the trajectory was inferred with Monocle3 or Slingshot rather than Monocle2-style
  models — Lamian modules 1 and 2 do not support those trajectories, and tailoring
  them is deferred to future work
- When the question is plain temporal differential expression (TDE) along pseudotime
  without a group covariate, rather than group-by-pseudotime XDE testing
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Monocle
  role: Pseudotime method whose single-sample gene expression model is tailored into
    the new function monocle2TrajTestCorr for XDE testing
- name: Lamian
  role: Framework whose workflow and expdata example dataset the adapted Monocle2
    model serves as a benchmark baseline; provides the permutation-based likelihood
    ratio test strategy
  repo: https://github.com/Winnie09/Lamian
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-42841-y
    title: A statistical framework for differential pseudotime analysis with multiple
      single-cell RNA-seq samples
  source_content_hash: sha256:e688ab6d47288d0a4238eb33039c9c9abe5afb969a10335c421218f0c45428dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Winnie09/Lamian
  tools:
  - Lamian
  - Monocle
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# monocle2-generalized-additive-model-fitting

## Summary

Adapts the Monocle2 pseudotime expression model into generalized additive models compared by permutation-based likelihood ratio test, enabling Monocle2 — which does not natively support XDE — to test group-specific intercept and trend differences along pseudotime.

## When to use

Use when a Monocle2-style single-sample pseudotime analysis must be extended to covariate-associated differential expression (XDE), testing whether a sample-level group covariate shifts gene expression intercepts or pseudotime trends.

## When NOT to use

- When analyzing multi-sample scRNA-seq where inference must account for cross-sample variability — Lamian draws statistical inference after accounting for sample-level variation, which this Monocle2-style adaptation does not
- When the trajectory was inferred with Monocle3 or Slingshot rather than Monocle2-style models — Lamian modules 1 and 2 do not support those trajectories, and tailoring them is deferred to future work
- When the question is plain temporal differential expression (TDE) along pseudotime without a group covariate, rather than group-by-pseudotime XDE testing

## Inputs

- Gene-by-cell expression matrix from the Lamian example dataset expdata (100 genes, 1000 cells)
- Per-cell pseudotime values along the inferred trajectory
- Per-cell sample-group covariate (design matrix: intercept in column 1, group in column 2, testvar = 2)

## Outputs

- monocle2_xde_results.csv containing per-gene likelihood-ratio statistic, p-value, and FDR.overall for all 100 genes
- XDE gene set defined by FDR.overall < 0.05

## How to apply

Load the example dataset expdata (100 genes, 1000 cells) and build the design matrix with the intercept in column 1 and the sample-group covariate in column 2 (testvar = 2). For each gene, fit the null Monocle2-style generalized additive model g(E(Y)) = beta0 + f2(pseudotime) with a spline on pseudotime, then fit the full model g(E(Y)) = beta0 + f1(group) + f2(pseudotime) + f2(pseudotime)*f1(group), which adds the group factor, the pseudotime spline, and their interaction so both mean-shift (intercept) and group-by-pseudotime trend differences are parameterized. Compare the nested models with a permutation-based likelihood ratio test, using permuiter >= 100 as recommended for real applications. Convert p-values to FDR and call XDE genes at FDR.overall < 0.05, writing per-gene statistics to monocle2_xde_results.csv. This adapted model is used as a benchmark baseline because Monocle2 itself does not natively support XDE analysis.

## Related tools

- **Monocle** (Pseudotime method whose single-sample gene expression model is tailored into the new function monocle2TrajTestCorr for XDE testing)
- **Lamian** (Framework whose workflow and expdata example dataset the adapted Monocle2 model serves as a benchmark baseline; provides the permutation-based likelihood ratio test strategy) — https://github.com/Winnie09/Lamian

## Examples

```
R: null <- g(E(Y)) ~ beta0 + f2(pseudotime); full <- g(E(Y)) ~ beta0 + f1(group) + f2(pseudotime) + f2(pseudotime)*f1(group); permutation LRT with permuiter >= 100
```

## Evaluation signals

- Output table contains one likelihood-ratio statistic, p-value, and FDR for each of the 100 genes in expdata
- XDE gene set is exactly the genes with FDR.overall < 0.05
- Nested model structure is correct: null g(E(Y)) ~ beta0 + f2(pseudotime) is strictly contained in the full model that adds f1(group), f2(pseudotime), and f2(pseudotime)*f1(group)
- Permutation iterations meet permuiter >= 100 for real applications
- Design matrix has the intercept in column 1 and the group covariate in column 2 (testvar = 2)

## Limitations

- Monocle2 does not natively support XDE; this is a tailored adaptation rather than an off-the-shelf method, and it is used as a benchmark baseline
- Lamian modules 1 and 2 do not support Monocle2, Monocle3, or Slingshot trajectories natively; tailoring these methods for seamless connection is deferred to future work
- Permutation-based testing requires sufficient iterations (permuiter >= 100 for real applications), so runtime scales with the number of genes and permutations

## Evidence

- [discussion] To conduct a hypothesis test comparing two models, we use a permutation-based likelihood ratio test.: "To conduct a hypothesis test comparing two models, we use a permutation-based likelihood ratio test."
- [methods] we will use an example dataset **expdata** of 100 genes and 1000 cells to demonstration the workflow: "we will use an example dataset **expdata** of 100 genes and 1000 cells to demonstration the workflow"
- [readme] Lamian accounts for variability across biological samples which other existing pseudotime methods do not consider.: "Lamian accounts for variability across biological samples which other existing pseudotime methods do not consider."
