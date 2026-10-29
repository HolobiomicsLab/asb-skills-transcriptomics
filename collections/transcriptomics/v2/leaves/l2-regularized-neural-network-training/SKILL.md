---
name: l2-regularized-neural-network-training
description: Use when you need to train a neural autoencoder on preprocessed (log2(x+1),
  variance=1, mean=1) single-cell expression data to infer circular/periodic pseudo-time,
  and MSE+L2 regularization is the fitting objective.
when_to_use_negative:
- The process is non-periodic / linear (e.g. a developmental trajectory), where linear
  pseudo-time methods such as Monocle or Wanderlust — not a circular-manifold autoencoder
  — are appropriate; Cyclum targets 'latent periodic' processes
- You need marker-gene-based cell-cycle phase scoring (e.g. Cyclone or scanpy/Seurat
  CellCycleScoring style); Cyclum's methodology explicitly relies on the circular
  manifold instead of marker genes, though it can refine external phase labels afterwards
- 'Input data is raw/unnormalized counts: the least-square fit assumes the log2(x+1)
  transform with variance = 1, mean = 1 scaling, so training directly on raw counts
  violates the model''s expectations'
edam_operation: http://edamontology.org/operation_3937
edam_topics:
- http://edamontology.org/topic_3862
- http://edamontology.org/topic_3517
tools:
- name: Cyclum
  role: Package whose cyclum.models.ae module builds the tanh-encoder / cos-sin-decoder
    autoencoder trained with the MSE + L2 objective
  repo: https://github.com/KChen-lab/Cyclum.git
- name: Keras
  role: Framework used to construct, compile, and train (and serialize to .h5) the
    L2-regularized sinusoidal autoencoder
  repo: https://keras.io/
- name: TensorFlow
  role: Backend (v1.x, tested at 1.14.0) on which the Keras autoencoder training executes,
    CPU or GPU
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:8bc56bfbfbb62bb59c6ead91c929f9c5bc18270e51cc0f5b232a5335a734e6db
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/Cyclum.git
  tools:
  - Cyclum
  - Keras
  - TensorFlow
  edam_operation: http://edamontology.org/operation_3937
  edam_topics:
  - http://edamontology.org/topic_3862
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# l2-regularized-neural-network-training

## Summary

Trains Cyclum's sinusoidal autoencoder — a tanh-activated MLP encoder whose output feeds a cosine/sine decoder layer mapped linearly back to gene space — with a mean-squared-error objective plus L2 weight regularization, implemented in Keras with TensorFlow, to embed scRNA-seq cells on a circular manifold and recover periodic pseudo-time.

## When to use

Use when you need to train a neural autoencoder on preprocessed (log2(x+1), variance=1, mean=1) single-cell expression data to infer circular/periodic pseudo-time, and MSE+L2 regularization is the fitting objective.

## When NOT to use

- The process is non-periodic / linear (e.g. a developmental trajectory), where linear pseudo-time methods such as Monocle or Wanderlust — not a circular-manifold autoencoder — are appropriate; Cyclum targets 'latent periodic' processes
- You need marker-gene-based cell-cycle phase scoring (e.g. Cyclone or scanpy/Seurat CellCycleScoring style); Cyclum's methodology explicitly relies on the circular manifold instead of marker genes, though it can refine external phase labels afterwards
- Input data is raw/unnormalized counts: the least-square fit assumes the log2(x+1) transform with variance = 1, mean = 1 scaling, so training directly on raw counts violates the model's expectations

## Inputs

- Preprocessed scRNA-seq expression matrix (cells x genes), log2(x+1)-transformed, scaled to variance = 1 and centered to mean = 1 (e.g. the mESC dataset)
- Autoencoder architecture specification: tanh MLP encoder + cosine/sine first decoder layer + linear output layer
- GPU-enabled or CPU environment with TensorFlow 1.x (tested: tensorflow 1.14.0, keras 2.2.4, numpy 1.16.5, h5py 2.9.0)

