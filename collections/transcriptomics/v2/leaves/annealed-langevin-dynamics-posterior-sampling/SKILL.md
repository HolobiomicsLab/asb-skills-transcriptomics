---
name: annealed-langevin-dynamics-posterior-sampling
description: Use when decomposing aggregated spot-level ST expression (e.g. Visium/Slide-seq
  spots with multiple cells) into per-cell gene expressions given a scRNA-seq reference
  and a trained conditional score model.
when_to_use_negative:
- The goal is only per-spot cell-type proportions — deconvolution methods like RCTD,
  Cell2location, or CARD suffice without per-cell expression sampling
- Single-cell resolution spatial data (e.g. MERFISH) already exists; gene imputation
  rather than spot decomposition is needed
- No trained conditional score network or scRNA-seq reference is available, since
  posterior sampling requires s_theta and cell-type means mu_k
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: SpatialScope
  role: Provides the conditional score network (Train_scRef.py, model checkpoints)
    and implements annealed Langevin posterior sampling for gene expression decomposition
  repo: https://github.com/YangLabHKUST/SpatialScope
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:db1a6384af3e94d67515e6c534a352211632060b19b3a7cfe708cfdd1fad2e64
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLabHKUST/SpatialScope
  tools:
  - SpatialScope
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# Annealed Langevin dynamics posterior sampling

## Summary

Samples per-cell gene expressions from the posterior p(X_i|y_i,k_i) via annealed Langevin dynamics using a conditional score network s_theta(x, sigma_l, mu_k), enabling SpatialScope to decompose aggregated spot-level expression into single-cell expressions.

## When to use

Use when decomposing aggregated spot-level ST expression (e.g. Visium/Slide-seq spots with multiple cells) into per-cell gene expressions given a scRNA-seq reference and a trained conditional score model.

## When NOT to use

- The goal is only per-spot cell-type proportions — deconvolution methods like RCTD, Cell2location, or CARD suffice without per-cell expression sampling
- Single-cell resolution spatial data (e.g. MERFISH) already exists; gene imputation rather than spot decomposition is needed
- No trained conditional score network or scRNA-seq reference is available, since posterior sampling requires s_theta and cell-type means mu_k

## Inputs

- Batch-corrected spot-level gene expression profile y_i (e.g. .h5ad ST data)
- Trained conditional score network checkpoint (e.g. model_5000.pt from Train_scRef.py)
- scRNA-seq reference with per-cell-type mean expression vectors mu_k (e.g. Ref_Heart_sanger_D2.h5ad, top 1000 HVGs + 50 marker genes per type)
- Per-spot cell type labels k_i from the cell type identification step
- Simulated validation spot y = x1 + x2 + noise with known ground-truth per-cell expressions (optional)

## Outputs

- Posterior-mean per-cell gene expression vectors x_i,m at single-cell resolution
- Decomposition validation metrics written to decomposition_validation.csv (cosine similarity to simulated ground truth)

## How to apply

After batch-effect correction (divide each gene count by exp(gamma_g_hat) and renormalize total counts to the reference mean per-cell total), initialize per-cell expressions and run annealed Langevin: X(t+1) = X(t) + eta * [grad_X log p(y|X) + stacked s_theta(x_i,m, sigma_l, mu_k_i,m)] + sqrt(2*eta)*eps. Sweep L=232 geometrically spaced noise levels (sigma_begin=50 down to sigma_end~0.002), with step size eta = eta0 * sigma_l^2 / sigma_1^2 and T=5 steps per level, adding observation noise sigma_yl = sqrt(sigma_l) at each level. Initialize each noise level from the previous level's final state (annealing), repeat over R repeats, and average the final samples to obtain posterior means. For validation, construct simulated two-cell spots y = x1 + x2 + noise with known ground truth via the forward model y_i | X_i ~ N(f(X_i), sigma_y^2 I) with f = sum_m (exp(x_i,m + mu_k_i,m) - 1), and score decomposition quality by cosine similarity between posterior means and ground truth.

## Related tools

- **SpatialScope** (Provides the conditional score network (Train_scRef.py, model checkpoints) and implements annealed Langevin posterior sampling for gene expression decomposition) — https://github.com/YangLabHKUST/SpatialScope

## Examples

```
python Train_scRef.py --sigma_begin 50 --sigma_end 0.002 --step_lr 3e-7; python Nuclei_Seg.py; python Cell_Type_ID.py; python Gene_Exp_Decomposition.py --ckpt model_5000.pt
```

## Evaluation signals

- Cosine similarity between posterior-mean decomposed expressions x1, x2 and the ground-truth simulated cells is high (the metric the paper uses to benchmark decomposition performance)
- The trained checkpoint (e.g. model_5000.pt) loads successfully for posterior sampling
- Posterior samples are averaged over R repeats and per noise level chain-initialization follows annealing (each level starts from the previous level's final state)
- Spot inputs were batch-corrected: gene counts divided by exp(gamma_g_hat) and total counts normalized to the reference mean per-cell total
- The forward model residual is consistent with y_i | X_i ~ N(f(X_i), sigma_y^2 I) with f = sum_m (exp(x_i,m + mu_k_i,m) - 1)

## Limitations

- Training the deep generative score model is time-consuming — measured in hours or days
- Platform effects between ST and scRNA-seq must be appropriately corrected or gene expression decomposition results are unsatisfactory
- For Slide-seq, nucleus segmentation is replaced by singlet/doublet classification, assuming at most two cell types per spot

## Evidence

- [methods] Annealed Langevin dynamics posterior sampling: [evidence span withheld — anchor status: not_found]
- [methods] Batch effect correction before decomposition: "we adjust the batch effects between ST and single-cell reference data before we perform gene expression decomposition"
- [methods] Score model learned from single-cell reference: "We first learn a score-based generative model to approximate the expression distribution of different cell types"
- [methods] Slide-seq two-cell assumption: "we replace the first step nucleus segmentation by singlet/doublet classification, which assumes that at most two cell types co-exist within a Slide-seq spot"
- [discussion] Training cost caveat: "the training time of the deep generative models is measured in hours or days"
