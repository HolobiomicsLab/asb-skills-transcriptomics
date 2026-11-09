---
name: cell-type-marker-verification-from-loadings
description: Use when a scRNA-seq count matrix (e.g. SingleCellExperiment) has been
  embedded with correspondence analysis and you want to check whether high-loading
  genes correspond to known cell-type markers (e.g. GNLY, TYROBP, LYZ, S100A8, CD74,
  HLA-DRA) and interpret cell-population associations.
when_to_use_negative:
- The goal is choosing the best dimension-reduction method for downstream clustering
  accuracy — the article's benchmarks show adapted CA variants (notably CA with Freeman-Tukey
  residuals) outperform standard CA, which is not the point of marker verification
  from loadings.
- The dataset is a complex tissue without well-defined cell clusters — the article
  notes Zhengmix and SCMixology benchmarks comprise well-defined cell clusters and
  are simpler than data from complex tissues, so marker recovery expectations differ.
- Ground-truth cell labels need independent validation — the article cautions benchmark
  annotations came from original publications and were not independently validated.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: corral
  role: R/Bioconductor package used to compute standard CA on the count matrix, interfacing
    directly with SingleCellExperiment, and to obtain cell and gene embeddings and
    loadings
  repo: github.com/laurenhsu1/corral
- name: made4
  role: Earlier package implementing CA (for microarray and bulk RNA-seq data) that
    provides biplot-style visualization of CA results
- name: gridExtra
  role: Used to arrange biplot panels when rendering the CA biplot figure
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-022-26434-1
    title: Correspondence analysis for dimension reduction, batch integration, and
      visualization of single-cell RNA-seq data
  source_content_hash: sha256:a7d86958645bfce7995a32734aeafc3635f04f16d58e440fee237a50106746fa
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor
  - R
  - corral
  - gridExtra
  - made4
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# cell-type-marker-verification-from-loadings

## Summary

Identify and verify known cell-type marker genes in scRNA-seq data by ranking features on the L2 norm of their correspondence analysis (CA) loadings and inspecting the top-weight genes on a first-two-dimension CA biplot, as implemented with the corral R/Bioconductor package.

## When to use

Use when a scRNA-seq count matrix (e.g. SingleCellExperiment) has been embedded with correspondence analysis and you want to check whether high-loading genes correspond to known cell-type markers (e.g. GNLY, TYROBP, LYZ, S100A8, CD74, HLA-DRA) and interpret cell-population associations.

## When NOT to use

- The goal is choosing the best dimension-reduction method for downstream clustering accuracy — the article's benchmarks show adapted CA variants (notably CA with Freeman-Tukey residuals) outperform standard CA, which is not the point of marker verification from loadings.
- The dataset is a complex tissue without well-defined cell clusters — the article notes Zhengmix and SCMixology benchmarks comprise well-defined cell clusters and are simpler than data from complex tissues, so marker recovery expectations differ.
- Ground-truth cell labels need independent validation — the article cautions benchmark annotations came from original publications and were not independently validated.

## Inputs

- scRNA-seq count matrix as a SingleCellExperiment object (e.g. Zhengmix8eq PBMC dataset)
- Cell-type annotation labels or prior biological knowledge of expected marker genes

## Outputs

- Ranked list of genes by L2 norm of CA loadings (top features, e.g. top 20)
- CA biplot of the first two dimensions with cells and labeled top genes (e.g. zhengmix8ea_ca_biplot.png)
- Verified list of biologically significant marker genes recovered from the loadings

## How to apply

Run standard CA on the scRNA-seq count matrix with the corral package, which computes the embedding by SVD of the Pearson chi-squared residual matrix without log-transforming the counts. Compute the L2 norm of each gene's loadings in the CA solution and rank features by this weight, selecting the top genes (e.g. the top 20 furthest from the origin in the first two CA components). Render a CA biplot of the first two dimensions showing cells together with the labeled top-weight genes (e.g. arranging panels with gridExtra). Validate by confirming that the labeled high-weight features include biologically expected markers such as GNLY (NK cells) and TYROBP (NK and CD14 monocytes) — the biplot gives a geometric interpretation of features and objects in the same space, facilitating cluster interpretation.

## Related tools

- **corral** (R/Bioconductor package used to compute standard CA on the count matrix, interfacing directly with SingleCellExperiment, and to obtain cell and gene embeddings and loadings) — github.com/laurenhsu1/corral
- **made4** (Earlier package implementing CA (for microarray and bulk RNA-seq data) that provides biplot-style visualization of CA results)
- **gridExtra** (Used to arrange biplot panels when rendering the CA biplot figure)

## Examples

```
library(corral); sce <- corral(sce); top_genes <- order(sqrt(rowSums(attr(sce, 'corral_preproc')$u^2)), decreasing=TRUE)[1:20]
```

## Evaluation signals

- The labeled high-weight features on the biplot include the six expected marker genes GNLY, TYROBP, LYZ, S100A8, CD74, and HLA-DRA.
- The top 20 genes ranked by L2 norm of their loadings are the furthest from the origin in the first two CA components.
- The biplot output file (e.g. zhengmix8ea_ca_biplot.png) exists and renders correctly.
- The CA embedding was computed from the Pearson chi-squared residual matrix without log-transformation of counts (no log(x + 1) step before SVD).
- Cell populations on the biplot separate along the first two CA dimensions in a way consistent with the known cell types of the Zhengmix8eq dataset.

## Limitations

- No tool or package versions reported for corral or dependencies, so exact reproduction may require environment troubleshooting.
- The Zhengmix and SCMixology benchmarks comprise well-defined cell clusters and are by design simpler than data from complex tissues, so marker recovery on complex tissues may be harder.
- Dataset cell type annotations come from original publications and were not independently validated.

## Evidence

- [intro] We show that the CA biplot provides a geometric interpretation of features and objects in the same space, which in turn facilitates efficient exploratory data analysis and cluster interpretation.: "We show that the CA biplot provides a geometric interpretation of features and objects in the same space, which in turn facilitates efficient exploratory data analysis and cluster interpretation."
- [intro] CA is computed by SVD of the Pearson residuals to reveal the row-column associations that deviate from expectation: "CA is computed by SVD of the Pearson residuals to reveal the row-column associations that deviate from expectation"
- [intro] We implemented standard and adapted CA for scRNAseq in corral, an R/Bioconductor package that interfaces directly with Bioconductor classes (including SingleCellExperiment): "We implemented standard and adapted CA for scRNAseq in corral, an R/Bioconductor package that interfaces directly with Bioconductor classes (including SingleCellExperiment)"
- [readme] The package contains two main function calls: 1. `corral` for dimensionality reduction on a single table: "1. `corral` for dimensionality reduction on a single table"
- [intro] In made4, Culhane et al. implemented CA for microarray and bulk RNA-seq data: "In made4, Culhane et al. implemented CA for microarray and bulk RNA-seq data"
