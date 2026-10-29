---
name: mann-whitney-differential-expression-testing
description: Use when you have per-sample small RNA-seq read-count files (tab-delimited,
  RNA molecules as rows, samples as columns) from COMPSRA's Annotation Module and
  need case-vs-control differential expression p-values per RNA.
when_to_use_negative:
- Input is raw FASTQ data — differential expression must be run on read-count files
  already produced by the Annotation Module, not on sequencing reads directly
- You need parametric differential expression models (e.g., negative-binomial GLMs)
  or effect sizes/multiple-testing-adjusted statistics beyond the Mann-Whitney U p-value
  that COMPSRA reports by default
- Only a single sample per group is available — the Mann-Whitney rank sum test requires
  multiple samples per group to rank
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0659
tools:
- name: COMPSRA
  role: Function module (-fd/-fm/-fms) aggregates per-sample Annotation Module read
    counts, applies CpM normalization, and runs the Mann-Whitney (Wilcoxon Rank Sum)
    case-versus-control differential expression test
  repo: https://github.com/cougarlj/COMPSRA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-020-61495-0
    title: 'COMPSRA: a COMprehensive Platform for Small RNA-Seq data Analysis'
  source_content_hash: sha256:4b13f5f23621f2ef133fc12cb7f28f3724eb61da28cfe8f6ed11c5cb3c0acfd0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cougarlj/COMPSRA
  tools:
  - COMPSRA
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# mann-whitney-differential-expression-testing

## Summary

Performs case-versus-control differential expression of small RNA molecules using the Mann-Whitney rank sum test (Wilcoxon Rank Sum Test) on CpM-normalized per-sample read counts aggregated by COMPSRA's Function module.

## When to use

Use when you have per-sample small RNA-seq read-count files (tab-delimited, RNA molecules as rows, samples as columns) from COMPSRA's Annotation Module and need case-vs-control differential expression p-values per RNA.

## When NOT to use

- Input is raw FASTQ data — differential expression must be run on read-count files already produced by the Annotation Module, not on sequencing reads directly
- You need parametric differential expression models (e.g., negative-binomial GLMs) or effect sizes/multiple-testing-adjusted statistics beyond the Mann-Whitney U p-value that COMPSRA reports by default
- Only a single sample per group is available — the Mann-Whitney rank sum test requires multiple samples per group to rank

## Inputs

- Per-sample read-count tab-delimited text files output by COMPSRA's Annotation Module (aggregated into a matrix of RNA molecules as rows and samples as columns)
- Sample list file (-inf, e.g., sample.list) enumerating the FASTQ-derived sample files
- Case/control sample assignments (e.g., -fdcase 1-6 -fdctrl 7-12)
- Selected small RNA classes (-fdclass, e.g., miRNA, piRNA, tRNA, snoRNA, snRNA, circRNA class indices)

## Outputs

- Tab-delimited differential expression output file (e.g., COMPSRA_DEG.tsv) containing annotated RNA molecules across selected classes with case-versus-control Mann-Whitney U test p-values
- Optionally, a merged count matrix (e.g., COMPSRA_MERGE) when only count-file aggregation (-fm -fms) is requested

## How to apply

Merge the per-sample read-count files output by COMPSRA's Annotation Module into a data matrix (RNA molecules as rows, samples as columns) using the Function module (-fm/-fms) or run the differential expression mode directly with -fd. Mark sample columns as case or control by index ranges (e.g., -fdcase 1-6 -fdctrl 7-12) and restrict analysis to the small RNA classes of interest via -fdclass (e.g., 1,2,3,4,5,6 for the six annotated RNA types). Normalize each sample library with Count-per-Million (CpM), the default (-fdnorm cpm), so each library is scaled to one million reads. Then test each RNA molecule for differential expression with the Mann-Whitney rank sum test, the default statistical test (-fdtest mwu). The rationale is that a non-parametric rank-based test is appropriate for small sample counts typical of small RNA-seq case/control studies, and CpM normalizes for library size differences before testing.

## Related tools

- **COMPSRA** (Function module (-fd/-fm/-fms) aggregates per-sample Annotation Module read counts, applies CpM normalization, and runs the Mann-Whitney (Wilcoxon Rank Sum) case-versus-control differential expression test) — https://github.com/cougarlj/COMPSRA

## Examples

```
java -jar COMPSRA.jar -ref hg38 -fun -fd -fdclass 1,2,3,4,5,6 -fdcase 1-6 -fdctrl 7-12 -fdnorm cpm -fdtest mwu -fdann -pro COMPSRA_DEG -inf ./example/sample.list -out ./example_out/
```

## Evaluation signals

- The differential expression output file exists and is tab-delimited
- The output table contains the annotated RNA molecules from the selected small RNA classes (miRNA, piRNA, snRNA, snoRNA, tRNA, circRNA)
- Each RNA row carries a case-versus-control Mann-Whitney U test p-value, consistent with the paper's stated default statistical test
- Sample assignment flags resolved to non-overlapping case and control index ranges (e.g., 1-6 vs 7-12)
- Counts were CpM-normalized (each sample library scaled to one million reads) prior to testing

## Limitations

- Mann-Whitney U test p-values are the default and only stated statistical test; no mention of multiple-testing correction or effect-size estimation in the paper
- Differential expression operates on the RNA types annotated by COMPSRA's small RNA databases (miRBase, piRNABank/piRBase/piRNACluster, gtRNAdb, GENCODE 27, circBase); unannotated molecules are outside this workflow
- Requires grouping samples by index ranges from the merged count matrix, so sample order in the input file list must be correct

## Evidence

- [intro] COMPSRA can perform a differential expression analysis with the p value from the Mann-Whitney U test as default: "COMPSRA can perform a differential expression analysis with the p value from the Mann-Whitney U test as default"
- [methods] The default normalization method is Count-per-Million (CpM), which normalizes each sample library size into one million reads: "The default normalization method is Count-per-Million (CpM), which normalizes each sample library size into one million reads"
- [readme] perform a case versus control differential expression analysis for each RNA molecule using the Mann-Whitney rank sum test (Wilcoxon Rank Sum Test) as the default statistical test: "perform a case versus control differential expression analysis for each RNA molecule using the Mann-Whitney rank sum test (Wilcoxon Rank Sum Test) as the default statistical test"
- [readme] java -jar COMPSRA.jar -ref hg38 -fun -fd -fdclass 1,2,3,4,5,6 -fdcase 1-6 -fdctrl 7-12 -fdnorm cpm -fdtest mwu -fdann -pro COMPSRA_DEG: "java -jar COMPSRA.jar -ref hg38 -fun -fd -fdclass 1,2,3,4,5,6 -fdcase 1-6 -fdctrl 7-12 -fdnorm cpm -fdtest mwu -fdann -pro COMPSRA_DEG"
