---
name: singlecellhaystack-result-ranking
description: Use when you have `haystack` result objects (p-values per gene) and need
  to rank genes by significance and/or benchmark the ranked list against a curated
  marker database such as CellMarker.
when_to_use_negative:
- Your goal is cluster-based DEG testing (e.g., Seurat/monocle cluster markers) rather
  than clustering-independent DEG ranking in a continuous embedding or spatial space
- Genes have not been filtered for detection in ≥10 cells, inflating low-expression
  noise and invalidating tail-rank comparisons
- No external marker reference (e.g., CellMarker) or ranked p-value list is available,
  so rank-bin enrichment cannot be computed
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: singleCellHaystack
  role: R package providing the `haystack` function (Kullback-Leibler divergence on
    binary detection data) and `show_result_haystack` for ranked DEG retrieval
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: R
  role: Implementation language for the haystack analysis and rank-bin marker enrichment
    tabulation
- name: Seurat
  role: Used to preprocess single-cell data (e.g., 10x Visium anterior1) and supply
    embeddings/coordinates as haystack input space
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-17900-3
    title: A clustering-independent method for finding differentially expressed genes
      in single-cell transcriptome data
  source_content_hash: sha256:f991c7072ac6d27b1a4a38cf1f7b09a627f3210910f691fbe3da5433c0b1d702
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - R
  - Seurat
  - singleCellHaystack
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# singlecellhaystack-result-ranking

## Summary

Rank the DEGs returned by the singleCellHaystack `haystack` function by significance (log.p.vals via `show_result_haystack`) and validate the ranking by quantifying enrichment of high-confidence CellMarker marker genes (reported in ≥5 publications) among consecutive 50-gene rank bins, as done across 136 real single-cell transcriptome datasets.

## When to use

Use when you have `haystack` result objects (p-values per gene) and need to rank genes by significance and/or benchmark the ranked list against a curated marker database such as CellMarker.

## When NOT to use

- Your goal is cluster-based DEG testing (e.g., Seurat/monocle cluster markers) rather than clustering-independent DEG ranking in a continuous embedding or spatial space
- Genes have not been filtered for detection in ≥10 cells, inflating low-expression noise and invalidating tail-rank comparisons
- No external marker reference (e.g., CellMarker) or ranked p-value list is available, so rank-bin enrichment cannot be computed

## Inputs

- Binary gene-by-cell detection matrix (gene detected or not per cell) from a scRNA-seq dataset
- 2D+ cell coordinates (t-SNE/UMAP, first 20 PCs, pseudotime, or tissue spatial coordinates)
- CellMarker mouse marker gene database with per-marker publication counts
- singleCellHaystack `haystack` result object (gene p-values / log.p.vals)

## Outputs

- Ranked DEG list sorted by log.p.vals (via show_result_haystack)
- marker_enrichment_by_rank.csv: per-rank-bin (bins of 50) counts of high- and low-confidence CellMarker markers
- Top-50 marker frequency estimate (~32.4% expected across 136 datasets)

## How to apply

Run the `haystack` function on a binary gene-detection matrix against a multidimensional cell space (t-SNE, UMAP, first 20 principal components, pseudotime, or physical 2D spatial coordinates), filtering genes detected in fewer than 10 cells beforehand. Retrieve the ranked gene list by sorting the result on log.p.vals with `show_result_haystack`, as used for the example and Tabula Muris-style scRNA-seq datasets. To validate ranking quality, download the CellMarker mouse marker database, split markers into high-confidence (reported in ≥5 publications) and low-confidence (1–4 publications) classes, partition the ranked list into consecutive bins of 50 ranks, and count high-confidence marker frequency per bin. Tabulate per-bin frequencies with marker-class annotation and rank-bin index into marker_enrichment_by_rank.csv. High-confidence markers are only 2.2% of all genes, so the expected enrichment in the top bin (~32.4% of the top 50) is a strong signal; frequency should decay toward the tail of the list.

## Related tools

- **singleCellHaystack** (R package providing the `haystack` function (Kullback-Leibler divergence on binary detection data) and `show_result_haystack` for ranked DEG retrieval) — https://github.com/alexisvdb/singleCellHaystack
- **R** (Implementation language for the haystack analysis and rank-bin marker enrichment tabulation)
- **Seurat** (Used to preprocess single-cell data (e.g., 10x Visium anterior1) and supply embeddings/coordinates as haystack input space) — https://github.com/satijalab/seurat

## Examples

```
res <- haystack(dat.tsne, dat.expression); ranked <- show_result_haystack(res)
```

## Evaluation signals

- First rank bin (top 50 genes) shows a high-confidence CellMarker frequency consistent with ~32.4% of the top 50 ranked DEGs
- Marker frequency decreases monotonically (or near-monotonically) toward the tail of the ranked list
- High-confidence markers (≥5 publications) constitute ~2.2% of all background genes, so any per-bin excess far above 2.2% indicates enrichment
- The output table marker_enrichment_by_rank.csv contains one row per (marker-class, rank-bin) with expected columns: marker class, rank-bin index, frequency

## Limitations

- The current `haystack` implementation requires binary detection data (gene detected or not per cell); continuous expression is not supported, and genes with identical detection patterns receive identical DKL and p-values regardless of expression level
- P-values are inflated because the input coordinates (PCs, t-SNE, UMAP) derive from the same gene expression data used for DEG testing; ranks are more reliable than absolute p-values
- The detection threshold definition is left to the user and may be arbitrary; zero counts may reflect dropouts and imputation is not applied

## Evidence

- [results] High-confidence CellMarker genes (reported in >=5 publications), though only 2.2% of all genes, made up on average 32.4% of the top 50 ranked DEGs across the 136 datasets: [evidence span withheld — anchor status: not_found]
- [methods] haystack call: "res <- haystack(dat.tsne, dat.expression)"
- [methods] gene filter: "We filter genes with less 10 cells with non-zero counts. This reduces the computational time by eliminating very lowly expressed genes."
- [discussion] binary detection requirement: "The current implementation of our method requires binary detection data as input (i.e., a gene is either detected or not in each cell)."
- [abstract] KL divergence method description: "Our method uses Kullback-Leibler divergence to find genes that are expressed in subsets of cells that are non-randomly positioned in a multidimensional space"
- [discussion] p-value inflation caveat: "singleCellHaystack suffers from the same issue, because the input coordinates (PCs, t-SNE, or UMAP coordinates) are dimensions containing a large proportion of the variability in the original data."
