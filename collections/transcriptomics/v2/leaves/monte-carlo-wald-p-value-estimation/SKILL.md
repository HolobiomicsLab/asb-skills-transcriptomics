---
name: monte-carlo-wald-p-value-estimation
description: Use when you need p-values (not just effect sizes) for per-cell-type
  between-condition transcriptomic distances from scDist mixed-model output, especially
  with few patients per group where MLE-based tests would be biased.
when_to_use_negative:
- Input is raw (unnormalized) counts — scDist requires normalized expression and cannot
  currently be applied directly to counts
- The question is differential abundance (proportion changes) rather than differential
  state — use DA-seq, Milo, or Meld instead
- Data are over-clustered — condition-specific effects can cause downward bias and
  false negatives as the number of clusters increases
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scDist
  role: Implements the PC projection, mixed-model fitting, spike-slab shrinkage, posterior
    distance sampling, and Monte Carlo Wald empirical p-value in one call
  repo: https://github.com/phillipnicol/scDist
- name: lme4
  role: Fits the per-PC linear mixed model (response fixed effect, patient random
    effect) whose Wald statistics feed the Monte Carlo p-value
- name: lmerTest
  role: Estimates the Satterthwaite degrees of freedom nu_k used in the Wald test
    statistic distribution
- name: irlba
  role: Truncated SVD to compute the top-K PCA loadings used as the plug-in estimator
    for U
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:e7667169553f4f48e183423fe878b2fcf29e08b42b22e9a7a7e8e8875ef8af4a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/phillipnicol/scDist
  tools:
  - irlba
  - lme4
  - lmerTest
  - scDist
  - simData
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# Monte Carlo Wald p-value estimation

## Summary

scDist's inference core: after shrinkage of squared PC loadings toward zero, the distance D_K is estimated from posterior samples and its null significance is obtained as an empirical Monte Carlo p-value computed from draws of the sum of Wald statistics, with Satterthwaite degrees of freedom from lmerTest. This provides statistically rigorous significance calls on cell-type transcriptomic distances despite small patient numbers.

## When to use

Use when you need p-values (not just effect sizes) for per-cell-type between-condition transcriptomic distances from scDist mixed-model output, especially with few patients per group where MLE-based tests would be biased.

## When NOT to use

- Input is raw (unnormalized) counts — scDist requires normalized expression and cannot currently be applied directly to counts
- The question is differential abundance (proportion changes) rather than differential state — use DA-seq, Milo, or Meld instead
- Data are over-clustered — condition-specific effects can cause downward bias and false negatives as the number of clusters increases

## Inputs

- Normalized single-cell count matrix (scTransform Pearson residuals recommended)
- Per-cell metadata with condition (fixed.effects, e.g. 'response'), replicate ('patient'), and cluster label ('clusters') columns

## Outputs

- Per-cell-type results data frame with columns: Dist., 95% CI (low), 95% CI (upper), p.val
- Empirical Monte Carlo p-value derived from draws of the sum of Wald statistics
- Posterior distribution of D_K from which the distance and CI are summarized

## How to apply

Project normalized counts onto the top-K truncated-SVD PCA loadings (irlba) and, for each PC k, fit an independent linear mixed model with lme4 using 'response' as the fixed effect and 'patient' as the random effect. Apply the spike-slab post-hoc Bayesian shrinkage to the fixed-effect coefficients (U beta)_k, because the MLE of the distance is upward-biased with few patients, and compute the posterior distribution of D_K. For the test, use the sum of Wald statistics across PCs; nu_k (the degrees of freedom) is estimated with Satterthwaite's approximation in lmerTest, and the empirical p-value is computed from Monte Carlo draws of the sum of Wald statistics. Read the per-cell-type p-value from scDist's results data frame alongside the estimated distance and 95% CI (e.g., cell types a and g recovered estimates ~33 with p ~ 1e-5, while unchanged cell type f was non-significant at p = 0.948).

## Related tools

- **scDist** (Implements the PC projection, mixed-model fitting, spike-slab shrinkage, posterior distance sampling, and Monte Carlo Wald empirical p-value in one call) — https://github.com/phillipnicol/scDist
- **lme4** (Fits the per-PC linear mixed model (response fixed effect, patient random effect) whose Wald statistics feed the Monte Carlo p-value)
- **lmerTest** (Estimates the Satterthwaite degrees of freedom nu_k used in the Wald test statistic distribution)
- **irlba** (Truncated SVD to compute the top-K PCA loadings used as the plug-in estimator for U)

## Examples

```
out <- scDist(sim$Y, sim$meta.data, fixed.effects="response", random.effects="patient", clusters="clusters"); out$results$p.val
```

## Evaluation signals

- Results data frame contains all four expected columns: Dist., 95% CI (low), 95% CI (upper), and p.val for every cell type
- On simulated data (scDist::simData(nct=10, N1=5, N2=5)), cell types with large true distances (a, g, true ~31) are recovered with estimates ~33 and significant p-values (~1e-5)
- The unchanged cell type f is correctly non-significant (p = 0.9478) with distance shrunk to 0
- Monte Carlo p-values reflect the empirical floor of 1/MC draws (e.g., 0.0000099999 in the demo), not exact zeros
- 95% CI bounds bracket the posterior distance estimate and CI width widens for cell types with fewer effective observations

## Limitations

- The MLE of the distance is substantially upward-biased with few patients, which is why post-hoc Bayesian shrinkage is applied before testing
- The test assumes a spherical-normal (shared variance) structure; violations could lead to miscalibrated standard errors and hypothesis tests
- Results are sensitive to the choice of the number of principal components K; no adaptive procedure exists (fixed heuristic of 20 or 50 PCs)
- Double dipping (using expression data for both annotation and distance estimation) and clustering errors can bias estimated distances and tests

## Evidence

- [discussion] nu_k Satterthwaite df: "νk is estimated using Satterthwaite’s approximation in lmerTest."
- [discussion] per-PC mixed model fitting: "Estimate (Uβ)k by using lme427 to fit the model (6) using the PC scores corresponding to the k-th loading"
- [other] post-hoc shrinkage rationale: [evidence span withheld — anchor status: not_found]
- [readme] results schema: "The results data frame gives a summary of the estimated distance and
uncertainty for each cell type"
- [readme] validation on simulation: "out <- scDist(sim$Y,sim$meta.data,fixed.effects = "response",
              random.effects="patient",
              clusters="clusters")"
