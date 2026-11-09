---
name: gaussian-affinity-estimation
description: Use when you have a high-dimensional data matrix (e.g. mnist2500_X.txt)
  plus data/null 2D t-SNE or UMAP embeddings and need a local, per-cell measure of
  how faithfully each cell's neighborhood survives embedding.
when_to_use_negative:
- Input is a non-normalized dataset — the article notes EMBEDR's null-generation process
  has not been extended to non-normalized data
- You need a single global embedding-quality score rather than local, per-cell estimates
  — the article's stated objective is a scheme that provides 'local (per cell) estimates
  of DR-induced error as opposed to a single global estimate'
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: EMBEDR
  role: Provides the EES/DKL quality statistic implementation and affinity computation
    underlying this skill
  repo: https://github.com/ejohnson643/EMBEDR
- name: numpy
  role: Loads the high-dimensional data matrix and manipulates affinity arrays
- name: scipy
  role: Required dependency for EMBEDR; supports the divergence/affinity computations
- name: scikit-learn
  role: Required dependency for EMBEDR; supports nearest-neighbor computation for
    affinity construction
- name: openTSNE
  role: Custom variation used to generate the 2D t-SNE embeddings whose low-dimensional
    affinities are computed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2022.100443
    title: 'EMBEDR: Distinguishing signal from noise in single-cell omics data'
  source_content_hash: sha256:df930fcc3c843c105098d6bc87ec1116def65b15d3154b3832e0e0b5dc50fe72
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
  - Python
  - numpy
  - openTSNE
  - scikit-learn
  - scipy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# gaussian-affinity-estimation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes per-cell neighborhood affinity distributions in the high-dimensional data (Gaussian kernel, perplexity-controlled) and in 2D embeddings (Student-t kernel), so their per-cell Kullback-Leibler divergence yields the EMBEDR empirical embedding statistic (EES).

## When to use

Use when you have a high-dimensional data matrix (e.g. mnist2500_X.txt) plus data/null 2D t-SNE or UMAP embeddings and need a local, per-cell measure of how faithfully each cell's neighborhood survives embedding.

## When NOT to use

- Input is a non-normalized dataset — the article notes EMBEDR's null-generation process has not been extended to non-normalized data
- You need a single global embedding-quality score rather than local, per-cell estimates — the article's stated objective is a scheme that provides 'local (per cell) estimates of DR-induced error as opposed to a single global estimate'

## Inputs

- High-dimensional data matrix (e.g. mnist2500_X.txt loaded via np.loadtxt)
- Pre-generated 2D embeddings of the real data (t-SNE or UMAP)
- Pre-generated 2D embeddings of marginally-resampled null datasets

## Outputs

- Per-cell Gaussian-kernel affinity distributions for the high-dimensional space
- Per-cell Student-t kernel affinity distributions for each 2D embedding
- Per-cell KL divergence (EES) table across all data and null embeddings, e.g. embedr_kl_divergences.csv

## How to apply

Load the high-dimensional matrix with numpy and the pre-generated data and null 2D embeddings from t-SNE or UMAP. For every cell, compute Gaussian-kernel affinity neighborhoods in the high-dimensional space, setting the perplexity (or nearest_neighbors for UMAP) at the EMBEDR default of 10% of the number of samples. For each 2D embedding (both real-data and marginally-resampled null embeddings), compute Student-t kernel affinities among cells. Then calculate the per-cell Kullback-Leibler divergence between each cell's high-dimensional and low-dimensional affinity distributions — small DKL indicates the neighborhood is similarly structured in both spaces — and assemble values across all data and null embeddings (e.g. into embedr_kl_divergences.csv) for comparison against the null distribution.

## Related tools

- **EMBEDR** (Provides the EES/DKL quality statistic implementation and affinity computation underlying this skill) — https://github.com/ejohnson643/EMBEDR
- **numpy** (Loads the high-dimensional data matrix and manipulates affinity arrays)
- **scipy** (Required dependency for EMBEDR; supports the divergence/affinity computations)
- **scikit-learn** (Required dependency for EMBEDR; supports nearest-neighbor computation for affinity construction)
- **openTSNE** (Custom variation used to generate the 2D t-SNE embeddings whose low-dimensional affinities are computed)

## Examples

```
X = np.loadtxt("./data/mnist2500_X.txt").astype(float); embObj = EMBEDR(project_dir='./'); embObj.fit(X)
```

## Evaluation signals

- Perplexity (t-SNE) or nearest_neighbors (UMAP) parameter equals 10% of the number of samples unless explicitly overridden
- Per-cell KL divergences are non-negative, and low values correspond to cells whose neighborhoods are similarly structured in both spaces (Figure 2A)
- Per-cell DKL values for real-data embeddings are empirically compared against the same statistic on null (marginally-resampled) embeddings to form EMBEDR p-values
- Affinity rows are computed per cell for every cell in the matrix (2500 in the MNIST example), and results assemble into a single table covering all data and null embeddings

## Limitations

- Calculated p values have finite resolution because null distributions are empirical; the number of nulls bounds the lower limit of p values
- The choice of perplexity/neighborhood scale is a user-specified hyperparameter, and DR distortions depend on such hyperparameters
- No theoretical description of the tails of the null distributions exists, and no principled method for parameterizing the null distribution is available

## Evidence

- [other] The EES is computed per cell by comparing each cell's affinity distributions in the high-dimensional space and in the lower-dimensional embedding using the Kullback-Leibler divergence (DKL): [evidence span withheld — anchor status: not_found]
- [other] small DKL indicates the cell's neighborhood is similarly structured in both spaces: [evidence span withheld — anchor status: not_found]
- [readme] readme perplexity default: [evidence span withheld — anchor status: not_found]
