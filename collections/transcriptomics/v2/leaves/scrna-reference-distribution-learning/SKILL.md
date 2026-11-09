---
name: scrna-reference-distribution-learning
description: Use when a spatial transcriptomics dataset (Visium, Slide-seq V2, MERFISH)
  must be integrated with a cell-type-annotated scRNA-seq reference and a generative
  model of single-cell expression distributions is needed for cell type identification
  or gene expression imputation/decomposition.
when_to_use_negative:
- The goal is only spot-level cell-type proportion estimates, not single-cell-resolution
  decomposition or whole-transcriptome imputation — deconvolution methods like RCTD,
  Cell2location, or CARD are cheaper and sufficient.
- No cell-type-annotated single-cell reference is available for the tissue of interest,
  since the model is trained directly on reference expression distributions.
- 'Training turnaround is unacceptable: the paper notes deep generative model training
  is measured in hours or days, and pretrained checkpoints may not exist for your
  reference.'
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SpatialScope
  role: Implements conditional score-based generative model training on the scRNA-seq
    reference and annealed Langevin dynamics posterior sampling for ST integration
  repo: https://github.com/YangLabHKUST/SpatialScope
- name: Tangram
  role: Baseline imputation method compared against the learned generative model (project_genes);
    SpatialScope improved MAE by ~33.6% over Tangram on MERFISH MOp
- name: gimVI
  role: Baseline imputation method (model.get_imputed_values with normalized=False);
    SpatialScope improved MAE by ~34.3% over gimVI
  repo: https://github.com/scverse/scvi-tools
- name: SpaOTsc
  role: Baseline optimal-transport imputation method (gamma_mapping matrix); SpatialScope
    improved MAE by ~43.4% over SpaOTsc
  repo: https://github.com/zcang/SpaOTsc
- name: novoSpaRc
  role: Baseline imputation method with recommended defaults; SpatialScope improved
    MAE by ~53.6% over novoSpaRc
  repo: https://github.com/rajewsky-lab/novosparc
- name: SpaGE
  role: Baseline imputation method with n_pv = Ngenes/2 when >50 training genes are
    used; performed comparably to SpatialScope
- name: stPlus
  role: Autoencoder baseline imputation method with recommended defaults; performed
    comparably to SpatialScope
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:5ca4b6d6b499c418aea7258be8457a7659f9bd2cd46c3c3992e6c7de0184b758
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.95
n_claims: 20
---

# scrna-reference-distribution-learning

## Summary

Train a conditional score-based deep generative model on a cell-type-labeled scRNA-seq reference (e.g., the mouse MOp droplet snRNA-seq atlas, 26,431 cells, 20 cell types) so its learned score function sθ(x, σl, μk) can later support posterior sampling of transcriptome-wide expression for spatial transcriptomics via SpatialScope.

## When to use

Use when a spatial transcriptomics dataset (Visium, Slide-seq V2, MERFISH) must be integrated with a cell-type-annotated scRNA-seq reference and a generative model of single-cell expression distributions is needed for cell type identification or gene expression imputation/decomposition.

## When NOT to use

- The goal is only spot-level cell-type proportion estimates, not single-cell-resolution decomposition or whole-transcriptome imputation — deconvolution methods like RCTD, Cell2location, or CARD are cheaper and sufficient.
- No cell-type-annotated single-cell reference is available for the tissue of interest, since the model is trained directly on reference expression distributions.
- Training turnaround is unacceptable: the paper notes deep generative model training is measured in hours or days, and pretrained checkpoints may not exist for your reference.

## Inputs

- Cell-type-labeled scRNA-seq/snRNA-seq reference matrix (e.g., mouse MOp droplet-based snRNA-seq, 26,431 cells, 20 cell types, 24,387 genes)
- Spatial transcriptomics count matrix (e.g., MERFISH mouse1_slice180 with 254 genes, Visium 3813-spot human heart, Slide-seq V2 mouse cerebellum)
- Pretrained score-model checkpoint (e.g., model_5000.pt) or training config for the conditional score-based generative model

## Outputs

- Trained conditional score-based generative model sθ(x, σl, μk) of per-cell-type expression distributions
- Imputed transcriptome-wide gene expression values per spatial cell/spot (e.g., MAE table mae_imputation_merfish.csv for left-out markers Cux2, Otof, Rorb, Rspo1, Sulf2, Fezf2, Osr1)
- Posterior samples p(Xi|yi,ki) per cell used for cell type identification and gene expression decomposition

