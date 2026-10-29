---
name: sine-profile-dynamics-parameter-optimization
description: Use when fitting gene dynamics models from scRNA-seq expression data
  lacking splicing information, by optimizing sine-profile kinetic parameters per
  target gene via a generalized EM algorithm over TF weights and latent times.
when_to_use_negative:
- The dataset already has reliable spliced/unspliced counts and scVelo's dynamical
  EM model is preferred — TFvelo is slower than scVelo, a weakness for large-scale
  datasets
- You need per-gene phase portraits for genes whose dynamics TFvelo cannot fit well
  — the paper notes some genes remain poorly fitted (Fig. S14)
- A nonlinear TF-weight estimator is required — only the bounded linear model (lsq_linear)
  is implemented; the paper notes more complex models may replace it but none are
  provided
edam_operation: http://edamontology.org/operation_3899
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: TFvelo
  role: Implements the generalized EM optimization of sine-profile dynamics parameters
    and TF weights built from TF expression instead of splicing information
  repo: github.com/xiaoyeye/TFvelo
- name: ScVelo
  role: Framework on which TFvelo is developed; scVelo 0.2.4 is installed in the TFvelo
    environment
  repo: theislab/scvelo
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-45661-w
    title: 'TFvelo: gene regulation inspired RNA velocity estimation'
  source_content_hash: sha256:35fb7c3591ceba6572468278cc584a5e5008b0907df6a44de12b60fd5e640e9c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - ScVelo
  - TFvelo
  edam_operation: http://edamontology.org/operation_3899
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# sine-profile-dynamics-parameter-optimization

## Summary

Fits per-gene RNA dynamics in TFvelo by optimizing sine-profile parameters (alpha, beta, theta, gamma) together with TF weights within a generalized EM loop, assigning each cell a latent time on the model curve. It replaces the splicing-based dynamical model of scVelo with a TF-expression-based model.

## When to use

Use when fitting gene dynamics models from scRNA-seq expression data lacking splicing information, by optimizing sine-profile kinetic parameters per target gene via a generalized EM algorithm over TF weights and latent times.

## When NOT to use

- The dataset already has reliable spliced/unspliced counts and scVelo's dynamical EM model is preferred — TFvelo is slower than scVelo, a weakness for large-scale datasets
- You need per-gene phase portraits for genes whose dynamics TFvelo cannot fit well — the paper notes some genes remain poorly fitted (Fig. S14)
- A nonlinear TF-weight estimator is required — only the bounded linear model (lsq_linear) is implemented; the paper notes more complex models may replace it but none are provided

## Inputs

- Preprocessed scRNA-seq AnnData object (e.g. the pancreas dataset, auto-downloaded by TFvelo_run_demo.py)
- TF-target candidate sets from ENCODE and ChEA databases (ENCODE.zip must be decompressed first)
- Hyperparameters: init_weight_method, WX_method, WX_thres, max_n_TF, max_iter, n_time_points, n_neighbors, n_jobs

## Outputs

- Fitted per-gene sine-profile dynamics parameters (alpha, beta, theta, gamma) and TF weight vectors
- Per-cell latent times and RNA velocities
- h5ad result file, e.g. 'TFvelo_pancreas/rc.h5ad'

## How to apply

Build on the scVelo framework (TFvelo is developed on top of scvelo 0.2.4). For each target gene, select up to max_n_TF candidate TFs from ENCODE and ChEA databases and initialize the TF weight matrix via Spearman correlation (init_weight_method = Correlation) between TF and target expression; initialize gamma from the steady-state estimate and set multiple starting values for theta. In the E step, assign each cell a latent time by grid search over n_time_points discrete time points, taking the nearest point on the current model curve. In the M step, alternate between (1) updating the TF weight vector W by bounded linear regression with lsq_linear (trust-region reflective), clipping weights to (-20, 20) as enforced by WX_thres, and (2) updating the sine-profile parameters alpha, beta, theta, and gamma given the current times and weights. Iterate until max_iter iterations of the generalized EM algorithm are completed, then persist the fitted per-gene dynamics and per-cell latent times to an h5ad file (e.g. TFvelo_pancreas/rc.h5ad).

## Related tools

- **TFvelo** (Implements the generalized EM optimization of sine-profile dynamics parameters and TF weights built from TF expression instead of splicing information) — github.com/xiaoyeye/TFvelo
- **ScVelo** (Framework on which TFvelo is developed; scVelo 0.2.4 is installed in the TFvelo environment) — theislab/scvelo

## Examples

```
python TFvelo_run_demo.py --dataset_name pancreas
```

## Evaluation signals

- The script runs to completion for the configured max_iter generalized EM iterations without error
- The output file TFvelo_pancreas/rc.h5ad exists and contains fitted per-gene dynamics parameters and per-cell latent times
- Optimized TF weights satisfy the |W| <= 20 bound enforced by WX_thres
- Phase portraits between TF representation and target genes show a clockwise curve, indicating well-fitted dynamics
- Latent times are discretized to the n_time_points grid used in the E-step assignment

## Limitations

- TFvelo is slower than scVelo; this could be a weakness for large-scale datasets, though running with more CPUs in parallel (n_jobs) can reduce running time
- TFvelo still cannot fit the dynamics of some genes well (Fig. S14); improving modeling for more genes remains challenging
- The current linear model for estimating TF weights may be insufficient; no implementation of alternative estimators is provided

## Evidence

- [intro] ScVelo10 models the dynamics without the steady state assumption and employs an Expectation-Maximization (EM) approach: "ScVelo10 models the dynamics without the steady state assumption and employs an Expectation-Maximization (EM) approach"
- [discussion] TFvelo estimates gene dynamics from TF abundance rather than splicing information: "TFvelo models the dynamics of a target gene with the expression level of TFs and the target gene itself"
- [readme] n_time_points readme: "--n_time_points: The number of time points in the time assinment (E step of the generalized EM algorithm)."
- [readme] WX_thres readme: "--WX_thres: The max absolute value for weights."
- [readme] result storage readme: "This will automatically download, preprocess and run TF model on pancrease dataset. The result will be stored in 'TFvelo_pancreas/rc.h5ad'."
- [readme] TF databases readme: "--TF_databases: The way to select candidate TFs. Use ENCODE and ChEA by default."
