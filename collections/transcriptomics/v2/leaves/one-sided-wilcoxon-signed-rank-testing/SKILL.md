---
name: one-sided-wilcoxon-signed-rank-testing
description: Use when you have paired per-gene scores (e.g., marker-gene specificity
  with vs. without constraints) averaged over repeated clusterings and need a one-sided
  test of whether the constrained condition improves scores.
when_to_use_negative:
- When the two conditions being compared are unpaired (e.g., independent genes or
  independent runs rather than paired per-gene score pairs)
- When scores are not averaged across the same number of repeats (ten scDCC runs)
  for both conditions, breaking the pairing assumption
- When the goal is unsupervised clustering performance rather than testing whether
  constraints improve marker-gene cluster specificity
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3170
tools:
- name: scDCC
  role: Produces the ten paired unconstrained and pairwise-constrained cluster assignments
    whose marker-gene specificity scores are compared by the Wilcoxon test
  repo: https://github.com/ttgump/scDCC
- name: Scanpy 1.0.4
  role: Loads the human liver count matrix from the scDCC_data figshare deposit
- name: Python 3.6.8
  role: Runtime environment for computing DESeq2 Wald-statistic-based specificity
    scores and running the Wilcoxon signed-rank test
- name: PyTorch 1.5.1+cu101
  role: Runtime for running scDCC (scDCC.py) on an Nvidia Tesla P100 to generate the
    paired cluster assignments
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-22008-3
    title: Model-based deep embedding for constrained clustering analysis of single
      cell RNA-seq data
  source_content_hash: sha256:9182827ec7963a8472b2298ea60e2be628a6d33186bf461776619b871fb3e524
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# one-sided-wilcoxon-signed-rank-testing

## Summary

A paired non-parametric significance test used in the scDCC study to verify that adding soft pairwise constraints to scDCC clustering increases DESeq2-based cluster-specificity scores for 55 uniquely assigned marker genes in the human liver dataset (reported p-value = 0.00024).

## When to use

Use when you have paired per-gene scores (e.g., marker-gene specificity with vs. without constraints) averaged over repeated clusterings and need a one-sided test of whether the constrained condition improves scores.

## When NOT to use

- When the two conditions being compared are unpaired (e.g., independent genes or independent runs rather than paired per-gene score pairs)
- When scores are not averaged across the same number of repeats (ten scDCC runs) for both conditions, breaking the pairing assumption
- When the goal is unsupervised clustering performance rather than testing whether constraints improve marker-gene cluster specificity

## Inputs

- Count matrix from the scDCC_data figshare deposit (https://figshare.com/articles/dataset/scDCC_data/21563517) for the human liver dataset
- Ten independent unconstrained scDCC cluster assignments of the human liver cells
- Ten constrained scDCC cluster assignments generated via --n_pairwise
- List of 55 uniquely assigned marker genes for the human liver cell types

## Outputs

- Per-gene table of averaged DESeq2-based specificity scores (with and without constraints) for 55 marker genes
- One-sided Wilcoxon signed-rank p-value (expected 0.00024) written to specificity_wilcoxon_results.csv

## How to apply

Run scDCC on the human liver scRNA-seq count matrix (loaded with Scanpy 1.0.4 in Python 3.6.8) ten times in unconstrained mode and ten times with pairwise constraints generated via the --n_pairwise parameter, using PyTorch 1.5.1+cu101 on an Nvidia Tesla P100. For each repeat, compute per-gene DESeq2 Wald-statistic-based cluster-specificity scores for the 55 uniquely assigned marker genes under each clustering, then average the scores across the ten repeats for the with- and without-constraints conditions. Apply a one-sided Wilcoxon signed-rank test to the paired averaged scores to test whether constraints increase marker-gene specificity. Write the per-gene table (both score columns) plus the p-value to specificity_wilcoxon_results.csv; the test should reproduce the reported p-value of 0.00024.

## Related tools

- **scDCC** (Produces the ten paired unconstrained and pairwise-constrained cluster assignments whose marker-gene specificity scores are compared by the Wilcoxon test) — https://github.com/ttgump/scDCC
- **Scanpy 1.0.4** (Loads the human liver count matrix from the scDCC_data figshare deposit)
- **Python 3.6.8** (Runtime environment for computing DESeq2 Wald-statistic-based specificity scores and running the Wilcoxon signed-rank test)
- **PyTorch 1.5.1+cu101** (Runtime for running scDCC (scDCC.py) on an Nvidia Tesla P100 to generate the paired cluster assignments)

## Examples

```
python scDCC_pairwise_Human_liver.py
```

## Evaluation signals

- The one-sided Wilcoxon signed-rank p-value on the paired averaged scores reproduces the reported 0.00024
- The output table specificity_wilcoxon_results.csv contains exactly 55 rows (one per uniquely assigned marker gene) with both specificity-score columns populated
- Each condition's scores are the average of exactly ten scDCC repeats (ten unconstrained, ten with --n_pairwise constraints)
- Most of the 55 marker genes show a higher constrained than unconstrained specificity score, consistent with the reported significant improvement

## Limitations

- The result is specific to the human liver dataset and its 55 uniquely assigned marker genes; it does not automatically generalize to other datasets or marker panels
- Purely unsupervised clustering may not produce biologically interpretable clusters, so the paired comparison depends on the constraint-generation settings used for scDCC
- The scDCC runs require the original environment (Python 3.6.8, PyTorch 1.5.1+cu101, Nvidia Tesla P100); different hardware/software may change cluster assignments and the test outcome

## Evidence

- [methods] human liver dataset: "the wrapper to run scDCC on the human liver dataset (Figure 6)"
- [methods] scDCC_data (figshare, https://figshare.com/articles/dataset/scDCC_data/21563517): "Datasets used in the study is available in: https://figshare.com/articles/dataset/scDCC_data/21563517"
- [methods] Generate pairwise constraints: "--n_pairwise: number of pairwise constraints want to generate"
- [abstract] Motivation for constrained clustering: "purely unsupervised clustering methods may not produce biologically interpretable clusters, which complicates cell type assignment"
