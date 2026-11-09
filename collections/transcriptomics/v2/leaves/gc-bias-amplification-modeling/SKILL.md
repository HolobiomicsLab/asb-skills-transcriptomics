---
name: gc-bias-amplification-modeling
description: Use when simulating Nanopore single-cell/spatial long-read FASTQ from
  a known cell-by-isoform matrix and you need realistic GC-dependent amplification,
  truncation, and error profiles to benchmark isoform quantification accuracy.
when_to_use_negative:
- When analyzing real Nanopore reads rather than generating synthetic benchmarks —
  the amplification and error models only apply to simulated data
- When the analysis goal is isoform switching or PSI inference from existing data
  (use Longcell/LongcellPre analysis functions instead of simulation)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Longcell
  role: Isoform quantification method evaluated on the GC-bias-simulated data; its
    R package also underpins downstream benchmarking analysis
  repo: github.com/yuntianf/Longcell
- name: seqtk
  role: Downsamples simulated FASTQ files (v1.3, sampling rates 0.2–1) to emulate
    different sequencing depths after amplification modeling
- name: minimap2
  role: v2.24 splice alignment of simulated reads against GRCh38 with Gencode v39
    junc-bed used by quantification methods
- name: Sicelore2
  role: Competing isoform quantifier (v1) benchmarked against Longcell on the simulated
    data
- name: FLAMES
  role: Competing isoform quantifier (git commit 774e16ae) benchmarked on the simulated
    datasets
- name: IsoQuant
  role: Competing quantifier (v3.3); Longcell+IsoQuant variant evaluated on full-transcriptome
    simulations
- name: wf-single-cell
  role: Competing quantifier (v0.2.9) benchmarked on the simulated datasets
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:5c031bcc2d93f63c470f6a092d955e864d74f6a3b00b7395eed63067cfa4b776
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 13
---

# gc-bias-amplification-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A simulation component that models GC-dependent PCR amplification bias when generating synthetic single-cell Nanopore full-length reads, alongside Bernoulli-geometric end truncation and four-state Markov-chain sequencing errors. It enables realistic gold-standard benchmarks (Spearman correlation, MYL6 NAGNAG MSE) for isoform quantification tools like Longcell, Sicelore2, FLAMES, IsoQuant, and wf-single-cell.

## When to use

Use when simulating Nanopore single-cell/spatial long-read FASTQ from a known cell-by-isoform matrix and you need realistic GC-dependent amplification, truncation, and error profiles to benchmark isoform quantification accuracy.

## When NOT to use

- When analyzing real Nanopore reads rather than generating synthetic benchmarks — the amplification and error models only apply to simulated data
- When the analysis goal is isoform switching or PSI inference from existing data (use Longcell/LongcellPre analysis functions instead of simulation)

## Inputs

- Cell-by-gene expression matrix from a 10X sample (e.g., 10X CRCLM) or Nanopore Visium sample (e.g., MOB) as the ground-truth basis
- Isoform sequences and annotations (GRCh38 genome, Gencode v39 GTF/junc-bed)
- Sequencing-quality transition matrices (low, middle, high) for four-state Markov-chain errors
- Amplification parameters (GC-sigmoid coefficient k, default 20; truncation probability 0.6; Geometric(0.05) truncation length)

## Outputs

- Simulated full-length Nanopore FASTQ reads with GC-dependent amplification, 10x adapter/barcode/UMI, truncation, and quality-stratified errors
- Gold-standard cell-by-isoform count matrix for full-join correlation/MSE evaluation
- benchmark_table.csv aggregating mean per-cell Spearman correlations and MYL6 NAGNAG isoform-proportion MSE per method

## How to apply

Simulate the cell-by-isoform expression matrix from a 10X gene expression matrix (e.g., 1000 isoforms drawn from 187 highly variable genes of the CRCLM sample), then generate full-length transcripts with 10x-style adapter, cell barcode, and UMI attached at the 5' end. Apply GC-dependent amplification with A = k*sigmoid(0.5*(G-50)) + 2 using default k = 20, combined with Bernoulli-geometric end truncation (truncation probability 0.6, truncation length ~ Geometric(0.05)) and a second pore-blocking truncation round. Introduce four-state Markov-chain (correct/indel/del/wrong-base) sequencing errors using low-, middle-, and high-quality transition matrices. Downsample the resulting FASTQ with seqtk v1.3 (rates 0.2–1) to emulate sequencing depth variation, then evaluate downstream quantifiers by per-cell Spearman correlation against the gold-standard matrix. Rationale: capturing these error/coverage artifacts ensures benchmark rankings (e.g., Longcell's 0.76/0.80/0.82 mean per-cell Spearman correlations) reflect true robustness rather than idealized reads.

## Related tools

- **Longcell** (Isoform quantification method evaluated on the GC-bias-simulated data; its R package also underpins downstream benchmarking analysis) — github.com/yuntianf/Longcell
- **seqtk** (Downsamples simulated FASTQ files (v1.3, sampling rates 0.2–1) to emulate different sequencing depths after amplification modeling)
- **minimap2** (v2.24 splice alignment of simulated reads against GRCh38 with Gencode v39 junc-bed used by quantification methods)
- **Sicelore2** (Competing isoform quantifier (v1) benchmarked against Longcell on the simulated data)
- **FLAMES** (Competing isoform quantifier (git commit 774e16ae) benchmarked on the simulated datasets)
- **IsoQuant** (Competing quantifier (v3.3); Longcell+IsoQuant variant evaluated on full-transcriptome simulations)
- **wf-single-cell** (Competing quantifier (v0.2.9) benchmarked on the simulated datasets)

## Examples

```
from scipy.special import expit; A = 20*expit(0.5*(G-50)) + 2  # GC-dependent amplification for transcripts with G GC-content
```

## Evaluation signals

- Reproduced mean per-cell Spearman correlations on simulated low/middle/high-quality data match 0.76/0.80/0.82 for Longcell, and 0.85/0.87 for full-transcriptome/downsampling runs
- MYL6 NAGNAG (4-isoform/240-cell) isoform-proportion MSE matches 1.02/1.22/1.48 across the three error regimes, with Longcell lowest (1.02 vs 1.22 and 1.48)
- Output FASTQ reads carry 10x-style adapter, cell barcode, and UMI at the 5' end, and exhibit truncation probability 0.6 with Geometric(0.05) truncation lengths
- Each method's cell-by-isoform matrix fully joins the gold-standard matrix by isoform before computing per-cell correlations
- Downsampling sweep (seqtk rates 0.2–1) yields monotonically degrading correlation with decreasing depth

## Limitations

- Benchmark validity depends on the assumed error/amplification models; the influence of sequencing errors, read truncation, and mapping errors on single-cell long-read isoform quantification had not been characterized prior to this study
- Biological mechanisms underlying observed isoform patterns remain unexplained and require further research
- No changelog or versioned release information was found for the Longcell repository, complicating reproducibility of the exact simulation code state

## Evidence

- [methods] GC-dependent amplification with Bernoulli-geometric truncation: [evidence span withheld — anchor status: not_found]
- [methods] Markov-chain sequencing error model: [evidence span withheld — anchor status: not_found]
- [readme] Longcell installation for reproducible simulation/benchmark environment: [evidence span withheld — anchor status: external_doc]
