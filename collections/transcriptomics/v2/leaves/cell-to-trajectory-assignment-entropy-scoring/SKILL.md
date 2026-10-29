---
name: cell-to-trajectory-assignment-entropy-scoring
description: Use when benchmarking trajectory inference methods on a branching (bifurcating)
  dataset with close, fine lineages, where purity of cell-to-trajectory assignments
  and per-path pseudotime accuracy must be quantified.
when_to_use_negative:
- The dataset has no branching structure (e.g., linear or cycle trajectories) — entropy
  scoring after the branching point is undefined or uninformative.
- No ground-truth trajectory labels are available, since purity of cell-to-trajectory
  assignments cannot be judged without the simulated truth.
- You need cluster-level trajectories only (e.g., Directed-PAGA output); entropy scoring
  is designed for cell-level assignment purity on fine, close lineages.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellPath
  role: Method under evaluation; provides cell-level trajectory and pseudotime assignments
    via all_in_one and cellpath_obj.pseudo_order on scVelo-annotated adata
  repo: https://github.com/PeterZZQ/CellPaths.git
- name: scVelo
  role: Upstream RNA velocity inference; computes the velocity matrix stored in adata
    before CellPath initialization
- name: VeTra
  role: Competing trajectory inference method run with clusternumber = 4 to obtain
    trajectory assignments for the entropy comparison
- name: Slingshot
  role: Competing trajectory inference method run with a user-specified root cluster
    on the same simulated dataset
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100095
    title: Inference of high-resolution trajectories in single-cell RNA-seq data by
      using RNA velocity
  source_content_hash: sha256:a2971bf69a08cccd7a96c4ebcae903aded80e9761a40c6b4933e81b66528bee3
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# cell-to-trajectory-assignment-entropy-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the normalized average entropy score of cell-to-trajectory assignments after the branching point to quantify how purely a trajectory inference method (CellPath, VeTra, or Slingshot) assigns cells from the same true trajectory to the same inferred path. Used together with per-path Kendall rank correlations of pseudotime to benchmark methods on a dyngen bifurcating dataset.

## When to use

Use when benchmarking trajectory inference methods on a branching (bifurcating) dataset with close, fine lineages, where purity of cell-to-trajectory assignments and per-path pseudotime accuracy must be quantified.

## When NOT to use

- The dataset has no branching structure (e.g., linear or cycle trajectories) — entropy scoring after the branching point is undefined or uninformative.
- No ground-truth trajectory labels are available, since purity of cell-to-trajectory assignments cannot be judged without the simulated truth.
- You need cluster-level trajectories only (e.g., Directed-PAGA output); entropy scoring is designed for cell-level assignment purity on fine, close lineages.

## Inputs

- dyngen bifurcating simulated dataset (4 simulation runs, 1,587 cells, 99 genes) with ground-truth pseudotime and branching structure, as velocity-annotated adata
- trajectory/pseudotime outputs from CellPath (cellpath_obj.pseudo_order), VeTra (clusternumber = 4), and Slingshot (root cluster specified)

## Outputs

- normalized average entropy score per method (Figure 7G comparison)
- per-path Kendall rank correlations between inferred pseudotime and simulated ground truth (Figure 7F comparison)
- bifurcating_benchmark_metrics.csv recording all Kendall correlations and entropy values

## How to apply

Run each competing method on the same simulated dataset: CellPath via cp.CellPath(adata) on velocity-annotated adata followed by all_in_one with num_metacells set for dataset size, n_neighs = 10, pruning = False, insertion = True, prop_insert = 0.50, extracting pseudotime and branching assignments from cellpath_obj.pseudo_order; VeTra with clusternumber = 4; Slingshot with a user-specified root cluster. For each method, compute per-path Kendall rank correlations between inferred pseudotime and simulated ground truth, then compute the normalized average entropy score of cell-to-trajectory assignments after the branching point. The method whose inferred paths assign cells from the same true trajectory most purely attains the highest normalized average entropy score. Record all Kendall correlations and entropy values in bifurcating_benchmark_metrics.csv for validation.

## Related tools

- **CellPath** (Method under evaluation; provides cell-level trajectory and pseudotime assignments via all_in_one and cellpath_obj.pseudo_order on scVelo-annotated adata) — https://github.com/PeterZZQ/CellPaths.git
- **scVelo** (Upstream RNA velocity inference; computes the velocity matrix stored in adata before CellPath initialization)
- **VeTra** (Competing trajectory inference method run with clusternumber = 4 to obtain trajectory assignments for the entropy comparison)
- **Slingshot** (Competing trajectory inference method run with a user-specified root cluster on the same simulated dataset)

## Examples

```
cellpath_obj = cp.CellPath(adata = adata, preprocess = True); cellpath_obj.all_in_one(num_metacells = num_metacells, n_neighs = 10, pruning = False, insertion = True, prop_insert = 0.50)
```

## Evaluation signals

- CellPath attains the highest normalized average entropy score among CellPath, VeTra, and Slingshot on the dyngen branching dataset, indicating the purest cell-to-trajectory assignments.
- CellPath and Slingshot obtain comparable and higher Kendall correlation scores than VeTra (Figure 7F).
- All Kendall correlations and entropy values are recorded in bifurcating_benchmark_metrics.csv.
- Dataset invariants hold: 4 simulation runs producing 1,587 cells and 99 genes with retained ground-truth pseudotime and branching structure.
- Pseudotime and branching assignments are extracted from cellpath_obj.pseudo_order after all_in_one completes.

## Limitations

- Performance of all RNA velocity-based methods (including CellPath) is limited by the accuracy of RNA velocity inference given noise in estimated velocity.
- Slingshot requires a user-specified root cluster, introducing a parameter the simulated-benchmark must supply externally.
- The benchmark relies on simulated ground truth (dyngen); entropy-based purity conclusions may not transfer directly to real datasets without ground-truth trajectory labels.

## Evidence

- [other] Compute the normalized average entropy score of cell-to-trajectory assignments after the branching point for all three methods (Figure 7G comparison).: [evidence span withheld — anchor status: not_found]
- [abstract] We evaluate CellPath on both real and simulated datasets and show that CellPath finds more accurate and detailed trajectories than the state-of-the-art TI methods using or not using RNA velocity: "We evaluate CellPath on both real and simulated datasets and show that CellPath finds more accurate and detailed trajectories than the state-of-the-art TI methods using or not using RNA velocity"
- [readme] cellpath_obj.all_in_one(num_metacells = num_metacells, n_neighs = 10, pruning = False, num_trajs = num_trajs, insertion = True, prop_insert = 0.50): "cellpath_obj.all_in_one(num_metacells = num_metacells, n_neighs = 10, pruning = False, num_trajs = num_trajs, insertion = True, prop_insert = 0.50)"
- [methods] Pseudo-time and branching assignment result cellpath_obj.pseudo_order: "`Pseudo-time and branching assignment result"
