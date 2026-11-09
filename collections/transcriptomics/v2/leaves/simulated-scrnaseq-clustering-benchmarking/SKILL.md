---
name: simulated-scrnaseq-clustering-benchmarking
description: Use when you need to quantify whether a cell-cell similarity-driven embedding
  (cMAP) improves clustering of simulated scRNA-seq expression matrices over PCA or
  Leiden baselines, using adjusted Rand index against ground-truth cell labels.
when_to_use_negative:
- The dataset contains data points from only one class (no contrast) — CCSF then behaves
  like kernel PCA and offers only linearization and speed benefits over PCA
- The analysis goal is spatial reconstruction requiring marker genes or a reference
  atlas rather than clustering benchmarking
- The data are raw counts that must remain unnormalized for a downstream method that
  specifies its own preprocessing (this pipeline expects z-scored columns with ddof=1)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: ccsf
  role: Learns the cell-cell similarity metric and number of classes (metric_learning),
    computes the cMAP embedding used as the clustering input
  repo: github.com/xinglab-ai/ccsf
- name: scikit-learn
  role: Provides PCA embedding baseline and k-means clustering; pinned to version
    1.1.1 in the install command
- name: SciPy
  role: Column-wise z-score normalization of the expression data via scipy.stats.zscore(axis=0,
    ddof=1)
- name: pandas
  role: Loads the simulated expression matrix and label CSV files and writes the ari_comparison.csv
    table
- name: matplotlib
  role: Produces cMAP/PCA scatter plots colored by cluster or ground-truth label assignments
- name: Splatter
  role: Simulator used to generate the scRNA-seq benchmark datasets with ground-truth
    labels
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2023.100840
    title: Leveraging cell-cell similarity for high-performance spatial and temporal
      cellular mappings from gene expression data
  source_content_hash: sha256:1aba677a5f672548ea3181d921933b3dfc05629c8f9091ffd0373f068d5e3a35
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xinglab-ai/ccsf
  tools:
  - SciPy
  - Splatter
  - ccsf
  - matplotlib
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# simulated-scrnaseq-clustering-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark the CCSF cMAP embedding against PCA (and Leiden) baselines for clustering Splatter-simulated scRNA-seq data, using k-means with the CCSF-learned class count and scoring with adjusted Rand index and clustering accuracy. CCSF reportedly improves ARI by more than 10% over PCA on Splatter simulations.

## When to use

Use when you need to quantify whether a cell-cell similarity-driven embedding (cMAP) improves clustering of simulated scRNA-seq expression matrices over PCA or Leiden baselines, using adjusted Rand index against ground-truth cell labels.

## When NOT to use

- The dataset contains data points from only one class (no contrast) — CCSF then behaves like kernel PCA and offers only linearization and speed benefits over PCA
- The analysis goal is spatial reconstruction requiring marker genes or a reference atlas rather than clustering benchmarking
- The data are raw counts that must remain unnormalized for a downstream method that specifies its own preprocessing (this pipeline expects z-scored columns with ddof=1)

## Inputs

- Splatter-simulated scRNA-seq expression matrix (cells x genes, CSV/numeric matrix)
- Ground-truth cell class labels for the simulated dataset
- Choice of number of random sample points for metric learning (2,000 if dataset is larger)

## Outputs

- ari_comparison.csv with adjusted Rand index and clustering accuracy for CCSF cMAP, PCA, and Leiden clusterings
- cMAP and PCA embedding coordinates (numClass-1 components)
- Scatter plots of cMAP and PCA embeddings colored by cluster/label assignments
- Learned similarity metric string (expected 'correlation') and learned number of classes numClass

## How to apply

Install the pinned stack with 'pip install ccsf scikit-learn==1.1.1'. Load the Splatter-simulated expression matrix and ground-truth labels, then z-score normalize columns with scipy.stats.zscore(axis=0, ddof=1). If the dataset has more than 2,000 data points, randomly sample 2,000 points and learn the similarity metric and number of classes via manifolder.metric_learning (expected metric: 'correlation'). Compute the cMAP embedding with CCSF(n_clusters=numClass, num_comp=numClass-1, metric='correlation') and a scikit-learn PCA embedding of the same normalized data. Cluster both embeddings with k-means at k = learned numClass, additionally run Leiden clustering as a baseline, then compute adjusted Rand index and clustering accuracy against ground truth and write ari_comparison.csv. Judge success when CCSF's ARI exceeds PCA's by more than 10% and cMAP scatter plots (plt.scatter(embedding[:,0], embedding[:,1], c=label.T, s=0.5)) show better class separation.

## Related tools

- **ccsf** (Learns the cell-cell similarity metric and number of classes (metric_learning), computes the cMAP embedding used as the clustering input) — github.com/xinglab-ai/ccsf
- **scikit-learn** (Provides PCA embedding baseline and k-means clustering; pinned to version 1.1.1 in the install command)
- **SciPy** (Column-wise z-score normalization of the expression data via scipy.stats.zscore(axis=0, ddof=1))
- **pandas** (Loads the simulated expression matrix and label CSV files and writes the ari_comparison.csv table)
- **matplotlib** (Produces cMAP/PCA scatter plots colored by cluster or ground-truth label assignments)
- **Splatter** (Simulator used to generate the scRNA-seq benchmark datasets with ground-truth labels)

## Examples

```
from ccsf import CCSF; from scipy import stats; manifolder = CCSF(random_state=0); metric, numClass = manifolder.metric_learning(stats.zscore(data[sampleix], axis=0, ddof=1), verbose=False); embedding = CCSF(n_clusters=numClass, num_comp=numClass-1, metric=metric).cMap(data={'data': data})[0]
```

## Evaluation signals

- metric_learning on the 2,000-cell sample returns 'correlation' as the learned metric
- CCSF cMAP clustering achieves an adjusted Rand index more than 10% higher than the PCA-embedding k-means baseline on the Splatter-simulated data (per Note S8)
- ari_comparison.csv contains rows for CCSF, PCA, and Leiden with both ARI and clustering accuracy values against ground-truth labels
- cMAP scatter plot (embedding[:,0] vs embedding[:,1], c=label.T, s=0.5) shows visibly tighter class separation than the corresponding PCA plot
- Reproducibility holds when the pipeline is re-run with the same rngState=0 random seed

## Limitations

- If the dataset contains data points from only one class (no contrast between points), CCSF works similarly to kernel PCA, and its only benefits over PCA are data linearization and computational speed
- Perfect reconstruction may not hold when space geometry is complex (e.g., tissue cross-section) or gene expression count measurements are inaccurate
- The Python implementation version is not specified in the paper (only MATLAB 2019a), and scikit-learn is pinned to 1.1.1 for the benchmark, so version drift may affect results
- The Leiden clustering resolution is ambiguously reported ('138'), leaving that baseline parameter partially undocumented
- The paper provides no accession numbers or external repository links for the analyzed scRNA-seq datasets; Splatter simulation parameters must come from Note S8

## Evidence

- [supplementary] CCSF improves clustering results by more than 10% in terms of the adjusted Rand (AR) index compared to PCA: [evidence span withheld — anchor status: not_found]
