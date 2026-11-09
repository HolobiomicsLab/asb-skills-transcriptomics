---
name: small-rna-seq-preprocessing
description: Use when small RNA-seq FASTQ files (demultiplexed, optionally .gz) must
  be reduced to unique sequences with counts before genomic alignment and feature
  counting in the tinyRNA workflow, or when adapter-derived degenerate bases need
  trimming.
when_to_use_negative:
- Input reads are already collapsed (each FASTA record already carries a count field)
  — re-collapsing would misattribute counts
- Input is a count matrix or feature table rather than raw sequencing reads
- Analysis is not small RNA-seq within the tinyRNA context and requires per-read identities
  downstream of counting
edam_operation: http://edamontology.org/operation_0231
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: tinyRNA
  role: Workflow (coordinated with CWL) in which this collapse step runs and whose
    downstream alignment and counting steps consume collapsed.fa
  repo: https://github.com/MontgomeryLab/tinyRNA
- name: tiny-count
  role: Downstream counting tool that quantifies the collapsed FASTA with hierarchical
    feature selection rules
  repo: https://github.com/MontgomeryLab/tinyRNA
- name: Python
  role: Implementation language of the FASTX parser used to read the FASTQ and tally
    unique-sequence counts
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad065
    title: 'tiny-count: a counting tool for hierarchical classification and quantification
      of small RNA-seq reads with single-nucleotide precision'
  source_content_hash: sha256:254347c25afb96543ed6f02f808246a09eba6a0c7f7721a8d21bff967b993801
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MontgomeryLab/tinyRNA
  tools:
  - CWL
  - Python
  - tiny-count
  - tinyRNA
  edam_operation: http://edamontology.org/operation_0231
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# small-rna-seq-preprocessing

## Summary

Collapses demultiplexed small RNA-seq FASTQ reads into unique-sequence FASTA records whose identifiers embed per-sequence occurrence counts (FASTX-Toolkit collapse convention), producing collapsed.fa that feeds the tinyRNA alignment and counting steps while drastically reducing resource demands.

## When to use

Use when small RNA-seq FASTQ files (demultiplexed, optionally .gz) must be reduced to unique sequences with counts before genomic alignment and feature counting in the tinyRNA workflow, or when adapter-derived degenerate bases need trimming.

## When NOT to use

- Input reads are already collapsed (each FASTA record already carries a count field) — re-collapsing would misattribute counts
- Input is a count matrix or feature table rather than raw sequencing reads
- Analysis is not small RNA-seq within the tinyRNA context and requires per-read identities downstream of counting

## Inputs

- Demultiplexed small RNA-seq FASTQ files (.fastq or .fastq.gz)
- Optional adapter/trimming preferences for degenerate-base removal

## Outputs

- collapsed.fa — FASTA with one record per unique sequence, identifier embedding the per-sequence count (FASTX-Toolkit collapse convention)

## How to apply

Read the small RNA FASTQ with a Python-based FASTX parser and group reads by identical nucleotide sequence, tallying each unique sequence's occurrence count in memory. Emit each unique sequence exactly once as a FASTA record whose identifier embeds the count in the FASTX-Toolkit collapse convention, and write all records to collapsed.fa, the input consumed by downstream alignment and tiny-count feature counting. Optionally trim degenerate bases (e.g., Ns) that often come from adapter sequences used in library preparation. Validate that collapsed.fa parses as valid FASTA and that every identifier carries a count field consistent with the FASTX-Toolkit-compatible format — acceptance is qualitative, as the paper reports no numeric threshold.

## Related tools

- **tinyRNA** (Workflow (coordinated with CWL) in which this collapse step runs and whose downstream alignment and counting steps consume collapsed.fa) — https://github.com/MontgomeryLab/tinyRNA
- **tiny-count** (Downstream counting tool that quantifies the collapsed FASTA with hierarchical feature selection rules) — https://github.com/MontgomeryLab/tinyRNA
- **Python** (Implementation language of the FASTX parser used to read the FASTQ and tally unique-sequence counts)

## Examples

```
tiny run
```

## Evaluation signals

- collapsed.fa parses as valid FASTA
- Every FASTA identifier carries a count field consistent with the FASTX-Toolkit-compatible collapse format
- Each unique nucleotide sequence appears exactly once in collapsed.fa (no duplicate sequence records)
- Sum of embedded counts across records equals the number of reads in the input FASTQ
- Degenerate-base trimming (if enabled) removed only adapter-derived ambiguous bases without altering canonical nucleotide calls

## Limitations

- Acceptance is a qualitative check: the paper reports no numeric threshold for collapse correctness
- Degenerate-base trimming addresses adapter-derived ambiguous bases; other adapter contamination must be handled before collapsing
- Sequencing data must be demultiplexed per the tinyRNA input requirements; stable-release installation may have issues on Apple Silicon-based systems

## Evidence

- [abstract] tiny-count can be run alone or as part of tinyRNA, a workflow that provides a basic all-in-one command line-based solution for small RNA-seq data analysis: "tiny-count can be run alone or as part of tinyRNA, a workflow that provides a basic all-in-one command line-based solution for small RNA-seq data analysis"
- [abstract] tiny-count is a highly flexible counting tool that allows for hierarchical classification and quantification of small RNA reads from high-throughput sequencing data.: "tiny-count is a highly flexible counting tool that allows for hierarchical classification and quantification of small RNA reads from high-throughput sequencing data."
- [readme] You can execute the workflow in its entirety for a full end-to-end analysis pipeline, or you can execute individual steps on their own.: [evidence span withheld — anchor status: external_doc]
- [readme] Files must be demultiplexed.: [evidence span withheld — anchor status: external_doc]
