---
name: conditional-score-function-learning-from-scrna-seq
description: 'Use when building SpatialScope''s gene expression decomposition: a cell-type-labeled
  scRNA-seq reference (e.g. .h5ad) must be converted into a conditional score network
  usable for posterior sampling of per-cell expressions.'
when_to_use_negative:
- The goal is only cell-type proportion estimation per spot without single-cell-resolution
  expressions — deconvolution methods like RCTD, Cell2location, or CARD estimate proportions
  only and do not need a score model
- The task is gene imputation into sparse ST data (e.g. with Tangram, gimVI, SpaGE),
  which does not require training a generative score function
- No cell-type-labeled scRNA-seq reference is available, since the conditional score
  network requires mu_k per cell type from labeled single-cell data
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: SpatialScope
  role: Provides Train_scRef.py to train the conditional score network on the scRNA-seq
    reference and the annealed Langevin sampler that uses the learned score
  repo: https://github.com/YangLabHKUST/SpatialScope
- name: RCTD (spacexr)
  role: Baseline deconvolution method compared against; performs platform effect normalization
    but no score-based generative modeling
  repo: https://github.com/dmcable/spacexr
- name: Cell2location
  role: Baseline cell-type deconvolution method for comparison
  repo: https://github.com/BayraktarLab/cell2location
- name: CARD
  role: Baseline spatially informed deconvolution method for comparison
  repo: https://github.com/YingMa0107/CARD
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:8ec53f5c6b79eaa1be58164a3b993360f3e89469fcf25f200f3ee08adfe51fd7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLabHKUST/SpatialScope
  tools:
  - CARD
  - Cell2location
  - RCTD (spacexr)
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

# Conditional score function learning from scRNA-seq

## Summary

Train SpatialScope's conditional score-based generative network s_theta(x, sigma_l, mu_k) on a cell-type-labeled scRNA-seq reference via denoising score matching across multiple Gaussian noise levels, so that annealed Langevin dynamics can later sample per-cell expressions for gene expression decomposition.

## When to use

Use when building SpatialScope's gene expression decomposition: a cell-type-labeled scRNA-seq reference (e.g. .h5ad) must be converted into a conditional score network usable for posterior sampling of per-cell expressions.

## When NOT to use

- The goal is only cell-type proportion estimation per spot without single-cell-resolution expressions — deconvolution methods like RCTD, Cell2location, or CARD estimate proportions only and do not need a score model
- The task is gene imputation into sparse ST data (e.g. with Tangram, gimVI, SpaGE), which does not require training a generative score function
- No cell-type-labeled scRNA-seq reference is available, since the conditional score network requires mu_k per cell type from labeled single-cell data

## Inputs

- Cell-type-labeled scRNA-seq reference in .h5ad format (e.g. Ref_Heart_sanger_D2.h5ad), normalized to total counts and log(1+x) transformed
- Gene selection: top 1000 highly variable genes plus 50 marker genes per cell type
- Per-cell-type mean expression vectors mu_k
- Training hyperparameters: sigma_begin=50, sigma_end=0.002, step_lr=3e-7

## Outputs

- Trained conditional score network checkpoint (e.g. model_5000.pt) s_theta(x, sigma_l, mu_k)
- Score evaluations consumed by annealed Langevin dynamics posterior sampling for per-cell expressions

## How to apply

Load the scRNA-seq reference (e.g. Ref_Heart_sanger_D2.h5ad), perform total-count normalization and log(1+x) transformation, subtract each cell type's mean expression vector mu_k, and restrict genes to the top 1000 highly variable genes plus 50 marker genes per cell type. Train the conditional score network s_theta(x, sigma_l, mu_k) with Train_scRef.py using denoising score matching over L Gaussian noise levels (sigma_begin=50, sigma_end=0.002, step_lr=3e-7), with weights lambda_l = sigma_l^2, saving checkpoints such as model_5000.pt. The score function is conditioned on the noise level sigma_l and the cell type mean mu_k so that a single model approximates the expression distribution of all cell types; before decomposition, batch effects on spot-level profiles are corrected by dividing gene counts by exp(gamma_g_hat) and normalizing to the reference mean per-cell total. The learned score is then consumed by annealed Langevin dynamics (L=232 levels, T=5 steps per level) to sample from the posterior p(Xi|yi,ki).

## Related tools

- **SpatialScope** (Provides Train_scRef.py to train the conditional score network on the scRNA-seq reference and the annealed Langevin sampler that uses the learned score) — https://github.com/YangLabHKUST/SpatialScope
- **RCTD (spacexr)** (Baseline deconvolution method compared against; performs platform effect normalization but no score-based generative modeling) — https://github.com/dmcable/spacexr
- **Cell2location** (Baseline cell-type deconvolution method for comparison) — https://github.com/BayraktarLab/cell2location
- **CARD** (Baseline spatially informed deconvolution method for comparison) — https://github.com/YingMa0107/CARD

## Examples

```
python Train_scRef.py --sc-ref Ref_Heart_sanger_D2.h5ad --sigma_begin 50 --sigma_end 0.002 --step_lr 3e-7  # saves checkpoints such as model_5000.pt
```

## Evaluation signals

- The trained checkpoint (e.g. model_5000.pt) loads and supports conditional score evaluation for all cell types
- Posterior-mean decomposed per-cell expressions x1, x2 from a simulated two-cell spot y = x1 + x2 + noise achieve high cosine similarity to the ground-truth simulated cells (written to decomposition_validation.csv)
- Noise-level configuration matches the paper: L = 232 levels spanning sigma_begin=50 down to sigma_end=0.002 with step size eta = eta0 * sigma_l^2 / sigma_1^2 and T = 5 steps per level
- Gene space of the trained model equals top 1000 HVGs plus 50 marker genes per cell type after log(1+x) normalization and mu_k subtraction

## Limitations

- Training the deep generative models is time-consuming — measured in hours or days
- Platform effects must be appropriately corrected (gene-specific gamma_g_hat and depth normalization), otherwise cell type identification and gene expression decomposition results would not be satisfactory
- No software version numbers or computational environment details are reported

## Evidence

- [methods] Conditional score network training: "We first learn a score-based generative model to approximate the expression distribution of different cell types"
- [methods] Posterior sampling with Langevin dynamics: [evidence span withheld — anchor status: not_found]
- [methods] Reference preprocessing and gene selection: [evidence span withheld — anchor status: not_found]
- [methods] Three-step pipeline placement: "the SpatialScope method comprises of three steps: nucleus segmentation, cell type identification, and gene expression decomposition"
- [discussion] Batch effect correction requirement: "Cell type identification and gene expression decomposition results would not be satisfactory if the platform effects are not appropriately corrected."