## How to apply

Normalize the single-cell reference with total counts normalization to the median transcript count and log(1+x) transformation, and adjust batch effects (gene-specific and sequencing-depth differences) between the ST data and the reference before decomposition. Train the conditional score-based generative model on the reference to approximate the expression distribution of each cell type, or load a matched pretrained checkpoint (e.g., model_5000.pt for the mouse MOp snRNA-seq reference). Restrict training genes to those overlapping both modalities (e.g., 252 of 254 MERFISH MOp genes overlap the 26,431-cell snRNA-seq reference). At inference time, sample from the posterior p(Xi|yi,ki) using annealed Langevin dynamics with progressive noise reduction from σL to σ1 and step size η = η0·σl²/σ1², conditioned on the observed spatial expression profile and cell-type label (Potts spatial smoothness prior, ν = 10 default). Rationale: the learned per-cell-type score function enables transcriptome-wide imputation and single-cell-resolution decomposition that deconvolution methods (RCTD, Cell2location, CARD) and alignment-based imputation (Tangram, gimVI) cannot achieve on sparse ST data.

## Related tools

- **SpatialScope** (Implements conditional score-based generative model training on the scRNA-seq reference and annealed Langevin dynamics posterior sampling for ST integration) — https://github.com/YangLabHKUST/SpatialScope
- **Tangram** (Baseline imputation method compared against the learned generative model (project_genes); SpatialScope improved MAE by ~33.6% over Tangram on MERFISH MOp)
- **gimVI** (Baseline imputation method (model.get_imputed_values with normalized=False); SpatialScope improved MAE by ~34.3% over gimVI) — https://github.com/scverse/scvi-tools
- **SpaOTsc** (Baseline optimal-transport imputation method (gamma_mapping matrix); SpatialScope improved MAE by ~43.4% over SpaOTsc) — https://github.com/zcang/SpaOTsc
- **novoSpaRc** (Baseline imputation method with recommended defaults; SpatialScope improved MAE by ~53.6% over novoSpaRc) — https://github.com/rajewsky-lab/novosparc
- **SpaGE** (Baseline imputation method with n_pv = Ngenes/2 when >50 training genes are used; performed comparably to SpatialScope)
- **stPlus** (Autoencoder baseline imputation method with recommended defaults; performed comparably to SpatialScope)

## Examples

```
git clone https://github.com/YangLabHKUST/SpatialScope.git; cd SpatialScope; conda env create -f environment.yml
```

## Evaluation signals

- Leave-one-out marker MAE: impute each left-out cortical layer marker (Cux2, Otof, Rorb, Rspo1, Sulf2, Fezf2, Osr1) and confirm MAE improvements of ~33.6%, 34.3%, 43.4%, and 53.6% over Tangram, gimVI, SpaOTsc, and novoSpaRc, with parity to SpaGE, stPlus, and Seurat (Fig. 6b, c).
- mаемаe_imputation_merfish.csv contains MAE rows for SpatialScope plus all seven baselines with values matching the reported relative improvements.
- Normalization invariant: reference cells are total-count normalized to the median transcript count and log(1+x) transformed before training.
- Gene overlap check: training genes equal the intersection of ST panel and reference genes (e.g., 252 of 254 MERFISH genes overlapping the snRNA-seq reference).
- Sampling diagnostics: annealed Langevin dynamics progresses monotonically from σL to σ1 with step size η = η0·σl²/σ1² and Potts prior ν = 10.

## Limitations

- Training the deep generative model is time-consuming, measured in hours or days; use the matched pretrained checkpoint when available (e.g., model_5000.pt).
- Cell type identification and gene expression decomposition results would not be satisfactory if platform effects between ST and single-cell data are not appropriately corrected.
- The Potts model assumption that neighboring cells belong to the same cell type may not always be valid; adaptive learning of neighborhood similarity via DGI/GCN is left for future work.

## Evidence

- [methods] Learn a conditional score-based generative model from single-cell reference data: "We first learn a score-based generative model to approximate the expression distribution of different cell types"
- [methods] Posterior sampling via annealed Langevin dynamics: [evidence span withheld — anchor status: not_found]
- [methods] Batch effect correction before decomposition: "we adjust the batch effects between ST and single-cell reference data before we perform gene expression decomposition"
- [results] Benchmark result versus baselines: [evidence span withheld — anchor status: not_found]
- [discussion] Training cost caveat: "the training time of the deep generative models is measured in hours or days"
