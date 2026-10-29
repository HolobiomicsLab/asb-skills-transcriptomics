---
name: kl-divergence-based-spatial-deg-ranking
description: Use when you have spatial transcriptomics counts plus per-spot 2D tissue
  coordinates (e.g., a 10x Visium Seurat object) and want to identify spatially regulated
  genes without defining spatial clusters or bins.
when_to_use_negative:
- Your coordinates are a dimensionality-reduction embedding (t-SNE/UMAP/PCA) rather
  than physical spot positions — this skill specifically requires the actual 2D coordinates
  of spots inside the tissue
- You need continuous expression values for DEG testing — the current implementation
  requires binary detection data and genes with different expression levels but the
  same detection pattern receive identical D_KL and p-values
- Genes are detected in fewer than 10 spots/cells — the article's filter removes these
  very lowly expressed genes before running haystack
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: singleCellHaystack
  role: Provides haystack(), which computes Kullback-Leibler divergence D_KL of gene
    detection distributions against random positioning, plus show_result_haystack()
    for ranked output; use the 'binary' branch for the version described in this article
  repo: github.com/alexisvdb/singleCellHaystack
- name: Seurat
  role: Loads the Visium slice, exposes counts via GetAssayData, tissue coordinates
    via GetTissueCoordinates, and log-normalization via NormalizeData
  repo: https://github.com/satijalab/seurat
- name: SeuratData
  role: Distributes and installs the stxBrain spatial dataset packages used as input
    (InstallData/LoadData)
  repo: https://github.com/satijalab/seurat-data
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-17900-3
    title: A clustering-independent method for finding differentially expressed genes
      in single-cell transcriptome data
  source_content_hash: sha256:992efd8ff3f06f3d1966b30fcd7f6a3e49ea15d38e64740c7593e077bd127d52
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - Seurat
  - SeuratData
  - singleCellHaystack
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# kl-divergence-based-spatial-deg-ranking

## Summary

Ranks spatially differentially expressed genes by applying singleCellHaystack's Kullback-Leibler divergence-based haystack() to the actual 2D tissue spot coordinates of a spatial transcriptomics dataset (e.g., 10x Visium), rather than a dimensionality-reduction embedding.

## When to use

Use when you have spatial transcriptomics counts plus per-spot 2D tissue coordinates (e.g., a 10x Visium Seurat object) and want to identify spatially regulated genes without defining spatial clusters or bins.

## When NOT to use

- Your coordinates are a dimensionality-reduction embedding (t-SNE/UMAP/PCA) rather than physical spot positions — this skill specifically requires the actual 2D coordinates of spots inside the tissue
- You need continuous expression values for DEG testing — the current implementation requires binary detection data and genes with different expression levels but the same detection pattern receive identical D_KL and p-values
- Genes are detected in fewer than 10 spots/cells — the article's filter removes these very lowly expressed genes before running haystack

## Inputs

- Seurat spatial object (e.g., 10x Visium stxBrain anterior1 slice loaded via SeuratData::LoadData)
- Counts matrix from GetAssayData
- 2D tissue spot coordinates from GetTissueCoordinates
- Detection cutoff parameter (e.g., 1, counts > 0)

## Outputs

- Ranked spatial DEG table with columns D_KL, log.p.vals, log.p.adj (e.g., spatial_deg_visium_anterior1.tsv)
- Top-ranked gene list from show_result_haystack(res, n=10)

## How to apply

Load the spatial dataset with Seurat/SeuratData (e.g., stxBrain anterior1 Visium slice via LoadData/InstallData), extract the counts matrix with GetAssayData, and filter out genes with non-zero counts in fewer than 10 spots to drop very lowly expressed genes and cut computation. Log-normalize with NormalizeData, then extract the true tissue coordinates with GetTissueCoordinates (not a UMAP/t-SNE embedding) and run haystack() on those 2D coordinates with the normalized expression matrix, using a binary detection cutoff of 1 (counts > 0) and set.seed(123) for reproducibility. Rank results with show_result_haystack (top 10) and write the full table (D_KL, log.p.vals, log.p.adj) to a TSV; note singleCellHaystack requires binary detection data as input, so the cutoff defines the 'detected' state per spot.

## Related tools

- **singleCellHaystack** (Provides haystack(), which computes Kullback-Leibler divergence D_KL of gene detection distributions against random positioning, plus show_result_haystack() for ranked output; use the 'binary' branch for the version described in this article) — github.com/alexisvdb/singleCellHaystack
- **Seurat** (Loads the Visium slice, exposes counts via GetAssayData, tissue coordinates via GetTissueCoordinates, and log-normalization via NormalizeData) — https://github.com/satijalab/seurat
- **SeuratData** (Distributes and installs the stxBrain spatial dataset packages used as input (InstallData/LoadData)) — https://github.com/satijalab/seurat-data

## Examples

```
set.seed(123); res <- haystack(x = spots_2D_coords, y = normalized_counts, detection = counts > 0); show_result_haystack(res, n = 10)
```

## Evaluation signals

- Output TSV exists, is parseable, and contains the expected columns D_KL, log.p.vals, log.p.adj
- Top-ranked spatial DEGs include the expected structure-correlated genes: Gpr88, Pde10a, Rgs9 (caudate putamen), Slc17a7, Nptxr (cerebral cortex), Pcp4l1 (olfactory bulb)
- Genes with fewer than 10 spots/cells with non-zero counts are absent from the ranked table
- Ranks are reproducible across runs with set.seed(123)
- Input coordinates are spot tissue coordinates, not an embedding — verify dimensions match spot count and values are physical tissue positions

## Limitations

- singleCellHaystack currently requires binary detection data; two genes with the same detection pattern but very different expression levels get identical D_KL and p-values (hard-threshold weakness)
- Many zero counts may reflect dropout from lack of sensitivity; no imputation is applied
- The method can return inflated p-values because coordinates (here, spatial positions are an exception, but in general PCs/t-SNE/UMAP inputs) reuse the gene expression data; fix deferred to future updates
- The updated singleCellHaystack v1.0 on master/CRAN (2022, Sci Rep) differs from the version described in this paper — use the 'binary' GitHub branch to reproduce this workflow

## Evidence

- [abstract] Our method uses Kullback-Leibler divergence to find genes that are expressed in subsets of cells that are non-randomly positioned in a multidimensional space: "Our method uses Kullback-Leibler divergence to find genes that are expressed in subsets of cells that are non-randomly positioned in a multidimensional space"
- [methods] spatial coordinates as input space: "we are not using an embedding as input space here, but the actual 2D coordinates of spots inside the tissue"
- [methods] low-expression gene filter: "We filter genes with less 10 cells with non-zero counts. This reduces the computational time by eliminating very lowly expressed genes."
- [discussion] binary detection requirement: "The current implementation of our method requires binary detection data as input (i.e., a gene is either detected or not in each cell)."
- [readme] version branch caveat: [evidence span withheld — anchor status: external_doc]
