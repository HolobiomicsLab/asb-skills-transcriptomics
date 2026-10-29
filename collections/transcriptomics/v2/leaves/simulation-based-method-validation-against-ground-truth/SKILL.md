---
name: simulation-based-method-validation-against-ground-truth
description: 'Use when a new distance-estimation or differential-state method must
  be checked for accuracy and calibration: simulated data with known per-cell-type
  true distances are available or can be generated (here via scDist::simData with
  nct=10, N1=N2=5).'
when_to_use_negative:
- 'Data are over-clustered: the article shows over-clustering makes scDist produce
  false negatives, with median distance decaying toward 0 as cluster number increases,
  so simulated-validation conclusions will not transfer'
- 'You need a cluster-free differential-state test: the article states designing a
  cluster-free extension of scDist is future work, so validation against clustered
  simulation ground truth cannot certify such a setting'
- 'Only raw (unnormalized) counts are available: scDist cannot currently be applied
  directly to raw counts, so the simulation must supply normalized data (scTransform
  Pearson residuals recommended)'
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scDist
  role: 'Method under test: estimates between-condition cell-type distances with per-PC
    mixed models, empirical-Bayes shrinkage, and Monte Carlo Wald tests; provides
    simData for ground-truth simulation'
  repo: github.com/phillipnicol/scDist
- name: simData
  role: Generates the fully simulated dataset (nct=10, N1=5, N2=5) with normalized
    counts, metadata, and ground-truth distances D.true
  repo: github.com/phillipnicol/scDist
- name: lme4
  role: Fits the per-PC linear mixed model (response fixed effect, patient random
    effect) on PC scores
- name: lmerTest
  role: Provides Satterthwaite approximation of the degrees of freedom nu_k used in
    the Wald-test p-values
- name: irlba
  role: Truncated SVD used to compute the top-K PCA loadings onto which cells are
    projected to obtain PC scores
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:71954ace670644dc839468132a4143134d35a6eb6fde959a9f97cee2414fbea3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - irlba
  - lme4
  - lmerTest
  - scDist
  - simData
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# simulation-based-method-validation-against-ground-truth

## Summary

Validate scDist's distance inference by generating a fully simulated single-cell dataset with known ground-truth between-condition distances (simData), running the full PC-projection plus mixed-model pipeline, and comparing estimated distances and p-values against D.true per cell type.

## When to use

Use when a new distance-estimation or differential-state method must be checked for accuracy and calibration: simulated data with known per-cell-type true distances are available or can be generated (here via scDist::simData with nct=10, N1=N2=5).

## When NOT to use

- Data are over-clustered: the article shows over-clustering makes scDist produce false negatives, with median distance decaying toward 0 as cluster number increases, so simulated-validation conclusions will not transfer
- You need a cluster-free differential-state test: the article states designing a cluster-free extension of scDist is future work, so validation against clustered simulation ground truth cannot certify such a setting
- Only raw (unnormalized) counts are available: scDist cannot currently be applied directly to raw counts, so the simulation must supply normalized data (scTransform Pearson residuals recommended)

## Inputs

- Simulated normalized count matrix from scDist::simData (e.g., 1000 genes x 5100 cells, 10 cell types, 5 patients per group)
- Per-cell metadata data frame with columns: response (condition), patient (replicate), clusters (cell type)
- Ground-truth between-condition distances sim$D.true for each simulated cell type

## Outputs

- Per-cell-type results data frame with columns: Dist. (estimated distance), 95% CI (low), 95% CI (upper), p.val
- Per-cell-type relative error of the estimated distance versus sim$D.true
- Optional DistPlot visualization of estimated distances and distGenes plot of perturbation-associated genes

## How to apply

