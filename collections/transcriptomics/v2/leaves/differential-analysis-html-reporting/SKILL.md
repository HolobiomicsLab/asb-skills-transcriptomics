---
name: differential-analysis-html-reporting
description: Use when a ribosome profiling pipeline has produced a per-transcript
  count matrix (e.g. from htseq-count) and conditions are defined in config.yaml,
  and you need DESeq2-based differential calls plus a consolidated HTML report.
when_to_use_negative:
- Input is already a differential-expression result table or a normalized expression
  matrix — this skill expects raw per-transcript counts from htseq-count.
- Samples failed replicate validation (PCA / Spearman correlation) or lack CDS enrichment
  / metagene periodicity; RiboDoc guidance is to remove low-quality samples and rerun
  before trusting differential calls.
- Only P-site offset estimation or read-length QC is needed — use riboWaltz/TRiP-based
  skills instead of the DESeq2 reporting step.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: RiboDoc
  role: Docker/Snakemake pipeline that orchestrates the DESeq2 differential expression
    step and assembles the final HTML report from config.yaml parameters
  repo: https://github.com/equipeGST/RiboDoc
- name: DESeq2
  role: R package that normalizes counts (estimateSizeFactors), fits dispersions (estimateDispersions)
    and runs the nbinomWaldTest for differential expression
  repo: https://genomebiology.biomedcentral.com/articles/10.1186/s13059-014-0550-8
- name: HTSeq
  role: htseq-count generates the per-transcript count matrix consumed by the DESeq2
    step
  repo: https://htseq.readthedocs.io/en/release_0.11.1/count.html#
- name: riboWaltz
  role: 'Upstream P-site offset prediction whose 25-35 nt RPF-selected outputs feed
    the quantitative analysis (alternative: TRiP)'
  repo: https://github.com/LabTranslationalArchitectomics/riboWaltz
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.csbj.2021.05.014
    title: 'RiboDoc: A Docker-based package for ribosome profiling analysis'
  source_content_hash: sha256:3b70a99ee3bb4ad93d222807efe06f887af71dece172ab6ebe3075e0a55e4b51
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
  - riboWaltz
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# differential-analysis-html-reporting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run DESeq2 differential expression on RiboDoc's ribosome profiling count matrix (estimateSizeFactors, estimateDispersions, nbinomWaldTest), apply user-configured p-val and log2FoldChange thresholds, and assemble result tables and QC/diagnostic plots into an HTML report.

## When to use

Use when a ribosome profiling pipeline has produced a per-transcript count matrix (e.g. from htseq-count) and conditions are defined in config.yaml, and you need DESeq2-based differential calls plus a consolidated HTML report.

## When NOT to use

- Input is already a differential-expression result table or a normalized expression matrix — this skill expects raw per-transcript counts from htseq-count.
- Samples failed replicate validation (PCA / Spearman correlation) or lack CDS enrichment / metagene periodicity; RiboDoc guidance is to remove low-quality samples and rerun before trusting differential calls.
- Only P-site offset estimation or read-length QC is needed — use riboWaltz/TRiP-based skills instead of the DESeq2 reporting step.

## Inputs

- count_matrix_by_transcript_IDs.csv — merged ribosome profiling count matrix produced by htseq-count
- config.yaml with reference_condition (e.g. 'Wild_Type'), comparison conditions (e.g. 'Mutant'), p-val (e.g. '0.01') and logFC (e.g. '0') thresholds
- RPF-length-filtered alignments (reads 25-35 nt) from upstream RiboDoc steps

## Outputs

- DESeq2_by_gene result table (genes called up- or downregulated vs the reference condition)
- DESeq2_by_transcript result table (transcript-level differential calls)
- Names correspondence list between genes and transcripts
- Diagnostic plots: dispersion estimates, log2FoldChange, MA and volcano plots; PCA and Spearman correlation for replicate validation
- Yeast_RiboSeq.Final_report.html under the DESeq2_CDS.25-35 results folder

## How to apply

Load the merged ribosome profiling count matrix (count_matrix_by_transcript_IDs.csv produced by htseq-count) into R together with the conditions defined in config.yaml, including a reference condition (e.g. 'Wild_Type') and comparison conditions (e.g. 'Mutant'). Normalize counts and fit the negative-binomial GLM with DESeq2's estimateSizeFactors, estimateDispersions and nbinomWaldTest. Apply the user-defined significance thresholds from config.yaml — a p-value threshold ('p-val', e.g. '0.01') and a log2FoldChange threshold ('logFC', e.g. '0') — to call differentially expressed genes or transcripts against the reference condition. Generate result tables grouped by gene (DESeq2_by_gene) and by transcript (DESeq2_by_transcript) plus a names correspondence list. Produce diagnostic plots (dispersion estimates, log2FoldChange, MA and volcano plots) alongside replicate-validation statistics (PCA and Spearman correlation), then assemble all tables, figures and the analysis summary into Yeast_RiboSeq.Final_report.html under the DESeq2_CDS.25-35 results folder.

## Related tools

- **RiboDoc** (Docker/Snakemake pipeline that orchestrates the DESeq2 differential expression step and assembles the final HTML report from config.yaml parameters) — https://github.com/equipeGST/RiboDoc
- **DESeq2** (R package that normalizes counts (estimateSizeFactors), fits dispersions (estimateDispersions) and runs the nbinomWaldTest for differential expression) — https://genomebiology.biomedcentral.com/articles/10.1186/s13059-014-0550-8
- **HTSeq** (htseq-count generates the per-transcript count matrix consumed by the DESeq2 step) — https://htseq.readthedocs.io/en/release_0.11.1/count.html#
- **riboWaltz** (Upstream P-site offset prediction whose 25-35 nt RPF-selected outputs feed the quantitative analysis (alternative: TRiP)) — https://github.com/LabTranslationalArchitectomics/riboWaltz

## Examples

```
docker pull equipegst/ribodoc
```

## Evaluation signals

- The DESeq2_CDS.25-35 output folder contains the HTML report (Yeast_RiboSeq.Final_report.html), count matrices, and tables for both by-gene and by-transcript groupings.
- Reported differential calls respect the configured p-val (e.g. 0.01) and logFC thresholds from config.yaml.
- Diagnostic figures (dispersion, log2FoldChange, MA, volcano) and replicate validation (PCA, Spearman correlation) are present in the report.
- All result rows include a names-correspondence entry linking gene-level and transcript-level identifiers.
- Rerunning RiboDoc only re-executes the steps whose recalculation is required, leaving unchanged DESeq2 outputs intact.

## Limitations

- Thresholds (p-val, logFC, reference condition) must be supplied in config.yaml; ambiguous or misspelled condition names in the fastq file naming ([CONDITION]_[NAME].[REPLICATE].fastq.gz) break grouping.
- Low-quality samples must be removed and RiboDoc rerun before differential analysis is meaningful.
- The example GEO dataset (GSE173856) does not align enough material for the riboWaltz pipeline, so TRiP must be selected instead, affecting downstream outputs.

## Evidence

- [full_text] DESeq2 is run on the DDS object via the estimateSizeFactors, estimateDispersions and nbinomWaldTest subfunctions; afterwards, tables of up- and downregulated features and an HTML report are generated.: [evidence span withheld — anchor status: not_found]
