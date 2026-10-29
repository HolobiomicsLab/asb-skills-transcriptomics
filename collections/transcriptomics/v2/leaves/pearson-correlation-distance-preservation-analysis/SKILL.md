---
name: pearson-correlation-distance-preservation-analysis
description: Use when evaluating whether a 2D embedding of high-dimensional single-cell
  (CyTOF/scRNA-seq) data retains global inter-cluster geometry, using manually annotated
  cell-population labels and centroid distance concordance.
when_to_use_negative:
- When only local neighbourhood preservation matters — the PCC of centroid distances
  measures global inter-cluster structure, not fine-grained local structure
- When no manually annotated cell populations are available, since centroid distances
  require labelled clusters
- For evaluating label transfer or classification accuracy directly — the paper complements
  this with SVM classification accuracy and phenograph ARI, which this skill does
  not compute
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0092
tools:
- name: ivis
  role: Produces the 2D embedding whose global structure preservation is scored by
    the centroid-distance PCC; used with k=15 and model='maaten' defaults
  repo: github.com/beringresearch/ivis
- name: t-SNE
  role: Baseline embedding method compared against ivis via median PCC of centroid
    distances
- name: Python
  role: Implementation language for preprocessing (arcsinh, MinMaxScaler), embedding,
    and Mantel-test PCC computation
  repo: https://github.com/beringresearch/ivis
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41598-019-45301-0
    title: Structure-preserving visualisation of high dimensional single-cell datasets
  source_content_hash: sha256:4fc16f641ea24435637c44a9e2f26d5372994ed7481c54e95922044023a973a0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - ivis
  - t-SNE
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# pearson-correlation-distance-preservation-analysis

## Summary

Quantifies how well a dimensionality-reduction embedding (e.g. ivis or t-SNE) preserves global inter-cluster structure by correlating Euclidean distance matrices of manually annotated cell-population centroids in original vs. embedding space using a Mantel test with Pearson's product-moment correlation coefficient (PCC).

## When to use

Use when evaluating whether a 2D embedding of high-dimensional single-cell (CyTOF/scRNA-seq) data retains global inter-cluster geometry, using manually annotated cell-population labels and centroid distance concordance.

## When NOT to use

- When only local neighbourhood preservation matters — the PCC of centroid distances measures global inter-cluster structure, not fine-grained local structure
- When no manually annotated cell populations are available, since centroid distances require labelled clusters
- For evaluating label transfer or classification accuracy directly — the paper complements this with SVM classification accuracy and phenograph ARI, which this skill does not compute

## Inputs

- High-dimensional single-cell expression matrix (CyTOF protein expression, e.g. BMMC or Samusik benchmark datasets)
- Manually annotated cell-population (cluster) labels for the cells
- 2D embedding coordinates from ivis or t-SNE fitted on preprocessed subsamples

## Outputs

- Per-subsample PCC values from the Mantel test comparing original-space and embedding-space centroid distance matrices
- Median PCC per dataset/embedding method (e.g. ivis-BMMC 0.76, t-SNE-Samusik 0.14) summarised in pcc_ari_benchmark_results.csv

## How to apply

Preprocess CyTOF data with an arcsinh transformation (X = np.arcsinh(X/5)) followed by MinMaxScaler scaling to the [0, 1] range. Draw random subsamples (100 x 10,000 cells) per dataset and compute 2D ivis embeddings (k=15, model='maaten') and t-SNE embeddings on each subsample. Compute Euclidean distance matrices between centroids of the manually annotated cell populations in the original high-dimensional space and in each embedding space, then assess concordance with a Mantel test using Pearson's product-moment correlation coefficient (PCC). Aggregate the median PCC across subsamples; for the BMMC and Samusik CyTOF benchmarks the reported medians were ivis-BMMC 0.76, t-SNE-BMMC 0.53, ivis-Samusik 0.73, t-SNE-Samusik 0.14, demonstrating ivis better preserves global structure.

## Related tools

- **ivis** (Produces the 2D embedding whose global structure preservation is scored by the centroid-distance PCC; used with k=15 and model='maaten' defaults) — github.com/beringresearch/ivis
- **t-SNE** (Baseline embedding method compared against ivis via median PCC of centroid distances)
- **Python** (Implementation language for preprocessing (arcsinh, MinMaxScaler), embedding, and Mantel-test PCC computation) — https://github.com/beringresearch/ivis

## Examples

```
X = MinMaxScaler().fit_transform(np.arcsinh(X/5)); emb = Ivis(embedding_dims=2, k=15, model='maaten').fit_transform(X)
```

## Evaluation signals

- Median PCC for ivis on BMMC should be near 0.76 (t-SNE near 0.53) and for Samusik near 0.73 (t-SNE near 0.14) when reproducing the paper's benchmark
- Per-subsample PCC values should be distributed across all 100 random 10,000-cell subsamples before taking the median
- Distance matrices must be built from centroids of the manually annotated populations, not from all cells pairwise
- Data must be arcsinh-transformed (X/5) and MinMax-scaled to [0,1] before embedding; skipping this changes PCC outcomes
- PCC values for ivis should exceed t-SNE values on the same subsamples if global structure preservation is confirmed

## Limitations

- PCC on centroid distances captures only global inter-cluster geometry; ivis's robustness claims to hyperparameter settings are referenced via supplementary figures rather than detailed in the main text
- t-SNE's failure to preserve global structure can yield spurious clusters with clean boundaries in the presence of random noise, so PCC comparisons should be interpreted alongside cluster-quality metrics such as phenograph ARI
- Results depend on the subsampling regime (100 x 10,000-cell random subsamples); different subsample sizes or counts will shift median PCC values

## Evidence

- [methods] The level of correlation between the original distance matrix and the distance matrices in the embedding spaces was then assessed using the `Mantel test: "The level of correlation between the original distance matrix and the distance matrices in the embedding spaces was then assessed using the `Mantel test"
- [methods] arcsinh + minmax preprocessing: "Prior to preprocessing, values in both datasets were scaled to [0, 1] range."
- [readme] ivis preserves global structure better than t-SNE: [evidence span withheld — anchor status: external_doc]
- [abstract] ivis preserves global data structures in a low-dimensional space: "Results on simulated and real datasets demonstrate that ivis preserves global data structures in a low-dimensional space"
