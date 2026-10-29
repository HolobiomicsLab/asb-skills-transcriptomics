---
name: tpr-fpr-benchmark-evaluation
description: Use when benchmarking an SVG-detection method's false positive behavior
  — e.g. validating that simulated noise genes are not called significant and that
  coordinate-permuted null data yields FPR near nominal thresholds.
when_to_use_negative:
- The goal is to identify SVGs in real experimental data rather than calibrate or
  benchmark a method's error rates — use the standard nnSVG ranking/p-value workflow
  instead
- The method under evaluation does not report per-gene adjusted p-values or LR statistics
  compatible with nominal-threshold FPR computation
- Interest is in identifying spatial domain marker genes (mean-level changes between
  domains) rather than continuously varying SVGs, which is the scope of this paper's
  benchmark
edam_operation: http://edamontology.org/operation_3793
edam_topics:
- http://edamontology.org/topic_2269
tools:
- name: nnSVG
  role: 'SVG-detection method under benchmark: run on simulated, ablated, and null
    datasets to compute TPR/FPR at nominal thresholds'
  repo: https://bioconductor.org/packages/nnSVG
- name: Bioconductor
  role: Framework providing nnSVG, SpatialExperiment, and STexampleData for loading
    and processing the Visium DLPFC and ST mouse OB inputs
  repo: https://bioconductor.org/packages/nnSVG
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39748-z
    title: nnSVG for the scalable identification of spatially variable genes using
      nearest-neighbor Gaussian processes
  source_content_hash: sha256:5eb3c15367053fec756919a819efb531ca1f28ef66bf932f199a359fbb652eaf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - nnSVG
  edam_operation: http://edamontology.org/operation_3793
  edam_topics:
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# tpr-fpr-benchmark-evaluation

## Summary

Reproduces nnSVG's simulation-based sensitivity/specificity benchmark: parameterize 1000 simulated genes (100 SVGs, 900 noise genes) from MOBP statistics on Visium DLPFC coordinates, then compute true and false positive rates at adjusted p-value cutoffs 0.01/0.05/0.10 plus coordinate-shuffled null calibration.

## When to use

Use when benchmarking an SVG-detection method's false positive behavior — e.g. validating that simulated noise genes are not called significant and that coordinate-permuted null data yields FPR near nominal thresholds.

## When NOT to use

- The goal is to identify SVGs in real experimental data rather than calibrate or benchmark a method's error rates — use the standard nnSVG ranking/p-value workflow instead
- The method under evaluation does not report per-gene adjusted p-values or LR statistics compatible with nominal-threshold FPR computation
- Interest is in identifying spatial domain marker genes (mean-level changes between domains) rather than continuously varying SVGs, which is the scope of this paper's benchmark

## Inputs

- Visium human DLPFC reference dataset (SpatialExperiment object, e.g. from STexampleData) for MOBP-derived simulation parameters
- ST mouse olfactory bulb (ST mouse OB) dataset for coordinate-permuted null calibration
- Simulated expression matrices: 1000 genes (100 SVGs, 900 noise genes) over DLPFC spot coordinates, varied by length scale and expression strength
- Coordinate-permuted (shuffled) null simulation datasets

## Outputs

- simulation_benchmark_results.csv with reproduced TPR/FPR metrics at adjusted-p-value cutoffs 0.01, 0.05, 0.10
- Ablation TPR curve from the coordinate-shuffling variant
- Null false-positive rates at cutoff 0.05 for Visium DLPFC and ST mouse OB

## How to apply

Extract MOBP gene statistics (expression strength, spatial autocorrelation) from the Visium human DLPFC reference dataset to parameterize the simulation, then simulate 1000 genes over the DLPFC spot coordinates varying length scale and expression strength. Run nnSVG (nearest-neighbor Gaussian process with gene-specific length scale estimation and likelihood-ratio test) on the main simulations and on a coordinate-shuffling ablation variant, and on coordinate-permuted null simulations for both Visium DLPFC and ST mouse OB data. Compute TPR and FPR at adjusted-p-value cutoffs 0.01, 0.05, and 0.10 for the main simulations, the ablation TPR curve, and null FPR at cutoff 0.05, writing all metrics to a CSV (e.g. simulation_benchmark_results.csv). Judge correctness by comparing reproduced FPRs to the reported 0.003/0.016/0.031 at nominal 0.01/0.05/0.1 and null FPRs of 1.5% (Visium DLPFC) and 1.0% (ST mouse OB) at cutoff 0.05.

## Related tools

- **nnSVG** (SVG-detection method under benchmark: run on simulated, ablated, and null datasets to compute TPR/FPR at nominal thresholds) — https://bioconductor.org/packages/nnSVG
- **Bioconductor** (Framework providing nnSVG, SpatialExperiment, and STexampleData for loading and processing the Visium DLPFC and ST mouse OB inputs) — https://bioconductor.org/packages/nnSVG

## Examples

```
spe <- Visium_humanDLPFC(); spe <- filter_genes(spe); set.seed(123); spe <- nnSVG(spe, n_threads = 1)
```

## Evaluation signals

- Reproduced FPRs at nominal p-value thresholds 0.01/0.05/0.1 match the reported 0.003/0.016/0.031 in the medium length scale, medium expression strength scenario
- Coordinate-permuted null FPR at cutoff 0.05 matches reported 1.5% (Visium DLPFC) and 1.0% (ST mouse OB), confirming conservatism
- Ablation (coordinate-shuffling) TPR curve follows the reported trend across length scale and expression strength settings
- Simulated dataset structure holds: 1000 genes with 100 true SVGs and 900 noise genes parameterized from MOBP statistics
- All metrics present in simulation_benchmark_results.csv for main simulations, ablation, and null runs

## Limitations

- Transcriptome-wide datasets with >=10,000 spatial locations require runtimes of several hours on 10 processor cores of a high-performance compute cluster
- Some small negative values occur in estimated LR statistics, which are difficult to interpret, mainly for lower-ranked genes
- P-value calibration for low-expressed genes could be improved; adaptive filtering thresholds to remove low-expressed genes are proposed but not yet implemented

## Evidence

- [results] Conservative FPR under simulation: [evidence span withheld — anchor status: not_found]
- [abstract] Linear scaling claim: "scales linearly with the number of spatial locations"
- [methods] Simulation design grounded in real data: [evidence span withheld — anchor status: not_found]
- [results] Null calibration targets: [evidence span withheld — anchor status: not_found]
- [discussion] Runtime limitation: "runtimes are on the order of several hours when using 10 processor cores on a high-performance compute cluster"
