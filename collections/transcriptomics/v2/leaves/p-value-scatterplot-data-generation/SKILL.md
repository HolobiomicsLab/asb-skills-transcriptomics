---
name: p-value-scatterplot-data-generation
description: Use when two DEG-prediction methods have been run on the same scRNA-seq
  dataset and you need per-gene p-value tables and top-N rank-overlap counts to quantify
  agreement.
when_to_use_negative:
- The current implementation of singleCellHaystack requires binary detection data
  as input, so it is the wrong choice if you need continuous expression levels (e.g.,
  normalized read counts) as the basis for the comparison
- singleCellHaystack is a clustering-independent method; do not use this comparison
  workflow when you need DEGs defined strictly relative to pre-existing cell cluster
  labels rather than non-random positioning in a multidimensional space
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: singleCellHaystack
  role: clustering-independent DEG predictor using Kullback-Leibler divergence; supplies
    one set of per-gene p-values for the scatterplot
  repo: github.com/alexisvdb/singleCellHaystack
- name: Seurat
  role: performs FindClusters and FindAllMarkers (default Wilcoxon rank-sum test)
    to supply the second set of per-gene p-values
  repo: github.com/satijalab/seurat
- name: R
  role: language in which both packages run and the p-value table / overlap statistics
    are compiled
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-17900-3
    title: A clustering-independent method for finding differentially expressed genes
      in single-cell transcriptome data
  source_content_hash: sha256:2f6c92a999ecb938f8d4023a3861a4fe9232246eab46c20cfe23d44c964adb7a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - singleCellHaystack
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# p-value scatterplot data generation

## Summary

Compiles per-gene significance values from two DEG prediction approaches — singleCellHaystack (Kullback-Leibler divergence) and Seurat FindAllMarkers (Wilcoxon rank-sum) — into a table suitable for a p-value scatterplot, and quantifies rank-overlap of their top-ranked genes on the Tabula Muris bone marrow dataset.

## When to use

Use when two DEG-prediction methods have been run on the same scRNA-seq dataset and you need per-gene p-value tables and top-N rank-overlap counts to quantify agreement.

## When NOT to use

- The current implementation of singleCellHaystack requires binary detection data as input, so it is the wrong choice if you need continuous expression levels (e.g., normalized read counts) as the basis for the comparison
- singleCellHaystack is a clustering-independent method; do not use this comparison workflow when you need DEGs defined strictly relative to pre-existing cell cluster labels rather than non-random positioning in a multidimensional space

## Inputs

- bone marrow (Tabula Muris) scRNA-seq gene expression matrix
- precomputed singleCellHaystack result object (haystack) with per-gene p-values
- Seurat object with cell clustering metadata

## Outputs

- gene-by-method table of p-values suitable for a p-value scatterplot
- top-100 rank-overlap statistics (shared gene count between methods)
- per-gene rank table for disagreement genes (Ramp1, Il2rb, Grm8, Tnni1)

## How to apply

Load the bone marrow scRNA-seq data and the precomputed singleCellHaystack results (haystack object) in R, having first filtered genes with fewer than 10 cells with non-zero counts. Cluster cells with Seurat FindClusters and call markers per cluster with FindAllMarkers using the default Wilcoxon rank-sum test. Extract per-gene p-values from both methods, compile them into a gene-by-method table aligned by gene name, and compute rank-overlap statistics for the top 100 high-scoring genes of each approach. Also extract the ranks of specific disagreement genes (Ramp1, Il2rb, Grm8, Tnni1) in both methods to characterize per-gene discrepancies. The expected result per Supplementary Fig. 8 is that the top-100 lists share only 3 genes.

## Related tools

- **singleCellHaystack** (clustering-independent DEG predictor using Kullback-Leibler divergence; supplies one set of per-gene p-values for the scatterplot) — github.com/alexisvdb/singleCellHaystack
- **Seurat** (performs FindClusters and FindAllMarkers (default Wilcoxon rank-sum test) to supply the second set of per-gene p-values) — github.com/satijalab/seurat
- **R** (language in which both packages run and the p-value table / overlap statistics are compiled)

## Examples

```
library(Seurat); markers <- FindAllMarkers(obj, test.use = "wilcox")
```

## Evaluation signals

- Top-100 high-scoring gene overlap between the two methods must equal exactly 3 shared genes, matching Supplementary Fig. 8
- Ranks of Ramp1, Il2rb, Grm8, and Tnni1 must match the values reported in the manuscript
- Gene sets in the p-value table must be name-matched across methods (rows are shared genes, not per-method gene lists)
- Input data must satisfy the gene filter of non-zero counts in at least 10 cells before analysis

## Limitations

- singleCellHaystack suffers from p-value inflation because the input coordinates (PCs, t-SNE, or UMAP) capture a large proportion of the variability in the original data, so p-values in the scatterplot should be interpreted cautiously
- The method requires binary detection data (a gene is detected or not in each cell); genes with different expression levels but the same detection pattern get identical DKL and p-value
- Late-2022 singleCellHaystack v1.0 on GitHub/CRAN differs from the original method; the original version must be installed from the 'binary' branch to reproduce this comparison

## Evidence

- [supplementary] The top 100 high-scoring genes of both approaches have only 3 genes in common (Supplementary Fig. 8).: "The top 100 high-scoring genes of both approaches have only 3 genes in common (Supplementary Fig. 8)."
- [methods] gene filtering: "We filter genes with less 10 cells with non-zero counts. This reduces the computational time by eliminating very lowly expressed genes."
- [methods] haystack call: "res <- haystack(dat.tsne, dat.expression)"
- [abstract] KL divergence method definition: "Our method uses Kullback-Leibler divergence to find genes that are expressed in subsets of cells that are non-randomly positioned in a multidimensional space"
- [discussion] binary input requirement: "The current implementation of our method requires binary detection data as input (i.e., a gene is either detected or not in each cell)."
- [discussion] p-value inflation caveat: "singleCellHaystack suffers from the same issue, because the input coordinates (PCs, t-SNE, or UMAP coordinates) are dimensions containing a large proportion of the variability in the original data."
- [readme] version branch warning: "The version on CRAN
is also this updated version. For the older version described
[here](https://doi.org/10.1038/s41467-020-17900-3), please use branch
“binary”."
