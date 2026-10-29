---
name: spearman-correlation-weight-initialization
description: Reference for initializing TF weight matrices when fitting TFvelo gene
  dynamics models, where candidate TFs (ENCODE/ChEA) need expression-correlation-based
  starting weights per target gene.
when_to_use_negative:
- Target genes with no candidate TFs recovered from ENCODE/ChEA, since there are no
  TF-target pairs to correlate
- Datasets where expression correlation is a poor proxy for regulation and a different
  init_weight_method is warranted
- Running ScVelo's splicing-based EM instead, where TF weights are not part of the
  model at all
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: TFvelo
  role: Implements the correlation-based weight initialization and subsequent generalized
    EM weight/dynamics optimization
  repo: https://github.com/xiaoyeye/TFvelo
- name: ScVelo
  role: Underlying EM/dynamical framework on which TFvelo is built; its steady-state
    estimate initializes gamma
  repo: https://github.com/theislab/scvelo
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-45661-w
    title: 'TFvelo: gene regulation inspired RNA velocity estimation'
  source_content_hash: sha256:7f3eba1ffa085603b28fee87838bb06cb9b81286be52cee9315e48e3f7e0ae43
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xiaoyeye/TFvelo
  tools:
  - ScVelo
  - TFvelo
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Spearman-correlation-weight-initialization

## Summary

TFvelo initializes the per-gene transcription-factor weight matrix via Spearman correlation between TF and target gene expression (init_weight_method = Correlation), providing a regulation-informed starting point before the generalized EM refines weights by bounded lsq_linear regression.

## When to use

Reference for initializing TF weight matrices when fitting TFvelo gene dynamics models, where candidate TFs (ENCODE/ChEA) need expression-correlation-based starting weights per target gene.

## When NOT to use

- Target genes with no candidate TFs recovered from ENCODE/ChEA, since there are no TF-target pairs to correlate
- Datasets where expression correlation is a poor proxy for regulation and a different init_weight_method is warranted
- Running ScVelo's splicing-based EM instead, where TF weights are not part of the model at all

## Inputs

- Preprocessed scRNA-seq expression matrix (h5ad, e.g. pancreas dataset) with spliced/unspliced-independent expression levels
- Candidate TF list per target gene selected from ENCODE and ChEA databases (up to max_n_TF per gene)
- Steady-state estimate of gamma per gene
- Hyperparameters: init_weight_method, WX_thres, max_n_TF, max_iter, n_time_points

## Outputs

- Initialized per-gene TF weight matrix (Spearman-correlation-based weights within WX_thres)
- Fitted per-gene dynamics parameters (alpha, beta, theta, gamma) and TF weights after EM
- Annotated AnnData with per-cell latent times saved to TFvelo_<dataset>/rc.h5ad

## How to apply

After selecting candidate TFs for each target gene (up to max_n_TF per gene from ENCODE and ChEA), set init_weight_method = Correlation so the initial weight of each TF for a target gene is its Spearman rank correlation between TF and target expression. This warm-starts the M-step weight update, which then runs bounded linear regression (WX_method = lsq_linear, trust-region reflective) with weights clipped to the (-20, 20) range enforced by WX_thres. The initialized weights, together with gamma from the steady-state estimate and multiple starting values for theta, feed the generalized EM loop: E-step assigns latent times by grid search over n_time_points, and M-steps alternately update weights and sine-profile dynamics parameters (alpha, beta, theta, gamma) until max_iter iterations complete.

## Related tools

- **TFvelo** (Implements the correlation-based weight initialization and subsequent generalized EM weight/dynamics optimization) — https://github.com/xiaoyeye/TFvelo
- **ScVelo** (Underlying EM/dynamical framework on which TFvelo is built; its steady-state estimate initializes gamma) — https://github.com/theislab/scvelo

## Examples

```
python TFvelo_run_demo.py --dataset_name pancreas --init_weight_method Correlation
```

## Evaluation signals

- Initial weights are finite and within the (-20, 20) bound enforced by WX_thres
- Each gene's weight vector has dimension ≤ max_n_TF, matching the ENCODE/ChEA candidate TF set for that gene
- Initialization uses Spearman (rank) correlation, as specified by init_weight_method = Correlation, not Pearson
- Subsequent M-step weight update runs lsq_linear (trust-region reflective) without divergence or clipping violations
- Pipeline completes max_iter EM iterations and writes TFvelo_pancreas/rc.h5ad with fitted dynamics and per-cell latent times

## Limitations

- The article notes the current linear model for estimating TF weights may be insufficient and no implementation for alternatives is provided
- TFvelo is slower than ScVelo, which could be a weakness when applied to large-scale datasets
- TFvelo still cannot fit the dynamics of some genes well (Fig. S14)

## Evidence

- [methods] Initialize the TF weight matrix using the correlation method (init_weight_method = Correlation, Spearman correlation between TF and target expression), initialize gamma from the steady-state estimate: [evidence span withheld — anchor status: not_found]
- [readme] README hyperparameter description: "--init_weight_method: The method to initialize the weights. Correlation is adopted by default."
- [methods] M-step weight update description: [evidence span withheld — anchor status: not_found]
- [methods] TF database selection: "--TF_databases: The way to select candidate TFs. Use ENCODE and ChEA by default."
- [readme] EM iteration limit: "--max_iter: Max number of iterations in the generalized EM algorithm."
