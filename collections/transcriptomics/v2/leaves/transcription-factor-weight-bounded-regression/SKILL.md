---
name: transcription-factor-weight-bounded-regression
description: Use when modeling a target gene's RNA dynamics as a weighted function
  of candidate TF expressions (from ENCODE/ChEA) and you need bounded, solvable TF
  weight estimates during EM-based gene dynamics fitting.
when_to_use_negative:
- The dataset lacks identifiable TF-target regulation (no ENCODE/ChEA candidates for
  the gene), so the weight vector cannot be meaningfully constrained
- You need splicing-based velocity on standard spliced/unspliced data; scVelo's dynamical
  EM model is the appropriate baseline instead
- A nonlinear (non-linear-model) TF-weight estimator is required — the article notes
  the linear model may be insufficient and no alternative implementation is provided
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0605
tools:
- name: TFvelo
  role: Implements the generalized EM loop whose M-step performs the bounded lsq_linear
    TF weight update; built on the scvelo framework
  repo: https://github.com/xiaoyeye/TFvelo
- name: ScVelo
  role: Framework (dynamical EM approach) on which TFvelo's optimization is developed;
    also the splicing-based baseline
  repo: https://github.com/theislab/scvelo
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-45661-w
    title: 'TFvelo: gene regulation inspired RNA velocity estimation'
  source_content_hash: sha256:cac7ee10d2fdb188f57fcc43f4d67da3598e7560675f6a5ab63bb73789318c59
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaoyeye/TFvelo
  tools:
  - ScVelo
  - TFvelo
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0605
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# transcription-factor-weight-bounded-regression

## Summary

Fits per-gene transcription factor weights in TFvelo by bounded linear regression (lsq_linear with trust-region reflective solver) inside the M-step of a generalized EM loop, clipping weights to the (-20, 20) range enforced by WX_thres.

## When to use

Use when modeling a target gene's RNA dynamics as a weighted function of candidate TF expressions (from ENCODE/ChEA) and you need bounded, solvable TF weight estimates during EM-based gene dynamics fitting.

## When NOT to use

- The dataset lacks identifiable TF-target regulation (no ENCODE/ChEA candidates for the gene), so the weight vector cannot be meaningfully constrained
- You need splicing-based velocity on standard spliced/unspliced data; scVelo's dynamical EM model is the appropriate baseline instead
- A nonlinear (non-linear-model) TF-weight estimator is required — the article notes the linear model may be insufficient and no alternative implementation is provided

## Inputs

- Preprocessed single-cell expression AnnData (h5ad), e.g. the auto-downloaded pancreas dataset
- Candidate TF-target edges from the ENCODE and ChEA databases (up to max_n_TF TFs per gene)
- Initial TF weight matrix from Spearman correlation between TF and target expression
- Current per-cell latent times from the E-step of the generalized EM algorithm

## Outputs

- Per-gene bounded TF weight vectors W with values clipped to (-20, 20)
- Fitted model stored as TFvelo_pancreas/rc.h5ad containing per-gene dynamics and per-cell latent times

## How to apply

For each target gene, first select candidate TFs from the ENCODE and ChEA databases (up to max_n_TF per gene) and initialize the TF weight matrix via Spearman correlation between TF and target expression (init_weight_method = Correlation). Within the generalized EM loop, after the E-step assigns each cell a latent time by grid search over n_time_points discrete points on the model curve, update the TF weight vector W by bounded linear regression using scipy's lsq_linear with the trust-region reflective method (WX_method = lsq_linear), clipping weights to the (-20, 20) range set by WX_thres. Alternate this weight M-step with the dynamics M-step (updating alpha, beta, theta, gamma) and the E-step until max_iter iterations complete. The bounded formulation keeps weights physically interpretable and stabilizes the regression when TFs are correlated.

## Related tools

- **TFvelo** (Implements the generalized EM loop whose M-step performs the bounded lsq_linear TF weight update; built on the scvelo framework) — https://github.com/xiaoyeye/TFvelo
- **ScVelo** (Framework (dynamical EM approach) on which TFvelo's optimization is developed; also the splicing-based baseline) — https://github.com/theislab/scvelo

## Examples

```
python TFvelo_run_demo.py --dataset_name pancreas --WX_method lsq_linear --init_weight_method Correlation --WX_thres 20
```

## Evaluation signals

- The script runs to completion and the output file TFvelo_pancreas/rc.h5ad exists with fitted per-gene dynamics and per-cell latent times
- All fitted TF weights fall within the (-20, 20) bounds enforced by WX_thres (no unbounded coefficients)
- Default hyperparameters (WX_method=lsq_linear, init_weight_method=Correlation, ENCODE+ChEA TF databases) reproduce the manuscript results
- Fitted genes show a clockwise curve on the phase portrait between the TF representation and the target gene
- The EM loop terminates after max_iter iterations without divergence of the weight or dynamics parameters

## Limitations

- TFvelo is slower than scVelo, which could be a weakness on large-scale datasets; parallelization over more CPUs can reduce running time
- The current linear model for estimating TF weights may be insufficient; more complex models are suggested but no alternative implementation is provided
- TFvelo still cannot fit the dynamics of some genes well (Fig. S14)

## Evidence

- [methods] M step (weights): update the TF weight vector W for each gene by bounded linear regression with lsq_linear (trust-region reflective), clipping weights to the (-20, 20) range enforced by WX_thres.: [evidence span withheld — anchor status: not_found]
- [readme] --WX_method: The method to optimize weight. lsq_linear is adopted by default.: "--WX_method: The method to optimize weight. lsq_linear is adopted by default."
- [readme] --init_weight_method: The method to initialize the weights. Correlation is adopted by default.: "--init_weight_method: The method to initialize the weights. Correlation is adopted by default."
- [results] TFvelo adopts a generalized EM algorithm that alternately updates three groups of parameters; in the E-step, each cell is assigned a latent time by locating the nearest point on the model curve.: [evidence span withheld — anchor status: not_found]
- [discussion] more complex models may be explored to replace the current linear model in TFvelo for estimating the weight of each TFs: "more complex models may be explored to replace the current linear model in TFvelo for estimating the weight of each TFs"
