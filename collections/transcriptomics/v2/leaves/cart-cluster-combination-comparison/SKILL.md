---
name: cart-cluster-combination-comparison
description: Use when evaluating whether visually adjacent or suspicious clusters
  in a t-SNE/UMAP embedding (e.g. CAR-T clusters 3+4+9+11) should be merged, by comparing
  per-cluster ROGUE purity scores against the merged cluster's ROGUE score.
when_to_use_negative:
- When the embedding has not been optimized — cluster assignments should come from
  the scDEED-optimized t-SNE, not default-hyperparameter embeddings
- When only a single predefined cluster exists and no merge-vs-split comparison is
  being tested
tools:
- name: Rogue
  role: R package whose CalculateRogue function computes the cluster purity metric
    for each cluster and merged combination
- name: Seurat
  role: Reads and manipulates the Seurat objects holding the Hydra and CAR-T cluster
    labels
  repo: https://satijalab.org/seurat/
- name: R
  role: Analysis environment in which CalculateRogue is run on each cluster combination
- name: scDEED
  role: Provided the optimized t-SNE hyperparameters/embedding under which the CAR-T
    cluster assignments being compared were defined
  repo: github.com/JSB-UCLA/scDEED
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-45891-y
    title: Statistical method scDEED for detecting dubious 2D single-cell embeddings
      and optimizing t-SNE and UMAP hyperparameters
  source_content_hash: sha256:4ecd121fdd41b2857c58f39d1af45ac77c862a32ede0c3ebae99fc14bac0b595
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Rogue
  - Seurat
  - scDEED
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# cart-cluster-combination-comparison

## Summary

Recompute cluster purity with the ROGUE metric for individual CAR-T clusters (3, 4, 9, 11) versus their merged combination (3+4+9+11) under the scDEED-optimized t-SNE embedding, to test whether merging clusters meaningfully reduces purity.

## When to use

Use when evaluating whether visually adjacent or suspicious clusters in a t-SNE/UMAP embedding (e.g. CAR-T clusters 3+4+9+11) should be merged, by comparing per-cluster ROGUE purity scores against the merged cluster's ROGUE score.

## When NOT to use

- When the embedding has not been optimized — cluster assignments should come from the scDEED-optimized t-SNE, not default-hyperparameter embeddings
- When only a single predefined cluster exists and no merge-vs-split comparison is being tested

## Inputs

- Seurat object with cluster assignments for CAR-T cells under the scDEED-optimized t-SNE embedding
- Seurat object with cluster labels for the five Hydra neuron ectodermal subtypes
- Cluster membership labels (per-cell cluster IDs) in R

## Outputs

- Per-cluster ROGUE purity scores (e.g. 0.887/0.885/0.886 for CAR-T combinations; 0.744 for the merged 3+4+9+11 combination)
- Average and standard deviation of ROGUE across subtypes (Hydra: average 0.768, sd 0.055)
- rogue_scores.csv compiling all recomputed values

## How to apply

Load published Seurat cluster assignments (CAR-T clusters under the scDEED-optimized t-SNE, and optionally the five Hydra neuron ectodermal subtypes) into R using the Seurat package. Install the Rogue R package and run CalculateRogue separately on each candidate cluster to obtain per-cluster purity values. Run CalculateRogue on the merged cluster combination (e.g. clusters 3+4+9+11). Compare: the merged CAR-T combination scored 0.832 versus 0.887/0.885/0.886 for the individual combinations, and the unified Hydra ectodermal neuron cluster scored 0.744 versus per-subtype values of 0.710/0.784/0.714/0.793/0.839 (average 0.768, sd 0.055), indicating merging did not meaningfully reduce purity. Compile all values into rogue_scores.csv and check reproduction of these reported numbers as validation.

## Related tools

- **Rogue** (R package whose CalculateRogue function computes the cluster purity metric for each cluster and merged combination)
- **Seurat** (Reads and manipulates the Seurat objects holding the Hydra and CAR-T cluster labels) — https://satijalab.org/seurat/
- **R** (Analysis environment in which CalculateRogue is run on each cluster combination)
- **scDEED** (Provided the optimized t-SNE hyperparameters/embedding under which the CAR-T cluster assignments being compared were defined) — github.com/JSB-UCLA/scDEED

## Examples

```
devtools::install_github("JSB-UCLA/scDEED"); library(Rogue); rogue_per_cluster <- sapply(cluster_list, function(cl) CalculateRogue(seurat_obj, clusters = cl))
```

## Evaluation signals

- Recomputed Hydra per-subtype ROGUE values match 0.710/0.784/0.714/0.793/0.839 with average 0.768 and sd 0.055
- Unified Hydra ectodermal neuron cluster ROGUE reproduces 0.744
- CAR-T combination ROGUE values reproduce 0.887/0.885/0.886, and the merged 3+4+9+11 cluster reproduces 0.832
- All recomputed values are present in rogue_scores.csv with one row per cluster/combination

## Limitations

- A high ROGUE score for a merged cluster (e.g. 0.832 vs ~0.886) still shows some purity reduction, so the metric informs but does not definitively settle merge decisions
- Results depend on the cluster assignments being defined under the scDEED-optimized t-SNE; other embeddings may yield different combinations
- Seurat version differences (5.2.1 vs 5.1.0) were noted as affecting installation in the scDEED workflow

## Evidence

- [other] Hydra per-subtype ROGUE of 0.710/0.784/0.714/0.793/0.839 (average 0.768, sd 0.055) versus unified 0.744: [evidence span withheld — anchor status: not_found]
- [other] Run CalculateRogue on each of the five Hydra neuron ectodermal subtype clusters separately to obtain per-subtype ROGUE values.: [evidence span withheld — anchor status: not_found]
- [readme] scDEED uses Seurat objects. This input_counts contains 10000 randomly selected cells from the Hydra dataset.: "scDEED uses Seurat objects. This input_counts contains 10000 randomly selected cells from the Hydra dataset."
- [readme] You can install scDEED from GitHub using devtools.: "You can install scDEED from GitHub using devtools. It should install in 1 minute, but may take longer if you need to update dependencies."
