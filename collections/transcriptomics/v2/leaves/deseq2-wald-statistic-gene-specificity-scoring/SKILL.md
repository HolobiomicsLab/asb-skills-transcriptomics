---
name: deseq2-wald-statistic-gene-specificity-scoring
description: Use when you need to quantify how specifically a set of marker genes
  (e.g., the 55 uniquely assigned human liver marker genes) maps to clusters produced
  by two or more alternative clustering solutions, and to test whether one solution
  yields higher marker-gene specificity.
when_to_use_negative:
- You have no predefined marker gene set and want a purely data-driven differential
  expression ranking rather than a specificity score for known markers
- The clustering solutions being compared are not repeat-paired over the same marker
  genes, so a one-sided Wilcoxon signed-rank test on paired scores is inappropriate
- The goal is cell-type annotation from scratch rather than benchmarking marker-gene
  specificity of existing cluster assignments
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scDCC
  role: Produces the two clustering solutions (unconstrained and pairwise-constrained
    via --n_pairwise) whose marker-gene specificity is scored and compared
  repo: https://github.com/ttgump/scDCC
- name: Scanpy 1.0.4
  role: Loads and preprocesses the human liver count matrix from the figshare deposit
- name: Python 3.6.8
  role: Runtime environment for the scoring and Wilcoxon-testing workflow
- name: Nvidia Tesla P100
  role: GPU used to run scDCC (PyTorch 1.5.1+cu101) for the ten repeats per condition
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-22008-3
    title: Model-based deep embedding for constrained clustering analysis of single
      cell RNA-seq data
  source_content_hash: sha256:397140f9ef6050f2b9ab15757ec99596ae24ac3fe8f8294b142db803e7044b94
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ttgump/scDCC
  tools:
  - Nvidia Tesla P100
  - Python
  - Python 3.6.8
  - Scanpy
  - Scanpy 1.0.4
  - scDCC
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# deseq2-wald-statistic-gene-specificity-scoring

## Summary

Score each marker gene's cluster specificity using DESeq2 Wald-statistics computed per cluster, then average scores across repeated clusterings to compare alternative cluster assignments (e.g., scDCC with vs. without pairwise constraints) on the human liver dataset.

## When to use

Use when you need to quantify how specifically a set of marker genes (e.g., the 55 uniquely assigned human liver marker genes) maps to clusters produced by two or more alternative clustering solutions, and to test whether one solution yields higher marker-gene specificity.

## When NOT to use

- You have no predefined marker gene set and want a purely data-driven differential expression ranking rather than a specificity score for known markers
- The clustering solutions being compared are not repeat-paired over the same marker genes, so a one-sided Wilcoxon signed-rank test on paired scores is inappropriate
- The goal is cell-type annotation from scratch rather than benchmarking marker-gene specificity of existing cluster assignments

## Inputs

- scRNA-seq count matrix for the human liver dataset (scDCC_data figshare deposit, https://figshare.com/articles/dataset/scDCC_data/21563517)
- Ten unconstrained scDCC cluster assignments (repeat-level cluster labels per cell)
- Ten constrained scDCC cluster assignments generated via the --n_pairwise parameter
- List of the 55 uniquely assigned marker genes for the human liver cell types

## Outputs

- Per-gene table of averaged DESeq2-based cluster-specificity scores (with vs. without constraints), 55 rows, saved as specificity_wilcoxon_results.csv
- One-sided Wilcoxon signed-rank p-value comparing paired averaged specificity scores (expected 0.00024)

## How to apply

Load the human liver scRNA-seq count matrix from the figshare deposit (scDCC_data) with Scanpy 1.0.4 in Python 3.6.8. For each of ten independent runs, compute per-gene DESeq2 Wald-statistic-based cluster-specificity scores for the 55 uniquely assigned marker genes under each clustering (scDCC unconstrained vs. scDCC with --n_pairwise-generated constraints). Average the per-gene scores across the ten repeats within each condition. Apply a one-sided Wilcoxon signed-rank test on the paired averaged scores to test whether constraints increase marker-gene specificity; the article reports p = 0.00024. Write the 55-row per-gene table (with- and without-constraints columns) plus the p-value to specificity_wilcoxon_results.csv.

## Related tools

- **scDCC** (Produces the two clustering solutions (unconstrained and pairwise-constrained via --n_pairwise) whose marker-gene specificity is scored and compared) — https://github.com/ttgump/scDCC
- **Scanpy 1.0.4** (Loads and preprocesses the human liver count matrix from the figshare deposit)
- **Python 3.6.8** (Runtime environment for the scoring and Wilcoxon-testing workflow)
- **Nvidia Tesla P100** (GPU used to run scDCC (PyTorch 1.5.1+cu101) for the ten repeats per condition)

## Examples

```
python scDCC_pairwise_Human_liver.py --n_pairwise <N>  # then score 55 marker genes via DESeq2 Wald stats per clustering and run scipy.stats.wilcoxon(scores_with, scores_without, alternative='greater')
```

## Evaluation signals

- The computed one-sided Wilcoxon p-value on paired averaged scores reproduces the reported value of 0.00024
- The output table specificity_wilcoxon_results.csv contains exactly 55 rows (one per uniquely assigned marker gene)
- The table contains two specificity-score columns (with constraints and without constraints) with no missing values
- Averaged scores are computed over ten repeats per condition (ten unconstrained and ten constrained scDCC runs)
- Most of the 55 marker genes show higher DESeq2-based specificity scores under constrained clustering, consistent with the reported finding

## Limitations

- Scores depend on repeat-to-repeat variability of scDCC clusterings, hence the need to average over ten repeats per condition
- The validation target (p = 0.00024) is specific to the human liver dataset and the 55 uniquely assigned marker genes; other datasets or gene sets will not reproduce it
- DESeq2 Wald-statistic-based specificity assumes the count-model framework and the pre-defined marker set; purely unsupervised clusterings may not produce biologically interpretable clusters

## Evidence

- [methods] Human liver dataset used for specificity scoring: "the wrapper to run scDCC on the human liver dataset (Figure 6)"
- [methods] Data source for the count matrix: "Datasets used in the study is available in: https://figshare.com/articles/dataset/scDCC_data/21563517"
- [methods] Constraint generation parameter: "--n_pairwise: number of pairwise constraints want to generate"
- [abstract] Rationale for evaluating marker-gene specificity of clusters: "purely unsupervised clustering methods may not produce biologically interpretable clusters, which complicates cell type assignment"
- [readme] Tool scope: clustering as critical step with domain knowledge: "Clustering is a critical step in single cell-based studies. Most existing methods support unsupervised clustering without the a priori exploitation of any domain knowledge."
