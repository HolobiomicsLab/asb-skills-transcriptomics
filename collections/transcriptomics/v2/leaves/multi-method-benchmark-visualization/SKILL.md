---
name: multi-method-benchmark-visualization
description: Use when comparing a new trajectory inference method's pseudotime accuracy
  against baselines on simulated single-cell datasets with ground truth, using Kendall
  rank correlation as the metric.
when_to_use_negative:
- Datasets lack ground-truth pseudotime (e.g., purely real data without simulation),
  making Kendall correlation against truth impossible — use qualitative trajectory
  assessment instead
- The goal is comparing trajectory topology/structure rather than cell ordering accuracy
  — Kendall correlation only evaluates pseudotime ordering
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: CellPath
  role: Primary method under evaluation; infers cell trajectories and pseudotime by
    integrating gene expression and RNA velocity
  repo: https://github.com/PeterZZQ/CellPaths.git
- name: scVelo
  role: Calculates RNA velocity used in CellPath preprocessing during initialization
- name: Slingshot
  role: Baseline non-velocity-based TI method run with the true root cluster provided
- name: VeTra
  role: Baseline RNA velocity-based TI method run with clusternumber 3 and 4
- name: CellRank
  role: Baseline RNA velocity-based method whose latent_time output serves as inferred
    pseudotime
- name: VeloSim
  role: Simulates binary-tree single-cell datasets (1,999 cells, 500 genes) with ground-truth
    pseudotime
  repo: https://github.com/PeterZZQ/CellPath_test
- name: dyngen
  role: Simulates binary-tree datasets (2,000 cells) with ground-truth pseudotime
  repo: https://github.com/PeterZZQ/CellPath_test
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100095
    title: Inference of high-resolution trajectories in single-cell RNA-seq data by
      using RNA velocity
  source_content_hash: sha256:3bda9f994b8554731011e0a1ec31f66a74a6739e6f4b0eaea7973c381c71239c
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
  - dyngen
  - scVelo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 16
---

# multi-method-benchmark-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark CellPath's pseudotime inference against competing trajectory inference methods (Slingshot, VeTra, CellRank) on simulated tree-structured datasets by computing Kendall rank correlation against ground-truth pseudotime and rendering comparison boxplots (Figures 6G and S6G).

## When to use

Use when comparing a new trajectory inference method's pseudotime accuracy against baselines on simulated single-cell datasets with ground truth, using Kendall rank correlation as the metric.

## When NOT to use

- Datasets lack ground-truth pseudotime (e.g., purely real data without simulation), making Kendall correlation against truth impossible — use qualitative trajectory assessment instead
- The goal is comparing trajectory topology/structure rather than cell ordering accuracy — Kendall correlation only evaluates pseudotime ordering

## Inputs

- Simulated tree-structured single-cell datasets from VeloSim (1,999 cells, 500 genes) and dyngen (2,000 cells) with ground-truth pseudotime
- scRNA-seq count matrix and RNA velocity matrix (adata with scvelo velocity) for CellPath initialization
- Simulation and benchmark scripts from github.com/PeterZZQ/CellPath_test

## Outputs

- Per-replicate Kendall rank correlation values between each method's inferred pseudotime and ground-truth pseudotime
- Boxplot figure comparing Kendall correlations across methods (figure_6G_S6G_kendall_boxplots.png)
- Per-method inferred pseudotime (e.g., cellpath_obj.pseudo_order for CellPath, latent_time for CellRank)

## How to apply

Generate tree-structured simulated single-cell datasets with known ground-truth pseudotime using VeloSim (1,999 cells, 500 genes) and dyngen (2,000 cells) binary-tree simulators, with scripts from the PeterZZQ/CellPath_test repository. Run CellPath on each dataset, initializing with scvelo preprocessing and calling all_in_one with n_neighs=10, pruning=False, insertion=True, prop_insert=0.50, then extract inferred pseudotime from cellpath_obj.pseudo_order. Run the baselines on the same data: Slingshot with the true root cluster provided, VeTra with clusternumber set to both 3 and 4, and CellRank using its latent_time output as pseudotime. For every dataset replicate, compute the Kendall rank correlation between each method's inferred pseudotime and the simulated ground truth, aggregate correlations per method and dataset type, and render comparison boxplots saved as figure_6G_S6G_kendall_boxplots.png.

## Related tools

- **CellPath** (Primary method under evaluation; infers cell trajectories and pseudotime by integrating gene expression and RNA velocity) — https://github.com/PeterZZQ/CellPaths.git
- **scVelo** (Calculates RNA velocity used in CellPath preprocessing during initialization)
- **Slingshot** (Baseline non-velocity-based TI method run with the true root cluster provided)
- **VeTra** (Baseline RNA velocity-based TI method run with clusternumber 3 and 4)
- **CellRank** (Baseline RNA velocity-based method whose latent_time output serves as inferred pseudotime)
- **VeloSim** (Simulates binary-tree single-cell datasets (1,999 cells, 500 genes) with ground-truth pseudotime) — https://github.com/PeterZZQ/CellPath_test
- **dyngen** (Simulates binary-tree datasets (2,000 cells) with ground-truth pseudotime) — https://github.com/PeterZZQ/CellPath_test

## Examples

```
cellpath_obj = cp.CellPath(adata = adata, preprocess = True); cellpath_obj.all_in_one(num_metacells = num_metacells, n_neighs = 10, pruning = False, insertion = True, prop_insert = 0.50)
```

## Evaluation signals

- The boxplot figure figure_6G_S6G_kendall_boxplots.png exists and shows per-method Kendall correlation distributions
- CellPath shows higher Kendall rank correlation against ground-truth pseudotime than VeTra (clusternumber 3 and 4) and CellRank on the simulated tree datasets
- CellPath performs comparably to or better than Slingshot run with the true root information, matching the reported finding that both generally perform best
- Kendall correlations are computed separately per dataset replicate and aggregated per method and dataset type (VeloSim vs dyngen binary trees)

## Limitations

- Performance of all RNA velocity-based methods is limited by the accuracy of RNA velocity inference; noise in estimated RNA velocity affects CellPath, VeTra, and CellRank alike
- Slingshot baseline is given true root information, which is a favorable condition a real analysis may not have
- RNA velocity estimation challenges remain unresolved (citing Bergen et al., 2021)

## Evidence

- [results] On tree-structured simulated datasets (VeloSim and dyngen binary trees), CellPath and Slingshot (with true root information) generally perform best in pseudotime inference accuracy, as measured by Kendall rank correlation, outperforming VeTra and CellRank.: [evidence span withheld — anchor status: not_found]
