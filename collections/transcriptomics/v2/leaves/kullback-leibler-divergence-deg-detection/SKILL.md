---
name: kullback-leibler-divergence-deg-detection
description: Use when you have a scRNA-seq expression matrix (genes x cells) plus
  a multidimensional cell embedding (PCA, t-SNE, UMAP, pseudotime, or 2D spatial coordinates)
  and need DEGs without defining cell clusters beforehand.
when_to_use_negative:
- Your expression data are continuous normalized counts and you need methods using
  continuous expression — the current implementation requires binary detection data;
  continuous-measure support is planned but not yet available.
- You have already clustered cells and need cluster-marker DEGs from a standard clustering-based
  workflow (e.g. Seurat or monocle 2 based pipelines).
- 'Strictly calibrated p-values are required: singleCellHaystack returns inflated
  p-values because the input coordinates (PCs, t-SNE, UMAP) already capture much of
  the gene expression variability.'
- You need to distinguish genes with different expression levels but the same detection
  pattern — the hard detection threshold gives them identical D_KL and p-values.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: singleCellHaystack
  role: Core R package implementing Kullback-Leibler divergence-based DEG prediction
    without clustering; provides haystack/haystack_highD, show_result_haystack, kmeans_haystack,
    and hclust_haystack. Install via install.packages('singleCellHaystack') or the
    'binary' GitHub branch for the 2020 Nature Communications version.
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: R
  role: Runtime environment; the package is an R library loaded with library(singleCellHaystack).
- name: Seurat
  role: Used in the manuscript workflows to load/preprocess single-cell data (e.g.
    Visium brain, Slide-seqV2 hippocampus) before running haystack on coordinates
    or embeddings.
  repo: https://github.com/satijalab/seurat
- name: SeuratData
  role: Distributes the example datasets (stxBrain anterior1 Visium brain, ssHippo
    Slide-seqV2) used in the spatial singleCellHaystack vignettes.
  repo: https://github.com/satijalab/seurat-data
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-17900-3
    title: A clustering-independent method for finding differentially expressed genes
      in single-cell transcriptome data
  source_content_hash: sha256:edf423db7045bf1d270d32eb6b17edfcee1389842e312d480e5d7c501c00911d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - R
  - Seurat
  - SeuratData
  - singleCellHaystack
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# kullback-leibler-divergence-deg-detection

## Summary

singleCellHaystack uses Kullback-Leibler divergence to identify differentially expressed genes that are non-randomly positioned in a multidimensional cell space (PCs, t-SNE, UMAP, pseudotime, or spatial coordinates), without clustering cells first. It is implemented as an R package (CRAN/GitHub) and was demonstrated on 136 real transcriptome datasets plus spatial transcriptomics data.

## When to use

Use when you have a scRNA-seq expression matrix (genes x cells) plus a multidimensional cell embedding (PCA, t-SNE, UMAP, pseudotime, or 2D spatial coordinates) and need DEGs without defining cell clusters beforehand.

## When NOT to use

- Your expression data are continuous normalized counts and you need methods using continuous expression — the current implementation requires binary detection data; continuous-measure support is planned but not yet available.
- You have already clustered cells and need cluster-marker DEGs from a standard clustering-based workflow (e.g. Seurat or monocle 2 based pipelines).
- Strictly calibrated p-values are required: singleCellHaystack returns inflated p-values because the input coordinates (PCs, t-SNE, UMAP) already capture much of the gene expression variability.
- You need to distinguish genes with different expression levels but the same detection pattern — the hard detection threshold gives them identical D_KL and p-values.

## Inputs

- Gene x cell binary detection matrix (expression matrix), genes as rows and cells as columns
- Multidimensional cell coordinates: PCs (e.g. first 50 PCs from PCA on the expression data), t-SNE/UMAP embedding, pseudotime values, or 2D spatial spot coordinates
- Optionally a gene detection threshold defined by the user to binarize expression counts

## Outputs

- Ranked DEG table ordered by log p-values / D_KL divergence (e.g. bone_marrow_deg_table.csv), with top-ranked genes such as Ramp1
- haystack result object containing grid coordinates and D_KL statistics usable with show_result_haystack

## How to apply

