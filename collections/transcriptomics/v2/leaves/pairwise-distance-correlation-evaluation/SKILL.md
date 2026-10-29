---
name: pairwise-distance-correlation-evaluation
description: Use when evaluating recovered 2D cell/spot coordinates from scRNA-seq
  against a spatial transcriptomics reference (Visium, MERSCOPE, MERFISH) where true
  test-set coordinates are known and methods like CeLEry, Tangram, spaOTsc, and novoSpaRc
  are being compared.
when_to_use_negative:
- Evaluating spatial domain / tissue-layer recovery, where the output is a categorical
  label and Scanpy rank_genes_groups-selected DE genes are used instead of a coordinate-distance
  metric
- Applying to held-out cells with no ground-truth spatial coordinates, since the Pearson
  correlation requires true pairwise distances on the test set
- Using it as the sole criterion when the goal is gene-expression pattern recovery
  (e.g., ADRB1 map or ligand-receptor pattern recovery), which needs separate expression-map
  comparisons
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: CeLEry
  role: Supervised deep neural network (3 hidden layers of 50, 10, 5 nodes, ReLU,
    2-dim sigmoid output) whose predicted coordinates are scored by this metric
  repo: https://github.com/QihuangZhang/CeLEry
- name: Tangram
  role: Baseline spatial mapping method evaluated on the same holdout splits for comparison
    of pairwise-distance correlations
- name: spaOTsc
  role: Optimal-transport baseline mapped to identical test coordinates for benchmark
    comparison
- name: novoSpaRc
  role: Optimal-transport baseline included in the pairwise-distance correlation comparison
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-39895-3
    title: Leveraging spatial transcriptomics data to recover cell locations in single-cell
      RNA-seq with CeLEry
  source_content_hash: sha256:e379c17e46e6931d95ca4a1acd6bcdc2c9a36eb7f73f31101c118d8b8421ffa6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/QihuangZhang/CeLEry
  tools:
  - 10x Visium
  - CeLEry
  - PyTorch
  - Scanpy
  - Tangram
  - novoSpaRc
  - spaGCN
  - spaOTsc
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7619047619047619
n_claims: 21
---

# pairwise-distance-correlation-evaluation

## Summary

A benchmarking metric from the CeLEry study that scores 2D spatial location recovery by computing the Pearson correlation between the pairwise Euclidean-distance matrices of true and predicted spot/cell coordinates, enabling quantitative comparison against Tangram, spaOTsc, and novoSpaRc.

## When to use

Use when evaluating recovered 2D cell/spot coordinates from scRNA-seq against a spatial transcriptomics reference (Visium, MERSCOPE, MERFISH) where true test-set coordinates are known and methods like CeLEry, Tangram, spaOTsc, and novoSpaRc are being compared.

## When NOT to use

- Evaluating spatial domain / tissue-layer recovery, where the output is a categorical label and Scanpy rank_genes_groups-selected DE genes are used instead of a coordinate-distance metric
- Applying to held-out cells with no ground-truth spatial coordinates, since the Pearson correlation requires true pairwise distances on the test set
- Using it as the sole criterion when the goal is gene-expression pattern recovery (e.g., ADRB1 map or ligand-receptor pattern recovery), which needs separate expression-map comparisons

## Inputs

- Ground-truth 2D coordinates of held-out spots/cells, rescaled to [0,1]
- Predicted 2D coordinates per method (CeLEry DNN, Tangram, spaOTsc, novoSpaRc) on identical train/test splits
- Preprocessed and standardized expression matrices (spot-level log-transformed UMI counts, gene-level z-scored) on spatially variable genes

## Outputs

- Pearson correlation between true and predicted pairwise-distance matrices, per method and scenario/holdout fraction
- Euclidean-distance distributions comparing predicted vs true coordinate spacing
- Benchmark comparison table (e.g., celery_mouse3_benchmark_results.csv)
- Optional structural similarity index (SSIM) between reconstructed and ground-truth 2D gene-expression maps

## How to apply

