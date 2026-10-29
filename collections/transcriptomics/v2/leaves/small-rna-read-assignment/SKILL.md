---
name: small-rna-read-assignment
description: Use when assigning demultiplexed small RNA-seq reads (FASTQ) to features
  from GFF3/GFF2/GTF annotations, especially when loci produce multiple small RNA
  classes (piRNA vs siRNA) or when miRNA/isomiR variants must be distinguished by
  single-nucleotide precision.
when_to_use_negative:
- FASTQ input is not demultiplexed — the README requires 'Files must be demultiplexed'
  before tiny-count can process it
- Feature annotation lacks the required column 9 ID/gene_id/Parent tags or uses multi-valued
  Parent tags, which tiny-count does not yet support
- The task is differential expression analysis of already-computed count matrices
  rather than read-level feature assignment — that is a downstream tinyRNA step
edam_operation: http://edamontology.org/operation_3799
tools:
- name: tiny-count
  role: Performs the hierarchical classification and quantification of small RNA reads
    against reference features
  repo: github.com/MontgomeryLab/tinyRNA
- name: Cython
  role: Language for the custom StepVector that performs interval-overlap queries
    against stored feature intervals, replacing HTSeq's version
- name: tinyRNA
  role: End-to-end workflow in which tiny-count can be run, coordinated with CWL
  repo: https://github.com/MontgomeryLab/tinyRNA
- name: HTSeq
  role: Baseline StepVector implementation used in equivalence tests and runtime comparisons
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad065
    title: 'tiny-count: a counting tool for hierarchical classification and quantification
      of small RNA-seq reads with single-nucleotide precision'
  source_content_hash: sha256:7f5bdb1b46bbee054b4430f87f468a46133e1fb4a6fd88642788e9cf063914b3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Cython
  - HTSeq
  - Python
  - tiny-count
  - tinyRNA
  edam_operation: http://edamontology.org/operation_3799
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# small-rna-read-assignment

## Summary

Assign and quantify small RNA-seq reads to reference features using tiny-count's interval-overlap machinery (a custom Cython StepVector replacing HTSeq's, with ~30% average runtime reduction on the read-assignment workload), resolving distinct small RNA classes such as piRNAs and siRNAs from the same locus with single-nucleotide precision.

## When to use

Use when assigning demultiplexed small RNA-seq reads (FASTQ) to features from GFF3/GFF2/GTF annotations, especially when loci produce multiple small RNA classes (piRNA vs siRNA) or when miRNA/isomiR variants must be distinguished by single-nucleotide precision.

## When NOT to use

- FASTQ input is not demultiplexed — the README requires 'Files must be demultiplexed' before tiny-count can process it
- Feature annotation lacks the required column 9 ID/gene_id/Parent tags or uses multi-valued Parent tags, which tiny-count does not yet support
- The task is differential expression analysis of already-computed count matrices rather than read-level feature assignment — that is a downstream tinyRNA step

## Inputs

- Demultiplexed sequencing data in FASTQ (optionally gzipped)
- Reference annotations in GFF3 / GFF2 / GTF with ID, gene_id, or Parent attributes in column 9
- Reference genome in FASTA with case-sensitive chromosome identifiers matching the annotation file
- tinyRNA configuration files: Run Config, Paths File, Features Sheet (selection rules), Samples Sheet

## Outputs

- Feature-level small RNA counts from hierarchical classification and quantification
- Pipeline counts and statistics outputs of the tinyRNA workflow
- Benchmark/equivalence measurements (e.g. stepvector_benchmark.csv with equivalence flags and per-run wall-clock times)

## How to apply

Compile tiny-count's piecewise-constant StepVector replacement in Cython alongside its Python codebase; this replaces HTSeq's StepVector, whose iteration and intermediate object creation were the bottleneck for resolving features overlapping an alignment interval. Run interval-overlap queries of each aligned read's interval against the stored feature intervals loaded from the GFF3/GFF2/GTF annotation (features must carry ID, gene_id, or Parent tags in column 9). Apply hierarchical selection rules to filter candidate features by 5' nucleotide, read length, alignment position relative to reference features, and mismatch count, so that distinct classes produced from the same locus (e.g. piRNAs and siRNAs) are resolved. Accept the implementation only if returned feature sets match HTSeq StepVector exactly (zero mismatches) for identical query intervals, and record per-run wall-clock times to confirm the expected ~30% average runtime reduction on the read-assignment workload. tiny-count can be run standalone or as part of the tinyRNA workflow coordinated with CWL.

## Related tools

- **tiny-count** (Performs the hierarchical classification and quantification of small RNA reads against reference features) — github.com/MontgomeryLab/tinyRNA
- **Cython** (Language for the custom StepVector that performs interval-overlap queries against stored feature intervals, replacing HTSeq's version)
- **tinyRNA** (End-to-end workflow in which tiny-count can be run, coordinated with CWL) — https://github.com/MontgomeryLab/tinyRNA
- **HTSeq** (Baseline StepVector implementation used in equivalence tests and runtime comparisons)

## Examples

```
conda install -c bioconda -c conda-forge tiny-count
```

## Evaluation signals

- Interval-overlap queries against the Cython StepVector return feature sets identical to HTSeq StepVector's (zero mismatches) for every tested query interval
- Average wall-clock runtime on the tiny-count read-assignment workload is reduced relative to the HTSeq StepVector baseline (~30%, hardware-dependent)
- Distinct small RNA classes produced from the same locus (piRNAs vs siRNAs) are resolved into separate counts
- miRNA vs isomiR variants are distinguished with single-nucleotide precision
- Selection rules correctly gate features by 5' nucleotide, length, alignment position relative to reference features, and mismatch number

## Limitations

- The Cython StepVector's ~30% runtime reduction is hardware-dependent
- Multi-valued Parent tags in GFF annotations are not yet supported
- Chromosome identifiers in the FASTA reference are case sensitive and must exactly match the annotation file
- Stable-release installation may fail on Apple Silicon-based systems (development release addresses this)

## Evidence

- [abstract] tiny-count is a highly flexible counting tool: "tiny-count is a highly flexible counting tool that allows for hierarchical classification and quantification of small RNA reads from high-throughput sequencing data."
- [abstract] Selection rules filter reads: "Selection rules can be used to filter reads by 5′ nucleotide, length, position of alignments in relation to reference features, and by the number of mismatches to reference sequences."
- [abstract] single-nucleotide precision: "It can distinguish small RNA variants, such as miRNAs and isomiRs, with single-nucleotide precision."
- [abstract] implementation languages: "tiny-count and other tinyRNA tools are implemented in Python, C++, Cython, and R, and the workflow is coordinated with CWL."
- [readme] annotation requirements: [evidence span withheld — anchor status: external_doc]
- [readme] demultiplexing requirement: [evidence span withheld — anchor status: not_found]
