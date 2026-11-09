---
name: repeated-run-stability-boxplot-visualization
description: Use when a stochastic deep model's dimension reduction varies across
  runs (e.g. VASC with variational/Gumbel-softmax optimization) and you must show
  whether an added component, such as the ZI dropout layer, improves median clustering
  scores and tightens their spread.
when_to_use_negative:
- The model or optimizer is deterministic (no stochastic optimization), so repeated-run
  variation and stability boxplots are meaningless
- The dataset is not small-cell-count scRNA-seq and the question is not stability
  of a low-dimensional embedding against known labels; VASC costs several hours on
  a single GPU for thousands of cells
- The comparison involves other dimension reduction methods (PCA, t-SNE, ZIFA) rather
  than ablation of the ZI layer within VASC
edam_operation: http://edamontology.org/operation_0571
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3170
tools:
- name: VASC
  role: Provides the deep variational autoencoder class (vasc.py) trained with and
    without the Gumbel-based zero-inflated layer for the 20 repeated runs per dataset
  repo: github.com/wang-research/VASC
- name: sklearn
  role: k-means clustering of the low-dimensional embedding and sklearn.metrics computation
    of NMI and ARI per run
- name: h5py
  role: Reads the raw scRNA-seq count matrices (Biase, Goolam, Pollen, Yan) stored
    as h5 files
- name: keras
  role: Backend on which the VASC variational autoencoder and its stochastic optimization
    are implemented
  repo: github.com/keras-team/keras
- name: numpy
  role: Gene filtering, log-expression normalization, and aggregation of the 20-run
    score arrays
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2018.08.003
    title: 'VASC: Dimension Reduction and Visualization of Single-Cell RNA-Seq Data
      by Deep Variational Autoencoder'
  source_content_hash: sha256:6574efb72c84ab1ac31cec1440b462341f1d3215f24e36a91de9a00153c688dc
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - VASC
  - h5py
  - keras
  - numpy
  - scikit-learn
  - sklearn
  edam_operation: http://edamontology.org/operation_0571
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# Repeated-run stability boxplot visualization

## Summary

Quantifies the run-to-run stability of VASC's stochastic dimension reduction by repeating training with and without the zero-inflated (Gumbel) ZI layer across small scRNA-seq datasets (Biase, Goolam, Pollen, Yan) and rendering paired NMI/ARI boxplots, replicating the paper's Figure 4C comparison.

## When to use

Use when a stochastic deep model's dimension reduction varies across runs (e.g. VASC with variational/Gumbel-softmax optimization) and you must show whether an added component, such as the ZI dropout layer, improves median clustering scores and tightens their spread.

## When NOT to use

- The model or optimizer is deterministic (no stochastic optimization), so repeated-run variation and stability boxplots are meaningless
- The dataset is not small-cell-count scRNA-seq and the question is not stability of a low-dimensional embedding against known labels; VASC costs several hours on a single GPU for thousands of cells
- The comparison involves other dimension reduction methods (PCA, t-SNE, ZIFA) rather than ablation of the ZI layer within VASC

## Inputs

- Raw scRNA-seq count matrices (h5 files read via h5py) for the Biase, Goolam, Pollen, and Yan datasets
- Known per-cell label vectors for each dataset
- Trained VASC embeddings from the vasc.py VASC class, 20 runs per dataset per condition (ZI layer enabled vs. disabled)

## Outputs

- Per-run NMI and ARI scores for each dataset and each ZI-layer condition (20 runs each)
- Paired boxplots of NMI/ARI distributions replicating the Figure 4C layout
- Comparison verdict: number of datasets (out of 4) where ZI-enabled VASC has higher median and tighter IQR

## How to apply

Load the Biase, Goolam, Pollen, and Yan scRNA-seq count matrices with h5py/NumPy and preprocess them (gene filtering plus normalization to log-expression) before feeding the VASC class in vasc.py. Train the full VASC model with the Gumbel-based zero-inflated layer enabled for 20 independent runs per dataset, then train the ablated model with the ZI layer disabled under identical TensorFlow/Keras settings. On each run, cluster the low-dimensional embedding with k-means (scikit-learn) against the known cell labels and compute NMI and ARI via sklearn.metrics. Aggregate the 20-run NMI/ARI distributions per condition per dataset and draw paired boxplots following the Figure 4C layout. Accept the reproduction when the ZI-enabled VASC shows a higher median and tighter interquartile spread on at least three of the four datasets, consistent with the paper's claim that the ZI layer improved stability and average performance on three of four datasets.

## Related tools

- **VASC** (Provides the deep variational autoencoder class (vasc.py) trained with and without the Gumbel-based zero-inflated layer for the 20 repeated runs per dataset) — github.com/wang-research/VASC
- **sklearn** (k-means clustering of the low-dimensional embedding and sklearn.metrics computation of NMI and ARI per run)
- **h5py** (Reads the raw scRNA-seq count matrices (Biase, Goolam, Pollen, Yan) stored as h5 files)
- **keras** (Backend on which the VASC variational autoencoder and its stochastic optimization are implemented) — github.com/keras-team/keras
- **numpy** (Gene filtering, log-expression normalization, and aggregation of the 20-run score arrays)

## Examples

```
from sklearn.cluster import KMeans; from sklearn.metrics import normalized_mutual_info_score, adjusted_rand_score; km = KMeans(n_clusters=k).fit(embedding); nmi = normalized_mutual_info_score(labels, km.labels_); ari = adjusted_rand_score(labels, km.labels_)
```

## Evaluation signals

- Exactly 20 independent runs per dataset per condition (with-ZI and without-ZI) are completed for each of Biase, Goolam, Pollen, and Yan
- NMI and ARI are computed with sklearn.metrics against the known cell labels for every run, not averaged over runs before clustering
- ZI-enabled VASC shows a higher median NMI/ARI than the ZI-disabled variant on three of the four datasets
- ZI-enabled NMI/ARI boxplots have a visibly tighter interquartile spread than ZI-disabled boxplots on the same three datasets
- The rendered figure's layout matches the paired per-dataset boxplot structure of Figure 4C

## Limitations

- The stochastic optimization used in VASC model learning introduces variation in the dimension reduction, so repeated runs are required before any stability conclusion can be drawn
- The reported improvement holds on only three of the four datasets (Biase, Goolam, Pollen, Yan), so failure to reproduce on a single dataset is not disqualifying
- VASC model learning costs several hours on a desktop with a single GPU for datasets with several thousand cells, limiting how many repeated runs are practical on large data

## Evidence

- [discussion] Stochastic optimization run-to-run variation: "The stochastic optimization algorithms, used in the VASC model learning, introduce variations in the dimension reduction. Repeated runs are thus recommended"
- [intro] ZI layer improves stability on three of four datasets: [evidence span withheld — anchor status: not_found]
- [methods] VASC code structure for the runs: "vasc.py: contains a class VASC and a function vase"
- [methods] Required package versions for the workflow: "+ sklearn 0.18.1
+ tensorflow 1.1.0
+ keras 2.0.6"
- [readme] Small dataset included for demonstration: "A small dataset from Biase is included for demonstration."
