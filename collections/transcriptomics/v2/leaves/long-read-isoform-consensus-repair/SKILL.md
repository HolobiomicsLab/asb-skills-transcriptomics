---
name: long-read-isoform-consensus-repair
description: Use when Nanopore single-cell/spatial long reads are grouped into UMI
  clusters and you must fix misalignment or end truncation before isoform quantification,
  e.g. false-positive small internal exons or truncated read ends from high sequencing
  error.
when_to_use_negative:
- Reads lack UMI/PCR-duplicate structure (no clusters to take a consensus over), e.g.
  bulk long-read data without cellular barcodes
- You are doing downstream alternative-splicing statistics (phi/psi heterogeneity
  analysis) rather than pre-processing — Longcell steps 3–4 upstream outputs should
  already exist
- UMI collapsing with fixed edit distance is already trusted (e.g. Sicelore2/FLAMES
  output) and no misalignment/truncation correction is needed
edam_operation: http://edamontology.org/operation_0292
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3168
tools:
- name: Longcell
  role: R package implementing UMI-cluster consensus collapse, misalignment correction
    via attribution matrices, and truncation repair
  repo: github.com/yuntianf/Longcell
- name: LongcellPre
  role: Upstream producer of gene strand annotation (gene_bed.rds) needed to build
    meta splicing sites
  repo: github.com/yuntianf/Longcell
- name: Nanopore
  role: Sequencing instrument producing the long reads whose low per-base accuracy
    (85–99%) necessitates consensus repair
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:1126720722e340a5b3c8192d434a1765504df4c9ca842d53ca0e64f4d836283f
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
  - LongcellPre
  - Nanopore
  edam_operation: http://edamontology.org/operation_0292
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# long-read-isoform-consensus-repair

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Longcell's UMI-cluster consensus correction (workflow step 3): within each UMI-based read cluster of Nanopore long reads, collapse PCR-duplicate reads into a consensus alignment that repairs truncated ends and mis-mapped isoforms, retaining the majority-vote isoform as the cluster representative.

## When to use

Use when Nanopore single-cell/spatial long reads are grouped into UMI clusters and you must fix misalignment or end truncation before isoform quantification, e.g. false-positive small internal exons or truncated read ends from high sequencing error.

## When NOT to use

- Reads lack UMI/PCR-duplicate structure (no clusters to take a consensus over), e.g. bulk long-read data without cellular barcodes
- You are doing downstream alternative-splicing statistics (phi/psi heterogeneity analysis) rather than pre-processing — Longcell steps 3–4 upstream outputs should already exist
- UMI collapsing with fixed edit distance is already trusted (e.g. Sicelore2/FLAMES output) and no misalignment/truncation correction is needed

## Inputs

- Single-cell Nanopore long-read BAM/isoform alignment table with recovered cell barcodes and UMIs (CRCLM sample used as example)
- Gene strand annotation (gene_bed.rds from LongcellPre, derived from Gencode v39 GTF)
- Per-gene, per-UMI-cluster read-to-isoform assignments

## Outputs

- Per-gene square attribution matrix A of isoform-to-representative read counts
- Corrected per-cluster consensus alignments with repaired ends
- Majority-vote isoform assignment per UMI cluster (e.g. exported to misalignment_correction_report.csv)

## How to apply

Load the single-cell isoform data and gene strand annotation (gene_bed.rds from LongcellPre) into a Splice object via creatSplice_from_df. For each gene and UMI cluster, rank isoforms by read counts, length, and mapping quality to choose a cluster representative. Build per-cluster attribution tables counting reads of each isoform attributed to each representative, summed across UMI clusters and cells into a square attribution matrix A per gene; classify isoform i as mis-mapped when A_ii / sum(off-diagonal A_ij) falls below the default threshold of 2, and reassign its reads to the isoform j with maximum A_ij (rationale: the isoform with the smaller bulk proportion co-occurring in clusters is designated misalignment). Within each UMI cluster, repair truncated ends by comparison to complete reads sharing the UMI (meta-isoform grouping tolerates end truncations and internal exons shorter than 100 bp), collapse reads into a consensus alignment, and keep only the majority-vote isoform for clusters still holding multiple isoforms.

## Related tools

- **Longcell** (R package implementing UMI-cluster consensus collapse, misalignment correction via attribution matrices, and truncation repair) — github.com/yuntianf/Longcell
- **LongcellPre** (Upstream producer of gene strand annotation (gene_bed.rds) needed to build meta splicing sites) — github.com/yuntianf/Longcell
- **Nanopore** (Sequencing instrument producing the long reads whose low per-base accuracy (85–99%) necessitates consensus repair)

## Examples

```
crc_splice = creatSplice_from_df(iso)
```

## Evaluation signals

- Misaligned reads such as VPS28-201 exon 8 are attributed to the dominant correctly-mapped isoform after correction
- Constitutive-exon PSI estimates in the CRCLM sample become more consistent across reads after correction
- Every isoform reassignment follows the rule A_ii / sum(A_ij, j != i) < 2 with reassignment to argmax A_ij
- Each UMI cluster collapses to a single consensus alignment; clusters retaining multiple isoforms keep only the majority-vote isoform
- Attribution matrices and corrected assignments are written to the report file (misalignment_correction_report.csv)

## Limitations

- High Nanopore per-base sequencing error (85–99%) can wrongly map small internal exons, causing false-positive novel isoforms; the 100 bp internal-exon tolerance and threshold-2 rule are heuristics that may not generalize
- Fixed-edit-distance UMI collapsing (as in Sicelore2/FLAMES) overestimates UMI counts under high error, so upstream UMI recovery quality limits this correction
- Influence of sequencing errors, read truncation, and mapping errors on single-cell long-read quantification was not characterized prior to this study, so no external benchmarks exist

## Evidence

- [discussion] we correct for read truncation and mapping errors by taking a "consensus" among reads assigned to the same UMI, which correspond to PCR duplicates: [evidence span withheld — anchor status: not_found]
