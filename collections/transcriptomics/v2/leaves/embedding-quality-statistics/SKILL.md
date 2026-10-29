---
name: embedding-quality-statistics
description: Use when per-cell quality scores are needed for 2D embeddings (t-SNE
  or UMAP) of high-dimensional data such as scRNA-seq or MNIST, to distinguish real
  structure from algorithm-induced noise.
when_to_use_negative:
- The dataset is not normalized — the article notes EMBEDR's null-generation process
  has not been extended to non-normalized datasets, so per-cell KL statistics against
  such nulls are not supported
- A single global embedding quality estimate is sufficient — EES provides local, per-cell
  estimates, which is unnecessary for global-only summaries
- The goal is DR quality assessment for a dimensionality-reduction algorithm other
  than t-SNE or UMAP, which are the only DRAs currently supported by the EMBEDR package
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: EMBEDR
  role: Python package whose EMBEDR class generates the data and null embeddings and
    computes the per-cell KL divergence quality statistic
  repo: https://github.com/ejohnson643/EMBEDR
- name: numpy
  role: Loads the high-dimensional matrix (np.loadtxt of mnist2500_X.txt) and handles
    array operations for affinity computation
- name: scipy
  role: Required dependency of the EMBEDR package, used for affinity/KL-divergence
    computations
- name: scikit-learn
  role: Required dependency of the EMBEDR package, used for nearest-neighbor and affinity
    calculations
- name: openTSNE
  role: Custom variation of the openTSNE implementation used to generate the t-SNE
    2D embeddings whose quality is scored
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2022.100443
    title: 'EMBEDR: Distinguishing signal from noise in single-cell omics data'
  source_content_hash: sha256:f7961f3f2cbc03d98f96b4d751e3573f2e3c8daddef995d4200002ea656dd474
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# embedding-quality-statistics

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes the EMBEDR per-cell quality statistic, the empirical embedding statistic (EES), as the Kullback-Leibler divergence between each cell's affinity distributions in the high-dimensional space and in a low-dimensional embedding, for both real-data and null embeddings.

## When to use

Use when per-cell quality scores are needed for 2D embeddings (t-SNE or UMAP) of high-dimensional data such as scRNA-seq or MNIST, to distinguish real structure from algorithm-induced noise.

## When NOT to use

- The dataset is not normalized — the article notes EMBEDR's null-generation process has not been extended to non-normalized datasets, so per-cell KL statistics against such nulls are not supported
- A single global embedding quality estimate is sufficient — EES provides local, per-cell estimates, which is unnecessary for global-only summaries
- The goal is DR quality assessment for a dimensionality-reduction algorithm other than t-SNE or UMAP, which are the only DRAs currently supported by the EMBEDR package

## Inputs

- High-dimensional data matrix (e.g. mnist2500_X.txt, float)
- Pre-generated 2D data embeddings (t-SNE or UMAP)
- Pre-generated 2D null embeddings from marginally-resampled null datasets
- Hyperparameter setting: perplexity (t-SNE) or nearest_neighbors (UMAP)

## Outputs

- Per-cell KL divergence (EES) values for each data embedding
- Per-cell KL divergence values for each null embedding
- Combined table embedr_kl_divergences.csv

## How to apply

Load the high-dimensional matrix (e.g. mnist2500_X.txt via np.loadtxt) along with the pre-generated data and null 2D embeddings from t-SNE or UMAP. Compute Gaussian-kernel affinity neighborhoods for every cell in the high-dimensional space, setting perplexity (or nearest_neighbors for UMAP) at the default of 10% of the number of samples. Compute Student-t kernel affinities among cells in each 2D embedding. For each embedding, calculate the per-cell KL divergence between the high-dimensional and low-dimensional affinity distributions — small DKL means the cell's neighborhood is similarly structured in both spaces. Assemble the per-cell KL divergences across all data embeddings and all null embeddings into a single table (embedr_kl_divergences.csv); these null distributions then bound the resolution of downstream EMBEDR p-values, which are computed empirically from the number of nulls embedded.

## Related tools

- **EMBEDR** (Python package whose EMBEDR class generates the data and null embeddings and computes the per-cell KL divergence quality statistic) — https://github.com/ejohnson643/EMBEDR
- **numpy** (Loads the high-dimensional matrix (np.loadtxt of mnist2500_X.txt) and handles array operations for affinity computation)
- **scipy** (Required dependency of the EMBEDR package, used for affinity/KL-divergence computations)
- **scikit-learn** (Required dependency of the EMBEDR package, used for nearest-neighbor and affinity calculations)
- **openTSNE** (Custom variation of the openTSNE implementation used to generate the t-SNE 2D embeddings whose quality is scored)

## Examples

```
X = np.loadtxt("./data/mnist2500_X.txt").astype(float); embObj = EMBEDR(project_dir='./'); embObj.fit(X); embObj.plot()
```

## Evaluation signals

- Per-cell DKL values are small for cells whose neighborhoods are similarly structured in high- and low-dimensional spaces; large values flag poorly embedded cells
- The output table embedr_kl_divergences.csv contains one row per cell with columns covering every data embedding and every null embedding
- The perplexity (or nearest_neighbors for UMAP) used for affinities equals 10% of the number of samples, matching the package default
- The empirical null distribution of DKL values is bounded in resolution by the number of null embeddings generated; p-values below this bound cannot be resolved
- Coloring the embedding by the derived EMBEDR p-value (via embObj.fit and embObj.plot) should highlight structured regions with low p-values when real signal exists

## Limitations

- There is a finite resolution to the calculated p values since the null distributions are calculated empirically; the number of nulls bounds the lower limit of p values
- No theoretical description of the tails of the null distributions exists, and no principled method for parameterizing the null distribution is available
- EMBEDR's null-generation process has not been extended to non-normalized datasets
- Runtime depends on the speed of current DR algorithm implementations used to generate many null embeddings; GPU extensions were not included

## Evidence

- [methods] The EES is computed per cell by comparing each cell's affinity distributions in the high-dimensional space and in the lower-dimensional embedding using the Kullback-Leibler divergence: [evidence span withheld — anchor status: not_found]
- [methods] small DKL indicates the cell's neighborhood is similarly structured in both spaces: [evidence span withheld — anchor status: not_found]
