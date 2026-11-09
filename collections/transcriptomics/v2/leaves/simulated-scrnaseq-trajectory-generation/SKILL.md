---
name: simulated-scrnaseq-trajectory-generation
description: Use when benchmarking trajectory inference or RNA-velocity-based methods
  against known ground truth, or when replicated simulated datasets (e.g., 785 cells
  x 600 genes cycle-tree) are needed to quantify pseudotime accuracy.
when_to_use_negative:
- The goal is inference on real scRNA-seq data without a ground-truth trajectory —
  simulation is only for controlled benchmarking, and the article notes all velocity-based
  methods are affected by noise in estimated RNA velocity.
- The analysis requires realistic dataset complexity (batch effects, dropouts, real
  topology uncertainty) that the simple cycle-tree simulation with fixed 785x600 dimensions
  does not capture.
- Evaluating methods whose output is cluster-level only (e.g., Directed-PAGA), which
  cannot produce the high-resolution per-cell pseudotime needed for per-path Kendall
  correlation.
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: CellPath
  role: Trajectory inference method under evaluation on the simulated cycle-tree datasets;
    run via all_in_one with n_neighs=10, pruning=False, insertion=True, prop_insert=0.50
  repo: https://github.com/PeterZZQ/CellPaths.git
- name: scVelo
  role: Computes the RNA velocity matrix that CellPath and velocity-based baselines
    consume from the simulated spliced/unspliced counts
- name: CellRank
  role: Baseline method combined with DPT to produce comparison pseudotime assignments
    on each simulated replicate
- name: VeTra
  role: Velocity-based baseline run with clusternumber=4, averaged over min_dist values
    0.4, 0.5, 0.8
- name: Slingshot
  role: Non-velocity baseline run with the ground-truth root cell supplied
- name: VeloSim
  role: Generates the simulated cycle-tree datasets with ground-truth pseudotime using
    distinct random seeds
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100095
    title: Inference of high-resolution trajectories in single-cell RNA-seq data by
      using RNA velocity
  source_content_hash: sha256:3524a538358178c3ca2bd816a50e65ef0a5013945b5b37d926a21645915ec1e0
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
  - VeloSim
  - scVelo
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# simulated-scrnaseq-trajectory-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generate simulated single-cell RNA-seq datasets with known ground-truth trajectories (e.g., VeloSim cycle-tree topologies with spliced/unspliced counts and pseudotime) so that trajectory inference methods such as CellPath can be benchmarked against baselines using Kendall rank correlation.

## When to use

Use when benchmarking trajectory inference or RNA-velocity-based methods against known ground truth, or when replicated simulated datasets (e.g., 785 cells x 600 genes cycle-tree) are needed to quantify pseudotime accuracy.

## When NOT to use

- The goal is inference on real scRNA-seq data without a ground-truth trajectory — simulation is only for controlled benchmarking, and the article notes all velocity-based methods are affected by noise in estimated RNA velocity.
- The analysis requires realistic dataset complexity (batch effects, dropouts, real topology uncertainty) that the simple cycle-tree simulation with fixed 785x600 dimensions does not capture.
- Evaluating methods whose output is cluster-level only (e.g., Directed-PAGA), which cannot produce the high-resolution per-cell pseudotime needed for per-path Kendall correlation.

## Inputs

- VeloSim-simulated cycle-tree datasets (anndata/.h5ad with spliced/unspliced counts, ground-truth pseudotime, path/branch assignments)
- Random seeds for replicate generation (10 replicates, 785 cells, 600 genes each)
- Ground-truth root cell identity (needed for Slingshot baseline)

## Outputs

- Simulated replicate datasets with ground-truth trajectory topology and pseudotime
- Per-method, per-path Kendall rank correlation table (kendall_scores.csv)
- Inferred pseudotime/trajectory assignments from each TI method (e.g., cellpath_obj.pseudo_order)

## How to apply

Generate multiple simulated datasets with VeloSim using distinct random seeds, each with a fixed cell and gene count (e.g., 10 replicates of 785 cells and 600 genes on a cycle-tree topology), retaining the ground-truth pseudotime and path assignments for each cell. Provide the simulated data as an anndata object containing spliced/unspliced counts so that RNA velocity can be computed upstream with scVelo (or supplied to CellPath with preprocess=False/True accordingly). Run each inference method — CellPath via all_in_one (n_neighs=10, pruning=False, insertion=True, prop_insert=0.50), CellRank+DPT, VeTra (clusternumber=4, averaged over min_dist 0.4/0.5/0.8), Vdpt, and Slingshot with the ground-truth root cell supplied — on every replicate. Compute Kendall rank correlation between each method's inferred pseudotime and the ground truth for every path in every replicate, aggregate per-method per-path scores, and write results to a table such as kendall_scores.csv. The simulated cycle-tree dataset ships with the CellPath repo under example_data/simulated/ and run_cellpath.ipynb demonstrates the pipeline on it.

## Related tools

- **CellPath** (Trajectory inference method under evaluation on the simulated cycle-tree datasets; run via all_in_one with n_neighs=10, pruning=False, insertion=True, prop_insert=0.50) — https://github.com/PeterZZQ/CellPaths.git
- **scVelo** (Computes the RNA velocity matrix that CellPath and velocity-based baselines consume from the simulated spliced/unspliced counts)
- **CellRank** (Baseline method combined with DPT to produce comparison pseudotime assignments on each simulated replicate)
- **VeTra** (Velocity-based baseline run with clusternumber=4, averaged over min_dist values 0.4, 0.5, 0.8)
- **Slingshot** (Non-velocity baseline run with the ground-truth root cell supplied)
- **VeloSim** (Generates the simulated cycle-tree datasets with ground-truth pseudotime using distinct random seeds)

## Examples

```
cellpath_obj = cp.CellPath(adata = adata, preprocess = True); cellpath_obj.all_in_one(num_metacells = num_metacells, n_neighs = 10, pruning = False, num_trajs = num_trajs, insertion = True, prop_insert = 0.50)
```

## Evaluation signals

- Per-path Kendall rank correlation values reproduce the Figure 5G pattern: CellPath achieves the highest correlation among CellRank, VeTra, Vdpt, and Slingshot on cycle-tree data
- Each of the 10 replicates differs (distinct random seeds) yet retains the 785-cell x 600-gene cycle-tree structure with ground-truth pseudotime
- kendall_scores.csv contains one row per method per path per replicate with correlation values in [-1, 1]
- CellPath output (cellpath_obj.pseudo_order) assigns cells to trajectories covering the expected proportion of inserted cells (prop_insert=0.50)
- Baseline methods converge: VeTra results stable across the three min_dist values (0.4, 0.5, 0.8) after averaging

## Limitations

- Performance of all RNA velocity-based methods, including on simulated data, is limited by the accuracy of RNA velocity inference given its noise
- Simulated cycle-tree data is a single topology; results may not generalize to other trajectory structures, and CellPath cannot yet align locally detected paths into a global view
- RNA velocity estimation challenges remain unresolved (Bergen et al., 2021), affecting the fidelity of velocity-based benchmarking even on simulations

## Evidence
