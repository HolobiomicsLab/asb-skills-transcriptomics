---
name: adjusted-rand-index-benchmark-evaluation
description: Use when benchmarking clustering strategies on simulated scRNA-seq data
  with known ground-truth population labels, e.g. SymSim UMI datasets varying Sigma,
  alpha, depth, and rare-population proportion.
when_to_use_negative:
- Input data lacks ground-truth population labels, so adjusted Rand index cannot be
  computed against known populations
- The task is trajectory inference or differential expression benchmarking — the SymSim
  paper evaluates those with AUROC/Spearman metrics, not ARI
- Data were simulated with batch effects via DivideBatches; the clustering benchmark
  described here uses the single-batch UMI setting
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: SymSim
  role: Simulates true and observed UMI counts with known population labels for the
    clustering benchmark
  repo: https://github.com/YosefLab/SymSim
- name: PCA
  role: Dimensionality reduction providing a latent space for k-means clustering baseline
- name: scVI
  role: Deep generative embedding whose latent space is clustered with k-means; top-performing
    method by ARI
- name: SIMLR
  role: Dimensionality reduction and clustering method compared in the benchmark
- name: Seurat
  role: Louvain graph-based clustering compared against k-means-based methods via
    ARI
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-019-10500-w
    title: Simulating multiple faceted variability in single cell RNA sequencing
  source_content_hash: sha256:5c81fa739a60b78be4de016ec1d8c9ea5fbac41fbc81a7354a82ebcc112a245c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/SymSim
  tools:
  - PCA
  - RJAGS
  - SIMLR
  - Seurat
  - SymSim
  - ggplot2
  - scVI
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# adjusted-rand-index-benchmark-evaluation

## Summary

Evaluate clustering method performance on simulated single-cell RNA-seq data by computing adjusted Rand index (ARI) between predicted and true population labels, as done in the SymSim paper to benchmark PCA/k-means, scVI/k-means, SIMLR, and Seurat Louvain clustering.

## When to use

Use when benchmarking clustering strategies on simulated scRNA-seq data with known ground-truth population labels, e.g. SymSim UMI datasets varying Sigma, alpha, depth, and rare-population proportion.

## When NOT to use

- Input data lacks ground-truth population labels, so adjusted Rand index cannot be computed against known populations
- The task is trajectory inference or differential expression benchmarking — the SymSim paper evaluates those with AUROC/Spearman metrics, not ARI
- Data were simulated with batch effects via DivideBatches; the clustering benchmark described here uses the single-batch UMI setting

## Inputs

- SymSim-simulated observed UMI count matrices from True2ObservedCounts (protocol='UMI')
- True population labels for each simulated cell from SimulateTrueCounts (Phyla5 tree, evf_type='discrete')
- Simulation parameter grid (N, Sigma, alpha_mean/alpha_sd, depth_mean/depth_sd, Prop) from Supplementary Table 6

## Outputs

- ARI results table with one row per grid combination and clustering method
- Standardized regression coefficients of ARI on simulation parameters (Fig. 5a reproduction)
- ARI-vs-N curves stratified by rare-population proportion Prop (Fig. 5b-c reproduction)

## How to apply

Simulate observed UMI counts with SymSim's SimulateTrueCounts (evf_type='discrete', Phyla5 tree) followed by True2ObservedCounts (protocol='UMI') across the parameter grid of N, Sigma (e.g. 0.4), alpha (alpha_mean/alpha_sd), depth, and rare-population proportion Prop from Supplementary Table 6. Embed cells with PCA, scVI, and SIMLR, cluster with k-means (k equal to the true number of populations) in the PCA and scVI latent spaces, and add Seurat Louvain graph-based clustering and SIMLR clustering as competing methods. For every method and grid combination, compute the adjusted Rand index between predicted and true labels using cal_ARI, and assemble the results into an ARI table. Fit a multiple linear regression of ARI on (N, Sigma, alpha, Depth, Prop) and extract standardized coefficients (Fig. 5a), then subset by Prop to plot ARI vs N curves (Fig. 5b-c). Correct application yields ARI tables with one row per grid-point per method and trends matching the paper: scVI + k-means achieves the highest ARI in most settings, ARI increases with N and decreases with smaller rare-population proportion.

## Related tools

- **SymSim** (Simulates true and observed UMI counts with known population labels for the clustering benchmark) — https://github.com/YosefLab/SymSim
- **PCA** (Dimensionality reduction providing a latent space for k-means clustering baseline)
- **scVI** (Deep generative embedding whose latent space is clustered with k-means; top-performing method by ARI)
- **SIMLR** (Dimensionality reduction and clustering method compared in the benchmark)
- **Seurat** (Louvain graph-based clustering compared against k-means-based methods via ARI)

## Examples

```
devtools::install_github('YosefLab/SymSim'); sim <- True2ObservedCounts(SimulateTrueCounts(nCells=1000, nGenes=1000, evf_type='discrete', phyla=Phyla5(), sigma=0.4, prop=0.05), protocol='UMI')
```

## Evaluation signals

- ARI table contains exactly one row per grid combination (N, Sigma, alpha, Depth, Prop) per clustering method
- scVI followed by k-means achieves the highest ARI in most simulation settings
- ARI increases with number of cells N and decreases as rare-population proportion Prop shrinks (e.g. 10% vs 5%)
- Standardized regression coefficients from the multiple linear regression qualitatively match the Fig. 5a bar plot
- k is set equal to the true number of simulated populations when running k-means

## Limitations

- Conclusions depend on SymSim's UMI simulation setting; the pipeline omits reverse transcription and library cleanup steps, so ARI rankings may not transfer to non-UMI protocols
- When the rare population accounts for 5% of cells Seurat is second best, but with 10% it performs slightly worse than Seurat and PCA/k-means — relative rankings shift across the Prop grid
- scVI is only applicable to large datasets; for small non-UMI data the paper notes this constraint, limiting scVI/k-means benchmarking to sufficiently large N

## Evidence

- [discussion] We apply three different dimensionality reduction methods to cluster cells simulated from multiple discrete populations: PCA, scVI, and SIMLR.: "We apply three different dimensionality reduction methods to cluster cells simulated from multiple discrete populations: PCA, scVI, and SIMLR."
- [abstract] SymSim can be used to benchmark clustering, differential expression, and trajectory inference methods and to examine parameter effects on their performance: "We demonstrate how SymSim can be used for benchmarking methods for clustering, differential expression and trajectory inference, and for examining the effects of various parameters on their"
- [abstract] SymSim can evaluate number of cells needed to detect rare population: "We also show how SymSim can be used to evaluate the number of cells required to detect a rare population under various scenarios."
- [methods] True2ObservedCounts converts true counts to observed: "True2ObservedCounts then simulates the library preparation and sequencing procedures, and convert the true transcript counts into observed read counts or UMI counts"
- [readme] SymSim installation instructions: "devtools::install_github("YosefLab/SymSim")"
