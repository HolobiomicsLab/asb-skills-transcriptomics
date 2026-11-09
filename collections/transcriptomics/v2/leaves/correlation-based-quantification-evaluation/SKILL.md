---
name: correlation-based-quantification-evaluation
description: Use when benchmarking Nanopore long-read isoform quantification pipelines
  (Longcell, IsoQuant, FLAMES, Sicelore2, wf-single-cell) against a supervised ground-truth
  count matrix on the same cells.
when_to_use_negative:
- Input is already a validated count matrix needing no benchmarking against a reference
  (the skill evaluates quantification, it does not produce it)
- No ground-truth or Illumina-supervised quantification exists for the same cells/isoforms,
  so per-cell correlation vectors cannot be joined
- Goal is short-read (Illumina-only) quantification evaluation rather than long-read
  Nanopore pipeline benchmarking
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Longcell
  role: 'Nanopore preprocessing: adapter/polyA search, k-mer cell barcode matching
    (edit-distance cutoff 3, deviant start-position filtering), Needleman-Wunsch UMI
    clustering; its denoised reads combined with IsoQuant produced the top-correlated
    quantification'
  repo: github.com/yuntianf/Longcell
- name: IsoQuant
  role: Isoform quantification engine run on Longcell-denoised reads and standalone
    (v3.3) as a benchmarked method
- name: FLAMES
  role: Benchmarked isoform quantification method (git commit 774e16ae53a1430e03081970827e93f1fbaecead),
    default parameters
- name: Sicelore2
  role: Benchmarked isoform quantification method (v1) using fixed-edit-distance UMI
    collapsing
- name: wf-single-cell
  role: Benchmarked Nanopore pipeline (v0.2.9) achieving the second-best mean correlation
    (0.66)
- name: minimap2
  role: Spliced read alignment shared by all methods (GRCh38, Gencode v39 junction
    BED, --secondary=no --sam-hit-only) to keep the comparison controlled
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:5bbff33a696a990a905bf58d8a71875891637d15f92d4043f9e187af0fc8e7e2
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
  - Sicelore2
  - minimap2
  - wf-single-cell
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# correlation-based-quantification-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates single-cell/spatial long-read isoform quantification methods by computing per-cell Spearman correlations between each method's cell-by-isoform count matrix and a ground-truth (Illumina-supervised) quantification, then averaging across cells to rank methods on benchmarks like the MOB Visium Nanopore dataset.

## When to use

Use when benchmarking Nanopore long-read isoform quantification pipelines (Longcell, IsoQuant, FLAMES, Sicelore2, wf-single-cell) against a supervised ground-truth count matrix on the same cells.

## When NOT to use

- Input is already a validated count matrix needing no benchmarking against a reference (the skill evaluates quantification, it does not produce it)
- No ground-truth or Illumina-supervised quantification exists for the same cells/isoforms, so per-cell correlation vectors cannot be joined
- Goal is short-read (Illumina-only) quantification evaluation rather than long-read Nanopore pipeline benchmarking

## Inputs

- Nanopore long-read FASTQ files (e.g., MOB Visium Nanopore sample, 918 cells/spots)
- Cell-by-isoform count matrix from each method under test
- Illumina-supervised ground-truth cell-by-isoform count matrix (Lebrigand et al. barcode/UMI recovery)
- Reference genome GRCh38 FASTA and Gencode v39 GTF-derived splice junction BED
- Method-specific parameters/versions (FLAMES git commit 774e16ae53a1430e03081970827e93f1fbaecead, sicelore2 v1, wf-single-cell v0.2.9, IsoQuant v3.3)

## Outputs

- Per-cell Spearman correlation values for each method
- Mean per-cell Spearman correlation per method (e.g., 0.70 Longcell+IsoQuant, 0.66 wf-single-cell, 0.58 IsoQuant)
- Comparison table written to spearman_correlations.csv

## How to apply

