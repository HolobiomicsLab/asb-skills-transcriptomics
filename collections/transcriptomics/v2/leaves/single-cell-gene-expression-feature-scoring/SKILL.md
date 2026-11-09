---
name: single-cell-gene-expression-feature-scoring
description: Use when the RMT eigen-decomposition of a single-cell covariance matrix
  has already been partitioned into noise-bulk and signal-outlier eigenvectors and
  you must rank individual genes by signal-like variance (e.g., to select ~1,500 genes
  as the paper does).
when_to_use_negative:
- Signal eigenvectors have not yet been identified or partitioned from the noise bulk
  — this is the final classification step and requires an existing RMT eigen-decomposition
- The goal is to suppress sparsity effects (eigenvector localization caused by dropout)
  rather than to score/rank genes — use the sparsity filter step instead, which discards
  genes responsible for localized eigenvectors in the randomized case
tools:
- name: Randomly
  role: RMT denoising library that performs the eigenvector-based feature scoring
    and selection of signal-responsible genes
  repo: https://github.com/luisaparicio/randomly
- name: Python 3
  role: Required runtime; Randomly only works with Python 3 (not Python 2)
- name: Jupyter Notebook
  role: Convenient execution environment for running the Randomly gene-scoring workflow
  repo: http://jupyter.org/
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2020.100035
    title: A Random Matrix Theory Approach to Denoise Single-Cell Data
  source_content_hash: sha256:90d5756ab44800106b4ba0d2823c8beb6c985fb7cd8cb894def9861a19a7a6e2
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/RabadanLab/randomly
  repo_url: https://github.com/luisaparicio/randomly
  tools:
  - Jupyter Notebook
  - Python 3
  - Randomly
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# single-cell-gene-expression-feature-scoring

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Final classification step of the Randomly RMT pipeline that scores every gene as signal-like versus noise-like by chi-squared testing each gene's variance projected onto signal versus noise eigenvector subsets, then FDR-corrects and ranks genes to yield a concrete feature selection for single-cell denoising.

## When to use

Use when the RMT eigen-decomposition of a single-cell covariance matrix has already been partitioned into noise-bulk and signal-outlier eigenvectors and you must rank individual genes by signal-like variance (e.g., to select ~1,500 genes as the paper does).

## When NOT to use

- Signal eigenvectors have not yet been identified or partitioned from the noise bulk — this is the final classification step and requires an existing RMT eigen-decomposition
- The goal is to suppress sparsity effects (eigenvector localization caused by dropout) rather than to score/rank genes — use the sparsity filter step instead, which discards genes responsible for localized eigenvectors in the randomized case

## Inputs

- RMT eigen-decomposition of the single-cell expression covariance matrix (eigenvectors partitioned into noise bulk and signal outlier subsets)
- Sparse cell-gene expression matrix, e.g. the 95% sparse 6,573 human PBMC matrix (Kang et al.) or 3,005 mouse cortex cells (Zeisel et al.)
- Noise eigenvector subset identified via RMT universality (Tracy-Widom / Marchenko-Pastur) statistics

## Outputs

- signal_like_genes.csv: ranked gene table with gene identifier, chi-squared statistic, Benjamini-Hochberg corrected p-value, and rank
- Selection of the top 1,500 genes flagged/selected as signal-like
- A low-rank representation of the single-cell data built from the signal-like genes

## How to apply

Load the eigen-decomposition of the single-cell expression covariance matrix and split eigenvectors into the noise bulk (consistent with RMT universality, ~95% of information) and the signal outlier subset. Project each gene onto both eigenvector subsets to obtain per-gene variance contributions in the signal and noise regimes. Run a chi-squared test for variance on each gene's signal versus noise projections, assessing departure from the noise expectation, then apply a Benjamini-Hochberg false discovery rate correction across all genes' chi-squared p-values. Rank genes by corrected significance of signal-like variance and select the top genes as signal-like (the paper reports selecting exactly 1,500 genes) and write the ranked table to signal_like_genes.csv. The scoring runs in Python 3 via the Randomly library (installable with pip, convenient in a Jupyter Notebook).

## Related tools

- **Randomly** (RMT denoising library that performs the eigenvector-based feature scoring and selection of signal-responsible genes) — https://github.com/luisaparicio/randomly
- **Python 3** (Required runtime; Randomly only works with Python 3 (not Python 2))
- **Jupyter Notebook** (Convenient execution environment for running the Randomly gene-scoring workflow) — http://jupyter.org/

## Examples

```
pip install randomly; python -c "import randomly"
```

## Evaluation signals

- signal_like_genes.csv exists and contains exactly one row per tested gene
- Exactly 1,500 genes are flagged/selected as signal-like, matching the paper's reported selection
- Every row carries a gene identifier, a chi-squared statistic, a Benjamini-Hochberg corrected p-value, and a rank column
- Corrected p-values are monotone-consistent with the ranking (higher rank = stronger corrected significance of signal-like variance)
- Selected genes drive the low-rank representation, distinguishable from the noise bulk that is consistent with RMT universality

## Limitations

- Randomly only works with Python 3 (not Python 2)
- No version or release information is provided for the publicly available algorithm code
- Sparsity can cause spurious eigenvector localization, falsely identifying meaningful directions in the data (~3% of single-cell information is spurious signal induced by sparsity), so genes must be evaluated against the randomized-data baseline to avoid confounding sparsity effects with biological signal

## Evidence
