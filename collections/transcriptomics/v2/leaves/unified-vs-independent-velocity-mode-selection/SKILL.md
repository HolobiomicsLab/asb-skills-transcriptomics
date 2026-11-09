---
name: unified-vs-independent-velocity-mode-selection
description: Use when running UniTVelo on a count-matrix AnnData and you must decide
  between the unified latent time model and gene-independent latent time, e.g. for
  simple differentiation vs complex multi-branching systems like bone marrow differentiation.
when_to_use_negative:
- The goal is comparing against scVelo rather than configuring UniTVelo — run scVelo's
  'stochastic' (covariance-based) and 'dynamical' (likelihood-based) modes instead;
  UniTVelo's FIT_OPTION does not apply to baselines
- The dataset lacks spliced/unspliced RNA layers or a meaningful cell-type label in
  adata.obs, since utv.run_model requires both for fitting and direction evaluation
- The trajectory requires chromatin accessibility integration (e.g. MultiVelo-style
  multiomic velocity), which UniTVelo's two FIT_OPTION modes do not model
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: UniTVelo
  role: RNA velocity framework whose Configuration object exposes FIT_OPTION ('1'
    Unified-time mode, '2' Independent mode), R2_ADJUST, IROOT, and GPU; run via utv.run_model
    and evaluated via utv.evaluate
  repo: https://github.com/StatBiomed/UniTVelo
- name: scVelo
  role: Supplies the AnnData count-matrix datasets used as inputs and the comparator
    'stochastic' and 'dynamical' modes; its plotting functions (scv.pl.velocity_embedding_stream)
    visualize the fitted velocity field
- name: Python 3
  role: Implementation language; UniTVelo is built on Python 3 and TensorFlow 2, installed
    via pip install unitvelo
  repo: https://github.com/StatBiomed/UniTVelo
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-022-34188-7
    title: 'UniTVelo: temporally unified RNA velocity reinforces single-cell trajectory
      inference'
  source_content_hash: sha256:3d0f9d02f3bfcfeb0d821790a6914f4876fec2386a7df0973defc121c04a5872
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/StatBiomed/UniTVelo
  tools:
  - Python 3
  - UniTVelo
  - scVelo
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# unified-vs-independent-velocity-mode-selection

## Summary

Guides the choice between UniTVelo's FIT_OPTION '1' (Unified-time mode, one shared latent time across the transcriptome) and FIT_OPTION '2' (Independent mode, gene-specific time metrics) when fitting RNA velocity on a dataset. The unified mode incorporates stably and monotonically changed genes, while the independent mode suits complex, multi-branching trajectories.

## When to use

Use when running UniTVelo on a count-matrix AnnData and you must decide between the unified latent time model and gene-independent latent time, e.g. for simple differentiation vs complex multi-branching systems like bone marrow differentiation.

## When NOT to use

- The goal is comparing against scVelo rather than configuring UniTVelo — run scVelo's 'stochastic' (covariance-based) and 'dynamical' (likelihood-based) modes instead; UniTVelo's FIT_OPTION does not apply to baselines
- The dataset lacks spliced/unspliced RNA layers or a meaningful cell-type label in adata.obs, since utv.run_model requires both for fitting and direction evaluation
- The trajectory requires chromatin accessibility integration (e.g. MultiVelo-style multiomic velocity), which UniTVelo's two FIT_OPTION modes do not model

## Inputs

- Count-matrix AnnData (.h5ad) with spliced and unspliced layers, e.g. from scVelo datasets or the authors' SharePoint-hosted ten public datasets (pancreas, dentate gyrus, retina, scNT-seq GSE158646, scEU-seq E-MTAB-7713, hindbrain, mouse erythroid, human erythroid, bone marrow)
- Cell-type label column name in adata.obs passed as the 'label' argument to utv.run_model
- Ground-truth cluster_edges for Cross-Boundary Direction Correctness evaluation

## Outputs

- Fitted AnnData with velocity estimates, latent_time, and velocity graph produced by utv.run_model
- CBDir (Cross-Boundary Direction Correctness) metrics dict from utv.evaluate for comparing mode choices
- Velocity embedding stream plots and per-gene phase portraits via scv.pl.velocity_embedding_stream and utv.pl.plot_range

## How to apply

