---
name: single-cell-trajectory-simulation
description: Use when benchmarking trajectory inference or pseudotime methods (e.g.,
  CellPath, Slingshot, VeTra, CellRank) against a known ground truth that real scRNA-seq
  data cannot provide.
when_to_use_negative:
- Evaluating methods on real biological datasets where no ground-truth pseudotime
  exists — use qualitative trajectory agreement or known biology instead
- Benchmarking clustering-only or fate-probability tools that do not output pseudotime
  or per-path trajectories comparable to the simulated ground truth
- The simulated topology does not match the method's assumptions (e.g., methods restricted
  to one connected trajectory on multi-path tree data), confounding fairness of comparison
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: VeloSim
  role: Simulates binary-tree single-cell datasets (1,999 cells, 500 genes) with ground-truth
    pseudotime for benchmarking
- name: dyngen
  role: Simulates binary-tree (2,000 cells) and bifurcating (1,587 cells, 99 genes)
    datasets with ground-truth pseudotime and branching structure
- name: CellPath
  role: TI method under evaluation; run with all_in_one (n_neighs=10, pruning=False,
    insertion=True, prop_insert=0.50) to infer trajectories and pseudotime
  repo: https://github.com/PeterZZQ/CellPaths.git
- name: scVelo
  role: Computes RNA velocity used during CellPath preprocessing of the simulated
    data
- name: Slingshot
  role: Baseline TI method; requires the true root cluster to be provided
- name: VeTra
  role: Baseline RNA velocity-based TI method; run with clusternumber 3 and 4 (or
    4 on the bifurcating dataset)
- name: CellRank
  role: Baseline method; its latent_time output is used as inferred pseudotime
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100095
    title: Inference of high-resolution trajectories in single-cell RNA-seq data by
      using RNA velocity
  source_content_hash: sha256:fa72f375cb26419c1ca5aa783d5d0ce23d3f73edc4b37b5496510f220a898c1d
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
  - Monocle
  - Slingshot
  - SymSim
  - VeTra
  - VeloSim
  - dyngen
  - dynmethods
  - scVelo
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# single-cell-trajectory-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate simulated single-cell RNA-seq datasets with known trajectory topologies (e.g., binary trees, bifurcations) and ground-truth pseudotime using simulators such as VeloSim and dyngen, enabling quantitative benchmarking of trajectory inference (TI) methods via Kendall rank correlation and assignment purity metrics.

## When to use

Use when benchmarking trajectory inference or pseudotime methods (e.g., CellPath, Slingshot, VeTra, CellRank) against a known ground truth that real scRNA-seq data cannot provide.

## When NOT to use

- Evaluating methods on real biological datasets where no ground-truth pseudotime exists — use qualitative trajectory agreement or known biology instead
- Benchmarking clustering-only or fate-probability tools that do not output pseudotime or per-path trajectories comparable to the simulated ground truth
- The simulated topology does not match the method's assumptions (e.g., methods restricted to one connected trajectory on multi-path tree data), confounding fairness of comparison

## Inputs

- Simulation scripts from the PeterZZQ/CellPath_test repository
- Simulator parameters (cell count, gene count, tree/bifurcation topology, number of simulation runs)
- Ground-truth pseudotime and branching labels emitted by VeloSim or dyngen
- Simulated count matrix (optionally with spliced/unspliced layers for RNA velocity)

## Outputs

- Simulated scRNA-seq datasets with ground-truth pseudotime (e.g., 1,999 cells x 500 genes VeloSim binary tree; 2,000-cell dyngen binary tree; 1,587 cells x 99 genes dyngen bifurcation)
- Per-method Kendall rank correlations against ground truth (figure_6G_S6G_kendall_boxplots.png; bifurcating_benchmark_metrics.csv)
- Normalized average entropy scores of cell-to-trajectory assignments after the branching point

## How to apply

