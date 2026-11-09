---
name: sample-outlier-detection-percentile-thresholding
description: Use when a raw barcode counts table (e.g. from BARtab or a GEO deposit
  such as GSE246611) needs sample-level and barcode-level QC before downstream barcoding
  analysis.
when_to_use_negative:
- Input is already a QC-filtered counts table with sample-level and per-barcode filters
  applied — reapplying percentile cutoffs would remove valid samples
- Single-cell or spatial transcriptomics barcode data, where bartools UMI-based filters
  (--umi_count_filter, --umi_fraction_filter) are the appropriate QC route instead
  of bulk sample percentiles
- FASTQ-level preprocessing needs (read trimming, barcode extraction, clustering)
  — use BARtab pipeline modules (FASTQC, FILTER_READS, CUTADAPT_READS, starcode) rather
  than counts-table QC
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: bartools
  role: Open-source R package providing the sample-level and per-barcode QC filter
    workflows for population-level barcoding counts tables
  repo: https://github.com/DaneVass/bartools/
- name: R
  role: Runtime environment for bartools and for loading the counts table and writing
    the filtered TSV
- name: tidyverse
  role: R package collection used to load and manipulate the raw barcode counts table
    during filtering
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2024.100763
    title: Analysis of synthetic cellular barcodes in the genome and transcriptome
      with BARtab and bartools
  source_content_hash: sha256:d06322c3b7bcf899fed94d394442e611a0ced226c9b470877acddd6080e86f2e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/DaneVass/bartools
  tools:
  - BARtab
  - R
  - bartools
  - tidyverse
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# Sample-outlier-detection-percentile-thresholding

## Summary

A bartools-based QC skill for population-level cellular barcoding datasets that removes low-quality samples via a sample-level 5th-percentile total-read-count cutoff and low-abundance barcodes via a per-barcode threshold (<10 reads in <4 samples), yielding a high-confidence barcodes x samples counts matrix.

## When to use

Use when a raw barcode counts table (e.g. from BARtab or a GEO deposit such as GSE246611) needs sample-level and barcode-level QC before downstream barcoding analysis.

## When NOT to use

- Input is already a QC-filtered counts table with sample-level and per-barcode filters applied — reapplying percentile cutoffs would remove valid samples
- Single-cell or spatial transcriptomics barcode data, where bartools UMI-based filters (--umi_count_filter, --umi_fraction_filter) are the appropriate QC route instead of bulk sample percentiles
- FASTQ-level preprocessing needs (read trimming, barcode extraction, clustering) — use BARtab pipeline modules (FASTQC, FILTER_READS, CUTADAPT_READS, starcode) rather than counts-table QC

## Inputs

- Raw barcode counts table (barcodes x samples) from a population-level barcoding experiment, e.g. GEO accession GSE246611 (SPLINTR dose-escalation dataset, 1,790 barcodes x 47 samples before filtering)
- Sample metadata / sample IDs matching counts table columns

## Outputs

- Filtered counts matrix of high-confidence barcodes x retained samples (reference result: 1,680 barcodes x 41 samples)
- TSV file of filtered counts, e.g. dose_escalation_filtered_counts.tsv
- List of removed low-quality outlier samples (6 at the 5th-percentile cutoff) and removed low-abundance barcodes (110)

## How to apply

Load the raw barcode counts table (e.g. the SPLINTR dose-escalation dataset, GEO accession GSE246611) into R using the tidyverse, then compute per-sample total read counts and flag samples falling below the 5th percentile of that distribution as low-quality outliers — in the reference dataset this removed 6 samples. Next apply the low-stringency per-barcode filter, retaining only barcodes with at least 10 reads in at least 4 samples, which eliminates low-abundance barcodes (110 removed in the reference analysis). The rationale is that samples with anomalously low total barcode reads reflect poor library quality, while barcodes detectable in few samples with few reads are not high-confidence lineage labels. Write the resulting filtered counts matrix to a TSV (dose_escalation_filtered_counts.tsv) and validate its dimensions before reuse. Thresholds can be tuned (stringent vs low-stringency per-barcode filters) using bartools QC functions.

## Related tools

- **bartools** (Open-source R package providing the sample-level and per-barcode QC filter workflows for population-level barcoding counts tables) — https://github.com/DaneVass/bartools/
- **R** (Runtime environment for bartools and for loading the counts table and writing the filtered TSV)
- **tidyverse** (R package collection used to load and manipulate the raw barcode counts table during filtering)

## Examples

```
counts <- read.delim("GSE246611_counts.tsv", row.names=1); keep <- rowSums(counts >= 10) >= 4; filtered <- counts[keep, colSums(counts) >= quantile(colSums(counts), 0.05)]; write.table(filtered, "dose_escalation_filtered_counts.tsv", sep="\t")
```

## Evaluation signals

- Output TSV exists, parses as a valid TSV, and its dimensions equal the reported 1,680 barcodes x 41 samples
- Exactly 6 samples flagged as outliers by the 5th-percentile total-read-count cutoff (47 -> 41 samples in the reference dataset)
- Exactly 110 low-abundance barcodes removed by the <10 reads in <4 samples rule (1,790 -> 1,680 barcodes)
- Every retained barcode has at least 10 reads in at least 4 retained samples (invariant check on the filtered matrix)
- Filtered counts values are a subset of the raw counts table (no barcode counts altered, only rows/columns dropped)

## Limitations

- Thresholds are dataset-scale dependent: the 5th-percentile sample cutoff and the 10-reads/4-samples barcode rule were chosen for the dose-escalation dataset and may need retuning for datasets of different size or depth
-  bartools and BARtab currently do not officially support evolving barcode methods, although several functions already available will likely have utility
- Per-barcode stringency is a choice: high-stringency filters remove more barcodes, so the low-stringency (<10 reads in <4 samples) setting used here trades sensitivity for retaining more barcodes

## Evidence

- [methods] Sample-level 5th-percentile filter removes outliers: [evidence span withheld — anchor status: not_found]
- [results] Low-stringency per-barcode filter yields the reference filtered dataset: [evidence span withheld — anchor status: not_found]
- [intro] bartools is the R package underlying this QC workflow: "we developed bartools, a flexible open-source R package that incorporates workflows for population-level cellular barcoding data analysis"
- [readme] Manuscript code repository contains the dose-escalation bartools analysis: [evidence span withheld — anchor status: external_doc]
