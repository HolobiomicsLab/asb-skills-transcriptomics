---
name: cell-type-colored-embedding-visualization
description: Use when a 2D ivis embedding of scRNA-seq data has been computed and
  you need to visually validate cell-type segregation and global cluster relationships
  against published figures.
when_to_use_negative:
- The embedding has no annotated cell-type labels, so coloring and cluster-segregation
  validation cannot be performed
- The goal is quantifying local neighborhood accuracy rather than visual/structural
  inspection — use the Mantel test or SVM classifier evaluation instead of scatter
  plots alone
- t-SNE or other DR outputs with random-noise sensitivity are being interpreted —
  the paper warns t-SNE 'tends to yield spurious clusters with clean boundaries',
  so plots should not be assumed structure-preserving without ivis
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ivis
  role: Computes the 2D embedding being visualized; run with k=15, model='maaten'
    for datasets under 500,000 observations
  repo: https://github.com/beringresearch/ivis
- name: Python
  role: Language for scatter-plotting embeddings colored by cell type with matplotlib
    and checking output PNGs
  repo: https://github.com/beringresearch/ivis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-019-45301-0
    title: Structure-preserving visualisation of high dimensional single-cell datasets
  source_content_hash: sha256:e2d92ca255e56afb8fa9e21207c46a4c21cea35ecad00e9a29989ee942b65d9d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/beringresearch/ivis
  tools:
  - Python
  - ivis
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# cell-type-colored-embedding-visualization

## Summary

Generates 2D scatter plots of ivis embeddings of scRNA-seq datasets (hippocampus, metastatic melanoma, retinal bipolar) with cells colored by annotated cell type, to verify that cell types form distinct clusters and that global relationships (e.g. hippocampal information flow from dentate gyrus to CA3 and CA1) are preserved.

## When to use

Use when a 2D ivis embedding of scRNA-seq data has been computed and you need to visually validate cell-type segregation and global cluster relationships against published figures.

## When NOT to use

- The embedding has no annotated cell-type labels, so coloring and cluster-segregation validation cannot be performed
- The goal is quantifying local neighborhood accuracy rather than visual/structural inspection — use the Mantel test or SVM classifier evaluation instead of scatter plots alone
- t-SNE or other DR outputs with random-noise sensitivity are being interpreted — the paper warns t-SNE 'tends to yield spurious clusters with clean boundaries', so plots should not be assumed structure-preserving without ivis

## Inputs

- 2D ivis embedding coordinates (numpy array, n_observations x 2)
- Annotated cell-type labels per observation (e.g. hippocampus, melanoma, or retinal bipolar scRNA-seq datasets)
- Preprocessed expression matrix (MinMaxScaler [0,1]-scaled; melanoma reduced to PCA n=50)

## Outputs

- PNG scatter plot of the 2D embedding colored by cell type
- Qualitative assessment of cell-type cluster segregation and preservation of global relationships
- Optional Mantel test Pearson correlation between embedding and original distance matrices

## How to apply

After preprocessing (MinMaxScaler scaling to [0, 1], and PCA-50 reduction for high-dimensional datasets such as the melanoma SCP11 dataset of 4,645 observations x 23,686 features), train ivis with the recommended hyperparameters for datasets under 500,000 observations (k=15, model='maaten') to obtain a 2D embedding. Scatter-plot the embedding coordinates with matplotlib, coloring each cell by its annotated cell-type label. Judge correctness qualitatively by confirming cell types form distinct, non-artificially-packed clusters that preserve global relationships — e.g. the hippocampus embedding should capture the flow of sensory information from the dentate gyrus to CA3 and CA1 nuclei via mutual cell proximities, and retinal bipolar cell types should segregate as in the published figure. Where labels permit, complement the visual check with a Mantel test of distance-matrix correlation (Pearson correlation coefficient) against the original PCA space. Confirm each generated PNG exists before reporting.

## Related tools

- **ivis** (Computes the 2D embedding being visualized; run with k=15, model='maaten' for datasets under 500,000 observations) — https://github.com/beringresearch/ivis
- **Python** (Language for scatter-plotting embeddings colored by cell type with matplotlib and checking output PNGs) — https://github.com/beringresearch/ivis

## Examples

```
from ivis import Ivis; from sklearn.preprocessing import MinMaxScaler; X_scaled = MinMaxScaler().fit_transform(X); emb = Ivis(embedding_dims=2, k=15).fit_transform(X_scaled); plt.scatter(emb[:,0], emb[:,1], c=labels)
```

## Evaluation signals

- Each generated PNG exists on disk before reporting
- Cell types form distinct clusters that are not artificially packed in the 2D embedding
- Hippocampus embedding shows dentate gyrus → CA3 → CA1 cluster proximities matching the paper's information-flow finding
- Retinal bipolar cell types segregate as in the published retinal bipolar figure
- Mantel test Pearson correlation between embedding distance matrix and original (PCA-space) distance matrix is positive where labels permit

## Limitations

- Validation is largely qualitative; the paper supplements it with Mantel tests using random stratified subsamples (n=50) of 1000 observations
- t-SNE-based visualizations can mislead: 'in the presence of random noise, the algorithm tends to yield spurious clusters with clean boundaries', so visual inspection alone is not proof of structure preservation
- Cell-type coloring requires pre-existing annotations; unlabelled cells (as in 61% of the Levine-32-dim dataset) cannot contribute to the qualitative check

## Evidence

- [discussion] For instance, ivis embeddings of the hippocampus dataset captured distinct nuclei clusters that correspond to known cell types and anatomical regions: "For instance, ivis embeddings of the hippocampus dataset captured distinct nuclei clusters that correspond to known cell types and anatomical regions"
- [methods] Prior to preprocessing, values in both datasets were scaled to [0, 1] range.: "Prior to preprocessing, values in both datasets were scaled to [0, 1] range."
- [methods] The level of correlation between the original distance matrix and the distance matrices in the embedding spaces was then assessed using the `Mantel test`: "The level of correlation between the original distance matrix and the distance matrices in the embedding spaces was then assessed using the `Mantel test`"
- [discussion] the algorithm tends to yield spurious clusters with clean boundaries, potentially hindering accurate interpretation and discovery (Fig. 1C): "the algorithm tends to yield spurious clusters with clean boundaries, potentially hindering accurate interpretation and discovery (Fig. 1C)"
- [readme] model = Ivis(embedding_dims=2, k=15): "model = Ivis(embedding_dims=2, k=15)"
