---
name: kendall-rank-correlation-analysis
description: Use when benchmarking pseudotime inferred by trajectory-inference methods
  (CellPath, Slingshot, VeTra, CellRank) against ground-truth pseudotime from simulated
  single-cell datasets, e.g. VeloSim or dyngen binary trees.
when_to_use_negative:
- The datasets are real (e.g. mouse hematopoiesis, dentate-gyrus, pancreatic endocrinogenesis,
  forebrain) with no ground-truth pseudotime, so Kendall correlation against a known
  pseudotime cannot be computed
- The candidate method does not output per-cell pseudotime (e.g. dynamo, which estimates
  velocity and fates but does not extract cell trajectories)
- You need trajectory-topology or direction accuracy rather than relative pseudotime
  ordering accuracy, which Kendall rank correlation does not capture
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2229
tools:
- name: CellPath
  role: Trajectory-inference method under benchmark; its pseudo_order pseudotime is
    scored by Kendall correlation against simulated ground truth
  repo: github.com/PeterZZQ/CellPath
- name: scVelo
  role: Provides RNA velocity preprocessing used when initializing CellPath on the
    simulated datasets
- name: Slingshot
  role: Baseline (non-velocity-based) TI method run with the root cluster provided;
    pseudotime scored by Kendall correlation
- name: VeTra
  role: Baseline RNA velocity-based TI method run with clusternumber=3 and 4; pseudotime
    scored by Kendall correlation
- name: CellRank
  role: Baseline RNA velocity-based TI method; its latent_time output is used as pseudotime
    and scored by Kendall correlation
- name: VeloSim
  role: Simulator generating binary-tree single-cell datasets (1,999 cells, 500 genes)
    with ground-truth pseudotime
- name: dyngen
  role: Simulator generating binary-tree single-cell datasets (2,000 cells) with ground-truth
    pseudotime
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100095
    title: Inference of high-resolution trajectories in single-cell RNA-seq data by
      using RNA velocity
  source_content_hash: sha256:c7da57b2ee291fcfd83bc2b27dc08b02a8267b4abb31d816664a99182e1cebf1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/PeterZZQ/CellPath
  tools:
  - CellPath
  - CellRank
  - Slingshot
  - VeTra
  - VeloSim
  - dyngen
  - scVelo
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# kendall-rank-correlation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks trajectory-inference pseudotime accuracy by computing Kendall rank correlation between each method's inferred pseudotime and simulated ground-truth pseudotime, then aggregating per-method correlations into comparison boxplots (Figures 6G and S6G) on simulated tree-structured single-cell datasets.

## When to use

Use when benchmarking pseudotime inferred by trajectory-inference methods (CellPath, Slingshot, VeTra, CellRank) against ground-truth pseudotime from simulated single-cell datasets, e.g. VeloSim or dyngen binary trees.

## When NOT to use

- The datasets are real (e.g. mouse hematopoiesis, dentate-gyrus, pancreatic endocrinogenesis, forebrain) with no ground-truth pseudotime, so Kendall correlation against a known pseudotime cannot be computed
- The candidate method does not output per-cell pseudotime (e.g. dynamo, which estimates velocity and fates but does not extract cell trajectories)
- You need trajectory-topology or direction accuracy rather than relative pseudotime ordering accuracy, which Kendall rank correlation does not capture

## Inputs

- Simulated single-cell datasets with ground-truth pseudotime (VeloSim binary trees: 1,999 cells x 500 genes; dyngen binary trees: 2,000 cells), simulation scripts from PeterZZQ/CellPath_test
- Inferred pseudotime per method: cellpath_obj.pseudo_order (CellPath), Slingshot pseudotime with root cluster provided, VeTra pseudotime (clusternumber=3 and 4), CellRank latent_time

## Outputs

- Kendall rank correlation values per method per dataset replicate
- Aggregated per-method Kendall correlation distributions rendered as boxplots corresponding to Figures 6G and S6G (figure_6G_S6G_kendall_boxplots.png)

## How to apply

Generate simulated single-cell datasets with known ground-truth pseudotime — binary trees from VeloSim (1,999 cells, 500 genes) and dyngen (2,000 cells) using scripts in the PeterZZQ/CellPath_test repository. Run each candidate trajectory-inference method on the same datasets: CellPath (initialized with scvelo preprocessing, then all_in_one with n_neighs=10, pruning=False, insertion=True, prop_insert=0.50; pseudotime read from cellpath_obj.pseudo_order), Slingshot with the root cluster provided, VeTra with clusternumber set to 3 and 4, and CellRank using its latent_time output as pseudotime. For each method and dataset replicate, compute the Kendall rank correlation between inferred pseudotime and the simulated ground-truth pseudotime. Aggregate the correlations per method and dataset type and render the comparison boxplots corresponding to Figures 6G and S6G (e.g. saved as figure_6G_S6G_kendall_boxplots.png). Correct application shows CellPath and Slingshot (with true root information) generally achieving the highest Kendall correlations, outperforming VeTra and CellRank.

## Related tools

- **CellPath** (Trajectory-inference method under benchmark; its pseudo_order pseudotime is scored by Kendall correlation against simulated ground truth) — github.com/PeterZZQ/CellPath
- **scVelo** (Provides RNA velocity preprocessing used when initializing CellPath on the simulated datasets)
- **Slingshot** (Baseline (non-velocity-based) TI method run with the root cluster provided; pseudotime scored by Kendall correlation)
- **VeTra** (Baseline RNA velocity-based TI method run with clusternumber=3 and 4; pseudotime scored by Kendall correlation)
- **CellRank** (Baseline RNA velocity-based TI method; its latent_time output is used as pseudotime and scored by Kendall correlation)
- **VeloSim** (Simulator generating binary-tree single-cell datasets (1,999 cells, 500 genes) with ground-truth pseudotime)
- **dyngen** (Simulator generating binary-tree single-cell datasets (2,000 cells) with ground-truth pseudotime)

## Examples

```
cellpath_obj = cp.CellPath(adata=adata, preprocess=True); cellpath_obj.all_in_one(num_metacells=num_metacells, n_neighs=10, pruning=False, num_trajs=num_trajs, insertion=True, prop_insert=0.50); kendalltau(cellpath_obj.pseudo_order['pseudotime'], truth_pseudotime)
```

## Evaluation signals

- The boxplot figure (figure_6G_S6G_kendall_boxplots.png) exists and contains one Kendall-correlation distribution per method and dataset type (VeloSim, dyngen binary trees)
- CellPath and Slingshot (with true root information) show generally higher Kendall correlations than VeTra and CellRank, consistent with the reported qualitative result
- Kendall correlations fall within [-1, 1], with well-performing methods showing strongly positive values against ground-truth pseudotime on every replicate
- Each method's correlation is computed on the identical simulated dataset replicates, ensuring a paired comparison across methods

## Limitations

- Performance of all RNA velocity-based methods, including CellPath, VeTra, and CellRank, is affected by the accuracy of RNA velocity inference given the noise in estimated RNA velocity
- Slingshot is only given the true root cluster information, which is not available in real analyses; comparisons without root information were not the basis of this benchmark
- Kendall correlation assesses pseudotime ordering only and does not evaluate trajectory topology, direction assignment, or resolution of detected paths

## Evidence
