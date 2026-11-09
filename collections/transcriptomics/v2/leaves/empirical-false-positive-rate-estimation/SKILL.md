---
name: empirical-false-positive-rate-estimation
description: Use when you must verify that a marker-gene or association test controls
  type-I error at the nominal 0.05 level across subject-level overdispersion levels
  — e.g. benchmarking NEBULA against naive glm.nb on unbalanced multi-subject single-cell
  groupings.
when_to_use_negative:
- Input is observed real expression data rather than simulated null counts — this
  skill estimates type-I error and requires replicates generated under the null model,
  not real data analysis
- Cells come from a single subject or the analysis has no multi-subject hierarchical
  structure, so there is no subject-level overdispersion component to sweep and no
  naive-vs-mixed-model contrast
- The goal is power or effect-size estimation under an alternative, not false-positive
  control under the null
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: nebula
  role: Fits the NEBULA NBGMM (nebula function with subject IDs as id) on each simulated
    gene to obtain the mixed-model p-values under the null; returns raw p-values and
    convergence codes for QC
  repo: https://github.com/lhe17/nebula
- name: glm.nb (MASS, via R)
  role: Provides the naive negative binomial regression baseline that ignores subject
    structure (treating cells as independent), whose FPR inflation is the benchmark
    against NEBULA
- name: group_cell
  role: Reorders cells so cells of the same subject are consecutive before passing
    the count matrix to nebula
  repo: https://github.com/lhe17/nebula
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-021-02146-6
    title: NEBULA is a fast negative binomial mixed model for differential or co-expression
      analysis of large-scale multi-subject single-cell data
  source_content_hash: sha256:b7b5799b5c3df706625410a68a95488e2fdd89697b86da02e42419af30f938d6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lhe17/nebula
  tools:
  - R
  - glm.nb (MASS, via R)
  - group_cell
  - nebula
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# empirical-false-positive-rate-estimation

## Summary

A simulation-based procedure for measuring the empirical false-positive rate of a regression method under a null mixed model, sweeping subject-level overdispersion to reveal type-I-error inflation. In the NEBULA paper it demonstrates that naive negative binomial regression inflates false positives on unbalanced multi-subject scRNA-seq data while NEBULA's NBGMM keeps FPR near 0.05.

## When to use

Use when you must verify that a marker-gene or association test controls type-I error at the nominal 0.05 level across subject-level overdispersion levels — e.g. benchmarking NEBULA against naive glm.nb on unbalanced multi-subject single-cell groupings.

## When NOT to use

- Input is observed real expression data rather than simulated null counts — this skill estimates type-I error and requires replicates generated under the null model, not real data analysis
- Cells come from a single subject or the analysis has no multi-subject hierarchical structure, so there is no subject-level overdispersion component to sweep and no naive-vs-mixed-model contrast
- The goal is power or effect-size estimation under an alternative, not false-positive control under the null

## Inputs

- Real-data grouping template: per-cell subject IDs and cluster/subcluster labels (e.g. astrocyte metadata from syn21261143 with AST0–AST3 labels)
- Simulation grid: subject-level overdispersion sigma^2 values {0, 0.02, 0.05, 0.1} with cell-level overdispersion fixed at 1
- Gene-by-cell count matrix (integer or sparse dgCMatrix) cells grouped consecutively by subject for nebula input

## Outputs

- Vector of raw p-values for the null predictor per replicate, method (glm.nb vs NEBULA), and sigma^2 condition
- Empirical false-positive rate per method and sigma^2 condition (fraction of p-values < 0.05)
- Tabulation/plot of empirical FPR versus subject-level overdispersion comparing naive NB regression with NEBULA (reproducing the Fig 5e comparison)

## How to apply

Extract the empirical grouping structure from the real dataset (e.g. the 3386 astrocyte cells' subject IDs and AST0–AST3 subcluster labels, noting AST3 derives from only ~4 subjects, so cells-per-subject is highly unbalanced). For each subject-level overdispersion sigma^2 in {0, 0.02, 0.05, 0.1} with cell-level overdispersion fixed at 1, simulate gene counts under the null negative binomial mixed model using the empirical cell-per-subject and cell-per-cluster group sizes. Analyze each simulated gene twice: a naive negative binomial regression via glm.nb that treats cells as independent, and the NEBULA NBGMM via the nebula function with subject IDs passed as id. Collect raw p-values for the null predictor across replicates and compute the empirical FPR as the fraction of p-values below the nominal 0.05 threshold per sigma^2 condition and method. Correct application is judged by the expected contrast: glm.nb FPR inflates as sigma^2 rises from 0 to 0.1 (inflation already visible at sigma^2 = 0.05), while NEBULA's FPR stays near 0.05; flag and exclude replicates with nebula convergence codes <= -20.

## Related tools

- **nebula** (Fits the NEBULA NBGMM (nebula function with subject IDs as id) on each simulated gene to obtain the mixed-model p-values under the null; returns raw p-values and convergence codes for QC) — https://github.com/lhe17/nebula
- **glm.nb (MASS, via R)** (Provides the naive negative binomial regression baseline that ignores subject structure (treating cells as independent), whose FPR inflation is the benchmark against NEBULA)
- **group_cell** (Reorders cells so cells of the same subject are consecutive before passing the count matrix to nebula) — https://github.com/lhe17/nebula

## Examples

```
re = nebula(sample_data$count, sample_data$sid, pred = df, ncore = 1); fpr <- mean(re$summary$p_X < 0.05)
```

## Evaluation signals

- NEBULA's empirical FPR stays near the nominal 0.05 level as sigma^2 increases from 0 to 0.1
- Naive glm.nb FPR inflates with subject-level overdispersion, with inflation already beginning at sigma^2 = 0.05 and rising rapidly thereafter
- Replicates with nebula convergence code <= -20 are excluded or flagged as non-convergent before tabulating p-values
- Genes with estimated cell-level overdispersion >100 (or subject-level overdispersion >1 when testing subject-level variables) are filtered before interpretation
- Empirical FPR is computed as the fraction of raw (unadjusted) p-values below 0.05 per sigma^2 condition and method

## Limitations

- Misspecification of the random-effects distribution may itself lead to inflated FPR in testing a subject-level predictor when the number of subjects is small
- NEBULA-HL shows a downward bias in estimating the subject-level overdispersion, and NEBULA-LN sacrifices some asymptotic efficiency in estimating cell-level overdispersion
- Genes with a very large subject-level overdispersion (>1) should be removed or interpreted cautiously when the variable of interest is subject-level
- Testing a subject-level variable is highly sensitive to estimation of the subject-level variance component when the number of subjects is small (e.g. the AST3 cluster drawing from ~4 subjects)

## Evidence

- [intro] NEBULA reduced false positives in selecting cell-type marker genes: "We showed that NEBULA reduced false positives in selecting cell-type marker genes, especially when the numbers of cells across subjects were unbalanced"
- [readme] Raw p-values from nebula: "The p-values returned by `nebula` are raw p-values (not adjusted for multiple testing)."
- [methods] Convergence QC threshold: "Generally, a convergence code $≤$ -20 suggests that the algorithm does not converge well. The results should be interpreted with caution in these cases."
- [discussion] Random-effects misspecification inflates FPR: "misspecification of the distribution of the random effects may also lead to inflated FPR in testing a subject-level predictor"
- [methods] Subject-level overdispersion filter: "If the variable of interest is subject-level, genes with a very large subject-level overdispersion (>1) should be removed or interpreted cautiously as well."
