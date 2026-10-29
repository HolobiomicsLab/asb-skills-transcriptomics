---
name: surface-protein-imputation-from-scrna-seq
description: Use when the input is a UMI-based scRNA-seq count matrix (e.g., Seurat
  v3 object from HCA CBMC/BMMC or AML samples) and cell-surface protein levels such
  as CD56, CD16, CD57, CD34, CD38 are needed but CITE-seq/REAP-seq was not performed.
when_to_use_negative:
- Input expression data is not UMI-based count data (cTP-net can only apply to count-based
  expression input, UMI-based); TPM/RPKM CITE-seq accuracy is untested
- Target cells are outside the hematopoietic lineage — the article states imputation
  of immune markers on non-hematopoietic cells was not tried
- The panel of 24 immunophenotype markers is insufficient and other surface proteins
  are required — the 100-protein model was still in training and not released
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: cTP-net
  role: Deep neural network transfer-learning model that imputes surface protein relative
    abundances from the normalized denoised expression matrix via cTPnet()
  repo: https://github.com/zhouzilu/cTPnet
- name: SAVER-X
  role: Denoises scRNA-seq counts with default parameters before imputation (strongly
    recommended in the cTPnet package pipeline)
- name: Seurat
  role: Provides LogNormalize transformation of denoised counts and preprocess_seurat3/postprocess_seurat3
    object handling, plus UMAP for downstream visualization
- name: R
  role: Runtime environment for the cTPnet package (also available in Python); requires
    numpy 1.19.3 on some platforms
  repo: https://github.com/zhouzilu/cTPnet
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:b6ccfe6b49dfa384373aa15faf4b20f4ea727446ac27d3e4681a145b55eea734
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhouzilu/cTPnet
  tools:
  - R
  - SAVER-X
  - Seurat
  - cTP-net
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# surface-protein-imputation-from-scrna-seq

## Summary

Impute the relative abundances of 24 immunophenotype surface protein markers from scRNA-seq UMI count matrices using cTP-net, a transfer-learning deep neural network pretrained on paired CITE-seq/REAP-seq data. The skill enables surface-phenotype characterization (e.g., NK CD56/CD16 gradients, AML differentiation stages) when only transcriptome measurements are available.

## When to use

Use when the input is a UMI-based scRNA-seq count matrix (e.g., Seurat v3 object from HCA CBMC/BMMC or AML samples) and cell-surface protein levels such as CD56, CD16, CD57, CD34, CD38 are needed but CITE-seq/REAP-seq was not performed.

## When NOT to use

- Input expression data is not UMI-based count data (cTP-net can only apply to count-based expression input, UMI-based); TPM/RPKM CITE-seq accuracy is untested
- Target cells are outside the hematopoietic lineage — the article states imputation of immune markers on non-hematopoietic cells was not tried
- The panel of 24 immunophenotype markers is insufficient and other surface proteins are required — the 100-protein model was still in training and not released

## Inputs

- UMI-based scRNA-seq count matrix in Seurat v3 format (e.g., HCA MantonCB1/MantonBM1, van Galen AML)
- Pre-trained 24-protein cTP-net model file
- 24-protein gene panel definition (gene_names_protein24) and shared_gene training feature set

## Outputs

- Matrix of imputed relative abundances for 24 surface proteins, one value per cell
- Seurat object with imputed protein abundances attached (via postprocess_seurat3)
- Downstream UMAP embedding of imputed protein space and per-cell immunophenotype annotations

## How to apply

First denoise the scRNA-seq counts with SAVER-X using default parameters, since omitting denoising retracts cTP-net prediction correlation by 0.02 — more than any other parameter tweak. Transform the denoised counts (Λ) with the Seurat default LogNormalize function; cTP-net only accepts UMI-based count input. Align the feature space against the shared_gene training set (fill missing training genes with zeros) and run preprocess_seurat3(seurat_data, dprotein) with the 24-protein panel (gene_names_protein24), then call cTPnet(data, data_type, model_file_path, dprotein) with the pretrained model — for ~4000 cells this takes under 1 minute with no GPU; set options(stringsAsFactors = FALSE) before running to avoid the dim(X) demo error. Recover imputed abundances with postprocess_seurat3(seurat_data, y_pred). Downstream, imputed proteins support cell-type assignment, NK CD56/CD16 gradient statistics, and myeloid trajectory (CD34+ -> CD38+CD123+ -> CD11c+/CD14+) placement on a Seurat UMAP.

