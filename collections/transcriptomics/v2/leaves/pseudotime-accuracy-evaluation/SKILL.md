---
name: pseudotime-accuracy-evaluation
description: Use when benchmarking trajectory inference methods on simulated single-cell
  datasets with known ground-truth pseudotime, e.g. quantifying pseudotime accuracy
  via Kendall rank correlation.
when_to_use_negative:
- Real scRNA-seq datasets (e.g., hematopoiesis, dentate gyrus, pancreatic endocrinogenesis,
  forebrain) where no simulated ground-truth pseudotime exists for Kendall correlation
- When the goal is trajectory topology or direction assignment rather than pseudotime
  ordering accuracy along paths
- Datasets where estimated RNA velocity is unreliable, since performance of all RNA
  velocity-based methods is affected by velocity inference accuracy
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: CellPath
  role: Trajectory inference method under evaluation; its pseudo_order output is the
    inferred pseudotime scored against ground truth (all_in_one with n_neighs=10,
    pruning=False, insertion=True, prop_insert=0.50)
  repo: https://github.com/PeterZZQ/CellPaths.git
- name: CellPath_test
  role: Manuscript test scripts and simulation pipeline for reproducing the pseudotime
    benchmark and Figures 6G/S6G
  repo: https://github.com/PeterZZQ/CellPath_test
- name: scVelo
  role: Computes the RNA velocity used during CellPath preprocessing before pseudotime
    inference
- name: Slingshot
  role: Non-velocity-based baseline; run with the root cluster provided and scored
    by Kendall correlation
- name: VeTra
  role: RNA velocity-based baseline run with clusternumber set to 3 and 4; pseudotime
    scored by Kendall correlation
- name: CellRank
  role: RNA velocity-based baseline; its latent_time output is used as the inferred
    pseudotime for scoring
- name: VeloSim
  role: Simulates binary-tree datasets (1,999 cells, 500 genes) supplying the ground-truth
    pseudotime for evaluation
- name: dyngen
  role: Simulates binary-tree datasets (2,000 cells) supplying the ground-truth pseudotime
    for evaluation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100095
    title: Inference of high-resolution trajectories in single-cell RNA-seq data by
      using RNA velocity
  source_content_hash: sha256:c98bc6613656f29832d462843d7d3232ffdb86c1b3c286686498959646701a9c
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
  - CellPath_test
  - CellRank
  - Slingshot
  - VeTra
  - VeloSim
  - dyngen
  - scVelo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# pseudotime-accuracy-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark pseudotime inference accuracy by computing Kendall rank correlation between each trajectory inference method's inferred pseudotime and simulated ground-truth pseudotime on tree-structured datasets, comparing CellPath against Slingshot, VeTra, and CellRank.

## When to use

Use when benchmarking trajectory inference methods on simulated single-cell datasets with known ground-truth pseudotime, e.g. quantifying pseudotime accuracy via Kendall rank correlation.

## When NOT to use

- Real scRNA-seq datasets (e.g., hematopoiesis, dentate gyrus, pancreatic endocrinogenesis, forebrain) where no simulated ground-truth pseudotime exists for Kendall correlation
- When the goal is trajectory topology or direction assignment rather than pseudotime ordering accuracy along paths
- Datasets where estimated RNA velocity is unreliable, since performance of all RNA velocity-based methods is affected by velocity inference accuracy

## Inputs

- Simulated scRNA-seq datasets with ground-truth pseudotime (VeloSim binary tree: 1,999 cells x 500 genes; dyngen binary tree: 2,000 cells)
- scRNA-seq count matrix and RNA velocity matrix, with velocity calculated via scvelo
- Inferred pseudotime per method: cellpath_obj.pseudo_order (CellPath), Slingshot pseudotime, VeTra pseudotime (clusternumber 3 and 4), CellRank latent_time

## Outputs

