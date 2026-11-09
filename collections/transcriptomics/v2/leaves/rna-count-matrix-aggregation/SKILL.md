---
name: rna-count-matrix-aggregation
description: Use when multiple small RNA-seq samples have been annotated by COMPSRA
  and per-sample tab-delimited RNA count files must be merged into one RNA-by-sample
  count matrix before downstream normalization or differential expression.
when_to_use_negative:
- Case/control differential expression statistics are needed — use the -fd mode with
  -fdcase/-fdctrl, -fdnorm cpm and -fdtest mwu instead of merge-only -fm/-fms
- Input is raw FASTQ or unmapped reads — samples must first pass QC, Alignment (STAR)
  and Annotation modules before counts exist to merge
- Only a single sample is available — there is nothing to aggregate across sample
  columns
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0099
tools:
- name: COMPSRA
  role: Provides the Function module (-fun -fm -fms) that aggregates per-sample Annotation-module
    read-count files into a merged RNA-by-sample tab-delimited matrix
  repo: https://github.com/cougarlj/COMPSRA
- name: STAR
  role: Upstream aligner whose Annotation-module-derived per-sample count files are
    the inputs being merged
provenance:
  source_task_ids:
  - task_007
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-61495-0
    title: 'COMPSRA: a COMprehensive Platform for Small RNA-Seq data Analysis'
  source_content_hash: sha256:718d5fe26a4e037196ddaffe9841b4293a0b84d2b10acae3fff89c30d2c5159e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cougarlj/COMPSRA
  tools:
  - COMPSRA
  - STAR
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0099
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# rna-count-matrix-aggregation

## Summary

Aggregate per-sample small RNA read-count files output by the COMPSRA Annotation module into a single tab-delimited matrix with RNA molecules as rows and samples as columns, using the COMPSRA Function module's -fm/-fms merge mode.

## When to use

Use when multiple small RNA-seq samples have been annotated by COMPSRA and per-sample tab-delimited RNA count files must be merged into one RNA-by-sample count matrix before downstream normalization or differential expression.

## When NOT to use

- Case/control differential expression statistics are needed — use the -fd mode with -fdcase/-fdctrl, -fdnorm cpm and -fdtest mwu instead of merge-only -fm/-fms
- Input is raw FASTQ or unmapped reads — samples must first pass QC, Alignment (STAR) and Annotation modules before counts exist to merge
- Only a single sample is available — there is nothing to aggregate across sample columns

## Inputs

- Per-sample tab-delimited RNA read-count files output by the COMPSRA Annotation module
- Sample list file (.list) with one input file per line (passed via -inf)
- Annotation class selection (e.g., -fdclass 1,2,3,4,5,6 for miRNA, piRNA, snRNA, snoRNA, tRNA, circRNA)

## Outputs

- Tab-delimited RNA-by-sample read-count matrix (e.g., COMPSRA_MERGE output), RNA molecules as rows and sample columns showing read counts

## How to apply

Run the COMPSRA Function module (-fun) in merge-only mode with -fm and -fms to specify which sample columns (e.g., -fms 1-12) to aggregate, after the Annotation module has produced per-sample count files split by RNA type (-fdclass 1,2,3,4,5,6 for miRNA, piRNA, snRNA, snoRNA, tRNA, circRNA). Use -fdann to include annotations and -pro to name the merged output (e.g., COMPSRA_MERGE); supply multiple samples via a sample list file with -inf rather than repeated -in arguments. The output is a data matrix of RNA molecules as rows and samples as columns showing read counts per RNA across samples. This merge step is a standalone alternative to running the full differential expression workflow (-fd), and can also be embedded at the end of the full QC-Alignment-Annotation pipeline command. Validate that the merged table is tab-delimited, contains the expected sample columns, and that per-RNA counts match the source Annotation module count files.

## Related tools

- **COMPSRA** (Provides the Function module (-fun -fm -fms) that aggregates per-sample Annotation-module read-count files into a merged RNA-by-sample tab-delimited matrix) — https://github.com/cougarlj/COMPSRA
- **STAR** (Upstream aligner whose Annotation-module-derived per-sample count files are the inputs being merged)

## Examples

```
java -jar COMPSRA.jar -ref hg38 -fun -fm -fms 1-12 -fdclass 1,2,3,4,5,6 -fdann -pro COMPSRA_MERGE -inf ./example/sample.list -out ./example_out/
```

## Evaluation signals

- Output file exists and is tab-delimited as specified for Function-module count outputs
- Matrix dimensions equal the number of annotated RNA molecules (rows) by the number of samples selected in -fms (columns)
- Per-RNA cell values match the read counts in the corresponding per-sample Annotation-module count files
- Only the requested RNA classes (-fdclass) are present as rows
- Merge-only runs produce a count matrix without differential expression columns (no case/control p-values), distinguishing -fm/-fms from -fd output

## Limitations

- Merge-only mode does not normalize counts (CpM normalization is part of the -fd differential expression path, via -fdnorm cpm)
- Counts are split by RNA type; the user must request the desired classes explicitly (-fdclass) or rows for some molecule types will be absent
- COMPSRA's annotation databases are version-locked to reference genome builds (LiftOver used to reconcile, e.g., hg19 vs hg38), so merged matrices depend on the -ref genome version used

## Evidence

- [readme] With more than one sample FASTQ file inputs, the out-put are further aggregated into a data matrix of RNA molecules as rows and samples as columns showing the read counts of an RNA molecule across different samples.: "With more than one sample FASTQ file inputs, the out-put are further aggregated into a data matrix of RNA molecules as rows and samples as columns"
- [readme] The read count of each RNA molecule that is identified in the Annotation Module is outputted as a tab-delimited text file according to RNA type.: "The read count of each RNA molecule that is identified in the Annotation Module is outputted as a tab-delimited text file according to RNA type."
- [readme] (Note: If you only want to merge the count files, you can use -fm -fms.): "(Note: If you only want to merge the count files, you can use -fm -fms.)"
- [results] COMPSRA detected 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs: "COMPSRA detected 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs, as well as 608 archaea"
