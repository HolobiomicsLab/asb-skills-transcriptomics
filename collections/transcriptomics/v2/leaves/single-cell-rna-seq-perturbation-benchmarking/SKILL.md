---
name: single-cell-rna-seq-perturbation-benchmarking
description: Use when evaluating whether a differential-state method (e.g., Augur)
  inflates false positives as patient-level variability (tau^2) grows, by simulating
  null scRNA-seq data with beta=0 and comparing against scDist's calibrated p-values.
when_to_use_negative:
- When conditions truly differ (beta != 0), the null calibration check is not informative
  and power rather than false-positive control should be evaluated
- 'When data are raw counts: scDist requires normalized input (scTransform Pearson
  residuals recommended) and cannot yet be applied directly to counts'
- 'When performing differential abundance (proportion changes) rather than differential
  state: DA-seq, Milo, or Meld are the appropriate tools, not this benchmark'
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scDist
  role: 'Mixed-effects benchmark method: estimates between-condition distances and
    empirical p-values under null simulations'
  repo: https://github.com/phillipnicol/scDist
- name: Augur
  role: Machine-learning comparator whose AUC-based cell-type scores are benchmarked
    for false positives under tau^2 variation
- name: lme4
  role: Fits the per-PC linear mixed models underlying scDist's distance estimation
- name: lmerTest
  role: Provides Satterthwaite approximation for the degrees of freedom nu_k used
    in Wald-statistic p-values
- name: Seurat
  role: Provides and partitions the COVID-19 blood scRNA-seq object used for repeated
    3-vs-3 null splits
- name: ggplot2
  role: Visualization of AUC fractions and distance-vs-tau^2 trends
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:a344920bf35c43c18d424a1311cade788d888b39715751324e1ca77c7ececdd8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/phillipnicol/scDist
  tools:
  - Augur
  - Seurat
  - ggplot2
  - irlba
  - lme4
  - lmerTest
  - scDist
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# single-cell-rna-seq-perturbation-benchmarking

## Summary

Benchmark workflow for comparing perturbed-cell-type detection methods (Augur AUC-based vs. scDist mixed-effects distance) under null-simulated scRNA-seq conditions, quantifying false positives driven by individual-to-individual variability.

## When to use

Use when evaluating whether a differential-state method (e.g., Augur) inflates false positives as patient-level variability (tau^2) grows, by simulating null scRNA-seq data with beta=0 and comparing against scDist's calibrated p-values.

## When NOT to use

- When conditions truly differ (beta != 0), the null calibration check is not informative and power rather than false-positive control should be evaluated
- When data are raw counts: scDist requires normalized input (scTransform Pearson residuals recommended) and cannot yet be applied directly to counts
- When performing differential abundance (proportion changes) rather than differential state: DA-seq, Milo, or Meld are the appropriate tools, not this benchmark

## Inputs

- Null-simulated scRNA-seq normalized count matrix and metadata from scDist::simData (10 cell types, 5 patients per group, beta=0, tau^2 varied 0-1)
- COVID-19 peripheral blood Seurat object partitioned 20 times into 3-vs-3 healthy sample groups
- Per-cell metadata with condition, patient, and cluster assignments

## Outputs

- Per-partition Augur AUC scores for each cell type (fraction of AUCs > 0.5; e.g., 93% exceeding 0.5)
- scDist results data frame with estimated distance, 95% CI (low/upper), and p-value per cell type
- False-positive rate tables and distance-vs-tau^2 trend plots reproducing Figs. 1A-C and 3A-C

## How to apply

