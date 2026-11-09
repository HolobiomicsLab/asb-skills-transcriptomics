---
name: chi-squared-variance-testing
description: Use when you already have an RMT eigen-decomposition of a single-cell
  covariance matrix partitioned into noise-bulk and signal-outlier eigenvectors and
  need to classify individual genes as signal- versus noise-responsible.
when_to_use_negative:
- You do not yet have the RMT eigen-decomposition or the noise/signal eigenvector
  partition — this is the final step, not the whole pipeline; the earlier sparsity
  filter must run first
- You want to test differential expression between cell groups — this tests variance
  departures from the RMT noise expectation, not group-wise expression differences
- Your data is dense or already fully denoised, so the random-matrix (noise bulk)
  assumption underlying the chi-squared expectation no longer holds
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Randomly
  role: Library implementing the RMT denoising algorithm whose final step is the chi-squared
    variance test-based signal gene selection
  repo: https://github.com/luisaparicio/randomly
- name: Python 3
  role: Required runtime — Randomly only works with Python 3 (not Python 2)
- name: Jupyter Notebook
  role: Recommended environment for running the Randomly tutorial workflow
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2020.100035
    title: A Random Matrix Theory Approach to Denoise Single-Cell Data
  source_content_hash: sha256:253bd99297eb9507c22e1f67dc489881817d32524dacd92e03d263daaf2d2dda
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
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# chi-squared-variance-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

The final feature-selection step of the Randomly RMT denoising algorithm: each gene's variance contributions onto signal versus noise eigenvector subsets are compared with a chi-squared variance test, Benjamini-Hochberg FDR-corrected, and genes are ranked to select the ~1,500 most signal-like genes.

## When to use

Use when you already have an RMT eigen-decomposition of a single-cell covariance matrix partitioned into noise-bulk and signal-outlier eigenvectors and need to classify individual genes as signal- versus noise-responsible.

## When NOT to use

- You do not yet have the RMT eigen-decomposition or the noise/signal eigenvector partition — this is the final step, not the whole pipeline; the earlier sparsity filter must run first
- You want to test differential expression between cell groups — this tests variance departures from the RMT noise expectation, not group-wise expression differences
- Your data is dense or already fully denoised, so the random-matrix (noise bulk) assumption underlying the chi-squared expectation no longer holds

## Inputs

- RMT eigen-decomposition (eigenvalues + eigenvectors) of the single-cell expression covariance matrix, pre-partitioned into noise-bulk and signal-outlier eigenvector subsets
- Sparse cell-gene expression matrix (e.g. ~95% sparse, as in the 6,573 human PBMC Kang et al. dataset)
- FDR threshold for Benjamini-Hochberg correction across all genes' chi-squared p-values

## Outputs

- signal_like_genes.csv with one row per tested gene: gene identifier, chi-squared statistic, BH-corrected p-value, and rank
- Selection of exactly 1,500 genes flagged as signal-like

## How to apply

Load the eigen-decomposition of the single-cell expression covariance matrix and split eigenvectors into the RMT-universality-consistent noise bulk and the signal outlier subset. Project every gene onto both eigenvector subsets to get per-gene variance contributions in each regime, then run a chi-squared test for variance on each gene's signal versus noise projections to assess departure from the noise expectation. Correct all genes' chi-squared p-values across the genome with Benjamini-Hochberg FDR and rank genes by corrected significance of signal-like variance. Rationale: the model treats any single-cell dataset as a random matrix (noise) plus a low-rank perturbation (signal), so genes whose projection variance departs from the noise expectation are the ones 'responsible' for the signal eigenvectors. Write the ranked table to signal_like_genes.csv and select the top 1,500 most signal-like genes per the paper's reported selection.

## Related tools

- **Randomly** (Library implementing the RMT denoising algorithm whose final step is the chi-squared variance test-based signal gene selection) — https://github.com/luisaparicio/randomly
- **Python 3** (Required runtime — Randomly only works with Python 3 (not Python 2))
- **Jupyter Notebook** (Recommended environment for running the Randomly tutorial workflow)

## Examples

```
pip install randomly
```

## Evaluation signals

- signal_like_genes.csv exists and contains exactly one row per tested gene
- Exactly 1,500 genes are flagged/selected as signal-like, matching the paper's reported selection
- Every corrected p-value lies in [0, 1] and the table is sorted by corrected significance (rank 1 = smallest FDR)
- Each row carries the four required columns: gene identifier, chi-squared statistic, corrected p-value, rank
- Roughly 95% of genes' information should fall in the noise bulk — a selection far larger or smaller than ~1,500 of the tested genes indicates the noise/signal eigenvector partition was mis-set

## Limitations

- Sparsity can cause spurious eigenvector localization (~3% of information in single-cell data is spurious signal induced by sparsity), so the preceding sparsity filter (removing genes responsible for localized eigenvectors in the randomized data) must be applied first or the chi-squared selection will confound sparsity effects with biological signal
- Randomly only works with Python 3 (not Python 2)
- No version or release information is provided for the publicly available algorithm code

## Evidence

- [intro] modeling rationale: "any single-cell dataset can be modeled as a random matrix (that encodes the noise) plus a low-rank perturbation (which is the signal)."
- [intro] sparsity filter prerequisite: "we propose a feature selection method that discards the features (genes) that are responsible for the localized eigenvectors in the randomized case."
- [discussion] gene selection capability: "This method is also able to select genes responsible for potentially interesting biological signals."
- [readme] runtime constraint: "Randomly only works with Python 3 (not Python 2). The easiest way to install Randomly is via the command line with pip"
- [abstract] sparsity confound: "sparsity can cause spurious eigenvector localization, falsely identifying meaningful directions in the data"