After each method predicts 2D coordinates (trained on spatially variable genes selected with spaGCN's rank_genes_groups), rescale the true spatial coordinates to the [0,1] range (matching CeLEry's sigmoid output scaling) and compute the Euclidean-distance matrix among spots for both the ground-truth and predicted coordinate sets. Compute the Pearson correlation between the upper triangles of the true and predicted pairwise-distance matrices for every method, training scenario, and holdout fraction (e.g., 10/30/50% holdoff on Visium super-pixels). Benchmark CeLEry against Tangram, spaOTsc, and novoSpaRc on identical splits: expected reference values include MERSCOPE Scenario 1-3 (CeLEry 0.72/0.69/0.74 vs Tangram 0.38) and MERFISH Mouse3_sagittal (~0.89 single-slice, ~0.90 merged-slice vs Tangram 0.68/0.63). Assemble per-scenario metrics into a comparison table (e.g., celery_mouse3_benchmark_results.csv) and, for Visium tasks, additionally compare reconstructed gene-expression maps via the structural similarity index. Acceptance is CeLEry's pairwise-distance correlation exceeding all baselines and matching the method ordering reported in the paper's figures.

## Related tools

- **CeLEry** (Supervised deep neural network (3 hidden layers of 50, 10, 5 nodes, ReLU, 2-dim sigmoid output) whose predicted coordinates are scored by this metric) — https://github.com/QihuangZhang/CeLEry
- **Tangram** (Baseline spatial mapping method evaluated on the same holdout splits for comparison of pairwise-distance correlations)
- **spaOTsc** (Optimal-transport baseline mapped to identical test coordinates for benchmark comparison)
- **novoSpaRc** (Optimal-transport baseline included in the pairwise-distance correlation comparison)

## Examples

```
from scipy.spatial.distance import pdist; from scipy.stats import pearsonr; r = pearsonr(pdist(true_xy), pdist(pred_xy))[0]
```

## Evaluation signals

- CeLEry's pairwise-distance Pearson correlation exceeds Tangram, spaOTsc, and novoSpaRc at every holdout fraction (10/30/50%) and training scenario
- Reference magnitudes reproduced: MERSCOPE Scenarios 1-3 give CeLEry 0.72/0.69/0.74 vs Tangram 0.38/— and MERFISH gives ~0.89 (single-slice) and ~0.90 (merged-slice) vs Tangram 0.68/0.63
- Predicted coordinates fall within [0,1] (consistent with the sigmoid output layer and coordinate rescaling)
- Euclidean-distance distributions of predicted coordinates match the published figure patterns (Fig. 5a-c, Fig. 6, Fig. 7)
- Reconstructed 2D gene-expression maps (e.g., ADRB1) show higher structural similarity to ground truth than baseline reconstructions

## Limitations

- CeLEry is suboptimal on single-cell-resolution platforms with small gene panels (e.g., targeted MERFISH/Xenium panels); performance is expected to improve with single-cell-resolution, transcriptome-wide ST data
- The number of VAE-augmented replicates is a user-specified hyperparameter that requires cross-validation, so benchmark scores depend on augmentation choices
- Evaluation requires held-out data with known true coordinates; on real dissociated scRNA-seq the metric cannot be computed directly

## Evidence

- [methods] We train a supervised deep neural network to learn the relationship between gene expression features and the spatial locations in the ST dataset: "We train a supervised deep neural network to learn the relationship between gene expression features and the spatial locations in the ST dataset"
- [discussion] CeLEry outperformed Tangram, spaOTsc, and novoSpaRc for spatial domain and 2D location recovery: "we demonstrate that CeLEry outperformed other state-of-the-art methods such as Tangram, spaOTsc, and novoSpaRc for both spatial domain and 2D location recovery"
- [readme] CeLEry takes spatial transcriptomic data as input for the training data and the scRNA-seq as testing data set.: "CeLEry takes spatial transcriptomic data as input for the training data and the scRNA-seq as testing data set."
- [abstract] Benchmarking on brain and cancer tissues with Visium, MERSCOPE, MERFISH, and Xenium shows CeLEry reliably recovers location information: "Our comprehensive benchmarking evaluations on multiple datasets generated from brain and cancer tissues using Visium, MERSCOPE, MERFISH, and Xenium demonstrate that CeLEry can reliably recover the"
