---
name: spearman-correlation-protein-rna-association-testing
description: Use when you have imputed surface protein abundances per cell (from cTP-net
  on denoised scRNA-seq) and need to test protein-space cell-state structure, e.g.,
  opposing CD56/CD16 gradients separating CD56bright and CD56dim NK subpopulations.
when_to_use_negative:
- Input expression is TPM/RPKM or otherwise not UMI-based count data — cTP-net can
  only apply to count-based expression input (UMI-based)
- Raw, undenoised scRNA-seq counts are used directly for imputation — SAVER-X denoising
  is strongly recommended and its absence retracts prediction performance by 0.02
  in correlation
- Cells are outside the hematopoietic lineage — the authors have not tried performing
  imputation of these immune-related markers on non-hematopoietic cells
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3361
tools:
- name: cTP-net
  role: Imputes the 24 surface protein relative abundances from normalized denoised
    scRNA-seq that are then subjected to the Spearman/F/Fisher/bootstrap association
    tests
  repo: https://github.com/zhouzilu/cTPnet
- name: SAVER-X
  role: Denoises the scRNA-seq counts with default parameters before imputation, a
    step without which correlation performance retracts by 0.02
- name: Seurat
  role: Provides the default LogNormalize transformation of denoised counts and the
    seurat3 pre/post-processing wrappers (preprocess_seurat3 / postprocess_seurat3)
- name: R
  role: Implementation environment for running cTPnet, the Seurat wrappers, and the
    correlation/statistical tests
  repo: https://github.com/zhouzilu/cTPnet
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:f657f0d47720d4b640e13acaaa202e715abb4f4708f5a1256a4bfd72fa731c54
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhouzilu/cTPnet
  tools:
  - R
  - SAVER-X
  - Seurat
  - cTP-net
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3361
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# spearman-correlation-protein-rna-association-testing

## Summary

Statistical workflow for testing whether cTP-net imputed surface protein levels (e.g., CD56, CD16, CD57 on HCA CBMC/BMMC NK cells) reveal biologically meaningful gradients that are absent from the transcriptome alone, using Spearman correlation plus F-, Fisher exact, and bootstrap tests.

## When to use

Use when you have imputed surface protein abundances per cell (from cTP-net on denoised scRNA-seq) and need to test protein-space cell-state structure, e.g., opposing CD56/CD16 gradients separating CD56bright and CD56dim NK subpopulations.

## When NOT to use

- Input expression is TPM/RPKM or otherwise not UMI-based count data — cTP-net can only apply to count-based expression input (UMI-based)
- Raw, undenoised scRNA-seq counts are used directly for imputation — SAVER-X denoising is strongly recommended and its absence retracts prediction performance by 0.02 in correlation
- Cells are outside the hematopoietic lineage — the authors have not tried performing imputation of these immune-related markers on non-hematopoietic cells

## Inputs

- HCA MantonCB1 and MantonBM1 scRNA-seq count matrices (UMI-based counts)
- SAVER-X denoised, Seurat LogNormalize-normalized expression matrices aligned to the 24-protein gene panel (gene_names_protein24)
- cTP-net pre-trained 24-protein model file (model_file_path)
- NK-cell subset annotations (e.g., from Seurat/UMAP clustering of the HCA data)

## Outputs

- Per-cell imputed relative abundances of 24 surface proteins (postprocess_seurat3 output)
- Spearman correlation between imputed CD56 and CD16 in NK cells (expected rho = -0.47)
- F-test p-values comparing CD56/CD16 gradient in imputed protein space vs raw RNA space (1.667e-15 vs 0.9377)
- Fisher exact test p-value on genes differential between reciprocal CD56/CD16 groups (1.07e-04) and bootstrap p-value for imputed CD57 CBMC vs BMMC (p < 1e-06)

## How to apply

