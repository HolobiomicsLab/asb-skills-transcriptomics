---
name: leave-one-out-marker-gene-validation
description: Use when an image-based ST panel omits genes that are measured in a reference
  or validation set, and you need to quantify imputation accuracy (MAE) against measured
  expression for held-out marker genes.
when_to_use_negative:
- The target genes have no measured ground-truth expression in the ST data, so MAE
  against measured values cannot be computed
- The ST data is seq-based (e.g. Visium) with spot-level mixtures rather than single-cell
  image-based data where leave-one-out on a gene panel is defined
- The reference scRNA-seq does not cover the held-out genes, leaving no basis for
  the generative model to impute them
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3170
tools:
- name: SpatialScope
  role: Imputes held-out marker genes via annealed Langevin dynamics posterior sampling
    with the learned score function sθ(x, σl, μk)
  repo: https://github.com/YangLabHKUST/SpatialScope
- name: Tangram
  role: Baseline imputation via project_genes
- name: gimVI
  role: Baseline imputation via model.get_imputed_values with normalized=False
  repo: https://github.com/scverse/scvi-tools
- name: SpaGE
  role: Baseline imputation with n_pv = Ngenes/2
  repo: https://github.com/tabdelaal/SpaGE
- name: SpaOTsc
  role: Baseline imputation using the gamma_mapping matrix
  repo: https://github.com/zcang/SpaOTsc
- name: novoSpaRc
  role: Baseline imputation with recommended defaults
  repo: https://github.com/rajewsky-lab/novosparc
- name: stPlus
  role: Baseline imputation with recommended defaults
  repo: https://github.com/xy-chen16/stPlus
- name: Seurat
  role: Baseline imputation via TransferData
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-023-43629-w
    title: Integrating spatial and single-cell transcriptomics data using deep generative
      models with SpatialScope
  source_content_hash: sha256:8a9afb2dca60abc9c87f81c5c86eac484ffb67da5d0aaa120693b6c01fd38116
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
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 18
---

# Leave-one-out marker gene validation

## Summary

Benchmark spatial gene-expression imputation accuracy by removing known cortical layer marker genes (Cux2, Otof, Rorb, Rspo1, Sulf2, Fezf2, Osr1) from a MERFISH panel, imputing each held-out gene with SpatialScope and baseline methods, and scoring predictions by mean absolute error (MAE).

## When to use

Use when an image-based ST panel omits genes that are measured in a reference or validation set, and you need to quantify imputation accuracy (MAE) against measured expression for held-out marker genes.

## When NOT to use

- The target genes have no measured ground-truth expression in the ST data, so MAE against measured values cannot be computed
- The ST data is seq-based (e.g. Visium) with spot-level mixtures rather than single-cell image-based data where leave-one-out on a gene panel is defined
- The reference scRNA-seq does not cover the held-out genes, leaving no basis for the generative model to impute them

## Inputs

- MERFISH mouse1_slice180 data (254 genes) with measured expression of the seven cortical layer marker genes
- Paired droplet-based mouse MOp snRNA-seq reference (26,431 cells, 20 cell types), total-count normalized and log(1+x) transformed
- Trained SpatialScope score-based generative model checkpoint (e.g. model_5000.pt)
- Cell type labels from the SpatialScope identification step (Potts spatial smoothness prior, ν = 10)

## Outputs

- Imputed expression values for each left-out marker gene (Cux2, Otof, Rorb, Rspo1, Sulf2, Fezf2, Osr1) per method
- mae_imputation_merfish.csv with MAE values for SpatialScope and all seven baselines
- Percent MAE improvement of SpatialScope over each baseline for comparison with Fig. 6b, c

## How to apply

For each marker gene, hold it out of the MERFISH mouse1_slice180 panel (254 genes; 252 overlapping the snRNA-seq reference are used for training), impute it with the trained conditional score-based generative model via annealed Langevin dynamics posterior sampling conditioned on the observed 252-gene profile, and compute MAE against the actually measured expression. Run the seven baselines with paper-specified settings: Tangram (project_genes), gimVI (model.get_imputed_values, normalized=False), SpaGE (n_pv = Ngenes/2 since >50 genes are used), SpaOTsc (gamma_mapping matrix), novoSpaRc and stPlus (recommended defaults), and Seurat (TransferData). Compile all MAE values into mae_imputation_merfish.csv. Correct application is judged by reproducing the reported pattern: SpatialScope improves MAE by ~33.6%, 34.3%, 43.4%, and 53.6% over Tangram, gimVI, SpaOTsc, and novoSpaRc respectively, while performing comparably to SpaGE, stPlus, and Seurat (Fig. 6b, c).

## Related tools

- **SpatialScope** (Imputes held-out marker genes via annealed Langevin dynamics posterior sampling with the learned score function sθ(x, σl, μk)) — https://github.com/YangLabHKUST/SpatialScope
- **Tangram** (Baseline imputation via project_genes)
- **gimVI** (Baseline imputation via model.get_imputed_values with normalized=False) — https://github.com/scverse/scvi-tools
- **SpaGE** (Baseline imputation with n_pv = Ngenes/2) — https://github.com/tabdelaal/SpaGE
- **SpaOTsc** (Baseline imputation using the gamma_mapping matrix) — https://github.com/zcang/SpaOTsc
- **novoSpaRc** (Baseline imputation with recommended defaults) — https://github.com/rajewsky-lab/novosparc
- **stPlus** (Baseline imputation with recommended defaults) — https://github.com/xy-chen16/stPlus
- **Seurat** (Baseline imputation via TransferData)

## Examples

```
python imputation_merfish.py --st mouse1_slice180 --ref MOp_snRNAseq --markers Cux2,Otof,Rorb,Rspo1,Sulf2,Fezf2,Osr1 --ckpt model_5000.pt --out mae_imputation_merfish.csv
```

## Evaluation signals

- mae_imputation_merfish.csv contains MAE entries for SpatialScope and all seven baselines across the seven left-out markers
- SpatialScope MAE improvements of ~33.6%, 34.3%, 43.4%, and 53.6% over Tangram, gimVI, SpaOTsc, and novoSpaRc respectively are reproduced
- SpatialScope MAE is within comparable range of SpaGE, stPlus, and Seurat, matching Fig. 6b, c
- Each imputation used the correct baseline setting (e.g. SpaGE n_pv = Ngenes/2, gimVI normalized=False)
- Held-out markers were excluded from the training gene set before imputation

## Limitations

- MAE is computed against measured expression of genes deliberately left out of the panel; only a small set of seven cortical layer markers was used, so conclusions may not generalize to all genes
- Training the deep generative models is time-consuming (hours or days), which raises the cost of repeated leave-one-out evaluations
- Platform effects must be appropriately corrected between ST and scRNA-seq data or imputation results will not be satisfactory

## Evidence

- [other] SpatialScope significantly outperformed Tangram, gimVI, SpaOTsc, and novoSpaRc in imputing the seven left-out cortical layer marker genes: [evidence span withheld — anchor status: not_found]
- [methods] Impute unmeasured genes by running annealed Langevin dynamics posterior sampling: [evidence span withheld — anchor status: not_found]
- [methods] we used the slice, mouse1_slice180, from mouse1_sample4 to evaluate the imputation performance: "we used the slice, mouse1_slice180, from mouse1_sample4 to evaluate the imputation performance"
- [methods] we first performed total counts normalization and log(1+x) transformation: [evidence span withheld — anchor status: not_found]
- [readme] The SpaGE function takes as input ... the number of principal vectors (PVs): [evidence span withheld — anchor status: not_found]
