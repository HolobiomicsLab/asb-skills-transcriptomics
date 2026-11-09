---
name: bulk-alternative-splicing-comparison
description: Use when Nanopore long-read alignments (e.g., minimap2 v2.24 spliced
  alignments to GRCh38) show inflated PSI or suspected false-positive small internal
  exons, and you need to quantify the PSI shift from Longcell's consensus correction
  on bulk isoform quantification.
when_to_use_negative:
- Input data is not Nanopore long-read spliced alignment output (e.g., Illumina short-read
  or already consensus-corrected isoform counts)
- The goal is single-cell or spot-level splicing heterogeneity (phi metrics via genesSitesPhi/phiPlot)
  rather than bulk exon-level PSI correction comparison
- Exon read counts are below the constitutive-exon cutoff (> 30), where PSI estimates
  are noisy and the bulk comparison is unreliable
tools:
- name: Longcell
  role: R package whose UMI-cluster consensus correction removes mapping errors (wrongly
    mapped small internal exons) so pre- vs post-correction bulk PSI can be compared
  repo: github.com/yuntianf/Longcell
- name: Nanopore
  role: Long-read sequencing platform producing the reads whose low per-base accuracy
    (85–99%) drives the misalignment being quantified
- name: dplyr
  role: Used in the Longcell methods for tabulating and filtering exon-level PSI/splice-site
    tables in R
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:340e1a8338031829f6a96774353687055f1e349faabbe8dd22e7b7c4ef99414a
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# bulk-alternative-splicing-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies bulk percent-spliced-in (PSI) for constitutive internal exons in a Nanopore long-read sample before and after Longcell's UMI-clustering misalignment correction, revealing how mapping errors bias isoform quantification (e.g., CRCLM average PSI shifting from 0.67 to 0.92 and ~20% of VPS28-201 reads wrongly covering its small middle exon).

## When to use

Use when Nanopore long-read alignments (e.g., minimap2 v2.24 spliced alignments to GRCh38) show inflated PSI or suspected false-positive small internal exons, and you need to quantify the PSI shift from Longcell's consensus correction on bulk isoform quantification.

## When NOT to use

- Input data is not Nanopore long-read spliced alignment output (e.g., Illumina short-read or already consensus-corrected isoform counts)
- The goal is single-cell or spot-level splicing heterogeneity (phi metrics via genesSitesPhi/phiPlot) rather than bulk exon-level PSI correction comparison
- Exon read counts are below the constitutive-exon cutoff (> 30), where PSI estimates are noisy and the bulk comparison is unreliable

## Inputs

- Nanopore FASTQ reads from a bulk or single-cell/spatial sample (e.g., CRCLM colorectal metastasis to liver)
- GRCh38 reference genome
- Gencode v39 GTF-derived splice junction BED (generated with paftools.js)
- Read-level splice-site alignment table output by minimap2 v2.24 spliced alignment
- Longcell/LongcellPre gene BED with strand information (gene_bed.rds from LongcellPre)

## Outputs

- Pre-correction per-exon PSI table for constitutive internal exons (read count > 30)
- Post-correction per-exon PSI table after UMI-clustering misalignment correction
- Per-isoform wrongly-mapped flags (Aii / sum_i≠j Aij < 2)
- Wrongly-mapped read fraction for suspect exons (e.g., ~20% for the small middle exon of VPS28-201)
- psi_correction_comparison.csv summarizing pre/post-correction PSI values

## How to apply

Process the Nanopore reads (e.g., the CRCLM sample) through LongcellPre — adapter search, polyA detection, cell barcode/UMI assignment — then align to GRCh38 with minimap2 v2.24 in spliced alignment mode using a splice-junction BED generated from the Gencode v39 GTF via paftools.js. Build a Splice object from the read-level splice-site table with creatSplice_from_df, compute pre-correction PSI by tallying spliced-in versus gene-level counts per internal exon, and retain constitutive internal exons with read count > 30. Apply Longcell's UMI-cluster consensus correction, flagging isoform i as wrongly mapped when Aii / sum_i≠j Aij < 2 (default threshold), then recompute PSI on the same exon set — the CRCLM average should rise from 0.67 to ~0.92. Separately, quantify the fraction of reads containing a falsely mapped small middle exon (e.g., ~20% for VPS28-201) by inspecting the exon-level splice-site vector pre-correction, since sequencing-error accumulation causes small internal exons to be wrongly mapped and falsely discovered as novel isoforms.

## Related tools

- **Longcell** (R package whose UMI-cluster consensus correction removes mapping errors (wrongly mapped small internal exons) so pre- vs post-correction bulk PSI can be compared) — github.com/yuntianf/Longcell
- **Nanopore** (Long-read sequencing platform producing the reads whose low per-base accuracy (85–99%) drives the misalignment being quantified)
- **dplyr** (Used in the Longcell methods for tabulating and filtering exon-level PSI/splice-site tables in R)

## Examples

```
library(Longcell); crc_splice = creatSplice_from_df(iso)
```

## Evaluation signals

- Post-correction average PSI across retained constitutive internal exons (read count > 30) reaches ~0.92 from a pre-correction value of ~0.67 in the CRCLM sample
- Pre-correction wrongly-mapped read fraction for the small middle exon of VPS28-201 is ~20%, and drops after consensus correction
- All isoforms flagged as wrongly mapped satisfy the default criterion Aii / sum_i≠j Aij < 2
- psi_correction_comparison.csv contains paired pre/post PSI values for the identical exon set (same exons in both columns)
- Splice object reconstructs reproducibly via creatSplice_from_df on the same read-level splice-site table

## Limitations

- Existing fixed-edit-distance UMI collapsing methods (e.g., Sicelore2, FLAMES) overestimate UMI counts under high sequencing error, causing inflated quantification if used instead of Longcell's statistical clustering
- Small internal exons can be wrongly mapped under error accumulation, leading to false-positive novel isoform discovery before correction
- The influence of sequencing errors, read truncation, and mapping errors on single-cell long-read isoform quantification had not been characterized prior to this study; other datasets may not show the same 0.67→0.92 shift
- No changelog or versioned release information is documented for the Longcell repository, complicating reproducibility pinning

## Evidence

- [discussion] we correct for read truncation and mapping errors by taking a "consensus" among reads assigned to the same UMI: [evidence span withheld — anchor status: not_found]
- [intro] With the accumulation of sequencing errors in long reads, small internal exons are sometimes wrongly mapped,: "With the accumulation of sequencing errors in long reads, small internal exons are sometimes wrongly mapped, leading to the false-positive discovery of novel isoforms."
- [intro] Existing fixed-edit-distance UMI collapsing methods overestimate UMI counts under high sequencing error,: "If sequencing error is high, existing methods overestimate the number of UMIs, causing inflated quantification."
- [methods] reads were aligned to the human Genome (GRCh38) with minimap2 v2.24: "reads were aligned to the human Genome (GRCh38) with minimap2 v2.24"
