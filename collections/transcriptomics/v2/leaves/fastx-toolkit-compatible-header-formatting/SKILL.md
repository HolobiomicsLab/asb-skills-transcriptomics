---
name: fastx-toolkit-compatible-header-formatting
description: Use when you hold demultiplexed small RNA FASTQ reads and need a count-annotated
  collapsed FASTA (collapsed.fa) to feed the alignment and counting steps of a small
  RNA-seq pipeline such as tinyRNA/tiny-count.
when_to_use_negative:
- Input is already a collapsed FASTA or a counts/feature table rather than raw demultiplexed
  FASTQ reads
- Sequencing data is not demultiplexed — the workflow requires per-sample FASTQ files,
  and FASTQ(.gz) inputs must be demultiplexed
- The downstream counting tool accepts raw FASTQ directly and collapsing is not needed
  (tiny-count can be run standalone without the full tinyRNA pipeline)
edam_operation: http://edamontology.org/operation_0004
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: tinyRNA
  role: End-to-end small RNA-seq workflow whose data pre-processing step produces
    the collapsed FASTA files via the tiny-collapse tool
  repo: https://github.com/MontgomeryLab/tinyRNA
- name: tiny-count
  role: Downstream counting tool that consumes the collapsed FASTA for hierarchical
    feature counting; installable standalone via Bioconda
  repo: https://anaconda.org/bioconda/tiny-count
- name: Python
  role: Implementation language for tinyRNA tools; a Python-based FASTX parser reads
    the FASTQ and tallies unique sequences
  repo: https://github.com/MontgomeryLab/tinyRNA
- name: CWL
  role: Coordinates the tinyRNA workflow steps, including the collapse step feeding
    alignment and counting
  repo: https://github.com/MontgomeryLab/tinyRNA
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad065
    title: 'tiny-count: a counting tool for hierarchical classification and quantification
      of small RNA-seq reads with single-nucleotide precision'
  source_content_hash: sha256:68bf80bd999f6d303c875f41945f2c42af9843f95467df3eb3939e194d1c3d83
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MontgomeryLab/tinyRNA
  tools:
  - CWL
  - Python
  - tiny-count
  - tinyRNA
  edam_operation: http://edamontology.org/operation_0004
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# fastx-toolkit-compatible-header-formatting

## Summary

Collapse a small RNA FASTQ into a non-redundant collapsed.fa in which each unique sequence appears once and its FASTA identifier embeds the per-sequence read count in the FASTX-Toolkit collapse convention, optionally trimming degenerate adapter bases. This is the tiny-collapse step of the tinyRNA workflow that shrinks the input for downstream genomic alignment and feature counting.

## When to use

Use when you hold demultiplexed small RNA FASTQ reads and need a count-annotated collapsed FASTA (collapsed.fa) to feed the alignment and counting steps of a small RNA-seq pipeline such as tinyRNA/tiny-count.

## When NOT to use

- Input is already a collapsed FASTA or a counts/feature table rather than raw demultiplexed FASTQ reads
- Sequencing data is not demultiplexed — the workflow requires per-sample FASTQ files, and FASTQ(.gz) inputs must be demultiplexed
- The downstream counting tool accepts raw FASTQ directly and collapsing is not needed (tiny-count can be run standalone without the full tinyRNA pipeline)

## Inputs

- Demultiplexed small RNA sequencing reads in FASTQ (optionally gzipped) format
- Optional adapter sequence parameters for degenerate-base trimming

## Outputs

- collapsed.fa — a FASTA file with one record per unique sequence, each identifier embedding the per-sequence count in the FASTX-Toolkit collapse convention

## How to apply

Read the small RNA FASTQ with a Python-based FASTX parser, group reads by identical nucleotide sequence, and tally each unique sequence's occurrence count in memory. Emit every unique sequence exactly once as a FASTA record whose identifier embeds the count in the FASTX-Toolkit collapse convention (e.g. a header of the form 'count_N'), and optionally trim degenerate bases often included in library-preparation adapter sequences before collapsing. Write all records to collapsed.fa, which is the file consumed by the downstream alignment and tiny-count feature-counting steps of the tinyRNA pipeline. Collapsing is what substantially reduces the resource demands of the alignment and counting stages, because aligners see one copy per unique sequence rather than every raw read.

## Related tools

- **tinyRNA** (End-to-end small RNA-seq workflow whose data pre-processing step produces the collapsed FASTA files via the tiny-collapse tool) — https://github.com/MontgomeryLab/tinyRNA
- **tiny-count** (Downstream counting tool that consumes the collapsed FASTA for hierarchical feature counting; installable standalone via Bioconda) — https://anaconda.org/bioconda/tiny-count
- **Python** (Implementation language for tinyRNA tools; a Python-based FASTX parser reads the FASTQ and tallies unique sequences) — https://github.com/MontgomeryLab/tinyRNA
- **CWL** (Coordinates the tinyRNA workflow steps, including the collapse step feeding alignment and counting) — https://github.com/MontgomeryLab/tinyRNA

## Examples

```
conda activate tinyrna; tiny run
```

## Evaluation signals

- collapsed.fa parses as valid FASTA
- Every FASTA identifier carries a count field consistent with the FASTX-Toolkit-compatible format (qualitative acceptance check; no numeric threshold is reported in the paper)
- Each unique nucleotide sequence appears exactly once in collapsed.fa
- The sum of the embedded counts equals the total number of reads in the input FASTQ
- Downstream alignment and tiny-count steps run without format errors on collapsed.fa

## Limitations

- No numeric acceptance threshold for the collapsed output is reported in the paper — validation is a qualitative format check
- FASTQ inputs must be demultiplexed before this step; the pipeline does not multiplex/demultiplex
- Degenerate-base trimming is optional and targets adapter-derived degenerate bases; untrimmed degenerate bases may otherwise split otherwise-identical sequences into distinct records

## Evidence

- [full_text] workflow collapse step: [evidence span withheld — anchor status: not_found]
- [full_text] rationale for collapsing: [evidence span withheld — anchor status: not_found]
- [abstract] implementation and workflow coordination: "tiny-count and other tinyRNA tools are implemented in Python, C++, Cython, and R, and the workflow is coordinated with CWL."
- [readme] input requirement: [evidence span withheld — anchor status: not_found]
