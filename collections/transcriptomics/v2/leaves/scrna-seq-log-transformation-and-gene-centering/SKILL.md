---
name: scrna-seq-log-transformation-and-gene-centering
description: Use when a raw scRNA-Seq count matrix (genes × cells) is about to be
  fed into ccRemover or PCA-based factor analysis, and the paper's protocol requires
  log-scaling and gene-wise centering first.
when_to_use_negative:
- Data are already normalized on a log scale and gene-centered (e.g. re-processing
  a corrected matrix like ccRemover's xhat output)
- The downstream method does not operate on PCA of the expression matrix and does
  not require gene-wise centering
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: ccRemover
  role: Downstream method applied to the log-transformed, centered matrix to identify
    and remove cell-cycle principal components
  repo: https://github.com/cran/ccRemover
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/srep33892
    title: Identifying and removing the cell-cycle effect from single-cell RNA-Sequencing
      data
  source_content_hash: sha256:b48e0257d0a571959721082d0e03bf2cc9573a99d473c40bf4c654f66755ac81
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cran/ccRemover
  tools:
  - Biomart
  - Python
  - ccRemover
  - org.Mm.eg.db
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# scrna-seq-log-transformation-and-gene-centering

## Summary

Pre-processing step for scRNA-Seq expression matrices in which data are log-transformed and centered on a gene-by-gene basis prior to cell-cycle effect removal with ccRemover, so that principal components capture comparable variance scales across genes.

## When to use

Use when a raw scRNA-Seq count matrix (genes × cells) is about to be fed into ccRemover or PCA-based factor analysis, and the paper's protocol requires log-scaling and gene-wise centering first.

## When NOT to use

- Data are already normalized on a log scale and gene-centered (e.g. re-processing a corrected matrix like ccRemover's xhat output)
- The downstream method does not operate on PCA of the expression matrix and does not require gene-wise centering

## Inputs

- scRNA-Seq expression matrix (genes as rows, cells as columns), e.g. from Buettner et al. mESC (E-MTAB-2802), T helper cell, or lung adenocarcinoma datasets

## Outputs

- Log-transformed, gene-by-gene centered expression matrix ready for ccRemover / PCA-based cell-cycle effect removal

## How to apply

Ensure the expression matrix has genes as rows and cells as columns, then transform the measurements to a log scale and center each gene so its mean is zero across cells. For the lung adenocarcinoma dataset the paper specifies adding 1 to each measurement before taking the natural log. Centering gene-by-gene is recommended before applying ccRemover (e.g., with cutoff = 3, max_it = 4, nboot = 200, ntop = 10), because PCA-based detection of cell-cycle effects assumes comparably scaled gene features; the log shift also stabilizes the zero-inflated, skewed count distributions typical of scRNA-Seq. Apply optional gene filtering (e.g., removing genes with zero expression in over two thirds of cells) before transformation.

## Related tools

- **ccRemover** (Downstream method applied to the log-transformed, centered matrix to identify and remove cell-cycle principal components) — https://github.com/cran/ccRemover

## Examples

```
xhat <- ccRemover(dat, cutoff = 3, max_it = 4, nboot = 200, ntop = 10, bar=FALSE)
```

## Evaluation signals

- Every gene row of the output matrix has mean ≈ 0 across cells after centering
- Values are on a log scale (e.g. natural log of raw measurement + 1), not raw counts
- Genes with zero expression in over two thirds of cells (e.g. lung adenocarcinoma: 57,820 → 10,977 genes) were removed before transformation
- The centered matrix is oriented genes × cells before being passed to ccRemover with cutoff = 3, max_it = 4, nboot = 200, ntop = 10

## Limitations

- Annotation databases used to label cell-cycle genes downstream are always incomplete and inaccurate, which affects what the transformed data are compared against
- If the cell-cycle effect influences only a few control genes (below ~8% affected), PCA on the transformed matrix may fail to capture it
- For data with very low cell-cycle activity (e.g. glioblastoma), a smaller cutoff CΔ = 2 is used instead of 3

## Evidence

- [methods] We recommend transforming the data to a log scale and centering it on a gene-by-gene basis.: "We recommend transforming the data to a log scale and centering it on a gene-by-gene basis."
- [results] The data was transformed to a log-scale by adding 1 to each of the measurements and taking the natural log: "The data was transformed to a log-scale by adding 1 to each of the measurements and taking the natural log"
- [results] any genes which had zero expression for over two thirds of the cells were removed from the data, leaving 10,977 genes: "any genes which had zero expression for over two thirds of the cells were removed from the data, leaving 10,977 genes of which 757 were annotated to the cell cycle"
