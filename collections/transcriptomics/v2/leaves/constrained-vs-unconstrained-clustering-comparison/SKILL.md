---
name: constrained-vs-unconstrained-clustering-comparison
description: Use when evaluating whether integrating partial prior knowledge as soft
  pairwise constraints into a scRNA-seq deep clustering model improves clustering
  or marker-gene specificity versus an unsupervised baseline.
when_to_use_negative:
- When no partial prior knowledge or marker information is available to convert into
  pairwise constraints — scDCC's advantage over unsupervised clustering relies on
  such constraints.
- When the goal is hard-constrained clustering — the article explicitly converts prior
  knowledge into soft pairwise constraints added to the loss, unlike hard-constrained
  algorithms such as COP K-means.
- When the input is not scRNA-seq count data with dropout events and high dimensionality,
  the motivating setting for this comparison.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scDCC
  role: Constrained and unconstrained deep clustering of scRNA-seq data; run with
    and without pairwise constraints to produce the compared cluster assignments
  repo: https://github.com/ttgump/scDCC
- name: Scanpy 1.0.4
  role: Loading and preprocessing the scRNA-seq count matrices from the figshare deposit
- name: PyTorch 1.5.1+cu101
  role: Deep learning framework in which scDCC.py is executed (Nvidia Tesla P100)
- name: Python 3.6.8
  role: Runtime environment for running the scDCC wrappers and analysis scripts
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-22008-3
    title: Model-based deep embedding for constrained clustering analysis of single
      cell RNA-seq data
  source_content_hash: sha256:7788a681140d3fb71a2ed38b17072a0d2e143244f0b8cea4b8261fd180761149
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# constrained-vs-unconstrained-clustering-comparison

## Summary

A benchmarking skill for scDCC (Single Cell Deep Constrained Clustering) that compares cluster quality when soft pairwise constraints are added versus purely unsupervised clustering, using repeated runs and statistical tests (e.g., one-sided Wilcoxon signed-rank on marker-gene specificity scores) to quantify whether prior knowledge improves clustering.

## When to use

Use when evaluating whether integrating partial prior knowledge as soft pairwise constraints into a scRNA-seq deep clustering model improves clustering or marker-gene specificity versus an unsupervised baseline.

## When NOT to use

- When no partial prior knowledge or marker information is available to convert into pairwise constraints — scDCC's advantage over unsupervised clustering relies on such constraints.
- When the goal is hard-constrained clustering — the article explicitly converts prior knowledge into soft pairwise constraints added to the loss, unlike hard-constrained algorithms such as COP K-means.
- When the input is not scRNA-seq count data with dropout events and high dimensionality, the motivating setting for this comparison.

## Inputs

- scRNA-seq count matrix from the scDCC_data figshare deposit (e.g., human liver dataset or 10X CITE PBMC dataset)
- Set of known marker genes or partial domain knowledge convertible to pairwise constraints
- Number of pairwise constraints to generate (--n_pairwise parameter)

## Outputs

- Ten unconstrained and ten constrained cluster assignments
- Per-gene averaged cluster-specificity scores (with and without constraints), e.g., specificity_wilcoxon_results.csv with 55 rows and both score columns
- One-sided Wilcoxon signed-rank p-value comparing paired scores (expected 0.00024 for the human liver marker-gene analysis)

## How to apply

Download the study data from the figshare deposit (scDCC_data, https://figshare.com/articles/dataset/scDCC_data/21563517) and load count matrices with Scanpy 1.0.4 in Python 3.6.8. Run scDCC (scDCC.py, PyTorch 1.5.1+cu101 on an Nvidia Tesla P100) in unconstrained mode ten times for independent cluster assignments, then run scDCC with pairwise constraints ten times, generating constraints via the --n_pairwise parameter. For the human liver dataset this is done via scDCC_pairwise_Human_liver.py (analogously scDCC_pairwise_CITE_PBMC.py for the 10X CITE PBMC dataset). Compare the two conditions with an appropriate paired statistic: for example, compute per-gene DESeq2 Wald-statistic-based cluster-specificity scores for the 55 uniquely assigned marker genes, average across the ten repeats for each condition, and apply a one-sided Wilcoxon signed-rank test to the paired scores. The rationale is that converting partial prior knowledge into soft pairwise constraints added to the loss function should produce more biologically meaningful clusters than purely unsupervised clustering.

## Related tools

- **scDCC** (Constrained and unconstrained deep clustering of scRNA-seq data; run with and without pairwise constraints to produce the compared cluster assignments) — https://github.com/ttgump/scDCC
- **Scanpy 1.0.4** (Loading and preprocessing the scRNA-seq count matrices from the figshare deposit)
- **PyTorch 1.5.1+cu101** (Deep learning framework in which scDCC.py is executed (Nvidia Tesla P100))
- **Python 3.6.8** (Runtime environment for running the scDCC wrappers and analysis scripts)

## Examples

```
python scDCC_pairwise_Human_liver.py --n_pairwise <N>
```

## Evaluation signals

- The one-sided Wilcoxon signed-rank p-value on paired averaged specificity scores reproduces the reported value of 0.00024 for the human liver dataset.
- The output specificity table contains exactly 55 rows (the 55 uniquely assigned marker genes) with both with-constraints and without-constraints score columns.
- Each condition (with and without constraints) is represented by ten independent clustering runs whose scores are averaged before testing.
- For most of the 55 marker genes, the DESeq2-based specificity score under constrained clustering exceeds the unconstrained score.
- Constraints are generated through the --n_pairwise parameter of scDCC, matching the paper's soft pairwise-constraint design.

## Limitations

- Prior-knowledge methods in this space, and constraint quality, depend on available marker genes and predefined cell types; the article notes such methods 'are developed in the context of the marker genes and lack the flexibility to integrate other kinds of prior information'.
- Purely unsupervised clustering may fail to produce biologically interpretable clusters on high-dimensional, dropout-heavy scRNA-seq data, so unconstrained baselines can be weak comparators.
- Discussion-section content is absent from the provided document, so broader caveats about the comparison could not be extracted.

## Evidence

- [intro] Unlike traditional hard-constrained clustering algorithms (e.g., COP K-means36), we convert (partial) prior knowledge into soft pairwise constraints and add them as additional terms into the loss: "Unlike traditional hard-constrained clustering algorithms (e.g., COP K-means36), we convert (partial) prior knowledge into soft pairwise constraints and add them as additional terms into the loss"
- [abstract] Experiments on various scRNA-seq datasets from thousands to tens of thousands of cells show that scDCC can significantly improve clustering performance: "Experiments on various scRNA-seq datasets from thousands to tens of thousands of cells show that scDCC can significantly improve clustering performance"
- [abstract] purely unsupervised clustering methods may not produce biologically interpretable clusters, which complicates cell type assignment: "purely unsupervised clustering methods may not produce biologically interpretable clusters, which complicates cell type assignment"
- [methods] python scDCC_pairwise_Human_liver.py: "python scDCC_pairwise_Human_liver.py"
- [methods] --n_pairwise: number of pairwise constraints want to generate: "--n_pairwise: number of pairwise constraints want to generate"
- [methods] Datasets used in the study is available in: https://figshare.com/articles/dataset/scDCC_data/21563517: "Datasets used in the study is available in: https://figshare.com/articles/dataset/scDCC_data/21563517"
