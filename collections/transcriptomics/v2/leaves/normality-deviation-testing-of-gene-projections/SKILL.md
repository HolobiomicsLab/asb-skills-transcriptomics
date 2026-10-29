---
name: normality-deviation-testing-of-gene-projections
description: Use when a cell-gene expression matrix is highly sparse (e.g. ~95% zeros)
  and you must separate spurious sparsity-driven eigenvector localization from true
  biological signal before denoising or feature selection.
when_to_use_negative:
- The matrix is dense or already low-rank denoised, so sparsity-induced eigenvector
  localization is not a concern
- You are testing the raw (non-randomized) matrix, where deviations from normality
  may reflect true biological signal rather than sparsity artifacts
- The dataset is too small for RMT universality predictions (Tracy-Widom / Marchenko-Pastur)
  to be meaningful
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_0203
tools:
- name: Randomly
  role: Python library that randomizes the dataset, decomposes the randomized matrix,
    and tests gene projections onto eigenvectors for deviations from RMT-expected
    normality
  repo: https://github.com/luisaparicio/randomly
- name: Python 3
  role: Runtime environment; Randomly only works with Python 3, not Python 2
- name: pip
  role: Installation method for the Randomly package (pip install randomly)
- name: Jupyter Notebook
  role: Recommended environment in which to run Randomly interactively
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2020.100035
    title: A Random Matrix Theory Approach to Denoise Single-Cell Data
  source_content_hash: sha256:c7f335bec414083c0cda826cad1e6dd5d489f42d0e0eee38d660416d415d3c2c
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
  - pip
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# Normality-deviation testing of gene projections

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A random matrix theory (RMT)-based statistical test that projects each gene onto eigenvectors of a randomized (label-permuted) single-cell expression matrix and flags genes whose projections deviate from the normal distribution expected under RMT universality, identifying genes responsible for sparsity-induced spurious eigenvector localization.

## When to use

Use when a cell-gene expression matrix is highly sparse (e.g. ~95% zeros) and you must separate spurious sparsity-driven eigenvector localization from true biological signal before denoising or feature selection.

## When NOT to use

- The matrix is dense or already low-rank denoised, so sparsity-induced eigenvector localization is not a concern
- You are testing the raw (non-randomized) matrix, where deviations from normality may reflect true biological signal rather than sparsity artifacts
- The dataset is too small for RMT universality predictions (Tracy-Widom / Marchenko-Pastur) to be meaningful

## Inputs

- Sparse cell-gene expression matrix (e.g. 95%-sparse matrix of 6,573 human PBMCs from Kang et al., GSE96583, or 3,005 mouse cortex cells from Zeisel et al., GSE60361)
- Randomized control matrix produced by independently permuting cell labels for each gene

## Outputs

- Per-gene flags identifying genes whose eigenvector projections deviate from the normal distribution expected under RMT universality
- List of genes responsible for localized eigenvectors in the randomized (sparsity-only) matrix, used for the first algorithm filter step (e.g. written to sparsity_filtered_genes.csv)

## How to apply

Load the sparse cell-gene matrix (e.g. 6,573 human PBMCs from Kang et al. GSE96583 or 3,005 mouse cortex cells from Zeisel et al. GSE60361) into Python 3 with the Randomly library. Build a sparsity-only control by independently permuting cell labels for each gene, which preserves marginal expression while destroying inter-gene correlation. Decompose the randomized matrix and project each gene onto the eigenvectors; under RMT universality these projections should follow a normal distribution, so flag genes whose projections show large deviations from normality. Genes flagged this way are responsible for localized eigenvectors in the sparsity-only case and are discarded in the first step of the denoising algorithm to suppress spurious sparsity effects (~3% of information). The rationale is that departures from RMT universality distributions (TW/MP) indicate signal, so testing on the randomized data isolates non-biological, sparsity-induced departures from genuine biological ones.

## Related tools

- **Randomly** (Python library that randomizes the dataset, decomposes the randomized matrix, and tests gene projections onto eigenvectors for deviations from RMT-expected normality) — https://github.com/luisaparicio/randomly
- **Python 3** (Runtime environment; Randomly only works with Python 3, not Python 2)
- **pip** (Installation method for the Randomly package (pip install randomly))
- **Jupyter Notebook** (Recommended environment in which to run Randomly interactively)

## Examples

```
pip install randomly; import randomly
```

## Evaluation signals

- Projections of flagged genes onto eigenvectors of the randomized matrix show statistically significant departures from a normal distribution, while unflagged genes' projections are compatible with RMT universality predictions
- The randomized control matrix preserves each gene's marginal expression distribution (permutation-only check) while destroying inter-gene correlations
- The proportion of information attributed to sparsity-induced spurious signal is around 3%, consistent with the paper's abstract-level decomposition (~95% random, ~3% spurious sparsity, ~2% biological)
- After discarding flagged genes, localized eigenvectors in the randomized case are suppressed and remaining deviations from RMT universality can be attributed to potential biological signal

## Limitations

- Predictive quantitative models of gene expression are lacking, so the correct expression distribution and i.i.d. assumptions are unclear
- There is no complete quantitative understanding of all noise sources in single-cell measurements, which RMT universality only partially addresses
- No version or release information is provided for the publicly available algorithm code
- Randomly only works with Python 3 (not Python 2)

## Evidence