Generate a fully simulated dataset with scDist::simData(nct=10, N1=5, N2=5) yielding the normalized count matrix Y (~1000 genes x 5100 cells), metadata (response, patient, clusters), and ground-truth distances D.true for 10 cell types. Compute top-K PCA loadings from the normalized counts via truncated SVD (irlba), project cells onto the loadings to get PC scores, and fit an independent lme4 linear mixed model per PC with 'response' as fixed effect and 'patient' as random effect via scDist(Y, meta.data, fixed.effects='response', random.effects='patient', clusters='clusters'). Apply the spike-slab empirical-Bayes shrinkage to the squared PC loadings (the MLE is upward-biased with few patients), compute D_K from posterior samples, and derive Monte Carlo empirical p-values from sums of Wald statistics with Satterthwaite df from lmerTest. Compare each cell type's estimated distance to sim$D.true and record relative error; in the demo, cell types a and g (true ~31.5 and ~31.3) are recovered as ~33.0 with p~1e-5, while unchanged cell type f (true ~2.76) is correctly shrunk to 0.0 with p=0.948. Confirm the results table contains distance, 95% CI low/upper bounds, and p-value columns.

## Related tools

- **scDist** (Method under test: estimates between-condition cell-type distances with per-PC mixed models, empirical-Bayes shrinkage, and Monte Carlo Wald tests; provides simData for ground-truth simulation) — github.com/phillipnicol/scDist
- **simData** (Generates the fully simulated dataset (nct=10, N1=5, N2=5) with normalized counts, metadata, and ground-truth distances D.true) — github.com/phillipnicol/scDist
- **lme4** (Fits the per-PC linear mixed model (response fixed effect, patient random effect) on PC scores)
- **lmerTest** (Provides Satterthwaite approximation of the degrees of freedom nu_k used in the Wald-test p-values)
- **irlba** (Truncated SVD used to compute the top-K PCA loadings onto which cells are projected to obtain PC scores)

## Examples

```
library(scDist); sim <- simData(nct=10,N1=5,N2=5); out <- scDist(sim$Y, sim$meta.data, fixed.effects="response", random.effects="patient", clusters="clusters"); out$results; sim$D.true
```

## Evaluation signals

- Estimated distances match sim$D.true within tolerance: in the demo, cell types a and g with true distances ~31.5 and ~31.3 are recovered at ~33.0 with significant p-values (~1e-5)
- Unchanged cell type f (true distance ~2.76) is correctly non-significant, shrunk to estimated distance 0.0 with p=0.948 (no false positive)
- Results table schema check: contains exactly the columns Dist., 95% CI (low), 95% CI (upper), and p.val for each of the 10 cell types
- Posterior shrinkage behaves as expected: estimated distances are not inflated relative to ground truth despite only 5 patients per group (upward MLE bias is corrected)
- DistPlot and distGenes(out, cluster='a') render without error, confirming downstream summary/visualization components run on the validated output

## Limitations

- The MLE of the distance is upward-biased with few patients, hence the need for post-hoc Bayesian shrinkage; residual estimation error remains (demo recovers ~33 vs true ~31.5)
- Results are sensitive to the choice of the number of principal components K, for which only a fixed heuristic (20 or 50) exists with no adaptive selection procedure
- Over-clustering the simulation causes downward bias and false negatives (median distance decays toward 0 as cluster number increases), so validation conclusions depend on reasonable clustering
- Performance may degrade if normalization differs from scTransform Pearson residuals, and scDist cannot yet be applied directly to raw counts
- Violation of the spherical-normal (shared variance) assumption could miscalibrate standard errors and hypothesis tests

## Evidence

- [readme] sim <- simData(nct=10,N1=5,N2=5): "Generate simulated data with 10 cell types and 5 patients in each group"
- [readme] out$results schema: "The results data frame gives a summary of the estimated distance and
uncertainty for each cell type"
- [discussion] upward MLE bias motivates shrinkage: [evidence span withheld — anchor status: not_found]
- [discussion] per-PC mixed models with lme4 and Satterthwaite df: "Estimate (Uβ)k by using lme427 to fit the model (6) using the PC scores corresponding to the k-th loading"
- [discussion] over-clustering causes false negatives: "As the number of clusters increases, the median distance decays towards 0, which demonstrates that scDist can produce false negatives when the data is over-clustered"
- [discussion] sensitivity to number of PCs: "Another point of sensitivity for scDist is the choice of the number of principal components used to estimate the distance."
