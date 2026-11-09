---
name: training-loss-convergence-monitoring
description: Use when training uniPort's coupled-VAE (uniport.Run with default lambda_s=0.5,
  lambda_recon=1.0, lambda_kl=0.5, lambda_ot=1.0, iteration=30000) and you must confirm
  the composite VAE + Minibatch-UOT loss decreases rather than diverges.
when_to_use_negative:
- Input data has not been preprocessed (normalize_total, log1p, MaxAbsScaler 0-1 scaling)
  — loss behavior is then uninterpretable and any divergence reflects bad scaling,
  not model failure
- A different integration framework (e.g. Seurat CCA, Harmony, LIGER NMF) is used
  — those have no coupled-VAE/UOT composite loss whose trajectory can be logged
- Only a single dataset with no cross-dataset alignment goal — the UOT term of the
  loss is then meaningless
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: uniPort
  role: Provides the coupled-VAE + Minibatch-UOT training loop (uniport.Run) whose
    total loss is logged and monitored; default iteration=30000
  repo: https://github.com/caokai1073/uniport
- name: PyTorch
  role: Framework in which the shared Encoder, DSBN decoder and per-dataset Decoders
    are built and from which per-iteration loss values are obtained
- name: scanpy
  role: Loads the preprocessed PBMC AnnData inputs (adata1.h5ad / adata2.h5ad / adata_cm.h5ad)
    that feed the training loop being monitored
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-022-35094-8
    title: A unified computational framework for single-cell data integration with
      optimal transport
  source_content_hash: sha256:26599bae8945b3fb94b028829d0629419195cfa1ed5e42bff387b7844661324e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/caokai1073/uniport
  tools:
  - PyTorch
  - scanpy
  - uniPort
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# training-loss-convergence-monitoring

## Summary

Monitors the total loss L = -L_ELBO* + gamma * L_UOT* of uniPort's coupled-VAE with Minibatch-UOT training to verify the objective is actually being minimized across the 30000 training iterations on datasets such as the PBMC AnnData matrices.

## When to use

Use when training uniPort's coupled-VAE (uniport.Run with default lambda_s=0.5, lambda_recon=1.0, lambda_kl=0.5, lambda_ot=1.0, iteration=30000) and you must confirm the composite VAE + Minibatch-UOT loss decreases rather than diverges.

## When NOT to use

- Input data has not been preprocessed (normalize_total, log1p, MaxAbsScaler 0-1 scaling) — loss behavior is then uninterpretable and any divergence reflects bad scaling, not model failure
- A different integration framework (e.g. Seurat CCA, Harmony, LIGER NMF) is used — those have no coupled-VAE/UOT composite loss whose trajectory can be logged
- Only a single dataset with no cross-dataset alignment goal — the UOT term of the loss is then meaningless

## Inputs

- Preprocessed AnnData matrices: adata1.h5ad and adata2.h5ad (dataset-specific HVGs) and adata_cm.h5ad (k=2000 common HVGs), read with scanpy
- uniport.Run hyperparameters: lambda_s=0.5, lambda_recon=1.0, lambda_kl=0.5, lambda_ot=1.0, iteration=30000
- Per-iteration total loss values emitted by the coupled-VAE training loop (PyTorch)

## Outputs

- Per-iteration log of total loss L = -L_ELBO* + gamma * L_UOT* over 30000 iterations
- training_loss_curve.png loss trajectory plot
- Convergence verdict confirming the coupled-VAE + Minibatch-UOT objective is minimized

## How to apply

During each training iteration, log the total loss L = -L_ELBO* + gamma * L_UOT*, which combines the reconstruction term (lambda_recon, DSBN + dataset-specific decoders), the KL divergence term (lambda_kl=0.5), and the Minibatch-UOT alignment term (lambda_ot=1.0) computed on the shared encoder's Gaussian outputs (mu, sigma) as C_ij = ||mu_x - mu_y||^2 + ||sigma_x - sigma_y||^2. Iterate up to 30000 iterations (one minibatch per iteration), appending the loss value at each step. Plot the logged trajectory (e.g. as training_loss_curve.png) and inspect it for a monotonic-or-noisy-but-decreasing trend rather than divergence or plateau at the initial value. The common/specific gene balance lambda_s=0.5 weights how much the encoder must satisfy UOT alignment across datasets versus within-dataset reconstruction, so a failure to decrease can indicate a badly scaled input (gene values should be MaxAbsScaler-normalized to 0-1 per dataset) or mis-set lambda weights.

## Related tools

- **uniPort** (Provides the coupled-VAE + Minibatch-UOT training loop (uniport.Run) whose total loss is logged and monitored; default iteration=30000) — https://github.com/caokai1073/uniport
- **PyTorch** (Framework in which the shared Encoder, DSBN decoder and per-dataset Decoders are built and from which per-iteration loss values are obtained)
- **scanpy** (Loads the preprocessed PBMC AnnData inputs (adata1.h5ad / adata2.h5ad / adata_cm.h5ad) that feed the training loop being monitored)

## Examples

```
import uniport as up; adata = up.Run(adatas=[adata1, adata2], adata_cm=adata_cm, lambda_s=0.5, lambda_recon=1.0, lambda_kl=0.5, lambda_ot=1.0, iteration=30000)
```

## Evaluation signals

- Logged total loss L = -L_ELBO* + gamma * L_UOT* decreases over the full 30000 training iterations on the PBMC data
- Loss curve (training_loss_curve.png) shows no divergence or sustained plateau at the initial value
- Default hyperparameters lambda_s=0.5, lambda_recon=1.0, lambda_kl=0.5, lambda_ot=1.0 were used (or a justified grid choice from lambda in {0.5,1.0,5.0}, gamma in {0.5,1.0}, lambda_s in {0.5,1.0})
- Downstream latent embeddings from a converged run show coherent cross-dataset alignment consistent with the decreasing UOT term

## Limitations

- Storing the dense OT plan T (save_OT=True) needs substantial memory; computing capacity should be ascertained before using this option
- Which hyperparameter combination from the searched grids (lambda in {0.5,1.0,5.0}, gamma in {0.5,1.0}, lambda_s in {0.5,1.0}) was selected per dataset is not stated, so loss trajectories may not be directly comparable across datasets
- Loss decrease alone does not guarantee biologically meaningful integration; UOT alignment is computed on gene activity scores for scATAC rather than directly modeling regulatory signals

## Evidence

- [readme] uniport.Run defaults and total loss weighting: "**lambda_s**: balanced parameter for common and specific genes. Default: 0.5"
- [readme] iteration default: "**iteration**: max iterations for training. Training one batch_size samples is one iteration. Default: 30000"
- [discussion] grid search of loss weights: "We chose parameters for training coupled-VAE and Minibatch-UOT from λ ∈ {0.5, 1.0, 5.0}, γ ∈ {0.5, 1.0}, and λs ∈ {0.5, 1.0} for all datasets"
- [discussion] input normalization requirement: "We normalized values of each gene to the range of 0-1 within each dataset by the MaxAbsScaler function"
- [discussion] memory caveat for dense OT plan: "Note that storing a dense matrix T needs more memory. Therefore, computing capacity with respect to memory should be ascertained before using this option"
- [readme] save_OT option in README: "**save_OT**: if True, output a global OT plan. Need more memory. Default: False"