- Kendall rank correlation between inferred and ground-truth pseudotime per method, dataset, and replicate
- Aggregated per-method comparison boxplots reproducing Figures 6G and S6G (figure_6G_S6G_kendall_boxplots.png)

## How to apply

Generate binary-tree simulated single-cell datasets with ground-truth pseudotime using VeloSim (1,999 cells, 500 genes) and dyngen (2,000 cells), with scripts from the PeterZZQ/CellPath_test repository. Run each method on the same data: CellPath initialized as cp.CellPath(adata=adata, preprocess=True) with scvelo-computed velocity followed by all_in_one(n_neighs=10, pruning=False, insertion=True, prop_insert=0.50), Slingshot with the root cluster provided, VeTra with clusternumber set to 3 and 4, and CellRank using its latent_time output as pseudotime. For every dataset replicate, compute the Kendall rank correlation between each method's inferred pseudotime (cellpath_obj.pseudo_order for CellPath) and the simulated ground-truth pseudotime. Aggregate Kendall correlations per method and dataset type and render the comparison boxplots corresponding to Figures 6G and S6G, saved as figure_6G_S6G_kendall_boxplots.png. The evaluation is correct when CellPath (and Slingshot given true root information) ranks highest, matching the paper's claim that CellPath finds more accurate and detailed trajectories than state-of-the-art TI methods with or without RNA velocity.

## Related tools

- **CellPath** (Trajectory inference method under evaluation; its pseudo_order output is the inferred pseudotime scored against ground truth (all_in_one with n_neighs=10, pruning=False, insertion=True, prop_insert=0.50)) — https://github.com/PeterZZQ/CellPaths.git
- **CellPath_test** (Manuscript test scripts and simulation pipeline for reproducing the pseudotime benchmark and Figures 6G/S6G) — https://github.com/PeterZZQ/CellPath_test
- **scVelo** (Computes the RNA velocity used during CellPath preprocessing before pseudotime inference)
- **Slingshot** (Non-velocity-based baseline; run with the root cluster provided and scored by Kendall correlation)
- **VeTra** (RNA velocity-based baseline run with clusternumber set to 3 and 4; pseudotime scored by Kendall correlation)
- **CellRank** (RNA velocity-based baseline; its latent_time output is used as the inferred pseudotime for scoring)
- **VeloSim** (Simulates binary-tree datasets (1,999 cells, 500 genes) supplying the ground-truth pseudotime for evaluation)
- **dyngen** (Simulates binary-tree datasets (2,000 cells) supplying the ground-truth pseudotime for evaluation)

## Examples

```
cellpath_obj = cp.CellPath(adata=adata, preprocess=True); cellpath_obj.all_in_one(num_metacells=num_metacells, n_neighs=10, pruning=False, num_trajs=num_trajs, insertion=True, prop_insert=0.50)
```

## Evaluation signals

- Kendall rank correlation is computed per replicate against the simulated ground-truth pseudotime for every method and dataset type, then aggregated for the boxplots
- CellPath and Slingshot (with true root information) show the highest Kendall correlations, outperforming VeTra and CellRank, consistent with the reported qualitative result
- figure_6G_S6G_kendall_boxplots.png exists and renders per-method comparison boxplots matching Figures 6G and S6G
- CellPath runs used the manuscript parameters n_neighs=10, pruning=False, insertion=True, prop_insert=0.50 and returned cellpath_obj.pseudo_order
- VeTra was evaluated at both clusternumber=3 and clusternumber=4, and CellRank's pseudotime was taken from its latent_time output

## Limitations

- Performance of all RNA velocity-based methods is limited by the accuracy of RNA velocity inference
- Slingshot only competes in this benchmark when the true root cluster is provided
- Ground-truth-based Kendall evaluation is restricted to simulated datasets (VeloSim, dyngen binary trees); real datasets lack pseudotime truth
- Challenges in the estimation of RNA velocity still exist (Bergen et al., 2021)

## Evidence
