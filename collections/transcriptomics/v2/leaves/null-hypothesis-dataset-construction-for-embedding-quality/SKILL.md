---
name: null-hypothesis-dataset-construction-for-embedding-quality
description: 'Use when establishing the null expectation for embedding quality of
  scRNA-seq (e.g., Tabula Muris bone marrow) data: you need signalless data with preserved
  per-gene marginals to compute EMBEDR p-values.'
when_to_use_negative:
- Input is non-normalized data — the paper states EMBEDR's null-generation process
  has not been extended to non-normalized datasets
- The goal is to test differential-expression or co-expression signals directly; marginal
  resampling destroys joint gene correlations by design, so the null is only appropriate
  for embedding-quality hypothesis testing
- You need exact tail probabilities rather than empirical estimates — null distributions
  are computed empirically, giving p-values of finite resolution
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: EMBEDR
  role: Implements the marginal-resampling null generation and uses the null embeddings
    for the empirical hypothesis test of embedding quality
  repo: https://github.com/ejohnson643/EMBEDR
- name: numpy
  role: Holds the cells x genes count matrix and performs per-gene resampling with
    replacement to build the null matrix
- name: scipy
  role: Required dependency of the EMBEDR package for installation and numerical routines
- name: seaborn
  role: Overlays empirical per-gene marginal distributions (histograms/KDEs) of real
    vs. null matrices for validation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2022.100443
    title: 'EMBEDR: Distinguishing signal from noise in single-cell omics data'
  source_content_hash: sha256:039e32088664ba0e9fc120afe6439b8f6d54b19e4a257f6c9cab5df33a6b9dca
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
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# null-hypothesis-dataset-construction-for-embedding-quality

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs data-driven null datasets for EMBEDR's empirical hypothesis test by marginal resampling: each gene's expression values are independently drawn with replacement from that gene's empirical distribution, producing a null matrix of the same shape whose per-gene marginals match the real data while inter-gene correlations are destroyed.

## When to use

Use when establishing the null expectation for embedding quality of scRNA-seq (e.g., Tabula Muris bone marrow) data: you need signalless data with preserved per-gene marginals to compute EMBEDR p-values.

## When NOT to use

- Input is non-normalized data — the paper states EMBEDR's null-generation process has not been extended to non-normalized datasets
- The goal is to test differential-expression or co-expression signals directly; marginal resampling destroys joint gene correlations by design, so the null is only appropriate for embedding-quality hypothesis testing
- You need exact tail probabilities rather than empirical estimates — null distributions are computed empirically, giving p-values of finite resolution

## Inputs

- Preprocessed single-cell count matrix (cells x genes numpy array), e.g., Tabula Muris bone marrow FACS counts
- Number of null embeddings to generate (n_null_embed parameter)

## Outputs

- Null data matrix (null_data_matrix.npy) with identical shape to the input and near-identical per-gene marginal distributions
- Overlay plots of real vs. null per-gene marginal distributions (histograms/KDEs, as in Figure 3B)
- Null embeddings used to build the empirical null distribution for EMBEDR p-values

## How to apply

Load the preprocessed Tabula Muris bone marrow count matrix (cells x genes) into a numpy array. For every gene column independently, draw values with replacement from that column's empirical distribution to build a null data matrix of identical shape; this destroys inter-gene (correlation) structure while preserving each gene's marginal distribution, which is the key property making the null 'biologically realistic' at the single-gene level. Save the result (e.g., null_data_matrix.npy) and feed both real and null data through the same DR algorithm (t-SNE via FIt-SNE or UMAP) so EMBEDR can compare neighborhood-quality statistics against the null expectation. Validate by overlaying per-gene marginal histograms/KDEs (seaborn/matplotlib) for a representative gene panel, confirming real and null marginals coincide as in Figure 3B, and confirming the null matrix shape matches the input. Use 3-10 null embeddings in practice (n_null_embed), noting that the number of nulls bounds the resolution of the resulting p-values.

## Related tools

- **EMBEDR** (Implements the marginal-resampling null generation and uses the null embeddings for the empirical hypothesis test of embedding quality) — https://github.com/ejohnson643/EMBEDR
- **numpy** (Holds the cells x genes count matrix and performs per-gene resampling with replacement to build the null matrix)
- **scipy** (Required dependency of the EMBEDR package for installation and numerical routines)
- **seaborn** (Overlays empirical per-gene marginal distributions (histograms/KDEs) of real vs. null matrices for validation)

## Examples

```
import numpy as np; X = np.loadtxt('./data/tabula-muris/bone_marrow_counts.csv', delimiter=','); null_X = np.column_stack([np.random.choice(X[:, g], size=X.shape[0], replace=True) for g in range(X.shape[1])]); np.save('null_data_matrix.npy', null_X)
```

## Evaluation signals

- Null matrix has exactly the same shape (cells x genes) as the input count matrix
- Per-gene marginal distributions of real and null data visually coincide when overlaid (histograms/KDEs), reproducing Figure 3B
- Per-gene marginals are 'nearly identical' between real and null data, while the joint (correlation) distribution of genes is altered
- The same number of null embeddings as specified by n_null_embed is generated and cached in the project directory
- EMBEDR p-values computed against the null have finite resolution bounded by the number of nulls embedded

## Limitations

- The null-generation process has not been extended to non-normalized datasets, so EMBEDR cannot yet quality-check entire data-processing pipelines
- There is a finite resolution to the calculated p values since null distributions are empirical; the number of nulls embedded determines the lower bound on p values
- No theoretical description of the tails of the null distributions exists, and no principled method for parameterizing the null distribution is available

## Evidence

- [results] Null datasets are constructed via marginal resampling: each gene's expression values are independently drawn with replacement from that gene's distribution in the original data: [evidence span withheld — anchor status: not_found]
