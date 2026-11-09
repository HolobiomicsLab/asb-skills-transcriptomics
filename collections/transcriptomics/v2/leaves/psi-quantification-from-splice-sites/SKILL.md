---
name: psi-quantification-from-splice-sites
description: Use when you have read-level splice-site tables from Nanopore long reads
  (e.g., aligned to GRCh38 with minimap2 v2.24) and need bulk PSI per internal exon,
  especially to quantify how UMI-consensus correction shifts PSI for constitutive
  exons.
when_to_use_negative:
- Input is single-cell or spatial PSI heterogeneity analysis (psiCellPlot, genesSitesPhi
  beta-binomial metrics) rather than bulk PSI correction from a splice-site table
- Reads are short-read (Illumina) alignments where the Longcell UMI-consensus misalignment
  correction does not apply
- Exon read counts are below 30, since constitutive-exon PSI estimates are only retained
  above that threshold in this workflow
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Longcell
  role: Provides UMI-clustering consensus correction that flags isoform i as wrongly
    mapped when Aii / sum_{i≠j} Aij < 2, and creatSplice_from_df to build the Splice
    object from the read-level splice-site table
  repo: https://github.com/yuntianf/Longcell
- name: Nanopore
  role: Long-read sequencing platform generating the reads whose splice-site vectors
    feed PSI computation
- name: dplyr
  role: Used in the methods for tabulating and summarizing per-exon PSI values
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:cb5e64796b66dcb79a83d2f78cfdee2f3ced1b4281f733f8a5c26d5328aea458
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/yuntianf/Longcell
  repo_url: https://github.com/yuntianf/Longcell
  tools:
  - Longcell
  - Nanopore
  - dplyr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# psi-quantification-from-splice-sites

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute bulk percent-spliced-in (PSI) estimates for internal exons from read-level splice-site tables of Nanopore long reads, and apply Longcell's UMI-clustering consensus correction to remove mapping artifacts such as wrongly mapped small internal exons before recomputing PSI.

## When to use

Use when you have read-level splice-site tables from Nanopore long reads (e.g., aligned to GRCh38 with minimap2 v2.24) and need bulk PSI per internal exon, especially to quantify how UMI-consensus correction shifts PSI for constitutive exons.

## When NOT to use

- Input is single-cell or spatial PSI heterogeneity analysis (psiCellPlot, genesSitesPhi beta-binomial metrics) rather than bulk PSI correction from a splice-site table
- Reads are short-read (Illumina) alignments where the Longcell UMI-consensus misalignment correction does not apply
- Exon read counts are below 30, since constitutive-exon PSI estimates are only retained above that threshold in this workflow

## Inputs

- Read-level splice-site table from minimap2 v2.24 spliced alignments of Nanopore long reads (GRCh38, Gencode v39 GTF-derived splice junction bed)
- Gencode v39 GTF / paftools.js splice junction bed
- Longcell gene_bed.rds from LongcellPre (gene strand information for building meta splicing sites)

## Outputs

- Pre-correction PSI per internal exon (spliced-in vs. gene counts, constitutive exons with read count > 30)
- Post-correction PSI per internal exon after UMI-clustering misalignment correction (average bulk PSI ~0.92 vs. ~0.67 pre-correction)
- Wrongly-mapped exon fractions (e.g., ~20% of VPS28-201 reads containing the falsely mapped small middle exon pre-correction)
- psi_correction_comparison.csv with pre/post-correction values

## How to apply

Process CRCLM Nanopore reads through Longcell's adapter search, polyA detection, cell barcode/UMI assignment, then align to GRCh38 with minimap2 v2.24 in spliced mode using a Gencode v39 GTF-derived splice junction bed. Build a Splice object from the read-level splice-site table via creatSplice_from_df. Compute pre-correction PSI by tallying spliced-in versus gene counts per internal exon from raw minimap2 alignments, retaining constitutive internal exons with read count > 30. Apply Longcell's UMI-clustering misalignment correction, flagging isoform i as wrongly mapped when Aii / sum_{i≠j} Aij < 2 (default threshold), then recompute post-correction PSI for the same exon set. The rationale is that small internal exons are sometimes wrongly mapped under accumulated sequencing error, causing false-positive novel isoforms; the consensus among reads assigned to the same UMI (PCR duplicates) corrects read truncation and mapping errors, raising average constitutive-exon PSI from ~0.67 to ~0.92 in the CRCLM sample.

## Related tools

- **Longcell** (Provides UMI-clustering consensus correction that flags isoform i as wrongly mapped when Aii / sum_{i≠j} Aij < 2, and creatSplice_from_df to build the Splice object from the read-level splice-site table) — https://github.com/yuntianf/Longcell
- **Nanopore** (Long-read sequencing platform generating the reads whose splice-site vectors feed PSI computation)
- **dplyr** (Used in the methods for tabulating and summarizing per-exon PSI values)

## Examples

```
crc_splice = creatSplice_from_df(iso)
```

## Evaluation signals

- Post-correction average PSI across retained constitutive internal exons (read count > 30) reaches ~0.92, up from ~0.67 pre-correction
- Pre-correction wrongly-mapped fraction for the small middle exon of VPS28-201 is ~20% of reads, confirmed by inspecting that transcript's exon-level splice-site vector
- All isoforms flagged by the misalignment rule satisfy Aii / sum_{i≠j} Aij < 2 (default threshold)
- PSI values for the same exon set are computed identically pre- and post-correction so the delta is attributable to the correction
- psi_correction_comparison.csv is written with pre- and post-correction PSI columns matching the expected shifts

## Limitations

- Small internal exons are sometimes wrongly mapped with accumulated sequencing errors, leading to false-positive discovery of novel isoforms before correction
- The correction relies on multiple reads per UMI (PCR duplicates); read truncation reduces the number of comparable reads per cluster
- Constitutive-exon PSI estimates are restricted to exons with read count > 30, limiting coverage of lowly expressed exons

## Evidence

- [discussion] Consensus among same-UMI reads corrects truncation and mapping errors: [evidence span withheld — anchor status: not_found]
- [readme] Longcell install instructions from README: [evidence span withheld — anchor status: external_doc]
