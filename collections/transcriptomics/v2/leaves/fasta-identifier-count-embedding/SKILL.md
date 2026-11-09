---
name: fasta-identifier-count-embedding
description: Use when raw demultiplexed small RNA-seq FASTQ reads must be reduced
  to unique sequences with embedded counts (FASTX-Toolkit-style headers) before alignment
  and counting, e.g. preparing collapsed.fa for tinyRNA.
when_to_use_negative:
- Input is already a collapsed FASTA with count-embedded identifiers (re-collapsing
  would discard the count information embedded in headers)
- Input is a feature count table or alignment output rather than raw sequencing reads
- Input FASTQ files are not demultiplexed, which the tinyRNA workflow requires of
  sequencing data
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3168
tools:
- name: tinyRNA
  role: Workflow within which the collapse step runs; produces collapsed FASTA files
    consumed by downstream alignment and counting
  repo: https://github.com/MontgomeryLab/tinyRNA
- name: Python
  role: Implementation language of the tinyRNA tools, including the FASTX-parser-based
    collapsing of unique sequences
- name: CWL
  role: Coordinates the tinyRNA workflow steps, including the collapse step, as an
    all-in-one command-line pipeline
- name: tiny-count
  role: Downstream counting tool that consumes the pipeline output after collapsing;
    installable standalone via Bioconda
  repo: https://anaconda.org/bioconda/tiny-count
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad065
    title: 'tiny-count: a counting tool for hierarchical classification and quantification
      of small RNA-seq reads with single-nucleotide precision'
  source_content_hash: sha256:0b5ff56457a135617a4b5dd63d485fea8b7bcea20a180b152ecc97785bd29239
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MontgomeryLab/tinyRNA
  tools:
  - CWL
  - Python
  - tiny-count
  - tinyRNA
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3168
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# fasta-identifier-count-embedding

## Summary

Collapse unique small RNA sequencing reads into a FASTA file in which each record's identifier embeds the per-sequence read count using the FASTX-Toolkit collapse convention, as done by the tiny-collapse step of the tinyRNA workflow. This reduces redundant sequences before downstream genomic alignment and feature counting with tiny-count.

## When to use

Use when raw demultiplexed small RNA-seq FASTQ reads must be reduced to unique sequences with embedded counts (FASTX-Toolkit-style headers) before alignment and counting, e.g. preparing collapsed.fa for tinyRNA.

## When NOT to use

- Input is already a collapsed FASTA with count-embedded identifiers (re-collapsing would discard the count information embedded in headers)
- Input is a feature count table or alignment output rather than raw sequencing reads
- Input FASTQ files are not demultiplexed, which the tinyRNA workflow requires of sequencing data

## Inputs

- Demultiplexed small RNA-seq FASTQ file (.fastq, .fastq.gz)
- Optional adapter sequence definition with degenerate bases to trim

## Outputs

- collapsed.fa: FASTA file with one record per unique sequence, identifier embedding the per-sequence count in FASTX-Toolkit collapse convention

## How to apply

Read the small RNA FASTQ file with a Python-based FASTX parser, then group reads by identical nucleotide sequence and tally each unique sequence's occurrence count in memory. Emit each unique sequence exactly once as a FASTA record whose identifier embeds the per-sequence count following the FASTX-Toolkit collapse convention (header carrying the count field). Optionally trim degenerate bases that are often included in adapter sequences used in library preparation. Write all collapsed records to collapsed.fa, which is the input consumed by the downstream alignment and feature-counting steps of the tinyRNA pipeline. Collapsing is what substantially reduces the resource demands of downstream genomic alignment and feature counting, because duplicated reads are represented once with a count instead of as separate entries.

## Related tools

- **tinyRNA** (Workflow within which the collapse step runs; produces collapsed FASTA files consumed by downstream alignment and counting) — https://github.com/MontgomeryLab/tinyRNA
- **Python** (Implementation language of the tinyRNA tools, including the FASTX-parser-based collapsing of unique sequences)
- **CWL** (Coordinates the tinyRNA workflow steps, including the collapse step, as an all-in-one command-line pipeline)
- **tiny-count** (Downstream counting tool that consumes the pipeline output after collapsing; installable standalone via Bioconda) — https://anaconda.org/bioconda/tiny-count

## Evaluation signals

- collapsed.fa parses as valid FASTA
- Every FASTA identifier carries a count field consistent with the FASTX-Toolkit-compatible collapse format
- Each unique nucleotide sequence appears exactly once in collapsed.fa (no duplicate sequence records)
- Sum of embedded counts equals the total number of input FASTQ reads (modulo any adapter-trimmed reads)
- collapsed.fa is accepted without error by the downstream alignment and counting steps of the tinyRNA pipeline

## Limitations

- The paper reports a qualitative acceptance check only: no numeric threshold is given for validating the collapsed output
- The tinyRNA README notes installation issues may arise on Apple Silicon–based systems for stable releases
- Collapse step requires demultiplexed FASTQ input; single-end small RNA reads in FASTQ(.gz) format

## Evidence

- [abstract] tiny-count can be run alone or as part of tinyRNA, a workflow that provides a basic all-in-one command line-based solution for small RNA-seq data analysis: "tiny-count can be run alone or as part of tinyRNA, a workflow that provides a basic all-in-one command line-based solution for small RNA-seq data analysis"
- [abstract] tiny-count and other tinyRNA tools are implemented in Python, C++, Cython, and R, and the workflow is coordinated with CWL: "tiny-count and other tinyRNA tools are implemented in Python, C++, Cython, and R, and the workflow is coordinated with CWL."
- [intro] tinyRNA is a set of tools to simplify the analysis of next-generation sequencing data: "tinyRNA is a set of tools to simplify the analysis of next-generation sequencing data. The goal of this specific repository is to provide an entire workflow for processing small RNA sequencing data"
- [readme] Files must be demultiplexed (README sequencing-data requirement): [evidence span withheld — anchor status: not_found]