Install scDist from GitHub with devtools::install_github("phillipnicol/scDist") and load Augur, Seurat, lme4 and lmerTest in R. Generate null-simulated datasets with the scDist simData utility (beta=0, sigma^2=1, tau^2 varied over 0-1) and repeatedly partition a COVID-19 blood Seurat object into balanced 3-vs-3 healthy sample groups (20 partitions). Run Augur on each partition to obtain per-cell-type AUCs and record the fraction exceeding 0.5; run scDist on the same inputs with fixed.effects="response", random.effects="patient", clusters="clusters", fitting per-PC linear mixed models via lme4 and computing empirical p-values from Monte Carlo sums of Wald statistics with Satterthwaite degrees of freedom from lmerTest. Tabulate Augur AUCs and scDist distances, 95% CIs and p-values per tau^2 value and per partition, then quantify the scDist false-positive rate at beta=0 and the trend of the distance estimate across tau^2. The rationale is that Augur falsely interprets individual-to-individual variability as between-condition differences, while scDist's mixed-effects model should control false positives under the null.

## Related tools

- **scDist** (Mixed-effects benchmark method: estimates between-condition distances and empirical p-values under null simulations) — https://github.com/phillipnicol/scDist
- **Augur** (Machine-learning comparator whose AUC-based cell-type scores are benchmarked for false positives under tau^2 variation)
- **lme4** (Fits the per-PC linear mixed models underlying scDist's distance estimation)
- **lmerTest** (Provides Satterthwaite approximation for the degrees of freedom nu_k used in Wald-statistic p-values)
- **Seurat** (Provides and partitions the COVID-19 blood scRNA-seq object used for repeated 3-vs-3 null splits)
- **ggplot2** (Visualization of AUC fractions and distance-vs-tau^2 trends)

## Examples

```
devtools::install_github("phillipnicol/scDist"); out <- scDist(sim$Y, sim$meta.data, fixed.effects="response", random.effects="patient", clusters="clusters")
```

## Evaluation signals

- Under beta=0, scDist's empirical p-values control the false-positive rate (no cell type flagged significant beyond nominal alpha)
- Augur failure mode reproduced: ~93% of Augur AUCs exceed 0.5 and the same cell type (RBCs) is flagged perturbed in 20/20 trials
- scDist's estimated distance does not grow with tau^2, while Augur AUCs converge toward the maximum of 1 as patient-level variability increases
- scDist results data frame contains distance, 95% CI (low/upper), and p-value for every cell type, matching the README schema
- Simulation demo recovers true distances in sim$D.true within reported confidence intervals (e.g., cell type a: 33.04 estimated vs 31.50 true)

## Limitations

- Results are sensitive to the choice of the number of principal components used to estimate the distance; current choice is a fixed heuristic (20 or 50) with no adaptive K selection
- Double dipping of expression data for annotation and distance estimation can cause over-clustering and downward bias in estimated distances, producing false negatives as cluster number increases
- Violation of the spherical-normal (shared variance) assumption could lead to miscalibrated standard errors and hypothesis tests; raw counts cannot be used directly
- Software/data availability for the four immunotherapy scRNA cohorts and seven bulk RNA-seq cohorts is not fully provided in the document

## Evidence

- [abstract] Augur false-positive behavior under null: "detecting changing cell types is challenging due to individual-to-individual and cohort-to-cohort variability and naive approaches based on current computational tools lead to false positive findings"
- [intro] Augur limitation motivating benchmark: "To overcome these limitations, Augur9 uses a machine learning approach to quantify the cell-type specific separation between the two conditions."
- [intro] scDist mixed-model basis: "This method, which we call scDist, introduces an interpretable metric for comparing different cell types while accounting for individual-to-individual and technical variability"
- [methods] scDist installation command: "From the R console, `devtools::install_github("phillipnicol/scDist")`"
- [methods] scDist mixed-model call: "out <- scDist(sim$Y,sim$meta.data,fixed.effects = "response",
              random.effects="patient",
              clusters="clusters")"
- [discussion] Per-PC mixed-model fitting: "Estimate (Uβ)k by using lme427 to fit the model (6) using the PC scores corresponding to the k-th loading"
- [discussion] Satterthwaite degrees of freedom: "νk is estimated using Satterthwaite’s approximation in lmerTest."
- [readme] README demo setup and outputs schema: "Generate simulated data with 10 cell types and 5 patients in each group"
- [readme] README results schema: "The results data frame gives a summary of the estimated distance and
uncertainty for each cell type"