## Related tools

- **cTP-net** (Deep neural network transfer-learning model that imputes surface protein relative abundances from the normalized denoised expression matrix via cTPnet()) — https://github.com/zhouzilu/cTPnet
- **SAVER-X** (Denoises scRNA-seq counts with default parameters before imputation (strongly recommended in the cTPnet package pipeline))
- **Seurat** (Provides LogNormalize transformation of denoised counts and preprocess_seurat3/postprocess_seurat3 object handling, plus UMAP for downstream visualization)
- **R** (Runtime environment for the cTPnet package (also available in Python); requires numpy 1.19.3 on some platforms) — https://github.com/zhouzilu/cTPnet

## Examples

```
options(stringsAsFactors=FALSE); seurat_data <- preprocess_seurat3(seurat_data, dprotein=gene_names_protein24); y_pred <- cTPnet(data, data_type, model_file_path, dprotein=gene_names_protein24); seurat_data <- postprocess_seurat3(seurat_data, y_pred)
```

## Evaluation signals

- Reproduced NK-cell statistics match reported values: Spearman rho = -0.47 for imputed CD56 vs CD16, F-test p = 1.667e-15 (protein) vs 0.9377 (RNA)
- Fisher exact test on differential genes between CD56-high/CD16-low and CD56-low/CD16-high imputed groups gives p = 1.07e-04; bootstrap test for imputed CD57 CBMC-vs-BMMC gives p < 1e-06
- AML UMAP reproduces the Fig. 4 myeloid trajectory ordering (CD34+ progenitors upstream of CD38+CD123+ and CD11c+/CD14+) and AML707B shows a distinct CD38-high cluster
- Benchmark correlation of cTP-net predictions vs CITE-seq measured abundances exceeds raw RNA and SAVER-X-denoised RNA baselines (Fig. 1b scatter/correlation)
- No dimension error: run completes without 'dims [product 1992] do not match the length of object[2000]' error, confirming options(stringsAsFactors = FALSE) was set

## Limitations

- cTP-net only applicable to UMI-based count expression input; prediction accuracy for TPM/RPKM CITE-seq data is unknown because such data was not available for testing
- Generalization to cell types outside the hematopoietic lineage was not tested
- Omitting SAVER-X denoising reduces prediction correlation by 0.02; comparisons in benchmarks are limited to the same cells as the validation set to account for subset variation
- Windows users may hit 'ModuleNotFoundError: No module named ctpnet' — requires numpy 1.19.3, and Windows is not recommended

## Evidence

- [abstract] single cell Transcriptome to Protein prediction with deep neural network (cTP-net), to impute surface protein abundances from scRNA-seq data: "single cell Transcriptome to Protein prediction with deep neural network (cTP-net), to impute surface protein abundances from scRNA-seq data"
- [discussion] without SAVER-X denoising, the cTP-net prediction performance retracts by 0.02 in correlation, more significant than any other parameter tweaks: "without SAVER-X denoising, the cTP-net prediction performance retracts by 0.02 in correlation, more significant than any other parameter tweaks"
- [discussion] Genes that are in the training data but not in the testing data are filled with zeros.: "Genes that are in the training data but not in the testing data are filled with zeros."
- [discussion] cTP-net can only apply to count-based expression input (UMI-based).: "cTP-net can only apply to count-based expression input (UMI-based)."
- [readme] Given a dataset of 4000 cells, the running time of cTP-net should be less than 1 mins. No GPU required for running this algorithm.: "Given a dataset of 4000 cells, the running time of cTP-net should be less than 1 mins. No GPU required for running this algorithm."
- [methods] This cTPnet package includes two analysis tools: (1) SAVERX, we stronly recommend denoise the scRNA-seq data before impute the surface protein abundance, and (2) cTP-net, which impute: "This cTPnet package includes two analysis tools: (1) **SAVERX**, we stronly recommend denoise the scRNA-seq data before impute the surface protein abundance, and (2) **cTP-net**, which impute"
- [readme] Add options(stringsAsFactors = FALSE) before running the code.: "Add `options(stringsAsFactors = FALSE)` before running the code."
