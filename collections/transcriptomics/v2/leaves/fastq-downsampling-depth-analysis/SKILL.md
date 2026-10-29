---
name: fastq-downsampling-depth-analysis
description: 'Use when quantifying how isoform-quantification accuracy degrades with
  sequencing depth: you have simulated Nanopore FASTQ reads with 10x-style cell barcode/UMI,
  a ground-truth cell-by-isoform matrix, and need depth-response curves of per-cell
  accuracy.'
when_to_use_negative:
- Real (non-simulated) Nanopore reads with no ground-truth isoform expression matrix
  — per-cell correlation to truth cannot be computed
- The question is the effect of per-base sequencing error quality (low/middle/high
  transition matrices), which is benchmarked by error regime rather than by downsampling
- Bulk or spatial analysis goals (e.g., spatial isoform switching with LongcellPre/Longcell
  Splice objects) rather than method benchmarking
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: seqtk
  role: Subsamples simulated FASTQ files (v1.3) at rates 0.2–1 to emulate different
    sequencing depths
- name: Longcell
  role: Quantifies isoform expression from each downsampled FASTQ; depth-response
    is measured as its per-cell correlation to ground truth
  repo: github.com/yuntianf/Longcell
- name: IsoQuant
  role: Coupled with Longcell (Longcell+IsoQuant variant) as the best-performing full-transcriptome
    quantification configuration (0.87 at full depth)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:bab1ca546e1badb6353bbde01fe114c916f3642a6a4d2a5bfffd06dea19e56cb
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# fastq-downsampling-depth-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark long-read single-cell isoform quantification accuracy as a function of sequencing depth by subsampling simulated Nanopore FASTQ reads with seqtk and correlating method outputs against a simulated ground-truth cell-by-isoform matrix. It isolates the effect of read depth from the effects of read quality (error regime).

## When to use

Use when quantifying how isoform-quantification accuracy degrades with sequencing depth: you have simulated Nanopore FASTQ reads with 10x-style cell barcode/UMI, a ground-truth cell-by-isoform matrix, and need depth-response curves of per-cell accuracy.

## When NOT to use

- Real (non-simulated) Nanopore reads with no ground-truth isoform expression matrix — per-cell correlation to truth cannot be computed
- The question is the effect of per-base sequencing error quality (low/middle/high transition matrices), which is benchmarked by error regime rather than by downsampling
- Bulk or spatial analysis goals (e.g., spatial isoform switching with LongcellPre/Longcell Splice objects) rather than method benchmarking

## Inputs

- Simulated Nanopore FASTQ reads with 10x-style adapter, cell barcode, and UMI at the 5' end (e.g., generated from the MOB Nanopore Visium sample)
- Ground-truth cell-by-isoform expression matrix used for simulation
- seqtk v1.3 for FASTQ subsampling
- Reference GRCh38 genome and Gencode v39 splice-junction bed

## Outputs

- Downsampled FASTQ files at each sampling rate (0.2 to 1)
- Per-method cell-by-isoform count matrices for each depth point
- Mean per-cell Spearman correlation (ground truth vs. method output) as a function of sequencing depth

## How to apply

First simulate the full-transcriptome cell-by-isoform matrix (e.g., from the MOB Nanopore Visium sample) with the 10x-style adapter/barcode/UMI simulation pipeline at high data quality, so each read has a known isoform of origin. Then downsample the resulting FASTQ files with seqtk v1.3 at sampling rates from 0.2 to 1 to emulate different sequencing depths. Run Longcell (adapter search, k-mer cell-barcode matching with k=6, minimap2 v2.24 splice alignment against GRCh38 with the Gencode v39 junc-bed, igraph Louvain UMI clustering) plus its IsoQuant-coupled variant, and competitors (FLAMES, Sicelore2, wf-single-cell), on every downsampled file. Fully join each method's output cell-by-isoform count matrix with the ground truth by isoform and compute per-cell correlations between ground truth and output. Plot mean per-cell correlation versus sampling rate; the article reports Longcell at 0.85 and Longcell+IsoQuant at 0.87 mean correlation on the full-transcriptome simulation at full depth.

## Related tools

- **seqtk** (Subsamples simulated FASTQ files (v1.3) at rates 0.2–1 to emulate different sequencing depths)
- **Longcell** (Quantifies isoform expression from each downsampled FASTQ; depth-response is measured as its per-cell correlation to ground truth) — github.com/yuntianf/Longcell
- **IsoQuant** (Coupled with Longcell (Longcell+IsoQuant variant) as the best-performing full-transcriptome quantification configuration (0.87 at full depth))

## Examples

```
seqtk sample -s 1 simulated_highq.fastq 0.2 > simulated_highq_s0.2.fastq
```

## Evaluation signals

- At full depth (sampling rate 1), reproduced mean per-cell correlations match the reported 0.85 (Longcell) and 0.87 (Longcell+IsoQuant) on the full-transcriptome simulation
- Per-cell correlation increases monotonically (with diminishing returns) as the seqtk sampling rate rises from 0.2 to 1
- Each method's count matrix fully joins with the ground-truth matrix by isoform before correlation is computed (no isoform dropped by an inner join)
- Sampling rates cover the exact range 0.2 to 1 as produced by seqtk v1.3 downsampling
- As a cross-check on the same simulation framework, MYL6 NAGNAG isoform-proportion MSE reproduces 1.02/1.22/1.48 across error regimes

## Limitations

- Accuracy-vs-depth conclusions rest on simulated reads (Bernoulli-geometric end truncation with truncation probability 0.6, GC-dependent amplification, Markov-chain sequencing errors), not real Nanopore data
- Depth range is limited to sampling rates 0.2–1 of the simulated high-quality dataset; behavior below 20% depth is not characterized
- Fixed-edit-distance UMI-collapsing competitors (Sicelore2, FLAMES) overestimate UMI counts under high sequencing error, which inflates their quantification and can confound depth comparisons

## Evidence

- [methods] downsample the resulting fastq files with seqtk v1.3 at sampling rates from 0.2 to 1 to emulate different sequencing depths: [evidence span withheld — anchor status: not_found]
