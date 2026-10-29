---
name: hold-out-label-sampling
description: Use when ground-truth cell-type labels are available for only a fraction
  of cells in an scRNA-seq dataset and you want to convert them into soft pairwise
  constraints for semi-supervised clustering with scDCC.
when_to_use_negative:
- No partial labels or prior cell-type knowledge exist for the dataset, since the
  constraints must be derived from a labeled hold-out set
- Prior knowledge is limited to marker genes and predefined cell types — the paper
  notes CellAssign/SCINA-style marker-based approaches are a different paradigm than
  soft pairwise constraints
- Hard-constrained clustering is required (e.g., COP-K-means-style guarantees); scDCC
  deliberately treats constraints as soft penalty terms
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: scDCC
  role: Consumes the sampled must-link/cannot-link constraints as soft penalty terms
    added to the deep-embedding clustering loss
  repo: https://github.com/ttgump/scDCC
- name: Scanpy
  role: Preprocesses the count matrix before label splitting and constraint sampling
- name: PyTorch
  role: Framework in which the scDCC model and its constraint-augmented loss are implemented
    (1.5.1+cu101)
  repo: https://github.com/ttgump/scDCC
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-22008-3
    title: Model-based deep embedding for constrained clustering analysis of single
      cell RNA-seq data
  source_content_hash: sha256:c65bcec295280fb6ba923b435a0945632b0f143c4ed3e95583f92a7641334b28
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ttgump/scDCC
  tools:
  - PyTorch
  - Python
  - Scanpy
  - scDCC
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# hold-out-label-sampling

## Summary

Sample must-link and cannot-link pairwise constraints from a small labeled hold-out set (10% of cells) to inject partial domain knowledge into scDCC's constrained clustering loss on scRNA-seq data.

## When to use

Use when ground-truth cell-type labels are available for only a fraction of cells in an scRNA-seq dataset and you want to convert them into soft pairwise constraints for semi-supervised clustering with scDCC.

## When NOT to use

- No partial labels or prior cell-type knowledge exist for the dataset, since the constraints must be derived from a labeled hold-out set
- Prior knowledge is limited to marker genes and predefined cell types — the paper notes CellAssign/SCINA-style marker-based approaches are a different paradigm than soft pairwise constraints
- Hard-constrained clustering is required (e.g., COP-K-means-style guarantees); scDCC deliberately treats constraints as soft penalty terms

## Inputs

- Preprocessed scRNA-seq count matrix (Scanpy-normalized) with ground-truth cell-type labels for a subset of cells
- 10% labeled hold-out split of the cell labels
- Target number of pairwise constraints (--n_pairwise)
- Known cell-type count (--n_clusters) and clustering-loss weight (--gamma)

## Outputs

- Sets of must-link and cannot-link cell pairs sampled from the hold-out labels
- scDCC cluster assignments for all cells
- NMI, CA, and ARI metrics per constraint count, aggregated over ten repeats (e.g., retina_clustering_metrics.csv)

## How to apply

Preprocess the count matrix with Scanpy, then set aside a 10% hold-out subset of the known cell-type labels. From this labeled subset, sample must-link pairs (same cell type) and cannot-link pairs (different cell types) at increasing --n_pairwise values, so clustering quality can be traced as a function of constraint count. Pass the constraints to scDCC (scDCC.py, PyTorch 1.5.1+cu101), where they are added as soft penalty terms — weighted by --gamma — to the clustering loss rather than used as hard constraints, following the paper's design of converting partial prior knowledge into soft pairwise constraints. Evaluate NMI, CA, and ARI between predicted and ground-truth labels over ten repeats per constraint count (as in the Macosko 14,653-cell and Shekhar 27,499-cell retina experiments of Fig. 4).

## Related tools

- **scDCC** (Consumes the sampled must-link/cannot-link constraints as soft penalty terms added to the deep-embedding clustering loss) — https://github.com/ttgump/scDCC
- **Scanpy** (Preprocesses the count matrix before label splitting and constraint sampling)
- **PyTorch** (Framework in which the scDCC model and its constraint-augmented loss are implemented (1.5.1+cu101)) — https://github.com/ttgump/scDCC

## Examples

```
python scDCC.py --n_clusters 15 --gamma 0.1 --n_pairwise 500
```

## Evaluation signals

- NMI, CA, and ARI between predicted and ground-truth labels should increase monotonically (or consistently) as --n_pairwise increases, as observed on the Macosko and Shekhar retina datasets in Fig. 4
- Metrics are stable across ten independent repeats per constraint count; report mean and spread
- Constraints are drawn only from the 10% labeled hold-out, so the remaining 90% of labels serve as an untouched test set
- Number of sampled must-link and cannot-link pairs equals the requested --n_pairwise value
- --n_clusters matches the known cell-type count for each dataset

## Limitations

- Requires a labeled subset; purely unlabeled datasets cannot seed constraints
- Performance depends on --gamma, the weight of the clustering/constraint loss, which must be tuned
- Constraint counts are evaluated per dataset size; results over ten repeats are needed because single runs vary

## Evidence

- [intro] Unlike traditional hard-constrained clustering algorithms (e.g., COP K-means36), we convert (partial) prior knowledge into soft pairwise constraints and add them as additional terms into the loss: "Unlike traditional hard-constrained clustering algorithms (e.g., COP K-means36), we convert (partial) prior knowledge into soft pairwise constraints and add them as additional terms into the loss"
- [methods] Preprocess each count matrix with Scanpy and split labels into a 10% hold-out set from which must-link and cannot-link pairwise constraints are sampled at increasing --n_pairwise values.: [evidence span withheld — anchor status: not_found]
- [methods] --n_pairwise: number of pairwise constraints want to generate: "--n_pairwise: number of pairwise constraints want to generate"
- [results] scDCC showed consistent improvements over baselines as the number of pairwise constraints (built from a 10% labeled hold-out set) increased, with results evaluated over ten repeats.: [evidence span withheld — anchor status: not_found]
- [abstract] Here we report a principled clustering method named scDCC, that integrates domain knowledge into the clustering step.: "Here we report a principled clustering method named scDCC, that integrates domain knowledge into the clustering step."
