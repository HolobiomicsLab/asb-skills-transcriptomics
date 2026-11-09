---
name: cross-method-trajectory-comparison
description: Use when a simulated dataset with known pseudotime and branching structure
  is available and you need to quantify which TI method assigns cells to branches
  most purely and reconstructs pseudotime most accurately.
when_to_use_negative:
- The dataset lacks simulated ground-truth pseudotime or branching labels, so neither
  Kendall correlation nor assignment purity can be computed.
- The biological system is a single unbranched trajectory, making branch-purity entropy
  and branching-aware comparison moot.
- No RNA velocity can be estimated (e.g., no spliced/unspliced counts), since CellPath
  and VeTra depend on velocity input and the comparison would be confounded.
edam_operation: http://edamontology.org/operation_2478
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CellPath
  role: Primary TI method under comparison; infers high-resolution trajectories and
    pseudotime from gene expression plus RNA velocity
  repo: https://github.com/PeterZZQ/CellPaths.git
- name: VeTra
  role: Competing RNA velocity-based TI method providing alternative branch assignments
    (clusternumber = 4)
- name: Slingshot
  role: Competing expression-only TI method requiring a user-specified root cluster
- name: scVelo
  role: Upstream RNA velocity calculation supplying the velocity matrix consumed by
    CellPath
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100095
    title: Inference of high-resolution trajectories in single-cell RNA-seq data by
      using RNA velocity
  source_content_hash: sha256:1c9cd90cc47ad3073ed71d77e83a46deb19551292e420af7a47c97cde9175f50
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PeterZZQ/CellPath
  repo_url: https://github.com/PeterZZQ/CellPaths.git
  tools:
  - CellPath
  - Slingshot
  - VeTra
  - scVelo
  edam_operation: http://edamontology.org/operation_2478
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cross-method-trajectory-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark trajectory inference methods (CellPath, VeTra, Slingshot) on a simulated branching dataset by comparing per-path Kendall correlations against ground-truth pseudotime and the normalized average entropy score of cell-to-trajectory assignments after the branching point.

## When to use

Use when a simulated dataset with known pseudotime and branching structure is available and you need to quantify which TI method assigns cells to branches most purely and reconstructs pseudotime most accurately.

## When NOT to use

- The dataset lacks simulated ground-truth pseudotime or branching labels, so neither Kendall correlation nor assignment purity can be computed.
- The biological system is a single unbranched trajectory, making branch-purity entropy and branching-aware comparison moot.
- No RNA velocity can be estimated (e.g., no spliced/unspliced counts), since CellPath and VeTra depend on velocity input and the comparison would be confounded.

## Inputs

- Simulated scRNA-seq count matrix with RNA velocity (scVelo-annotated adata, .h5ad) from a dyngen bifurcating simulation
- Ground-truth simulated pseudotime and branching labels per cell
- Root cluster identity for Slingshot

## Outputs

- Per-path Kendall rank correlation values for each method (CellPath, VeTra, Slingshot)
- Normalized average entropy scores of cell-to-trajectory assignments after the branching point for each method
- Benchmark metrics table (bifurcating_benchmark_metrics.csv)

## How to apply

Generate a dyngen bifurcating simulation (e.g., 4 runs, 1,587 cells, 99 genes) retaining ground-truth pseudotime and branching structure. Run CellPath by initializing cp.CellPath with the velocity-annotated adata and calling all_in_one with num_metacells sized for the data, n_neighs = 10, pruning = False, insertion = True, prop_insert = 0.50, then read assignments and pseudotime from cellpath_obj.pseudo_order. Run VeTra with clusternumber = 4 and Slingshot with a user-specified root cluster on the same data. For each method, compute per-path Kendall rank correlations between inferred pseudotime and the simulated ground truth, and compute the normalized average entropy score of cell-to-trajectory assignments after the branching point as a purity measure — higher entropy indicates purer assignment of cells from the same true trajectory.

## Related tools

- **CellPath** (Primary TI method under comparison; infers high-resolution trajectories and pseudotime from gene expression plus RNA velocity) — https://github.com/PeterZZQ/CellPaths.git
- **VeTra** (Competing RNA velocity-based TI method providing alternative branch assignments (clusternumber = 4))
- **Slingshot** (Competing expression-only TI method requiring a user-specified root cluster)
- **scVelo** (Upstream RNA velocity calculation supplying the velocity matrix consumed by CellPath)

## Examples

```
cellpath_obj = cp.CellPath(adata = adata, preprocess = True); cellpath_obj.all_in_one(num_metacells = num_metacells, n_neighs = 10, pruning = False, insertion = True, prop_insert = 0.50)
```

## Evaluation signals

- CellPath attains the highest normalized average entropy score among CellPath, VeTra, and Slingshot, indicating the purest cell-to-trajectory assignment after the branching point.
- CellPath and Slingshot obtain comparable and higher Kendall correlation scores than VeTra on the dyngen branching dataset.
- All Kendall correlations and entropy values are recorded in bifurcating_benchmark_metrics.csv with one entry per method and path.
- CellPath output (cellpath_obj.pseudo_order) contains both pseudotime and branching assignments for every cell passed in the adata.

## Limitations

- Performance of all RNA velocity-based methods is limited by the accuracy of RNA velocity inference given its noise.
- Slingshot requires a user-specified root cluster, introducing a prior not needed by CellPath or VeTra.
- VeTra's use of 2D projections can cause significant information loss for gene expression and RNA velocity data, which affects its benchmark standing.

## Evidence

- [other] Run CellPath on the simulated data by initializing cp.CellPath with the velocity-annotated adata and calling all_in_one with num_metacells: "cellpath_obj.all_in_one(num_metacells = num_metacells, n_neighs = 10, pruning = False, num_trajs = num_trajs, insertion = True, prop_insert = 0.50)"
- [other] Slingshot requires a user-specified root cluster: [evidence span withheld — anchor status: not_found]
- [readme] Pseudotime and branching assignment result stored in pseudo_order: "Pseudo-time and branching assignment result
``ncellpath_obj.pseudo_order"
- [abstract] CellPath integrates gene expression and RNA velocity: "We present CellPath, a method that infers cell trajectories by integrating single-cell gene expression and RNA velocity information."
- [discussion] CellPath compared against velocity-based TI methods including VeTra: "We have conducted extensive tests to compare CellPath with other RNA velocity-based methods for TI, including VeTra, CellRank, and Directed-PAGA."
