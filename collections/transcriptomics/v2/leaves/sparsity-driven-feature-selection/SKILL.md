---
name: sparsity-driven-feature-selection
description: Use when a cell-gene expression matrix is highly sparse (e.g., ~95% zeros
  in scRNA-seq) and localized eigenvectors must be separated into sparsity-induced
  artifacts versus true biological signal before RMT denoising.
when_to_use_negative:
- The dataset is only mildly sparse or dense — sparsity-induced eigenvector localization
  will be negligible, so gene removal risks discarding true signal
- The goal is imputation of zero counts (e.g., with MAGIC or scImpute) rather than
  removal of genes responsible for sparsity-induced localized eigenvectors
- The matrix has already had spurious genes removed in a prior denoising pass
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Randomly
  role: Python library implementing the randomization, eigenvector decomposition,
    and gene-discarding sparsity filter
  repo: github.com/luisaparicio/randomly
- name: Python 3
  role: Runtime required for Randomly (does not work with Python 2)
- name: pip
  role: Used to install Randomly from the command line
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2020.100035
    title: A Random Matrix Theory Approach to Denoise Single-Cell Data
  source_content_hash: sha256:f370f57ab5d775e2ce1595a7be76d3dabbe18554e66d32fdc4270ffe12305150
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
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# sparsity-driven-feature-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

The first step of the RMT denoising algorithm (Randomly): identify and discard genes that introduce spurious effects due to sparsity by testing gene projections onto eigenvectors of a sparsity-only randomized control for deviations from RMT-universality normality.

## When to use

Use when a cell-gene expression matrix is highly sparse (e.g., ~95% zeros in scRNA-seq) and localized eigenvectors must be separated into sparsity-induced artifacts versus true biological signal before RMT denoising.

## When NOT to use

- The dataset is only mildly sparse or dense — sparsity-induced eigenvector localization will be negligible, so gene removal risks discarding true signal
- The goal is imputation of zero counts (e.g., with MAGIC or scImpute) rather than removal of genes responsible for sparsity-induced localized eigenvectors
- The matrix has already had spurious genes removed in a prior denoising pass

## Inputs

- Highly sparse cell-gene expression matrix (~95% zeros), e.g., 6,573 human PBMCs from Kang et al. (GSE96583)
- 3,005 mouse cortex cells from Zeisel et al. (GSE60361) as an alternative benchmark dataset
- Sparsity-only control matrix obtained by permuting cell labels per gene

## Outputs

- List of discarded genes that introduce spurious sparsity effects (e.g., sparsity_filtered_genes.csv)
- Filtered expression matrix with sparsity-flagged genes removed

## How to apply

Load the sparse cell-gene expression matrix (e.g., the 95%-sparse matrix of 6,573 human PBMCs from Kang et al., or 3,005 mouse cortex cells from Zeisel et al.) using the Randomly library in Python 3. Randomize the dataset by independently permuting the cell labels for each gene, which preserves marginal expression but destroys inter-gene correlations, yielding a sparsity-only control matrix. Decompose the randomized matrix and test each gene's projections onto eigenvectors for deviations from the normal distribution expected under RMT universality; genes whose projections deviate are flagged as responsible for localized eigenvectors induced by sparsity. Discard these flagged genes from the raw matrix as the algorithm's first step, since roughly 3% of single-cell information is spurious signal induced by sparsity, and record the removed-gene list (e.g., sparsity_filtered_genes.csv) alongside the filtered matrix.

## Related tools

- **Randomly** (Python library implementing the randomization, eigenvector decomposition, and gene-discarding sparsity filter) — github.com/luisaparicio/randomly
- **Python 3** (Runtime required for Randomly (does not work with Python 2))
- **pip** (Used to install Randomly from the command line)

## Examples

```
pip install randomly
```

## Evaluation signals

- The flagged genes are exactly those whose projections onto eigenvectors of the randomized (cell-label-permuted) matrix deviate from the normal distribution expected under RMT universality
- The fraction of removed signal should align with the paper's finding that about 3% of single-cell information is spurious sparsity-induced signal
- The randomized control must preserve per-gene marginal expression while destroying inter-gene correlations (independent permutation of cell labels per gene)
- The removed-gene list is written to sparsity_filtered_genes.csv and the filtered matrix retains the remaining genes of the raw matrix
- Randomly runs under Python 3 (not Python 2), so the pipeline completes without interpreter errors

## Limitations

- No version or release information is provided for the publicly available algorithm code
- A complete quantitative understanding of the different noise sources in single-cell measurements is lacking; departures from RMT-universality normality may conflate sparsity effects with true signal
- The correct expression distribution and the validity of the i.i.d. assumption are unclear, which limits purely parametric alternatives
- Randomly only works with Python 3 (not Python 2)

## Evidence
