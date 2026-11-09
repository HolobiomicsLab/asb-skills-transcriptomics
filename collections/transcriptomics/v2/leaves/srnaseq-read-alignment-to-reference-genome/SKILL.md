---
name: srnaseq-read-alignment-to-reference-genome
description: Use when you have sRNA-Seq data (SRA accession numbers or FASTQ files)
  for a plant species WITH a reference genome and need to predict novel miRNA precursors
  from mapped reads.
when_to_use_negative:
- The species has no reference genome sequence — iwa-miRNA cannot handle sRNA-Seq
  data from species without genome sequences
- Most sequencing reads do not map to the reference genome — unmapped reads are ignored
  in the current version, so no candidates will be derived from them
edam_operation: http://edamontology.org/operation_3198
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0659
tools:
- name: iwa-miRNA
  role: Framework providing the miRNAPredict module that maps sRNA-Seq reads to the
    reference genome with GMAP and predicts novel miRNA precursors
  repo: https://github.com/cma2015/iwa-miRNA
- name: Galaxy
  role: Web platform hosting the iwa-miRNA modules so alignment/prediction can be
    run without computational skills
  repo: https://github.com/cma2015/iwa-miRNA
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2021.02.010
    title: Interactive Web-Based Annotation of Plant MicroRNAs with iwa-miRNA
  source_content_hash: sha256:0d4cd7fbdd31b018445a088ed8f95382880196d737b4fe80bc6dd96e2e4f7779
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cma2015/iwa-miRNA
  tools:
  - Galaxy
  - iwa-miRNA
  edam_operation: http://edamontology.org/operation_3198
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0659
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# sRNA-Seq read alignment to reference genome

## Summary

Aligns small RNA sequencing reads to a formatted plant reference genome using GMAP (via the iwa-miRNA miRNAPredict module) to predict novel miRNA precursor candidates that are later merged with database-annotated precursors.

## When to use

Use when you have sRNA-Seq data (SRA accession numbers or FASTQ files) for a plant species WITH a reference genome and need to predict novel miRNA precursors from mapped reads.

## When NOT to use

- The species has no reference genome sequence — iwa-miRNA cannot handle sRNA-Seq data from species without genome sequences
- Most sequencing reads do not map to the reference genome — unmapped reads are ignored in the current version, so no candidates will be derived from them

## Inputs

- sRNA-Seq reads as SRA accession numbers
- sRNA-Seq reads as uploaded FASTQ files
- Formatted reference genome sequences and annotation (from genomePrepare)

## Outputs

- Predicted (novel) miRNA precursors from sRNA-Seq data
- Precursors subsequently translated by miRNATranslate into a uniform genomic coordinate format

## How to apply

First run genomePrepare with the species name or genome sequences and annotation to produce formatted reference genome sequences and annotation. Then run miRNAPredict, providing sRNA-Seq inputs as either SRA accession numbers or uploaded FASTQ files; reads are mapped to the prepared reference genome with GMAP, and novel miRNA precursors are predicted from the small RNA sequencing data. In the A. thaliana benchmark, 1063 publicly available sRNA-Seq datasets from the Columbia ecotype were used, yielding 435 predicted miRNA precursors (302 previously unannotated in miRBase, PmiREN, sRNAanno, or PsRNA). Output precursors then pass through miRNATranslate to a uniform genomic coordinate system before merging with annotated precursors. Correct application is judged by producing predicted precursors compatible with miRNATranslate input and, for the benchmark, by recovering the paper-reported prediction scale.

## Related tools

- **iwa-miRNA** (Framework providing the miRNAPredict module that maps sRNA-Seq reads to the reference genome with GMAP and predicts novel miRNA precursors) — https://github.com/cma2015/iwa-miRNA
- **Galaxy** (Web platform hosting the iwa-miRNA modules so alignment/prediction can be run without computational skills) — https://github.com/cma2015/iwa-miRNA

## Evaluation signals

- miRNAPredict runs successfully only after genomePrepare has produced formatted reference genome sequences and annotation
- Predicted precursors are accepted as input by miRNATranslate, which emits miRNAs and precursors in a uniform format
- For the A. thaliana Columbia benchmark with 1063 sRNA-Seq datasets, the run recovers approximately 435 predicted miRNA precursors, of which 302 were previously unannotated
- Merging annotated (365 precursors / 625 mature forms) and predicted precursors yields the benchmark 667 precursor candidates / 1190 mature candidates
- Combined candidate list is output in GFF3 format

## Limitations

- Cannot handle sRNA-Seq data from species without reference genome sequences
- The current version ignores sequencing reads that do not map to the reference genome
- Predicted precursors merged with annotated ones certainly contain false positive hits and require wet-lab validation

## Evidence

- [methods] miRNAPredict accepts sRNA-Seq inputs and maps reads to the reference genome: [evidence span withheld — anchor status: not_found]
- [methods] genomePrepare formats the genome before alignment: [evidence span withheld — anchor status: not_found]
- [results] A. thaliana prediction benchmark scale: "Using 1063 sRNA-Seq datasets (File S1) for the Columbia ecotype of A. thaliana as inputs, iwa-miRNA predicted 435 miRNA precursors, 302 of which were not previously annotated"
- [results] Combined output scale of MiRNA Compilation: "This resulted in the identification of 667 miRNA precursor candidates, corresponding to 1190 mature miRNA candidates (Table S2)"
- [discussion] Unmapped reads are discarded, limiting recall: "In the current version, iwa-miRNA ignores sequencing reads that do not map to the reference genome."
