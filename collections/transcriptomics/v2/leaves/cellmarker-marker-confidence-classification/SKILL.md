---
name: cellmarker-marker-confidence-classification
description: Use when you have a ranked DEG list (e.g. singleCellHaystack log.p.vals
  from scRNA-seq data) and want to validate biological relevance by checking whether
  known cell type markers are over-represented among the top ranks rather than uniformly
  distributed.
when_to_use_negative:
- The gene ranking does not come from a DEG prediction method (e.g. it is an unranked
  feature table or a clustering result), since the skill depends on partitioning a
  ranked list into consecutive rank bins.
- No curated marker database with publication-count metadata is available for the
  organism, so the high-/low-confidence split (≥5 vs 1–4 publications) cannot be defined.
- The goal is DEG prediction itself rather than post-hoc validation of an existing
  ranked DEG list — use singleCellHaystack's `haystack` for prediction instead.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: singleCellHaystack
  role: Produces the ranked DEG list (`haystack` + `show_result_haystack`, sorted
    by log.p.vals) whose rank bins are scanned for marker enrichment
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: R
  role: Implementation language for the marker classification, rank-binning, and CSV
    tabulation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-17900-3
    title: A clustering-independent method for finding differentially expressed genes
      in single-cell transcriptome data
  source_content_hash: sha256:d85f9af44ede01e563b95bdeec5b1c908d28f0149c6af90ad216cf865b1ba9bc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - R
  - singleCellHaystack
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 17
---

# cellmarker-marker-confidence-classification

## Summary

Classify CellMarker mouse marker genes into high-confidence (reported in ≥5 publications) and low-confidence (1–4 publications) classes, then quantify how often each class appears in consecutive rank-bins of a singleCellHaystack DEG list to test whether top-ranked DEGs are enriched for known cell type markers.

## When to use

Use when you have a ranked DEG list (e.g. singleCellHaystack log.p.vals from scRNA-seq data) and want to validate biological relevance by checking whether known cell type markers are over-represented among the top ranks rather than uniformly distributed.

## When NOT to use

- The gene ranking does not come from a DEG prediction method (e.g. it is an unranked feature table or a clustering result), since the skill depends on partitioning a ranked list into consecutive rank bins.
- No curated marker database with publication-count metadata is available for the organism, so the high-/low-confidence split (≥5 vs 1–4 publications) cannot be defined.
- The goal is DEG prediction itself rather than post-hoc validation of an existing ranked DEG list — use singleCellHaystack's `haystack` for prediction instead.

## Inputs

- CellMarker mouse marker gene database with per-marker publication counts
- Ranked DEG list from singleCellHaystack's `haystack` function, sorted by log.p.vals (retrievable via `show_result_haystack`)

## Outputs

- marker_enrichment_by_rank.csv: per-rank-bin marker frequencies with marker-class annotation (high- vs low-confidence) and rank-bin index

## How to apply

Download the CellMarker mouse marker database, which records for each marker the number of publications reporting it, and split markers into a high-confidence class (reported in ≥5 publications) and a low-confidence class (1–4 publications). Take the ranked DEG list produced by the singleCellHaystack `haystack` function, sorted by log.p.vals (e.g. via `show_result_haystack`), and partition it into consecutive bins of 50 ranks. Per bin, count the frequency of high-confidence markers, annotate each bin with the marker class and rank-bin index, and write the table to marker_enrichment_by_rank.csv. In the paper this was applied across 136 real transcriptome datasets: high-confidence markers, though only 2.2% of all genes, made up on average 32.4% of the top 50 ranked DEGs (Fig. 5).

## Related tools

- **singleCellHaystack** (Produces the ranked DEG list (`haystack` + `show_result_haystack`, sorted by log.p.vals) whose rank bins are scanned for marker enrichment) — https://github.com/alexisvdb/singleCellHaystack
- **R** (Implementation language for the marker classification, rank-binning, and CSV tabulation)

## Examples

```
library(singleCellHaystack); res <- haystack(dat.tsne, dat.expression); ranked <- show_result_haystack(res); # then bin ranks of 50 and count markers with pub_count >= 5 per bin
```

## Evaluation signals

- The first bin (top 50 ranks) shows a high-confidence marker frequency consistent with the reported ~32.4% enrichment across the 136 analyzed transcriptome datasets.
- Marker frequency in the high-confidence class decreases monotonically (or near-monotonically) toward the tail of the ranked list.
- Overall fraction of high-confidence markers in the genome is ~2.2% of all genes — a much larger top-bin share (e.g. ~32%) confirms enrichment over background.
- Output CSV contains, for every rank bin, both the marker-class annotation and the rank-bin index, with no missing bins.

## Limitations

- The enrichment statistic depends on the quality and publication-bias structure of CellMarker: markers reported in many publications may simply reflect well-studied cell types and genes, not necessarily better markers.
- The 50-rank bin width and the ≥5-publications confidence cutoff are arbitrary analysis choices; results may shift with different binning or cutoffs.
- singleCellHaystack's p-values can be inflated because input coordinates (PCs, t-SNE, UMAP) reuse gene expression variability, which affects the ranking being validated.

## Evidence

- [results] High-confidence CellMarker genes ... made up on average 32.4% of the top 50 ranked DEGs across the 136 datasets: [evidence span withheld — anchor status: not_found]
- [methods] res <- haystack(dat.tsne, dat.expression): "res <- haystack(dat.tsne, dat.expression)"
- [abstract] singleCellHaystack is implemented as an R package and is available from CRAN and GitHub.: "singleCellHaystack is implemented as an R package and is available from CRAN and GitHub."
- [discussion] p-value inflation in singleCellHaystack: "singleCellHaystack suffers from the same issue, because the input coordinates (PCs, t-SNE, or UMAP coordinates) are dimensions containing a large proportion of the variability"
