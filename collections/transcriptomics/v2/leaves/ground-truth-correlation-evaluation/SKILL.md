---
name: ground-truth-correlation-evaluation
description: Use when a simulation provides a known cell-by-isoform ground truth and
  you need to quantify how accurately a long-read isoform quantification method recovers
  per-cell isoform expression across error regimes or sequencing depths.
when_to_use_negative:
- Input is real experimental data with no known ground-truth cell-by-isoform matrix
  — per-cell correlation against gold standard is undefined
- Evaluating methods on downstream analyses (isoform switching, differential splicing)
  rather than raw quantification accuracy
- Comparing methods that emit incompatible identifiers without a full isoform-level
  join; the correlation is only valid after fully joining matrices by isoform
edam_operation: http://edamontology.org/operation_3478
edam_topics:
- http://edamontology.org/topic_0102
- http://edamontology.org/topic_3170
tools:
- name: Longcell
  role: Method under benchmark; recovers barcodes/UMIs and quantifies isoforms from
    simulated Nanopore reads
  repo: github.com/yuntianf/Longcell
- name: Sicelore2
  role: Competing fixed-edit-distance UMI-collapsing method benchmarked against Longcell
- name: FLAMES
  role: Competing isoform-quantification method (git commit 774e16ae), given the barcode
    whitelist
- name: wf-single-cell
  role: Competing method (v0.2.9) run with default parameters on simulated data
- name: IsoQuant
  role: Competing method (v3.3) fed Sicelore2 barcode-identified FASTQ; also coupled
    with Longcell for full-transcriptome simulation
- name: seqtk
  role: v1.3 downsampling of simulated FASTQs at rates 0.2–1 to emulate sequencing
    depths
- name: minimap2
  role: v2.24 splice alignment against GRCh38 with Gencode v39 junc-bed within the
    Longcell pipeline
- name: igraph
  role: v1.5.1 Louvain UMI clustering inside Longcell before quantification
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:5ece65231329893cf182503cc7666ce47cd1521db84078ad8ce9c8ebf390edb0
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
  - igraph
  - minimap2
  - seqtk
  - wf-single-cell
  edam_operation: http://edamontology.org/operation_3478
  edam_topics:
  - http://edamontology.org/topic_0102
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# ground-truth-correlation-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark isoform-quantification accuracy by fully joining each method's cell-by-isoform count matrix with a simulated gold-standard matrix and computing per-cell Spearman correlations, plus isoform-proportion MSE for the 4-isoform/240-cell MYL6 NAGNAG simulation.

## When to use

Use when a simulation provides a known cell-by-isoform ground truth and you need to quantify how accurately a long-read isoform quantification method recovers per-cell isoform expression across error regimes or sequencing depths.

## When NOT to use

- Input is real experimental data with no known ground-truth cell-by-isoform matrix — per-cell correlation against gold standard is undefined
- Evaluating methods on downstream analyses (isoform switching, differential splicing) rather than raw quantification accuracy
- Comparing methods that emit incompatible identifiers without a full isoform-level join; the correlation is only valid after fully joining matrices by isoform

## Inputs

- Simulated cell-by-isoform gold-standard expression matrix (from 10X CRCLM or MOB Nanopore Visium gene expression)
- Simulated Nanopore FASTQ files at low/middle/high quality and at seqtk v1.3 downsampling rates 0.2–1
- Per-method output cell-by-isoform count matrices (Longcell, Longcell+IsoQuant, FLAMES, Sicelore2, wf-single-cell, IsoQuant)
- 10X barcode whitelist supplied to FLAMES, Sicelore2, and Longcell
- MYL6 NAGNAG simulation (4 isoforms, 240 cells) ground-truth isoform proportions

## Outputs

- Per-cell Spearman correlation between ground truth and each method's output matrix
- Mean per-cell correlation per method per data quality and per sequencing depth
- Mean isoform-proportion MSE per method for the MYL6 NAGNAG simulation
- benchmark_table.csv aggregating correlations and MSE across methods

## How to apply

