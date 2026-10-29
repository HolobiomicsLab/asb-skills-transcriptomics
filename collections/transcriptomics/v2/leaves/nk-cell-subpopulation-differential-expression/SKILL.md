---
name: nk-cell-subpopulation-differential-expression
description: Use when jointly analyzing multiple single-cell immunotherapy cohorts
  of CD45+ cells to identify which cell subpopulation (e.g., NK cells) differs transcriptomically
  between responders and non-responders.
when_to_use_negative:
- Differential abundance is the goal — scDist performs differential state (transcriptomic
  distance), not proportion changes (use DA-seq, Milo, or Meld instead)
- 'Data is over-clustered: condition-specific effects can cause over-clustering and
  downward-biased distances, producing false negatives'
- Only raw counts are available without prior normalization — scDist cannot currently
  be applied directly to raw counts
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scDist
  role: 'Core method: mixed-effects distance estimation across cohorts and per-gene
    refitting within the NK-2 cluster'
  repo: github.com/phillipnicol/scDist
- name: lme4
  role: Fits the per-PC and per-gene linear mixed-effects models with responder status
    as fixed effect
- name: lmerTest
  role: Estimates degrees of freedom via Satterthwaite's approximation for the Wald
    tests
- name: scTransform
  role: Recommended normalization of counts to Pearson residuals before scDist
- name: Seurat
  role: Azimuth reference-mapping workflow for cell-population annotation
- name: irlba
  role: Computes PCA loadings/PC scores used as the low-dimensional embedding for
    distance estimation
- name: ggplot2
  role: Visualization of ranked distances and gene signatures (DistPlot/distGenes
    outputs)
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:18e2e977b7d165f338ee2c77df9290a992a2523237fe61cda356b4b6f8579088
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
yes_rate: 0.8125
n_claims: 16
---

# nk-cell-subpopulation-differential-expression

## Summary

Applies scDist's mixed-effects framework jointly across single-cell immunotherapy cohorts to detect responder-associated transcriptomic perturbation in FCER1G+ NK cells, then refits the same per-gene mixed model within the NK-2 subpopulation to derive its differential-expression signature.

## When to use

Use when jointly analyzing multiple single-cell immunotherapy cohorts of CD45+ cells to identify which cell subpopulation (e.g., NK cells) differs transcriptomically between responders and non-responders.

## When NOT to use

- Differential abundance is the goal — scDist performs differential state (transcriptomic distance), not proportion changes (use DA-seq, Milo, or Meld instead)
- Data is over-clustered: condition-specific effects can cause over-clustering and downward-biased distances, producing false negatives
- Only raw counts are available without prior normalization — scDist cannot currently be applied directly to raw counts

## Inputs

- Normalized scRNA-seq count matrices (scTransform Pearson residuals) from four immunotherapy CD45+ cohorts
- Per-cell metadata: cluster/cell-type annotation (Azimuth), patient id, cohort id, responder status
- PCA/PC-score representations of the normalized expression matrix

## Outputs

- Per-cell-type results table with estimated distance, 95% CI (low/upper), and Monte Carlo p-value
- Ranked perturbation list (Fig. 6-style discovery ranking) with FCER1G+ NK cells as top-ranked responder-associated population
- Per-gene differential-expression signature within the NK-2 subpopulation

## How to apply

Obtain the four public immunotherapy scRNA cohorts (expression counts plus patient responder status) and normalize with scTransform Pearson residuals, annotating cells via the Azimuth/Seurat reference-mapping workflow. Exclude any cell type not present in more than 10% of samples, then fit scDist's mixed-effects model on PCA/PC-score representations using lme4/lmerTest, with responder status as fixed effect and the nested random effect (1|gamma:omega) to absorb patient-to-patient and cohort-to-cohort variability. Compute Monte Carlo empirical p-values from sums of Wald statistics and rank cell types by estimated distance with 95% CIs, reproducing the Fig. 6 discovery ranking. Once the NK (FCER1G+) compartment is top-ranked, refit the same mixed model per gene within the NK-2 population to extract its responder-associated differential-expression signature; validate the signature in bulk transcriptomes from 789 patients.

## Related tools

- **scDist** (Core method: mixed-effects distance estimation across cohorts and per-gene refitting within the NK-2 cluster) — github.com/phillipnicol/scDist
- **lme4** (Fits the per-PC and per-gene linear mixed-effects models with responder status as fixed effect)
- **lmerTest** (Estimates degrees of freedom via Satterthwaite's approximation for the Wald tests)
- **scTransform** (Recommended normalization of counts to Pearson residuals before scDist)
- **Seurat** (Azimuth reference-mapping workflow for cell-population annotation)
- **irlba** (Computes PCA loadings/PC scores used as the low-dimensional embedding for distance estimation)
- **ggplot2** (Visualization of ranked distances and gene signatures (DistPlot/distGenes outputs))

## Examples

```
library(scDist); out <- scDist(sim$Y, sim$meta.data, fixed.effects = "response", random.effects = "patient", clusters = "clusters")
```

## Evaluation signals

- Output results data frame contains one row per tested cell type with distance, 95% CI (low/upper), and p-value columns
- Top-ranked perturbed populations include the NK (FCER1G+) cell compartment reported in the paper
- Estimated distances come with calibrated 95% CIs and Monte Carlo empirical p-values from sums of Wald statistics
- Joint-model results reproduce the Fig. 6 ranking, and the derived NK-2 signature validates in bulk transcriptomes from 789 patients
- Cell types absent from more than 10% of samples are excluded before model fitting

## Limitations

- Results are sensitive to the choice of the number of principal components used to estimate the distance; current choice is a fixed heuristic (20 or 50 PCs), with no adaptive procedure for selecting K
- Double dipping of expression data for annotation and distance estimation can cause over-clustering and downward bias in estimated distances
- scDist cannot currently be applied directly to raw counts, and performance could be affected by sub-optimal normalization approaches other than Pearson residuals
- Significant clustering/annotation errors cause unavoidable bias; a cluster-free extension does not yet exist, and violation of the spherical-normal assumption could miscalibrate standard errors

## Evidence

- [intro] jointly infer information from five single-cell immunotherapy cohorts to reveal differences in NK cell subpopulations between responders and non-responders: "jointly inferring information from five single-cell immunotherapy cohorts, revealing significant differences in a subpopulation of NK cells between immunotherapy responders and non-responders"
- [discussion] exclude cell types in less than 10% of samples: "Subsequently, we excluded any cell types not present in more than 10% of the samples from further analysis."
- [discussion] scTransform normalization recommendation: "We recommend using scTransform13 to normalize, although the method is compatible with any normalization approach."
- [discussion] lme4 fitting of PC-score mixed models: "Estimate (Uβ)k by using lme427 to fit the model (6) using the PC scores corresponding to the k-th loading"
- [methods] scDist call with fixed and random effects: "out <- scDist(sim$Y,sim$meta.data,fixed.effects = "response",
              random.effects="patient",
              clusters="clusters")"
- [readme] results data frame schema: "The results data frame gives a summary of the estimated distance and
uncertainty for each cell type"
- [discussion] over-clustering false negatives: "As the number of clusters increases, the median distance decays towards 0, which demonstrates that scDist can produce false negatives when the data is over-clustered"
- [discussion] sensitivity to number of PCs: "Another point of sensitivity for scDist is the choice of the number of principal components used to estimate the distance."