## Outputs

- Trained Keras autoencoder model serialized to an .h5 file (cyclum_ae_model.h5)
- Circular latent (encoder) coordinates for each cell
- Per-cell pseudo-time wrapped via modulus into [0, 2π], saved as pseudotime_2pi.csv

## How to apply

Preprocess the expression matrix with log2(x+1), then scale to variance = 1 and center to mean = 1, which the Cyclum FAQ recommends as working well. Build the encoder as a multi-layer perceptron with tanh activations via the cyclum.models.ae module; build the decoder so its first layer computes cosine and sine activations of the encoder output and a subsequent linear layer maps back to gene-expression space. Compile the network with a least-square (MSE) objective plus L2 weight regularization in Keras with TensorFlow — this is the key step distinguishing this skill from plain autoencoder training, since L2 regularizes weights to stabilize the fit to the predefined circular manifold, making the model robust to random noise and small sample sizes. Train on the preprocessed matrix until reconstruction loss converges ( Cyclum was tested with keras 2.2.4 / tensorflow 1.14.0; force those versions in setup.py if newer dependencies fail). Save the trained model to a .h5 file (e.g. cyclum_ae_model.h5) and export the wrapped pseudo-time, modulus-folded into [0, 2π], as pseudotime_2pi.csv.

## Related tools

- **Cyclum** (Package whose cyclum.models.ae module builds the tanh-encoder / cos-sin-decoder autoencoder trained with the MSE + L2 objective) — https://github.com/KChen-lab/Cyclum.git
- **Keras** (Framework used to construct, compile, and train (and serialize to .h5) the L2-regularized sinusoidal autoencoder) — https://keras.io/
- **TensorFlow** (Backend (v1.x, tested at 1.14.0) on which the Keras autoencoder training executes, CPU or GPU)

## Examples

```
conda create -n cyclum python=3.7 pip; conda activate cyclum; git clone https://github.com/KChen-lab/Cyclum.git; cd Cyclum; pip install .
```

## Evaluation signals

- cyclum_ae_model.h5 exists and is loadable by Keras (keras.models.load_model succeeds)
- Every value in pseudotime_2pi.csv falls within [0, 2π] (modulus-wrapped invariant)
- Reconstruction (mean-squared-error) loss decreases monotonically and converges during training on the preprocessed mESC matrix
- Inferred latent coordinates trace a closed circular trajectory consistent across cells (pseudo-time is one value per cell and cyclically consistent)
- Runtime sanity check: a dataset of 12,280 cells x 33,694 genes should train in roughly 40 min on a laptop GPU (GTX 960M, 2 GB graphic memory)

## Limitations

- Cyclum was tested only with specific dependency versions (python 3.7.4, keras 2.2.4, tensorflow 1.14.0, numpy 1.16.5); pip may install latest versions that were not tested, and TensorFlow < 1.15.2 has a known security issue
- The paper reports no version numbers for Python/Keras/TensorFlow in the article text itself, and hardware is specified only informally (GTX 960M laptop, 2 GB graphic memory) with detailed runtime relegated to Supplementary Note 2
- One pseudo-time value per cell is not meaningful in isolation — it must be interpreted relative to other cells along the circular manifold

## Evidence

- [methods] how_to_apply preprocessing: "scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well."
- [discussion] how_to_apply implementation stack: "The Cyclum package is efficiently implemented in Python using Keras with TensorFlow and has been comprehensively tested."
- [readme] how_to_apply rationale (sinusoidal autoencoder): [evidence span withheld — anchor status: external_doc]
- [discussion] evaluation_signals runtime benchmark: "Cyclum can analyze an scRNA-seq dataset consisting of 12,280 cells 33,694 genes in 40 min on a laptop computer with a GTX 960M GPU and 2 GB graphic memory"
- [readme] limitations version pinning: [evidence span withheld — anchor status: external_doc]
