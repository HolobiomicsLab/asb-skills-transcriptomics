---
name: constraint-perturbation-robustness-analysis
description: Use when a clustering pipeline consumes prior pairwise constraints and
  you need to verify that noisy or partially incorrect domain knowledge does not hurt
  performance relative to running the method without constraints.
when_to_use_negative:
- The clustering method being tested does not accept pairwise constraints — there
  is nothing to perturb and the unsupervised baseline alone suffices.
- The goal is to assess sensitivity to data corruption (e.g., dropout or gene-count
  noise) rather than corruption of the supervision/constraints themselves.
- Ground-truth cell type labels are unavailable for the datasets, so clustering performance
  scores cannot be computed for the comparison.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDCC
  role: Constrained-clustering method whose robustness to erroneous pairwise constraints
    is being probed; rerun with corrupted constraints via scDCC_pairwise.py
  repo: https://github.com/ttgump/scDCC
- name: Scanpy
  role: Preprocessing of the scRNA-seq benchmark datasets before constraint generation
    and clustering
- name: PyTorch
  role: Backend used to run scDCC_pairwise.py for each perturbed and baseline run
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-22008-3
    title: Model-based deep embedding for constrained clustering analysis of single
      cell RNA-seq data
  source_content_hash: sha256:048cb2cecc6d3d29aeffe5d7e46e4e557aae7c91b0fbbf9724c91c9cb25bf765
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# constraint-perturbation-robustness-analysis

## Summary

A robustness-testing skill that quantifies how a constrained-clustering method (e.g., scDCC) degrades when a controlled fraction of pairwise must-link/cannot-link constraints is deliberately mislabeled. It anchors the assessment against a 0-constraint unsupervised baseline across multiple constraint counts and datasets.

## When to use

Use when a clustering pipeline consumes prior pairwise constraints and you need to verify that noisy or partially incorrect domain knowledge does not hurt performance relative to running the method without constraints.

## When NOT to use

- The clustering method being tested does not accept pairwise constraints — there is nothing to perturb and the unsupervised baseline alone suffices.
- The goal is to assess sensitivity to data corruption (e.g., dropout or gene-count noise) rather than corruption of the supervision/constraints themselves.
- Ground-truth cell type labels are unavailable for the datasets, so clustering performance scores cannot be computed for the comparison.

## Inputs

- Preprocessed small scRNA-seq benchmark datasets (e.g., from the scDCC_data figshare deposit: https://figshare.com/articles/dataset/scDCC_data/21563517)
- Generated pairwise must-link/cannot-link constraint sets over a range of constraint counts (via --n_pairwise)
- Corrupted constraint sets with 5% and 10% of must-link/cannot-link labels flipped
- 0-constraint unsupervised baseline clustering results for each dataset

## Outputs

- robustness_metrics.csv with columns: dataset, constraint count, error rate (0%, 5%, 10%), performance score
- Per-run clustering performance metrics (e.g., ARI/NMI) for each dataset × constraint count × error-rate combination
- Comparison of perturbed runs against the 0-constraint baseline per dataset

## How to apply

Generate pairwise must-link/cannot-link constraints for each dataset over a range of constraint counts using the --n_pairwise parameter of the scDCC wrapper, after preprocessing the scRNA-seq data with Scanpy. Corrupt a fixed fraction of the constraints by flipping must-link to cannot-link and vice versa — first 5%, then 10% — and rerun scDCC (scDCC_pairwise.py, PyTorch) keeping --gamma (clustering loss weight), --ml_weight (must-link loss weight), and --cl_weight (cannot-link loss weight) at their default settings for every constraint count and dataset. Compute clustering performance metrics for each perturbed run and compare them against the corresponding 0-constraint unsupervised baseline run on the same dataset. Compile all metrics into robustness_metrics.csv with columns for dataset, constraint count, error rate (0%, 5%, 10%), and performance score. The validation criterion: at 5% error the constrained runs should still beat the 0-constraint baseline on all datasets, while degradation may appear at 10% error on some datasets (e.g., mouse bladder cells).

## Related tools

- **scDCC** (Constrained-clustering method whose robustness to erroneous pairwise constraints is being probed; rerun with corrupted constraints via scDCC_pairwise.py) — https://github.com/ttgump/scDCC
- **Scanpy** (Preprocessing of the scRNA-seq benchmark datasets before constraint generation and clustering)
- **PyTorch** (Backend used to run scDCC_pairwise.py for each perturbed and baseline run)

## Examples

```
python scDCC_pairwise_CITE_PBMC.py
```

## Evaluation signals

- At 5% constraint error, scDCC clustering performance exceeds the 0-constraint baseline on all four benchmark datasets.
- At 10% constraint error, degradation relative to the baseline appears on at least some datasets (e.g., mouse bladder cells) — a lower-than-5% performance is expected there, not a constant improvement.
- robustness_metrics.csv has exactly the schema: dataset, constraint count, error rate (0%, 5%, 10%), performance score, with rows covering every dataset × constraint-count × error-rate combination.
- The corruption step strictly flips labels (must-link → cannot-link and vice versa) rather than dropping constraints, so the number of constraints per run is unchanged across error rates.
- --gamma, --ml_weight, and --cl_weight remain at default values across all perturbed runs, isolating the error rate as the only manipulated variable.

## Limitations

- Robustness holds only up to ~5% erroneous constraints; at a 10% error rate scDCC begins to underperform the 0-constraint baseline on some datasets (e.g., mouse bladder cells).
- Conclusions were established on small scRNA-seq benchmark datasets; extrapolation to very large datasets (tens of thousands of cells) is not verified by this perturbation experiment.
- The corruption model is symmetric label flipping of pairwise constraints; other noise models (e.g., asymmetric mislabeling or constraint dropout) are not covered.

## Evidence

- [supplementary] With 5% erroneous pairwise constraints, scDCC still achieves better clustering results than the 0-constraint baseline on all datasets, showing robustness to noisy constraints: [evidence span withheld — anchor status: not_found]
- [methods] n_pairwise parameter quote: "--n_pairwise: number of pairwise constraints want to generate"
- [readme] scDCC README context quote: "Clustering is a critical step in single cell-based studies."
- [methods] Wrapper invocation quote: "python scDCC_pairwise_CITE_PBMC.py"
