---
name: benchmark-count-reproduction
description: Use when validating or benchmarking a published bioinformatics pipeline
  by re-executing its workflow steps on the same inputs and checking that every reported
  count matches the original paper's numbers.
when_to_use_negative:
- The sRNA-Seq data are from a species without a reference genome — iwa-miRNA cannot
  handle such data and ignores reads that do not map to the reference
- The goal is functional downstream analysis of validated miRNAs (e.g. GO annotation,
  miRNA set enrichment, cross-species comparison) rather than count-level reproduction
  of the annotation workflow
- Only a subset of the original databases or datasets is available — counts will not
  be comparable to the published benchmarks
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3170
tools:
- name: iwa-miRNA
  role: Galaxy-based framework whose modules (miRNARetrival, genomePrepare, miRNAPredict,
    miRNATranslate, miRNASelection) are re-executed to reproduce the benchmark counts
  repo: https://github.com/cma2015/iwa-miRNA
- name: Galaxy
  role: Web platform in which the iwa-miRNA modules are organized with user-friendly
    interfaces for reproducible re-execution
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2021.02.010
    title: Interactive Web-Based Annotation of Plant MicroRNAs with iwa-miRNA
  source_content_hash: sha256:b04a5a7b8cc9836fb654c8f14d0ce10c5ce31a3b545f550f60e3c8b1222c6eb7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cma2015/iwa-miRNA
  tools:
  - Galaxy
  - iwa-miRNA
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# benchmark-count-reproduction

## Summary

A method for independently reproducing published benchmark counts from an iwa-miRNA analysis — here, the A. thaliana Columbia-ecotype miRNA annotation counts (aggregated 365, predicted 435, total 667, Ara-Set1 330, Ara-Set2 308, overlap 208) — by re-running the tool's module pipeline and comparing each count against the paper's results section.

## When to use

Use when validating or benchmarking a published bioinformatics pipeline by re-executing its workflow steps on the same inputs and checking that every reported count matches the original paper's numbers.

## When NOT to use

- The sRNA-Seq data are from a species without a reference genome — iwa-miRNA cannot handle such data and ignores reads that do not map to the reference
- The goal is functional downstream analysis of validated miRNAs (e.g. GO annotation, miRNA set enrichment, cross-species comparison) rather than count-level reproduction of the annotation workflow
- Only a subset of the original databases or datasets is available — counts will not be comparable to the published benchmarks

## Inputs

- Species name 'Arabidopsis thaliana' for database retrieval
- Four public plant miRNA databases: miRBase v22.1, PmiREN v1.0, sRNAanno v1.0, PsRNA v1.0
- TAIR10 genome sequence and annotation files
- 1063 SRA accession numbers for A. thaliana Columbia-ecotype sRNA-Seq datasets (File S1)
- Test datasets from github.com/cma2015/iwa-miRNA/tree/master/Test_data

## Outputs

- arabidopsis_benchmark_counts.csv with counts: aggregated 365, predicted 435, total 667, Set1 330, Set2 308, overlap 208
- Uniform-format precursor/mature candidate list from miRNATranslate (667 precursors / 1190 mature candidates, cf. Table S2)
- Ara-Set1 (330 high-throughput-criteria precursors) and Ara-Set2 (308 one-class SVM precursors) candidate sets

## How to apply

Re-run the iwa-miRNA pipeline module by module in the same order as the source study. First run miRNARetrival with species name 'Arabidopsis thaliana' against miRBase v22.1, PmiREN v1.0, sRNAanno v1.0, and PsRNA v1.0, expecting 365 precursors / 625 mature forms. Next run genomePrepare with the TAIR10 genome sequence and annotation files, then miRNAPredict on the 1063 SRA sRNA-Seq datasets for the Columbia ecotype (File S1), expecting 435 predicted precursors. Run miRNATranslate to convert retrieval and prediction outputs into a uniform genomic-coordinate format and aggregate them into 667 precursor candidates / 1190 mature candidates (Table S2), and finally miRNASelection with default parameters to obtain both the high-throughput-criteria set (Ara-Set1, expected 330) and the one-class SVM set (Ara-Set2, expected 308), recording the 208-member overlap. Collect all reproduced counts into arabidopsis_benchmark_counts.csv and validate each value against the paper's results section; any mismatch flags a reproducibility failure or a database-version drift.

## Related tools

- **iwa-miRNA** (Galaxy-based framework whose modules (miRNARetrival, genomePrepare, miRNAPredict, miRNATranslate, miRNASelection) are re-executed to reproduce the benchmark counts) — https://github.com/cma2015/iwa-miRNA
- **Galaxy** (Web platform in which the iwa-miRNA modules are organized with user-friendly interfaces for reproducible re-execution)

## Evaluation signals

- miRNARetrival against the four databases returns exactly 365 miRNA precursors corresponding to 625 mature forms
- miRNAPredict on the 1063 SRA datasets yields exactly 435 predicted miRNA precursors
- miRNATranslate aggregation produces 667 precursor candidates and 1190 mature candidates (Table S2)
- miRNASelection with default parameters yields 330 Ara-Set1 (high-throughput criteria) and 308 Ara-Set2 (one-class SVM) candidates with a 208-member overlap
- Every row of arabidopsis_benchmark_counts.csv equals the value reported in the paper's results section

## Limitations

- It cannot handle sRNA-Seq data from species without genome sequences
- Sequencing reads that do not map to the reference genome are ignored in the current version
- The aggregated candidate list certainly contains false positive hits; wet experiments are still required to validate candidates before functional investigation
- Database versions must match the paper (miRBase v22.1, PmiREN v1.0, sRNAanno v1.0, PsRNA v1.0) or counts will drift from the published values

## Evidence

- [results] Using 1063 sRNA-Seq datasets (File S1) for the Columbia ecotype of A. thaliana as inputs, iwa-miRNA predicted 435 miRNA precursors: "Using 1063 sRNA-Seq datasets (File S1) for the Columbia ecotype of A. thaliana as inputs, iwa-miRNA predicted 435 miRNA precursors"
- [results] we obtained a total of 365 miRNA precursors corresponding to 625 mature forms from the four databases: "we obtained a total of 365 miRNA precursors corresponding to 625 mature forms from the four databases"
- [results] There were 330 miRNA precursor candidates that passed the high-throughput criteria (denoted as Ara-Set1): "There were 330 miRNA precursor candidates that passed the high-throughput criteria (denoted as Ara-Set1)"
- [results] iwa-miRNA built a one-class SVM classifier to predict 308 miRNA precursor candidates (Ara-Set2) as true positives: "iwa-miRNA built a one-class SVM classifier to predict 308 miRNA precursor candidates (Ara-Set2) as true positives"
- [readme] Test datasets referred in the tutorials for iwa-miRNA: https://github.com/cma2015/iwa-miRNA/tree/master/Test_data: "Test datasets referred in the tutorials for iwa-miRNA: https://github.com/cma2015/iwa-miRNA/tree/master/Test_data"
- [discussion] It cannot handle sRNA-Seq data from species without genome sequences.: "It cannot handle sRNA-Seq data from species without genome sequences."
