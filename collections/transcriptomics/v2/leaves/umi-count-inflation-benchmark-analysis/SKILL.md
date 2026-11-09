---
name: umi-count-inflation-benchmark-analysis
description: Use when recovering true UMIs from error-prone Nanopore single-cell long
  reads, or when testing whether a UMI collapsing strategy (e.g. DBSCAN at eps=2)
  inflates UMI counts at high PCR amplification.
when_to_use_negative:
- UMIs come from a short-read platform with per-base accuracy high enough that fixed-edit-distance
  collapsing (e.g. Sicelore2 or FLAMES style) does not inflate counts
- Ground truth is unknown and the goal is downstream isoform analysis (use Longcell's
  quantification pipeline functions like HighExprsGene or getIsoform instead of the
  benchmark)
- Input is already a cell-by-isoform expression matrix — this skill operates upstream
  at the UMI-recovery step
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_3308
tools:
- name: Longcell
  role: Supplies the meta-isoform-group-level iterative UMI clustering procedure that
    tolerates Nanopore sequencing errors
  repo: github.com/yuntianf/Longcell
- name: igraph
  role: R package (v1.5.1) implementing the iterative Louvain algorithm used to split
    the UMI similarity graph into clusters
- name: dbscan
  role: Fixed-edit-distance baseline (eps=2) collapsed on the same simulated samples
    to expose UMI-count inflation
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:8d0c4c4592c86fd84696683e61171204caeea78de0ca2209cabb5a6b6af8e0fa
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
n_claims: 11
---

# umi-count-inflation-benchmark-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark of Longcell's iterative UMI clustering within meta-isoform groups against fixed-edit-distance collapsing (DBSCAN, eps=2), quantifying how UMI-count estimates inflate under Nanopore sequencing error as PCR amplification increases from 2x to 50x.

## When to use

Use when recovering true UMIs from error-prone Nanopore single-cell long reads, or when testing whether a UMI collapsing strategy (e.g. DBSCAN at eps=2) inflates UMI counts at high PCR amplification.

## When NOT to use

- UMIs come from a short-read platform with per-base accuracy high enough that fixed-edit-distance collapsing (e.g. Sicelore2 or FLAMES style) does not inflate counts
- Ground truth is unknown and the goal is downstream isoform analysis (use Longcell's quantification pipeline functions like HighExprsGene or getIsoform instead of the benchmark)
- Input is already a cell-by-isoform expression matrix — this skill operates upstream at the UMI-recovery step

## Inputs

- Simulated adapter 10-mer UMI read samples with sequencing errors, spanning PCR amplification folds 2-50, with ground-truth UMI counts per sample
- Putative UMI sequences grouped by cell and meta-isoform-group (isoforms interconvertible by truncation or short-exon misalignment)
- UMI length l (for edge-weight cutoffs l/2+2 and l/2+4)

## Outputs

- UMI clusters per meta-isoform group and derived UMI count estimates (Longcell)
- Comparison UMI counts from the DBSCAN (eps=2) baseline
- umi_clustering_benchmark.csv tabulating estimated UMI counts per amplification fold for both methods against ground truth

## How to apply

First generate simulated adapter 10-mer UMI read samples carrying Nanopore-style sequencing errors across PCR amplification folds from 2 to 50, retaining ground-truth UMI counts per sample. For each meta-isoform group within each cell, compute pairwise Needleman-Wunsch scores between putative UMIs, build a graph weighted by those scores, and filter edges with weight below l/2+2 for UMI length l. Apply the iterative Louvain algorithm (igraph v1.5.1), recursively splitting subgraphs until each is highly connected (min_cut > 3) or the minimum edge weight exceeds l/2+4, yielding UMI clusters and thus UMI counts; run DBSCAN with eps=2 on the same samples as the fixed-distance baseline. Tabulate estimated counts per amplification fold for both methods against ground truth and write umi_clustering_benchmark.csv; the rationale is that fixed edit-distance collapsing overestimates UMIs under high sequencing error, while Longcell's error-tolerant clustering stays stable.

## Related tools

- **Longcell** (Supplies the meta-isoform-group-level iterative UMI clustering procedure that tolerates Nanopore sequencing errors) — github.com/yuntianf/Longcell
- **igraph** (R package (v1.5.1) implementing the iterative Louvain algorithm used to split the UMI similarity graph into clusters)
- **dbscan** (Fixed-edit-distance baseline (eps=2) collapsed on the same simulated samples to expose UMI-count inflation)

## Examples

```
library(igraph); g <- graph_from_adjacency_matrix(W, mode="undirected", weighted=TRUE); clusters <- cluster_louvain(g)
```

## Evaluation signals

- Longcell's estimated UMI counts remain stable across amplification folds 2-50, with no inflation from UMI scattering, matching ground truth
- DBSCAN (eps=2) baseline overestimates UMI numbers at high sequencing error, reproducing the reported Fig. 1B trend
- Edge filtering thresholds are respected: edges below l/2+2 removed, iterative splitting stops when min_cut > 3 or minimum edge weight exceeds l/2+4
- umi_clustering_benchmark.csv contains per-fold estimated counts for both methods with a ground-truth column present

## Limitations

- Fixed-edit-distance methods (Sicelore2, FLAMES, DBSCAN eps=2) overestimate UMI counts under high sequencing error, causing inflated quantification
- UMI recovery is more vulnerable to Nanopore errors than cell barcode recovery because UMIs are shorter, randomly generated, and lack a whitelist
- Benchmark relies on simulated adapter 10-mer UMIs; behavior on real 10x-pattern UMIs within meta-isoform groups depends on the truncation/misalignment grouping assumption

## Evidence

- [discussion] consensus among PCR-duplicate reads: [evidence span withheld — anchor status: not_found]
- [readme] devtools install of Longcell: [evidence span withheld — anchor status: external_doc]
