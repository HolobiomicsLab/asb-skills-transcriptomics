---
name: clustering-diversity-filtering-with-ari
description: Use when combining multiple base clustering solutions (e.g. nine scRNA-seq
  algorithms on feature-selected scRNA-seq data) and redundant, highly similar label
  sets must be removed before meta-clustering.
when_to_use_negative:
- Only a single clustering solution is available, so no pairwise diversity can be
  assessed
- Base clusterings are already known to be complementary and the ensemble size is
  small enough that pruning would discard informative labels
- The downstream combiner can directly handle redundancy (e.g. weighting schemes that
  already down-weight correlated clusterings)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: scEFSC
  role: Implements the diversity-based label filtering step between base clustering
    and wMetaC meta-clustering
  repo: https://github.com/Conan-Bian/scEFSC
- name: SC3
  role: One of the nine base clustering algorithms whose labels are compared by pairwise
    ARI and filtered by diversity
  repo: github.com/hemberg-lab/SC3
- name: CIDR
  role: Base clustering algorithm contributing label vectors to the ARI diversity
    filter
  repo: github.com/VCCRI/CIDR
- name: Seurat
  role: Base clustering algorithm (reduction dimension 10) whose labels enter the
    pairwise ARI comparison
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.csbj.2022.04.023
    title: 'scEFSC: Accurate single-cell RNA-seq data analysis via ensemble consensus
      clustering based on multiple feature selections'
  source_content_hash: sha256:a190fa2c5d43d914ef239d0c941073ed9009de559d34574905746945267ef4b3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/Conan-Bian/scEFSC
  repo_url: https://github.com/Conan-Bian/scEFSC
  tools:
  - CIDR
  - Monocle
  - RaceID
  - Rphenograph
  - SC3
  - SHARP
  - SINCERA
  - Seurat
  - pcaReduce
  - scEFSC
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# clustering-diversity-filtering-with-ari

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A pruning step in the scEFSC ensemble that computes pairwise Adjusted Rand Index (ARI) between all base clustering labels and removes labels whose similarity vector has the lowest variance, i.e. the least diverse clusterings, before weighted meta-clustering.

## When to use

Use when combining multiple base clustering solutions (e.g. nine scRNA-seq algorithms on feature-selected scRNA-seq data) and redundant, highly similar label sets must be removed before meta-clustering.

## When NOT to use

- Only a single clustering solution is available, so no pairwise diversity can be assessed
- Base clusterings are already known to be complementary and the ensemble size is small enough that pruning would discard informative labels
- The downstream combiner can directly handle redundancy (e.g. weighting schemes that already down-weight correlated clusterings)

## Inputs

- Per-cell cluster label vectors from multiple base clustering algorithms (e.g. 4 feature subsets × 9 scRNA-seq clustering algorithms in scEFSC)
- Pairwise ARI similarity matrix between all clustering labels

## Outputs

- Filtered set of clustering labels (labels with lowest-variance ARI similarity vectors removed)
- Retained label set ready for wMetaC weighted meta-clustering

## How to apply

After each of the derived feature subsets has been clustered (in scEFSC: four feature subsets from Low Variance, Laplacian Score, SPEC, and MCFS each clustered by nine algorithms such as SC3, CIDR, monocle, pcaReduce, Rphenograph, Seurat, SHARP, SINCERA, and RaceID using the known number of cell types as k), compute the pairwise ARI between all individual clustering label vectors. For each label, assemble its similarity vector (its ARIs to all other labels) and remove the labels whose similarity vector has the lowest variance, which identifies the clusterings contributing the least diversity to the ensemble. Retain the remaining labels as input to the wMetaC weighted meta-clustering step, where the clustering similarity matrix is converted to colocation weights w(i,j)=s(i,j)(1−s(i,j)). This filtering prevents redundant, near-identical clusterings from dominating the consensus.

## Related tools

- **scEFSC** (Implements the diversity-based label filtering step between base clustering and wMetaC meta-clustering) — https://github.com/Conan-Bian/scEFSC
- **SC3** (One of the nine base clustering algorithms whose labels are compared by pairwise ARI and filtered by diversity) — github.com/hemberg-lab/SC3
- **CIDR** (Base clustering algorithm contributing label vectors to the ARI diversity filter) — github.com/VCCRI/CIDR
- **Seurat** (Base clustering algorithm (reduction dimension 10) whose labels enter the pairwise ARI comparison)

## Examples

```
library(mclust); sim <- sapply(labels_list, function(l) sapply(labels_list, function(m) adjustedRandIndex(l, m))); keep <- which(apply(sim, 1, var) > min(apply(sim, 1, var)))
```

## Evaluation signals

- Pairwise ARI matrix is computed over all individual clustering labels with values in [-1, 1], with 1 indicating identical partitions
- Removed labels are exactly those whose ARI similarity vectors have the lowest variance (least diverse clusterings)
- The retained label set remains non-empty and feeds into the wMetaC similarity matrix with weights w(i,j)=s(i,j)(1−s(i,j))
- Final ensemble labels achieve non-degenerate agreement with the dataset's published cell-type labels when scored by NMI and ARI
- Ensemble with filtering should not degrade NMI/ARI relative to keeping all labels on benchmark datasets such as GSE81608 or GSE85241

## Limitations

- Requires the known number of cell types as k for the base clustering algorithms, so it is not directly applicable to unknown-k settings
- Filtering by lowest similarity-vector variance assumes low-variance labels are redundant rather than stable-and-correct; a highly consistent good clustering could in principle be pruned
- Evaluated only on fourteen real scRNA-seq datasets; performance on other data modalities is untested

## Evidence

- [full_text] Workflow context of pairwise ARI: [evidence span withheld — anchor status: not_found]
- [readme] Weighted combination after filtering: [evidence span withheld — anchor status: not_found]
