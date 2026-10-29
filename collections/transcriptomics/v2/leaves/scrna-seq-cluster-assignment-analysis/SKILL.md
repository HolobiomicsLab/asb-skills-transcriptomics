---
name: scrna-seq-cluster-assignment-analysis
description: Decision support for whether visually distinct scRNA-seq clusters in
  a scDEED-optimized t-SNE embedding should be unified into one cluster — e.g. five
  Hydra neuron ectodermal subtypes or CAR-T clusters 3/4/9/11.
when_to_use_negative:
- The input is raw count data without defined cluster labels — scDEED/Seurat preprocessing
  (NormalizeData, ScaleData, RunPCA, clustering) must be run first
- The question is whether the 2D embedding itself is dubious or hyperparameters need
  tuning — use the scDEED optimization skill, not ROGUE purity comparison
- Cluster identity is already validated by external biology (e.g. marker genes); ROGUE
  merging tests would not change the conclusion
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0102
tools:
- name: Rogue
  role: R package whose CalculateRogue function computes cluster purity for individual
    and merged cluster sets
- name: Seurat
  role: Holds the scRNA-seq data and cluster assignments consumed by CalculateRogue;
    scDEED also uses Seurat objects
  repo: https://satijalab.org/seurat/
- name: scDEED
  role: Defines the optimized t-SNE embedding under which the CAR-T cluster assignments
    being purity-tested were produced
  repo: github.com/JSB-UCLA/scDEED
- name: R
  role: Language in which Seurat and Rogue are run to recompute and compile ROGUE
    scores
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-45891-y
    title: Statistical method scDEED for detecting dubious 2D single-cell embeddings
      and optimizing t-SNE and UMAP hyperparameters
  source_content_hash: sha256:7b31d6bbcaae2002086c3fd586692c2a2d8b7dd5196b77c2d0e685fab3cfdb42
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Rogue
  - Seurat
  - scDEED
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# scrna-seq-cluster-assignment-analysis

## Summary

Quantitatively test whether single-cell clusters identified in a 2D embedding (t-SNE/UMAP) should be merged or kept separate by computing the ROGUE cluster-purity metric on each cluster and on merged combinations, using Seurat cluster labels as input.

## When to use

Decision support for whether visually distinct scRNA-seq clusters in a scDEED-optimized t-SNE embedding should be unified into one cluster — e.g. five Hydra neuron ectodermal subtypes or CAR-T clusters 3/4/9/11.

## When NOT to use

- The input is raw count data without defined cluster labels — scDEED/Seurat preprocessing (NormalizeData, ScaleData, RunPCA, clustering) must be run first
- The question is whether the 2D embedding itself is dubious or hyperparameters need tuning — use the scDEED optimization skill, not ROGUE purity comparison
- Cluster identity is already validated by external biology (e.g. marker genes); ROGUE merging tests would not change the conclusion

## Inputs

- Seurat object with cluster assignments (e.g. Hydra dataset neuron ectodermal subtypes; CAR-T clusters under scDEED-optimized t-SNE)
- Pre-embedding PCA-reduced data within the Seurat object
- Cluster identity labels (e.g. seurat_clusters metadata column)

## Outputs

- Per-cluster ROGUE purity values (e.g. 0.710/0.784/0.714/0.793/0.839 for the five Hydra subtypes)
- Average and standard deviation of per-cluster ROGUE values (e.g. 0.768, sd 0.055)
- ROGUE value for the merged/unified cluster (e.g. 0.744 unified Hydra ectodermal neurons; 0.832 for CAR-T 3+4+9+11)
- rogue_scores.csv compiling all recomputed values

## How to apply

Load the published Seurat object cluster labels (e.g. the five Hydra neuron ectodermal subtypes, or CAR-T clusters defined under the scDEED-optimized t-SNE). Install the R Rogue package and run CalculateRogue separately on each individual cluster to get per-cluster purity values, then run it on the merged/unified cluster set. Compute the average and standard deviation of per-cluster ROGUE values to compare against the unified cluster's value. Compile all recomputed scores (e.g. into rogue_scores.csv). Interpretation: if the unified cluster's ROGUE (e.g. 0.744 for the Hydra ectodermal neuron set) is close to the per-subtype average (0.768, sd 0.055), unification does not meaningfully reduce purity; if merged-cluster ROGUE drops substantially (e.g. 0.832 for CAR-T clusters 3+4+9+11 vs 0.885–0.887 for the alternatives), merging is not supported.

## Related tools

- **Rogue** (R package whose CalculateRogue function computes cluster purity for individual and merged cluster sets)
- **Seurat** (Holds the scRNA-seq data and cluster assignments consumed by CalculateRogue; scDEED also uses Seurat objects) — https://satijalab.org/seurat/
- **scDEED** (Defines the optimized t-SNE embedding under which the CAR-T cluster assignments being purity-tested were produced) — github.com/JSB-UCLA/scDEED
- **R** (Language in which Seurat and Rogue are run to recompute and compile ROGUE scores)

## Examples

```
devtools::install_github("JSB-UCLA/scDEED")
```

## Evaluation signals

- Recomputed Hydra per-subtype ROGUE values match the reported 0.710/0.784/0.714/0.793/0.839 with average 0.768 and sd 0.055
- Unified Hydra ectodermal neuron ROGUE equals 0.744, within ~0.024 of the per-subtype average, supporting the 'no meaningful purity loss' conclusion
- CAR-T combination ROGUE values reproduce 0.887/0.885/0.886, with the merged clusters 3+4+9+11 giving a lower 0.832
- rogue_scores.csv contains one row per tested cluster/combination with values matching the reported figures
- Identical Seurat cluster labels (not re-clustered data) are passed to CalculateRogue so results are directly comparable to published values

## Limitations

- A near-equal unified ROGUE (0.744 vs average 0.768) shows that visual subclusters in the t-SNE plot may not correspond to meaningful purity differences — unification of the five Hydra ectodermal neuron subtypes was not supported by a purity drop
- scDEED/Seurat installation is sensitive to Seurat version (5.2.1 vs 5.1.0 differences were reported affecting installation)
- Differences between cluster interpretations may reflect randomness rather than biological signals under optimized hyperparameters, so purity scores should not be over-interpreted

## Evidence

- [intro] It remains questionable whether a visual cluster in the 2D-embedding space is trustworthy or should be divided into more than one cluster: "it remains questionable whether a visual cluster in the 2D-embedding space is trustworthy or should be divided into more than one cluster"
- [methods] scDEED uses Seurat objects, which provide the cluster labels input to ROGUE purity computation: "scDEED uses Seurat objects. This input_counts contains 10000 randomly selected cells from the Hydra dataset."
- [abstract] 2D embeddings whose clusters are assessed for merge/split decisions: "t-distributed stochastic neighbor embedding (t-SNE) and uniform manifold approximation and projection (UMAP) are commonly used for visualizing cell clusters"
- [readme] A cell embedding is considered dubious if neighbors before and after the embedding differ: "A cell embedding is considered 'dubious' if the cell's neighbors before and after the embedding differ drastically."
- [readme] Hyperparameter-optimized embeddings underlie trustworthy cluster interpretation: "An intuitive way to select optimal hyperparameters is by minimizing the number of dubious cell embeddings"
