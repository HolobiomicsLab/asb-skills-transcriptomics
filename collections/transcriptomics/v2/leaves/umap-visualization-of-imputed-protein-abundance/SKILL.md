---
name: umap-visualization-of-imputed-protein-abundance
description: Use when scRNA-seq cells (e.g. van Galen AML or normal bone marrow) have
  had surface proteins imputed by cTP-net and you need a 2D embedding to order cells
  along a differentiation trajectory or annotate immunophenotype clusters.
when_to_use_negative:
- Expression input is not UMI-based counts (e.g. TPM or RPKM metrics) — prediction
  accuracy is unknown and cTP-net only applies to count-based UMI input
- Cells are outside the hematopoietic lineage — generalization of the immune-marker
  imputation to non-hematopoietic cells was not tested
- A CITE-seq/REAP-seq run already provides directly measured protein abundances, making
  imputation unnecessary
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0092
tools:
- name: cTP-net
  role: Imputes the 24 surface protein abundances that form the features of the UMAP
    embedding
  repo: https://github.com/zhouzilu/cTPnet
- name: SAVER-X
  role: Denoises the scRNA-seq counts before imputation; recommended default-parameter
    denoising step preceding UMAP feature generation
- name: Seurat
  role: LogNormalize transformation of denoised counts and default pipeline with UMAP
    generation on the imputed protein matrix
- name: R
  role: Environment in which the cTPnet package (preprocess_seurat3/cTPnet/postprocess_seurat3)
    and Seurat UMAP are run
  repo: https://github.com/zhouzilu/cTPnet
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-14391-0
    title: Surface protein imputation from single cell transcriptomes by deep neural
      networks
  source_content_hash: sha256:d4751b11c41d0b3a6a1ba16a08070d9aa72761efeb3e12c3a90fafca6a660dd3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zhouzilu/cTPnet
  tools:
  - R
  - SAVER-X
  - Seurat
  - cTP-net
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# UMAP visualization of imputed protein abundance

## Summary

Builds a Seurat UMAP embedding from cTP-net-imputed surface protein levels (e.g. the 24-protein immunophenotype panel) so malignant AML cells can be placed along the myeloid differentiation trajectory (CD34+ -> CD38+CD123+ -> CD11c+/CD14+) and aberrant clusters such as the hyper-CD38 immunophenotype in AML707B can be annotated.

## When to use

Use when scRNA-seq cells (e.g. van Galen AML or normal bone marrow) have had surface proteins imputed by cTP-net and you need a 2D embedding to order cells along a differentiation trajectory or annotate immunophenotype clusters.

## When NOT to use

- Expression input is not UMI-based counts (e.g. TPM or RPKM metrics) — prediction accuracy is unknown and cTP-net only applies to count-based UMI input
- Cells are outside the hematopoietic lineage — generalization of the immune-marker imputation to non-hematopoietic cells was not tested
- A CITE-seq/REAP-seq run already provides directly measured protein abundances, making imputation unnecessary

## Inputs

- SAVER-X denoised, LogNormalize-transformed scRNA-seq UMI count matrix (Seurat v3 object)
- Pretrained cTP-net model file and 24-protein panel gene list (gene_names_protein24)
- shared_gene training feature set
- Cell annotations distinguishing normal bone marrow cells from Day-0 AML samples

## Outputs

- Seurat UMAP embedding of cTP-net-imputed surface protein abundances
- Myeloid differentiation trajectory ordering (CD34+ -> CD38+CD123+ -> CD11c+/CD14+)
- Differentiation-stage assignment for each of the 12 AML patients (three stage categories)
- Annotated hyper-CD38 cluster for sample AML707B

## How to apply

First run the cTP-net imputation pipeline: denoise the UMI count matrix with SAVER-X (default parameters; skipping this retracts prediction correlation by 0.02), transform with Seurat default LogNormalize, align genes against the shared_gene feature set (filling missing training genes with zeros), and call cTPnet() with the pretrained model and the 24-protein panel (gene_names_protein24) using preprocess_seurat3/postprocess_seurat3. Restrict the embedding to normal bone marrow cells and Day-0 AML samples, then apply the default Seurat pipeline and RunUMAP on the imputed protein abundance matrix. Verify the myeloid trajectory ordering CD34+ progenitors upstream of CD38+CD123+ and CD11c+/CD14+ populations, assign each of the 12 AML patients to one of the three reported differentiation-stage categories, and annotate the distinct CD38-high cluster in the RUNX1/RUNX1T1-fusion sample AML707B.

## Related tools

- **cTP-net** (Imputes the 24 surface protein abundances that form the features of the UMAP embedding) — https://github.com/zhouzilu/cTPnet
- **SAVER-X** (Denoises the scRNA-seq counts before imputation; recommended default-parameter denoising step preceding UMAP feature generation)
- **Seurat** (LogNormalize transformation of denoised counts and default pipeline with UMAP generation on the imputed protein matrix)
- **R** (Environment in which the cTPnet package (preprocess_seurat3/cTPnet/postprocess_seurat3) and Seurat UMAP are run) — https://github.com/zhouzilu/cTPnet

## Examples

```
y_pred <- cTPnet(data = X, data_type = 'seurat3', model_file_path = model, dprotein = gene_names_protein24); seu <- postprocess_seurat3(seu, y_pred); seu <- RunUMAP(seu, features = gene_names_protein24)
```

## Evaluation signals

- UMAP reproduces the Fig. 4 myeloid trajectory ordering: CD34+ progenitors upstream of CD38+CD123+ and CD11c+/CD14+ populations
- The 12 AML patients fall into the three reported differentiation-stage categories consistent with the original study
- Sample AML707B shows a distinct, spatially separated CD38-high cluster on the UMAP
- Imputed protein values are on the transformed relative abundance scale produced by postprocess_seurat3, not raw barcode counts
- Missing training genes were filled with zeros so the input feature space matches the pretrained shared_gene set

## Limitations

- cTP-net can only apply to count-based (UMI) expression input; TPM/RPKM accuracy is unknown
- Omitting SAVER-X denoising reduces prediction correlation by 0.02, more than any other parameter tweak
- Generalization to cell types outside the hematopoietic lineage was not tested
- A 100-protein extension model is still in training; the released pretrained model covers the 24-protein panel

## Evidence

- [results] Seurat UMAP of imputed surface protein abundances restricted to normal bone marrow and Day-0 AML: "We applied default pipeline of Seurat and generated UMAP plot for both data sets"
- [discussion] cTP-net imputed immunophenotypes place AML cells on the myeloid differentiation trajectory: "it is able to impute immunophenotypes for malignant cells from AML, and that these immunophenotypes allow placement of the cells along the myeloid differentiation trajectory"
- [discussion] Denoised counts are transformed with Seurat LogNormalize before use as cTP-net input: "Denoised counts (Λ) were further transformed with Seurat default LogNormalize function"
- [methods] Feature-space alignment by filling missing training genes with zeros: "Genes that are in the training data but not in the testing data are filled with zeros."
- [readme] Running-time practicality of the cTP-net step feeding the UMAP: "Given a dataset of 4000 cells, the running time of cTP-net should be less than 1 mins. No GPU required for running this algorithm."
