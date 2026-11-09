---
name: nanopore-long-read-simulation
description: Use when benchmarking or validating long-read isoform quantification
  pipelines where real data lack gold-standard isoform counts, and a simulated cell-by-isoform
  matrix with known error profiles is needed as ground truth.
when_to_use_negative:
- When real Nanopore data with independent experimental validation (e.g., targeted
  sequencing) are available and simulation artifacts would weaken the claim
- When the goal is biological discovery of isoform switching rather than method benchmarking
  — use the Longcell analysis package instead of the simulator
- When the quantification method cannot accept FASTQ with 10x-style 5' adapter/barcode/UMI
  structure, since the simulation design assumes that read architecture
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Longcell
  role: Quantification method under test; pipeline of adapter search, k-mer barcode
    matching (k=6), minimap2 splice alignment, and Louvain UMI clustering applied
    to simulated reads
  repo: https://github.com/yuntianf/Longcell
- name: Sicelore2
  role: Competing fixed-edit-distance UMI-collapsing quantification method benchmarked
    on simulated data
- name: FLAMES
  role: Competing isoform quantification method run with default parameters on simulated
    FASTQ
- name: IsoQuant
  role: Competing aligner/quantifier run on Sicelore2 barcode-identified FASTQ and
    coupled with Longcell (Longcell+IsoQuant) for full-transcriptome simulation
- name: wf-single-cell
  role: Competing workflow-based quantification method (v0.2.9) benchmarked on simulated
    datasets
- name: seqtk
  role: Downsampling of simulated FASTQ files (v1.3) at sampling rates 0.2 to 1 to
    emulate different sequencing depths
- name: minimap2
  role: Splice-aware alignment of simulated reads to GRCh38 (v2.24) within the Longcell
    preprocessing pipeline
- name: igraph
  role: Louvain clustering of UMIs (v1.5.1) used to collapse PCR-duplicate reads from
    the simulation into consensus alignments
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:973913fd00ad37594f10603130b48bec2aa0554096fecf35088dc77fa63c8c13
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
  - FLAMES
  - IsoQuant
  - Longcell
  - Nanopore
  - Sicelore2
  - igraph
  - minimap2
  - seqtk
  - wf-single-cell
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# nanopore-long-read-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A benchmark-oriented simulation skill that generates synthetic single-cell Nanopore long-read FASTQ data with known ground-truth isoform expression, used in the Longcell paper to compare isoform quantification methods (Longcell, Sicelore2, FLAMES, IsoQuant, wf-single-cell) under controlled error, truncation, and depth regimes.

## When to use

Use when benchmarking or validating long-read isoform quantification pipelines where real data lack gold-standard isoform counts, and a simulated cell-by-isoform matrix with known error profiles is needed as ground truth.

## When NOT to use

- When real Nanopore data with independent experimental validation (e.g., targeted sequencing) are available and simulation artifacts would weaken the claim
- When the goal is biological discovery of isoform switching rather than method benchmarking — use the Longcell analysis package instead of the simulator
- When the quantification method cannot accept FASTQ with 10x-style 5' adapter/barcode/UMI structure, since the simulation design assumes that read architecture

## Inputs

- Gene expression matrix from the 10X CRCLM sample (cell x gene counts)
- MOB Nanopore Visium sample expression data for full-transcriptome simulation
- GRCh38 genome reference and Gencode v39 GTF-derived splice junction bed
- 10X-style adapter sequences, cell barcode whitelist, and UMI design
- Markov-chain error transition matrices for low, middle, and high Nanopore quality

## Outputs

- Simulated Nanopore-style FASTQ files with 10x adapter, cell barcode, UMI, truncation, and sequencing errors
- Ground-truth cell-by-isoform expression matrix
- benchmark_table.csv of mean per-cell Spearman correlations per method/quality level and MYL6 NAGNAG isoform-proportion MSE values
- Downsampled FASTQ files at sampling rates 0.2–1 for depth benchmarks

## How to apply

