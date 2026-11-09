---
name: clustering-agreement-metric-computation
description: Use when you need to score clustering quality of 2D/low-dimensional embeddings
  (e.g., from VASC) against known cell-type labels, or to compare score distributions
  across repeated stochastic runs (e.g., with vs. without a ZI layer).
when_to_use_negative:
- No ground-truth cell labels are available for the dataset — NMI/ARI require reference
  labels and cannot be computed
- The goal is comparing embeddings directly without clustering (e.g., 2D visualization
  assessment of rare cell populations) rather than label agreement
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scikit-learn (sklearn)
  role: k-means clustering of embeddings and NMI/ARI computation via sklearn.metrics
- name: VASC
  role: produces the low-dimensional embeddings whose k-means clusters are scored;
    used with and without the zero-inflated (Gumbel-based) layer for 20 repeated runs
    per dataset
  repo: github.com/wang-research/VASC
- name: Keras
  role: deep-learning backend used to train the VASC variational autoencoder whose
    stochastic optimization produces the repeated-run score distributions
  repo: keras-team/keras
- name: NumPy
  role: array handling of count matrices and score vectors during preprocessing and
    aggregation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2018.08.003
    title: 'VASC: Dimension Reduction and Visualization of Single-Cell RNA-Seq Data
      by Deep Variational Autoencoder'
  source_content_hash: sha256:851357b25081678c8a03f0b8d822dbadb7609b52051e97adb940b4ff3588662a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Keras
  - NumPy
  - Python
  - VASC
  - h5py
  - scikit-learn
  - scikit-learn (sklearn)
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# clustering-agreement-metric-computation

## Summary

Quantify the agreement between cell-cluster assignments recovered from low-dimensional embeddings of scRNA-seq data and known cell labels using NMI (normalized mutual information) and ARI (adjusted Rand index) from scikit-learn, aggregated over repeated runs to assess clustering performance and stability.

## When to use

Use when you need to score clustering quality of 2D/low-dimensional embeddings (e.g., from VASC) against known cell-type labels, or to compare score distributions across repeated stochastic runs (e.g., with vs. without a ZI layer).

## When NOT to use

- No ground-truth cell labels are available for the dataset — NMI/ARI require reference labels and cannot be computed
- The goal is comparing embeddings directly without clustering (e.g., 2D visualization assessment of rare cell populations) rather than label agreement

## Inputs

- 2D/low-dimensional cell embeddings from VASC (one embedding per independent run)
- Known cell labels for each dataset (e.g., Biase, Goolam, Pollen, Yan scRNA-seq datasets)
- Number of clusters (set to the number of known cell types, e.g., k for k-means)

## Outputs

- Per-run NMI and ARI scores (sklearn.metrics) comparing k-means clusters to known labels
- Aggregated 20-run NMI/ARI distributions per condition per dataset
- Paired boxplots (e.g., replicating the Figure 4C layout) of NMI/ARI with vs. without the ZI layer

## How to apply

For each run, cluster the learned 2D/low-dimensional embedding (e.g., with k-means from scikit-learn) and compare the predicted cluster assignments against the known cell labels using sklearn.metrics NMI and ARI. Repeat this per independent training run — in the VASC ablation study 20 repeated runs per dataset (Biase, Goolam, Pollen, Yan) were used because the stochastic optimization of the variational autoencoder introduces run-to-run variation. Aggregate the 20-run NMI/ARI score distributions per condition and compare them (e.g., paired boxplots of with-ZI vs. without-ZI variants). Judge success by a higher median and tighter interquartile spread of NMI/ARI in the favored condition, matching the reported pattern where the ZI layer improved stability and average performance on three of four datasets.

## Related tools

- **scikit-learn (sklearn)** (k-means clustering of embeddings and NMI/ARI computation via sklearn.metrics)
- **VASC** (produces the low-dimensional embeddings whose k-means clusters are scored; used with and without the zero-inflated (Gumbel-based) layer for 20 repeated runs per dataset) — github.com/wang-research/VASC
- **Keras** (deep-learning backend used to train the VASC variational autoencoder whose stochastic optimization produces the repeated-run score distributions) — keras-team/keras
- **NumPy** (array handling of count matrices and score vectors during preprocessing and aggregation)

## Examples

```
from sklearn.cluster import KMeans; from sklearn.metrics import normalized_mutual_info_score as nmi, adjusted_rand_score as ari; labels = KMeans(n_clusters=k, n_init=20).fit_predict(embedding); s_nmi, s_ari = nmi(cell_labels, labels), ari(cell_labels, labels)
```

## Evaluation signals

- Each run yields finite NMI and ARI values in [0, 1] (ARI may be slightly negative for chance-level clustering) computed against the known cell labels
- With-ZI condition shows higher median and tighter interquartile spread of NMI/ARI than without-ZI on three of four datasets (Biase, Goolam, Pollen, Yan), replicating Figure 4C
- Score distributions are based on 20 independent runs per dataset per condition, as repeated runs are recommended given stochastic optimization
- Number of k-means clusters matches the number of known cell types for each dataset

## Limitations

- The stochastic optimization algorithms used in VASC model learning introduce variations in the dimension reduction; a single run is unreliable and repeated runs are needed for a consensus
- Only one small dataset (Biase) ships with the repository; the other datasets (Goolam, Pollen, Yan) must be sourced separately
- VASC model learning can take several hours per run on a single-GPU desktop for datasets with several thousand cells, so a 20-run x 2-condition protocol is compute-intensive

## Evidence

- [discussion] The stochastic optimization algorithms, used in the VASC model learning, introduce variations in the dimension reduction. Repeated runs are thus recommended: "The stochastic optimization algorithms, used in the VASC model learning, introduce variations in the dimension reduction. Repeated runs are thus recommended"
- [results] The inclusion of the ZI layer improved both the stability and the average performances of VASC on three of the four tested datasets: [evidence span withheld — anchor status: not_found]
- [methods] vasc.py: contains a class VASC and a function vase: "vasc.py: contains a class VASC and a function vase"
- [readme] sklearn 0.18.1 prerequisite for clustering metrics: "+ sklearn 0.18.1
+ tensorflow 1.1.0
+ keras 2.0.6"
- [readme] A small dataset from Biase is included for demonstration.: "A small dataset from Biase is included for demonstration."
