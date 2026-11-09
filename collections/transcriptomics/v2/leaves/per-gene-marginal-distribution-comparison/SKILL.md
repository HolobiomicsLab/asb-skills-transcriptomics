---
name: per-gene-marginal-distribution-comparison
description: Use when a marginally-resampled null data matrix has been generated from
  scRNA-seq counts and you must verify that each gene's empirical distribution is
  preserved before using the null for EMBEDR p-value estimation.
when_to_use_negative:
- When the goal is to compare joint/correlation structure rather than per-gene marginals
  — marginal resampling intentionally alters the joint distribution of genes
- When the dataset has not been preprocessed/normalized — the article notes EMBEDR's
  null-generation process has not been extended to non-normalized datasets
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: numpy
  role: Holds the cells x genes count matrix and performs per-gene column-wise resampling
    with replacement to build the null matrix
  repo: https://github.com/ejohnson643/EMBEDR
- name: seaborn
  role: Generates the overlaid per-gene marginal distribution plots (histograms/KDEs)
    used to validate real vs. null marginals
  repo: https://github.com/ejohnson643/EMBEDR
- name: EMBEDR
  role: Package whose marginal-resampling null-generation scheme produces the null
    dataset being validated
  repo: https://github.com/ejohnson643/EMBEDR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2022.100443
    title: 'EMBEDR: Distinguishing signal from noise in single-cell omics data'
  source_content_hash: sha256:ca2c3e56a53ce12003d883584554fbbfd50c3074d751eabb3f8295b6c8c9eb09
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ejohnson643/EMBEDR
  repo_url: https://github.com/ejohnson643/EMBEDR
  tools:
  - EMBEDR
  - numpy
  - scipy
  - seaborn
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# per-gene-marginal-distribution-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Validation procedure for EMBEDR's marginal-resampling null datasets: for each gene in a single-cell count matrix (e.g. Tabula Muris bone marrow cells x genes), the empirical marginal distribution of the resampled null matrix is overlaid with that of the real data using histograms/KDEs, confirming that marginal resampling destroys inter-gene correlations while preserving per-gene marginals (as in Figure 3B).

## When to use

Use when a marginally-resampled null data matrix has been generated from scRNA-seq counts and you must verify that each gene's empirical distribution is preserved before using the null for EMBEDR p-value estimation.

## When NOT to use

- When the goal is to compare joint/correlation structure rather than per-gene marginals — marginal resampling intentionally alters the joint distribution of genes
- When the dataset has not been preprocessed/normalized — the article notes EMBEDR's null-generation process has not been extended to non-normalized datasets

## Inputs

- Preprocessed scRNA-seq count matrix (cells x genes, e.g. Tabula Muris bone marrow counts) as a numpy array
- Marginally-resampled null data matrix of identical shape (e.g. saved as null_data_matrix.npy)

## Outputs

- Overlay plots (histograms/KDEs) of real vs. null per-gene marginal distributions for a representative panel of genes
- Validation confirmation that the null matrix shape equals the input shape and per-gene marginals coincide

## How to apply

Load the preprocessed count matrix (e.g. Tabula Muris bone marrow FACS counts, cells x genes) into a numpy array. For every gene column independently, draw values with replacement from that column's empirical distribution to build a null data matrix of identical shape, which destroys inter-gene (correlation) structure while preserving per-gene marginals. Then use seaborn/matplotlib to overlay the empirical per-gene marginal distributions (histograms/KDEs) of the real and null matrices for a representative panel of genes. Correctness is judged by (1) exact shape equality between the real and null matrices and (2) visual/near-identical coincidence of real vs. null per-gene marginals, matching Figure 3B of the EMBEDR paper. This matters because EMBEDR's empirical hypothesis test relies on the null being signal-free yet biologically realistic at the marginal level.

## Related tools

- **numpy** (Holds the cells x genes count matrix and performs per-gene column-wise resampling with replacement to build the null matrix) — https://github.com/ejohnson643/EMBEDR
- **seaborn** (Generates the overlaid per-gene marginal distribution plots (histograms/KDEs) used to validate real vs. null marginals) — https://github.com/ejohnson643/EMBEDR
- **EMBEDR** (Package whose marginal-resampling null-generation scheme produces the null dataset being validated) — https://github.com/ejohnson643/EMBEDR

## Examples

```
import numpy as np, seaborn as sns; real = np.load('data.npy'); null = np.load('null_data_matrix.npy'); assert real.shape == null.shape; sns.kdeplot(real[:, gene], label='real'); sns.kdeplot(null[:, gene], label='null')
```

## Evaluation signals

- Null data matrix has exactly the same shape (cells x genes) as the original input matrix
- Real and null per-gene marginal distributions visually coincide in overlaid histograms/KDEs (near-identical marginals, as in Figure 3B of the paper)
- Inter-gene correlations in the null matrix differ from the original data (joint distribution altered)
- Each gene's null values are drawn only from that gene's own empirical distribution in the original data

## Limitations

- The null-generation (and hence this validation) applies to preprocessed/normalized data; EMBEDR's null-generation process has not been extended to non-normalized datasets
- Marginal resampling preserves per-gene marginals but by design destroys the joint (correlation) distribution of genes
- p values derived from such empirical nulls have finite resolution bounded by the number of nulls embedded

## Evidence

- [results] Null datasets are constructed via marginal resampling: each gene's expression values are independently drawn with replacement from that gene's distribution in the original data: [evidence span withheld — anchor status: not_found]
