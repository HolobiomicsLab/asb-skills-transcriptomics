---
name: latent-time-estimation
description: Use when spliced/unspliced AnnData (scVelo-preprocessed) yields reversed
  or distorted trajectory directions from scVelo's stochastic/dynamical modes, and
  a unified latent time is needed to correct directionality.
when_to_use_negative:
- The dataset lacks ground-truth cluster edges / cell-type labels, so direction correctness
  cannot be assessed against expected maturation ordering
- The analysis needs gene-specific time on complex branching data better served by
  the independent FIT_OPTION='2' mode rather than a single unified latent time
- Only chromatin-accessibility-integrated velocity (e.g., MultiVelo-style) is required;
  UniTVelo models spliced/unspliced RNA only
edam_operation: http://edamontology.org/operation_2423
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: UniTVelo
  role: Fits spliced/unspliced dynamics with an RBF-based model and estimates the
    unified latent time and per-gene velocities (utv.run_model); also provides utv.evaluate
    for CBDir and utv.pl.plot_range for phase portraits
  repo: https://github.com/StatBiomed/UniTVelo
- name: scVelo
  role: Preprocesses the AnnData (spliced/unspliced matrices, velocity genes) and
    provides the stochastic/dynamical baseline modes and velocity_embedding_stream
    visualization
- name: TensorFlow 2
  role: Backend for UniTVelo's automatic-differentiation model fitting on GPU
provenance:
  source_task_ids:
  - task_003
  - task_007
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-34188-7
    title: 'UniTVelo: temporally unified RNA velocity reinforces single-cell trajectory
      inference'
  source_content_hash: sha256:2a9e3e217b811406319baf3f00332f88c806011da6fb89691d40d20272c6a81f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/StatBiomed/UniTVelo
  tools:
  - Jupyter
  - Python 3
  - TensorFlow 2
  - UniTVelo
  - scVelo
  edam_operation: http://edamontology.org/operation_2423
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# latent-time-estimation

## Summary

Estimate a temporally unified latent time across the transcriptome with UniTVelo to orient single-cell RNA velocity trajectories correctly (e.g., erythroid maturation on MURK-gene datasets, OPC→COP→NFOL oligodendrocyte paths) and evaluate direction correctness against ground-truth cluster edges.

## When to use

Use when spliced/unspliced AnnData (scVelo-preprocessed) yields reversed or distorted trajectory directions from scVelo's stochastic/dynamical modes, and a unified latent time is needed to correct directionality.

## When NOT to use

- The dataset lacks ground-truth cluster edges / cell-type labels, so direction correctness cannot be assessed against expected maturation ordering
- The analysis needs gene-specific time on complex branching data better served by the independent FIT_OPTION='2' mode rather than a single unified latent time
- Only chromatin-accessibility-integrated velocity (e.g., MultiVelo-style) is required; UniTVelo models spliced/unspliced RNA only

## Inputs

- AnnData (.h5ad) with spliced/unspliced count matrices and cell-type annotations in adata.obs (e.g., mouse erythroid 9k cells, human erythroid MEMP-to-Late-Erythroid 30k cells, P20 mouse hindbrain pons, human bone marrow)
- Ground-truth cluster_edges defining expected transition boundaries between cell-type labels
- Label column name in adata.obs specifying cell types

## Outputs

- AnnData annotated with per-cell UniTVelo velocities and a unified latent_time (saved e.g. as erythroid_velocity_comparison.h5ad)
- Velocity embedding stream plots (scv.pl.velocity_embedding_stream) and latent time scatter plots
- Cross Boundary Direction Correctness (CBDir) metrics from utv.evaluate for UniTVelo vs scVelo
- Spliced-unspliced phase portraits for selected genes (e.g., Abcg2, Smim1, Cnn3, Cyr61)

## How to apply

Load the scVelo-preprocessed AnnData (spliced/unspliced matrices, velocity genes) and subclass utv.config.Configuration setting R2_ADJUST=True, IROOT=None, FIT_OPTION='1' (Unified-time mode), and GPU=0; FIT_OPTION='2' is the gene-independent mode for complex datasets. Fit with adata = utv.run_model(path_to_adata, label, config_file=velo), where label is the adata.obs cell-type column, and run scVelo's dynamical mode on the same data as a baseline. Render scv.pl.velocity_embedding_stream (dpi=100, colored by cell-type label) to check that the expected maturation ordering (e.g., OPC→COP→NFOL→MFOL) is recovered. Visualize the inferred latent time with scv.pl.scatter(color='latent_time', color_map='gnuplot', size=20), inspect MURK-gene phase portraits via utv.pl.plot_range(gene_name, adata, velo, show_ax=True, time_metric='latent_time'), and quantify direction correctness with utv.evaluate(adata, cluster_edges, label, 'velocity') (Cross Boundary Direction Correctness), comparing against the ground-truth cluster_edges.

## Related tools

- **UniTVelo** (Fits spliced/unspliced dynamics with an RBF-based model and estimates the unified latent time and per-gene velocities (utv.run_model); also provides utv.evaluate for CBDir and utv.pl.plot_range for phase portraits) — https://github.com/StatBiomed/UniTVelo
- **scVelo** (Preprocesses the AnnData (spliced/unspliced matrices, velocity genes) and provides the stochastic/dynamical baseline modes and velocity_embedding_stream visualization)
- **TensorFlow 2** (Backend for UniTVelo's automatic-differentiation model fitting on GPU)

## Examples

```
velo = utv.config.Configuration(); velo.R2_ADJUST=True; velo.IROOT=None; velo.FIT_OPTION='1'; velo.GPU=0; adata = utv.run_model(path_to_adata, label, config_file=velo)
```

## Evaluation signals

- Velocity embedding stream shows the expected maturation ordering where scVelo's dynamical mode returns reversed/distorted directions (e.g., corrected erythroid direction on MURK-gene datasets)
- UniTVelo CBDir exceeds the scVelo baseline, consistent with paper values (e.g., 0.609 vs 0.347 and 0.074 for OPC→COP→NFOL in the P20 hindbrain pons)
- Latent time scatter (color_map='gnuplot') shows a monotonic pseudotime concordant with manually seeded diffusion-map pseudotime from root cells
- Phase portraits of known MURK genes (Abcg2, Smim1, Cnn3, Cyr61) are consistent under the unified latent_time metric
- Resulting annotated .h5ad contains both methods' velocity layers and UniTVelo's latent_time field

## Limitations

- No method exists for unsupervised selection of dynamically informed genes; stated as an open challenge
- Possible over-counting in unspliced read counting may cause a few genes to dramatically affect cell transition metrics
- Sub-sampling strategy fits the model on a subset of cells and predicts the rest; effect on accuracy across diverse datasets is not yet evaluated
- Bayesian treatment of latent time uncertainty is anticipated but not implemented
- Datasets larger than ~50,000 cells may exceed 12GB GPU memory (NVIDIA GeForce RTX 2080 Ti)

## Evidence

- [abstract] unified latent time across transcriptome: "Uniquely, it also supports the inference of a unified latent time across the transcriptome."
- [intro] unified time resolves directionality: "we also introduce a unified latent time across the whole transcriptome, which can resolve the discrepancy of directionality between genes"
- [readme] configuration parameters: "Sub-class and override base configuration file (here lists a few frequently used), please refer `config.py` for detailed arguments."
- [readme] latent time visualization: "scv.pl.scatter(adata, color='latent_time', color_map='gnuplot', size=20)"
- [methods] run model with label column: "adata = utv.run_model(path_to_adata, label, config_file=velo)"
