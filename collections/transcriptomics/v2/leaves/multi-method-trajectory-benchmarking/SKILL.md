---
name: multi-method-trajectory-benchmarking
description: Use when comparing pseudotime/TI methods on datasets with known ground-truth
  cell ordering (e.g., VeloSim-simulated cycle-tree data), or when needing a per-path
  Kendall rank correlation benchmark against baseline velocity-based and expression-based
  methods.
when_to_use_negative:
- The dataset has no ground-truth pseudotime or simulated trajectory structure, so
  Kendall correlation against truth cannot be computed.
- The goal is applying a single TI method to a new real dataset (e.g., hematopoiesis,
  dentate gyrus, pancreatic endocrinogenesis, forebrain) rather than head-to-head
  method benchmarking.
- The analysis requires only cluster-level trajectory topology without pseudotime
  ordering comparison (e.g., Directed-PAGA output).
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellPath
  role: Method under test; infers multiple high-resolution trajectories and pseudotime
    by integrating gene expression and RNA velocity, run via all_in_one with n_neighs=10,
    pruning=False, insertion=True, prop_insert=0.50
  repo: https://github.com/PeterZZQ/CellPaths.git
- name: CellRank
  role: Baseline TI method combined with DPT to produce comparison pseudotime assignments
- name: VeTra
  role: Baseline velocity-based TI method run with clusternumber=4, results averaged
    over min_dist values 0.4, 0.5, and 0.8
- name: Vdpt
  role: Additional velocity-based comparison method run on each dataset
- name: Slingshot
  role: Expression-only baseline TI method run on each dataset with the ground-truth
    root cell supplied
- name: scVelo
  role: Computes the RNA velocity matrix used as input by CellPath and the other velocity-based
    baselines
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100095
    title: Inference of high-resolution trajectories in single-cell RNA-seq data by
      using RNA velocity
  source_content_hash: sha256:d196098c464d31bc0f00e369dabeec8b832f4cb9b63e619c19abd944e63adc2c
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
  - Angle
  - CellPath
  - CellRank
  - Cellpath
  - Cytopath
  - Monocle3
  - PAGA
  - Slingshot
  - Vdpt
  - VeTra
  - scVelo
  - scvelo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- trajectory-method-benchmarking
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# multi-method-trajectory-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark trajectory inference (TI) methods — CellPath, CellRank+DPT, VeTra, Vdpt, Slingshot — on simulated VeloSim cycle-tree datasets by scoring each method's pseudotime against ground truth with Kendall rank correlation. It quantifies how accurately each method recovers cell ordering, following the Figure 5G-style evaluation of the CellPath paper (Zhang & Zhang, 2021).

## When to use

Use when comparing pseudotime/TI methods on datasets with known ground-truth cell ordering (e.g., VeloSim-simulated cycle-tree data), or when needing a per-path Kendall rank correlation benchmark against baseline velocity-based and expression-based methods.

## When NOT to use

- The dataset has no ground-truth pseudotime or simulated trajectory structure, so Kendall correlation against truth cannot be computed.
- The goal is applying a single TI method to a new real dataset (e.g., hematopoiesis, dentate gyrus, pancreatic endocrinogenesis, forebrain) rather than head-to-head method benchmarking.
- The analysis requires only cluster-level trajectory topology without pseudotime ordering comparison (e.g., Directed-PAGA output).

## Inputs

- VeloSim-simulated cycle-tree datasets as AnnData (.h5ad) objects with 785 cells, 600 genes, ground-truth pseudotime, and RNA velocity (from scVelo)
- scRNA-seq count matrix plus RNA velocity matrix (scVelo/velocyto output) for each replicate
- Ground-truth root cell annotation for Slingshot

## Outputs

- Per-method, per-path Kendall rank correlation scores aggregated across 10 replicates, written to kendall_scores.csv
- CellPath pseudotime assignments (cellpath_obj.pseudo_order) and equivalent pseudotime vectors from CellRank+DPT, VeTra, Vdpt, and Slingshot

## How to apply

Generate 10 replicate VeloSim cycle-tree datasets (785 cells, 600 genes) with distinct random seeds, storing each as an AnnData (.h5ad) object. Run CellPath on each replicate with all_in_one (n_neighs=10, pruning=False, insertion=True, prop_insert=0.50) to get per-trajectory pseudotime via pseudo_order; run CellRank combined with DPT, VeTra (clusternumber=4, averaged over min_dist in {0.4, 0.5, 0.8}), Vdpt, and Slingshot (supplying the ground-truth root cell) on the same data. For each method and every path in every replicate, compute Kendall rank correlation between inferred and ground-truth pseudotime, aggregate scores, and write the per-method, per-path table to kendall_scores.csv. Validate by checking that CellPath achieves the highest Kendall correlation among methods, matching the pattern reported in Figure 5G.

## Related tools

- **CellPath** (Method under test; infers multiple high-resolution trajectories and pseudotime by integrating gene expression and RNA velocity, run via all_in_one with n_neighs=10, pruning=False, insertion=True, prop_insert=0.50) — https://github.com/PeterZZQ/CellPaths.git
- **CellRank** (Baseline TI method combined with DPT to produce comparison pseudotime assignments)
- **VeTra** (Baseline velocity-based TI method run with clusternumber=4, results averaged over min_dist values 0.4, 0.5, and 0.8)
- **Vdpt** (Additional velocity-based comparison method run on each dataset)
- **Slingshot** (Expression-only baseline TI method run on each dataset with the ground-truth root cell supplied)
- **scVelo** (Computes the RNA velocity matrix used as input by CellPath and the other velocity-based baselines)

## Examples

```
cellpath_obj = cp.CellPath(adata=adata, preprocess=True); cellpath_obj.all_in_one(num_metacells=num_metacells, n_neighs=10, pruning=False, num_trajs=num_trajs, insertion=True, prop_insert=0.50)
```

## Evaluation signals

- CellPath achieves the highest Kendall rank correlation among the compared methods, matching the pattern in Figure 5G.
- kendall_scores.csv contains per-method, per-path rows covering all 10 replicates and all five comparison methods.
- Each VeloSim replicate uses a distinct random seed yet has the same dimensions (785 cells, 600 genes).
- VeTra scores are the average over min_dist values {0.4, 0.5, 0.8}, not a single run.
- CellPath's pseudo_order contains per-trajectory pseudotime and branching assignments for all cells (including inserted cells up to prop_insert=0.50).

## Limitations

- Performance of all RNA velocity-based methods is limited by the accuracy of RNA velocity inference, given the noise in estimated velocity.
- CellPath cannot yet align locally detected paths into a global view or connect points between paths, which complicates comparing multi-path outputs.
- Challenges in RNA velocity estimation remain unresolved (as noted citing Bergen et al., 2021), affecting velocity-based baselines and CellPath alike.

## Evidence
