---
name: cross-method-accuracy-comparison
description: Use when you need to quantify whether a supervised annotator (NeuCA)
  outperforms existing clustering/annotation methods across training-size and testing-size
  regimes on scRNA-seq data with known labels.
when_to_use_negative:
- No training data with known labels exist for the tissue of interest — supervised
  annotation is not viable and unsupervised methods are the only option
- The new sample is expected to contain cell types absent from the reference/training
  data — NeuCA is designed only for settings where all cell types already exist in
  the reference
- The goal is detecting novel or rare/unknown cell types rather than exhaustive label
  assignment among known types
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: NeuCA
  role: Supervised neural-network classifier (feed-forward or hierarchical, chosen
    via tau=0.95 correlation threshold) whose annotation accuracy is benchmarked
  repo: https://bioconductor.org/packages/NeuCA/
- name: Seurat
  role: Unsupervised clustering baseline evaluated by ARI on the same simulation splits
- name: SC3
  role: Unsupervised clustering baseline evaluated by ARI on the same simulation splits
- name: R
  role: R/Bioconductor environment in which NeuCA, Seurat, and SC3 are run and metrics
    are aggregated into the results CSV
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-021-04473-4
    title: A neural network-based method for exhaustive cell label assignment using
      single cell RNA-seq data
  source_content_hash: sha256:0f19cdf2dc555ba0b717d601a2349ebc5efd39c54bda936b8a1c659bafe1fdc2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - NeuCA
  - R
  - SC3
  - Seurat
  - beyondcell
  - devtools
  - rTensor
  - scDEAL
  - scRank
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# cross-method-accuracy-comparison

## Summary

Benchmarks NeuCA against unsupervised baselines (Seurat, SC3) on simulated PBMC cell-annotation tasks, varying training proportion (10/20/50/80%) and testing size (800/1600/4000 cells) and scoring accuracy (accurately assigned rate) and ARI.

## When to use

Use when you need to quantify whether a supervised annotator (NeuCA) outperforms existing clustering/annotation methods across training-size and testing-size regimes on scRNA-seq data with known labels.

## When NOT to use

- No training data with known labels exist for the tissue of interest — supervised annotation is not viable and unsupervised methods are the only option
- The new sample is expected to contain cell types absent from the reference/training data — NeuCA is designed only for settings where all cell types already exist in the reference
- The goal is detecting novel or rare/unknown cell types rather than exhaustive label assignment among known types

## Inputs

- Real PBMC scRNA-seq count data downloaded from the 10X Genomics website (https://support.10xgenomics.com/)
- Normalized expression matrix with training and testing subsets taken over overlapped genes
- Training-set cell type labels used to fit the NeuCA classifier
- Simulation splits defined by training proportion p in {10%, 20%, 50%, 80%} and testing size n in {800, 1600, 4000} cells

## Outputs

- NeuCA accurately assigned rate per training-proportion and testing-size setting
- ARI values for Seurat and SC3 per setting
- pbmc_simulation_benchmark_results.csv containing all computed metrics

## How to apply

Download real PBMC data from the 10X Genomics website, normalize the expression matrix, and subset to overlapped genes to build simulation datasets. Draw p% of cells as the training set with p in {10%, 20%, 50%, 80%} and n test cells from the remainder, with n varied in {800, 1600, 4000}; build three settings — all major types, five T cell subtypes only, and major types with T cells merged. Train the NeuCA classifier on each split (tau=0.95 correlation threshold selects feed-forward vs hierarchical network, ReLU activation, RMSprop, mini-batch training) and run Seurat and SC3 on the same splits. Compute NeuCA's accurately assigned rate and Seurat/SC3 ARI per setting, and write all metrics to pbmc_simulation_benchmark_results.csv for side-by-side comparison.

## Related tools

- **NeuCA** (Supervised neural-network classifier (feed-forward or hierarchical, chosen via tau=0.95 correlation threshold) whose annotation accuracy is benchmarked) — https://bioconductor.org/packages/NeuCA/
- **Seurat** (Unsupervised clustering baseline evaluated by ARI on the same simulation splits)
- **SC3** (Unsupervised clustering baseline evaluated by ARI on the same simulation splits)
- **R** (R/Bioconductor environment in which NeuCA, Seurat, and SC3 are run and metrics are aggregated into the results CSV)

## Examples

```
predicted.label <- NeuCA(train = train_sce, test = test_sce, model.size = "big", verbose = FALSE)
```

## Evaluation signals

- As testing size grows from 800 to 4000 cells, Seurat's ARI rises from about 0.45 to 0.5 and SC3's from about 0.60 to 0.65
- NeuCA's accurately assigned rate rises from about 0.86 to 0.90 as the training proportion increases
- NeuCA remains the highest performer (accuracy and ARI) over all compared existing methods in every training/testing-size setting
- All per-setting metrics are present in pbmc_simulation_benchmark_results.csv for NeuCA, Seurat, and SC3 across the three simulation settings

## Limitations

- NeuCA fits only situations where all cell types in the new sample already exist in the reference/training data
- Cross-condition prediction demonstrated on ASD data may not hold in other scenarios
- Deciding whether unknown/novel cell types exist in the data is not addressed by the method; expert consultation is recommended

## Evidence

- [methods] PBMC simulation source data: "Real PBMC data from the 10X Genomics website (https://support.10xgenomics.com/)31 were downloaded and processed to generate our simulation datasets"
- [intro] Supervised vs unsupervised rationale: "when training data are available, the improved accuracy, speed, and reproducibility of supervised methods easily outperform unsupervised ones"
- [methods] Correlation strategy threshold: "NeuCA first computes the averaged expression profile matrix for all the cell types in the training data"
- [readme] NeuCA automatic model choice: "NeuCA can detect whether highly-correlated cell types exist in the training dataset, and automatically determine if a general neural-network model will be adopted"
