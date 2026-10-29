---
name: retinal-progenitor-branch-reconstruction
description: Use when analyzing a multi-branch developmental scRNA-seq dataset with
  a clear cell cycle phase (e.g., E15.5 retina) where scVelo's dynamical mode produces
  a disconnected terminal path and a gene-specific latent time is preferred.
when_to_use_negative:
- Datasets best served by a single unified latent time across the transcriptome (FIT_OPTION
  '1' Unified-time mode) rather than gene-specific times
- Datasets lacking a cell cycle phase or multi-branch structure where the Independent
  mode's added flexibility is unnecessary
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: UniTVelo
  role: Estimates RNA velocity with the Independent (gene-specific time) mode (FIT_OPTION
    '2') and latent time on the retinal dataset
  repo: github.com/StatBiomed/UniTVelo
- name: scVelo
  role: Provides velocity embedding stream and scatter plotting functions (scv.pl.velocity_embedding_stream,
    scv.pl.scatter) and a dynamical-mode baseline comparison
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-34188-7
    title: 'UniTVelo: temporally unified RNA velocity reinforces single-cell trajectory
      inference'
  source_content_hash: sha256:54a68ca8d50e4e5e7e4aa77bd702d256b465329b9d4ed356c23ce4927f723f07
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python 3
  - UniTVelo
  - scVelo
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# retinal-progenitor-branch-reconstruction

## Summary

Reconstruct the E15.5 mouse retinal development trajectory (cycling progenitors feeding PR, RGC, and AC/HC branches) using UniTVelo's Independent (gene-specific time) mode, and verify against Supplementary Fig. S6 that the RGC terminal population remains connected, unlike scVelo's dynamical mode.

## When to use

Use when analyzing a multi-branch developmental scRNA-seq dataset with a clear cell cycle phase (e.g., E15.5 retina) where scVelo's dynamical mode produces a disconnected terminal path and a gene-specific latent time is preferred.

## When NOT to use

- Datasets best served by a single unified latent time across the transcriptome (FIT_OPTION '1' Unified-time mode) rather than gene-specific times
- Datasets lacking a cell cycle phase or multi-branch structure where the Independent mode's added flexibility is unnecessary

## Inputs

- Spliced/unspliced count matrices for the E15.5 mouse retina development dataset as an AnnData object (.h5ad) compatible with scVelo/UniTVelo
- Cell-type label column in adata.obs specifying retinal populations (cycling progenitors, PR, RGC, AC/HC)

## Outputs

- AnnData annotated with velocity and gene-specific latent_time from UniTVelo Independent mode
- Velocity embedding stream plot colored by cell type (dpi=100)
- Latent time scatter plot (color_map='gnuplot', size=20)

## How to apply

Load the spliced/unspliced count matrices as an AnnData object and create a UniTVelo configuration via utv.config.Configuration with FIT_OPTION set to '2' to select the Independent mode suited to complex, multi-branch datasets, R2_ADJUST=True, and IROOT=None. Run utv.run_model passing the cell-type label column from adata.obs and the configuration object. Render velocity embedding streams with scv.pl.velocity_embedding_stream colored by cell type at dpi=100 and latent time with scv.pl.scatter (color_map='gnuplot', size=20). Judge success qualitatively: cycling progenitor transitions should feed all three branches (PR, RGC, AC/HC) and the RGC terminal population should stay connected, matching Supplementary Fig. S6 rather than scVelo's disconnected RGC path.

## Related tools

- **UniTVelo** (Estimates RNA velocity with the Independent (gene-specific time) mode (FIT_OPTION '2') and latent time on the retinal dataset) — github.com/StatBiomed/UniTVelo
- **scVelo** (Provides velocity embedding stream and scatter plotting functions (scv.pl.velocity_embedding_stream, scv.pl.scatter) and a dynamical-mode baseline comparison)

## Examples

```
import unitvelo as utv; velo = utv.config.Configuration(); velo.R2_ADJUST = True; velo.IROOT = None; velo.FIT_OPTION = '2'; adata = utv.run_model('retina_e155.h5ad', label, config_file=velo); scv.pl.velocity_embedding_stream(adata, color=label, dpi=100, title='')
```

## Evaluation signals

- Dentate gyrus / retina stream shows cycling progenitor transitions feeding all three retinal branches (PR, RGC, AC/HC)
- RGC terminal population is connected to the trajectory, matching Supplementary Fig. S6, unlike scVelo's disconnected RGC terminal path
- Stream plot rendered with scv.pl.velocity_embedding_stream colored by the cell-type label at dpi=100 without local disturbances along the trajectory
- Latent time scatter (color='latent_time', color_map='gnuplot', size=20) orders progenitors before terminal cell types
- Configuration used FIT_OPTION='2', R2_ADJUST=True, IROOT=None as specified

## Limitations

- No method for unsupervised selection of dynamically informed genes for projecting cell transitions; stated as an open challenge
- Severely low signal-to-noise ratio in unspliced mRNAs can make velocity estimations inaccurate or inconsistent
- Possible over-counting in unspliced read counting is unresolved and may affect cell transition metrics

## Evidence

- [readme] Independent mode retains gene-specific time: "Retaining gene-spcific time matrics (`Independent mode`) for complex datasets"
- [readme] FIT_OPTION selects mode: "velo.FIT_OPTION (str), '1' Unified-time mode (default), '2' Independent mode"
- [methods] run_model invocation: "# 3. Run model (label refers to column name in adata.obs specifying celltypes)::

    adata = utv.run_model(path_to_adata, label, config_file=velo)"
- [results] finding on retina dataset: [evidence span withheld — anchor status: not_found]
- [intro] retina has clear cell cycle phase: "retina development with a clear cell cycle phase"
