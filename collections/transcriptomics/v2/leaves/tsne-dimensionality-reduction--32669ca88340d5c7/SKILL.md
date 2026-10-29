---
name: tsne-dimensionality-reduction
description: Use when a raw (dropout-containing), noisy, or imputed scRNA-Seq matrix
  needs a 2-D visualization of cell-type clusters, or when computing average silhouette
  width (ASW) on the t-SNE embedding to compare imputation methods.
when_to_use_negative:
- Data are already an embedded feature representation (e.g. prior PCA-50 coordinates);
  re-applying the full take_norm -> PCA chain would be redundant
- The goal is gene-level interpretation or marker discovery rather than 2-D visualization
  or cluster-structure scoring of cells
- Cluster metrics are needed on the full-dimensional expression space rather than
  on a 2-D visualization embedding
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scikit-learn
  role: Provides PCA(n_components=50), TSNE(n_components=2), and KMeans used in the
    take_norm -> PCA -> t-SNE -> ASW chain
- name: AutoClass
  role: Produces the imputed matrices (via AutoClassImpute) whose embeddings are compared
    against noisy input and true counts
  repo: https://github.com/datapplab/AutoClass
- name: numpy
  role: Array backend for the count matrix (X = expr.values) passed through normalization
    and reduction
- name: matplotlib
  role: Plots the 2-D t-SNE embeddings colored by the simulated group or cell-type
    labels
provenance:
  source_task_ids:
  - task_002
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-29576-y
    title: A universal deep neural network for in-depth cleaning of single-cell RNA-Seq
      data
  source_content_hash: sha256:ec1162838673a6a37b8b3f121df338488f7994c237a9bb205dc8c54f8a20382d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/datapplab/AutoClass
  tools:
  - AutoClass
  - DCA
  - MAGIC
  - Python
  - SAVER
  - matplotlib
  - numpy
  - scImpute
  - scikit-learn
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# tsne-dimensionality-reduction

## Summary

A two-stage embedding routine from the AutoClass paper: normalize the scRNA-Seq count matrix with take_norm, compress to 50 PCA components, then embed with scikit-learn TSNE(n_components=2) to visualize and score cell-type structure on raw, noisy, or imputed data.

## When to use

Use when a raw (dropout-containing), noisy, or imputed scRNA-Seq matrix needs a 2-D visualization of cell-type clusters, or when computing average silhouette width (ASW) on the t-SNE embedding to compare imputation methods.

## When NOT to use

- Data are already an embedded feature representation (e.g. prior PCA-50 coordinates); re-applying the full take_norm -> PCA chain would be redundant
- The goal is gene-level interpretation or marker discovery rather than 2-D visualization or cluster-structure scoring of cells
- Cluster metrics are needed on the full-dimensional expression space rather than on a 2-D visualization embedding

## Inputs

- scRNA-Seq count matrix (CSV, e.g. Dataset1_raw.csv / Dataset1_true.csv), cells x genes
- Imputed count matrix from AutoClassImpute or a control imputer (DCA, SAVER, scImpute, MAGIC)
- Vector of cell-type/group labels (e.g. six simulated Splatter groups, Tian et al. cell-line labels)

## Outputs

- 2-D t-SNE embedding of the normalized, PCA-reduced matrix
- Matplotlib scatter plot of the embedding colored by group labels
- Average silhouette width (ASW) score computed on the t-SNE embedding

## How to apply

Load the count matrix (e.g. Dataset1_raw.csv) with pandas, normalize with take_norm to get X_norm, then reduce with PCA(n_components=50).fit_transform(X_norm) before calling TSNE(n_components=2).fit_transform(PCA_raw). Apply the identical take_norm -> PCA(50) -> t-SNE(2) chain to AutoClass-imputed output, regular-autoencoder ablation output, and the true-count matrix so embeddings are comparable. Color the 2-D embedding by known group labels (e.g. the six simulated Splatter groups or Tian et al. cell-line labels) with matplotlib to visually assess cluster recovery. Optionally run KMeans(n_clusters=K, random_state=1) on the t-SNE coordinates and compute average silhouette width (ASW) against true labels to quantify how well each embedding recovers the expected cell-type pattern.

## Related tools

- **scikit-learn** (Provides PCA(n_components=50), TSNE(n_components=2), and KMeans used in the take_norm -> PCA -> t-SNE -> ASW chain)
- **AutoClass** (Produces the imputed matrices (via AutoClassImpute) whose embeddings are compared against noisy input and true counts) — https://github.com/datapplab/AutoClass
- **numpy** (Array backend for the count matrix (X = expr.values) passed through normalization and reduction)
- **matplotlib** (Plots the 2-D t-SNE embeddings colored by the simulated group or cell-type labels)

## Examples

```
X_norm = take_norm(X); PCA_raw = PCA(n_components=50).fit_transform(X_norm); tsne = TSNE(n_components=2).fit_transform(PCA_raw)
```

## Evaluation signals

- PCA is applied with n_components=50 before t-SNE with n_components=2, matching the paper's exact parameterization
- The same normalization (take_norm) and reduction chain is applied to all matrices being compared (raw, imputed, true counts)
- On Dataset 1, the AutoClass-imputed t-SNE embedding shows six visually separated groups consistent with the Splatter simulation
- ASW computed on the t-SNE embedding is higher for AutoClass output than for regular-autoencoder or control-method (DCA, SAVER, scImpute, MAGIC) output
- KMeans fit on the t-SNE embedding uses a fixed random_state (e.g. random_state=1) for reproducibility

## Limitations

- t-SNE embeddings are stochastic; cluster recovery is judged visually and via ASW, so results depend on random seed and embedding stability
- ASW computed on a 2-D t-SNE embedding is a visualization-derived metric and may not reflect structure in the original high-dimensional space
- The reduction chain is demonstrated on specific data sizes (1000 genes x 500 cells for Dataset 1; 305 cells x 13,426 genes for Tian et al. subsets of 1000/5000 genes); scaling behavior beyond these is not established here

## Evidence

- [results] PCA then t-SNE parameters: "PCA_raw = PCA(n_components=50).fit_transform(X_norm)
tsne_raw = TSNE(n_components=2).fit_transform(PCA_raw)"
- [results] Normalization step preceding reduction: "X_norm = take_norm(X)"
- [results] Clustering on embedding for quality metrics: "kmeans = KMeans(n_clusters = K,random_state=1).fit(tsne)"
- [results] Imputed data being embedded: "res = AutoClassImpute(X,dropout_rate=0.3)"
- [methods] Toolchain including sklearn and matplotlib: "pip install numpy pandas sklearn matplotlib"
- [intro] Purpose of embedding in the ablation: "AutoClass but not the regular autoencoder was able to recover cell type pattern, indicating the classifier component is necessary"
