---
name: imputation-baseline-method-benchmarking
description: Use when you need to quantitatively compare gene expression imputation
  accuracy across multiple ST integration methods by holding out measured genes and
  computing MAE against ground truth.
when_to_use_negative:
- Your spatial data already covers the whole transcriptome (e.g. Visium or Slide-seq
  with full coverage), so there are no unmeasured genes to impute
- You lack a paired scRNA-seq reference annotated for cell types — all seven benchmarked
  imputation methods are reference-based
- You need cell-type proportion deconvolution per spot rather than whole-transcriptome
  imputation at single-cell resolution (use RCTD, Cell2location, or CARD instead)
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: SpatialScope
  role: Method under evaluation; imputes held-out genes via score-based generative
    model with annealed Langevin dynamics posterior sampling
  repo: https://github.com/YangLabHKUST/SpatialScope
- name: Tangram
  role: Baseline imputation method run with project_genes
- name: gimVI
  role: Baseline imputation method run with model.get_imputed_values(normalized=False)
- name: SpaGE
  role: Baseline imputation method run with n_pv = Ngenes/2 since >50 genes are used
  repo: https://github.com/tabdelaal/SpaGE
- name: SpaOTsc
  role: Baseline imputation method run with the gamma_mapping matrix
  repo: https://github.com/zcang/SpaOTsc
- name: novoSpaRc
  role: Baseline imputation method run with recommended defaults
  repo: https://github.com/rajewsky-lab/novosparc
- name: stPlus
  role: Baseline imputation method run with recommended defaults
  repo: https://github.com/xy-chen16/stPlus
- name: Seurat
  role: Baseline imputation method run with TransferData
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:9cacf327cc98932c7decee5f1724788dab0d0b1ff00f94fff2d90dc100448c3f
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
  - scanpy
  - scikit-learn
  - stPlus
  - uniPort
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- spatial-imputation-benchmark-against-baselines
triage_status: promoted
yes_rate: 1.0
n_claims: 14
---

# imputation-baseline-method-benchmarking

## Summary

Benchmarks SpatialScope against seven existing gene imputation methods (Tangram, gimVI, SpaGE, SpaOTsc, novoSpaRc, stPlus, Seurat) using leave-one-out cortical layer marker genes in the MERFISH MOp dataset, quantifying imputation accuracy via mean absolute error (MAE).

## When to use

Use when you need to quantitatively compare gene expression imputation accuracy across multiple ST integration methods by holding out measured genes and computing MAE against ground truth.

## When NOT to use

- Your spatial data already covers the whole transcriptome (e.g. Visium or Slide-seq with full coverage), so there are no unmeasured genes to impute
- You lack a paired scRNA-seq reference annotated for cell types — all seven benchmarked imputation methods are reference-based
- You need cell-type proportion deconvolution per spot rather than whole-transcriptome imputation at single-cell resolution (use RCTD, Cell2location, or CARD instead)

## Inputs

- MERFISH mouse1_slice180 spatial dataset (254 genes, single-cell resolution)
- Paired droplet-based mouse MOp snRNA-seq reference (26,431 cells, 20 cell types)
- Pretrained SpatialScope checkpoint (e.g. model_5000.pt)
- List of leave-one-out cortical layer marker genes (Cux2, Otof, Rorb, Rspo1, Sulf2, Fezf2, Osr1)

## Outputs

- mae_imputation_merfish.csv: MAE values per method per held-out marker gene
- Per-gene imputed expression vectors from SpatialScope and the seven baselines
- Percent MAE improvement summary (Fig. 6b, c equivalent)

## How to apply