Simulate cell-by-isoform matrices from the 10X CRCLM gene expression matrix (1000 isoforms from 187 highly variable genes) and from the MOB Nanopore Visium sample, generating reads with 10x-style adapters, cell barcodes, and UMIs plus Bernoulli-geometric truncation (p=0.6, length ~ Geometric(0.05)), GC-dependent amplification, and low/middle/high-quality error regimes. Run each method (Longcell, Longcell+IsoQuant, FLAMES, Sicelore2, wf-single-cell, IsoQuant) on the simulated FASTQs, then fully join each output matrix with ground truth by isoform and compute the per-cell correlation between ground truth and output for every method. For the MYL6 4-isoform/240-cell NAGNAG simulation, compute the mean isoform-proportion MSE across the three error regimes. Aggregate mean per-cell correlations (low/middle/high quality; 0.2–1 seqtk v1.3 downsampling rates for depth) and MYL6 MSE values into benchmark_table.csv and rank methods. Correct application is judged by reproducing the reported correlations 0.76/0.80/0.82 (low/middle/high) and 0.85/0.87 (full transcriptome) and MSE 1.02/1.22/1.48, with Longcell at or near the top.

## Related tools

- **Longcell** (Method under benchmark; recovers barcodes/UMIs and quantifies isoforms from simulated Nanopore reads) — github.com/yuntianf/Longcell
- **Sicelore2** (Competing fixed-edit-distance UMI-collapsing method benchmarked against Longcell)
- **FLAMES** (Competing isoform-quantification method (git commit 774e16ae), given the barcode whitelist)
- **wf-single-cell** (Competing method (v0.2.9) run with default parameters on simulated data)
- **IsoQuant** (Competing method (v3.3) fed Sicelore2 barcode-identified FASTQ; also coupled with Longcell for full-transcriptome simulation)
- **seqtk** (v1.3 downsampling of simulated FASTQs at rates 0.2–1 to emulate sequencing depths)
- **minimap2** (v2.24 splice alignment against GRCh38 with Gencode v39 junc-bed within the Longcell pipeline)
- **igraph** (v1.5.1 Louvain UMI clustering inside Longcell before quantification)

## Examples

```
seqtk sample simulated.fastq 0.5 > downsampled.fastq  # then per method: cor(method_counts[i, ], truth[i, ], method='spearman') across fully-joined isoforms
```

## Evaluation signals

- Mean per-cell Spearman correlations match reported values 0.76/0.80/0.82 for low/middle/high quality and 0.85/0.87 for the full-transcriptome downsampling
- MYL6 NAGNAG mean MSE values match 1.02 (Longcell), 1.22, and 1.48, with Longcell lowest
- Every method's output matrix fully joins with the ground-truth matrix by isoform with no unmatched rows before computing per-cell correlations
- Longcell ranks at or near the top of the compared methods on both correlation and MSE metrics
- Downsampled full-transcriptome results (seqtk 0.2–1) show a monotonic or near-monotonic trend in correlation with sequencing depth

## Limitations

- Ground truth exists only in simulation; correlations quantify accuracy against simulated expression, not real data
- Results depend on the specific error model: Bernoulli-geometric truncation (p=0.6, Geometric(0.05)), GC-dependent amplification (A = k*sigmoid(0.5*(G-50)) + 2, k=20), and the four-state Markov error transition matrices
- Sequencing errors, read truncation, and mapping errors inflate UMI counts in fixed-edit-distance methods, which is the very artifact this benchmark is designed to expose

## Evidence

- [other] On simulated single-cell Nanopore data, Longcell achieves the highest correlation with the known gold-standard isoform expression: [evidence span withheld — anchor status: not_found]
- [methods] Fully join each method's output cell-by-isoform count matrix with the ground-truth matrix by isoform: [evidence span withheld — anchor status: not_found]
- [methods] downsample the resulting fastq files with seqtk v1.3 at sampling rates from 0.2 to 1 to emulate different sequencing depths: [evidence span withheld — anchor status: not_found]
