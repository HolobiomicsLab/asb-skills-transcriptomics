---
name: circular-pseudotime-wrapping
description: Use when a Cyclum sinusoidal autoencoder has been trained on log2(x+1)-transformed,
  variance-1-scaled scRNA-seq data and the circular latent stage of each cell must
  be converted into a wrapped [0, 2π] pseudo-time.
when_to_use_negative:
- The process of interest is non-periodic (a linear or branching trajectory) — use
  linear pseudo-time methods such as Monocle or Wanderlust, which Cyclum's paper notes
  are 'not optimal for representing such nonlinear periodicity'
- The goal is only to remove cell-cycle covariates from an expression matrix without
  recovering per-cell stages — Cyclum's deconvolution path may be the wrong interface
- Only marker-gene-based phase scoring (e.g., Cyclone-style PCA/gene-pair scoring)
  is needed; Cyclum's methodology 'is to rely on the circular manifold, instead of
  the marker genes'
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Cyclum
  role: Trains the sinusoidal autoencoder, provides cyclum.tuning.CyclumAutoTune for
    component selection and cyclum.postproc.refine_labels for mapping wrapped pseudo-time
    to cell-cycle phases
  repo: https://github.com/KChen-lab/Cyclum
- name: Keras
  role: Implements the autoencoder (tanh encoder, cos/sin decoder layer, MSE + L2-regularized
    compilation) and stores the model as a loadable .h5 artifact
- name: TensorFlow
  role: Backend executing the Keras autoencoder during training and latent-coordinate
    extraction; requires TensorFlow 1.x per the README
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-020-15295-9
    title: Latent periodic process inference from single-cell RNA-seq data
  source_content_hash: sha256:98d6066a52666f34e93ab300373009392575082df62a40359039f63b3fb183fc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/KChen-lab/Cyclum
  tools:
  - Cyclum
  - Keras
  - TensorFlow
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# circular-pseudotime-wrapping

## Summary

Extracts the circular latent coordinate inferred by the Cyclum sinusoidal autoencoder and wraps the per-cell pseudo-time into the canonical [0, 2π] interval via the modulus operation, so that stages of a periodic process (e.g., cell cycle) are represented on a single continuous circle.

## When to use

Use when a Cyclum sinusoidal autoencoder has been trained on log2(x+1)-transformed, variance-1-scaled scRNA-seq data and the circular latent stage of each cell must be converted into a wrapped [0, 2π] pseudo-time.

## When NOT to use

- The process of interest is non-periodic (a linear or branching trajectory) — use linear pseudo-time methods such as Monocle or Wanderlust, which Cyclum's paper notes are 'not optimal for representing such nonlinear periodicity'
- The goal is only to remove cell-cycle covariates from an expression matrix without recovering per-cell stages — Cyclum's deconvolution path may be the wrong interface
- Only marker-gene-based phase scoring (e.g., Cyclone-style PCA/gene-pair scoring) is needed; Cyclum's methodology 'is to rely on the circular manifold, instead of the marker genes'

## Inputs

- Trained Cyclum sinusoidal autoencoder model (Keras .h5 file)
- Per-cell circular latent coordinates extracted from the autoencoder bottleneck
- Preprocessed scRNA-seq expression matrix (log2(x+1), variance=1, mean=1), e.g., the mESC dataset

## Outputs

- Wrapped per-cell pseudo-time in [0, 2π] (e.g., pseudotime_2pi.csv)
- Persisted Keras model file (cyclum_ae_model.h5)
- Optionally refined cell-cycle phase labels (e.g., G1/S/G2M) aligned to the wrapped pseudo-time

## How to apply

First ensure the input expression matrix has been preprocessed as Cyclum recommends: log2(x+1) transform, scaled to variance = 1 and centered to mean = 1. Train the autoencoder (tanh MLP encoder; decoder whose first layer computes cosine and sine activations and whose subsequent linear layer maps back to gene space; mean-square-error objective with L2 weight regularization in Keras/TensorFlow) until reconstruction loss converges — optionally first selecting the number of linear components with cyclum.tuning.CyclumAutoTune to find the 'most circular' manifold. Read the circular latent coordinates from the trained model and compute the wrapped stage as mod(latent_stage, 2π) for every cell, since one pseudo-time per cell is the inferred stage of that cell. Save the Keras model (e.g., cyclum_ae_model.h5) and the per-cell wrapped pseudo-time (e.g., pseudotime_2pi.csv). Downstream, the wrapped pseudo-time can refine external labels such as scanpy.tl.score_genes_cell_cycle or Seurat::CellCycleScoring via cyclum.postproc.refine_labels.

## Related tools

- **Cyclum** (Trains the sinusoidal autoencoder, provides cyclum.tuning.CyclumAutoTune for component selection and cyclum.postproc.refine_labels for mapping wrapped pseudo-time to cell-cycle phases) — https://github.com/KChen-lab/Cyclum
- **Keras** (Implements the autoencoder (tanh encoder, cos/sin decoder layer, MSE + L2-regularized compilation) and stores the model as a loadable .h5 artifact)
- **TensorFlow** (Backend executing the Keras autoencoder during training and latent-coordinate extraction; requires TensorFlow 1.x per the README)

## Examples

```
import numpy as np; pseudotime_2pi = np.mod(model.predict(X).argmax(axis=None) if False else latent_stage, 2 * np.pi); pd.DataFrame(pseudotime_2pi).to_csv('pseudotime_2pi.csv')
```

## Evaluation signals

- Every value in the wrapped pseudo-time output (e.g., pseudotime_2pi.csv) falls within [0, 2π]
- The saved cyclum_ae_model.h5 exists and is loadable by Keras
- Reconstruction loss of the autoencoder converged during training
- Refined phase labels are locally consistent along the wrapped pseudo-time (e.g., an 'S' surrounded by 'G1' cells is re-labeled 'G1' by refine_labels)
- Wrapped pseudo-time stage order agrees with independent phase labels or with the reCAT baseline comparison reported in the paper

## Limitations

- Cyclum was tested on specific dependency versions (python 3.7.4, keras 2.2.4, tensorflow 1.14.0, numpy 1.16.5, pandas 0.25.2, scikit-learn 0.21.3, h5py 2.9.0); TensorFlow 1.x is required, and TensorFlow < 1.15.2 has a known security issue
- pip will install the latest dependencies, which differ from tested versions; exact versions must be forced via setup.py install_requires
- The design fits data to predefined circular manifolds — robust to random noise and small sample sizes, but appropriate only for periodic processes
- One pseudo-time value per cell is meaningful only as a stage within the periodic process, not as an absolute clock

## Evidence

- [intro] Cyclum identifies circular trajectories and unwraps pseudo-time: "It effectively unfolds the circular manifold onto a linear space to obtain precise pseudo-time."
- [methods] Recommended preprocessing for the expression matrix: "scaled (variance = 1), centered (mean = 1) log-transformed (:math:`\log_2(x+1)`) data work well."
- [readme] Label refinement along pseudo-time behavior: [evidence span withheld — anchor status: external_doc]
- [readme] Auto-tuning the number of linear components before fitting the circle: [evidence span withheld — anchor status: external_doc]
- [discussion] Model-based robustness rationale for the circular manifold: "Cyclum is a model-based approach that fits the data to predefined circular manifolds. This design makes Cyclum more robust to handle random noise and small sample sizes."
