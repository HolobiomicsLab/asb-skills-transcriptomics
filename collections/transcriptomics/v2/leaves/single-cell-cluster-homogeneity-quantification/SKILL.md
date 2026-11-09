---
name: single-cell-cluster-homogeneity-quantification
description: Use when a scDEED-optimized t-SNE/UMAP visualization raises the question
  of whether a visual cluster is trustworthy or should be divided, and you need a
  numeric purity score per cluster and per merged cluster combination.
when_to_use_negative:
- When the input is raw counts with no existing cluster assignments — cluster labels
  must already be defined (e.g., from a Seurat workflow under an optimized t-SNE)
  before purity can be scored
- When the question is about embedding quality or hyperparameter choice (perplexity/min.dist)
  rather than cluster homogeneity — use scDEED's dubious-embedding detection instead
- When clusters come from unoptimized embeddings, since t-SNE and UMAP 2D embeddings
  may not reliably inform similarities among cell clusters
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Rogue
  role: R package whose CalculateRogue function computes the per-cluster and merged-cluster
    purity scores compared across Hydra and CAR-T cluster combinations
- name: Seurat
  role: Reads the Seurat objects and provides the cluster identity labels consumed
    by CalculateRogue; scDEED uses Seurat objects
- name: scDEED
  role: Optimized the t-SNE hyperparameters under which the CAR-T cluster assignments
    (clusters 3/4/9/11) used as input to the purity analysis were defined
  repo: https://github.com/JSB-UCLA/scDEED
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-45891-y
    title: Statistical method scDEED for detecting dubious 2D single-cell embeddings
      and optimizing t-SNE and UMAP hyperparameters
  source_content_hash: sha256:c1c679fc27e48a17d51488403be8c92af940b1b8d570e2b1f09dd46774544737
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/JSB-UCLA/scDEED
  tools:
  - R
  - Rogue
  - Seurat
  - scDEED
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# single-cell-cluster-homogeneity-quantification

## Summary

Quantify the homogeneity/purity of single-cell clusters with the ROGUE metric to test whether merging or splitting clusters (e.g., the five Hydra neuron ectodermal subtypes, or CAR-T clusters 3/4/9/11) meaningfully changes cluster purity.

## When to use

Use when a scDEED-optimized t-SNE/UMAP visualization raises the question of whether a visual cluster is trustworthy or should be divided, and you need a numeric purity score per cluster and per merged cluster combination.

## When NOT to use

- When the input is raw counts with no existing cluster assignments — cluster labels must already be defined (e.g., from a Seurat workflow under an optimized t-SNE) before purity can be scored
- When the question is about embedding quality or hyperparameter choice (perplexity/min.dist) rather than cluster homogeneity — use scDEED's dubious-embedding detection instead
- When clusters come from unoptimized embeddings, since t-SNE and UMAP 2D embeddings may not reliably inform similarities among cell clusters

## Inputs

- Seurat object with published cluster identity labels (e.g., five Hydra neuron ectodermal subtypes)
- Seurat object cluster assignments defined under the scDEED-optimized t-SNE (CAR-T clusters)
- Cluster membership vector / identity column mapping cells to cluster IDs

## Outputs

- Per-cluster ROGUE purity scores
- ROGUE score for unified/merged cluster combinations
- Average and standard deviation of per-cluster ROGUE values
- rogue_scores.csv compiling all recomputed ROGUE values

## How to apply

Load published Seurat objects with their cluster labels (e.g., the five Hydra neuron ectodermal subtypes and CAR-T clusters defined under the scDEED-optimized t-SNE) into R via Seurat. Install and load the R package Rogue, then run CalculateRogue separately on each individual cluster, on the unified cluster (e.g., all five ectodermal neuron subtypes merged), and on merged combinations of interest (e.g., CAR-T clusters 3+4+9+11). Compute the average and standard deviation across per-cluster ROGUE values to establish a reference purity distribution. Compile all recomputed values into rogue_scores.csv. Judgment: unification is justified only if the merged cluster's ROGUE does not meaningfully drop below the per-cluster average (Hydra: per-subtype mean 0.768, sd 0.055 vs unified 0.744; CAR-T: 0.887/0.885/0.886 vs 0.832 for merged 3+4+9+11, where the drop indicates the merge is not homogeneous).

## Related tools

- **Rogue** (R package whose CalculateRogue function computes the per-cluster and merged-cluster purity scores compared across Hydra and CAR-T cluster combinations)
- **Seurat** (Reads the Seurat objects and provides the cluster identity labels consumed by CalculateRogue; scDEED uses Seurat objects)
- **scDEED** (Optimized the t-SNE hyperparameters under which the CAR-T cluster assignments (clusters 3/4/9/11) used as input to the purity analysis were defined) — https://github.com/JSB-UCLA/scDEED

## Examples

```
library(Seurat); library(Rogue); rogue_val <- CalculateRogue(seu, identity = "seurat_clusters")
```

## Evaluation signals

- Recomputed Hydra per-subtype ROGUE values should be 0.710/0.784/0.714/0.793/0.839 with average 0.768 and standard deviation 0.055
- Unified five-subtype Hydra cluster ROGUE should equal approximately 0.744, i.e., not meaningfully below the per-subtype average
- CAR-T cluster combination ROGUE under the optimized t-SNE should be 0.887/0.885/0.886, versus 0.832 for the merged clusters 3+4+9+11
- rogue_scores.csv should contain every recomputed per-cluster and merged-combination score and match the published values
- All scores must be computed on the same cluster label set (Seurat identity column) used in the published analysis

## Limitations

- t-SNE and UMAP 2D embeddings may not reliably inform similarities among cell clusters, so cluster boundaries fed into the purity analysis depend on the embedding used
- Seurat version differences (5.2.1 vs 5.1.0) can affect package installation and reproducibility of the Seurat-object-based workflow
- Merging clusters only reduces purity meaningfully when the combined ROGUE drops below the per-cluster reference distribution (as with CAR-T 3+4+9+11: 0.832 vs ~0.886), not merely because clusters look separate in 2D

## Evidence

- [other] per-subtype and merged-cluster ROGUE workflow definition: [evidence span withheld — anchor status: not_found]
- [other] validation values for Hydra and CAR-T ROGUE scores: [evidence span withheld — anchor status: not_found]
- [intro] motivation: whether a visual cluster should be subdivided: "it remains questionable whether a visual cluster in the 2D-embedding space is trustworthy or should be divided into more than one cluster"
- [methods] Seurat objects as the data container: "scDEED uses Seurat objects."
- [readme] Hydra dataset used in the scDEED package tutorial: "This input_counts contains 10000 randomly selected cells from the Hydra dataset."
