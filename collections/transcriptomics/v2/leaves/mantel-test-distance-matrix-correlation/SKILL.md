---
name: mantel-test-distance-matrix-correlation
description: Use when comparing a low-dimensional embedding (e.g. ivis or t-SNE of
  CyTOF data) against the original high-dimensional space to test whether distances
  between manually annotated cell-population centroids are preserved.
when_to_use_negative:
- When the goal is local neighbourhood preservation rather than global inter-cluster
  geometry — other metrics (e.g. ARI after phenograph clustering) address that question
- When cluster annotations are unavailable, since the test requires centroid distances
  between manually annotated cell populations
- When comparing raw single-cell pairwise distances rather than cluster-centroid structure
  — the paper's protocol is defined at the centroid level
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0091
tools:
- name: ivis
  role: Produces the 2D embedding whose global structure preservation is being quantified
    via the Mantel test (k=15, model='maaten' recommended defaults)
  repo: https://github.com/beringresearch/ivis
- name: t-SNE
  role: Baseline embedding method compared against ivis in the same Mantel-test PCC
    evaluation
- name: Python
  role: Implementation environment for subsampling, embedding, centroid distance matrices,
    and Mantel test
  repo: https://github.com/beringresearch/ivis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-019-45301-0
    title: Structure-preserving visualisation of high dimensional single-cell datasets
  source_content_hash: sha256:0aeb0c65232f06b8d4b097e49f1a7f4a243119c2bb1fe64263f11cbdb9885bb3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beringresearch/ivis
  tools:
  - Python
  - ivis
  - t-SNE
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# mantel-test-distance-matrix-correlation

## Summary

Quantifies preservation of global inter-cluster structure after dimensionality reduction by building Euclidean distance matrices between annotated cluster centroids in original and embedding spaces and correlating them with the Mantel test using Pearson's product-moment correlation coefficient (PCC).

## When to use

Use when comparing a low-dimensional embedding (e.g. ivis or t-SNE of CyTOF data) against the original high-dimensional space to test whether distances between manually annotated cell-population centroids are preserved.

## When NOT to use

- When the goal is local neighbourhood preservation rather than global inter-cluster geometry — other metrics (e.g. ARI after phenograph clustering) address that question
- When cluster annotations are unavailable, since the test requires centroid distances between manually annotated cell populations
- When comparing raw single-cell pairwise distances rather than cluster-centroid structure — the paper's protocol is defined at the centroid level

## Inputs

- Preprocessed CyTOF feature matrix (arcsinh-transformed X/5 and MinMaxScaler-scaled to [0,1])
- Manually annotated cell population (cluster) labels per cell
- 2D embedding coordinates from ivis or t-SNE per subsample

## Outputs

- Euclidean distance matrices between cluster centroids in original and embedding spaces
- Per-subsample PCC values from the Mantel test for each embedding method
- Median PCC summary (e.g. written to pcc_ari_benchmark_results.csv)

## How to apply

First reduce each dataset (e.g. BMMC and Samusik CyTOF data preprocessed with arcsinh transformation X/5 followed by MinMaxScaler scaling to [0,1]) to 2D with the embedding method under test. Compute Euclidean distance matrices between centroids of the manually annotated cell populations, once in the original feature space and once in each embedding space. Apply the Mantel test with Pearson's product-moment correlation coefficient (PCC) to assess concordance between the original and embedding distance matrices; repeat over random subsamples (e.g. 100 subsamples of 10,000 cells) to obtain a distribution of PCC values and report the median. Compare medians across methods — in the paper ivis achieved median PCC of 0.76 (BMMC) and 0.73 (Samusik) versus 0.53 and 0.14 for t-SNE — and save per-subsample and median values to a summary CSV.

## Related tools

- **ivis** (Produces the 2D embedding whose global structure preservation is being quantified via the Mantel test (k=15, model='maaten' recommended defaults)) — https://github.com/beringresearch/ivis
- **t-SNE** (Baseline embedding method compared against ivis in the same Mantel-test PCC evaluation)
- **Python** (Implementation environment for subsampling, embedding, centroid distance matrices, and Mantel test) — https://github.com/beringresearch/ivis

## Examples

```
from scipy.spatial.distance import cdist; D_orig = cdist(centroids_orig, centroids_orig); D_emb = cdist(centroids_emb, centroids_emb); pcc = mantel_test(D_orig, D_emb, corr='pearson')
```

## Evaluation signals

- Median PCC over 100 subsamples should approach reported values: ivis-BMMC 0.76, t-SNE-BMMC 0.53, ivis-Samusik 0.73, t-SNE-Samusik 0.14
- Original-space and embedding-space distance matrices must be square matrices over the same set of manually annotated cell-population centroids
- Per-subsample PCC distribution should be non-degenerate (varies across the 100 random 10,000-cell subsamples)
- PCC values should fall in [-1, 1], with values near 0 (as for t-SNE on Samusik) correctly signalling poor global structure preservation
- Summary CSV should contain both per-subsample and median PCC entries for every method/dataset pair

## Limitations

- t-SNE can yield spurious clusters, which degrades its global-structure correlation (median PCC as low as 0.14 on Samusik)
- PCC computed on centroid distances requires reliable manual annotations; unassigned cells (61% unassigned in Levine 32-dim) must be excluded
- Results are estimated over random subsamples, so single-subsample PCC values may not be stable estimates of global structure preservation

## Evidence

- [methods] The level of correlation between the original distance matrix and the distance matrices in the embedding spaces was then assessed using the `Mantel test: "The level of correlation between the original distance matrix and the distance matrices in the embedding spaces was then assessed using the `Mantel test"
- [methods] Random stratified subsampling (n=50) of 1000 observations: "Random stratified subsamples (n=50) of 1000 observations were used to generate a continuum of PCC values for each embedding technique."
- [discussion] Using two synthetic and three CyTOF datasets, we have shown that ivis reduces dimensionality whilst preserving the “global” structure in a dataset: "Using two synthetic and three CyTOF datasets, we have shown that ivis reduces dimensionality whilst preserving the “global” structure in a dataset"
- [readme] Often, ivis performs better at preserving global structure of the data than t-SNE: [evidence span withheld — anchor status: external_doc]
