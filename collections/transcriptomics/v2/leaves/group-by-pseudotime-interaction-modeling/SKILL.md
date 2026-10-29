---
name: group-by-pseudotime-interaction-modeling
description: Use when performing XDE (covariate-associated differential expression
  along pseudotime) on scRNA-seq data with a trajectory method like Monocle2 that
  lacks native XDE support and needs group-specific intercept/trend tests.
when_to_use_negative:
- When the trajectory already comes from a method Lamian modules 1 and 2 directly
  support — the discussion notes Lamian modules 1 and 2 do not natively interoperate
  with Monocle2/Monocle3/slingshot and tailoring is deferred to future work, so this
  adaptation is only needed for such methods
- When the goal is testing expression changes along pseudotime without a group covariate
  (plain TDE) — use the null-model-only pseudotime spline test instead of the interaction
  model
edam_operation: http://edamontology.org/operation_3565
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Lamian
  role: Provides the XDE testing framework (permutation-based likelihood ratio test
    comparing nested GAMs) and the example dataset expdata used to benchmark the adapted
    Monocle2 model
  repo: https://github.com/Winnie09/Lamian
- name: Monocle
  role: Pseudotime analysis method whose GAM is tailored into the monocle2TrajTestCorr
    function to add group and group-by-pseudotime interaction terms
- name: scRNA-seq
  role: 'Data modality: single-cell expression data along a pseudotemporal trajectory
    is the input the interaction models are fit to'
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-42841-y
    title: A statistical framework for differential pseudotime analysis with multiple
      single-cell RNA-seq samples
  source_content_hash: sha256:5afe897aa9544ce0ff316dd026718bf69f1416e4c3302aaa91fe5b0c7f14b69d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Winnie09/Lamian
  tools:
  - Lamian
  - Monocle
  - scRNA-seq
  edam_operation: http://edamontology.org/operation_3565
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# group-by-pseudotime-interaction-modeling

## Summary

Adapts Monocle2-style generalized additive models to test covariate-associated differential expression along pseudotime (XDE) by fitting a null model g(E(Y)) ~ beta0 + f2(pseudotime) against a full model with a group factor, pseudotime spline, and their interaction, assessed via permutation-based likelihood ratio testing.

## When to use

Use when performing XDE (covariate-associated differential expression along pseudotime) on scRNA-seq data with a trajectory method like Monocle2 that lacks native XDE support and needs group-specific intercept/trend tests.

## When NOT to use

- When the trajectory already comes from a method Lamian modules 1 and 2 directly support — the discussion notes Lamian modules 1 and 2 do not natively interoperate with Monocle2/Monocle3/slingshot and tailoring is deferred to future work, so this adaptation is only needed for such methods
- When the goal is testing expression changes along pseudotime without a group covariate (plain TDE) — use the null-model-only pseudotime spline test instead of the interaction model

## Inputs

- Gene-by-cell expression matrix (e.g., Lamian example dataset expdata: 100 genes x 1000 cells)
- Per-cell pseudotime values from a trajectory reconstruction (Monocle2-style)
- Design matrix with intercept in column 1 and sample-group covariate in column 2 (testvar = 2)

## Outputs

- Per-gene likelihood-ratio statistic, p-value, and FDR.overall table (e.g., monocle2_xde_results.csv)
- Set of XDE genes defined by FDR.overall < 0.05

## How to apply

Load the expression data (e.g., the Lamian example dataset expdata with 100 genes and 1000 cells) and build a design matrix with the intercept in column 1 and the sample-group covariate in column 2 (testvar = 2). For each gene, fit the null generalized additive model g(E(Y)) = beta0 + f2(pseudotime) with a spline on pseudotime, then fit the full model g(E(Y)) = beta0 + f1(group) + f2(pseudotime) + f2(pseudotime)*f1(group), which adds the group factor and the group-by-pseudotime interaction so both mean shifts (intercept differences) and trend differences can be tested. Compute the likelihood-ratio statistic between the nested models and assess significance with a permutation-based likelihood ratio test (the article's XDE detection approach), using permuiter >= 100 permutations for real applications. Convert p-values to FDR and call XDE genes at FDR.overall < 0.05.

## Related tools

- **Lamian** (Provides the XDE testing framework (permutation-based likelihood ratio test comparing nested GAMs) and the example dataset expdata used to benchmark the adapted Monocle2 model) — https://github.com/Winnie09/Lamian
- **Monocle** (Pseudotime analysis method whose GAM is tailored into the monocle2TrajTestCorr function to add group and group-by-pseudotime interaction terms)
- **scRNA-seq** (Data modality: single-cell expression data along a pseudotemporal trajectory is the input the interaction models are fit to)

## Examples

```
Rscript monocle2_xde_test.R --expr expdata --testvar 2 --permuiter 100 --out monocle2_xde_results.csv
```

## Evaluation signals

- Output table contains a likelihood-ratio statistic, p-value, and FDR for each of the 100 example genes
- XDE gene set is exactly those with FDR.overall < 0.05
- Null model formula is g(E(Y)) ~ beta0 + f2(pseudotime) and full model adds f1(group) + f2(pseudotime) + f2(pseudotime)*f1(group) — interaction term must be present
- Permutation-based LRT uses permuiter >= 100 iterations for real-data runs
- Results are comparable to the Lamian XDE benchmark baseline when run on the same expdata input

## Limitations

- Monocle2 does not natively support XDE; this skill relies on a tailored adaptation (monocle2TrajTestCorr), and the paper notes that tailoring methods for seamless connection to Lamian modules 1 and 2 is deferred to future work
- The Lamian permutation-based testing framework (and hence this benchmark) does not account for all sources of uncertainty simultaneously — pseudotime inference uncertainty, gene expression/cell abundance uncertainty, and cross-sample variability remain open issues

## Evidence

- [discussion] XDE detection using permutation-based likelihood ratio test comparing nested models: "To conduct a hypothesis test comparing two models, we use a permutation-based likelihood ratio test."
- [discussion] XDE supports group and group-by-pseudotime covariates: "characterize how sample covariates modifies the pseudotemporal dynamics of cell abundance and gene expression"
- [discussion] Lamian modules do not interoperate with Monocle2/slingshot trajectories: "A future direction is to tailor these methods to improve their scalability and/or interoperability to allow their seamless connection to Lamian modules 1 and 2."
- [methods] expdata example dataset used for workflow demonstration: "we will use an example dataset **expdata** of 100 genes and 1000 cells to demonstration the workflow"
