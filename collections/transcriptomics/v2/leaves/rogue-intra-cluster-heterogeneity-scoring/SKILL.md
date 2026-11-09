---
name: rogue-intra-cluster-heterogeneity-scoring
description: Use when deciding whether visually distinct single-cell clusters should
  be unified or kept separate — compute per-cluster and merged-cluster ROGUE scores
  and compare them (e.g., Hydra neuron ectodermal subtypes vs. their unified cluster).
when_to_use_negative:
- Input clusters have not been defined on an embedding whose hyperparameters were
  validated (e.g., not the scDEED-optimized t-SNE for the CAR-T data) — cluster boundaries
  themselves may be dubious
- The question concerns 2D embedding reliability or hyperparameter optimization rather
  than cluster purity — use scDEED's reliability scores instead of ROGUE
- Input is raw unclustered expression data with no cell-to-cluster assignment available
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Rogue
  role: R package whose CalculateRogue function computes intra-cluster purity for
    each cluster and each merged cluster combination
- name: Seurat
  role: Reads and holds the cluster labels as Seurat objects that Rogue consumes;
    scDEED also uses Seurat objects for the optimized t-SNE
  repo: https://satijalab.org/seurat/
- name: scDEED
  role: Produced the optimized t-SNE under which the CAR-T cluster assignments being
    purity-tested were defined
  repo: github.com/JSB-UCLA/scDEED
- name: R
  role: Analysis environment in which Rogue, Seurat, and scDEED are run and rogue_scores.csv
    is compiled
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-45891-y
    title: Statistical method scDEED for detecting dubious 2D single-cell embeddings
      and optimizing t-SNE and UMAP hyperparameters
  source_content_hash: sha256:792a80faf37fffa4924bccb84a2c1f1716b5476a13af3d199754899fa198b106
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# rogue-intra-cluster-heterogeneity-scoring

## Summary

Quantifies intra-cluster purity of single-cell transcriptomic clusters with the Rogue R package's CalculateRogue, to test whether merging visual clusters (e.g., five Hydra neuron ectodermal subtypes, or CAR-T clusters 3+4+9+11 under the scDEED-optimized t-SNE) meaningfully reduces purity.

## When to use

Use when deciding whether visually distinct single-cell clusters should be unified or kept separate — compute per-cluster and merged-cluster ROGUE scores and compare them (e.g., Hydra neuron ectodermal subtypes vs. their unified cluster).

## When NOT to use

- Input clusters have not been defined on an embedding whose hyperparameters were validated (e.g., not the scDEED-optimized t-SNE for the CAR-T data) — cluster boundaries themselves may be dubious
- The question concerns 2D embedding reliability or hyperparameter optimization rather than cluster purity — use scDEED's reliability scores instead of ROGUE
- Input is raw unclustered expression data with no cell-to-cluster assignment available

## Inputs

- Seurat object with published cluster labels (e.g., five Hydra neuron ectodermal subtypes; CAR-T clusters defined under the scDEED-optimized t-SNE)
- Gene expression count matrix backing the cluster assignments
- Definition of merged/combined cluster groupings to test (e.g., clusters 3+4+9+11)

## Outputs

- Per-cluster ROGUE purity scores (e.g., 0.710/0.784/0.714/0.793/0.839 for Hydra subtypes)
- Merged-cluster ROGUE score (e.g., 0.744 unified; 0.832 for CAR-T 3+4+9+11)
- Average and standard deviation of per-cluster ROGUE values (0.768, sd 0.055)
- rogue_scores.csv compiling all recomputed values

## How to apply

Load the published cluster assignments into R as Seurat objects (scDEED uses Seurat objects, so the same object structure carries the t-SNE-optimized labels). Install the Rogue package and run CalculateRogue separately on each subtype cluster (the five Hydra neuron ectodermal subtypes yielded 0.710/0.784/0.714/0.793/0.839), then run it on the unified cell set to get the merged score (0.744). Compute the mean and standard deviation of the per-cluster scores (0.768, sd 0.055) to judge whether the merged value represents a meaningful purity loss — here a 0.024 drop was judged not meaningful. Repeat for alternative cluster partitions, such as CAR-T clusters under the scDEED-optimized t-SNE (0.887/0.885/0.886 for separate combinations vs. 0.832 for merged clusters 3+4+9+11), and compile all recomputed values into rogue_scores.csv.

## Related tools

- **Rogue** (R package whose CalculateRogue function computes intra-cluster purity for each cluster and each merged cluster combination)
- **Seurat** (Reads and holds the cluster labels as Seurat objects that Rogue consumes; scDEED also uses Seurat objects for the optimized t-SNE) — https://satijalab.org/seurat/
- **scDEED** (Produced the optimized t-SNE under which the CAR-T cluster assignments being purity-tested were defined) — github.com/JSB-UCLA/scDEED
- **R** (Analysis environment in which Rogue, Seurat, and scDEED are run and rogue_scores.csv is compiled)

## Examples

```
library(Rogue); rogue_score <- CalculateRogue(seurat_obj, ident = "seurat_clusters")
```

## Evaluation signals

- Hydra per-subtype ROGUE reproduces 0.710/0.784/0.714/0.793/0.839 with average 0.768 and standard deviation 0.055
- Unified Hydra ectodermal neuron set yields ROGUE ≈ 0.744 (drop of ~0.024 from the 0.768 average, i.e., no meaningful purity reduction)
- CAR-T cluster combinations under the optimized t-SNE reproduce 0.887/0.885/0.886, versus 0.832 for the merged clusters 3+4+9+11
- All recomputed scores are present in rogue_scores.csv with one row per cluster combination

## Limitations

- A merged cluster's purity drop may be small relative to the per-cluster standard deviation (0.744 vs. 0.768 ± 0.055), so ROGUE alone does not prove subtypes are biologically identical
- Cluster labels depend on the embedding used; dubious 2D embeddings can make cluster boundaries unreliable, and the scDEED paper notes Seurat version differences (5.2.1 vs 5.1.0) can affect installation/reproducibility
- Caution is needed when interpreting visual clusters if the number of dubious embeddings shows a complex trend with hyperparameter value — cluster topology may be complex

## Evidence

- [intro] motivation for testing whether visual clusters should be merged: "it remains questionable whether a visual cluster in the 2D-embedding space is trustworthy or should be divided into more than one cluster"
- [full_text] Hydra ROGUE benchmark values: [evidence span withheld — anchor status: not_found]
- [methods] Seurat object input requirement shared with scDEED pipelines: "scDEED uses Seurat objects."
- [full_text] CAR-T merged-cluster comparison values: [evidence span withheld — anchor status: not_found]
