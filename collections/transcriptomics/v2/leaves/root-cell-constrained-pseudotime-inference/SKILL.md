---
name: root-cell-constrained-pseudotime-inference
description: Use when simulated single-cell datasets (e.g. VeloSim cycle-tree, 785
  cells x 600 genes) have known ground-truth pseudotime and a known root cell, and
  you need to quantify how accurately TI methods recover cell ordering.
when_to_use_negative:
- Datasets lack a known ground-truth root cell and pseudotime, so Kendall correlation
  against ground truth cannot be computed
- The analysis goal is trajectory discovery on real data rather than method benchmarking
  — use the TI method directly instead of the comparison harness
- Topology is a simple linear path where direction is unambiguous without a supplied
  root cell
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: CellPath
  role: Primary TI method under evaluation; run via all_in_one to obtain per-trajectory
    pseudotime from expression and RNA velocity
  repo: https://github.com/PeterZZQ/CellPaths.git
- name: CellRank
  role: Velocity-based baseline; combined with DPT to produce baseline pseudotime
    assignments
- name: VeTra
  role: Velocity-based baseline; run with clusternumber=4 and results averaged over
    min_dist values 0.4, 0.5, 0.8
- name: Vdpt
  role: Velocity-based comparison method run on each dataset
- name: Slingshot
  role: Expression-only baseline constrained by the supplied ground-truth root cell
- name: scVelo
  role: Computes the RNA velocity matrix used as CellPath input (preprocess flag controls
    whether velocity is computed at initialization)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100095
    title: Inference of high-resolution trajectories in single-cell RNA-seq data by
      using RNA velocity
  source_content_hash: sha256:e98d95c200fdc6f4b74e8b1d517a8a4726684d52830870b170a22ca7b4a92e29
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
  - Vdpt
  - VeTra
  - scVelo
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# root-cell-constrained-pseudotime-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark cell ordering (pseudotime) accuracy by running trajectory inference methods—CellPath, CellRank+DPT, VeTra, Vdpt, and Slingshot with a supplied ground-truth root cell—on simulated cycle-tree datasets and scoring inferred orderings against ground truth with Kendall rank correlation.

## When to use

Use when simulated single-cell datasets (e.g. VeloSim cycle-tree, 785 cells x 600 genes) have known ground-truth pseudotime and a known root cell, and you need to quantify how accurately TI methods recover cell ordering.

## When NOT to use

- Datasets lack a known ground-truth root cell and pseudotime, so Kendall correlation against ground truth cannot be computed
- The analysis goal is trajectory discovery on real data rather than method benchmarking — use the TI method directly instead of the comparison harness
- Topology is a simple linear path where direction is unambiguous without a supplied root cell

## Inputs

- 10 simulated cycle-tree datasets from VeloSim (785 cells, 600 genes each, distinct random seeds), with ground-truth pseudotime and ground-truth root cell
- h5ad AnnData objects with spliced/unspliced counts for RNA velocity calculation via scVelo

## Outputs

- Per-method, per-path pseudotime assignments (e.g. cellpath_obj.pseudo_order for CellPath)
- kendall_scores.csv: Kendall rank correlation scores per method, per path, per replicate

## How to apply

Generate 10 VeloSim cycle-tree replicates (785 cells, 600 genes) with distinct random seeds. Run CellPath via all_in_one(num_metacells, n_neighs=10, pruning=False, num_trajs, insertion=True, prop_insert=0.50) after initializing cp.CellPath(adata=adata, preprocess=True) with scVelo-computed velocity. Run CellRank combined with DPT, VeTra (clusternumber=4, averaged over min_dist {0.4, 0.5, 0.8}), and Vdpt as velocity-based comparisons, and Slingshot with the ground-truth root cell explicitly supplied so the direction of its principal curve is constrained. Compute Kendall rank correlation between each method's inferred pseudotime and the ground-truth pseudotime per path in every replicate, aggregate, and write kendall_scores.csv. Rationale: root-cell constraint removes direction ambiguity for expression-only methods like Slingshot, giving them a fair footing against velocity-informed methods such as CellPath.

## Related tools

- **CellPath** (Primary TI method under evaluation; run via all_in_one to obtain per-trajectory pseudotime from expression and RNA velocity) — https://github.com/PeterZZQ/CellPaths.git
- **CellRank** (Velocity-based baseline; combined with DPT to produce baseline pseudotime assignments)
- **VeTra** (Velocity-based baseline; run with clusternumber=4 and results averaged over min_dist values 0.4, 0.5, 0.8)
- **Vdpt** (Velocity-based comparison method run on each dataset)
- **Slingshot** (Expression-only baseline constrained by the supplied ground-truth root cell)
- **scVelo** (Computes the RNA velocity matrix used as CellPath input (preprocess flag controls whether velocity is computed at initialization))

## Examples

```
cellpath_obj = cp.CellPath(adata=adata, preprocess=True); cellpath_obj.all_in_one(num_metacells=num_metacells, n_neighs=10, pruning=False, num_trajs=num_trajs, insertion=True, prop_insert=0.50)
```

## Evaluation signals

- Per-path Kendall correlation values should reproduce the Figure 5G pattern in which CellPath achieves the highest correlation among the compared methods on cycle-tree data
- Each of the 10 replicates must use a distinct random seed and yield per-path scores, aggregated per method in kendall_scores.csv
- Slingshot runs must consume the supplied ground-truth root cell; direction flips from a wrong root would degrade its scores
- VeTra scores must be the average over the three min_dist values (0.4, 0.5, 0.8), not a single run
- CellPath output structure check: cellpath_obj.pseudo_order contains pseudo-time and branching assignment per trajectory

## Limitations

- Performance of all RNA velocity-based methods is limited by the accuracy of RNA velocity inference; noise in estimated velocity affects CellPath, CellRank, VeTra, and Vdpt alike
- Slingshot requires a supplied root cell; without it, direction assignment is unreliable for snapshot scRNA-seq data
- VeTra results depend on 2D projections that can cause significant information loss, and its weakly connected components yield local rather than global paths

## Evidence

- [other] On the cycle-tree simulated datasets, CellPath achieves higher Kendall rank correlation scores than CellRank, VeTra (averaged over min_dist values 0.4, 0.5, 0.8), Vdpt, and Slingshot: [evidence span withheld — anchor status: not_found]
- [other] CellPath validation against Figure 5G: [evidence span withheld — anchor status: not_found]
