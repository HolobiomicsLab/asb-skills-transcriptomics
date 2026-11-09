---
name: cross-database-candidate-overlap-counting
description: Use when a combined list of annotated plus newly predicted miRNA precursors
  must be tallied against miRBase, PmiREN, sRNAanno, and PsRNA to count database-specific
  overlaps and identify previously unannotated candidates.
when_to_use_negative:
- The sRNA-Seq reads derive from a species without a reference genome, since iwa-miRNA
  cannot handle sRNA-Seq data from species without genome sequences
- Reads do not map to the reference genome — the current version ignores unmapped
  sequencing reads, so overlap counts would be incomplete
- The goal is downstream functional analysis of the candidates (e.g., GO annotation
  or miRNA set enrichment), which the MiRNA Compilation overlap tally does not provide
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_0780
tools:
- name: iwa-miRNA
  role: Galaxy-based framework whose MiRNA Compilation module runs miRNARetrival,
    miRNAPredict, and miRNATranslate to merge annotated and predicted precursors into
    one candidate list for overlap counting
  repo: https://github.com/cma2015/iwa-miRNA
- name: Galaxy
  role: Hosts the iwa-miRNA modules with user-friendly web interfaces so the aggregation
    and merging steps are reproducible without command-line skills
  repo: https://github.com/cma2015/iwa-miRNA
- name: miRBase v22.1
  role: One of the four representative plant miRNA databases whose annotations are
    aggregated and against which candidate overlap is tallied
- name: PmiREN v1.0
  role: Source database for already annotated precursors included in the aggregation
    and overlap count
- name: sRNAanno v1.0
  role: Source database for already annotated precursors included in the aggregation
    and overlap count
- name: PsRNA v1.0
  role: Source database for already annotated precursors included in the aggregation
    and overlap count
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2021.02.010
    title: Interactive Web-Based Annotation of Plant MicroRNAs with iwa-miRNA
  source_content_hash: sha256:a410b826d3a14e03afc69eaccc4ad383e38af56f479a75b1025c16db1bf981de
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cma2015/iwa-miRNA
  tools:
  - Galaxy
  - PmiREN v1.0
  - PsRNA v1.0
  - iwa-miRNA
  - miRBase v22.1
  - sRNAanno v1.0
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_0780
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# Cross-database candidate overlap counting

## Summary

Quantifies how many combined miRNA precursor candidates produced by the iwa-miRNA MiRNA Compilation module overlap each of the four source databases (miRBase, PmiREN, sRNAanno, PsRNA), revealing which predictions are novel relative to existing annotation.

## When to use

Use when a combined list of annotated plus newly predicted miRNA precursors must be tallied against miRBase, PmiREN, sRNAanno, and PsRNA to count database-specific overlaps and identify previously unannotated candidates.

## When NOT to use

- The sRNA-Seq reads derive from a species without a reference genome, since iwa-miRNA cannot handle sRNA-Seq data from species without genome sequences
- Reads do not map to the reference genome — the current version ignores unmapped sequencing reads, so overlap counts would be incomplete
- The goal is downstream functional analysis of the candidates (e.g., GO annotation or miRNA set enrichment), which the MiRNA Compilation overlap tally does not provide

## Inputs

- Already annotated miRNA precursors from miRBase v22.1, PmiREN v1.0, sRNAanno v1.0, and PsRNA v1.0 (output of miRNARetrival)
- Predicted miRNA precursors from sRNA-Seq data (SRA accession numbers or uploaded fastq files; output of miRNAPredict)
- Formatted reference genome sequences and annotation (output of genomePrepare)

## Outputs

- Combined miRNA precursor candidate list in GFF3 format (e.g., 667 precursors / 1190 mature candidates for A. thaliana)
- Per-database overlap tallies indicating how many candidates overlap each of the four source databases
- Count of predicted precursors not previously annotated in any of the four databases (e.g., 302 of 435 in A. thaliana)

## How to apply

