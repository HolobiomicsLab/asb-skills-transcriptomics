---
name: ribosome-profiling-count-matrix-normalization
description: Use when a ribosome profiling count matrix (e.g. count_matrix_by_transcript_IDs.csv
  from htseq-count) is available with replicate samples grouped into reference and
  comparison conditions, and differential translation/expression calls are needed.
when_to_use_negative:
- The input is raw FastQ reads and upstream steps (trimming, contaminant removal,
  alignment, htseq-count) have not yet been run — RiboDoc performs those first
- The goal is P-site offset estimation or periodicity diagnostics rather than differential
  expression — use riboWaltz/TRiP instead
- Samples have already been normalized or a feature table with differential calls
  already exists
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: DESeq2
  role: Normalizes counts and fits the negative-binomial GLM (estimateSizeFactors,
    estimateDispersions, nbinomWaldTest) to call differentially expressed genes/transcripts
  repo: https://github.com/equipeGST/RiboDoc
- name: RiboDoc
  role: Docker/Snakemake pipeline that orchestrates the DESeq2 step and generates
    tables, QC plots and the HTML report
  repo: https://github.com/equipeGST/RiboDoc
- name: HTSeq
  role: Produces the raw count matrix (htseq-count) that is the input to the DESeq2
    analysis
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.014
    title: 'RiboDoc: A Docker-based package for ribosome profiling analysis'
  source_content_hash: sha256:9baa277b1a98787b0dec3126ae0fb2cfa6cef10645d84b1532a8da4d3baa78b5
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
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# ribosome-profiling-count-matrix-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Normalize ribosome profiling count matrices and call differentially expressed genes/transcripts by running the DESeq2 negative-binomial GLM (estimateSizeFactors, estimateDispersions, nbinomWaldTest) on HTSeq-count-derived counts, applying user-configured p-val and log2FoldChange thresholds and packaging results with QC plots into an HTML report.

## When to use

Use when a ribosome profiling count matrix (e.g. count_matrix_by_transcript_IDs.csv from htseq-count) is available with replicate samples grouped into reference and comparison conditions, and differential translation/expression calls are needed.

## When NOT to use

- The input is raw FastQ reads and upstream steps (trimming, contaminant removal, alignment, htseq-count) have not yet been run — RiboDoc performs those first
- The goal is P-site offset estimation or periodicity diagnostics rather than differential expression — use riboWaltz/TRiP instead
- Samples have already been normalized or a feature table with differential calls already exists

## Inputs

- Merged ribosome profiling count matrix by transcript IDs (count_matrix_by_transcript_IDs.csv from htseq-count)
- config.yaml with reference_condition, comparison conditions, p-val (e.g. 0.01) and logFC (e.g. 0) thresholds
- Gene/transcript names correspondence list

## Outputs

- DESeq2 result tables grouped by gene (DESeq2_by_gene) and by transcript (DESeq2_by_transcript)
- Diagnostic plots: dispersion estimates, log2FoldChange, MA and volcano plots
- Replicate-validation statistics: PCA and pairwise Spearman correlation
- HTML report Yeast_RiboSeq.Final_report.html in the DESeq2_CDS.25-35 results folder

## How to apply

Load the merged ribosome profiling count matrix produced by HTSeq-count into R together with the conditions from config.yaml (reference_condition such as 'Wild_Type' and comparison conditions such as 'Mutant'). Run the DESeq2 negative-binomial GLM on the DDS object via the estimateSizeFactors, estimateDispersions and nbinomWaldTest subfunctions to normalize counts and fit the model. Apply the user-defined significance thresholds from config.yaml — the p-value threshold (p-val, e.g. 0.01) to reject the null hypothesis and the log2FoldChange threshold (logFC, e.g. 0) that selects only genes or transcripts with a sufficient expression ratio between the comparison and control groups. Generate result tables grouped by gene (DESeq2_by_gene) and by transcript (DESeq2_by_transcript) plus a names correspondence list, produce diagnostic plots (dispersion estimates, log2FoldChange, MA and volcano plots, PCA and Spearman correlation for replicate validation), and assemble everything into the HTML report Yeast_RiboSeq.Final_report.html under the DESeq2_CDS.25-35 results folder.

## Related tools

- **DESeq2** (Normalizes counts and fits the negative-binomial GLM (estimateSizeFactors, estimateDispersions, nbinomWaldTest) to call differentially expressed genes/transcripts) — https://github.com/equipeGST/RiboDoc
- **RiboDoc** (Docker/Snakemake pipeline that orchestrates the DESeq2 step and generates tables, QC plots and the HTML report) — https://github.com/equipeGST/RiboDoc
- **HTSeq** (Produces the raw count matrix (htseq-count) that is the input to the DESeq2 analysis)

## Examples

```
dds <- estimateSizeFactors(dds); dds <- estimateDispersions(dds); dds <- nbinomWaldTest(dds)
```

## Evaluation signals

- The DESeq2_CDS.25-35 output folder contains the HTML report Yeast_RiboSeq.Final_report.html, count matrices, and tables for both the by-gene and by-transcript groupings
- Reported differential calls respect the configured p-val and logFC thresholds from config.yaml
- Size factors, dispersions and Wald test statistics are present for all samples/genes, indicating estimateSizeFactors, estimateDispersions and nbinomWaldTest completed
- Replicate-validation outputs (PCA and pairwise Spearman correlation) confirm suitable replicates before trusting differential calls
- MA and volcano plots are consistent with the number of up- and downregulated features in the result tables

## Limitations

- Differential calls are only meaningful after QC (CDS enrichment, metagene periodicity, read-length ~28 nt) and replicate validation; low-quality samples must be removed and RiboDoc rerun for only the required steps
- Thresholds (p-val, logFC) and condition labels must be set in config.yaml; defaults like RPF length selection of 25-35 nt define which counts enter the analysis
- RiboDoc covers only the initial steps of RiboSeq analysis up to differential expression; deeper, more specific downstream analyses require other tools

## Evidence
