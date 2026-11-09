---
name: dbscan-baseline-umi-count-comparison
description: 'Use when benchmarking UMI collapsing on Nanopore single-cell reads:
  comparing an error-tolerant graph-clustering UMI counter against fixed-edit-distance
  DBSCAN (eps=2) to quantify UMI-count inflation at high PCR amplification or error
  rates.'
when_to_use_negative:
- Input is short-read scRNA-seq with whitelist-validated cell barcodes and low-error
  UMIs, where fixed-edit-distance collapsing is adequate
- UMI sequences are whitelisted or fixed-design rather than randomly generated, so
  sequencing-error-driven scattering is not a concern
- The goal is downstream alternative splicing analysis (differential splicing, PSI
  estimation) rather than UMI recovery benchmarking
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: dbscan
  role: Fixed-edit-distance (eps=2) baseline UMI clustering method used to quantify
    UMI-count inflation under high sequencing error
- name: igraph
  role: R package (v1.5.1) providing the iterative Louvain algorithm and min-cut criterion
    for Longcell's error-tolerant UMI clustering, the method being benchmarked
  repo: github.com/yuntianf/Longcell
- name: Longcell
  role: Pipeline whose Needleman-Wunsch graph-based iterative UMI clustering is compared
    against the DBSCAN baseline
  repo: github.com/yuntianf/Longcell
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:10a775c5799b157eac3c9c3fb6166b9c3fb47094065bd9ffd3d4bfcffe1a879b
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# dbscan-baseline-umi-count-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking pattern that runs fixed-edit-distance DBSCAN (eps=2) UMI collapsing as a baseline against Longcell's error-tolerant iterative Louvain clustering, on simulated Nanopore adapter 10-mer UMI reads across 2-50x PCR amplification, to expose the UMI-count inflation that fixed-distance methods exhibit under high sequencing error.

## When to use

Use when benchmarking UMI collapsing on Nanopore single-cell reads: comparing an error-tolerant graph-clustering UMI counter against fixed-edit-distance DBSCAN (eps=2) to quantify UMI-count inflation at high PCR amplification or error rates.

## When NOT to use

- Input is short-read scRNA-seq with whitelist-validated cell barcodes and low-error UMIs, where fixed-edit-distance collapsing is adequate
- UMI sequences are whitelisted or fixed-design rather than randomly generated, so sequencing-error-driven scattering is not a concern
- The goal is downstream alternative splicing analysis (differential splicing, PSI estimation) rather than UMI recovery benchmarking

## Inputs

- Simulated adapter 10-mer UMI read sets with Nanopore sequencing errors, sampled across PCR amplification folds 2-50, with ground-truth UMI counts per sample
- Putative recovered UMI sequences grouped per cell per meta-isoform group (UMI length l)
- Pairwise Needleman-Wunsch similarity scores between UMIs within each meta-isoform group

## Outputs

- umi_clustering_benchmark.csv: estimated UMI counts per amplification fold for Longcell (iterative Louvain) vs DBSCAN (eps=2) vs ground truth
- Per-sample UMI cluster assignments from each method

## How to apply

Generate or obtain simulated UMI read samples (adapter 10-mers with Nanopore-like sequencing errors) sampled at amplification folds from 2 to 50, retaining ground-truth UMI counts per sample. For the error-tolerant method, first group putative UMIs within each cell at the meta-isoform-group level (isoforms interconvertible by truncation or short-exon misalignment), compute pairwise Needleman-Wunsch similarity between UMIs, build a UMI graph with edges filtered below l/2+2 (UMI length l), and run the iterative Louvain algorithm (R igraph v1.5.1), recursively splitting subgraphs until min_cut > 3 or the minimum edge weight exceeds l/2+4. Run DBSCAN with eps=2 as the fixed-edit-distance baseline on the same simulated samples. Tabulate estimated UMI counts per amplification fold for both methods against ground truth and write umi_clustering_benchmark.csv.

## Related tools

- **dbscan** (Fixed-edit-distance (eps=2) baseline UMI clustering method used to quantify UMI-count inflation under high sequencing error)
- **igraph** (R package (v1.5.1) providing the iterative Louvain algorithm and min-cut criterion for Longcell's error-tolerant UMI clustering, the method being benchmarked) — github.com/yuntianf/Longcell
- **Longcell** (Pipeline whose Needleman-Wunsch graph-based iterative UMI clustering is compared against the DBSCAN baseline) — github.com/yuntianf/Longcell

## Examples

```
library(dbscan); clusters <- dbscan::dbscan(umi_dist_matrix, eps = 2)  # baseline UMI collapsing on simulated 10-mer UMI distances
```

## Evaluation signals

- Longcell's estimated UMI counts remain stable across amplification folds 2-50 (no inflation from UMI scattering), tracking ground truth
- The DBSCAN (eps=2) baseline overestimates UMI numbers at high sequencing error, reproducing the inflation trend reported in Fig. 1B
- umi_clustering_benchmark.csv contains one estimated-vs-ground-truth count pair per amplification fold for both methods
- Graph edge filtering threshold equals l/2+2 and the recursive split stopping condition (min_cut > 3, min edge weight > l/2+4) is respected
- Both methods are run on the identical simulated sample set so counts are directly comparable per fold

## Limitations

- The DBSCAN inflation is demonstrated on simulated adapter 10-mer UMI samples with controlled error profiles; real Nanopore UMI recovery may show different error structure
- DBSCAN eps=2 is a fixed-distance heuristic; alternative eps values or density parameters are not evaluated
- Results depend on correct prior grouping of UMIs at the meta-isoform-group level (isoforms interconvertible by truncation or short-exon misalignment)

## Evidence

- [other] Fixed-distance baseline definition: [evidence span withheld — anchor status: not_found]
- [other] Error-tolerant comparison method parameters: [evidence span withheld — anchor status: not_found]
