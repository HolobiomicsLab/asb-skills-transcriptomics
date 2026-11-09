---
name: fastq-read-sequence-collapsing
description: Use when a small RNA-seq FASTQ(.gz) file must be reduced to unique sequences
  with per-sequence counts before alignment and counting, e.g. producing collapsed.fa
  as consumed by tiny-count in the tinyRNA pipeline.
when_to_use_negative:
- The input is already a collapsed FASTA with count-bearing headers — re-collapsing
  would double-count or corrupt the embedded count fields.
- Per-read information (individual read identities, per-read alignment positions)
  must be preserved downstream, since collapsing discards everything but the unique
  sequence and its total count.
- The FASTQ file is not demultiplexed — the tinyRNA pipeline requires demultiplexed
  sequencing data as input.
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3169
tools:
- name: tiny-collapse
  role: 'Performs the collapse: counts unique sequences from the sequencing data and
    emits collapsed FASTA records with counts embedded in identifiers, optionally
    trimming adapter-derived degenerate bases.'
  repo: https://github.com/MontgomeryLab/tinyRNA
- name: tinyRNA
  role: Workflow that coordinates the collapse step (via CWL) and feeds collapsed.fa
    into the downstream alignment and counting steps.
  repo: https://github.com/MontgomeryLab/tinyRNA
- name: Python
  role: Implementation language of the tinyRNA tools, including the Python-based FASTX
    parsing used to read and group the FASTQ reads during collapse.
  repo: https://github.com/MontgomeryLab/tinyRNA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad065
    title: 'tiny-count: a counting tool for hierarchical classification and quantification
      of small RNA-seq reads with single-nucleotide precision'
  source_content_hash: sha256:5d03f343f3eecbc8c99388ab5809c5438c4c503ac459916da86ec2ba193d415d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MontgomeryLab/tinyRNA
  tools:
  - CWL
  - Python
  - tiny-collapse
  - tinyRNA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3169
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# fastq-read-sequence-collapsing

## Summary

Collapses demultiplexed small RNA-seq FASTQ reads into a collapsed FASTA in which each unique nucleotide sequence appears exactly once with its occurrence count embedded in the identifier (FASTX-Toolkit collapse convention), as done by the tiny-collapse step of the tinyRNA workflow to shrink the input for downstream genomic alignment and feature counting.

## When to use

Use when a small RNA-seq FASTQ(.gz) file must be reduced to unique sequences with per-sequence counts before alignment and counting, e.g. producing collapsed.fa as consumed by tiny-count in the tinyRNA pipeline.

## When NOT to use

- The input is already a collapsed FASTA with count-bearing headers — re-collapsing would double-count or corrupt the embedded count fields.
- Per-read information (individual read identities, per-read alignment positions) must be preserved downstream, since collapsing discards everything but the unique sequence and its total count.
- The FASTQ file is not demultiplexed — the tinyRNA pipeline requires demultiplexed sequencing data as input.

## Inputs

- Demultiplexed small RNA-seq sequencing data in FASTQ format (plain or gzip-compressed, per tinyRNA input requirements)
- Optional adapter sequence information for trimming degenerate bases introduced during library preparation

## Outputs

- collapsed.fa — a collapsed FASTA file with one record per unique sequence, each identifier embedding the per-sequence count in the FASTX-Toolkit collapse convention
- Per-sequence occurrence counts usable as abundance values in downstream tiny-count quantification

## How to apply

Read the small RNA FASTQ file with a Python-based FASTX parser (the tinyRNA tools are implemented in Python and coordinated with CWL). Group reads by identical nucleotide sequence, tallying the occurrence count of each unique sequence in memory rather than writing per-read intermediates. Emit each unique sequence exactly once as a FASTA record whose identifier embeds the per-sequence count in the FASTX-Toolkit collapse convention (e.g. a count field in the header). Write all collapsed records to collapsed.fa, the file consumed by the downstream alignment and feature counting steps of the tinyRNA workflow. Optionally trim degenerate bases that often originate from library-preparation adapter sequences before collapsing. The rationale is that collapsing substantially reduces the resource demands of genomic alignment and counting, since tiny-count then processes one record per unique sequence instead of one record per raw read.

## Related tools

- **tiny-collapse** (Performs the collapse: counts unique sequences from the sequencing data and emits collapsed FASTA records with counts embedded in identifiers, optionally trimming adapter-derived degenerate bases.) — https://github.com/MontgomeryLab/tinyRNA
- **tinyRNA** (Workflow that coordinates the collapse step (via CWL) and feeds collapsed.fa into the downstream alignment and counting steps.) — https://github.com/MontgomeryLab/tinyRNA
- **Python** (Implementation language of the tinyRNA tools, including the Python-based FASTX parsing used to read and group the FASTQ reads during collapse.) — https://github.com/MontgomeryLab/tinyRNA

## Evaluation signals

- collapsed.fa parses as valid FASTA without errors.
- Every FASTA identifier carries a count field consistent with the FASTX-Toolkit collapse convention (qualitative acceptance check; no numeric threshold is reported in the paper).
- Each unique nucleotide sequence appears exactly once in collapsed.fa (no duplicate sequence records).
- The sum of all embedded per-sequence counts equals the total number of reads in the input FASTQ.
- Downstream tiny-count alignment and counting runs successfully against collapsed.fa as its read input.

## Limitations

- The paper reports only a qualitative acceptance check for the collapsed output (valid FASTA, count-bearing identifiers), with no numeric threshold for collapse quality.
- Collapsing discards per-read information; only unique sequences and their totals survive, which constrains analyses needing read-level detail.
- tinyRNA stable-release installation may have issues on Apple Silicon-based systems, which can affect running the collapse step via the full workflow.
- Trimming degenerate bases applies specifically to degenerate bases often included in library-preparation adapter sequences; it is not a general quality-trimming step.

## Evidence

- [abstract] tiny-count can be run alone or as part of tinyRNA, a workflow that provides a basic all-in-one command line-based solution for small RNA-seq data analysis: "tiny-count can be run alone or as part of tinyRNA, a workflow that provides a basic all-in-one command line-based solution for small RNA-seq data analysis"
- [abstract] tiny-count and other tinyRNA tools are implemented in Python, C++, Cython, and R, and the workflow is coordinated with CWL.: "tiny-count and other tinyRNA tools are implemented in Python, C++, Cython, and R, and the workflow is coordinated with CWL."
- [readme] tinyRNA is a set of tools to simplify the analysis of next-generation sequencing data.: "tinyRNA is a set of tools to simplify the analysis of next-generation sequencing data."
- [readme] Files must be demultiplexed.: [evidence span withheld — anchor status: external_doc]
