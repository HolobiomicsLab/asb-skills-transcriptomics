---
name: clustering-performance-benchmarking
description: Use when you need to quantify whether adding noisy pairwise constraints
  improves scRNA-seq clustering — e.g., testing robustness of scDCC to 5% or 10% mislabeled
  must-link/cannot-link constraints versus an unsupervised baseline.
when_to_use_negative:
- The goal is to evaluate constraint-free unsupervised clustering only — no pairwise
  constraints will be generated or corrupted, so the benchmarking design adds nothing
- Datasets are not small scRNA-seq benchmarks with known labels or reference clusters,
  making clustering performance scores uninterpretable
- Prior knowledge is limited to marker genes and predefined cell types (e.g., CellAssign/SCINA-style
  settings), which the article treats as a different prior-integration paradigm than
  soft pairwise constraints
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDCC
  role: Constrained clustering method under test; pairwise constraints are added as
    soft penalty terms to its loss function
  repo: https://github.com/ttgump/scDCC
- name: PyTorch
  role: Deep-learning framework in which scDCC_pairwise.py is implemented and run
    for each perturbation condition
  repo: https://github.com/ttgump/scDCC/blob/master/scDCC.py
- name: Scanpy
  role: Preprocessing of the scRNA-seq benchmark datasets before constraint generation
    and clustering
- name: Python
  role: Environment for running the scDCC wrappers, constraint corruption, and metric
    compilation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-22008-3
    title: Model-based deep embedding for constrained clustering analysis of single
      cell RNA-seq data
  source_content_hash: sha256:65aca641c61019afa3984731d2f789e99412ee5a34c46a4f3bb2723898f39693
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# clustering-performance-benchmarking

## Summary

Benchmark a constrained clustering method (scDCC) against its 0-constraint unsupervised baseline by systematically varying the number and error rate of pairwise must-link/cannot-link constraints and compiling clustering metrics into a comparison table.

## When to use

Use when you need to quantify whether adding noisy pairwise constraints improves scRNA-seq clustering — e.g., testing robustness of scDCC to 5% or 10% mislabeled must-link/cannot-link constraints versus an unsupervised baseline.

## When NOT to use

- The goal is to evaluate constraint-free unsupervised clustering only — no pairwise constraints will be generated or corrupted, so the benchmarking design adds nothing
- Datasets are not small scRNA-seq benchmarks with known labels or reference clusters, making clustering performance scores uninterpretable
- Prior knowledge is limited to marker genes and predefined cell types (e.g., CellAssign/SCINA-style settings), which the article treats as a different prior-integration paradigm than soft pairwise constraints

## Inputs

- Small scRNA-seq benchmark datasets from figshare (scDCC_data, https://figshare.com/articles/dataset/scDCC_data/21563517), e.g. mouse bladder cells
- Preprocessed expression matrices (Scanpy-normalized)
- Pairwise constraint lists (must-link and cannot-link pairs) generated via --n_pairwise
- 0-constraint unsupervised baseline clustering runs for comparison

## Outputs

- robustness_metrics.csv with columns: dataset, constraint count, error rate (0%/5%/10%), performance score
- Per-run clustering performance metrics for each dataset, constraint count, and error rate
- Robustness comparison showing perturbed vs. baseline performance (Fig. S9-style)

## How to apply

Download the four small scRNA-seq benchmark datasets from the figshare deposit (https://figshare.com/articles/dataset/scDCC_data/21563517) and preprocess with Scanpy 1.0.4. Generate pairwise must-link/cannot-link constraints over a range of counts using the --n_pairwise parameter of the scDCC wrapper, then corrupt a fixed fraction (5%, then 10%) by flipping must-link to cannot-link and vice versa. Rerun scDCC_pairwise.py (PyTorch 1.5.1+cu101) with --gamma (clustering loss weight), --ml_weight (must-link loss weight), and --cl_weight (cannot-link loss weight) held at defaults. Compute clustering performance metrics for each perturbed run, compare against the corresponding 0-constraint baseline run, and compile results into robustness_metrics.csv with columns dataset, constraint count, error rate (0%, 5%, 10%), and performance score.

## Related tools

- **scDCC** (Constrained clustering method under test; pairwise constraints are added as soft penalty terms to its loss function) — https://github.com/ttgump/scDCC
- **PyTorch** (Deep-learning framework in which scDCC_pairwise.py is implemented and run for each perturbation condition) — https://github.com/ttgump/scDCC/blob/master/scDCC.py
- **Scanpy** (Preprocessing of the scRNA-seq benchmark datasets before constraint generation and clustering)
- **Python** (Environment for running the scDCC wrappers, constraint corruption, and metric compilation)

## Examples

```
python scDCC_pairwise.py --n_pairwise 200 --gamma 1 --ml_weight 1 --cl_weight 1  # run per dataset at 0%, 5%, and 10% flipped constraints
```

## Evaluation signals

- At 5% constraint error, scDCC performance exceeds the 0-constraint baseline on all four benchmark datasets (robustness confirmed)
- At 10% constraint error, degradation appears on some datasets (e.g., mouse bladder cells underperforms the baseline) — the table should show this breakdown
- robustness_metrics.csv has exactly the columns dataset, constraint count, error rate (0%, 5%, 10%), and performance score, with rows spanning every constraint count and dataset
- Corrupted constraints are produced by flipping must-link to cannot-link and vice versa at the stated fraction (5% then 10%) for every constraint count and dataset
- gamma, ml_weight, and cl_weight remain at their default settings across all runs so differences are attributable to constraint noise only

## Limitations

- Robustness holds only up to ~5% erroneous constraints; at a 10% error rate scDCC begins to underperform the 0-constraint baseline on some datasets (e.g., mouse bladder cells)
- Results are established on four small scRNA-seq benchmark datasets; the article's larger experiments (10X CITE PBMC, human liver) were run separately and are not the basis of this robustness claim
- Constraint corruption is simulated by label flipping, which may not capture all realistic forms of prior-knowledge error

## Evidence

- [supplementary] With 5% erroneous pairwise constraints, scDCC still achieves better clustering results than the 0-constraint baseline on all datasets, showing robustness to noisy constraints: [evidence span withheld — anchor status: not_found]
- [supplementary] at 10% error rate scDCC underperforms on some datasets: [evidence span withheld — anchor status: not_found]
- [methods] pairwise constraints generated via n_pairwise parameter: "--n_pairwise: number of pairwise constraints want to generate"
- [intro] constraints added as loss terms rather than hard constraints: "we convert (partial) prior knowledge into soft pairwise constraints and add them as additional terms into the loss"
- [methods] datasets available on figshare: "Datasets used in the study is available in: https://figshare.com/articles/dataset/scDCC_data/21563517"
- [abstract] scDCC improves clustering across dataset sizes: "Experiments on various scRNA-seq datasets from thousands to tens of thousands of cells show that scDCC can significantly improve clustering performance"
