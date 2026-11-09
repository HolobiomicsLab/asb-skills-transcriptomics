---
name: spatial-mapping-baseline-benchmarking
description: Use when a supervised location-recovery model (e.g., CeLEry) must be
  compared against mapping baselines on the same spatial transcriptomics/scRNA-seq
  splits, using pairwise-distance Pearson correlation as the headline metric.
when_to_use_negative:
- The task is spatial domain/layer classification with categorical labels rather than
  continuous 2D coordinate recovery — CeLEry's output layer and metric differ (categorical
  output, differentially expressed genes via Scanpy).
- No spatial transcriptomics reference with known locations exists for training, since
  all compared methods require ST data to learn or infer the mapping.
- Single-cell-resolution platforms with small gene panels are the only option — CeLEry
  is reportedly suboptimal in that regime.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CeLEry
  role: Supervised deep learning method whose 2D location recovery is being benchmarked;
    trained on rescaled [0,1] coordinates with a 2-dimensional sigmoid output
  repo: https://github.com/QihuangZhang/CeLEry
- name: Tangram
  role: Baseline spatial mapping method run on the same scenario splits for comparison
- name: spaOTsc
  role: Optimal-transport baseline that constructs the spatial mapping of cells on
    identical splits
- name: novoSpaRc
  role: Optimal-transform baseline for spatially mapping cells to tissue locations
    on identical splits
- name: spaGCN
  role: Source of the rank_genes_groups function used to select spatially variable
    genes for 2D location recovery
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-39895-3
    title: Leveraging spatial transcriptomics data to recover cell locations in single-cell
      RNA-seq with CeLEry
  source_content_hash: sha256:cc986fbc4d62d476eb50997623d135915964ba931b0c930cd1da6de704732f2a
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
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# spatial-mapping-baseline-benchmarking

## Summary

Benchmark a spatial location recovery method (CeLEry) against baseline spatial mapping methods — Tangram, spaOTsc, and novoSpaRc — on shared train/test splits, scoring recovery quality by the Pearson correlation between pairwise Euclidean-distance matrices of true and predicted 2D coordinates.

## When to use

Use when a supervised location-recovery model (e.g., CeLEry) must be compared against mapping baselines on the same spatial transcriptomics/scRNA-seq splits, using pairwise-distance Pearson correlation as the headline metric.

## When NOT to use

- The task is spatial domain/layer classification with categorical labels rather than continuous 2D coordinate recovery — CeLEry's output layer and metric differ (categorical output, differentially expressed genes via Scanpy).
- No spatial transcriptomics reference with known locations exists for training, since all compared methods require ST data to learn or infer the mapping.
- Single-cell-resolution platforms with small gene panels are the only option — CeLEry is reportedly suboptimal in that regime.

## Inputs

- Spatial transcriptomics reference with single-cell resolution and 2D coordinates (Vizgen MERSCOPE mouse brain replicates S1R1 and S1R2)
- Query scRNA-seq data or held-out replicate sections to be mapped
- Cell-by-gene expression matrices after filtering, log normalization, and gene-level standardization
- True 2D spatial coordinates per cell for computing distance matrices

## Outputs

- Pairwise-distance Pearson correlation per method per scenario (CeLEry 0.72/0.69/0.74 vs. Tangram, spaOTsc, novoSpaRc for Scenarios 1–3)
- Euclidean-distance distributions comparing true vs. predicted coordinates for all methods
- Recovered per-cell 2D coordinate predictions usable for downstream expression mapping (e.g., ADRB1 signal across predicted locations)

## How to apply

First preprocess the spatial reference (e.g., Vizgen MERSCOPE mouse brain replicates S1R1 and S1R2) by eliminating cells with low UMI counts or a small number of expressed genes, applying cell-level log transformation and gene-level standardization, and selecting spatially variable genes via the rank_genes_groups function in the spaGCN package. Define the three scenario splits: Scenario 1 splits one replicate into left/right halves, Scenario 2 trains on S1R2 and tests on S1R1, and Scenario 3 merges both replicates into a combined training set by subsetting common genes. Rescale the 2D spatial coordinates to the [0,1] range, then train CeLEry's 3-hidden-layer network (50, 10, 5 nodes, ReLU activations, 2-dimensional sigmoid output, squared-error coordinate loss) and run Tangram, spaOTsc, and novoSpaRc on identical splits. For each method, compute the Pearson correlation between the pairwise Euclidean-distance matrices of true and predicted coordinates, which measures whether relative spatial structure — not just absolute positions — is recovered; report Euclidean-distance distributions and a downstream expression map (e.g., ADRB1 aggregated across predicted locations) as supporting visual checks.

## Related tools

- **CeLEry** (Supervised deep learning method whose 2D location recovery is being benchmarked; trained on rescaled [0,1] coordinates with a 2-dimensional sigmoid output) — https://github.com/QihuangZhang/CeLEry
- **Tangram** (Baseline spatial mapping method run on the same scenario splits for comparison)
- **spaOTsc** (Optimal-transport baseline that constructs the spatial mapping of cells on identical splits)
- **novoSpaRc** (Optimal-transform baseline for spatially mapping cells to tissue locations on identical splits)
- **spaGCN** (Source of the rank_genes_groups function used to select spatially variable genes for 2D location recovery)

## Examples

```
pip install CeLEryPy
```

## Evaluation signals

- CeLEry's pairwise-distance Pearson correlation should reach ~0.72 (Scenario 1), ~0.69 (Scenario 2), and ~0.74 (Scenario 3), exceeding Tangram (~0.38), spaOTsc (~0.67), and novoSpaRc (~0.56) in Scenario 1.
- True and predicted 2D coordinate matrices must have comparable pairwise Euclidean-distance distributions; plot them per method as a sanity check.
- Predicted coordinates fall within the [0,1] rescaled range consistent with the sigmoid output layer.
- The downstream ADRB1 expression map aggregated over CeLEry's predicted locations should match the spatial pattern of Fig. 6.
- All methods must be evaluated on identical train/test splits (left/right half split, S1R2→S1R1, merged-replicate common-gene subset) for the comparison to be valid.

## Limitations

- Software versions for CeLEry and competing tools (Tangram, spaOTsc, novoSpaRc) are not reported in the article text, complicating exact reproduction.
- CeLEry is suboptimal on single-cell-resolution platforms with small gene panels; performance should improve with transcriptome-wide, single-cell-resolution ST references.
- Pairwise-distance correlation measures relative spatial structure; it does not certify absolute coordinate accuracy and should be paired with distance-distribution and expression-map checks.

## Evidence

- [results] CeLEry achieved a pairwise-distance Pearson correlation of 0.72 in Scenario 1: [evidence span withheld — anchor status: not_found]
- [discussion] Baseline ranking claim: "we demonstrate that CeLEry outperformed other state-of-the-art methods such as Tangram, spaOTsc, and novoSpaRc for both spatial domain and 2D location recovery"
- [methods] SVG selection step: "Spatially variable genes are selected using the rank_genes_groups function in the spaGCN package (for 2D location recovery)"
- [methods] Cell filtering step: "cells with low UMI counts or a small number of expressed genes were eliminated"
- [intro] novoSpaRc baseline description: [evidence span withheld — anchor status: not_found]
- [readme] CeLEry training procedure: "A deep neural network is trained to learn the relationship between the spotwise gene expression and location information"
