---
name: kmeans-centroid-optimization
description: Use when reducing high-dimensional gene expression (scRNA-seq or TCGA-style)
  matrices to a low-dimensional cell-cell similarity representation via CCSF, specifically
  for the centroid-localization step that defines P cell groups.
when_to_use_negative:
- 'The dataset contains data points from only one class (no contrast between data
  points): CCSF then behaves like kernel PCA and offers only linearization and speed
  benefits over PCA'
- Input is already a low-dimensional embedding or spatial coordinate table rather
  than a gene expression matrix
- The analysis goal is spatial reconstruction with marker genes or a reference atlas
  — novoSpaRc alone is the appropriate route, not CCSF centroid optimization
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3518
tools:
- name: ccsf
  role: 'Python package implementing CCSF: metric_learning returns the metric and
    numClass, and CCSF(n_clusters=numClass, num_comp=numClass-1, metric=metric) drives
    centroid localization and cMap embedding'
  repo: github.com/xinglab-ai/ccsf
- name: scikit-learn
  role: Provides k-means++ initialization/optimization used to locate the P centroids
    and cluster quality indices for metric selection
- name: scipy
  role: scipy.stats.zscore (axis=0, ddof=1) z-scores the sampled expression matrix
    per feature before metric learning and centroid optimization
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2023.100840
    title: Leveraging cell-cell similarity for high-performance spatial and temporal
      cellular mappings from gene expression data
  source_content_hash: sha256:6768fc5d497f77643b36227c52e52759e9be0532bf6cc647bfa49a62c06cea93
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xinglab-ai/ccsf
  tools:
  - ccsf
  - numpy
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# kmeans-centroid-optimization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stage 2 of the CCSF framework: locate P class centroids by minimizing the distance between centroids and all data points using k-means++ optimization, then compute the cell-centroid similarity matrix that seeds the Gram-matrix and class-discriminative optimization stages.

## When to use

Use when reducing high-dimensional gene expression (scRNA-seq or TCGA-style) matrices to a low-dimensional cell-cell similarity representation via CCSF, specifically for the centroid-localization step that defines P cell groups.

## When NOT to use

- The dataset contains data points from only one class (no contrast between data points): CCSF then behaves like kernel PCA and offers only linearization and speed benefits over PCA
- Input is already a low-dimensional embedding or spatial coordinate table rather than a gene expression matrix
- The analysis goal is spatial reconstruction with marker genes or a reference atlas — novoSpaRc alone is the appropriate route, not CCSF centroid optimization

## Inputs

- Gene expression matrix (e.g., TCGA tcga_data.csv loaded with pandas, converted to numpy array)
- Number of classes/centroids P (numClass) and learned metric (e.g., 'correlation') from metric_learning
- z-scored data sample (scipy.stats.zscore axis=0, ddof=1) for metric learning when dataset exceeds 2,000 points

## Outputs

- Cell-centroid similarity matrix D (M cells x P centroids)
- Cell-cell similarity matrix S_t (Gram matrix of D)
- Q = P-1 class-discriminative CCSF components and 2D cMap embedding

## How to apply

First determine P (numClass) and the metric via the preceding metric_learning step (e.g., metric_learning on n=2,000 z-scored points sampled with random_state=0 returns 'correlation' and numClass for TCGA). Then run k-means++ initialization/optimization to locate P centroids minimizing the distance between centroids and all data points, and compute the cell-centroid similarity matrix D. Center D and take its Gram matrix to obtain the cell-cell similarity matrix S_t, which feeds the class-discriminative optimization solved with two SVDs forming W = U*Sigma^-1*B. Instantiate the pipeline as CCSF(n_clusters=numClass, num_comp=np.array([numClass-1]), metric='correlation') so that num_comp = numClass-1 class-discriminative components (Q = P-1) are produced for the cMap embedding.

## Related tools

- **ccsf** (Python package implementing CCSF: metric_learning returns the metric and numClass, and CCSF(n_clusters=numClass, num_comp=numClass-1, metric=metric) drives centroid localization and cMap embedding) — github.com/xinglab-ai/ccsf
- **scikit-learn** (Provides k-means++ initialization/optimization used to locate the P centroids and cluster quality indices for metric selection)
- **scipy** (scipy.stats.zscore (axis=0, ddof=1) z-scores the sampled expression matrix per feature before metric learning and centroid optimization)

## Examples

```
manifolder = CCSF(n_clusters=numClass, num_comp=np.array([numClass-1]), metric='correlation'); embedding_CMAP = manifolder.cMap(data={'data': data})
```

## Evaluation signals

- metric_learning on the sampled, z-scored TCGA data (n=2,000, random_state=0) returns 'correlation' as the selected metric
- Number of CCSF components equals numClass-1 (Q = P-1 class-discriminative components)
- cMap embedding produces a 2D scatter (cMAP1 vs cMAP2) colored by cell labels
- Reproducibility check: 1,000 different CCSF initializations yield little change (<1%) in DEMaP for zebrafish data and no change in Drosophila spatial reconstruction
- Pin scikit-learn==1.1.1 per the README installation instruction for reproducible k-means++ behavior

## Limitations

- If the dataset contains data points from only one class, CCSF works similarly to kernel PCA; its only benefits over PCA are data linearization and computational speed
- Perfect spatial reconstruction may not hold when space geometry is complex (e.g., tissue cross-section) or gene expression count measurements are inaccurate
- Datasets with more than 2,000 data points must be subsampled to 2,000 randomly selected points for learning the similarity metric
- Python implementation version is not specified in the paper (only MATLAB 2019a); reproducibility requires the README-pinned scikit-learn==1.1.1

## Evidence

- [intro] Locate P centroids by minimizing the distance between them and all the data points: "(2) P centroids are located by minimizing the distance between them and all the data points."
- [intro] Gram matrix of the cell-centroid similarity yields the cell-cell similarity matrix: "(4) The cell-cell similarity matrix is obtained by computing the gram matrix of the cell-centroid similarity."
- [intro] Class-discriminative optimization reduces dimensionality on the similarity matrix: "(5) A class-discriminative optimization is performed on the similarity matrix to reduce the data dimensionality."
- [discussion] Subsample to 2,000 points for learning the similarity metric: "If the number of data points in the analyzed dataset is more than 2,000, then 2,000 randomly selected data points were used for learning the similarity metric in the first step of CCSF."
- [discussion] Single-class dataset degrades CCSF to kernel PCA: "if there is no contrast between the data points (i.e., the dataset contains data points from only one class), CCSF works similarly to kernel PCA."
- [readme] README instantiation of CCSF with correlation metric and cMap embedding: "manifolder = CCSF(n_clusters=numClass, num_comp=num_comp,metric=metric)
embedding_CMAP = manifolder.cMap(data=feed_data)"
