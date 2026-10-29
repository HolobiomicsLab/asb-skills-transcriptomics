---
name: machine-readable-assay-specification-parsing
description: Use when raw single-cell FASTQ reads must be preprocessed and the read
  structure (barcode/UMI/cDNA positions and strand) must be derived from an assay
  specification file rather than hand-written for a specific chemistry.
when_to_use_negative:
- Input is already a count matrix (cell x gene mtx/h5ad) — use matrix QC/filtering
  tools like mx filter/inspect instead of seqspec index
- The library chemistry/read structure is already known and no assay specification
  file exists — hand-writing the technology string may be simpler than authoring a
  seqspec from scratch
- The target tool is not supported by seqspec index's per-tool formatting (only tool-specific
  formats like kb are emitted)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0080
tools:
- name: seqspec
  role: Machine-readable assay specification and CLI whose `index` subcommand extracts
    barcode/UMI/cDNA Regions and formats the tool-specific technology string
  repo: https://github.com/IGVF/seqspec
- name: cellatlas
  role: 'Downstream consumer: uses seqspec and kallisto bustools to facilitate universal
    preprocessing of single-cell genomics data'
  repo: https://github.com/cellatlas/cellatlas.git
- name: kb-python
  role: Preprocessing tool (kallisto | bustools) whose `-t kb` technology string format
    is produced by seqspec index
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btae168
    title: A machine-readable specification for genomics assays
  source_content_hash: sha256:9e50ac34a700f008058a5ec3c2f6ce4b4e23c82898aea5e1bee6dfb03cbe5fe9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/IGVF/seqspec
  tools:
  - cellatlas
  - kb-python
  - seqspec
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0080
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# machine-readable-assay-specification-parsing

## Summary

Parse a machine-readable seqspec YAML assay specification to extract library structure (barcode, UMI, cDNA regions) and derive tool-specific technology strings for single-cell preprocessing tools such as STARsolo/kb.

## When to use

Use when raw single-cell FASTQ reads must be preprocessed and the read structure (barcode/UMI/cDNA positions and strand) must be derived from an assay specification file rather than hand-written for a specific chemistry.

## When NOT to use

- Input is already a count matrix (cell x gene mtx/h5ad) — use matrix QC/filtering tools like mx filter/inspect instead of seqspec index
- The library chemistry/read structure is already known and no assay specification file exists — hand-writing the technology string may be simpler than authoring a seqspec from scratch
- The target tool is not supported by seqspec index's per-tool formatting (only tool-specific formats like kb are emitted)

## Inputs

- seqspec assay specification YAML file (e.g. specs/dogmaseq-dig/spec.yaml from the IGVF/seqspec repository)
- FASTQ read files for the target modality (e.g. fastqs/rna_R1_SRR18677638.fastq.gz, rna_R2_SRR18677638.fastq.gz)
- modality identifier (e.g. rna, atac, tag, protein) present in the seqspec

## Outputs

- Tool-specific technology string: colon-separated start,end,strand triplets per read (e.g. 0,0,16:0,16,28:1,0,102)
- Technology string file (e.g. rna_technology_string.txt) consumed by STARsolo/kb preprocessing

## How to apply

Install seqspec from the IGVF/seqspec GitHub repository with pip, then obtain the validated assay spec YAML (e.g. specs/dogmaseq-dig/spec.yaml). Run `seqspec index` with the target tool (`-t kb`), the modality (`-m rna`), and the FASTQ reads for that modality (comma-separated R1/R2 gz files). The command extracts the barcode, UMI, and cDNA Regions from the seqspec, positionally indexes them, and formats them on a per-tool basis, emitting colon-separated start,end,strand triplets per read. Capture the emitted triplet string (expected form 0,0,16:0,16,28:1,0,102 for the RNA modality of the dogmaseq-dig assay) and write it to a technology string file for downstream use in STARsolo/kb preprocessing.

## Related tools

- **seqspec** (Machine-readable assay specification and CLI whose `index` subcommand extracts barcode/UMI/cDNA Regions and formats the tool-specific technology string) — https://github.com/IGVF/seqspec
- **cellatlas** (Downstream consumer: uses seqspec and kallisto bustools to facilitate universal preprocessing of single-cell genomics data) — https://github.com/cellatlas/cellatlas.git
- **kb-python** (Preprocessing tool (kallisto | bustools) whose `-t kb` technology string format is produced by seqspec index)

## Examples

```
seqspec index -t kb -m rna -r fastqs/rna_R1_SRR18677638.fastq.gz,rna_R2_SRR18677638.fastq.gz spec.yaml
```

## Evaluation signals

- The technology string file exists and its contents equal the seqspec index output, e.g. 0,0,16:0,16,28:1,0,102 for the RNA modality of dogmaseq-dig
- The string contains exactly one colon-separated start,end,strand triplet per read supplied via -r, with each triplet comma-separated into three integers
- Start/end offsets are consistent with the region positions recorded in the seqspec YAML (barcode/UMI/cDNA regions within each read)
- The generated string is accepted as a valid technology by the target preprocessing tool (STARsolo/kb) without manual edits

## Limitations

- seqspec index formats the technology string on a per-tool basis, so a string generated with -t kb is not necessarily valid for other tools
- The output depends on a validated spec.yaml being available; incomplete or unvalidated assay specifications will yield incorrect positional indexing
- The article's SRA prefetch and fastq-dump section is incomplete ('todo: prefetch and fastq-dump'), so obtaining FASTQs from SRA is not fully covered by the documented workflow

## Evidence

- [results] seqspec index extracts the barcode, UMI, and cDNA Regions from a seqspec, positionally indexes them, and formats them on a per-tool basis: [evidence span withheld — anchor status: not_found]
- [abstract] We present seqspec, a machine-readable specification for libraries produced by genomics assays: "We present seqspec, a machine-readable specification for libraries produced by genomics assays that facilitates standardization of preprocessing and enables tracking and comparison of genomics assays"
- [methods] pip install git+https://github.com/IGVF/seqspec.git: "pip install git+https://github.com/IGVF/seqspec.git"
- [methods] A sample dataset derived from the original data can be found here: https://github.com/IGVF/seqspec/tree/main/specs/dogmaseq-dig: "A sample dataset derived from the original data can be found here: https://github.com/IGVF/seqspec/tree/main/specs/dogmaseq-dig"
- [methods] fastqs/rna_R1_SRR18677638.fastq.gz: "fastqs/rna_R1_SRR18677638.fastq.gz"