Load the MERFISH mouse1_slice180 data (254 genes) and the paired droplet-based mouse MOp snRNA-seq reference (26,431 cells, 20 cell types); normalize total counts per cell to the median transcript count and apply log(1+x) transformation. Train the conditional score-based generative model on the 252 overlapping training genes (or load a matched pretrained checkpoint such as model_5000.pt), identify per-cell labels with the Potts spatial smoothness prior (nu = 10), and impute each held-out gene via annealed Langevin dynamics posterior sampling with progressive noise reduction from sigma_L to sigma_1 and step size eta = eta0*sigma_l^2/sigma_1^2. Run each baseline with the paper's settings: Tangram (project_genes), gimVI (model.get_imputed_values with normalized=False), SpaGE (n_pv = Ngenes/2 when >50 genes are used), SpaOTsc (gamma_mapping matrix), novoSpaRc and stPlus (recommended defaults), and Seurat (TransferData). For each leave-one-out marker (Cux2, Otof, Rorb, Rspo1, Sulf2, Fezf2, Osr1), compute MAE against the measured expression and compile results into mae_imputation_merfish.csv.

## Related tools

- **SpatialScope** (Method under evaluation; imputes held-out genes via score-based generative model with annealed Langevin dynamics posterior sampling) — https://github.com/YangLabHKUST/SpatialScope
- **Tangram** (Baseline imputation method run with project_genes)
- **gimVI** (Baseline imputation method run with model.get_imputed_values(normalized=False))
- **SpaGE** (Baseline imputation method run with n_pv = Ngenes/2 since >50 genes are used) — https://github.com/tabdelaal/SpaGE
- **SpaOTsc** (Baseline imputation method run with the gamma_mapping matrix) — https://github.com/zcang/SpaOTsc
- **novoSpaRc** (Baseline imputation method run with recommended defaults) — https://github.com/rajewsky-lab/novosparc
- **stPlus** (Baseline imputation method run with recommended defaults) — https://github.com/xy-chen16/stPlus
- **Seurat** (Baseline imputation method run with TransferData)

## Examples

```
git clone https://github.com/YangLabHKUST/SpatialScope.git; python SpatialScope.py --mitochonderial_percentage=None --reference='Mouse_MOp' --ST='mouse1_slice180' --Cell_Type='Cell_type_Nuclei_segmentation' --ckpt='model_5000.pt'
```

## Evaluation signals

- SpatialScope MAE improvements over baselines should reproduce ~33.6% (Tangram), ~34.3% (gimVI), ~43.4% (SpaOTsc), and ~53.6% (novoSpaRc), matching Fig. 6b, c
- SpatialScope MAE should be comparable to SpaGE, stPlus, and Seurat on the seven cortical layer markers
- All seven leave-one-out markers (Cux2, Otof, Rorb, Rspo1, Sulf2, Fezf2, Osr1) must be absent from the 252-gene training panel before imputation
- The compiled mae_imputation_merfish.csv must contain one MAE entry per method per marker with no missing values

## Limitations

- Training the deep generative model is time-consuming, measured in hours or days
- The Potts model assumption that neighboring cells belong to the same cell type may not always be valid
- Platform effects between ST and scRNA-seq data must be corrected or cell type identification and gene expression decomposition results would not be satisfactory

## Evidence

- [other] SpatialScope significantly outperformed Tangram, gimVI, SpaOTsc, and novoSpaRc in imputing the seven left-out cortical layer marker genes: [evidence span withheld — anchor status: not_found]
- [methods] MOp dataset selection: "we used the slice, mouse1_slice180, from mouse1_sample4 to evaluate the imputation performance"
- [methods] Reference dataset description: "a paired droplet-based snRNA-seq profiles collected from mouse MOp as the reference, which measures the expression of 26,431 cells across 20 cell types"
- [methods] Posterior sampling procedure: [evidence span withheld — anchor status: not_found]
- [intro] Baseline methods inadequate: "methods developed to infer unmeasured gene expressions, such as Tangram18, gimVI19 and SpaGE20 are not sufficiently accurate"
- [readme] SpatialScope repo install: "$ git clone https://github.com/YangLabHKUST/SpatialScope.git"
