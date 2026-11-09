---
name: marker-enrichment-validation
description: Use when you have a ranked DEG list from scRNA-seq analysis and need
  to assess whether top-ranked genes are biologically meaningful by testing enrichment
  of known cell type marker genes by rank position.
when_to_use_negative:
- The input is an unranked gene set or cluster assignment rather than a gene list
  ordered by statistical significance (e.g. p-value), since the method relies on rank-bin
  partitioning
- No cell type marker reference database is available for the organism or tissue under
  study, so enrichment cannot be computed against a marker gold standard
edam_operation: http://edamontology.org/operation_3501
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: singleCellHaystack
  role: Produces the ranked DEG list via the `haystack` function (Kullback-Leibler
    divergence, no clustering), ranked by log.p.vals and accessed with `show_result_haystack`
  repo: github.com/alexisvdb/singleCellHaystack
- name: R
  role: 'Implementation language: splitting CellMarker markers by publication-count
    confidence class, binning the ranked list into 50-rank bins, and writing marker_enrichment_by_rank.csv'
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-17900-3
    title: A clustering-independent method for finding differentially expressed genes
      in single-cell transcriptome data
  source_content_hash: sha256:a677b12df4140d26ea62b17aa5fd51c0bad95be2fa1a9dd39cfdf0cfe0d63920
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - singleCellHaystack
  edam_operation: http://edamontology.org/operation_3501
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# marker-enrichment-validation

## Summary

Validates a ranked DEG list (e.g. from singleCellHaystack's haystack function sorted by log.p.vals) by quantifying enrichment of known cell type markers across rank bins, using the CellMarker mouse database stratified by publication count into high- (>=5 publications) and low-confidence (1-4 publications) marker classes.

## When to use

Use when you have a ranked DEG list from scRNA-seq analysis and need to assess whether top-ranked genes are biologically meaningful by testing enrichment of known cell type marker genes by rank position.

## When NOT to use

- The input is an unranked gene set or cluster assignment rather than a gene list ordered by statistical significance (e.g. p-value), since the method relies on rank-bin partitioning
- No cell type marker reference database is available for the organism or tissue under study, so enrichment cannot be computed against a marker gold standard

## Inputs

- Ranked DEG list from singleCellHaystack `haystack` (sorted by log.p.vals, e.g. via `show_result_haystack`)
- CellMarker mouse marker gene database with per-marker publication counts

## Outputs

- marker_enrichment_by_rank.csv: per-bin frequencies of high-confidence markers with marker-class annotation and rank-bin index
- Validation summary comparing the top-50-bin marker frequency against the reported ~32.4% enrichment baseline

## How to apply

Download the CellMarker mouse marker database, which records for each marker the number of publications reporting it, and split markers into a high-confidence class (>=5 publications) and a low-confidence class (1-4 publications). Obtain the ranked DEG list produced by singleCellHaystack's `haystack` function, sorted by log.p.vals (e.g. via `show_result_haystack` as used for the example and Tabula Muris-style scRNA-seq datasets). Partition the ranked list into consecutive bins of 50 ranks and count, per bin, the frequency of high-confidence markers. Tabulate per-bin frequencies with marker-class annotation and rank-bin index, writing the table to marker_enrichment_by_rank.csv. The rationale is that a method recovering biologically relevant DEGs should recover known markers preferentially at the top: in the source study, high-confidence markers (only 2.2% of all genes) made up on average 32.4% of the top 50 ranked DEGs across 136 real transcriptome datasets (Fig. 5).

## Related tools

- **singleCellHaystack** (Produces the ranked DEG list via the `haystack` function (Kullback-Leibler divergence, no clustering), ranked by log.p.vals and accessed with `show_result_haystack`) — github.com/alexisvdb/singleCellHaystack
- **R** (Implementation language: splitting CellMarker markers by publication-count confidence class, binning the ranked list into 50-rank bins, and writing marker_enrichment_by_rank.csv)

## Examples

```
res <- haystack(dat.tsne, dat.expression); ranked <- show_result_haystack(res)
```

## Evaluation signals

- The first rank bin (top 50 ranks) shows a high-confidence marker frequency consistent with the reported ~32.4% average enrichment observed across the 136 analyzed transcriptome datasets
- Marker frequency decreases monotonically (or near-monotonically) toward the tail of the ranked DEG list
- marker_enrichment_by_rank.csv has one row per rank bin per marker class, with marker-class annotation and rank-bin index columns populated
- High-confidence markers constitute only ~2.2% of all genes, so any bin frequency substantially above ~2.2% indicates enrichment over background

## Limitations

- singleCellHaystack p-values can be inflated because the input coordinates (PCs, t-SNE, or UMAP) derive from the same expression data used for DEG prediction, which may affect rank ordering
- The method requires binary detection data; two genes with different expression levels but identical detection patterns receive identical DKL values and p-values, which can affect ranking ties
- CellMarker confidence classes are defined solely by publication counts (>=5 vs 1-4), an external heuristic that may not reflect marker reliability in a specific tissue

## Evidence

- [results] High-confidence CellMarker enrichment among top-ranked DEGs: [evidence span withheld — anchor status: not_found]
- [methods] haystack produces the ranked DEG list: "res <- haystack(dat.tsne, dat.expression)"
- [abstract] singleCellHaystack ranks DEGs without clustering: "Our method uses Kullback-Leibler divergence to find genes that are expressed in subsets of cells that are non-randomly positioned in a multidimensional space"
- [other] Ranking basis is p-value ordering: [evidence span withheld — anchor status: not_found]
- [discussion] Binary detection requirement limits expression-level resolution of ranks: "two genes might be detected in the same subset of cells, but one might have ten-fold higher"
