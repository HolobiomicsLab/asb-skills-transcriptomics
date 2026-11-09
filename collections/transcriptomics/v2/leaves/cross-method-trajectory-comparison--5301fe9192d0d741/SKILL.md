---
name: cross-method-trajectory-comparison
description: Use when comparing RNA velocity methods on spliced/unspliced AnnData
  from complex differentiation scenarios (cell cycles, sparse cell types, multi-branch
  lineages) to decide which tool's trajectory matches expected biology.
when_to_use_negative:
- Datasets with simple, well-behaved kinetics where scVelo's default dynamical mode
  already suffices — the independent mode targets complex, multi-branch or cell-cycle
  scenarios
- Analyses requiring a quantitative benchmark with ground-truth cluster edges only
  — this skill's validation is a qualitative check against published trajectory figures;
  use utv.evaluate with cluster_edges instead when ground truth is available
- Datasets lacking spliced/unspliced layer information, since the workflow is built
  on the spliced/unspliced count matrices
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: UniTVelo
  role: Primary velocity estimator run in Independent mode (FIT_OPTION='2') on complex
    multi-branch datasets; invoked via utv.run_model with a Configuration object
  repo: github.com/StatBiomed/UniTVelo
- name: scVelo
  role: Comparison baseline run in dynamical (likelihood-based) mode; its disconnected
    RGC terminal path on E15.5 retina is the reference failure mode
- name: Python 3
  role: Implementation environment for UniTVelo (with TensorFlow 2) and for running
    the analysis notebooks
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-022-34188-7
    title: 'UniTVelo: temporally unified RNA velocity reinforces single-cell trajectory
      inference'
  source_content_hash: sha256:eba4589a5408f2b557dae198228c94558b953afb2e2330c4be16e84a6154cc76
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Python 3
  - UniTVelo
  - scVelo
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# cross-method-trajectory-comparison

## Summary

Reproduces and compares RNA velocity trajectories across methods (e.g., UniTVelo independent mode vs. scVelo dynamical mode) on complex multi-branch or cell-cycle datasets such as mouse dentate gyrus neurogenesis and E15.5 retina development, to judge which method recovers biologically connected trajectories.

## When to use

Use when comparing RNA velocity methods on spliced/unspliced AnnData from complex differentiation scenarios (cell cycles, sparse cell types, multi-branch lineages) to decide which tool's trajectory matches expected biology.

## When NOT to use

- Datasets with simple, well-behaved kinetics where scVelo's default dynamical mode already suffices — the independent mode targets complex, multi-branch or cell-cycle scenarios
- Analyses requiring a quantitative benchmark with ground-truth cluster edges only — this skill's validation is a qualitative check against published trajectory figures; use utv.evaluate with cluster_edges instead when ground truth is available
- Datasets lacking spliced/unspliced layer information, since the workflow is built on the spliced/unspliced count matrices

## Inputs

- AnnData object with spliced/unspliced count matrices for mouse dentate gyrus (P12/P35) neurogenesis
- AnnData object with spliced/unspliced count matrices for E15.5 retina development
- Cell-type label column name in adata.obs (passed as label to utv.run_model)
- UniTVelo configuration object (utv.config.Configuration) with FIT_OPTION, R2_ADJUST, IROOT set

## Outputs

- AnnData with computed velocity and latent_time fields
- Velocity embedding stream plots colored by cell-type label (dpi=100)
- Latent time scatter plots (color_map='gnuplot', size=20)
- Qualitative trajectory comparison vs. Supplementary Fig. S6 (RGC terminal connectivity, OPC-to-OL ordering, three retinal branches PR/RGC/AC-HC)

## How to apply

Load the spliced/unspliced count matrices (e.g., mouse dentate gyrus P12/P35 and E15.5 retina) as AnnData objects. Configure UniTVelo via utv.config.Configuration with FIT_OPTION='2' (Independent, gene-specific-time mode suited to complex multi-branch datasets), R2_ADJUST=True, and IROOT=None. Run utv.run_model passing the cell-type label column from adata.obs and the config object, then render scv.pl.velocity_embedding_stream colored by cell type (dpi=100) and latent time via scv.pl.scatter with color_map='gnuplot'. For the comparison, run scVelo's dynamical mode on the same data and contrast the stream plots: the acceptance check is qualitative agreement with Supplementary Fig. S6 — strengthened OPC-to-OL sub-lineage signal in the dentate gyrus and cycling progenitor transitions feeding all three retinal branches (PR, RGC, AC/HC) with no disconnected RGC terminal path, which is the failure mode seen in scVelo's dynamical mode.

## Related tools

- **UniTVelo** (Primary velocity estimator run in Independent mode (FIT_OPTION='2') on complex multi-branch datasets; invoked via utv.run_model with a Configuration object) — github.com/StatBiomed/UniTVelo
- **scVelo** (Comparison baseline run in dynamical (likelihood-based) mode; its disconnected RGC terminal path on E15.5 retina is the reference failure mode)
- **Python 3** (Implementation environment for UniTVelo (with TensorFlow 2) and for running the analysis notebooks)

## Examples

```
velo = utv.config.Configuration(); velo.R2_ADJUST = True; velo.IROOT = None; velo.FIT_OPTION = '2'; adata = utv.run_model(path_to_adata, label, config_file=velo); scv.pl.velocity_embedding_stream(adata, color=label, dpi=100, title='')
```

## Evaluation signals

- Retina stream plot shows cycling progenitor transitions feeding all three differentiation branches (PR, RGC, AC/HC)
- RGC terminal population remains connected in the UniTVelo stream, unlike the disconnected path produced by scVelo dynamical mode
- Dentate gyrus stream shows strengthened OPC-to-OL sub-lineage ordering consistent with Supplementary Fig. S6
- Latent time scatter (color_map='gnuplot') progresses monotonically along the expected differentiation direction
- Model was run with FIT_OPTION='2', R2_ADJUST=True, IROOT=None as specified for complex multi-branch datasets

## Limitations

- UniTVelo retains gene-specific time (Independent mode) for complex datasets, but no method exists for unsupervised selection of dynamically informed genes for projecting cell transitions — stated as an open challenge
- Possible over-counting in unspliced read counting is unresolved and may affect cell transition metrics
- Sub-sampling strategy fits the model on a subset of cells and predicts the rest; its effect on accuracy across diverse datasets has not been evaluated
- Validation here is qualitative (match to Supplementary Fig. S6), not a quantitative metric

## Evidence

- [results] Both scVelo and UniTVelo capture the expected trajectories: [evidence span withheld — anchor status: not_found]
- [results] scVelo disconnected RGC path: [evidence span withheld — anchor status: not_found]
- [readme] FIT_OPTION modes: "velo.FIT_OPTION (str), '1' Unified-time mode (default), '2' Independent mode"
- [readme] independent mode rationale: "Retaining gene-spcific time matrics (`Independent mode`) for complex datasets"
- [methods] run_model invocation: "adata = utv.run_model(path_to_adata, label, config_file=velo)"
- [methods] latent time visualization: "scv.pl.scatter(adata, color='latent_time', color_map='gnuplot', size=20)"
