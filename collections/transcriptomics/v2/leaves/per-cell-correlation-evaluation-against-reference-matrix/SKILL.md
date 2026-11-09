---
name: per-cell-correlation-evaluation-against-reference-matrix
description: Use when evaluating a single-cell long-read isoform quantification method
  against a matched gold-standard count matrix, e.g. Nanopore output vs PacBio IsoSeq
  truth for the same Jurkat 5' cDNA library.
when_to_use_negative:
- No library-matched gold-standard quantification exists for the same sample — per-cell
  correlations against an unrelated matrix are not interpretable
- The goal is discovering isoform switching or splicing heterogeneity (use Longcell's
  analysis modules like getIsoform or genesSitesPhi instead of benchmarking)
- The method's isoform identifiers cannot be joined to the truth matrix by isoform,
  so the full join yields an empty or near-empty row set
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Longcell
  role: Quantification method under evaluation; its output matrix is scored against
    the PacBio IsoSeq gold standard
  repo: github.com/yuntianf/Longcell
- name: Sicelore2
  role: Next-best comparator method in the benchmark (fixed-edit-distance UMI collapsing);
    the 8.72% improvement baseline
- name: FLAMES
  role: Comparator method run on the same Nanopore fastq with shared mapping, reference,
    annotation, and barcode whitelist
- name: Nanopore
  role: Sequencing platform producing the R10 FASTQ whose quantification is being
    evaluated against PacBio HiFi truth
- name: IsoSeq
  role: PacBio pipeline (v3.4.0) used to build the ground-truth isoform matrix via
    UMI-collapse
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:595f9fc532b8afa73c1609c2b76a457fc77b96692f4f96c024b7193c7bfba373
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
  - IsoSeq
  - Longcell
  - Nanopore
  - Sicelore2
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# per-cell-correlation-evaluation-against-reference-matrix

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmarking skill that scores single-cell isoform quantification methods by fully joining each method's cell-by-isoform count matrix to a library-matched PacBio IsoSeq gold-standard matrix and computing per-cell Spearman correlations. It quantifies accuracy gains, e.g. Longcell's mean 0.43 vs an 8.72% improvement over Sicelore2.

## When to use

Use when evaluating a single-cell long-read isoform quantification method against a matched gold-standard count matrix, e.g. Nanopore output vs PacBio IsoSeq truth for the same Jurkat 5' cDNA library.

## When NOT to use

- No library-matched gold-standard quantification exists for the same sample — per-cell correlations against an unrelated matrix are not interpretable
- The goal is discovering isoform switching or splicing heterogeneity (use Longcell's analysis modules like getIsoform or genesSitesPhi instead of benchmarking)
- The method's isoform identifiers cannot be joined to the truth matrix by isoform, so the full join yields an empty or near-empty row set

## Inputs

- Nanopore FASTQ (Promethion R10.4.1, FLO-PRO114M run) from the GSP-based Jurkat targeted 5' cDNA library (18 genes, 110 isoforms, 5881 cells)
- Matched PacBio HiFi reads for the same Jurkat sample (gold-standard source)
- GRCh38 reference genome and Gencode v39 GTF annotation
- Cell-barcode whitelist for the Jurkat library
- Cell-by-isoform count matrices output by each benchmarked method (Longcell, Sicelore2, FLAMES, wf-single-cell, IsoQuant)

## Outputs

- per_cell_isoform_correlation.csv with per-cell Spearman correlation values for all 5881 cells for every method
- Per-method per-cell correlation distribution summary
- Mean per-cell Spearman per method (Longcell expected 0.43) and percent improvement over the next-best method (8.72% vs Sicelore2)

## How to apply

First build the ground-truth isoform matrix from PacBio HiFi reads of the same sample using IsoSeq v3.4.0 with default parameters except --min-aln-coverage = 0.5 and --min-aln-identity = 0.9 for UMI-collapse. Run every quantification method (Longcell, Sicelore2, FLAMES, wf-single-cell, IsoQuant) with default parameters on the same Nanopore fastq, enforcing identical confounders: minimap2 v2.24 in spliced alignment mode, GRCh38 reference, Gencode v39 GTF, and a shared cell-barcode whitelist for FLAMES, Sicelore2, and Longcell. Fully join each method's output cell-by-isoform matrix with the truth matrix by isoform, then compute the Spearman correlation between ground truth and output for each of the 5881 cells. Summarize the per-cell correlation distribution per method and report the mean (Longcell expected 0.43) and percent improvement over the next-best method (8.72% vs Sicelore2). The rationale is that per-cell correlation controls for library composition while the shared-mapper design isolates each method's UMI/error-handling logic.

## Related tools

- **Longcell** (Quantification method under evaluation; its output matrix is scored against the PacBio IsoSeq gold standard) — github.com/yuntianf/Longcell
- **Sicelore2** (Next-best comparator method in the benchmark (fixed-edit-distance UMI collapsing); the 8.72% improvement baseline)
- **FLAMES** (Comparator method run on the same Nanopore fastq with shared mapping, reference, annotation, and barcode whitelist)
- **Nanopore** (Sequencing platform producing the R10 FASTQ whose quantification is being evaluated against PacBio HiFi truth)
- **IsoSeq** (PacBio pipeline (v3.4.0) used to build the ground-truth isoform matrix via UMI-collapse)

## Examples

```
per_cell_spearman <- sapply(rownames(ground_truth), function(cell) cor(ground_truth[cell, ], method_counts[cell, ], method = "spearman"))
```

## Evaluation signals

- per_cell_isoform_correlation.csv contains per-cell correlation values for all 5881 cells for every method (per the task's validation criterion)
- Longcell's mean per-cell Spearman correlation against the PacBio Isoseq gold standard reproduces the reported 0.43
- Longcell's relative improvement over Sicelore2 reproduces the reported 8.72%
- All methods share identical mapping software (minimap2 v2.24 spliced mode), reference (GRCh38), annotation (Gencode v39 GTF), and cell-barcode whitelist, so score differences are attributable to quantification logic
- The full join by isoform between each method matrix and the ground-truth matrix succeeds with a non-empty isoform intersection

## Limitations

- The benchmark covers only a targeted Jurkat panel (18 genes, 110 isoforms, 5881 cells), so per-cell correlation scores do not generalize to whole-transcriptome libraries
- Nanopore's low per-base accuracy (85–99%) inflates UMI counts in fixed-edit-distance collapsing methods like Sicelore2 and FLAMES, confounding their scores upward relative to error-aware pipelines
- Read truncation and mapping errors can create false-positive novel isoforms that fail to join the PacBio-derived truth matrix by isoform, biasing per-cell correlations

## Evidence

- [other] Longcell mean correlation vs PacBio gold standard: [evidence span withheld — anchor status: not_found]
