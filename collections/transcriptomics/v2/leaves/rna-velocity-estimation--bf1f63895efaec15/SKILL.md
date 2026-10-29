---
name: rna-velocity-estimation
description: Use when you have spliced/unspliced count matrices (AnnData/h5ad) from
  differentiation datasets such as pancreas, dentate gyrus, retina, hindbrain, erythroid
  or bone marrow, and need directed cell transitions, latent time, or CBDir benchmarking
  against scVelo.
when_to_use_negative:
- Input lacks spliced/unspliced layers (e.g., only normalized counts or a feature
  table) — velocity estimation requires unspliced reads
- Dataset exceeds ~50,000 cells on a GPU with 12GB memory and sub-sampling is not
  acceptable, since accuracy impact of sub-sampling is not yet evaluated
- You need unsupervised selection of dynamically informed genes for projecting cell
  transitions — the authors state this remains an open challenge
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: UniTVelo
  role: 'Core method: fits spliced/unspliced dynamics via RBF transcription rates,
    estimates velocities and unified or gene-specific latent time, and provides utv.run_model
    and utv.evaluate'
  repo: github.com/StatBiomed/UniTVelo
- name: scVelo
  role: Preprocessing (gene filtering, moments) and baseline comparator in 'stochastic'
    and 'dynamical' modes; also used for velocity_embedding_stream and latent-time
    scatter plots
- name: CellRank
  role: Cell fate mapping used to validate UniTVelo's recovered Fev+ pre-endocrine
    to Delta cell transitions on pancreas data
- name: TensorFlow
  role: Automatic-differentiation backend on which UniTVelo is implemented (TensorFlow
    2, GPU-accelerated via CUDA)
provenance:
  source_task_ids:
  - task_002
  - task_006
  - task_008
  - task_003
  - task_007
  source_papers:
  - doi: 10.1038/s41467-022-34188-7
    title: 'UniTVelo: temporally unified RNA velocity reinforces single-cell trajectory
      inference'
  source_content_hash: sha256:9423ff66ab63b5686cd32e96d6905ea61f6754c7f079be6696e1ee8fe94dd196
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CellRank
  - Jupyter
  - Python 3
  - TensorFlow
  - UniTVelo
  - scVelo
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# rna-velocity-estimation

## Summary

Estimate RNA velocity and inferred trajectory direction on single-cell spliced/unspliced data with UniTVelo, a TensorFlow 2 framework that models spliced/unspliced dynamics with radial-basis-function transcription rates and either a unified latent time across the transcriptome (FIT_OPTION='1') or gene-specific time (FIT_OPTION='2'), benchmarked against scVelo's stochastic and dynamical modes.

## When to use

Use when you have spliced/unspliced count matrices (AnnData/h5ad) from differentiation datasets such as pancreas, dentate gyrus, retina, hindbrain, erythroid or bone marrow, and need directed cell transitions, latent time, or CBDir benchmarking against scVelo.

## When NOT to use

- Input lacks spliced/unspliced layers (e.g., only normalized counts or a feature table) — velocity estimation requires unspliced reads
- Dataset exceeds ~50,000 cells on a GPU with 12GB memory and sub-sampling is not acceptable, since accuracy impact of sub-sampling is not yet evaluated
- You need unsupervised selection of dynamically informed genes for projecting cell transitions — the authors state this remains an open challenge

## Inputs

- AnnData (.h5ad) object with spliced and unspliced count matrices and a cell-type label column in adata.obs
- Ground-truth cluster_edges defining expected cluster-to-cluster transitions for CBDir evaluation
- UniTVelo configuration object (R2_ADJUST, IROOT, FIT_OPTION, GPU)

## Outputs

- AnnData annotated with per-gene/per-cell velocities and latent_time
- Velocity embedding stream plots (dpi=100, colored by cell type)
- Cross-Boundary Direction Correctness (CBDir) metrics from utv.evaluate
- Spliced-unspliced phase portraits per gene via utv.pl.plot_range

## How to apply

