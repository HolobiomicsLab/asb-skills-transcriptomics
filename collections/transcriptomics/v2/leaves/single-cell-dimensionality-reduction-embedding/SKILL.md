---
name: single-cell-dimensionality-reduction-embedding
description: Use when a QC-filtered, normalized scRNA-seq count matrix (e.g., the
  Ind4 breast epithelium sample from GEO GSE113197, genes as rows and cells as columns)
  must be projected to low-dimensional space (PCA, then UMAP) before graph-based clustering
  and marker-based cell annotation.
when_to_use_negative:
- The input is already an embedding or trajectory result rather than a QC-filtered,
  normalized expression matrix — re-embedding would double-reduce the data.
- The goal is computing differentiation activity scores with ORIGINS activity(expression_matrix,
  differentiation_edges), which consumes the expression matrix directly and does not
  require an embedding step.
- The data are not single-cell expression profiles (e.g., bulk RNA-seq or PPI edge
  lists such as differentiation_edges), where per-cell clustering in embedded space
  is meaningless.
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Runtime environment in which the PCA/UMAP embedding and downstream graph-based
    clustering of the breast epithelial matrix are executed
- name: Seurat
  role: Workflow framework used to reproduce the Ind4 cell-type annotation, including
    the dimensionality reduction (PCA followed by UMAP) step
- name: dplyr
  role: Part of the dplyr/magrittr utility stack used to QC-filter and normalize the
    gene expression matrix before dimensionality reduction
- name: magrittr
  role: Pipe-based utility in the same R preprocessing stack that prepares the normalized
    matrix for PCA/UMAP
- name: ORIGINS
  role: 'Downstream consumer in the same breast workflow: after the count matrix is
    processed, ORIGINS computes differentiation activity on the expression matrix
    to select trajectory origins'
  repo: https://github.com/danielasenraoka/ORIGINS
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.mex.2022.101778
    title: 'ORIGINS: A protein network-based approach to quantify cell pluripotency
      from scRNA-seq data'
  source_content_hash: sha256:9c2c093f7a5c15503d98706c3435890f9eabe6a8b2d48f207c832a31ee9b4d15
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/danielasenraoka/ORIGINS
  tools:
  - ORIGINS
  - R
  - Seurat
  - dplyr
  - magrittr
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# single-cell-dimensionality-reduction-embedding

## Summary

PCA followed by a neighbor-embedding (UMAP) on a normalized single-cell expression matrix, as performed in the GSE113197 (Ind4) breast epithelial workflow that goes from count matrix to cluster annotation and trajectory inference. The low-dimensional embedding is what makes graph-based clustering of epithelial subpopulations feasible.

## When to use

Use when a QC-filtered, normalized scRNA-seq count matrix (e.g., the Ind4 breast epithelium sample from GEO GSE113197, genes as rows and cells as columns) must be projected to low-dimensional space (PCA, then UMAP) before graph-based clustering and marker-based cell annotation.

## When NOT to use

- The input is already an embedding or trajectory result rather than a QC-filtered, normalized expression matrix — re-embedding would double-reduce the data.
- The goal is computing differentiation activity scores with ORIGINS activity(expression_matrix, differentiation_edges), which consumes the expression matrix directly and does not require an embedding step.
- The data are not single-cell expression profiles (e.g., bulk RNA-seq or PPI edge lists such as differentiation_edges), where per-cell clustering in embedded space is meaningless.

## Inputs

- Raw count matrix for the Ind4 breast epithelial sample from GEO under accession GSE113197
- QC-filtered, normalized gene expression matrix (each row is a gene, each column is a cell)

## Outputs

- PCA score matrix (cells × principal components)
- UMAP 2D embedding coordinates per cell
- Low-dimensional representation feeding graph-based clustering and the cluster_annotation_table.csv of Basal/L1/L2 labels

## How to apply

After quality-control filtering of low-quality cells and genes and normalization in R using the dplyr/magrittr utility stack (Seurat workflow), run PCA on the normalized gene expression matrix of the Ind4 breast epithelial sample, then compute a neighbor embedding such as UMAP on the top principal components. The embedded coordinates become the input for graph-based clustering that resolves the epithelial subpopulations (Basal, Luminal 1, Luminal 2). This is the count-matrix-to-annotation stage of the complete breast-dataset workflow that continues on to trajectory inference with ORIGINS. The number of retained PCs is an analyst decision — it must be sufficient to separate the three epithelial clusters — since neither the article nor the README specifies concrete PCA/UMAP parameters.

## Related tools

- **R** (Runtime environment in which the PCA/UMAP embedding and downstream graph-based clustering of the breast epithelial matrix are executed)
- **Seurat** (Workflow framework used to reproduce the Ind4 cell-type annotation, including the dimensionality reduction (PCA followed by UMAP) step)
- **dplyr** (Part of the dplyr/magrittr utility stack used to QC-filter and normalize the gene expression matrix before dimensionality reduction)
- **magrittr** (Pipe-based utility in the same R preprocessing stack that prepares the normalized matrix for PCA/UMAP)
- **ORIGINS** (Downstream consumer in the same breast workflow: after the count matrix is processed, ORIGINS computes differentiation activity on the expression matrix to select trajectory origins) — https://github.com/danielasenraoka/ORIGINS

## Evaluation signals

- The embedding resolves exactly three epithelial clusters (Basal, Luminal 1, Luminal 2) matching the published Ind4 annotation, as verified in cluster_annotation_table.csv.
- The blue cluster identified as Luminal 1 (L1) shows high expression of SLPI and LTF, typical Luminal Progenitor markers, when markers are projected onto the embedded clusters.
- Basal cells express KRT14/KRT5/KRT17/ACTA2/MYLK and Luminal-2 cells express ANKRD30A/AREG/AGR2, confirming each cluster is supported by its canonical marker gene set.
- The embedding was computed on the normalized matrix (rows = genes, columns = cells) after QC filtering, not on raw counts.
- The reduced representation is consistent with the count-matrix-to-trajectory stage of the published breast workflow, allowing the same data to feed ORIGINS differentiation-activity scoring.

## Limitations

- Neither the article nor the README specifies concrete PCA/UMAP parameters (number of PCs, neighbors), so the embedding quality depends on analyst choices.
- The Basal/L1/L2 marker scheme is specific to breast epithelium; the same ORIGINS workflow covers other healthy human tissues (colon GSE125970, hematopoietic bone marrow GSE117498, lung GSE161383) that would require their own annotation markers.
- The embedding is only as good as the upstream QC filtering of low-quality cells and genes and the normalization performed with the dplyr/magrittr stack.

## Evidence

- [abstract] abstract: "We also illustrate a complete workflow from the count matrix to trajectory inference using the breast data set."
- [supplementary] supplementary: "available on the GEO database under the accession codes: GSE113197 for breast epithelial cells"
- [readme] readme: "expression_matrix is a matrix or data frame, where each row is a gene and each column is a cell."
