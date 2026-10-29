---
name: neighborhood-correspondence-statistical-testing
description: Use when you need to statistically judge whether local structures in
  a t-SNE/UMAP embedding reflect real high-dimensional signal rather than stochastic
  DR distortion, e.g. to color cells by embedding quality or pick DR hyperparameters.
when_to_use_negative:
- Datasets that are not normalized — the article notes 'EMBEDR's null-generation process
  has not been extended to non-normalized datasets', so quality analyses of entire
  unnormalized pipelines are out of scope
- When you need very small p values with high resolution — p values have finite empirical
  resolution bounded by the number of null embeddings, so the number of nulls you
  can afford sets the lower bound on p
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: EMBEDR
  role: Python package implementing the empirical marginal-resampling hypothesis test;
    its EMBEDR class fit method generates data and null embeddings and computes per-sample
    p values, and its plot method renders the p-value-colored embedding
  repo: github.com/ejohnson643/EMBEDR
- name: openTSNE
  role: Backend for the t-SNE dimensionality reduction algorithm whose embeddings
    are evaluated by the neighborhood correspondence test (custom variation of the
    openTSNE implementation)
- name: numpy
  role: Loads and holds the input count matrix (e.g. np.loadtxt of the Tabula Muris
    or MNIST feature table)
- name: scipy
  role: Installation dependency of the EMBEDR package used in the empirical statistical
    comparison
- name: scikit-learn
  role: Installation dependency of the EMBEDR package (nearest-neighbor computation
    for EES)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2022.100443
    title: 'EMBEDR: Distinguishing signal from noise in single-cell omics data'
  source_content_hash: sha256:1a04ec6159c309357e100c7af851e518c43f37a8fef79ea207d173af6c4afc5e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/ejohnson643/EMBEDR
  tools:
  - EMBEDR
  - Python
  - numpy
  - openTSNE
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# neighborhood-correspondence-statistical-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Performs an empirical hypothesis test on dimensionally-reduced embeddings by comparing each sample's neighborhood correspondence between the high-dimensional space and its projection against a null distribution generated via marginal resampling, yielding per-sample EMBEDR p values that separate real structure from algorithm-induced noise.

## When to use

Use when you need to statistically judge whether local structures in a t-SNE/UMAP embedding reflect real high-dimensional signal rather than stochastic DR distortion, e.g. to color cells by embedding quality or pick DR hyperparameters.

## When NOT to use

- Datasets that are not normalized — the article notes 'EMBEDR's null-generation process has not been extended to non-normalized datasets', so quality analyses of entire unnormalized pipelines are out of scope
- When you need very small p values with high resolution — p values have finite empirical resolution bounded by the number of null embeddings, so the number of nulls you can afford sets the lower bound on p

## Inputs

- High-dimensional feature matrix (e.g. scRNA-seq counts, ~4,800 Tabula Muris bone marrow cells from GEO GSE109774, loaded as a numpy array; MNIST 2500 as mnist2500_X.txt)
- Embeddings from a dimensionality reduction algorithm (t-SNE via openTSNE or UMAP)
- Marginally-resampled null data set and its embeddings

## Outputs

- Per-sample EMBEDR p values in [0, 1] (e.g. saved as a table of per-cell p-values)
- Embedding plot colored by EMBEDR p-value (e.g. embedr_pvalue_embedding.png)
- Empirical null EES* distribution used for the hypothesis test

## How to apply

Fit the EMBEDR object to generate several data embeddings (n_data_embed, recommended 3-10) plus at least one marginally-resampled null data set embedding (n_null_embed, typically 1). For each sample, compute the EES (neighborhood correspondence between the original high-dimensional space and each projection), compare the observed EES against the aggregated null EES* distribution by taking the fraction of null statistics smaller than the observed statistic, and average the resulting empirical p values across the Nembed data embeddings to get a single per-sample quality metric pi. Perplexity/nearest_neighbors should default to 10% of the number of samples, and the openTSNE-backed t-SNE implementation is used. Render the embedding colored by the per-cell p values (as in Figure 2D of the paper's Tabula Muris bone marrow analysis).

## Related tools

- **EMBEDR** (Python package implementing the empirical marginal-resampling hypothesis test; its EMBEDR class fit method generates data and null embeddings and computes per-sample p values, and its plot method renders the p-value-colored embedding) — github.com/ejohnson643/EMBEDR
- **openTSNE** (Backend for the t-SNE dimensionality reduction algorithm whose embeddings are evaluated by the neighborhood correspondence test (custom variation of the openTSNE implementation))
- **numpy** (Loads and holds the input count matrix (e.g. np.loadtxt of the Tabula Muris or MNIST feature table))
- **scipy** (Installation dependency of the EMBEDR package used in the empirical statistical comparison)
- **scikit-learn** (Installation dependency of the EMBEDR package (nearest-neighbor computation for EES))

## Examples

```
embObj = EMBEDR(project_dir='./', DRA='tsne', n_data_embed=3, n_null_embed=1); embObj.fit(X); embObj.plot()
```

## Evaluation signals

- All per-sample EMBEDR p values lie within [0, 1]
- The output figure embedr_pvalue_embedding.png exists and shows cells colored by p value (cf. Figure 2D)
- The fraction of null EES* statistics smaller than each observed EES statistic is computed consistently across all Nembed data embeddings before averaging
- Number of data embeddings generated matches n_data_embed (3-10 recommended) and at least n_null_embed=1 null embedding exists before p-value calculation
- Perplexity (or nearest_neighbors for UMAP) is set at 10% of the number of samples if the default was used

## Limitations

- There is a finite resolution to the calculated p values since null distributions are computed empirically; the number of nulls that can be embedded determines the lower bound on achievable p values
- No theoretical description of the tails of the null distributions exists, and there is no principled method for parameterizing the null distribution
- Runtime depends on the speed of the underlying DR algorithm implementations used to generate many null embeddings; GPU extensions and quadratic rate optimization schemes were not included in this work

## Evidence

- [full_text] Each cell's EES statistic is compared against the aggregated null EES∗ distribution: [evidence span withheld — anchor status: not_found]
