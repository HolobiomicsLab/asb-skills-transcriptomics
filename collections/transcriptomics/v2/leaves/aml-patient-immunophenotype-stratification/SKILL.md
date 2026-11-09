---
name: aml-patient-immunophenotype-stratification
description: Use when AML (or other hematopoietic) scRNA-seq count matrices need imputed
  surface protein levels to assign malignant cells to myeloid differentiation stages
  or to flag aberrant immunophenotype clusters such as hyper-CD38.
when_to_use_negative:
- Input cells are not of the hematopoietic lineage — the article states 'we have not
  tried to perform imputation of these immune-related markers on cells that are not
  of the hematopoietic lineage'
- Expression data are TPM or RPKM metrics rather than UMI-based counts — prediction
  accuracy is unknown and cTP-net 'can only apply to count-based expression input
  (UMI-based)'
- The input already contains measured (CITE-seq/REAP-seq) surface protein abundances
  — imputation is unnecessary
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: cTP-net
  role: Imputes the 24 surface protein abundances from the denoised, normalized transcriptome
    via the cTPnet() function with preprocess_seurat3/postprocess_seurat3 wrappers
  repo: https://github.com/zhouzilu/cTPnet
- name: SAVER-X
  role: Denoises the scRNA-seq counts (default parameters) before cTP-net imputation,
    improving prediction correlation by 0.02
- name: Seurat
  role: Provides LogNormalize transformation of denoised counts and UMAP construction
    of the imputed protein matrix
- name: R
  role: Execution environment for the cTPnet R package and the Seurat v3 vignette
    pipeline
  repo: https://github.com/zhouzilu/cTPnet
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:2c27d2e19fbcf6f09126d1484f1515a7c96eb5825860f60f2e7ff921628c5bbc
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# aml-patient-immunophenotype-stratification

## Summary

Stratifies AML patients by the differentiation stage of their malignant cell-of-origin using cTP-net-imputed surface protein abundances (24-immunophenotype panel) from scRNA-seq, reproducing the van Galen myeloid trajectory CD34+ -> CD38+CD123+ -> CD11c+/CD14+ and detecting aberrant clusters such as the hyper-CD38 immunophenotype in the RUNX1/RUNX1T1-fusion sample AML707B.

## When to use

Use when AML (or other hematopoietic) scRNA-seq count matrices need imputed surface protein levels to assign malignant cells to myeloid differentiation stages or to flag aberrant immunophenotype clusters such as hyper-CD38.

## When NOT to use

- Input cells are not of the hematopoietic lineage — the article states 'we have not tried to perform imputation of these immune-related markers on cells that are not of the hematopoietic lineage'
- Expression data are TPM or RPKM metrics rather than UMI-based counts — prediction accuracy is unknown and cTP-net 'can only apply to count-based expression input (UMI-based)'
- The input already contains measured (CITE-seq/REAP-seq) surface protein abundances — imputation is unnecessary

## Inputs

- van Galen AML scRNA-seq UMI count matrix in Seurat v3 format
- Pretrained cTP-net model file
- 24-protein panel gene/protein list (gene_names_protein24)
- shared_gene training feature set

## Outputs

- Imputed surface protein abundance matrix (24 immunophenotype markers per cell)
- Seurat UMAP embedding of imputed protein abundances for normal BM and Day-0 AML cells
- Differentiation-stage category assignment for each of the 12 AML patients
- Annotated hyper-CD38 immunophenotype cluster for sample AML707B

## How to apply

Load the van Galen AML Seurat v3 count matrix and denoise it with SAVER-X default parameters, since omitting denoising retracts cTP-net prediction correlation by 0.02. Transform the SAVER-X denoised counts with the Seurat default LogNormalize function, align genes against the shared_gene training feature set (filling missing training genes with zeros), and run cTPnet() with the pretrained model file and the 24-protein panel (gene_names_protein24) via preprocess_seurat3/postprocess_seurat3. Build a Seurat UMAP of the imputed protein abundances restricted to normal bone marrow cells and Day-0 AML samples, verify the ordering CD34+ progenitors upstream of CD38+CD123+ and CD11c+/CD14+ populations, assign each of the 12 AML patients to one of three differentiation-stage categories, and annotate the distinct CD38-high cluster in AML707B on the UMAP.

## Related tools

- **cTP-net** (Imputes the 24 surface protein abundances from the denoised, normalized transcriptome via the cTPnet() function with preprocess_seurat3/postprocess_seurat3 wrappers) — https://github.com/zhouzilu/cTPnet
- **SAVER-X** (Denoises the scRNA-seq counts (default parameters) before cTP-net imputation, improving prediction correlation by 0.02)
- **Seurat** (Provides LogNormalize transformation of denoised counts and UMAP construction of the imputed protein matrix)
- **R** (Execution environment for the cTPnet R package and the Seurat v3 vignette pipeline) — https://github.com/zhouzilu/cTPnet

## Examples

```
X <- preprocess_seurat3(seurat_data, gene_names_protein24); y_pred <- cTPnet(X, 'seurat3', model_file_path, gene_names_protein24); seurat_data <- postprocess_seurat3(seurat_data, y_pred)
```

## Evaluation signals

- UMAP reproduces the myeloid trajectory ordering: CD34+ progenitors upstream of CD38+CD123+ and CD11c+/CD14+ populations (Fig. 4)
- The 12 AML patients fall into the three reported differentiation-stage categories, consistent with the original van Galen study
- AML707B shows a distinct CD38-high cluster annotated on the UMAP
- Gene alignment succeeds without dropping shared features: genes present in training but absent in the input are filled with zeros
- cTP-net runtime on ~4000 cells is under ~1 minute on a normal computer without GPU, indicating the pipeline ran correctly

## Limitations

- Omitting SAVER-X denoising retracts cTP-net prediction correlation by 0.02, more than any other parameter tweak
- cTP-net only applies to UMI-based count expression input; accuracy for TPM/RPKM metrics is unknown
- Generalization to cell types outside the hematopoietic lineage was not tested
- Windows installations may fail with a ctpnet module import error unless numpy 1.19.3 is used; the authors do not recommend Windows

## Evidence

- [discussion] SAVER-X denoising is required: "without SAVER-X denoising, the cTP-net prediction performance retracts by 0.02 in correlation, more significant than any other parameter tweaks"
- [discussion] cTP-net imputes immunophenotypes and places AML cells on the myeloid trajectory: "it is able to impute immunophenotypes for malignant cells from AML, and that these immunophenotypes allow placement of the cells along the myeloid differentiation trajectory"
- [methods] Missing training genes are zero-filled during feature alignment: "Genes that are in the training data but not in the testing data are filled with zeros."
- [methods] Package includes SAVERX denoising plus cTP-net imputation: "This cTPnet package includes two analysis tools: (1) **SAVERX**, we stronly recommend denoise the scRNA-seq data before impute"
- [readme] Runtime and no GPU needed for the imputation step: "Given a dataset of 4000 cells, the running time of cTP-net should be less than 1 mins. No GPU required for running this algorithm."
