---
name: generalized-em-iteration-rna-velocity
description: Use when fitting TF-based gene dynamics and latent times on scRNA-seq
  data with TFvelo, or when reproducing/tuning the EM optimization loop (weight updates,
  dynamics updates, time assignment) for RNA velocity without splicing counts.
when_to_use_negative:
- 'The dataset is very large and runtime matters: the article notes TFvelo time efficiency
  is lower than scVelo in the same framework, which ''could be a weakness when being
  applied to the large-scale datasets'''
- Splicing (unspliced/spliced) counts are available and a splicing-based dynamical
  model such as scVelo is preferred, since TFvelo was designed for datasets without
  splicing information
- Only summary/feature tables are available without the gene expression matrix needed
  for TF-target correlation-based weight initialization
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: TFvelo
  role: 'Implements the generalized EM algorithm: E-step time assignment and M-steps
    updating TF weights (lsq_linear) and dynamics parameters per gene'
  repo: https://github.com/xiaoyeye/TFvelo
- name: ScVelo
  role: Framework on which TFvelo is built ('we develop TFvelo based on the framework
    of scvelo'); scvelo==0.2.4 is a pinned dependency
  repo: https://github.com/theislab/scvelo
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-45661-w
    title: 'TFvelo: gene regulation inspired RNA velocity estimation'
  source_content_hash: sha256:2211e60deb6ef1ca43e99e94f2e834d70edc0007b6202b5b93476fc20a182939
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaoyeye/TFvelo
  tools:
  - ScVelo
  - TFvelo
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# generalized-em-iteration-rna-velocity

## Summary

Reconstructs TFvelo's generalized EM loop for fitting per-gene dynamics from transcription factor abundance: it alternates a latent-time E-step (grid-search time assignment over n_time_points discrete points) with two M-steps that update TF weights (bounded lsq_linear regression) and the sine-profile dynamics parameters (alpha, beta, theta, gamma).

## When to use

Use when fitting TF-based gene dynamics and latent times on scRNA-seq data with TFvelo, or when reproducing/tuning the EM optimization loop (weight updates, dynamics updates, time assignment) for RNA velocity without splicing counts.

## When NOT to use

- The dataset is very large and runtime matters: the article notes TFvelo time efficiency is lower than scVelo in the same framework, which 'could be a weakness when being applied to the large-scale datasets'
- Splicing (unspliced/spliced) counts are available and a splicing-based dynamical model such as scVelo is preferred, since TFvelo was designed for datasets without splicing information
- Only summary/feature tables are available without the gene expression matrix needed for TF-target correlation-based weight initialization

## Inputs

- Preprocessed scRNA-seq AnnData object (e.g., pancreas dataset, auto-downloaded and preprocessed by TFvelo_run_demo.py)
- TF-target databases: ENCODE (decompressed ENCODE.zip) and ChEA
- Hyperparameters: max_n_TF, n_time_points, WX_thres, max_iter, init_weight_method, WX_method

## Outputs

- Fitted AnnData file TFvelo_pancreas/rc.h5ad containing per-gene dynamics parameters (alpha, beta, theta, gamma), TF weight vectors, and per-cell latent times
- Visualization artifacts (pseudotime/streamplot on UMAP, phase portrait fits) via TFvelo_analysis_demo.py

## How to apply

Run TFvelo via its scvelo-based framework on a preprocessed dataset (e.g., pancreas) with candidate TFs selected per target gene (up to max_n_TF) from ENCODE and ChEA. Initialize the TF weight matrix with init_weight_method=Correlation (Spearman correlation between TF and target expression), initialize gamma from the steady-state estimate, and set multiple starting values for theta. In the E-step, assign each cell a latent time by grid search over n_time_points discrete time points, taking the nearest point on the current model curve. In the M-steps, update each gene's TF weight vector W by bounded linear regression with lsq_linear (trust-region reflective), clipping weights to (-20, 20) via WX_thres, then update the sine-profile parameters alpha, beta, theta, and gamma given current times and weights. Iterate until max_iter iterations of the generalized EM algorithm, then save the fitted AnnData to TFvelo_pancreas/rc.h5ad.

## Related tools

- **TFvelo** (Implements the generalized EM algorithm: E-step time assignment and M-steps updating TF weights (lsq_linear) and dynamics parameters per gene) — https://github.com/xiaoyeye/TFvelo
- **ScVelo** (Framework on which TFvelo is built ('we develop TFvelo based on the framework of scvelo'); scvelo==0.2.4 is a pinned dependency) — https://github.com/theislab/scvelo

## Examples

```
conda activate TFvelo_env && python TFvelo_run_demo.py --dataset_name pancreas
```

## Evaluation signals

- The script runs to completion and the output file TFvelo_pancreas/rc.h5ad exists with fitted per-gene dynamics and per-cell latent times
- TF weights are clipped to the (-20, 20) range enforced by WX_thres and optimized with lsq_linear (trust-region reflective)
- The EM loop completes max_iter iterations of alternating E-step (time assignment over n_time_points) and M-steps (weights, dynamics)
- Running with default parameters reproduces the manuscript results on the pancreas dataset
- Phase portrait fits for best-fitted genes show a desired clockwise curve between TF representation and target

## Limitations

- TFvelo is slower than scVelo in the same framework, a weakness for large-scale datasets; running with more CPUs in parallel can reduce runtime
- TFvelo still cannot fit the dynamics of some genes well (Fig. S14); improving modeling for more genes remains challenging
- The current linear model for estimating TF weights may be insufficient; no implementation for alternative models is provided

## Evidence

- [results] E step: assign each cell a latent time by grid search over n_time_points discrete time points, taking the nearest point on the current model curve.: [evidence span withheld — anchor status: not_found]
- [results] update the TF weight vector W for each gene by bounded linear regression with lsq_linear (trust-region reflective), clipping weights to the (-20, 20) range enforced by WX_thres.: [evidence span withheld — anchor status: not_found]
- [readme] --max_iter: Max number of iterations in the generalized EM algorithm.: "--max_iter: Max number of iterations in the generalized EM algorithm."
- [readme] we develop TFvelo based on the framework of scvelo.: "we develop TFvelo based on the framework of scvelo"
- [readme] The result will be stored in 'TFvelo_pancreas/rc.h5ad'.: "The result will be stored in 'TFvelo_pancreas/rc.h5ad'."
- [readme] --init_weight_method: The method to initialize the weights. Correlation is adopted by default.: "--init_weight_method: The method to initialize the weights. Correlation is adopted by default."
