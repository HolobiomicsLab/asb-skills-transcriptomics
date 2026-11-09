---
name: merfish-gene-imputation-evaluation
description: Use when you need to quantitatively compare gene imputation accuracy
  on image-based ST (e.g., MERFISH mouse1_slice180, 254 genes) against a paired snRNA-seq
  reference by leaving marker genes out and computing MAE.
when_to_use_negative:
- The ST data already measures the whole transcriptome (e.g., Slide-seq V2 at whole-transcriptome
  coverage), so no unmeasured genes need imputation
- The goal is spot-level cell-type deconvolution (proportions per spot) rather than
  single-cell-resolution imputation of unmeasured genes — deconvolution tools like
  RCTD, Cell2location, or CARD are the appropriate choice
- No paired or relevant scRNA-seq reference is available, since all methods here are
  reference-based
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3170
tools:
- name: SpatialScope
  role: Deep generative model (score-based, annealed Langevin dynamics) used to impute
    held-out genes at single-cell resolution
  repo: github.com/YangLabHKUST/SpatialScope
- name: Tangram
  role: Baseline imputation method run with project_genes
- name: gimVI
  role: Baseline imputation method run with model.get_imputed_values(normalized=False)
  repo: github.com/scverse/scvi-tools
- name: SpaGE
  role: Baseline imputation method run with n_pv = Ngenes/2 (since >50 genes are used)
  repo: github.com/tabdelaal/SpaGE
- name: SpaOTsc
  role: Baseline imputation method using the gamma_mapping matrix
  repo: github.com/zcang/SpaOTsc
- name: novoSpaRc
  role: Baseline imputation method run with recommended defaults
  repo: github.com/rajewsky-lab/novosparc
- name: stPlus
  role: Baseline imputation method run with recommended defaults
  repo: github.com/xy-chen16/stPlus
- name: Seurat
  role: Baseline imputation method using TransferData
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:4b7b8750d37feaf951461dea5e3cb29c48e694b7248341a83cc2c4b2552d7e8a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Seurat
  - SpaGE
  - SpaOTsc
  - SpatialScope
  - Tangram
  - gimVI
  - novoSpaRc
  - stPlus
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 16
---

# merfish-gene-imputation-evaluation

## Summary

Benchmark protocol for evaluating whole-transcriptome gene imputation on image-based MERFISH data, in which cortical layer marker genes are held out of the panel, imputed by SpatialScope (conditional score-based generative model with annealed Langevin dynamics) and seven baselines, and scored by mean absolute error against measured expression.

## When to use

Use when you need to quantitatively compare gene imputation accuracy on image-based ST (e.g., MERFISH mouse1_slice180, 254 genes) against a paired snRNA-seq reference by leaving marker genes out and computing MAE.

## When NOT to use

- The ST data already measures the whole transcriptome (e.g., Slide-seq V2 at whole-transcriptome coverage), so no unmeasured genes need imputation
- The goal is spot-level cell-type deconvolution (proportions per spot) rather than single-cell-resolution imputation of unmeasured genes — deconvolution tools like RCTD, Cell2location, or CARD are the appropriate choice
- No paired or relevant scRNA-seq reference is available, since all methods here are reference-based

## Inputs

- MERFISH MOp spatial data (mouse1_slice180, 254 genes)
- Paired droplet-based mouse MOp snRNA-seq reference (26,431 cells, 20 cell types)
- Pretrained SpatialScope checkpoint (e.g., model_5000.pt)
- List of leave-one-out cortical layer marker genes: Cux2, Otof, Rorb, Rspo1, Sulf2, Fezf2, Osr1

## Outputs

- Imputed expression values per marker gene per cell for SpatialScope and each baseline method
- mae_imputation_merfish.csv: table of MAE values for SpatialScope and all seven baselines

## How to apply

