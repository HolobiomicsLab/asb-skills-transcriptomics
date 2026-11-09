---
name: pseudotime-kendall-correlation-evaluation
description: Use when evaluating inferred cell orderings on simulated single-cell
  RNA-seq data with known ground-truth pseudotime and branching structure, to rank
  TI methods (CellPath, VeTra, Slingshot, CellRank, Vdpt) by per-path Kendall correlation
  and assignment entropy.
when_to_use_negative:
- The dataset has no ground-truth pseudotime or branching annotation (e.g., purely
  real data with no simulation-based reference), so Kendall correlation against truth
  cannot be computed
- The method under comparison does not output cell-level pseudotime or trajectory
  assignments (e.g., dynamo, which 'can estimate RNA velocity and predict cell fates,
  but it does not extract cell trajectories')
- The goal is a single global ordering; per-path Kendall correlation assumes a method
  outputs multiple paths whose cell assignments must be matched to ground-truth branches
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellPath
  role: Primary TI method under evaluation; outputs per-trajectory pseudotime and
    branching assignments via `pseudo_order`
  repo: https://github.com/PeterZZQ/CellPaths.git
- name: VeTra
  role: Velocity-based baseline run with clusternumber=4 (min_dist averaged over 0.4,
    0.5, 0.8) to provide competing trajectory assignments
- name: Slingshot
  role: Non-velocity baseline requiring a user-specified root cluster, providing pseudotime
    for Kendall comparison
- name: CellRank
  role: Velocity-based baseline combined with DPT to produce baseline pseudotime for
    Kendall correlation
- name: scVelo
  role: Computes RNA velocity stored in the adata input to CellPath and other velocity-based
    baselines
provenance:
  source_task_ids:
  - task_006
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100095
    title: Inference of high-resolution trajectories in single-cell RNA-seq data by
      using RNA velocity
  source_content_hash: sha256:a66a2f5481c7fc9903f77fa3a49229a4beecee184351dac3ed955fe9b899ce28
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
  - CellRank
  - Slingshot
  - VeTra
  - scVelo
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# pseudotime-kendall-correlation-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark trajectory-inference pseudotime accuracy by computing per-path Kendall rank correlations against simulated ground truth (e.g., VeloSim cycle-tree and dyngen bifurcating datasets), and assess cell-to-trajectory assignment purity via normalized average entropy scores, as done for CellPath versus CellRank, VeTra, Vdpt, and Slingshot.

## When to use

Use when evaluating inferred cell orderings on simulated single-cell RNA-seq data with known ground-truth pseudotime and branching structure, to rank TI methods (CellPath, VeTra, Slingshot, CellRank, Vdpt) by per-path Kendall correlation and assignment entropy.

## When NOT to use

- The dataset has no ground-truth pseudotime or branching annotation (e.g., purely real data with no simulation-based reference), so Kendall correlation against truth cannot be computed
- The method under comparison does not output cell-level pseudotime or trajectory assignments (e.g., dynamo, which 'can estimate RNA velocity and predict cell fates, but it does not extract cell trajectories')
- The goal is a single global ordering; per-path Kendall correlation assumes a method outputs multiple paths whose cell assignments must be matched to ground-truth branches

## Inputs

- Simulated scRNA-seq count matrix with RNA velocity (VeloSim cycle-tree: 785 cells, 600 genes; dyngen bifurcating: 1,587 cells, 99 genes) in AnnData (.h5ad) with ground-truth pseudotime and branching labels
- Inferred per-path pseudotime vectors and cell-to-trajectory assignments from each TI method (e.g., CellPath `pseudo_order` output)
- Root/root-cluster labels for methods requiring them (Slingshot)

## Outputs

- Per-method, per-path Kendall rank correlation table (e.g., kendall_scores.csv, bifurcating_benchmark_metrics.csv)
- Normalized average entropy scores of cell-to-trajectory assignments after branching (Figure 7G-style comparison)

## How to apply

Run each trajectory-inference method on the same simulated dataset (e.g., CellPath via `all_in_one(num_metacells=..., n_neighs=10, pruning=False, insertion=True, prop_insert=0.50)` on velocity-annotated adata, VeTra with clusternumber=4 averaged over min_dist 0.4/0.5/0.8, Slingshot with a user-specified root cluster). Extract per-trajectory pseudotime from each method's output (e.g., `cellpath_obj.pseudo_order` for CellPath) and compute the Kendall rank correlation between each inferred path's pseudotime and the simulated ground truth for every path in every replicate. For datasets with branching (dyngen bifurcating, 4 runs, 1,587 cells, 99 genes), additionally compute the normalized average entropy score of cell-to-trajectory assignments after the branching point to quantify purity of branch assignment. Aggregate per-method, per-path scores across replicates (e.g., 10 VeloSim cycle-tree replicates of 785 cells, 600 genes, distinct seeds) into a CSV table and compare patterns against the reported figures (Figure 5G, Figure 7F/7G).

## Related tools

- **CellPath** (Primary TI method under evaluation; outputs per-trajectory pseudotime and branching assignments via `pseudo_order`) — https://github.com/PeterZZQ/CellPaths.git
- **VeTra** (Velocity-based baseline run with clusternumber=4 (min_dist averaged over 0.4, 0.5, 0.8) to provide competing trajectory assignments)
- **Slingshot** (Non-velocity baseline requiring a user-specified root cluster, providing pseudotime for Kendall comparison)
- **CellRank** (Velocity-based baseline combined with DPT to produce baseline pseudotime for Kendall correlation)
- **scVelo** (Computes RNA velocity stored in the adata input to CellPath and other velocity-based baselines)

## Examples

```
cellpath_obj = cp.CellPath(adata=adata, preprocess=True); cellpath_obj.all_in_one(num_metacells=num_metacells, n_neighs=10, pruning=False, insertion=True, prop_insert=0.50); pseudo = cellpath_obj.pseudo_order
```

## Evaluation signals

- Per-path Kendall rank correlations match the reported pattern: CellPath achieves the highest correlation among compared methods on cycle-tree replicates (Figure 5G pattern)
- On the dyngen bifurcating dataset, CellPath attains the highest normalized average entropy score among CellPath, VeTra, and Slingshot, with CellPath and Slingshot obtaining comparable, higher Kendall correlations than VeTra (Figure 7F/7G pattern)
- Every path in every replicate has a recorded Kendall score; results table written per-method, per-path with no missing entries
- Simulated replicates use distinct random seeds (10 cycle-tree datasets, 4 dyngen runs) so per-path scores are non-degenerate and reproducible

## Limitations

- Given the noise in the estimated RNA velocity, the performance of all RNA velocity-based methods are affected by accuracy in RNA velocity inference
- VeTra's use of 2D projections can cause significant information loss, which affects its baseline scores
- Slingshot requires a user-specified root cluster, introducing a dependency on correct root annotation
- Score comparisons depend on matching inferred paths to ground-truth branches, which can be ambiguous for methods detecting local rather than global paths

## Evidence
