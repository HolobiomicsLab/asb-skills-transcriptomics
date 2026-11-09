---
name: score-based-generative-model-training
description: Use when you need a trained conditional score network from a single-cell
  RNA-seq reference before running SpatialScope's Langevin-dynamics posterior sampling
  for gene expression decomposition or imputation.
when_to_use_negative:
- When only cell-type proportions per spot are needed and single-cell resolution is
  unnecessary — deconvolution methods like RCTD, Cell2location, CARD, or spatialDWLS
  estimate proportions without a generative model
- When no paired scRNA-seq reference is available, since the score network must approximate
  the expression distribution of labeled cell types from reference data
- When a quick alignment of existing reference cells to spots suffices — alignment-based
  methods such as Tangram or CytoSPACE assign cells rather than sampling from a learned
  distribution
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: SpatialScope
  role: Framework providing Train_scRef.py to train the conditional score network
    s_theta(x, sigma_l, mu_k) via denoising score matching and to run annealed Langevin
    dynamics posterior sampling
  repo: https://github.com/YangLabHKUST/SpatialScope
provenance:
  source_task_ids:
  - task_008
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:834662b41439beac364bc0e174953b14a1cd22d15bfc0b56561bd8db987d2852
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YangLabHKUST/SpatialScope
  tools:
  - Seurat
  - SpaGE
  - SpaOTsc
  - SpatialScope
  - Tangram
  - gimVI
  - novoSpaRc
  - stPlus
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# score-based-generative-model-training

## Summary

Trains SpatialScope's conditional score-based generative network s_theta(x, sigma_l, mu_k) on a cell-type-labeled scRNA-seq reference (e.g. Ref_Heart_sanger_D2.h5ad) via denoising score matching, so the learned score function can later drive annealed Langevin dynamics posterior sampling for spot decomposition and gene imputation.

## When to use

Use when you need a trained conditional score network from a single-cell RNA-seq reference before running SpatialScope's Langevin-dynamics posterior sampling for gene expression decomposition or imputation.

## When NOT to use

- When only cell-type proportions per spot are needed and single-cell resolution is unnecessary — deconvolution methods like RCTD, Cell2location, CARD, or spatialDWLS estimate proportions without a generative model
- When no paired scRNA-seq reference is available, since the score network must approximate the expression distribution of labeled cell types from reference data
- When a quick alignment of existing reference cells to spots suffices — alignment-based methods such as Tangram or CytoSPACE assign cells rather than sampling from a learned distribution

## Inputs

- Cell-type-labeled scRNA-seq reference in .h5ad format (e.g. Ref_Heart_sanger_D2.h5ad, MERFISH MOp droplet-based snRNA-seq reference with 26,431 cells across 20 cell types)
- Training hyperparameters: sigma_begin=50, sigma_end=0.002, step_lr=3e-7, L Gaussian noise levels
- Gene selection sets: top 1000 highly variable genes plus 50 marker genes per cell type

## Outputs

- Trained conditional score network checkpoint (e.g. model_5000.pt) usable for posterior sampling
- Posterior samples of per-cell gene expressions obtained via annealed Langevin dynamics; posterior means validated against simulated two-cell spots (cosine similarity written to decomposition_validation.csv)

## How to apply

First normalize the scRNA-seq reference (.h5ad, e.g. Ref_Heart_sanger_D2.h5ad) with total-count normalization and log(1+x) transformation, subtract each cell type's mean expression vector mu_k, and restrict to the top 1000 highly variable genes plus 50 marker genes per cell type (demo reference: 2,000 HVGs, at most 3,000 cells per type). Then train the conditional score network s_theta(x, sigma_l, mu_k) via denoising score matching over L Gaussian noise levels with sigma_L > ... > sigma_1 ~ 0 and loss weights lambda_l = sigma_l^2, using Train_scRef.py with sigma_begin=50, sigma_end=0.002, step_lr=3e-7, saving checkpoints such as model_5000.pt. Before downstream use, correct platform/batch effects on the ST spot profile by dividing each gene count by exp(gamma_g_hat) and normalizing to the reference's mean per-cell total count, since uncorrected platform effects degrade cell type identification and decomposition. The trained checkpoint is consumed by annealed Langevin dynamics (L = 232 noise levels, T = 5 steps per level, step size eta = eta0 * sigma_l^2 / sigma_1^2) to sample from the posterior p(Xi|yi,ki).

## Related tools

- **SpatialScope** (Framework providing Train_scRef.py to train the conditional score network s_theta(x, sigma_l, mu_k) via denoising score matching and to run annealed Langevin dynamics posterior sampling) — https://github.com/YangLabHKUST/SpatialScope

## Examples

```
python Train_scRef.py --sc_data Ref_Heart_sanger_D2.h5ad --sigma_begin 50 --sigma_end 0.002 --step_lr 3e-7
```

## Evaluation signals

- The trained checkpoint (e.g. model_5000.pt) loads successfully and supports annealed Langevin posterior sampling with L = 232 noise levels and T = 5 steps per level
- Posterior-mean decomposed per-cell expressions from simulated two-cell spots (y = x1 + x2 + noise, forward model y_i | X_i ~ N(f(X_i), sigma_y^2 I)) achieve high cosine similarity to ground truth, recorded in decomposition_validation.csv
- Reference preprocessing preserves the expected feature set: total-count normalization, log(1+x) transformation, top 1000 HVGs plus 50 marker genes per cell type
- Loss weights follow lambda_l = sigma_l^2 across noise levels with sigma_L > ... > sigma_1 ~ 0, sigma_begin=50, sigma_end=0.002
- Platform-effect-corrected spot profiles (divided by exp(gamma_g_hat), renormalized to reference mean per-cell total) yield satisfactory downstream cell type identification and decomposition

## Limitations

- Training the deep generative model is time-consuming — the article notes training time is measured in hours or days
- Platform effects between ST and the single-cell reference must be appropriately corrected, otherwise cell type identification and gene expression decomposition results are unsatisfactory
- The score network is conditional on cell type means mu_k, so reference quality, cell type label coverage, and the HVG/marker gene restriction bound what the model can represent

## Evidence

- [methods] Score network is learned from single-cell reference data: "We first learn a score-based generative model to approximate the expression distribution of different cell types"
- [methods] Preprocessing and gene selection for the reference: [evidence span withheld — anchor status: not_found]
- [discussion] Training cost caveat: "it can be time-consuming to train a generative model to approximate the distribution from single-cell reference data"
- [readme] Repository installation for training pipeline: "$ git clone https://github.com/YangLabHKUST/SpatialScope.git"
