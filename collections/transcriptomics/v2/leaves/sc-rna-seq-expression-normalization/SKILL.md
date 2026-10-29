---
name: sc-rna-seq-expression-normalization
description: Use when a raw scRNA-seq count matrix (e.g., a GEO download like the
  GSE113197 Ind4 breast epithelial sample) must be QC-filtered and normalized in R
  before clustering, annotation, or activity() scoring.
when_to_use_negative:
- Input is bulk RNA-seq or a pooled sample rather than a single-cell expression matrix,
  where per-cell mean-expression normalization is meaningless.
- The expression matrix is already normalized and only downstream steps (Seurat clustering,
  ORIGINS activity computation, trajectory inference) remain.
- The goal is computing PPIN activity from a published normalized matrix without reprocessing
  the raw counts.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: dplyr
  role: R data-manipulation package used to QC-filter and normalize the scRNA-seq
    expression matrix (listed as an Imports dependency of ORIGINS)
- name: magrittr
  role: Pipe-based utility used with dplyr for the preprocessing/normalization chain
    (listed as an Imports dependency of ORIGINS)
- name: ORIGINS
  role: Downstream consumer of the normalized expression matrix; its activity() function
    requires genes-as-rows, cells-as-columns normalized input
  repo: https://github.com/danielasenraoka/ORIGINS
- name: ORIGINS2
  role: Extended version whose activity() function explicitly documents that the input
    must be the normalized expression matrix of the data set
  repo: https://github.com/danielasenraoka/ORIGINS2
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.mex.2022.101778
    title: 'ORIGINS: A protein network-based approach to quantify cell pluripotency
      from scRNA-seq data'
  source_content_hash: sha256:2b21def618369f6392ac2e5669dacd714c2f7846a5920692c40a8d8ba4ca4ee5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/danielasenraoka/ORIGINS
  tools:
  - ORIGINS
  - ORIGINS2
  - R
  - dplyr
  - magrittr
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# sc-rna-seq-expression-normalization

## Summary

Quality-control filtering and normalization of a raw single-cell RNA-seq count matrix into a genes-by-cells expression matrix suitable for downstream analyses such as Seurat clustering and ORIGINS PPIN-activity scoring. In the ORIGINS workflow this step sits between GEO download (e.g., GSE113197 breast epithelial counts) and differentiation-activity computation.

## When to use

Use when a raw scRNA-seq count matrix (e.g., a GEO download like the GSE113197 Ind4 breast epithelial sample) must be QC-filtered and normalized in R before clustering, annotation, or activity() scoring.

## When NOT to use

- Input is bulk RNA-seq or a pooled sample rather than a single-cell expression matrix, where per-cell mean-expression normalization is meaningless.
- The expression matrix is already normalized and only downstream steps (Seurat clustering, ORIGINS activity computation, trajectory inference) remain.
- The goal is computing PPIN activity from a published normalized matrix without reprocessing the raw counts.

## Inputs

- raw scRNA-seq count matrix downloaded from GEO (e.g., GSE113197 breast epithelial sample), genes x cells orientation
- quality-control thresholds for low-quality cells and genes
- R environment with dplyr and magrittr for preprocessing

## Outputs

- normalized gene expression matrix (rows = genes, columns = cells) suitable for activity() or Seurat clustering
- downstream: cluster labels and top marker genes per cluster (e.g., cluster_annotation_table.csv with Basal, Luminal-1, Luminal-2)

## How to apply

Download the raw count matrix for the sample of interest from GEO (e.g., accession GSE113197 for breast epithelial cells) and load it into R with the dplyr/magrittr utility stack. Apply quality-control filtering to remove low-quality cells and genes, then normalize the gene expression matrix so values are comparable across cells. Orient the result with genes as rows and cells as columns, since ORIGINS' activity() function and the PyOrigins formulation (Y, the transposed normalized expression matrix) both expect this layout. The normalized matrix then feeds graph-based clustering (PCA followed by UMAP and neighbor clustering) for cell-type annotation, or directly into diff_activity <- activity(expression_matrix, differentiation_edges). The rationale is that ORIGINS' activity score sums expression over PPIN edges and normalizes by the number of edges and per-cell mean expression, so unnormalized counts would bias the score toward deeply sequenced cells.

## Related tools

- **dplyr** (R data-manipulation package used to QC-filter and normalize the scRNA-seq expression matrix (listed as an Imports dependency of ORIGINS))
- **magrittr** (Pipe-based utility used with dplyr for the preprocessing/normalization chain (listed as an Imports dependency of ORIGINS))
- **ORIGINS** (Downstream consumer of the normalized expression matrix; its activity() function requires genes-as-rows, cells-as-columns normalized input) — https://github.com/danielasenraoka/ORIGINS
- **ORIGINS2** (Extended version whose activity() function explicitly documents that the input must be the normalized expression matrix of the data set) — https://github.com/danielasenraoka/ORIGINS2

## Examples

```
library(ORIGINS); diff_activity <- activity(normalized_expression_matrix, differentiation_edges)
```

## Evaluation signals

- Normalized matrix orientation check: rows are genes and columns are cells, matching the activity(expression_matrix, ...) contract.
- No negative or non-finite values remain after normalization; per-cell expression distributions are comparable across cells.
- After normalization plus clustering, the breast epithelial data resolves exactly three annotated clusters (Basal, L1, L2), each supported by its canonical markers (Basal: KRT14/KRT5/KRT17/ACTA2/MYLK; L1: SLPI/LTF; L2: ANKRD30A/AREG/AGR2).
- activity() on the normalized matrix returns a per-cell differentiation activity vector without errors from the differentiation_edges adjacency data frame.
- The cluster_annotation_table.csv output contains exactly three annotated epithelial clusters with the expected marker gene sets.

## Limitations

- ORIGINS' activity score normalization divides by e * mu_i (number of PPIN edges times per-cell mean expression); cells with extreme mean expression or many unexpressed PPIN genes can yield distorted scores if the expression matrix is not properly normalized first.
- ORIGINS2's activity() normalization parameter (FALSE = sum of PPIN edges, TRUE = normalized by edge count) affects the output scale, so consistency must be maintained across samples.
- The benchmark scope of the surrounding workflow is four healthy human datasets (breast GSE113197, colon GSE125970, hematopoietic GSE117498, lung GSE161383); behavior on diseased or non-human data is not demonstrated.

## Evidence

- [readme] expression_matrix is a matrix or data frame containing the normalized expression matrix of the data set, where each row is a gene and each column is a cell.: [evidence span withheld — anchor status: not_found]
- [abstract] We also illustrate a complete workflow from the count matrix to trajectory inference using the breast data set.: "We also illustrate a complete workflow from the count matrix to trajectory inference using the breast data set."
- [supplementary] available on the GEO database under the accession codes: GSE113197 for breast epithelial cells: "available on the GEO database under the accession codes: GSE113197 for breast epithelial cells"
- [readme] This factor enables comparisons between samples and various PPINs.: [evidence span withheld — anchor status: external_doc]
