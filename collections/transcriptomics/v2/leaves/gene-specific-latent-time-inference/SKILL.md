---
name: gene-specific-latent-time-inference
description: Use when the dataset contains complex branching, cycling progenitors,
  or sparse cell types where a single transcriptome-wide latent time is inappropriate,
  and per-gene time is needed to recover trajectory structure.
when_to_use_negative:
- Datasets where a single transcriptome-wide ordering suffices and stably, monotonically
  changed genes dominate — use the Unified-time mode (FIT_OPTION='1') instead
- Downstream tasks requiring unsupervised selection of dynamically informed genes
  — the paper states this remains an open challenge
- Analyses expecting a Bayesian quantification of latent time uncertainty — this is
  anticipated but not implemented in UniTVelo
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: UniTVelo
  role: Implements the Independent (gene-specific time) mode via utv.run_model with
    FIT_OPTION='2'; Python 3 and TensorFlow 2 based
  repo: https://github.com/StatBiomed/UniTVelo
- name: scVelo
  role: Used for velocity embedding stream visualization and latent time scatter plots
    (scv.pl.velocity_embedding_stream, scv.pl.scatter); its dynamical mode serves
    as a comparison baseline
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-34188-7
    title: 'UniTVelo: temporally unified RNA velocity reinforces single-cell trajectory
      inference'
  source_content_hash: sha256:9870826f6758377557cbd6459e13ea59774871220d75b8af5a10d70c48dd7531
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/StatBiomed/UniTVelo
  tools:
  - Python 3
  - UniTVelo
  - scVelo
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# gene-specific-latent-time-inference

## Summary

Run UniTVelo's Independent mode (FIT_OPTION='2') to assign gene-specific latent time metrics, retaining per-gene time instead of a unified transcriptome-wide time. This is the recommended mode for complex differentiation scenarios such as cell cycles, sparse cell types, and multi-branch trajectories (e.g., retina development with PR, RGC, and AC/HC branches).

## When to use

Use when the dataset contains complex branching, cycling progenitors, or sparse cell types where a single transcriptome-wide latent time is inappropriate, and per-gene time is needed to recover trajectory structure.

## When NOT to use

- Datasets where a single transcriptome-wide ordering suffices and stably, monotonically changed genes dominate — use the Unified-time mode (FIT_OPTION='1') instead
- Downstream tasks requiring unsupervised selection of dynamically informed genes — the paper states this remains an open challenge
- Analyses expecting a Bayesian quantification of latent time uncertainty — this is anticipated but not implemented in UniTVelo

## Inputs

- AnnData object (or path) with spliced/unspliced count matrices
- Cell-type label column name in adata.obs
- UniTVelo Configuration object (utv.config.Configuration) with FIT_OPTION='2', R2_ADJUST=True, IROOT=None

## Outputs

- AnnData annotated with velocity and gene-specific latent time
- Velocity embedding stream plot colored by cell type
- Latent time scatter plot (color_map='gnuplot', size=20)

## How to apply

Load the spliced/unspliced AnnData object and create a UniTVelo configuration via utv.config.Configuration, setting FIT_OPTION='2' to select the Independent (gene-specific time) mode instead of the default Unified-time mode ('1'). Enable R2_ADJUST so the linear regression R-squared is computed on the full data rather than only extreme quantiles, and leave IROOT as None unless a root cell cluster is known. Call utv.run_model(path_to_adata, label, config_file=velo), passing the cell-type label column name from adata.obs; the model fits the dynamics of spliced and unspliced RNAs via flexible transcription activities and concurrently updates each gene's velocity and its gene-specific latent time. Visualize with scv.pl.velocity_embedding_stream (color=label, dpi=100) and scv.pl.scatter(color='latent_time', color_map='gnuplot', size=20). Rationale: the Independent mode suits datasets like the dentate gyrus (P12/P35) and E15.5 retina where a unified time would smooth over cycling progenitor transitions and distinct branches.

## Related tools

- **UniTVelo** (Implements the Independent (gene-specific time) mode via utv.run_model with FIT_OPTION='2'; Python 3 and TensorFlow 2 based) — https://github.com/StatBiomed/UniTVelo
- **scVelo** (Used for velocity embedding stream visualization and latent time scatter plots (scv.pl.velocity_embedding_stream, scv.pl.scatter); its dynamical mode serves as a comparison baseline)

## Examples

```
velo = utv.config.Configuration(); velo.FIT_OPTION = '2'; velo.R2_ADJUST = True; velo.IROOT = None; adata = utv.run_model(path_to_adata, label, config_file=velo); scv.pl.velocity_embedding_stream(adata, color=label, dpi=100)
```

## Evaluation signals

- Dentate gyrus stream plot shows strengthened OPC-to-OL sub-lineage ordering matching Supplementary Fig. S6
- E15.5 retina stream recovers cycling progenitor transitions feeding all three branches (PR, RGC, AC/HC)
- RGC terminal population remains connected in the velocity stream, unlike the disconnected path produced by scVelo dynamical mode
- Latent time scatter (gnuplot color map) is continuous and ordered along the expected developmental direction
- Optional utv.evaluate Cross Boundary Direction Correctness metrics computed with cluster_edges ground truth

## Limitations

- No unsupervised method exists for selecting dynamically informed genes for projecting cell transitions (open challenge)
- Possible over-counting in unspliced read counting may cause a few genes to dramatically affect cell transition metrics
- Sub-sampling strategy (fit on a cell subset, predict the rest) may affect accuracy across diverse datasets and has not been fully evaluated
- Latent time uncertainty is not propagated in a Bayesian manner
- Datasets larger than ~50,000 cells may need GPU memory (12GB reference: NVIDIA GeForce RTX 2080 Ti); UniTVelo falls back to CPU via velo.GPU = -1

## Evidence

- [readme] Retaining gene-spcific time matrics (`Independent mode`) for complex datasets: "Retaining gene-spcific time matrics (`Independent mode`) for complex datasets"
- [readme] FIT_OPTION '2' selects Independent mode per README arguments: "velo.FIT_OPTION (str), '1' Unified-time mode (default), '2' Independent mode"
- [results] Both scVelo and UniTVelo captured cycling progenitor transitions and all three differentiation branches on E15.5 retina: [evidence span withheld — anchor status: not_found]
- [methods] Velocity embedding stream visualization command from methods: "scv.pl.velocity_embedding_stream(adata, color=label, dpi=100, title='')"
- [abstract] UniTVelo was benchmarked on systems with weak kinetics or complex branches: "With ten datasets, we demonstrate that UniTVelo returns the expected trajectory in different biological systems, including hematopoietic differentiation and those even with weak kinetics or complex"
