---
name: icgc-paca-au-sample-metadata-handling
description: Use when a task requires paired-sample analysis of the ICGC PACA-AU cohort
  (primary vs metastatic vs derived cell line comparisons) across both RNAseq and
  microarray platforms using pdacR-parsed objects.
when_to_use_negative:
- When the analysis does not require pairing (e.g., cohort-level subtype classification
  or survival analysis on TCGA-PAAD), plain pdacR dataset loading and filtering suffices
- When the input data are un-parsed raw ICGC downloads rather than pdacR-parsed .rds
  objects, since the sampInfo structure and ICGC_PARSE_Nones flags will not exist
- When you need to merge samples across RNAseq and microarray into a single expression
  matrix, since pdacR explicitly warns it cannot robustly merge datasets across platforms
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: pdacR
  role: R package providing the parsed ICGC PACA-AU RNAseq/microarray datasets, matching
    sampInfo metadata, and gene_lists object loaded from the data directory
  repo: github.com/rmoffitt/pdacR
- name: R
  role: Runtime language for loading .rds objects and joining sampInfo metadata to
    define paired sample classes
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s42003-023-04461-6
    title: Open-source curation of a pancreatic ductal adenocarcinoma gene expression
      analysis platform (pdacR) supports a two-subtype model
  source_content_hash: sha256:11a98880f0321abc1d8d06f685478cf0158294174831fd7652dc0e592abb222e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - pdacR
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# icgc-paca-au-sample-metadata-handling

## Summary

Method for loading and joining the paired ICGC PACA-AU RNAseq and microarray expression matrices with their sampInfo metadata tables to identify matched primary, metastatic, and cell-line samples while excluding samples flagged by ICGC_PARSE_Nones, enabling paired analyses such as signature-loss testing.

## When to use

Use when a task requires paired-sample analysis of the ICGC PACA-AU cohort (primary vs metastatic vs derived cell line comparisons) across both RNAseq and microarray platforms using pdacR-parsed objects.

## When NOT to use

- When the analysis does not require pairing (e.g., cohort-level subtype classification or survival analysis on TCGA-PAAD), plain pdacR dataset loading and filtering suffices
- When the input data are un-parsed raw ICGC downloads rather than pdacR-parsed .rds objects, since the sampInfo structure and ICGC_PARSE_Nones flags will not exist
- When you need to merge samples across RNAseq and microarray into a single expression matrix, since pdacR explicitly warns it cannot robustly merge datasets across platforms

## Inputs

- Parsed ICGC PACA-AU RNAseq expression matrix (.rds, genes x samples) from the pdacR data directory
- Parsed ICGC PACA-AU microarray expression matrix (.rds, genes x samples) from the pdacR data directory
- sampInfo sample metadata tables matching each PACA-AU expression matrix
- ICGC_PARSE_Nones exclusion flags from the pdacR parsing output

## Outputs

- Paired-sample class table linking primary, metastatic, and cell-line samples across RNAseq and microarray platforms
- Filtered sample lists per platform with non-usable (ICGC_PARSE_Nones-flagged) samples removed

## How to apply

Load the parsed ICGC PACA-AU RNAseq and microarray expression sets from the pdacR data directory as .rds objects, each an expression matrix of genes x samples with a matching sampInfo metadata table. Use the sampInfo metadata to identify paired samples within and between the two PACA-AU platforms, defining pair classes of primary-primary, primary-metastatic, and primary-cell line. Exclude samples flagged by ICGC_PARSE_Nones (samples that ICGC parsing flagged as non-usable), mirroring pdacR's curation philosophy where sample-level metadata must be consulted to remove inappropriate samples before analysis. Keep platform labels alongside class labels so downstream per-platform comparisons (e.g., z-scored gene set scores) are not confounded by cross-platform batch effects.

## Related tools

- **pdacR** (R package providing the parsed ICGC PACA-AU RNAseq/microarray datasets, matching sampInfo metadata, and gene_lists object loaded from the data directory) — github.com/rmoffitt/pdacR
- **R** (Runtime language for loading .rds objects and joining sampInfo metadata to define paired sample classes)

## Examples

```
expr_seq <- readRDS(system.file("data", "PACA-AU_RNAseq.rds", package="pdacR")); paired <- subset(sampInfo, sampleClass %in% c("primary","metastatic","cell_line") & !(sampleID %in% ICGC_PARSE_Nones))
```

## Evaluation signals

- Every retained sample in the paired table has a non-null sample class label and platform annotation
- No sample flagged by ICGC_PARSE_Nones appears in any downstream pair set
- Pair classes are restricted to primary-primary, primary-metastatic, and primary-cell line relationships within and across the two PACA-AU platforms
- Each paired sample resolves to exactly one entry in the corresponding sampInfo table (join produces no duplicate or orphan samples)
- The resulting paired sample table feeds downstream signature scoring (e.g., icgc_gene_set_scores.csv) without cross-platform mislabeling

## Limitations

- pdacR notes an 'inability to robustly merge datasets' across platforms, so paired samples must be analyzed per-platform rather than pooled into one matrix
- Bulk transcriptomic data limit the ability to fully separate cellular populations, so sample class assignments depend on ICGC's provided metadata rather than single-cell deconvolution
- Aggregation across platforms 'introduced more problems than it solved' and was removed from pdacR; multi-platform complexity applies here as well

## Evidence

- [other] Load the parsed ICGC PACA-AU RNAseq and microarray expression sets from the pdacR data directory using the parse_ICGC-derived .rds objects (matrix of genes x samples with matching sampInfo).: [evidence span withheld — anchor status: not_found]
- [other] Identify paired samples within and between the two PACA-AU platforms (primary-primary, primary-metastatic, primary-cell line) using the sampInfo metadata, excluding samples flagged by ICGC_PARSE_Nones.: [evidence span withheld — anchor status: not_found]
- [discussion] another layer of complexity to aggregation and batch correction: "our datasets come from a variety of platforms, including bulk RNA sequencing and microarrays, adding another layer of complexity to aggregation and batch correction"
- [discussion] inability to robustly merge datasets: "Another limitation inherent to this sort of data collection is the inability to robustly merge datasets"
- [readme] data directory holds parsed datasets with gene signatures object: "Data/ holds all parsed datasets, as well as the gene signatures object and PurIST classifier object."
