---
name: latent-time-inference-verification
description: Use when scRNA-seq data with spliced and unspliced mRNA counts (AnnData/.h5ad)
  must be assigned a directed trajectory and latent time without metabolic-labeling
  priors, and the velocity fit needs per-gene R2 verification.
when_to_use_negative:
- Input lacks an unspliced mRNA layer (spliced-only counts), since the model requires
  un/spliced phase portraits
- Metabolic-labeling kinetic information is the analysis goal (this skill explicitly
  targets inference without labeling knowledge)
- Trajectory direction is already established and only clustering or pseudotime ranking
  of an existing embedding is needed
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: UniTVelo
  role: 'Core model: fits spliced/unspliced phase portraits via TensorFlow 2, estimates
    velocity and unified latent time, and exposes per-gene R2 goodness-of-fit'
  repo: github.com/StatBiomed/UniTVelo
- name: scVelo
  role: Used for velocity embedding stream visualization (scv.pl.velocity_embedding_stream)
    and latent time scatter plots
- name: TensorFlow
  role: Backend providing automatic differentiation for UniTVelo model fitting on
    GPU or CPU
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-34188-7
    title: 'UniTVelo: temporally unified RNA velocity reinforces single-cell trajectory
      inference'
  source_content_hash: sha256:326ebbb878089e5bdf4bfc0d91189235dcf650704461768ce383d6dbca05be98
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Jupyter
  - Python 3
  - TensorFlow
  - UniTVelo
  - scVelo
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# latent-time-inference-verification

## Summary

Verifies RNA-velocity latent time and velocity field inference by configuring UniTVelo's unified-time mode on spliced/unspliced counts, then assessing goodness-of-fit via per-gene coefficient of determination R2 and qualitative checks of the velocity embedding stream.

## When to use

Use when scRNA-seq data with spliced and unspliced mRNA counts (AnnData/.h5ad) must be assigned a directed trajectory and latent time without metabolic-labeling priors, and the velocity fit needs per-gene R2 verification.

## When NOT to use

- Input lacks an unspliced mRNA layer (spliced-only counts), since the model requires un/spliced phase portraits
- Metabolic-labeling kinetic information is the analysis goal (this skill explicitly targets inference without labeling knowledge)
- Trajectory direction is already established and only clustering or pseudotime ranking of an existing embedding is needed

## Inputs

- AnnData (.h5ad) single-cell object with spliced and unspliced mRNA count layers (e.g., scEU-seq intestinal organoid data)
- Cell-type annotation column name in adata.obs
- Optional cluster_edges ground truth for Cross Boundary Direction Correctness evaluation

## Outputs

- Per-cell latent time values stored in adata.obs['latent_time']
- Velocity field and velocity embedding stream plot per cell type
- Per-gene coefficient of determination (R2) histogram as goodness-of-fit summary
- Gene-specific phase portraits with fitted dynamics (utv.pl.plot_range)
- Optional evaluation metrics dict from utv.evaluate (Cross Boundary Direction Correctness)

## How to apply

Load the spliced/unspliced AnnData object and sub-class UniTVelo's base configuration: set velo.R2_ADJUST=True so R-squared regression uses full data rather than extreme quantiles, velo.IROOT=None, velo.FIT_OPTION='1' for Unified-time mode (a unified latent time across the whole transcriptome that resolves directionality discrepancy between genes), and velo.GPU=0 (or -1 for CPU). Run utv.run_model on the AnnData, passing the cell-type column from adata.obs as label; the model fits phase portraits with radial basis function transcription rates and concurrently updates cell time. Inspect scv.pl.velocity_embedding_stream output for the expected directed structure (e.g., two differentiation branches), and plot 'latent_time' with the gnuplot color map to check monotonic ordering from progenitors. Finally compute per-gene R2 as a goodness-of-fit indicator and render phase portraits via utv.pl.plot_range with time_metric='latent_time'; genes with strong spliced dynamics but limited unspliced/spliced imbalance should fit well.

## Related tools

- **UniTVelo** (Core model: fits spliced/unspliced phase portraits via TensorFlow 2, estimates velocity and unified latent time, and exposes per-gene R2 goodness-of-fit) — github.com/StatBiomed/UniTVelo
- **scVelo** (Used for velocity embedding stream visualization (scv.pl.velocity_embedding_stream) and latent time scatter plots)
- **TensorFlow** (Backend providing automatic differentiation for UniTVelo model fitting on GPU or CPU)

## Examples

```
import unitvelo as utv; velo = utv.config.Configuration(); velo.R2_ADJUST=True; velo.IROOT=None; velo.FIT_OPTION='1'; velo.GPU=0; adata = utv.run_model(path_to_adata, label, config_file=velo); scv.pl.velocity_embedding_stream(adata, color=label, dpi=100)
```

## Evaluation signals

- Velocity embedding stream reproduces the expected two-branch organoid structure (secretory and enterocyte branches) with correct local paneth-to-goblet direction
- Per-gene R2 histogram matches the reported Fig. 4c-e goodness-of-fit pattern for the dataset
- Latent time scatter (gnuplot color map) shows a monotonic gradient consistent with progenitor-to-differentiated ordering
- Phase portraits (utv.pl.plot_range) show fitted transcriptional dynamics tracking spliced/unspliced points for dynamically informed genes
- If ground truth is available, utv.evaluate Cross Boundary Direction Correctness metrics are positive

## Limitations

- No unsupervised method exists to select dynamically informed genes for projecting cell transitions; stated as an open challenge
- Possible over-counting in unspliced reads is unresolved and may make a few genes dramatically affect cell transition metrics
- Sub-sampling strategy (fit on a subset of cells, predict the rest) has not been evaluated for accuracy impact across diverse datasets
- Latent time uncertainty is not propagated in a Bayesian manner
- Datasets larger than ~50,000 cells require a GPU with at least 12GB memory

## Evidence

- [abstract] Unified-time mode with R2 goodness-of-fit: "Uniquely, it also supports the inference of a unified latent time across the transcriptome."
- [discussion] R2 as goodness-of-fit indicator: "we illustrated that the coefficient of determination R2 could be an informative indicator of goodness-of-fit"
- [methods] Configuration and run_model workflow: "adata = utv.run_model(path_to_adata, label, config_file=velo)"
- [methods] Latent time visualization: "scv.pl.scatter(adata, color='latent_time', color_map='gnuplot', size=20)"
- [readme] README: configuration parameters: "velo.FIT_OPTION (str), '1' Unified-time mode (default), '2' Independent mode"
- [readme] README: GPU memory requirement: "larger than 50,000 cells for a GPU with 12GB memory"
