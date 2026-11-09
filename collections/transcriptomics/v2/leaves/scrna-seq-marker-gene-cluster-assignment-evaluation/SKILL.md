---
name: scrna-seq-marker-gene-cluster-assignment-evaluation
description: Use when you need to quantify whether a clustering solution (e.g., scDCC
  with pairwise constraints vs. unconstrained) improves marker-gene cluster specificity
  in an scRNA-seq dataset, via paired per-gene specificity scores and a one-sided
  Wilcoxon test.
when_to_use_negative:
- The goal is cell type assignment from predefined marker-gene sets (e.g., CellAssign
  or SCINA-style annotation) rather than evaluating cluster specificity of a fitted
  clustering solution.
- You are clustering an unrestricted dataset without a set of uniquely assigned marker
  genes to score specificity against.
- You only have a single clustering per condition and cannot form the ten paired repeats
  required for the paired signed-rank comparison.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDCC
  role: constrained/unconstrained clustering of the human liver scRNA-seq data to
    produce paired cluster assignments
  repo: https://github.com/ttgump/scDCC
- name: Scanpy 1.0.4
  role: loading and handling the scRNA-seq count matrix
- name: PyTorch 1.5.1+cu101
  role: framework in which scDCC.py is executed on the Nvidia Tesla P100
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-22008-3
    title: Model-based deep embedding for constrained clustering analysis of single
      cell RNA-seq data
  source_content_hash: sha256:fe1e308945987553d0a4776424faf620b4901e58995458ee7b30e3ce2680c175
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ttgump/scDCC
  tools:
  - PyTorch 1.5.1+cu101
  - Python
  - Scanpy
  - Scanpy 1.0.4
  - scDCC
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# scrna-seq-marker-gene-cluster-assignment-evaluation

## Summary

Evaluates whether constrained clustering (scDCC) improves cluster specificity of uniquely assigned marker genes in the human liver scRNA-seq dataset by comparing DESeq2 Wald-statistic-based specificity scores with vs. without pairwise constraints using a one-sided Wilcoxon signed-rank test (reported p = 0.00024).

## When to use

Use when you need to quantify whether a clustering solution (e.g., scDCC with pairwise constraints vs. unconstrained) improves marker-gene cluster specificity in an scRNA-seq dataset, via paired per-gene specificity scores and a one-sided Wilcoxon test.

## When NOT to use

- The goal is cell type assignment from predefined marker-gene sets (e.g., CellAssign or SCINA-style annotation) rather than evaluating cluster specificity of a fitted clustering solution.
- You are clustering an unrestricted dataset without a set of uniquely assigned marker genes to score specificity against.
- You only have a single clustering per condition and cannot form the ten paired repeats required for the paired signed-rank comparison.

## Inputs

- human liver scRNA-seq count matrix from the scDCC_data figshare deposit
- 55 uniquely assigned marker genes for the human liver cell types
- ten unconstrained scDCC cluster assignments
- ten pairwise-constrained scDCC cluster assignments (generated via --n_pairwise)

## Outputs

- specificity_wilcoxon_results.csv: per-gene table with 55 rows and averaged specificity-score columns (with and without constraints) plus the one-sided Wilcoxon p-value (expected 0.00024)

## How to apply

Download the scDCC_data deposit from figshare (https://figshare.com/articles/dataset/scDCC_data/21563517) and load the human liver count matrix with Scanpy 1.0.4 in Python 3.6.8. Run scDCC.py (PyTorch 1.5.1+cu101, Nvidia Tesla P100) ten times in unconstrained mode and ten times with pairwise constraints generated via the --n_pairwise parameter, yielding ten paired clustering assignments per condition. For each repeat, compute per-gene DESeq2 Wald-statistic-based cluster-specificity scores for the 55 uniquely assigned marker genes, then average scores across the ten repeats for each condition. Apply a one-sided Wilcoxon signed-rank test to the paired averaged scores (with vs. without constraints) to test whether constraints increase marker-gene specificity, expecting p ≈ 0.00024. Write the 55-row per-gene table of averaged specificity scores plus the p-value to specificity_wilcoxon_results.csv.

## Related tools

- **scDCC** (constrained/unconstrained clustering of the human liver scRNA-seq data to produce paired cluster assignments) — https://github.com/ttgump/scDCC
- **Scanpy 1.0.4** (loading and handling the scRNA-seq count matrix)
- **PyTorch 1.5.1+cu101** (framework in which scDCC.py is executed on the Nvidia Tesla P100)

## Examples

```
python scDCC_pairwise_Human_liver.py --n_pairwise <n>
```

## Evaluation signals

- The one-sided Wilcoxon signed-rank p-value on the paired averaged specificity scores reproduces the reported value of 0.00024.
- The output CSV contains exactly 55 rows (one per uniquely assigned marker gene) with both specificity-score columns populated.
- All ten unconstrained and ten constrained scDCC runs complete and each yields a valid cluster assignment for the human liver cells.
- Averaged DESeq2 Wald-statistic-based specificity scores are computed identically (same gene set, same statistic) for the with- and without-constraints conditions.
- The specificity score trend shows constraints increasing DESeq2-based scores for most of the 55 marker genes.

## Limitations

- Results are specific to the human liver dataset and its 55 uniquely assigned marker genes; generalization to other datasets or marker sets is not established by this evaluation.
- Comparisons depend on stochastic scDCC runs (ten repeats per condition), so reproducibility may vary without fixed random seeds.
- Requires the specific software environment (Python 3.6.8, Scanpy 1.0.4, PyTorch 1.5.1+cu101) and GPU (Nvidia Tesla P100) used in the study.

## Evidence

- [methods] Run scDCC_pairwise_Human_liver.py: "python scDCC_pairwise_Human_liver.py"
- [methods] pairwise constraint generation parameter: "--n_pairwise: number of pairwise constraints want to generate"
- [methods] human liver dataset wrapper: "the wrapper to run scDCC on the human liver dataset (Figure 6)"
- [methods] data availability: "Datasets used in the study is available in: https://figshare.com/articles/dataset/scDCC_data/21563517"
- [abstract] clustering improves performance: "Experiments on various scRNA-seq datasets from thousands to tens of thousands of cells show that scDCC can significantly improve clustering performance"
