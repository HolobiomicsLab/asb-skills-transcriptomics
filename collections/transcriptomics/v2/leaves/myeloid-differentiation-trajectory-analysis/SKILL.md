---
name: myeloid-differentiation-trajectory-analysis
description: Use when primary AML scRNA-seq samples need a cell-of-origin differentiation-stage
  call from imputed surface protein levels rather than transcriptome clustering alone.
when_to_use_negative:
- Input expression is not UMI-based count data — cTP-net can only apply to count-based
  expression input (UMI-based)
- Cells are outside the hematopoietic lineage — the article notes imputation of these
  immune-related markers was not tested on non-hematopoietic cells
- Goal is protein prediction from TPM/RPKM expression metrics — prediction accuracy
  is unknown as no such CITE-seq data were available for testing
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: cTP-net
  role: Imputes the 24-protein surface immunophenotype from the denoised scRNA-seq
    input; also wraps SAVERX denoising in its package
  repo: https://github.com/zhouzilu/cTPnet
- name: SAVER-X
  role: Denoises the raw AML scRNA-seq counts (default parameters) before imputation,
    a step whose omission reduces correlation by 0.02
  repo: https://github.com/zhouzilu/cTPnet
- name: Seurat
  role: LogNormalize transformation of denoised counts as cTP-net input, and UMAP
    construction/annotation of the imputed protein matrix
  repo: https://github.com/zhouzilu/cTPnet
- name: R
  role: Implementation environment for the Seurat v3 cTP-net workflow (preprocess_seurat3
    / cTPnet / postprocess_seurat3)
  repo: https://github.com/zhouzilu/cTPnet
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:13c56b889f1761f6c17d499ef86ecf2e9cfeed56082e60aaff0a15468d6ea126
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhouzilu/cTPnet
  tools:
  - GraphDDP
  - GraphEmbed
  - R
  - SAVER-X
  - Seurat
  - cTP-net
  - pygraphviz
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# myeloid-differentiation-trajectory-analysis

## Summary

Characterizes the myeloid differentiation stage of malignant AML cells by running cTP-net on SAVER-X-denoised, LogNormalize-transformed scRNA-seq data to impute a 24-protein immunophenotype panel, then ordering cells along the CD34+ -> CD38+CD123+ -> CD11c+/CD14+ trajectory and assigning each patient a differentiation-stage category.

## When to use

Use when primary AML scRNA-seq samples need a cell-of-origin differentiation-stage call from imputed surface protein levels rather than transcriptome clustering alone.

## When NOT to use

- Input expression is not UMI-based count data — cTP-net can only apply to count-based expression input (UMI-based)
- Cells are outside the hematopoietic lineage — the article notes imputation of these immune-related markers was not tested on non-hematopoietic cells
- Goal is protein prediction from TPM/RPKM expression metrics — prediction accuracy is unknown as no such CITE-seq data were available for testing

## Inputs

- van Galen AML scRNA-seq UMI count matrix in Seurat v3 (R) format
- SAVER-X denoised, LogNormalize-transformed expression matrix (cTP-net input)
- Pretrained cTP-net model file for the 24-protein immunophenotype panel (gene_names_protein24)
- Patient/sample metadata (12 AML patients, Day-0 samples) distinguishing malignant vs normal bone marrow cells

## Outputs

- Imputed 24-protein surface abundance matrix per malignant AML cell
- UMAP embedding of imputed protein levels (normal BM + Day-0 AML) showing the myeloid differentiation trajectory
- Differentiation-stage assignment for each of the 12 AML patients (three categories)
- Annotation of aberrant clusters, e.g. the CD38-high immunophenotype in sample AML707B

## How to apply

