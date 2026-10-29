---
name: attribution-matrix-construction
description: Use when UMI-clustered Nanopore long-read isoform assignments show cross-isoform
  read leakage within clusters and per-gene attribution matrices are needed to identify
  and correct mis-mapped isoforms.
when_to_use_negative:
- Input lacks isoform-level per-read alignments (e.g. short-read 10X gene expression
  data), so no attribution of reads to representative isoforms can be tabulated
- The gene is not highly expressed (below the HighExprsGene cutoff, thresh = 500 in
  the CRCLM workflow), leaving attribution counts too sparse to support the diagonal-dominance
  >= 2 rule
- The task is upstream UMI recovery/collapsing by fixed edit distance (the Sicelore2/FLAMES
  approach), not post-clustering diagnosis of misaligned reads
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: Longcell
  role: R package providing creatSplice_from_df and the per-gene attribution-matrix
    construction used to flag mis-mapped isoforms and reassign their reads
  repo: github.com/yuntianf/Longcell
- name: LongcellPre
  role: Supplies gene_bed.rds with the gene strand information required for downstream
    meta splicing site construction
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:48a212120b17b0fcf1cacf0c9f3af248e7cf9b8059be07e1bd95e787510eae2f
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# attribution-matrix-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs square per-gene isoform attribution matrices from Longcell's UMI-cluster read assignments and applies a diagonal-dominance rule (A_ii / sum of off-diagonal A_ij >= 2) to detect and correct misaligned isoforms in Nanopore single-cell/spatial long-read data.

## When to use

Use when UMI-clustered Nanopore long-read isoform assignments show cross-isoform read leakage within clusters and per-gene attribution matrices are needed to identify and correct mis-mapped isoforms.

## When NOT to use

- Input lacks isoform-level per-read alignments (e.g. short-read 10X gene expression data), so no attribution of reads to representative isoforms can be tabulated
- The gene is not highly expressed (below the HighExprsGene cutoff, thresh = 500 in the CRCLM workflow), leaving attribution counts too sparse to support the diagonal-dominance >= 2 rule
- The task is upstream UMI recovery/collapsing by fixed edit distance (the Sicelore2/FLAMES approach), not post-clustering diagnosis of misaligned reads

## Inputs

- Single-cell Nanopore isoform assignment table for the CRCLM sample (data frame consumed by Longcell's creatSplice_from_df)
- Gene strand annotation as gene_bed.rds from the LongcellPre package
- Reads grouped into UMI clusters per cell with per-read isoform labels

## Outputs

- Square per-gene isoform attribution matrix A (rows = isoform of origin, columns = cluster representative)
- misalignment_correction_report.csv containing the attribution matrices and corrected read-to-isoform assignments

## How to apply

Load the single-cell isoform data and gene strand annotation into a Longcell Splice object via creatSplice_from_df, then for each gene and UMI cluster order isoforms by read counts, length, and mapping quality, taking the top-ranked isoform as the cluster representative. Build per-cluster attribution tables counting reads of each isoform attributed to each representative, and sum these tables across all UMI clusters and cells into a square attribution matrix A per gene (A_ij = reads of isoform i attributed to representative j). Classify isoform i as mis-mapped when A_ii divided by the sum of off-diagonal A_ij falls below the default threshold of 2, then reassign its reads to the isoform j with the maximum A_ij; the rationale is that the misaligned isoform typically has the smaller bulk-level proportion within the confounded pair. After reassignment, repair truncated ends within each UMI cluster by comparison to complete reads sharing the UMI (meta-isoform grouping tolerates end truncations and internal exons shorter than 100 bp) and retain only the majority-vote isoform for clusters still holding multiple isoforms.

## Related tools

- **Longcell** (R package providing creatSplice_from_df and the per-gene attribution-matrix construction used to flag mis-mapped isoforms and reassign their reads) — github.com/yuntianf/Longcell
- **LongcellPre** (Supplies gene_bed.rds with the gene strand information required for downstream meta splicing site construction)

## Examples

```
crc_splice = creatSplice_from_df(iso); heg = HighExprsGene(crc_splice, thresh = 500)
```

## Evaluation signals

- For every retained isoform i, the diagonal-dominance ratio A_ii / sum of off-diagonal A_ij is >= 2; isoforms below this threshold are flagged and reassigned
- Misaligned reads of VPS28-201 exon 8 are attributed to the dominant correctly-mapped isoform after correction
- Constitutive-exon PSI estimates in the CRCLM sample become more consistent across reads after correction
- misalignment_correction_report.csv is produced and contains both attribution matrices and corrected assignments
- Each UMI cluster ends with a single majority-vote isoform after truncated ends are repaired against complete reads sharing the UMI

## Limitations

- The mis-mapping threshold of 2 on A_ii / off-diagonal sum is a fixed default and may misclassify lowly attributed isoforms in sparse genes
- Accumulated Nanopore sequencing errors (85-99% per-base accuracy) can wrongly map small internal exons, creating false-positive novel isoforms that contaminate attribution counts
- Fixed-edit-distance UMI collapsing (as in Sicelore2/FLAMES) overestimates UMI counts under high sequencing error, inflating the counts fed into attribution tables

## Evidence

- [intro] With the accumulation of sequencing errors in long reads, small internal exons are sometimes wrongly mapped, leading to the false-positive discovery of novel isoforms.: "With the accumulation of sequencing errors in long reads, small internal exons are sometimes wrongly mapped, leading to the false-positive discovery of novel isoforms."
- [discussion] we correct for read truncation and mapping errors by taking a "consensus" among reads assigned to the same UMI, which correspond to PCR duplicates: [evidence span withheld — anchor status: not_found]
- [methods] The gene strand information is necessary to build meta splicing sites in the downstream analysis, here we can use the `gene_bed.rds` output from the LongcellPre package: "The gene strand information is necessary to build meta splicing sites in the downstream analysis, here we can use the `gene_bed.rds` output from the LongcellPre package"
- [readme] Longcell is a package for single cell/Visium spot alternative splicing analysis.: "Longcell is a package for single cell/Visium spot alternative splicing analysis."