Load the MERFISH mouse1_slice180 data (254 genes) and the paired droplet-based mouse MOp snRNA-seq reference (26,431 cells, 20 cell types); normalize total counts per cell to the median transcript count and apply log(1+x) transformation. Train the conditional score-based generative model on the 252 genes overlapping the reference (or load the pretrained checkpoint, e.g. model_5000.pt), then identify cell type labels for each MERFISH cell using the SpatialScope cell type identification step with the Potts spatial smoothness prior (ν = 10 default). Impute each left-out marker by running annealed Langevin dynamics posterior sampling conditioned on the observed 252-gene profile, with the learned score sθ(x, σl, μk), progressive noise reduction from σL to σ1, and step size η = η0·σl²/σ1². Run the baselines per the paper's settings: Tangram (project_genes), gimVI (model.get_imputed_values with normalized=False), SpaGE (n_pv = Ngenes/2 since >50 genes are used), SpaOTsc (gamma_mapping matrix), novoSpaRc and stPlus with recommended defaults, and Seurat TransferData. For each leave-one-out marker (Cux2, Otof, Rorb, Rspo1, Sulf2, Fezf2, Osr1), compute MAE against the measured expression and compile all values into mae_imputation_merfish.csv.

## Related tools

- **SpatialScope** (Deep generative model (score-based, annealed Langevin dynamics) used to impute held-out genes at single-cell resolution) — github.com/YangLabHKUST/SpatialScope
- **Tangram** (Baseline imputation method run with project_genes)
- **gimVI** (Baseline imputation method run with model.get_imputed_values(normalized=False)) — github.com/scverse/scvi-tools
- **SpaGE** (Baseline imputation method run with n_pv = Ngenes/2 (since >50 genes are used)) — github.com/tabdelaal/SpaGE
- **SpaOTsc** (Baseline imputation method using the gamma_mapping matrix) — github.com/zcang/SpaOTsc
- **novoSpaRc** (Baseline imputation method run with recommended defaults) — github.com/rajewsky-lab/novosparc
- **stPlus** (Baseline imputation method run with recommended defaults) — github.com/xy-chen16/stPlus
- **Seurat** (Baseline imputation method using TransferData)

## Examples

```
git clone https://github.com/YangLabHKUST/SpatialScope.git && cd SpatialScope && conda env create -f environment.yml  # then run CellTypeIdentification and GeneExpressionDecomposition notebooks on mouse1_slice180
```

## Evaluation signals

- mae_imputation_merfish.csv shows SpatialScope improving MAE over Tangram, gimVI, SpaOTsc, and novoSpaRc by ~33.6%, 34.3%, 43.4%, and 53.6% respectively, reproducing Fig. 6b, c
- SpatialScope MAE is comparable to SpaGE, stPlus, and Seurat on the seven left-out markers
- Normalization check: per-cell counts sum to the median transcript count before log(1+x) transformation for both MERFISH and snRNA-seq data
- The training gene set contains exactly the 252 MERFISH genes that overlap the snRNA-seq reference; the imputed marker is verified absent from both the MERFISH panel and the model conditioning input
- Cell type identification with Potts prior ν = 10 yields coherent spatial layers consistent with cortical layer marker expression

## Limitations

- Training the deep generative model is time-consuming — measured in hours or days — though matched pretrained checkpoints can be loaded
- The Potts model assumption that neighboring cells belong to the same cell type may not always be valid
- Platform effects must be appropriately corrected between ST and single-cell reference data, or cell type identification and gene expression decomposition results would not be satisfactory
- Image-based ST gene panels capture only a few hundred genes, so imputation accuracy is evaluated only on the small set of left-out marker genes

## Evidence

- [other] SpatialScope significantly outperformed Tangram, gimVI, SpaOTsc, and novoSpaRc in imputing the seven left-out cortical layer marker genes: [evidence span withheld — anchor status: not_found]
- [methods] MERFISH imputation benchmark setup: "we used the slice, mouse1_slice180, from mouse1_sample4 to evaluate the imputation performance"
- [intro] Image-based ST limited panel: "they often lack whole-transcriptome coverage, resulting in only a few hundred genes in real applications"
- [other] Potts prior in cell type identification: [evidence span withheld — anchor status: not_found]
- [readme] SpaGE principal vectors setting: [evidence span withheld — anchor status: external_doc]
- [methods] Normalization protocol: [evidence span withheld — anchor status: not_found]