Subclass utv.config.Configuration and set FIT_OPTION per the dataset's trajectory complexity, following Supplementary Table S2 mappings used in the ten-dataset benchmark. Set FIT_OPTION='1' (Unified-time mode, the default) for datasets with clear progression such as mouse erythroid development, pancreas, or retina, where a single unified latent time across the whole transcriptome regularizes directionality and resolves discrepancies between genes. Set FIT_OPTION='2' (Independent mode) for complex or multi-branching datasets such as bone marrow differentiation, where each gene retains its own latent time metric. Alongside FIT_OPTION, set R2_ADJUST=True (R-squared regression on full data rather than extreme quantile) and IROOT=None unless a root cell cluster is known, and specify velo.GPU (0 by default, -1 for CPU). Run via utv.run_model(path_to_adata, label, config_file=velo) with label naming the adata.obs celltype column, then verify the choice by computing Cross-Boundary Direction Correctness with utv.evaluate(adata, cluster_edges, label, 'velocity') against ground-truth cluster_edges.

## Related tools

- **UniTVelo** (RNA velocity framework whose Configuration object exposes FIT_OPTION ('1' Unified-time mode, '2' Independent mode), R2_ADJUST, IROOT, and GPU; run via utv.run_model and evaluated via utv.evaluate) — https://github.com/StatBiomed/UniTVelo
- **scVelo** (Supplies the AnnData count-matrix datasets used as inputs and the comparator 'stochastic' and 'dynamical' modes; its plotting functions (scv.pl.velocity_embedding_stream) visualize the fitted velocity field)
- **Python 3** (Implementation language; UniTVelo is built on Python 3 and TensorFlow 2, installed via pip install unitvelo) — https://github.com/StatBiomed/UniTVelo

## Examples

```
velo = utv.config.Configuration(); velo.R2_ADJUST = True; velo.IROOT = None; velo.FIT_OPTION = '1'; velo.GPU = 0; adata = utv.run_model(path_to_adata, label, config_file=velo)
```

## Evaluation signals

- The Configuration object has FIT_OPTION set to '1' or '2' (matching Supplementary Table S2 for that dataset) before calling utv.run_model
- utv.evaluate(adata, cluster_edges, label, 'velocity') returns a Cross-Boundary Direction Correctness score that is positive and consistent with the expected trajectory direction reported in Table 1
- In Unified-time mode, scv.pl.scatter on 'latent_time' shows a coherent monotonic gradient across cells rather than per-gene inconsistency
- TensorFlow reports the expected GPU (velo.GPU=0) or falls back to CPU (-1) without error, confirming the fitting ran on the intended device
- Re-running with the alternative FIT_OPTION on a known dataset (e.g. mouse erythroid, expected CBDir 0.793) confirms the chosen mode reproduces the paper's relative ranking

## Limitations

- The article leaves method selection largely empirical: which datasets need Independent vs Unified-time mode is specified per Supplementary Table S2 rather than by an automatic rule
- Existing RNA velocity results can be erroneous due to model violation or lack of temporal regularization, and severely low signal-to-noise ratio in unspliced mRNAs can bias either mode
- Possible over-counting in unspliced read counting may cause a few genes to dramatically affect cell transition metrics from the RNA velocity vector
- It remains an open challenge to select dynamically informed genes for projecting cell transitions in an unsupervised manner, affecting mode comparison
- Datasets larger than ~50,000 cells may need a GPU with more than 12GB memory or the sub-sampling strategy

## Evidence

- [readme] FIT_OPTION configuration: "* -- `velo.FIT_OPTION` (str), '1' Unified-time mode (default), '2' Independent mode"
- [abstract] unified latent time rationale: "Uniquely, it also supports the inference of a unified latent time across the transcriptome."
- [intro] unified time resolves gene directionality discrepancy: "we also introduce a unified latent time across the whole transcriptome, which can resolve the discrepancy of directionality between genes"
- [readme] two modes in README features: "Retaining gene-spcific time matrics (`Independent mode`) for complex datasets"
- [methods] run and evaluate workflow: "adata = utv.run_model(path_to_adata, label, config_file=velo)"
- [methods] implementation stack: "Temporally unified RNA velocity for single cell trajectory inference (UniTVelo) is implementated on Python 3 and TensorFlow 2."