Load the van Galen AML count matrix in R (Seurat v3 format) and denoise it with SAVER-X using default parameters, since omitting this step reduces cTP-net prediction correlation by 0.02 — more than any other parameter tweak. Transform the denoised counts with the Seurat default LogNormalize function, align genes against the shared_gene training feature set (filling missing training genes with zeros), and run cTPnet() with the pretrained model file and the 24-protein panel (gene_names_protein24) via preprocess_seurat3/postprocess_seurat3. Build a Seurat UMAP of the imputed protein abundances restricted to normal bone marrow cells and Day-0 AML samples, order cells by imputed CD34, CD38/CD123, and CD11c/CD14 levels to confirm the myeloid trajectory, and assign each of the 12 AML patients to one of three differentiation-stage categories. Scan patient-specific imputed profiles for aberrant clusters such as the hyper-CD38 immunophenotype in the RUNX1/RUNX1T1-fusion sample AML707B and annotate it on the UMAP.

## Related tools

- **cTP-net** (Imputes the 24-protein surface immunophenotype from the denoised scRNA-seq input; also wraps SAVERX denoising in its package) — https://github.com/zhouzilu/cTPnet
- **SAVER-X** (Denoises the raw AML scRNA-seq counts (default parameters) before imputation, a step whose omission reduces correlation by 0.02) — https://github.com/zhouzilu/cTPnet
- **Seurat** (LogNormalize transformation of denoised counts as cTP-net input, and UMAP construction/annotation of the imputed protein matrix) — https://github.com/zhouzilu/cTPnet
- **R** (Implementation environment for the Seurat v3 cTP-net workflow (preprocess_seurat3 / cTPnet / postprocess_seurat3)) — https://github.com/zhouzilu/cTPnet

## Examples

```
R: y_pred <- postprocess_seurat3(seurat_data, cTPnet(preprocess_seurat3(seurat_data, gene_names_protein24), data_type, model_file_path, gene_names_protein24))
```

## Evaluation signals

- UMAP reproduces the myeloid trajectory ordering: CD34+ progenitors upstream of CD38+CD123+ and CD11c+/CD14+ populations
- The 12 AML patients fall into the three reported differentiation-stage categories, consistent with the original study
- Sample AML707B (RUNX1/RUNX1T1 fusion) shows a distinct CD38-high cluster on the imputed-protein UMAP
- Running the pipeline with SAVER-X denoising yields ~0.02 higher imputed-protein correlation than without denoising
- All training genes absent from the query matrix are filled with zeros so the feature space matches shared_gene

## Limitations

- Omitting SAVER-X denoising retracts prediction performance by 0.02 in correlation, more than any other parameter tweak
- cTP-net can only apply to count-based expression input (UMI-based)
- Imputation of these immune-related markers has not been tried on cells outside the hematopoietic lineage
- A model extending prediction beyond the current protein panel (toward 100 surface proteins) was still in training at publication

## Evidence

- [discussion] it is able to impute immunophenotypes for malignant cells from AML, and that these immunophenotypes allow placement of the cells along the myeloid differentiation trajectory: "it is able to impute immunophenotypes for malignant cells from AML, and that these immunophenotypes allow placement of the cells along the myeloid differentiation trajectory"
- [discussion] without SAVER-X denoising, the cTP-net prediction performance retracts by 0.02 in correlation: "without SAVER-X denoising, the cTP-net prediction performance retracts by 0.02 in correlation, more significant than any other parameter tweaks"
- [discussion] Denoised counts (Λ) were further transformed with Seurat default LogNormalize function: "Denoised counts (Λ) were further transformed with Seurat default LogNormalize function"
- [discussion] Genes that are in the training data but not in the testing data are filled with zeros.: "Genes that are in the training data but not in the testing data are filled with zeros."
- [readme] This cTPnet package includes two analysis tools: (1) SAVERX, we stronly recommend denoise the scRNA-seq data before impute the surface protein abundance, and (2) cTP-net, which impute: "This cTPnet package includes two analysis tools: (1) **SAVERX**, we stronly recommend denoise the scRNA-seq data before impute the surface protein abundance, and (2) **cTP-net**, which impute"
- [methods] cTPnet(data, data_type, model_file_path, dprotein): "- `cTPnet(data, data_type, model_file_path, dprotein)`"
