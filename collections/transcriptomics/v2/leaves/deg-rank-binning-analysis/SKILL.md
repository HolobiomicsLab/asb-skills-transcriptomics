---
name: deg-rank-binning-analysis
description: Use when a ranked DEG list (sorted by p-value, e.g. log.p.vals from show_result_haystack)
  needs benchmarking against a curated marker database to test whether top hits are
  enriched for known cell-type markers.
when_to_use_negative:
- No ranked DEG list is available yet — this skill consumes singleCellHaystack output
  (haystack results sorted by log.p.vals), not raw count matrices or unprocessed scRNA-seq
  data
- Continuous expression values are the intended input — the method version described
  requires binary detection data (a gene is either detected or not in each cell),
  so continuous normalized counts are not directly supported
- No curated marker reference (e.g. CellMarker with publication counts) exists for
  the organism studied, making marker-enrichment validation impossible
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: singleCellHaystack
  role: Produces the ranked DEG predictions (haystack / show_result_haystack, sorted
    by log.p.vals) that are binned by rank and tested for CellMarker marker enrichment
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: R
  role: Implementation environment for singleCellHaystack and for the rank-binning,
    marker-class counting, and CSV export steps
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-17900-3
    title: A clustering-independent method for finding differentially expressed genes
      in single-cell transcriptome data
  source_content_hash: sha256:63e099b23f8adf2a7c300ae1a5fd805f3514a6ec10d990531501515282da9012
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - R
  - singleCellHaystack
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# deg-rank-binning-analysis

## Summary

Validates clustering-independent DEG predictions (e.g. from singleCellHaystack's Kullback-Leibler-divergence-based haystack function) by checking whether top-ranked DEGs are enriched for known, high-confidence CellMarker cell-type marker genes using consecutive rank bins of 50.

## When to use

Use when a ranked DEG list (sorted by p-value, e.g. log.p.vals from show_result_haystack) needs benchmarking against a curated marker database to test whether top hits are enriched for known cell-type markers.

## When NOT to use

- No ranked DEG list is available yet — this skill consumes singleCellHaystack output (haystack results sorted by log.p.vals), not raw count matrices or unprocessed scRNA-seq data
- Continuous expression values are the intended input — the method version described requires binary detection data (a gene is either detected or not in each cell), so continuous normalized counts are not directly supported
- No curated marker reference (e.g. CellMarker with publication counts) exists for the organism studied, making marker-enrichment validation impossible

## Inputs

- Ranked DEG list from singleCellHaystack (genes sorted by log.p.vals, e.g. via show_result_haystack on haystack output)
- CellMarker mouse marker gene database with per-marker publication counts
- Pre-defined marker classes: high-confidence (>=5 publications) and low-confidence (1-4 publications)

## Outputs

- marker_enrichment_by_rank.csv — per-rank-bin table of marker frequencies with marker-class and rank-bin-index annotations
- Per-bin frequency summary (e.g. top-50 bin enrichment vs. tail of the ranked list)

## How to apply

Download the CellMarker mouse marker database and split markers into a high-confidence class (reported in >=5 publications) and a low-confidence class (1-4 publications). Obtain the DEG list produced by the singleCellHaystack `haystack` function, sorted by log.p.vals as returned via show_result_haystack, on the transcriptome datasets of interest (the paper analyzes 136 real datasets). Partition the ranked list into consecutive bins of 50 ranks and count, per bin, the frequency of high-confidence markers, annotating each row with the marker class and rank-bin index. Write the result to marker_enrichment_by_rank.csv. Rationale: if the method recovers biology, the top-50 bin should contain a high fraction of known markers while later bins should not.

## Related tools

- **singleCellHaystack** (Produces the ranked DEG predictions (haystack / show_result_haystack, sorted by log.p.vals) that are binned by rank and tested for CellMarker marker enrichment) — https://github.com/alexisvdb/singleCellHaystack
- **R** (Implementation environment for singleCellHaystack and for the rank-binning, marker-class counting, and CSV export steps)

## Examples

```
res <- haystack(dat.tsne, dat.expression)
```

## Evaluation signals

- The first bin (top 50 ranks) must show a high-confidence marker frequency consistent with the reported ~32.4% mean enrichment across the 136 analyzed datasets
- Marker frequency must decrease toward the tail of the ranked list (later rank bins contain progressively fewer high-confidence markers)
- marker_enrichment_by_rank.csv exists and every row carries both a marker-class annotation and a rank-bin index
- High-confidence markers are a small fraction of all genes (~2.2%), so top-bin enrichment far above this background baseline indicates success

## Limitations

- singleCellHaystack p-values may be inflated because the input coordinates (PCs, t-SNE, or UMAP) are derived from the same gene expression data, a double-use issue deferred to future updates
- The method requires binary detection data as input and uses a hard detection threshold left to the user, which is arbitrary and makes genes with different expression levels but identical detection patterns indistinguishable
- Many zero counts may reflect dropout events from lack of sensitivity; no imputation is applied

## Evidence

- [abstract] Illustrative scale of the benchmarking context: "We illustrate the usage of singleCellHaystack through applications on 136 real transcriptome datasets and a spatial transcriptomics dataset"
- [methods] How ranked results are obtained from haystack: "res <- haystack(dat.tsne, dat.expression)"
- [readme] What the method computes, underlying the ranking used for binning: [evidence span withheld — anchor status: external_doc]
- [discussion] P-value inflation caveat affecting downstream rank comparisons: "singleCellHaystack suffers from the same issue, because the input coordinates (PCs, t-SNE, or UMAP coordinates) are dimensions containing a large proportion of the variability in the original data."
