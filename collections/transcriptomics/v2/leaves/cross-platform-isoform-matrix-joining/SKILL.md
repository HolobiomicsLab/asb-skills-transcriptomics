---
name: cross-platform-isoform-matrix-joining
description: Use when benchmarking single-cell isoform quantification methods against
  a cross-platform gold standard (e.g., Nanopore output vs a matched PacBio IsoSeq
  reference) and per-cell accuracy metrics are needed.
when_to_use_negative:
- No matched cross-platform ground-truth dataset exists, so there is no PacBio IsoSeq
  matrix to join against
- The goal is downstream splicing analysis (highly variable isoforms, differential
  alternative splicing) from a single matrix rather than cross-platform accuracy benchmarking
- Input is a single method's raw long-read FASTQ and no cell-by-isoform count matrix
  has been produced yet
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3168
tools:
- name: Longcell
  role: Benchmarked quantification method whose Nanopore-derived cell-by-isoform matrix
    is joined against the PacBio gold standard; produced the best mean per-cell Spearman
    (0.43)
  repo: github.com/yuntianf/Longcell
- name: Sicelore2
  role: Next-best comparison method using fixed-edit-distance UMI collapsing; baseline
    for the 8.72% relative improvement
- name: FLAMES
  role: Comparison quantification method run with default parameters on the same Nanopore
    fastq and shared GRCh38/Gencode v39 mapping setup
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:f4b97543b2bcca4e86f61c89c5ae81c1d2295a525bc54127a132ab6d54636484
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
  - Sicelore2
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cross-platform-isoform-matrix-joining

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarking skill that fully joins each method's Nanopore-derived cell-by-isoform count matrix with a PacBio IsoSeq ground-truth matrix by isoform and computes per-cell Spearman correlations, yielding quantification accuracy metrics (Longcell mean 0.43, +8.72% over Sicelore2 on the Jurkat benchmark).

## When to use

Use when benchmarking single-cell isoform quantification methods against a cross-platform gold standard (e.g., Nanopore output vs a matched PacBio IsoSeq reference) and per-cell accuracy metrics are needed.

## When NOT to use

- No matched cross-platform ground-truth dataset exists, so there is no PacBio IsoSeq matrix to join against
- The goal is downstream splicing analysis (highly variable isoforms, differential alternative splicing) from a single matrix rather than cross-platform accuracy benchmarking
- Input is a single method's raw long-read FASTQ and no cell-by-isoform count matrix has been produced yet

## Inputs

- Nanopore FASTQ (Promethion R10.4.1, FLO-PRO114M) of the GSP-based Jurkat 5' cDNA targeted library (18 genes, 5881 cells)
- Matched PacBio HiFi reads for the same sample (gold-standard source)
- Cell-by-isoform count matrix from each benchmark method (Longcell, Sicelore2, FLAMES, wf-single-cell, IsoQuant)
- Ground-truth cell-by-isoform matrix from IsoSeq v3.4.0 UMI-collapse
- GRCh38 reference genome and Gencode v39 GTF
- Cell-barcode whitelist

## Outputs

- per_cell_isoform_correlation.csv with per-cell Spearman correlation values for all 5881 cells for every method
- Mean per-cell Spearman correlation per method (Longcell = 0.43)
- Percent improvement of best method over next-best (8.72% over Sicelore2)

## How to apply

Build the ground-truth cell-by-isoform matrix by processing matched PacBio HiFi reads with IsoSeq v3.4.0 (default parameters except --min-aln-coverage = 0.5 and --min-aln-identity = 0.9 for UMI-collapse). Run every benchmark method (FLAMES, Sicelore2, wf-single-cell, IsoQuant, Longcell) on the same Nanopore fastq — e.g. the Promethion R10.4.1 (FLO-PRO114M) run of the GSP-based Jurkat targeted library (18 genes, 110 isoforms, 5881 cells) — under identical mapping software (minimap2 v2.24, spliced alignment mode), reference genome (GRCh38), isoform annotation (Gencode v39 GTF), and supply the cell-barcode whitelist to FLAMES, Sicelore2, and Longcell. Fully join each method's output matrix with the ground-truth matrix by isoform so every isoform is retained, then compute the Spearman correlation between ground truth and predicted counts separately for each cell. Summarize the per-cell correlation distribution per method and report each method's mean correlation and percent improvement over the next-best method (Longcell 0.43, 8.72% over Sicelore2).

## Related tools

- **Longcell** (Benchmarked quantification method whose Nanopore-derived cell-by-isoform matrix is joined against the PacBio gold standard; produced the best mean per-cell Spearman (0.43)) — github.com/yuntianf/Longcell
- **Sicelore2** (Next-best comparison method using fixed-edit-distance UMI collapsing; baseline for the 8.72% relative improvement)
- **FLAMES** (Comparison quantification method run with default parameters on the same Nanopore fastq and shared GRCh38/Gencode v39 mapping setup)

## Examples

```
cor_per_cell <- sapply(intersect(colnames(gt_matrix), colnames(pred_matrix)), function(c) cor(gt_matrix[,c], pred_matrix[,c], method="spearman", use="complete.obs"))
```

## Evaluation signals

- per_cell_isoform_correlation.csv contains per-cell correlation values for all 5881 cells for every method (schema completeness check)
- Longcell's mean per-cell Spearman correlation reproduces the reported 0.43
- Longcell's relative improvement over Sicelore2 reproduces the reported 8.72%
- All methods were run with the same minimap2 v2.24 spliced alignment, GRCh38, and Gencode v39 GTF so correlations are comparable
- The full join by isoform retains all 110 isoforms from the ground-truth matrix (no isoform silently dropped by an inner-join)

## Limitations

- Nanopore per-base sequencing accuracy (85–99%) means fixed-edit-distance UMI collapsing in existing methods can overestimate UMI counts and inflate quantification
- Benchmark scope is limited to a targeted 18-gene/110-isoform Jurkat 5' cDNA library, not a whole-transcriptome comparison
- Ground truth derives from PacBio HiFi UMI-collapse with specific thresholds (--min-aln-coverage 0.5, --min-aln-identity 0.9); different thresholds would change the gold standard

## Evidence

- [other] Fully join each method's output cell-by-isoform count matrix with the ground-truth matrix by isoform and compute the correlation between ground truth and output for each cell.: [evidence span withheld — anchor status: not_found]
- [other] verify that per_cell_isoform_correlation.csv contains per-cell correlation values for all 5881 cells for every method: [evidence span withheld — anchor status: not_found]
