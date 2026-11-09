---
name: nanopore-cell-barcode-umi-recovery
description: Use when raw Nanopore FASTQ reads from 10x 3'/5' or Visium-barcoded libraries
  need cell-barcode/UMI extraction and error-robust UMI deduplication before isoform
  or gene quantification.
when_to_use_negative:
- Input reads are short Illumina reads with high per-base accuracy, where simple exact
  or fixed-edit-distance UMI collapsing suffices and UMI scattering is minimal
- The library has no cell-barcode/UMI structure (e.g. bulk long-read cDNA), so barcode
  recovery is meaningless
- Only downstream alternative-splicing statistics are needed and Longcell-pre barcode/UMI
  outputs already exist — use the Longcell analysis package instead
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3170
tools:
- name: Longcell
  role: R/Python pipeline performing adapter search, k-mer whitelist barcode matching
    (edit distance <= 3), Needleman-Wunsch UMI graph clustering, and error-adaptive
    scatter pruning on Nanopore reads
  repo: github.com/yuntianf/Longcell
- name: LongcellPre
  role: Preprocessing companion producing gene strand/gene_bed.rds needed to build
    meta splicing sites after barcode/UMI recovery
  repo: github.com/yuntianf/Longcell
- name: Nanopore
  role: Sequencing platform whose 85–99% per-base error rate motivates the error-tolerant
    barcode/UMI recovery design
provenance:
  source_task_ids:
  - task_008
  - task_002
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:d7b63c00241b479bde5d1f6243cda03e59195527f577c9ff2297abeda6acc2ca
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
  - IsoQuant
  - Longcell
  - LongcellPre
  - Nanopore
  - Sicelore2
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# nanopore-cell-barcode-umi-recovery

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Longcell's preprocessing module for recovering cell barcodes and UMIs from noisy Nanopore single-cell/spatial long reads: sliding-window adapter search, k-mer dictionary barcode matching (edit-distance cutoff 3), Needleman-Wunsch-based UMI clustering with Louvain, and error-driven scatter pruning. It exists because Nanopore per-base accuracy (85–99%) makes whitelist-exact and fixed-edit-distance UMI collapsing unreliable.

## When to use

Use when raw Nanopore FASTQ reads from 10x 3'/5' or Visium-barcoded libraries need cell-barcode/UMI extraction and error-robust UMI deduplication before isoform or gene quantification.

## When NOT to use

- Input reads are short Illumina reads with high per-base accuracy, where simple exact or fixed-edit-distance UMI collapsing suffices and UMI scattering is minimal
- The library has no cell-barcode/UMI structure (e.g. bulk long-read cDNA), so barcode recovery is meaningless
- Only downstream alternative-splicing statistics are needed and Longcell-pre barcode/UMI outputs already exist — use the Longcell analysis package instead

## Inputs

- Nanopore FASTQ reads from single-cell (10x GSP, e.g. PromethION FLO-PRO114M R10.4.1 runs) or Visium spatially barcoded libraries
- Cell-barcode whitelist (e.g. 10x 737K-august-2016.txt)
- Reference genome GRCh38 FASTA and Gencode v39 GTF-derived splice junction BED for downstream minimap2 -ax splice alignment

## Outputs

- Per-read cell-barcode and UMI assignments with Needleman-Wunsch quality checks
- Denoised UMI cluster assignments per isoform per cell (Louvain clusters with scatter pruning applied)
- Estimated error-driven singleton ratio r
- Denoised reads/count matrix feeding downstream isoform quantification (e.g. Longcell+IsoQuant or Longcell native modules)

## How to apply

Search each read for the 10 bp / 2 bp sliding-window adapter upstream of the cell barcode, then align candidate barcodes against the whitelist using a k-mer dictionary (k=6) with an edit-distance cutoff of 3 and deviant start-position filtering. Recover UMIs by clustering their Needleman-Wunsch similarity graph with Louvain clustering, using an edge cutoff of l/2+2 (min_cut > 3, stopping when the minimum edge weight exceeds l/2+4) so PCR duplicates of the same UMI collapse despite sequencing errors. Estimate the error-driven singleton ratio r from the Needleman-Wunsch score distribution of the 12mer adapters upstream of confidently identified cell barcodes, counting adapters scoring below k = l/2+2. Within each isoform in each cell, rank UMI cluster sizes increasingly and iteratively remove the smallest clusters until the removed sum satisfies removed >= r * remaining / (1-r), reporting the pruned UMI count; this corrects UMI scattering from high-error singletons that fixed-edit-distance methods (Sicelore2, FLAMES) over-count. Align the denoised reads with minimap2 (spliced mode, GRCh38) for downstream isoform quantification.

## Related tools

- **Longcell** (R/Python pipeline performing adapter search, k-mer whitelist barcode matching (edit distance <= 3), Needleman-Wunsch UMI graph clustering, and error-adaptive scatter pruning on Nanopore reads) — github.com/yuntianf/Longcell
- **LongcellPre** (Preprocessing companion producing gene strand/gene_bed.rds needed to build meta splicing sites after barcode/UMI recovery) — github.com/yuntianf/Longcell
- **Nanopore** (Sequencing platform whose 85–99% per-base error rate motivates the error-tolerant barcode/UMI recovery design)

## Examples

```
devtools::install_github("yuntianf/Longcell", dependencies=TRUE)
```

## Evaluation signals

- Estimated singleton ratio r falls in a plausible range derived from the Needleman-Wunsch score distribution of 12mer adapters (adapters scoring below k = l/2+2)
- Pruned Nanopore UMI cluster-size distribution of a single-isoform gene matches the matched Illumina UMI cluster-size distribution (cf. Supplementary Fig. 5C)
- Pruned UMI counts remain stable across GC-dependent PCR amplification folds (A = k*sigmoid(0.5*(G-50)) + 2, k=20), reproducing the Fig. 1B invariance
- Recovered cell count and per-cell UMI counts are consistent with the expected library (e.g. 5881 cells in the Jurkat GSP library; 918 spots in the MOB Visium sample)
- Downstream benchmark: mean per-cell Spearman correlation against PacBio Iso-Seq or Illumina-supervised ground truth improves over fixed-edit-distance baselines (Longcell 0.43 vs Sicelore2 0.40 on the Jurkat Nanopore R10 benchmark)

## Limitations

- UMIs are shorter and randomly generated (no whitelist), making their recovery more vulnerable to sequencing errors than cell barcodes
- If sequencing error is high and scatter pruning is not applied, existing fixed-edit-distance methods overestimate UMI counts, causing inflated quantification
- Accumulated errors and read truncation can misalign small internal exons, producing false-positive novel isoforms even after barcode/UMI recovery

## Evidence

- [methods] k-mer dictionary barcode matching and Needleman-Wunsch UMI clustering parameters: [evidence span withheld — anchor status: not_found]
- [readme] install Longcell from GitHub: [evidence span withheld — anchor status: external_doc]
