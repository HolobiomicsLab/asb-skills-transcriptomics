---
name: single-cell-trajectory-visualization
description: Use when an AnnData object with spliced/unspliced counts has been fitted
  with UniTVelo (or scVelo) and you must render the velocity stream, latent time,
  phase portraits, or direction-correctness metrics to judge the trajectory.
when_to_use_negative:
- Input data lacks unspliced/spliced counts or metabolic-labeling-informed layers
  needed for velocity estimation
- The goal is integrating chromatin accessibility into velocity — MultiVelo, not UniTVelo,
  is the aligned framework
- Dataset is larger than ~50,000 cells without a GPU with >=12GB memory (e.g., NVIDIA
  GeForce RTX 2080 Ti)
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
tools:
- name: UniTVelo
  role: Fits the temporally unified RNA velocity model, estimates latent_time, and
    provides plotting/evaluation utilities (utv.pl.plot_range, utv.evaluate)
  repo: https://github.com/StatBiomed/UniTVelo
- name: scVelo
  role: Comparator baselines (stochastic, dynamical modes) and plotting backend for
    velocity_embedding_stream and latent-time scatter
- name: TensorFlow
  role: Automatic-differentiation backend for UniTVelo model fitting, with optional
    GPU acceleration
provenance:
  source_task_ids:
  - task_004
  - task_005
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-34188-7
    title: 'UniTVelo: temporally unified RNA velocity reinforces single-cell trajectory
      inference'
  source_content_hash: sha256:fb6be23c6c45616b8e49e58a7b6b728acbe7d6122b6e3ab92fb6cd7467510b3c
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# single-cell-trajectory-visualization

## Summary

Visualize and sanity-check RNA velocity trajectories inferred by UniTVelo/scVelo on AnnData objects, using velocity embedding streams, latent-time scatters, per-gene phase portraits, and R2 goodness-of-fit histograms to reveal differentiation branches and their directions.

## When to use

Use when an AnnData object with spliced/unspliced counts has been fitted with UniTVelo (or scVelo) and you must render the velocity stream, latent time, phase portraits, or direction-correctness metrics to judge the trajectory.

## When NOT to use

- Input data lacks unspliced/spliced counts or metabolic-labeling-informed layers needed for velocity estimation
- The goal is integrating chromatin accessibility into velocity — MultiVelo, not UniTVelo, is the aligned framework
- Dataset is larger than ~50,000 cells without a GPU with >=12GB memory (e.g., NVIDIA GeForce RTX 2080 Ti)

## Inputs

- AnnData object (.h5ad) with spliced and unspliced mRNA counts and an adata.obs cell-type column (label)
- Fitted UniTVelo model/config object returned by utv.run_model
- Ground-truth cluster_edges list for Cross Boundary Direction Correctness evaluation
- Gene names for phase portraits (e.g., Cd44, Celf2, Taok3)

## Outputs

- Velocity embedding stream plot (per method: UniTVelo vs scVelo stochastic/dynamical)
- Latent time scatter (adata.obs['latent_time'], gnuplot colormap)
- Per-gene phase portraits via utv.pl.plot_range
- CBDir evaluation metrics dict from utv.evaluate and per-gene R2 histogram (e.g., Fig. 4c)

## How to apply

After fitting UniTVelo with utv.run_model(path_to_adata, label, config_file=velo) — configured with R2_ADJUST=True, IROOT=None, FIT_OPTION='1' (Unified-time mode) or '2' (Independent mode), and GPU=0 or -1 for CPU — render the velocity field with scv.pl.velocity_embedding_stream(adata, color=label, dpi=100, title='') and compare against scVelo's stochastic/dynamical streams to check branch directionality (e.g., reversed erythroid branch and distorted monocyte branch in human bone marrow). Plot inferred latent time via scv.pl.scatter(adata, color='latent_time', color_map='gnuplot', size=20) and verify it concords with manually seeded diffusion-map pseudotime. For mechanistic inspection, draw per-gene phase portraits with utv.pl.plot_range(gene_name, adata, velo, show_ax=True, time_metric='latent_time') (e.g., Cd44, Celf2, Taok3). For quantitative validation, compute Cross Boundary Direction Correctness with metrics = utv.evaluate(adata, cluster_edges, label, 'velocity') using ground-truth cluster_edges, and per-gene R2 histograms as goodness-of-fit indicators.

## Related tools

- **UniTVelo** (Fits the temporally unified RNA velocity model, estimates latent_time, and provides plotting/evaluation utilities (utv.pl.plot_range, utv.evaluate)) — https://github.com/StatBiomed/UniTVelo
- **scVelo** (Comparator baselines (stochastic, dynamical modes) and plotting backend for velocity_embedding_stream and latent-time scatter)
- **TensorFlow** (Automatic-differentiation backend for UniTVelo model fitting, with optional GPU acceleration)

## Examples

```
import unitvelo as utv; import scvelo as scv; velo = utv.config.Configuration(); velo.R2_ADJUST = True; velo.IROOT = None; velo.FIT_OPTION = '1'; velo.GPU = 0; adata = utv.run_model(path_to_adata, label, config_file=velo)
```

## Evaluation signals

- Velocity stream reproduces the expected topology: two clear differentiation branches (secretory, enterocyte) with local paneth-to-goblet direction on intestinal organoid; corrected erythroid and monocyte directions on human bone marrow
- UniTVelo latent time is in good concordance with manually seeded diffusion-map pseudotime
- CBDir from utv.evaluate is positive for expected transitions (paper: UniTVelo 0.793 vs scVelo dynamical -0.445 on mouse erythroid)
- Per-gene R2 histogram matches the reported Fig. 4c-e goodness-of-fit pattern
- Phase portraits show coherent spliced/unspliced dynamics against latent_time for inspected genes

## Limitations

- No unsupervised method for selecting dynamically informed genes for projecting cell transitions; stated as an open challenge
- Possible over-counting in unspliced read counting is unresolved and may mean a few genes dramatically affect cell transition metrics
- Effect of the sub-sampling strategy (fit on a subset of cells, predict the rest) on accuracy across diverse datasets is not evaluated
- UniTVelo can be substantially slower than scVelo (e.g., ~8x more time on the human erythroid dataset); >50,000 cells need a GPU with 12GB memory

## Evidence

- [readme] The model estimates velocity of each gene and updates cell time based on phase portraits concurrently.: "The model estimates velocity of each gene and updates cell time based on phase portraits concurrently."
- [methods] latent time scatter snippet: "scv.pl.scatter(adata, color='latent_time', color_map='gnuplot', size=20)"
- [discussion] R2 as goodness-of-fit indicator: "we illustrated that the coefficient of determination R2 could be an informative indicator of goodness-of-fit"
- [intro] Ten-dataset demonstration scope: "The capabilities and generalization abilities of UniTVelo is demonstrated on various developmental trajectories across 10 datasets"
- [discussion] GPU memory limit for large datasets: "larger than 50,000 cells for a GPU with 12GB memory"