Load spliced/unspliced matrices into an AnnData object and preprocess with scVelo (gene filtering, moments). Sub-class utv.config.Configuration setting R2_ADJUST=True, IROOT=None, GPU=0, and choose FIT_OPTION: '1' Unified-time mode for datasets where a single latent time resolves directionality discrepancies (e.g., MURK-gene-containing mouse/human erythroid data), or '2' Independent mode for complex multi-branch or cell-cycle datasets (dentate gyrus, E15.5 retina, pancreas Fev+→Delta transitions). Run utv.run_model(path_to_adata, label, config_file=velo), passing the adata.obs cell-type label column; on datasets >50,000 cells, enable the sub-sampling strategy since >12GB GPU memory is needed otherwise. Visualize with scv.pl.velocity_embedding_stream(adata, color=label, dpi=100) and latent time via scv.pl.scatter(color='latent_time', color_map='gnuplot'), then quantify direction correctness with utv.evaluate(adata, cluster_edges, label, 'velocity') using ground-truth cluster_edges.

## Related tools

- **UniTVelo** (Core method: fits spliced/unspliced dynamics via RBF transcription rates, estimates velocities and unified or gene-specific latent time, and provides utv.run_model and utv.evaluate) — github.com/StatBiomed/UniTVelo
- **scVelo** (Preprocessing (gene filtering, moments) and baseline comparator in 'stochastic' and 'dynamical' modes; also used for velocity_embedding_stream and latent-time scatter plots)
- **CellRank** (Cell fate mapping used to validate UniTVelo's recovered Fev+ pre-endocrine to Delta cell transitions on pancreas data)
- **TensorFlow** (Automatic-differentiation backend on which UniTVelo is implemented (TensorFlow 2, GPU-accelerated via CUDA))

## Examples

```
velo = utv.config.Configuration(); velo.R2_ADJUST = True; velo.IROOT = None; velo.FIT_OPTION = '1'; velo.GPU = 0; adata = utv.run_model(path_to_adata, label, config_file=velo)
```

## Evaluation signals

- utv.evaluate returns positive (direction-correct) CBDir values for expected cluster_edges, e.g., 0.793 for UniTVelo on mouse erythroid vs −0.445/0.129 for scVelo dynamical/stochastic
- Velocity stream plots recover expected cell-type ordering, e.g., OPC→COP→NFOL→MFOL oligodendrocyte maturation in P20 hindbrain (CBDir 0.609 vs 0.347 and 0.074)
- In independent mode, retinal trajectory shows cycling progenitors feeding all three branches (PR, RGC, AC/HC) with a connected RGC terminal population
- Latent time scatter (color_map='gnuplot') shows monotonic progression from progenitor to differentiated clusters consistent with IROOT/diffusion-map initialization
- CSV comparison table contains one CBDir row per method × dataset (UniTVelo, scVelo stochastic, scVelo dynamical) matching Table 1 ranking

## Limitations

- Existing RNA velocity methods return erroneous results due to model violation (e.g., MURK genes) or lack of temporal regularization; UniTVelo mitigates but does not eliminate this
- No method for unsupervised selection of dynamically informed genes for projecting cell transitions
- Possible over-counting in unspliced read counting is unresolved and may affect cell transition metrics
- Bayesian treatment of latent time uncertainty is anticipated but not implemented
- Datasets larger than 50,000 cells require more than 12GB GPU memory unless the sub-sampling strategy is used; sub-sampling's effect on accuracy is unevaluated

## Evidence

- [abstract] Uniquely, it also supports the inference of a unified latent time across the transcriptome.: "Uniquely, it also supports the inference of a unified latent time across the transcriptome."
- [readme] Temporally unified RNA velocity for single cell trajectory inference (UniTVelo) is implementated on Python 3 and TensorFlow 2.: "Temporally unified RNA velocity for single cell trajectory inference (UniTVelo) is implementated on Python 3 and TensorFlow 2."
- [readme] velo.FIT_OPTION (str), '1' Unified-time mode (default), '2' Independent mode: "* -- `velo.FIT_OPTION` (str), '1' Unified-time mode (default), '2' Independent mode"
- [readme] metrics = utv.evaluate(adata, cluster_edges, label, 'velocity'): "metrics = utv.evaluate(adata, cluster_edges, label, 'velocity')"
- [methods] Velocity embedding stream visualization via scv.pl.velocity_embedding_stream: "scv.pl.velocity_embedding_stream(adata, color=label, dpi=100, title='')"