First run miRNARetrival with the species name to aggregate already annotated miRNA precursors from miRBase v22.1, PmiREN v1.0, sRNAanno v1.0, and PsRNA v1.0, then run miRNAPredict on sRNA-Seq inputs (SRA accession numbers or fastq files) mapping reads to the reference genome with GMAP. Run miRNATranslate on the outputs of miRNARetrival and miRNAPredict to convert annotated and predicted miRNAs into a uniform genomic coordinate system — this normalization is what makes overlap counting possible. Merge the translated annotated and predicted precursors into a single candidate list (GFF3) and, for each candidate, determine which of the four databases it is annotated in; tally candidates per database and count those present in none (novel). For the A. thaliana benchmark, the run should yield 365 annotated precursors (625 mature forms), 435 predicted precursors (302 unannotated), and a merged 667 precursor candidates corresponding to 1190 mature candidates.

## Related tools

- **iwa-miRNA** (Galaxy-based framework whose MiRNA Compilation module runs miRNARetrival, miRNAPredict, and miRNATranslate to merge annotated and predicted precursors into one candidate list for overlap counting) — https://github.com/cma2015/iwa-miRNA
- **Galaxy** (Hosts the iwa-miRNA modules with user-friendly web interfaces so the aggregation and merging steps are reproducible without command-line skills) — https://github.com/cma2015/iwa-miRNA
- **miRBase v22.1** (One of the four representative plant miRNA databases whose annotations are aggregated and against which candidate overlap is tallied)
- **PmiREN v1.0** (Source database for already annotated precursors included in the aggregation and overlap count)
- **sRNAanno v1.0** (Source database for already annotated precursors included in the aggregation and overlap count)
- **PsRNA v1.0** (Source database for already annotated precursors included in the aggregation and overlap count)

## Examples

```
iwa-miRNA: run miRNARetrival --species Arabidopsis_thaliana; miRNAPredict --input SRA_accessions --reference genome.fa; miRNATranslate --annotated out --predicted out; merge and tally overlaps vs 4 databases
```

## Evaluation signals

- The combined output file exists and is in GFF3 format
- Annotated-pool recovery matches the database aggregation expectation (A. thaliana: 365 precursors / 625 mature forms from the four databases)
- Prediction-pool scale matches sRNA-Seq expectation (A. thaliana: 435 predicted precursors from 1063 sRNA-Seq datasets)
- Novelty count matches the benchmark (302 of 435 predicted A. thaliana precursors absent from all four databases)
- Merged totals match the paper-reported benchmark: 667 precursor candidates corresponding to 1190 mature candidates

## Limitations

- The aggregated candidate list certainly contains false positive hits, so overlap counting alone does not certify candidates
- Cannot handle sRNA-Seq data from species without genome sequences
- Sequencing reads that do not map to the reference genome are ignored in the current version
- Further wet experiments are required to validate candidates before functional investigation

## Evidence

- [methods] Run miRNATranslate to translate annotated and predicted miRNAs into a uniform genomic coordinate system.: [evidence span withheld — anchor status: not_found]
- [results] Merge annotated and predicted precursors and tally database overlaps per the A. thaliana workflow.: "This resulted in the identification of 667 miRNA precursor candidates, corresponding to 1190 mature miRNA candidates (Table S2)"
- [intro] iwa-miRNA not only provides functions for automatically incorporating miRNA annotations from four representative databases (i.e., miRBase, PmiREN, sRNAanno, and PsRNA) but also builds a: "iwa-miRNA not only provides functions for automatically incorporating miRNA annotations from four representative databases (i.e., miRBase, PmiREN, sRNAanno, and PsRNA)"
- [discussion] The aggregation of annotated and predicted miRNAs yields candidates containing false positives.: "The aggregation of already annotated and newly predicted miRNAs generates a comprehensive collection of miRNA candidates, which certainly contain false positive hits"
- [readme] The three functional modules of iwa-miRNA include MiRNA Compilation as described in the README.: "The iwa-miRNA is composed with three functional modules: MiRNA Compilation, MiRNA Selection, and Manual Curation."
