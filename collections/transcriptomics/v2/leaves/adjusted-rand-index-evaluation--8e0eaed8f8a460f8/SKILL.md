---
name: adjusted-rand-index-evaluation
description: Use when a scRNA-seq or tabular embedding has been clustered (k-means,
  Leiden) and you need a label-based agreement score, such as comparing CCSF cMAP
  clustering against PCA or Leiden baselines on data with known ground-truth cell
  labels.
when_to_use_negative:
- No ground-truth labels exist — the AR index is a supervised agreement measure and
  cannot be computed against unknown classes
- Single-class dataset — CCSF degenerates to kernel PCA-like behavior, so the CCSF-vs-PCA
  AR comparison loses meaning
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: ccsf
  role: Produces the cMAP embedding whose k-means clustering is scored with the AR
    index; metric_learning supplies numClass used as k
  repo: github.com/xinglab-ai/ccsf
- name: scikit-learn
  role: Provides k-means clustering and adjusted Rand index computation (install with
    scikit-learn==1.1.1)
- name: Splatter
  role: Simulates scRNA-seq datasets with ground-truth labels used as the AR index
    reference
- name: pandas
  role: Writes the AR index / clustering accuracy comparison table to ari_comparison.csv
- name: matplotlib
  role: Renders cMAP and PCA scatter plots colored by cluster/label assignments
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2023.100840
    title: Leveraging cell-cell similarity for high-performance spatial and temporal
      cellular mappings from gene expression data
  source_content_hash: sha256:8035b7d8ee57cd3fd5983766825e06100258af46e96f58b64e190da77665eb9e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/xinglab-ai/ccsf
  tools:
  - Splatter
  - ccsf
  - matplotlib
  - pandas
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# adjusted-rand-index-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates and compares clustering quality (e.g., CCSF cMAP vs. PCA or Leiden embeddings of scRNA-seq data) by computing the adjusted Rand (AR) index and clustering accuracy against ground-truth cell labels, the protocol used to show CCSF improves AR index by more than 10% over PCA on Splatter-simulated datasets.

## When to use

Use when a scRNA-seq or tabular embedding has been clustered (k-means, Leiden) and you need a label-based agreement score, such as comparing CCSF cMAP clustering against PCA or Leiden baselines on data with known ground-truth cell labels.

## When NOT to use

- No ground-truth labels exist — the AR index is a supervised agreement measure and cannot be computed against unknown classes
- Single-class dataset — CCSF degenerates to kernel PCA-like behavior, so the CCSF-vs-PCA AR comparison loses meaning

## Inputs

- Cluster label assignments from each embedding (e.g., CCSF cMAP k-means clusters, PCA k-means clusters, Leiden clusters)
- Ground-truth cell label vector (e.g., tcga_label.csv-style label array or Splatter simulation ground truth)
- 2D embeddings (cMAP and PCA) for supporting colored scatter plots

## Outputs

- Adjusted Rand index per clustering method
- Clustering accuracy per method
- Comparison table written to ari_comparison.csv
- Scatter plots of cMAP and PCA embeddings colored by cluster/label assignments

## How to apply

Cluster each candidate embedding with k-means using k equal to the number of classes learned by CCSF.metric_learning (the expected metric is 'correlation'), and optionally also with the Leiden algorithm as a baseline. Compute the adjusted Rand index and clustering accuracy of each clustering against the ground-truth labels (e.g., via scikit-learn on the label vectors). Run the comparison on Splatter-simulated scRNA-seq data where CCSF is reported to improve the AR index by more than 10% over PCA (Note S8). Write the per-method comparison table (e.g., to ari_comparison.csv) so embeddings can be judged by relative AR index, and support it with scatter plots of the cMAP and PCA embeddings colored by cluster/label assignments using plt.scatter(embedding[:,0], embedding[:,1], c=label.T, s=0.5).

## Related tools

- **ccsf** (Produces the cMAP embedding whose k-means clustering is scored with the AR index; metric_learning supplies numClass used as k) — github.com/xinglab-ai/ccsf
- **scikit-learn** (Provides k-means clustering and adjusted Rand index computation (install with scikit-learn==1.1.1))
- **Splatter** (Simulates scRNA-seq datasets with ground-truth labels used as the AR index reference)
- **pandas** (Writes the AR index / clustering accuracy comparison table to ari_comparison.csv)
- **matplotlib** (Renders cMAP and PCA scatter plots colored by cluster/label assignments)

## Examples

```
pip install ccsf scikit-learn==1.1.1; from sklearn.metrics import adjusted_rand_score; ari = adjusted_rand_score(truth_labels, kmeans_labels)
```

## Evaluation signals

- AR index computed for CCSF cMAP clustering should exceed the PCA baseline by more than 10% on Splatter-simulated data, per Note S8
- k for k-means equals numClass returned by CCSF.metric_learning (which should yield 'correlation' as the metric)
- ari_comparison.csv contains one row per method (CCSF, PCA, Leiden) with ARI and clustering accuracy columns
- Scatter plots show cMAP and PCA embeddings colored by label assignments using c=label.T, s=0.5
- Pipeline is reproducible with a fixed rngState (e.g., rngState=0) so repeated runs give identical AR values

## Limitations

- CCSF's advantage over PCA diminishes with no contrast between data points: if the dataset contains data points from only one class, CCSF works similarly to kernel PCA, and its only benefits over PCA are data linearization and computational speed
- The number of classes (numClass) must be chosen or learned, analogous to choosing the number of principal components in PCA; a wrong numClass skews the AR comparison
- Leiden clustering resolution is reported ambiguously in the article ('resolution of 138'), leaving a key baseline parameter uncertain

## Evidence

- [results] On Splatter-simulated datasets, CCSF improves clustering results by more than 10% in terms of the adjusted Rand (AR) index compared to PCA (Note S8).: [evidence span withheld — anchor status: not_found]
- [methods] metric,numClass = manifolder.metric_learning(dataSampled, verbose=False): "metric,numClass = manifolder.metric_learning(dataSampled, verbose=False)"
- [discussion] If the number of data points in the analyzed dataset is more than 2,000, then 2,000 randomly selected data points were used for learning the similarity metric in the first step of CCSF.: "If the number of data points in the analyzed dataset is more than 2,000, then 2,000 randomly selected data points were used for learning the similarity metric in the first step of CCSF."
- [discussion] if there is no contrast between the data points (i.e., the dataset contains data points from only one class), CCSF works similarly to kernel PCA.: "if there is no contrast between the data points (i.e., the dataset contains data points from only one class), CCSF works similarly to kernel PCA."
- [readme] plt.scatter(embedding[:,0],embedding[:,1],c=label.T,s=0.5): "plt.scatter(embedding[:,0],embedding[:,1],c=label.T,s=0.5)"