Simulate the cell-by-isoform expression matrix from a 10X CRCLM gene expression matrix (1000 isoforms drawn from 187 highly variable genes) or from the MOB Nanopore Visium sample for full-transcriptome runs. Generate full-length transcripts with a 10x-style adapter, cell barcode, and UMI attached at the 5' end, then apply Bernoulli-geometric end truncation (truncation probability 0.6, truncation length ~ Geometric(0.05)), GC-dependent amplification (A = k*sigmoid(0.5*(G-50)) + 2, default k = 20), a second pore-blocking truncation round, and four-state Markov-chain sequencing errors (correct/indel/deletion/wrong-base) using low-, middle-, and high-quality transition matrices. Emulate sequencing depth by downsampling the resulting FASTQ with seqtk v1.3 at sampling rates from 0.2 to 1. Feed the simulated FASTQ into each quantification method (Longcell pipeline: adapter search, k-mer barcode matching with k=6, minimap2 v2.24 splice alignment to GRCh38 with Gencode v39 junc-bed, igraph v1.5.1 Louvain UMI clustering; plus FLAMES, Sicelore2, wf-single-cell, IsoQuant), then join output cell-by-isoform matrices with ground truth to compute per-cell Spearman correlations and isoform-proportion MSE.

## Related tools

- **Longcell** (Quantification method under test; pipeline of adapter search, k-mer barcode matching (k=6), minimap2 splice alignment, and Louvain UMI clustering applied to simulated reads) — https://github.com/yuntianf/Longcell
- **Sicelore2** (Competing fixed-edit-distance UMI-collapsing quantification method benchmarked on simulated data)
- **FLAMES** (Competing isoform quantification method run with default parameters on simulated FASTQ)
- **IsoQuant** (Competing aligner/quantifier run on Sicelore2 barcode-identified FASTQ and coupled with Longcell (Longcell+IsoQuant) for full-transcriptome simulation)
- **wf-single-cell** (Competing workflow-based quantification method (v0.2.9) benchmarked on simulated datasets)
- **seqtk** (Downsampling of simulated FASTQ files (v1.3) at sampling rates 0.2 to 1 to emulate different sequencing depths)
- **minimap2** (Splice-aware alignment of simulated reads to GRCh38 (v2.24) within the Longcell preprocessing pipeline)
- **igraph** (Louvain clustering of UMIs (v1.5.1) used to collapse PCR-duplicate reads from the simulation into consensus alignments)

## Examples

```
devtools::install_github("yuntianf/Longcell", dependencies=TRUE)
```

## Evaluation signals

- Mean per-cell Spearman correlations between each method's output and the gold-standard matrix should reproduce 0.76/0.80/0.82 (low/middle/high quality) and 0.85/0.87 (full transcriptome), with Longcell at or near the top
- MYL6 NAGNAG isoform-proportion MSE should reproduce 1.02/1.22/1.48, with Longcell achieving the lowest MSE among compared methods
- Each method's output cell-by-isoform matrix must fully join with the ground-truth matrix by isoform identifiers before computing per-cell correlations
- Downsampled FASTQ at rate 1.0 should be identical to the unsampled simulated FASTQ; lower seqtk rates should yield approximately proportional read counts
- Simulated reads should carry 5'-end adapter, whitelisted cell barcodes, and UMIs, with truncation and error patterns matching the specified Bernoulli-geometric and Markov-chain parameters

## Limitations

- Simulation-based accuracy reflects the assumed error, truncation, GC-amplification, and Markov-chain models; real Nanopore libraries may deviate from these parameters
- Fixed-edit-distance UMI collapsing methods overestimate UMI counts under high sequencing error, which can bias comparison conclusions toward consensus-based approaches
- The 1000-isoform/187-gene CRCLM simulation and MOB Visium full-transcriptome simulation cover limited tissue and gene contexts; results may not generalize to other cell types or chemistries

## Evidence

- [readme] devtools::install_github("yuntianf/Longcell",dependencies=TRUE): [evidence span withheld — anchor status: not_found]
