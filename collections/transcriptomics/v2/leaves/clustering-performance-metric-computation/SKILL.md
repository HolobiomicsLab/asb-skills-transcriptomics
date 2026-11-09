---
name: clustering-performance-metric-computation
description: Use when evaluating scRNA-seq clustering outputs against known cell-type
  labels, especially when measuring how clustering quality (NMI, CA, ARI) changes
  as the number of pairwise must-link/cannot-link constraints increases.
when_to_use_negative:
- No ground-truth cell-type labels exist, so NMI/CA/ARI against reference labels cannot
  be computed (e.g., purely exploratory clustering)
- The goal is estimating the number of clusters rather than scoring a fixed-K clustering
  against known labels
- Evaluating marker-based cell-type assignment methods (e.g., CellAssign, SCINA) where
  prior knowledge is limited to marker genes and predefined cell types rather than
  clustering quality
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scDCC
  role: Constrained clustering model whose predicted labels are scored with NMI, CA,
    and ARI; trained with --n_clusters set to known cell-type count and --gamma as
    clustering-loss weight
  repo: https://github.com/ttgump/scDCC
- name: Python 3.6.8
  role: Runtime for implementing the metric computations and repeated training/evaluation
    loop
- name: PyTorch 1.5.1+cu101
  role: Deep-learning framework backing the scDCC training whose repeated runs produce
    the label vectors to score
- name: Scanpy 1.0.4
  role: Preprocessing of the Macosko and Shekhar retina count matrices prior to clustering
    and metric evaluation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-22008-3
    title: Model-based deep embedding for constrained clustering analysis of single
      cell RNA-seq data
  source_content_hash: sha256:afaebac453391e31d8eb8fec200c43c36406834a8dad4f9453788ca2a4a8ce76
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ttgump/scDCC
  tools:
  - PyTorch 1.5.1+cu101
  - Python
  - Python 3.6.8
  - Scanpy
  - Scanpy 1.0.4
  - scDCC
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# clustering-performance-metric-computation

## Summary

Compute normalized mutual information (NMI), clustering accuracy (CA), and adjusted Rand index (ARI) between predicted scRNA-seq cluster labels and ground-truth cell-type labels, repeated over multiple runs to assess whether constrained clustering (e.g., scDCC with pairwise constraints) genuinely improves over baselines.

## When to use

Use when evaluating scRNA-seq clustering outputs against known cell-type labels, especially when measuring how clustering quality (NMI, CA, ARI) changes as the number of pairwise must-link/cannot-link constraints increases.

## When NOT to use

- No ground-truth cell-type labels exist, so NMI/CA/ARI against reference labels cannot be computed (e.g., purely exploratory clustering)
- The goal is estimating the number of clusters rather than scoring a fixed-K clustering against known labels
- Evaluating marker-based cell-type assignment methods (e.g., CellAssign, SCINA) where prior knowledge is limited to marker genes and predefined cell types rather than clustering quality

## Inputs

- Predicted cluster label vector for all cells from a trained clustering model (e.g., scDCC PyTorch output)
- Ground-truth cell-type label vector for all cells
- Repeat-level predicted/true label pairs across ten runs per constraint setting
- Constraint count setting (--n_pairwise) and dataset identifier for aggregation

## Outputs

- Per-repeat NMI, CA, and ARI values between predicted and ground-truth labels
- Aggregated mean and spread of NMI, CA, ARI across ten repeats for each constraint count and dataset (e.g., retina_clustering_metrics.csv matching Figure 4 / Figure S10 layout)

## How to apply

For each trained clustering model (e.g., scDCC trained on the Macosko 14,653-cell and Shekhar 27,499-cell mouse retina count matrices preprocessed with Scanpy), compare the predicted cluster assignments to ground-truth labels by computing NMI, CA, and ARI. Because constraint sampling and stochastic training introduce variability, repeat training and evaluation ten times per constraint count (constraints drawn from a 10% labeled hold-out set at increasing --n_pairwise values), then aggregate the mean and spread of the three metrics across repeats. Write the aggregated per-constraint-count, per-dataset metrics to a table such as retina_clustering_metrics.csv, mirroring the layout of Figure 4 / Figure S10. Improvement is judged by consistent monotonic gains in NMI/ARI (and decreasing CA) over baseline (unsupervised) methods as constraints increase.

## Related tools

- **scDCC** (Constrained clustering model whose predicted labels are scored with NMI, CA, and ARI; trained with --n_clusters set to known cell-type count and --gamma as clustering-loss weight) — https://github.com/ttgump/scDCC
- **Python 3.6.8** (Runtime for implementing the metric computations and repeated training/evaluation loop)
- **PyTorch 1.5.1+cu101** (Deep-learning framework backing the scDCC training whose repeated runs produce the label vectors to score)
- **Scanpy 1.0.4** (Preprocessing of the Macosko and Shekhar retina count matrices prior to clustering and metric evaluation)

## Examples

```
python scDCC.py --dataset retina --n_clusters <K> --gamma <g> --n_pairwise <n>  # then compute NMI, CA, ARI vs ground truth over 10 repeats
```

## Evaluation signals

- NMI and ARI increase monotonically (within repeat spread) as the number of pairwise constraints grows on both Macosko (14,653 cells) and Shekhar (27,499 cells) datasets
- CA decreases correspondingly as constraints increase, consistent with improved cell-type assignment
- Metrics are stable across ten repeats per constraint count (report mean and spread rather than single-run values)
- Constrained scDCC results consistently outperform unsupervised baselines at matched cluster counts (--n_clusters equal to known cell-type count)
- Output CSV matches the Figure 4 / Figure S10 layout (rows = constraint counts, columns = metrics per dataset)

## Limitations

- Results depend on the 10% labeled hold-out set from which must-link and cannot-link constraints are sampled; different samplings change metric trajectories
- Single training runs are stochastic; evaluating without ten repeats risks drawing conclusions from noise
- Metrics require ground-truth labels and a known cell-type count (--n_clusters); they cannot be applied to unlabeled exploratory datasets

## Evidence

- [abstract] Experiments on various scRNA-seq datasets from thousands to tens of thousands of cells show that scDCC can significantly improve clustering performance: "Experiments on various scRNA-seq datasets from thousands to tens of thousands of cells show that scDCC can significantly improve clustering performance"
- [methods] Number of pairwise constraints: "--n_pairwise: number of pairwise constraints want to generate"
- [results] Workflow metric specification: [evidence span withheld — anchor status: not_found]
- [results] Retina finding: [evidence span withheld — anchor status: not_found]
- [methods] Framework versions: "pytorch -- 1.5.1+cu101 (https://pytorch.org)"
