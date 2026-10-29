---
name: per-sample-read-count-matrix-merging
description: Use when COMPSRA annotation has produced per-sample read-count files
  for multiple FASTQ inputs and you need a single merged RNA-molecule × sample count
  matrix (e.g., 12 serum samples × 6 RNA classes).
when_to_use_negative:
- You need case-vs-control differential expression testing rather than a plain merged
  matrix — use the -fd mode with -fdcase/-fdctrl instead of merge-only -fm.
- Input is raw FASTQ or unaligned BAM — counts must first be produced by the QC, Alignment
  and Annotation modules.
- Only a single sample is available — no cross-sample matrix merging is required;
  the per-sample count file is already the output.
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: COMPSRA
  role: Function module (-fun -fm -fms -fdclass) aggregates per-sample annotation
    count files into merged matrices and applies CpM normalization
  repo: https://github.com/cougarlj/COMPSRA
- name: Java
  role: Runtime environment (JRE 8+) used to execute COMPSRA.jar for the merge command
- name: STAR
  role: Upstream aligner producing the BAMs whose counts feed the merge; not invoked
    during merging itself
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41598-020-61495-0
    title: 'COMPSRA: a COMprehensive Platform for Small RNA-Seq data Analysis'
  source_content_hash: sha256:abee027395e2fff14581d0a99377fc382aca5808fe5e7fe9cfa798308654437d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cougarlj/COMPSRA
  tools:
  - COMPSRA
  - Java
  - STAR
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# per-sample-read-count-matrix-merging

## Summary

Aggregates COMPSRA per-sample RNA read-count files (tab-delimited, one per RNA type) into a matrix of RNA molecules as rows and samples as columns, with optional Count-per-Million (CpM) normalization, via the COMPSRA Function module.

## When to use

Use when COMPSRA annotation has produced per-sample read-count files for multiple FASTQ inputs and you need a single merged RNA-molecule × sample count matrix (e.g., 12 serum samples × 6 RNA classes).

## When NOT to use

- You need case-vs-control differential expression testing rather than a plain merged matrix — use the -fd mode with -fdcase/-fdctrl instead of merge-only -fm.
- Input is raw FASTQ or unaligned BAM — counts must first be produced by the QC, Alignment and Annotation modules.
- Only a single sample is available — no cross-sample matrix merging is required; the per-sample count file is already the output.

## Inputs

- Per-sample tab-delimited RNA read-count files produced by the COMPSRA Annotation module (one per RNA type per sample)
- A sample list file (one FASTQ or sample name per line) passed via -inf
- Sample index range (e.g., -fms 1-12) and RNA class list (-fdclass 1,2,3,4,5,6)

## Outputs

- Merged count matrix of RNA molecules as rows and samples as columns (tab-delimited text), one per RNA class
- Optionally CpM-normalized count matrix
- Distinct-molecule detection counts per RNA type (e.g., compsra_detection_counts.csv) at a read-count threshold

## How to apply

After the QC, Alignment and Annotation modules have written per-sample tab-delimited count files for each RNA class (-ac 1,2,3,4,5,6 covering miRNA, piRNA, snRNA, snoRNA, tRNA and circRNA), run the Function module in merge-only mode with -fun -fm and specify the samples to merge with -fms (e.g., -fms 1-12) and the RNA classes with -fdclass 1,2,3,4,5,6; supply the sample list with -inf and a project name with -pro. The module concatenates counts so that rows are RNA molecules and columns are samples, matching the read counts of an RNA molecule across different samples. Apply the default Count-per-Million normalization where appropriate, which normalizes each sample library size into one million reads. For the 12 healthy human serum samples, tabulate distinct molecules per RNA type at the article's read-count threshold of 5 and compare against the reported 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs.

## Related tools

- **COMPSRA** (Function module (-fun -fm -fms -fdclass) aggregates per-sample annotation count files into merged matrices and applies CpM normalization) — https://github.com/cougarlj/COMPSRA
- **Java** (Runtime environment (JRE 8+) used to execute COMPSRA.jar for the merge command)
- **STAR** (Upstream aligner producing the BAMs whose counts feed the merge; not invoked during merging itself)

## Examples

```
java -jar COMPSRA.jar -ref hg38 -fun -fm -fms 1-12 -fdclass 1,2,3,4,5,6 -fdann -pro COMPSRA_MERGE -inf ./example/sample.list -out ./example_out/
```

## Evaluation signals

- Merged matrix has RNA molecules as rows and exactly the requested number of sample columns (e.g., 12 for -fms 1-12).
- Column sums before normalization match the corresponding per-sample count files; after CpM each sample column sums to one million reads.
- Distinct molecules per RNA type at read-count threshold 5 reproduce the article values: 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs, 7285 circRNAs.
- Output files are tab-delimited text named with the -pro project prefix and cover all classes in -fdclass 1,2,3,4,5,6.
- No sample columns are missing or reordered relative to the order in the -inf sample list file.

## Limitations

- The merge-only mode (-fm) does not perform differential expression; case/control testing requires -fd with -fdcase/-fdctrl sample ranges.
- Merging is only meaningful for RNA classes that were annotated in every sample (-ac 1,2,3,4,5,6 must have run upstream); absent count files for a class will produce no matrix.
- The software version used for the article's 12-serum-sample reproduction is not stated in the results section, so exact count reproduction may vary slightly across COMPSRA releases (e.g., v1.0.3).

## Evidence

- [readme] With more than one sample FASTQ file inputs, the out-put are further aggregated into a data matrix of RNA molecules as rows and samples as columns showing the read counts of an RNA molecule across different samples.: "With more than one sample FASTQ file inputs, the out-put are further aggregated into a data matrix of RNA molecules as rows and samples as columns"
- [methods] The default normalization method is Count-per-Million (CpM), which normalizes each sample library size into one million reads: "The default normalization method is Count-per-Million (CpM), which normalizes each sample library size into one million reads"
- [readme] If you only want to merge the count files, you can use -fm -fms.: "If you only want to merge the count files, you can use -fm -fms."
- [results] COMPSRA detected 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs: "COMPSRA detected 375 miRNAs, 280 piRNAs, 167 snRNAs, 88 snoRNAs, 401 tRNAs and 7285 circRNAs"
