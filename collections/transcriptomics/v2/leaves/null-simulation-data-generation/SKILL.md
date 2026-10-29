---
name: null-simulation-data-generation
description: Use when benchmarking differential-state or perturbation-detection tools
  under null conditions where no cell type should be detected as changed, to measure
  false-positive rates as patient-level variability grows.
when_to_use_negative:
- When the goal is detecting real perturbations in data with a genuine between-condition
  effect (beta != 0), not benchmarking false-positive control
- When comparing methods for differential abundance (e.g., DA-seq, Milo, Meld) on
  cell-type proportions rather than differential transcriptomic state
- When data cannot be normalized to Pearson residuals via scTransform — scDist is
  not designed for raw counts
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scDist
  role: Provides the simData utility used to generate null-simulated datasets (beta=0,
    tau^2 varied) and serves as the benchmarked method whose mixed-model distance
    estimates and p-values are evaluated
  repo: github.com/phillipnicol/scDist
- name: Augur
  role: Benchmarked comparator producing per-cell-type AUC scores evaluated for false
    positives under null conditions
- name: Seurat
  role: Holds and partitions the COVID-19 blood scRNA-seq object into 3-vs-3 healthy
    sample groups
- name: lme4
  role: Fits the per-PC linear mixed models underlying scDist distance estimates on
    simulated inputs
- name: lmerTest
  role: Supplies Satterthwaite degrees-of-freedom approximations for the mixed-model
    inference
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:8d4806d775767982689feadd384fe05e8d33b57a96e82b0679cf0998fa08a83a
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
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# null-simulation-data-generation

## Summary

Generate null-condition simulated single-cell RNA-seq datasets (no true between-condition effect, beta=0) using the scDist simData utility with varied patient-level variability (tau^2), plus random 3-vs-3 partitions of a COVID-19 blood Seurat object, to benchmark whether differential-state tools like Augur and scDist falsely flag perturbed cell types.

## When to use

Use when benchmarking differential-state or perturbation-detection tools under null conditions where no cell type should be detected as changed, to measure false-positive rates as patient-level variability grows.

## When NOT to use

- When the goal is detecting real perturbations in data with a genuine between-condition effect (beta != 0), not benchmarking false-positive control
- When comparing methods for differential abundance (e.g., DA-seq, Milo, Meld) on cell-type proportions rather than differential transcriptomic state
- When data cannot be normalized to Pearson residuals via scTransform — scDist is not designed for raw counts

## Inputs

- Normalized scRNA-seq count matrix plus metadata assigning each cell to condition, patient, and cluster
- COVID-19 blood Seurat object for random healthy 3-vs-3 sample partitioning
- Simulation parameters: beta=0 (null), sigma^2=1, tau^2 grid from 0 to 1

## Outputs

- Per-cell-type Augur AUC scores for each of 20 partitions, including fraction of AUCs > 0.5 and whether RBCs are flagged as perturbed
- Per-cell-type scDist distance estimates with 95% CIs and p-values for each tau^2 value and partition
- False-positive rate at beta=0 and trend of the distance estimate across tau^2 (comparison with Figs. 1A-C and 3A-C)

## How to apply

Install scDist from GitHub with devtools::install_github("phillipnicol/scDist") and load Augur, Seurat, lme4 and lmerTest in R. Generate null-simulated scRNA-seq datasets with simData setting beta=0, sigma^2=1, and tau^2 varied over 0-1 (default simulation uses 10 cell types and 5 patients per group); additionally partition the COVID-19 blood Seurat object 20 times into 3-vs-3 healthy sample groups. Run each candidate method (Augur per-cell-type AUCs; scDist with fixed.effects="response", random.effects="patient", clusters="clusters") on every simulated partition. Tabulate, per tau^2 value and per partition, Augur AUCs (fraction of AUCs > 0.5, RBC flagging) and scDist distance estimates with 95% CIs and p-values. Rationale: at beta=0 there is no true perturbation, so any increase in estimated condition difference with tau^2 signals a false positive driven by individual-to-individual variability rather than biology.

## Related tools

- **scDist** (Provides the simData utility used to generate null-simulated datasets (beta=0, tau^2 varied) and serves as the benchmarked method whose mixed-model distance estimates and p-values are evaluated) — github.com/phillipnicol/scDist
- **Augur** (Benchmarked comparator producing per-cell-type AUC scores evaluated for false positives under null conditions)
- **Seurat** (Holds and partitions the COVID-19 blood scRNA-seq object into 3-vs-3 healthy sample groups)
- **lme4** (Fits the per-PC linear mixed models underlying scDist distance estimates on simulated inputs)
- **lmerTest** (Supplies Satterthwaite degrees-of-freedom approximations for the mixed-model inference)

## Examples

```
out <- scDist(sim$Y,sim$meta.data,fixed.effects="response",random.effects="patient",clusters="clusters")
```

## Evaluation signals

- In null simulations (beta=0), scDist controls the false-positive rate and its distance estimate does not grow with tau^2
- Expected Augur behavior reproduced: ~93% of Augur AUCs exceed 0.5 and RBCs are flagged perturbed in 20/20 trials, converging to AUC of 1 as tau^2 grows
- Each simData run yields a 1000-gene x (nct x patients) normalized count matrix with per-cell metadata columns response, patient, and clusters
- scDist results data frame reports Dist., 95% CI (low/upper), and p.val per cell type; p-values are empirical from Monte Carlo sums of Wald statistics
- Distance estimates and CIs are reproducible across the tau^2 grid and match the qualitative patterns of Figs. 1A-C and 3A-C

## Limitations

- Double dipping of expression data for annotation and distance estimation can cause over-clustering and downward bias in estimated distances
- Results are sensitive to the choice of the number of principal components used to estimate the distance; no adaptive procedure exists for selecting K (fixed heuristic of 20 or 50)
- Violation of the spherical-normal (shared variance) assumption could lead to miscalibrated standard errors and hypothesis tests; scDist cannot currently be applied directly to raw counts

## Evidence

- [abstract] naive approaches lead to false positives under patient variability: "naive approaches based on current computational tools lead to false positive findings"
- [methods] simData generates the simulated benchmark data: "Generate simulated data with 10 cell types and 5 patients in each group"
- [abstract] scDist is the mixed-effects tool under evaluation: "we propose a computational tool, scDist, based on a mixed-effects model that provides a statistically rigorous and computationally"
- [readme] simData usage and metadata structure are defined in the README demo: "sim <- simData(nct=10,N1=5,N2=5)"
- [readme] scDist models patient-to-patient variability via random effects: "scDist uses linear-mixed effects models to correct for sample-to-sample variability."
