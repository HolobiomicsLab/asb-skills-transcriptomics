---
name: umi-based-misalignment-correction
description: Use when Nanopore long reads aligned with minimap2 yield spurious isoforms
  from wrongly mapped small internal exons (e.g., VPS28-201) or truncated ends, and
  read-level counts must be corrected within UMI clusters before isoform quantification.
when_to_use_negative:
- Data are short reads or full-length Illumina-based single-cell data where UMI error
  rates do not require long-read consensus correction
- UMIs have not been recovered or are unreliable, since the correction depends on
  grouping PCR duplicates sharing the same UMI
- The goal is de novo isoform discovery where wrongly mapped small internal exons
  should be flagged rather than reassigned to annotated isoforms
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3524
- http://edamontology.org/topic_3170
tools:
- name: Longcell
  role: Implements the UMI-cluster attribution-matrix correction, truncation repair,
    and consensus collapse (creatSplice_from_df and downstream Splice-object functions)
  repo: github.com/yuntianf/Longcell
- name: Nanopore
  role: Source of long reads whose per-read spliced alignments carry the misalignment
    and truncation errors being corrected
- name: dplyr
  role: Used in the Longcell R workflow to tally and reshape read-level splice-site
    counts for PSI computation
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:abc04ff35d4853d7192016b22b3e5ef950db67224237c6bac38d3c66a4fac83d
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
  - dplyr
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3524
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# umi-based-misalignment-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Longcell's UMI-cluster-based correction step that detects isoforms frequently co-occurring within the same UMI cluster, designates the lower-bulk-proportion isoform as misalignment (A_ii / sum_{i≠j} A_ij < 2), and reassigns or consensus-corrects those reads to fix mapping and truncation errors in Nanopore single-cell isoform quantification.

## When to use

Use when Nanopore long reads aligned with minimap2 yield spurious isoforms from wrongly mapped small internal exons (e.g., VPS28-201) or truncated ends, and read-level counts must be corrected within UMI clusters before isoform quantification.

## When NOT to use

- Data are short reads or full-length Illumina-based single-cell data where UMI error rates do not require long-read consensus correction
- UMIs have not been recovered or are unreliable, since the correction depends on grouping PCR duplicates sharing the same UMI
- The goal is de novo isoform discovery where wrongly mapped small internal exons should be flagged rather than reassigned to annotated isoforms

## Inputs

- Read-level splice-site / isoform assignment table from minimap2 v2.24 spliced alignments to GRCh38 (Gencode v39 GTF-derived splice junction bed)
- Recovered cell barcode and UMI assignments per read from Longcell's adapter search and polyA detection
- Gene strand annotation (gene_bed.rds from LongcellPre)
- Optional: pre-correction PSI tallies (spliced-in vs gene counts per internal exon)

## Outputs

- Per-gene square attribution matrices A (isoform-to-representative read counts summed over UMI clusters and cells)
- Corrected per-read isoform assignments (mis-mapped reads reassigned to dominant isoform j)
- Consensus alignments per UMI cluster with repaired truncated ends
- Post-correction PSI estimates and a misalignment_correction_report.csv

## How to apply

Build a Splice object from the read-level splice-site table with creatSplice_from_df, carrying gene strand information from the LongcellPre gene_bed.rds. For each gene and UMI cluster, order isoforms by read counts, length, and mapping quality to pick a cluster representative; build per-cluster attribution tables that count reads of each isoform attributed to each representative, and sum them across all UMI clusters and cells into a square attribution matrix A per gene. Flag isoform i as mis-mapped when A_ii divided by the sum of off-diagonal A_ij falls below the default threshold of 2, then reassign its reads to the isoform j with maximal A_ij — the rationale is that isoforms which frequently coexist in the same UMI cluster but differ greatly in bulk proportion indicate sequencing-error-driven misalignment. Within each UMI cluster (PCR duplicates), repair truncated ends against complete reads sharing the UMI (meta-isoform grouping tolerates end truncations and internal exons shorter than 100 bp), collapse reads into a consensus alignment, and retain the majority-vote isoform for clusters still holding multiple isoforms. On the CRCLM Nanopore sample this lifts average PSI for constitutive internal exons (read count > 30) from 0.67 to 0.92 and removes the ~20% pre-correction fraction of VPS28-201 reads wrongly mapped at its small middle exon (exon 8).

## Related tools

- **Longcell** (Implements the UMI-cluster attribution-matrix correction, truncation repair, and consensus collapse (creatSplice_from_df and downstream Splice-object functions)) — github.com/yuntianf/Longcell
- **Nanopore** (Source of long reads whose per-read spliced alignments carry the misalignment and truncation errors being corrected)
- **dplyr** (Used in the Longcell R workflow to tally and reshape read-level splice-site counts for PSI computation)

## Examples

```
library(Longcell); crc_splice = creatSplice_from_df(iso)
```

## Evaluation signals

- Average bulk PSI for constitutive internal exons (read count > 30) in the CRCLM sample shifts from ~0.67 pre-correction to ~0.92 post-correction
- Pre-correction fraction of VPS28-201 reads containing the falsely mapped small middle exon is ~20%, and post-correction those reads are attributed to the dominant correctly-mapped isoform
- For each flagged isoform i, A_ii / sum_{i≠j} A_ij < 2 before reassignment, and reassigned reads go to the j with maximum A_ij
- Corrected output contains only majority-vote isoforms per UMI cluster, with truncated ends repaired against complete reads sharing the UMI
- Attribution matrices and corrected assignments round-trip into misalignment_correction_report.csv with consistent row/column isoform labels

## Limitations

- Misalignment classification assumes the higher-bulk isoform is correct; the isoform with the smaller bulk proportion is designated as misalignment, which could mislabel genuinely low-abundance co-clustered isoforms
- Low Nanopore per-base accuracy (85–99%) makes UMI recovery itself error-prone, so clusters may be imperfectly grouped before correction
- Small internal exons are sometimes wrongly mapped with accumulated sequencing errors, driving false-positive novel isoform discovery that correction must detect via the A-matrix threshold

## Evidence

- [results] Longcell identifies isoforms which frequently coexist in the same UMI cluster and compares their bulk level counts, designating the isoform with the smaller bulk proportion as misalignment and correcting it within each UMI cluster.: [evidence span withheld — anchor status: not_found]
- [discussion] we correct for read truncation and mapping errors by taking a 'consensus' among reads assigned to the same UMI, which correspond to PCR duplicates: [evidence span withheld — anchor status: not_found]
