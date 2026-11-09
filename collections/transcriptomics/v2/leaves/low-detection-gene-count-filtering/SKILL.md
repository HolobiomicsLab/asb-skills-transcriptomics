---
name: low-detection-gene-count-filtering
description: Use when preparing a counts matrix (e.g. 10x Visium spatial data or scRNA-seq)
  as input to haystack() and you need to eliminate very lowly expressed genes before
  running the DEG search.
when_to_use_negative:
- Input is already normalized expression or a filtered feature table rather than raw
  counts — the filter must operate on raw non-zero count detection
- The analysis requires very lowly expressed genes as targets (e.g. rare cell-type
  markers below the 10-cell detection floor) — this filter deliberately discards them
edam_operation: http://edamontology.org/operation_3933
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2229
tools:
- name: singleCellHaystack
  role: Downstream DEG prediction method whose binary detection input (cutoff 1) motivates
    the low-detection gene filter
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: Seurat
  role: Provides GetAssayData to extract the raw counts matrix on which the >= 10-cell/spot
    detection filter is applied
  repo: https://github.com/satijalab/seurat
- name: SeuratData
  role: Supplies the stxBrain anterior1 Visium dataset whose counts matrix is filtered
  repo: https://github.com/satijalab/seurat-data
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-020-17900-3
    title: A clustering-independent method for finding differentially expressed genes
      in single-cell transcriptome data
  source_content_hash: sha256:67e8bc0c0325a13f6f33e9d26df7dac8f7e85416b75c8b9aed53ae3346b27ee6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - R
  - Seurat
  - SeuratData
  - singleCellHaystack
  edam_operation: http://edamontology.org/operation_3933
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# low-detection-gene-count-filtering

## Summary

A preprocessing step for singleCellHaystack analyses that removes very lowly expressed genes — genes with non-zero counts in fewer than 10 cells (or spots) — to reduce computation and noise before Kullback-Leibler divergence-based DEG prediction.

## When to use

Use when preparing a counts matrix (e.g. 10x Visium spatial data or scRNA-seq) as input to haystack() and you need to eliminate very lowly expressed genes before running the DEG search.

## When NOT to use

- Input is already normalized expression or a filtered feature table rather than raw counts — the filter must operate on raw non-zero count detection
- The analysis requires very lowly expressed genes as targets (e.g. rare cell-type markers below the 10-cell detection floor) — this filter deliberately discards them

## Inputs

- Raw UMI counts matrix (genes x cells or genes x spots) extracted from a Seurat object (e.g. 10x Visium stxBrain anterior1 via GetAssayData)
- Detection threshold parameter (minimum 10 cells/spots with non-zero counts)

## Outputs

- Filtered counts matrix restricted to genes with non-zero counts in >= 10 cells/spots
- Gene subset used for subsequent NormalizeData and haystack() runs

## How to apply

After loading the raw counts (e.g. via Seurat's GetAssayData on a stxBrain anterior1 Visium object), filter out genes detected in fewer than 10 cells or spots with non-zero counts, since the article states this 'reduces the computational time by eliminating very lowly expressed genes'. Apply the filter on the raw counts matrix before normalization (NormalizeData log normalization follows). Keep the same threshold consistent with the downstream detection definition: haystack() uses binary detection data with a detection cutoff of 1 (counts > 0), so genes surviving the filter must still be detectable in a meaningful number of observations. For spatial Visium data the same rule applies at the spot level, e.g. keeping genes with non-zero counts in at least 10 tissue spots. Document which genes were removed, as the filtered matrix (not the full matrix) is what gets passed to haystack() together with the coordinates.

## Related tools

- **singleCellHaystack** (Downstream DEG prediction method whose binary detection input (cutoff 1) motivates the low-detection gene filter) — https://github.com/alexisvdb/singleCellHaystack
- **Seurat** (Provides GetAssayData to extract the raw counts matrix on which the >= 10-cell/spot detection filter is applied) — https://github.com/satijalab/seurat
- **SeuratData** (Supplies the stxBrain anterior1 Visium dataset whose counts matrix is filtered) — https://github.com/satijalab/seurat-data

## Examples

```
counts <- GetAssayData(anterior1, slot='counts'); gene.subset <- rownames(counts)[Matrix::rowSums(counts > 0) >= 10]; dat.expression <- counts[gene.subset, ]
```

## Evaluation signals

- Every gene retained in the filtered matrix has non-zero counts in at least 10 cells (or Visium spots)
- The filter is applied on raw counts, before NormalizeData log normalization and before haystack() with detection cutoff 1
- Top-ranked haystack results on the filtered Visium anterior1 data reproduce the article's spatial DEGs: Gpr88, Pde10a, Rgs9 (caudate putamen), Slc17a7, Nptxr (cerebral cortex), Pcp4l1 (olfactory bulb)
- The filtered gene set is substantially smaller than the full matrix, consistent with removing very lowly expressed genes and reducing computation

## Limitations

- The 10-cell threshold is a heuristic to cut computation; it discards genuinely very lowly expressed genes regardless of biological relevance
- Zero counts in single-cell data may reflect dropout from lack of sensitivity rather than true absence, so detection-based filtering can conflate dropout with true low expression
- The current singleCellHaystack implementation requires binary detection data (a hard threshold, e.g. counts > 0), so the filter only guarantees detection-frequency, not expression-level, relevance

## Evidence

- [methods] filter genes detected in fewer than 10 cells: "We filter genes with less 10 cells with non-zero counts. This reduces the computational time by eliminating very lowly expressed genes."
- [methods] spatial DEG analysis on Visium coordinates: "we are not using an embedding as input space here, but the actual 2D coordinates of spots inside the tissue"
- [discussion] binary detection requirement: "The current implementation of our method requires binary detection data as input (i.e., a gene is either detected or not in each cell)."
- [readme] updated package scope: [evidence span withheld — anchor status: external_doc]
