---
name: statistical-threshold-filtering-gene-expression
description: Use when a RiboSeq pipeline has produced normalized count matrices (e.g.
  HTSeq-count count_matrix_by_transcript_IDs.csv) and you must define significance
  cutoffs (p-val, logFC) to accept/reject differential expression calls against a
  reference condition.
when_to_use_negative:
- Input is raw FastQ reads or untrimmed alignments — the DESeq2 statistical step requires
  the HTSeq-count matrix that only exists after trimming (Cutadapt), contaminant removal
  (Bowtie2), alignment (Hisat2/Bowtie2) and counting upstream.
- The goal is P-site offset estimation or periodicity QC rather than differential
  expression calling — use the riboWaltz/TRiP module instead.
- Sample replicates have already been flagged as low-quality and removed; RiboDoc
  should be rerun first with only the required recalculated steps before re-thresholding
  results.
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: DESeq2
  role: Normalizes ribosome-profiling counts and fits the negative-binomial GLM (estimateSizeFactors,
    estimateDispersions, nbinomWaldTest) whose Wald p-values and log2FoldChange are
    thresholded to call differential expression.
  repo: https://genomebiology.biomedcentral.com/articles/10.1186/s13059-014-0550-8
- name: RiboDoc
  role: Docker/Snakemake pipeline that orchestrates the DESeq2 differential-expression
    step, applies config.yaml thresholds, and assembles the result tables and HTML
    report.
  repo: https://github.com/equipeGST/RiboDoc
- name: HTSeq
  role: Produces the count_matrix_by_transcript_IDs.csv count matrix that serves as
    the input to DESeq2 differential analysis.
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.014
    title: 'RiboDoc: A Docker-based package for ribosome profiling analysis'
  source_content_hash: sha256:97d3d0d272ec8d5055df686c695cd4a445c4fb525194206ec6b077067d734d55
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/equipeGST/RiboDoc
  tools:
  - DESeq2
  - HTSeq
  - RiboDoc
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# statistical-threshold-filtering-gene-expression

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Calls differentially expressed genes or transcripts in RiboDoc's quantitative module by applying user-configured p-value (p-val, e.g. 0.01) and log2FoldChange (logFC, e.g. 0) thresholds to DESeq2 negative-binomial Wald test results computed on ribosome-profiling count matrices.

## When to use

Use when a RiboSeq pipeline has produced normalized count matrices (e.g. HTSeq-count count_matrix_by_transcript_IDs.csv) and you must define significance cutoffs (p-val, logFC) to accept/reject differential expression calls against a reference condition.

## When NOT to use

- Input is raw FastQ reads or untrimmed alignments — the DESeq2 statistical step requires the HTSeq-count matrix that only exists after trimming (Cutadapt), contaminant removal (Bowtie2), alignment (Hisat2/Bowtie2) and counting upstream.
- The goal is P-site offset estimation or periodicity QC rather than differential expression calling — use the riboWaltz/TRiP module instead.
- Sample replicates have already been flagged as low-quality and removed; RiboDoc should be rerun first with only the required recalculated steps before re-thresholding results.

## Inputs

- count_matrix_by_transcript_IDs.csv — merged ribosome profiling count matrix produced by HTSeq-count
- config.yaml with reference_condition (e.g. 'Wild_Type'), comparison conditions (e.g. 'Mutant'), p-val (e.g. '0.01') and logFC (e.g. '0') thresholds
- gzip-compressed sample metadata implied by FastQ naming convention [CONDITION]_[NAME].[REPLICATE].fastq.gz

## Outputs

- DESeq2_by_gene and DESeq2_by_transcript result tables of up- and downregulated features
- Names correspondence list between genes and transcripts
- QC and diagnostic plots: dispersion estimates, log2FoldChange, MA and volcano plots; PCA and Spearman correlation replicate-validation statistics
- HTML report Yeast_RiboSeq.Final_report.html under the DESeq2_CDS.25-35 results folder

## How to apply

Load the merged ribosome profiling count matrix (count_matrix_by_transcript_IDs.csv from HTSeq-count) into R with conditions parsed from config.yaml (reference_condition such as 'Wild_Type' and comparison conditions such as 'Mutant'). Normalize counts and fit the negative-binomial GLM in DESeq2 via estimateSizeFactors, estimateDispersions and nbinomWaldTest, then apply the user-defined p-val threshold from config.yaml to reject the null hypothesis and the logFC threshold to keep only genes or transcripts with a sufficient expression ratio between comparison and control groups. Generate result tables grouped both by gene (DESeq2_by_gene) and by transcript (DESeq2_by_transcript) plus a names-correspondence list, and assemble tables and figures (dispersion, log2FoldChange, MA, volcano, PCA, Spearman correlation) into the HTML report under the DESeq2_CDS.25-35 results folder. The dual-threshold design rationale is that p-value alone does not capture effect size, so log2FoldChange filtering complements statistical significance.

## Related tools

- **DESeq2** (Normalizes ribosome-profiling counts and fits the negative-binomial GLM (estimateSizeFactors, estimateDispersions, nbinomWaldTest) whose Wald p-values and log2FoldChange are thresholded to call differential expression.) — https://genomebiology.biomedcentral.com/articles/10.1186/s13059-014-0550-8
- **RiboDoc** (Docker/Snakemake pipeline that orchestrates the DESeq2 differential-expression step, applies config.yaml thresholds, and assembles the result tables and HTML report.) — https://github.com/equipeGST/RiboDoc
- **HTSeq** (Produces the count_matrix_by_transcript_IDs.csv count matrix that serves as the input to DESeq2 differential analysis.)

## Evaluation signals

- The DESeq2_CDS.25-35 output folder contains the HTML report (Yeast_RiboSeq.Final_report.html), count matrices, and tables for both by-gene and by-transcript groupings.
- Reported differential calls respect the configured p-val (e.g. 0.01) and logFC (e.g. 0) thresholds from config.yaml.
- Up- and downregulated feature tables exist for both DESeq2_by_gene and DESeq2_by_transcript groupings plus the names correspondence list.
- Diagnostic plots (dispersion, log2FoldChange, MA, volcano) and replicate-validation statistics (PCA, Spearman correlation) are present in the report.
- Size-factor normalization and dispersion estimates completed without errors, i.e. the DESeq2 estimateSizeFactors/estimateDispersions/nbinomWaldTest chain ran on the DDS object.

## Limitations

- Thresholds are entirely user-specified in config.yaml (p-val, logFC); no default significance guidance is enforced by the pipeline.
- Differential calling at the transcript level depends on reads being selected in the 25-35 nt RPF length window and on sufficient aligned material; datasets with poor alignment may not support downstream quantitative analysis.
- Low-quality samples must be removed and RiboDoc rerun before differential results can be trusted; the pipeline supports rerunning only the steps needing recalculation.

## Evidence
