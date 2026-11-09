---
name: differential-state-analysis
description: Use when scRNA-seq data with cell-type annotations and condition labels
  (e.g., immunotherapy responder status per patient) must be screened for which annotated
  cell type shows the largest transcriptomic shift between conditions, accounting
  for patient/replicate structure.
when_to_use_negative:
- The goal is differential abundance (changing cell-type proportions between conditions)
  — use DA-seq, Milo, or Meld instead; scDist targets differential state of predefined
  cell types
- Input is raw (unnormalized) counts — scDist cannot currently be applied directly
  to counts and requires normalized data (a future version may adapt this)
- Data are heavily over-clustered relative to condition-specific effects, which downward-biases
  estimated distances and can produce false negatives
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scDist
  role: 'Core method: fits the mixed-effects model on PC scores per cell type, returns
    distance/CIs/empirical p-values, and provides DistPlot and distGenes for visualization'
  repo: github.com/phillipnicol/scDist
- name: lme4
  role: Fits the per-PC linear mixed model (model 6) on the PC scores with condition
    fixed effect and patient random effect
- name: lmerTest
  role: Provides Satterthwaite's approximation for the degrees-of-freedom parameter
    (nu_k) used in Wald statistics
- name: scTransform
  role: Recommended normalization producing Pearson residuals on which scDist is run
- name: Seurat
  role: Reference-mapping (Azimuth) annotation workflow used to assign cell populations
    prior to scDist testing
- name: irlba
  role: Efficient PCA used to obtain loadings and PC scores as the low-dimensional
    embedding for the plug-in distance estimator
- name: ggplot2
  role: Plotting backend underlying DistPlot and distGenes result visualizations
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:641ab6ab2e363f612ee82e102bdc5103e3666d32854e0877c118d71dbfa729aa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Seurat
  - ggplot2
  - irlba
  - lme4
  - lmerTest
  - scDist
  - scTransform
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# differential-state-analysis

## Summary

Detects predefined cell types whose transcriptomic profiles differ between conditions (differential state) using scDist, a mixed-effects model that estimates the distance between condition means in gene-expression space while absorbing individual-to-individual and cohort-to-cohort variability. Unlike differential abundance approaches (DA-seq, Milo, Meld), it targets changing expression states rather than changing proportions.

## When to use

Use when scRNA-seq data with cell-type annotations and condition labels (e.g., immunotherapy responder status per patient) must be screened for which annotated cell type shows the largest transcriptomic shift between conditions, accounting for patient/replicate structure.

## When NOT to use

- The goal is differential abundance (changing cell-type proportions between conditions) — use DA-seq, Milo, or Meld instead; scDist targets differential state of predefined cell types
- Input is raw (unnormalized) counts — scDist cannot currently be applied directly to counts and requires normalized data (a future version may adapt this)
- Data are heavily over-clustered relative to condition-specific effects, which downward-biases estimated distances and can produce false negatives

## Inputs

- Normalized scRNA-seq count matrix (genes x cells) with scTransform Pearson residuals recommended
- Per-cell metadata assigning each cell to a condition (fixed effect, e.g., response), patient/sample (random effect), and cluster/cell type
- Optional cohort/batch labels to include in a nested random effect (e.g., 1|gamma:omega)
- Choice of number of principal components K (fixed heuristic, 20 or 50)

## Outputs

- Results data frame with one row per tested cell type: estimated distance, 95% CI (low, upper), and empirical p-value
- Distance ranking of cell types (DistPlot visualization)
- Genes associated with the perturbation in a chosen cluster (distGenes plot, e.g., FCER1G+ NK-2 signature)

## How to apply

Normalize counts with scTransform Pearson residuals (scDist is compatible with other normalizations but Pearson residuals are recommended). Annotate cells against a reference (e.g., the Azimuth/Seurat workflow) and exclude any cell type not present in more than 10% of samples before testing. Use PCA loadings as a plug-in estimator for U and fit, with lme4/lmerTest, a linear mixed-effects model on PC scores for each cell type, with the condition (e.g., 'response') as fixed effect and patient (nested with cohort, e.g., 1|gamma:omega) as random effect. Compute Monte Carlo empirical p-values from sums of Wald statistics and rank cell types by estimated distance with 95% CIs; the results data frame reports Dist., 95% CI (low/upper), and p.val per cell type. Derive the responder-associated gene signature by re-fitting the same mixed model per gene within the top perturbed population. Beware double dipping: using the same expression data for annotation/clustering and distance estimation can over-cluster the data and bias distances downward toward false negatives.

