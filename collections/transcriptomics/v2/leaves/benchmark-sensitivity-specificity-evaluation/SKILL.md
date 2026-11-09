---
name: benchmark-sensitivity-specificity-evaluation
description: Use when a miRNA/small RNA detection method must be benchmarked against
  a reference truth set (e.g., SRP120169) and you need quantitative sensitivity/specificity,
  including per-abundance-tier performance.
when_to_use_negative:
- Input is already a validated detection list with known TP/FP counts — compute metrics
  directly instead of re-running the pipeline
- No external reference truth set exists for the samples, making TP/FN (and thus sensitivity)
  undefined
- The goal is tool runtime or diversity comparison (e.g., COMPSRA vs exceRpt per-sample
  minutes or RNA-type counts) rather than detection accuracy
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0099
- http://edamontology.org/topic_3308
tools:
- name: COMPSRA
  role: Pipeline under evaluation; runs QC, alignment, annotation, and count-matrix
    aggregation whose miRNA output is scored against the reference set
  repo: https://github.com/cougarlj/COMPSRA
- name: STAR
  role: Default COMPSRA aligner used to map qualified reads to hg38 before miRNA annotation
    (one mismatch, --outFilterMismatchNmax 1)
- name: miRBase
  role: Provides the mature miRNA reference background against which detected miRNAs
    are classified as true/false positives
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41598-020-61495-0
    title: 'COMPSRA: a COMprehensive Platform for Small RNA-Seq data Analysis'
  source_content_hash: sha256:bf022e53c902a3c6872b489151f490b78c8168020f19015060a43cd99a690f10
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cougarlj/COMPSRA
  tools:
  - COMPSRA
  - STAR
  - miRBase
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0099
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# benchmark-sensitivity-specificity-evaluation

## Summary

Validates a small RNA-seq detection pipeline (e.g., COMPSRA) against an external reference dataset by computing sensitivity and specificity of detected miRNAs, both overall and for abundance-restricted subsets such as the top 30% most abundant miRNAs.

## When to use

Use when a miRNA/small RNA detection method must be benchmarked against a reference truth set (e.g., SRP120169) and you need quantitative sensitivity/specificity, including per-abundance-tier performance.

## When NOT to use

- Input is already a validated detection list with known TP/FP counts — compute metrics directly instead of re-running the pipeline
- No external reference truth set exists for the samples, making TP/FN (and thus sensitivity) undefined
- The goal is tool runtime or diversity comparison (e.g., COMPSRA vs exceRpt per-sample minutes or RNA-type counts) rather than detection accuracy

## Inputs

- 12 SRP120169 serum small RNA-seq FASTQ files
- miRBase v21 mature miRNA sequences as detection background
- SRP120169 reference miRNA set (benchmark truth list)
- COMPSRA miRNA-by-sample read count matrix from the Function module

## Outputs

- sensitivity_specificity.tsv with two rows: overall and top-30% sensitivity/specificity pairs
- Per-miRNA true/false positive/negative classification table

## How to apply

First run the pipeline under evaluation on the benchmark samples using the exact QC, alignment, and annotation parameters that define the tool's standard behavior (for COMPSRA on serum: adapter TGGAATTCTCGGGTGCCAAGG, -rb 4, -rh/-rt/-rr 20, read-length grouping -rlh 8,17, STAR alignment to hg38 with --outFilterMismatchNmax 1, and Annotation with -ann -ac 1 at full-overlap Rolp = 1). Aggregate per-sample miRNA read counts into a miRNA-by-sample matrix via the Function module (e.g., -fun -fm -fms 1-12). Then compare the detected miRNA list (against a miRBase mature-miRNA background) to the reference set under the paper's detection thresholds, tallying true/false positives and negatives to compute sensitivity = TP/(TP+FN) and specificity = TN/(TN+FP). Finally restrict to the top 30% most abundant miRNAs and recompute both metrics; in the COMPSRA paper this reproduced 85%/87% overall and 93%/98% for the top 30% tier.

## Related tools

- **COMPSRA** (Pipeline under evaluation; runs QC, alignment, annotation, and count-matrix aggregation whose miRNA output is scored against the reference set) — https://github.com/cougarlj/COMPSRA
- **STAR** (Default COMPSRA aligner used to map qualified reads to hg38 before miRNA annotation (one mismatch, --outFilterMismatchNmax 1))
- **miRBase** (Provides the mature miRNA reference background against which detected miRNAs are classified as true/false positives)

## Examples

```
java -jar COMPSRA.jar -ref hg38 -qc -ra TGGAATTCTCGGGTGCCAAGG -rb 4 -rh 20 -rt 20 -rr 20 -rlh 8,17 -aln -mt star -ann -ac 1 -fun -fm -fms 1-12 -fdann -pro SRP120169 -inf sample.list -out ./example_out/
```

## Evaluation signals

- sensitivity_specificity.tsv contains exactly two metric pairs: 85%/87% (overall) and 93%/98% (top 30% of miRNAs)
- TP+FN for the overall comparison equals the size of the SRP120169 reference miRNA set
- Top-30% sensitivity and specificity both exceed their overall counterparts (93 > 85; 98 > 87)
- QC module retained reads fall in the expected length interval after removing the 3' adapter TGGAATTCTCGGGTGCCAAGG and 4 random ligation bases (-rb 4), per the -rlh 8,17 grouping
- The miRNA count matrix has 12 sample columns matching the SRP120169 serum samples

## Limitations

- Sensitivity/specificity depend on the detection thresholds chosen for the reference set; changing them changes the truth labels
- The benchmark uses serum samples of 12 healthy human controls, so metrics may not generalize to diseased or low-input samples
- COMPSRA's prebuilt annotation databases are version-locked (e.g., miRBase20 pre-built at publication; the task uses miRBase v21 as background), so background choice affects classification
- Only reads mapped to the human genome are annotated; microbial and unmapped reads are outside this benchmark

## Evidence

- [other] COMPSRA achieved a sensitivity of 85% and specificity of 87% against the SRP120169 benchmark overall, and a sensitivity of 93% and specificity of 98% when restricted to the top 30% of identified miRNAs.: [evidence span withheld — anchor status: not_found]
- [readme] COMPSRA QC module parameters for serum FASTQ processing: "java -jar COMPSRA.jar -ref hg38 -qc -ra TGGAATTCTCGGGTGCCAAGG -rb 4 -rh 20 -rt 20 -rr 20 -rlh 8,17 -in ./example/sample01.fastq -out ./example_out/"
- [readme] Function module count matrix aggregation: "If you only want to merge the count files, you can use -fm -fms."
- [methods] Annotation module database coverage and full-overlap annotation criterion: "COMPSRA currently uses several different (and expandable) small RNA reference databases for annotating human genome mapped reads"
- [intro] COMPSRA takes FASTQ files as input and outputs per-sample RNA counts profile: "It takes FASTQ file as inputs and outputs the counts profile of each type of RNA molecule type per FASTQ file"