Load the expression matrix as genes (rows) x cells (columns) with binary detection calls (the current implementation requires detection data, not continuous counts). Compute the input multidimensional space — e.g. run PCA and retain the first 50 PCs as done for the Tabula Muris bone marrow FACS-sorted dataset (5,250 cells, 13,756 genes), or use t-SNE/UMAP coordinates or actual 2D spatial spot coordinates. Optionally filter genes detected in fewer than 10 cells to reduce computation time on very lowly expressed genes. Call haystack (haystack_highD for ≥2D spaces, default parameters) with the coordinates as x and the expression matrix as expression; a median-mode run on the bone marrow data takes about 225 s with a set random seed. Retrieve ranked results with show_result_haystack ordered by log p-values / D_KL divergence — the top-ranked DEG on the Tabula Muris bone marrow data is Ramp1, detected only in a subset of cells — and write the table (e.g. bone_marrow_deg_table.csv).

## Related tools

- **singleCellHaystack** (Core R package implementing Kullback-Leibler divergence-based DEG prediction without clustering; provides haystack/haystack_highD, show_result_haystack, kmeans_haystack, and hclust_haystack. Install via install.packages('singleCellHaystack') or the 'binary' GitHub branch for the 2020 Nature Communications version.) — https://github.com/alexisvdb/singleCellHaystack
- **R** (Runtime environment; the package is an R library loaded with library(singleCellHaystack).)
- **Seurat** (Used in the manuscript workflows to load/preprocess single-cell data (e.g. Visium brain, Slide-seqV2 hippocampus) before running haystack on coordinates or embeddings.) — https://github.com/satijalab/seurat
- **SeuratData** (Distributes the example datasets (stxBrain anterior1 Visium brain, ssHippo Slide-seqV2) used in the spatial singleCellHaystack vignettes.) — https://github.com/satijalab/seurat-data

## Examples

```
library(singleCellHaystack); res <- haystack(dat.pca[, 1:50], dat.expression); show_result_haystack(res)
```

## Evaluation signals

- Top-ranked DEG in the ranked table matches Ramp1 when reproducing the Tabula Muris bone marrow FACS-sorted dataset (5,250 cells, 13,756 genes, 50 PCs input) with default haystack_highD parameters and a set random seed.
- Results from show_result_haystack are sorted by log p-values / D_KL divergence, and every returned gene has a finite D_KL and p-value.
- Median-mode runtime is on the order of ~225 s for the bone marrow dataset — substantially faster runs may indicate wrong input dimensions or missing genes.
- Genes detected in fewer than 10 cells were removed before analysis (filter applied to very lowly expressed genes).
- Genes with identical detection patterns across cells receive identical D_KL and p-values — a sanity check that the binary detection input was used rather than continuous counts.

## Limitations

- P-values are inflated due to double use of gene expression data (input coordinates derive from the same expression matrix); the fix was deferred to future updates.
- Only binary detection data are supported; continuous expression (e.g. normalized read counts) support is planned but not yet implemented.
- The hard detection threshold is a weak point: two genes with the same detection pattern but ten-fold different expression levels get identical D_KL and p-values.
- The detection-threshold definition is left to the user, introducing arbitrariness.
- Many zero counts reflect dropout events; imputation could partly address this but is not applied.

## Evidence

- [abstract] Our method uses Kullback-Leibler divergence to find genes that are expressed in subsets of cells that are non-randomly positioned in a multidimensional space: "Our method uses Kullback-Leibler divergence to find genes that are expressed in subsets of cells that are non-randomly positioned in a multidimensional space"
- [methods] res <- haystack(dat.tsne, dat.expression): "res <- haystack(dat.tsne, dat.expression)"
- [methods] We filter genes with less 10 cells with non-zero counts.: "We filter genes with less 10 cells with non-zero counts. This reduces the computational time by eliminating very lowly expressed genes."
- [discussion] The current implementation of our method requires binary detection data as input (i.e., a gene is either detected or not in each cell).: "The current implementation of our method requires binary detection data as input (i.e., a gene is either detected or not in each cell)."
- [readme] For the older version described here, please use branch 'binary'.: "For the older version described
[here](https://doi.org/10.1038/s41467-020-17900-3), please use branch
“binary”."
