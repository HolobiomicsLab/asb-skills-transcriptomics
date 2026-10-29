---
name: spatial-transcriptomics-simulation-framework
description: Use when benchmarking or validating an SVG detection method's p-value
  calibration and sensitivity — e.g., measuring FPR at nominal 0.01/0.05/0.1 cutoffs,
  running coordinate-shuffle ablations, or building coordinate-permuted null simulations
  for spatially-resolved transcriptomics data.
when_to_use_negative:
- The goal is identifying SVGs in real experimental data rather than validating method
  calibration — run nnSVG directly on the dataset instead of building simulations
- The analysis targets spatial domain marker genes (mean-level changes between domains)
  — the framework addresses continuously varying SVGs, and the paper explicitly focuses
  'on methods to identify SVGs, not spatial domain marker genes'
- The input data lack spatial coordinates or a defined spot layout, so coordinates
  cannot be permuted for null simulation
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: nnSVG
  role: The SVG detection method under evaluation; run on simulated datasets, coordinate-shuffle
    ablations, and coordinate-permuted nulls to measure TPR/FPR
  repo: https://bioconductor.org/packages/nnSVG
- name: Bioconductor
  role: Framework for installing nnSVG with its dependencies (SpatialExperiment, STexampleData)
    and hosting the SpatialExperiment input format
  repo: https://bioconductor.org/packages/nnSVG
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39748-z
    title: nnSVG for the scalable identification of spatially variable genes using
      nearest-neighbor Gaussian processes
  source_content_hash: sha256:db87ca0038c5c13299d0cfe39f447bed98d6ecb17aed6e37f926b477368244cb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - nnSVG
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# spatial-transcriptomics-simulation-framework

## Summary

A simulation-based validation framework for spatially variable gene (SVG) detection methods, in which 1000 genes (100 SVGs, 900 noise genes) parameterized from MOBP statistics are simulated over Visium DLPFC spot coordinates with varied length scale and expression strength, and false positive/true positive rates are measured against nominal thresholds.

## When to use

Use when benchmarking or validating an SVG detection method's p-value calibration and sensitivity — e.g., measuring FPR at nominal 0.01/0.05/0.1 cutoffs, running coordinate-shuffle ablations, or building coordinate-permuted null simulations for spatially-resolved transcriptomics data.

## When NOT to use

- The goal is identifying SVGs in real experimental data rather than validating method calibration — run nnSVG directly on the dataset instead of building simulations
- The analysis targets spatial domain marker genes (mean-level changes between domains) — the framework addresses continuously varying SVGs, and the paper explicitly focuses 'on methods to identify SVGs, not spatial domain marker genes'
- The input data lack spatial coordinates or a defined spot layout, so coordinates cannot be permuted for null simulation

## Inputs

- SpatialExperiment object of Visium human DLPFC (e.g., via STexampleData::Visium_humanDLPFC(), 33538 genes x 4992 spots, tissue spots retained in_tissue == 1)
- log-transformed normalized counts (logcounts) matrix computed with scran computeLibraryFactors/logNormCounts after filter_genes()
- Numeric matrix of spatial spot coordinates for simulation overlays and coordinate shuffling/permutation

## Outputs

- simulation_benchmark_results.csv with TPR and FPR at adjusted-p-value cutoffs 0.01, 0.05, and 0.10
- Null-calibration false positive rates at cutoff 0.05 for Visium DLPFC and ST mouse OB permuted datasets
- TPR curve across the coordinate-shuffling ablation variants
- Per-gene nnSVG results (LR_stat, rank, pval, padj, prop_sv, phi) stored in rowData of the SpatialExperiment object

## How to apply

First extract gene statistics (expression strength, spatial autocorrelation) from a known SVG such as MOBP in the Visium human DLPFC reference dataset to parameterize the simulation. Simulate 1000 genes over the DLPFC spot coordinates, varying length scale (phi) and expression strength across scenarios (e.g., medium/medium), mixing 100 SVGs with 900 noise genes. Run nnSVG — the nearest-neighbor Gaussian process model with gene-specific length scale estimation and a likelihood-ratio (LR) test — on the simulations, on a coordinate-shuffling ablation variant, and on coordinate-permuted null datasets for both Visium DLPFC and ST mouse OB. Compute true positive and false positive rates at adjusted-p-value cutoffs 0.01, 0.05, and 0.10, and null false positive rates at cutoff 0.05, writing metrics to simulation_benchmark_results.csv. Correct application reproduces the reported conservative FPR behavior (0.003/0.016/0.031 at 0.01/0.05/0.1) and null FPRs of 1.5% (Visium DLPFC) and 1.0% (ST mouse OB).

## Related tools

- **nnSVG** (The SVG detection method under evaluation; run on simulated datasets, coordinate-shuffle ablations, and coordinate-permuted nulls to measure TPR/FPR) — https://bioconductor.org/packages/nnSVG
- **Bioconductor** (Framework for installing nnSVG with its dependencies (SpatialExperiment, STexampleData) and hosting the SpatialExperiment input format) — https://bioconductor.org/packages/nnSVG

## Examples

```
spe <- Visium_humanDLPFC(); spe <- spe[, colData(spe)$in_tissue == 1]; spe <- filter_genes(spe); spe <- computeLibraryFactors(spe); spe <- logNormCounts(spe); set.seed(123); spe <- nnSVG(spe, n_threads = 1)
```

## Evaluation signals

- Reproduced FPRs in the medium length scale / medium expression strength scenario equal 0.003, 0.016, and 0.031 at nominal p-value thresholds 0.01, 0.05, and 0.1
- Null-calibration false positive rates at cutoff 0.05 match the reported 1.5% (Visium DLPFC) and 1.0% (ST mouse OB), confirming conservative behavior
- The ablation TPR curve follows the reported trend across length-scale/expression-strength settings
- nnSVG results for simulated genes populate the expected rowData columns (LR_stat, rank, pval, padj, prop_sv) in the SpatialExperiment object
- All reproduced metrics are written to simulation_benchmark_results.csv with a fixed seed for reproducibility

## Limitations

- Small negative values in estimated LR statistics can occur, mainly for lower-ranked genes, and are difficult to interpret
- P-value calibration for low-expressed genes could be improved via adaptive filtering thresholds, which are not yet implemented
- Transcriptome-wide datasets with >=10,000 spatial locations require runtimes of several hours with 10 processor cores on a high-performance compute cluster
- Runtime depends on the number of genes; more stringent gene filtering can reduce it

## Evidence

- [abstract] Performance was demonstrated using experimental data from several technological platforms and simulations: "We demonstrate the performance of our method using experimental data from several technological platforms and simulations"
- [discussion] Negative LR statistics limitation: "we observe some small negative values in the estimated LR statistics, which are difficult to interpret"
- [discussion] P-value calibration caveat for low-expressed genes: "This could be improved by developing adaptive filtering thresholds that carefully remove low-expressed genes, which could also improve the calibration of p-values for low-expressed genes"
- [readme] nnSVG results stored in rowData: "The main results of interest are:

- `LR_stat`: likelihood ratio (LR) statistics used to rank SVGs
- `rank`: rank of top SVGs according to LR statistics"
- [readme] Gene filtering default parameters used before simulation: "# filter low-expressed and mitochondrial genes
# using function from nnSVG package with default filtering parameters
spe <- filter_genes(spe)"
