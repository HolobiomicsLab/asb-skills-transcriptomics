---
name: baseline-spatial-mapping-method-configuration
description: Use when benchmarking a new spatial-relationship reconstruction method
  for single-cell RNA-seq data against SageNet, CeLEry, CytoSpace, NovoSpaRc, and
  Tangram and each baseline must be run under its authors' own settings.
when_to_use_negative:
- The query SC data has a large gap from the ST reference — forcing SC assignment
  to resemble the ST can introduce more bias across mapping methods
- The goal is not local spatial reconstruction (e.g., deconvolving spot-level ST data
  or imputing genes), since this skill targets benchmarking SC spatial-relationship
  methods
- Running SageNet when the gene number is very large, since its memory consumption
  explodes with the number of genes
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CellContrast
  role: Proposed method being benchmarked; trained with infoNCE contrastive pairs
    (k=80 nearest spatial neighbors, temperature 0.05, SGD momentum 0.9, weight_decay
    5e-4, lr 0.1 cosine decay, 3,000 epochs, checkpoint epoch_3000.pt) and evaluated
    alongside the baselines
  repo: https://github.com/HKU-BAL/CellContrast
- name: SageNet
  role: 'Baseline: graph neural network trained on seqFISH spatial partitions for
    20 epochs with reported GLASSO regularization, producing an estimated cell-cell
    distance matrix'
- name: CeLEry
  role: 'Baseline: supervised deep neural network run with default parameters for
    spatial-information recovery'
- name: CytoSpace
  role: 'Baseline: SC-to-ST mapping run with ''-single-cell'' and ''-noss 10000'';
    neighbors derived from Euclidean distances of predicted coordinates'
- name: NovoSpaRc
  role: 'Baseline: optimal-transport-based assignment with num_neighbors_s/t=5, alpha_linear=0.8,
    epsilon=5e-3'
- name: Tangram
  role: 'Baseline: SC-to-ST mapping run with all genes and default parameters; neighbors
    derived from Euclidean distances of predicted coordinates'
- name: Scipy
  role: jensenshannon function used to compute the JSD between predicted and true
    cell-type proportions of the k=20 nearest neighbors
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2024.101022
    title: 'CellContrast: Reconstructing spatial relationships in single-cell RNA
      sequencing data via deep contrastive learning'
  source_content_hash: sha256:5897bc09fe22ac8f7836ad0fabda90945f3c1de0a9df98ea46c8e38a6251fa52
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/HKU-BAL/CellContrast
  tools:
  - CeLEry
  - CellContrast
  - CytoSpace
  - NovoSpaRc
  - SageNet
  - Scipy
  - Tangram
  - scran
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# baseline-spatial-mapping-method-configuration

## Summary

Configures the five SC-to-ST spatial mapping baselines (SageNet, CeLEry, CytoSpace, NovoSpaRc, Tangram) with the authors' specified settings so they can be benchmarked fairly against a contrastive-learning method like CellContrast on reconstruction tasks.

## When to use

Use when benchmarking a new spatial-relationship reconstruction method for single-cell RNA-seq data against SageNet, CeLEry, CytoSpace, NovoSpaRc, and Tangram and each baseline must be run under its authors' own settings.

## When NOT to use

- The query SC data has a large gap from the ST reference — forcing SC assignment to resemble the ST can introduce more bias across mapping methods
- The goal is not local spatial reconstruction (e.g., deconvolving spot-level ST data or imputing genes), since this skill targets benchmarking SC spatial-relationship methods
- Running SageNet when the gene number is very large, since its memory consumption explodes with the number of genes

## Inputs

- SeqFISH mouse organogenesis ST reference section with truth cell locations in .obs[['x','y']] (embryo 1 L1, 10,150 cells), log-normalized with scran, as AnnData .h5ad
- SeqFISH test sections with truth locations (embryo 1 L2, embryo 2 L2, embryo 3 L2; 351 genes), log-normalized with scran, as AnnData .h5ad
- Per-baseline configuration: SageNet spatial partitions and GLASSO setting; CeLEry defaults; CytoSpace '-single-cell' and '-noss 10000'; NovoSpaRc num_neighbors_s/t=5, alpha_linear=0.8, epsilon=5e-3; Tangram all genes, default parameters

## Outputs

- Per-sample neighbor hit ratio, Jensen-Shannon distance, and Spearman's rank correlation values for all six methods across the three test sections, written to results.csv
- Predicted spatial relationships per baseline (cell-neighbor sets from SC-SC similarities, or predicted coordinates for coordinate-output methods such as CytoSpace and Tangram)

## How to apply

