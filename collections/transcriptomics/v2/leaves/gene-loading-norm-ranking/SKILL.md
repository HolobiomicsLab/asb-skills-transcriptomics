---
name: gene-loading-norm-ranking
description: Use when you have a CA-embedded scRNA-seq dataset (e.g., Zhengmix8eq
  SingleCellExperiment with corral output) and need to identify which genes carry
  the highest weight in the first two CA dimensions and interpret their association
  with cell populations.
when_to_use_negative:
- Input data has already been log(x + 1) transformed for a typical PCA-based workflow
  — standard CA as implemented in corral decomposes the Pearson chi-squared residual
  matrix of raw counts and avoids the distortive log-transformation
- The goal is batch integration across multiple tables — use corralm (multi-table
  CA) instead of single-table corral for this feature-ranking visualization
- A different dimension reduction method (PCA, glmPCA, ZINB-WaVE, ZIFA) produced the
  embeddings, since gene loadings are then not on the CA chi-squared residual geometry
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: corral
  role: Computes standard CA on the scRNA-seq count matrix by SVD of the Pearson chi-squared
    residual matrix and returns cell and gene embeddings used for the loading-norm
    ranking
  repo: github.com/laurenhsu1/corral
- name: made4
  role: Earlier Bioconductor implementation of CA for microarray and bulk RNA-seq
    data that provides biplot visualization functionality
- name: gridExtra
  role: Arranges the CA biplot panels in the final figure
- name: R
  role: Runtime environment for the corral-based CA and plotting workflow
- name: Bioconductor
  role: Package ecosystem providing SingleCellExperiment classes that corral interfaces
    with directly
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:0839118985896878c39beb9b215ae71b5fdc08bc7e76503eaa47772eb3f83d2a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - corral
  - gridExtra
  - made4
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# gene-loading-norm-ranking

## Summary

Ranks genes by the L2 norm of their loadings from a correspondence analysis (CA) solution and renders a CA biplot labeling the top-weighted features, to identify which genes drive cell-population separation in scRNA-seq embeddings.

## When to use

Use when you have a CA-embedded scRNA-seq dataset (e.g., Zhengmix8eq SingleCellExperiment with corral output) and need to identify which genes carry the highest weight in the first two CA dimensions and interpret their association with cell populations.

## When NOT to use

- Input data has already been log(x + 1) transformed for a typical PCA-based workflow — standard CA as implemented in corral decomposes the Pearson chi-squared residual matrix of raw counts and avoids the distortive log-transformation
- The goal is batch integration across multiple tables — use corralm (multi-table CA) instead of single-table corral for this feature-ranking visualization
- A different dimension reduction method (PCA, glmPCA, ZINB-WaVE, ZIFA) produced the embeddings, since gene loadings are then not on the CA chi-squared residual geometry

## Inputs

- scRNA-seq count matrix stored as a SingleCellExperiment object (e.g., Zhengmix8eq PBMC dataset)
- CA embedding output from corral (gene loadings in the first two CA dimensions)

## Outputs

- Ranked list of genes by L2 norm of CA loadings (top 20 features)
- CA biplot of the first two dimensions with cells and 20 labeled top genes (e.g., zhengmix8ea_ca_biplot.png)

## How to apply

Run standard correspondence analysis on the raw scRNA-seq count matrix with the corral R/Bioconductor package, which computes the embedding by SVD of the Pearson chi-squared residual matrix without log-transformation of counts. Extract the gene (row/feature) loadings from the CA solution and compute the L2 norm of each gene's loading vector in the first two dimensions. Rank genes by this L2 norm and select the top 20 features (those furthest from the origin in the biplot). Render a biplot of the first two CA dimensions showing cells plus the 20 labeled top genes, arranging panels with gridExtra. Judge success by whether biologically significant genes such as GNLY (NK cells), TYROBP (NK and CD14 monocytes), LYZ, S100A8, CD74, and HLA-DRA appear among the labeled high-weight features.

## Related tools

- **corral** (Computes standard CA on the scRNA-seq count matrix by SVD of the Pearson chi-squared residual matrix and returns cell and gene embeddings used for the loading-norm ranking) — github.com/laurenhsu1/corral
- **made4** (Earlier Bioconductor implementation of CA for microarray and bulk RNA-seq data that provides biplot visualization functionality)
- **gridExtra** (Arranges the CA biplot panels in the final figure)
- **R** (Runtime environment for the corral-based CA and plotting workflow)
- **Bioconductor** (Package ecosystem providing SingleCellExperiment classes that corral interfaces with directly)

## Examples

```
sce_corral <- corral(sce, whichmat = 'counts'); gene_norm <- sqrt(rowSums(attr(sce_corral,'corral')$u[,1:2]^2))
```

## Evaluation signals

- The labeled high-weight features on the biplot include GNLY, TYROBP, LYZ, S100A8, CD74, and HLA-DRA
- The top 20 labeled genes are the 20 features furthest from the origin in the first two CA components (highest L2 norm of loadings)
- The output plot file (e.g., zhengmix8ea_ca_biplot.png) exists and renders correctly
- The CA was computed on raw counts (no log-transformation), i.e., via SVD of the Pearson chi-squared residual matrix
- The biplot displays cells and genes projected in the same CA space, enabling geometric interpretation of gene-cell population associations

## Limitations

- Benchmarking datasets like Zhengmix lack independently validated cell type annotations; gene-cell type associations (e.g., GNLY for NK cells) rest on the original study's assignments
- No tool or package versions for corral or its dependencies are reported, so exact reproducibility may vary
- Standard CA can be outperformed on cluster recovery by CA adaptations such as Freeman-Tukey residuals (CA-FT), which may give different high-loading gene rankings

## Evidence

- [abstract] CA is based on decomposition of a chi-squared residual matrix, avoiding distortive log-transformation: "CA is based on decomposition of a chi-squared residual matrix, avoiding distortive log-transformation"
- [intro] CA biplot geometric interpretation: "We show that the CA biplot provides a geometric interpretation of features and objects in the same space, which in turn facilitates efficient exploratory data analysis"
- [intro] corral interfaces with Bioconductor classes: "We implemented standard and adapted CA for scRNAseq in corral, an R/Bioconductor package that interfaces directly with Bioconductor classes"
- [intro] CA computed via SVD of Pearson residuals: "CA is computed by SVD of the Pearson residuals to reveal the row-column associations that deviate from expectation"
- [readme] corral install and main calls: "1. `corral` for dimensionality reduction on a single table"