Generate simulated datasets with the desired topology: use VeloSim to create binary-tree datasets (e.g., 1,999 cells, 500 genes) and dyngen to create binary-tree (2,000 cells) and bifurcating datasets (e.g., 4 simulation runs producing 1,587 cells and 99 genes), retaining the simulated ground-truth pseudotime and branching structure. Scripts for these simulations are deposited in the PeterZZQ/CellPath_test repository, with simulation scripts in PeterZZQ/CellPath_test and the simulated cycle-tree example dataset in example_data/simulated/. Run each TI method on the identical simulated data — for CellPath, initialize cp.CellPath with scvelo-preprocessed adata and call all_in_one with n_neighs=10, pruning=False, insertion=True, prop_insert=0.50 and a dataset-appropriate num_metacells — then compare inferred pseudotime to ground truth using Kendall rank correlation per path (Figures 6G/S6G, 7F) and, for branching data, normalized average entropy of cell-to-trajectory assignments (Figure 7G). Because Slingshot needs a user-specified root and VeTra a clusternumber (3 or 4), fix these competing parameters explicitly so the comparison is reproducible. Validation succeeds when CellPath achieves higher Kendall correlations than VeTra and CellRank, comparable or better correlations than Slingshot, and the highest normalized average entropy score on the dyngen branching dataset.

## Related tools

- **VeloSim** (Simulates binary-tree single-cell datasets (1,999 cells, 500 genes) with ground-truth pseudotime for benchmarking)
- **dyngen** (Simulates binary-tree (2,000 cells) and bifurcating (1,587 cells, 99 genes) datasets with ground-truth pseudotime and branching structure)
- **CellPath** (TI method under evaluation; run with all_in_one (n_neighs=10, pruning=False, insertion=True, prop_insert=0.50) to infer trajectories and pseudotime) — https://github.com/PeterZZQ/CellPaths.git
- **scVelo** (Computes RNA velocity used during CellPath preprocessing of the simulated data)
- **Slingshot** (Baseline TI method; requires the true root cluster to be provided)
- **VeTra** (Baseline RNA velocity-based TI method; run with clusternumber 3 and 4 (or 4 on the bifurcating dataset))
- **CellRank** (Baseline method; its latent_time output is used as inferred pseudotime)

## Examples

```
cellpath_obj = cp.CellPath(adata=adata, preprocess=True); cellpath_obj.all_in_one(num_metacells=num_metacells, n_neighs=10, pruning=False, insertion=True, prop_insert=0.50)
```

## Evaluation signals

- Boxplot figure figure_6G_S6G_kendall_boxplots.png exists and shows CellPath and Slingshot (with true root) attaining the highest Kendall rank correlations on VeloSim and dyngen binary-tree datasets
- On the dyngen bifurcating dataset, CellPath attains the highest normalized average entropy score among CellPath, VeTra, and Slingshot, with per-path Kendall correlations and entropy values recorded in bifurcating_benchmark_metrics.csv
- Each simulated replicate carries ground-truth pseudotime and branching labels so Kendall correlation is computable for every method and path
- CellPath is run with the fixed parameters n_neighs=10, pruning=False, insertion=True, prop_insert=0.50, matching the manuscript's settings
- Slingshot is given the true root cluster and VeTra a fixed clusternumber so baseline comparisons are reproducible

## Limitations

- Performance of all RNA velocity-based methods is limited by the accuracy of RNA velocity inference; noise in estimated RNA velocity affects all such methods
- Simulated datasets may not capture all real-data complexity; benchmark conclusions (e.g., VeTra's lower Kendall correlations) reflect the specific VeloSim/dyngen topologies used

## Evidence

- [other] Generate binary-tree simulated single-cell datasets using the VeloSim simulator (1,999 cells, 500 genes) and the dyngen simulator (2,000 cells), with ground-truth pseudotime: [evidence span withheld — anchor status: not_found]
- [other] Compute the Kendall rank correlation between each method's inferred pseudotime and the simulated ground-truth pseudotime for every dataset replicate.: [evidence span withheld — anchor status: not_found]
- [other] Run CellPath on each simulated dataset (initialize cp.CellPath with preprocessing via scvelo, then run all_in_one with n_neighs=10, pruning=False, insertion=True, prop_insert=0.50): [evidence span withheld — anchor status: not_found]