Run each candidate method (e.g., Longcell denoising + IsoQuant, FLAMES commit 774e16ae53, sicelore2 v1, wf-single-cell v0.2.9, IsoQuant v3.3) on the same Nanopore FASTQ with identical mapping (minimap2 -ax splice --junc-bed --secondary=no --sam-hit-only), reference genome (GRCh38), and annotation (Gencode v39 GTF). Collect each method's cell-by-isoform count matrix and fully join it with the Illumina-supervised ground-truth matrix (cell barcode and UMI recovery from Lebrigand et al.) by isoform so that per-cell vectors align. For each of the 918 MOB Visium cells, compute the Spearman correlation between the method's isoform counts and the ground-truth counts, then average across cells to yield one mean value per method. Judge correctness by checking whether reproduced means match reported benchmark values (Longcell+IsoQuant 0.70, wf-single-cell 0.66, IsoQuant 0.58) within a reasonable tolerance, and write the comparison table to spearman_correlations.csv. Rationale: fixed-edit-distance UMI collapsing overestimates UMI counts under high Nanopore sequencing error (85-99% per-base accuracy), so correlation against a supervised reference directly measures whether barcode/UMI recovery and consensus correction translate into accurate isoform-level quantification.

## Related tools

- **Longcell** (Nanopore preprocessing: adapter/polyA search, k-mer cell barcode matching (edit-distance cutoff 3, deviant start-position filtering), Needleman-Wunsch UMI clustering; its denoised reads combined with IsoQuant produced the top-correlated quantification) — github.com/yuntianf/Longcell
- **IsoQuant** (Isoform quantification engine run on Longcell-denoised reads and standalone (v3.3) as a benchmarked method)
- **FLAMES** (Benchmarked isoform quantification method (git commit 774e16ae53a1430e03081970827e93f1fbaecead), default parameters)
- **Sicelore2** (Benchmarked isoform quantification method (v1) using fixed-edit-distance UMI collapsing)
- **wf-single-cell** (Benchmarked Nanopore pipeline (v0.2.9) achieving the second-best mean correlation (0.66))
- **minimap2** (Spliced read alignment shared by all methods (GRCh38, Gencode v39 junction BED, --secondary=no --sam-hit-only) to keep the comparison controlled)

## Examples

```
from scipy.stats import spearmanr; per_cell = truth.apply(lambda c: spearmanr(c, pred[c.name]).statistic, axis=0); per_cell.mean().to_csv("spearman_correlations.csv")
```

## Evaluation signals

- Reproduced mean per-cell Spearman correlations match reported values (Longcell+IsoQuant 0.70, wf-single-cell 0.66, IsoQuant 0.58) within a reasonable tolerance
- Correlations are computed on isoforms fully joined across method and ground-truth matrices, so per-cell vectors have identical isoform ordering
- All benchmarked methods share identical mapping (minimap2 spliced mode, GRCh38, Gencode v39), so ranking differences are attributable to barcode/UMI recovery and quantification, not alignment
- Comparison table spearman_correlations.csv contains one mean correlation value per method and is reproducible from the same FASTQ inputs
- All 918 MOB Visium cells contribute per-cell correlations to the mean

## Limitations

- Accuracy is only relative to the Illumina-supervised reference; the ground truth itself depends on Illumina barcode/UMI recovery and may not capture all Nanopore-detected isoforms
- High Nanopore sequencing error (85-99% per-base) causes fixed-edit-distance UMI collapsing methods to overestimate UMI counts and inflate quantification, biasing their correlations downward
- Low sequencing accuracy and read truncation can misalign small internal exons, creating false-positive novel isoforms that reduce correlation even with correct barcode recovery
- Result is a single mean scalar per method, which can mask per-cell and per-isoform variability

## Evidence

- [other] On the MOB Visium Nanopore benchmark (918 cells), Longcell-IsoQuant and wf-single-cell achieved the best mean per-cell Spearman correlations (0.70 and 0.66): [evidence span withheld — anchor status: not_found]
- [other] verify reproduced correlations and write CSV: [evidence span withheld — anchor status: not_found]
