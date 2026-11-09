---
name: nested-model-likelihood-ratio-testing
description: Use when testing whether adding a sample covariate (group, condition)
  or its interaction with pseudotime significantly improves a gene-level generalized
  model, e.g., to call XDE/TDE genes.
when_to_use_negative:
- When the compared models are not nested (the likelihood-ratio statistic is only
  valid between nested null and full models).
- When the goal is trajectory inference itself (Monocle/TSCAN/Slingshot) rather than
  testing covariate-associated expression differences along pseudotime.
- When sample-level variability must be modeled natively by Lamian's multi-sample
  framework — this skill's Monocle2 adaptation (monocle2TrajTestCorr) exists because
  Monocle2 does not natively support XDE analysis and Lamian modules 1 and 2 do not
  support Monocle2/3 or Slingshot trajectories.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: Lamian
  role: Provides the permutation-based likelihood ratio test framework for comparing
    nested null and full models in differential pseudotime analysis
  repo: https://github.com/Winnie09/Lamian
- name: Monocle
  role: Trajectory/pseudotime framework adapted into monocle2TrajTestCorr so group
    intercept and trend differences along pseudotime can be tested
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-42841-y
    title: A statistical framework for differential pseudotime analysis with multiple
      single-cell RNA-seq samples
  source_content_hash: sha256:56fca56a68509da76b5640a8d5db442c1d81a067c42c082a57fa057110f6c053
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Winnie09/Lamian
  tools:
  - Lamian
  - Monocle
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# Nested-model likelihood-ratio testing

## Summary

Hypothesis-testing strategy used by Lamian (and its Monocle2 adaptation monocle2TrajTestCorr) in which a null generalized model is nested within a full model and a permutation-based likelihood ratio test assesses whether the added covariate terms (e.g., group, group-by-pseudotime interaction) significantly improve fit.

## When to use

Use when testing whether adding a sample covariate (group, condition) or its interaction with pseudotime significantly improves a gene-level generalized model, e.g., to call XDE/TDE genes.

## When NOT to use

- When the compared models are not nested (the likelihood-ratio statistic is only valid between nested null and full models).
- When the goal is trajectory inference itself (Monocle/TSCAN/Slingshot) rather than testing covariate-associated expression differences along pseudotime.
- When sample-level variability must be modeled natively by Lamian's multi-sample framework — this skill's Monocle2 adaptation (monocle2TrajTestCorr) exists because Monocle2 does not natively support XDE analysis and Lamian modules 1 and 2 do not support Monocle2/3 or Slingshot trajectories.

## Inputs

- Gene-by-cell expression matrix (e.g., Lamian example dataset expdata with 100 genes and 1000 cells)
- Per-cell pseudotime values from a reconstructed trajectory
- Design matrix with intercept in column 1 and sample-group covariate in column 2 (testvar = 2)
- Number of permutations (permuiter, >= 100 recommended for real data)

## Outputs

- Per-gene likelihood-ratio statistics
- Per-gene permutation-based p-values
- FDR values (FDR.overall) used to define the XDE gene set at FDR.overall < 0.05
- Results table saved as monocle2_xde_results.csv

## How to apply

For each gene, fit a null model g(E(Y)) ~ beta0 + f2(pseudotime) with a spline on pseudotime, then fit the full model g(E(Y)) ~ beta0 + f1(group) + f2(pseudotime) + f2(pseudotime)*f1(group), which adds the group factor and the group-by-pseudotime interaction so that group-specific intercept (mean shift) and trend differences along pseudotime can be tested. Compute the likelihood-ratio statistic between the nested null and full models and assess significance via a permutation-based likelihood ratio test, using at least 100 (permuiter >= 100) permutations for real applications. Finally convert p-values to FDR and call significant (XDE) genes at the FDR.overall < 0.05 cutoff. In the Lamian benchmark, the design matrix carries the intercept in column 1 and the sample-group covariate in column 2 (testvar = 2).

## Related tools

- **Lamian** (Provides the permutation-based likelihood ratio test framework for comparing nested null and full models in differential pseudotime analysis) — https://github.com/Winnie09/Lamian
- **Monocle** (Trajectory/pseudotime framework adapted into monocle2TrajTestCorr so group intercept and trend differences along pseudotime can be tested)

## Examples

```
devtools::install_github("Winnie09/Lamian"); data(expdata)  # 100 genes x 1000 cells; fit null g(E(Y)) ~ beta0 + f2(pseudotime) vs full ~ beta0 + f1(group) + f2(pseudotime) + interaction; permuiter >= 100; call XDE at FDR.overall < 0.05
```

## Evaluation signals

- The output table contains a likelihood-ratio statistic, p-value, and FDR for each of the 100 genes in the expdata example.
- The XDE gene set is exactly the set of genes with FDR.overall < 0.05.
- permuiter is set to >= 100 permutations as recommended for real applications (fewer only acceptable for quick demos).
- The full model strictly contains all null-model terms (intercept + pseudotime spline), confirming model nesting.
- Saved results file monocle2_xde_results.csv includes per-gene statistics, p-values, and FDR values.

## Limitations

- Monocle2 does not natively support XDE analysis; the monocle2TrajTestCorr function is a custom adaptation, and Lamian modules 1 and 2 do not seamlessly connect to Monocle2, Monocle3, or Slingshot trajectories (tailoring deferred to future work).
- Permutation-based testing is computationally costly; fewer than ~100 permutations gives unstable p-values for real applications.
- Significance calls depend on the chosen FDR cutoff (0.05) and the number of permutations used.

## Evidence

- [other] Monocle2 was adapted for XDE detection by tailoring its model into the new function monocle2TrajTestCorr: [evidence span withheld — anchor status: not_found]
- [discussion] Permutation-based LRT is Lamian's model comparison strategy: "To conduct a hypothesis test comparing two models, we use a permutation-based likelihood ratio test."
- [methods] expdata example dataset scale: "we will use an example dataset **expdata** of 100 genes and 1000 cells to demonstration the workflow"
- [discussion] Permutation scheme for TDE accounts for pseudotime variability within samples: "In each permutation, we first bootstrap the cells to account for pseudotime variability, and we then permute the pseudotime of the cells within each sample."
- [readme] Lamian installation via GitHub: [evidence span withheld — anchor status: external_doc]
