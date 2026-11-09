---
name: constitutive-exon-psi-validation
description: Use when validating that UMI-cluster misalignment/truncation correction
  improved isoform assignments in single-cell or spatial Nanopore long-read data,
  e.g. by checking VPS28-201 exon-8 read attribution and constitutive-exon PSI consistency
  in the CRCLM sample.
when_to_use_negative:
- Input lacks single-cell/UMI barcodes, so reads cannot be grouped into UMI clusters
  for consensus correction (e.g. bulk long-read data)
- The goal is isoform quantification rather than validating the Step-3 misalignment/truncation
  correction step itself
- Reads are short reads processed by Sicelore2 or FLAMES-style fixed-edit-distance
  UMI collapsing, which this Longcell consensus-based correction does not apply to
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Longcell
  role: R package that constructs the Splice object (creatSplice_from_df), performs
    UMI-cluster consensus correction of misaligned and truncated reads, and computes
    constitutive-exon PSI used for validation
  repo: github.com/yuntianf/Longcell
- name: LongcellPre
  role: Supplies gene strand annotation (gene_bed.rds) needed to build meta splicing
    sites downstream
- name: Nanopore
  role: Long-read sequencing platform producing the 85-99% accuracy reads that require
    UMI-cluster consensus correction
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:1fb28d0dd624f84a323c6bf3d4136203c9602f7e2db41744ff66b9970f6fdd2d
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# constitutive-exon-psi-validation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A validation skill for Longcell's Step-3 UMI-cluster-based misalignment and truncation correction on single-cell Nanopore isoform data, which checks that misaligned reads are reassigned to dominant correctly-mapped isoforms and that constitutive-exon PSI estimates become more consistent across reads.

## When to use

Use when validating that UMI-cluster misalignment/truncation correction improved isoform assignments in single-cell or spatial Nanopore long-read data, e.g. by checking VPS28-201 exon-8 read attribution and constitutive-exon PSI consistency in the CRCLM sample.

## When NOT to use

- Input lacks single-cell/UMI barcodes, so reads cannot be grouped into UMI clusters for consensus correction (e.g. bulk long-read data)
- The goal is isoform quantification rather than validating the Step-3 misalignment/truncation correction step itself
- Reads are short reads processed by Sicelore2 or FLAMES-style fixed-edit-distance UMI collapsing, which this Longcell consensus-based correction does not apply to

## Inputs

- CRCLM single-cell isoform read data frame (e.g. loaded as `iso`) with cell barcodes, UMIs, and alignments
- Gene strand annotation as gene_bed.rds from the LongcellPre package
- Gencode v39-derived splice junction bed (generated from GTF via paftools.js)
- Reads aligned to GRCh38 with minimap2 v2.24

## Outputs

- Corrected read-to-isoform assignments within each UMI cluster
- Per-gene square attribution matrices A
- misalignment_correction_report.csv containing attribution matrices and corrected assignments
- Improved constitutive-exon PSI estimates

## How to apply

Load the single-cell isoform data and gene strand annotation into a Longcell Splice object via creatSplice_from_df, using gene_bed.rds from LongcellPre to build meta splicing sites. For each gene and UMI cluster, rank isoforms by read counts, length, and mapping quality, and select the top-ranked isoform as the cluster representative. Build per-cluster attribution tables, sum them into a square attribution matrix A per gene, and classify isoform i as mis-mapped when A_ii divided by the sum of off-diagonal A_ij is below the default threshold of 2, reassigning its reads to the isoform j with maximum A_ij. Within each UMI cluster, repair truncated ends against complete reads sharing the UMI (meta-isoform grouping tolerates end truncations and internal exons shorter than 100 bp), collapse reads into a consensus alignment, and retain only the majority-vote isoform for clusters still holding multiple isoforms. Judge success by confirming that misaligned reads of VPS28-201 exon 8 are attributed to the dominant correctly-mapped isoform and that constitutive-exon PSI estimates in the CRCLM sample become more consistent across reads, then persist the attribution matrices and corrected assignments to misalignment_correction_report.csv.

## Related tools

- **Longcell** (R package that constructs the Splice object (creatSplice_from_df), performs UMI-cluster consensus correction of misaligned and truncated reads, and computes constitutive-exon PSI used for validation) — github.com/yuntianf/Longcell
- **LongcellPre** (Supplies gene strand annotation (gene_bed.rds) needed to build meta splicing sites downstream)
- **Nanopore** (Long-read sequencing platform producing the 85-99% accuracy reads that require UMI-cluster consensus correction)

## Examples

```
crc_splice = creatSplice_from_df(iso)
```

## Evaluation signals

- Misaligned reads of VPS28-201 exon 8 are attributed to the dominant correctly-mapped isoform after correction
- Constitutive-exon PSI estimates in the CRCLM sample improve, i.e. become more consistent across reads
- An isoform is flagged mis-mapped exactly when A_ii / sum(off-diagonal A_ij) < 2 (default threshold) and its reads move to the isoform j with maximum A_ij
- Truncated read ends are repaired only against complete reads sharing the same UMI, with internal exons shorter than 100 bp tolerated in meta-isoform grouping
- misalignment_correction_report.csv exists and contains the per-gene attribution matrices and corrected assignments

## Limitations

- Nanopore per-base sequencing accuracy of 85-99% makes UMI recovery error-prone, since UMIs are shorter and randomly generated; fixed-edit-distance collapsing (Sicelore2, FLAMES) can overestimate UMIs and inflate quantification
- Accumulated sequencing errors can wrongly map small internal exons, causing false-positive novel isoform discovery, which motivates but also limits the correction
- No changelog was found for the Longcell repository, and the discussion section provides no DOI, formal title, authors, or versioned release information for the tool

## Evidence

- [discussion] Consensus correction among reads assigned to the same UMI (PCR duplicates) to correct read truncation and mapping errors: [evidence span withheld — anchor status: not_found]
