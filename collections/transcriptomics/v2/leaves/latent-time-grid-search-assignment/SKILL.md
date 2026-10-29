---
name: latent-time-grid-search-assignment
description: Use when fitting TFvelo gene dynamics models (RNA velocity from TF expression)
  and you need per-cell latent times that must be re-assigned as TF weights and dynamics
  parameters (alpha, beta, theta, gamma) are updated each EM iteration.
when_to_use_negative:
- The dataset lacks suitable TF information or relies on unspliced/spliced counts
  that standard splicing-based velocity models (e.g. scvelo alone) can already model
  well
- 'Speed is critical: TFvelo''s time efficiency is lower than scVelo, a known weakness
  for large-scale datasets'
- You need a fully continuous latent-time estimate rather than assignment to one of
  n_time_points discrete grid positions
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: TFvelo
  role: Implements the generalized EM loop whose E-step performs the grid-search latent
    time assignment over n_time_points; configured via --n_time_points and --max_iter
  repo: github.com/xiaoyeye/TFvelo
- name: ScVelo
  role: Base framework on which TFvelo is developed; scvelo 0.2.4 is a required dependency
    of the TFvelo environment
  repo: theislab/scvelo
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-45661-w
    title: 'TFvelo: gene regulation inspired RNA velocity estimation'
  source_content_hash: sha256:0a35ff5cca5b0014520a4ab0bfb0bf82b8ef8cb43eaf8c45ccc739a2c87ecccf
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ScVelo
  - TFvelo
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# latent-time-grid-search-assignment

## Summary

The E-step of TFvelo's generalized EM algorithm: each cell is assigned a latent time by grid search over n_time_points discrete time points, taking the point nearest to the cell on the current model curve. This couples per-cell temporal assignment to the iteratively refined per-gene dynamics.

## When to use

Use when fitting TFvelo gene dynamics models (RNA velocity from TF expression) and you need per-cell latent times that must be re-assigned as TF weights and dynamics parameters (alpha, beta, theta, gamma) are updated each EM iteration.

## When NOT to use

- The dataset lacks suitable TF information or relies on unspliced/spliced counts that standard splicing-based velocity models (e.g. scvelo alone) can already model well
- Speed is critical: TFvelo's time efficiency is lower than scVelo, a known weakness for large-scale datasets
- You need a fully continuous latent-time estimate rather than assignment to one of n_time_points discrete grid positions

## Inputs

- Preprocessed scRNA-seq AnnData object (e.g. the auto-downloaded pancreas dataset, .h5ad)
- Candidate TFs per target gene selected from ENCODE and ChEA databases (up to max_n_TF per gene)
- Initialized TF weight matrix (init_weight_method=Correlation, Spearman correlation between TF and target expression)
- Steady-state estimates of gamma and multiple starting values for theta
- Hyperparameters: n_time_points, max_iter, WX_thres

## Outputs

- Per-cell latent times stored in the fitted AnnData object
- Updated per-gene TF weight vectors W (clipped to (-20, 20) by WX_thres)
- Updated per-gene sine-profile dynamics parameters alpha, beta, theta, gamma
- Fitted model result saved to TFvelo_pancreas/rc.h5ad

## How to apply

Within each iteration of the generalized EM loop, hold the current model curves fixed and perform a grid search over n_time_points discrete time points; for each cell, locate the nearest point on the model curve of its target genes and assign that time as the cell's latent time. Use the assigned times in the subsequent M steps: first update the TF weight vector W per gene by bounded linear regression (lsq_linear, trust-region reflective, weights clipped by WX_thres to (-20, 20)), then update the sine-profile dynamics parameters alpha, beta, theta, and gamma given the current times and weights. Repeat the E-step and both M-steps until max_iter iterations are completed. The n_time_points parameter controls the granularity of the discrete time grid and thus the resolution of cell ordering; TFvelo builds on the scvelo 0.2.4 framework, which also uses an EM approach for its dynamical model.

## Related tools

- **TFvelo** (Implements the generalized EM loop whose E-step performs the grid-search latent time assignment over n_time_points; configured via --n_time_points and --max_iter) — github.com/xiaoyeye/TFvelo
- **ScVelo** (Base framework on which TFvelo is developed; scvelo 0.2.4 is a required dependency of the TFvelo environment) — theislab/scvelo

## Examples

```
python TFvelo_run_demo.py --dataset_name pancreas
```

## Evaluation signals

- The script runs to completion through max_iter generalized EM iterations without error
- Output file TFvelo_pancreas/rc.h5ad exists and contains fitted per-gene dynamics and per-cell latent times
- Per-cell latent times are restricted to the discrete grid of n_time_points time points used in the E-step
- TF weight vectors remain within the (-20, 20) bound enforced by WX_thres after the M-step
- Downstream visualization (TFvelo_analysis_demo.py) shows coherent pseudotime ordering and streamplot on UMAP

## Limitations

- TFvelo is slower than scVelo, which could be a weakness when applied to large-scale datasets; running with more CPUs in parallel (n_jobs) can reduce runtime
- TFvelo still cannot fit the dynamics of some genes well (Fig. S14), so latent times inferred from poorly-fit genes may be unreliable
- The current linear model for estimating TF weights may be insufficient; alternatives are not implemented
- Latent time is assigned by nearest-point grid search, so its resolution is bounded by n_time_points

## Evidence

- [methods] n_time_points: The number of time points in the time assinment (E step of the generalized EM algorithm).: "--n_time_points: The number of time points in the time assinment (E step of the generalized EM algorithm)."
- [readme] Max number of iterations in the generalized EM algorithm.: "--max_iter: Max number of iterations in the generalized EM algorithm."
- [readme] WX_thres: The max absolute value for weights.: "--WX_thres: The max absolute value for weights."
- [readme] we develop TFvelo based on the framework of scvelo: "Due to the wide usage of scVelo (Bergen, Volker, et al. "Generalizing RNA velocity to transient cell states through dynamical modeling.""
- [readme] the result will be stored in 'TFvelo_pancreas/rc.h5ad': "This will automatically download, preprocess and run TF model on pancrease dataset. The result will be stored in 'TFvelo_pancreas/rc.h5ad'."
- [discussion] the time efficiency of TFvelo is lower than baseline approaches: "the time efficiency of TFvelo is lower than baseline approaches in the same framework, e.g., scVelo."
