---
name: graph-based-iterative-louvain-umi-clustering
description: Use when quantifying isoforms from single-cell/spatial Nanopore long
  reads whose UMIs carry high sequencing error, and fixed edit-distance collapsing
  (e.g., Sicelore2, FLAMES, DBSCAN) inflates UMI counts at high PCR amplification.
when_to_use_negative:
- Input is short-read data with whitelisted cell barcodes and low-error UMIs, where
  fixed edit-distance collapsing (Sicelore2/FLAMES-style) suffices
- UMIs exceed the error model's length assumptions — thresholds l/2+2 and l/2+4 are
  defined per UMI length l and may not generalize to much longer random barcodes
- Reads lack UMI or cell-barcode structure entirely, since grouping is performed within
  each cell at the meta-isoform-group level
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3308
tools:
- name: Longcell
  role: Pipeline implementing meta-isoform grouping, Needleman-Wunsch graph construction,
    edge filtering, and iterative UMI clustering
  repo: github.com/yuntianf/Longcell
- name: igraph
  role: R package (v1.5.1) providing the iterative Louvain community-detection algorithm
    and min-cut checks used to split the UMI similarity graph
- name: dbscan
  role: Fixed-edit-distance baseline (eps=2) run on the same simulated samples to
    demonstrate UMI-count inflation at high sequencing error
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:f985db141d062cbeba12d526b7cda80134e9d7b13ff65c336e0565ed4be3012e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/yuntianf/Longcell
  tools:
  - Longcell
  - dbscan
  - igraph
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 4
---

# graph-based-iterative-louvain-umi-clustering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Longcell's Step-2 UMI error-correction: within each cell and meta-isoform group, putative UMIs are clustered via an iterative Louvain procedure on a Needleman-Wunsch similarity graph, avoiding the UMI-count inflation that fixed-edit-distance methods like DBSCAN (eps=2) show under Nanopore sequencing errors.

## When to use

Use when quantifying isoforms from single-cell/spatial Nanopore long reads whose UMIs carry high sequencing error, and fixed edit-distance collapsing (e.g., Sicelore2, FLAMES, DBSCAN) inflates UMI counts at high PCR amplification.

## When NOT to use

- Input is short-read data with whitelisted cell barcodes and low-error UMIs, where fixed edit-distance collapsing (Sicelore2/FLAMES-style) suffices
- UMIs exceed the error model's length assumptions — thresholds l/2+2 and l/2+4 are defined per UMI length l and may not generalize to much longer random barcodes
- Reads lack UMI or cell-barcode structure entirely, since grouping is performed within each cell at the meta-isoform-group level

## Inputs

- Putative UMI sequences recovered per cell from Nanopore long reads (e.g., adapter 10-mer UMIs) with associated meta-isoform-group assignments
- UMI length l per group (used for Needleman-Wunsch edge-weight thresholds l/2+2 and l/2+4)
- Optional benchmark input: simulated UMI read samples with sequencing errors across PCR amplification folds 2-50 and ground-truth UMI counts

## Outputs

- UMI clusters per meta-isoform group per cell, each cluster representing one corrected true UMI
- Cell-by-isoform UMI-count estimates used for downstream isoform quantification
- Benchmark table (e.g., umi_clustering_benchmark.csv) of estimated UMI counts per amplification fold vs ground truth and vs DBSCAN baseline

## How to apply

For each cell, first group putative UMIs at the meta-isoform-group level, where isoforms are interconvertible by truncation or short-exon misalignment. Compute pairwise Needleman-Wunsch scores between UMIs as similarity, build a graph with UMIs as nodes and scores as edge weights, and filter out edges with weight below l/2+2 for UMI length l. Apply the iterative Louvain algorithm (R igraph v1.5.1), recursively splitting subgraphs until each is highly connected (min_cut > 3) or the minimum edge weight exceeds l/2+4; each final cluster yields one true UMI and its read count yields the UMI-level expression. Validate on simulated UMI samples (e.g., adapter 10-mers at 2-50x amplification with ground-truth counts) where estimated counts must stay stable across amplification folds, unlike the DBSCAN (eps=2) baseline which overestimates UMIs at high sequencing error.

## Related tools

- **Longcell** (Pipeline implementing meta-isoform grouping, Needleman-Wunsch graph construction, edge filtering, and iterative UMI clustering) — github.com/yuntianf/Longcell
- **igraph** (R package (v1.5.1) providing the iterative Louvain community-detection algorithm and min-cut checks used to split the UMI similarity graph)
- **dbscan** (Fixed-edit-distance baseline (eps=2) run on the same simulated samples to demonstrate UMI-count inflation at high sequencing error)

## Examples

```
library(igraph); com <- cluster_louvain(g, weights = E(g)$nw_score)  # g: UMI Needleman-Wunsch graph, edges >= l/2+2
```

## Evaluation signals

- Estimated UMI counts remain stable across simulated PCR amplification folds 2-50 with no inflation (no UMI scattering), matching the Fig. 1B trend
- DBSCAN (eps=2) baseline on the same samples overestimates UMI numbers at high sequencing error, confirming the contrast
- Iterative splitting terminates: every subgraph either has min_cut > 3 or its minimum edge weight exceeds l/2+4
- All retained graph edges have Needleman-Wunsch weight at or above l/2+2 for UMI length l
- UMI counts derived from final clusters are consistent with ground-truth counts on simulated adapter 10-mer samples

## Limitations

- Fixed edit-distance baselines overestimate UMIs when sequencing error is high; conversely the Needleman-Wunsch/Louvain approach is tuned to Nanopore error rates (85-99% per-base accuracy) and its thresholds may not transfer to other platforms
- UMI recovery is intrinsically more error-prone than cell-barcode recovery because UMIs are shorter and randomly generated with no whitelist
- UMI correction operates within meta-isoform groups; grouping depends on upstream isoform calls where read truncation and misaligned small internal exons can create false-positive novel isoforms

## Evidence
