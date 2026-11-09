---
name: single-cell-count-matrix-quality-control
description: Use when starting from a raw scRNA-seq count matrix downloaded from GEO
  (e.g., GSE113197 breast epithelial cells) that must be filtered for low-quality
  cells and genes and normalized before clustering or marker-based cell annotation.
when_to_use_negative:
- The input is already a normalized expression matrix or an annotated cell-by-cluster
  table rather than raw counts
- The goal is PPIN-based pluripotency/differentiation scoring itself (use the ORIGINS
  activity() computation on a normalized matrix instead)
- The data are bulk RNA-seq or non-single-cell expression data, where per-cell QC
  filtering is not applicable
edam_operation: http://edamontology.org/operation_3221
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: R
  role: Analysis environment in which the count-matrix QC filtering and normalization
    are performed as part of the ORIGINS workflow
  repo: https://github.com/danielasenraoka/ORIGINS
- name: dplyr
  role: Imported utility package used for the data-manipulation steps of QC and normalization
- name: magrittr
  role: Imported utility package providing the pipe-style syntax used in the R normalization
    workflow
- name: ORIGINS
  role: Downstream consumer of the normalized expression matrix; its activity() function
    expects a matrix or data frame where each row is a gene and each column is a cell
  repo: https://github.com/danielasenraoka/ORIGINS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.mex.2022.101778
    title: 'ORIGINS: A protein network-based approach to quantify cell pluripotency
      from scRNA-seq data'
  source_content_hash: sha256:705dbd44d55cee62134b3fad48fe9f74948617e1e953fb79f1de664f38b4a24c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/danielasenraoka/ORIGINS
  tools:
  - ORIGINS
  - R
  - dplyr
  - magrittr
  edam_operation: http://edamontology.org/operation_3221
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# Single-cell count matrix quality control

## Summary

Quality-control and normalization of a raw single-cell RNA-seq count matrix (e.g., the Ind4 breast epithelial sample of GSE113197) in R using dplyr/magrittr, producing a clean normalized expression matrix that feeds downstream dimensionality reduction, clustering, and cell-type annotation in the ORIGINS/Seurat-style published breast dataset workflow.

## When to use

Use when starting from a raw scRNA-seq count matrix downloaded from GEO (e.g., GSE113197 breast epithelial cells) that must be filtered for low-quality cells and genes and normalized before clustering or marker-based cell annotation.

## When NOT to use

- The input is already a normalized expression matrix or an annotated cell-by-cluster table rather than raw counts
- The goal is PPIN-based pluripotency/differentiation scoring itself (use the ORIGINS activity() computation on a normalized matrix instead)
- The data are bulk RNA-seq or non-single-cell expression data, where per-cell QC filtering is not applicable

## Inputs

- Raw single-cell count matrix for the Ind4 breast epithelial sample from GEO accession GSE113197 (rows = genes, columns = cells)
- R environment with dplyr and magrittr installed

## Outputs

- QC-filtered, normalized gene expression matrix (rows = genes, columns = cells)
- Downstream: cluster labels and top marker genes per cluster written to cluster_annotation_table.csv (three epithelial clusters: Basal, L1, L2)

## How to apply

Download the raw count matrix for the sample of interest (e.g., the Ind4 breast epithelial sample from GEO accession GSE113197) so rows are genes and columns are cells, matching the expression_matrix convention expected by the ORIGINS R package. In R, load the dplyr/magrittr utility stack and apply quality-control filtering to remove low-quality cells and genes, then normalize the gene expression matrix. The resulting normalized matrix is the input for PCA followed by a neighbor-embedding such as UMAP, graph-based clustering of epithelial subpopulations, and marker-based annotation (Basal: KRT14/KRT5/KRT17/ACTA2/MYLK; Luminal-1: SLPI/LTF; Luminal-2: ANKRD30A/AREG/AGR2). Rationale: the article illustrates 'a complete workflow from the count matrix to trajectory inference using the breast data set', so QC/normalization is the mandatory first stage before any PPIN-based pluripotency scoring or annotation downstream.

## Related tools

- **R** (Analysis environment in which the count-matrix QC filtering and normalization are performed as part of the ORIGINS workflow) — https://github.com/danielasenraoka/ORIGINS
- **dplyr** (Imported utility package used for the data-manipulation steps of QC and normalization)
- **magrittr** (Imported utility package providing the pipe-style syntax used in the R normalization workflow)
- **ORIGINS** (Downstream consumer of the normalized expression matrix; its activity() function expects a matrix or data frame where each row is a gene and each column is a cell) — https://github.com/danielasenraoka/ORIGINS

## Evaluation signals

- The normalized expression matrix has genes as rows and cells as columns, matching the expression_matrix format expected by ORIGINS
- Downstream graph-based clustering resolves exactly three annotated epithelial clusters (Basal, Luminal-1/L1, Luminal-2/L2)
- Each cluster is supported by its canonical marker gene set: KRT14/KRT5/KRT17/ACTA2/MYLK for Basal, SLPI/LTF for L1, ANKRD30A/AREG/AGR2 for L2
- The blue cluster shows high SLPI and LTF expression consistent with its Luminal 1 (L1) annotation
- cluster_annotation_table.csv contains the cluster labels and top marker genes per cluster

## Limitations

- The article only demonstrates the count-matrix-to-annotation workflow on the breast dataset (GSE113197); generalization to other tissues is asserted via the additional benchmark datasets (colon GSE125970, hematopoietic GSE117498, lung GSE161383) but the QC stage is illustrated for breast cells
- ORIGINS is noted as more memory-efficient than LandSCENT and CytoTRACE, but very large matrices still require the R memory environment described in the methods

## Evidence

- [abstract] A complete workflow from count matrix to trajectory inference is illustrated using the breast dataset: "We also illustrate a complete workflow from the count matrix to trajectory inference using the breast data set."
- [supplementary] Breast epithelial scRNA-seq source data: "available on the GEO database under the accession codes: GSE113197 for breast epithelial cells"
- [readme] Expected expression matrix orientation: "expression_matrix is a matrix or data frame, where each row is a gene and each column is a cell."
- [intro] ORIGINS consumes the expression matrix after QC/normalization: "R package that provides a function to compute activity of a Protein-protein Interaction Network (PPIN) asociated with differentiation for scRNA-seq data."