## Related tools

- **scDist** (Core method: fits the mixed-effects model on PC scores per cell type, returns distance/CIs/empirical p-values, and provides DistPlot and distGenes for visualization) — github.com/phillipnicol/scDist
- **lme4** (Fits the per-PC linear mixed model (model 6) on the PC scores with condition fixed effect and patient random effect)
- **lmerTest** (Provides Satterthwaite's approximation for the degrees-of-freedom parameter (nu_k) used in Wald statistics)
- **scTransform** (Recommended normalization producing Pearson residuals on which scDist is run)
- **Seurat** (Reference-mapping (Azimuth) annotation workflow used to assign cell populations prior to scDist testing)
- **irlba** (Efficient PCA used to obtain loadings and PC scores as the low-dimensional embedding for the plug-in distance estimator)
- **ggplot2** (Plotting backend underlying DistPlot and distGenes result visualizations)

## Examples

```
devtools::install_github("phillipnicol/scDist"); out <- scDist(sim$Y, sim$meta.data, fixed.effects="response", random.effects="patient", clusters="clusters")
```

## Evaluation signals

- Output results data frame contains one row per tested cell type with exactly the columns Dist., 95% CI (low), 95% CI (upper), p.val
- Empirical p-values respect the Monte Carlo floor (simulated demo shows minimum p.val = 0.0000099999 for 100,000-draw empirical p-values)
- On the built-in simData (10 cell types, 5 patients/group), estimated distances should be close to sim$D.true (e.g., cluster a: 33.04 vs true 31.50; cluster f: 0.00 vs true 2.76) and top-ranked clusters match simulated perturbed types
- Applied to the immunotherapy cohorts, top-ranked perturbed populations must include the NK (FCER1G+) cell compartment, and validated in bulk transcriptomes from 789 patients
- Filter check: any cell type present in fewer than 10% of samples is excluded before testing

## Limitations

- Double dipping expression data for annotation and distance estimation causes over-clustering and downward bias in estimated distances; as the number of clusters increases, median distance decays toward 0, producing false negatives
- Results are sensitive to the number of principal components K, chosen by a fixed heuristic (20 or 50) with no adaptive selection procedure
- scDist cannot be applied directly to raw counts; sub-optimal normalization (other than Pearson residuals) may hurt performance
- Violation of the spherical-normal (shared variance) assumption can miscalibrate standard errors and tests (though not estimation); significant clustering/annotation errors cause unavoidable bias, and a cluster-free extension does not yet exist

## Evidence

- [intro] Differential state analysis definition: "In contrast, differential state analysis seeks to detect predefined cell types with distinct transcriptomic profiles between conditions."
- [abstract] Mixed-effects model rationale: "we propose a computational tool, scDist, based on a mixed-effects model that provides a statistically rigorous and computationally efficient approach"
- [discussion] Recommended normalization: "We recommend using scTransform13 to normalize, although the method is compatible with any normalization approach."
- [discussion] PCA plug-in estimator and lme4 fitting: "Estimate (Uβ)k by using lme427 to fit the model (6) using the PC scores corresponding to the k-th loading"
- [discussion] Sample prevalence filter: "Subsequently, we excluded any cell types not present in more than 10% of the samples from further analysis."
- [intro] Immunotherapy discovery finding: "jointly inferring information from five single-cell immunotherapy cohorts, revealing significant differences in a subpopulation of NK cells between immunotherapy responders and non-responders"
- [readme] README results schema: "The results data frame gives a summary of the estimated distance and
uncertainty for each cell type"
- [discussion] Over-clustering failure mode: "As the number of clusters increases, the median distance decays towards 0, which demonstrates that scDist can produce false negatives when the data is over-clustered"
