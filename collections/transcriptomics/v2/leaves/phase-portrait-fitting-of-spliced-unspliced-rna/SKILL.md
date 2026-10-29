---
name: phase-portrait-fitting-of-spliced-unspliced-rna
description: Use when analyzing scRNA-seq data with both spliced and unspliced count
  layers (AnnData) to infer RNA velocity and cell-fate trajectories on multi-branching
  differentiation without labeling metadata.
when_to_use_negative:
- Data lacks an unspliced mRNA layer (e.g., standard 10x output without intronic counting)
  — phase-portrait fitting requires un/spliced pairs
- Metabolic-labeling information (e.g., scEU-seq pulse-chase labels) is available
  and should be exploited directly rather than inferred
- Goal is evaluation against ground-truth cluster edges without defined cluster_edges/label
  annotations for utv.evaluate
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: UniTVelo
  role: 'Core model: fits spliced/unspliced phase portraits with RBF transcription
    rates and unified latent time via utv.run_model'
  repo: https://github.com/StatBiomed/UniTVelo
- name: scVelo
  role: Visualization utilities (velocity_embedding_stream, scatter of latent_time)
    and benchmark comparison baseline
- name: Python 3
  role: Implementation language for UniTVelo, on TensorFlow 2 with automatic differentiation
- name: TensorFlow
  role: Backend providing automatic differentiation for model fitting; GPU acceleration
    configurable via velo.GPU
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-34188-7
    title: 'UniTVelo: temporally unified RNA velocity reinforces single-cell trajectory
      inference'
  source_content_hash: sha256:f16ffeeb59118c9027a16fc01cce8b6d3bce2feff6466da28868a94e21f0ae39
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/StatBiomed/UniTVelo
  tools:
  - Jupyter
  - Python 3
  - TensorFlow
  - UniTVelo
  - scVelo
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# phase-portrait-fitting-of-spliced-unspliced-rna

## Summary

Fit UniTVelo's joint dynamic model to spliced and unspliced mRNA phase portraits, estimating per-gene velocities and a unified latent time without metabolic-labeling knowledge, using an RBF-based transcription rate over Python 3/TensorFlow 2.

## When to use

Use when analyzing scRNA-seq data with both spliced and unspliced count layers (AnnData) to infer RNA velocity and cell-fate trajectories on multi-branching differentiation without labeling metadata.

## When NOT to use

- Data lacks an unspliced mRNA layer (e.g., standard 10x output without intronic counting) — phase-portrait fitting requires un/spliced pairs
- Metabolic-labeling information (e.g., scEU-seq pulse-chase labels) is available and should be exploited directly rather than inferred
- Goal is evaluation against ground-truth cluster edges without defined cluster_edges/label annotations for utv.evaluate

## Inputs

- AnnData (.h5ad) single-cell object with spliced and unspliced count layers (e.g., scEU-seq intestinal organoid data)
- Name of the cell-type annotation column in adata.obs (label)
- UniTVelo configuration object (utv.config.Configuration) with R2_ADJUST, IROOT, FIT_OPTION, GPU overrides

## Outputs

- AnnData annotated with per-cell/per-gene RNA velocity and velocity embedding
- Latent time per cell (adata.obs['latent_time'])
- Velocity embedding stream plot colored by cell type
- Per-gene phase portrait fits via utv.pl.plot_range
- Per-gene R2 (coefficient of determination) goodness-of-fit values and histogram

## How to apply

Load the spliced/unspliced AnnData object and import unitvelo as utv. Sub-class the base configuration via utv.config.Configuration(): set velo.R2_ADJUST=True (linear regression R-squared on the full data rather than the extreme quantile), velo.IROOT=None (no diffusion-map time initialization from a specified root cluster), velo.FIT_OPTION='1' for Unified-time mode, and velo.GPU=0 (or -1 for CPU). Run adata = utv.run_model(path_to_adata, label, config_file=velo), where label names the cell-type column in adata.obs; the model updates gene velocity and cell time concurrently by fitting phase portraits. Visualize with scv.pl.velocity_embedding_stream(adata, color=label, dpi=100), inspect latent time via scv.pl.scatter(adata, color='latent_time', color_map='gnuplot', size=20), and render per-gene phase portraits with utv.pl.plot_range(gene_name, adata, velo, show_ax=True, time_metric='latent_time'). Compute per-gene R2 (coefficient of determination) as a goodness-of-fit indicator; R2-adjusted fitting on the full data rather than extreme quantiles is a key decision point for datasets with weak kinetics.

## Related tools

- **UniTVelo** (Core model: fits spliced/unspliced phase portraits with RBF transcription rates and unified latent time via utv.run_model) — https://github.com/StatBiomed/UniTVelo
- **scVelo** (Visualization utilities (velocity_embedding_stream, scatter of latent_time) and benchmark comparison baseline)
- **Python 3** (Implementation language for UniTVelo, on TensorFlow 2 with automatic differentiation)
- **TensorFlow** (Backend providing automatic differentiation for model fitting; GPU acceleration configurable via velo.GPU)

## Examples

```
velo = utv.config.Configuration(); velo.R2_ADJUST=True; velo.IROOT=None; velo.FIT_OPTION='1'; velo.GPU=0; adata = utv.run_model(path_to_adata, label, config_file=velo)
```

## Evaluation signals

- Velocity embedding stream reproduces expected biological structure: two clear differentiation branches (secretory, enterocyte) and local paneth-to-goblet direction on the intestinal organoid dataset
- Per-gene R2 coefficient-of-determination histogram matches the reported Fig. 4c-e goodness-of-fit pattern
- Latent time scatter (color_map='gnuplot') shows a monotonic progression consistent with the differentiation trajectory
- Latent time is unified across the transcriptome when FIT_OPTION='1' (no per-gene directionality discrepancy)
- TensorFlow device check confirms GPU is used when velo.GPU is set (tf.config.list_physical_devices('GPU'))

## Limitations

- Fitting without metabolic-labeling knowledge: velocity quality relies on model assumptions; existing methods can return erroneous results due to model violation or lack of temporal regularization
- Low signal-to-noise ratio in unspliced mRNAs can make velocity estimates inaccurate or inconsistent
- Datasets larger than ~50,000 cells may need a GPU with at least 12GB memory (e.g., NVIDIA GeForce RTX 2080 Ti); otherwise fitting is slow (e.g., ~8x more time on the human erythroid dataset)
- No unsupervised method yet for selecting dynamically informed genes for projecting cell transitions; over-counting in unspliced reads may affect transition metrics

## Evidence

- [readme] Sub-class and override base configuration file: "velo.R2_ADJUST = True 
velo.IROOT = None
velo.FIT_OPTION = '1'
velo.GPU = 0"
- [readme] Unified-time vs independent mode: "velo.FIT_OPTION` (str), '1' Unified-time mode (default), '2' Independent mode"
- [methods] Run model call: "adata = utv.run_model(path_to_adata, label, config_file=velo)"
- [methods] Phase portrait visualization: "utv.pl.plot_range(gene_name, adata, velo, show_ax=True, time_metric='latent_time')"
- [discussion] R2 as goodness-of-fit: "we illustrated that the coefficient of determination R2 could be an informative indicator of goodness-of-fit"
- [methods] Implementation stack: "Temporally unified RNA velocity for single cell trajectory inference (UniTVelo) is implementated on Python 3 and TensorFlow 2"
