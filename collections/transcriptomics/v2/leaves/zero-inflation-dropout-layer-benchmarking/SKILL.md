---
name: zero-inflation-dropout-layer-benchmarking
description: Use when you must quantify whether an explicit dropout-modeling component
  improves reproducibility of a VAE's low-dimensional embeddings, e.g. comparing NMI/ARI
  boxplots over ≥20 repeated runs on small scRNA-seq datasets like Biase, Goolam,
  Pollen, and Yan.
when_to_use_negative:
- Input data is a large 10x Genomics dataset with thousands of cells rather than the
  small datasets used for this ablation, since the claim being tested is specific
  to small-dataset stability
- The goal is a one-shot evaluation of a single model fit — the article notes stochastic
  optimization introduces run-to-run variation, so single runs cannot support the
  stability comparison
- The model architecture being benchmarked contains no dropout/zeros-modeling layer
  to ablate
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3577
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: VASC
  role: Deep variational autoencoder under ablation; trained with the Gumbel-based
    zero-inflation layer enabled and disabled across repeated runs
  repo: github.com/wang-research/VASC
- name: keras
  role: Deep-learning framework used to implement and train the VASC variational autoencoder
    for each of the 20 repeated runs per condition
  repo: https://github.com/keras-team/keras
- name: sklearn
  role: Provides k-means clustering of embeddings and NMI/ARI metrics for scoring
    each run
- name: h5py
  role: Loads the scRNA-seq count matrices for the four small datasets
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.gpb.2018.08.003
    title: 'VASC: Dimension Reduction and Visualization of Single-Cell RNA-Seq Data
      by Deep Variational Autoencoder'
  source_content_hash: sha256:b3081a2b82935de5c83f3202bca114858a89e287f1bab8e0ba14c34bd8c03152
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python
  - VASC
  - h5py
  - keras
  - scikit-learn
  - sklearn
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3577
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# zero-inflation-dropout-layer-benchmarking

## Summary

Ablation-benchmarking skill for the VASC variational autoencoder: repeatedly train the scRNA-seq dimension-reduction model with and without its Gumbel-distribution zero-inflation (dropout) layer and compare NMI/ARI clustering stability across runs on small datasets.

## When to use

Use when you must quantify whether an explicit dropout-modeling component improves reproducibility of a VAE's low-dimensional embeddings, e.g. comparing NMI/ARI boxplots over ≥20 repeated runs on small scRNA-seq datasets like Biase, Goolam, Pollen, and Yan.

## When NOT to use

- Input data is a large 10x Genomics dataset with thousands of cells rather than the small datasets used for this ablation, since the claim being tested is specific to small-dataset stability
- The goal is a one-shot evaluation of a single model fit — the article notes stochastic optimization introduces run-to-run variation, so single runs cannot support the stability comparison
- The model architecture being benchmarked contains no dropout/zeros-modeling layer to ablate

## Inputs

- scRNA-seq count matrices for the Biase, Goolam, Pollen, and Yan datasets (h5py/numpy-readable)
- Known cell-type labels for each dataset for external clustering validation
- VASC source code (vasc.py with class VASC, helpers.py, demo.py, config.py)

## Outputs

- 20-run NMI/ARI score distributions for ZI-enabled and ZI-disabled conditions on each of the four datasets
- Paired boxplots replicating the Figure 4C layout
- Median and interquartile-range comparison demonstrating stability gains on 3 of 4 datasets

## How to apply

Load the Biase, Goolam, Pollen, and Yan scRNA-seq count matrices with h5py/numpy, apply gene filtering and normalization to log-expression, and feed them to the VASC class in vasc.py. Train the full model, which 'uses the Gumbel distribution to better model the zero and near-zero dropout events', for 20 independent runs per dataset using the stochastic variational-autoencoder optimization in TensorFlow/Keras; then train the ablated model with the ZI layer disabled under identical settings. For each run, cluster the 2D/low-dimensional embedding (e.g. k-means from scikit-learn) against known cell labels and compute NMI and ARI with sklearn.metrics. Aggregate the 20-run NMI/ARI distributions for with-ZI vs without-ZI conditions per dataset and render paired boxplots replicating Figure 4C. Accept the reproduction only if the ZI-enabled variant shows a higher median and tighter interquartile spread on three of the four datasets, as reported.

## Related tools

- **VASC** (Deep variational autoencoder under ablation; trained with the Gumbel-based zero-inflation layer enabled and disabled across repeated runs) — github.com/wang-research/VASC
- **keras** (Deep-learning framework used to implement and train the VASC variational autoencoder for each of the 20 repeated runs per condition) — https://github.com/keras-team/keras
- **sklearn** (Provides k-means clustering of embeddings and NMI/ARI metrics for scoring each run)
- **h5py** (Loads the scRNA-seq count matrices for the four small datasets)

## Examples

```
import h5py; from vasc import VASC; model = VASC(data, zi_layer=True); emb = model.train(); scores = [nmi_score(labels, kmeans(emb).labels_) for _ in range(20)]
```

## Evaluation signals

- ZI-enabled VASC shows higher median NMI/ARI than the ZI-disabled variant on three of the four datasets (Biase, Goolam, Pollen, Yan)
- NMI/ARI interquartile spread is tighter (more stable) for the ZI-enabled condition on the same 3-of-4 datasets
- Each condition consists of exactly 20 independent runs per dataset, computed with identical preprocessing and training settings
- Boxplots reproduce the paired, per-dataset layout of the paper's Figure 4C
- Preprocessing yields gene-filtered, log-normalized expression matrices as valid input to the VASC class

## Limitations

- Stochastic optimization in VASC model learning introduces run-to-run variation in the dimension reduction; repeated runs are required and a consensus should be taken
- The ZI-layer benefit does not hold on all four datasets — it improves stability and average performance on only three of four
- VASC model learning costs several hours on a single-GPU desktop for datasets with several thousand cells, so 20-run ablations are compute-heavy
- Pinned legacy dependency versions (tensorflow 1.1.0, keras 2.0.6, sklearn 0.18.1, h5py 2.7.0, numpy 1.12.1) may not run on modern environments
- Determining the optimal latent dimension is difficult without prior knowledge, which can confound the ablation if dimensions differ between conditions

## Evidence

- [intro] ZI layer improves stability on small datasets: "In addition, it uses the Gumbel distribution to better model the zero and near-zero dropout events."
- [discussion] Stochastic optimization requires repeated runs: "The stochastic optimization algorithms, used in the VASC model learning, introduce variations in the dimension reduction. Repeated runs are thus recommended"
- [methods] Software dependencies and VASC class: "vasc.py: contains a class VASC and a function vase"
- [readme] README lists version-pinned prerequisites: "+ Python 3.5+
+ numpy 1.12.1
+ h5py 2.7.0
+ sklearn 0.18.1
+ tensorflow 1.1.0
+ keras 2.0.6"
- [discussion] Compute cost caveat: "it costs several hours for the VASC model learning using a desktop-level computer with single GPU card"
