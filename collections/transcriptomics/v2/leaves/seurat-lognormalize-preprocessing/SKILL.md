---
name: seurat-lognormalize-preprocessing
description: Use when preparing UMI-based scRNA-seq count matrices (e.g., HCA MantonCB1/MantonBM1
  or CITE-seq PBMC) as input to cTP-net protein imputation, after SAVER-X denoising
  and before running the pre-trained 24-protein model.
when_to_use_negative:
- Input expression data is not UMI-based count data (e.g., TPM or RPKM) — the article
  notes such prediction accuracy is unknown and that 'cTP-net can only apply to count-based
  expression input (UMI-based).'
- Raw (undenoised) counts fed directly to imputation — SAVER-X denoising is strongly
  recommended, and skipping it degrades correlation by 0.02.
- Cells outside the hematopoietic lineage — the article states imputation of these
  immune-related markers was not tried on non-hematopoietic cells.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Seurat
  role: Provides the default LogNormalize transformation applied to SAVER-X-denoised
    counts before cTP-net imputation
- name: SAVER-X
  role: Denoises scRNA-seq counts with default parameters prior to LogNormalize; skipping
    it reduces cTP-net correlation by 0.02
- name: cTP-net
  role: Package whose preprocess_seurat3/postprocess_seurat3 functions align feature
    spaces and recover imputed 24-protein abundances; available in R and Python
  repo: https://github.com/zhouzilu/cTPnet
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:4d652ff778b9fc582df50d9f20e34634d9a9313d0f2ab667a800e90a117c4018
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhouzilu/cTPnet
  tools:
  - Python
  - R
  - SAVER-X
  - Seurat
  - cTP-net
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# seurat-lognormalize-preprocessing

## Summary

Seurat default LogNormalize transformation of SAVER-X-denoised scRNA-seq counts to prepare the normalized expression matrix that serves as input to cTP-net surface protein imputation, with feature-space alignment to the pre-trained protein panel via the cTPnet package's preprocess/postprocess functions.

## When to use

Use when preparing UMI-based scRNA-seq count matrices (e.g., HCA MantonCB1/MantonBM1 or CITE-seq PBMC) as input to cTP-net protein imputation, after SAVER-X denoising and before running the pre-trained 24-protein model.

## When NOT to use

- Input expression data is not UMI-based count data (e.g., TPM or RPKM) — the article notes such prediction accuracy is unknown and that 'cTP-net can only apply to count-based expression input (UMI-based).'
- Raw (undenoised) counts fed directly to imputation — SAVER-X denoising is strongly recommended, and skipping it degrades correlation by 0.02.
- Cells outside the hematopoietic lineage — the article states imputation of these immune-related markers was not tried on non-hematopoietic cells.

## Inputs

- SAVER-X denoised scRNA-seq count matrix (UMI-based counts, e.g., HCA MantonCB1 / MantonBM1 or CITE-seq PBMC)
- Seurat v2 or v3 object from the cTPnet package vignettes
- 24-protein gene panel (gene_names_protein24, or gene_names_protein12 for the 12-protein panel)

## Outputs

- LogNormalize-normalized denoised expression matrix aligned to the cTP-net training feature space (input X for cTPnet())
- Imputed relative surface protein abundances for 24 immunophenotype markers per cell after postprocess_seurat3

## How to apply

First denoise the scRNA-seq counts with SAVER-X using default parameters, since the article states that without SAVER-X denoising cTP-net prediction performance retracts by 0.02 in correlation. Then transform the denoised counts with the Seurat default LogNormalize function to obtain the normalized denoised count matrix X. Align the feature space to the cTP-net training data by calling preprocess_seurat3(seurat_data, dprotein) with the 24-protein gene panel (gene_names_protein24) — Seurat v2 workflows use preprocess_seurat2 instead — and fill genes present in training but absent from the testing data with zeros. After cTPnet() imputation, apply the matching postprocess_seurat3(seurat_data, y_pred) to recover imputed relative surface protein abundances. Set options(stringsAsFactors = FALSE) before running to avoid the demo dimension error. On a normal computer this step supports imputing 4000 cells in under 1 minute with no GPU.

## Related tools

- **Seurat** (Provides the default LogNormalize transformation applied to SAVER-X-denoised counts before cTP-net imputation)
- **SAVER-X** (Denoises scRNA-seq counts with default parameters prior to LogNormalize; skipping it reduces cTP-net correlation by 0.02)
- **cTP-net** (Package whose preprocess_seurat3/postprocess_seurat3 functions align feature spaces and recover imputed 24-protein abundances; available in R and Python) — https://github.com/zhouzilu/cTPnet

## Examples

```
options(stringsAsFactors = FALSE); seurat_data <- preprocess_seurat3(seurat_data, gene_names_protein24)
```

## Evaluation signals

- The normalized matrix X retains the cell dimension of the input (~4000 cells) and the gene dimension matched to the 24-protein cTP-net panel, with missing genes filled by zeros.
- Downstream imputation of a 4000-cell dataset completes in < 1 minute without GPU, as stated in the README.
- On HCA NK cells, the imputed CD56/CD16 Spearman correlation reaches rho = -0.47 and the protein-space F-test p = 1.667e-15 (vs 0.9377 in RNA space), confirming the LogNormalize + cTP-net pipeline reproduces Fig. 3f-g.
- No demo error 'Error in dim(X) <- c(n, length(X)/n)' — indicates options(stringsAsFactors = FALSE) was set and preprocessing ran correctly.
- In the CITE-seq PBMC 90/10 holdout benchmark, per-protein correlations of cTP-net predictions exceed both raw and SAVER-X-denoised RNA baselines (Fig. 1b).

## Limitations

- cTP-net only applies to UMI-based count expression input; accuracy for TPM/RPKM-normalized data is untested.
- Skipping SAVER-X denoising before LogNormalize retracts prediction performance by 0.02 in correlation, more than any other parameter tweak.
- Genes in the training data but not in the testing data must be filled with zeros, which can degrade predictions when the feature gap is large.
- On Windows, the Python ctpnet module may fail to import; numpy 1.19.3 is required, and running on Windows is not recommended.

## Evidence

- [discussion] Denoised counts (Λ) were further transformed with Seurat default LogNormalize function: "Denoised counts (Λ) were further transformed with Seurat default LogNormalize function"
- [readme] SAVERX strongly recommended before imputation: "This cTPnet package includes two analysis tools: (1) **SAVERX**, we stronly recommend denoise the scRNA-seq data before impute the surface protein abundance, and (2) **cTP-net**, which impute"
- [discussion] UMI count input restriction: "cTP-net can only apply to count-based expression input (UMI-based)."
- [readme] Runtime expectation: "Given a dataset of 4000 cells, the running time of cTP-net should be less than 1 mins. No GPU required for running this algorithm."
- [discussion] SAVER-X denoising impact on performance: "without SAVER-X denoising, the cTP-net prediction performance retracts by 0.02 in correlation, more significant than any other parameter tweaks"