First impute the 24-protein panel by denoising HCA MantonCB1/MantonBM1 counts with SAVER-X (default parameters; skipping denoising costs 0.02 in correlation), LogNormalize with Seurat defaults, align the feature space with preprocess_seurat3(seurat_data, dprotein) using gene_names_protein24, then run cTPnet(data, data_type, model_file_path, dprotein) with the pre-trained 24-protein model and recover relative abundances via postprocess_seurat3(seurat_data, y_pred); set options(stringsAsFactors = FALSE) beforehand to avoid a dimension error in the demo. Within the NK-cell subset, compute the Spearman correlation between imputed CD56 and CD16 and compare the protein-space gradient against raw RNA space with an F-test. Genes differential between the CD56-high/CD16-low and CD56-low/CD16-high imputed groups are tested with a Fisher exact test, and imputed CD57 differences between CBMC and BMMC are assessed with a bootstrap test. Accept the run when the statistics match the reported values: Spearman rho = -0.47, F-test p = 1.667e-15 (protein) vs 0.9377 (RNA), Fisher p = 1.07e-04, bootstrap p < 1e-06.

## Related tools

- **cTP-net** (Imputes the 24 surface protein relative abundances from normalized denoised scRNA-seq that are then subjected to the Spearman/F/Fisher/bootstrap association tests) — https://github.com/zhouzilu/cTPnet
- **SAVER-X** (Denoises the scRNA-seq counts with default parameters before imputation, a step without which correlation performance retracts by 0.02)
- **Seurat** (Provides the default LogNormalize transformation of denoised counts and the seurat3 pre/post-processing wrappers (preprocess_seurat3 / postprocess_seurat3))
- **R** (Implementation environment for running cTPnet, the Seurat wrappers, and the correlation/statistical tests) — https://github.com/zhouzilu/cTPnet

## Examples

```
options(stringsAsFactors = FALSE); y_pred <- cTPnet(data, data_type, model_file_path, gene_names_protein24); protein <- postprocess_seurat3(seurat_data, y_pred); cor.test(protein$CD56[nk], protein$CD16[nk], method = "spearman")
```

## Evaluation signals

- Spearman rho between imputed CD56 and CD16 in NK cells reproduces approximately -0.47, while the corresponding RNA abundances do not show the opposing gradient
- F-test comparing protein-space vs RNA-space gradient yields p = 1.667e-15 (protein) and 0.9377 (RNA), consistent with Fig. 3f-g
- Fisher exact test on genes differential between CD56-high/CD16-low and CD56-low/CD16-high imputed groups gives p = 1.07e-04 (Supplementary Figs. 8-10)
- Bootstrap test of imputed CD57 between CBMC and BMMC gives p < 1e-06
- Dimension-error sanity check: the demo run requires options(stringsAsFactors = FALSE) set beforehand to avoid 'Error in dim(X) <- c(n, length(X)/n)'

## Limitations

- cTP-net can only apply to count-based expression input (UMI-based); TPM/RPKM accuracy is untested
- Without SAVER-X denoising, cTP-net prediction performance retracts by 0.02 in correlation, more than any other parameter tweak
- Generalization beyond the hematopoietic lineage was not tested; Windows users may need numpy 1.19.3 to import the ctpnet Python module

## Evidence

- [methods] cTPnet invocation: "cTPnet(data, data_type, model_file_path, dprotein)"
- [discussion] denoising importance: "without SAVER-X denoising, the cTP-net prediction performance retracts by 0.02 in correlation, more significant than any other parameter tweaks"
- [discussion] feature-space alignment: "Genes that are in the training data but not in the testing data are filled with zeros."
- [discussion] input restriction: "cTP-net can only apply to count-based expression input (UMI-based)."
- [readme] runtime and preprocessing flag: "Given a dataset of 4000 cells, the running time of cTP-net should be less than 1 mins. No GPU required for running this algorithm."
- [readme] demo error fix: "Add `options(stringsAsFactors = FALSE)` before running the code."
