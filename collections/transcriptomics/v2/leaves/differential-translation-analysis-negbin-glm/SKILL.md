---
name: differential-translation-analysis-negbin-glm
description: Use when the input is a merged ribosome-profiling count matrix (e.g.
  HTSeq-count output like count_matrix_by_transcript_IDs.csv) plus biological conditions,
  and the goal is calling up/downregulated features with DESeq2.
when_to_use_negative:
- Input is raw FastQ reads or BAM alignments rather than a per-feature count matrix
  — the counting step (HTSeq-count) must run first.
- The analysis goal is P-site offset prediction or periodicity QC (use riboWaltz/TRiP),
  not differential expression.
- The dataset lacks biological replicates suitable for negative-binomial dispersion
  estimation across the declared conditions.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3517
tools:
- name: DESeq2
  role: Performs the normalization and negative-binomial differential expression test
    on the Ribo-seq count matrix via estimateSizeFactors, estimateDispersions and
    nbinomWaldTest.
- name: RiboDoc
  role: Docker/Snakemake pipeline that orchestrates the DESeq2 differential analysis
    step, applies the configured p-val and logFC thresholds, and assembles tables,
    plots and the HTML report.
  repo: https://github.com/equipeGST/RiboDoc
- name: HTSeq
  role: Generates the per-transcript count matrix (count_matrix_by_transcript_IDs.csv)
    that serves as the input to the DESeq2 step.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.014
    title: 'RiboDoc: A Docker-based package for ribosome profiling analysis'
  source_content_hash: sha256:7b7949712d5c6b57133b8e0ac8158841c1e83031af49232e719f2dedba569468
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/equipeGST/RiboDoc
  repo_url: https://github.com/equipeGST/RiboDoc
  tools:
  - DESeq2
  - HTSeq
  - RiboDoc
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# differential-translation-analysis-negbin-glm

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run DESeq2's negative-binomial GLM (estimateSizeFactors, estimateDispersions, nbinomWaldTest) on ribosome-profiling count matrices to call differentially translated genes/transcripts, and produce result tables plus diagnostic figures and an HTML report as implemented in the RiboDoc pipeline.

## When to use

Use when the input is a merged ribosome-profiling count matrix (e.g. HTSeq-count output like count_matrix_by_transcript_IDs.csv) plus biological conditions, and the goal is calling up/downregulated features with DESeq2.

## When NOT to use

- Input is raw FastQ reads or BAM alignments rather than a per-feature count matrix — the counting step (HTSeq-count) must run first.
- The analysis goal is P-site offset prediction or periodicity QC (use riboWaltz/TRiP), not differential expression.
- The dataset lacks biological replicates suitable for negative-binomial dispersion estimation across the declared conditions.

## Inputs

- Merged ribosome profiling count matrix (e.g. count_matrix_by_transcript_IDs.csv from HTSeq-count)
- config.yaml with reference_condition, comparison conditions, p-val (e.g. 0.01) and logFC thresholds
- Names correspondence list mapping transcripts to genes

## Outputs

- Differential-expression result tables by gene (DESeq2_by_gene) and by transcript (DESeq2_by_transcript)
- Tables of up- and downregulated features respecting p-val and logFC thresholds
- Diagnostic plots: dispersion estimates, log2FoldChange, MA and volcano plots
- Replicate-validation statistics: PCA and pairwise Spearman correlation
- HTML report (e.g. Yeast_RiboSeq.Final_report.html) under the DESeq2_CDS.25-35 results folder

## How to apply

Load the merged Ribo-seq count matrix (count_matrix_by_transcript_IDs.csv produced by HTSeq-count) into R with the conditions declared in config.yaml (reference_condition such as 'Wild_Type' and comparison conditions such as 'Mutant'). Fit the DESeq2 negative-binomial model by sequentially applying estimateSizeFactors, estimateDispersions and nbinomWaldTest on the DDS object, which normalizes and compares the data. Apply the user-defined significance thresholds from config.yaml — a p-value threshold (p-val, e.g. '0.01') to reject the null hypothesis and a log2FoldChange threshold (logFC, e.g. '0') selecting only genes or transcripts with a sufficient expression ratio. Generate result tables grouped both by gene (DESeq2_by_gene) and by transcript (DESeq2_by_transcript) plus the names correspondence list. Produce diagnostic plots (dispersion estimates, log2FoldChange, MA and volcano plots) alongside replicate-validation statistics (PCA and Spearman correlation), and assemble everything into the HTML report (e.g. Yeast_RiboSeq.Final_report.html under the DESeq2_CDS.25-35 results folder).

## Related tools

- **DESeq2** (Performs the normalization and negative-binomial differential expression test on the Ribo-seq count matrix via estimateSizeFactors, estimateDispersions and nbinomWaldTest.)
- **RiboDoc** (Docker/Snakemake pipeline that orchestrates the DESeq2 differential analysis step, applies the configured p-val and logFC thresholds, and assembles tables, plots and the HTML report.) — https://github.com/equipeGST/RiboDoc
- **HTSeq** (Generates the per-transcript count matrix (count_matrix_by_transcript_IDs.csv) that serves as the input to the DESeq2 step.)

## Evaluation signals

- The DESeq2_CDS.25-35 output folder contains the HTML report (e.g. Yeast_RiboSeq.Final_report.html), count matrices, and tables for both by-gene and by-transcript groupings.
- Reported differential calls respect the configured p-val (e.g. 0.01) and logFC thresholds from config.yaml.
- Normalization and model fitting succeeded without errors from estimateSizeFactors, estimateDispersions and nbinomWaldTest on the DDS object.
- Diagnostic plots (dispersion, MA, volcano, log2FoldChange) are generated, and PCA/Spearman correlation validate replicate consistency.
- Up- and downregulated feature tables are non-empty and internally consistent with the names correspondence list.

## Limitations

- Thresholds (p-val, logFC, reference/comparison conditions) must be defined in config.yaml; misconfigured conditions or thresholds change the differential calls.
- Low-quality samples distort the model: RiboDoc may need to be rerun after sample removal, recomputing only the steps for which recalculation is required.
- The differential step depends on upstream RPF length selection (default 25-35 nt) and prior QC (read-length distribution, metagene periodicity, CDS enrichment) confirming active ribosome footprints.

## Evidence
