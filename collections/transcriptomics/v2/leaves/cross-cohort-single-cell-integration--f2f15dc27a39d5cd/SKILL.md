---
name: cross-cohort-single-cell-integration
description: Use when several scRNA-seq cohorts with responder-status annotations
  must be jointly scanned for perturbed cell populations, and naive per-cell-type
  testing would be confounded by individual-to-individual and cohort-to-cohort variability.
when_to_use_negative:
- The goal is differential abundance (changing cell proportions), not transcriptomic
  state — use DA-seq, Milo, or Meld instead
- Only raw counts are available without normalization — scDist cannot currently be
  applied directly to raw counts
- The data are heavily over-clustered — over-clustering drives estimated distances
  toward 0 and produces false negatives
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scDist
  role: Core mixed-effects distance estimator fit per cell type across cohorts
  repo: github.com/phillipnicol/scDist
- name: lme4
  role: Fits the per-PC linear mixed model on PC scores
- name: lmerTest
  role: Estimates nu (degrees of freedom) via Satterthwaite's approximation for Wald
    statistics
- name: scTransform
  role: Recommended normalization to Pearson residuals before scDist
- name: Seurat
  role: Azimuth reference-mapping workflow used to annotate cell populations
- name: irlba
  role: PCA / PC-score computation feeding the low-dimensional embedding used by scDist
- name: ggplot2
  role: Visualization backend for DistPlot and distGenes outputs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:5aeb5284549ccc7636f41fe0447fc6853737a984d022cb59eb01fe70866a51cc
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# cross-cohort-single-cell-integration

## Summary

Jointly test multiple single-cell immunotherapy scRNA cohorts for cell-type-specific transcriptomic differences between responders and non-responders using scDist's mixed-effects model, which separates responder signal from patient-to-patient and cohort-to-cohort variability. This surfaced the FCER1G+ NK-cell compartment that the authors validated in bulk transcriptomes from 789 patients.

## When to use

Use when several scRNA-seq cohorts with responder-status annotations must be jointly scanned for perturbed cell populations, and naive per-cell-type testing would be confounded by individual-to-individual and cohort-to-cohort variability.

## When NOT to use

- The goal is differential abundance (changing cell proportions), not transcriptomic state — use DA-seq, Milo, or Meld instead
- Only raw counts are available without normalization — scDist cannot currently be applied directly to raw counts
- The data are heavily over-clustered — over-clustering drives estimated distances toward 0 and produces false negatives

## Inputs

- Normalized single-cell expression matrices from multiple cohorts (CD45+ cells, scTransform Pearson residuals)
- Cell-level metadata assigning each cell to cluster, condition (responder status), patient, and cohort
- Cell-type annotations from Azimuth/Seurat reference mapping

## Outputs

- Results data frame with one row per tested cell type: estimated distance, 95% CI (low/upper), and Monte Carlo empirical p-value
- Ranked discovery list of perturbed populations (e.g., FCER1G+ NK cells) with DistPlot visualization
- Responder-associated differential-expression signature for the top perturbed cluster via distGenes

## How to apply

Obtain the public CD45+ immunotherapy scRNA cohorts (expression counts plus patient responder status) and normalize with scTransform Pearson residuals. Annotate cell populations via the Seurat-based Azimuth reference mapping workflow, then exclude any cell type not present in more than 10% of samples before scDist input preparation. Project cells to PCA (irlba) and use the PCA loadings as a plug-in estimator for U, fitting a per-PC linear mixed model with lme4 (Satterthwaite nu via lmerTest) with responder status as the fixed effect and the nested random effect (1|gamma:omega) to absorb patient and cohort variability. Compute Monte Carlo empirical p-values from sums of Wald statistics and rank cell types by estimated distance with 95% confidence intervals. Re-fit the mixed model per gene within the top perturbed population (e.g., the FCER1G+ NK-2 cluster) using distGenes to derive the responder-associated differential-expression signature.

## Related tools

- **scDist** (Core mixed-effects distance estimator fit per cell type across cohorts) — github.com/phillipnicol/scDist
- **lme4** (Fits the per-PC linear mixed model on PC scores)
- **lmerTest** (Estimates nu (degrees of freedom) via Satterthwaite's approximation for Wald statistics)
- **scTransform** (Recommended normalization to Pearson residuals before scDist)
- **Seurat** (Azimuth reference-mapping workflow used to annotate cell populations)
- **irlba** (PCA / PC-score computation feeding the low-dimensional embedding used by scDist)
- **ggplot2** (Visualization backend for DistPlot and distGenes outputs)

## Examples

```
out <- scDist(sim$Y,sim$meta.data,fixed.effects = "response", random.effects="patient", clusters="clusters")
```

## Evaluation signals

- The results data frame contains exactly the four columns per tested cell type: 'Dist.', '95% CI (low)', '95% CI (upper)', 'p.val' (p-values bounded below by the Monte Carlo floor ~0.0000099999)
- Top-ranked perturbed populations include the FCER1G+ NK cell compartment, matching the Fig. 6 discovery ranking
- Cell types present in ≤10% of samples are absent from the tested set (filter applied before model fitting)
- Distances are non-negative with 95% CIs that bracket the point estimate; estimated distances should approximate ground truth in the simData demo (e.g., cluster a: est. 33.04 vs true 31.50)
- Responder-associated signature derived by per-gene refits within the NK-2 population is verifiable against bulk transcriptomes from 789 patients

## Limitations

- Results are sensitive to the choice of the number of principal components used to estimate the distance; selection is a fixed heuristic (20 or 50), with no adaptive procedure
- Double dipping expression data for annotation and distance estimation can cause over-clustering and downward bias in estimated distances, producing false negatives
- Violation of the spherical-normal (shared variance) assumption could lead to miscalibrated standard errors and hypothesis tests
- scDist cannot currently be applied directly to raw counts, and significant clustering/annotation errors cause unavoidable bias (cluster-free extension is future work)

## Evidence

- [abstract] scDist mixed-effects model: "we propose a computational tool, scDist, based on a mixed-effects model that provides a statistically rigorous and computationally efficient approach"
- [intro] immunotherapy discovery: "jointly inferring information from five single-cell immunotherapy cohorts, revealing significant differences in a subpopulation of NK cells between immunotherapy responders and non-responders"
- [discussion] 10% presence filter: "Subsequently, we excluded any cell types not present in more than 10% of the samples from further analysis."
- [discussion] PCA plug-in estimator: "Estimate (Uβ)k by using lme427 to fit the model (6) using the PC scores corresponding to the k-th loading"
- [readme] mixed-model rationale: "When there are multiple replicates (i.e., patients), `scDist` uses linear-mixed effects models to correct for sample-to-sample variability."
