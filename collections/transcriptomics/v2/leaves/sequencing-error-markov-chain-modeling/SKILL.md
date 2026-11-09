---
name: sequencing-error-markov-chain-modeling
description: Use when benchmarking isoform quantification methods (e.g. Longcell,
  Sicelore2, FLAMES, wf-single-cell, IsoQuant) on simulated Nanopore reads where per-read
  error processes must be modeled at controlled low/middle/high quality regimes with
  a ground-truth cell-by-isoform matrix for scoring.
when_to_use_negative:
- Input is real Nanopore FASTQ data rather than a simulation — the gold-standard-correlation
  scoring this error model enables does not apply
- Short-read (Illumina) data, where error profiles and 10x 5'-attached full-length-read
  structure differ from the Nanopore four-state chain modeled here
- Evaluating biological splicing variation (e.g. isoform switching mechanisms) rather
  than computational-method accuracy under controlled error regimes
edam_operation: http://edamontology.org/operation_2426
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Longcell
  role: Simulated reads are benchmarked against Longcell's isoform quantification
    (barcode recovery, k-mer matching k=6, minimap2 alignment, Louvain UMI clustering)
  repo: github.com/yuntianf/Longcell
- name: Nanopore
  role: Sequencing platform whose per-base error characteristics (85–99% accuracy)
    the four-state Markov chain is calibrated to emulate
- name: seqtk
  role: Downsamples simulated FASTQ (v1.3, sampling rates 0.2 to 1) to emulate different
    sequencing depths
- name: minimap2
  role: Splice alignment (v2.24) of simulated reads against GRCh38 with the Gencode
    v39 junc-bed
- name: igraph
  role: Louvain UMI clustering (v1.5.1) used in the quantification being benchmarked
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-025-60902-2
    title: Single cell and spatial alternative splicing analysis with Nanopore long
      read sequencing
  source_content_hash: sha256:a60648f77d4ee259cca8a2d964fe1dfe8aedc843a0acca83b9070192db9afdf0
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
  - Nanopore
  - Sicelore2
  - igraph
  - minimap2
  - seqtk
  edam_operation: http://edamontology.org/operation_2426
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# sequencing-error-markov-chain-modeling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A simulation approach from the Longcell paper that models Nanopore sequencing as a four-state Markov chain (correct/insertion/deletion/wrong-base) at low, middle, and high quality transition matrices, layered onto simulated full-length transcripts so that single-cell isoform quantification methods can be benchmarked against a known gold-standard expression matrix.

## When to use

Use when benchmarking isoform quantification methods (e.g. Longcell, Sicelore2, FLAMES, wf-single-cell, IsoQuant) on simulated Nanopore reads where per-read error processes must be modeled at controlled low/middle/high quality regimes with a ground-truth cell-by-isoform matrix for scoring.

## When NOT to use

- Input is real Nanopore FASTQ data rather than a simulation — the gold-standard-correlation scoring this error model enables does not apply
- Short-read (Illumina) data, where error profiles and 10x 5'-attached full-length-read structure differ from the Nanopore four-state chain modeled here
- Evaluating biological splicing variation (e.g. isoform switching mechanisms) rather than computational-method accuracy under controlled error regimes

## Inputs

- Ground-truth cell-by-isoform expression matrix (e.g. simulated from the 10X CRCLM gene expression matrix or the MOB Nanopore Visium sample)
- Reference genome (GRCh38) and annotation (Gencode v39 GTF / junc-bed)
- Low, middle, and high quality four-state Markov-chain error transition matrices
- 10x-style barcode whitelist for read simulation

## Outputs

- Simulated Nanopore FASTQ reads with 5' adapter, cell barcode, UMI, truncation, GC-biased amplification, and Markov-chain sequencing errors
- Simulated reads downsampled with seqtk v1.3 at sampling rates 0.2–1 for sequencing-depth analysis
- Benchmark table (benchmark_table.csv) of mean per-cell Spearman correlations and MYL6 NAGNAG MSE values per method per error regime

## How to apply

Simulate a cell-by-isoform expression matrix (e.g. 1000 isoforms from 187 highly variable genes of the 10X CRCLM matrix), then generate full-length transcripts with a 10x-style adapter, cell barcode, and UMI at the 5' end. Apply Bernoulli-geometric end truncation (truncation probability 0.6, truncation length ~ Geometric(0.05)), GC-dependent amplification (A = k*sigmoid(0.5*(G-50)) + 2, default k = 20), and a second pore-blocking truncation round before sequencing. Emit reads by drawing per-base states from a four-state Markov chain — correct/indel/del/wrong-base — using separate low, middle, and high quality transition matrices, which mirrors the 85–99% per-base accuracy of real Nanopore data. Run each quantification method on the simulated FASTQ and join its output cell-by-isoform matrix with the gold-standard matrix to compute per-cell Spearman correlations; vary error regime to test robustness.

## Related tools

- **Longcell** (Simulated reads are benchmarked against Longcell's isoform quantification (barcode recovery, k-mer matching k=6, minimap2 alignment, Louvain UMI clustering)) — github.com/yuntianf/Longcell
- **Nanopore** (Sequencing platform whose per-base error characteristics (85–99% accuracy) the four-state Markov chain is calibrated to emulate)
- **seqtk** (Downsamples simulated FASTQ (v1.3, sampling rates 0.2 to 1) to emulate different sequencing depths)
- **minimap2** (Splice alignment (v2.24) of simulated reads against GRCh38 with the Gencode v39 junc-bed)
- **igraph** (Louvain UMI clustering (v1.5.1) used in the quantification being benchmarked)

## Examples

```
seqtk sample -s100 sim_reads.fq 0.5 > sim_reads_ds.fq
```

## Evaluation signals

- Reproduced mean per-cell Spearman correlations should match reported 0.76/0.80/0.82 for low/middle/high quality data
- Full-transcriptome downsampling correlations should match reported 0.85/0.87 (Longcell+IsoQuant best at 0.87)
- MYL6 NAGNAG isoform-proportion MSE should match 1.02/1.22/1.48 across the three error regimes
- Simulated read properties must reflect the pipeline parameters: truncation probability 0.6 with Geometric(0.05) lengths, GC amplification factor A = k*sigmoid(0.5*(G-50)) + 2 with k = 20
- Method output matrices must fully join with the gold-standard matrix by isoform before per-cell correlation is computed

## Limitations

- Simulation relies on simplified error and truncation models (four-state Markov chain, Bernoulli-geometric truncation, GC-dependent amplification) that may not capture all real Nanopore error structure
- Ground-truth-correlation benchmarking is only possible on simulated data; the article notes the influence of sequencing errors, read truncation, and mapping errors on single-cell long-read isoform quantification had not been characterized previously
- Downsampling experiments are limited to the simulated fastq files with seqtk v1.3 at sampling rates 0.2 to 1

## Evidence

- [other] four-state Markov-chain error model at three quality tiers: [evidence span withheld — anchor status: not_found]
- [other] Simulation pipeline with truncation and amplification steps: [evidence span withheld — anchor status: not_found]