Prepare the same AnnData (.h5ad) inputs for all methods — SeqFISH sections with truth locations in .obs[['x','y']], log-normalized with scran (e.g., embryo 1 L1 with 10,150 cells and 351 genes as reference). Train SageNet on seqFISH spatial partitions for 20 epochs using its reported GLASSO regularization; run CeLEry with default parameters; run CytoSpace with '-single-cell' and '-noss 10000'; configure NovoSpaRc with num_neighbors_s/t=5, alpha_linear=0.8, epsilon=5e-3; and run Tangram on all genes with default parameters. For each trained method, infer spatial relationships on the held-out test sections (embryo 1 L2, embryo 2 L2, embryo 3 L2), deriving cell neighbors from SC-SC similarities or, for coordinate-output methods like CytoSpace and Tangram, from Euclidean distances of predicted coordinates, and global positions by SC-ST mapping to the reference. Evaluate all six methods identically with three metrics: average nearest neighbor hit for k = 20, 40, ..., 200; Jensen-Shannon distance between predicted and true cell-type proportions of the k=20 nearest neighbors (scipy jensenshannon); and Spearman's rank correlation between predicted and true pairwise distances per cell.

## Related tools

- **CellContrast** (Proposed method being benchmarked; trained with infoNCE contrastive pairs (k=80 nearest spatial neighbors, temperature 0.05, SGD momentum 0.9, weight_decay 5e-4, lr 0.1 cosine decay, 3,000 epochs, checkpoint epoch_3000.pt) and evaluated alongside the baselines) — https://github.com/HKU-BAL/CellContrast
- **SageNet** (Baseline: graph neural network trained on seqFISH spatial partitions for 20 epochs with reported GLASSO regularization, producing an estimated cell-cell distance matrix)
- **CeLEry** (Baseline: supervised deep neural network run with default parameters for spatial-information recovery)
- **CytoSpace** (Baseline: SC-to-ST mapping run with '-single-cell' and '-noss 10000'; neighbors derived from Euclidean distances of predicted coordinates)
- **NovoSpaRc** (Baseline: optimal-transport-based assignment with num_neighbors_s/t=5, alpha_linear=0.8, epsilon=5e-3)
- **Tangram** (Baseline: SC-to-ST mapping run with all genes and default parameters; neighbors derived from Euclidean distances of predicted coordinates)
- **Scipy** (jensenshannon function used to compute the JSD between predicted and true cell-type proportions of the k=20 nearest neighbors)

## Examples

```
python cellContrast.py eval --ref_data_path ref_ST.h5ad --query_data_path query_ST.h5ad --model_folder cellContrast_models/ --parameter_file parameters/parameters_singleCell.json --save_path results.csv
```

## Evaluation signals

- CellContrast should reach median average hit ratio near 47.32% versus SageNet 36.22% and CeLEry 14.27% on the SeqFISH embryo 1 L1 reference setup
- Average JSD on embryo 1 L2 should be near 0.237 for CellContrast versus 0.289 SageNet and 0.324 CytoSpace
- Spearman's rho on embryo 1 L2 should be near 0.874 for CellContrast
- Neighbor hits must be computed over the full grid k = 20, 40, ..., 200 and summarized as a median average hit ratio
- results.csv must contain neighbor hit, JSD, and Spearman values for all six methods across all three test sections (embryo 1 L2, embryo 2 L2, embryo 3 L2)

## Limitations

- Tangram can generate low probabilities for matched SC-ST pairs and CytoSpace might filter out some SC cells, potentially leading to incomplete reconstructions
- SageNet's preprocessing of spatial partitions is dataset-specific and intricate, and its memory consumption explodes when the number of genes increases
- Supervised baselines like CeLEry do not fully consider local cell environment reconstruction, which can disadvantage them on neighbor-level metrics

## Evidence

- [intro] SageNet employs a graph neural network to model the gene interaction network from the ST data and produces an estimated cell-cell distance matrix for the query cells.: "SageNet employs a graph neural network to model the gene interaction network from the ST data and produces an estimated cell-cell distance matrix for the query cells."
- [intro] CytoSpace generates distinct SC-to-ST predictions based on inferred cell counts per ST spot and corresponding gene expression.: "CytoSpace generates distinct SC-to-ST predictions based on inferred cell counts per ST spot and corresponding gene expression."
- [discussion] CellContrast consistently outperformed existing methods in local spatial reconstruction, such as SageNet,14 CeLEry,15 CytoSpace,13 NovoSpaRc,11 and Tangram,12: "CellContrast consistently outperformed existing methods in local spatial reconstruction, such as SageNet,14 CeLEry,15 CytoSpace,13 NovoSpaRc,11 and Tangram,12"
- [readme] three metrics are included: nearest neighbor hit, Jessen-Shannon distance, and Spearman's rank correlation.: "three metrics are included: nearest neighbor hit, Jessen-Shannon distance, and Spearman's rank correlation."
- [intro] However, Tangram can generate low probabilities for matched SC-ST pairs, and CytoSpace might filter out some SC cells to ensure optimal subsets of samples: "However, Tangram can generate low probabilities for matched SC-ST pairs, and CytoSpace might filter out some SC cells to ensure optimal subsets of samples"
