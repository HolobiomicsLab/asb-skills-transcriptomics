---
name: radial-basis-function-mean-function-implementation
description: 'Use when implementing or reproducing UniTVelo''s top-down EM inference
  loop: a preprocessed AnnData with spliced/unspliced layers must be fit with an RBF
  gene-level mean function and velocity computed as ds/dt.'
when_to_use_negative:
- Data lacks unspliced/spliced count layers or has not been preprocessed — scVelo-style
  preprocessing is assumed upstream
- The goal is steady-state or scVelo dynamical-model velocity based on un/spliced
  deviation from equilibrium rather than spliced-oriented RBF dynamics
- Datasets larger than ~50,000 cells without sufficient GPU memory (the paper notes
  >50,000 cells for a GPU with 12GB memory as a scaling limit)
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: UniTVelo
  role: Package implementing the RBF-based top-down EM inference loop, configuration
    switches, and plotting utilities (utv.run_model, utv.config, utv.pl.plot_range,
    utv.evaluate)
  repo: https://github.com/StatBiomed/UniTVelo
- name: TensorFlow
  role: Backend for automatic differentiation and Adam gradient-based optimization
    of kinetic parameters and latent times
- name: scVelo
  role: Provides preprocessed tutorial datasets (e.g., Mouse Erythroid, Human BoneMarrow)
    and velocity embedding stream visualization
- name: Python 3
  role: Implementation language for the model
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-34188-7
    title: 'UniTVelo: temporally unified RNA velocity reinforces single-cell trajectory
      inference'
  source_content_hash: sha256:601e576d151ebd0484a2e9b02278573a2d8aa79d859d21fb30e15d716969ccad
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/StatBiomed/UniTVelo
  tools:
  - Python 3
  - TensorFlow
  - UniTVelo
  - scVelo
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# radial-basis-function-mean-function-implementation

## Summary

Implements UniTVelo's RBF mean function over latent time that models the spliced RNA trajectory and transcription rate, so per-gene RNA velocity is obtained as the time derivative ds/dt of the spliced profile rather than deviation from a steady-state equilibrium.

## When to use

Use when implementing or reproducing UniTVelo's top-down EM inference loop: a preprocessed AnnData with spliced/unspliced layers must be fit with an RBF gene-level mean function and velocity computed as ds/dt.

## When NOT to use

- Data lacks unspliced/spliced count layers or has not been preprocessed — scVelo-style preprocessing is assumed upstream
- The goal is steady-state or scVelo dynamical-model velocity based on un/spliced deviation from equilibrium rather than spliced-oriented RBF dynamics
- Datasets larger than ~50,000 cells without sufficient GPU memory (the paper notes >50,000 cells for a GPU with 12GB memory as a scaling limit)

## Inputs

- Preprocessed AnnData object (.h5ad) with spliced and unspliced count layers
- Cell-type label column in adata.obs (passed as `label` to run_model)
- Configuration object (utv.config.Configuration) with R2_ADJUST, IROOT, FIT_OPTION, GPU settings
- Optional cluster_edges ground truth for utv.evaluate metrics

## Outputs

- Per-gene RNA velocity vectors (adata.layers['velocity']) computed as ds/dt of the RBF spliced profile
- Cell latent time estimates (adata.obs['latent_time']) in unified-time or independent mode
- Phase portraits via utv.pl.plot_range and velocity embedding streams via scv.pl.velocity_embedding_stream
- Evaluation metrics (e.g., Cross Boundary Direction Correctness) from utv.evaluate

## How to apply

Load a preprocessed AnnData object (e.g., Mouse Erythroid or Human BoneMarrow tutorial datasets) with spliced and unspliced count layers via scanpy/scVelo. Define the gene-level mean function as a radial basis function (RBF) over latent time modeling the spliced RNA trajectory and transcription rate, and compute velocity as the time derivative of the spliced counts (ds/dt). Embed this in a maximum-likelihood EM loop that alternately updates per-gene kinetic parameters and cell latent times via TensorFlow 2 automatic differentiation with an Adam optimizer. Select the latent-time mode via FIT_OPTION: '1' shares a single unified latent time across the whole transcriptome, while '2' assigns a gene-specific independent latent time. Wire configuration switches R2_ADJUST (R-squared on extreme quantile vs full data), IROOT (root cell cluster enabling diffusion-map based time initialization, default None), and GPU (-1 for CPU mode). Validate with the coefficient of determination R2 as the goodness-of-fit indicator for dynamically informed genes and by checking that phase portraits and latent time reproduce the expected directional stream.

## Related tools

- **UniTVelo** (Package implementing the RBF-based top-down EM inference loop, configuration switches, and plotting utilities (utv.run_model, utv.config, utv.pl.plot_range, utv.evaluate)) — https://github.com/StatBiomed/UniTVelo
- **TensorFlow** (Backend for automatic differentiation and Adam gradient-based optimization of kinetic parameters and latent times)
- **scVelo** (Provides preprocessed tutorial datasets (e.g., Mouse Erythroid, Human BoneMarrow) and velocity embedding stream visualization)
- **Python 3** (Implementation language for the model)

## Examples

```
velo = utv.config.Configuration(); velo.R2_ADJUST = True; velo.FIT_OPTION = '1'; adata = utv.run_model(path_to_adata, label, config_file=velo)
```

## Evaluation signals

- Coefficient of determination R2 serves as the goodness-of-fit indicator for dynamically informed genes
- Returned trajectory reproduces the expected directional velocity stream on a validation dataset (e.g., hematopoietic differentiation)
- Phase portraits from utv.pl.plot_range show concurrent velocity/time updates consistent with the fitted RBF mean function
- Latent time scatter (scv.pl.scatter color='latent_time') shows a coherent ordering matching biological progression
- TensorFlow reports the expected GPU (or falls back to CPU when velo.GPU = -1) during fitting

## Limitations

- No method for unsupervised selection of dynamically informed genes for projecting cell transitions — stated as an open challenge
- Bayesian treatment of latent time uncertainty is anticipated but not implemented
- Effect of sub-sampling on accuracy across diverse datasets has not been evaluated
- Possible over-counting in unspliced read counting is unresolved and may affect cell transition metrics
- Scaling: around 8x more runtime on the human erythroid dataset; datasets larger than 50,000 cells require a GPU with at least 12GB memory

## Evidence

- [intro] developed UniTVelo, a statistical method that models the full dynamics of gene expression with a radial basis function (RBF) and quantifies RNA velocity in a top-down manner: "developed UniTVelo, a statistical method that models the full dynamics of gene expression with a radial basis function (RBF) and quantifies RNA velocity in a top-down manner"
- [methods] Temporally unified RNA velocity for single cell trajectory inference (UniTVelo) is implementated on Python 3 and TensorFlow 2.: "Temporally unified RNA velocity for single cell trajectory inference (UniTVelo) is implementated on Python 3 and TensorFlow 2."
- [readme] The model estimates velocity of each gene and updates cell time based on phase portraits concurrently.: "The model estimates velocity of each gene and updates cell time based on phase portraits concurrently."
- [readme] velo.FIT_OPTION (str), '1' Unified-time mode (default), '2' Independent mode: "velo.FIT_OPTION (str), '1' Unified-time mode (default), '2' Independent mode"
- [discussion] we illustrated that the coefficient of determination R2 could be an informative indicator of goodness-of-fit: "we illustrated that the coefficient of determination R2 could be an informative indicator of goodness-of-fit"
