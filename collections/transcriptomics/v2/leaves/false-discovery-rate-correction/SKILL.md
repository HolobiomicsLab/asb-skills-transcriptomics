---
name: false-discovery-rate-correction
description: Use when many per-gene statistical tests (e.g. chi-squared variance tests
  of signal vs noise eigenvector projections) must be multiple-testing corrected before
  ranking or selecting genes.
when_to_use_negative:
- Only a small number of hypothesis tests is performed, so multiple-testing correction
  is unnecessary
- The input is a pre-ranked or pre-filtered feature list rather than per-gene test
  statistics requiring FDR control
- The goal is filtering sparsity-induced effects (randomizing cell labels and discarding
  genes responsible for localized eigenvectors) rather than ranking signal-like genes
  after testing
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Randomly
  role: Library that produces the RMT eigen-decomposition, per-gene chi-squared statistics,
    and FDR correction for denoising single-cell data
  repo: github.com/RabadanLab/randomly
- name: Python 3
  role: Runtime environment; Randomly only works with Python 3 (not Python 2)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2020.100035
    title: A Random Matrix Theory Approach to Denoise Single-Cell Data
  source_content_hash: sha256:896ba22ed7742bd23b1a2a72a7f8458f777c31dea28a7fdb7209993969e1b94b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/RabadanLab/randomly
  tools:
  - Python 3
  - Randomly
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# false-discovery-rate-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Per-gene chi-squared tests on signal versus noise eigenvector projections (from the Randomly RMT pipeline) are corrected for multiple testing with a false discovery rate (Benjamini-Hochberg) procedure so that a ranked, statistically defensible set of signal-like genes can be selected.

## When to use

Use when many per-gene statistical tests (e.g. chi-squared variance tests of signal vs noise eigenvector projections) must be multiple-testing corrected before ranking or selecting genes.

## When NOT to use

- Only a small number of hypothesis tests is performed, so multiple-testing correction is unnecessary
- The input is a pre-ranked or pre-filtered feature list rather than per-gene test statistics requiring FDR control
- The goal is filtering sparsity-induced effects (randomizing cell labels and discarding genes responsible for localized eigenvectors) rather than ranking signal-like genes after testing

## Inputs

- Per-gene chi-squared test p-values (and statistics) from signal versus noise eigenvector projection variances of the single-cell covariance matrix
- RMT eigen-decomposition of the single-cell expression covariance matrix with noise-bulk and signal-outlier eigenvector subsets

## Outputs

- Corrected (FDR-adjusted) p-values for every tested gene
- Ranked gene table signal_like_genes.csv with columns: gene identifier, chi-squared statistic, corrected p-value, rank
- Selected set of signal-like genes (1,500 per the paper's reported selection)

## How to apply

After Randomly partitions the eigen-decomposition of the single-cell expression covariance matrix into a noise bulk (consistent with RMT universality) and a signal outlier subset, project every gene onto both eigenvector subsets to obtain per-gene variance contributions in each regime. Run a chi-squared test for variance on each gene's signal versus noise projections to assess whether its variance departs from the noise expectation, yielding one raw p-value per tested gene. Because thousands of genes are tested simultaneously, apply a false discovery rate correction (Benjamini-Hochberg) across all genes' chi-squared p-values rather than thresholding raw p-values. Rank genes by corrected significance of signal-like variance and select the top genes (the paper's workflow selects 1,500 signal-like genes). Write the ranked table (gene identifier, chi-squared statistic, corrected p-value, rank) to signal_like_genes.csv.

## Related tools

- **Randomly** (Library that produces the RMT eigen-decomposition, per-gene chi-squared statistics, and FDR correction for denoising single-cell data) — github.com/RabadanLab/randomly
- **Python 3** (Runtime environment; Randomly only works with Python 3 (not Python 2))

## Examples

```
pip install randomly; import randomly
```

## Evaluation signals

- The output file signal_like_genes.csv exists and contains one row per tested gene
- Exactly 1,500 genes are flagged/selected as signal-like, matching the paper's reported selection
- Every row carries a gene identifier, chi-squared statistic, corrected p-value, and rank
- Corrected p-values are monotonically consistent with raw chi-squared p-values (Benjamini-Hochberg ordering preserved) and all corrected values lie in [0, 1]
- Highly ranked genes correspond to projections whose variance departs from the RMT noise expectation

## Limitations

- Only the last ~2% of information in single-cell data reflects true biological signal; ~95% is RMT-compatible noise and ~3% is spurious signal induced by sparsity, so the selected gene set is small and must be interpreted carefully
- Sparsity can cause spurious eigenvector localization, falsely identifying meaningful directions in the data; this must be handled by the upstream randomized-data sparsity filter before FDR ranking
- No version or release information is provided for the publicly available algorithm code
- Roughly 3% of the information in single-cell data is spurious signal induced by sparsity

## Evidence
