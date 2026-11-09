---
name: pacbio-isoseq-gold-standard-generation
description: Use when matched PacBio HiFi and Nanopore R10 reads exist for the same
  single-cell 5' cDNA library and you need an accuracy gold standard for evaluating
  isoform quantification methods like Longcell, FLAMES, Sicelore2, or wf-single-cell.
when_to_use_negative:
- No matched PacBio HiFi data exist for the Nanopore library, so no gold standard
  can be constructed
- The goal is discovery-oriented isoform/splicing analysis (e.g. differential alternative
  splicing via Longcell's LongcellPre/Splice objects) rather than method benchmarking
- Data are spatial Visium spots rather than single-cell libraries, where the Jurkat
  per-cell correlation benchmark does not apply
edam_operation: http://edamontology.org/operation_0335
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0102
tools:
- name: Longcell
  role: Nanopore single-cell isoform quantification method under evaluation against
    the PacBio gold standard
  repo: github.com/yuntianf/Longcell
- name: Sicelore2
  role: Existing fixed-edit-distance UMI-collapsing benchmark method (next-best, per-cell
    Spearman baseline)
- name: FLAMES
  role: Existing UMI-collapsing isoform quantification benchmark run with default
    parameters on the Nanopore FASTQ
- name: Nanopore
  role: Sequencing instrument (Promethion R10.4.1, FLO-PRO114M) producing the query
    reads being quantified
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:1c8b54d21ea61a834c48e02825bc1d5eca153eebfab206f9b1efb71b7325a320
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/yuntianf/Longcell
  tools:
  - FLAMES
  - Longcell
  - Nanopore
  - Sicelore2
  edam_operation: http://edamontology.org/operation_0335
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0102
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# pacbio-isoseq-gold-standard-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Build a PacBio IsoSeq ground-truth cell-by-isoform matrix from matched PacBio HiFi reads of the Jurkat 18-gene/110-isoform/5881-cell library, then benchmark Nanopore-based single-cell isoform quantification methods against it using per-cell Spearman correlation (Longcell mean = 0.43, +8.72% over Sicelore2).

## When to use

Use when matched PacBio HiFi and Nanopore R10 reads exist for the same single-cell 5' cDNA library and you need an accuracy gold standard for evaluating isoform quantification methods like Longcell, FLAMES, Sicelore2, or wf-single-cell.

## When NOT to use

- No matched PacBio HiFi data exist for the Nanopore library, so no gold standard can be constructed
- The goal is discovery-oriented isoform/splicing analysis (e.g. differential alternative splicing via Longcell's LongcellPre/Splice objects) rather than method benchmarking
- Data are spatial Visium spots rather than single-cell libraries, where the Jurkat per-cell correlation benchmark does not apply

## Inputs

- Nanopore FASTQ from Promethion R10.4.1 (FLO-PRO114M) run of GSP-based Jurkat targeted sequencing (18 genes, 5881 cells)
- Matched PacBio HiFi reads for the same Jurkat 5' cDNA sample
- GRCh38 reference genome
- Gencode v39 GTF annotation
- Cell-barcode whitelist

## Outputs

- Ground-truth cell-by-isoform count matrix from PacBio IsoSeq UMI-collapse
- per_cell_isoform_correlation.csv with per-cell Spearman correlation values for all 5881 cells for every method
- Method comparison summary (Longcell mean correlation 0.43; 8.72% relative improvement over Sicelore2)

## How to apply

Process the PacBio HiFi reads with IsoSeq v3.4.0 in UMI-collapse mode, overriding defaults with --min-aln-coverage = 0.5 and --min-aln-identity = 0.9 to produce the ground-truth isoform matrix. Run each Nanopore-based benchmark method (FLAMES, Sicelore2, wf-single-cell, IsoQuant, Longcell) with default parameters on the Nanopore FASTQ from the Promethion R10.4.1 (FLO-PRO114M) run, standardizing minimap2 v2.24 spliced alignment, GRCh38, and Gencode v39 GTF across all methods, and supplying the cell-barcode whitelist to FLAMES, Sicelore2, and Longcell. Fully join each method's cell-by-isoform count matrix with the ground-truth matrix by isoform, compute per-cell Spearman correlation over all 5881 cells, and compare method means. The rationale is that PacBio HiFi accuracy makes UMI-collapsed isoform counts a reliable reference, whereas Nanopore 85-99% per-base error inflates UMI counts under fixed-edit-distance collapsing and creates false-positive novel isoforms from misaligned small internal exons.

## Related tools

- **Longcell** (Nanopore single-cell isoform quantification method under evaluation against the PacBio gold standard) — github.com/yuntianf/Longcell
- **Sicelore2** (Existing fixed-edit-distance UMI-collapsing benchmark method (next-best, per-cell Spearman baseline))
- **FLAMES** (Existing UMI-collapsing isoform quantification benchmark run with default parameters on the Nanopore FASTQ)
- **Nanopore** (Sequencing instrument (Promethion R10.4.1, FLO-PRO114M) producing the query reads being quantified)

## Examples

```
isoseq collapse-umi --min-aln-coverage 0.5 --min-aln-identity 0.9 polished.hifi_reads.bam ground_truth_isoforms.bam
```

## Evaluation signals

- per_cell_isoform_correlation.csv contains per-cell correlation values for all 5881 cells for every method
- Longcell mean per-cell Spearman correlation reproduces the reported 0.43
- Longcell's relative improvement over Sicelore2 reproduces the reported 8.72%
- All benchmark methods share identical mapping (minimap2 v2.24 spliced mode), genome (GRCh38), and annotation (Gencode v39 GTF)
- IsoSeq UMI-collapse used the specified overrides: --min-aln-coverage 0.5 and --min-aln-identity 0.9

## Limitations

- Nanopore per-base accuracy of 85-99% makes UMI recovery more error-prone than cell barcode recovery, since UMIs are shorter and randomly generated
- Fixed-edit-distance UMI collapsing (as in Sicelore2 and FLAMES) overestimates UMI counts under high sequencing error, causing inflated quantification
- Sequencing error accumulation and read truncation can wrongly map small internal exons, producing false-positive novel isoforms
