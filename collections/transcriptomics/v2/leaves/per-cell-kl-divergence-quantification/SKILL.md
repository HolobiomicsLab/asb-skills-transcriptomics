---
name: per-cell-kl-divergence-quantification
description: Use when you need a local, per-cell quality measure for a 2D embedding
  (t-SNE or UMAP) — e.g. to feed EMBEDR p-value estimation, compare DR algorithms,
  or tune perplexity/n_neighbors.
when_to_use_negative:
- Your dataset is not normalized — the article notes EMBEDR's null-generation process
  has not been extended to non-normalized datasets, so per-cell DKL against such nulls
  is not supported
- You only need a single global DR quality score rather than local, per-cell error
  estimates or per-sample p-values
- You want theoretical/parametric p-values — null distributions here are purely empirical,
  bounded by the number of nulls embedded
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: EMBEDR
  role: Python package implementing the EES per-cell KL divergence statistic and the
    surrounding empirical hypothesis test with null embeddings
  repo: https://github.com/ejohnson643/EMBEDR
- name: numpy
  role: Loads the high-dimensional matrix (np.loadtxt) and supports array computations
    for affinity/KL calculations
- name: scipy
  role: Required dependency of the EMBEDR package used in affinity/KL divergence computation
- name: scikit-learn
  role: Required dependency for nearest-neighbor graphs and affinity computations
- name: openTSNE
  role: Custom variation of the openTSNE implementation used to generate the 2D t-SNE
    embeddings whose quality is quantified
- name: FIt-SNE
  role: Fast t-SNE implementation used to accelerate generation of the many data and
    null embeddings
  repo: https://github.com/KlugerLab/FIt-SNE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2022.100443
    title: 'EMBEDR: Distinguishing signal from noise in single-cell omics data'
  source_content_hash: sha256:937f2a2b3ed9d0520bcbc5570c5ecf97bbbaba194d59ee605fc3bfe5eea8a488
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
  - FIt-SNE
  - Python
  - numpy
  - openTSNE
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 15
---

# per-cell-kl-divergence-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes EMBEDR's empirical embedding statistic (EES): a per-cell Kullback-Leibler divergence between each cell's high-dimensional affinity distribution and its low-dimensional affinity distribution, quantifying how well each cell's local neighborhood is preserved by a t-SNE or UMAP embedding of real or null (marginal-resampled) data.

## When to use

Use when you need a local, per-cell quality measure for a 2D embedding (t-SNE or UMAP) — e.g. to feed EMBEDR p-value estimation, compare DR algorithms, or tune perplexity/n_neighbors.

## When NOT to use

- Your dataset is not normalized — the article notes EMBEDR's null-generation process has not been extended to non-normalized datasets, so per-cell DKL against such nulls is not supported
- You only need a single global DR quality score rather than local, per-cell error estimates or per-sample p-values
- You want theoretical/parametric p-values — null distributions here are purely empirical, bounded by the number of nulls embedded

## Inputs

- High-dimensional data matrix (e.g. mnist2500_X.txt loaded via np.loadtxt as float)
- Pre-generated 2D data embeddings (t-SNE or UMAP, multiple random initializations)
- Pre-generated 2D null embeddings from marginally-resampled null datasets
- Hyperparameter setting: perplexity (t-SNE) or nearest_neighbors (UMAP), default 10% of samples

## Outputs

- Per-cell KL divergence values for each data embedding and each null embedding
- Combined table saved as embedr_kl_divergences.csv
- Downstream EMBEDR per-sample p-values from comparing data KL divergences to the empirical null distribution

## How to apply

Load the high-dimensional matrix (e.g. mnist2500_X.txt) with numpy and the pre-generated data and null 2D embeddings from t-SNE or UMAP. Compute Gaussian-kernel affinity neighborhoods for every cell in the high-dimensional space, setting perplexity (or nearest_neighbors for UMAP) at the default of 10% of the number of samples; compute Student-t kernel affinities among cells in each 2D embedding (both data and null). For each embedding, calculate the KL divergence per cell between its high-dimensional and low-dimensional affinity distributions — small DKL indicates the cell's neighborhood is similarly structured in both spaces. Assemble the per-cell KL divergences across all data embeddings and all null embeddings into one table (embedr_kl_divergences.csv); these values are then compared against the empirical null distribution to yield EMBEDR p-values. Affinity matrices and nearest-neighbor graphs can be cached to avoid recomputation across sweeps.

## Related tools

- **EMBEDR** (Python package implementing the EES per-cell KL divergence statistic and the surrounding empirical hypothesis test with null embeddings) — https://github.com/ejohnson643/EMBEDR
- **numpy** (Loads the high-dimensional matrix (np.loadtxt) and supports array computations for affinity/KL calculations)
- **scipy** (Required dependency of the EMBEDR package used in affinity/KL divergence computation)
- **scikit-learn** (Required dependency for nearest-neighbor graphs and affinity computations)
- **openTSNE** (Custom variation of the openTSNE implementation used to generate the 2D t-SNE embeddings whose quality is quantified)
- **FIt-SNE** (Fast t-SNE implementation used to accelerate generation of the many data and null embeddings) — https://github.com/KlugerLab/FIt-SNE

## Examples

```
from EMBEDR import EMBEDR; import numpy as np; X = np.loadtxt("./data/mnist2500_X.txt").astype(float); embObj = EMBEDR(project_dir='./'); embObj.fit(X)
```

## Evaluation signals

- Small per-cell DKL values for cells whose neighborhoods are similarly structured in high- and low-dimensional spaces (Figure 2A behavior)
- The KL divergences assemble cleanly into a single table with one row per cell per embedding across all data and null embeddings (embedr_kl_divergences.csv)
- Data-embedding DKL values consistently lower than the empirical null distribution yield significant per-sample EMBEDR p-values
- Perplexity (t-SNE) or nearest_neighbors (UMAP) matches the default of 10% of the number of samples unless overridden
- Cached nearest-neighbor graphs and affinity matrices are reused across runs with matching parameters, so repeated runs give identical DKL values

## Limitations

- P-value resolution is finite because null distributions are empirical: the number of nulls embedded determines the lower bound on achievable p-values
- No theoretical description of the tails of the null distributions or principled parameterization of the null exists
- Runtime depends on the speed of the underlying DR algorithm implementations used to generate many null embeddings; GPU and quadratic-rate DR extensions were not included in this work
- Null-generation process has not been extended to non-normalized datasets

## Evidence

- [other] Compute Student-t kernel affinities among cells in each 2D embedding (both data and null embeddings).: [evidence span withheld — anchor status: not_found]
- [other] small DKL indicates the cell's neighborhood is similarly structured in both spaces: [evidence span withheld — anchor status: not_found]
