---
name: variational-autoencoder-model-training
description: Use when a raw or preprocessed scRNA-seq expression matrix (e.g. h5/H5-loaded
  counts from Hemberg-group, Biase, Goolam, Pollen, Yan, or 10x Genomics datasets)
  must be reduced to a 2D-or-larger latent representation while explicitly modeling
  technical dropout events.
when_to_use_negative:
- Input is not scRNA-seq count data with dropout events but already a clean low-dimensional
  or feature table
- The exact latent dimension must be chosen automatically without prior knowledge
  — the paper notes 'The determination of the optimal dimension is a tricky task if
  prior knowledge is limited'
- A single fast run is required — model learning costs several hours for datasets
  with several thousand cells on a single-GPU desktop
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: VASC
  role: 'Core model: vasc.py provides the class VASC and function vase implementing
    the deep variational autoencoder with Gumbel dropout modeling'
  repo: https://github.com/wang-research/VASC
- name: tensorflow
  role: Backend (version 1.1.0) for the stochastic variational optimization of the
    autoencoder
- name: keras
  role: Backend (version 2.0.6) for building and training the encoder/decoder/ZI-layer
    network
  repo: https://github.com/keras-team/keras
- name: h5py
  role: Loading expression matrices (version 2.7.0) and saving trained weights to
    vasc_model.h5
- name: Anaconda
  role: Recommended distribution to install Python 3.5+, numpy, h5py, sklearn, tensorflow
    and keras prerequisites
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_007
  - task_002
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2018.08.003
    title: 'VASC: Dimension Reduction and Visualization of Single-Cell RNA-Seq Data
      by Deep Variational Autoencoder'
  source_content_hash: sha256:6aab8e29bd85fc3340d864f02d6d44ad2aa3b8674f38fbf65c2721071ca4a20b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/wang-research/VASC
  tools:
  - Anaconda
  - Python
  - VASC
  - h5py
  - keras
  - numpy
  - scikit-learn
  - sklearn
  - tensorflow
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# variational-autoencoder-model-training

## Summary

Train the VASC deep variational autoencoder (vasc.py class VASC / function vase) to learn a nonlinear, dropout-aware low-dimensional representation of scRNA-seq count data, using a Gumbel-based zero-inflation layer. This is the core model-fitting step underlying VASC dimension reduction, visualization, and downstream NMI/ARI clustering benchmarks.

## When to use

Use when a raw or preprocessed scRNA-seq expression matrix (e.g. h5/H5-loaded counts from Hemberg-group, Biase, Goolam, Pollen, Yan, or 10x Genomics datasets) must be reduced to a 2D-or-larger latent representation while explicitly modeling technical dropout events.

## When NOT to use

- Input is not scRNA-seq count data with dropout events but already a clean low-dimensional or feature table
- The exact latent dimension must be chosen automatically without prior knowledge — the paper notes 'The determination of the optimal dimension is a tricky task if prior knowledge is limited'
- A single fast run is required — model learning costs several hours for datasets with several thousand cells on a single-GPU desktop

## Inputs

- scRNA-seq expression matrix (counts or normalized log-expression) loaded with h5py/numpy, e.g. the Biase demo dataset or the Petropoulos E-MTAB-3929 embryo matrix (1529 cells)
- Known cell labels (for supervised post-training evaluation via k-means NMI/ARI, not used during training)
- Down-sampled variants of the input (10%-100% of cells or 5000-300000 reads per cell) for sensitivity training

## Outputs

- Trained VASC model weights saved to vasc_model.h5
- 2D (or user-chosen dimension, e.g. 2-20) latent embedding of all cells
- Per-epoch training-loss trajectory (e.g. loss_curve.png) showing convergence
- Run-wise NMI/ARI scores from k-means on the latent space for stability assessment

## How to apply

Instantiate the VASC class from vasc.py on a TensorFlow 1.1.0 / Keras 2.0.6 backend: build the 3-layer encoder (512-128-32 units, ReLU activations, L1 regularization), Gaussian latent sampling via the re-parameterization trick, and a mirrored decoder with a sigmoid output layer. Enable the zero-inflation layer that models dropout with Gumbel-softmax (temperature tau in the 0.5-1 range), since this improved stability and average NMI/ARI on three of four small datasets (Biase, Goolam, Pollen, Yan). Compile with RMSprop at learning rate 0.0001 and train for approximately 100 epochs, recording loss per epoch to confirm convergence. For datasets with several thousand cells, expect several hours on a single-GPU desktop; for small sample sizes, consider fixing the co-variance matrix. Because stochastic optimization introduces run-to-run variation, repeat runs (e.g. 20 repeats for stability analyses) and take consensus before interpreting the latent embedding.

## Related tools

- **VASC** (Core model: vasc.py provides the class VASC and function vase implementing the deep variational autoencoder with Gumbel dropout modeling) — https://github.com/wang-research/VASC
- **tensorflow** (Backend (version 1.1.0) for the stochastic variational optimization of the autoencoder)
- **keras** (Backend (version 2.0.6) for building and training the encoder/decoder/ZI-layer network) — https://github.com/keras-team/keras
- **h5py** (Loading expression matrices (version 2.7.0) and saving trained weights to vasc_model.h5)
- **Anaconda** (Recommended distribution to install Python 3.5+, numpy, h5py, sklearn, tensorflow and keras prerequisites)

## Examples

```
from vasc import VASC; model = VASC(x_train); model.train(lr=0.0001, epochs=100); model.save_weights('vasc_model.h5')
```

## Evaluation signals

- Training loss converges to a lower, monotone-decreasing limit within roughly the first 100 epochs
- Repeated runs (e.g. 20 repeats) yield consistently converged loss curves and low run-to-run variance; with the ZI layer enabled, NMI/ARI medians are higher and interquartile spreads tighter on 3 of 4 small datasets
- k-means on the learned latent space reproduces reported NMI/ARI (e.g. VASC best on 15/20 datasets for NMI and 17/20 for ARI on Hemberg-group datasets)
- Down-sampling on Pollen shows stable NMI/ARI when ≥50% of cells are sampled
- Saved vasc_model.h5 loads and reproduces a qualitatively consistent 2D embedding (e.g. E3-E7 stage ordering on Petropoulos embryos) across repeated training runs

## Limitations

- Stochastic optimization introduces run-to-run variation in the dimension reduction; repeated runs are recommended to reach a consensus
- Model learning costs several hours for datasets with several thousand cells on a single-GPU desktop computer
- Determining the optimal latent dimension is difficult without prior knowledge; when sample size is small it is better to fix the co-variance matrix
- Specific version pins (tensorflow 1.1.0, keras 2.0.6, sklearn 0.18.1) are required by the reference implementation and may not work with modern Keras 3

## Evidence

- [methods] vasc.py class and vase function: "vasc.py: contains a class VASC and a function vase"
- [intro] Gumbel dropout modeling: "In addition, it uses the Gumbel distribution to better model the zero and near-zero dropout events."
- [discussion] Run-to-run variation caveat: "The stochastic optimization algorithms, used in the VASC model learning, introduce variations in the dimension reduction. Repeated runs are thus recommended"
- [readme] Prerequisites from README: "+ Python 3.5+
+ numpy 1.12.1
+ h5py 2.7.0
+ sklearn 0.18.1
+ tensorflow 1.1.0
+ keras 2.0.6"
- [methods] Demo entry points: "We gave a demo.py and config.py to demonstrate the use of VASC."
