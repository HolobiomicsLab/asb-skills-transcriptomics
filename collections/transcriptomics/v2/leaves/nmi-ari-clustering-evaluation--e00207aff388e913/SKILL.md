---
name: nmi-ari-clustering-evaluation
description: Use when quantitatively comparing dimension-reduction or clustering methods
  on labeled scRNA-seq datasets, e.g., scoring k-means on 2D latent representations
  with NMI/ARI against true cell labels.
when_to_use_negative:
- Data lack known cell-type labels — NMI/ARI cannot be computed against ground truth
- Input is raw counts and the goal is dimension reduction itself rather than method
  evaluation (train VASC first, then score its 2D output)
- The number of cell types is unknown, since k-means k is fixed to the known number
  of cell types per dataset
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: VASC
  role: Produces the 2D latent representation that k-means clusters and NMI/ARI score,
    via the VASC class and vase function in vasc.py
  repo: https://github.com/wang-research/VASC
- name: sklearn
  role: Provides k-means clustering, PCA, t-SNE, and sklearn.metrics NMI/ARI computation
    (v0.18.1)
- name: tensorflow
  role: Backend for the VASC deep variational autoencoder (v1.1.0)
- name: keras
  role: Deep-learning backend used to build and train the VASC model (v2.0.6)
  repo: https://github.com/keras-team/keras
- name: h5py
  role: Loads the Hemberg-group expression matrices and labels (v2.7.0)
- name: numpy
  role: Array handling for expression matrices and score aggregation (v1.12.1)
provenance:
  source_task_ids:
  - task_007
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2018.08.003
    title: 'VASC: Dimension Reduction and Visualization of Single-Cell RNA-Seq Data
      by Deep Variational Autoencoder'
  source_content_hash: sha256:6006862a1ca79b64b546f04ee1c948633a00791a3d3f7af4cd72c254cc9a9fb3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wang-research/VASC
  tools:
  - Python
  - VASC
  - h5py
  - keras
  - numpy
  - sklearn
  - tensorflow
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# nmi-ari-clustering-evaluation

## Summary

Benchmark workflow that scores k-means cluster assignments on low-dimensional (2D) representations of scRNA-seq data against known cell-type labels using NMI and ARI, as used to compare VASC against PCA, t-SNE, ZIFA, and SIMLR on 20 Hemberg-group datasets.

## When to use

Use when quantitatively comparing dimension-reduction or clustering methods on labeled scRNA-seq datasets, e.g., scoring k-means on 2D latent representations with NMI/ARI against true cell labels.

## When NOT to use

- Data lack known cell-type labels — NMI/ARI cannot be computed against ground truth
- Input is raw counts and the goal is dimension reduction itself rather than method evaluation (train VASC first, then score its 2D output)
- The number of cell types is unknown, since k-means k is fixed to the known number of cell types per dataset

## Inputs

- scRNA-seq expression count matrices for the Hemberg-group datasets (read via h5py/numpy)
- True cell-type label vectors for each dataset
- 2D latent representations produced by VASC (vasc.py) and baselines (PCA, t-SNE, ZIFA, SIMLR)

## Outputs

- Per-dataset NMI and ARI scores for each method (sklearn.metrics on k-means vs. true labels)
- Aggregated comparison table, e.g., nmi_ari_benchmark.csv, with mean ranks across the 20 datasets
- Down-sampling sensitivity table, e.g., pollen_downsampling_nmi_ari.csv

## How to apply

Load the expression matrix and true cell labels for each dataset with h5py and numpy, then obtain a 2D representation from each method: VASC via the VASC class / vase function in vasc.py (TensorFlow 1.1.0 / Keras 2.0.6 deep variational autoencoder using the Gumbel distribution to model dropout events) and the baselines (PCA, t-SNE via sklearn 0.18.1, ZIFA, SIMLR). Apply k-means (sklearn) to each 2D representation using the known number of cell types as k. Compute normalized mutual information (NMI) and adjusted rand index (ARI) between k-means assignments and true labels with sklearn.metrics. Because VASC's stochastic optimization introduces run-to-run variation, repeat runs and aggregate per-dataset NMI/ARI into a comparison table (e.g., nmi_ari_benchmark.csv) with mean ranks per method before judging performance.

## Related tools

- **VASC** (Produces the 2D latent representation that k-means clusters and NMI/ARI score, via the VASC class and vase function in vasc.py) — https://github.com/wang-research/VASC
- **sklearn** (Provides k-means clustering, PCA, t-SNE, and sklearn.metrics NMI/ARI computation (v0.18.1))
- **tensorflow** (Backend for the VASC deep variational autoencoder (v1.1.0))
- **keras** (Deep-learning backend used to build and train the VASC model (v2.0.6)) — https://github.com/keras-team/keras
- **h5py** (Loads the Hemberg-group expression matrices and labels (v2.7.0))
- **numpy** (Array handling for expression matrices and score aggregation (v1.12.1))

## Examples

```
from sklearn.metrics import normalized_mutual_info_score, adjusted_rand_score; nmi = normalized_mutual_info_score(labels, km.labels_); ari = adjusted_rand_score(labels, km.labels_)
```

## Evaluation signals

- NMI and ARI values fall in [0, 1], with ARI allowed to reach 0 for chance-level clustering
- k-means k equals the known number of cell types for each dataset
- On the 20-dataset benchmark, VASC should rank in the top two methods on every dataset and be best on ~15 datasets for NMI and ~17 for ARI
- Down-sampling sensitivity check: NMI/ARI on the Pollen dataset remain comparable when cell sampling exceeds 50%
- Repeated runs are consistent within expected stochastic variation, per the paper's recommendation to repeat stochastic optimization

## Limitations

- Stochastic optimization in VASC learning introduces run-to-run variation in the dimension reduction; repeated runs are recommended, so single-run NMI/ARI scores may not be reproducible
- VASC model learning takes several hours on a single-GPU desktop for datasets with several thousand cells
- Determination of the optimal latent dimension is difficult without prior knowledge; the Pollen analysis found no improvement beyond a latent dimension of 2

## Evidence

- [intro] intro_comparison: "We systematically compared VASC with several state-of-the-art dimension reduction methods on 20 datasets."
- [intro] gumbel_dropout: "In addition, it uses the Gumbel distribution to better model the zero and near-zero dropout events."
- [methods] vasc_code_layout: "vasc.py: contains a class VASC and a function vase"
- [readme] demo_files: "We gave a demo.py and config.py to demonstrate the use of VASC."
- [discussion] stochastic_variation: "The stochastic optimization algorithms, used in the VASC model learning, introduce variations in the dimension reduction. Repeated runs are thus recommended"
- [discussion] runtime_cost: "it costs several hours for the VASC model learning using a desktop-level computer with single GPU card"
