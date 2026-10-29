---
name: single-cell-nanopore-error-correction
description: Use when single-cell or spatially barcoded Nanopore long-read alignments
  suffer from read truncation, misaligned small internal exons, or sequencing-error-driven
  false-positive novel isoforms that must be corrected before isoform quantification.
when_to_use_negative:
- Input is short-read sequencing data or data without UMIs — the consensus correction
  relies on reads sharing a UMI being PCR duplicates
- The goal is de novo novel isoform discovery from raw alignments without error correction
  — misaligned small internal exons here produce false-positive novel isoforms rather
  than reliable discoveries
- Downstream analysis only needs fixed-edit-distance UMI collapsing (e.g., Sicelore2,
  FLAMES style) — those methods overestimate UMIs under high sequencing error and
  are the approach this step replaces
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3320
tools:
- name: Longcell
  role: R package implementing UMI-cluster-based consensus correction of misaligned
    and truncated reads for isoform quantification
  repo: github.com/yuntianf/Longcell
- name: LongcellPre
  role: Upstream preprocessing package whose gene_bed.rds provides gene strand information
    needed for meta splicing sites
  repo: github.com/yuntianf/Longcell
- name: Nanopore
  role: Sequencing instrument producing the long reads (85–99% per-base accuracy)
    that this correction step addresses
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:238e9cd6b562f24e2e526bdde0447f7f4ef656c6715ec54eb613d94bea089ab6
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
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3320
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# single-cell-nanopore-error-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Longcell's Step 3 consensus-correction approach for single-cell Nanopore long reads: within each UMI-based read cluster (PCR duplicates), reads are collapsed into a consensus alignment and misaligned or truncated isoform reads are reassigned, improving isoform quantification accuracy.

## When to use

Use when single-cell or spatially barcoded Nanopore long-read alignments suffer from read truncation, misaligned small internal exons, or sequencing-error-driven false-positive novel isoforms that must be corrected before isoform quantification.

## When NOT to use

- Input is short-read sequencing data or data without UMIs — the consensus correction relies on reads sharing a UMI being PCR duplicates
- The goal is de novo novel isoform discovery from raw alignments without error correction — misaligned small internal exons here produce false-positive novel isoforms rather than reliable discoveries
- Downstream analysis only needs fixed-edit-distance UMI collapsing (e.g., Sicelore2, FLAMES style) — those methods overestimate UMIs under high sequencing error and are the approach this step replaces

## Inputs

- Single-cell or spatially barcoded Nanopore long-read isoform alignments (e.g., CRCLM sample reads aligned to GRCh38 with minimap2 v2.24)
- Gene strand annotation / gene_bed.rds from the LongcellPre package (splice junction bed derived from Gencode v39 GTF via paftools.js)
- Recovered cell barcode and UMI assignments per read (from Step 1/2 of the Longcell pipeline)

## Outputs

- Per-gene square attribution matrix A (counts of reads of each isoform attributed to each cluster representative)
- Corrected per-read isoform assignments after misalignment reassignment and truncation repair
- Consensus alignments per UMI cluster (e.g., written to misalignment_correction_report.csv)

## How to apply

Load the single-cell isoform alignments and gene strand annotation into a Longcell Splice object via creatSplice_from_df, using the gene_bed.rds from LongcellPre. For each gene and UMI cluster, rank isoforms by read counts, length, and mapping quality and pick the top-ranked isoform as the cluster representative. Build per-cluster attribution tables of reads of each isoform attributed to each representative, sum them into a square attribution matrix A per gene, classify isoform i as mis-mapped when A_ii divided by the sum of off-diagonal A_ij is below the default threshold of 2, and reassign its reads to isoform j with maximal A_ij — the paper designates the isoform with smaller bulk proportion as misalignment. Within each UMI cluster, repair truncated ends against complete reads sharing the UMI (meta-isoform grouping tolerates end truncations and internal exons shorter than 100 bp), collapse reads into a consensus alignment, and keep the majority-vote isoform for clusters still holding multiple isoforms. The rationale is that reads sharing a UMI are PCR duplicates, so a 'consensus' among them corrects truncation and mapping errors accumulated in Nanopore reads (85–99% per-base accuracy).

## Related tools

- **Longcell** (R package implementing UMI-cluster-based consensus correction of misaligned and truncated reads for isoform quantification) — github.com/yuntianf/Longcell
- **LongcellPre** (Upstream preprocessing package whose gene_bed.rds provides gene strand information needed for meta splicing sites) — github.com/yuntianf/Longcell
- **Nanopore** (Sequencing instrument producing the long reads (85–99% per-base accuracy) that this correction step addresses)

## Examples

```
library(Longcell); crc_splice = creatSplice_from_df(iso)
```

## Evaluation signals

- Attribution matrix diagnostic: mis-mapped isoforms have A_ii / sum(off-diagonal A_ij) below the default threshold of 2; correctly mapped isoforms exceed it
- Corrected output attributes misaligned reads (e.g., VPS28-201 exon 8) to the dominant correctly-mapped isoform
- Constitutive-exon PSI estimates in the CRCLM sample become more consistent across reads after correction
- No false-positive novel isoforms remain from wrongly mapped small internal exons (< 100 bp tolerated in meta-isoform grouping)
- Truncated read ends within each UMI cluster match the complete reads sharing that UMI after repair

## Limitations

- High Nanopore per-base sequencing error (85–99%) makes UMI recovery vulnerable, since UMIs are shorter and randomly generated without a whitelist; residual UMI errors propagate into cluster construction
- Existing fixed-edit-distance UMI collapsing methods (Sicelore2, FLAMES) overestimate UMI counts under high sequencing error, so upstream UMI collapsing choice affects the correction
- The influence of sequencing errors, read truncation, and mapping errors on single-cell long-read isoform quantification had not been characterized prior to this study, so edge-case behaviors may be unknown

## Evidence

- [discussion] we correct for read truncation and mapping errors by taking a "consensus" among reads assigned to the same UMI: [evidence span withheld — anchor status: not_found]
