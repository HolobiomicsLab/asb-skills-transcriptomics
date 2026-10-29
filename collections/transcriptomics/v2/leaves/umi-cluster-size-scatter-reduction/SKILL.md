---
name: umi-cluster-size-scatter-reduction
description: Use when quantifying isoforms from single-cell or spatially barcoded
  Nanopore long reads with high per-base error (85-99%), where UMI scattering inflates
  UMI counts and fixed-edit-distance collapsing would overestimate expression.
when_to_use_negative:
- Data with low sequencing error (e.g., short-read Illumina UMI data), where fixed-edit-distance
  UMI collapsing suffices and adaptive pruning could remove true small clusters
- Reads lacking recoverable cell barcodes/UMIs, since the pruning operates on per-isoform
  UMI cluster sizes within cells
- Already-deduplicated UMI count matrices, where no cluster-size distribution is available
  to prune
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: Longcell
  role: Implements the UMI clustering (Louvain on Needleman-Wunsch similarity graph)
    and the scatter-reduction pruning of small clusters during isoform quantification
  repo: github.com/yuntianf/Longcell
- name: Nanopore
  role: Source platform producing the long reads whose high error rate motivates the
    pruning; reads are aligned with minimap2 spliced mode against GRCh38
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:64fbedcfed5e7d8276592392c57b3b2df12a01779e80e64c3392cc14c5d75418
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
  - Nanopore
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# umi-cluster-size-scatter-reduction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Longcell's adaptive UMI-cluster pruning step: within each isoform in each cell, UMI clusters are ranked by size and the smallest clusters (e.g., singletons) are iteratively removed as error-driven scatter, because sequencing errors inflate UMI counts in Nanopore long-read quantification.

## When to use

Use when quantifying isoforms from single-cell or spatially barcoded Nanopore long reads with high per-base error (85-99%), where UMI scattering inflates UMI counts and fixed-edit-distance collapsing would overestimate expression.

## When NOT to use

- Data with low sequencing error (e.g., short-read Illumina UMI data), where fixed-edit-distance UMI collapsing suffices and adaptive pruning could remove true small clusters
- Reads lacking recoverable cell barcodes/UMIs, since the pruning operates on per-isoform UMI cluster sizes within cells
- Already-deduplicated UMI count matrices, where no cluster-size distribution is available to prune

## Inputs

- Nanopore long reads with recovered cell barcodes and UMIs (barcode/UMI assignment from the LongcellPre sliding-window adapter search and k-mer dictionary alignment)
- UMI clusters per isoform per cell, built by Louvain clustering on the Needleman-Wunsch similarity graph (edge cutoff l/2+2, min_cut > 3)
- Needleman-Wunsch score distribution of 12mer adapters for estimating the singleton ratio r

## Outputs

- Pruned per-isoform, per-cell UMI counts (m-p)
- Estimated error-driven singleton ratio r
- Pruned UMI cluster-size distribution and QC plots (e.g., pruned UMI count vs. GC-dependent amplification fold A = k*sigmoid(0.5*(G-50)) + 2, k=20)

## How to apply

First estimate the error-driven singleton ratio r from the Needleman-Wunsch score distribution of the 12mer adapters upstream of confidently identified cell barcodes, counting adapters scoring below the cutoff k = l/2+2. Then, for each isoform within each cell, rank the UMI cluster sizes s1...sm in increasing order and iteratively remove the smallest cluster until the cumulative removed size satisfies the balance condition (removed sum >= r * remaining sum / (1-r)); the pruned UMI count is reported as m-p. The rationale is that within an isoform, true UMI cluster sizes should be comparable, so clusters in the left tail of the empirical size distribution are attributable to UMI scattering from sequencing errors rather than distinct transcripts. Unlike Sicelore2/FLAMES-style fixed edit-distance collapsing, this adaptive pruning scales removal to the estimated error rate per library.

## Related tools

- **Longcell** (Implements the UMI clustering (Louvain on Needleman-Wunsch similarity graph) and the scatter-reduction pruning of small clusters during isoform quantification) — github.com/yuntianf/Longcell
- **Nanopore** (Source platform producing the long reads whose high error rate motivates the pruning; reads are aligned with minimap2 spliced mode against GRCh38)

## Examples

```
devtools::install_github("yuntianf/Longcell",dependencies=TRUE); library(Longcell)
```

## Evaluation signals

- The pruned Nanopore UMI cluster-size distribution of a single-isoform gene matches the matched Illumina UMI cluster-size distribution (Supplementary Fig. 5C)
- Pruned UMI counts remain stable across GC-dependent PCR amplification folds A = k*sigmoid(0.5*(G-50)) + 2 (k=20) across meta-isoform groups (Fig. 1B evidence)
- The stopping rule is respected: removal continues only until the cumulative removed size >= r * remaining sum / (1-r); report the resulting m-p
- The estimated singleton ratio r is recorded and reproducible from the 12mer adapter NW score distribution below k = l/2+2
- Pruned counts are lower than or equal to unpruned cluster counts, and no larger cluster is removed before all smaller ones (sizes ranked increasingly)

## Limitations

- Pruning depends on the estimated singleton ratio r; a misestimated r (from the 12mer adapter NW score distribution) can over- or under-prune genuine small UMI clusters
- UMI recovery is inherently more vulnerable to sequencing errors than cell barcodes because UMIs are shorter and randomly generated, so upstream recovery errors propagate into cluster sizes
- Existing fixed-edit-distance approaches (Sicelore2, FLAMES) overestimate UMIs under high error; this skill is calibrated for high-error Nanopore data, not a general deduplication method

## Evidence

- [full_text] Longcell ranks UMI clusters by size within each isoform and adaptively prunes those in the left tail of the empirical distribution (e.g., singletons), since small clusters are likely due to errors.: [evidence span withheld — anchor status: not_found]
- [readme] devtools::install_github("yuntianf/Longcell",dependencies=TRUE): [evidence span withheld — anchor status: external_doc]
