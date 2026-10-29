---
name: maximum-likelihood-em-optimization
description: Use when inferring RNA velocity and latent time jointly from spliced/unspliced
  count matrices (AnnData) with a top-down dynamic model, especially when unified
  transcriptome-wide latent time or gene-specific time modes are needed.
when_to_use_negative:
- When downstream analysis requires unsupervised selection of dynamically informed
  genes — the paper states this remains an open challenge in UniTVelo
- When Bayesian propagation of latent time uncertainty is required — the article notes
  Bayesian treatment of latent time uncertainty is anticipated but not implemented
- When spliced/unspliced count data is unavailable (e.g. only spliced reads were sequenced),
  since the model relies on both layers
tools:
- name: UniTVelo
  role: Implements the top-down EM velocity/latent-time inference loop with the RBF
    spliced-RNA mean function and unified-time/independent modes
  repo: https://github.com/StatBiomed/UniTVelo
- name: TensorFlow
  role: Provides automatic differentiation and the Adam gradient-based optimizer used
    to fit gene kinetic parameters and cell latent times
- name: scVelo
  role: Supplies preprocessing and the tutorial datasets (spliced/unspliced layers)
    and visualization utilities such as velocity_embedding_stream
- name: Python 3
  role: Runtime environment in which UniTVelo and TensorFlow 2 are implemented and
    executed
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-022-34188-7
    title: 'UniTVelo: temporally unified RNA velocity reinforces single-cell trajectory
      inference'
  source_content_hash: sha256:595136fc4d25d18d55b758a5a6c3286f8c0fd147eb70c9eb2ba8a09b55050242
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/StatBiomed/UniTVelo
  tools:
  - Python 3
  - TensorFlow
  - UniTVelo
  - scVelo
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# maximum-likelihood-em-optimization

## Summary

An EM-style maximum-likelihood inference loop from UniTVelo that alternately estimates per-gene kinetic parameters and cell latent times via TensorFlow 2 automatic differentiation and a gradient-based Adam optimizer, computing RNA velocity as the time derivative (ds/dt) of a radial basis function (RBF) modeled spliced-RNA profile rather than from deviation to a steady-state equilibrium.

## When to use

Use when inferring RNA velocity and latent time jointly from spliced/unspliced count matrices (AnnData) with a top-down dynamic model, especially when unified transcriptome-wide latent time or gene-specific time modes are needed.

## When NOT to use

- When downstream analysis requires unsupervised selection of dynamically informed genes — the paper states this remains an open challenge in UniTVelo
- When Bayesian propagation of latent time uncertainty is required — the article notes Bayesian treatment of latent time uncertainty is anticipated but not implemented
- When spliced/unspliced count data is unavailable (e.g. only spliced reads were sequenced), since the model relies on both layers

## Inputs

- Preprocessed AnnData object (.h5ad) with spliced and unspliced count layers, e.g. Mouse Erythroid or Human BoneMarrow tutorial datasets
- Configuration object specifying R2_ADJUST, IROOT, FIT_OPTION ('1' unified-time or '2' independent), and GPU index
- Label column name in adata.obs specifying cell types

## Outputs

- Per-gene and per-cell RNA velocity vectors stored in the AnnData object
- Latent time estimates per cell ('latent_time' column)
- Dynamically informed genes with coefficient of determination R2 goodness-of-fit scores
- Phase portrait plots (utv.pl.plot_range) and velocity embedding stream plots

## How to apply

Load a preprocessed AnnData object with spliced and unspliced layers (e.g. Mouse Erythroid or Human BoneMarrow datasets) using scanpy/scVelo. Define a gene-level mean function over latent time with a radial basis function (RBF) modeling the spliced RNA trajectory and transcription rate, then compute velocity as the derivative of spliced counts (ds/dt). Run the EM loop that alternately updates per-gene kinetic parameters and cell latent times using automatic differentiation with Adam in TensorFlow 2, so velocity and cell time are estimated concurrently. Choose the latent-time mode via the FIT_OPTION configuration: '1' shares a single unified latent time across the whole transcriptome (resolving directionality discrepancies between genes), while '2' assigns a gene-specific independent latent time. Configure R2_ADJUST (R-squared on extreme quantile vs full data), IROOT (root cell cluster for diffusion-map-based time initialization, default None), and GPU (card index, -1 for CPU). Inspect phase portraits via utv.pl.plot_range and latent time maps to confirm the concurrent velocity/time updates behave as described.

## Related tools

- **UniTVelo** (Implements the top-down EM velocity/latent-time inference loop with the RBF spliced-RNA mean function and unified-time/independent modes) — https://github.com/StatBiomed/UniTVelo
- **TensorFlow** (Provides automatic differentiation and the Adam gradient-based optimizer used to fit gene kinetic parameters and cell latent times)
- **scVelo** (Supplies preprocessing and the tutorial datasets (spliced/unspliced layers) and visualization utilities such as velocity_embedding_stream)
- **Python 3** (Runtime environment in which UniTVelo and TensorFlow 2 are implemented and executed)

## Examples

```
velo = utv.config.Configuration(); velo.R2_ADJUST = True; velo.FIT_OPTION = '1'; velo.GPU = 0; adata = utv.run_model(path_to_adata, label, config_file=velo)
```

## Evaluation signals

- Coefficient of determination R2 serves as the goodness-of-fit indicator for dynamically informed genes
- The returned latent time and velocity stream should reproduce the expected directional trajectory on a validation dataset (e.g. hematopoietic differentiation, bone marrow branching)
- In unified-time mode ('1'), a single latent time is shared across the whole transcriptome; in independent mode ('2'), each gene receives its own latent time matrix — verify the mode-specific time structure
- Phase portraits of individual genes should show spliced-unspliced dynamics consistent with the fitted RBF mean function and derivative-based velocities

## Limitations

- GPU memory constrains dataset size — larger than 50,000 cells requires a GPU with 12GB memory (e.g. NVIDIA GeForce RTX 2080 Ti)
- Sub-sampling strategy fits the model on a subset of cells and predicts the rest; the effect on accuracy across diverse datasets has not been evaluated
- Possible over-counting in unspliced read counting is unresolved and may affect cell transition metrics
- No method exists for unsupervised selection of dynamically informed genes, stated as an open challenge

## Evidence

- [intro] Top-down RBF design: "developed UniTVelo, a statistical method that models the full dynamics of gene expression with a radial basis function (RBF) and quantifies RNA velocity in a top-down manner"
- [methods] Implementation stack: "Temporally unified RNA velocity for single cell trajectory inference (UniTVelo) is implementated on Python 3 and TensorFlow 2."
- [abstract] Unified latent time: "Uniquely, it also supports the inference of a unified latent time across the transcriptome."
- [readme] Concurrent EM updates: "The model estimates velocity of each gene and updates cell time based on phase portraits concurrently."
- [readme] Automatic differentiation architecture: "UniTVelo is designed based on TensorFlow's automatic differentiation architecture."
- [readme] FIT_OPTION mode switch: "velo.FIT_OPTION (str), '1' Unified-time mode (default), '2' Independent mode"
