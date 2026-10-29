---
name: constitutive-internal-exon-filtering
description: Use when quantifying bulk PSI from Nanopore long reads aligned to GRCh38
  and you need to isolate constitutive internal exons (read count > 30) and correct
  minimap2 mapping errors before computing splicing-in ratios.
when_to_use_negative:
- Input is a gene-level expression matrix without read-level splice-site assignments
  — there is no basis for per-exon spliced-in versus gene counts
- Exons are alternative (not constitutive) internal exons or have low coverage (read
  count ≤ 30), where PSI ratios are noisy and correction behavior differs
- Data are short-read sequencing, where UMI-based consensus correction of long-read
  mapping errors does not apply
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3518
tools:
- name: Longcell
  role: Statistical pipeline that recovers cell barcodes and UMIs, performs UMI-cluster
    consensus correction of misaligned small internal exons, and supports Splice-object
    PSI computation via creatSplice_from_df
  repo: github.com/yuntianf/Longcell
- name: minimap2
  role: Aligns Nanopore reads to GRCh38 in spliced alignment mode (v2.24) using the
    Gencode v39 splice junction bed; its raw alignments provide the pre-correction
    PSI baseline
- name: dplyr
  role: R package used in the Longcell methods workflow to filter and tabulate exon-level
    read counts during PSI computation
- name: Nanopore
  role: Long-read sequencing platform producing the reads whose spliced alignments
    are filtered and corrected
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:09adf3ede62f9460c4e0c718f6c1915fe04c29d94862b94b062e78d4e511faf2
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
  - minimap2
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# constitutive-internal-exon-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A Longcell-based filtering and correction workflow that restricts PSI estimation to constitutive internal exons with read count > 30 in Nanopore CRCLM data, applies UMI-consensus misalignment correction, and quantifies how wrongly mapped small internal exons bias isoform quantification (average PSI shifts 0.67 → 0.92).

## When to use

Use when quantifying bulk PSI from Nanopore long reads aligned to GRCh38 and you need to isolate constitutive internal exons (read count > 30) and correct minimap2 mapping errors before computing splicing-in ratios.

## When NOT to use

- Input is a gene-level expression matrix without read-level splice-site assignments — there is no basis for per-exon spliced-in versus gene counts
- Exons are alternative (not constitutive) internal exons or have low coverage (read count ≤ 30), where PSI ratios are noisy and correction behavior differs
- Data are short-read sequencing, where UMI-based consensus correction of long-read mapping errors does not apply

## Inputs

- Nanopore FASTQ reads from a single-cell/spatially barcoded sample (e.g., CRCLM metastasis to liver)
- GRCh38 reference genome
- Gencode v39 GTF-derived splice junction bed (generated with paftools.js)
- gene_bed.rds output from LongcellPre (gene strand information for meta splicing sites)

## Outputs

- Pre- and post-correction PSI values for constitutive internal exons with read count > 30 (average PSI 0.67 pre-correction, 0.92 post-correction)
- Fraction of reads with falsely mapped small internal exons per transcript (e.g., ~20% for VPS28-201 small middle exon pre-correction)
- psi_correction_comparison.csv with all computed values

## How to apply

Process CRCLM Nanopore reads through Longcell's adapter search, polyA detection, cell barcode and UMI assignment, then align to GRCh38 with minimap2 v2.24 in spliced alignment mode using the splice junction bed derived from the Gencode v39 GTF (via paftools.js). Build a Splice object from the read-level splice-site table with creatSplice_from_df, then compute pre-correction PSI by tallying spliced-in versus gene counts per internal exon from raw minimap2 alignments, retaining constitutive internal exons with read count > 30. Apply Longcell's UMI-clustering consensus correction — flagging isoform i as wrongly mapped when Aii / sum(i≠j) Aij < 2 (default threshold) — and recompute post-correction PSI for the same exon set. The rationale, per the article, is that sequencing-error accumulation causes small internal exons to be wrongly mapped, inflating PSI variation and creating false-positive novel isoforms; consensus among reads sharing a UMI (PCR duplicates) corrects truncation and mapping errors.

## Related tools

- **Longcell** (Statistical pipeline that recovers cell barcodes and UMIs, performs UMI-cluster consensus correction of misaligned small internal exons, and supports Splice-object PSI computation via creatSplice_from_df) — github.com/yuntianf/Longcell
- **minimap2** (Aligns Nanopore reads to GRCh38 in spliced alignment mode (v2.24) using the Gencode v39 splice junction bed; its raw alignments provide the pre-correction PSI baseline)
- **dplyr** (R package used in the Longcell methods workflow to filter and tabulate exon-level read counts during PSI computation)
- **Nanopore** (Long-read sequencing platform producing the reads whose spliced alignments are filtered and corrected)

## Examples

```
crc_splice = creatSplice_from_df(iso)
```

## Evaluation signals

- Post-correction average PSI across retained constitutive internal exons (read count > 30) reaches ~0.92, up from the pre-correction baseline of ~0.67
- Pre-correction wrongly-mapped fraction for the VPS28-201 small middle exon is ~20%, computed from the exon-level splice-site vector for that transcript
- Misalignment flagging reproduces Longcell's default rule: isoform i is flagged when Aii / sum(i≠j) Aij < 2
- All pre/post PSI values and VPS28-201 fractions are written to psi_correction_comparison.csv with matching exon identifiers across correction rounds
- Splice object construction succeeds via creatSplice_from_df on the read-level splice-site table before any filtering

## Limitations

- Small internal exons are sometimes wrongly mapped due to accumulated sequencing errors in long reads, leading to false-positive discovery of novel isoforms if correction is not applied
- Fixed-edit-distance UMI collapsing methods (e.g., Sicelore2, FLAMES) overestimate UMI counts under high sequencing error, causing inflated quantification; Longcell's statistical clustering is needed instead
- Nanopore per-base accuracy (85–99%) and read truncation remain inherent error sources that the consensus step only partially mitigates

## Evidence

- [discussion] Consensus correction among reads assigned to the same UMI corrects read truncation and mapping errors.: [evidence span withheld — anchor status: not_found]
