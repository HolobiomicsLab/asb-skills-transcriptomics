---
name: keras-model-definition-and-persistence
description: Use when a circular (periodic) pseudo-time model must be defined as a
  Keras autoencoder with cos/sin decoder activations, trained on preprocessed scRNA-seq
  data, and saved as a loadable .h5 artifact.
when_to_use_negative:
- The trajectory in the data is linear rather than periodic — Cyclum's circular manifold
  and sinusoidal decoder are designed for latent periodic processes (e.g., cell cycle),
  not linear pseudo-time.
- You only need cell-cycle phase scores without a trained circular model — use scanpy.tl.score_genes_cell_cycle
  or Seurat::CellCycleScoring, or refine their labels with Cyclum's postproc.refine_labels
  instead of building a new network.
- Your environment mandates TensorFlow 2.x — the Cyclum stack was tested on TF 1.14.0
  (latest tested 1.15.2), and older versions are flagged for a known security issue
  only below 1.15.2.
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: Cyclum
  role: Provides the cyclum.models.ae module used to build the tanh encoder and cos/sin
    decoder, plus CyclumAutoTune to select the proper number of linear components
    for the most circular manifold; handles saving the trained model to .h5.
  repo: https://github.com/KChen-lab/cyclum
- name: Keras
  role: Framework in which the sinusoidal autoencoder is rewritten, compiled (MSE
    loss + L2 regularization), trained, and saved/loaded as an .h5 artifact.
  repo: https://keras.io/
- name: TensorFlow
  role: Backend for Keras model definition, training, and persistence; must be a TF
    1.x release (1.14.0 tested, 1.15.2 latest tested) for Cyclum compatibility.
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:c533979ca453c8a6d4423acb7580bfaa41c0d34641b7871310ac9fdd30c40c81
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/cyclum
  tools:
  - Cyclum
  - Keras
  - TensorFlow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# keras-model-definition-and-persistence

## Summary

Defines Cyclum's sinusoidal autoencoder in Keras/TensorFlow — a tanh-activation MLP encoder whose output feeds a first decoder layer computing cosine and sine activations, followed by a linear layer back to gene expression space — trains it with a least-square (MSE) objective plus L2 weight regularization, and persists the trained network to an .h5 file for reload and validation.

## When to use

Use when a circular (periodic) pseudo-time model must be defined as a Keras autoencoder with cos/sin decoder activations, trained on preprocessed scRNA-seq data, and saved as a loadable .h5 artifact.

## When NOT to use

- The trajectory in the data is linear rather than periodic — Cyclum's circular manifold and sinusoidal decoder are designed for latent periodic processes (e.g., cell cycle), not linear pseudo-time.
- You only need cell-cycle phase scores without a trained circular model — use scanpy.tl.score_genes_cell_cycle or Seurat::CellCycleScoring, or refine their labels with Cyclum's postproc.refine_labels instead of building a new network.
- Your environment mandates TensorFlow 2.x — the Cyclum stack was tested on TF 1.14.0 (latest tested 1.15.2), and older versions are flagged for a known security issue only below 1.15.2.

## Inputs

- Preprocessed mESC scRNA-seq expression matrix (log2(x+1) transform, variance = 1, mean = 1)
- Autoencoder architecture specification from the cyclum.models.ae module (tanh MLP encoder; cos/sin first decoder layer + linear second decoder layer)
- Training configuration: MSE (least-square) objective with L2 weight regularization

## Outputs

- Trained Keras model saved as cyclum_ae_model.h5 (loadable via keras.models.load_model)
- Wrapped circular pseudo-time per cell in [0, 2π], written to pseudotime_2pi.csv

## How to apply

First preprocess the mESC scRNA-seq matrix with log2(x+1), then scale to variance = 1 and center to mean = 1, since 'scaled (variance = 1), centered (mean = 1) log-transformed (log2(x+1)) data work well'. Build the encoder as a multi-layer perceptron with tanh activations using cyclum.models.ae; build the decoder so its first layer computes cosine and sine activations of the encoder output and a subsequent linear layer maps back to gene expression space — this sinusoidal activation is what folds the latent space onto a circular manifold. Compile with a least-square (MSE) objective plus L2 weight regularization in Keras with TensorFlow, and train until the reconstruction loss converges. Persist the trained model to cyclum_ae_model.h5 (Keras .h5, backed by h5py==2.9.0 in the tested stack) and reload it with keras.models.load_model to confirm the artifact is intact. Pin the tested stack (keras==2.2.4, tensorflow==1.14.0, numpy==1.16.5, h5py==2.9.0) since pip installs latest dependencies that differ from where the program was tested.

## Related tools

- **Cyclum** (Provides the cyclum.models.ae module used to build the tanh encoder and cos/sin decoder, plus CyclumAutoTune to select the proper number of linear components for the most circular manifold; handles saving the trained model to .h5.) — https://github.com/KChen-lab/cyclum
- **Keras** (Framework in which the sinusoidal autoencoder is rewritten, compiled (MSE loss + L2 regularization), trained, and saved/loaded as an .h5 artifact.) — https://keras.io/
- **TensorFlow** (Backend for Keras model definition, training, and persistence; must be a TF 1.x release (1.14.0 tested, 1.15.2 latest tested) for Cyclum compatibility.)

## Examples

```
cyclum_ae.save('cyclum_ae_model.h5'); restored = keras.models.load_model('cyclum_ae_model.h5')
```

## Evaluation signals

- cyclum_ae_model.h5 exists and is loadable by keras.models.load_model without error, recovering the tanh encoder and cos/sin decoder layers.
- The compiled objective is mean squared error (least-square reconstruction) plus L2 weight regularization, and training was run until the reconstruction loss converged.
- Decoder structure check: first decoder layer emits cosine and sine activations of the encoder output; the final layer is linear mapping back to gene expression space.
- Every pseudo-time value in pseudotime_2pi.csv falls within [0, 2π] after the modulus wrap.
- Dependency versions match the tested table (keras==2.2.4, tensorflow==1.14.0, numpy==1.16.5, h5py==2.9.0) or, at minimum, a TF 1.x release.

## Limitations

- Cyclum requires TensorFlow 1.x; pip installs the latest dependencies by default, which differ from the tested versions, so versions must be forced via install_requires in setup.py (e.g., keras==2.2.4, tensorflow==1.14.0).
- TensorFlow < 1.15.2 has a known security issue and is not recommended; the paper itself states no version numbers for Python, Keras, or TensorFlow.
- Hardware/runtime guidance is informal: a 12,280-cell × 33,694-gene dataset takes ~40 min on a laptop with a GTX 960M GPU and 2 GB graphic memory (details relegated to Supplementary Note 2).
- The autoencoder realization is currently the only one; a Gaussian Process Latent Variable Model realization is announced but not yet added.

## Evidence

- [readme] The autoencoder is now rewritten using [keras](https://keras.io/), in a more readable way.: [evidence span withheld — anchor status: external_doc]
- [readme] Please make sure that you have TensorFlow 1.x. Cyclum is compatible with newer versions shown in the "Latest tested" column.: [evidence span withheld — anchor status: external_doc]
- [methods] scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well.: "scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well."
- [discussion] The Cyclum package is efficiently implemented in Python using Keras with TensorFlow and has been comprehensively tested.: "The Cyclum package is efficiently implemented in Python using Keras with TensorFlow and has been comprehensively tested."
