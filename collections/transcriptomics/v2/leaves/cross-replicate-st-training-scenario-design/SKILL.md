---
name: cross-replicate-st-training-scenario-design
description: Use when benchmarking 2D location or spatial domain recovery on ST data
  with replicate sections, and you need training scenarios ranging from same-section
  splits to cross-replicate transfer with merged training sets.
when_to_use_negative:
- Only a single non-replicate ST section is available, making cross-replicate or merged-training
  scenarios impossible
- The goal is spatial domain (categorical) recovery, which requires differentially
  expressed genes via Scanpy's rank_genes_groups rather than spatially variable gene
  selection
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3512
tools:
- name: CeLEry
  role: Supervised deep neural network trained on each scenario split to predict 2D
    coordinates
  repo: https://github.com/QihuangZhang/CeLEry
- name: Tangram
  role: Baseline spatial mapping method run on the same scenario splits
- name: spaOTsc
  role: Optimal-transport baseline spatial mapping method on the same splits
- name: novoSpaRc
  role: Optimal-transform baseline spatial mapping method on the same splits
- name: spaGCN
  role: Provides rank_genes_groups for spatially variable gene selection used in split
    construction for 2D location recovery
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-39895-3
    title: Leveraging spatial transcriptomics data to recover cell locations in single-cell
      RNA-seq with CeLEry
  source_content_hash: sha256:badcb98dc9449a41bd114e188a77e98113b2b277505f08e9b2051663af56bd08
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/QihuangZhang/CeLEry
  tools:
  - CeLEry
  - Tangram
  - novoSpaRc
  - spaGCN
  - spaOTsc
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# cross-replicate-st-training-scenario-design

## Summary

Designs intra- and cross-replicate training/testing splits of spatial transcriptomics reference sections (e.g., Vizgen MERSCOPE mouse brain S1R1/S1R2) to benchmark spatial location recovery methods under progressively harder generalization scenarios.

## When to use

Use when benchmarking 2D location or spatial domain recovery on ST data with replicate sections, and you need training scenarios ranging from same-section splits to cross-replicate transfer with merged training sets.

## When NOT to use

- Only a single non-replicate ST section is available, making cross-replicate or merged-training scenarios impossible
- The goal is spatial domain (categorical) recovery, which requires differentially expressed genes via Scanpy's rank_genes_groups rather than spatially variable gene selection

## Inputs

- Vizgen MERSCOPE mouse brain replicate sections (S1R1, S1R2) with per-cell coordinates, UMI counts, and gene expression matrix
- Common gene panel shared across replicate sections (for merged training in Scenario 3)

## Outputs

- Three train/test splits: within-section left/right halves (Scenario 1), cross-replicate S1R2→S1R1 (Scenario 2), and merged-replicate training set (Scenario 3)
- Filtered, log-transformed, gene-standardized expression matrices on selected spatially variable genes per scenario
- Spatial coordinates rescaled to [0,1] for each scenario split

## How to apply

Construct three nested scenarios from replicate sections: Scenario 1 splits one replicate (e.g., MERSCOPE S1R1) into left/right halves for within-section training/testing; Scenario 2 trains on one replicate (S1R2) and tests on the other (S1R1) to test cross-replicate generalization; Scenario 3 merges both replicates into a combined training set after subsetting to common genes. Before splitting, filter cells with low UMI counts or few expressed genes, apply cell-level log transformation and gene-level standardization, and select spatially variable genes with spaGCN's rank_genes_groups for 2D location recovery. Rescale 2D coordinates to the [0,1] range so the same splits can train the CeLEry neural network and feed identical inputs to Tangram, spaOTsc, and novoSpaRc baselines. The rationale is that harder scenarios isolate how well gene-expression-to-location relationships learned from one tissue section transfer to an independently generated replicate.

## Related tools

- **CeLEry** (Supervised deep neural network trained on each scenario split to predict 2D coordinates) — https://github.com/QihuangZhang/CeLEry
- **Tangram** (Baseline spatial mapping method run on the same scenario splits)
- **spaOTsc** (Optimal-transport baseline spatial mapping method on the same splits)
- **novoSpaRc** (Optimal-transform baseline spatial mapping method on the same splits)
- **spaGCN** (Provides rank_genes_groups for spatially variable gene selection used in split construction for 2D location recovery)

## Examples

```
import CeLEry; model = CeLEry.trainCeLEry(trainX=merscope_S1R1_left_spatialvg, trainCoord=rescaled_2d_coords, hidden_layer_size=[50,10,5])
```

## Evaluation signals

- Scenario 1: CeLEry pairwise-distance Pearson correlation of 0.72, exceeding Tangram (0.38), spaOTsc (0.67), and novoSpaRc (0.56) as in Fig. 6
- CeLEry target correlations across scenarios: 0.72 (Scenario 1), 0.69 (Scenario 2), 0.74 (Scenario 3); Scenario 3 (merged training) should not degrade relative to within-section performance
- Euclidean-distance distributions of true vs. predicted coordinates match published Fig. 6 patterns across all methods
- Predicted per-cell 2D coordinates used to render the ADRB1 expression recovery map reproduce the published figure pattern

## Limitations

- CeLEry is suboptimal on single-cell-resolution platforms with small gene panels; performance depends on transcriptome-wide or sufficiently broad gene coverage
- The number of augmentation replicates is a user-specified hyperparameter requiring cross-validation rather than a fixed recommended setting
- Software versions of CeLEry and competing tools (Tangram, spaOTsc, novoSpaRc) are not reported in the section text

## Evidence

- [other] Scenario 1 splits one replicate into left/right halves, Scenario 2 trains on S1R2 and tests on S1R1, and Scenario 3 merges both replicates into a combined training set by subsetting common genes.: [evidence span withheld — anchor status: not_found]
- [results] CeLEry achieved a pairwise-distance Pearson correlation of 0.72 in Scenario 1, outperforming Tangram (0.38), spaOTsc (0.67), and novoSpaRc (0.56).: [evidence span withheld — anchor status: not_found]
- [methods] Filter cells with low UMI counts or a small number of expressed genes, apply cell-level log transformation and gene-level standardization, and select spatially variable genes with spaGCN's rank_genes_groups for 2D location recovery.: "cells with low UMI counts or a small number of expressed genes were eliminated"
- [discussion] The number of replicates to be generated is a hyperparameter prespecified by users: "The number of replicates to be generated is a hyperparameter prespecified by users"
- [readme] Comprehensive evaluations on multiple datasets generated from mouse and human brains show that CeLEry can reliably recover the spatial location information for cells in scRNA-seq.: "Comprehensive evaluations on multiple datasets generated from mouse and human brains show that CeLEry can reliably recover the spatial location information"
