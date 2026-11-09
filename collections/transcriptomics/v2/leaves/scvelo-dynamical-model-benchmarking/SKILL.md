---
name: scvelo-dynamical-model-benchmarking
description: Use when spliced/unspliced AnnData from a hematopoietic or erythroid
  differentiation dataset yields suspicious or reversed velocity directions from scVelo,
  and a benchmark against a unified-time method is needed.
when_to_use_negative:
- Datasets without spliced/unspliced count layers or velocity genes identified, since
  both scVelo dynamical mode and UniTVelo require them
- Scenarios where the scVelo dynamical trajectory direction is already correct and
  matches expected biology — no benchmark/correction step is needed
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: scVelo
  role: Likelihood-based 'dynamical' mode run as the distorted-direction baseline
    on the same erythroid AnnData objects
- name: UniTVelo
  role: Unified latent-time RNA velocity method used to correct trajectory directions
    and benchmark against scVelo dynamical mode
  repo: https://github.com/StatBiomed/UniTVelo
- name: Python 3
  role: Implementation environment (UniTVelo is implemented on Python 3 and TensorFlow
    2)
- name: Jupyter
  role: Notebook environment for reproducible analyses (e.g. notebooks/Figure2_ErythroidMouse.ipynb)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-022-34188-7
    title: 'UniTVelo: temporally unified RNA velocity reinforces single-cell trajectory
      inference'
  source_content_hash: sha256:f397aa6a09e1eed4ebe1e53a920329a1cdd175bc18eec6f570c9542b91ae0f5b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/StatBiomed/UniTVelo
  tools:
  - Jupyter
  - Python 3
  - UniTVelo
  - scVelo
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# scvelo-dynamical-model-benchmarking

## Summary

Benchmark scVelo's likelihood-based 'dynamical' mode against UniTVelo's unified latent time on erythroid haematopoiesis datasets containing MURK genes, where scVelo returns distorted or reversed maturation trajectory directions.

## When to use

Use when spliced/unspliced AnnData from a hematopoietic or erythroid differentiation dataset yields suspicious or reversed velocity directions from scVelo, and a benchmark against a unified-time method is needed.

## When NOT to use

- Datasets without spliced/unspliced count layers or velocity genes identified, since both scVelo dynamical mode and UniTVelo require them
- Scenarios where the scVelo dynamical trajectory direction is already correct and matches expected biology — no benchmark/correction step is needed

## Inputs

- Mouse erythroid AnnData object (~9k cells) with spliced/unspliced count matrices
- Human erythroid MEMP-to-Late-Erythroid AnnData object (~30k cells) with spliced/unspliced count matrices
- Cell-type label column in adata.obs (e.g. erythroid maturation stages)
- Ground-truth cluster_edges specifying expected erythroid trajectory transitions

## Outputs

- Velocity embedding stream plots comparing scVelo dynamical mode vs UniTVelo directions
- UniTVelo latent time scatter plot (color_map='gnuplot')
- Spliced-unspliced phase portraits for MURK genes Abcg2, Smim1, Cnn3, Cyr61
- Cross Boundary Direction Correctness metrics from utv.evaluate
- Annotated AnnData (erythroid_velocity_comparison.h5ad) containing both methods' velocities and UniTVelo latent time

## How to apply

Preprocess the mouse erythroid (9k cells) and human erythroid MEMP-to-Late-Erythroid (30k cells) AnnData objects with scVelo so spliced/unspliced matrices and velocity genes are defined, then run scVelo's likelihood-based 'dynamical' mode on the same objects used for UniTVelo (configured with R2_ADJUST=True, IROOT=None, FIT_OPTION='1', GPU=0). Visualize both methods' inferred directions with scv.pl.velocity_embedding_stream (dpi=100, colored by cell-type label) to determine whether scVelo's dynamical mode distorts or reverses the expected erythroid maturation order while UniTVelo corrects it. Project UniTVelo's latent time with scv.pl.scatter (color='latent_time', color_map='gnuplot', size=20) and inspect phase portraits of MURK genes (Abcg2, Smim1, Cnn3, Cyr61) via utv.pl.plot_range (show_ax=True, time_metric='latent_time'). Finally compute Cross Boundary Direction Correctness with utv.evaluate against ground-truth cluster_edges for the erythroid label, and save both methods' velocities plus UniTVelo latent time as erythroid_velocity_comparison.h5ad.

## Related tools

- **scVelo** (Likelihood-based 'dynamical' mode run as the distorted-direction baseline on the same erythroid AnnData objects)
- **UniTVelo** (Unified latent-time RNA velocity method used to correct trajectory directions and benchmark against scVelo dynamical mode) — https://github.com/StatBiomed/UniTVelo
- **Python 3** (Implementation environment (UniTVelo is implemented on Python 3 and TensorFlow 2))
- **Jupyter** (Notebook environment for reproducible analyses (e.g. notebooks/Figure2_ErythroidMouse.ipynb))

## Examples

```
velo = utv.config.Configuration(); velo.R2_ADJUST=True; velo.IROOT=None; velo.FIT_OPTION='1'; velo.GPU=0; adata = utv.run_model(path_to_adata, label, config_file=velo); metrics = utv.evaluate(adata, cluster_edges, label, 'velocity')
```

## Evaluation signals

- scv.pl.velocity_embedding_stream shows UniTVelo returning the expected erythroid maturation direction on both mouse and human datasets while scVelo dynamical mode shows distorted lineage inference
- Cross Boundary Direction Correctness computed via utv.evaluate against ground-truth cluster_edges improves for UniTVelo relative to scVelo dynamical mode
- UniTVelo latent time (plotted with color_map='gnuplot') increases monotonically along the expected erythroid maturation order
- Phase portraits of MURK genes Abcg2, Smim1, Cnn3, Cyr61 can be inspected via utv.pl.plot_range to identify assumption-violating genes
- Final comparison AnnData erythroid_velocity_comparison.h5ad contains velocity layers from both methods plus the latent_time column

## Limitations

- Existing RNA velocity methods often return erroneous results, partly due to model violation by MURK genes or lack of temporal regularization
- No method exists for unsupervised selection of dynamically informed genes for projecting cell transitions — stated as an open challenge
- Possible over-counting in unspliced read counting is not resolved and may affect cell transition metrics; effect of UniTVelo's sub-sampling strategy on accuracy across diverse datasets is unevaluated

## Evidence

- [abstract] UniTVelo corrects trajectory directions on MURK-gene datasets: "With ten datasets, we demonstrate that UniTVelo returns the expected trajectory in different biological systems, including hematopoietic differentiation"
- [intro] Why benchmarking scVelo dynamical mode is needed: "velocity estimations are still found to be inaccurate or inconsistent when recovering cellular transitions13,14, partly because of the severely low signal-to-noise ratio in unspliced mRNAs"
- [intro] scVelo dynamical mode as benchmark: [evidence span withheld — anchor status: not_found]
- [readme] Configuration and model run: "velo = utv.config.Configuration()
velo.R2_ADJUST = True 
velo.IROOT = None
velo.FIT_OPTION = '1'
velo.GPU = 0"
- [methods] Evaluation metrics via utv.evaluate: "# Cross Boundary Direction Correctness
# Ground truth should be given via `cluster_edges`
metrics = {}
metrics = utv.evaluate(adata, cluster_edges, label, 'velocity')"
