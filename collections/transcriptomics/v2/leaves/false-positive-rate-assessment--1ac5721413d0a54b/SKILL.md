---
name: false-positive-rate-assessment
description: Use when benchmarking a cell-type perturbation detection method under
  null conditions (beta=0) to test whether individual-to-individual variability inflates
  false positives, or to validate that a mixed-effects alternative controls FPR.
when_to_use_negative:
- Data contain a genuine expected condition effect (beta != 0) — the null design would
  mask real perturbations; use a powered simulation instead
- Only a single sample per condition is available — patient-level random effects cannot
  be estimated, so FPR from patient variability cannot be assessed
- The goal is differential abundance (cell-type proportion changes) rather than differential
  state within predefined cell types
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: scDist
  role: Mixed-effects comparator and simulation source (simData) for null data; provides
    distances, CIs, and empirical p-values under beta=0
  repo: github.com/phillipnicol/scDist
- name: Augur
  role: Tool under test; per-cell-type AUC scores are evaluated for false positives
    under null conditions
- name: lme4
  role: Fits per-PC linear mixed models within scDist for the null comparison
- name: lmerTest
  role: Provides Satterthwaite approximation of degrees of freedom for Wald statistic
    p-values
- name: Seurat
  role: Provides the COVID-19 blood object that is partitioned into 3-vs-3 healthy
    groups for negative-control trials
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:c8bc795d5d025a1a2ec06104bd4c4c89b4ee968cb526737edd62ae760711cf96
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Augur
  - Seurat
  - ggplot2
  - irlba
  - lme4
  - lmerTest
  - scDist
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# false-positive-rate-assessment

## Summary

A negative-control benchmarking protocol that quantifies how often a differential-state tool (e.g., Augur AUC-based scoring) calls perturbed cell types when no true perturbation exists, by simulating null scRNA-seq data with varied patient-level variability (tau^2 = 0-1) and running scDist as a mixed-model comparator that should control the false-positive rate.

## When to use

Use when benchmarking a cell-type perturbation detection method under null conditions (beta=0) to test whether individual-to-individual variability inflates false positives, or to validate that a mixed-effects alternative controls FPR.

## When NOT to use

- Data contain a genuine expected condition effect (beta != 0) — the null design would mask real perturbations; use a powered simulation instead
- Only a single sample per condition is available — patient-level random effects cannot be estimated, so FPR from patient variability cannot be assessed
- The goal is differential abundance (cell-type proportion changes) rather than differential state within predefined cell types

## Inputs

- Null-simulated scRNA-seq count matrix and metadata from scDist simData (10 cell types, 5 patients per group, beta=0, sigma^2=1, tau^2 in 0-1)
- Real Seurat object (COVID-19 blood) partitioned into 3-vs-3 healthy sample groups across 20 random splits
- Per-cell metadata columns: response (condition), patient (replicate), clusters (cell type labels)

## Outputs

- Per-partition Augur AUC scores per cell type and fraction of AUCs > 0.5
- scDist results data frame with distance estimates, 95% CIs, and p-values per cell type
- FPR-vs-tau^2 curves: Augur false-positive trend vs. scDist calibrated FPR at beta=0

## How to apply

Generate null-simulated scRNA-seq datasets with scDist's simData utility (beta=0, sigma^2=1, tau^2 varied over 0-1) and additionally partition a real COVID-19 blood Seurat object 20 times into 3-vs-3 healthy sample groups so no condition effect exists. Run the tool under test (Augur) on each partition to obtain per-cell-type AUC scores, recording the fraction of AUCs > 0.5 and whether low-information cell types like RBCs are spuriously flagged. Run scDist on the same inputs with fixed.effects='response', random.effects='patient', clusters='clusters', fitting per-PC linear mixed models via lme4 with Satterthwaite degrees of freedom from lmerTest and empirical p-values from Monte Carlo sums of Wald statistics. Tabulate, per tau^2 value and per partition, Augur AUCs and scDist distance estimates, 95% CIs and p-values, then quantify the false-positive rate at beta=0 and the trend of the distance estimate across tau^2. The rationale is that naive AUC-based approaches interpret individual-to-individual variability as between-condition differences, whereas the mixed-effects model explicitly models patient as a random effect.

## Related tools

- **scDist** (Mixed-effects comparator and simulation source (simData) for null data; provides distances, CIs, and empirical p-values under beta=0) — github.com/phillipnicol/scDist
- **Augur** (Tool under test; per-cell-type AUC scores are evaluated for false positives under null conditions)
- **lme4** (Fits per-PC linear mixed models within scDist for the null comparison)
- **lmerTest** (Provides Satterthwaite approximation of degrees of freedom for Wald statistic p-values)
- **Seurat** (Provides the COVID-19 blood object that is partitioned into 3-vs-3 healthy groups for negative-control trials)

## Examples

```
devtools::install_github("phillipnicol/scDist"); sim <- simData(nct=10,N1=5,N2=5); out <- scDist(sim$Y,sim$meta.data,fixed.effects="response",random.effects="patient",clusters="clusters")
```

## Evaluation signals

- Under null conditions, scDist's false-positive rate should remain controlled (non-significant p-values at beta=0) and its distance estimate should not grow with tau^2
- Augur's estimated condition differences should grow with patient-level variability, converging to maximum AUC of 1 — this confirms the expected false-positive pattern (93% of AUCs > 0.5, RBCs flagged in 20/20 trials)
- Results should reproduce the patterns of Figs. 1A-C and 3A-C (FPR trend across tau^2 values 0-1)
- scDist results data frame must contain Dist., 95% CI (low), 95% CI (upper), and p.val columns for every cell type
- Empirical p-values from Monte Carlo Wald statistics should be stable across repeated partitions (minimum ~0.00001 granularity for 10,000 draws)

## Limitations

- Double dipping of expression data for annotation and distance estimation can cause over-clustering and downward bias in estimated distances, leading scDist to produce false negatives as cluster number increases
- Results are sensitive to the number of principal components used (fixed heuristic of 20 or 50; no adaptive K selection)
- scDist requires normalized input (Pearson residuals/scTransform recommended) and cannot be applied directly to raw counts; violation of the spherical-normal assumption could miscalibrate standard errors and tests

## Evidence

- [intro] Augur false positives under null variability: "naive approaches based on current computational tools lead to false positive findings"
- [intro] Mixed-model design rationale: "This method, which we call scDist, introduces an interpretable metric for comparing different cell types while accounting for individual-to-individual and technical variability"
- [discussion] Mixed-model fitting: "Estimate (Uβ)k by using lme427 to fit the model (6) using the PC scores corresponding to the k-th loading"
- [discussion] Degrees of freedom: "νk is estimated using Satterthwaite’s approximation in lmerTest."
- [readme] Installation and scDist call: "devtools::install_github("phillipnicol/scDist")"
- [readme] Demo invocation: "out <- scDist(sim$Y,sim$meta.data,fixed.effects = "response",
              random.effects="patient",
              clusters="clusters")"
- [discussion] Over-clustering false negatives: "As the number of clusters increases, the median distance decays towards 0, which demonstrates that scDist can produce false negatives when the data is over-clustered"
- [discussion] PC sensitivity: "Another point of sensitivity for scDist is the choice of the number of principal components used to estimate the distance."
