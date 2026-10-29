---
name: case-control-small-rna-expression-analysis
description: Use when you have per-sample small RNA-seq read-count files from COMPSRA's
  Annotation module and samples are labeled as case and control groups, and you need
  per-RNA differential expression p-values.
when_to_use_negative:
- Input is raw FASTQ data that has not yet passed through COMPSRA's QC, Alignment,
  and Annotation modules — the Function module expects per-sample read-count files
- Only merging of count files into a matrix is needed and no case/control contrast
  exists — use the merge-only options (-fm -fms) instead of -fd
- A parametric differential expression method with replicate-level modeling (e.g.,
  DESeq2-style dispersion estimation) is specifically required — COMPSRA's default
  is the non-parametric Mann-Whitney U test p-value
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0659
tools:
- name: COMPSRA
  role: Runs the Function module (-fun -fd) that aggregates read counts, applies CpM
    normalization, and computes Mann-Whitney differential expression p-values
  repo: https://github.com/cougarlj/COMPSRA
- name: COMPSRA v1.0
  role: Java stand-alone platform whose Function module implements the case-versus-control
    differential expression analysis
  repo: https://github.com/cougarlj/COMPSRA
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-020-61495-0
    title: 'COMPSRA: a COMprehensive Platform for Small RNA-Seq data Analysis'
  source_content_hash: sha256:179fb7d309b289a8f54aa7f23465afd3f902d21a3a3b656855412dfc94c52838
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cougarlj/COMPSRA
  tools:
  - COMPSRA
  - COMPSRA v1.0
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9230769230769231
n_claims: 13
---

# case-control-small-rna-expression-analysis

## Summary

Performs case-versus-control differential expression analysis of small RNA-seq read counts using COMPSRA's Function module, which normalizes per-sample libraries to Count-per-Million (CpM) and tests each RNA molecule with the Mann-Whitney rank sum test (Wilcoxon Rank Sum Test) as the default statistical test.

## When to use

Use when you have per-sample small RNA-seq read-count files from COMPSRA's Annotation module and samples are labeled as case and control groups, and you need per-RNA differential expression p-values.

## When NOT to use

- Input is raw FASTQ data that has not yet passed through COMPSRA's QC, Alignment, and Annotation modules — the Function module expects per-sample read-count files
- Only merging of count files into a matrix is needed and no case/control contrast exists — use the merge-only options (-fm -fms) instead of -fd
- A parametric differential expression method with replicate-level modeling (e.g., DESeq2-style dispersion estimation) is specifically required — COMPSRA's default is the non-parametric Mann-Whitney U test p-value

## Inputs

- Per-sample small RNA-seq read-count files (tab-delimited text per RNA type) output by COMPSRA's Annotation module
- A sample list file (-inf, e.g., sample.list) naming the FASTQ-derived samples in order
- Case and control sample ID assignments (e.g., -fdcase 1-6 -fdctrl 7-12)
- RNA class IDs to restrict analysis (e.g., -fdclass 1,2,3,4,5,6)

## Outputs

- Tab-delimited differential expression results file (e.g., COMPSRA_DEG.tsv) containing annotated RNA molecules and case-versus-control Mann-Whitney U test p-values
- Merged count matrix with RNA molecules as rows and samples as columns (when merging only, e.g., COMPSRA_MERGE)

## How to apply

Load the per-sample read-count files output by COMPSRA's Annotation module and merge them into a data matrix with RNA molecules as rows and samples as columns (Function module, -fm/-fms). Mark each sample column as case or control in the command line (e.g., -fdcase 1-6 -fdctrl 7-12) and restrict analysis to the requested small RNA classes with -fdclass (e.g., 1,2,3,4,5,6). Normalize each sample library to Count-per-Million (CpM), the default normalization (-fdnorm cpm), which scales each library to one million reads. Test each RNA molecule for differential expression using the Mann-Whitney rank sum test as the default statistical test (-fdtest mwu), then write results to a tab-delimited output file named via -pro (e.g., COMPSRA_DEG.tsv). The rationale is that CpM normalization accounts for differing library sizes and the non-parametric Mann-Whitney U test makes no distributional assumption about read counts across groups.

## Related tools

- **COMPSRA** (Runs the Function module (-fun -fd) that aggregates read counts, applies CpM normalization, and computes Mann-Whitney differential expression p-values) — https://github.com/cougarlj/COMPSRA
- **COMPSRA v1.0** (Java stand-alone platform whose Function module implements the case-versus-control differential expression analysis) — https://github.com/cougarlj/COMPSRA

## Examples

```
java -jar COMPSRA.jar -ref hg38 -fun -fd -fdclass 1,2,3,4,5,6 -fdcase 1-6 -fdctrl 7-12 -fdnorm cpm -fdtest mwu -fdann -pro COMPSRA_DEG -inf ./example/sample.list -out ./example_out/
```

## Evaluation signals

- The output file (e.g., COMPSRA_DEG.tsv) exists and is tab-delimited
- The output table contains the annotated RNA molecules across the selected RNA classes (-fdclass) with CpM-normalized values
- Each RNA molecule row includes a case-versus-control Mann-Whitney U test p-value, consistent with the paper's stated default statistical test
- The merged matrix has RNA molecules as rows and the correct number of sample columns matching the sample list (-inf)
- Case and control sample assignments in the output correspond to the -fdcase and -fdctrl ranges given on the command line

## Limitations

- Only one statistical test (Mann-Whitney U / Wilcoxon Rank Sum) and one normalization (CpM) are the defaults; the article does not document alternative tests or multiple-testing correction in the Function module
- Requires the Annotation module's per-sample read counts first, so it cannot be run standalone from FASTQ without the upstream pipeline
- Sample labeling is done purely by sample IDs on the command line, so misordered sample lists will silently swap case/control assignments

## Evidence

- [intro] COMPSRA can perform a differential expression analysis with the p value from the Mann-Whitney U test as default: "COMPSRA can perform a differential expression analysis with the p value from the Mann-Whitney U test as default"
- [methods] The default normalization method is Count-per-Million (CpM): "The default normalization method is Count-per-Million (CpM), which normalizes each sample library size into one million reads"
- [methods] Mann-Whitney rank sum test as the default statistical test: "perform a case versus control differential expression analysis for each RNA molecule using the Mann-Whitney rank sum test (Wilcoxon Rank Sum Test) as the default statistical test"
- [readme] Function module README command with -fdcase -fdctrl -fdnorm cpm -fdtest mwu: "java -jar COMPSRA.jar -ref hg38 -fun -fd -fdclass 1,2,3,4,5,6 -fdcase 1-6 -fdctrl 7-12 -fdnorm cpm -fdtest mwu -fdann -pro COMPSRA_DEG -inf ./example/sample.list -out ./example_out/"
- [readme] README description of marking columns as case or control: "The user can mark each sample FASTQ file column as either a case or a control in the command line"
