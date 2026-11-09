---
name: single-cell-rnaseq-simulation
description: Use when you need controlled synthetic scRNA-seq data with known cell
  types, patients, and true condition distances to benchmark differential-state methods
  or test scalability.
when_to_use_negative:
- When the analysis goal is real biological discovery — simulated data with planted
  effects cannot substitute for real COVID-19 or immunotherapy cohorts
- When you need raw (unnormalized) count data — simData returns normalized counts
  and scDist cannot currently be applied directly to raw counts
- When the simulation's spherical-normal and cluster-structure assumptions would bias
  the comparison (e.g., testing cluster-free methods, since over-clustering causes
  downward bias in estimated distances)
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDist
  role: Provides the simData generator used to create simulated scRNA-seq datasets
    with known cell types and true distances, plus the analysis method being benchmarked
  repo: https://github.com/phillipnicol/scDist
- name: lme4
  role: Fits the mixed-effects model on PC scores of simulated data when validating
    recovery of true distances
- name: lmerTest
  role: Estimates degrees of freedom via Satterthwaite's approximation for p-values
    on simulated data
- name: Augur
  role: Comparison method run on simulated/subsampled data for runtime benchmarking
    (~5 min vs scDist's ~50 s on the COVID-19 data)
- name: nDEG
  role: Comparison method timed on the 1M-cell simulated dataset to record the >60x
    speedup of scDist
- name: irlba
  role: Fast PCA used to obtain the low-dimensional embedding and PC loadings on the
    simulated count matrix
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-51649-3
    title: Robust identification of perturbed cell types in single-cell RNA-seq data
  source_content_hash: sha256:eee2999ff03dc1d774023928b030f53010441dd92bea0df454adb8b7741d9cd8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/phillipnicol/scDist
  tools:
  - Augur
  - irlba
  - lme4
  - lmerTest
  - nDEG
  - scDist
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# single-cell-rnaseq-simulation

## Summary

Generate synthetic single-cell RNA-seq datasets with scDist's simData function (e.g., 10 cell types, 5 patients per group) to benchmark method runtime and accuracy, as done in Nicol et al. 2024 to demonstrate scDist recovers true between-condition distances and scales to 1M cells in ~10 minutes.

## When to use

Use when you need controlled synthetic scRNA-seq data with known cell types, patients, and true condition distances to benchmark differential-state methods or test scalability.

## When NOT to use

- When the analysis goal is real biological discovery — simulated data with planted effects cannot substitute for real COVID-19 or immunotherapy cohorts
- When you need raw (unnormalized) count data — simData returns normalized counts and scDist cannot currently be applied directly to raw counts
- When the simulation's spherical-normal and cluster-structure assumptions would bias the comparison (e.g., testing cluster-free methods, since over-clustering causes downward bias in estimated distances)

## Inputs

- Parameter specification for simData: number of cell types (nct, e.g., 10), patients per group (N1, N2, e.g., 5 each), random seed
- Target analysis method to benchmark (e.g., scDist, Augur, nDEG)

## Outputs

- Normalized simulated count matrix sim$Y (genes x cells, e.g., 1000 x 5100 for default parameters)
- Metadata data frame sim$meta.data with columns response, patient, clusters
- Ground-truth vector of true per-cell-type distances sim$D.true
- Runtime/speedup measurements (e.g., runtime_benchmark.csv timings for scDist vs Augur vs nDEG)

## How to apply

Call scDist's simData generator with explicit parameters such as nct=10 cell types and N1=5, N2=5 patients per response group, which returns a normalized 1000-gene count matrix (sim$Y) and metadata with response, patient, and clusters columns. The generator also returns sim$D.true, the vector of true between-condition distances per cell type, which serves as ground truth for evaluating whether a method's estimated distances and p-values recover the planted effects. For runtime benchmarking, scale parameters up to very large datasets (e.g., 1,000,000 cells) and time the target method with wall-clock timing, as the article did to show scDist completes 1M cells in ~10 minutes. Validate the simulation by comparing each method's out$results distance estimates against sim$D.true and checking that strongly perturbed clusters (e.g., cluster a, true distance 31.5) get significant p-values while unperturbed clusters (e.g., cluster f) do not.

## Related tools

- **scDist** (Provides the simData generator used to create simulated scRNA-seq datasets with known cell types and true distances, plus the analysis method being benchmarked) — https://github.com/phillipnicol/scDist
- **lme4** (Fits the mixed-effects model on PC scores of simulated data when validating recovery of true distances)
- **lmerTest** (Estimates degrees of freedom via Satterthwaite's approximation for p-values on simulated data)
- **Augur** (Comparison method run on simulated/subsampled data for runtime benchmarking (~5 min vs scDist's ~50 s on the COVID-19 data))
- **nDEG** (Comparison method timed on the 1M-cell simulated dataset to record the >60x speedup of scDist)
- **irlba** (Fast PCA used to obtain the low-dimensional embedding and PC loadings on the simulated count matrix)

## Examples

```
library(scDist); sim <- simData(nct=10,N1=5,N2=5); out <- scDist(sim$Y,sim$meta.data,fixed.effects="response",random.effects="patient",clusters="clusters")
```

## Evaluation signals

- Estimated distances in out$results should rank cell types consistently with sim$D.true (e.g., cluster a estimated 33.04 vs true 31.50; cluster f estimated 0.00 vs true 2.76)
- Strongly perturbed clusters should receive significant p-values (~0.0000099999, the Monte Carlo floor) while unperturbed clusters (cluster f) should not
- Output dimensions should match the specified parameters: 1000 genes x 5100 cells for nct=10, N1=5, N2=5
- Benchmark timing table should reproduce scDist ~50 s vs Augur ~5 min on the 13-sample COVID-19 data, ~5-fold average speedup across subsamples, ~10 min for 1M cells, and >60x speedup over nDEG
- Installation completes in under a minute and the demo runs in under a minute on a standard machine with R >= 4.0.0

## Limitations

- Results are sensitive to the choice of the number of principal components used to estimate the distance; no adaptive procedure for selecting K exists (fixed heuristic of 20 or 50 PCs)
- Over-clustering causes downward bias: 'As the number of clusters increases, the median distance decays towards 0', so simulated planted effects may be missed
- scDist (and hence the simulation-based benchmark) assumes normalized data and a spherical-normal variance structure; violations can lead to miscalibrated standard errors
- Runtime conclusions are hardware- and dataset-dependent; reported numbers (~50 s vs 5 min, ~10 min for 1M cells) are specific to the article's setup

## Evidence

- [readme] Generate simulated data with 10 cell types and 5 patients in each group: "Generate simulated data with 10 cell types and 5 patients in each group"
- [readme] simData returns normalized counts and metadata: "The input to `scDist` is a normalized count matrix and correpsonding metadata that describes what condition and patient each cell belongs to."
- [readme] True distances serve as ground truth: "The true distances are"
- [discussion] scDist is computationally efficient for benchmarking: "We present scDist, a statistically rigorous and computationally fast method for detecting cell-type specific differences across multiple groups or conditions."
- [readme] scDist uses mixed-effects models for replicates: "When there are multiple
replicates (i.e., patients), `scDist` uses linear-mixed effects models
to correct for sample-to-sample variability."
