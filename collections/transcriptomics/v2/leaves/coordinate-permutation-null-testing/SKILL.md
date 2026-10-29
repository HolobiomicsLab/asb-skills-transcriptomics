---
name: coordinate-permutation-null-testing
description: Use when benchmarking SVG detection methods on spatially-resolved transcriptomics
  data and you need to verify that reported p-values are calibrated by measuring false
  positive rates on coordinate-permuted null simulations.
when_to_use_negative:
- When the goal is true positive rate / power benchmarking on real spatial signal
  rather than false positive calibration under a null of no spatial autocorrelation
- When testing spatial domain marker genes (mean-level changes between domains) —
  the paper focuses on methods to identify continuously varying SVGs, not domain marker
  genes
edam_operation: http://edamontology.org/operation_2238
tools:
- name: nnSVG
  role: Runs the nearest-neighbor Gaussian process LR test on coordinate-permuted
    null data to obtain p-values for false positive rate estimation
  repo: https://bioconductor.org/packages/nnSVG
- name: Bioconductor
  role: Framework hosting nnSVG, SpatialExperiment, and STexampleData used to build
    and process the null simulations
  repo: https://bioconductor.org/packages/nnSVG
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39748-z
    title: nnSVG for the scalable identification of spatially variable genes using
      nearest-neighbor Gaussian processes
  source_content_hash: sha256:3f4be6c9d44a9bad0ac1b537a85b83bd724d845cd11c83b1691c9a5fb42ee70c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - nnSVG
  edam_operation: http://edamontology.org/operation_2238
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# Coordinate-permutation null testing

## Summary

Builds null calibration for nnSVG by permuting or shuffling spatial coordinates before running the nearest-neighbor Gaussian process model, then measures empirical false positive rates at nominal p-value cutoffs to verify p-value calibration and conservatism of the SVG likelihood-ratio test.

## When to use

Use when benchmarking SVG detection methods on spatially-resolved transcriptomics data and you need to verify that reported p-values are calibrated by measuring false positive rates on coordinate-permuted null simulations.

## When NOT to use

- When the goal is true positive rate / power benchmarking on real spatial signal rather than false positive calibration under a null of no spatial autocorrelation
- When testing spatial domain marker genes (mean-level changes between domains) — the paper focuses on methods to identify continuously varying SVGs, not domain marker genes

## Inputs

- Visium human DLPFC SpatialExperiment object from STexampleData (33538 genes x 4992 spots, filtered to in_tissue spots)
- ST mouse OB (olfactory bulb) spatial transcriptomics dataset
- Simulated datasets: 1000 genes (100 SVGs, 900 noise genes) parameterized from MOBP statistics over DLPFC spot coordinates
- Spatial coordinates matrix paired with logcounts (log-transformed normalized counts from scran)

## Outputs

- Empirical null false positive rates at adjusted p-value cutoff 0.05 (expected ~1.5% Visium DLPFC, ~1.0% ST mouse OB)
- FPR values at nominal thresholds 0.01/0.05/0.1 (reported: 0.003/0.016/0.031)
- simulation_benchmark_results.csv containing all reproduced null-calibration metrics
- Per-gene pval and padj columns in rowData of the SpatialExperiment object

## How to apply

Generate coordinate-permuted null simulations for both Visium human DLPFC and ST mouse OB datasets by shuffling spot coordinates relative to gene expression. Run nnSVG (nearest-neighbor Gaussian process with gene-specific length scale estimation and likelihood-ratio test, fitted via BRISC) on each null dataset. Compute empirical false positive rates at a nominal cutoff of 0.05 on adjusted p-values. Expect conservatism: in the medium length scale, medium expression strength simulation scenario, nnSVG's FPR was 0.003/0.016/0.031 at nominal 0.01/0.05/0.1 thresholds, and null FPRs of 1.5% (Visium DLPFC) and 1.0% (ST mouse OB) at cutoff 0.05. Write reproduced metrics to simulation_benchmark_results.csv and compare against these reported values for validation.

## Related tools

- **nnSVG** (Runs the nearest-neighbor Gaussian process LR test on coordinate-permuted null data to obtain p-values for false positive rate estimation) — https://bioconductor.org/packages/nnSVG
- **Bioconductor** (Framework hosting nnSVG, SpatialExperiment, and STexampleData used to build and process the null simulations) — https://bioconductor.org/packages/nnSVG

## Examples

```
set.seed(123); spatialCoords(spe) <- spatialCoords(spe)[sample(ncol(spe)), ]; spe <- nnSVG(spe, n_threads = 1); mean(rowData(spe)$padj <= 0.05)
```

## Evaluation signals

- Null false positive rate at adjusted-p cutoff 0.05 should be ~1.5% for Visium DLPFC and ~1.0% for ST mouse OB permutations
- FPR in the medium length scale, medium expression strength scenario should reproduce 0.003, 0.016, 0.031 at nominal p-value thresholds 0.01, 0.05, 0.1, demonstrating conservatism
- The coordinate-shuffling ablation TPR curve should follow the trend reported in the paper
- All reproduced metrics must be written to simulation_benchmark_results.csv with cutoffs 0.01/0.05/0.10 represented
- Per-gene outputs must include LR_stat, pval, and padj columns as stored in rowData of the SpatialExperiment object

## Limitations

- Small negative values are observed in estimated LR statistics, which are difficult to interpret; this occurs mainly for lower-ranked genes
- P-value calibration for low-expressed genes could be improved; adaptive filtering thresholds that carefully remove low-expressed genes have been proposed but not yet implemented
- Gene filtering (e.g. at least 3 counts in at least 0.5% of spatial locations) affects the gene universe over which FPR is computed and can reduce runtime with more stringent filtering

## Evidence

- [results] Conservatism of nnSVG under the null: [evidence span withheld — anchor status: not_found]
- [methods] Coordinate-shuffling ablation workflow: [evidence span withheld — anchor status: not_found]
- [readme] nnSVG model basis: [evidence span withheld — anchor status: not_found]
- [discussion] Small negative LR statistics limitation: "we observe some small negative values in the estimated LR statistics, which are difficult to interpret"
- [discussion] Low-expression p-value calibration gap: "This could be improved by developing adaptive filtering thresholds that carefully remove low-expressed genes"
